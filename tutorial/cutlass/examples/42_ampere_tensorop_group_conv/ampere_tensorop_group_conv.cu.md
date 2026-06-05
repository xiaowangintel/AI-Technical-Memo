# ampere_tensorop_group_conv.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/42_ampere_tensorop_group_conv/ampere_tensorop_group_conv.cu`
**Purpose / 用途**: Shows how to configure, verify, and profile Ampere Tensor Core grouped convolutions in CUTLASS. / 展示如何在 CUTLASS 中配置、验证并分析 Ampere Tensor Core 分组卷积。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1–31 — License Header / 许可证头

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
**EN**: Standard NVIDIA BSD-3-Clause license header, common to all CUTLASS examples.

**CN**: 标准 NVIDIA BSD-3-Clause 许可证头，所有 CUTLASS 示例均包含此头。

---

### Lines 32–47 — File Docstring / 文件说明注释

```cpp
/**
This example shows how to run group convolution kernels using functions and data structures
provided by CUTLASS using tensor cores; which we run on a NVIDIA Ampere GPU.

There are 2 group conv mode:
  1. cutlass::conv::GroupMode::kSingleGroup
      This mode is for large K problem size: k_per_group (K/groups) equals or larger than
      threadblock_tile_N. One or multiple threadblocks calculate data of one group.
  2. cutlass::conv::GroupMode::kMultipleGroup
      This mode is for small K problem size: k_per_group (K/groups) is smaller than threadblock_tile_N.
      One threadblock will calculate data from more than one group.

Function profile_convolution_selecter() shows how to choose kernel with different group mode according
to problem size and threadblock_tile size.
*/
```
**EN**: The docstring concisely defines the two group-conv modes. The threshold `threadblock_tile_N` is the compile-time constant `ThreadblockShape::kN` (= 64 in this example). When `K/G < 64` the `kMultipleGroup` path is chosen; otherwise `kSingleGroup`. `kMultipleGroup` only supports the Analytic iterator because its tile-to-group mapping requires general-purpose pointer arithmetic not amenable to compile-time specialisation.

**CN**: 文档注释简洁地定义了两种分组卷积模式。阈值 `threadblock_tile_N` 是编译期常量 `ThreadblockShape::kN`（本示例中为 64）。当 `K/G < 64` 时选择 `kMultipleGroup` 路径；否则选择 `kSingleGroup`。`kMultipleGroup` 仅支持 Analytic 迭代器，因为其 tile→group 映射需要通用指针算术，不适合编译期特化。

---

### Lines 48–68 — Includes / 头文件包含

```cpp
#include <iostream>
#include <sstream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"
#include "cutlass/conv/kernel/default_conv2d_group_fprop.h"
#include "cutlass/conv/device/implicit_gemm_convolution.h"

#include "cutlass/util/command_line.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/convolution.h"
#include "cutlass/util/reference/device/convolution.h"
#include "cutlass/util/tensor_view_io.h"

#include "helper.h"
```
**EN**: Key convolution-specific headers:
- `cutlass/conv/kernel/default_conv2d_group_fprop.h` — the meta-programming policy selector for grouped forward convolution. It assembles the correct combination of iterator, threadblock, warp-level MMA, and epilogue types based on the provided template parameters.
- `cutlass/conv/device/implicit_gemm_convolution.h` — the device-level operator wrapper for implicit GEMM convolution (transforms the convolution problem into a GEMM problem internally).
- `cutlass/util/reference/device/convolution.h` and `host/convolution.h` — GPU and CPU reference convolution implementations for correctness verification.
- `helper.h` — example-local header providing `CUDA_CHECK` and `CUTLASS_CHECK` macros.

**CN**: 关键卷积专用头文件：
- `cutlass/conv/kernel/default_conv2d_group_fprop.h` — 分组前向卷积的元编程策略选择器。它根据提供的模板参数组合出正确的迭代器、线程块、warp 级 MMA 和 epilogue 类型。
- `cutlass/conv/device/implicit_gemm_convolution.h` — 隐式 GEMM 卷积的设备级算子封装（内部将卷积问题转化为 GEMM 问题）。
- `cutlass/util/reference/device/convolution.h` 和 `host/convolution.h` — 用于正确性验证的 GPU 和 CPU 参考卷积实现。
- `helper.h` — 示例本地头文件，提供 `CUDA_CHECK` 和 `CUTLASS_CHECK` 宏。

---

### Lines 69–110 — Global Type Aliases: Element Types, Layouts, MMA, Epilogue / 全局类型别名：元素类型、布局、MMA、Epilogue

```cpp
// The code section below describes datatype for input, output tensors and computation between
// elements 
using ElementAccumulator = float;                  // Data type of accumulator
using ElementComputeEpilogue = float;              // Data type of epilogue computation (alpha, beta)
using ElementInputA = cutlass::half_t;             // Data type of elements in input tensor
using ElementInputB = cutlass::half_t;             // Data type of elements in input tensor
using ElementOutput = float;                       // Data type of elements in output tensor

using LayoutInputA = cutlass::layout::TensorNHWC;
using LayoutInputB = cutlass::layout::TensorNHWC;
using LayoutOutput = cutlass::layout::TensorNHWC;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm80;

// This code section describes the tile size a thread block will compute
using ThreadblockShape = cutlass::gemm::GemmShape<64, 64, 64>;   // Threadblock tile shape

// This code section describes tile size a warp will compute
using WarpShape = cutlass::gemm::GemmShape<32, 32, 64>;          // Warp tile shape

// This code section describes the size of MMA op
using InstructionShape = cutlass::gemm::GemmShape<16, 8, 16>;    // TensorCore instruction shape

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

// Number of pipelines you want to use
constexpr int NumStages = 3;

// This code section describes the epilogue part of the kernel, we use default value
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementOutput,                                     // Data type of output matrix.
    128 / cutlass::sizeof_bits<ElementOutput>::value,  // The number of elements per vectorized.
                                                       // memory access. This becomes the vector width of
                                                       // math instructions in the epilogue too.
    ElementAccumulator,                                // Data type of accumulator
    ElementComputeEpilogue>;                           // Data type for alpha/beta in linear combination
```
**EN**: Type-alias breakdown:
- **FP16 input × FP32 accumulate → FP32 output**: standard mixed-precision convolution. CUTLASS's `half_t` maps to `__half`.
- **NHWC layout**: all tensors use NHWC (batch, height, width, channels) — the memory-efficient layout for convolutions on modern GPUs because channel-dimension elements are contiguous, enabling vectorised 128-bit loads.
- **ThreadblockShape<64,64,64>**: each CTA computes a 64-output-channel × 64-pixel tile accumulating 64 input channels per main-loop step.
- **WarpShape<32,32,64>**: four warps (2×2) tile the 64×64 CTA block.
- **InstructionShape<16,8,16>**: the Ampere FP16 Tensor Core `mma.sync.aligned.m16n8k16` warp-level instruction.
- **`EpilogueOp` vectorization width** `= 128/32 = 4`: each thread writes a 4-element vector in the output store (FP32, 128-bit store).
- **NumStages = 3**: three-stage software pipeline — two stages prefetch from global memory while the third computes.

**CN**: 类型别名详解：
- **FP16 输入 × FP32 累加 → FP32 输出**：标准混合精度卷积。CUTLASS 的 `half_t` 映射到 `__half`。
- **NHWC 布局**：所有张量使用 NHWC（批次、高度、宽度、通道）——这是现代 GPU 上卷积的高效内存布局，因为通道维度元素连续，支持向量化 128 位加载。
- **ThreadblockShape<64,64,64>**：每个 CTA 计算 64 输出通道 × 64 像素的 tile，每次主循环步骤累加 64 个输入通道。
- **WarpShape<32,32,64>**：四个 warp（2×2）平铺 64×64 CTA 块。
- **InstructionShape<16,8,16>**：Ampere FP16 Tensor Core 的 `mma.sync.aligned.m16n8k16` warp 级指令。
- **`EpilogueOp` 向量化宽度** `= 128/32 = 4`：每个线程在输出存储时写入 4 元素向量（FP32，128 位存储）。
- **NumStages = 3**：三阶段软件流水线——两个阶段从全局内存预取，第三阶段执行计算。

---

### Lines 111–130 — `AnalyticSingleGroupKernel` / `AnalyticSingleGroupOperation` / 解析式单组内核与算子

```cpp
// Analytic kernel and operation for single group problem size
using AnalyticSingleGroupKernel = typename cutlass::conv::kernel::DefaultConv2dGroupFprop<
  ElementInputA, LayoutInputA,
  ElementInputB, LayoutInputB,
  ElementOutput, LayoutOutput,
  ElementAccumulator,
  MMAOp,
  SmArch,
  ThreadblockShape,
  WarpShape,
  InstructionShape,
  EpilogueOp,
  SwizzleThreadBlock,
  NumStages,
  cutlass::arch::OpMultiplyAdd,
  cutlass::conv::GroupMode::kSingleGroup,
  cutlass::conv::IteratorAlgorithm::kAnalytic
>::Kernel;
using AnalyticSingleGroupOperation = cutlass::conv::device::ImplicitGemmConvolution<AnalyticSingleGroupKernel>;
```
**EN**: `DefaultConv2dGroupFprop<...>::Kernel` is the primary meta-function that assembles the complete grouped convolution kernel type from individual building blocks. The two trailing non-type template parameters distinguish this variant:
- `GroupMode::kSingleGroup` — the tile iterator is configured to load strips belonging to exactly one group per CTA.
- `IteratorAlgorithm::kAnalytic` — the iterator computes memory addresses on-the-fly from the 4D tensor coordinates (N, H, W, C). This is slower than `kOptimized` but handles all padding, dilation, and stride configurations generically.

`ImplicitGemmConvolution<Kernel>` wraps the kernel with the device-level API: `can_implement()`, `initialize()`, `operator()`.

**CN**: `DefaultConv2dGroupFprop<...>::Kernel` 是主要的元函数，它将各个构建块组装成完整的分组卷积内核类型。末尾两个非类型模板参数区分了此变体：
- `GroupMode::kSingleGroup` — tile 迭代器被配置为每个 CTA 只加载属于一个组的数据条带。
- `IteratorAlgorithm::kAnalytic` — 迭代器从 4D 张量坐标（N, H, W, C）动态计算内存地址。这比 `kOptimized` 慢，但能通用地处理所有填充、膨胀和步幅配置。

`ImplicitGemmConvolution<Kernel>` 用设备级 API 封装内核：`can_implement()`、`initialize()`、`operator()`。

---

### Lines 131–172 — `AnalyticMultipleGroupKernel`, `AnalyticMultipleGroupOperation`, `OptimizedSingleGroupKernel`, `OptimizedSingleGroupOperation` / 多组内核与优化内核

```cpp
// Analytic kernel and operation for multiple group problem size
using AnalyticMultipleGroupKernel = typename cutlass::conv::kernel::DefaultConv2dGroupFprop<
  ElementInputA, LayoutInputA,
  ElementInputB, LayoutInputB,
  ElementOutput, LayoutOutput,
  ElementAccumulator,
  MMAOp,
  SmArch,
  ThreadblockShape,
  WarpShape,
  InstructionShape,
  EpilogueOp,
  SwizzleThreadBlock,
  NumStages,
  cutlass::arch::OpMultiplyAdd,
  cutlass::conv::GroupMode::kMultipleGroup,
  cutlass::conv::IteratorAlgorithm::kAnalytic
>::Kernel;
using AnalyticMultipleGroupOperation = cutlass::conv::device::ImplicitGemmConvolution<AnalyticMultipleGroupKernel>;

// Optimized kernel and operation for single group problem size
using OptimizedSingleGroupKernel = typename cutlass::conv::kernel::DefaultConv2dGroupFprop<
  ElementInputA, LayoutInputA,
  ElementInputB, LayoutInputB,
  ElementOutput, LayoutOutput,
  ElementAccumulator,
  MMAOp,
  SmArch,
  ThreadblockShape,
  WarpShape,
  InstructionShape,
  EpilogueOp,
  SwizzleThreadBlock,
  NumStages,
  cutlass::arch::OpMultiplyAdd,
  cutlass::conv::GroupMode::kSingleGroup,
  cutlass::conv::IteratorAlgorithm::kOptimized
>::Kernel;
using OptimizedSingleGroupOperation = cutlass::conv::device::ImplicitGemmConvolution<OptimizedSingleGroupKernel>;

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Three of the four possible (GroupMode × IteratorAlgorithm) combinations are instantiated (the fourth — `kMultipleGroup + kOptimized` — is unsupported):
- **`kMultipleGroup + kAnalytic`**: For small `k_per_group`. The iterator computes which group each channel belongs to at runtime; the Analytic algorithm is needed because the channel-to-group mapping introduces irregular strides.
- **`kSingleGroup + kOptimized`**: For large `k_per_group`. The Optimized iterator pre-computes a compact descriptor of the filter positions to visit, reducing address-computation overhead per GEMM row. This yields the highest throughput on regular, padded convolutions.

All four names follow the convention `<Algorithm><GroupMode>Kernel/Operation`.

**CN**: 四种可能的（GroupMode × IteratorAlgorithm）组合中实例化了三种（第四种——`kMultipleGroup + kOptimized`——不受支持）：
- **`kMultipleGroup + kAnalytic`**：适用于小 `k_per_group`。迭代器在运行时计算每个通道属于哪个组；由于通道到组的映射引入了不规则步幅，需要使用 Analytic 算法。
- **`kSingleGroup + kOptimized`**：适用于大 `k_per_group`。Optimized 迭代器预先计算要访问的滤波器位置的紧凑描述符，减少每行 GEMM 的地址计算开销，在规则的带填充卷积上产生最高吞吐量。

所有四个名称遵循 `<算法><分组模式>Kernel/Operation` 的命名约定。

---

### Lines 173–205 — `Options` Struct: Fields and Defaults / 选项结构体：字段与默认值

```cpp
// Command line options parsing
struct Options {

  bool help;
  cutlass::Tensor4DCoord input_size;
  cutlass::Tensor4DCoord filter_size;
  cutlass::Tensor4DCoord padding;
  cutlass::MatrixCoord conv_stride;
  cutlass::MatrixCoord dilation;
  int groups;
  bool reference_check;
  bool measure_performance;
  int iterations;
  ElementComputeEpilogue alpha;
  ElementComputeEpilogue beta;
  bool optimized;
  std::string tag;

  Options():
    help(false),
    input_size(1, 32, 32, 32),
    filter_size(32, 3, 3, 32),
    padding(1, 1, 1, 1),
    conv_stride(1, 1),
    dilation(1, 1),
    groups(1),
    reference_check(false),
    measure_performance(false),
    iterations(20),
    alpha(1),
    beta(0),
    optimized(false) { }
```
**EN**: `Tensor4DCoord` encodes 4D tensor shapes in CUTLASS as `(n, h, w, c)`. For the filter tensor, CUTLASS uses `(k, r, s, c/g)` — where `r` and `s` are filter height and width and `c/g` is the per-group input channel count (= `C / G`). The `padding` Tensor4DCoord is used as `(pad_top, pad_bottom, pad_left, pad_right)` (though this example only uses symmetric padding). The `optimized` flag at runtime selects between Analytic and Optimized iterator strategies.

**CN**: `Tensor4DCoord` 在 CUTLASS 中将 4D 张量形状编码为 `(n, h, w, c)`。对于滤波器张量，CUTLASS 使用 `(k, r, s, c/g)`——其中 `r` 和 `s` 是滤波器的高和宽，`c/g` 是每组输入通道数（= `C / G`）。`padding` Tensor4DCoord 用作 `(pad_top, pad_bottom, pad_left, pad_right)`（尽管本示例仅使用对称填充）。运行时的 `optimized` 标志用于在 Analytic 和 Optimized 迭代器策略之间选择。

---

### Lines 206–231 — `Options::valid()` / 参数合法性验证

```cpp
  // Verify the problem size is compatible with the CUTLASS Convolution implementation.
  bool valid() {

    //
    // CUTLASS attempts to load 128b vectors of cutlass::half_t (F16) elements. Consequently,
    // all pointers, strides, and tensor extents must be divisible by 8 elements.
    //
    int const kAlignment = 8;

    if ((input_size.c() % kAlignment) ||
      (filter_size.n() % kAlignment)) {

      // misaligned tensors
      return false;
    }

    // Invalid padding
    if ((padding.h() != filter_size.h() / 2) ||
      (padding.w() != filter_size.w() / 2)) {

      return false;
    }

    return true;
  }
```
**EN**: Enforces two constraints:
1. **Alignment**: input channels `C` and output channels `K` must be multiples of 8. This is required because CUTLASS loads data in 128-bit vectors: `128 / sizeof_bits<half_t>` = 8 elements per load. Misalignment would cause incorrect or illegal memory accesses.
2. **Symmetric half-padding**: the example only supports "same" convolution where `padding = filter_size // 2` (padding that preserves the spatial dimensions when stride=1).

**CN**: 强制执行两个约束：
1. **对齐**：输入通道 `C` 和输出通道 `K` 必须是 8 的倍数。这是必要的，因为 CUTLASS 以 128 位向量加载数据：`128 / sizeof_bits<half_t>` = 每次加载 8 个元素。未对齐会导致错误或非法的内存访问。
2. **对称半填充**：示例仅支持 "same" 卷积，其中 `padding = filter_size // 2`（当步幅为 1 时保持空间维度不变的填充方式）。

---

### Lines 232–296 — `Options::update()` and `parse()` / 更新与解析

```cpp
  /// Updates input and filter sizes
  void update(
    cutlass::Tensor4DCoord input_size,
    cutlass::Tensor4DCoord filter_size) {

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

    if (cmd.check_cmd_line_flag("optimized")) {
      optimized = true;
    }

    cmd.get_cmd_line_argument("n", input_size.n());
    cmd.get_cmd_line_argument("h", input_size.h());
    cmd.get_cmd_line_argument("w", input_size.w());
    cmd.get_cmd_line_argument("c", input_size.c());

    cmd.get_cmd_line_argument("k", filter_size.n());
    cmd.get_cmd_line_argument("r", filter_size.h());
    cmd.get_cmd_line_argument("s", filter_size.w());

    cmd.get_cmd_line_argument("g", groups);
    filter_size.c() = input_size.c() / groups;

    cmd.get_cmd_line_argument("u", conv_stride.row());
    cmd.get_cmd_line_argument("v", conv_stride.column());

    cmd.get_cmd_line_argument("alpha", alpha);
    cmd.get_cmd_line_argument("beta", beta);
    
    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("tag", tag);

    if (filter_size.h() == 3 && filter_size.w() == 3) {
      padding = {1, 1, 1, 1};
    }
    else {
      filter_size.h() = 1;
      filter_size.w() = 1;
      padding = {0, 0, 0, 0};
    }
  }
```
**EN**: `parse()` derives `filter_size.c() = input_size.c() / groups` — the number of input channels per group, which is the per-group filter depth. The `groups` parameter (`--g`) divides the C dimension. The code simplifies filter choices: only 3×3 (with 1×1 padding) or 1×1 (with no padding) filters are supported. `update()` auto-computes symmetric padding from filter height and width.

**CN**: `parse()` 推导 `filter_size.c() = input_size.c() / groups`——每组的输入通道数，即每组滤波器的深度。`groups` 参数（`--g`）用于划分 C 维度。代码简化了滤波器选择：仅支持 3×3（带 1×1 填充）或 1×1（无填充）滤波器。`update()` 从滤波器的高和宽自动计算对称填充。

---

### Lines 297–350 — `Options::print_usage()`, `output_size()`, and `gflops()` / 使用说明、输出尺寸与 GFLOP 计算

```cpp
  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "42_ampere_tensorop_group_conv example\n\n"
      << "  This example uses Ampere's Tensor Core operators on F16 data types to compute\n"
      << "  forward grouped convolution on tensors of layout NHWC.\n\n"
      << "Options:\n\n"
      << "  --help               If specified, displays this usage statement.\n\n"
      << "  --n=<int>            Input tensor extent N\n"
      << "  --h=<int>            Input tensor extent H\n"
      << "  --w=<int>            Input tensor extent W\n"
      << "  --c=<int>            Input tensor extent C\n"
      << "  --k=<int>            Filter extent K\n"
      << "  --r=<int>            Filter extent R\n"
      << "  --s=<int>            Filter extent S\n\n"
      << "  --g=<int>            Conv groups G\n\n"
      << "  --u=<int>            Conv stride_h\n\n"
      << "  --v=<int>            Conv stride_w\n\n"
      << "  --alpha=<float>      Epilogue scalar alpha\n"
      << "  --beta=<float>       Epilogue scalar beta\n\n"
      << "  --ref-check          If set (true), reference check is computed\n"
      << "  --perf-check         If set (true), performance is measured.\n"
      << "  --optimized          If set (true), use optimized kernel, otherwise use analytic kernel.\n"
      << "  --iterations=<int>   Number of profiling iterations to perform.\n"
      << "  --tag=<string>       String to replicate across the first column in the results table\n";

    out << "\n\nExamples:\n\n"
      << "$ ./examples/42_ampere_tensorop_group_conv/42_ampere_tensorop_group_conv  --n=4 --h=16 --w=16 --c=256 --k=128 --r=3 --s=3 --g=8 --ref-check\n\n"
      << "$ ./examples/42_ampere_tensorop_group_conv/42_ampere_tensorop_group_conv  --n=4 --h=16 --w=16 --c=256 --k=128 --r=3 --s=3 --g=2 --ref-check\n\n"
      << "$ ./examples/42_ampere_tensorop_group_conv/42_ampere_tensorop_group_conv  --n=4 --h=16 --w=16 --c=256 --k=128 --r=3 --s=3 --g=2 --ref-check --optimized\n\n";

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
    int64_t fmas = output_size().product() * int64_t(filter_size.h() * filter_size.w() * filter_size.c());
    
    // Two flops per multiply-add
    return 2.0 * double(fmas) / double(1.0e9) / runtime_s;
  }
};
```
**EN**: `output_size()` implements the standard convolution output dimension formula: `P = (H + 2·pad - R) / stride + 1` and similarly for `Q`. The output tensor shape is `(N, P, Q, K)`. `gflops()` counts FMAs as `N × P × Q × K × R × S × (C/G)` — for each output element there are `R·S·(C/G)` multiply-adds. Note that `filter_size.c() = C/G` (per-group filter depth), so the per-group FLOP count is automatically correct.

**CN**: `output_size()` 实现了标准的卷积输出维度公式：`P = (H + 2·pad - R) / stride + 1`，Q 类似。输出张量形状为 `(N, P, Q, K)`。`gflops()` 将 FMA 计数为 `N × P × Q × K × R × S × (C/G)`——每个输出元素有 `R·S·(C/G)` 个乘加运算。注意 `filter_size.c() = C/G`（每组滤波器深度），因此每组的 FLOP 计数自动正确。

---

### Lines 351–402 — `Result` Struct / 结果结构体

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////

struct Result {
  double runtime_ms;
  double gflops;
  cutlass::Status status;
  cutlass::Status reference_check;
  cudaError_t error;

  Result(): 
    runtime_ms(0), 
    gflops(0),
    status(cutlass::Status::kSuccess),
    reference_check(cutlass::Status::kInvalid),
    error(cudaSuccess) { }

  static std::ostream & print_header(std::ostream &out, Options const &options) {

    if (!options.tag.empty()) {
      out << "Name,";
    }

    out << "Layer,N,H,W,C,K,R,S,G,Runtime,GFLOPs";

    return out;
  }

  std::ostream & print(std::ostream &out, int idx, Options const &options) {

    if (!options.tag.empty()) {
      out << options.tag << ",";
    }

    out
      << "conv_" << idx << ","
      << options.input_size.n() << ","
      << options.input_size.h() << ","
      << options.input_size.w() << ","
      << options.input_size.c() << ","
      << options.filter_size.n() << ","
      << options.filter_size.h() << ","
      << options.filter_size.w() << ","
      << options.groups << ","
      << runtime_ms << ","
      << gflops;

    return out;
  }
};

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: `reference_check` uses `cutlass::Status::kInvalid` as a sentinel (when reference checking is disabled), `kSuccess` when the kernel output matches the reference, and `kErrorInternal` on mismatch. The `print_header` / `print` pair generates a CSV-style row reporting `Layer,N,H,W,C,K,R,S,G,Runtime,GFLOPs`, convenient for automated benchmarking.

**CN**: `reference_check` 使用 `cutlass::Status::kInvalid` 作为哨兵值（禁用参考检查时），`kSuccess` 表示内核输出与参考匹配，`kErrorInternal` 表示不匹配。`print_header` / `print` 对生成 CSV 格式行，报告 `Layer,N,H,W,C,K,R,S,G,Runtime,GFLOPs`，便于自动化基准测试。

---

### Lines 403–461 — `profile_convolution()`: Tensor Allocation and Initialization / 张量分配与初始化

```cpp
/// Runs one benchmark
template <typename Conv2dOperation>
Result profile_convolution(Options const &options) {

  Result result;

  //
  // Allocate host-device tensors using the CUTLASS Utilities.
  //

  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(options.input_size);
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(options.filter_size);
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c(options.output_size());
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d(options.output_size());
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_ref_d(options.output_size());

  //
  // Initialize tensors
  //

  // Fill tensor A on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_a.host_view(),
      1,
      ElementInputA(7),
      ElementInputA(-8),
      0);

  // Fill tensor B on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_b.host_view(),
      1,
      ElementInputB(7),
      ElementInputB(-8),
      0);

  // Fill tensor C on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_c.host_view(),
      1,
      ElementOutput(7),
      ElementOutput(-8),
      0);

  // Fill tensor D on host with zeros
  cutlass::reference::host::TensorFill(
      tensor_d.host_view());

  // Fill tensor D for reference on host with zeros
  cutlass::reference::host::TensorFill(
      tensor_ref_d.host_view());

  // Copy data from host to GPU
  tensor_a.sync_device();
  tensor_b.sync_device();
  tensor_c.sync_device();
  tensor_d.sync_device();
  tensor_ref_d.sync_device();
```
**EN**: `cutlass::HostTensor<Element, Layout>` manages both host and device memory for a tensor with a given shape and layout. The `Tensor4DCoord` shape is passed directly to its constructor. `TensorFillRandomUniform` fills with values in [-8, 7] (range chosen to avoid FP16 overflow during convolution). `sync_device()` copies the host buffer to the device buffer — CUTLASS host tensors maintain mirrored host/device copies and a `sync_device()` / `sync_host()` API for explicit transfer.

**CN**: `cutlass::HostTensor<Element, Layout>` 管理给定形状和布局的张量的主机和设备内存。`Tensor4DCoord` 形状直接传递给其构造函数。`TensorFillRandomUniform` 用 [-8, 7] 范围内的值填充（范围选择是为了避免卷积过程中 FP16 溢出）。`sync_device()` 将主机缓冲区复制到设备缓冲区——CUTLASS 主机张量维护镜像的主机/设备副本，并通过 `sync_device()` / `sync_host()` API 进行显式传输。

---

### Lines 462–517 — `profile_convolution()`: Problem Setup and Kernel Launch / 问题设置与内核启动

```cpp
  //
  // Define arguments for CUTLASS Convolution
  //

  cutlass::conv::Mode mode = cutlass::conv::Mode::kCrossCorrelation;

  // Split K dimension into 1 partitions
  int split_k_slices = 1;

  // Construct Conv2dProblemSize with user defined output size
  cutlass::conv::Conv2dProblemSize problem_size(
      options.input_size,
      options.filter_size,
      options.padding,
      options.conv_stride,
      options.dilation,
      options.output_size(),
      mode,
      split_k_slices,
      options.groups
  );

  // Construct Conv2dOperation::Argument structure with conv2d 
  // problem size, data pointers, and epilogue values
  typename Conv2dOperation::Arguments arguments{
    problem_size,
    tensor_a.device_ref(),
    tensor_b.device_ref(),
    tensor_c.device_ref(),
    tensor_d.device_ref(),
    {options.alpha, options.beta},
  };

  //
  // Initialize CUTLASS Convolution
  //

  Conv2dOperation implicit_gemm_op;

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
**EN**: `Conv2dProblemSize` is the unified descriptor that CUTLASS uses to parameterise the entire implicit-GEMM convolution. It encapsulates all spatial parameters (input size, filter size, padding, stride, dilation, output size, split-K, and group count). `mode = kCrossCorrelation` means the filter is applied as a cross-correlation (standard convolution, no filter flip) — the vast majority of deep-learning convolutions use this mode. `split_k_slices = 1` means no split-K reduction; all K-tiles are computed by a single CTA. `device_ref()` returns a `TensorRef` (pointer + layout stride) to the device allocation. `can_implement()` checks whether the current device supports this exact kernel instantiation (SM version, shared memory size). `initialize()` sets up internal state and the workspace (used for split-K atomics when `split_k_slices > 1`). `operator()` launches the GPU kernel.

**CN**: `Conv2dProblemSize` 是 CUTLASS 用于参数化整个隐式 GEMM 卷积的统一描述符。它封装了所有空间参数（输入尺寸、滤波器尺寸、填充、步幅、膨胀、输出尺寸、split-K 和分组数）。`mode = kCrossCorrelation` 表示滤波器以互相关方式应用（标准卷积，不翻转滤波器）——绝大多数深度学习卷积使用此模式。`split_k_slices = 1` 表示不使用 split-K 归约；所有 K-tile 由单个 CTA 计算。`device_ref()` 返回设备分配的 `TensorRef`（指针 + 布局步幅）。`can_implement()` 检查当前设备是否支持此精确的内核实例化（SM 版本、共享内存大小）。`initialize()` 设置内部状态和工作区（当 `split_k_slices > 1` 时用于 split-K 原子操作）。`operator()` 启动 GPU 内核。

---

### Lines 518–632 — `profile_convolution()`: Reference Check and Performance Measurement / 参考检查与性能测量

```cpp

  //
  // Optional reference check
  //

  if (options.reference_check) {
    std::cout << "Verification on device...\n";

    // Compute with reference implementation
    cutlass::reference::device::Conv2dFprop<
      ElementInputA,
      LayoutInputA,
      ElementInputB,
      LayoutInputB,
      ElementOutput,
      LayoutOutput,
      ElementComputeEpilogue,
      ElementAccumulator,
      cutlass::NumericConverter<ElementOutput, ElementComputeEpilogue>
    >(
      problem_size,
      tensor_a.device_ref(),
      tensor_b.device_ref(),
      tensor_c.device_ref(),
      tensor_ref_d.device_ref(),
      options.alpha,
      options.beta
    );

    tensor_ref_d.sync_host();

    // Check if output from CUTLASS kernel and reference kernel are equal or not
    tensor_d.sync_host();

    bool passed = cutlass::reference::host::TensorEquals(
      tensor_d.host_view(),
      tensor_ref_d.host_view());

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

  //
  // Performance measurement
  //

  if (options.measure_performance) {

    cudaEvent_t events[2];
    
    for (auto & event : events) {
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

    // Print average runtime and GFLOPs.
    result.runtime_ms = double(runtime_ms) / double(options.iterations);
    result.gflops = options.gflops(result.runtime_ms / 1000.0);

    // Cleanup
    for (auto event : events) {
      (void)cudaEventDestroy(event);
    }
  }

  return result;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The GPU reference convolution `Conv2dFprop` is a simple looped CUDA kernel that directly implements the mathematical definition of convolution without any optimisation. It uses `NumericConverter<ElementOutput, ElementComputeEpilogue>` for output type casting. Performance is measured using CUDA events bracketing `options.iterations` warm kernel invocations. Average runtime is divided by `iterations` to reduce timing variance.

**CN**: GPU 参考卷积 `Conv2dFprop` 是一个简单的循环 CUDA 内核，直接实现卷积的数学定义，不做任何优化。它使用 `NumericConverter<ElementOutput, ElementComputeEpilogue>` 进行输出类型转换。性能通过括住 `options.iterations` 次预热内核调用的 CUDA 事件来测量。平均运行时间除以 `iterations` 以减少计时方差。

---

### Lines 633–657 — `profile_convolution_selecter()` / 分组模式自动选择器

```cpp
Result profile_convolution_selecter(Options const &options) {
  int k_per_group = options.filter_size.n() / options.groups;

  // In group conv, if k_per_group < threadblock_N, one Threadblock will calculate multiple groups
  if (k_per_group < ThreadblockShape::kN) { // MultipleGroup mode
    if (options.optimized) {
      std::cerr << "Invalid problem: optimized group conv kernel doesn't support MultipleGroup (one CTA calculate multiple groups) mode" << std::endl;
      exit(-1);
    } else {
      std::cout << "Select AnalyticMultipleGroupOperation\n";
      return profile_convolution<AnalyticMultipleGroupOperation>(options);
    }
  } else { // SingleGroup mode
    if (options.optimized) {
      std::cout << "Select OptimizedSingleGroupOperation\n";
      return profile_convolution<OptimizedSingleGroupOperation>(options);
    } else {
      std::cout << "Select AnalyticSingleGroupOperation\n";
      return profile_convolution<AnalyticSingleGroupOperation>(options);
    }
  }
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The selector compares `k_per_group = K / G` against the compile-time constant `ThreadblockShape::kN = 64`. This boundary determines whether the output-channel tile of a single CTA can be filled by one group alone:
- If `k_per_group ≥ 64`: one group has at least 64 output channels — a full CTA tile. Use `kSingleGroup` (Analytic or Optimized).
- If `k_per_group < 64`: one group has fewer output channels than a CTA tile, so the CTA must span multiple groups (`kMultipleGroup`, Analytic only).

The Optimized iterator is rejected for `kMultipleGroup` because it pre-computes strides assuming a single group boundary, which is invalid when a CTA spans multiple groups.

**CN**: 选择器将 `k_per_group = K / G` 与编译期常量 `ThreadblockShape::kN = 64` 进行比较。这个边界决定了单个 CTA 的输出通道 tile 是否能由一个组单独填满：
- 若 `k_per_group ≥ 64`：一个组有至少 64 个输出通道——恰好填满一个 CTA tile。使用 `kSingleGroup`（Analytic 或 Optimized）。
- 若 `k_per_group < 64`：一个组的输出通道数少于 CTA tile，因此 CTA 必须跨越多个组（`kMultipleGroup`，仅 Analytic）。

Optimized 迭代器在 `kMultipleGroup` 情况下被拒绝，因为它预先计算了假设单个组边界的步幅，而当 CTA 跨越多个组时这是无效的。

---

### Lines 658–706 — `main()` / 主函数

```cpp
int main(int argc, char const **args) {

  bool notSupported = false;

  // Ampere Tensor Core operations exposed with mma.sync are first available in CUDA 11.0.
  //
  // CUTLASS must be compiled with CUDA 11 Toolkit to run Conv2dFprop examples.
  if (!(__CUDACC_VER_MAJOR__ > 11 || (__CUDACC_VER_MAJOR__ == 11 && __CUDACC_VER_MINOR__ >= 0))) {
    std::cerr << "Ampere Tensor Core operations must be compiled with CUDA 11.0 Toolkit or later." << std::endl;
    notSupported = true;
  }

  cudaDeviceProp props;
  CUDA_CHECK(cudaGetDeviceProperties(&props, 0));

  if (!(props.major > 8 || (props.major == 8 && props.minor >= 0))) {
    std::cerr << "Ampere Tensor Ops must be run on a machine with compute capability at least 80."
              << std::endl;
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

  Result result = profile_convolution_selecter(options);

  Result::print_header(std::cout, options) << std::endl;
  result.print(std::cout, 1, options) << std::endl;

  return 0;
}

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: `main()` enforces two hardware requirements: CUDA 11.0+ (for `mma.sync` Ampere Tensor Core support in the compiler) and SM 8.0+ (Ampere architecture). A single call to `profile_convolution_selecter()` dispatches to the correct kernel type. Results are printed in the CSV-compatible `print()` format. Note that this example supports only one problem size per run (unlike the grouped GEMM example), so there is no outer problem-size loop.

**CN**: `main()` 强制执行两个硬件要求：CUDA 11.0+（编译器支持 `mma.sync` Ampere Tensor Core）和 SM 8.0+（Ampere 架构）。对 `profile_convolution_selecter()` 的一次调用分派到正确的内核类型。结果以 CSV 兼容的 `print()` 格式打印。注意，与分组 GEMM 示例不同，本示例每次运行仅支持一个问题规模，因此没有外层问题规模循环。

---
## Key Concepts / 关键概念
- Grouped convolution in CUTLASS is expressed as implicit GEMM with group-aware tiling policies. / CUTLASS 中的分组卷积通过带组感知分块策略的隐式 GEMM 表达。
- Runtime selection chooses between analytic and optimized iterator algorithms based on problem shape. / 运行时选择器根据问题形状在 analytic 与 optimized 迭代器算法间切换。
- The example couples reference convolution, kernel launch, and profiler output in one reproducible harness. / 该示例将参考卷积、内核启动与性能输出整合为可复现的测试框架。
## Dependencies / 依赖项
- `cutlass/conv/device/implicit_gemm_convolution.h` — Device API for implicit-GEMM convolution kernels / 隐式 GEMM 卷积内核的设备 API
- `cutlass/conv/kernel/default_conv2d_group_fprop.h` — Default grouped-convolution kernel builder for forward propagation / 前向分组卷积默认内核构建器
- `cutlass/util/reference/host/convolution.h` — Host reference convolution for verification / 用于验证的主机端参考卷积实现
- `helper.h` — Example-local helpers for data setup and reporting / 示例本地的数据准备与输出辅助工具
