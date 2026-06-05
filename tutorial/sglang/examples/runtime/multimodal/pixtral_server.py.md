# pixtral_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/multimodal/pixtral_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example configures multimodal serving or embedding generation for a vision-language model. / 该示例用于配置视觉语言模型的多模态服务或多模态嵌入生成。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module overview and usage
````python
"""
Usage:
# Run a Pixtral model with SGLang:
# HuggingFace:
python -m sglang.launch_server --model-path mistral-community/pixtral-12b --port=30000
# ModelScope:
python -m sglang.launch_server --model-path AI-ModelScope/pixtral-12b --port=30000

# Then test it with:
python pixtral_server.py

This script tests Pixtral model with both single and multiple images.
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 15-22: Import dependencies and runtime symbols
````python
import argparse
import asyncio
import json

import aiohttp
import requests

from sglang.utils import normalize_base_url
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 24-25: Set top-level configuration
````python
IMAGE_TOKEN_SEP = "\n[IMG]"
ROUTE = "/generate"
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 28-33: Handle request
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

### Lines 36-70: Handle concurrent
````python
async def test_concurrent(args):
    url = f"{normalize_base_url(args.host, args.port)}{ROUTE}"

    # Single image test
    if args.single_image:
        prompt = f"<s>[INST]Describe this image in detail.{IMAGE_TOKEN_SEP}[/INST]"
        image_url = "https://picsum.photos/id/237/400/300"
        modality = ["image"]
    # Multiple images test
    else:
        image_urls = [
            "https://picsum.photos/id/237/400/300",
            "https://picsum.photos/id/27/500/500",
        ]
        prompt = f"<s>[INST]How many photos are there? Describe each in a very short sentence.{IMAGE_TOKEN_SEP * len(image_urls)}[/INST]"
        image_url = image_urls
        modality = ["multi-images"]

    response = await send_request(
        url,
        {
            "text": prompt,
            "image_data": image_url,
            "sampling_params": {
                "max_new_tokens": 100,
                "temperature": 0.7,
                "top_p": 0.9,
            },
            "modalities": modality,
        },
    )

    print(f"Response: {response}")
    if "text" in response:
        print("\nOutput text:", response["text"])
````
**EN:** This function encapsulates the “Handle concurrent” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle concurrent”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 73-112: Handle streaming
````python
def test_streaming(args):
    url = f"{normalize_base_url(args.host, args.port)}/generate"

    # Single image test
    if args.single_image:
        prompt = f"<s>[INST]Describe this image in detail.{IMAGE_TOKEN_SEP}[/INST]"
        image_data = "https://picsum.photos/id/237/400/300"
        modality = ["image"]
    # Multiple images test
    else:
        image_urls = [
            "https://picsum.photos/id/237/400/300",
            "https://picsum.photos/id/27/500/500",
        ]
        prompt = f"<s>[INST]How many photos are there? Describe each in a very short sentence.{IMAGE_TOKEN_SEP * len(image_urls)}[/INST]"
        image_data = image_urls
        modality = ["multi-images"]

    pload = {
        "text": prompt,
        "image_data": image_data,
        "sampling_params": {"max_new_tokens": 100, "temperature": 0.7, "top_p": 0.9},
        "modalities": modality,
        "stream": True,
    }

    response = requests.post(url, json=pload, stream=True)

    print("Streaming response:")
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
    print("\n")
````
**EN:** This function encapsulates the “Handle streaming” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle streaming”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 115-129: Program entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", type=str, default="127.0.0.1")
    parser.add_argument("--port", type=int, default=30000)
    parser.add_argument(
        "--single-image",
        action="store_true",
        help="Test with single image instead of multiple images",
    )
    parser.add_argument("--no-stream", action="store_true", help="Don't test streaming")
    args = parser.parse_args()

    asyncio.run(test_concurrent(args))
    if not args.no_stream:
        test_streaming(args)
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, asyncio, json
- **Third-party / 第三方**: aiohttp, requests
- **Project-specific / 项目相关**: sglang.utils.normalize_base_url
