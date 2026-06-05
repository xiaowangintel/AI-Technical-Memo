# profiler — Documentation Analysis / 文档分析

## Source / 来源
- Source: `/root/xw/sycl-tla/media/docs/cpp/profiler.md`
- Purpose: Documents the CUTLASS profiler build flow, kernel instantiation controls, command-line usage, GEMM/convolution arguments, and output interpretation. / 说明 CUTLASS profiler 的构建流程、内核实例化控制、命令行用法、GEMM/卷积参数以及结果输出解释。

## Content Analysis / 内容分析
### CUTLASS Profiler
**EN:** The opening establishes the profiler as both a correctness harness and a performance tool for GEMM, sparse GEMM, Conv2d, and Conv3d. It also explains why default builds instantiate only a narrow kernel subset unless CMake options expand coverage.
**CN:** 开头把 profiler 定位成同时承担“正确性验证”和“性能测量”的工具，可覆盖 GEMM、稀疏 GEMM、Conv2d 与 Conv3d。它还解释了默认构建为何只实例化较小的内核子集，以及如何用 CMake 扩大覆盖面。

### Emitting kernels via `emit_kernel_listing.py`
**EN:** This section introduces a selective kernel emission path that is more practical than compiling every generated kernel. The key idea is to pair kernel generation with runtime argument lists so regression and profiling workflows stay targeted.
**CN:** 本节介绍了一条“选择性发射内核”的路径，比起把所有生成内核都编译出来更实用。核心思想是把内核生成和运行时参数列表绑定，从而让回归测试与 profiling 工作流更聚焦。

### Instantiating more kernels with Hopper / Blackwell
**EN:** These sections explain the four-digit `CUTLASS_LIBRARY_INSTANTIATION_LEVEL` scheme. The document is effectively a tuning manual for build-time search space: instruction shapes, MMA multipliers, cluster shapes, and schedule pruning determine how aggressively the profiler expands kernel variants.
**CN:** 这些章节解释了四位数 `CUTLASS_LIBRARY_INSTANTIATION_LEVEL` 机制。它本质上是一份“构建期搜索空间调优手册”：指令形状、MMA 倍率、cluster 形状和调度裁剪共同决定 profiler 会把内核变体扩展到什么程度。

### Instantiating more MMA shapes with Hopper / Mixed input data type kernels for Hopper
**EN:** The Hopper-specific subsections show that modern kernels are not just about more tiles, but also about more datatype combinations and layout variants. Mixed-dtype kernels, runtime encodings, and shuffled layouts are treated as first-class search axes.
**CN:** Hopper 专属小节说明，现代内核扩展不只是更多 tile 形状，还包括更多数据类型组合和布局变体。混合数据类型、运行时编码以及 shuffled layout 都被当成一等搜索维度。

### CUTLASS Profiler usage
**EN:** The long help dump is best read as a taxonomy of controls: execution mode, device selection, initialization, library references, profiling policy, verification policy, and reporting. This section turns the profiler from a black box into a scriptable experiment driver.
**CN:** 大段帮助信息更适合被理解为一套“控制项分类表”：执行模式、设备选择、初始化、库参考实现、性能采样策略、验证策略与报告输出。它把 profiler 从黑盒工具变成了可脚本化的实验驱动器。

### GEMM and GEMM Arguments
**EN:** The GEMM part dives into operation-specific knobs such as tensor descriptors, tile shapes, cluster shapes, PDL, mixed-dtype runtime encodings, and data distributions. It shows that CUTLASS profiler is not only for benchmarking named kernels, but also for sweeping algorithmic parameter spaces.
**CN:** GEMM 部分深入到操作级别参数，包括张量描述、tile 形状、cluster 形状、PDL、混合数据类型运行时编码和数据分布等。这表明 CUTLASS profiler 不只是测固定命名内核，也能系统地扫描算法参数空间。

### Exhaustive search mode and usage examples
**EN:** This section adds automation for top-k ranking and fixed-shape exhaustive search. The important design idea is separating kernel filtering from workload specification, so users can search broadly while still controlling data initialization and problem shapes.
**CN:** 本节引入按 GFLOP/s 排序和固定形状穷举搜索等自动化能力。关键设计思想是将“内核筛选”和“工作负载定义”分离，使用户既能广泛搜索，又能精确控制初始化方式与问题规模。

### Output and CUTLASS 3.0 GEMM procedural names
**EN:** These sections teach readers how to interpret profiler reports and decode verbose kernel names. The naming breakdown is especially valuable because many modern kernel properties—architecture, math class, datatypes, cluster shape, stage policy, layouts, and custom schedules—are encoded directly in the name string.
**CN:** 这些章节教读者如何解读 profiler 输出，以及如何拆解冗长的内核命名。名称解析尤为重要，因为现代内核的许多属性——架构、计算类别、数据类型、cluster 形状、stage 策略、布局和自定义调度——都直接编码在名字里。

### Convolution and Convolution Arguments
**EN:** The convolution half mirrors the GEMM half: it lists problem-shape parameters, iterator modes, math classes, and example commands for CUDA-core and Tensor-Core paths. This symmetry makes the profiler feel like a unified frontend over multiple operator families.
**CN:** 卷积部分与 GEMM 部分形成镜像：列出问题尺寸参数、iterator 模式、计算类别，以及 CUDA Core / Tensor Core 路径的示例命令。这种对称性说明 profiler 实际上是多个算子家族共享的一套统一前端。

## Key Concepts / 关键概念
- Build-time kernel instantiation control / 构建期内核实例化控制
- Runtime profiling and verification / 运行时性能测量与正确性验证
- Search-space pruning vs exhaustive tuning / 搜索空间裁剪与穷举调优
- Procedural kernel naming / 过程化内核命名
- Unified CLI for GEMM and convolution / 统一的 GEMM 与卷积命令行接口

## Related Files / 相关文件
- `tools/profiler/src/main.cpp` — profiler entry point / profiler 入口
- `tools/profiler/src/gemm_operation_profiler.cu` — GEMM profiling path / GEMM profiling 实现
- `tools/profiler/src/conv2d_operation_profiler.cu` — Conv2d profiling path / Conv2d profiling 实现
- `python/cutlass_library/emit_kernel_listing.py` — selective kernel list generation / 选择性生成内核列表
- `media/docs/cpp/dependent_kernel_launch.md` — referenced PDL background / 文中引用的 PDL 背景文档
