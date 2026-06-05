# test_vlm_tp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/vlm/test_vlm_tp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates vlm tp4 behavior in SGLang's vlm area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 vlm 领域中与 vlm tp4 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
VLM TP=4 per-commit test using Qwen3.5-27B with MMMU evaluation.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-16: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `sglang.srt.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `sglang.srt.utils`, `sglang.test.ci.ci_register`。

### Lines 18-22: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=133, stage="base-c", runner_config="4-gpu-h100")

QWEN35_27B_MODEL = "Qwen/Qwen3.5-27B"
MMMU_ACCURACY_THRESHOLD = 0.65
MMMU_NUM_EXAMPLES = 32
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-25: class TestVLMTP4 declaration / 类 TestVLMTP4 声明
```python
class TestVLMTP4(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 26-53: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN35_27B_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--tp-size",
                "4",
                "--cuda-graph-max-bs",
                "32",
                "--mem-fraction-static",
                "0.8",
                "--trust-remote-code",
                "--mamba-scheduler-strategy",
                "extra_buffer",
                "--mamba-track-interval",
                "128",
                "--mamba-ssm-dtype",
                "bfloat16",
                "--chunked-prefill-size",
                "2048",
                "--max-running-requests",
                "128",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 55-58: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 60-78: test case mmmu accuracy / 测试用例 mmmu accuracy
```python
    def test_mmmu_accuracy(self):
        args = SimpleNamespace(
            model=self.model,
            eval_name="mmmu",
            num_examples=MMMU_NUM_EXAMPLES,
            num_threads=16,
            max_tokens=2048,
            chat_template_kwargs={"enable_thinking": False},
            base_url=self.base_url,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval(args)
        print(f"MMMU score: {metrics['score']}")
        self.assertGreaterEqual(
            metrics["score"],
            MMMU_ACCURACY_THRESHOLD,
            f"MMMU accuracy {metrics['score']:.4f} below threshold {MMMU_ACCURACY_THRESHOLD}",
        )
```
**EN:** This test exercises `test_mmmu_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmmu_accuracy`。

### Lines 81-82: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestVLMTP4`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestVLMTP4.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestVLMTP4.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestVLMTP4.test_mmmu_accuracy`: This test exercises `test_mmmu_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmmu_accuracy`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 82
