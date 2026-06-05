# interleaved_thinking.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/interleaved_thinking.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page introduces interleaved thinking as a tool-calling capability where the model reasons between tool invocations instead of producing a single uninterrupted response.
  **CN:** 本文介绍了 interleaved thinking：模型不会一次性连续作答，而是在多次工具调用之间进行推理。
- **EN:** The document focuses on the practical value of the feature: reacting to tool outputs, chaining calls step by step, and making more nuanced decisions from intermediate results.
  **CN:** 文档强调该能力的实际价值：根据工具输出继续推理、逐步串联多次调用，并基于中间结果做更细致的决策。
- **EN:** It also calls out the cost trade-off explicitly—higher token usage and latency—so enabling the feature is positioned as a budget/performance decision rather than a free upgrade.
  **CN:** 文档也明确指出代价：token 开销和响应延迟都会增加，因此这更像是预算与性能之间的取舍，而不是“免费升级”。
- **EN:** Support is currently limited and concrete: the page lists `moonshotai/Kimi-K2-Thinking` and `MiniMaxAI/MiniMax-M2`, each mapped to a reasoning parser name.
  **CN:** 当前支持范围也写得很具体：文档列出了 `moonshotai/Kimi-K2-Thinking` 与 `MiniMaxAI/MiniMax-M2`，并给出各自对应的 reasoning parser 名称。
- **EN:** The example shows the end-to-end workflow: start `vllm serve` with tool and reasoning parsers enabled, call the OpenAI-compatible chat API with `tools`, append both tool calls and returned reasoning to the message history, then make a second completion request after tool execution.
  **CN:** 示例展示了完整流程：先用启用 tool/reasoning parser 的参数启动 `vllm serve`，再通过 OpenAI 兼容聊天接口传入 `tools`，把模型返回的工具调用和 reasoning 一并加入消息历史，执行工具后再发起第二次 completion 请求。

## Key Concepts / 关键概念
- **EN:** Interleaved thinking = reasoning steps inserted between tool calls.  
  **CN:** Interleaved thinking = 在工具调用之间插入推理步骤。
- **EN:** Main benefit: better multi-step tool use based on intermediate observations.  
  **CN:** 核心收益：基于中间观察结果更好地完成多步工具调用。
- **EN:** Main cost: increased token consumption and latency.  
  **CN:** 主要代价：token 消耗增加、延迟上升。
- **EN:** Requires compatible models plus matching `--tool-call-parser` / `--reasoning-parser` setup.  
  **CN:** 需要兼容模型，并正确配置 `--tool-call-parser` / `--reasoning-parser`。
- **EN:** Client workflow must preserve both tool outputs and reasoning in conversation state.  
  **CN:** 客户端流程必须把工具输出和 reasoning 一起保存在对话状态中。
