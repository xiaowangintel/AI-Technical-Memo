# env_vars.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/configuration/env_vars.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This page documents environment-based configuration for vLLM and embeds the variable definitions from code. It also warns users not to confuse internal service variables such as `VLLM_PORT` and `VLLM_HOST_IP` with external API server binding options.

**EN:** The Kubernetes warning is especially important: service names can auto-generate environment variables that accidentally collide with vLLM’s `VLLM_` namespace.

**CN:** 该页记录 vLLM 的环境变量配置方式，并直接从代码中嵌入变量定义。同时特别提醒用户，不要把 `VLLM_PORT`、`VLLM_HOST_IP` 这类内部变量误当作外部 API 服务绑定参数。

**CN:** 其中 Kubernetes 的提醒尤为关键：服务名自动生成的环境变量可能与 vLLM 的 `VLLM_` 命名空间发生冲突。

## Key Concepts / 关键概念
- **EN:** Environment configuration — Explains the environment-variable layer of vLLM setup.
- **CN:** 环境变量配置 — 说明 vLLM 配置体系中的环境变量层。
- **EN:** Internal vs external ports — Clarifies that some variables are for internal coordination, not API exposure.
- **CN:** 内部端口与外部端口 — 澄清某些变量用于内部协调，而不是 API 对外监听。
- **EN:** Kubernetes collision risk — Warns about accidental conflicts from service-generated env vars.
- **CN:** Kubernetes 冲突风险 — 提醒服务自动注入环境变量可能造成命名冲突。
