# sm90_ssd_epilogue.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/collective/sm90_ssd_epilogue.hpp`  
**Purpose / 用途**: Hopper SM90 SSD epilogue collective that turns accumulated intra/inter results into output tensors, final states, and optional D/Z post-processing. / Hopper SM90 SSD 的 epilogue collective：把 intra/inter 累加结果写成输出张量、最终状态，并处理可选的 D/Z 后处理。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (lines 34-205) — Template parameters, storage, layouts, and TMA descriptors
```cpp
template<
  class ElementAcc_, class Element_, class TileShape_, class EpilogueTile_,
  class SmemLayoutX_, class SmemLayoutY_, class SmemLayoutPartialY_,
  class SmemLayoutP_, class SmemLayoutZ_,
  int StagesD_, int StagesY_, int StagesZ_,
  bool HAS_D_, bool D_HAS_HDIM_, bool HAS_Z_>
struct SsdEpilogue {
  struct CollectiveStorage {
    alignas(SmemAlignmentY) ArrayEngine<ElementAcc, cosize_v<SmemLayoutPartialY>> smem_y_partial;
    alignas(SmemAlignmentY) ArrayEngine<ElementY  , cosize_v<SmemLayoutY>>        smem_y;
    alignas(SmemAlignmentY) ArrayEngine<ElementZ  , cosize_v<SmemLayoutZ>>        smem_z;
  };

  using EpiloadPipelineD = cutlass::PipelineTmaAsync<StagesD>;
  using EpiloadPipelineZ = cutlass::PipelineTmaAsync<StagesZ>;
  using StorePipeline    = cutlass::PipelineTmaStore<StagesY>;
  using StorePPipeline   = cutlass::PipelineTmaStore<1>;
```
**EN**
- The template is parameterized by tile shapes, shared-memory layouts, stage counts, and feature switches (`HAS_D`, `D_HAS_HDIM`, `HAS_Z`). This lets the same epilogue map to several SSD kernel variants.
- The epilogue owns only the buffers that are unique to output composition: partial intra accumulation, final `Y`, and optional `Z`. Recurrent state `P` and skip term `D` stay in the mainloop storage.
- `Params` then builds TMA descriptors for storing `Y`, storing `P`, loading `Z`, and a tensor view for `D`. `to_underlying_arguments()` binds raw pointers and layouts to those descriptors at runtime.

**CN**
- 模板参数覆盖 tile 形状、共享内存布局、pipeline stage 数，以及功能开关（`HAS_D`、`D_HAS_HDIM`、`HAS_Z`）。因此同一个 epilogue 可以适配多个 SSD kernel 变体。
- epilogue 自己只持有输出组合阶段独有的缓冲：intra 局部累加、最终 `Y`、以及可选 `Z`。递归状态 `P` 和跳连项 `D` 仍放在 mainloop 存储中。
- 随后的 `Params` 构造了 `Y` 存储、`P` 存储、`Z` 加载的 TMA 描述符，以及 `D` 的 tensor 视图。`to_underlying_arguments()` 在运行时把原始指针和 layout 绑定进去。

### Block 2 (lines 208-259) — Loading `D` into the epilogue side
```cpp
template<class Params, class ProblemShape, class EpiloadPipeline, class PipelineState, class TensorStorage>
CUTLASS_DEVICE
void load_d(int const& blk_coord, Params const& params, ProblemShape const& problem_size,
            EpiloadPipeline& pipeline, PipelineState& pipeline_state,
            TensorStorage& shared_tensors) {
  if constexpr (D_HAS_HDIM) {
    int lane_predicate = cute::elect_one_sync();
    if (lane_predicate) {
      pipeline.producer_acquire(pipeline_state);
      auto* tma_barrier = pipeline.producer_get_barrier(pipeline_state);
      copy(bulk_atom.with(*tma_barrier), gD(_,blk_coord), sD(_,write_stage));
      ++pipeline_state;
    }
  }
}
```
**EN**
- `load_d()` is only meaningful when `D` has a head dimension (`D_HAS_HDIM`). In that case one elected lane issues a bulk gmem→smem copy into `smem_d`, staged by the epilogue-load pipeline.
- `load_d_tail()` closes the producer side with `producer_tail()` so cluster/block teardown does not race past unfinished pipeline stages.
- If `D` exists but is scalar-per-block instead of vector-per-head, the code skips this preload path and later broadcasts the scalar directly in `update_d()`.

**CN**
- 只有当 `D` 带 head 维度（`D_HAS_HDIM`）时，`load_d()` 才真正执行。此时由一个被选中的 lane 发起 bulk gmem→smem 拷贝，把 `D` 装入 `smem_d`，并由 epilogue load pipeline 分阶段管理。
- `load_d_tail()` 通过 `producer_tail()` 正确结束生产者侧，避免 cluster/block 退出时越过未完成的 pipeline stage。
- 如果 `D` 存在但只是每个 block 一个标量，而不是按 head 展开的向量，那么这里不会预加载，而是在 `update_d()` 中直接广播标量。

### Block 3 (lines 261-352) — Optional `Z` gate prefetch with TMA
```cpp
auto load_z_init(Params const& params, ProblemShape const& problem_size) {
  Tensor mZ_mkl = params.tma_load_z.get_tma_tensor(make_shape(L,D,C,EH*B));
  Tensor gZ_mkl = local_tile(mZ_mkl, TileShape{}, make_coord(_,_,_), Step<_1,_1,X>{});
  return make_tuple(gZ_mkl);
}

void load_z(..., EpiLoadPipeline& pipeline, PipelineState& pipeline_state, ... ) {
  if constexpr (HAS_Z) {
    if (cute::elect_one_sync()) {
      for (int chunk_idx = 0; chunk_idx < chunk; ++chunk_idx) {
        pipeline.producer_acquire(pipeline_state);
        auto* tma_barrier = pipeline.producer_get_barrier(pipeline_state);
        copy(params.tma_load_z.with(*tma_barrier, mcast_mask_z),
             tZgZ(_,chunk_idx), tZsZ(_,write_stage));
        ++pipeline_state;
      }
    }
  }
}
```
**EN**
- `load_z_init()` creates the global tensor view for all chunked `Z` tiles. `load_z()` then partitions that tensor and streams each chunk into `smem_z` through TMA.
- Only one elected lane issues the TMA instructions; other threads rely on the pipeline/barrier protocol for visibility.
- `Z` is optional. When enabled, it is consumed later by `store()` and passed through `SiLu` before gating the final output.

**CN**
- `load_z_init()` 先构造所有 chunk 的 `Z` 全局 tensor 视图，`load_z()` 再把它按当前 block 分块，并通过 TMA 流式搬运到 `smem_z`。
- 只有一个被选中的 lane 负责发出 TMA 指令；其余线程通过 pipeline/barrier 协议获得可见性保证。
- `Z` 是可选项。启用后，它会在 `store()` 中被消费，并先经过 `SiLu`，再作为最终输出门控。

### Block 4 (lines 354-432) — `store_intra()` bridges the two SSD branches
```cpp
auto store_intra(..., CooperatePipeline& cooperate_pipeline,
                 CooperatePipelineState& cooperate_pipe_producer_state,
                 TensorIntra& tIntra, TiledMma tiled_mma,
                 TensorStorage& shared_tensors) {
  Tensor tRS_rIntra = thread_r2s.retile_S(tIntra);
  Tensor tRS_sY_frg = recast<Array<ElementAcc, FragmentSize>>(sY_epi(thread_idx,_,_));

  for (int epi_n = 0; epi_n < size<3>(gY_epi); ++epi_n) {
    for (int epi_m = 0; epi_m < size<2>(gY_epi); ++epi_m) {
      cooperate_pipeline.producer_acquire(cooperate_pipe_producer_state);
      copy(tRS_rY_frg, tRS_sY_frg(_,cooperate_pipe_producer_state.index()));
      cutlass::arch::fence_view_async_shared();
      cooperate_pipeline.producer_commit(cooperate_pipe_producer_state);
      ++cooperate_pipe_producer_state;
    }
  }
}
```
**EN**
- This function does not write global memory. Instead, it converts the intra-branch accumulator into epilogue-layout fragments and places them into `smem_y_partial` through `CooperatePipeline`.
- The logical role is a handoff from Consumer0 (intra branch) to Consumer1 (final epilogue branch). The two warpgroups stay decoupled except for this explicit pipeline.
- The tiling math maps MMA fragments to epilogue subtiles so later `store()` can add the intra term at the correct `(epi_m, epi_n)` location.

**CN**
- 这个函数不会写全局内存。它把 intra 分支的累加器重排成 epilogue 所需的 fragment，并通过 `CooperatePipeline` 放入 `smem_y_partial`。
- 它的逻辑作用是把 Consumer0（intra 分支）的结果交接给 Consumer1（最终 epilogue 分支）。两个 warpgroup 只通过这个显式 pipeline 协作。
- 其中的 tiling 映射把 MMA fragment 对齐到 epilogue 子 tile，因此后续 `store()` 能在正确的 `(epi_m, epi_n)` 位置把 intra 项加回去。

### Block 5 (lines 434-480) — `update_d()` materializes the skip term
```cpp
auto update_d(int const& blk_coord, Params params, bool is_first_iteration,
              EpiloadPipeline& epi_load_pipeline,
              EpiloadPipelineState& epi_load_pipe_consumer_state,
              TiledMma tiled_mma, TensorStorage& shared_tensors) {
  auto sD  = make_tensor(make_smem_ptr(shared_tensors.smem_d.data()), d_tv_layout)
               (make_coord(thread_idx,_), make_coord(_,_,read_stage));
  auto tSR_rD = make_tensor<ElementD>(shape(sD));
  auto tD = make_tensor<ElementAcc>(shape(sD));

  if constexpr (D_HAS_HDIM) {
    if (is_first_iteration) epi_load_pipeline.consumer_wait(epi_load_pipe_consumer_state);
    copy(tSR_sD, tSR_rD);
    type_convert<ElementD, ElementAcc>(tSR_rD, tD);
  } else if constexpr (HAS_D) {
    auto value = static_cast<ElementAcc>(gD(_0{}, blk_coord));
    for (int ii = 0; ii < size(tSR_rD); ++ii) tD(ii) = value;
  }
  return make_tuple(tD);
}
```
**EN**
- `update_d()` converts the external skip coefficient `D` into an accumulator-shaped fragment aligned with the current GMMA tile.
- For vector `D`, it waits on the epilogue load pipeline once, then reads from `smem_d` and converts to accumulator precision.
- For scalar `D`, it broadcasts one value across the whole fragment. The result is later multiplied by `X` inside `store()`.

**CN**
- `update_d()` 把外部跳连系数 `D` 变成与当前 GMMA tile 对齐的累加器 fragment。
- 如果 `D` 是向量形式，就在第一次使用时等待 epilogue load pipeline，然后从 `smem_d` 读出并转换到累加精度。
- 如果 `D` 是标量形式，则把一个值广播到整个 fragment。之后在 `store()` 中与 `X` 相乘。

### Block 6 (lines 483-639) — `store()` setup: tiling, fragment views, and the deferred TMA store lambda
```cpp
Tensor gY_mn = local_tile(mY_mn, take<0,2>(TileShape{}), make_coord(_,_,_))(_,_,_0{},_0{},chunk,blk_coord);
Tensor gY_epi = flat_divide(gY_mn, EpilogueTile{});
Tensor sY_epi = as_position_independent_swizzle_tensor(make_tensor(make_smem_ptr(ptr_sY), SmemLayoutY{}));
Tensor sX_epi = as_position_independent_swizzle_tensor(make_tensor(make_smem_ptr(ptr_sX), SmemLayoutX{}));
Tensor sZ_epi = as_position_independent_swizzle_tensor(make_tensor(make_smem_ptr(ptr_sZ), SmemLayoutZ{}));
...
auto tma_store_fn = [&] (int epi_m, int epi_n) {
  cutlass::arch::fence_view_async_shared();
  synchronize();
  if (issue_tma_store) {
    copy(params.tma_store_y, bSG_sY(_,store_pipe_producer_state.index()), bSG_gY(_,epi_m,epi_n));
    store_pipeline.producer_commit(store_pipe_producer_state);
  }
  ++store_pipe_producer_state;
  if (issue_tma_store) store_pipeline.producer_acquire(store_pipe_producer_state);
  synchronize();
};
```
**EN**
- `store()` is the real SSD output composer. It prepares global and shared-memory tiles for `Y`, `X`, and optional `Z`, plus the partial-intra buffer previously produced by `store_intra()`.
- The function builds both register fragments and TMA partitions so each subtile can be computed in registers, written to `smem_y`, and then asynchronously stored to global memory.
- `tma_store_fn()` delays the actual TMA store until the latest safe point. This is a Hopper optimization: compute can continue while the store pipeline drains, but the next shared-memory buffer must still be acquired in order.

**CN**
- `store()` 是真正的 SSD 输出组合器。它准备 `Y`、`X`、可选 `Z` 的全局/共享内存 tile，以及之前由 `store_intra()` 生成的 intra 局部缓冲。
- 该函数同时构造寄存器 fragment 和 TMA 分区，使每个 subtile 先在寄存器中完成计算，再写入 `smem_y`，最后异步存回全局内存。
- `tma_store_fn()` 会把真正的 TMA store 延迟到“最后一个安全时刻”。这是 Hopper 上常见的优化：计算可以与 store pipeline 排空重叠，但下一块共享内存缓冲仍必须按顺序 acquire。

### Block 7 (lines 640-741) — Final epilogue math and `Y` store
```cpp
if constexpr (HAS_Z) {
  copy(tiled_s2r_z, tSR_sZ(_,_,_,epi_m,epi_n,pipeline_state_z.index()), tSR_rZ);
  type_convert<ElementZ, ElementAcc>(tSR_rZ, tRS_rZ);
  cutlass::epilogue::thread::SiLu<ElementAcc> op;
  for (int ii = 0; ii < size(tRS_rZ); ++ii) tRS_rZ(ii) = op(tRS_rZ(ii));
}

cooperate_pipeline.consumer_wait(cooperate_pipe_consumer_state);
copy(tSR_sY_frg(_,cooperate_pipe_consumer_state.index()), tSR_rY_frg);
cooperate_pipeline.consumer_release(cooperate_pipe_consumer_state);
...
for (int epi_v = 0; epi_v < size(tRS_rCompute_frg); ++epi_v) {
  tRS_rCompute_frg(epi_v) = tRS_rDeltaA_frg_mn(r2s_v + epi_v) *
                            tRS_rInter_frg_mn(r2s_v + epi_v) +
                            tSR_rY_frg(epi_v);
  if constexpr (HAS_D) tRS_rCompute_frg(epi_v) = tRS_rD_frg_mn(r2s_v + epi_v) * tRS_rX_frg(epi_v) + tRS_rCompute_frg(epi_v);
  if constexpr (HAS_Z) tRS_rCompute_frg(epi_v) = tRS_rCompute_frg(epi_v) * tRS_rZ_frg(epi_v);
}
copy(tiled_r2s, tRS_rY, tRS_sY(_,_,_,store_pipe_producer_state.index()));
```
**EN**
- This loop is where the SSD decomposition becomes the final output equation. It reads:
  1. inter result from `tInter`,
  2. per-position exponential factor from `tDeltaA`,
  3. intra partial from `smem_y_partial`,
  4. optional skip operand `D * X`,
  5. optional gate `SiLU(Z)`.
- The accumulation order is explicit: `deltaA * inter + intra`, then `+ D * X` if present, then gate by `SiLU(Z)` if present.
- After conversion back to output precision, the data is stored to `smem_y`, and the deferred TMA store commits each subtile to global memory.
- At the end, the function releases the `X` and `Z` consumer pipeline stages, because this chunk no longer needs those tiles.

**CN**
- 这个循环就是 SSD 分解最终汇合成输出公式的地方。它会读取：
  1. 来自 `tInter` 的 inter 结果，
  2. 来自 `tDeltaA` 的逐位置指数因子，
  3. 来自 `smem_y_partial` 的 intra 局部结果，
  4. 可选跳连项 `D * X`，
  5. 可选门控 `SiLU(Z)`。
- 累加顺序在代码里写得很清楚：先 `deltaA * inter + intra`，若启用 `D` 再加上 `D * X`，若启用 `Z` 再乘上 `SiLU(Z)`。
- 完成后会把结果转换回输出精度，写入 `smem_y`，再由延迟的 TMA store 把每个 subtile 提交到全局内存。
- 在函数结尾，`X` 与 `Z` 的 consumer pipeline stage 会被释放，因为当前 chunk 已经不再需要这些 tile。

### Block 8 (lines 743-834) — Final state `P` store and local type conversion helper
```cpp
auto store_p(..., StorePipeline& store_pipeline, StorePipelineState& store_pipe_producer_state,
             TensorStorage& shared_tensors) {
  Tensor gP_mn = local_tile(mP_mn, take<1,3>(TileShape{}), make_coord(_,_,blk_coord));
  Tensor sP_epi = as_position_independent_swizzle_tensor(make_tensor(make_smem_ptr(ptr_sP), SmemLayoutP{}));
  ...
  if (issue_tma_store) {
    copy(params.tma_store_p, bSG_sY, bSG_gY);
    store_pipeline.producer_commit(store_pipe_producer_state);
  }
}

template<class ElementSrc, class ElementDst, class TensorSrc, class TensorDst>
CUTLASS_DEVICE auto type_convert(TensorSrc& tS, TensorDst& tD) {
  NumericArrayConverter<ElementDst, ElementSrc, 2> converter;
  ...
}
```
**EN**
- `store_p()` exports the final recurrent SSD state from shared memory to global memory. This is the chunk boundary object consumed outside the current tile.
- The implementation mirrors `Y` storage but is simpler: there is only one-stage buffering, because `P` is a single final state tile rather than a stream of subtiles.
- `type_convert()` is the utility used throughout the file to switch between source precision (for loads/stores) and accumulator precision (for compute).

**CN**
- `store_p()` 把最终的 SSD 递归状态从共享内存导出到全局内存。这是当前 tile 之外还要继续使用的 chunk 边界状态对象。
- 它的实现与 `Y` 的存储类似，但更简单：这里只需要单 stage 缓冲，因为 `P` 是一个最终状态 tile，而不是一串 subtile 流。
- `type_convert()` 是本文件的基础工具函数，负责在加载/存储精度与计算累加精度之间做转换。

---

## Key Concepts / 关键概念

1. **TMA load/store pipelines**
   - **EN:** `PipelineTmaAsync` is used for loading `D` and `Z`, while `PipelineTmaStore` is used for asynchronously storing `Y` and `P` back to global memory.
   - **CN:** `PipelineTmaAsync` 用于加载 `D` 与 `Z`，`PipelineTmaStore` 用于把 `Y` 与 `P` 异步写回全局内存。
2. **Warp specialization handoff**
   - **EN:** the intra path and inter path run in different consumer warpgroups. `store_intra()` writes the intra result into a cooperation pipeline so the final consumer can add it during `store()`.
   - **CN:** intra 路径和 inter 路径在不同的 consumer warpgroup 中执行。`store_intra()` 先把 intra 结果写入协作流水线，随后最终消费者在 `store()` 中把它加回来。
3. **SSD recurrence composition**
   - **EN:** the final output is assembled as `Y = intra + deltaA_col * inter (+ D * X)`, then optionally gated by `SiLU(Z)`.
   - **CN:** 最终输出按 `Y = intra + deltaA_col * inter (+ D * X)` 组合，如果启用 `Z`，再乘上 `SiLU(Z)` 门控。
4. **Barrier discipline on Hopper**
   - **EN:** the file relies on `fence_view_async_shared()`, named barriers, and producer/consumer pipeline state transitions to make TMA, GMMA, and shared-memory exchanges visible in the right order.
   - **CN:** 该文件依赖 `fence_view_async_shared()`、named barrier 以及 producer/consumer pipeline 状态推进，保证 TMA、GMMA 和共享内存交换按正确顺序可见。

## Dependencies / 依赖项

- `cutlass/epilogue/collective/collective_builder.hpp`
  - **EN:** provides CUTLASS collective and pipeline building blocks used by the SSD epilogue.
  - **CN:** 提供 SSD epilogue 所需的 CUTLASS collective 与 pipeline 基础组件。
- `cutlass/fast_math.h`
  - **EN:** supports the fast math path used around activation/gating.
  - **CN:** 为激活/门控相关的快速数学路径提供支持。
- `examples/111_hopper_ssd/collective/sm90_ssd_gemm_tma_warpspecialized.hpp`
  - **EN:** defines the mainloop shared-memory layouts and the fragments that this file consumes, especially `X`, `P`, and `D` storage conventions.
  - **CN:** 定义了本文件要消费的 mainloop 共享内存布局和片段，尤其是 `X`、`P`、`D` 的存储约定。
- `examples/111_hopper_ssd/kernel/sm90_ssd_kernel_tma_warpspecialized.hpp`
  - **EN:** orchestrates which warps produce `D/Z`, which warpgroup produces intra partials, and which warpgroup performs the final store.
  - **CN:** 负责调度哪些 warp 生产 `D/Z`，哪个 warpgroup 生产 intra 局部结果，以及哪个 warpgroup 执行最终写回。
