# fused_multihead_attention_fixed_seqlen.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/fused_multihead_attention_fixed_seqlen.cu`
**Purpose / 用途**: Fixed-sequence-length forward FMHA example that builds packed Q/K/V inputs, runs the fused kernel, checks correctness, and profiles throughput / 固定序列长度的前向 FMHA 示例，构造紧凑 Q/K/V 输入、运行融合内核、校验正确性并测量吞吐
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

/*! \file
    \brief CUTLASS Attention Example.

    This workload computes a fused multi head attention.
    Because it keeps the attention matrix in shared memory, it's both faster and
    uses less global memory.

    This is based on `"Self-Attention Does Not Need O(n^2) Memory" <http://arxiv.org/abs/2112.05682>`_,
    and very similar to `"FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness" <https://arxiv.org/abs/2205.14135>`_.

    Algorithm:
      In short, we can compute the output incrementally in blocks of size B,
      we just need to divide the final result by the sum of all coefficients in
      the softmax (which we compute incrementally) with the following pseudo-code:

      ``​`
      s_prime = torch.zeros([num_queries, B])
      O = torch.zeros([num_queries, head_size_v])
      for i in range(0, K.shape[0], B):
        si = exp((Q . K[i * B:(i+1) * B].t) * scale)
        sum_coefs += attn_unscaled.sum(-1)
        O  += si . V[i * B:(i+1) * B]
      O = O / s_prime
      ``​`

      In practice, and for numerical stability reasons,
      we also subtract the maximum so far (`mi`) before doing
      the exponential. When we encounter new keys, the maximum
      used to compute O so far (`m_prime`) can differ from the
      current maximum, so we update O before accumulating with

      ``​`
      O       = O * exp(m_prime - mi)
      m_prime = mi
      ``​`

    Implementation details:
      - `si` is stored in shared memory between the 2 back to back gemms
      - we keep and accumulate the output
      directly in registers if we can (`head_size_v <= 128`).
      Otherwise, we store it & accumulate in global memory (slower)
      - blocks are parallelized across the batch dimension, the number
      of heads, and the query sequence size


    Examples:

      # Run an attention example with default setup
      $ ./examples/41_fused_multi_head_attention/41_fused_multi_head_attention_fixed_seqlen

      # Run an attention example with custom setup
      $ ./examples/41_fused_multi_head_attention/41_fused_multi_head_attention_fixed_seqlen --head_number=2 --batch_size=3 --head_size=32 --head_size_v=64 --seq_length=512 --seq_length_kv=1024 --causal=true

      Acknowledgement: Fixed-sequence-length FMHA code was upstreamed by Meta xFormers (https://github.com/facebookresearch/xformers).
*/

/////////////////////////////////////////////////////////////////////////////////////////////////

#include <vector>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/kernel/gemm_grouped.h"
#include "cutlass/gemm/kernel/default_gemm_grouped.h"
#include "cutlass/gemm/device/gemm_grouped.h"
#include "cutlass/gemm/device/gemm_universal.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/host/gemm_complex.h"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_norm.h"

#include "cutlass/layout/matrix.h"
#include "cutlass/gemm/kernel/gemm_grouped.h"
#include "cutlass/gemm/kernel/gemm_transpose_operands.h"
#include "cutlass/gemm/kernel/default_gemm.h"
#include "cutlass/gemm/kernel/default_gemm_complex.h"
#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/gemm/gemm.h"

#include "cutlass/epilogue/threadblock/epilogue_with_visitor.h"
#include "cutlass/fast_math.h"
#include "kernel_forward.h"

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Lines 1-123. The opening comment explains the FlashAttention-style incremental softmax idea, while the include list shows this file is a forward FMHA example built on `kernel_forward.h` plus CUTLASS GEMM utilities. Unlike the backward harness, it is a self-contained benchmark/test program with command-line parsing, reference checking, and direct kernel launch support.
**CN**: 第1-123行：开头注释说明了类似 FlashAttention 的增量 softmax 思路，而包含列表表明该文件是一个基于 `kernel_forward.h` 与 CUTLASS GEMM 工具的前向 FMHA 示例。与反向 harness 不同，它是一个自包含的 benchmark/测试程序，具备命令行解析、参考结果校验以及直接 kernel 启动能力。
```cpp

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

/////////////////////////////////////////////////////////////////////////////////////////////////

// Command line options parsing
struct Options {

  bool help;
  bool error;
  bool reference_check;
  bool use_mask;
  bool causal;

  std::vector<cutlass::gemm::GemmCoord> problem_sizes0;
  std::vector<cutlass::gemm::GemmCoord> problem_sizes1;

  std::vector<cutlass::gemm::GemmCoord> problem_sizes0_real;
  std::vector<cutlass::gemm::GemmCoord> problem_sizes1_real;

  int alignment;
  int head_number;
  int batch_size;
  int head_size;
  int head_size_v;
  int seq_length;
  int seq_length_kv;
  int iterations;

  // alpha0, alpha1 and beta are fixed 
  // in this multi-head attention example
  float alpha0;
  float alpha1;
  float beta;

  //
  // Methods
  // 

  Options():
    help(false),
    error(false),
    alignment(1),
    reference_check(true),
    head_number(12),
    batch_size(16),
    head_size(64),
    head_size_v(64),
    seq_length(1024),
    seq_length_kv(1024),
    use_mask(false),
    iterations(20),
    causal(false)
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("alignment", alignment, 1);
    cmd.get_cmd_line_argument("head_number", head_number, 12);
    cmd.get_cmd_line_argument("batch_size", batch_size, 16);
    cmd.get_cmd_line_argument("head_size", head_size, 64);
    cmd.get_cmd_line_argument("head_size_v", head_size_v, head_size);
    cmd.get_cmd_line_argument("seq_length", seq_length, 1024);
    cmd.get_cmd_line_argument("seq_length_kv", seq_length_kv, seq_length);
    cmd.get_cmd_line_argument("use_mask", use_mask, false);
    cmd.get_cmd_line_argument("iterations", iterations, 20);
    cmd.get_cmd_line_argument("reference-check", reference_check, true);
    cmd.get_cmd_line_argument("causal", causal, true);

    randomize_problems();

  }
```
**EN**: Lines 124-223. The `Result` and `Options` types capture benchmark status and all runtime knobs. `Options::parse()` maps command-line arguments into attention dimensions, causal and masking flags, iteration count, and reference checking. At this stage the file establishes the host-side configuration space that later determines which FMHA kernel template can be instantiated and how the runner fills its parameters.
**CN**: 第124-223行：`Result` 与 `Options` 类型负责保存 benchmark 状态以及全部运行参数。`Options::parse()` 将命令行参数映射为 attention 维度、causal/mask 开关、迭代次数和参考校验选项。到这里，文件已经建立了后续 FMHA 模板实例化与参数填充所依赖的宿主侧配置空间。
```cpp
  void randomize_problems() {

    int problem_count = head_number * batch_size;

    problem_sizes0.reserve(problem_count);
    problem_sizes1.reserve(problem_count);

    // When using mask, the original inputs are not padded
    // and we need to save these info.
    if (use_mask) {
      problem_sizes0_real.reserve(problem_count);
      problem_sizes1_real.reserve(problem_count);
    }

    for (int i = 0; i < batch_size; ++i) {
      // problems belonging to the same batch share the same seq len
      int m_real = seq_length;
      int mkv_real = seq_length_kv;
      int m = (m_real + alignment - 1) / alignment * alignment;
      int mkv = (mkv_real + alignment - 1) / alignment * alignment;
      int k0 = head_size;
      int k1 = head_size_v;

      for (int j = 0; j < head_number; ++j) {
        cutlass::gemm::GemmCoord problem0(m, mkv, k0);
        cutlass::gemm::GemmCoord problem1(m, k1, mkv);
        problem_sizes0.push_back(problem0);
        problem_sizes1.push_back(problem1);

        if (use_mask) {
          cutlass::gemm::GemmCoord problem0_real(m_real, mkv_real, k0);
          cutlass::gemm::GemmCoord problem1_real(m_real, k1, mkv_real);
          problem_sizes0_real.push_back(problem0_real);
          problem_sizes1_real.push_back(problem1_real);
        }
      }
    }
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "41_fused_multi_head_attention_fixed_seqlen\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --head_number=<int>         Head number in multi-head attention (default: --head_number=12)\n"
      << "  --batch_size=<int>          Batch size in multi-head attention (default: --batch_size=16)\n"
      << "  --head_size=<int>           Head size in multi-head attention (default: --head_size=64)\n"
      << "  --head_size_v=<int>         Head size in multi-head attention for V (default: --head_size_v=head_size)\n"
      << "  --seq_length=<int>          Sequence length in multi-head attention for Q (default: --seq_length=1024)\n"
      << "  --seq_length_kv=<int>       Sequence length in multi-head attention for K/V (default: --seq_length_kv=seq_length)\n"
      << "  --use_mask=<bool>           If true, performs padding-like masking in softmax.\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n"
      << "  --reference-check=<bool>    If true, performs reference check.\n"
      << "  --causal=<bool>             If true, uses causal masking.\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {

    // Number of real-valued multiply-adds 
    int64_t fops = int64_t();

    for (size_t i = 0; i < problem_sizes0.size(); ++i) {
      auto const& problem0 = problem_sizes0[i];
      auto const& problem1 = problem_sizes1[i];
      for (int row = 0; row < problem0.m(); ++row) {
        int num_cols0 = problem0.n();
        if (causal) {
          num_cols0 = std::min(row + 1, num_cols0);
        }
        // P <- Q . K_t
        fops += 2 * num_cols0 * problem0.k();
        // P <- exp(P - max(P))
        fops += 2 * num_cols0;
        // S <- sum(P)
        fops += num_cols0 - 1;
        // O <- P . V
        fops += 2 * num_cols0 * problem1.n();
        // O <- O / S
        fops += num_cols0 * problem1.n();
      }
    }

    return double(fops) / double(1.0e9) / runtime_s;
  }
};
```
**EN**: Lines 224-312. `randomize_problems()` builds one `(Q·K^T)` problem and one `(P·V)` problem per batch/head pair, but in the fixed-sequence example every batch element uses the same `seq_length` and `seq_length_kv`. The helper also stores optional “real” sizes for masking, emits usage text, and estimates GFLOPs by replaying the logical FMHA computation cost row by row under optional causal masking.
**CN**: 第224-312行：`randomize_problems()` 会为每个 batch/head 组合构造一个 `(Q·K^T)` 问题和一个 `(P·V)` 问题，但在固定序列长度版本中，每个 batch 元素都使用相同的 `seq_length` 与 `seq_length_kv`。该辅助逻辑还会在启用 mask 时记录“真实”尺寸、输出用法说明，并按行重放 FMHA 逻辑计算量来估算 GFLOPs，同时考虑可选的 causal masking。
```cpp



///////////////////////////////////////////////////////////////////////////////////////////////////

template <typename Attention>
class TestbedAttention {
public:

  //
  // Type definitions
  //

  using ElementQ = typename Attention::scalar_t;
  using ElementK = typename Attention::scalar_t;
  using ElementP = typename Attention::accum_t;
  using ElementAccumulator = typename Attention::accum_t;
  using ElementV = typename Attention::scalar_t;
  using ElementO = typename Attention::output_t;

  using ElementCompute = typename Attention::accum_t;

  using ElementNorm = typename Attention::accum_t;
  using ElementSum = typename Attention::accum_t;
  using ElementSoftmaxCompute = typename Attention::accum_t;

  using LayoutQ = cutlass::layout::RowMajor;
  using LayoutK = cutlass::layout::ColumnMajor;
  using LayoutP = cutlass::layout::RowMajor;
  using LayoutV = cutlass::layout::RowMajor;
  using LayoutO = cutlass::layout::RowMajor;

  using MatrixCoord = typename LayoutP::TensorCoord;

private:

  //
  // Data members
  //

  Options & options;

  /// Initialization
  cutlass::Distribution::Kind init_Q;
  cutlass::Distribution::Kind init_K;
  cutlass::Distribution::Kind init_P;
  cutlass::Distribution::Kind init_V;
  cutlass::Distribution::Kind init_O;
  uint32_t seed;

  cutlass::DeviceAllocation<cutlass::gemm::GemmCoord> problem_sizes_device0;
  cutlass::DeviceAllocation<cutlass::gemm::GemmCoord> problem_sizes_device1;
  cutlass::DeviceAllocation<cutlass::gemm::GemmCoord> problem_sizes_device0_real;

  std::vector<int64_t> offset_Q;
  std::vector<int64_t> offset_K;
  std::vector<int64_t> offset_P;
  std::vector<int64_t> offset_V;
  std::vector<int64_t> offset_O;

  std::vector<int64_t> ldq_host;
  std::vector<int64_t> ldk_host;
  std::vector<int64_t> ldp_host;
  std::vector<int64_t> ldv_host;
  std::vector<int64_t> ldo_host;
  std::vector<int64_t> seqlen_host;

  cutlass::DeviceAllocation<int64_t> ldq;
  cutlass::DeviceAllocation<int64_t> ldk;
  cutlass::DeviceAllocation<int64_t> ldp;
  cutlass::DeviceAllocation<int64_t> ldv;
  cutlass::DeviceAllocation<int64_t> ldo;
  cutlass::DeviceAllocation<int64_t> seqlen;

  cutlass::DeviceAllocation<ElementQ> block_Q;
  cutlass::DeviceAllocation<ElementK> block_K;
  cutlass::DeviceAllocation<ElementP> block_P;
  cutlass::DeviceAllocation<ElementV> block_V;
  cutlass::DeviceAllocation<ElementO> block_O;
  cutlass::DeviceAllocation<ElementNorm> block_Norm;
  cutlass::DeviceAllocation<ElementSum> block_Sum;

  cutlass::DeviceAllocation<int64_t> offset_P_Device;

  cutlass::DeviceAllocation<ElementQ *> ptr_Q;
  cutlass::DeviceAllocation<ElementK *> ptr_K;
  cutlass::DeviceAllocation<ElementP *> ptr_P;
  cutlass::DeviceAllocation<ElementV *> ptr_V;
  cutlass::DeviceAllocation<ElementO *> ptr_O;

public:

  //
  // Methods
  //

  TestbedAttention(
    Options &options_,
    cutlass::Distribution::Kind init_Q_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_K_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_P_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_V_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_O_ = cutlass::Distribution::Uniform,
    uint32_t seed_ = 3080
  ):
    options(options_), init_Q(init_Q_), init_K(init_K_), init_P(init_P_), init_V(init_V_), init_O(init_O_), seed(seed_) { }

  int problem_count() const {
    return (options.head_number * options.batch_size);
  }

private:

  /// Helper to initialize a tensor view
  template <typename Element>
  void initialize_tensor_(
    Element *ptr,
    size_t capacity, 
    cutlass::Distribution::Kind dist_kind,
    uint32_t seed) {

    if (dist_kind == cutlass::Distribution::Uniform) {

      Element scope_max, scope_min;
      int bits_input = cutlass::sizeof_bits<Element>::value;
      int bits_output = cutlass::sizeof_bits<ElementP>::value;

      if (bits_input == 1) {
        scope_max = 2;
        scope_min = 0;
      } else if (bits_input <= 8) {
        scope_max = 2;
        scope_min = -2;
      } else if (bits_output == 16) {
        scope_max = 8;
        scope_min = -8;
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

  /// Initializes data structures
```
**EN**: Lines 313-476. `TestbedAttention` is the main host-side orchestrator. This first section defines all scalar/layout aliases, owns device buffers for problem sizes, strides, offsets, Q/K/P/V/O storage, and pointer arrays, and provides `initialize_tensor_()` for deterministic random initialization. The class is templated on the concrete `Attention` kernel type, so the same testbed can drive multiple launch configurations.
**CN**: 第313-476行：`TestbedAttention` 是主要的宿主侧调度器。这一段先定义标量与布局别名，持有 problem size、步长、偏移、Q/K/P/V/O 存储以及指针数组等设备缓冲区，并提供 `initialize_tensor_()` 用于确定性的随机初始化。该类以具体 `Attention` kernel 类型为模板参数，因此同一个 testbed 可以驱动多种启动配置。
```cpp
  void initialize_() {

    //
    // Set scalors for the mha example
    //

    options.alpha0 = 1.0f / sqrt(float(options.head_size));
    options.alpha1 = 1.0f;
    options.beta = 0;

    //
    // Choose random problem sizes
    //

    // construct a few problems of random sizes
    srand(seed);

    int64_t total_elements_Q = 0;
    int64_t total_elements_K = 0;
    int64_t total_elements_P = 0;
    int64_t total_elements_V = 0;
    int64_t total_elements_O = 0;

    ldq_host.resize(problem_count());
    ldk_host.resize(problem_count());
    ldp_host.resize(problem_count());
    ldv_host.resize(problem_count());
    ldo_host.resize(problem_count());
    seqlen_host.resize(problem_count());

    // Create tensors in BMHK format, where
    // B = batch_size
    // M = sequence length
    // H = num_heads
    // K = embedding size per head
    int64_t batch_offset_Q, batch_offset_K, batch_offset_V, batch_offset_O;

    for (int32_t b = 0; b < options.batch_size; ++b) {
      batch_offset_Q = total_elements_Q;
      batch_offset_K = total_elements_K;
      batch_offset_V = total_elements_V;
      batch_offset_O = total_elements_O;
      for (int32_t h = 0; h < options.head_number; ++h) {
        int32_t i = h + b * options.head_number;

        auto problem0 = options.problem_sizes0.at(i);
        auto problem1 = options.problem_sizes1.at(i);

        ldq_host.at(i) = LayoutQ::packed({problem0.m(), options.head_number * problem0.k()}).stride(0);
        ldk_host.at(i) = LayoutK::packed({options.head_number * problem0.k(), problem0.n()}).stride(0);
        ldp_host.at(i) = LayoutP::packed({problem0.m(), problem0.n()}).stride(0);
        ldv_host.at(i) = LayoutV::packed({problem1.k(), options.head_number * problem1.n()}).stride(0);
        ldo_host.at(i) = LayoutO::packed({problem1.m(), options.head_number * problem1.n()}).stride(0);

        // m = n for attention problems.
        seqlen_host.at(i) = problem0.m();

        offset_Q.push_back(batch_offset_Q + h * problem0.k());
        offset_K.push_back(batch_offset_K + h * problem0.k());
        offset_P.push_back(total_elements_P);
        offset_V.push_back(batch_offset_V + h * problem0.k());
        offset_O.push_back(batch_offset_O + h * problem1.n());

        int64_t elements_Q = problem0.m() * problem0.k();
        int64_t elements_K = problem0.k() * problem0.n();
        int64_t elements_P = problem0.m() * problem0.n();
        int64_t elements_V = problem1.k() * problem1.n();
        int64_t elements_O = problem1.m() * problem1.n();

        total_elements_Q += elements_Q;
        total_elements_K += elements_K;
        total_elements_P += elements_P;
        total_elements_V += elements_V;
        total_elements_O += elements_O;
      }
    }

    problem_sizes_device0.reset(problem_count());
    problem_sizes_device1.reset(problem_count());
    problem_sizes_device0.copy_from_host(options.problem_sizes0.data());
    problem_sizes_device1.copy_from_host(options.problem_sizes1.data());

    if (options.use_mask) {
      problem_sizes_device0_real.reset(problem_count());
      problem_sizes_device0_real.copy_from_host(options.problem_sizes0_real.data());
    }

    ldq.reset(problem_count());
    ldk.reset(problem_count());
    ldp.reset(problem_count());
    ldv.reset(problem_count());
    ldo.reset(problem_count());
    seqlen.reset(problem_count());

    ldq.copy_from_host(ldq_host.data());
    ldk.copy_from_host(ldk_host.data());
    ldp.copy_from_host(ldp_host.data());
    ldv.copy_from_host(ldv_host.data());
    ldo.copy_from_host(ldo_host.data());
    seqlen.copy_from_host(seqlen_host.data());

    //
    // Assign pointers
    //

    block_Q.reset(total_elements_Q);
    block_K.reset(total_elements_K);
    block_P.reset(total_elements_P);
    block_V.reset(total_elements_V);
    block_O.reset(total_elements_O);

    offset_P_Device.reset(problem_count());

    // sync offset with device
    cutlass::device_memory::copy_to_device(offset_P_Device.get(), offset_P.data(), offset_P.size());

    std::vector<ElementQ *> ptr_Q_host(problem_count());
    std::vector<ElementK *> ptr_K_host(problem_count());
    std::vector<ElementP *> ptr_P_host(problem_count());
    std::vector<ElementV *> ptr_V_host(problem_count());
    std::vector<ElementO *> ptr_O_host(problem_count());
    std::vector<ElementNorm *> ptr_norm_host(problem_count());
    std::vector<ElementSum *> ptr_sum_host(problem_count());

    for (int32_t i = 0; i < problem_count(); ++i) {
      ptr_Q_host.at(i) = block_Q.get() + offset_Q.at(i);
      ptr_K_host.at(i) = block_K.get() + offset_K.at(i);
      ptr_P_host.at(i) = block_P.get() + offset_P.at(i);
      ptr_V_host.at(i) = block_V.get() + offset_V.at(i);
      ptr_O_host.at(i) = block_O.get() + offset_O.at(i);
    }

    ptr_Q.reset(problem_count());
    ptr_Q.copy_from_host(ptr_Q_host.data());
    
    ptr_K.reset(problem_count());
    ptr_K.copy_from_host(ptr_K_host.data());
    
    ptr_P.reset(problem_count());
    ptr_P.copy_from_host(ptr_P_host.data());

    ptr_V.reset(problem_count());
    ptr_V.copy_from_host(ptr_V_host.data());

    ptr_O.reset(problem_count());
    ptr_O.copy_from_host(ptr_O_host.data());

    //
    // Initialize the problems of the workspace
    //

    initialize_tensor_(block_Q.get(), total_elements_Q, init_Q, seed + 1);
    initialize_tensor_(block_K.get(), total_elements_K, init_K, seed + 2);
    initialize_tensor_(block_V.get(), total_elements_V, init_V, seed + 3);

  }
```
**EN**: Lines 477-632. `initialize_()` converts the fixed logical FMHA shapes into packed BMHK storage. It computes per-head offsets, leading dimensions, total storage sizes, device-side `GemmCoord` arrays, and pointer tables that point each grouped subproblem into the big Q/K/P/V/O buffers. Because sequence length is fixed, the layout logic is simpler than the variable-length file: batch-local offsets are reused consistently across heads, and no per-problem scheduler workspace is needed.
**CN**: 第477-632行：`initialize_()` 将固定的逻辑 FMHA 形状转换为紧凑的 BMHK 存储布局。它计算每个 head 的偏移、leading dimension、总存储大小、设备端 `GemmCoord` 数组，以及指向大块 Q/K/P/V/O 缓冲区中各子问题位置的指针表。由于序列长度固定，这里的布局逻辑比可变长度版本更简单：batch 内偏移可在各 head 间稳定复用，也不需要额外的按问题调度 workspace。
```cpp

  template<typename Element>
  bool verify_tensor_(std::vector<Element> vector_Input, \
                       std::vector<Element> vector_Input_Ref,
                       int64_t verify_length = -1) {

    int64_t size = (vector_Input.size() < vector_Input_Ref.size()) ? vector_Input.size() : vector_Input_Ref.size();
    size = (verify_length == -1) ? size : verify_length;

    // 0.05 for absolute error
    float abs_tol = 5e-2f;
    // 10% for relative error
    float rel_tol = 1e-1f;
    for (int64_t i = 0; i < size; ++i) {
      float diff = (float)(vector_Input.at(i) - vector_Input_Ref.at(i));
      float abs_diff = fabs(diff);
      float abs_ref = fabs((float)vector_Input_Ref.at(i) + 1e-5f);
      float relative_diff = abs_diff / abs_ref;
      if ( (isnan(vector_Input_Ref.at(i)) || isnan(abs_diff) || isinf(abs_diff)) ||  (abs_diff > abs_tol && relative_diff > rel_tol)) {
        printf("[%d/%d] diff = %f, rel_diff = %f, {computed=%f, ref=%f}.\n", int(i), int(size), abs_diff, relative_diff, (float)(vector_Input.at(i)), (float)(vector_Input_Ref.at(i)));
        return false;
      }

    }

    return true;
  }

  /// Verifies the result is a GEMM
  bool verify_() {

    bool passed = true;

    for (int32_t b = 0; b < options.batch_size; ++b) {
      int32_t i = b * options.head_number;
      // Problem size is the same for all heads
      cutlass::gemm::GemmCoord problem0 = options.problem_sizes0.at(b * options.head_number);
      cutlass::gemm::GemmCoord problem1 = options.problem_sizes1.at(b * options.head_number);

      MatrixCoord extent_Q{problem0.m(), problem0.k()};
      MatrixCoord extent_K{problem0.k(), problem0.n()};
      MatrixCoord extent_P{problem0.m(), problem0.n()};
      MatrixCoord extent_V{problem1.k(), problem1.n()};
      MatrixCoord extent_O{problem1.m(), problem1.n()};

      LayoutO layout_O(ldo_host.at(i));
      std::vector<ElementO> matrix_O(layout_O.capacity(extent_O));
      cutlass::device_memory::copy_to_host(matrix_O.data(),   block_O.get() + offset_O.at(i), matrix_O.size());
      cutlass::DeviceAllocation<ElementO>    block_Ref_O(layout_O.capacity(extent_O));

      for (int32_t h = 0; h < options.head_number; ++h) {
        i = h + b * options.head_number;

        LayoutQ layout_Q(ldq_host.at(i));
        LayoutK layout_K(ldk_host.at(i));
        LayoutP layout_P(ldp_host.at(i));
        LayoutV layout_V(ldv_host.at(i));

        cutlass::TensorView<ElementQ, LayoutQ> view_Q(block_Q.get() + offset_Q.at(i), layout_Q, extent_Q);
        cutlass::TensorView<ElementK, LayoutK> view_K(block_K.get() + offset_K.at(i), layout_K, extent_K);
        cutlass::TensorView<ElementV, LayoutV> view_V(block_V.get() + offset_V.at(i), layout_V, extent_V);
        cutlass::TensorView<ElementO, LayoutO> view_Ref_O_device(block_Ref_O.get() + offset_O.at(i) - offset_O.at(b * options.head_number), layout_O, extent_O);

        cutlass::DeviceAllocation<ElementP>    block_Ref_P(layout_P.capacity(extent_P));
        cutlass::TensorView<ElementP, LayoutP> view_Ref_P_device(block_Ref_P.get(), layout_P, extent_P);

        // Reference GEMM
        cutlass::reference::device::GemmComplex<
            ElementQ, LayoutQ,
            ElementK, LayoutK,
            ElementP, LayoutP, 
            ElementCompute, ElementAccumulator
        >(
          problem0,
          ElementAccumulator(options.alpha0), 
          view_Q,
          Attention::MM0::Mma::kTransformA,
          view_K,
          Attention::MM0::Mma::kTransformB,
          ElementAccumulator(options.beta), 
          view_Ref_P_device, 
          view_Ref_P_device, 
          ElementAccumulator(0)
        );

        // Compute softmax for P. We need to explicitly compute softmax
        // over P because softmax is fused to the second GEMM in the
        // profiled implementation.
        std::vector<ElementP> matrix_Ref(layout_P.capacity(extent_P));
        cutlass::device_memory::copy_to_host(matrix_Ref.data(), block_Ref_P.get(), matrix_Ref.size());
        cutlass::TensorView<ElementP, LayoutP> view_Ref_host(matrix_Ref.data(), layout_P, extent_P);
        std::vector<ElementNorm> vector_Norm_Ref(problem0.m());
        std::vector<ElementSum> vector_Sum_Ref(problem0.m());

        int n_dim = options.use_mask ? options.problem_sizes0_real.at(i).n() : problem0.n();

        // Compute softmax for reference matrix
        for (int m = 0; m < problem0.m(); m++) {
          int n_dim_row = n_dim;
          if (options.causal) {
            n_dim_row = std::min(m + 1, n_dim);
          }
          ElementSoftmaxCompute max = ElementSoftmaxCompute(view_Ref_host.ref().at({m, 0}));
          for (int n = 1; n < n_dim_row; n++) {
            max = std::max(max, ElementSoftmaxCompute(view_Ref_host.ref().at({m, n})));
          }

          vector_Norm_Ref.at(m) = ElementNorm(max);

          ElementSoftmaxCompute sum = ElementSoftmaxCompute();
          for (int n = 0; n < n_dim_row; n++) {
            sum += std::exp( ElementSoftmaxCompute(view_Ref_host.ref().at({m, n})) - max );
          }
          ElementSoftmaxCompute inv_sum = ElementSoftmaxCompute(1.0f / sum);

          vector_Sum_Ref.at(m) = ElementSum(inv_sum);

          for (int n = 0; n < n_dim_row; n++) {
            view_Ref_host.ref().at({m, n}) = ElementP(
              std::exp( ElementSoftmaxCompute(view_Ref_host.ref().at({m, n})) - max ) * inv_sum
            );
          }
          // Mask out the rest of the attention matrix
          for (int n = n_dim_row; n < n_dim; ++n) {
            view_Ref_host.ref().at({m, n}) = ElementP(0);
          }
        }

        // when not using mask, problem_real and problem share the same sizes
        if (options.use_mask) {
          for (int m = 0; m < problem0.m(); m++) {
            for (int n = n_dim; n < problem0.n(); n++) {
              view_Ref_host.ref().at({m, n}) = ElementP(0);
            }
          }
        }

        cutlass::device_memory::copy_to_device(block_Ref_P.get(), matrix_Ref.data(), matrix_Ref.size());

        // Reference GEMM
        cutlass::reference::device::GemmComplex<
            ElementP, LayoutP,
            ElementV, LayoutV,
            ElementO, LayoutO, 
            ElementCompute, ElementAccumulator
        >(
          problem1,
          ElementAccumulator(options.alpha1), 
          view_Ref_P_device,
          Attention::MM0::Mma::kTransformA,
          view_V,
          Attention::MM0::Mma::kTransformB,
          ElementAccumulator(options.beta), 
          view_Ref_O_device, 
          view_Ref_O_device, 
          ElementAccumulator(0)
        );
      }

      // Copy to host memory
      std::vector<ElementO> matrix_Ref_O(layout_O.capacity(extent_O));
      cutlass::device_memory::copy_to_host(matrix_Ref_O.data(), block_Ref_O.get(), matrix_Ref_O.size());

      // printf("Pb %d: \n    Q=(offset=%d, ldq=%d)\n    K=(offset=%d, ldk=%d)\n    O=(offset=%d, ldo=%d)\n",
      //   int(i), int(offset_Q[i]), int(ldq_host[i]), int(offset_K[i]), int(ldk_host[i]), int(offset_O[i]), int(ldo_host[i]));
  
      bool verified_O = false;

      if (!verified_O) {
        verified_O = verify_tensor_<ElementO>(matrix_O, matrix_Ref_O);
      }

      passed = passed && verified_O;

      if (!passed) {
        std::cerr << "\n***\nError - problem " << i << " (batch " << b << ") failed the QA check\n***\n" << std::endl;

        if (!verified_O) {
          std::cout << "Final matrix output is incorrect" << std::endl;
        }

        return passed;
      }
    }

    return passed;
  }
```
**EN**: Lines 633-819. The verification path reconstructs FMHA in two stages: a reference GEMM for `Q·K^T`, an explicit numerically stable softmax with optional causal/mask trimming, and a second GEMM for `P·V`. The resulting reference output is compared against the kernel output with mixed absolute/relative tolerances. This block is important because it makes the forward invocation flow explicit in host code even though the profiled kernel fuses those steps internally.
**CN**: 第633-819行：校验路径按两个阶段重建 FMHA：先对 `Q·K^T` 做参考 GEMM，再执行带数值稳定处理的显式 softmax，并根据 causal/mask 进行裁剪，最后再做一次 `P·V` GEMM。得到的参考输出会以绝对/相对误差混合阈值与 kernel 输出比较。该部分很重要，因为虽然被测 kernel 在内部将这些步骤融合了，但宿主代码把完整前向调用流清楚地展开了。
```cpp

public:


  /// Executes a CUTLASS Attention kernel and measures runtime.
  Result profile() {

    Result result;
    result.passed = false;

    // Initialize the problem
    initialize_();

    typename Attention::Params p;
    { // set parameters
      p.query_ptr = block_Q.get();
      p.key_ptr = block_K.get();
      p.value_ptr = block_V.get();
      p.logsumexp_ptr = nullptr; // Only needed for bw
      p.output_accum_ptr = nullptr;
      if (Attention::kNeedsOutputAccumulatorBuffer) {
        cudaMalloc(&p.output_accum_ptr, block_O.size() * sizeof(typename Attention::output_accum_t));
      }
      p.output_ptr = block_O.get();

      // TODO: support arbitrary seq lengths
      // if (cu_seqlens_q.has_value()) {
      //   p.cu_seqlens_q_ptr = (int32_t*)cu_seqlens_q->data_ptr();
      //   p.cu_seqlens_k_ptr = (int32_t*)cu_seqlens_k->data_ptr();
      // }

      p.scale = options.alpha0;

      p.num_heads = options.head_number;
      p.num_batches = options.batch_size;
      p.head_dim = options.head_size;
      p.head_dim_value = options.head_size_v;
      p.num_queries = options.seq_length;
      p.num_keys = options.seq_length_kv;
      if (options.causal) {
        p.custom_mask_type = Attention::CausalFromTopLeft;
      }

      // All tensors are in BMHK shapes
      p.q_strideH = options.head_size;
      p.k_strideH = options.head_size;
      p.v_strideH = options.head_size_v;
      p.q_strideM = int32_t(ldq_host[0]);
      p.k_strideM = int32_t(ldk_host[0]);
      p.v_strideM = int32_t(ldv_host[0]);
      p.q_strideB = p.q_strideM * options.seq_length;
      p.k_strideB = p.k_strideM * options.seq_length_kv;
      p.v_strideB = p.v_strideM * options.seq_length_kv;
      p.o_strideM = p.head_dim_value * p.num_heads;
    }

    // launch kernel :)
    constexpr auto kernel_fn = attention_kernel_batched_impl<Attention>;
    int smem_bytes = sizeof(typename Attention::SharedStorage);
    if (smem_bytes > 0xc000) {
      cudaFuncSetAttribute(kernel_fn, cudaFuncAttributeMaxDynamicSharedMemorySize, smem_bytes);
    }
    if (!Attention::check_supported(p)) {
      std::cerr << "Kernel does not support these inputs" << std::endl;
      return result;
    }
    kernel_fn<<<p.getBlocksGrid(), p.getThreadsGrid(), smem_bytes>>>(p);

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

    if (options.reference_check) {
      result.passed = verify_();
    }

    //
    // Warm-up run
    //

    kernel_fn<<<p.getBlocksGrid(), p.getThreadsGrid(), smem_bytes>>>(p);

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to run CUTLASS Attention kernel." << std::endl;
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

    for (int iter = 0; iter < options.iterations; ++iter) {
      kernel_fn<<<p.getBlocksGrid(), p.getThreadsGrid(), smem_bytes>>>(p);
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
    result.runtime_ms = double(runtime_ms) / double(options.iterations);
    result.gflops = options.gflops(result.runtime_ms / 1000.0);

    //
    // Cleanup
    //

    for (auto event : events) {
      (void)cudaEventDestroy(event);
    }

    std::cout << std::endl;
    std::cout << "CUTLASS Attention:\n"
      << "====================================================" << std::endl;
    std::cout << "    " << " {seq length Q, seq length KV, head size, head size V, head number, batch size} = {" << options.seq_length \
      << ", " << options.seq_length_kv << ", " << options.head_size << ", " << options.head_size_v << ", " << options.head_number\
      << ", " << options.batch_size << "}." << std::endl;
    std::cout << std::endl;
    std::cout << "    " << "Runtime: " << result.runtime_ms << " ms" << std::endl;
    std::cout << "    " << "GFLOPs: " << result.gflops << std::endl;

    return result;
  }
};
```
**EN**: Lines 820-995. `profile()` wires benchmark data into `Attention::Params`. It sets Q/K/V/output pointers, allocates `output_accum_ptr` only when `kNeedsOutputAccumulatorBuffer` says the kernel needs a separate accumulation buffer, fills scale, sizes, mask mode, and BMHK strides, and then launches `attention_kernel_batched_impl<Attention>` directly. After synchronization it optionally verifies correctness, performs a warm-up, times repeated launches with CUDA events, and reports runtime plus GFLOPs.
**CN**: 第820-995行：`profile()` 会把 benchmark 数据接入 `Attention::Params`。它设置 Q/K/V/output 指针，只在 `kNeedsOutputAccumulatorBuffer` 表明需要单独累加缓冲区时才分配 `output_accum_ptr`，然后填入 scale、尺寸、mask 模式和 BMHK 步长，最终直接启动 `attention_kernel_batched_impl<Attention>`。同步后它可选地做正确性校验，进行一次预热，再用 CUDA event 对重复启动进行计时，并输出 runtime 与 GFLOPs。
```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

template <
  int kQueriesPerBlock,
  int kKeysPerBlock,
  int kMaxK
>
int run_attention(Options& options) {
  using Attention = AttentionKernel<
    cutlass::half_t,      // scalar_t
    cutlass::arch::Sm80,  // ArchTag
    true,                 // Memory is aligned
    kQueriesPerBlock,
    kKeysPerBlock,
    kMaxK,
    false,                // Supports dropout
    false                 // Supports bias
  >;

  //
  // Test and profile
  //

  TestbedAttention<Attention> testbed(options);

  Result result = testbed.profile();
  if (!result.passed) {
    std::cout << "Profiling CUTLASS attention has failed.\n";
    std::cout << "\nFailed\n";
    return -1;
  }

  std::cout << "\nPassed\n";
  return 0;
}
```
**EN**: Lines 996-1031. `run_attention<kQueriesPerBlock, kKeysPerBlock, kMaxK>()` is the template dispatch layer. It instantiates `AttentionKernel<half_t, Sm80, ...>` with concrete tile sizes and a maximum head dimension, plugs that type into `TestbedAttention`, and turns benchmark success/failure into a simple process return code. This is how command-line dimensions eventually choose a concrete kernel template.
**CN**: 第996-1031行：`run_attention<kQueriesPerBlock, kKeysPerBlock, kMaxK>()` 是模板分发层。它以具体 tile 大小和最大 head 维度实例化 `AttentionKernel<half_t, Sm80, ...>`，再把该类型交给 `TestbedAttention` 执行，并把 benchmark 成败转换为简单的进程返回码。这就是命令行尺寸最终映射到具体 kernel 模板的方式。
```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc, char const **args) {

  //
  // This example uses mma.sync to directly access Tensor Cores to achieve peak performance.
  //

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
      << "CUTLASS's CUTLASS Attention example requires a GPU of NVIDIA's Ampere Architecture or "
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

  if (options.use_mask) {
    std::cerr << "--use_mask is not supported at the moment\n";
    return -2;
  }
  if (options.alignment != 1) {
    std::cerr << "--alignment=1 is the only supported value\n";
    return -2;
  }

  // Determine kernel configuration based on head size.
  // If head size is less than or equal to 64, each block operates over 64 queries and
  // 64 keys, and partial results can be stored in the register file.
  // If head size is greater than 64, each block operates over 32 queries and 128 keys,
  // and partial results are stored in shared memory.
  if (options.head_size_v > 64) {
    static int const kQueriesPerBlock = 32;
    static int const kKeysPerBlock = 128;
    if (options.head_size_v <= 128) {
      return run_attention<kQueriesPerBlock, kKeysPerBlock, 128>(options);
    } else {
      return run_attention<kQueriesPerBlock, kKeysPerBlock, 65536>(options);
    }
  } else {
    static constexpr int kMaxK = 64; // <- Decrease to 32/16 if your problem is smaller
    static int const kQueriesPerBlock = 64;
    static int const kKeysPerBlock = 64;
    return run_attention<kQueriesPerBlock, kKeysPerBlock, kMaxK>(options);
  }
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Lines 1032-1110. `main()` performs architecture-specific and runtime guards first: it requires CUDA 11+ and an Ampere-or-later GPU (`props.major >= 8`), rejects unsupported `use_mask` and non-unit alignment, then selects one of two launch families based on `head_size_v`. Small value dimensions use `64x64` query/key blocks and `kMaxK=64`, while larger value dimensions switch to `32x128` tiles and either `kMaxK=128` or a large fallback. This final block is the top-level policy that wires user arguments into concrete FMHA kernel templates.
**CN**: 第1032-1110行：`main()` 先做架构相关与运行时保护：要求 CUDA 11+ 且 GPU 至少是 Ampere（`props.major >= 8`），同时拒绝当前不支持的 `use_mask` 和非 1 对齐。随后它根据 `head_size_v` 在两类启动配置之间选择：较小的 value 维度使用 `64x64` 的 query/key block 和 `kMaxK=64`，较大的 value 维度则切换到 `32x128` tile，并在 `kMaxK=128` 与更大的兜底值之间选择。这个收尾块就是把用户参数接入具体 FMHA kernel 模板的最高层策略。
---
## Key Concepts / 关键概念
- The forward computation is conceptually `Q·K^T -> softmax -> P·V`, but the actual kernel fuses those stages and the host code reconstructs them only for verification. / 前向计算在概念上是 `Q·K^T -> softmax -> P·V`，但实际 kernel 会把这些阶段融合起来，宿主代码只是在校验路径中显式重建它们。
- Fixed sequence length keeps problem construction simple: one consistent BMHK layout, one direct kernel launch path, and no grouped scheduler or per-problem workspace. / 固定序列长度使问题构造更简单：统一的 BMHK 布局、单一路径的直接 kernel 启动，以及无需 grouped 调度器或按问题分配的 workspace。
- Launch configuration is selected from `main()` by `head_size_v`, then materialized through `run_attention()` into `AttentionKernel<half_t, Sm80, ...>`. / 启动配置在 `main()` 中按 `head_size_v` 选择，再经 `run_attention()` 实体化为 `AttentionKernel<half_t, Sm80, ...>`。
- Architecture-specific safety is explicit: Ampere+ only, CUDA 11+, `Attention::check_supported(p)`, and optional dynamic shared-memory enlargement before launch. / 架构安全约束是显式的：仅支持 Ampere+、要求 CUDA 11+、通过 `Attention::check_supported(p)` 校验参数，并在需要时于启动前提升动态共享内存上限。
## Dependencies / 依赖项
- `kernel_forward.h` — Provides the fixed-shape fused forward kernel and parameter interface / 提供固定形状融合前向内核及其参数接口
- `cutlass/util/command_line.h` — Parses example flags such as head size, sequence length, and causality / 解析 head 大小、序列长度和因果模式等命令行参数
- `cutlass/util/reference/*` — Supplies host/device reference GEMM, fill, copy, and compare helpers / 提供主机与设备参考 GEMM、填充、拷贝和比较工具
- `cutlass/gemm/device/default_gemm_configuration.h` — Exposes architecture-specific GEMM defaults used by the example policy / 提供示例策略依赖的架构相关 GEMM 默认配置
