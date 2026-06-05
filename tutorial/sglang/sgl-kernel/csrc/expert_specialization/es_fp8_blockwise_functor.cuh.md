# es_fp8_blockwise_functor.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/expert_specialization/es_fp8_blockwise_functor.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Types and data layout
```cpp
#pragma once
#include <cuda.h>

#include <iostream>

#include "cute/tensor.hpp"
#include "es_fp8_blockwise_traits.cuh"

namespace expert_specialization {

using namespace cute;

template <typename ElementAB, typename ElementSF, typename ElementD>
struct Fp8BlockwiseGroupedGemmOffsetFunctor {
  // Input
  int* expert_offsets{nullptr};
  // Base pointers
  ElementAB* a_base{nullptr};
  ElementAB* b_base{nullptr};
  ElementD* out_base{nullptr};
  ElementSF* a_scales_base{nullptr};
  ElementSF* b_scales_base{nullptr};
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmOffsetFunctor`, `cute`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmOffsetFunctor`、`cute`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 24-45: Local implementation details
```cpp
  // Output
  // Pointer Array for A/B
  ElementAB** a_offsets{nullptr};
  ElementAB** b_offsets{nullptr};
  ElementSF** a_scales_offsets{nullptr};
  ElementSF** b_scales_offsets{nullptr};
  ElementD** out_offsets{nullptr};

  Fp8BlockwiseGroupedGemmOffsetFunctor() = default;
  Fp8BlockwiseGroupedGemmOffsetFunctor(
      int* _expert_offsets,
      ElementAB* _a_base,
      ElementAB* _b_base,
      ElementD* _out_base,
      ElementSF* _a_scales_base,
      ElementSF* _b_scales_base,
      ElementAB** _a_offsets,
      ElementAB** _b_offsets,
      ElementSF** _a_scales_offsets,
      ElementSF** _b_scales_offsets,
      ElementD** _out_offsets)
      : expert_offsets(_expert_offsets),
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 46-68: Local implementation details
```cpp
        a_base(_a_base),
        b_base(_b_base),
        out_base(_out_base),
        a_scales_base(_a_scales_base),
        b_scales_base(_b_scales_base),
        a_offsets(_a_offsets),
        b_offsets(_b_offsets),
        a_scales_offsets(_a_scales_offsets),
        b_scales_offsets(_b_scales_offsets),
        out_offsets(_out_offsets) {}

  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    int64_t expert_offset = static_cast<int64_t>(expert_offsets[expert_id]);
    int64_t a_stride = 0;
    int64_t b_stride = 0;
    int64_t a_scale_stride = 0;
    int64_t b_scale_stride = 0;

    a_stride = expert_offset * k;
    b_stride = expert_id * k * n;
    a_scale_stride = expert_offset * k / 128;
    b_scale_stride = expert_id * k * n / 128 / 128;
```
**EN:** This section fills in the local implementation details around `a_base`, completing the behavior required by the file.
**CN:** 本段补充了`a_base`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 69-88: Types and data layout
```cpp
    a_offsets[expert_id] = a_base + a_stride;
    b_offsets[expert_id] = b_base + b_stride;
    a_scales_offsets[expert_id] = a_scales_base + a_scale_stride;
    b_scales_offsets[expert_id] = b_scales_base + b_scale_stride;
    out_offsets[expert_id] = out_base + expert_offset * n;
  }
};

template <typename PerfConfig>
struct Fp8BlockwiseGroupedGemmSFLayoutFunctor {
  using ScaleConfig = typename PerfConfig::ScaleConfig;
  using LayoutSFA = typename PerfConfig::LayoutSFA;
  using LayoutSFB = typename PerfConfig::LayoutSFB;
  LayoutSFA* layout_sfa_base{nullptr};
  LayoutSFB* layout_sfb_base{nullptr};

  Fp8BlockwiseGroupedGemmSFLayoutFunctor() = default;
  Fp8BlockwiseGroupedGemmSFLayoutFunctor(LayoutSFA* _layout_sfa_base, LayoutSFB* _layout_sfb_base)
      : layout_sfa_base(_layout_sfa_base), layout_sfb_base(_layout_sfb_base) {}
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmSFLayoutFunctor`, `ScaleConfig`, `LayoutSFA`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmSFLayoutFunctor`、`ScaleConfig`、`LayoutSFA`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 89-109: Types and data layout
```cpp
  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    LayoutSFA* layout_sfa_ptr = layout_sfa_base + expert_id;
    LayoutSFB* layout_sfb_ptr = layout_sfb_base + expert_id;
    *layout_sfa_ptr = ScaleConfig::tile_atom_to_shape_SFA(cute::make_shape(m, n, k, 1));
    *layout_sfb_ptr = ScaleConfig::tile_atom_to_shape_SFB(cute::make_shape(m, n, k, 1));
  }
};

// [Unused]: Specialization for Swap A/B
template <>
struct Fp8BlockwiseGroupedGemmSFLayoutFunctor<PerfConfigLowMH20> {
  using ScaleConfig = typename PerfConfigLowMH20::ScaleConfig;
  using LayoutSFA = typename PerfConfigLowMH20::LayoutSFA;
  using LayoutSFB = typename PerfConfigLowMH20::LayoutSFB;
  LayoutSFA* layout_sfa_base{nullptr};
  LayoutSFB* layout_sfb_base{nullptr};

  Fp8BlockwiseGroupedGemmSFLayoutFunctor() = default;
  Fp8BlockwiseGroupedGemmSFLayoutFunctor(LayoutSFA* _layout_sfa_base, LayoutSFB* _layout_sfb_base)
      : layout_sfa_base(_layout_sfa_base), layout_sfb_base(_layout_sfb_base) {}
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmSFLayoutFunctor`, `tile_atom_to_shape_SFA`, `tile_atom_to_shape_SFB`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmSFLayoutFunctor`、`tile_atom_to_shape_SFA`、`tile_atom_to_shape_SFB`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 110-133: Types and data layout
```cpp
  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    LayoutSFA* layout_sfa_ptr = layout_sfa_base + expert_id;
    LayoutSFB* layout_sfb_ptr = layout_sfb_base + expert_id;
    *layout_sfa_ptr = ScaleConfig::tile_atom_to_shape_SFA(cute::make_shape(n, m, k, 1));
    *layout_sfb_ptr = ScaleConfig::tile_atom_to_shape_SFB(cute::make_shape(n, m, k, 1));
  }
};

template <typename PerfConfig>
struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor;

template <>
struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigLowMH20> {
  int* problem_sizes{nullptr};

  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor() = default;
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor(int* _problem_sizes) : problem_sizes(_problem_sizes) {}

  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    float m_f = __int2float_rn(m);
    float n_f = __int2float_rn(n);
    float k_f = __int2float_rn(k);
    float arithmetic_intensity = 2.0f * m_f * n_f * k_f / (m_f * k_f + k_f * n_f + 2.0f * m_f * n_f);
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`, `tile_atom_to_shape_SFA`, `tile_atom_to_shape_SFB`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`、`tile_atom_to_shape_SFA`、`tile_atom_to_shape_SFB`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 134-153: Types and data layout
```cpp
    if (m <= 32 || arithmetic_intensity < 70.0f) {
      // Swap A/B
      problem_sizes[expert_id * 3 + 0] = n;
      problem_sizes[expert_id * 3 + 1] = m;
      problem_sizes[expert_id * 3 + 2] = k;
    } else {
      problem_sizes[expert_id * 3 + 0] = 0;
      problem_sizes[expert_id * 3 + 1] = 0;
      problem_sizes[expert_id * 3 + 2] = 0;
    }
  }
};

template <>
struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigLowMHx00> {
  int* problem_sizes{nullptr};

  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor() = default;
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor(int* _problem_sizes) : problem_sizes(_problem_sizes) {}
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 154-174: Types and data layout
```cpp
  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    if (m <= 32) {
      // Swap A/B
      problem_sizes[expert_id * 3 + 0] = n;
      problem_sizes[expert_id * 3 + 1] = m;
      problem_sizes[expert_id * 3 + 2] = k;
    } else {
      problem_sizes[expert_id * 3 + 0] = 0;
      problem_sizes[expert_id * 3 + 1] = 0;
      problem_sizes[expert_id * 3 + 2] = 0;
    }
  }
};

template <>
struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigMiddleMH20> {
  int* problem_sizes{nullptr};

  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor() = default;
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor(int* _problem_sizes) : problem_sizes(_problem_sizes) {}
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 175-196: Types and data layout
```cpp
  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    float m_f = __int2float_rn(m);
    float n_f = __int2float_rn(n);
    float k_f = __int2float_rn(k);
    float arithmetic_intensity = 2.0f * m_f * n_f * k_f / (m_f * k_f + k_f * n_f + 2.0f * m_f * n_f);

    if ((!(m <= 32 || arithmetic_intensity < 70.0f)) && m <= 64) {
      problem_sizes[expert_id * 3 + 0] = m;
      problem_sizes[expert_id * 3 + 1] = n;
      problem_sizes[expert_id * 3 + 2] = k;
    } else {
      problem_sizes[expert_id * 3 + 0] = 0;
      problem_sizes[expert_id * 3 + 1] = 0;
      problem_sizes[expert_id * 3 + 2] = 0;
    }
  }
};

template <>
struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigMiddleMHx00> {
  int* problem_sizes{nullptr};
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`, `__int2float_rn`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`、`__int2float_rn`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 197-219: Types and data layout
```cpp
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor() = default;
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor(int* _problem_sizes) : problem_sizes(_problem_sizes) {}

  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    if (m > 32 && m <= 64) {
      problem_sizes[expert_id * 3 + 0] = n;
      problem_sizes[expert_id * 3 + 1] = m;
      problem_sizes[expert_id * 3 + 2] = k;
    } else {
      problem_sizes[expert_id * 3 + 0] = 0;
      problem_sizes[expert_id * 3 + 1] = 0;
      problem_sizes[expert_id * 3 + 2] = 0;
    }
  }
};

template <>
struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigHighMH20> {
  int* problem_sizes{nullptr};

  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor() = default;
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor(int* _problem_sizes) : problem_sizes(_problem_sizes) {}
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 220-241: Types and data layout
```cpp
  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    float m_f = __int2float_rn(m);
    float n_f = __int2float_rn(n);
    float k_f = __int2float_rn(k);
    float arithmetic_intensity = 2.0f * m_f * n_f * k_f / (m_f * k_f + k_f * n_f + 2.0f * m_f * n_f);

    if ((!(m <= 32 || arithmetic_intensity < 70.0f)) && m > 64) {
      problem_sizes[expert_id * 3 + 0] = m;
      problem_sizes[expert_id * 3 + 1] = n;
      problem_sizes[expert_id * 3 + 2] = k;
    } else {
      problem_sizes[expert_id * 3 + 0] = 0;
      problem_sizes[expert_id * 3 + 1] = 0;
      problem_sizes[expert_id * 3 + 2] = 0;
    }
  }
};

template <>
struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigHighMHx00> {
  int* problem_sizes{nullptr};
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`, `__int2float_rn`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`、`__int2float_rn`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 242-257: Control flow and branching
```cpp
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor() = default;
  Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor(int* _problem_sizes) : problem_sizes(_problem_sizes) {}

  void CUTE_DEVICE operator()(int64_t expert_id, int m, int n, int k) {
    if (m > 64) {
      problem_sizes[expert_id * 3 + 0] = m;
      problem_sizes[expert_id * 3 + 1] = n;
      problem_sizes[expert_id * 3 + 2] = k;
    } else {
      problem_sizes[expert_id * 3 + 0] = 0;
      problem_sizes[expert_id * 3 + 1] = 0;
      problem_sizes[expert_id * 3 + 2] = 0;
    }
  }
};
```
**EN:** This section drives `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 258-281: Kernel implementation
```cpp
template <
    typename OffsetFunctor,
    typename ScaleLayoutFunctor,
    typename LowMProblemSizeFilterFunctor,
    typename MiddleMProblemSizeFilterFunctor,
    typename HighMProblemSizeFilterFunctor>
__global__ void groupedGemmPreComputeKernel(
    int* problem_sizes,
    OffsetFunctor offset_functor,
    ScaleLayoutFunctor sf_functor,
    LowMProblemSizeFilterFunctor lm_psf_functor,
    MiddleMProblemSizeFilterFunctor mm_psf_functor,
    HighMProblemSizeFilterFunctor hm_psf_functor) {
  int64_t expert_id = static_cast<int64_t>(threadIdx.x);
  int m = problem_sizes[expert_id * 3 + 0];
  int n = problem_sizes[expert_id * 3 + 1];
  int k = problem_sizes[expert_id * 3 + 2];

  offset_functor(expert_id, m, n, k);
  sf_functor(expert_id, m, n, k);
  lm_psf_functor(expert_id, m, n, k);
  mm_psf_functor(expert_id, m, n, k);
  hm_psf_functor(expert_id, m, n, k);
}
```
**EN:** This section implements `groupedGemmPreComputeKernel`, `offset_functor`, `sf_functor`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`groupedGemmPreComputeKernel`、`offset_functor`、`sf_functor`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 282-282: Local implementation details
```cpp

```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 283-283: Local implementation details
```cpp
}  // namespace expert_specialization
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cute/tensor.hpp`, `es_fp8_blockwise_traits.cuh`
- **External headers / 外部头文件**: `cuda.h`, `iostream`
- **Path context / 路径上下文**: expert_specialization / es_fp8_blockwise_functor.cuh
