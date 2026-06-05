# test_priority_scheduler_random.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/core/test_priority_scheduler_random.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `priority scheduler random` behavior and regressions in the v1 stack. / 验证 v1 栈中 `priority 调度器 random` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-22)
```python
import random
import uuid

import pytest

from vllm.config import VllmConfig
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItem,
    PlaceholderRange,
)
from vllm.sampling_params import SamplingParams
from vllm.utils.hashing import get_hash_fn_by_name
from vllm.v1.core.kv_cache_utils import get_request_block_hasher, init_none_hash
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.outputs import DraftTokenIds, ModelRunnerOutput
from vllm.v1.request import Request

from .test_scheduler import create_scheduler_with_priority
from .utils import EOS_TOKEN_ID
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, ...`. Local helpers come from `tests.v1.core.test_scheduler, tests.v1.core.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, ...`。 本地测试辅助逻辑来自 `tests.v1.core.test_scheduler, tests.v1.core.utils`。

### Module state / 模块级状态 (line 24)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _create_random_request (lines 27-85)
```python
def _create_random_request(
    max_tokens_range: tuple[int, int],
    num_tokens_range: tuple[int, int],
    arrival_time_range: tuple[float, float],
    priority_range: tuple[int, int],
    num_mm_item_range: tuple[int, int],
    vllm_config: VllmConfig,
):
    max_tokens = random.randint(*max_tokens_range)
    num_tokens = random.randint(*num_tokens_range)
    priority = random.randint(*priority_range)
    arrival_time = random.uniform(*arrival_time_range)
    num_mm_item = random.randint(*num_mm_item_range)

    mm_positions: list[PlaceholderRange] = []
    for mm_start in sorted(
        random.sample(range(num_tokens), min(num_mm_item, num_tokens))
    ):
    # ... excerpt omitted for brevity ...
        mm_features=mm_features if mm_features else None,
        arrival_time=arrival_time,
        priority=priority,
        block_hasher=block_hasher,
    )
    return request
```
**EN:** Helper function `_create_random_request` encapsulates reusable logic for `random request`. Inputs: `max_tokens_range, num_tokens_range, arrival_time_range, priority_range, num_mm_item_range, vllm_config`. Key calls include `random.randint, random.uniform, sorted, SamplingParams, sampling_params.update_from_generation_config, enumerate`.
**CN:** 辅助函数 `_create_random_request` 封装了与 `random request` 相关的可复用逻辑。 输入参数：`max_tokens_range, num_tokens_range, arrival_time_range, priority_range, num_mm_item_range, vllm_config`。 关键调用包括 `random.randint, random.uniform, sorted, SamplingParams, sampling_params.update_from_generation_config, enumerate`。

### _mock_execute_model (lines 88-111)
```python
def _mock_execute_model(
    scheduler_output: SchedulerOutput, num_output_tokens_range: tuple[int, int]
) -> ModelRunnerOutput:
    request_ids: list[str] = []
    request_ids.extend(req.req_id for req in scheduler_output.scheduled_new_reqs)
    request_ids.extend(scheduler_output.scheduled_cached_reqs.req_ids)
    random.shuffle(request_ids)

    num_output_tokens = [
        random.randint(*num_output_tokens_range) for _ in range(len(request_ids))
    ]
    sampled_token_ids = [
        [random.randint(0, 100) for _ in range(num_tokens)]
        for num_tokens in num_output_tokens
    ]

    return ModelRunnerOutput(
        req_ids=request_ids,
        req_id_to_index={req_id: i for i, req_id in enumerate(request_ids)},
        sampled_token_ids=sampled_token_ids,
        logprobs=None,
        prompt_logprobs_dict={},
        pooler_output=[],
    )
```
**EN:** Helper function `_mock_execute_model` encapsulates reusable logic for `execute model`. Inputs: `scheduler_output, num_output_tokens_range`. Key calls include `request_ids.extend, random.shuffle, ModelRunnerOutput, random.randint, range, len`.
**CN:** 辅助函数 `_mock_execute_model` 封装了与 `execute model` 相关的可复用逻辑。 输入参数：`scheduler_output, num_output_tokens_range`。 关键调用包括 `request_ids.extend, random.shuffle, ModelRunnerOutput, random.randint, range, len`。

### _mock_draft_token_ids (lines 114-140)
```python
def _mock_draft_token_ids(
    scheduler_output: SchedulerOutput,
    num_output_tokens_range: tuple[int, int],
    seen_request_prompt_length: dict[str, int],
) -> DraftTokenIds:
    request_ids: list[str] = []
    sampled_token_ids: list[list[int]] = []
    for request in scheduler_output.scheduled_new_reqs:
        assert request.req_id not in seen_request_prompt_length
        seen_request_prompt_length[request.req_id] = len(request.prompt_token_ids or [])
        if request.num_computed_tokens >= seen_request_prompt_length[request.req_id]:
            num_tokens = random.randint(*num_output_tokens_range)
            request_ids.append(request.req_id)
            sampled_token_ids.append(
                [random.randint(0, 100) for _ in range(num_tokens)]
            )
    for req_id, num_computed_tokens in zip(
        scheduler_output.scheduled_cached_reqs.req_ids,
        scheduler_output.scheduled_cached_reqs.num_computed_tokens,
    ):
        if num_computed_tokens >= seen_request_prompt_length[req_id]:
            num_tokens = random.randint(*num_output_tokens_range)
            request_ids.append(req_id)
            sampled_token_ids.append(
                [random.randint(0, 100) for _ in range(num_tokens)]
            )
    return DraftTokenIds(req_ids=request_ids, draft_token_ids=sampled_token_ids)
```
**EN:** Helper function `_mock_draft_token_ids` encapsulates reusable logic for `draft token ids`. Inputs: `scheduler_output, num_output_tokens_range, seen_request_prompt_length`. Key calls include `zip, DraftTokenIds, len, random.randint, request_ids.append, sampled_token_ids.append`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_mock_draft_token_ids` 封装了与 `draft token ids` 相关的可复用逻辑。 输入参数：`scheduler_output, num_output_tokens_range, seen_request_prompt_length`。 关键调用包括 `zip, DraftTokenIds, len, random.randint, request_ids.append, sampled_token_ids.append`。 其中包含 1 个内部断言，用于保护前置假设。

### _check_valid_scheduler_output (lines 143-173)
```python
def _check_valid_scheduler_output(
    scheduler_output: SchedulerOutput,
    seen_request_ids: set[str],
    seen_mm_hashes: set[str],
):
    for req in scheduler_output.scheduled_new_reqs:
        assert req.req_id not in seen_request_ids
        seen_request_ids.add(req.req_id)
    for req_id in scheduler_output.scheduled_cached_reqs.req_ids:
        assert req_id in seen_request_ids

    req_ids = set[str]()
    req_ids.update(req.req_id for req in scheduler_output.scheduled_new_reqs)
    req_ids.update(scheduler_output.scheduled_cached_reqs.req_ids)

    assert set(scheduler_output.num_scheduled_tokens.keys()) == req_ids
    assert (
        sum(scheduler_output.num_scheduled_tokens.values())
        == scheduler_output.total_num_scheduled_tokens
    )

    assert set(scheduler_output.scheduled_spec_decode_tokens.keys()) <= req_ids
    assert set(scheduler_output.scheduled_encoder_inputs.keys()) <= req_ids

    for req in scheduler_output.scheduled_new_reqs:
        for mm_feature in req.mm_features:
            seen_mm_hashes.add(mm_feature.identifier)
    for mm_hash in scheduler_output.free_encoder_mm_hashes:
        assert mm_hash in seen_mm_hashes

    assert scheduler_output.finished_req_ids <= seen_request_ids
```
**EN:** Helper function `_check_valid_scheduler_output` encapsulates reusable logic for `check valid scheduler output`. Inputs: `scheduler_output, seen_request_ids, seen_mm_hashes`. Key calls include `set, req_ids.update, seen_request_ids.add, sum, num_scheduled_tokens.keys, num_scheduled_tokens.values`. It includes 8 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_check_valid_scheduler_output` 封装了与 `check valid 调度器 output` 相关的可复用逻辑。 输入参数：`scheduler_output, seen_request_ids, seen_mm_hashes`。 关键调用包括 `set, req_ids.update, seen_request_ids.add, sum, num_scheduled_tokens.keys, num_scheduled_tokens.values`。 其中包含 8 个内部断言，用于保护前置假设。

### test_priority_scheduling_blast (lines 188-260)
```python
def test_priority_scheduling_blast(
    enable_prefix_caching: bool,
    num_speculative_tokens: int | None,
    max_input_tokens: int,
    max_output_tokens: int,
    max_num_seqs: int,
    num_blocks: int,
):
    random.seed(42)
    seen_request_prompt_length = dict[str, int]()
    seen_request_ids = set[str]()
    seen_mm_hashes = set[str]()

    scheduler = create_scheduler_with_priority(
        model="Qwen/Qwen2.5-VL-3B-Instruct",
        max_num_seqs=max_num_seqs,
        enable_prefix_caching=enable_prefix_caching,
        num_blocks=num_blocks,
    # ... excerpt omitted for brevity ...
                _mock_draft_token_ids(
                    scheduler_output,
                    (0, num_speculative_tokens),
                    seen_request_prompt_length,
                )
            )
```
**EN:** Parameterized test covering `priority scheduling blast`. Parameter axes: `enable_prefix_caching, num_speculative_tokens`. Inputs/fixtures: `enable_prefix_caching, num_speculative_tokens, max_input_tokens, max_output_tokens, max_num_seqs, num_blocks`. It exercises `mark.parametrize, random.seed, dict, set, create_scheduler_with_priority, range`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `priority scheduling blast` 的测试用例。 参数维度：`enable_prefix_caching, num_speculative_tokens`。 输入或 fixture：`enable_prefix_caching, num_speculative_tokens, max_input_tokens, max_output_tokens, max_num_seqs, num_blocks`。 该测试会调用 `mark.parametrize, random.seed, dict, set, create_scheduler_with_priority, range`。 主要通过 mock、回调或输出检查来完成验证。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.outputs, vllm.v1.request`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.outputs, vllm.v1.request`。
- **EN:** Local test helpers: `tests.v1.core.test_scheduler, tests.v1.core.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.core.test_scheduler, tests.v1.core.utils`。
- **EN:** Standard-library support: `random, uuid`.
- **CN:** 标准库支持：`random, uuid`。
