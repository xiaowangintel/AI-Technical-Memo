# dispatch_policy_extra.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/63_hopper_gemm_with_weight_prefetch/collective/dispatch_policy_extra.hpp`  
**Purpose / 用途**: Adds custom Hopper mainloop schedule tags and a lightweight dispatch-policy wrapper so CUTLASS collective builders can instantiate FP8 weight-prefetch kernels beyond the stock schedule set. / 为 CUTLASS 的 collective builder 增加自定义 Hopper 主循环调度 tag 与轻量级 dispatch-policy 包装，从而实例化超出默认调度集合的 FP8 权重预取内核。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-35)
```cpp
#pragma once

namespace cutlass::gemm {
```
**EN**: The file is intentionally tiny. After the standard license header it only opens `cutlass::gemm`, which is important because the new schedule tags must live in the same namespace family CUTLASS builders and collective dispatch traits already search.

**CN**: 这个文件刻意保持很小。标准许可证头之后只进入 `cutlass::gemm` 命名空间；这一点很关键，因为新的 schedule tag 必须放在 CUTLASS builder 与 collective dispatch traits 已经会查找的同一命名空间体系中。

### Block 2 (Lines 36-47)
```cpp
// Standard non-persistent kernel with a single producer warp, and one prefetch warp.
// `A` is assumed to be static...
struct KernelTmaWarpSpecializedFP8FastAccumWithPrefetch { };

// Non-persistent kernel with two producer warps...
struct KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA { };
```
**EN**: These two empty structs are pure type tags, but their comments precisely define the intended execution policy. `KernelTmaWarpSpecializedFP8FastAccumWithPrefetch` means a non-persistent warp-specialized FP8 fast-accumulation kernel with one normal producer warp plus one extra prefetch warp; the `A` operand is assumed static, so it can be prefetched while the producer warp is waiting on grid dependency control. `KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA` goes further and splits DMA responsibility across two producer warps, one for `A` and one for `B`, while keeping a dedicated prefetch warp. Those tags are the extension points matched later by builder specializations.

**CN**: 这两个空结构体本身只是类型 tag，但注释精确给出了预期执行策略。`KernelTmaWarpSpecializedFP8FastAccumWithPrefetch` 表示一种非 persistent 的 warp-specialized FP8 fast-accum 内核：除了常规 producer warp 外，再增加一个预取 warp；由于假定 `A` 是静态权重，所以可以在 producer warp 等待 grid dependency control 时预取 `A`。`KernelTmaWarpSpecializedFP8FastAccumWithPrefetchAndSplitDMA` 更进一步，把 DMA 职责拆给两个 producer warp（分别服务 `A` 和 `B`），同时保留专用预取 warp。后续 builder 特化正是通过匹配这些 tag 接入扩展调度。

### Block 3 (Lines 49-61)
```cpp
template<
  int Stages_,
  class ClusterShape_ = Shape<_1,_1,_1>,
  class KernelSchedule = KernelTmaWarpSpecializedFP8FastAccumWithPrefetch
>
struct MainloopSm90TmaGmmaWarpSpecializedWithPrefetch {
  constexpr static int Stages = Stages_;
  using ClusterShape = ClusterShape_;
  using ArchTag = arch::Sm90;
  using Schedule = KernelSchedule;
};
```
**EN**: `MainloopSm90TmaGmmaWarpSpecializedWithPrefetch` is the actual dispatch-policy carrier. It packages four compile-time facts that the rest of CUTLASS expects from a mainloop policy: pipeline stage count, cluster shape, target architecture (`Sm90`), and schedule tag. The type itself contains no behavior; its job is to give builder specializations and collective implementations a stable policy object they can pattern-match on.

**CN**: `MainloopSm90TmaGmmaWarpSpecializedWithPrefetch` 才是真正承载 dispatch policy 的包装类型。它把 CUTLASS 其余组件需要的四个编译期事实打包起来：pipeline stage 数、cluster 形状、目标架构（`Sm90`）以及 schedule tag。这个类型本身不实现行为；它的作用是给 builder 特化与 collective 实现提供一个稳定的“策略对象”，方便做类型匹配。

---

## Key Concepts / 关键概念

- **Dispatch-policy extension / 调度策略扩展**: The file extends CUTLASS by adding new schedule tags rather than modifying generic builder code directly. / 该文件通过新增 schedule tag 扩展 CUTLASS，而不是直接改写通用 builder 逻辑。
- **Type-level policy plumbing / 类型级策略传递**: Execution intent is represented entirely in types, which builder specializations can recognize at compile time. / 执行意图完全通过类型表达，builder 特化可在编译期识别。
- **Prefetch-aware Hopper mainloops / 面向预取的 Hopper 主循环**: Both tags describe variants that overlap static-weight prefetch with the normal TMA/GMMA pipeline. / 两个 tag 都描述了将静态权重预取与常规 TMA/GMMA 流水重叠的变体。

## Dependencies / 依赖项

- **`cute::Shape<_1,_1,_1>`**: Default single-CTA cluster shape. / 默认单 CTA 的 cluster 形状。
- **`cutlass::arch::Sm90`**: Hard-codes that these schedules target Hopper. / 明确这些调度只面向 Hopper。
- **Sibling builder specializations**: `examples/63_hopper_gemm_with_weight_prefetch/collective/builder.hpp` consumes these tags to build a concrete `CollectiveMma`. / 同目录下的 `builder.hpp` 会消费这些 tag，并生成具体的 `CollectiveMma`。
