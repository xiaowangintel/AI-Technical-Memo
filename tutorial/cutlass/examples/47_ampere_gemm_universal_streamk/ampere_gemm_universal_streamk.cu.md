# ampere_gemm_universal_streamk.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/47_ampere_gemm_universal_streamk/ampere_gemm_universal_streamk.cu`  
**Purpose / 用途**: Compares classic data-parallel GEMM, Stream-K GEMM, and Split-K behavior on Ampere by holding the math kernel constant and changing only the threadblock swizzle and Stream-K scheduling parameters. / 在 Ampere 上对比经典数据并行 GEMM、Stream-K GEMM 与 Split-K 行为：保持数学内核不变，只改变 threadblock swizzle 和 Stream-K 调度参数。

---

## Line-by-Line Analysis / 逐行分析

### Experiment motivation and expected outputs (Lines 32-91)

```cpp
/***************************************************************************************************
 Example contrasting the Stream-K parallel decomposition for GEMM threadblocks versus the
 "classic data-parallel" and "Split-K" decompositions.

 For more details regarding the Stream-K method, see "Stream-K: Work-centric Parallel Decomposition
 for Dense Matrix-Matrix Multiplication on the GPU" (https://arxiv.org/abs/2301.03598)

 Requires NVIDIA Ampere or newer device (SM80+).

 - To lock persistence mode, power (400W), clocks (1005MHz) for evaluation (assumes device 0 and A100)

     cutlass$ sudo nvidia-smi -pm 1 -i 0

     cutlass$ sudo nvidia-smi -i 0 -pl 400

     cutlass$ sudo nvidia-smi -i 0 -lgc 1005

 - Build and run:

     cutlass$ mkdir build

     cutlass$ cd build

     cutlass/build$ cmake .. -DCUTLASS_NVCC_ARCHS=80

     cutlass/build$ make 47_ampere_gemm_universal_streamk

     cutlass/build$ ./examples/47_ampere_gemm_universal_streamk/47_ampere_gemm_universal_streamk

        10000 timing iterations of 2048 x 2048 x 2048 matrix-matrix multiply

        Basic data-parallel GEMM
          Disposition: Passed
          Avg runtime: 0.112633 ms
          GFLOPs: 152530

        StreamK GEMM with default load-balancing
          Disposition: Passed
          Avg runtime: 0.0941929 ms
          GFLOPs: 182390
          Speedup vs Basic-DP: 1.196

        StreamK emulating basic data-parallel GEMM
          Disposition: Passed
          Avg runtime: 0.113119 ms
          GFLOPs: 151875
          Speedup vs Basic-DP: 0.996

        Basic split-K GEMM with tile-splitting factor 2
          Disposition: Passed
          Avg runtime: 0.104772 ms
          GFLOPs: 163973

        StreamK emulating Split-K GEMM with tile-splitting factor 2
          Disposition: Passed
          Avg runtime: 0.105379 ms
          GFLOPs: 163029
          Speedup vs Basic-SplitK: 0.994

 **************************************************************************************************/
```

**EN**: The header frames the file as an experiment in work decomposition rather than new math. It explains that the example contrasts classic data-parallel decomposition, Stream-K load balancing, and Split-K-style tile splitting, then shows representative runtimes and speedups so the reader knows exactly what behavioral differences to look for.

**CN**: 文件头把这个示例定位为“工作分解方式”的实验，而不是新的数学实现。它说明示例将对比经典数据并行分解、Stream-K 负载均衡以及 Split-K 风格的 K 维切分，并给出代表性的运行时间和加速比，让读者从一开始就知道应关注哪些行为差异。

### Kernel configuration: only the swizzle changes (Lines 111-189)

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations (cutlass_tensorop_h16816gemm_128x128_32x4_nn_align8)
/////////////////////////////////////////////////////////////////////////////////////////////////

// A matrix configuration
using         ElementA    = cutlass::half_t;                                // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)

// B matrix configuration
using         ElementB    = cutlass::half_t;                                // Element type for B matrix operand
using         LayoutB     = cutlass::layout::RowMajor;                      // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)

// C/D matrix configuration
using         ElementC    = cutlass::half_t;                                // Element type for C and D matrix operands
using         LayoutC     = cutlass::layout::RowMajor;                      // Layout type for C and D matrix operands
constexpr int AlignmentC  = 128 / cutlass::sizeof_bits<ElementC>::value;    // Memory access granularity/alignment of C/D matrices in units of elements (up to 16 bytes)

// Multiply-accumulate blocking/pipelining details
using ElementAccumulator  = cutlass::half_t;                          // Element type for internal accumulation
using ArchTag             = cutlass::arch::Sm80;                      // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;           // Operator class tag
using ThreadblockShape    = cutlass::gemm::GemmShape<128, 128, 32>;   // Threadblock-level tile size (concept: GemmShape)
using WarpShape           = cutlass::gemm::GemmShape<64, 64, 32>;     // Warp-level tile size (concept: GemmShape)
using InstructionShape    = cutlass::gemm::GemmShape<16, 8, 16>;      // Instruction-level tile size (concept: GemmShape)
constexpr int NumStages   = 4;                                        // Number of global->shared pipeline stages used in the GEMM mainloop

// Epilogue output operator
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementC,               // Element type for C and D matrix operands
    AlignmentC,             // Memory access granularity of C and D matrix in units of elements
    ElementAccumulator,     // Element type from internal accumaccumulation
    ElementAccumulator>;    // Data type used to compute linear combination

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

// Classic data-parallel device GEMM implementation type
using DeviceGemmBasic = cutlass::gemm::device::GemmUniversal<
    ElementA, LayoutA,
    ElementB, LayoutB,
    ElementC, LayoutC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOp,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
    NumStages,
    AlignmentA,
    AlignmentB>;

// StreamK device GEMM implementation type
using DeviceGemmStreamK = cutlass::gemm::device::GemmUniversal<
    ElementA, LayoutA,
    ElementB, LayoutB,
    ElementC, LayoutC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOp,
    cutlass::gemm::threadblock::ThreadblockSwizzleStreamK, // <-- Only difference
    NumStages,
    AlignmentA,
    AlignmentB>;
```

**EN**: Both kernels use the same Ampere tensor-core math: half-precision inputs/outputs, a 128x128x32 threadblock tile, 64x64x32 warp tiles, and a 16x8x16 instruction shape. `DeviceGemmBasic` uses the identity swizzle, while `DeviceGemmStreamK` swaps in `ThreadblockSwizzleStreamK`. That single type substitution is the essence of the example: scheduling policy changes while the underlying GEMM kernel remains otherwise identical.

**CN**: 两个内核使用完全相同的 Ampere Tensor Core 数学配置：half 输入输出、128x128x32 的线程块 tile、64x64x32 的 warp tile，以及 16x8x16 的指令级 tile。`DeviceGemmBasic` 使用恒等 swizzle，而 `DeviceGemmStreamK` 仅把它替换为 `ThreadblockSwizzleStreamK`。这一处类型替换正是示例的核心：只改调度策略，不改底层 GEMM 数学。

### Benchmark state and command-line options (Lines 196-300)

```cpp
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
    avg_runtime_ms(avg_runtime_ms), gflops(gflops), status(status), error(error), passed(true)
  {}

};


/// Command line options parsing
struct Options
{
  std::string               command_name;
  bool                      help;
  cutlass::gemm::GemmCoord  problem_size;
  float                     alpha;
  float                     beta;
  int                       split_k_factor;
  int                       avail_sms;
  bool                      reference_check;
  int                       iterations;

  cutlass::HostTensor<ElementA, LayoutA> tensor_a;
  cutlass::HostTensor<ElementB, LayoutB> tensor_b;
  cutlass::HostTensor<ElementC, LayoutC> tensor_c;
  cutlass::HostTensor<ElementC, LayoutC> tensor_d;
  cutlass::HostTensor<ElementC, LayoutC> tensor_ref_d;

  Options(std::string command_name) :
    command_name(command_name),
    help(false),
    problem_size({2048, 2048, 2048}),
    alpha(1.0f),
    beta(0.0f),
    split_k_factor(1),
    avail_sms(-1),              // Number of device SMs to use is unlimited
    reference_check(true),
    iterations(10000)
  {}

  bool valid() const
  {
    return true;
  }

  void parse(int argc, char const **args)
  {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    cmd.get_cmd_line_argument("m", problem_size.m());
    cmd.get_cmd_line_argument("n", problem_size.n());
    cmd.get_cmd_line_argument("k", problem_size.k());
    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);
    cmd.get_cmd_line_argument("split", split_k_factor);
    cmd.get_cmd_line_argument("iterations", iterations);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const
  {
    out
      << "Performs a GEMM computation.\n"
      << "\n"
      << "Options:\n"
      << "\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --m=<int>                   GEMM M dimension\n"
      << "  --n=<int>                   GEMM N dimension\n"
      << "  --k=<int>                   GEMM K dimension\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --split=<int>               Split-K factor to emulate\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";

    out
      << "\n\nExamples:\n\n"
      << "$ " << command_name << " --m=1024 --n=512 --k=1024 --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const
  {
    // Two flops per multiply-add
    return 2.0 * double(problem_size.product()) / double(1.0e9) / runtime_s;
  }
};
```

**EN**: The `Result` struct records pass/fail and throughput, while `Options` owns both GEMM parameters and all host/device tensors used by the benchmark. Two fields matter most for scheduling analysis: `split_k_factor` controls K-dimension tile splitting, and `avail_sms` controls how many SMs Stream-K is allowed to balance across. When `avail_sms = 1`, Stream-K is intentionally forced into a data-parallel-like execution pattern.

**CN**: `Result` 用于记录正确性和吞吐率，`Options` 则同时保存 GEMM 参数以及 benchmark 需要的全部 host/device tensor。对调度分析最关键的两个字段是：`split_k_factor` 控制 K 维 tile 切分；`avail_sms` 控制 Stream-K 允许跨多少个 SM 做负载均衡。当 `avail_sms = 1` 时，Stream-K 会被故意约束成接近数据并行的执行模式。

### Argument builders for basic GEMM and Stream-K GEMM (Lines 307-368)

```cpp
/// Populates a DeviceGemmBasic::Arguments structure from the given commandline options
typename DeviceGemmBasic::Arguments args_from_options(
    const DeviceGemmBasic &device_gemm,
    const Options &options,
    cutlass::HostTensor<ElementA, LayoutA> &tensor_a,
    cutlass::HostTensor<ElementB, LayoutB> &tensor_b,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_c,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_d)
{
  return typename DeviceGemmBasic::Arguments(
    cutlass::gemm::GemmUniversalMode::kGemm,  // universal mode
    options.problem_size,                     // problem_size
    options.split_k_factor,                   // batch count / splitk slices
    {                                         // epilogue parameters
      ElementAccumulator(options.alpha),
      ElementAccumulator(options.beta)
    },
    tensor_a.device_data(),                   // ptr_A
    tensor_b.device_data(),                   // ptr_B
    tensor_c.device_data(),                   // ptr_C
    tensor_d.device_data(),                   // ptr_D
    options.problem_size.mk().product(),      // batch_stride_A
    options.problem_size.nk().product(),      // batch_stride_B
    options.problem_size.mn().product(),      // batch_stride_C
    options.problem_size.mn().product(),      // batch_stride_D
    tensor_a.layout().stride(0),              // stride_a
    tensor_b.layout().stride(0),              // stride_b
    tensor_c.layout().stride(0),              // stride_c
    tensor_d.layout().stride(0));             // stride_d
}

/// Populates a DeviceGemmStreamK::Arguments structure from the given commandline options
typename DeviceGemmStreamK::Arguments args_from_options(
    const DeviceGemmStreamK &device_gemm,
    const Options &options,
    cutlass::HostTensor<ElementA, LayoutA> &tensor_a,
    cutlass::HostTensor<ElementB, LayoutB> &tensor_b,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_c,
    cutlass::HostTensor<ElementC, LayoutC> &tensor_d)
{
  return typename DeviceGemmStreamK::Arguments(
    cutlass::gemm::GemmUniversalMode::kGemm,  // universal mode
    options.problem_size,                     // problem_size
    options.split_k_factor,                   // batch count / splitk slices
    {                                         // epilogue parameters
      ElementAccumulator(options.alpha),
      ElementAccumulator(options.beta)
    },
    tensor_a.device_data(),                   // ptr_A
    tensor_b.device_data(),                   // ptr_B
    tensor_c.device_data(),                   // ptr_C
    tensor_d.device_data(),                   // ptr_D
    options.problem_size.mk().product(),      // batch_stride_A
    options.problem_size.nk().product(),      // batch_stride_B
    options.problem_size.mn().product(),      // batch_stride_C
    options.problem_size.mn().product(),      // batch_stride_D
    tensor_a.layout().stride(0),              // stride_a
    tensor_b.layout().stride(0),              // stride_b
    tensor_c.layout().stride(0),              // stride_c
    tensor_d.layout().stride(0),              // stride_d
    options.avail_sms);                       // avail_sms
}
```

**EN**: The two `args_from_options()` overloads are nearly identical by design. Both pass the same universal mode, problem size, split-K factor, alpha/beta, data pointers, batch strides, and leading dimensions. The only semantic extension in the Stream-K overload is the final `options.avail_sms` argument, which feeds the runtime scheduler with the width of the load-balancing window.

**CN**: 两个 `args_from_options()` 重载是故意保持高度相似的：它们都传入相同的 universal mode、问题规模、split-K 因子、alpha/beta、数据指针、batch stride 和 leading dimension。Stream-K 版本唯一新增的语义参数是最后的 `options.avail_sms`，它告诉运行时调度器可以在多宽的 SM 范围内做负载均衡。

### Generic execution driver for each scheduling strategy (Lines 371-438)

```cpp
/// Execute a given example GEMM computation
template <typename DeviceGemmT>
Result run(std::string description, Options &options)
{
  // Display test description
  std::cout << std::endl << description << std::endl;

  // Zero-initialize test output matrix D
  cutlass::reference::host::TensorFill(options.tensor_d.host_view());
  options.tensor_d.sync_device();

  // Instantiate CUTLASS kernel depending on templates
  DeviceGemmT device_gemm;

  // Create a structure of gemm kernel arguments suitable for invoking an instance of DeviceGemmT
  auto arguments = args_from_options(device_gemm, options, options.tensor_a, options.tensor_b, options.tensor_c, options.tensor_d);

  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = DeviceGemmT::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Check the problem size is supported or not
  CUTLASS_CHECK(device_gemm.can_implement(arguments));

  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(device_gemm.initialize(arguments, workspace.get()));

  // Correctness / Warmup iteration
  CUTLASS_CHECK(device_gemm());

  // Copy output data from CUTLASS and reference kernel to host for comparison
  options.tensor_d.sync_host();

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  Result result;
  result.passed = cutlass::reference::host::TensorEquals(
    options.tensor_d.host_view(),
    options.tensor_ref_d.host_view());

  std::cout << "  Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;

  // Run profiling loop
  if (options.iterations > 0)
  {
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(device_gemm());
    }
    timer.stop();

    // Compute average runtime and GFLOPs.
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);

    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPs: " << result.gflops << std::endl;
  }

  if (!result.passed) {
    exit(-1);
  }

  return result;
}
```

**EN**: The templated `run()` helper zeroes D, builds the chosen GEMM arguments, allocates workspace, checks support, launches one correctness iteration, compares against the precomputed reference, and then times repeated executions. Because both the basic and Stream-K kernels satisfy the same adapter interface, this single function can benchmark them side by side without changing any surrounding logic.

**CN**: 模板化的 `run()` 辅助函数会清零 D、构造所选 GEMM 参数、分配工作区、检查是否支持、运行一次正确性迭代、与预先算好的参考结果比较，然后对重复执行进行计时。由于基本版和 Stream-K 版都满足同一套 adapter 接口，这个函数可以在不修改外围逻辑的情况下并排评测两者。

### Environment checks, tensor initialization, and reference GEMM (Lines 442-554)

```cpp
int main(int argc, const char **argv)
{
  // CUTLASS must be compiled with CUDA 11.0 Toolkit to run these examples.
  if (!(__CUDACC_VER_MAJOR__ >= 11)) {
    std::cerr << "Ampere Tensor Core operations must be compiled with CUDA 11.0 Toolkit or later." << std::endl;

    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  // Current device must must have compute capability at least 80
  cudaDeviceProp props;
  int current_device_id;
  CUDA_CHECK(cudaGetDevice(&current_device_id));
  CUDA_CHECK(cudaGetDeviceProperties(&props, current_device_id));
  if (!((props.major * 10 + props.minor) >= 80))
  {
    std::cerr << "Ampere Tensor Core operations must be run on a machine with compute capability at least 80."
              << std::endl;

    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  // Parse commandline options
  Options options("ampere_streamk_gemm");
  options.parse(argc, argv);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  std::cout <<
    options.iterations << " timing iterations of " <<
    options.problem_size.m() << " x " <<
    options.problem_size.n() << " x " <<
    options.problem_size.k() << " matrix-matrix multiply" << std::endl;

  if (!options.valid()) {
    std::cerr << "Invalid problem." << std::endl;
    return -1;
  }


  //
  // Initialize GEMM datasets
  //

  // Initialize tensors using CUTLASS helper functions
  options.tensor_a.resize(options.problem_size.mk());       // <- Create matrix A with dimensions M x K
  options.tensor_b.resize(options.problem_size.kn());       // <- Create matrix B with dimensions K x N
  options.tensor_c.resize(options.problem_size.mn());       // <- Create matrix C with dimensions M x N
  options.tensor_d.resize(options.problem_size.mn());       // <- Create matrix D with dimensions M x N used to store output from CUTLASS kernel
  options.tensor_ref_d.resize(options.problem_size.mn());   // <- Create matrix D with dimensions M x N used to store output from reference kernel

  // Fill matrix A on host with uniform-random data [-2, 2]
  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_a.host_view(),
      1,
      ElementA(2),
      ElementA(-2),
      0);

  // Fill matrix B on host with uniform-random data [-2, 2]
  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_b.host_view(),
      1,
      ElementB(2),
      ElementB(-2),
      0);

  // Fill matrix C on host with uniform-random data [-2, 2]
  cutlass::reference::host::TensorFillRandomUniform(
      options.tensor_c.host_view(),
      1,
      ElementC(2),
      ElementC(-2),
      0);


  //
  // Compute reference output
  //

  // Copy data from host to GPU
  options.tensor_a.sync_device();
  options.tensor_b.sync_device();
  options.tensor_c.sync_device();

  // Zero-initialize reference output matrix D
  cutlass::reference::host::TensorFill(options.tensor_ref_d.host_view());
  options.tensor_ref_d.sync_device();

  // Create instantiation for device reference gemm kernel
  DeviceGemmReference gemm_reference;

  // Launch device reference gemm kernel
  gemm_reference(
    options.problem_size,
    ElementAccumulator(options.alpha),
    options.tensor_a.device_ref(),
    options.tensor_b.device_ref(),
    ElementAccumulator(options.beta),
    options.tensor_c.device_ref(),
    options.tensor_ref_d.device_ref());

  // Wait for kernels to finish
  CUDA_CHECK(cudaDeviceSynchronize());

  // Copy output data from reference kernel to host for comparison
  options.tensor_ref_d.sync_host();
```

**EN**: The entrypoint first enforces CUDA 11+ and SM80+ requirements, then allocates A/B/C/D tensors and fills them with host random values. Before testing CUTLASS kernels it computes a device reference GEMM and copies the result back to host. This is important for the Stream-K comparison: every later measurement varies only scheduling, not correctness criteria or numerical reference path.

**CN**: 入口函数先检查 CUDA 11+ 与 SM80+ 条件，然后分配 A/B/C/D 张量，并在 host 上填充随机值。在测试 CUTLASS 内核之前，它先运行一个 device 参考 GEMM，并把结果拷回 host。对 Stream-K 对比来说，这一步很重要，因为后续所有评测变化的只有调度方式，而不是正确性标准或数值参考路径。

### Comparing default Stream-K, data-parallel emulation, and Split-K emulation (Lines 560-592)

```cpp
  // Test default operation
  if (options.split_k_factor == 1)
  {
    // Compare basic data-parallel version versus StreamK version using default load-balancing heuristics
    Result basic_dp         = run<DeviceGemmBasic>("Basic data-parallel GEMM", options);
    Result streamk_default  = run<DeviceGemmStreamK>("StreamK GEMM with default load-balancing", options);

    printf("  Speedup vs Basic-DP: %.3f\n", (basic_dp.avg_runtime_ms / streamk_default.avg_runtime_ms));

    // Show that StreamK can emulate basic data-parallel GEMM when we set the number of SMs to load-balance across = 1
    options.avail_sms       = 1;        // Set loadbalancing width to 1 SM (no load balancing)
    Result streamk_dp       = run<DeviceGemmStreamK>("StreamK emulating basic data-parallel GEMM", options);
    options.avail_sms       = -1;       // Reset loadbalancing width to unspecified SMs (i.e., the number of device SMs)

    printf("  Speedup vs Basic-DP: %.3f\n", (basic_dp.avg_runtime_ms / streamk_dp.avg_runtime_ms));

    options.split_k_factor++;     // Increment splitting factor for next evaluation

  }

  // Show that StreamK can emulate "Split-K" with a tile-splitting factor
  Result basic_splitk = run<DeviceGemmBasic>(
    std::string("Basic split-K GEMM with tile-splitting factor ") + std::to_string(options.split_k_factor),
    options);

  Result streamk_splitk = run<DeviceGemmStreamK>(
    std::string("StreamK emulating Split-K GEMM with tile-splitting factor ") + std::to_string(options.split_k_factor),
    options);

  printf("  Speedup vs Basic-SplitK: %.3f\n", (basic_splitk.avg_runtime_ms / streamk_splitk.avg_runtime_ms));

  return 0;
}
```

**EN**: The final block performs the actual scheduling experiment. When `split_k_factor == 1`, it compares basic GEMM against default Stream-K and then forces `avail_sms = 1` so Stream-K emulates classic data parallelism. After incrementing the split factor, it runs both the basic and Stream-K kernels again to show that Stream-K can also emulate Split-K. This is the precise pedagogical point of the example: Stream-K is presented as a generalized scheduler that can degenerate into either traditional pattern.

**CN**: 最后这段代码真正执行调度实验。当 `split_k_factor == 1` 时，它先对比基本 GEMM 与默认 Stream-K，然后把 `avail_sms = 1`，让 Stream-K 模拟经典数据并行。接着增加 split 因子，再次运行基础版和 Stream-K 版，展示 Stream-K 同样可以模拟 Split-K。这个示例最重要的教学意义就在这里：Stream-K 被展示成一种更一般化的调度器，可以退化成两种传统模式。

---

## Key Concepts / 关键概念

- **EN**: Stream-K scheduling is isolated to the swizzle/runtime-argument layer; the math kernel itself is intentionally unchanged.
  **CN**: Stream-K 调度被刻意限制在 swizzle/运行时参数层，数学内核本身保持不变。
- **EN**: `avail_sms` is the key runtime control: default lets Stream-K spread work broadly, while `1` collapses it into data-parallel behavior.
  **CN**: `avail_sms` 是最关键的运行时控制项：默认值让 Stream-K 在更宽的 SM 范围内分摊工作，而设为 `1` 会让它收缩成数据并行行为。
- **EN**: `split_k_factor` lets the same benchmark show both classic Split-K and Stream-K’s ability to mimic it.
  **CN**: `split_k_factor` 让同一个 benchmark 同时展示经典 Split-K，以及 Stream-K 模拟 Split-K 的能力。
- **EN**: The experiment is reliable because both scheduling variants share the same reference path, data initialization, and profiling harness.
  **CN**: 这个实验之所以可信，是因为两种调度变体共享同一个参考路径、同一套数据初始化和同一套性能测试框架。

## Dependencies / 依赖项

- **EN**: `cutlass/gemm/device/gemm_universal.h` provides both the classic GEMM adapter and the Stream-K-capable adapter used here.
  **CN**: `cutlass/gemm/device/gemm_universal.h` 同时提供了这里使用的经典 GEMM adapter 和支持 Stream-K 的 adapter。
- **EN**: `cutlass::gemm::threadblock::ThreadblockSwizzleStreamK` is the scheduling primitive that differentiates the Stream-K path.
  **CN**: `cutlass::gemm::threadblock::ThreadblockSwizzleStreamK` 是区分 Stream-K 路径的核心调度原语。
- **EN**: Reference correctness uses `cutlass::reference::device::Gemm` plus host-side tensor comparison helpers.
  **CN**: 参考正确性依赖 `cutlass::reference::device::Gemm` 以及 host 侧的张量比较辅助函数。
- **EN**: `helper.h` supplies the timer and CUDA/CUTLASS error-checking macros used by the profiling loop.
  **CN**: `helper.h` 提供了计时器以及性能测试循环中使用的 CUDA/CUTLASS 错误检查宏。
