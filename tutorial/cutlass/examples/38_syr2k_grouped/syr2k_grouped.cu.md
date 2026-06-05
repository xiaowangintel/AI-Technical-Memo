# syr2k_grouped.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/38_syr2k_grouped/syr2k_grouped.cu`
**Purpose / 用途**: Implements grouped SYR2K benchmarking and correctness checks for heterogeneous problem sizes in CUTLASS. / 实现 CUTLASS 中异构问题规模的分组 SYR2K 基准测试与正确性校验。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1–30 — License Header / 许可证头

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
```
**EN**: Standard NVIDIA BSD-3-Clause license block. All CUTLASS examples carry this header; it imposes no restrictions on reading or using the code for research or production purposes.

**CN**: 标准 NVIDIA BSD-3-Clause 许可证头。所有 CUTLASS 示例均包含此头；它不限制代码用于研究或生产目的。

---

### Lines 31–54 — File Docstring / 文件说明注释

```cpp

/*! \file
    \brief SYR2K Grouped Example.

    This workload computes a batch of SYR2K operations with distinct problem sizes. This example closely
    follows 24_gemm_grouped.

    Examples:

      # Runs a grouped SYR2K with 100 random problem sizes
      $ ./examples/38_syr2k_grouped/38_syr2k_grouped --groups=100

      # Runs a grouped SYR2K with 100 random problem sizes (with SYR2K-K dimension equal to 1024)
      $ ./examples/38_syr2k_grouped/24_gemm_grouped --groups=100 --k=1024 --verbose=true

      # Runs a grouped SYR2K that is equivalent to a batched SYR2K
      $ ./examples/38_syr2k_grouped/38_syr2k_grouped --groups=100 --n=1024 --k=1024 --verbose=true

      # Execute grouped SYR2K and profile with NSight
      $ nv-nsight-cu-cli ./examples/38_syr2k_grouped/38_syr2k_grouped --n=256 --k=256 --verbose=true \
                                                                    --iterations=1 --reference-check=false

*/
```
**EN**: Doxygen `\file` block that describes the high-level purpose and provides ready-to-use command-line invocations. SYR2K (Symmetric Rank-2K update) is a BLAS Level-3 routine; the "grouped" variant amortises per-kernel launch overhead by batching many differently-sized SYR2K problems into one GPU kernel call. The examples also show how to use NSight for profiling.

**CN**: Doxygen `\file` 块，描述示例的高层目的，并提供可直接运行的命令行示例。SYR2K（对称秩-2K 更新）是 BLAS Level-3 例程；"grouped" 变体通过将多个不同规模的 SYR2K 问题打包进一次 GPU 内核调用来摊销每次启动的开销。示例还展示了如何结合 NSight 进行性能分析。

---

### Lines 55–87 — Includes / 头文件包含

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

#include <chrono>
#include <iostream>
#include <fstream>
#include <sstream>
#include <unordered_map>
#include <vector>

#include "cutlass/blas3.h"
#include "cutlass/cutlass.h"
#include "cutlass/device_kernel.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/gemm/kernel/rank_2k_grouped.h"
#include "cutlass/gemm/kernel/default_rank_2k_grouped.h"
#include "cutlass/gemm/device/rank_2k_grouped.h"
#include "cutlass/gemm/device/rank_2k.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/host/rank_2k_complex.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_norm.h"

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Key CUTLASS headers:
- `cutlass/blas3.h` — BLAS-3 enumerations such as `FillMode` (upper/lower triangular) and `BlasMode` (Symmetric/Hermitian).
- `cutlass/gemm/kernel/rank_2k_grouped.h` / `default_rank_2k_grouped.h` — the grouped kernel policy and default parameter selector.
- `cutlass/gemm/device/rank_2k_grouped.h` / `rank_2k.h` — the device-level operator wrappers for grouped and single-problem Rank-2K.
- Reference utilities (`rank_2k_complex.h`, `tensor_fill.h`) provide CPU golden-reference implementations and data initialisation helpers.

**CN**: 关键 CUTLASS 头文件：
- `cutlass/blas3.h` — BLAS-3 枚举，如 `FillMode`（上/下三角）和 `BlasMode`（对称/Hermitian）。
- `cutlass/gemm/kernel/rank_2k_grouped.h` / `default_rank_2k_grouped.h` — 分组内核策略和默认参数选择器。
- `cutlass/gemm/device/rank_2k_grouped.h` / `rank_2k.h` — 分组与单问题 Rank-2K 的设备级算子封装。
- 参考工具（`rank_2k_complex.h`、`tensor_fill.h`）提供 CPU 黄金参考实现和数据初始化辅助函数。

---

### Lines 88–114 — `Result` Struct / 结果结构体

```cpp
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
**EN**: Aggregates per-run metrics. `initialization_time_ms` is unique to this example and captures the time taken by `rank2k.initialize()` separately from the kernel execution time — useful when profiling the host-precompute scheduling overhead (`kHostPrecompute` mode builds tile→problem mapping on the CPU). `cutlass::Status` is an enum covering `kSuccess`, various error codes etc.

**CN**: 聚合每次运行的性能指标。`initialization_time_ms` 是本示例特有的字段，单独记录 `rank2k.initialize()` 的耗时（与内核执行时间分开）——这在分析 `kHostPrecompute` 调度模式下 CPU 预计算 tile→problem 映射的开销时非常有用。`cutlass::Status` 是枚举类型，包含 `kSuccess` 及各种错误码。

---

### Lines 115–180 — `Options` Struct: Fields & Scheduler Modes / 选项结构体：字段与调度模式

```cpp
// Command line options parsing
struct Options {

  bool help;
  bool error;
  bool reference_check;
  bool profile_initialization;
  bool sort_problems;

  std::vector<cutlass::gemm::GemmCoord> problem_sizes;

  int alignment;
  int problem_count;
  int iterations;
  int cuda_streams;
  bool verbose;
  float alpha;
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

  Options():
    help(false),
    error(false),
    alignment(8),
    reference_check(true),
    profile_initialization(false),
    sort_problems(false),
    problem_count(5),
    iterations(20),
    cuda_streams(0),
    verbose(false),
    alpha(1),
    beta(),
    scheduler_modes({GroupScheduleMode::kDeviceOnly})
  { }
```
**EN**: `problem_sizes` stores a `GemmCoord` (m, n, k) for each group. For SYR2K only N and K are meaningful; M is set equal to N, so the output matrix is always square (N×N). `GroupScheduleMode` is the central parameter that selects between:
- `kDeviceOnly` — tile→problem assignment happens entirely on the GPU; no host pre-computation needed, very low initialization cost.
- `kHostPrecompute` — the CPU iterates over all tiles and writes a problem-index table into a device buffer before the kernel launch; this removes per-tile scheduling overhead inside the kernel at the cost of potentially slower initialization for large group counts.

**CN**: `problem_sizes` 为每个分组存储一个 `GemmCoord`（m, n, k）。对于 SYR2K，只有 N 和 K 有意义；M 被设置为等于 N，因此输出矩阵始终为方阵（N×N）。`GroupScheduleMode` 是核心参数，用于选择：
- `kDeviceOnly` — tile→problem 分配完全在 GPU 端进行，无需主机预计算，初始化成本极低。
- `kHostPrecompute` — CPU 遍历所有 tile 并在内核启动前将问题索引表写入设备缓冲区；这消除了内核内部的逐 tile 调度开销，代价是对于大分组数时初始化可能较慢。

---

### Lines 181–304 — `Options::parse()` and `randomize_problems()` / 命令行解析与随机问题生成

```cpp
  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("alignment", alignment, 8);
    cmd.get_cmd_line_argument("groups", problem_count, 5);
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
  }

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

    // SYR2K is defined via only N and K.
    if (cmd_line_m != -1) {
      std::cerr << "Parameter M is ignored for SYR2K\n";
      error = true;
      return;
    }

    problem_sizes.reserve(problem_count);

    for (int i = 0; i < problem_count; ++i) {
      int n = cmd_line_n;
      int k = cmd_line_k;

      if (n < 1) {
        n = alignment * ((rand() % 256) + 1);
      }

      if (k < 1) {
        k = alignment * ((rand() % 256) + 1);
      }

      // SYR2K is defined only in terms of N and K. Replicate N into
      // the SYR2K-N dimension.
      cutlass::gemm::GemmCoord problem(n, n, k);

      problem_sizes.push_back(problem);
    }
  }
```
**EN**: `cutlass::CommandLine` provides a thin key-value argument parser. `alignment` (default 8) ensures all N and K dimensions are multiples of 8 — necessary to satisfy CUTLASS's memory-alignment requirement for vectorised loads. Random problem sizes are generated in the range [alignment, 256·alignment]. The SYR2K constraint M=N is enforced by constructing `GemmCoord(n, n, k)`, not `(m, n, k)`.

**CN**: `cutlass::CommandLine` 提供了一个轻量的键值参数解析器。`alignment`（默认值 8）确保所有 N 和 K 维度均为 8 的倍数——这是满足 CUTLASS 向量化访问内存对齐要求的必要条件。随机问题规模在 [alignment, 256·alignment] 范围内生成。SYR2K 约束 M=N 通过构造 `GemmCoord(n, n, k)` 而非 `(m, n, k)` 来强制执行。

---

### Lines 305–425 — `benchmark_problems()`, `print_usage()`, and `gflops()` / 基准问题加载、使用说明与 GFLOP 计算

```cpp
  /// Load a benchmark
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

    problem_count = int(problem_sizes.size());
    return true;
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "38_syr2k_grouped\n\n"
      << "  This example profiles the performance of a 'grouped' SYR2K kernel. This example closely follows 24_gemm_grouped\n\n"
      << "Options:\n\n"
      << "  --help                           If specified, displays this usage statement.\n\n"
      << "  --benchmark=<str>                Executes a benchmark problem size.\n"
      << "  --output_file=<str>              Path to a CSV file to output results. If it exists already, results are appended.\n"
      << "  --tag=<str>                      String tag to prepend to the CSV file.\n"
      << "  --groups=<int>                   Number of individual SYR2K problems (default: --groups=15)\n"
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
      << "  --sort-problems=<bool>           If true, sorts problem sizes in descending order of SYR2K-K dimension.\n";

    out << "\n\nExamples:\n\n"

      << "# Runs a grouped SYR2K with 100 random problem sizes\n"
      << "$ ./examples/38_syr2k_grouped/38_syr2k_grouped --groups=100\n\n"

      << "# Runs a grouped SYR2K with 100 random problem sizes (with K dimension equal to 1024)\n"
      << "$ ./examples/38_syr2k_grouped/38_syr2k_grouped --groups=100 --k=1024 --verbose=true\n\n"

      << "# Runs a grouped SYR2K that is equivalent to a batched SYR2K\n"
      << "$ ./examples/38_syr2k_grouped/38_syr2k_grouped --groups=100 --n=1024 --k=1024 --verbose=true\n\n"

      << "# Runs a grouped SYR2K with each different scheduler mode\n"
      << "$ ./examples/38_syr2k_grouped/38_syr2k_grouped --scheduler-modes=all\n\n"

      << "# Runs a grouped SYR2K with each different scheduler mode and profiles host-side initialization time\n"
      << "$ ./examples/38_syr2k_grouped/38_syr2k_grouped --scheduler-modes=all --profile-initialization=true\n\n"

      << "# Runs a grouped SYR2K problem given an externally supplied benchmark file. This is a text file in which\n"
      << "# Each line contains a unique group index and an MxNxK triple indicating problemsize. NOTE that the\n"
      << "# GEMM-M and GEMM-N dimensions must match.\n"
      << "#\n"
      << "# For example, assume the following are the contents of 'problems.txt'\n"
      << "#\n"
      << "# 0 256x256x520\n"
      << "# 1 264x264x1024\n"
      << "# 2 48x48x1024\n"
      << "#\n"
      << "$ ./examples/38_syr2k_grouped/38_syr2k_grouped --benchmark=problems.txt\n\n"

      << "# Execute Grouped SYR2K and profile with NSight\n"
      << "$ nv-nsight-cu-cli ./examples/38_syr2k_grouped/38_syr2k_grouped --n=256 --k=256 --verbose=true --iterations=1 --reference-check=false\n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {

    // Number of real-valued multiply-adds
    int64_t fmas = int64_t();

    for (auto const & problem : problem_sizes) {
      fmas += problem.product();
    }

    // SYR2K is defined as (A x BT) + (B x AT), so the number of FMAs is twice that in a GEMM
    fmas *= 2;

    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: `benchmark_problems()` loads problem sizes from a text file (format: `index MxNxK`), rounding each dimension up to the nearest multiple of `alignment`. The `gflops()` helper correctly accounts for the SYR2K formula: the operation is the sum of two GEMM-like products (A·Bᵀ + B·Aᵀ), so the FMA count is doubled relative to a single GEMM of the same dimensions. FLOPs = 2 × 2 × n² × k (factor 2 for SYR2K symmetry, factor 2 for multiply-add).

**CN**: `benchmark_problems()` 从文本文件加载问题规模（格式：`索引 MxNxK`），并将每个维度向上取整至 `alignment` 的倍数。`gflops()` 辅助函数正确地考虑了 SYR2K 公式：该操作是两个类 GEMM 乘积之和（A·Bᵀ + B·Aᵀ），因此与相同维度的单个 GEMM 相比，FMA 计数翻倍。FLOPs = 2 × 2 × n² × k（因子 2 来自 SYR2K 对称性，另一个因子 2 来自乘加运算）。

---

### Lines 426–498 — `BaseTestbed`: Type Extraction / 基础测试台：类型提取

```cpp
template <typename Rank2K>
class BaseTestbed {
public:
  //
  // Type definitions
  //

  using ElementA = typename Rank2K::ElementA;
  using ElementB = typename Rank2K::ElementB;
  using ElementC = typename Rank2K::ElementC;
  using ElementAccumulator = typename Rank2K::ElementAccumulator;

  using EpilogueOutputOp = typename Rank2K::Rank2Kkernel::Epilogue::OutputOp;
  using ElementCompute = typename EpilogueOutputOp::ElementCompute;

  using LayoutA = typename Rank2K::LayoutA;
  using LayoutB = typename Rank2K::LayoutB;
  using LayoutC = typename Rank2K::LayoutC;

  using MatrixCoord = typename LayoutC::TensorCoord;

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

  cutlass::DeviceAllocation<int64_t> lda;
  cutlass::DeviceAllocation<int64_t> ldb;
  cutlass::DeviceAllocation<int64_t> ldc;
  cutlass::DeviceAllocation<int64_t> ldd;

  cutlass::DeviceAllocation<ElementA> block_A;
  cutlass::DeviceAllocation<ElementB> block_B;
  cutlass::DeviceAllocation<ElementC> block_C;
  cutlass::DeviceAllocation<ElementC> block_D;

  cutlass::DeviceAllocation<ElementA *> ptr_A;
  cutlass::DeviceAllocation<ElementB *> ptr_B;
  cutlass::DeviceAllocation<ElementC *> ptr_C;
  cutlass::DeviceAllocation<ElementC *> ptr_D;

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
```
**EN**: `BaseTestbed` is a CRTP-style helper templatised on the device-level Rank2K operator. Type aliases are extracted from the kernel type through CUTLASS's nested type convention. The key data layout:
- All group matrices are stored in **contiguous flat device allocations** (`block_A`, `block_B`, etc.).
- `offset_A[i]` gives the byte offset of problem `i` within `block_A`.
- `ptr_A`, `ptr_B`, `ptr_C`, `ptr_D` are **device arrays of pointers** — each element is a device pointer to one problem's matrix. The grouped kernel receives these pointer-arrays.
- `lda`, `ldb`, `ldc`, `ldd` are **device arrays of leading dimensions** (one per problem).
- `problem_sizes_device` is a device copy of the `GemmCoord` array passed to the grouped kernel.

**CN**: `BaseTestbed` 是一个以设备级 Rank2K 算子为模板参数的 CRTP 风格辅助类。类型别名通过 CUTLASS 的嵌套类型约定从内核类型中提取。关键数据布局：
- 所有分组矩阵存储在**连续的设备内存平铺分配**中（`block_A`、`block_B` 等）。
- `offset_A[i]` 给出问题 `i` 在 `block_A` 中的偏移量。
- `ptr_A`、`ptr_B`、`ptr_C`、`ptr_D` 是**设备指针数组**——每个元素都是指向某个问题矩阵的设备指针。分组内核接收这些指针数组。
- `lda`、`ldb`、`ldc`、`ldd` 是**设备端 leading dimension 数组**（每个问题一个）。
- `problem_sizes_device` 是传递给分组内核的 `GemmCoord` 数组的设备副本。

---

### Lines 499–554 — `initialize_tensor()` / 张量初始化辅助函数

```cpp
  /// Helper to initialize a tensor view
  template <typename Element>
  void initialize_tensor(
    Element *ptr,
    size_t capacity,
    cutlass::Distribution::Kind dist_kind,
    uint32_t seed) {

    if (dist_kind == cutlass::Distribution::Uniform) {

      Element scope_max, scope_min;
      int bits_input = cutlass::sizeof_bits<Element>::value;
      int bits_output = cutlass::sizeof_bits<ElementC>::value;

      if (bits_input == 1) {
        scope_max = 2;
        scope_min = 0;
      } else if (bits_input <= 8) {
        scope_max = 2;
        scope_min = -2;
      } else if (bits_output == 16) {
        if (cutlass::sizeof_bits<ElementAccumulator>::value <= 16) {
          scope_max = 5;
          scope_min = -5;
        }
        else {
          scope_max = 8;
          scope_min = -8;
        }
      } else {
        scope_max = 8;
        scope_min = -8;
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
```
**EN**: Fills a flat device buffer using CUTLASS reference utilities. The Uniform case scales the numeric range based on the element's bit-width to avoid accumulator overflow when the K-dimension is large. `BlockFillRandomUniform` and `BlockFillRandomGaussian` are GPU-side fill kernels driven by a seed, ensuring reproducibility. The scope ranges (e.g. [-8, 8] for FP64) are chosen conservatively for numerics.

**CN**: 使用 CUTLASS 参考工具填充平坦的设备缓冲区。均匀分布情况下，数值范围根据元素的位宽进行缩放，以防止当 K 维度较大时累加器溢出。`BlockFillRandomUniform` 和 `BlockFillRandomGaussian` 是由种子驱动的 GPU 端填充内核，确保可重现性。数值范围（例如 FP64 的 [-8, 8]）出于数值稳定性的考虑保守选取。

---

### Lines 555–652 — `allocate()` and `initialize()` / 分配与初始化设备内存

```cpp
  /// Allocates device-side data
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

      lda_host.at(i) = LayoutA::packed({problem.n(), problem.k()}).stride(0);
      ldb_host.at(i) = LayoutB::packed({problem.n(), problem.k()}).stride(0);
      ldc_host.at(i) = LayoutC::packed({problem.n(), problem.n()}).stride(0);
      ldd_host.at(i) = LayoutC::packed({problem.n(), problem.n()}).stride(0);

      offset_A.push_back(total_elements_A);
      offset_B.push_back(total_elements_B);
      offset_C.push_back(total_elements_C);
      offset_D.push_back(total_elements_D);

      int64_t elements_A = problem.n() * problem.k();
      int64_t elements_B = problem.n() * problem.k();
      int64_t elements_C = problem.n() * problem.n();
      int64_t elements_D = problem.n() * problem.n();

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
  }

  /// Initializes device-side data
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
  }
```
**EN**: `allocate()` computes the packed (row-major or column-major) leading stride for each problem using the layout's `packed()` factory. Because SYR2K writes a symmetric square output (N×N), only N appears in C/D dimensions. All problems are concatenated in a single flat device allocation and each problem's pointer is computed as `block_A.get() + offset_A[i]`. `initialize()` copies the host-side arrays of strides and pointers to device memory, then populates the data tensors with random values (different seeds per tensor to avoid correlated data).

**CN**: `allocate()` 使用布局的 `packed()` 工厂函数为每个问题计算紧密排列（行优先或列优先）的 leading stride。由于 SYR2K 写入对称方阵输出（N×N），C/D 维度中只出现 N。所有问题被连接在一个连续的设备内存分配中，每个问题的指针计算为 `block_A.get() + offset_A[i]`。`initialize()` 将主机端的步长和指针数组复制到设备内存，然后用随机值填充数据张量（每个张量使用不同的种子以避免数据相关性）。

---

### Lines 653–723 — `BaseTestbed::verify()` / 验证正确性

```cpp
  /// Verifies the result is a SYR2K
  bool verify() {

    bool passed = true;

    for (int32_t i = 0; i < problem_count(); ++i) {
      cutlass::gemm::GemmCoord problem = options.problem_sizes.at(i);

      LayoutA layout_A(lda_host.at(i));
      LayoutB layout_B(ldb_host.at(i));
      LayoutC layout_C(ldc_host.at(i));
      LayoutC layout_D(ldd_host.at(i));

      cutlass::HostTensor<ElementA, LayoutA> host_A(
        typename LayoutA::TensorCoord(problem.n(), problem.k()), /*device_backed=*/false);
      cutlass::HostTensor<ElementB, LayoutB> host_B(
        typename LayoutB::TensorCoord(problem.n(), problem.k()), /*device_backed=*/false);
      cutlass::HostTensor<ElementC, LayoutC> host_C(
        typename LayoutC::TensorCoord(problem.n(), problem.n()), /*device_backed=*/false);
      cutlass::HostTensor<ElementC, LayoutC> host_D(
        typename LayoutC::TensorCoord(problem.n(), problem.n()), /*device_backed=*/false);

      cutlass::device_memory::copy_to_host(host_A.host_data(), block_A.get() + offset_A.at(i), problem.n() * problem.k());
      cutlass::device_memory::copy_to_host(host_B.host_data(), block_B.get() + offset_B.at(i), problem.n() * problem.k());
      cutlass::device_memory::copy_to_host(host_C.host_data(), block_C.get() + offset_C.at(i), problem.n() * problem.n());
      cutlass::reference::host::BlockFillSequential(
        host_D.host_data(), problem.n() * problem.n(), ElementC(), ElementC());

      MatrixCoord extent_C{problem.n(), problem.n()};

      // Reference Rank2K
      cutlass::reference::host::Rank2KComplex<
        ElementA, LayoutA,
        ElementB, LayoutB,
        ElementC, LayoutC,
        ElementC, ElementAccumulator
      >(
        problem,
        (double)options.alpha,
        host_A.host_view(),
        Rank2K::kTransformA,
        host_B.host_view(),
        Rank2K::kTransformB,
        (double)options.beta,
        host_C.host_view(),
        host_D.host_view(),
        ElementAccumulator(0),
        Rank2K::kFillModeC,
        Rank2K::kBlasMode
      );

      // Copy to host memory
      std::vector<ElementC> matrix_D(layout_D.capacity(extent_C));
      cutlass::device_memory::copy_to_host(matrix_D.data(), block_D.get() + offset_D.at(i), matrix_D.size());

      cutlass::TensorView<ElementC, LayoutC> view_D(matrix_D.data(), layout_D, extent_C);
      cutlass::TensorView<ElementC, LayoutC> view_Ref = host_D.host_view();

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
**EN**: For each problem in the group the routine copies device results back to the host and runs `cutlass::reference::host::Rank2KComplex` — a scalar CPU reference that supports complex transforms (`kTransformA/B`) and both `kSymmetric` and `kHermitian` BLAS modes. `kFillModeC` indicates whether to compare only the lower or upper triangle of C. `TensorEquals` performs an element-wise comparison. The reference uses `ElementAccumulator(0)` as the initial accumulator value (no β·C term in the accumulation itself; α and β are handled separately).

**CN**: 对分组中的每个问题，该函数将设备结果复制回主机，并运行 `cutlass::reference::host::Rank2KComplex`——一个标量 CPU 参考实现，支持复数变换（`kTransformA/B`）以及 `kSymmetric` 和 `kHermitian` BLAS 模式。`kFillModeC` 指示是否只比较 C 的下三角或上三角部分。`TensorEquals` 执行逐元素比较。参考实现使用 `ElementAccumulator(0)` 作为初始累加值（累加过程本身不含 β·C 项；α 和 β 单独处理）。

---

### Lines 724–1029 — `TestbedConventional`: Conventional (Per-Stream) Baseline / 常规（逐流）基线测试台

```cpp
template <typename Rank2K>
class TestbedConventional : BaseTestbed<Rank2K> {
public:
  TestbedConventional(
    Options &options_,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint32_t seed_ = 3080
  ): BaseTestbed<Rank2K>(options_, init_A_, init_B_, init_C_, seed_) {}

  /// Verbose printing of problem sizes
  void print_problem_sizes() {

    // Print groups
    std::cout << this->problem_count() << " groups:\n";

    int32_t idx = 0;
    int64_t total_tiles = 0;

    for (auto const & problem : this->options.problem_sizes) {
      int tiles =
        ((problem.m() + Rank2K::ThreadblockShape::kM - 1) / Rank2K::ThreadblockShape::kM) *
        ((problem.n() + Rank2K::ThreadblockShape::kN - 1) / Rank2K::ThreadblockShape::kN);

      total_tiles += tiles;

      std::cout << "  [" << idx << "]: "
        << problem.m() << "-by-" << problem.n() << "-by-" << problem.k()
        << " (" << tiles << " threadblock tiles)" << "\n";

      ++idx;
    }
    std::cout << std::endl;
  }

  /// Executes a conventional SYR2K kernel.
  Result profile() {
    std::cout << "Conventional Rank2K:\n"
      << "====================================================" << std::endl;

    Result result;
    result.passed = false;

    // Initialize the problem
    this->allocate();
    this->initialize();

    if (this->options.verbose) {
      print_problem_sizes();
    }

    //
    // Create CUDA streams to maximize concurrency of SYR2K kernels
    //
    int32_t effective_streams = (this->options.cuda_streams ? this->options.cuda_streams : 1);
    std::vector<cudaStream_t> cuda_streams;
    char const *provider = "CUTLASS";

    //
    // Warmup run
    //

    if (this->options.cuda_streams) {
      for (int i = 0; i < this->options.cuda_streams; ++i) {
        cudaStream_t stream;

        result.error = cudaStreamCreate(&stream);
        if (result.error != cudaSuccess) {
        std::cerr << "Failed to create CUDA stream." << std::endl;
          return result;
        }
        cuda_streams.push_back(stream);
      }
    }
    else {
      cuda_streams.push_back(nullptr);
    }

    // Use 'D' for the in/out workspace
    this->block_D.copy_from_device(this->block_C.get());

    for (size_t i = 0; i < this->options.problem_sizes.size(); ++i) {
      cutlass::gemm::GemmCoord const & problem = this->options.problem_sizes[i];
      int32_t batch_count = 1;
      int64_t lda = this->lda_host.at(i);
      int64_t ldb = this->ldb_host.at(i);
      int64_t ldc = this->ldc_host.at(i);
      typename Rank2K::ElementA* ptrA = this->block_A.get() + this->offset_A.at(i);
      typename Rank2K::ElementB* ptrB = this->block_B.get() + this->offset_B.at(i);
      typename Rank2K::ElementC* ptrC = this->block_C.get() + this->offset_C.at(i);
      typename Rank2K::ElementC* ptrD = this->block_D.get() + this->offset_D.at(i);

      //
      // Initialize the CUTLASS SYR2K operator
      //

      // Configure the SYR2K arguments
      typename Rank2K::EpilogueOutputOp::Params epilogue_op(this->options.alpha, this->options.beta);

      typename Rank2K::Arguments arguments{
        cutlass::gemm::GemmUniversalMode::kGemm,
        problem,
        batch_count,
        epilogue_op,
        (void const *)ptrA,
        (void const *)ptrB,
        (void const *)ptrC,
        (void       *)ptrD,
        int64_t(),
        int64_t(),
        int64_t(),
        int64_t(),
        int64_t(lda),
        int64_t(ldb),
        int64_t(ldc),
        int64_t(ldc)
      };

      Rank2K rank2k_op;

      cutlass::Status status = rank2k_op.initialize(arguments);

      if (status != cutlass::Status::kSuccess) {
        std::cerr << "CUTLASS error on line " << __LINE__ << std::endl;
        return result;
      }

      status = rank2k_op();

      if (status != cutlass::Status::kSuccess) {
        std::cerr << "CUTLASS error on line " << __LINE__ << std::endl;
        return result;
      }
    }

    //
    // Wait for completion
    //

    result.error = cudaDeviceSynchronize();

    if (result.error != cudaSuccess)  {
      std::cerr << "Kernel execution error: " << cudaGetErrorString(result.error);
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

    //
    // Wait for completion
    //

    result.error = cudaDeviceSynchronize();

    if (result.error != cudaSuccess)  {
      std::cerr << "Kernel execution error: " << cudaGetErrorString(result.error);
      return result;
    }

    // Record an event at the start of a series of SYR2K operations
    result.error = cudaEventRecord(events[0]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    //
    // Run profiling loop
    //

    size_t last_stream_idx = 0;

    for (int iter = 0; iter < this->options.iterations; ++iter) {
      for (size_t i = 0; i < this->options.problem_sizes.size(); ++i) {
        cutlass::gemm::GemmCoord const & problem = this->options.problem_sizes[i];
        int32_t batch_count = 1;
        int64_t lda = this->lda_host.at(i);
        int64_t ldb = this->ldb_host.at(i);
        int64_t ldc = this->ldc_host.at(i);
        typename Rank2K::ElementA* ptrA = this->block_A.get() + this->offset_A.at(i);
        typename Rank2K::ElementB* ptrB = this->block_B.get() + this->offset_B.at(i);
        typename Rank2K::ElementC* ptrC = this->block_C.get() + this->offset_C.at(i);
        typename Rank2K::ElementC* ptrD = this->block_D.get() + this->offset_D.at(i);

        last_stream_idx = (i % effective_streams);

        //
        // Initialize the CUTLASS SYR2K operator
        //

        // Configure the SYR2K arguments
        typename Rank2K::EpilogueOutputOp::Params epilogue_op(this->options.alpha, this->options.beta);

        typename Rank2K::Arguments arguments{
          cutlass::gemm::GemmUniversalMode::kGemm,
          problem,
          batch_count,
          epilogue_op,
          (void const *)ptrA,
          (void const *)ptrB,
          (void const *)ptrC,
          (void       *)ptrD,
          int64_t(),
          int64_t(),
          int64_t(),
          int64_t(),
          int64_t(lda),
          int64_t(ldb),
          int64_t(ldc),
          int64_t(ldc)
        };

        Rank2K rank2k_op;

        cutlass::Status status = rank2k_op.initialize(arguments);

        if (status != cutlass::Status::kSuccess) {
          std::cerr << "CUTLASS error on line " << __LINE__ << std::endl;
          return result;
        }

        status = rank2k_op(cuda_streams[last_stream_idx]);

        if (status != cutlass::Status::kSuccess) {
          std::cerr << "CUTLASS error on line " << __LINE__ << std::endl;
          return result;
        }
      }
    }

    //
    // Stop profiling loop
    //

    // Record an event when the SYR2K operations have been launched.
    result.error = cudaEventRecord(events[1]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    //
    // Wait for work to be completed
    //

    result.error = cudaDeviceSynchronize();

    if (result.error != cudaSuccess)  {
      std::cerr << "Kernel execution error: " << cudaGetErrorString(result.error);
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
      (void)cudaEventDestroy(event);
    }

    for (auto stream : cuda_streams) {
      if (stream) {
        (void)cudaStreamDestroy(stream);
      }
    }

    std::cout << "    " << this->options.problem_sizes.size() << " conventional Rank2Ks launched" << std::endl;
    std::cout << std::endl;
    std::cout << "    " << "Conventional Runtime: " << result.runtime_ms << " ms" << std::endl;
    std::cout << "    " << "Conventional  GFLOPS: " << result.gflops << std::endl;

    if (this->options.output_file.good()) {
      this->options.output_file << this->options.output_tag << "," << provider << ",conventional,"
        << this->problem_count() << "," << result.runtime_ms << "," << result.gflops << std::endl;
    }

    result.passed = true;
    return result;
  }
};
```
**EN**: `TestbedConventional` launches one `cutlass::gemm::device::Rank2K` kernel per problem on a round-robin set of CUDA streams. `print_problem_sizes()` shows the tile count (number of CTA threadblocks) for each problem using `ThreadblockShape::kM/kN`. The `Arguments` struct uses `GemmUniversalMode::kGemm` to signal a standard (non-batched) Rank2K. Streaming parallelism is limited by GPU resources and memory bandwidth. This baseline is compared against the grouped kernel to quantify the benefit of fused scheduling.

**CN**: `TestbedConventional` 对每个问题在轮询的 CUDA stream 集合上各启动一个 `cutlass::gemm::device::Rank2K` 内核。`print_problem_sizes()` 使用 `ThreadblockShape::kM/kN` 显示每个问题的 tile 数（CTA 线程块数）。`Arguments` 结构体使用 `GemmUniversalMode::kGemm` 表示标准（非批处理）Rank2K。流式并行性受 GPU 资源和内存带宽限制。该基线与分组内核进行比较，以量化融合调度的收益。

---

### Lines 1030–1096 — `TestbedGrouped`: Kernel Type Specialisation / 分组测试台：内核类型特化

```cpp
template <typename Rank2K_, cutlass::gemm::kernel::GroupScheduleMode GroupScheduleMode_>
class TestbedGrouped : BaseTestbed<Rank2K_> {
public:
  TestbedGrouped(
    Options &options_,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint32_t seed_ = 3080
  ) : BaseTestbed<Rank2K_>(options_, init_A_, init_B_, init_C_, seed_) {}

  // Redefine Rank2K with different GroupScheduleMode_
    using Rank2Kkernel = typename cutlass::gemm::kernel::DefaultRank2KGrouped<
    typename Rank2K_::ElementA, typename Rank2K_::LayoutA, Rank2K_::kTransformA, Rank2K_::kAlignmentA,
    typename Rank2K_::ElementB, typename Rank2K_::LayoutB, Rank2K_::kTransformB, Rank2K_::kAlignmentB,
    typename Rank2K_::ElementC, typename Rank2K_::LayoutC, Rank2K_::kFillModeC,
    typename Rank2K_::ElementAccumulator,
    typename Rank2K_::OperatorClass,
    typename Rank2K_::ArchTag,
    typename Rank2K_::ThreadblockShape,
    typename Rank2K_::WarpShape,
    typename Rank2K_::InstructionShape,
    typename Rank2K_::EpilogueOutputOp,
    typename Rank2K_::ThreadblockSwizzle,
    Rank2K_::kStages,
    typename Rank2K_::Operator::ArchMmaOperator::Operator,
    Rank2K_::kBlasMode,
    GroupScheduleMode_>::Rank2Kkernel;

  using Rank2K = cutlass::gemm::device::Rank2KGrouped<Rank2Kkernel>;

  /// Verbose printing of problem sizes
  void print_problem_sizes() {

    // Print groups
    std::cout << this->problem_count() << " groups:\n";

    int32_t idx = 0;
    int64_t total_tiles = 0;

    for (auto const & problem : this->options.problem_sizes) {
      int tiles = Rank2K::problem_tile_count(problem);
      total_tiles += tiles;

      std::cout << "  [" << idx << "]: "
        << problem.m() << "-by-" << problem.n() << "-by-" << problem.k()
        << " (" << tiles << " threadblock tiles)" << "\n";

      ++idx;
    }
    std::cout << std::endl;
  }

  /// Sort problems in descending order of problem-K dimension
  void sort_problems() {
    Rank2K::sort_problems(this->options.problem_count,
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
```
**EN**: `TestbedGrouped` is templatised on both the base Rank2K type *and* the `GroupScheduleMode_` enum value. Inside the class it re-derives the actual grouped kernel by instantiating `DefaultRank2KGrouped<...>` with the scheduling mode baked in as a non-type template parameter. `DefaultRank2KGrouped` is a kernel policy selector that emits the correct tile iterator, scheduler, and epilogue combination. `Rank2K::sufficient()` queries the number of threadblocks the device can support.

**CN**: `TestbedGrouped` 以基础 Rank2K 类型和 `GroupScheduleMode_` 枚举值为双模板参数。在类内部，它通过以调度模式作为非类型模板参数实例化 `DefaultRank2KGrouped<...>` 来重新推导实际的分组内核。`DefaultRank2KGrouped` 是内核策略选择器，负责生成正确的 tile 迭代器、调度器和 epilogue 组合。`Rank2K::sufficient()` 查询设备能支持的线程块数量。

---

### Lines 1097–1297 — `TestbedGrouped::profile()` — Grouped Launch & Profiling / 分组启动与性能分析

```cpp
  /// Executes a grouped kernel and measures runtime.
  Result profile() {
    std::string sched_mode = this->options.scheduler_mode_to_str.find(GroupScheduleMode_)->second;
    std::cout << std::endl;
    std::cout << "Grouped Rank2K (CUTLASS) with mode " << sched_mode << ":\n"
      << "====================================================" << std::endl;

    Result result;

    int threadblock_count = Rank2K::sufficient(this->options.problem_sizes.data(), this->options.problem_count);

    // Early exit
    if (!threadblock_count) {
      std::cout << "Active CUDA device lacks hardware resources to run CUTLASS Grouped SYR2K kernel." << std::endl;
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

    // Configure the Rank2K arguments
    typename Rank2K::EpilogueOutputOp::Params epilogue_op(this->options.alpha, this->options.beta);

    // Configure Rank2K arguments
    typename Rank2K::Arguments args(
      cutlass::gemm::GemmUniversalMode::kGemm,
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

    // Initialize the Rank2K object
    Rank2K rank2k{};
    size_t workspace_size = rank2k.get_workspace_size(args);
    cutlass::DeviceAllocation<uint8_t> workspace(workspace_size);

    result.status = rank2k.initialize(args, workspace.get());

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize CUTLASS Grouped Rank2K kernel." << std::endl;
      return result;
    }

    // Run the grouped Rank2K object
    result.status = rank2k.run();

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to run CUTLASS Grouped Rank2K kernel." << std::endl;
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
    // Warm-up run of the grouped Rank2K object
    //
    result.status = rank2k.run();

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to run CUTLASS Grouped Rank2K kernel." << std::endl;
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

    // Record an event at the start of a series of SYR2K operations
    result.error = cudaEventRecord(events[0]);
    if (result.error != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result.error) << std::endl;
      return result;
    }

    //
    // Run profiling loop
    //

    for (int iter = 0; iter < this->options.iterations; ++iter) {
      rank2k();
    }

    //
    // Stop profiling loop
    //

    // Record an event when the Rank2K operations have been launched.
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
      (void)cudaEventDestroy(event);
    }

    // Optionally profile initialization
    if (this->options.profile_initialization) {
      // Warm up
      rank2k.initialize(args, workspace.get());

      auto start_time = std::chrono::high_resolution_clock::now();
      for (int32_t i = 0; i < this->options.iterations; ++i) {
        rank2k.initialize(args, workspace.get());
      }
      auto end_time = std::chrono::high_resolution_clock::now();

      std::chrono::duration<double, std::milli> duration = end_time - start_time;
      duration /= double(this->options.iterations);
      result.initialization_time_ms = duration.count();
    }

    int64_t total_tiles = Rank2K::group_tile_count(args);
    std::cout << "    " << total_tiles << " total threadblock tiles." << std::endl;

    std::cout << std::endl;
    std::cout << "    " << "Grouped Runtime: " << result.runtime_ms << " ms" << std::endl;
    std::cout << "    " << "Grouped  GFLOPs: " << result.gflops << std::endl;
    if (this->options.profile_initialization) {
      std::cout << "    " << "Init    Runtime: " << result.initialization_time_ms << " ms" << std::endl;
    }

    if (this->options.output_file.good()) {
      this->options.output_file << this->options.output_tag << ",CUTLASS,grouped-" << sched_mode << ","
        << this->problem_count() << "," << result.runtime_ms << "," << result.gflops << std::endl;
    }

    std::cout << "\nPassed\n";

    return result;
  }
};
```
**EN**: This is the core of the grouped SYR2K launch. Key observations:
1. `Rank2K::sufficient()` checks that the device has enough SM resources (shared memory) to execute the grouped kernel. It returns the number of concurrent threadblocks.
2. `Arguments` passes **device pointer arrays** for A, B, C, D and **device leading-dimension arrays** — not a single monolithic matrix.
3. `get_workspace_size()` / `initialize()` allocates and sets up a CUDA workspace for the grouped scheduler (tile→problem lookup tables for `kHostPrecompute`, or work counters for `kDeviceOnly`).
4. `rank2k.run()` launches the single fused kernel. All groups are dispatched from one kernel grid.
5. The kernel invocation operator `rank2k()` re-uses the already-initialised state for the profiling loop.
6. `group_tile_count()` sums all tile counts across groups for reporting.

**CN**: 这是分组 SYR2K 启动的核心。关键要点：
1. `Rank2K::sufficient()` 检查设备是否有足够的 SM 资源（共享内存）来执行分组内核，返回并发线程块数量。
2. `Arguments` 传入**设备指针数组**（A、B、C、D）和**设备 leading dimension 数组**——而非单个整体矩阵。
3. `get_workspace_size()` / `initialize()` 为分组调度器分配并设置 CUDA 工作区（`kHostPrecompute` 模式下的 tile→problem 查找表，或 `kDeviceOnly` 模式下的工作计数器）。
4. `rank2k.run()` 启动单个融合内核，所有分组从一个内核 grid 中调度。
5. 内核调用运算符 `rank2k()` 在分析循环中复用已初始化的状态。
6. `group_tile_count()` 对所有分组的 tile 计数求和以供报告。

---

### Lines 1298–1422 — `main()`: Type Definition and Operator Instantiation / 主函数：类型定义与算子实例化

```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, char const **args) {

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (__CUDACC_VER_MAJOR__ < 11 || props.major < 8) {

    //
    // This example requires an NVIDIA Ampere-architecture GPU.
    //

    std::cout
      << "CUTLASS's Grouped Rank2K example requires a GPU of NVIDIA's Ampere Architecture or "
      << "later (compute capability 80 or greater).\n";

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
  // Define the Grouped and Conventional Rank2K types
  //

  using ElementA = double;
  using ElementB = double;
  using ElementOutput = double;
  using ElementAccumulator = double;
  const cutlass::FillMode kFillModeC = cutlass::FillMode::kLower;
  const int kAlignmentA = 1;
  const int kAlignmentB = 1;
  const cutlass::ComplexTransform kTransformA = cutlass::ComplexTransform::kNone;
  const cutlass::ComplexTransform kTransformB = cutlass::ComplexTransform::kNone;

  using LayoutA = cutlass::layout::ColumnMajor;
  using LayoutB = cutlass::layout::ColumnMajor;
  using LayoutC = cutlass::layout::ColumnMajor;

  using OperatorClass = cutlass::arch::OpClassTensorOp;
  using ArchTag = cutlass::arch::Sm80;

  using ThreadblockShape = cutlass::gemm::GemmShape<32, 32, 16>;
  using WarpShape = cutlass::gemm::GemmShape<16, 16, 16>;
  using InstructionShape = cutlass::gemm::GemmShape<8, 8, 4>;

  using EpilogueOutputOp = cutlass::epilogue::thread::LinearCombination<
        ElementOutput, 1,
        ElementAccumulator, ElementAccumulator>;

  // NOTE: Threadblock swizzling is currently not supported by CUTLASS's grouped kernels.
  // This parameter is passed in at present to match the APIs of other kernels. The parameter
  // is unused within the kernel.
  using ThreadblockSwizzle = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

  const int kStages = 4;
  const bool kSplitKSerial = false;
  using Operator = cutlass::arch::OpMultiplyAdd;
  const cutlass::BlasMode kBlasMode = cutlass::BlasMode::kSymmetric;

  // Define a grouped Rank2K kernel with all template parameters set except
  // for scheduling mode. This will be used as the template for all scheduling
  // modes executed.
  using Rank2Kkernel = typename cutlass::gemm::kernel::DefaultRank2KGrouped<
    ElementA, LayoutA, kTransformA, kAlignmentA,
    ElementB, LayoutB, kTransformB, kAlignmentB,
    ElementOutput, LayoutC, kFillModeC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp,
    ThreadblockSwizzle,
    kStages,
    Operator,
    kBlasMode>::Rank2Kkernel;

  using Rank2KGrouped = cutlass::gemm::device::Rank2KGrouped<Rank2Kkernel>;

  // Rank2k operator
  using Rank2KConventional = cutlass::gemm::device::Rank2K<
    ElementA, LayoutA,
    ElementB, LayoutB,
    ElementOutput, LayoutC, kFillModeC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueOutputOp,
    ThreadblockSwizzle,
    kStages,
    kAlignmentA,
    kAlignmentB,
    kSplitKSerial,
    Operator,
    kTransformA,
    kTransformB,
    kBlasMode
  >;
```
**EN**: Template parameter breakdown:
- **FP64 throughout**: `ElementA = ElementB = ElementOutput = ElementAccumulator = double` — FP64 Tensor Core path on Ampere uses the `mma.sync.aligned.m8n8k4.f64` instruction.
- **ThreadblockShape (32×32×16)**: Each CTA computes a 32×32 tile of C accumulating 16 elements of K per mainloop iteration.
- **WarpShape (16×16×16)**: Each warp handles a 16×16 output tile; four warps tile the 32×32 threadblock.
- **InstructionShape (8×8×4)**: Maps to the FP64 Tensor Core MMA shape.
- **kFillModeC = kLower**: Only the lower triangular part of C is written (SYR2K symmetry property).
- **kBlasMode = kSymmetric**: Selects the symmetric update (as opposed to Hermitian).
- **kStages = 4**: Four-stage software pipeline in the mainloop for double-buffered prefetching.
- **`ThreadblockSwizzle` note**: Swizzling is not yet supported in grouped kernels; the parameter is present only for API compatibility.

**CN**: 模板参数详解：
- **全程使用 FP64**：`ElementA = ElementB = ElementOutput = ElementAccumulator = double`——Ampere 上的 FP64 Tensor Core 路径使用 `mma.sync.aligned.m8n8k4.f64` 指令。
- **ThreadblockShape（32×32×16）**：每个 CTA 计算 C 的 32×32 tile，每次主循环迭代累加 16 个 K 元素。
- **WarpShape（16×16×16）**：每个 warp 处理 16×16 输出 tile；四个 warp 平铺 32×32 线程块。
- **InstructionShape（8×8×4）**：对应 FP64 Tensor Core MMA 形状。
- **kFillModeC = kLower**：只写 C 的下三角部分（SYR2K 对称性质）。
- **kBlasMode = kSymmetric**：选择对称更新（相对于 Hermitian）。
- **kStages = 4**：主循环中的四阶段软件流水线，用于双缓冲预取。
- **`ThreadblockSwizzle` 注意**：分组内核尚不支持 swizzle；该参数仅出于 API 兼容性而保留。

---

### Lines 1423–1466 — `main()`: Profile Loop and Entry / 主函数：性能分析循环与入口

```cpp

  //
  // Profile it
  //

  TestbedConventional<Rank2KConventional> testbed(options);

  Result result = testbed.profile();
  if (!result.passed) {
    std::cout << "Profiling CUTLASS conventional Rank2K has failed.\n";
    std::cout << "\nFailed\n";
    return -1;
  }

  using GroupScheduleMode = cutlass::gemm::kernel::GroupScheduleMode;
  for (GroupScheduleMode mode : options.scheduler_modes) {
    Result result;
    switch (mode) {
      case GroupScheduleMode::kDeviceOnly:
        {
          TestbedGrouped<Rank2KGrouped, GroupScheduleMode::kDeviceOnly> runner(options);
          result = runner.profile();
          break;
        }
      case GroupScheduleMode::kHostPrecompute:
        {
          TestbedGrouped<Rank2KGrouped, GroupScheduleMode::kHostPrecompute> runner(options);
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
**EN**: `main()` first runs the conventional baseline, then iterates over the requested scheduler modes instantiating `TestbedGrouped` with each as a compile-time constant (template non-type parameter). The `switch` is necessary because `GroupScheduleMode_` must be a compile-time constant — CUTLASS selects different kernel code paths statically. After the first mode, `options.verbose` is cleared to avoid printing the problem-size table multiple times. Both scheduler modes use the same device-level `Rank2KGrouped` type; the difference is baked into the underlying `Rank2Kkernel` type.

**CN**: `main()` 首先运行常规基线，然后遍历请求的调度模式，将每个模式作为编译期常量（非类型模板参数）实例化 `TestbedGrouped`。`switch` 语句是必要的，因为 `GroupScheduleMode_` 必须是编译期常量——CUTLASS 静态地选择不同的内核代码路径。第一个模式之后，`options.verbose` 被清除，以避免多次打印问题规模表。两种调度模式使用相同的设备级 `Rank2KGrouped` 类型；区别被编码进底层 `Rank2Kkernel` 类型中。

---
## Key Concepts / 关键概念
- Grouped Rank2K scheduling compares per-problem launches with fused grouped kernels. / 分组 Rank2K 调度对比逐问题启动与融合分组内核。
- CUTLASS scheduler modes move tile-to-problem mapping either onto the device or into host-side precomputation. / CUTLASS 调度模式可将 tile 到问题的映射放在设备端，或提前在主机端预计算。
- The testbed pattern separates allocation, reference checking, and profiling for templated device operators. / Testbed 模式将模板化设备算子的内存分配、参考校验与性能分析解耦。
## Dependencies / 依赖项
- `cutlass/gemm/device/rank_2k_grouped.h` — Device-level grouped SYR2K operator / 设备级分组 SYR2K 算子
- `cutlass/gemm/kernel/default_rank_2k_grouped.h` — Builds the grouped SYR2K kernel type from template policies / 根据模板策略构建分组 SYR2K 内核类型
- `cutlass/util/host_tensor.h` — Host/device tensor container for initialization and verification / 用于初始化与校验的主机/设备张量容器
- `cutlass/util/reference/host/rank_2k_complex.h` — CPU reference implementation for correctness checks / 用于正确性校验的 CPU 参考实现
