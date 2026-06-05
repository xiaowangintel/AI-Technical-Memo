# 65_distributed_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/65_distributed_gemm/65_distributed_gemm.cu`  
**Purpose / 用途**: Demonstrates Hopper distributed GEMM by lifting a single-device SM90 GEMM into CUTLASS experimental DistGEMM wrappers, slicing tensors per GPU, coordinating peer access, and benchmarking multi-GPU execution against a single-GPU reference. / 演示 Hopper 上的分布式 GEMM：先定义一个单卡 SM90 GEMM，再通过 CUTLASS 实验性 DistGEMM 封装器把它扩展到多 GPU，按设备切分张量、配置点对点访问，并与单卡参考实现做正确性和性能对比。

---

## Line-by-Line Analysis / 逐行分析

### Overview, prerequisites, and distributed headers (Lines 32-123)

```cpp
/*! \file
    \brief Distributed GEMM (DistGEMM) for Hopper

    This example runs Tensor Parallel GEMMs using the (experimental) Distributed GEMM API in 
    CUTLASS. For more information, please refer to README.md.

    Note that Distributed GEMM assumes an any-to-any NVLink network topology.
    To check whether your device is compatible, run:

      $ nvidia-smi topo -m

    and make sure there's an any-to-any NVLink topology. It would look like this:

                GPU0    GPU1    GPU2    GPU3    GPU4    GPU5    GPU6    GPU7
        GPU0     X      NV18    NV18    NV18    NV18    NV18    NV18    NV18
        GPU1    NV18     X      NV18    NV18    NV18    NV18    NV18    NV18
        GPU2    NV18    NV18     X      NV18    NV18    NV18    NV18    NV18
        GPU3    NV18    NV18    NV18     X      NV18    NV18    NV18    NV18
        GPU4    NV18    NV18    NV18    NV18     X      NV18    NV18    NV18
        GPU5    NV18    NV18    NV18    NV18    NV18     X      NV18    NV18
        GPU6    NV18    NV18    NV18    NV18    NV18    NV18     X      NV18
        GPU7    NV18    NV18    NV18    NV18    NV18    NV18    NV18     X

    You should also additionally check if the driver enables peer to peer access:

      $ nvidia-smi topo -p2p r

    Output should be something like this:

               GPU0    GPU1    GPU2    GPU3    GPU4    GPU5    GPU6    GPU7
        GPU0   X       OK      OK      OK      OK      OK      OK      OK
        GPU1   OK      X       OK      OK      OK      OK      OK      OK
        GPU2   OK      OK      X       OK      OK      OK      OK      OK
        GPU3   OK      OK      OK      X       OK      OK      OK      OK
        GPU4   OK      OK      OK      OK      X       OK      OK      OK
        GPU5   OK      OK      OK      OK      OK      X       OK      OK
        GPU6   OK      OK      OK      OK      OK      OK      X       OK
        GPU7   OK      OK      OK      OK      OK      OK      OK      X

    It is recommended to build this target with the following flag to enable 
    Grid Dependency Control instructions (GDC) in CUTLASS:
      - CUTLASS_ENABLE_GDC_FOR_SM90

    Example:

      $ mkdir build && cd build

      $ cmake .. -DCUTLASS_NVCC_ARCHS="90a" -DCUTLASS_ENABLE_GDC_FOR_SM90=1

      $ cd examples/65_distributed_gemm

      $ make

      $ ./65_distributed_gemm
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
#include "cutlass/util/reference/host/error_metrics.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"

// Distributed GEMM headers
#include "cutlass/experimental/distributed/device/dist_gemm_universal_wrapper.hpp"
#include "cutlass/experimental/distributed/kernel/dist_gemm_kernel_wrapper.hpp"
#include "cutlass/experimental/distributed/schedules/dist_gemm_1d_schedules.hpp"

#include "helper.h"

// Distributed GEMM helpers
#include "dist_gemm_helpers.h"
```

**EN**: The file header explains that this example targets Hopper tensor parallel GEMM on an any-to-any NVLink fabric, recommends enabling GDC for SM90, and includes both standard CUTLASS GEMM headers and experimental distributed wrappers. The two local helper headers encapsulate utility code used specifically by this example.

**CN**: 文件头先说明该示例面向 Hopper 上的张量并行 GEMM，需要任意两卡可达的 NVLink 拓扑，并建议为 SM90 打开 GDC。随后包含常规 CUTLASS GEMM 头文件以及实验性的分布式封装头；两个本地 helper 头则提供本示例专用的辅助工具。

### Tensor-parallel size and distributed schedule selection (Lines 131-149)

```cpp
// TP size (= number of processors/GPUs)
using TP = _8;
static constexpr int TP_ = TP{};

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) && \
  (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 6))

// Distributed GEMM tiling/sharding schedule
// Choices:
//
// * All Gather + GEMM:
//   * AllGather1D_TilingCD_RotatingA
//   * AllGather1D_TilingCD_RotatingB
//
// * GEMM + Reduce Scatter:
//   * ReduceScatter1D_TilingA_RotatingC
//   * ReduceScatter1D_TilingB_RotatingC

using DistSchedule = cutlass::distributed::schedules::AllGather1D_TilingCD_RotatingA<TP>;
```

**EN**: The example hard-codes an 8-way tensor-parallel configuration (`TP = _8`). `AllGather1D_TilingCD_RotatingA` defines the orchestration strategy: outputs are 1D tiled in the C/D space while A slices rotate across ranks so each device can eventually see the needed operand fragments for its local GEMM work.

**CN**: 示例把张量并行规模固定为 8 卡（`TP = _8`）。`AllGather1D_TilingCD_RotatingA` 给出了调度策略：结果在 C/D 空间做 1D 分块，而 A 的分片在各 rank 之间轮转/聚合，使每张卡都能拿到完成本地 GEMM 所需的操作数片段。

### Single-device GEMM kernel and distributed wrapper types (Lines 155-255)

```cpp
// A matrix configuration
using         ElementA    = cutlass::half_t;                                // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = cutlass::half_t;                                // Element type for B matrix operand
using         LayoutB     = cutlass::layout::ColumnMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C matrix configuration
using         ElementC    = cutlass::half_t;                                // Element type for C and D matrix operands
using         LayoutC     = cutlass::layout::ColumnMajor;                   // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

// D matrix configuration
using         ElementD    = ElementC;
using         LayoutD     = LayoutC;
constexpr int AlignmentD  = AlignmentC;

// Core kernel configurations
using ElementAccumulator  = cutlass::half_t;                                // Element type for internal accumulation
using ElementCompute      = cutlass::half_t;                                // Element type for epilogue computation
using ArchTag             = cutlass::arch::Sm90;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_128,_256,_64>;                           // Threadblock-level tile size
using ClusterShape        = Shape<_1,_2,_1>;                                // Shape of the threadblocks in a cluster

using KernelSchedule      = cutlass::gemm::KernelTmaWarpSpecializedPingpong;
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

// We're going to use the single-device GEMM as reference
using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

// Instantiate Distributed GEMM kernel
using DistGemmKernel = cutlass::distributed::kernel::DistributedGemmKernelWrapper<
  GemmKernel,
  DistSchedule
>;
using DistGemm = cutlass::distributed::device::DistributedGemmUniversalAdapter<DistGemmKernel>;

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

using HostTensorA = typename cutlass::HostTensor<ElementA, LayoutA>;
using HostTensorB = typename cutlass::HostTensor<ElementB, LayoutB>;
using HostTensorC = typename cutlass::HostTensor<ElementC, LayoutC>;
using HostTensorD = typename cutlass::HostTensor<ElementD, LayoutD>;

// Reference GEMM tensors
HostTensorA tensor_A;
HostTensorB tensor_B;
HostTensorC tensor_C;
HostTensorD tensor_D;
HostTensorD tensor_ref_D;

// DistGEMM tensors (multi-device)
HostTensorA tensor_A_arr[TP_];
HostTensorB tensor_B_arr[TP_];
HostTensorD tensor_C_arr[TP_];
HostTensorD tensor_D_arr[TP_];

#endif // (defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED) &&
```

**EN**: This block defines the baseline Hopper GEMM: half-precision A/B/C/D, a 128x256x64 tile, a 1x2x1 cluster, a TMA warp-specialized ping-pong mainloop, and a TMA warp-specialized epilogue. That concrete `GemmKernel` is then reused twice: once as a normal `GemmUniversalAdapter` for reference, and once inside `DistributedGemmKernelWrapper`/`DistributedGemmUniversalAdapter` so the same kernel body becomes a distributed collective.

**CN**: 这里先定义基础的 Hopper GEMM：A/B/C/D 均为 half，线程块 tile 为 128x256x64，cluster 为 1x2x1，主循环采用 TMA warp-specialized ping-pong，epilogue 采用 TMA warp-specialized。随后同一个 `GemmKernel` 被复用两次：一次包装成普通 `GemmUniversalAdapter` 作为单卡参考；一次放进 `DistributedGemmKernelWrapper`/`DistributedGemmUniversalAdapter` 中，变成分布式 collective。

### CLI options, result container, and global tensor state (Lines 262-348)

```cpp
// Command line options parsing
struct Options {

  bool help = false;

  float alpha = 1.f, beta = 0.f;
  int iterations = 100;
  int warmup_iterations = 10;
  int m = 16384, n = 106496, k = 16384, l = 1;
  float eps = 0.f;

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
    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);
    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("warmup-iterations", warmup_iterations);
    cmd.get_cmd_line_argument("eps", eps);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "65_distributed_gemm\n\n"
      << "  Hopper Distributed GEMM (DistGEMM). \n"
      << "  For more details please refer to the source file.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   Sets the L extent (batch) of the GEMM (default: 1)\n"
      << "  --alpha=<f32>               Epilogue scalar alpha (default: 1.0)\n"
      << "  --beta=<f32>                Epilogue scalar beta (default: 0.0)\n"
      << "  --iterations=<int>          Number of profiling iterations to perform (default: 100)\n"
      << "  --warmup-iterations=<int>   Number of warmup iterations prior to profiling (default: 10)\n"
      << "  --eps=<f32>                 Threshold for error compared to reference " 
      << "GEMM (default: 0.0)\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << "65_distributed_gemm" << " --m=16384 --n=106496 --k=16384 \n\n";

    return out;
  }

  /// Compute performance in TFLOP/s
  double tflops(double runtime_s) const {

    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * m * n * k * l / TP_;
    double tflop = double(flop) / double(1.0e12);
    return tflop / runtime_s;
  }
};

/// Result structure
struct Result {
  double avg_runtime_ms;
  double tflops;
  cutlass::Status status;
  cudaError_t error;
  bool passed;

  Result(
    double avg_runtime_ms = 0,
    double tflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cudaError_t error = cudaSuccess)
  :
    avg_runtime_ms(avg_runtime_ms), tflops(tflops), status(status), error(error), passed(false)
  {}

};

#if (__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 6))
```

**EN**: The `Options` struct exposes problem size, alpha/beta, warmup/profiling counts, and error tolerance. Its `tflops()` method intentionally divides the total flop count by `TP_`, because each device only executes its local shard. `Result` stores runtime, throughput, and pass/fail state for later reporting.

**CN**: `Options` 结构体暴露了问题规模、alpha/beta、预热/测试迭代数以及误差阈值。`tflops()` 会把总 FLOP 除以 `TP_`，因为每张卡只执行自己的局部分片。`Result` 则统一保存运行时间、吞吐率和正确性状态，供后续打印。

### Tensor initialization and peer-access orchestration (Lines 354-460)

```cpp
/// Helper to initialize a block of device data
template <typename Element, typename Layout>
bool initialize_tensor(
  cutlass::TensorView<Element, Layout> view,
  uint64_t seed,
  bool is_device_tensor = false) {

  double scope_max, scope_min;
  int bits = cutlass::sizeof_bits<Element>::value;

  if (bits == 1) {
    scope_max = 2;
    scope_min = 0;
  }
  else if (bits <= 16) {
    scope_max = 2;
    scope_min = -2;
  }
  else {
    scope_max = 8;
    scope_min = -8;
  }

  if (is_device_tensor) {
    using Real = typename cutlass::RealType<Element>::Type;
    cutlass::reference::device::TensorFillRandomUniform(
      view, seed, static_cast<Real>(scope_max), static_cast<Real>(scope_min), 0);
    cudaDeviceSynchronize();
  } else {
    cutlass::reference::host::TensorFillRandomUniform(
      view, seed, scope_max, scope_min, 0);
  }

  return true;
}

/// Initialize operands to be used in the GEMM and reference GEMM
void initialize(const Options &options) {
  auto problem_shape = cute::make_tuple(options.m, options.n, options.k, options.l);

  // Setup (reference) GEMM tensors
  auto shape_A = cute::select<0,2,3>(problem_shape);
  auto shape_B = cute::select<1,2,3>(problem_shape);
  auto shape_C = cute::select<0,1,3>(problem_shape);
  auto shape_D = cute::select<0,1,3>(problem_shape);

  stride_A = cutlass::make_cute_packed_stride(StrideA{}, shape_A);
  stride_B = cutlass::make_cute_packed_stride(StrideB{}, shape_B);
  stride_C = cutlass::make_cute_packed_stride(StrideC{}, shape_C);
  stride_D = cutlass::make_cute_packed_stride(StrideD{}, shape_D);

  auto a_coord = cutlass::make_Coord(size<2>(shape_A)*size<0>(shape_A), size<1>(shape_A));
  auto b_coord = cutlass::make_Coord(size<2>(shape_B)*size<0>(shape_B), size<1>(shape_B));
  auto c_coord = cutlass::make_Coord(size<2>(shape_C)*size<0>(shape_C), size<1>(shape_C));

  tensor_A.resize(a_coord);
  tensor_B.resize(b_coord);
  tensor_C.resize(c_coord);
  tensor_D.resize(c_coord);
  tensor_ref_D.resize(c_coord);

  initialize_tensor(tensor_A.device_view(), seed + 2022, /* is_device_tensor = */ true);
  initialize_tensor(tensor_B.device_view(), seed + 2023, /* is_device_tensor = */ true);
  initialize_tensor(tensor_C.device_view(), seed + 2024, /* is_device_tensor = */ true);

  tensor_A.sync_host();
  tensor_B.sync_host();
  tensor_C.sync_host();
  tensor_D.sync_host();
  tensor_ref_D.sync_host();

  // Set up DistGEMM tensors
  auto local_shape_A = DistSchedule::get_local_a_shape(problem_shape);
  auto local_shape_B = DistSchedule::get_local_b_shape(problem_shape);
  auto local_shape_C = DistSchedule::get_local_c_shape(problem_shape);
  auto local_shape_D = DistSchedule::get_local_d_shape(problem_shape);

  auto a_coord_device = cutlass::make_Coord(size(local_shape_A), 1);
  auto b_coord_device = cutlass::make_Coord(size(local_shape_B), 1);
  auto c_coord_device = cutlass::make_Coord(size(local_shape_C), 1);

  int primary_device_idx;
  CUDA_CHECK(cudaGetDevice(&primary_device_idx));

  // Enable any-to-any access
  for (int device_idx = 0; device_idx < TP_; ++device_idx) {
    int can_access;
    CUDA_CHECK(cudaSetDevice(device_idx));
    for (int peer_idx = 0; peer_idx < TP_; ++peer_idx) {
      if (peer_idx != device_idx) {
        CUDA_CHECK(cudaDeviceCanAccessPeer(&can_access, device_idx, peer_idx));
        if (not can_access) {
          std::cerr << "FAILURE: Device " << device_idx << " can't access device " << peer_idx << "." <<
            std::endl;
          exit(EXIT_FAILURE);
        }
        CUDA_CHECK(cudaDeviceEnablePeerAccess(peer_idx, 0));
      }
    }

    tensor_A_arr[device_idx].resize(a_coord_device);
    tensor_B_arr[device_idx].resize(b_coord_device);
    tensor_C_arr[device_idx].resize(c_coord_device);
    tensor_D_arr[device_idx].resize(c_coord_device);
  }
  CUDA_CHECK(cudaSetDevice(primary_device_idx));
}
```

**EN**: `initialize_tensor()` chooses random ranges based on element width. `initialize()` allocates global reference tensors, derives packed CuTe strides, and then asks the distributed schedule for each device’s local A/B/C/D shapes. The most important orchestration step is the nested peer-access loop: every device checks and enables access to every other device, which is required because DistGEMM moves data across GPUs through the any-to-any NVLink topology.

**CN**: `initialize_tensor()` 会根据元素位宽选择随机初始化范围。`initialize()` 先为全局参考张量分配空间并推导 packed stride，再通过分布式调度查询每张卡对应的本地 A/B/C/D 形状。最关键的编排步骤是双层 peer-access 循环：每张卡都要检查并开启对其他所有设备的访问权限，因为 DistGEMM 需要依赖任意互连的 NVLink 拓扑在 GPU 间搬运数据。

### Mapping global tensors into local DistGEMM arguments (Lines 462-549)

```cpp
/// Commandline options -> Gemm/DistGemm Arguments
using GemmArguments = typename Gemm::Arguments;
GemmArguments gemm_args_from_options(const Options &options) {
  typename Gemm::Arguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,
    {options.m, options.n, options.k, options.l},
    {tensor_A.device_data(), stride_A, tensor_B.device_data(), stride_B},
    {
      {static_cast<ElementCompute>(options.alpha), static_cast<ElementCompute>(options.beta)},
      tensor_C.device_data(), stride_C,
      tensor_ref_D.device_data(), stride_D
    }
  };

  return arguments;
}

using DistGemmArguments = typename DistGemm::Arguments;
DistGemmArguments dist_gemm_args_from_options(
    const Options &options,
    int device_idx,
    cudaStream_t stream) {

  auto problem_shape = cute::make_tuple(options.m, options.n, options.k, options.l);

  auto global_A = cute::make_tensor(tensor_A.device_data(),
      cute::make_layout(cute::make_shape(options.m, options.k, options.l), stride_A));
  auto global_B = cute::make_tensor(tensor_B.device_data(),
      cute::make_layout(cute::make_shape(options.n, options.k, options.l), stride_B));
  auto global_C = cute::make_tensor(tensor_C.device_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_C));

  auto global_A_device_slice = DistSchedule::get_device_slice_A(global_A, device_idx);
  auto global_B_device_slice = DistSchedule::get_device_slice_B(global_B, device_idx);
  auto global_C_device_slice = DistSchedule::get_device_slice_C(global_C, device_idx);

  auto local_shape_A = DistSchedule::get_local_a_shape(problem_shape);
  auto local_shape_B = DistSchedule::get_local_b_shape(problem_shape);
  auto local_shape_C = DistSchedule::get_local_c_shape(problem_shape);
  auto local_shape_D = DistSchedule::get_local_d_shape(problem_shape);

  auto local_stride_A = cutlass::make_cute_packed_stride(StrideA{}, local_shape_A);
  auto local_stride_B = cutlass::make_cute_packed_stride(StrideB{}, local_shape_B);
  auto local_stride_C = cutlass::make_cute_packed_stride(StrideC{}, local_shape_C);
  auto local_stride_D = cutlass::make_cute_packed_stride(StrideD{}, local_shape_D);

  auto local_A = cute::make_tensor(
      tensor_A_arr[device_idx].device_data(),
      make_layout(local_shape_A, local_stride_A));
  auto local_B = cute::make_tensor(
      tensor_B_arr[device_idx].device_data(),
      make_layout(local_shape_B, local_stride_B));
  auto local_C = cute::make_tensor(
      tensor_C_arr[device_idx].device_data(),
      make_layout(local_shape_C, local_stride_C));
  auto local_D = cute::make_tensor(
      tensor_D_arr[device_idx].device_data(),
      make_layout(local_shape_D, local_stride_D));

  // Copy over tensor tiles for the first iteration
  cutlass::device_copy(global_A_device_slice, local_A, stream);
  cutlass::device_copy(global_B_device_slice, local_B, stream);
  cutlass::device_copy(global_C_device_slice, local_C, stream);

  DistGemmArguments arguments{
    cutlass::gemm::GemmUniversalMode::kGemm,                                       // mode
    problem_shape,                                                                 // problem shape
    {
      reinterpret_cast<const ElementA*>(local_A.data()),
      local_A.stride(),
      reinterpret_cast<const ElementB*>(local_B.data()),
      local_B.stride()
    },                                                                             // mainloop
    {
      {                                                                            // epilogue.thread
        static_cast<ElementCompute>(options.alpha),
        static_cast<ElementCompute>(options.beta)
      },
      reinterpret_cast<const ElementC*>(local_C.data()),
      local_C.stride(),
      reinterpret_cast<const ElementD*>(local_D.data()),
      local_D.stride(),
    },                                                                             // epilogue
    {},                                                                            // hw_info
    {}                                                                             // scheduler
  };

  return arguments;
```

**EN**: The single-GPU reference path is straightforward, but `dist_gemm_args_from_options()` is the heart of the distributed mapping. It materializes CuTe tensors over the global A/B/C buffers, uses the schedule to extract the slice owned by `device_idx`, creates local tensors with locally packed strides, copies the first iteration’s slices into per-device buffers, and then packages those local buffers into distributed mainloop and epilogue arguments.

**CN**: 单卡参考路径比较直接，而 `dist_gemm_args_from_options()` 才是分布式映射的核心。它先在全局 A/B/C 缓冲区上构造 CuTe tensor，再用调度器提取 `device_idx` 对应的切片，随后用本地 packed stride 构造本地 tensor，把首轮需要的数据复制到各自设备缓冲区中，最后将这些局部缓冲区打包成分布式 mainloop 与 epilogue 参数。

### Collecting local outputs and validating correctness (Lines 552-594)

```cpp
// Gathers results, moves back to the original full-sized D tensor on the primary device.
void gather_results(const Options &options, int device_idx, cudaStream_t stream = nullptr) {

  auto problem_shape = cute::make_tuple(options.m, options.n, options.k, options.l);

  // Global dest
  auto global_D = cute::make_tensor(tensor_D.device_data(),
      cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_D));
  auto global_D_device_slice = DistSchedule::get_device_slice_D(global_D, device_idx);

  // Device_idx local dest
  auto local_shape_D = DistSchedule::get_local_d_shape(problem_shape);
  auto local_stride_D = cutlass::make_cute_packed_stride(StrideD{}, local_shape_D);
  auto local_D = cute::make_tensor(
      tensor_D_arr[device_idx].device_data(),
      make_layout(local_shape_D, local_stride_D)
  );

  // Copy to global dest
  cutlass::device_copy(local_D, global_D_device_slice, stream);
}

bool verify(const Options &options) {
  tensor_D.sync_host();
  tensor_ref_D.sync_host();

  bool passed = false;
  if (options.eps == 0.f) {
    passed = cutlass::reference::host::TensorEquals(tensor_ref_D.host_view(), tensor_D.host_view());
  } else {
    double err = cutlass::reference::host::TensorRelativeErrorMetric(
      tensor_D.host_view(),
      tensor_ref_D.host_view());
    passed = err < 1e-5;
  }

  if (options.m <= 64 && options.n <= 64) {
    std::cout << "GEMM output:\n" << tensor_D.host_view() << "\n\n";
    std::cout << "Reference output:\n" << tensor_ref_D.host_view() << "\n\n";
  }

  return passed;
}
```

**EN**: `gather_results()` inverts the slicing logic: it interprets the full D tensor on the primary device, takes the destination slice for one device, and copies that device’s local D tile back into its global position. `verify()` then compares the gathered distributed result against the single-device reference either exactly or with a relative error metric.

**CN**: `gather_results()` 执行的是切分逻辑的反向过程：它把主设备上的完整 D 解释成全局 tensor，定位到某个设备对应的目标切片，再把该设备的本地 D tile 回填到全局位置。之后 `verify()` 会把汇总后的分布式结果与单卡参考结果做逐元素精确比较，或者在给定阈值下做相对误差比较。

### Reference GEMM, per-device streams, workspaces, and distributed initialization (Lines 596-718)

```cpp
/// Execute a given example GEMM computation
int run(Options &options) {

  int primary_device_idx;
  cudaError_t device_get_result = cudaGetDevice(&primary_device_idx);
  if (device_get_result != cudaSuccess) {
    throw std::runtime_error("cudaGetDevice() failed");
  }

  initialize(options);

  // Reference single-GPU GEMM
  Gemm reference_gemm;
  cutlass::device_memory::allocation<uint8_t> reference_workspace;

  auto reference_arguments = gemm_args_from_options(options);
  size_t reference_workspace_size = Gemm::get_workspace_size(reference_arguments);
  reference_workspace = cutlass::device_memory::allocation<uint8_t>(reference_workspace_size);

  CUTLASS_CHECK(reference_gemm.can_implement(reference_arguments));
  CUTLASS_CHECK(reference_gemm.initialize(reference_arguments, reference_workspace.get()));
  CUTLASS_CHECK(reference_gemm.run());

  using ElementBarrier = typename DistGemm::ElementBarrier;
  using ElementFlag = typename DistGemmKernel::ElementFlag;

  // Set up per-device streams
  cudaStream_t stream_arr[TP_];

  for (int device_idx = 0; device_idx < TP_; ++device_idx) {
    CUDA_CHECK(cudaSetDevice(device_idx));

    // Create stream
    CUDA_CHECK(cudaStreamCreate(&stream_arr[device_idx]));
  }

  // Instantiate DistGEMM
  DistGemm dist_gemm_arr[TP_];  // Distributed GEMM array for multiple devices

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace_arr[TP_];
  cutlass::device_memory::allocation<uint8_t> exclusive_workspace_arr[TP_];

  // Cross-device workspace pointer array for gemm.initialize()
  void * workspace_ptr_arr[TP_];
  void * exclusive_workspace_ptr_arr[TP_];

  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  DistGemmArguments arguments_[TP_];

  for (int device_idx = 0; device_idx < TP_; ++device_idx) {
    CUDA_CHECK(cudaSetDevice(device_idx));

    arguments_[device_idx] = dist_gemm_args_from_options(options, device_idx, stream_arr[device_idx]);

    // Using the arguments, query for extra workspace required for matrix multiplication computation
    size_t workspace_size = DistGemm::get_workspace_size(arguments_, device_idx);
    size_t exclusive_workspace_size = DistGemm::get_exclusive_workspace_size();

    workspace_arr[device_idx] = cutlass::device_memory::allocation<uint8_t>(workspace_size);
    exclusive_workspace_arr[device_idx] = cutlass::device_memory::allocation<uint8_t>(exclusive_workspace_size);

    // Throw workspace pointers into arrays for gemm.initialize()
    workspace_ptr_arr[device_idx] = workspace_arr[device_idx].get();
    exclusive_workspace_ptr_arr[device_idx] = exclusive_workspace_arr[device_idx].get();

    // Zero out exclusive workspace
    cudaMemsetAsync(exclusive_workspace_ptr_arr[device_idx], 0, exclusive_workspace_size, stream_arr[device_idx]);

    cudaDeviceSynchronize();
  }

  for (int device_idx = 0; device_idx < TP_; ++device_idx) {
    CUDA_CHECK(cudaSetDevice(device_idx));

    // Check if the problem size is supported or not
    CUTLASS_CHECK(dist_gemm_arr[device_idx].can_implement(arguments_[device_idx]));

#if defined(CUTLASS_ENABLE_GDC_FOR_SM90)
    bool launch_with_pdl = true;
#else
    bool launch_with_pdl = false;
#endif

    // Initialize CUTLASS kernel with arguments and workspace pointer
    CUTLASS_CHECK(dist_gemm_arr[device_idx].initialize(
          arguments_,
          workspace_ptr_arr,
          exclusive_workspace_ptr_arr,
          device_idx,
          stream_arr[device_idx],
          launch_with_pdl
          ));

    cudaDeviceSynchronize();
  }

  // Correctness / Warmup iteration
  std::cout << std::endl << "  running DistGEMM..." << std::endl;

  for (int device_idx = 0; device_idx < TP_; ++device_idx) {
    CUDA_CHECK(cudaSetDevice(device_idx));
    CUTLASS_CHECK(dist_gemm_arr[device_idx].run(stream_arr[device_idx]));
  }
  for (int device_idx = 0; device_idx < TP_; ++device_idx) {
    CUDA_CHECK(cudaStreamSynchronize(stream_arr[device_idx]));
    CUDA_CHECK(cudaGetLastError());
    gather_results(options, device_idx);
  }

  std::cout << "  running DistGEMM finished without runtime errors" << std::endl;

  //// Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;

  result.passed = verify(options);

  std::cout << std::endl << "  Disposition (eps: " << options.eps << "): " << 
    (result.passed ? "Passed" : "Failed") << std::endl;

  if (!result.passed) {
    exit(-1);
  }
```

**EN**: `run()` first executes a normal Hopper GEMM to build the gold reference. It then creates one CUDA stream per device, allocates both shared workspace and exclusive workspace per rank, zeroes the exclusive workspace, and calls `DistGemm::initialize()` with arrays of arguments and workspace pointers. That array-based initialize step is the critical orchestration interface: each rank learns not only its own buffers, but also the cross-rank state needed to participate in the collective launch.

**CN**: `run()` 会先执行一个普通 Hopper GEMM 生成金标准参考结果。然后它为每张卡创建一个 CUDA stream，为每个 rank 分配共享工作区和独占工作区，清零独占工作区，并通过参数数组和工作区指针数组调用 `DistGemm::initialize()`。这个“数组式初始化”就是分布式编排的关键接口：每个 rank 不仅知道自己的缓冲区，也同时拿到了参与 collective 启动所需的跨 rank 状态。

### Warmup, distributed timing, and per-rank performance reporting (Lines 721-803)

```cpp
  if (options.iterations > 0) {
    float elapsed_ms = 0.f;

    // Warmup
    std::cout << "  Warming up for " << options.warmup_iterations << " iterations." << std::endl;
    for (int warmup_iter = 0; warmup_iter < options.warmup_iterations; ++warmup_iter) {
      for (int device_idx = 0; device_idx < TP_; ++device_idx) {
        CUDA_CHECK(cudaSetDevice(device_idx));
        CUTLASS_CHECK(dist_gemm_arr[device_idx].run(stream_arr[device_idx]));
      }
    }

    for (int device_idx = 0; device_idx < TP_; ++device_idx) {
      CUDA_CHECK(cudaSetDevice(device_idx));
      CUDA_CHECK(cudaStreamSynchronize(stream_arr[device_idx]));
    }

    CUDA_CHECK(cudaSetDevice(primary_device_idx));

    // Benchmark
    std::cout << "  Profiling for " << options.iterations << " iterations." << std::endl;
    using AtomicBoolean = cuda::atomic<bool>;
    AtomicBoolean* atomic_flag_ptr;
    CUDA_CHECK(cudaHostAlloc(&atomic_flag_ptr, sizeof(AtomicBoolean), cudaHostAllocPortable));
    atomic_flag_ptr->store(false);

    cutlass::DistGpuTimer<TP_> timer;

    for (int device_idx = 0; device_idx < TP_; ++device_idx) {
      CUDA_CHECK(cudaSetDevice(device_idx));
      cutlass::delay_kernel<<<1, 1, 0, stream_arr[device_idx]>>>(atomic_flag_ptr);
      CUDA_CHECK(cudaGetLastError());
    }

    for (int device_idx = 0; device_idx < TP_; ++device_idx) {
      timer.start(device_idx, stream_arr[device_idx]);
    }

    atomic_flag_ptr->store(true);

    for (int profile_iter = 0; profile_iter < options.iterations; ++profile_iter) {
      for (int device_idx = 0; device_idx < TP_; ++device_idx) {
        CUDA_CHECK(cudaSetDevice(device_idx));
        CUTLASS_CHECK(dist_gemm_arr[device_idx].run(stream_arr[device_idx]));
      }
    }

    for (int device_idx = 0; device_idx < TP_; ++device_idx) {
      CUDA_CHECK(cudaSetDevice(device_idx));
      timer.stop(device_idx, stream_arr[device_idx]);
    }

    CUDA_CHECK(cudaSetDevice(primary_device_idx));

    for (int device_idx = 0; device_idx < TP_; ++device_idx) {
      elapsed_ms = max(elapsed_ms, timer.elapsed_millis(device_idx));
    }

    // Compute average runtime and TFLOPs.
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    double avg_runtime_s = (double)(result.avg_runtime_ms / 1000.0);
    result.tflops = options.tflops(avg_runtime_s);

    auto [local_M, local_N, local_K, local_L] = DistSchedule::get_local_gemm_shape(
        cute::make_tuple(options.m, options.n, options.k, options.l));

    std::cout << std::endl;
    std::cout << "  TP: " << TP::value << std::endl;
    std::cout << "  Problem Size: " << 
      options.m << " x " << 
      options.n << " x " << 
      options.k << " x " << 
      options.l << std::endl;
    std::cout << "  Local GEMM Problem Size: " << 
      local_M << " x " << 
      local_N << " x " << 
      local_K << " x " << 
      local_L<< std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  TFLOPS: " << result.tflops << std::endl;
  }

  return 0;
```

**EN**: The profiling path launches warmup iterations on every device, then uses a host-pinned atomic flag plus `cutlass::delay_kernel` to align start times across GPUs. `DistGpuTimer<TP_>` records each device independently, and the example reports the maximum elapsed time across devices as the distributed runtime. It also prints both global and local problem shapes so readers can see what each rank actually computed.

**CN**: 性能测试阶段会先在所有设备上执行预热，然后借助 host-pinned 原子标志和 `cutlass::delay_kernel` 对齐各 GPU 的起始时刻。`DistGpuTimer<TP_>` 为每张卡分别计时，而示例取所有设备中的最大耗时作为分布式运行时间。同时，它还会打印全局问题规模和本地 GEMM 规模，让读者明确每个 rank 实际计算了什么。

### Runtime guards and program entrypoint (Lines 810-869)

```cpp
int main(int argc, char const **args) {

  // CUTLASS must be compiled with CUDA Toolkit 12.6 or newer to run this example
  // and must have compute capability at least 90.
  // Some necessary cuda graph APIs were only introduced in CUDA 12.6.
  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 6)) {
    std::cerr << "This example requires CUDA 12.6 or newer." << std::endl;
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  int num_devices;
  CUDA_CHECK(cudaGetDeviceCount(&num_devices));
  if (num_devices < TP_) {
    std::cerr << "Distributed GEMM is compiled with TP = " << TP::value << ", but " << 
      "found only " << num_devices << " devices." <<
      std::endl;
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
      << "This example requires a GPU of NVIDIA's Hopper Architecture "
      << "(compute capability 90)." << std::endl;
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

#if ((__CUDACC_VER_MAJOR__ > 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 6)))
  run(options);
#else
    std::cerr
      << "This example must be compiled with `sm90a` and CUDA Toolkit 12.6 or later." << std::endl;
    return 0;
#endif

  return 0;
}
```

**EN**: The entrypoint enforces the architectural contract: CUDA 12.6+, at least eight visible GPUs, and Hopper SM90 capability. If those conditions are met it parses the CLI and dispatches to `run(options)`. Returning zero on unsupported systems keeps the example buildable in broader test matrices without treating unsupported hardware as a failure.

**CN**: 入口函数负责执行运行时约束检查：CUDA 12.6 及以上、至少 8 张可见 GPU、并且设备必须是 Hopper SM90。满足条件后才解析命令行并调用 `run(options)`。在不支持的平台上返回 0，而不是报错退出，是为了让示例能在更宽泛的测试矩阵中正常编译通过。

---

## Key Concepts / 关键概念

- **EN**: Distributed GEMM orchestration here is schedule-driven: the schedule owns local/global shape derivation, tensor slicing, and result gathering semantics.
  **CN**: 这里的分布式 GEMM 编排是“调度驱动”的：本地/全局形状推导、张量切片方式以及结果回收语义都由调度器统一定义。
- **EN**: The same `GemmKernel` is reused for both reference and distributed execution, which keeps kernel math constant while changing only the orchestration layer.
  **CN**: 同一个 `GemmKernel` 同时服务于单卡参考和分布式执行，因此保持了内核数学逻辑不变，只替换了外层编排层。
- **EN**: Peer access plus per-device streams/workspaces are mandatory because ranks exchange and coordinate operand/dataflow state across GPUs.
  **CN**: 必须启用 peer access 并为每张卡准备独立 stream 与工作区，因为各个 rank 需要跨 GPU 交换并协调操作数与数据流状态。
- **EN**: Distributed performance is measured by the slowest participating rank, so the example reports the max timer value across GPUs.
  **CN**: 分布式性能受最慢 rank 限制，因此示例最终取各 GPU 计时中的最大值作为整体运行时间。

## Dependencies / 依赖项

- **EN**: `cutlass/experimental/distributed/device/dist_gemm_universal_wrapper.hpp`, `.../kernel/dist_gemm_kernel_wrapper.hpp`, and `.../schedules/dist_gemm_1d_schedules.hpp` provide the experimental distributed runtime, kernel wrapper, and sharding policy.
  **CN**: `cutlass/experimental/distributed/device/dist_gemm_universal_wrapper.hpp`、`.../kernel/dist_gemm_kernel_wrapper.hpp` 与 `.../schedules/dist_gemm_1d_schedules.hpp` 提供了实验性的分布式运行时、内核封装器和分片策略。
- **EN**: `cute/tensor.hpp` and packed-stride helpers are used to materialize both global tensors and per-device local tiles in a uniform way.
  **CN**: `cute/tensor.hpp` 以及 packed stride 工具负责统一描述全局 tensor 和各设备局部 tile。
- **EN**: `helper.h` and `dist_gemm_helpers.h` supply CUDA/CUTLASS checking helpers and distributed example support utilities.
  **CN**: `helper.h` 与 `dist_gemm_helpers.h` 提供 CUDA/CUTLASS 的检查宏以及分布式示例需要的辅助能力。
- **EN**: CUDA peer-access APIs, streams, and host-pinned synchronization primitives are part of the orchestration contract, not optional conveniences.
  **CN**: CUDA 的 peer-access API、stream 与 host-pinned 同步原语属于编排协议的一部分，并非可有可无的附加功能。
