# generative_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/generative_models.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This overview explains vLLM generative models as implementations of the text-generation interface that turn final hidden states into token log probabilities and then sample outputs.
  **CN:** 该概览说明 vLLM 的生成式模型实现了文本生成接口：先将最终隐藏状态转为 token 对数概率，再通过采样得到输出。
- **EN:** The page mainly organizes the user-facing surface area: runner selection, offline APIs like `LLM.generate`, and online APIs such as Completions and Chat.
  **CN:** 页面主要梳理面向用户的使用入口，包括 runner 选择、`LLM.generate` 等离线 API，以及 Completions、Chat 等在线 API。

## Key Concepts / 关键概念
- **EN:** Generative models are the default path for most LLM-style tasks in vLLM.
  **CN:** 生成式模型是 vLLM 中大多数 LLM 类任务的默认路径。
- **EN:** `--runner generate` / `--runner auto` controls execution mode.
  **CN:** `--runner generate` / `--runner auto` 控制执行模式。
- **EN:** Sampling and generation configuration determine how logprobs become final text.
  **CN:** 采样参数与生成配置决定了如何从 logprobs 得到最终文本。
