# token_in_token_out_vlm_engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/token_in_token_out/token_in_token_out_vlm_engine.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example works directly with token IDs and low-level request/response objects instead of plain text prompts. / 该示例直接处理 token ID 以及底层请求/响应对象，而不是普通文本提示。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Import dependencies and runtime symbols
````python
import argparse
import dataclasses
from typing import Tuple

from transformers import AutoProcessor

from sglang import Engine
from sglang.lang.chat_template import get_chat_template_by_model_path
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.server_args import ServerArgs
from sglang.test.test_utils import DEFAULT_IMAGE_URL
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 14-34: Get input ids
````python
def get_input_ids(
    server_args: ServerArgs, model_config: ModelConfig
) -> Tuple[list[int], list]:
    chat_template = get_chat_template_by_model_path(model_config.model_path)
    text = f"{chat_template.image_token}What is in this picture?"
    image_data = [DEFAULT_IMAGE_URL]

    processor = AutoProcessor.from_pretrained(
        model_config.model_path, trust_remote_code=server_args.trust_remote_code
    )

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

### Lines 37-62: Drive the engine runtime
````python
def token_in_out_example(
    server_args: ServerArgs,
):
    input_ids, image_data = get_input_ids(
        server_args,
        ModelConfig(
            server_args.model_path,
            trust_remote_code=server_args.trust_remote_code,
            model_override_args=server_args.json_model_override_args,
        ),
    )
    backend = Engine(**dataclasses.asdict(server_args))

    output = backend.generate(
        input_ids=input_ids,
        image_data=image_data,
        sampling_params={
            "temperature": 0.8,
            "max_new_tokens": 32,
        },
    )

    print("===============================")
    print(f"Output token ids: ", output["output_ids"])

    backend.shutdown()
````
**EN:** This function constructs or uses an SGLang engine instance and wraps a concrete runtime interaction around it.
**CN:** 该函数会构建或使用一个 SGLang 引擎实例，并围绕它封装具体的运行时交互。

### Lines 65-74: Program entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    ServerArgs.add_cli_args(parser)
    args = [
        "--model-path=Qwen/Qwen2-VL-2B",
    ]
    args = parser.parse_args(args=args)
    server_args = ServerArgs.from_cli_args(args)
    server_args.skip_tokenizer_init = True
    token_in_out_example(server_args)
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Token-in/token-out / Token 输入输出**: The interface operates directly on token IDs and low-level tensors. / 该接口直接处理 token ID 和底层张量。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, dataclasses, typing.Tuple
- **Third-party / 第三方**: transformers.AutoProcessor
- **Project-specific / 项目相关**: sglang.Engine, sglang.lang.chat_template.get_chat_template_by_model_path, sglang.srt.configs.model_config.ModelConfig, sglang.srt.server_args.ServerArgs, sglang.test.test_utils.DEFAULT_IMAGE_URL
