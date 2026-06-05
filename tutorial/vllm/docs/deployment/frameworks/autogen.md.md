# autogen.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/autogen.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page shows how to pair vLLM with Microsoft AutoGen for multi-agent applications. It covers installing the required AutoGen packages, starting a compatible vLLM chat-completions server, and using the OpenAI-compatible client path that AutoGen already supports.
- **CN:** 该页面展示了如何将 vLLM 与 Microsoft AutoGen 结合，用于多智能体应用。内容涵盖 AutoGen 所需包的安装、兼容的 vLLM 聊天补全服务启动，以及 AutoGen 已支持的 OpenAI 兼容客户端接入方式。

## Key Concepts / 关键概念
- **EN:** AutoGen consumes vLLM through the OpenAI-compatible API rather than a bespoke integration. **CN:** AutoGen 通过 OpenAI 兼容 API 接入 vLLM，而不是专门的定制集成。
- **EN:** Environment setup is important because both vLLM and modern AutoGen packages must be installed together. **CN:** 环境准备很重要，因为需要同时安装 vLLM 和新版 AutoGen 相关包。
- **EN:** The guide targets agentic workflows, not just single-turn text generation. **CN:** 该指南面向智能体工作流，而不仅是单轮文本生成。
