# serving.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/speech_to_text/realtime/serving.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements the serving layer that coordinates preprocessing, engine calls, and response shaping. Scope: speech-to-text realtime. / 实现服务层，负责预处理、引擎调用与响应整形。 作用域：语音转文本 / 实时。

## Line-by-Line Analysis / 逐行分析
### Lines 4-18 — Imports and shared dependencies
```python
import asyncio
from collections.abc import AsyncGenerator
from functools import cached_property
from typing import Literal, cast

import numpy as np

from vllm.engine.protocol import EngineClient, StreamingInput
from vllm.entrypoints.logger import RequestLogger
from vllm.entrypoints.openai.engine.serving import OpenAIServing
from vllm.entrypoints.openai.models.serving import OpenAIServingModels
from vllm.inputs import PromptType
from vllm.logger import init_logger
from vllm.model_executor.models.interfaces import SupportsRealtime
from vllm.renderers.inputs.preprocess import parse_model_prompt
```
**EN:** This import block pulls in standard-library modules such as `asyncio`, `collections`, `functools`, `typing`, uses third-party packages like `numpy`, depends on internal helpers such as `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.entrypoints.openai.engine.serving`, `vllm.entrypoints.openai.models.serving`, `vllm.inputs`, `vllm.logger`.
**CN:** 该导入块引入 `asyncio`, `collections`, `functools`, `typing` 等标准库模块，使用 `numpy` 等第三方库，依赖 `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.entrypoints.openai.engine.serving`, `vllm.entrypoints.openai.models.serving`, `vllm.inputs`, `vllm.logger` 等 vLLM 内部模块。

### Lines 20-20 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 23-88 — Class `OpenAIServingRealtime`
```python
class OpenAIServingRealtime(OpenAIServing):
    """Realtime audio transcription service via WebSocket streaming.

    Provides streaming audio-to-text transcription by transforming audio chunks
    into StreamingInput objects that can be consumed by the engine.
    """

    def __init__(
        self,
        engine_client: EngineClient,
        models: OpenAIServingModels,
        *,
        request_logger: RequestLogger | None,
    ):
        super().__init__(
            engine_client=engine_client,
            models=models,
            request_logger=request_logger,
        )

        self.task_type: Literal["realtime"] = "realtime"

        logger.info("OpenAIServingRealtime initialized for task: %s", self.task_type)

    @cached_property
    def model_cls(self) -> type[SupportsRealtime]:
        """Get the model class that supports transcription."""
        from vllm.model_executor.model_loader import get_model_cls
...
            ),
        )

        async for prompt in stream_input_iter:
            parsed_prompt = parse_model_prompt(model_config, prompt)
            (engine_input,) = await renderer.render_cmpl_async([parsed_prompt])

            yield StreamingInput(prompt=engine_input)
```
**EN:** Class `OpenAIServingRealtime` is introduced here. Its docstring describes the intent as: Realtime audio transcription service via WebSocket streaming.
**CN:** 这里定义类 `OpenAIServingRealtime`。其文档字符串说明了该类的职责与使用方式。

## Key Concepts / 关键概念
- Async request handling / 异步请求处理
- Incremental result emission / 增量结果输出
- Engine-backed serving orchestration / 基于引擎的服务编排
- Realtime/WebSocket handling / 实时/WebSocket 处理
- Tokenization or token-level processing / 分词或 token 级处理
- Prompt or multimodal rendering / 提示词或多模态渲染
- Speech transcription flow / 语音转写流程
- Realtime session flow / 实时会话流程

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `collections`, `functools`, `typing`
- **Third-party / 第三方**: `numpy`
- **vLLM Internal / vLLM 内部**: `vllm.engine.protocol`, `vllm.entrypoints.logger`, `vllm.entrypoints.openai.engine.serving`, `vllm.entrypoints.openai.models.serving`, `vllm.inputs`, `vllm.logger`, `vllm.model_executor.models.interfaces`, `vllm.renderers.inputs.preprocess`, `vllm.model_executor.model_loader`
