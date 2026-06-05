# test_lora_qwen3_8b_logprob_diff.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_qwen3_8b_logprob_diff.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora qwen3 8b logprob diff behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora qwen3 8b logprob diff 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

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

### Lines 15-24: supporting statements / 辅助语句
```python
"""
Regression test for Qwen3-8B LoRA logprob accuracy.

Compares SGLang LoRA logprobs against reference training logprobs from a
pre-computed dataset. The LoRA adapter and reference data are downloaded from:
https://huggingface.co/datasets/yushengsu/lora-diff-Qwen3-8B

Usage:
    python -m unittest test_lora_qwen3_8b_logprob_diff
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 26-38: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import os
import unittest
from unittest.mock import patch

import torch
import torch.nn as nn
from huggingface_hub import snapshot_download

import sglang as sgl
from sglang.srt.lora.utils import auto_detect_lora_target_modules
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `os`, `unittest`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `os`, `unittest`, `unittest.mock`。

### Lines 40-50: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=40, stage="extra-a", runner_config="1-gpu-large")

BASE_MODEL = "Qwen/Qwen3-8B"
LORA_HF_REPO = "yushengsu/lora-diff-Qwen3-8B"
LORA_BACKEND = "triton"
MAX_LORA_RANK = 32
TP_SIZE = 1
PREFILL_ATTENTION_BACKEND = "fa4"
DECODE_ATTENTION_BACKEND = "fa4"

KL_THRESHOLD = 5e-3
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 53-56: function kl v2 / 函数 kl v2
```python
def kl_v2(a, b):
    a = torch.tensor(a) if not torch.is_tensor(a) else a
    b = torch.tensor(b) if not torch.is_tensor(b) else b
    return (((a - b) ** 2) * 0.5).mean().item()
```
**EN:** This block implements `kl_v2` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `kl_v2`，承担模块行为中的一个聚焦逻辑片段。

### Lines 59-67: helper routine get prompt logprobs / 辅助流程 get prompt logprobs
```python
def get_prompt_logprobs(engine, input_ids, lora_path):
    out = engine.generate(
        input_ids=input_ids,
        sampling_params={"max_new_tokens": 0, "temperature": 0.0},
        return_logprob=True,
        logprob_start_len=0,
        lora_path=lora_path,
    )
    return [logprob for logprob, _, _ in out["meta_info"]["input_token_logprobs"]][1:]
```
**EN:** This helper encapsulates `get_prompt_logprobs` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_prompt_logprobs`，以便周围测试复用准备、执行或校验逻辑。

### Lines 70-70: class _MockLinearBase declaration / 类 _MockLinearBase 声明
```python
class _MockLinearBase(nn.Module):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `nn.Module`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `nn.Module`。

### Lines 71-71: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 74-74: class _MockFusedMoE declaration / 类 _MockFusedMoE 声明
```python
class _MockFusedMoE(nn.Module):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `nn.Module`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `nn.Module`。

### Lines 75-75: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 78-78: class _MockParallelLMHead declaration / 类 _MockParallelLMHead 声明
```python
class _MockParallelLMHead(nn.Module):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `nn.Module`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `nn.Module`。

### Lines 79-79: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 82-102: function build qwen3 mock / 函数 build qwen3 mock
```python
def _build_qwen3_mock():
    """Build a lightweight nn.Module tree that mirrors Qwen3-8B's named modules."""
    model = nn.Module()
    inner = nn.Module()
    layer = nn.Module()

    attn = nn.Module()
    attn.qkv_proj = _MockLinearBase()
    attn.o_proj = _MockLinearBase()
    layer.self_attn = attn

    mlp = nn.Module()
    mlp.gate_up_proj = _MockLinearBase()
    mlp.down_proj = _MockLinearBase()
    layer.mlp = mlp

    inner.layers = nn.ModuleList([layer])
    inner.embed_tokens = nn.Embedding(10, 8)  # not a LinearBase — should be excluded
    model.model = inner
    model.lm_head = _MockParallelLMHead()
    return model
```
**EN:** Build a lightweight nn.Module tree that mirrors Qwen3-8B's named modules. This block implements `_build_qwen3_mock` and captures one focused piece of the module's behavior.
**CN:** Build a lightweight nn.Module tree that mirrors Qwen3-8B's named modules. 该代码块实现 `_build_qwen3_mock`，承担模块行为中的一个聚焦逻辑片段。

### Lines 105-106: class TestLoRAQwen3_8BLogprobDiff declaration / 类 TestLoRAQwen3_8BLogprobDiff 声明
```python
class TestLoRAQwen3_8BLogprobDiff(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 107-126: test case auto detect lora target modules / 测试用例 auto detect lora target modules
```python
    def test_auto_detect_lora_target_modules(self):
        """Verify auto_detect_lora_target_modules returns the expected module
        set for a Qwen3-8B-like (dense) architecture.  Catches silent renames
        of internal param names that would break LoRA auto-detection."""
        model = _build_qwen3_mock()

        with (
            patch("sglang.srt.layers.linear.LinearBase", _MockLinearBase),
            patch(
                "sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE", _MockFusedMoE
            ),
            patch(
                "sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead",
                _MockParallelLMHead,
            ),
        ):
            detected = auto_detect_lora_target_modules(model)

        expected = {"qkv_proj", "o_proj", "gate_up_proj", "down_proj", "lm_head"}
        self.assertEqual(detected, expected)
```
**EN:** Verify auto_detect_lora_target_modules returns the expected module set for a Qwen3-8B-like (dense) architecture. This test exercises `test_auto_detect_lora_target_modules` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify auto_detect_lora_target_modules returns the expected module set for a Qwen3-8B-like (dense) architecture. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_lora_target_modules`。

### Lines 128-187: test case lora qwen3 8b logprob accuracy / 测试用例 lora qwen3 8b logprob accuracy
```python
    def test_lora_qwen3_8b_logprob_accuracy(self):
        adapter_path = snapshot_download(
            LORA_HF_REPO,
            repo_type="dataset",
        )

        engine = sgl.Engine(
            model_path=BASE_MODEL,
            tp_size=TP_SIZE,
            enable_lora=True,
            max_lora_rank=MAX_LORA_RANK,
            lora_paths={"my_lora": adapter_path},
            lora_backend=LORA_BACKEND,
            attention_backend="flashinfer",
            prefill_attention_backend=PREFILL_ATTENTION_BACKEND,
            decode_attention_backend=DECODE_ATTENTION_BACKEND,
        )

        try:
            cdata = torch.load(
                os.path.join(adapter_path, "compare_sample_train_data.pt"),
                weights_only=False,
            )

            base_logprobs = get_prompt_logprobs(engine, cdata["tokens"], lora_path=None)
            logprobs = get_prompt_logprobs(engine, cdata["tokens"], lora_path="my_lora")

            base_t = torch.tensor(base_logprobs)
            lora_t = torch.tensor(logprobs)
            diff = (base_t - lora_t).abs()
            print(
                f"[VERIFY] base vs lora: mean_diff={diff.mean().item():.6f}, "
                f"max_diff={diff.max().item():.6f}, "
                f"identical={torch.equal(base_t, lora_t)}"
            )

            self.assertFalse(
                torch.equal(base_t, lora_t),
                "LoRA logprobs should differ from base model logprobs",
            )

            kl_sglang_trainer = kl_v2(cdata["training_logprobs"], logprobs)
            kl_orig_trainer = kl_v2(
                cdata["training_logprobs"], cdata["sampling_logprobs"]
            )
            kl_sglang_orig = kl_v2(logprobs, cdata["sampling_logprobs"])

            print(f"KL(orig_sampler, trainer) = {kl_orig_trainer:.6e}")
            print(f"KL(sglang, trainer)       = {kl_sglang_trainer:.6e}")
            print(f"KL(sglang, orig_sampler)  = {kl_sglang_orig:.6e}")

            self.assertLessEqual(
                kl_sglang_trainer,
                KL_THRESHOLD,
                f"KL(sglang, trainer) = {kl_sglang_trainer:.6e} exceeds "
                f"threshold {KL_THRESHOLD}",
            )

        finally:
            engine.shutdown()
```
**EN:** This test exercises `test_lora_qwen3_8b_logprob_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_qwen3_8b_logprob_accuracy`。

### Lines 190-201: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    try:
        mp.set_start_method("spawn")
    except RuntimeError:
        pass

    try:
        unittest.main(warnings="ignore", verbosity=2)
    finally:
        if torch.cuda.is_available():
            torch.cuda.empty_cache()
            torch.cuda.synchronize()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `kl_v2`: This block implements `kl_v2` and captures one focused piece of the module's behavior. / 该代码块实现 `kl_v2`，承担模块行为中的一个聚焦逻辑片段。
- `get_prompt_logprobs`: This helper encapsulates `get_prompt_logprobs` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_prompt_logprobs`，以便周围测试复用准备、执行或校验逻辑。
- `_MockLinearBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_MockFusedMoE`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_MockParallelLMHead`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_build_qwen3_mock`: Build a lightweight nn.Module tree that mirrors Qwen3-8B's named modules. / 该代码块实现 `_build_qwen3_mock`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoRAQwen3_8BLogprobDiff`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRAQwen3_8BLogprobDiff.test_auto_detect_lora_target_modules`: Verify auto_detect_lora_target_modules returns the expected module set for a Qwen3-8B-like (dense) architecture. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_lora_target_modules`。
- `TestLoRAQwen3_8BLogprobDiff.test_lora_qwen3_8b_logprob_accuracy`: This test exercises `test_lora_qwen3_8b_logprob_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_qwen3_8b_logprob_accuracy`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `os`, `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn`, `huggingface_hub`
- **Internal modules / 内部模块**: `sglang`, `sglang.srt.lora.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 201
