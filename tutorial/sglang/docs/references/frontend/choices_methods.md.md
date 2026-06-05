# choices_methods.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/frontend/choices_methods.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This doc describes the choices methods supported by SGLang. The optional choices_method arg determines how options supplied to SGLang's choices primitive are selected. / 该文档围绕 Choices Methods in SGLang 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This doc describes the choices methods supported by SGLang. The optional choices_method arg determines how options supplied to SGLang's choices primitive are selected.
**CN:** 本节围绕 Overview 展开，概述了 choices_method, OpenAI, choices, RuntimeEndpoint 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Token Length Normalized
**EN:** Token length normalized is the default SGLang choices method. It selects the option with the highest average logprob across all of its tokens.
**CN:** 本节围绕 令牌 Length Normalized 展开，概述了 Paris, tokens, choices_method, Token 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Greedy Token Selection
**EN:** Greedy token selection simply selects the option with the highest logprob for its initial token.
**CN:** 本节围绕 Greedy 令牌 Selection 展开，概述了 option, token, def, Name 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Unconditional Likelihood Normalized
**EN:** Unconditional likelihood normalized selects the option with the highest average token logprob once normalized by the unconditional token logprobs, as described in this EleutherAI blogpost.
**CN:** 本节围绕 Unconditional Likelihood Normalized 展开，概述了 LLM, Usage, Paris, token 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** choices_method / **CN:** choices_method
- **EN:** option / **CN:** option
- **EN:** Paris / **CN:** Paris
- **EN:** choices / **CN:** choices
- **EN:** Usage / **CN:** Usage
- **EN:** France / **CN:** France
- **EN:** London / **CN:** London
- **EN:** Berlin / **CN:** Berlin

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
