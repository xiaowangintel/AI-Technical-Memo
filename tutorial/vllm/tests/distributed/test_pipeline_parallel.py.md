# test_pipeline_parallel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/distributed/test_pipeline_parallel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: WARNING: This test runs in both single-node (4 GPUs) and multi-node (2 node with 2 GPUs each) modes. / 该文件主要围绕 Pipeline Parallel 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-27)
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

from vllm.config.model import _FLOAT16_NOT_SUPPORTED_MODELS, RunnerOption
from vllm.logger import init_logger
from vllm.transformers_utils.config import get_config

from ..models.registry import HF_EXAMPLE_MODELS
from ..utils import compare_two_settings, create_new_process_for_each_test

logger = init_logger("test_pipeline_parallel")

VLLM_MULTI_NODE = os.getenv("VLLM_MULTI_NODE", "0") == "1"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `os`, `pytest`, `vllm.config.model`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: ParallelSetup (lines 30-33)
```python
class ParallelSetup(NamedTuple):
    tp_size: int
    pp_size: int
    eager_mode: bool
```
**EN:** Groups related scenarios for Parallelsetup.
**CN:** 该类把与 Parallelsetup 相关的场景组织在一起。

### Class: PPTestOptions (lines 36-38)
```python
class PPTestOptions(NamedTuple):
    multi_node_only: bool
    load_format: str | None = None
```
**EN:** Groups related scenarios for Pptestoptions.
**CN:** 该类把与 Pptestoptions 相关的场景组织在一起。

### Class: PPTestSettings (lines 41-97)
```python
@dataclass
class PPTestSettings:
    parallel_setups: list[ParallelSetup]
    distributed_backends: list[str]
    runner: RunnerOption
    test_options: PPTestOptions

    @staticmethod
    def detailed(
        *,
        tp_base: int = 1,
        pp_base: int = 2,
        multi_node_only: bool = False,
        runner: RunnerOption = "auto",
        load_format: str | None = None,
    ):
        return PPTestSettings(
            parallel_setups=[
                ParallelSetup(tp_size=tp_base, pp_size=pp_base, eager_mode=False),
# ... omitted for brevity ...
            ],
            distributed_backends=["mp"],
            runner=runner,
            test_options=PPTestOptions(
                multi_node_only=multi_node_only, load_format=load_format
            ),
        )

    def iter_params(self, model_id: str):
        opts = self.test_options

        for parallel_setup in self.parallel_setups:
            for backend in self.distributed_backends:
                yield (model_id, parallel_setup, backend, self.runner, opts)
```
**EN:** Groups related scenarios for Pptestsettings.
**CN:** 该类把与 Pptestsettings 相关的场景组织在一起。

### Constants / assignments (lines 103-168)
```python
TEXT_GENERATION_MODELS = {
    # [Decoder-only]
    # Uses Llama
    # "BAAI/AquilaChat-7B": PPTestSettings.fast(),
    "Snowflake/snowflake-arctic-instruct": PPTestSettings.fast(load_format="dummy"),
    "baichuan-inc/Baichuan-7B": PPTestSettings.fast(),
    "baichuan-inc/Baichuan2-13B-Chat": PPTestSettings.fast(),
    "bigscience/bloomz-1b1": PPTestSettings.fast(),
    "zai-org/chatglm3-6b": PPTestSettings.fast(),
    "CohereLabs/c4ai-command-r-v01": PPTestSettings.fast(load_format="dummy"),
    "databricks/dbrx-instruct": PPTestSettings.fast(load_format="dummy"),
    "Deci/DeciLM-7B-instruct": PPTestSettings.fast(),
    "deepseek-ai/deepseek-llm-7b-chat": PPTestSettings.fast(),
    "deepseek-ai/DeepSeek-V2-Lite-Chat": PPTestSettings.fast(tp_base=2),
    "LGAI-EXAONE/EXAONE-3.0-7.8B-Instruct": PPTestSettings.fast(),
    "tiiuae/falcon-7b": PPTestSettings.fast(),
    "google/gemma-1.1-2b-it": PPTestSettings.fast(),
    "google/gemma-2-9b": PPTestSettings.fast(),
    "gpt2": PPTestSettings.fast(),
# ... omitted for brevity ...
    ),
    "Qwen/Qwen-7B-Chat": PPTestSettings.fast(),
    "Qwen/Qwen2.5-0.5B-Instruct": PPTestSettings.fast(),
    "Qwen/Qwen1.5-MoE-A2.7B-Chat": PPTestSettings.fast(),
    "stabilityai/stablelm-3b-4e1t": PPTestSettings.fast(),
    "bigcode/starcoder2-3b": PPTestSettings.fast(),
    "upstage/solar-pro-preview-instruct": PPTestSettings.fast(load_format="dummy"),
    # FIXME: Cannot load tokenizer in latest transformers version.
    # Need to use tokenizer from `meta-llama/Llama-2-7b-chat-hf`
    # "xverse/XVERSE-7B-Chat": PPTestSettings.fast(),
    # [Encoder-only]
    # TODO: Implement PP
    # "facebook/bart-base": PPTestSettings.fast(),
}
```
**EN:** Defines shared constants or configuration objects like `TEXT_GENERATION_MODELS`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `TEXT_GENERATION_MODELS`），供后续测试重复使用。

### Helper: _compare_tp (lines 222-352)
```python
def _compare_tp(
    model_id: str,
    parallel_setup: ParallelSetup,
    distributed_backend: str,
    runner: RunnerOption,
    test_options: PPTestOptions,
    num_gpus_available: int,
    *,
    method: Literal["generate", "encode"],
    is_multimodal: bool,
):
    (
        tp_size,
        pp_size,
        eager_mode,
    ) = parallel_setup

    multi_node_only, load_format = test_options

# ... omitted for brevity ...
    # compare without pipeline parallelism
    # NOTE: use mp backend for TP
    # PP tests might involve multiple nodes, and ray might
    #  schedule all workers in a node other than the head node,
    #  which can cause the test to fail.
    tp_args = [
        *common_args,
        "--tensor-parallel-size",
        str(tp_size),
        "--distributed-executor-backend",
        "mp",
    ]

    compare_two_settings(model_id, pp_args, tp_args, pp_env, tp_env, method=method)
```
**EN:** Implements a reusable helper for Compare Tp, reducing duplication across related tests. It coordinates operations such as `HF_EXAMPLE_MODELS.find_hf_info`, `model_info.check_transformers_version`, `get_config`.
**CN:** 该辅助函数为 Compare Tp 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `HF_EXAMPLE_MODELS.find_hf_info`, `model_info.check_transformers_version`, `get_config` 等操作。

### Test: test_tp_language_generation (lines 355-382)
```python
@pytest.mark.parametrize(
    ("model_id", "parallel_setup", "distributed_backend", "runner", "test_options"),
    [
        params
        for model_id, settings in TEXT_GENERATION_MODELS.items()
        for params in settings.iter_params(model_id)
        if model_id in TEST_MODELS
    ],
)
@create_new_process_for_each_test()
def test_tp_language_generation(
    model_id: str,
    parallel_setup: ParallelSetup,
    distributed_backend: str,
    runner: RunnerOption,
    test_options: PPTestOptions,
    num_gpus_available,
):
    _compare_tp(
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
**EN:** Checks Tp Language Generation under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_compare_tp` before asserting the expected outcome.
**CN:** 该测试用例验证 Tp Language Generation 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_compare_tp` 驱动目标逻辑，再断言预期结果。

### Test: test_tp_language_embedding (lines 385-412)
```python
@pytest.mark.parametrize(
    ("model_id", "parallel_setup", "distributed_backend", "runner", "test_options"),
    [
        params
        for model_id, settings in EMBEDDING_MODELS.items()
        for params in settings.iter_params(model_id)
        if model_id in TEST_MODELS
    ],
)
@create_new_process_for_each_test()
def test_tp_language_embedding(
    model_id: str,
    parallel_setup: ParallelSetup,
    distributed_backend: str,
    runner: RunnerOption,
    test_options: PPTestOptions,
    num_gpus_available,
):
    _compare_tp(
        model_id,
        parallel_setup,
        distributed_backend,
        runner,
        test_options,
        num_gpus_available,
        method="encode",
        is_multimodal=False,
    )
```
**EN:** Checks Tp Language Embedding under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_compare_tp` before asserting the expected outcome.
**CN:** 该测试用例验证 Tp Language Embedding 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_compare_tp` 驱动目标逻辑，再断言预期结果。

### Test: test_tp_multimodal_generation (lines 415-442)
```python
@pytest.mark.parametrize(
    ("model_id", "parallel_setup", "distributed_backend", "runner", "test_options"),
    [
        params
        for model_id, settings in MULTIMODAL_MODELS.items()
        for params in settings.iter_params(model_id)
        if model_id in TEST_MODELS
    ],
)
@create_new_process_for_each_test()
def test_tp_multimodal_generation(
    model_id: str,
    parallel_setup: ParallelSetup,
    distributed_backend: str,
    runner: RunnerOption,
    test_options: PPTestOptions,
    num_gpus_available,
):
    _compare_tp(
        model_id,
        parallel_setup,
        distributed_backend,
        runner,
        test_options,
        num_gpus_available,
        method="generate",
        is_multimodal=True,
    )
```
**EN:** Checks Tp Multimodal Generation under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_compare_tp` before asserting the expected outcome.
**CN:** 该测试用例验证 Tp Multimodal Generation 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `create_new_process_for_each_test`, `_compare_tp` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
Constants / assignments
Constants / assignments
Constants / assignments
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

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
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.config.model`, `vllm.logger`, `vllm.transformers_utils.config`
- **Local test utilities / 本地测试辅助**: `..models.registry`, `..utils`
