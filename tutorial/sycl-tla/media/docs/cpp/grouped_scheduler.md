# grouped_scheduler.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/sycl-tla/media/docs/cpp/grouped_scheduler.md`
- **Purpose:** **EN:** Explains how CUTLASS grouped kernels schedule tiles across multiple problems and what optimizations improve balance and scheduling cost. **CN:** 说明 CUTLASS 的 grouped kernel 如何在多个问题之间调度 tile，以及哪些优化可以改善负载均衡和调度开销。

## Content Analysis / 内容分析
### Introduction to Grouped Kernel Schedulers
**EN:** The introduction defines grouped kernels as persistent kernels that launch fewer blocks than total tiles and rely on a scheduler (`ProblemVisitor`) to hand out work dynamically. This establishes scheduling itself as a first-class performance concern.
**CN:** 引言把 grouped kernel 定义为一种持久化内核：启动的 block 数少于总 tile 数，再由调度器（`ProblemVisitor`）动态发放工作。这使“调度本身”成为一个一等性能问题。

### Grouped GEMM Scheduler
**EN:** The grouped GEMM scheduler uses a round-robin tile assignment across all problems. The document illustrates that the strategy is simple and general, but it depends on each block being able to cheaply determine which problem and tile correspond to its running index.
**CN:** grouped GEMM scheduler 采用跨问题的 round-robin tile 分配方式。文档通过图示说明这种策略简单且通用，但前提是每个 block 都能低成本地判断当前索引对应的是哪个问题、哪个 tile。

### Computing the schedule for a given block
**EN:** This section explains the concrete lookup logic: maintain a thread-local running tile index, search for the owning problem, then rasterize the intra-problem tile coordinate. It is the bridge from abstract scheduling policy to executable per-block logic.
**CN:** 这一节讲解具体查找逻辑：维护线程块自己的运行 tile 索引，搜索其所属问题，再把问题内的一维 tile 编号栅格化成坐标。它把抽象调度策略落成了可执行的 per-block 逻辑。

### Grouped Rank2K Scheduler
**EN:** The document then shows why a generic grouped GEMM scheduler wastes work for triangular outputs such as SYR2K/HER2K. Because many nominal tiles are inactive, a naive round-robin policy causes severe load imbalance.
**CN:** 接着文档说明：对于 SYR2K/HER2K 这类三角输出，通用 grouped GEMM scheduler 会浪费大量工作。由于很多名义上的 tile 实际无效，朴素的 round-robin 策略会带来严重的负载不均。

### Specializing the scheduler for triangular problems
**EN:** To fix that, the scheduler maps threadblock IDs directly into active triangular coordinates. The derivation of row and column formulas, plus handling for non-square grids and upper/lower variants, turns the section into a compact mathematical scheduler design note.
**CN:** 为了解决这个问题，调度器改为把线程块 ID 直接映射到有效的三角区域坐标。文中对行列公式的推导，以及对非方形网格和上下三角情况的处理，使这一节成为一份紧凑的数学化调度设计说明。

### Scheduler modes
**EN:** The scheduler-mode section compares two implementation strategies: pure device-side scheduling and host-precomputed schedules. The analysis is practical because it frames the trade-off in terms of device-host communication, overlap opportunities, and scheduling overhead.
**CN:** scheduler mode 一节比较了两种实现策略：纯设备侧调度与主机预计算调度。它的价值在于把权衡讲得很落地——关注主机/设备通信、可否重叠执行，以及调度逻辑本身的开销。

### `GroupScheduleMode::kDeviceOnly` (default)
**EN:** The default mode parallelizes problem search across warp lanes and uses prefix sums to identify which problem owns the current tile index. It favors fully device-resident workflows where parameters are produced on the GPU.
**CN:** 默认模式会在 warp 各 lane 之间并行搜索问题，并利用前缀和找出当前 tile 索引所属的问题。它更适合参数本来就由 GPU 侧生成的全设备驻留流程。

### Precomputing schedules on the host: `GroupScheduleMode::kHostPrecompute`
**EN:** Host precompute moves the search work off the GPU by materializing `(problem_idx, problem_starting_tile)` tuples per block. The document is careful to note that this representation is best when each block touches only a small number of tiles per problem.
**CN:** host precompute 通过为每个 block 预先生成 `(problem_idx, problem_starting_tile)` 元组，把搜索工作挪到 GPU 之外。文中也明确提醒：这种表示方式最适合“每个 block 在每个问题中只处理少量 tile”的场景。

### Which scheduler mode should I use?
**EN:** The decision guide turns the previous technical explanation into deployment advice: choose based on where inputs originate, whether host work can overlap with prior kernels, and how compute-light the grouped problems are.
**CN:** 这一决策指南把前面的技术解释转化成部署建议：重点看输入参数来自哪里、主机工作能否与前序 kernel 重叠，以及 grouped 问题本身是否计算密度较低。

### Improving Load Balance by Sorting Problems
**EN:** The final major section observes that equal tile counts do not imply equal work when K differs across problems. Sorting by descending K can reduce long-tail imbalance, though the document wisely avoids calling it universally beneficial.
**CN:** 最后一大节指出：即使每个问题的 tile 数相同，只要 K 不同，工作量也可能完全不同。按 K 降序排序能够减少长尾不平衡，但文档也很谨慎，没有把它说成放之四海而皆准的优化。

## Key Concepts / 关键概念
- **`ProblemVisitor`** — **EN:** Scheduler object that tells a persistent block which tile to compute next. **CN:** 告诉持久化 block 下一步该算哪个 tile 的调度对象。
- **Round-robin assignment** — **EN:** Default policy that cycles tiles across blocks and problems. **CN:** 在 block 与问题之间循环分配 tile 的默认策略。
- **Triangular mapping** — **EN:** Closed-form mapping from block IDs to active tiles for Rank2K-style outputs. **CN:** 将 block ID 闭式映射到 Rank2K 三角有效 tile 的方法。
- **Host precompute** — **EN:** A scheduling mode that shifts lookup work from GPU to CPU. **CN:** 把调度查找工作从 GPU 挪到 CPU 的一种模式。
- **Problem sorting** — **EN:** Reordering problems, often by K, to reduce grouped-kernel imbalance. **CN:** 通常按 K 重排问题，以降低 grouped kernel 的负载不均。

## Related Files / 相关文件
- `examples/24_gemm_grouped/gemm_grouped.cu` — **EN:** Example showing grouped GEMM setup and sorting. **CN:** 展示 grouped GEMM 配置与排序的示例。
- `include/cutlass/gemm/kernel/grouped_problem_visitor.h` — **EN:** Header that defines scheduler modes and visitor logic. **CN:** 定义调度模式和 visitor 逻辑的头文件。
- `media/docs/cpp/efficient_gemm.md` — **EN:** Background on persistent kernels and load-balancing trade-offs. **CN:** 提供持久化内核与负载均衡权衡的背景说明。

