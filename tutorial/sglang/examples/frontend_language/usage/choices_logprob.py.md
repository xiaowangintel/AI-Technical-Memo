# choices_logprob.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/choices_logprob.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates choices logprob within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 choices logprob。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module overview and usage
````python
"""
Usage:
python -m sglang.launch_server --model-path meta-llama/Llama-2-7b-chat-hf --port 30000
python choices_logprob.py
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

### Lines 10-13: Define tool_use prompt program
````python
@sgl.function
def tool_use(s, question):
    s += "To answer this question: " + question + ", "
    s += "I need to use a " + sgl.gen("tool", choices=["calculator", "search engine"])
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 16-39: Parse arguments and run the workflow
````python
def main():
    # Run one case
    question = "What is 5 + 5?"
    state = tool_use.run(question)
    print("questions:", question)
    print("choice:", state["tool"])
    meta_info = state.get_meta_info("tool")
    print("logprobs of choice 1", meta_info["input_token_logprobs"][0])
    print("logprobs of choice 2", meta_info["input_token_logprobs"][1])
    print("-" * 50)

    # Run a batch
    questions = [
        "What is 5 + 6?",
        "Who is Michael Jordan?",
    ]
    states = tool_use.run_batch([{"question": q} for q in questions])
    for question, state in zip(questions, states):
        print("questions:", question)
        print("choice:", state["tool"])
        meta_info = state.get_meta_info("tool")
        print("logprobs of choice 1", meta_info["input_token_logprobs"][0])
        print("logprobs of choice 2", meta_info["input_token_logprobs"][1])
        print("-" * 50)
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 42-44: Program entry point
````python
if __name__ == "__main__":
    sgl.set_default_backend(sgl.RuntimeEndpoint("http://localhost:30000"))
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。

## Dependencies / 依赖关系
- **Project-specific / 项目相关**: sglang
