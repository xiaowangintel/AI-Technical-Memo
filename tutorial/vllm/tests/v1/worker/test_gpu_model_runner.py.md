# test_gpu_model_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/worker/test_gpu_model_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `GPU model runner` behavior and regressions in the v1 stack. / 验证 v1 栈中 `GPU 模型执行器` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-47)
```python
from types import SimpleNamespace
from unittest.mock import Mock

import numpy as np
import pytest
import torch
import vllm.v1.worker.gpu_model_runner as gpu_model_runner_module
from vllm.config import (
    AttentionConfig,
    CacheConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.distributed.parallel_state import (
# ... excerpt omitted for brevity ...
from vllm.v1.sample.metadata import SamplingMetadata
from vllm.v1.spec_decode.metadata import SpecDecodeMetadata
from vllm.v1.worker.gpu_input_batch import InputBatch
from vllm.v1.worker.gpu_model_runner import GPUModelRunner
from vllm.v1.worker.utils import AttentionGroup, select_common_block_size
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch`. vLLM modules under test include `vllm.v1.worker.gpu_model_runner, vllm.config, vllm.distributed.parallel_state, vllm.model_executor.layers.attention, vllm.model_executor.layers.mamba.mamba_mixer2, ...`. Local helpers come from `tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch`。 被测试的 vLLM 模块包括 `vllm.v1.worker.gpu_model_runner, vllm.config, vllm.distributed.parallel_state, vllm.model_executor.layers.attention, vllm.model_executor.layers.mamba.mamba_mixer2, ...`。 本地测试辅助逻辑来自 `tests.utils`。

### Module state / 模块级状态 (lines 49-51)
```python
BLOCK_SIZE = 16
NUM_BLOCKS = 10
DEVICE_TYPE = current_platform.device_type
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `BLOCK_SIZE, NUM_BLOCKS, DEVICE_TYPE`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`BLOCK_SIZE, NUM_BLOCKS, DEVICE_TYPE`。

### initialize_kv_cache (lines 54-87)
```python
def initialize_kv_cache(runner: GPUModelRunner):
    """
    Only perform necessary steps in GPUModelRunner.initialize_kv_cache()
    attn_spec = FullAttentionSpec(
        block_size=BLOCK_SIZE,
        num_kv_heads=runner.model_config.get_num_kv_heads(runner.parallel_config),
        head_size=runner.model_config.get_head_size(),
        dtype=runner.kv_cache_dtype,
    )
    tensor_size = attn_spec.page_size_bytes * NUM_BLOCKS
    kv_cache_config = KVCacheConfig(
        num_blocks=NUM_BLOCKS,
        kv_cache_tensors=[
            KVCacheTensor(size=tensor_size, shared_by=["layer.0"]),
        ],
        kv_cache_groups=[
            KVCacheGroupSpec(layer_names=["layer.0"], kv_cache_spec=attn_spec)
    # ... excerpt omitted for brevity ...
        block_sizes=[kv_cache_config.kv_cache_groups[0].kv_cache_spec.block_size],
        kernel_block_sizes=[
            kv_cache_config.kv_cache_groups[0].kv_cache_spec.block_size
    runner.initialize_attn_backend(kv_cache_config)
```
**EN:** Helper function `initialize_kv_cache` encapsulates reusable logic for `initialize KV cache`. Inputs: `runner`. Key calls include `FullAttentionSpec, KVCacheConfig, InputBatch, runner.initialize_attn_backend, model_config.get_num_kv_heads, model_config.get_head_size`.
**CN:** 辅助函数 `initialize_kv_cache` 封装了与 `initialize KV 缓存` 相关的可复用逻辑。 输入参数：`runner`。 关键调用包括 `FullAttentionSpec, KVCacheConfig, InputBatch, runner.initialize_attn_backend, model_config.get_num_kv_heads, model_config.get_head_size`。

### get_vllm_config (lines 90-114)
```python
def get_vllm_config():
    model_config = ModelConfig(
        model="facebook/opt-125m",
        dtype="float16",
        seed=42,
    )
    scheduler_config = SchedulerConfig(
        max_num_seqs=10,
        max_num_batched_tokens=512,
        max_model_len=512,
        is_encoder_decoder=model_config.is_encoder_decoder,
    )
    cache_config = CacheConfig(
        block_size=BLOCK_SIZE,
        gpu_memory_utilization=0.9,
        cache_dtype="auto",
    )
    parallel_config = ParallelConfig()
    vllm_config = VllmConfig(
        model_config=model_config,
        cache_config=cache_config,
        scheduler_config=scheduler_config,
        parallel_config=parallel_config,
    )
    return vllm_config
```
**EN:** Helper function `get_vllm_config` encapsulates reusable logic for `vllm config`. Key calls include `ModelConfig, SchedulerConfig, CacheConfig, ParallelConfig, VllmConfig`.
**CN:** 辅助函数 `get_vllm_config` 封装了与 `vllm config` 相关的可复用逻辑。 关键调用包括 `ModelConfig, SchedulerConfig, CacheConfig, ParallelConfig, VllmConfig`。

### model_runner (lines 118-129)
```python
def model_runner():
    vllm_config = get_vllm_config()
    with set_current_vllm_config(vllm_config):
        model_config = vllm_config.model_config
        num_heads = model_config.get_num_kv_heads(vllm_config.parallel_config)
        head_size = model_config.get_head_size()
        vllm_config.compilation_config.static_forward_context["layer.0"] = Attention(
            num_heads, head_size, 0.1
        )
        runner = GPUModelRunner(vllm_config, DEVICE_TYPE)
        initialize_kv_cache(runner)
        yield runner
```
**EN:** Fixture/helper `model_runner` prepares reusable state for downstream tests. Key calls include `get_vllm_config, set_current_vllm_config, model_config.get_num_kv_heads, model_config.get_head_size, Attention, GPUModelRunner`.
**CN:** `model_runner` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `get_vllm_config, set_current_vllm_config, model_config.get_num_kv_heads, model_config.get_head_size, Attention, GPUModelRunner`。

### Module state / 模块级状态 (line 132)
```python
model_runner_2 = model_runner
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `model_runner_2`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`model_runner_2`。

### _schedule_new_request (lines 135-165)
```python
def _schedule_new_request(*req_ids: str) -> SchedulerOutput:
    new_reqs = []
    num_scheduled_tokens = {}
    total_num_scheduled_tokens = 0
    for req_id in req_ids:
        new_reqs.append(
            NewRequestData(
                req_id=req_id,
                prompt_token_ids=[1, 2, 3],
                mm_features=[],
                sampling_params=SamplingParams(),
                pooling_params=None,
                block_ids=([0],),
                num_computed_tokens=0,
                lora_request=None,
            )
        )
        num_scheduled_tokens[req_id] = 3
        total_num_scheduled_tokens += num_scheduled_tokens[req_id]

    return SchedulerOutput(
        scheduled_new_reqs=new_reqs,
        scheduled_cached_reqs=CachedRequestData.make_empty(),
        num_scheduled_tokens=num_scheduled_tokens,
        total_num_scheduled_tokens=total_num_scheduled_tokens,
        scheduled_spec_decode_tokens={},
        scheduled_encoder_inputs={},
        num_common_prefix_blocks=[],
        finished_req_ids=set(),
        free_encoder_mm_hashes=[],
    )
```
**EN:** Helper function `_schedule_new_request` encapsulates reusable logic for `schedule new request`. Key calls include `SchedulerOutput, new_reqs.append, NewRequestData, CachedRequestData.make_empty, set, SamplingParams`.
**CN:** 辅助函数 `_schedule_new_request` 封装了与 `schedule new request` 相关的可复用逻辑。 关键调用包括 `SchedulerOutput, new_reqs.append, NewRequestData, CachedRequestData.make_empty, set, SamplingParams`。

### _schedule_cached_requests (lines 168-193)
```python
def _schedule_cached_requests(
    req_ids: list[str],
    num_scheduled_tokens: dict[str, int],
    new_token_ids: list[list[int]],
    num_computed_tokens: list[int],
    num_output_tokens: list[int],
) -> SchedulerOutput:
    return SchedulerOutput(
        scheduled_new_reqs=[],
        scheduled_cached_reqs=CachedRequestData(
            req_ids=req_ids,
            resumed_req_ids=set(),
            new_token_ids=new_token_ids,
            all_token_ids={},
            new_block_ids=[None] * len(req_ids),
            num_computed_tokens=num_computed_tokens,
            num_output_tokens=num_output_tokens,
        ),
        num_scheduled_tokens=num_scheduled_tokens,
        total_num_scheduled_tokens=sum(num_scheduled_tokens.values()),
        scheduled_spec_decode_tokens={},
        scheduled_encoder_inputs={},
        num_common_prefix_blocks=[],
        finished_req_ids=set(),
        free_encoder_mm_hashes=[],
    )
```
**EN:** Helper function `_schedule_cached_requests` encapsulates reusable logic for `schedule cached requests`. Inputs: `req_ids, num_scheduled_tokens, new_token_ids, num_computed_tokens, num_output_tokens`. Key calls include `SchedulerOutput, CachedRequestData, sum, set, num_scheduled_tokens.values, len`.
**CN:** 辅助函数 `_schedule_cached_requests` 封装了与 `schedule cached requests` 相关的可复用逻辑。 输入参数：`req_ids, num_scheduled_tokens, new_token_ids, num_computed_tokens, num_output_tokens`。 关键调用包括 `SchedulerOutput, CachedRequestData, sum, set, num_scheduled_tokens.values, len`。

### _is_req_scheduled (lines 196-197)
```python
def _is_req_scheduled(model_runner, req_id: str) -> bool:
    return req_id in model_runner.input_batch.req_id_to_index
```
**EN:** Helper function `_is_req_scheduled` encapsulates reusable logic for `is req scheduled`. Inputs: `model_runner, req_id`.
**CN:** 辅助函数 `_is_req_scheduled` 封装了与 `is req scheduled` 相关的可复用逻辑。 输入参数：`model_runner, req_id`。

### _is_req_added (lines 200-201)
```python
def _is_req_added(model_runner, req_id: str) -> bool:
    return req_id in model_runner.requests
```
**EN:** Helper function `_is_req_added` encapsulates reusable logic for `is req added`. Inputs: `model_runner, req_id`.
**CN:** 辅助函数 `_is_req_added` 封装了与 `is req added` 相关的可复用逻辑。 输入参数：`model_runner, req_id`。

### _is_sampling_metadata_changed (lines 204-207)
```python
def _is_sampling_metadata_changed(
    model_runner, sampling_metadata_before: SamplingMetadata
):
    return model_runner.input_batch.sampling_metadata is not (sampling_metadata_before)
```
**EN:** Helper function `_is_sampling_metadata_changed` encapsulates reusable logic for `is sampling metadata changed`. Inputs: `model_runner, sampling_metadata_before`.
**CN:** 辅助函数 `_is_sampling_metadata_changed` 封装了与 `is 采样 metadata changed` 相关的可复用逻辑。 输入参数：`model_runner, sampling_metadata_before`。

### _is_req_state_block_table_match (lines 210-219)
```python
def _is_req_state_block_table_match(model_runner, req_id: str) -> bool:
    req_index = model_runner.input_batch.req_id_to_index[req_id]
    block_table = model_runner.input_batch.block_table[0]
    req_state = model_runner.requests[req_id]
    if block_table.num_blocks_per_row[req_index] != len(req_state.block_ids[0]):
        return False
    num_blocks = block_table.num_blocks_per_row[req_index]
    return (
        block_table.block_table.np[req_index, :num_blocks] == req_state.block_ids[0]
    ).all()
```
**EN:** Helper function `_is_req_state_block_table_match` encapsulates reusable logic for `is req state block table match`. Inputs: `model_runner, req_id`. Key calls include `all, len`.
**CN:** 辅助函数 `_is_req_state_block_table_match` 封装了与 `is req state block table match` 相关的可复用逻辑。 输入参数：`model_runner, req_id`。 关键调用包括 `all, len`。

### _make_mock_backend_for_kernel_block_size (lines 222-230)
```python
def _make_mock_backend_for_kernel_block_size(
    supported_sizes: list[int | MultipleOf],
):
    class _MockBackend:
        @staticmethod
        def get_supported_kernel_block_sizes():
            return supported_sizes

    return _MockBackend()
```
**EN:** Helper function `_make_mock_backend_for_kernel_block_size` encapsulates reusable logic for `mock backend for kernel block size`. Inputs: `supported_sizes`. Key calls include `_MockBackend`.
**CN:** 辅助函数 `_make_mock_backend_for_kernel_block_size` 封装了与 `mock 后端 for kernel block size` 相关的可复用逻辑。 输入参数：`supported_sizes`。 关键调用包括 `_MockBackend`。

### _make_kv_cache_spec (lines 233-234)
```python
def _make_kv_cache_spec() -> FullAttentionSpec:
    return FullAttentionSpec(block_size=1, num_kv_heads=1, head_size=1, dtype="float16")
```
**EN:** Helper function `_make_kv_cache_spec` encapsulates reusable logic for `KV cache spec`. Key calls include `FullAttentionSpec`.
**CN:** 辅助函数 `_make_kv_cache_spec` 封装了与 `KV 缓存 spec` 相关的可复用逻辑。 关键调用包括 `FullAttentionSpec`。

### test_select_common_block_size_prefers_manager_block_size (lines 237-242)
```python
def test_select_common_block_size_prefers_manager_block_size():
    backend_a = _make_mock_backend_for_kernel_block_size([MultipleOf(32)])
    backend_b = _make_mock_backend_for_kernel_block_size([64, MultipleOf(16)])

    selected_size = select_common_block_size(128, [backend_a, backend_b])
    assert selected_size == 128
```
**EN:** Test case covering `select common block size prefers manager block size`. It exercises `_make_mock_backend_for_kernel_block_size, select_common_block_size, MultipleOf`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `select common block size prefers manager block size` 的测试用例。 该测试会调用 `_make_mock_backend_for_kernel_block_size, select_common_block_size, MultipleOf`。 代码主体包含 1 个显式断言。

### test_select_common_block_size_uses_largest_shared_int (lines 245-250)
```python
def test_select_common_block_size_uses_largest_shared_int():
    backend_a = _make_mock_backend_for_kernel_block_size([128, 64])
    backend_b = _make_mock_backend_for_kernel_block_size([64, 32])

    selected_size = select_common_block_size(256, [backend_a, backend_b])
    assert selected_size == 64
```
**EN:** Test case covering `select common block size uses largest shared int`. It exercises `_make_mock_backend_for_kernel_block_size, select_common_block_size`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `select common block size uses largest shared int` 的测试用例。 该测试会调用 `_make_mock_backend_for_kernel_block_size, select_common_block_size`。 代码主体包含 1 个显式断言。

### test_sample_tokens_receives_pp_sampled_ids_only_on_non_last_rank (lines 258-284)
```python
def test_sample_tokens_receives_pp_sampled_ids_only_on_non_last_rank(
    monkeypatch: pytest.MonkeyPatch,
    world_size: int,
    is_last_rank: bool,
    expected_calls: int,
):
    runner = GPUModelRunner.__new__(GPUModelRunner)
    runner.execute_model_state = None
    runner.kv_connector_output = None
    runner.use_async_scheduling = True
    receive_calls = 0

    def receive_prev_sampled_token_ids():
        nonlocal receive_calls
        receive_calls += 1

    runner._pp_receive_prev_sampled_token_ids_to_input_batch = (
        receive_prev_sampled_token_ids
    )
    monkeypatch.setattr(
        gpu_model_runner_module,
        "get_pp_group",
        lambda: SimpleNamespace(world_size=world_size, is_last_rank=is_last_rank),
    )

    assert GPUModelRunner.sample_tokens(runner, None) is None
    assert receive_calls == expected_calls
```
**EN:** Test case covering `sample tokens receives pp sampled ids only on non last rank`. Inputs/fixtures: `monkeypatch, world_size, is_last_rank, expected_calls`. It exercises `mark.parametrize, GPUModelRunner.__new__, monkeypatch.setattr, GPUModelRunner.sample_tokens, SimpleNamespace`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `sample tokens receives pp sampled ids only on non last rank` 的测试用例。 输入或 fixture：`monkeypatch, world_size, is_last_rank, expected_calls`。 该测试会调用 `mark.parametrize, GPUModelRunner.__new__, monkeypatch.setattr, GPUModelRunner.sample_tokens, SimpleNamespace`。 代码主体包含 2 个显式断言。

### test_sample_tokens_skips_pp_group_lookup_without_async_scheduling (lines 288-302)
```python
def test_sample_tokens_skips_pp_group_lookup_without_async_scheduling(
    monkeypatch: pytest.MonkeyPatch,
):
    runner = GPUModelRunner.__new__(GPUModelRunner)
    runner.execute_model_state = None
    runner.kv_connector_output = None
    runner.use_async_scheduling = False

    monkeypatch.setattr(
        gpu_model_runner_module,
        "get_pp_group",
        pytest.fail,
    )

    assert GPUModelRunner.sample_tokens(runner, None) is None
```
**EN:** Test case covering `sample tokens skips pp group lookup without async scheduling`. Inputs/fixtures: `monkeypatch`. It exercises `GPUModelRunner.__new__, monkeypatch.setattr, GPUModelRunner.sample_tokens`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `sample tokens skips pp group lookup without async scheduling` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `GPUModelRunner.__new__, monkeypatch.setattr, GPUModelRunner.sample_tokens`。 代码主体包含 1 个显式断言。

### test_select_common_block_size_no_valid_option (lines 305-310)
```python
def test_select_common_block_size_no_valid_option():
    backend_a = _make_mock_backend_for_kernel_block_size([64])
    backend_b = _make_mock_backend_for_kernel_block_size([MultipleOf(16)])

    with pytest.raises(ValueError):
        select_common_block_size(48, [backend_a, backend_b])
```
**EN:** Test case covering `select common block size no valid option`. It exercises `_make_mock_backend_for_kernel_block_size, pytest.raises, select_common_block_size, MultipleOf`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `select common block size no valid option` 的测试用例。 该测试会调用 `_make_mock_backend_for_kernel_block_size, pytest.raises, select_common_block_size, MultipleOf`。 主要通过预期异常检查来完成验证。

### test_update_states_new_request (lines 313-324)
```python
def test_update_states_new_request(model_runner, dist_init):
    req_id = "req_0"

    # new req
    scheduler_output = _schedule_new_request(req_id)

    metadata_before = model_runner.input_batch.sampling_metadata
    model_runner._update_states(scheduler_output)
    assert _is_sampling_metadata_changed(model_runner, metadata_before)
    assert _is_req_added(model_runner, req_id)
    assert _is_req_scheduled(model_runner, req_id)
    assert _is_req_state_block_table_match(model_runner, req_id)
```
**EN:** Test case covering `update states new request`. Inputs/fixtures: `model_runner, dist_init`. It exercises `_schedule_new_request, model_runner._update_states, _is_sampling_metadata_changed, _is_req_added, _is_req_scheduled, _is_req_state_block_table_match`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `update states new request` 的测试用例。 输入或 fixture：`model_runner, dist_init`。 该测试会调用 `_schedule_new_request, model_runner._update_states, _is_sampling_metadata_changed, _is_req_added, _is_req_scheduled, _is_req_state_block_table_match`。 代码主体包含 4 个显式断言。

### test_update_states_request_finished (lines 327-354)
```python
def test_update_states_request_finished(model_runner, dist_init):
    req_id = "req_0"

    # new req
    scheduler_output = _schedule_new_request(req_id)

    model_runner._update_states(scheduler_output)
    assert _is_req_added(model_runner, req_id)
    assert _is_req_scheduled(model_runner, req_id)

    # finish req
    scheduler_output = SchedulerOutput(
        scheduled_new_reqs=[],
        scheduled_cached_reqs=CachedRequestData.make_empty(),
        num_scheduled_tokens={},
        total_num_scheduled_tokens=0,
        scheduled_spec_decode_tokens={},
        scheduled_encoder_inputs={},
        num_common_prefix_blocks=[],
        finished_req_ids={req_id},
        free_encoder_mm_hashes=[],
    )

    metadata_before = model_runner.input_batch.sampling_metadata
    model_runner._update_states(scheduler_output)
    assert _is_sampling_metadata_changed(model_runner, metadata_before)
    assert not _is_req_added(model_runner, req_id)
    assert not _is_req_scheduled(model_runner, req_id)
```
**EN:** Test case covering `update states request finished`. Inputs/fixtures: `model_runner, dist_init`. It exercises `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, _is_sampling_metadata_changed`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `update states request finished` 的测试用例。 输入或 fixture：`model_runner, dist_init`。 该测试会调用 `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, _is_sampling_metadata_changed`。 代码主体包含 5 个显式断言。

### test_update_states_request_resumed (lines 357-412)
```python
def test_update_states_request_resumed(model_runner, dist_init):
    req_id = "req_0"

    # new req
    scheduler_output = _schedule_new_request(req_id)
    model_runner._update_states(scheduler_output)
    assert _is_req_added(model_runner, req_id)
    assert _is_req_scheduled(model_runner, req_id)
    # unschedule req
    scheduler_output = SchedulerOutput(
        scheduled_new_reqs=[],
        scheduled_cached_reqs=CachedRequestData.make_empty(),
        num_scheduled_tokens={},
        total_num_scheduled_tokens=0,
        scheduled_spec_decode_tokens={},
        scheduled_encoder_inputs={},
    # ... excerpt omitted for brevity ...
    assert not _is_req_scheduled(model_runner, req_id)
    metadata_before = model_runner.input_batch.sampling_metadata
    assert _is_sampling_metadata_changed(model_runner, metadata_before)
    assert _is_req_state_block_table_match(model_runner, req_id)
```
**EN:** Test case covering `update states request resumed`. Inputs/fixtures: `model_runner, dist_init`. It exercises `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, CachedRequestData`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `update states request resumed` 的测试用例。 输入或 fixture：`model_runner, dist_init`。 该测试会调用 `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, CachedRequestData`。 代码主体包含 8 个显式断言。

### test_get_nans_in_logits (lines 415-472)
```python
def test_get_nans_in_logits(model_runner, dist_init):
    req_ids = ("req_0", "req_1")

    scheduler_output = _schedule_new_request(*req_ids)
    model_runner._update_states(scheduler_output)
    logits = torch.tensor(
        [
            [1.0, 2.0, 3.0],
            [3.0, 2.0, 1.0],
        ],
        device=DEVICE_TYPE,
    )
    result = model_runner._get_nans_in_logits(logits)
    assert result == {"req_0": 0, "req_1": 0}
    # ... excerpt omitted for brevity ...
    assert result == {"req_0": 1, "req_1": 2}
    assert result == {"req_0": 0, "req_1": 2}
    assert result == {"req_0": 1, "req_1": 0}
            [float("nan"), 2.0, 3.0],
    assert result == {"req_0": 2, "req_1": 0}
```
**EN:** Test case covering `get nans in logits`. Inputs/fixtures: `model_runner, dist_init`. It exercises `_schedule_new_request, model_runner._update_states, torch.tensor, model_runner._get_nans_in_logits, float`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `get nans in logits` 的测试用例。 输入或 fixture：`model_runner, dist_init`。 该测试会调用 `_schedule_new_request, model_runner._update_states, torch.tensor, model_runner._get_nans_in_logits, float`。 代码主体包含 6 个显式断言。

### test_update_states_no_changes (lines 475-503)
```python
def test_update_states_no_changes(model_runner, dist_init):
    req_id = "req_0"

    # new req
    scheduler_output = _schedule_new_request(req_id)

    model_runner._update_states(scheduler_output)
    assert _is_req_added(model_runner, req_id)
    assert _is_req_scheduled(model_runner, req_id)

    # schedule req
    scheduler_output = SchedulerOutput(
        scheduled_new_reqs=[],
        scheduled_cached_reqs=CachedRequestData.make_empty(),
        num_scheduled_tokens={req_id: 1},
        total_num_scheduled_tokens=1,
        scheduled_spec_decode_tokens={},
        scheduled_encoder_inputs={},
        num_common_prefix_blocks=[],
        finished_req_ids=set(),
        free_encoder_mm_hashes=[],
    )

    metadata_before = model_runner.input_batch.sampling_metadata
    model_runner._update_states(scheduler_output)
    assert not _is_sampling_metadata_changed(model_runner, metadata_before)
    assert _is_req_added(model_runner, req_id)
    assert _is_req_scheduled(model_runner, req_id)
    assert _is_req_state_block_table_match(model_runner, req_id)
```
**EN:** Test case covering `update states no changes`. Inputs/fixtures: `model_runner, dist_init`. It exercises `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, _is_req_state_block_table_match`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `update states no changes` 的测试用例。 输入或 fixture：`model_runner, dist_init`。 该测试会调用 `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, _is_req_state_block_table_match`。 代码主体包含 6 个显式断言。

### test_update_states_request_unscheduled (lines 506-540)
```python
def test_update_states_request_unscheduled(model_runner, dist_init):
    req_ids = ("req_0", "req_1")

    # new reqs
    scheduler_output = _schedule_new_request(*req_ids)
    model_runner._update_states(scheduler_output)
    assert _is_req_added(model_runner, req_ids[0])
    assert _is_req_scheduled(model_runner, req_ids[0])
    assert _is_req_added(model_runner, req_ids[1])
    assert _is_req_scheduled(model_runner, req_ids[1])
    # unschedule req_1
    scheduler_output = SchedulerOutput(
        scheduled_new_reqs=[],
        scheduled_cached_reqs=CachedRequestData.make_empty(),
    # ... excerpt omitted for brevity ...
    assert _is_sampling_metadata_changed(model_runner, metadata_before)
    assert not _is_req_scheduled(model_runner, req_ids[1])
```
**EN:** Test case covering `update states request unscheduled`. Inputs/fixtures: `model_runner, dist_init`. It exercises `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, _is_sampling_metadata_changed`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `update states request unscheduled` 的测试用例。 输入或 fixture：`model_runner, dist_init`。 该测试会调用 `_schedule_new_request, model_runner._update_states, _is_req_added, _is_req_scheduled, SchedulerOutput, _is_sampling_metadata_changed`。 代码主体包含 9 个显式断言。

### test_update_states_pp_non_async_multi_request_keeps_token_buffers_consistent (lines 543-604)
```python
def test_update_states_pp_non_async_multi_request_keeps_token_buffers_consistent(
    model_runner, model_runner_2, dist_init, monkeypatch
):
    req_ids = ["req_0", "req_1"]
    non_last_runner = model_runner
    last_runner = model_runner_2
    non_last_runner.use_async_scheduling = False
    last_runner.use_async_scheduling = False

    # Both ranks start from the same request set.
    monkeypatch.setattr(
        "vllm.v1.worker.gpu_model_runner.get_pp_group",
        lambda: SimpleNamespace(is_last_rank=False, world_size=2),
    )
    non_last_runner._update_states(_schedule_new_request(*req_ids))
    last_runner._update_states(_schedule_new_request(*req_ids))
    sampled_by_last_rank = {req_ids[0]: 101, req_ids[1]: 201}
    # ... excerpt omitted for brevity ...
        assert non_last_len == last_len
        assert (
            non_last_runner.input_batch.token_ids_cpu[
                non_last_idx, :non_last_len
            ].tolist()
            == last_runner.input_batch.token_ids_cpu[last_idx, :last_len].tolist()
        )
```
**EN:** Test case covering `update states pp non async multi request keeps token buffers consistent`. Inputs/fixtures: `model_runner, model_runner_2, dist_init, monkeypatch`. It exercises `monkeypatch.setattr, non_last_runner._update_states, last_runner._update_states, sampled_by_last_rank.items, _schedule_cached_requests, _schedule_new_request`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `update states pp non async multi request keeps token buffers consistent` 的测试用例。 输入或 fixture：`model_runner, model_runner_2, dist_init, monkeypatch`。 该测试会调用 `monkeypatch.setattr, non_last_runner._update_states, last_runner._update_states, sampled_by_last_rank.items, _schedule_cached_requests, _schedule_new_request`。 代码主体包含 2 个显式断言。

### test_update_states_pp_async_multi_request_keeps_rank_state_consistent (lines 607-669)
```python
def test_update_states_pp_async_multi_request_keeps_rank_state_consistent(
    model_runner, model_runner_2, dist_init, monkeypatch
):
    req_ids = ["req_0", "req_1"]
    non_last_runner = model_runner
    last_runner = model_runner_2
    non_last_runner.use_async_scheduling = True
    last_runner.use_async_scheduling = True

    # Both ranks start from the same request set.
    monkeypatch.setattr(
        "vllm.v1.worker.gpu_model_runner.get_pp_group",
        lambda: SimpleNamespace(is_last_rank=False, world_size=2),
    )
    non_last_runner._update_states(_schedule_new_request(*req_ids))
    last_runner._update_states(_schedule_new_request(*req_ids))
    # Simulate async previous-step sampled tokens known on both ranks.
    # ... excerpt omitted for brevity ...
        assert non_last_len == last_len
        assert (
            non_last_runner.input_batch.token_ids_cpu[
                non_last_idx, :non_last_len
            ].tolist()
            == last_runner.input_batch.token_ids_cpu[last_idx, :last_len].tolist()
        )
```
**EN:** Test case covering `update states pp async multi request keeps rank state consistent`. Inputs/fixtures: `model_runner, model_runner_2, dist_init, monkeypatch`. It exercises `monkeypatch.setattr, non_last_runner._update_states, last_runner._update_states, _schedule_cached_requests, _schedule_new_request, sampled_by_last_rank.items`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `update states pp async multi request keeps rank state consistent` 的测试用例。 输入或 fixture：`model_runner, model_runner_2, dist_init, monkeypatch`。 该测试会调用 `monkeypatch.setattr, non_last_runner._update_states, last_runner._update_states, _schedule_cached_requests, _schedule_new_request, sampled_by_last_rank.items`。 代码主体包含 2 个显式断言。

### test_kv_cache_stride_order (lines 672-718)
```python
def test_kv_cache_stride_order(monkeypatch, model_runner):
    # This test checks if GPUModelRunner initializes correctly when an attention
    # backend enforces a non-default KV cache stride order.
    n_heads = model_runner.model_config.get_num_kv_heads(model_runner.parallel_config)
    head_size = model_runner.model_config.get_head_size()

    # Get the expected shape from the backend's get_kv_cache_shape method
    # to ensure compatibility with different backends (triton vs flexattention)
    attn_backend = None
    for attn_group in model_runner._attn_group_iterator():
        attn_backend = attn_group.backend
        break
    assert attn_backend is not None, "No attention backend found"
    expected_kv_cache_shape = list(
        attn_backend.get_kv_cache_shape(NUM_BLOCKS, BLOCK_SIZE, n_heads, head_size)
    )
    # ... excerpt omitted for brevity ...
    # Permutation that gets you back to expected kv shape
            assert not include_num_layers_dimension
            return test_stride
        kv_cache_shape = model_runner.kv_caches[0].shape
        assert list(kv_cache_shape) == expected_kv_cache_shape
        if default_stride == test_stride:
            assert all(kv.is_contiguous() for kv in model_runner.kv_caches)
        else:
            assert all(not kv.is_contiguous() for kv in model_runner.kv_caches)
```
**EN:** Test case covering `KV cache stride order`. Inputs/fixtures: `monkeypatch, model_runner`. It exercises `model_config.get_num_kv_heads, model_config.get_head_size, model_runner._attn_group_iterator, list, tuple, attn_backend.get_kv_cache_shape`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 缓存 stride order` 的测试用例。 输入或 fixture：`monkeypatch, model_runner`。 该测试会调用 `model_config.get_num_kv_heads, model_config.get_head_size, model_runner._attn_group_iterator, list, tuple, attn_backend.get_kv_cache_shape`。 代码主体包含 5 个显式断言。

### test_update_config (lines 721-727)
```python
def test_update_config(model_runner):
    # Simple update
    model_runner.update_config({"load_config": {"load_format": "dummy"}})
    assert model_runner.load_config.load_format == "dummy"
    # Raise error on non-existing config
    with pytest.raises(AssertionError):
        model_runner.update_config({"do_not_exist_config": "dummy"})
```
**EN:** Test case covering `update config`. Inputs/fixtures: `model_runner`. It exercises `model_runner.update_config, pytest.raises`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `update config` 的测试用例。 输入或 fixture：`model_runner`。 该测试会调用 `model_runner.update_config, pytest.raises`。 代码主体包含 1 个显式断言。

### test_load_model_weights_inplace (lines 730-744)
```python
def test_load_model_weights_inplace(dist_init, model_runner, model_runner_2):
    # In this test, model_runner loads model + weights in one go, while
    # model_runner_2 loads dummy weights first then load real weights inplace
    model_runner.load_model()
    original_load_format = model_runner_2.load_config.load_format
    model_runner_2.update_config({"load_config": {"load_format": "dummy"}})
    model_runner_2.load_model()  # Initial model loading with dummy weights
    assert str(model_runner.get_model().state_dict()) != str(
        model_runner_2.get_model().state_dict()
    )
    model_runner_2.update_config({"load_config": {"load_format": original_load_format}})
    model_runner_2.reload_weights()  # Load real weights inplace
    assert str(model_runner.get_model().state_dict()) == str(
        model_runner_2.get_model().state_dict()
    )
```
**EN:** Test case covering `load model weights inplace`. Inputs/fixtures: `dist_init, model_runner, model_runner_2`. It exercises `model_runner.load_model, model_runner_2.update_config, model_runner_2.load_model, model_runner_2.reload_weights, str, get_model.state_dict`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `load model weights inplace` 的测试用例。 输入或 fixture：`dist_init, model_runner, model_runner_2`。 该测试会调用 `model_runner.load_model, model_runner_2.update_config, model_runner_2.load_model, model_runner_2.reload_weights, str, get_model.state_dict`。 代码主体包含 2 个显式断言。

### test_reload_weights_before_load_model (lines 747-749)
```python
def test_reload_weights_before_load_model(model_runner):
    with pytest.raises(ValueError):
        model_runner.reload_weights()
```
**EN:** Test case covering `reload weights before load model`. Inputs/fixtures: `model_runner`. It exercises `pytest.raises, model_runner.reload_weights`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `reload weights before load model` 的测试用例。 输入或 fixture：`model_runner`。 该测试会调用 `pytest.raises, model_runner.reload_weights`。 主要通过预期异常检查来完成验证。

### test_sample_passes_reordered_draft_probs_to_rejection_sampler (lines 752-782)
```python
def test_sample_passes_reordered_draft_probs_to_rejection_sampler():
    runner = object.__new__(GPUModelRunner)
    runner.use_async_scheduling = False
    runner.input_batch = SimpleNamespace(
        sampling_metadata=Mock(spec=SamplingMetadata),
        update_async_output_token_ids=Mock(),
        req_ids=["req_a", "req_b", "req_c"],
    )
    runner.rejection_sampler = Mock(return_value="sampler_output")
    runner.sampler = Mock()
    runner._draft_prob_req_ids = ["req_c", "req_a", "req_b"]
    runner._draft_probs = torch.arange(3 * 3 * 4, dtype=torch.float32).reshape(3, 3, 4)

    spec_decode_metadata = SpecDecodeMetadata.make_dummy(
        [[1, 2], [], [3]],
        device=torch.device("cpu"),
    )
    logits = torch.randn(6, 4)

    output = GPUModelRunner._sample(runner, logits, spec_decode_metadata)

    assert output == "sampler_output"
    passed_draft_probs = runner.rejection_sampler.call_args.args[1]
    expected_draft_probs = torch.cat(
        [
            runner._draft_probs[1, :2],
            runner._draft_probs[0, :1],
        ],
        dim=0,
    )
    assert torch.equal(passed_draft_probs, expected_draft_probs)
```
**EN:** Test case covering `sample passes reordered draft probs to rejection sampler`. It exercises `object.__new__, SimpleNamespace, Mock, arange.reshape, SpecDecodeMetadata.make_dummy, torch.randn`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `sample passes reordered draft probs to rejection sampler` 的测试用例。 该测试会调用 `object.__new__, SimpleNamespace, Mock, arange.reshape, SpecDecodeMetadata.make_dummy, torch.randn`。 代码主体包含 2 个显式断言。

### test_init_kv_cache_with_kv_sharing_invalid_target_layer_order (lines 785-809)
```python
def test_init_kv_cache_with_kv_sharing_invalid_target_layer_order(default_vllm_config):
    torch.set_default_dtype(torch.float16)
    layer_0 = "model.layers.0.self_attn.attn"
    layer_1 = "model.layers.1.self_attn.attn"
    error_msg = f"{layer_1} must come before the current layer"
    with pytest.raises(ValueError, match=error_msg):
        fwd_context = {
            # initialization below will fail because target layer is invalid;
            # the target layer needs to come before layer 1
            layer_0: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_0,
                kv_sharing_target_layer_name=layer_1,
            ),
            layer_1: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_1,
            ),
        }
        # suppress var not used error
        assert fwd_context is not None
```
**EN:** Test case covering `init KV cache with KV sharing invalid target layer order`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.set_default_dtype, pytest.raises, Attention`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `init KV 缓存 with kv sharing invalid target layer order` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.set_default_dtype, pytest.raises, Attention`。 代码主体包含 1 个显式断言。

### test_init_kv_cache_with_kv_sharing_target_layer_not_exist (lines 812-836)
```python
def test_init_kv_cache_with_kv_sharing_target_layer_not_exist(default_vllm_config):
    torch.set_default_dtype(torch.float16)
    layer_0 = "model.layers.0.self_attn.attn"
    layer_1 = "model.layers.1.self_attn.attn"
    invalid_layer = "model.layers.0.cross_attn.attn"
    error_msg = f"{invalid_layer} is not a valid Attention layer in the model"
    with pytest.raises(ValueError, match=error_msg):
        fwd_context = {
            layer_0: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_0,
            ),
            layer_1: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_1,
                # invalid layer: cross_attn.atn doesn't exist!
                kv_sharing_target_layer_name=invalid_layer,
            ),
        }
        # suppress var not used error
        assert fwd_context is not None
```
**EN:** Test case covering `init KV cache with KV sharing target layer not exist`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.set_default_dtype, pytest.raises, Attention`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `init KV 缓存 with kv sharing target layer not exist` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.set_default_dtype, pytest.raises, Attention`。 代码主体包含 1 个显式断言。

### test_init_kv_cache_with_kv_sharing_target_same_as_current (lines 839-863)
```python
def test_init_kv_cache_with_kv_sharing_target_same_as_current(default_vllm_config):
    torch.set_default_dtype(torch.float16)
    layer_0 = "model.layers.0.self_attn.attn"
    layer_1 = "model.layers.1.self_attn.attn"
    error_msg = f"{layer_1} cannot be the same as the current layer"
    with pytest.raises(ValueError, match=error_msg):
        fwd_context = {
            # initialization below will fail because target layer is invalid;
            # the target layer needs to come before layer 1
            layer_0: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_0,
            ),
            layer_1: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_1,
                kv_sharing_target_layer_name=layer_1,
            ),
        }
        # suppress var not used error
        assert fwd_context is not None
```
**EN:** Test case covering `init KV cache with KV sharing target same as current`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.set_default_dtype, pytest.raises, Attention`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `init KV 缓存 with kv sharing target same as current` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.set_default_dtype, pytest.raises, Attention`。 代码主体包含 1 个显式断言。

### test_init_kv_cache_without_kv_sharing (lines 866-930)
```python
def test_init_kv_cache_without_kv_sharing(default_vllm_config):
    torch.set_default_dtype(torch.float16)
    layer_0 = "model.layers.0.self_attn.attn"
    layer_1 = "model.layers.1.self_attn.attn"
    vllm_config = get_vllm_config()
    with set_current_vllm_config(vllm_config):
        fwd_context = {
            layer_0: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_0,
            ),
            layer_1: Attention(
                prefix=layer_1,
    # ... excerpt omitted for brevity ...
        assert fwd_context is not None
    assert len(kv_cache_spec) == 2
    assert len(runner.shared_kv_cache_layers) == 0
    num_expected_blocks = 327680  # 20GB / 32KB / 2 (num layers)
    assert kv_cache_config.num_blocks == num_expected_blocks
    assert len(kv_cache_config.kv_cache_tensors) == 2

    # check layer 1 added to kv cache group's layer names
    assert len(kv_cache_config.kv_cache_groups) == 1
    assert len(kv_cache_config.kv_cache_groups[0].layer_names) == 2
    assert kv_cache_config.kv_cache_groups[0].layer_names[0] == layer_0
    assert kv_cache_config.kv_cache_groups[0].layer_names[1] == layer_1
```
**EN:** Test case covering `init KV cache without KV sharing`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.set_default_dtype, get_vllm_config, GPUModelRunner, runner.get_kv_cache_spec, estimate_max_model_len, runner.initialize_kv_cache`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `init KV 缓存 without kv sharing` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.set_default_dtype, get_vllm_config, GPUModelRunner, runner.get_kv_cache_spec, estimate_max_model_len, runner.initialize_kv_cache`。 代码主体包含 13 个显式断言。

### test_init_kv_cache_with_kv_sharing_valid (lines 933-1000)
```python
def test_init_kv_cache_with_kv_sharing_valid(default_vllm_config):
    torch.set_default_dtype(torch.float16)
    layer_0 = "model.layers.0.self_attn.attn"
    layer_1 = "model.layers.1.self_attn.attn"
    vllm_config = get_vllm_config()
    with set_current_vllm_config(vllm_config):
        fwd_context = {
            layer_0: Attention(
                num_heads=8,
                head_size=64,
                scale=1.0,
                prefix=layer_0,
            ),
            layer_1: Attention(
                prefix=layer_1,
    # ... excerpt omitted for brevity ...
        assert fwd_context is not None
    assert len(kv_cache_spec) == 1
    assert layer_0 in kv_cache_spec
    assert runner.shared_kv_cache_layers[layer_1] == layer_0
    num_expected_blocks = 655360  # 20GB / 32KB
    assert kv_cache_config.num_blocks == num_expected_blocks

    # check layer 1 added to kv cache group's layer names
    assert len(kv_cache_config_after_init.kv_cache_groups) == 1
    assert len(kv_cache_config_after_init.kv_cache_groups[0].layer_names) == 2
    assert kv_cache_config_after_init.kv_cache_groups[0].layer_names[0] == layer_0
    assert kv_cache_config_after_init.kv_cache_groups[0].layer_names[1] == layer_1
```
**EN:** Test case covering `init KV cache with KV sharing valid`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.set_default_dtype, get_vllm_config, GPUModelRunner, runner.get_kv_cache_spec, estimate_max_model_len, runner.initialize_kv_cache`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `init KV 缓存 with kv sharing valid` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.set_default_dtype, get_vllm_config, GPUModelRunner, runner.get_kv_cache_spec, estimate_max_model_len, runner.initialize_kv_cache`。 代码主体包含 13 个显式断言。

### test_hybrid_attention_mamba_tensor_shapes (lines 1007-1195)
```python
def test_hybrid_attention_mamba_tensor_shapes():
    """
    The GPU model runner creates different views into the
    KVCacheTensors for the attention and mamba layers
    (via _reshape_kv_cache_tensors function). This test verifies
    that the views are compatible: writing a mamba block
    will not corrupt an attention block and vice versa

    set_random_seed(42)
    update_environment_variables(
        {
            "RANK": "0",
            "LOCAL_RANK": "0",
            "WORLD_SIZE": "1",
            "MASTER_ADDR": "localhost",
            "MASTER_PORT": "12345",
    # ... excerpt omitted for brevity ...
        assert fwd_context is not None
    assert attn_shape[0] % num_blocks == 0
            expected = attn_blocks_constant[i]
            assert torch.equal(actual_kv[0], expected)
            assert torch.equal(actual_kv[1], expected)
            expected_conv = conv_blocks_constant[i]
            actual_conv = vllm_ctx[layer].kv_cache[0][kv_block, :]
            actual_ssm = vllm_ctx[layer].kv_cache[1][kv_block, :]
            expected_ssm = ssm_blocks_constant[i]
            assert torch.equal(actual_conv, expected_conv)
            assert torch.equal(actual_ssm, expected_ssm)
```
**EN:** Test case covering `hybrid attentionMamba tensor shapes`. It exercises `mark.skipif, set_random_seed, update_environment_variables, torch.set_default_dtype, ModelConfig, SchedulerConfig`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid attentionmamba tensor shapes` 的测试用例。 该测试会调用 `mark.skipif, set_random_seed, update_environment_variables, torch.set_default_dtype, ModelConfig, SchedulerConfig`。 代码主体包含 8 个显式断言。

### test_update_hybrid_attention_mamba_layout_with_num_block_2_rewrites_stride (lines 1198-1222)
```python
def test_update_hybrid_attention_mamba_layout_with_num_block_2_rewrites_stride():
    from vllm.v1.attention.backends.flash_attn import FlashAttentionBackend

    ambiguous_cache = torch.empty((2, 2, BLOCK_SIZE, 1, 8), dtype=torch.float16)
    """Ambiguous, because both dims[0=kv_dim] and dims[1=num_blocks] == 2"""
    hidden_size = ambiguous_cache.shape[2:].numel()
    assert ambiguous_cache.stride()[:2] == (2 * hidden_size, hidden_size)

    attention_spec = AttentionSpec(
        block_size=BLOCK_SIZE, num_kv_heads=1, head_size=8, dtype=torch.float16
    )
    runner_stub = SimpleNamespace(
        cache_config=SimpleNamespace(cache_dtype="auto"),
        _kv_cache_spec_attn_group_iterator=lambda: iter(
            [AttentionGroup(FlashAttentionBackend, ["attn"], attention_spec, 0)]
        ),
    )
    GPUModelRunner._update_hybrid_attention_mamba_layout(
        runner_stub, {"attn": ambiguous_cache}, [BLOCK_SIZE]
    )

    assert ambiguous_cache.stride()[:2] == (hidden_size, 2 * hidden_size), """\
        We expect _update_hybrid_attention_mamba_layout to re-stride the cache from:
        (2, num_blocks) -> (num_blocks, 2), even when num_blocks==2, 
        which was ambiguous before get_kv_cache_block_dim was used"""
```
**EN:** Test case covering `update hybrid attentionMamba layout with num block 2 rewrites stride`. It exercises `torch.empty, shape.numel, AttentionSpec, SimpleNamespace, GPUModelRunner._update_hybrid_attention_mamba_layout, ambiguous_cache.stride`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `update hybrid attentionmamba layout with num block 2 rewrites stride` 的测试用例。 该测试会调用 `torch.empty, shape.numel, AttentionSpec, SimpleNamespace, GPUModelRunner._update_hybrid_attention_mamba_layout, ambiguous_cache.stride`。 代码主体包含 2 个显式断言。

### test_hybrid_block_table_initialization (lines 1225-1276)
```python
def test_hybrid_block_table_initialization():
    """Test hybrid block table with different kernel and kvcache_manager block
    sizes."""
    from vllm.v1.worker.block_table import BlockTable

    # Test configuration: kvcache_manager block size = 32,
    # kernel block size = 16
    block_size = 32
    kernel_block_sizes = [16]
    max_num_reqs = 10
    max_num_blocks_per_req = 20
    max_num_batched_tokens = 512
    cp_kv_cache_interleave_size = 8
    block_table = BlockTable(
        block_size=block_size,
        max_num_reqs=max_num_reqs,
        max_num_blocks_per_req=max_num_blocks_per_req,
    # ... excerpt omitted for brevity ...
    assert block_table.use_hybrid_blocks is True
    assert block_table.block_size == kernel_block_sizes[0]
    assert block_table.blocks_per_kv_block == (
    # Get expected kernel blocks from the implementation for verification.
    expected_kernel_blocks = block_table.map_to_kernel_blocks(
    # Verify block table state
    assert block_table.num_blocks_per_row[req_index] == len(expected_kernel_blocks)
    assert np.array_equal(
        block_table.block_table.np[req_index, : len(expected_kernel_blocks)],
        expected_kernel_blocks,
    )
```
**EN:** Test case covering `hybrid block table initialization`. It exercises `BlockTable, block_table.append_row, block_table.map_to_kernel_blocks, np.array_equal, np.array, len`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid block table initialization` 的测试用例。 该测试会调用 `BlockTable, block_table.append_row, block_table.map_to_kernel_blocks, np.array_equal, np.array, len`。 代码主体包含 5 个显式断言。

### test_input_batch_with_kernel_block_sizes (lines 1279-1313)
```python
def test_input_batch_with_kernel_block_sizes():
    """Test InputBatch initialization with kernel_block_sizes parameter."""
    max_num_reqs = 10
    max_model_len = 512
    max_num_batched_tokens = 512
    device = torch.device(DEVICE_TYPE)
    pin_memory = False
    vocab_size = 50272

    # Test with different kernel block sizes
    block_sizes = [32, 64]
    kernel_block_sizes = [16, 32]
    input_batch = InputBatch(
        max_num_reqs=max_num_reqs,
        max_model_len=max_model_len,
        max_num_batched_tokens=max_num_batched_tokens,
        device=device,
    # ... excerpt omitted for brevity ...
    assert len(input_batch.block_table.block_tables) == len(block_sizes)
        if kv_size != kernel_size:
            assert block_table.use_hybrid_blocks is True
            assert block_table.block_size == kernel_size
        else:
            assert block_table.use_hybrid_blocks is False
```
**EN:** Test case covering `input batch with kernel block sizes`. It exercises `torch.device, InputBatch, enumerate, len, zip`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `input 批处理 with kernel block sizes` 的测试用例。 该测试会调用 `torch.device, InputBatch, enumerate, len, zip`。 代码主体包含 5 个显式断言。

### test_hybrid_cache_integration (lines 1316-1379)
```python
def test_hybrid_cache_integration(default_vllm_config, dist_init):
    """Test hybrid cache architecture integration with GPUModelRunner."""
    # Create a new model runner with hybrid cache configuration
    vllm_config = get_vllm_config()

    # Configure hybrid cache with different kvcache_manager block size
    vllm_config.cache_config.block_size = 32
    model_config = vllm_config.model_config
    num_heads = model_config.get_num_kv_heads(vllm_config.parallel_config)
    head_size = model_config.get_head_size()
    vllm_config.compilation_config.static_forward_context["layer.0"] = Attention(
        num_heads, head_size, 0.1
    )
    runner = GPUModelRunner(vllm_config, DEVICE_TYPE)
    # Initialize KV cache with configuration
    # ... excerpt omitted for brevity ...
    assert block_table.block_size == (
    scheduler_output = _schedule_new_request(req_id)
    # Update states should work with hybrid blocks
    runner._update_states(scheduler_output)
    assert _is_req_scheduled(runner, req_id)
    assert _is_req_state_block_table_match(runner, req_id)
```
**EN:** Test case covering `hybrid cache integration`. Inputs/fixtures: `default_vllm_config, dist_init`. It exercises `get_vllm_config, model_config.get_num_kv_heads, model_config.get_head_size, Attention, GPUModelRunner, FullAttentionSpec`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid 缓存 integration` 的测试用例。 输入或 fixture：`default_vllm_config, dist_init`。 该测试会调用 `get_vllm_config, model_config.get_num_kv_heads, model_config.get_head_size, Attention, GPUModelRunner, FullAttentionSpec`。 代码主体包含 3 个显式断言。

### test_is_uniform_decode (lines 1382-1463)
```python
def test_is_uniform_decode() -> None:
    # Normal
    assert GPUModelRunner._is_uniform_decode(
        max_num_scheduled_tokens=1,
        uniform_decode_query_len=1,
        num_tokens=16,
        num_reqs=16,
    )
    assert not GPUModelRunner._is_uniform_decode(
        max_num_scheduled_tokens=2,
    # ... excerpt omitted for brevity ...
        num_reqs=15,
        force_uniform_decode=False,
```
**EN:** Test case covering `is uniform decode`. It exercises `GPUModelRunner._is_uniform_decode`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `is uniform decode` 的测试用例。 该测试会调用 `GPUModelRunner._is_uniform_decode`。 代码主体包含 12 个显式断言。

### test_mamba_cache_raises_when_max_num_seqs_exceeds_blocks (lines 1470-1568)
```python
def test_mamba_cache_raises_when_max_num_seqs_exceeds_blocks():
    """Test that a ValueError is raised when max_num_seqs exceeds the
    available Mamba cache blocks for hybrid models with FULL cudagraphs.

    See: https://github.com/vllm-project/vllm/issues/34094
    """
    set_random_seed(42)
    update_environment_variables(
        {
            "RANK": "0",
            "LOCAL_RANK": "0",
            "WORLD_SIZE": "1",
            "MASTER_ADDR": "localhost",
            "MASTER_PORT": "12345",
        }
    )
    from tests.utils import ensure_current_vllm_config
    # ... excerpt omitted for brevity ...
        assert fwd_context is not None
        # Force max_num_seqs to exceed num_blocks so the check triggers.
        runner.max_num_reqs = num_blocks + 100
        with pytest.raises(ValueError, match="max_num_seqs"):
            runner.initialize_kv_cache(kv_cache_config)
```
**EN:** Test case covering `Mamba cache raises when max num seqs exceeds blocks`. It exercises `mark.skipif, set_random_seed, update_environment_variables, torch.set_default_dtype, ModelConfig, SchedulerConfig`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `mamba 缓存 raises when max num seqs exceeds blocks` 的测试用例。 该测试会调用 `mark.skipif, set_random_seed, update_environment_variables, torch.set_default_dtype, ModelConfig, SchedulerConfig`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch`.
- **CN:** 外部库：`numpy, pytest, torch`。
- **EN:** vLLM modules under test: `vllm.v1.worker.gpu_model_runner, vllm.config, vllm.distributed.parallel_state, vllm.model_executor.layers.attention, vllm.model_executor.layers.mamba.mamba_mixer2, vllm.platforms, vllm.sampling_params, vllm.utils.mem_constants, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.worker.gpu_model_runner, vllm.config, vllm.distributed.parallel_state, vllm.model_executor.layers.attention, vllm.model_executor.layers.mamba.mamba_mixer2, vllm.platforms, vllm.sampling_params, vllm.utils.mem_constants, ...`。
- **EN:** Local test helpers: `tests.utils`.
- **CN:** 本地测试辅助模块：`tests.utils`。
- **EN:** Standard-library support: `types, unittest.mock`.
- **CN:** 标准库支持：`types, unittest.mock`。
