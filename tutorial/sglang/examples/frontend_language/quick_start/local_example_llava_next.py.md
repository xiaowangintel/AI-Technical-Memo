# local_example_llava_next.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/quick_start/local_example_llava_next.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This quick-start example shows how to connect SGLang to the Local backend and execute multimodal requests. / 该快速入门示例展示了如何将 SGLang 连接到 Local 后端，并执行多模态请求。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Module overview and usage
````python
"""
Usage: python3 local_example_llava_next.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 5-6: Import dependencies and runtime symbols
````python
import sglang as sgl
from sglang.lang.chat_template import get_chat_template
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 9-12: Define image_qa prompt program
````python
@sgl.function
def image_qa(s, image_path, question):
    s += sgl.user(sgl.image(image_path) + question)
    s += sgl.assistant(sgl.gen("answer"))
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 15-19: Run one request
````python
def single():
    state = image_qa.run(
        image_path="images/cat.jpeg", question="What is this?", max_new_tokens=128
    )
    print(state["answer"], "\n")
````
**EN:** This function runs the example once with concrete inputs and prints the returned result in a human-readable form.
**CN:** 该函数会使用具体输入执行一次示例，并以便于阅读的形式打印返回结果。

### Lines 22-32: Stream incremental output
````python
def stream():
    state = image_qa.run(
        image_path="images/cat.jpeg",
        question="What is this?",
        max_new_tokens=64,
        stream=True,
    )

    for out in state.text_iter("answer"):
        print(out, end="", flush=True)
    print()
````
**EN:** This function enables streaming mode and consumes the result incrementally so the caller can observe partial output as soon as it is produced.
**CN:** 该函数会启用流式模式并逐步消费结果，因此调用方可以在内容产生后立即看到部分输出。

### Lines 35-44: Execute batched requests
````python
def batch():
    states = image_qa.run_batch(
        [
            {"image_path": "images/cat.jpeg", "question": "What is this?"},
            {"image_path": "images/dog.jpeg", "question": "What is this?"},
        ],
        max_new_tokens=128,
    )
    for s in states:
        print(s["answer"], "\n")
````
**EN:** This function sends multiple requests together and iterates over the returned batch results.
**CN:** 该函数会把多个请求一起发送，并遍历返回的批量结果。

### Lines 47-78: Program entry point
````python
if __name__ == "__main__":
    import multiprocessing as mp

    mp.set_start_method("spawn", force=True)

    runtime = sgl.Runtime(model_path="lmms-lab/llama3-llava-next-8b")
    runtime.endpoint.chat_template = get_chat_template("llama-3-instruct-llava")

    # Or you can use the 72B model
    # runtime = sgl.Runtime(model_path="lmms-lab/llava-next-72b", tp_size=8)
    # runtime.endpoint.chat_template = get_chat_template("chatml-llava")

    sgl.set_default_backend(runtime)
    print(f"chat template: {runtime.endpoint.chat_template.name}")

    # Or you can use API models
    # sgl.set_default_backend(sgl.OpenAI("gpt-4-vision-preview"))
    # sgl.set_default_backend(sgl.VertexAI("gemini-pro-vision"))

    # Run a single request
    print("\n========== single ==========\n")
    single()

    # Stream output
    print("\n========== stream ==========\n")
    stream()

    # Run a batch of requests
    print("\n========== batch ==========\n")
    batch()

    runtime.shutdown()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。

## Dependencies / 依赖关系
- **Standard library / 标准库**: multiprocessing
- **Project-specific / 项目相关**: sglang, sglang.lang.chat_template.get_chat_template
