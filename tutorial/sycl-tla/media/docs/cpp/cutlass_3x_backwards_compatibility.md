# cutlass_3x_backwards_compatibility.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/sycl-tla/media/docs/cpp/cutlass_3x_backwards_compatibility.md`
- **Purpose:** **EN:** Explains how CUTLASS 3.0 preserves practical compatibility with CUTLASS 2.x at the device, kernel, and layout levels. **CN:** 说明 CUTLASS 3.0 如何在 device、kernel 和布局层面尽可能保持与 CUTLASS 2.x 的实际兼容性。

## Content Analysis / 内容分析
### Compatible Device API
**EN:** The document starts from the host-facing entry point, `GemmUniversalAdapter`, and explains that it is the common wrapper for both 2.x and 3.x kernels. This section reassures existing users that old and new kernels can coexist in the same translation unit.
**CN:** 文档先从主机侧入口 `GemmUniversalAdapter` 讲起，说明它是 2.x 与 3.x 内核共用的统一包装器。这一节传达的核心信息是：旧内核和新内核可以在同一个翻译单元中共存。

### Device API design differences
**EN:** Here the authors clarify that 3.x device code is more generic and less tightly coupled to one kernel implementation. They also point out a new power-user path that bypasses argument lowering and directly reuses or manages kernel params.
**CN:** 这一节进一步说明 3.x 的 device 代码更加通用，不再像 2.x 那样与某一种 kernel 实现强耦合。同时指出了一个面向高级用户的新路径：可以绕过参数降级流程，直接复用或管理 kernel params。

### Compatible Kernel API
**EN:** The kernel section introduces `cutlass::gemm::kernel::GemmUniversal` as the universal composition point for both APIs. The core rule is simple: a `cute::tuple`/shape-style first argument means 3.x semantics; otherwise the type is interpreted as a 2.x-style kernel composition.
**CN:** kernel 部分把 `cutlass::gemm::kernel::GemmUniversal` 定义为兼容两代 API 的统一组合点。核心规则很直接：如果第一个模板参数是 `cute::tuple`/shape 风格，就是 3.x 语义；否则按 2.x 风格解释。

### Kernel API design differences
**EN:** This section explains one of the major semantic changes: 3.x kernels own more launch planning logic, including grid shape queries. That shift is necessary for persistent and architecture-specific schedules that cannot be fully dictated by the device wrapper.
**CN:** 这一节解释了一个关键语义变化：3.x 内核自身承担更多启动规划逻辑，包括 grid shape 查询。这种变化对持久化和架构专用 schedule 很有必要，因为它们不能完全由 device wrapper 统一决定。

### Threadblock API and Inner Loops
**EN:** The discussion then acknowledges that compatibility stops being literal below the kernel layer. CUTLASS 3.x moves new development toward collectives and CuTe atoms, effectively replacing many old `threadblock`, `warp`, and `thread` extension points.
**CN:** 接下来文档明确指出：真正的“逐层兼容”在 kernel 以下并不成立。CUTLASS 3.x 把新的开发重心转向 collective 与 CuTe atom，实际上替代了许多旧的 `threadblock`、`warp` 和 `thread` 扩展点。

### Porting from 2.x to 3.0 API
**EN:** The porting section gives a migration-oriented vocabulary shift, especially for how A/B/C layouts are described. Instead of reusing row/column-major language that changes meaning across operands, 3.x adopts uniform outer/inner (M/N/K-major) terminology.
**CN:** 迁移部分给出了面向 3.0 的术语转换，重点是 A/B/C 布局的描述方式。3.x 不再沿用会随操作数变化语义的 row/column-major，而改为统一的 outer/inner（M/N/K-major）表达。

### CUTLASS 2.x layout tags and CUTLASS 3.0 major modes
**EN:** This section is especially valuable because it explains the conceptual shift from tag-only layouts to explicit shape-plus-stride thinking. The comparison table helps users reinterpret old `RowMajor`/`ColumnMajor` knowledge in the new CuTe model.
**CN:** 这一节尤其有价值，因为它解释了从“仅靠 tag 的布局”到“显式形状加步长”思维方式的转变。配套表格帮助用户把旧的 `RowMajor`/`ColumnMajor` 认知重新映射到新的 CuTe 模型里。

### Conversions between 2.x tags and 3.0 types
**EN:** The final technical section lists the `TagToStride*` and `StrideToLayoutTag*` helpers that translate between both worlds. It also honestly warns that the mapping is approximate, which is an important practical note for reflection-heavy or generic code.
**CN:** 最后一节列出了 `TagToStride*` 与 `StrideToLayoutTag*` 等辅助工具，用来在两套体系之间做转换。文档也坦诚指出这种映射只是“尽力逼近”，这对依赖反射或泛型适配的代码非常重要。

## Key Concepts / 关键概念
- **`GemmUniversalAdapter`** — **EN:** A universal host-side wrapper that can launch both 2.x and 3.x kernels. **CN:** 可同时启动 2.x 和 3.x 内核的统一主机侧包装器。
- **`GemmUniversal`** — **EN:** The shared kernel-level entry type used for compatibility across API generations. **CN:** 跨 API 代际兼容所使用的统一 kernel 入口类型。
- **`IsCutlass3GemmKernel`** — **EN:** Type trait used to distinguish 3.x kernels from legacy ones. **CN:** 用于区分 3.x 内核与旧内核的类型特征。
- **Tag/Stride conversion** — **EN:** Utilities that bridge 2.x layout tags and 3.x CuTe stride types. **CN:** 在 2.x 布局标签与 3.x CuTe stride 类型之间搭桥的工具。

## Related Files / 相关文件
- `media/docs/cpp/gemm_api_3x.md` — **EN:** Full explanation of the new 3.x hierarchy referenced here. **CN:** 这里反复引用的 3.x 层次结构完整说明。
- `media/docs/cpp/cutlass_3x_design.md` — **EN:** Design rationale behind the compatibility choices. **CN:** 解释这些兼容性设计背后的整体动机。
- `include/cutlass/gemm/device/gemm_universal_adapter.h` — **EN:** Header implementing the compatible device adapter. **CN:** 实现兼容 device adapter 的头文件。
- `include/cutlass/gemm/gemm.h` — **EN:** Contains layout-conversion utilities referenced by the document. **CN:** 包含文档提到的布局转换工具。

