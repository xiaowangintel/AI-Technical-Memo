# custom_op.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/custom_op.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document defines `CustomOp` as vLLM’s abstraction for backend-dependent operator dispatch and as a registration point for both in-tree and out-of-tree implementations. **CN:** 本文将 `CustomOp` 定义为 vLLM 中用于后端相关算子分发的抽象层，同时也是树内与树外实现的注册入口。
- **EN:** It explains that a custom op can route `forward()` to platform-specific methods such as `forward_cuda`, `forward_cpu`, `forward_xpu`, or `forward_oot`, with `forward_native` as fallback. **CN:** 文档说明一个 custom op 可以把 `forward()` 路由到 `forward_cuda`、`forward_cpu`、`forward_xpu` 或 `forward_oot` 等平台特定实现，并以 `forward_native` 作为兜底。
- **EN:** A major theme is configuration-driven enablement: whether a custom op is active depends on `compilation_config.custom_ops`, with defaults varying by compile mode and backend. **CN:** 文档的重要主题之一是“由配置决定是否启用”：custom op 是否生效取决于 `compilation_config.custom_ops`，而默认行为又随编译模式和后端变化。
- **EN:** The write-up also serves as a catalog, listing many registered op categories such as attention, activation, linear layers, MoE, quantization, and multimodal encoder pieces. **CN:** 这篇文档同时也是一份清单，列出了 attention、activation、linear、MoE、量化以及多模态编码器等多个已注册算子类别。
- **EN:** The second half is practical guidance for contributors and hardware-plugin authors on how to implement and register new ops without patching core vLLM code. **CN:** 后半部分则面向贡献者与硬件插件作者，说明如何在不修改 vLLM 核心代码的前提下实现并注册新算子。

## Key Concepts / 关键概念
- **EN:** **Dual registries** — `CustomOp` keeps separate registries for built-in ops and OOT plugin replacements. **CN:** **双注册表**——`CustomOp` 分别维护内置算子与 OOT 插件替换算子的注册表。
- **EN:** **Platform dispatch chain** — the call path chooses `forward_cpu/cuda/hip/xpu/tpu/oot`, then falls back to `forward_native` if needed. **CN:** **平台分发链**——调用路径会优先选择 `forward_cpu/cuda/hip/xpu/tpu/oot`，必要时再回退到 `forward_native`。
- **EN:** **Fine-grained enablement** — users can globally enable all, disable all, or explicitly add/remove named ops through `custom_ops` config. **CN:** **细粒度启用控制**——用户可通过 `custom_ops` 配置全量启用、全量禁用，或对命名算子做显式增删。
- **EN:** **Compile interaction** — when Inductor compile is active, many custom ops default to disabled so compiler-generated kernels can replace them. **CN:** **与编译流程的交互**——当 Inductor 编译启用时，许多 custom op 默认会被关闭，以便由编译器生成的 kernel 接管。
- **EN:** **Multimodal exceptions** — some multimodal ops may be forcibly enabled for performance, showing that dispatch policy is not purely global. **CN:** **多模态例外**——部分多模态算子可被强制启用以保障性能，这说明分发策略并非完全是全局静态的。
- **EN:** **OOT plugin integration** — hardware vendors can override built-in ops by registering `forward_oot()` implementations through the plugin system. **CN:** **OOT 插件接入**——硬件厂商可通过插件系统注册 `forward_oot()` 实现，从而覆盖内置算子。
