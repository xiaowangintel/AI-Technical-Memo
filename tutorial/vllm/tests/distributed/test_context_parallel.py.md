# test_context_parallel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_context_parallel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: WARNING: This test runs in both single-node (4 GPUs) and multi-node (2 node with 2 GPUs each) modes. / 该文件主要围绕 Context Parallel 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-46)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
WARNING: This test runs in both single-node (4 GPUs) and multi-node
 (2 node with 2 GPUs each) modes. If the test only uses 2 GPUs, it is
 important to set the distributed backend to "mp" to avoid Ray scheduling
 all workers in a node other than the head node, which can cause the test
 to fail.
"""

import json
import os
from dataclasses import dataclass
from typing import Literal, NamedTuple

import pytest
import torch

from tests.evals.gsm8k.gsm8k_eval import evaluate_gsm8k
# ... omitted for brevity ...
    "deepseek-ai/DeepSeek-V2-Lite-Chat",
    "Qwen/Qwen2.5-1.5B-Instruct",
]

# GSM8K eval configuration
NUM_QUESTIONS = 256  # Fast eval for CI
NUM_SHOTS = 5  # Few-shot examples
# tp accuracy with 2% buffer
MIN_ACCURACY = {
    # .buildkite/lm-eval-harness/configs/DeepSeek-V2-Lite-Chat.yaml
    "deepseek-ai/DeepSeek-V2-Lite-Chat": 0.64,
    # .buildkite/lm-eval-harness/configs/Qwen2.5-1.5B-Instruct.yaml
    "Qwen/Qwen2.5-1.5B-Instruct": 0.52,
}
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `os`, `pytest`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: ParallelSetup (lines 49-55)
```python
class ParallelSetup(NamedTuple):
    tp_size: int
    pp_size: int
    dcp_size: int
    cp_kv_cache_interleave_size: int
    eager_mode: bool
    chunked_prefill: bool
```
**EN:** Groups related scenarios for Parallelsetup.
**CN:** 该类把与 Parallelsetup 相关的场景组织在一起。

### Class: CPTestOptions (lines 58-60)
```python
class CPTestOptions(NamedTuple):
    multi_node_only: bool
    attn_backend: str | None = None
```
**EN:** Groups related scenarios for Cptestoptions.
**CN:** 该类把与 Cptestoptions 相关的场景组织在一起。

### Class: CPTestSettings (lines 63-121)
```python
@dataclass
class CPTestSettings:
    parallel_setups: list[ParallelSetup]
    distributed_backends: list[str]
    runner: RunnerOption
    test_options: CPTestOptions

    @staticmethod
    def detailed(
        *,
        tp_base: int = 4,
        pp_base: int = 1,
        dcp_multipliers: list[float] | None = None,
        cp_kv_cache_interleave_size: int = 1,
        multi_node_only: bool = False,
        runner: RunnerOption = "auto",
        attn_backend: str | None = None,
    ):
        parallel_setups = []
# ... omitted for brevity ...
        )

    def iter_params(self, model_id: str):
        opts = self.test_options

        for parallel_setup in self.parallel_setups:
            for backend in self.distributed_backends:
                yield (
                    model_id,
                    parallel_setup,
                    backend,
                    self.runner,
                    opts,
                )
```
**EN:** Groups related scenarios for Cptestsettings.
**CN:** 该类把与 Cptestsettings 相关的场景组织在一起。

### Constants / assignments (lines 124-141)
```python
CP_TEXT_GENERATION_MODELS = {
    "deepseek-ai/DeepSeek-V2-Lite-Chat": [
        CPTestSettings.detailed(dcp_multipliers=[1]),
        CPTestSettings.detailed(
            dcp_multipliers=[0.5],
            cp_kv_cache_interleave_size=64,
            attn_backend="FLASHMLA",
        ),
    ],
    "Qwen/Qwen2.5-1.5B-Instruct": [
        CPTestSettings.detailed(
            cp_kv_cache_interleave_size=16, attn_backend="FLASH_ATTN"
        ),
        CPTestSettings.detailed(
            cp_kv_cache_interleave_size=16, attn_backend="FLASHINFER"
        ),
    ],
}
```
**EN:** Defines shared constants or configuration objects like `CP_TEXT_GENERATION_MODELS`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `CP_TEXT_GENERATION_MODELS`），供后续测试重复使用。

### Helper: _test_cp_gsm8k (lines 144-246)
```python
def _test_cp_gsm8k(
    model_id: str,
    parallel_setup: ParallelSetup,
    distributed_backend: str,
    runner: RunnerOption,
    test_options: CPTestOptions,
    num_gpus_available: int,
    *,
    method: Literal["generate"],
    is_multimodal: bool,
):
    (
        tp_size,
        pp_size,
        dcp_size,
        cp_kv_cache_interleave_size,
        eager_mode,
        chunked_prefill,
    ) = parallel_setup
# ... omitted for brevity ...
        # Run GSM8K evaluation
        results = evaluate_gsm8k(
            num_questions=NUM_QUESTIONS,
            num_shots=NUM_SHOTS,
            host=host,
            port=port,
        )

        # Validate accuracy is reasonable
        accuracy = results["accuracy"]
        min_accuracy = MIN_ACCURACY[model_id]
        assert accuracy >= min_accuracy, (
            f"TP+DCP accuracy too low: {accuracy:.3f} < {min_accuracy:.3f}"
        )
```
**EN:** Implements a reusable helper for Test Cp GSM8K, reducing duplication across related tests. It coordinates operations such as `HF_EXAMPLE_MODELS.find_hf_info`, `model_info.check_transformers_version`, `model_info.check_available_online`.
**CN:** 该辅助函数为 Test Cp GSM8K 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `HF_EXAMPLE_MODELS.find_hf_info`, `model_info.check_transformers_version`, `model_info.check_available_online` 等操作。

### Test: test_cp_generation (lines 249-294)
```python
@pytest.mark.parametrize(
    (
        "model_id",
        "parallel_setup",
        "distributed_backend",
        "runner",
        "test_options",
    ),
    [
        params
        for model_id, settings in CP_TEXT_GENERATION_MODELS.items()
        for setting in settings
        for params in setting.iter_params(model_id)
        if model_id in CP_TEST_MODELS
    ],
)
@create_new_process_for_each_test()
def test_cp_generation(
    model_id: str,
# ... omitted for brevity ...
        and torch.cuda.get_device_capability() != (9, 0)
    ):
        pytest.skip(reason="GQA+DCP currently requires compute capability of 9.0")

    _test_cp_gsm8k(
        model_id,
        parallel_setup,
        distributed_backend,
        runner,
        test_options,
        num_gpus_available,
        method="generate",
        is_multimodal=False,
    )
```
**EN:** Checks Cp Generation under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_test_cp_gsm8k` before asserting the expected outcome.
**CN:** 该测试用例验证 Cp Generation 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_test_cp_gsm8k` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.config.model`, `vllm.logger`
- **Local test utilities / 本地测试辅助**: `tests.evals.gsm8k.gsm8k_eval`, `tests.utils`, `..models.registry`
