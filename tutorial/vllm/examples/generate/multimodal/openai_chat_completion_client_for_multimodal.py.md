# openai_chat_completion_client_for_multimodal.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/generate/multimodal/openai_chat_completion_client_for_multimodal.py`
- **Repository**: vllm-project/vllm
- **Purpose**: An example showing how to use vLLM to serve multimodal models and run online serving with OpenAI client / 演示多模态生成与预处理流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""An example showing how to use vLLM to serve multimodal models
and run online serving with OpenAI client.

Launch the vLLM server with the following command:

(single image inference with Llava)
vllm serve llava-hf/llava-1.5-7b-hf

(multi-image inference with Phi-3.5-vision-instruct)
vllm serve microsoft/Phi-3.5-vision-instruct --runner generate \
    --trust-remote-code --max-model-len 4096 --limit-mm-per-prompt.image 2

(audio inference with Ultravox)
vllm serve fixie-ai/ultravox-v0_5-llama-3_2-1b \
    --max-model-len 4096 --trust-remote-code

run the script with
python openai_chat_completion_client_for_multimodal.py --chat-type audio
"""
```
**EN:** An example showing how to use vLLM to serve multimodal models and run online serving with OpenAI client.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os

import pybase64 as base64
import requests
from openai import OpenAI

from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as os, pybase64, requests, and openai and pulls in vLLM APIs like vllm.utils.argparse_utils.
**CN:** 这一部分加载 os、pybase64、requests，以及 openai 等辅助库，并引入 vllm.utils.argparse_utils 等 vLLM API。

### Top-level setup
```python
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

client = OpenAI(
    # defaults to os.environ.get("OPENAI_API_KEY")
    api_key=openai_api_key,
    base_url=openai_api_base,
)

headers = {"User-Agent": "vLLM Example Client"}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as openai_api_key, openai_api_base, client, and headers. It also performs early helper calls such as OpenAI.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 openai_api_key、openai_api_base、client，以及 headers 等变量。它还会提前执行 OpenAI 等辅助调用。

### Function: encode_base64_content_from_file
```python
def encode_base64_content_from_file(file_path: str) -> str:
    """Encode a local file content to base64 format."""

    with open(file_path, "rb") as file:
        file_content = file.read()
        result = base64.b64encode(file_content).decode("utf-8")

    return result
```
**EN:** Encode a local file content to base64 format.. It works with parameters such as file_path. Key operations include open, file.read, decode, and base64.b64encode. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 file_path 等参数。关键操作包括 open、file.read、decode，以及 base64.b64encode。其返回值会继续传给示例管线的下一阶段。

### Function: run_text_only
```python
def run_text_only(model: str, max_completion_tokens: int) -> None:
    chat_completion = client.chat.completions.create(
        messages=[{"role": "user", "content": "What's the capital of France?"}],
        model=model,
        max_completion_tokens=max_completion_tokens,
    )

    result = chat_completion.choices[0].message.content
    print("Chat completion output:\n", result)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model and max_completion_tokens. Key operations include client.chat.completions.create and print.
**CN:** 该函数编排端到端工作流。它会处理 model 和 max_completion_tokens 等参数。关键操作包括 client.chat.completions.create 和 print。

### Function: run_single_image
```python
def run_single_image(model: str, max_completion_tokens: int) -> None:
    ## Use image url in the payload
    image_url = "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
    image_file = "/path/to/image.jpg"  # local file
    chat_completion_from_url = client.chat.completions.create(
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's in this image?"},
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                    },
                ],
            }
        ],
        model=model,
        max_completion_tokens=max_completion_tokens,
    )
    # ... key logic omitted for brevity ...
                                "url": f"data:image/jpeg;base64,{local_image_base64}"
                            },
                        },
                    ],
                }
            ],
            model=model,
            max_completion_tokens=max_completion_tokens,
        )

        result = chat_completion_from_local_image_base64.choices[0].message.content
        print("Chat completion output from base64 encoded local image:", result)
    else:
        print(f"Local image file not found at {image_file}, skipping local file test.")
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model and max_completion_tokens. Key operations include print, client.chat.completions.create, os.path.exists, encode_base64_content_from_url, and encode_base64_content_from_file.
**CN:** 该函数编排端到端工作流。它会处理 model 和 max_completion_tokens 等参数。关键操作包括 print、client.chat.completions.create、os.path.exists、encode_base64_content_from_url，以及 encode_base64_content_from_file。

### Function: run_multi_image
```python
def run_multi_image(model: str, max_completion_tokens: int) -> None:
    image_url_duck = "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/duck.jpg"
    image_url_lion = "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/lion.jpg"
    chat_completion_from_url = client.chat.completions.create(
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What are the animals in these images?"},
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url_duck},
                    },
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url_lion},
                    },
                ],
            }
        ],
        model=model,
        max_completion_tokens=max_completion_tokens,
    )

    result = chat_completion_from_url.choices[0].message.content
    print("Chat completion output:\n", result)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model and max_completion_tokens. Key operations include client.chat.completions.create and print.
**CN:** 该函数编排端到端工作流。它会处理 model 和 max_completion_tokens 等参数。关键操作包括 client.chat.completions.create 和 print。

### Function: run_video
```python
def run_video(model: str, max_completion_tokens: int) -> None:
    video_url = "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerFun.mp4"
    video_base64 = encode_base64_content_from_url(video_url)

    ## Use video url in the payload
    chat_completion_from_url = client.chat.completions.create(
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's in this video?"},
                    {
                        "type": "video_url",
                        "video_url": {"url": video_url},
                    },
                ],
            }
        ],
        model=model,
        max_completion_tokens=max_completion_tokens,
    # ... key logic omitted for brevity ...
                    {"type": "text", "text": "What's in this video?"},
                    {
                        "type": "video_url",
                        "video_url": {"url": f"data:video/mp4;base64,{video_base64}"},
                    },
                ],
            }
        ],
        model=model,
        max_completion_tokens=max_completion_tokens,
    )

    result = chat_completion_from_base64.choices[0].message.content
    print("Chat completion output from base64 encoded video:\n", result)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model and max_completion_tokens. Key operations include client.chat.completions.create, print, and encode_base64_content_from_url.
**CN:** 该函数编排端到端工作流。它会处理 model 和 max_completion_tokens 等参数。关键操作包括 client.chat.completions.create、print，以及 encode_base64_content_from_url。

### Function: run_audio
```python
def run_audio(model: str, max_completion_tokens: int) -> None:
    from vllm.assets.audio import AudioAsset

    audio_url = AudioAsset("winning_call").url
    audio_base64 = encode_base64_content_from_url(audio_url)

    # OpenAI-compatible schema (`input_audio`)
    chat_completion_from_base64 = client.chat.completions.create(
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's in this audio?"},
                    {
                        "type": "input_audio",
                        "input_audio": {
                            # Any format supported by soundfile/PyAV is supported
                            "data": audio_base64,
                            "format": "wav",
                        },
    # ... key logic omitted for brevity ...
                        "audio_url": {
                            # Any format supported by soundfile/PyAV is supported
                            "url": f"data:audio/ogg;base64,{audio_base64}"
                        },
                    },
                ],
            }
        ],
        model=model,
        max_completion_tokens=max_completion_tokens,
    )

    result = chat_completion_from_base64.choices[0].message.content
    print("Chat completion output from base64 encoded audio:\n", result)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model and max_completion_tokens. Key operations include client.chat.completions.create, print, encode_base64_content_from_url, and AudioAsset.
**CN:** 该函数编排端到端工作流。它会处理 model 和 max_completion_tokens 等参数。关键操作包括 client.chat.completions.create、print、encode_base64_content_from_url，以及 AudioAsset。

### Function: run_multi_audio
```python
def run_multi_audio(model: str, max_completion_tokens: int) -> None:
    from vllm.assets.audio import AudioAsset

    # Two different audios to showcase batched inference.
    audio_url = AudioAsset("winning_call").url
    audio_base64 = encode_base64_content_from_url(audio_url)
    audio_url2 = AudioAsset("azacinto_foscolo").url
    audio_base64_2 = encode_base64_content_from_url(audio_url2)

    # OpenAI-compatible schema (`input_audio`)
    chat_completion_from_base64 = client.chat.completions.create(
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "Are these two audios the same?"},
                    {
                        "type": "input_audio",
                        "input_audio": {
                            "data": audio_base64,
                            "format": "wav",
                        },
                    },
                    {
                        "type": "input_audio",
                        "input_audio": {
                            "data": audio_base64_2,
                            "format": "wav",
                        },
                    },
                ],
            }
        ],
        model=model,
        max_completion_tokens=max_completion_tokens,
    )

    result = chat_completion_from_base64.choices[0].message.content
    print("Chat completion output from input audio:\n", result)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as model and max_completion_tokens. Key operations include encode_base64_content_from_url, AudioAsset, client.chat.completions.create, and print.
**CN:** 该函数编排端到端工作流。它会处理 model 和 max_completion_tokens 等参数。关键操作包括 encode_base64_content_from_url、AudioAsset、client.chat.completions.create，以及 print。

### Top-level setup
```python
example_function_map = {
    "text-only": run_text_only,
    "single-image": run_single_image,
    "multi-image": run_multi_image,
    "multi-audio": run_multi_audio,
    "video": run_video,
    "audio": run_audio,
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as example_function_map.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 example_function_map 等变量。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser(
        description="Demo on using OpenAI client for online serving with "
        "multimodal language models served with vLLM."
    )
    parser.add_argument(
        "--chat-type",
        "-c",
        type=str,
        default="single-image",
        choices=list(example_function_map.keys()),
        help="Conversation type with multimodal data.",
    )
    parser.add_argument(
        "--max-completion-tokens",
        "-n",
        type=int,
        default=128,
        help="Maximum number of tokens to generate for each completion.",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, FlexibleArgumentParser, parser.parse_args, list, and example_function_map.keys. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、FlexibleArgumentParser、parser.parse_args、list，以及 example_function_map.keys。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args) -> None:
    chat_type = args.chat_type
    model = client.models.list().data[0].id
    example_function_map[chat_type](model, args.max_completion_tokens)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include client.models.list.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 client.models.list。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    main(args)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `pybase64`, `requests`, `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `encode_base64_content_from_file`, `run_text_only`, `run_single_image`, `run_multi_image`, `run_video`, `run_audio`, `run_multi_audio`, `parse_args` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `OpenAI`, `open`, `file.read`, `decode`, `base64.b64encode`, `client.chat.completions.create`, `print`, `os.path.exists` reveal the main execution path / 这些调用体现了主要执行链路。
