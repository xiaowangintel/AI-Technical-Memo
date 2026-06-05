# openai_chat_completion_client_with_tools.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/tool_calling/openai_chat_completion_client_with_tools.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Set up this example by starting a vLLM OpenAI-compatible server with tool call options enabled / 演示基于 vLLM 或 OpenAI 兼容 API 的工具调用交互。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Set up this example by starting a vLLM OpenAI-compatible server with tool call
options enabled. For example:

IMPORTANT: for mistral, you must use one of the provided mistral tool call
templates, or your own - the model default doesn't work for tool calls with vLLM
See the vLLM docs on OpenAI server & tool calling for more details.

vllm serve mistralai/Mistral-7B-Instruct-v0.3 \
            --chat-template examples/tool_chat_template_mistral.jinja \
            --enable-auto-tool-choice --tool-call-parser mistral

OR
vllm serve NousResearch/Hermes-2-Pro-Llama-3-8B \
            --chat-template examples/tool_chat_template_hermes.jinja \
            --enable-auto-tool-choice --tool-call-parser hermes
"""
```
**EN:** Set up this example by starting a vLLM OpenAI-compatible server with tool call options enabled.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import json
from typing import Any

from openai import OpenAI
```
**EN:** This block loads helper libraries such as json, typing, and openai.
**CN:** 这一部分加载 json、typing，以及 openai 等辅助库。

### Top-level setup
```python
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

properties = {
    "city": {
        "type": "string",
        "description": "The city to find the weather for, e.g. 'San Francisco'",
    },
    "state": {
        "type": "string",
        "description": "the two-letter abbreviation for the state that the city is"
        " in, e.g. 'CA' which would mean 'California'",
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
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as openai_api_key, openai_api_base, properties, tools, and messages.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 openai_api_key、openai_api_base、properties、tools，以及 messages 等变量。

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

### Function: handle_tool_calls_stream
```python
def handle_tool_calls_stream(
    client: OpenAI,
    messages: list[dict[str, str]],
    model: str,
    tools: list[dict[str, Any]],
) -> list[Any]:
    tool_calls_stream = client.chat.completions.create(
        messages=messages, model=model, tools=tools, stream=True
    )
    chunks = []
    print("chunks: ")
    for chunk in tool_calls_stream:
        chunks.append(chunk)
        if chunk.choices[0].delta.tool_calls:
            print(chunk.choices[0].delta.tool_calls[0])
        else:
            print(chunk.choices[0].delta)
    return chunks
```
**EN:** This function sends a request and handles the reply. It works with parameters such as client, messages, model, and tools. Key operations include print, client.chat.completions.create, and chunks.append. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。它会处理 client、messages、model，以及 tools 等参数。关键操作包括 print、client.chat.completions.create，以及 chunks.append。其返回值会继续传给示例管线的下一阶段。

### Function: handle_tool_calls_arguments
```python
def handle_tool_calls_arguments(chunks: list[Any]) -> list[str]:
    arguments = []
    tool_call_idx = -1
    print("arguments: ")
    for chunk in chunks:
        if chunk.choices[0].delta.tool_calls:
            tool_call = chunk.choices[0].delta.tool_calls[0]
            if tool_call.index != tool_call_idx:
                if tool_call_idx >= 0:
                    print(f"streamed tool call arguments: {arguments[tool_call_idx]}")
                tool_call_idx = chunk.choices[0].delta.tool_calls[0].index
                arguments.append("")
            if tool_call.id:
                print(f"streamed tool call id: {tool_call.id} ")

            if tool_call.function:
                if tool_call.function.name:
                    print(f"streamed tool call name: {tool_call.function.name}")

                if tool_call.function.arguments:
                    arguments[tool_call_idx] += tool_call.function.arguments

    return arguments
```
**EN:** This function builds and validates command-line arguments. It works with parameters such as chunks. Key operations include print and arguments.append. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。它会处理 chunks 等参数。关键操作包括 print 和 arguments.append。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    # Initialize OpenAI client
    client = OpenAI(
        # defaults to os.environ.get("OPENAI_API_KEY")
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    # Get available models and select one
    models = client.models.list()
    model = models.data[0].id

    chat_completion = client.chat.completions.create(
        messages=messages, model=model, tools=tools
    )

    print("-" * 70)
    print("Chat completion results:")
    print(chat_completion)
    print("-" * 70)
    # ... key logic omitted for brevity ...
            {
                "role": "tool",
                "content": result,
                "tool_call_id": call.id,
                "name": call.function.name,
            }
        )

    chat_completion_2 = client.chat.completions.create(
        messages=messages, model=model, tools=tools, stream=False
    )
    print("Chat completion2 results:")
    print(chat_completion_2)
    print("-" * 70)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print, client.chat.completions.create, messages.append, OpenAI, and client.models.list.
**CN:** 该函数编排端到端工作流。关键操作包括 print、client.chat.completions.create、messages.append、OpenAI，以及 client.models.list。

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
- **External libraries / 外部库**: `json`, `typing`, `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `get_current_weather`, `handle_tool_calls_stream`, `handle_tool_calls_arguments`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `client.chat.completions.create`, `chunks.append`, `arguments.append`, `messages.append`, `OpenAI`, `client.models.list`, `handle_tool_calls_stream` reveal the main execution path / 这些调用体现了主要执行链路。
