# claude_code.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/integrations/claude_code.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This integration guide shows how Claude Code can talk to a vLLM server through vLLM's Anthropic Messages API compatibility layer.
  **CN:** 本集成指南说明 Claude Code 如何通过 vLLM 的 Anthropic Messages API 兼容层连接到 vLLM 服务。
- **EN:** It focuses on end-to-end setup: selecting a tool-calling model, starting `vllm serve`, exporting Anthropic-style environment variables, and handling compatibility caveats.
  **CN:** 它聚焦端到端配置：选择具备 tool calling 能力的模型、启动 `vllm serve`、导出 Anthropic 风格环境变量，并处理兼容性注意事项。

## Key Concepts / 关键概念
- **EN:** Claude Code relies on a Messages-style API, so `ANTHROPIC_BASE_URL` must point to vLLM.
  **CN:** Claude Code 依赖 Messages 风格 API，因此 `ANTHROPIC_BASE_URL` 必须指向 vLLM。
- **EN:** Tool calling requires `--enable-auto-tool-choice` and an appropriate `--tool-call-parser`.
  **CN:** tool calling 需要 `--enable-auto-tool-choice` 与合适的 `--tool-call-parser`。
- **EN:** Older vLLM versions may need a setting tweak to avoid Claude Code prefix-caching regressions.
  **CN:** 旧版 vLLM 可能需要额外设置，以避免 Claude Code 引起的 prefix caching 性能回退。
