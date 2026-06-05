# test_async_llm_dp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/distributed/test_async_llm_dp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `async LLM dp` behavior and regressions in the v1 stack. / 验证 v1 栈中 `async llm dp` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-23)
```python
import asyncio
import os
import time
from contextlib import ExitStack
from dataclasses import dataclass
from typing import Any

import pytest

from vllm import SamplingParams
from vllm.config import VllmConfig
from vllm.engine.arg_utils import AsyncEngineArgs
from vllm.inputs import PromptType
from vllm.outputs import RequestOutput
from vllm.platforms import current_platform
from vllm.sampling_params import RequestOutputKind
from vllm.v1.engine.async_llm import AsyncLLM
from vllm.v1.engine.core_client import DPAsyncMPClient
from vllm.v1.metrics.loggers import StatLoggerBase
from vllm.v1.metrics.stats import IterationStats, MultiModalCacheStats, SchedulerStats
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm, vllm.config, vllm.engine.arg_utils, vllm.inputs, vllm.outputs, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.engine.arg_utils, vllm.inputs, vllm.outputs, ...`。

### Module state / 模块级状态 (line 25)
```python
DP_SIZE = int(os.getenv("DP_SIZE", 2))
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DP_SIZE`. Shared setup calls include `int, os.getenv`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DP_SIZE`。 共享初始化调用包括 `int, os.getenv`。

### generate (lines 28-62)
```python
async def generate(
    engine: AsyncLLM,
    request_id: str,
    prompt: PromptType,
    output_kind: RequestOutputKind,
    max_tokens: int,
    prompt_logprobs: int | None = None,
    data_parallel_rank: int | None = None,
) -> tuple[int, str]:
    # Ensure generate doesn't complete too fast for cancellation test.
    await asyncio.sleep(0.2)

    count = 0
    sampling_params = SamplingParams(
        max_tokens=max_tokens,
        ignore_eos=True,
        output_kind=output_kind,
        temperature=0,
    # ... excerpt omitted for brevity ...
        else:
            count = num_tokens
        await asyncio.sleep(0.0)
    return count, request_id
```
**EN:** Helper function `generate` encapsulates reusable logic for `generate`. Inputs: `engine, request_id, prompt, output_kind, max_tokens, prompt_logprobs, data_parallel_rank`. Key calls include `SamplingParams, engine.generate, asyncio.sleep, len`.
**CN:** 辅助函数 `generate` 封装了与 `generate` 相关的可复用逻辑。 输入参数：`engine, request_id, prompt, output_kind, max_tokens, prompt_logprobs, data_parallel_rank`。 关键调用包括 `SamplingParams, engine.generate, asyncio.sleep, len`。

### test_load (lines 82-186)
```python
async def test_load(
    model: str,
    output_kind: RequestOutputKind,
    data_parallel_backend: str,
    async_scheduling: bool,
):
    if async_scheduling and data_parallel_backend == "ray":
        # TODO(NickLucche) Re-enable when async scheduling is supported
        pytest.skip("Async scheduling is not supported with ray")
    elif data_parallel_backend == "ray" and current_platform.is_rocm():
        pytest.skip(
            "Ray as the distributed executor backend is not supported with ROCm."
        )
    stats_loggers = {}

    @dataclass
    class SimpleStatsLogger(StatLoggerBase):
        init_count: int = 0
    # ... excerpt omitted for brevity ...
            assert num_generated_tokens == NUM_EXPECTED_TOKENS, (
                f"expected {NUM_EXPECTED_TOKENS}"
        assert not engine.output_processor.has_unfinished_requests()
        assert not core_client.engines_running
        assert not core_client.reqs_in_flight
        assert len(stats_loggers) == DP_SIZE
        for sl in stats_loggers.values():
            slogger: SimpleStatsLogger = sl
            assert slogger.finished_req_count > NUM_REQUESTS // (DP_SIZE + 1), (
                f"requests are imbalanced: {stats_loggers}"
            )
```
**EN:** Parameterized test covering `load`. Parameter axes: `model, output_kind, data_parallel_backend, async_scheduling`. Inputs/fixtures: `model, output_kind, data_parallel_backend, async_scheduling`. It exercises `mark.parametrize, pytest.skip, ExitStack, AsyncEngineArgs, AsyncLLM.from_engine_args, after.callback`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `load` 的测试用例。 参数维度：`model, output_kind, data_parallel_backend, async_scheduling`。 输入或 fixture：`model, output_kind, data_parallel_backend, async_scheduling`。 该测试会调用 `mark.parametrize, pytest.skip, ExitStack, AsyncEngineArgs, AsyncLLM.from_engine_args, after.callback`。 代码主体包含 7 个显式断言。

### Module state / 模块级状态 (lines 195-197)
```python
DP_PAUSE_MODEL = "hmellor/tiny-random-LlamaForCausalLM"
DP_PAUSE_MODEL_MOE = "ibm-research/PowerMoE-3b"
DP_PAUSE_PROMPT = "This is a test of data parallel pause"
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DP_PAUSE_MODEL, DP_PAUSE_MODEL_MOE, DP_PAUSE_PROMPT`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DP_PAUSE_MODEL, DP_PAUSE_MODEL_MOE, DP_PAUSE_PROMPT`。

### _get_dp_pause_engine_args (lines 200-210)
```python
def _get_dp_pause_engine_args(expert_parallel: bool) -> AsyncEngineArgs:
    """Engine args for DP pause tests: MoE+EP when expert_parallel else small Llama."""
    model = DP_PAUSE_MODEL_MOE if expert_parallel else DP_PAUSE_MODEL
    return AsyncEngineArgs(
        model=model,
        enforce_eager=True,
        tensor_parallel_size=int(os.getenv("TP_SIZE", 1)),
        data_parallel_size=DP_SIZE,
        data_parallel_backend="mp",
        enable_expert_parallel=expert_parallel,
    )
```
**EN:** Helper function `_get_dp_pause_engine_args` encapsulates reusable logic for `dp pause engine args`. Inputs: `expert_parallel`. Key calls include `AsyncEngineArgs, int, os.getenv`.
**CN:** 辅助函数 `_get_dp_pause_engine_args` 封装了与 `dp pause 引擎 args` 相关的可复用逻辑。 输入参数：`expert_parallel`。 关键调用包括 `AsyncEngineArgs, int, os.getenv`。

### test_dp_pause_resume_basic (lines 215-236)
```python
async def test_dp_pause_resume_basic(expert_parallel: bool):
    """Pausing from the client (one call) pauses all DP ranks; resume clears it."""
    with ExitStack() as after:
        engine_args = _get_dp_pause_engine_args(expert_parallel)
        engine = AsyncLLM.from_engine_args(engine_args)
        after.callback(engine.shutdown)

        assert not await engine.is_paused()
        await engine.pause_generation(mode="abort")
        assert await engine.is_paused()
        await engine.resume_generation()
        assert not await engine.is_paused()

        # Engine still works after resume
        sampling_params = SamplingParams(max_tokens=5)
        async for out in engine.generate(
            request_id="after-resume",
            prompt=DP_PAUSE_PROMPT,
            sampling_params=sampling_params,
        ):
            pass
        assert out.finished
```
**EN:** Parameterized test covering `dp pause resume basic`. Parameter axes: `expert_parallel`. Inputs/fixtures: `expert_parallel`. It exercises `mark.parametrize, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, SamplingParams`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `dp pause resume basic` 的测试用例。 参数维度：`expert_parallel`。 输入或 fixture：`expert_parallel`。 该测试会调用 `mark.parametrize, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, SamplingParams`。 代码主体包含 4 个显式断言。

### test_dp_pause_abort (lines 241-289)
```python
async def test_dp_pause_abort(expert_parallel: bool):
    """Pause with abort from one client aborts in-flight requests on all DP ranks."""
    with ExitStack() as after:
        engine_args = _get_dp_pause_engine_args(expert_parallel)
        engine = AsyncLLM.from_engine_args(engine_args)
        after.callback(engine.shutdown)

        # Start several requests so they are distributed across ranks
        sampling_params = SamplingParams(max_tokens=500, ignore_eos=True)
        num_requests = 4
        outputs_by_id: dict[str, list[RequestOutput]] = {}
        async def gen(rid: str):
            out_list: list[RequestOutput] = []
            outputs_by_id[rid] = out_list
            async for out in engine.generate(
                request_id=rid,
                prompt=DP_PAUSE_PROMPT,
    # ... excerpt omitted for brevity ...
            return out_list[-1] if out_list else None
            assert final is not None, f"req-{i} had no output"
            assert final.finished
            assert final.outputs[0].finish_reason == "abort"
        assert await engine.is_paused()
        assert not await engine.is_paused()
            prompt=DP_PAUSE_PROMPT,
            sampling_params=SamplingParams(max_tokens=5),
        ):
            pass
        assert out.finished
        assert not engine.output_processor.has_unfinished_requests()
```
**EN:** Parameterized test covering `dp pause abort`. Parameter axes: `expert_parallel`. Inputs/fixtures: `expert_parallel`. It exercises `mark.parametrize, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, SamplingParams`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `dp pause abort` 的测试用例。 参数维度：`expert_parallel`。 输入或 fixture：`expert_parallel`。 该测试会调用 `mark.parametrize, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, SamplingParams`。 代码主体包含 7 个显式断言。

### test_dp_pause_keep_then_resume (lines 294-344)
```python
async def test_dp_pause_keep_then_resume(expert_parallel: bool):
    """Start generation, pause after a few tokens (keep mode), resume; verify gap."""

    pause_duration = 2.0
    min_tokens_before_pause = 3
    with ExitStack() as after:
        engine_args = _get_dp_pause_engine_args(expert_parallel)
        engine = AsyncLLM.from_engine_args(engine_args)
        after.callback(engine.shutdown)
        sampling_params = SamplingParams(max_tokens=15, ignore_eos=True)
        token_times: list[tuple[int, float]] = []
        pause_token_idx = 0
        async def generator_task():
            nonlocal pause_token_idx
            out = None
    # ... excerpt omitted for brevity ...
            return out
        assert final_output is not None and final_output.finished
        assert await engine.is_paused() is False
        assert pause_token_idx >= min_tokens_before_pause
            pause_gap = (
                token_times[pause_token_idx][1] - token_times[pause_token_idx - 1][1]
            )
            assert pause_gap >= pause_duration * 0.8, (
                f"Expected gap ~{pause_duration}s after pause, got {pause_gap:.3f}s"
```
**EN:** Parameterized test covering `dp pause keep then resume`. Parameter axes: `expert_parallel`. Inputs/fixtures: `expert_parallel`. It exercises `mark.parametrize, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, SamplingParams`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `dp pause keep then resume` 的测试用例。 参数维度：`expert_parallel`。 输入或 fixture：`expert_parallel`。 该测试会调用 `mark.parametrize, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, SamplingParams`。 代码主体包含 4 个显式断言。

### test_dp_pause_keep_race_staggered_engines (lines 348-405)
```python
async def test_dp_pause_keep_race_staggered_engines():
    """Race: send pause(keep) to engine 0, then add two requests,
    then pause(keep) to engine 1. Ensures no deadlock when pause
    requests are staggered and requests arrive in between."""
    if DP_SIZE != 2:
        pytest.skip("test_dp_pause_keep_race_staggered_engines requires DP_SIZE=2")

    with ExitStack() as after:
        engine_args = _get_dp_pause_engine_args(expert_parallel=True)
        engine = AsyncLLM.from_engine_args(engine_args)
        after.callback(engine.shutdown)
        client = engine.engine_core
        original_call_utility = client.call_utility_async
        mid_pause_tasks: list[asyncio.Task] = []
        async def staggered_pause_keep(method: str, *args) -> Any:
    # ... excerpt omitted for brevity ...
                return await original_call_utility(method, *args)
            return results[0]
        await engine.pause_generation(mode="keep")
        assert await engine.is_paused()
        await engine.resume_generation()
        assert not await engine.is_paused()
        # Let the two requests we sent mid-pause complete
        await asyncio.gather(*mid_pause_tasks)
```
**EN:** Async test covering `dp pause keep race staggered engines`. It exercises `pytest.skip, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, asyncio.create_task`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `dp pause keep race staggered engines` 的测试用例。 该测试会调用 `pytest.skip, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, asyncio.create_task`。 代码主体包含 2 个显式断言。

### test_dp_pause_barrier_request_deadlock (lines 409-515)
```python
async def test_dp_pause_barrier_request_deadlock():
    """
    Test that start_dp_wave is ignored while paused.

    Sequence:
      1. Pause all engines (PAUSED_ALL).
      2. Send barrier to engine 0 only — blocks in dist.barrier(dp_group).
      3. Send a request routed to engine 1.
      4. Wait for any (buggy) START_DP_WAVE propagation.
      5. Send barrier to engine 1 — completes in fixed code, deadlocks
         in buggy code because engine 1 is stuck in EP all-to-all.
    if DP_SIZE != 2:
        pytest.skip("requires DP_SIZE=2")
    with ExitStack() as after:
        engine_args = _get_dp_pause_engine_args(expert_parallel=True)
        engine = AsyncLLM.from_engine_args(engine_args)
    # ... excerpt omitted for brevity ...
        assert await engine.is_paused()
                return await original_call_utility(method, *args)
                return engine_1
            return result
            )
        await engine.resume_generation()
        assert not await engine.is_paused()
        # Let the two requests we sent mid-barrier complete.
        await asyncio.gather(*mid_barrier_tasks)
```
**EN:** Async test covering `dp pause barrier request deadlock`. It exercises `pytest.skip, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, engine.get_supported_tasks`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `dp pause barrier request deadlock` 的测试用例。 该测试会调用 `pytest.skip, ExitStack, _get_dp_pause_engine_args, AsyncLLM.from_engine_args, after.callback, engine.get_supported_tasks`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.engine.arg_utils, vllm.inputs, vllm.outputs, vllm.platforms, vllm.sampling_params, vllm.v1.engine.async_llm, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.engine.arg_utils, vllm.inputs, vllm.outputs, vllm.platforms, vllm.sampling_params, vllm.v1.engine.async_llm, ...`。
- **EN:** Standard-library support: `asyncio, os, time, contextlib, dataclasses, typing`.
- **CN:** 标准库支持：`asyncio, os, time, contextlib, dataclasses, typing`。
