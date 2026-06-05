# gemm_api_3x.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/sycl-tla/media/docs/cpp/gemm_api_3x.md`
- **Purpose:** **EN:** Defines the CUTLASS 3.0 GEMM hierarchy and explains how users assemble kernels from collective, kernel, and device components. **CN:** 定义 CUTLASS 3.0 的 GEMM 层次结构，并说明用户如何从 collective、kernel 和 device 组件组装内核。

## Content Analysis / 内容分析
### CUTLASS GEMM Model
**EN:** The model section reinterprets GEMM as a layered loop nest spanning cluster scheduling, collective mainloops, tiled MMA/copy, and atom-level instructions. It is the conceptual backbone for the entire 3.x API.
**CN:** 模型部分把 GEMM 重新解释为跨越 cluster 调度、collective mainloop、tiled MMA/copy 以及 atom 级指令的分层循环结构，是整个 3.x API 的概念骨架。

### CUTLASS GEMM Components
**EN:** This section maps each layer to a concrete type: `GemmUniversalAdapter`, `GemmUniversal`, `CollectiveMma`, `TiledMma/TiledCopy`, and atoms. It also shows the recommended assembly order from collective to kernel to device wrapper.
**CN:** 这一节把每一层映射到具体类型：`GemmUniversalAdapter`、`GemmUniversal`、`CollectiveMma`、`TiledMma/TiledCopy` 和 atom，并给出从 collective 到 kernel 再到 device wrapper 的推荐组装顺序。

### Collective API
**EN:** The collective layer is described as the largest cooperating execution unit that still shares synchronization and data-movement machinery. This framing is important because it replaces older threadblock-centric thinking with a more general abstraction.
**CN:** 文档把 collective 层定义为“仍能共享同步与数据搬运机制的最大协作执行单元”。这一点很重要，因为它用更一般化的抽象替代了旧版本偏 threadblock 中心的思维方式。

### Collective Mainloops
**EN:** The `CollectiveMma` section explains the expert-facing interface for mainloops, including dispatch policy, global-memory strides, tiled copy, shared-memory layout atoms, and transform hooks. It shows that 3.x favors explicit composition over hidden defaults.
**CN:** `CollectiveMma` 小节介绍了面向专家用户的 mainloop 接口，包括 dispatch policy、全局内存步长、tiled copy、共享内存 layout atom 以及变换钩子，体现了 3.x “显式组合优先于隐藏默认值”的设计取向。

### Collective Dispatch Policies
**EN:** Dispatch policies are presented as the central extension and tuning point. They encode stages, cluster shape, schedule tags, and architecture assumptions, allowing CUTLASS to keep one vocabulary type while still specializing aggressively.
**CN:** dispatch policy 被定位为核心扩展点和调优点。它编码了 stage 数、cluster 形状、schedule 标签以及架构假设，使 CUTLASS 能在保留统一词汇类型的同时进行强力特化。

### Collective Builder for `CollectiveMma`s
**EN:** The builder section offers the pragmatic path for most users: feed in architecture, operator class, layouts, alignments, and tile shapes, and receive a suitable collective. The document is careful to note that this convenience layer is helpful but not exhaustive.
**CN:** builder 部分为大多数用户提供了务实路径：给出架构、操作类、布局、对齐和 tile 形状，就能生成合适的 collective。文档也明确提醒，这一便利层很有用，但并不覆盖全部设计空间。

### Epilogue
**EN:** The epilogue discussion is brief but clarifies that output-side logic remains collective and reusable outside GEMM-specific namespaces. This keeps the postprocessing path flexible for fused or non-GEMM computations.
**CN:** epilogue 部分虽然简短，但明确指出输出侧逻辑仍然属于 collective 范畴，并且不局限于 GEMM 命名空间，从而为融合操作或非 GEMM 计算保留了灵活性。

### Kernel API
**EN:** The kernel section defines `kernel::GemmUniversal` as the stateless device-side composition of mainloop and epilogue. It stresses that the kernel owns scheduling and launch-shape logic, which is essential for persistent and architecture-specific schedules.
**CN:** kernel 部分把 `kernel::GemmUniversal` 定义为由 mainloop 和 epilogue 组合而成的无状态设备侧内核，并强调 kernel 自身负责调度与启动形状逻辑，这对持久化和架构专用 schedule 至关重要。

### Device API
**EN:** `GemmUniversalAdapter` is presented as the reusable host handle that manages argument lowering, parameter lifetime, and launch details. The section shows how 3.x separates host orchestration from kernel internals more cleanly than before.
**CN:** `GemmUniversalAdapter` 被描述为可复用的主机侧句柄，用于管理参数降级、参数生命周期和启动细节。该节体现了 3.x 如何比以往更清晰地分离主机侧编排与内核内部逻辑。

### Tiled MMA and Copy
**EN:** This layer generalizes old warp-level ideas into reusable tilings of atoms across threads and data. The explanation makes it clear that CUTLASS now treats math and copy as similarly composable tiled operations.
**CN:** 这一层把旧的 warp 级概念推广为跨线程和数据的 atom 平铺操作。文档强调，CUTLASS 现在把数学运算和数据拷贝都视为可以对称组合的 tiled 操作。

### Atom API
**EN:** The atom section closes the hierarchy by defining the smallest indivisible accelerated operation. It also directs readers to CuTe tutorials, implicitly positioning CuTe as the place to learn the lowest-level mechanics in depth.
**CN:** atom 小节以“最小不可分的硬件加速操作”结束整个层次结构，并把读者引向 CuTe 教程，暗示深入理解最低层机制应当转到 CuTe 文档中完成。

## Key Concepts / 关键概念
- **`CollectiveMma`** — **EN:** The primary 3.x mainloop abstraction for cooperating execution units. **CN:** 3.x 中面向协作执行单元的核心 mainloop 抽象。
- **Dispatch policy** — **EN:** The type-level control point for schedule selection and tuning. **CN:** 用于选择 schedule 和调优的类型级控制点。
- **`GemmUniversal`** — **EN:** Stateless kernel composition point for CUTLASS GEMMs. **CN:** CUTLASS GEMM 的无状态 kernel 组合点。
- **`GemmUniversalAdapter`** — **EN:** Reusable host-side launcher and parameter manager. **CN:** 可复用的主机侧启动器和参数管理器。
- **Atom** — **EN:** The smallest thread/data unit bound to a hardware-accelerated op. **CN:** 与硬件加速操作绑定的最小线程/数据单元。

## Related Files / 相关文件
- `media/docs/cpp/cutlass_3x_design.md` — **EN:** Explains why this hierarchy looks the way it does. **CN:** 解释该层次结构为何如此设计。
- `media/docs/cpp/cutlass_3x_backwards_compatibility.md` — **EN:** Describes how 3.x interfaces relate to 2.x. **CN:** 描述 3.x 接口与 2.x 的关系。
- `include/cutlass/gemm/collective/collective_builder.hpp` — **EN:** Builder API referenced in the document. **CN:** 文中引用的 builder API 头文件。
- `include/cutlass/gemm/kernel/gemm_universal.hpp` — **EN:** Kernel entry type described by the guide. **CN:** 本指南讲解的 kernel 入口类型定义。

