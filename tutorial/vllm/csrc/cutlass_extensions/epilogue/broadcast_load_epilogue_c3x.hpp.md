# broadcast_load_epilogue_c3x.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cutlass_extensions/epilogue/broadcast_load_epilogue_c3x.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Extends CUTLASS SM90 epilogue visitors so row, column, or scalar broadcast values can always be read from device memory during quantized epilogues. / [CN] 扩展 CUTLASS 的 SM90 epilogue visitor，使行广播、列广播或标量广播都能在量化 epilogue 中统一从设备内存读取。

## Line-by-Line Analysis / 逐行分析

### Modified upstream contract
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
**EN:** The file-level note explains the main vLLM change versus upstream CUTLASS: scalar and broadcast tensors are passed as device pointers instead of host-side scalar values. That lets one compiled kernel support per-tensor and per-channel/per-token quantization without graph-breaking device-to-host moves.
**CN:** 文件顶部说明了相对上游 CUTLASS 的核心改动：标量与广播张量不再通过主机侧标量传入，而是统一使用设备指针。这让同一个已编译内核同时支持 per-tensor 与 per-channel/per-token 量化，并避免设备到主机的数据搬运造成图中断。

### Row/scalar visitor configuration
```cpp
// Row vector broadcast
template<
  int Stages,
  class CtaTileShapeMNK,
  class Element,
  class StrideMNL = Stride<_0,_1,_0>,
  int Alignment = 128 / sizeof_bits_v<Element>
>
struct Sm90RowOrScalarBroadcast {
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
    Element const* ptr_row = nullptr;
    bool row_broadcast = true;
    StrideMNL dRow = {};
  };
```
**EN:** `Sm90RowOrScalarBroadcast` declares the layout constraints for row-wise broadcasting, reserves shared memory for one CTA tile row slice, and stores runtime arguments as `(ptr_row, row_broadcast, dRow)`. The boolean explicitly tells the visitor whether the pointer references a full row tensor or a single scalar value.
**CN:** `Sm90RowOrScalarBroadcast` 定义了行广播的布局约束，为一个 CTA tile 的行片段预留共享内存，并将运行时参数组织为 `(ptr_row, row_broadcast, dRow)`。其中布尔值明确区分该指针指向整行张量还是单个标量。

### Row callback load path
```cpp
    CUTLASS_DEVICE void
    begin() {
      if (!params.row_broadcast) {
        fill(tSR_rRow, *(params.ptr_row));
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
**EN:** `begin()` handles the two execution modes. In scalar mode it fills the register fragment directly from `*ptr_row`; in row-broadcast mode it copies the needed tile from global memory to shared memory, zero-fills out-of-bounds elements, and synchronizes threads with the epilogue named barrier. `begin_loop()` then issues the shared-memory-to-register copy once per M-major subtile.
**CN:** `begin()` 处理两种执行模式：标量模式下直接用 `*ptr_row` 填充寄存器片段；行广播模式下则把需要的 tile 从全局内存搬到共享内存，对越界元素补零，并通过 epilogue 命名屏障同步线程。随后 `begin_loop()` 在每个 M-major 子块上执行一次共享内存到寄存器的加载。

### Row callback construction
```cpp
    cute::Tensor mRow = make_tensor(make_gmem_ptr(params.ptr_row), make_shape(M,N,L), params.dRow);
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
      params);
```
**EN:** `get_consumer_store_callbacks()` converts the raw device pointer into CUTE tensors, tiles the row data to CTA scope, builds both global-to-shared and shared-to-register partitions, and returns a callback object capturing those views. This is the bridge between CUTLASS epilogue tiling metadata and the custom broadcast logic.
**CN:** `get_consumer_store_callbacks()` 会把原始设备指针包装成 CUTE 张量，将行数据裁剪到 CTA 作用域，构造全局到共享内存以及共享到寄存器两级分块视图，并返回持有这些视图的回调对象。这一步是 CUTLASS epilogue 分块元数据与自定义广播逻辑之间的连接点。

### Column/scalar visitor path
```cpp
    CUTLASS_DEVICE void
    begin() {
      if (!params.col_broadcast) {
        fill(tCrCol, *(params.ptr_col));
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

    template <typename ElementAccumulator, int FragmentSize>
    CUTLASS_DEVICE Array<Element, FragmentSize>
    visit(Array<ElementAccumulator, FragmentSize> const& frg_acc, int epi_v, int epi_m, int epi_n) {
      Array<Element, FragmentSize> frg_col;
      cute::Tensor tCrCol_mn = tCrCol(_,_,_,epi_m,epi_n);

      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < FragmentSize; ++i) {
        frg_col[i] = tCrCol_mn(epi_v * FragmentSize + i);
      }

      return frg_col;
```
**EN:** The column variant skips shared memory and relies on the accumulator layout already distributing column values across threads. `begin()` slices away stride-0 N dimensions to avoid redundant loads, builds a predicate from identity coordinates to guard the M dimension, and `visit()` reads the prepared register fragment for each accumulator fragment.
**CN:** 列广播版本不使用共享内存，而是利用累加器布局已经按线程分发列元素这一事实。`begin()` 会切掉 stride-0 的 N 维以避免重复加载，再根据 identity 坐标构造谓词来保护 M 维边界；`visit()` 则在每个累加器片段访问时读取已准备好的寄存器片段。

## Key Concepts / 关键概念
- EN: Device-resident scalar/broadcast parameters keep quantization metadata on GPU and avoid `torch.compile` graph breaks.
  CN: 设备驻留的标量/广播参数让量化元数据始终留在 GPU 上，并避免 `torch.compile` 图中断。
- EN: Row broadcast uses shared memory because many threads reuse the same row slice inside the epilogue.
  CN: 行广播使用共享内存，因为 epilogue 内多个线程会重复使用同一行片段。
- EN: Column broadcast can load directly to registers because the tiled epilogue layout already matches the needed access pattern.
  CN: 列广播可直接加载到寄存器，因为分块后的 epilogue 布局本身就匹配所需访问模式。

## Dependencies / 依赖关系
- EN: Depends on CUTLASS epilogue visitor interfaces such as `ProducerLoadArgs`, `ConsumerStoreArgs`, and `EmptyConsumerStoreCallbacks`.
  CN: 依赖 CUTLASS 的 epilogue visitor 接口，例如 `ProducerLoadArgs`、`ConsumerStoreArgs` 与 `EmptyConsumerStoreCallbacks`。
- EN: Uses CUTE tensor builders like `make_tensor`, `local_tile`, `make_identity_tensor`, and `sm90_partition_for_epilogue`.
  CN: 使用 CUTE 的张量构造工具，如 `make_tensor`、`local_tile`、`make_identity_tensor` 和 `sm90_partition_for_epilogue`。
- EN: Synchronizes through `cutlass::arch::NamedBarrier` reserved for the epilogue pipeline.
  CN: 通过 `cutlass::arch::NamedBarrier` 中为 epilogue 预留的屏障进行同步。
