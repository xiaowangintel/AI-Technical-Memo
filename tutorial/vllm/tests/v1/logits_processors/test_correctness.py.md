# test_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/logits_processors/test_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `correctness` behavior and regressions in the v1 stack. / 验证 v1 栈中 `correctness` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-39)
```python
import random
from collections.abc import Callable
from typing import NamedTuple, TypeAlias

import numpy as np
import pytest
import torch
from tests.utils import create_new_process_for_each_test
from tests.v1.sample.utils import (
    LogitsprocsTestFakes,
    create_fake_logits,
    create_penalty_tensor,
    create_prompt_tokens_tensor,
    fake_apply_logitsprocs,
    fake_update_logitsprocs_state,
)
from vllm.config import VllmConfig
# ... excerpt omitted for brevity ...
from vllm.v1.sample.metadata import SamplingMetadata
from vllm.v1.sample.thinking_budget_state import (
    ThinkingBudgetStateHolder,
    maybe_create_thinking_budget_state_holder,
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch`. vLLM modules under test include `vllm.config, vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.v1.sample.logits_processor, ...`. Local helpers come from `tests.utils, tests.v1.sample.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.v1.sample.logits_processor, ...`。 本地测试辅助逻辑来自 `tests.utils, tests.v1.sample.utils`。

### Module state / 模块级状态 (lines 41-63)
```python
PIN_MEMORY_AVAILABLE = is_pin_memory_available()
MAX_NUM_REQS = 256
VOCAB_SIZE = 1024
NUM_OUTPUT_TOKENS = 20
DEVICE_TYPE = current_platform.device_type
DEVICES = [
    f"{DEVICE_TYPE}:{i}"
    for i in range(1 if current_platform.device_count() == 1 else 2)
]
MAX_NUM_PROMPT_TOKENS = 64
MIN_TOKENS_LEN_THRESHOLD = 5
REQS_PER_LOGITPROC = 50
STR_NO_LOGITPROC = "none"
# Thinking budget uses ``ThinkingBudgetStateHolder`` (not a logits processor).
STR_THINKING_BUDGET = "thinking_budget"

# Thinking token budget testing constants
THINKING_TOKEN_BUDGET = 5
THINK_START_TOKEN_ID = 999
THINK_END_TOKEN_ID = 998

# LogitsProcessor subclass or "none"
LogitprocType: TypeAlias = type[LogitsProcessor] | str
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `PIN_MEMORY_AVAILABLE, MAX_NUM_REQS, VOCAB_SIZE, NUM_OUTPUT_TOKENS, DEVICE_TYPE, DEVICES, ...`. Shared setup calls include `is_pin_memory_available, range, current_platform.device_count`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`PIN_MEMORY_AVAILABLE, MAX_NUM_REQS, VOCAB_SIZE, NUM_OUTPUT_TOKENS, DEVICE_TYPE, DEVICES, ...`。 共享初始化调用包括 `is_pin_memory_available, range, current_platform.device_count`。

### LogitsProcsRequestParams (lines 66-99)
```python
class LogitsProcsRequestParams:
    """Encapsulates key params for a single request in a batch.

    Params can be customized based on the enabled logitproc
    """
    workload_index: int
    logitproc_type: LogitprocType  # Logitproc enabled, specified by str id
    out_tokens: list[int]  # Output tokens required for min tokens test
    prompt_tokens: list[int]  # Dummy prompt tokens placeholder
    params: SamplingParams  # Settings customized for logitproc
    def __init__(self, workload_index: int, logitproc_type: LogitprocType):
        self.workload_index = workload_index
        self.logitproc_type = logitproc_type
        # Number of output tokens is randomly 0 or twice the min-tokens
        # threshold which will be used in testing.
        # Generate diverse random tokens for all processors (more realistic)
    # ... excerpt omitted for brevity ...
        self.params = _sampling_params_from_logitproc(logitproc_type)
    def __str__(self):
        """For debugging"""
        summ = ", ".join(f"{k}={v}" for k, v in vars(self).items())
        return f"MyClass({summ})"
```
**EN:** Class `LogitsProcsRequestParams` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `LogitsProcsRequestParams` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

### MockReasoningConfig (lines 102-107)
```python
class MockReasoningConfig:
    """Minimal reasoning config for ``ThinkingBudgetStateHolder`` tests."""

    reasoning_start_token_ids = [THINK_START_TOKEN_ID]
    reasoning_end_token_ids = [THINK_END_TOKEN_ID]
    enabled = True
```
**EN:** Class `MockReasoningConfig` groups 0 test method(s).
**CN:** 类 `MockReasoningConfig` 组织了 0 个测试方法。

### _generate_fake_sampling_metadata (lines 110-171)
```python
def _generate_fake_sampling_metadata(
    num_output_tokens: int,
    batch_size: int,
    vocab_size: int,
    device: torch.device,
) -> SamplingMetadata:
    """Generate fake sampling metadata with fake logitsprocs"""
    output_token_ids: list[list[int]] = []
    prompt_token_ids: list[list[int]] = []
    for _ in range(batch_size):
        output_token_ids.append(
            np.random.randint(0, vocab_size, size=num_output_tokens).tolist()
        )
        prompt_token_ids.append(
            np.random.randint(
                0, vocab_size, size=np.random.randint(1, MAX_NUM_PROMPT_TOKENS)
            ).tolist()
    # ... excerpt omitted for brevity ...
        allowed_token_ids_mask=None,
        bad_words_token_ids={},
        logitsprocs=logitsprocs,
        thinking_budget_state_holder=thinking_holder,
    )
    return fake_sampling_metadata
```
**EN:** Helper function `_generate_fake_sampling_metadata` encapsulates reusable logic for `generate fake sampling metadata`. Inputs: `num_output_tokens, batch_size, vocab_size, device`. Key calls include `range, VllmConfig, MockReasoningConfig, build_logitsprocs, maybe_create_thinking_budget_state_holder, SamplingMetadata`.
**CN:** 辅助函数 `_generate_fake_sampling_metadata` 封装了与 `generate fake 采样 metadata` 相关的可复用逻辑。 输入参数：`num_output_tokens, batch_size, vocab_size, device`。 关键调用包括 `range, VllmConfig, MockReasoningConfig, build_logitsprocs, maybe_create_thinking_budget_state_holder, SamplingMetadata`。

### _generate_test_fakes (lines 174-187)
```python
def _generate_test_fakes(batch_size: int, device: str) -> LogitsprocsTestFakes:
    """Generate fake logits and sampling metadata"""
    fake_logits = create_fake_logits(batch_size, VOCAB_SIZE)
    # Create one dominant token per batch, to support min-p test
    for i in range(batch_size):
        fake_logits[i, 0] = 10.0  # High logit for first token
        fake_logits[i, 1:] = 1e-2  # Others remain low
    sampling_metadata = _generate_fake_sampling_metadata(
        NUM_OUTPUT_TOKENS, batch_size, VOCAB_SIZE, torch.device(device)
    )
    return LogitsprocsTestFakes(
        logits=fake_logits,
        sampling_metadata=sampling_metadata,
    )
```
**EN:** Helper function `_generate_test_fakes` encapsulates reusable logic for `generate test fakes`. Inputs: `batch_size, device`. Key calls include `create_fake_logits, range, _generate_fake_sampling_metadata, LogitsprocsTestFakes, torch.device`.
**CN:** 辅助函数 `_generate_test_fakes` 封装了与 `generate test fakes` 相关的可复用逻辑。 输入参数：`batch_size, device`。 关键调用包括 `create_fake_logits, range, _generate_fake_sampling_metadata, LogitsprocsTestFakes, torch.device`。

### _sampling_params_from_logitproc (lines 190-196)
```python
def _sampling_params_from_logitproc(logitproc_type: LogitprocType) -> SamplingParams:
    """Customize request SamplingParams for a specified logitproc"""
    # SamplingParams for req with no logitproc
    kwargs = {"min_p": 0.0, "logit_bias": None, "min_tokens": 0}
    if fxn := logitsprocs_test_mapping[logitproc_type].gen_request_fxn:
        fxn(kwargs)
    return SamplingParams(**kwargs)
```
**EN:** Helper function `_sampling_params_from_logitproc` encapsulates reusable logic for `sampling params from logitproc`. Inputs: `logitproc_type`. Key calls include `SamplingParams, fxn`.
**CN:** 辅助函数 `_sampling_params_from_logitproc` 封装了与 `采样 params from logitproc` 相关的可复用逻辑。 输入参数：`logitproc_type`。 关键调用包括 `SamplingParams, fxn`。

### _generate_mixed_logitsprocs_batch_params (lines 199-231)
```python
def _generate_mixed_logitsprocs_batch_params(
    reqs_per_logitproc: int,
    logitsprocs_types: list[LogitprocType],
) -> list[LogitsProcsRequestParams]:
    """Define key params for a batch of requests with a different
    logitproc enabled per request.

    The batch will have `reqs_per_logitproc` repeats for all
    `logitsprocs_types` under test, including the case where
    no logitsproc is enabled. The batch is randomly shuffled. The
    size of the batch is `reqs_per_logitproc` times
    `n = len(logitsprocs_types)`
    Args:
      reqs_per_logitproc: number of requests using each logitproc
      logitsprocs_types: logitsprocs under test
    Returns:
    # ... excerpt omitted for brevity ...
    return [
        LogitsProcsRequestParams(
            workload_index=idx,
            logitproc_type=logitsprocs_types[pdx // reqs_per_logitproc],
        )
        for idx, pdx in enumerate(batch_perm)
    ]
```
**EN:** Helper function `_generate_mixed_logitsprocs_batch_params` encapsulates reusable logic for `generate mixed logitsprocs batch params`. Inputs: `reqs_per_logitproc, logitsprocs_types`. Key calls include `random.sample, len, range, LogitsProcsRequestParams, enumerate`.
**CN:** 辅助函数 `_generate_mixed_logitsprocs_batch_params` 封装了与 `generate mixed logitsprocs 批处理 params` 相关的可复用逻辑。 输入参数：`reqs_per_logitproc, logitsprocs_types`。 关键调用包括 `random.sample, len, range, LogitsProcsRequestParams, enumerate`。

### _raise_error_invalid (lines 234-246)
```python
def _raise_error_invalid(
    msg_suffix: str,
    batch_index: int,
    request_params: LogitsProcsRequestParams,
    step_idx: int,
    err_cls: type[Exception] = ValueError,
) -> None:
    raise err_cls(
        f"Validation failed for step={step_idx}, "
        f"batch_index={batch_index}, "
        f"workload_index={request_params.workload_index}, "
        f"req_params={request_params}. Reason: {msg_suffix}"
    )
```
**EN:** Helper function `_raise_error_invalid` encapsulates reusable logic for `raise error invalid`. Inputs: `msg_suffix, batch_index, request_params, step_idx, err_cls`. Key calls include `err_cls`.
**CN:** 辅助函数 `_raise_error_invalid` 封装了与 `raise error invalid` 相关的可复用逻辑。 输入参数：`msg_suffix, batch_index, request_params, step_idx, err_cls`。 关键调用包括 `err_cls`。

### _logit_bias_params (lines 249-253)
```python
def _logit_bias_params(kwargs: dict) -> None:
    """Logit bias config"""
    kwargs["logit_bias"] = {
        random.randint(0, VOCAB_SIZE - 1): random.choice([-0.1, 0.2])
    }
```
**EN:** Helper function `_logit_bias_params` encapsulates reusable logic for `logit bias params`. Inputs: `kwargs`. Key calls include `random.randint, random.choice`.
**CN:** 辅助函数 `_logit_bias_params` 封装了与 `logit bias params` 相关的可复用逻辑。 输入参数：`kwargs`。 关键调用包括 `random.randint, random.choice`。

### _logit_bias_validate (lines 256-296)
```python
def _logit_bias_validate(
    test_fakes: LogitsprocsTestFakes,
    persistent_batch: list[LogitsProcsRequestParams],
    logits_new: torch.Tensor,
    batch_index: int,
    request_params: LogitsProcsRequestParams,
    step_idx: int,
) -> None:
    """Validate logit bias logitproc applied correctly"""
    logit_bias = request_params.params.logit_bias
    logits_old = test_fakes.logits[persistent_batch[batch_index].workload_index].cpu()
    logits_new = logits_new[batch_index].cpu()
    for token_id in range(VOCAB_SIZE):
        logit_old_value = logits_old[token_id]
        logit_new_value = logits_new[token_id]
        if token_id in logit_bias:
            bias_value = logit_bias[token_id]
            exp_value = bias_value + logit_old_value
    # ... excerpt omitted for brevity ...
                        f"does not match expected value {exp_value} "
                        f"does not match expected value {logit_old_value}"
                    ),
                    batch_index=batch_index,
                    request_params=request_params,
                    step_idx=step_idx,
                )
```
**EN:** Helper function `_logit_bias_validate` encapsulates reusable logic for `logit bias validate`. Inputs: `test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`. Key calls include `logits.cpu, logits_new.cpu, range, pytest.approx, _raise_error_invalid`.
**CN:** 辅助函数 `_logit_bias_validate` 封装了与 `logit bias validate` 相关的可复用逻辑。 输入参数：`test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`。 关键调用包括 `logits.cpu, logits_new.cpu, range, pytest.approx, _raise_error_invalid`。

### _min_p_params (lines 299-301)
```python
def _min_p_params(kwargs: dict) -> None:
    """Min-p logitproc config"""
    kwargs["min_p"] = 0.1
```
**EN:** Helper function `_min_p_params` encapsulates reusable logic for `min p params`. Inputs: `kwargs`.
**CN:** 辅助函数 `_min_p_params` 封装了与 `min p params` 相关的可复用逻辑。 输入参数：`kwargs`。

### _min_p_validate (lines 304-342)
```python
def _min_p_validate(
    test_fakes: LogitsprocsTestFakes,
    persistent_batch: list[LogitsProcsRequestParams],
    logits_new: torch.Tensor,
    batch_index: int,
    request_params: LogitsProcsRequestParams,
    step_idx: int,
) -> None:
    """Validate min-p logitproc applied correctly"""
    for token_id in range(VOCAB_SIZE):
        logits_for_token = logits_new[batch_index][token_id]
        if token_id == 0:
            # Dominant token should always be unmasked
            if logits_for_token == -float("inf"):
                _raise_error_invalid(
                    msg_suffix="Invalid: dominant token 0 masked (-inf)",
                    batch_index=batch_index,
                    request_params=request_params,
    # ... excerpt omitted for brevity ...
                    _raise_error_invalid(
                        msg_suffix=f"Invalid: token {token_id} masked when min_p=0.0",
                        batch_index=batch_index,
                        request_params=request_params,
                        step_idx=step_idx,
                    )
```
**EN:** Helper function `_min_p_validate` encapsulates reusable logic for `min p validate`. Inputs: `test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`. Key calls include `range, _raise_error_invalid, float`.
**CN:** 辅助函数 `_min_p_validate` 封装了与 `min p validate` 相关的可复用逻辑。 输入参数：`test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`。 关键调用包括 `range, _raise_error_invalid, float`。

### _min_tokens_params (lines 345-351)
```python
def _min_tokens_params(kwargs: dict) -> None:
    """Min-tokens logitproc config"""
    kwargs["min_tokens"] = MIN_TOKENS_LEN_THRESHOLD
    kwargs["stop_token_ids"] = [
        np.random.randint(0, VOCAB_SIZE - 1)
        for _ in range(np.random.randint(0, VOCAB_SIZE))
    ]
```
**EN:** Helper function `_min_tokens_params` encapsulates reusable logic for `min tokens params`. Inputs: `kwargs`. Key calls include `random.randint, range`.
**CN:** 辅助函数 `_min_tokens_params` 封装了与 `min tokens params` 相关的可复用逻辑。 输入参数：`kwargs`。 关键调用包括 `random.randint, range`。

### _min_tokens_validate (lines 354-448)
```python
def _min_tokens_validate(
    test_fakes: LogitsprocsTestFakes,
    persistent_batch: list[LogitsProcsRequestParams],
    logits_new: torch.Tensor,
    batch_index: int,
    request_params: LogitsProcsRequestParams,
    step_idx: int,
) -> None:
    """Validate min-tokens logitsproc applied correctly"""
    ref_num_out_tokens = len(request_params.out_tokens)
    min_reached = ref_num_out_tokens >= MIN_TOKENS_LEN_THRESHOLD
    ref_all_stop_token_ids = request_params.params.all_stop_token_ids
    mt_lp: MinTokensLogitsProcessor = next(
        test_fakes.get_logitsprocs_by_cls(MinTokensLogitsProcessor)
    )
    assert isinstance(mt_lp, MinTokensLogitsProcessor)
    min_tok = mt_lp.min_toks.get(batch_index, None)

    # ... excerpt omitted for brevity ...
                        f"is (output len={ref_num_out_tokens})"
                    ),
                    batch_index=batch_index,
                    request_params=request_params,
                    step_idx=step_idx,
                )
```
**EN:** Helper function `_min_tokens_validate` encapsulates reusable logic for `min tokens validate`. Inputs: `test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`. Key calls include `len, next, isinstance, min_toks.get, range, test_fakes.get_logitsprocs_by_cls`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_min_tokens_validate` 封装了与 `min tokens validate` 相关的可复用逻辑。 输入参数：`test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`。 关键调用包括 `len, next, isinstance, min_toks.get, range, test_fakes.get_logitsprocs_by_cls`。 其中包含 1 个内部断言，用于保护前置假设。

### _thinking_budget_params (lines 451-453)
```python
def _thinking_budget_params(kwargs: dict) -> None:
    """Set SamplingParams kwargs for thinking token budget tests"""
    kwargs["thinking_token_budget"] = THINKING_TOKEN_BUDGET
```
**EN:** Helper function `_thinking_budget_params` encapsulates reusable logic for `thinking budget params`. Inputs: `kwargs`.
**CN:** 辅助函数 `_thinking_budget_params` 封装了与 `thinking budget params` 相关的可复用逻辑。 输入参数：`kwargs`。

### _thinking_budget_validate (lines 456-542)
```python
def _thinking_budget_validate(
    test_fakes: LogitsprocsTestFakes,
    persistent_batch: list[LogitsProcsRequestParams],
    logits_new: torch.Tensor,
    batch_index: int,
    request_params: LogitsProcsRequestParams,
    step_idx: int,
) -> None:
    """Validate ``ThinkingBudgetStateHolder`` thinking-budget behavior.

    State is keyed by **batch slot** (same index space as logits rows), matching
    ``sync_batch`` / sampler integration (see PR #34668 discussion).
    """
    holder = test_fakes.sampling_metadata.thinking_budget_state_holder
    assert holder is not None
    state = holder._state.get(batch_index)
    params = request_params.params
    # ... excerpt omitted for brevity ...
        expected_budget = params.thinking_token_budget
        if actual_budget != expected_budget:
                    f"Budget mismatch: expected {expected_budget}, got {actual_budget}"
                expected_end_token_id = end_tokens[
                forced_logit = float(logits_new[batch_index, expected_end_token_id])
                            f"Expected forced end token {expected_end_token_id} "
                            f"with large logit, got {forced_logit}"
                        ),
                        batch_index=batch_index,
                        request_params=request_params,
                        step_idx=step_idx,
                    )
```
**EN:** Helper function `_thinking_budget_validate` encapsulates reusable logic for `thinking budget validate`. Inputs: `test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`. Key calls include `_state.get, hasattr, _raise_error_invalid, len, range, holder.has_tracked_requests`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_thinking_budget_validate` 封装了与 `thinking budget validate` 相关的可复用逻辑。 输入参数：`test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`。 关键调用包括 `_state.get, hasattr, _raise_error_invalid, len, range, holder.has_tracked_requests`。 其中包含 1 个内部断言，用于保护前置假设。

### _none_validate (lines 545-570)
```python
def _none_validate(
    test_fakes: LogitsprocsTestFakes,
    persistent_batch: list[LogitsProcsRequestParams],
    logits_new: torch.Tensor,
    batch_index: int,
    request_params: LogitsProcsRequestParams,
    step_idx: int,
) -> None:
    """Validate that no logits processors are applied"""
    logits = test_fakes.logits[persistent_batch[batch_index].workload_index].cpu()
    ref_logits = logits_new[batch_index]
    if not torch.all(ref_logits == logits):
        mismatch_toks = (ref_logits != logits).nonzero(as_tuple=True)[0].tolist()
        mismatch_strs = []
        for token in mismatch_toks:
            val = float(logits[token])
            ref_val = float(ref_logits[token])
            mismatch_strs.append(f"({token=},{val=},{ref_val=})")
        _raise_error_invalid(
            msg_suffix=(
                f"Unexpected modification of logits: {','.join(mismatch_strs)}"
            ),
            batch_index=batch_index,
            request_params=request_params,
            step_idx=step_idx,
        )
```
**EN:** Helper function `_none_validate` encapsulates reusable logic for `none validate`. Inputs: `test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`. Key calls include `logits.cpu, torch.all, nonzero.tolist, _raise_error_invalid, float, mismatch_strs.append`.
**CN:** 辅助函数 `_none_validate` 封装了与 `none validate` 相关的可复用逻辑。 输入参数：`test_fakes, persistent_batch, logits_new, batch_index, request_params, step_idx`。 关键调用包括 `logits.cpu, torch.all, nonzero.tolist, _raise_error_invalid, float, mismatch_strs.append`。

### LogitsprocTestHelpers (lines 573-577)
```python
class LogitsprocTestHelpers(NamedTuple):
    """Supports setting up and validating logitsprocs unit tests."""

    eval_fxn: Callable
    gen_request_fxn: Callable | None = None
```
**EN:** Class `LogitsprocTestHelpers` groups 0 test method(s). Bases: `NamedTuple`.
**CN:** 类 `LogitsprocTestHelpers` 组织了 0 个测试方法。 基类：`NamedTuple`。

### Module state / 模块级状态 (lines 580-594)
```python
logitsprocs_test_mapping = {
    STR_NO_LOGITPROC: LogitsprocTestHelpers(eval_fxn=_none_validate),
    LogitBiasLogitsProcessor: LogitsprocTestHelpers(
        gen_request_fxn=_logit_bias_params, eval_fxn=_logit_bias_validate
    ),
    MinPLogitsProcessor: LogitsprocTestHelpers(
        gen_request_fxn=_min_p_params, eval_fxn=_min_p_validate
    ),
    MinTokensLogitsProcessor: LogitsprocTestHelpers(
        gen_request_fxn=_min_tokens_params, eval_fxn=_min_tokens_validate
    ),
    STR_THINKING_BUDGET: LogitsprocTestHelpers(
        gen_request_fxn=_thinking_budget_params, eval_fxn=_thinking_budget_validate
    ),
}
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `logitsprocs_test_mapping`. Shared setup calls include `LogitsprocTestHelpers`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`logitsprocs_test_mapping`。 共享初始化调用包括 `LogitsprocTestHelpers`。

### _get_test_cases (lines 597-612)
```python
def _get_test_cases() -> list[list[str]]:
    """Each test case is a set of logitsprocs"""
    logitsprocs_types = list(logitsprocs_test_mapping.keys())

    # Isolate thinking-budget handling from other processors to avoid cross-talk.
    thinking_id: LogitprocType = STR_THINKING_BUDGET
    other_processors = [
        p for p in logitsprocs_types if p != STR_NO_LOGITPROC and p != thinking_id
    ]

    return (
        [[STR_NO_LOGITPROC]]
        + [[logitproc_type, STR_NO_LOGITPROC] for logitproc_type in other_processors]
        + [other_processors]
        + [[thinking_id]]
    )
```
**EN:** Helper function `_get_test_cases` encapsulates reusable logic for `test cases`. Key calls include `list, logitsprocs_test_mapping.keys`.
**CN:** 辅助函数 `_get_test_cases` 封装了与 `test cases` 相关的可复用逻辑。 关键调用包括 `list, logitsprocs_test_mapping.keys`。

### _generate_fake_step_update (lines 615-749)
```python
def _generate_fake_step_update(
    persistent_batch: list[LogitsProcsRequestParams],
    workload_params: list[LogitsProcsRequestParams],
    wdx: int,
    batch_update_builder: BatchUpdateBuilder,
) -> tuple[BatchUpdate | None, int, int]:
    batch_size = len(persistent_batch)
    workload_size = len(workload_params)
    workload_reqs_remaining = workload_size - wdx
    max_add_remove_per_step = max(1, int(0.2 * workload_size))

    # 50% of steps: add no reqs
    # Other 50%: add a limited number of reqs (less than the number
    # of workload reqs remaining, less than an arbitrary max)
    # If no workload reqs remain: 100% of steps have 0 adds
    num_step_add = (
        random.choice(
            [
    # ... excerpt omitted for brevity ...
        assert first_empty_index is not None
    return (
        batch_update_builder.get_and_reset(condensed_batch_size),
        wdx,
        workload_size - wdx,
    )
```
**EN:** Helper function `_generate_fake_step_update` encapsulates reusable logic for `generate fake step update`. Inputs: `persistent_batch, workload_params, wdx, batch_update_builder`. Key calls include `len, max, min, random.sample, added.extend, persistent_batch.extend`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_generate_fake_step_update` 封装了与 `generate fake step update` 相关的可复用逻辑。 输入参数：`persistent_batch, workload_params, wdx, batch_update_builder`。 关键调用包括 `len, max, min, random.sample, added.extend, persistent_batch.extend`。 其中包含 1 个内部断言，用于保护前置假设。

### _assert_valid (lines 752-783)
```python
def _assert_valid(
    batch_size: int,
    persistent_batch: list[LogitsProcsRequestParams],
    test_fakes: LogitsprocsTestFakes,
    slice_idxs: list[int],
    logits_w_lp: torch.Tensor,
    step_idx: int,
) -> None:
    if not slice_idxs:
        # Trivial case of empty persistent batch
        assert len(persistent_batch) == 0
        if logits_w_lp.shape[0] != 0:
            raise ValueError(
                "Fake persistent batch is empty but logitsprocs "
                f"output batch has shape {logits_w_lp.shape}"
            )
        return

    # Validate logits for each fake request
    for batch_index in range(batch_size):
        request_params = persistent_batch[batch_index]
        # Invoke the appropriate validation function for
        # the logitproc employed by this request
        fxn = logitsprocs_test_mapping[request_params.logitproc_type].eval_fxn
        fxn(
            test_fakes=test_fakes,
            persistent_batch=persistent_batch,
            logits_new=logits_w_lp,
            batch_index=batch_index,
            request_params=request_params,
            step_idx=step_idx,
        )
```
**EN:** Helper function `_assert_valid` encapsulates reusable logic for `assert valid`. Inputs: `batch_size, persistent_batch, test_fakes, slice_idxs, logits_w_lp, step_idx`. Key calls include `range, fxn, len, ValueError`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_assert_valid` 封装了与 `assert valid` 相关的可复用逻辑。 输入参数：`batch_size, persistent_batch, test_fakes, slice_idxs, logits_w_lp, step_idx`。 关键调用包括 `range, fxn, len, ValueError`。 其中包含 1 个内部断言，用于保护前置假设。

### _slot_outputs_for_metadata (lines 786-794)
```python
def _slot_outputs_for_metadata(
    persistent_batch: list[LogitsProcsRequestParams], pad_len: int
) -> list[list[int]]:
    """Per-batch-slot output token ids aligned with ``SamplingMetadata`` rows."""
    rows: list[list[int]] = [[] for _ in range(pad_len)]
    for i, req in enumerate(persistent_batch):
        if i < pad_len:
            rows[i] = list(req.out_tokens)
    return rows
```
**EN:** Helper function `_slot_outputs_for_metadata` encapsulates reusable logic for `slot outputs for metadata`. Inputs: `persistent_batch, pad_len`. Key calls include `enumerate, range, list`.
**CN:** 辅助函数 `_slot_outputs_for_metadata` 封装了与 `slot outputs for metadata` 相关的可复用逻辑。 输入参数：`persistent_batch, pad_len`。 关键调用包括 `enumerate, range, list`。

### test_logitsprocs (lines 801-864)
```python
def test_logitsprocs(
    device: str, reqs_per_logitproc: int, logitsprocs_under_test: list[LogitprocType]
):
    random.seed(40)
    torch.set_default_device(device)

    # Define a shuffled batch of requests which individually use a different
    # logitproc, or no logitproc at all
    workload_params = _generate_mixed_logitsprocs_batch_params(
        reqs_per_logitproc=reqs_per_logitproc, logitsprocs_types=logitsprocs_under_test
    )
    workload_size = len(workload_params)
    # Create fake test data structures for testing.
    test_fakes = _generate_test_fakes(workload_size, device)
    wdx = 0  # Next request index in workload to add
    persistent_batch: list[
    # ... excerpt omitted for brevity ...
            slice_idxs=slice_idxs,
            logits_w_lp=logits_w_lp,
            step_idx=step_idx,
        )
        step_idx += 1
```
**EN:** Parameterized test covering `logitsprocs`. Parameter axes: `device, reqs_per_logitproc, logitsprocs_under_test`. Inputs/fixtures: `device, reqs_per_logitproc, logitsprocs_under_test`. It exercises `create_new_process_for_each_test, mark.parametrize, random.seed, torch.set_default_device, _generate_mixed_logitsprocs_batch_params, len`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `logitsprocs` 的测试用例。 参数维度：`device, reqs_per_logitproc, logitsprocs_under_test`。 输入或 fixture：`device, reqs_per_logitproc, logitsprocs_under_test`。 该测试会调用 `create_new_process_for_each_test, mark.parametrize, random.seed, torch.set_default_device, _generate_mixed_logitsprocs_batch_params, len`。 主要通过 mock、回调或输出检查来完成验证。

### MockReasoningNoEndTokens (lines 867-871)
```python
class MockReasoningNoEndTokens:
    """Reasoning config with no end token ids (disables enforcement in holder)."""

    reasoning_start_token_ids = [THINK_START_TOKEN_ID]
    reasoning_end_token_ids: list[int] = []
```
**EN:** Class `MockReasoningNoEndTokens` groups 0 test method(s).
**CN:** 类 `MockReasoningNoEndTokens` 组织了 0 个测试方法。

### test_maybe_create_thinking_budget_holder_without_reasoning (lines 874-886)
```python
def test_maybe_create_thinking_budget_holder_without_reasoning():
    cfg = VllmConfig()
    assert cfg.reasoning_config is None
    assert (
        maybe_create_thinking_budget_state_holder(
            None,
            cfg.scheduler_config.max_num_seqs,
            0,
            torch.device("cpu"),
            False,
        )
        is None
    )
```
**EN:** Test case covering `maybe create thinking budget holder without reasoning`. It exercises `VllmConfig, maybe_create_thinking_budget_state_holder, torch.device`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `maybe create thinking budget holder without reasoning` 的测试用例。 该测试会调用 `VllmConfig, maybe_create_thinking_budget_state_holder, torch.device`。 代码主体包含 2 个显式断言。

### test_thinking_budget_holder_has_tracked_after_sync_add (lines 889-916)
```python
def test_thinking_budget_holder_has_tracked_after_sync_add():
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningConfig()
    h = ThinkingBudgetStateHolder(
        vc.reasoning_config,
        vc.scheduler_config.max_num_seqs,
        0,
        torch.device("cpu"),
        False,
    )
    assert not h.has_tracked_requests()
    h.sync_batch(
        BatchUpdate(
            batch_size=1,
            removed=(),
            added=[
                (
                    0,
                    SamplingParams(thinking_token_budget=3),
                    None,
                    [THINK_START_TOKEN_ID],
                )
            ],
            moved=(),
        )
    )
    assert h.has_tracked_requests()
    assert h._state[0]["thinking_token_budget"] == 3
```
**EN:** Test case covering `thinking budget holder has tracked after sync add`. It exercises `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.has_tracked_requests, torch.device`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder has tracked after sync add` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.has_tracked_requests, torch.device`。 代码主体包含 3 个显式断言。

### test_thinking_budget_holder_sync_remove_clears_state (lines 919-946)
```python
def test_thinking_budget_holder_sync_remove_clears_state():
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningConfig()
    h = ThinkingBudgetStateHolder(
        vc.reasoning_config,
        vc.scheduler_config.max_num_seqs,
        0,
        torch.device("cpu"),
        False,
    )
    h.sync_batch(
        BatchUpdate(
            batch_size=1,
            removed=(),
            added=[
                (
                    0,
                    SamplingParams(thinking_token_budget=3),
                    None,
                    [],
                )
            ],
            moved=(),
        )
    )
    assert h.has_tracked_requests()
    h.sync_batch(BatchUpdate(batch_size=0, removed=(0,), added=(), moved=()))
    assert not h.has_tracked_requests()
```
**EN:** Test case covering `thinking budget holder sync remove clears state`. It exercises `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.has_tracked_requests, torch.device`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder sync remove clears state` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.has_tracked_requests, torch.device`。 代码主体包含 2 个显式断言。

### test_thinking_budget_holder_sync_add_without_budget_drops_row (lines 949-967)
```python
def test_thinking_budget_holder_sync_add_without_budget_drops_row():
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningConfig()
    h = ThinkingBudgetStateHolder(
        vc.reasoning_config,
        vc.scheduler_config.max_num_seqs,
        0,
        torch.device("cpu"),
        False,
    )
    h.sync_batch(
        BatchUpdate(
            batch_size=1,
            removed=(),
            added=[(0, SamplingParams(), None, [])],
            moved=(),
        )
    )
    assert not h.has_tracked_requests()
```
**EN:** Test case covering `thinking budget holder sync add without budget drops row`. It exercises `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, torch.device, BatchUpdate`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder sync add without budget drops row` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, torch.device, BatchUpdate`。 代码主体包含 1 个显式断言。

### test_thinking_budget_holder_swap_exchanges_state (lines 970-1011)
```python
def test_thinking_budget_holder_swap_exchanges_state():
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningConfig()
    h = ThinkingBudgetStateHolder(
        vc.reasoning_config,
        vc.scheduler_config.max_num_seqs,
        0,
        torch.device("cpu"),
        False,
    )
    h.sync_batch(
        BatchUpdate(
            batch_size=2,
            removed=(),
            added=[
                (
                    0,
                    SamplingParams(thinking_token_budget=3),
    # ... excerpt omitted for brevity ...
            added=(),
            moved=[(0, 1, MoveDirectionality.SWAP)],
        )
    assert h._state[0]["thinking_token_budget"] == b1
    assert h._state[1]["thinking_token_budget"] == b0
```
**EN:** Test case covering `thinking budget holder swap exchanges state`. It exercises `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, torch.device, BatchUpdate`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder swap exchanges state` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, torch.device, BatchUpdate`。 代码主体包含 2 个显式断言。

### test_thinking_budget_holder_unidirectional_move (lines 1014-1049)
```python
def test_thinking_budget_holder_unidirectional_move():
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningConfig()
    h = ThinkingBudgetStateHolder(
        vc.reasoning_config,
        vc.scheduler_config.max_num_seqs,
        0,
        torch.device("cpu"),
        False,
    )
    h.sync_batch(
        BatchUpdate(
            batch_size=2,
            removed=(),
            added=[
                (
                    1,
                    SamplingParams(thinking_token_budget=4),
    # ... excerpt omitted for brevity ...
    assert 1 in h._state and 0 not in h._state
            added=(),
            moved=[(1, 0, MoveDirectionality.UNIDIRECTIONAL)],
        )
    assert 0 in h._state and 1 not in h._state
    assert h._state[0]["thinking_token_budget"] == 4
```
**EN:** Test case covering `thinking budget holder unidirectional move`. It exercises `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, torch.device, BatchUpdate`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder unidirectional move` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, torch.device, BatchUpdate`。 代码主体包含 3 个显式断言。

### test_thinking_budget_holder_update_state_repeat_indices_last_row_wins (lines 1052-1083)
```python
def test_thinking_budget_holder_update_state_repeat_indices_last_row_wins():
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningConfig()
    h = ThinkingBudgetStateHolder(
        vc.reasoning_config,
        vc.scheduler_config.max_num_seqs,
        0,
        torch.device("cpu"),
        False,
    )
    h.sync_batch(
        BatchUpdate(
            batch_size=1,
            removed=(),
            added=[
                (
                    0,
                    SamplingParams(thinking_token_budget=5),
                    None,
                    [THINK_START_TOKEN_ID],
                )
            ],
            moved=(),
        )
    )
    out_lists = [[THINK_START_TOKEN_ID], [THINK_START_TOKEN_ID, 10, 11, 12, 13, 14]]
    h.update_state(
        out_lists,
        None,
        torch.tensor([0, 0], dtype=torch.long),
    )
    assert h._state[0]["output_tok_ids"] == out_lists[1]
```
**EN:** Test case covering `thinking budget holder update state repeat indices last row wins`. It exercises `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.update_state, torch.device`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder update state repeat indices last row wins` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.update_state, torch.device`。 代码主体包含 1 个显式断言。

### test_thinking_budget_holder_spec_mode_tensor_layout (lines 1086-1095)
```python
def test_thinking_budget_holder_spec_mode_tensor_layout():
    h = ThinkingBudgetStateHolder(
        MockReasoningConfig(),
        8,
        2,
        torch.device("cpu"),
        False,
    )
    assert h.in_spec_mode
    assert h.mask.shape[0] == 8 * (2 + 1)
```
**EN:** Test case covering `thinking budget holder spec mode tensor layout`. It exercises `ThinkingBudgetStateHolder, MockReasoningConfig, torch.device`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder spec mode tensor layout` 的测试用例。 该测试会调用 `ThinkingBudgetStateHolder, MockReasoningConfig, torch.device`。 代码主体包含 2 个显式断言。

### test_thinking_budget_holder_empty_end_tokens_disables_row (lines 1098-1124)
```python
def test_thinking_budget_holder_empty_end_tokens_disables_row():
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningNoEndTokens()
    h = ThinkingBudgetStateHolder(
        vc.reasoning_config,
        vc.scheduler_config.max_num_seqs,
        0,
        torch.device("cpu"),
        False,
    )
    h.sync_batch(
        BatchUpdate(
            batch_size=1,
            removed=(),
            added=[
                (
                    0,
                    SamplingParams(thinking_token_budget=5),
                    None,
                    [THINK_START_TOKEN_ID],
                )
            ],
            moved=(),
        )
    )
    h.update_state([[THINK_START_TOKEN_ID, 1]], None, None)
    assert h._state[0]["thinking_token_budget"] == -1
```
**EN:** Test case covering `thinking budget holder empty end tokens disables row`. It exercises `VllmConfig, MockReasoningNoEndTokens, ThinkingBudgetStateHolder, h.sync_batch, h.update_state, torch.device`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget holder empty end tokens disables row` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningNoEndTokens, ThinkingBudgetStateHolder, h.sync_batch, h.update_state, torch.device`。 代码主体包含 1 个显式断言。

### test_thinking_budget_enforced_without_penalties (lines 1127-1196)
```python
def test_thinking_budget_enforced_without_penalties():
    """Regression test for gpu_input_batch.py bug.

    When thinking_budget_tracks_reqs=True and no penalties/bad_words are set,
    the old code computed needs_output_token_ids=False (inverted condition:
    ``or not thinking_budget_tracks_reqs``), causing update_state to receive
    an empty list and skip _update_think_state for every request.
    Fix: changed ``or not thinking_budget_tracks_reqs`` to
    ``or thinking_budget_tracks_reqs`` so that output_token_ids is populated
    whenever the thinking budget state holder has tracked requests.
    This test verifies that update_state correctly calls _update_think_state
    (setting in_end=True) when given the real output_token_ids, and that
    passing an empty list (the pre-fix behavior) prevents budget enforcement.
    """
    vc = VllmConfig()
    vc.reasoning_config = MockReasoningConfig()
    # ... excerpt omitted for brevity ...
    assert h.has_tracked_requests()
    assert not h._state[0].get("in_end", False), (
    # After exactly `budget` thinking tokens the holder should force end token.
    assert h._state[0].get("in_end", False), (
        "Budget exceeded: in_end should be True so that apply_to_logits "
        "forces the end token"
    )
```
**EN:** Test case covering `thinking budget enforced without penalties`. It exercises `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.has_tracked_requests, h.update_state`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `thinking budget enforced without penalties` 的测试用例。 该测试会调用 `VllmConfig, MockReasoningConfig, ThinkingBudgetStateHolder, h.sync_batch, h.has_tracked_requests, h.update_state`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Logits post-processing behavior
- **CN:** logits 后处理行为

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch`.
- **CN:** 外部库：`numpy, pytest, torch`。
- **EN:** vLLM modules under test: `vllm.config, vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.thinking_budget_state`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.platforms, vllm.sampling_params, vllm.utils.platform_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.thinking_budget_state`。
- **EN:** Local test helpers: `tests.utils, tests.v1.sample.utils`.
- **CN:** 本地测试辅助模块：`tests.utils, tests.v1.sample.utils`。
- **EN:** Standard-library support: `random, collections.abc, typing`.
- **CN:** 标准库支持：`random, collections.abc, typing`。
