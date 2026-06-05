# test_qwen35_densemodel_lora.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_qwen35_densemodel_lora.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Qwen35 Densemodel LoRA behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Assert Exact Outputs, Assert Prefix Outputs, Run Text LoRA Sample. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 Qwen35 Densemodel LoRA 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-40)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import pytest
from transformers import AutoTokenizer

import vllm
import vllm.config
from vllm.assets.image import ImageAsset
from vllm.lora.request import LoRARequest
from vllm.platforms import current_platform

from ..utils import create_new_process_for_each_test, multi_gpu_test

MODEL_PATH = "Qwen/Qwen3.5-4B"
TEXT_LORA_ID = 1
VL_LORA_ID = 2

# text-only task
# ... omitted for brevity ...


# visual caption
VL_QUESTION = "What is in the image?"
VL_TEST_IMAGES = [
    ImageAsset("stop_sign"),
    ImageAsset("cherry_blossom"),
]
VL_EXPECTED_LORA_OUTPUT = [
    'A red STOP sign stands prominently in the foreground, with a traditional Chinese gate adorned with red lanterns and the Chinese characters "中華門" in the background, signaling the entrance to a Chinatown. A black car passes by on the street, and stone lion statues guard the entrance to the culturally rich area.',  # noqa: E501
    "A vibrant blue sky serves as a backdrop for the iconic Tokyo Skytree, partially obscured by the delicate pink blossoms of cherry trees in full bloom.",  # noqa: E501
]

TOKENIZER = AutoTokenizer.from_pretrained(MODEL_PATH, trust_remote_code=True)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `transformers`, `vllm`, `vllm.config`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _assert_exact_outputs (lines 43-46)
```python
def _assert_exact_outputs(
    generated_texts: list[str], expected_outputs: list[str]
) -> None:
    assert generated_texts == expected_outputs
```
**EN:** Implements a reusable helper for Assert Exact Outputs, reducing duplication across related tests.
**CN:** 该辅助函数为 Assert Exact Outputs 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Helper: _assert_prefix_outputs (lines 49-58)
```python
def _assert_prefix_outputs(
    generated_texts: list[str],
    expected_outputs: list[str],
) -> None:
    assert len(generated_texts) == len(expected_outputs)
    for generated_text, expected_text in zip(generated_texts, expected_outputs):
        assert expected_text.startswith(generated_text), (
            f"Generated {generated_text!r} is not a prefix of expected "
            f"{expected_text!r}"
        )
```
**EN:** Implements a reusable helper for Assert Prefix Outputs, reducing duplication across related tests. It coordinates operations such as `zip`, `len`, `expected_text.startswith`.
**CN:** 该辅助函数为 Assert Prefix Outputs 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `zip`, `len`, `expected_text.startswith` 等操作。

### Test: test_qwen35_text_lora (lines 316-341)
```python
@pytest.mark.skipif(
    current_platform.is_cuda_alike(), reason="Skipping to avoid redundant model tests"
)
@create_new_process_for_each_test()
def test_qwen35_text_lora(
    qwen35_text_lora_files, qwen35_vl_lora_files, maybe_enable_lora_dual_stream
):
    llm = vllm.LLM(
        model=MODEL_PATH,
        max_model_len=4096,
        enable_lora=True,
        max_loras=2,
        max_num_seqs=4,
        max_lora_rank=8,
        enforce_eager=True,
        trust_remote_code=True,
        enable_tower_connector_lora=True,
        mm_processor_cache_gb=0,
        limit_mm_per_prompt={"image": 1},
    )

    _assert_qwen35_text_vl_and_mixed_lora(
        llm,
        qwen35_text_lora_files,
        qwen35_vl_lora_files,
    )
```
**EN:** Checks Qwen35 Text LoRA under a focused test scenario. The body exercises logic via `pytest.mark.skipif`, `create_new_process_for_each_test`, `vllm.LLM` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen35 Text LoRA 在特定场景下的行为。 函数体会先通过 `pytest.mark.skipif`, `create_new_process_for_each_test`, `vllm.LLM` 驱动目标逻辑，再断言预期结果。

### Test: test_qwen35_text_lora_tp4 (lines 344-370)
```python
@multi_gpu_test(num_gpus=4)
def test_qwen35_text_lora_tp4(
    qwen35_text_lora_files, qwen35_vl_lora_files, maybe_enable_lora_dual_stream
):
    llm = vllm.LLM(
        model=MODEL_PATH,
        max_model_len=4096,
        enable_lora=True,
        max_loras=2,
        max_lora_rank=8,
        max_num_seqs=4,
        enforce_eager=True,
        tensor_parallel_size=4,
        trust_remote_code=True,
        enable_tower_connector_lora=True,
        mm_processor_cache_gb=0,
        limit_mm_per_prompt={"image": 1},
        compilation_config=vllm.config.CompilationConfig(
            cudagraph_specialize_lora=False,
        ),
    )

    _assert_qwen35_text_vl_and_mixed_lora(
        llm,
        qwen35_text_lora_files,
        qwen35_vl_lora_files,
    )
```
**EN:** Checks Qwen35 Text LoRA Tp4 under a focused test scenario. The body exercises logic via `multi_gpu_test`, `vllm.LLM`, `_assert_qwen35_text_vl_and_mixed_lora` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen35 Text LoRA Tp4 在特定场景下的行为。 函数体会先通过 `multi_gpu_test`, `vllm.LLM`, `_assert_qwen35_text_vl_and_mixed_lora` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
_run_text_lora_sample
_run_vl_lora_sample
_build_text_prompts
_build_vl_prompts
_run_mixed_lora_sample
_run_mixed_lora_and_base_sample
_assert_qwen35_text_vl_and_mixed_lora
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

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
- **Third-party / 第三方依赖**: `pytest`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.config`, `vllm.assets.image`, `vllm.lora.request`, `vllm.platforms`
- **Local test utilities / 本地测试辅助**: `..utils`
