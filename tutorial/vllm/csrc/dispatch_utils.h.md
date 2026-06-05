# dispatch_utils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/dispatch_utils.h`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Collects reusable macro-based dispatch helpers that convert runtime tensor metadata and small integer parameters into compile-time template constants. **CN:** 集中定义可复用的宏分发辅助工具，把运行时张量元数据和小整数参数转换为编译期模板常量。

## Line-by-Line Analysis / 逐行分析

### Floating-point, FP8, and quantized dtype dispatch / 浮点、FP8 与量化 dtype 分发
```cpp
// Need a special dispatch case macro since we will nest the FP8 dispatch.
// Instead of the usual 'scalar_t', this names the dispatched type 'fp8_t'.
#define AT_DISPATCH_FP8_CASE(enum_type, ...) \
  AT_PRIVATE_CASE_TYPE_USING_HINT(enum_type, fp8_t, __VA_ARGS__)

#define VLLM_DISPATCH_CASE_FLOATING_TYPES(...)         \
  AT_DISPATCH_CASE(at::ScalarType::Float, __VA_ARGS__) \
  AT_DISPATCH_CASE(at::ScalarType::Half, __VA_ARGS__)  \
  AT_DISPATCH_CASE(at::ScalarType::BFloat16, __VA_ARGS__)

#define VLLM_DISPATCH_FLOATING_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME, VLLM_DISPATCH_CASE_FLOATING_TYPES(__VA_ARGS__))

#define VLLM_DISPATCH_CASE_HALF_TYPES(...)            \
  AT_DISPATCH_CASE(at::ScalarType::Half, __VA_ARGS__) \
  AT_DISPATCH_CASE(at::ScalarType::BFloat16, __VA_ARGS__)

#define VLLM_DISPATCH_HALF_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME, VLLM_DISPATCH_CASE_HALF_TYPES(__VA_ARGS__))

// ROCm devices might use either fn or fnuz, so set up dispatch table for both.
// A host-based check at runtime will create a preferred FP8 type for ROCm
// such that the correct kernel is dispatched.
#ifdef USE_ROCM
  #define VLLM_DISPATCH_CASE_FP8_TYPES(...)                          \
    AT_DISPATCH_FP8_CASE(at::ScalarType::Float8_e4m3fn, __VA_ARGS__) \
    AT_DISPATCH_FP8_CASE(at::ScalarType::Float8_e4m3fnuz, __VA_ARGS__)

  #define VLLM_DISPATCH_CASE_QUANT_TYPES(...)                      \
    AT_DISPATCH_CASE(at::ScalarType::Float8_e4m3fn, __VA_ARGS__)   \
    AT_DISPATCH_CASE(at::ScalarType::Float8_e4m3fnuz, __VA_ARGS__) \
    AT_DISPATCH_CASE(at::ScalarType::Char, __VA_ARGS__)
#else
  #define VLLM_DISPATCH_CASE_FP8_TYPES(...) \
    AT_DISPATCH_FP8_CASE(at::ScalarType::Float8_e4m3fn, __VA_ARGS__)

  #define VLLM_DISPATCH_CASE_QUANT_TYPES(...)                    \
    AT_DISPATCH_CASE(at::ScalarType::Float8_e4m3fn, __VA_ARGS__) \
    AT_DISPATCH_CASE(at::ScalarType::Char, __VA_ARGS__)
#endif

// When using this dispatch macro, the type is 'fp8_t' not 'scalar_t'.
// See AT_DISPATCH_FP8_CASE above.
#define VLLM_DISPATCH_FP8_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME, VLLM_DISPATCH_CASE_FP8_TYPES(__VA_ARGS__))

#define VLLM_DISPATCH_QUANT_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME, VLLM_DISPATCH_CASE_QUANT_TYPES(__VA_ARGS__))
```
**EN:** These macros wrap ATen dispatch primitives to cover the dtypes used heavily inside vLLM kernels. The ROCm branch expands FP8 dispatch to both e4m3fn and e4m3fnuz because AMD devices may expose either encoding.
**CN:** 这些宏基于 ATen 的 dispatch 原语，覆盖了 vLLM 内核中常见的数据类型。ROCm 分支会把 FP8 分发扩展到 e4m3fn 和 e4m3fnuz 两种编码，因为 AMD 设备可能暴露任一格式。

### Byte and integral dispatch families / Byte 与整数分发族
```cpp
#define VLLM_DISPATCH_CASE_FLOATING_AND_BYTE_TYPES(...)   \
  AT_DISPATCH_CASE(at::ScalarType::Float, __VA_ARGS__)    \
  AT_DISPATCH_CASE(at::ScalarType::Half, __VA_ARGS__)     \
  AT_DISPATCH_CASE(at::ScalarType::BFloat16, __VA_ARGS__) \
  AT_DISPATCH_CASE(at::ScalarType::Byte, __VA_ARGS__)

#define VLLM_DISPATCH_FLOATING_AND_BYTE_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME,                               \
                     VLLM_DISPATCH_CASE_FLOATING_AND_BYTE_TYPES(__VA_ARGS__))

#define VLLM_DISPATCH_CASE_INTEGRAL_TYPES(...)         \
  AT_DISPATCH_CASE(at::ScalarType::Byte, __VA_ARGS__)  \
  AT_DISPATCH_CASE(at::ScalarType::Char, __VA_ARGS__)  \
  AT_DISPATCH_CASE(at::ScalarType::Short, __VA_ARGS__) \
  AT_DISPATCH_CASE(at::ScalarType::Int, __VA_ARGS__)   \
  AT_DISPATCH_CASE(at::ScalarType::Long, __VA_ARGS__)

#define VLLM_DISPATCH_CASE_INTEGRAL_AND_UNSIGNED_TYPES(...) \
  AT_DISPATCH_CASE(at::ScalarType::Byte, __VA_ARGS__)       \
  AT_DISPATCH_CASE(at::ScalarType::Char, __VA_ARGS__)       \
  AT_DISPATCH_CASE(at::ScalarType::Short, __VA_ARGS__)      \
  AT_DISPATCH_CASE(at::ScalarType::Int, __VA_ARGS__)        \
  AT_DISPATCH_CASE(at::ScalarType::Long, __VA_ARGS__)       \
  AT_DISPATCH_CASE(at::ScalarType::UInt16, __VA_ARGS__)     \
  AT_DISPATCH_CASE(at::ScalarType::UInt32, __VA_ARGS__)     \
  AT_DISPATCH_CASE(at::ScalarType::UInt64, __VA_ARGS__)

#define VLLM_DISPATCH_INTEGRAL_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME, VLLM_DISPATCH_CASE_INTEGRAL_TYPES(__VA_ARGS__))

#define VLLM_DISPATCH_INTEGRAL_AND_UNSIGNED_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(                                              \
      TYPE, NAME, VLLM_DISPATCH_CASE_INTEGRAL_AND_UNSIGNED_TYPES(__VA_ARGS__))
```
**EN:** The next block provides convenience wrappers for float-or-byte, signed integral, and signed-plus-unsigned integral families. This keeps kernel launchers compact and consistent across many C++ extension entry points.
**CN:** 下一组宏为 float-or-byte、带符号整数以及带符号+无符号整数家族提供便捷包装，使大量 C++ 扩展入口中的 kernel launch 代码更紧凑一致。

### Compile-time value dispatch / 编译期数值分发
```cpp
#define VLLM_DISPATCH_VEC_SIZE(VEC_SIZE, ...) \
  switch (VEC_SIZE) {                         \
    case 16: {                                \
      constexpr int vec_size = 16;            \
      __VA_ARGS__();                          \
      break;                                  \
    }                                         \
    case 8: {                                 \
      constexpr int vec_size = 8;             \
      __VA_ARGS__();                          \
      break;                                  \
    }                                         \
    case 4: {                                 \
      constexpr int vec_size = 4;             \
      __VA_ARGS__();                          \
      break;                                  \
    }                                         \
    case 2: {                                 \
      constexpr int vec_size = 2;             \
      __VA_ARGS__();                          \
      break;                                  \
    }                                         \
    default: {                                \
      constexpr int vec_size = 1;             \
      __VA_ARGS__();                          \
      break;                                  \
    }                                         \
  }

#define VLLM_DISPATCH_BOOL(expr, const_expr, ...) \
  if (expr) {                                     \
    constexpr bool const_expr = true;             \
    __VA_ARGS__();                                \
  } else {                                        \
    constexpr bool const_expr = false;            \
    __VA_ARGS__();                                \
  }

#define VLLM_DISPATCH_GROUP_SIZE(group_size, const_group_size, ...) \
  if (group_size == 128) {                                          \
    constexpr int const_group_size = 128;                           \
    __VA_ARGS__();                                                  \
  } else if (group_size == 64) {                                    \
    constexpr int const_group_size = 64;                            \
    __VA_ARGS__();                                                  \
  }

#define VLLM_DISPATCH_RANK234(NUM_DIMS, ...)                                   \
  switch (NUM_DIMS) {                                                          \
    case 2: {                                                                  \
      constexpr int tensor_rank = 2;                                           \
      __VA_ARGS__();                                                           \
      break;                                                                   \
    }                                                                          \
    case 3: {                                                                  \
      constexpr int tensor_rank = 3;                                           \
      __VA_ARGS__();                                                           \
      break;                                                                   \
    }                                                                          \
    case 4: {                                                                  \
      constexpr int tensor_rank = 4;                                           \
      __VA_ARGS__();                                                           \
      break;                                                                   \
    }                                                                          \
    default:                                                                   \
      TORCH_CHECK(false, "Expects rank 2, 3 or 4 tensors but got ", NUM_DIMS); \
  }
```
**EN:** VLLM_DISPATCH_VEC_SIZE, VLLM_DISPATCH_BOOL, VLLM_DISPATCH_GROUP_SIZE, and VLLM_DISPATCH_RANK234 turn runtime integers/booleans into constexpr names inside the chosen branch. That pattern is critical when template-specialized kernels need compile-time constants for vector width, rank, or group size.
**CN:** VLLM_DISPATCH_VEC_SIZE、VLLM_DISPATCH_BOOL、VLLM_DISPATCH_GROUP_SIZE 和 VLLM_DISPATCH_RANK234 会把运行时整数/布尔值转成分支内部的 constexpr 名称。当模板特化内核需要在编译期固定向量宽度、张量 rank 或 group size 时，这种模式非常关键。

## Key Concepts / 关键概念

- **EN:** These macros reduce boilerplate around ATen's switch-based type dispatch.
  **CN:** 这些宏减少了围绕 ATen switch 型类型分发的样板代码。
- **EN:** Runtime-to-constexpr dispatch is a recurring pattern in CUDA extension code.
  **CN:** 把运行时参数转成 constexpr 是 CUDA 扩展代码中的常见模式。
- **EN:** ROCm FP8 support needs dual-type handling because the preferred host dtype is chosen dynamically.
  **CN:** ROCm 的 FP8 支持需要同时处理两种类型，因为宿主侧偏好 dtype 是动态决定的。

## Dependencies / 依赖关系

- **EN:** Built directly on top of torch/all.h and PyTorch dispatch macros.
  **CN:** 直接构建在 torch/all.h 与 PyTorch dispatch 宏之上。
- **EN:** Included by kernels such as fused_qknorm_rope_kernel.cu and fused_deepseek_v4_qnorm_rope_kv_insert_kernel.cu.
  **CN:** 被 fused_qknorm_rope_kernel.cu、fused_deepseek_v4_qnorm_rope_kv_insert_kernel.cu 等内核文件包含。
