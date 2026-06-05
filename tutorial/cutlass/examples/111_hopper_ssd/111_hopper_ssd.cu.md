# 111_hopper_ssd.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/111_hopper_ssd.cu`  
**Purpose / 用途**: Host-side Hopper SSD example that builds tensors, runs the DeltaA cumsum prepass and SM90 SSD kernel, and verifies results against the reference implementation. / Hopper SSD 的主机端示例：负责构造张量、执行 DeltaA 累加预处理与 SM90 SSD 内核，并用参考实现校验结果。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-63 — License, includes, and compile guard / 许可证、头文件与编译保护
```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include <iostream>
#include "cutlass/util/command_line.h"

#include "cutlass/cutlass.h"

#include "cute/tensor.hpp"
#include "cute/layout.hpp"
#include "cutlass/kernel_hardware_info.hpp"

#include "thrust/universal_vector.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/device/tensor_fill.h"
#include "cutlass/util/reference/device/tensor_compare.h"

#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

#include "reference/reference_ssd_cumsum.hpp"
#include "reference/reference_ssd.hpp"

#include "cutlass/transform/device/transform_universal_adapter.hpp"

#include "device/ssd.hpp"
#include "kernel/sm90_ssd_kernel_builder.hpp"

using namespace cute;

```
**EN**: The file starts as a runnable example, so it pulls in command-line helpers, tensor/layout utilities, CUTLASS runtime pieces, and the local SSD reference and device-kernel headers. The `CUTLASS_ARCH_MMA_SM90_SUPPORTED` guard makes the SSD-specific code compile only when Hopper GMMA support is available.
**CN**: 这个文件是可执行示例，因此先引入命令行工具、张量/布局工具、CUTLASS 运行时组件，以及本地的 SSD 参考实现和设备内核头文件。`CUTLASS_ARCH_MMA_SM90_SUPPORTED` 保护宏确保只有在支持 Hopper GMMA 时才编译 SSD 相关逻辑。

### Lines 64-154 — Options structure and problem shape / 参数结构与问题规模
```cpp
// Command line options parsing
struct Options {

  using Element = cutlass::bfloat16_t;
  using ElementAcc = float;
  using ElementDA = float;
  static constexpr bool D_HAS_HDIM = true;
  static constexpr bool HAS_D = true;
  static constexpr bool HAS_Z = true;

  bool help;
  bool error;
  
  // All static number now
  int G = 2;
  int B = 3;
  int E = 2;
  int H = 2;
  // Reference kernel doesn't support dynamic C now.
  static constexpr auto C = Int<8>{};
  static constexpr auto D = Int<64>{};
  static constexpr auto L = Int<128>{};
  static constexpr auto N = Int<128>{};
  int EH = E * H;

  int iterations;
  bool verify;
  bool verbose;

  int warmups;
  bool measure;

  Options():
    help(false),
    error(false),
    iterations(1), verify(true),
    measure(false), warmups(3)
  {}

  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);

    Options defaults;

    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }

    cmd.get_cmd_line_argument("iterations", iterations, defaults.iterations);
    cmd.get_cmd_line_argument("G", G, defaults.G);
    cmd.get_cmd_line_argument("B", B, defaults.B);
    cmd.get_cmd_line_argument("E", E, defaults.E);
    cmd.get_cmd_line_argument("H", H, defaults.H);
    verbose = cmd.check_cmd_line_flag("verbose");
    verify = !(cmd.check_cmd_line_flag("without_verify"));

    EH = E*H;

    if (iterations > 1) {
      measure = true;
      verbose = true;
    }

    auto problem_shape = cute::make_tuple(G, B, EH, C, L, D, N);
    cute::print("problem_shape : "); cute::print(problem_shape); cute::print("\n");
  }

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "111_hopper_ssd\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --iterations=<int>          Benchmarking iterations.\n"
      << "  --without_verify            Don't verify the results.\n"
      << "  --verbose                   Print execution time per kernel\n"
      << "  --G=<int>                   Group\n"
      << "  --B=<int>                   Batch\n"
      << "  --E=<int>                   Expanded factor\n"
      << "  --H=<int>                   Number of heads\n"
      << "\n";

    return out;
  }

  
  auto get_problem_shape() const {
    return cute::make_tuple(G, B, EH, C, L, D, N);
  }
```
**EN**: `Options` collects the example configuration: element types, feature flags (`HAS_D`, `HAS_Z`), and the SSD dimensions `(G, B, EH, C, L, D, N)`. Parsing also enables benchmark mode when `iterations > 1`, and it prints the problem shape so the user can see which chunk count `C`, per-chunk length `L`, hidden dimension `D`, and state dimension `N` are being used.

A notable detail is that `G` is user-configurable here, but the later testbed asserts `g == 1`; mathematically, groups partition heads, yet this example currently documents and validates only the single-group path.
**CN**: `Options` 汇总了示例配置：数据类型、功能开关（`HAS_D`、`HAS_Z`）以及 SSD 的规模 `(G, B, EH, C, L, D, N)`。解析命令行后，如果 `iterations > 1` 就进入基准模式，并打印问题形状，让用户看到块数 `C`、每块长度 `L`、隐藏维 `D` 和状态维 `N`。

一个需要注意的细节是：这里允许配置 `G`，但后面的 `TestBed` 会断言 `g == 1`；从数学上讲，group 用来划分 heads，但这个示例当前只实现并验证了单 group 路径。

### Lines 155-214 — Canonical tensor layouts / 规范张量布局
```cpp

  // acceptable layout by cuDNN
  // x       [b, eh, d, c, l]
  // delta   [b, eh, c, l]
  // delta_A [b, eh, c, l]
  // B       [b,  g, n, c, l]
  // C       [b,  g, n, c, l]
  // y       [b, eh, d, c, l]
  // fstate  [b, eh, d, n]

  auto layoutX() const {
    auto layout = make_layout(make_shape(L, C, D, EH, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }

  auto layoutDelta() const {
    auto layout = make_layout(make_shape(L, C, EH, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }

  auto layoutDeltaA() const {
    auto layout = make_layout(make_shape(L, C, EH, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }

  auto layoutB() const {
    auto layout = make_layout(make_shape(L, C, N, G, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }

  auto layoutC() const {
    auto layout = make_layout(make_shape(L, C, N, G, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }

  auto layoutY() const {
    auto layout = make_layout(make_shape(L, C, D, EH, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }

  auto layoutF() const {
    auto layout = make_layout(make_shape(N, D, EH, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }

  auto layoutD() const {
    if constexpr (D_HAS_HDIM) {
      auto layout = make_layout(make_shape(D, EH));
      return make_layout(reverse(layout.shape()), reverse(layout.stride()));
    }
    else {
      auto layout = make_layout(make_shape(Int<1>{}, EH));
      return make_layout(reverse(layout.shape()), reverse(layout.stride()));
    }
  }

  auto layoutZ() const {
    auto layout = make_layout(make_shape(L, C, D, EH, B));
    return make_layout(reverse(layout.shape()), reverse(layout.stride()));
  }
```
**EN**: These helpers define the semantic layouts seen by the example and the reference code. The comments tie each tensor to the SSD formulation: `x` and `y` live on `[b, eh, d, c, l]`, `delta` and `delta_A` on `[b, eh, c, l]`, `B` and `C` on `[b, g, n, c, l]`, and `fstate` on `[b, eh, d, n]`.

This is the math-facing view of the algorithm: batch `b`, expanded head `eh`, chunk index `c`, in-chunk position `l`, hidden dimension `d`, and state dimension `n` are all spelled out explicitly.
**CN**: 这些辅助函数定义了示例和参考实现看到的“语义布局”。注释直接把每个张量映射到 SSD 公式：`x` 与 `y` 是 `[b, eh, d, c, l]`，`delta` 与 `delta_A` 是 `[b, eh, c, l]`，`B` 与 `C` 是 `[b, g, n, c, l]`，`fstate` 是 `[b, eh, d, n]`。

这部分体现的是面向数学公式的视角：批次 `b`、扩展头 `eh`、块编号 `c`、块内位置 `l`、隐藏维 `d` 和状态维 `n` 都被明确编码进布局中。

### Lines 215-308 — Kernel-transformed layouts / 面向内核的重排布局
```cpp

  // transformed layout for kernel parameters

  auto layoutX_transformed() const {
    auto layout = make_layout(make_shape(L,int32_t(C),D,EH*B));
    return make_layout(
      make_shape(D,L,int32_t(C),EH*B),
      make_stride(
        stride<2>(layout),
        stride<0>(layout),
        stride<1>(layout),
        stride<3>(layout)
      )
    );
  }

  auto layoutB_transformed() const {
    auto layout = make_layout(make_shape(L,int32_t(C),N,G*B));
    return make_layout(
      make_shape(L,N,int32_t(C),G*B),
      make_stride(
        stride<0>(layout),
        stride<2>(layout),
        stride<1>(layout),
        stride<3>(layout)
      )
    );
  }

  auto layoutC_transformed() const {
    auto layout = make_layout(make_shape(L,int32_t(C),N,G*B));
    return make_layout(
      make_shape(L,N,int32_t(C),G*B),
      make_stride(
        stride<0>(layout),
        stride<2>(layout),
        stride<1>(layout),
        stride<3>(layout)
      )
    );
  }

  auto layoutDelta_transformed() const {
    return make_layout(make_shape(L,int32_t(C),EH*B));
  }

  auto layoutY_transformed() const {
    auto layout = make_layout(make_shape(L,int32_t(C),D,EH*B));
    return make_layout(
      make_shape(L,D,int32_t(C),EH*B), // (M,K,L,...)
      make_stride(
        stride<0>(layout),
        stride<2>(layout),
        stride<1>(layout),
        stride<3>(layout)
      )
    );
  }

  auto layoutF_transformed() const {
    auto layout = make_layout(make_shape(N,D,EH*B));
    return make_layout(
      make_shape(D,N,EH*B),
      make_stride(
        stride<1>(layout),
        stride<0>(layout),
        stride<2>(layout)
      )
    );
  }

  auto layoutD_transformed() const {
    if constexpr (D_HAS_HDIM) {
      return make_layout(make_shape(D, EH));
    }
    else {
      return make_layout(make_shape(Int<1>{}, EH));
    }
  }

  auto layoutZ_transformed() const {
    auto layout = make_layout(make_shape(L,int32_t(C),D,EH*B));
    return make_layout(
      make_shape(L,D,int32_t(C),EH*B),
      make_stride(
        stride<0>(layout),
        stride<2>(layout),
        stride<1>(layout),
        stride<3>(layout)
      )
    );
  }

};
```
**EN**: The `_transformed()` accessors permute dimensions without changing the underlying SSD math. Their job is to expose memory orders that better match Hopper tile traversal, GMMA operand expectations, and coalesced loads/stores inside the device kernel.

For example, `X` is reinterpreted so `D` and `L` become prominent iteration dimensions, while `B`, `C`, `Y`, `F`, `D`, and `Z` are rearranged to match the kernel’s state-update and projection stages. This is where host orchestration meets tile scheduling: the example chooses layouts that let the kernel schedule chunk, sequence, hidden, and state work efficiently.
**CN**: 这些 `_transformed()` 接口只是在不改变 SSD 数学意义的前提下重排维度。目的在于暴露更适合 Hopper 瓦片遍历、GMMA 操作数形式以及设备端协同访存的内存顺序。

例如，`X` 被重新解释后会让 `D` 和 `L` 成为更关键的迭代维度；`B`、`C`、`Y`、`F`、`D`、`Z` 也都被改排，以配合内核中的状态更新与投影阶段。这正是主机端编排与 tile scheduling 的连接点：主机选择布局，设备内核才能高效调度 chunk、序列、隐藏维和状态维工作。

### Lines 309-341 — Input initialization helper / 输入初始化辅助函数
```cpp

template <typename Element>
static void
initialize_values(
    thrust::universal_vector<Element>& dst_ptr,
    cutlass::Distribution::Kind dist_kind,
    uint64_t seed,
    Element var = Element(1.f)) {
  if (cutlass::Distribution::Uniform == dist_kind) {
    int scope = 2;
    cutlass::reference::host::BlockFillRandomUniform(
        dst_ptr.data().get(), dst_ptr.size(), seed, scope, -scope, 0);
  }
  else if (cutlass::Distribution::AllZeros == dist_kind) {
    cutlass::reference::host::BlockFillRandomUniform(
        dst_ptr.data().get(), dst_ptr.size(), seed, 0, 0, 0);
  }
  else if (cutlass::Distribution::AllOnes == dist_kind) {
    cutlass::reference::host::BlockFillRandomUniform(
        dst_ptr.data().get(), dst_ptr.size(), seed, 1, 1, 0);
  } 
  else if (cutlass::Distribution::Gaussian == dist_kind) {
    cutlass::reference::device::BlockFillRandomGaussian(
      dst_ptr.data().get(), dst_ptr.size(), seed, (Element) 0, var);
  }
  else if (cutlass::Distribution::Sequential == dist_kind) {
    cutlass::reference::host::BlockFillSequential(dst_ptr.data().get(), dst_ptr.size());
  }
  else {
    std::cerr << "Invalid distribution kind!\n.";
    exit(1);
  }
}
```
**EN**: `initialize_values()` centralizes data generation for all tensors. Uniform and sequential patterns are useful for structure checks, while Gaussian `delta`/`delta_A` values give the recurrence nontrivial decay factors without making the reference and device results diverge too wildly.
**CN**: `initialize_values()` 统一管理所有张量的数据生成。Uniform 和 Sequential 便于做结构性检查，而高斯分布的 `delta`/`delta_A` 会给递推带来非平凡的衰减因子，同时又不至于让参考结果和设备结果差得太离谱。

### Lines 342-382 — TestBed storage and kernel aliases / TestBed 存储与内核别名
```cpp

template <
  class Options_
>
struct TestBed {
  using Option = Options_;
  using Element = typename Option::Element;
  using ElementDA = typename Option::ElementDA;
  using ElementAcc = typename Option::ElementAcc;

  thrust::universal_vector<Element> tensor_X;
  thrust::universal_vector<Element> tensor_DeltaA;
  thrust::universal_vector<ElementDA> tensor_DeltaA_cumsum;
  thrust::universal_vector<Element> tensor_Delta;
  thrust::universal_vector<Element> tensor_B;
  thrust::universal_vector<Element> tensor_C;
  thrust::universal_vector<Element> tensor_D;
  thrust::universal_vector<Element> tensor_Y;
  thrust::universal_vector<Element> tensor_Z;
  thrust::universal_vector<Element> tensor_Y_ref_0;
  thrust::universal_vector<Element> tensor_Y_ref_1;
  thrust::universal_vector<Element> tensor_F;
  thrust::universal_vector<Element> tensor_F_ref_0;
  thrust::universal_vector<Element> tensor_F_ref_1;

  cutlass::Distribution::Kind init_X      = cutlass::Distribution::Uniform;
  cutlass::Distribution::Kind init_DeltaA = cutlass::Distribution::Gaussian;
  cutlass::Distribution::Kind init_Delta  = cutlass::Distribution::Gaussian;
  cutlass::Distribution::Kind init_B      = cutlass::Distribution::Uniform;
  cutlass::Distribution::Kind init_C      = cutlass::Distribution::Uniform;

  using TileShape = decltype(make_shape(Options::L, Options::D, Options::N)); // (L, D, N)
  using SsdOperation = cutlass::ssd::device::SSD<
    typename cutlass::ssd::kernel::Sm90SsdBuilder<
      Element, ElementDA, ElementAcc, Element,
      TileShape,
      Option::HAS_D, Option::D_HAS_HDIM, Option::HAS_Z
    >::Kernel>;
  using CumsumKenrel = cutlass::ssd::kernel::CumsumKernel<Element, ElementDA, TileShape>;
  using CumsumOperation = cutlass::transform::device::TransformUniversalAdapter<CumsumKenrel>;

```
**EN**: `TestBed` owns every input, output, and reference tensor used by the example. It also fixes the SSD tile shape to `(L, D, N)` and defines two operations: the main SSD kernel built by `Sm90SsdBuilder`, and a separate `CumsumOperation` that computes the prefix sums of `DeltaA` into a higher-precision buffer.

The split between `Element` and `ElementDA` is important: the data path can stay in BF16, while recurrence scalars derived from `DeltaA` are accumulated in `float` for numerical stability.
**CN**: `TestBed` 持有示例所需的全部输入、输出和参考张量。它还把 SSD 的 tile 形状固定为 `(L, D, N)`，并定义了两个操作：由 `Sm90SsdBuilder` 构建的主 SSD 内核，以及单独计算 `DeltaA` 前缀和的 `CumsumOperation`。

`Element` 与 `ElementDA` 的拆分很关键：数据主路径可以保持 BF16，而由 `DeltaA` 派生出的递推标量则在 `float` 中累计，以提高数值稳定性。

### Lines 383-441 — Allocation, randomization, and cumsum arguments / 分配、随机化与 cumsum 参数
```cpp
  bool initialize(Options const& options, const cutlass::KernelHardwareInfo& hw_info, uint64_t seed = 2023) {
    auto [g, b, eh, c, l, d, n] = options.get_problem_shape();
    assert(g == 1 && "Only group size == 1 is supported") ;

    auto size_X      = b * eh * c * l * d;
    auto size_DeltaA = b * eh * c * l;
    auto size_Delta  = b * eh * c * l;
    auto size_B      = g * b * c * n * l;
    auto size_C      = g * b * c * n * l;
    auto size_Y      = b * eh * c * l * d;
    auto size_F      = b * eh * d * n;

    tensor_X      .resize(sizeof(Element) * size(options.layoutX()));
    tensor_DeltaA .resize(sizeof(Element) * size(options.layoutDeltaA()));
    tensor_Delta  .resize(sizeof(Element) * size(options.layoutDelta()));
    tensor_B      .resize(sizeof(Element) * size(options.layoutB()));
    tensor_C      .resize(sizeof(Element) * size(options.layoutC()));
    tensor_D      .resize(sizeof(Element) * size(options.layoutD()));
    tensor_Z      .resize(sizeof(Element) * size(options.layoutZ()));
    tensor_Y      .resize(sizeof(Element) * size(options.layoutY()));
    tensor_Y_ref_0.resize(sizeof(Element) * size(options.layoutY()));
    tensor_Y_ref_1.resize(sizeof(Element) * size(options.layoutY()));
    tensor_F      .resize(sizeof(Element) * size(options.layoutF()));
    tensor_F_ref_0.resize(sizeof(Element) * size(options.layoutF()));
    tensor_F_ref_1.resize(sizeof(Element) * size(options.layoutF()));

    tensor_DeltaA_cumsum.resize(sizeof(ElementDA) * size(options.layoutDeltaA()));

    // Limit distribution to reduce skew between hosts and devices
    initialize_values(tensor_X, init_X, seed);
    initialize_values(tensor_DeltaA, init_DeltaA, seed + 1, Element(0.05f));
    initialize_values(tensor_Delta, init_Delta, seed + 3, Element(0.05f));
    initialize_values(tensor_B, init_B, seed + 5);
    initialize_values(tensor_C, init_C, seed + 7);
    initialize_values(tensor_D, init_C, seed + 9);
    initialize_values(tensor_Z, init_X, seed);

    cudaError_t result;
    result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the Initialization kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
    }

    // apply cumsum(device) before kernel launch
    typename CumsumOperation::Arguments arguments{
      make_shape(int(b), int(eh), int(c), int(l)),
      {
        tensor_DeltaA.data().get(),
        tensor_DeltaA_cumsum.data().get(),
      },
      hw_info
    };

    CumsumOperation op;

    size_t workspace_size = CumsumOperation::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

```
**EN**: `initialize()` allocates all tensors using the semantic layouts, fills them with data, and synchronizes once to catch initialization errors. It then builds the argument pack for the cumsum kernel that transforms raw `DeltaA[b, eh, c, l]` into a float prefix-sum buffer before the main SSD launch.

Algorithmically, this is the prepass that turns long products of exponentials into differences of prefix sums. Instead of recomputing a recurrence factor inside every tile, the kernel can later recover semiseparable decay terms from the precomputed cumulative representation.
**CN**: `initialize()` 按语义布局分配所有张量、填充数据，并先做一次同步以尽早发现初始化错误。随后它构造 cumsum 内核的参数包，把原始的 `DeltaA[b, eh, c, l]` 先转换成 `float` 前缀和缓冲区，再交给主 SSD 内核使用。

从算法角度看，这一步就是前处理：把长链式的指数乘积改写成前缀和差值。这样主内核就不必在每个 tile 内重复维护完整递推，而是可以从预计算的累计表示中恢复半可分结构的衰减项。

### Lines 442-514 — Launching and timing the cumsum prepass / 启动并计时 cumsum 预处理
```cpp
    cutlass::Status status = op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "This kernel is not supported. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return false;
    }

    status = op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return false;
    }

    // may be used uninitialized 
    cudaEvent_t start;
    cudaEvent_t end;
    cudaEventCreate(&start);
    cudaEventCreate(&end);

    // warm up
    if (options.measure) {
      for (int i = 0; i < options.warmups; i++) {
        status = op.run();
        if (status != cutlass::Status::kSuccess) {
          std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                    << cudaGetErrorString(cudaGetLastError()) << std::endl;
          return false;
        }
      }
    }
    result = cudaEventRecord(start);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return false;
    }
    // Run
    for (int i = 0; i < options.iterations; i++) {
      status = op.run();
      if (status != cutlass::Status::kSuccess) {
        std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                  << cudaGetErrorString(cudaGetLastError()) << std::endl;
        return false;
      }
    }
    result = cudaEventRecord(end);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return false;
    }

    float runtime_ms = 0;
    result = cudaEventElapsedTime(&runtime_ms, start, end);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result) << std::endl;
      return false;
    }
    runtime_ms /= static_cast<float>(options.iterations);

    if (options.verbose) {
      printf("[iters = %d, warmups = %d] cumsum kernel runtime_ms = %.4f\n", options.iterations, options.warmups, runtime_ms);
    }

    return true;
  }

```
**EN**: This block validates the cumsum kernel, initializes workspace, optionally warms it up, and measures runtime with CUDA events. The operation itself is small but conceptually critical: it prepares the recurrence metadata that the main SSD kernel consumes.
**CN**: 这一段完成 cumsum 内核的能力检查、工作区初始化、可选预热，以及基于 CUDA event 的计时。这个操作本身规模不大，但在概念上非常关键：它准备了主 SSD 内核需要读取的递推元数据。

### Lines 515-530 — Device sufficiency check / 设备能力检查
```cpp
  bool sufficient() const {
    int device_idx;
    cudaError_t result = cudaGetDevice(&device_idx);
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaGetDevice() API call failed.");
    }

    int max_smem_size;
    result = cudaDeviceGetAttribute(&max_smem_size, cudaDevAttrMaxSharedMemoryPerBlockOptin, device_idx);
    if (result != cudaSuccess) {
      throw std::runtime_error("cudaDeviceGetAttribute() failed");
    }

    return true;
  }

```
**EN**: `sufficient()` queries the active device and the maximum opt-in shared-memory size. In this example it always returns `true`, so it behaves as a lightweight capability probe rather than a hard admission test.
**CN**: `sufficient()` 查询当前设备以及可申请的最大共享内存大小。这个示例里它最终总是返回 `true`，因此更像是一个轻量级能力探测，而不是严格的准入检查。

### Lines 531-568 — Preparing main SSD kernel arguments / 构造主 SSD 内核参数
```cpp
  bool run(Options const& options, const cutlass::KernelHardwareInfo& hw_info) {
    if (!sufficient()) {
      std::cerr << "Test waived due to insufficient CUDA device.\n";
      return true;
    }

    if (!initialize(options, hw_info)) {
      std::cerr << "Failed to initialize the test.\n";
      return true;
    };

    auto [g, b, eh, c, l, d, n] = options.get_problem_shape();
    typename SsdOperation::Arguments arguments{
      make_shape(int(g), int(b), int(eh), int(c), int(l), int(d), int(n)),
      { 
        tensor_X.data().get(),
        tensor_DeltaA_cumsum.data().get(),
        tensor_Delta.data().get(),
        tensor_B.data().get(),
        tensor_C.data().get(),
        options.layoutX_transformed(),
        options.layoutB_transformed(),
        options.layoutC_transformed(),
        options.layoutDelta_transformed()
      },
      { 
        tensor_Y.data().get(),
        tensor_F.data().get(),
        tensor_D.data().get(),
        tensor_Z.data().get(),
        options.layoutY_transformed(),
        options.layoutF_transformed(),
        options.layoutD_transformed(),
        options.layoutZ_transformed()
      },
      hw_info
    };

```
**EN**: `run()` packages the transformed input/output layouts together with raw pointers for `X`, `DeltaA_cumsum`, `Delta`, `B`, `C`, `Y`, `F`, `D`, and `Z`. This is the host-side contract for the actual SSD forward pass: chunk-local computation and chunk-to-chunk state passing are hidden inside the device kernel, while the host only provides a consistent view of memory and problem shape.
**CN**: `run()` 将变换后的输入/输出布局与 `X`、`DeltaA_cumsum`、`Delta`、`B`、`C`、`Y`、`F`、`D`、`Z` 的原始指针一起打包。这里定义了真实 SSD 前向计算的主机端契约：chunk 内计算和 chunk 间状态传递都封装在设备内核内部，而主机端只负责提供一致的内存视图和问题规模。

### Lines 569-644 — Launching and timing the main SSD kernel / 启动并计时主 SSD 内核
```cpp
    SsdOperation op;

    size_t workspace_size = SsdOperation::get_workspace_size(arguments);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

    cutlass::Status status = op.can_implement(arguments);
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "This kernel is not supported. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return false;
    }

    status = op.initialize(arguments, workspace.get());
    if (status != cutlass::Status::kSuccess) {
      std::cerr << "Failed to initialize the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(cudaGetLastError()) << std::endl;
      return false;
    }

    cudaError_t result;
    // may be used uninitialized 
    cudaEvent_t start;
    cudaEvent_t end;
    cudaEventCreate(&start);
    cudaEventCreate(&end);

    // warm up
    if (options.measure) {
      for (int i = 0; i < options.warmups; i++) {
        status = op.run();
        if (status != cutlass::Status::kSuccess) {
          std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                    << cudaGetErrorString(cudaGetLastError()) << std::endl;
          return false;
        }
      }
    }
    result = cudaEventRecord(start);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return false;
    }
    // Run
    for (int i = 0; i < options.iterations; i++) {
      status = op.run();
      if (status != cutlass::Status::kSuccess) {
        std::cerr << "Failed to launch the CUTLASS kernel. Last CUDA error is: "
                  << cudaGetErrorString(cudaGetLastError()) << std::endl;
        return false;
      }
    }
    result = cudaEventRecord(end);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventRecord() failed: " << cudaGetErrorString(result) << std::endl;
      return false;
    }

    result = cudaDeviceSynchronize();
    if (result != cudaSuccess) {
      std::cerr << "Error running the CUTLASS kernel. Last CUDA error is: "
                << cudaGetErrorString(result) << std::endl;
      return false;
    }

    float runtime_ms = 0;
    result = cudaEventElapsedTime(&runtime_ms, start, end);
    if (result != cudaSuccess) {
      std::cerr << "cudaEventElapsed() failed: " << cudaGetErrorString(result) << std::endl;
      return false;
    }
    runtime_ms /= static_cast<float>(options.iterations);

    if (options.verbose) {
      printf("[iters = %d, warmups = %d] ssd kernel runtime_ms = %.4f\n", options.iterations, options.warmups, runtime_ms);
      printf("smem size = %d\n", SsdOperation::Kernel::SharedStorageSize);
    }
```
**EN**: This section mirrors the cumsum launch path: capability check, initialization, warmup, repeated execution, synchronization, and timing. The printed shared-memory size is especially relevant on Hopper because the SSD kernel uses carefully staged tiles to balance state reuse, GMMA throughput, and occupancy.
**CN**: 这一段与 cumsum 的启动流程基本对称：先检查能力，再初始化、预热、多次执行、同步并计时。打印出的共享内存大小在 Hopper 上尤其重要，因为 SSD 内核通常会通过精心设计的分块与流水来平衡状态复用、GMMA 吞吐和占用率。

### Lines 645-682 — Wrapping tensors and calling the reference path / 包装张量并调用参考路径
```cpp
    // Matrix
    // x       [b, eh, d, c, l]
    // delta   [b, eh, c, l]
    // delta_A [b, eh, c, l]
    // B       [b,  g, n, c, l]
    // C       [b,  g, n, c, l]
    // y       [b, eh, d, c, l]
    // fstate  [b, eh, d, n]
    auto mY_ref_0 = cute::make_tensor(tensor_Y_ref_0.data().get(), options.layoutY());
    auto mY_ref_1 = cute::make_tensor(tensor_Y_ref_1.data().get(), options.layoutY());
    auto mY_res   = cute::make_tensor(tensor_Y.data().get(),       options.layoutY());
    auto mF_ref_0 = cute::make_tensor(tensor_F_ref_0.data().get(), options.layoutF());
    auto mF_ref_1 = cute::make_tensor(tensor_F_ref_1.data().get(), options.layoutF());
    auto mF_res   = cute::make_tensor(tensor_F.data().get(),       options.layoutF());
    auto mX       = cute::make_tensor(tensor_X.data().get(),       options.layoutX());
    auto mB       = cute::make_tensor(tensor_B.data().get(),       options.layoutB());
    auto mC       = cute::make_tensor(tensor_C.data().get(),       options.layoutC());
    auto mD       = cute::make_tensor(tensor_D.data().get(),       options.layoutD());
    auto mZ       = cute::make_tensor(tensor_Z.data().get(),       options.layoutZ());
    auto mDelta   = cute::make_tensor(tensor_Delta.data().get(),   options.layoutDelta());
    auto mDeltaA  = cute::make_tensor(tensor_DeltaA.data().get(),  options.layoutDeltaA());

    // Reference Device kernel
    if (options.verify) {
      ssd_reference<Option::HAS_D, Option::D_HAS_HDIM, Option::HAS_Z>(
        mY_ref_1,
        mF_ref_1,
        mX,
        mDelta,
        mDeltaA,
        mB,
        mC,
        mD,
        mZ,
        options
      );
    }

```
**EN**: After execution, raw buffers are wrapped as CuTe tensors using the semantic layouts so they can be compared and interpreted mathematically. If verification is enabled, `ssd_reference()` computes the same SSD forward pass in an explicit, readable form: chunk-local semiseparable mixing, inter-chunk state recurrence, optional direct term `D`, optional gate `Z`, and final state output `F`.
**CN**: 内核执行完成后，原始缓冲区会按语义布局包装成 CuTe 张量，便于比较和按数学含义解释。如果开启校验，`ssd_reference()` 会用显式、可读的方式重新计算一次 SSD 前向：包括 chunk 内半可分混合、chunk 间状态递推、可选直通项 `D`、可选门控 `Z`，以及最终状态输出 `F`。

### Lines 683-776 — Reference comparison and diagnostics / 参考比较与诊断输出
```cpp
    bool passed = true;

    if (options.verify) {
      printf("[TensorY]verifying...\n");
      passed &= compare_reference<5>(mY_ref_1, mY_res);
      printf("[TensorF]verifying...\n");
      passed &= compare_reference<4>(mF_ref_1, mF_res);
    }

    return passed;
  }

  template<
    int TensorDim,
    class Engine, class Layout
  >
  static constexpr bool
  compare_reference(
      cute::Tensor<Engine, Layout> const& reference,
      cute::Tensor<Engine, Layout> const& computed,
      float epsilon = 0.05f) {
    if (size(reference) != size(computed)) {
      return false;
    }

    bool passed = true;
    if (epsilon == 0.0f) {
      // fast refcheck w/o epsilon
      for (size_t i = 0; i < size_t(size(reference)); ++i) {
        if (reference(i) != computed(i)) {
          passed = false;
          printf("[%llu] %f, %f\n", static_cast<unsigned long long>(i),
            float(reference(i)), float(computed(i)));
          break;
        }
      }
    }
    else {
      // refcheck with epsilon
      for (size_t i = 0; i < size_t(size(reference)); ++i) {
        auto ref = static_cast<float>(reference(i));
        auto act = static_cast<float>(computed(i));
        auto abs_error = std::abs(act - ref);
        auto rel_error = abs_error / (std::max(std::abs(act), std::abs(ref)) + 0.00001f);
        if (std::isnan(abs_error) || std::isnan(rel_error) ||
            std::min(rel_error, abs_error) > epsilon) {
          passed = false;
          printf("[%llu] %f, %f\n", static_cast<unsigned long long>(i),
            float(reference(i)), float(computed(i)));
          break;
        }
      }
    }
    if (not passed) {
      // x       [b, eh, d, c, l]
      // delta   [b, eh, c, l]
      // delta_A [b, eh, c, l]
      // B       [b,  g, n, c, l]
      // C       [b,  g, n, c, l]
      // y       [b, eh, d, c, l]
      // fstate  [b, eh, d, n]
      auto m = cute::shape<2>(reference);
      auto n = cute::shape<TensorDim-1>(reference);
      printf("reference:\n");
      for (int mi = 0; mi < m; ++mi) {
        for (int ni = 0; ni < n; ++ni) {
          if constexpr (TensorDim == 5) {
            printf("%.4f ", static_cast<float>(reference(0,0,mi,2,ni)));
          }
          else {
            printf("%.4f ", static_cast<float>(reference(0,0,mi,ni)));
          }
        }
        printf("\n");
      }
      printf("\n");
      printf("computed:\n");
      for (int mi = 0; mi < m; ++mi) {
        for (int ni = 0; ni < n; ++ni) {
          if constexpr (TensorDim == 5) {
            printf("%.4f ", static_cast<float>(computed(0,0,mi,2,ni)));
          }
          else {
            printf("%.4f ", static_cast<float>(computed(0,0,mi,ni)));
          }
        }
        printf("\n");
      }
      printf("\n");
    }
    return passed;
  }
};

```
**EN**: `compare_reference()` performs either exact or epsilon-based checks and stops at the first mismatch. When verification fails it prints a representative slice of the tensor, which helps identify whether the error comes from the output path `Y` or the carried state `F`.
**CN**: `compare_reference()` 支持精确比较和带容差的比较，并在第一次发现不匹配时立即停止。若校验失败，它会打印一个具有代表性的张量切片，便于判断问题更可能出在输出路径 `Y` 还是状态传递结果 `F` 上。

### Lines 777-801 — Architecture and CUDA version gate / 架构与 CUDA 版本检查
```cpp
#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

int main(int argc, char const **args) {

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
**EN**: The example refuses to run unless it sees CUDA 12+ and a Hopper SM90 GPU. That restriction is not about SSD math itself; it reflects that the showcased device kernel is specialized for Hopper GMMA instructions.
**CN**: 这个示例只有在检测到 CUDA 12+ 和 Hopper SM90 GPU 时才会继续运行。这个限制并不是 SSD 数学本身需要，而是因为这里展示的设备内核专门面向 Hopper 的 GMMA 指令实现。

### Lines 802-849 — Program entry point / 程序入口
```cpp
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

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

  // Execute kernel

  printf("start testing....\n");

  // The KernelHardwareInfo struct holds the number of SMs on the GPU with a given device ID. This
  // information is used by the underlying kernel.
  cutlass::KernelHardwareInfo hw_info;

  // Change device_id to another value if you are running on a machine with multiple GPUs and wish
  // to use a GPU other than that with device ID 0.
  hw_info.device_id = 0;
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  // Check Device/Host ref kernel
  TestBed<Options> testbed{};
  bool passed = testbed.run(options, hw_info);

  if (passed) {
    printf("everything is ok.\n");
  }
  else {
    printf("something is wrong!!!!!\n");
  }

#endif // defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)

  return 0;
}
```
**EN**: `main()` parses options, queries `KernelHardwareInfo`, constructs the testbed, and runs the full pipeline. In other words, this file is the host orchestrator for the example: it does setup, prepass launch, kernel launch, validation, and user-facing reporting, while the real SSD math is delegated to the reference code and the SM90 kernel.
**CN**: `main()` 负责解析参数、查询 `KernelHardwareInfo`、构造 `TestBed`，并执行整条流程。换句话说，这个文件就是示例的主机端调度器：完成初始化、预处理启动、主内核启动、结果校验和面向用户的输出，而真正的 SSD 数学计算则分别由参考代码和 SM90 内核承担。

---

## Key Concepts / 关键概念

- Host orchestration versus device execution / 主机端编排与设备端执行分离
- Prefix-sum prepass for `DeltaA` / 面向 `DeltaA` 的前缀和预处理
- Chunked SSD problem shape `(G, B, EH, C, L, D, N)` / 分块 SSD 问题规模 `(G, B, EH, C, L, D, N)`
- Layout permutation for Hopper tile scheduling / 为 Hopper 瓦片调度服务的布局重排
- Verification against an explicit semiseparable reference / 与显式半可分参考实现对比验证

## Dependencies / 依赖项

- `cutlass/util/command_line.h` — parses example CLI options / 解析命令行参数
- `reference/reference_ssd_cumsum.hpp` — provides the standalone cumsum prepass kernel / 提供独立的 cumsum 预处理内核
- `reference/reference_ssd.hpp` — provides the readable SSD reference implementation / 提供可读的 SSD 参考实现
- `cutlass/transform/device/transform_universal_adapter.hpp` — wraps the cumsum kernel as a CUTLASS device operation / 将 cumsum 内核包装成 CUTLASS 设备操作
- `device/ssd.hpp` — declares the SSD device operation launched by the example / 声明示例启动的 SSD 设备操作
- `kernel/sm90_ssd_kernel_builder.hpp` — builds the Hopper SM90 SSD kernel specialization / 构建 Hopper SM90 SSD 内核特化
- `cutlass/kernel_hardware_info.hpp` — passes device-id and SM-count metadata to kernels / 向内核传递设备与 SM 数信息
