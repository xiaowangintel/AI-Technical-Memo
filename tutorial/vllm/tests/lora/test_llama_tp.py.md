# test_llama_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_llama_tp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Llama Tp behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Do Sample, Generate And Test, Llama LoRA. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 Llama Tp 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-36)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import subprocess
import sys

import pytest

import vllm
import vllm.config
from vllm import LLM
from vllm.lora.request import LoRARequest
from vllm.model_executor.model_loader.tensorizer import TensorizerConfig
from vllm.platforms import current_platform

from ..utils import VLLM_PATH, create_new_process_for_each_test, multi_gpu_test

PROMPT_TEMPLATE = """<|eot_id|><|start_header_id|>user<|end_header_id|>
I want you to act as a SQL terminal in front of an example database, you need only to return the sql command to me.Below is an instruction that describes a task, Write a response that appropriately completes the request.
"
# ... omitted for brevity ...
The People_ID of candidate is the foreign key of People_ID of people.
###Input:
{context}
###Response:<|eot_id|><|start_header_id|>assistant<|end_header_id|>
"""  # noqa: E501

EXPECTED_LORA_OUTPUT = [
    "SELECT count(*) FROM candidate",
    "SELECT count(*) FROM candidate",
    "SELECT poll_source FROM candidate GROUP BY poll_source ORDER BY count(*) DESC LIMIT 1",  # noqa: E501
    "SELECT poll_source FROM candidate GROUP BY poll_source ORDER BY count(*) DESC LIMIT 1",  # noqa: E501
]

MODEL_PATH = "meta-llama/Llama-3.2-3B-Instruct"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `subprocess`, `sys`, `pytest`, `vllm`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: do_sample (lines 39-94)
```python
def do_sample(
    llm: vllm.LLM,
    lora_path: str,
    lora_id: int,
    tensorizer_config_dict: dict | None = None,
) -> list[str]:
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

    sampling_params = vllm.SamplingParams(
        temperature=0, max_tokens=64, stop=["<|im_end|>"]
# ... omitted for brevity ...
    generated_texts: list[str] = []
    for output in outputs:
        prompt = output.prompt
        generated_text = output.outputs[0].text
        # The output should include  correct lora_request info
        if lora_request is not None:
            assert output.lora_request.lora_name == lora_request.lora_name
            assert output.lora_request.lora_int_id == lora_request.lora_int_id
            assert output.lora_request.lora_path == lora_request.lora_path
        else:
            assert output.lora_request is None
        generated_texts.append(generated_text)
        print(f"Prompt: {prompt!r}, Generated text: {generated_text!r}")
    return generated_texts
```
**EN:** Implements a reusable helper for Do Sample, reducing duplication across related tests. It coordinates operations such as `vllm.SamplingParams`, `PROMPT_TEMPLATE.format`, `llm.generate`.
**CN:** 该辅助函数为 Do Sample 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `vllm.SamplingParams`, `PROMPT_TEMPLATE.format`, `llm.generate` 等操作。

### Helper: generate_and_test (lines 97-123)
```python
def generate_and_test(
    llm, llama32_lora_files, tensorizer_config_dict: dict | None = None
):
    print("lora adapter created")
    print("lora 1")
    assert (
        do_sample(
            llm,
            llama32_lora_files,
            tensorizer_config_dict=tensorizer_config_dict,
            lora_id=1,
        )
        == EXPECTED_LORA_OUTPUT
    )

    print("lora 2")
    assert (
        do_sample(
            llm,
            llama32_lora_files,
            tensorizer_config_dict=tensorizer_config_dict,
            lora_id=2,
        )
        == EXPECTED_LORA_OUTPUT
    )

    print("removing lora")
```
**EN:** Implements a reusable helper for Generate And Test, reducing duplication across related tests. It coordinates operations such as `print`, `do_sample`.
**CN:** 该辅助函数为 Generate And Test 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `print`, `do_sample` 等操作。

### Test: test_llama_lora (lines 126-140)
```python
@create_new_process_for_each_test()
@pytest.mark.parametrize("cudagraph_specialize_lora", [True, False])
def test_llama_lora(llama32_lora_files, cudagraph_specialize_lora: bool):
    llm = vllm.LLM(
        MODEL_PATH,
        enable_lora=True,
        # also test odd max_num_seqs
        max_num_seqs=7,
        max_model_len=1024,
        max_loras=4,
        compilation_config=vllm.config.CompilationConfig(
            cudagraph_specialize_lora=cudagraph_specialize_lora,
        ),
    )
    generate_and_test(llm, llama32_lora_files)
```
**EN:** Checks Llama LoRA under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `create_new_process_for_each_test`, `pytest.mark.parametrize`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Llama LoRA 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `create_new_process_for_each_test`, `pytest.mark.parametrize`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

### Test: test_llama_lora_tp4 (lines 143-156)
```python
@pytest.mark.skipif(
    current_platform.is_cuda_alike(), reason="Skipping to avoid redundant model tests"
)
@multi_gpu_test(num_gpus=4)
def test_llama_lora_tp4(llama32_lora_files):
    llm = vllm.LLM(
        MODEL_PATH,
        enable_lora=True,
        max_num_seqs=7,
        max_model_len=1024,
        max_loras=4,
        tensor_parallel_size=4,
    )
    generate_and_test(llm, llama32_lora_files)
```
**EN:** Checks Llama LoRA Tp4 under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `multi_gpu_test`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Llama LoRA Tp4 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `multi_gpu_test`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

### Test: test_llama_lora_tp4_fully_sharded_loras (lines 159-170)
```python
@multi_gpu_test(num_gpus=4)
def test_llama_lora_tp4_fully_sharded_loras(llama32_lora_files):
    llm = vllm.LLM(
        MODEL_PATH,
        enable_lora=True,
        max_num_seqs=8,
        max_loras=4,
        max_model_len=1024,
        tensor_parallel_size=4,
        fully_sharded_loras=True,
    )
    generate_and_test(llm, llama32_lora_files)
```
**EN:** Checks Llama LoRA Tp4 Fully Sharded Loras under a focused test scenario. The body exercises logic via `multi_gpu_test`, `vllm.LLM`, `generate_and_test` before asserting the expected outcome.
**CN:** 该测试用例验证 Llama LoRA Tp4 Fully Sharded Loras 在特定场景下的行为。 函数体会先通过 `multi_gpu_test`, `vllm.LLM`, `generate_and_test` 驱动目标逻辑，再断言预期结果。

### Test: test_tp2_serialize_and_deserialize_lora (lines 173-246)
```python
@multi_gpu_test(num_gpus=2)
def test_tp2_serialize_and_deserialize_lora(
    tmp_path,
    llama32_lora_files,
):
    # Run the tensorizing of the LoRA adapter and the model in a subprocess
    # to guarantee cleanup

    tp_size = 2
    model_name = "model-rank-%03d.tensors"

    model_ref = MODEL_PATH
    lora_path = llama32_lora_files
    suffix = "test"
    try:
        result = subprocess.run(
            [
                sys.executable,
                f"{VLLM_PATH}/examples/features/tensorize_vllm_model.py",
# ... omitted for brevity ...
        # during the first inference step.
        gpu_memory_utilization=0.85,
    )

    tc_as_dict = tensorizer_config.to_serializable()

    print("lora adapter created")
    print("lora 1")
    assert (
        do_sample(
            loaded_llm, llama32_lora_files, tensorizer_config_dict=tc_as_dict, lora_id=1
        )
        == EXPECTED_LORA_OUTPUT
    )
```
**EN:** Checks Tp2 Serialize And Deserialize LoRA under a focused test scenario. The body exercises logic via `multi_gpu_test`, `print`, `TensorizerConfig` before asserting the expected outcome.
**CN:** 该测试用例验证 Tp2 Serialize And Deserialize LoRA 在特定场景下的行为。 函数体会先通过 `multi_gpu_test`, `print`, `TensorizerConfig` 驱动目标逻辑，再断言预期结果。

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
- **LoRA adapters / LoRA 适配器**
  - **EN:** The scenarios focus on adapter loading, composition, or low-rank update behavior.
  - **CN:** 这些场景关注适配器加载、组合方式或低秩更新行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `subprocess`, `sys`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.config`, `vllm.lora.request`, `vllm.model_executor.model_loader.tensorizer`, `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `..utils`
