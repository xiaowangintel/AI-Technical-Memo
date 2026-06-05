# attention_backends.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/attention_backends.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This file is an auto-generated capability matrix rather than a narrative design note, so its main value is as a reference for backend selection and validation behavior. **CN:** 这份文档是自动生成的能力矩阵，而不是传统设计说明，因此其主要价值在于作为后端选择与校验行为的参考表。
- **EN:** It documents two configuration surfaces—simple flags and structured `AttentionConfig`—for both CLI and Python users. **CN:** 文档为 CLI 和 Python 用户同时说明了两类配置入口：简化 flag 与结构化 `AttentionConfig`。
- **EN:** A key operational point is the distinction between manual selection and automatic selection: explicit choice triggers validation, while implicit choice walks priority lists until a compatible backend is found. **CN:** 一个关键的运维要点是手动选择与自动选择的区别：显式指定会先做兼容性校验，而默认模式会沿优先级列表寻找首个可用后端。
- **EN:** The document separates standard attention and MLA attention, and further distinguishes MLA prefill backends from decode backends. **CN:** 文档将标准 attention 与 MLA attention 分开描述，并进一步把 MLA 的 prefill 后端和 decode 后端拆分说明。
- **EN:** The bulk of the file is a feature matrix covering dtype, KV cache dtype, block size, head size, sink support, multimodal prefix support, DCP, and compute capability. **CN:** 文档主体是能力矩阵，覆盖 dtype、KV cache dtype、block size、head size、sink 支持、多模态 prefix、DCP 以及计算能力要求等信息。

## Key Concepts / 关键概念
- **EN:** **Selection interfaces** — users can choose backends via `--attention-backend`, `--attention-config.backend`, or `AttentionConfig` in Python. **CN:** **选择接口**——用户可通过 `--attention-backend`、`--attention-config.backend` 或 Python 中的 `AttentionConfig` 指定后端。
- **EN:** **Validation-first manual mode** — an incompatible manually selected backend raises an error with a concrete reason list. **CN:** **手动模式先校验**——若手动指定的后端不兼容，会直接报错并给出具体原因列表。
- **EN:** **Priority-based auto mode** — when unspecified, vLLM iterates through hardware-specific priority orders to pick the first valid backend. **CN:** **按优先级自动选择**——未显式指定时，vLLM 会按硬件相关的优先级顺序挑选首个兼容后端。
- **EN:** **Standard vs MLA paths** — MLA uses a different backend ecosystem and even splits prefill and decode backend choices. **CN:** **标准与 MLA 路径**——MLA 使用不同的后端生态，且 prefill 与 decode 还分别选择不同后端。
- **EN:** **Hardware-aware support** — backend availability depends on GPU generation (for example, Blackwell vs Ampere/Hopper) and required compute capability. **CN:** **硬件感知支持**——后端是否可用取决于 GPU 代际（如 Blackwell 与 Ampere/Hopper）以及所需计算能力。
- **EN:** **Feature legend** — the tables encode fine-grained support such as sink attention, non-causal attention, sparse MLA, multimodal prefix, and DCP. **CN:** **特性图例**——表格细粒度表达了 sink attention、非因果 attention、稀疏 MLA、多模态 prefix 与 DCP 等支持情况。
