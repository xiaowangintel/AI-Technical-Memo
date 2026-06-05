# registration.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/model/registration.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Explains how vLLM chooses model implementations through a registry and shows two integration paths: built-in registration inside the repository and out-of-tree registration through plugins.
- **CN:** 本文解释了 vLLM 如何通过注册表选择模型实现，并给出两条接入路径：仓库内置注册与仓库外插件注册。
- **EN:** It also highlights practical concerns such as alphabetical ordering of lists, updating supported-model docs, and using lazy imports to avoid CUDA re-initialization errors in forked subprocesses.
- **CN:** 文档还强调了一些实际注意点，例如列表按字母排序、同步更新支持模型文档，以及使用懒加载避免 fork 子进程中的 CUDA 重新初始化错误。

## Key Concepts / 关键概念
- **EN:** Model registry — The registry is the central mechanism that maps architecture names to runnable model classes.
- **CN:** 模型注册表 — 注册表是把架构名称映射到可运行模型类的核心机制。
- **EN:** Built-in model path — In-tree models are added under `vllm/model_executor/models` and registered in `registry.py`.
- **CN:** 内置模型路径 — 仓库内模型需要放到 `vllm/model_executor/models` 下，并在 `registry.py` 中注册。
- **EN:** Plugin registration — External models can be exposed through a plugin entrypoint without modifying vLLM itself.
- **CN:** 插件注册 — 外部模型可以通过插件入口暴露给 vLLM，而无需修改主仓库代码。
- **EN:** Lazy import string — Registering via `"module:Class"` can prevent eager CUDA initialization during plugin loading.
- **CN:** 懒加载字符串 — 使用 `"module:Class"` 形式注册，可避免插件加载时过早初始化 CUDA。
- **EN:** Multimodal requirement — Multimodal models must additionally implement the `SupportsMultiModal` interface.
- **CN:** 多模态要求 — 多模态模型还必须实现 `SupportsMultiModal` 接口。
