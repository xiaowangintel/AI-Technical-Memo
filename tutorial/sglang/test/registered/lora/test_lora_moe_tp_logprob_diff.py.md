# test_lora_moe_tp_logprob_diff.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_moe_tp_logprob_diff.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora moe tp logprob diff behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora moe tp logprob diff 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: supporting source context / 辅助源码上下文
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

### Lines 16-33: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import unittest
from typing import Any, Dict, List

import torch

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.lora_utils import (
    MOE_BASE_MODEL_PATH,
    MOE_LORA_PATH,
    MOE_LORA_TEST_PROMPTS,
)
from sglang.test.runners import SRTRunner
from sglang.test.test_utils import (
    DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
    CustomTestCase,
    is_in_ci,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `unittest`, `typing`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `unittest`, `typing`, `torch`。

### Lines 35-38: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=200, stage="extra-a", runner_config="2-gpu-large")

LOGPROB_THRESHOLD = 5e-04
MAX_NEW_TOKENS = 10
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 41-71: function run sglang moe lora / 函数 run sglang moe lora
```python
def _run_sglang_moe_lora(
    tp_size: int,
    prompts: List[str],
    port: int = DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
) -> Dict[str, Any]:
    lora_paths_per_prompt = [MOE_LORA_PATH] * len(prompts)

    with SRTRunner(
        model_path=MOE_BASE_MODEL_PATH,
        torch_dtype=torch.bfloat16,
        model_type="generation",
        tp_size=tp_size,
        lora_paths=[MOE_LORA_PATH],
        max_loras_per_batch=1,
        trust_remote_code=True,
        disable_radix_cache=True,
        port=port,
        attention_backend="flashinfer",
        mem_fraction_static=0.80,
    ) as runner:
        outputs = runner.forward(
            prompts,
            max_new_tokens=MAX_NEW_TOKENS,
            lora_paths=lora_paths_per_prompt,
        )

    return {
        "top_input_logprobs": outputs.top_input_logprobs,
        "top_output_logprobs": outputs.top_output_logprobs,
        "output_strs": outputs.output_strs,
    }
```
**EN:** This block implements `_run_sglang_moe_lora` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_sglang_moe_lora`，承担模块行为中的一个聚焦逻辑片段。

### Lines 74-74: class TestMoELoRATP2Logprobs declaration / 类 TestMoELoRATP2Logprobs 声明
```python
class TestMoELoRATP2Logprobs(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 75-76: supporting statements / 辅助语句
```python
    """Compare TP=1 vs TP=2 MoE LoRA: output strings must match and logprobs
    must stay within threshold."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 78-140: method assert tp parity / 方法 assert tp parity
```python
    def _assert_tp_parity(
        self,
        prompts: List[str],
        label: str,
    ):
        print(f"\n{'=' * 100}")
        print(f"  {label}: running TP=1")
        print(f"{'=' * 100}")

        tp1 = _run_sglang_moe_lora(tp_size=1, prompts=prompts)
        torch.cuda.empty_cache()

        print(f"\n{'=' * 100}")
        print(f"  {label}: running TP=2")
        print(f"{'=' * 100}")

        tp2 = _run_sglang_moe_lora(tp_size=2, prompts=prompts)

        print(f"\n{'=' * 100}")
        print(
            f"{'ID':<4} | {'String':<8} | {'Decode Max Diff':<18} | "
            f"{'Decode Mean Diff':<18} | {'Status':<8} | {'Output (TP1)'}"
        )
        print("-" * 100)

        for i in range(len(prompts)):
            tp1_str = tp1["output_strs"][i].strip()
            tp2_str = tp2["output_strs"][i].strip()

            self.assertEqual(
                tp1_str,
                tp2_str,
                f"Output string mismatch on prompt {i}: "
                f"TP1='{tp1_str}' vs TP2='{tp2_str}'",
            )

            tp1_raw = tp1["top_output_logprobs"][i]
            tp2_raw = tp2["top_output_logprobs"][i]
            tp1_lps = torch.tensor(
                [t[0] if isinstance(t, list) else t for t in tp1_raw]
            )
            tp2_lps = torch.tensor(
                [t[0] if isinstance(t, list) else t for t in tp2_raw]
            )
            min_len = min(tp1_lps.shape[0], tp2_lps.shape[0])
            diff = torch.abs(tp1_lps[:min_len] - tp2_lps[:min_len])
            max_diff = torch.max(diff).item() if min_len > 0 else 0.0
            mean_diff = torch.mean(diff).item() if min_len > 0 else 0.0

            status = "PASS" if max_diff < LOGPROB_THRESHOLD else "FAIL"
            print(
                f"{i:<4} | {'OK':<8} | {max_diff:<18.6e} | "
                f"{mean_diff:<18.6e} | {status:<8} | {tp1_str[:40]}"
            )

            self.assertLessEqual(
                max_diff,
                LOGPROB_THRESHOLD,
                f"Decode logprob diff too large on prompt {i}: "
                f"max_diff={max_diff:.6e} > threshold={LOGPROB_THRESHOLD:.0e}",
            )

        print("=" * 100)
```
**EN:** This block implements `_assert_tp_parity` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_tp_parity`，承担模块行为中的一个聚焦逻辑片段。

### Lines 142-147: test case moe lora tp2 vs tp1 basic / 测试用例 moe lora tp2 vs tp1 basic
```python
    def test_moe_lora_tp2_vs_tp1_basic(self):
        """Basic TP=1 vs TP=2 parity with a small prompt set."""
        self._assert_tp_parity(
            prompts=MOE_LORA_TEST_PROMPTS[:5],
            label="MoE LoRA TP parity (basic)",
        )
```
**EN:** Basic TP=1 vs TP=2 parity with a small prompt set. This test exercises `test_moe_lora_tp2_vs_tp1_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Basic TP=1 vs TP=2 parity with a small prompt set. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_lora_tp2_vs_tp1_basic`。

### Lines 149-155: test case moe lora tp2 vs tp1 full / 测试用例 moe lora tp2 vs tp1 full
```python
    @unittest.skipIf(is_in_ci(), "Skipping full test in CI")
    def test_moe_lora_tp2_vs_tp1_full(self):
        """Full TP=1 vs TP=2 parity across all prompts."""
        self._assert_tp_parity(
            prompts=MOE_LORA_TEST_PROMPTS,
            label="MoE LoRA TP parity (full)",
        )
```
**EN:** Full TP=1 vs TP=2 parity across all prompts. This test exercises `test_moe_lora_tp2_vs_tp1_full` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Full TP=1 vs TP=2 parity across all prompts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_lora_tp2_vs_tp1_full`。

### Lines 158-169: direct execution entry point / 直接执行入口
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
- `_run_sglang_moe_lora`: This block implements `_run_sglang_moe_lora` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_sglang_moe_lora`，承担模块行为中的一个聚焦逻辑片段。
- `TestMoELoRATP2Logprobs`: Compare TP=1 vs TP=2 MoE LoRA: output strings must match and logprobs must stay within threshold. / 用于组织相关测试、夹具或辅助方法。
- `TestMoELoRATP2Logprobs._assert_tp_parity`: This block implements `_assert_tp_parity` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_tp_parity`，承担模块行为中的一个聚焦逻辑片段。
- `TestMoELoRATP2Logprobs.test_moe_lora_tp2_vs_tp1_basic`: Basic TP=1 vs TP=2 parity with a small prompt set. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_lora_tp2_vs_tp1_basic`。
- `TestMoELoRATP2Logprobs.test_moe_lora_tp2_vs_tp1_full`: Full TP=1 vs TP=2 parity across all prompts. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_lora_tp2_vs_tp1_full`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.lora_utils`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 169
