# openai_realtime_microphone_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/speech_to_text/realtime/openai_realtime_microphone_client.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Minimal Gradio demo for real-time speech transcription using the vLLM Realtime API / 演示 vLLM 示例目录中与 openai realtime microphone client 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Minimal Gradio demo for real-time speech transcription using the vLLM Realtime API.

Start the vLLM server first:

    vllm serve mistralai/Voxtral-Mini-4B-Realtime-2602 --enforce-eager

Then run this script:

    python openai_realtime_microphone_client.py --host localhost --port 8000

Use --share to create a public Gradio link.

Requirements: websockets, numpy, gradio
"""
```
**EN:** Minimal Gradio demo for real-time speech transcription using the vLLM Realtime API.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import asyncio
import json
import queue
import threading

import gradio as gr
import numpy as np
import pybase64 as base64
import websockets
```
**EN:** This block loads helper libraries such as argparse, asyncio, json, queue, and threading.
**CN:** 这一部分加载 argparse、asyncio、json、queue，以及 threading 等辅助库。

### Top-level setup
```python
SAMPLE_RATE = 16_000

# Global state
audio_queue: queue.Queue = queue.Queue()
transcription_text = ""
is_running = False
ws_url = ""
model = ""
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as SAMPLE_RATE, audio_queue, transcription_text, is_running, ws_url, and model. It also performs early helper calls such as queue.Queue.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 SAMPLE_RATE、audio_queue、transcription_text、is_running、ws_url，以及 model 等变量。它还会提前执行 queue.Queue 等辅助调用。

### Async function: websocket_handler
```python
async def websocket_handler():
    """Connect to WebSocket and handle audio streaming + transcription."""
    global transcription_text, is_running

    async with websockets.connect(ws_url) as ws:
        # Wait for session.created
        await ws.recv()

        # Validate model
        await ws.send(json.dumps({"type": "session.update", "model": model}))

        # Signal ready
        await ws.send(json.dumps({"type": "input_audio_buffer.commit"}))

        async def send_audio():
            while is_running:
                try:
                    chunk = await asyncio.get_event_loop().run_in_executor(
                        None, lambda: audio_queue.get(timeout=0.1)
                    )
                    await ws.send(
                        json.dumps(
                            {"type": "input_audio_buffer.append", "audio": chunk}
                        )
                    )
                except queue.Empty:
                    continue

        async def receive_transcription():
            global transcription_text
            async for message in ws:
                data = json.loads(message)
                if data.get("type") == "transcription.delta":
                    transcription_text += data["delta"]

        await asyncio.gather(send_audio(), receive_transcription())
```
**EN:** Connect to WebSocket and handle audio streaming + transcription.. Key operations include ws.send, json.dumps, websockets.connect, ws.recv, and asyncio.gather.
**CN:** 该函数以异步方式封装示例中的可复用步骤。关键操作包括 ws.send、json.dumps、websockets.connect、ws.recv，以及 asyncio.gather。

### Function: start_websocket
```python
def start_websocket():
    """Start WebSocket connection in background thread."""
    global is_running
    is_running = True
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    try:
        loop.run_until_complete(websocket_handler())
    except Exception as e:
        print(f"WebSocket error: {e}")
```
**EN:** Start WebSocket connection in background thread.. Key operations include asyncio.new_event_loop, asyncio.set_event_loop, loop.run_until_complete, websocket_handler, and print.
**CN:** 该函数封装示例中的可复用步骤。关键操作包括 asyncio.new_event_loop、asyncio.set_event_loop、loop.run_until_complete、websocket_handler，以及 print。

### Function: start_recording
```python
def start_recording():
    """Start the transcription service."""
    global transcription_text
    transcription_text = ""
    thread = threading.Thread(target=start_websocket, daemon=True)
    thread.start()
    return gr.update(interactive=False), gr.update(interactive=True), ""
```
**EN:** Start the transcription service.. Key operations include gr.update, threading.Thread, and thread.start. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。关键操作包括 gr.update、threading.Thread，以及 thread.start。其返回值会继续传给示例管线的下一阶段。

### Function: stop_recording
```python
def stop_recording():
    """Stop the transcription service."""
    global is_running
    is_running = False
    return gr.update(interactive=True), gr.update(interactive=False), transcription_text
```
**EN:** Stop the transcription service.. Key operations include gr.update. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。关键操作包括 gr.update。其返回值会继续传给示例管线的下一阶段。

### Function: process_audio
```python
def process_audio(audio):
    """Process incoming audio and queue for streaming."""
    global transcription_text

    if audio is None or not is_running:
        return transcription_text

    sample_rate, audio_data = audio

    # Convert to mono if stereo
    if len(audio_data.shape) > 1:
        audio_data = audio_data.mean(axis=1)

    # Normalize to float
    if audio_data.dtype == np.int16:
        audio_float = audio_data.astype(np.float32) / 32767.0
    else:
        audio_float = audio_data.astype(np.float32)

    # Resample to 16kHz if needed
    if sample_rate != SAMPLE_RATE:
        num_samples = int(len(audio_float) * SAMPLE_RATE / sample_rate)
        audio_float = np.interp(
            np.linspace(0, len(audio_float) - 1, num_samples),
            np.arange(len(audio_float)),
            audio_float,
        )

    # Convert to PCM16 and base64 encode
    pcm16 = (audio_float * 32767).astype(np.int16)
    b64_chunk = base64.b64encode(pcm16.tobytes()).decode("utf-8")
    audio_queue.put(b64_chunk)

    return transcription_text
```
**EN:** Process incoming audio and queue for streaming.. It works with parameters such as audio. Key operations include len, audio_data.astype, astype, decode, and audio_queue.put. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 audio 等参数。关键操作包括 len、audio_data.astype、astype、decode，以及 audio_queue.put。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
with gr.Blocks(title="Real-time Speech Transcription") as demo:
    gr.Markdown("# Real-time Speech Transcription")
    gr.Markdown("Click **Start** and speak into your microphone.")

    with gr.Row():
        start_btn = gr.Button("Start", variant="primary")
        stop_btn = gr.Button("Stop", variant="stop", interactive=False)

    audio_input = gr.Audio(sources=["microphone"], streaming=True, type="numpy")
    transcription_output = gr.Textbox(label="Transcription", lines=5)

    start_btn.click(
        start_recording, outputs=[start_btn, stop_btn, transcription_output]
    )
    stop_btn.click(stop_recording, outputs=[start_btn, stop_btn, transcription_output])
    audio_input.stream(
        process_audio, inputs=[audio_input], outputs=[transcription_output]
    )
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It also performs early helper calls such as gr.Markdown, gr.Button, gr.Blocks, gr.Audio, and gr.Textbox.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。它还会提前执行 gr.Markdown、gr.Button、gr.Blocks、gr.Audio，以及 gr.Textbox 等辅助调用。

### Entry point
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Realtime WebSocket Transcription with Gradio"
    )
    parser.add_argument(
        "--model",
        type=str,
        default="mistralai/Voxtral-Mini-4B-Realtime-2602",
        help="Model that is served and should be pinged.",
    )
    parser.add_argument(
        "--host", type=str, default="localhost", help="vLLM server host"
    )
    parser.add_argument("--port", type=int, default=8000, help="vLLM server port")
    parser.add_argument(
        "--share", action="store_true", help="Create public Gradio link"
    )
    args = parser.parse_args()

    ws_url = f"ws://{args.host}:{args.port}/v1/realtime"
    model = args.model
    demo.launch(share=args.share)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parser.add_argument, argparse.ArgumentParser, parser.parse_args, and demo.launch.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parser.add_argument、argparse.ArgumentParser、parser.parse_args，以及 demo.launch。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Interactive application wrappers / 交互式应用封装**: UI or service layers turn model calls into an interactive user experience. / 界面或服务层把模型调用包装成交互式用户体验。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `argparse`, `asyncio`, `json`, `queue`, `threading`, `gradio`, `numpy`, `pybase64` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `websocket_handler`, `start_websocket`, `start_recording`, `stop_recording`, `process_audio` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `queue.Queue`, `ws.send`, `json.dumps`, `websockets.connect`, `ws.recv`, `asyncio.gather`, `json.loads`, `send_audio` reveal the main execution path / 这些调用体现了主要执行链路。
