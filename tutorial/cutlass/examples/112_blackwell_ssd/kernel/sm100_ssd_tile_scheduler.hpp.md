# sm100_ssd_tile_scheduler.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/112_blackwell_ssd/kernel/sm100_ssd_tile_scheduler.hpp`  
**Purpose / 用途**: Persistent tile scheduler for Blackwell SSD that assigns chunk/head tiles to CTAs and manages traversal order. / Blackwell SSD 的 persistent tile 调度器：负责把 chunk/head tile 分配给 CTA 并管理遍历顺序。

---

## Line-by-Line Analysis / 逐行分析

### 1. Scheduler state and packed params (lines 42-57) / 调度器状态与打包参数（42-57 行）

```cpp
struct Params {
  int num_blocks;
  int num_groups;
  FastDivmod divmod_eh;
  FastDivmod divmod_ngroup_ratio;
  KernelHardwareInfo hw_info;
};

int block_idx = 0;
Params params;
```

**EN** The scheduler stores just enough information to decode a linear persistent block index into SSD coordinates. `FastDivmod` is used so index decomposition stays cheap inside the kernel loop.

**CN** 调度器只保存了将线性 persistent block 索引解码为 SSD 坐标所需的最小信息。这里使用 `FastDivmod`，是为了让内核循环中的索引拆分开销更低。

### 2. Converting problem size into scheduler params (lines 59-88) / 从问题规模生成调度参数（59-88 行）

```cpp
auto [G, B, EH, C, L, D, N] = problem_size;
...
int num_blocks = B * EH;
int ngroup_ratio = EH / G;

return Params {
  num_blocks,
  G,
  {EH},
  {ngroup_ratio},
  hw_info
};
```

**EN** The scheduler ignores sequence-local dimensions (`C`, `L`, `D`, `N`) because those are handled inside the kernel body. At the CTA scheduling level, SSD is decomposed primarily along batch `B` and expanded-head `EH`.

**CN** 调度器并不关心序列内部维度（`C`、`L`、`D`、`N`），因为这些维度是在 kernel 主体内部处理的。从 CTA 调度层面看，SSD 首先是沿着 batch `B` 和 expanded-head `EH` 拆分的。

### 3. SM-aware persistent grid sizing (lines 67-76, 90-93) / 感知 SM 数量的 persistent 网格大小（67-76、90-93 行）

```cpp
int sm_count = hw_info.sm_count;
if (sm_count <= 0) {
  sm_count = KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
}
...
dim3 grid(std::min(params.num_blocks, params.hw_info.sm_count), 1, 1);
```

**EN** This is classic persistent-kernel behavior: launch at most one resident CTA per SM initially, then let each CTA advance by `gridDim.x`. It is especially useful for SSD because each logical block contains a nontrivial chunk loop over `C`.

**CN** 这就是典型的 persistent kernel 行为：初始时最多每个 SM 放一个驻留 CTA，然后让每个 CTA 以 `gridDim.x` 为步长继续前进。对 SSD 来说这很合适，因为每个逻辑 block 内部还包含对 `C` 的非平凡 chunk 循环。

### 4. Validity and linear block identity (lines 95-103) / 有效性检查与线性 block 标识（95-103 行）

```cpp
bool is_valid() {
  return block_idx < params.num_blocks;
}

auto get_block_coord() {
  return block_idx;
}
```

**EN** `get_block_coord()` returns the raw logical tile id. In the Blackwell kernel, this raw id is used for X/Delta/D/Y-style tensors that are indexed per `(B, EH)` instance.

**CN** `get_block_coord()` 返回原始逻辑 tile id。在 Blackwell 内核中，这个原始 id 会用于按 `(B, EH)` 实例索引的 X/Delta/D/Y 一类张量。

### 5. Group-aware remapping for `B/C` tensors (lines 105-121) / 面向 `B/C` 张量的 group 重映射（105-121 行）

```cpp
params.divmod_eh(b_idx, eh_idx, block_idx);
params.divmod_ngroup_ratio(g_idx, rest_idx, eh_idx);
return (params.num_groups * b_idx + g_idx);
```

**EN** This method is the SSD-specific part of the scheduler. Expanded heads `EH` are mapped back to group `G`, because the `B` and `C` tensors are shared at group granularity rather than per-expanded-head granularity. The separate `get_block_coord_eh()` keeps the original expanded-head index when the kernel needs it.

**CN** 这是调度器里最 SSD 化的一部分。`EH` 会被重新映射回 group `G`，因为 `B` 和 `C` 张量是按 group 粒度共享的，而不是按每个 expanded head 单独存储。与此同时，`get_block_coord_eh()` 又保留了原始 expanded-head 索引，以便内核在需要时继续使用。

### 6. Persistent advancement rule (lines 123-127) / persistent 前进规则（123-127 行）

```cpp
PersistentTileScheduler& operator++() {
  block_idx += gridDim.x;
  return *this;
}
```

**EN** Every CTA walks the logical workload in a strided fashion. This means the scheduler itself is intentionally minimal; the real SSD recurrence scheduling happens one level deeper inside the SM100 kernel, where each logical block also iterates over chunk dimension `C`.

**CN** 每个 CTA 都以步进方式遍历逻辑工作负载。这说明调度器本身被故意设计得很薄；真正的 SSD recurrence 调度发生在更深一层的 SM100 内核里，因为每个逻辑 block 内部还会继续遍历 chunk 维 `C`。

## Architecture Note / 架构说明

**EN** Compared with the Hopper SSD example, this scheduler is essentially the same. The architectural shift from Hopper to Blackwell happens mainly in the builder and kernel body, not in this persistent block-to-tile mapping logic.

**CN** 和 Hopper SSD 示例相比，这个 scheduler 基本保持不变。从 Hopper 到 Blackwell 的架构迁移，主要体现在 builder 与 kernel 主体，而不是这里的 persistent block-to-tile 映射逻辑。

---

## Key Concepts / 关键概念

- Persistent scheduling / 常驻式调度
  - **EN** Grid size is capped by SM count, not by total problem tiles.
  - **CN** 网格大小受 SM 数量限制，而不是直接等于全部问题 tile 数。
- Logical block = `(B, EH)` slice / 逻辑 block 对应 `(B, EH)` 切片
  - **EN** `num_blocks = B * EH` shows that each persistent task corresponds to one batch-expanded-head pair.
  - **CN** `num_blocks = B * EH` 表明每个 persistent 任务对应一个 batch-expanded-head 对。
- Group remapping for `B/C`
  - **EN** `get_block_coord_b()` remaps expanded heads back to group index `G`, which matches the grouped `B/C` tensors used in SSD.
  - **CN** `get_block_coord_b()` 会把 expanded head 重新映射回组索引 `G`，这和 SSD 中按 group 组织的 `B/C` 张量一致。

## Dependencies / 依赖项

- `cutlass/cutlass.h`
- `cutlass/fast_math.h`
- `cutlass/kernel_hardware_info.h`
- Main helper types:
  - `FastDivmod`
  - `KernelHardwareInfo`
  - `dim3`
