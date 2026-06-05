# test_model_arch_config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/config/test_model_arch_config.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for ModelArchitectureConfig and its integration with ModelConfig. / 该文件主要围绕 Model Arch Config 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-56)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Tests for ModelArchitectureConfig and its integration with ModelConfig."""

import json
from pathlib import Path

import pytest

from vllm.config import ModelConfig, ParallelConfig, SpeculativeConfig
from vllm.transformers_utils.model_arch_config_convertor import (
    ModelArchConfigConvertorBase,
)

BASE_TRUST_REMOTE_CODE_MODELS = {
    "nvidia/Llama-3_3-Nemotron-Super-49B-v1",
    "nvidia/NVIDIA-Nemotron-3-Nano-30B-A3B-BF16",
    "XiaomiMiMo/MiMo-7B-RL",
    "stepfun-ai/Step-3.5-Flash",
# ... omitted for brevity ...
    # Models using base convertor
    "lmsys/gpt-oss-20b-bf16",
    "deepseek-ai/DeepSeek-V3.2-Exp",
    "meta-llama/Llama-4-Scout-17B-16E-Instruct",
] + list(BASE_TRUST_REMOTE_CODE_MODELS)

# (target_model, draft_model, trust_remote_code)
SPECULATIVE_MODELS = [
    ("JackFram/llama-68m", "abhigoyal/vllm-medusa-llama-68m-random", False),
    ("luccafong/deepseek_mtp_main_random", "luccafong/deepseek_mtp_draft_random", True),
    ("eagle618/deepseek-v3-random", "eagle618/eagle-deepseek-v3-random", True),
    ("meta-llama/Meta-Llama-3-8B-Instruct", "yuhuili/EAGLE-LLaMA3-Instruct-8B", True),
    ("meta-llama/Llama-3.1-8B-Instruct", "yuhuili/EAGLE3-LLaMA3.1-Instruct-8B", True),
]
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `pathlib`, `pytest`, `vllm.config`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _load_groundtruth (lines 59-63)
```python
def _load_groundtruth(filename: str) -> dict:
    """Load groundtruth JSON from the test directory."""
    groundtruth_path = Path(__file__).parent / filename
    with open(groundtruth_path) as f:
        return json.load(f)
```
**EN:** Load groundtruth JSON from the test directory. It coordinates operations such as `open`, `json.load`, `Path`.
**CN:** 该辅助函数为 Load Groundtruth 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `open`, `json.load`, `Path` 等操作。

### Helper: _assert_model_arch_config (lines 66-96)
```python
def _assert_model_arch_config(
    model_config, expected: dict, check_head_size: bool = True
):
    """Assert model_arch_config matches expected values."""
    model_arch_config = model_config.model_arch_config
    assert model_arch_config.architectures == expected["architectures"]
    assert model_arch_config.model_type == expected["model_type"]
    assert model_arch_config.text_model_type == expected["text_model_type"]
    assert model_arch_config.hidden_size == expected["hidden_size"]
    assert (
        model_arch_config.total_num_hidden_layers == expected["total_num_hidden_layers"]
    )
    assert (
        model_arch_config.total_num_attention_heads
        == expected["total_num_attention_heads"]
    )
    assert model_arch_config.vocab_size == expected["vocab_size"]
    assert model_arch_config.total_num_kv_heads == expected["total_num_kv_heads"]
    assert model_arch_config.num_experts == expected["num_experts"]
    assert model_arch_config.is_deepseek_mla == expected["is_deepseek_mla"]

    torch_dtype = ModelArchConfigConvertorBase.get_torch_dtype(
        model_config.hf_config,
        model_config.model,
        revision=model_config.revision,
        config_format="hf",
    )
    assert str(torch_dtype) == expected["dtype"]

    if check_head_size:
        assert model_arch_config.head_size == expected["head_size"]
```
**EN:** Assert model_arch_config matches expected values. It coordinates operations such as `ModelArchConfigConvertorBase.get_torch_dtype`, `str`.
**CN:** 该辅助函数为 Assert Model Arch Config 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `ModelArchConfigConvertorBase.get_torch_dtype`, `str` 等操作。

### Helper: _assert_model_config_methods (lines 99-114)
```python
def _assert_model_config_methods(
    model_config, expected: dict, check_head_size: bool = True
):
    """Assert model_config methods return expected values."""
    assert model_config.architectures == expected["architectures"]
    assert model_config.get_vocab_size() == expected["vocab_size"]
    assert model_config.get_hidden_size() == expected["hidden_size"]
    assert model_config.get_total_num_kv_heads() == expected["total_num_kv_heads"]
    assert model_config.get_num_experts() == expected["num_experts"]
    assert (
        model_config.get_total_num_hidden_layers()
        == expected["total_num_hidden_layers"]
    )

    if check_head_size:
        assert model_config.get_head_size() == expected["head_size"]
```
**EN:** Assert model_config methods return expected values. It coordinates operations such as `model_config.get_vocab_size`, `model_config.get_hidden_size`, `model_config.get_total_num_kv_heads`.
**CN:** 该辅助函数为 Assert Model Config Methods 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `model_config.get_vocab_size`, `model_config.get_hidden_size`, `model_config.get_total_num_kv_heads` 等操作。

### Test: test_base_model_arch_config (lines 117-128)
```python
@pytest.mark.parametrize("model", BASE_MODELS_TO_TEST)
def test_base_model_arch_config(model: str):
    """Test model architecture config for base models."""
    groundtruth = _load_groundtruth("base_model_arch_groundtruth.json")
    expected = groundtruth[model]

    model_config = ModelConfig(
        model, trust_remote_code=model in BASE_TRUST_REMOTE_CODE_MODELS
    )

    _assert_model_arch_config(model_config, expected)
    _assert_model_config_methods(model_config, expected)
```
**EN:** Test model architecture config for base models. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `_load_groundtruth`, `ModelConfig` before asserting the expected outcome.
**CN:** 该测试用例验证 Base Model Arch Config 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `_load_groundtruth`, `ModelConfig` 驱动目标逻辑，再断言预期结果。

### Test: test_draft_model_arch_config (lines 131-157)
```python
@pytest.mark.parametrize(
    "target_model,draft_model,trust_remote_code", SPECULATIVE_MODELS
)
def test_draft_model_arch_config(
    target_model: str, draft_model: str, trust_remote_code: bool
):
    """Test model architecture config for draft/speculative models."""
    groundtruth = _load_groundtruth("draft_model_arch_groundtruth.json")
    expected = groundtruth[draft_model]

    target_model_config = ModelConfig(target_model, trust_remote_code=trust_remote_code)
    speculative_config = SpeculativeConfig(
        model=draft_model,
        num_speculative_tokens=1,
        target_model_config=target_model_config,
        target_parallel_config=ParallelConfig(),
    )
    model_config = speculative_config.draft_model_config

    # For medusa models, head_size may cause division by zero before
    # model_arch_config was introduced, so we conditionally check it
    check_head_size = isinstance(expected["head_size"], int)

    _assert_model_arch_config(model_config, expected, check_head_size=check_head_size)
    _assert_model_config_methods(
        model_config, expected, check_head_size=check_head_size
    )
```
**EN:** Test model architecture config for draft/speculative models. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `_load_groundtruth`, `ModelConfig` before asserting the expected outcome.
**CN:** 该测试用例验证 Draft Model Arch Config 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `_load_groundtruth`, `ModelConfig` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `pathlib`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.config`, `vllm.transformers_utils.model_arch_config_convertor`
