# sm100_ssd_kernel_builder.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/112_blackwell_ssd/kernel/sm100_ssd_kernel_builder.hpp`  
**Purpose / 用途**: Compile-time Blackwell builder that assembles the SM100 SSD mainloop, epilogue, scheduler, and final kernel type. / Blackwell 的编译期 builder：负责装配 SM100 SSD 主循环、epilogue、调度器和最终 kernel 类型。

---

## Line-by-Line Analysis / 逐行分析

### 1. Includes and builder role (lines 34-40) / 头文件与 builder 角色（34-40 行）

```cpp
#include "../collective/sm100_ssd_epilogue.hpp"
#include "../collective/sm100_ssd_gemm_tma_warpspecialized.hpp"
#include "../kernel/sm100_ssd_kernel_tma_warpspecialized.hpp"
#include "../kernel/sm100_ssd_tile_scheduler.hpp"
```

**EN** These includes already reveal the file's job: it is not a standalone algorithm implementation, but a composition point that selects the Blackwell mainloop, epilogue, kernel body, and scheduler.

**CN** 这些头文件已经说明了此文件的职责：它不是独立的算法实现，而是一个组合点，用来选定 Blackwell 版本的 mainloop、epilogue、kernel 主体和 scheduler。

### 2. SM100 helper factories for tiled MMA construction (lines 42-76) / 用于构造 tiled MMA 的 SM100 辅助工厂（42-76 行）

```cpp
constexpr auto sm100_make_ts_tiled_mma() {
  return cutlass::gemm::collective::detail::sm100_make_1sm_ts_trivial_tiled_mma<...>();
}

constexpr auto sm100_make_ss_tiled_mma() {
  return cutlass::gemm::collective::detail::sm100_make_1sm_trivial_tiled_mma<...>();
}
```

**EN** The builder offers two flavors of SM100 MMA construction: `ts` and `ss`. Their exact low-level encoding lives in CUTLASS internals, but at this level they express which operand-major pair is needed for each SSD sub-problem.

**CN** builder 提供了两种 SM100 MMA 构造方式：`ts` 和 `ss`。底层编码细节在 CUTLASS 内部，这里主要表达的是：SSD 的每个子问题分别需要哪种操作数主序组合。

### 3. Template parameters, fixed architecture, and tile decomposition (lines 80-108) / 模板参数、固定架构与 tile 拆分（80-108 行）

```cpp
using ArchTag = cutlass::arch::Sm100;
using ClusterShape = Shape<_1,_1,_1>;
static constexpr int StagesInput = 2;
static constexpr int StagesOutput = 2;

using TileShapeIntraBMM1 = decltype(make_shape(get<0>(TileShape{}), get<0>(TileShape{}), get<2>(TileShape{})));
using TileShapeIntraBMM2 = decltype(make_shape(get<0>(TileShape{}), get<1>(TileShape{}), get<0>(TileShape{})));
using TileShapeInterBMM1 = decltype(make_shape(get<2>(TileShape{}), get<1>(TileShape{}), get<0>(TileShape{})));
using TileShapeInterBMM2 = decltype(make_shape(get<0>(TileShape{}), get<1>(TileShape{}), get<2>(TileShape{})));
```

**EN** The core design choice appears here: one logical SSD tile `(L, D, N)` is lowered into four different GEMM/BMM views. The naming strongly suggests the intended separation between chunk-local work (`Intra*`, where `L` dominates) and cross-chunk state propagation (`Inter*`, where `N` and `D` become prominent).

**CN** 这里出现了核心设计选择：一个逻辑上的 SSD tile `(L, D, N)` 被降解成四种不同的 GEMM/BMM 视图。命名上已经明显体现出职责划分：`Intra*` 偏向 chunk 内计算（`L` 更突出），`Inter*` 偏向 chunk 间状态传播（`N` 和 `D` 更突出）。

### 4. Four tiled MMAs that encode the recurrence decomposition (lines 109-124) / 编码 recurrence 拆分的四个 tiled MMA（109-124 行）

```cpp
using TiledMmaIntra1 = decltype(detail::sm100_make_ss_tiled_mma<..., cute::UMMA::Major::MN, cute::UMMA::Major::MN>());
using TiledMmaIntra2 = decltype(detail::sm100_make_ts_tiled_mma<..., cute::UMMA::Major::K,  cute::UMMA::Major::K >());
using TiledMmaInter1 = decltype(detail::sm100_make_ts_tiled_mma<..., cute::UMMA::Major::K,  cute::UMMA::Major::K >());
using TiledMmaInter2 = decltype(detail::sm100_make_ss_tiled_mma<..., cute::UMMA::Major::MN, cute::UMMA::Major::K >());
```

**EN** This is the most important mathematical-to-hardware bridge in the file. The comments (`LxLxN`, `LxDxL`, `NxDxL`, `LxDxN`) show that SSD/Mamba2 is not executed as one monolithic recurrence kernel; instead, it is expressed as multiple matrix-style phases whose operand orders are chosen to match SM100 UMMA expectations.

**CN** 这是文件中最关键的“数学到硬件”桥接点。注释里的形状（`LxLxN`、`LxDxL`、`NxDxL`、`LxDxN`）说明：SSD/Mamba2 并不是作为单一的大 recurrence 内核执行，而是被改写成多个矩阵式阶段，并根据 SM100 的 UMMA 需求选择不同的操作数顺序。

### 5. Partitioned MMA shapes and layout atoms (lines 126-180) / 分区后的 MMA 形状与布局原子（126-180 行）

```cpp
using MmaShapeC_MK = decltype(partition_shape_A(TiledMmaIntra1{}, ...));
using MmaShapeB_NK = decltype(partition_shape_B(TiledMmaIntra1{}, ...));
...
using GmemTiledCopyX = cute::SM90_TMA_LOAD;
using GmemTiledCopyB = cute::SM90_TMA_LOAD;
using GmemTiledCopyC = cute::SM90_TMA_LOAD;
```

**EN** Once the four MMAs are chosen, the builder derives per-operand partition shapes and then selects SM100 shared-memory layout atoms. One subtle but important point is that TMA load atoms still use the `SM90_TMA_LOAD` name; the CuTe naming survives across architectures even though the surrounding kernel is clearly SM100-specific.

**CN** 在选定四个 MMA 之后，builder 会继续推导每个操作数的分区形状，并选择 SM100 共享内存布局原子。一个容易误读但很重要的细节是：TMA load atom 仍然使用 `SM90_TMA_LOAD` 这个名字；这是 CuTe 的命名沿用，不代表这个内核不是 SM100 专用。

### 6. SMEM/TMEM layout materialization (lines 182-225) / SMEM/TMEM 布局实体化（182-225 行）

```cpp
using SmemLayoutX = decltype(UMMA::tile_to_mma_shape(... Int<StagesInput>{} ...));
using SmemLayoutB = decltype(UMMA::tile_to_mma_shape(... Int<StagesInput>{} ...));
using TmemLayoutB = decltype(UMMA::tile_to_mma_shape(... Int<1>{} ...));
...
using TmemLayoutQ = decltype(UMMA::tile_to_mma_shape(... Int<1>{} ...));
using SmemLayoutQ = decltype(UMMA::tile_to_mma_shape(... Int<2>{} ...));
```

**EN** This section decides where each tensor lives while the kernel runs. X/B/C/P/Q all get explicit staged layouts, and two of them also get TMEM layouts. That is a major Blackwell-specific clue: compared with the Hopper builder, which is GMMA-centric and shared-memory oriented, the SM100 builder explicitly prepares TMEM-backed intermediate dataflow.

**CN** 这一段决定了内核运行时各张量的驻留位置。X/B/C/P/Q 都拥有明确的分阶段布局，其中两个中间量还拥有专门的 TMEM 布局。这是一个非常明显的 Blackwell 特征：相比 Hopper builder 更偏向 GMMA 和共享内存，SM100 builder 明确为中间数据流准备了基于 TMEM 的路径。

### 7. Epilogue layouts and output staging (lines 227-240) / Epilogue 布局与输出分阶段存储（227-240 行）

```cpp
using EpilogueTile = Shape<Int<128>, Int<32>>;
using SmemLayoutY = decltype(tile_to_shape(... Int<StagesOutput>{} ...));
using SmemLayoutStoreP = decltype(tile_to_shape(... Int<1>{} ...));
```

**EN** The builder fixes the epilogue tile to `128x32`, then creates the shared-memory layouts needed to write `Y` and the state tensor `P`. This reflects the SSD split between sequence output and carried state.

**CN** builder 把 epilogue tile 固定为 `128x32`，然后构造写回 `Y` 与状态张量 `P` 所需的共享内存布局。这正对应了 SSD 中“序列输出”和“跨 chunk 传递状态”这两条结果路径。

### 8. Final assembly into CUTLASS collectives and kernel type (lines 242-255) / 最终装配为 CUTLASS collectives 与 kernel 类型（242-255 行）

```cpp
using CollectiveMainloop = cutlass::ssd::collective::SsdMainloopTmaWarpSpecialized<...>;
using CollectiveEpilogue = cutlass::ssd::collective::SsdEpilogue<...>;
using TileScheduler = cutlass::ssd::kernel::PersistentTileScheduler;
using Kernel = cutlass::ssd::kernel::SsdKernelTmaWarpSpecialized<CollectiveMainloop, CollectiveEpilogue, TileScheduler>;
```

**EN** This is where the SSD kernel becomes concrete. The builder locks in the Blackwell mainloop, Blackwell epilogue, persistent scheduler, and warp-specialized kernel shell. Compared with the Hopper builder in `examples/111_hopper_ssd`, the visible differences are the move from GMMA-oriented layouts to UMMA/TMEM-oriented layouts, plus the absence of Hopper's `HAS_Z` path in this SM100 builder.

**CN** 这里是 SSD 内核真正具体化的地方。builder 在此固定了 Blackwell mainloop、Blackwell epilogue、persistent scheduler 以及 warp-specialized kernel 外壳。和 `examples/111_hopper_ssd` 中的 Hopper builder 相比，最明显的区别是：从以 GMMA 为中心的布局切换到以 UMMA/TMEM 为中心的布局，同时这个 SM100 builder 也没有继续暴露 Hopper 版本里的 `HAS_Z` 路径。

---

## Key Concepts / 关键概念

- Four BMM-style phases / 四个类 BMM 阶段
  - **EN** The file splits the SSD recurrence into two intra-chunk MMAs and two inter-chunk MMAs.
  - **CN** 文件把 SSD recurrence 拆成两个 chunk 内 MMA 和两个 chunk 间 MMA。
- UMMA on SM100 / SM100 上的 UMMA
  - **EN** Blackwell uses `UMMA::Major` and SM100-specific tiled MMA builders.
  - **CN** Blackwell 使用 `UMMA::Major` 与 SM100 专用 tiled MMA 构造器。
- TMA + SMEM + TMEM
  - **EN** Inputs are staged through TMA/SMEM layouts, while some intermediates (`B`, `Q`) get dedicated TMEM layouts.
  - **CN** 输入经由 TMA/SMEM 分阶段搬运，部分中间量（如 `B`、`Q`）还拥有专门的 TMEM 布局。
- Builder as composition root / 组合根节点
  - **EN** The file does not run kernels; it wires together the mainloop, epilogue, scheduler, and final kernel type.
  - **CN** 这个文件不执行内核；它负责把 mainloop、epilogue、scheduler 和最终 kernel 类型装配起来。

## Dependencies / 依赖项

- `../collective/sm100_ssd_epilogue.hpp`
- `../collective/sm100_ssd_gemm_tma_warpspecialized.hpp`
- `../kernel/sm100_ssd_kernel_tma_warpspecialized.hpp`
- `../kernel/sm100_ssd_tile_scheduler.hpp`
- `cutlass/epilogue/collective/collective_builder.hpp`
- CUTLASS/CuTe concepts used heavily:
  - `UMMA::Major`
  - `partition_shape_A/B`
  - `sm100_smem_selector`, `ss_smem_selector`
  - `UMMA::tile_to_mma_shape`, `tile_to_shape`
