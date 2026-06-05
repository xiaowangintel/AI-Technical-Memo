# test_lora_tied_lm_head.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_tied_lm_head.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora tied lm head behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora tied lm head 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 15-29: supporting statements / 辅助语句
```python
"""
Test LoRA on models with tied lm_head (tie_word_embeddings=True).

When tie_word_embeddings=True, lm_head shares the same weight tensor as
embed_tokens. PyTorch's named_modules() deduplicates by object identity,
so lm_head won't appear as a separate module. This test validates that
SGLang correctly handles this case by untying lm_head before LoRA wrapping.

The test:
1. Programmatically creates a LoRA adapter with lm_head in target_modules
   using PEFT on a model with tie_word_embeddings=True (Qwen/Qwen2.5-0.5B).
2. Compares logprobs between HuggingFace+PEFT and SGLang to ensure numerical
   consistency. This implicitly verifies no NaN values are produced and that
   LoRA is actually being applied (since HF+PEFT is the trusted reference).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 31-37: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import os
import shutil
import tempfile
import unittest

import torch
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `os`, `shutil`, `tempfile`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `os`, `shutil`, `tempfile`。

### Lines 39-45: supporting statements / 辅助语句
```python
try:
    from peft import LoraConfig, get_peft_model
except ImportError:
    import subprocess

    subprocess.check_call(["pip", "install", "peft", "--no-deps"])
    from peft import LoraConfig, get_peft_model
```
**EN:** This block performs supporting work through calls such as `check_call`, preparing state for nearby definitions.
**CN:** 该代码块通过 `check_call` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 47-51: module imports and dependencies / 模块导入与依赖
```python
from transformers import AutoModelForCausalLM

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.runners import HFRunner, SRTRunner
from sglang.test.test_utils import DEFAULT_PORT_FOR_SRT_TEST_RUNNER, CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `transformers`, `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `transformers`, `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`。

### Lines 53-64: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, suite="nightly-1-gpu", nightly=True)

# Use a small model with tie_word_embeddings=True
BASE_MODEL = "Qwen/Qwen2.5-0.5B"

TEST_PROMPTS = [
    "AI is a field of computer science focused on",
    "The capital of France is",
]

MAX_NEW_TOKENS = 16
LOGPROB_THRESHOLD = 2e-1
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 67-126: helper routine create lora adapter with lm head / 辅助流程 create lora adapter with lm head
```python
def create_lora_adapter_with_lm_head(base_model_name: str, output_dir: str):
    """
    Programmatically create a LoRA adapter that targets lm_head,
    using a model with tie_word_embeddings=True.

    The adapter uses randomly initialized LoRA weights (no training).
    This is sufficient to test that:
    - SGLang can load the adapter without errors
    - lm_head LoRA is applied (output differs from base model)
    - Logprobs match between HF and SGLang
    """
    model = AutoModelForCausalLM.from_pretrained(
        base_model_name,
        torch_dtype=torch.float16,
        device_map="cpu",
    )

    # Verify the model actually has tied embeddings
    assert (
        model.config.tie_word_embeddings
    ), f"Expected tie_word_embeddings=True for {base_model_name}"

    # Only target lm_head to isolate the test to the tied-embedding scenario.
    lora_config = LoraConfig(
        r=8,
        lora_alpha=16,
        target_modules=["lm_head"],
        lora_dropout=0,
        bias="none",
        task_type="CAUSAL_LM",
    )

    peft_model = get_peft_model(model, lora_config)

    # PEFT initializes lora_B to zeros by default, which makes the adapter
    # produce identical output to the base model. Initialize lora_B with
    # non-zero random weights so the adapter has a visible effect.
    with torch.no_grad():
        for name, param in peft_model.named_parameters():
            if "lora_B" in name:
                torch.nn.init.normal_(param, mean=0.0, std=0.02)

    peft_model.save_pretrained(output_dir)

    # Verify the saved adapter contains lm_head keys
    from safetensors import safe_open

    safetensors_path = os.path.join(output_dir, "adapter_model.safetensors")
    f = safe_open(safetensors_path, framework="pt")
    lm_head_keys = [k for k in f.keys() if "lm_head" in k]
    assert (
        len(lm_head_keys) > 0
    ), f"Expected lm_head LoRA weights in adapter, got keys: {sorted(f.keys())}"

    print(f"Created LoRA adapter at {output_dir}")
    print(f"  lm_head keys: {lm_head_keys}")

    # Clean up the model to free memory
    del peft_model, model
    torch.cuda.empty_cache()
```
**EN:** Programmatically create a LoRA adapter that targets lm_head, using a model with tie_word_embeddings=True. This helper encapsulates `create_lora_adapter_with_lm_head` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Programmatically create a LoRA adapter that targets lm_head, using a model with tie_word_embeddings=True. 该辅助函数封装了 `create_lora_adapter_with_lm_head`，以便周围测试复用准备、执行或校验逻辑。

### Lines 129-129: class TestLoRATiedLMHead declaration / 类 TestLoRATiedLMHead 声明
```python
class TestLoRATiedLMHead(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 130-134: class-level constants and configuration for `TestLoRATiedLMHead` / 类级常量与配置
```python
    """
    Test that LoRA works correctly on models with tied lm_head.
    """

    _adapter_dir = None
```
**EN:** This block defines shared names such as `_adapter_dir`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_adapter_dir` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 136-141: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        """Create a temporary LoRA adapter with lm_head targeting."""
        super().setUpClass()
        cls._adapter_dir = tempfile.mkdtemp(prefix="sglang_test_lora_tied_lm_head_")
        create_lora_adapter_with_lm_head(BASE_MODEL, cls._adapter_dir)
```
**EN:** Create a temporary LoRA adapter with lm_head targeting. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Create a temporary LoRA adapter with lm_head targeting. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 143-148: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        """Clean up the temporary adapter directory."""
        if cls._adapter_dir and os.path.exists(cls._adapter_dir):
            shutil.rmtree(cls._adapter_dir)
        super().tearDownClass()
```
**EN:** Clean up the temporary adapter directory. This routine releases resources and restores state after the related tests finish.
**CN:** Clean up the temporary adapter directory. 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 150-215: test case tied lm head lora hf sgl logprob match / 测试用例 tied lm head lora hf sgl logprob match
```python
    def test_tied_lm_head_lora_hf_sgl_logprob_match(self):
        """
        Compare logprobs between HuggingFace+PEFT and SGLang+LoRA
        for a tied lm_head adapter, ensuring numerical consistency.
        """
        prompts = TEST_PROMPTS[:2]

        # Run SGLang with LoRA
        with SRTRunner(
            BASE_MODEL,
            torch_dtype=torch.float16,
            model_type="generation",
            lora_paths=[self._adapter_dir],
            max_loras_per_batch=1,
            lora_backend="triton",
            lora_target_modules=["lm_head"],
            disable_cuda_graph=True,
            disable_radix_cache=True,
            mem_fraction_static=0.80,
            port=DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
        ) as srt_runner:
            srt_outputs = srt_runner.forward(
                prompts,
                max_new_tokens=MAX_NEW_TOKENS,
                lora_paths=[self._adapter_dir] * len(prompts),
            )

        torch.cuda.empty_cache()

        # Run HuggingFace with LoRA (via PEFT)
        with HFRunner(
            BASE_MODEL,
            torch_dtype=torch.float16,
            model_type="generation",
        ) as hf_runner:
            hf_outputs = hf_runner.forward(
                prompts,
                max_new_tokens=MAX_NEW_TOKENS,
                lora_paths=[self._adapter_dir] * len(prompts),
            )

        # Compare prefill logprobs
        for i in range(len(prompts)):
            srt_logprobs = torch.tensor(srt_outputs.top_input_logprobs[i])
            hf_logprobs = torch.tensor(hf_outputs.top_input_logprobs[i])
            max_diff = torch.max(torch.abs(srt_logprobs - hf_logprobs)).item()
            print(f"Prompt {i} prefill logprob max_diff (SGLang vs HF): {max_diff:.6e}")
            self.assertLess(
                max_diff,
                LOGPROB_THRESHOLD,
                f"Prompt {i}: prefill logprob diff {max_diff:.6e} "
                f"exceeds threshold {LOGPROB_THRESHOLD:.0e}",
            )

        # Compare decode logprobs
        for i in range(len(prompts)):
            srt_logprobs = torch.tensor(srt_outputs.top_output_logprobs[i])
            hf_logprobs = torch.tensor(hf_outputs.top_output_logprobs[i])
            max_diff = torch.max(torch.abs(srt_logprobs - hf_logprobs)).item()
            print(f"Prompt {i} decode logprob max_diff (SGLang vs HF): {max_diff:.6e}")
            self.assertLess(
                max_diff,
                LOGPROB_THRESHOLD,
                f"Prompt {i}: decode logprob diff {max_diff:.6e} "
                f"exceeds threshold {LOGPROB_THRESHOLD:.0e}",
            )
```
**EN:** Compare logprobs between HuggingFace+PEFT and SGLang+LoRA for a tied lm_head adapter, ensuring numerical consistency. This test exercises `test_tied_lm_head_lora_hf_sgl_logprob_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Compare logprobs between HuggingFace+PEFT and SGLang+LoRA for a tied lm_head adapter, ensuring numerical consistency. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tied_lm_head_lora_hf_sgl_logprob_match`。

### Lines 218-224: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    try:
        mp.set_start_method("spawn")
    except RuntimeError:
        pass

    unittest.main(warnings="ignore")
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `create_lora_adapter_with_lm_head`: Programmatically create a LoRA adapter that targets lm_head, using a model with tie_word_embeddings=True. / 该辅助函数封装了 `create_lora_adapter_with_lm_head`，以便周围测试复用准备、执行或校验逻辑。
- `TestLoRATiedLMHead`: Test that LoRA works correctly on models with tied lm_head. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRATiedLMHead.setUpClass`: Create a temporary LoRA adapter with lm_head targeting. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLoRATiedLMHead.tearDownClass`: Clean up the temporary adapter directory. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestLoRATiedLMHead.test_tied_lm_head_lora_hf_sgl_logprob_match`: Compare logprobs between HuggingFace+PEFT and SGLang+LoRA for a tied lm_head adapter, ensuring numerical consistency. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tied_lm_head_lora_hf_sgl_logprob_match`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `os`, `shutil`, `tempfile`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `transformers`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 224
