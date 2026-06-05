# offline_batch_inference_eagle.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/engine/offline_batch_inference_eagle.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This runtime example demonstrates offline batch inference eagle with the SGLang engine APIs. / 该运行时示例展示了如何使用 SGLang 引擎 API 完成 offline batch inference eagle。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Import dependencies and runtime symbols
````python
import sglang as sgl
````
**EN:** The file only needs the SGLang frontend/runtime API, which it imports under the `sgl` alias.
**CN:** 该文件只依赖 SGLang 前端/运行时 API，并以 `sgl` 别名导入。

### Lines 4-32: Parse arguments and run the workflow
````python
def main():
    # Sample prompts.
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]

    # Create a sampling params object.
    sampling_params = {"temperature": 0, "max_new_tokens": 30}

    # Create an LLM.
    llm = sgl.Engine(
        model_path="meta-llama/Llama-2-7b-chat-hf",
        speculative_algorithm="EAGLE",
        speculative_draft_model_path="lmsys/sglang-EAGLE-llama2-chat-7B",
        speculative_num_steps=3,
        speculative_eagle_topk=4,
        speculative_num_draft_tokens=16,
        cuda_graph_max_bs=8,
    )

    outputs = llm.generate(prompts, sampling_params)

    # Print the outputs.
    for prompt, output in zip(prompts, outputs):
        print("===============================")
        print(f"Prompt: {prompt}\nGenerated text: {output['text']}")
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 35-38: Program entry point
````python
# The __main__ condition is necessary here because we use "spawn" to create subprocesses
# Spawn starts a fresh program every time, if there is no __main__, it will run into infinite loop to keep spawning processes from sgl.Engine
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。

## Dependencies / 依赖关系
- **Project-specific / 项目相关**: sglang
