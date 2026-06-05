# 54_hopper_fp8_warp_specialized_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/54_hopper_fp8_warp_specialized_gemm/54_hopper_fp8_warp_specialized_gemm.cu`  
**Purpose / 用途**: Shows how to configure and run a Hopper FP8 warp-specialized GEMM with fused scaling, optional auxiliary output, activation, and amax tracking. / 展示如何配置并运行一个 Hopper FP8 warp-specialized GEMM，同时融合缩放、可选辅助输出、激活函数以及 amax 跟踪。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-58

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
    \brief Simple Hopper FP8 GEMM example using CUTLASS 3.0 APIs for NVIDIA Hopper architecture

    This example demonstrate a simple way to instantiate and run a FP8 GEMM using the new CUTLASS 3.0
    APIs on NVIDIA Hopper architecture. New features that will be showcased in this example are as follows:

    1. NVIDIA Hopper architecture introduces a new series of tensor core instructions (GMMA)
    which are more efficient than the Ampere tensor core instructions.

    2. NVIDIA Hopper architecture includes new Tensor Memory Accelerator (TMA) unit to transfer large
    blocks of data efficiently between global memory and shared memory. TMA also supports asynchronous
    copies between thread blocks in a cluster.

    3. This example uses the Warp Specialized kernel design (see /media/docs/efficient_gemm.md for details).

    4. This example shows all important fusions used by FP8 gemm kernels,
    i.e., scale factor for A, B, C, D tensor, the abs_max value of D tensor.

    5. A simple way to tune the CTA rasterization direction and swizzle pattern of Hopper kernels. Both the
    CTA rasterization direction and swizzle pattern impact cross-CTA locality of accesses. By tuning we can
    improve performance.

    Examples:

      $ ./examples/54_hopper_fp8_warp_specialized_gemm/54_hopper_fp8_warp_specialized_gemm --m=2048 --n=2048 --k=2048 --rasterization=N --swizzle=2
*/

```

**EN**: The file-level comment frames the example as a Hopper FP8 GEMM showcase. Besides GMMA and TMA, it explicitly highlights epilogue fusions unique to FP8 workflows: scale factors for A/B/C/D, auxiliary output, activation, and absolute-maximum tracking for dynamic scaling.
**CN**: 文件级注释把该示例定位为 Hopper FP8 GEMM 展示案例。除 GMMA 与 TMA 外，它还明确强调了 FP8 工作流中特有的 epilogue 融合：A/B/C/D 的缩放因子、辅助输出、激活函数，以及用于动态缩放的绝对值最大值跟踪。

### Lines 59-90

```cpp
#include <iostream>

#include "cutlass/cutlass.h"
#include "cutlass/numeric_types.h"

#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"
#include "cutlass/gemm/kernel/tile_scheduler_params.h"
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gett.hpp"


#include "helper.h"
#include "hopper_fp8_commandline.hpp"

using namespace cute;

```

**EN**: The include set adds the usual CUTLASS collective-builder infrastructure plus `hopper_fp8_commandline.hpp`, which centralizes the richer option set needed by FP8 examples. This already hints that runtime configuration is more involved than in the simpler TF32 GEMM example.
**CN**: 头文件列表除了常规 CUTLASS collective-builder 基础设施之外，还引入了 `hopper_fp8_commandline.hpp`，它集中管理 FP8 示例所需的更复杂参数集。这已经暗示：相比简单的 TF32 GEMM，FP8 的运行时配置会更繁复。

### Lines 91-178

```cpp
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////

// A matrix configuration
using         ElementA    = cutlass::float_e4m3_t;                          // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = cutlass::float_e4m3_t;                          // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C matrix configuration
using         ElementC    = cutlass::float_e4m3_t;                          // Element type for C and D matrix operands
using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// D matrix configuration
using         ElementD    = ElementC;
using         LayoutD     = LayoutC;
constexpr int AlignmentD  = AlignmentC;

// Auxiliary matrix configuration and other fusion types
using         ElementAux   = ElementC;
using         LayoutAux    = LayoutC;
using         ElementAmax  = float;
using         ElementBias  = float;

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ElementCompute      = float;                                          // Element type for epilogue computation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_128,_128,_128>;                           // Threadblock-level tile size
using ClusterShape        = Shape<_1,_2,_1>;                                // Shape of the threadblocks in a cluster
using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecializedCooperative;
using EpilogueSchedule    = cutlass::epilogue::TmaWarpSpecializedCooperative;
using EpilogueTileType    = cutlass::epilogue::collective::EpilogueTileAuto;
using FusionOperation     = cutlass::epilogue::fusion::ScaledLinCombPerRowBiasEltActAmaxAux<
    LayoutAux, cutlass::epilogue::thread::ReLU, ElementD, ElementCompute, ElementAux, ElementAmax, ElementBias, ElementC>;

using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    TileShape, ClusterShape,
    EpilogueTileType,
    ElementAccumulator, ElementCompute,
    ElementC, LayoutC, AlignmentC,
    ElementD, LayoutD, AlignmentD,
    EpilogueSchedule,
    FusionOperation
  >::CollectiveOp;

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementA, LayoutA, AlignmentA,
    ElementB, LayoutB, AlignmentB,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloop,
    CollectiveEpilogue
>;

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

// Extract information from Gemm kernel.
using EpilogueOutputOp  = typename Gemm::EpilogueOutputOp;
using ElementScalar     = typename EpilogueOutputOp::ElementScalar;
using ElementAmax       = typename EpilogueOutputOp::ElementAmax;
using ActivationFunctor = typename EpilogueOutputOp::ActivationFn;

using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideC = typename Gemm::GemmKernel::StrideC;
using StrideD = typename Gemm::GemmKernel::StrideD;
using StrideAux = StrideD;

```

**EN**: This block chooses the compile-time kernel structure. A/B/C use FP8 E4M3, accumulation and epilogue compute stay in FP32, and the tile/cluster are set to `128x128x128` and `1x2x1`. The most important traits are `KernelTmaWarpSpecializedCooperative` and `TmaWarpSpecializedCooperative`, which force a cooperative Hopper schedule built around TMA-fed, GMMA-executed warp-specialized pipelines. `FusionOperation` selects `ScaledLinCombPerRowBiasEltActAmaxAux`, so the epilogue is not just a store path—it is a compile-time composition of scaling, bias slot, activation, auxiliary output, and amax collection. `StageCountAutoCarveout` again sizes the mainloop after accounting for epilogue shared storage.
**CN**: 这一段选择了 kernel 的编译期结构。A/B/C 使用 FP8 E4M3，累加与 epilogue 计算保留在 FP32，tile/cluster 设置为 `128x128x128` 和 `1x2x1`。最关键的 traits 是 `KernelTmaWarpSpecializedCooperative` 与 `TmaWarpSpecializedCooperative`，它们强制选用以 TMA 供数、GMMA 执行的 cooperative Hopper warp-specialized 调度。`FusionOperation` 选择了 `ScaledLinCombPerRowBiasEltActAmaxAux`，因此 epilogue 不只是存储路径，而是缩放、bias 槽位、激活、辅助输出和 amax 统计的编译期组合。`StageCountAutoCarveout` 仍然会在考虑 epilogue 共享内存之后再确定主循环的 stage 数。

### Lines 179-217

```cpp
constexpr bool IsDFp8 =
    cute::is_same_v<ElementD, cutlass::float_e4m3_t> or
    cute::is_same_v<ElementD, cutlass::float_e5m2_t>;

constexpr bool IsAuxFp8 =
    cute::is_same_v<ElementAux, cutlass::float_e4m3_t> or
    cute::is_same_v<ElementAux, cutlass::float_e5m2_t>;

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;
StrideAux stride_aux;
uint64_t seed;

cutlass::HostTensor<ElementA  , LayoutA  > tensor_A;
cutlass::HostTensor<ElementB  , LayoutB  > tensor_B;
cutlass::HostTensor<ElementC  , LayoutC  > tensor_C;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_D;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_ref_D;
cutlass::HostTensor<ElementAux, LayoutAux> tensor_aux;
cutlass::HostTensor<ElementAux, LayoutAux> tensor_ref_aux;

using LayoutScalar = cutlass::layout::PackedVectorLayout;
cutlass::HostTensor<ElementScalar, LayoutScalar> scalar_alpha;
cutlass::HostTensor<ElementScalar, LayoutScalar> scalar_beta;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_A;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_B;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_C;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_D;
cutlass::HostTensor<ElementScalar, LayoutScalar> scale_aux;
cutlass::HostTensor<ElementAmax  , LayoutScalar> abs_max_D;
cutlass::HostTensor<ElementAmax  , LayoutScalar> reference_abs_max_D;
cutlass::HostTensor<ElementAmax  , LayoutScalar> abs_max_aux;
cutlass::HostTensor<ElementAmax  , LayoutScalar> reference_abs_max_aux;

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

```

**EN**: After the kernel aliases, the file derives runtime-relevant types and allocates host-side state. `IsDFp8` and `IsAuxFp8` are compile-time traits that later guard optional allocations and checks. The dense collection of `HostTensor` objects reflects the richer FP8 epilogue contract: besides A/B/C/D, the example may carry aux output, scalar alpha/beta, per-tensor scales, and amax buffers.
**CN**: 在 kernel 类型别名之后，文件继续推导与运行时相关的类型并声明主机侧状态。`IsDFp8` 与 `IsAuxFp8` 是编译期 traits，后续会用来保护可选的分配和检查。大量 `HostTensor` 对象的存在也反映了 FP8 epilogue 契约更丰富：除了 A/B/C/D 外，示例还可能携带辅助输出、alpha/beta 标量、各张量缩放因子以及 amax 缓冲区。

### Lines 218-243

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90Params::RasterOrderOptions;

/// Result structure
struct Result
{
  double avg_runtime_ms;
  double gflops;
  cutlass::Status status;
  cudaError_t error;
  bool passed;

  Result(
    double avg_runtime_ms = 0,
    double gflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cudaError_t error = cudaSuccess)
  :
    avg_runtime_ms(avg_runtime_ms), gflops(gflops), status(status), error(error), passed(false)
  {}

};

```

**EN**: `RasterOrderOptions` and `Result` are the small runtime utilities shared with the simpler example: they describe scheduler traversal options and store benchmark/correctness results.
**CN**: `RasterOrderOptions` 与 `Result` 是与简化示例共用的小型运行时工具：前者描述调度器遍历方式，后者保存性能与正确性结果。

### Lines 244-354

```cpp
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM setup and evaluation
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Helper to initialize a block of device data
template <typename Element, typename Layout>
bool initialize_tensor(
  cutlass::TensorView<Element, Layout> view,
  uint64_t seed) {

  double scope_max, scope_min;
  int bits_input = cutlass::sizeof_bits<Element>::value;
  int bits_output = cutlass::sizeof_bits<Element>::value;

  if (bits_input == 1) {
    scope_max = 2;
    scope_min = 0;
  }
  else if (bits_input <= 8) {
    scope_max = 2;
    scope_min = -2;
  }
  else if (bits_output == 16) {
    scope_max = 5;
    scope_min = -5;
  }
  else {
    scope_max = 8;
    scope_min = -8;
  }
  cutlass::reference::host::TensorFillRandomUniform(
    view, seed, scope_max, scope_min, 0);

  return true;
}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options<RasterOrderOptions> &options) {

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.m, options.n, options.l));
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));
  stride_aux = stride_D;

  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);

  tensor_A.resize(a_coord);
  tensor_B.resize(b_coord);
  tensor_C.resize(c_coord);
  tensor_D.resize(c_coord);
  tensor_ref_D.resize(c_coord);

  initialize_tensor(tensor_A.host_view(), seed + 2022);
  initialize_tensor(tensor_B.host_view(), seed + 2023);
  initialize_tensor(tensor_C.host_view(), seed + 2024);

  tensor_A.sync_device();
  tensor_B.sync_device();
  tensor_C.sync_device();
  tensor_D.sync_device();

  if (options.save_aux) {
    tensor_aux.resize(c_coord);
    tensor_aux.sync_device();
    tensor_ref_aux.resize(c_coord);
  }

  if (options.device_scale) {
    scalar_alpha.resize(cutlass::make_Coord(1));
    scalar_beta.resize(cutlass::make_Coord(1));
    scale_A.resize(cutlass::make_Coord(1));
    scale_B.resize(cutlass::make_Coord(1));
    scale_C.resize(cutlass::make_Coord(1));
    scale_D.resize(cutlass::make_Coord(1));
    scale_aux.resize(cutlass::make_Coord(1));

    cutlass::reference::host::TensorFill(scalar_alpha.host_view(), options.alpha);
    cutlass::reference::host::TensorFill(scalar_beta.host_view(), options.beta);
    cutlass::reference::host::TensorFill(scale_A.host_view(), options.scale_a);
    cutlass::reference::host::TensorFill(scale_B.host_view(), options.scale_b);
    cutlass::reference::host::TensorFill(scale_C.host_view(), options.scale_c);
    cutlass::reference::host::TensorFill(scale_D.host_view(), options.scale_d);
    cutlass::reference::host::TensorFill(scale_aux.host_view(), options.scale_aux);

    scalar_alpha.sync_device();
    scalar_beta.sync_device();
    scale_A.sync_device();
    scale_B.sync_device();
    scale_C.sync_device();
    scale_D.sync_device();
    scale_aux.sync_device();
  }

  if (IsDFp8 && options.save_amax) {
    abs_max_D.resize(cutlass::make_Coord(1));
    abs_max_D.sync_device();
    reference_abs_max_D.resize(cutlass::make_Coord(1));
  }

  if (IsAuxFp8 && options.save_aux && options.save_amax) {
    abs_max_aux.resize(cutlass::make_Coord(1));
    abs_max_aux.sync_device();
    reference_abs_max_aux.resize(cutlass::make_Coord(1));
  }
}

```

**EN**: `initialize_tensor` selects data ranges based on element precision, and `initialize` allocates/resizes all host tensors, computes packed strides including the batch-like `L` mode, fills base inputs, syncs them to device memory, and conditionally allocates aux/scalar/amax tensors depending on the option set. This conditional allocation logic mirrors the compile-time fusion traits with runtime enable/disable switches.
**CN**: `initialize_tensor` 根据元素精度选择填充值范围，而 `initialize` 则负责分配/调整所有主机张量、计算包含批次式 `L` 维度在内的 packed stride、填充基础输入并同步到设备，同时依据选项集按需分配 aux、标量和 amax 张量。这种条件分配逻辑正好对应了编译期融合 traits 在运行时的开启/关闭开关。

### Lines 355-408

```cpp
/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options<RasterOrderOptions> &options)
{
  typename Gemm::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {tensor_A.device_data(), stride_A, tensor_B.device_data(), stride_B},
    {
      {}, // epilogue.thread
      tensor_C.device_data(), stride_C,
      tensor_D.device_data(), stride_D
    }
  };

  auto &fusion_args = arguments.epilogue.thread;
  fusion_args.alpha = options.alpha;
  fusion_args.beta = options.beta;
  fusion_args.alpha_ptr = scalar_alpha.device_data();
  fusion_args.beta_ptr = scalar_beta.device_data();
  fusion_args.scale_a = options.scale_a;
  fusion_args.scale_b = options.scale_b;
  fusion_args.scale_c = options.scale_c;
  fusion_args.scale_a_ptr = scale_A.device_data();
  fusion_args.scale_b_ptr = scale_B.device_data();
  fusion_args.scale_c_ptr = scale_C.device_data();

  // ignored if tensor types are not fp8
  fusion_args.scale_d = options.scale_d;
  fusion_args.scale_aux = options.scale_aux;
  fusion_args.scale_d_ptr = scale_D.device_data();
  fusion_args.scale_aux_ptr = scale_aux.device_data();

  // leaving/setting these as nullptr disables the fusion at runtime
  fusion_args.bias_ptr = nullptr;

  if (options.save_aux) {
    fusion_args.aux_ptr = tensor_aux.device_data();
    fusion_args.dAux = stride_aux;
    if (options.save_amax) {
      fusion_args.amax_aux_ptr = abs_max_aux.device_data();
    }
  }

  if (options.save_amax) {
    fusion_args.amax_D_ptr = abs_max_D.device_data();
  }

  arguments.scheduler.raster_order = options.raster;
  // The tile scheduler will swizzle up to 8 and with the nearest multiple of 2 (i.e., 1, 2, 4, and 8)
  arguments.scheduler.max_swizzle_size = options.swizzle;

  return arguments;
}

```

**EN**: `args_from_options` builds the `Gemm::Arguments` object and then populates `arguments.epilogue.thread`, which is where the fused FP8 epilogue is configured. Scalar values and their device pointers are both supplied, scale factors for A/B/C/D/Aux are attached, and setting optional pointers such as `bias_ptr`, `aux_ptr`, or `amax_*_ptr` to `nullptr` disables those features at runtime without changing the compiled kernel. Scheduler rasterization and swizzle are also attached here.
**CN**: `args_from_options` 会构造 `Gemm::Arguments`，随后填充 `arguments.epilogue.thread`，而这里正是融合 FP8 epilogue 的运行时配置入口。代码同时提供标量值及其设备指针，挂接 A/B/C/D/Aux 的缩放因子，并通过把 `bias_ptr`、`aux_ptr` 或 `amax_*_ptr` 等可选指针设为 `nullptr`，在不改变已编译 kernel 的情况下于运行时关闭对应功能。调度器的 rasterization 与 swizzle 参数也在此处完成绑定。

### Lines 409-479

```cpp
bool verify(const Options<RasterOrderOptions> &options) {
  //
  // Compute reference output
  //

  // Create instantiation for device reference gemm kernel
  auto A = cute::make_tensor(tensor_A.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.k, options.l), stride_A));
  auto B = cute::make_tensor(tensor_B.host_data(),
      cute::make_layout(cute::make_shape(options.n, options.k, options.l), stride_B));
  auto C = cute::make_tensor(tensor_C.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_C));
  auto D = cute::make_tensor(tensor_ref_D.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_D));
  auto Aux = cute::make_tensor(tensor_ref_aux.host_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_aux));
  using unused_t = decltype(D);

  cutlass::reference::host::GettMainloopParams<ElementAccumulator, decltype(A), decltype(B)> mainloop_params{A, B};

  cutlass::reference::host::GettEpilogueParams<
      ElementScalar,
      ElementScalar,
      ElementAccumulator,
      ElementCompute,
      decltype(C),
      decltype(D),
      unused_t, // bias
      decltype(Aux),
      unused_t, // valpha
      unused_t, // vbeta
      ActivationFunctor
  > epilogue_params;

  epilogue_params.C = C;
  epilogue_params.D = D;
  epilogue_params.Aux = Aux;
  epilogue_params.alpha = options.alpha;
  epilogue_params.beta = options.beta;
  epilogue_params.scale_a = options.scale_a;
  epilogue_params.scale_b = options.scale_b;
  epilogue_params.scale_c = options.scale_c;
  epilogue_params.scale_d = options.scale_d;
  epilogue_params.scale_aux = options.scale_aux;
  epilogue_params.abs_max_D = reference_abs_max_D.host_data();
  epilogue_params.abs_max_Aux = reference_abs_max_aux.host_data();

  // get reference result
  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);

  // compare_reference
  tensor_D.sync_host();
  bool passed = cutlass::reference::host::TensorEquals(tensor_ref_D.host_view(), tensor_D.host_view());

  if (IsDFp8 && options.save_amax) {
    abs_max_D.sync_host();
    passed &= abs_max_D.at(cutlass::make_Coord(0)) == reference_abs_max_D.at(cutlass::make_Coord(0));
  }

  if (options.save_aux) {
    tensor_aux.sync_host();
    passed &= cutlass::reference::host::TensorEquals(tensor_ref_aux.host_view(), tensor_aux.host_view());
    if (IsAuxFp8 && options.save_amax) {
      abs_max_aux.sync_host();
      passed &= abs_max_aux.at(cutlass::make_Coord(0)) == reference_abs_max_aux.at(cutlass::make_Coord(0));
    }
  }

  return passed;
}

```

**EN**: `verify` computes a host reference using CUTLASS GETT/Gemm3x helpers. It builds tensor views for A/B/C/D/Aux, fills `GettMainloopParams` and `GettEpilogueParams` with the same scaling/activation/amax configuration as the device kernel, runs `Gemm3x`, and then compares D, aux, and any requested amax values. This is essential because FP8 correctness is defined by the fused epilogue contract, not by plain GEMM alone.
**CN**: `verify` 借助 CUTLASS 的 GETT/Gemm3x 辅助组件在主机端计算参考结果。它为 A/B/C/D/Aux 构建张量视图，把与设备 kernel 相同的缩放/激活/amax 配置填入 `GettMainloopParams` 和 `GettEpilogueParams`，运行 `Gemm3x`，随后比较 D、aux 以及按需请求的 amax 值。对于 FP8 来说，这一步尤其关键，因为正确性是由“融合后的 epilogue 契约”定义的，而不只是普通 GEMM。

### Lines 480-550

```cpp
/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options<RasterOrderOptions> &options)
{
  initialize(options);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options(options);

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
  Result result;
  result.passed = verify(options);

  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;

  if (!result.passed) {
    exit(-1);
  }

  // Run profiling loop
  if (options.iterations > 0)
  {
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(gemm.run());
    }
    timer.stop();

    // Compute average runtime and GFLOPs.
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);

    std::string raster = "Heuristic";

    if (options.raster == RasterOrderOptions::AlongN) {
      raster = "Along N";
    }
    else if (options.raster == RasterOrderOptions::AlongM) {
      raster = "Along M";
    }

    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << 'x' << options.l << std::endl;
    std::cout << "  Rasterization: " << raster << " with a maximum CTA swizzle of " << options.swizzle << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```

**EN**: `run` follows the standard CUTLASS example flow—initialize, build arguments, allocate workspace, check implementability, initialize, run, verify, and benchmark—but the measured kernel now represents a much richer async pipeline: Hopper TMA+GMMA cooperative mainloop plus a fusion-heavy epilogue configured at runtime through thread-op arguments.
**CN**: `run` 遵循 CUTLASS 示例的标准流程：初始化、构造参数、分配 workspace、检查可实现性、初始化、运行、验证与基准测试；但此时被测的 kernel 已经代表了一个更复杂的异步流水线：前端是 Hopper 的 cooperative TMA+GMMA 主循环，后端则是通过线程级参数在运行时配置的高融合度 epilogue。

### Lines 551-602

```cpp

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

  Options<RasterOrderOptions> options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  //
  // Evaluate CUTLASS kernels
  //

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
  run<Gemm>(options);
#endif

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: `main` performs the usual CUDA 12 / SM90 checks, parses the richer FP8 option set, and dispatches the pre-instantiated `Gemm` type. As in other examples, unsupported systems return success so the example can live safely in cross-platform CI.
**CN**: `main` 执行常规的 CUDA 12 / SM90 检查，解析更丰富的 FP8 参数集，并调用预实例化好的 `Gemm` 类型。与其他示例一样，不支持的平台会直接返回成功，从而保证它能安全地存在于跨平台 CI 中。

---

## Key Concepts / 关键概念

**EN**: This example is all about compile-time and runtime layering: compile-time builders choose the cooperative Hopper TMA+GMMA pipeline, while runtime epilogue-thread arguments switch individual FP8 fusions on or off.
  **CN**: 该示例的核心在于“编译期 + 运行时”的双层配置：编译期 builder 选择 cooperative Hopper TMA+GMMA 流水线，运行时 epilogue-thread 参数再开启或关闭具体的 FP8 融合功能。
**EN**: The epilogue is more complex than the mainloop interface suggests: it may scale outputs, produce an auxiliary tensor, apply activation, and record amax for later quantization steps.
  **CN**: Epilogue 的复杂度远高于主循环接口表面所显示的内容：它可能缩放输出、生成辅助张量、应用激活函数，并记录 amax 供后续量化步骤使用。
**EN**: `StageCountAutoCarveout` is especially important in FP8 kernels because fusion-heavy epilogues can consume significant shared memory, directly affecting mainloop staging depth.
  **CN**: 在 FP8 kernel 中，`StageCountAutoCarveout` 尤其重要，因为高融合度 epilogue 可能占用大量共享内存，直接影响主循环的 staging 深度。
**EN**: Verification must reproduce both the math and the fused metadata paths (aux/amax/scales), otherwise an FP8 kernel can appear correct while violating its quantization contract.
  **CN**: 验证阶段必须同时复现数学结果与融合后的元数据路径（aux/amax/scale），否则某个 FP8 kernel 即使数值上看似正确，也可能违背其量化契约。

## Dependencies / 依赖项

**EN**: `hopper_fp8_commandline.hpp` provides the richer option object used to control scaling, auxiliary output, and amax behavior.
  **CN**: `hopper_fp8_commandline.hpp` 提供了更丰富的选项对象，用于控制缩放、辅助输出与 amax 行为。
**EN**: CUTLASS collective builders define the cooperative SM90 mainloop and epilogue, while `cutlass/epilogue/fusion/...` types are pulled in indirectly via the chosen fusion operation.
  **CN**: CUTLASS collective builder 负责定义 cooperative SM90 主循环与 epilogue，而 `cutlass/epilogue/fusion/...` 相关类型则通过所选的 fusion operation 间接引入。
**EN**: `cutlass/util/reference/host/gett.hpp` and related host reference helpers provide the CPU-side fused reference computation.
  **CN**: `cutlass/util/reference/host/gett.hpp` 及相关主机参考辅助组件提供了 CPU 侧的融合参考计算。
