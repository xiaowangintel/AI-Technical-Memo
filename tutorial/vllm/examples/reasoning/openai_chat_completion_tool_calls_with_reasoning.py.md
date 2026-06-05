# openai_chat_completion_tool_calls_with_reasoning.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/reasoning/openai_chat_completion_tool_calls_with_reasoning.py`
- **Repository**: vllm-project/vllm
- **Purpose**: An example demonstrates how to use tool calling with reasoning models like QwQ-32B / 演示带推理信息的 OpenAI 兼容响应。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
An example demonstrates how to use tool calling with reasoning models 
like QwQ-32B. The reasoning will not be parsed by the tool 
calling process; only the final output will be parsed.

To run this example, you need to start the vLLM server with both 
the reasoning parser and tool calling enabled.

```bash
vllm serve Qwen/QwQ-32B \
     --reasoning-parser deepseek_r1 \
     --enable-auto-tool-choice --tool-call-parser hermes
     
```

"""
```
**EN:** An example demonstrates how to use tool calling with reasoning models like QwQ-32B.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from openai import OpenAI
```
**EN:** This block loads helper libraries such as openai.
**CN:** 这一部分加载 openai 等辅助库。

### Function: get_current_weather
```python
def get_current_weather(city: str, state: str, unit: "str"):
    return (
        "The weather in Dallas, Texas is 85 degrees fahrenheit. It is "
        "partly cloudly, with highs in the 90's."
    )
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as city, state, and unit. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 city、state，以及 unit 等参数。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
available_tools = {"get_current_weather": get_current_weather}

# Modify OpenAI's API key and API base to use vLLM's API server.
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

properties = {
    "city": {
        "type": "string",
        "description": "The city to find the weather for, e.g. 'San Francisco'",
    },
    "state": {
# ... key logic omitted for brevity ...
    {"role": "assistant", "content": "I'm doing well! How can I help you?"},
    {
        "role": "user",
        "content": (
            "Can you tell me what the temperate will be in Dallas, in fahrenheit?"
        ),
    },
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as available_tools, openai_api_key, openai_api_base, properties, tools, and messages.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 available_tools、openai_api_key、openai_api_base、properties、tools，以及 messages 等变量。

### Function: extract_reasoning_and_calls
```python
def extract_reasoning_and_calls(chunks: list):
    reasoning = ""
    tool_call_idx = -1
    arguments = []
    function_names = []
    for chunk in chunks:
        if chunk.choices[0].delta.tool_calls:
            tool_call = chunk.choices[0].delta.tool_calls[0]
            if tool_call.index != tool_call_idx:
                tool_call_idx = chunk.choices[0].delta.tool_calls[0].index
                arguments.append("")
                function_names.append("")

            if tool_call.function:
                if tool_call.function.name:
                    function_names[tool_call_idx] = tool_call.function.name

                if tool_call.function.arguments:
                    arguments[tool_call_idx] += tool_call.function.arguments
        else:
            if hasattr(chunk.choices[0].delta, "reasoning"):
                reasoning += chunk.choices[0].delta.reasoning
    return reasoning, arguments, function_names
```
**EN:** This function sends a request and handles the reply. It works with parameters such as chunks. Key operations include hasattr, arguments.append, and function_names.append. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。它会处理 chunks 等参数。关键操作包括 hasattr、arguments.append，以及 function_names.append。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    client = OpenAI(
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    models = client.models.list()
    model = models.data[0].id

    print("---------Full Generate With Automatic Function Calling-------------")
    tool_calls = client.chat.completions.create(
        messages=messages, model=model, tools=tools
    )
    print(f"reasoning: {tool_calls.choices[0].message.reasoning}")
    print(f"function name: {tool_calls.choices[0].message.tool_calls[0].function.name}")
    print(
        f"function arguments: "
        f"{tool_calls.choices[0].message.tool_calls[0].function.arguments}"
    )

    # ... key logic omitted for brevity ...
        messages=messages,
        model=model,
        tools=tools,
        tool_choice={"type": "function", "function": {"name": "get_current_weather"}},
        stream=True,
    )

    chunks = list(tool_calls_stream)

    reasoning, arguments, function_names = extract_reasoning_and_calls(chunks)
    print(f"reasoning: {reasoning}")
    print(f"function name: {function_names[0]}")
    print(f"function arguments: {arguments[0]}")
    print("\n\n")
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print, client.chat.completions.create, list, extract_reasoning_and_calls, and OpenAI.
**CN:** 该函数编排端到端工作流。关键操作包括 print、client.chat.completions.create、list、extract_reasoning_and_calls，以及 OpenAI。

### Entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Tool calling / 工具调用**: Model outputs are mapped to callable tools or structured action requests. / 模型输出会映射为可调用工具或结构化动作请求。
- **Reasoning traces / 推理轨迹**: The example exposes or consumes explicit reasoning content alongside final answers. / 该示例会在最终答案之外展示或消费显式推理内容。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `get_current_weather`, `extract_reasoning_and_calls`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `hasattr`, `arguments.append`, `function_names.append`, `print`, `client.chat.completions.create`, `list`, `extract_reasoning_and_calls`, `OpenAI` reveal the main execution path / 这些调用体现了主要执行链路。
