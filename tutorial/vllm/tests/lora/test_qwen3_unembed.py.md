# test_qwen3_unembed.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/test_qwen3_unembed.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for Qwen3 unembed LoRA support. / 该文件主要围绕 Qwen3 Unembed 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-23)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
Tests for Qwen3 unembed LoRA support.

This test creates synthetic LoRA weights that include lm_head (output embedding)
to verify that Qwen3 properly supports LoRA on the unembed/lm_head layer.
"""

import json
import os
import tempfile

import numpy as np
import torch
from safetensors.torch import save_file

from vllm import LLM, SamplingParams
from vllm.lora.request import LoRARequest

MODEL_PATH = "Qwen/Qwen3-0.6B"
HIDDEN_SIZE = 1024
VOCAB_SIZE = 151936
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `os`, `numpy`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: create_qwen3_lora_with_lm_head (lines 26-61)
```python
def create_qwen3_lora_with_lm_head(save_dir: str, rank: int = 8) -> None:
    """Create synthetic Qwen3 LoRA weights with lm_head."""
    lora_weights = {}
    for module in ["q_proj", "v_proj"]:
        lora_A = torch.from_numpy(
            np.random.randn(rank, HIDDEN_SIZE).astype(np.float16) * 0.01
        )
        lora_B = torch.zeros(HIDDEN_SIZE, rank, dtype=torch.float16)
        key_prefix = f"base_model.model.model.layers.0.self_attn.{module}"
        lora_weights[f"{key_prefix}.lora_A.weight"] = lora_A
        lora_weights[f"{key_prefix}.lora_B.weight"] = lora_B

    # lm_head LoRA weights
    lora_weights["base_model.model.lm_head.lora_A.weight"] = torch.from_numpy(
        np.random.randn(rank, HIDDEN_SIZE).astype(np.float16) * 0.01
    )
    lora_weights["base_model.model.lm_head.lora_B.weight"] = torch.zeros(
        VOCAB_SIZE, rank, dtype=torch.float16
    )
# ... omitted for brevity ...
        "base_model_name_or_path": MODEL_PATH,
        "task_type": "CAUSAL_LM",
        "inference_mode": True,
        "r": rank,
        "lora_alpha": rank * 2,
        "lora_dropout": 0.0,
        "bias": "none",
        "target_modules": ["q_proj", "v_proj", "lm_head"],
    }

    os.makedirs(save_dir, exist_ok=True)
    with open(os.path.join(save_dir, "adapter_config.json"), "w") as f:
        json.dump(adapter_config, f)
    save_file(lora_weights, os.path.join(save_dir, "adapter_model.safetensors"))
```
**EN:** Create synthetic Qwen3 LoRA weights with lm_head. It coordinates operations such as `torch.from_numpy`, `torch.zeros`, `os.makedirs`.
**CN:** 该辅助函数为 Create Qwen3 LoRA With Lm Head 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.from_numpy`, `torch.zeros`, `os.makedirs` 等操作。

### Test: test_qwen3_unembed_lora (lines 64-100)
```python
def test_qwen3_unembed_lora():
    """Verify Qwen3 can load and generate with LoRA adapters with lm_head."""
    with tempfile.TemporaryDirectory() as tmpdir:
        # Initialize engine first (before creating torch tensors)
        llm = LLM(
            model=MODEL_PATH,
            enable_lora=True,
            max_loras=4,
            max_lora_rank=8,
            max_model_len=128,
            gpu_memory_utilization=0.8,
            enforce_eager=True,
        )

        # Create LoRA weights after engine init
        create_qwen3_lora_with_lm_head(tmpdir, rank=8)

        lora_request = LoRARequest("lm_head_lora", 1, tmpdir)
        llm.llm_engine.add_lora(lora_request)
# ... omitted for brevity ...
        sampling_params = SamplingParams(temperature=0, max_tokens=32)
        prompts = ["Hello, my name is"]

        # Generate with base model (no LoRA)
        base_outputs = llm.generate(prompts, sampling_params, use_tqdm=False)
        assert len(base_outputs) == 1
        assert len(base_outputs[0].outputs[0].text) > 0

        # Generate with lm_head LoRA
        lora_outputs = llm.generate(
            prompts, sampling_params, lora_request=lora_request, use_tqdm=False
        )
        assert len(lora_outputs) == 1
        assert len(lora_outputs[0].outputs[0].text) > 0
```
**EN:** Verify Qwen3 can load and generate with LoRA adapters with lm_head. The body exercises logic via `tempfile.TemporaryDirectory`, `LLM`, `create_qwen3_lora_with_lm_head` before asserting the expected outcome.
**CN:** 该测试用例验证 Qwen3 Unembed LoRA 在特定场景下的行为。 函数体会先通过 `tempfile.TemporaryDirectory`, `LLM`, `create_qwen3_lora_with_lm_head` 驱动目标逻辑，再断言预期结果。

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
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `tempfile`
- **Third-party / 第三方依赖**: `numpy`, `torch`, `safetensors.torch`
- **vLLM internal / vLLM 内部依赖**: `vllm`, `vllm.lora.request`
