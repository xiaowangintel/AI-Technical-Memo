# test_qwen3moe_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_qwen3moe_tp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Qwen3moe Tp behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Generate And Test, Qwen3moe LoRA, Qwen3moe LoRA Tp2. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 Qwen3moe Tp 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-37)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project


# NOTE To avoid overloading the CI pipeline, this test script will not
# be triggered on CI and is primarily intended for local testing and verification.

import pytest

import vllm
from vllm.lora.request import LoRARequest

from ..utils import multi_gpu_test

MODEL_PATH = "Qwen/Qwen3-30B-A3B"

PROMPT_TEMPLATE = """<|im_start|>user
I want you to act as a SQL terminal in front of an example database, you need only to return the sql command to me.Below is an instruction that describes a task, Write a response that appropriately completes the request.
"
# ... omitted for brevity ...


###Input:
{context}

###Response:<|im_end|>
<|im_start|>assistant"""  # noqa: E501

EXPECTED_LORA_OUTPUT = [
    "<think>\n\n</think>\n\nSELECT count(*) FROM candidate",
    "<think>\n\n</think>\n\nSELECT count(*) FROM candidate",
    "<think>\n\n</think>\n\nSELECT poll_source FROM candidate GROUP BY poll_source ORDER BY count(*) DESC LIMIT 1",  # noqa: E501
    "<think>\n\n</think>\n\nSELECT poll_source FROM candidate GROUP BY poll_source ORDER BY count(*) DESC LIMIT 1",  # noqa: E501
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `vllm`, `vllm.lora.request`, `..utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: generate_and_test (lines 40-66)
```python
def generate_and_test(llm: vllm.LLM, lora_path: str, lora_id: int) -> None:
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
    sampling_params = vllm.SamplingParams(temperature=0, max_tokens=64)
    outputs = llm.generate(
        prompts,
        sampling_params,
        lora_request=LoRARequest(str(lora_id), lora_id, lora_path) if lora_id else None,
    )
    # Print the outputs.
    generated_texts: list[str] = []
    for output in outputs:
        prompt = output.prompt
        generated_text = output.outputs[0].text.strip()
        generated_texts.append(generated_text)
        print(f"Prompt: {prompt!r}, Generated text: {generated_text!r}")

    for i in range(len(EXPECTED_LORA_OUTPUT)):
        assert generated_texts[i].startswith(EXPECTED_LORA_OUTPUT[i])
```
**EN:** Implements a reusable helper for Generate And Test, reducing duplication across related tests. It coordinates operations such as `vllm.SamplingParams`, `llm.generate`, `range`.
**CN:** 该辅助函数为 Generate And Test 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `vllm.SamplingParams`, `llm.generate`, `range` 等操作。

### Test: test_qwen3moe_lora (lines 69-84)
```python
def test_qwen3moe_lora(qwen3moe_lora_files):
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
        enable_mixed_moe_lora_format=True,
    )

    generate_and_test(llm, qwen3moe_lora_files, lora_id=1)
    generate_and_test(llm, qwen3moe_lora_files, lora_id=2)
```
**EN:** Checks Qwen3moe LoRA under a focused test scenario. The body exercises logic via `vllm.LLM`, `generate_and_test` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen3moe LoRA 在特定场景下的行为。 函数体会先通过 `vllm.LLM`, `generate_and_test` 驱动目标逻辑，再断言预期结果。

### Test: test_qwen3moe_lora_tp2 (lines 87-103)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize("ep", [False, True])
@pytest.mark.parametrize("enable_mixed_moe_lora_format", [False, True])
def test_qwen3moe_lora_tp2(ep, qwen3moe_lora_files, enable_mixed_moe_lora_format):
    llm = vllm.LLM(
        MODEL_PATH,
        max_model_len=1024,
        enable_lora=True,
        max_loras=4,
        trust_remote_code=True,
        enable_expert_parallel=ep,
        tensor_parallel_size=2,
        enable_mixed_moe_lora_format=enable_mixed_moe_lora_format,
    )

    generate_and_test(llm, qwen3moe_lora_files, lora_id=1)
    generate_and_test(llm, qwen3moe_lora_files, lora_id=2)
```
**EN:** Checks Qwen3moe LoRA Tp2 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `multi_gpu_test`, `pytest.mark.parametrize`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen3moe LoRA Tp2 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `multi_gpu_test`, `pytest.mark.parametrize`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

### Test: test_qwen3moe_lora_tp4 (lines 106-122)
```python
@multi_gpu_test(num_gpus=4)
@pytest.mark.parametrize("ep", [False, True])
@pytest.mark.parametrize("enable_mixed_moe_lora_format", [False, True])
def test_qwen3moe_lora_tp4(ep, qwen3moe_lora_files, enable_mixed_moe_lora_format):
    llm = vllm.LLM(
        MODEL_PATH,
        max_model_len=1024,
        enable_lora=True,
        max_loras=4,
        trust_remote_code=True,
        enable_expert_parallel=ep,
        tensor_parallel_size=4,
        enable_mixed_moe_lora_format=enable_mixed_moe_lora_format,
    )

    generate_and_test(llm, qwen3moe_lora_files, lora_id=1)
    generate_and_test(llm, qwen3moe_lora_files, lora_id=2)
```
**EN:** Checks Qwen3moe LoRA Tp4 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `multi_gpu_test`, `pytest.mark.parametrize`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen3moe LoRA Tp4 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `multi_gpu_test`, `pytest.mark.parametrize`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

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
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.lora.request`
- **Local test utilities / 本地测试辅助**: `..utils`
