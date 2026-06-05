# openai_chat_completion_client_with_tools_xlam.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/tool_calling/openai_chat_completion_client_with_tools_xlam.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Set up this example by starting a vLLM OpenAI-compatible server with tool call options enabled for xLAM-2 models: vllm serve --model Salesforce/Llama-xLAM-2-8b-fc-r --enable-auto-tool-choice --tool-call-parser xlam OR / 演示基于 vLLM 或 OpenAI 兼容 API 的工具调用交互。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Set up this example by starting a vLLM OpenAI-compatible server with tool call
options enabled for xLAM-2 models:

vllm serve --model Salesforce/Llama-xLAM-2-8b-fc-r --enable-auto-tool-choice --tool-call-parser xlam

OR

vllm serve --model Salesforce/xLAM-2-3b-fc-r --enable-auto-tool-choice --tool-call-parser xlam
"""
```
**EN:** Set up this example by starting a vLLM OpenAI-compatible server with tool call options enabled for xLAM-2 models: vllm serve --model Salesforce/Llama-xLAM-2-8b-fc-r --enable-auto-tool-choice --tool-call-parser xlam OR...
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import json
import time

from openai import OpenAI
```
**EN:** This block loads helper libraries such as json, time, and openai.
**CN:** 这一部分加载 json、time，以及 openai 等辅助库。

### Top-level setup
```python
openai_api_key = "empty"
openai_api_base = "http://localhost:8000/v1"
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as openai_api_key and openai_api_base.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 openai_api_key 和 openai_api_base 等变量。

### Function: get_weather
```python
def get_weather(location: str, unit: str):
    return f"Weather in {location} is 22 degrees {unit}."
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as location and unit. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 location 和 unit 等参数。其返回值会继续传给示例管线的下一阶段。

### Function: calculate_expression
```python
def calculate_expression(expression: str):
    try:
        result = eval(expression)
        return f"The result of {expression} is {result}"
    except Exception as e:
        return f"Could not calculate {expression}: {e}"
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as expression. Key operations include eval. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 expression 等参数。关键操作包括 eval。其返回值会继续传给示例管线的下一阶段。

### Function: translate_text
```python
def translate_text(text: str, target_language: str):
    return f"Translation of '{text}' to {target_language}: [translated content]"
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as text and target_language. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 text 和 target_language 等参数。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather in a given location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City and state, e.g., 'San Francisco, CA'",
    # ... key logic omitted for brevity ...
]

# Map of function names to implementations
tool_functions = {
    "get_weather": get_weather,
    "calculate_expression": calculate_expression,
    "translate_text": translate_text,
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as tools and tool_functions.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 tools 和 tool_functions 等变量。

### Function: process_response
```python
def process_response(response, tool_functions, original_query):
    """Process a non-streaming response with possible tool calls"""

    print("\n--- Response Output ---")

    # Check if the response has content
    if response.choices[0].message.content:
        print(f"Content: {response.choices[0].message.content}")

    # Check if the response has tool calls
    if response.choices[0].message.tool_calls:
        print("--------------------------------")
        print(f"Tool calls: {response.choices[0].message.tool_calls}")
        print("--------------------------------")

        # Collect all tool calls and results before making follow-up request
        tool_results = []
        assistant_message = {"role": "assistant"}

        if response.choices[0].message.content:
    # ... key logic omitted for brevity ...
        follow_up_messages.extend(tool_results)

        # Get completion with all tool results in a single follow-up
        follow_up_response = client.chat.completions.create(
            model=client.models.list().data[0].id,
            messages=follow_up_messages,
            stream=False,
        )

        print("\n--- Follow-up Response ---")
        print(follow_up_response.choices[0].message.content)
        print("--- End Follow-up ---\n")

    print("--- End Response ---\n")
```
**EN:** Process a non-streaming response with possible tool calls. It works with parameters such as response, tool_functions, and original_query. Key operations include print, follow_up_messages.extend, client.chat.completions.create, json.loads, and assistant_tool_calls.append.
**CN:** 该函数封装示例中的可复用步骤。它会处理 response、tool_functions，以及 original_query 等参数。关键操作包括 print、follow_up_messages.extend、client.chat.completions.create、json.loads，以及 assistant_tool_calls.append。

### Function: run_test_case
```python
def run_test_case(query, test_name):
    """Run a single test case with the given query"""
    print(f"\n{'=' * 50}\nTEST CASE: {test_name}\n{'=' * 50}")
    print(f"Query: '{query}'")

    start_time = time.time()

    # Create non-streaming chat completion request
    response = client.chat.completions.create(
        model=client.models.list().data[0].id,
        messages=[{"role": "user", "content": query}],
        tools=tools,
        tool_choice="auto",
        stream=False,
    )

    # Process the non-streaming response, passing the original query
    process_response(response, tool_functions, query)

    end_time = time.time()
    print(f"Test completed in {end_time - start_time:.2f} seconds")
```
**EN:** Run a single test case with the given query. It works with parameters such as query and test_name. Key operations include print, time.time, client.chat.completions.create, process_response, and client.models.list.
**CN:** 该函数编排端到端工作流。它会处理 query 和 test_name 等参数。关键操作包括 print、time.time、client.chat.completions.create、process_response，以及 client.models.list。

### Function: main
```python
def main():
    # Initialize OpenAI client
    global client
    client = OpenAI(
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    # Run test cases
    test_cases = [
        ("I want to know the weather in San Francisco", "Weather Information"),
        ("Calculate 25 * 17 + 31", "Math Calculation"),
        ("Translate 'Hello world' to Spanish", "Text Translation"),
        ("What is the weather in Tokyo and New York in celsius", "Multiple Tool Usage"),
    ]

    # Execute all test cases
    for query, test_name in test_cases:
        run_test_case(query, test_name)
        time.sleep(1)  # Small delay between tests

    print("\nAll tests completed.")
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include OpenAI, print, run_test_case, and time.sleep.
**CN:** 该函数编排端到端工作流。关键操作包括 OpenAI、print、run_test_case，以及 time.sleep。

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

## Dependencies / 依赖关系
- **External libraries / 外部库**: `json`, `time`, `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `get_weather`, `calculate_expression`, `translate_text`, `process_response`, `run_test_case`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `eval`, `print`, `follow_up_messages.extend`, `client.chat.completions.create`, `json.loads`, `assistant_tool_calls.append`, `tool_results.append`, `client.models.list` reveal the main execution path / 这些调用体现了主要执行链路。
