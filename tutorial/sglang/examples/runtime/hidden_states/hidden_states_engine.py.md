# hidden_states_engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/hidden_states/hidden_states_engine.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example exposes hidden-state extraction through either the engine API or the server interface. / 该示例展示了如何通过引擎 API 或服务接口提取隐藏状态。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module overview and usage
````python
"""
Usage:
python hidden_states.py

Note that each time you change the `return_hidden_states` parameter,
the cuda graph will be recaptured, which might lead to a performance hit.
So avoid getting hidden states and completions alternately.
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 10-12: Import dependencies and runtime symbols
````python
import torch

import sglang as sgl
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 15-60: Parse arguments and run the workflow
````python
def main():
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]
    # Create an LLM.
    llm = sgl.Engine(
        model_path="Alibaba-NLP/gte-Qwen2-1.5B-instruct",
        enable_return_hidden_states=True,
    )

    sampling_params = {
        "temperature": 0.8,
        "top_p": 0.95,
        "max_new_tokens": 10,
    }

    outputs = llm.generate(
        prompts, sampling_params=sampling_params, return_hidden_states=True
    )

    llm.shutdown()

    for prompt, output in zip(prompts, outputs):
        for i in range(len(output["meta_info"]["hidden_states"])):
            output["meta_info"]["hidden_states"][i] = torch.tensor(
                output["meta_info"]["hidden_states"][i], dtype=torch.bfloat16
            )
        print("===============================")
        print(
            f"Prompt: {prompt}\n"
            f"Generated text: {output['text']}\n"
            f"Prompt_Tokens: {output['meta_info']['prompt_tokens']}\t"
            f"Completion_tokens: {output['meta_info']['completion_tokens']}"
        )
        print("Hidden states: ")
        hidden_states = torch.cat(
            [
                i.unsqueeze(0) if len(i.shape) == 1 else i
                for i in output["meta_info"]["hidden_states"]
            ]
        )
        print(hidden_states)
        print()
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 63-66: Program entry point
````python
# The __main__ condition is necessary here because we use "spawn" to create subprocesses
# Spawn starts a fresh program every time, if there is no __main__, it will run into infinite loop to keep spawning processes from sgl.Engine
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。

## Dependencies / 依赖关系
- **Third-party / 第三方**: torch
- **Project-specific / 项目相关**: sglang
