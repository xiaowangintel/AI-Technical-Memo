# test_qwen36_moe_lora.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_qwen36_moe_lora.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Qwen36 MoE LoRA behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Build Prompts, Generate, Run Mixed 2d 3d LoRA Test. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 Qwen36 MoE LoRA 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-31)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import pytest

import vllm
import vllm.config
from vllm.assets.image import ImageAsset
from vllm.lora.request import LoRARequest

from ..utils import multi_gpu_test

MODEL_PATH = "Qwen/Qwen3.6-35B-A3B"

LORA_2D_ID = 1
LORA_3D_ID = 2

PROMPT_TEMPLATE = """<|im_start|>user
<|vision_start|><|image_pad|><|vision_end|>What is in the image?<|im_end|>
<|im_start|>assistant
<think>

</think>
"""

# Visual captioning prompts: each image will be paired with one LoRA in the
# mixed-batch case so we can check per-prompt routing.
VL_TEST_IMAGES = [
    ImageAsset("stop_sign"),
    ImageAsset("cherry_blossom"),
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `vllm`, `vllm.config`, `..utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _build_prompts (lines 34-41)
```python
def _build_prompts() -> list[dict]:
    return [
        {
            "prompt": PROMPT_TEMPLATE,
            "multi_modal_data": {"image": asset.pil_image},
        }
        for asset in VL_TEST_IMAGES
    ]
```
**EN:** Implements a reusable helper for Build Prompts, reducing duplication across related tests.
**CN:** 该辅助函数为 Build Prompts 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Helper: _generate (lines 44-50)
```python
def _generate(llm: vllm.LLM, lora_request) -> list[str]:
    outputs = llm.generate(
        _build_prompts(),
        vllm.SamplingParams(temperature=0, max_tokens=128),
        lora_request=lora_request,
    )
    return [out.outputs[0].text.strip() for out in outputs]
```
**EN:** Implements a reusable helper for Generate, reducing duplication across related tests. It coordinates operations such as `llm.generate`, `_build_prompts`, `vllm.SamplingParams`.
**CN:** 该辅助函数为 Generate 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `llm.generate`, `_build_prompts`, `vllm.SamplingParams` 等操作。

### Helper: _run_mixed_2d_3d_lora_test (lines 53-124)
```python
def _run_mixed_2d_3d_lora_test(
    lora_2d_files: str,
    lora_3d_files: str,
    tensor_parallel_size: int,
    fully_sharded_loras: bool,
) -> None:
    llm = vllm.LLM(
        model=MODEL_PATH,
        max_model_len=4096,
        enable_lora=True,
        enable_mixed_moe_lora_format=True,
        max_loras=2,
        max_lora_rank=8,
        max_num_seqs=4,
        enforce_eager=True,
        tensor_parallel_size=tensor_parallel_size,
        enable_expert_parallel=not fully_sharded_loras,
        fully_sharded_loras=fully_sharded_loras,
        trust_remote_code=True,
# ... omitted for brevity ...
        f"Mixed-batch 3D output {mixed_outputs[1]!r} does not match "
        f"standalone 3D output {outputs_3d_alone[1]!r}"
    )

    # Reverse assignment: neither adapter should be silently aliased.
    swapped_outputs = _generate(llm, [lora_3d, lora_2d])
    assert swapped_outputs[0] == outputs_3d_alone[0], (
        f"Swapped-batch 3D output {swapped_outputs[0]!r} does not match "
        f"standalone 3D output {outputs_3d_alone[0]!r}"
    )
    assert swapped_outputs[1] == outputs_2d_alone[1], (
        f"Swapped-batch 2D output {swapped_outputs[1]!r} does not match "
        f"standalone 2D output {outputs_2d_alone[1]!r}"
    )
```
**EN:** Implements a reusable helper for Run Mixed 2d 3d LoRA Test, reducing duplication across related tests. It coordinates operations such as `vllm.LLM`, `LoRARequest`, `_generate`.
**CN:** 该辅助函数为 Run Mixed 2d 3d LoRA Test 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `vllm.LLM`, `LoRARequest`, `_generate` 等操作。

### Test: test_qwen36_moe_mixed_2d_3d_lora_tp2 (lines 127-140)
```python
@pytest.mark.skip(reason="This model is too big, so skip this test temporarily.")
@pytest.mark.parametrize("fully_sharded_loras", [False, True])
@multi_gpu_test(num_gpus=2)
def test_qwen36_moe_mixed_2d_3d_lora_tp2(
    qwen36_moe_2d_lora_files,
    qwen36_moe_3d_lora_files,
    fully_sharded_loras,
):
    _run_mixed_2d_3d_lora_test(
        lora_2d_files=qwen36_moe_2d_lora_files,
        lora_3d_files=qwen36_moe_3d_lora_files,
        tensor_parallel_size=2,
        fully_sharded_loras=fully_sharded_loras,
    )
```
**EN:** Checks Qwen36 MoE Mixed 2d 3d LoRA Tp2 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skip`, `pytest.mark.parametrize`, `multi_gpu_test` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen36 MoE Mixed 2d 3d LoRA Tp2 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skip`, `pytest.mark.parametrize`, `multi_gpu_test` 驱动目标逻辑，再断言预期结果。

### Test: test_qwen36_moe_mixed_2d_3d_lora_tp4 (lines 143-156)
```python
@pytest.mark.skip(reason="This model is too big, so skip this test temporarily.")
@pytest.mark.parametrize("fully_sharded_loras", [False, True])
@multi_gpu_test(num_gpus=4)
def test_qwen36_moe_mixed_2d_3d_lora_tp4(
    qwen36_moe_2d_lora_files,
    qwen36_moe_3d_lora_files,
    fully_sharded_loras,
):
    _run_mixed_2d_3d_lora_test(
        lora_2d_files=qwen36_moe_2d_lora_files,
        lora_3d_files=qwen36_moe_3d_lora_files,
        tensor_parallel_size=4,
        fully_sharded_loras=fully_sharded_loras,
    )
```
**EN:** Checks Qwen36 MoE Mixed 2d 3d LoRA Tp4 under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.skip`, `pytest.mark.parametrize`, `multi_gpu_test` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen36 MoE Mixed 2d 3d LoRA Tp4 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.skip`, `pytest.mark.parametrize`, `multi_gpu_test` 驱动目标逻辑，再断言预期结果。

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
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.config`, `vllm.assets.image`, `vllm.lora.request`
- **Local test utilities / 本地测试辅助**: `..utils`
