# ell_block_sparse_gemm.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/43_ell_block_sparse_gemm/ell_block_sparse_gemm.cu`
**Purpose / 用途**: Implements a Blocked-ELL sparse GEMM example with host-side setup, verification, and benchmarking. / 实现一个 Blocked-ELL 稀疏 GEMM 示例，包含主机端准备、验证与基准测试。
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
**EN**: Standard NVIDIA BSD-3-Clause license, present in all CUTLASS examples.

**CN**: 标准 NVIDIA BSD-3-Clause 许可证，所有 CUTLASS 示例均包含此许可证。

---

### Lines 31–63 — File Docstring: ELL Format Description / 文件说明注释：ELL 格式描述

```cpp

/*! \file
    \brief Block-Ell sparse gemm example.

    This example performs a Sparse-matrix dense-matrix multiplication (SpMM) operation.
    Matrix A is stored in the Blocked-Ellpack (Blocked-ELL) storage format.
    Details about the Blocked-Ellpack (Blocked-ELL) storage format can be found here:
    https://docs.nvidia.com/cuda/cusparse/index.html#cusparse-generic-spmat-create-blockedell
    Whereas matrix B is a dense matrix.

    Blocked-Ellpack or Blocked-ELL storage format comprises of two matrices.
    First is a packed matrix (ellValue matrix) that stores non-zero values in consecutive blocks,
    represented by tensor_a in this example. Second is a matrix of indices (ellColInd matrix),
    represented by tensor_ell_idx in this example, that represent the column indices of the 
    corresponding non-zero blocks. All rows in the matrices must have the same number of blocks.
    ellColInd can contain -1 values for indicating empty blocks. These matrices store elements in
    row-major order.

    Description of parameters and tensors used to represent the Blocked-Ellpack (ELL) format
    for this example:
      a_rows              - Rows in the sparse matrix.
      a_cols              - Columns in the sparse matrix.
      a_ell_blocksize     - Size of the ELL-Blocks.
      a_ell_num_columns   - Number of columns in the Blocked-Ellpack format (ellValue columns)
      tensor_a            - ellValue matrix, whose size is (a_rows * a_ell_num_columns)
      tensor_ell_idx      - Blocked-ELL Column indices (ellColInd), whose size is
                            (a_rows / a_ell_blocksize) * (a_ell_num_columns / a_ell_blocksize)
      tensor_b            - Input dense matrix whose size is (a_cols * n)
      tensor_c/tensor_d   - Output dense matrix whose size is (a_rows * n)
      {a_rows, n, a_cols} - Problem size
    
*/
```
**EN**: The Blocked-ELL format is a block-structured extension of the classic ELLPACK format (used in sparse linear algebra). Key concepts:
- **`a_ell_num_columns`**: the number of non-zero columns kept per row in the packed matrix. All rows must have the same number, so rows with fewer non-zeros are padded with `-1` index sentinels.
- **`a_ell_blocksize`**: the square block size (e.g., 16). The column index matrix is `(a_rows/blocksize) × (a_ell_num_columns/blocksize)` — each index entry covers an entire `blocksize × blocksize` block of values.
- **`tensor_ell_idx`** entries of `-1` signal an empty (all-zero) block; the kernel skips these blocks, realizing the sparsity savings.
- **Problem size** is expressed as `{a_rows, n, a_cols}` matching the GEMM convention `{M, N, K}`.

**CN**: Blocked-ELL 格式是经典 ELLPACK 格式（用于稀疏线性代数）的块结构扩展。关键概念：
- **`a_ell_num_columns`**：打包矩阵中每行保留的非零列数。所有行必须具有相同数量，因此非零元素较少的行用 `-1` 索引哨兵填充。
- **`a_ell_blocksize`**：方块大小（例如 16）。列索引矩阵为 `(a_rows/blocksize) × (a_ell_num_columns/blocksize)`——每个索引条目覆盖整个 `blocksize × blocksize` 数值块。
- **`tensor_ell_idx`** 中的 `-1` 条目表示空（全零）块；内核跳过这些块，从而实现稀疏性节省。
- **问题规模** 表示为 `{a_rows, n, a_cols}`，对应 GEMM 约定 `{M, N, K}`。

---

### Lines 64–91 — Includes / 头文件包含

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <unordered_map>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/kernel/gemm_grouped.h"
#include "cutlass/gemm/kernel/default_gemm_grouped.h"
#include "cutlass/gemm/device/ell_gemm.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/host/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/host_uncompress.h"

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Key ELL-specific headers:
- `cutlass/gemm/device/ell_gemm.h` — the device-level `EllGemm` operator that implements Blocked-ELL SpMM. It wraps a grouped-GEMM kernel internally, treating each ELL block as a separate GEMM tile.
- `cutlass/gemm/kernel/gemm_grouped.h` / `default_gemm_grouped.h` — the underlying grouped kernel infrastructure reused by `EllGemm`.
- `cutlass/util/host_uncompress.h` — provides `cutlass::uncompress_ell_block_sparse()` which expands the packed ELL tensor back into a full dense matrix for reference-check purposes.

**CN**: 关键 ELL 专用头文件：
- `cutlass/gemm/device/ell_gemm.h` — 实现 Blocked-ELL SpMM 的设备级 `EllGemm` 算子。它在内部封装了一个分组 GEMM 内核，将每个 ELL 块视为独立的 GEMM tile。
- `cutlass/gemm/kernel/gemm_grouped.h` / `default_gemm_grouped.h` — `EllGemm` 复用的底层分组内核基础设施。
- `cutlass/util/host_uncompress.h` — 提供 `cutlass::uncompress_ell_block_sparse()`，将打包的 ELL 张量展开回完整的稠密矩阵，用于参考检查。

---

### Lines 92–115 — `Result` Struct / 结果结构体

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
```
**EN**: Standard result aggregator holding timing, GFLOP/s, CUTLASS status, CUDA error, and pass/fail. `gflops` here reports the *theoretical* dense FLOP count (ignoring the actual sparsity savings) as `2 × a_rows × a_cols × n / 1e9 / runtime_s`, giving a lower bound on the effective hardware utilisation.

**CN**: 标准结果聚合器，保存计时、GFLOP/s、CUTLASS 状态、CUDA 错误和通过/失败标志。此处 `gflops` 报告的是*理论上*的稠密 FLOP 计数（忽略实际的稀疏性节省），计算为 `2 × a_rows × a_cols × n / 1e9 / runtime_s`，给出有效硬件利用率的下界。

---

### Lines 116–210 — `Options` Struct / 选项结构体

```cpp
// Command line options parsing
struct Options {

  bool help;
  bool reference_check;
  int iterations;
  int cuda_streams;
  int a_rows, n, a_cols;
  int a_ell_num_columns;
  int a_ell_blocksize;
  int a_base;
  float alpha;
  float beta;

  //
  // Methods
  // 

  Options():
    help(false),
    reference_check(true),
    iterations(20),
    cuda_streams(0),
    a_rows(1024),
    n(1024),
    a_cols(1024),
    a_ell_num_columns(512),
    a_ell_blocksize(16),
    a_base(0),
    alpha(1),
    beta()
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
    }

    cmd.get_cmd_line_argument("alpha", alpha, 1.0f);
    cmd.get_cmd_line_argument("beta", beta, 0.0f);    
    cmd.get_cmd_line_argument("iterations", iterations, 20);
    cmd.get_cmd_line_argument("streams", cuda_streams, 0);
    cmd.get_cmd_line_argument("reference-check", reference_check, true);

    cmd.get_cmd_line_argument("a_rows", a_rows, 1024);
    cmd.get_cmd_line_argument("n", n, 1024);
    cmd.get_cmd_line_argument("a_cols", a_cols, 1024);

    cmd.get_cmd_line_argument("a_ell_num_columns", a_ell_num_columns, 512);
    cmd.get_cmd_line_argument("a_ell_blocksize", a_ell_blocksize, 16);
    cmd.get_cmd_line_argument("a_base", a_base, 0);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "43_ell_block_sparse_gemm\n\n"
      << "  This example profiles the performance of a ELL block sparse GEMM kernel.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement.\n\n"
      << "  --a_rows=<int>              Sets the number of the rows of the sparse matrix.\n"
      << "  --n=<int>                   Sets the N dimension.\n"
      << "  --a_cols=<int>              Sets the number of columns of the sparse matrix.\n"
      << "  --a_ell_num_columns=<int>   Sets the actual number of columns of the Blocked-Ellpack format.\n"
      << "  --a_ell_blocksize=<int>     Sets the size of the ELL-Block.\n"
      << "  --a_base=<int>              Sets the base index.\n"
      << "  --alpha=<f32>               Epilogue scalar alpha (real part)\n"
      << "  --beta=<f32>                Epilogue scalar beta (real part)\n\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n"
      << "  --reference-check=<bool>    If true, performs reference check.\n";

    out << "\n\nExamples:\n\n"

      << "# Runs a 1024x1024x1024 ELL block sparse GEMM with 16x16 block size and actual 512 non-zero columns in A operand\n"
      << "$ ./examples/43_ell_block_sparse_gemm/43_ell_block_sparse_gemm --a_rows=1024 --n=1024 --a_cols=1024 --a_ell_num_columns=512 --a_ell_blocksize=16\n\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const {

    // Number of real-valued multiply-adds 
    int64_t fmas = (int64_t)a_rows * (int64_t)a_cols * (int64_t)n;
    
    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The ELL parameters are distinct from a standard GEMM:
- `a_rows × a_cols` — dimensions of the *logical* sparse matrix (not its stored representation).
- `a_ell_num_columns` — number of columns in the *packed* ELL value matrix (must be ≤ `a_cols`). This is the effective number of non-zero columns per row. The default (512 out of 1024) represents 50% column sparsity.
- `a_ell_blocksize` — block size for both the value blocks and the index blocks. Must divide both `a_rows` and `a_ell_num_columns`. Default is 16.
- `a_base` — base index for the column index array (0 = C-style, 1 = Fortran-style), matching cuSPARSE conventions.
- `gflops()` reports dense-equivalent FLOPs (`2 × M × N × K`) rather than actual sparse FLOPs, making comparisons with dense baselines straightforward.

**CN**: ELL 参数与标准 GEMM 有所不同：
- `a_rows × a_cols` — *逻辑*稀疏矩阵的维度（不是其存储表示）。
- `a_ell_num_columns` — *打包* ELL 值矩阵中的列数（必须 ≤ `a_cols`）。这是每行的有效非零列数。默认值（1024 中的 512）表示 50% 的列稀疏度。
- `a_ell_blocksize` — 值块和索引块的块大小。必须同时整除 `a_rows` 和 `a_ell_num_columns`。默认值为 16。
- `a_base` — 列索引数组的基础索引（0 = C 风格，1 = Fortran 风格），与 cuSPARSE 约定匹配。
- `gflops()` 报告稠密等效 FLOP（`2 × M × N × K`），而非实际的稀疏 FLOP，便于与稠密基线进行比较。

---

### Lines 211–275 — `Testbed`: Type Aliases and Data Members / 测试台：类型别名与数据成员

```cpp
template <typename Gemm>
class Testbed {
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

private:

  //
  // Data members
  //

  Options options;

  /// Initialization
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
  cutlass::Distribution::Kind init_C;
  cutlass::Distribution::Kind init_ELL;
  uint32_t seed;

  cutlass::HostTensor<ElementA, LayoutA> tensor_a;
  cutlass::HostTensor<ElementB, LayoutB> tensor_b;
  cutlass::HostTensor<ElementC, LayoutC> tensor_c;
  cutlass::HostTensor<ElementC, LayoutC> tensor_d;

  cutlass::HostTensor<ElementA, LayoutA> tensor_a_uncompressed;
  cutlass::HostTensor<ElementC, LayoutC> reference_d;

  cutlass::HostTensor<int32_t, LayoutA> tensor_ell_idx;

public:

  //
  // Methods
  //

  Testbed(
    Options const &options_,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_ELL_ = cutlass::Distribution::Uniform,
    uint32_t seed_ = 3080
  ):
    options(options_), init_A(init_A_), init_B(init_B_), init_C(init_C_), init_ELL(init_ELL_), seed(seed_) { }

private:
```
**EN**: This `Testbed` is parameterised on the `EllGemm` device type and extracts its element and layout types. The critical data members:
- `tensor_a` — the **ELL value matrix** (packed non-zeros), shape `(a_rows, a_ell_num_columns)`. Layout is `RowMajor`.
- `tensor_ell_idx` — the **ELL column-index matrix** (int32), shape `(a_rows/blocksize, a_ell_num_columns/blocksize)`. Each entry is the block-column index in the original sparse matrix, or -1 for empty blocks.
- `tensor_a_uncompressed` — a CPU-side dense expansion of `tensor_a` used only by the reference GEMM; not passed to the GPU SpMM kernel.
- `init_ELL` — a fourth `Distribution::Kind` specifically for populating the index tensor.

**CN**: 此 `Testbed` 以 `EllGemm` 设备类型为参数，并提取其元素和布局类型。关键数据成员：
- `tensor_a` — **ELL 值矩阵**（打包的非零值），形状为 `(a_rows, a_ell_num_columns)`，布局为 `RowMajor`。
- `tensor_ell_idx` — **ELL 列索引矩阵**（int32），形状为 `(a_rows/blocksize, a_ell_num_columns/blocksize)`。每个条目是原始稀疏矩阵中的块列索引，或 -1 表示空块。
- `tensor_a_uncompressed` — `tensor_a` 的 CPU 端稠密展开，仅供参考 GEMM 使用；不传递给 GPU SpMM 内核。
- `init_ELL` — 第四个 `Distribution::Kind`，专门用于填充索引张量。

---

### Lines 276–329 — `initialize_tensor_()` / 张量初始化辅助函数

```cpp
  /// Helper to initialize a tensor view
  template <typename Element, typename Layout>
  void initialize_tensor_(
    cutlass::TensorView<Element, Layout> view,
    cutlass::Distribution::Kind dist_kind,
    uint32_t seed) {

    if (dist_kind == cutlass::Distribution::Uniform) {

      Element scope_max, scope_min;
      int bits_input = cutlass::sizeof_bits<Element>::value;
      int bits_output = cutlass::sizeof_bits<typename Gemm::ElementC>::value;

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

      cutlass::reference::host::TensorFillRandomUniform(
        view, seed, scope_max, scope_min, 0);
    } 
    else if (dist_kind == cutlass::Distribution::Gaussian) {

      cutlass::reference::host::TensorFillRandomGaussian(
        view, seed, Element(), Element(0.5f));
    }
    else if (dist_kind == cutlass::Distribution::Sequential) {

      // Fill with increasing elements
      cutlass::reference::host::BlockFillSequential(
        view.data(), view.capacity(), Element(1), Element());
    } else {

      // Fill with all 1s
      cutlass::reference::host::BlockFillSequential(
        view.data(), view.capacity(), Element(), Element(1));
    }
  }
```
**EN**: Unlike the device-side `BlockFillRandomUniform` used in Example 38, this version operates on host memory via `TensorView`. The range scaling logic (`scope_max`/`scope_min`) is identical: it prevents numerical overflow in the FP16 accumulator for large K-dimension problems. For ELL SpMM the K-dimension is `a_cols` (the full sparse-matrix column count), which can be large, making conservative fill ranges important.

**CN**: 与示例 38 中使用的设备端 `BlockFillRandomUniform` 不同，此版本通过 `TensorView` 在主机内存上操作。范围缩放逻辑（`scope_max`/`scope_min`）相同：它防止大 K 维度问题中 FP16 累加器的数值溢出。对于 ELL SpMM，K 维度为 `a_cols`（完整稀疏矩阵的列数），可能很大，因此保守的填充范围非常重要。

---

### Lines 330–371 — `initialize_()`: ELL Tensor Setup / ELL 张量初始化设置

```cpp
  /// Initializes data structures
  void initialize_() {
    tensor_a.resize(cutlass::make_Coord(options.a_rows, options.a_ell_num_columns));
    tensor_b.resize(cutlass::make_Coord(options.a_cols, options.n));
    tensor_c.resize(cutlass::make_Coord(options.a_rows, options.n));
    tensor_d.resize(cutlass::make_Coord(options.a_rows, options.n));

    tensor_a_uncompressed.resize(cutlass::make_Coord(options.a_rows, options.a_cols));
    reference_d.resize(cutlass::make_Coord(options.a_rows, options.n));

    tensor_ell_idx.resize(cutlass::make_Coord(options.a_rows / options.a_ell_blocksize,
                          options.a_ell_num_columns / options.a_ell_blocksize));

    //
    // Initialize the problems of the workspace
    //

    initialize_tensor_(tensor_a.host_view(), init_A, seed * 2021);
    initialize_tensor_(tensor_b.host_view(), init_B, seed * 2022);
    initialize_tensor_(tensor_c.host_view(), init_C, seed * 2023);

    if (init_ELL == cutlass::Distribution::Uniform) {
      cutlass::reference::host::TensorFillRandomEllIdx(
          tensor_ell_idx.host_view(), seed,
          options.a_rows / options.a_ell_blocksize,
          options.a_ell_num_columns / options.a_ell_blocksize,
          options.a_cols / options.a_ell_blocksize);

    } else {
      for(int i = 0; i < options.a_rows / options.a_ell_blocksize; ++i) {
        for(int j = 0; j < options.a_ell_num_columns / options.a_ell_blocksize; ++j) {
          tensor_ell_idx.at({i, j}) = j+3;
        }
      }
    }

    tensor_a.sync_device();
    tensor_b.sync_device();
    tensor_c.sync_device();
    tensor_d.sync_device();
    tensor_ell_idx.sync_device();
  }
```
**EN**: The tensor shapes reveal the ELL data layout:
- `tensor_a` shape: `(a_rows, a_ell_num_columns)` — a row-major matrix of non-zero values; each row stores `a_ell_num_columns` values regardless of actual sparsity.
- `tensor_ell_idx` shape: `(a_rows/blocksize, a_ell_num_columns/blocksize)` — one integer per block, identifying which column of the *original* sparse matrix that block belongs to.
- `TensorFillRandomEllIdx` generates valid random column block indices in the range `[0, a_cols/blocksize)`, potentially with `-1` entries for empty blocks. The sequential fallback assigns contiguous block column indices `j+3` (offset to avoid column 0/1/2, simulating a specific sparsity pattern).
- `sync_device()` transfers all host tensors (including the index tensor) to device memory before kernel launch.

**CN**: 张量形状揭示了 ELL 数据布局：
- `tensor_a` 形状：`(a_rows, a_ell_num_columns)` — 行优先的非零值矩阵；无论实际稀疏性如何，每行存储 `a_ell_num_columns` 个值。
- `tensor_ell_idx` 形状：`(a_rows/blocksize, a_ell_num_columns/blocksize)` — 每个块一个整数，标识该块属于*原始*稀疏矩阵的哪一列。
- `TensorFillRandomEllIdx` 在范围 `[0, a_cols/blocksize)` 内生成有效的随机列块索引，可能包含 `-1` 条目表示空块。顺序回退方案将连续的块列索引 `j+3` 赋值（偏移量避开列 0/1/2，模拟特定的稀疏模式）。
- `sync_device()` 在内核启动前将所有主机张量（包括索引张量）传输到设备内存。

---

### Lines 372–445 — `verify_()`: Reference Check via Decompression / 参考检查：通过解压缩验证

```cpp

  /// Verifies the result is a GEMM
  bool verify_() {

    bool passed = true;

    tensor_d.sync_host();

    cutlass::uncompress_ell_block_sparse(
          tensor_a_uncompressed.host_ref(),
          tensor_a.host_ref(),
          tensor_ell_idx.host_ref(),
          options.a_rows,
          options.a_cols,
          options.a_ell_num_columns,
          options.a_ell_blocksize
    );

    cutlass::reference::host::Gemm<
        typename Gemm::ElementA, typename Gemm::LayoutA,                                             
        typename Gemm::ElementB, typename Gemm::LayoutB,                                             
        typename Gemm::ElementC, typename Gemm::LayoutC,                                             
        ElementCompute,
        ElementAccumulator, typename Gemm::Operator>                                                 
        reference_gemm;                                                                              
    
    reference_gemm(                                                                                  
      {options.a_rows, options.n, options.a_cols},
      options.alpha, 
      tensor_a_uncompressed.host_ref(), 
      tensor_b.host_ref(),
      options.beta,
      reference_d.host_ref(),
      ElementAccumulator(0)
    );

    // Reference check
    passed = cutlass::reference::host::TensorEquals(tensor_d.host_view(), reference_d.host_view());

    if (!passed) {
      std::cerr << "\n***\nError - problem failed the QA check\n***\n" << std::endl;

      std::stringstream fname;

      fname << "error_43_ell_block_sparse_gemm"
            << "mnk_"
            << options.a_rows << "x"
            << options.n << "x"
            << options.a_cols << "_"
            << options.a_ell_num_columns << "_"
            << options.a_ell_blocksize << ".txt";

      std::cout << fname.str() << std::endl;

      std::ofstream results(fname.str());

      results
        << "alpha: " << ElementCompute(options.alpha) << "\n"
        << "beta: "  << ElementCompute(options.beta) << "\n"
        << "block size: " << options.a_ell_blocksize << "\n"
        << "\nA:\n" << tensor_a.host_view() << "\n"
        << "\nA Ell Index:\n" << tensor_ell_idx.host_view() << "\n"
        << "\nB:\n" << tensor_b.host_view() << "\n"
        << "\nC:\n" << tensor_c.host_view() << "\n"
        << "\nD reference:\n" << reference_d.host_view() << "\n"
        << "\nD computed:\n" << tensor_d.host_view() << "\n";


      return passed;
    }
    
    return passed;
  }
```
**EN**: The verification strategy is:
1. `sync_host()` — copies the GPU-computed `tensor_d` back to the host.
2. `uncompress_ell_block_sparse()` — expands the packed ELL value tensor (`tensor_a`) plus the column index tensor (`tensor_ell_idx`) into a full dense `(a_rows × a_cols)` matrix (`tensor_a_uncompressed`). Blocks with index `-1` are filled with zeros.
3. A standard host-side GEMM reference (`cutlass::reference::host::Gemm`) computes `reference_d = alpha × A_dense × B + beta × C` using the uncompressed A.
4. `TensorEquals` compares element-by-element. Any discrepancy writes a detailed error file (`error_43_ell_block_sparse_gemm_*_*.txt`) with all tensors printed.

This round-trip validation confirms that the ELL addressing in the kernel correctly maps each non-zero block to its logical column in the sparse matrix.

**CN**: 验证策略为：
1. `sync_host()` — 将 GPU 计算的 `tensor_d` 复制回主机。
2. `uncompress_ell_block_sparse()` — 将打包的 ELL 值张量（`tensor_a`）加上列索引张量（`tensor_ell_idx`）展开为完整的稠密 `(a_rows × a_cols)` 矩阵（`tensor_a_uncompressed`）。索引为 `-1` 的块用零填充。
3. 标准主机端 GEMM 参考（`cutlass::reference::host::Gemm`）使用解压缩后的 A 计算 `reference_d = alpha × A_dense × B + beta × C`。
4. `TensorEquals` 逐元素比较。任何差异都会写入详细的错误文件（`error_43_ell_block_sparse_gemm_*_*.txt`），打印所有张量。

这种往返验证确认了内核中的 ELL 寻址正确地将每个非零块映射到稀疏矩阵中的逻辑列。

---

### Lines 446–477 — `Testbed::sufficient()` / 硬件资源充足性检查

```cpp
public:

  /// Returns the number of threadblocks to launch if the kernel can run on the target
  /// device. Otherwise, returns zero.
  bool sufficient() const {
    //
    // Determine SMEM requirements and waive if not satisfied
    //

    size_t smem_size = sizeof(typename Gemm::GemmKernel::SharedStorage);

    cudaDeviceProp properties;
    int device_idx;
    cudaError_t result = cudaGetDevice(&device_idx);

    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDevice() API call failed.");
    }

    result = cudaGetDeviceProperties(&properties, device_idx);

    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDeviceProperties() failed");
    }

    if (properties.sharedMemPerBlockOptin < smem_size) {
      return false;
    }

    return true;
  }
```
**EN**: Checks that the active GPU has enough per-block shared memory to accommodate the kernel's `SharedStorage` union. The ELL GEMM kernel inherits the shared memory layout from the grouped GEMM kernel, which allocates space for the mainloop pipeline buffers (A and B tiles), the epilogue workspace, and the work-scheduling counter. `sharedMemPerBlockOptin` (also written `sharedMemPerBlockOpt`) is the maximum shared memory per block when using `cudaFuncSetAttribute(PREFER_SHARED_MEMORY_CARVEOUT)` — on Ampere this is up to 163 KB per SM. If the required shared memory exceeds this, the kernel cannot run.

**CN**: 检查活动 GPU 是否有足够的每块共享内存来容纳内核的 `SharedStorage` 联合体。ELL GEMM 内核从分组 GEMM 内核继承共享内存布局，该布局为主循环流水线缓冲区（A 和 B tile）、epilogue 工作区和工作调度计数器分配空间。`sharedMemPerBlockOptin`（也写作 `sharedMemPerBlockOpt`）是使用 `cudaFuncSetAttribute(PREFER_SHARED_MEMORY_CARVEOUT)` 时每块的最大共享内存——在 Ampere 上每个 SM 最高可达 163 KB。如果所需共享内存超过此值，内核将无法运行。

---

### Lines 478–636 — `Testbed::profile()`: ELL SpMM Launch and Profiling / ELL SpMM 启动与性能分析

```cpp
  /// Executes a BlockedEll SpMM kernel and measures runtime.
  Result profile() {

    Result result;

    // Early exit
    if (!sufficient()) {
      std::cout << "Active CUDA device lacks hardware resources to run CUTLASS BlockedEll SpMM kernel." << std::endl;
      return result;
    }

    result.passed = false;

    // Initialize the problem
    initialize_();

    // Configure the GEMM arguments
    typename EpilogueOutputOp::Params epilogue_op(options.alpha, options.beta);

    // Configure GEMM arguments
    typename Gemm::Arguments args(
      {options.a_rows, options.n, options.a_cols},
      tensor_a.device_ref(),
      tensor_b.device_ref(),
      tensor_c.device_ref(),
      tensor_d.device_ref(),
      tensor_ell_idx.device_data(),
      options.a_ell_num_columns,
      options.a_ell_blocksize,
      options.a_base,
      epilogue_op 
    );

    // Initialize the GEMM object
    Gemm gemm{};

    result.status = gemm.initialize(args);

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize CUTLASS BlockedEll SpMM kernel." << std::endl;
      return result;
    }

    // Run the BlockedEll SpMM object
    result.status = gemm.run();

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to run CUTLASS BlockedEll SpMM kernel." << std::endl;
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

    if (options.reference_check) {
      result.passed = verify_();
    }

    //
    // Warm-up run
    //
    result.status = gemm.run();

    if (result.status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to run CUTLASS BlockedEll SpMM kernel." << std::endl;
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
    result.runtime_ms = double(runtime_ms) / double(options.iterations);
    result.gflops = options.gflops(result.runtime_ms / 1000.0);

    //
    // Cleanup
    //

    for (auto event : events) {
      (void)cudaEventDestroy(event);
    }

    std::cout << std::endl;
    std::cout << "ELL Block Sparse GEMM (CUTLASS):\n"
      << "====================================================" << std::endl;

    std::cout << std::endl;
    std::cout << "    " << "Runtime: " << result.runtime_ms << " ms" << std::endl;
    std::cout << "    " << " GFLOPs: " << result.gflops << std::endl;

    return result;
  }
};

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The `EllGemm::Arguments` structure uniquely extends the standard GEMM arguments with four ELL-specific fields:
- `tensor_ell_idx.device_data()` — raw `int32*` pointer to the block column index array on device.
- `a_ell_num_columns` — number of columns in the packed ELL matrix (the "width" of the ELL structure).
- `a_ell_blocksize` — block size used to interpret both the value and index tensors.
- `a_base` — base index offset for column indices (0 or 1).

Internally, `EllGemm` maps the ELL structure onto the grouped-GEMM scheduler: each non-empty ELL block becomes one "group" in the grouped GEMM, with the block's column index used to derive the offset into the dense B matrix. This reuse of the grouped kernel avoids duplicating the scheduler and epilogue infrastructure. The profiling flow follows the same warm-up + CUDA-events pattern as the other examples.

**CN**: `EllGemm::Arguments` 结构体在标准 GEMM 参数的基础上独特地添加了四个 ELL 专用字段：
- `tensor_ell_idx.device_data()` — 指向设备上块列索引数组的原始 `int32*` 指针。
- `a_ell_num_columns` — 打包 ELL 矩阵中的列数（ELL 结构的"宽度"）。
- `a_ell_blocksize` — 用于解释值张量和索引张量的块大小。
- `a_base` — 列索引的基础索引偏移量（0 或 1）。

在内部，`EllGemm` 将 ELL 结构映射到分组 GEMM 调度器：每个非空 ELL 块成为分组 GEMM 中的一个"组"，块的列索引用于推导进入稠密 B 矩阵的偏移量。这种对分组内核的复用避免了重复调度器和 epilogue 基础设施。性能分析流程遵循与其他示例相同的预热 + CUDA 事件模式。

---

### Lines 637–740 — `main()`: EllGemm Type Definition and Execution / 主函数：EllGemm 类型定义与执行

```cpp
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
      << "CUTLASS's BlockedEll SpMM example requires a GPU of NVIDIA's Ampere Architecture or "
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

  //
  // Define the BlockedEll type
  //

  using ElementA = cutlass::half_t;
  using ElementB = cutlass::half_t;
  using ElementOutput = cutlass::half_t;
  using ElementAccumulator = float;

  using LayoutA = cutlass::layout::RowMajor;
  using LayoutB = cutlass::layout::ColumnMajor;
  using LayoutC = cutlass::layout::ColumnMajor;

  constexpr int32_t kAlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;
  constexpr int32_t kAlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;

  using ThreadblockShape = cutlass::gemm::GemmShape<128, 128, 32>;
  using WarpShape = cutlass::gemm::GemmShape<64, 64, 32>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;

  constexpr int32_t kStages = 4;
  using Gemm = typename cutlass::gemm::device::EllGemm<
    ElementA, 
    LayoutA, 
    ElementB,
    LayoutB, 
    ElementOutput,
    LayoutC,
    ElementAccumulator, 
    cutlass::arch::OpClassTensorOp, 
    cutlass::arch::Sm80,
    ThreadblockShape,
    WarpShape, 
    InstructionShape,
    cutlass::epilogue::thread::LinearCombination<
        ElementOutput, 128 / cutlass::sizeof_bits<ElementOutput>::value,
        ElementAccumulator, ElementAccumulator>,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<8>, 
    kStages, kAlignmentA, kAlignmentB>;

  //
  // Profile it
  //

  Testbed<Gemm> testbed(options);

  if (!testbed.sufficient()) {
    std::cout << "The active CUDA device lacks sufficient hardware resources to execute this kernel.\n";
    return 0;
  }

  Result result = testbed.profile();
  if (!result.passed) {
    std::cout << "Profiling CUTLASS ELL block sparse GEMM has failed.\n";
    std::cout << "\nFailed\n";
    return -1;
  }

  std::cout << "\nPassed\n";

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Template parameter analysis for `EllGemm`:
- **FP16 × FP32 accumulate → FP16 output**: maximum throughput for deep-learning inference workloads. Both A and B are `cutlass::half_t`.
- **LayoutA = RowMajor**: the ELL value matrix is stored row-major (as required by the ELL format — rows correspond to sparse matrix rows).
- **LayoutB = ColumnMajor**: the dense B matrix is column-major, enabling efficient column-wise access when accumulating across the K dimension.
- **ThreadblockShape<128,128,32>**: a large 128×128 output tile. With `a_ell_blocksize=16`, each ELL block is a 16×16 sub-tile of the 128×128 CTA tile, so 8×8 = 64 ELL blocks contribute to one CTA's output.
- **WarpShape<64,64,32>**: four warps (2×2) tile the 128×128 CTA block.
- **InstructionShape<16,8,16>**: Ampere FP16 `mma.sync.aligned.m16n8k16` Tensor Core instruction.
- **`GemmIdentityThreadblockSwizzle<8>`**: swizzle factor of 8 — assigns threadblocks in a round-robin pattern across 8 "swizzle groups" to improve L2 cache hit rate for matrix B.
- **kStages = 4**: four-stage mainloop pipeline for aggressive prefetching.
- **kAlignmentA = 8**: enables 128-bit (16-byte) vectorised loads of FP16 from the ELL value tensor.

**CN**: `EllGemm` 的模板参数分析：
- **FP16 × FP32 累加 → FP16 输出**：深度学习推理工作负载的最大吞吐量。A 和 B 均为 `cutlass::half_t`。
- **LayoutA = RowMajor**：ELL 值矩阵以行优先方式存储（ELL 格式要求——行对应稀疏矩阵的行）。
- **LayoutB = ColumnMajor**：稠密 B 矩阵以列优先方式存储，在沿 K 维度累加时实现高效的按列访问。
- **ThreadblockShape<128,128,32>**：大型 128×128 输出 tile。当 `a_ell_blocksize=16` 时，每个 ELL 块是 128×128 CTA tile 的 16×16 子 tile，因此 8×8 = 64 个 ELL 块贡献给一个 CTA 的输出。
- **WarpShape<64,64,32>**：四个 warp（2×2）平铺 128×128 CTA 块。
- **InstructionShape<16,8,16>**：Ampere FP16 `mma.sync.aligned.m16n8k16` Tensor Core 指令。
- **`GemmIdentityThreadblockSwizzle<8>`**：swizzle 因子为 8——在 8 个"swizzle 组"之间以轮询模式分配线程块，提高矩阵 B 的 L2 缓存命中率。
- **kStages = 4**：四阶段主循环流水线，用于积极的预取。
- **kAlignmentA = 8**：允许对 ELL 值张量进行 128 位（16 字节）向量化 FP16 加载。

---
## Key Concepts / 关键概念
- Blocked-ELL stores sparse blocks and column indices separately so Tensor Core kernels can stream regular tiles. / Blocked-ELL 将稀疏块数据与列索引分离存储，使 Tensor Core 内核能够规则地流式处理 tile。
- CUTLASS EllGemm hides sparse addressing behind the same device-operator pattern used by dense GEMM examples. / CUTLASS `EllGemm` 在与稠密 GEMM 相同的设备算子模式下封装了稀疏寻址。
- Verification requires rebuilding dense reference views from compressed sparse metadata. / 正确性验证需要根据压缩稀疏元数据重建稠密参考视图。
## Dependencies / 依赖项
- `cutlass/gemm/device/ell_gemm.h` — Device operator for Blocked-ELL sparse GEMM / Blocked-ELL 稀疏 GEMM 的设备算子
- `cutlass/util/host_uncompress.h` — Expands compressed sparse tensors into dense references / 将压缩稀疏张量展开为稠密参考数据
- `cutlass/util/reference/host/gemm.h` — Dense host GEMM used as the correctness baseline / 作为正确性基线的稠密主机端 GEMM
- `cutlass/util/host_tensor.h` — Tensor storage for sparse metadata and dense matrices / 用于稀疏元数据与稠密矩阵的张量存储
