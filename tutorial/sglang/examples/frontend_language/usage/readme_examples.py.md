# readme_examples.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/readme_examples.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates readme examples within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 readme examples。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module overview and usage
````python
"""
Usage:
python -m sglang.launch_server --model-path meta-llama/Llama-2-7b-chat-hf --port 30000
python readme_examples.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 7-7: Import dependencies and runtime symbols
````python
import sglang as sgl
````
**EN:** The file only needs the SGLang frontend/runtime API, which it imports under the `sgl` alias.
**CN:** 该文件只依赖 SGLang 前端/运行时 API，并以 `sgl` 别名导入。

### Lines 10-22: Define tool_use prompt program
````python
@sgl.function
def tool_use(s, question):
    s += "To answer this question: " + question + ". "
    s += (
        "I need to use a "
        + sgl.gen("tool", choices=["calculator", "search engine"])
        + ". "
    )

    if s["tool"] == "calculator":
        s += "The math expression is" + sgl.gen("expression")
    elif s["tool"] == "search engine":
        s += "The key word to search is" + sgl.gen("word")
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 25-39: Define tip_suggestion prompt program
````python
@sgl.function
def tip_suggestion(s):
    s += (
        "Here are two tips for staying healthy: "
        "1. Balanced Diet. 2. Regular Exercise.\n\n"
    )

    forks = s.fork(2)
    for i, f in enumerate(forks):
        f += f"Now, expand tip {i+1} into a paragraph:\n"
        f += sgl.gen(f"detailed_tip", max_tokens=256, stop="\n\n")

    s += "Tip 1:" + forks[0]["detailed_tip"] + "\n"
    s += "Tip 2:" + forks[1]["detailed_tip"] + "\n"
    s += "In summary" + sgl.gen("summary")
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 42-49: Define regular_expression_gen prompt program
````python
@sgl.function
def regular_expression_gen(s):
    s += "Q: What is the IP address of the Google DNS servers?\n"
    s += "A: " + sgl.gen(
        "answer",
        temperature=0,
        regex=r"((25[0-5]|2[0-4]\d|[01]?\d\d?).){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)",
    )
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 52-55: Define text_qa prompt program
````python
@sgl.function
def text_qa(s, question):
    s += "Q: " + question + "\n"
    s += "A:" + sgl.gen("answer", stop="\n")
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 58-61: Handle tool use
````python
def driver_tool_use():
    state = tool_use.run(question="What is the capital of the United States?")
    print(state.text())
    print("\n")
````
**EN:** This function encapsulates the “Handle tool use” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle tool use”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 64-67: Handle tip suggestion
````python
def driver_tip_suggestion():
    state = tip_suggestion.run()
    print(state.text())
    print("\n")
````
**EN:** This function encapsulates the “Handle tip suggestion” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle tip suggestion”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 70-73: Handle regex
````python
def driver_regex():
    state = regular_expression_gen.run()
    print(state.text())
    print("\n")
````
**EN:** This function encapsulates the “Handle regex” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle regex”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 76-88: Execute batched requests
````python
def driver_batching():
    states = text_qa.run_batch(
        [
            {"question": "What is the capital of the United Kingdom?"},
            {"question": "What is the capital of France?"},
            {"question": "What is the capital of Japan?"},
        ],
        progress_bar=True,
    )

    for s in states:
        print(s.text())
    print("\n")
````
**EN:** This function sends multiple requests together and iterates over the returned batch results.
**CN:** 该函数会把多个请求一起发送，并遍历返回的批量结果。

### Lines 91-98: Stream incremental output
````python
def driver_stream():
    state = text_qa.run(
        question="What is the capital of France?", temperature=0.1, stream=True
    )

    for out in state.text_iter():
        print(out, end="", flush=True)
    print("\n")
````
**EN:** This function enables streaming mode and consumes the result incrementally so the caller can observe partial output as soon as it is produced.
**CN:** 该函数会启用流式模式并逐步消费结果，因此调用方可以在内容产生后立即看到部分输出。

### Lines 101-109: Program entry point
````python
if __name__ == "__main__":
    # sgl.set_default_backend(sgl.OpenAI("gpt-3.5-turbo-instruct"))
    sgl.set_default_backend(sgl.RuntimeEndpoint("http://localhost:30000"))

    driver_tool_use()
    driver_tip_suggestion()
    driver_regex()
    driver_batching()
    driver_stream()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。

## Dependencies / 依赖关系
- **Project-specific / 项目相关**: sglang
