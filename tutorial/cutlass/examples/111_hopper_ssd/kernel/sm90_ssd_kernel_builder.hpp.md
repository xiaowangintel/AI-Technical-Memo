# sm90_ssd_kernel_builder.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/111_hopper_ssd/kernel/sm90_ssd_kernel_builder.hpp`  
**Purpose / 用途**: Compile-time Hopper builder that assembles the SM90 SSD mainloop, epilogue, scheduler, and final kernel type from template parameters. / Hopper 的编译期 builder：根据模板参数装配 SM90 SSD 主循环、epilogue、调度器以及最终 kernel 类型。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (lines 32-65) — Includes, template parameters, aliases, and feature flags
```cpp
#pragma once

#include "../collective/sm90_ssd_epilogue.hpp"
#include "../collective/sm90_ssd_gemm_tma_warpspecialized.hpp"
#include "../kernel/sm90_ssd_kernel_tma_warpspecialized.hpp"
#include "../kernel/sm90_ssd_tile_scheduler.hpp"

#include "cutlass/cutlass.h"
#include "cutlass/epilogue/collective/collective_builder.hpp"

namespace cutlass::ssd::kernel {

template<
  class Element_,
  class ElementDA_,
  class ElementAcc_,
  class ElementY_,
  class TileShape_,
  bool HAS_D_,
  bool D_HAS_HDIM_,
  bool HAS_Z_
>
struct Sm90SsdBuilder {
  using Element = Element_;
  using ElementDA = ElementDA_;
  using ElementAcc = ElementAcc_;
  using ElementY = ElementY_;
  using TileShape = TileShape_;

  static constexpr bool HAS_D = HAS_D_;
  static constexpr bool D_HAS_HDIM = D_HAS_HDIM_;
  static constexpr bool HAS_Z = HAS_Z_;
```
**EN**
- The template parameters describe both the math payload and the model variant.
- This opening section makes clear that the builder sits above the mainloop, epilogue, scheduler, and final kernel shell.
- The three boolean flags are especially important because they control which extra SSD/Mamba2 terms the epilogue must materialize.

**CN**
- 模板参数同时描述了数学计算负载与模型变体。
- 开头这部分清楚表明 builder 位于 mainloop、epilogue、scheduler 和最终 kernel 外壳之上。
- 三个布尔开关尤其关键，因为它们决定 epilogue 需要具体化哪些额外的 SSD/Mamba2 项。

### Block 2 (lines 66-79) — Stage counts and epilogue tile derivation
```cpp
static constexpr int StagesY = 2;
static constexpr int StagesX = 2;
static constexpr int StagesZ = 1; // smem size limitation
using EpilogueTileType = cutlass::epilogue::collective::EpilogueTileAuto;
using Schedule = cutlass::epilogue::TmaWarpSpecialized;
using EpilogueTile = decltype(cutlass::epilogue::collective::detail::sm90_compute_tile_shape_or_override<
  ElementY, EpilogueTileType, Schedule, TileShape>());
```
**EN**
- X and Y use double buffering, while Z uses only one stage because shared-memory pressure is already high.
- The epilogue tile is computed with SM90-specific helper logic so it matches a TMA-warp-specialized Hopper epilogue schedule.

**CN**
- X 与 Y 使用双缓冲，而 Z 只保留一个 stage，因为共享内存压力已经很大。
- epilogue tile 通过 SM90 专用辅助逻辑计算，以便匹配 TMA-warp-specialized 的 Hopper epilogue 调度方式。

### Block 3 (lines 74-87) — Y and X shared-memory layouts
```cpp
using SmemLayoutAtomY = decltype(cutlass::gemm::collective::detail::ss_smem_selector<
    cute::GMMA::Major::MN, ElementY, decltype(get<0>(EpilogueTile{})), decltype(get<1>(EpilogueTile{}))>());
using SmemLayoutY = decltype(tile_to_shape(
    SmemLayoutAtomY{},
    make_shape(size<0>(EpilogueTile{}), size<1>(EpilogueTile{}), Int<StagesY>{}),
    Step<_2,_1,_3>{}));

using SmemLayoutAtomX = decltype(cutlass::gemm::collective::detail::ss_smem_selector<
    cute::GMMA::Major::MN, Element, decltype(get<0>(TileShape{})), decltype(get<1>(TileShape{}))>());
using SmemLayoutX = decltype(tile_to_shape(
    SmemLayoutAtomY{},
    make_shape(size<0>(TileShape{}), size<1>(TileShape{}), Int<StagesX>{}),
    Step<_2,_1,_3>{}));
```
**EN**
- This block chooses staged SMEM layouts for the output path (`Y`) and the mainloop input path (`X`).
- The builder uses CUTLASS/CuTe layout selectors so the resulting layouts follow Hopper-friendly GMMA storage conventions.
- As written, `SmemLayoutX` is built from `SmemLayoutAtomY`, so the code literally reuses Y’s atom basis when forming X’s staged layout.

**CN**
- 这一段为输出路径（`Y`）和 mainloop 输入路径（`X`）选择带 stage 的共享内存布局。
- Builder 使用 CUTLASS/CuTe 的布局选择器，使得到的布局符合 Hopper 友好的 GMMA 存储约定。
- 按代码字面看，`SmemLayoutX` 是基于 `SmemLayoutAtomY` 构造的，也就是说在形成 X 的 staged layout 时复用了 Y 的原子布局基础。

### Block 4 (lines 88-103) — Z layout and partial-Y accumulator layout
```cpp
using SmemLayoutAtomZ = decltype(cutlass::gemm::collective::detail::ss_smem_selector<
    cute::GMMA::Major::MN, Element, decltype(get<0>(TileShape{})), decltype(get<1>(TileShape{}))>());
using SmemLayoutZ = decltype(tile_to_shape(
    SmemLayoutAtomZ{},
    make_shape(size<0>(TileShape{}), size<1>(TileShape{}), Int<StagesZ>{}),
    Step<_2,_1,_3>{}));

static constexpr auto epi_tile_m = size<0>(EpilogueTile{});
static constexpr auto epi_tile_n = size<1>(EpilogueTile{});
static constexpr auto partial_m = Int<128>{};
static constexpr auto partial_n = Int<epi_tile_m * epi_tile_n / 128>{};

using SmemLayoutAtomPartialY = typename GMMA::Layout_K_SW64_Atom<ElementAcc>;
using SmemLayoutPartialY = decltype(tile_to_shape(
    SmemLayoutAtomPartialY{}, 
    make_shape(partial_m, partial_n, Int<StagesY>{})));
```
**EN**
- Z gets its own staged layout because it is loaded separately by the epilogue path.
- Partial Y accumulators are stored with a GMMA-friendly SW64 atom, which is the temporary handoff format before final output assembly.
- This is where the builder starts translating recurrence output needs into concrete Hopper storage shapes.

**CN**
- Z 之所以拥有独立的 staged layout，是因为它会在 epilogue 路径中被单独加载。
- `partial Y` 累加结果采用对 GMMA 友好的 SW64 原子布局存储，它是最终输出组装前的中间交接格式。
- 从这里开始，builder 就把递推输出需求转换成具体的 Hopper 存储形状。

### Block 5 (lines 105-113) — Compose mainloop, epilogue, scheduler, and final kernel type
```cpp
using CollectiveMainloop = cutlass::ssd::collective::SsdMainloopTmaWarpSpecialized<Element, ElementDA, ElementAcc, ElementY, TileShape, StagesX>;
using CollectiveEpilogue = cutlass::ssd::collective::SsdEpilogue<
  ElementAcc, ElementY, TileShape,
  EpilogueTile, SmemLayoutX, SmemLayoutY, SmemLayoutPartialY, typename CollectiveMainloop::SmemLayoutP, SmemLayoutZ,
  StagesX, StagesY, StagesZ,
  HAS_D, D_HAS_HDIM, HAS_Z>;
using TileScheduler = cutlass::ssd::kernel::PersistentTileScheduler;
using Kernel = cutlass::ssd::kernel::SsdKernelTmaWarpSpecialized<CollectiveMainloop, CollectiveEpilogue, TileScheduler>;
```
**EN**
- This final composition step is the essence of the builder.
- The SSD recurrence becomes a runnable Hopper kernel because the builder picks a mainloop for intra/inter math, an epilogue for Y/P/D/Z handling, and a persistent scheduler for tile traversal.
- `Kernel` is the concrete type later launched by the device wrapper in `device/ssd.hpp`.

**CN**
- 这个最终组合步骤就是 builder 的核心所在。
- SSD 递推之所以能变成可运行的 Hopper kernel，是因为 builder 在这里选定了用于 intra/inter 数学计算的 mainloop、用于 Y/P/D/Z 处理的 epilogue，以及用于 tile 遍历的 persistent scheduler。
- `Kernel` 就是之后由 `device/ssd.hpp` 中设备包装器真正启动的具体类型。

---

## Key Concepts / 关键概念

1. **Builder as compile-time glue**
   - **EN:** The file contains almost no runtime logic; its purpose is type construction and policy selection.
   - **CN:** 这个文件几乎没有运行时逻辑，它的职责是做类型装配和策略选择。
2. **Recurrence decomposition into collectives**
   - **EN:** SSD is not expressed as one monolithic kernel here. Instead, the builder instantiates a specialized mainloop and epilogue and then wraps them in a top-level kernel.
   - **CN:** 这里并没有把 SSD 写成一个整体式内核，而是先实例化专门化的 mainloop 和 epilogue，再把它们包进顶层 kernel。
3. **Shared-memory layout selection**
   - **EN:** The builder derives SMEM layouts for X, Y, Z, and partial accumulators with CUTLASS/CuTe layout helpers so Hopper GMMA/TMA paths get hardware-friendly storage.
   - **CN:** Builder 用 CUTLASS/CuTe 的布局辅助工具推导 X、Y、Z 和部分累加结果的共享内存布局，从而为 Hopper 的 GMMA/TMA 路径提供对硬件友好的存储形式。
4. **Feature switches encode model variants**
   - **EN:** `HAS_D`, `D_HAS_HDIM`, and `HAS_Z` select whether the epilogue must incorporate extra SSD terms such as diagonal/state scaling and gating.
   - **CN:** `HAS_D`、`D_HAS_HDIM` 和 `HAS_Z` 用来选择 epilogue 是否需要并入额外的 SSD 项，例如对角/状态缩放和门控项。

## Dependencies / 依赖项

- `../collective/sm90_ssd_epilogue.hpp`
  - **EN:** Defines the SSD epilogue collective responsible for Y/P stores and optional D/Z integration.
  - **CN:** 定义 SSD epilogue collective，负责 Y/P 写回以及可选的 D/Z 融合。
- `../collective/sm90_ssd_gemm_tma_warpspecialized.hpp`
  - **EN:** Defines the SSD mainloop collective that maps recurrence math onto Hopper TMA/GMMA execution.
  - **CN:** 定义 SSD mainloop collective，把递推数学映射到 Hopper 的 TMA/GMMA 执行路径上。
- `../kernel/sm90_ssd_kernel_tma_warpspecialized.hpp`
  - **EN:** Supplies the top-level warp-specialized kernel instantiated by the builder.
  - **CN:** 提供由 builder 实例化的顶层 warp-specialized kernel。
- `../kernel/sm90_ssd_tile_scheduler.hpp`
  - **EN:** Supplies the persistent CTA scheduler used by the final kernel.
  - **CN:** 提供最终 kernel 使用的 persistent CTA scheduler。
- `cutlass/epilogue/collective/collective_builder.hpp`
  - **EN:** Provides helper machinery for epilogue tile computation on SM90.
  - **CN:** 提供 SM90 epilogue tile 计算所需的辅助模板机制。
