# 69_hopper_mixed_dtype_grouped_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/69_hopper_mixed_dtype_grouped_gemm/69_hopper_mixed_dtype_grouped_gemm.cu`  
**Purpose / 用途**: This file demonstrates Hopper grouped GEMM with mixed input types and optional per-group scaling. It assembles SM90 pointer-array/TMA collectives, allocates one heterogeneous problem list, materializes per-group pointers and strides, supports either shared scalar alpha/beta or per-group alpha/beta arrays, and validates the result by explicitly dequantizing B before running a reference GEMM. / 这个文件演示了 Hopper 上带混合输入类型和可选逐组 scale 的 grouped GEMM。它构建 SM90 的 pointer-array/TMA collective，分配一组异构问题列表，生成每组自己的指针与 stride，同时支持共享的标量 alpha/beta 或逐组 alpha/beta 数组，并通过先显式反量化 B 再运行参考 GEMM 的方式做结果校验。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-96)
```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

/*! \file
    \brief 
    Hopper Mixed-input Grouped GEMM example using CUTLASS 3 APIs for NVIDIA Hopper architecture. 
    See 55_hopper_mixed_dtype_gemm.cu for more details about Mixed-input GEMMs.

    Limitations:
      1) Only support row-wise scaling. Zero-points and block-wise scaling is currently not supported.

    To run this example:

      $ ./examples/69_hopper_mixed_dtype_grouped_gemm/69_hopper_mixed_dtype_grouped_gemm --m=2048 --n=2048 --k=2048 --mode=1 --groups=10

      The above example command makes all 10 groups to be sized at the given m, n, k sizes.
      Skipping any of the problem dimensions randomizes it across the different groups.
      Same applies for alpha and beta values that are randomized across the different groups.
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <numeric>
#include <typeinfo>
#include <float.h>

#include "cutlass/cutlass.h"

#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/thread/linear_combination.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/group_array_problem_shape.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gett.hpp"
#include "cutlass/util/mixed_dtype_utils.hpp"

#include "helper.h"
#include "grouped_mixed_dtype_utils.hpp"

using namespace cute;
using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int,int,int>>; // <M,N,K> per group
using MmaType = cutlass::bfloat16_t;
using QuantType = cutlass::float_e5m2_t;
#if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The prologue explains that this is a grouped mixed-input Hopper GEMM and lists its current limitations. The include set is broad because the file combines CUTLASS 3 kernel construction, tensor/stride helpers, reference code, host-side comparison utilities, and example-specific mixed-dtype helpers. `ProblemShape`, `MmaType`, and `QuantType` establish the core theme immediately: BF16 math with FP8-like quantized weights inside grouped execution.
**CN**: 前言说明这是一个 grouped mixed-input 的 Hopper GEMM，并列出当前限制。这里的 include 很全，因为文件同时涉及 CUTLASS 3 内核构造、张量/stride 辅助、参考实现、主机侧比较工具，以及示例自己的 mixed-dtype 辅助逻辑。`ProblemShape`、`MmaType` 和 `QuantType` 一开始就点明了核心主题：在 grouped 执行中用 BF16 做乘加，并处理类似 FP8 的量化权重。

### Block 2 (Lines 97-145)
```cpp
constexpr int TileShapeK = 128 * 8 / sizeof_bits<MmaType>::value;

// A matrix configuration
using         ElementA    = MmaType;
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = QuantType;                                      // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// This example manually swaps and transposes, so keep transpose of input layouts
using LayoutA_Transpose = typename cutlass::layout::LayoutTranspose<LayoutA>::type;
using LayoutB_Transpose = typename cutlass::layout::LayoutTranspose<LayoutB>::type;

using ElementZero = cutlass::bfloat16_t;
using ElementScale = cutlass::bfloat16_t;
using LayoutScale = cutlass::layout::RowMajor;

// C/D matrix configuration
using         ElementC    = cutlass::half_t;                                // Element type for C and D matrix operands
using         LayoutC     = cutlass::layout::RowMajor;                      // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// D matrix configuration
using         ElementD    = ElementC;
using         LayoutD     = LayoutC;
constexpr int AlignmentD  = 128 / cutlass::sizeof_bits<ElementD>::value;

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_128,_16,cute::Int<TileShapeK>>;                           // Threadblock-level tile size
using ClusterShape        = Shape<_1,_1,_1>;                                // Shape of the threadblocks in a cluster
using StageCountType = cutlass::gemm::collective::StageCountAuto;           // Stage count maximized based on the tile size
using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperative;
using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative; // Epilogue to launch

using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
    TileShape, ClusterShape,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementAccumulator,
    ElementC, typename cutlass::layout::LayoutTranspose<LayoutC>::type *, AlignmentC,
    ElementD, typename cutlass::layout::LayoutTranspose<LayoutD>::type *, AlignmentD,
    EpilogueSchedule
  >::CollectiveOp;
```
**EN**: This block defines the core SM90 kernel shape. `TileShape`, `ClusterShape`, `KernelSchedule`, and `EpilogueSchedule` are the key CUTLASS template parameters: they determine threadblock tiling, cluster topology, and that the kernel uses the Hopper pointer-array TMA cooperative pipeline. The epilogue builder also transposes C/D layouts at the type level so the device kernel sees the memory organization expected by the chosen schedule.
**CN**: 这一段定义了核心 SM90 内核形状。`TileShape`、`ClusterShape`、`KernelSchedule` 与 `EpilogueSchedule` 是最关键的 CUTLASS 模板参数：它们决定 threadblock 分块、cluster 拓扑，以及内核采用 Hopper 的 pointer-array TMA cooperative 流水。epilogue builder 还在类型层面对 C/D 布局做了转置，使设备内核看到与所选调度匹配的内存组织。

### Block 3 (Lines 146-195)
```cpp

using CollectiveMainloopConvertOnly = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementB, LayoutB_Transpose *, AlignmentB,
    ElementA, LayoutA_Transpose *, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelConvertOnly = cutlass::gemm::kernel::GemmUniversal<
    ProblemShape,
    CollectiveMainloopConvertOnly,
    CollectiveEpilogue
>;

using GemmConvertOnly = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelConvertOnly>;

// =========================================================== MIXED INPUT WITH SCALES ===========================================================================
// The Scale information must get paired with the operand that will be scaled. In this example, B is scaled so we make a tuple of B's information and the scale information.
using CollectiveMainloopScaleOnly = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale>, LayoutB_Transpose *, AlignmentB,
    ElementA, LayoutA_Transpose *, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleOnly = cutlass::gemm::kernel::GemmUniversal<
    ProblemShape, 
    CollectiveMainloopScaleOnly,
    CollectiveEpilogue
>;

using GemmScaleOnly = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleOnly>;

using StrideA = typename GemmConvertOnly::GemmKernel::InternalStrideA;
using StrideB = typename GemmConvertOnly::GemmKernel::InternalStrideB;
using StrideC = typename GemmConvertOnly::GemmKernel::InternalStrideC;
using StrideD = typename GemmConvertOnly::GemmKernel::InternalStrideD;
using StrideC_ref = cutlass::detail::TagToStrideC_t<LayoutC>;
using StrideD_ref = cutlass::detail::TagToStrideC_t<LayoutD>;
using StrideS = typename CollectiveMainloopScaleOnly::StrideScale;
using StrideS_ref = cutlass::detail::TagToStrideB_t<LayoutScale>;

```
**EN**: Here the file declares two mainloop variants. `CollectiveMainloopConvertOnly` treats B as a quantized tensor that is only converted, while `CollectiveMainloopScaleOnly` binds B together with a scale tensor as `cute::tuple<ElementB, ElementScale>`, which is the CUTLASS 3 way to express fused mixed-input metadata. The stride aliases extracted at the end are later used to build per-group packed strides for both kernel and reference paths.
**CN**: 这里声明了两个 mainloop 变体。`CollectiveMainloopConvertOnly` 把 B 视为仅做类型转换的量化张量；`CollectiveMainloopScaleOnly` 则把 B 和 scale 张量绑定成 `cute::tuple<ElementB, ElementScale>`，这是 CUTLASS 3 表达“融合的混合输入元数据”的方式。结尾提取出的 stride 别名会在后续为 kernel 路径和 reference 路径分别构建逐组 packed stride。

### Block 4 (Lines 196-260)
```cpp
// Host-side allocations
std::vector<int64_t> offset_A;
std::vector<int64_t> offset_B;
std::vector<int64_t> offset_B_dq;
std::vector<int64_t> offset_C;
std::vector<int64_t> offset_D;
std::vector<int64_t> offset_scale;
std::vector<int64_t> offset_zero;

std::vector<StrideA> stride_A_host;
std::vector<StrideB> stride_B_host;
std::vector<StrideC> stride_C_host;
std::vector<StrideD> stride_D_host;
std::vector<StrideC_ref> stride_C_host_ref;
std::vector<StrideD_ref> stride_D_host_ref;
std::vector<StrideS> stride_S_host;
std::vector<StrideS_ref> stride_S_host_ref;

std::vector<ElementAccumulator> alpha_host;
std::vector<ElementAccumulator> beta_host;

uint64_t seed = 2020;

// Device-side allocations
cutlass::DeviceAllocation<typename ProblemShape::UnderlyingProblemShape> problem_sizes;

cutlass::DeviceAllocation<MmaType> block_A;
cutlass::DeviceAllocation<QuantType> block_B;
cutlass::DeviceAllocation<MmaType> block_B_dq;
cutlass::DeviceAllocation<ElementScale> block_scale;
cutlass::DeviceAllocation<ElementZero> block_zero;
cutlass::DeviceAllocation<ElementC> block_C;
cutlass::DeviceAllocation<typename GemmConvertOnly::EpilogueOutputOp::ElementOutput> block_D;
cutlass::DeviceAllocation<typename GemmConvertOnly::EpilogueOutputOp::ElementOutput> block_ref_D;

cutlass::DeviceAllocation<const MmaType *> ptr_A;
cutlass::DeviceAllocation<const QuantType *> ptr_B;
cutlass::DeviceAllocation<const MmaType *> ptr_B_dq;
cutlass::DeviceAllocation<const ElementScale *> ptr_scale;
cutlass::DeviceAllocation<const ElementZero *> ptr_zero;
cutlass::DeviceAllocation<const ElementC *> ptr_C;
cutlass::DeviceAllocation<typename GemmConvertOnly::EpilogueOutputOp::ElementOutput *> ptr_D;

cutlass::DeviceAllocation<StrideA> stride_A;
cutlass::DeviceAllocation<StrideB> stride_B;
cutlass::DeviceAllocation<StrideC> stride_C;
cutlass::DeviceAllocation<StrideD> stride_D;
cutlass::DeviceAllocation<StrideC_ref> stride_C_ref;
cutlass::DeviceAllocation<StrideD_ref> stride_D_ref;
cutlass::DeviceAllocation<StrideS_ref> stride_S_ref;
cutlass::DeviceAllocation<StrideS> stride_S;

// Note, this is an array of pointers to alpha and beta scaling values per group
cutlass::DeviceAllocation<ElementAccumulator*> alpha_device;
cutlass::DeviceAllocation<ElementAccumulator*> beta_device;
cutlass::DeviceAllocation<ElementAccumulator> block_alpha;
cutlass::DeviceAllocation<ElementAccumulator> block_beta;

#endif // defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

using Options = GroupedMixedDtypeOptions<QuantType>;
```
**EN**: This region declares the host-side offset tables, stride caches, alpha/beta vectors, and device allocations used across all groups. The important grouped-GEMM idea is visible here: instead of one problem shape and one tensor per operand, the program prepares arrays of problem shapes, arrays of pointers, arrays of strides, and optional arrays of scalar coefficients so the kernel can switch from one problem to the next inside a single launch.
**CN**: 这一段声明了跨所有 group 共享的主机侧 offset 表、stride 缓存、alpha/beta 向量以及设备分配对象。grouped GEMM 的关键思想在这里已经很清楚：程序不再只有一个问题形状和一套操作数，而是准备“问题形状数组、指针数组、stride 数组，以及可选的标量数组”，使内核能在一次 launch 中切换处理不同 problem。

### Block 5 (Lines 261-334)
```cpp

#if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM setup and evaluation
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Allocates device-side data
void allocate(Options const& options) {
  int64_t total_elements_A = 0;
  int64_t total_elements_B = 0;
  int64_t total_elements_B_dq = 0;
  int64_t total_elements_C = 0;
  int64_t total_elements_D = 0;
  int64_t total_elements_scale = 0;
  int64_t total_elements_zero = 0;

  for (int32_t i = 0; i < options.groups; ++i) {

    auto problem = options.problem_sizes_host.at(i);
    auto M = get<0>(problem);
    auto N = get<1>(problem);
    auto K = get<2>(problem);

    int const scale_k = cutlass::ceil_div(options.k, options.c);

    offset_A.push_back(total_elements_A);
    offset_B.push_back(total_elements_B * cutlass::sizeof_bits<QuantType>::value / 8);
    offset_B_dq.push_back(total_elements_B_dq);
    offset_C.push_back(total_elements_C);
    offset_D.push_back(total_elements_D);
    offset_scale.push_back(total_elements_scale);
    offset_zero.push_back(total_elements_zero);

    int64_t elements_A = M * K;
    int64_t elements_B = K * N ;
    int64_t elements_B_dq = K * N;
    int64_t elements_C = M * N;
    int64_t elements_D = M * N;
    int64_t elements_scale = scale_k * N;
    int64_t elements_zero = scale_k * N;

    total_elements_A += elements_A;
    total_elements_B += elements_B;
    total_elements_B_dq += elements_B_dq;
    total_elements_C += elements_C;
    total_elements_D += elements_D;
    total_elements_scale += elements_scale;
    total_elements_zero += elements_zero;

    stride_A_host.push_back(cutlass::make_cute_packed_stride(StrideA{}, {M, K, 1}));
    stride_B_host.push_back(cutlass::make_cute_packed_stride(StrideB{}, {N, K, 1}));
    stride_C_host.push_back(cutlass::make_cute_packed_stride(StrideC{}, {N, M, 1}));
    stride_D_host.push_back(cutlass::make_cute_packed_stride(StrideD{}, {N, M, 1}));
    stride_C_host_ref.push_back(cutlass::make_cute_packed_stride(StrideC_ref{}, {M, N, 1}));
    stride_D_host_ref.push_back(cutlass::make_cute_packed_stride(StrideD_ref{}, {M, N, 1}));
    stride_S_host_ref.push_back(cutlass::make_cute_packed_stride(StrideS_ref{}, {N, scale_k, 1}));
    stride_S_host.push_back(cutlass::make_cute_packed_stride(StrideS{}, {N, scale_k, 1}));
  }

  block_A.reset(total_elements_A);
  block_B.reset(total_elements_B);
  block_B_dq.reset(total_elements_B_dq);
  block_C.reset(total_elements_C);
  block_D.reset(total_elements_D);
  block_ref_D.reset(total_elements_D);
  block_scale.reset(total_elements_scale);
  block_zero.reset(total_elements_zero);

  block_alpha.reset(options.groups);
  block_beta.reset(options.groups);
}

/// Initialize operands to be used in the GEMM and reference GEMM
```
**EN**: `Options` is aliased to `GroupedMixedDtypeOptions`, then `allocate()` walks every group, computes the size of A/B/C/D/scale/zero storage, records offsets, and builds per-group packed strides. Note that `offset_B` converts element counts to byte offsets for the quantized type, while `scale_k = ceil_div(k, c)` reflects grouped scaling granularity along K chunks. This block is where the grouped problem list turns into concrete storage planning.
**CN**: `Options` 被别名为 `GroupedMixedDtypeOptions`，随后 `allocate()` 遍历每个 group，计算 A/B/C/D/scale/zero 所需的容量、记录偏移，并生成逐组 packed stride。需要注意的是：`offset_B` 会把元素数转换为量化类型的字节偏移，而 `scale_k = ceil_div(k, c)` 则体现了沿 K 分块的 grouped scaling 粒度。这里是“问题列表”落地为“具体存储规划”的地方。

### Block 6 (Lines 335-437)
```cpp
void initialize(Options &options) {

  uint64_t seed = 2020;

  problem_sizes.reset(options.groups);
  problem_sizes.copy_from_host(options.problem_sizes_host.data());

  //
  // Assign pointers
  //

  std::vector<MmaType *> ptr_A_host(options.groups);
  std::vector<QuantType *> ptr_B_host(options.groups);
  std::vector<MmaType *> ptr_B_dq_host(options.groups);
  std::vector<ElementC *> ptr_C_host(options.groups);
  std::vector<ElementC *> ptr_D_host(options.groups);
  std::vector<ElementScale *> ptr_scale_host(options.groups);
  std::vector<ElementZero *> ptr_zero_host(options.groups);
  std::vector<ElementAccumulator *> ptr_alpha_host(options.groups);
  std::vector<ElementAccumulator *> ptr_beta_host(options.groups);

  for (int32_t i = 0; i < options.groups; ++i) {
    ptr_A_host.at(i) = block_A.get() + offset_A.at(i);
    ptr_B_host.at(i) = block_B.get() + offset_B.at(i);
    ptr_B_dq_host.at(i) = block_B_dq.get() + offset_B_dq.at(i);
    ptr_C_host.at(i) = block_C.get() + offset_C.at(i);
    ptr_D_host.at(i) = block_D.get() + offset_D.at(i);
    ptr_scale_host.at(i) = block_scale.get() + offset_scale.at(i);
    ptr_zero_host.at(i) = block_zero.get() + offset_zero.at(i);
    alpha_host.push_back((options.alpha == FLT_MAX) ? static_cast<ElementAccumulator>((rand() % 5) + 1) : options.alpha);
    beta_host.push_back((options.beta == FLT_MAX) ? static_cast<ElementAccumulator>(rand() % 5) : options.beta);
    ptr_alpha_host.at(i) = block_alpha.get() + i;
    ptr_beta_host.at(i) = block_beta.get() + i;
  }

  ptr_A.reset(options.groups);
  ptr_A.copy_from_host(ptr_A_host.data());

  ptr_B.reset(options.groups);
  ptr_B.copy_from_host(ptr_B_host.data());

  ptr_B_dq.reset(options.groups);
  ptr_B_dq.copy_from_host(ptr_B_dq_host.data());

  ptr_C.reset(options.groups);
  ptr_C.copy_from_host(ptr_C_host.data());

  ptr_D.reset(options.groups);
  ptr_D.copy_from_host(ptr_D_host.data());

  ptr_scale.reset(options.groups);
  ptr_scale.copy_from_host(ptr_scale_host.data());

  ptr_zero.reset(options.groups);
  ptr_zero.copy_from_host(ptr_zero_host.data());

  stride_A.reset(options.groups);
  stride_A.copy_from_host(stride_A_host.data());

  stride_B.reset(options.groups);
  stride_B.copy_from_host(stride_B_host.data());

  stride_C.reset(options.groups);
  stride_C.copy_from_host(stride_C_host.data());

  stride_D.reset(options.groups);
  stride_D.copy_from_host(stride_D_host.data());

  stride_C_ref.reset(options.groups);
  stride_C_ref.copy_from_host(stride_C_host_ref.data());

  stride_D_ref.reset(options.groups);
  stride_D_ref.copy_from_host(stride_D_host_ref.data());

  stride_S_ref.reset(options.groups);
  stride_S_ref.copy_from_host(stride_S_host_ref.data());

  stride_S.reset(options.groups);
  stride_S.copy_from_host(stride_S_host.data());

  alpha_device.reset(options.groups);
  alpha_device.copy_from_host(ptr_alpha_host.data());
  beta_device.reset(options.groups);
  beta_device.copy_from_host(ptr_beta_host.data());

  initialize_tensor(block_A, seed + 2023);
  initialize_tensor(block_B, seed + 2022);
  initialize_tensor(block_C, seed + 2021);
  initialize_scale(block_scale, options);
  initialize_zero(block_zero, options);
  block_alpha.copy_from_host(alpha_host.data());
  block_beta.copy_from_host(beta_host.data());

  problem_sizes.reset(options.groups);
  // Reverse MN -> NM for SwapAB
  for (int32_t i = 0; i < options.groups; ++i) {
    auto [M, N, K] = options.problem_sizes_host[i];
    options.problem_sizes_host[i] = make_tuple(N, M, K);
  }
  problem_sizes.copy_from_host(options.problem_sizes_host.data());
}

/// Populates a Gemm::Arguments structure from the given commandline options
```
**EN**: `initialize()` copies problem sizes to device memory, creates host pointer arrays for all operands and metadata tensors, optionally randomizes per-group alpha/beta values, copies every pointer/stride array to device memory, and initializes the data blocks. The final `N,M,K -> swap to N,M,K` rewrite is subtle but important: because the kernel uses swapped/transposed operand conventions internally, the host problem list is rewritten to match the kernel’s expected ordering.
**CN**: `initialize()` 把问题大小拷到设备端，为所有操作数和元数据张量构造主机侧指针数组，可选地随机化逐组 alpha/beta，再把所有指针/stride 数组复制到设备内存，并初始化数据块。最后对问题形状做的重写非常关键：由于内核内部采用了交换/转置后的操作数约定，主机侧问题列表必须改写成内核期望的顺序。

### Block 7 (Lines 438-499)
```cpp
template <typename Gemm>
typename Gemm::Arguments args_from_options(Options const& options, bool host_problem_shapes_available = true)
{ 
  cutlass::KernelHardwareInfo hw_info;
  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  typename Gemm::Arguments arguments;
  decltype(arguments.epilogue.thread) fusion_args;

  if (options.alpha != FLT_MAX && options.beta != FLT_MAX) {
    // If both alpha/beta are provided (via cmd line args) and are scalar, i.e., same alpha/beta applies to all batches.
    fusion_args.alpha = options.alpha;
    fusion_args.beta = options.beta;
    fusion_args.alpha_ptr = nullptr;
    fusion_args.beta_ptr = nullptr;
    fusion_args.alpha_ptr_array = nullptr;
    fusion_args.beta_ptr_array = nullptr;
    // Single alpha and beta for all groups
    fusion_args.dAlpha = {cute::_0{}, cute::_0{}, 0};
    fusion_args.dBeta = {cute::_0{}, cute::_0{}, 0};
  }
  else {
    // If pointers to alpha/beta are provided, i.e., alpha/beta can differ between batches/groups.
    fusion_args.alpha = 0;
    fusion_args.beta = 0;
    fusion_args.alpha_ptr = nullptr;
    fusion_args.beta_ptr = nullptr;
    fusion_args.alpha_ptr_array = alpha_device.get();
    fusion_args.beta_ptr_array = beta_device.get();
    // One alpha and beta per each group
    fusion_args.dAlpha = {cute::_0{}, cute::_0{}, 1};
    fusion_args.dBeta = {cute::_0{}, cute::_0{}, 1};
  }

  if constexpr (Gemm::CollectiveMainloop::KernelConversionMode == Gemm::CollectiveMainloop::ConversionMode::DirectConvert) {
    arguments = typename Gemm::Arguments {
      cutlass::gemm::GemmUniversalMode::kGrouped,
      {options.groups, problem_sizes.get(), nullptr},
      {ptr_B.get(), stride_B.get(), ptr_A.get(), stride_A.get()},
      {fusion_args, ptr_C.get(), stride_C.get(), ptr_D.get(), stride_D.get()},
      hw_info
    };
  }
  else if constexpr (Gemm::CollectiveMainloop::KernelConversionMode == Gemm::CollectiveMainloop::ConversionMode::ConvertAndScale) {
    arguments = typename Gemm::Arguments {
      cutlass::gemm::GemmUniversalMode::kGrouped,
      {options.groups, problem_sizes.get(), nullptr},
      {ptr_B.get(), stride_B.get(), ptr_A.get(), stride_A.get(), ptr_scale.get(), stride_S.get(), options.c},
      {fusion_args, ptr_C.get(), stride_C.get(), ptr_D.get(), stride_D.get()},
      hw_info
    };
  } 
  else {
    std::cerr << "Invalid mode " << options.mode << ". Must be 0, 1 or 2." << std::endl;
    exit(-1);
  }
  return arguments;
}

```
**EN**: `args_from_options()` builds the actual grouped-kernel arguments. The epilogue fusion arguments either carry one scalar alpha/beta for all groups or arrays of alpha/beta pointers with `dAlpha`/`dBeta` strides describing per-group stepping. The mainloop arguments then branch on `KernelConversionMode`: direct convert uses only A/B, while convert-and-scale also passes `ptr_scale`, `stride_S`, and the chunk size `options.c`.
**CN**: `args_from_options()` 负责真正构造 grouped 内核参数。epilogue 的 fusion 参数既可以是“所有组共享一个 alpha/beta 标量”，也可以是“每组一个 alpha/beta 指针数组”，而 `dAlpha`/`dBeta` 则描述按 group 递进的步长。随后 mainloop 参数根据 `KernelConversionMode` 分支：直接转换模式只传 A/B；转换加缩放模式还会额外传 `ptr_scale`、`stride_S` 和分块大小 `options.c`。

### Block 8 (Lines 500-592)
```cpp
bool verify(Options const& options) {
  bool passed = true;

  constexpr bool IsFP8Input = cute::is_same_v<MmaType, cutlass::float_e4m3_t> || cute::is_same_v<MmaType, cutlass::float_e5m2_t>;
  using FP8Sched = cute::conditional_t<size<0>(TileShape{}) == 64, cutlass::gemm::KernelTmaWarpSpecializedPingpongFP8FastAccum, cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8FastAccum>;
  using ScheduleRef = cute::conditional_t<IsFP8Input, FP8Sched, cutlass::gemm::collective::KernelScheduleAuto>;


  using CollectiveMainloopRef = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    MmaType, LayoutA, AlignmentA,
    MmaType, LayoutB, AlignmentB,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAuto,
    ScheduleRef
  >::CollectiveOp;

  using CollectiveEpilogueRef = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
    TileShape, ClusterShape,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementAccumulator,
    ElementC, LayoutC, AlignmentC,
    ElementD, LayoutD, AlignmentD,
    cutlass::epilogue::NoSmemWarpSpecialized
  >::CollectiveOp;

  using GemmKernelRef = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int>, // Indicates ProblemShape
    CollectiveMainloopRef,
    CollectiveEpilogueRef
  >;

  using GemmRef = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelRef>;
  using StrideA_verif = typename GemmRef::GemmKernel::StrideA;
  using StrideB_verif = typename GemmRef::GemmKernel::StrideB;
  using StrideC_verif = typename GemmRef::GemmKernel::StrideC;
  using StrideD_verif = typename GemmRef::GemmKernel::StrideD;

  const ElementD epsilon(1e-2f);
  const ElementD non_zero_floor(1e-4f);

  for (int32_t i = 0; i < options.groups; ++i) {
    auto problem = options.problem_sizes_host.at(i);
    // we don't swap and transpose in the verify so revert the problem shape.
    auto N = get<0>(problem);
    auto M = get<1>(problem);
    auto K = get<2>(problem);
    if (M == 0) {
      continue;
    }
    else {
      StrideA_verif stride_A_verif;
      StrideB_verif stride_B_verif;

      stride_A_verif = cutlass::make_cute_packed_stride(StrideA_verif{}, cute::make_shape(M, K, 1));
      stride_B_verif = cutlass::make_cute_packed_stride(StrideB_verif{}, cute::make_shape(N, K, 1));

      int const scale_k = cutlass::ceil_div(options.k, options.c);
      auto layout_B = make_layout(cute::make_shape(N, K, Int<1>{}), stride_B_host.at(i));
      auto layout_scale_zero = make_layout(cute::make_shape(N, scale_k, Int<1>{}), stride_S_host_ref.at(i));
      cudaStream_t stream = cudaStreamDefault;
      cutlass::dequantize(block_B_dq.get() + offset_B_dq.at(i), block_B.get() + offset_B.at(i), layout_B, block_scale.get() + offset_scale.at(i), block_zero.get() + offset_zero.at(i), layout_scale_zero, options.c, stream);

      //
      // Compute reference output
      //

      typename GemmRef::Arguments arguments{
        cutlass::gemm::GemmUniversalMode::kGemm,
        {M, N, K},
        {block_A.get() + offset_A.at(i), stride_A_verif, block_B_dq.get() + offset_B_dq.at(i), stride_B_verif},
        {{alpha_host.at(i), beta_host.at(i)}, block_C.get() + offset_C.at(i), stride_C_host_ref.at(i), block_ref_D.get() + offset_D.at(i), stride_D_host_ref.at(i)}
      };

      // Run the gemm where the scaling is performed outside of the kernel.
      GemmRef gemm_ref;
      size_t workspace_size = GemmRef::get_workspace_size(arguments);
      cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
      CUTLASS_CHECK(gemm_ref.can_implement(arguments));
      CUTLASS_CHECK(gemm_ref.initialize(arguments, workspace.get()));
      CUTLASS_CHECK(gemm_ref.run());

      // Wait for kernel to finish
      CUDA_CHECK(cudaDeviceSynchronize());

      passed &= cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_D.get() + offset_D.at(i), block_D.get() + offset_D.at(i), M * N, epsilon, non_zero_floor);
      std::cout << "Group " << i << ": " << options.problem_sizes_host[i] << ", alpha: " << alpha_host[i] << ", beta: " << beta_host[i] << " Status: " << passed << std::endl;
    }
  }
  return passed;
}
```
**EN**: `verify()` constructs a separate reference GEMM path using dequantized B data. It chooses an SM90 reference schedule, explicitly calls `cutlass::dequantize()` to expand quantized B with scale/zero tensors, runs a standard GEMM per group, and compares the reference output with relative tolerances. This is the clearest place to understand the semantics of the fused kernel: the production kernel is expected to match "dequantize B, then GEMM".
**CN**: `verify()` 构造了一条使用反量化 B 数据的参考 GEMM 路径。它选择一个 SM90 参考调度，显式调用 `cutlass::dequantize()` 用 scale/zero 张量展开量化 B，对每个 group 运行标准 GEMM，并用相对误差阈值比较输出。这一段最能说明融合内核的语义：生产内核应该等价于“先反量化 B，再做 GEMM”。

### Block 9 (Lines 593-638)
```cpp

/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options &options, bool host_problem_shapes_available = true)
{
  allocate(options);
  initialize(options);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options<Gemm>(options, host_problem_shapes_available);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Check if the problem size is supported or not
  CUTLASS_CHECK(gemm.can_implement(arguments));

  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));

  // Correctness / Warmup iteration
  CUTLASS_CHECK(gemm.run());

  std::cout << "We passed all checks\n";
  // Check if output from CUTLASS kernel and reference kernel are equal or not
  MixedDtypeResult result;
  result.passed = verify(options);
  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
  grouped_mixed_dtype_profiling(gemm, options, result, alpha_host, beta_host);
  if (!result.passed) {
    exit(-1);
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

///////////////////////////////////////////////////////////////////////////////////////////////////

```
**EN**: `run()` glues the setup together: allocate, initialize, instantiate the chosen grouped GEMM, query workspace, check capability, initialize, execute once, verify, and then profile through `grouped_mixed_dtype_profiling()`. Grouped scheduling itself is mostly inside CUTLASS, but this function supplies the scheduler with exactly what it needs: a device array of problem shapes, per-group operand pointers/strides, and hardware info.
**CN**: `run()` 把整个流程串起来：分配、初始化、实例化所选 grouped GEMM、查询 workspace、检查能力、初始化、先执行一次、做校验，再通过 `grouped_mixed_dtype_profiling()` 进行性能测试。grouped 调度的核心逻辑主要封装在 CUTLASS 内部，但这个函数向调度器提供了所需的一切：设备端问题形状数组、逐组操作数指针/stride，以及硬件信息。

### Block 10 (Lines 639-692)
```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.3 Toolkit to run this example
  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 3)) {
    std::cerr << "This example requires CUDA 12.3 or newer.\n";
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (props.major != 9 || props.minor != 0) {
    std::cerr
      << "This example requires a GPU of NVIDIA's Hopper Architecture (compute capability 90).\n";
    return 0;
  }
  

  //
  // Parse options
  //

  Options options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  //
  // Evaluate CUTLASS kernels
  //

#if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
  if (options.mode == MixedDtypeGemmMode::ConvertOnly) {
    std::cout << "Running in no scale mode." << std::endl;
    run<GemmConvertOnly>(options, false);
  }
  else if (options.mode == MixedDtypeGemmMode::ScaleOnly) {
    std::cout << "Running in group scale mode." << std::endl;
    run<GemmScaleOnly>(options, false);
  }
#endif

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: `main()` validates the CUDA toolkit and Hopper device, parses options, prints help when requested, and dispatches to either the convert-only or scale-only grouped kernel. The explicit runtime mode dispatch mirrors the earlier type aliases: different mixed-input semantics are realized as different `CollectiveMainloop` instantiations selected by a small host-side switch.
**CN**: `main()` 负责检查 CUDA 工具链与 Hopper 设备、解析参数、按需打印帮助，并根据模式选择“仅转换”或“带 scale”的 grouped 内核。这里的运行时分发与前面的类型别名相互呼应：不同的 mixed-input 语义最终落实为不同的 `CollectiveMainloop` 实例，再通过一个小的主机侧分支进行选择。

---

## Key Concepts / 关键概念

**EN**:
- Grouped GEMM means one launch handles many independent GEMM problems. The scheduler lives inside CUTLASS, but the host must provide a device array of problem shapes plus matching pointer/stride arrays for every operand.
- The schedule `KernelPtrArrayTmaWarpSpecializedCooperative` combines three Hopper ideas: pointer-array addressing, TMA-fed data movement, and warp-specialized cooperative orchestration.
- Mixed-input metadata is expressed in the type system. `cute::tuple<ElementB, ElementScale>` tells CUTLASS that operand B is accompanied by a scale tensor that participates in conversion.
- Per-group alpha/beta support is not a separate kernel; it is encoded through epilogue fusion arguments that either broadcast one scalar or step through pointer arrays.
- Verification makes the fused semantics explicit by calling `cutlass::dequantize()` before a reference GEMM.

**CN**:
- Grouped GEMM 表示一次 launch 处理多个彼此独立的 GEMM 问题。调度器本身封装在 CUTLASS 内部，但主机端必须提供设备侧的问题形状数组，以及与之对应的逐操作数指针/stride 数组。
- `KernelPtrArrayTmaWarpSpecializedCooperative` 这个调度名同时体现了三种 Hopper 思想：指针数组寻址、由 TMA 驱动的数据搬运，以及 warp-specialized cooperative 协作方式。
- 混合输入元数据是在类型系统里表达的。`cute::tuple<ElementB, ElementScale>` 告诉 CUTLASS：操作数 B 旁边还伴随一个参与转换的 scale 张量。
- 逐组 alpha/beta 并不是另一套独立内核，而是通过 epilogue fusion 参数来编码：要么广播单个标量，要么沿指针数组逐组读取。
- 校验阶段通过先调用 `cutlass::dequantize()` 再运行参考 GEMM，把融合语义明确写了出来。

## Dependencies / 依赖项

**EN**:
- `grouped_mixed_dtype_utils.hpp` extends the generic mixed-dtype helpers with grouped-problem parsing, randomization/loading of per-group shapes, and grouped profiling utilities.
- `cutlass/gemm/group_array_problem_shape.hpp` defines the grouped-problem shape descriptor consumed by `GemmUniversalMode::kGrouped`.
- `cutlass/util/mixed_dtype_utils.hpp` and `cutlass::dequantize()` support explicit reference dequantization for validation.
- The CUTLASS 3 collective-builder and epilogue-builder headers are responsible for materializing the SM90 mainloop/epilogue types selected in this file.

**CN**:
- `grouped_mixed_dtype_utils.hpp` 在通用 mixed-dtype 辅助逻辑之上，补充了 grouped 问题解析、逐组形状随机化/加载，以及 grouped profiling 工具。
- `cutlass/gemm/group_array_problem_shape.hpp` 定义了 `GemmUniversalMode::kGrouped` 消费的 grouped 问题形状描述符。
- `cutlass/util/mixed_dtype_utils.hpp` 与 `cutlass::dequantize()` 支撑了校验阶段显式反量化的参考路径。
- CUTLASS 3 的 collective-builder 和 epilogue-builder 头文件负责实例化本文件选择的 SM90 mainloop/epilogue 类型。
