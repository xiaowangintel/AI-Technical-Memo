# glm45.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/basic_usage/glm45.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Launch GLM-4.5 / GLM-4.6 / GLM-4.7 with SGLang To serve GLM-4.5 / GLM-4.6 FP8 models on 8xH100/H200 GPUs: EAGLE Speculative Decoding **Description**: SGLang has supported GLM-4.5 / GLM-4.6 models. / 该文档围绕 Glm45 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Launch GLM-4.5 / GLM-4.6 / GLM-4.7 with SGLang
**EN:** To serve GLM-4.5 / GLM-4.6 FP8 models on 8xH100/H200 GPUs: ``bash python3 -m sglang.launch_server --model zai-org/GLM-4.6-FP8 --tp 8 ``
**CN:** 本节围绕 Launch GLM-4.5 / GLM-4.6 / GLM-4.7 with SGLang 展开，概述了 GLM-4.5, FP8, --tp, GPUs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE Speculative Decoding
**EN:** **Description**: SGLang has supported GLM-4.5 / GLM-4.6 models with EAGLE speculative decoding. **Usage**: Add arguments --speculative-algorithm, --speculative-num-steps, --speculative-eagle-topk and --speculative-num-draft-tokens to enable this feature.
**CN:** 本节围绕 EAGLE 投机解码 展开，概述了 EAGLE, --speculative-algorithm, --speculative-num-steps, --speculative-eagle-topk 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Thinking Budget for GLM-4.5 / GLM-4.6
**EN:** **Note**: For GLM-4.7, --tool-call-parser should be set to glm47, for GLM-4.5 and GLM-4.6, it should be set to glm45. In SGLang, we can implement thinking budget with CustomLogitProcessor.
**CN:** 本节围绕 Thinking Budget for GLM-4.5 / GLM-4.6 展开，概述了 Glm4MoeThinkingBudgetLogitProcessor, import, GLM-4.6, CustomLogitProcessor 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** GLM-4.6 / **CN:** GLM-4.6
- **EN:** GLM-4.5 / **CN:** GLM-4.5
- **EN:** EAGLE / **CN:** EAGLE
- **EN:** glm45 / **CN:** glm45
- **EN:** Glm4MoeThinkingBudgetLogitProcessor / **CN:** Glm4MoeThinkingBudgetLogitProcessor
- **EN:** Launch / **CN:** Launch
- **EN:** import / **CN:** import
- **EN:** GLM-4.7 / **CN:** GLM-4.7

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
