# 56_hopper_ptr_array_batched_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/56_hopper_ptr_array_batched_gemm/56_hopper_ptr_array_batched_gemm.cu`  
**Purpose / 用途**: This file demonstrates Hopper pointer-array batched GEMM in CUTLASS 3. Instead of using one strided tensor per operand, it builds arrays of device pointers for A/B/C/D so each batch can live at an arbitrary address, then uses SM90 pointer-array TMA kernels to update descriptors on the fly while comparing cooperative and pingpong warp-specialized schedules. / 这个文件演示了 CUTLASS 3 中的 Hopper 指针数组批量 GEMM。它没有使用统一步长的批量张量，而是为 A/B/C/D 构造设备指针数组，使每个 batch 都可以位于任意地址；随后利用 SM90 的 pointer-array TMA 内核在运行时动态更新描述符，并比较 cooperative 与 pingpong 两种 warp-specialized 调度。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-73)
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
    \brief Hopper Ptr-Array Batched GEMM example using CUTLASS 3 APIs for NVIDIA Hopper architecture.

    This example demonstrates an implementation of Ptr-Array Batched GEMM using a TMA + GMMA
    warp-specialized cooperative kernel.
    The new feature showcased in this example is on-the-fly modification of TMA descriptors
    to move between batches (represented by l).

    To run this example:

      $ ./examples/56_hopper_ptr_array_batched_gemm/56_hopper_ptr_array_batched_gemm --m=2048 --n=2048 --k=2048 --l=10
*/

#include <iostream>

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

#include "helper.h"

using namespace cute;

#if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
```
**EN**: The file header states the example goal, then includes the CUTLASS 3 collective-builder headers, reference GEMM helpers, and common utility headers. The compile guard `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED` foreshadows the main Hopper requirement: this example depends on modifiable TMA descriptors, a feature only available on the intended architecture/toolkit combination.
**CN**: 文件开头先说明示例目标，然后引入 CUTLASS 3 collective-builder 头文件、参考 GEMM 工具和公共辅助头。编译保护宏 `CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED` 预示了本例最关键的 Hopper 前提：它依赖“可修改的 TMA 描述符”，该能力只在特定架构/工具链组合下可用。

### Block 2 (Lines 74-156)
```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////

// A matrix configuration
using         ElementA    = cutlass::half_t;                                // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = cutlass::half_t;                                // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C/D matrix configuration
using         ElementC    = cutlass::half_t;                                // Element type for C and D matrix operands
using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using StageCountType = cutlass::gemm::collective::StageCountAuto;           // Stage count maximized based on the tile size

// Different configs for pingpong/cooperative
struct CooperativeConfig {
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperative;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
  using TileShape           = Shape<_256,_128,_64>;
  using ClusterShape        = Shape<_1,_2,_1>;
};

struct PingpongConfig {
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpong;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using TileShape           = Shape<_64,_128,_64>;
  using ClusterShape        = Shape<_1,_1,_1>;
};

template <typename ScheduleConfig>
struct GemmGivenSchedule {
  using TileShape           = typename ScheduleConfig::TileShape;                   // Threadblock-level tile size
  using ClusterShape        = typename ScheduleConfig::ClusterShape;                // Shape of the threadblocks in a cluster
  using KernelSchedule      = typename ScheduleConfig::KernelSchedule;              // Kernel to launch
  using EpilogueSchedule    = typename ScheduleConfig::EpilogueSchedule;            // Epilogue to launch

  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      cutlass::arch::Sm90, cutlass::arch::OpClassTensorOp,
      TileShape, ClusterShape,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator, ElementAccumulator,
      ElementC, LayoutC, AlignmentC,
      ElementC, LayoutC, AlignmentC,
      EpilogueSchedule
    >::CollectiveOp;

  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      ArchTag, OperatorClass,
      ElementA, LayoutA, AlignmentA,
      ElementB, LayoutB, AlignmentB,
      ElementAccumulator,
      TileShape, ClusterShape,
      cutlass::gemm::collective::StageCountAutoCarveout<
        static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
      KernelSchedule
    >::CollectiveOp;

  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
      cutlass::gemm::ArrayProblemShape<Shape<int,int,int,int>>,
      CollectiveMainloop,
      CollectiveEpilogue
  >;

  using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
};

using GemmKernel = GemmGivenSchedule<CooperativeConfig>::GemmKernel;
using Gemm = GemmGivenSchedule<CooperativeConfig>::Gemm;

using GemmKernelPingpong = GemmGivenSchedule<PingpongConfig>::GemmKernel;
using GemmPingpong = GemmGivenSchedule<PingpongConfig>::Gemm;
```
**EN**: This block defines the kernel configuration. It fixes operand types/layouts, then introduces two schedule families: cooperative and pingpong. `GemmGivenSchedule` is the reusable template that plugs `TileShape`, `ClusterShape`, `KernelSchedule`, and `EpilogueSchedule` into CUTLASS `CollectiveBuilder` instantiations. In other words, the CUTLASS-specific template parameters here describe how Hopper should tile the problem, how many CTAs cooperate in a cluster, and which TMA/warp-specialized pipeline should be generated.
**CN**: 这一段定义内核配置：先固定操作数类型和布局，再引入两种调度家族——cooperative 与 pingpong。`GemmGivenSchedule` 是一个可复用模板，把 `TileShape`、`ClusterShape`、`KernelSchedule` 与 `EpilogueSchedule` 注入 CUTLASS `CollectiveBuilder` 实例。换言之，这里的 CUTLASS 模板参数描述的是 Hopper 应如何分块、CTA 集群如何协作，以及应生成哪种 TMA/warp-specialized 流水。

### Block 3 (Lines 157-204)
```cpp


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

StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;
uint64_t seed;

std::vector<int64_t> offset_A;
std::vector<int64_t> offset_B;
std::vector<int64_t> offset_C;
std::vector<int64_t> offset_D;

cutlass::DeviceAllocation<typename Gemm::ElementA> block_A;
cutlass::DeviceAllocation<typename Gemm::ElementB> block_B;
cutlass::DeviceAllocation<typename Gemm::ElementC> block_C;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_D;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput> block_ref_D;

cutlass::DeviceAllocation<const typename Gemm::ElementA *> ptr_A;
cutlass::DeviceAllocation<const typename Gemm::ElementB *> ptr_B;
cutlass::DeviceAllocation<const typename Gemm::ElementC *> ptr_C;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput *> ptr_D;
cutlass::DeviceAllocation<typename Gemm::EpilogueOutputOp::ElementOutput *> ptr_ref_D;

#endif // defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////

// Command line options parsing
```
**EN**: After the kernel aliases are selected, the file declares the device-side reference GEMM type, stride aliases, global stride state, offset arrays, and the storage objects for both contiguous operand blocks and pointer arrays. The distinction matters: the actual matrix data is stored in large contiguous buffers, while the kernel interface receives arrays of per-batch base pointers.
**CN**: 在选定内核别名之后，文件声明了设备端参考 GEMM 类型、stride 别名、全局 stride 状态、offset 数组，以及连续数据块和指针数组的存储对象。这里的区分很重要：真实矩阵数据放在大块连续缓冲区中，而内核接口接收的是“每个 batch 一个基址指针”的数组。

### Block 4 (Lines 205-274)
```cpp
struct Options {

  bool help = false;

  float alpha = 1.0f;
  float beta = 0.0f;
  int iterations = 10;
  int m = 1024, n = 512, k = 1024, l = 10;

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
    cmd.get_cmd_line_argument("l", l);
    cmd.get_cmd_line_argument("alpha", alpha, 1.f);
    cmd.get_cmd_line_argument("beta", beta, 0.f);
    cmd.get_cmd_line_argument("iterations", iterations);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "56_hopper_ptr_array_batched_gemm\n\n"
      << "  Hopper FP32 GEMM using a Warp Specialized kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   Sets the batch count for Ptr-Array GEMM\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "56_hopper_ptr_array_batched_gemm" << " --m=1024 --n=512 --k=1024 --l=10 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const
  {
    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * m * n * k * l;
    double gflop = double(flop) / double(1.0e9);
    return gflop / runtime_s;
  }
};

/// Result structure
struct Result
{
  double avg_runtime_ms = 0.0;
  double gflops = 0.0;
  cutlass::Status status = cutlass::Status::kSuccess;
  cudaError_t error = cudaSuccess;
  bool passed = false;
};

#if defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)
```
**EN**: `Options` provides the CLI surface for problem shape, batch count, alpha/beta, and iteration count; `Result` stores runtime and validation status. These two structs keep execution policy separate from kernel type aliases, which makes it easier to run the same setup against both schedule variants.
**CN**: `Options` 提供问题规模、batch 数、alpha/beta 和迭代次数等命令行接口；`Result` 则保存性能和校验状态。这两个结构体把执行策略和内核类型别名分离开，便于用同一套输入同时测试两种调度方案。

### Block 5 (Lines 275-338)
```cpp

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
    scope_max = static_cast<Element>(2);
    scope_min = static_cast<Element>(0);
  } else if (bits_input <= 8) {
    scope_max = static_cast<Element>(2);
    scope_min = static_cast<Element>(-2);
  } else {
    scope_max = static_cast<Element>(8);
    scope_min = static_cast<Element>(-8);
  }

  cutlass::reference::device::BlockFillRandomUniform(
    block.get(), block.size(), seed, scope_max, scope_min, 0);

  return true;
}

/// Allocates device-side data
void allocate(const Options &options) {
  int64_t total_elements_A = 0;
  int64_t total_elements_B = 0;
  int64_t total_elements_C = 0;
  int64_t total_elements_D = 0;

  for (int32_t i = 0; i < options.l; ++i) {

    offset_A.push_back(total_elements_A);
    offset_B.push_back(total_elements_B);
    offset_C.push_back(total_elements_C);
    offset_D.push_back(total_elements_D);

    int64_t elements_A = options.m * options.k;
    int64_t elements_B = options.k * options.n;
    int64_t elements_C = options.m * options.n;
    int64_t elements_D = options.m * options.n;

    total_elements_A += elements_A;
    total_elements_B += elements_B;
    total_elements_C += elements_C;
    total_elements_D += elements_D;
  }

  block_A.reset(total_elements_A);
  block_B.reset(total_elements_B);
  block_C.reset(total_elements_C);
  block_D.reset(total_elements_D);
  block_ref_D.reset(total_elements_D);
}

/// Initialize operands to be used in the GEMM and reference GEMM
```
**EN**: `initialize_block()` fills arbitrary device buffers with range-aware random data. `allocate()` then computes per-batch offsets, accumulates the total storage required for all batches, and allocates one large block for each operand/result tensor. This is the bridge between the conceptual pointer-array API and a compact storage model convenient for examples.
**CN**: `initialize_block()` 负责给任意设备缓冲区填充范围感知的随机数据。随后 `allocate()` 计算每个 batch 的偏移、累加所有 batch 所需总容量，并为各个操作数/结果张量分配一整块连续存储。这一层把“概念上的指针数组接口”和“示例中方便管理的紧凑存储模型”连接起来。

### Block 6 (Lines 339-399)
```cpp
void initialize(const Options &options) {

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.m, options.n, options.l));
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));

  //
  // Assign pointers
  //

  std::vector<ElementA *> ptr_A_host(options.l);
  std::vector<ElementB *> ptr_B_host(options.l);
  std::vector<ElementC *> ptr_C_host(options.l);
  std::vector<ElementC *> ptr_D_host(options.l);

  for (int32_t i = 0; i < options.l; ++i) {
    ptr_A_host.at(i) = block_A.get() + offset_A.at(i);
    ptr_B_host.at(i) = block_B.get() + offset_B.at(i);
    ptr_C_host.at(i) = block_C.get() + offset_C.at(i);
    ptr_D_host.at(i) = block_D.get() + offset_D.at(i);
  }

  ptr_A.reset(options.l);
  ptr_A.copy_from_host(ptr_A_host.data());

  ptr_B.reset(options.l);
  ptr_B.copy_from_host(ptr_B_host.data());

  ptr_C.reset(options.l);
  ptr_C.copy_from_host(ptr_C_host.data());

  ptr_D.reset(options.l);
  ptr_D.copy_from_host(ptr_D_host.data());

  initialize_block(block_A, seed + 2023);
  initialize_block(block_B, seed + 2022);
  initialize_block(block_C, seed + 2021);
}

/// Populates a Gemm::Arguments structure from the given commandline options
template <typename GemmT>
typename GemmT::Arguments args_from_options(const Options &options)
{
  cutlass::KernelHardwareInfo hw_info;
  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  typename GemmT::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kArray,
    {{options.m, options.n, options.k, options.l}},
    {ptr_A.get(), stride_A, ptr_B.get(), stride_B},
    {{options.alpha, options.beta}, ptr_C.get(), stride_C, ptr_D.get(), stride_D},
    hw_info
  };

  return arguments;
}

```
**EN**: `initialize()` builds packed strides, derives host-side pointer arrays from the contiguous allocations, copies those pointer arrays to device memory, and initializes A/B/C data. `args_from_options()` is the key pointer-array batching entry: it creates `GemmUniversalMode::kArray` arguments and passes `{ptr_A, ptr_B, ptr_C, ptr_D}` plus the packed strides and hardware info. That is what distinguishes this example from a strided-batched GEMM.
**CN**: `initialize()` 构建 packed stride，根据连续存储推导主机侧指针数组，再把这些指针数组拷到设备端，并初始化 A/B/C 数据。`args_from_options()` 是指针数组批量化的关键入口：它创建 `GemmUniversalMode::kArray` 参数，并传入 `{ptr_A, ptr_B, ptr_C, ptr_D}`、对应 stride 以及硬件信息。这正是它区别于 strided-batched GEMM 的地方。

### Block 7 (Lines 400-433)
```cpp
bool verify(const Options &options) {
  bool passed = true;
  for (int32_t i = 0; i < options.l; ++i) {
    cutlass::TensorRef ref_A(block_A.get() + offset_A.at(i), Gemm::LayoutA::packed({options.m, options.k}));
    cutlass::TensorRef ref_B(block_B.get() + offset_B.at(i), Gemm::LayoutB::packed({options.k, options.n}));
    cutlass::TensorRef ref_C(block_C.get() + offset_C.at(i), Gemm::LayoutC::packed({options.m, options.n}));
    cutlass::TensorRef ref_D(block_ref_D.get() + offset_D.at(i), Gemm::LayoutD::packed({options.m, options.n}));

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
    passed &= cutlass::reference::device::BlockCompareEqual(block_ref_D.get() + offset_D.at(i), block_D.get() + offset_D.at(i), options.m * options.n);
  }
  return passed;
}

```
**EN**: `verify()` iterates over batches, materializes per-batch `TensorRef` objects using the stored offsets, runs a reference GEMM for each batch, synchronizes, and compares every result block against the CUTLASS output. Because pointer-array batching decouples storage from addressing, explicit per-batch verification is the safest correctness check.
**CN**: `verify()` 逐个 batch 构造基于偏移的 `TensorRef`，对每个 batch 运行参考 GEMM、同步后再与 CUTLASS 输出逐块比较。由于指针数组批量化把“存储”和“寻址”解耦了，显式逐 batch 校验是最稳妥的正确性检查方式。

### Block 8 (Lines 434-499)
```cpp
/// Execute a given example GEMM computation
template <typename GemmT>
int run(Options &options)
{
  allocate(options);
  initialize(options);

  // Instantiate CUTLASS kernel depending on templates
  GemmT gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = args_from_options<GemmT>(options);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = GemmT::get_workspace_size(arguments);

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

    // Compute average setup and runtime and GFLOPs.
    float elapsed_ms       = timer.elapsed_millis();
    result.avg_runtime_ms  = double(elapsed_ms) / double(options.iterations);
    result.gflops          = options.gflops(result.avg_runtime_ms / 1000.0);

    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << std::endl;
    std::cout << "  Batches     : " << options.l  << std::endl;
    std::cout << "  Alpha, Beta : " << options.alpha << ',' << options.beta << std::endl;
    std::cout << "  Avg runtime : " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  TFLOPS      : " << result.gflops / 1000.0 << std::endl;
  }

  return 0;
}

#endif // defined(CUTLASS_ARCH_MMA_MODIFIABLE_TMA_SM90_SUPPORTED)

```
**EN**: `run()` is the standard CUTLASS execution pipeline: allocate, initialize, instantiate the device GEMM, query workspace, check `can_implement()`, initialize the kernel, run once for correctness, verify, then profile repeated `initialize()+run()` pairs. Re-initializing inside the benchmark loop is intentional here because pointer-array kernels may need descriptor setup work that is part of the real end-to-end cost.
**CN**: `run()` 体现了标准的 CUTLASS 执行流程：分配、初始化、实例化设备 GEMM、查询 workspace、检查 `can_implement()`、初始化内核、先跑一次用于正确性，再做校验，最后对多次 `initialize()+run()` 进行性能测试。这里在基准循环中重复初始化是有意为之，因为指针数组内核往往需要描述符配置，这本来就是端到端成本的一部分。

### Block 9 (Lines 500-553)
```cpp
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
  std::cout << "\n*** Cooperative schedule ***" << std::endl;
  run<Gemm>(options);
  std::cout << "\n*** Pingpong schedule ***" << std::endl;
  run<GemmPingpong>(options);
#endif

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////

```
**EN**: `main()` enforces the CUDA 12.3 + Hopper environment, parses options, and then runs both schedule variants under the same inputs. Printing separate "Cooperative" and "Pingpong" sections makes the file a focused scheduling comparison for pointer-array batched GEMM on SM90.
**CN**: `main()` 强制检查 CUDA 12.3 与 Hopper 环境，解析参数后，在相同输入下分别运行两种调度方案。单独打印 “Cooperative” 和 “Pingpong” 两个部分，使这个文件成为一个专门比较 SM90 上指针数组批量 GEMM 调度差异的示例。

---

## Key Concepts / 关键概念

**EN**:
- Pointer-array batching is different from strided batching: each batch is addressed by an explicit pointer rather than by `base + batch_id * stride_batch`.
- The schedule names `KernelPtrArrayTmaWarpSpecializedCooperative` and `...Pingpong` tell you this is a Hopper TMA pipeline specialized for pointer arrays and different warp-specialized producer/consumer orchestration styles.
- `StageCountAutoCarveout<sizeof(CollectiveEpilogue::SharedStorage)>` lets CUTLASS pick the mainloop stage count after reserving shared memory needed by the epilogue.
- The example stores batch data contiguously anyway, but passes pointer arrays to mimic the general case where batches are irregularly placed in memory.

**CN**:
- 指针数组批量化不同于 strided batching：每个 batch 通过显式指针寻址，而不是依赖 `base + batch_id * stride_batch`。
- `KernelPtrArrayTmaWarpSpecializedCooperative` 和 `...Pingpong` 这些调度名说明：这里使用的是 Hopper 上针对指针数组优化的 TMA 流水，以及不同的 warp-specialized 生产者/消费者协作风格。
- `StageCountAutoCarveout<sizeof(CollectiveEpilogue::SharedStorage)>` 让 CUTLASS 在预留 epilogue 所需共享内存之后，再自动决定 mainloop 的 stage 数。
- 虽然示例内部仍把 batch 数据连续存放，但向内核传递的是指针数组，这更贴近“各 batch 在内存中不规则分布”的通用场景。

## Dependencies / 依赖项

**EN**:
- `cutlass/gemm/group_array_problem_shape.hpp` and `GemmUniversalMode::kArray` support the pointer-array problem description.
- `CollectiveBuilder` and `GemmUniversalAdapter` build the actual Hopper device kernels from the schedule/type/layout choices.
- `cutlass/util/reference/device/gemm.h` provides the verification kernel used batch by batch.
- `helper.h` supplies `GpuTimer`, CUDA-check macros, and other example infrastructure.

**CN**:
- `cutlass/gemm/group_array_problem_shape.hpp` 与 `GemmUniversalMode::kArray` 支撑了指针数组问题描述方式。
- `CollectiveBuilder` 和 `GemmUniversalAdapter` 根据调度、类型和布局选择生成真正的 Hopper 设备内核。
- `cutlass/util/reference/device/gemm.h` 提供逐 batch 校验时使用的参考内核。
- `helper.h` 提供 `GpuTimer`、CUDA 检查宏等示例基础设施。
