# marlin_dtypes.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/marlin_dtypes.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Map vLLM scalar type IDs to CUDA scalar types, Tensor Core fragments, and conversion helpers. / [CN] 将 vLLM 的 scalar type ID 映射到 CUDA 标量类型、Tensor Core 片段类型和数值转换辅助函数。

## Line-by-Line Analysis / 逐行分析
### Primary template
```cpp
template <long scalar_type_id>
class MarlinScalarType {};
```
**EN:** The unspecialized template is intentionally empty. Every supported compute/weight type must provide a specialization, which makes unsupported combinations fail at compile time.
**CN:** 未特化模板故意保持为空。每一种受支持的计算/权重类型都必须提供特化，从而让不支持的组合在编译期失败。

### FP16 specialization
```cpp
template <>
class MarlinScalarType<vllm::kFloat16.id()> {
 public:
  using scalar_t = half;
  using scalar_t2 = half2;
  using FragA = Vec<half2, 4>;
  using FragB = Vec<half2, 2>;
  using FragC = Vec<float, 4>;
  using FragS = Vec<half2, 1>;
  using FragZP = Vec<half2, 4>;
  static __device__ float inline num2float(const half x) {
    return __half2float(x);
  }
};
```
**EN:** For FP16 activations/output, Marlin uses `half2`-based fragments for operand tiles and keeps accumulation in FP32. The helper methods convert between scalar and packed-two representations.
**CN:** 对于 FP16 激活/输出，Marlin 使用基于 `half2` 的片段表示操作数 tile，并在 FP32 中累加。辅助函数负责在标量和双元素打包表示之间转换。

### BF16 specialization
```cpp
template <>
class MarlinScalarType<vllm::kBFloat16.id()> {
 public:
  using scalar_t = nv_bfloat16;
  using scalar_t2 = nv_bfloat162;
  using FragA = Vec<nv_bfloat162, 4>;
  using FragB = Vec<nv_bfloat162, 2>;
  using FragC = Vec<float, 4>;
};
```
**EN:** The BF16 specialization mirrors the FP16 layout, but uses BF16 intrinsics and conversion helpers guarded for SM80+ where native BF16 support exists.
**CN:** BF16 特化基本复用了 FP16 的布局，但改用 BF16 内建类型和转换辅助函数，并通过条件编译限制在支持原生 BF16 的 SM80+ 架构上使用。

### FP8 specialization
```cpp
template <>
class MarlinScalarType<vllm::kFE4M3fn.id()> {
 public:
  using scalar_t = __nv_fp8_e4m3;
  using scalar_t2 = __nv_fp8x2_e4m3;
  using scalar_t4 = __nv_fp8x4_e4m3;
  using FragA = Vec<__nv_fp8x4_e4m3, 4>;
  using FragB = Vec<__nv_fp8x4_e4m3, 2>;
  using FragC = Vec<float, 4>;
};
```
**EN:** FP8 uses x4 packed fragment types because the MMA instruction consumes more densely packed 8-bit lanes than FP16/BF16. Output is still accumulated in FP32.
**CN:** FP8 使用 x4 打包片段类型，因为对应的 MMA 指令会比 FP16/BF16 更紧密地消费 8-bit 通道。输出仍然在 FP32 中累加。

### INT8 specialization
```cpp
template <>
class MarlinScalarType<vllm::kS8.id()> {
 public:
  using scalar_t = int8_t;
  using scalar_t2 = int16_t;
  using scalar_t4 = int32_t;
  using FragA = Vec<int32_t, 4>;
  using FragB = Vec<int32_t, 2>;
  using FragC = Vec<float, 4>;
};
```
**EN:** INT8 activations/weights use integer fragment storage so the tensor core integer MMA path can read them directly. Even here, Marlin later converts results into floating-point output fragments.
**CN:** INT8 激活/权重使用整数片段存储，这样 Tensor Core 的整数 MMA 路径可以直接读取它们。即便如此，Marlin 后续仍会把结果转成浮点输出片段。

### Reverse mapping by CUDA scalar type
```cpp
template <typename scalar_t>
class MarlinScalarType2 {};

template <>
class MarlinScalarType2<half> : public MarlinScalarType<vllm::kFloat16.id()> {};
```
**EN:** `MarlinScalarType2` lets the code start from a CUDA C++ type and recover the same trait bundle. That is useful in templated helper code where the scalar type is known but the vLLM enum is not.
**CN:** `MarlinScalarType2` 允许代码从 CUDA C++ 类型反向获取同一组 trait。这在某些模板辅助代码里很有用：那时已知标量类型，但不知道 vLLM 枚举值。

## Key Concepts / 关键概念
- One traits class defines fragment layouts, packed scalar aliases, and conversions together. / 一个 traits 类同时定义片段布局、打包标量别名和转换逻辑。
- Accumulation fragments are FP32 even when inputs are low precision. / 即使输入是低精度，累加片段也统一采用 FP32。

## Dependencies / 依赖关系
- Depends on `marlin.cuh` and `core/scalar_type.hpp`. / 依赖 `marlin.cuh` 和 `core/scalar_type.hpp`。
- Consumed by `marlin_mma.h`, `dequant.h`, `kernel.h`, and `marlin_template.h`. / 被 `marlin_mma.h`、`dequant.h`、`kernel.h` 和 `marlin_template.h` 使用。
