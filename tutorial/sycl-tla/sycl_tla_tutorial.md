# SYCL-TLA Tutorial for CUTLASS Users / 面向 CUTLASS 用户的 SYCL-TLA 教程

**EN:** This tutorial is written for engineers who already understand CUTLASS and CuTe, and want to learn what SYCL-TLA adds for Intel GPUs.
**CN:** 本教程面向已经理解 CUTLASS 和 CuTe、但希望学习 SYCL-TLA 在 Intel GPU 上新增内容的工程师。

**EN:** The emphasis is on the Intel Xe extension layer: what changes, why it changes, and how to read or write kernels that use those changes.
**CN:** 重点放在 Intel Xe 扩展层：哪些地方发生了变化、为什么变化，以及如何阅读或编写使用这些变化的内核。

**EN:** The tutorial intentionally does not re-teach CUTLASS basics such as layout algebra, TiledMMA fundamentals, or the general CollectiveBuilder story in detail.
**CN:** 本教程有意不再详细重讲 CUTLASS 基础，例如 layout algebra、TiledMMA 基本概念，以及通用的 CollectiveBuilder 框架。

**EN:** Instead, every section asks a CUTLASS-centric question: what is identical to upstream, what is Intel-specific, and where is the extension point in SYCL-TLA.
**CN:** 相反，每一节都围绕 CUTLASS 用户最关心的问题展开：哪些内容与上游一致，哪些是 Intel 特有扩展，SYCL-TLA 的扩展点在哪里。

**EN:** Primary repository references used here include `../media/docs/cpp/xe_rearchitecture.md`, `../media/docs/cpp/cute/10_intel_overview.md`, `../media/docs/cpp/cute/11_intel_gemm_companion.md`, `../media/docs/cpp/cute/12_intel_performance_guide.md`, `../media/docs/cpp/cute/xe_2d_copy.md`, `../media/docs/cpp/slm_pipeline_construction.md`, `../media/docs/cpp/build/building_with_sycl_support.md`, and `../media/docs/python/xe_cutlass_library.md`.
**CN:** 本文主要参考仓库中的 `../media/docs/cpp/xe_rearchitecture.md`、`../media/docs/cpp/cute/10_intel_overview.md`、`../media/docs/cpp/cute/11_intel_gemm_companion.md`、`../media/docs/cpp/cute/12_intel_performance_guide.md`、`../media/docs/cpp/cute/xe_2d_copy.md`、`../media/docs/cpp/slm_pipeline_construction.md`、`../media/docs/cpp/build/building_with_sycl_support.md` 和 `../media/docs/python/xe_cutlass_library.md`。

**EN:** Representative examples used throughout come from `../examples/00_bmg_gemm`, `../examples/01_bmg_gemm_with_collective_builder`, `../examples/02_bmg_gemm_mixed_dtype`, `../examples/05_bmg_gemm_with_epilogues`, `../examples/06_bmg_flash_attention`, `../examples/08_bmg_gemm_f8`, `../examples/cute/tutorial/xe_gemm.cpp`, `../examples/cute/tutorial/xe_gemm_slm.cpp`, and `../examples/python/cutlass_library/xe20_gemm_bf16.py`.
**CN:** 全文会反复引用 `../examples/00_bmg_gemm`、`../examples/01_bmg_gemm_with_collective_builder`、`../examples/02_bmg_gemm_mixed_dtype`、`../examples/05_bmg_gemm_with_epilogues`、`../examples/06_bmg_flash_attention`、`../examples/08_bmg_gemm_f8`、`../examples/cute/tutorial/xe_gemm.cpp`、`../examples/cute/tutorial/xe_gemm_slm.cpp` 和 `../examples/python/cutlass_library/xe20_gemm_bf16.py`。

## How to Read This Tutorial / 如何阅读本教程

**EN:** Read Sections 1 to 4 if you want the mental model first: lineage, hardware, SYCL mapping, and CuTe extension points.
**CN:** 如果你想先建立整体心智模型，请先阅读第 1 到第 4 节：版本脉络、硬件、SYCL 映射，以及 CuTe 扩展点。

**EN:** Read Sections 5 to 9 if your main job is kernel work: GEMM, epilogue/EVT, mixed precision, Flash Attention, and SLM pipelines.
**CN:** 如果你的主要工作是内核开发，请重点阅读第 5 到第 9 节：GEMM、epilogue/EVT、混合精度、Flash Attention 与 SLM pipeline。

**EN:** Read Sections 10 to 12 if you care about integration: Python generation, build system setup, and performance tuning.
**CN:** 如果你更关注集成层，请重点阅读第 10 到第 12 节：Python 生成、构建系统配置与性能调优。

**EN:** Read the appendices when porting existing CUTLASS code, because the glossary, example map, and pitfall list are designed as a desk reference.
**CN:** 当你在迁移已有 CUTLASS 代码时，请阅读附录，因为术语表、示例地图和陷阱清单都被设计成可随手查阅的案头参考。

**EN:** A good workflow is: first copy a working example, then replace only the policy layer you need, and finally tune tile/stage/scheduler choices with measurement.
**CN:** 推荐工作流是：先复制一个可运行示例，再只替换你需要的 policy 层，最后通过测量来调优 tile、stage 和 scheduler 的选择。

**EN:** When this tutorial says “same as CUTLASS”, it means the conceptual interface is shared; it does not mean the hardware realization is the same as CUDA.
**CN:** 当本文说“与 CUTLASS 相同”时，指的是概念接口相同；并不意味着底层硬件实现与 CUDA 完全一致。

**EN:** When this tutorial says “Intel-specific”, it usually points to one of four mechanisms: subgroup width 16, DPAS/XMX, 2D block copy, or Xe-specific scheduling/epilogues.
**CN:** 当本文说“Intel 特有”时，通常指向以下四类机制之一：16 宽 subgroup、DPAS/XMX、2D block copy，或 Xe 特有的调度/epilogue。

## Table of Contents / 目录

**EN:** 1. Introduction and lineage.
**CN:** 1. 介绍与版本脉络。

**EN:** 2. CUTLASS-to-SYCL-TLA delta map.
**CN:** 2. CUTLASS 到 SYCL-TLA 的差异地图。

**EN:** 3. Intel Xe GPU architecture for kernel writers.
**CN:** 3. 面向内核开发者的 Intel Xe GPU 架构。

**EN:** 4. SYCL programming model and CUDA mapping.
**CN:** 4. SYCL 编程模型与 CUDA 映射。

**EN:** 5. CuTe extensions for Intel Xe.
**CN:** 5. 面向 Intel Xe 的 CuTe 扩展。

**EN:** 6. GEMM structure on Intel GPUs.
**CN:** 6. Intel GPU 上的 GEMM 结构。

**EN:** 7. Xe epilogues, EVT, and fusion callbacks.
**CN:** 7. Xe epilogue、EVT 与 fusion callback。

**EN:** 8. Mixed precision and quantization flows.
**CN:** 8. 混合精度与量化流程。

**EN:** 9. Flash Attention on BMG.
**CN:** 9. BMG 上的 Flash Attention。

**EN:** 10. SLM pipelines and subgroup specialization.
**CN:** 10. SLM pipeline 与 subgroup specialization。

**EN:** 11. Python interface and kernel generation.
**CN:** 11. Python 接口与内核生成。

**EN:** 12. Build system and performance guide.
**CN:** 12. 构建系统与性能指南。

**EN:** 13. Porting checklist, example map, pitfalls, and glossary.
**CN:** 13. 迁移清单、示例地图、常见陷阱与术语表。

## 1. Introduction / 介绍

**EN:** SYCL-TLA stands for SYCL Templates for Linear Algebra. It is a fork of NVIDIA CUTLASS that extends CUTLASS and CuTe APIs to SYCL and Intel GPUs.
**CN:** SYCL-TLA 的全称是 SYCL Templates for Linear Algebra。它是 NVIDIA CUTLASS 的一个分支，将 CUTLASS 和 CuTe API 扩展到 SYCL 与 Intel GPU。

**EN:** Historically, the project was previously referred to as CUTLASS-SYCL, so older files, comments, or examples may still use that older name.
**CN:** 从历史上看，这个项目之前也被称为 CUTLASS-SYCL，因此旧文件、注释或示例中仍可能出现该名称。

**EN:** The important idea is not “CUTLASS rewritten in SYCL”, but “CUTLASS abstractions preserved while the hardware-specific backend is retargeted to Intel Xe”.
**CN:** 关键点不是“用 SYCL 重写 CUTLASS”，而是“保留 CUTLASS 抽象，同时把硬件相关后端重新定向到 Intel Xe”。

**EN:** SYCL-TLA fast-follows NVIDIA CUTLASS releases. The repository README maps SYCL-TLA releases 0.1 to 0.9 to CUTLASS releases from 3.9 through 4.2.1.
**CN:** SYCL-TLA 会快速跟进 NVIDIA CUTLASS 的版本。仓库 README 给出了从 SYCL-TLA 0.1 到 0.9 与 CUTLASS 3.9 到 4.2.1 的对应关系。

**EN:** For practical engineering, this means you should expect the CUTLASS 3.x/4.x decomposition style to remain recognizable: CuTe atoms, collectives, builders, GEMM kernels, EVT epilogues, and a manifest-driven Python path.
**CN:** 对工程实践而言，这意味着你仍然会看到熟悉的 CUTLASS 3.x/4.x 分解方式：CuTe atom、collective、builder、GEMM kernel、EVT epilogue，以及 manifest 驱动的 Python 路径。

**EN:** What changes is the Intel execution model and the policy layer: subgroup width, DPAS atom families, block-2D copy atoms, reorder operations, kernel schedules, epilogues, and tuning rules.
**CN:** 发生变化的是 Intel 执行模型与 policy 层：subgroup 宽度、DPAS atom 家族、block-2D copy atom、reorder 操作、kernel schedule、epilogue 与调优规则。

**EN:** The two main Intel target families in current documentation are PVC and BMG.
**CN:** 当前文档中的两个主要 Intel 目标平台是 PVC 与 BMG。

**EN:** PVC means Ponte Vecchio, marketed as Intel Data Center GPU Max Series, and corresponds to Xe-HPC-class hardware.
**CN:** PVC 指 Ponte Vecchio，也就是 Intel Data Center GPU Max 系列，对应 Xe-HPC 级别硬件。

**EN:** BMG means BattleMage, including Arc B580-class devices, and corresponds to Xe2-class hardware in the current README and examples.
**CN:** BMG 指 BattleMage，包括 Arc B580 这一类设备，在当前 README 和示例中对应 Xe2 级别硬件。

**EN:** In type-level dispatch you will see `cutlass::arch::Xe12` and `cutlass::arch::Xe20`; in the Intel overview these are forwarded to the common `arch::IntelXe` builder logic.
**CN:** 在类型级调度里你会看到 `cutlass::arch::Xe12` 与 `cutlass::arch::Xe20`；在 Intel 概览中，这些架构标签最终会被转发到共同的 `arch::IntelXe` builder 逻辑。

**EN:** SYCL-TLA remains header-only as a library. You build examples, tests, generated libraries, and benchmark applications, but most of the extension logic lives in headers under `../include`.
**CN:** SYCL-TLA 作为库仍然保持 header-only 形式。你会编译示例、测试、生成库和 benchmark 应用，但大多数扩展逻辑都位于 `../include` 下的头文件中。

**EN:** If you already know CUTLASS, the shortest summary is: SYCL-TLA keeps the decomposition, but swaps CUDA warp/Tensor Core/TMA assumptions for SYCL subgroup/XMX-DPAS/Xe block-2D assumptions.
**CN:** 如果你已经了解 CUTLASS，最简短的总结是：SYCL-TLA 保留了分解方式，但把 CUDA 的 warp/Tensor Core/TMA 假设替换成了 SYCL subgroup/XMX-DPAS/Xe block-2D 假设。

**EN:** The most important documents for this shift are `xe_rearchitecture.md` and `10_intel_overview.md`, because they explain why old Xe atoms were redesigned and what new APIs you should prefer.
**CN:** 理解这种变化最重要的文档是 `xe_rearchitecture.md` 与 `10_intel_overview.md`，因为它们解释了为什么旧 Xe atom 要重构，以及现在应该优先使用哪些新 API。

**EN:** A recurring theme is that SYCL-TLA tries to expose more real hardware structure than the legacy Intel path did, especially for VNNI layouts, block-2D operations, and subgroup-owned register tiles.
**CN:** 一个反复出现的主题是：相比旧版 Intel 路径，SYCL-TLA 更努力地暴露真实硬件结构，尤其是 VNNI 布局、block-2D 操作以及 subgroup 共同拥有的寄存器 tile。

**EN:** That is why the tutorial focuses on extension points rather than on generic CuTe syntax.
**CN:** 这也是为什么本教程会把重点放在扩展点，而不是通用 CuTe 语法本身。

## 2. CUTLASS to SYCL-TLA Delta Map / CUTLASS 到 SYCL-TLA 的差异地图

**EN:** The fastest way to orient yourself is to separate concepts that are preserved from mechanisms that are replaced.
**CN:** 最快的定位方式，是把“被保留的概念”与“被替换的机制”分开看。

**EN:** Preserved concepts include CuTe layouts/tensors, TiledMMA, TiledCopy, Collectives, GEMM kernels, EVT-style epilogues, and builder-driven kernel selection.
**CN:** 被保留的概念包括 CuTe 的 layout/tensor、TiledMMA、TiledCopy、Collective、GEMM kernel、EVT 风格 epilogue，以及 builder 驱动的 kernel 选择。

**EN:** Replaced mechanisms include warp-sized reasoning, Tensor Core atom names, cp.async/TMA memory movement, CUDA launch syntax, and CUDA-specific scheduling heuristics.
**CN:** 被替换的机制包括 warp 粒度思维、Tensor Core atom 命名、cp.async/TMA 数据搬运、CUDA 启动语法，以及 CUDA 特定的调度启发式。

**EN:** In CUDA CUTLASS you often think in 32-thread warps. In SYCL-TLA you must think in 16-lane subgroups.
**CN:** 在 CUDA CUTLASS 中你常常以 32 线程 warp 思考；在 SYCL-TLA 中你必须改成 16 lane 的 subgroup 心智模型。

**EN:** In CUDA CUTLASS you often think of per-thread fragments. In SYCL-TLA the deeper truth is subgroup-owned fragments represented by `SubgroupTensor`.
**CN:** 在 CUDA CUTLASS 中你常常以“每线程 fragment”思考；而在 SYCL-TLA 中，更底层的真实语义是由 `SubgroupTensor` 表示的“subgroup 共同拥有 fragment”。

**EN:** In CUDA CUTLASS you typically move tiles with cp.async/TMA plus shared memory staging. In SYCL-TLA the default fast path is often global-memory/L1 to GRF via block-2D loads, with SLM used only when it earns its keep.
**CN:** 在 CUDA CUTLASS 中，你通常会用 cp.async/TMA 加共享内存分级搬运 tile；而在 SYCL-TLA 中，默认快路径通常是通过 block-2D load 直接从 global memory/L1 搬到 GRF，只有在确实值得时才引入 SLM。

**EN:** In NVIDIA code, B-operand layout concerns are often hidden behind Tensor Core loaders. In Intel Xe code, VNNI layout is a first-class issue, because DPAS and VNNI loads want data in a very specific packing.
**CN:** 在 NVIDIA 代码中，B 操作数的布局问题常常被 Tensor Core loader 隐藏；而在 Intel Xe 代码中，VNNI 布局是一级问题，因为 DPAS 与 VNNI load 都要求数据采用非常特定的打包方式。

**EN:** That is why SYCL-TLA introduces explicit reorder operations and new copy helper APIs instead of only renaming CUDA concepts.
**CN:** 这也是为什么 SYCL-TLA 不只是改名，而是引入了显式 reorder 操作和新的 copy helper API。

**EN:** Another practical difference is that Intel builders default to cluster shape `<1,1,1>` for Xe GEMM examples today.
**CN:** 另一个实际差异是，当前 Xe GEMM 示例中的 Intel builder 默认使用 cluster shape `<1,1,1>`。

**EN:** The Intel collective builders also choose subgroup tilings automatically, cap subgroup count around 32, and default to L1-staged pipelines rather than mandatory SLM pipelines.
**CN:** Intel collective builder 还会自动选择 subgroup tiling，把 subgroup 数量限制在大约 32 左右，并默认采用 L1-staged pipeline，而不是强制使用 SLM pipeline。

**EN:** At the epilogue level, Intel Xe code keeps the EVT abstraction but changes the realization: 2D load/store atoms, Xe callback specializations, and Xe visitor-tree components such as `XeAuxStore`.
**CN:** 在 epilogue 层，Intel Xe 代码保留 EVT 抽象，但具体实现发生了变化：使用 2D load/store atom、Xe callback 特化，以及 `XeAuxStore` 之类的 Xe visitor-tree 组件。

**EN:** At the Python level, manifest generation still exists, but the backend can emit Xe-specific EVT types and Xe architecture IDs such as `xe20`.
**CN:** 在 Python 层，manifest 生成仍然存在，但后端可以发射 Xe 特有的 EVT 类型以及 `xe20` 这样的 Xe 架构 ID。

**EN:** A productive mindset is therefore: preserve your algorithm decomposition, then replace the hardware contract.
**CN:** 因此，一个高效的心智模型是：保留你的算法分解方式，然后替换底层硬件契约。

**EN:** That contract on Xe is expressed mainly through `XE_DPAS_TT`, `XE_LOAD_2D*`, `XE_STORE_2D`, `XE_PREFETCH_2D`, `reorder`, `sub_group_size<16>`, and `grf_size<256>`.
**CN:** 在 Xe 上，这个契约主要由 `XE_DPAS_TT`、`XE_LOAD_2D*`、`XE_STORE_2D`、`XE_PREFETCH_2D`、`reorder`、`sub_group_size<16>` 和 `grf_size<256>` 来表达。

**EN:** If you remember only one rule, remember this one: SYCL-TLA is CUTLASS decomposition plus Intel Xe execution semantics.
**CN:** 如果你只记住一条规则，那就是：SYCL-TLA 等于 CUTLASS 分解方式加上 Intel Xe 执行语义。

## 3. Intel Xe GPU Architecture / Intel Xe GPU 架构

**EN:** For SYCL-TLA users, a useful execution hierarchy is: GPU -> Xe-core -> EU thread/subgroup -> XMX DPAS.
**CN:** 对 SYCL-TLA 用户来说，一个有用的执行层级模型是：GPU -> Xe-core -> EU thread/subgroup -> XMX DPAS。

**EN:** The exact microarchitecture differs between PVC and BMG, but the software contract that matters to SYCL-TLA is stable enough to program against at the template level.
**CN:** PVC 与 BMG 的精确微架构细节并不完全相同，但对 SYCL-TLA 来说，模板层真正关心的软件契约已经足够稳定。

**EN:** The `11_intel_gemm_companion.md` document deliberately maps Xe-core to the role an SM plays in CUDA mental models.
**CN:** `11_intel_gemm_companion.md` 明确把 Xe-core 映射到 CUDA 心智模型里类似 SM 的角色。

**EN:** The document also maps the EU execution granularity that runs lockstep code to a SYCL `sub_group`, and it is always 16 lanes for the Xe path discussed here.
**CN:** 该文档还把执行锁步代码的 EU 粒度映射成 SYCL `sub_group`，而在这里讨论的 Xe 路径中它固定为 16 lane。

**EN:** This fixed width is not an implementation accident. Many template choices, fragment layouts, and reorder rules assume exactly 16.
**CN:** 这种固定宽度并不是实现偶然。很多模板选择、fragment 布局和 reorder 规则都精确依赖 16。

**EN:** The matrix engine is XMX, and the instruction model exposed to CuTe is DPAS.
**CN:** 矩阵引擎是 XMX，而暴露给 CuTe 的指令模型是 DPAS。

**EN:** DPAS is the Xe analogue of a Tensor Core MMA operation, but the shape contract is different: N is fixed at 16, M ranges from 1 to 8, and K depends on operand bit width.
**CN:** DPAS 可以视为 Xe 版本的 Tensor Core MMA，但形状契约不同：N 固定为 16，M 在 1 到 8 之间，而 K 则取决于操作数位宽。

**EN:** For BF16 or FP16, a common DPAS atom is effectively 8x16x16. For TF32 the K dimension is smaller. For INT8 and INT4 the K dimension becomes larger.
**CN:** 对 BF16 或 FP16 而言，常见 DPAS atom 的有效形状是 8x16x16。对 TF32 来说 K 更小；对 INT8 和 INT4 来说 K 会更大。

**EN:** One reason the Xe rearchitecture exists is to expose the real B-matrix VNNI layout required by DPAS instead of hiding it behind fragile legacy fragments.
**CN:** Xe 重构存在的一个重要原因，是把 DPAS 所需要的真实 B 矩阵 VNNI 布局显式暴露出来，而不是继续隐藏在脆弱的旧 fragment 表达中。

**EN:** Memory hierarchy should be read in two layers: hardware-visible caches and software-visible movement points.
**CN:** 理解内存层级时，最好分成两层：硬件可见 cache 与软件可见的数据移动节点。

**EN:** At a high level you have off-chip global memory, then shared cache resources such as L3, then per-core cache such as L1, then GRF, and optionally SLM as a programmer-managed work-group scratchpad.
**CN:** 在高层次上，你有片外 global memory，然后是类似 L3 的共享缓存资源，再往下是每 core 的 L1，然后是 GRF，另外还有可选的、由程序员管理的 work-group scratchpad，也就是 SLM。

**EN:** In practice most Xe GEMM kernels in SYCL-TLA reason mostly about Global -> L1 -> GRF, and only introduce SLM when sharing, preprocessing, or alignment constraints demand it.
**CN:** 在实践中，SYCL-TLA 中的大多数 Xe GEMM kernel 主要围绕 Global -> L1 -> GRF 思考，只有在数据共享、预处理或对齐限制要求时才引入 SLM。

**EN:** The performance guide explicitly says that standard BF16/FP16 GEMM on BMG often bypasses SLM and streams directly through block-2D loads into registers.
**CN:** 性能指南明确指出，在 BMG 上的标准 BF16/FP16 GEMM 通常会绕过 SLM，而是通过 block-2D load 直接把数据流入寄存器。

**EN:** This is a major difference from many CUDA tutorials, where shared memory is always foregrounded.
**CN:** 这与很多 CUDA 教程形成鲜明对比，因为在那些教程中，共享内存几乎总是主角。

**EN:** Subgroups are the most important execution unit for Intel Xe kernel writers.
**CN:** 对于 Intel Xe 内核开发者来说，subgroup 是最重要的执行单位。

**EN:** The Xe rearchitecture document explains subgroup-owned private arrays with a round-robin ownership rule: lane `i` owns elements `i`, `i+16`, `i+32`, and so on.
**CN:** Xe 重构文档用一个轮转规则解释 subgroup 共同拥有的私有数组：lane `i` 拥有第 `i`、`i+16`、`i+32` 等元素。

**EN:** That rule is why reinterpretation between 32-bit and 16-bit views can change ownership and therefore act like a shuffle, not just a bitcast.
**CN:** 正因为如此，32 位与 16 位视图之间的重解释会改变元素归属，因此它更像一次 shuffle，而不只是 bitcast。

**EN:** For sub-byte types the ownership story is even more subtle because CUTLASS packs elements into bytes first.
**CN:** 对于 sub-byte 类型，归属关系会更微妙，因为 CUTLASS 会先把元素打包进字节。

**EN:** This detail matters when you try to combine INT4/INT8 dequantization, VNNI packing, and reorder operations inside a single subgroup pipeline.
**CN:** 当你试图在同一个 subgroup pipeline 中组合 INT4/INT8 反量化、VNNI 打包与 reorder 时，这一点尤其关键。

**EN:** The safest design habit is to trust the helper APIs and the fragment partitioning functions, and only reason manually about ownership when debugging a layout mismatch.
**CN:** 最安全的设计习惯是尽量依赖 helper API 和 fragment partitioning 函数，只有在调试布局不匹配时才手动推演 ownership。

**EN:** From a performance perspective, the architecture lesson is simple: maximize useful DPAS work per block-2D issue, avoid misaligned messages, and watch GRF pressure relentlessly.
**CN:** 从性能角度看，架构层面的结论很简单：让每次 block-2D issue 尽量喂出更多有效 DPAS 计算，避免不对齐消息，并持续关注 GRF 压力。

**EN:** That is why tile shape, pipeline depth, and copy atom selection dominate Xe tuning.
**CN:** 这也解释了为什么 tile shape、pipeline depth 和 copy atom 选择会主导 Xe 调优。

**EN:** For reference, current documentation and README focus on PVC and BMG because those are the Intel GPU families with validated SYCL-TLA support and examples.
**CN:** 作为参考，当前文档与 README 聚焦 PVC 和 BMG，因为它们是已有 SYCL-TLA 验证支持与示例的 Intel GPU 家族。

### 3.1 Execution hierarchy cheat sheet / 执行层级速查

```text
GPU
  -> Xe-core
      -> EU execution context / SYCL sub_group (16 lanes)
          -> XMX systolic array executing DPAS
```

**EN:** The point of this diagram is not microarchitectural precision. Its purpose is to keep your template and scheduling choices honest.
**CN:** 这张图的目的不是追求微架构级精确，而是帮助你在做模板和调度选择时保持心智模型正确。

**EN:** When a SYCL-TLA example sets `sub_group_size<16>`, it is encoding part of the hardware contract, not a random optimization hint.
**CN:** 当某个 SYCL-TLA 示例设置 `sub_group_size<16>` 时，它编码的是硬件契约的一部分，而不是随意的优化提示。

**EN:** When a GEMM builder limits subgroup count near 32, it is trying to match a useful occupancy and tile-distribution regime on Xe.
**CN:** 当 GEMM builder 把 subgroup 数量限制在大约 32 左右时，它是在匹配 Xe 上更合理的 occupancy 与 tile 分布区间。

## 4. SYCL Programming Model / SYCL 编程模型

**EN:** At the host side, the most visible difference from CUDA is that kernels are launched through `sycl::queue` instead of a CUDA stream plus triple-chevron syntax.
**CN:** 在主机端，和 CUDA 最显眼的区别是：内核通过 `sycl::queue` 启动，而不是使用 CUDA stream 加三尖括号语法。

**EN:** `sycl::queue` packages device selection, submission ordering, and synchronization semantics.
**CN:** `sycl::queue` 封装了设备选择、提交顺序以及同步语义。

**EN:** SYCL-TLA examples usually rely on a default queue helper or accept an existing queue from the application, which is the natural integration point for larger runtimes.
**CN:** SYCL-TLA 示例通常依赖默认 queue helper，或者接收应用层已有 queue；对于大型运行时系统，这是最自然的集成点。

**EN:** At the kernel-launch level, the equivalent of CUDA grid/block configuration is `sycl::nd_range`.
**CN:** 在 kernel 启动层面，与 CUDA 的 grid/block 配置相对应的是 `sycl::nd_range`。

**EN:** Work-group shape replaces CTA shape, and local range times grid shape replaces CUDA grid dimensions.
**CN:** work-group 形状取代 CTA 形状，而 local range 乘以逻辑 tile 网格则对应 CUDA 的 grid 维度。

**EN:** A `sub_group` in SYCL is the closest semantic match to a warp, but on the Xe path it is 16-wide, not 32-wide.
**CN:** SYCL 中的 `sub_group` 在语义上最接近 warp，但在 Xe 路径上它是 16 宽，而不是 32 宽。

**EN:** That difference shows up everywhere: fragment sizes, block-2D data distribution, and how many work-items participate in one DPAS or one subgroup-scope copy.
**CN:** 这种差异会体现在所有地方：fragment 大小、block-2D 数据分发，以及一次 DPAS 或 subgroup-scope copy 有多少 work-item 参与。

**EN:** SYCL-TLA examples often set Intel-specific kernel properties in addition to `nd_range`, especially subgroup size and GRF size.
**CN:** SYCL-TLA 示例通常会在 `nd_range` 之外设置 Intel 特定 kernel property，尤其是 subgroup size 和 GRF size。

**EN:** The idiom is `sycl::ext::oneapi::experimental::sub_group_size<16>` plus `sycl::ext::intel::experimental::grf_size<256>`.
**CN:** 常见写法是 `sycl::ext::oneapi::experimental::sub_group_size<16>` 加 `sycl::ext::intel::experimental::grf_size<256>`。

**EN:** These properties are not decoration. The performance guide and examples assume them when describing large-tile Xe GEMM behavior.
**CN:** 这些 property 不是装饰项。性能指南与示例在讨论大 tile Xe GEMM 行为时，默认都假设它们已经设置。

**EN:** Within device code, SYCL-TLA examples often fetch the current `nd_item` via `sycl::ext::oneapi::this_work_item::get_nd_item<2>()`.
**CN:** 在 device 代码中，SYCL-TLA 示例常通过 `sycl::ext::oneapi::this_work_item::get_nd_item<2>()` 获取当前 `nd_item`。

**EN:** That gives access to work-group IDs and local IDs, which are then mapped to CuTe slices using `get_slice(local_id)`.
**CN:** 这样可以得到 work-group ID 和 local ID，再通过 `get_slice(local_id)` 把它们映射到 CuTe slice。

**EN:** The CUTLASS-centric way to think about this is: SYCL gives the launch geometry; CuTe maps that geometry onto copy and MMA ownership.
**CN:** 更适合 CUTLASS 用户的理解方式是：SYCL 提供启动几何，而 CuTe 把这种几何映射到 copy 与 MMA 的 ownership。

**EN:** SYCL also replaces CUDA barriers with SYCL or SPIR-V flavored synchronization primitives in the underlying helper layers.
**CN:** SYCL 也会通过 SYCL 或 SPIR-V 风格的同步原语，来替换 CUDA 式 barrier。

**EN:** For simple CuTe tutorial kernels you will see `barrier_arrive()` and `barrier_wait()` helpers rather than raw SYCL group barriers.
**CN:** 在简单的 CuTe 教程内核中，你会看到 `barrier_arrive()` 与 `barrier_wait()` 这样的 helper，而不是直接使用原始 SYCL group barrier。

**EN:** This is important because Xe GEMM often uses split barriers to keep the load/prefetch/compute pipeline loosely synchronized without over-serializing the work-group.
**CN:** 这很重要，因为 Xe GEMM 常常使用 split barrier，让 load/prefetch/compute pipeline 保持松耦合同步，而不把整个 work-group 过度串行化。

**EN:** If you know CUDA well, the right migration mindset is not “translate syntax”, but “translate scheduling assumptions”.
**CN:** 如果你很熟悉 CUDA，正确的迁移心态不是“翻译语法”，而是“翻译调度假设”。

**EN:** The syntax change is easy. The scheduling change is where correctness and performance are won or lost.
**CN:** 语法变化其实很容易；真正决定正确性和性能的是调度假设的变化。

### 4.1 SYCL launch example / SYCL 启动示例

```cpp
sycl::range<2> local = {size(mma), 1};
sycl::range<2> global = {
    local[0] * ceil_div(shape<0>(B), get<1>(mma.tile_mnk())),
    local[1] * ceil_div(shape<0>(A), get<0>(mma.tile_mnk()))
};

namespace syclex = sycl::ext::oneapi::experimental;
namespace intelex = sycl::ext::intel::experimental;

syclex::properties kernel_props {
    syclex::sub_group_size<16>,
    intelex::grf_size<256>
};

Q.parallel_for(sycl::nd_range<2>(global, local), kernel_props, [=](auto) {
    gemm_device(A, B, C, mma);
});
```

**EN:** This pattern is adapted from `../examples/cute/tutorial/xe_gemm.cpp` and is the canonical low-level SYCL-TLA launch skeleton.
**CN:** 这个模式改编自 `../examples/cute/tutorial/xe_gemm.cpp`，可以视作 SYCL-TLA 的经典低层启动骨架。

**EN:** If you are coming from a CUDA CUTLASS kernel, notice that launch properties become explicit host-side arguments rather than hidden architecture assumptions.
**CN:** 如果你来自 CUDA CUTLASS 内核，请注意：启动属性在这里会变成显式的主机端参数，而不是隐含的架构假设。

**EN:** The local size is typically the number of work-items implied by the `TiledMMA`, which already includes subgroup tiling decisions.
**CN:** local size 通常就是 `TiledMMA` 隐含的 work-item 数量，而 `TiledMMA` 本身已经包含了 subgroup tiling 决策。

## 5. CuTe Extensions for Intel Xe / 面向 Intel Xe 的 CuTe 扩展

**EN:** The Intel overview document names the Xe-specific CuTe pieces very clearly: 2D block copy atoms, XMX/DPAS MMA atoms, `SubgroupTensor`, and `TiledMMAHelper`-driven assembly.
**CN:** Intel 概览文档对 Xe 特有 CuTe 组件的命名非常清晰：2D block copy atom、XMX/DPAS MMA atom、`SubgroupTensor`，以及由 `TiledMMAHelper` 驱动的组装。

**EN:** If you already know upstream CuTe, you should think of these as backend-specific atoms and helper constructors rather than a new conceptual system.
**CN:** 如果你已经熟悉上游 CuTe，应当把它们理解为后端特定的 atom 和 helper constructor，而不是全新的概念体系。

**EN:** The new DPAS family is centered on a single template: `XE_DPAS_TT<M, TypeD, TypeA, TypeB, TypeC>`.
**CN:** 新的 DPAS 家族围绕一个统一模板展开：`XE_DPAS_TT<M, TypeD, TypeA, TypeB, TypeC>`。

**EN:** The legacy per-shape named Xe MMA structs still exist for compatibility, but new code should prefer `XE_DPAS_TT`.
**CN:** 旧版按形状命名的 Xe MMA struct 仍然保留以兼容历史代码，但新代码应优先使用 `XE_DPAS_TT`。

**EN:** Similarly, the legacy `XE_2D_*` copy names still exist, but the parameterized `XE_LOAD_2D`, `XE_LOAD_2D_VNNI`, `XE_LOAD_2D_TRANSPOSE`, `XE_STORE_2D`, and `XE_PREFETCH_2D` are the preferred modern API.
**CN:** 同样，旧版 `XE_2D_*` copy 名称仍然存在，但参数化的 `XE_LOAD_2D`、`XE_LOAD_2D_VNNI`、`XE_LOAD_2D_TRANSPOSE`、`XE_STORE_2D` 与 `XE_PREFETCH_2D` 才是当前推荐的新 API。

**EN:** The modern API exists because the Xe rearchitecture wanted to expose real hardware behavior more accurately: real VNNI layout, real 2D operations, and less fake fragment bookkeeping.
**CN:** 之所以有现代 API，是因为 Xe 重构希望更准确地暴露真实硬件行为：真实的 VNNI 布局、真实的 2D 操作，以及更少“虚构的” fragment 记账。

**EN:** `TiledMMAHelper` is especially useful for CUTLASS users because it lets you keep the normal CuTe mental model while filling in Intel-specific permutation details.
**CN:** `TiledMMAHelper` 对 CUTLASS 用户尤其有用，因为它让你保留正常的 CuTe 心智模型，同时把 Intel 特有的 permutation 细节隐藏起来。

**EN:** `SubgroupTensor` is the other major shift. It models a register tile owned collectively by the subgroup, not independently by each lane.
**CN:** `SubgroupTensor` 是另一个重大变化。它建模的是由整个 subgroup 共同拥有的寄存器 tile，而不是每个 lane 各自独立拥有的 tile。

**EN:** That is why Intel-specific partition methods include `partition_sg_fragment_A/B/C` and `partition_sg_fragment_S/D` rather than only the thread-fragment style familiar from CUDA examples.
**CN:** 这也是为什么 Intel 特定的 partition 方法会提供 `partition_sg_fragment_A/B/C` 与 `partition_sg_fragment_S/D`，而不只是 CUDA 示例中熟悉的 thread-fragment 风格接口。

**EN:** The helper APIs `make_block_2d_copy_A/B/C/D` and `make_block_2d_prefetch` are intended to let most users avoid hand-authoring Xe copy traits.
**CN:** helper API `make_block_2d_copy_A/B/C/D` 与 `make_block_2d_prefetch` 的设计目标，就是让大多数用户无需手写 Xe copy trait。

**EN:** Under the hood, `copy_traits_xe_2d.hpp` chooses x/y modes, element interpretation, and copy scope so that copy layout and MMA layout match when possible.
**CN:** 在底层，`copy_traits_xe_2d.hpp` 会自动选择 x/y mode、元素解释方式以及 copy 作用域，以便在可能时让 copy 布局与 MMA 布局直接匹配。

**EN:** When they do not match, SYCL-TLA relies on `reorder` as an explicit and optimizable bridge.
**CN:** 当两者不匹配时，SYCL-TLA 就会依赖 `reorder` 作为显式且可优化的桥接操作。

**EN:** This is philosophically different from hiding layout conversion in undocumented fragment setup steps.
**CN:** 这种设计哲学与把布局转换隐藏在不透明的 fragment 构造步骤中截然不同。

**EN:** For engineers maintaining kernels, this explicitness is a major advantage because debugging becomes a question of copy layout versus MMA layout, not magic register state.
**CN:** 对维护内核的工程师来说，这种显式性是巨大的优势，因为调试时你只需分析 copy 布局与 MMA 布局的关系，而不是猜测神秘的寄存器状态。

**EN:** For engineers optimizing kernels, it is also valuable because the compiler can elide a `reorder` when the helper-selected layouts already match.
**CN:** 对优化内核的工程师来说，这也很有价值，因为如果 helper 选择出的布局已经匹配，编译器就可以把 `reorder` 彻底消掉。

**EN:** The CuTe Intel overview also reminds readers that current Intel examples recommend starting with runnable GEMM examples before hand-authoring new atom combinations.
**CN:** CuTe Intel 概览还提醒读者：当前的 Intel 示例建议先从可运行 GEMM 示例出发，再手动拼装新的 atom 组合。

**EN:** That recommendation is good engineering advice, not just educational guidance.
**CN:** 这不仅是教学建议，也是很实用的工程建议。

### 5.1 Modern Xe atom families / 现代 Xe atom 家族

```cpp
using MmaOp = cute::XE_DPAS_TT<8,
                               cute::dpas_type::f,
                               cute::dpas_type::bf,
                               cute::dpas_type::bf,
                               cute::dpas_type::f>;

using CopyA = cute::XE_LOAD_2D<16, 32, 32>;
using CopyB = cute::XE_LOAD_2D_VNNI<16, 32, 32>;
using CopyC = cute::XE_STORE_2D<32, 8, 16>;
```

**EN:** This snippet captures the spirit of the redesign: one unified DPAS family and a small, orthogonal family of parameterized 2D copy operations.
**CN:** 这段代码很好地体现了重构的精神：一个统一的 DPAS 家族，加上一组小而正交的参数化 2D copy 操作。

**EN:** The data-type aliases in `cute::dpas_type` are worth learning, because they make mixed-precision declarations much easier to scan.
**CN:** `cute::dpas_type` 中的数据类型别名很值得记住，因为它们能让混合精度声明更容易阅读。

**EN:** The copy atom parameters are also meaningful: `Bits`, `Height`, `Width`, and optionally `BlockWidth` correspond to real block-2D message configuration choices.
**CN:** copy atom 参数同样具有真实语义：`Bits`、`Height`、`Width` 和可选的 `BlockWidth` 都对应真实的 block-2D message 配置选择。

## 6. 2D Block Copy, Copy Traits, and Reorder / 2D Block Copy、Copy Trait 与 Reorder

**EN:** The Xe rearchitecture document spends a lot of effort on block-2D copy because this is one of the biggest differences from the CUDA path.
**CN:** Xe 重构文档花了很大篇幅讨论 block-2D copy，因为这是与 CUDA 路径差异最大的部分之一。

**EN:** PVC introduced hardware 2D block messages that move rectangular tiles between global memory and registers, with bounds handling and some layout transformations built into the message format.
**CN:** PVC 引入了硬件 2D block message，可在 global memory 与寄存器之间搬运矩形 tile，并在消息格式中内建越界处理与部分布局变换。

**EN:** The modern CuTe API exposes five logical operations: plain load, VNNI load, transpose load, store, and prefetch.
**CN:** 现代 CuTe API 暴露出五类逻辑操作：普通 load、VNNI load、transpose load、store 与 prefetch。

**EN:** The width dimension always refers to the contiguous-in-memory direction, not necessarily the direction that will remain contiguous after a transpose load.
**CN:** width 维始终指内存中连续的方向，而不是 transpose load 之后仍会保持连续的方向。

**EN:** This point matters because Intel block-2D terminology uses image-like x/y naming instead of linear-algebra-first row/column naming.
**CN:** 这一点很重要，因为 Intel 的 block-2D 术语更接近图像中的 x/y 命名，而不是线性代数优先的行/列命名。

**EN:** For regular loads, VNNI loads, and stores, `Width` is the total width in memory and may reflect multiple adjacent hardware blocks through the `BlockWidth` parameter.
**CN:** 对于普通 load、VNNI load 与 store，`Width` 表示内存中的总宽度；如果硬件使用多个相邻 block，则这一点会通过 `BlockWidth` 参数体现。

**EN:** The helper APIs choose these parameters heuristically from tensor layout and MMA requirements so that users do not have to manually encode x/y modes most of the time.
**CN:** helper API 会根据 tensor 布局与 MMA 需求启发式地选择这些参数，因此用户大多数时候无需手动编码 x/y mode。

**EN:** The helper family comes in two styles: work-group-scope helpers tied to an existing `TiledMMA`, and subgroup-scope helpers created from an explicit copy op plus tensor.
**CN:** helper 家族有两种风格：一种是和现有 `TiledMMA` 绑定的 work-group scope helper；另一种是基于显式 copy op 和 tensor 创建的 subgroup-scope helper。

**EN:** The former is what CUTLASS-level users usually want. The latter is what low-level CuTe kernel writers need for custom flows.
**CN:** 前者通常更适合 CUTLASS 层用户；后者则是低层 CuTe kernel 作者在自定义流程中经常需要的工具。

**EN:** Creation and use follow a proxy-copy pattern: the copy object captures the global tensor, and later you partition a coordinate tensor to specify which tile to move.
**CN:** 创建与使用遵循一种 proxy-copy 模式：copy 对象先捕获全局 tensor，之后再通过 partition 一个坐标 tensor 来指定要移动哪一块。

**EN:** This may feel unusual if you are used to plain pointer-plus-stride copies, but it aligns well with CuTe tiling and predication patterns.
**CN:** 如果你习惯了普通的 pointer-plus-stride copy，这种模式一开始可能有些陌生；但它与 CuTe 的 tiling 和 predication 体系是高度一致的。

**EN:** The Xe performance guide lists important constraints: base pointer 64B aligned, pitch usually 16B aligned, width 4B aligned, load heights up to 32 rows, and store heights up to 8 rows.
**CN:** Xe 性能指南列出了几个关键约束：基地址 64B 对齐、pitch 通常 16B 对齐、width 4B 对齐、load 高度最多 32 行，而 store 高度最多只有 8 行。

**EN:** The stronger limit on stores is especially important when designing epilogue tiles.
**CN:** store 更严格的高度限制在设计 epilogue tile 时尤其重要。

**EN:** `XE_LOAD_2D_VNNI` is only valid for 8-bit or 16-bit element sizes, which is exactly why it is a natural fit for INT8, FP16, or BF16 B operands.
**CN:** `XE_LOAD_2D_VNNI` 只支持 8 位或 16 位元素大小，这也正是它天然适合 INT8、FP16 或 BF16 B 操作数的原因。

**EN:** `XE_LOAD_2D_TRANSPOSE` is more constrained; for sub-32-bit transposed loads you frequently need a different load strategy plus `reorder`.
**CN:** `XE_LOAD_2D_TRANSPOSE` 的限制更强；对于 sub-32-bit 的转置 load，你往往需要改用其他 load 策略再配合 `reorder`。

**EN:** `reorder` is therefore not a slow-path embarrassment. It is an intentional first-class primitive for register-level layout change and type conversion.
**CN:** 因此 `reorder` 不是“不得已的慢路径”。它是一个有意设计的一等原语，用于寄存器级布局变换和类型转换。

**EN:** The rearchitecture document explicitly says that `reorder` can fuse data-type conversion with layout conversion to reduce hidden overhead.
**CN:** 重构文档明确指出，`reorder` 可以把数据类型转换与布局转换融合起来，从而减少隐藏开销。

**EN:** That is particularly relevant for dequantization and FP8 upconversion, where the storage type is not the compute type.
**CN:** 这在反量化与 FP8 上转换中尤其相关，因为存储类型往往不同于计算类型。

**EN:** The practical rule is: first try to let `make_block_2d_copy_*` choose a DPAS-ready layout; if it cannot, insert `reorder` and reason about it explicitly.
**CN:** 实践规则是：先尝试让 `make_block_2d_copy_*` 自动选出可直接喂 DPAS 的布局；如果做不到，就显式插入 `reorder` 并围绕它进行分析。

### 6.1 Low-level CuTe GEMM loop / 低层 CuTe GEMM 循环

```cpp
for (int k_tile = 0; k_tile < k_tile_count; k_tile++, k_tile_prefetch++) {
  barrier_arrive(barrier_scope);

  copy(copy_a, tAgA(_,_,_,k_tile), tArA);
  copy(copy_b, tBgB(_,_,_,k_tile), tBrB);

  prefetch(prefetch_a, pAgA(_,_,_,k_tile_prefetch));
  prefetch(prefetch_b, pBgB(_,_,_,k_tile_prefetch));

  reorder(tArA, tCrA);
  reorder(tBrB, tCrB);

  gemm(mma, tCrA, tCrB, tCrC);
  barrier_wait(barrier_scope);
}
```

**EN:** This excerpt from `../examples/cute/tutorial/xe_gemm.cpp` is the simplest place to see the Intel Xe data path in one loop.
**CN:** 这段来自 `../examples/cute/tutorial/xe_gemm.cpp` 的代码，是观察 Intel Xe 数据路径的最简单窗口。

**EN:** The important order is prefetch -> copy -> reorder -> gemm, not because the names are new, but because the hardware assumptions under each stage are new.
**CN:** 关键顺序是 prefetch -> copy -> reorder -> gemm；重要的不是名字是否陌生，而是每个阶段背后的硬件假设已经改变。

**EN:** If copy fragments already match MMA fragments, the compiler can often erase `reorder`.
**CN:** 如果 copy fragment 已经和 MMA fragment 匹配，编译器通常可以把 `reorder` 消掉。

**EN:** If they do not match, the reorder remains and should be treated as an explicit, measurable transformation rather than mysterious overhead.
**CN:** 如果二者不匹配，reorder 就会保留下来，此时应把它视作一个显式、可测量的变换，而不是神秘开销。

## 7. GEMM on Intel GPUs / Intel GPU 上的 GEMM

**EN:** At the lowest level, Xe GEMM is still a tiled mainloop plus epilogue design, but the mainloop is specialized around subgroup-cooperative block-2D loads and DPAS atoms.
**CN:** 在最低层面上，Xe GEMM 仍然是 tiled mainloop 加 epilogue 的设计，但 mainloop 围绕 subgroup 协作的 block-2D load 与 DPAS atom 做了专门化。

**EN:** At the CUTLASS level, the workhorse kernel family lives under headers such as `xe_gemm.hpp`, `xe_gemm_cooperative.hpp`, and `xe_gemm_array_cooperative.hpp`.
**CN:** 在 CUTLASS 层，核心 kernel 家族位于 `xe_gemm.hpp`、`xe_gemm_cooperative.hpp` 与 `xe_gemm_array_cooperative.hpp` 等头文件中。

**EN:** The builder path for Intel Xe mainloops is `include/cutlass/gemm/collective/builders/xe_mma_builder.inl`.
**CN:** Intel Xe mainloop 的 builder 路径位于 `include/cutlass/gemm/collective/builders/xe_mma_builder.inl`。

**EN:** That builder chooses a suitable `XE_DPAS_TT` atom, constructs a `TiledMMA`, selects subgroup tiling, and defaults to an L1-staged mainloop.
**CN:** 该 builder 会选择合适的 `XE_DPAS_TT` atom，构造 `TiledMMA`，确定 subgroup tiling，并默认选择 L1-staged mainloop。

**EN:** For ordinary Xe GEMM, `StageCountAuto` becomes a practical default. The builder uses 3 stages for the standard cooperative path and 2 stages for grouped pointer-array cooperative kernels.
**CN:** 对常规 Xe GEMM 来说，`StageCountAuto` 是很实用的默认选项。builder 在标准 cooperative 路径中会使用 3 个 stage，而在 grouped pointer-array cooperative kernel 中默认使用 2 个 stage。

**EN:** The builder also fixes `ClusterShape` to `<1,1,1>` for the current Intel Xe path.
**CN:** builder 还会把当前 Intel Xe 路径的 `ClusterShape` 固定为 `<1,1,1>`。

**EN:** This is one of several clues that current Xe support is optimized around subgroup and work-group decomposition, not around NVIDIA-style cluster features.
**CN:** 这也是一个明显信号：当前 Xe 支持的优化中心是 subgroup 与 work-group 分解，而不是 NVIDIA 风格的 cluster 特性。

**EN:** The default data path is a cooperative L1-staged mainloop, represented by dispatch policies such as `MainloopXeL1Staged`.
**CN:** 默认数据路径是一种 cooperative 的 L1-staged mainloop，由 `MainloopXeL1Staged` 之类的 dispatch policy 表示。

**EN:** Grouped kernels use related policies such as `MainloopXeL1StagedGroup` together with pointer-array cooperative kernels.
**CN:** grouped kernel 则使用 `MainloopXeL1StagedGroup` 等相关 policy，并搭配 pointer-array cooperative kernel。

**EN:** The cooperative kernel specialization in `xe_gemm_cooperative.hpp` only supports `StreamKScheduler` and `PersistentScheduler`, not `GroupScheduler`.
**CN:** `xe_gemm_cooperative.hpp` 中的 cooperative kernel 特化只支持 `StreamKScheduler` 与 `PersistentScheduler`，并不支持 `GroupScheduler`。

**EN:** The pointer-array cooperative specialization in `xe_gemm_array_cooperative.hpp` is where grouped GEMM lives, and it pairs naturally with `xe_array_epilogue`.
**CN:** `xe_gemm_array_cooperative.hpp` 中的 pointer-array cooperative 特化才是 grouped GEMM 的主要落点，它也会自然搭配 `xe_array_epilogue`。

**EN:** The stream-K scheduler on Xe tracks `M_idx`, `N_idx`, `K_idx`, `L_idx`, `k_tile_count`, and `k_tile_remaining`, which is a direct clue that one work item may span multiple K tile chunks.
**CN:** Xe 上的 stream-K scheduler 会跟踪 `M_idx`、`N_idx`、`K_idx`、`L_idx`、`k_tile_count` 和 `k_tile_remaining`，这直接说明一个 work item 可能覆盖多个 K tile 分块。

**EN:** The grouped scheduler tracks problem arrays and linearized tiles across multiple GEMM problems rather than split-K progress.
**CN:** grouped scheduler 则会跟踪多个 GEMM 问题上的 problem array 与线性化 tile，而不是 split-K 进度。

**EN:** This scheduler distinction matters because it changes not only work assignment but also what epilogue and reduction behavior are legal.
**CN:** 这种 scheduler 差异之所以重要，是因为它不只影响 work assignment，也决定了哪些 epilogue 与 reduction 行为是合法的。

**EN:** The example `../examples/00_bmg_gemm/00_bmg_gemm.cpp` is the reference starting point for standard BF16 GEMM on BMG.
**CN:** `../examples/00_bmg_gemm/00_bmg_gemm.cpp` 是 BMG 上标准 BF16 GEMM 的参考起点。

**EN:** The example `../examples/01_bmg_gemm_with_collective_builder/01_bmg_gemm_with_collective_builder.cpp` shows the shortest modern builder-based path.
**CN:** `../examples/01_bmg_gemm_with_collective_builder/01_bmg_gemm_with_collective_builder.cpp` 展示了最简洁的现代 builder 路径。

**EN:** The low-level tutorials `xe_gemm.cpp`, `xe_gemm_slm.cpp`, and `xe_gemm_subgroup_specialization_slm.cpp` are valuable because they expose the machinery that CUTLASS-level examples intentionally hide.
**CN:** 底层教程 `xe_gemm.cpp`、`xe_gemm_slm.cpp` 与 `xe_gemm_subgroup_specialization_slm.cpp` 很有价值，因为它们显式展示了 CUTLASS 层示例刻意隐藏的底层机制。

**EN:** As a porting strategy, start from the builder example unless you need custom scheduling, custom copies, unusual dequantization, or an SLM pipeline.
**CN:** 从迁移策略上讲，除非你需要自定义调度、自定义 copy、非常规反量化或 SLM pipeline，否则应先从 builder 示例开始。

**EN:** Drop down to low-level CuTe only when the policy layer stops expressing what your kernel needs.
**CN:** 只有当 policy 层已经无法表达你的 kernel 需求时，才应该下沉到低层 CuTe。

### 7.1 Builder-based GEMM / 基于 Builder 的 GEMM

```cpp
using TileShape = Shape<_256, _256, _32>;

using CollectiveMainloop = cutlass::gemm::collective::CollectiveBuilder<
    cutlass::arch::IntelXe, cutlass::arch::OpClassTensorOp,
    ElementInputA, LayoutA, AlignmentA,
    ElementInputB, LayoutB, AlignmentB,
    ElementAccumulator,
    TileShape, Shape<_1, _1, _1>,
    cutlass::gemm::collective::StageCountAuto,
    cutlass::gemm::collective::KernelScheduleAuto
>::CollectiveOp;
```

**EN:** This excerpt from `../examples/01_bmg_gemm_with_collective_builder/01_bmg_gemm_with_collective_builder.cpp` is the preferred high-level entry point for many new kernels.
**CN:** 这段来自 `../examples/01_bmg_gemm_with_collective_builder/01_bmg_gemm_with_collective_builder.cpp` 的代码，是很多新 kernel 的首选高层入口。

**EN:** The important part is not the syntax itself, but what you no longer have to spell out: the DPAS atom, subgroup layout, and default copy strategy are inferred.
**CN:** 真正重要的并不是语法本身，而是你不再需要手写的那些内容：DPAS atom、subgroup 布局和默认 copy 策略都会被推导出来。

**EN:** That inference is specifically Xe-aware because it uses the Intel Xe builder specialization.
**CN:** 这种推导之所以成立，是因为它调用的是 Intel Xe 专门化 builder。

### 7.2 Low-level CUTLASS mainloop / 低层 CUTLASS 主循环

```cpp
using TiledMma = typename TiledMMAHelper<
    MMA_Atom<XE_DPAS_TT<8, float, cute::bfloat16_t>>,
    Layout<TileShape>,
    Layout<Shape<_8, _4, _1>, Stride<_4, _1, _0>>
>::TiledMMA;

constexpr int PipelineStages = 2;
using GEMMDispatchPolicy = cutlass::gemm::MainloopXeL1Staged<PipelineStages>;
```

**EN:** This lower-level style, shown in `../examples/00_bmg_gemm/00_bmg_gemm.cpp`, is ideal when you want to expose and tune the exact Xe mainloop ingredients.
**CN:** 这种更底层的写法出现在 `../examples/00_bmg_gemm/00_bmg_gemm.cpp` 中，适合那些需要暴露并调优 Xe mainloop 精确组成元素的场景。

**EN:** The explicit `Layout<Shape<_8,_4,_1>, Stride<_4,_1,_0>>` is the classic 8x4 subgroup arrangement for a 256x256 tile.
**CN:** 显式的 `Layout<Shape<_8,_4,_1>, Stride<_4,_1,_0>>` 是 256x256 tile 上经典的 8x4 subgroup 布局。

**EN:** The stride order makes the arrangement N-major, which helps adjacent subgroups share B-side cache lines efficiently.
**CN:** 其 stride 顺序让布局在 N 方向优先，这有助于相邻 subgroup 更高效地共享 B 侧 cache line。

## 8. Epilogue System: xe_epilogue, xe_array_epilogue, and EVT / Epilogue 系统：xe_epilogue、xe_array_epilogue 与 EVT

**EN:** The Xe epilogue path keeps the CUTLASS 3.x worldview: epilogues are collectives, fusion is represented through callbacks and visitor trees, and the epilogue can decide whether it needs C loads or auxiliary stores.
**CN:** Xe epilogue 路径保留了 CUTLASS 3.x 的世界观：epilogue 仍然是 collective，fusion 通过 callback 与 visitor tree 表示，epilogue 也可以自行决定是否需要加载 C 或写入辅助张量。

**EN:** What changes is the implementation substrate: Xe uses 2D load/store copy atoms, subgroup-aware reorder, and Xe-specialized fusion callback types.
**CN:** 变化的是实现底座：Xe 使用 2D load/store copy atom、subgroup 感知的 reorder，以及 Xe 特化的 fusion callback 类型。

**EN:** The primary header for the common path is `include/cutlass/epilogue/collective/xe_epilogue.hpp`.
**CN:** 公共路径的主头文件是 `include/cutlass/epilogue/collective/xe_epilogue.hpp`。

**EN:** The grouped pointer-array counterpart is `include/cutlass/epilogue/collective/xe_array_epilogue.hpp`.
**CN:** 与 grouped pointer-array 对应的头文件是 `include/cutlass/epilogue/collective/xe_array_epilogue.hpp`。

**EN:** The Intel epilogue builder specialization lives in `include/cutlass/epilogue/collective/builders/xe_builder.inl`.
**CN:** Intel epilogue builder 特化位于 `include/cutlass/epilogue/collective/builders/xe_builder.inl`。

**EN:** That builder forwards Xe12 and Xe20 to the shared IntelXe implementation and defaults to `IntelXeGeneric` or `IntelXeGenericGroup` epilogue schedules.
**CN:** 该 builder 会把 Xe12 与 Xe20 转发到共享的 IntelXe 实现，并默认使用 `IntelXeGeneric` 或 `IntelXeGenericGroup` epilogue schedule。

**EN:** The grouped schedule matters because grouped GEMM passes arrays of pointers and strides for C and D, not single tensors.
**CN:** grouped schedule 之所以重要，是因为 grouped GEMM 传入的是 C 和 D 的指针数组与 stride 数组，而不是单个 tensor。

**EN:** Inside `xe_epilogue.hpp`, the epilogue first derives a default epilogue tile from the MMA tile and cache-line preferences.
**CN:** 在 `xe_epilogue.hpp` 中，epilogue 会先根据 MMA tile 和 cache-line 偏好推导默认 epilogue tile。

**EN:** It then chooses default C-load and D-store operations automatically, including transpose loads for column-major C when legal.
**CN:** 之后它会自动选择默认的 C load 与 D store 操作；如果 C 是列主且满足条件，也会自动选择 transpose load。

**EN:** The epilogue has an identity fast path: if the fusion callback reports `alpha == 1` and `beta == 0`, the implementation can skip the full visitor machinery and just reorder plus store the accumulators.
**CN:** epilogue 提供了 identity fast path：如果 fusion callback 报告 `alpha == 1` 且 `beta == 0`，实现就可以跳过完整 visitor 机制，直接把 accumulator 做 reorder 后写回。

**EN:** This is an important optimization because the most common epilogue in many benchmarks is indeed plain store or linear combination with beta zero.
**CN:** 这是一个很重要的优化，因为在很多 benchmark 中，最常见的 epilogue 确实就是简单 store，或者 beta 为零的线性组合。

**EN:** When the epilogue is not identity, Xe epilogue loads C if needed, reorders it to the compute layout, runs callbacks on vector fragments, performs reduction hooks if needed, and finally stores D.
**CN:** 当 epilogue 不是 identity 时，Xe epilogue 会在需要时加载 C，把它 reorder 到计算布局，对向量片段执行 callback，在需要时进行 reduction hook，最后再存储 D。

**EN:** The callback choreography is very CUTLASS 3.x in spirit: `begin`, `begin_loop`, `previsit`, `visit`, `reduce`, `end_loop`, `end`.
**CN:** callback 的编排在精神上非常 CUTLASS 3.x：`begin`、`begin_loop`、`previsit`、`visit`、`reduce`、`end_loop`、`end`。

**EN:** The Xe-specific part is how the fragments are moved and represented around that choreography.
**CN:** Xe 特有的部分，是围绕这套 choreography 如何移动和表示 fragment。

**EN:** `FusionCallbacks` specializations in `xe_callbacks.hpp` cover `LinearCombination`, `LinCombEltAct`, and Xe-specific split-K combinations.
**CN:** `xe_callbacks.hpp` 中的 `FusionCallbacks` 特化覆盖了 `LinearCombination`、`LinCombEltAct` 和 Xe 特定的 split-K 组合。

**EN:** The linear-combination specialization exposes `is_identity()`, which is how the epilogue fast path is triggered.
**CN:** 线性组合特化暴露了 `is_identity()`，这正是 epilogue fast path 的触发条件。

**EN:** The visitor-side helper header `xe_visitor.hpp` adds Xe-oriented building blocks such as `XeAuxStore` and reduction workspace helpers.
**CN:** visitor 侧的辅助头文件 `xe_visitor.hpp` 还引入了 `XeAuxStore` 与 reduction workspace helper 等 Xe 导向构件。

**EN:** `XeAuxStore` is important because many fused kernels want to emit an auxiliary tensor such as probabilities, bias intermediates, or top-k metadata.
**CN:** `XeAuxStore` 很重要，因为很多融合 kernel 想同时输出一个辅助 tensor，例如概率矩阵、bias 中间值或者 top-k 元数据。

**EN:** Its implementation can choose either a scalar/vectorized path or a block-2D store path, depending on copy op availability and alignment.
**CN:** 它的实现可以根据 copy op 是否可用以及对齐情况，在 scalar/vectorized 路径和 block-2D store 路径之间选择。

**EN:** This is an example of a broader pattern in SYCL-TLA: Intel-specific pieces are exposed as orthogonal building blocks, not embedded as one-off hacks in a single kernel.
**CN:** 这体现了 SYCL-TLA 的一个更广泛模式：Intel 特定部件被设计成正交构件，而不是作为某个单独 kernel 里的特例 hack。

**EN:** The examples under `../examples/05_bmg_gemm_with_epilogues` are the best place to see these pieces used in complete kernels.
**CN:** `../examples/05_bmg_gemm_with_epilogues` 中的示例，是观察这些构件如何在完整 kernel 中协同工作的最佳入口。

### 8.1 GELU epilogue example / GELU epilogue 示例

```cpp
using EpilogueOp = cutlass::epilogue::fusion::LinCombEltAct<
    cutlass::epilogue::thread::GELU,
    ElementOutput,
    ElementComputeEpilogue,
    ElementAccumulator,
    ElementAccumulator,
    cutlass::FloatRoundStyle::round_to_nearest>;
```

**EN:** This is the exact style used in `../examples/05_bmg_gemm_with_epilogues/05_bmg_gemm_with_epilogue_gelu.cpp`.
**CN:** 这正是 `../examples/05_bmg_gemm_with_epilogues/05_bmg_gemm_with_epilogue_gelu.cpp` 使用的风格。

**EN:** Conceptually this is still an EVT epilogue, but on Xe it executes through Xe callback specializations and Xe epilogue collectives.
**CN:** 从概念上讲，这仍然是 EVT epilogue；但在 Xe 上，它会通过 Xe callback 特化和 Xe epilogue collective 来执行。

**EN:** If you understand EVT on CUTLASS 3.x for NVIDIA, you do not need to relearn fusion semantics. You only need to learn the Xe implementation substrate.
**CN:** 如果你已经理解 NVIDIA 上 CUTLASS 3.x 的 EVT，那么不必重新学习 fusion 语义；你只需要理解 Xe 的实现底座即可。

### 8.2 xe_array_epilogue / xe_array_epilogue

**EN:** `xe_array_epilogue.hpp` is the grouped/pointer-array counterpart of the standard Xe epilogue.
**CN:** `xe_array_epilogue.hpp` 是标准 Xe epilogue 的 grouped/pointer-array 对应版本。

**EN:** It is relevant when each GEMM problem in a grouped launch has its own C and D pointers and possibly its own stride descriptors.
**CN:** 当 grouped launch 中的每个 GEMM 问题都拥有各自的 C、D 指针以及可能不同的 stride 描述时，这个文件就会发挥作用。

**EN:** That is why its argument structure carries `ElementC const**` and `ElementD**` rather than single pointers.
**CN:** 这也是它的参数结构会携带 `ElementC const**` 和 `ElementD**`，而不是单一指针的原因。

**EN:** You normally encounter it indirectly through grouped GEMM examples and grouped builder-driven kernels rather than instantiating it manually first.
**CN:** 通常你会先在 grouped GEMM 示例和 grouped builder 内核中间接遇到它，而不是一上来就手工实例化它。

## 9. Mixed Precision and Quantization / 混合精度与量化

**EN:** SYCL-TLA places unusual emphasis on mixed precision because Intel Xe kernels frequently combine storage types, compute types, and epilogue/output types that do not match.
**CN:** SYCL-TLA 特别强调混合精度，因为 Intel Xe 内核经常需要同时处理彼此不相同的存储类型、计算类型以及 epilogue/输出类型。

**EN:** The README explicitly highlights support for FP16, BF16, FP8, INT8, INT4, tensor-wise quantization, channel-wise quantization, and group-wise quantization.
**CN:** README 明确强调了对 FP16、BF16、FP8、INT8、INT4、tensor-wise quantization、channel-wise quantization 和 group-wise quantization 的支持。

**EN:** At the CuTe layer, the critical mechanisms are DPAS type combinations, VNNI-aware loads, and reorder operations that can also change data type.
**CN:** 在 CuTe 层，关键机制是 DPAS 的类型组合、VNNI 感知 load，以及可同时完成数据类型变换的 reorder 操作。

**EN:** At the CUTLASS collective layer, the important additions are mixed-input mainloop policies and tuple-style operand descriptors that carry quantized data together with scale and zero-point metadata.
**CN:** 在 CUTLASS collective 层，重要新增内容是 mixed-input mainloop policy，以及以 tuple 方式描述量化操作数并携带 scale/zero-point 元数据的接口。

**EN:** For example, the mixed-precision example with FP16 A and INT8 B uses `MainloopIntelXeXMX16MixedPrecision`.
**CN:** 例如，使用 FP16 A 和 INT8 B 的混合精度示例会使用 `MainloopIntelXeXMX16MixedPrecision`。

**EN:** The B operand can be represented as a tuple of quantized element type, scale type, scale stride, zero type, and zero stride.
**CN:** B 操作数可以表示为一个 tuple，其中包含量化元素类型、scale 类型、scale stride、zero 类型和 zero stride。

**EN:** This is a practical difference from vanilla CUTLASS tutorials, where mixed-input handling is often described abstractly but not tied to Intel-specific VNNI/reorder constraints.
**CN:** 这与普通 CUTLASS 教程的不同点在于：后者常从抽象层面讲混合输入，而不会把它与 Intel 特有的 VNNI/reorder 约束紧密绑在一起。

**EN:** The Xe rearchitecture document explicitly calls out the need to expose VNNI layouts so the compiler does not insert hidden interleave/deinterleave traffic around DPAS.
**CN:** Xe 重构文档明确指出，必须显式暴露 VNNI 布局，才能避免编译器在 DPAS 前后插入隐藏的 interleave/deinterleave 流量。

**EN:** That is one reason why sub-byte and quantized paths deserve first-class treatment instead of being bolted on later.
**CN:** 这也是为什么 sub-byte 和量化路径需要一开始就作为一等公民设计，而不是事后补丁式加入。

**EN:** The FP8 example on BMG is also revealing: the kernel computes with FP16 MMA after upconverting FP8 inputs.
**CN:** BMG 上的 FP8 示例也很有启发性：它会先把 FP8 输入上转换，再使用 FP16 MMA 进行计算。

**EN:** This is a good reminder that “supports FP8” does not necessarily mean the hardware atom itself consumes FP8 directly in the same way all vendor backends do.
**CN:** 这很好地提醒我们：“支持 FP8” 并不一定意味着硬件 atom 会像所有厂商后端那样直接以 FP8 原生消费数据。

**EN:** In SYCL-TLA, the important engineering question is where the upconversion or dequantization happens and whether it can be fused with reorder or copy.
**CN:** 在 SYCL-TLA 中，更重要的工程问题是：上转换或反量化发生在哪里，以及它能否与 reorder 或 copy 融合。

**EN:** The examples under `../examples/02_bmg_gemm_mixed_dtype` are especially useful because they show several combinations: BF16/S8, F16/U4/F16, F16/U4/S8, and tensor-wise scaling flows.
**CN:** `../examples/02_bmg_gemm_mixed_dtype` 下的示例尤其有用，因为它们展示了多种组合：BF16/S8、F16/U4/F16、F16/U4/S8，以及 tensor-wise scaling 流程。

**EN:** The dequantization helper logic in those examples also shows why sub-byte zero points and group-wise metadata add real layout complexity.
**CN:** 这些示例中的反量化辅助逻辑也清楚说明了：sub-byte zero point 与 group-wise 元数据会带来真实的布局复杂度。

**EN:** You should expect more layout algebra and more explicit tensor manipulation in quantized kernels than in plain BF16 kernels.
**CN:** 因此，与普通 BF16 kernel 相比，量化 kernel 往往需要更多 layout algebra 和更显式的 tensor 操作。

**EN:** That is normal on Xe, not a sign that you are doing something wrong.
**CN:** 这在 Xe 上是正常现象，并不意味着你的设计出了问题。

**EN:** The right optimization question is whether the required conversions are happening in the cheapest stage of the pipeline.
**CN:** 正确的优化问题是：这些必要转换是否发生在 pipeline 中代价最低的阶段。

**EN:** Very often, the answer involves choosing the right copy atom, the right reorder form, and the right accumulator/output type combination together.
**CN:** 很多时候，答案都意味着要联动地选择合适的 copy atom、合适的 reorder 形式，以及合适的 accumulator/output 类型组合。

### 9.1 Mixed-input example / 混合输入示例

```cpp
using ElementInputA = half_t;
using ElementInputB = int8_t;
using GEMMDispatchPolicy =
    cutlass::gemm::MainloopIntelXeXMX16MixedPrecision<PipelineStages>;

using CollectiveMainloop = cutlass::gemm::collective::CollectiveMma<
    GEMMDispatchPolicy,
    TileShape,
    ElementInputA,
    cutlass::gemm::TagToStrideA_t<LayoutA>,
    cute::tuple<ElementInputB, ElementScale, StrideScale, ElementZero, StrideZero>,
    cutlass::gemm::TagToStrideB_t<LayoutB>,
    TiledMma,
    GmemTiledCopyA, void, void, cute::identity,
    GmemTiledCopyB, void, void, cute::identity>;
```

**EN:** This structure is adapted from `../examples/02_bmg_gemm_mixed_dtype/02_bmg_gemm_f16_s8_f16_tensorwise.cpp`.
**CN:** 这个结构改编自 `../examples/02_bmg_gemm_mixed_dtype/02_bmg_gemm_f16_s8_f16_tensorwise.cpp`。

**EN:** The key Intel-specific insight is that mixed precision is not only about accumulator math; it is equally about how operand B is loaded, packed, and dequantized into a DPAS-friendly shape.
**CN:** 其中最关键的 Intel 特定洞见是：混合精度不只是 accumulator 的数学问题，同样也是 B 操作数如何被加载、打包并反量化为 DPAS 友好形状的问题。

**EN:** That is why mixed-input policies and copy-selection choices appear side by side.
**CN:** 这也解释了为什么 mixed-input policy 与 copy 选择会在代码里并排出现。

### 9.2 FP8 example / FP8 示例

```cpp
using ElementInputA = cutlass::float_e4m3_t;
using ElementInputB = cutlass::float_e4m3_t;
using GmemTiledCopyA = XE_2D_U8x32x32_LD_N;
using GmemTiledCopyB = XE_2D_U8x32x32_LD_V;
using GEMMDispatchPolicy = cutlass::gemm::MainloopIntelW8A8<PipelineStages>;
```

**EN:** This style comes from `../examples/08_bmg_gemm_f8/08_bmg_gemm_f8.cpp`.
**CN:** 这种写法来自 `../examples/08_bmg_gemm_f8/08_bmg_gemm_f8.cpp`。

**EN:** The host-side verification in that example explicitly converts FP8 to FP16 before running the reference GEMM, which makes the upconversion strategy obvious.
**CN:** 该示例的主机端验证会显式先把 FP8 转成 FP16，再运行参考 GEMM，这使得其上转换策略一目了然。

**EN:** From a tutorial perspective, this is the right lesson to extract: follow the data transformation path, not just the type names.
**CN:** 从教程角度看，真正应该提炼的经验是：跟踪数据变换路径，而不是只看类型名称。

## 10. Flash Attention on BMG / BMG 上的 Flash Attention

**EN:** SYCL-TLA includes a serious Flash Attention path on BMG, not just GEMM examples.
**CN:** SYCL-TLA 在 BMG 上提供了真正可用的 Flash Attention 路径，而不仅仅是 GEMM 示例。

**EN:** The main front-end example is `../examples/06_bmg_flash_attention/06_xe_fmha_fwd.cpp`, with most option and runner logic in `../examples/06_bmg_flash_attention/xe_fmha_fwd_runner.hpp`.
**CN:** 主要前端示例是 `../examples/06_bmg_flash_attention/06_xe_fmha_fwd.cpp`，而大部分选项与 runner 逻辑则在 `../examples/06_bmg_flash_attention/xe_fmha_fwd_runner.hpp` 中。

**EN:** The file is organized around compile-time mode switches: `PREFILL`, `DECODE`, and `PERSISTENT`.
**CN:** 这个文件围绕几个编译期开关组织：`PREFILL`、`DECODE` 和 `PERSISTENT`。

**EN:** This matters because prefill and decode have very different tile shapes and pipeline needs on Xe.
**CN:** 这很重要，因为在 Xe 上，prefill 与 decode 对 tile shape 和 pipeline 的需求差异非常大。

**EN:** In decode mode, `seq_len_qo` is effectively 1, so the kernel is shaped around long KV traversal rather than large query tiles.
**CN:** 在 decode 模式下，`seq_len_qo` 基本就是 1，因此 kernel 的组织会围绕长 KV 遍历，而不是大 query tile。

**EN:** In the provided example, decode mode typically uses `PipelineStages = 1`, while prefill uses `PipelineStages = 2`.
**CN:** 在给出的示例中，decode 模式通常使用 `PipelineStages = 1`，而 prefill 则使用 `PipelineStages = 2`。

**EN:** This directly matches the performance guide note that decode kernels often want shallower pipelines.
**CN:** 这与性能指南中“decode kernel 常常希望使用更浅 pipeline”的结论完全一致。

**EN:** Persistent mode changes the scheduling story again: instead of short-lived work-groups, you aim to keep work resident and recycle it over more tiles.
**CN:** persistent 模式会再次改变调度逻辑：目标不再是短生命周期 work-group，而是尽量保持工作驻留并在更多 tile 之间复用。

**EN:** The runner exposes scheduler choice as `Individual` versus `Persistent`, which is valuable because it surfaces scheduling as an application-level knob.
**CN:** runner 将调度方式暴露为 `Individual` 与 `Persistent` 两种选择，这非常有价值，因为它把调度真正提升成了应用层可调参数。

**EN:** The FMHA runner also supports causal masking, variable-length sequences, and paged KV cache.
**CN:** FMHA runner 还支持 causal mask、variable-length 序列以及 paged KV cache。

**EN:** Paged KV is especially relevant for inference systems where KV storage is not contiguous.
**CN:** 在推理系统中，paged KV 尤其重要，因为这类系统中的 KV 存储往往并不连续。

**EN:** The option parser requires page size to be a multiple of 128 and validates cache sequence length divisibility accordingly.
**CN:** 该选项解析器要求 page size 必须是 128 的倍数，并会据此检查 KV cache 序列长度是否可整除。

**EN:** This tells you something important about the hardware path: alignment and paging granularity remain first-class constraints even at the attention level.
**CN:** 这也揭示了一个重要事实：即便在 attention 层面，对齐和分页粒度仍然是第一类约束。

**EN:** When you study the example, focus less on the command-line plumbing and more on how tile shapes change with `HEAD_DIM`, mode, and persistence.
**CN:** 阅读该示例时，应少关注命令行封装，多关注 tile shape 如何随着 `HEAD_DIM`、模式以及 persistence 选项而变化。

**EN:** For example, prefill with `HEAD_DIM == 128` uses `Shape<_256, _32, _32>` for both QK and PV stages, while decode uses KV-tile-oriented shapes.
**CN:** 例如，在 prefill 且 `HEAD_DIM == 128` 时，QK 与 PV 阶段都使用 `Shape<_256, _32, _32>`；而在 decode 模式下则会转向以 KV tile 为中心的形状。

**EN:** That is exactly the kind of architectural adaptation SYCL-TLA is about: keep the algorithm, retune the decomposition for the target hardware path.
**CN:** 这正是 SYCL-TLA 的核心思路：算法不变，但要为目标硬件路径重新调整分解方式。

**EN:** In other words, Flash Attention on Xe is not “special” because attention is special; it is special because it exercises the same Xe extension points under a more demanding schedule.
**CN:** 换句话说，Xe 上的 Flash Attention 并不是因为 attention 自身“特殊”才特殊；真正特殊的是它在更严苛的调度下完整激活了同一套 Xe 扩展点。

### 10.1 FMHA mode selection / FMHA 模式选择

```cpp
#ifdef DECODE
  constexpr int PipelineStages = 1;
#else
  constexpr int PipelineStages = 2;
#endif

#if PERSISTENT
  return FMHAConfig<false, ShapeQK, ShapePV, ShapeOut,
                    SubgroupLayoutQK, void, PipelineStages,
                    /*persistent=*/true, ElementQ, ElementK, ElementV>::run(options);
#endif
```

**EN:** This condensed fragment captures the tuning message: pipeline depth and scheduling mode are part of the algorithm contract on Xe.
**CN:** 这段压缩代码体现了核心调优信息：在 Xe 上，pipeline 深度与调度模式都是算法契约的一部分。

**EN:** If you port Flash Attention from another backend, treat these choices as first-order design parameters, not late-stage micro-optimizations.
**CN:** 如果你从其他后端迁移 Flash Attention，请把这些选择视为一等设计参数，而不是最后阶段的微优化。

## 11. SLM (Shared Local Memory) / SLM（Shared Local Memory）

**EN:** The SLM pipeline document is intentionally careful: it does not say SLM is always faster, and it does not present SLM as the default Xe GEMM path.
**CN:** SLM pipeline 文档的态度非常谨慎：它并没有说 SLM 总是更快，也没有把 SLM 描述成 Xe GEMM 的默认路径。

**EN:** Instead, it gives a checklist for when SLM becomes necessary or profitable.
**CN:** 相反，它给出的是“什么时候 SLM 才值得使用”的判断清单。

**EN:** The document lists expensive A/B preprocessing, unavailable block-2D loads, gather-heavy patterns such as MoE or GQA, and cross-subgroup data sharing as typical reasons.
**CN:** 文档列出的典型原因包括：A/B 预处理代价高、block-2D load 不可用、MoE/GQA 这类 gather 密集模式，以及跨 subgroup 数据共享。

**EN:** The central API additions are cooperative global-load constructors, SLM layout constructors, and SLM copy constructors.
**CN:** 核心 API 新增主要有三类：cooperative global-load constructor、SLM layout constructor，以及 SLM copy constructor。

**EN:** For global to register loading, the proposed high-level entry points are `make_coop_block_2d_copy_A/B`.
**CN:** 对于 global 到寄存器的加载，高层入口是 `make_coop_block_2d_copy_A/B`。

**EN:** For layouting SLM to match DPAS needs, the key helpers are `make_A_slm_layout` and `make_B_slm_layout`.
**CN:** 为了让 SLM 布局匹配 DPAS 需求，关键 helper 是 `make_A_slm_layout` 与 `make_B_slm_layout`。

**EN:** For building the register-to-SLM and SLM-to-register copies, the main helpers are `make_A_slm_copies` and `make_B_slm_copies`.
**CN:** 用于构造寄存器到 SLM 以及 SLM 到寄存器 copy 的主要 helper 是 `make_A_slm_copies` 与 `make_B_slm_copies`。

**EN:** There is also a generic `make_slm_copy` family for copying between `SubgroupTensor` fragments and SLM tensors.
**CN:** 此外还提供了通用的 `make_slm_copy` 家族，用于在 `SubgroupTensor` fragment 与 SLM tensor 之间复制数据。

**EN:** The recommended layout principle is simple but important: arrange SLM so each subgroup can read the data it needs contiguously for MMA.
**CN:** 推荐的布局原则简单但非常重要：在 SLM 中安排数据，使每个 subgroup 都能以对 MMA 友好的连续方式读取自己需要的数据。

**EN:** In practice that means the SLM layout is often chosen to match the layout DPAS wants, not the layout the global tensor happened to have.
**CN:** 实践中，这意味着 SLM 布局通常是围绕 DPAS 想要的布局来选的，而不是沿袭 global tensor 原本的布局。

**EN:** The `xe_gemm_slm.cpp` tutorial shows a double-buffered SLM path with `make_coop_block_2d_copy_*`, `make_A_slm_copies`, barriers with work-group memory semantics, and two SLM stages.
**CN:** `xe_gemm_slm.cpp` 教程展示了一个双缓冲 SLM 路径，其中包括 `make_coop_block_2d_copy_*`、`make_A_slm_copies`、带工作组内存语义的 barrier，以及两个 SLM stage。

**EN:** The `xe_gemm_subgroup_specialization_slm.cpp` tutorial goes further by splitting producer and consumer roles across work-items.
**CN:** `xe_gemm_subgroup_specialization_slm.cpp` 教程更进一步，把不同 work-item 划分成 producer 和 consumer 两类角色。

**EN:** That is subgroup specialization: some lanes focus on moving data into SLM while others focus on consuming it for DPAS.
**CN:** 这就是 subgroup specialization：一部分 lane 专门负责把数据搬进 SLM，另一部分 lane 专门负责消费这些数据去做 DPAS。

**EN:** This is not always the best answer, but it becomes attractive when the data-movement path and compute path have very different pressure profiles.
**CN:** 这并不总是最优解，但当数据搬运路径与计算路径的压力分布差异很大时，它就会很有吸引力。

**EN:** The performance guide reinforces the same message: use SLM deliberately, not reflexively.
**CN:** 性能指南也再次强调了同样的信息：要有意识地使用 SLM，而不是条件反射地把它加进去。

**EN:** If a direct block-2D-to-GRF path works and register pressure is acceptable, that path is often simpler and faster.
**CN:** 如果 block-2D 到 GRF 的直达路径已经可用，并且寄存器压力可接受，那么它通常会更简单也更快。

**EN:** If register pressure or cross-subgroup reuse dominates, then SLM can become the better contract.
**CN:** 如果寄存器压力或跨 subgroup 复用开始主导成本，SLM 才会成为更合适的契约。

### 11.1 SLM pipeline sketch / SLM pipeline 示例骨架

```cpp
auto copy_a_g2r = make_coop_block_2d_copy_A(tiled_mma, A);
auto [copy_a_r2s, copy_a_s2r] = make_A_slm_copies(tiled_mma, copy_a_g2r);

copy(copy_a_g2r, tAgA, tArA_in);
reorder(tArA_in, tArA_out);
copy(copy_a_r2s, tArA_out, tAsA_out);
barrier_arrive(SPIRVScope::ScopeWorkgroup,
               SPIRVMemorySemantics::SemanticsRelease |
               SPIRVMemorySemantics::SemanticsWGMemory);
barrier_wait(SPIRVScope::ScopeWorkgroup,
             SPIRVMemorySemantics::SemanticsAcquire |
             SPIRVMemorySemantics::SemanticsWGMemory);
copy(copy_a_s2r, tAsA_in, tCrA_in);
```

**EN:** This condensed pattern comes from the SLM tutorial path and makes the extra contract obvious: SLM adds copies and barriers, so it must earn those costs.
**CN:** 这段压缩模式来自 SLM 教程路径，并清楚暴露了额外契约：SLM 会引入额外 copy 与 barrier，因此它必须证明自己值得这些成本。

**EN:** The reward is that you can control sharing, staging, and expensive preprocessing more explicitly than in the direct L1-to-GRF path.
**CN:** 它带来的回报是：相比直接的 L1 到 GRF 路径，你可以更显式地控制共享、分级存储以及昂贵预处理。

## 12. Python Interface / Python 接口

**EN:** SYCL-TLA keeps the CUTLASS idea of a generated operation library and runtime manifest, and extends it to Xe architectures and Xe-specific emitted code.
**CN:** SYCL-TLA 保留了 CUTLASS 的“生成操作库 + 运行时 manifest”思想，并将其扩展到 Xe 架构和 Xe 特定的发射代码。

**EN:** The tutorial document `xe_cutlass_library.md` describes a two-phase system: build-time Python generation and runtime C++ registration into a `Manifest`.
**CN:** 教程文档 `xe_cutlass_library.md` 将其描述为两阶段系统：构建期 Python 生成，以及运行期 C++ 注册到 `Manifest`。

**EN:** At build time, Python filters operations by architecture and emits C++ initialization files.
**CN:** 在构建期，Python 会按架构过滤操作，并生成 C++ 初始化文件。

**EN:** At runtime, the generated code registers operations such as Xe20 GEMM kernels into a searchable catalog.
**CN:** 在运行期，生成的代码会把 Xe20 GEMM 等操作注册进一个可搜索目录。

**EN:** The key C++ runtime APIs include `cutlass::library::Manifest`, `initialize_all`, `initialize_all_gemm_operations`, and `initialize_all_xe20_gemm_operations`.
**CN:** 关键的 C++ 运行时 API 包括 `cutlass::library::Manifest`、`initialize_all`、`initialize_all_gemm_operations` 以及 `initialize_all_xe20_gemm_operations`。

**EN:** The example `../examples/11_xe20_cutlass_library` packages generated kernels into a shared library that Python can load via `ctypes`.
**CN:** 示例 `../examples/11_xe20_cutlass_library` 会把生成的 kernel 打包成共享库，供 Python 通过 `ctypes` 加载。

**EN:** The example `../examples/python/cutlass_library/xe20_gemm_bf16.py` shows the practical runtime protocol: load library, query workspace size, allocate workspace, then execute.
**CN:** `../examples/python/cutlass_library/xe20_gemm_bf16.py` 展示了最实用的运行协议：加载库、查询 workspace 大小、分配 workspace，然后执行。

**EN:** This protocol is important because many generated CUTLASS/SYCL-TLA operations require workspace for scheduling or reduction state.
**CN:** 这个协议非常重要，因为许多生成式 CUTLASS/SYCL-TLA 操作都需要 workspace 来保存调度或归约状态。

**EN:** The Python story gets more interesting around EVT.
**CN:** 当把视角转向 EVT 时，Python 路径会更有意思。

**EN:** The file `python/cutlass_cppgen/epilogue/epilogue.py` exposes a `trace` frontend that traces a Python function and turns it into an epilogue visitor.
**CN:** `python/cutlass_cppgen/epilogue/epilogue.py` 提供了一个 `trace` 前端，可以追踪一个 Python 函数并把它转换成 epilogue visitor。

**EN:** The emitter backend under `python/cutlass_cppgen/backend/evt/backend/emitter_base.py` emits `XeEVT<...>` for Intel Xe12 and Xe20 targets.
**CN:** `python/cutlass_cppgen/backend/evt/backend/emitter_base.py` 中的发射器会在 Intel Xe12 和 Xe20 目标上生成 `XeEVT<...>`。

**EN:** This is the Python-level mirror of the C++ epilogue story: preserve the visitor abstraction, swap the backend implementation.
**CN:** 这正是 C++ epilogue 故事在 Python 层的镜像：visitor 抽象不变，但后端实现被替换成 Xe 版本。

**EN:** There is also an `epilogue_visitor` property path in `python/cutlass_cppgen/op/op.py`, which tells you that Python plans can carry visitor functors as first-class configuration.
**CN:** `python/cutlass_cppgen/op/op.py` 还提供了 `epilogue_visitor` 属性路径，这说明 Python plan 可以把 visitor functor 当作一等配置来携带。

**EN:** For engineers building higher-level runtimes, this is a big deal because it means fusion can be decided in Python while code generation still lands on Xe-specific C++ templates.
**CN:** 对构建高层运行时的工程师来说，这非常关键，因为它意味着融合逻辑可以在 Python 中决定，而代码生成仍能落在 Xe 特定的 C++ 模板上。

**EN:** The deprecated notebook examples still help conceptually, but the Xe-specific implementation path now lives in `cutlass_cppgen` rather than in old notebook-only flows.
**CN:** 旧版 notebook 示例在概念上仍然有帮助，但 Xe 特定实现路径现在主要位于 `cutlass_cppgen`，而不是旧的 notebook 专用流程中。

### 12.1 Python runtime example / Python 运行时示例

```python
workspace_size = c_size_t(0)
result = lib.sycl_tla_gemm_xe20_bf16(
    c_void_p(), c_void_p(), c_void_p(), c_void_p(),
    M, N, K, B,
    lda, ldb, ldc, ldd,
    0, 0, 0, 0,
    1,
    byref(workspace_size),
    c_void_p(),
    c_void_p(),
)
```

**EN:** This pattern is taken from `../examples/python/cutlass_library/xe20_gemm_bf16.py`.
**CN:** 这个模式来自 `../examples/python/cutlass_library/xe20_gemm_bf16.py`。

**EN:** The first call is not “run”; it is a workspace query. That distinction is easy to miss if you only looked at static C++ examples.
**CN:** 第一次调用并不是“执行”，而是查询 workspace。若你只看过静态 C++ 示例，很容易忽略这一点。

**EN:** The Python interface therefore mirrors the same separation of concerns as the C++ device adapter: capability check, workspace sizing, initialization, then execution.
**CN:** 因此，Python 接口与 C++ device adapter 一样遵循同样的关注点分离：能力检查、workspace 定尺、初始化，然后执行。

## 13. Build System with SYCL Support / 带 SYCL 支持的构建系统

**EN:** The official build documentation is `../media/docs/cpp/build/building_with_sycl_support.md`.
**CN:** 官方构建文档是 `../media/docs/cpp/build/building_with_sycl_support.md`。

**EN:** For Intel GPU support, the documented requirements are DPC++, Linux, and a recent Intel Compute Runtime and Graphics Compiler.
**CN:** 对于 Intel GPU 支持，文档要求使用 DPC++、Linux，以及较新的 Intel Compute Runtime 和 Graphics Compiler。

**EN:** The main CMake switch is `-DCUTLASS_ENABLE_SYCL=ON`.
**CN:** 主要的 CMake 开关是 `-DCUTLASS_ENABLE_SYCL=ON`。

**EN:** The architecture target is selected with `-DDPCPP_SYCL_TARGET=...`.
**CN:** 目标架构通过 `-DDPCPP_SYCL_TARGET=...` 选择。

**EN:** Typical Intel target strings include `intel_gpu_pvc`, `intel_gpu_bmg_g21`, and `intel_gpu_bmg_g31`.
**CN:** 典型的 Intel 目标字符串包括 `intel_gpu_pvc`、`intel_gpu_bmg_g21` 和 `intel_gpu_bmg_g31`。

**EN:** The README also notes that `bmg` can compile for both BMG variants together.
**CN:** README 还指出，使用 `bmg` 可以同时为两个 BMG 变体编译。

**EN:** The usual oneAPI compiler invocation is `CC=icx CXX=icpx cmake .. -G Ninja ...`.
**CN:** 常见的 oneAPI 编译器调用方式是 `CC=icx CXX=icpx cmake .. -G Ninja ...`。

**EN:** The build docs also mention nightly LLVM builds, where `CC=clang CXX=clang++` is used with the DPC++ distribution on the path.
**CN:** 构建文档也提到 nightly LLVM 方式，此时会在 DPC++ 环境路径下使用 `CC=clang CXX=clang++`。

**EN:** For example/test builds, oneMKL is used for random-number generation in the documented setup.
**CN:** 在文档所描述的构建环境中，示例和测试会使用 oneMKL 来做随机数生成。

**EN:** Ahead-of-time compilation matters on Xe, and the build docs list several environment variables to improve generated code quality.
**CN:** Ahead-of-time 编译在 Xe 上很重要，而构建文档也列出了若干用于提高生成代码质量的环境变量。

**EN:** Those variables include `SYCL_PROGRAM_COMPILE_OPTIONS="-ze-opt-large-register-file"` and `IGC_ExtraOCLOptions="-cl-intel-256-GRF-per-thread"`.
**CN:** 这些变量包括 `SYCL_PROGRAM_COMPILE_OPTIONS="-ze-opt-large-register-file"` 和 `IGC_ExtraOCLOptions="-cl-intel-256-GRF-per-thread"`。

**EN:** The important conceptual point is that AOT and large-register-file hints are part of the performance recipe, not merely deployment trivia.
**CN:** 概念上更重要的是：AOT 与 large-register-file 提示是性能配方的一部分，而不只是部署层细节。

**EN:** When you compare kernels across systems, mismatched AOT/JIT or GRF options can easily explain large performance gaps.
**CN:** 当你在不同系统间比较 kernel 时，AOT/JIT 或 GRF 选项不一致，往往就足以解释巨大的性能差距。

**EN:** The build docs also describe NVIDIA backend support through SYCL as validation-only, which reinforces the point that the Xe path is the production focus of these extensions.
**CN:** 构建文档还说明通过 SYCL 支持 NVIDIA 后端只是用于验证，这进一步说明 Xe 路径才是这些扩展真正的生产焦点。

**EN:** After configuration, individual examples are built as Ninja targets such as `00_bmg_gemm` or `06_xe_fmha_fwd`.
**CN:** 完成配置后，各个示例会以 Ninja target 的形式构建，例如 `00_bmg_gemm` 或 `06_xe_fmha_fwd`。

**EN:** For debugging alignment issues in Xe 2D copy, add `-DCUTE_ENABLE_XE_BLOCK_2D_ASSERT=1` to the CMake C++ flags in a debug build.
**CN:** 如果你想调试 Xe 2D copy 的对齐问题，可以在调试构建中把 `-DCUTE_ENABLE_XE_BLOCK_2D_ASSERT=1` 加入 CMake 的 C++ flags。

### 13.1 Build example / 构建示例

```bash
source /opt/intel/oneapi/setvars.sh
CC=icx CXX=icpx cmake .. -G Ninja   -DCUTLASS_ENABLE_SYCL=ON   -DDPCPP_SYCL_TARGET=intel_gpu_bmg_g21   -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

export SYCL_PROGRAM_COMPILE_OPTIONS="-ze-opt-large-register-file"
export IGC_VISAOptions="-perfmodel"
export IGC_VectorAliasBBThreshold=10000
export IGC_ExtraOCLOptions="-cl-intel-256-GRF-per-thread"

ninja 00_bmg_gemm
```

**EN:** This combines the documented CMake flow with the documented AOT/performance hint environment variables.
**CN:** 这段示例把文档中的 CMake 流程与 AOT/性能提示环境变量结合在一起。

**EN:** For reproducible performance work on Xe, keeping these settings explicit is a good habit.
**CN:** 在 Xe 上做可复现性能实验时，显式记录这些设置是一个很好的习惯。

## 14. Performance Guide / 性能指南

**EN:** The official performance guide is `../media/docs/cpp/cute/12_intel_performance_guide.md`.
**CN:** 官方性能指南是 `../media/docs/cpp/cute/12_intel_performance_guide.md`。

**EN:** It focuses on manual CuTe-level tuning for Xe, and that is exactly the right scope for serious kernel work.
**CN:** 它聚焦于 Xe 上的 CuTe 层手动调优，这正是严肃内核工作最需要的范围。

**EN:** The most important tuning knobs are tile size, pipeline stage count, memory alignment, reorder cost, and GRF pressure.
**CN:** 最重要的调优旋钮包括 tile size、pipeline stage 数、内存对齐、reorder 成本，以及 GRF 压力。

**EN:** For BF16/FP16 GEMM on BMG, the recurring recommended starting point is `Shape<_256,_256,_32>` with an 8x4 subgroup layout.
**CN:** 对于 BMG 上的 BF16/FP16 GEMM，反复被推荐的起始配置是 `Shape<_256,_256,_32>` 加上 8x4 subgroup 布局。

**EN:** For TF32 the guide suggests `Shape<_256,_256,_16>` as the analogous starting point.
**CN:** 对于 TF32，指南给出的对应起始配置是 `Shape<_256,_256,_16>`。

**EN:** Increasing M and N gives more compute per work-group, but can reduce occupancy and increase GRF pressure.
**CN:** 增大 M 和 N 会让每个 work-group 获得更多计算量，但也可能降低 occupancy 并增加 GRF 压力。

**EN:** Increasing K can amortize load overhead, but also enlarges copy fragments and can cause spills.
**CN:** 增大 K 可以摊薄 load 开销，但也会扩大 copy fragment，进而带来 spill 风险。

**EN:** Pipeline stages are a classic Xe tradeoff: more stages hide latency, fewer stages save registers.
**CN:** pipeline stage 是 Xe 上的经典权衡：更多 stage 更能隐藏延迟，更少 stage 则更节省寄存器。

**EN:** The guide explicitly recommends starting from 2 stages on BMG BF16 GEMM and trying 3 only if register spill stays under control.
**CN:** 指南明确建议在 BMG BF16 GEMM 上先从 2 个 stage 开始；只有在寄存器 spill 仍受控时再尝试 3 个 stage。

**EN:** Flash Attention decode is a good counterexample, because it often prefers only 1 stage.
**CN:** Flash Attention decode 就是一个很好的反例，因为它经常只需要 1 个 stage。

**EN:** A large part of Xe tuning is simply respecting 2D block message constraints and padding your problem accordingly.
**CN:** Xe 调优中很大一部分工作其实就是遵守 2D block message 的约束，并相应地为问题做 padding。

**EN:** If base pointer, pitch, or width alignment is broken, the fastest kernel design on paper is worthless.
**CN:** 如果基地址、pitch 或 width 对齐被破坏，那么纸面上再漂亮的最快 kernel 设计也毫无意义。

**EN:** The runtime-assert option for Xe block-2D copies is therefore extremely valuable during bring-up.
**CN:** 因此，在 bring-up 阶段启用 Xe block-2D copy 的运行时断言是非常有价值的。

**EN:** Another core principle is to watch GRF usage before celebrating a larger tile.
**CN:** 另一个核心原则是：在为更大 tile 欢呼之前，先看 GRF 使用情况。

**EN:** The guide repeatedly warns that Xe threads have a fixed register budget, and compiler spill to SLM can erase the expected gain from larger tiles or deeper pipelines.
**CN:** 指南反复警告：Xe 线程的寄存器预算是固定的，而编译器向 SLM spill 会抹掉大 tile 或深 pipeline 带来的预期收益。

**EN:** This is why `grf_size<256>` and AOT large-register-file options appear both in examples and in build guidance.
**CN:** 这也是为什么 `grf_size<256>` 与 AOT large-register-file 选项会同时出现在示例和构建指导中。

**EN:** Occupancy on Xe is also influenced by subgroup count per work-group. The guide notes diminishing returns beyond roughly 32 subgroups.
**CN:** Xe 上的 occupancy 还会受到每个 work-group 中 subgroup 数量的影响。指南指出超过大约 32 个 subgroup 后收益会递减。

**EN:** So if you are exploring alternative subgroup layouts, do not assume more subgroups is automatically better.
**CN:** 因此，当你尝试不同 subgroup 布局时，不要假设 subgroup 越多越好。

**EN:** A disciplined tuning workflow is: start from a known-good example, profile, change one knob at a time, verify alignment, and re-measure.
**CN:** 一个有纪律的调优流程应该是：从已知可用示例开始，做 profile，一次只改一个旋钮，验证对齐，然后重新测量。

**EN:** The guide explicitly recommends Intel PTI for GPU or Intel VTune for this profiling step.
**CN:** 指南明确推荐在这一阶段使用 Intel PTI for GPU 或 Intel VTune。

**EN:** The correct optimization target on Xe is not “more clever code”. It is “more useful DPAS work under the hardware message, register, and cache constraints”.
**CN:** 在 Xe 上，正确的优化目标不是“写出更聪明的代码”，而是“在硬件消息、寄存器和缓存约束下，让 DPAS 做到更多有效工作”。

**EN:** This is a good mental filter whenever you are tempted to add a new transformation step.
**CN:** 每当你想加入新的变换步骤时，这都是一个很好的心智过滤器。

**EN:** Ask whether the step increases DPAS utilization, removes a memory bottleneck, reduces spill, or improves message legality. If not, it is probably not worth the complexity.
**CN:** 你应该问自己：这个步骤是否提高了 DPAS 利用率、消除了内存瓶颈、减少了 spill，或者提升了消息合法性？如果都没有，它大概率不值得。

### 14.1 Performance starting recipe / 性能起步配方

**EN:** Start from `00_bmg_gemm` for BF16 or FP16 GEMM on BMG.
**CN:** 在 BMG 上做 BF16 或 FP16 GEMM 时，请先从 `00_bmg_gemm` 开始。

**EN:** Keep `TileShape = Shape<_256,_256,_32>` unless measurement tells you otherwise.
**CN:** 除非测量明确告诉你应该更改，否则请保持 `TileShape = Shape<_256,_256,_32>`。

**EN:** Use `sub_group_size<16>` and `grf_size<256>` consistently across experiments.
**CN:** 在所有实验中都一致使用 `sub_group_size<16>` 与 `grf_size<256>`。

**EN:** Start with 2 pipeline stages for BMG GEMM, then try 3 only if spill remains acceptable.
**CN:** BMG GEMM 先从 2 个 pipeline stage 开始，只有在 spill 仍可接受时再尝试 3。

**EN:** Enable Xe block-2D assertions in debug builds before trusting any performance number.
**CN:** 在信任任何性能数字之前，先在调试构建中打开 Xe block-2D 断言。

**EN:** Measure before and after every scheduler, tile, or copy-atom change.
**CN:** 每次更改 scheduler、tile 或 copy atom 前后都要测量。

## 15. Porting Checklist from CUTLASS/CUDA to SYCL-TLA / 从 CUTLASS/CUDA 迁移到 SYCL-TLA 的清单

**EN:** Keep the algorithmic decomposition first: problem tiling, accumulator math, fusion intent, and pipeline structure should be preserved conceptually.
**CN:** 首先保留算法分解：问题切分、accumulator 数学、fusion 意图和 pipeline 结构在概念上都应该尽量保留。

**EN:** Replace warp assumptions with subgroup assumptions everywhere.
**CN:** 把所有 warp 假设都替换成 subgroup 假设。

**EN:** Assume subgroup width 16 until proven otherwise; do not port any 32-wide warp logic blindly.
**CN:** 在没有明确证据前，都应假设 subgroup 宽度是 16；不要盲目移植任何 32 宽 warp 逻辑。

**EN:** Replace Tensor Core atom names with `XE_DPAS_TT` or let the Xe builders infer them.
**CN:** 把 Tensor Core atom 名称替换成 `XE_DPAS_TT`，或者交给 Xe builder 自动推导。

**EN:** Replace cp.async/TMA assumptions with Xe block-2D copy assumptions.
**CN:** 把 cp.async/TMA 的思维替换成 Xe block-2D copy 的思维。

**EN:** Do not assume SLM is mandatory. First test whether direct L1/GRF streaming is already optimal.
**CN:** 不要假设 SLM 是必需的。应先测试直接的 L1/GRF 流式路径是否已经最优。

**EN:** Make VNNI layout explicit in your reasoning, especially for B operands, quantized operands, and any dequantization path.
**CN:** 在推理时必须显式考虑 VNNI 布局，尤其是 B 操作数、量化操作数以及任何反量化路径。

**EN:** Use helper constructors like `make_block_2d_copy_A/B` before hand-rolling Xe copy traits.
**CN:** 在自己手写 Xe copy trait 之前，先尝试使用 `make_block_2d_copy_A/B` 这样的 helper constructor。

**EN:** Treat `reorder` as part of the design, not as a temporary workaround.
**CN:** 把 `reorder` 当作设计的一部分，而不是临时权宜之计。

**EN:** Use CollectiveBuilder if the policy layer can still express your kernel, and only drop down to low-level CuTe when it cannot.
**CN:** 如果 policy 层还能表达你的 kernel，请优先使用 CollectiveBuilder；只有在它表达不了时才下沉到低层 CuTe。

**EN:** Carry `KernelHardwareInfo` correctly, because Xe schedulers may need the EU/SM count to shape the persistent launch.
**CN:** 要正确传递 `KernelHardwareInfo`，因为 Xe scheduler 可能需要 EU/SM 数量来决定 persistent launch 的形状。

**EN:** Validate alignment early using `CUTE_ENABLE_XE_BLOCK_2D_ASSERT`.
**CN:** 尽早用 `CUTE_ENABLE_XE_BLOCK_2D_ASSERT` 验证对齐。

**EN:** Use builder defaults first for stage count and schedule, then specialize only after profiling.
**CN:** 先采用 builder 的默认 stage count 和 schedule，profile 之后再做专门化。

**EN:** Expect grouped and pointer-array kernels to use different kernel/epilogue machinery than single-problem kernels.
**CN:** 要预期 grouped 和 pointer-array kernel 会使用与单问题 kernel 不同的 kernel/epilogue 机制。

**EN:** When porting fused epilogues, preserve the visitor-tree logic but retarget the copy/store implementation to Xe.
**CN:** 在迁移融合 epilogue 时，应保留 visitor-tree 逻辑，但把 copy/store 实现重新定向到 Xe。

**EN:** When porting quantized kernels, trace the storage type, compute type, reorder step, and final epilogue type as a single pipeline.
**CN:** 在迁移量化 kernel 时，要把存储类型、计算类型、reorder 步骤和最终 epilogue 类型当作一条完整 pipeline 来跟踪。

**EN:** When porting Flash Attention, retune tile shape, stage count, and scheduler for prefill and decode separately.
**CN:** 在迁移 Flash Attention 时，要分别为 prefill 和 decode 重新调优 tile shape、stage count 和 scheduler。

**EN:** Finally, verify correctness after each structural change. Xe performance tuning is not meaningful if the copy constraints are already broken.
**CN:** 最后，每做一次结构性修改后都要验证正确性。如果 copy 约束已经被破坏，Xe 性能调优就没有意义。

## 16. Example Map / 示例地图

**EN:** `../examples/cute/tutorial/xe_gemm.cpp`: the best low-level reference for prefetch, block-2D copy, reorder, and DPAS in one place.
**CN:** `../examples/cute/tutorial/xe_gemm.cpp`：低层参考首选，能在一个地方看到 prefetch、block-2D copy、reorder 与 DPAS。

**EN:** `../examples/cute/tutorial/xe_gemm_slm.cpp`: shows how to build an SLM-backed pipeline with cooperative copies and explicit barriers.
**CN:** `../examples/cute/tutorial/xe_gemm_slm.cpp`：展示如何用 cooperative copy 和显式 barrier 构建基于 SLM 的 pipeline。

**EN:** `../examples/cute/tutorial/xe_gemm_subgroup_specialization_slm.cpp`: shows subgroup specialization with producer/consumer roles.
**CN:** `../examples/cute/tutorial/xe_gemm_subgroup_specialization_slm.cpp`：展示带 producer/consumer 角色划分的 subgroup specialization。

**EN:** `../examples/00_bmg_gemm/00_bmg_gemm.cpp`: standard BMG GEMM with explicit Xe dispatch policy and default linear-combination epilogue.
**CN:** `../examples/00_bmg_gemm/00_bmg_gemm.cpp`：标准 BMG GEMM，包含显式 Xe dispatch policy 和默认线性组合 epilogue。

**EN:** `../examples/01_bmg_gemm_with_collective_builder/01_bmg_gemm_with_collective_builder.cpp`: shortest modern builder-driven path, including fused ReLU.
**CN:** `../examples/01_bmg_gemm_with_collective_builder/01_bmg_gemm_with_collective_builder.cpp`：最简洁的现代 builder 路径，还包含融合 ReLU。

**EN:** `../examples/02_bmg_gemm_mixed_dtype/*`: mixed-input, quantized, zero-point, and dequantization examples.
**CN:** `../examples/02_bmg_gemm_mixed_dtype/*`：混合输入、量化、zero-point 与反量化示例集合。

**EN:** `../examples/03_bmg_gemm_streamk/03_bmg_gemm_streamk.cpp`: practical Stream-K example.
**CN:** `../examples/03_bmg_gemm_streamk/03_bmg_gemm_streamk.cpp`：实用的 Stream-K 示例。

**EN:** `../examples/04_bmg_grouped_gemm/04_bmg_grouped_gemm.cpp`: grouped GEMM reference.
**CN:** `../examples/04_bmg_grouped_gemm/04_bmg_grouped_gemm.cpp`：grouped GEMM 参考实现。

**EN:** `../examples/05_bmg_gemm_with_epilogues/*`: epilogue fusion examples including GELU, ReLU, SiLU, softmax, bias, and split-K related flows.
**CN:** `../examples/05_bmg_gemm_with_epilogues/*`：epilogue 融合示例，包括 GELU、ReLU、SiLU、softmax、bias 和 split-K 相关流程。

**EN:** `../examples/06_bmg_flash_attention/06_xe_fmha_fwd.cpp`: Flash Attention V2 forward path with prefill/decode/persistent variants.
**CN:** `../examples/06_bmg_flash_attention/06_xe_fmha_fwd.cpp`：Flash Attention V2 前向路径，包含 prefill/decode/persistent 变体。

**EN:** `../examples/07_bmg_dual_gemm/07_bmg_dual_gemm.cpp`: dual-GEMM fusion pattern.
**CN:** `../examples/07_bmg_dual_gemm/07_bmg_dual_gemm.cpp`：dual-GEMM 融合模式。

**EN:** `../examples/08_bmg_gemm_f8/08_bmg_gemm_f8.cpp`: FP8 input path with upconversion to FP16 MMA.
**CN:** `../examples/08_bmg_gemm_f8/08_bmg_gemm_f8.cpp`：FP8 输入路径，并在内部上转换到 FP16 MMA。

**EN:** `../examples/09_bmg_grouped_gemm_f8/09_bmg_grouped_gemm_f8.cpp`: grouped FP8 GEMM.
**CN:** `../examples/09_bmg_grouped_gemm_f8/09_bmg_grouped_gemm_f8.cpp`：grouped FP8 GEMM。

**EN:** `../examples/10_bmg_grouped_gemm_mixed_dtype/*`: grouped mixed-dtype GEMM examples.
**CN:** `../examples/10_bmg_grouped_gemm_mixed_dtype/*`：grouped mixed-dtype GEMM 示例。

**EN:** `../examples/11_xe20_cutlass_library/*`: generated library path for Python/runtime integration.
**CN:** `../examples/11_xe20_cutlass_library/*`：面向 Python/运行时集成的生成库路径。

**EN:** `../examples/12_xe20_moe_gemm_cute_interface/*`: MoE GEMM with CuTe interface, useful when studying gather-heavy or grouped scenarios.
**CN:** `../examples/12_xe20_moe_gemm_cute_interface/*`：基于 CuTe 接口的 MoE GEMM，适合研究 gather 密集或 grouped 场景。

**EN:** `../examples/python/cutlass_library/xe20_gemm_bf16.py`: Python ctypes call path for a generated Xe20 GEMM shared library.
**CN:** `../examples/python/cutlass_library/xe20_gemm_bf16.py`：面向生成式 Xe20 GEMM 共享库的 Python ctypes 调用路径。

## 17. Common Pitfalls / 常见陷阱

**EN:** Pitfall: forgetting `sub_group_size<16>` and then debugging wrong answers as if they were arithmetic bugs.
**CN:** 陷阱：忘记设置 `sub_group_size<16>`，然后把错误结果当成算术 bug 去排查。

**EN:** Pitfall: assuming SLM is always required because a CUDA port used shared memory.
**CN:** 陷阱：因为 CUDA 版本用了共享内存，就想当然地认为 SLM 一定需要。

**EN:** Pitfall: using a legal MMA atom with an illegal copy path because the block-2D alignment rules were ignored.
**CN:** 陷阱：MMA atom 本身合法，但 copy 路径因为忽略 block-2D 对齐规则而非法。

**EN:** Pitfall: ignoring VNNI layout and paying hidden reorder/interleave costs that the compiler inserts later.
**CN:** 陷阱：忽略 VNNI 布局，最终为编译器插入的隐藏 reorder/interleave 付出代价。

**EN:** Pitfall: increasing tile size without measuring spill or occupancy.
**CN:** 陷阱：只增大 tile size，却不测 spill 和 occupancy。

**EN:** Pitfall: increasing pipeline stages without checking whether latency hiding improved more than register pressure worsened.
**CN:** 陷阱：增加 pipeline stage，却不检查它带来的 latency hiding 是否真的大于寄存器压力恶化。

**EN:** Pitfall: treating `reorder` as a sign of failure instead of a legitimate Xe primitive.
**CN:** 陷阱：把 `reorder` 视为失败信号，而不是把它当成合法的 Xe 原语。

**EN:** Pitfall: mixing grouped and single-problem assumptions in kernel arguments or epilogue argument types.
**CN:** 陷阱：在 kernel 参数或 epilogue 参数类型中混用 grouped 与单问题的假设。

**EN:** Pitfall: benchmarking with different AOT/GRF settings across runs and then comparing the numbers as if they were equivalent.
**CN:** 陷阱：不同实验使用了不同的 AOT/GRF 设置，却把结果当成等价数据来比较。

**EN:** Pitfall: reading only example code and not the companion/performance docs, which explain why the example was written that way.
**CN:** 陷阱：只读示例代码而不读 companion/performance 文档，结果不知道示例为什么要那样写。

**EN:** Pitfall: verifying only performance and not correctness after changing copy atoms, stage counts, or quantization metadata layout.
**CN:** 陷阱：修改 copy atom、stage count 或量化元数据布局后只看性能、不做正确性验证。

**EN:** Pitfall: assuming that because a path works on one Xe generation it automatically preserves the same optimal tile and stage choices on another.
**CN:** 陷阱：认为某条路径在一个 Xe 代际上有效，就会在另一个代际上自动保持同样最优的 tile 和 stage 选择。

## 18. CUTLASS-to-Xe Glossary / CUTLASS 到 Xe 术语表

**EN:** Warp -> `sub_group` (16 lanes on Xe).
**CN:** Warp -> `sub_group`（Xe 上固定 16 lane）。

**EN:** CTA / threadblock -> work-group.
**CN:** CTA / threadblock -> work-group。

**EN:** Grid -> `nd_range` launch domain.
**CN:** Grid -> `nd_range` 启动域。

**EN:** Thread -> work-item.
**CN:** Thread -> work-item。

**EN:** Shared memory -> SLM.
**CN:** Shared memory -> SLM。

**EN:** Tensor Core MMA -> DPAS / XMX.
**CN:** Tensor Core MMA -> DPAS / XMX。

**EN:** GMMA/HMMA atom name -> `XE_DPAS_TT`.
**CN:** GMMA/HMMA atom 名称 -> `XE_DPAS_TT`。

**EN:** cp.async / TMA -> `XE_LOAD_2D*` and `XE_PREFETCH_2D`.
**CN:** cp.async / TMA -> `XE_LOAD_2D*` 与 `XE_PREFETCH_2D`。

**EN:** Regular global store path -> `XE_STORE_2D` when legal.
**CN:** 常规全局写回路径 -> 合法时使用 `XE_STORE_2D`。

**EN:** Per-thread fragment intuition -> subgroup-owned `SubgroupTensor` intuition.
**CN:** “每线程 fragment”直觉 -> “subgroup 共同拥有的 `SubgroupTensor`”直觉。

**EN:** Tensor Core B layout trick -> explicit VNNI layout reasoning.
**CN:** Tensor Core 的 B 布局技巧 -> 显式的 VNNI 布局推理。

**EN:** Implicit fragment reshaping -> explicit `reorder`.
**CN:** 隐式 fragment 重排 -> 显式 `reorder`。

**EN:** CUDA launch stream -> `sycl::queue`.
**CN:** CUDA launch stream -> `sycl::queue`。

**EN:** <<<grid, block>>> -> `parallel_for(nd_range, properties, lambda)`.
**CN:** `<<<grid, block>>>` -> `parallel_for(nd_range, properties, lambda)`。

**EN:** SM count query -> `KernelHardwareInfo::query_device_multiprocessor_count`.
**CN:** SM 数查询 -> `KernelHardwareInfo::query_device_multiprocessor_count`。

**EN:** Warpgroup scheduling intuition -> subgroup/work-group scheduling intuition.
**CN:** Warpgroup 调度直觉 -> subgroup/work-group 调度直觉。

**EN:** TMA legality checks -> Xe block-2D legality checks.
**CN:** TMA 合法性检查 -> Xe block-2D 合法性检查。

**EN:** Epilogue visitor tree -> EVT on Xe plus Xe callbacks.
**CN:** epilogue visitor tree -> Xe 上的 EVT 加 Xe callback。

**EN:** Fused auxiliary store -> `XeAuxStore` in visitor space.
**CN:** 融合辅助写回 -> visitor 空间中的 `XeAuxStore`。

**EN:** Single-problem GEMM -> `xe_gemm` / `xe_gemm_cooperative` family.
**CN:** 单问题 GEMM -> `xe_gemm` / `xe_gemm_cooperative` 家族。

**EN:** Grouped GEMM -> pointer-array cooperative kernel family.
**CN:** grouped GEMM -> pointer-array cooperative kernel 家族。

**EN:** Standard epilogue -> `xe_epilogue`.
**CN:** 标准 epilogue -> `xe_epilogue`。

**EN:** Grouped/pointer-array epilogue -> `xe_array_epilogue`.
**CN:** grouped/pointer-array epilogue -> `xe_array_epilogue`。

**EN:** Automatic mainloop selection -> `CollectiveBuilder<arch::IntelXe,...>`.
**CN:** 自动 mainloop 选择 -> `CollectiveBuilder<arch::IntelXe,...>`。

**EN:** Automatic epilogue selection -> Xe epilogue builder specialization.
**CN:** 自动 epilogue 选择 -> Xe epilogue builder 特化。

**EN:** BF16 baseline tile -> `Shape<_256,_256,_32>`.
**CN:** BF16 基线 tile -> `Shape<_256,_256,_32>`。

**EN:** Common subgroup layout -> 8x4, N-major.
**CN:** 常见 subgroup 布局 -> 8x4，N 方向优先。

**EN:** Pipeline depth -> prefetch distance.
**CN:** pipeline 深度 -> prefetch 距离。

**EN:** Register pressure -> GRF pressure.
**CN:** 寄存器压力 -> GRF 压力。

**EN:** Large register mode hint -> `grf_size<256>` plus AOT options.
**CN:** 大寄存器模式提示 -> `grf_size<256>` 加 AOT 选项。

**EN:** Transpose load limitation -> often requires 32/64-bit path.
**CN:** transpose load 限制 -> 往往要求 32/64 位路径。

**EN:** Quantized B friendliness -> often via VNNI load plus reorder/dequant.
**CN:** 量化 B 友好路径 -> 往往通过 VNNI load 加 reorder/反量化。

**EN:** FP8 support -> often upconvert before MMA.
**CN:** FP8 支持 -> 常常先上转换再进入 MMA。

**EN:** Decode attention -> usually shallow pipeline.
**CN:** decode attention -> 通常更浅的 pipeline。

**EN:** Prefill attention -> usually larger tiles and deeper pipeline.
**CN:** prefill attention -> 通常更大 tile 和更深 pipeline。

**EN:** Persistent scheduling -> keep work resident longer.
**CN:** persistent scheduling -> 让工作更长时间驻留。

**EN:** SLM path -> use only when sharing/preprocessing/alignment justify it.
**CN:** SLM 路径 -> 只有在共享/预处理/对齐限制证明它值得时才使用。

**EN:** Global -> GRF path -> often the default fast path.
**CN:** Global -> GRF 路径 -> 常常是默认快路径。

**EN:** `make_block_2d_copy_A/B/C/D` -> preferred copy helpers.
**CN:** `make_block_2d_copy_A/B/C/D` -> 推荐的 copy helper。

**EN:** `make_block_2d_prefetch` -> preferred prefetch helper.
**CN:** `make_block_2d_prefetch` -> 推荐的 prefetch helper。

**EN:** `make_coop_block_2d_copy_A/B` -> cooperative load helpers for SLM pipelines.
**CN:** `make_coop_block_2d_copy_A/B` -> 面向 SLM pipeline 的 cooperative load helper。

**EN:** `make_A_slm_layout` / `make_B_slm_layout` -> SLM layout helpers.
**CN:** `make_A_slm_layout` / `make_B_slm_layout` -> SLM 布局 helper。

**EN:** `make_A_slm_copies` / `make_B_slm_copies` -> SLM copy helpers.
**CN:** `make_A_slm_copies` / `make_B_slm_copies` -> SLM copy helper。

**EN:** `FusionCallbacks` -> policy-specific epilogue adapter.
**CN:** `FusionCallbacks` -> 面向特定 policy 的 epilogue 适配层。

**EN:** Identity epilogue fast path -> alpha 1, beta 0.
**CN:** identity epilogue 快路径 -> alpha 为 1、beta 为 0。

**EN:** Manifest generation -> build-time filtering plus runtime registration.
**CN:** Manifest 生成 -> 构建期过滤加运行期注册。

**EN:** Python `trace` for EVT -> create visitor from Python function.
**CN:** Python `trace` 用于 EVT -> 从 Python 函数生成 visitor。

**EN:** `XeEVT` emitter -> Python backend emits Xe-specific epilogue visitor types.
**CN:** `XeEVT` emitter -> Python 后端发射 Xe 特定的 epilogue visitor 类型。

**EN:** PVC target string -> `intel_gpu_pvc`.
**CN:** PVC 目标字符串 -> `intel_gpu_pvc`。

**EN:** BMG target strings -> `intel_gpu_bmg_g21` and `intel_gpu_bmg_g31`.
**CN:** BMG 目标字符串 -> `intel_gpu_bmg_g21` 与 `intel_gpu_bmg_g31`。

**EN:** Xe block-2D debug option -> `CUTE_ENABLE_XE_BLOCK_2D_ASSERT=1`.
**CN:** Xe block-2D 调试选项 -> `CUTE_ENABLE_XE_BLOCK_2D_ASSERT=1`。

**EN:** AOT large register hint -> `-ze-opt-large-register-file`.
**CN:** AOT 大寄存器提示 -> `-ze-opt-large-register-file`。

**EN:** GRF compile hint -> `-cl-intel-256-GRF-per-thread`.
**CN:** GRF 编译提示 -> `-cl-intel-256-GRF-per-thread`。

**EN:** Primary tuning tools -> Intel PTI for GPU and Intel VTune.
**CN:** 主要调优工具 -> Intel PTI for GPU 与 Intel VTune。

**EN:** Reference BF16 GEMM example -> `00_bmg_gemm`.
**CN:** 参考 BF16 GEMM 示例 -> `00_bmg_gemm`。

**EN:** Reference builder example -> `01_bmg_gemm_with_collective_builder`.
**CN:** 参考 builder 示例 -> `01_bmg_gemm_with_collective_builder`。

**EN:** Reference epilogue fusion example -> `05_bmg_gemm_with_epilogues`.
**CN:** 参考 epilogue 融合示例 -> `05_bmg_gemm_with_epilogues`。

**EN:** Reference FMHA example -> `06_xe_fmha_fwd`.
**CN:** 参考 FMHA 示例 -> `06_xe_fmha_fwd`。

**EN:** Reference FP8 example -> `08_bmg_gemm_f8`.
**CN:** 参考 FP8 示例 -> `08_bmg_gemm_f8`。

**EN:** Reference Python generated-library example -> `xe20_gemm_bf16.py`.
**CN:** 参考 Python 生成库示例 -> `xe20_gemm_bf16.py`。

## 19. Final Takeaways / 最后总结

**EN:** SYCL-TLA is most productive when you keep CUTLASS decomposition and replace only the hardware contract.
**CN:** 当你保留 CUTLASS 的分解方式、只替换硬件契约时，SYCL-TLA 的生产力最高。

**EN:** On Intel Xe, that contract is written in subgroup width 16, DPAS, block-2D copies, reorder, Xe schedulers, and Xe epilogues.
**CN:** 在 Intel Xe 上，这套契约主要写在 16 宽 subgroup、DPAS、block-2D copy、reorder、Xe scheduler 与 Xe epilogue 之中。

**EN:** Most optimization questions can be reduced to three checks: is the copy legal, is the layout DPAS-friendly, and is GRF pressure acceptable.
**CN:** 大多数优化问题都可以归结为三个检查项：copy 是否合法、布局是否对 DPAS 友好、GRF 压力是否可接受。

**EN:** If you use the examples and builders as scaffolding, SYCL-TLA extensions are much easier to learn than they first appear.
**CN:** 如果你把示例和 builder 当作脚手架来使用，SYCL-TLA 扩展实际上会比初看时容易掌握得多。

**EN:** The right way to grow from user to maintainer is to move in layers: example -> builder -> low-level CuTe -> custom scheduler or SLM pipeline.
**CN:** 从使用者成长为维护者的正确路径是分层推进：example -> builder -> 低层 CuTe -> 自定义 scheduler 或 SLM pipeline。

**EN:** That layered progression mirrors the design of SYCL-TLA itself.
**CN:** 这种分层成长路径，也正好映射了 SYCL-TLA 自身的设计方式。
