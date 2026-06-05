# 112_blackwell_ssd.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/112_blackwell_ssd/112_blackwell_ssd.cu`  
**Purpose / 用途**: Host-side Blackwell SSD example that prepares layouts, runs the DeltaA cumsum prepass and SM100 SSD kernel, and verifies outputs against the reference path. / Blackwell SSD 的主机端示例：负责准备布局、执行 DeltaA 累加预处理与 SM100 SSD 内核，并用参考路径校验输出。

---

## Line-by-Line Analysis / 逐行分析

### 1. Lines 1-62 | License, includes, and architecture guard | 许可证、头文件与架构门控

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

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)

#include "reference/reference_ssd_cumsum.hpp"
#include "reference/reference_ssd.hpp"

#include "cutlass/transform/device/transform_universal_adapter.hpp"

#include "device/ssd.hpp"
#include "kernel/sm100_ssd_kernel_builder.hpp"

using namespace cute;
```

- **EN:** The opening block pulls in the host utilities, CuTe layout/tensor helpers, reference implementations, and the SM100 SSD kernel pieces. The `#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)` guard makes the example compile only when Blackwell MMA support is enabled.

- **CN:** 开头这一段引入主机工具、CuTe 布局/张量辅助、参考实现以及 SM100 SSD kernel 组件。`#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)` 让示例只在启用了 Blackwell MMA 支持时才编译相关路径。

### 2. Lines 63-102 | Option types, feature flags, and fixed problem sizes | 选项类型、功能开关与固定问题规模

```cpp

// Command line options parsing
struct Options {

  using Element = cutlass::bfloat16_t;
  using ElementAcc = float;
  using ElementDA = float;
  static constexpr bool D_HAS_HDIM = true;
  static constexpr bool HAS_D = true;
  // Blackwell SSD doesn't support Z now(huge perf drop).
  static constexpr bool HAS_Z = false;

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
```

- **EN:** `Options` chooses bf16 inputs with fp32 accumulation/cumsum, enables residual `D`, disables `Z`, and fixes the SSD tile sizes `C=8`, `L=128`, `D=64`, `N=128`. Runtime parameters only change the batch/group/head-side counts (`G`, `B`, `E`, `H`).

- **CN:** `Options` 选择 bf16 输入、fp32 累加/前缀和，开启残差 `D`，关闭 `Z`，并固定 SSD 的分块尺寸 `C=8`、`L=128`、`D=64`、`N=128`。运行时只允许改变批量/分组/头相关的计数（`G`、`B`、`E`、`H`）。

### 3. Lines 103-132 | Command-line parsing and benchmark mode selection | 命令行解析与基准模式选择

```cpp

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
```

- **EN:** The parser reads user overrides, derives `EH = E * H`, and automatically turns on measurement/verbose mode when multiple iterations are requested. It also prints the final problem tuple so the runtime shape is visible before launching kernels.

- **CN:** 解析器读取用户覆盖项，推导 `EH = E * H`，并在迭代次数大于 1 时自动开启计时/详细输出模式。它还会打印最终的问题形状元组，便于在启动 kernel 前看到实际运行规模。

### 4. Lines 133-155 | Usage text and problem-shape accessor | 帮助信息与问题形状访问器

```cpp

  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {

    out << "112_blackwell_ssd\n\n"
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

- **EN:** This small block exposes the CLI help text and returns the canonical SSD shape tuple `(G, B, EH, C, L, D, N)`. That tuple is the central contract reused by both host launch code and the reference path.

- **CN:** 这一小段提供命令行帮助文本，并返回规范的 SSD 形状元组 `(G, B, EH, C, L, D, N)`。这个元组是主机启动代码和参考实现共享的核心约定。

### 5. Lines 156-215 | Logical tensor layouts in external order | 外部逻辑顺序下的张量布局

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

- **EN:** These helpers describe the public tensor views for `X`, `Delta`, `DeltaA`, `B`, `C`, `Y`, `F`, `D`, and `Z`. The code builds shapes in a convenient internal order and then reverses shape/stride so callers see the intended cuDNN-like layouts such as `[b, eh, d, c, l]`.

- **CN:** 这些辅助函数定义了 `X`、`Delta`、`DeltaA`、`B`、`C`、`Y`、`F`、`D`、`Z` 的对外张量视图。代码先用便于构造的内部顺序创建 shape，再通过反转 shape/stride 让调用者看到期望的、类似 cuDNN 的逻辑布局，例如 `[b, eh, d, c, l]`。

### 6. Lines 216-309 | Kernel-facing transformed layouts | 面向内核的变换后布局

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

- **EN:** The transformed layouts collapse batch/head or batch/group dimensions and reorder axes so the SSD kernel receives its preferred memory contract, e.g. `X` as `(D, L, C, EH*B)` and `F` as `(D, N, EH*B)`. This is the bridge between user-friendly host tensors and the hardware-tuned Blackwell kernel.

- **CN:** 变换后的布局会折叠 batch/head 或 batch/group 维度，并重排坐标轴，使 SSD kernel 获得它偏好的内存契约，例如把 `X` 组织成 `(D, L, C, EH*B)`，把 `F` 组织成 `(D, N, EH*B)`。这就是“用户友好”的主机张量与“面向硬件调优”的 Blackwell kernel 之间的桥梁。

### 7. Lines 310-342 | Host-side tensor initialization helper | 主机侧张量初始化辅助函数

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

- **EN:** `initialize_values` centralizes how each tensor is filled. Uniform initialization is used for data-like tensors, while `Delta` and `DeltaA` can be Gaussian with a small variance to keep exponentials and scans numerically well behaved.

- **CN:** `initialize_values` 统一管理各张量的填充方式。数据类张量通常使用均匀分布，而 `Delta` 和 `DeltaA` 可以用较小方差的高斯分布，以让指数运算和扫描在数值上更稳定。

### 8. Lines 343-383 | TestBed state and kernel type aliases | TestBed 状态与 kernel 类型别名

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
    typename cutlass::ssd::kernel::Sm100SsdBuilder<
      Element, ElementDA, ElementAcc, Element,
      TileShape,
      Option::HAS_D, Option::D_HAS_HDIM
    >::Kernel>;
  using CumsumKenrel = cutlass::ssd::kernel::CumsumKernel<Element, ElementDA, TileShape>;
  using CumsumOperation = cutlass::transform::device::TransformUniversalAdapter<CumsumKenrel>;
```

- **EN:** `TestBed` owns all input/output/reference buffers and binds the concrete operation types. In particular it instantiates the SM100 SSD device operator and the auxiliary `CumsumKernel` wrapped by `TransformUniversalAdapter`.

- **CN:** `TestBed` 持有全部输入/输出/参考缓冲区，并绑定具体的算子类型。特别是，这里实例化了 SM100 SSD 设备算子，以及通过 `TransformUniversalAdapter` 封装的辅助 `CumsumKernel`。

### 9. Lines 384-513 | Allocation, randomization, and `DeltaA` cumulative-sum preprocessing | 分配、随机初始化与 `DeltaA` 累计和预处理

```cpp
  bool initialize(Options const& options, const cutlass::KernelHardwareInfo& hw_info, uint64_t seed = 2024) {
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
```

- **EN:** This is the first major orchestration stage: allocate buffers from the declared layouts, fill them, synchronize, and then launch the cumsum transform over shape `(b, eh, c, l)`. The timing logic here measures the preprocessing step separately from the main SSD kernel.

- **CN:** 这是第一个主要的编排阶段：根据声明的布局分配缓冲区，完成随机初始化与同步，然后对 `(b, eh, c, l)` 形状启动 cumsum 变换。这里的计时逻辑会把这一步预处理与后续 SSD 主核分开统计。

### 10. Lines 514-530 | Lightweight device sufficiency check | 轻量级设备可用性检查

```cpp
  }

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

- **EN:** `sufficient()` queries the active device and one shared-memory attribute, but currently returns `true` once those calls succeed. In other words, it acts more like an environment sanity check than a strict capability filter.

- **CN:** `sufficient()` 查询当前设备及一个共享内存属性，但只要这些调用成功就直接返回 `true`。也就是说，它更像环境健全性检查，而不是严格的能力过滤。

### 11. Lines 531-568 | Build SSD arguments from transformed tensors | 用变换后的张量构造 SSD 参数

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
        // tensor_Z.data().get(),
        options.layoutY_transformed(),
        options.layoutF_transformed(),
        options.layoutD_transformed(),
        // options.layoutZ_transformed()
      },
      hw_info
    };
```

- **EN:** The host packages the problem shape, raw pointers, and transformed layouts into `SsdOperation::Arguments`. Inputs and outputs are kept in separate sub-structures, and `D` is passed as the enabled residual term while `Z` stays commented out because this example disables it.

- **CN:** 主机把问题形状、原始指针和变换后布局打包进 `SsdOperation::Arguments`。输入与输出分别放在独立子结构中；`D` 作为已启用的残差项传入，而 `Z` 由于本示例禁用该路径而保持注释状态。

### 12. Lines 569-645 | Main SSD launch, workspace, and timing | 主 SSD 启动、工作区与计时

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

- **EN:** This block checks whether the Blackwell SSD kernel can implement the requested problem, allocates workspace, performs warmups, times `op.run()`, and prints runtime plus shared-memory footprint when verbose mode is enabled.

- **CN:** 这一段检查 Blackwell SSD kernel 是否能实现当前问题，分配工作区，执行预热，计时 `op.run()`，并在详细模式下打印运行时间与共享内存占用。

### 13. Lines 646-693 | Reference execution and result verification | 参考执行与结果校验

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

    bool passed = true;

    if (options.verify) {
      printf("[TensorY]verifying...\n");
      passed &= compare_reference<5>(mY_ref_1, mY_res);
      printf("[TensorF]verifying...\n");
      passed &= compare_reference<4>(mF_ref_1, mF_res);
    }

    return passed;
```

- **EN:** After the device kernel finishes, the code re-wraps all universal vectors as CuTe tensors in logical layout form, runs the reference SSD implementation when requested, and compares both the output sequence `Y` and the final recurrent state `F`.

- **CN:** 设备 kernel 完成后，代码会把所有 universal vector 重新包装成逻辑布局下的 CuTe 张量，在需要时运行参考 SSD 实现，并同时比较输出序列 `Y` 与最终递归状态 `F`。

### 14. Lines 694-775 | Tolerance-based tensor comparison helper | 带容差的张量比较辅助函数

```cpp
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
            printf("%.4f ", static_cast<float>(reference(0,0,mi,0,ni)));
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
            printf("%.4f ", static_cast<float>(computed(0,0,mi,0,ni)));
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
```

- **EN:** `compare_reference` supports either exact comparison or an epsilon-based check using both absolute and relative error. On the first mismatch it prints the offending element and a small tensor slice, which is practical when debugging state-space kernels.

- **CN:** `compare_reference` 既支持精确比较，也支持结合绝对误差与相对误差的 epsilon 检查。一旦发现首个不匹配，它会打印出错元素及一个小片段张量，这对调试状态空间 kernel 很实用。

### 15. Lines 776-850 | Program entry point and Blackwell runtime gating | 程序入口与 Blackwell 运行时门控

```cpp
};

#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)

int main(int argc, char const **args) {

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (__CUDACC_VER_MAJOR__ < 12 || props.major < 10) {
    std::cout
      << "This example requires a GPU of NVIDIA's Blackwell Architecture or "
      << "later (compute capability 100 or greater) and CUDA 12.0 or greater.\n";
    return 0;
  }
  else if (__CUDACC_VER_MAJOR__ < 12 || (props.major != 10 || props.minor != 0)) {
    std::cout
      << "This example requires a GPU of NVIDIA's Blackwell Architecture "
      << "(compute capability 100) and CUDA 12.0 or greater.\n";
    return 0;
  }

#if defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)

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

#endif // defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)

  return 0;
}
```

- **EN:** `main()` validates the GPU architecture/CUDA version, parses options, fills `KernelHardwareInfo`, runs the `TestBed`, and prints a simple pass/fail message. This is the top-level bridge from the executable to the actual Blackwell SSD orchestration path.

- **CN:** `main()` 会校验 GPU 架构与 CUDA 版本，解析选项，填充 `KernelHardwareInfo`，运行 `TestBed`，并输出简单的通过/失败信息。它是可执行程序与真正 Blackwell SSD 编排路径之间的最上层桥梁。

## Takeaways | 总结

- **EN:** The file is primarily about host orchestration: shape/layout normalization, preprocessing scans, device launch setup, and correctness validation around the real Blackwell SSD kernel.

- **CN:** 这个文件的核心是主机端编排：统一形状与布局、执行预处理扫描、配置设备端启动参数，并围绕真正的 Blackwell SSD kernel 完成正确性校验。

---

## Key Concepts / 关键概念

- **Blackwell host orchestration / Blackwell 主机编排**

  - **EN:** The example separates orchestration from math: the host prepares shapes, layouts, workspace, events, hardware info, and the two kernel launches needed for SSD.

  - **CN:** 这个示例把“编排”和“数学”分开：主机负责准备形状、布局、工作区、计时事件、硬件信息，以及 SSD 需要的两次 kernel 启动。

- **Chunked scan preprocessing / 分块扫描预处理**

  - **EN:** Before the main kernel, `DeltaA` is converted into a cumulative-sum form. That mirrors the chunk boundary scan used by Mamba2/SSD and avoids recomputing prefix sums inside the main kernel.

  - **CN:** `DeltaA` 会在主核之前先变成累计和形式。这对应 Mamba2/SSD 的分块扫描边界信息，也避免在主核里重复做前缀求和。

- **Semiseparable SSD dataflow / 半可分 SSD 数据流**

  - **EN:** The host wires tensors for the diagonal intra-chunk term, chunk-to-chunk state propagation, and the final state/output epilogue that reconstruct semiseparable SSM behavior.

  - **CN:** 主机端把张量按“块内对角项、块间状态传播、最终状态/输出回写”三部分连接起来，从而恢复半可分 SSM 的执行流程。

- **Layout adaptation with CuTe / 用 CuTe 做布局改写**

  - **EN:** The public tensor layouts follow the logical `[b, eh, d, c, l]`-style views, while transformed layouts collapse and permute dimensions into the exact contract expected by the kernel builder.

  - **CN:** 对外的张量布局保持 `[b, eh, d, c, l]` 这类逻辑视图，而变换后的布局会折叠并重排维度，以匹配 kernel builder 所要求的精确参数约定。

- **Blackwell-specific gating / Blackwell 专用门控**

  - **EN:** The example explicitly checks for CUDA 12+ and compute capability 10.0, and the code path itself is guarded by `CUTLASS_ARCH_MMA_SM100_SUPPORTED`.

  - **CN:** 示例显式检查 CUDA 12+ 与 10.0 计算能力，并用 `CUTLASS_ARCH_MMA_SM100_SUPPORTED` 对 Blackwell/SM100 路径做编译期门控。

## Dependencies / 依赖项

- **`reference/reference_ssd_cumsum.hpp`**

  - **EN:** Defines the auxiliary transform kernel that computes cumulative sums of `DeltaA` before the SSD kernel runs.

  - **CN:** 定义辅助变换核，在 SSD 主核启动前对 `DeltaA` 做累计和预处理。

- **`reference/reference_ssd.hpp`**

  - **EN:** Provides the scalar/CuTe reference used for correctness checking of `Y` and final states.

  - **CN:** 提供用于校验 `Y` 和最终状态的标量/CuTe 参考实现。

- **`device/ssd.hpp` + `kernel/sm100_ssd_kernel_builder.hpp`**

  - **EN:** These headers instantiate the real Blackwell SSD device operator and its kernel type.

  - **CN:** 这两个头文件共同实例化真正的 Blackwell SSD 设备算子及其 kernel 类型。

- **CUTLASS/CuTe utilities**

  - **EN:** Command-line parsing, layout construction, hardware discovery, tensor fillers, and transform adapters all come from CUTLASS/CuTe utility layers.

  - **CN:** 命令行解析、布局构造、硬件发现、张量填充和 transform adapter 等能力都来自 CUTLASS/CuTe 工具层。
