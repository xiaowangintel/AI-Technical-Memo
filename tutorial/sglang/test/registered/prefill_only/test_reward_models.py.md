# test_reward_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_reward_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates reward models behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 reward models 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import unittest

import torch

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.runners import HFRunner, SRTRunner
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `unittest`, `torch`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `unittest`, `torch`, `sglang.test.ci.ci_register`。

### Lines 9-26: supporting source context / 辅助源码上下文
```python

# Reward model tests

# Copyright 2023-2024 SGLang Team
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

### Lines 27-51: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=166, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=132, suite="stage-b-test-1-gpu-small-amd-nondeterministic")

MODELS = [
    ("LxzGordon/URM-LLaMa-3.1-8B", 1, 4e-2),
    ("Skywork/Skywork-Reward-Llama-3.1-8B-v0.2", 1, 4e-2),
    # Qwen3-based reward model (uses Qwen3ForSequenceClassification)
    ("Skywork/Skywork-Reward-V2-Qwen3-0.6B", 1, 1.5e-1),
]
TORCH_DTYPES = [torch.float16]

# PROMPT = "Jane has 12 apples. She gives 4 apples to her friend Mark, then buys 1 more apple, and finally splits all her apples equally among herself and her 2 siblings. How many apples does each person get?"
# RESPONSE1 = "1. Jane starts with 12 apples and gives 4 to Mark. 12 - 4 = 8. Jane now has 8 apples.\n2. Jane buys 1 more apple. 8 + 1 = 9. Jane now has 9 apples.\n3. Jane splits the 9 apples equally among herself and her 2 siblings (3 people in total). 9 ÷ 3 = 3 apples each. Each person gets 3 apples."
# RESPONSE2 = "1. Jane starts with 12 apples and gives 4 to Mark. 12 - 4 = 8. Jane now has 8 apples.\n2. Jane buys 1 more apple. 8 + 1 = 9. Jane now has 9 apples.\n3. Jane splits the 9 apples equally among her 2 siblings (2 people in total). 9 ÷ 2 = 4.5 apples each. Each person gets 4 apples."

PROMPT = (
    "What is the range of the numeric output of a sigmoid node in a neural network?"
)
RESPONSE1 = "The output of a sigmoid node is bounded between -1 and 1."
RESPONSE2 = "The output of a sigmoid node is bounded between 0 and 1."

CONVS = [
    [{"role": "user", "content": PROMPT}, {"role": "assistant", "content": RESPONSE1}],
    [{"role": "user", "content": PROMPT}, {"role": "assistant", "content": RESPONSE2}],
]
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 54-55: class TestRewardModels declaration / 类 TestRewardModels 声明
```python
class TestRewardModels(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 56-58: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 60-92: helper routine assert close reward scores / 辅助流程 assert close reward scores
```python
    def assert_close_reward_scores(
        self,
        convs,
        model_path,
        tp_size,
        torch_dtype,
        tolerance,
    ) -> None:
        with HFRunner(
            model_path,
            torch_dtype=torch_dtype,
            model_type="reward",
        ) as hf_runner:
            hf_outputs = hf_runner.forward(convs)

        with SRTRunner(
            model_path,
            torch_dtype=torch_dtype,
            model_type="reward",
        ) as srt_runner:
            prompts = srt_runner.tokenizer.apply_chat_template(
                convs, tokenize=False, return_dict=False
            )
            srt_outputs = srt_runner.forward(prompts)

        hf_scores = torch.tensor(hf_outputs.scores)
        srt_scores = torch.tensor(srt_outputs.scores)
        print(f"{hf_scores=}")
        print(f"{srt_scores=}")

        assert torch.all(
            abs(hf_scores - srt_scores) < tolerance
        ), "reward scores are not all close"
```
**EN:** This helper encapsulates `assert_close_reward_scores` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `assert_close_reward_scores`，以便周围测试复用准备、执行或校验逻辑。

### Lines 94-99: test case reward scores / 测试用例 reward scores
```python
    def test_reward_scores(self):
        for model, tp_size, tolerance in MODELS:
            for torch_dtype in TORCH_DTYPES:
                self.assert_close_reward_scores(
                    CONVS, model, tp_size, torch_dtype, tolerance
                )
```
**EN:** This test exercises `test_reward_scores` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reward_scores`。

### Lines 102-103: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestRewardModels`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRewardModels.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestRewardModels.assert_close_reward_scores`: This helper encapsulates `assert_close_reward_scores` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `assert_close_reward_scores`，以便周围测试复用准备、执行或校验逻辑。
- `TestRewardModels.test_reward_scores`: This test exercises `test_reward_scores` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reward_scores`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 103
