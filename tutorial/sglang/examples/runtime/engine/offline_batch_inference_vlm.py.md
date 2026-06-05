# offline_batch_inference_vlm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/engine/offline_batch_inference_vlm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This runtime example demonstrates offline batch inference vlm with the SGLang engine APIs. / 该运行时示例展示了如何使用 SGLang 引擎 API 完成 offline batch inference vlm。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Module overview and usage
````python
"""
Usage:
python offline_batch_inference_vlm.py --model-path Qwen/Qwen2-VL-7B-Instruct
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 6-11: Import dependencies and runtime symbols
````python
import argparse
import dataclasses

import sglang as sgl
from sglang.srt.parser.conversation import chat_templates
from sglang.srt.server_args import ServerArgs
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 14-41: Parse arguments and run the workflow
````python
def main(
    server_args: ServerArgs,
):
    vlm = sgl.Engine(**dataclasses.asdict(server_args))

    conv = chat_templates[server_args.chat_template].copy()
    image_token = conv.image_token

    image_url = "https://github.com/sgl-project/sglang/blob/main/examples/assets/example_image.png?raw=true"

    prompt = f"What's in this image?\n{image_token}"

    sampling_params = {
        "temperature": 0.001,
        "max_new_tokens": 30,
    }

    output = vlm.generate(
        prompt=prompt,
        image_data=image_url,
        sampling_params=sampling_params,
    )

    print("===============================")
    print(f"Prompt: {prompt}")
    print(f"Generated text: {output['text']}")

    vlm.shutdown()
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 44-52: Program entry point
````python
# The __main__ condition is necessary here because we use "spawn" to create subprocesses
# Spawn starts a fresh program every time, if there is no __main__, it will run into infinite loop to keep spawning processes from sgl.Engine
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    ServerArgs.add_cli_args(parser)
    args = parser.parse_args()

    server_args = ServerArgs.from_cli_args(args)
    main(server_args)
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, dataclasses
- **Project-specific / 项目相关**: sglang, sglang.srt.parser.conversation.chat_templates, sglang.srt.server_args.ServerArgs
