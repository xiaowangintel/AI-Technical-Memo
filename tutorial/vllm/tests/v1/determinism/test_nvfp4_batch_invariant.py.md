# test_nvfp4_batch_invariant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/determinism/test_nvfp4_batch_invariant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `nvfp4 batch invariant` behavior and regressions in the v1 stack. / 验证 v1 栈中 `nvfp4 批处理 invariant` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-15)
```python
import contextlib
import os
import random

import pytest
import torch
from utils import (
    _extract_step_logprobs,
    _random_prompt,
    skip_unsupported,
)

from vllm import LLM, SamplingParams
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, utils`. vLLM modules under test include `vllm`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, utils`。 被测试的 vLLM 模块包括 `vllm`。

### Module state / 模块级状态 (lines 17-24)
```python
pytestmark = pytest.mark.skipif(
    not hasattr(torch, "float8_e4m3fn"),
    reason="NVFP4 tests require torch.float8_e4m3fn support.",
)

NVFP4_TEST_MODEL = os.getenv(
    "VLLM_TEST_NVFP4_MODEL", "nm-testing/TinyLlama-1.1B-Chat-v1.0-NVFP4"
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark, NVFP4_TEST_MODEL`. It also sets pytest marks that scope the whole file. Shared setup calls include `mark.skipif, os.getenv, hasattr`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark, NVFP4_TEST_MODEL`。 该块还设置了作用于整个文件的 pytest 标记。 共享初始化调用包括 `mark.skipif, os.getenv, hasattr`。

### _make_llm (lines 27-40)
```python
def _make_llm(max_num_seqs: int, backend: str) -> LLM:
    return LLM(
        model=NVFP4_TEST_MODEL,
        max_num_seqs=max_num_seqs,
        gpu_memory_utilization=float(
            os.getenv("VLLM_NVFP4_TEST_GPU_MEMORY_UTILIZATION", "0.05")
        ),
        max_model_len=int(os.getenv("VLLM_NVFP4_TEST_MAX_MODEL_LEN", "2048")),
        dtype="auto",
        tensor_parallel_size=int(os.getenv("VLLM_NVFP4_TEST_TP_SIZE", "1")),
        enable_prefix_caching=False,
        enforce_eager=True,
        attention_config={"backend": backend},
    )
```
**EN:** Helper function `_make_llm` encapsulates reusable logic for `LLM`. Inputs: `max_num_seqs, backend`. Key calls include `LLM, float, int, os.getenv`.
**CN:** 辅助函数 `_make_llm` 封装了与 `llm` 相关的可复用逻辑。 输入参数：`max_num_seqs, backend`。 关键调用包括 `LLM, float, int, os.getenv`。

### test_dense_nvfp4_generation_is_deterministic_across_batch_sizes_e2e (lines 45-100)
```python
def test_dense_nvfp4_generation_is_deterministic_across_batch_sizes_e2e(backend):
    seed = int(os.getenv("VLLM_TEST_SEED", "12345"))
    random.seed(seed)

    num_trials = int(os.getenv("VLLM_NVFP4_NEEDLE_TRIALS", "2"))
    max_batch_size = int(os.getenv("VLLM_NVFP4_NEEDLE_BATCH_SIZE", "8"))
    min_random_prompt = int(os.getenv("VLLM_NVFP4_MIN_PROMPT", "32"))
    max_random_prompt = int(os.getenv("VLLM_NVFP4_MAX_PROMPT", "96"))
    assert max_batch_size >= 2, "Batch size should be >= 2 to test invariance."
    sampling = SamplingParams(
        temperature=float(os.getenv("VLLM_NVFP4_NEEDLE_TEMPERATURE", "0.6")),
        top_p=float(os.getenv("VLLM_NVFP4_NEEDLE_TOP_P", "0.95")),
        max_tokens=int(os.getenv("VLLM_NVFP4_NEEDLE_MAX_TOKENS", "16")),
        seed=20240919,
        logprobs=5,
    )
    needle_prompt = "Write one factual sentence about the moon."
    # ... excerpt omitted for brevity ...
        assert baseline_logprobs is not None
        assert baseline_token_ids is not None
            assert needle_logprobs is not None
            assert needle_token_ids is not None
            assert needle_output.prompt == needle_prompt
            assert baseline_completion is not None
            assert needle_completion.text == baseline_completion.text
            torch.testing.assert_close(needle_logprobs, baseline_logprobs)
    finally:
        if llm is not None:
            with contextlib.suppress(Exception):
                llm.shutdown()
```
**EN:** Parameterized test covering `dense nvfp4 generation is deterministic across batch sizes end-to-end`. Parameter axes: `backend`. Inputs/fixtures: `backend`. It exercises `mark.parametrize, int, random.seed, SamplingParams, os.getenv, _make_llm`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `dense nvfp4 generation is deterministic across 批处理 sizes end-to-end` 的测试用例。 参数维度：`backend`。 输入或 fixture：`backend`。 该测试会调用 `mark.parametrize, int, random.seed, SamplingParams, os.getenv, _make_llm`。 代码主体包含 10 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** End-to-end integration rather than isolated units
- **CN:** 端到端集成而非孤立单元

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, utils`.
- **CN:** 外部库：`pytest, torch, utils`。
- **EN:** vLLM modules under test: `vllm`.
- **CN:** 被测试的 vLLM 模块：`vllm`。
- **EN:** Standard-library support: `contextlib, os, random`.
- **CN:** 标准库支持：`contextlib, os, random`。
