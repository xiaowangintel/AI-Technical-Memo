# together_example_complete.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/quick_start/together_example_complete.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This quick-start example shows how to connect SGLang to the Together backend and execute completion-style requests. / 该快速入门示例展示了如何将 SGLang 连接到 Together 后端，并执行补全式请求。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module overview and usage
````python
"""
Usage:
export TOGETHER_API_KEY=sk-******
python3 together_example_complete.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 7-9: Import dependencies and runtime symbols
````python
import os

import sglang as sgl
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 12-23: Define few_shot_qa prompt program
````python
@sgl.function
def few_shot_qa(s, question):
    s += """The following are questions with answers.
Q: What is the capital of France?
A: Paris
Q: What is the capital of Germany?
A: Berlin
Q: What is the capital of Italy?
A: Rome
"""
    s += "Q: " + question + "\n"
    s += "A:" + sgl.gen("answer", stop="\n", temperature=0)
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 26-32: Run one request
````python
def single():
    state = few_shot_qa.run(question="What is the capital of the United States?")
    answer = state["answer"].strip().lower()

    assert "washington" in answer, f"answer: {state['answer']}"

    print(state.text())
````
**EN:** This function runs the example once with concrete inputs and prints the returned result in a human-readable form.
**CN:** 该函数会使用具体输入执行一次示例，并以便于阅读的形式打印返回结果。

### Lines 35-42: Stream incremental output
````python
def stream():
    state = few_shot_qa.run(
        question="What is the capital of the United States?", stream=True
    )

    for out in state.text_iter("answer"):
        print(out, end="", flush=True)
    print()
````
**EN:** This function enables streaming mode and consumes the result incrementally so the caller can observe partial output as soon as it is produced.
**CN:** 该函数会启用流式模式并逐步消费结果，因此调用方可以在内容产生后立即看到部分输出。

### Lines 45-54: Execute batched requests
````python
def batch():
    states = few_shot_qa.run_batch(
        [
            {"question": "What is the capital of the United States?"},
            {"question": "What is the capital of China?"},
        ]
    )

    for s in states:
        print(s["answer"])
````
**EN:** This function sends multiple requests together and iterates over the returned batch results.
**CN:** 该函数会把多个请求一起发送，并遍历返回的批量结果。

### Lines 57-76: Program entry point
````python
if __name__ == "__main__":
    backend = sgl.OpenAI(
        model_name="mistralai/Mixtral-8x7B-Instruct-v0.1",
        is_chat_model=False,
        base_url="https://api.together.xyz/v1",
        api_key=os.environ.get("TOGETHER_API_KEY"),
    )
    sgl.set_default_backend(backend)

    # Run a single request
    print("\n========== single ==========\n")
    single()

    # Stream output
    print("\n========== stream ==========\n")
    stream()

    # Run a batch of requests
    print("\n========== batch ==========\n")
    batch()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。

## Dependencies / 依赖关系
- **Standard library / 标准库**: os
- **Project-specific / 项目相关**: sglang
