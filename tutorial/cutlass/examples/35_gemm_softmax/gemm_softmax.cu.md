# gemm_softmax.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/35_gemm_softmax/gemm_softmax.cu`
**Purpose / 用途**: Demonstrates, verifies, and profiles the example GEMM+Softmax pipeline by parsing CLI options, allocating batched tensors, launching the composed kernels, comparing against a reference implementation, and reporting performance. / 该示例通过解析命令行、分配批量张量、启动组合内核、与参考实现比对并输出性能，演示、验证并评测 GEMM+Softmax 流水线。
---
## Line-by-Line Analysis / 逐行分析
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

/**

*/
```
**EN**: The banner marks this as the executable driver for the example. Unlike the headers, this file owns user interaction, allocation, verification, and benchmarking.
**CN**: 文件头表明这里是示例的可执行驱动程序。与两个头文件不同，它负责用户交互、内存分配、结果验证和性能测试。

```cpp
#include <cmath>
#include <iostream>
#include <vector>
#include <limits>

#include "cutlass/cutlass.h"
#include "cutlass/arch/memory.h"
#include "cutlass/arch/memory_sm75.h"
#include "cutlass/gemm/device/gemm_complex.h"
#include "cutlass/numeric_types.h"
#include "cutlass/numeric_size.h"
#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"

#include "cutlass/util/reference/host/gemm_complex.h"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/host/tensor_reduce.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/error_metrics.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/numeric_size.h" // cutlass::bits_to_bytes

#include "cutlass/layout/matrix.h"
#include "cutlass/epilogue/thread/linear_combination.h"
/////////////////////////////////////////////////////////////////////////////////////////////////

#include "gemm_with_softmax.h"
```
**EN**: The includes combine core CUTLASS headers, reference GEMM/tensor utilities, the command-line parser, tensor I/O helpers, and the local gemm_with_softmax.h composition header. Together they provide everything needed to run both the fast path and the reference path.
**CN**: 这些头文件组合了 CUTLASS 核心组件、参考 GEMM/张量工具、命令行解析器、张量 I/O 工具以及本地的 gemm_with_softmax.h 组合头文件，从而同时支持“快速路径”和“参考验证路径”。

```cpp
#define TRACE(x) { std::cout << "gemm_softmax.cu:" << __LINE__ << "  " << x << std::endl; }

/////////////////////////////////////////////////////////////////////////////////////////////////

enum class Disposition {
  kPassed,
  kIncorrect,
  kNotVerified
};
```
**EN**: TRACE is a simple debug macro for ad-hoc prints, and Disposition is the small result enum used by main() to report whether verification passed, failed, or was skipped.
**CN**: TRACE 是一个简单的调试输出宏；Disposition 是 main() 用来汇报结果的小型枚举，表示验证通过、失败或未执行。

```cpp
// Command line options parsing
struct Options {

  bool help;
  cutlass::gemm::GemmCoord problem_size;
  int batch_count;
  int iterations;
  unsigned seed;
  float alpha;
  float beta;
  bool verification_enabled;
  float tolerance;

  Options():
    help(false),
    problem_size({16, 24, 64}),
    batch_count(16),
    iterations(20),
    seed(2022),
    alpha(1),
    beta(0),
    verification_enabled(true),
    tolerance(1e-5f)
  { }
```
**EN**: Options stores all user-configurable parameters: GEMM size, batch count, alpha/beta scaling, iteration count, random seed, verification toggle, and numeric tolerance. The constructor establishes a small default problem and enables both verification and profiling by default.
**CN**: Options 保存所有用户可配置参数：GEMM 尺寸、batch 数、alpha/beta 缩放、迭代次数、随机种子、是否验证以及数值容差。构造函数给出一个较小的默认问题规模，并默认开启验证与性能测试。

```cpp
  bool valid() {

    return true;
  }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    cmd.get_cmd_line_argument("m", problem_size.m());
    cmd.get_cmd_line_argument("n", problem_size.n());
    cmd.get_cmd_line_argument("k", problem_size.k());

    cmd.get_cmd_line_argument("batch_count", batch_count);

    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);

    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("verify", verification_enabled);
    cmd.get_cmd_line_argument("seed", seed);
    cmd.get_cmd_line_argument("tolerance", tolerance);
  }
```
**EN**: valid() currently accepts any input, while parse() uses CUTLASS’s command-line helper to override the default fields. The API surface intentionally mirrors the printed flags, so the example remains easy to script from the shell.
**CN**: valid() 当前对输入不做额外限制，而 parse() 使用 CUTLASS 提供的命令行工具覆盖默认字段。这个 API 表面与后面打印的命令行选项一一对应，便于从 shell 脚本中调用该示例。

```cpp
  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "35_gemm_softmax example\n\n"
      << "  This example uses the CUTLASS Library to compute GEMM + Softmax for arbitrary problem sizes.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --m=<int>                   GEMM M dimension\n"
      << "  --n=<int>                   GEMM N dimension\n"
      << "  --k=<int>                   GEMM K dimension\n"
      << "  --batch_count=<int>         Batch number\n"
      << "  --alpha=<f32>               Epilogue scalar alpha\n"
      << "  --beta=<f32>                Epilogue scalar beta\n\n"
      << "  --seed=<int>                Random number seed (1*)\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform (0 to disable profiling).\n\n"
      << "  --verify=<bool>             If true, performs reference calculation.\n\n"
      << "  --tolerance <float>         Error tolerance\n"
    ;

    out << "\n\nExamples:\n\n"
      << "$ ./examples/35_gemm_softmax/35_gemm_softmax --m=1024 --n=512 \\\n"
      << "     --alpha=2 --beta=0.707 \n\n";

    return out;
  }

  /// Returns true if the environment and Toolkit support this
  bool supported(bool verbose = true) const {

    // Ampere Tensor Core operations exposed with mma.sync and ldmatrix are first available
    // in CUDA 11.0.
    //
    // CUTLASS must be compiled with CUDA 11.0 Toolkit to run these examples.
    if (!(__CUDACC_VER_MAJOR__ >= 11)) {
      if (verbose) {
        std::cerr << "Ampere Tensor Core operations must be compiled with CUDA 11.0 Toolkit or later." << std::endl;
      }
      return false;
    }

    cudaDeviceProp props;

    cudaError_t error = cudaGetDeviceProperties(&props, 0);
    if (error != cudaSuccess) {
      if (verbose) {
        std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
      }
      return false;
    }

    if (!((props.major * 10 + props.minor) >= 80)) {
      if (verbose) {
        std::cerr << "Ampere Tensor Core operations must be run on a machine with compute capability at least 80."
                  << std::endl;
      }
      return false;
    }

    return true;
  }
```
**EN**: print_usage() documents the CLI, and supported() checks the runtime environment. The example requires a CUDA 11+ toolchain and a GPU with compute capability 80 or higher because its chosen Tensor Core path targets Ampere-class mma.sync/ldmatrix behavior.
**CN**: print_usage() 用于打印命令行帮助，而 supported() 负责检查运行环境。该示例要求 CUDA 11 及以上工具链，并且 GPU 计算能力至少为 80，因为它选择的 Tensor Core 路径依赖 Ampere 时代的 mma.sync / ldmatrix 能力。

```cpp
struct Testbed {

  //
  // Type definitions
  //


  using ElementA = cutlass::half_t;
  using ElementB = cutlass::half_t;
  using ElementC = cutlass::half_t;
  using ElementCompute = float;
  using ElementD = ElementC;
  using ElementSoftmax = ElementC;

  using LayoutA = cutlass::layout::RowMajor;
  using LayoutB = cutlass::layout::ColumnMajor;

  using ThreadblockShape = cutlass::gemm::GemmShape<128, 128, 32>;
  using WarpShape        = cutlass::gemm::GemmShape<64, 64, 32>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;

  using OperatorClass = cutlass::arch::OpClassTensorOp;
  using ArchTag = cutlass::arch::Sm80;

  // ApplyShape impacts the final Softmax performance a lot.
  // Set ApplyShape::kColumn to be the next multiple of 32 number that is after
  // (gemm_N / alignment).
  // Set ApplyShape::kRow to max(1, 128 / ApplyShape::kColumn).
  using ApplyShape = cutlass::MatrixShape<1, 1024>;

  static int const kStages = 3;

  /// Linear scaling operator
  using EpilogueFunctorOp = cutlass::epilogue::thread::LinearCombination<
    ElementC,
    128 / cutlass::sizeof_bits<ElementC>::value,
    ElementCompute,
    ElementCompute
  >;

  using GemmSoftmax = cutlass::GemmSoftmax<
    ElementA, LayoutA,
    ElementB, LayoutB,
    ElementC,
    ElementCompute,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueFunctorOp,
    kStages,
    ApplyShape
  >;

  using ElementNorm = typename GemmSoftmax::ElementNorm;
  using ElementSum = typename GemmSoftmax::ElementSum;
  using LayoutC = typename GemmSoftmax::LayoutC;
  using LayoutN = typename GemmSoftmax::LayoutN;
  using LayoutS = typename GemmSoftmax::LayoutS;
  using MatrixCoord = typename LayoutC::TensorCoord;
```
**EN**: Testbed fixes one concrete instantiation of the generic GemmSoftmax template: half-precision A/B/C, float accumulation, row-major A, column-major B, Tensor Core MMA on SM80, and a 128x128x32 threadblock shape. The comments around ApplyShape are important: the softmax-apply kernel expects a tile shape chosen to cover the row efficiently, so this example uses MatrixShape<1, 1024>. The final aliases expose the derived norm/sum/layout types from GemmSoftmax.
**CN**: Testbed 把通用的 GemmSoftmax 模板固定为一个具体实例：A/B/C 使用 half，累加使用 float，A 为 RowMajor，B 为 ColumnMajor，在 SM80 上使用 Tensor Core MMA，线程块形状为 128x128x32。关于 ApplyShape 的注释非常关键：softmax 应用内核依赖一个能高效覆盖整行的 tile 形状，因此这里选择 MatrixShape<1, 1024>。最后几行别名则从 GemmSoftmax 中导出了 norm/sum/layout 等派生类型。

```cpp
  //
  // Data members
  //

  Options const &options;


  cutlass::HostTensor<ElementNorm, LayoutC>     reference_N;

  cutlass::DeviceAllocation<ElementA> block_A;
  cutlass::DeviceAllocation<ElementB> block_B;
  cutlass::DeviceAllocation<ElementC> block_C;
  cutlass::DeviceAllocation<ElementD> block_D;
  cutlass::DeviceAllocation<ElementD> block_Ref;
  cutlass::DeviceAllocation<ElementSoftmax> block_Softmax;
  cutlass::DeviceAllocation<ElementNorm> block_Norm;
  cutlass::DeviceAllocation<ElementSum> block_Sum;

  int block_num = (options.problem_size.n() + GemmSoftmax::ThreadblockShape::kN - 1) / GemmSoftmax::ThreadblockShape::kN;

  cutlass::gemm::GemmCoord problem = options.problem_size;

  int64_t lda = LayoutA::packed({problem.m(), problem.k()}).stride(0);
  int64_t ldb = LayoutB::packed({problem.k(), problem.n()}).stride(0);
  int64_t ldc = LayoutC::packed({problem.m(), problem.n()}).stride(0);

  // fixed rowmajor for norm and sum
  int64_t ldn = problem.m();
  int64_t lds = ldn;

  int64_t total_elements_A_per_batch = problem.m() * problem.k();
  int64_t total_elements_B_per_batch = problem.k() * problem.n();
  int64_t total_elements_C_per_batch = problem.m() * problem.n();
  int64_t total_elements_D_per_batch = problem.m() * problem.n();
  int64_t total_elements_partial_norm_per_batch = block_num * problem.m();

  int64_t total_elements_A = total_elements_A_per_batch * options.batch_count;
  int64_t total_elements_B = total_elements_B_per_batch * options.batch_count;
  int64_t total_elements_C = total_elements_C_per_batch * options.batch_count;
  int64_t total_elements_D = total_elements_D_per_batch * options.batch_count;
  int64_t total_elements_partial_norm = total_elements_partial_norm_per_batch * options.batch_count;

  //
  // Methods
  //

  Testbed(
    Options const &options_
  ):
    options(options_)
  {
    reference_N.reset({options.problem_size.m(), 1}, false);
  }
```
**EN**: These members describe the test fixture’s working set. The code allocates device buffers for A/B/C/D, reference output, softmax output, and the partial norm/sum workspaces; computes packed leading dimensions; and sizes the norm/sum workspace as (number of N-tiles) × M × batch_count because the visitor emits one partial statistic per output row per GEMM tile. The constructor also initializes a host tensor used for reference row maxima.
**CN**: 这些成员定义了测试夹具的工作集。代码为 A/B/C/D、参考输出、softmax 输出以及局部 norm/sum 工作区分配设备缓冲区，计算紧凑布局下的 leading dimension，并把 norm/sum 工作区尺寸设为“(N 方向 tile 数) × M × batch_count”，因为 visitor 会为每个输出 tile 的每一行生成一份局部统计。构造函数还初始化了一个主机张量，用于保存参考实现中的逐行最大值。

```cpp
  /// Run
  Disposition run() {

    Disposition disposition = Disposition::kNotVerified;

    //
    // Initialize the workspace
    //

    initialize();

    //
    // Launch device kernel
    //
    cutlass::Status status = cutlass::Status::kSuccess;

    status = execute_device_kernel();

    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Device execution failed." << std::endl;
      return disposition;
    }

    cudaError_t result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Device synchronize failed with error "
        << cudaGetErrorString(result) << std::endl;
      return disposition;
    }

    //
    // Verify
    //

    if (options.verification_enabled) {

      bool passed = verify();

      if (passed) {
        disposition = Disposition::kPassed;
      }
      else {
        disposition = Disposition::kIncorrect;
      }
    }

    //
    // Profiling
    //
    if (options.iterations) {
      profile();
    }

    return disposition;
  }
```
**EN**: run() is the high-level scenario driver: initialize the tensors, launch the device pipeline, synchronize, optionally verify against a reference, optionally profile repeated runs, and return a Disposition enum summarizing the outcome.
**CN**: run() 是高层场景驱动函数：先初始化张量，再启动设备端流水线，同步，按需与参考实现比对，按需进行重复运行性能测试，最后返回一个概括结果的 Disposition 枚举。

```cpp
  /// Random initialization
  void initialize() {

    block_A.reset(total_elements_A);
    block_B.reset(total_elements_B);
    block_C.reset(total_elements_C);
    block_D.reset(total_elements_D);
    block_Softmax.reset(total_elements_D);
    block_Ref.reset(total_elements_D_per_batch);
    block_Norm.reset(total_elements_partial_norm);
    block_Sum.reset(total_elements_partial_norm);

    cutlass::reference::device::BlockFillRandomUniform(
            block_A.get(), total_elements_A, options.seed, ElementA(5), ElementA(-5), 0);

    cutlass::reference::device::BlockFillRandomUniform(
            block_B.get(), total_elements_B, options.seed + 1, ElementB(5), ElementB(-5), 0);

    cutlass::reference::device::BlockFillRandomUniform(
            block_C.get(), total_elements_C, options.seed + 2, ElementC(5), ElementC(-5), 0);

    cutlass::reference::device::BlockFillRandomUniform(
            block_D.get(), total_elements_D, options.seed + 3, ElementD(5), ElementD(-5), 0);

    cutlass::reference::device::BlockFillRandomUniform(
            block_Ref.get(), total_elements_D_per_batch, options.seed + 3, ElementD(5), ElementD(-5), 0);

    cutlass::reference::device::BlockFillRandomUniform(
            block_Softmax.get(), total_elements_D, options.seed + 3, ElementSoftmax(5), ElementSoftmax(-5), 0);

    cutlass::reference::host::TensorFill(
      reference_N.host_view(),
      ElementNorm()
    );

  }
```
**EN**: initialize() allocates all device buffers and fills A/B/C/D/reference/softmax buffers with random values. The reference_N host tensor is zeroed so the later host-side max-reduction starts from a clean state.
**CN**: initialize() 负责分配所有设备缓冲区，并用随机值填充 A/B/C/D/reference/softmax 缓冲区。reference_N 主机张量被清零，以便后续主机侧逐行最大值归约从干净状态开始。

```cpp
  cutlass::Status execute_device_kernel() {

    cutlass::Status status = cutlass::Status::kSuccess;

    //
    // Setup arguments
    //

    GemmSoftmax::Arguments args(
      options.problem_size,
      options.batch_count,
      {block_A.get(), lda},
      {block_B.get(), ldb},
      {block_C.get(), ldc},
      {block_D.get(), ldc},
      {
        ElementCompute(options.alpha),
        ElementCompute(options.beta)
      },
      {block_Norm.get(), ldn},
      {block_Sum.get(), lds},
      {block_Softmax.get(), ldc},
      total_elements_A_per_batch,
      total_elements_B_per_batch,
      total_elements_C_per_batch,
      total_elements_D_per_batch,
      total_elements_partial_norm_per_batch,
      total_elements_partial_norm_per_batch,
      total_elements_D_per_batch
    );

    //
    // Launch
    //

    GemmSoftmax gemm_softmax;

    // Initialize
    status = gemm_softmax.initialize(args);
    if (status != cutlass::Status::kSuccess) {
      return status;
    }

    // Run
    status = gemm_softmax();

    return status;
  }
```
**EN**: execute_device_kernel() translates the test fixture state into GemmSoftmax::Arguments: TensorRefs, alpha/beta epilogue parameters, batch strides, and workspaces. It then constructs the GemmSoftmax operator, initializes it, and launches the three-stage pipeline through its call operator.
**CN**: execute_device_kernel() 把测试夹具中的状态转换为 GemmSoftmax::Arguments，包括 TensorRef、alpha/beta 的 epilogue 参数、batch 跨距以及各类工作区。随后它构造 GemmSoftmax 操作器，完成 initialize()，并通过函数调用运算符启动三阶段流水线。

```cpp
  template<typename Element>
  bool verify_tensor(std::vector<Element> vector_Input, \
                       std::vector<Element> vector_Input_Ref) {

    auto size = int64_t((vector_Input.size() < vector_Input_Ref.size()) ? vector_Input.size() : vector_Input_Ref.size());
    float abs_tol = options.tolerance;
    float rel_tol = options.tolerance;
    
    for (int64_t i = 0; i < size; ++i) {
      float diff = (float)(vector_Input.at(i) - vector_Input_Ref.at(i));
      float abs_diff = fabs(diff);
      float abs_ref = fabs((float)vector_Input_Ref.at(i));
      float relative_diff = abs_ref > abs_tol ? abs_diff / abs_ref : 0;
      if ( (isnan(abs_diff) || isinf(abs_diff)) ||  (abs_diff > rel_tol && relative_diff > rel_tol)) {
        printf("diff = %f, {%f, %f}.\n", abs_diff, (float)(vector_Input.at(i)), (float)(vector_Input_Ref.at(i)));
        return false;
      }

    }

    return true;
  }
```
**EN**: verify_tensor() is a generic elementwise comparator with absolute and relative tolerance logic. It reports the first mismatching value, which is sufficient for an example whose main purpose is functional validation rather than exhaustive diagnostics.
**CN**: verify_tensor() 是一个通用逐元素比较器，同时使用绝对误差和相对误差逻辑。它在发现首个不匹配值时立即打印出来，这对于以功能验证为主的示例来说已经足够。

```cpp
  /// Verifies the reference matches
  bool verify() {

    LayoutA layout_A(lda);
    LayoutB layout_B(ldb);
    LayoutC layout_C(ldc);
    LayoutN Layout_N(ldn);
    LayoutS Layout_S(lds);

    MatrixCoord extent_A{problem.m(), problem.k()};
    MatrixCoord extent_B{problem.k(), problem.n()};
    MatrixCoord extent_C{problem.m(), problem.n()};

    for (int batch_idx = 0; batch_idx < options.batch_count; batch_idx++) {

      cutlass::TensorView<ElementA, LayoutA> view_A(block_A.get() + total_elements_A_per_batch * batch_idx, layout_A, extent_A);
      cutlass::TensorView<ElementB, LayoutB> view_B(block_B.get() + total_elements_B_per_batch * batch_idx, layout_B, extent_B);
      cutlass::TensorView<ElementC, LayoutC> view_C(block_C.get() + total_elements_C_per_batch * batch_idx, layout_C, extent_C);
      cutlass::TensorView<ElementC, LayoutC> view_Ref_device(block_Ref.get(), layout_C, extent_C);

      cutlass::reference::device::GemmComplex<
          ElementA, LayoutA,
          ElementB, LayoutB,
          ElementC, LayoutC, 
          ElementCompute, ElementCompute
      >(
        problem,
        options.alpha, 
        view_A,
        cutlass::ComplexTransform::kNone,
        view_B,
        cutlass::ComplexTransform::kNone,
        options.beta, 
        view_C, 
        view_Ref_device, 
        ElementCompute(0)
      );

      // Copy reference results to host memory for verification
      std::vector<ElementD> matrix_D_Ref(layout_C.capacity(extent_C));
      cutlass::device_memory::copy_to_host(matrix_D_Ref.data(), block_Ref.get(), matrix_D_Ref.size());
```
**EN**: verify() begins by rebuilding layout objects and tensor views, then computes a reference GEMM result per batch using CUTLASS’s reference device GEMM helper. The result is copied back to host memory because the softmax reference is computed on the CPU in the next block.
**CN**: verify() 先重建布局对象与 TensorView，然后对每个 batch 调用 CUTLASS 的参考设备 GEMM 例程生成参考矩阵。之所以要把结果拷回主机，是因为下一段会在 CPU 上继续计算 softmax 参考值。

```cpp
      cutlass::TensorView<ElementD, LayoutC> view_Ref(matrix_D_Ref.data(), layout_C, extent_C);

      std::vector<ElementSoftmax> matrix_Softmax_Ref(layout_C.capacity(extent_C));
      cutlass::TensorView<ElementSoftmax, LayoutC> view_Softmax_Ref(matrix_Softmax_Ref.data(), layout_C, extent_C);

      // Copy computed results to host memory
      std::vector<ElementD> matrix_D(layout_C.capacity(extent_C));
      cutlass::device_memory::copy_to_host(matrix_D.data(), block_D.get() + total_elements_D_per_batch * batch_idx, matrix_D.size());

      std::vector<ElementD> matrix_Softmax(layout_C.capacity(extent_C));
      cutlass::device_memory::copy_to_host(matrix_Softmax.data(), block_Softmax.get() + total_elements_D_per_batch * batch_idx, matrix_Softmax.size());

      // Compute the norm
      for (int m = 0; m < options.problem_size.m(); ++m) {
        reference_N.at({m, 0}) = view_Ref.ref().at({m, 0});
        for (int n = 1; n < options.problem_size.n(); ++n) {
          reference_N.at({m, 0}) = std::max(reference_N.at({m, 0}), ElementNorm(view_Ref.ref().at({m, n})));
        }
      }

      // Compute softmax
      for (int m = 0; m < options.problem_size.m(); ++m) {

        float sum = float();

        for (int n = 0; n < options.problem_size.n(); ++n) {
          sum += std::exp( float(view_Ref.ref().at({m, n})) - float(reference_N.at({m, 0})) );
        }

        float inv_sum = float(1.0f / sum);

        for (int n = 0; n < options.problem_size.n(); ++n) {

          view_Softmax_Ref.ref().at({m, n}) = ElementSoftmax(
            std::exp( float(view_Ref.ref().at({m, n})) - float(reference_N.at({m, 0})) ) * inv_sum
          );
        }
      }

      // Verification checks - set any of these to 'true' to override the verification checks.
      bool verified_D = false;
      bool verified_Softmax = false;

      // Verify softmax output
      if (!verified_D) {
        verified_D = verify_tensor<ElementC>(matrix_D, matrix_D_Ref);
      }

      if (!verified_Softmax) {
        verified_Softmax = verify_tensor<ElementSoftmax>(matrix_Softmax, matrix_Softmax_Ref);
      }

      if (!verified_D || !verified_Softmax) {

        std::cerr << "Verification check failed for tensor Softmax at batch " << batch_idx << "\n";

        // Summarize which checks failed
        if (!verified_D) {
          std::cerr << "Verification of D tensor failed\n";
        }

        if (!verified_Softmax) {
          std::cerr << "Verification of Softmax tensor failed\n";
        }

        return false;
      }

    }

    return true;
  }
```
**EN**: The rest of verify() computes the true row-wise softmax reference: first find each row maximum, then compute the denominator and normalized exponentials, and finally compare both the GEMM output D and the final softmax output against device results. This mirrors the staged algorithm implemented in the headers: the device path materializes D and softmax, while the host path recomputes the same semantics directly for correctness checking.
**CN**: verify() 的剩余部分计算真正的逐行 softmax 参考：先求每一行的最大值，再计算分母与归一化指数值，最后把设备端的 GEMM 输出 D 和最终 softmax 输出分别与参考结果比较。这与头文件中实现的分阶段算法一一对应：设备路径会产生 D 和 softmax，而主机路径则直接重算相同语义用于正确性校验。

```cpp
  /// Profiles
  bool profile() {

    //
    // Profile
    //

    cutlass::Status status = cutlass::Status::kSuccess;
    cudaError_t result;
    cudaEvent_t events[2];
    int const kIterations = options.iterations;

    for (cudaEvent_t &evt : events) {
      result = cudaEventCreate(&evt);
      if (result != cudaSuccess) {
        std::cerr << "cudaEventCreate failed with error " << cudaGetErrorString(result) << std::endl;
        return false;
      }
    }

    result = cudaEventRecord(events[0]);

    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed with error " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    for (int iter = 0; iter < kIterations; ++iter) {

      status = execute_device_kernel();

      if (status != cutlass::Status::kSuccess) {
        std::cerr << "Device execution failed." << std::endl;
        return false;
      }
    }

    result = cudaEventRecord(events[1]);

    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed with error " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    result = cudaDeviceSynchronize();

    if (result != cudaSuccess) {
      std::cerr << "cudaDeviceSynchronize() failed with error " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    float elapsed_ms = 0;
    result = cudaEventElapsedTime(&elapsed_ms, events[0], events[1]);

    if (result != cudaSuccess) {
      std::cerr << "cudaEventElapsedTime() failed with error " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    for (cudaEvent_t &evt : events) {
      result = cudaEventDestroy(evt);
      if (result != cudaSuccess) {
        std::cerr << "cudaEventDestroy() failed with error " << cudaGetErrorString(result) << std::endl;
        return false;
      }
    }

    int64_t flops = int64_t(options.problem_size.m()) * options.problem_size.n() * options.problem_size.k() * 2;
    int64_t bytes = cutlass::bits_to_bytes<int64_t>(
      (cutlass::sizeof_bits<ElementD>::value * 2 + cutlass::sizeof_bits<ElementSoftmax>::value) *
      options.problem_size.m() * options.problem_size.n());

    double gflops_per_second = double(flops) * kIterations * options.batch_count / double(elapsed_ms / 1000.0f) / double(1.0e9);
    double gbytes_per_second = double(bytes) * kIterations * options.batch_count / double(elapsed_ms / 1000.0f) / double(1 << 30);

    double elapsed_ms_per_iter = double(elapsed_ms) / kIterations;

    std::cout << "         Problem: "
              << options.problem_size.m() << "-by-" << options.problem_size.n() << "-by-" << options.problem_size.k()
              << ", batch size: " << options.batch_count
              << std::endl;

    std::cout << "         Runtime: " << elapsed_ms_per_iter << " ms\n" << std::endl;

    std::cout << "          GFLOPs: " << gflops_per_second << "  GFLOPs" << std::endl;
    std::cout << "Memory bandwidth: " << gbytes_per_second << "  GiB/s" << std::endl;

    return true;
  }
};
```
**EN**: profile() measures repeated end-to-end execution with CUDA events. After timing kIterations runs, it reports elapsed time per iteration, arithmetic throughput, and an approximate memory bandwidth figure computed from D and softmax traffic. The profile is intentionally simple but good enough to show how the fused pipeline behaves.
**CN**: profile() 使用 CUDA event 统计端到端重复执行时间。在计时 kIterations 次运行后，它输出每次迭代的耗时、算术吞吐率，以及根据 D 和 softmax 读写流量估算出的内存带宽。这个性能测试逻辑刻意保持简单，但足以展示融合流水线的表现。

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, const char **argv) {

  // Options parsing
  Options options;
  options.parse(argc, argv);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }

  if (!options.supported()) {
    return 0;
  }

  // Run
  Testbed testbed(options);

  Disposition disposition = testbed.run();

  std::cout << std::endl;

  switch (disposition) {
    case Disposition::kPassed:
      std::cout << "Passed" << std::endl;
      break;
    case Disposition::kIncorrect:
      std::cout << "Incorrect" << std::endl;
      break;
    case Disposition::kNotVerified:
      std::cout << "Not verified" << std::endl;
      break;
  }

  return (disposition == Disposition::kPassed ? 0 : -1);
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: main() wires the example together: parse options, print help when requested, exit early on unsupported hardware, run the testbed, print the final status, and return success only when verification passed.
**CN**: main() 把整个示例串起来：解析参数、在需要时打印帮助、在硬件不满足要求时提前退出、运行测试夹具、打印最终状态，并且只有在验证通过时才返回成功。

---
## Key Concepts / 关键概念
- Example-level device operator instantiation / 示例级设备操作器实例化
- Batched GEMM + softmax verification flow / batched GEMM + softmax 验证流程
- Reference checking via CUTLASS GEMM plus host softmax / 通过 CUTLASS 参考 GEMM 与主机 softmax 进行校验
- ApplyShape tuning for the softmax apply stage / softmax 应用阶段的 ApplyShape 调优
- Workspace sizing for partial row statistics / 局部逐行统计工作区尺寸设计
- CUDA-event end-to-end profiling / 基于 CUDA event 的端到端性能测试
## Dependencies / 依赖项
- `gemm_with_softmax.h` — defines the composed GEMM+softmax pipeline launched by the example / 定义示例实际启动的 GEMM+softmax 组合流水线
- `cutlass/util/command_line.h` — parses CLI flags into the Options struct / 把命令行参数解析到 Options 结构体
- `cutlass/util/host_tensor.h` — host-side tensor storage for reference data / 用于参考数据的主机侧张量存储
- `cutlass/util/reference/device/gemm_complex.h` — reference device GEMM used during verification / 验证阶段使用的参考设备 GEMM
- `cutlass/util/reference/device/tensor_fill.h` and `cutlass/util/reference/host/tensor_fill.h` — random and zero initialization helpers / 随机填充与清零初始化工具
- `cutlass/layout/matrix.h` — packed layout helpers for leading-dimension computation / 计算 leading dimension 所需的布局辅助接口
- `cutlass/epilogue/thread/linear_combination.h` — alpha/beta epilogue functor used by the GEMM stage / GEMM 阶段使用的 alpha/beta 线性组合 epilogue functor
