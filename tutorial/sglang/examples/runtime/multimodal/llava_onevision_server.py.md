# llava_onevision_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/multimodal/llava_onevision_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example configures multimodal serving or embedding generation for a vision-language model. / 该示例用于配置视觉语言模型的多模态服务或多模态嵌入生成。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module overview and usage
````python
"""
Usage:

python3 -m sglang.launch_server --model-path lmms-lab/llava-onevision-qwen2-72b-ov --port=30000 --tp-size=8

python3 llava_onevision_server.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 9-20: Import dependencies and runtime symbols
````python
import io
import os
import sys
import time

import numpy as np
import openai
import pybase64
import requests
from PIL import Image

from sglang.srt.utils.video_decoder import VideoDecoderWrapper
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 27-38: Handle video
````python
def download_video(url, cache_dir):
    file_path = os.path.join(cache_dir, "jobs.mp4")
    os.makedirs(cache_dir, exist_ok=True)

    response = requests.get(url)
    response.raise_for_status()

    with open(file_path, "wb") as f:
        f.write(response.content)

    print(f"File downloaded and saved to: {file_path}")
    return file_path
````
**EN:** This function encapsulates the “Handle video” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle video”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 41-42: Create OpenAI client
````python
def create_openai_client(base_url):
    return openai.Client(api_key="EMPTY", base_url=base_url)
````
**EN:** This function encapsulates the “Create OpenAI client” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Create OpenAI client”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 45-79: Handle stream request test
````python
def image_stream_request_test(client):
    print("----------------------Image Stream Request Test----------------------")
    stream_request = client.chat.completions.create(
        model="default",
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": "https://raw.githubusercontent.com/sgl-project/sglang/main/assets/logo.png"
                        },
                    },
                    {
                        "type": "text",
                        "text": "Please describe this image. Please list the benchmarks and the models.",
                    },
                ],
            },
        ],
        temperature=0.7,
        max_tokens=1024,
        stream=True,
    )
    stream_response = ""

    for chunk in stream_request:
        if chunk.choices[0].delta.content is not None:
            content = chunk.choices[0].delta.content
            stream_response += content
            sys.stdout.write(content)
            sys.stdout.flush()

    print("-" * 30)
````
**EN:** This function encapsulates the “Handle stream request test” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle stream request test”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 82-126: Handle image stream request test
````python
def multi_image_stream_request_test(client):
    print(
        "----------------------Multi-Images Stream Request Test----------------------"
    )
    stream_request = client.chat.completions.create(
        model="default",
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": "https://raw.githubusercontent.com/sgl-project/sglang/main/assets/logo.png"
                        },
                        "modalities": "multi-images",
                    },
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": "https://raw.githubusercontent.com/sgl-project/sglang/main/examples/assets/example_image.png"
                        },
                        "modalities": "multi-images",
                    },
                    {
                        "type": "text",
                        "text": "I have shown you two images. Please describe the two images to me.",
                    },
                ],
            },
        ],
        temperature=0.7,
        max_tokens=1024,
        stream=True,
    )
    stream_response = ""

    for chunk in stream_request:
        if chunk.choices[0].delta.content is not None:
            content = chunk.choices[0].delta.content
            stream_response += content
            sys.stdout.write(content)
            sys.stdout.flush()

    print("-" * 30)
````
**EN:** This function encapsulates the “Handle image stream request test” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle image stream request test”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 129-149: Handle stream request test
````python
def video_stream_request_test(client, video_path):
    print("------------------------Video Stream Request Test----------------------")
    messages = prepare_video_messages(video_path)

    video_request = client.chat.completions.create(
        model="default",
        messages=messages,
        temperature=0,
        max_tokens=1024,
        stream=True,
    )
    print("-" * 30)
    video_response = ""

    for chunk in video_request:
        if chunk.choices[0].delta.content is not None:
            content = chunk.choices[0].delta.content
            video_response += content
            sys.stdout.write(content)
            sys.stdout.flush()
    print("-" * 30)
````
**EN:** This function encapsulates the “Handle stream request test” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle stream request test”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 152-181: Handle speed test
````python
def image_speed_test(client):
    print("----------------------Image Speed Test----------------------")
    start_time = time.perf_counter()
    request = client.chat.completions.create(
        model="default",
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": "https://raw.githubusercontent.com/sgl-project/sglang/main/assets/logo.png"
                        },
                    },
                    {
                        "type": "text",
                        "text": "Please describe this image. Please list the benchmarks and the models.",
                    },
                ],
            },
        ],
        temperature=0,
        max_tokens=1024,
    )
    end_time = time.perf_counter()
    response = request.choices[0].message.content
    print(response)
    print("-" * 30)
    print_speed_test_results(request, start_time, end_time)
````
**EN:** This function encapsulates the “Handle speed test” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle speed test”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 184-199: Handle speed test
````python
def video_speed_test(client, video_path):
    print("------------------------Video Speed Test------------------------")
    messages = prepare_video_messages(video_path)

    start_time = time.perf_counter()
    video_request = client.chat.completions.create(
        model="default",
        messages=messages,
        temperature=0,
        max_tokens=1024,
    )
    end_time = time.perf_counter()
    video_response = video_request.choices[0].message.content
    print(video_response)
    print("-" * 30)
    print_speed_test_results(video_request, start_time, end_time)
````
**EN:** This function encapsulates the “Handle speed test” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle speed test”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 202-233: Handle video messages
````python
def prepare_video_messages(video_path):
    max_frames_num = 32
    decoder = VideoDecoderWrapper(video_path)
    total_frame_num = len(decoder)
    uniform_sampled_frames = np.linspace(
        0, total_frame_num - 1, max_frames_num, dtype=int
    )
    frame_idx = uniform_sampled_frames.tolist()
    frames = decoder.get_frames_at(frame_idx)

    base64_frames = []
    for frame in frames:
        pil_img = Image.fromarray(frame)
        buff = io.BytesIO()
        pil_img.save(buff, format="JPEG")
        base64_str = pybase64.b64encode(buff.getvalue()).decode("utf-8")
        base64_frames.append(base64_str)

    messages = [{"role": "user", "content": []}]

    for base64_frame in base64_frames:
        frame_format = {
            "type": "image_url",
            "image_url": {"url": f"data:image/jpeg;base64,{base64_frame}"},
            "modalities": "video",
        }
        messages[0]["content"].append(frame_format)

    prompt = {"type": "text", "text": "Please describe the video in detail."}
    messages[0]["content"].append(prompt)

    return messages
````
**EN:** This function encapsulates the “Handle video messages” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle video messages”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 236-247: Handle speed test results
````python
def print_speed_test_results(request, start_time, end_time):
    total_tokens = request.usage.total_tokens
    completion_tokens = request.usage.completion_tokens
    prompt_tokens = request.usage.prompt_tokens

    print(f"Total tokens: {total_tokens}")
    print(f"Completion tokens: {completion_tokens}")
    print(f"Prompt tokens: {prompt_tokens}")
    print(f"Time taken: {end_time - start_time} seconds")
    print(f"Token per second: {total_tokens / (end_time - start_time)}")
    print(f"Completion token per second: {completion_tokens / (end_time - start_time)}")
    print(f"Prompt token per second: {prompt_tokens / (end_time - start_time)}")
````
**EN:** This function encapsulates the “Handle speed test results” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle speed test results”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 250-261: Parse arguments and run the workflow
````python
def main():
    url = "https://raw.githubusercontent.com/EvolvingLMMs-Lab/sglang/dev/onevision_local/assets/jobs.mp4"
    cache_dir = os.path.expanduser("~/.cache")
    video_path = download_video(url, cache_dir)

    client = create_openai_client("http://127.0.0.1:30000/v1")

    image_stream_request_test(client)
    multi_image_stream_request_test(client)
    video_stream_request_test(client, video_path)
    image_speed_test(client)
    video_speed_test(client, video_path)
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 264-265: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。

## Dependencies / 依赖关系
- **Standard library / 标准库**: io, os, sys, time
- **Third-party / 第三方**: PIL.Image, numpy, openai, pybase64, requests
- **Project-specific / 项目相关**: sglang.srt.utils.video_decoder.VideoDecoderWrapper
