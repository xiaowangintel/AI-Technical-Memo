# test_minicpmv_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_minicpmv_tp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Minicpmv Tp behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Do Sample, Minicpmv LoRA, Minicpmv Tp4 Wo Fully Sharded Loras. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 Minicpmv Tp 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-39)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from importlib.metadata import version

import pytest
from packaging.version import Version

import vllm
from vllm.assets.image import ImageAsset
from vllm.lora.request import LoRARequest
from vllm.platforms import current_platform

from ..utils import multi_gpu_test

pytestmark = pytest.mark.skipif(
    Version("5.0") <= Version(version("transformers")),
    reason=(
        "MiniCPMV custom processor uses tokenizer.im_start_id which is not "
# ... omitted for brevity ...
PROMPT_TEMPLATE = (
    "<|begin_of_text|><|start_header_id|>user<|end_header_id|>\n\n"
    "(<image>./</image>)\nWhat is in the image?<|eot_id|>"
    "<|start_header_id|>assistant<|end_header_id|>\n\n"
)

IMAGE_ASSETS = [
    ImageAsset("stop_sign"),
]

# After fine-tuning with LoRA, all generated content should start begin `A`.
EXPECTED_OUTPUT = [
    "A red and white stop sign with a Chinese archway in the background featuring red lanterns and gold accents.",  # noqa: E501
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `importlib.metadata`, `pytest`, `packaging.version`, `vllm`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: do_sample (lines 42-68)
```python
def do_sample(llm: vllm.LLM, lora_path: str, lora_id: int) -> list[str]:
    sampling_params = vllm.SamplingParams(
        temperature=0,
        max_tokens=5,
        stop_token_ids=[128001, 128009],  # eos_id, eot_id
    )

    inputs = [
        {
            "prompt": PROMPT_TEMPLATE,
            "multi_modal_data": {"image": asset.pil_image},
        }
        for asset in IMAGE_ASSETS
    ]

    outputs = llm.generate(
        inputs,
        sampling_params,
        lora_request=LoRARequest(str(lora_id), lora_id, lora_path) if lora_id else None,
    )
    # Print the outputs.
    generated_texts: list[str] = []
    for output in outputs:
        generated_text = output.outputs[0].text.strip()
        generated_texts.append(generated_text)
        print(f"Generated text: {generated_text!r}")
    return generated_texts
```
**EN:** Implements a reusable helper for Do Sample, reducing duplication across related tests. It coordinates operations such as `vllm.SamplingParams`, `llm.generate`, `output.outputs[0].text.strip`.
**CN:** 该辅助函数为 Do Sample 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `vllm.SamplingParams`, `llm.generate`, `output.outputs[0].text.strip` 等操作。

### Test: test_minicpmv_lora (lines 71-91)
```python
@pytest.mark.skipif(
    current_platform.is_cuda_alike(), reason="Skipping to avoid redundant model tests"
)
def test_minicpmv_lora(minicpmv_lora_files):
    llm = vllm.LLM(
        MODEL_PATH,
        max_num_seqs=2,
        enable_lora=True,
        max_loras=2,
        max_lora_rank=8,
        enforce_eager=True,
        max_model_len=2048,
        limit_mm_per_prompt={"image": 2, "video": 0},
        trust_remote_code=True,
    )
    output1 = do_sample(llm, minicpmv_lora_files, lora_id=1)
    for i in range(len(EXPECTED_OUTPUT)):
        assert EXPECTED_OUTPUT[i].startswith(output1[i])
    output2 = do_sample(llm, minicpmv_lora_files, lora_id=2)
    for i in range(len(EXPECTED_OUTPUT)):
        assert EXPECTED_OUTPUT[i].startswith(output2[i])
```
**EN:** Checks Minicpmv LoRA under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `vllm.LLM`, `do_sample` before asserting the expected outcome.
**CN:** 该测试用例验证 Minicpmv LoRA 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `vllm.LLM`, `do_sample` 驱动目标逻辑，再断言预期结果。

### Test: test_minicpmv_tp4_wo_fully_sharded_loras (lines 94-111)
```python
@pytest.mark.skipif(
    current_platform.is_cuda_alike(), reason="Skipping to avoid redundant model tests"
)
@multi_gpu_test(num_gpus=4)
def test_minicpmv_tp4_wo_fully_sharded_loras(minicpmv_lora_files):
    llm = vllm.LLM(
        MODEL_PATH,
        enable_lora=True,
        max_num_seqs=2,
        max_loras=4,
        max_lora_rank=64,
        tensor_parallel_size=4,
        limit_mm_per_prompt={"image": 2, "video": 0},
        trust_remote_code=True,
    )
    output_tp = do_sample(llm, minicpmv_lora_files, lora_id=1)
    for i in range(len(EXPECTED_OUTPUT)):
        assert EXPECTED_OUTPUT[i].startswith(output_tp[i])
```
**EN:** Checks Minicpmv Tp4 Wo Fully Sharded Loras under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `multi_gpu_test`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Minicpmv Tp4 Wo Fully Sharded Loras 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `multi_gpu_test`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

### Test: test_minicpmv_tp4_fully_sharded_loras (lines 114-135)
```python
@pytest.mark.skipif(
    current_platform.is_cuda_alike(), reason="Skipping to avoid redundant model tests"
)
@multi_gpu_test(num_gpus=4)
def test_minicpmv_tp4_fully_sharded_loras(minicpmv_lora_files):
    llm = vllm.LLM(
        MODEL_PATH,
        enable_lora=True,
        max_num_seqs=2,
        max_loras=2,
        max_lora_rank=8,
        tensor_parallel_size=4,
        trust_remote_code=True,
        limit_mm_per_prompt={"image": 1, "video": 0},
        fully_sharded_loras=True,
    )
    output_tp = do_sample(llm, minicpmv_lora_files, lora_id=1)
    for i in range(len(EXPECTED_OUTPUT)):
        assert EXPECTED_OUTPUT[i].startswith(output_tp[i])
    output_tp = do_sample(llm, minicpmv_lora_files, lora_id=2)
    for i in range(len(EXPECTED_OUTPUT)):
        assert EXPECTED_OUTPUT[i].startswith(output_tp[i])
```
**EN:** Checks Minicpmv Tp4 Fully Sharded Loras under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `multi_gpu_test`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Minicpmv Tp4 Fully Sharded Loras 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `multi_gpu_test`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib.metadata`
- **Third-party / 第三方依赖**: `pytest`, `packaging.version`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.assets.image`, `vllm.lora.request`, `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `..utils`
