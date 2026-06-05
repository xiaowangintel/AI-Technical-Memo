# test_whisper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_whisper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Integration tests for Whisper models with LoRA adapters. / 该文件主要围绕 Whisper 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-22)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
Integration tests for Whisper models with LoRA adapters.

These tests verify that Whisper models can correctly load and use LoRA adapters
for speech-to-text transcription tasks.
"""

import pytest

import vllm
from vllm.assets.audio import AudioAsset
from vllm.lora.request import LoRARequest

from ..utils import create_new_process_for_each_test

# Model configuration
WHISPER_MODEL = "openai/whisper-small"

# Test prompts for Whisper transcription
WHISPER_PROMPT = "<|startoftranscript|><|en|><|transcribe|><|notimestamps|>"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `vllm`, `vllm.assets.audio`, `..utils`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: use_spawn_for_whisper (lines 27-30)
```python
@pytest.fixture(autouse=True)
def use_spawn_for_whisper(monkeypatch):
    """Whisper has issues with forked workers, use spawn instead."""
    monkeypatch.setenv("VLLM_WORKER_MULTIPROC_METHOD", "spawn")
```
**EN:** Whisper has issues with forked workers, use spawn instead. The fixture mainly builds or returns values through `pytest.fixture`, `monkeypatch.setenv`.
**CN:** 该代码块定义 pytest 夹具 `use_spawn_for_whisper`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `monkeypatch.setenv` 构造或返回测试所需的值。

### Helper: create_whisper_llm (lines 33-43)
```python
def create_whisper_llm(enable_lora: bool = True, max_loras: int = 2):
    """Create a Whisper LLM instance with optional LoRA support."""
    return vllm.LLM(
        model=WHISPER_MODEL,
        enable_lora=enable_lora,
        max_loras=max_loras if enable_lora else 1,
        max_lora_rank=64,
        max_model_len=448,
        dtype="half",
        enforce_eager=True,  # For stability in tests
    )
```
**EN:** Create a Whisper LLM instance with optional LoRA support. It coordinates operations such as `vllm.LLM`.
**CN:** 该辅助函数为 Create Whisper LLM 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `vllm.LLM` 等操作。

### Helper: run_whisper_inference (lines 46-79)
```python
def run_whisper_inference(
    llm: vllm.LLM,
    lora_path: str | None = None,
    lora_id: int = 1,
) -> list[str]:
    """Run Whisper inference with optional LoRA adapter."""
    # Load test audio
    audio_asset = AudioAsset("mary_had_lamb")
    audio_data = audio_asset.audio_and_sample_rate

    inputs = [
        {
            "prompt": WHISPER_PROMPT,
            "multi_modal_data": {"audio": audio_data},
        }
    ]

    sampling_params = vllm.SamplingParams(
        temperature=0,
        max_tokens=200,
    )

    # Prepare LoRA request if adapter path is provided
    lora_request = None
    if lora_path:
        lora_request = LoRARequest(
            lora_name=f"whisper_lora_{lora_id}",
            lora_int_id=lora_id,
            lora_path=lora_path,
        )

    outputs = llm.generate(inputs, sampling_params, lora_request=lora_request)

    return [output.outputs[0].text for output in outputs]
```
**EN:** Run Whisper inference with optional LoRA adapter. It coordinates operations such as `AudioAsset`, `vllm.SamplingParams`, `llm.generate`.
**CN:** 该辅助函数为 Run Whisper Inference 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `AudioAsset`, `vllm.SamplingParams`, `llm.generate` 等操作。

### Test: test_whisper_lora_inference (lines 82-102)
```python
@create_new_process_for_each_test()
def test_whisper_lora_inference(whisper_lora_files):
    """Test basic Whisper inference with a LoRA adapter.

    This test verifies that:
    1. Whisper model can be loaded with LoRA support enabled
    2. A LoRA adapter can be applied during inference
    3. The model produces valid transcription output
    """
    llm = create_whisper_llm(enable_lora=True)

    # Run inference with LoRA
    outputs = run_whisper_inference(llm, lora_path=whisper_lora_files, lora_id=1)

    # Verify we got a non-empty transcription
    assert len(outputs) == 1
    assert len(outputs[0]) > 0, "Expected non-empty transcription output"

    # The output should contain some recognizable words from the audio
    # (Mary had a little lamb)
    print(f"Transcription output: {outputs[0]}")
```
**EN:** Test basic Whisper inference with a LoRA adapter. The body exercises logic via `create_new_process_for_each_test`, `create_whisper_llm`, `run_whisper_inference` before asserting the expected outcome.
**CN:** 该测试用例验证 Whisper LoRA Inference 在特定场景下的行为。 函数体会先通过 `create_new_process_for_each_test`, `create_whisper_llm`, `run_whisper_inference` 驱动目标逻辑，再断言预期结果。

### Test: test_whisper_multi_lora (lines 105-126)
```python
@create_new_process_for_each_test()
def test_whisper_multi_lora(whisper_lora_files):
    """Test Whisper with multiple LoRA adapter IDs.

    This test verifies that the same LoRA adapter can be loaded with
    different IDs and produce consistent results.
    """
    llm = create_whisper_llm(enable_lora=True, max_loras=4)

    # Test with different LoRA IDs using the same adapter
    outputs_lora1 = run_whisper_inference(llm, lora_path=whisper_lora_files, lora_id=1)
    outputs_lora2 = run_whisper_inference(llm, lora_path=whisper_lora_files, lora_id=2)

    # Both should produce valid outputs
    assert len(outputs_lora1[0]) > 0
    assert len(outputs_lora2[0]) > 0

    # Same adapter with different IDs should produce same output
    assert outputs_lora1 == outputs_lora2, (
        f"Expected same outputs for same adapter with different IDs. "
        f"Got: {outputs_lora1} vs {outputs_lora2}"
    )
```
**EN:** Test Whisper with multiple LoRA adapter IDs. The body exercises logic via `create_new_process_for_each_test`, `create_whisper_llm`, `run_whisper_inference` before asserting the expected outcome.
**CN:** 该测试用例验证 Whisper Multi LoRA 在特定场景下的行为。 函数体会先通过 `create_new_process_for_each_test`, `create_whisper_llm`, `run_whisper_inference` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.assets.audio`, `vllm.lora.request`
- **Local test utilities / 本地测试辅助**: `..utils`
