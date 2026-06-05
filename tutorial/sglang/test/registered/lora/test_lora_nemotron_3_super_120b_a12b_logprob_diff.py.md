# test_lora_nemotron_3_super_120b_a12b_logprob_diff.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_nemotron_3_super_120b_a12b_logprob_diff.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora nemotron 3 super 120b a12b logprob diff behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora nemotron 3 super 120b a12b logprob diff 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

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
Regression test for NVIDIA-Nemotron-3-Super-120B-A12B-BF16 LoRA logprob accuracy.

Compares SGLang LoRA logprobs against reference training logprobs from a
pre-computed dataset. The LoRA adapter and reference data are downloaded from:
https://huggingface.co/datasets/opherlie/lora-test-case-NVIDIA-Nemotron-3-Super-120B-A12B-BF16

Usage:
    python -m unittest test_lora_nemotron_3_super_120b_a12b_logprob_diff
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 26-35: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import os
import unittest

import torch
from huggingface_hub import snapshot_download

import sglang as sgl
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `os`, `unittest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `os`, `unittest`, `torch`。

### Lines 37-49: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=100, stage="extra-b", runner_config="4-gpu-b200")

BASE_MODEL = "nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-BF16"
LORA_HF_REPO = "opherlie/lora-test-case-NVIDIA-Nemotron-3-Super-120B-A12B-BF16"
LORA_BACKEND = "triton"
MAX_LORA_RANK = 64
TP_SIZE = 4
MOE_RUNNER_BACKEND = "triton"
EXPERTS_SHARED_OUTER_LORAS = True
LORA_USE_VIRTUAL_EXPERTS = True
DISABLE_SHARED_EXPERTS_FUSION = True

KL_THRESHOLD = 2.5e-3
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 52-55: function kl v2 / 函数 kl v2
```python
def kl_v2(a, b):
    a = torch.tensor(a) if not torch.is_tensor(a) else a
    b = torch.tensor(b) if not torch.is_tensor(b) else b
    return (((a - b) ** 2) * 0.5).mean().item()
```
**EN:** This block implements `kl_v2` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `kl_v2`，承担模块行为中的一个聚焦逻辑片段。

### Lines 58-72: helper routine get prompt logprobs / 辅助流程 get prompt logprobs
```python
def get_prompt_logprobs(engine, input_ids, lora_path):
    if isinstance(input_ids, torch.Tensor):
        input_ids = [input_ids.tolist()]
    elif not isinstance(input_ids[0], list):
        input_ids = [input_ids]
    out = engine.generate(
        input_ids=input_ids,
        sampling_params={"max_new_tokens": 0, "temperature": 0.0},
        return_logprob=True,
        logprob_start_len=0,
        lora_path=lora_path,
    )
    if isinstance(out, list):
        out = out[0]
    return [logprob for logprob, _, _ in out["meta_info"]["input_token_logprobs"]][1:]
```
**EN:** This helper encapsulates `get_prompt_logprobs` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_prompt_logprobs`，以便周围测试复用准备、执行或校验逻辑。

### Lines 75-76: class TestLoRANemotron3Super120B_A12B_LogprobDiff declaration / 类 TestLoRANemotron3Super120B_A12B_LogprobDiff 声明
```python
class TestLoRANemotron3Super120B_A12B_LogprobDiff(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 77-137: test case lora nemotron 3 super 120b a12b logprob accuracy / 测试用例 lora nemotron 3 super 120b a12b logprob accuracy
```python
    def test_lora_nemotron_3_super_120b_a12b_logprob_accuracy(self):
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
            moe_runner_backend=MOE_RUNNER_BACKEND,
            experts_shared_outer_loras=EXPERTS_SHARED_OUTER_LORAS,
            lora_use_virtual_experts=LORA_USE_VIRTUAL_EXPERTS,
            disable_shared_experts_fusion=DISABLE_SHARED_EXPERTS_FUSION,
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
**EN:** This test exercises `test_lora_nemotron_3_super_120b_a12b_logprob_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_nemotron_3_super_120b_a12b_logprob_accuracy`。

### Lines 140-151: direct execution entry point / 直接执行入口
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
- `TestLoRANemotron3Super120B_A12B_LogprobDiff`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRANemotron3Super120B_A12B_LogprobDiff.test_lora_nemotron_3_super_120b_a12b_logprob_accuracy`: This test exercises `test_lora_nemotron_3_super_120b_a12b_logprob_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_nemotron_3_super_120b_a12b_logprob_accuracy`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `os`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `huggingface_hub`
- **Internal modules / 内部模块**: `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 151
