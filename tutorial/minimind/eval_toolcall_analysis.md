# eval_toolcall.py 代码分析

## 1. 文件概述 (File Overview)
`eval_toolcall.py` 是 MiniMind 的 **Tool Calling benchmark / demo** 脚本。它既能直接驱动本地模型 (`backend=local`)，也能调用 OpenAI-compatible API (`backend=api`)。脚本内置工具定义、模拟工具执行器、测试用例与多轮工具调用循环，用于验证模型是否能正确生成 `<tool_call>`、接收 tool result，并继续完成最终回答。

## 2. 依赖说明 (Dependencies)
- `os`, `sys`：修正导入路径。
- `re`, `json`：解析 `<tool_call>` 标签与 JSON 参数。
- `time`, `random`：测速与随机 seed。
- `argparse`, `warnings`：参数解析与静默 warning。
- `torch`：本地模型加载与推理。
- `datetime`：模拟时间类工具返回值。
- `AutoTokenizer`, `AutoModelForCausalLM`, `TextStreamer`：本地文本生成。
- `OpenAI`：API 后端调用。
- `MiniMindConfig`, `MiniMindForCausalLM`：本地 MiniMind 结构。
- `setup_seed`, `get_model_params`：随机种子与参数量输出。

## 3. 逐行代码分析 (Line-by-Line Analysis)
- **第 1-16 行**：导入路径修复、正则/JSON/时间相关模块、本地模型组件与 OpenAI client，并关闭 warning。
- **第 18-27 行**：定义 `TOOLS`，采用 OpenAI function calling 风格的 JSON Schema。每个工具都包含 `name`、`description`、`parameters`，方便模板注入与 API 兼容。
- **第 29-38 行**：定义 `MOCK_RESULTS`，为每个工具提供一个本地 mock executor；这样无需真实外部服务也能完整跑通 tool call 流程。
- **第 40 行**：把 `TOOLS` 转成 `TOOL_MAP`，便于按名称快速索引。
- **第 42-43 行**：`get_tools(names)` 根据名字过滤工具列表。
- **第 45-54 行**：定义 `TEST_CASES`，每个样例都包含 `prompt` 与允许使用的 `tools`，用于自动评测。

### 3.1 本地模型初始化
- **第 57-67 行**：`init_model(args)`：
  - **第 58 行**：统一加载 tokenizer；
  - **第 59-63 行**：若走原生权重，则构造 `MiniMindForCausalLM` 并加载 `.pth`；
  - **第 64-65 行**：否则加载 Transformers 模型；
  - **第 66 行**：打印参数量；
  - **第 67 行**：返回 `fp16 + eval + device` 模型和 tokenizer。

### 3.2 Tool call 解析函数
- **第 70-78 行**：`parse_tool_calls(text)` 用正则 `r'<tool_call>(.*?)</tool_call>'` 抓取所有标签片段，再用 `json.loads()` 解析，适用于本地模型输出的文本协议。
- **第 81-96 行**：`parse_tool_call_from_text(content)` 把 `<tool_call>{...}</tool_call>` 转成更接近 OpenAI `tool_calls` 结构的列表，每个对象都带 `id` 与 `function.name/arguments`。

### 3.3 工具执行器
- **第 99-112 行**：`execute_tool(call, arguments=None)`：
  - 同时兼容本地字典格式与 API 格式；
  - 尝试把参数字符串反序列化成 JSON；
  - 从 `MOCK_RESULTS` 中找到对应函数并执行；
  - 如果工具不存在或执行失败，则返回错误对象而不是抛异常。

### 3.4 本地生成与 API 生成
- **第 115-130 行**：`generate(model, tokenizer, messages, tools, args)`：
  - 用 `apply_chat_template(..., tools=tools, open_thinking=False)` 把工具 schema 注入 prompt；
  - tokenization 后调用 `model.generate()`；
  - 使用 `TextStreamer` 做终端流式打印；
  - 结束后 `decode()` 得到完整回答并可选打印速度。
- **第 133-174 行**：`chat_api(client, messages, tools, args, stream=True)`：
  - 调用 `client.chat.completions.create(...)` 请求 API；
  - **非流式**：直接从 `choice.message.tool_calls` 读取工具调用；若服务端没返回结构化字段，则退回 `parse_tool_call_from_text(content)`；
  - **流式**：逐块拼装 `delta.content` 与 `delta.tool_calls`，并通过 `index` 重建完整工具调用对象；
  - 结束后若仍无结构化工具调用，就再次从文本回退解析。

### 3.5 多轮 Tool Calling 主循环
- **第 177-200 行**：`run_case(...)` 是脚本核心：
  - **第 178 行**：初始化 `messages`；
  - **第 179-184 行**：根据 `backend` 选择本地推理或 API 请求；
  - **第 185-186 行**：若没有工具调用，则说明回答结束，退出循环；
  - **第 187-191 行**：若是 API 后端，把 SDK 返回对象统一转成普通字典格式；
  - **第 192 行**：把 assistant 消息写回上下文；API 模式额外写入结构化 `tool_calls`；
  - **第 193-199 行**：逐个执行工具，打印调用日志与工具结果，再把 `role=tool` 的结果消息回填给模型；
  - 这样模型就能在下一轮根据工具返回值继续推理，直到不再请求工具。

### 3.6 程序入口
- **第 202-220 行**：定义 CLI 参数，包括 `backend`、模型路径、生成参数、API 地址与是否流式。
- **第 222-225 行**：根据 `backend` 初始化本地模型或 OpenAI client。
- **第 226 行**：选择自动测试或手动模式。
- **第 228 行**：自动模式下从 `TEST_CASES` 生成 case 列表；手动模式下默认开放全部工具。
- **第 229-236 行**：遍历 case，设置随机 seed，打印可用工具与问题，然后调用 `run_case(...)` 执行完整 tool-calling 流程。
- **第 239-240 行**：标准入口。

## 4. 关键算法解析 (Key Algorithm Deep-Dive)
### 4.1 Tool definition 注入
`TOOLS` 使用 JSON Schema 风格定义函数签名，再通过 `tokenizer.apply_chat_template(..., tools=tools)` 注入 prompt。这样模型在看到工具说明后，能够输出 `<tool_call>` 或 API `tool_calls`。

### 4.2 Mock execution
`MOCK_RESULTS` 让脚本无需真实天气、汇率或翻译服务，也能完成端到端调试。这种 mock backend 很适合验证“模型会不会调工具”，而不是验证外部服务准确性。

### 4.3 多轮 tool call loop
`while True` 循环的核心模式是：
1. 用户提问；
2. 模型决定是否请求工具；
3. 本地执行 mock tool；
4. 把 tool result 回填到消息历史；
5. 再次让模型继续回答。
这就是典型的 **ReAct / tool-augmented generation** 流程。

### 4.4 Local vs API backend
- **local**：模型输出文本中的 `<tool_call>`，由脚本自己正则解析；
- **api**：优先读取结构化 `tool_calls`，若服务未完全兼容，再回退到文本解析。
这种双通路设计使脚本既适合训练后本地评估，也适合服务部署后的接口验证。

## 5. 调用关系 (Call Graph)
```text
main()
 ├─ init_model(args) / OpenAI(...)
 ├─ 构造 cases
 └─ run_case(prompt, tools, ...)
     ├─ generate(...)            [local]
     │   ├─ tokenizer.apply_chat_template(..., tools=tools)
     │   ├─ tokenizer(...)
     │   ├─ model.generate(...)
     │   └─ parse_tool_calls()
     └─ chat_api(...)            [api]
         ├─ client.chat.completions.create(...)
         ├─ 解析 delta.tool_calls / message.tool_calls
         └─ parse_tool_call_from_text() [fallback]
     └─ execute_tool(...)
         └─ MOCK_RESULTS[name](args)
```
与 tokenizer 的关键连接点是 `apply_chat_template(..., tools=...)`；与模型的关键连接点是 `generate()`；与 API 服务的关键连接点是 `chat.completions.create()`。
