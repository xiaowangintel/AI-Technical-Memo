# openai_realtime_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/speech_to_text/realtime/openai_realtime_client.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This script demonstrates how to use the vLLM Realtime WebSocket API to perform audio transcription by uploading an audio file / 演示 vLLM 示例目录中与 openai realtime client 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This script demonstrates how to use the vLLM Realtime WebSocket API to perform
audio transcription by uploading an audio file.

Before running this script, you must start the vLLM server with a realtime-capable
model, for example:

    vllm serve mistralai/Voxtral-Mini-4B-Realtime-2602 --enforce-eager

Requirements:
- vllm with audio support
- websockets
- numpy

The script:
1. Connects to the Realtime WebSocket endpoint
2. Converts an audio file to PCM16 @ 16kHz
3. Sends audio chunks to the server
4. Receives and prints transcription as it streams
"""
```
**EN:** This script demonstrates how to use the vLLM Realtime WebSocket API to perform audio transcription by uploading an audio file.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import asyncio
import json

import numpy as np
import pybase64 as base64
import websockets

from vllm.assets.audio import AudioAsset
from vllm.multimodal.media.audio import load_audio
```
**EN:** This block loads helper libraries such as argparse, asyncio, json, numpy, and pybase64 and pulls in vLLM APIs like vllm.assets.audio and vllm.multimodal.media.audio.
**CN:** 这一部分加载 argparse、asyncio、json、numpy，以及 pybase64 等辅助库，并引入 vllm.assets.audio 和 vllm.multimodal.media.audio 等 vLLM API。

### Function: audio_to_pcm16_base64
```python
def audio_to_pcm16_base64(audio_path: str) -> str:
    """
    Load an audio file and convert it to base64-encoded PCM16 @ 16kHz.
    """
    # Load audio and resample to 16kHz mono
    audio, _ = load_audio(audio_path, sr=16000, mono=True)
    # Convert to PCM16
    pcm16 = (audio * 32767).astype(np.int16)
    # Encode as base64
    return base64.b64encode(pcm16.tobytes()).decode("utf-8")
```
**EN:** Load an audio file and convert it to base64-encoded PCM16 @ 16kHz.. It works with parameters such as audio_path. Key operations include load_audio, astype, decode, base64.b64encode, and pcm16.tobytes. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 audio_path 等参数。关键操作包括 load_audio、astype、decode、base64.b64encode，以及 pcm16.tobytes。其返回值会继续传给示例管线的下一阶段。

### Async function: realtime_transcribe
```python
async def realtime_transcribe(audio_path: str, host: str, port: int, model: str):
    """
    Connect to the Realtime API and transcribe an audio file.
    """
    uri = f"ws://{host}:{port}/v1/realtime"

    async with websockets.connect(uri) as ws:
        # Wait for session.created
        response = json.loads(await ws.recv())
        if response["type"] == "session.created":
            print(f"Session created: {response['id']}")
        else:
            print(f"Unexpected response: {response}")
            return

        # Validate model
        await ws.send(json.dumps({"type": "session.update", "model": model}))

        # Signal ready to start
        await ws.send(json.dumps({"type": "input_audio_buffer.commit"}))
    # ... key logic omitted for brevity ...
        # Receive transcription
        print("Transcription: ", end="", flush=True)
        while True:
            response = json.loads(await ws.recv())
            if response["type"] == "transcription.delta":
                print(response["delta"], end="", flush=True)
            elif response["type"] == "transcription.done":
                print(f"\n\nFinal transcription: {response['text']}")
                if response.get("usage"):
                    print(f"Usage: {response['usage']}")
                break
            elif response["type"] == "error":
                print(f"\nError: {response['error']}")
                break
```
**EN:** Connect to the Realtime API and transcribe an audio file.. It works with parameters such as audio_path, host, port, and model. Key operations include print, ws.send, json.dumps, json.loads, and len.
**CN:** 该函数以异步方式封装示例中的可复用步骤。它会处理 audio_path、host、port，以及 model 等参数。关键操作包括 print、ws.send、json.dumps、json.loads，以及 len。

### Function: main
```python
def main(args):
    if args.audio_path:
        audio_path = args.audio_path
    else:
        # Use default audio asset
        audio_path = str(AudioAsset("mary_had_lamb").get_local_path())
        print(f"No audio path provided, using default: {audio_path}")

    asyncio.run(realtime_transcribe(audio_path, args.host, args.port, args.model))
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include asyncio.run, str, print, realtime_transcribe, and get_local_path.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 asyncio.run、str、print、realtime_transcribe，以及 get_local_path。

### Entry point
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Realtime WebSocket Transcription Client"
    )
    parser.add_argument(
        "--model",
        type=str,
        default="mistralai/Voxtral-Mini-4B-Realtime-2602",
        help="Model that is served and should be pinged.",
    )
    parser.add_argument(
        "--audio_path",
    # ... key logic omitted for brevity ...
    parser.add_argument(
        "--port",
        type=int,
        default=8000,
        help="vLLM server port (default: 8000)",
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
- **vLLM APIs / vLLM API**: `vllm.assets.audio`, `vllm.multimodal.media.audio` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `asyncio`, `json`, `numpy`, `pybase64`, `websockets` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `audio_to_pcm16_base64`, `realtime_transcribe`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `load_audio`, `astype`, `decode`, `base64.b64encode`, `pcm16.tobytes`, `print`, `ws.send`, `json.dumps` reveal the main execution path / 这些调用体现了主要执行链路。
