# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/sample/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers reused by the v1 sample tests. / 为 v1 `sample` 测试提供可复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-14)
```python
from collections.abc import Iterator
from enum import Enum
from typing import NamedTuple

import regex as re
import torch

from vllm import CompletionOutput
from vllm.utils.torch_utils import make_tensor_with_pad
from vllm.v1.sample.logits_processor import BatchUpdate, LogitsProcessor
from vllm.v1.sample.metadata import SamplingMetadata
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `regex, torch`. vLLM modules under test include `vllm, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `regex, torch`。 被测试的 vLLM 模块包括 `vllm, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata`。

### BatchLogprobsComposition (lines 17-23)
```python
class BatchLogprobsComposition(Enum):
    """Types of logprobs configs to include in test batch"""

    NONE = 0
    SAMPLE = 1
    PROMPT = 2
    SAMPLE_PROMPT = 3
```
**EN:** Class `BatchLogprobsComposition` groups 0 test method(s). Bases: `Enum`.
**CN:** 类 `BatchLogprobsComposition` 组织了 0 个测试方法。 基类：`Enum`。

### Module state / 模块级状态 (line 26)
```python
BatchLogprobsSpecType = list[tuple[int | None, int | None]]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `BatchLogprobsSpecType`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`BatchLogprobsSpecType`。

### get_test_batch (lines 29-96)
```python
def get_test_batch(
    batch_logprobs_composition: BatchLogprobsComposition,
) -> BatchLogprobsSpecType:
    """Generate logprobs configs for a batch of requests

    A given request's logprobs configuration is (1) num_sample_logprobs and (2)
    num_prompt_logprobs. The batch logprobs configuration is the list of request
    logprobs configs.
    batch_logprobs_composition == NONE yields a batch with no sample or prompt
    logprobs
    batch_logprobs_composition == SAMPLE yields a batch with some requests
    configured for sample logprobs only, and others configured for no logprobs
    batch_logprobs_composition == PROMPT yields a batch with some requests
    configured for prompt logprobs only, and others configured for no logprobs
    # ... excerpt omitted for brevity ...
        return [(None, None)]
        return [
            (None, 6),
            (None, 5),
            (None, 0),
        ]
    else:
        raise ValueError("Invalid logprobs batch configuration for test.")
```
**EN:** Helper function `get_test_batch` encapsulates reusable logic for `test batch`. Inputs: `batch_logprobs_composition`. Key calls include `ValueError`.
**CN:** 辅助函数 `get_test_batch` 封装了与 `test 批处理` 相关的可复用逻辑。 输入参数：`batch_logprobs_composition`。 关键调用包括 `ValueError`。

### assert_incr_detok_str_matches_non_incr_detok_str (lines 99-127)
```python
def assert_incr_detok_str_matches_non_incr_detok_str(
    incremental_detokenization_str: str,
    non_incremental_detokenization_str: str,
    msg: str,
) -> None:
    """Compare incrementally detok. text to non-incrementally detok. text

    Fail if the strings mismatch after non-alphanumeric characters are stripped
    out.

    Rationale: incremental detokenization in the text generation process allows
    the tokenizer to adjust the next token text output based on the token's
    context in the string. However, logprobs detokenization detokenizes each
    token individually, and the resultant strings may include some
    non-alphanumeric placeholder characters where there could be i.e.
    whitespace. So, this function compares only the alphanumeric text
    between two strings and fails if there is a mismatch, which helps
    with validating logprobs detokenization.

    Args:
      incremental_detokenization_str: incrementally-detokenized generated text
      non_incremental_detokenization_str: non-incrementally-detokenized logprob
                                          tokens
      msg: error message if `assert` fails
    """
    rgx = r"[^a-zA-Z0-9]+"
    assert re.sub(rgx, "", incremental_detokenization_str) == re.sub(
        rgx, "", non_incremental_detokenization_str
    ), msg
```
**EN:** Helper function `assert_incr_detok_str_matches_non_incr_detok_str` encapsulates reusable logic for `assert incr detok str matches non incr detok str`. Inputs: `incremental_detokenization_str, non_incremental_detokenization_str, msg`. Key calls include `re.sub`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `assert_incr_detok_str_matches_non_incr_detok_str` 封装了与 `assert incr detok str matches non incr detok str` 相关的可复用逻辑。 输入参数：`incremental_detokenization_str, non_incremental_detokenization_str, msg`。 关键调用包括 `re.sub`。 其中包含 1 个内部断言，用于保护前置假设。

### compute_correct_cumulative_logprob (lines 130-142)
```python
def compute_correct_cumulative_logprob(completion_output: CompletionOutput) -> float:
    """Compute known-good value for evaluating cumulative logprob

    Args:
      completion_output: completion output from engine

    Returns:
      Known-good cumulative logprob value
    """
    token_ids = completion_output.token_ids
    logprobs = completion_output.logprobs
    assert logprobs is not None
    return sum([lp[tok_id].logprob for tok_id, lp in zip(token_ids, logprobs)])
```
**EN:** Helper function `compute_correct_cumulative_logprob` encapsulates reusable logic for `compute correct cumulative logprob`. Inputs: `completion_output`. Key calls include `sum, zip`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `compute_correct_cumulative_logprob` 封装了与 `compute correct cumulative logprob` 相关的可复用逻辑。 输入参数：`completion_output`。 关键调用包括 `sum, zip`。 其中包含 1 个内部断言，用于保护前置假设。

### create_fake_logits (lines 145-147)
```python
def create_fake_logits(batch_size: int, vocab_size: int) -> torch.Tensor:
    fake_logits = torch.full((batch_size, vocab_size), 1e-2, dtype=torch.float)
    return fake_logits
```
**EN:** Helper function `create_fake_logits` encapsulates reusable logic for `fake logits`. Inputs: `batch_size, vocab_size`. Key calls include `torch.full`.
**CN:** 辅助函数 `create_fake_logits` 封装了与 `fake logits` 相关的可复用逻辑。 输入参数：`batch_size, vocab_size`。 关键调用包括 `torch.full`。

### create_penalty_tensor (lines 150-155)
```python
def create_penalty_tensor(
    batch_size: int, penalty_value: float, device: torch.device
) -> torch.Tensor:
    return torch.full(
        (batch_size,), fill_value=penalty_value, dtype=torch.float, device=device
    )
```
**EN:** Helper function `create_penalty_tensor` encapsulates reusable logic for `penalty tensor`. Inputs: `batch_size, penalty_value, device`. Key calls include `torch.full`.
**CN:** 辅助函数 `create_penalty_tensor` 封装了与 `penalty tensor` 相关的可复用逻辑。 输入参数：`batch_size, penalty_value, device`。 关键调用包括 `torch.full`。

### create_prompt_tokens_tensor (lines 158-169)
```python
def create_prompt_tokens_tensor(
    prompt_token_ids: list[list[int]],
    vocab_size: int,
    device: torch.device,
) -> torch.Tensor:
    return make_tensor_with_pad(
        prompt_token_ids,
        pad=vocab_size,
        device=device,
        dtype=torch.int64,
        pin_memory=False,
    )
```
**EN:** Helper function `create_prompt_tokens_tensor` encapsulates reusable logic for `prompt tokens tensor`. Inputs: `prompt_token_ids, vocab_size, device`. Key calls include `make_tensor_with_pad`.
**CN:** 辅助函数 `create_prompt_tokens_tensor` 封装了与 `prompt tokens tensor` 相关的可复用逻辑。 输入参数：`prompt_token_ids, vocab_size, device`。 关键调用包括 `make_tensor_with_pad`。

### LogitsprocsTestFakes (lines 172-196)
```python
class LogitsprocsTestFakes(NamedTuple):
    """Wraps fake data structures to support testing"""

    logits: torch.Tensor
    sampling_metadata: SamplingMetadata

    def get_logitsprocs_by_cls(
        self,
        cls: type[LogitsProcessor],
    ) -> Iterator[LogitsProcessor]:
        """Yield logits processors of a specific class.

        Args:
          cls: :class:`LogitsProcessor` subclass

        Returns:
          Iterator over logits processors
        """
        return (
            lp for lp in self.sampling_metadata.logitsprocs.all if isinstance(lp, cls)
        )

    def get_logitsprocs(self) -> Iterator[LogitsProcessor]:
        """Iterator over all logits processors."""
        return self.sampling_metadata.logitsprocs.all
```
**EN:** Class `LogitsprocsTestFakes` groups 0 test method(s) and 2 helper/fixture method(s). Bases: `NamedTuple`.
**CN:** 类 `LogitsprocsTestFakes` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。 基类：`NamedTuple`。

### fake_update_logitsprocs_state (lines 199-209)
```python
def fake_update_logitsprocs_state(
    test_fakes: LogitsprocsTestFakes,
    batch_update: BatchUpdate | None,
) -> None:
    """Imitate logits processors persistent batch state update
    in engine core"""
    for logitproc in test_fakes.get_logitsprocs():
        logitproc.update_state(batch_update)
    holder = test_fakes.sampling_metadata.thinking_budget_state_holder
    if holder is not None:
        holder.sync_batch(batch_update)
```
**EN:** Helper function `fake_update_logitsprocs_state` encapsulates reusable logic for `fake update logitsprocs state`. Inputs: `test_fakes, batch_update`. Key calls include `test_fakes.get_logitsprocs, logitproc.update_state, holder.sync_batch`.
**CN:** 辅助函数 `fake_update_logitsprocs_state` 封装了与 `fake update logitsprocs state` 相关的可复用逻辑。 输入参数：`test_fakes, batch_update`。 关键调用包括 `test_fakes.get_logitsprocs, logitproc.update_state, holder.sync_batch`。

### fake_apply_logitsprocs (lines 212-238)
```python
def fake_apply_logitsprocs(
    test_fakes: LogitsprocsTestFakes,
    slice_indices: list[int],
    slot_output_token_ids: list[list[int]] | None = None,
) -> torch.Tensor:
    """Imitate application of logits processors in engine core.

    When ``thinking_budget_state_holder`` has tracked requests, this mirrors
    :meth:`Sampler.apply_logits_processors` by refreshing per-slot
    ``output_token_ids`` (if ``slot_output_token_ids`` is provided), then
    ``update_state`` + ``apply_to_logits`` on the holder after built-in logits
    processors.
    """
    logits = test_fakes.logits[torch.tensor(slice_indices, dtype=torch.long)].clone()
    for processor in test_fakes.get_logitsprocs():
        logits = processor.apply(logits)

    md = test_fakes.sampling_metadata
    holder = md.thinking_budget_state_holder
    if holder is not None and holder.has_tracked_requests():
        if slot_output_token_ids is not None:
            for i, toks in enumerate(slot_output_token_ids):
                if i < len(md.output_token_ids):
                    md.output_token_ids[i] = list(toks)
        holder.update_state(md.output_token_ids, md.spec_token_ids, None)
        logits = holder.apply_to_logits(logits, False, md.spec_token_ids)
    return logits
```
**EN:** Helper function `fake_apply_logitsprocs` encapsulates reusable logic for `fake apply logitsprocs`. Inputs: `test_fakes, slice_indices, slot_output_token_ids`. Key calls include `logits.clone, test_fakes.get_logitsprocs, processor.apply, holder.has_tracked_requests, holder.update_state, holder.apply_to_logits`.
**CN:** 辅助函数 `fake_apply_logitsprocs` 封装了与 `fake apply logitsprocs` 相关的可复用逻辑。 输入参数：`test_fakes, slice_indices, slot_output_token_ids`。 关键调用包括 `logits.clone, test_fakes.get_logitsprocs, processor.apply, holder.has_tracked_requests, holder.update_state, holder.apply_to_logits`。

### create_allowed_token_ids (lines 241-258)
```python
def create_allowed_token_ids(
    batch_size: int,
    vocab_size: int,
    num_allowed_token_ids: int,
    device: torch.device,
) -> torch.Tensor | None:
    mask: torch.Tensor | None = None
    for i in range(batch_size):
        if i % 2 == 1:
            continue
        if mask is None:
            mask = torch.zeros(
                (batch_size, vocab_size), dtype=torch.bool, device=device
            )
        start = min(i, vocab_size - 1)
        end = min(i + num_allowed_token_ids, vocab_size - 1)
        mask[i, start:end] = True
    return mask
```
**EN:** Helper function `create_allowed_token_ids` encapsulates reusable logic for `allowed token ids`. Inputs: `batch_size, vocab_size, num_allowed_token_ids, device`. Key calls include `range, min, torch.zeros`.
**CN:** 辅助函数 `create_allowed_token_ids` 封装了与 `allowed token ids` 相关的可复用逻辑。 输入参数：`batch_size, vocab_size, num_allowed_token_ids, device`。 关键调用包括 `range, min, torch.zeros`。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性
- **EN:** Logits post-processing behavior
- **CN:** logits 后处理行为
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查

## Dependencies / 依赖关系
- **EN:** External libraries: `regex, torch`.
- **CN:** 外部库：`regex, torch`。
- **EN:** vLLM modules under test: `vllm, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata`。
- **EN:** Standard-library support: `collections.abc, enum, typing`.
- **CN:** 标准库支持：`collections.abc, enum, typing`。
