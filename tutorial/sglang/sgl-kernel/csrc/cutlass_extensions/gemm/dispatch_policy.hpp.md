# dispatch_policy.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cutlass_extensions/gemm/dispatch_policy.hpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Headers and compile-time setup
```cpp
// Adapted from https://github.com/vllm-project/vllm/blob/main/csrc/cutlass_extensions/gemm/dispatch_policy.hpp

#pragma once

#include "cutlass/gemm/dispatch_policy.hpp"

namespace cutlass::gemm {

//////////////////////////////////////////////////////////////////////////////

// FP8 related policies (including Blocked Scaled Accumulation)
//  `ScaleGranularityM` specifies scaling granularity along M, while zero-value
//  `ScaleGranularityM` indicates that scaling granularity is
//  `size<0>(TileShape_MNK{})` along M.
template <int ScaleGranularityM = 0>
struct KernelTmaWarpSpecializedCooperativeFP8BlockScaledSubGroupMAccum : KernelTmaWarpSpecializedCooperative {};
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 18-35: Types and data layout
```cpp
// n-buffer in smem (Hopper TMA), pipelined with Hopper GMMA and TMA, Warp
// specialized dynamic schedule For FP8 kernels with Block Scaling
template <
    int Stages_,
    class ClusterShape_ = Shape<_1, _1, _1>,
    class KernelSchedule = KernelTmaWarpSpecialized,
    int ScaleGranularityM = 0  // `ScaleGranularityM` specifies scaling granularity along M,
                               // while zero-value `ScaleGranularityM` indicates that scaling
                               // granularity is `size<0>(TileShape_MNK{})` along M.
    >
struct MainloopSm90TmaGmmaWarpSpecializedBlockScalingSubGroupMFP8
    : MainloopSm90TmaGmmaWarpSpecialized<Stages_, ClusterShape_, KernelSchedule> {
  static_assert(
      cute::
          is_same_v<KernelSchedule, KernelTmaWarpSpecializedCooperativeFP8BlockScaledSubGroupMAccum<ScaleGranularityM>>,
      "KernelSchedule must be one of the warp specialized policies");
};
```
**EN:** This section defines `ClusterShape_`, `KernelSchedule`, `MainloopSm90TmaGmmaWarpSpecializedBlockScalingSubGroupMFP8`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`ClusterShape_`、`KernelSchedule`、`MainloopSm90TmaGmmaWarpSpecializedBlockScalingSubGroupMFP8`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 36-37: Local implementation details
```cpp
//////////////////////////////////////////////////////////////////////////////
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 38-38: Local implementation details
```cpp
}  // namespace cutlass::gemm
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass/gemm/dispatch_policy.hpp`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cutlass_extensions / gemm / dispatch_policy.hpp
