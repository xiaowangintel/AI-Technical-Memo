# sm90_ssd_tile_scheduler.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/kernel/sm90_ssd_tile_scheduler.hpp`  
**Purpose / 用途**: Persistent tile scheduler for Hopper SSD that assigns chunk/head tiles to CTAs and drives grid traversal. / Hopper SSD 的 persistent tile 调度器：负责把 chunk/head tile 分配给 CTA 并驱动网格遍历。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (lines 32-57) — Includes, params, state, and constructor
```cpp
#pragma once

#include "cutlass/cutlass.h"
#include "cutlass/fast_math.h"
#include "cutlass/kernel_hardware_info.h"

namespace cutlass::ssd::kernel {

struct PersistentTileScheduler {

  struct Params {
    int num_blocks;
    int num_groups;
    FastDivmod divmod_eh;
    FastDivmod divmod_ngroup_ratio;

    KernelHardwareInfo hw_info;
  };

  int block_idx = 0;
  Params params;

  CUTLASS_DEVICE
  PersistentTileScheduler(Params const& params) : block_idx(blockIdx.x), params(params) {}
```
**EN**
- `Params` stores exactly the runtime state needed by the scheduler: total work count, group count, precomputed divmod helpers, and hardware information.
- `block_idx` is initialized from the physical CUDA block id, then later advanced persistently by the scheduler itself.

**CN**
- `Params` 精确保存了 scheduler 所需的运行时状态：总工作量、group 数、预计算的 divmod 辅助器以及硬件信息。
- `block_idx` 先从物理 CUDA block id 初始化，随后再由 scheduler 自己以 persistent 方式推进。

### Block 2 (lines 59-88) — Lower problem shape into scheduler params
```cpp
template<class ProblemSize, class ClusterShape, class TileShape>
static Params to_underlying_arguments(
    ProblemSize const& problem_size, KernelHardwareInfo hw_info,
    ClusterShape const& cluster_shape, TileShape const& tile_shape)
{
  auto [G, B, EH, C, L, D, N] = problem_size;

  int sm_count = hw_info.sm_count;
  if (sm_count <= 0) {
    ...
    sm_count = KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
  }

  int num_blocks = B * EH;
  int ngroup_ratio = EH / G;

  return Params {
    num_blocks,
    G,
    {EH},
    {ngroup_ratio},
    hw_info
  };
}
```
**EN**
- The logical tile space is flattened to `B * EH` work items.
- `EH` and `EH / G` are precomputed as `FastDivmod` objects so later coordinate recovery is cheap inside the kernel.
- `C`, `L`, `D`, and `N` belong to the full SSD problem shape, but they do not affect CTA assignment at this scheduling layer.

**CN**
- 逻辑 tile 空间被展平为 `B * EH` 个工作项。
- `EH` 和 `EH / G` 被预计算为 `FastDivmod` 对象，以便后续在 kernel 中低成本恢复坐标。
- `C`、`L`、`D` 和 `N` 虽然属于完整的 SSD 问题形状，但在这个调度层并不影响 CTA 分配。

### Block 3 (lines 90-103) — Grid shape, validity, and flat coordinate
```cpp
static dim3 get_grid_shape(Params const& params) {
  dim3 grid(std::min(params.num_blocks, params.hw_info.sm_count), 1, 1);
  return grid;
}

CUTLASS_DEVICE
bool is_valid() {
  return block_idx < params.num_blocks;
}

CUTLASS_DEVICE
auto get_block_coord() {
  return block_idx;
}
```
**EN**
- The grid is capped at SM count, which is the defining move of persistent scheduling.
- `is_valid()` tells a CTA whether more logical tiles remain.
- `get_block_coord()` exposes the flat `(B, EH)` work id used by head-specific paths.

**CN**
- grid 被限制在 SM 数以内，这是 persistent scheduling 的标志性动作。
- `is_valid()` 用来判断当前 CTA 是否还有未处理的逻辑 tile。
- `get_block_coord()` 暴露扁平化的 `(B, EH)` 工作 id，供按 head 区分的路径使用。

### Block 4 (lines 105-121) — Group remapping and EH extraction
```cpp
CUTLASS_DEVICE
auto get_block_coord_b() {
  int eh_idx, b_idx;
  int g_idx, rest_idx;
  params.divmod_eh(b_idx, eh_idx, block_idx);
  params.divmod_ngroup_ratio(g_idx, rest_idx, eh_idx);
  return (params.num_groups * b_idx + g_idx);
}

CUTLASS_DEVICE
auto get_block_coord_eh() {
  int eh_idx, b_idx;
  params.divmod_eh(b_idx, eh_idx, block_idx);
  return eh_idx;
}
```
**EN**
- `get_block_coord_b()` remaps a flat `(B, EH)` work id into `(B, G)` space, which is exactly what the B/C loading path needs for group-shared tensors.
- `get_block_coord_eh()` extracts just the expanded-head coordinate, which the kernel later uses for head-indexed tensors such as D.

**CN**
- `get_block_coord_b()` 会把扁平化的 `(B, EH)` 工作 id 重映射到 `(B, G)` 空间，这正是 B/C 加载路径在处理 group 共享张量时所需要的。
- `get_block_coord_eh()` 则提取纯粹的扩展 head 坐标，后续 kernel 会用它来索引诸如 D 这样按 head 组织的张量。

### Block 5 (lines 123-127) — Persistent increment
```cpp
CUTLASS_DEVICE
PersistentTileScheduler& operator++() {
  block_idx += gridDim.x;
  return *this;
}
```
**EN**
- This one line is the core of persistence: each CTA jumps by the full grid width to claim its next logical tile.
- Because the grid was capped at SM count earlier, the same heavy CTA can keep the SM busy across many SSD tiles.

**CN**
- 这一行就是持久化调度的核心：每个 CTA 以整个 grid 宽度为步长跳到下一个逻辑 tile。
- 由于前面已经把 grid 限制在 SM 数以内，同一个较重的 CTA 就可以在多个 SSD tile 上持续复用并保持 SM 忙碌。

---

## Key Concepts / 关键概念

1. **Persistent CTA scheduling**
   - **EN:** The scheduler keeps the grid small and reuses each CTA across many logical tiles.
   - **CN:** 调度器保持较小的 grid，并让每个 CTA 复用处理多个逻辑 tile。
2. **Problem shape interpretation**
   - **EN:** The SSD problem shape is treated as `[G, B, EH, C, L, D, N]`, but scheduler assignment depends mainly on `B`, `EH`, and `G`.
   - **CN:** SSD 问题形状被视为 `[G, B, EH, C, L, D, N]`，但调度分配主要依赖 `B`、`EH` 和 `G`。
3. **Grouped-head remapping**
   - **EN:** `get_block_coord_b()` converts a flat `(B, EH)` work id into `(B, G)` space because some SSD operands are group-shared instead of head-specific.
   - **CN:** `get_block_coord_b()` 会把扁平化的 `(B, EH)` 工作 id 转成 `(B, G)` 空间，因为某些 SSD 操作数是 group 共享的，而不是每个 head 独有的。
4. **Hardware-aware launch sizing**
   - **EN:** The scheduler uses `KernelHardwareInfo` to clamp the grid to SM count and queries SM count if the caller did not provide it.
   - **CN:** 调度器使用 `KernelHardwareInfo` 把 grid 限制到 SM 数量；如果调用方没有提供 SM 数，它会自行查询。

## Dependencies / 依赖项

- `cutlass/cutlass.h`
  - **EN:** Supplies CUTLASS annotations and tracing support.
  - **CN:** 提供 CUTLASS 标注与 trace 支持。
- `cutlass/fast_math.h`
  - **EN:** Provides `FastDivmod`, which decodes flat block ids efficiently inside the kernel.
  - **CN:** 提供 `FastDivmod`，用于在 kernel 内高效解码扁平 block id。
- `cutlass/kernel_hardware_info.h`
  - **EN:** Provides `KernelHardwareInfo`, including SM-count query helpers.
  - **CN:** 提供 `KernelHardwareInfo`，包括 SM 数查询辅助接口。
