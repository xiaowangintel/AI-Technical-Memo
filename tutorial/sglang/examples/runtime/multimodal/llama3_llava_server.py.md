# llama3_llava_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/multimodal/llama3_llava_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example configures multimodal serving or embedding generation for a vision-language model. / 该示例用于配置视觉语言模型的多模态服务或多模态嵌入生成。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module overview and usage
````python
"""
Usage:
# Installing latest llava-next: pip install git+https://github.com/LLaVA-VL/LLaVA-NeXT.git
# Installing latest sglang.

# Endpoint Service CLI:
python -m sglang.launch_server --model-path lmms-lab/llama3-llava-next-8b --port=30000

python3 llama3_llava_server.py

Output:
"Friends posing for a fun photo with a life-sized teddy bear, creating a playful and memorable moment."
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 15-24: Import dependencies and runtime symbols
````python
import argparse
import asyncio
import copy
import json

import aiohttp
import requests
from llava.conversation import conv_llava_llama_3

from sglang.utils import normalize_base_url
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 27-32: Handle request
````python
async def send_request(url, data, delay=0):
    await asyncio.sleep(delay)
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json=data) as resp:
            output = await resp.json()
    return output
````
**EN:** This function encapsulates the “Handle request” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle request”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 35-65: Handle concurrent
````python
async def test_concurrent(args):
    url = normalize_base_url(args.host, args.port)

    prompt = "<image>\nPlease generate caption towards this image."
    conv_template = copy.deepcopy(conv_llava_llama_3)
    conv_template.append_message(role=conv_template.roles[0], message=prompt)
    conv_template.append_message(role=conv_template.roles[1], message=None)
    prompt_with_template = conv_template.get_prompt()
    response = []
    for i in range(1):
        response.append(
            send_request(
                url + "/generate",
                {
                    "text": prompt_with_template,
                    "image_data": "https://farm4.staticflickr.com/3175/2653711032_804ff86d81_z.jpg",
                    "sampling_params": {
                        "max_new_tokens": 1024,
                        "temperature": 0,
                        "top_p": 1.0,
                        "presence_penalty": 2,
                        "frequency_penalty": 2,
                        "stop": "<|eot_id|>",
                    },
                },
            )
        )

    rets = await asyncio.gather(*response)
    for ret in rets:
        print(ret["text"])
````
**EN:** This function encapsulates the “Handle concurrent” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle concurrent”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 68-104: Handle streaming
````python
def test_streaming(args):
    url = normalize_base_url(args.host, args.port)
    prompt = "<image>\nPlease generate caption towards this image."
    conv_template = copy.deepcopy(conv_llava_llama_3)
    conv_template.append_message(role=conv_template.roles[0], message=prompt)
    conv_template.append_message(role=conv_template.roles[1], message=None)
    prompt_with_template = conv_template.get_prompt()
    pload = {
        "text": prompt_with_template,
        "sampling_params": {
            "max_new_tokens": 1024,
            "temperature": 0,
            "top_p": 1.0,
            "presence_penalty": 2,
            "frequency_penalty": 2,
            "stop": "<|eot_id|>",
        },
        "image_data": "https://farm4.staticflickr.com/3175/2653711032_804ff86d81_z.jpg",
        "stream": True,
    }
    response = requests.post(
        url + "/generate",
        json=pload,
        stream=True,
    )

    prev = 0
    for chunk in response.iter_lines(decode_unicode=False):
        chunk = chunk.decode("utf-8")
        if chunk and chunk.startswith("data:"):
            if chunk == "data: [DONE]":
                break
            data = json.loads(chunk[5:].strip("\n"))
            output = data["text"].strip()
            print(output[prev:], end="", flush=True)
            prev = len(output)
    print("")
````
**EN:** This function encapsulates the “Handle streaming” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle streaming”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 107-113: Program entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", type=str, default="127.0.0.1")
    parser.add_argument("--port", type=int, default=30000)
    args = parser.parse_args()
    asyncio.run(test_concurrent(args))
    test_streaming(args)
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, asyncio, copy, json
- **Third-party / 第三方**: aiohttp, llava.conversation.conv_llava_llama_3, requests
- **Project-specific / 项目相关**: sglang.utils.normalize_base_url
