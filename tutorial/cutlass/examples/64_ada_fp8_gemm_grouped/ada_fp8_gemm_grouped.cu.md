# ada_fp8_gemm_grouped.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/64_ada_fp8_gemm_grouped/ada_fp8_gemm_grouped.cu`  
**Purpose / 用途**: Analyze an Ada FP8 grouped GEMM example with per-group output scaling, scheduling modes, and benchmark flow. / 分析一个带按 group 输出缩放、调度模式与 benchmark 流程的 Ada FP8 grouped GEMM 示例。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-64
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
    \brief Ada FP8 GEMM Grouped With Per-Group Scale Example.

    This workload computes a batch of GEMM operations with distinct problem sizes. Pointers to matrices
    in Global Memory are passed to the kernel in array (also held in Global Memory). Similarly,
    leading dimensions and problem sizes are stored in arrays in GMEM.

    This differs from "Batched Array" GEMM because the size of each GEMM problem in the Grouped GEMM
    concept may be distinct.

    The differences between this and the examples/24_gemm_grouped are: (1) this example scales the output of each GEMM by a different scalar value specified by alpha_ptr_array. (2) this example uses FP8 tensorcore.

    This benchmark program initializes a workspace with random problem sizes for a given number of
    groups. Command line options enable overriding M, N, and/or K dimensions with uniform values to
    model problems more similar to the traditional batched GEMM.

    Additionally, problem sizes are collected and binned to compute the same problem as a series of
    conventional batched GEMMs (setup for this problem is not timed). This demonstrates the performance
    enhancement achieved by implementing a specialized grouped GEMM kernel.

    Examples:

      # Runs a grouped GEMM with 100 random problem sizes
      $ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --groups=100

      # Runs a grouped GEMM with 100 random problem sizes (with GEMM-K dimension equal to 1024)
      $ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --groups=100 --k=1024 --verbose=true

      # Runs a grouped GEMM that is equivalent to a batched GEMM
      $ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --groups=100 --m=2048 --n=1024 --k=1024 --verbose=true

      # Execute Grouped GEMM and profile with NSight
      $ nv-nsight-cu-cli ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --m=256 --n=256 --k=256 --verbose=true \
```
- **EN**: The file opens with the BSD-3-Clause license and a detailed example-level header. The header explains that this is an Ada FP8 grouped GEMM example where each GEMM in the group may have a different shape and its own output scaling factor alpha.
- **CN**: 文件首先给出 BSD-3-Clause 许可证与示例级说明。说明指出这是一个 Ada FP8 grouped GEMM 示例：组内每个 GEMM 都可以有不同尺寸，并且拥有各自的输出缩放系数 alpha。

### Lines 65-96
```cpp
                                                                    --iterations=1 --reference-check=false

*/

/////////////////////////////////////////////////////////////////////////////////////////////////

#include <chrono>
#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <map>
#include <unordered_map>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/kernel/gemm_grouped_per_group_scale.h"
#include "cutlass/gemm/kernel/default_gemm_grouped_per_group_scale.h"
#include "cutlass/gemm/device/gemm_grouped.h"
#include "cutlass/gemm/device/gemm_universal.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/host/gemm_complex.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/device/tensor_fill.h"
```
- **EN**: This block includes standard C++ utilities plus CUTLASS GEMM, grouped-kernel, device wrapper, command-line, tensor, and reference-check headers. Together they supply problem description types, grouped dispatch machinery, memory helpers, random initialization, and correctness validation.
- **CN**: 这一段引入标准 C++ 工具头文件，以及 CUTLASS 的 GEMM、grouped kernel、device 封装、命令行、张量与参考校验头文件。它们共同提供问题描述类型、grouped 调度机制、内存辅助、随机初始化与正确性验证能力。

### Lines 97-127
```cpp
#include "cutlass/util/reference/host/tensor_norm.h"

/////////////////////////////////////////////////////////////////////////////////////////////////

/// Result structure
struct Result {

  double runtime_ms;
  double initialization_time_ms;
  double gflops;
  cutlass::Status status;
  cudaError_t error;
  bool passed;

  //
  // Methods
  //

  Result(
    double runtime_ms = 0,
    double initialization_time_ms = 0,
    double gflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cudaError_t error = cudaSuccess
  ):
    runtime_ms(runtime_ms), initialization_time_ms(initialization_time_ms), gflops(gflops),
    status(status), error(error), passed(true) { }
};

/////////////////////////////////////////////////////////////////////////////////////////////////

```
- **EN**: Result is a small POD-style container for benchmark outputs: kernel runtime, optional initialization time, throughput, CUTLASS status, CUDA error, and pass/fail state. Its constructor defaults to a successful zero-valued result so later code can early-return safely.
- **CN**: Result 是一个轻量结果容器，用于保存 benchmark 输出：内核运行时间、可选初始化时间、吞吐率、CUTLASS 状态、CUDA 错误和通过/失败标志。构造函数默认生成“成功且数值为零”的结果，便于后续代码安全地提前返回。

### Lines 128-137
```cpp
/// Hash function for cutlass::gemm::GemmCoord
struct HashGemmCoord {
  size_t operator()(cutlass::gemm::GemmCoord const &problem) const {
    std::hash<int> hasher;
    return (hasher(problem.m() * 3)) ^ (hasher(1 + problem.n() * 5)) ^ (hasher(2 + problem.k() * 7));
  }
};

/////////////////////////////////////////////////////////////////////////////////////////////////

```
- **EN**: HashGemmCoord defines a custom hash for GemmCoord so problem sizes can be used as keys in unordered_map. The hash mixes M, N, and K with small multipliers, which is sufficient for grouping identical shapes in this sample.
- **CN**: HashGemmCoord 为 GemmCoord 提供自定义哈希，使问题尺寸可以作为 unordered_map 的键。它对 M、N、K 做简单混合，在本示例中足以用于把相同形状归入同一桶。

### Lines 138-194
```cpp
// Command line options parsing
struct Options {

  bool help;
  bool error;
  bool reference_check;
  bool profile_initialization;
  bool sort_problems;

  std::vector<cutlass::gemm::GemmCoord> problem_sizes;

  // problem size bins
  std::unordered_map<
    cutlass::gemm::GemmCoord,
    std::vector<int32_t>,
    HashGemmCoord> problem_bins;

  int alignment;
  int problem_count;
  int iterations;
  int cuda_streams;
  bool verbose;
  float alpha;
  std::vector<float> alpha_array;
  float beta;
  std::string benchmark_path;

  std::string   output_tag;
  std::ofstream output_file;

  using GroupScheduleMode = cutlass::gemm::kernel::GroupScheduleMode;
  std::vector<GroupScheduleMode> scheduler_modes;

  std::unordered_map<std::string, GroupScheduleMode>
    str_to_scheduler_mode = {
      {"kDeviceOnly", GroupScheduleMode::kDeviceOnly},
      {"kHostPrecompute", GroupScheduleMode::kHostPrecompute}
    };

  struct GroupScheduleModeHash {
    size_t operator()(GroupScheduleMode m) const {
      return static_cast<size_t>(m);
    }
  };

  std::unordered_map<GroupScheduleMode, std::string, GroupScheduleModeHash>
    scheduler_mode_to_str = {
      {GroupScheduleMode::kDeviceOnly, "kDeviceOnly"},
      {GroupScheduleMode::kHostPrecompute, "kHostPrecompute"}
    };

  std::vector<GroupScheduleMode> all_scheduler_modes = {GroupScheduleMode::kDeviceOnly, GroupScheduleMode::kHostPrecompute};

  //
  // Methods
  //

```
- **EN**: Options stores CLI switches, the generated or loaded problem list, per-shape bins, scalar settings, scheduler-mode maps, and CSV-output state. The scheduler strings map to CUTLASS grouped scheduling modes: device-only scheduling or host-precomputed scheduling.
- **CN**: Options 保存命令行开关、生成或加载的问题列表、按形状分桶的结果、标量设置、调度模式映射以及 CSV 输出状态。这里的字符串会映射到 CUTLASS 的 grouped 调度模式：纯设备端调度或主机端预计算调度。

### Lines 195-211
```cpp
  Options():
    help(false),
    error(false),
    alignment(16),
    reference_check(true),
    profile_initialization(false),
    sort_problems(false),
    problem_count(15),
    iterations(20),
    cuda_streams(0),
    verbose(false),
    alpha(1),
    beta(),
    scheduler_modes({GroupScheduleMode::kDeviceOnly})
  { }

  // Parses the command line
```
- **EN**: The constructor establishes default benchmark behavior: 16-element alignment, 15 grouped problems, 20 timing iterations, reference checking enabled, alpha=1, beta default-initialized to 0, and the scheduler defaulting to kDeviceOnly.
- **CN**: 构造函数设定默认 benchmark 行为：16 元素对齐、15 个 grouped 问题、20 次计时迭代、开启参考校验、alpha=1、beta 默认初始化为 0，并将调度模式默认设为 kDeviceOnly。

### Lines 212-297
```cpp
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("alignment", alignment, 16);
    cmd.get_cmd_line_argument("groups", problem_count, 15);
    cmd.get_cmd_line_argument("alpha", alpha, 1.0f);
    cmd.get_cmd_line_argument("beta", beta, 0.0f);
    cmd.get_cmd_line_argument("iterations", iterations, 20);
    cmd.get_cmd_line_argument("streams", cuda_streams, 0);
    cmd.get_cmd_line_argument("verbose", verbose, false);
    cmd.get_cmd_line_argument("reference-check", reference_check, true);
    cmd.get_cmd_line_argument("profile-initialization", profile_initialization, false);
    cmd.get_cmd_line_argument("sort-problems", sort_problems, false);
    cmd.get_cmd_line_argument("benchmark", benchmark_path);

    std::vector<std::string> scheduler_mode_strs;
    cmd.get_cmd_line_arguments("scheduler-modes", scheduler_mode_strs);

    if (!scheduler_mode_strs.empty()) {
      scheduler_modes.clear();
      if (scheduler_mode_strs.size() == 1 && scheduler_mode_strs[0] == "all") {
        scheduler_modes = all_scheduler_modes;
      } else {
        for (std::string precomp_str : scheduler_mode_strs) {
          auto it = str_to_scheduler_mode.find(precomp_str);
          if (it != str_to_scheduler_mode.end()) {
            scheduler_modes.push_back(it->second);
          } else if (precomp_str == "all") {
            std::cerr << "Flag --scheduler-modes=all must not contain other scheduler modes in list." << std::endl;
            error = true;
            return;
          } else {
            std::cerr << "Unrecognized scheduler mode '" << precomp_str << "'" << std::endl;
            error = true;
            return;
          }
        }
      }
    }

    std::string output_path;
    cmd.get_cmd_line_argument("tag", output_tag);
    cmd.get_cmd_line_argument("output_file", output_path);

    if (!output_path.empty()) {

      std::ios_base::openmode open_mode = std::ios_base::out;

      std::ifstream input_file(output_path.c_str());

      if (input_file.good()) {
        open_mode = std::ios_base::app;
        input_file.close();
      }

      output_file.open(output_path.c_str(), open_mode);

      if (output_file.good() && open_mode != std::ios_base::app) {
        output_file << "Tag,Provider,Kind,Groups,Runtime,GFLOPs\n";
      }
    }

    // Decide how to initialize the problems
    if (!benchmark_path.empty()) {
      if (!benchmark_problems()) {
        error = true;
        problem_sizes.clear();
        return;
      }
    }
    else {
      randomize_problems(cmd);
    }

    // Post-process the problem sizes
    bin_problems();

    // Initialize alpha array
    randomize_alpha_ptr_array(cmd);
  }

```
- **EN**: parse() reads CLI flags, validates scheduler-mode strings, prepares optional CSV output, then chooses either benchmark-file loading or random problem generation. After the problem list is ready, it bins identical shapes and builds a per-group alpha array for the epilogue.
- **CN**: parse() 负责读取命令行参数、校验 scheduler 模式字符串、准备可选 CSV 输出，然后在“基准文件加载”和“随机生成问题”之间二选一。问题列表准备好后，它会将相同形状分桶，并为 epilogue 构建按 group 区分的 alpha 数组。

### Lines 298-337
```cpp
  void randomize_problems(cutlass::CommandLine &cmd) {

    //
    // For now, randomly choose the problem sizes.
    //

    int cmd_line_m = -1;
    int cmd_line_n = -1;
    int cmd_line_k = -1;

    cmd.get_cmd_line_argument("m", cmd_line_m);
    cmd.get_cmd_line_argument("n", cmd_line_n);
    cmd.get_cmd_line_argument("k", cmd_line_k);

    problem_sizes.reserve(problem_count);

    for (int i = 0; i < problem_count; ++i) {

      int m = cmd_line_m;
      int n = cmd_line_n;
      int k = cmd_line_k;

      if (m < 1) {
        m = alignment * ((rand() % 256) + 1);
      }

      if (n < 1) {
        n = alignment * ((rand() % 256) + 1);
      }

      if (k < 1) {
        k = alignment * ((rand() % 256) + 1);
      }

      cutlass::gemm::GemmCoord problem(m, n, k);

      problem_sizes.push_back(problem);
    }
  }

```
- **EN**: randomize_problems() optionally respects uniform --m/--n/--k overrides; otherwise it draws each dimension randomly in aligned multiples of 16. The result is a true grouped GEMM workload where each group may carry a different MxNxK triple.
- **CN**: randomize_problems() 会优先使用统一的 --m/--n/--k 覆盖值；否则按 16 的倍数随机生成每个维度。生成后的工作负载就是典型的 grouped GEMM：每个 group 都可能有不同的 MxNxK 组合。

### Lines 338-345
```cpp
  void randomize_alpha_ptr_array(cutlass::CommandLine &cmd) {
    alpha_array.resize(problem_count);
    for (int i = 0; i < problem_count; ++i) {
        alpha_array[i] = static_cast<float>((rand() % 100) - 50 + alpha);
    }
  }

  /// Load a benchmark
```
- **EN**: randomize_alpha_ptr_array() creates one scaling value per problem. Each value is randomized around the base alpha, which later becomes the per-group epilogue scale loaded through a device-side pointer array.
- **CN**: randomize_alpha_ptr_array() 为每个问题生成一个缩放值。每个值都围绕基础 alpha 随机变化，后续会作为按 group 区分的 epilogue 缩放系数，通过设备端指针数组读取。

### Lines 346-387
```cpp
  bool benchmark_problems() {
    std::ifstream file(benchmark_path);
    if (!file.good()) {
      return false;
    }

    while (file.good()) {

      int idx = -1;
      std::string extent_str;

      file >> idx >> extent_str;

      if (idx < 0 || extent_str.empty()) {
        break;
      }

      cutlass::gemm::GemmCoord extent;
      std::vector<std::string> tokens;

      cutlass::CommandLine::tokenize(tokens, extent_str, 'x');

      for (int i = 0; i < int(tokens.size()); ++i) {
        int x = std::atoi(tokens.at(i).c_str());

        // round up
        if (x % alignment) {
          x += (alignment - (x % alignment));
        }

        extent.at(i) = x;
      }

      if (extent.product()) {
        problem_sizes.push_back(extent);
      }
    }

    return true;
  }

  /// Post processes the problems
```
- **EN**: benchmark_problems() reads a text file of problem indices and MxNxK strings, tokenizes dimensions by x, rounds each dimension up to the requested alignment, and appends only valid nonzero extents. This lets external workloads drive the grouped benchmark.
- **CN**: benchmark_problems() 从文本文件读取问题索引和 MxNxK 字符串，用 x 切分维度，将每个维度向上补齐到指定对齐值，并只保留合法的非零规模。这样就能用外部工作负载驱动 grouped benchmark。

### Lines 388-409
```cpp
  void bin_problems() {

    problem_bins.clear();

    problem_count = int(problem_sizes.size());

    //
    // Insert the problem sizes into a sorted container class. This is *NOT* necessary
    // to run the CUTLASS kernel, but it enables the execution of cublas's batched GEMM.
    //
    for (int i = 0; i < int(problem_sizes.size()); ++i) {
      auto it = problem_bins.find(problem_sizes.at(i));
      if (it == problem_bins.end()) {
        problem_bins.insert({problem_sizes.at(i), std::vector<int32_t>({i}) });
      }
      else {
        it->second.push_back(i);
      }
    }
  }

  /// Prints the usage statement.
```
- **EN**: bin_problems() rebuilds the shape-to-index map after parsing. CUTLASS grouped GEMM itself does not need this binning, but the example keeps it so identical shapes can be recognized and compared with more conventional batched execution styles.
- **CN**: bin_problems() 在解析完成后重建“形状到索引列表”的映射。CUTLASS grouped GEMM 本身并不依赖这个分桶，但示例保留它，以便识别相同形状并与传统 batched 执行方式进行比较。

### Lines 410-469
```cpp
  std::ostream & print_usage(std::ostream &out) const {

    out << "64_ada_fp8_gemm_grouped\n\n"
      << "  This example profiles the performance of a 'grouped' GEMM kernel. This is similar to batched GEMM\n"
      << "  in that multiple, independent GEMMs are computed by one grid launch. It differs in that each\n"
      << "  'group' may compute a unique problem size. Problem sizes and pointers to matrices are both stored\n"
      << "  in device Global Memory and loaded by the kernel.\n\n"
      << "Options:\n\n"
      << "  --help                           If specified, displays this usage statement.\n\n"
      << "  --benchmark=<str>                Executes a benchmark problem size.\n"
      << "  --output_file=<str>              Path to a CSV file to output results. If it exists already, results are appended.\n"
      << "  --tag=<str>                      String tag to prepend to the CSV file.\n"
      << "  --groups=<int>                   Number of individual GEMM problems (default: --groups=15)\n"
      << "  --m=<int>                        Sets the M dimension for all groups. Otherwise, it is selected randomly\n"
      << "  --n=<int>                        Sets the N dimension for all groups. Otherwise, it is selected randomly\n"
      << "  --k=<int>                        Sets the K dimension for all groups. Otherwise, it is selected randomly\n"
      << "  --alpha=<f32>                    Epilogue scalar alpha (real part)\n"
      << "  --beta=<f32>                     Epilogue scalar beta (real part)\n"
      << "  --scheduler-modes=<str>          List of scheduler modes to be profile for grouped GEMM scheduler (default: --scheduler_modes=kDeviceOnly)\n"
      << "  --iterations=<int>               Number of profiling iterations to perform.\n"
      << "  --reference-check=<bool>         If true, performs reference check.\n"
      << "  --verbose=<bool>                 If true, prints problem sizes and batching structure.\n"
      << "  --profile-initialization=<bool>  If true, profiles the device-level kernel's initialization.\n"
      << "  --sort-problems=<bool>           If true, sorts problem sizes in descending order of GEMM-K dimension.\n";

    out << "\n\nExamples:\n\n"

      << "# Runs a grouped GEMM with 100 random problem sizes\n"
      << "$ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --groups=100\n\n"

      << "# Runs a grouped GEMM with 100 random problem sizes (with GEMM-K dimension equal to 1024)\n"
      << "$ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --groups=100 --k=1024 --verbose=true\n\n"

      << "# Runs a grouped GEMM that is equivalent to a batched GEMM\n"
      << "$ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --groups=100 --m=2048 --n=1024 --k=1024 --verbose=true\n\n"

      << "# Runs a grouped GEMM with each different scheduler mode\n"
      << "$ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --scheduler-modes=all\n\n"

      << "# Runs a grouped GEMM with each different scheduler mode and profiles host-side initialization time\n"
      << "$ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --scheduler-modes=all --profile-initialization=true\n\n"

      << "# Runs a grouped GEMM problem given an externally supplied benchmark file. This is a text file in which\n"
      << "# Each line contains a unique group index and an MxNxK triple indicating problemsize.\n"
      << "#\n"
      << "# For example, assume the following are the contents of 'problems.txt'\n"
      << "#\n"
      << "# 0 1024x256x520\n"
      << "# 1 520x264x1024\n"
      << "# 2 96x48x1024\n"
      << "#\n"
      << "$ ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --benchmark=problems.txt\n\n"

      << "# Execute Grouped GEMM and profile with NSight\n"
      << "$ nv-nsight-cu-cli ./examples/64_ada_fp8_gemm_grouped/64_ada_fp8_gemm_grouped --m=256 --n=256 --k=256 --verbose=true --iterations=1 --reference-check=false\n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
```
- **EN**: print_usage() emits the command-line help text and examples. It documents grouped-GEMM behavior, benchmark-file input, scheduler-mode enumeration, optional initialization profiling, and representative invocations for random, uniform, and Nsight-profiled runs.
- **CN**: print_usage() 输出命令行帮助与示例。它说明 grouped GEMM 的行为、benchmark 文件输入方式、scheduler 模式枚举、可选的初始化计时，以及随机、统一尺寸与 Nsight 分析等典型调用方法。

### Lines 470-485
```cpp
  double gflops(double runtime_s) const {

    // Number of real-valued multiply-adds
    int64_t fmas = int64_t();

    for (auto const & problem : problem_sizes) {
      fmas += problem.product();
    }

    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};

///////////////////////////////////////////////////////////////////////////////////////////////////

```
- **EN**: gflops() sums M*N*K over all problems, interprets each fused multiply-add as two floating-point operations, and converts the total work into GFLOP/s using the measured runtime. This treats the whole grouped launch as one aggregated workload.
- **CN**: gflops() 对所有问题的 M*N*K 求和，把每次乘加视为两次浮点运算，再结合测得的运行时间换算为 GFLOP/s。也就是说，它把整个 grouped launch 视为一个聚合工作负载来统计吞吐。

### Lines 486-558
```cpp
template <typename Gemm>
class BaseTestbed {
public:
  //
  // Type definitions
  //

  using ElementA = typename Gemm::ElementA;
  using ElementB = typename Gemm::ElementB;
  using ElementC = typename Gemm::ElementC;
  using ElementAccumulator = typename Gemm::ElementAccumulator;

  using EpilogueOutputOp = typename Gemm::GemmKernel::Epilogue::OutputOp;
  using ElementCompute = typename EpilogueOutputOp::ElementCompute;

  using LayoutA = typename Gemm::LayoutA;
  using LayoutB = typename Gemm::LayoutB;
  using LayoutC = typename Gemm::LayoutC;

  using MatrixCoord = typename LayoutC::TensorCoord;

  using DeviceGemmReference = cutlass::reference::device::Gemm<
    ElementA,
    LayoutA,
    ElementB,
    LayoutB,
    ElementC,
    LayoutC,
    ElementAccumulator,
    ElementAccumulator>;

  //
  // Data members
  //

  Options & options;

  /// Initialization
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
  cutlass::Distribution::Kind init_C;
  uint32_t seed;

  cutlass::DeviceAllocation<cutlass::gemm::GemmCoord> problem_sizes_device;

  std::vector<int64_t> offset_A;
  std::vector<int64_t> offset_B;
  std::vector<int64_t> offset_C;
  std::vector<int64_t> offset_D;

  std::vector<int64_t> lda_host;
  std::vector<int64_t> ldb_host;
  std::vector<int64_t> ldc_host;
  std::vector<int64_t> ldd_host;
  std::vector<ElementCompute *> alpha_ptr_array_host;

  cutlass::DeviceAllocation<int64_t> lda;
  cutlass::DeviceAllocation<int64_t> ldb;
  cutlass::DeviceAllocation<int64_t> ldc;
  cutlass::DeviceAllocation<int64_t> ldd;

  cutlass::DeviceAllocation<ElementA> block_A;
  cutlass::DeviceAllocation<ElementB> block_B;
  cutlass::DeviceAllocation<ElementC> block_C;
  cutlass::DeviceAllocation<ElementC> block_D;
  cutlass::DeviceAllocation<ElementCompute> alpha_array_device;

  cutlass::DeviceAllocation<ElementA *> ptr_A;
  cutlass::DeviceAllocation<ElementB *> ptr_B;
  cutlass::DeviceAllocation<ElementC *> ptr_C;
  cutlass::DeviceAllocation<ElementC *> ptr_D;
  cutlass::DeviceAllocation<ElementCompute *> alpha_ptr_array_device;

```
- **EN**: BaseTestbed starts by extracting types from the selected GEMM: operand/output/accumulator types, layouts, epilogue compute type, and a reference GEMM alias. It then declares all host metadata and device allocations needed to pack multiple differently sized matrices into contiguous buffers plus pointer arrays.
- **CN**: BaseTestbed 首先从所选 GEMM 类型中提取各种类型信息：输入/输出/累加器类型、布局、epilogue 计算类型以及参考 GEMM 别名。随后声明主机端元数据与设备端分配对象，用于把多个不同尺寸矩阵打包进连续缓冲区，并建立对应的指针数组。

### Lines 559-573
```cpp
  BaseTestbed(
    Options &options_,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint32_t seed_ = 3080
  ):
    options(options_), init_A(init_A_), init_B(init_B_), init_C(init_C_), seed(seed_) { }

  int problem_count() const {
    return options.problem_count;
  }

  /// Helper to initialize a tensor view
  template <typename Element>
```
- **EN**: The constructor forwards initialization settings, while problem_count() simply reflects the parsed number of grouped problems. This keeps all later setup and verification logic parameterized by the active Options instance.
- **CN**: 构造函数转发初始化设置，而 problem_count() 只是返回解析后的 grouped 问题数量。这样后续所有初始化与校验逻辑都可以统一依赖当前的 Options 实例。

### Lines 574-628
```cpp
  void initialize_tensor(
    Element *ptr,
    size_t capacity,
    cutlass::Distribution::Kind dist_kind,
    uint32_t seed) {

    if (dist_kind == cutlass::Distribution::Uniform) {

      Element scope_max, scope_min;
      int bits_input = cutlass::sizeof_bits<Element>::value;
      int bits_output = cutlass::sizeof_bits<typename Gemm::ElementC>::value;

      if (bits_input == 1) {
        scope_max = static_cast<Element>(2);
        scope_min = static_cast<Element>(0);
      } else if (bits_input <= 8) {
        scope_max = static_cast<Element>(2);
        scope_min = static_cast<Element>(-2);
      } else if (bits_output == 16) {
        if (cutlass::sizeof_bits<ElementAccumulator>::value <= 16) {
          scope_max = static_cast<Element>(5);
          scope_min = static_cast<Element>(-5);
        }
        else {
          scope_max = static_cast<Element>(8);
          scope_min = static_cast<Element>(-8);
        }
      } else {
        scope_max = static_cast<Element>(8);
        scope_min = static_cast<Element>(-8);
      }

      cutlass::reference::device::BlockFillRandomUniform(
        ptr, capacity, seed, scope_max, scope_min, 0);
    }
    else if (dist_kind == cutlass::Distribution::Gaussian) {

      cutlass::reference::device::BlockFillRandomGaussian(
        ptr, capacity, seed, Element(), Element(0.5f));
    }
    else if (dist_kind == cutlass::Distribution::Sequential) {

      // Fill with increasing elements
      cutlass::reference::device::BlockFillSequential(
        ptr, capacity, Element(1), Element());
    }
    else {

      // Fill with all 1s
      cutlass::reference::device::BlockFillSequential(
        ptr, capacity, Element(), Element(1));
    }
  }

  /// Allocates device-side data
```
- **EN**: initialize_tensor() chooses distribution ranges based on element bit width and output/accumulator precision, then calls CUTLASS device reference fillers. For FP8-sized inputs it keeps values in a narrow range to reduce overflow pressure and make validation practical.
- **CN**: initialize_tensor() 会根据元素位宽以及输出/累加精度选择分布范围，然后调用 CUTLASS 的设备端参考填充函数。对于 FP8 级别的输入，它使用较窄的数值范围，以降低溢出压力并让结果校验更稳定。

### Lines 629-680
```cpp
  void allocate() {
    int64_t total_elements_A = 0;
    int64_t total_elements_B = 0;
    int64_t total_elements_C = 0;
    int64_t total_elements_D = 0;

    lda_host.resize(problem_count());
    ldb_host.resize(problem_count());
    ldc_host.resize(problem_count());
    ldd_host.resize(problem_count());

    for (int32_t i = 0; i < problem_count(); ++i) {

      auto problem = options.problem_sizes.at(i);

      lda_host.at(i) = LayoutA::packed({problem.m(), problem.k()}).stride(0);
      ldb_host.at(i) = LayoutB::packed({problem.k(), problem.n()}).stride(0);
      ldc_host.at(i) = LayoutC::packed({problem.m(), problem.n()}).stride(0);
      ldd_host.at(i) = LayoutC::packed({problem.m(), problem.n()}).stride(0);

      offset_A.push_back(total_elements_A);
      offset_B.push_back(total_elements_B);
      offset_C.push_back(total_elements_C);
      offset_D.push_back(total_elements_D);

      int64_t elements_A = problem.m() * problem.k();
      int64_t elements_B = problem.k() * problem.n();
      int64_t elements_C = problem.m() * problem.n();
      int64_t elements_D = problem.m() * problem.n();

      total_elements_A += elements_A;
      total_elements_B += elements_B;
      total_elements_C += elements_C;
      total_elements_D += elements_D;
    }

    lda.reset(problem_count());
    ldb.reset(problem_count());
    ldc.reset(problem_count());
    ldd.reset(problem_count());

    block_A.reset(total_elements_A);
    block_B.reset(total_elements_B);
    block_C.reset(total_elements_C);
    block_D.reset(total_elements_D);

    alpha_ptr_array_host.resize(problem_count());
    alpha_array_device.reset(problem_count());
    alpha_ptr_array_device.reset(problem_count());
  }

  /// Initializes device-side data
```
- **EN**: allocate() computes per-problem packed leading dimensions, prefix-sum offsets into large contiguous A/B/C/D buffers, and total element counts. It then allocates device storage for matrices, leading-dimension arrays, and the per-group alpha pointer machinery.
- **CN**: allocate() 计算每个问题在紧凑布局下的 leading dimension、A/B/C/D 大缓冲区中的前缀和偏移量，以及总元素数。随后为矩阵数据、leading dimension 数组和按 group 的 alpha 指针机制分配设备存储。

### Lines 681-737
```cpp
  void initialize() {
    problem_sizes_device.reset(problem_count());
    problem_sizes_device.copy_from_host(options.problem_sizes.data());

    lda.copy_from_host(lda_host.data());
    ldb.copy_from_host(ldb_host.data());
    ldc.copy_from_host(ldc_host.data());
    ldd.copy_from_host(ldd_host.data());

    //
    // Assign pointers
    //

    std::vector<ElementA *> ptr_A_host(problem_count());
    std::vector<ElementB *> ptr_B_host(problem_count());
    std::vector<ElementC *> ptr_C_host(problem_count());
    std::vector<ElementC *> ptr_D_host(problem_count());

    for (int32_t i = 0; i < problem_count(); ++i) {
      ptr_A_host.at(i) = block_A.get() + offset_A.at(i);
      ptr_B_host.at(i) = block_B.get() + offset_B.at(i);
      ptr_C_host.at(i) = block_C.get() + offset_C.at(i);
      ptr_D_host.at(i) = block_D.get() + offset_D.at(i);
    }

    ptr_A.reset(problem_count());
    ptr_A.copy_from_host(ptr_A_host.data());

    ptr_B.reset(problem_count());
    ptr_B.copy_from_host(ptr_B_host.data());

    ptr_C.reset(problem_count());
    ptr_C.copy_from_host(ptr_C_host.data());

    ptr_D.reset(problem_count());
    ptr_D.copy_from_host(ptr_D_host.data());

    //
    // Initialize the problems of the workspace
    //

    initialize_tensor(block_A.get(), block_A.size(), init_A, seed * 2021);
    initialize_tensor(block_B.get(), block_B.size(), init_B, seed * 2022);
    initialize_tensor(block_C.get(), block_C.size(), init_C, seed * 2023);

    cutlass::reference::device::BlockFillSequential(
      block_D.get(), block_D.size(), ElementC(), ElementC());

    // Initialize alpha array
    alpha_array_device.copy_from_host(options.alpha_array.data());
    for (int32_t i = 0; i < problem_count(); ++i) {
      alpha_ptr_array_host.at(i) = alpha_array_device.get() + i;
    }
    alpha_ptr_array_device.copy_from_host(alpha_ptr_array_host.data());
  }

  /// Verifies the result is a GEMM
```
- **EN**: initialize() copies problem sizes and strides to the device, builds host-side pointer arrays that reference each submatrix inside the packed buffers, randomizes A/B/C, zeros D, and uploads the per-group alpha values plus the device pointer array that points to them.
- **CN**: initialize() 会把问题尺寸与 stride 拷到设备端，在主机端构建指向各子矩阵的指针数组，把这些子矩阵映射到打包缓冲区中；然后随机初始化 A/B/C，将 D 清零，并上传按 group 的 alpha 值以及指向这些值的设备指针数组。

### Lines 738-803
```cpp
  bool verify() {

    bool passed = true;

    for (int32_t i = 0; i < problem_count(); ++i) {
      cutlass::gemm::GemmCoord problem = options.problem_sizes.at(i);

      LayoutA layout_A(lda_host.at(i));
      LayoutB layout_B(ldb_host.at(i));
      LayoutC layout_C(ldc_host.at(i));
      LayoutC layout_D(ldd_host.at(i));

      MatrixCoord extent_A{problem.m(), problem.k()};
      MatrixCoord extent_B{problem.k(), problem.n()};
      MatrixCoord extent_C{problem.m(), problem.n()};

      cutlass::TensorView<ElementA, LayoutA> view_A(block_A.get() + offset_A.at(i), layout_A, extent_A);
      cutlass::TensorView<ElementB, LayoutB> view_B(block_B.get() + offset_B.at(i), layout_B, extent_B);
      cutlass::TensorView<ElementC, LayoutC> view_C(block_C.get() + offset_C.at(i), layout_C, extent_C);

      cutlass::DeviceAllocation<ElementC>    block_Ref(layout_D.capacity(extent_C));
      cutlass::TensorView<ElementC, LayoutC> view_Ref_device(block_Ref.get(), layout_D, extent_C);

      // Reference GEMM
      cutlass::reference::device::GemmComplex<
          ElementA, LayoutA,
          ElementB, LayoutB,
          ElementC, LayoutC,
          ElementCompute, ElementAccumulator
      >(
        problem,
        options.alpha_array[i],
        view_A,
        Gemm::kTransformA,
        view_B,
        Gemm::kTransformB,
        options.beta,
        view_C,
        view_Ref_device,
        ElementAccumulator(0)
      );

      // Copy to host memory
      std::vector<ElementC> matrix_D(layout_D.capacity(extent_C));
      std::vector<ElementC> matrix_Ref(layout_D.capacity(extent_C));

      cutlass::device_memory::copy_to_host(matrix_D.data(),   block_D.get() + offset_D.at(i), matrix_D.size());
      cutlass::device_memory::copy_to_host(matrix_Ref.data(), block_Ref.get(),                matrix_D.size());

      cutlass::TensorView<ElementC, LayoutC> view_D(  matrix_D.data(),   layout_D, extent_C);
      cutlass::TensorView<ElementC, LayoutC> view_Ref(matrix_Ref.data(), layout_D, extent_C);

      // Reference check
      passed = cutlass::reference::host::TensorEquals(view_D, view_Ref);

      if (!passed) {
        std::cerr << "\n***\nError - problem " << i << " failed the QA check\n***\n" << std::endl;
        return passed;
      }
    }

    return passed;
  }

};

```
- **EN**: verify() checks each grouped GEMM independently. It builds tensor views for the packed submatrices, runs a device-side reference GEMMComplex with that problem’s alpha and beta, copies both reference and produced D back to host memory, and compares them elementwise.
- **CN**: verify() 会逐个独立校验每个 grouped GEMM。它为打包后的子矩阵建立 tensor view，使用该问题自己的 alpha 与 beta 调用设备端参考实现 GemmComplex，把参考结果和实际输出 D 都拷回主机，再逐元素比较。

### Lines 804-840
```cpp
template <typename Gemm_, cutlass::gemm::kernel::GroupScheduleMode GroupScheduleMode_>
class TestbedGrouped : BaseTestbed<Gemm_> {
public:
  TestbedGrouped(
    Options &options_,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint32_t seed_ = 3080
  ): BaseTestbed<Gemm_>(options_, init_A_, init_B_, init_C_, seed_) {}

  // Redefine GEMM with different GroupScheduleMode_
  using GemmKernel = typename cutlass::gemm::kernel::DefaultGemmGroupedPerGroupScale<
    typename Gemm_::ElementA,
    typename Gemm_::LayoutA,
    Gemm_::kTransformA,
    Gemm_::kAlignmentA,
    typename Gemm_::ElementB,
    typename Gemm_::LayoutB,
    Gemm_::kTransformB,
    Gemm_::kAlignmentB,
    typename Gemm_::ElementC,
    typename Gemm_::LayoutC,
    typename Gemm_::ElementAccumulator,
    typename Gemm_::OperatorClass,
    typename Gemm_::ArchTag,
    typename Gemm_::ThreadblockShape,
    typename Gemm_::WarpShape,
    typename Gemm_::InstructionShape,
    typename Gemm_::EpilogueOutputOp,
    typename Gemm_::ThreadblockSwizzle,
    Gemm_::kStages,
    GroupScheduleMode_>::GemmKernel;

  using Gemm = cutlass::gemm::device::GemmGrouped<GemmKernel>;

  /// Verbose printing of problem sizes
```
- **EN**: TestbedGrouped specializes BaseTestbed for one scheduler mode. The key alias is DefaultGemmGroupedPerGroupScale<...>::GemmKernel, which asks CUTLASS to compose a default GEMM mainloop/epilogue with a grouped wrapper that supports per-group scaling. The lower-level dispatch policy is intentionally inferred by CUTLASS from these template arguments rather than named explicitly here.
- **CN**: TestbedGrouped 针对某一个调度模式特化 BaseTestbed。核心别名是 DefaultGemmGroupedPerGroupScale<...>::GemmKernel，它要求 CUTLASS 把默认 GEMM 的主循环/epilogue 与支持按 group 缩放的 grouped 封装组合起来。更底层的 dispatch policy 并未在此处显式写出，而是由 CUTLASS 根据这些模板参数自动推导。

### Lines 841-863
```cpp
  void print_problem_sizes() {
    std::cout << std::endl;

    // Print groups
    std::cout << this->problem_count() << " groups:\n";

    int32_t idx = 0;
    int64_t total_tiles = 0;

    for (auto const & problem : this->options.problem_sizes) {
      int tiles = Gemm::problem_tile_count(problem);
      total_tiles += tiles;

      std::cout << "  [" << idx << "]: "
        << problem.m() << "-by-" << problem.n() << "-by-" << problem.k()
        << " (" << tiles << " threadblock tiles)" << "\n";

      ++idx;
    }
    std::cout << std::endl;
  }

  /// Sort problems in descending order of problem-K dimension
```
- **EN**: print_problem_sizes() prints every group’s M, N, K and the number of threadblock tiles CUTLASS estimates for it. This is useful for understanding how irregular problems expand into grouped scheduling work.
- **CN**: print_problem_sizes() 会打印每个 group 的 M、N、K，以及 CUTLASS 估计得到的 threadblock tile 数量。这有助于理解不规则问题是如何展开成 grouped 调度工作量的。

### Lines 864-877
```cpp
  void sort_problems() {
    Gemm::sort_problems(this->options.problem_count,
                        this->options.problem_sizes.data(),
                        this->lda_host.data(),
                        this->ldb_host.data(),
                        this->ldc_host.data(),
                        this->ldd_host.data(),
                        this->offset_A.data(),
                        this->offset_B.data(),
                        this->offset_C.data(),
                        this->offset_D.data());
  }

  /// Executes a grouped kernel and measures runtime
```
- **EN**: sort_problems() calls CUTLASS-provided reordering logic to sort problems by descending K while keeping sizes, strides, and packed-buffer offsets consistent. This can improve scheduler behavior for some grouped workloads.
- **CN**: sort_problems() 调用 CUTLASS 提供的重排逻辑，按 K 维降序排序问题，同时保持尺寸、stride 与打包缓冲区偏移的一致性。对某些 grouped 工作负载，这样做有助于改善调度行为。

### Lines 878-944
```cpp
  Result profile() {
    std::string sched_mode = this->options.scheduler_mode_to_str.find(GroupScheduleMode_)->second;

    std::cout << std::endl;
    std::cout << "Grouped GEMM (CUTLASS) with mode " << sched_mode << ":\n"
      << "====================================================" << std::endl;

    Result result;

    int threadblock_count = Gemm::sufficient(this->options.problem_sizes.data(), this->options.problem_count);

    // Early exit
    if (!threadblock_count) {
      std::cout << "Active CUDA device lacks hardware resources to run CUTLASS Grouped GEMM kernel." << std::endl;
      return result;
    }

    result.passed = false;

    // Initialize the problem
    this->allocate();
    if (this->options.sort_problems) {
      sort_problems();
    }
    this->initialize();

    if (this->options.verbose) {
      print_problem_sizes();
    }

    // Configure the GEMM arguments
    typename Gemm::EpilogueOutputOp::ElementCompute ** alpha_ptr_array = this->alpha_ptr_array_device.get();
    typename Gemm::EpilogueOutputOp::Params epilogue_op(alpha_ptr_array, nullptr);

    // Configure GEMM arguments
    typename Gemm::Arguments args(
      this->problem_sizes_device.get(),
      this->problem_count(),
      threadblock_count,
      epilogue_op,
      this->ptr_A.get(),
      this->ptr_B.get(),
      this->ptr_C.get(),
      this->ptr_D.get(),
      this->lda.get(),
      this->ldb.get(),
      this->ldc.get(),
      this->ldd.get(),
      this->options.problem_sizes.data()
    );

    // Initialize the GEMM object
    Gemm gemm;

    size_t workspace_size = gemm.get_workspace_size(args);
    cutlass::DeviceAllocation<uint8_t> workspace(workspace_size);

    result.status = gemm.initialize(args, workspace.get());

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize CUTLASS Grouped GEMM kernel." << std::endl;
      return result;
    }

    // Run the grouped GEMM object
    result.status = gemm.run();

```
- **EN**: The first half of profile() prints the selected scheduling mode, asks CUTLASS how many threadblocks are sufficient, allocates and initializes data, optionally sorts and prints problems, then builds the epilogue and grouped-kernel arguments. The epilogue is configured with alpha_ptr_array and a null beta pointer array, so kernel-side scaling is per-group alpha only and beta falls back to zero in the output operator.
- **CN**: profile() 的前半部分会打印所选调度模式，向 CUTLASS 查询所需 threadblock 数，分配并初始化数据，按需排序并打印问题，然后构造 epilogue 和 grouped kernel 参数。这里的 epilogue 只传入 alpha_ptr_array，并把 beta 指针数组设为 null，因此内核侧只做按 group 的 alpha 缩放，而 beta 会在输出算子中回退为 0。

### Lines 945-1040
```cpp
    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to run CUTLASS Grouped GEMM kernel." << std::endl;
      return result;
    }

    // Wait for completion
    result.error = cudaDeviceSynchronize();

    if (result.error != cudaSuccess)  {
      std::cerr << "Kernel execution error: " << cudaGetErrorString(result.error);
      return result;
    }

    //
    // Verify correctness
    //
    result.passed = true;

    if (this->options.reference_check) {
      result.passed = this->verify();
    }

    //
    // Warm-up run of the grouped GEMM object
    //
    result.status = gemm.run();

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to run CUTLASS Grouped GEMM kernel." << std::endl;
      return result;
    }

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

    // Record an event at the start of a series of GEMM operations
    result.error = cudaEventRecord(events[0]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    //
    // Run profiling loop
    //

    for (int iter = 0; iter < this->options.iterations; ++iter) {
      gemm();
    }

    //
    // Stop profiling loop
    //

    // Record an event when the GEMM operations have been launched.
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
    result.runtime_ms = double(runtime_ms) / double(this->options.iterations);
    result.gflops = this->options.gflops(result.runtime_ms / 1000.0);

    //
    // Cleanup
    //

    for (auto event : events) {
```
- **EN**: The middle of profile() executes one checked run, synchronizes, optionally verifies correctness, performs a warm-up, creates CUDA events, launches the grouped GEMM repeatedly via gemm(), and measures the average runtime. This section is the core performance path for the example.
- **CN**: profile() 的中段会先执行一次带检查的运行并同步，按需做正确性验证，然后预热、创建 CUDA event，通过 gemm() 重复发射 grouped GEMM，并测量平均运行时间。这一段是示例的核心性能测试路径。

### Lines 1041-1082
```cpp
      (void)cudaEventDestroy(event);
    }

    // Optionally profile initialization
    if (this->options.profile_initialization) {
      // Warm up
      gemm.initialize(args, workspace.get());

      auto start_time = std::chrono::high_resolution_clock::now();
      for (int32_t i = 0; i < this->options.iterations; ++i) {
        gemm.initialize(args, workspace.get());
      }
      auto end_time = std::chrono::high_resolution_clock::now();

      std::chrono::duration<double, std::milli> duration = end_time - start_time;
      duration /= double(this->options.iterations);
      result.initialization_time_ms = duration.count();
    }

    int64_t total_tiles = Gemm::group_tile_count(args);
    std::cout << "    " << total_tiles << " total threadblock tiles." << std::endl;

    std::cout << std::endl;
    std::cout << "    " << "Grouped Runtime: " << result.runtime_ms << " ms" << std::endl;
    std::cout << "    " << "Grouped  GFLOPs: " << result.gflops << std::endl;
    if (this->options.profile_initialization) {
      std::cout << "    " << "Init    Runtime: " << result.initialization_time_ms << " ms" << std::endl;
    }

    if (this->options.output_file.good()) {
      this->options.output_file << this->options.output_tag << ",CUTLASS,grouped-" << sched_mode << ","
        << this->options.problem_count << "," << result.runtime_ms << "," << result.gflops << std::endl;
    }

    std::cout << "\nPassed\n";

    return result;
  }
};

///////////////////////////////////////////////////////////////////////////////////////////////////

```
- **EN**: The final part of profile() optionally measures host-side initialize() overhead, queries the total grouped tile count, prints runtime statistics, appends a CSV row when requested, and reports success. This separates launch-time setup cost from steady-state kernel throughput.
- **CN**: profile() 的最后部分会按需测量主机侧 initialize() 开销，查询 grouped 总 tile 数，打印运行统计信息，并在需要时追加一行 CSV，最后报告成功。这样可以把启动阶段的准备成本与稳态内核吞吐分开观察。

### Lines 1083-1132
```cpp
int main(int argc, char const **args) {

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (__CUDACC_VER_MAJOR__ < 12 || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ < 4)) {
    std::cerr << "This example requires CUDA 12.4 or greater." << std::endl;
    return 0;
  }
  int device_idx;
  cudaError_t result = cudaGetDevice(&device_idx);
  cudaDeviceProp properties;
    result = cudaGetDeviceProperties(&properties, device_idx);

  if (result != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() failed with error: " << cudaGetErrorString(result) << std::endl;
    return 0;
  }

  if (!(properties.major == 8 && properties.minor == 9)) {
    std::cerr << "CUTLASS's Ada FP8 Gemm Grouped example requires a device of compute capability 89.\n" << std::endl;
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

  if (options.error) {
    std::cerr << "Aborting execution." << std::endl;
    return -1;
  }

  //
  // Define the Grouped and Batched GEMM types
  //

```
- **EN**: main() begins with runtime environment checks: it queries device properties, requires CUDA 12.4+, and enforces compute capability 8.9 because this example targets Ada FP8 Tensor Core support specifically. It then parses options and exits early for help or invalid input.
- **CN**: main() 先做运行环境检查：查询设备属性、要求 CUDA 12.4 及以上，并强制要求计算能力 8.9，因为该示例专门面向 Ada 的 FP8 Tensor Core 支持。随后解析参数，并在请求帮助或输入无效时提前退出。

### Lines 1133-1178
```cpp
  using ElementA = cutlass::float_e4m3_t;
  using ElementB = cutlass::float_e4m3_t;
  using ElementOutput = cutlass::bfloat16_t;
  using ElementAccumulator = float;

  using LayoutA = cutlass::layout::RowMajor;
  using LayoutB = cutlass::layout::ColumnMajor;
  using LayoutC = cutlass::layout::RowMajor;

  constexpr int ElementsPerAccessA = 128 / cutlass::sizeof_bits<ElementA>::value;
  constexpr int ElementsPerAccessB = 128 / cutlass::sizeof_bits<ElementB>::value;

  // Define a grouped GEMM kernel with all template parameters set except
  // for scheduling mode. This will be used as the template for all scheduling
  // modes executed.
  using GemmKernel = typename cutlass::gemm::kernel::DefaultGemmGroupedPerGroupScale<
    ElementA,
    LayoutA,
    cutlass::ComplexTransform::kNone,
    ElementsPerAccessA,
    ElementB,
    LayoutB,
    cutlass::ComplexTransform::kNone,
    ElementsPerAccessB,
    ElementOutput, LayoutC,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm89,
    cutlass::gemm::GemmShape<64, 128, 64>,
    cutlass::gemm::GemmShape<64, 32, 64>,
    cutlass::gemm::GemmShape<16, 8, 32>,
    cutlass::epilogue::thread::LinearCombination<
        ElementOutput, 128 / cutlass::sizeof_bits<ElementOutput>::value,
        ElementAccumulator, ElementAccumulator>,
    // NOTE: Threadblock swizzling is currently not supported by CUTLASS's grouped kernels.
    // This parameter is passed in at present to match the APIs of other kernels. The parameter
    // is unused within the kernel.
    cutlass::gemm::threadblock::GemmBatchedIdentityThreadblockSwizzle,
    4>::GemmKernel;

  using GemmGrouped = cutlass::gemm::device::GemmGrouped<GemmKernel>;

  //
  // Profile it
  //

```
- **EN**: This block defines the concrete GEMM types. A and B use FP8 e4m3, the output uses BF16, accumulation is in FP32, A is row-major, B is column-major, and C/D are row-major. The chosen alignments imply 128-bit vector accesses (16 FP8 values per access), while the kernel targets OpClassTensorOp on Sm89 with threadblock/warp/instruction shapes 64x128x64, 64x32x64, and 16x8x32, a 4-stage pipeline, and a LinearCombination epilogue that writes BF16 vectors.
- **CN**: 这一段定义了具体 GEMM 类型：A 与 B 使用 FP8 e4m3，输出使用 BF16，累加使用 FP32；A 为行主序，B 为列主序，C/D 为行主序。所选对齐意味着使用 128 位向量访问（每次访问 16 个 FP8 元素），内核目标是 Sm89 上的 TensorOp，并采用 64x128x64、64x32x64、16x8x32 的 threadblock/warp/instruction 形状、4 级流水，以及把 BF16 向量写回的 LinearCombination epilogue。

### Lines 1179-1208
```cpp
  using GroupScheduleMode = cutlass::gemm::kernel::GroupScheduleMode;
  for (GroupScheduleMode mode : options.scheduler_modes) {
    Result result;
    switch (mode) {
      case GroupScheduleMode::kDeviceOnly:
        {
          TestbedGrouped<GemmGrouped, GroupScheduleMode::kDeviceOnly> runner(options);
          result = runner.profile();
          break;
        }
      case GroupScheduleMode::kHostPrecompute:
        {
          TestbedGrouped<GemmGrouped, GroupScheduleMode::kHostPrecompute> runner(options);
          result = runner.profile();
          break;
        }
    }

    if (result.error != cudaSuccess) {
      return 1;
    }

    // Override verbose flag to avoid printing duplicate information for each scheduling mode
    options.verbose = false;
  }

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN**: The closing loop runs the benchmark once for each requested GroupScheduleMode, instantiating TestbedGrouped with either kDeviceOnly or kHostPrecompute. After each run it checks CUDA status and suppresses duplicate verbose printing for subsequent modes, then returns success.
- **CN**: 结尾的循环会针对每个请求的 GroupScheduleMode 运行一次 benchmark，分别用 kDeviceOnly 或 kHostPrecompute 实例化 TestbedGrouped。每次运行后都会检查 CUDA 状态，并关闭后续模式的重复 verbose 输出，最后返回成功。

---

## Key Concepts / 关键概念

- **Grouped GEMM / Grouped GEMM**: One kernel launch processes many GEMMs whose M/N/K sizes may differ; the kernel pulls per-problem pointers, strides, and sizes from device memory. / 单次内核启动处理多个 GEMM，且各自的 M/N/K 可以不同；内核会从设备内存读取每个问题的指针、stride 和尺寸。
- **Per-group scaling / 按 group 缩放**: The epilogue uses an array of alpha pointers so each GEMM can apply a different output scale. In this file, beta is not wired into the kernel arguments and effectively remains zero there. / epilogue 使用 alpha 指针数组，使每个 GEMM 都能应用不同的输出缩放。在本文件中，beta 没有接入内核参数，因此在内核侧实际上保持为 0。
- **CUTLASS kernel composition / CUTLASS 内核组合**: `DefaultGemmGroupedPerGroupScale` builds a grouped kernel by combining CUTLASS’s default GEMM mainloop and epilogue with `GemmGroupedPerGroupScale`. / `DefaultGemmGroupedPerGroupScale` 通过把 CUTLASS 默认 GEMM 主循环与 epilogue 同 `GemmGroupedPerGroupScale` 组合起来，构建 grouped 内核。
- **Dispatch policy / Dispatch policy**: The file chooses high-level template knobs (`OpClassTensorOp`, `Sm89`, layouts, alignments, tile shapes, stages), while the detailed low-level dispatch policy is inferred inside CUTLASS `DefaultGemm`/`DefaultGemmConfiguration`. / 文件只给出高层模板旋钮（`OpClassTensorOp`、`Sm89`、布局、对齐、tile 形状、流水级数），更底层的 dispatch policy 则由 CUTLASS 的 `DefaultGemm`/`DefaultGemmConfiguration` 在内部推导。
- **Ada FP8 path / Ada FP8 路径**: `Sm89` plus FP8 `float_e4m3_t` inputs target Ada Tensor Cores. Accumulation stays in FP32 and the result is written as BF16, balancing performance and numerical stability. / `Sm89` 配合 FP8 `float_e4m3_t` 输入，面向 Ada Tensor Core。累加保持 FP32，结果以 BF16 写回，在性能与数值稳定性之间做平衡。
- **Scheduling modes / 调度模式**: `kDeviceOnly` computes tile-to-problem assignment on device, while `kHostPrecompute` prepares schedule data during initialization. The example can benchmark both. / `kDeviceOnly` 在设备端计算 tile 到问题的分配，`kHostPrecompute` 则在初始化阶段预先准备调度数据。该示例可以对两者分别做 benchmark。
- **Data movement / 数据移动**: Host vectors describe problem metadata and packed offsets; device allocations hold contiguous matrix storage and pointer arrays; verification copies each result tile back to host for comparison. / 主机端向量描述问题元数据与打包偏移；设备分配保存连续矩阵存储与指针数组；验证阶段再把每个结果块拷回主机进行比较。

## Dependencies / 依赖项

- **CUDA runtime**: `cudaGetDeviceProperties`, `cudaDeviceSynchronize`, CUDA events, and device memory operations. / CUDA 运行时：设备属性查询、同步、CUDA event 与设备内存操作。
- **CUTLASS GEMM stack**: `cutlass/gemm/kernel/default_gemm_grouped_per_group_scale.h`, `cutlass/gemm/kernel/gemm_grouped_per_group_scale.h`, `cutlass/gemm/device/gemm_grouped.h`, and supporting GEMM/layout/type headers. / CUTLASS GEMM 栈：`cutlass/gemm/kernel/default_gemm_grouped_per_group_scale.h`、`cutlass/gemm/kernel/gemm_grouped_per_group_scale.h`、`cutlass/gemm/device/gemm_grouped.h` 以及相关 GEMM/布局/类型头文件。
- **Epilogue operator**: `cutlass::epilogue::thread::LinearCombination` supplies per-group alpha pointer-array loading and vectorized BF16 output. / Epilogue 算子：`cutlass::epilogue::thread::LinearCombination` 提供按 group 的 alpha 指针数组加载与 BF16 向量化输出。
- **Utility helpers**: CUTLASS command-line parsing, random/distribution fillers, `DeviceAllocation`, tensor views, and host/device reference GEMM utilities. / 工具辅助：CUTLASS 的命令行解析、随机/分布填充、`DeviceAllocation`、tensor view，以及主机/设备参考 GEMM 工具。
