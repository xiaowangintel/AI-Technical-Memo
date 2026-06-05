# hidden_states_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/hidden_states/hidden_states_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example exposes hidden-state extraction through either the engine API or the server interface. / 该示例展示了如何通过引擎 API 或服务接口提取隐藏状态。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Module overview and usage
````python
"""
Usage:

python hidden_states_server.py

Note that each time you change the `return_hidden_states` parameter,
the cuda graph will be recaptured, which might lead to a performance hit.
So avoid getting hidden states and completions alternately.
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 11-15: Import dependencies and runtime symbols
````python
import requests
import torch

from sglang.test.test_utils import is_in_ci
from sglang.utils import terminate_process, wait_for_server
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 17-20: Set top-level configuration
````python
if is_in_ci():
    from docs.backend.patch import launch_server_cmd
else:
    from sglang.utils import launch_server_cmd
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 23-77: Parse arguments and run the workflow
````python
def main():
    # Launch the server
    server_process, port = launch_server_cmd(
        "python -m sglang.launch_server --model-path Alibaba-NLP/gte-Qwen2-1.5B-instruct --enable-return-hidden-states --host 0.0.0.0"
    )
    wait_for_server(f"http://localhost:{port}", process=server_process)

    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]

    sampling_params = {
        "temperature": 0.8,
        "top_p": 0.95,
        "max_new_tokens": 10,
    }

    json_data = {
        "text": prompts,
        "sampling_params": sampling_params,
        "return_hidden_states": True,
    }

    response = requests.post(
        f"http://localhost:{port}/generate",
        json=json_data,
    )

    terminate_process(server_process)

    outputs = response.json()
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

### Lines 80-81: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。

## Dependencies / 依赖关系
- **Third-party / 第三方**: docs.backend.patch.launch_server_cmd, requests, torch
- **Project-specific / 项目相关**: sglang.test.test_utils.is_in_ci, sglang.utils.launch_server_cmd, sglang.utils.terminate_process, sglang.utils.wait_for_server
