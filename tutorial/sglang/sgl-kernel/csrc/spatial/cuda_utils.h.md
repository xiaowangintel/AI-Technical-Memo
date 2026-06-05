# cuda_utils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/spatial/cuda_utils.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Runtime integration and dispatch
```cpp
#include <cuda.h>
#include <cuda_runtime.h>

#include <iostream>

#define CUDA_RT(call)                                                                                        \
  do {                                                                                                       \
    cudaError_t _status = (call);                                                                            \
    if (_status != cudaSuccess) {                                                                            \
      std::cerr << "ERROR: CUDA RT call \"" << #call << "\" in line " << __LINE__ << " of file " << __FILE__ \
                << " failed with " << cudaGetErrorString(_status) << std::endl;                              \
      TORCH_CHECK(                                                                                           \
          false,                                                                                             \
          c10::str(                                                                                          \
              "ERROR: CUDA RT call \"",                                                                      \
              #call,                                                                                         \
              "\" in line ",                                                                                 \
              __LINE__,                                                                                      \
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 19-36: Runtime integration and dispatch
```cpp
              " of file ",                                                                                   \
              __FILE__,                                                                                      \
              " failed with ",                                                                               \
              cudaGetErrorString(_status)));                                                                 \
    }                                                                                                        \
  } while (0)

#define CUDA_DRV(call)                                                                                        \
  do {                                                                                                        \
    CUresult _status = (call);                                                                                \
    if (_status != CUDA_SUCCESS) {                                                                            \
      const char* err_str;                                                                                    \
      cuGetErrorString(_status, &err_str);                                                                    \
      std::cerr << "ERROR: CUDA DRV call \"" << #call << "\" in line " << __LINE__ << " of file " << __FILE__ \
                << " failed with " << err_str << std::endl;                                                   \
      TORCH_CHECK(                                                                                            \
          false,                                                                                              \
          c10::str(                                                                                           \
```
**EN:** This section uses `cudaGetErrorString`, `cuGetErrorString`, `err_str` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cudaGetErrorString`、`cuGetErrorString`、`err_str`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 37-46: Control flow and branching
```cpp
              "ERROR: CUDA DRV call \"",                                                                      \
              #call,                                                                                          \
              "\" in line ",                                                                                  \
              __LINE__,                                                                                       \
              " of file ",                                                                                    \
              __FILE__,                                                                                       \
              " failed with ",                                                                                \
              err_str));                                                                                      \
    }                                                                                                         \
  } while (0)
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

## Key Concepts / 关键概念
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `cuda.h`, `cuda_runtime.h`, `iostream`
- **Path context / 路径上下文**: spatial / cuda_utils.h
