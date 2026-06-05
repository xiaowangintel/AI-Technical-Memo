# Exceptions.h — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/cuda/Exceptions.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file declares CUDA backend support, with primary focus on `CuDNNError`, `AT_CUDNN_FRONTEND_CHECK`, `AT_CUDNN_CHECK_WITH_SHAPES`.
- 用途（中文）: 该文件声明CUDA 后端支持，核心关注对象是 `CuDNNError`, `AT_CUDNN_FRONTEND_CHECK`, `AT_CUDNN_CHECK_WITH_SHAPES`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#pragma once

#include <cublas_v2.h>
#include <cusparse.h>
#include <c10/macros/Export.h>

#if !defined(USE_ROCM)
#include <cusolver_common.h>
#else
#include <hipsolver/hipsolver.h>
#endif

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 13-23
```cpp
#if defined(USE_CUDSS)
#include <cudss.h>
#endif

#include <ATen/Context.h>
#include <c10/util/Exception.h>
#include <c10/cuda/CUDAException.h>


namespace c10 {

```
- EN: Focus symbols: `c10`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`c10`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 24-33
```cpp
class CuDNNError : public c10::Error {
  using Error::Error;
};

}  // namespace c10

#define AT_CUDNN_FRONTEND_CHECK(EXPR, ...)                                                      \
  do {                                                                                          \
    auto error_object = EXPR;                                                                   \
    if (!error_object.is_good()) {                                                              \
```
- EN: Focus symbols: `CuDNNError`, `AT_CUDNN_FRONTEND_CHECK`, `c10`, `is_good`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`CuDNNError`, `AT_CUDNN_FRONTEND_CHECK`, `c10`, `is_good`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 34-43
```cpp
      TORCH_CHECK_WITH(CuDNNError, false,                                                       \
            "cuDNN Frontend error: ", error_object.get_message());                              \
    }                                                                                           \
  } while (0)                                                                                   \

#define AT_CUDNN_CHECK_WITH_SHAPES(EXPR, ...) AT_CUDNN_CHECK(EXPR, "\n", ##__VA_ARGS__)

// See Note [CHECK macro]
#define AT_CUDNN_CHECK(EXPR, ...)                                                               \
  do {                                                                                          \
```
- EN: Focus symbols: `AT_CUDNN_CHECK_WITH_SHAPES`, `AT_CUDNN_CHECK`, `TORCH_CHECK_WITH`, `get_message`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`AT_CUDNN_CHECK_WITH_SHAPES`, `AT_CUDNN_CHECK`, `TORCH_CHECK_WITH`, `get_message`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 44-57
```cpp
    cudnnStatus_t status = EXPR;                                                                \
    if (status != CUDNN_STATUS_SUCCESS) {                                                       \
      if (status == CUDNN_STATUS_NOT_SUPPORTED) {                                               \
        TORCH_CHECK_WITH(CuDNNError, false,                                                     \
            "cuDNN error: ",                                                                    \
            cudnnGetErrorString(status),                                                        \
            ". This error may appear if you passed in a non-contiguous input.", ##__VA_ARGS__); \
      } else {                                                                                  \
        TORCH_CHECK_WITH(CuDNNError, false,                                                     \
            "cuDNN error: ", cudnnGetErrorString(status), ##__VA_ARGS__);                       \
      }                                                                                         \
    }                                                                                           \
  } while (0)

```
- EN: Focus symbols: `TORCH_CHECK_WITH`, `cudnnGetErrorString`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`TORCH_CHECK_WITH`, `cudnnGetErrorString`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 58-70
```cpp
namespace at::cuda::blas {
C10_EXPORT const char* _cublasGetErrorEnum(cublasStatus_t error);
} // namespace at::cuda::blas

#define TORCH_CUDABLAS_CHECK(EXPR)                              \
  do {                                                          \
    cublasStatus_t __err = EXPR;                                \
    TORCH_CHECK(__err == CUBLAS_STATUS_SUCCESS,                 \
                "CUDA error: ",                                 \
                at::cuda::blas::_cublasGetErrorEnum(__err),     \
                " when calling `" #EXPR "`");                   \
  } while (0)

```
- EN: Focus symbols: `TORCH_CUDABLAS_CHECK`, `at::cuda::blas`, `_cublasGetErrorEnum`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CUDABLAS_CHECK`, `at::cuda::blas`, `_cublasGetErrorEnum`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 71-81
```cpp
const char *cusparseGetErrorString(cusparseStatus_t status);

#define TORCH_CUDASPARSE_CHECK(EXPR)                            \
  do {                                                          \
    cusparseStatus_t __err = EXPR;                              \
    TORCH_CHECK(__err == CUSPARSE_STATUS_SUCCESS,               \
                "CUDA error: ",                                 \
                cusparseGetErrorString(__err),                  \
                " when calling `" #EXPR "`");                   \
  } while (0)

```
- EN: Focus symbols: `TORCH_CUDASPARSE_CHECK`, `cusparseGetErrorString`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CUDASPARSE_CHECK`, `cusparseGetErrorString`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 82-91
```cpp
#if defined(USE_CUDSS)
namespace at::cuda::cudss {
C10_EXPORT const char* cudssGetErrorMessage(cudssStatus_t error);
} // namespace at::cuda::solver

#define TORCH_CUDSS_CHECK(EXPR)                                         \
  do {                                                                  \
    cudssStatus_t __err = EXPR;                                         \
    if (__err == CUDSS_STATUS_EXECUTION_FAILED) {                       \
      TORCH_CHECK_LINALG(                                               \
```
- EN: Focus symbols: `TORCH_CUDSS_CHECK`, `at::cuda::cudss`, `at::cuda::solver`, `cudssGetErrorMessage`, `TORCH_CHECK_LINALG`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`TORCH_CUDSS_CHECK`, `at::cuda::cudss`, `at::cuda::solver`, `cudssGetErrorMessage`, `TORCH_CHECK_LINALG`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 92-101
```cpp
          false,                                                        \
          "cudss error: ",                                              \
          at::cuda::cudss::cudssGetErrorMessage(__err),                 \
          ", when calling `" #EXPR "`",                                 \
          ". This error may appear if the input matrix contains NaN. ");\
    } else {                                                            \
      TORCH_CHECK(                                                      \
          __err == CUDSS_STATUS_SUCCESS,                                \
          "cudss error: ",                                              \
          at::cuda::cudss::cudssGetErrorMessage(__err),                 \
```
- EN: Focus symbols: `cudssGetErrorMessage`, `TORCH_CHECK`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`cudssGetErrorMessage`, `TORCH_CHECK`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 102-113
```cpp
          ", when calling `" #EXPR "`. ");                              \
    }                                                                   \
  } while (0)
#else
#define TORCH_CUDSS_CHECK(EXPR) EXPR
#endif

namespace at::cuda::solver {
#if !defined(USE_ROCM)

C10_EXPORT const char* cusolverGetErrorMessage(cusolverStatus_t status);

```
- EN: Focus symbols: `TORCH_CUDSS_CHECK`, `at::cuda::solver`, `cusolverGetErrorMessage`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`TORCH_CUDSS_CHECK`, `at::cuda::solver`, `cusolverGetErrorMessage`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 114-123
```cpp
constexpr const char* _cusolver_backend_suggestion =            \
  "If you keep seeing this error, you may use "                 \
  "`torch.backends.cuda.preferred_linalg_library()` to try "    \
  "linear algebra operators with other supported backends. "    \
  "See https://pytorch.org/docs/stable/backends.html#torch.backends.cuda.preferred_linalg_library";

// When cuda >= 11.5, cusolver normally finishes execution and sets info array indicating convergence issue.
#define TORCH_CUSOLVER_CHECK(EXPR)                                      \
  do {                                                                  \
    cusolverStatus_t __err = EXPR;                                      \
```
- EN: Focus symbols: `TORCH_CUSOLVER_CHECK`, `preferred_linalg_library`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`TORCH_CUSOLVER_CHECK`, `preferred_linalg_library`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 124-133
```cpp
    if (__err == CUSOLVER_STATUS_INVALID_VALUE) {                       \
      TORCH_CHECK_LINALG(                                               \
          false,                                                        \
          "cusolver error: ",                                           \
          at::cuda::solver::cusolverGetErrorMessage(__err),             \
          ", when calling `" #EXPR "`",                                 \
          ". This error may appear if the input matrix contains NaN. ", \
          at::cuda::solver::_cusolver_backend_suggestion);              \
    } else {                                                            \
      TORCH_CHECK(                                                      \
```
- EN: Focus symbols: `TORCH_CHECK_LINALG`, `cusolverGetErrorMessage`, `TORCH_CHECK`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`TORCH_CHECK_LINALG`, `cusolverGetErrorMessage`, `TORCH_CHECK`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 134-145
```cpp
          __err == CUSOLVER_STATUS_SUCCESS,                             \
          "cusolver error: ",                                           \
          at::cuda::solver::cusolverGetErrorMessage(__err),             \
          ", when calling `" #EXPR "`. ",                               \
          at::cuda::solver::_cusolver_backend_suggestion);              \
    }                                                                   \
  } while (0)

#else // defined(USE_ROCM)

C10_EXPORT const char* hipsolverGetErrorMessage(hipsolverStatus_t status);

```
- EN: Focus symbols: `cusolverGetErrorMessage`, `hipsolverGetErrorMessage`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cusolverGetErrorMessage`, `hipsolverGetErrorMessage`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 146-155
```cpp
constexpr const char* _hipsolver_backend_suggestion =           \
  "If you keep seeing this error, you may use "                 \
  "`torch.backends.cuda.preferred_linalg_library()` to try "    \
  "linear algebra operators with other supported backends. "    \
  "See https://pytorch.org/docs/stable/backends.html#torch.backends.cuda.preferred_linalg_library";

#define TORCH_CUSOLVER_CHECK(EXPR)                                      \
  do {                                                                  \
    hipsolverStatus_t __err = EXPR;                                     \
    if (__err == HIPSOLVER_STATUS_INVALID_VALUE) {                      \
```
- EN: Focus symbols: `TORCH_CUSOLVER_CHECK`, `preferred_linalg_library`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`TORCH_CUSOLVER_CHECK`, `preferred_linalg_library`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 156-165
```cpp
      TORCH_CHECK_LINALG(                                               \
          false,                                                        \
          "hipsolver error: ",                                          \
          at::cuda::solver::hipsolverGetErrorMessage(__err),            \
          ", when calling `" #EXPR "`",                                 \
          ". This error may appear if the input matrix contains NaN. ", \
          at::cuda::solver::_hipsolver_backend_suggestion);             \
    } else {                                                            \
      TORCH_CHECK(                                                      \
          __err == HIPSOLVER_STATUS_SUCCESS,                            \
```
- EN: Focus symbols: `TORCH_CHECK_LINALG`, `hipsolverGetErrorMessage`, `TORCH_CHECK`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`TORCH_CHECK_LINALG`, `hipsolverGetErrorMessage`, `TORCH_CHECK`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 166-176
```cpp
          "hipsolver error: ",                                          \
          at::cuda::solver::hipsolverGetErrorMessage(__err),            \
          ", when calling `" #EXPR "`. ",                               \
          at::cuda::solver::_hipsolver_backend_suggestion);             \
    }                                                                   \
  } while (0)
#endif
} // namespace at::cuda::solver

#define AT_CUDA_CHECK(EXPR) C10_CUDA_CHECK(EXPR)

```
- EN: Focus symbols: `AT_CUDA_CHECK`, `at::cuda::solver`, `hipsolverGetErrorMessage`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`AT_CUDA_CHECK`, `at::cuda::solver`, `hipsolverGetErrorMessage`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 177-186
```cpp
// For CUDA Driver API
//
// This is here instead of in c10 because NVRTC is loaded dynamically via a stub
// in ATen, and we need to use its nvrtcGetErrorString.
// See NOTE [ USE OF NVRTC AND DRIVER API ].
#if !defined(USE_ROCM)

#define AT_CUDA_DRIVER_CHECK(EXPR)                                          \
  do {                                                                      \
    CUresult __err = EXPR;                                                  \
```
- EN: Focus symbols: `AT_CUDA_DRIVER_CHECK`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`AT_CUDA_DRIVER_CHECK`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 187-198
```cpp
    if (__err != CUDA_SUCCESS) {                                            \
      const char* err_str;                                                  \
      [[maybe_unused]] CUresult get_error_str_err =                         \
          at::globalContext().getNVRTC().cuGetErrorString(__err, &err_str); \
      if (get_error_str_err != CUDA_SUCCESS) {                              \
        TORCH_CHECK(false, "CUDA driver error: unknown error");             \
      } else {                                                              \
        TORCH_CHECK(false, "CUDA driver error: ", err_str);                 \
      }                                                                     \
    }                                                                       \
  } while (0)

```
- EN: Focus symbols: `globalContext`, `getNVRTC`, `cuGetErrorString`, `TORCH_CHECK`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`globalContext`, `getNVRTC`, `cuGetErrorString`, `TORCH_CHECK`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 199-210
```cpp
#else

#define AT_CUDA_DRIVER_CHECK(EXPR)                                                \
  do {                                                                            \
    CUresult __err = EXPR;                                                        \
    if (__err != CUDA_SUCCESS) {                                                  \
      TORCH_CHECK(false, "CUDA driver error: ", static_cast<int>(__err));                   \
    }                                                                             \
  } while (0)

#endif

```
- EN: Focus symbols: `AT_CUDA_DRIVER_CHECK`, `TORCH_CHECK`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`AT_CUDA_DRIVER_CHECK`, `TORCH_CHECK`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

### Lines 211-220
```cpp
// For CUDA NVRTC
//
// Note: As of CUDA 10, nvrtc error code 7, NVRTC_ERROR_BUILTIN_OPERATION_FAILURE,
// incorrectly produces the error string "NVRTC unknown error."
// The following maps it correctly.
//
// This is here instead of in c10 because NVRTC is loaded dynamically via a stub
// in ATen, and we need to use its nvrtcGetErrorString.
// See NOTE [ USE OF NVRTC AND DRIVER API ].
#define AT_CUDA_NVRTC_CHECK(EXPR)                                                                   \
```
- EN: Focus symbols: `AT_CUDA_NVRTC_CHECK`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`AT_CUDA_NVRTC_CHECK`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 221-230
```cpp
  do {                                                                                              \
    nvrtcResult __err = EXPR;                                                                       \
    if (__err != NVRTC_SUCCESS) {                                                                   \
      if (static_cast<int>(__err) != 7) {                                                           \
        TORCH_CHECK(false, "CUDA NVRTC error: ", at::globalContext().getNVRTC().nvrtcGetErrorString(__err));  \
      } else {                                                                                      \
        TORCH_CHECK(false, "CUDA NVRTC error: NVRTC_ERROR_BUILTIN_OPERATION_FAILURE");                        \
      }                                                                                             \
    }                                                                                               \
  } while (0)
```
- EN: Focus symbols: `TORCH_CHECK`, `globalContext`, `getNVRTC`, `nvrtcGetErrorString`. This block executes runtime branching, validation, or state updates. This is where the file reacts to dynamic conditions instead of only describing static structure.
- CN: 关注符号：`TORCH_CHECK`, `globalContext`, `getNVRTC`, `nvrtcGetErrorString`。该代码块执行运行时分支、校验或状态更新。这里体现了文件对动态条件的响应，而不只是静态结构描述。

## Key Concepts / 关键概念
- CUDA backend support / CUDA 后端支持
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `c10/macros/Export.h`, `ATen/Context.h`, `c10/util/Exception.h`, `c10/cuda/CUDAException.h`
- External/system includes / 外部或系统头: `cublas_v2.h`, `cusparse.h`, `cusolver_common.h`, `hipsolver/hipsolver.h`, `cudss.h`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/cuda/Exceptions.cpp`
- Inferred semantic dependencies / 推断出的语义依赖: runtime validation / 运行时校验; namespace scoping / 命名空间作用域
