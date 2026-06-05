# 55_hopper_mixed_dtype_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/55_hopper_mixed_dtype_gemm/55_hopper_mixed_dtype_gemm.cu`  
**Purpose / 用途**:  / 对 55_hopper_mixed_dtype_gemm.cu 的双语代码分析。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-107) — File header and include set | 文件头与依赖引入

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
    \brief Hopper GEMM example with different data types using CUTLASS 3.0 APIs for NVIDIA Hopper architecture

    This example shows how to perform GEMM where the input tensors A and B have different element types. CUTLASS currently supports upcasting
    from a narrower (fewer bits) to a wider (more bits) type and utilizing the tensor core instruction for the wider type. For instance, when doing
    INT8 x FP16, CUTLASS will convert INT8 -> FP16 and do math using FP16 tensor cores. Similarly, for INT4 x INT8, it will upcast to INT8 and issue math 
    using INT8 tensor cores.

    The narrower type always passes through the register file. Therefore, in cases where the narrower type is operand B, the collective will implicitly swap 
    A and B in the main loop. However, implicit swaps do not support TMA epilogues. Consequently, it is essential to consider this when constructing the epilogue, 
    as illustrated in this example.

    Note that in this example, we explicitly swap A and B in order to use TMA epilogues. We do this since TMA epilogues are more performant on problem sizes of interest.

    It is expected that the scale's K dimension be scale_k = ceil_div(problem_k, group_size). 
    
    Scales are always expected to be MN major. This means the fastest changing dimension must be M if A is scaled or N if B is scaled.
    
    If A is being scaled, the scales should have shape [M, scale_k],  while if B is scaled, it must have shape [N, scale_k].

    The implementation only supports "group-wise" scales. However, we can make it work for per-column scales by setting the groups size
    equal to the gemm problem K.

    Limitations:
      1) The narrow type must always be in K-major format.
      2) The scales and zeros must be MN major. That means if A is scaled, it must be column major, but if B is scaled it must be row major.
      3) The scales and the zeros must have the same layout and groupsize.
      4) The groupsize must be greater or equal to tile shape k.
      5) When dealing with 8-bit x {4-bit, 2-bit}, both inputs must be in K-major format.
      6) Currently, TMA epilogues cannot be used when the narrow type is the B operand. This limitation arises because the implementation always swaps the 
         operands to ensure that the narrow type passes through the register file, and TMA epilogues do not currently support implicit swap + transpose operations. 
         We plan to address this limitation in the future. However, we address this in the example by explicitly swapping and transposing the operands.
    
    Optimizing suggestions:
      1) Use a small tile size, since the register pressure for this GEMM (and RS GEMM in general) is high (it uses a lot of register space).
      2) Try avoid using scale or zero mode cause the computations will be the bottleneck.

    Examples:
      
      Runs the mixed input batched gemm (with batch size 2), converting B to the type of A (mode 0)
      $ ./examples/55_hopper_mixed_dtype_gemm/55_hopper_mixed_dtype_gemm --m=2048 --n=2048 --k=2048 --l=2 --mode=0

      Runs the mixed input gemm, and applies a scaling factor to B before mma (mode 1). Applies a vector of scales to the entire
      matrix (group size is the same as the gemm k dimension).
      $ ./examples/55_hopper_mixed_dtype_gemm/55_hopper_mixed_dtype_gemm --m=4096 --n=5120 --k=8192 --g=8192 --mode=1

      Runs the mixed input gemm, and applies a scaling factor and adds a zero-point to B before mma (mode 2). Uses a group size of 128.
      $ ./examples/55_hopper_mixed_dtype_gemm/55_hopper_mixed_dtype_gemm --m=2048 --n=5120 --k=8192 --g=128 --mode=2
*/

#include <iostream>

#include "cutlass/cutlass.h"

#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/thread/linear_combination.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/mixed_dtype_utils.hpp"

#include "helper.h"
#include "mixed_dtype_utils.hpp"
```
**EN**: The opening comment explains the core idea of mixed-dtype GEMM on Hopper: a narrower operand is upcast to the wider MMA type, and this example explicitly swaps/transposes operands so a TMA epilogue remains usable. The include set pulls in CUTLASS collective builders, universal GEMM adapters, mixed-dtype utilities, and helper code used later for initialization and profiling.
**CN**: 开头注释先说明 Hopper 上混合数据类型 GEMM 的核心思路：较窄的输入会被提升到更宽的 MMA 类型；本例通过显式交换/转置操作数，保证仍可使用 TMA epilogue。随后引入的头文件覆盖了 CUTLASS collective builder、universal GEMM 适配器、mixed-dtype 工具以及后续初始化和性能测试需要的辅助代码。

### Block 2 (Lines 108-170) — Core type aliases and epilogue construction | 核心类型别名与 Epilogue 构造

```cpp
using namespace cute;

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////
using MmaType = cutlass::half_t;
using QuantType = cutlass::float_e4m3_t;
constexpr int TileShapeK = 128 * 8 / sizeof_bits<MmaType>::value;

// A matrix configuration
using         ElementA    = MmaType;                                        // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = QuantType;                                      // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// This example manually swaps and transposes, so keep transpose of input layouts
using LayoutA_Transpose = typename cutlass::layout::LayoutTranspose<LayoutA>::type;
using LayoutB_Transpose = typename cutlass::layout::LayoutTranspose<LayoutB>::type;

using ElementZero = cutlass::half_t;
using ElementScale = cutlass::half_t;
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
using ElementCompute      = float;                                          // Element type for epilogue computation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_128,_128,cute::Int<TileShapeK>>;         // Threadblock-level tile size
using ClusterShape        = Shape<_1,_1,_1>;                                // Shape of the threadblocks in a cluster
using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecializedCooperative;  // Kernel to launch based on the default setting in the Collective Builder 
using EpilogueSchedule    = cutlass::epilogue::TmaWarpSpecializedCooperative;
using EpilogueTileType    = cutlass::epilogue::collective::EpilogueTileAuto;

using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
    TileShape, ClusterShape,
    EpilogueTileType,
    ElementAccumulator, ElementAccumulator,
    // Transpose layout of D here since we use explicit swap + transpose
    // the void type for C tells the builder to allocate 0 smem for the C matrix.
    // We can enable this if beta == 0 by changing ElementC to void below.
    ElementC, typename cutlass::layout::LayoutTranspose<LayoutC>::type, AlignmentC,
    ElementD, typename cutlass::layout::LayoutTranspose<LayoutD>::type, AlignmentD,
    EpilogueSchedule // This is the only epi supporting the required swap + transpose.
  >::CollectiveOp;
```
**EN**: This block defines the wide MMA type (`half_t`), the narrow quantized input (`float_e4m3_t`), the transpose layouts used by the explicit SwapAB path, and the scale/zero element types. The epilogue builder is configured with transposed C/D layouts because the kernel consumes swapped operands but still needs to write logically correct output tiles.
**CN**: 这一段定义了宽 MMA 类型（`half_t`）、窄量化输入（`float_e4m3_t`）、显式 SwapAB 路径使用的转置布局，以及 scale/zero 的元素类型。Epilogue builder 这里对 C/D 使用转置布局，是因为内核内部已经交换了操作数，但最终仍要写出逻辑上正确的输出块。

### Block 3 (Lines 171-237) — Mainloop variants for convert / scale / zero-point | 转换 / 缩放 / 零点三种 Mainloop 变体

```cpp
// ============================================================ MIXED INPUT NO SCALES ============================================================================
// The collective will infer that the narrow type should be upcasted to the wide type.
// We swap A and B operands to the builder here
using CollectiveMainloopConvertOnly = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementB, LayoutB_Transpose, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelConvertOnly = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopConvertOnly,
    CollectiveEpilogue
>;

using GemmConvertOnly = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelConvertOnly>;

// =========================================================== MIXED INPUT WITH SCALES ===========================================================================
// The Scale information must get paired with the operand that will be scaled. In this example, B is scaled so we make a tuple of B's information and the scale information.
using CollectiveMainloopScaleOnly = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale>, LayoutB_Transpose, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleOnly = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopScaleOnly,
    CollectiveEpilogue
>;

using GemmScaleOnly = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleOnly>;

// =========================================================== MIXED INPUT WITH SCALES AND ZEROS ==================================================================
// We specify scale + zero elements to indicate that we require both. Scales and biases have the same format.
using CollectiveMainloopScaleWithZeroPoint = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    cute::tuple<ElementB, ElementScale, ElementZero>, LayoutB_Transpose, AlignmentB,
    ElementA, LayoutA_Transpose, AlignmentA,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernelScaleWithZeroPoint = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopScaleWithZeroPoint,
    CollectiveEpilogue
>;

using GemmScaleWithZeroPoint = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelScaleWithZeroPoint>;
// =================================================================================================================================================================
```
**EN**: The example creates three `CollectiveMainloop` variants. `ConvertOnly` describes direct upcasting of B into A’s MMA type. `ScaleOnly` encodes B as a tuple `(quantized value, scale)` so scaling happens in the mainloop. `ScaleWithZeroPoint` extends that tuple with zero-point data. Each collective is then wrapped in a `GemmUniversal` kernel and a device adapter.
**CN**: 这里构造了三个 `CollectiveMainloop` 变体。`ConvertOnly` 表示仅把 B 直接提升为 A 的 MMA 类型；`ScaleOnly` 把 B 编码为 `(量化值, scale)` 元组，使缩放在 mainloop 中完成；`ScaleWithZeroPoint` 则进一步加入 zero-point。随后每个 collective 都被包装成 `GemmUniversal` 内核以及设备侧适配器。

### Block 4 (Lines 238-287) — Stride aliases and device-side state | 步长别名与设备侧状态

```cpp

using StrideA = cutlass::detail::TagToStrideA_t<LayoutA>;
using StrideB = cutlass::detail::TagToStrideB_t<LayoutB>;
using StrideC = typename GemmKernelScaleWithZeroPoint::StrideC;
using StrideD = typename GemmKernelScaleWithZeroPoint::StrideD;

using StrideC_ref = cutlass::detail::TagToStrideC_t<LayoutC>;
using StrideD_ref = cutlass::detail::TagToStrideC_t<LayoutD>;

//
// Data members
//

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideC_ref stride_C_ref;
StrideD stride_D;
StrideD_ref stride_D_ref;
uint64_t seed;

// Scale and Zero share a stride since the layout and shapes must be the same.
using StrideS = typename CollectiveMainloopScaleWithZeroPoint::StrideScale;
using StrideS_ref = cutlass::detail::TagToStrideB_t<LayoutScale>;
StrideS stride_S;
StrideS_ref stride_S_ref;

cutlass::DeviceAllocation<ElementA> block_A;
cutlass::DeviceAllocation<ElementB> block_B;
cutlass::DeviceAllocation<ElementA> block_B_dq;
cutlass::DeviceAllocation<ElementScale> block_scale;
cutlass::DeviceAllocation<ElementZero> block_zero;
cutlass::DeviceAllocation<ElementC> block_C;
cutlass::DeviceAllocation<typename GemmScaleWithZeroPoint::EpilogueOutputOp::ElementOutput> block_D;
cutlass::DeviceAllocation<typename GemmScaleWithZeroPoint::EpilogueOutputOp::ElementOutput> block_ref_D;

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM setup and evaluation
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Initialize operands to be used in the GEMM and reference GEMM
```
**EN**: After defining the kernel families, the file extracts stride types for the actual kernel path and for the reference path. It then declares global testbed state: packed strides, the scale stride family, and device allocations for A/B/C/D, dequantized B, and scale/zero buffers. This is the persistent host-side state that later functions fill and reuse.
**CN**: 在内核类型定义完成后，代码进一步提取实际内核路径与参考路径各自使用的 stride 类型。随后声明测试平台全局状态：打包步长、scale 的步长类型，以及 A/B/C/D、反量化 B、scale/zero 缓冲区的设备分配。这些全局对象会被后续初始化、校验和性能函数反复复用。

### Block 5 (Lines 288-330) — Tensor initialization and reference dequantization | 张量初始化与参考反量化

```cpp
void initialize(MixedDtypeOptions const& options) {

  auto shape_b = cute::make_shape(options.n, options.k, options.l);
  int const scale_k = cutlass::ceil_div(options.k, options.g);
  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, shape_b);
  // Reverse stride here due to swap and transpose
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.n, options.m, options.l));
  stride_C_ref = cutlass::make_cute_packed_stride(StrideC_ref{}, cute::make_shape(options.m, options.n, options.l));
  // Reverse stride here due to swap and transpose
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.n, options.m, options.l));
  stride_D_ref = cutlass::make_cute_packed_stride(StrideD_ref{}, cute::make_shape(options.m, options.n, options.l));

  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);

  block_A.reset(a_coord.product());
  block_B.reset(b_coord.product());
  block_B_dq.reset(b_coord.product());
  block_C.reset(c_coord.product());
  block_D.reset(c_coord.product());
  block_ref_D.reset(c_coord.product());

  block_scale.reset(scale_k * options.l * options.n);
  block_zero.reset(scale_k * options.l * options.n);

  initialize_tensor(block_A, seed + 2022);
  initialize_tensor(block_B, seed + 2021);
  initialize_tensor(block_C, seed + 2020);
  initialize_scale(block_scale, options);
  initialize_zero(block_zero, options);

  auto layout_B = make_layout(shape_b, stride_B);

  auto shape_scale_zero = cute::make_shape(options.n, scale_k, options.l);
  stride_S = cutlass::make_cute_packed_stride(StrideS{}, cute::make_shape(options.n, scale_k, options.l));
  stride_S_ref = cutlass::make_cute_packed_stride(StrideS_ref{}, cute::make_shape(options.n, scale_k, options.l));
  auto layout_scale_zero = cute::make_layout(shape_scale_zero, stride_S_ref);

  cudaStream_t stream = cudaStreamDefault;
  cutlass::dequantize(block_B_dq.get(), block_B.get(), layout_B, block_scale.get(), block_zero.get(), layout_scale_zero, options.g, stream);
}
```
**EN**: `initialize()` computes packed strides for the swapped execution path and the unswapped reference path, allocates all device buffers, fills operands, prepares the scale/zero tensors, and finally calls `cutlass::dequantize(...)` to materialize a dense wide-type copy of B. That dequantized tensor is crucial for the reference GEMM used in verification.
**CN**: `initialize()` 会分别为交换后执行路径与未交换的参考路径计算 packed stride，分配全部设备缓冲区，填充输入，并准备 scale/zero 张量。最后它调用 `cutlass::dequantize(...)` 把量化 B 展开为宽类型的密集副本，这个反量化张量正是后续参考 GEMM 校验的基础。

### Block 6 (Lines 331-364) — Mode-dependent argument packing | 按模式打包运行时参数

```cpp

/// Populates a Gemm::Arguments structure from the given commandline options
template <typename Args>
Args args_from_options(MixedDtypeOptions const& options)
{
// Swap the A and B tensors, as well as problem shapes here.
  if (options.mode == MixedDtypeGemmMode::ConvertOnly) {
    return Args {
      cutlass::gemm::GemmUniversalMode::kGemm,
      {options.n, options.m, options.k, options.l},
      {block_B.get(), stride_B, block_A.get(), stride_A},
      {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D}
    };
  } 
  else if (options.mode == MixedDtypeGemmMode::ScaleOnly) {
    return Args {
      cutlass::gemm::GemmUniversalMode::kGemm,
      {options.n, options.m, options.k, options.l},
      {block_B.get(), stride_B, block_A.get(), stride_A, block_scale.get(), stride_S, options.g},
      {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D}
    };
  } 
  else if (options.mode == MixedDtypeGemmMode::ScaleWithZeroPoint) {
    return Args {
      cutlass::gemm::GemmUniversalMode::kGemm,
      {options.n, options.m, options.k, options.l},
      {block_B.get(), stride_B, block_A.get(), stride_A, block_scale.get(), stride_S, options.g, block_zero.get()},
      {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D}
    };
  } else {
    std::cerr << "Invalid mode " << options.mode << ". Must be 0, 1 or 2." << std::endl;
    exit(-1);
  }
}
```
**EN**: `args_from_options()` translates CLI options into the exact `Gemm::Arguments` form expected by CUTLASS. The function swaps the problem shape from `{M,N,K,L}` to `{N,M,K,L}` to match the explicit SwapAB kernel definition, then chooses among the convert-only, scale-only, or scale+zero-point payloads. The narrow B operand is always placed first because the kernel was built around the swapped ordering.
**CN**: `args_from_options()` 将命令行选项转成 CUTLASS 所需的 `Gemm::Arguments`。函数会先把问题形状从 `{M,N,K,L}` 改成 `{N,M,K,L}`，以匹配显式 SwapAB 的内核定义，然后根据模式选择仅转换、缩放或缩放+零点三种参数负载。由于内核以交换后的顺序构建，较窄的 B 操作数始终排在最前。

### Block 7 (Lines 365-428) — Reference GEMM verification path | 参考 GEMM 校验路径

```cpp

bool verify(MixedDtypeOptions const& options) {
  //
  // Compute reference output
  //

  // In this example, we use the GPU default kernels as a reference (unfused scale)
  // This avoids numerical differences due to different accumulation order.

  // Again, due to numerical differences, we must use fast acc here when the mma type is
  // FP8 as the fused implementation only supports fast acc at the moment.
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
      Shape<int,int,int,int>, // Indicates ProblemShape
      CollectiveMainloopRef,
      CollectiveEpilogueRef
  >;

  using GemmRef = cutlass::gemm::device::GemmUniversalAdapter<GemmKernelRef>;

  typename GemmRef::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {block_A.get(), stride_A, block_B_dq.get(), stride_B},
    {{options.alpha, options.beta}, block_C.get(), stride_C_ref, block_ref_D.get(), stride_D_ref}
  };

  // Run the gemm where the scaling is performed outside of the kernel.
  GemmRef gemm_ref;
  size_t workspace_size = GemmRef::get_workspace_size(arguments);
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
  CUTLASS_CHECK(gemm_ref.can_implement(arguments));
  CUTLASS_CHECK(gemm_ref.initialize(arguments, workspace.get()));
  CUTLASS_CHECK(gemm_ref.run());

  // compare_reference
  ElementD const epsilon(1e-2f);
  ElementD const non_zero_floor(1e-4f);
  bool passed = cutlass::reference::device::BlockCompareRelativelyEqual(block_ref_D.get(), block_D.get(), block_D.size(), epsilon, non_zero_floor);
  return passed;
}
```
**EN**: `verify()` builds a separate reference kernel that consumes wide-type A and wide-type dequantized B. The code conditionally selects a fast-accumulation FP8 schedule when the MMA type itself is FP8, initializes a standard GEMM argument list, runs the unfused reference kernel, and compares the reference output with the mixed-dtype kernel output using relative error thresholds.
**CN**: `verify()` 会构造一个独立的参考内核，它直接使用宽类型 A 与已经反量化的宽类型 B。若 MMA 类型本身是 FP8，代码还会选择对应的 fast-accumulation 调度。随后它初始化标准 GEMM 参数、运行未融合的参考内核，并用相对误差阈值把参考结果与 mixed-dtype 内核结果进行比较。

### Block 8 (Lines 429-467) — Execution and profiling wrapper | 执行与性能统计包装层

```cpp

/// Execute a given example GEMM computation
template <typename Gemm>
int run(MixedDtypeOptions &options)
{
  initialize(options);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options<typename Gemm::Arguments>(options);

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

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  MixedDtypeResult result;
  result.passed = verify(options);
  mixed_dtype_profiling(gemm, options, result);
  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
  if (!result.passed) {
    exit(-1);
  }

  return 0;
}
```
**EN**: `run()` is the end-to-end driver for one kernel instantiation. It initializes tensors, builds arguments, allocates workspace, checks `can_implement`, initializes the kernel, launches one warmup/correctness run, verifies the result, and then forwards the kernel object plus options into `mixed_dtype_profiling(...)` for repeated timing.
**CN**: `run()` 是单个内核实例的端到端驱动器：先初始化张量，再构造参数、申请 workspace、检查 `can_implement`、初始化内核，执行一次预热/正确性运行，然后完成结果校验，最后把 kernel 对象和 options 交给 `mixed_dtype_profiling(...)` 做重复计时。

### Block 9 (Lines 468-540) — Program entry and runtime mode selection | 程序入口与运行模式选择

```cpp

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

///////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA 12.0 Toolkit to run this example
  // and must have compute capability at least 90.
  if (__CUDACC_VER_MAJOR__ < 12) {
    std::cerr << "This example requires CUDA 12 or newer.\n";
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

  MixedDtypeOptions options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  //
  // Evaluate CUTLASS kernels
  //

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
  if (options.mode == MixedDtypeGemmMode::ConvertOnly) {
    std::cout << "Running in no scale mode." << std::endl;
    run<GemmConvertOnly>(options);
  }
  else if (options.mode == MixedDtypeGemmMode::ScaleOnly) {
    if (options.g == options.k) {
      std::cout << "Running in per-column scale mode." << std::endl;
    } else {
      std::cout << "Running in group scale mode." << std::endl;
    }
    run<GemmScaleOnly>(options);
  }
  else if (options.mode == MixedDtypeGemmMode::ScaleWithZeroPoint) {
    if (options.g == options.k) {
      std::cout << "Running in per-column scale and zero mode." << std::endl;
    } else {
      std::cout << "Running in group scale and zero mode." << std::endl;
    }
    run<GemmScaleWithZeroPoint>(options);
  }
#endif

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The `main()` function performs CUDA toolkit and Hopper capability checks, parses command-line options, prints help when requested, and dispatches to the correct kernel family based on `options.mode`. The printed messages distinguish convert-only, group-scale, and group-scale+zero execution so the user knows which mixed-dtype path is active.
**CN**: `main()` 先检查 CUDA Toolkit 版本和 Hopper 架构能力，再解析命令行参数；若请求帮助则打印用法；否则根据 `options.mode` 分发到对应的内核族。输出信息会明确区分仅转换、分组缩放以及分组缩放+零点三条 mixed-dtype 路径，便于确认当前运行模式。

---

## Key Concepts / 关键概念

**EN**: Explicit SwapAB for TMA epilogue
  **CN**: 因为窄类型位于 B 时隐式交换不支持 TMA epilogue，所以示例通过显式 SwapAB 和转置手动整理 A/B/C/D 的布局与问题形状。
**EN**: Scale tensor semantics
  **CN**: 当 B 被缩放时，scale 张量按 `[N, ceil_div(K, g), L]` 的语义组织，`g` 是 group size；zero-point 张量与 scale 共享相同布局约束。
**EN**: Mode-driven argument setup
  **CN**: `args_from_options()` 直接映射三种转换模式，使 mainloop 参数与 `CollectiveMainloop::*ConversionMode` 一一对应。
**EN**: Reference strategy
  **CN**: 校验路径先在核外反量化 B，再运行普通宽类型 GEMM，这样可以避免融合实现与参考实现因累加顺序不同带来的额外偏差。
**EN**: Shared-memory carveout coupling
  **CN**: 主循环的 stage 数通过 `StageCountAutoCarveout<sizeof(CollectiveEpilogue::SharedStorage)>` 自动扣除 epilogue 共享内存占用。

## Dependencies / 依赖项

**EN**: CUTLASS collective builders and universal GEMM adapter
  **CN**: 依赖 `collective_builder`、`gemm_universal` 和 `GemmUniversalAdapter` 把 mainloop 与 epilogue 组合成完整 Hopper GEMM。
**EN**: Mixed-dtype utility helpers
  **CN**: 依赖 `cutlass/util/mixed_dtype_utils.hpp` 与本目录 `mixed_dtype_utils.hpp` 完成 scale/zero 初始化、命令行封装和 profiling。
**EN**: Reference and comparison utilities
  **CN**: 依赖设备侧 tensor 比较、反量化与 host/device 参考实现来验证 fused mixed-dtype kernel 的正确性。
**EN**: Example helper glue
  **CN**: `helper.h` 提供了示例中常见的 CUDA 检查与计时辅助。
