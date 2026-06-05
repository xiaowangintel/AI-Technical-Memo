# chatbox.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/chatbox.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This guide shows how to use Chatbox as a desktop front end for a vLLM server. It walks through starting a supported chat model in vLLM and then mapping Chatbox settings such as API host, path, and model name to the OpenAI-compatible endpoint.
- **CN:** 本指南说明如何把 Chatbox 作为 vLLM 服务的桌面前端使用。内容包括启动支持的 vLLM 聊天模型，并将 Chatbox 中的 API Host、路径和模型名等配置映射到 OpenAI 兼容端点。

## Key Concepts / 关键概念
- **EN:** Chatbox is used as a client UI, while vLLM remains the inference backend. **CN:** Chatbox 用作客户端界面，而 vLLM 仍然是推理后端。
- **EN:** The integration depends on OpenAI-compatible API mode and correct endpoint fields. **CN:** 集成依赖 OpenAI 兼容 API 模式以及正确的端点字段配置。
- **EN:** The guide is aimed at rapid local testing of chat models through a desktop app. **CN:** 该指南面向通过桌面应用快速测试聊天模型。
