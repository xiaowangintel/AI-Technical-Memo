# quickstart.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** /root/xw/sycl-tla/media/docs/cpp/quickstart.md
- **Purpose:** Provides the main practical guide for configuring, building, testing, and using CUTLASS, including example GEMM launches and library integration. / 提供 CUTLASS 的主要实践指南，覆盖配置、构建、测试、使用，以及 GEMM 示例启动和库集成。

## Content Analysis / 内容分析
### Prerequisites
**EN:** Defines the baseline environment: CUDA Toolkit, CMake, a C++17-capable host compiler, and Python. It also notes optional cuBLAS and cuDNN dependencies, making the reader aware of both hard and soft requirements.
**CN:** 定义基础环境要求：CUDA Toolkit、CMake、支持 C++17 的宿主编译器以及 Python。同时指出 cuBLAS 与 cuDNN 是可选依赖，让读者区分硬性条件与增强能力。

### Initial build steps
**EN:** Starts from an empty build directory and shows architecture-specific CMake commands. The section is practical and optimization-aware: it suggests disabling tests, enabling unity build, or filtering library operations and kernels to reduce compile time.
**CN:** 从空的 build 目录开始，给出面向特定架构的 CMake 命令。本节非常务实，也体现编译优化意识：建议关闭测试、启用 unity build，或通过过滤 operations 和 kernels 来缩短编译时间。

### Build and run the CUTLASS Profiler
**EN:** Shows how to build the profiler and use it to benchmark GEMM and convolution kernels. The sample outputs teach the reader what success, runtime, bandwidth, and math-throughput reports look like in practice.
**CN:** 说明如何构建 profiler，并用它评测 GEMM 和卷积内核。示例输出让读者直观看到成功状态、运行时间、带宽和算力指标在实际中如何呈现。

### Build and run CUTLASS Unit Tests
**EN:** Explains the default test path with make test_unit, emphasizes that no tests should fail, and then shows how the hierarchical test layout enables narrower targets such as warp-level GEMM tests.
**CN:** 解释默认测试路径 make test_unit，强调测试原则上不应失败，并进一步说明分层测试结构如何支持更细粒度的目标，例如仅运行 warp 级 GEMM 测试。

### Building for Multiple Architectures
**EN:** Maps CUTLASS_NVCC_ARCHS values to NVIDIA architecture families from Maxwell through Blackwell. This section is short but operationally important because architecture scoping strongly affects compile time and kernel availability.
**CN:** 将 CUTLASS_NVCC_ARCHS 的取值映射到从 Maxwell 到 Blackwell 的 NVIDIA 架构家族。虽然篇幅不长，但操作上很关键，因为架构范围会直接影响编译时间和可用内核。

### Using CUTLASS within other applications
**EN:** Shifts from building CUTLASS itself to consuming it as a dependency. The example focuses on include paths and C++17 mode, using a tiny host-side numeric example to keep integration approachable.
**CN:** 从“构建 CUTLASS 本身”转向“把 CUTLASS 作为依赖使用”。示例重点放在 include 路径与 C++17 编译模式上，并用一个极小的宿主侧数值例子降低集成门槛。

### Launching a GEMM kernel in CUDA
**EN:** Presents the classic device::Gemm path. The code walks through type selection, problem-size definition, host tensor allocation, leading dimensions, and the final kernel launch call, making it a concrete template-instantiation example.
**CN:** 给出经典的 device::Gemm 使用路径。代码依次展示类型选择、问题规模定义、HostTensor 分配、leading dimension 设置以及最终 kernel 启动调用，是一个非常具体的模板实例化示例。

### Launching a GEMM kernel using CUTLASS 3.0 or newer
**EN:** Introduces the newer 3.x collective-builder style for Hopper-class kernels. The section is more advanced: it decomposes a GEMM into mainloop, epilogue, kernel, adapter, problem shape, strides, and launch arguments.
**CN:** 介绍面向 CUTLASS 3.x 的新式 collective-builder 写法，用于 Hopper 级别内核。本节更进阶：它把 GEMM 分解为 mainloop、epilogue、kernel、adapter、problem shape、stride 和启动参数。

### CUTLASS Library
**EN:** Describes the runtime library layer that manages compiled kernel instances without exposing templates in client code. It compares the host API to BLAS-style launches and explains how SDK examples link against cutlass_lib.
**CN:** 描述运行时库层：它管理已编译的内核实例，而无需在客户端代码中直接暴露模板。文档把其宿主侧 API 类比为 BLAS 风格调用，并说明 SDK 示例如何链接 cutlass_lib。

### Example CMake Commands, GEMM CMake Examples, and Convolution CMake Examples
**EN:** These sections are a cookbook for controlling build breadth. They explain all-kernel builds, unity builds, and fine-grained kernel filters for GEMM and convolution so users can trade compilation cost against coverage.
**CN:** 这些部分像一本“CMake 菜谱”，用于控制构建范围。它们说明如何构建全部内核、如何启用 unity build，以及如何对 GEMM 和卷积内核做细粒度过滤，从而在编译成本与覆盖范围之间权衡。

### Instantiating a Blackwell SM100 GEMM kernel
**EN:** Focuses on a very specific advanced topic: hand-instantiating an SM100 GEMM kernel, including FP8 and blockscaled variants. The analysis value here is that it shows how Blackwell support reuses familiar epilogue patterns while introducing dispatch-policy details and schedule choices.
**CN:** 聚焦一个非常具体的高级主题：手动实例化 Blackwell SM100 GEMM 内核，包括 FP8 与 blockscaled 变体。它的价值在于展示了 Blackwell 支持如何复用熟悉的 epilogue 模式，同时引入 dispatch policy 与调度选择等新细节。

### Copyright
**EN:** Ends with standard BSD-3-Clause license text.
**CN:** 结尾是标准 BSD-3-Clause 许可证文本。

## Key Concepts / 关键概念
- Architecture-targeted CMake configuration / 面向架构的 CMake 配置
- Profiler-driven validation and benchmarking / 基于 profiler 的验证与性能评测
- Hierarchical unit-test organization / 分层单元测试组织方式
- Template API vs runtime library API / 模板 API 与运行时库 API
- Selective kernel instantiation / 选择性内核实例化
- CUTLASS 3.x collective builders / CUTLASS 3.x collective builder 体系

## Related Files / 相关文件
- /root/xw/sycl-tla/media/docs/cpp/getting_started.rst — Top-level onboarding page that links here / 链接到本页的顶层入门页
- /root/xw/sycl-tla/media/docs/cpp/build/index.rst — Build subsection for platform-specific setup / 提供平台化构建说明的子章节
- /root/xw/sycl-tla/media/docs/cpp/code_organization.md — Repository structure context for the components mentioned here / 为本文涉及组件提供仓库结构背景
- /root/xw/sycl-tla/media/docs/cpp/doxygen_mainpage.md — High-level project overview paired with this practical guide / 与本实践指南互补的项目总览
