# test_streaming_input.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/e2e/general/test_streaming_input.py`
- **Repository**: vllm-project/vllm
- **Purpose**: End-to-end tests for the streaming input feature in AsyncLLM. / 该文件的文档字符串表明其用途：`end-to-end tests for the streaming input feature in asyncllm`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-13)
```python
"""
End-to-end tests for the streaming input feature in AsyncLLM.

These tests verify that:
1. Streaming inputs work correctly with bunched inputs (queued)
2. Streaming inputs work correctly with spaced out inputs
3. Outputs are equivalent whether inputs are bunched or spaced
4. Cancelling the output stream correctly aborts the session
5. Closing the input stream correctly signals completion
6. Queued inputs are cancelled when the session is aborted
"""
```
**EN:** Module docstring that declares the scope of the file: End-to-end tests for the streaming input feature in AsyncLLM.
**CN:** 模块文档字符串直接说明了文件范围：`end-to-end tests for the streaming input feature in asyncllm`。

### Imports and setup / 导入与设置 (lines 15-27)
```python
import asyncio
from collections.abc import AsyncGenerator

import pytest
import pytest_asyncio

from vllm import SamplingParams
from vllm.engine.protocol import StreamingInput
from vllm.outputs import RequestOutput
from vllm.platforms import current_platform
from vllm.sampling_params import RequestOutputKind
from vllm.utils.torch_utils import set_default_torch_num_threads
from vllm.v1.engine.async_llm import AsyncLLM
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, pytest_asyncio`. vLLM modules under test include `vllm, vllm.engine.protocol, vllm.outputs, vllm.platforms, vllm.sampling_params, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, pytest_asyncio`。 被测试的 vLLM 模块包括 `vllm, vllm.engine.protocol, vllm.outputs, vllm.platforms, vllm.sampling_params, ...`。

### Module state / 模块级状态 (lines 29-33)
```python
if not current_platform.is_cuda():
    pytest.skip(reason="V1 currently only supported on CUDA.", allow_module_level=True)

# Use a small model that doesn't require authentication for fast tests
MODEL = "facebook/opt-125m"
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MODEL`. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `current_platform.is_cuda, pytest.skip`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MODEL`。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `current_platform.is_cuda, pytest.skip`。

### engine (lines 37-55)
```python
async def engine():
    """Create an AsyncLLM engine for the test.

    Note: Using function scope because pytest_asyncio creates a new event loop
    for each test, and the output_handler task gets cancelled between tests
    with module scope.
    """
    from vllm.engine.arg_utils import AsyncEngineArgs

    engine_args = AsyncEngineArgs(
        model=MODEL, enforce_eager=True, gpu_memory_utilization=0.7
    )
    with set_default_torch_num_threads(1):
        engine = AsyncLLM.from_engine_args(engine_args)
    try:
        yield engine
    finally:
        engine.shutdown()
        await asyncio.sleep(0.1)
```
**EN:** Fixture/helper `engine` prepares reusable state for downstream tests. Key calls include `pytest_asyncio.fixture, AsyncEngineArgs, set_default_torch_num_threads, AsyncLLM.from_engine_args, engine.shutdown, asyncio.sleep`.
**CN:** `engine` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `pytest_asyncio.fixture, AsyncEngineArgs, set_default_torch_num_threads, AsyncLLM.from_engine_args, engine.shutdown, asyncio.sleep`。

### get_sampling_params (lines 58-65)
```python
def get_sampling_params(max_tokens: int = 20) -> SamplingParams:
    """Create sampling params for streaming input tests."""
    return SamplingParams(
        max_tokens=max_tokens,
        ignore_eos=True,
        output_kind=RequestOutputKind.DELTA,
        temperature=0.0,  # Deterministic for reproducibility
    )
```
**EN:** Helper function `get_sampling_params` encapsulates reusable logic for `sampling params`. Inputs: `max_tokens`. Key calls include `SamplingParams`.
**CN:** 辅助函数 `get_sampling_params` 封装了与 `采样 params` 相关的可复用逻辑。 输入参数：`max_tokens`。 关键调用包括 `SamplingParams`。

### collect_outputs (lines 68-78)
```python
async def collect_outputs(
    output_gen: AsyncGenerator[RequestOutput, None],
) -> tuple[list[RequestOutput], str]:
    """Collect all outputs from a generate call, return outputs and full text."""
    outputs: list[RequestOutput] = []
    full_text = ""
    async for output in output_gen:
        outputs.append(output)
        if output.outputs and output.outputs[0].text:
            full_text += output.outputs[0].text
    return outputs, full_text
```
**EN:** Helper function `collect_outputs` encapsulates reusable logic for `collect outputs`. Inputs: `output_gen`. Key calls include `outputs.append`.
**CN:** 辅助函数 `collect_outputs` 封装了与 `collect outputs` 相关的可复用逻辑。 输入参数：`output_gen`。 关键调用包括 `outputs.append`。

### test_streaming_input_bunched (lines 82-118)
```python
async def test_streaming_input_bunched(engine: AsyncLLM):
    """Test streaming input where all inputs are sent at once (bunched/queued).

    This tests the case where multiple inputs arrive before any completes.
    The inputs should be queued and processed in sequence.
    """
    request_id = "test_bunched"
    sampling_params = get_sampling_params(max_tokens=10)
    # Create an input generator that yields all inputs quickly
    async def bunched_input_generator() -> AsyncGenerator[StreamingInput, None]:
        # Send multiple inputs rapidly - they should be queued
        yield StreamingInput(prompt="Hello, my name is")
        yield StreamingInput(prompt=" Alice and I like")
        yield StreamingInput(prompt=" to code in Python")
    outputs, full_text = await collect_outputs(
        engine.generate(
    # ... excerpt omitted for brevity ...
    assert len(outputs) > 0, "Should have received outputs"
    assert outputs[-1].finished, "Last output should be marked as finished"
    for output in outputs[:-1]:
        assert not output.finished, "Intermediate outputs should not be finished"
    # Verify we generated some text
    assert len(full_text) > 0, "Should have generated text"
    print(f"Bunched test generated: {full_text}")
```
**EN:** Async test covering `streaming input bunched`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, print, collect_outputs, len, StreamingInput`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input bunched` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, print, collect_outputs, len, StreamingInput`。 代码主体包含 4 个显式断言。

### test_streaming_input_spaced (lines 122-187)
```python
async def test_streaming_input_spaced(engine: AsyncLLM):
    """Test streaming input where inputs are spaced out.

    This tests the case where each input completes processing before the
    next one is sent. Each chunk should be prefilled, generate tokens,
    then the next chunk should be processed.
    """
    request_id = "test_spaced"
    sampling_params = get_sampling_params(max_tokens=10)
    # Track when each input is sent
    input_times: list[float] = []
    outputs_per_chunk: list[int] = [0, 0, 0]
    current_chunk = 0
    async def spaced_input_generator() -> AsyncGenerator[StreamingInput, None]:
        nonlocal current_chunk
        import time
    # ... excerpt omitted for brevity ...
        yield StreamingInput(prompt="Hello, my name is")
        yield StreamingInput(prompt=" Alice and I like")
        yield StreamingInput(prompt=" to code in Python")
    assert len(outputs) > 0, "Should have received outputs"
    assert outputs[-1].finished, "Last output should be marked as finished"
    # (with spaced inputs, we should see outputs distributed across chunks)
    chunks_with_outputs = sum(1 for c in outputs_per_chunk if c > 0)
    assert chunks_with_outputs >= 1, "Should have outputs from at least one chunk"
    print(f"Spaced test generated: {full_text}")
    print(f"Outputs per chunk: {outputs_per_chunk}")
```
**EN:** Async test covering `streaming input spaced`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, engine.generate, sum, print, input_times.append`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input spaced` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, engine.generate, sum, print, input_times.append`。 代码主体包含 3 个显式断言。

### test_streaming_input_output_equivalence (lines 191-227)
```python
async def test_streaming_input_output_equivalence(engine: AsyncLLM):
    """Test that bunched and spaced inputs produce equivalent outputs.

    When the same prompts are provided either bunched or spaced,
    the final concatenated output should be the same (with deterministic
    sampling).
    """
    prompts = ["Hello, my name is", " Bob and I work", " at Anthropic"]
    sampling_params = get_sampling_params(max_tokens=15)
    # Test bunched inputs
    async def bunched_gen() -> AsyncGenerator[StreamingInput, None]:
        for prompt in prompts:
            yield StreamingInput(prompt=prompt)
    _, bunched_text = await collect_outputs(
        engine.generate(bunched_gen(), sampling_params, "equiv_bunched")
    )
    # ... excerpt omitted for brevity ...
    assert bunched_text == spaced_text, (
        f"Bunched and spaced should produce same output.\n"
        f"Bunched: {bunched_text!r}\n"
        f"Spaced: {spaced_text!r}"
    print(f"Equivalence test passed. Generated: {bunched_text}")
```
**EN:** Async test covering `streaming input output equivalence`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, print, collect_outputs, engine.generate, StreamingInput`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input output equivalence` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, print, collect_outputs, engine.generate, StreamingInput`。 代码主体包含 1 个显式断言。

### test_streaming_input_cancel_output_stream (lines 231-285)
```python
async def test_streaming_input_cancel_output_stream(engine: AsyncLLM):
    """Test that cancelling the output stream aborts the entire session.

    When the consumer cancels iteration over the output generator,
    the session should be aborted including any queued inputs.
    """
    request_id = "test_cancel_output"
    sampling_params = get_sampling_params(max_tokens=1000)
    input_completed = asyncio.Event()
    input_task_cancelled = False
    async def slow_input_generator() -> AsyncGenerator[StreamingInput, None]:
        nonlocal input_task_cancelled
        try:
            yield StreamingInput(prompt="Tell me a very long story about")
            yield StreamingInput(prompt=" a dragon and a knight")
    # ... excerpt omitted for brevity ...
            yield StreamingInput(prompt=" who become friends")
    assert outputs_received >= 5, "Should have received outputs before cancel"
    assert input_task_cancelled, "Input task should have been cancelled"
    # Verify the session is properly cleaned up
    assert not engine.output_processor.has_unfinished_requests(), (
        "Should have no unfinished requests after cancel"
    )
    print(f"Cancel test passed. Received {outputs_received} outputs before cancel")
```
**EN:** Async test covering `streaming input cancel output stream`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, asyncio.Event, engine.generate, print, slow_input_generator`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input cancel output stream` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, asyncio.Event, engine.generate, print, slow_input_generator`。 代码主体包含 3 个显式断言。

### test_streaming_input_close_signals_completion (lines 289-322)
```python
async def test_streaming_input_close_signals_completion(engine: AsyncLLM):
    """Test that closing the input stream signals completion.

    When the input generator finishes (naturally or via return),
    the session should complete with finished=True on the last output.
    """
    request_id = "test_close_completion"
    sampling_params = get_sampling_params(max_tokens=15)
    input_generator_finished = False
    async def limited_input_generator() -> AsyncGenerator[StreamingInput, None]:
        nonlocal input_generator_finished
        yield StreamingInput(prompt="What is 2 + 2? The answer is")
        # Generator finishes naturally here
        input_generator_finished = True
    outputs, _ = await collect_outputs(
    # ... excerpt omitted for brevity ...
    assert input_generator_finished, "Input generator should have finished"
    assert len(outputs) > 0, "Should have received outputs"
    assert outputs[-1].finished, "Last output should be marked as finished"
    # Verify the session is cleaned up
    assert not engine.output_processor.has_unfinished_requests(), (
        "Should have no unfinished requests"
    )
    print("Close completion test passed")
```
**EN:** Async test covering `streaming input close signals completion`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, print, collect_outputs, len, output_processor.has_unfinished_requests`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input close signals completion` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, print, collect_outputs, len, output_processor.has_unfinished_requests`。 代码主体包含 4 个显式断言。

### test_streaming_input_abort_queued_inputs (lines 326-385)
```python
async def test_streaming_input_abort_queued_inputs(engine: AsyncLLM):
    """Test that aborting the session cancels queued inputs.

    When multiple inputs are queued and the session is aborted,
    all pending inputs should be cancelled.
    """
    request_id = "test_abort_queued"
    # Use large max_tokens to ensure we have time to queue inputs
    sampling_params = get_sampling_params(max_tokens=2000)
    inputs_sent = 0
    input_cancelled = False
    async def many_inputs_generator() -> AsyncGenerator[StreamingInput, None]:
        nonlocal inputs_sent, input_cancelled
        try:
            # Send several inputs to fill the queue
            for i in range(10):
    # ... excerpt omitted for brevity ...
                yield StreamingInput(prompt=f" Part {i}: Tell me about the number {i}.")
    assert outputs_received >= 10, "Should have received outputs before abort"
    assert input_cancelled or inputs_sent == 10, (
    assert not engine.output_processor.has_unfinished_requests(), (
    )
    print(
        f"Abort queued test passed. Sent {inputs_sent} inputs, "
        f"received {outputs_received} outputs"
```
**EN:** Async test covering `streaming input abort queued inputs`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, engine.generate, print, many_inputs_generator, asyncio.sleep`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input abort queued inputs` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, engine.generate, print, many_inputs_generator, asyncio.sleep`。 代码主体包含 3 个显式断言。

### test_streaming_input_error_propagation (lines 389-416)
```python
async def test_streaming_input_error_propagation(engine: AsyncLLM):
    """Test that errors in the input generator are propagated to the caller."""
    request_id = "test_error_propagation"
    sampling_params = get_sampling_params(max_tokens=20)

    class InputError(Exception):
        pass

    async def error_input_generator() -> AsyncGenerator[StreamingInput, None]:
        yield StreamingInput(prompt="Start with this")
        await asyncio.sleep(0.1)
        raise InputError("Simulated input error")

    # Note: The current implementation catches exceptions and puts them
    # in the queue, so we should get the error when iterating outputs
    with pytest.raises(InputError, match="Simulated input error"):
        async for _ in engine.generate(
            error_input_generator(), sampling_params, request_id
        ):
            pass

    # Give time for cleanup
    await asyncio.sleep(0.3)

    # Verify the session is cleaned up
    assert not engine.output_processor.has_unfinished_requests(), (
        "Should have no unfinished requests after error"
    )
```
**EN:** Async test covering `streaming input error propagation`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, InputError, pytest.raises, engine.generate, asyncio.sleep`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input error propagation` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, InputError, pytest.raises, engine.generate, asyncio.sleep`。 代码主体包含 1 个显式断言。

### test_streaming_input_multiple_concurrent_sessions (lines 420-457)
```python
async def test_streaming_input_multiple_concurrent_sessions(engine: AsyncLLM):
    """Test multiple concurrent streaming input sessions.

    Multiple streaming sessions should be able to run concurrently
    without interfering with each other.
    """
    num_sessions = 3
    results: list[tuple[str, str]] = []
    async def run_session(session_id: int) -> tuple[str, str]:
        request_id = f"test_concurrent_{session_id}"
        sampling_params = get_sampling_params(max_tokens=10)
        prompts = [f"Session {session_id}: Hello", f" world from session {session_id}"]
        async def input_gen() -> AsyncGenerator[StreamingInput, None]:
            for prompt in prompts:
                yield StreamingInput(prompt=prompt)
    # ... excerpt omitted for brevity ...
        return request_id, text
    assert len(results) == num_sessions
    for request_id, text in results:
        assert len(text) > 0, f"Session {request_id} should have generated text"
        print(f"{request_id}: {text}")
    # Verify cleanup
    assert not engine.output_processor.has_unfinished_requests()
```
**EN:** Async test covering `streaming input multiple concurrent sessions`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, asyncio.create_task, asyncio.gather, len, print`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input multiple concurrent sessions` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, asyncio.create_task, asyncio.gather, len, print`。 代码主体包含 3 个显式断言。

### test_streaming_input_per_chunk_sampling_params (lines 461-487)
```python
async def test_streaming_input_per_chunk_sampling_params(engine: AsyncLLM):
    """Test that per-chunk sampling params are respected.

    Each StreamingInput can have its own sampling_params.
    """
    request_id = "test_per_chunk_params"
    base_params = get_sampling_params(max_tokens=10)

    async def variable_params_generator() -> AsyncGenerator[StreamingInput, None]:
        # First chunk with base params
        yield StreamingInput(prompt="Count to five:", sampling_params=base_params)

        # Second chunk with different max_tokens
        chunk_params = get_sampling_params(max_tokens=5)
        yield StreamingInput(
            prompt=" Now count backwards:", sampling_params=chunk_params
        )

    outputs, full_text = await collect_outputs(
        engine.generate(variable_params_generator(), base_params, request_id)
    )

    assert len(outputs) > 0, "Should have received outputs"
    assert outputs[-1].finished, "Last output should be finished"
    assert len(full_text) > 0, "Should have generated text"

    print(f"Per-chunk params test generated: {full_text}")
```
**EN:** Async test covering `streaming input per chunk sampling params`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, print, collect_outputs, len, StreamingInput`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input per chunk 采样 params` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, print, collect_outputs, len, StreamingInput`。 代码主体包含 3 个显式断言。

### test_streaming_input_empty_generator (lines 491-510)
```python
async def test_streaming_input_empty_generator(engine: AsyncLLM):
    """Test behavior when the input generator yields nothing.

    An empty generator should still produce a finished output.
    """
    request_id = "test_empty_generator"
    sampling_params = get_sampling_params(max_tokens=10)

    async def empty_generator() -> AsyncGenerator[StreamingInput, None]:
        # Don't yield anything
        return
        yield  # Make it a generator

    outputs: list[RequestOutput] = []
    async for output in engine.generate(empty_generator(), sampling_params, request_id):
        outputs.append(output)

    # Should still get a finished marker
    assert len(outputs) >= 1, "Should receive at least one output"
    assert outputs[-1].finished, "Should have a finished output"
```
**EN:** Async test covering `streaming input empty generator`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, engine.generate, empty_generator, outputs.append, len`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input empty generator` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, engine.generate, empty_generator, outputs.append, len`。 代码主体包含 2 个显式断言。

### test_streaming_input_single_chunk (lines 514-534)
```python
async def test_streaming_input_single_chunk(engine: AsyncLLM):
    """Test streaming input with a single chunk.

    This is effectively the same as a regular non-streaming request,
    but using the streaming input API.
    """
    request_id = "test_single_chunk"
    sampling_params = get_sampling_params(max_tokens=15)

    async def single_chunk_generator() -> AsyncGenerator[StreamingInput, None]:
        yield StreamingInput(prompt="What color is the sky? The sky is")

    outputs, full_text = await collect_outputs(
        engine.generate(single_chunk_generator(), sampling_params, request_id)
    )

    assert len(outputs) > 0
    assert outputs[-1].finished
    assert "blue" in full_text.lower() or len(full_text) > 0

    print(f"Single chunk test generated: {full_text}")
```
**EN:** Async test covering `streaming input single chunk`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, print, collect_outputs, len, StreamingInput`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input single chunk` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, print, collect_outputs, len, StreamingInput`。 代码主体包含 3 个显式断言。

### test_streaming_input_reuse_request_id (lines 538-563)
```python
async def test_streaming_input_reuse_request_id(engine: AsyncLLM):
    """Test that request IDs can be reused after a session completes."""
    request_id = "test_reuse_id"
    sampling_params = get_sampling_params(max_tokens=5)

    # First session
    async def gen1() -> AsyncGenerator[StreamingInput, None]:
        yield StreamingInput(prompt="First session")

    _, text1 = await collect_outputs(
        engine.generate(gen1(), sampling_params, request_id)
    )

    # Second session with same ID
    async def gen2() -> AsyncGenerator[StreamingInput, None]:
        yield StreamingInput(prompt="Second session")

    _, text2 = await collect_outputs(
        engine.generate(gen2(), sampling_params, request_id)
    )

    assert len(text1) > 0
    assert len(text2) > 0
    assert not engine.output_processor.has_unfinished_requests()

    print(f"Reuse ID test: session 1: {text1}, session 2: {text2}")
```
**EN:** Async test covering `streaming input reuse request id`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, print, collect_outputs, len, output_processor.has_unfinished_requests`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input reuse request id` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, print, collect_outputs, len, output_processor.has_unfinished_requests`。 代码主体包含 3 个显式断言。

### test_streaming_input_validation_errors (lines 567-591)
```python
async def test_streaming_input_validation_errors(engine: AsyncLLM):
    """Test that invalid configurations raise appropriate errors."""

    async def dummy_generator() -> AsyncGenerator[StreamingInput, None]:
        yield StreamingInput(prompt="test")

    # Test n > 1 is rejected
    with pytest.raises(ValueError, match="Input streaming not currently supported"):
        params_n2 = SamplingParams(max_tokens=10, n=2)
        async for _ in engine.generate(dummy_generator(), params_n2, "test_n2"):
            pass

    # Test FINAL_ONLY is rejected
    with pytest.raises(ValueError, match="Input streaming not currently supported"):
        params_final = SamplingParams(
            max_tokens=10, output_kind=RequestOutputKind.FINAL_ONLY
        )
        async for _ in engine.generate(dummy_generator(), params_final, "test_final"):
            pass

    # Test stop strings are rejected
    with pytest.raises(ValueError, match="Input streaming not currently supported"):
        params_stop = SamplingParams(max_tokens=10, stop=["stop"])
        async for _ in engine.generate(dummy_generator(), params_stop, "test_stop"):
            pass
```
**EN:** Async test covering `streaming input validation errors`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, pytest.raises, SamplingParams, engine.generate, StreamingInput, dummy_generator`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `streaming input validation errors` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, pytest.raises, SamplingParams, engine.generate, StreamingInput, dummy_generator`。 主要通过预期异常检查来完成验证。

### test_streaming_input_delayed_generator_exit (lines 595-657)
```python
async def test_streaming_input_delayed_generator_exit(engine: AsyncLLM):
    """Test that output generator exits when input generator closes after outputs.

    This tests the case where:
    1. Multiple inputs are sent and fully processed
    2. The engine has finished
    3. The input generator doesn't exit until after the engine finishes
    4. The output generator should exit properly once the input generator exits
    """
    request_id = "test_delayed_exit"
    sampling_params = get_sampling_params(max_tokens=10)
    engine_finished_event = asyncio.Event()
    input_generator_exited = False
    finish_count = 0
    async def delayed_exit_input_generator() -> AsyncGenerator[StreamingInput, None]:
        nonlocal input_generator_exited
    # ... excerpt omitted for brevity ...
        yield StreamingInput(prompt="Hello, my name is")
        yield StreamingInput(prompt=" Alice")
    assert input_generator_exited, (
    assert len(outputs) > 0, "Should have received outputs"
    assert len(full_text) > 0, "Should have generated text"
    # Verify the session is cleaned up
    assert not engine.output_processor.has_unfinished_requests(), (
        "Should have no unfinished requests"
    )
    print(f"Delayed exit test passed. Generated: {full_text}")
```
**EN:** Async test covering `streaming input delayed generator exit`. Inputs/fixtures: `engine`. It exercises `mark.asyncio, get_sampling_params, asyncio.Event, engine.generate, print, delayed_exit_input_generator`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `streaming input delayed generator exit` 的测试用例。 输入或 fixture：`engine`。 该测试会调用 `mark.asyncio, get_sampling_params, asyncio.Event, engine.generate, print, delayed_exit_input_generator`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排
- **EN:** End-to-end integration rather than isolated units
- **CN:** 端到端集成而非孤立单元

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, pytest_asyncio`.
- **CN:** 外部库：`pytest, pytest_asyncio`。
- **EN:** vLLM modules under test: `vllm, vllm.engine.protocol, vllm.outputs, vllm.platforms, vllm.sampling_params, vllm.utils.torch_utils, vllm.v1.engine.async_llm, vllm.engine.arg_utils`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.engine.protocol, vllm.outputs, vllm.platforms, vllm.sampling_params, vllm.utils.torch_utils, vllm.v1.engine.async_llm, vllm.engine.arg_utils`。
- **EN:** Standard-library support: `asyncio, collections.abc, time`.
- **CN:** 标准库支持：`asyncio, collections.abc, time`。
