# heuristics — Documentation Analysis / 文档分析

## Source / 来源
- Source: `/root/xw/sycl-tla/media/docs/cpp/heuristics.md`
- Purpose: Describes CUTLASS GEMM heuristics for narrowing autotuning search space with NVIDIA's analytical matmul heuristic package. / 说明 CUTLASS 的 GEMM 启发式流程，借助 NVIDIA 的解析式 matmul heuristic 缩小自动调优搜索空间。

## Content Analysis / 内容分析
### GEMM Heuristics
**EN:** The title section defines the document's role clearly: it is about reducing autotuning cost, not replacing profiling entirely. The heuristic ranks candidate kernels analytically so the build and measurement loops can focus on a smaller set.
**CN:** 标题部分已经明确本文角色：它的目标是降低自动调优成本，而不是彻底替代 profiling。启发式方法会先做解析式排序，让构建与测量流程只关注较小的候选集合。

### Overview
**EN:** The overview connects CUTLASS heuristics to `nvidia-matmul-heuristics`, emphasizing estimated performance based on problem size and hardware SKU. This frames the system as hardware-aware guidance for kernel selection.
**CN:** “Overview” 把 CUTLASS heuristics 与 `nvidia-matmul-heuristics` 联系起来，强调它会结合问题规模和硬件 SKU 估计性能，本质上是面向硬件的内核选择指导。

### Coverage
**EN:** Coverage is intentionally narrow and marked experimental: only dense GEMM for selected floating-point formats on Hopper and Blackwell is supported. The limitation matters because users should treat the feature as a filter, not a universal oracle.
**CN:** 支持范围刻意保持较窄且明确标注为实验特性：目前只覆盖 Hopper 与 Blackwell 上若干浮点格式的稠密 GEMM。这个限制很关键，因为用户应把它视为筛选器，而不是万能决策器。

### Usage / Quick Start
**EN:** The quick-start path is practical and linear: install the dependency, prepare JSON problem descriptions, configure CMake with heuristics options, and profile the emitted CSV test list. The document treats build-time generation and run-time profiling as two halves of the same workflow.
**CN:** 快速开始部分采用非常实用的线性流程：安装依赖、准备 JSON 问题描述、带启发式选项配置 CMake，然后对生成的 CSV 测试列表执行 profiling。它把构建期生成与运行期测量视为同一工作流的前后两半。

### Install Dependencies / Prepare Input File / Build / Profile
**EN:** The substeps reveal the contract expected by the heuristics system: structured problem metadata in JSON in exchange for a bounded set of kernel configurations and a profiler-consumable test list.
**CN:** 这些子步骤揭示了启发式系统所要求的“输入契约”：用户需要提供结构化的 JSON 问题元数据，系统则返回受限规模的内核配置集合和可供 profiler 消费的测试列表。

### Direct Usage in Python
**EN:** The final section highlights that the heuristics logic is reusable outside the canned CMake flow. Advanced users can call Python APIs directly to integrate custom emitters or prebuilt kernels.
**CN:** 最后一节强调该启发式逻辑并不局限于默认 CMake 流程。高级用户可以直接调用 Python API，把它接入自定义 emitter 或预构建内核流程中。

## Key Concepts / 关键概念
- Analytical kernel ranking / 解析式内核排序
- Search-space reduction for autotuning / 自动调优搜索空间缩减
- Problem metadata in JSON / JSON 问题元数据
- Build-time testlist generation / 构建期测试列表生成
- Python-level integration / Python 层集成方式

## Related Files / 相关文件
- `python/cutlass_library/heuristics.py` — direct Python heuristic entry points / Python 启发式入口
- `python/cutlass_library/sm90_shapes.py` — Hopper shape definitions / Hopper 形状定义
- `python/cutlass_library/sm100_shapes.py` — Blackwell shape definitions / Blackwell 形状定义
- `media/docs/cpp/profiler.md` — downstream profiling workflow / 下游 profiling 工作流文档
