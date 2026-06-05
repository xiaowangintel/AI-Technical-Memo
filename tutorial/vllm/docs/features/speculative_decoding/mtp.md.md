# mtp.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/speculative_decoding/mtp.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
**EN:** MTP is a speculative decoding method where the target model includes native multi-token prediction capability. Unlike draft-model-based methods, you do not need to provide a separate draft model. Major sections include `Gemma 4 Assistant Models`, `Offline Example`, `Online Example`.
**CN:** 本页属于 vLLM 的推测解码文档集合，围绕 `MTP (Multi-Token Prediction)` 说明方法、配置与使用场景。 主要小节包括 `Gemma 4 Assistant Models`, `Offline Example`, `Online Example`。

## Key Concepts / 关键概念
- **EN:** The document is part of vLLM's speculative decoding material, so latency reduction and proposer behavior are recurring themes.
  **CN:** 该文档属于 vLLM 的推测解码资料，降时延与 proposer 行为是反复出现的主题。
- **EN:** `Gemma 4 Assistant Models` narrows the document to a concrete subtopic readers must understand in practice.
  **CN:** `Gemma 4 Assistant Models` 把文档收束到一个需要实际理解和操作的具体子主题。
- **EN:** `Offline Example` uses concrete examples to bridge configuration and execution.
  **CN:** `Offline Example` 通过具体示例衔接配置与实际执行。
- **EN:** `Online Example` uses concrete examples to bridge configuration and execution.
  **CN:** `Online Example` 通过具体示例衔接配置与实际执行。
