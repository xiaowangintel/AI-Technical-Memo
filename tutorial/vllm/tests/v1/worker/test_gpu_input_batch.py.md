# test_gpu_input_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/worker/test_gpu_input_batch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `GPU input batch` behavior and regressions in the v1 stack. / 验证 v1 栈中 `gpu input 批处理` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-20)
```python
import inspect
from collections.abc import Sequence

import numpy as np
import pytest
import torch

from vllm.platforms import current_platform
from vllm.sampling_params import SamplingParams
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.utils.torch_utils import make_tensor_with_pad
from vllm.v1.pool.metadata import PoolingMetadata
from vllm.v1.sample.logits_processor import LogitsProcessors
from vllm.v1.sample.metadata import SamplingMetadata
from vllm.v1.utils import CpuGpuBuffer
from vllm.v1.worker.block_table import BlockTable, MultiGroupBlockTable
from vllm.v1.worker.gpu_input_batch import CachedRequestState, InputBatch
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch`. vLLM modules under test include `vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.pool.metadata, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.pool.metadata, ...`。

### Module state / 模块级状态 (lines 22-27)
```python
VOCAB_SIZE = 1024
NUM_OUTPUT_TOKENS = 20
MAX_PROMPT_SIZE = 100
DEVICE_TYPE = current_platform.device_type
DEVICES = [f"{DEVICE_TYPE}:{i}" for i in range(min(current_platform.device_count(), 2))]
MAX_NUM_PROMPT_TOKENS = 64
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `VOCAB_SIZE, NUM_OUTPUT_TOKENS, MAX_PROMPT_SIZE, DEVICE_TYPE, DEVICES, MAX_NUM_PROMPT_TOKENS`. Shared setup calls include `range, min, current_platform.device_count`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`VOCAB_SIZE, NUM_OUTPUT_TOKENS, MAX_PROMPT_SIZE, DEVICE_TYPE, DEVICES, MAX_NUM_PROMPT_TOKENS`。 共享初始化调用包括 `range, min, current_platform.device_count`。

### _compare_objs (lines 30-64)
```python
def _compare_objs(obj1, obj2, skip: Sequence = ("logitsprocs", "batch_update_builder")):
    attrs = inspect.getmembers(obj1, lambda a: not (inspect.isroutine(a)))
    attr_names = set(
        [a[0] for a in attrs if not (a[0].startswith("__") and a[0].endswith("__"))]
    )
    for attr_name in attr_names:
        if attr_name in skip:
            continue

        a = getattr(obj1, attr_name)
        b = getattr(obj2, attr_name)
        is_same = False
        if isinstance(a, torch.Tensor):
            if a.numel() == 0 or b.numel() == 0:
                is_same = a.numel() == 0 and b.numel() == 0
            elif torch.allclose(a, b):
                is_same = True
    # ... excerpt omitted for brevity ...
            is_same = True
        elif isinstance(a, CpuGpuBuffer):
            is_same = np.allclose(a.np, b.np) and torch.allclose(a.gpu, b.gpu)
        assert is_same, (
            f"Attribute {attr_name} is different in {obj1} and {obj2}: {a} != {b}"
        )
```
**EN:** Helper function `_compare_objs` encapsulates reusable logic for `compare objs`. Inputs: `obj1, obj2, skip`. Key calls include `inspect.getmembers, set, getattr, isinstance, inspect.isroutine, torch.allclose`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_compare_objs` 封装了与 `compare objs` 相关的可复用逻辑。 输入参数：`obj1, obj2, skip`。 关键调用包括 `inspect.getmembers, set, getattr, isinstance, inspect.isroutine, torch.allclose`。 其中包含 1 个内部断言，用于保护前置假设。

### _remove_requests (lines 67-85)
```python
def _remove_requests(
    input_batch: InputBatch, batch_size: int, reqs: list[CachedRequestState]
) -> set[str]:
    """
    Remove some requests randomly from the batch and returns
    set of request removed
    """

    num_reqs_to_remove = np.random.randint(0, batch_size)
    req_indices_to_remove: set[int] = set()
    for _ in range(num_reqs_to_remove):
        req_index_to_remove = np.random.randint(0, batch_size)
        req_indices_to_remove.add(req_index_to_remove)

    req_ids_to_remove: set[str] = set()
    for index in req_indices_to_remove:
        input_batch.remove_request(reqs[index].req_id)
        req_ids_to_remove.add(reqs[index].req_id)
    return req_ids_to_remove
```
**EN:** Helper function `_remove_requests` encapsulates reusable logic for `remove requests`. Inputs: `input_batch, batch_size, reqs`. Key calls include `random.randint, set, range, req_indices_to_remove.add, input_batch.remove_request, req_ids_to_remove.add`.
**CN:** 辅助函数 `_remove_requests` 封装了与 `remove requests` 相关的可复用逻辑。 输入参数：`input_batch, batch_size, reqs`。 关键调用包括 `random.randint, set, range, req_indices_to_remove.add, input_batch.remove_request, req_ids_to_remove.add`。

### _construct_expected_sampling_metadata (lines 88-180)
```python
def _construct_expected_sampling_metadata(
    reqs: list[CachedRequestState],
    req_ids_retained: set[int],
    req_id_index_in_input_batch: dict[str, int],
    device: torch.device,
) -> SamplingMetadata:
    """
    Constructs and returns the expected SamplingMetadata for this
    batch.
    num_reqs = len(req_ids_retained)
    output_token_ids: list[list[int]] = [list() for _ in range(num_reqs)]
    prompt_token_ids: list[list[int]] = [list() for _ in range(num_reqs)]
    presence_penalties = [0.0 for _ in range(num_reqs)]
    frequency_penalties = [0.0 for _ in range(num_reqs)]
    repetition_penalties = [1.0 for _ in range(num_reqs)]
    top_k = [0 for _ in range(num_reqs)]
    top_p = [0.0 for _ in range(num_reqs)]
    # ... excerpt omitted for brevity ...
    return SamplingMetadata(
            and all(x == 1 for x in repetition_penalties)
        ),
        allowed_token_ids_mask=allowed_token_ids_mask,
        bad_words_token_ids=bad_words_token_ids,
        logitsprocs=LogitsProcessors(),
    )
```
**EN:** Helper function `_construct_expected_sampling_metadata` encapsulates reusable logic for `construct expected sampling metadata`. Inputs: `reqs, req_ids_retained, req_id_index_in_input_batch, device`. Key calls include `len, torch.zeros, SamplingMetadata, list, range, torch.tensor`.
**CN:** 辅助函数 `_construct_expected_sampling_metadata` 封装了与 `construct expected 采样 metadata` 相关的可复用逻辑。 输入参数：`reqs, req_ids_retained, req_id_index_in_input_batch, device`。 关键调用包括 `len, torch.zeros, SamplingMetadata, list, range, torch.tensor`。

### _create_sampling_params (lines 183-195)
```python
def _create_sampling_params():
    return SamplingParams(
        top_k=np.random.randint(1, 10),
        top_p=np.random.uniform(0.0, 1.0),
        presence_penalty=np.random.uniform(-2.0, 2.0),
        repetition_penalty=np.random.uniform(0.0, 2.0),
        frequency_penalty=np.random.uniform(-2.0, 2.0),
        min_tokens=np.random.randint(1, 10),
        stop_token_ids=[
            np.random.randint(0, VOCAB_SIZE) for _ in range(np.random.randint(10))
        ],
        logit_bias={0: np.random.uniform(-3.0, 3.0)},
    )
```
**EN:** Helper function `_create_sampling_params` encapsulates reusable logic for `sampling params`. Key calls include `SamplingParams, random.randint, random.uniform, range`.
**CN:** 辅助函数 `_create_sampling_params` 封装了与 `采样 params` 相关的可复用逻辑。 关键调用包括 `SamplingParams, random.randint, random.uniform, range`。

### _construct_cached_request_state (lines 198-217)
```python
def _construct_cached_request_state(req_id_suffix: int):
    prompt_token_ids = [
        np.random.randint(0, VOCAB_SIZE)
        for _ in range(np.random.randint(0, MAX_PROMPT_SIZE))
    ]
    output_token_ids = [
        np.random.randint(0, VOCAB_SIZE)
        for _ in range(np.random.randint(0, NUM_OUTPUT_TOKENS))
    ]
    return CachedRequestState(
        req_id=f"req_id_{req_id_suffix}",
        prompt_token_ids=prompt_token_ids,
        sampling_params=_create_sampling_params(),
        pooling_params=None,
        mm_features=[],
        block_ids=([],),
        generator=None,
        num_computed_tokens=len(output_token_ids),
        output_token_ids=output_token_ids,
    )
```
**EN:** Helper function `_construct_cached_request_state` encapsulates reusable logic for `construct cached request state`. Inputs: `req_id_suffix`. Key calls include `CachedRequestState, random.randint, range, _create_sampling_params, len`.
**CN:** 辅助函数 `_construct_cached_request_state` 封装了与 `construct cached request state` 相关的可复用逻辑。 输入参数：`req_id_suffix`。 关键调用包括 `CachedRequestState, random.randint, range, _create_sampling_params, len`。

### test_sampling_metadata_in_input_batch (lines 222-311)
```python
def test_sampling_metadata_in_input_batch(device: str, batch_size: int):
    """
    Tests the logic for managing sampling metadata in the InputBatch.

    This test involves adding a set of requests to the InputBatch,
    followed by removing a subset of them. Afterward, the batch is compacted,
    and the `make_sampling_metadata` method is invoked on the batch. The
    output of `make_sampling_metadata` is then compared against the expected
    results to ensure correctness.
    Note: Ignore logits processor logic, which is tested separately
    input_batch: InputBatch = InputBatch(
        max_num_reqs=batch_size,
        max_model_len=1024,
        max_num_batched_tokens=1024,
        device=torch.device(device),
        pin_memory=is_pin_memory_available(),
    # ... excerpt omitted for brevity ...
        assert req_index == assigned_req_index
    # Create expected output.
    expected_sampling_metadata = _construct_expected_sampling_metadata(
        return (t1 is None and t2 is None) or (
    # Assert the actual and expected output.
    assert torch.allclose(
            sampling_metadata.allowed_token_ids_mask,
        )
    assert (
        expected_sampling_metadata.bad_words_token_ids
        == sampling_metadata.bad_words_token_ids
    )
```
**EN:** Parameterized test covering `sampling metadata in input batch`. Parameter axes: `device, batch_size`. Inputs/fixtures: `device, batch_size`. It exercises `mark.parametrize, InputBatch, range, _remove_requests, input_batch.condense, input_batch._make_sampling_metadata`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `采样 metadata in input 批处理` 的测试用例。 参数维度：`device, batch_size`。 输入或 fixture：`device, batch_size`。 该测试会调用 `mark.parametrize, InputBatch, range, _remove_requests, input_batch.condense, input_batch._make_sampling_metadata`。 代码主体包含 12 个显式断言。

### test_swap_states_in_input_batch (lines 317-378)
```python
def test_swap_states_in_input_batch(device: str, batch_size: int, swap_list: list):
    """
    Tests the logic for managing sampling metadata in the InputBatch.

    This test involves adding a set of requests to the InputBatch,
    followed by removing a subset of them. Afterward, the batch is compacted,
    and the `make_sampling_metadata` method is invoked on the batch. The
    output of `make_sampling_metadata` is then compared against the expected
    results to ensure correctness.
    Note: Ignore logits processor logic, which is tested separately
    input_batch: InputBatch = InputBatch(
        max_num_reqs=batch_size,
        max_model_len=1024,
        max_num_batched_tokens=1024,
        device=torch.device(device),
        pin_memory=is_pin_memory_available(),
    # ... excerpt omitted for brevity ...
        assert assigned_req_index == req_index
    input_batch.refresh_metadata()
    ref_input_batch.refresh_metadata()
    _compare_objs(input_batch, ref_input_batch)
```
**EN:** Parameterized test covering `swap states in input batch`. Parameter axes: `device, batch_size, swap_list`. Inputs/fixtures: `device, batch_size, swap_list`. It exercises `mark.parametrize, InputBatch, range, reqs.copy, input_batch.refresh_metadata, ref_input_batch.refresh_metadata`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `swap states in input 批处理` 的测试用例。 参数维度：`device, batch_size, swap_list`。 输入或 fixture：`device, batch_size, swap_list`。 该测试会调用 `mark.parametrize, InputBatch, range, reqs.copy, input_batch.refresh_metadata, ref_input_batch.refresh_metadata`。 代码主体包含 2 个显式断言。

### _construct_pooling_request (lines 381-398)
```python
def _construct_pooling_request(req_id_suffix: int, pooling_params=None):
    from vllm.pooling_params import PoolingParams

    prompt_token_ids = [
        np.random.randint(0, VOCAB_SIZE)
        for _ in range(np.random.randint(10, MAX_PROMPT_SIZE))
    ]
    return CachedRequestState(
        req_id=f"pool_req_{req_id_suffix}",
        prompt_token_ids=prompt_token_ids,
        sampling_params=None,
        pooling_params=pooling_params or PoolingParams(task="classify"),
        mm_features=[],
        block_ids=([],),
        generator=None,
        num_computed_tokens=0,
        output_token_ids=[],
    )
```
**EN:** Helper function `_construct_pooling_request` encapsulates reusable logic for `construct pooling request`. Inputs: `req_id_suffix, pooling_params`. Key calls include `CachedRequestState, random.randint, range, PoolingParams`.
**CN:** 辅助函数 `_construct_pooling_request` 封装了与 `construct pooling request` 相关的可复用逻辑。 输入参数：`req_id_suffix, pooling_params`。 关键调用包括 `CachedRequestState, random.randint, range, PoolingParams`。

### test_pooling_prompt_lens_not_aliased (lines 402-440)
```python
def test_pooling_prompt_lens_not_aliased(device: str):
    """Verify that prompt_lens in PoolingMetadata does not share memory
    with the internal num_prompt_tokens pinned buffer. Guards against possible
    non-determinism in pooling metadata due to mutations to the internal buffer.
    """
    batch_size = 4
    input_batch = InputBatch(
        max_num_reqs=batch_size * 2,
        max_model_len=MAX_PROMPT_SIZE + NUM_OUTPUT_TOKENS,
        max_num_batched_tokens=batch_size * (MAX_PROMPT_SIZE + NUM_OUTPUT_TOKENS),
        device=torch.device(device),
        pin_memory=is_pin_memory_available(),
        vocab_size=VOCAB_SIZE,
        block_sizes=[16],
        kernel_block_sizes=[16],
        is_pooling_model=True,
    )

    # ... excerpt omitted for brevity ...
    # prompt_lens must be unaffected by the mutation
    assert torch.equal(metadata.prompt_lens, prompt_lens_snapshot), (
        "prompt_lens shares memory with internal pinned buffer; "
        "mutations to num_prompt_tokens_cpu_tensor corrupted prompt_lens. "
        f"Expected {prompt_lens_snapshot}, got {metadata.prompt_lens}"
```
**EN:** Parameterized test covering `pooling prompt lens not aliased`. Parameter axes: `device`. Inputs/fixtures: `device`. It exercises `mark.parametrize, InputBatch, range, input_batch.refresh_metadata, input_batch.get_pooling_metadata, prompt_lens.clone`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `pooling prompt lens not aliased` 的测试用例。 参数维度：`device`。 输入或 fixture：`device`。 该测试会调用 `mark.parametrize, InputBatch, range, input_batch.refresh_metadata, input_batch.get_pooling_metadata, prompt_lens.clone`。 代码主体包含 1 个显式断言。

### test_pooling_metadata_token_id_buffers (lines 450-485)
```python
def test_pooling_metadata_token_id_buffers(
    pooling_params: dict[str, object],
    expect_device_prompt_token_ids: bool,
    expect_cpu_prompt_token_ids: bool,
):
    from vllm.pooling_params import PoolingParams

    input_batch = InputBatch(
        max_num_reqs=1,
        max_model_len=MAX_PROMPT_SIZE + NUM_OUTPUT_TOKENS,
        max_num_batched_tokens=MAX_PROMPT_SIZE + NUM_OUTPUT_TOKENS,
        device=torch.device("cpu"),
        pin_memory=False,
        vocab_size=VOCAB_SIZE,
        block_sizes=[16],
        kernel_block_sizes=[16],
        is_pooling_model=True,
    )
    # ... excerpt omitted for brevity ...
        assert input_batch.sampling_metadata.prompt_token_ids is not None
        assert metadata.prompt_token_ids is not None
        assert metadata.get_prompt_token_ids()[0].tolist() == req.prompt_token_ids
        assert input_batch.sampling_metadata.prompt_token_ids is None
        assert metadata.prompt_token_ids is None
    if expect_cpu_prompt_token_ids:
        assert metadata.prompt_token_ids_cpu is not None
        assert metadata.get_prompt_token_ids_cpu()[0].tolist() == req.prompt_token_ids
    else:
        assert metadata.prompt_token_ids_cpu is None
```
**EN:** Test case covering `pooling metadata token id buffers`. Inputs/fixtures: `pooling_params, expect_device_prompt_token_ids, expect_cpu_prompt_token_ids`. It exercises `mark.parametrize, InputBatch, _construct_pooling_request, input_batch.add_request, input_batch.refresh_metadata, input_batch.get_pooling_metadata`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `pooling metadata token id buffers` 的测试用例。 输入或 fixture：`pooling_params, expect_device_prompt_token_ids, expect_cpu_prompt_token_ids`。 该测试会调用 `mark.parametrize, InputBatch, _construct_pooling_request, input_batch.add_request, input_batch.refresh_metadata, input_batch.get_pooling_metadata`。 代码主体包含 8 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch`.
- **CN:** 外部库：`numpy, pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.pool.metadata, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.utils, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.pool.metadata, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.utils, ...`。
- **EN:** Standard-library support: `inspect, collections.abc`.
- **CN:** 标准库支持：`inspect, collections.abc`。
