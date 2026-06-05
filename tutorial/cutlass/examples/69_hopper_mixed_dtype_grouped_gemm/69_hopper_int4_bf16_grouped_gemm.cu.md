# 69_hopper_int4_bf16_grouped_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/69_hopper_mixed_dtype_grouped_gemm/69_hopper_int4_bf16_grouped_gemm.cu`  
**Purpose / 用途**:  / 对 69_hopper_int4_bf16_grouped_gemm.cu 的双语代码分析。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-87) — Header comment and include set | 文件头说明与依赖引入

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
    See 55_hopper_int4_bf16_gemm.cu for more details about W4A16 GEMMs with layout shuffling.

    Limitations:
      1) Only support row-wise scaling. Zero-points and block-wise scaling is currently not supported.

    To run this example:

      $ ./examples/69_hopper_mixed_dtype_grouped_gemm/69_hopper_int4_bf16_grouped_gemm --m=2048 --n=2048 --k=2048 --mode=1 --groups=10

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
```
**EN**: The opening comment frames the example as grouped W4A16 GEMM on Hopper and immediately states an important limitation: only row-wise scaling is supported, while zero-point and blockwise scaling are not. The include set pulls in grouped problem-shape support, CUTLASS reference helpers, mixed-dtype utilities, and the grouped utility header that extends the base option logic.
**CN**: 开头注释把本例定位为 Hopper 上的 grouped W4A16 GEMM，并且马上给出一个重要限制：目前只支持按行缩放，不支持 zero-point 与 block-wise scaling。随后引入的头文件则覆盖 grouped problem shape 支持、CUTLASS 参考实现工具、mixed-dtype 工具，以及扩展基础选项逻辑的 grouped 工具头。

### Block 2 (Lines 88-163) — Grouped types, quantized reorder layout, and epilogue path | 分组类型、量化重排布局与 Epilogue 路径

```cpp

using namespace cute;

using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int,int,int>>; // <M,N,K> per group
using MmaType = cutlass::bfloat16_t;
using QuantType = cutlass::int4b_t;
constexpr int TileShapeK = 128 * 8 / sizeof_bits<MmaType>::value;

#if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////

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

// Need to pass a pointer type to make the 3rd dimension of Stride be _0
using StrideA = cute::remove_pointer_t<cutlass::detail::TagToStrideA_t<LayoutA*>>;
using StrideB = cute::remove_pointer_t<cutlass::detail::TagToStrideB_t<LayoutB*>>;

// Define the CuTe layout for reoredered quantized tensor B
// LayoutAtomQuant places values that will be read by the same thread in contiguous locations in global memory.
// It specifies the reordering within a single warp's fragment
// using ValueShuffle = Layout<_1>;                          // no value reordering
using ValueShuffle = Layout<Shape<_2,_4>, Stride<_4,_1>>; // order [0,2,4,6,1,3,5,7]
int constexpr NumShuffleAtoms = 1;
using MmaAtomShape = Layout<Shape<_1,Int<NumShuffleAtoms>>>;
using LayoutAtomQuant = decltype(cutlass::compute_memory_reordering_atom<MmaType, MmaAtomShape, ValueShuffle>());
using LayoutB_Reordered = decltype(cute::tile_to_shape(LayoutAtomQuant{}, Layout<Shape<int,int,Int<1>>, StrideB>{}));

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
**EN**: This block defines the grouped problem type, the bf16/int4 operand types, and the special reorder layout used when offline shuffling is enabled. `ValueShuffle`, `LayoutAtomQuant`, and `LayoutB_Reordered` describe how packed INT4 values are reorganized so a thread can read the fragment it needs more contiguously. The block also builds the pointer-array epilogue and the base convert-only mainloop/kernel path.
**CN**: 这一段定义了 grouped 问题类型、bf16/int4 操作数类型，以及在离线 shuffle 打开时使用的特殊重排布局。`ValueShuffle`、`LayoutAtomQuant` 和 `LayoutB_Reordered` 共同描述了打包 INT4 值如何重新组织，以便线程更连续地取到自己所需的 fragment。与此同时，本段还构造了基于指针数组的 epilogue，以及基础的 convert-only mainloop/kernel 路径。

### Block 3 (Lines 164-241) — Shuffled and scale-aware kernel variants | 带 shuffle 与 scale 的内核变体

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

using CollectiveMainloopConvertOnlyShuffled = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementB, LayoutB_Reordered *, AlignmentB,
    ElementA, LayoutA_Transpose *, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelConvertOnlyShuffled = cutlass::gemm::kernel::GemmUniversal<
    ProblemShape,
    CollectiveMainloopConvertOnlyShuffled,
    CollectiveEpilogue
>;

using GemmConvertOnlyShuffled = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelConvertOnlyShuffled>;

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

using CollectiveMainloopScaleOnlyShuffled = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale>, LayoutB_Reordered *, AlignmentB,
    ElementA, LayoutA_Transpose *, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleOnlyShuffled = cutlass::gemm::kernel::GemmUniversal<
    ProblemShape, 
    CollectiveMainloopScaleOnlyShuffled,
    CollectiveEpilogue
>;

using GemmScaleOnlyShuffled = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleOnlyShuffled>;
```
**EN**: The file then adds the remaining kernel families: shuffled convert-only, non-shuffled scale-only, and shuffled scale-only. The scale-aware builders pair `ElementB` with `ElementScale`, which is how mixed-dtype argument setup communicates that B must be converted and scaled before MMA. The stride aliases at the end bridge the grouped kernel path to both runtime arguments and the later reference implementation.
**CN**: 接下来代码补齐其余内核族：带 shuffle 的 convert-only、非 shuffle 的 scale-only，以及带 shuffle 的 scale-only。带 scale 的 builder 会把 `ElementB` 与 `ElementScale` 配成一个元组，这正是 mixed-dtype 参数设置向 mainloop 表达“B 在 MMA 前需要先转换并缩放”的方式。末尾的 stride 别名则把 grouped 内核路径与运行时参数、参考实现连接起来。

### Block 4 (Lines 242-314) — Host/device grouped storage and per-group alpha/beta arrays | 主机/设备侧 grouped 存储与逐组 alpha/beta 数组

```cpp
using StrideC = typename GemmKernelConvertOnly::InternalStrideC;
using StrideD = typename GemmKernelConvertOnly::InternalStrideD;
using StrideC_ref = cutlass::detail::TagToStrideC_t<LayoutC>;
using StrideD_ref = cutlass::detail::TagToStrideC_t<LayoutD>;
using StrideS = typename CollectiveMainloopScaleOnly::StrideScale;
using StrideS_ref = cutlass::detail::TagToStrideB_t<LayoutScale>;

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
cutlass::DeviceAllocation<LayoutB_Reordered> layout_B_reordered;
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

// Command line options parsing
```
**EN**: Grouped scheduling needs more than raw data buffers: it needs offset tables, per-group packed strides, device arrays of group pointers, and optional per-group alpha/beta storage. This block declares exactly that state. The comments around `alpha_device` and `beta_device` are especially important because they explain that alpha and beta may be supplied as arrays of pointers, letting every group use different epilogue scalars.
**CN**: grouped scheduling 需要的不只是原始数据缓冲区，还包括 offset 表、每组 packed stride、设备侧 group 指针数组，以及可选的逐组 alpha/beta 存储。本段正是在声明这些状态。尤其是 `alpha_device` 与 `beta_device` 附近的注释非常关键：它们说明 alpha/beta 可以作为“指针数组”提供，从而允许每个 group 使用不同的 epilogue 标量。

### Block 5 (Lines 315-352) — Grouped options and CLI surface | 分组选项与命令行接口

```cpp
struct Options : GroupedMixedDtypeOptions<QuantType> {
  using Base = GroupedMixedDtypeOptions<QuantType>;

  bool shuffle = true;

  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);
    cmd.get_cmd_line_argument("shuffle", shuffle);

    this->Base::parse(argc, args);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "69_hopper_int4_bf16_grouped_gemm\n\n"
      << "  Hopper Mixed Dtype Grouped GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM for all groups\n"
      << "  --n=<int>                   Sets the N extent of the GEMM for all groups\n"
      << "  --k=<int>                   Sets the K extent of the GEMM for all groups\n"
      << "  --groups=<int>              Sets the number of individual GEMM problems for Grouped GEMM\n"
      << "  --mode=<int>                The mode to run the gemm. 0 does (A @ B), 1 means A @ (scale * B), 2 means A @ (scale * B + zero-point).\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform\n\n"
      << "  --warmup=<int>              Number of warmup iterations to perform\n\n"
      << "  --shuffle=<boolean>         Enable the offline layout swizzling.\n\n"
      << "  --benchmark=<str>           Executes a benchmark problem size.\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "69_hopper_int4_bf16_grouped_gemm" << " --m=1024 --n=512 --k=1024 --groups=10 --alpha=1 --beta=0 \n\n";

    return out;
  }
};
```
**EN**: The local `Options` type extends `GroupedMixedDtypeOptions` with a `shuffle` flag and a custom usage printer. This is the user-facing entry point for grouped scheduling choices: the same binary can switch between fixed or random group shapes, scalar or per-group alpha/beta, convert-only or scale-only mode, and offline shuffling of the quantized weights.
**CN**: 本地 `Options` 类型在 `GroupedMixedDtypeOptions` 的基础上新增了 `shuffle` 开关和定制的 usage 输出。这一段就是 grouped scheduling 选择的用户入口：同一个二进制既能在固定/随机 group 形状之间切换，也能在标量或逐组 alpha/beta、convert-only 或 scale-only 模式，以及量化权重的离线 shuffle 之间切换。

### Block 6 (Lines 353-425) — Per-group allocation and stride planning | 逐组分配与步长规划

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
```
**EN**: `allocate()` walks over every grouped problem shape and computes total element counts, byte offsets, and per-group packed strides for A/B/C/D/scale/zero. This is the host-side scheduling prepass that makes grouped execution possible: once these tables exist, the kernel can treat each group as an independent GEMM while still launching a single grouped operation.
**CN**: `allocate()` 会遍历每个 grouped 问题，计算总元素数、字节偏移以及每组 A/B/C/D/scale/zero 的 packed stride。它本质上就是 grouped 执行的主机侧预调度过程：有了这些表，内核就能把每个 group 当作独立 GEMM 处理，同时仍然以一次 grouped 操作统一发射。

### Block 7 (Lines 426-557) — Pointer-array construction, dequantization, shuffle, and shape swap | 指针数组构造、反量化、shuffle 与问题形状交换

```cpp
/// Initialize operands to be used in the GEMM and reference GEMM
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

  
  for (int32_t i = 0; i < options.groups; ++i) {
    int const scale_k = cutlass::ceil_div(options.k, options.c);
    auto shape_B = cute::make_shape(cute::get<1>(options.problem_sizes_host[i]), cute::get<2>(options.problem_sizes_host[i]), Int<1>{});
    auto shape_scale = cute::make_shape(cute::get<1>(options.problem_sizes_host[i]), scale_k, Int<1>{});
    auto layout_B = make_layout(shape_B, stride_B_host.at(i));
    auto layout_scale = make_layout(shape_scale, stride_S_host_ref.at(i));
    cudaStream_t stream = cudaStreamDefault;
    cutlass::dequantize(block_B_dq.get() + offset_B_dq.at(i), block_B.get() + offset_B.at(i), layout_B, block_scale.get() + offset_scale.at(i), block_zero.get() + offset_zero.at(i), layout_scale, options.c, stream);
  }

  problem_sizes.reset(options.groups);

  if (options.shuffle) {
    std::vector<LayoutB_Reordered> layout_B_reordered_host(options.groups);
    for (int32_t i = 0; i < options.groups; ++i) {
      auto shape_B = cute::make_shape(cute::get<1>(options.problem_sizes_host[i]), cute::get<2>(options.problem_sizes_host[i]), Int<1>{});
      auto layout_B = make_layout(shape_B, stride_B_host.at(i));
      // Repeat the reorder layout atom to tile the whole tensor shape 
      layout_B_reordered_host[i] = tile_to_shape(LayoutAtomQuant{}, shape_B);
      cutlass::reorder_tensor(block_B.get() + offset_B.at(i), layout_B, layout_B_reordered_host[i]);
      if (i == 0) {
        print("Quantized tensor layout: ");
        print(layout_B_reordered_host[0]);
        print("\n");
      }
    }
    layout_B_reordered.reset(options.groups);
    layout_B_reordered.copy_from_host(layout_B_reordered_host.data());
  }

  // Reverse MN -> NM for SwapAB
  for (int32_t i = 0; i < options.groups; ++i) {
    auto [M, N, K] = options.problem_sizes_host[i];
    options.problem_sizes_host[i] = make_tuple(N, M, K);
  }
  problem_sizes.copy_from_host(options.problem_sizes_host.data());
}
```
**EN**: `initialize()` is the heart of the grouped setup. It uploads the per-group problem list, builds host arrays of pointers into the flattened device buffers, copies pointer and stride arrays to device memory, initializes tensor contents, dequantizes each group’s INT4 B tensor into a bf16 reference copy, optionally applies offline reordering, and finally swaps every host problem shape from `(M,N,K)` to `(N,M,K)` so the grouped kernel matches the explicit SwapAB convention.
**CN**: `initialize()` 是 grouped 设置流程的核心。它会上传逐组问题列表，构造指向扁平化设备缓冲区的主机指针数组，把指针数组和 stride 数组复制到设备端，初始化张量数据，为每个 group 把 INT4 的 B 反量化成 bf16 参考副本，并在需要时执行离线重排。最后，它还会把每个主机端问题形状从 `(M,N,K)` 改写成 `(N,M,K)`，以匹配显式 SwapAB 的 grouped 内核约定。

### Block 8 (Lines 558-632) — Grouped mixed-dtype argument setup | 分组 mixed-dtype 参数设置

```cpp

/// Populates a Gemm::Arguments structure from the given commandline options
template <typename Gemm>
typename Gemm::Arguments args_from_options(Options const& options, bool host_problem_shapes_available = true)
{
  using Args = typename Gemm::Arguments;
  auto&& dB = [&]() {
    // NOTE: add GemmScaleWithZeroPointShuffled
    if constexpr (cute::is_same_v<Gemm, GemmConvertOnlyShuffled> ||
                  cute::is_same_v<Gemm, GemmScaleOnlyShuffled>) {
      // offline swizzling is enabled.
      return layout_B_reordered.get();
    }
    else {
      return stride_B.get();
    }
  }();
  cutlass::KernelHardwareInfo hw_info;
  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  Args arguments;
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
    arguments = Args {
      cutlass::gemm::GemmUniversalMode::kGrouped,
      {options.groups, problem_sizes.get(), nullptr},
      {ptr_B.get(), dB, ptr_A.get(), stride_A.get()},
      {fusion_args, ptr_C.get(), stride_C.get(), ptr_D.get(), stride_D.get()},
      hw_info
    };
  }
  else if constexpr (Gemm::CollectiveMainloop::KernelConversionMode == Gemm::CollectiveMainloop::ConversionMode::ConvertAndScale) {
    arguments = Args {
      cutlass::gemm::GemmUniversalMode::kGrouped,
      {options.groups, problem_sizes.get(), nullptr},
      {ptr_B.get(), dB, ptr_A.get(), stride_A.get(), ptr_scale.get(), stride_S.get(), options.c},
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
**EN**: `args_from_options()` turns all that prepared state into a single `Gemm::Arguments` object. It first chooses whether B should be described by ordinary strides or by `LayoutB_Reordered`, then fills hardware info, and next selects one of two alpha/beta modes: a single scalar for all groups, or pointer arrays so each group can read its own scalar. Finally it emits grouped arguments in `kGrouped` mode, with either direct-convert payloads or convert+scale payloads. This function is the clearest place to study grouped scheduling plus mixed-dtype argument setup together.
**CN**: `args_from_options()` 会把前面准备好的全部状态汇总成一个 `Gemm::Arguments`。它先决定 B 是用普通 stride 还是 `LayoutB_Reordered` 来描述，再填入硬件信息，然后在两种 alpha/beta 模式中做选择：要么所有 group 共用一个标量，要么使用指针数组让每个 group 读取自己的标量。最后，它以 `kGrouped` 模式生成 grouped 参数，分别覆盖 direct-convert 或 convert+scale 两种负载。这个函数是同时理解“grouped scheduling + mixed-dtype 参数设置”的最佳入口。

### Block 9 (Lines 633-720) — Per-group reference verification | 逐组参考校验

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
**EN**: Verification reverts each host-side problem shape back to `(M,N,K)` because the reference GEMM does not use the explicit SwapAB trick. It builds a standard reference kernel, runs each group separately on the dequantized B tensor, synchronizes the device, and compares every group’s output slice against the grouped kernel result. The log line prints the reverted problem size plus the alpha/beta chosen for that group.
**CN**: 校验阶段会把每个主机端问题形状重新解释回 `(M,N,K)`，因为参考 GEMM 不采用显式 SwapAB 技巧。它构造一个普通参考内核，对每个 group 单独使用反量化后的 B 运行一次，再同步设备，并把每组输出切片与 grouped 内核结果逐一比较。日志中还会打印回退后的问题规模以及该组使用的 alpha/beta。

### Block 10 (Lines 721-761) — Grouped execution wrapper | 分组执行包装层

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
```
**EN**: The `run()` wrapper mirrors the single-GEMM example but adds the grouped allocation step. It allocates the flattened storage, initializes all pointer arrays, builds grouped arguments, launches the kernel, verifies correctness, and calls `grouped_mixed_dtype_profiling(...)` to measure the one-launch-many-problems execution model.
**CN**: `run()` 包装层与单 GEMM 示例类似，但多了 grouped 分配步骤。它会先分配扁平化存储，再初始化全部指针数组，构造 grouped 参数，启动内核，完成正确性校验，并调用 `grouped_mixed_dtype_profiling(...)` 对“一次发射、多问题执行”的模型进行计时。

### Block 11 (Lines 762-831) — Program entry and runtime dispatch | 程序入口与运行时分发

```cpp

#endif // defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

///////////////////////////////////////////////////////////////////////////////////////////////////

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
    if (options.shuffle) {
      std::cout << "Offline shuffle enabled." << std::endl;
      run<GemmConvertOnlyShuffled>(options, false);
    } else {
      std::cout << "Offline shuffle disabled." << std::endl;
      run<GemmConvertOnly>(options, false);
    }
  }
  else if (options.mode == MixedDtypeGemmMode::ScaleOnly) {
    std::cout << "Running in per-column scale mode." << std::endl;
    if (options.shuffle) {
      std::cout << "Offline shuffle enabled." << std::endl;
      run<GemmScaleOnlyShuffled>(options, false);
    } else {
      std::cout << "Offline shuffle disabled." << std::endl;
      run<GemmScaleOnly>(options, false);
    }
  }
#endif

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The entry point checks CUDA 12.3+, confirms Hopper capability, parses grouped options, and dispatches to one of four kernel adapters based on `mode` and `shuffle`. Only convert-only and scale-only are actually launched here, matching the file-level limitation note that zero-point support is not part of the runnable path.
**CN**: 程序入口会检查 CUDA 12.3+ 和 Hopper 架构能力，解析 grouped 选项，然后依据 `mode` 和 `shuffle` 分发到四个内核适配器之一。这里实际只会运行 convert-only 和 scale-only 两条路径，这也与文件开头声明的限制一致：zero-point 并不在当前可执行路径中。

---

## Key Concepts / 关键概念

**EN**: Grouped scheduling data model
  **CN**: grouped GEMM 依赖三类设备侧数组：问题形状数组、指针数组、stride/布局数组；三者共同告诉 kernel 每个 group 的真实位置和尺寸。
**EN**: Offline INT4 shuffle
  **CN**: `LayoutAtomQuant` 与 `LayoutB_Reordered` 让打包 INT4 权重在内存中按线程访存模式重排，从而改善读入局部性。
**EN**: Per-group epilogue scalars
  **CN**: alpha/beta 可以是单个标量，也可以是逐组指针数组；后一种模式允许不同 group 使用不同缩放。
**EN**: Mixed-dtype grouped arguments
  **CN**: scale-only 路径通过 `(B, scale)` 逻辑元组把量化权重与 row-wise scale 一起送入 grouped mainloop。
**EN**: Reference uses dequantized B
  **CN**: 参考实现不直接读取 INT4，而是先使用 `cutlass::dequantize` 得到 bf16 版本的 B，以便与 grouped mixed-dtype kernel 做公平比较。

## Dependencies / 依赖项

**EN**: Grouped utility header
  **CN**: 直接依赖 `grouped_mixed_dtype_utils.hpp` 提供 grouped 选项解析与 grouped profiling。
**EN**: CUTLASS grouped problem-shape support
  **CN**: 依赖 `cutlass/gemm/group_array_problem_shape.hpp` 与 pointer-array schedule 标签来表达 grouped 问题。
**EN**: Mixed-dtype utilities and reorder helpers
  **CN**: 依赖 `cutlass/util/mixed_dtype_utils.hpp` 提供反量化与重排工具。
**EN**: Reference GEMM and tensor compare helpers
  **CN**: 依赖设备参考 GEMM、tensor compare/fill 以及 host 侧参考工具完成 correctness 检查。
