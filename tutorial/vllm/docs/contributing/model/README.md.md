# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/model/README.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Acts as the landing page for model integration and immediately advises contributors to try the Transformers backend before adding a custom in-tree implementation.
- **CN:** 本文是模型接入文档的入口页，并首先提醒贡献者：在编写仓库内自定义实现之前，先尝试 Transformers 后端是否已经可用。
- **EN:** Its main role is navigational: it frames integration complexity, links to specialized guides, and points readers to GitHub issues or developer Slack for help.
- **CN:** 它的主要作用是导航：先说明接入复杂度，再链接到更细分的指南，并告诉读者遇到问题时可以到 GitHub issue 或开发者 Slack 求助。

## Key Concepts / 关键概念
- **EN:** Transformers backend first — Some decoder models work without native vLLM implementation, so contributors should test that path first.
- **CN:** 优先尝试 Transformers 后端 — 部分 decoder 模型无需原生 vLLM 实现即可工作，因此应先验证这一路径。
- **EN:** Architecture-dependent effort — The amount of work depends on how closely the model matches existing supported designs.
- **CN:** 接入工作量取决于架构 — 接入成本取决于该模型与现有支持架构的相似程度。
- **EN:** Guide map — The page routes readers to basic, registration, test, multimodal, and transcription tutorials.
- **CN:** 指南地图 — 该页面把读者引导到基础接入、注册、测试、多模态和语音转写等子教程。
