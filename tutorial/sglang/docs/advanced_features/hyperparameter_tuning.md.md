# hyperparameter_tuning.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/hyperparameter_tuning.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Achieving high throughput for offline batch inference Achieving a large batch size is the most important thing for attaining high throughput in offline batch inference. / 该文档围绕 Hyperparameter Tuning 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Achieving high throughput for offline batch inference
**EN:** Achieving a large batch size is the most important thing for attaining high throughput in offline batch inference. When the server is running at full load in a steady state, look for the following in the log: ``Decode batch.
**CN:** 本节围绕 Achieving high throughput for offline batch inference 展开，概述了 --mem-fraction-static, memory, OOM, CUDA 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** --mem-fraction-static / **CN:** --mem-fraction-static
- **EN:** memory / **CN:** memory
- **EN:** OOM / **CN:** OOM
- **EN:** CUDA / **CN:** CUDA
- **EN:** cache / **CN:** 缓存
- **EN:** mem-fraction-static / **CN:** mem-fraction-static
- **EN:** usage / **CN:** usage
- **EN:** requests / **CN:** requests

## Dependencies / 依赖关系
- `../advanced_features/sgl_model_gateway.md`
