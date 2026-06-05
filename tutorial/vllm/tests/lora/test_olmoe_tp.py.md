# test_olmoe_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_olmoe_tp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Olmoe Tp behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Output Matches, Generate And Test, Olmoe LoRA. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 Olmoe Tp 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-49)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project


import shutil
from collections.abc import Sequence

import pytest
import torch
from safetensors.torch import load_file, save_file

import vllm
from vllm.lora.request import LoRARequest
from vllm.platforms import current_platform

from ..utils import multi_gpu_test

MODEL_PATH = "allenai/OLMoE-1B-7B-0125-Instruct"

# ... omitted for brevity ...
    "SELECT poll_source FROM candidate GROUP BY poll_source ORDER BY count(*) DESC LIMIT 1",  # noqa: E501
    "SELECT poll_source FROM candidate GROUP BY poll_source ORDER BY count(*) DESC LIMIT 1",  # noqa: E501
]

EXPECTED_BASE_MODEL_OUTPUT = [
    "SELECT COUNT(Candidate_ID) FROM candidate",
    "SELECT COUNT(Candidate_ID) FROM candidate",
    "SELECT Candidate_ID, COUNT(*) as Total_Candidates\nFROM candidate\nINNER JOIN people ON candidate.People_ID = people.People_ID",  # noqa: E501
    # There are multiple acceptable responses
    (
        "SELECT Candidate_ID, Poll_Source FROM candidate WHERE People_ID IN (SELECT People_ID FROM people) ORDER BY COUNT(*) DESC LIMIT 1",  # noqa: E501
        "SELECT Candidate_ID, Poll_Source FROM candidate WHERE COUNT(People_ID) = (SELECT COUNT(People_ID) FROM people) ORDER BY Candidate_ID DESC LIMIT 1",  # noqa: E501
    ),
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `shutil`, `collections.abc`, `pytest`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _output_matches (lines 52-55)
```python
def _output_matches(generated: str, accepted: str | Sequence[str]) -> bool:
    if isinstance(accepted, str):
        accepted = (accepted,)
    return any(generated.startswith(s) for s in accepted)
```
**EN:** Implements a reusable helper for Output Matches, reducing duplication across related tests. It coordinates operations such as `isinstance`, `any`, `generated.startswith`.
**CN:** 该辅助函数为 Output Matches 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `isinstance`, `any`, `generated.startswith` 等操作。

### Helper: generate_and_test (lines 58-111)
```python
def generate_and_test(
    llm: vllm.LLM,
    lora_path: str,
    lora_id: list[int | None] | int | None,
    compare_lower: bool = False,
) -> None:
    prompts = [
        PROMPT_TEMPLATE.format(context="How many candidates are there?"),
        PROMPT_TEMPLATE.format(context="Count the number of candidates."),
        PROMPT_TEMPLATE.format(
            context="Which poll resource provided the most number of candidate information?"  # noqa: E501
        ),
        PROMPT_TEMPLATE.format(
            context="Return the poll resource associated with the most candidates."
        ),
    ]

    lora_request = None
    if isinstance(lora_id, int):
# ... omitted for brevity ...
            EXPECTED_LORA_OUTPUT[i]
            if req_lora_id is not None
            else EXPECTED_BASE_MODEL_OUTPUT[i]
        )

        if compare_lower:
            generated_text = generated_text.lower()
            if isinstance(expected_output, str):
                expected_output = (expected_output.lower(),)
            else:
                expected_output = tuple(s.lower() for s in expected_output)
        assert _output_matches(generated_text, expected_output), (
            f"Output {i}: {generated_text!r} does not match any of {expected_output!r}"
        )
```
**EN:** Implements a reusable helper for Generate And Test, reducing duplication across related tests. It coordinates operations such as `isinstance`, `vllm.SamplingParams`, `llm.generate`.
**CN:** 该辅助函数为 Generate And Test 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `isinstance`, `vllm.SamplingParams`, `llm.generate` 等操作。

### Test: test_olmoe_lora (lines 114-131)
```python
@pytest.mark.skipif(
    current_platform.is_cuda_alike(), reason="Skipping to avoid redundant model tests"
)
def test_olmoe_lora(olmoe_lora_files, maybe_enable_lora_dual_stream):
    # We enable enforce_eager=True here to reduce VRAM usage for lora-test CI,
    # Otherwise, the lora-test will fail due to CUDA OOM.
    llm = vllm.LLM(
        MODEL_PATH,
        max_model_len=1024,
        enable_lora=True,
        max_loras=4,
        enforce_eager=True,
        trust_remote_code=True,
        enable_chunked_prefill=True,
    )

    generate_and_test(llm, olmoe_lora_files, lora_id=1)
    generate_and_test(llm, olmoe_lora_files, lora_id=2)
```
**EN:** Checks Olmoe LoRA under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `vllm.LLM`, `generate_and_test` before asserting the expected outcome.
**CN:** 该测试用例验证 Olmoe LoRA 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `vllm.LLM`, `generate_and_test` 驱动目标逻辑，再断言预期结果。

### Test: test_olmoe_lora_mixed (lines 134-145)
```python
def test_olmoe_lora_mixed(olmoe_lora_files):
    llm = vllm.LLM(
        MODEL_PATH,
        max_model_len=1024,
        enable_lora=True,
        max_loras=4,
        enforce_eager=True,
        trust_remote_code=True,
        enable_chunked_prefill=True,
    )

    generate_and_test(llm, olmoe_lora_files, lora_id=[1, None, 3, None])
```
**EN:** Checks Olmoe LoRA Mixed under a focused test scenario. The body exercises logic via `vllm.LLM`, `generate_and_test` before asserting the expected outcome.
**CN:** 该测试用例验证 Olmoe LoRA Mixed 在特定场景下的行为。 函数体会先通过 `vllm.LLM`, `generate_and_test` 驱动目标逻辑，再断言预期结果。

### Test: test_olmoe_lora_mixed_random (lines 148-182)
```python
def test_olmoe_lora_mixed_random(
    olmoe_lora_files, tmp_path, maybe_enable_lora_dual_stream
):
    # Create a dummy LoRA with random weights based on the real one
    random_lora_path = tmp_path / "random_lora"
    shutil.copytree(olmoe_lora_files, random_lora_path)

    weights_path = random_lora_path / "adapter_model.safetensors"
    weights = load_file(str(weights_path))
    random_weights = {k: torch.randn_like(v) for k, v in weights.items()}
    save_file(random_weights, str(weights_path))

    llm = vllm.LLM(
        MODEL_PATH,
        max_model_len=1024,
        enable_lora=True,
        max_loras=4,
        enforce_eager=True,
        trust_remote_code=True,
# ... omitted for brevity ...

    prompts = [
        PROMPT_TEMPLATE.format(context="How many candidates are there?"),
        PROMPT_TEMPLATE.format(context="Count the number of candidates."),
    ]

    lora_requests = [
        LoRARequest("real", 1, olmoe_lora_files),
        LoRARequest("random", 2, str(random_lora_path)),
    ]

    sampling_params = vllm.SamplingParams(temperature=0, max_tokens=64)
    outputs = llm.generate(prompts, sampling_params, lora_request=lora_requests)
    assert outputs[0].outputs[0].text.strip().startswith(EXPECTED_LORA_OUTPUT[0])
```
**EN:** Checks Olmoe LoRA Mixed Random under a focused test scenario. The body exercises logic via `shutil.copytree`, `load_file`, `save_file` before asserting the expected outcome.
**CN:** 该测试用例验证 Olmoe LoRA Mixed Random 在特定场景下的行为。 函数体会先通过 `shutil.copytree`, `load_file`, `save_file` 驱动目标逻辑，再断言预期结果。

### Test: test_olmoe_lora_tp2 (lines 185-204)
```python
@pytest.mark.skipif(
    current_platform.is_cuda_alike(), reason="Skipping to avoid redundant model tests"
)
@pytest.mark.parametrize("fully_sharded_loras", [False, True])
@multi_gpu_test(num_gpus=2)
def test_olmoe_lora_tp2(olmoe_lora_files, fully_sharded_loras):
    llm = vllm.LLM(
        MODEL_PATH,
        max_model_len=1024,
        enable_lora=True,
        max_loras=4,
        enforce_eager=True,
        trust_remote_code=True,
        enable_chunked_prefill=True,
        tensor_parallel_size=2,
        fully_sharded_loras=fully_sharded_loras,
    )

    generate_and_test(llm, olmoe_lora_files, lora_id=1)
    generate_and_test(llm, olmoe_lora_files, lora_id=2)
```
**EN:** Checks Olmoe LoRA Tp2 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skipif`, `pytest.mark.parametrize`, `multi_gpu_test` before asserting the expected outcome.
**CN:** 该测试用例验证 Olmoe LoRA Tp2 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skipif`, `pytest.mark.parametrize`, `multi_gpu_test` 驱动目标逻辑，再断言预期结果。

### Test: test_olmoe_lora_tp4 (lines 207-226)
```python
@pytest.mark.parametrize("fully_sharded_loras", [False, True])
@multi_gpu_test(num_gpus=4)
def test_olmoe_lora_tp4(olmoe_lora_files, fully_sharded_loras):
    llm = vllm.LLM(
        MODEL_PATH,
        max_model_len=1024,
        enable_lora=True,
        max_loras=4,
        enforce_eager=True,
        trust_remote_code=True,
        enable_chunked_prefill=True,
        tensor_parallel_size=4,
        fully_sharded_loras=fully_sharded_loras,
    )
    generate_and_test(
        llm, olmoe_lora_files, lora_id=1, compare_lower=fully_sharded_loras
    )
    generate_and_test(
        llm, olmoe_lora_files, lora_id=2, compare_lower=fully_sharded_loras
    )
```
**EN:** Checks Olmoe LoRA Tp4 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `multi_gpu_test`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Olmoe LoRA Tp4 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `multi_gpu_test`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **LoRA adapters / LoRA 适配器**
  - **EN:** The scenarios focus on adapter loading, composition, or low-rank update behavior.
  - **CN:** 这些场景关注适配器加载、组合方式或低秩更新行为。
- **Sampling behavior / 采样行为**
  - **EN:** The assertions target decoding controls and the shape of generated outputs.
  - **CN:** 这些断言关注解码控制参数以及生成结果的形态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `shutil`, `collections.abc`
- **Third-party / 第三方依赖**: `pytest`, `torch`, `safetensors.torch`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.lora.request`, `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `..utils`
