# base.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/training/weight_transfer/base.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This document explains the abstract contract behind vLLM weight transfer backends. It is aimed at developers who want to understand or extend the system with their own transport engine.
- **CN:** 本文解释了 vLLM 权重传输后端背后的抽象契约，面向希望理解或扩展该系统、自定义传输引擎的开发者。

## Key Concepts / 关键概念
- **EN:** `WeightTransferEngine` is parameterized by backend-specific init and update dataclasses.
  **CN:** `WeightTransferEngine` 由后端特定的初始化与更新数据类进行参数化。
- **EN:** Implementations must define initialization, receiving, shutdown, and trainer-side sending behavior.
  **CN:** 实现者必须定义初始化、接收权重、关闭资源以及训练端发送权重的行为。
- **EN:** Request objects stay backend-agnostic by carrying plain dictionaries that are parsed into typed dataclasses.
  **CN:** 请求对象通过普通字典保持后端无关，再解析为强类型数据类。
- **EN:** Custom engines should stream weights incrementally and register with the factory, which supports lazy loading.
  **CN:** 自定义引擎应以增量方式流式加载权重，并注册到支持懒加载的工厂中。
