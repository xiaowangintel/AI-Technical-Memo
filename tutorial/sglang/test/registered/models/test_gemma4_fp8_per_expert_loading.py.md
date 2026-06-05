# test_gemma4_fp8_per_expert_loading.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/models/test_gemma4_fp8_per_expert_loading.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates gemma4 fp8 per expert loading behavior in SGLang's models area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 models 领域中与 gemma4 fp8 per expert loading 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: supporting statements / 辅助语句
```python
"""End-to-end test for compressed-tensors per-expert FP8 MoE checkpoint
loading on Gemma4 (e.g. RedHatAI/gemma-4-26B-A4B-it-FP8-Dynamic).

Regression coverage for the load_weights path that recognises
`experts.<id>.{gate,up,down}_proj.{weight,weight_scale}` keys and folds
them into SGLang's fused FusedMoE parameters. Without that path, all
routed-expert weights are silently skipped at load time and the model
emits only `<pad>` tokens at inference (GSM8K collapses to 0.0).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 11-24: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import get_device_sm, kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `requests`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `requests`, `sglang.srt.utils`。

### Lines 25-27: supporting source context / 辅助源码上下文
```python

# Compressed-tensors per-expert FP8 MoE checkpoint that exercises the
# loader path (gated repo + ~27 GB download + 4 GPUs at TP=4).
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 28-28: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, stage="base-c", runner_config="4-gpu-h100")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 31-32: class TestGemma4FP8PerExpertLoading declaration / 类 TestGemma4FP8PerExpertLoading 声明
```python
@unittest.skipIf(get_device_sm() < 90, "Test requires CUDA SM 90 or higher")
class TestGemma4FP8PerExpertLoading(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 33-40: class-level constants and configuration for `TestGemma4FP8PerExpertLoading` / 类级常量与配置
```python
    """Three-stage check that catches the silent-skip failure mode:
    1. server health
    2. completion is not the all-`<pad>` garbage state
    3. GSM8K accuracy matches the BF16 baseline
    """

    model = "RedHatAI/gemma-4-26B-A4B-it-FP8-Dynamic"
    base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This block defines shared names such as `model`, `base_url`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `base_url` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 42-55: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--tp",
                "4",
                "--trust-remote-code",
                "--random-seed",
                "42",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 57-59: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 61-63: test case health / 测试用例 health
```python
    def test_health(self):
        r = requests.get(self.base_url + "/health")
        self.assertEqual(r.status_code, 200)
```
**EN:** This test exercises `test_health` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health`。

### Lines 65-83: test case basic generation not garbage / 测试用例 basic generation not garbage
```python
    def test_basic_generation_not_garbage(self):
        """Pre-fix the server starts but every routed expert is zero-init,
        which leads chat completions to deterministic `<pad>` spam."""
        r = requests.post(
            self.base_url + "/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "What is 7 + 5?"}],
                "temperature": 0,
                "max_tokens": 32,
            },
        )
        self.assertEqual(r.status_code, 200)
        text = r.json()["choices"][0]["message"]["content"]
        self.assertNotIn(
            "<pad>", text, f"Output looks like the pre-fix garbage state: {text!r}"
        )
        self.assertGreater(len(text.strip()), 0, "Empty completion")
        self.assertIn("12", text, f"Expected the answer to mention '12': {text!r}")
```
**EN:** Pre-fix the server starts but every routed expert is zero-init, which leads chat completions to deterministic `<pad>` spam. This test exercises `test_basic_generation_not_garbage` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Pre-fix the server starts but every routed expert is zero-init, which leads chat completions to deterministic `<pad>` spam. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_generation_not_garbage`。

### Lines 85-105: test case gsm8k accuracy / 测试用例 gsm8k accuracy
```python
    def test_gsm8k_accuracy(self):
        """Pre-fix this scores exactly 0.00 (zero routed-expert weights);
        post-fix it matches the BF16 baseline (~0.95 on 20 samples)."""
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            num_examples=20,
            num_threads=16,
        )
        metrics = run_eval(args)
        score = float(metrics["score"])
        print(f"Gemma4 FP8 per-expert GSM8K-20 score: {score:.3f}")
        # Threshold rules out the failure mode (0.00) while leaving ample
        # margin under the BF16 baseline (~0.95).
        self.assertGreaterEqual(
            score,
            0.80,
            f"Per-expert FP8 ckpt accuracy collapsed: {score} "
            "(pre-fix value is 0.00; BF16 baseline is ~0.95).",
        )
```
**EN:** Pre-fix this scores exactly 0.00 (zero routed-expert weights); post-fix it matches the BF16 baseline (~0.95 on 20 samples). This test exercises `test_gsm8k_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Pre-fix this scores exactly 0.00 (zero routed-expert weights); post-fix it matches the BF16 baseline (~0.95 on 20 samples). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_accuracy`。

### Lines 108-109: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestGemma4FP8PerExpertLoading`: Three-stage check that catches the silent-skip failure mode: 1. / 用于组织相关测试、夹具或辅助方法。
- `TestGemma4FP8PerExpertLoading.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestGemma4FP8PerExpertLoading.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestGemma4FP8PerExpertLoading.test_health`: This test exercises `test_health` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health`。
- `TestGemma4FP8PerExpertLoading.test_basic_generation_not_garbage`: Pre-fix the server starts but every routed expert is zero-init, which leads chat completions to deterministic `<pad>` spam. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_generation_not_garbage`。
- `TestGemma4FP8PerExpertLoading.test_gsm8k_accuracy`: Pre-fix this scores exactly 0.00 (zero routed-expert weights); post-fix it matches the BF16 baseline (~0.95 on 20 samples). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_accuracy`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 109
