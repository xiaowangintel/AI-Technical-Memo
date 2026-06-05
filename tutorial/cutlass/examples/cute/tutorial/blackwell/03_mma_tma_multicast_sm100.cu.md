# 03_mma_tma_multicast_sm100.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/cute/tutorial/blackwell/03_mma_tma_multicast_sm100.cu`

**Purpose / 用途**: Extends Tutorial 02 with TMA **multicast**: a 4×4 cluster of CTAs cooperatively loads A (shared across the N-axis) and B (shared across the M-axis), reducing GMEM bandwidth by up to 4× in each dimension. / 在教程 02 基础上增加 TMA **多播**：4×4 的 CTA 簇协作加载 A（沿 N 轴共享）和 B（沿 M 轴共享），在每个维度上最多减少 4× 的 GMEM 带宽消耗。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1–68: License & includes (identical to Tutorial 02) / 许可证与头文件（与教程 02 相同）

```cpp
// Same includes as Tutorial 02
```

**EN**: Identical to Tutorial 02. No new headers are required — `SM90_TMA_LOAD_MULTICAST` and `create_tma_multicast_mask` are already part of `<cute/tensor.hpp>`.  
**CN**: 与教程 02 相同。无需新头文件——`SM90_TMA_LOAD_MULTICAST` 和 `create_tma_multicast_mask` 已经是 `<cute/tensor.hpp>` 的一部分。

---

### Lines 72–117: Tutorial 03 overview comment / 教程 03 概述注释

```cpp
// Key extensions from Tutorial 02:
// 1. Introduce ClusterShape (4x4x1) for coordinated execution
// 2. Introduce TMA multicast
// 3. Enhanced TMA <-> MMA synchronization for cluster-wide operations
//
// TMA multicast: A is broadcast to all CTAs with same M-coord (N-axis)
//                B is broadcast to all CTAs with same N-coord (M-axis)
```

**EN**: The three conceptual additions in Tutorial 03:  
1. **ClusterShape `(4,4,1)`** — 16 CTAs form a cluster. CTAs at the same M-row share the same A tile; CTAs at the same N-column share the same B tile.  
2. **TMA multicast** — One TMA copy delivers data to **multiple CTAs' SMEM simultaneously** by setting a multicast bitmask. Each bit in the 16-bit mask corresponds to one CTA in the cluster.  
3. **Cluster-wide synchronization** — `cluster_sync()` replaces `__syncthreads()` for inter-CTA coordination; the `mma_barrier` is initialized with `num_ctas = M + N - 1` participants instead of 1.  

**CN**: 教程 03 的三个概念新增：  
1. **簇形状 `(4,4,1)`** — 16 个 CTA 组成一个簇。M 行相同的 CTA 共享同一个 A tile；N 列相同的 CTA 共享同一个 B tile。  
2. **TMA 多播** — 一次 TMA 拷贝通过设置多播位掩码**同时向多个 CTA 的 SMEM 传送数据**。16 位掩码中的每一位对应簇中的一个 CTA。  
3. **簇范围同步** — `cluster_sync()` 替换 `__syncthreads()` 用于 CTA 间协调；`mma_barrier` 以 `num_ctas = M + N - 1` 参与者初始化，而非 1。

---

### Lines 118–137: `SharedStorage` struct (identical to Tutorial 02) / 共享内存结构体（与教程 02 相同）

```cpp
struct SharedStorage {
  alignas(128) cute::ArrayEngine<TypeA, cute::cosize_v<ASmemLayout>> A;
  alignas(128) cute::ArrayEngine<TypeB, cute::cosize_v<BSmemLayout>> B;
  alignas(16) cute::uint64_t mma_barrier;
  alignas(16) cute::uint64_t tma_barrier;
  alignas(16) cute::uint32_t tmem_base_ptr;
};
```

**EN**: Structurally identical to Tutorial 02. The multicast changes do not require new SMEM fields — each CTA still has its own private SMEM buffer; multicast simply means multiple CTAs receive their own copy of the same data from a single TMA call.  
**CN**: 结构上与教程 02 相同。多播变化不需要新的 SMEM 字段——每个 CTA 仍然有自己的私有 SMEM 缓冲区；多播仅意味着多个 CTA 从单次 TMA 调用中各自接收同一数据的副本。

---

### Lines 139–156: Kernel signature (identical to Tutorial 02) / 内核签名（与教程 02 相同）

```cpp
template <class SharedStorage, ..., class TmaAtomA, class TmaAtomB, ...>
__global__ static void
gemm_device(...,
            CUTE_GRID_CONSTANT TmaAtomA const tma_atom_A,
            CUTE_GRID_CONSTANT TmaAtomB const tma_atom_B,
            Alpha alpha, Beta beta)
```

**EN**: Identical signature to Tutorial 02. The multicast semantics are encoded entirely inside `tma_atom_A` and `tma_atom_B` (created with `SM90_TMA_LOAD_MULTICAST` on the host); the kernel code that differs from Tutorial 02 is in the `tma_partition` call and the `copy` call.  
**CN**: 与教程 02 相同的签名。多播语义完全编码在 `tma_atom_A` 和 `tma_atom_B` 中（在主机端以 `SM90_TMA_LOAD_MULTICAST` 创建）；与教程 02 不同的内核代码在 `tma_partition` 调用和 `copy` 调用中。

---

### Lines 157–255: Prologue — coordinates, SMEM, TMEM (same as Tutorial 02) / 序言——坐标、SMEM、TMEM（与教程 02 相同）

```cpp
Layout cluster_layout_vmnk = tiled_divide(make_layout(cluster_shape),
                                          make_tile(typename TiledMMA::AtomThrID{}));
// With cluster_shape=(4,4,1): cluster_layout_vmnk has size<1>=4, size<2>=4

auto mma_coord_vmnk = make_coord(
    blockIdx.x % size<0>(cluster_layout_vmnk),
    blockIdx.x / size<0>(cluster_layout_vmnk),
    blockIdx.y, _);
```

**EN**: The cluster layout decomposition now yields a 4×4 grid of CTAs within the cluster. `size<1>(cluster_layout_vmnk) = 4` (4 CTAs along M) and `size<2>(cluster_layout_vmnk) = 4` (4 CTAs along N). Each CTA computes a 128×256 MMA tile, but the 16 CTAs together cover a 4×128 × 4×256 = 512×1024 output tile per cluster.  
**CN**: 簇布局分解现在产生簇内的 4×4 CTA 网格。`size<1>(cluster_layout_vmnk) = 4`（沿 M 方向 4 个 CTA），`size<2>(cluster_layout_vmnk) = 4`（沿 N 方向 4 个 CTA）。每个 CTA 计算一个 128×256 的 MMA tile，但 16 个 CTA 共同覆盖每个簇 4×128 × 4×256 = 512×1024 的输出 tile。

---

### Lines 257–305: TMA multicast partitioning (KEY CHANGE vs Tutorial 02) / TMA 多播划分（相对教程 02 的关键变化）

```cpp
auto cta_in_cluster_coord_vmnk =
    cluster_layout_vmnk.get_flat_coord(int(cute::block_rank_in_cluster()));
// e.g., for CTA 6 in a 4x4 cluster: coord = (0, 1, 2, _)  [V=0, M=1, N=2]

// A is multicast along N-axis (all CTAs with same M-coord receive same A tile)
auto [tAgA, tAsA] = tma_partition(
    tma_atom_A,
    get<2>(cta_in_cluster_coord_vmnk),          // This CTA's N-coordinate
    make_layout(size<2>(cluster_layout_vmnk)),  // N-axis layout (size=4)
    group_modes<0,3>(tCsA), group_modes<0,3>(tCgA));

// B is multicast along M-axis (all CTAs with same N-coord receive same B tile)
auto [tBgB, tBsB] = tma_partition(
    tma_atom_B,
    get<1>(cta_in_cluster_coord_vmnk),          // This CTA's M-coordinate
    make_layout(size<1>(cluster_layout_vmnk)),  // M-axis layout (size=4)
    group_modes<0,3>(tCsB), group_modes<0,3>(tCgB));
```

**EN**: This is the core multicast logic. The key insight is:  
- **A tile** depends only on M and K (not N). All 4 CTAs at the same M-row need the **same A tile**. By passing the CTA's N-coordinate and the N-axis layout to `tma_partition`, CuTe arranges for TMA to multicast A to all 4 N-axis CTAs simultaneously.  
- **B tile** depends only on N and K (not M). All 4 CTAs at the same N-column need the **same B tile**. B is multicast to all 4 M-axis CTAs.  
- The `tma_partition` arguments change from `(Int<0>{}, Layout<_1>{}, ...)` in Tutorial 02 to `(cta_n_coord, Layout<_4>{}, ...)` — this tells CuTe which axis to multicast across and how many CTAs participate.  

**CN**: 这是多播逻辑的核心。关键洞察是：  
- **A tile** 仅依赖于 M 和 K（不依赖 N）。同一 M 行的 4 个 CTA 需要**相同的 A tile**。通过将 CTA 的 N 坐标和 N 轴布局传递给 `tma_partition`，CuTe 安排 TMA 同时将 A 多播到所有 4 个 N 轴 CTA。  
- **B tile** 仅依赖于 N 和 K（不依赖 M）。同一 N 列的 4 个 CTA 需要**相同的 B tile**。B 被多播到所有 4 个 M 轴 CTA。  
- `tma_partition` 参数从教程 02 中的 `(Int<0>{}, Layout<_1>{}, ...)` 变为 `(cta_n_coord, Layout<_4>{}, ...)`——这告诉 CuTe 沿哪个轴进行多播以及有多少 CTA 参与。

---

### Lines 300–320: Multicast bitmask computation / 多播位掩码计算

```cpp
// Illustration of multicast masks for a 4x4 cluster (CTA 1,2 example):
//   A multicast (row 1 broadcasts to its 4 N-neighbors):
//    0  1  2  3
// 0  -  -  -  -
// 1  X  X  X  X   <- tma_mcast_mask_A = 0x2222 (col 1 of each row)
// 2  -  -  -  -
// 3  -  -  -  -
//
//   B multicast (col 2 broadcasts to its 4 M-neighbors):
//    0  1  2  3
// 0  -  -  X  -
// 1  -  -  X  -   <- tma_mcast_mask_B = 0x0F00 (row 2, all cols)
// 2  -  -  X  -
// 3  -  -  X  -

uint16_t tma_mcast_mask_a = create_tma_multicast_mask<2>(
    cluster_layout_vmnk, cta_in_cluster_coord_vmnk);
uint16_t tma_mcast_mask_b = create_tma_multicast_mask<1>(
    cluster_layout_vmnk, cta_in_cluster_coord_vmnk);
uint16_t mma_mcast_mask_c =
    create_tma_multicast_mask<0,1>(cluster_layout_vmnk, cta_in_cluster_coord_vmnk) |
    create_tma_multicast_mask<0,2>(cluster_layout_vmnk, cta_in_cluster_coord_vmnk);
```

**EN**: `create_tma_multicast_mask<Mode>(cluster_layout, cta_coord)` generates a 16-bit bitmask where bit `i` is set if CTA `i` in the cluster should receive the multicast. The template argument `<Mode>` specifies which mode of the cluster layout to project:  
- `<2>` (N-mode): All CTAs at the same M-coord → A multicast bitmask.  
- `<1>` (M-mode): All CTAs at the same N-coord → B multicast bitmask.  
- `<0,1>` and `<0,2>` combined: CTAs that participate in the MMA-result accumulation across both M and V-modes → MMA completion multicast mask for `umma_arrive_multicast`.  

The masks are CTA-specific — each CTA computes its own mask based on its cluster coordinates.

**CN**: `create_tma_multicast_mask<Mode>(cluster_layout, cta_coord)` 生成一个 16 位位掩码，其中位 `i` 置位表示簇中的 CTA `i` 应接收多播。模板参数 `<Mode>` 指定投影簇布局的哪个模式：  
- `<2>`（N 模式）：同一 M 坐标的所有 CTA → A 多播位掩码。  
- `<1>`（M 模式）：同一 N 坐标的所有 CTA → B 多播位掩码。  
- `<0,1>` 和 `<0,2>` 组合：跨 M 和 V 模式参与 MMA 结果累加的 CTA → `umma_arrive_multicast` 的 MMA 完成多播掩码。  

掩码是 CTA 特定的——每个 CTA 根据自己的簇坐标计算自己的掩码。

---

### Lines 322–333: Barrier initialization — cluster-aware / 屏障初始化——簇感知

```cpp
if (elect_one_warp && elect_one_thr) {
  int num_mcast_participants =
      size<1>(cluster_layout_vmnk) +   // M-participants for B multicast (4)
      size<2>(cluster_layout_vmnk) - 1; // N-participants for A multicast (4), minus 1 to avoid double-count
  // num_mcast_participants = 4 + 4 - 1 = 7

  cute::initialize_barrier(shared_storage.mma_barrier,
                           /* num_ctas */ num_mcast_participants);
  cute::initialize_barrier(shared_storage.tma_barrier,
                           /* num_threads */ 1);
}
cute::cluster_sync();  // cluster-wide sync instead of __syncthreads
```

**EN**: Critical change: `mma_barrier` is now initialized with `num_mcast_participants` (7 for a 4×4 cluster) instead of 1. This is because `umma_arrive_multicast` will signal the barrier from **multiple CTAs** (all CTAs that share the same A or B tile). The barrier must know how many arrival signals to expect before releasing. `cluster_sync()` is a cluster-wide synchronization primitive that ensures all 16 CTAs in the cluster observe the barrier initialization before proceeding.  
**CN**: 关键变化：`mma_barrier` 现在以 `num_mcast_participants`（4×4 簇为 7）而非 1 初始化。这是因为 `umma_arrive_multicast` 将从**多个 CTA**（所有共享相同 A 或 B tile 的 CTA）发出信号。屏障必须知道在释放前期望多少个到达信号。`cluster_sync()` 是确保簇中所有 16 个 CTA 在继续之前都观察到屏障初始化的簇范围同步原语。

---

### Lines 334–375: Mainloop — multicast TMA + multicast MMA arrive / 主循环——多播 TMA + 多播 MMA 到达

```cpp
tiled_mma.accumulate_ = UMMA::ScaleOut::Zero;

for (int k_tile = 0; k_tile < size<3>(tCgA); ++k_tile) {

  // Multicast TMA load (single thread, multicast bitmask applied)
  if (elect_one_warp && elect_one_thr) {
    cute::set_barrier_transaction_bytes(shared_storage.tma_barrier,
                                       tma_transaction_bytes);
    copy(tma_atom_A.with(shared_storage.tma_barrier, tma_mcast_mask_a),
         tAgA(_,k_tile), tAsA);   // A -> SMEM of all N-axis CTAs
    copy(tma_atom_B.with(shared_storage.tma_barrier, tma_mcast_mask_b),
         tBgB(_,k_tile), tBsB);   // B -> SMEM of all M-axis CTAs
  }

  cute::wait_barrier(shared_storage.tma_barrier, tma_barrier_phase_bit);
  tma_barrier_phase_bit ^= 1;

  if (elect_one_warp) {
    for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
      gemm(tiled_mma, tCrA(_,_,k_block), tCrB(_,_,k_block), tCtAcc);
      tiled_mma.accumulate_ = UMMA::ScaleOut::One;
    }
    // CHANGED: multicast arrive notifies all participating CTAs
    cutlass::arch::umma_arrive_multicast(&shared_storage.mma_barrier,
                                         mma_mcast_mask_c);
  }
  cute::wait_barrier(shared_storage.mma_barrier, mma_barrier_phase_bit);
  mma_barrier_phase_bit ^= 1;
}
```

**EN**: Two key differences from Tutorial 02:  
1. **`tma_atom_A.with(barrier, tma_mcast_mask_a)`** — The `.with()` now takes **two** arguments: the barrier AND the multicast bitmask. The TMA hardware delivers data to all CTAs whose bits are set in the mask.  
2. **`umma_arrive_multicast(..., mma_mcast_mask_c)`** — After MMA, instead of `umma_arrive` (which signals only the local barrier), `umma_arrive_multicast` signals the barriers of **all CTAs listed in the mask**. This allows the cluster-wide `mma_barrier` (expected 7 arrivals) to be satisfied correctly.  

**CN**: 与教程 02 的两个关键差异：  
1. **`tma_atom_A.with(barrier, tma_mcast_mask_a)`** — `.with()` 现在接受**两个**参数：屏障和多播位掩码。TMA 硬件将数据传递到掩码中位置位的所有 CTA。  
2. **`umma_arrive_multicast(..., mma_mcast_mask_c)`** — MMA 完成后，用 `umma_arrive_multicast` 替代 `umma_arrive`（后者只信号本地屏障），向**掩码中列出的所有 CTA** 的屏障发出信号。这使得簇范围的 `mma_barrier`（期望 7 次到达）能够正确满足。

---

### Lines 377–408: Epilogue (identical to Tutorial 02) / 尾声（与教程 02 相同）

```cpp
TiledCopy tiled_t2r_copy = make_tmem_copy(SM100_TMEM_LOAD_32dp32b1x{}, tCtAcc);
// TMEM -> RMEM -> GMEM  (D = alpha*acc + beta*C)
```

**EN**: Unchanged from Tutorial 02. Each CTA independently reads its accumulated result from TMEM and writes its portion of D to GMEM.  
**CN**: 与教程 02 相同。每个 CTA 独立从 TMEM 读取其累加结果，并将其 D 的部分写入 GMEM。

---

### Lines 410–600: Host function — multicast TMA descriptor creation / 主机函数——多播 TMA 描述符创建

```cpp
// Cluster shape: 4x4x1 (16 CTAs total)
auto cluster_shape = make_shape(Int<4>{}, Int<4>{}, Int<1>{});
Layout cluster_layout_vmnk = tiled_divide(make_layout(cluster_shape),
                                           make_tile(typename decltype(tiled_mma)::AtomThrID{}));
```

**EN**: The most visible change from Tutorial 02's host code: cluster shape is `(4,4,1)` instead of `(1,1,1)`. This drives the multicast factor of 4 in each dimension.

```cpp
// A: multicast to size<2>(cluster_layout_vmnk) = 4 CTAs along N
Copy_Atom tma_atom_A = make_tma_atom(
    SM90_TMA_LOAD_MULTICAST{},           // CHANGED: multicast variant
    mA,
    sA_layout,
    select<0,2>(mma_tiler),
    size<2>(cluster_layout_vmnk)         // CHANGED: multicast count = 4
);

// B: multicast to size<1>(cluster_layout_vmnk) = 4 CTAs along M
Copy_Atom tma_atom_B = make_tma_atom(
    SM90_TMA_LOAD_MULTICAST{},           // CHANGED: multicast variant
    mB,
    sB_layout,
    select<1,2>(mma_tiler),
    size<1>(cluster_layout_vmnk)         // CHANGED: multicast count = 4
);
```

**EN**: Two critical changes to `make_tma_atom`:  
1. `SM90_TMA_LOAD_MULTICAST{}` — Selects the **multicast-capable** TMA load instruction. This variant programs the TMA hardware to accept a runtime bitmask and deliver data to multiple SMEM destinations.  
2. Additional `size<N>(cluster_layout_vmnk)` argument — Specifies the **number of CTAs to multicast to**. The TMA descriptor must encode the multicast factor at descriptor creation time, as it affects the underlying memory transaction format.  

```cpp
// Grid and cluster launch
dim3 dimBlock(128);
dim3 dimCluster(4, 4, 1);   // 16-CTA cluster
dim3 dimGrid(
    ceil_div(Gemm_M, bM * size<1>(cluster_layout_vmnk)) * dimCluster.x,
    ceil_div(Gemm_N, bN * size<2>(cluster_layout_vmnk)) * dimCluster.y);
```

**EN**: The grid is sized so that each **cluster** covers a `(4×bM) × (4×bN)` output tile. `dimGrid.x` is divided by `size<1>` (4) and `dimGrid.y` by `size<2>` (4), then multiplied by the cluster dimensions, resulting in the correct number of independent problem tiles across the full output matrix.  
**CN**: 以上三段（主机端关键变化）：  
- 簇形状从 `(1,1,1)` 变为 `(4,4,1)`，驱动每个维度 4 倍的多播系数。  
- `make_tma_atom` 使用 `SM90_TMA_LOAD_MULTICAST{}` 选择多播 TMA 变体，并添加多播计数参数。  
- 网格维度按簇大小缩放：每个簇覆盖 `(4×bM) × (4×bN)` 的输出 tile。

---

### Lines 603–731: `main` (same structure as Tutorial 02) / `main`（与教程 02 相同结构）

```cpp
// SM100 check, MNK defaults: 512x1024x256
// Must be divisible by cluster×tile: 512/(4×128)=1, 1024/(4×256)=1 → fits
// TypeA=TypeB=half_t, TypeC=TypeD=float
// initialize_tensor, device copy, gemm_host call, reference_gemm, compare
```

**EN**: `main` is structurally identical to Tutorial 02. The only behavioral change is that the launched kernel uses a 4×4 cluster with multicast TMA, which is entirely handled inside `gemm_host_f16xf16_f32_f32_tnt`. The correctness check remains the same.  
**CN**: `main` 与教程 02 在结构上相同。唯一的行为变化是启动的内核使用带多播 TMA 的 4×4 簇，完全在 `gemm_host_f16xf16_f32_f32_tnt` 内部处理。正确性检查保持不变。

---

## Tutorial Progression Summary / 教程递进总结

| Feature / 特性 | Tutorial 01 | Tutorial 02 | Tutorial 03 |
|---|---|---|---|
| GMEM→SMEM copy / GMEM→SMEM 拷贝 | `cooperative_copy` (sync) | TMA (async, 1 CTA) | TMA multicast (async, N CTAs) |
| Cluster shape / 簇形状 | `(1,1,1)` | `(1,1,1)` | `(4,4,1)` |
| TMA descriptor / TMA 描述符 | None / 无 | `SM90_TMA_LOAD` | `SM90_TMA_LOAD_MULTICAST` |
| Multicast mask / 多播掩码 | None / 无 | None / 无 | Per-CTA bitmask / 逐 CTA 位掩码 |
| MMA barrier participants / MMA 屏障参与者 | 1 | 1 | 7 (4+4-1) |
| Sync primitive / 同步原语 | `__syncthreads` | `wait_barrier` | `cluster_sync` + `wait_barrier` |
| MMA arrive / MMA 到达信号 | `umma_arrive` | `umma_arrive` | `umma_arrive_multicast` |
| GMEM bandwidth savings / GMEM 带宽节省 | 1× | 1× | Up to 4× for A, 4× for B / A 最多 4×，B 最多 4× |

---

## Key Concepts / 关键概念

- **TMA Multicast** — Single TMA call delivers data to multiple CTAs' SMEM simultaneously via a bitmask / 单次 TMA 调用通过位掩码同时将数据传送到多个 CTA 的 SMEM
- **`SM90_TMA_LOAD_MULTICAST`** — Multicast-capable TMA instruction tag; descriptor must encode multicast count at creation time / 支持多播的 TMA 指令标签；描述符创建时必须编码多播计数
- **Multicast bitmask** — 16-bit mask, one bit per cluster CTA; computed per-CTA using `create_tma_multicast_mask` / 16 位掩码，每个簇 CTA 一位；使用 `create_tma_multicast_mask` 逐 CTA 计算
- **`create_tma_multicast_mask<Mode>`** — Generates cluster-specific bitmask for A (N-axis), B (M-axis), or C accumulation / 生成 A（N 轴）、B（M 轴）或 C 累加的簇特定位掩码
- **`umma_arrive_multicast`** — Signals MMA completion to all CTAs in the multicast group / 向多播组中的所有 CTA 发出 MMA 完成信号
- **`cluster_sync()`** — Cluster-wide barrier (all 16 CTAs sync) replacing `__syncthreads` for inter-CTA coordination / 替代 `__syncthreads` 用于 CTA 间协调的簇范围屏障（所有 16 个 CTA 同步）
- **A/B multicast reuse ratio** — A is shared across 4 N-axis CTAs (4× reuse); B is shared across 4 M-axis CTAs (4× reuse) / A 在 4 个 N 轴 CTA 间共享（4× 复用）；B 在 4 个 M 轴 CTA 间共享（4× 复用）
- **`block_rank_in_cluster()`** — Returns this CTA's linearized index within its cluster (0–15 for 4×4 cluster) / 返回此 CTA 在其簇内的线性索引（4×4 簇为 0–15）
- **`num_mcast_participants = M + N - 1`** — The number of distinct CTAs that share A or B data; needed to correctly initialize the cluster-wide MMA barrier / 共享 A 或 B 数据的不同 CTA 数量；正确初始化簇范围 MMA 屏障所需

## Dependencies / 依赖项

- `<cute/tensor.hpp>` — `SM90_TMA_LOAD_MULTICAST`, `create_tma_multicast_mask`, `tma_partition`, `cluster_sync` / 多播 TMA、掩码创建、TMA 划分、簇同步
- `<cute/arch/cluster_sm90.hpp>` — `block_rank_in_cluster`, `elect_one_sync` / 簇内块排名、单线程选举
- `<cute/arch/tmem_allocator_sm100.hpp>` — SM100 TMEM lifecycle management / SM100 张量内存生命周期
- `<cutlass/arch/barrier.h>` — `umma_arrive_multicast`, `initialize_barrier`, `wait_barrier` / 多播 MMA 到达信号、屏障原语
- `<cutlass/cluster_launch.hpp>` — `ClusterLaunchParams`, `launch_kernel_on_cluster` for 4×4 cluster / 4×4 簇的簇感知内核启动
- `"example_utils.hpp"` — `reference_gemm`, `initialize_tensor`, `compare_results` / 参考 GEMM、初始化、比较
