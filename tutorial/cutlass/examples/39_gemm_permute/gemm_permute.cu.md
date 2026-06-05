# gemm_permute.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/39_gemm_permute/gemm_permute.cu`
**Purpose / 用途**: Drives fused-permute GEMM experiments, reference checks, and profiling for CUTLASS Example 39. / 作为 CUTLASS 示例 39 的主程序，负责融合置换 GEMM 的实验、参考校验与性能分析。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1–32 — License

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
```
**EN**: Standard NVIDIA BSD-3-Clause license header, present in all CUTLASS source files.

**CN**: 标准 NVIDIA BSD-3 许可证头，出现在所有 CUTLASS 源文件中。

---

### Lines 33–67 — File Comment / Usage Examples

```cpp
    \brief GEMM Permute Example.

    This example computes batched GEMM operations with output results permuted as reshaped tensors.

    We provide layout plugin as a flexible tool for users to add any customized input/output tensor permute operation, 
    or any other generalized global memory writeout address computation. To add a customized layout, add new class
    in include/cutlass/layout/permute.h

    In this example we use several permute operations (permute([0, 2, 1, 3]))

    In this example, we used Tensor4DPermuteBMM0213 layout to perform Batched GEMM with permute([0, 2, 1, 3]) on BMM
    whole output tensor, and used Tensor5DPermute20314 layout to perform Normal GEMM with permute([2, 0, 3, 1, 4]) on
    output matrix. The address computations are performed in compute(col_init, row_init, stride_init, 
    BMM_batch_idx) with {col_permute, row_permute and stride_permute} as new addresses after permute op.
    (check include/cutlass/layout/permute.h)

    Tips:
    
      1) Make sure to set batch_stride to zero for BMM permute; also the BMM GEMM should be in mode
      cutlass::gemm::GemmUniversalMode::kBatched instead of kArray.

      2) When the contiguous dimension is touched in permute op (for example [0, 2, 3, 1] for row-major matrix 
      or [1, 0, 2, 3] for column-major), Alignment should be set to 1 for the corresponding matrix. 
      If the last dimension is untouched,  one can set Alignment to be larger like 8 in our example.
      As a result, permute op without touching the unit stride dimension is recommended to obtain the best performance.

    Examples:

      # Runs a batched GEMM with 96 batches
      $ ./examples/39_gemm_permute/39_gemm_permute --problem-count=96

      # Runs a batched GEMM with 96 batches (with GEMM-K dimension equal to 1024)
      $ ./examples/39_gemm_permute/39_gemm_permute --problem-count=96 --k=1024 --verbose=true

      # Execute batched GEMM and profile with NSight
```
**EN**: The Doxygen block explains the core concept: permute layout functions are *plugins* injected into CUTLASS's `GemmUniversal` kernel. They are not post-processing steps — the address computation for each output element is modified in place by the layout's `operator()`. Two critical tips are highlighted:
- For batched-mode permutations (BMM), `batch_stride` must be **zero** so that the permute layout controls inter-batch addressing entirely.
- **Alignment** must be reduced to 1 whenever the permutation touches the contiguous (innermost) dimension, because vectorized loads/stores require the access pattern to remain contiguous.

**CN**: Doxygen 块解释了核心概念：置换布局函数是注入 CUTLASS `GemmUniversal` 内核的*插件*。它们不是后处理步骤——每个输出元素的地址计算由布局的 `operator()` 就地修改。强调了两个关键提示：
- 对于批处理模式置换（BMM），`batch_stride` 必须为**零**，以便置换布局完全控制批次间的寻址。
- 当置换操作涉及连续（最内层）维度时，**对齐**必须降低为 1，因为向量化加载/存储要求访问模式保持连续。

---

### Lines 68–102 — Includes

```cpp
      $ nv-nsight-cu-cli ./examples/39_gemm_permute/39_gemm_permute --m=256 --n=192 --k=256 --verbose=true --iterations=1 --reference-check=false

*/

/////////////////////////////////////////////////////////////////////////////////////////////////

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <map>
#include <unordered_map>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/device/gemm_universal.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/host/gemm_complex.h"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_norm.h"

#include "cutlass/layout/permute.h"

#include "layouts.h"
#include "permute_info.h"
```
**EN**: The includes fall into four groups:
- **STL headers**: `<iostream>`, `<vector>`, `<map>`, etc. for host-side bookkeeping.
- **CUTLASS core**: `cutlass.h` (basic types/macros), `gemm_universal.h` (the device GEMM template), `permute.h` (all permute layout classes).
- **CUTLASS utility**: device memory helpers, `TensorView` I/O, reference GEMM implementations (both host and device), tensor fill/compare utilities.
- **Local headers**: `layouts.h` (custom 4-D/5-D reference layouts) and `permute_info.h` (compile-time permute metadata traits).

**CN**: 包含文件分为四组：
- **STL 头文件**：`<iostream>`、`<vector>`、`<map>` 等用于主机端记账。
- **CUTLASS 核心**：`cutlass.h`（基本类型/宏）、`gemm_universal.h`（设备 GEMM 模板）、`permute.h`（所有置换布局类）。
- **CUTLASS 工具**：设备内存辅助工具、`TensorView` I/O、参考 GEMM 实现（主机端和设备端均有）、张量填充/比较工具。
- **本地头文件**：`layouts.h`（自定义 4-D/5-D 参考布局）和 `permute_info.h`（编译期置换元数据特征）。

---

### Lines 103–132 — Global Compile-Time Constants & Element Types

```cpp
/// Tensor4DPermuteBMM0213 --->
/// Permute layout function for 4-D permuted tensors for BMM with BMM tensor (dimension as [B, M, N]) reshaped
/// as [B/D1, D1, M, N]. Then perform permute([0, 2, 1, 3]) on the corresponding whole BMM tensor.
int constexpr D1 = 12;

/// Tensor5DPermute20314 --->
/// Permute layout function for 5-D permuted tensors with matrix (dimension as [M, N]) reshaped
/// as [M/T1, T1, T2, T3, N/T2/T3]. Then perform permute([2, 0, 3, 1, 4]) on the corresponding tensor.
int constexpr T1 = 16; 
int constexpr T2 = 3;
int constexpr T3 = 8;

/// Tensor4DPermute0213 --->
/// Permute layout function for 4-D permuted tensors with matrix (dimension as [M, N]) reshaped
/// as [M/S1, S1, S2, N/S2]. Then perform permute([0, 2, 1, 3]) on the corresponding tensor.
int constexpr S1 = 8; 
int constexpr S2 = 4;

// // // Alignments
int constexpr AlignmentA = 8;
int constexpr AlignmentB = 8;
int constexpr AlignmentC = 8;

/// GEMM element types
using ElementInput = cutlass::half_t;
using ElementOutput = cutlass::half_t;
using ElementAccumulator = float;

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: These constants are the **only place** where permute tile sizes are configured. Because the CUTLASS permute layouts are parameterized by compile-time integers, changing the problem decomposition requires recompilation.

- `D1=12`: the batch dimension is split as `[B/12, 12, M, N]`.
- `T1=16, T2=3, T3=8`: the 5-D permute factorizes M as `M/16` and N as `N/(3×8)=N/24`.
- `S1=8, S2=4`: the 4-D normal GEMM permute factorizes M as `M/8` and N as `N/4`.
- All alignments are 8 elements (one 128-bit vector for `half_t`), which is valid because the default permutations in this example preserve the unit-stride dimension.
- `ElementAccumulator = float`: FP32 accumulation is used regardless of FP16 inputs/outputs to maintain numerical precision in the Tensor Core pipeline.

**CN**: 这些常量是置换分块大小的**唯一配置位置**。由于 CUTLASS 置换布局由编译期整数参数化，更改问题分解需要重新编译。

- `D1=12`：批次维度被分解为 `[B/12, 12, M, N]`。
- `T1=16, T2=3, T3=8`：5-D 置换将 M 因式分解为 `M/16`，N 为 `N/(3×8)=N/24`。
- `S1=8, S2=4`：4-D 普通 GEMM 置换将 M 因式分解为 `M/8`，N 为 `N/4`。
- 所有对齐均为 8 个元素（`half_t` 的一个 128 位向量），有效是因为本示例中的默认置换保留了单位步长维度。
- `ElementAccumulator = float`：无论输入/输出为 FP16，均使用 FP32 累加，以在张量核心流水线中保持数值精度。

---

### Lines 133–154 — Error-Checking Macros

```cpp
/// Useful macros

#define CHECK_CUDA_CALL(call, handler) \
do { \
  cudaError_t __err = (call); \
  if (__err != cudaSuccess) { \
    std::cerr << #call " failed: " << cudaGetErrorString(__err) << std::endl; \
    handler; \
  } \
} while(0)

#define CHECK_CUTLASS_CALL(call, handler) \
do { \
  cutlass::Status __status = (call); \
  if (__status != cutlass::Status::kSuccess) { \
    std::cerr << #call " failed: " << cutlass::cutlassGetStatusString(__status) << std::endl; \
    handler; \
  } \
} while(0)

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Two thin wrapper macros that capture the call expression via `#call` for the error message. Both accept a `handler` expression (typically `return false`) so the error policy can be set at the call site. `do { ... } while(0)` is the standard macro idiom to make them safe in all syntactic contexts (e.g., inside an `if` without braces). `cutlass::cutlassGetStatusString` converts CUTLASS status codes to human-readable strings.

**CN**: 两个轻量级包装宏，通过 `#call` 在错误消息中捕获调用表达式。两者都接受 `handler` 表达式（通常为 `return false`），以便在调用处设置错误策略。`do { ... } while(0)` 是标准宏惯用法，在所有语法上下文中使其安全（例如，在没有大括号的 `if` 内）。`cutlass::cutlassGetStatusString` 将 CUTLASS 状态码转换为可读字符串。

---

### Lines 155–285 — `struct Options` — Command-Line Parsing

```cpp
// Command line options parsing
struct Options {

  bool help;
  bool error;
  bool reference_check;

  cutlass::gemm::GemmCoord problem_each;

  int batch_count;
  int iterations;
  int cuda_streams;
  bool verbose;
  float alpha;
  float beta;

  //
  // Methods
  // 

  Options():
    help(false),
    error(false),
    reference_check(true),
    batch_count(-1),
    iterations(20),
    cuda_streams(0),
    verbose(false),
    alpha(1),
    beta()
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("alpha", alpha, 1.0f);
    cmd.get_cmd_line_argument("beta", beta, 0.0f);    
    cmd.get_cmd_line_argument("iterations", iterations, 20);
    cmd.get_cmd_line_argument("streams", cuda_streams, 0);
    cmd.get_cmd_line_argument("verbose", verbose, false);
    cmd.get_cmd_line_argument("reference-check", reference_check, true);

    int m, n, k;

    cmd.get_cmd_line_argument("m", m, 384);
    cmd.get_cmd_line_argument("n", n, 192);
    cmd.get_cmd_line_argument("k", k, 384);
    cmd.get_cmd_line_argument("batch-count", batch_count, 96);

    problem_each = cutlass::gemm::GemmCoord(m, n, k);
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << 
      "39_gemm_permute\n"
      "\n"
      " This example tests and profiles the performance of normal GEMM and batched GEMM with different"
      " combinations of fused permutations of input and output tensors."
      "\n"
      " Permutations considered in this example:\n"
      "\n"
      " Normal GEMM:\n"
      " 1) Tensor4DPermute0213: matrix of shape [X, Y] is reshaped as [X/S1, S1, S2, Y/S2] and has its dimensions"
      " permuted as [0, 2, 1, 3], resulting in shape [X/S1, S2, S1, Y/S2] viewed as matrix of shape [X*S2/S1, Y*S1/S2].\n"
      " 2) Tensor5DPermute20314: matrix of shape [X, Y] is reshaped as [X/T1, T1, T2, T3, Y/T2/T3] and has its dimensions"
      " permuted as [2, 0, 3, 1, 4], resulting in shape [T2, X/T1, T3, T1, Y/T2/T3] viewed as matrix of shape [X*T2/T1, Y*T1/T2].\n"
       "\n"
      " Batched GEMM:\n"
      " 3) Tensor4DPermuteBMM0213: batched tensor of 3D shape [B, X, Y] is reshaped as 4D shape [B/D1, D1, X, Y]"
      " and has its dimensions permuted as [0, 2, 1, 3], resulting in shape [B/D1, X, D1, Y] viewed as"
      " a matrix of shape [B/D1, X, Y*D1] for batched GEMM purposes.\n"
      "\n"
      " Note: S1, S2, D1, D2, T1, T2, T3 are compile-time constants defined in gemm_permute.cu."
      " Runtime specification of these values is not supported."
      " These values along with alignment requirements place constraints on supported matrix sizes.\n"
      "\n"
      " Note: X, Y above may refer to M, N or K dimensions of GEMM problem, depending on the tensor considered (A, B or D)."
      " For the output tensor D the values correspond directly to dimensions of D, whereas for A and B the original dimensions"
      " X', Y' are inferred from the ones supplied to the GEMM, taking into account the permute operation.\n"
      "\n"
      "Options:\n"
      "\n"
      "  --help                      If specified, displays this usage statement.\n\n"
      "  --batch-count=<int>         Sets the number of batches in batched GEMM (batch number for BMM). (default: --batch-count=768)\n"
      "  --m=<int>                   Sets the M dimension for both batched GEMM and normal GEMM problems. (default: --m=128)\n"
      "  --n=<int>                   Sets the N dimension for both batched GEMM and normal GEMM problems. (default: --n=192)\n"
      "  --k=<int>                   Sets the K dimension for both batched GEMM and normal GEMM problems. (default: --k=384)\n"
      "  --alpha=<f32>               Epilogue scalar alpha (real part)\n"
      "  --beta=<f32>                Epilogue scalar beta (real part)\n\n"
      "  --iterations=<int>          Number of profiling iterations to perform.\n"
      "  --reference-check=<bool>    If true, performs reference check.\n"
      "  --verbose=<bool>            If true, prints problem sizes and batching structure.\n"
      "\n"
      "Examples:\n"
      "\n"
      "# Runs a batched GEMM with 96 batches\n"
      "$ ./examples/39_gemm_permute/39_gemm_permute --batch-count=96\n"
      "\n"
      "# Runs a batched GEMM with 96 batches (with GEMM-K dimension equal to 1024)\n"
      "$ ./examples/39_gemm_permute/39_gemm_permute --batch-count=96 --k=1024 --verbose=true\n"
      "\n"
      "# Execute batched GEMM and profile with NSight\n"
      "$ nv-nsight-cu-cli ./examples/39_gemm_permute/39_gemm_permute --m=256 --n=192 --k=256 --verbose=true --iterations=1 --reference-check=false\n"
      "\n";

    return out;
  }

  /// Compute performance in GFLOP/s
  double gflops(double runtime_s, bool batched) const {

    // Number of real-valued multiply-adds 
    int64_t fmas = int64_t();

    fmas += problem_each.product() * (batched ? batch_count : 1);
    
    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: `Options` uses CUTLASS's `CommandLine` utility (a minimal argument parser) to populate problem dimensions, batch count, epilogue scalars, and flags. `GemmCoord` stores `(M, N, K)` as a typed 3-D coordinate. The defaults — M=384, N=192, K=384, batch=96 — were chosen to be compatible with all the compile-time tile factors (384 % 16 = 0, 384 % 8 = 0, 192 % 24 = 0, 96 % 12 = 0, etc.). `gflops()` computes throughput as `2 × M × N × K × batch / time`, the standard FLOP count for a multiply-add operation.

**CN**: `Options` 使用 CUTLASS 的 `CommandLine` 工具（一个轻量级参数解析器）来填充问题维度、批次数量、尾声标量和标志位。`GemmCoord` 将 `(M, N, K)` 存储为类型化的 3-D 坐标。默认值——M=384、N=192、K=384、batch=96——被选择为与所有编译期分块因子兼容（384 % 16 = 0, 384 % 8 = 0, 192 % 24 = 0, 96 % 12 = 0 等）。`gflops()` 按 `2 × M × N × K × batch / time` 计算吞吐量，这是乘加运算的标准 FLOP 计数。

---

### Lines 286–338 — `permute_host_impl` & `permute_host` — CPU Reference Permutation

```cpp
namespace { // (anonymous)

/// Dimension-generic permutation loop
template<int I, typename Element, typename Layout, typename PermuteOp, typename Coord>
void permute_host_impl(
    cutlass::TensorView<Element const, Layout> const & input,
    cutlass::TensorView<Element, Layout> const & output,
    PermuteOp && permute,
    Coord & coord
) {
  static_assert(Layout::kRank == Coord::kRank, "Incompatible Layout and Coord types");
  if constexpr (I == Coord::kRank) {
    output.at(permute(coord)) = input.at(coord);
  }
  else {
    for (coord[I] = 0; coord[I] < input.extent(I); ++coord[I]) {
      permute_host_impl<I+1>(input, output, std::forward<PermuteOp>(permute), coord);
    }
  }
}

} // namespace (anonymous)

/// Perform a reference (host-based) permutation of an input tensor
template<typename PermuteLayout, typename Element, typename Layout>
void permute_host(
    cutlass::TensorView<Element const, Layout> const &input,
    cutlass::TensorView<Element, Layout> const &output,
    int batch_count) {
  Layout layout = input.layout();
  cutlass::MatrixCoord extent = input.extent();

  std::size_t num_elems = layout.capacity(extent) * batch_count;
  std::vector<Element> h_input(num_elems);
  cutlass::device_memory::copy_to_host(h_input.data(), input.data(), num_elems);

  std::vector<Element> h_output(num_elems);

  using Info = PermuteInfo<PermuteLayout>;
  using TensorLayout = typename Info::Layout;

  auto shape_orig = Info::original_shape(extent, batch_count);
  auto shape_perm = Info::permute(shape_orig);

  cutlass::TensorView<Element const, TensorLayout> view_input(h_input.data(), TensorLayout::packed(shape_orig), shape_orig); 
  cutlass::TensorView<Element, TensorLayout> view_output(h_output.data(), TensorLayout::packed(shape_perm), shape_perm);

  decltype(shape_orig) coord;
  permute_host_impl<0>(view_input, view_output, Info::permute, coord);

  cutlass::device_memory::copy_to_device(output.data(), h_output.data(), num_elems);
}
```
**EN**: This is the **golden reference** implementation used to verify CUTLASS GPU results.

`permute_host_impl` is a compile-time recursive loop using `if constexpr`. At depth `I < kRank` it iterates over dimension `I` of the input coordinate, recursing to depth `I+1`. At `I == kRank` (the base case) it writes `input.at(coord)` to `output.at(permute(coord))`, applying the coordinate permutation via the `PermuteOp` functor.

`permute_host` orchestrates the full flow:
1. Copies device data to host vectors (`h_input`, `h_output`).
2. Uses `PermuteInfo<PermuteLayout>` to determine the original higher-rank tensor shape and its permuted shape.
3. Wraps raw host pointers in `cutlass::TensorView` objects with tightly-packed layouts (`TensorLayout::packed()`).
4. Calls `permute_host_impl<0>` to iterate all coordinates.
5. Copies the result back to the device output buffer.

`TensorView` semantics: a non-owning view pairing a pointer, a layout object, and an extent. `view.at(coord)` computes `ptr + layout(coord)` and dereferences it. Tightly packed means `capacity == product(extents)`.

**CN**: 这是用于验证 CUTLASS GPU 结果的**黄金参考**实现。

`permute_host_impl` 是使用 `if constexpr` 的编译期递归循环。在深度 `I < kRank` 时，它遍历输入坐标的维度 `I`，递归到深度 `I+1`。在 `I == kRank`（基本情况）时，它通过 `PermuteOp` 仿函数应用坐标置换，将 `input.at(coord)` 写入 `output.at(permute(coord))`。

`permute_host` 编排完整流程：
1. 将设备数据复制到主机向量（`h_input`、`h_output`）。
2. 使用 `PermuteInfo<PermuteLayout>` 确定原始高秩张量形状及其置换后的形状。
3. 使用紧密打包布局（`TensorLayout::packed()`）将原始主机指针包装在 `cutlass::TensorView` 对象中。
4. 调用 `permute_host_impl<0>` 遍历所有坐标。
5. 将结果复制回设备输出缓冲区。

`TensorView` 语义：将指针、布局对象和范围配对的非拥有视图。`view.at(coord)` 计算 `ptr + layout(coord)` 并解引用它。紧密打包意味着 `capacity == product(extents)`。

---

### Lines 339–353 — `LayoutInfo` Trait

```cpp
template<typename Layout>
struct LayoutInfo;

template<>
struct LayoutInfo<cutlass::layout::RowMajor> {
  static std::string name() { return "RowMajor"; }
};

template<>
struct LayoutInfo<cutlass::layout::ColumnMajor> {
  static std::string name() { return "ColumnMajor"; }
};

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: A minimal two-specialization traits struct for printing layout names to `std::cout`. Unlike `PermuteInfo`, the primary template is complete (but empty), so using an unsupported layout would compile but produce no name output. These names appear in the header line printed before each test: `"A=RowMajor,Tensor4DPermute0213<8,4>"`.

**CN**: 用于将布局名称打印到 `std::cout` 的最小双特化特征结构体。与 `PermuteInfo` 不同，主模板是完整的（但为空），因此使用不支持的布局会编译成功但不产生名称输出。这些名称出现在每次测试前打印的标题行中：`"A=RowMajor,Tensor4DPermute0213<8,4>"`。

---

### Lines 354–391 — `class Testbed` — Data Members & Constructor

```cpp
template <typename ElementA, typename ElementB, typename ElementC>
class Testbed {
private:

  //
  // Data members
  //

  Options & options;

  /// Initialization
  cutlass::Distribution::Kind init_A;
  cutlass::Distribution::Kind init_B;
  cutlass::Distribution::Kind init_C;
  uint32_t seed;

  cutlass::DeviceAllocation<ElementA> block_A;
  cutlass::DeviceAllocation<ElementB> block_B;
  cutlass::DeviceAllocation<ElementC> block_C;
  cutlass::DeviceAllocation<ElementC> block_D;

public:

  //
  // Methods
  //

  Testbed(
    Options &options_,
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform,
    uint32_t seed_ = 3090
  ):
    options(options_), init_A(init_A_), init_B(init_B_), init_C(init_C_), seed(seed_) { }

private:
```
**EN**: `Testbed` is parameterized by element types for each of the three matrices (A, B, C/D). It holds a reference to `Options` (not a copy) to avoid re-parsing. The four `DeviceAllocation` members are RAII wrappers around `cudaMalloc`/`cudaFree` that own GPU memory for the A, B, C, and D matrices across all test runs. The distribution kind enumerates Uniform/Gaussian/Sequential fill patterns used in `initialize_tensor_`.

**CN**: `Testbed` 按三个矩阵（A、B、C/D）各自的元素类型参数化。它持有对 `Options` 的引用（而非副本）以避免重复解析。四个 `DeviceAllocation` 成员是 `cudaMalloc`/`cudaFree` 的 RAII 包装器，在所有测试运行中拥有 A、B、C 和 D 矩阵的 GPU 内存。分布类型枚举 `initialize_tensor_` 中使用的 Uniform/Gaussian/Sequential 填充模式。

---

### Lines 392–416 — `print_tensor_info` (Verbose Diagnostic)

```cpp
  /// Print permutation info for one tensor
  template<typename PermuteLayout>
  void print_tensor_info(
      std::ostream & os,
      std::string const &tensor_name,
      int row_dim,
      int col_dim) {

    cutlass::MatrixCoord extent(options.problem_each.at(row_dim), options.problem_each.at(col_dim));
    using Info = PermuteInfo<PermuteLayout>;

    os << "tensor " << tensor_name << ": " << Info::desc() << "\n";
    os << "    extent: [" << extent.row() << ", " << extent.column() << "]";
    if (Info::kBatched) {
      os << ", batch count: " << options.batch_count;
    }
    os << "\n";
    if (!cutlass::layout::is_trivial_permute<PermuteLayout>) {
      auto shape_orig = Info::original_shape(extent, options.batch_count);
      auto shape_perm = Info::permute(shape_orig);
      os << "  original: [" << shape_orig << "]\n";
      os << "  permuted: [" << shape_perm << "]\n";
    }
  }
```
**EN**: Prints the tensor's logical extent, optionally the batch count for BMM tensors, and (for non-trivial permutations) both the original and permuted higher-rank shapes. `cutlass::layout::is_trivial_permute<P>` is a compile-time boolean trait that is `true` only for `NoPermute`, preventing unnecessary coordinate printing. `row_dim` and `col_dim` index into the `GemmCoord` (0=M, 1=N, 2=K), so tensor A uses `(row=0, col=2)` = `(M, K)` and B uses `(row=2, col=1)` = `(K, N)`.

**CN**: 打印张量的逻辑范围，可选地打印 BMM 张量的批次数，以及（对于非平凡置换）原始和置换后的高秩形状。`cutlass::layout::is_trivial_permute<P>` 是一个编译期布尔特征，仅对 `NoPermute` 为 `true`，防止不必要的坐标打印。`row_dim` 和 `col_dim` 索引到 `GemmCoord`（0=M, 1=N, 2=K），因此张量 A 使用 `(row=0, col=2)` = `(M, K)`，B 使用 `(row=2, col=1)` = `(K, N)`。

---

### Lines 417–460 — `check_tensor_shape` — Shape Compatibility Validation

```cpp
  /// Check shape compatibility for one tensor
  template<typename Layout, typename PermuteLayout, int Alignment>
  bool check_tensor_shape(
      std::string const &tensor_name,
      int row_dim,
      int col_dim) {

    cutlass::MatrixCoord extent(options.problem_each.at(row_dim), options.problem_each.at(col_dim));

    using Info = PermuteInfo<PermuteLayout>;

    auto rowAlign = cutlass::platform::is_same<Layout, cutlass::layout::ColumnMajor>::value ? Alignment : 1;
    auto colAlign = cutlass::platform::is_same<Layout, cutlass::layout::RowMajor>::value ? Alignment : 1;

    auto rowFactor = Info::kRowFactor * rowAlign;
    auto colFactor = Info::kColumnFactor * colAlign;

    // Assumes row-major layout
    bool const valid_row = extent.row() % rowFactor == 0;
    if (!valid_row) {
      std::cerr << "Invalid tensor " << tensor_name << " row size = " << extent.row() << ", "
                   "must be divisible by " << rowFactor << ", "
                   "required by " << Info::name() << 
                   (rowAlign > 1 ? (" and alignment of " + std::to_string(rowAlign)) : "") << std::endl;
    }

    bool const valid_col = extent.column() % colFactor == 0;
    if (!valid_col) {
      std::cerr << "Invalid tensor " << tensor_name << " column size = " << extent.column() << ", "
                   "must be divisible by " << colFactor << ", "
                   "required by " << Info::name() << 
                   (colAlign > 1 ? (" and alignment of " + std::to_string(colAlign)) : "") << std::endl;
    }

    bool const valid_bsz = options.batch_count % Info::kBatchFactor == 0;
    if (!valid_bsz) {
      std::cerr << "Invalid batch count = " << options.batch_count << ", "
                   "must be divisible by " << Info::kBatchFactor << ", "
                   "required by " << Info::name() << std::endl;
    }

    return valid_row && valid_col && valid_bsz;
  }
```
**EN**: Before running a test, this validates that all three divisibility requirements are met:
1. **Row alignment**: for a column-major layout the row dimension is the contiguous one so the hardware alignment requirement applies to it; for row-major the row dimension is strided, so only the permute factor applies.
2. **Column alignment**: the mirror of the above.
3. **Batch factor**: the batch count must be divisible by `kBatchFactor` (relevant only for BMM permutes where the batch axis is folded into the permuted shape).

Rather than aborting, failed checks print an error and return `false`, causing `profile_GEMM_permute` to skip the test gracefully.

**CN**: 在运行测试之前，验证三个整除性要求：
1. **行对齐**：对于列主序布局，行维度是连续的，因此硬件对齐要求适用于它；对于行主序，行维度是跨步的，因此只有置换因子适用。
2. **列对齐**：上述情况的镜像。
3. **批次因子**：批次数必须能被 `kBatchFactor` 整除（仅与批次轴被折叠进置换形状的 BMM 置换相关）。

失败的检查不会中止程序，而是打印错误并返回 `false`，使 `profile_GEMM_permute` 优雅地跳过测试。

---

### Lines 461–516 — `initialize_tensor_` — Device Fill Dispatch

```cpp
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
**EN**: Dispatches to one of four device-side fill kernels based on the requested distribution. The uniform range is scaled adaptively: narrow ranges (`[-2,2]`) for low-bit-width integers to avoid overflow, wider ranges (`[-8,8]`) for FP16 with FP32 accumulators. This adaptive scaling prevents accumulator overflow in the reference GEMM while still exercising a meaningful value range. All fills use GPU kernels directly (no host-side data generation + copy), keeping initialization fast even for large tensors.

**CN**: 根据请求的分布类型分发到四个设备端填充核函数之一。均匀范围自适应缩放：低位宽整数使用窄范围（`[-2,2]`）以避免溢出，FP16（FP32 累加器）使用更宽的范围（`[-8,8]`）。这种自适应缩放防止参考 GEMM 中的累加器溢出，同时仍然测试有意义的值范围。所有填充直接使用 GPU 核函数（无主机端数据生成 + 复制），即使对于大张量也保持初始化速度。

---

### Lines 517–542 — `initialize` — Buffer Allocation & Fill

```cpp
  /// Initializes data structures
  void initialize(int batch_count) {

    srand(seed);

    int64_t total_elements_A = options.problem_each.m() * options.problem_each.k() * batch_count;
    int64_t total_elements_B = options.problem_each.n() * options.problem_each.k() * batch_count;
    int64_t total_elements_C = options.problem_each.m() * options.problem_each.n() * batch_count;
    int64_t total_elements_D = options.problem_each.m() * options.problem_each.n() * batch_count;

    // Allocate space
    block_A.reset(total_elements_A);
    block_B.reset(total_elements_B);
    block_C.reset(total_elements_C);
    block_D.reset(total_elements_D);

    // Initialize input tensors
    initialize_tensor_(block_A.get(), total_elements_A, init_A, seed * 2021);
    initialize_tensor_(block_B.get(), total_elements_B, init_B, seed * 2022);
    initialize_tensor_(block_C.get(), total_elements_C, init_C, seed * 2023);

    cutlass::reference::device::BlockFillSequential(
      block_D.get(), total_elements_D, ElementC(), ElementC());
  }
```
**EN**: Allocates GPU memory for all four matrices using the product of problem dimensions × batch count as the flat allocation size. Note that all batches share a contiguous allocation with no gap — CUTLASS's `GemmUniversal` in kBatched mode uses `batch_stride` to jump between batches, and setting `batch_stride=0` for permuted tensors tells the kernel that the permute layout function handles inter-batch addressing. D is zero-initialized (sequential fill with step 0) to detect any unwritten elements.

**CN**: 使用问题维度 × 批次数之积作为平坦分配大小，为所有四个矩阵分配 GPU 内存。注意所有批次共享一个没有间隙的连续分配——CUTLASS 的 `GemmUniversal` 在 kBatched 模式下使用 `batch_stride` 在批次间跳转，对置换张量设置 `batch_stride=0` 告知内核置换布局函数处理批次间的寻址。D 被零初始化（步长为 0 的顺序填充）以检测任何未写入的元素。

---

### Lines 543–622 — `validate` — Reference Check

```cpp
  /// Check device GEMM results against a reference implementation with separate host-based permutation
  template<typename Gemm>
  bool validate(Gemm const &gemm) {

    bool constexpr kBatched = PermuteInfo<typename Gemm::PermuteALayout>::kBatched 
                           || PermuteInfo<typename Gemm::PermuteBLayout>::kBatched 
                           || PermuteInfo<typename Gemm::PermuteDLayout>::kBatched;
                      
    int const batch_count = kBatched ? options.batch_count : 1;

    cutlass::gemm::GemmCoord problem = options.problem_each;

    cutlass::MatrixCoord extent_A{problem.m(), problem.k()};
    cutlass::MatrixCoord extent_B{problem.k(), problem.n()};
    cutlass::MatrixCoord extent_C{problem.m(), problem.n()};

    using LayoutA = typename Gemm::LayoutA;
    using LayoutB = typename Gemm::LayoutB;
    using LayoutC = typename Gemm::LayoutC;

    LayoutA layout_A(LayoutA::packed(extent_A));
    LayoutB layout_B(LayoutB::packed(extent_B));
    LayoutC layout_C(LayoutC::packed(extent_C));

    auto size_A = layout_A.capacity(extent_A) * batch_count;
    auto size_B = layout_B.capacity(extent_B) * batch_count;
    auto size_C = layout_C.capacity(extent_C) * batch_count;
    
    cutlass::TensorView<ElementA, LayoutA> view_A(block_A.get(), layout_A, extent_A);
    cutlass::TensorView<ElementB, LayoutB> view_B(block_B.get(), layout_B, extent_B);
    cutlass::TensorView<ElementC, LayoutC> view_C(block_C.get(), layout_C, extent_C);
    cutlass::TensorView<ElementC, LayoutC> view_D(block_D.get(), layout_C, extent_C);

    cutlass::DeviceAllocation<ElementA> block_A_perm(size_A);
    cutlass::DeviceAllocation<ElementA> block_B_perm(size_B);

    cutlass::TensorView<ElementA, LayoutA> view_A_perm(block_A_perm.get(), layout_A, extent_A);
    cutlass::TensorView<ElementB, LayoutB> view_B_perm(block_B_perm.get(), layout_B, extent_B);

    permute_host<typename Gemm::PermuteALayout>(view_A.const_view(), view_A_perm, batch_count);
    permute_host<typename Gemm::PermuteBLayout>(view_B.const_view(), view_B_perm, batch_count);

    cutlass::DeviceAllocation<ElementC>    block_D_ref(size_C);
    cutlass::TensorView<ElementC, LayoutC> view_D_ref(block_D_ref.get(), layout_C, extent_C);

    using EpilogueOutputOp = typename Gemm::GemmKernel::Epilogue::OutputOp;

    // Reference GEMM
    cutlass::reference::device::GemmComplex<
        ElementA, LayoutA,
        ElementB, LayoutB,
        ElementC, LayoutC, 
        typename EpilogueOutputOp::ElementCompute,
        typename Gemm::ElementAccumulator
    >(
      problem,
      options.alpha, 
      view_A_perm,
      Gemm::kTransformA,
      view_B_perm,
      Gemm::kTransformB,
      options.beta, 
      view_C, 
      view_D_ref, 
      ElementAccumulator(0),
      batch_count,
      options.problem_each.m() * options.problem_each.k(),
      options.problem_each.n() * options.problem_each.k(),
      options.problem_each.m() * options.problem_each.n(),
      options.problem_each.m() * options.problem_each.n()
    );

    cutlass::DeviceAllocation<ElementC>    block_D_perm(size_C);
    cutlass::TensorView<ElementC, LayoutC> view_D_perm(block_D_perm.get(), layout_C, extent_C);
    permute_host<typename Gemm::PermuteDLayout>(view_D_ref.const_view(), view_D_perm, batch_count);

    // Reference check
    return cutlass::reference::device::BlockCompareEqual(view_D_perm.data(), view_D.data(), size_C);
}
```
**EN**: The validation flow mirrors what the GPU kernel does, but using separate host-side steps:
1. **Permute inputs**: `permute_host<PermuteALayout>` applies the *input* permutation to A (and B). For an *inverse* permute on A, this pre-permutes A so that after the kernel's inverse permute the data lands in the correct GEMM order.
2. **Reference GEMM**: runs a device-side reference GEMM on the permuted A/B with standard batch strides.
3. **Permute output**: `permute_host<PermuteDLayout>` applies the *output* permutation to the reference D.
4. **Compare**: `BlockCompareEqual` does element-wise comparison on the device to avoid PCIe bandwidth cost.

This 3-step approach correctly models any combination of input and output permutations.

**CN**: 验证流程反映了 GPU 内核所做的工作，但使用单独的主机端步骤：
1. **置换输入**：`permute_host<PermuteALayout>` 对 A（和 B）应用*输入*置换。对于 A 上的*逆*置换，这会预置换 A，使得内核的逆置换后数据落在正确的 GEMM 顺序中。
2. **参考 GEMM**：在置换后的 A/B 上使用标准批次步长运行设备端参考 GEMM。
3. **置换输出**：`permute_host<PermuteDLayout>` 对参考 D 应用*输出*置换。
4. **比较**：`BlockCompareEqual` 在设备上进行逐元素比较，以避免 PCIe 带宽成本。

这种 3 步方法正确地模拟了输入和输出置换的任意组合。

---

### Lines 623–773 — `profile_GEMM_permute` — GEMM Invocation & Profiling

```cpp
public:

  template<typename Gemm>
  bool profile_GEMM_permute() {

    using LayoutA = typename Gemm::LayoutA;
    using LayoutB = typename Gemm::LayoutB;
    using LayoutC = typename Gemm::LayoutC;

    using PermuteALayout = typename Gemm::PermuteALayout;
    using PermuteBLayout = typename Gemm::PermuteBLayout;
    using PermuteDLayout = typename Gemm::PermuteDLayout;

    bool constexpr kBatched = PermuteInfo<PermuteALayout>::kBatched 
                           || PermuteInfo<PermuteBLayout>::kBatched 
                           || PermuteInfo<PermuteDLayout>::kBatched;

    std::cout << "\n"
                 "====================================================\n"
                 << (kBatched ? "Batched" : "Normal") << " GEMM:"
                 << "\n  A=" << LayoutInfo<LayoutA>::name() << "," << PermuteInfo<PermuteALayout>::name()
                 << "\n  B=" << LayoutInfo<LayoutB>::name() << "," << PermuteInfo<PermuteBLayout>::name()
                 << "\n  D=" << LayoutInfo<LayoutC>::name() << "," << PermuteInfo<PermuteDLayout>::name()
                 << "\n"
                 "====================================================\n";

    if (options.verbose) {
      print_tensor_info<PermuteALayout>(std::cout, "A", 0, 2);
      print_tensor_info<PermuteBLayout>(std::cout, "B", 2, 1);
      print_tensor_info<PermuteDLayout>(std::cout, "D", 0, 1);
    }
    std::cout << std::endl;

    bool valid = true;
    valid &= check_tensor_shape<LayoutA, PermuteALayout, Gemm::kAlignmentA>("A", 0, 2);
    valid &= check_tensor_shape<LayoutB, PermuteBLayout, Gemm::kAlignmentB>("B", 2, 1);
    valid &= check_tensor_shape<LayoutC, PermuteDLayout, Gemm::kAlignmentC>("D", 0, 1);
    if (!valid)
    {
      std::cout << "Skipped test" << std::endl;
      return true;
    }

    int const batch_count = kBatched ? options.batch_count : 1;

    // Initialize the problem
    initialize(batch_count);

    // Configure the GEMM arguments
    using EpilogueOutputOp = typename Gemm::GemmKernel::Epilogue::OutputOp;
    typename EpilogueOutputOp::Params epilogue_op(options.alpha, options.beta);

    // Please make sure all problem_sizes are the same for kBatched mode
    auto problem = options.problem_each;

    cutlass::MatrixCoord extent_A{problem.m(), problem.k()};
    cutlass::MatrixCoord extent_B{problem.k(), problem.n()};
    cutlass::MatrixCoord extent_C{problem.m(), problem.n()};

    LayoutA layout_A(LayoutA::packed(extent_A));
    LayoutB layout_B(LayoutB::packed(extent_B));
    LayoutC layout_C(LayoutC::packed(extent_C));

    // Configure GEMM arguments
    typename Gemm::Arguments arguments{
      kBatched ? cutlass::gemm::GemmUniversalMode::kBatched : cutlass::gemm::GemmUniversalMode::kGemm,
      problem,
      batch_count,
      epilogue_op,
      (void*)block_A.get(),
      (void*)block_B.get(),
      (void*)block_C.get(),
      (void*)block_D.get(),
      // For any non-trivial permute the batch stride must be set to 0
      cutlass::layout::is_trivial_permute<PermuteALayout> ? layout_A.capacity(extent_A) : 0,
      cutlass::layout::is_trivial_permute<PermuteBLayout> ? layout_B.capacity(extent_B) : 0,
      layout_C.capacity(extent_C),
      cutlass::layout::is_trivial_permute<PermuteDLayout> ? layout_C.capacity(extent_C) : 0,
      layout_A.stride(0),
      layout_B.stride(0),
      layout_C.stride(0),
      layout_C.stride(0),
    };

    // Initialize the GEMM object
    Gemm gemm_normal;

    CHECK_CUTLASS_CALL(gemm_normal.initialize(arguments, nullptr), return false);

    // Run the normal GEMM object
    CHECK_CUTLASS_CALL(gemm_normal.run(), return false);

    // Wait for completion
    CHECK_CUDA_CALL(cudaDeviceSynchronize(), return false);

    //
    // Verify correctness
    //
    if (options.reference_check) {
      if (validate(gemm_normal)) {
        std::cout << "\nPassed verification\n" << std::endl;
      }
      else {
        std::cerr << "\n*** Error - problem failed the QA check ***\n" << std::endl;
        return false;
      }
    }

    // Warm-up run of the normal GEMM object
    CHECK_CUTLASS_CALL(gemm_normal.run(), return false);

    // Construct events
    cudaEvent_t events[2];
    for (auto & event : events) {
      CHECK_CUDA_CALL(cudaEventCreate(&event), return false);
    }

    // Record an event at the start of a series of GEMM operations
    CHECK_CUDA_CALL(cudaEventRecord(events[0]), return false);

    // Run profiling loop
    for (int iter = 0; iter < options.iterations; ++iter) {
      gemm_normal();
    }

    // Record an event when the GEMM operations have been launched.
    CHECK_CUDA_CALL(cudaEventRecord(events[1]), return false);

    // Wait for work on the device to complete.
    CHECK_CUDA_CALL(cudaEventSynchronize(events[1]), return false);

    // Measure elapsed runtime
    float runtime_total_ms = 0;
    CHECK_CUDA_CALL(cudaEventElapsedTime(&runtime_total_ms, events[0], events[1]), return false);

    // Compute average runtime and GFLOPs.
    double runtime_avg_ms = double(runtime_total_ms) / double(options.iterations);
    double gflops = options.gflops(runtime_avg_ms / 1000.0, kBatched);

    // Cleanup
    for (auto event : events) {
      CHECK_CUDA_CALL(cudaEventDestroy(event), return false);
    }

    std::cout << "    Runtime: " << runtime_avg_ms << " ms\n"
                 "     GFLOPs: " << gflops << std::endl;

    return true;
  }
};
```
**EN**: The core profiling method. Key design points:

**Batch stride = 0 rule**: For any non-trivial permute layout, the `batch_stride` field in `Gemm::Arguments` is set to `0`. This is the critical contract: when `batch_stride == 0` the kernel uses the permute layout's `operator()` to compute the global memory address for each element, bypassing the normal `ptr + batch_idx * batch_stride` offset. For trivial (NoPermute) tensors, the normal batch stride (`capacity`) is used.

**GEMM mode**: `kBatched` selects `GemmUniversalMode::kBatched` (a loop over batches using a single kernel launch) vs `kGemm` (single problem). The BMM permute layouts require `kBatched`; normal GEMM permutes use `kGemm` with `batch_count=1`.

**Profiling**: Uses CUDA events bracketing `iterations` kernel launches. A warm-up run before the timed loop ensures that device-side JIT compilation and cache effects are stabilized. The per-iteration average runtime is reported alongside GFLOPs.

**CN**: 核心性能分析方法。关键设计点：

**批次步长 = 0 规则**：对于任何非平凡置换布局，`Gemm::Arguments` 中的 `batch_stride` 字段被设置为 `0`。这是关键契约：当 `batch_stride == 0` 时，内核使用置换布局的 `operator()` 计算每个元素的全局内存地址，绕过正常的 `ptr + batch_idx * batch_stride` 偏移。对于平凡（NoPermute）张量，使用正常的批次步长（`capacity`）。

**GEMM 模式**：`kBatched` 选择 `GemmUniversalMode::kBatched`（使用单次内核启动对批次循环）vs `kGemm`（单问题）。BMM 置换布局需要 `kBatched`；普通 GEMM 置换使用 `kGemm`，`batch_count=1`。

**性能分析**：使用 CUDA 事件包围 `iterations` 次内核启动。计时循环前的预热运行确保设备端 JIT 编译和缓存效应已稳定。报告每次迭代的平均运行时间和 GFLOPs。

---

### Lines 774–810 — `GemmPermute` Alias Template

```cpp
/// Shorthand alist for GEMM instantiations
template<typename LayoutA, typename PermuteALayout,
         typename LayoutB, typename PermuteBLayout,
         typename LayoutC, typename PermuteDLayout>
using GemmPermute = cutlass::gemm::device::GemmUniversal<
  ElementInput, LayoutA,
  ElementInput, LayoutB,
  ElementOutput, LayoutC,
  ElementAccumulator,
  cutlass::arch::OpClassTensorOp,
  cutlass::arch::Sm80,
  cutlass::gemm::GemmShape<128, 128, 32>,
  cutlass::gemm::GemmShape<64, 64, 32>,
  cutlass::gemm::GemmShape<16, 8, 16>,
  cutlass::epilogue::thread::LinearCombination<
    ElementOutput, 
    AlignmentC, //128 / cutlass::sizeof_bits<ElementOutput>::value,
    ElementAccumulator, 
    ElementAccumulator
  >,
  cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<8>,
  4,              /*kStages*/
  AlignmentA,     /*AlignmentA*/
  AlignmentB,     /*AlignmentB*/
  cutlass::arch::OpMultiplyAdd,
  cutlass::ComplexTransform::kNone,
  cutlass::ComplexTransform::kNone,
  false,  /*GatherA*/
  false,  /*GatherB*/
  false,  /*ScatterD*/
  PermuteDLayout,  /*PermuteDLayout*/
  typename cutlass::layout::InversePermute<PermuteALayout>::type,  /*PermuteALayout*/
  typename cutlass::layout::InversePermute<PermuteBLayout>::type   /*PermuteBLayout*/
>;

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: This alias condenses the full `GemmUniversal` template instantiation into a 6-parameter form. Notable choices:
- **Sm80 + TensorOp + `GemmShape<16,8,16>`**: Ampere `mma.sync` instruction with FP16 inputs and FP32 accumulation, giving the best throughput on A100/A30.
- **Threadblock tile 128×128×32, warp tile 64×64×32**: A typical high-occupancy configuration for FP16 GEMM on Ampere.
- **kStages=4**: 4-stage software pipeline for async memory prefetch, hiding global memory latency.
- **`GemmIdentityThreadblockSwizzle<8>`**: Tile swizzle with 8 cohorts for L2 cache locality.
- **`InversePermute<PermuteALayout>::type`**: The A and B permute parameters are passed as their **inverse** types. This is because for input tensors the CUTLASS kernel reads from permuted memory and must apply the *inverse* mapping to translate the logical GEMM coordinate back to the physical address. The output tensor D receives the *forward* permute (`PermuteDLayout`) because the kernel writes to a permuted address space.

**CN**: 该别名将完整的 `GemmUniversal` 模板实例化压缩为 6 参数形式。值得注意的选择：
- **Sm80 + TensorOp + `GemmShape<16,8,16>`**：使用 FP16 输入和 FP32 累加的 Ampere `mma.sync` 指令，在 A100/A30 上提供最佳吞吐量。
- **线程块分块 128×128×32，warp 分块 64×64×32**：Ampere 上 FP16 GEMM 的典型高占用率配置。
- **kStages=4**：4 阶段软件流水线用于异步内存预取，隐藏全局内存延迟。
- **`GemmIdentityThreadblockSwizzle<8>`**：8 个组的分块交换以获得 L2 缓存局部性。
- **`InversePermute<PermuteALayout>::type`**：A 和 B 的置换参数以其**逆**类型传递。这是因为对于输入张量，CUTLASS 内核从置换内存中读取，必须应用*逆*映射将逻辑 GEMM 坐标转换回物理地址。输出张量 D 接收*正向*置换（`PermuteDLayout`），因为内核写入置换后的地址空间。

---

### Lines 811–852 — `main()` — Setup & GPU Capability Check

```cpp
int main(int argc, char const **args) {

  //
  // This example uses mma.sync to directly access Tensor Cores to achieve peak performance.
  //

  cudaDeviceProp props;

  CHECK_CUDA_CALL(cudaGetDeviceProperties(&props, 0), return EXIT_FAILURE);

  if (__CUDACC_VER_MAJOR__ < 11 || props.major < 8) {
  
    //
    // This example requires an NVIDIA Ampere-architecture GPU.
    //

    std::cout << "CUTLASS's GEMM+Permute example requires a GPU of NVIDIA's Ampere Architecture "
                 "or later (compute capability 80 or greater).\n";

    return EXIT_SUCCESS;
  }

  //
  // Parse options
  //

  Options options;
  
  options.parse(argc, args);

  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return EXIT_SUCCESS;
  }

  if (options.error) {
    std::cerr << "Aborting execution." << std::endl;
    return EXIT_FAILURE;
  }

  //
  // Define GEMM types to test
```
**EN**: The capability check guards both the CUDA compiler version (`__CUDACC_VER_MAJOR__ < 11`) and the device compute capability (`props.major < 8`). CUTLASS's `mma.sync` with the `GemmShape<16,8,16>` instruction requires `sm_80` (Ampere) or later because that instruction was introduced in Ampere. On pre-Ampere devices the example exits cleanly with `EXIT_SUCCESS` rather than `EXIT_FAILURE` to avoid CI failures on older GPUs.

**CN**: 能力检查同时守护 CUDA 编译器版本（`__CUDACC_VER_MAJOR__ < 11`）和设备计算能力（`props.major < 8`）。使用 `GemmShape<16,8,16>` 指令的 CUTLASS `mma.sync` 需要 `sm_80`（Ampere）或更新版本，因为该指令在 Ampere 中引入。在 Ampere 之前的设备上，示例以 `EXIT_SUCCESS` 而非 `EXIT_FAILURE` 干净退出，以避免在旧 GPU 上的 CI 失败。

---

### Lines 853–1062 — GEMM Type Definitions (TTT / NNN / NNT / TTN Normal GEMMs)

```cpp
  //

  //
  // TTT (Row-major) GEMMs
  //

  using TTTGemmNormalPermuteNone = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute
  >;

  using TTTGemmNormalPermuteA = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute
  >;

  using TTTGemmNormalPermuteAD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  using TTTGemmNormalPermuteB = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute
  >;

  using TTTGemmNormalPermuteBD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  using TTTGemmNormalPermuteD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  using TTTGemmNormalPermuteAB = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute
  >;

  using TTTGemmNormalPermuteABD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  //
  // NNN (Col-major) GEMMs
  //

  using NNNGemmNormalPermuteNone = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using NNNGemmNormalPermuteA = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using NNNGemmNormalPermuteAD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;

  using NNNGemmNormalPermuteB = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using NNNGemmNormalPermuteBD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;

  using NNNGemmNormalPermuteD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;

  using NNNGemmNormalPermuteAB = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using NNNGemmNormalPermuteABD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;

  //
  // NNT (Col-major inputs, row-major output) GEMMs
  //

  using NNTGemmNormalPermuteNone = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute
  >;

  using NNTGemmNormalPermuteA = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute
  >;

  using NNTGemmNormalPermuteAD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  using NNTGemmNormalPermuteB = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute
  >;

  using NNTGemmNormalPermuteBD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  using NNTGemmNormalPermuteD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  using NNTGemmNormalPermuteAB = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute
  >;

  using NNTGemmNormalPermuteABD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermute0213ColumnMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>
  >;

  //
  // TTN (Row-major inputs, col-major output) GEMMs
  //

  using TTNGemmNormalPermuteNone = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using TTNGemmNormalPermuteA = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using TTNGemmNormalPermuteAD = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;

  using TTNGemmNormalPermuteB = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using TTNGemmNormalPermuteBD = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;

  using TTNGemmNormalPermuteD = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::RowMajor,    cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;

  using TTNGemmNormalPermuteAB = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using TTNGemmNormalPermuteABD = GemmPermute<
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::RowMajor,    cutlass::layout::Tensor4DPermute0213RowMajor<S1, S2>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>
  >;
```
**EN**: The four layout families (TTT, NNN, NNT, TTN) cover all combinations of row-major (T) and column-major (N) for the three matrices A, B, and D. For each family, 8 permute variants are defined:
- `PermuteNone` — no permutation on any matrix (baseline).
- `PermuteA/B/D` — permutation on exactly one matrix.
- `PermuteAD/BD` — permutation on two matrices.
- `PermuteAB` — permutation on both inputs.
- `PermuteABD` — permutation on all three matrices.

The choice of *which* permute type is applied to each layout family is intentional: `Tensor4DPermute0213RowMajor` is used for row-major A, while `Tensor4DPermute0213ColumnMajor` is used for column-major B — each is specialized for the appropriate memory access pattern.

**CN**: 四个布局族（TTT、NNN、NNT、TTN）涵盖三个矩阵 A、B 和 D 的行主序（T）和列主序（N）的所有组合。对于每个族，定义了 8 种置换变体：
- `PermuteNone` — 任何矩阵都不置换（基线）。
- `PermuteA/B/D` — 恰好一个矩阵上的置换。
- `PermuteAD/BD` — 两个矩阵上的置换。
- `PermuteAB` — 两个输入矩阵上的置换。
- `PermuteABD` — 三个矩阵全部置换。

对每个布局族选择*哪种*置换类型是有意为之的：`Tensor4DPermute0213RowMajor` 用于行主序 A，而 `Tensor4DPermute0213ColumnMajor` 用于列主序 B——每种都专门针对适当的内存访问模式。

---

### Lines 1063–1154 — BMM GEMM Type Definitions (TTT / NNN Batched)

```cpp
  //
  // TTT (Row-major) BMMs
  //

  using TTTGemmBatchedPermuteA = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute
  >;

  using TTTGemmBatchedPermuteAD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>
  >;

  using TTTGemmBatchedPermuteB = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute
  >;

  using TTTGemmBatchedPermuteBD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>
  >;

  using TTTGemmBatchedPermuteD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>
  >;

  using TTTGemmBatchedPermuteAB = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::NoPermute,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>
  >;

  using TTTGemmBatchedPermuteABD = GemmPermute<
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>,
    cutlass::layout::RowMajor, cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>
  >;

  //
  // NNN (Col-major) BMMs
  //

  using NNNGemmBatchedPermuteA = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using NNNGemmBatchedPermuteAD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>
  >;

  using NNNGemmBatchedPermuteB = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using NNNGemmBatchedPermuteBD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>
  >;

  using NNNGemmBatchedPermuteD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>
  >;

  using NNNGemmBatchedPermuteAB = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::NoPermute
  >;

  using NNNGemmBatchedPermuteABD = GemmPermute<
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>,
    cutlass::layout::ColumnMajor, cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>
  >;
```
**EN**: BMM variants use `Tensor4DPermuteBMM0213RowMajor<D1>` (row-major) and `Tensor4DPermuteBMM0321ColumnMajor<D1>` (column-major). These split the batch dimension and swap it with a spatial dimension: `[B, M, N]` → `[B/D1, D1, M, N]` → `[B/D1, M, D1, N]`. This is useful in multi-head attention where the batch dimension encodes both batch and head indices. Note that when the same permute type appears on multiple tensors within one `using` alias, all permuted tensors must share compatible shape constraints.

**CN**: BMM 变体使用 `Tensor4DPermuteBMM0213RowMajor<D1>`（行主序）和 `Tensor4DPermuteBMM0321ColumnMajor<D1>`（列主序）。这些将批次维度分割并与空间维度交换：`[B, M, N]` → `[B/D1, D1, M, N]` → `[B/D1, M, D1, N]`。这在多头注意力机制中很有用，其中批次维度同时编码批次索引和注意力头索引。注意当相同的置换类型出现在一个 `using` 别名中的多个张量上时，所有置换张量必须共享兼容的形状约束。

---

### Lines 1155–1223 — `main()` — Test Loop & Final Report

```cpp
  //
  // Profile it
  //

  Testbed<ElementInput, ElementInput, ElementOutput> testbed(options);

  bool result = true;

  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteNone>();
  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteA>();
  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteAD>();
  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteB>();
  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteBD>();
  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteD>();
  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteAB>();
  result &= testbed.profile_GEMM_permute<TTTGemmNormalPermuteABD>();

  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteNone>();
  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteA>();
  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteAD>();
  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteB>();
  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteBD>();
  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteD>();
  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteAB>();
  result &= testbed.profile_GEMM_permute<NNNGemmNormalPermuteABD>();

  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteNone>();
  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteA>();
  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteAD>();
  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteB>();
  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteBD>();
  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteD>();
  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteAB>();
  result &= testbed.profile_GEMM_permute<NNTGemmNormalPermuteABD>();

  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteNone>();
  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteA>();
  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteAD>();
  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteB>();
  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteBD>();
  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteD>();
  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteAB>();
  result &= testbed.profile_GEMM_permute<TTNGemmNormalPermuteABD>();

  result &= testbed.profile_GEMM_permute<TTTGemmBatchedPermuteA>();
  result &= testbed.profile_GEMM_permute<TTTGemmBatchedPermuteAD>();
  result &= testbed.profile_GEMM_permute<TTTGemmBatchedPermuteB>();
  result &= testbed.profile_GEMM_permute<TTTGemmBatchedPermuteBD>();
  result &= testbed.profile_GEMM_permute<TTTGemmBatchedPermuteD>();
  result &= testbed.profile_GEMM_permute<TTTGemmBatchedPermuteAB>();
  result &= testbed.profile_GEMM_permute<TTTGemmBatchedPermuteABD>();

  result &= testbed.profile_GEMM_permute<NNNGemmBatchedPermuteA>();
  result &= testbed.profile_GEMM_permute<NNNGemmBatchedPermuteAD>();
  result &= testbed.profile_GEMM_permute<NNNGemmBatchedPermuteB>();
  result &= testbed.profile_GEMM_permute<NNNGemmBatchedPermuteBD>();
  result &= testbed.profile_GEMM_permute<NNNGemmBatchedPermuteD>();
  result &= testbed.profile_GEMM_permute<NNNGemmBatchedPermuteAB>();
  result &= testbed.profile_GEMM_permute<NNNGemmBatchedPermuteABD>();

  std::cout << "\n"
               "====================================================\n"
               "Finished (" << (result ? "PASS" : "FAIL") << ")\n"
               "====================================================" << std::endl;

  return result ? EXIT_SUCCESS : EXIT_FAILURE;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The main test loop profiles all ~50 GEMM variants sequentially, using `&=` accumulation to track any single failure. Each call to `profile_GEMM_permute<T>()` is fully self-contained: it allocates, initializes, runs, optionally validates, profiles, and reports. The final `PASS`/`FAIL` banner and return code are suitable for automated testing (CI). Total coverage: 4 layout families × 7 normal permute variants + 2 layout families × 7 batched permute variants = 42 cases, including the baselines.

**CN**: 主测试循环按顺序对约 50 种 GEMM 变体进行性能分析，使用 `&=` 累加跟踪任何单次失败。每次调用 `profile_GEMM_permute<T>()` 都是完全自包含的：分配、初始化、运行、可选验证、性能分析和报告。最终的 `PASS`/`FAIL` 横幅和返回码适合自动化测试（CI）。总覆盖率：4 个布局族 × 7 种普通置换变体 + 2 个布局族 × 7 种批处理置换变体 = 42 种情况（包含基线）。

---
## Key Concepts / 关键概念
- GemmUniversal can fuse layout permutation into loads and stores through compile-time layout policies. / `GemmUniversal` 可通过编译期布局策略将置换融合进读写路径。
- Host-side reference permutation is essential for validating CUTLASS iterator and epilogue behavior. / 主机端参考置换对于验证 CUTLASS 迭代器与 epilogue 行为至关重要。
- Template aliases expose many layout combinations while keeping the benchmarking harness reusable. / 模板别名可以暴露多种布局组合，同时保持基准框架可复用。
## Dependencies / 依赖项
- `cutlass/gemm/device/gemm_universal.h` — Universal CUTLASS GEMM entry point with flexible layouts / 支持灵活布局的通用 CUTLASS GEMM 入口
- `cutlass/layout/permute.h` — Defines CUTLASS permute layout tags used by the example / 定义示例使用的 CUTLASS 置换布局标签
- `layouts.h` — Custom host-side tensor layouts for verification / 用于校验的自定义主机端张量布局
- `permute_info.h` — Compile-time metadata describing each permute policy / 描述各置换策略的编译期元数据
