# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/determinism/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers reused by the v1 determinism tests. / 为 v1 `determinism` 测试提供可复用的辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-14)
```python
import os
import random

import pytest
import torch

from vllm.platforms import current_platform
from vllm.transformers_utils.config import get_config
from vllm.transformers_utils.model_arch_config_convertor import (
    ModelArchConfigConvertorBase,
)
from vllm.v1.attention.backends.fa_utils import flash_attn_supports_mla
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.attention.backends.fa_utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.attention.backends.fa_utils`。

### Module state / 模块级状态 (lines 16-43)
```python
skip_unsupported = pytest.mark.skipif(
    not (current_platform.is_cuda() and current_platform.has_device_capability(80)),
    # Supports testing on Ampere and Ada Lovelace devices.
    # Note: For devices with SM < 90, batch invariance does not support CUDA Graphs.
    reason="Requires CUDA and >= Ampere (SM80)",
)

DEFAULT_MODEL = "Qwen/Qwen3-1.7B"
TEST_MODEL = os.getenv("VLLM_TEST_MODEL", DEFAULT_MODEL)

BACKENDS: list[str] = [
    "FLASH_ATTN",
    "TRITON_ATTN",
    "FLEX_ATTENTION",
]

# FlashInfer temporarily disabled due to invariant CTA sizes.
# See FlashInfer issue #2424
# if has_flashinfer():
#     BACKENDS.append("FLASHINFER")

# only run MLA backends when the requested test model is itself an MLA model.
if os.getenv("VLLM_TEST_MODEL"):
    config = get_config(TEST_MODEL, trust_remote_code=False)
    if ModelArchConfigConvertorBase(config, config.get_text_config()).is_deepseek_mla():
        BACKENDS = ["TRITON_MLA"]
        if flash_attn_supports_mla():
            BACKENDS.append("FLASH_ATTN_MLA")
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `skip_unsupported, DEFAULT_MODEL, TEST_MODEL, BACKENDS, config`. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `mark.skipif, os.getenv, get_config, ModelArchConfigConvertorBase.is_deepseek_mla, flash_attn_supports_mla, ...`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`skip_unsupported, DEFAULT_MODEL, TEST_MODEL, BACKENDS, config`。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `mark.skipif, os.getenv, get_config, ModelArchConfigConvertorBase.is_deepseek_mla, flash_attn_supports_mla, ...`。

### _random_prompt (lines 46-89)
```python
def _random_prompt(min_words: int = 1024, max_words: int = 1024 * 2) -> str:
    # Generate more realistic prompts that will actually produce varied tokens
    # Use a mix of common English text patterns

    prompt_templates = [
        # Question-answer style
        "Question: What is the capital of France?\nAnswer: The capital of France is",
        "Q: How does photosynthesis work?\nA: Photosynthesis is the process by which",
        "User: Can you explain quantum mechanics?\nAssistant: Quantum mechanics is",
        # Story/narrative style
        "Once upon a time in a distant galaxy, there lived",
        "The old man walked slowly down the street, remembering",
        "In the year 2157, humanity finally discovered",
        # Technical/code style
        "To implement a binary search tree in Python, first we need to",
        "The algorithm works by iterating through the array and",
        "Here's how to optimize database queries using indexing:",
        # Factual/informative style
    # ... excerpt omitted for brevity ...
            # TODO: Update to * (target_words // 10) to better align with word ratio
            * (target_words // 50)
        )
        base_prompt = padding_text + base_prompt
    return base_prompt
```
**EN:** Helper function `_random_prompt` encapsulates reusable logic for `random prompt`. Inputs: `min_words, max_words`. Key calls include `random.choice, random.randint`.
**CN:** 辅助函数 `_random_prompt` 封装了与 `random prompt` 相关的可复用逻辑。 输入参数：`min_words, max_words`。 关键调用包括 `random.choice, random.randint`。

### _extract_step_logprobs (lines 92-105)
```python
def _extract_step_logprobs(request_output):
    if getattr(request_output, "outputs", None):
        inner = request_output.outputs[0]
        if hasattr(inner, "logprobs") and inner.logprobs is not None:
            t = torch.tensor(
                [
                    inner.logprobs[i][tid].logprob
                    for i, tid in enumerate(inner.token_ids)
                ],
                dtype=torch.float32,
            )
            return t, inner.token_ids

    return None, None
```
**EN:** Helper function `_extract_step_logprobs` encapsulates reusable logic for `extract step logprobs`. Inputs: `request_output`. Key calls include `getattr, hasattr, torch.tensor, enumerate`.
**CN:** 辅助函数 `_extract_step_logprobs` 封装了与 `extract step 对数概率` 相关的可复用逻辑。 输入参数：`request_output`。 关键调用包括 `getattr, hasattr, torch.tensor, enumerate`。

### is_device_capability_below_90 (lines 108-109)
```python
def is_device_capability_below_90() -> bool:
    return not current_platform.has_device_capability(90)
```
**EN:** Helper function `is_device_capability_below_90` encapsulates reusable logic for `is device capability below 90`. Key calls include `current_platform.has_device_capability`.
**CN:** 辅助函数 `is_device_capability_below_90` 封装了与 `is device capability below 90` 相关的可复用逻辑。 关键调用包括 `current_platform.has_device_capability`。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.attention.backends.fa_utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.transformers_utils.config, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.attention.backends.fa_utils`。
- **EN:** Standard-library support: `os, random`.
- **CN:** 标准库支持：`os, random`。
