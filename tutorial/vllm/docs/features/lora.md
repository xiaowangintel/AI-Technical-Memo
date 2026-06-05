# lora.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/lora.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document explains how to use LoRA (Low-Rank Adaptation) adapters with vLLM for both offline inference and online serving. It covers loading adapters, per-request adapter selection, server-side serving, and dynamic runtime loading/unloading via API endpoints.

**CN:** 本文档介绍如何在 vLLM 中使用 LoRA（低秩自适应）适配器进行离线推理和在线服务，内容涵盖适配器加载、按请求选择适配器、服务端配置，以及通过 API 端点动态加载/卸载适配器。

### Offline Inference / 离线推理

**EN:** Instantiate `LLM` with `enable_lora=True`, then pass a `LoRARequest(name, id, path)` to `llm.generate()`. Each request can specify a different adapter, enabling efficient multi-adapter batch serving with minimal overhead.

**CN:** 使用 `enable_lora=True` 实例化 `LLM`，然后向 `llm.generate()` 传入 `LoRARequest(name, id, path)`。每个请求可以指定不同的适配器，以极低开销实现高效的多适配器批次服务。

### Online Serving / 在线服务

**EN:** Use `--enable-lora --lora-modules {name}={path}` when starting `vllm serve`. Adapters appear as separate model IDs under the `/v1/models` endpoint. Requests reference the adapter by its assigned name in the `model` field.

**CN:** 启动 `vllm serve` 时使用 `--enable-lora --lora-modules {name}={path}`。适配器作为独立模型 ID 出现在 `/v1/models` 端点下，请求通过 `model` 字段中的适配器名称来引用它。

### Dynamic Loading / 动态加载

**EN:** Enabled by setting `VLLM_ALLOW_RUNTIME_LORA_UPDATING=True`. Use `POST /v1/load_lora_adapter` and `POST /v1/unload_lora_adapter` to add/remove adapters at runtime. **Warning:** This is a security risk; only use in trusted isolated environments.

**CN:** 通过设置 `VLLM_ALLOW_RUNTIME_LORA_UPDATING=True` 启用。使用 `POST /v1/load_lora_adapter` 和 `POST /v1/unload_lora_adapter` 在运行时动态添加/移除适配器。**警告：** 存在安全风险，仅在受信任的隔离环境中使用。

### Configuration Parameters / 配置参数

**EN:** Key options include `max_loras` (max simultaneously loaded adapters), `max_lora_rank`, and `max_cpu_loras` (CPU-side adapter cache size).

**CN:** 关键选项包括 `max_loras`（最多同时加载的适配器数）、`max_lora_rank`（最大秩），以及 `max_cpu_loras`（CPU 端适配器缓存大小）。

## Key Concepts / 关键概念

- **EN:** LoRA (Low-Rank Adaptation) — a parameter-efficient fine-tuning technique that injects small trainable matrices into existing weight matrices. **CN:** LoRA（低秩自适应）——一种参数高效微调技术，向现有权重矩阵中注入小型可训练矩阵。
- **EN:** `LoRARequest` — the per-request object that identifies which adapter to apply (name, unique int ID, local path). **CN:** `LoRARequest`——标识应用哪个适配器的请求级对象（名称、唯一整数 ID、本地路径）。
- **EN:** Multi-adapter batching — vLLM can serve requests for different LoRA adapters within the same batch efficiently. **CN:** 多适配器批次处理——vLLM 可在同一批次中高效服务不同 LoRA 适配器的请求。
- **EN:** `enable_lora=True` / `--enable-lora` — the flag required for both offline and online LoRA usage. **CN:** `enable_lora=True` / `--enable-lora`——离线和在线 LoRA 使用均需要设置的标志。
- **EN:** `SupportsLoRA` interface — only models implementing this interface are compatible with LoRA adapters in vLLM. **CN:** `SupportsLoRA` 接口——只有实现该接口的模型才与 vLLM 中的 LoRA 适配器兼容。
