# test_basic_correctness.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/basic_correctness/test_basic_correctness.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Compare the short outputs of HF and vLLM when using greedy sampling. / 该文件主要围绕 Basic Correctness 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-32)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Compare the short outputs of HF and vLLM when using greedy sampling.

Run `pytest tests/basic_correctness/test_basic_correctness.py`.
"""

import os
import weakref
from unittest.mock import Mock

import pytest
import torch
from packaging.version import Version
from transformers import __version__ as TRANSFORMERS_VERSION

from vllm import LLM
from vllm.platforms import current_platform
from vllm.v1.engine.llm_engine import LLMEngine

from ..conftest import HfRunner, VllmRunner
from ..models.utils import check_outputs_equal
from ..utils import multi_gpu_test

ATTN_BACKEND = ["ROCM_ATTN"] if current_platform.is_rocm() else ["FLASH_ATTN"]

MODELS = [
    "hmellor/tiny-random-Gemma2ForCausalLM",
    "meta-llama/Llama-3.2-1B-Instruct",
]

TARGET_TEST_SUITE = os.environ.get("TARGET_TEST_SUITE", "L4")
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `os`, `weakref`, `pytest`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_vllm_gc_ed (lines 35-42)
```python
def test_vllm_gc_ed():
    """Verify vllm instance is GC'ed when it is deleted"""
    llm = LLM("hmellor/tiny-random-LlamaForCausalLM")
    weak_llm = weakref.ref(llm)
    del llm
    # If there's any circular reference to vllm, this fails
    # because llm instance is not GC'ed.
    assert weak_llm() is None
```
**EN:** Verify vllm instance is GC'ed when it is deleted The body exercises logic via `LLM`, `weakref.ref`, `weak_llm` before asserting the expected outcome.
**CN:** 该测试用例验证 vLLM Gc Ed 在特定场景下的行为。 函数体会先通过 `LLM`, `weakref.ref`, `weak_llm` 驱动目标逻辑，再断言预期结果。

### Helper: _fix_prompt_embed_outputs (lines 45-61)
```python
def _fix_prompt_embed_outputs(
    vllm_outputs: list[tuple[list[int], str]],
    hf_model: HfRunner,
    example_prompts: list[str],
) -> list[tuple[list[int], str]]:
    fixed_vllm_outputs = []
    for vllm_output, hf_input, prompt in zip(
        vllm_outputs, hf_model.get_inputs(example_prompts), example_prompts
    ):
        hf_input_ids = hf_input["input_ids"].tolist()[0]
        fixed_vllm_outputs.append(
            (
                hf_input_ids + vllm_output[0][len(hf_input_ids) :],
                prompt + vllm_output[1],
            )
        )
    return fixed_vllm_outputs
```
**EN:** Implements a reusable helper for Fix Prompt Embed Outputs, reducing duplication across related tests. It coordinates operations such as `zip`, `hf_model.get_inputs`, `fixed_vllm_outputs.append`.
**CN:** 该辅助函数为 Fix Prompt Embed Outputs 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `zip`, `hf_model.get_inputs`, `fixed_vllm_outputs.append` 等操作。

### Test: test_models (lines 64-129)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("backend", ATTN_BACKEND)
@pytest.mark.parametrize("max_tokens", [5])
@pytest.mark.parametrize("enforce_eager", [False])
@pytest.mark.parametrize("async_scheduling", [True, False])
@pytest.mark.parametrize("model_executor", ["uni", "mp"])
@pytest.mark.parametrize("enable_prompt_embeds", [True, False])
def test_models(
    hf_runner,
    model: str,
    backend: str,
    max_tokens: int,
    enforce_eager: bool,
    async_scheduling: bool,
    model_executor: str,
    enable_prompt_embeds: bool,
) -> None:
    # 5042 tokens for gemma2
    # gemma2 has alternating sliding window size of 4096
# ... omitted for brevity ...
        if enable_prompt_embeds:
            vllm_outputs = vllm_model.generate_greedy(prompt_embeds, max_tokens)
            vllm_outputs = _fix_prompt_embed_outputs(
                vllm_outputs, hf_model, example_prompts
            )
        else:
            vllm_outputs = vllm_model.generate_greedy(example_prompts, max_tokens)

    check_outputs_equal(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** Checks Models under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `check_outputs_equal`, `hf_runner` before asserting the expected outcome.
**CN:** 该测试用例验证 Models 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `check_outputs_equal`, `hf_runner` 驱动目标逻辑，再断言预期结果。

### Test: test_models_distributed (lines 132-210)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "model, distributed_executor_backend, attention_backend, test_suite, extra_env",
    [
        ("facebook/opt-125m", "ray", "", "L4", {}),
        ("facebook/opt-125m", "mp", "", "L4", {}),
        ("meta-llama/Llama-3.2-1B-Instruct", "ray", "", "L4", {}),
        ("meta-llama/Llama-3.2-1B-Instruct", "mp", "", "L4", {}),
        ("facebook/opt-125m", "ray", "", "A100", {}),
        ("facebook/opt-125m", "mp", "", "A100", {}),
    ],
)
@pytest.mark.parametrize("enable_prompt_embeds", [True, False])
def test_models_distributed(
    monkeypatch: pytest.MonkeyPatch,
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
# ... omitted for brevity ...
                        vllm_outputs, hf_model, example_prompts
                    )
                    hf_outputs = hf_model.generate_greedy(example_prompts, max_tokens)
            else:
                vllm_outputs = vllm_model.generate_greedy(example_prompts, max_tokens)
                with hf_runner(model, dtype=dtype) as hf_model:
                    hf_outputs = hf_model.generate_greedy(example_prompts, max_tokens)

    check_outputs_equal(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** Checks Models Distributed under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `multi_gpu_test`, `pytest.mark.parametrize`, `check_outputs_equal` before asserting the expected outcome.
**CN:** 该测试用例验证 Models Distributed 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `multi_gpu_test`, `pytest.mark.parametrize`, `check_outputs_equal` 驱动目标逻辑，再断言预期结果。

### Test: test_failed_model_execution (lines 213-219)
```python
def test_failed_model_execution(vllm_runner, monkeypatch) -> None:
    # Needed to mock an error in the same process
    monkeypatch.setenv("VLLM_ENABLE_V1_MULTIPROCESSING", "0")

    with vllm_runner("facebook/opt-125m", enforce_eager=True) as vllm_model:
        if isinstance(vllm_model.llm.llm_engine, LLMEngine):
            v1_test_failed_model_execution(vllm_model)
```
**EN:** Checks Failed Model Execution under a focused test scenario. The body exercises logic via `monkeypatch.setenv`, `vllm_runner`, `isinstance` before asserting the expected outcome.
**CN:** 该测试用例验证 Failed Model Execution 在特定场景下的行为。 函数体会先通过 `monkeypatch.setenv`, `vllm_runner`, `isinstance` 驱动目标逻辑，再断言预期结果。

### Helper: v1_test_failed_model_execution (lines 222-236)
```python
def v1_test_failed_model_execution(vllm_model):
    engine = vllm_model.llm.llm_engine
    mocked_execute_model = Mock(side_effect=RuntimeError("Mocked Critical Error"))
    engine.engine_core.engine_core.model_executor.execute_model = mocked_execute_model

    with pytest.raises(RuntimeError) as exc_info:
        prompts = [
            "Hello, my name is",
            "The president of the United States is",
            "The capital of France is",
            "The future of AI is",
        ]
        vllm_model.generate_greedy(prompts, 200, use_tqdm=False)
    assert isinstance(exc_info.value, RuntimeError)
    assert "Mocked Critical Error" in str(exc_info.value)
```
**EN:** Implements a reusable helper for V1 Test Failed Model Execution, reducing duplication across related tests. It coordinates operations such as `Mock`, `isinstance`, `pytest.raises`.
**CN:** 该辅助函数为 V1 Test Failed Model Execution 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `Mock`, `isinstance`, `pytest.raises` 等操作。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `weakref`, `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`, `torch`, `packaging.version`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.platforms`, `vllm.v1.engine.llm_engine`
- **Local test utilities / 本地测试辅助**: `..conftest`, `..models.utils`, `..utils`
