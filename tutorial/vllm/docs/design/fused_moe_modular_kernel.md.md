# fused_moe_modular_kernel.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/fused_moe_modular_kernel.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** Explains why the FusedMoE path is decomposed into modular pieces so many kernel combinations can be supported without duplicating end-to-end implementations. **CN:** 说明为何将 FusedMoE 路径拆分为模块化组件，从而在支持多种 kernel 组合时避免重复实现整条流程。
- **EN:** Distinguishes standard contiguous activations from batched expert-major activations, and shows how the dispatch layout affects prepare/finalize logic and expert execution. **CN:** 区分标准连续激活与按 expert 组织的 batched 激活，并说明 dispatch 布局如何影响 prepare/finalize 逻辑与 expert 执行。
- **EN:** Describes the roles of TopKWeightAndReduce, prepare/finalize modules, expert modules, and the orchestration layer that connects them. **CN:** 描述 TopKWeightAndReduce、prepare/finalize 模块、expert 模块以及将它们串联起来的编排层各自承担的职责。
- **EN:** Includes contributor-oriented guidance for adding new modular backends, initializing them from methods, testing compatibility, and profiling performance. **CN:** 包含面向贡献者的扩展说明，涵盖新增模块后端、通过 method 初始化、验证兼容性以及进行性能分析的方法。

## Key Concepts / 关键概念
- **EN:** Activation format: standard tensors use `(M, K)`, while batched tensors use `(num_experts, max_tokens, K)`. **CN:** 激活格式：标准张量使用 `(M, K)`，而 batched 张量使用 `(num_experts, max_tokens, K)`。
- **EN:** TopK delegation: weighting and reduction can be handled inside the expert path or deferred to finalize logic through a strategy object. **CN:** TopK 委托：加权与归约既可以在 expert 路径内部完成，也可以通过策略对象延后到 finalize 阶段完成。
- **EN:** Modular composability: prepare/finalize backends and expert implementations are designed to mix when formats and capabilities align. **CN:** 模块可组合性：当格式与能力匹配时，prepare/finalize 后端与 expert 实现可以自由组合。
- **EN:** Workspace planning: expert modules declare workspace shape and dtype so the top-level kernel can allocate intermediates consistently. **CN:** 工作区规划：expert 模块声明 workspace 的形状与数据类型，便于顶层 kernel 一致地分配中间缓冲区。
- **EN:** Validation workflow: the document points to unit tests, compatibility scripts, and profiling tools as the intended extension path. **CN:** 验证流程：文档将单元测试、兼容性脚本和 profiling 工具作为推荐的扩展验证路径。
