# dispatch_utils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/dispatch_utils.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Stable-ABI dispatch macro layer that mirrors ATen-style type switching using PyTorch header-only APIs. / 稳定 ABI 的分发宏层，用 PyTorch header-only API 模拟 ATen 风格的类型分发。

## Line-by-Line Analysis / 逐行分析
### FP8 case helper
```cpp
#define VLLM_STABLE_DISPATCH_FP8_CASE(enum_type, ...) \
  THO_PRIVATE_CASE_TYPE_USING_HINT(enum_type, fp8_t, __VA_ARGS__)
```
**EN:** FP8 dispatch uses the alias `fp8_t` instead of the usual `scalar_t`, because FP8 dispatch is sometimes nested inside broader type switches and needs a distinct type name.
**CN:** FP8 分发使用 `fp8_t` 而不是常见的 `scalar_t`，因为 FP8 分发有时会嵌套在更大的类型分发中，需要一个独立的类型名来避免混淆。

### Floating and FP8 type switches
```cpp
#define VLLM_STABLE_DISPATCH_CASE_FLOATING_TYPES(...)                  \
  THO_DISPATCH_CASE(torch::headeronly::ScalarType::Float, __VA_ARGS__) \
  THO_DISPATCH_CASE(torch::headeronly::ScalarType::Half, __VA_ARGS__)  \
  THO_DISPATCH_CASE(torch::headeronly::ScalarType::BFloat16, __VA_ARGS__)
...
#ifdef USE_ROCM
  #define VLLM_STABLE_DISPATCH_CASE_FP8_TYPES(...)                 \
    VLLM_STABLE_DISPATCH_FP8_CASE(                                 \
        torch::headeronly::ScalarType::Float8_e4m3fn, __VA_ARGS__) \
    VLLM_STABLE_DISPATCH_FP8_CASE(                                 \
        torch::headeronly::ScalarType::Float8_e4m3fnuz, __VA_ARGS__)
```
**EN:** These macros centralize platform-sensitive dispatch behavior. CUDA exposes OCP FP8, while ROCm may need both OCP and FNUZ variants, so the header hides that difference behind a single vLLM macro family.
**CN:** 这些宏把平台相关的分发差异集中封装起来。CUDA 主要暴露 OCP FP8；ROCm 可能同时需要 OCP 与 FNUZ 两种格式，因此该头文件用统一的 vLLM 宏族屏蔽了这种差异。

### Half-type and boolean dispatch
```cpp
#define VLLM_STABLE_DISPATCH_HALF_TYPES(TYPE, NAME, ...) \
  THO_DISPATCH_SWITCH(TYPE, NAME,                        \
                      VLLM_STABLE_DISPATCH_CASE_HALF_TYPES(__VA_ARGS__))
...
#define VLLM_STABLE_DISPATCH_BOOL(expr, const_expr, ...) \
  if (expr) {                                            \
    constexpr bool const_expr = true;                    \
    __VA_ARGS__();                                       \
  } else {                                               \
    constexpr bool const_expr = false;                   \
    __VA_ARGS__();                                       \
  }
```
**EN:** The half-type helper narrows dispatch to `Half` and `BFloat16`, while the boolean helper turns a runtime flag into a compile-time constant branch. This pattern is heavily used by template-rich CUDA kernels.
**CN:** half 类型辅助宏把分发限制在 `Half` 与 `BFloat16`；布尔辅助宏则把运行时标志转成编译期常量分支。这种模式在模板化很强的 CUDA 内核中非常常见。

## Key Concepts / 关键概念
- **Stable ABI dispatch / 稳定 ABI 分发**: Avoids direct dependence on ATen dispatch macros.
- **Platform-specific FP8 handling / 平台相关 FP8 处理**: Abstracts CUDA vs ROCm FP8 enum differences.
- **Compile-time branching from runtime flags / 从运行时标志生成编译期分支**.

## Dependencies / 依赖关系
- Depends on `torch/headeronly` dispatch and scalar-type headers.
- Used by stable-ABI CUDA/C++ ops across `csrc/libtorch_stable`.
