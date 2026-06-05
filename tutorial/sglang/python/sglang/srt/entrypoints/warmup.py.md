# warmup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/warmup.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements warmup logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 warmup 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-4: Import runtime dependencies / 导入运行时依赖
```python
import logging
from typing import TYPE_CHECKING, List
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-7: Import runtime dependencies / 导入运行时依赖
```python
import numpy as np
import tqdm
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-10: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.disaggregation.utils import FAKE_BOOTSTRAP_HOST
from sglang.srt.managers.io_struct import GenerateReqInput
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 12-13: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 15-15: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__file__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 17-17: Provide supporting module logic / 提供辅助模块逻辑
```python
_warmup_registry = {}
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 20-25: Implement warmup / 实现warmup
```python
def warmup(name: str):
    def decorator(fn):
        _warmup_registry[name] = fn
        return fn

    return decorator
```
**EN:** This block implements the function `warmup(name)`. It focuses on handling the warmup responsibilities represented by `warmup`, providing reusable behavior for the warmup pipeline.
**CN:** 该代码块实现函数 `warmup(name)`。它围绕 `warmup` 所承担的 warmup 相关职责展开，为对应处理链路提供可复用能力。

### Lines 28-38: Implement async execute warmups / 实现异步execute warmups
```python
async def execute_warmups(
    disaggregation_mode: str,
    warmup_names: List[str],
    tokenizer_manager: TokenizerManager,
):
    for warmup_name in warmup_names:
        if warmup_name not in _warmup_registry:
            logger.warning(f"Could not find custom warmup {warmup_name}")
            continue
        logger.info(f"Running warmup {warmup_name}")
        await _warmup_registry[warmup_name](disaggregation_mode, tokenizer_manager)
```
**EN:** This block implements the async function `execute_warmups(disaggregation_mode, warmup_names, tokenizer_manager)`. It focuses on handling the warmup responsibilities represented by `execute_warmups`, providing reusable behavior for the warmup pipeline.
**CN:** 该代码块实现异步函数 `execute_warmups(disaggregation_mode, warmup_names, tokenizer_manager)`。它围绕 `execute_warmups` 所承担的 warmup 相关职责展开，为对应处理链路提供可复用能力。

### Lines 39-41: Provide supporting module logic / 提供辅助模块逻辑
```python


@warmup("whisper_autodetect")
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 42-77: Implement async whisper autodetect / 实现异步whisper autodetect
```python
async def whisper_autodetect(
    disaggregation_mode: str, tokenizer_manager: TokenizerManager
):
    """Pre-compile the xgrammar FSM for both Whisper auto-detect regexes.

    The first request that uses each structured-generation regex incurs a
    ~15-20s compilation cost. xgrammar caches compiled grammars by the
    exact regex string, so we warm both the notimestamps and timestamps
    variants here — otherwise the first ``language=None +
    timestamp_granularities`` request would still pay the full spike.
    """
    # A short silent audio encoded as base64 WAV (0.1s, 16kHz, mono) —
    # soundfile produces the WAV header + PCM data from a list of floats.
    import base64
    import io

    import soundfile as sf

    from sglang.srt.entrypoints.openai.transcription_adapters.whisper import (
        FUSED_AUTODETECT_FLAG,
        WHISPER_AUTODETECT_REGEX,
        WHISPER_AUTODETECT_TS_REGEX,
    )

    sr, dur = 16000, 0.1
    n = int(sr * dur)
    buf = io.BytesIO()
    sf.write(buf, [0.0] * n, sr, format="WAV")
    audio_b64 = base64.b64encode(buf.getvalue()).decode()
    audio_data_uri = f"data:audio/wav;base64,{audio_b64}"

    for variant_name, regex in (
        ("notimestamps", WHISPER_AUTODETECT_REGEX),
        ("timestamps", WHISPER_AUTODETECT_TS_REGEX),
    ):
        logger.info(
```
**EN:** This block implements the async function `whisper_autodetect(disaggregation_mode, tokenizer_manager)`. It focuses on Pre-compile the xgrammar FSM for both Whisper auto-detect regexes., providing reusable behavior for the warmup pipeline.
**CN:** 该代码块实现异步函数 `whisper_autodetect(disaggregation_mode, tokenizer_manager)`。它围绕 `whisper_autodetect` 所承担的 warmup 相关职责展开，为对应处理链路提供可复用能力。

### Lines 78-105: Continue async whisper autodetect / 继续说明异步whisper autodetect
```python
            "Compiling Whisper auto-detect regex FSM (%s, one-time, ~15-20s)...",
            variant_name,
        )
        req = GenerateReqInput(
            text="",
            audio_data=audio_data_uri,
            sampling_params={
                "max_new_tokens": 4,
                "temperature": 0,
                "regex": regex,
                "skip_special_tokens": False,
                "spaces_between_special_tokens": False,
                FUSED_AUTODETECT_FLAG: True,
            },
            modalities=["audio"],
        )
        # PD prefill servers assert req.bootstrap_room is not None in the
        # default follow_bootstrap_room scheduler; the fake values match
        # what the voice_chat warmup uses for the same reason.
        if disaggregation_mode != "null":
            req.bootstrap_room = 0
            req.bootstrap_host = FAKE_BOOTSTRAP_HOST
        # Drain the generator so the FSM is fully installed and any
        # downstream exception surfaces instead of being swallowed after
        # the first yield.
        async for _ in tokenizer_manager.generate_request(req, None):
            pass
    logger.info("Whisper auto-detect regex FSMs compiled.")
```
**EN:** This block implements the async function `whisper_autodetect(disaggregation_mode, tokenizer_manager)`. It focuses on Pre-compile the xgrammar FSM for both Whisper auto-detect regexes., providing reusable behavior for the warmup pipeline.
**CN:** 该代码块实现异步函数 `whisper_autodetect(disaggregation_mode, tokenizer_manager)`。它围绕 `whisper_autodetect` 所承担的 warmup 相关职责展开，为对应处理链路提供可复用能力。

### Lines 106-108: Provide supporting module logic / 提供辅助模块逻辑
```python


@warmup("voice_chat")
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 109-127: Implement async voice chat / 实现异步voice 聊天
```python
async def voice_chat(disaggregation_mode: str, tokenizer_manager: TokenizerManager):
    # this warms up the fused_moe triton kernels and caches them
    # if we don't do this we break real time inference for voice chat
    for i in tqdm.trange(1, 512):
        size = i * 4
        generate_req_input = GenerateReqInput(
            input_ids=(np.random.randint(2**16, size=[size])).tolist(),
            sampling_params={
                "max_new_tokens": 30,
                "temperature": 0.8,
                "stop_token_ids": [1],
                "min_p": 0.0,
            },
        )
        if disaggregation_mode != "null":
            generate_req_input.bootstrap_room = 0
            generate_req_input.bootstrap_host = FAKE_BOOTSTRAP_HOST

        await tokenizer_manager.generate_request(generate_req_input, None).__anext__()
```
**EN:** This block implements the async function `voice_chat(disaggregation_mode, tokenizer_manager)`. It focuses on handling the warmup responsibilities represented by `voice_chat`, providing reusable behavior for the warmup pipeline.
**CN:** 该代码块实现异步函数 `voice_chat(disaggregation_mode, tokenizer_manager)`。它围绕 `voice_chat` 所承担的 warmup 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Main callables / 主要可调用对象**: warmup, execute_warmups, whisper_autodetect, voice_chat
- **Domain focus / 领域焦点**: warmup / warmup
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, typing
- **Third-party / 第三方库**: __future__, base64, io, numpy, soundfile, tqdm
- **Local Modules / 本地模块**: sglang.srt.disaggregation.utils, sglang.srt.entrypoints.openai.transcription_adapters.whisper, sglang.srt.managers.io_struct, sglang.srt.managers.tokenizer_manager
