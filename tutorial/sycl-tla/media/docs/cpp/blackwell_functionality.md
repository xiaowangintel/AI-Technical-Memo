# blackwell_functionality.md — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/sycl-tla/media/docs/cpp/blackwell_functionality.md`
- **Purpose:** **EN:** Serves as a detailed reference for Blackwell SM100/SM120 GEMMs, covering instructions, datatypes, layouts, tile shapes, epilogues, and builder usage. **CN:** 作为 Blackwell SM100/SM120 GEMM 的详细参考，覆盖指令、数据类型、布局、tile 形状、epilogue 以及 builder 的使用方式。

## Content Analysis / 内容分析
### New in Blackwell SM100
**EN:** The opening section positions `tcgen05.mma` as the core hardware advance of SM100 and summarizes the new instruction families, including legacy, narrow-precision, and block-scaled variants. It establishes the performance motivation for the rest of the document.
**CN:** 开头部分把 `tcgen05.mma` 定位为 SM100 的核心硬件升级，并概览了新的指令家族，包括传统类型、窄精度以及 block-scaled 变体，为后续内容建立性能动机。

### Block Scaled GEMMs
**EN:** This subsection explains the mathematical form of block scaling, where A and B values are multiplied by per-block scale factors along K. The text is especially useful because it connects PTX semantics, tensor shapes, and scale-factor vector size in one place.
**CN:** 这一小节解释了 block scaling 的数学形式：A 和 B 的值会沿 K 维乘上分块的缩放因子。它把 PTX 语义、张量形状和 scale-factor 向量大小放在一起说明，信息密度很高。

### Blackwell Narrow Precision Data Types
**EN:** The document catalogs Blackwell’s 4-bit, 6-bit, and 8-bit floating-point formats, and then maps them to CUTLASS `mx_*` and `nv_*` wrapper types. This section matters because supported scale-factor types and vector sizes directly constrain legal kernels.
**CN:** 文档列举了 Blackwell 的 4 位、6 位和 8 位浮点格式，并进一步映射到 CUTLASS 的 `mx_*` 与 `nv_*` 包装类型。之所以重要，是因为可用的 scale-factor 类型和向量长度会直接限制合法内核配置。

### Layouts, Tensor Alignment Requirements to Target `tcgen05.mma` Instructions
**EN:** This section is mainly a compatibility matrix: it records which type/layout combinations are valid, what alignment is required, and which instruction kind or unit test corresponds to each case. In practice, it functions as a correctness checklist before kernel construction.
**CN:** 这一节本质上是一张兼容性总表：说明哪些类型/布局组合有效、需要什么对齐方式，以及分别对应哪类指令和单元测试。实际使用时，它更像是构建内核前的正确性检查表。

### MMA tile shapes supported
**EN:** The tile-shape tables translate datatype and layout constraints into concrete 1SM/2SM dispatch policies. Rather than explaining algorithms, this part acts as the architectural lookup layer between legal math instructions and performant kernel schedules.
**CN:** tile 形状表把数据类型与布局约束进一步落实为具体的 1SM/2SM dispatch policy。它不是在解释算法，而是在“合法数学指令”和“高性能 kernel schedule”之间提供一层架构级查表关系。

### Epilogue config supported
**EN:** The epilogue section mirrors the mainloop discussion by listing dispatch policies, per-SM tile interpretations, and auto-dispatch behavior. It clarifies that selecting a mainloop is not enough; the epilogue must be compatible with the chosen instruction family and tiling.
**CN:** epilogue 部分与 mainloop 呼应，列出了 dispatch policy、每个 SM 的 tile 解释方式以及自动分派逻辑。它强调：只选 mainloop 还不够，epilogue 也必须与指令家族和 tiling 方案匹配。

### Building a Block Scaled Kernel
**EN:** This is the document’s most practical section. It walks through describing tensors, selecting accumulator precision, choosing tile and cluster shapes, configuring the epilogue, and optionally enabling block-scale-factor fusion for the output path.
**CN:** 这是整篇文档中最实用的部分。它按步骤演示如何描述张量、设置累加精度、选择 tile 与 cluster 形状、配置 epilogue，并在需要时为输出路径启用 block-scale-factor fusion。

### Scale Factor Layouts
**EN:** The layout discussion explains that scale-factor tensors follow a specific 512-byte block structure and K-major arrangement across larger tensors. It also points readers to `Sm1xxBlockScaledConfig`, which hides otherwise tedious layout construction details.
**CN:** 该部分说明缩放因子张量遵循特定的 512 字节基本块结构，并在更大张量上采用 K-major 排列。同时引导读者使用 `Sm1xxBlockScaledConfig`，避免手工构造复杂布局。

### Blackwell SM120 GEMMs
**EN:** The SM120 half of the document reuses much of the SM100 mental model but calls out consumer-GPU-specific constraints. The differences are operationally important: fixed cluster size, TN-only layout, required `EpilogueScheduleAuto`, and a smaller set of valid tile/policy combinations.
**CN:** 文档后半部分介绍 SM120，它沿用了 SM100 的很多思路，但特别指出了面向消费级 GPU 的限制。这些差异对实际使用很关键：固定 cluster 大小、只支持 TN 布局、必须使用 `EpilogueScheduleAuto`，以及更少的 tile/policy 组合。

### Cluster Size / Tensor Layout / Kernel Schedule / Tile size
**EN:** These subsections summarize the SM120-specific constraints in deployment terms, making them easy to compare against SM100. They effectively function as a migration checklist for builders targeting RTX 5000-series Blackwell GPUs.
**CN:** 这些小节把 SM120 的限制按部署视角汇总，便于与 SM100 对照。它们实际上构成了一份面向 RTX 5000 系列 Blackwell GPU 的迁移检查清单。

## Key Concepts / 关键概念
- **`tcgen05.mma`** — **EN:** Fifth-generation tensor-core MMA family introduced for Blackwell SM100. **CN:** Blackwell SM100 引入的第五代张量核心 MMA 指令族。
- **Block-scaled GEMM** — **EN:** A GEMM mode where multiplicands are multiplied by per-block scale factors along K. **CN:** 一种沿 K 维为乘数块附加缩放因子的 GEMM 模式。
- **`mx_*` / `nv_*` types** — **EN:** CUTLASS narrow-precision wrappers that encode data type plus scale-factor conventions. **CN:** CUTLASS 的窄精度包装类型，同时编码数据类型和缩放因子约定。
- **Dispatch policy** — **EN:** The tag that selects a valid architecture-specific kernel schedule. **CN:** 用于选择合法架构专用 kernel schedule 的标签类型。
- **Scale-factor layout** — **EN:** The specialized memory layout used to store block-scaling metadata efficiently. **CN:** 用于高效存放 block scaling 元数据的专用内存布局。

## Related Files / 相关文件
- `media/docs/cpp/blackwell.rst` — **EN:** Index page for Blackwell-specific documentation. **CN:** Blackwell 专题文档入口页。
- `media/docs/cpp/gemm_api_3x.md` — **EN:** Explains the collective-builder workflow referenced by this guide. **CN:** 解释本文频繁引用的 collective-builder 工作流。
- `examples/72_blackwell_narrow_precision_gemm/` — **EN:** Dense narrow-precision/block-scaled examples. **CN:** 窄精度与 block-scaled 稠密 GEMM 示例目录。
- `examples/79_blackwell_geforce_gemm/` — **EN:** Example set for SM120/Geforce-style kernels. **CN:** 面向 SM120/GeForce 风格内核的示例目录。

