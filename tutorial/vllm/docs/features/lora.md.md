# lora.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/lora.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document explains how vLLM serves LoRA adapters on top of a base model for both offline inference and OpenAI-compatible online serving, starting from adapter download and `enable_lora=True` initialization.
  **CN:** 本文说明 vLLM 如何在基座模型之上提供 LoRA adapter，覆盖离线推理与 OpenAI 兼容在线服务，并从下载 adapter 与设置 `enable_lora=True` 讲起。
- **EN:** The offline path centers on `LoRARequest(name, id, path)`, letting each generate call select an adapter with low per-request overhead.
  **CN:** 离线路径以 `LoRARequest(name, id, path)` 为核心，使每次生成请求都能以较低开销选择对应 adapter。
- **EN:** For server mode, the guide covers static registration through `--lora-modules`, model discovery via `/v1/models`, and request-time selection by using the LoRA adapter name as the `model` field.
  **CN:** 对服务模式，文档说明了通过 `--lora-modules` 静态注册、用 `/v1/models` 查看模型列表，以及在请求中把 LoRA 名称当作 `model` 字段来选择 adapter。
- **EN:** A major section is runtime management: adapters can be loaded or unloaded dynamically through dedicated endpoints or resolver plugins, but the text repeatedly warns that runtime updating and remote resolution are insecure outside trusted environments.
  **CN:** 文档的重要部分是运行时管理：adapter 可以通过专用接口或 resolver 插件动态加载/卸载，但也反复警告运行时更新与远程解析只适用于可信环境，在生产环境存在安全风险。
- **EN:** The guide also documents newer and more specialized capabilities: `load_inplace` for replacing weights under the same name, JSON `--lora-modules` with `base_model_name`, mixed 2D/3D MoE LoRA formats, model-card lineage, multimodal default LoRAs, and selective module targeting.
  **CN:** 文档还覆盖了较新的专项能力：同名热替换的 `load_inplace`、带 `base_model_name` 的 JSON `--lora-modules`、2D/3D MoE LoRA 混合格式、模型卡 lineage、多模态默认 LoRA，以及对特定模块的定向应用。
- **EN:** Operational advice is practical: choose `max_lora_rank` close to the real maximum rank to avoid wasted memory, and use `--lora-target-modules` when only certain layers need adaptation.
  **CN:** 运维建议也很实用：`max_lora_rank` 应尽量贴近实际最大 rank 以避免浪费显存，并可借助 `--lora-target-modules` 只对特定层启用 LoRA。

## Key Concepts / 关键概念
- **EN:** `enable_lora=True` activates LoRA support for models implementing `SupportsLoRA`.  
  **CN:** `enable_lora=True` 为实现 `SupportsLoRA` 的模型开启 LoRA 支持。
- **EN:** `LoRARequest` carries adapter name, unique integer ID, and adapter path.  
  **CN:** `LoRARequest` 包含 adapter 名称、唯一整数 ID 与 adapter 路径。
- **EN:** Static serving uses `--lora-modules`; dynamic serving uses `/v1/load_lora_adapter`, `/v1/unload_lora_adapter`, or resolver plugins.  
  **CN:** 静态服务使用 `--lora-modules`；动态服务使用 `/v1/load_lora_adapter`、`/v1/unload_lora_adapter` 或 resolver 插件。
- **EN:** Runtime LoRA updating is flexible but security-sensitive.  
  **CN:** 运行时 LoRA 更新很灵活，但安全敏感。
- **EN:** Mixed MoE LoRA formats require correct `is_3d_lora_weight` declarations or outputs may be silently wrong.  
  **CN:** 混合 MoE LoRA 格式要求正确声明 `is_3d_lora_weight`，否则结果可能静默出错。
- **EN:** Multimodal default LoRAs can auto-apply adapters based on modality presence, with current one-LoRA-per-prompt limits.  
  **CN:** 多模态默认 LoRA 可根据模态自动应用 adapter，但当前仍受“每个 prompt 仅一个 LoRA”限制。
