# utils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/utils.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Headers and compile-time setup
```cpp
/* Copyright 2025 SGLang Team. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

#pragma once

#include <ATen/Tensor.h>
#include <cuda_runtime.h>
#include <torch/all.h>

#ifdef USE_ROCM
#include <hip/hip_runtime.h>
#endif
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 26-48: Runtime integration and dispatch
```cpp
#ifdef USE_ROCM
// Adapted from flashinfer-rocm [PR#491](https://github.com/flashinfer-ai/flashinfer/pull/491)
#define _DISPATCH_CASE_F16(c_type, ...) \
  case at::ScalarType::Half: {          \
    using c_type = __half;              \
    return __VA_ARGS__();               \
  }

#define _DISPATCH_CASE_BF16(c_type, ...) \
  case at::ScalarType::BFloat16: {       \
    using c_type = __hip_bfloat16;       \
    return __VA_ARGS__();                \
  }
#endif  // USE_ROCM

#ifndef USE_ROCM
// Adapt from FlashInfer
#ifdef FLASHINFER_ENABLE_F16
#define _DISPATCH_CASE_F16(c_type, ...) \
  case at::ScalarType::Half: {          \
    using c_type = nv_half;             \
    return __VA_ARGS__();               \
  }
```
**EN:** This section uses `__VA_ARGS__`, `c_type` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`__VA_ARGS__`、`c_type`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 49-72: Runtime integration and dispatch
```cpp
#else
#define _DISPATCH_CASE_F16(c_type, ...)
#endif  // FLASHINFER_ENABLE_F16

#ifdef FLASHINFER_ENABLE_BF16
#define _DISPATCH_CASE_BF16(c_type, ...) \
  case at::ScalarType::BFloat16: {       \
    using c_type = nv_bfloat16;          \
    return __VA_ARGS__();                \
  }
#else
#define _DISPATCH_CASE_BF16(c_type, ...)
#endif  // FLASHINFER_ENABLE_BF16

#ifdef FLASHINFER_ENABLE_FP8_E4M3
#define _DISPATCH_CASE_FP8_E4M3(c_type, ...) \
  case at::ScalarType::Float8_e4m3fn: {      \
    using c_type = __nv_fp8_e4m3;            \
    return __VA_ARGS__();                    \
  }
#else
#define _DISPATCH_CASE_FP8_E4M3(c_type, ...)
#endif  // FLASHINFER_ENABLE_FP8_E4M3
```
**EN:** This section uses `__VA_ARGS__`, `c_type` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`__VA_ARGS__`、`c_type`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 73-95: Runtime integration and dispatch
```cpp
#ifdef FLASHINFER_ENABLE_FP8_E5M2
#define _DISPATCH_CASE_FP8_E5M2(c_type, ...) \
  case at::ScalarType::Float8_e5m2: {        \
    using c_type = __nv_fp8_e5m2;            \
    return __VA_ARGS__();                    \
  }
#else
#define _DISPATCH_CASE_FP8_E5M2(c_type, ...)
#endif  // FLASHINFER_ENABLE_FP8_E5M2

#define DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FP16(pytorch_dtype, c_type, ...)                 \
  [&]() -> bool {                                                                        \
    switch (pytorch_dtype) {                                                             \
      _DISPATCH_CASE_F16(c_type, __VA_ARGS__)                                            \
      _DISPATCH_CASE_BF16(c_type, __VA_ARGS__)                                           \
      default:                                                                           \
        std::ostringstream oss;                                                          \
        oss << __PRETTY_FUNCTION__ << " failed to dispatch data type " << pytorch_dtype; \
        TORCH_CHECK(false, oss.str());                                                   \
        return false;                                                                    \
    }                                                                                    \
  }()
```
**EN:** This section uses `_DISPATCH_CASE_FP8_E5M2`, `__VA_ARGS__`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_DISPATCH_CASE_FP8_E5M2`、`__VA_ARGS__`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 96-123: Runtime integration and dispatch
```cpp
#define DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FP8(pytorch_dtype, c_type, ...)                      \
  [&]() -> bool {                                                                            \
    switch (pytorch_dtype) {                                                                 \
      _DISPATCH_CASE_FP8_E4M3(c_type, __VA_ARGS__)                                           \
      _DISPATCH_CASE_FP8_E5M2(c_type, __VA_ARGS__)                                           \
      default:                                                                               \
        std::ostringstream oss;                                                              \
        oss << __PRETTY_FUNCTION__ << " failed to dispatch fp8 data type " << pytorch_dtype; \
        TORCH_CHECK(false, oss.str());                                                       \
        return false;                                                                        \
    }                                                                                        \
  }()

#define DISPATCH_PYTORCH_DTYPE_TO_CTYPE(pytorch_dtype, c_type, ...)                      \
  [&]() -> bool {                                                                        \
    switch (pytorch_dtype) {                                                             \
      _DISPATCH_CASE_F16(c_type, __VA_ARGS__)                                            \
      _DISPATCH_CASE_BF16(c_type, __VA_ARGS__)                                           \
      _DISPATCH_CASE_FP8_E4M3(c_type, __VA_ARGS__)                                       \
      _DISPATCH_CASE_FP8_E5M2(c_type, __VA_ARGS__)                                       \
      default:                                                                           \
        std::ostringstream oss;                                                          \
        oss << __PRETTY_FUNCTION__ << " failed to dispatch data type " << pytorch_dtype; \
        TORCH_CHECK(false, oss.str());                                                   \
        return false;                                                                    \
    }                                                                                    \
  }()
```
**EN:** This section uses `DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FP8`, `DISPATCH_PYTORCH_DTYPE_TO_CTYPE`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FP8`、`DISPATCH_PYTORCH_DTYPE_TO_CTYPE`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 124-148: Runtime integration and dispatch
```cpp
#define _DISPATCH_SWITCH(var_name, cond, ...)                                           \
  [&]() -> bool {                                                                       \
    switch (cond) {                                                                     \
      __VA_ARGS__                                                                       \
      default:                                                                          \
        std::ostringstream oss;                                                         \
        oss << __PRETTY_FUNCTION__ << " failed to dispatch " var_name " " << int(cond); \
        TORCH_CHECK(false, oss.str());                                                  \
        return false;                                                                   \
    }                                                                                   \
  }()

#define _DISPATCH_SWITCH_U16x2(var1_name, var2_name, cond1, cond2, ...)                                             \
  [&]() -> bool {                                                                                                   \
    switch (pack_u16(cond1, cond2)) {                                                                               \
      __VA_ARGS__                                                                                                   \
      default:                                                                                                      \
        std::ostringstream oss;                                                                                     \
        oss << __PRETTY_FUNCTION__ << " failed to dispatch (" var1_name ", " var2_name "): (" << int(cond1) << ", " \
            << int(cond2) << ")";                                                                                   \
        TORCH_CHECK(false, oss.str());                                                                              \
        return false;                                                                                               \
    }                                                                                                               \
  }()
```
**EN:** This section uses `_DISPATCH_SWITCH`, `_DISPATCH_SWITCH_U16x2`, `int` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_DISPATCH_SWITCH`、`_DISPATCH_SWITCH_U16x2`、`int`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 149-172: Templates, aliases, and constants
```cpp
#define _DISPATCH_CASE(case_expr, case_var, ...) \
  case case_expr: {                              \
    constexpr auto case_var = case_expr;         \
    return __VA_ARGS__();                        \
  }

#define _DISPATCH_CASE_U16x2(case_expr1, case_expr2, case_var1, case_var2, ...) \
  case pack_u16(case_expr1, case_expr2): {                                      \
    constexpr auto case_var1 = case_expr1;                                      \
    constexpr auto case_var2 = case_expr2;                                      \
    return __VA_ARGS__();                                                       \
  }

#define DISPATCH_BOOL(expr, const_expr, ...) \
  [&]() -> bool {                            \
    if (expr) {                              \
      constexpr bool const_expr = true;      \
      return __VA_ARGS__();                  \
    } else {                                 \
      constexpr bool const_expr = false;     \
      return __VA_ARGS__();                  \
    }                                        \
  }()
```
**EN:** This section defines `DISPATCH_BOOL`, `__VA_ARGS__`, `case_var`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`DISPATCH_BOOL`、`__VA_ARGS__`、`case_var`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 173-198: Runtime integration and dispatch
```cpp
inline void check_shape(const at::Tensor& a, const at::Tensor& b, const char* a_name, const char* b_name) {
  TORCH_CHECK(a.dim() == b.dim(), a_name, ".dim() != ", b_name, ".dim(). ", a.dim(), " vs ", b.dim());
  for (int i = 0; i < a.dim(); ++i) {
    TORCH_CHECK(a.size(i) == b.size(i), a_name, ".size(", i, ") != ", b_name, ".size(", i, ")");
  }
}

inline constexpr uint32_t pack_u16(uint16_t a, uint16_t b) {
  return (uint32_t(a) << 16) | uint32_t(b);
}

#define CHECK_GQA_HEAD_DIVISIBLE(num_qo_heads, num_kv_heads) \
  TORCH_CHECK(                                               \
      num_qo_heads % num_kv_heads == 0,                      \
      "num_qo_heads(",                                       \
      num_qo_heads,                                          \
      ") must be divisible by num_kv_heads(",                \
      num_kv_heads,                                          \
      ")")

#define CHECK_CUDA(x) TORCH_CHECK(x.is_cuda(), #x " must be a CUDA tensor")

#define CHECK_CONTIGUOUS(x) TORCH_CHECK(x.is_contiguous(), #x " must be contiguous")
#define CHECK_LAST_DIM_CONTIGUOUS(x) \
  TORCH_CHECK(x.strides()[x.strides().size() - 1] == 1, #x "must be contiguous at last dimension")
```
**EN:** This section uses `check_shape`, `pack_u16`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`check_shape`、`pack_u16`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 199-218: Runtime integration and dispatch
```cpp
#define CHECK_INPUT(x) \
  CHECK_CUDA(x);       \
  CHECK_CONTIGUOUS(x)
#define CHECK_LAST_DIM_CONTIGUOUS_INPUT(x) \
  CHECK_CUDA(x);                           \
  CHECK_LAST_DIM_CONTIGUOUS(x)

#define CHECK_DIM(d, x) TORCH_CHECK(x.dim() == d, #x " must be a " #d "D tensor")

#define CHECK_SHAPE(a, b) check_shape(a, b, #a, #b)

#define CHECK_EQ(a, b) TORCH_CHECK((a) == (b), "CHECK_EQ(" #a ", " #b ") failed. ", a, " vs ", b)

#define CHECK_GE(a, b) TORCH_CHECK((a) >= (b), "CHECK_GE(" #a ", " #b ") failed. ", a, " vs ", b)

inline bool is_float8_tensor(const at::Tensor& tensor) {
  return tensor.scalar_type() == at::ScalarType::Float8_e4m3fn || tensor.scalar_type() == at::ScalarType::Float8_e5m2;
}
#endif  // USE_ROCM
```
**EN:** This section uses `CHECK_LAST_DIM_CONTIGUOUS`, `CHECK_INPUT`, `CHECK_CONTIGUOUS` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_LAST_DIM_CONTIGUOUS`、`CHECK_INPUT`、`CHECK_CONTIGUOUS`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 219-244: Types and data layout
```cpp
struct cuda_error : public std::runtime_error {
  /**
   * @brief Constructs a `cuda_error` object with the given `message`.
   *
   * @param message The error char array used to construct `cuda_error`
   */
  cuda_error(const char* message) : std::runtime_error(message) {}
  /**
   * @brief Constructs a `cuda_error` object with the given `message` string.
   *
   * @param message The `std::string` used to construct `cuda_error`
   */
  cuda_error(std::string const& message) : cuda_error{message.c_str()} {}
};

#define CHECK_CUDA_SUCCESS(cmd)                                         \
  do {                                                                  \
    cudaError_t e = cmd;                                                \
    if (e != cudaSuccess) {                                             \
      std::stringstream _message;                                       \
      auto s = cudaGetErrorString(e);                                   \
      _message << std::string(s) + "\n" << __FILE__ << ':' << __LINE__; \
      throw cuda_error(_message.str());                                 \
    }                                                                   \
  } while (0)
```
**EN:** This section defines `cuda_error`, `cudaGetErrorString`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`cuda_error`、`cudaGetErrorString`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 245-270: Runtime integration and dispatch
```cpp
#define CHECK_IS_CUDA(x) TORCH_CHECK(x.device().is_cuda(), #x " must be a CUDA tensor")
#define CHECK_IS_CONTIGUOUS(x) TORCH_CHECK(x.is_contiguous(), #x " must be contiguous")
#define CHECK_CUDA_INPUT(x) \
  CHECK_IS_CUDA(x);         \
  CHECK_IS_CONTIGUOUS(x)

inline int getSMVersion() {
  int device{-1};
  CHECK_CUDA_SUCCESS(cudaGetDevice(&device));
  int sm_major = 0;
  int sm_minor = 0;
  CHECK_CUDA_SUCCESS(cudaDeviceGetAttribute(&sm_major, cudaDevAttrComputeCapabilityMajor, device));
  CHECK_CUDA_SUCCESS(cudaDeviceGetAttribute(&sm_minor, cudaDevAttrComputeCapabilityMinor, device));
  return sm_major * 10 + sm_minor;
}

inline bool isDeviceType(const std::string& device_type) {
  int deviceCount;
  CHECK_CUDA_SUCCESS(cudaGetDeviceCount(&deviceCount));

  int device_id = -1;
  if (deviceCount >= 1) {
    CHECK_CUDA_SUCCESS(cudaGetDevice(&device_id));
  } else {
    return false;
  }
```
**EN:** This section uses `CHECK_IS_CONTIGUOUS`, `isDeviceType`, `CHECK_IS_CUDA` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_IS_CONTIGUOUS`、`isDeviceType`、`CHECK_IS_CUDA`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 271-295: Control flow and branching
```cpp

  cudaDeviceProp prop;
  CHECK_CUDA_SUCCESS(cudaGetDeviceProperties(&prop, device_id));
  if (device_type == std::string(prop.name)) {
    return true;
  }
  return false;
}

inline bool getBoolEnv(char const* name) {
  char const* env = std::getenv(name);
  return env && env[0] == '1' && env[1] == '\0';
}

inline bool getEnvEnablePDL() {
  static std::once_flag flag;
  static bool enablePDL = false;
  std::call_once(flag, [&]() {
    if (getSMVersion() >= 90) {
      // PDL will be enabled by setting the env variables `TRTLLM_ENABLE_PDL` to `1`
      enablePDL = getBoolEnv("TRTLLM_ENABLE_PDL");
    }
  });
  return enablePDL;
}
```
**EN:** This section drives `getBoolEnv`, `getEnvEnablePDL`, `std::call_once` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`getBoolEnv`、`getEnvEnablePDL`、`std::call_once`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 296-322: Device helpers and synchronization
```cpp

// SGLANG_SHFL_XOR_* adapted from https://github.com/vllm-project/vllm/blob/v0.7.3/csrc/cuda_compat.h#L19-L28
#ifndef USE_ROCM
#define SGLANG_SHFL_XOR_SYNC(mask, var, lane_mask) __shfl_xor_sync((mask), (var), (lane_mask))
#define SGLANG_SHFL_XOR_SYNC_WIDTH(mask, var, lane_mask, width) __shfl_xor_sync((mask), (var), (lane_mask), (width))
#else
#define SGLANG_SHFL_XOR_SYNC(mask, var, lane_mask) __shfl_xor((var), (lane_mask))
#define SGLANG_SHFL_XOR_SYNC_WIDTH(mask, var, lane_mask, width) __shfl_xor((var), (lane_mask), (width))
#endif

#define DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FLOAT_FP16(pytorch_dtype, c_type, ...)           \
  [&]() -> bool {                                                                        \
    switch (pytorch_dtype) {                                                             \
      case at::ScalarType::Float: {                                                      \
        using c_type = float;                                                            \
        return __VA_ARGS__();                                                            \
      }                                                                                  \
        _DISPATCH_CASE_F16(c_type, __VA_ARGS__)                                          \
        _DISPATCH_CASE_BF16(c_type, __VA_ARGS__)                                         \
      default:                                                                           \
        std::ostringstream oss;                                                          \
        oss << __PRETTY_FUNCTION__ << " failed to dispatch data type " << pytorch_dtype; \
        TORCH_CHECK(false, oss.str());                                                   \
        return false;                                                                    \
    }                                                                                    \
  }()
```
**EN:** This section implements `SGLANG_SHFL_XOR_SYNC`, `__VA_ARGS__`, `TORCH_CHECK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`SGLANG_SHFL_XOR_SYNC`、`__VA_ARGS__`、`TORCH_CHECK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 323-351: Device helpers and synchronization
```cpp
#define DISPATCH_CASE_INTEGRAL_TYPES(...)              \
  AT_DISPATCH_CASE(at::ScalarType::Byte, __VA_ARGS__)  \
  AT_DISPATCH_CASE(at::ScalarType::Char, __VA_ARGS__)  \
  AT_DISPATCH_CASE(at::ScalarType::Short, __VA_ARGS__) \
  AT_DISPATCH_CASE(at::ScalarType::Int, __VA_ARGS__)   \
  AT_DISPATCH_CASE(at::ScalarType::Long, __VA_ARGS__)

#define DISPATCH_INTEGRAL_TYPES(TYPE, NAME, ...) \
  AT_DISPATCH_SWITCH(TYPE, NAME, DISPATCH_CASE_INTEGRAL_TYPES(__VA_ARGS__))

#define DISPATCH_CASE_FLOAT_TYPES(...)                 \
  AT_DISPATCH_CASE(at::ScalarType::Float, __VA_ARGS__) \
  AT_DISPATCH_CASE(at::ScalarType::Half, __VA_ARGS__)  \
  AT_DISPATCH_CASE(at::ScalarType::BFloat16, __VA_ARGS__)

#define DISPATCH_FLOAT_TYPES(TYPE, NAME, ...) AT_DISPATCH_SWITCH(TYPE, NAME, DISPATCH_CASE_FLOAT_TYPES(__VA_ARGS__))

#define CEILDIV(x, y) (((x) + (y) - 1) / (y))

#ifndef USE_ROCM
#define WARP_SIZE 32
#else
#if defined(__GFX9__) || !defined(__HIP_DEVICE_COMPILE__)
#define WARP_SIZE 64
#else
#define WARP_SIZE 32
#endif
#endif
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 352-370: Device helpers and synchronization
```cpp
#ifdef USE_ROCM

#include "hip/hip_math_def.h"
#include "hip/hip_vec_dtypes.h"

#else

template <typename srcDtype>
__device__ __forceinline__ float castToFloat(srcDtype val) {
  return static_cast<srcDtype>(val);
}

template <typename dstDtype>
__device__ __forceinline__ dstDtype castFromFloat(float val) {
  return static_cast<dstDtype>(val);
}

#endif
```
**EN:** This section implements `castToFloat`, `castFromFloat`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`castToFloat`、`castFromFloat`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 371-393: Runtime integration and dispatch
```cpp
// add FP8 support
#ifndef USE_ROCM
#include <c10/util/Float8_e4m3fn.h>
using FP8_TYPE = c10::Float8_e4m3fn;
C10_HOST_DEVICE constexpr auto FP8_E4M3_MAX = std::numeric_limits<FP8_TYPE>::max();
#else  // USE_ROCM
#if HIP_FP8_TYPE_FNUZ
#include <c10/util/Float8_e4m3fnuz.h>
using FP8_TYPE = c10::Float8_e4m3fnuz;
constexpr auto FP8_E4M3_MAX = 224.0f;
#else
#if HIP_FP8_TYPE_E4M3
#include <c10/util/Float8_e4m3fn.h>
using FP8_TYPE = c10::Float8_e4m3fn;
C10_HOST_DEVICE constexpr auto FP8_E4M3_MAX = std::numeric_limits<FP8_TYPE>::max();
#else
#error "fp8 is not supported in this processor (arch < gfx942)."
#endif  // HIP_FP8_TYPE_E4M3
#endif  // HIP_FP8_TYPE_FNUZ
#endif  // USE_ROCM

#define FULL_MASK 0xffffffff
```
**EN:** This section uses `max`, `FP8_TYPE`, `FP8_E4M3_MAX` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`max`、`FP8_TYPE`、`FP8_E4M3_MAX`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 394-418: Device helpers and synchronization
```cpp
__device__ __forceinline__ float atomicMaxFloat(float* addr, float value) {
#ifndef USE_ROCM
  float old;
  old = (value >= 0) ? __int_as_float(atomicMax((int*)addr, __float_as_int(value)))
                     : __uint_as_float(atomicMin((unsigned int*)addr, __float_as_uint(value)));
  return old;
#else
  int* addr_as_i = (int*)addr;
  int old = *addr_as_i, assumed;
  do {
    assumed = old;
    old = atomicCAS(addr_as_i, assumed, __float_as_int(fmaxf(value, __int_as_float(assumed))));
  } while (assumed != old);
  return __int_as_float(old);
#endif
}

__device__ __forceinline__ float warpReduceMax(float value) {
  value = fmaxf(value, __shfl_xor_sync(FULL_MASK, value, 16));
  value = fmaxf(value, __shfl_xor_sync(FULL_MASK, value, 8));
  value = fmaxf(value, __shfl_xor_sync(FULL_MASK, value, 4));
  value = fmaxf(value, __shfl_xor_sync(FULL_MASK, value, 2));
  value = fmaxf(value, __shfl_xor_sync(FULL_MASK, value, 1));
  return value;
}
```
**EN:** This section implements `atomicMaxFloat`, `warpReduceMax`, `__int_as_float`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`atomicMaxFloat`、`warpReduceMax`、`__int_as_float`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 419-441: Device helpers and synchronization
```cpp

__device__ __forceinline__ float blockReduceMax(float value) {
  static __shared__ float warpLevelMaxs[WARP_SIZE];
  const int laneId = threadIdx.x % WARP_SIZE;
  const int warpId = threadIdx.x / WARP_SIZE;

  value = warpReduceMax(value);

  if (laneId == 0) warpLevelMaxs[warpId] = value;
  __syncthreads();

  value = (threadIdx.x < blockDim.x / WARP_SIZE) ? warpLevelMaxs[laneId] : 0;
  if (warpId == 0) value = warpReduceMax(value);

  return value;
}

// Pads to a multiple of `alignment` rows.
inline torch::Tensor pad_tensor(const torch::Tensor& tensor, int64_t alignment = 4, bool is_column_major = false) {
  int64_t rows = tensor.size(0);
  int64_t cols = tensor.size(1);
  int64_t pad_rows = (alignment - (rows % alignment)) % alignment;  // Compute padding size
```
**EN:** This section implements `blockReduceMax`, `pad_tensor`, `warpReduceMax`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`blockReduceMax`、`pad_tensor`、`warpReduceMax`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 442-461: Runtime integration and dispatch
```cpp
  if (pad_rows == 0) {
    return tensor;  // Already aligned
  }

  torch::Tensor padding = torch::zeros({pad_rows, cols}, tensor.options());
  torch::Tensor tensor_padded = torch::cat({tensor, padding}, 0);  // Pad along rows

  // Ensure column-major layout
  if (is_column_major) {
    return tensor_padded.t().contiguous().t();
  }
  return tensor_padded;
}

// Get the next power of 2 of a number
inline uint32_t next_pow2(uint32_t x) noexcept {
  if (x <= 1) return 1;
  return 1u << (32 - __builtin_clz(x - 1));
}
```
**EN:** This section uses `options`, `t`, `__builtin_clz` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`options`、`t`、`__builtin_clz`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 462-469: Local implementation details
```cpp
/*
 * LDG Support
 */
#ifndef USE_ROCM
#define SGLANG_LDG(arg) __ldg(arg)
#else
#define SGLANG_LDG(arg) *(arg)
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。
- **Synchronization / atomics / 同步与原子操作**: Uses atomics or explicit synchronization to coordinate parallel work. / 使用原子操作或显式同步协调并行工作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `hip/hip_math_def.h`, `hip/hip_vec_dtypes.h`
- **External headers / 外部头文件**: `ATen/Tensor.h`, `cuda_runtime.h`, `torch/all.h`, `hip/hip_runtime.h`, `c10/util/Float8_e4m3fn.h`, `c10/util/Float8_e4m3fnuz.h`
- **Path context / 路径上下文**: include / utils.h
