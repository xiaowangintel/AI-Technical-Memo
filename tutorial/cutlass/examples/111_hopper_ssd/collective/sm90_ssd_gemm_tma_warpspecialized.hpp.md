# sm90_ssd_gemm_tma_warpspecialized.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/collective/sm90_ssd_gemm_tma_warpspecialized.hpp`  
**Purpose / 用途**: Hopper SM90 SSD mainloop collective that maps semiseparable SSD math onto TMA-fed, warp-specialized GMMA pipelines. / Hopper SM90 SSD 的主循环 collective：把半可分 SSD 数学映射到由 TMA 供数、warp-specialized 的 GMMA 流水线中。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (lines 34-253) — Collective definitions, tile decomposition, shared memory, and runtime params
```cpp
using TileShapeIntraBMM1 = decltype(make_shape(L, L, N));
using TileShapeIntraBMM2 = decltype(make_shape(L, D, N));
using TileShapeInterBMM1 = decltype(make_shape(N, D, L));
using TileShapeInterBMM2 = decltype(make_shape(L, D, N));

using CollectiveIntraMma1 = typename cutlass::gemm::collective::CollectiveBuilder<...>::CollectiveOp;
using CollectiveIntraMma2 = typename cutlass::gemm::collective::CollectiveBuilder<...>::CollectiveOp;
using CollectiveInterMma1 = typename cutlass::gemm::collective::CollectiveBuilder<...>::CollectiveOp;
using CollectiveInterMma2 = typename cutlass::gemm::collective::CollectiveBuilder<...>::CollectiveOp;

struct SharedStorage : cute::aligned_struct<128, _0> {
  cute::array_aligned<Element, cute::cosize_v<SmemLayoutX>> smem_x;
  cute::array_aligned<Element, cute::cosize_v<SmemLayoutB>> smem_b;
  cute::array_aligned<Element, cute::cosize_v<SmemLayoutC>> smem_c;
  cute::array_aligned<Element, cute::cosize_v<SmemLayoutP>> smem_p;
  cute::array_aligned<Element, get<0>(TileShape{}) * Stages::value> smem_delta;
  cute::array_aligned<ElementDA, get<0>(TileShape{}) * Stages::value> smem_delta_a;
  cute::array_aligned<Element, get<1>(TileShape{}) * Stages::value> smem_d;
};
```
**EN**
- The core design decision is visible immediately: SSD is rewritten as four GEMMs with different tile shapes and operand major modes.
- `convert_to_gmma_rs()` is used for `IntraMma2` and `InterMma1`, which means those stages reinterpret CUTLASS MMAs into the GMMA RS operand form needed by the later dataflow.
- Shared memory is partitioned by semantic role: `X`, `B`, `C`, `P`, `Delta`, `DeltaA`, and `D`. This storage layout is what producer warps fill and consumer warpgroups read.
- `Arguments` and `Params` connect raw global tensors to TMA descriptors (`X/B/C`) or plain tensor views (`Delta/DeltaA`). `Delta` uses bulk-copy style access rather than a TMA descriptor object.

**CN**
- 这里一开始就能看到核心设计：SSD 被改写成四个 GEMM，每个 GEMM 具有不同的 tile 形状和操作数主序。
- `IntraMma2` 与 `InterMma1` 使用了 `convert_to_gmma_rs()`，说明这两个阶段会把 CUTLASS MMA 重新解释成后续数据流需要的 GMMA RS 操作数形式。
- 共享内存按语义角色划分：`X`、`B`、`C`、`P`、`Delta`、`DeltaA`、`D`。producer warp 往这里填数据，consumer warpgroup 从这里读数据。
- `Arguments` 与 `Params` 负责把全局张量连接到 TMA 描述符（`X/B/C`）或普通 tensor 视图（`Delta/DeltaA`）。其中 `Delta` 采用 bulk-copy 风格访问，而不是独立 TMA 描述符对象。

### Block 2 (lines 256-369) — Descriptor prefetch and staged TMA loading for `X`
```cpp
CUTLASS_DEVICE
static void prefetch_tma_descriptors(Params const& params) {
  cute::prefetch_tma_descriptor(params.tma_load_x.get_tma_descriptor());
  cute::prefetch_tma_descriptor(params.tma_load_b.get_tma_descriptor());
  cute::prefetch_tma_descriptor(params.tma_load_c.get_tma_descriptor());
}

void load_x(..., MainloopPipeline& pipeline, PipelineState& pipeline_state, ... ) {
  if (cute::elect_one_sync()) {
    for (int chunk_idx = 0; chunk_idx < chunk; ++chunk_idx) {
      pipeline.producer_acquire(pipeline_state);
      auto* tma_barrier = pipeline.producer_get_barrier(pipeline_state);
      copy(params.tma_load_x.with(*tma_barrier, mcast_mask_x),
           tXgX(_,_,_,chunk_idx), tXsX(_,_,_,write_stage));
      ++pipeline_state;
    }
  }
}
```
**EN**
- The file prefetches only the true TMA descriptors (`X/B/C`). `Delta` is absent because it is moved with bulk copies instead.
- `load_x_init()` constructs a chunked global tensor view, and `load_x()` streams every chunk tile of `X` into staged shared memory.
- Only one elected lane issues TMA instructions; the entire warpgroup later observes readiness through the pipeline state.
- `load_x_tail()` performs the usual producer epilogue so cluster-scope scheduling does not exit early.

**CN**
- 这里只预取真正的 TMA 描述符（`X/B/C`）。`Delta` 不在其中，因为它走的是 bulk copy 路径。
- `load_x_init()` 构造按 chunk 划分的全局 tensor 视图，`load_x()` 则把每个 chunk 的 `X` tile 流式搬入分 stage 的共享内存。
- 只有一个被选中的 lane 负责发 TMA 指令；后续整个 warpgroup 通过 pipeline 状态感知数据是否就绪。
- `load_x_tail()` 执行标准的 producer epilogue，避免 cluster 范围调度时过早退出。

### Block 3 (lines 371-526) — Coupled loading of `B` and `C`
```cpp
auto load_b_init(...) {
  Tensor mB_mkl = params.tma_load_b.get_tma_tensor(make_shape(L,N,C,G*B));
  Tensor gB_mkl = local_tile(mB_mkl, TileShapeIntraBMM1{}, make_coord(_,_,_), Step<_1, X,_1>{});
  return make_tuple(gB_mkl);
}

void load_b_c(..., MainloopPipelineB& pipeline_b, ..., MainloopPipelineC& pipeline_c, ... ) {
  if (cute::elect_one_sync()) {
    for (int chunk_idx = 0; chunk_idx < chunk; ++chunk_idx) {
      pipeline_b.producer_acquire(pipeline_state_b);
      copy(params.tma_load_b.with(*tma_barrier_b, mcast_mask_b), tBgB(_,_,_,chunk_idx), tBsB(_,_,_,write_stage_b));
      ++pipeline_state_b;

      pipeline_c.producer_acquire(pipeline_state_c);
      copy(params.tma_load_c.with(*tma_barrier_c, mcast_mask_c), tCgC(_,_,_,chunk_idx), tCsC(_,_,_,write_stage_c));
      ++pipeline_state_c;
    }
  }
}
```
**EN**
- `B` and `C` are loaded together because both the intra and inter branches need them, and they advance chunk-for-chunk in lockstep.
- The code keeps separate pipelines for `B` and `C`, but one producer loop fills both so the stage indices remain aligned with chunk traversal.
- `load_c_init()` mirrors `load_b_init()`, and `load_b_c_tail()` closes both producer pipelines.

**CN**
- `B` 与 `C` 被放在一起加载，因为 intra 与 inter 两条路径都要用到它们，而且它们沿 chunk 维度同步推进。
- 代码虽然为 `B` 和 `C` 保持独立 pipeline，但由同一个 producer 循环同时填充，从而保证 stage 索引与 chunk 遍历保持一致。
- `load_c_init()` 与 `load_b_init()` 对称，而 `load_b_c_tail()` 负责关闭这两个 producer pipeline。

### Block 4 (lines 528-593) — Loading `Delta` and `DeltaA`
```cpp
void load_delta(..., MainloopPipeline& pipeline, PipelineState& pipeline_state,
                TensorStorage& shared_tensors) {
  if (cute::elect_one_sync()) {
    for (int chunk_idx = 0; chunk_idx < chunk; ++chunk_idx) {
      pipeline.producer_acquire(pipeline_state);
      auto* tma_barrier = pipeline.producer_get_barrier(pipeline_state);
      copy(bulk_atom_dt.with(*tma_barrier), gDelta(_,chunk_idx,blk_coord), sDelta(_,write_stage));
      copy(bulk_atom_dA.with(*tma_barrier), gDeltaA(_,chunk_idx,blk_coord), sDeltaA(_,write_stage));
      ++pipeline_state;
    }
  }
}
```
**EN**
- `Delta` and `DeltaA` encode the semiseparable recurrence scalings. They are staged into shared memory with bulk copy atoms, one stage per chunk.
- This data is later consumed twice: once for the intra segment-sum transform and once for the inter recurrence/state update.
- `load_delta_tail()` provides the same cluster-safe producer epilogue as the other loaders.

**CN**
- `Delta` 与 `DeltaA` 编码了 semiseparable 递归缩放因子。它们通过 bulk copy atom 分 stage 搬入共享内存，每个 chunk 对应一个 stage。
- 这些数据之后会被消费两次：一次用于 intra 的 segment-sum 变换，一次用于 inter 的递归/状态更新。
- `load_delta_tail()` 与其它加载函数一样，提供 cluster 安全的 producer epilogue。

### Block 5 (lines 595-656) — `mma_intra_1()`: the first within-chunk GEMM
```cpp
auto mma_intra_1(..., MainloopPipelineB& pipeline_b, ..., MainloopPipelineC& pipeline_c, ... ) {
  Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_b.data()), SmemLayoutB{});
  Tensor sC = make_tensor(make_smem_ptr(shared_tensors.smem_c.data()), SmemLayoutC{});
  ...
  pipeline_b.consumer_wait(pipeline_state_b, barrier_token_b);
  pipeline_c.consumer_wait(pipeline_state_c, barrier_token_c);

  warpgroup_fence_operand(accumulator);
  warpgroup_arrive();
  gemm_zero_acc(tiled_mma, tCrC(_,_,_,read_c_stage), tBrB(_,_,_,read_b_stage), accumulator);
  warpgroup_commit_batch();
  warpgroup_wait<0>();
```
**EN**
- `mma_intra_1()` computes the first local-chunk interaction matrix. Its output is an `L x L`-shaped accumulator that still lacks the SSD recurrence scaling.
- The function waits on both `B` and `C` pipelines, then launches a GMMA batch with warpgroup synchronization primitives.
- After the GMMA is complete, it releases the `B` and `C` stages so later chunks can reuse those buffers.

**CN**
- `mma_intra_1()` 计算第一个 chunk 内交互矩阵。它的输出是一个 `L x L` 形状的累加器，但此时还没有乘上 SSD 递归缩放因子。
- 该函数先等待 `B` 与 `C` 两条 pipeline，然后利用 warpgroup 同步原语发起一批 GMMA。
- GMMA 完成后，它释放 `B` 和 `C` 的 stage，供后续 chunk 复用这些缓冲。

### Block 6 (lines 658-734) — `pre_intra_2()`: explicit lower-triangular segment-sum transform
```cpp
for (int ii = 0; ii < size(tIntra1); ++ii) {
  ElementAcc tmp(0.f);
  auto [m,n] = tC(ii);
  if (m >= n) {
    tmp = expf(tSR_DeltaA_col(ii) - tSR_DeltaA_row(ii));
  }
  tIntra1(ii) = tmp * tSR_Delta(ii) * tIntra1(ii);
}
```
**EN**
- This is the heart of the semiseparable math for the intra path. The code explicitly imposes a lower-triangular mask (`m >= n`) and multiplies each retained entry by `exp(deltaA_col - deltaA_row) * delta`.
- In other words, GEMM only produces the dense interaction matrix; SSD causality and decay are injected here as pointwise logic.
- The named barrier (`TransformBarrier`) ensures shared-memory views used for row/column interpretations of `DeltaA` are seen consistently.

**CN**
- 这里是 intra 路径 semiseparable 数学的核心。代码显式施加下三角掩码（`m >= n`），并对保留下来的每个元素乘上 `exp(deltaA_col - deltaA_row) * delta`。
- 换句话说，GEMM 只负责生成稠密交互矩阵；真正的 SSD 因果性与衰减是在这里通过逐点逻辑注入的。
- 命名 barrier（`TransformBarrier`）保证 `DeltaA` 在行/列两种共享内存解释下都能被一致地观察到。

### Block 7 (lines 737-792) — `mma_intra_2()`: apply the transformed local matrix to `X`
```cpp
Tensor tCrIntra = make_acc_into_op<Element>(tIntra1, typename TiledMmaIntra2::LayoutA_TV{});
Tensor accumulator = partition_fragment_C(tiled_mma, take<0, 2>(TileShapeIntraBMM2{}));
...
pipeline_x.consumer_wait(pipeline_state_x, barrier_token);
warpgroup_fence_operand(tCrIntra);
warpgroup_fence_operand(accumulator);
gemm_zero_acc(tiled_mma, tCrIntra, tCrX(_,_,_,read_stage), accumulator);
warpgroup_commit_batch();
warpgroup_wait<0>();
pipeline_x.consumer_release(pipeline_state_x);
```
**EN**
- `mma_intra_2()` multiplies the transformed `L x L` intra matrix by `X` to produce the chunk-local `L x D` contribution.
- `make_acc_into_op()` from `common.hpp` is important here: it reshapes the accumulator layout so it can be consumed as a GMMA operand.
- The result of this stage is what Consumer0 later hands to the epilogue through `store_intra()`.

**CN**
- `mma_intra_2()` 把变换后的 `L x L` intra 矩阵与 `X` 相乘，得到 chunk 内局部贡献 `L x D`。
- 这里 `common.hpp` 的 `make_acc_into_op()` 很关键：它把累加器布局重排成可被 GMMA 当作操作数消费的形式。
- 这个阶段的结果就是随后 Consumer0 通过 `store_intra()` 交给 epilogue 的那份局部贡献。

### Block 8 (lines 794-910) — `pre_inter_1()`: chunk-summary preparation for the recurrent branch
```cpp
auto [tCrB_mma, last_column] = pre_inter_1(...)
...
if (last_thread_per_row) {
  auto tSR_DeltaA_slice = tSR_DeltaA(make_coord(coord0,_,coord1),_,coord2);
  last_column = tSR_DeltaA_slice(_0{});
}
last_column = __shfl_sync(0xFFFFFFFF, last_column, 3, 32);
for (int ii = 0; ii < size(tSR_DeltaA); ++ii) {
  tSR_DeltaA(ii) = expf(last_column - tSR_DeltaA(ii));
}
for (int ii = 0; ii < size(tCompute); ++ii) {
  tCompute(ii) = tCrB_load(ii) * (tSR_DeltaA(ii) * tSR_Delta(ii));
}
copy(tCompute, tCrB_mma);
```
**EN**
- `pre_inter_1()` prepares the recurrent branch by scaling `B` with `Delta` and with a decay measured relative to the last column of `DeltaA` in the chunk.
- Extracting `last_column` is the chunk-boundary summary: it captures the chunk's terminal exponential state and is later reused to update the carry state.
- The result is a preconditioned `B` operand ready for the first inter GEMM.

**CN**
- `pre_inter_1()` 通过 `Delta` 和相对于本 chunk 最后一列 `DeltaA` 的衰减来缩放 `B`，为递归路径做预处理。
- 提取 `last_column` 的意义在于构造 chunk 边界摘要：它表示该 chunk 的末端指数状态，稍后会被拿来更新跨 chunk 的 carry state。
- 最终输出是一个已经预调制好的 `B` 操作数，可直接送入第一个 inter GEMM。

### Block 9 (lines 912-948) — `mma_inter_1()`: build the next state contribution from `X`
```cpp
auto mma_inter_1(..., TensorB tCrB, TensorStorage& shared_tensors) {
  Tensor tCsX = thr_mma.partition_B(sX);
  Tensor tCrX = thr_mma.make_fragment_B(tCsX);
  Tensor accumulator = partition_fragment_C(tiled_mma, take<0, 2>(TileShapeInterBMM1{}));
  ...
  gemm_zero_acc(tiled_mma, tCrB, tCrX(_,_,_,read_stage), accumulator);
}
```
**EN**
- This stage multiplies the preconditioned recurrent operand by `X` and produces the chunk contribution that will update the recurrent state.
- The `X` stage is not released here; later code intentionally delays release because both intra and inter consumers share the same `X` pipeline timing.

**CN**
- 这一阶段把已经预处理好的递归操作数与 `X` 相乘，得到用于更新递归状态的 chunk 贡献。
- 这里不会立刻释放 `X` 的 stage；后面的代码会有意延迟释放，因为 intra 与 inter 两个消费者共享 `X` pipeline 的时序。

### Block 10 (lines 950-1042) — State initialization, recurrence update, and shared-memory export of `P`
```cpp
auto state_init(TensorStorage& shared_tensors) {
  Tensor tensor_state = partition_fragment_C(tiled_mma, take<0, 2>(TileShapeInterBMM1{}));
  clear(tRS_rP);
  clear(tensor_state);
  copy(tiled_r2s, tRS_rP, tRS_sP);
  cutlass::arch::fence_view_async_shared();
  synchronize();
  return make_tuple(tensor_state);
}

void pre_inter_2(ElementAcc& last_column, TensorInter1& tInter1, TensorState& tState) {
  for (int ii = 0; ii < size(tInter1); ++ii) {
    tInter1(ii) = tInter1(ii) + expf(last_column) * static_cast<ElementAcc>(tState(ii));
  }
  copy(tInter1, tState);
}
```
**EN**
- `state_init()` zeros both the register resident state and the shared-memory `P` tile. The shared tile is stored in transposed form because the second inter GEMM wants `P` in its `B` operand layout.
- `pre_inter_2()` is the actual chunk recurrence: `new_state = inter1 + exp(last_column) * old_state`.
- `post_inter_2()` converts the updated state back to element precision and writes it to `smem_p`, making it available both for the next chunk's `mma_inter_2()` and for the final epilogue `store_p()`.

**CN**
- `state_init()` 同时清零寄存器态和共享内存里的 `P` tile。共享内存中的 `P` 以转置形式存储，因为第二个 inter GEMM 需要把 `P` 当作其 `B` 操作数布局来读。
- `pre_inter_2()` 就是真正的 chunk 递归：`new_state = inter1 + exp(last_column) * old_state`。
- `post_inter_2()` 会把更新后的状态转换回元素精度并写入 `smem_p`，既为下一个 chunk 的 `mma_inter_2()` 提供输入，也为最终 epilogue 的 `store_p()` 做准备。

### Block 11 (lines 1044-1122) — `mma_inter_2()`: reconstruct output from `C` and state `P`
```cpp
auto mma_inter_2(..., MainloopPipelineC& pipeline_c, ..., MainloopPipelineDelta& pipeline_delta, ... ) {
  Tensor sP = make_tensor(make_smem_ptr(shared_tensors.smem_p.data()), SmemLayoutP{});
  Tensor sC = make_tensor(make_smem_ptr(shared_tensors.smem_c.data()), SmemLayoutC{});
  ...
  gemm_zero_acc(tiled_mma, tCrC(_,_,_,read_c_stage), tPrP, accumulator);
  ...
  auto tDeltaA_col_ = make_tensor(make_smem_ptr(shared_tensors.smem_delta_a.data()), delta_a_col_tv_layout)
                        (make_coord(thread_idx,_), make_coord(_,_,read_delta_stage));
  copy(tDeltaA_col, tSR_DeltaA_col);
  for (int ii = 0; ii < size(tSR_DeltaA_col); ++ii) {
    tSR_DeltaA_col(ii) = expf(tSR_DeltaA_col(ii));
  }
  return make_tuple(accumulator, tSR_DeltaA_col);
}
```
**EN**
- `mma_inter_2()` multiplies `C` by the current recurrent state `P` to reconstruct the cross-chunk contribution to the output tile.
- It also loads the column-oriented `DeltaA` slice for the same chunk and exponentiates it. That factor is not applied here; it is returned to the epilogue, which multiplies it with the inter result during final output assembly.
- This separation is important: the mainloop produces reusable mathematical pieces, while the epilogue decides how they combine with intra, `D`, and `Z`.

**CN**
- `mma_inter_2()` 把 `C` 与当前递归状态 `P` 相乘，重建该输出 tile 的跨 chunk 贡献。
- 同时它还会读取同一 chunk 的列方向 `DeltaA` 切片并做指数变换。这里并不直接应用这个因子，而是把它返回给 epilogue，由 epilogue 在最终输出组合时与 inter 结果相乘。
- 这种拆分非常关键：mainloop 负责产生可复用的数学部件，而 epilogue 决定它们如何与 intra、`D`、`Z` 组合。

### Block 12 (lines 1124-1143) — Precision conversion helper
```cpp
template<class ElementSrc, class ElementDst, class TensorSrc, class TensorDst>
CUTLASS_DEVICE auto type_convert(TensorSrc& tS, TensorDst& tD) {
  NumericArrayConverter<ElementDst, ElementSrc, 2> converter;
  auto tS_frg = recast<Array<ElementSrc, 2>>(tS);
  auto tD_frg = recast<Array<ElementDst, 2>>(tD);
  for (int ii = 0; ii < size(tS_frg); ++ii) {
    tD_frg(ii) = converter(tS_frg(ii));
  }
}
```
**EN**
- The helper is simple, but it is used everywhere that SSD data crosses precision domains: shared-memory loads, GMMA operands, recurrent state stores, and epilogue handoff.
- In this example, precise placement of conversion matters because the algorithm mixes storage-friendly element types with accumulator-friendly compute types.

**CN**
- 这个辅助函数虽然简单，但贯穿了所有精度域切换：共享内存加载、GMMA 操作数、递归状态写回，以及与 epilogue 的交接。
- 在这个示例里，转换位置很关键，因为算法同时混合了适合存储的元素类型与适合计算的累加类型。

---

## Key Concepts / 关键概念

1. **Four-GEMM decomposition of SSD**
   - **EN:** `IntraBMM1/2` build the lower-triangular within-chunk contribution; `InterBMM1/2` propagate state across chunks and reconstruct the chunk output from the state.
   - **CN:** `IntraBMM1/2` 负责构造 chunk 内下三角贡献；`InterBMM1/2` 负责跨 chunk 状态传播，并从状态重建当前 chunk 输出。
2. **TMA-fed warp-specialized pipelines**
   - **EN:** producer warps stream `X`, `B`, `C`, `Delta`, and `DeltaA` into staged shared memory; consumer warpgroups wait on those pipelines and launch GMMA.
   - **CN:** producer warps 把 `X`、`B`、`C`、`Delta`、`DeltaA` 按 stage 流式搬入共享内存；consumer warpgroups 在相应 pipeline 上等待，然后启动 GMMA。
3. **Semiseparable math encoded as pointwise transforms**
   - **EN:** the segment-sum / exponential recurrence terms are not hidden in GEMM. They are inserted explicitly in `pre_intra_2()`, `pre_inter_1()`, and `pre_inter_2()`.
   - **CN:** segment-sum / 指数递归项并没有被“藏”进 GEMM，而是在 `pre_intra_2()`、`pre_inter_1()`、`pre_inter_2()` 中显式插入。
4. **State tensor `P` as the inter-chunk carrier**
   - **EN:** `P` is the compact recurrent state shared across chunks. It is initialized to zero, updated after each chunk, stored in shared memory in transposed form for GMMA, and finally exported by the epilogue.
   - **CN:** `P` 是跨 chunk 传递的紧凑递归状态。它先被清零初始化，在每个 chunk 后更新，以转置形式保存在共享内存中供 GMMA 使用，并最终由 epilogue 写回全局内存。

## Dependencies / 依赖项

- `cutlass/gemm/collective/collective_builder.hpp`
  - **EN:** used to instantiate the four SM90 GEMM collectives and extract their tiling/smem layouts.
  - **CN:** 用于实例化四个 SM90 GEMM collective，并提取它们的 tiling / 共享内存布局。
- `examples/111_hopper_ssd/collective/common.hpp`
  - **EN:** supplies helper utilities such as `gemm_zero_acc()`, `convert_to_gmma_rs()`, and `make_acc_into_op()` that adapt CUTLASS/CuTe fragments to the SSD flow.
  - **CN:** 提供 `gemm_zero_acc()`、`convert_to_gmma_rs()`、`make_acc_into_op()` 等辅助工具，把 CUTLASS/CuTe fragment 适配到 SSD 数据流。
- `examples/111_hopper_ssd/kernel/sm90_ssd_kernel_tma_warpspecialized.hpp`
  - **EN:** assigns producer warps for `X`, `B/C`, `Delta`, and consumer warpgroups for the intra/inter branches that call the functions in this file.
  - **CN:** 负责给 `X`、`B/C`、`Delta` 分配 producer warp，并给 intra/inter 分支分配 consumer warpgroup，实际调用本文件中的函数。
- `examples/111_hopper_ssd/collective/sm90_ssd_epilogue.hpp`
  - **EN:** consumes the tensors produced here: intra results, inter results, `DeltaA` column factors, `D`, and the final state `P`.
  - **CN:** 消费本文件产生的张量：intra 结果、inter 结果、`DeltaA` 列因子、`D`，以及最终状态 `P`。
