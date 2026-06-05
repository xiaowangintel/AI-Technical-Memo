# common.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cutlass_extensions/common.hpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Wraps or specializes CUTLASS/CUTE building blocks for GPU kernels. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 为 GPU 内核封装或特化 CUTLASS/CUTE 构件。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Types and data layout
```cpp
#pragma once

#include "cuda_runtime.h"
#include "cutlass/cutlass.h"

/**
 * A wrapper for a kernel that is used to guard against compilation on
 * architectures that will never use the kernel. The purpose of this is to
 * reduce the size of the compiled binary.
 * __CUDA_ARCH__ is not defined in host code, so this lets us smuggle the ifdef
 * into code that will be executed on the device where it is defined.
 */
template <typename Kernel>
struct enable_sm90_or_later : Kernel {
  template <typename... Args>
  CUTLASS_DEVICE void operator()(Args&&... args) {
#if defined __CUDA_ARCH__ && __CUDA_ARCH__ >= 900
    Kernel::operator()(std::forward<Args>(args)...);
#endif
  }
```
**EN:** This section defines `enable_sm90_or_later`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`enable_sm90_or_later`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 21-21: Local implementation details
```cpp
};
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cuda_runtime.h`, `cutlass/cutlass.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cutlass_extensions / common.hpp
