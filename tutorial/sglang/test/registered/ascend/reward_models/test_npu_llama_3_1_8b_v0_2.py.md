# test_npu_llama_3_1_8b_v0_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/reward_models/test_npu_llama_3_1_8b_v0_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on reward models npu llama 3 1 8b v0 2 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 reward models npu llama 3 1 8b v0 2 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Import dependencies
```python
import multiprocessing as mp
import unittest

import torch

from sglang.test.ascend.test_ascend_utils import (
    SKYWORK_REWARD_LLAMA_3_1_8B_V0_2_WEIGHTS_PATH,
)
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.runners import HFRunner, SRTRunner
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 13-13: Register CI metadata
```python
register_npu_ci(est_time=400, suite="full-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 15-29: Define module constants
```python
MODELS = [
    (SKYWORK_REWARD_LLAMA_3_1_8B_V0_2_WEIGHTS_PATH, 1, 4e-2),
]
TORCH_DTYPES = [torch.float16]

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
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 32-32: Define class TestLlama
```python
class TestLlama(CustomTestCase):
```
**EN:** This declaration introduces the `TestLlama` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLlama` 测试类，并说明它通过继承承担的职责。

### Lines 33-38: Document the class `TestLlama`
```python
    """Testcase: This test case validates that the reward scores from the Skywork/Skywork-Reward-Llama-3.1-8B-v0.2 model
    in the SGLang framework are less than 4e-2 different from the Hugging Face implementation.

    [Test Category] Model
    [Test Target] Skywork/Skywork-Reward-Llama-3.1-8B-v0.2
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestLlama`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestLlama`的设计意图。

### Lines 40-42: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 44-75: Define helper: assert close reward scores
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
            tp_size=tp_size,
            torch_dtype=torch_dtype,
            model_type="reward",
        ) as srt_runner:
            prompts = srt_runner.tokenizer.apply_chat_template(convs, tokenize=False)
            srt_outputs = srt_runner.forward(prompts)

        hf_scores = torch.tensor(hf_outputs.scores)
        srt_scores = torch.tensor(srt_outputs.scores)
        print(f"{hf_scores=}")
        print(f"{srt_scores=}")

        assert torch.all(
            abs(hf_scores - srt_scores) < tolerance
        ), "reward scores are not all close"
```
**EN:** This helper function encapsulates reusable logic inside `TestLlama` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestLlama` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 77-82: Run test: reward scores
```python
    def test_reward_scores(self):
        for model, tp_size, tolerance in MODELS:
            for torch_dtype in TORCH_DTYPES:
                self.assert_close_reward_scores(
                    CONVS, model, tp_size, torch_dtype, tolerance
                )
```
**EN:** This test method exercises reward scores and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reward scores 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 85-86: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `multiprocessing`, `torch`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
