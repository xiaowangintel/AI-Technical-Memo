# quant_conversions.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/fused_kernels/quant_conversions.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements low-level device-side conversion helpers from float-like values into int8 or FP8, with optional scale inversion. / [CN] 实现从 float 类值转换到 int8 或 FP8 的底层设备端辅助函数，并支持可选的 scale 取倒数语义。

## Line-by-Line Analysis / 逐行分析

### int8 conversion path
```cpp
static __device__ __forceinline__ int8_t float_to_int8_rn(float const x) {
#ifdef USE_ROCM
  static const float i8_min =
      static_cast<float>(std::numeric_limits<int8_t>::min());
  static const float i8_max =
      static_cast<float>(std::numeric_limits<int8_t>::max());
  // round
  float dst = std::nearbyint(x);
  // saturate

  // See https://github.com/pytorch/pytorch/issues/127666
  // See https://github.com/llvm/llvm-project/issues/95183
  // hip-clang std::clamp __glibcxx_assert_fail host function when building on
  // Arch/gcc14. The following replaces std::clamp usage with similar logic
  // dst = std::clamp(dst, i8_min, i8_max);
  dst = (dst < i8_min) ? i8_min : (dst > i8_max) ? i8_max : dst;
  return static_cast<int8_t>(dst);
#else
  // CUDA path
  uint32_t dst;
  asm volatile("cvt.rni.sat.s8.f32 %0, %1;" : "=r"(dst) : "f"(x));
  return reinterpret_cast<const int8_t&>(dst);
#endif
}
```
**EN:** `float_to_int8_rn` rounds to nearest and saturates into the int8 range. CUDA uses inline PTX (`cvt.rni.sat.s8.f32`) for a single instruction, while ROCm uses portable C++ logic plus a manual clamp because of toolchain-specific issues around `std::clamp`.
**CN:** `float_to_int8_rn` 会执行“最近整数舍入 + int8 饱和”。CUDA 路径用内联 PTX `cvt.rni.sat.s8.f32` 实现单指令转换；ROCm 路径则使用可移植 C++ 逻辑再手工 clamp，以绕开工具链对 `std::clamp` 的问题。

### FP8 clamp conversion
```cpp
template <typename fp8_type>
static __device__ __forceinline__ fp8_type float_to_fp8(float const x) {
  float const r =
      fmax(-quant_type_max_v<fp8_type>, fmin(x, quant_type_max_v<fp8_type>));
  return static_cast<fp8_type>(r);
}
```
**EN:** `float_to_fp8` clamps the input to the valid dynamic range of the target FP8 type and then relies on a plain cast. The target range comes from `quant_type_max_v`, so backend-specific max adjustments automatically flow into FP8 conversion.
**CN:** `float_to_fp8` 会先把输入裁剪到目标 FP8 类型的合法动态范围，再执行普通类型转换。目标范围来自 `quant_type_max_v`，因此后端特定的最大值调整会自动体现在 FP8 转换中。

### Type-specialized scaled quantization
```cpp
template <typename quant_type_t, bool is_scale_inverted, typename enable = void>
struct ScaledQuant;

template <typename quant_type_t, bool is_scale_inverted>
struct ScaledQuant<
    quant_type_t, is_scale_inverted,
    typename std::enable_if_t<std::is_same_v<quant_type_t, int8_t>>> {
  static __device__ __forceinline__ quant_type_t quant_fn(float const x,
                                                          float const scale) {
    if constexpr (is_scale_inverted) {
      return float_to_int8_rn(x * scale);
    } else {
      return float_to_int8_rn(x / scale);
    }
  }
};

template <typename quant_type_t, bool is_scale_inverted>
struct ScaledQuant<quant_type_t, is_scale_inverted,
                   typename std::enable_if_t<
                       std::is_same_v<quant_type_t, c10::Float8_e4m3fn> ||
                       std::is_same_v<quant_type_t, c10::Float8_e4m3fnuz>>> {
  static __device__ __forceinline__ quant_type_t quant_fn(float const x,
                                                          float const scale) {
    if constexpr (is_scale_inverted) {
      return float_to_fp8<quant_type_t>(x * scale);
    } else {
      return float_to_fp8<quant_type_t>(x / scale);
    }
  }
};
```
**EN:** `ScaledQuant` is specialized by output datatype. The int8 specialization uses `float_to_int8_rn`; the FP8 specialization uses `float_to_fp8`. The `is_scale_inverted` template flag decides whether the caller provides `scale` or `1/scale`.
**CN:** `ScaledQuant` 根据输出类型做特化：int8 版本调用 `float_to_int8_rn`，FP8 版本调用 `float_to_fp8`。模板参数 `is_scale_inverted` 决定调用方传入的是 `scale` 还是 `1/scale`。

### Loop-based conversion helper
```cpp
template <typename scalar_t, typename quant_type_t, bool is_scale_inverted>
__device__ void scaled_quant_conversion(quant_type_t* __restrict__ output,
                                        scalar_t const* __restrict__ input,
                                        float const scale, int const tid,
                                        int const num_elements,
                                        int const step) {
  for (int i = tid; i < num_elements; i += step) {
    output[i] = ScaledQuant<quant_type_t, is_scale_inverted>(input[i], scale);
  }
}
```
**EN:** `scaled_quant_conversion` is a tiny strided loop used by higher-level kernels: each thread walks over its assigned elements, applies the correct `ScaledQuant` specialization, and writes the quantized output.
**CN:** `scaled_quant_conversion` 是给更高层内核复用的小型跨步循环：每个线程遍历自己负责的元素，调用合适的 `ScaledQuant` 特化，并写出量化结果。

## Key Concepts / 关键概念
- EN: The file centralizes datatype-specific quantization behavior in one place.
  CN: 该文件把与数据类型相关的量化行为集中到一个位置。
- EN: The scale inversion flag avoids redundant reciprocal operations in callers that already precompute inverse scales.
  CN: scale 取倒数标志可避免调用方在已预先计算倒数时重复执行 reciprocal。
- EN: CUDA and ROCm share the same API even though the low-level conversion instructions differ.
  CN: 尽管底层转换指令不同，CUDA 与 ROCm 仍共享同一套 API。

## Dependencies / 依赖关系
- EN: Includes `vectorization.cuh` and FP8 common helpers for type definitions.
  CN: 包含 `vectorization.cuh` 与 FP8 common 头以获得类型定义。
- EN: Uses `quant_type_max_v` from `quantization/utils.cuh`.
  CN: 使用 `quantization/utils.cuh` 中的 `quant_type_max_v`。
- EN: Consumed by fused SiLU and layernorm quantization kernels.
  CN: 被融合 SiLU 与 layernorm 量化内核调用。
