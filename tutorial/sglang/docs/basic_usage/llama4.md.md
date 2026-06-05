# llama4.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/llama4.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Llama 4 is Meta's latest generation of open-source LLM model with industry-leading performance. SGLang has supported Llama 4 Scout (109B) and Llama 4 Maverick (400B) since v0.4.5. / 该文档围绕 Llama4 Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Llama 4 is Meta's latest generation of open-source LLM model with industry-leading performance. SGLang has supported Llama 4 Scout (109B) and Llama 4 Maverick (400B) since v0.4.5.
**CN:** 本节围绕 Overview 展开，概述了 Llama, LLM, Meta, Scout 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Llama 4 with SGLang
**EN:** To serve Llama 4 models on 8xH100/H200 GPUs: ``bash python3 -m sglang.launch_server \ --model-path meta-llama/Llama-4-Scout-17B-16E-Instruct \ --tp 8 \ --context-length 1000000 ``
**CN:** 本节围绕 Launch Llama 4 with SGLang 展开，概述了 Llama, --tp, GPUs, serve 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration Tips
**EN:** **OOM Mitigation**: Adjust --context-length to avoid a GPU out-of-memory issue. For the Scout model, we recommend setting this value up to 1M on 8\*H100 and up to 2.5M on 8\*H200.
**CN:** 本节围绕 配置 Tips 展开，概述了 H100, H200, fa3, Add 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE Speculative Decoding
**EN:** **Description**: SGLang has supported Llama 4 Maverick (400B) with EAGLE speculative decoding.
**CN:** 本节围绕 EAGLE 投机解码 展开，概述了 Llama, --speculative-algorithm, --speculative-num-steps, --speculative-eagle-topk 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Accuracy Test with lm_eval
**EN:** This section provides a comparison table for Accuracy Test with lm_eval, covering columns such as Llama-4-Scout-17B-16E-Instruct, Llama-4-Maverick-17B-128E-Instruct and examples such as Official Benchmark, SGLang.
**CN:** 本节围绕 Accuracy Test with lm_eval 展开，概述了 Llama-4-Scout-17B-16E-Instruct, Llama-4-Maverick-17B-128E-Instruct, model, --model 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Llama / **CN:** Llama
- **EN:** model / **CN:** 模型
- **EN:** H100 / **CN:** H100
- **EN:** Scout / **CN:** Scout
- **EN:** Maverick / **CN:** Maverick
- **EN:** --context-length / **CN:** --context-length
- **EN:** Llama-4-Scout-17B-16E-Instruct / **CN:** Llama-4-Scout-17B-16E-Instruct
- **EN:** Llama-4-Maverick-17B-128E-Instruct / **CN:** Llama-4-Maverick-17B-128E-Instruct

## Dependencies / 依赖关系
- `//github.com/meta-llama/llama-models/blob/main/models/llama4/MODEL_CARD.md`
