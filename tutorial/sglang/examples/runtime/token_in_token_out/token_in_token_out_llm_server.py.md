# token_in_token_out_llm_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/token_in_token_out/token_in_token_out_llm_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example works directly with token IDs and low-level request/response objects instead of plain text prompts. / 该示例直接处理 token ID 以及底层请求/响应对象，而不是普通文本提示。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module overview and usage
````python
"""
Usage:

python token_in_token_out_llm_server.py

"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 8-12: Import dependencies and runtime symbols
````python
import requests

from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.test_utils import is_in_ci
from sglang.utils import terminate_process, wait_for_server
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 14-20: Set top-level configuration
````python
if is_in_ci():
    from docs.backend.patch import launch_server_cmd
else:
    from sglang.utils import launch_server_cmd


MODEL_PATH = "meta-llama/Llama-3.1-8B-Instruct"
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 23-64: Parse arguments and run the workflow
````python
def main():
    # Launch the server
    server_process, port = launch_server_cmd(
        f"python -m sglang.launch_server --model-path {MODEL_PATH} --skip-tokenizer-init --host 0.0.0.0"
    )
    wait_for_server(f"http://localhost:{port}", process=server_process)

    # Sample prompts.
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]

    # Create a sampling params object.
    sampling_params = {"temperature": 0.8, "top_p": 0.95}

    # Tokenize inputs
    tokenizer = get_tokenizer(MODEL_PATH)
    token_ids_list = [tokenizer.encode(prompt) for prompt in prompts]

    json_data = {
        "input_ids": token_ids_list,
        "sampling_params": sampling_params,
    }

    response = requests.post(
        f"http://localhost:{port}/generate",
        json=json_data,
    )

    outputs = response.json()
    for prompt, output in zip(prompts, outputs):
        print("===============================")
        decode_output = tokenizer.decode(output["output_ids"])
        print(
            f"Prompt: {prompt}\nGenerated token ids: {output['output_ids']}\nGenerated text: {decode_output}"
        )
        print()

    terminate_process(server_process)
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 67-68: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Token-in/token-out / Token 输入输出**: The interface operates directly on token IDs and low-level tensors. / 该接口直接处理 token ID 和底层张量。

## Dependencies / 依赖关系
- **Third-party / 第三方**: docs.backend.patch.launch_server_cmd, requests
- **Project-specific / 项目相关**: sglang.srt.utils.hf_transformers_utils.get_tokenizer, sglang.test.test_utils.is_in_ci, sglang.utils.launch_server_cmd, sglang.utils.terminate_process, sglang.utils.wait_for_server
