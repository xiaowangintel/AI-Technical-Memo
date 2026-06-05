# dify.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/deployment/frameworks/dify.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document explains how to use Dify with vLLM as the model provider. It combines vLLM server setup with Dify deployment prerequisites such as Docker and Docker Compose, then shows the model configuration values needed inside the Dify UI.
- **CN:** 本文档说明如何将 Dify 与 vLLM 作为模型提供方一起使用。内容结合了 vLLM 服务端准备，以及 Dify 所需的 Docker、Docker Compose 等部署前提，随后给出 Dify 界面中需要填写的模型配置项。

## Key Concepts / 关键概念
- **EN:** Dify treats vLLM as an OpenAI-style model endpoint. **CN:** Dify 将 vLLM 视作一个 OpenAI 风格的模型端点。
- **EN:** The workflow spans both platform installation and in-product model registration. **CN:** 该流程同时覆盖平台安装与产品内部的模型注册。
- **EN:** Configuration focuses on endpoint URL, model identity, and completion mode. **CN:** 配置重点在于端点 URL、模型标识以及补全模式。
