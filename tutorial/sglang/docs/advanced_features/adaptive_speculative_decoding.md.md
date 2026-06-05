# adaptive_speculative_decoding.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/adaptive_speculative_decoding.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Adaptive speculative decoding lets SGLang adjust speculative_num_steps/speculative_num_draft_tokens at runtime instead of keeping a single fixed value for the whole server lifetime. / 该文档围绕 Adaptive 投机解码 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Adaptive speculative decoding lets SGLang adjust speculative_num_steps/speculative_num_draft_tokens at runtime instead of keeping a single fixed value for the whole server lifetime.
**CN:** 本节围绕 Overview 展开，概述了 Adaptive, speculative_num_steps/speculative_num_draft_tokens, one, lets 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Current support
**EN:** Only --speculative-algorithm EAGLE - Only --speculative-eagle-topk 1 - If either condition is not met, SGLang falls back to static speculative settings
**CN:** 本节围绕 Current support 展开，概述了 EAGLE, met, back, falls 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Why adaptive steps help
**EN:** speculative_num_steps controls how many draft-model autoregressive steps run in each speculative round. In practice, the best value depends on the current workload.
**CN:** 本节围绕 Why adaptive steps help 展开，概述了 num_steps, too, draft, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Design overview
**EN:** The adaptive mechanism has three pieces: - AdaptiveSpeculativeParams: the EMA-based policy - SpecRuntimeState: the per-tier runtime state bundle - AdaptiveController: the coordinator that chooses a tier and activates the matching runtime state At startup, SGLang pre-builds one runtime state per candidate tier.
**CN:** 本节围绕 Design overview 展开，概述了 SpecRuntimeState, state, runtime, tier 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Runtime flow
**EN:** │ │ Backends and CUDA graphs are never swapped mid-round. │ └─────────────────────────────────────────────────────────────────────┘ ``
**CN:** 本节围绕 Runtime flow 展开，概述了 draft, batch, model, current 等要点，并说明相关配置、流程、示例或限制条件。

### Section: How the policy decides
**EN:** After each verify pass, SGLang reads the accepted draft length per request, computes the batch average, smooths it with an exponential moving average (EMA), and switches among the pre-built candidate tiers [1, 3, 7] by default.
**CN:** 本节围绕 How the policy decides 展开，概述了 EMA, move, After, batch 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** speculative-adaptive-config is optional, but the speculative setup still needs to be valid for adaptive mode. Example config: `json ``
**CN:** 本节围绕 Usage 展开，概述了 --speculative-adaptive-config, EAGLE, speculative-adaptive-config, but 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Config file reference
**EN:** This section provides a comparison table for Config file reference, covering columns such as Key, Default, Meaning and examples such as candidate_steps, ema_alpha, update_interval, warmup_batches.
**CN:** 本节围绕 Config file reference 展开，概述了 Extra, candidate_steps,  | , before 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Monitoring
**EN:** This content focuses on Monitoring and highlights tier, active, speculative_num_steps, avg_spec_accept_length.
**CN:** 本节围绕 Monitoring 展开，概述了 tier, active, speculative_num_steps, avg_spec_accept_length 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Tuning tips
**EN:** This content focuses on Tuning tips and highlights Increase, Start, tiers, lower.
**CN:** 本节围绕 Tuning tips 展开，概述了 Increase, Start, tiers, lower 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** tier / **CN:** tier
- **EN:** draft / **CN:** draft
- **EN:** candidate_steps / **CN:** candidate_steps
- **EN:** model / **CN:** 模型
- **EN:** update_interval / **CN:** update_interval
- **EN:** speculative_num_steps / **CN:** speculative_num_steps
- **EN:** EMA / **CN:** EMA
- **EN:** one / **CN:** one

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
