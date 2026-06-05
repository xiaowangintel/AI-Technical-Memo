# 48_hopper_warp_specialized_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/48_hopper_warp_specialized_gemm/48_hopper_warp_specialized_gemm.cu`  
**Purpose / 用途**: Demonstrates how to instantiate, verify, and benchmark a Hopper TF32 GEMM using CUTLASS 3.x collective builders and warp-specialized scheduling. / 演示如何使用 CUTLASS 3.x collective builder 与 warp-specialized 调度来实例化、验证并测试一个 Hopper TF32 GEMM。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-56

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
    \brief Simple Hopper GEMM example using CUTLASS 3.0 APIs for NVIDIA Hopper architecture

    This example demonstrate a simple way to instantiate and run a TF32 GEMM using the new CUTLASS 3.0
    APIs on NVIDIA Hopper architecture. New features that will be showcased in this example are as follows:

    1. NVIDIA Hopper architecture introduces a new series of tensor core instructions (GMMA)
    which are more efficient than the Ampere tensor core instructions.

    2. NVIDIA Hopper architecture includes new Tensor Memory Accelerator (TMA) unit to transfer large
    blocks of data efficiently between global memory and shared memory. TMA also supports asynchronous
    copies between thread blocks in a cluster. Another advantage is that TMA can load in FP32 data and
    convert them implicitly to TF32.

    3. This example uses the Warp Specialized kernel design (see /media/docs/efficient_gemm.md for details).

    4. A simple way to tune the CTA rasterization direction and swizzle pattern of Hopper kernels. Both the 
    CTA rasterization direction and swizzle pattern impact cross-CTA locality of accesses. By tuning we can 
    improve performance.

    Examples:

      $ ./examples/48_hopper_warp_specialized_gemm/48_hopper_warp_specialized_gemm --m=2048 --n=2048 --k=2048 --rasterization=N --swizzle=2
*/

```

**EN**: The banner comment explains the example’s role: it is a compact Hopper GEMM built with CUTLASS 3 APIs, highlighting GMMA, TMA, warp specialization, and scheduler tuning. Even though the file does not manually write pipeline code, it is explicitly about selecting those Hopper mechanisms through builder APIs.
**CN**: 开头注释说明了示例的定位：它是一个基于 CUTLASS 3 API 的紧凑 Hopper GEMM 示例，重点展示 GMMA、TMA、warp specialization 与调度器调优。虽然本文件没有手写流水线代码，但它明确展示了如何通过 builder API 选择这些 Hopper 机制。

### Lines 57-84

```cpp
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
#include "cutlass/gemm/kernel/tile_scheduler_params.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/device/tensor_fill.h"

#include "helper.h"

using namespace cute;

```

**EN**: The include list pulls in CUTLASS collective builders, a default epilogue, reference GEMM helpers, and command-line/timing utilities. `using namespace cute;` signals that the example uses CuTe shape and layout types as the configuration language for the kernel.
**CN**: 头文件列表引入了 CUTLASS collective builder、默认 epilogue、参考 GEMM 辅助实现以及命令行/计时工具。`using namespace cute;` 表明该示例使用 CuTe 的形状与布局类型作为 kernel 的配置语言。

### Lines 85-177

```cpp
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////

// A matrix configuration
using         ElementA    = float;                                          // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = float;                                          // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C/D matrix configuration
using         ElementC    = float;                                          // Element type for C and D matrix operands
using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_128,_128,_32>;                           // Threadblock-level tile size
using ClusterShape        = Shape<_4,_2,_1>;                                // Shape of the threadblocks in a cluster
using StageCountType = cutlass::gemm::collective::StageCountAuto;           // Stage count maximized based on the tile size
using KernelSchedule = cutlass::gemm::collective::KernelScheduleAuto;       // Kernel to launch based on the default setting in the Collective Builder

using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
    TileShape, ClusterShape,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ElementAccumulator, ElementAccumulator,
    ElementC, LayoutC, AlignmentC,
    ElementC, LayoutC, AlignmentC,
    cutlass::epilogue::collective::EpilogueScheduleAuto
  >::CollectiveOp;

using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    ElementA, LayoutA, AlignmentA,
    ElementB, LayoutB, AlignmentB,
    ElementAccumulator,
    TileShape, ClusterShape,
    cutlass::gemm::collective::StageCountAutoCarveout<
      static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
    cutlass::gemm::collective::KernelScheduleAuto
  >::CollectiveOp;

using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int>, // Indicates ProblemShape
    CollectiveMainloop,
    CollectiveEpilogue
>;

using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

// Reference device GEMM implementation type
using DeviceGemmReference = cutlass::reference::device::Gemm<
  ElementA,
  LayoutA,
  ElementB,
  LayoutB,
  ElementC,
  LayoutC,
  ElementAccumulator,
  ElementAccumulator>;

using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideC = typename Gemm::GemmKernel::StrideC;
using StrideD = typename Gemm::GemmKernel::StrideD;

//
// Data members
//

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;
uint64_t seed;

cutlass::DeviceAllocation<typename Gemm::ElementA> block_A;
cutlass::DeviceAllocation<typename Gemm::ElementB> block_B;
cutlass::DeviceAllocation<typename Gemm::ElementC> block_C;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_D;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_ref_D;

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```

**EN**: This block is the compile-time heart of the example. It chooses TF32-capable FP32 operands, sets the threadblock tile (`128x128x32`) and cluster (`4x2x1`), then uses `CollectiveBuilder` twice: once for the epilogue and once for the mainloop. `StageCountAutoCarveout` subtracts epilogue shared-memory usage before picking the mainloop stage count, and `KernelScheduleAuto` lets CUTLASS select the appropriate Hopper warp-specialized TMA+GMMA implementation. The rest of the block extracts a device reference GEMM type and the stride aliases used later.
**CN**: 这一段是示例的编译期核心。它选择支持 TF32 的 FP32 操作数，设置 threadblock tile（`128x128x32`）与 cluster（`4x2x1`），随后两次调用 `CollectiveBuilder`：一次构建 epilogue，一次构建 mainloop。`StageCountAutoCarveout` 会先扣除 epilogue 占用的共享内存，再选择主循环的 stage 数；`KernelScheduleAuto` 则让 CUTLASS 自动选出合适的 Hopper warp-specialized TMA+GMMA 实现。其余代码提取了设备参考 GEMM 类型以及后续要用的 stride 别名。

### Lines 178-289

```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

using RasterOrderOptions = typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90Params::RasterOrderOptions;

// Command line options parsing
struct Options {

  bool help;

  float alpha, beta;
  int iterations;
  int m, n, k;
  RasterOrderOptions raster;
  int swizzle;

  Options():
    help(false),
    m(5120), n(4096), k(4096),
    alpha(1.f), beta(0.f),
    iterations(1000),
    raster(RasterOrderOptions::Heuristic),
    swizzle(1)
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("m", m);
    cmd.get_cmd_line_argument("n", n);
    cmd.get_cmd_line_argument("k", k);
    cmd.get_cmd_line_argument("alpha", alpha, 1.f);
    cmd.get_cmd_line_argument("beta", beta, 0.f);
    cmd.get_cmd_line_argument("iterations", iterations);

    char raster_char;
    cmd.get_cmd_line_argument("raster", raster_char);

    if (raster_char == 'N' || raster_char == 'n') {
      raster = RasterOrderOptions::AlongN;
    }
    else if (raster_char == 'M' || raster_char == 'm') {
      raster = RasterOrderOptions::AlongM;
    }
    else if (raster_char == 'H' || raster_char == 'h') {
      raster = RasterOrderOptions::Heuristic;
    }

    cmd.get_cmd_line_argument("swizzle", swizzle, 1);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "48_hopper_warp_specialized_gemm\n\n"
      << "  Hopper FP32 GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --raster=<char>             CTA Rasterization direction (N for along N, M for along M, and H for heuristic)\n\n"
      << "  --swizzle=<int>             CTA Rasterization swizzle\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "48_hopper_warp_specialized_gemm" << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const
  {
    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * m * n * k;
    double gflop = double(flop) / double(1.0e9);
    return gflop / runtime_s;
  }
};

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

**EN**: `RasterOrderOptions`, `Options`, and `Result` define the runtime control plane. Users can change problem size, alpha/beta, rasterization direction, and swizzle, while `gflops()` turns elapsed time into performance metrics. These knobs ultimately configure the persistent tile scheduler embedded in the chosen SM90 kernel.
**CN**: `RasterOrderOptions`、`Options` 与 `Result` 共同构成运行时控制层。用户可以修改问题规模、alpha/beta、rasterization 方向以及 swizzle；`gflops()` 用于把耗时换算成性能指标。这些旋钮最终会配置被选中 SM90 kernel 内部的 persistent tile scheduler。

### Lines 290-340

```cpp
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM setup and evaluation
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Helper to initialize a block of device data
template <class Element>
bool initialize_block(
  cutlass::DeviceAllocation<Element>& block,
  uint64_t seed=2023) {

  Element scope_max, scope_min;
  int bits_input = cutlass::sizeof_bits<Element>::value;

  if (bits_input == 1) {
    scope_max = Element(2);
    scope_min = Element(0);
  } else if (bits_input <= 8) {
    scope_max = Element(2);
    scope_min = Element(-2);
  } else {
    scope_max = Element(8);
    scope_min = Element(-8);
  }

  cutlass::reference::device::BlockFillRandomUniform(
    block.get(), block.size(), seed, scope_max, scope_min, 0);

  return true;
}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, {options.m, options.k, 1});
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, {options.n, options.k, 1});
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, {options.m, options.n, 1});
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, {options.m, options.n, 1});

  block_A.reset(options.m * options.k);
  block_B.reset(options.k * options.n);
  block_C.reset(options.m * options.n);
  block_D.reset(options.m * options.n);
  block_ref_D.reset(options.m * options.n);

  initialize_block(block_A, seed + 2023);
  initialize_block(block_B, seed + 2022);
  initialize_block(block_C, seed + 2021);
}

```

**EN**: `initialize_block` chooses a random-fill range based on element bit width, and `initialize` computes packed strides, allocates device buffers, and fills A/B/C. This is standard testbed setup, but it also shows how CUTLASS stride helpers bridge the logical `(M,N,K)` problem description to the physical layouts expected by the built kernel.
**CN**: `initialize_block` 会根据元素位宽选择随机填充范围，而 `initialize` 则负责计算 packed stride、分配设备缓冲区并填充 A/B/C。虽然这是标准的测试框架初始化，但它也展示了 CUTLASS 的 stride 辅助函数如何把逻辑上的 `(M,N,K)` 问题描述映射到构建后 kernel 所需的物理布局。

### Lines 341-360

```cpp
/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options)
{
  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  int device_id = 0;
  cutlass::KernelHardwareInfo kernel_hw_info = cutlass::KernelHardwareInfo::make_kernel_hardware_info<Gemm::GemmKernel>(device_id);

  typename Gemm::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k},
    {block_A.get(), stride_A, block_B.get(), stride_B},
    {{options.alpha, options.beta}, block_C.get(), stride_C, block_D.get(), stride_D},
    kernel_hw_info
  };

  arguments.scheduler.raster_order = options.raster;
  // The tile scheduler will swizzle up to 8 and with the nearest multiple of 2 (i.e., 1, 2, 4, and 8) 
  arguments.scheduler.max_swizzle_size = options.swizzle;

```

**EN**: `args_from_options` creates the `Gemm::Arguments` object, queries `KernelHardwareInfo`, and attaches the scheduler controls. Setting `raster_order` and `max_swizzle_size` here is how the host code steers CTA traversal without changing the kernel’s compile-time math pipeline.
**CN**: `args_from_options` 构造 `Gemm::Arguments`，查询 `KernelHardwareInfo`，并附加调度器控制参数。在这里设置 `raster_order` 和 `max_swizzle_size`，就是主机侧在不改变 kernel 编译期数学流水线的前提下，引导 CTA 遍历方式的手段。

### Lines 361-395

```cpp
  return arguments;
}

bool verify(const Options &options) {
  cutlass::TensorRef ref_A(block_A.get(), Gemm::LayoutA::packed({options.m, options.k}));
  cutlass::TensorRef ref_B(block_B.get(), Gemm::LayoutB::packed({options.k, options.n}));
  cutlass::TensorRef ref_C(block_C.get(), Gemm::LayoutC::packed({options.m, options.n}));
  cutlass::TensorRef ref_D(block_ref_D.get(), Gemm::LayoutD::packed({options.m, options.n}));

  //
  // Compute reference output
  //

  // Create instantiation for device reference gemm kernel
  DeviceGemmReference gemm_reference;

  // Launch device reference gemm kernel
  gemm_reference(
    {options.m, options.n, options.k},
    ElementAccumulator(options.alpha),
    ref_A,
    ref_B,
    ElementAccumulator(options.beta),
    ref_C,
    ref_D);

  // Wait for kernel to finish
  CUDA_CHECK(cudaDeviceSynchronize());

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  bool passed = cutlass::reference::device::BlockCompareEqual(block_ref_D.get(), block_D.get(), block_D.size());

  return passed;
}

```

**EN**: `verify` uses CUTLASS’s device reference GEMM to compute a baseline result directly on the GPU, synchronizes, and compares the reference output with the CUTLASS kernel output. This keeps the example self-contained and confirms that the builder-selected Hopper kernel is functionally correct before timing.
**CN**: `verify` 使用 CUTLASS 的设备参考 GEMM 直接在 GPU 上计算基线结果，随后同步并把参考输出与 CUTLASS kernel 输出进行比较。这样做让示例保持自包含，同时也能在计时前确认 builder 所选的 Hopper kernel 在功能上是正确的。

### Lines 396-467

```cpp
/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options &options)
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
      CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
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

    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << std::endl;
    std::cout << "  Rasterization: " << raster << " with a maximum CTA swizzle of " << options.swizzle << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```

**EN**: `run` performs the end-to-end execution flow: initialize data, instantiate the GEMM object, obtain workspace size, call `can_implement`, initialize, warm up, verify, and then benchmark. The profile loop re-initializes the kernel before each run, which mirrors the standard CUTLASS example pattern and ensures argument-dependent scheduler state is refreshed as needed.
**CN**: `run` 展示了端到端的执行流程：初始化数据、实例化 GEMM 对象、查询 workspace 大小、调用 `can_implement`、完成初始化、预热、验证，然后进入基准测试。性能测试循环会在每轮运行前重新初始化 kernel，这符合 CUTLASS 示例的常见模式，也能在需要时刷新依赖参数的调度器状态。

### Lines 468-519

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

  Options options;

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

**EN**: `main` checks the CUDA toolkit and GPU architecture, parses command-line options, prints usage when requested, and finally dispatches `run<Gemm>`. The SM90 guard keeps CI green on unsupported machines while making the example a concrete Hopper-only entry point.
**CN**: `main` 检查 CUDA Toolkit 与 GPU 架构、解析命令行参数、按需打印帮助，并最终调用 `run<Gemm>`。SM90 保护逻辑让示例在不支持的平台上也不会导致 CI 失败，同时又把它明确限定为 Hopper 专用入口。

---

## Key Concepts / 关键概念

**EN**: The file uses collective builders instead of handwritten kernels, so the most important “analysis target” is the compile-time configuration rather than explicit per-warp code.
  **CN**: 本文件使用 collective builder 而不是手写 kernel，因此最重要的“分析对象”是编译期配置，而不是显式的逐 warp 代码。
**EN**: `StageCountAutoCarveout` is a crucial compile-time trait: it balances mainloop pipeline depth against epilogue shared-memory usage.
  **CN**: `StageCountAutoCarveout` 是一个关键的编译期 trait：它会在主循环流水线深度与 epilogue 共享内存占用之间做平衡。
**EN**: `KernelScheduleAuto` and the SM90 architecture tag allow CUTLASS to select a Hopper mainloop that internally uses asynchronous TMA/GMMA warp-specialized execution.
  **CN**: `KernelScheduleAuto` 与 SM90 架构标签让 CUTLASS 能自动选择一个内部采用异步 TMA/GMMA warp-specialized 执行的 Hopper 主循环。
**EN**: The runtime knobs only tune scheduler traversal; they do not change the math kernel’s template-instantiated structure.
  **CN**: 运行时参数只调节调度器的遍历方式，并不会改变数学 kernel 通过模板实例化得到的结构。

## Dependencies / 依赖项

**EN**: `cutlass/gemm/collective/collective_builder.hpp` and `cutlass/epilogue/collective/collective_builder.hpp` are the central APIs that materialize the Hopper mainloop and epilogue.
  **CN**: `cutlass/gemm/collective/collective_builder.hpp` 与 `cutlass/epilogue/collective/collective_builder.hpp` 是实例化 Hopper 主循环与 epilogue 的核心 API。
**EN**: `cutlass/util/reference/device/gemm.h` and tensor compare/fill helpers provide the correctness baseline.
  **CN**: `cutlass/util/reference/device/gemm.h` 以及张量比较/填充辅助组件提供了正确性基线。
**EN**: `cutlass/gemm/kernel/tile_scheduler_params.h` defines the rasterization and swizzle controls consumed through `arguments.scheduler`.
  **CN**: `cutlass/gemm/kernel/tile_scheduler_params.h` 定义了通过 `arguments.scheduler` 使用的 rasterization 与 swizzle 控制项。
