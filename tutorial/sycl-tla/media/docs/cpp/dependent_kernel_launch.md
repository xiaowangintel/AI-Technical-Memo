# dependent_kernel_launch — Documentation Analysis / 文档分析

## Source / 来源
- Source: `/root/xw/sycl-tla/media/docs/cpp/dependent_kernel_launch.md`
- Purpose: Explains programmatic dependent launch (PDL) support in CUTLASS for safely overlapping kernels in the same CUDA stream. / 说明 CUTLASS 对 programmatic dependent launch（PDL）的支持，用于在同一 CUDA stream 中安全重叠多个内核。

## Content Analysis / 内容分析
### Dependent kernel launches
**EN:** The introduction describes PDL as a Hopper/Blackwell feature that lets consecutive kernels overlap while still preserving correctness around global-memory flushes. The central idea is controlled overlap, not unconstrained concurrency.
**CN:** 引言把 PDL 描述为 Hopper/Blackwell 上的一项能力：它允许相邻内核在保证全局内存刷写正确性的前提下发生重叠。其核心是“受控重叠”，而不是无约束并发。

### Using dependent launch in CUTLASS
**EN:** This section separates build-time enablement from runtime usage. First, macros add the necessary instructions to generated kernels; then the launch path must explicitly opt into PDL, which clarifies that support and activation are two different steps.
**CN:** 本节清楚地区分了“构建期开启”和“运行时使用”两件事：先通过宏把相关指令编进内核，再在运行时显式用参数启用 PDL。这说明“具备支持”和“真正启用”是两个不同阶段。

### Model-Aware Optimizations with PDL
**EN:** The optimization example adds a performance perspective: if one operand is known not to depend on the previous kernel, prologue work such as weight prefetch can overlap with the predecessor's final memory flush. This shows PDL as a scheduling tool, not merely a synchronization primitive.
**CN:** 优化示例进一步给出性能视角：如果某个输入已知不依赖前一个内核，就可以让权重预取等前导工作与前序内核的最终内存刷写并行进行。这表明 PDL 不只是同步原语，更是一种调度工具。

## Key Concepts / 关键概念
- Programmatic dependent launch / 程序化依赖启动
- Safe kernel overlap in one stream / 单一流中的安全内核重叠
- Build-time vs runtime enablement / 构建期开启与运行时启用
- Memory flush ordering / 内存刷写顺序保证
- Prefetch-based optimization opportunities / 基于预取的优化机会

## Related Files / 相关文件
- `media/docs/cpp/pipeline.md` — broader synchronization context / 更广义的同步背景
- `media/docs/cpp/profiler.md` — profiler flag `--use-pdl` references this topic / profiler 中 `--use-pdl` 会引用本主题
- `include/cutlass/arch/barrier.h` — related low-level synchronization support / 相关底层同步支持
