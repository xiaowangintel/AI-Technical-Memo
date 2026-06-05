# common.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/w8a8/fp8/common.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines cross-platform helper routines shared by FP8 quantization kernels, including platform selection, atomic max on floats, and scaled scalar FP8 conversion. / [CN] 定义 FP8 量化内核共用的跨平台辅助函数，包括平台选择、浮点原子最大值，以及带 scale 的标量 FP8 转换。

## Line-by-Line Analysis / 逐行分析
### Platform-specific include selection / 平台相关头文件选择
```cpp
#ifndef USE_ROCM
  #include "nvidia/quant_utils.cuh"
#else
  #include "amd/quant_utils.cuh"
#endif
```
**EN:** `common.cuh` deliberately hides backend differences. Higher-level kernels include this file and automatically receive the correct CUDA or ROCm conversion helpers.
**CN:** `common.cuh` 有意屏蔽后端差异。更上层的内核只需要包含这个文件，就能自动获得正确的 CUDA 或 ROCm 转换辅助函数。

### Choosing the preferred FP8 flavor / 选择首选 FP8 规格
```cpp
static bool is_fp8_ocp() {
#ifndef USE_ROCM
  return true;
#else
  auto dprops = at::cuda::getCurrentDeviceProperties();
  std::string device_arch = dprops->gcnArchName;
  size_t substring = device_arch.find("gfx94");
  return substring == std::string::npos;
#endif
}
```
**EN:** CUDA always reports OCP-style FP8 here. On ROCm, the helper checks the current GPU architecture string and avoids OCP on `gfx94*`, where a different FP8 variant is preferred.
**CN:** 在 CUDA 下这里总是返回 OCP 风格 FP8。在 ROCm 下，该辅助函数会检查当前 GPU 的架构字符串，并在 `gfx94*` 上避开 OCP，改用更合适的 FP8 变体。

### Atomic max for floating-point scales / 用于 scale 的浮点原子最大值
```cpp
__device__ __forceinline__ float atomicMaxFloat(float* addr, float value) {
  float old;
  old = (value >= 0)
            ? __int_as_float(atomicMax((int*)addr, __float_as_int(value)))
            : __uint_as_float(
                  atomicMin((unsigned int*)addr, __float_as_uint(value)));
  return old;
}
```
**EN:** CUDA/HIP do not offer a universal `atomicMax(float)` for all cases, especially around signed ordering. This helper implements correct behavior by routing non-negative and negative values through integer atomics with bit reinterpretation.
**CN:** CUDA/HIP 并没有在所有场景下都提供现成的 `atomicMax(float)`，尤其是涉及有符号排序时。这里通过位重解释，把非负值和负值分别映射到整数原子操作上，从而得到正确的行为。

### Scalar quantize helper with optional inverse scale / 可选择倒数 scale 的标量量化辅助函数
```cpp
template <bool is_scale_inverted, typename fp8_type>
__device__ __forceinline__ fp8_type scaled_fp8_conversion(float const val,
                                                          float const scale) {
  float x = 0.0f;
  if constexpr (is_scale_inverted) {
    x = val * scale;
  } else {
    x = val / scale;
  }

  float r =
      fmaxf(-quant_type_max_v<fp8_type>, fminf(x, quant_type_max_v<fp8_type>));
```
**EN:** This helper standardizes FP8 saturation and scaling. Some kernels pass the true scale, others pass its reciprocal; the boolean template eliminates the branch at compile time while keeping one common implementation.
**CN:** 这个辅助函数统一了 FP8 的缩放与饱和逻辑。有些内核传入真实的 scale，有些内核传入倒数；布尔模板参数会在编译期消除分支，同时保留一份公共实现。

### Backend-specific final cast / 后端相关的最终转换
```cpp
#ifndef USE_ROCM
  return fp8::vec_conversion<fp8_type, float>(r);
#else
  return fp8::cvt_c10<fp8_type>(r);
#endif
```
**EN:** NVIDIA uses `vec_conversion` because its CUDA-side utilities already expose float-to-FP8 conversion that way. ROCm uses `cvt_c10` to bridge into PyTorch FP8 wrapper types or HIP FP8 encodings.
**CN:** NVIDIA 侧使用 `vec_conversion`，因为其 CUDA 工具层已经按这种方式暴露了 float 到 FP8 的转换。ROCm 侧则使用 `cvt_c10`，以桥接到 PyTorch FP8 包装类型或 HIP FP8 编码。

## Key Concepts / 关键概念
- **Backend abstraction**: this header is the thin common layer above the platform-specific FP8 implementations. / **后端抽象层**：该头文件是平台专用 FP8 实现之上的一层轻量公共抽象。
- **Numerically safe saturation**: conversion always clamps into the representable FP8 range before encoding. / **数值安全的饱和**：在编码前总会先把值裁剪到可表示的 FP8 范围内。
- **Compile-time scale convention**: `is_scale_inverted` avoids duplicated helper functions. / **编译期 scale 约定**：`is_scale_inverted` 避免为不同 scale 方向重复写辅助函数。

## Dependencies / 依赖关系
- `nvidia/quant_utils.cuh` or `amd/quant_utils.cuh` provides the backend-specific conversion intrinsics. / `nvidia/quant_utils.cuh` 或 `amd/quant_utils.cuh` 提供后端相关的转换 intrinsic。
- `libtorch_stable/quantization/vectorization.cuh` and `../../utils.cuh` contribute vectorization and quantization constants used by callers. / `libtorch_stable/quantization/vectorization.cuh` 与 `../../utils.cuh` 为调用者提供向量化与量化常量支持。
- `quant_type_max_v<fp8_type>` is the key constant used to clamp values before encoding. / `quant_type_max_v<fp8_type>` 是编码前裁剪数值范围的关键常量。
