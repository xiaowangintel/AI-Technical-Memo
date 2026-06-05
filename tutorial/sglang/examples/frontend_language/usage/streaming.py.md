# streaming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/streaming.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates streaming within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 streaming。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Module overview and usage
````python
"""
Usage:
python3 streaming.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 6-8: Import dependencies and runtime symbols
````python
import asyncio

import sglang as sgl
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 11-17: Define multi_turn_question prompt program
````python
@sgl.function
def multi_turn_question(s, question_1, question_2):
    s += sgl.system("You are a helpful assistant.")
    s += sgl.user(question_1)
    s += sgl.assistant(sgl.gen("answer_1", max_tokens=256))
    s += sgl.user(question_2)
    s += sgl.assistant(sgl.gen("answer_2", max_tokens=256))
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 20-20: Configure the default backend
````python
sgl.set_default_backend(sgl.OpenAI("gpt-3.5-turbo"))
````
**EN:** The example selects a concrete backend once so later prompt executions automatically use the same model endpoint.
**CN:** 该示例先统一选择具体后端，这样后续提示执行都会自动使用同一个模型端点。

### Lines 23-32: Stream incremental output
````python
def stream_a_variable():
    state = multi_turn_question.run(
        question_1="What is the capital of the United States?",
        question_2="List two local attractions.",
        stream=True,
    )

    for out in state.text_iter(var_name="answer_2"):
        print(out, end="", flush=True)
    print("\n")
````
**EN:** This function enables streaming mode and consumes the result incrementally so the caller can observe partial output as soon as it is produced.
**CN:** 该函数会启用流式模式并逐步消费结果，因此调用方可以在内容产生后立即看到部分输出。

### Lines 35-44: Handle stream
````python
async def async_stream():
    state = multi_turn_question.run(
        question_1="What is the capital of the United States?",
        question_2="List two local attractions.",
        stream=True,
    )

    async for out in state.text_async_iter(var_name="answer_2"):
        print(out, end="", flush=True)
    print("\n")
````
**EN:** This function encapsulates the “Handle stream” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle stream”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 47-49: Program entry point
````python
if __name__ == "__main__":
    stream_a_variable()
    asyncio.run(async_stream())
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。

## Dependencies / 依赖关系
- **Standard library / 标准库**: asyncio
- **Project-specific / 项目相关**: sglang
