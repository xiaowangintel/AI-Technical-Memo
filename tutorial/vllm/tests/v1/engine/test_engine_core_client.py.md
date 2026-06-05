# test_engine_core_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/engine/test_engine_core_client.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `engine core client` behavior and regressions in the v1 stack. / 验证 v1 栈中 `引擎核心 client` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-46)
```python
import asyncio
import importlib
import inspect
import os
import signal
import time
import uuid
from concurrent.futures import Future
from dataclasses import dataclass
from threading import Thread
from types import SimpleNamespace
from typing import Any
from unittest.mock import MagicMock

import pytest
import torch
from transformers import AutoTokenizer
# ... excerpt omitted for brevity ...
    LATE_INTERACTION_MODE_CACHE_QUERY,
    LATE_INTERACTION_MODE_SCORE_DOC,
)
from ...distributed.conftest import MockSubscriber
from ...utils import create_new_process_for_each_test
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, transformers`. vLLM modules under test include `vllm, vllm.distributed.kv_events, vllm.engine.arg_utils, vllm.platforms, vllm.pooling_params, ...`. Local helpers come from `tests.utils, tests.distributed.conftest`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, transformers`。 被测试的 vLLM 模块包括 `vllm, vllm.distributed.kv_events, vllm.engine.arg_utils, vllm.platforms, vllm.pooling_params, ...`。 本地测试辅助逻辑来自 `tests.utils, tests.distributed.conftest`。

### Module state / 模块级状态 (lines 48-60)
```python
if not current_platform.is_cuda_alike():
    pytest.skip(
        reason="V1 currently only supported on CUDA-alike platforms.",
        allow_module_level=True,
    )

MODEL_NAME = "meta-llama/Llama-3.2-1B-Instruct"
TOKENIZER = AutoTokenizer.from_pretrained(MODEL_NAME)
PROMPT = "Hello my name is Robert and I love quantization kernels"
PROMPT_TOKENS = TOKENIZER(PROMPT).input_ids
TEST_MODULE = "tests.v1.engine.test_engine_core_client"

_REQUEST_COUNTER = 0
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MODEL_NAME, TOKENIZER, PROMPT, PROMPT_TOKENS, TEST_MODULE, _REQUEST_COUNTER`. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `AutoTokenizer.from_pretrained, current_platform.is_cuda_alike, pytest.skip, TOKENIZER`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MODEL_NAME, TOKENIZER, PROMPT, PROMPT_TOKENS, TEST_MODULE, _REQUEST_COUNTER`。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `AutoTokenizer.from_pretrained, current_platform.is_cuda_alike, pytest.skip, TOKENIZER`。

### make_request (lines 63-83)
```python
def make_request(
    params: SamplingParams, prompt_tokens_ids: list[int] | None = None
) -> EngineCoreRequest:
    if not prompt_tokens_ids:
        prompt_tokens_ids = PROMPT_TOKENS

    global _REQUEST_COUNTER
    _REQUEST_COUNTER += 1
    request_id = f"request-{_REQUEST_COUNTER}"
    return EngineCoreRequest(
        request_id=request_id,
        external_req_id=f"{request_id}-{uuid.uuid4()}",
        prompt_token_ids=prompt_tokens_ids,
        mm_features=None,
        sampling_params=params,
        pooling_params=None,
        arrival_time=time.time(),
        lora_request=None,
        cache_salt=None,
        data_parallel_rank=None,
    )
```
**EN:** Helper function `make_request` encapsulates reusable logic for `request`. Inputs: `params, prompt_tokens_ids`. Key calls include `EngineCoreRequest, time.time, uuid.uuid4`.
**CN:** 辅助函数 `make_request` 封装了与 `request` 相关的可复用逻辑。 输入参数：`params, prompt_tokens_ids`。 关键调用包括 `EngineCoreRequest, time.time, uuid.uuid4`。

### _reload_envs_module (lines 86-92)
```python
def _reload_envs_module():
    import vllm.envs as envs_mod

    cache_clear = getattr(getattr(envs_mod, "__getattr__", None), "cache_clear", None)
    if cache_clear is not None:
        cache_clear()
    return importlib.reload(envs_mod)
```
**EN:** Helper function `_reload_envs_module` encapsulates reusable logic for `reload envs module`. Key calls include `getattr, importlib.reload, cache_clear`.
**CN:** 辅助函数 `_reload_envs_module` 封装了与 `reload envs module` 相关的可复用逻辑。 关键调用包括 `getattr, importlib.reload, cache_clear`。

### _reload_core_client_module (lines 95-97)
```python
def _reload_core_client_module():
    module = importlib.import_module("vllm.v1.engine.core_client")
    return importlib.reload(module)
```
**EN:** Helper function `_reload_core_client_module` encapsulates reusable logic for `reload core client module`. Key calls include `importlib.import_module, importlib.reload`.
**CN:** 辅助函数 `_reload_core_client_module` 封装了与 `reload core client module` 相关的可复用逻辑。 关键调用包括 `importlib.import_module, importlib.reload`。

### test_mp_client_uses_env_timeout (lines 100-171)
```python
def test_mp_client_uses_env_timeout(monkeypatch: pytest.MonkeyPatch):
    timeout_value = 654
    monkeypatch.setenv("VLLM_ENGINE_READY_TIMEOUT_S", str(timeout_value))

    # Ensure that the environment variable is loaded if caching is enabled
    _reload_envs_module()
    core_client_mod = _reload_core_client_module()
    poll_timeouts: list[int] = []
    class ShadowSocket:
        def poll(self, timeout: int) -> int:
            # Capture the timeout value for each poll call
            poll_timeouts.append(timeout)
            return 1
        def recv_multipart(self):
            return (b"\x00\x00", b"")
    # ... excerpt omitted for brevity ...
                return SimpleNamespace(done=True)
            return (b"", b"")
    )
    try:
        # timeout_value is in seconds, but poll receives milliseconds
        assert poll_timeouts == [timeout_value * 1000]
    finally:
        client.shutdown()
```
**EN:** Test case covering `mp client uses env timeout`. Inputs/fixtures: `monkeypatch`. It exercises `monkeypatch.setenv, _reload_envs_module, _reload_core_client_module, monkeypatch.setattr, SimpleNamespace, core_client_mod.MPClient`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `mp client uses env timeout` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `monkeypatch.setenv, _reload_envs_module, _reload_core_client_module, monkeypatch.setattr, SimpleNamespace, core_client_mod.MPClient`。 代码主体包含 1 个显式断言。

### _make_pooling_request (lines 174-197)
```python
def _make_pooling_request(
    request_id: str, *, mode: str | None = None, query_key: str | None = None
) -> EngineCoreRequest:
    late_interaction_params = None
    if mode is not None and query_key is not None:
        late_interaction_params = LateInteractionParams(
            mode=mode,
            query_key=query_key,
        )

    return EngineCoreRequest(
        request_id=request_id,
        prompt_token_ids=[1, 2, 3],
        mm_features=None,
        sampling_params=None,
        pooling_params=PoolingParams(
            task="token_embed",
            late_interaction_params=late_interaction_params,
        ),
        arrival_time=time.time(),
        lora_request=None,
        cache_salt=None,
        data_parallel_rank=None,
    )
```
**EN:** Helper function `_make_pooling_request` encapsulates reusable logic for `pooling request`. Inputs: `request_id`. Key calls include `EngineCoreRequest, LateInteractionParams, PoolingParams, time.time`.
**CN:** 辅助函数 `_make_pooling_request` 封装了与 `pooling request` 相关的可复用逻辑。 输入参数：`request_id`。 关键调用包括 `EngineCoreRequest, LateInteractionParams, PoolingParams, time.time`。

### test_dplb_late_interaction_sticky_routing (lines 200-221)
```python
def test_dplb_late_interaction_sticky_routing():
    client = object.__new__(DPLBAsyncMPClient)
    client.client_count = 1
    client.reqs_in_flight = {}
    client.core_engines = [b"\x00\x00", b"\x01\x00", b"\x02\x00"]
    client.lb_engines = [[0, 0], [0, 0], [0, 0]]
    client.eng_start_index = 0

    query_key = "rerank-abc-query-0"
    query_request = _make_pooling_request(
        "query-req", mode=LATE_INTERACTION_MODE_CACHE_QUERY, query_key=query_key
    )
    doc_request = _make_pooling_request(
        "doc-req", mode=LATE_INTERACTION_MODE_SCORE_DOC, query_key=query_key
    )

    query_engine = client.get_core_engine_for_request(query_request)
    doc_engine = client.get_core_engine_for_request(doc_request)

    assert query_engine == doc_engine
    assert client.reqs_in_flight["query-req"] == query_engine
    assert client.reqs_in_flight["doc-req"] == doc_engine
```
**EN:** Test case covering `dplb late interaction sticky routing`. It exercises `object.__new__, _make_pooling_request, client.get_core_engine_for_request`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `dplb late interaction sticky routing` 的测试用例。 该测试会调用 `object.__new__, _make_pooling_request, client.get_core_engine_for_request`。 代码主体包含 3 个显式断言。

### test_dplb_non_late_interaction_still_uses_lb (lines 224-236)
```python
def test_dplb_non_late_interaction_still_uses_lb():
    client = object.__new__(DPLBAsyncMPClient)
    client.client_count = 1
    client.reqs_in_flight = {}
    client.core_engines = [b"\x00\x00", b"\x01\x00", b"\x02\x00"]
    client.lb_engines = [[2, 1], [0, 0], [1, 0]]
    client.eng_start_index = 0

    request = make_request(SamplingParams(max_tokens=1))
    chosen_engine = client.get_core_engine_for_request(request)

    assert chosen_engine == client.core_engines[1]
    assert client.lb_engines[1][0] == 1
```
**EN:** Test case covering `dplb non late interaction still uses lb`. It exercises `object.__new__, make_request, client.get_core_engine_for_request, SamplingParams`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `dplb non late interaction still uses lb` 的测试用例。 该测试会调用 `object.__new__, make_request, client.get_core_engine_for_request, SamplingParams`。 代码主体包含 2 个显式断言。

### loop_until_done (lines 239-253)
```python
def loop_until_done(client: EngineCoreClient, outputs: dict):
    while True:
        engine_core_outputs = client.get_output().outputs

        if len(engine_core_outputs) == 0:
            continue

        all_finished = True
        for out in engine_core_outputs:
            outputs[out.request_id].append(out)
            if not out.finished:
                all_finished = False

        if all_finished:
            break
```
**EN:** Helper function `loop_until_done` encapsulates reusable logic for `loop until done`. Inputs: `client, outputs`. Key calls include `client.get_output, len, outputs.append`.
**CN:** 辅助函数 `loop_until_done` 封装了与 `loop until done` 相关的可复用逻辑。 输入参数：`client, outputs`。 关键调用包括 `client.get_output, len, outputs.append`。

### loop_until_done_async (lines 256-270)
```python
async def loop_until_done_async(client: EngineCoreClient, outputs: dict):
    while True:
        engine_core_outputs = (await client.get_output_async()).outputs

        if len(engine_core_outputs) == 0:
            continue

        all_finished = True
        for out in engine_core_outputs:
            outputs[out.request_id].append(out)
            if not out.finished:
                all_finished = False

        if all_finished:
            break
```
**EN:** Helper function `loop_until_done_async` encapsulates reusable logic for `loop until done async`. Inputs: `client, outputs`. Key calls include `len, outputs.append, client.get_output_async`.
**CN:** 辅助函数 `loop_until_done_async` 封装了与 `loop until done async` 相关的可复用逻辑。 输入参数：`client, outputs`。 关键调用包括 `len, outputs.append, client.get_output_async`。

### loop_until_fully_done_async (lines 273-288)
```python
async def loop_until_fully_done_async(client: EngineCoreClient, outputs: dict):
    while True:
        engine_core_outputs = (await client.get_output_async()).outputs

        if len(engine_core_outputs) == 0:
            continue

        # Add outputs to the dict
        for out in engine_core_outputs:
            outputs[out.request_id].append(out)

        # Check if all request IDs in outputs have finished
        if all(outs and outs[-1].finished for outs in outputs.values()):
            break

        await asyncio.sleep(0.1)
```
**EN:** Helper function `loop_until_fully_done_async` encapsulates reusable logic for `loop until fully done async`. Inputs: `client, outputs`. Key calls include `all, len, outputs.append, asyncio.sleep, client.get_output_async, outputs.values`.
**CN:** 辅助函数 `loop_until_fully_done_async` 封装了与 `loop until fully done async` 相关的可复用逻辑。 输入参数：`client, outputs`。 关键调用包括 `all, len, outputs.append, asyncio.sleep, client.get_output_async, outputs.values`。

### echo (lines 292-298)
```python
def echo(self, msg: str, err_msg: str | None = None, sleep: float | None = None) -> str:
    print(f"echo util function called: {msg}, {err_msg}")
    if sleep is not None:
        time.sleep(sleep)
    if err_msg is not None:
        raise ValueError(err_msg)
    return msg
```
**EN:** Helper function `echo` encapsulates reusable logic for `echo`. Inputs: `msg, err_msg, sleep`. Key calls include `print, time.sleep, ValueError`.
**CN:** 辅助函数 `echo` 封装了与 `echo` 相关的可复用逻辑。 输入参数：`msg, err_msg, sleep`。 关键调用包括 `print, time.sleep, ValueError`。

### TestMessage (lines 302-305)
```python
class TestMessage:
    """Test dataclass for verifying custom type serialization."""

    message: str
```
**EN:** Class `TestMessage` groups 0 test method(s).
**CN:** 类 `TestMessage` 组织了 0 个测试方法。

### echo_dc (lines 309-318)
```python
def echo_dc(
    self,
    msg: str,
    return_list: bool = False,
) -> TestMessage | list[TestMessage]:
    print(f"echo dc util function called: {msg}")
    val = None if msg is None else TestMessage(msg)
    # Return dataclass to verify support for returning custom types
    # (for which there is special handling to make it work with msgspec).
    return [val for _ in range(3)] if return_list else val
```
**EN:** Helper function `echo_dc` encapsulates reusable logic for `echo dc`. Inputs: `msg, return_list`. Key calls include `print, TestMessage, range`.
**CN:** 辅助函数 `echo_dc` 封装了与 `echo dc` 相关的可复用逻辑。 输入参数：`msg, return_list`。 关键调用包括 `print, TestMessage, range`。

### echo_dc_dict (lines 322-334)
```python
def echo_dc_dict(
    self,
    msg: str,
    return_dict: bool = False,
) -> TestMessage | dict[str, TestMessage]:
    print(f"echo dc dict util function called: {msg}")
    val = None if msg is None else TestMessage(msg)
    # Return dict of dataclasses to verify support for returning dicts
    # with custom value types.
    if return_dict:
        return {"key1": val, "key2": val, "key3": val}
    else:
        return val
```
**EN:** Helper function `echo_dc_dict` encapsulates reusable logic for `echo dc dict`. Inputs: `msg, return_dict`. Key calls include `print, TestMessage`.
**CN:** 辅助函数 `echo_dc_dict` 封装了与 `echo dc dict` 相关的可复用逻辑。 输入参数：`msg, return_dict`。 关键调用包括 `print, TestMessage`。

### echo_dc_nested (lines 338-351)
```python
def echo_dc_nested(
    self,
    msg: str,
    structure_type: str = "list_of_dicts",
) -> Any:
    print(f"echo dc nested util function called: {msg}, structure: {structure_type}")
    val = None if msg is None else TestMessage(msg)

    structures = {
        "list_of_dicts": [{"a": val, "b": val}, {"c": val, "d": val}],
        "dict_of_lists": {"list1": [val, val], "list2": [val, val]},
        "deep_nested": {"outer": [{"inner": [val, val]}, {"inner": [val]}]},
    }
    return structures.get(structure_type, val)
```
**EN:** Helper function `echo_dc_nested` encapsulates reusable logic for `echo dc nested`. Inputs: `msg, structure_type`. Key calls include `print, structures.get, TestMessage`.
**CN:** 辅助函数 `echo_dc_nested` 封装了与 `echo dc nested` 相关的可复用逻辑。 输入参数：`msg, structure_type`。 关键调用包括 `print, structures.get, TestMessage`。

### future_echo (lines 354-364)
```python
def future_echo(self, value: Any, num_wait_loops: int = 2) -> Future:
    """Utility that returns a Future completed once the engine is idle
    (tests deferred utility path).
    """
    future: Future = Future()

    def idle(engine: EngineCore):
        future.set_result(value)

    self._idle_state_callbacks.append(idle)
    return future
```
**EN:** Helper function `future_echo` encapsulates reusable logic for `future echo`. Inputs: `value, num_wait_loops`. Key calls include `Future, _idle_state_callbacks.append, future.set_result`.
**CN:** 辅助函数 `future_echo` 封装了与 `future echo` 相关的可复用逻辑。 输入参数：`value, num_wait_loops`。 关键调用包括 `Future, _idle_state_callbacks.append, future.set_result`。

### subprocess_echo_patch (lines 375-396)
```python
def subprocess_echo_patch(monkeypatch, tmp_path):
    """Create sitecustomize.py so spawned subprocesses have echo method.

    This is needed because ROCm uses 'spawn' multiprocessing start method,
    which creates a fresh Python interpreter that doesn't inherit monkey-patches.
    By using sitecustomize.py, we ensure the patch is applied when Python starts.
    """
    sc = tmp_path / "sitecustomize.py"
    sc.write_text(
        "\n".join(
            [
                "import time",
                "from vllm.v1.engine.core import EngineCore",
                inspect.getsource(echo),
                "EngineCore.echo = echo",
            ]
        )
    )
    monkeypatch.setenv(
        "PYTHONPATH",
        os.pathsep.join(filter(None, [str(tmp_path), os.getenv("PYTHONPATH")])),
    )
```
**EN:** Fixture/helper `subprocess_echo_patch` prepares reusable state for downstream tests. Inputs: `monkeypatch, tmp_path`. Key calls include `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`.
**CN:** `subprocess_echo_patch` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`monkeypatch, tmp_path`。 关键调用包括 `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`。

### subprocess_echo_dc_patch (lines 400-420)
```python
def subprocess_echo_dc_patch(monkeypatch, tmp_path):
    """Create sitecustomize.py so spawned subprocesses have echo_dc method."""
    sc = tmp_path / "sitecustomize.py"
    sc.write_text(
        "\n".join(
            [
                "from dataclasses import dataclass",
                "",
                inspect.getsource(TestMessage),
                f"TestMessage.__module__ = '{TEST_MODULE}'",
                "",
                "from vllm.v1.engine.core import EngineCore",
                inspect.getsource(echo_dc),
                "EngineCore.echo_dc = echo_dc",
            ]
        )
    )
    monkeypatch.setenv(
        "PYTHONPATH",
        os.pathsep.join(filter(None, [str(tmp_path), os.getenv("PYTHONPATH")])),
    )
```
**EN:** Fixture/helper `subprocess_echo_dc_patch` prepares reusable state for downstream tests. Inputs: `monkeypatch, tmp_path`. Key calls include `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`.
**CN:** `subprocess_echo_dc_patch` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`monkeypatch, tmp_path`。 关键调用包括 `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`。

### subprocess_echo_dc_dict_patch (lines 424-444)
```python
def subprocess_echo_dc_dict_patch(monkeypatch, tmp_path):
    """Create sitecustomize.py so spawned subprocesses have echo_dc_dict method."""
    sc = tmp_path / "sitecustomize.py"
    sc.write_text(
        "\n".join(
            [
                "from dataclasses import dataclass",
                "",
                inspect.getsource(TestMessage),
                f"TestMessage.__module__ = '{TEST_MODULE}'",
                "",
                "from vllm.v1.engine.core import EngineCore",
                inspect.getsource(echo_dc_dict),
                "EngineCore.echo_dc_dict = echo_dc_dict",
            ]
        )
    )
    monkeypatch.setenv(
        "PYTHONPATH",
        os.pathsep.join(filter(None, [str(tmp_path), os.getenv("PYTHONPATH")])),
    )
```
**EN:** Fixture/helper `subprocess_echo_dc_dict_patch` prepares reusable state for downstream tests. Inputs: `monkeypatch, tmp_path`. Key calls include `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`.
**CN:** `subprocess_echo_dc_dict_patch` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`monkeypatch, tmp_path`。 关键调用包括 `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`。

### subprocess_echo_dc_nested_patch (lines 448-469)
```python
def subprocess_echo_dc_nested_patch(monkeypatch, tmp_path):
    """Create sitecustomize.py so spawned subprocesses have echo_dc_nested method."""
    sc = tmp_path / "sitecustomize.py"
    sc.write_text(
        "\n".join(
            [
                "from dataclasses import dataclass",
                "from typing import Any",
                "",
                inspect.getsource(TestMessage),
                f"TestMessage.__module__ = '{TEST_MODULE}'",
                "",
                "from vllm.v1.engine.core import EngineCore",
                inspect.getsource(echo_dc_nested),
                "EngineCore.echo_dc_nested = echo_dc_nested",
            ]
        )
    )
    monkeypatch.setenv(
        "PYTHONPATH",
        os.pathsep.join(filter(None, [str(tmp_path), os.getenv("PYTHONPATH")])),
    )
```
**EN:** Fixture/helper `subprocess_echo_dc_nested_patch` prepares reusable state for downstream tests. Inputs: `monkeypatch, tmp_path`. Key calls include `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`.
**CN:** `subprocess_echo_dc_nested_patch` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`monkeypatch, tmp_path`。 关键调用包括 `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`。

### subprocess_future_echo_patch (lines 473-491)
```python
def subprocess_future_echo_patch(monkeypatch, tmp_path):
    """Create sitecustomize.py so spawned subprocesses have future_echo method."""
    sc = tmp_path / "sitecustomize.py"
    sc.write_text(
        "\n".join(
            [
                "from concurrent.futures import Future",
                "from typing import Any",
                "",
                "from vllm.v1.engine.core import EngineCore",
                inspect.getsource(future_echo),
                "EngineCore.future_echo = future_echo",
            ]
        )
    )
    monkeypatch.setenv(
        "PYTHONPATH",
        os.pathsep.join(filter(None, [str(tmp_path), os.getenv("PYTHONPATH")])),
    )
```
**EN:** Fixture/helper `subprocess_future_echo_patch` prepares reusable state for downstream tests. Inputs: `monkeypatch, tmp_path`. Key calls include `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`.
**CN:** `subprocess_future_echo_patch` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`monkeypatch, tmp_path`。 关键调用包括 `sc.write_text, monkeypatch.setenv, join, pathsep.join, filter, inspect.getsource`。

### test_engine_core_client (lines 496-582)
```python
def test_engine_core_client(
    monkeypatch: pytest.MonkeyPatch,
    multiprocessing_mode: bool,
    subprocess_echo_patch,
):
    with monkeypatch.context() as m:
        # Monkey-patch core engine utility function to test.
        m.setattr(EngineCore, "echo", echo, raising=False)

        engine_args = EngineArgs(model=MODEL_NAME, enforce_eager=True)
        vllm_config = engine_args.create_engine_config(UsageContext.UNKNOWN_CONTEXT)
        executor_class = Executor.get_class(vllm_config)
        with set_default_torch_num_threads(1):
            client = EngineCoreClient.make_client(
                multiprocess_mode=multiprocessing_mode,
                asyncio_mode=False,
                vllm_config=vllm_config,
    # ... excerpt omitted for brevity ...
            assert len(outputs[req_id]) == MAX_TOKENS, (
                assert len(outputs[req_id]) < MAX_TOKENS, (
                assert len(outputs[req_id]) == MAX_TOKENS, (
            assert result == "testarg"
            with pytest.raises(Exception) as e_info:
                core_client.call_utility("echo", None, "help!")
            assert str(e_info.value) == "Call to echo method failed: help!"
```
**EN:** Parameterized test covering `engine core client`. Parameter axes: `multiprocessing_mode`. Inputs/fixtures: `monkeypatch, multiprocessing_mode, subprocess_echo_patch`. It exercises `create_new_process_for_each_test, mark.parametrize, monkeypatch.context, m.setattr, EngineArgs, engine_args.create_engine_config`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `引擎核心 client` 的测试用例。 参数维度：`multiprocessing_mode`。 输入或 fixture：`monkeypatch, multiprocessing_mode, subprocess_echo_patch`。 该测试会调用 `create_new_process_for_each_test, mark.parametrize, monkeypatch.context, m.setattr, EngineArgs, engine_args.create_engine_config`。 代码主体包含 5 个显式断言。

### test_engine_core_client_asyncio (lines 586-679)
```python
async def test_engine_core_client_asyncio(
    monkeypatch: pytest.MonkeyPatch,
    subprocess_echo_patch,
):
    with monkeypatch.context() as m:
        # Monkey-patch core engine utility function to test.
        m.setattr(EngineCore, "echo", echo, raising=False)

        engine_args = EngineArgs(model=MODEL_NAME, enforce_eager=True)
        vllm_config = engine_args.create_engine_config(
            usage_context=UsageContext.UNKNOWN_CONTEXT
        )
        executor_class = Executor.get_class(vllm_config)
        with set_default_torch_num_threads(1):
            client = EngineCoreClient.make_client(
                multiprocess_mode=True,
                asyncio_mode=True,
    # ... excerpt omitted for brevity ...
                assert len(outputs[req_id]) == MAX_TOKENS, (
                    assert len(outputs[req_id]) < MAX_TOKENS, (
                    assert len(outputs[req_id]) == MAX_TOKENS, (
            assert result == "testarg"
            with pytest.raises(Exception) as e_info:
            assert str(e_info.value) == "Call to echo method failed: help!"
            result = await asyncio.wait_for(
                core_client.call_utility_async("echo", "testarg3"), timeout=1.0
            )
            assert result == "testarg3"
        finally:
            client.shutdown()
```
**EN:** Async test covering `engine core client asyncio`. Inputs/fixtures: `monkeypatch, subprocess_echo_patch`. It exercises `mark.asyncio, monkeypatch.context, m.setattr, EngineArgs, engine_args.create_engine_config, Executor.get_class`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `引擎核心 client asyncio` 的测试用例。 输入或 fixture：`monkeypatch, subprocess_echo_patch`。 该测试会调用 `mark.asyncio, monkeypatch.context, m.setattr, EngineArgs, engine_args.create_engine_config, Executor.get_class`。 代码主体包含 7 个显式断言。

### test_engine_core_client_util_method_custom_return (lines 683-727)
```python
async def test_engine_core_client_util_method_custom_return(
    monkeypatch: pytest.MonkeyPatch,
    subprocess_echo_dc_patch,
):
    with monkeypatch.context() as m:
        # Must set insecure serialization to allow returning custom types.
        m.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")

        # Monkey-patch core engine utility function to test.
        m.setattr(EngineCore, "echo_dc", echo_dc, raising=False)
        engine_args = EngineArgs(model=MODEL_NAME, enforce_eager=True)
        vllm_config = engine_args.create_engine_config(
            usage_context=UsageContext.UNKNOWN_CONTEXT
        )
        executor_class = Executor.get_class(vllm_config)
        with set_default_torch_num_threads(1):
    # ... excerpt omitted for brevity ...
            assert isinstance(result, TestMessage) and result.message == "testarg2"
            assert isinstance(result, list) and all(
            assert result is None
            result = await core_client.call_utility_async("echo_dc", None, True)
            assert isinstance(result, list) and all(r is None for r in result)
        finally:
            client.shutdown()
```
**EN:** Async test covering `engine core client util method custom return`. Inputs/fixtures: `monkeypatch, subprocess_echo_dc_patch`. It exercises `mark.asyncio, monkeypatch.context, m.setenv, m.setattr, EngineArgs, engine_args.create_engine_config`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `引擎核心 client util method custom return` 的测试用例。 输入或 fixture：`monkeypatch, subprocess_echo_dc_patch`。 该测试会调用 `mark.asyncio, monkeypatch.context, m.setenv, m.setattr, EngineArgs, engine_args.create_engine_config`。 代码主体包含 4 个显式断言。

### test_engine_core_client_util_method_custom_dict_return (lines 731-784)
```python
async def test_engine_core_client_util_method_custom_dict_return(
    monkeypatch: pytest.MonkeyPatch,
    subprocess_echo_dc_dict_patch,
):
    with monkeypatch.context() as m:
        # Must set insecure serialization to allow returning custom types.
        m.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")

        # Monkey-patch core engine utility function to test.
        m.setattr(EngineCore, "echo_dc_dict", echo_dc_dict, raising=False)
        engine_args = EngineArgs(model=MODEL_NAME, enforce_eager=True)
        vllm_config = engine_args.create_engine_config(
            usage_context=UsageContext.UNKNOWN_CONTEXT
        )
        executor_class = Executor.get_class(vllm_config)
        with set_default_torch_num_threads(1):
    # ... excerpt omitted for brevity ...
            assert isinstance(result, TestMessage) and result.message == "testarg3"
            assert isinstance(result, dict) and len(result) == 3
                assert key in ["key1", "key2", "key3"]
                assert isinstance(val, TestMessage) and val.message == "testarg3"
            for key, val in result.items():
                assert val is None
        finally:
            client.shutdown()
```
**EN:** Async test covering `engine core client util method custom dict return`. Inputs/fixtures: `monkeypatch, subprocess_echo_dc_dict_patch`. It exercises `mark.asyncio, monkeypatch.context, m.setenv, m.setattr, EngineArgs, engine_args.create_engine_config`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `引擎核心 client util method custom dict return` 的测试用例。 输入或 fixture：`monkeypatch, subprocess_echo_dc_dict_patch`。 该测试会调用 `mark.asyncio, monkeypatch.context, m.setenv, m.setattr, EngineArgs, engine_args.create_engine_config`。 代码主体包含 7 个显式断言。

### test_engine_core_client_util_method_nested_structures (lines 788-894)
```python
async def test_engine_core_client_util_method_nested_structures(
    monkeypatch: pytest.MonkeyPatch,
    subprocess_echo_dc_nested_patch,
):
    with monkeypatch.context() as m:
        # Must set insecure serialization to allow returning custom types.
        m.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")

        # Monkey-patch core engine utility function to test.
        m.setattr(EngineCore, "echo_dc_nested", echo_dc_nested, raising=False)
        engine_args = EngineArgs(model=MODEL_NAME, enforce_eager=True)
        vllm_config = engine_args.create_engine_config(
            usage_context=UsageContext.UNKNOWN_CONTEXT
        )
        executor_class = Executor.get_class(vllm_config)
        with set_default_torch_num_threads(1):
    # ... excerpt omitted for brevity ...
            assert isinstance(result, list) and len(result) == 2
                assert isinstance(item, dict)
                    assert "a" in item and "b" in item
                    assert (
                    assert "c" in item and "d" in item
                for val in item.values():
                    assert val is None
        finally:
            client.shutdown()
```
**EN:** Async test covering `engine core client util method nested structures`. Inputs/fixtures: `monkeypatch, subprocess_echo_dc_nested_patch`. It exercises `mark.asyncio, monkeypatch.context, m.setenv, m.setattr, EngineArgs, engine_args.create_engine_config`. The body contains 23 explicit assertion(s).
**CN:** 该代码块是覆盖 `引擎核心 client util method nested structures` 的测试用例。 输入或 fixture：`monkeypatch, subprocess_echo_dc_nested_patch`。 该测试会调用 `mark.asyncio, monkeypatch.context, m.setenv, m.setattr, EngineArgs, engine_args.create_engine_config`。 代码主体包含 23 个显式断言。

### test_engine_core_client_future_utility_async (lines 898-936)
```python
async def test_engine_core_client_future_utility_async(
    monkeypatch: pytest.MonkeyPatch,
    subprocess_future_echo_patch,
):
    """Test that a utility returning a Future completes when the future is done
    (engine uses add_done_callback).
    """
    with monkeypatch.context() as m:
        m.setattr(EngineCore, "future_echo", future_echo, raising=False)

        engine_args = EngineArgs(model=MODEL_NAME, enforce_eager=True)
        vllm_config = engine_args.create_engine_config(
            usage_context=UsageContext.UNKNOWN_CONTEXT
        )
        executor_class = Executor.get_class(vllm_config)
        with set_default_torch_num_threads(1):
            client = EngineCoreClient.make_client(
    # ... excerpt omitted for brevity ...
            assert result == "future_result"
            # None is a valid result (num_wait_loops=0 → completes on first step)
            result = await core_client.call_utility_async("future_echo", None, 0)
            assert result is None
        finally:
            client.shutdown()
```
**EN:** Async test covering `engine core client future utility async`. Inputs/fixtures: `monkeypatch, subprocess_future_echo_patch`. It exercises `mark.asyncio, monkeypatch.context, m.setattr, EngineArgs, engine_args.create_engine_config, Executor.get_class`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `引擎核心 client future utility async` 的测试用例。 输入或 fixture：`monkeypatch, subprocess_future_echo_patch`。 该测试会调用 `mark.asyncio, monkeypatch.context, m.setattr, EngineArgs, engine_args.create_engine_config, Executor.get_class`。 代码主体包含 2 个显式断言。

### test_kv_cache_events (lines 951-1022)
```python
def test_kv_cache_events(
    multiprocessing_mode: bool,
    publisher_config,
    model_name: str,
    num_groups: int,
):
    block_size = 16
    num_blocks = 2

    engine_args = EngineArgs(
        model=model_name,
        enforce_eager=True,
        enable_prefix_caching=True,
        block_size=block_size,
    )
    engine_args.kv_events_config = publisher_config
    vllm_config = engine_args.create_engine_config(UsageContext.UNKNOWN_CONTEXT)
    # ... excerpt omitted for brevity ...
        assert result is not None, "No message received"
        assert seq == 0, "Sequence number mismatch"
        assert len(received.events) == num_groups, (
            assert isinstance(event, BlockStored), "We should have a BlockStored event"
            assert len(event.block_hashes) == num_blocks, (
            assert event.block_size == block_size, (
                "Token ids should be the same as the custom tokens"
            )
            assert event.group_idx == index
    finally:
        client.shutdown()
        subscriber.close()
```
**EN:** Parameterized test covering `KV cache events`. Parameter axes: `model_name, num_groups, multiprocessing_mode, publisher_config`. Inputs/fixtures: `multiprocessing_mode, publisher_config, model_name, num_groups`. It exercises `mark.parametrize, EngineArgs, engine_args.create_engine_config, Executor.get_class, endpoint.replace, MockSubscriber`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 缓存 events` 的测试用例。 参数维度：`model_name, num_groups, multiprocessing_mode, publisher_config`。 输入或 fixture：`multiprocessing_mode, publisher_config, model_name, num_groups`。 该测试会调用 `mark.parametrize, EngineArgs, engine_args.create_engine_config, Executor.get_class, endpoint.replace, MockSubscriber`。 代码主体包含 12 个显式断言。

### test_kv_cache_events_dp (lines 1032-1117)
```python
async def test_kv_cache_events_dp(
    multiprocessing_mode: bool,
    publisher_config,
):
    block_size = 16
    num_blocks = 2
    dp_size = 2
    tp_size = 2

    engine_args = EngineArgs(
        model=MODEL_NAME,
        enforce_eager=True,
        enable_prefix_caching=True,
        data_parallel_size=dp_size,
        tensor_parallel_size=tp_size,
        block_size=block_size,
    )
    engine_args.kv_events_config = publisher_config
    # ... excerpt omitted for brevity ...
        assert len(unique_dps) == 2, (
            f"Expected 2 unique data_parallel_ranks, got {len(unique_dps)}"
        )
    finally:
        client.shutdown()
        subscriber.close()
```
**EN:** Parameterized test covering `KV cache events dp`. Parameter axes: `multiprocessing_mode, publisher_config`. Inputs/fixtures: `multiprocessing_mode, publisher_config`. It exercises `mark.parametrize, multi_gpu_test, EngineArgs, engine_args.create_engine_config, Executor.get_class, endpoint.replace`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 缓存 events dp` 的测试用例。 参数维度：`multiprocessing_mode, publisher_config`。 输入或 fixture：`multiprocessing_mode, publisher_config`。 该测试会调用 `mark.parametrize, multi_gpu_test, EngineArgs, engine_args.create_engine_config, Executor.get_class, endpoint.replace`。 代码主体包含 1 个显式断言。

### test_startup_failure (lines 1121-1161)
```python
def test_startup_failure(monkeypatch: pytest.MonkeyPatch):
    with monkeypatch.context() as m, pytest.raises(Exception) as e_info:
        # Monkey-patch to extract core process pid while it's starting.
        core_proc_pid = [None]
        cepm_ctor = CoreEngineProcManager.__init__

        def patched_cepm_ctor(self: CoreEngineProcManager, *args, **kwargs):
            cepm_ctor(self, *args, **kwargs)
            core_proc_pid[0] = self.processes[0].pid
        m.setattr(CoreEngineProcManager, "__init__", patched_cepm_ctor)
        t = time.time()
        engine_args = EngineArgs(model=MODEL_NAME)
        vllm_config = engine_args.create_engine_config(
            usage_context=UsageContext.UNKNOWN_CONTEXT
        )
        executor_class = Executor.get_class(vllm_config)
    # ... excerpt omitted for brevity ...
            assert isinstance(child_pid, int)
            vllm_config=vllm_config,
            executor_class=executor_class,
            log_stats=True,
    assert "Engine core initialization failed" in str(e_info.value)
```
**EN:** Test case covering `startup failure`. Inputs/fixtures: `monkeypatch`. It exercises `mark.timeout, monkeypatch.context, pytest.raises, m.setattr, time.time, EngineArgs`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `startup failure` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `mark.timeout, monkeypatch.context, pytest.raises, m.setattr, time.time, EngineArgs`。 代码主体包含 2 个显式断言。

### test_engine_core_proc_instantiation_cuda_empty (lines 1165-1226)
```python
def test_engine_core_proc_instantiation_cuda_empty(monkeypatch: pytest.MonkeyPatch):
    """
    Test that EngineCoreProc can be instantiated when CUDA_VISIBLE_DEVICES
    is empty. This ensures the engine frontend does not need access to GPUs.

    from vllm.v1.engine.core import EngineCoreProc
    from vllm.v1.executor.abstract import Executor
    # Create a simple mock executor instead of a complex custom class
    mock_executor_class = MagicMock(spec=Executor)
    def create_mock_executor(vllm_config):
        mock_executor = MagicMock()
        # Only implement the methods that are actually called during init
        from vllm.v1.kv_cache_interface import FullAttentionSpec
    # ... excerpt omitted for brevity ...
        return mock_executor
            return EngineZmqAddresses(
            executor_class=mock_executor_class,
            log_stats=False,
            engine_index=0,
        )
        engine_core_proc.shutdown()
```
**EN:** Test case covering `engine core proc instantiation cuda empty`. Inputs/fixtures: `monkeypatch`. It exercises `create_new_process_for_each_test, MagicMock, FullAttentionSpec, monkeypatch.context, m.setenv, m.setattr`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `引擎核心 proc instantiation cuda empty` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `create_new_process_for_each_test, MagicMock, FullAttentionSpec, monkeypatch.context, m.setenv, m.setattr`。 主要通过 mock、回调或输出检查来完成验证。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, transformers`.
- **CN:** 外部库：`pytest, torch, transformers`。
- **EN:** vLLM modules under test: `vllm, vllm.distributed.kv_events, vllm.engine.arg_utils, vllm.platforms, vllm.pooling_params, vllm.usage.usage_lib, vllm.utils.torch_utils, vllm.v1.engine, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.distributed.kv_events, vllm.engine.arg_utils, vllm.platforms, vllm.pooling_params, vllm.usage.usage_lib, vllm.utils.torch_utils, vllm.v1.engine, ...`。
- **EN:** Local test helpers: `tests.utils, tests.distributed.conftest`.
- **CN:** 本地测试辅助模块：`tests.utils, tests.distributed.conftest`。
- **EN:** Standard-library support: `asyncio, importlib, inspect, os, signal, time, uuid, concurrent.futures, ...`.
- **CN:** 标准库支持：`asyncio, importlib, inspect, os, signal, time, uuid, concurrent.futures, ...`。
