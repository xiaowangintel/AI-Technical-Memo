# 63_hopper_gemm_with_weight_prefetch.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/63_hopper_gemm_with_weight_prefetch/63_hopper_gemm_with_weight_prefetch.cu`  
**Purpose / 用途**: Demonstrates a Hopper FP8 GEMM whose custom warp-specialized mainloop uses programmatic dependent launch (PDL) to prefetch weight tiles into L2 before the dependent activation traffic arrives, exposing runtime knobs for overlap and prefetch aggressiveness. / 演示一个 Hopper FP8 GEMM：其自定义 warp-specialized 主循环通过 programmatic dependent launch (PDL) 在依赖的激活流量到来之前把权重 tile 预取到 L2，并暴露 overlap 与 prefetch 激进程度两个运行时调参旋钮。

---

## Line-by-Line Analysis / 逐行分析

### PDL-based weight-prefetch motivation and custom headers (Lines 32-127)

```cpp
/*! \file
    \brief Hopper FP8 GEMM + L2 Weight Prefetch

    This example implements a non-persistent warp-specialized GEMM kernel for the Hopper
    architecture with programmatic dependent launch (PDL) enabling prefetching weights into
    L2 cache.
    
    For more information about dependent launch refer to the CUDA programming guide:
    https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programmatic-dependent-launch-and-synchronization

    In some cases, PDL can result in a window where a previous kernel is not actively utilizing 
    DRAM, and the next kernel sits idle until the previous finishes. During this window, the next
    kernel can begin loading a non-dependent operand (i.e. weights in a linear projection are
    typically static) and cache it in L2.

    The kernel and collective mainloop assume operand `A` corresponds to weights and operand `B`
    corresponds to activations (so we can have very small batch/token count).
    After initialization, the prefetch warp starts loading K tiles of `A` into an unused portion 
    of shared memory, and loads up to half of all K tiles that the same CTA would eventually load.
    The exact number of K tiles loaded is determined by `args.mainloop.prefetch_ratio` \in 
    [0.0, 1.0]. Smaller values result in less prefetching, and larger values result in more.
    Negative values result in a "best-effort" prefetch, meaning prefetcher will stop issuing weight
    loads as soon as the activation DMA warp starts loading (as soon as it is signaled that the 
    previous kernel has flushed its memory.)

    The DMA warp responsible for loading `A` will also begin loading K tiles until it fills up
    the available shared memory.
    The DMA warp responsible for loading `B` will wait until activations are flushed to global 
    memory by the preceding kernel.

    Another mainloop parameter, `args.mainloop.overlap_ratio` \in [0.0, 1.0] determines how early 
    the next kernel (the one doing the prefetch) is launched. Smaller values result in greater 
    overlap, and larger values result in smaller overlap. Negative values disable PDL completely,
    meaning there will be no overlap. This will make prefetch ineffective.

    These two runtime parameters should be tuned per problem size and GEMM config combination, and
    if feasible, per-operation in an entire layer or model.

    NOTE: you must build this target with the following flag to enable Grid Dependency Control
    instructions (GDC) in CUTLASS:
      - CUTLASS_ENABLE_GDC_FOR_SM90

    To lock persistence mode, power (350W), clocks (1005MHz) for evaluation (assumes device 0 and H100)

      $ sudo nvidia-smi -pm 1 -i 0

      $ sudo nvidia-smi -i 0 -pl 350

      $ sudo nvidia-smi -i 0 -lgc 1005

    Example:

      $ mkdir build && cd build

      $ cmake .. -DCUTLASS_NVCC_ARCHS="90a" -DCUTLASS_ENABLE_GDC_FOR_SM90=1

      $ cd examples/63_hopper_gemm_with_weight_prefetch

      $ make

      $ ./63_hopper_gemm_with_weight_prefetch --p=0.5 --o=0.5
*/

#include <iostream>

#include "cutlass/cutlass.h"
#include "cutlass/numeric_types.h"

#include "cute/tensor.hpp"
#include "cutlass/tensor_ref.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"
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


#include "collective/dispatch_policy_extra.hpp"
#include "collective/builder.hpp"
#include "kernel/sm90_gemm_tma_warpspecialized_with_prefetch.hpp"

#include "helper.h"
#include "gemm_with_weight_prefetch_commandline.hpp"
```

**EN**: The opening comment precisely defines the optimization model. Operand A is treated as static weights, operand B as activations, and an earlier kernel’s completion is used as the synchronization point for a later GEMM to begin prefetching A into L2. `prefetch_ratio` controls how much of A’s future K-tiles a CTA tries to stage, while `overlap_ratio` controls how early the next kernel is launched relative to the predecessor. The custom local headers included here are what make this non-standard schedule possible.

**CN**: 开头注释精确定义了这个优化模型：把 A 视为静态权重，把 B 视为激活，并利用前一内核的完成时机，让后一 GEMM 在真正依赖激活之前先把 A 预取到 L2。`prefetch_ratio` 控制一个 CTA 试图预取多少未来的 A 的 K-tile，`overlap_ratio` 则控制后一内核相对前一内核提前多少启动。这里引入的本地自定义头文件正是实现这种非常规调度的基础。

### FP8 kernel type and custom prefetch schedule selection (Lines 136-196)

```cpp
// A matrix configuration
using         ElementA    = cutlass::float_e4m3_t;                          // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = cutlass::float_e5m2_t;                          // Element type for B matrix operand
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

// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ElementCompute      = float;                                          // Element type for epilogue computation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_64,_64,_128>;                            // Threadblock-level tile size
// Cluster_N > 1 is not supported yet.
using ClusterShape        = Shape<_1,_1,_1>;                                // Shape of the threadblocks in a cluster
using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA;
using EpilogueSchedule    = cutlass::epilogue::TmaWarpSpecialized;
using EpilogueTileType    = cutlass::epilogue::collective::EpilogueTileAuto;

using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
    ArchTag, OperatorClass,
    TileShape, ClusterShape,
    EpilogueTileType,
    ElementAccumulator, ElementCompute,
    ElementC, LayoutC, AlignmentC,
    ElementD, LayoutD, AlignmentD,
    EpilogueSchedule
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
```

**EN**: This block selects FP8 inputs (`e4m3` weights and `e5m2` activations), FP32 accumulation/compute, a 64x64x128 tile, and a 1x1x1 cluster. The standout choice is `KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA`, a custom schedule type that tells the builder to instantiate a warp-specialized Hopper mainloop capable of both weight prefetch and split DMA responsibilities.

**CN**: 这里选择了 FP8 输入（`e4m3` 权重、`e5m2` 激活）、FP32 累加/计算、64x64x128 的线程块 tile，以及 1x1x1 的 cluster。其中最醒目的选择是 `KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA`：这是一个自定义调度类型，用来指示 builder 生成一个既支持权重预取又支持 split DMA 的 Hopper warp-specialized 主循环。

### Kernel-derived types, strides, and tensor storage (Lines 198-224)

```cpp
// Extract information from Gemm kernel.
using EpilogueOutputOp  = typename Gemm::EpilogueOutputOp;
using ElementScalar     = typename EpilogueOutputOp::ElementScalar;

using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideC = typename Gemm::GemmKernel::StrideC;
using StrideD = typename Gemm::GemmKernel::StrideD;

/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideC stride_C;
StrideD stride_D;
uint64_t seed;

cutlass::HostTensor<ElementA  , LayoutA  > tensor_A;
cutlass::HostTensor<ElementB  , LayoutB  > tensor_B;
cutlass::HostTensor<ElementC  , LayoutC  > tensor_C;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_D;
cutlass::HostTensor<ElementD  , LayoutD  > tensor_ref_D;

using LayoutScalar = cutlass::layout::PackedVectorLayout;
cutlass::HostTensor<ElementScalar, LayoutScalar> scalar_alpha;
cutlass::HostTensor<ElementScalar, LayoutScalar> scalar_beta;

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```

**EN**: After defining `Gemm`, the code extracts epilogue scalar types and the A/B/C/D stride types directly from the instantiated kernel. That keeps the example robust against changes in the builder-selected kernel interface. It then allocates host tensors for operands, outputs, reference output, and scalar alpha/beta values that can be passed by pointer into the fused epilogue.

**CN**: 在定义完 `Gemm` 之后，代码直接从已实例化的内核中提取 epilogue 标量类型以及 A/B/C/D 的 stride 类型。这样做可以避免手写接口细节，与 builder 生成的具体内核保持一致。随后它为输入、输出、参考输出以及可通过指针传入融合 epilogue 的 alpha/beta 标量分配 host tensor。

### Tensor initialization for FP8 test data (Lines 258-317)

```cpp
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
void initialize(const Options &options) {

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(options.m, options.n, options.l));
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));

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
}
```

**EN**: `initialize_tensor()` chooses narrower random ranges for lower-precision element types so the generated FP8 values stay in a numerically reasonable regime. `initialize()` computes packed strides for all operands, allocates host tensors, fills them, and synchronizes A/B/C/D to device memory. No distributed state is needed here; the novelty of the example lies in the custom kernel schedule, not in the data setup.

**CN**: `initialize_tensor()` 会为低精度元素类型选择较窄的随机范围，以便生成的 FP8 数值保持在合理的数值区间内。`initialize()` 负责计算各操作数的 packed stride、分配 host tensor、填充数据，并把 A/B/C/D 同步到 device。这里并不涉及分布式状态；示例的新意主要在自定义内核调度，而不是数据准备。

### Wiring epilogue scalars and runtime prefetch controls (Lines 318-342)

```cpp
/// Populates a Gemm::Arguments structure from the given commandline options
typename Gemm::Arguments args_from_options(const Options &options)
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

  arguments.mainloop.overlap_ratio = options.overlap_ratio;
  arguments.mainloop.prefetch_ratio = options.prefetch_ratio;

  return arguments;
}
```

**EN**: `args_from_options()` creates a standard universal GEMM argument pack, then mutates two advanced substructures. First, it fills `arguments.epilogue.thread` with scalar values and scalar pointers, demonstrating CUTLASS 3.x’s fused-epilogue argument model. Second, it assigns `arguments.mainloop.overlap_ratio` and `arguments.mainloop.prefetch_ratio`, which are the runtime knobs consumed by the custom prefetch-enabled mainloop.

**CN**: `args_from_options()` 先构造一份标准的 universal GEMM 参数，然后再修改两个高级子结构。第一，它填充 `arguments.epilogue.thread` 中的标量值和标量指针，展示了 CUTLASS 3.x 融合 epilogue 的参数组织方式。第二，它给 `arguments.mainloop.overlap_ratio` 与 `arguments.mainloop.prefetch_ratio` 赋值，这两个运行时参数会被自定义的预取主循环直接消费。

### Host-side GETT reference path (Lines 344-388)

```cpp
bool verify(const Options &options) {
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
      unused_t, // aux
      unused_t, // valpha
      unused_t  // vbeta
  > epilogue_params;

  epilogue_params.C = C;
  epilogue_params.D = D;
  epilogue_params.alpha = options.alpha;
  epilogue_params.beta = options.beta;

  // get reference result
  cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);

  // compare_reference
  tensor_D.sync_host();
  bool passed = cutlass::reference::host::TensorEquals(tensor_ref_D.host_view(), tensor_D.host_view());

  return passed;
}
```

**EN**: Instead of reusing another device kernel for verification, the example constructs CuTe host tensors and invokes `cutlass::reference::host::Gemm3x`. This is a clean choice for a custom prefetch schedule example: correctness is validated against the mathematical GEMM/eilogue definition rather than against another device implementation that might hide scheduling-specific issues.

**CN**: 与其再调用另一个 device kernel 来做验证，这个示例直接在 host 上构造 CuTe tensor，并调用 `cutlass::reference::host::Gemm3x`。对自定义预取调度示例而言，这是一个更干净的选择：验证依据是 GEMM/epilogue 的数学定义，而不是另一个可能掩盖调度问题的 device 实现。

### Launching the custom GEMM with or without PDL overlap (Lines 390-451)

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
  CUTLASS_CHECK(gemm.run(nullptr, nullptr, /* launch_with_pdl = */ options.overlap_ratio >= 0));

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
      CUTLASS_CHECK(gemm.run(nullptr, nullptr, /* launch_with_pdl = */ options.overlap_ratio >= 0));
    }
    timer.stop();

    // Compute average runtime and GFLOPs.
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    double avg_runtime_s = (double)(result.avg_runtime_ms / 1000.0);
    result.gflops = options.gflops(avg_runtime_s);
    result.eff_bw = options.effective_bandwidth(avg_runtime_s, sizeof(ElementA), sizeof(ElementB), sizeof(ElementC), sizeof(ElementD));

    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << 'x' << options.l << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
    std::cout << "  Effective bandwidth: " << result.eff_bw << " GB/s" << std::endl;
  }

  return 0;
}
```

**EN**: The execution flow is standard until the actual `gemm.run()` call. There the example explicitly passes `launch_with_pdl = (options.overlap_ratio >= 0)`. Negative overlap disables programmatic dependent launch and therefore makes prefetch ineffective by construction; non-negative overlap enables the custom launch path so the kernel can overlap predecessor completion with weight-prefetch activity. Profiling then reports both compute throughput and effective memory bandwidth.

**CN**: 执行流程在 `gemm.run()` 之前都比较常规；真正特殊之处在于它显式传入 `launch_with_pdl = (options.overlap_ratio >= 0)`。当 overlap 为负数时，会直接禁用 programmatic dependent launch，因此从定义上就不会发生有效预取；当 overlap 非负时，则开启自定义启动路径，使内核可以把前一内核的结束阶段与权重预取活动重叠起来。最后的 profiling 同时报告计算吞吐率和有效带宽。

### Program entry and architecture gating (Lines 457-500)

```cpp
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
```

**EN**: The entrypoint enforces CUDA 12+ and Hopper SM90, parses the custom command-line options, and runs the single custom `Gemm` instantiation. As in other CUTLASS examples, unsupported environments return zero so the example remains build-friendly on broader systems.

**CN**: 入口函数检查 CUDA 12+ 与 Hopper SM90 条件，解析自定义命令行参数，并运行唯一的自定义 `Gemm` 实例。和其他 CUTLASS 示例一样，在不支持的环境上返回 0，从而保证更广泛的平台上也能顺利构建。

---

## Key Concepts / 关键概念

- **EN**: Weight prefetch here is not generic caching advice; it is a specific Hopper PDL workflow where the next kernel begins issuing non-dependent A loads before dependent B traffic is ready.
  **CN**: 这里的权重预取并不是泛泛的缓存优化建议，而是 Hopper 上一种特定的 PDL 工作流：后一内核会在依赖的 B 流量准备好之前，先发起与之无关的 A 加载。
- **EN**: `prefetch_ratio` controls how many future weight K-tiles the prefetch warp attempts to pull toward L2, while `overlap_ratio` controls the kernel-launch overlap window.
  **CN**: `prefetch_ratio` 控制预取 warp 会尝试把多少未来的权重 K-tile 拉近到 L2，`overlap_ratio` 则控制内核启动的重叠窗口。
- **EN**: The custom schedule type is the compile-time hook that steers the collective builder toward the weight-prefetch implementation.
  **CN**: 自定义 schedule 类型就是编译期钩子，它把 collective builder 引导到支持权重预取的实现分支。
- **EN**: Host-side `Gemm3x` verification isolates math correctness from any specialized DMA/prefetch scheduling effects.
  **CN**: host 侧 `Gemm3x` 验证把数学正确性与特殊的 DMA/预取调度效果分离开来。

## Dependencies / 依赖项

- **EN**: The local `collective/builder.hpp`, `collective/dispatch_policy_extra.hpp`, and `kernel/sm90_gemm_tma_warpspecialized_with_prefetch.hpp` files define the custom mainloop builder and kernel schedule used by this example.
  **CN**: 本地的 `collective/builder.hpp`、`collective/dispatch_policy_extra.hpp` 和 `kernel/sm90_gemm_tma_warpspecialized_with_prefetch.hpp` 定义了本示例使用的自定义主循环 builder 与内核调度。
- **EN**: `gemm_with_weight_prefetch_commandline.hpp` supplies the `Options` type and runtime knobs for overlap and prefetch ratios.
  **CN**: `gemm_with_weight_prefetch_commandline.hpp` 提供了 `Options` 类型，以及 overlap/prefetch ratio 等运行时调参项。
- **EN**: `cutlass/util/reference/host/gett.hpp` provides the host-side `Gemm3x` verification path.
  **CN**: `cutlass/util/reference/host/gett.hpp` 提供了 host 侧的 `Gemm3x` 验证路径。
- **EN**: Standard CUTLASS collective builders still assemble the overall GEMM; the custom schedule only changes the chosen mainloop machinery.
  **CN**: 整体 GEMM 仍然由标准 CUTLASS collective builder 负责组装；自定义 schedule 只改变被选中的主循环机制。
