# es_sm100_mxfp8_blockscaled_functor.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/expert_specialization/es_sm100_mxfp8_blockscaled_functor.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Types and data layout
```cpp
#pragma once
#include <cuda.h>

#include "cute/tensor.hpp"
#include "cutlass/util/packed_stride.hpp"
#include "es_sm100_mxfp8_blockscaled_traits.cuh"

namespace expert_specialization {

using namespace cute;

template <typename GemmTraits>
struct Sm100Mxfp8BlockScaledOffsetFunctor {
  using Gemm = typename GemmTraits::Gemm;
  using ElementA = typename Gemm::ElementA;
  using ElementB = typename Gemm::ElementB;
  using ElementSF = typename GemmTraits::ElementSF;
  using ElementD = typename GemmTraits::ElementOutput;
  // Input
  int* expert_offsets{nullptr};
  int* blockscale_offsets{nullptr};
  // Output
```
**EN:** This section defines `Sm100Mxfp8BlockScaledOffsetFunctor`, `cute`, `Gemm`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Sm100Mxfp8BlockScaledOffsetFunctor`、`cute`、`Gemm`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 23-44: Local implementation details
```cpp
  ElementA* a_base{nullptr};
  ElementB* b_base{nullptr};
  ElementSF* sfa_base{nullptr};
  ElementSF* sfb_base{nullptr};
  ElementD* d_base{nullptr};
  ElementA** a_offsets{nullptr};
  ElementB** b_offsets{nullptr};
  ElementSF** sfa_offsets{nullptr};
  ElementSF** sfb_offsets{nullptr};
  ElementD** d_offsets{nullptr};

  Sm100Mxfp8BlockScaledOffsetFunctor() = default;
  Sm100Mxfp8BlockScaledOffsetFunctor(
      int* _expert_offsets,
      int* _blockscale_offsets,
      ElementA* _a_base,
      ElementB* _b_base,
      ElementSF* _sfa_base,
      ElementSF* _sfb_base,
      ElementD* _d_base,
      ElementA** _a_offsets,
      ElementB** _b_offsets,
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 45-69: Local implementation details
```cpp
      ElementSF** _sfa_offsets,
      ElementSF** _sfb_offsets,
      ElementD** _d_offsets)
      : expert_offsets{_expert_offsets},
        blockscale_offsets{_blockscale_offsets},
        a_base(_a_base),
        b_base(_b_base),
        sfa_base(_sfa_base),
        sfb_base(_sfb_base),
        d_base(_d_base),
        a_offsets(_a_offsets),
        b_offsets(_b_offsets),
        sfa_offsets(_sfa_offsets),
        sfb_offsets(_sfb_offsets),
        d_offsets(_d_offsets) {}

  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    int64_t expert_offset = static_cast<int64_t>(expert_offsets[expert_id]);
    int64_t blockscale_offset = static_cast<int64_t>(blockscale_offsets[expert_id]);
    int64_t a_stride = expert_offset * k;
    int64_t b_stride = expert_id * k * n;
    int64_t d_stride = expert_offset * n;
    int64_t sfa_stride = blockscale_offset * (k / 32);
    int64_t sfb_stride = expert_id * n * (k / 32);
```
**EN:** This section fills in the local implementation details around `a_base`, completing the behavior required by the file.
**CN:** 本段补充了`a_base`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 70-89: Types and data layout
```cpp
    a_offsets[expert_id] = a_base + a_stride;
    b_offsets[expert_id] = b_base + b_stride;
    sfa_offsets[expert_id] = sfa_base + sfa_stride;
    sfb_offsets[expert_id] = sfb_base + sfb_stride;
    d_offsets[expert_id] = d_base + d_stride;
  }
};

template <typename GemmTraits>
struct Sm100Mxfp8BlockScaledLayoutFunctor {
  using Sm1xxBlkScaledConfig = typename GemmTraits::Sm1xxBlkScaledConfig;
  using LayoutSFA = typename GemmTraits::LayoutSFA;
  using LayoutSFB = typename GemmTraits::LayoutSFB;
  LayoutSFA* layout_sfa_base{nullptr};
  LayoutSFB* layout_sfb_base{nullptr};

  Sm100Mxfp8BlockScaledLayoutFunctor() = default;
  Sm100Mxfp8BlockScaledLayoutFunctor(LayoutSFA* _layout_sfa_base, LayoutSFB* _layout_sfb_base)
      : layout_sfa_base(_layout_sfa_base), layout_sfb_base(_layout_sfb_base) {}
```
**EN:** This section defines `Sm100Mxfp8BlockScaledLayoutFunctor`, `Sm1xxBlkScaledConfig`, `LayoutSFA`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Sm100Mxfp8BlockScaledLayoutFunctor`、`Sm1xxBlkScaledConfig`、`LayoutSFA`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 90-110: Types and data layout
```cpp
  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    LayoutSFA* layout_sfa_ptr = layout_sfa_base + expert_id;
    LayoutSFB* layout_sfb_ptr = layout_sfb_base + expert_id;
    *layout_sfa_ptr = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFA(cute::make_shape(m, n, k, 1));
    *layout_sfb_ptr = Sm1xxBlkScaledConfig::tile_atom_to_shape_SFB(cute::make_shape(m, n, k, 1));
  }
};

template <typename GemmTraits>
struct Sm100Mxfp8BlockScaledStrideFunctor {
  using StrideA = typename GemmTraits::StrideA;
  using StrideB = typename GemmTraits::StrideB;
  using StrideD = typename GemmTraits::StrideD;
  StrideA* stride_A_base{nullptr};
  StrideB* stride_B_base{nullptr};
  StrideD* stride_D_base{nullptr};

  Sm100Mxfp8BlockScaledStrideFunctor() = default;
  Sm100Mxfp8BlockScaledStrideFunctor(StrideA* _stride_A_base, StrideB* _stride_B_base, StrideD* _stride_D_base)
      : stride_A_base(_stride_A_base), stride_B_base(_stride_B_base), stride_D_base(_stride_D_base) {}
```
**EN:** This section defines `Sm100Mxfp8BlockScaledStrideFunctor`, `tile_atom_to_shape_SFA`, `tile_atom_to_shape_SFB`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Sm100Mxfp8BlockScaledStrideFunctor`、`tile_atom_to_shape_SFA`、`tile_atom_to_shape_SFB`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 111-132: Kernel implementation
```cpp
  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    StrideA* stride_A = stride_A_base + expert_id;
    StrideB* stride_B = stride_B_base + expert_id;
    StrideD* stride_D = stride_D_base + expert_id;
    *stride_A = cutlass::make_cute_packed_stride(StrideA{}, {m, k, 1});
    *stride_B = cutlass::make_cute_packed_stride(StrideB{}, {n, k, 1});
    *stride_D = cutlass::make_cute_packed_stride(StrideD{}, {m, n, 1});
  }
};

template <typename OffsetFunctor, typename LayoutFunctor, typename StrideFunctor>
__global__ void sm100Mxfp8BlockscaledGroupedGemmPreComputeKernel(
    int* problem_sizes, OffsetFunctor offset_functor, LayoutFunctor layout_functor, StrideFunctor stride_functor) {
  int64_t expert_id = static_cast<int64_t>(threadIdx.x);
  int m = problem_sizes[expert_id * 3 + 0];
  int n = problem_sizes[expert_id * 3 + 1];
  int k = problem_sizes[expert_id * 3 + 2];

  offset_functor(expert_id, m, n, k);
  layout_functor(expert_id, m, n, k);
  stride_functor(expert_id, m, n, k);
}
```
**EN:** This section implements `sm100Mxfp8BlockscaledGroupedGemmPreComputeKernel`, `offset_functor`, `layout_functor`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`sm100Mxfp8BlockscaledGroupedGemmPreComputeKernel`、`offset_functor`、`layout_functor`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 133-133: Local implementation details
```cpp

```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 134-134: Local implementation details
```cpp
}  // namespace expert_specialization
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cute/tensor.hpp`, `cutlass/util/packed_stride.hpp`, `es_sm100_mxfp8_blockscaled_traits.cuh`
- **External headers / 外部头文件**: `cuda.h`
- **Path context / 路径上下文**: expert_specialization / es_sm100_mxfp8_blockscaled_functor.cuh
