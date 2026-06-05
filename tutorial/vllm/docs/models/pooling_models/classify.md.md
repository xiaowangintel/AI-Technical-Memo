# classify.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/pooling_models/classify.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page defines sequence classification as a pooling task that predicts one label distribution for each input sequence.
  **CN:** 本页将序列分类定义为一种 pooling 任务：为每个输入序列预测一组标签分布。
- **EN:** It bridges concepts to usage by mapping the task to offline APIs, online endpoints, supported models, and conversion workflows.
  **CN:** 文档将概念与用法连接起来，明确了该任务对应的离线 API、在线端点、支持模型和模型转换流程。

## Key Concepts / 关键概念
- **EN:** Primary offline entry points are `LLM.classify(...)` and `LLM.encode(..., pooling_task="classify")`.
  **CN:** 主要离线入口是 `LLM.classify(...)` 与 `LLM.encode(..., pooling_task="classify")`。
- **EN:** Online serving uses `/classify` or the generic `/pooling` endpoint.
  **CN:** 在线服务可使用 `/classify` 或通用 `/pooling` 端点。
- **EN:** Examples span standard text classification and model conversion via `--convert classify`.
  **CN:** 示例覆盖标准文本分类以及通过 `--convert classify` 的模型转换。
