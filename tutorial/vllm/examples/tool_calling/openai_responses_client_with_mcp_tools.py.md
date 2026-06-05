# openai_responses_client_with_mcp_tools.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/tool_calling/openai_responses_client_with_mcp_tools.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example demonstrating MCP (Model Context Protocol) tools with the Responses API / 演示基于 vLLM 或 OpenAI 兼容 API 的工具调用交互。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example demonstrating MCP (Model Context Protocol) tools with the Responses API.

This example shows how to use MCP tools with different allowed_tools configurations:
1. No filter (allows all tools from the MCP server)
2. Wildcard "*" (explicitly allows all tools)
3. Specific tool names (filters to only those tools)

Set up this example by starting a vLLM OpenAI-compatible server with MCP tools enabled.
For example:
vllm serve openai/gpt-oss-20b --enforce-eager --tool-server demo

Environment variables:
- VLLM_ENABLE_RESPONSES_API_STORE=1
- VLLM_GPT_OSS_SYSTEM_TOOL_MCP_LABELS=code_interpreter,container
- VLLM_GPT_OSS_HARMONY_SYSTEM_INSTRUCTIONS=1
"""
```
**EN:** Example demonstrating MCP (Model Context Protocol) tools with the Responses API.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from openai import OpenAI
```
**EN:** This block loads helper libraries such as openai.
**CN:** 这一部分加载 openai 等辅助库。

### Function: example_no_filter
```python
def example_no_filter():
    """Example with no allowed_tools filter - allows all tools."""
    print("=" * 60)
    print("Example 1: No allowed_tools filter (allows all tools)")
    print("=" * 60)

    base_url = "http://0.0.0.0:8000/v1"
    client = OpenAI(base_url=base_url, api_key="empty")
    model = client.models.list().data[0].id

    response = client.responses.create(
        model=model,
        input="Execute this code: print('Hello from Python!')",
        instructions="Use the Python tool to execute code.",
        tools=[
            {
                "type": "mcp",
                "server_label": "code_interpreter",
                "server_url": "http://localhost:8888",
                # No allowed_tools specified - all tools are available
            }
        ],
    )

    print(f"Status: {response.status}")
    print(f"Output: {response.output_text}")
    print()
```
**EN:** Example with no allowed_tools filter - allows all tools.. Key operations include print, OpenAI, client.responses.create, and client.models.list.
**CN:** 该函数编排端到端工作流。关键操作包括 print、OpenAI、client.responses.create，以及 client.models.list。

### Function: example_wildcard
```python
def example_wildcard():
    """Example with allowed_tools=['*'] - explicitly allows all tools."""
    print("=" * 60)
    print("Example 2: allowed_tools=['*'] (select all tools)")
    print("=" * 60)

    base_url = "http://0.0.0.0:8000/v1"
    client = OpenAI(base_url=base_url, api_key="empty")
    model = client.models.list().data[0].id

    response = client.responses.create(
        model=model,
        input="Execute this code: print('Hello from Python with wildcard!')",
        instructions="Use the Python tool to execute code.",
        tools=[
            {
                "type": "mcp",
                "server_label": "code_interpreter",
                "server_url": "http://localhost:8888",
                # Using "*" to explicitly allow all tools from this MCP server
                # This is equivalent to not specifying allowed_tools
                "allowed_tools": ["*"],
            }
        ],
    )

    print(f"Status: {response.status}")
    print(f"Output: {response.output_text}")
    print()
```
**EN:** Example with allowed_tools=['*'] - explicitly allows all tools.. Key operations include print, OpenAI, client.responses.create, and client.models.list.
**CN:** 该函数编排端到端工作流。关键操作包括 print、OpenAI、client.responses.create，以及 client.models.list。

### Function: example_specific_tools
```python
def example_specific_tools():
    """Example with specific allowed_tools list - filters available tools.

    Note: This example uses 'web_search_preview' (browser) which has multiple
    sub-tools: 'search', 'open', 'find'. The code_interpreter (python) doesn't
    have sub-tools, so filtering doesn't apply there.
    """
    print("=" * 60)
    print("Example 3: allowed_tools=['search'] (filter browser to specific tools)")
    print("=" * 60)

    base_url = "http://0.0.0.0:8000/v1"
    client = OpenAI(base_url=base_url, api_key="empty")
    model = client.models.list().data[0].id

    response = client.responses.create(
        model=model,
        input="Search for 'Python programming tutorials'",
        instructions="Use the browser tool to search.",
        tools=[
            {
                "type": "mcp",
                "server_label": "web_search_preview",
                "server_url": "http://localhost:8888",
                # Browser has tools: 'search', 'open', 'find'
                # Only allow 'search' - blocks 'open' and 'find'
                "allowed_tools": ["search"],
            }
        ],
    )

    print(f"Status: {response.status}")
    print(f"Output: {response.output_text}")
    print()
```
**EN:** Example with specific allowed_tools list - filters available tools.. Key operations include print, OpenAI, client.responses.create, and client.models.list.
**CN:** 该函数编排端到端工作流。关键操作包括 print、OpenAI、client.responses.create，以及 client.models.list。

### Function: example_object_format
```python
def example_object_format():
    """Example using object format for allowed_tools with browser tools."""
    print("=" * 60)
    print("Example 4: allowed_tools with object format")
    print("=" * 60)

    base_url = "http://0.0.0.0:8000/v1"
    client = OpenAI(base_url=base_url, api_key="empty")
    model = client.models.list().data[0].id

    response = client.responses.create(
        model=model,
        input="Search for 'machine learning' and open the first result",
        instructions="Use the browser tool.",
        tools=[
            {
                "type": "mcp",
                "server_label": "web_search_preview",
                "server_url": "http://localhost:8888",
                # Object format with tool_names field
                # Can also include read_only and other fields
                # Browser has tools: 'search', 'open', 'find'
                "allowed_tools": {
                    "tool_names": [
                        "search",
                        "open",
                    ],  # Allow search and open, block find
                    "read_only": False,
                },
            }
        ],
    )

    print(f"Status: {response.status}")
    print(f"Output: {response.output_text}")
    print()
```
**EN:** Example using object format for allowed_tools with browser tools.. Key operations include print, OpenAI, client.responses.create, and client.models.list.
**CN:** 该函数编排端到端工作流。关键操作包括 print、OpenAI、client.responses.create，以及 client.models.list。

### Function: main
```python
def main():
    """Run all examples."""
    print("\n" + "=" * 60)
    print("MCP Tools with allowed_tools Examples")
    print("=" * 60 + "\n")

    # Run all examples
    example_no_filter()
    example_wildcard()
    example_specific_tools()
    example_object_format()

    print("=" * 60)
    print("Summary:")
    print("  - No filter or '*' → All tools available from server")
    print("  - Specific list → Only those sub-tools available")
    print("  - Object format → More control with tool_names field")
    print("")
    print("Note: allowed_tools filters SUB-TOOLS within an MCP server:")
    print("  - code_interpreter (python): No sub-tools to filter")
    print("  - web_search_preview (browser): Has 'search', 'open', 'find'")
    print("=" * 60)
```
**EN:** Run all examples.. Key operations include print, example_no_filter, example_wildcard, example_specific_tools, and example_object_format.
**CN:** 该函数编排端到端工作流。关键操作包括 print、example_no_filter、example_wildcard、example_specific_tools，以及 example_object_format。

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
- **External libraries / 外部库**: `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `example_no_filter`, `example_wildcard`, `example_specific_tools`, `example_object_format`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `OpenAI`, `client.responses.create`, `client.models.list`, `example_no_filter`, `example_wildcard`, `example_specific_tools`, `example_object_format` reveal the main execution path / 这些调用体现了主要执行链路。
