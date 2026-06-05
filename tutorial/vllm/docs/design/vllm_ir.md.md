# vllm_ir.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/vllm_ir.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document introduces vLLM IR as a functional intermediate representation that sits between raw torch ops and higher-level vLLM layers, separating semantics from implementation and dispatch. **CN:** 文档将 vLLM IR 定义为一种函数式中间表示，位于原始 torch 算子与更高层 vLLM 层之间，用于把语义与实现、分发机制解耦。
- **EN:** It emphasizes design goals such as eager/compile consistency, transparent kernel selection, low registration boilerplate, out-of-tree extensibility, and interoperability with normal torch/custom ops. **CN:** 文档强调了多个设计目标，包括 eager/compile 一致性、透明的内核选择、低样板注册、树外扩展能力，以及与普通 torch/custom op 的互操作性。
- **EN:** The quick overview shows the lifecycle of an IR op: declare semantics with `@register_op`, register provider implementations with `register_impl`, and choose providers through priority lists from config or platform defaults. **CN:** 概览部分展示了 IR 算子的生命周期：用 `@register_op` 声明语义，用 `register_impl` 注册 provider 实现，再通过配置或平台默认优先级选择具体实现。
- **EN:** A major theme is the compilation pipeline: Dynamo traces IR ops into FX, `maybe_inplace` calls are functionalized and tracked, high-level passes run before lowering, implementations are lowered in, and clone cleanup restores zero-copy opportunities before Inductor codegen. **CN:** 文档的重要主题之一是编译流水线：Dynamo 先把 IR 算子追踪进 FX，`maybe_inplace` 调用会被函数化并记录，高层 pass 在 lowering 前执行，随后把具体实现降入图中，再通过 clone cleanup 恢复零拷贝机会，最后交给 Inductor 生成代码。
- **EN:** The `maybe_inplace` mechanism is presented as the main memory-efficiency lever, letting callers donate activations when they promise not to reuse them afterward. **CN:** `maybe_inplace` 机制被描述为主要的内存优化手段：调用方只要承诺后续不再复用激活，就可以“捐赠”这些张量以换取更高内存效率。
- **EN:** The document also covers provider naming, support predicates, eager-vs-compile parity, out-of-tree integrations, observability hooks, and migration from the older `CustomOp` system. **CN:** 文档还覆盖了 provider 命名、支持谓词、eager 与 compile 的一致性、树外集成、可观测性手段，以及从旧 `CustomOp` 系统迁移的方法。

## Key Concepts / 关键概念
- **EN:** **`IrOp` declaration:** the native PyTorch implementation defines semantics, serves as a fallback implementation, and becomes the reference for testing other kernels. **CN:** **`IrOp` 声明：** 原生 PyTorch 实现既定义了语义，又充当后备实现，同时也是其他内核测试时的参考基准。
- **EN:** **Provider priority lists:** kernel selection is controlled by ordered provider lists, combining user overrides with platform defaults. **CN:** **Provider 优先级列表：** 内核选择由有序 provider 列表控制，并把用户覆盖项与平台默认项组合在一起。
- **EN:** **`maybe_inplace` donation:** this overload permits in-place reuse of activation memory, but using donated tensors afterward is explicitly undefined behavior. **CN:** **`maybe_inplace` 捐赠：** 该重载允许原地复用激活内存，但文档明确指出，之后再使用这些已捐赠张量属于未定义行为。
- **EN:** **Lowering plus clone cleanup:** lowering may insert safety clones for mutating kernels, and a later pass removes unnecessary clones when donated inputs make zero-copy safe. **CN:** **Lowering 与 clone cleanup：** lowering 可能会为会修改输入的内核插入安全 clone，而后续 pass 会在已捐赠输入使零拷贝安全时删除这些多余 clone。
- **EN:** **Mode consistency:** eager dispatch and compile-time lowering use the same provider-selection logic so behavior stays aligned across development and deployment paths. **CN:** **模式一致性：** eager 分发与编译期 lowering 共享相同的 provider 选择逻辑，因此开发与部署路径上的行为能够保持一致。
