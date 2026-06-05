# connection.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/speech_to_text/realtime/connection.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Manages connection/session state for long-lived interactions. Scope: speech-to-text realtime. / 管理长连接交互中的连接与会话状态。 作用域：语音转文本 / 实时。

## Line-by-Line Analysis / 逐行分析
### Lines 4-28 — Imports and shared dependencies
```python
import asyncio
import json
from collections.abc import AsyncGenerator
from http import HTTPStatus
from uuid import uuid4

import numpy as np
import pybase64 as base64
from fastapi import WebSocket
from starlette.websockets import WebSocketDisconnect

from vllm import envs
from vllm.entrypoints.openai.engine.protocol import ErrorResponse, UsageInfo
from vllm.exceptions import VLLMValidationError
from vllm.logger import init_logger

from .protocol import (
    ErrorEvent,
    InputAudioBufferAppend,
    InputAudioBufferCommit,
    SessionCreated,
    TranscriptionDelta,
    TranscriptionDone,
)
from .serving import OpenAIServingRealtime
```
**EN:** This import block pulls in standard-library modules such as `asyncio`, `json`, `collections`, `http`, `uuid`, uses third-party packages like `numpy`, `pybase64`, `fastapi`, `starlette`, depends on internal helpers such as `vllm`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `.protocol`, `.serving`.
**CN:** 该导入块引入 `asyncio`, `json`, `collections`, `http`, `uuid` 等标准库模块，使用 `numpy`, `pybase64`, `fastapi`, `starlette` 等第三方库，依赖 `vllm`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `.protocol`, `.serving` 等 vLLM 内部模块。

### Lines 30-30 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 33-288 — Class `RealtimeConnection`
```python
class RealtimeConnection:
    """Manages WebSocket lifecycle and state for realtime transcription.

    This class handles:
    - WebSocket connection lifecycle (accept, receive, send, close)
    - Event routing (session.update, append, commit)
    - Audio buffering via asyncio.Queue
    - Generation task management
    - Error handling and cleanup
    """

    def __init__(self, websocket: WebSocket, serving: OpenAIServingRealtime):
        self.websocket = websocket
        self.connection_id = f"ws-{uuid4()}"
        self.serving = serving
        self.audio_queue: asyncio.Queue[np.ndarray | None] = asyncio.Queue()
        self.generation_task: asyncio.Task | None = None

        self._is_connected = False
        self._is_model_validated = False

        self._max_audio_filesize_mb = envs.VLLM_MAX_AUDIO_CLIP_FILESIZE_MB

    async def handle_connection(self):
        """Main connection loop."""
        await self.websocket.accept()
        logger.debug("WebSocket connection accepted: %s", self.connection_id)
        self._is_connected = True
...
        # Signal audio stream to stop
        self.audio_queue.put_nowait(None)

        # Cancel generation task if running
        if self.generation_task and not self.generation_task.done():
            self.generation_task.cancel()

        logger.debug("Connection cleanup complete: %s", self.connection_id)
```
**EN:** Class `RealtimeConnection` is introduced here. Its docstring describes the intent as: Manages WebSocket lifecycle and state for realtime transcription.
**CN:** 这里定义类 `RealtimeConnection`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Async request handling / 异步请求处理
- Incremental result emission / 增量结果输出
- Realtime/WebSocket handling / 实时/WebSocket 处理
- Tokenization or token-level processing / 分词或 token 级处理
- Speech transcription flow / 语音转写流程
- Realtime session flow / 实时会话流程
- Speech-to-text serving stack / 语音转文本服务栈

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `json`, `collections`, `http`, `uuid`
- **Third-party / 第三方**: `numpy`, `pybase64`, `fastapi`, `starlette`
- **vLLM Internal / vLLM 内部**: `vllm`, `vllm.entrypoints.openai.engine.protocol`, `vllm.exceptions`, `vllm.logger`, `.protocol`, `.serving`, `vllm.sampling_params`
