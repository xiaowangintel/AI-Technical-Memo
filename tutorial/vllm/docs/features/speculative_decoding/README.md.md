# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/speculative_decoding/README.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
**EN:** This document shows how to use Speculative Decoding with vLLM to reduce inter-token latency under medium-to-low QPS (query per second), memory-bound workloads. Major sections include `vLLM Speculation Methods`, `Method Selection at a Glance`, `Custom Proposer Backend (Experimental)`.
**CN:** 本页属于 vLLM 的推测解码文档集合，围绕 `Speculative Decoding` 说明方法、配置与使用场景。 主要小节包括 `vLLM Speculation Methods`, `Method Selection at a Glance`, `Custom Proposer Backend (Experimental)`。

## Key Concepts / 关键概念
- **EN:** The document is part of vLLM's speculative decoding material, so latency reduction and proposer behavior are recurring themes.
  **CN:** 该文档属于 vLLM 的推测解码资料，降时延与 proposer 行为是反复出现的主题。
- **EN:** The file explicitly marks some support as experimental, so extra validation may be required.
  **CN:** 文件明确将部分能力标记为实验性支持，因此可能需要额外验证。
- **EN:** `vLLM Speculation Methods` narrows the document to a concrete subtopic readers must understand in practice.
  **CN:** `vLLM Speculation Methods` 把文档收束到一个需要实际理解和操作的具体子主题。
- **EN:** `Method Selection at a Glance` narrows the document to a concrete subtopic readers must understand in practice.
  **CN:** `Method Selection at a Glance` 把文档收束到一个需要实际理解和操作的具体子主题。
