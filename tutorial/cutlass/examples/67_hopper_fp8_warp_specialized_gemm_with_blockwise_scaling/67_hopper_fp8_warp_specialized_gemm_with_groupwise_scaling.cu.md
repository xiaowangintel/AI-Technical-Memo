# 67_hopper_fp8_warp_specialized_gemm_with_groupwise_scaling.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling/67_hopper_fp8_warp_specialized_gemm_with_groupwise_scaling.cu`  
**Purpose / 用途**: Demonstrates a CUTLASS Hopper FP8 GEMM that combines groupwise/blockwise scaling, TMA-driven warp-specialized execution, and a cooperative epilogue for benchmarking and verification. / 演示一个基于 CUTLASS 的 Hopper FP8 GEMM 示例，将 groupwise/blockwise 缩放、TMA 驱动的 warp-specialized 执行以及协作式 epilogue 结合起来，用于性能测试与正确性验证。

---

## Line-by-Line Analysis / 逐行分析

**Lines 1-31 / 第 1-31 行**

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

```
**EN**: BSD-3-Clause license banner and file prologue. It is not part of the kernel logic, but it frames the example as official CUTLASS sample code.
**CN**: 这是 BSD-3-Clause 许可证与文件前言，不属于内核逻辑，但说明该示例是官方 CUTLASS 示例代码。

**Lines 32-53 / 第 32-53 行**

```cpp
/*! \file
    \brief Grouped scale Hopper FP8 GEMM example using CUTLASS 3.0 APIs for NVIDIA Hopper architecture
    This example demonstrate a grouped scaled FP8 GEMM using the new CUTLASS 3.0.
    APIs on NVIDIA Hopper architecture. New features that will be showcased in this example are as follows:
    1. NVIDIA Hopper architecture introduces a new series of tensor core instructions (GMMA)
    which are more efficient than the Ampere tensor core instructions.
    2. NVIDIA Hopper architecture includes new Tensor Memory Accelerator (TMA) unit to transfer large
    blocks of data efficiently between global memory and shared memory. TMA also supports asynchronous
    copies between thread blocks in a cluster.
    3. This example uses the Warp Specialized kernel design (see /media/docs/efficient_gemm.md for details).
    4. This example shows all important fusions used by FP8 gemm kernels, i.e., grouped scale factor along M for
    A, blocked scale factor along K for A tensor, blocked scale factor for B tensor.
    5. A simple way to tune the CTA rasterization direction and swizzle pattern of Hopper kernels. Both the
    CTA rasterization direction and swizzle pattern impact cross-CTA locality of accesses. By tuning we can
    improve performance.
    Examples:
      $ ./examples/67_hopper_fp8_warp_specialized_gemm_with_blockwise_scaling/67_hopper_fp8_warp_specialized_gemm_with_groupwise_scaling  \
        --m=2816 --n=3072 --k=16384 \
        --save_aux=false --save_amax=false \
        --device_scale=false --raster=h --swizzle=2
*/

```
**EN**: The file comment states the goal: a Hopper FP8 GEMM using CUTLASS 3.x, highlighting GMMA/WGMMA-style tensor-core execution, TMA transfers, warp specialization, fused scaling, and scheduler tuning knobs such as raster order and swizzle.
**CN**: 文件注释说明了目标：基于 CUTLASS 3.x 的 Hopper FP8 GEMM，重点展示 GMMA/WGMMA 风格张量核心执行、TMA 传输、warp specialization、融合缩放，以及 raster 顺序和 swizzle 等调度调优参数。

**Lines 54-84 / 第 54-84 行**

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
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gett.hpp"

// Includes from examples directory
#include "helper.h"
#include "hopper_fp8_commandline.hpp"

```
**EN**: This include set pulls in CUTLASS/CuTe tensor abstractions, GEMM collective builders, epilogue builders, reference utilities, and example-local helpers for command-line parsing. Together they provide the full stack from Hopper kernel construction to correctness checking.
**CN**: 这一组头文件引入了 CUTLASS/CuTe 张量抽象、GEMM collective builder、epilogue builder、参考实现工具，以及示例目录中的命令行辅助代码，从 Hopper 内核构造到正确性校验形成完整链路。

**Lines 85-88 / 第 85-88 行**

```cpp
using namespace cute;

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

```
**EN**: The code adopts the CuTe namespace and immediately gates the Hopper-specific implementation behind SM90 support, so unsupported builds do not instantiate these kernels.
**CN**: 代码使用 CuTe 命名空间，并立刻用 SM90 条件编译保护 Hopper 专用实现，从而避免在不支持的构建目标上实例化这些内核。

**Lines 89-102 / 第 89-102 行**

```cpp
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

```
**EN**: Operand A and B are FP8 e4m3 tensors. A is row-major and B is column-major, while both alignments are chosen as 128 bits worth of elements to match efficient vectorized memory access for Hopper tensor-core kernels.
**CN**: 操作数 A 和 B 都是 FP8 e4m3 张量。A 采用行主序、B 采用列主序；两者的对齐都按 128 bit 对应的元素数计算，以匹配 Hopper 张量核心内核高效的向量化访存方式。

**Lines 103-111 / 第 103-111 行**

```cpp
// C matrix configuration
using         ElementC    = void;                                           // Element type for C matrix operand
using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C matrix operand
constexpr int AlignmentC  = 1;                                              // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// D matrix configuration
using         ElementD    = cutlass::bfloat16_t;                            // Element type for D matrix operand
using         LayoutD     = cutlass::layout::RowMajor;                      // Layout type for D matrix operand
constexpr int AlignmentD  = 128 / cutlass::sizeof_bits<ElementD>::value;    // Memory access granularity/alignment of D matrix in units of elements (up to 16 bytes)
```
**EN**: The example does not use a source C tensor in the epilogue, so ElementC is void and AlignmentC is 1. The output D tensor is BF16 in row-major form, showing a common mixed-precision path: FP8 inputs, FP32 accumulation, BF16 output.
**CN**: 该示例在 epilogue 中不使用源张量 C，因此 ElementC 被设为 void，AlignmentC 为 1。输出 D 为行主序 BF16，体现了常见的混合精度路径：FP8 输入、FP32 累加、BF16 输出。

**Lines 112-141 / 第 112-141 行**

```cpp

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ElementBlockScale   = float;                                          // Element type for blockscaling during accumulation
using ElementCompute      = float;                                          // Element type for epilogue computation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_256,_128,_128>;                          // Threadblock-level tile size
using ClusterShape        = Shape<_1,_2,_1>;                                // Shape of the threadblocks in a cluster

constexpr int ScaleGranularityM = 1;
constexpr int ScaleGranularityN = 128;
constexpr int ScaleGranularityK = 128;

constexpr int ScaleMsPerTile = size<0>(TileShape{}) / ScaleGranularityM;
constexpr int ScaleNsPerTile = size<1>(TileShape{}) / ScaleGranularityN;

using ScaleConfig   = cutlass::detail::Sm90BlockwiseScaleConfig<ScaleGranularityM, ScaleGranularityN, ScaleGranularityK, cute::GMMA::Major::MN, cute::GMMA::Major::K>;

using LayoutSFA             = decltype(ScaleConfig::deduce_layoutSFA());                     // Layout type for SFA matrix operand
using LayoutSFB             = decltype(ScaleConfig::deduce_layoutSFB());                     // Layout type for SFB matrix operand

using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8Blockwise; 
using EpilogueSchedule    = cutlass::epilogue::TmaWarpSpecializedCooperative;

using EpilogueTileType    = Shape<_128,_128>;
using FusionOperation     = cutlass::epilogue::fusion::Sm90EVT<
        cutlass::epilogue::fusion::Sm90AccFetch
      >;

```
**EN**: This block defines the Hopper kernel policy: SM90 tensor-op execution, a 256x128x128 threadblock tile, a 1x2x1 cluster, FP32 accumulation/block-scale/compute types, and scale granularities M=1, N=128, K=128. The scale configuration encodes groupwise scaling along M for A plus blockwise scaling along K for A and along tiled regions for B. KernelSchedule selects a TMA warp-specialized cooperative FP8 blockwise mainloop, while the epilogue uses a cooperative TMA schedule with a minimal accumulator-fetch fusion tree.
**CN**: 这一段定义了 Hopper 内核策略：SM90 Tensor Core 执行、256x128x128 的 CTA tile、1x2x1 的 cluster、FP32 累加/块缩放/计算类型，以及 M=1、N=128、K=128 的缩放粒度。该缩放配置表达了 A 沿 M 的 groupwise scaling，以及 A 沿 K 和 B 对应分块区域上的 blockwise scaling。KernelSchedule 选择基于 TMA 的 cooperative warp-specialized FP8 blockwise 主循环，而 epilogue 使用 cooperative TMA 调度，并采用最简的 accumulator-fetch 融合树。

**Lines 142-171 / 第 142-171 行**

```cpp
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

using CollectiveMainloopWithBlockWiseScaling = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementA, cute::tuple<LayoutA, LayoutSFA>, AlignmentA,
    ElementB, cute::tuple<LayoutB, LayoutSFB>, AlignmentB,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))
    >,
    KernelSchedule
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloopWithBlockWiseScaling,
    CollectiveEpilogue
>;

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
```
**EN**: CUTLASS collective builders assemble the two major kernel halves. CollectiveEpilogue chooses the SM90 epilogue implementation and its shared-memory footprint; CollectiveMainloopWithBlockWiseScaling builds the Hopper mainloop that consumes FP8 inputs plus scale tensors. StageCountAutoCarveout automatically sizes the async pipeline stages after reserving shared memory for the epilogue, which is central to overlapping TMA copies and WGMMA-style math. The resulting kernel is wrapped in the universal device adapter.
**CN**: CUTLASS 的 collective builder 在这里拼装出内核的两大部分。CollectiveEpilogue 选择 SM90 epilogue 实现及其共享内存占用；CollectiveMainloopWithBlockWiseScaling 构建消费 FP8 输入与缩放张量的 Hopper 主循环。StageCountAutoCarveout 会在为 epilogue 预留共享内存后自动决定异步流水线 stage 数，这对重叠 TMA 拷贝与 WGMMA 风格计算非常关键。最终内核再由通用 device adapter 封装。

**Lines 172-203 / 第 172-203 行**

```cpp

// Extract information from Gemm kernel.
using EpilogueOutputOp  = typename Gemm::EpilogueOutputOp;
using ElementScalar     = ElementCompute;

using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideC = typename Gemm::GemmKernel::StrideC;
using StrideD = typename Gemm::GemmKernel::StrideD;

static_assert(cute::is_same_v<ElementAccumulator, ElementBlockScale>,
             "ElementAccumulator and ElementBlockScale should be same datatype");

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;
LayoutSFA layout_SFA;
LayoutSFB layout_SFB;
uint64_t seed;

using LayoutScalar = cutlass::layout::PackedVectorLayout;
cutlass::HostTensor<ElementA  , LayoutA  > tensor_A;
cutlass::HostTensor<ElementB  , LayoutB  > tensor_B;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_D;
cutlass::HostTensor<ElementBlockScale, LayoutScalar> blockscale_tensor_A;
cutlass::HostTensor<ElementBlockScale, LayoutScalar> blockscale_tensor_B;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_ref_D;


#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```
**EN**: Here the program extracts convenient aliases such as strides and epilogue output type, checks that accumulation and block-scale math use the same scalar type, and allocates global HostTensor objects plus runtime layout state. These globals make it easy for initialization, verification, and execution helpers to share the same buffers.
**CN**: 这里提取了 stride、epilogue 输出类型等便捷别名，静态检查累加与块缩放计算是否使用同一标量类型，并定义全局 HostTensor 与运行期布局状态。这样初始化、校验和执行辅助函数就能共享同一组缓冲区。

**Lines 204-230 / 第 204-230 行**

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
**EN**: After the first SM90 block closes, the file defines a reusable raster-order enum alias and a Result struct that stores runtime, throughput, CUTLASS/CUDA status, and pass/fail state. This separates generic benchmarking metadata from Hopper-only kernel code.
**CN**: 在第一个 SM90 条件编译块结束后，文件定义了可复用的 raster 顺序枚举别名，以及保存运行时间、吞吐率、CUTLASS/CUDA 状态和是否通过校验的 Result 结构体。这样可把通用基准测试元数据与 Hopper 专用内核代码分离。

**Lines 231-237 / 第 231-237 行**

```cpp

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM setup and evaluation
/////////////////////////////////////////////////////////////////////////////////////////////////

```
**EN**: A second SM90 guard begins for the executable path that prepares data, launches the GEMM, and validates results.
**CN**: 第二个 SM90 条件编译块开始，覆盖数据准备、GEMM 启动与结果校验等可执行路径。

**Lines 238-311 / 第 238-311 行**

```cpp
/// Helper to initialize a block of device data
template <typename Element, typename Layout>
bool initialize_tensor(
  cutlass::TensorView<Element, Layout> view,
  cutlass::Distribution::Kind dist_kind,
  uint64_t seed,
  bool is_device_tensor = false) {

  if (dist_kind == cutlass::Distribution::Uniform) {

    double scope_max, scope_min;
    int bits_input = cutlass::sizeof_bits<Element>::value;
    int bits_output = cutlass::sizeof_bits<Element>::value;

    if (bits_input == 1) {
      scope_max = 2;
      scope_min = 0;
    } else if (bits_input <= 8) {
      scope_max = 2;
      scope_min = -2;
    } else if (bits_output == 16) {
      scope_max = 5;
      scope_min = -5;
    } else {
      scope_max = 8;
      scope_min = -8;
    }

    if (is_device_tensor) {
      using Real = typename cutlass::RealType<Element>::Type;
      cutlass::reference::device::TensorFillRandomUniform(
        view, seed, static_cast<Real>(scope_max), static_cast<Real>(scope_min), bits_input);
    } else {
      cutlass::reference::host::TensorFillRandomUniform(
        view, seed, scope_max, scope_min, bits_input);
    }

  }
  else if (dist_kind == cutlass::Distribution::AllZeros) {
    if (is_device_tensor) {
      cutlass::reference::device::TensorFill(view);
    } else {
      cutlass::reference::host::TensorFill(view);
    }
  }
  else if (dist_kind == cutlass::Distribution::Identity) {

    if (is_device_tensor) {
      cutlass::reference::device::TensorFillIdentity(view);
    } else {
      cutlass::reference::host::TensorFillIdentity(view);
    }
  }
  else if (dist_kind == cutlass::Distribution::Gaussian) {
    if (is_device_tensor) {
      using Real = typename cutlass::RealType<Element>::Type;
      cutlass::reference::device::TensorFillRandomGaussian(view, seed, static_cast<Real>(0), static_cast<Real>(0.5));
    } else {
      cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
    }
  }
  else if (dist_kind == cutlass::Distribution::Sequential) {
    if (is_device_tensor) {
      cutlass::reference::device::BlockFillSequential(view.data(), view.capacity());
    } else {
      cutlass::reference::host::BlockFillSequential(view.data(), view.capacity());
    }
  }
  else {
    throw std::runtime_error("Not implementated.");
  }

  return true;
}
```
**EN**: initialize_tensor is a generic filler for dense tensors. It supports uniform, zero, identity, Gaussian, and sequential patterns on either host or device memory, choosing numeric ranges based on element bit width. For FP8 inputs this keeps random values within a small dynamic range, which is important for stable reference comparison.
**CN**: initialize_tensor 是通用张量初始化函数。它支持 uniform、zero、identity、Gaussian 和 sequential 等分布，并可在 host 或 device 端填充；数值范围会根据元素位宽调整。对于 FP8 输入，这能把随机值限制在较小动态范围内，有助于稳定地进行参考结果对比。

**Lines 312-373 / 第 312-373 行**

```cpp

/// Helper to initialize a block of device data (scale_tensors)
template <typename Element, typename Layout>
bool initialize_scale_tensor(
  cutlass::TensorView<Element, Layout> view,
  cutlass::Distribution::Kind dist_kind,
  uint64_t seed,
  bool is_device_tensor = false) {

  if (dist_kind == cutlass::Distribution::Uniform) {

    double scope_max, scope_min;

    scope_min = -1;
    scope_max = 1;

    if (is_device_tensor) {
      using Real = typename cutlass::RealType<Element>::Type;
      cutlass::reference::device::TensorFillRandomUniform(
        view, seed, static_cast<Real>(scope_max), static_cast<Real>(scope_min));  
    } else {
      cutlass::reference::host::TensorFillRandomUniform(
        view, seed, scope_max, scope_min);
    }
  }
  else if (dist_kind == cutlass::Distribution::AllZeros) {
    if (is_device_tensor) {
      cutlass::reference::device::TensorFill(view);
    } else {
      cutlass::reference::host::TensorFill(view);
    }
  }
  else if (dist_kind == cutlass::Distribution::Identity) {

    if (is_device_tensor) {
      cutlass::reference::device::TensorFillIdentity(view);
    } else {
      cutlass::reference::host::TensorFillIdentity(view);
    }
  }
  else if (dist_kind == cutlass::Distribution::Gaussian) {

    if (is_device_tensor) {
      using Real = typename cutlass::RealType<Element>::Type;
      cutlass::reference::device::TensorFillRandomGaussian(view, seed, static_cast<Real>(0), static_cast<Real>(0.5));
    } else {
      cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
    }
  }
  else if (dist_kind == cutlass::Distribution::Sequential) {
    if (is_device_tensor) {
      cutlass::reference::device::BlockFillSequential(view.data(), view.capacity());
    } else {
      cutlass::reference::host::BlockFillSequential(view.data(), view.capacity());
    }
  }
  else {
    throw std::runtime_error("Not implementated.");
  }

  return true;
}
```
**EN**: initialize_scale_tensor mirrors the previous helper but narrows the uniform range to [-1, 1], which is more appropriate for multiplicative scale factors. The same host/device and distribution branching is preserved so scale tensors can be generated consistently with operand tensors.
**CN**: initialize_scale_tensor 与前一个辅助函数类似，但将 uniform 范围收窄到 [-1, 1]，更适合作为乘性缩放因子。它保留了相同的 host/device 与分布分支逻辑，因此缩放张量可与操作数张量一致地生成。

**Lines 374-389 / 第 374-389 行**

```cpp

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options<RasterOrderOptions> &options) {

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.m, options.n, options.l));
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));

  // Layout SFA and SFB represent logically broadcasting data in CuTe.
  // E.g., if Layout SFA has shape ((ScaleGranularityM, M / ScaleGranularityM), (ScaleGraunularityK, K / ScaleGranularityK))
  // and strides ((0, 1), (0, M / ScaleGraunuarlityM)), then each collection of ScaleGranularityM x ScaleGranularityK
  // indices in the tensor map to the same offset.

  layout_SFA = ScaleConfig::tile_atom_to_shape_SFA(make_shape(options.m, options.n, options.k, options.l));
  layout_SFB = ScaleConfig::tile_atom_to_shape_SFB(make_shape(options.m, options.n, options.k, options.l));
```
**EN**: initialize first computes packed CuTe strides for A/B/C/D from the runtime problem size. It then derives layout_SFA and layout_SFB from ScaleConfig. The comments are important: these layouts are broadcast-style logical maps, so many tensor indices intentionally point to the same scale value, implementing grouped/blockwise scaling without materializing a full dense scale tensor.
**CN**: initialize 首先根据运行期问题规模为 A/B/C/D 计算紧凑的 CuTe stride。随后通过 ScaleConfig 推导 layout_SFA 和 layout_SFB。这里的注释非常关键：这些布局是带广播语义的逻辑映射，因此多个张量索引会故意对应同一个缩放值，从而在不显式构造稠密缩放张量的情况下实现 groupwise/blockwise scaling。

**Lines 390-428 / 第 390-428 行**

```cpp

  auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
  auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
  auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);
  auto groupscale_a_coord = cutlass::make_Coord(size(filter_zeros(layout_SFA)));
  auto groupscale_b_coord = cutlass::make_Coord(size(filter_zeros(layout_SFB)));

  tensor_A.resize(a_coord);
  blockscale_tensor_A.resize(groupscale_a_coord);
  tensor_B.resize(b_coord);
  blockscale_tensor_B.resize(groupscale_b_coord);
  tensor_D.resize(c_coord);
  tensor_ref_D.resize(c_coord);

  cutlass::Distribution::Kind dist_A = cutlass::Distribution::Gaussian;
  cutlass::Distribution::Kind dist_B = cutlass::Distribution::Gaussian;
  cutlass::Distribution::Kind dist_scaleA = cutlass::Distribution::Gaussian;
  cutlass::Distribution::Kind dist_scaleB = cutlass::Distribution::Gaussian;

  initialize_tensor(tensor_A.device_view(), dist_A, seed + 2022, true);
  initialize_tensor(tensor_B.device_view(), dist_B, seed + 2023, true);
  initialize_scale_tensor(blockscale_tensor_A.device_view(), dist_scaleA, seed + 2025, true);
  initialize_scale_tensor(blockscale_tensor_B.device_view(), dist_scaleB, seed + 2026, true);

  tensor_A.sync_host();
  tensor_B.sync_host();
  tensor_D.sync_host();
  blockscale_tensor_A.sync_host();
  blockscale_tensor_B.sync_host();

#if 0 // Dump blockscaled tensors
  // Print block scaling tensors on the host side.
  std::cout << "blockscale_tensor_A: " << blockscale_a_coord << std::endl;
  std::cout << blockscale_tensor_A.host_view() << "\n";
  std::cout << "blockscale_tensor_B: " << blockscale_b_coord << std::endl;
  std::cout << blockscale_tensor_B.host_view() << "\n";
#endif

}
```
**EN**: The function next computes physical coordinates, resizes storage, chooses Gaussian initialization for both FP8 operands and scale tensors, performs device-side fills, and syncs data back to the host. The disabled debug print block can dump the compact scale tensors for inspection.
**CN**: 接着函数计算物理坐标、调整存储大小，为 FP8 操作数和缩放张量都选择 Gaussian 初始化，在 device 端完成填充，并把数据同步回 host。被禁用的调试打印块可用于检查压缩后的缩放张量。

**Lines 429-457 / 第 429-457 行**

```cpp

/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options<RasterOrderOptions> &options)
{
  typename Gemm::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {tensor_A.device_data(),
     stride_A,
     tensor_B.device_data(),
     stride_B,
     blockscale_tensor_A.device_data(),
     layout_SFA,
     blockscale_tensor_B.device_data(),
     layout_SFB
     },
    {
      {}, // epilogue.thread
      nullptr, stride_C,
      tensor_D.device_data(), stride_D
    }
  };

  arguments.scheduler.raster_order = options.raster;
  // The tile scheduler will swizzle up to 8 and with the nearest multiple of 2 (i.e., 1, 2, 4, and 8)
  arguments.scheduler.max_swizzle_size = options.swizzle;

  return arguments;
}
```
**EN**: args_from_options packages runtime values into Gemm::Arguments. Besides the usual problem size, data pointers, and strides, it passes scale tensor pointers plus their broadcast layouts, and it wires scheduler controls for raster direction and CTA swizzle. This is where the command-line tuning knobs become concrete kernel launch parameters.
**CN**: args_from_options 将运行期参数打包成 Gemm::Arguments。除了常规的问题规模、数据指针和 stride，它还传入缩放张量指针及其广播布局，并设置调度器的 raster 方向和 CTA swizzle。也就是说，命令行中的调优选项会在这里转化为实际的内核启动参数。

**Lines 458-538 / 第 458-538 行**

```cpp

bool verify(const Options<RasterOrderOptions> &options) {
  //
  // Compute reference output
  //

  // Create instantiation for device reference gemm kernel
  auto A = cute::make_tensor(tensor_A.host_data(),
                             cute::make_layout(
                                cute::make_shape(options.m, options.k, options.l),
                                stride_A
                              )
                            );
  auto B = cute::make_tensor(tensor_B.host_data(),
                             cute::make_layout(
                               cute::make_shape(options.n, options.k, options.l),
                               stride_B
                              )
                            );
  auto D = cute::make_tensor(tensor_ref_D.host_data(),
                             cute::make_layout(
                                cute::make_shape(options.m, options.n, options.l),
                                stride_D
                              )
                            );

  auto SFA = cute::make_tensor(blockscale_tensor_A.host_data(), layout_SFA);
  auto SFB = cute::make_tensor(blockscale_tensor_B.host_data(), layout_SFB);

  using unused_t = decltype(D);

  cutlass::reference::host::GettBlockScalingMainloopParams<
      ElementAccumulator,
      decltype(A),
      decltype(SFA),
      decltype(B),
      decltype(SFB)
    > mainloop_params{A, SFA, B, SFB};

  cutlass::reference::host::GettEpilogueParams<
      ElementScalar,
      ElementScalar,
      ElementAccumulator,
      ElementCompute,
      unused_t,  // C
      decltype(D),
      unused_t, // bias
      unused_t, // aux
      unused_t, // valpha
      unused_t/*, // vbeta
      ActivationFunctor*/
  > epilogue_params;

  epilogue_params.D = D;
  epilogue_params.alpha = 1.0f;
  epilogue_params.beta = 0.0f;

  // get reference result
  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);

  // compare_reference
  bool passed = true;
  tensor_D.sync_host();
  passed &= cutlass::reference::host::TensorRelativelyEquals(tensor_D.host_view(),
      tensor_ref_D.host_view(), ElementD(options.epsilon), ElementD(options.non_zero_floor));
  double mse = cutlass::reference::host::TensorMSE(tensor_D.host_view(), tensor_ref_D.host_view());
  double mre = cutlass::reference::host::TensorMRE(tensor_D.host_view(), tensor_ref_D.host_view());
  double max_error = cutlass::reference::host::TensorGreatestError(tensor_D.host_view(), tensor_ref_D.host_view());
  std::cout << "  Result MSE: " << mse << ", MRE: " << mre << ", greatest error: " << max_error << std::endl;

#if 0
  std::cout << "tensor_ref_D.host_view() {" << std::endl
            << tensor_ref_D.host_view() << std::endl
            << "}"  << std::endl;
  std::cout << "tensor_D.host_view() {" << std::endl
            << tensor_D.host_view() << std::endl
            << "}"  << std::endl;
#endif

  return passed;
}
```
**EN**: verify reconstructs CuTe tensor views on the host for A, B, D, SFA, and SFB, then feeds them into CUTLASS reference GettBlockScalingMainloopParams and GettEpilogueParams. The reference path uses Gemm3x to reproduce the same scaled mainloop plus epilogue math in software, with alpha=1 and beta=0 because there is no source C contribution. It then compares device and reference outputs and reports numerical error statistics.
**CN**: verify 在 host 端重新构造 A、B、D、SFA 和 SFB 的 CuTe tensor view，然后将其传入 CUTLASS 参考实现的 GettBlockScalingMainloopParams 与 GettEpilogueParams。该参考路径通过 Gemm3x 复现相同的带缩放主循环与 epilogue 计算；由于没有源 C 参与，因此 alpha=1、beta=0。随后它比较 device 输出与参考结果，并报告数值误差统计。

**Lines 539-569 / 第 539-569 行**

```cpp

/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options<RasterOrderOptions> &options)
{

  bool skip = false;
  std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << 'x' << options.l << std::endl;
  std::cout << "  Tile shape (M, N, K): " << size<0>(TileShape{}) << ", " << size<1>(TileShape{}) << ", " << size<2>(TileShape{}) << std::endl;
  std::cout << "  ScaleGranularityM: " << ScaleGranularityM << " (ScaleMsPerTile: " << ScaleMsPerTile << ")" << std::endl;
  std::cout << "  ScaleGranularityN: " << ScaleGranularityN << " (ScaleNsPerTile: " << ScaleNsPerTile << ")" << std::endl;


  if (options.m < ScaleGranularityM) {
    std::cout << "  Skippig (m size: " << options.m << " less than ScaleGranularityM: " << ScaleGranularityM << "):" << std::endl;
    skip = true;
  }

  if (options.n < ScaleGranularityN) {
    std::cout << "  Skippig (n size: " << options.n << " less than ScaleGranularityN: " << ScaleGranularityN << "):" << std::endl;
    skip = true;
  }

  if (options.k < size<2>(TileShape{})) {
    std::cout << "  Skippig (k size: " << options.k << " less than TileShape[2]: " << size<2>(TileShape{}) << "):" << std::endl;
    skip = true;
  }

  if (!skip) std::cout << "  Running... " << std::endl;
  else return -1;

```
**EN**: run begins by printing the problem shape, tile shape, and scaling granularities, then rejects cases that are too small for the configured scaling/tile policy. Those checks reflect real policy constraints: if M, N, or K are smaller than the assumed grouping/block tile sizes, the example deliberately skips execution.
**CN**: run 开始时先打印问题规模、tile 形状和缩放粒度，然后拒绝那些小于当前缩放/分块策略要求的输入。这些检查反映了真实的策略约束：若 M、N 或 K 小于假设的分组或 tile 大小，示例会主动跳过执行。

**Lines 570-603 / 第 570-603 行**

```cpp
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
  if (options.verify) {
    result.passed = verify(options);

    std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
  }
  else {
    result.passed = true;
  }

```
**EN**: Once the problem is accepted, the program initializes tensors, instantiates the Gemm object, materializes arguments, queries workspace size, allocates workspace, checks can_implement, initializes the kernel, performs a warmup run, and optionally verifies correctness.
**CN**: 当问题规模合法后，程序初始化张量、实例化 Gemm 对象、生成参数、查询 workspace 大小、分配 workspace、调用 can_implement 检查、初始化内核、执行预热运行，并可选地校验正确性。

**Lines 604-638 / 第 604-638 行**

```cpp
  // Run profiling loop
  if (options.iterations > 0)
  {
    GpuTimer timer;
    for (int iter = 0; iter < options.warmup + options.iterations; ++iter) {
      if (iter == options.warmup)
        timer.start();
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
    std::cout << "  GFLOPS: " << result.gflops << std::endl << std::endl << std::endl;
  }

  return result.passed;
}

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```
**EN**: The profiling loop measures only the post-warmup iterations. It computes average runtime and GFLOPS, translates raster-order enums into readable text, and prints the tuned scheduler choices. Conceptually, these numbers reflect how well the Hopper async pipeline, TMA-fed mainloop, and warp-specialized tensor-core work distribution perform for the chosen tile schedule.
**CN**: 性能分析循环只统计预热之后的迭代。它计算平均运行时间和 GFLOPS，将 raster 顺序枚举转换为可读文本，并打印调优后的调度策略。从概念上说，这些指标反映了在给定 tile 调度下，Hopper 异步流水线、TMA 驱动主循环以及 warp-specialized 张量核心工作分配的整体表现。

**Lines 639-650 / 第 639-650 行**

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
```
**EN**: The Hopper-specific section is over, and main starts with a CUDA toolkit version guard. Returning 0 on older toolkits keeps the example benign in unsupported environments instead of failing the whole test suite.
**CN**: 这里 Hopper 专用代码区已经结束，main 函数首先检查 CUDA 工具链版本。在旧版本工具链上返回 0，可以让示例在不支持的环境中安全跳过，而不是让整个测试失败。

**Lines 651-661 / 第 651-661 行**

```cpp

  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (props.major != 9) {
    std::cerr
      << "This example requires a GPU of NVIDIA's Hopper Architecture or "
      << "later (compute capability 90 or greater).\n";
    return 0;
```
**EN**: The program queries device properties and enforces compute capability 9.0+, i.e. Hopper or later. That is required because the example relies on SM90-only features such as TMA and the Hopper tensor-core execution model used by these collectives.
**CN**: 程序读取设备属性并强制要求计算能力 9.0 及以上，也就是 Hopper 或更新架构。这是因为示例依赖 SM90 独有特性，例如 TMA，以及这些 collective 所使用的 Hopper 张量核心执行模型。

**Lines 662-674 / 第 662-674 行**

```cpp
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
```
**EN**: Command-line options are parsed into the reusable Options object, and the help path prints usage text before exiting. This keeps the rest of main focused on execution rather than argument handling.
**CN**: 命令行参数被解析到可复用的 Options 对象中；若请求帮助，则打印用法并退出。这样 main 的剩余部分就可以专注于执行逻辑，而不是参数处理。

**Lines 675-689 / 第 675-689 行**

```cpp

  //
  // Evaluate CUTLASS kernels
  //

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
  bool passed = run<Gemm>(options);
  if (!passed)
    return -1;
#endif

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Finally, the program dispatches run<Gemm>(options) under SM90 support, propagates failure as a non-zero exit code, and otherwise returns success. The trailing separator comment is simply the file terminator style used across CUTLASS examples.
**CN**: 最后，程序在 SM90 支持条件下调用 run<Gemm>(options)，若失败则返回非零退出码，否则返回成功。末尾的分隔注释只是 CUTLASS 示例常用的文件结束风格。

---

## Key Concepts / 关键概念

- **TMA**: The selected mainloop and epilogue schedules are TMA-based, meaning Hopper can move tiles between global and shared memory asynchronously with low thread overhead. / **TMA**：所选主循环与 epilogue 调度都基于 TMA，使 Hopper 能以较低线程开销在全局内存与共享内存之间异步搬运 tile。
- **WGMMA / GMMA-style tensor-core math**: The SM90 tensor-op collectives map the FP8 GEMM onto Hopper warpgroup tensor-core instructions, which are the compute engine paired with the async data pipeline. / **WGMMA / GMMA 风格张量核心计算**：SM90 TensorOp collective 会把该 FP8 GEMM 映射到 Hopper 的 warpgroup 张量核心指令上，它与异步数据流水线共同构成核心计算路径。
- **Warp specialization**: The cooperative warp-specialized schedule separates data-movement and math responsibilities so TMA traffic and tensor-core execution can overlap more effectively. / **Warp specialization**：协作式 warp-specialized 调度将数据搬运与计算职责解耦，从而更有效地重叠 TMA 传输与张量核心执行。
- **Async pipeline staging**: `StageCountAutoCarveout` automatically balances shared memory between epilogue storage and the mainloop pipeline stages. / **异步流水线分级**：`StageCountAutoCarveout` 会在 epilogue 共享内存占用与主循环流水线 stage 数之间自动平衡。
- **Groupwise + blockwise scaling**: `ScaleGranularityM = 1` gives very fine-grained grouping along M for A, while `ScaleGranularityN/K = 128` defines blockwise scaling structure for the FP8 path. / **Groupwise + blockwise scaling**：`ScaleGranularityM = 1` 使 A 在 M 方向具有非常细粒度的分组缩放，而 `ScaleGranularityN/K = 128` 则定义了 FP8 路径中的分块缩放结构。
- **Epilogue policy**: `Sm90EVT<Sm90AccFetch>` keeps the epilogue simple: fetch accumulated values and write BF16 D, with no source C accumulation because `ElementC` is `void` and `beta = 0`. / **Epilogue 策略**：`Sm90EVT<Sm90AccFetch>` 让 epilogue 保持简洁：提取累加结果并写出 BF16 的 D；由于 `ElementC` 为 `void` 且 `beta = 0`，因此不会叠加源 C。

## Dependencies / 依赖项

- **CUDA / Hardware**: CUDA 12+ and an SM90 (Hopper) or newer GPU. / **CUDA / 硬件**：需要 CUDA 12+ 和 SM90（Hopper）或更新的 GPU。
- **CUTLASS core**: `cutlass/cutlass.h`, numeric types, GEMM collective builders, kernel adapters, dispatch policies, and epilogue builders. / **CUTLASS 核心**：`cutlass/cutlass.h`、数值类型、GEMM collective builder、kernel adapter、dispatch policy 与 epilogue builder。
- **CuTe**: `cute/tensor.hpp` and related layout/shape utilities for runtime tensor/view construction. / **CuTe**：`cute/tensor.hpp` 及相关布局/形状工具，用于运行期张量与视图构造。
- **Reference and utility helpers**: command-line parsing, tensor initialization, host/device reference filling, and `Gemm3x` verification helpers. / **参考实现与工具辅助**：命令行解析、张量初始化、host/device 参考填充，以及 `Gemm3x` 校验辅助函数。
- **Example-local headers**: `helper.h` and `hopper_fp8_commandline.hpp` provide timer/check macros and option definitions used by the sample harness. / **示例本地头文件**：`helper.h` 与 `hopper_fp8_commandline.hpp` 提供了计时/检查宏以及该示例驱动程序使用的选项定义。
