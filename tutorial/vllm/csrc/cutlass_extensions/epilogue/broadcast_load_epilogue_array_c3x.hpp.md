# broadcast_load_epilogue_array_c3x.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cutlass_extensions/epilogue/broadcast_load_epilogue_array_c3x.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Adds the same SM90 broadcast epilogue visitors as the pointer version, but the source tensors are selected from per-group pointer arrays so each batch/group can use different scale or bias buffers. / [CN] 提供与单指针版本相同的 SM90 广播 epilogue visitor，但广播源来自按组组织的指针数组，因此每个 batch/group 都可选择不同的 scale 或 bias 缓冲区。

## Line-by-Line Analysis / 逐行分析

### Array-based upstream change
```cpp
// This file is a modified excerpt of
// include/cutlass/epilogue/fusion/sm90_visitor_load_tma_warpspecialized.hpp
// from https://github.com/NVIDIA/cutlass v3.5.0
// It has been modified to support either row/column or scalar broadcasting
// where the tensor being loaded from is always passed in via a device pointer.
// This lets one compiled kernel handle all cases of per-tensor or
// per-channel/per-token quantization.
//
// This interface also allows the scales to be passed in as tensors that
// consistently reside on the device, which avoids an issue with a previous
// implementation where scalars needed to be on the CPU since they
// were passed in via float values. This created a potential performance hazard
// if scales were initially on the device, and caused torch.compile graphs
// breaks when moving scales to the CPU.
```
**EN:** The header keeps the same design goal as the non-array version—device-resident broadcast/scalar inputs—but extends it to pointer arrays so runtime code can switch buffers per group without recompiling the GEMM epilogue.
**CN:** 该头文件与非数组版本有相同设计目标：让广播/标量输入驻留在设备上；同时进一步扩展为指针数组，从而允许运行时按 group 切换缓冲区，而无需重新编译 GEMM epilogue。

### Row array arguments
```cpp
// Row vector broadcast
template<
  int Stages,
  class CtaTileShapeMNK,
  class Element,
  class StrideMNL = Stride<_0,_1,_0>,
  int Alignment = 128 / sizeof_bits_v<Element>
>
struct Sm90RowOrScalarBroadcastArray {
  static_assert(Stages == 0, "Row broadcast doesn't support smem usage");
  static_assert(is_static_v<decltype(take<0,2>(StrideMNL{}))>); // batch stride can be dynamic or static
  static_assert(take<0,2>(StrideMNL{}) == Stride<_0,_1>{});

  struct SharedStorage { 
    array_aligned<Element, size<1>(CtaTileShapeMNK{})> smem;
  };

  // This struct has been modified to have a bool indicating that ptr_row is a 
  // scalar that must be broadcast, instead of containing a scalar that is 
  // valid if ptr_row is null.
  struct Arguments {
    const Element* const* ptr_row_array = nullptr;
    bool row_broadcast = true;
    StrideMNL dRow = {};
  };
```
**EN:** `Sm90RowOrScalarBroadcastArray` replaces `ptr_row` with `ptr_row_array`, i.e. an array of device pointers. The rest of the contract stays the same: row mode uses a tensor layout, scalar mode dereferences one element from the selected pointer.
**CN:** `Sm90RowOrScalarBroadcastArray` 将 `ptr_row` 替换为 `ptr_row_array`，即设备指针数组。其余约定基本不变：行模式按张量布局读取，标量模式则从选中的指针中解引用一个元素。

### Per-group row loading
```cpp
    CUTLASS_DEVICE void
    begin() {
      if (!params.row_broadcast) {
        fill(tSR_rRow, *(params.ptr_row_array[group]));
        return;
      }

      auto synchronize = [&] () { cutlass::arch::NamedBarrier::sync(thr_num, cutlass::arch::ReservedNamedBarriers::EpilogueBarrier); };
      cute::Tensor tGS_gRow_flt = filter_zeros(tGS_gRow);
      cute::Tensor tGS_sRow_flt = filter_zeros(tGS_sRow);
      cute::Tensor tGS_cRow_flt = make_tensor(tGS_cRow.data(), make_layout(tGS_gRow_flt.shape(), tGS_cRow.stride()));

      for (int i = 0; i < size(tGS_gRow_flt); ++i) {
        if (get<1>(tGS_cRow_flt(i)) >= size<1>(CtaTileShapeMNK{})) {
          continue; // OOB of SMEM, 
        }
        if (elem_less(tGS_cRow_flt(i), make_coord(get<0>(residue_tCcRow), get<1>(residue_tCcRow)))) {
          tGS_sRow_flt(i) = tGS_gRow_flt(i);
        }
        else {
          tGS_sRow_flt(i) = Element(0); // Set to Zero when OOB so LDS could be issue without any preds.
        }
      }
      synchronize();
    }

    CUTLASS_DEVICE void
    begin_loop(int epi_m, int epi_n) {
      if (epi_m == 0) { // Assumes M-major subtile loop
        if (!params.row_broadcast) return; // Do not issue LDS when row is scalar 
        cute::Tensor tSR_sRow_flt = filter_zeros(tSR_sRow(_,_,_,epi_m,epi_n));
        cute::Tensor tSR_rRow_flt = filter_zeros(tSR_rRow);
        copy(tSR_sRow_flt, tSR_rRow_flt);
      }
```
**EN:** The row callback mirrors the non-array implementation, except scalar fallback and global loads use `ptr_row_array[group]`. This makes the broadcast source depend on the current logical group while preserving the same shared-memory staging and OOB masking strategy.
**CN:** 行回调与非数组实现基本一致，只是标量回退与全局内存加载都改为访问 `ptr_row_array[group]`。这样广播源就能随当前逻辑 group 变化，同时保留原有的共享内存暂存与越界掩码策略。

### Binding row callbacks to batch/group index
```cpp
  get_consumer_store_callbacks(ConsumerStoreArgs<Args...> const& args) {
    auto [M, N, K, L] = args.problem_shape_mnkl;
    auto [m, n, k, l] = args.tile_coord_mnkl;
    using ThreadCount = decltype(size(args.tiled_copy));

    cute::Tensor mRow = make_tensor(make_gmem_ptr(params.ptr_row_array[l]), make_shape(M,N,1), params.dRow);
    cute::Tensor gRow = local_tile(mRow(_,_,l), take<0,2>(args.tile_shape_mnk), make_coord(m, n));          // (CTA_M, CTA_N)
    cute::Tensor sRow = make_tensor(make_smem_ptr(smem), 
        make_shape(size<0>(CtaTileShapeMNK{}), size<1>(CtaTileShapeMNK{})), make_shape(_0{}, _1{}));  // (CTA_M, CTA_N)
    //// G2S: Gmem to Smem
    auto tiled_g2s = make_tiled_copy(Copy_Atom<DefaultCopy, Element>{},
                                     Layout< Shape<_1, ThreadCount>, 
                                            Stride<_0,          _1>>{}, 
                                     Layout<_1>{});   
    auto thr_g2s = tiled_g2s.get_slice(args.thread_idx);
    cute::Tensor tGS_gRow = thr_g2s.partition_S(gRow);
    cute::Tensor tGS_sRow = thr_g2s.partition_D(sRow);

    //// G2S: Coord 
    auto cRow = make_identity_tensor(make_shape(size<0>(CtaTileShapeMNK{}), size<1>(CtaTileShapeMNK{})));
    cute::Tensor tGS_cRow = thr_g2s.partition_S(cRow);

    //// S2R: Smem to Reg
    cute::Tensor tSR_sRow = sm90_partition_for_epilogue<ReferenceSrc>(sRow, args.epi_tile, args.tiled_copy, args.thread_idx);
    cute::Tensor tSR_rRow = make_tensor_like(take<0,3>(tSR_sRow));                                           // (CPY,CPY_M,CPY_N)

    return ConsumerStoreCallbacks<decltype(tGS_gRow), decltype(tGS_sRow), decltype(tGS_cRow), decltype(tiled_g2s), decltype(tSR_sRow), decltype(tSR_rRow), decltype(args.tCcD), decltype(args.residue_cD), ThreadCount>(
      tGS_gRow, 
      tGS_sRow, 
      tGS_cRow, tiled_g2s, 
      tSR_sRow, 
      tSR_rRow, 
      args.tCcD, 
      args.residue_cD,
      ThreadCount{}, 
      l,
      params);
```
**EN:** The builder selects the pointer for the current `l` tile coordinate, constructs the tiled views from `params.ptr_row_array[l]`, and stores `l` inside the callback as `group`. That is the key adaptation that turns the visitor into a batched/per-group broadcast loader.
**CN:** 构造函数根据当前 tile 坐标 `l` 选择指针，从 `params.ptr_row_array[l]` 构造分块视图，并把 `l` 作为 `group` 保存在回调对象中。这一步是该 visitor 能支持批次/分组广播加载的关键。

### Per-group column loading
```cpp
    CUTLASS_DEVICE void
    begin() {
      if (!params.col_broadcast) {
        fill(tCrCol, *(params.ptr_col_array[group]));
        return;
      }

      // tCgCol has layout (CPY,CPY_M,CPY_N,EPI_M,EPI_N) where CPY_N and
      // EPI_N are stride-0 for the column broadcast. Slice those modes at
      // index 0 to avoid redundant copies AND ensure pred/data consistency
      static_assert(decltype(stride<2>(tCgCol))::value == 0, "Expected stride-0 CPY_N for col broadcast");
      static_assert(decltype(stride<4>(tCgCol))::value == 0, "Expected stride-0 EPI_N for col broadcast");

      auto tCgCol_s = tCgCol(_,_,0,_,0);      // (CPY,CPY_M,EPI_M)
      auto tCrCol_s = tCrCol(_,_,0,_,0);      // (CPY,CPY_M,EPI_M)
      auto tCcCol_s = tCcCol(_,_,0,_,0);      // (CPY,CPY_M,EPI_M)

      cute::Tensor pred = make_tensor<bool>(shape(tCgCol_s));
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(pred); ++i) {
        pred(i) = get<0>(tCcCol_s(i)) < m;
      }

      copy_if(pred, tCgCol_s, tCrCol_s);
    }
```
**EN:** For column broadcasts, `begin()` reads either a scalar from `ptr_col_array[group]` or slices the partitioned tensor to eliminate redundant stride-0 dimensions before `copy_if`. The dataflow is the same as the single-pointer version, only the source pointer becomes group-specific.
**CN:** 在列广播路径中，`begin()` 要么从 `ptr_col_array[group]` 读取一个标量，要么先对分块张量切片去掉多余的 stride-0 维，再执行 `copy_if`。数据流与单指针版本一致，只是源指针变成了 group 相关。

### Column callback binding
```cpp
  get_consumer_store_callbacks(ConsumerStoreArgs<Args...> const& args) {

    auto [M, N, K, L] = args.problem_shape_mnkl;
    auto [m, n, k, l] = args.tile_coord_mnkl;

    cute::Tensor mCol = make_tensor(make_gmem_ptr(params.ptr_col_array[l]), make_shape(M,N,1), params.dCol);
    cute::Tensor tCgCol = sm90_partition_for_epilogue<ReferenceSrc>(                         // (CPY,CPY_M,CPY_N,EPI_M,EPI_N)
      mCol, args.tile_shape_mnk, args.tile_coord_mnkl, args.epi_tile, args.tiled_copy, args.thread_idx);
    cute::Tensor tCrCol = make_tensor_like(tCgCol);                                          // (CPY,CPY_M,CPY_N,EPI_M,EPI_N)

    // Generate an identity tensor matching the shape of the global tensor and 
    //  partition the same way, this will be used to generate the predicate
    //  tensor for loading
    cute::Tensor cCol = make_identity_tensor(mCol.shape());
    cute::Tensor tCcCol = sm90_partition_for_epilogue<ReferenceSrc>(                         // (CPY,CPY_M,CPY_N,EPI_M,EPI_N)
      cCol, args.tile_shape_mnk, args.tile_coord_mnkl, args.epi_tile, args.tiled_copy, args.thread_idx);

    return ConsumerStoreCallbacks(
      cute::move(tCgCol), 
      cute::move(tCrCol), 
      cute::move(tCcCol), 
      args.problem_shape_mnkl,
      l,
      params
```
**EN:** The callback builder uses `params.ptr_col_array[l]` when creating the global tensor and passes `l` to the callback constructor. This keeps the CUTLASS epilogue interface unchanged while letting each batch/group pick its own column broadcast buffer.
**CN:** 该回调构造过程在创建全局张量时使用 `params.ptr_col_array[l]`，并把 `l` 传给回调构造函数。这样既保持 CUTLASS epilogue 接口不变，又允许每个 batch/group 使用各自的列广播缓冲区。

## Key Concepts / 关键概念
- EN: Pointer arrays let batched quantization kernels reuse one epilogue implementation across many groups.
  CN: 指针数组让批量量化内核能够在多个 group 上复用同一个 epilogue 实现。
- EN: The logical group is derived from the `l` tile coordinate and threaded through callback construction.
  CN: 逻辑 group 由 tile 坐标 `l` 推导出来，并在回调构造过程中一路传递。
- EN: The memory movement strategy is unchanged from the single-pointer variant; only source selection changes.
  CN: 其内存搬运策略与单指针版本一致，变化只在于源地址的选择。

## Dependencies / 依赖关系
- EN: Shares the same CUTLASS/CUTE epilogue visitor dependencies as `broadcast_load_epilogue_c3x.hpp`.
  CN: 与 `broadcast_load_epilogue_c3x.hpp` 共享相同的 CUTLASS/CUTE epilogue visitor 依赖。
- EN: Relies on runtime code to populate the pointer arrays with valid device addresses for each batch/group.
  CN: 依赖运行时代码为每个 batch/group 填充有效的设备地址数组。
- EN: Uses `make_gmem_ptr`, shared memory tensors, and `copy_if` exactly like the non-array implementation.
  CN: 与非数组实现一样使用 `make_gmem_ptr`、共享内存张量和 `copy_if`。
