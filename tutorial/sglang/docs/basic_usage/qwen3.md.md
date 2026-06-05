# qwen3.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/qwen3.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang has supported Qwen3-Next-80B-A3B-Instruct and Qwen3-Next-80B-A3B-Thinking since this PR. Launch Qwen3-Next with SGLang To serve Qwen3-Next models on 4xH100/H200 GPUs: Configuration Tips. / 该文档围绕 Qwen3-Next Usage 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang has supported Qwen3-Next-80B-A3B-Instruct and Qwen3-Next-80B-A3B-Thinking since this PR.
**CN:** 本节围绕 Overview 展开，概述了 Qwen3-Next-80B-A3B-Instruct, Qwen3-Next-80B-A3B-Thinking, since, supported 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Launch Qwen3-Next with SGLang
**EN:** To serve Qwen3-Next models on 4xH100/H200 GPUs: ``bash python3 -m sglang.launch_server --model Qwen/Qwen3-Next-80B-A3B-Instruct --tp 4 ``
**CN:** 本节围绕 Launch Qwen3-Next with SGLang 展开，概述了 Qwen3-Next, --tp, Qwen, GPUs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration Tips
**EN:** max-mamba-cache-size: Adjust --max-mamba-cache-size to increase mamba cache space and max running requests capability. It will decrease KV cache space as a trade-off.
**CN:** 本节围绕 配置 Tips 展开，概述了 float32, cache, bfloat16, --max-mamba-cache-size 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Mamba Radix Cache
**EN:** SGLang supports prefix caching for Qwen3-Next models named MambaRadixCache, which improves inference speed by reusing computation results.
**CN:** 本节围绕 Mamba Radix 缓存 展开，概述了 MambaRadixCache, extra_buffer, two, There 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE Speculative Decoding
**EN:** **Description**: SGLang has supported Qwen3-Next models with EAGLE speculative decoding. **Usage**: Add arguments --speculative-algorithm, --speculative-num-steps, --speculative-eagle-topk and --speculative-num-draft-tokens to enable this feature.
**CN:** 本节围绕 EAGLE 投机解码 展开，概述了 --speculative-num-steps, --speculative-eagle-topk, --speculative-num-draft-tokens, Add 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Qwen3-Next / **CN:** Qwen3-Next
- **EN:** float32 / **CN:** float32
- **EN:** MambaRadixCache / **CN:** MambaRadix缓存
- **EN:** mamba / **CN:** mamba
- **EN:** cache / **CN:** 缓存
- **EN:** models / **CN:** 模型s
- **EN:** bfloat16 / **CN:** bfloat16
- **EN:** Qwen3-Next-80B-A3B-Instruct / **CN:** Qwen3-Next-80B-A3B-Instruct

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
