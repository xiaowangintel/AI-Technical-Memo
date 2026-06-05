# openai_transcription_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/speech_to_text/openai/openai_transcription_client.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This script demonstrates how to use the vLLM API server to perform audio transcription with the `openai/whisper-large-v3` model / 演示 vLLM 示例目录中与 openai transcription client 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This script demonstrates how to use the vLLM API server to perform audio
transcription with the `openai/whisper-large-v3` model.

Before running this script, you must start the vLLM server with the following command:

    vllm serve openai/whisper-large-v3

Requirements:
- vLLM with audio support
- openai Python SDK
- httpx for streaming support

The script performs:
1. Synchronous transcription using OpenAI-compatible API.
2. Streaming transcription using raw HTTP request to the vLLM server.
"""
```
**EN:** This script demonstrates how to use the vLLM API server to perform audio transcription with the `openai/whisper-large-v3` model.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import asyncio

from openai import AsyncOpenAI, OpenAI

from vllm.assets.audio import AudioAsset
```
**EN:** This block loads helper libraries such as argparse, asyncio, and openai and pulls in vLLM APIs like vllm.assets.audio.
**CN:** 这一部分加载 argparse、asyncio，以及 openai 等辅助库，并引入 vllm.assets.audio 等 vLLM API。

### Function: sync_openai
```python
def sync_openai(
    audio_path: str,
    client: OpenAI,
    model: str,
    *,
    repetition_penalty: float = 1.3,
    hotwords: str = None,
):
    """
    Perform synchronous transcription using OpenAI-compatible API.
    """
    with open(audio_path, "rb") as f:
        transcription = client.audio.transcriptions.create(
            file=f,
            model=model,
            language="en",
            response_format="json",
            temperature=0.0,
            # Additional sampling params not provided by OpenAI API.
            extra_body=dict(
                seed=4419,
                repetition_penalty=repetition_penalty,
                hotwords=hotwords,
            ),
        )
        print("transcription result [sync]:", transcription.text)
```
**EN:** Perform synchronous transcription using OpenAI-compatible API.. It works with parameters such as audio_path, client, and model. Key operations include open, client.audio.transcriptions.create, print, and dict.
**CN:** 该函数封装示例中的可复用步骤。它会处理 audio_path、client，以及 model 等参数。关键操作包括 open、client.audio.transcriptions.create、print，以及 dict。

### Async function: stream_openai_response
```python
async def stream_openai_response(
    audio_path: str, client: AsyncOpenAI, model: str, hotwords: str = None
):
    """
    Perform asynchronous transcription using OpenAI-compatible API.
    """
    print("\ntranscription result [stream]:", end=" ")
    with open(audio_path, "rb") as f:
        transcription = await client.audio.transcriptions.create(
            file=f,
            model=model,
            language="en",
            response_format="json",
            temperature=0.0,
            # Additional sampling params not provided by OpenAI API.
            extra_body=dict(
                seed=420,
                top_p=0.6,
                hotwords=hotwords,
            ),
            stream=True,
        )
        async for chunk in transcription:
            if chunk.choices:
                content = chunk.choices[0].get("delta", {}).get("content")
                print(content, end="", flush=True)

    print()  # Final newline after stream ends
```
**EN:** Perform asynchronous transcription using OpenAI-compatible API.. It works with parameters such as audio_path, client, model, and hotwords. Key operations include print, get, open, client.audio.transcriptions.create, and dict.
**CN:** 该函数以异步方式向调用方持续输出增量更新。它会处理 audio_path、client、model，以及 hotwords 等参数。关键操作包括 print、get、open、client.audio.transcriptions.create，以及 dict。

### Function: stream_api_response
```python
def stream_api_response(audio_path: str, model: str, openai_api_base: str):
    """
    Perform streaming transcription using raw HTTP requests to the vLLM API server.
    """
    import json
    import os

    import requests

    api_url = f"{openai_api_base}/audio/transcriptions"
    headers = {"User-Agent": "Transcription-Client"}
    with open(audio_path, "rb") as f:
        files = {"file": (os.path.basename(audio_path), f)}
        data = {
            "stream": "true",
            "model": model,
            "language": "en",
            "response_format": "json",
        }

        print("\ntranscription result [stream]:", end=" ")
        response = requests.post(
            api_url, headers=headers, files=files, data=data, stream=True
        )
        for chunk in response.iter_lines(
            chunk_size=8192, decode_unicode=False, delimiter=b"\n"
        ):
            if chunk:
                data = chunk[len("data: ") :]
                data = json.loads(data.decode("utf-8"))
                data = data["choices"][0]
                delta = data["delta"]["content"]
                print(delta, end="", flush=True)

                finish_reason = data.get("finish_reason")
                if finish_reason is not None:
                    print(f"\n[Stream finished reason: {finish_reason}]")
                    break
```
**EN:** Perform streaming transcription using raw HTTP requests to the vLLM API server.. It works with parameters such as audio_path, model, and openai_api_base. Key operations include print, open, requests.post, response.iter_lines, and os.path.basename.
**CN:** 该函数向调用方持续输出增量更新。它会处理 audio_path、model，以及 openai_api_base 等参数。关键操作包括 print、open、requests.post、response.iter_lines，以及 os.path.basename。

### Function: main
```python
def main(args):
    mary_had_lamb = str(AudioAsset("mary_had_lamb").get_local_path())
    winning_call = str(AudioAsset("winning_call").get_local_path())

    # Modify OpenAI's API key and API base to use vLLM's API server.
    openai_api_key = "EMPTY"
    openai_api_base = "http://localhost:8000/v1"
    client = OpenAI(
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    model = client.models.list().data[0].id
    print(f"Using model: {model}")

    # Run the synchronous function
    sync_openai(
        audio_path=args.audio_path if args.audio_path else mary_had_lamb,
        client=client,
        model=model,
        repetition_penalty=args.repetition_penalty,
        hotwords=args.hotwords,
    )

    # Run the asynchronous function
    if "openai" in model:
        client = AsyncOpenAI(
            api_key=openai_api_key,
            base_url=openai_api_base,
        )
        asyncio.run(
            stream_openai_response(
                args.audio_path if args.audio_path else winning_call,
                client,
                model,
                hotwords=args.hotwords,
            )
        )
    else:
        stream_api_response(
            args.audio_path if args.audio_path else winning_call,
            model,
            openai_api_base,
        )
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include str, get_local_path, AudioAsset, OpenAI, and print.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 str、get_local_path、AudioAsset、OpenAI，以及 print。

### Entry point
```python
if __name__ == "__main__":
    # setup argparser
    parser = argparse.ArgumentParser(
        description="OpenAI Transcription Client using vLLM API Server"
    )
    parser.add_argument(
        "--audio_path",
        type=str,
        default=None,
        help="The path to the audio file to transcribe.",
    )
    parser.add_argument(
        "--repetition_penalty",
        type=float,
        default=1.3,
        help="repetition penalty",
    )
    parser.add_argument(
        "--hotwords",
        type=str,
        default=None,
        help="hotwords",
    )
    args = parser.parse_args()
    main(args)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parser.add_argument, argparse.ArgumentParser, parser.parse_args, and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parser.add_argument、argparse.ArgumentParser、parser.parse_args，以及 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm.assets.audio` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `asyncio`, `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `sync_openai`, `stream_openai_response`, `stream_api_response`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `open`, `client.audio.transcriptions.create`, `print`, `dict`, `get`, `requests.post`, `response.iter_lines`, `os.path.basename` reveal the main execution path / 这些调用体现了主要执行链路。
