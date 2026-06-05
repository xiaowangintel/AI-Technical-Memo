# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/engine/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers reused by the v1 engine tests. / 为 v1 `引擎` 测试提供可复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-15)
```python
import random
from dataclasses import dataclass
from typing import TypeAlias

import numpy as np
import torch
from transformers import PreTrainedTokenizer, PreTrainedTokenizerFast

from vllm.engine.arg_utils import EngineArgs
from vllm.v1.engine import EngineCoreOutput, FinishReason
from vllm.v1.metrics.stats import PrefillStats
from vllm.v1.outputs import LogprobsLists, LogprobsTensors
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, torch, transformers`. vLLM modules under test include `vllm.engine.arg_utils, vllm.v1.engine, vllm.v1.metrics.stats, vllm.v1.outputs`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, torch, transformers`。 被测试的 vLLM 模块包括 `vllm.engine.arg_utils, vllm.v1.engine, vllm.v1.metrics.stats, vllm.v1.outputs`。

### Module state / 模块级状态 (lines 17-34)
```python
GeneralTokenizerType: TypeAlias = PreTrainedTokenizer | PreTrainedTokenizerFast

# Number of sample logprobs to request when testing sample logprobs
NUM_SAMPLE_LOGPROBS_UNDER_TEST = 5
# Number of prompt logprobs to request when testing prompt logprobs
NUM_PROMPT_LOGPROBS_UNDER_TEST = 7

TOKENIZER_NAME = "meta-llama/Llama-3.2-1B"

FULL_STRINGS = [
    "My name is Robert from Neural Magic and I love working on vLLM so much!",
    "Red Hat is the best open source company by far across Linux, K8s, and AI.",
    "Nick is the name of my brother in addition to my colleague from Red Hat.",
]
STOP_STRINGS = ["I love working on", "company by far", "brother in"]
PROMPT_LEN = 5

random.seed(42)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `GeneralTokenizerType, NUM_SAMPLE_LOGPROBS_UNDER_TEST, NUM_PROMPT_LOGPROBS_UNDER_TEST, TOKENIZER_NAME, FULL_STRINGS, STOP_STRINGS, ...`. Shared setup calls include `random.seed`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`GeneralTokenizerType, NUM_SAMPLE_LOGPROBS_UNDER_TEST, NUM_PROMPT_LOGPROBS_UNDER_TEST, TOKENIZER_NAME, FULL_STRINGS, STOP_STRINGS, ...`。 共享初始化调用包括 `random.seed`。

### _create_random_top_logprob_test_vector (lines 37-58)
```python
def _create_random_top_logprob_test_vector(
    num_logprobs: int,
    lower: float,
    upper: float,
) -> torch.Tensor:
    """Create a random vector of top logprob float values.

    Use to create fake sample logprobs for testing.

    Note that a real production scenario would require
    logprobs to be sorted in descending order, something
    which is omitted in this function.

    Args:
      num_logprobs: number of top logprobs
      lower: lower range of logprob float values
      upper: upper range of logprob float values

    Returns:
      1D length-`num_logprobs` torch Tensor of float logprob values
    """
    return torch.rand(num_logprobs) * (upper - lower) + lower
```
**EN:** Helper function `_create_random_top_logprob_test_vector` encapsulates reusable logic for `random top logprob test vector`. Inputs: `num_logprobs, lower, upper`. Key calls include `torch.rand`.
**CN:** 辅助函数 `_create_random_top_logprob_test_vector` 封装了与 `random top logprob test vector` 相关的可复用逻辑。 输入参数：`num_logprobs, lower, upper`。 关键调用包括 `torch.rand`。

### _create_random_top_logprob_test_matrix (lines 61-83)
```python
def _create_random_top_logprob_test_matrix(
    shape: tuple,
    lower: float,
    upper: float,
) -> torch.Tensor:
    """Create a random matrix of top logprob float values.

    Use to create fake prompt logprobs for testing.

    Note that a real production scenario would require
    logprobs to be sorted in descending order along rows,
    something which is omitted in this function.

    Args:
      shape: (num_tokens,num_logprobs) tuple representing
             matrix shape
      lower: lower range of logprob float values
      upper: upper range of logprob float values

    Returns:
      2D num_tokens x num_logprobs torch Tensor of float logprob values
    """
    return torch.rand(*shape) * (upper - lower) + lower
```
**EN:** Helper function `_create_random_top_logprob_test_matrix` encapsulates reusable logic for `random top logprob test matrix`. Inputs: `shape, lower, upper`. Key calls include `torch.rand`.
**CN:** 辅助函数 `_create_random_top_logprob_test_matrix` 封装了与 `random top logprob test matrix` 相关的可复用逻辑。 输入参数：`shape, lower, upper`。 关键调用包括 `torch.rand`。

### _create_random_top_token_test_vector (lines 86-140)
```python
def _create_random_top_token_test_vector(
    num_logprobs: int,
    lower: int,
    upper: int,
    sampled_token_id: int,
    adjust_num_logprobs: bool = True,
) -> tuple[torch.Tensor, int]:
    """Create a random vector of top logprob token indices

    Use to create fake sample logprobs for testing. The sampled token
    ID must always be one of the top logprobs, which this dummy test
    vector generator enforces. OpenAI API
    compatible engines must be able to return an additional sample
    logprob for the sampled token if the sampled token was not
    among the top sample logprobs; `adjust_num_logprobs` emulates
    this behavior by increasing the vector length by 1 if
    `adjust_num_logprobs` is set.
    # ... excerpt omitted for brevity ...
        )
    else:
        # If not found, assign a random int between num_logprobs and 50700
        sampled_token_rank = random.randint(num_logprobs, 50700)
    return choice_tensor, sampled_token_rank
```
**EN:** Helper function `_create_random_top_token_test_vector` encapsulates reusable logic for `random top token test vector`. Inputs: `num_logprobs, lower, upper, sampled_token_id, adjust_num_logprobs`. Key calls include `nonzero.item, random.randint, torch.randperm, nonzero`.
**CN:** 辅助函数 `_create_random_top_token_test_vector` 封装了与 `random top token test vector` 相关的可复用逻辑。 输入参数：`num_logprobs, lower, upper, sampled_token_id, adjust_num_logprobs`。 关键调用包括 `nonzero.item, random.randint, torch.randperm, nonzero`。

### _create_random_top_token_test_matrix (lines 143-191)
```python
def _create_random_top_token_test_matrix(
    shape: tuple[int, int],
    lower: int,
    upper: int,
    tokens_list: list[int],
) -> tuple[torch.Tensor, torch.Tensor]:
    """Create a random matrix of top logprob token indices

    Use to create fake prompt logprobs for testing.
    Token ids are generated randomly and sampled without
    replacement.
    Args:
      shape: (num_tokens, num_logprobs) tuple representing
             matrix shape
      lower: lower range of token ids
      upper: upper range of token ids
    # ... excerpt omitted for brevity ...
        if token_index.numel() > 0:
            prompt_token_ranks[rdx] = token_index.item()
        else:
            prompt_token_ranks[rdx] = random.randint(shape[1], 50700)
    return matrix, prompt_token_ranks
```
**EN:** Helper function `_create_random_top_token_test_matrix` encapsulates reusable logic for `random top token test matrix`. Inputs: `shape, lower, upper, tokens_list`. Key calls include `torch.cat, torch.empty, range, torch.randperm, tensor.unsqueeze, choice_tensor.view`.
**CN:** 辅助函数 `_create_random_top_token_test_matrix` 封装了与 `random top token test matrix` 相关的可复用逻辑。 输入参数：`shape, lower, upper, tokens_list`。 关键调用包括 `torch.cat, torch.empty, range, torch.randperm, tensor.unsqueeze, choice_tensor.view`。

### decode_token (lines 194-207)
```python
def decode_token(
    tok_id: int,
    tokenizer: PreTrainedTokenizer,
) -> str:
    """Reproduce the process of detokenizing a token for testing purposes.

    Args:
      tok_id: token id to detokenize
      tokenizer: tokenizer to use for detokenization

    Returns:
      string representation of token
    """
    return tokenizer.convert_ids_to_tokens(tok_id)
```
**EN:** Helper function `decode_token` encapsulates reusable logic for `decode token`. Inputs: `tok_id, tokenizer`. Key calls include `tokenizer.convert_ids_to_tokens`.
**CN:** 辅助函数 `decode_token` 封装了与 `decode token` 相关的可复用逻辑。 输入参数：`tok_id, tokenizer`。 关键调用包括 `tokenizer.convert_ids_to_tokens`。

### generate_dummy_sample_logprobs (lines 210-256)
```python
def generate_dummy_sample_logprobs(
    sampled_tokens_list: list,
    num_logprobs: int,
    tokenizer: PreTrainedTokenizer,
) -> list[tuple[list[int], list[float], int]]:
    """Generate dummy sample logprobs

    Generate a test data structure which imitates the list of sample logprobs
    which would be assembled in the engine core during decode phase.
    Args:
      sampled_tokens_list: list of sampled tokens
      num_logprobs: return `num_logprobs` or `num_logprobs+1` logprobs per token
      tokenizer: model tokenizer to use for detokenization
    Returns
      list of (top token ids vector, logprobs vector, sampled token rank)
      Python lists tuples; in each tuple the logprobs and top token ids
    # ... excerpt omitted for brevity ...
    res_list_format = [
        (log_probs_tensor.tolist(), token_ids_tensor.tolist(), sampled_token_rank)
        for log_probs_tensor, token_ids_tensor, sampled_token_rank in res
    ]
    return res_list_format
```
**EN:** Helper function `generate_dummy_sample_logprobs` encapsulates reusable logic for `generate dummy sample logprobs`. Inputs: `sampled_tokens_list, num_logprobs, tokenizer`. Key calls include `_create_random_top_token_test_vector, res.append, log_probs_tensor.tolist, token_ids_tensor.tolist, len, _create_random_top_logprob_test_vector`.
**CN:** 辅助函数 `generate_dummy_sample_logprobs` 封装了与 `generate dummy sample 对数概率` 相关的可复用逻辑。 输入参数：`sampled_tokens_list, num_logprobs, tokenizer`。 关键调用包括 `_create_random_top_token_test_vector, res.append, log_probs_tensor.tolist, token_ids_tensor.tolist, len, _create_random_top_logprob_test_vector`。

### generate_dummy_prompt_logprobs_tensors (lines 259-304)
```python
def generate_dummy_prompt_logprobs_tensors(
    prompt_tokens_list: list,
    num_logprobs: int,
    tokenizer: PreTrainedTokenizer,
) -> LogprobsTensors:
    """Generate dummy prompt logprobs tensors

    Generate a test data structure which imitates the torch Tensors of prompt
    logprobs which would be assembled in the engine core during chunked
    prefill.
    Args:
      prompt_tokens_list: list of prompt tokens
      num_logprobs: return `num_logprobs` logprobs per token
      tokenizer: model tokenizer to use for detokenization
    Returns
      Single tuple of (logprobs matrix, top token ids matrix) torch Tensor,
    # ... excerpt omitted for brevity ...
    return LogprobsTensors(
        token_vector,
        _create_random_top_logprob_test_matrix(
            (num_prompt_logprobs, num_logprobs + 1), -100, 0
        ),
        prompt_token_ranks,
    )
```
**EN:** Helper function `generate_dummy_prompt_logprobs_tensors` encapsulates reusable logic for `generate dummy prompt logprobs tensors`. Inputs: `prompt_tokens_list, num_logprobs, tokenizer`. Key calls include `_create_random_top_token_test_matrix, LogprobsTensors, len, _create_random_top_logprob_test_matrix`.
**CN:** 辅助函数 `generate_dummy_prompt_logprobs_tensors` 封装了与 `generate dummy prompt 对数概率 tensors` 相关的可复用逻辑。 输入参数：`prompt_tokens_list, num_logprobs, tokenizer`。 关键调用包括 `_create_random_top_token_test_matrix, LogprobsTensors, len, _create_random_top_logprob_test_matrix`。

### DummyOutputProcessorTestVectors (lines 308-325)
```python
class DummyOutputProcessorTestVectors:
    """Dummy test vectors for output processor tests"""

    tokenizer: GeneralTokenizerType
    vllm_config: EngineArgs
    full_tokens: list[list[int]]  # Prompt + generated tokens
    prompt_tokens: list[list[int]]
    generation_tokens: list[list[int]]
    # Each request is associated with a tuple of
    # (top tokens, top logprobs, ranks) prompt logprobs tensors
    prompt_logprobs: list[LogprobsTensors]
    # Each request is associated with a sample logprobs; a request's
    # sample logprobs are a list of (top tokens, top logprobs, ranks)
    # sample logprobs tensors at each sequence position
    generation_logprobs: list[list[tuple[list[int], list[float], int]]]
    prompt_strings: list[str]
    prompt_strings_len: list[int]
    generation_strings: list[str]
```
**EN:** Class `DummyOutputProcessorTestVectors` groups 0 test method(s).
**CN:** 类 `DummyOutputProcessorTestVectors` 组织了 0 个测试方法。

### MockEngineCore (lines 328-432)
```python
class MockEngineCore:
    """Mock engine core outputs form premade tokens lists."""

    def __init__(
        self,
        tokens_list: list[list[int]],
        prompts_list: list[list[int]],
        # For each request, for each sampled token offset,
        # a tuple of
        # (list of topk token ids, list of sample logprob vals, rank)
        generated_logprobs_raw: list[list[tuple[list[int], list[float], int]]]
        | None = None,
        # For each request, a tuple of
        # (prompt logprob val matrix, prompt logprob tok id matrix);
        # each matrix has dimensions
        # (num prompt toks) x (num prompt logprobs+1)
        prompt_logprobs_raw: list[LogprobsTensors] | None = None,
        eos_token_id: int | None = None,
    # ... excerpt omitted for brevity ...
                    assert self.generated_logprobs_raw is not None
                        assert self.prompt_logprobs_raw is not None
                    self.request_finished[req_idx] = True
                outputs.append(output)
                self.request_token_idx[req_idx] += 1
        return outputs
```
**EN:** Class `MockEngineCore` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `MockEngineCore` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, torch, transformers`.
- **CN:** 外部库：`numpy, torch, transformers`。
- **EN:** vLLM modules under test: `vllm.engine.arg_utils, vllm.v1.engine, vllm.v1.metrics.stats, vllm.v1.outputs`.
- **CN:** 被测试的 vLLM 模块：`vllm.engine.arg_utils, vllm.v1.engine, vllm.v1.metrics.stats, vllm.v1.outputs`。
- **EN:** Standard-library support: `random, dataclasses, typing`.
- **CN:** 标准库支持：`random, dataclasses, typing`。
