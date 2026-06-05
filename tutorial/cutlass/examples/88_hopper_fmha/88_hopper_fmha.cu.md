# 88_hopper_fmha.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/88_hopper_fmha/88_hopper_fmha.cu`  
**Purpose / 用途**: Top-level Hopper FMHA sample runner that parses options, instantiates kernels, profiles them, and validates results / Hopper FMHA 的顶层示例运行器，负责解析参数、实例化 kernel、统计性能并校验结果

---

## Line-by-Line Analysis / 逐行分析

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
    \brief Example implementation of fused multi-head attention for Hopper using CUTLASS 3.

    This example showcases the use of CUTLASS to build forward and backward fused
    multi-head attention (FMHA) collectives from existing CUTLASS collectives targeting
    the NVIDIA Hopper architecture.
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    Background and motivation
    -------------------------
    CUTLASS is a highly flexible library that provides open-source building blocks
    for tensor core programming for GEMM or GEMM-like problems. Fused multi-head
    attention (FMHA) is a foundational kernel for large language models (LLMs) since it
    makes long sequence lengths feasible from a memory-usage perspective. It also
    improves computational efficiency since it transforms an outer-product-like and
    a matrix-vector-like GEMM into a fused operation with much higher arithmetic
    intensity. For more details, see Dao et al, 2022; Dao, 2023.
    Implementing this kernel in CUTLASS enabled easy customization and high
    performance.

    Introduction
    ------------
    The example targets the NVIDIA Hopper architecture, and takes advantage of
    warpgroup-wide tensor cores, the Tensor Memory Accelerator (TMA), just like
    GEMMs do. It provides both a forward and a backward pass (often abbreviated
    fwd and bwd in the code), and an optional FP8 mode for the forward pass.
    The code is structured into four layers: The runner (and the reference kernels)
    takes care of initialization, measurement, and testing; the device layer
    orchestrates kernel calls and partitions workspace; the kernel layer (just
    like the CUTLASS kernel layer); and the collective layer (most of the logic
    of FMHA is implemented here).

    Details
    -------
    This example contains a considerable amount of code. For a more detailed
    look at it, please refer to the README.md.

    Example usage:
      $ ./examples/88_hopper_fmha/88_hopper_fmha \
            --b=2048 --h=2048 --d=2048 --q=2048 --k=2048
*/

#include <iostream>

#include "cute/tensor.hpp"

#include "cutlass/cutlass.h"
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
#include "cutlass/util/command_line.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/tensor_fill.h"

#include "collective/fmha_fusion.hpp"
#include "device/fmha_device_bwd.hpp"
#include "device/device_universal.hpp"
#include "kernel/fmha_kernel_builder.hpp"
#include "reference/fmha_reference.hpp"
#include "reference/fmha_bwd_reference.hpp"
#include "reference/reference_abs_error.hpp"

using namespace cute;
using namespace cutlass::fmha::kernel;
using namespace cutlass::fmha::collective;
using namespace cutlass::fmha;

// Uncomment for FP8
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
// #define FP8
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
/// Command line options parsing
struct Options {

  bool help;
  bool error;

  int b, h, q, k, d;
  int iterations;
  bool verify;
  bool verbose;
  bool causal;
  bool residual;
  bool bwd;

  Options():
    help(false),
    error(false),
    b(16), h(16), q(1024), k(1024), d(128),
    iterations(3), verify(false),
    causal(false), residual(false), bwd(false), verbose(false)
  { }

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    Options defaults;

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("d", d, defaults.d);
    cmd.get_cmd_line_argument("h", h, -1);
    if (h == -1) h = 2048 / d;
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    cmd.get_cmd_line_argument("q", q, -1);
    cmd.get_cmd_line_argument("k", k, -1);
    if (q == -1) q = k;
    if (k == -1) k = q;
    if (q == -1 && k == -1) q = k = defaults.q;

    cmd.get_cmd_line_argument("b", b, -1);
    if (b == -1) b = 16384 / k;
    if (b == 0) b = 1;

    cmd.get_cmd_line_argument("iterations", iterations, defaults.iterations);
    verify = cmd.check_cmd_line_flag("verify");
    verbose = cmd.check_cmd_line_flag("verbose");

    std::string mask;
    cmd.get_cmd_line_argument<std::string>("mask", mask, "");
    if (mask == "no" || mask == "") {
      causal = residual = false;
    }
    else if (mask == "causal") {
      residual = false;
      causal = true;
    }
    else if (mask == "residual") {
      residual = true;
      causal = false;
    }

    bwd = cmd.check_cmd_line_flag("bwd");
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    out << "88_hopper_fmha\n\n"
      << "  This example showcases the use of CUTLASS's collective operation builders to easily construct\n"
      << "  fused multi-head attention forward-pass kernels targeting NVIDIA's Hopper architecture.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --b=<int>                   Sets the B extent\n"
      << "  --h=<int>                   Sets the H extent\n"
      << "  --q=<int>                   Sets the Q extent\n"
      << "  --k=<int>                   Sets the K extent\n"
      << "  --d=<int>                   Sets the D extent\n"
      << "  --iterations=<int>          Benchmarking iterations\n"
      << "  --verify                    Verify results\n"
      << "  --verbose                   Print smem and execution time per kernel\n"
      << "  --mask=<no|residual|causal> Enables masking\n"
      << "  --bwd                       Runs the backwards pass\n"
      << "\n";
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    return out;
  }
};
```

**EN**: This runtime-options block defines the knobs exposed to users and benchmarking scripts, then parses them into structured state. The surrounding example logic later uses those fields to size tensors, choose schedules, and decide whether to verify or profile the run. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这个运行期参数块定义了用户和基准脚本可见的控制旋钮，并把它们解析成结构化状态。后续示例逻辑会用这些字段来确定张量大小、选择调度策略，以及决定是否进行验证或性能测试。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
/// Helper to initialize a block of device data
template <class Element>
bool initialize_block(
  cutlass::DeviceAllocation<Element>& block,
  uint64_t seed=2023, bool init_one=false) {

  if (init_one) {
    cutlass::reference::device::BlockFillRandomUniform(
      block.get(), block.size(), seed, (Element) 1, (Element) 1);
  } else {
    cutlass::reference::device::BlockFillRandomGaussian(
      block.get(), block.size(), seed, (Element) 0, (Element) 1);
  }

  return true;
}
```

**EN**: This verification-oriented block computes, launches, or compares against a reference path so the example can separate functional correctness from performance tuning. CUTLASS samples rely on these checks to confirm that aggressive scheduling or narrow-precision choices still produce acceptable results. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一验证相关块会计算、启动或比对参考路径，使示例能够把“功能正确性”和“性能调优”区分开来。CUTLASS 样例依赖这些检查来确认：即便采用激进调度或窄精度配置，结果仍处于可接受范围内。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
struct ExampleResult {
  bool passed = false;
  bool verified = false;
  float runtime_ms = 0;
  double tflops_s = 0;
  size_t smem_size = 0;
};
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```

**EN**: This preprocessor block gates architecture-specific code or closes a specialized compilation path. It keeps the file buildable across toolchains and GPU targets even when the most optimized path only exists on a subset of architectures. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一预处理块负责为特定架构的代码设置编译门槛，或者结束某条专门化路径。这样即便最优路径只存在于部分 GPU 架构上，文件依旧可以在不同工具链与目标上保持可编译。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
  class TileShape,
  class DispatchPolicy,
  class ActiveFusion,
  class... KernelOptions
>
struct FwdRunner {

#ifdef FP8
  using Element = cutlass::float_e4m3_t;
  using ElementAccumulatorQK = find_option_t<Tag::kAccQK, float, KernelOptions...>;
#else
  using Element = cutlass::half_t;
  using ElementAccumulatorQK = float;
#endif

  using ElementAccumulatorPV = float;

  // B H Q K D
  using ProblemShapeType = cute::tuple<int, int, int, int, int>;
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
  using StrideQ = cute::tuple<int, _1, cute::tuple<int, int>>;  // Q D (B H)
  using StrideK = cute::tuple<int, _1, cute::tuple<int, int>>;  // K D (B H)
  using StrideV = std::conditional_t<sizeof(Element) == 1,
    cute::tuple<_1, int, cute::tuple<int, int>>,
    cute::tuple<int, _1, cute::tuple<int, int>>>;  // K D (B H)
  using StrideO = cute::tuple<int, _1, cute::tuple<int, int>>; // Q D (B H)
  using StrideLSE = cute::tuple<_1, cute::tuple<int, int>>; // Q (B H)

  using Operation = cutlass::device::Universal<
    typename cutlass::fmha::kernel::FmhaBuilder<
      Element, ElementAccumulatorQK, ElementAccumulatorPV,
      TileShape, StrideQ, StrideK, StrideV,
      ActiveFusion, DispatchPolicy, KernelOptions...
    >::Kernel>;

  //
  // Data members
  //

  /// Initialization
  StrideQ stride_Q;
  StrideK stride_K;
  StrideV stride_V;
  StrideO stride_O;
  StrideLSE stride_LSE;
  uint64_t seed = 0;

  cutlass::DeviceAllocation<Element> block_Q;
  cutlass::DeviceAllocation<Element> block_K;
  cutlass::DeviceAllocation<Element> block_V;
  cutlass::DeviceAllocation<Element> block_O;
  cutlass::DeviceAllocation<ElementAccumulatorPV> block_LSE;
  cutlass::DeviceAllocation<Element> block_ref_O;
  cutlass::DeviceAllocation<ElementAccumulatorPV> block_ref_LSE;

  //
  // Methods
  //
  bool verify(const ProblemShapeType& problem_size) {
    auto [B, H, Q, K, D] = problem_size;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    Tensor mQ = make_tensor(make_gmem_ptr(block_Q.get()),
      make_shape(Q, D, make_shape(B, H)),
      stride_Q);

    Tensor mK = make_tensor(make_gmem_ptr(block_K.get()),
      make_shape(K, D, make_shape(B, H)),
      stride_K);

    Tensor mV = make_tensor(make_gmem_ptr(block_V.get()),
      make_shape(K, D, make_shape(B, H)),
      stride_V);

    Tensor mO = make_tensor(make_gmem_ptr(block_ref_O.get()),
      make_shape(Q, D, make_shape(B, H)),
      stride_O);

    Tensor mLSE = make_tensor(make_gmem_ptr(block_ref_LSE.get()),
      make_shape(Q, make_shape(B, H)),
      stride_LSE);

    fmha_reference(problem_size, mQ, mK, mV, mO, mLSE, ActiveFusion{});
    cudaError_t result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Reference kernel failed. Last CUDA error: "
                << cudaGetErrorString(result) << std::endl;
      return false;
    }

    const double kMaxDiffThresh = sizeof(Element) == 1 ? 1e-1 : 1e-2;
    const double kMeanDiffThresh = sizeof(Element) == 1 ? 1e-1 : 1e-3;

    // Check if output from CUTLASS kernel and reference kernel are equal or not
    double max_diff = 0;
    double mean_diff = 0;
    reference_abs_diff(block_O, block_ref_O, max_diff, mean_diff);
    bool passed_O = (max_diff < kMaxDiffThresh) && (mean_diff < kMeanDiffThresh);
    if (! passed_O) {
      std::cerr << "failed O: max diff " << max_diff 
                << " mean " << mean_diff << std::endl;
    }
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    reference_abs_diff(block_LSE, block_ref_LSE, max_diff, mean_diff);
    bool passed_LSE = (max_diff < kMaxDiffThresh) && (mean_diff < kMeanDiffThresh);
    if ( ! passed_LSE) {
      std::cerr << "failed LSE: max diff " << max_diff 
                << " mean " << mean_diff << std::endl;
    }

    return passed_O && passed_LSE;
  }

  void initialize_stride(cute::tuple<int, int, int> const& shape, cute::tuple<_1, cute::tuple<int, int>>& stride) {
    auto [B, H, Q] = shape;
    stride = make_stride(_1{}, make_stride(H*Q, Q));
  }

  void initialize_stride(cute::tuple<int, int, int, int> const& shape, cute::tuple<int, _1, cute::tuple<int, int>>& stride) {
    auto [B, H, Q, D] = shape;
    stride = make_stride(D, _1{}, make_stride(H*Q*D, Q*D));
  }

  void initialize_stride(cute::tuple<int, int, int, int> const& shape, cute::tuple<_1, int, cute::tuple<int, int>>& stride) {
    auto [B, H, Q, D] = shape;
    stride = make_stride(_1{}, Q, make_stride(H*Q*D, Q*D));
  }

  /// Initialize operands to be used in the GEMM and reference GEMM
  void initialize(const ProblemShapeType& problem_size) {
    auto [B, H, Q, K, D] = problem_size;
    D = cutlass::round_up(D, 8); // Alignment

    auto shape_QO = cute::make_shape(B, H, Q, D);
    auto shape_KV = cute::make_shape(B, H, K, D);
    auto shape_LSE = cute::make_shape(B, H, Q);

    initialize_stride(shape_QO, stride_Q);
    initialize_stride(shape_KV, stride_K);
    initialize_stride(shape_KV, stride_V);
    initialize_stride(shape_QO, stride_O);
    initialize_stride(shape_LSE, stride_LSE);
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    block_Q.reset(size(shape_QO));
    block_K.reset(size(shape_KV));
    block_V.reset(size(shape_KV));
    block_O.reset(size(shape_QO));
    block_LSE.reset(size(shape_LSE));
    block_ref_O.reset(size(shape_QO));
    block_ref_LSE.reset(size(shape_LSE));

    initialize_block(block_Q, seed + 2023, false);
    initialize_block(block_K, seed + 2022, false);
    initialize_block(block_V, seed + 2021, false);
  }

  ExampleResult run(const Options& options, const cutlass::KernelHardwareInfo& hw_info) {
    ProblemShapeType problem_size = ProblemShapeType{options.b, options.h, options.q, options.k, options.d};

    initialize(problem_size);

    typename Operation::Arguments arguments{
      problem_size,
      { block_Q.get(), stride_Q,
        block_K.get(), stride_K,
        block_V.get(), stride_V },
      { block_O.get(), stride_O,
      block_LSE.get(), stride_LSE },
      hw_info
    };

    Operation op;

    ExampleResult example_result;

    example_result.smem_size = Operation::Kernel::SharedStorageSize;

    size_t workspace_size = Operation::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Manages shared memory and, on blackwell, often tmem-backed intermediate state.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 管理共享内存，并在 Blackwell 上经常涉及 TMEM 中间状态。

---

```cpp
    cutlass::Status status = op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "This kernel is not supported. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return example_result;
    }

    status = op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return example_result;
    }

    // Run
    status = op.run();
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return example_result;
    }

    cudaError_t result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    //
    // Construct events
    //

    cudaEvent_t events[2];
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    for (auto & event : events) {
      result = cudaEventCreate(&event);
      if (result != cudaSuccess) {
        std::cerr << "cudaEventCreate() failed: " << cudaGetErrorString(result) << std::endl;
        return example_result;
      }
    }

    // Record an event at the start of a series of GEMMs
    result = cudaEventRecord(events[0]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    for (int i = 0; i < options.iterations; i++) {
      status = op.run();
      if (status != cutlass::Status::kSuccess) {
        std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                  << cudaGetErrorString(cudaGetLastError()) << std::endl;
        return example_result;
      }
    }

    //
    // Stop profiling loop
    //

    // Record an event when the GEMMs are complete
    result = cudaEventRecord(events[1]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    // Wait for work on the device to complete.
    result = cudaEventSynchronize(events[1]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventSynchronize() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    // Measure elapsed runtime
    float runtime_ms = 0;
    result = cudaEventElapsedTime(&runtime_ms, events[0], events[1]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    runtime_ms /= static_cast<float>(options.iterations);

    double flops = 4.0 * (std::is_same_v<ActiveFusion, CausalFusion> ? 0.5 : 1.0);
    flops *= static_cast<double>(get<0>(problem_size));
    flops *= static_cast<double>(get<1>(problem_size));
    flops *= static_cast<double>(get<2>(problem_size));
    flops *= static_cast<double>(get<3>(problem_size));
    flops *= static_cast<double>(get<4>(problem_size));
    double tflops_s = flops * 1e-12 /*tera*/ / (runtime_ms * 1e-3 /*ms*/);
    example_result.tflops_s = tflops_s;
    example_result.runtime_ms = runtime_ms;

    result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    // Verify that the result is correct
    bool passed = true;
    if (options.verify) {
      passed = verify(problem_size);
      if (passed) example_result.verified = true;
    }
    
    if (!passed) {
      std::cerr << "Reference check failed" << std::endl;
      return example_result;
    }

    example_result.passed = true;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    return example_result;
  }

};
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
template<
  class TileShape,
  class DispatchPolicy,
  class ActiveFusion,
  class... KernelOptions
>
struct BwdRunner {

  using Element = cutlass::half_t;
  using ElementAccumulator = float;

  // B H Q K D
  using ProblemShapeType = cute::tuple<int, int, int, int, int>;

  using Operation = cutlass::fmha::device::FmhaBwd<Element, ElementAccumulator, TileShape, ActiveFusion, KernelOptions...>;
  
  // Just like forward
  using StrideQ = cute::tuple<int, int, int, _1>; // B H Q D
  using StrideK = cute::tuple<int, int, int, _1>; // B H K D
  using StrideV = cute::tuple<int, int, int, _1>; // B H K D
  using StrideO = cute::tuple<int, int, int, _1>; // B H Q D
  using StrideLSE = cute::tuple<int, int, _1>; // B H Q

  // Backwards specific
  using StrideDQ = cute::tuple<int, int, int, _1>; // B H Q D
  using StrideDK = cute::tuple<int, int, int, _1>; // B H K D
  using StrideDV = cute::tuple<int, int, int, _1>; // B H K D
  using StrideDO = cute::tuple<int, int, int, _1>; // B H Q D

  //
  // Data members
  //

  /// Initialization
  StrideQ stride_Q;
  StrideK stride_K;
  StrideV stride_V;
  StrideO stride_O;
  StrideLSE stride_LSE;
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  StrideDQ stride_dQ;
  StrideDK stride_dK;
  StrideDV stride_dV;
  StrideDO stride_dO;

  uint64_t seed = 0;

  cutlass::DeviceAllocation<Element> block_Q;
  cutlass::DeviceAllocation<Element> block_K;
  cutlass::DeviceAllocation<Element> block_V;
  cutlass::DeviceAllocation<Element> block_O;
  cutlass::DeviceAllocation<ElementAccumulator> block_LSE;

  cutlass::DeviceAllocation<Element> block_dQ;
  cutlass::DeviceAllocation<Element> block_dK;
  cutlass::DeviceAllocation<Element> block_dV;
  cutlass::DeviceAllocation<Element> block_dO;

  cutlass::DeviceAllocation<Element> block_ref_dQ;
  cutlass::DeviceAllocation<Element> block_ref_dK;
  cutlass::DeviceAllocation<Element> block_ref_dV;

  //
  // Methods
  //
  bool verify(const ProblemShapeType& problem_size) {
    auto [B, H, Q, K, D] = problem_size;

    Tensor mQ = make_tensor(make_gmem_ptr(block_Q.get()),
      make_shape(Q, D, make_shape(B, H)),
      make_stride(get<2>(stride_Q), get<3>(stride_Q), make_stride(get<0>(stride_Q), get<1>(stride_Q))));

    Tensor mK = make_tensor(make_gmem_ptr(block_K.get()),
      make_shape(K, D, make_shape(B, H)),
      make_stride(get<2>(stride_K), get<3>(stride_K), make_stride(get<0>(stride_K), get<1>(stride_K))));

    Tensor mV = make_tensor(make_gmem_ptr(block_V.get()),
      make_shape(K, D, make_shape(B, H)),
      make_stride(get<2>(stride_V), get<3>(stride_V), make_stride(get<0>(stride_V), get<1>(stride_V))));
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    Tensor mO = make_tensor(make_gmem_ptr(block_O.get()),
      make_shape(Q, D, make_shape(B, H)),
      make_stride(get<2>(stride_O), get<3>(stride_O), make_stride(get<0>(stride_O), get<1>(stride_O))));

    Tensor mLSE = make_tensor(make_gmem_ptr(block_LSE.get()),
      make_shape(Q, make_shape(B, H)),
      make_stride(get<2>(stride_LSE), make_stride(get<0>(stride_LSE), get<1>(stride_LSE))));

    Tensor mDQ = make_tensor(make_gmem_ptr(block_ref_dQ.get()),
      make_shape(Q, D, make_shape(B, H)),
      make_stride(get<2>(stride_dQ), get<3>(stride_dQ), make_stride(get<0>(stride_dQ), get<1>(stride_dQ))));

    Tensor mDK = make_tensor(make_gmem_ptr(block_ref_dK.get()),
      make_shape(K, D, make_shape(B, H)),
      make_stride(get<2>(stride_dK), get<3>(stride_dK), make_stride(get<0>(stride_dK), get<1>(stride_dK))));

    Tensor mDV = make_tensor(make_gmem_ptr(block_ref_dV.get()),
      make_shape(K, D, make_shape(B, H)),
      make_stride(get<2>(stride_dV), get<3>(stride_dV), make_stride(get<0>(stride_dV), get<1>(stride_dV))));
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    Tensor mDO = make_tensor(make_gmem_ptr(block_dO.get()),
      make_shape(Q, D, make_shape(B, H)),
      make_stride(get<2>(stride_dO), get<3>(stride_dO), make_stride(get<0>(stride_dO), get<1>(stride_dO))));
```

**EN**: This composition block packages lower-level collectives into a launchable kernel or device-level operation. In CUTLASS examples this is the point where compile-time pieces stop being isolated traits and become one executable path with a concrete problem shape interface. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一组合块把更底层的 collective 封装成可启动的 kernel 或设备级 operation。对 CUTLASS 示例来说，这正是编译期零件不再各自独立、而是收敛成一个具有明确问题形状接口的可执行路径的地方。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    fmha_bwd_reference(problem_size, mQ, mK, mV, mO, mLSE, mDO, mDQ, mDK, mDV, ActiveFusion{});
    cudaError_t result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Reference kernel failed. Last CUDA error: "
                << cudaGetErrorString(result) << std::endl;
      return false;
    }

    // Check if output from CUTLASS kernel and reference kernel are equal or not
    double max_diff = 0;
    double mean_diff = 0;
    reference_abs_diff(block_dQ, block_ref_dQ, max_diff, mean_diff);
    bool passed_dQ = (max_diff < 1e-2) && (mean_diff < 1e-3);
    if (! passed_dQ) {
      std::cerr << "failed dQ: max diff " << max_diff 
                << " mean " << mean_diff << std::endl;
    }

    reference_abs_diff(block_dK, block_ref_dK, max_diff, mean_diff);
    bool passed_dK = (max_diff < 1e-2) && (mean_diff < 1e-3);
    if (! passed_dK) {
      std::cerr << "failed dK: max diff " << max_diff 
                << " mean " << mean_diff << std::endl;
    }

    reference_abs_diff(block_dV, block_ref_dV, max_diff, mean_diff);
    bool passed_dV = (max_diff < 1e-2) && (mean_diff < 1e-3);
    if (! passed_dV) {
      std::cerr << "failed dV: max diff " << max_diff 
                << " mean " << mean_diff << std::endl;
    }

    return passed_dQ && passed_dK && passed_dV;
  }

  /// Initialize operands to be used in the GEMM and reference GEMM
  void initialize(const ProblemShapeType& problem_size) {
    auto [B, H, Q, K, D] = problem_size;
    D = cutlass::round_up(D, 8);  // Alignment
    Q = cutlass::round_up(Q, 8);  // Alignment
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    auto shape_QO = cute::make_shape(B, H, Q, D);
    auto shape_KV = cute::make_shape(B, H, K, D);
    auto shape_LSE = cute::make_shape(B, H, Q);

    stride_Q = cute::compact_row_major(shape_QO);
    stride_K = cute::compact_row_major(shape_KV);
    stride_V = cute::compact_row_major(shape_KV);
    stride_O = cute::compact_row_major(shape_QO);
    stride_LSE = cute::compact_row_major(shape_LSE);

    stride_dQ = stride_Q;
    stride_dK = stride_K;
    stride_dV = stride_V;
    stride_dO = stride_O;

    block_Q.reset(size(shape_QO));
    block_K.reset(size(shape_KV));
    block_V.reset(size(shape_KV));
    block_O.reset(size(shape_QO));
    block_LSE.reset(size(shape_LSE));

    block_dQ.reset(size(shape_QO));
    block_dK.reset(size(shape_KV));
    block_dV.reset(size(shape_KV));
    block_dO.reset(size(shape_QO));

    block_ref_dQ.reset(size(shape_QO));
    block_ref_dK.reset(size(shape_KV));
    block_ref_dV.reset(size(shape_KV));

    initialize_block(block_Q, seed + 2023, false);
    initialize_block(block_K, seed + 2022, false);
    initialize_block(block_V, seed + 2021, false);
    initialize_block(block_dO, seed + 2020, false);
    Tensor mQ = make_tensor(make_gmem_ptr(block_Q.get()),
      make_shape(Q, D, make_shape(B, H)),
      make_stride(get<2>(stride_Q), get<3>(stride_Q), make_stride(get<0>(stride_Q), get<1>(stride_Q))));

    Tensor mK = make_tensor(make_gmem_ptr(block_K.get()),
      make_shape(K, D, make_shape(B, H)),
      make_stride(get<2>(stride_K), get<3>(stride_K), make_stride(get<0>(stride_K), get<1>(stride_K))));
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    Tensor mV = make_tensor(make_gmem_ptr(block_V.get()),
      make_shape(K, D, make_shape(B, H)),
      make_stride(get<2>(stride_V), get<3>(stride_V), make_stride(get<0>(stride_V), get<1>(stride_V))));

    Tensor mO = make_tensor(make_gmem_ptr(block_O.get()),
      make_shape(Q, D, make_shape(B, H)),
      make_stride(get<2>(stride_O), get<3>(stride_O), make_stride(get<0>(stride_O), get<1>(stride_O))));

    Tensor mLSE = make_tensor(make_gmem_ptr(block_LSE.get()),
      make_shape(Q, make_shape(B, H)),
      make_stride(get<2>(stride_LSE), make_stride(get<0>(stride_LSE), get<1>(stride_LSE))));

    fmha_reference(problem_size, mQ, mK, mV, mO, mLSE, ActiveFusion{});
  }

  ExampleResult run(const Options& options, const cutlass::KernelHardwareInfo& hw_info) {
    ProblemShapeType problem_size = ProblemShapeType{options.b, options.h, options.q, options.k, options.d};

    initialize(problem_size);

    typename Operation::Arguments arguments{
      problem_size,
      block_Q.get(), stride_Q,
      block_K.get(), stride_K,
      block_V.get(), stride_V,
      block_O.get(), stride_O,
      block_LSE.get(), stride_LSE,
      block_dO.get(), stride_dO,
      block_dQ.get(), stride_dQ,
      block_dK.get(), stride_dK,
      block_dV.get(), stride_dV,
      hw_info
    };

    Operation op;

    ExampleResult example_result;

    example_result.smem_size = Operation::Operation::Kernel::SharedStorageSize;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Manages shared memory and, on blackwell, often tmem-backed intermediate state.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 管理共享内存，并在 Blackwell 上经常涉及 TMEM 中间状态。

---

```cpp
    size_t workspace_size = Operation::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

    cutlass::Status status = op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "This kernel is not supported. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return example_result;
    }

    status = op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return example_result;
    }

    // Run
    cudaMemset(block_dQ.get(), 0, block_dQ.size() * sizeof(Element));
    cudaMemset(block_dK.get(), 0, block_dK.size() * sizeof(Element));
    cudaMemset(block_dV.get(), 0, block_dV.size() * sizeof(Element));
    
    status = op.run();
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return example_result;
    }

    cudaError_t result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    //
    // Construct events
    //

    cudaEvent_t events[2];
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    for (auto & event : events) {
      result = cudaEventCreate(&event);
      if (result != cudaSuccess) {
        std::cerr << "cudaEventCreate() failed: " << cudaGetErrorString(result) << std::endl;
        return example_result;
      }
    }

    // Record an event at the start of a series of GEMMs
    result = cudaEventRecord(events[0]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    for (int i = 0; i < options.iterations; i++) {
      status = op.run();
      if (status != cutlass::Status::kSuccess) {
        std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                  << cudaGetErrorString(cudaGetLastError()) << std::endl;
        return example_result;
      }
    }

    //
    // Stop profiling loop
    //

    // Record an event when the GEMMs are complete
    result = cudaEventRecord(events[1]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    // Wait for work on the device to complete.
    result = cudaEventSynchronize(events[1]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventSynchronize() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    // Measure elapsed runtime
    float runtime_ms = 0;
    result = cudaEventElapsedTime(&runtime_ms, events[0], events[1]);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    runtime_ms /= static_cast<float>(options.iterations);

    double flops = 10.0 * (std::is_same_v<ActiveFusion, CausalFusion> ? 0.5 : 1.0);
    flops *= static_cast<double>(get<0>(problem_size));
    flops *= static_cast<double>(get<1>(problem_size));
    flops *= static_cast<double>(get<2>(problem_size));
    flops *= static_cast<double>(get<3>(problem_size));
    flops *= static_cast<double>(get<4>(problem_size));
    double tflops_s = flops * 1e-12 /*tera*/ / (runtime_ms * 1e-3 /*ms*/);
    example_result.tflops_s = tflops_s;
    example_result.runtime_ms = runtime_ms;

    result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return example_result;
    }

    // Verify that the result is correct
    bool passed = true;
    if (options.verify) {
      passed = verify(problem_size);
      if (passed) example_result.verified = true;
    }
    
    if (!passed) {
      std::cerr << "Reference check failed" << std::endl;
      return example_result;
    }

    example_result.passed = true;
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
    return example_result;
  }

};
```

**EN**: This storage block describes the shared-memory (and sometimes TMEM-related) state layout used by the kernel. Such declarations are performance-critical because they determine how pipeline stages, accumulator fragments, and epilogue scratch space coexist inside the CTA or cluster. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一存储块描述了内核所使用的共享内存（有时还包括 TMEM 相关）状态布局。它们是性能关键点，因为流水线阶段、累加器片段和 epilogue 临时空间如何在 CTA 或 cluster 内共存，都是由这里决定的。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
/// Helper to print a description of the example run and its result
void print_result(const std::string& description, ExampleResult result, bool verbose) {
  std::ios fmt(nullptr);
  fmt.copyfmt(std::cout);
  std::cout << (result.passed ? (result.verified ? " [OK]  " : " [--] ") : "[FAIL] ");
  std::cout << std::setw(32) << std::left << description;
  std::cout.copyfmt(fmt);
  std::cout << " : " << result.tflops_s << " TFLOPS/s" << std::endl;
  if (verbose) {
    std::cout << "       t=" << result.runtime_ms << "ms, "
        "smem=" << result.smem_size << "b" << std::endl;
  }
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
using KernelTma = cutlass::gemm::KernelTma;
using KernelCooperative = cutlass::gemm::KernelTmaWarpSpecializedCooperative;
using KernelPingpong = cutlass::gemm::KernelTmaWarpSpecializedPingpong;
```

**EN**: This configuration block defines compile-time aliases or constants that control data types, layouts, alignments, tile sizes, or enum-style modes. These choices are not cosmetic: in CUTLASS they steer which iterators, MMA atoms, and memory movement strategies become legal and efficient. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这个配置块定义了编译期别名或常量，用来控制数据类型、布局、对齐、tile 大小或枚举模式。这些选择并非表面参数：在 CUTLASS 中，它们会直接决定哪些迭代器、MMA atom 和数据搬运策略既合法又高效。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
template<class Fusion>
void run_fwd_32(Fusion fusion, Options const & options, cutlass::KernelHardwareInfo const& hw_info) {
  auto run = [&](auto shape, auto kernel, const char* name, auto... kernel_options) {
    FwdRunner<decltype(shape), decltype(kernel), Fusion, decltype(kernel_options)...> runner;
    auto result = runner.run(options, hw_info);
    print_result(name, result, options.verbose);
  };

  using HeadDim = _32;

  run(Shape< _64, _128, HeadDim>{}, KernelTma{}, "tma 64x128x32");
  run(Shape< _128, _64, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x64x32");
}

template<class Fusion>
void run_fwd_64(Fusion fusion, Options const & options, cutlass::KernelHardwareInfo const& hw_info) {
  auto run = [&](auto shape, auto kernel, const char* name, auto... kernel_options) {
    FwdRunner<decltype(shape), decltype(kernel), Fusion, decltype(kernel_options)...> runner;
    auto result = runner.run(options, hw_info);
    print_result(name, result, options.verbose);
  };

  using HeadDim = _64;

  run(Shape< _64, _128, HeadDim>{}, KernelTma{}, "tma 64x128x64");
  run(Shape< _128, _64, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x64x64");
  run(Shape< _128, _64, HeadDim>{}, KernelPingpong{}, "tma ws ping-pong 128x64x64");
}

template<class Fusion>
void run_fwd_128(Fusion fusion, Options const & options, cutlass::KernelHardwareInfo const& hw_info) {
  auto run = [&](auto shape, auto kernel, const char* name, auto... kernel_options) {
    FwdRunner<decltype(shape), decltype(kernel), Fusion, decltype(kernel_options)...> runner;
    auto result = runner.run(options, hw_info);
    print_result(name, result, options.verbose);
  };

  using HeadDim = _128;
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
  run(Shape<_128, _128, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x128x128");
#ifdef FP8
  run(Shape<_128, _256, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x256x128 acc fp16", Option<Tag::kAccQK, cutlass::half_t>{});
  run(Shape<_128, _256, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x256x128 acc fp32");
#endif
  run(Shape<_128, _128, HeadDim>{}, KernelPingpong{}, "tma ws ping-pong 128x128x128");
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
template<class Fusion>
void run_fwd_256(Fusion fusion, Options const & options, cutlass::KernelHardwareInfo const& hw_info) {
  auto run = [&](auto shape, auto kernel, const char* name, auto... kernel_options) {
    FwdRunner<decltype(shape), decltype(kernel), Fusion, decltype(kernel_options)...> runner;
    auto result = runner.run(options, hw_info);
    print_result(name, result, options.verbose);
  };

  using HeadDim = _256;

#ifdef FP8
  run(Shape<_128, _128, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x128x256");
  run(Shape<_128, _128, HeadDim>{}, KernelPingpong{}, "tma ws ping-pong 128x128x256");
#else
  run(Shape<_128, _64, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x64x256");
#endif
}

template<class Fusion>
void run_bwd_32(Fusion fusion, Options const & options, cutlass::KernelHardwareInfo const& hw_info) {
  auto run = [&](auto shape, auto kernel, const char* name, auto... kernel_options) {
    BwdRunner<decltype(shape), decltype(kernel), Fusion, decltype(kernel_options)...> runner;
    auto result = runner.run(options, hw_info);
    print_result(name, result, options.verbose);
  };

  using HeadDim = _32;

  run(Shape< _64, _128, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 64x128x32");
  run(Shape<_128, _128, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x128x32");
}

template<class Fusion>
void run_bwd_64(Fusion fusion, Options const & options, cutlass::KernelHardwareInfo const& hw_info) {
  auto run = [&](auto shape, auto kernel, const char* name, auto... kernel_options) {
    BwdRunner<decltype(shape), decltype(kernel), Fusion, decltype(kernel_options)...> runner;
    auto result = runner.run(options, hw_info);
    print_result(name, result, options.verbose);
  };

  using HeadDim = _64;
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
  run(Shape< _64, _128, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 64x128x64");
  run(Shape<_128, _128, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 128x128x64");
}

template<class Fusion>
void run_bwd_128(Fusion fusion, Options const & options, cutlass::KernelHardwareInfo const& hw_info) {
  auto run = [&](auto shape, auto kernel, const char* name, auto... kernel_options) {
    BwdRunner<decltype(shape), decltype(kernel), Fusion, decltype(kernel_options)...> runner;
    auto result = runner.run(options, hw_info);
    print_result(name, result, options.verbose);
  };

  using HeadDim = _128;
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
  run(Shape<_64, _128, HeadDim>{}, KernelCooperative{}, "tma ws cooperative 64x128x128");
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Relies on tma-style bulk movement or descriptor handling.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 依赖 TMA 风格的批量搬运或描述符处理。

---

```cpp
#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
```

**EN**: This preprocessor block gates architecture-specific code or closes a specialized compilation path. It keeps the file buildable across toolchains and GPU targets even when the most optimized path only exists on a subset of architectures. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一预处理块负责为特定架构的代码设置编译门槛，或者结束某条专门化路径。这样即便最优路径只存在于部分 GPU 架构上，文件依旧可以在不同工具链与目标上保持可编译。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
int main_single(int argc, char const **args) {

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (__CUDACC_VER_MAJOR__ < 12 || props.major < 9) {
    std::cout
      << "This example requires a GPU of NVIDIA's Hopper Architecture or "
      << "later (compute capability 90 or greater) and CUDA 12.0 or greater.\n";
    return 0;
  }
  
  else if (__CUDACC_VER_MAJOR__ < 12 || (props.major != 9 || props.minor != 0)) {
    std::cout
      << "This example requires a GPU of NVIDIA's Hopper Architecture "
      << "(compute capability 90) and CUDA 12.0 or greater.\n";
    return 0;
  }
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
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

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

  //
  // Run examples
  //

  // The KernelHardwareInfo struct holds the number of SMs on the GPU with a given device ID. This
  // information is used by the underlying kernel.
  cutlass::KernelHardwareInfo hw_info;

  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  std::cout << "###### B " << options.b << " H " << options.h << " Q " << options.q << " K " << options.k << " D " << options.d << " ";
  std::cout << (options.bwd ? "Backward" : "Forward") << " " << (options.causal ? "Causal" : "Full") << " ";
  std::cout << "#SM " << hw_info.sm_count << std::endl;
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
  auto with_fusion = [&](auto fn) {
    if (options.causal) {
      fn(CausalFusion{});
    } else if (options.residual){
      fn(ResidualFusion{});
    } else {
      fn(DefaultFusion{});
    }
  };

  with_fusion([&](auto fusion) {
    if (options.bwd) {
#ifndef FP8
      if (options.d <= 32) {
        run_bwd_32(fusion, options, hw_info);
      } else if (options.d <= 64) {
        run_bwd_64(fusion, options, hw_info);
      } else if (options.d <= 128) {
        run_bwd_128(fusion, options, hw_info);
      } else
#endif
      {
#ifdef FP8
        std::cout << "Backward is not implemented for FP8." << std::endl;
#else
        std::cout << "No backward kernel instantiated for d=" << options.d << std::endl;
#endif
      }
    } else {
#ifndef FP8
      if (options.d <= 32) {
        run_fwd_32(fusion, options, hw_info);
      } else
      if (options.d <= 64) {
        run_fwd_64(fusion, options, hw_info);
      } else
#endif
      if (options.d <= 128) {
        run_fwd_128(fusion, options, hw_info);
      } else
      if (options.d <= 256) {
        run_fwd_256(fusion, options, hw_info);
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Includes fp8/mxfp8-style narrow-precision behavior.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 包含 FP8/MXFP8 一类窄精度行为。

---

```cpp
      }
      else {
        std::cout << "No forward kernel instantiated for d=" << options.d << std::endl;
      }
    }
  });
#endif

  return 0;
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
int main(int argc, char const **args) {
  std::vector<std::string> full_arguments(args, args + argc);

  int result = 0;

  bool recursed = false;
  for (size_t i = 1; i < full_arguments.size(); i++) {
    if (full_arguments[i].find(',') != std::string::npos) {
      auto arg = full_arguments[i];
      size_t eq_pos = arg.find('=');
      std::string prefix = eq_pos == std::string::npos ? "" : arg.substr(0, eq_pos+1);
      std::string rest = eq_pos == std::string::npos ? arg : arg.substr(eq_pos+1);
      for (;;) {
        size_t comma_pos = rest.find(',');
        std::string current = rest.substr(0, comma_pos);
        full_arguments[i] = prefix + current;
        std::vector<const char*> next_args;
        for (auto& elem : full_arguments) { next_args.push_back(elem.data()); }
        main(argc, next_args.data());
        if (comma_pos == std::string::npos) break;
        rest = rest.substr(comma_pos+1);
      }
      recursed = true;
      break;
    }
  }

  if (! recursed) {
    main_single(argc, args);
  }

  return result;
}
```

**EN**: This block forms the outer control flow of the sample. It performs capability checks, selects the concrete specialization to run, invokes the operation, and reports the final status or throughput back to the user. Uses hopper-era gmma/tma building blocks. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一块构成了示例的最外层控制流：先做能力检查，再选择具体特化版本，调用 operation，最后把状态或吞吐结果回报给用户。使用 Hopper 时代的 GMMA/TMA 构件。 属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- FP8-family mixed-precision execution / FP8 家族混合精度执行
- TMA-driven data movement / 基于 TMA 的数据搬运
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层
- Reference-based numerical validation / 基于参考实现的数值验证

## Dependencies / 依赖项

- `iostream` — included dependency used by this example / 此示例使用的包含依赖
- `cute/tensor.hpp` — CuTe tensor and layout primitives / CuTe 张量与布局原语
- `cutlass/cutlass.h` — core CUTLASS types, architecture tags, and utilities / CUTLASS 核心类型、架构标签与工具
- `cutlass/util/command_line.h` — command-line parsing helpers / 命令行解析辅助
- `cutlass/util/distribution.h` — random/distribution-based tensor initialization / 随机与分布初始化辅助
- `cutlass/util/host_tensor.h` — host/device tensor allocation utilities / host/device 张量分配工具
- `cutlass/util/tensor_view_io.h` — tensor printing and inspection utilities / 张量打印与查看工具
- `cutlass/util/reference/device/tensor_fill.h` — device tensor fill helpers / 设备张量填充辅助
- `collective/fmha_fusion.hpp` — collective-stage helper for the attention/GEMM pipeline / 注意力或 GEMM 流水线的 collective 阶段辅助
- `device/fmha_device_bwd.hpp` — device wrapper or operation adapter / 设备封装或 operation 适配器
- `device/device_universal.hpp` — device wrapper or operation adapter / 设备封装或 operation 适配器
- `kernel/fmha_kernel_builder.hpp` — kernel-layer helper for launch-time execution / 启动期执行所需的内核层辅助
- `reference/fmha_reference.hpp` — reference path used for validation / 用于验证的参考路径
- `reference/fmha_bwd_reference.hpp` — reference path used for validation / 用于验证的参考路径
- `reference/reference_abs_error.hpp` — reference path used for validation / 用于验证的参考路径
