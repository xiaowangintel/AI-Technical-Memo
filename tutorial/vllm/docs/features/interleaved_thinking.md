# interleaved_thinking.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/interleaved_thinking.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes *interleaved thinking*, a vLLM feature that allows models to insert reasoning steps between tool calls, enabling more sophisticated multi-step decision-making. It covers supported models, a complete usage example with the OpenAI client, and a performance caveat.

**CN:** 本文档介绍*交错思考*功能，允许模型在工具调用之间插入推理步骤，从而实现更复杂的多步决策。内容涵盖支持的模型、使用 OpenAI 客户端的完整示例，以及性能注意事项。

### How It Works / 工作原理

**EN:** With interleaved thinking, the model reasons about a tool's result before deciding on the next action. This enables: chaining multiple tool calls with reasoning in between, making nuanced decisions based on intermediate results, and transparent reasoning for tool selection.

**CN:** 启用交错思考后，模型在决定下一步操作前会对工具结果进行推理，从而实现：在多次工具调用之间插入推理步骤、根据中间结果做出细粒度决策，以及工具选择的透明推理过程。

### Supported Models / 支持的模型

**EN:** `moonshotai/Kimi-K2-Thinking` (parser: `kimi_k2`) and `MiniMaxAI/MiniMax-M2` (parser: `minimax_m2`).

**CN:** `moonshotai/Kimi-K2-Thinking`（解析器：`kimi_k2`）和 `MiniMaxAI/MiniMax-M2`（解析器：`minimax_m2`）。

### Usage Pattern / 使用模式

**EN:** Launch the server with `--tool-call-parser`, `--reasoning-parser`, and `--enable-auto-tool-choice`. In the client, pass `tools` to the chat completion request, execute the called tool, then append the assistant message including `reasoning` to the conversation history before the next turn.

**CN:** 启动服务时指定 `--tool-call-parser`、`--reasoning-parser` 和 `--enable-auto-tool-choice`。在客户端，向对话补全请求中传入 `tools`，执行被调用的工具，然后将包含 `reasoning` 字段的助手消息追加到对话历史中，再进行下一轮。

### Caveat / 注意事项

**EN:** Interleaved thinking increases both token usage and response latency. Users must weigh budget and performance requirements before enabling.

**CN:** 交错思考会增加 token 用量和响应延迟，用户在启用前需权衡预算和性能需求。

## Key Concepts / 关键概念

- **EN:** Interleaved thinking — the model alternates between tool calls and explicit reasoning steps, as opposed to calling tools blindly in sequence. **CN:** 交错思考——模型在工具调用之间插入显式推理步骤，而非盲目地顺序调用工具。
- **EN:** `--reasoning-parser` — server flag specifying which parser extracts reasoning content from model output. **CN:** `--reasoning-parser`——指定从模型输出中提取推理内容的解析器的服务端标志。
- **EN:** `--tool-call-parser` — server flag for parsing structured tool-call outputs. **CN:** `--tool-call-parser`——解析结构化工具调用输出的服务端标志。
- **EN:** `reasoning` field — returned in the assistant message alongside `tool_calls`; must be included in subsequent conversation history. **CN:** `reasoning` 字段——在助手消息中与 `tool_calls` 一同返回，必须包含在后续对话历史中。
- **EN:** Multi-step tool chaining — the core use case: reason → call tool → get result → reason again → call next tool. **CN:** 多步工具链式调用——核心使用场景：推理→调用工具→获取结果→再次推理→调用下一个工具。
