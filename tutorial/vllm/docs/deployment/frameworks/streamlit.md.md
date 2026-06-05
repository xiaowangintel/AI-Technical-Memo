# streamlit.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/streamlit.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document explains how to build a simple Streamlit application on top of a vLLM server. It focuses on package installation, starting a supported chat model, and using Streamlit plus the OpenAI client interface to create an interactive front end.
- **CN:** 本文档说明如何在 vLLM 服务之上构建一个简单的 Streamlit 应用。重点是依赖安装、启动支持的聊天模型，以及结合 Streamlit 和 OpenAI 客户端接口构建交互式前端。

## Key Concepts / 关键概念
- **EN:** Streamlit acts as the presentation layer while vLLM remains the inference engine. **CN:** Streamlit 充当前端展示层，而 vLLM 保持为推理引擎。
- **EN:** The guide favors rapid prototyping of chat applications. **CN:** 该指南偏向聊天应用的快速原型开发。
- **EN:** OpenAI-compatible access makes it easy to reuse standard client code in the app. **CN:** OpenAI 兼容接口让应用可以直接复用标准客户端代码。
