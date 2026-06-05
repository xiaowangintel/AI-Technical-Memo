# sm100_mla_tile_scheduler.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/attention/mla/cutlass_sm100_mla/kernel/sm100_mla_tile_scheduler.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines tile schedulers for MLA kernels, including a simple one-tile-per-CTA mode and a persistent mode that reuses CTAs across tiles. / 为 MLA 内核定义 tile 调度器，包括简单的一次一块模式以及跨多个 tile 复用 CTA 的 persistent 模式。

## Line-by-Line Analysis / 逐行分析
### Individual scheduler
```cpp
struct Sm100MlaIndividualTileScheduler {
  struct Params {
    dim3 grid;
  };
  ...
  static Params to_underlying_arguments(
      ProblemShape const& problem_shape, KernelHardwareInfo hw_info,
      ClusterShape const& cluster_shape, int const& split_kv) {
    using namespace cute;
    dim3 grid(get<0>(cluster_shape), get<3>(problem_shape) /* Batch */, split_kv /*Maximum Split KV*/);
    return Params{ grid };
  }
```
**EN:** The individual scheduler maps each CTA directly to a fixed `(m-block, batch, split_kv)` coordinate. It is simple and deterministic, which is useful when persistence is not desired.
**CN:** Individual 调度器会把每个 CTA 直接映射到固定的 `(m-block, batch, split_kv)` 坐标。它实现简单且确定性强，适合不需要 persistent 调度的场景。

### One-shot CTA lifetime
```cpp
CUTLASS_DEVICE
auto get_block_coord() {
  using namespace cute;
  return make_coord(blockIdx.x, _0{}, blockIdx.y, blockIdx.z);
}

CUTLASS_DEVICE
Sm100MlaIndividualTileScheduler& operator++() {
  valid_ = false;
  return *this;
}
```
**EN:** A CTA in this mode owns exactly one tile. After the first iteration, `operator++()` invalidates the scheduler so the kernel exits the tile loop immediately.
**CN:** 在该模式下，一个 CTA 只负责一个 tile。第一次迭代结束后，`operator++()` 会让调度器失效，从而使内核立即退出 tile 循环。

### Persistent scheduler
```cpp
struct Sm100MlaPersistentTileScheduler {
  struct Params {
    int num_blocks;
    FastDivmod divmod_m_block;
    FastDivmod divmod_b;
    FastDivmod divmod_split_kv;
    KernelHardwareInfo hw_info;
  };
  ...
  int num_blocks = num_m_blocks * get<3>(problem_shape)  /* Batch */;
  num_blocks *= split_kv; /* Maximum Split KV*/
```
**EN:** The persistent scheduler precomputes the total logical block count and stores fast div/mod helpers so a smaller physical grid can iterate over many logical tiles.
**CN:** Persistent 调度器会预先计算逻辑 block 总数，并保存快速除法/取模辅助器，从而让较小的物理网格可以遍历大量逻辑 tile。

### Reusing CTAs across work
```cpp
static dim3 get_grid_shape(Params const& params) {
  dim3 grid(std::min(params.num_blocks, params.hw_info.sm_count), 1, 1);
  return grid;
}
...
CUTLASS_DEVICE
Sm100MlaPersistentTileScheduler& operator++() {
  block_idx += gridDim.x;
  return *this;
}
```
**EN:** Instead of launching one CTA per tile, persistent mode launches up to one CTA per SM and advances each CTA by `gridDim.x` logical blocks. This improves residency and reduces launch overhead for many small tiles.
**CN:** Persistent 模式不会为每个 tile 单独启动一个 CTA，而是最多按每个 SM 一个 CTA 启动，并让每个 CTA 以 `gridDim.x` 的步长遍历逻辑 block。这样可在大量小 tile 场景中提升驻留率并减少启动开销。

## Key Concepts / 关键概念
- **Individual vs persistent scheduling / 单次与常驻调度**: Two execution styles trade simplicity for better SM reuse.
- **FastDivmod decoding / FastDivmod 解码**: Converts a flat block index back into `(m, batch, split)` coordinates efficiently.
- **SM-aware grid sizing / 感知 SM 的网格大小**: Persistent mode uses the hardware SM count to choose grid size.

## Dependencies / 依赖关系
- Consumed by `sm100_fmha_mla_tma_warpspecialized.hpp` as the `TileScheduler` template parameter.
- Uses CUTLASS `KernelHardwareInfo` and `FastDivmod` utilities.
