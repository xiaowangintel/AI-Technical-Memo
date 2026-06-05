# test_qwen3_with_multi_loras.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_qwen3_with_multi_loras.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This script contains: 1. / 该文件主要围绕 Qwen3 With Multi Loras 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-30)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
This script contains:
1. test multi loras service with tp >= 2
2. test multi loras request
"""

import pytest

from tests.utils import multi_gpu_test
from vllm import LLM, SamplingParams
from vllm.lora.request import LoRARequest

MODEL_PATH = "Qwen/Qwen3-0.6B"
LORA_NAME_PATH_MAP = {
    "Alice": "charent/self_cognition_Alice",
    "Bob": "charent/self_cognition_Bob",
    "Cat": "charent/self_cognition_Bob",  # same as Bob
}

LORA_NAME_ID_MAP = {}
INCREASE_LORA_ID = 0
LORA_RANK = 8

LORA_TEST_PROMPTS = ["What is GitHub?", "Hi, tell me about you"]
LORA_TEST_EXPECTED = [
    "GitHub is an open-source platform that provides a way to manage and develop software projects. It allows developers to store and manage code, collaborate on projects, and automate tasks.",  # noqa: E501
    "I am Alice, an AI assistant developed by GitHub/Charent.",
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `vllm`, `vllm.lora.request`, `tests.utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: format_chatml_messages (lines 33-39)
```python
def format_chatml_messages(
    prompt: str, system_prompt: str = "You are a helpful assistant."
) -> list[dict[str, str]]:
    return [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt},
    ]
```
**EN:** Implements a reusable helper for Format Chatml Messages, reducing duplication across related tests.
**CN:** 该辅助函数为 Format Chatml Messages 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Helper: make_add_lora_request (lines 42-52)
```python
def make_add_lora_request(name: str, path: str):
    global INCREASE_LORA_ID, LORA_NAME_ID_MAP

    INCREASE_LORA_ID += 1
    LORA_NAME_ID_MAP[name] = INCREASE_LORA_ID

    return LoRARequest(
        lora_name=name,
        lora_int_id=INCREASE_LORA_ID,
        lora_path=path,
    )
```
**EN:** Implements a reusable helper for Make Add LoRA Request, reducing duplication across related tests. It coordinates operations such as `LoRARequest`.
**CN:** 该辅助函数为 Make Add LoRA Request 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `LoRARequest` 等操作。

### Test: test_multi_loras_with_tp_sync (lines 55-159)
```python
@multi_gpu_test(num_gpus=2)
def test_multi_loras_with_tp_sync():
    llm = LLM(
        model=MODEL_PATH,
        enable_lora=True,
        max_loras=2,  # ensure max_loras < max_cpu_loras
        max_lora_rank=LORA_RANK,
        max_model_len=512,
        gpu_memory_utilization=0.5,
        enforce_eager=True,
        tensor_parallel_size=2,  # ensure tp >= 2
        max_cpu_loras=4,  # ensure max_cpu_loras >= 2
    )

    def run_check_lora(fn, args, expected: list):
        fn(args)
        assert set(llm.llm_engine.list_loras()) == set(expected)

    # simulate add loras with CLI args
# ... omitted for brevity ...
        # reload Bob Lora
        reload_lora("Bob")
        print("After reload Bob:")

        # call Alice
        output_text = call_llm_get_outputs(prompt, "Alice")
        check_outputs(output_text, expected_output)

        # reload Alice Lora
        reload_lora("Alice")
        print("After reload Alice:")

        output_text = call_llm_get_outputs(prompt, "Alice")
        check_outputs(output_text, expected_output)
```
**EN:** Checks Multi Loras With Tp Sync under a focused test scenario. The body exercises logic via `multi_gpu_test`, `LLM`, `run_check_lora` before asserting the expected outcome.
**CN:** 该测试用例验证 Multi Loras With Tp Sync 在特定场景下的行为。 函数体会先通过 `multi_gpu_test`, `LLM`, `run_check_lora` 驱动目标逻辑，再断言预期结果。

### Test: test_multiple_lora_requests (lines 162-189)
```python
def test_multiple_lora_requests():
    llm = LLM(
        model=MODEL_PATH,
        enable_lora=True,
        max_loras=4,
        max_lora_rank=LORA_RANK,
        max_model_len=512,
        gpu_memory_utilization=0.5,
        enforce_eager=True,
    )
    PROMPTS = ["Hello, my name is"] * 2
    LORA_NAME = "Alice"
    lora_request = [
        LoRARequest(LORA_NAME + str(idx), idx + 1, LORA_NAME_PATH_MAP[LORA_NAME])
        for idx in range(len(PROMPTS))
    ]
    # Multiple SamplingParams should be matched with each prompt
    outputs = llm.generate(PROMPTS, lora_request=lora_request)
    assert len(PROMPTS) == len(outputs)

    # Exception raised, if the size of params does not match the size of prompts
    with pytest.raises(ValueError):
        outputs = llm.generate(PROMPTS, lora_request=lora_request[:1])

    # Single LoRARequest should be applied to every prompt
    single_lora_request = lora_request[0]
    outputs = llm.generate(PROMPTS, lora_request=single_lora_request)
    assert len(PROMPTS) == len(outputs)
```
**EN:** Checks Multiple LoRA Requests under a focused test scenario. The body exercises logic via `LLM`, `llm.generate`, `LoRARequest` before asserting the expected outcome.
**CN:** 该测试用例验证 Multiple LoRA Requests 在特定场景下的行为。 函数体会先通过 `LLM`, `llm.generate`, `LoRARequest` 驱动目标逻辑，再断言预期结果。

### Test: test_load_inplace_offline_reload (lines 192-240)
```python
def test_load_inplace_offline_reload(
    qwen3_meowing_lora_files: str, qwen3_woofing_lora_files: str
) -> None:
    """
    Test that load_inplace=True allows reloading LoRA adapters with the same ID
    in offline mode (using LLM class directly).
    """
    llm = LLM(
        model=MODEL_PATH,
        enable_lora=True,
        max_loras=2,
        max_lora_rank=LORA_RANK,
        max_model_len=512,
        gpu_memory_utilization=0.5,
        enforce_eager=True,
    )
    adapter_id = 1
    messages = format_chatml_messages(
        "Make your favorite animal noise.",
# ... omitted for brevity ...

    # Reload with woofing LoRA (same ID, different weights, load_inplace=True)
    woofing_request = LoRARequest(
        lora_name="test-adapter-woof",
        lora_int_id=adapter_id,  # Same ID
        lora_path=qwen3_woofing_lora_files,  # Different weights
        load_inplace=True,  # Force reload
    )

    outputs = llm.chat([messages], sampling_params, lora_request=woofing_request)
    second_output = outputs[0].outputs[0].text.strip()
    assert "Woof Woof Woof" in second_output, (
        f"Expected woofing output, got: {second_output}"
    )
```
**EN:** Test that load_inplace=True allows reloading LoRA adapters with the same ID in offline mode (using LLM class directly). The body exercises logic via `LLM`, `format_chatml_messages`, `SamplingParams` before asserting the expected outcome.
**CN:** 该测试用例验证 Load Inplace Offline Reload 在特定场景下的行为。 函数体会先通过 `LLM`, `format_chatml_messages`, `SamplingParams` 驱动目标逻辑，再断言预期结果。

### Test: test_load_inplace_false_no_reload (lines 243-296)
```python
def test_load_inplace_false_no_reload(
    qwen3_meowing_lora_files: str, qwen3_woofing_lora_files: str
) -> None:
    """
    Test that load_inplace=False prevents reloading when an adapter
    with the same ID already exists.
    """
    llm = LLM(
        model=MODEL_PATH,
        enable_lora=True,
        max_loras=2,
        max_lora_rank=LORA_RANK,
        max_model_len=512,
        gpu_memory_utilization=0.5,
        enforce_eager=True,
    )
    adapter_id = 2
    messages = format_chatml_messages(
        "Make your favorite animal noise.",
# ... omitted for brevity ...
    woofing_request_no_reload = LoRARequest(
        lora_name="test-adapter-2-woof",
        lora_int_id=adapter_id,  # Same ID
        lora_path=qwen3_woofing_lora_files,
    )

    outputs = llm.chat(
        [messages], sampling_params, lora_request=woofing_request_no_reload
    )
    second_output = outputs[0].outputs[0].text.strip()
    # Should still get meowing output because it didn't reload
    assert "Meow Meow Meow" in second_output, (
        f"Expected meowing output (no reload), got: {second_output}"
    )
```
**EN:** Test that load_inplace=False prevents reloading when an adapter with the same ID already exists. The body exercises logic via `LLM`, `format_chatml_messages`, `SamplingParams` before asserting the expected outcome.
**CN:** 该测试用例验证 Load Inplace False No Reload 在特定场景下的行为。 函数体会先通过 `LLM`, `format_chatml_messages`, `SamplingParams` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **LoRA adapters / LoRA 适配器**
  - **EN:** The scenarios focus on adapter loading, composition, or low-rank update behavior.
  - **CN:** 这些场景关注适配器加载、组合方式或低秩更新行为。
- **Sampling behavior / 采样行为**
  - **EN:** The assertions target decoding controls and the shape of generated outputs.
  - **CN:** 这些断言关注解码控制参数以及生成结果的形态。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.lora.request`
- **Local test utilities / 本地测试辅助**: `tests.utils`
