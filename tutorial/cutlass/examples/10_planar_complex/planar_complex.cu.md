# planar_complex.cu — Code Analysis / 代码分析

## Source / 源文件

`examples/10_planar_complex/planar_complex.cu`

## Purpose / 用途

**EN:** Demonstrates CUTLASS Library dispatch for strided-batched planar complex GEMM. Complex matrices are stored as separate real and imaginary planes, benchmarked with complex alpha/beta, and validated against a device reference implementation.

**CN:** 演示 CUTLASS Library 分发接口执行跨步批处理平面复数 GEMM。复数矩阵以实部平面和虚部平面分离存储，使用复数 alpha/beta 计时，并用设备端参考实现校验。

## Line-by-Line Analysis / 逐行分析

### Lines 1-75: license, build notes, and planar-complex overview / 许可证、构建说明与平面复数概览

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
/*! \file
    \brief Planar Complex GEMM

  This example demonstrates the CUTLASS Library's exposure of planar complex GEMM kernels supporting
  the batched strided mode.

  These kernels represent complex matrices by storing the real and imaginary parts of the matrix in
  disjoint regions in memory. These real-valued matrices are stored using existing cuBLAS layouts
  as either column-major or row-major layouts with a single leading dimension indicating the stride
  between columns or rows.

  The CUTLASS Library collects multiple template instantiations in a data structure and offers
  a BLAS-like dispatch API to invoke the appropriate kernel on the Volta or Turing architectures.

  CUTLASS decouples matrix layout from complex transformation, so four possible transformations
  are possible on the A and B operands:

    n:  column-major
    c:  column-major complex conjugate
    t:  row-major
    h:  row-major complex conjugate

  The CUTLASS Library contains many kernel instances specialized for architecture, data type, tile
  size, and alignment. This can result in long compile times.

  To build strictly the planar complex kernels needed for general application, execute the following
  CMake command in an empty build directory.
    
    $ cmake .. -DCUTLASS_NVCC_ARCHS="70;75;80" \
  	  -DCUTLASS_LIBRARY_KERNELS=cutlass_tensorop_*gemm_planar_complex

  This builds all planar complex GEMM variants for Volta and Turing architectures.

  To build strictly the kernels needed for this example, an even narrower filter string may be
  specified as follows. This only builds planar complex GEMMs targeting Tensor Cores for
  the 'CN' layout configuration (conjugate A operand with both A and B as column-major).

    $ cmake .. -DCUTLASS_NVCC_ARCHS="70;75;80" \
  	  -DCUTLASS_LIBRARY_KERNELS=cutlass_tensorop_f16_s*gemm_planar_complex_f16*cn

    $ make 10_planar_complex

    $ ./examples/10_planar_complex/10_planar_complex --m=2048 --n=1024 --k=512 --batch=10
*/

```

**EN:** The file comment explains the BSD license, the example goal, and the planar complex representation. Real and imaginary values live in disjoint regions, while each region uses normal row-major or column-major real layout. The documented transform letters mean: n = column-major no conjugate, c = column-major conjugate, t = row-major no conjugate, h = row-major conjugate. The CMake filters select only planar-complex library kernels to reduce compile time.

**CN:** 文件注释说明 BSD 许可证、示例目标和平面复数表示。实部和虚部位于互不重叠的内存区域，而每个区域本身使用普通行主序或列主序实数布局。注释中的变换字母含义为：n=列主序不共轭，c=列主序共轭，t=行主序不共轭，h=行主序共轭。CMake 过滤器用于只构建平面复数 Library kernel，从而减少编译时间。

### Lines 76-94: includes / 头文件

```cpp
#include <iostream>
#include <sstream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/host_tensor_planar_complex.h"

#include "cutlass/util/reference/device/tensor_fill.h"

#include "cutlass/util/reference/device/gemm_planar_complex.h"
#include "cutlass/util/reference/device/tensor_compare.h"

#include "cutlass/library/handle.h"

```

**EN:** The includes bring in CUTLASS core types, GEMM coordinates, command-line parsing, random/device-memory utilities, device reference planar-complex GEMM, tensor comparison, and the runtime Library handle.

**CN:** 这些头文件提供 CUTLASS 核心类型、GEMM 坐标、命令行解析、随机数与设备内存工具、设备端平面复数参考 GEMM、张量比较，以及运行时 Library 句柄。

### Lines 95-202: Result and Options / 结果结构与命令行选项

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Result structure
struct Result {

  double runtime_ms;
  double gflops;
  cutlass::Status status;
  cudaError_t error;
  bool passed;

  //
  // Methods
  //

  Result(
    double runtime_ms = 0,
    double gflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cudaError_t error = cudaSuccess
  ):
    runtime_ms(runtime_ms), gflops(gflops), status(status), error(error), passed(true) { }
};

///////////////////////////////////////////////////////////////////////////////////////////////////

// Command line options parsing
struct Options {

  bool help;

  cutlass::gemm::GemmCoord problem_size;
  int batch_count;
  cutlass::complex<float> alpha;
  cutlass::complex<float> beta;

  bool reference_check;
  int iterations;
  
  Options():
    help(false),
    problem_size({1024, 1024, 1024}),
    batch_count(1),
    reference_check(true),
    iterations(20),
    alpha(1),
    beta() { }

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
    cmd.get_cmd_line_argument("batch", batch_count);

    cmd.get_cmd_line_argument("alpha", alpha.real());
    cmd.get_cmd_line_argument("alpha_i", alpha.imag());
    cmd.get_cmd_line_argument("beta", beta.real());
    cmd.get_cmd_line_argument("beta_i", beta.imag());
    
    cmd.get_cmd_line_argument("iterations", iterations);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "10_planar_complex example\n\n"
      << "  This example uses the CUTLASS Library to execute Planar Complex GEMM computations.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --m=<int>                   GEMM M dimension\n"
      << "  --n=<int>                   GEMM N dimension\n"
      << "  --k=<int>                   GEMM K dimension\n"
      << "  --batch=<int>               Number of GEMM operations executed in one batch\n"
      << "  --alpha=<f32>               Epilogue scalar alpha (real part)\n"
      << "  --alpha_i=<f32>             Epilogue scalar alpha (imaginary part)\n"
      << "  --beta=<f32>                Epilogue scalar beta (real part)\n\n"
      << "  --beta_i=<f32>              Epilogue scalar beta (imaginary part)\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n\n";

    out << "\n\nExamples:\n\n"
      << "$ ./examples/10_planar_complex/10_planar_complex  --batch=7 --m=1024 --n=512 --k=1024 \\\n"
      << "     --alpha=2 --alpha_i=-2 --beta=0.707 --beta_i=-.707\n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {

    // Number of real-valued multiply-adds 
    int64_t fmas = problem_size.product() * batch_count * 4;
    
    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};

```

**EN:** Result stores runtime, throughput, CUTLASS status, CUDA error, and pass/fail state. Options stores M/N/K, batch count, complex alpha and beta, validation control, and iteration count. Parsing splits complex scalars into real and imaginary command-line parameters. The GFLOP/s estimate counts a complex multiply-add as four real FMAs, then two FLOPs per FMA.

**CN:** Result 保存运行时间、吞吐量、CUTLASS 状态、CUDA 错误和通过/失败状态。Options 保存 M/N/K、批次数、复数 alpha/beta、校验开关和迭代次数。解析时复数标量被拆成实部和虚部命令行参数。GFLOP/s 估算把一次复数乘加视为四个实数 FMA，并按每个 FMA 两个 FLOP 计算。

### Lines 203-248: testbed types and allocation / 测试平台类型与分配

```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////

/// Performance test environment for planar complex
class TestbedPlanarComplex {
public:

  using ElementA = cutlass::half_t;
  using LayoutA = cutlass::layout::ColumnMajor;
  using ElementB = cutlass::half_t;
  using LayoutB = cutlass::layout::ColumnMajor;
  using ElementC = cutlass::half_t;
  using LayoutC = cutlass::layout::ColumnMajor;
  using ElementCompute = float;
  using ElementAccumulator = float;

  //
  // Data members
  //

  cutlass::library::Handle handle;

  cutlass::gemm::GemmCoord problem_size;
  int batch_count;
  cutlass::DeviceAllocation<ElementA> tensor_A;
  cutlass::DeviceAllocation<ElementB> tensor_B;
  cutlass::DeviceAllocation<ElementC> tensor_C;
  cutlass::DeviceAllocation<ElementC> tensor_D;
  cutlass::DeviceAllocation<ElementC> tensor_D_ref;

  //
  // Methods
  //

  TestbedPlanarComplex(
    Options const &options
  ): 
    problem_size(options.problem_size), batch_count(options.batch_count) {

    // Allocate device memory for batched strided GEMM
    tensor_A.reset(int64_t(problem_size.m()) * problem_size.k() * batch_count * 2);
    tensor_B.reset(int64_t(problem_size.k()) * problem_size.n() * batch_count * 2);
    tensor_C.reset(int64_t(problem_size.m()) * problem_size.n() * batch_count * 2);
    tensor_D.reset(int64_t(problem_size.m()) * problem_size.n() * batch_count * 2);
    tensor_D_ref.reset(int64_t(problem_size.m()) * problem_size.n() * batch_count * 2);
  }

```

**EN:** The testbed uses half-precision storage for A/B/C/D planes, column-major layouts, and float accumulation/compute. Each allocation multiplies by two because every batch contains one real plane and one imaginary plane.

**CN:** 测试平台对 A/B/C/D 平面使用半精度存储、列主序布局以及 float 累加/计算。每个分配都乘以二，因为每个 batch 同时包含一个实部平面和一个虚部平面。

### Lines 249-292: initialization and planar strides / 初始化与平面跨距

```cpp
  void initialize() {

    uint64_t seed = 1073;

    // Use small integers to simplify correctness checking
    int scope_max = 6;
    int scope_min = -6;

    cutlass::reference::device::BlockFillRandomUniform(
        tensor_A.get(), tensor_A.size(), seed, ElementA(scope_max), ElementA(scope_min), 0);

    cutlass::reference::device::BlockFillRandomUniform(
        tensor_B.get(), tensor_B.size(), seed * 2019, ElementB(scope_max), ElementB(scope_min), 0);

    cutlass::reference::device::BlockFillRandomUniform(
        tensor_C.get(), tensor_C.size(), seed * 2020, ElementC(scope_max), ElementC(scope_min), 0);
  }

  Result profile(Options const &options) {

    Result result;

    initialize();

    ElementA *ptr_A = tensor_A.get();
    ElementB *ptr_B = tensor_B.get();
    ElementC *ptr_C = tensor_C.get();
    ElementC *ptr_D = tensor_D.get();

    int64_t batch_stride_A = int64_t(problem_size.m()) * problem_size.k() * 2;
    int64_t batch_stride_B = int64_t(problem_size.k()) * problem_size.n() * 2;
    int64_t batch_stride_C = int64_t(problem_size.m()) * problem_size.n() * 2;
    int64_t batch_stride_D = int64_t(problem_size.m()) * problem_size.n() * 2;

    typename LayoutA::Stride::Index lda = LayoutA::packed({problem_size.m(), problem_size.k()}).stride(0);
    typename LayoutB::Stride::Index ldb = LayoutB::packed({problem_size.k(), problem_size.n()}).stride(0);
    typename LayoutC::Stride::Index ldc = LayoutC::packed({problem_size.m(), problem_size.n()}).stride(0);
    typename LayoutC::Stride::Index ldd = LayoutC::packed({problem_size.m(), problem_size.n()}).stride(0);

    int64_t imag_stride_A = int64_t(problem_size.m()) * problem_size.k();
    int64_t imag_stride_B = int64_t(problem_size.k()) * problem_size.n();
    int64_t imag_stride_C = int64_t(problem_size.m()) * problem_size.n();
    int64_t imag_stride_D = int64_t(problem_size.m()) * problem_size.n();

```

**EN:** Initialization fills all planes with small random values. In profile(), base pointers are captured, batch strides include both real and imaginary planes, leading dimensions describe one real-valued plane, and imag_stride is the offset from a batch real plane to its imaginary plane.

**CN:** 初始化为所有平面填充小范围随机值。在 profile() 中获取基指针；batch stride 覆盖实部和虚部两个平面；leading dimension 描述单个实数平面；imag_stride 是同一 batch 中从实部平面到虚部平面的偏移。

### Lines 293-392: timing and `gemm_planar_complex` dispatch / 计时与 `gemm_planar_complex` 分发

```cpp
    //
    // Construct events
    //

    cudaEvent_t events[2];

    for (auto & event : events) {
      result.error = cudaEventCreate(&event);
      if (result.error != cudaSuccess) {
        std::cerr << "cudaEventCreate() failed: " << cudaGetErrorString(result.error) << std::endl;
        return -1;
      }
    }

    // Record an event at the start of a series of GEMMs
    result.error = cudaEventRecord(events[0]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    //
    // Run profiling loop
    //

    for (int iter = 0; iter < options.iterations; ++iter) {

      //
      // Execute the planar complex GEMM kernel via the CUTLASS Library's
      // dispatch routines.
      //
      // Note, for planar complex GEMM kernels, all numeric type arguments 
      // specify the data type of the base real types. These are understood to
      // apply to planar complex representations of matrices in memory and to complex<T>
      // structures for scalars.
      //
      // See tools/library/include/cutlass/library/handle.h for more details.
      //

      result.status = handle.gemm_planar_complex(
        problem_size.m(),                                 // GEMM M dimension
        problem_size.n(),                                 // GEMM N dimension
        problem_size.k(),                                 // GEMM K dimension

        cutlass::library::NumericTypeID::kF32,            // Base data type of complex-valued accumulation
        cutlass::library::NumericTypeID::kF32,            // Base data type of complex-valued alpha/beta scalars

        &options.alpha,                                   // Pointer to alpha scalar, of type complex<T>

        cutlass::library::NumericTypeID::kF16,            // Base data type of complex-valued A matrix
        cutlass::library::LayoutTypeID::kColumnMajor,     // Layout of A matrix
        cutlass::library::ComplexTransform::kConjugate,   // Complex transformation on A matrix operand
        ptr_A,                                            // Pointer to real part of A matrix
        ptr_A + imag_stride_A,                            // Pointer to imaginary part of A matrix
        lda,                                              // Leading dimension of real part of A matrix
        lda,                                              // Leading dimension of imaginary part of A matrix

        cutlass::library::NumericTypeID::kF16,            // Base data type of complex-valued B matrix
        cutlass::library::LayoutTypeID::kColumnMajor,     // Layout of B matrix
        cutlass::library::ComplexTransform::kNone,        // Complex transformation on B matrix operand
        ptr_B,                                            // Pointer to real part of B matrix
        ptr_B + imag_stride_B,                            // Pointer to imaginary part of B matrix
        ldb,                                              // Leading dimension of real part of B matrix
        ldb,                                              // Leading dimension of imaginary part of B matrix

        &options.beta,                                    // Pointer to beta scalar, of type complex<T>

        cutlass::library::NumericTypeID::kF16,            // Base data type of complex valued C and D matrices

        ptr_C,                                            // Pointer to real part of C matrix
        ptr_C + imag_stride_C,                            // Pointer to imaginary part of C matrix
        ldc,                                              // Leading dimension of real part of C matrix
        ldc,                                              // Leading dimension of imaginary part of C matrix

        ptr_D,                                            // Pointer to real part of D matrix
        ptr_D + imag_stride_D,                            // Pointer to imaginary part of D matrix
        ldd,                                              // Leading dimension of real part of D matrix
        ldd,                                              // Leading dimension of imaginary part of D matrix

        batch_count,                                      // Number of batched elements

        batch_stride_A,                                   // Stride between batches of real parts of A matrix
        batch_stride_A,                                   // Stride between batches of imaginary parts of A matrix

        batch_stride_B,                                   // Stride between batches of real parts of B matrix
        batch_stride_B,                                   // Stride between batches of imaginary parts of B matrix

        batch_stride_C,                                   // Stride between batches of real parts of C matrix
        batch_stride_C,                                   // Stride between batches of imaginary parts of C matrix

        batch_stride_D,                                   // Stride between batches of real parts of D matrix
        batch_stride_D                                    // Stride between batches of imaginary parts of D matrix
      );

      if (result.status != cutlass::Status::kSuccess) {
        std::cerr << "CUTLASS internal error - configuration not supported" << std::endl;
        return result;
      }
    }
    
```

**EN:** CUDA events measure the repeated GEMM loop. The Library call supplies M/N/K, accumulation and scalar base types, complex alpha, A metadata, B metadata, complex beta, C/D metadata, batch count, and per-plane batch strides. A is half planar complex column-major with conjugation; B is half planar complex column-major without conjugation. The selected epilogue is the standard complex linear combination D = alpha*conj(A)*B + beta*C.

**CN:** CUDA 事件用于测量重复 GEMM 循环。Library 调用传入 M/N/K、累加器和标量基础类型、复数 alpha、A 元数据、B 元数据、复数 beta、C/D 元数据、批次数以及各平面的 batch stride。A 是半精度列主序平面复数并做共轭；B 是半精度列主序平面复数且不共轭。所选 epilogue 是标准复数线性组合 D = alpha*conj(A)*B + beta*C。

### Lines 393-431: timing finalization / 结束计时

```cpp
    //
    // Stop profiling loop
    //

    // Record an event when the GEMMs are complete
    result.error = cudaEventRecord(events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Wait for work on the device to complete.
    result.error = cudaEventSynchronize(events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventSynchronize() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Measure elapsed runtime
    float runtime_ms = 0;
    result.error = cudaEventElapsedTime(&runtime_ms, events[0], events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    // Compute average runtime and GFLOPs.
    result.runtime_ms = double(runtime_ms) / double(options.iterations);
    result.gflops = options.gflops(result.runtime_ms / 1000.0);

    // Cleanup
    for (auto event : events) {
      (void)cudaEventDestroy(event);
    }

    if (handle.get_last_operation()) {
      std::cout << "Recently executed '" << handle.get_last_operation()->description().name << "'" << std::endl;
    }

```

**EN:** The stop event is recorded and synchronized, elapsed time is converted to an average per iteration, GFLOP/s is computed, events are destroyed, and the last dispatched library operation name is printed.

**CN:** 记录并同步结束事件，将总耗时换算为每次迭代平均时间，计算 GFLOP/s，销毁事件，并打印 Library 最近分发的操作名称。

### Lines 432-484: reference validation and output / 参考校验与输出

```cpp
    //
    // Compute reference in device code
    //

    if (options.reference_check) {

      result.passed = true;

      for (int64_t idx = 0; result.passed && idx < int64_t(batch_count); ++idx) {
        cutlass::reference::device::GemmPlanarComplex<
          ElementA, LayoutA,
          ElementB, LayoutB,
          ElementC, LayoutC,
          ElementAccumulator
        >(
          problem_size,
          options.alpha,
          {tensor_A.get() + idx * batch_stride_A, lda, imag_stride_A},
          cutlass::ComplexTransform::kConjugate,
          {tensor_B.get() + idx * batch_stride_B, ldb, imag_stride_B},
          cutlass::ComplexTransform::kNone,
          options.beta,
          {tensor_C.get() + idx * batch_stride_C, ldc, imag_stride_C},
          {tensor_D_ref.get() + idx * batch_stride_D, ldd, imag_stride_D}
        );

        ElementC epsilon = 0.1_hf;
        ElementC nonzero_floor = 0.1_hf;

        result.passed = cutlass::reference::device::BlockCompareRelativelyEqual(
          tensor_D.get() + idx * batch_stride_D,
          tensor_D_ref.get() + idx * batch_stride_D,
          batch_stride_D,
          epsilon,
          nonzero_floor
        );
      }

      if (result.passed) {
        std::cout << "Reference check passed." << std::endl;
      }
      else {
        std::cerr << "Error - reference check failed." << std::endl;
      }
    }

    std::cout << "Runtime: " << result.runtime_ms << " ms" << std::endl;
    std::cout << " GFLOPs: " << result.gflops << std::endl;

    return result;
  }
};

```

**EN:** For each batch, the device reference GemmPlanarComplex receives planar tensor references of the form {real pointer, leading dimension, imaginary stride}. The output buffer is compared against the reference across both planes using half-precision tolerances, then runtime and throughput are printed.

**CN:** 对每个 batch，设备端参考 GemmPlanarComplex 接收形如 {实部指针, leading dimension, 虚部偏移} 的平面张量引用。随后用半精度容差比较输出和参考结果的两个平面，并打印运行时间和吞吐量。

### Lines 485-567: main program and guards / 主程序与运行条件检查

```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, char const **args) {

  //
  // This example uses mma.sync to directly access Tensor Cores to achieve peak performance.
  //
  // Volta Tensor Core operations are first available in CUDA 10.1 Toolkit.
  //
  // Turing Tensor Core operations are first available in CUDA 10.2 Toolkit.
  //

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (props.major < 7) {
    std::cerr << "Volta Tensor Core operations must be run on a machine with compute capability at least 70."
              << std::endl;

    // Returning zero so this test passes on older architectures even though its actions are no-op.
    return 0;
  }
  else if (props.major == 7 && props.minor <= 2) {
    //
    // If running on the Volta architecture, at least CUDA 10.1 Toolkit is required to run this example.
    //
    if (!(__CUDACC_VER_MAJOR__ > 10 || (__CUDACC_VER_MAJOR__ == 10 && __CUDACC_VER_MINOR__ >= 1))) {
      std::cerr << "Volta Tensor Core operations must be compiled with CUDA 10.1 Toolkit or later." << std::endl;

      // Returning zero so this test passes on older Toolkits even though its actions are no-op.
      return 0;
    }
  }
  else if (props.major == 7 && props.minor >= 5) {
    //
    // If running on the Turing architecture, at least CUDA 10.2 Toolkit is required to run this example.
    //
    if (!(__CUDACC_VER_MAJOR__ > 10 || (__CUDACC_VER_MAJOR__ == 10 && __CUDACC_VER_MINOR__ >= 2))) {
      std::cerr << "Turing Tensor Core operations must be compiled with CUDA 10.2 Toolkit or later." << std::endl;
    
      // Returning zero so this test passes on older Toolkits even though its actions are no-op.
      return 0;
    }
  }
  else {
    // NVIDIA Ampere Architecture GPUs (SM80 and later) are fully supported on CUDA 11 Toolkit and beyond.
    //
    // fall through
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

  // Execute one problem size
  if (!options.valid()) {
    std::cerr << "Invalid problem." << std::endl;
    return -1;
  }

  TestbedPlanarComplex testbed(options);

  Result result = testbed.profile(options);

  return result.passed ? 0 : -1;
}

/////////////////////////////////////////////////////////////////////////////////////////////////

```

**EN:** main() checks GPU capability and minimum CUDA Toolkit support for Volta/Turing Tensor Core paths, parses options, handles --help, constructs the testbed, runs profiling, and returns success only when validation passes. Unsupported old environments return 0 as no-op for test friendliness.

**CN:** main() 检查 GPU 能力以及 Volta/Turing Tensor Core 路径所需的最低 CUDA Toolkit 版本，解析参数，处理 --help，构造测试平台，运行 profiling，并仅在校验通过时返回成功。过旧的不支持环境以 no-op 方式返回 0，方便测试。

## Key Concepts / 关键概念

**EN:**
- **Planar complex storage:** real and imaginary parts are separate dense real matrices.
- **Strided batching:** `batch_stride_*` moves between complex batches; `imag_stride_*` moves from real to imaginary plane inside one batch.
- **Complex transforms:** layout and conjugation are independent; this example computes with conjugated A and unmodified B.
- **CUTLASS Library dispatch:** runtime IDs select a pre-instantiated kernel.
- **Epilogue semantics:** built-in complex GEMM epilogue computes `D = alpha * op(A) * op(B) + beta * C`.

**CN:**
- **平面复数存储：** 实部和虚部分别是密集实数矩阵。
- **跨步批处理：** `batch_stride_*` 在复数 batch 之间移动；`imag_stride_*` 在同一 batch 内从实部移动到虚部。
- **复数变换：** 布局与共轭相互独立；本例对 A 共轭，对 B 不变换。
- **CUTLASS Library 分发：** 运行时 ID 选择预实例化 kernel。
- **Epilogue 语义：** 内置复数 GEMM epilogue 计算 `D = alpha * op(A) * op(B) + beta * C`。

## Dependencies / 依赖项

**EN:** CUDA runtime, CUTLASS core GEMM types, CUTLASS Library planar-complex kernels, device allocation utilities, reference device fill/GEMM/compare helpers, and SM70+ Tensor Core hardware with a compatible CUDA Toolkit.

**CN:** 依赖 CUDA runtime、CUTLASS GEMM 核心类型、CUTLASS Library 平面复数 kernel、设备内存工具、设备端填充/GEMM/比较参考工具，以及兼容 CUDA Toolkit 的 SM70+ Tensor Core 硬件。
