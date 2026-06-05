# token_in_token_out_vlm_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/token_in_token_out/token_in_token_out_vlm_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example works directly with token IDs and low-level request/response objects instead of plain text prompts. / 该示例直接处理 token ID 以及底层请求/响应对象，而不是普通文本提示。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module overview and usage
````python
"""
Usage:

python token_in_token_out_vlm_server.py

"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 8-15: Import dependencies and runtime symbols
````python
from typing import Tuple

import requests
from transformers import AutoProcessor

from sglang.lang.chat_template import get_chat_template_by_model_path
from sglang.test.test_utils import DEFAULT_IMAGE_URL, is_in_ci
from sglang.utils import terminate_process, wait_for_server
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 17-23: Set top-level configuration
````python
if is_in_ci():
    from docs.backend.patch import launch_server_cmd
else:
    from sglang.utils import launch_server_cmd


MODEL_PATH = "Qwen/Qwen2-VL-2B"
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 26-42: Get input ids
````python
def get_input_ids() -> Tuple[list[int], list]:
    chat_template = get_chat_template_by_model_path(MODEL_PATH)
    text = f"{chat_template.image_token}What is in this picture?"
    image_data = [DEFAULT_IMAGE_URL]

    processor = AutoProcessor.from_pretrained(MODEL_PATH)

    input_ids = (
        processor.tokenizer(
            text=[text],
            return_tensors="pt",
        )
        .input_ids[0]
        .tolist()
    )

    return input_ids, image_data
````
**EN:** This function encapsulates the “Get input ids” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Get input ids”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 45-74: Parse arguments and run the workflow
````python
def main():
    # Launch the server
    server_process, port = launch_server_cmd(
        f"python -m sglang.launch_server --model-path {MODEL_PATH} --skip-tokenizer-init --host 0.0.0.0"
    )
    wait_for_server(f"http://localhost:{port}", process=server_process)

    input_ids, image_data = get_input_ids()

    sampling_params = {
        "temperature": 0.8,
        "max_new_tokens": 32,
    }

    json_data = {
        "input_ids": input_ids,
        "image_data": image_data,
        "sampling_params": sampling_params,
    }

    response = requests.post(
        f"http://localhost:{port}/generate",
        json=json_data,
    )

    output = response.json()
    print("===============================")
    print(f"Output token ids: ", output["output_ids"])

    terminate_process(server_process)
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 77-78: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。
- **Token-in/token-out / Token 输入输出**: The interface operates directly on token IDs and low-level tensors. / 该接口直接处理 token ID 和底层张量。

## Dependencies / 依赖关系
- **Standard library / 标准库**: typing.Tuple
- **Third-party / 第三方**: docs.backend.patch.launch_server_cmd, requests, transformers.AutoProcessor
- **Project-specific / 项目相关**: sglang.lang.chat_template.get_chat_template_by_model_path, sglang.test.test_utils.DEFAULT_IMAGE_URL, sglang.test.test_utils.is_in_ci, sglang.utils.launch_server_cmd, sglang.utils.terminate_process, sglang.utils.wait_for_server
