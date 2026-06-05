# depthwise_simt_conv2dfprop.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/46_depthwise_simt_conv2dfprop/depthwise_simt_conv2dfprop.cu`  
**Purpose / 用途**: Depthwise SIMT Conv2d forward-propagation example This example demonstrates how to instantiate and execute a CUTLASS depthwise 2D forward convolution kernel using SIMT math on NHWC half-precision tensors. It is a complete, self-contained benchmark/validation driver: kernel type aliases, command-line parsing, tensor setup, kernel launch, optional reference checking, optional workspace dump, and timing all live in one file. / Depthwise SIMT Conv2d 前向传播示例 这个示例展示了如何在 NHWC 的 half 精度张量上，使用 SIMT 数学路径实例化并执行 CUTLASS depthwise 2D 前向卷积 kernel。它是一个完整自包含的 benchmark/校验驱动：kernel 类型别名、命令行解析、张量准备、kernel 启动、可选参考校验、可选 workspace 导出以及计时逻辑都集中在同一个文件里。

---

## Line-by-Line Analysis / 逐行分析

### Logical Block 1 / 逻辑块 1 — lines 32-76

```cpp
/**
This example shows how to run depthwise 2d convolution kernels using functions and data structures
provided by CUTLASS using SIMT instruction;

There are 3 types of implementations of depthwise 2d convoltion
  1. kAnalytic
    Implicit gemm 2d convoltion algorithm.
  2. kOptimized
    An optimized algorithm and supports arbitrary stride and dilation.
  3. kFixedStrideDilation
    An optimized algorithm with fixed stride and dilation to reduce the runtime computation and do
more optimizations.

In general, the perf of kFixedStrideDilation would be better than kOptimized. However, if the filter
size, stride or dilation is large, it would encounter register spilling and may hurt the perf. If
in this case, please use kOptimized.

For kOptimized and kFixedStrideDilation, in order to fully utilize GPU hardware resources and achieve
better perf, when the output tensor size is large, splitk should be enabled to achieve better perf.

In this example, it demonstrates how to construct and run a FixedStrideDilation depthwise 2d
convolution kernel.
*/

#include <iostream>
#include <fstream>
#include <sstream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"
#include "cutlass/conv/kernel/default_depthwise_fprop.h"
#include "cutlass/conv/device/implicit_gemm_convolution.h"
#include "cutlass/conv/device/direct_convolution.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/convolution.h"
#include "cutlass/util/tensor_view_io.h"

#include "helper.h"
```

**EN**: The introductory comment explains the three depthwise convolution iterator algorithms supported by CUTLASS and explicitly states that this example chooses the fixed-stride/dilation variant for better performance when the static assumptions are reasonable. The includes then pull in both convolution-specific CUTLASS headers and generic utility/reference helpers.
**CN**: 开头注释说明了 CUTLASS 支持的三种 depthwise 卷积迭代算法，并明确指出本示例选择固定 stride/dilation 版本，以便在静态假设成立时获得更好的性能。随后包含的头文件同时覆盖了卷积专用组件和通用的工具/参考实现。

### Logical Block 2 / 逻辑块 2 — lines 78-171

```cpp
// The code section below describes datatype for input, output tensors and computation between
// elements
using ElementAccumulator = cutlass::half_t;      // Data type of accumulator
using ElementComputeEpilogue = cutlass::half_t;  // Data type of epilogue computation (alpha, beta)
using ElementInputA = cutlass::half_t;           // Data type of elements in input tensor
using ElementInputB = cutlass::half_t;           // Data type of elements in input tensor
using ElementOutput = cutlass::half_t;           // Data type of elements in output tensor

using LayoutInputA = cutlass::layout::TensorNHWC;
using LayoutInputB = cutlass::layout::TensorNHWC;
using LayoutOutput = cutlass::layout::TensorNHWC;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassSimt;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm60;

// This code section describes the groups a thread block will compute
constexpr int groups_per_cta = 64;

// This code section describes the output tile <N, O, P, Q> a thread block will compute
using ThreadBlockOutputShape = cutlass::conv::TensorNHWCShape<1, 8, 8, groups_per_cta>;

// This code section describes the filter shape <R, S>
using FilterShape = cutlass::MatrixShape<3, 3>;

// Threadblock tile shape
using ThreadblockShape =
    cutlass::gemm::GemmShape<ThreadBlockOutputShape::kNHW, groups_per_cta, FilterShape::kCount>;

// This code section describes tile size a warp will computes
// WarpShape::kM = P * Q the warps would process
// WarpShape::kN = groups_per_cta that the warps would process
// WarpShape::kK = filter_size that the warps would process
using WarpShape = cutlass::gemm::GemmShape<16, groups_per_cta, FilterShape::kCount>;

// This code section describes the size of MMA op
using InstructionShape = cutlass::gemm::GemmShape<1, 1, 1>;

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock =
    cutlass::conv::threadblock::DepthwiseDirect2dConvIdentityThreadblockSwizzle<
        1,
        ThreadBlockOutputShape::kN,
        ThreadBlockOutputShape::kH,
        ThreadBlockOutputShape::kW>;

// Number of pipelines you want to use
constexpr int NumStages = 4;

// This code section describe iterator algorithm selected is kFixedStrideDilation
static cutlass::conv::IteratorAlgorithm const IteratorAlgorithm =
    cutlass::conv::IteratorAlgorithm::kFixedStrideDilation;
using StrideShape = cutlass::MatrixShape<1, 1>;
using DilationShape = cutlass::MatrixShape<1, 1>;

constexpr int kEpilogueElementsPerAccess = 128 / cutlass::sizeof_bits<ElementOutput>::value;

// This code section describes the epilogue part of the kernel, we use default value
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementOutput,               // Data type of output matrix.
    kEpilogueElementsPerAccess,  // The number of elements per vectorized.
                                 // memory access. This becomes the vector width of
                                 // math instructions in the epilogue too.
    ElementAccumulator,          // Data type of accumulator
    ElementComputeEpilogue,      // Data type for alpha/beta in linear combination
    cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling>;  // Epilogue scaling operation.

using DepthwiseDirect2dConv = typename cutlass::conv::kernel::DefaultDepthwiseDirect2dConvFprop<
    ElementInputA,
    LayoutInputA,
    ElementInputB,
    LayoutInputB,
    ElementOutput,
    LayoutOutput,
    ElementAccumulator,
    MMAOp,
    SmArch,
    ThreadblockShape,
    ThreadBlockOutputShape,
    FilterShape,
    WarpShape,
    InstructionShape,
    EpilogueOp,
    SwizzleThreadBlock,
    NumStages,
    cutlass::arch::OpMultiplyAdd,
    IteratorAlgorithm,
    cutlass::conv::StrideSupport::kFixed,
    StrideShape,
    DilationShape>::Kernel;

using Direct2dConv = cutlass::conv::device::DirectConvolution<DepthwiseDirect2dConv>;
```

**EN**: This block defines the kernel configuration: half precision for inputs/outputs/accumulators, NHWC layout everywhere, SIMT math on SM60, one CTA covering an `N x P x Q x groups_per_cta` output tile, a fixed `3x3` filter, four pipeline stages, a standard linear-combination epilogue, and a `DefaultDepthwiseDirect2dConvFprop` kernel wrapped by `DirectConvolution`. In other words, this is the compile-time shape of the example.
**CN**: 这一段定义了 kernel 的编译期配置：输入/输出/累加器都使用 half，全部采用 NHWC 布局，在 SM60 上走 SIMT 数学路径；一个 CTA 负责一个 `N x P x Q x groups_per_cta` 输出 tile；滤波器固定为 `3x3`；使用四级流水；epilogue 为标准线性组合；最后再通过 `DirectConvolution` 包装 `DefaultDepthwiseDirect2dConvFprop`。换句话说，这一段描述了整个示例的“编译期形状”。

### Logical Block 3 / 逻辑块 3 — lines 176-247

```cpp
struct Options {
  bool help;
  cutlass::Tensor4DCoord input_size;
  cutlass::Tensor4DCoord filter_size;
  cutlass::Tensor4DCoord padding;
  cutlass::MatrixCoord conv_stride;
  cutlass::MatrixCoord dilation;
  int groups;
  int splitk;
  bool reference_check;
  bool measure_performance;
  int iterations;
  bool save_workspace;
  ElementComputeEpilogue alpha;
  ElementComputeEpilogue beta;
  std::string tag;

  Options()
      : help(false),
        input_size(1, 128, 128, 32),
        filter_size(32, 3, 3, 1),
        groups(32),
        padding(1, 1, 1, 1),
        conv_stride(1, 1),
        dilation(1, 1),
        reference_check(false),
        measure_performance(true),
        iterations(20),
        save_workspace(false),
        alpha(1),
        beta(0),
        splitk(1) {}

  // Verify the problem size is compatible with the CUTLASS Convolution implementation.
  bool valid() {
    //
    // CUTLASS attempts to load 128b vectors of cutlass::half_t (F16) elements. Consequently,
    // all pointers, strides, and tensor extents must be divisible by 8 elements.
    //
    int const kAlignment = 8;

    if ((input_size.c() % kAlignment) || (filter_size.n() % kAlignment)) {
      // misaligned tensors
      return false;
    }

    // depthwise conv
    if (groups != input_size.c()) {
      return false;
    }

    if (filter_size.n() != groups) {
      return false;
    }

    // Invalid padding
    if ((padding.h() != filter_size.h() / 2) || (padding.w() != filter_size.w() / 2)) {
      return false;
    }

    // Filter size passed through command line does not match filter size template parameter
    if (filter_size.h() != FilterShape::kRow || filter_size.w() != FilterShape::kColumn) {
      std::cerr << "Filter size passed in (" << filter_size.h() << "x" << filter_size.w() << ") "
                << "must match the FilterShape template parameter of the convolution "
                << "(" << FilterShape::kRow << "x" << FilterShape::kColumn << "). "
                << "To use the filter shape passed in, change the FilterShape template "
                << "parameter and recompile this example."
                << std::endl;
      return false;
    }

    return true;
```

**EN**: `Options` stores all runtime-tunable problem parameters and provides `valid()`. The validation routine encodes the practical constraints of this particular kernel instantiation: channel count must be vectorizable by 8, depthwise grouping must match input channels and filter count, padding must be the “same” padding implied by the filter, and runtime filter shape must exactly match the compile-time `FilterShape` alias.
**CN**: `Options` 保存所有可在运行时调整的问题参数，并提供 `valid()`。这个校验函数把当前 kernel 实例化的实际约束写死在代码里：通道数必须满足 8 元素向量化，depthwise 分组必须与输入通道数和滤波器个数一致，padding 必须符合该滤波器隐含的 same padding，运行时输入的滤波器尺寸也必须与编译期 `FilterShape` 完全一致。

### Logical Block 4 / 逻辑块 4 — lines 250-359

```cpp
  /// Updates input and filter sizes
  void update(cutlass::Tensor4DCoord input_size, cutlass::Tensor4DCoord filter_size) {
    this->input_size = input_size;
    this->filter_size = filter_size;

    padding.n() = filter_size.h() / 2;
    padding.h() = filter_size.h() / 2;
    padding.w() = filter_size.w() / 2;
    padding.c() = filter_size.w() / 2;
  }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    if (cmd.check_cmd_line_flag("ref-check")) {
      reference_check = true;
    }

    if (cmd.check_cmd_line_flag("perf-check")) {
      measure_performance = true;
    }

    if (cmd.check_cmd_line_flag("save-workspace")) {
      save_workspace = true;
    }

    cmd.get_cmd_line_argument("n", input_size.n());
    cmd.get_cmd_line_argument("h", input_size.h());
    cmd.get_cmd_line_argument("w", input_size.w());
    cmd.get_cmd_line_argument("c", input_size.c());

    cmd.get_cmd_line_argument("k", filter_size.n());
    cmd.get_cmd_line_argument("r", filter_size.h());
    cmd.get_cmd_line_argument("s", filter_size.w());

    cmd.get_cmd_line_argument("g", groups);

    filter_size.c() = 1;
    filter_size.n() = input_size.c();

    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);
    cmd.get_cmd_line_argument("splitk", splitk);

    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("tag", tag);

    int32_t padding_h = filter_size.h() / 2;
    int32_t padding_w = filter_size.w() / 2;
    padding = {padding_h, padding_h, padding_w, padding_w};
  }

  /// Prints the usage statement.
  std::ostream &print_usage(std::ostream &out) const {
    out << "46_depthwise_gemm_fprop example\n\n"
        << "  This example uses Ampere's Tensor Core operators on F16 data types to compute\n"
        << "  forward convolution on tensors of layout NHWC.\n\n"
        << "Options:\n\n"
        << "  --help               If specified, displays this usage statement.\n\n"
        << "  --n=<int>            Input tensor extent N\n"
        << "  --h=<int>            Input tensor extent H\n"
        << "  --w=<int>            Input tensor extent W\n"
        << "  --c=<int>            Input tensor extent C\n"
        << "  --k=<int>            Filter extent K\n"
        << "  --r=<int>            Filter extent R\n"
        << "  --s=<int>            Filter extent S\n\n"
        << "  --g=<int>            Groups\n\n"
        << "  --alpha=<float>      Epilogue scalar alpha\n"
        << "  --beta=<float>       Epilogue scalar beta\n\n"
        << "  --splitk=<int>       Enable splitK\n\n"
        << "  --ref-check          If set (true), reference check on the host is computed\n"
        << "  --perf-check         If set (true), performance is measured.\n"
        << "  --iterations=<int>   Number of profiling iterations to perform.\n"
        << "  --save-workspace     If set, workspace is written to a text file.\n"
        << "  --tag=<string>       String to replicate across the first column in the results "
           "table\n";

    out << "\n\nExamples:\n\n"
        << "$ ./examples/46_depthwise_simt_conv2dfprop/46_depthwise_simt_conv2dfprop  --n=32 "
           "--h=224 --w=224 --c=128 --k=128 --g=128 --r=3 --s=3\n\n"
        << "$ ./examples/46_depthwise_simt_conv2dfprop/46_depthwise_simt_conv2dfprop  --n=1 "
           "--h=224 --w=224 --c=32 --k=32 --g=32 --r=3 --s=3 --splitk=10 --ref-check\n\n";

    return out;
  }

  /// Computes the output tensor size (NPQK)
  cutlass::Tensor4DCoord output_size() const {
    return cutlass::Tensor4DCoord(
        input_size.n(),
        (input_size.h() + padding.n() + padding.h() - filter_size.h()) / conv_stride.row() + 1,
        (input_size.w() + padding.w() + padding.c() - filter_size.w()) / conv_stride.column() + 1,
        filter_size.n());
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {
    // Number of multiply-adds = NPQK * CRS
    int64_t fmas =
        output_size().product() * int64_t(filter_size.h() * filter_size.w() * filter_size.c());

    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};
```

**EN**: The remaining `Options` methods handle command-line parsing, usage printing, output-size derivation, and GFLOPs estimation. A notable depthwise-specific detail is that `parse()` forces `filter_size.c() = 1` and `filter_size.n() = input_size.c()`, which is exactly the channel mapping expected by depthwise convolution: one filter channel per input channel/group.
**CN**: `Options` 的剩余方法负责命令行解析、帮助信息打印、输出尺寸推导以及 GFLOPs 估算。一个很关键的 depthwise 细节是：`parse()` 会强制设定 `filter_size.c() = 1` 和 `filter_size.n() = input_size.c()`，这正是 depthwise 卷积的通道映射方式——每个输入通道/组对应一组滤波器。

### Logical Block 5 / 逻辑块 5 — lines 363-410

```cpp
struct Result {
  double runtime_ms;
  double gflops;
  cutlass::Status status;
  cutlass::Status reference_check;
  cudaError_t error;

  Result()
      : runtime_ms(0),
        gflops(0),
        status(cutlass::Status::kSuccess),
        reference_check(cutlass::Status::kInvalid),
        error(cudaSuccess) {}

  static std::ostream &print_header(std::ostream &out, Options const &options) {
    if (!options.tag.empty()) {
      out << "Name,";
    }

    out << "Layer,N,H,W,C,K,R,S,G,stride_h,stride_w,dilation_h,dilation_w,splitK,Runtime,GFLOPs";

    return out;
  }

  std::ostream &print(std::ostream &out, int idx, Options const &options) {
    if (!options.tag.empty()) {
      out << options.tag << ",";
    }

    cutlass::Tensor4DCoord output_size = options.output_size();
    out << "conv_" << idx << "," << options.input_size.n() << "," << options.input_size.h() << ","
        << options.input_size.w() << "," << options.input_size.c() << ","

        << options.filter_size.n() << "," << options.filter_size.h() << ","
        << options.filter_size.w() << ","

        << options.groups << "," << options.conv_stride.row() << "," << options.conv_stride.column()
        << ","

        << options.dilation.row() << "," << options.dilation.column() << ","

        << options.splitk << ","

        << runtime_ms << "," << gflops;

    return out;
  }
};
```

**EN**: The `Result` struct is a lightweight reporting container that stores runtime, throughput, CUTLASS status, reference status, and CUDA error code. Its `print_header()` and `print()` methods emit a CSV-like table, making the example useful both interactively and in batch benchmarking scripts.
**CN**: `Result` 结构体是一个轻量级报告容器，保存运行时间、吞吐率、CUTLASS 状态、参考校验状态和 CUDA 错误码。`print_header()` 与 `print()` 会输出类似 CSV 的表格，因此这个示例既适合交互式运行，也适合批量 benchmark 脚本调用。

### Logical Block 6 / 逻辑块 6 — lines 415-511

```cpp
  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(options.input_size);
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(options.filter_size);
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b_transpose(options.filter_size);
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c(options.output_size());
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d(options.output_size());
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_ref_d(options.output_size());

  //
  // Initialize tensors
  //

  // Fill tensor A on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_a.host_view(), 1, ElementInputA(5), ElementInputA(-6), 0);

  // Fill tensor B on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_b.host_view(), 1, ElementInputB(3), ElementInputB(-6), 0);

  // Fill tensor C on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_c.host_view(), 1, ElementOutput(5), ElementOutput(-6), 0);

  // Fill tensor D on host with zeros
  cutlass::reference::host::TensorFill(tensor_d.host_view());

  // Fill tensor D for reference on host with zeros
  cutlass::reference::host::TensorFill(tensor_ref_d.host_view());

  // Copy data from host to GPU
  tensor_a.sync_device();
  tensor_b.sync_device();
  tensor_b_transpose.sync_device();
  tensor_c.sync_device();
  tensor_d.sync_device();
  tensor_ref_d.sync_device();

  //
  // Define arguments for CUTLASS Convolution
  //

  cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation;

  // Split P*Q into multiple CTA
  int split_k_slices = options.splitk;

  // Construct Conv2dProblemSize with user defined output size
  cutlass::conv::Conv2dProblemSize problem_size(options.input_size,
                                                options.filter_size,
                                                options.padding,
                                                options.conv_stride,
                                                options.dilation,
                                                options.output_size(),
                                                mode,
                                                split_k_slices,
                                                options.groups);

  // Construct Direc2dConv::Argument structure with conv2d
  // problem size, data pointers, and epilogue values
  typename Direct2dConv::Arguments arguments{problem_size,
                                             tensor_a.device_ref(),
                                             tensor_b.device_ref(),
                                             tensor_c.device_ref(),
                                             tensor_d.device_ref(),
                                             {options.alpha, options.beta},
                                             tensor_b_transpose.device_ref()};

  //
  // Initialize CUTLASS Convolution
  //

  Direct2dConv implicit_gemm_op;

  size_t workspace_size = implicit_gemm_op.get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  result.status = implicit_gemm_op.can_implement(arguments);
  CUTLASS_CHECK(result.status);

  result.status = implicit_gemm_op.initialize(arguments, workspace.get());
  CUTLASS_CHECK(result.status);

  //
  // Launch initialized CUTLASS kernel
  //
  result.status = implicit_gemm_op();

  CUTLASS_CHECK(result.status);
```

**EN**: `profile_convolution()` begins by allocating host/device tensors for input, filter, an auxiliary transposed-filter buffer, source/output tensors, and a reference output. It initializes tensors with random data, constructs a `Conv2dProblemSize`, builds `Direct2dConv::Arguments`, allocates workspace, checks `can_implement()`, initializes the device operator, and launches it once. This is the core host-to-device execution flow of the depthwise example.
**CN**: `profile_convolution()` 一开始会为输入、滤波器、辅助转置滤波器缓冲区、源/输出张量以及参考输出分配 HostTensor/DeviceTensor。随后用随机数初始化张量，构造 `Conv2dProblemSize`，再填充 `Direct2dConv::Arguments`，申请 workspace，调用 `can_implement()`，初始化设备算子，并先启动一次 kernel。这正是整个 depthwise 示例的主机到设备执行主流程。

### Logical Block 7 / 逻辑块 7 — lines 517-552

```cpp
  if (options.reference_check) {
    std::cout << "Verification on host...\n";

    // Compute with reference implementation
    cutlass::reference::host::Conv2dFprop<
        ElementInputA,
        LayoutInputA,
        ElementInputB,
        LayoutInputB,
        ElementOutput,
        LayoutOutput,
        ElementComputeEpilogue,
        ElementAccumulator >(problem_size,
                             tensor_a.host_ref(),
                             tensor_b.host_ref(),
                             tensor_c.host_ref(),
                             tensor_ref_d.host_ref(),
                             options.alpha,
                             options.beta);

    // Check if output from CUTLASS kernel and reference kernel are equal or not
    tensor_d.sync_host();

    bool passed =
        cutlass::reference::host::TensorEquals(tensor_d.host_view(), tensor_ref_d.host_view());

    if (!passed) {
      result.reference_check = cutlass::Status::kErrorInternal;
      std::cout << "ERROR - results miscompared.\n";
    } else {
      result.reference_check = cutlass::Status::kSuccess;
      std::cout << "Passed.\n";
    }
  } else {
    result.reference_check = cutlass::Status::kInvalid;
  }
```

**EN**: When `--ref-check` is enabled, the example computes a host reference with `cutlass::reference::host::Conv2dFprop`, copies the device result back, and compares them with `TensorEquals`. This is the functional correctness path for the example.
**CN**: 当启用 `--ref-check` 时，示例会通过 `cutlass::reference::host::Conv2dFprop` 计算主机参考结果，把设备输出拷回主机，再用 `TensorEquals` 做比较。这就是该示例的功能正确性校验路径。

### Logical Block 8 / 逻辑块 8 — lines 554-576

```cpp
  if (options.save_workspace) {
    std::stringstream ss;

    ss << "46_depthwise_simt_conv2dfprop" << options.input_size.n() << "x" << options.input_size.h()
       << "x" << options.input_size.w() << "x" << options.input_size.c() << "_"
       << options.filter_size.n() << "x" << options.filter_size.h() << "x"
       << options.filter_size.w() << "x" << options.filter_size.c() << ".dat";

    std::ofstream output_workspace(ss.str());

    output_workspace << "Input = \n"
                     << tensor_a.host_view() << "\n\n"
                     << "Filters = \n"
                     << tensor_b.host_view() << "\n\n";

    if (options.reference_check) {
      output_workspace << "Reference = \n" << tensor_ref_d.host_view() << "\n\n";
    }

    output_workspace << "Computed = \n" << tensor_d.host_view() << std::endl;

    std::cout << "Results written to '" << ss.str() << "'." << std::endl;
  }
```

**EN**: The optional workspace dump writes input, filter, optional reference, and computed output tensors into a deterministic `.dat` file whose name encodes the tensor shapes. It is useful when manually inspecting a failing or unexpected configuration.
**CN**: 可选的 workspace 导出会把输入、滤波器、可选参考结果以及实际计算输出写入一个确定命名的 `.dat` 文件，文件名中还编码了张量形状。这在手工排查失败或异常配置时非常方便。

### Logical Block 9 / 逻辑块 9 — lines 582-639

```cpp
  if (options.measure_performance) {
    cudaEvent_t events[2];

    for (auto &event : events) {
      result.error = cudaEventCreate(&event);
      if (result.error != cudaSuccess) {
        std::cerr << "cudaEventCreate() failed: " << cudaGetErrorString(result.error) << std::endl;
        return result;
      }
    }

    // Record an event at the start of a series of convolution operations.
    result.error = cudaEventRecord(events[0]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Launch a sequence of implicit GEMM operations on the device
    for (int iteration = 0; iteration < options.iterations; ++iteration) {
      result.status = implicit_gemm_op();
      CUTLASS_CHECK(result.status);
    }

    // Record an event when the convolutions have been launched.
    result.error = cudaEventRecord(events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Wait for work on the device to complete.
    result.error = cudaEventSynchronize(events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventSynchronize() failed: " << cudaGetErrorString(result.error)
                << std::endl;
      return result;
    }

    // Measure elapsed runtime
    float runtime_ms = 0;
    result.error = cudaEventElapsedTime(&runtime_ms, events[0], events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Print average runtime and GFLOPs.
    result.runtime_ms = double(runtime_ms) / double(options.iterations);
    result.gflops = options.gflops(result.runtime_ms / 1000.0);

    // Cleanup
    for (auto event : events) {
      (void)cudaEventDestroy(event);
    }
  }

  return result;
```

**EN**: The performance block uses CUDA events to time repeated launches of the already-initialized operator and then computes average runtime and GFLOPs. Because initialization and validation are outside the timed loop, the reported number focuses on kernel execution rather than setup overhead.
**CN**: 性能测量部分通过 CUDA events 对已经初始化好的算子进行重复启动计时，然后计算平均运行时间和 GFLOPs。由于初始化和校验都在计时循环之外，最终报告更聚焦于 kernel 执行本身，而不是准备阶段的开销。

### Logical Block 10 / 逻辑块 10 — lines 644-680

```cpp
int main(int argc, char const **args) {
  bool notSupported = false;

  cudaDeviceProp props;
  CUDA_CHECK(cudaGetDeviceProperties(&props, 0));

  if (!(props.major >= 6)) {
    std::cerr << "Run on a machine with compute capability at least 60." << std::endl;
    notSupported = true;
  }

  if (notSupported) {
    return 0;
  }

  Options options;

  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  // Execute one problem size
  if (!options.valid()) {
    std::cerr << "Invalid problem." << std::endl;
    return -1;
  }

  Result result = profile_convolution(options);

  Result::print_header(std::cout, options) << std::endl;
  result.print(std::cout, 1, options) << std::endl;

  return 0;
}
```

**EN**: The `main()` function enforces the minimum architecture requirement (SM60+), parses options, prints help when requested, validates the problem, invokes `profile_convolution()`, and finally prints one result row. This makes the file a complete example program rather than just a library snippet.
**CN**: `main()` 会先检查最低架构要求（SM60+），解析命令行参数，按需打印帮助，验证问题规模，调用 `profile_convolution()`，最后输出一行结果。也正因为如此，这个文件是一个完整的示例程序，而不只是库级别的代码片段。

---

## Key Concepts / 关键概念
**EN**: the file defines the `DefaultDepthwiseDirect2dConvFprop` kernel alias, wraps it in `cutlass::conv::device::DirectConvolution`, and drives it through `Options`, `Result`, `profile_convolution()`, and `main()`.
**CN**: 组合关系：该文件先定义 `DefaultDepthwiseDirect2dConvFprop` 的 kernel 别名，再用 `cutlass::conv::device::DirectConvolution` 包装，并通过 `Options`、`Result`、`profile_convolution()` 和 `main()` 进行驱动。

**EN**: Depthwise example flow: define a fixed-shape direct-convolution kernel → validate runtime options against that shape → allocate/init tensors → launch `DirectConvolution` → optionally verify and benchmark.
**CN**: Depthwise 示例流程：先定义固定形状的 direct-convolution kernel → 用该形状约束运行时参数 → 分配并初始化张量 → 启动 `DirectConvolution` → 按需校验并测量性能。

## Dependencies / 依赖项
**EN**: Standard-library dependency: `<iostream>` is used directly in this file.
**CN**: 标准库依赖：`<iostream>` 在该文件中被直接使用。

**EN**: Standard-library dependency: `<fstream>` is used directly in this file.
**CN**: 标准库依赖：`<fstream>` 在该文件中被直接使用。

**EN**: Standard-library dependency: `<sstream>` is used directly in this file.
**CN**: 标准库依赖：`<sstream>` 在该文件中被直接使用。

**EN**: CUTLASS dependency: `cutlass/cutlass.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/cutlass.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/gemm/device/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/gemm/device/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/conv/kernel/default_depthwise_fprop.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/conv/kernel/default_depthwise_fprop.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/conv/device/implicit_gemm_convolution.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/conv/device/implicit_gemm_convolution.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/conv/device/direct_convolution.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/conv/device/direct_convolution.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/command_line.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/command_line.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/host_tensor.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/host_tensor.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/tensor_view_io.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/tensor_view_io.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/device/gemm.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/device/gemm.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_compare.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_compare.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_copy.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_copy.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/tensor_fill.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/tensor_fill.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/reference/host/convolution.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/reference/host/convolution.h` 提供该文件直接使用的库级原语。

**EN**: CUTLASS dependency: `cutlass/util/tensor_view_io.h` supplies library primitives used directly in this file.
**CN**: CUTLASS 依赖：`cutlass/util/tensor_view_io.h` 提供该文件直接使用的库级原语。

**EN**: Project-local dependency: `helper.h` provides helper code included directly by this file.
**CN**: 项目内依赖：`helper.h` 提供该文件直接包含的辅助代码。
