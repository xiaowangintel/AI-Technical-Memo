# model_resolution.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/configuration/model_resolution.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This note explains how vLLM resolves Hugging Face models: it inspects the `architectures` field in `config.json` and maps that to a registered implementation. It then lists common failure modes such as missing architecture metadata, unofficial naming, or ambiguous names.

**EN:** The recommended fix is explicit override through `hf_overrides`, making this page a practical troubleshooting reference for model-loading mismatches.

**CN:** 这篇说明解释了 vLLM 如何解析 Hugging Face 模型：它会读取 `config.json` 中的 `architectures` 字段，并据此匹配已注册的实现。随后文档列出常见失败原因，例如缺少架构字段、非官方命名或同名架构歧义。

**CN:** 推荐的解决方法是通过 `hf_overrides` 显式覆盖，因此这页本质上是一份模型加载排障参考。

## Key Concepts / 关键概念
- **EN:** Architecture resolution — Maps HF model metadata to vLLM implementations.
- **CN:** 架构解析 — 把 HF 模型元数据映射到 vLLM 的实现。
- **EN:** hf_overrides — Lets users override config fields when automatic resolution fails.
- **CN:** hf_overrides — 当自动解析失败时，可用它覆盖配置字段。
- **EN:** Model ambiguity — Highlights how inconsistent naming can break automatic loading.
- **CN:** 模型歧义 — 说明命名不一致会如何破坏自动加载。
