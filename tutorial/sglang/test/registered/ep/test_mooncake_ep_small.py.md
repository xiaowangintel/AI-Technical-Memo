# test_mooncake_ep_small.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ep/test_mooncake_ep_small.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mooncake ep small behavior in SGLang's ep area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 ep 领域中与 mooncake ep small 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and dependencies / 模块导入与依赖
```python
import os
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import get_rdma_devices_args
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST_MLA,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `unittest`, `types`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `unittest`, `types`, `sglang.srt.utils`。

### Lines 18-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=82, stage="base-c", runner_config="deepep-4-gpu-h100")

ib_devices = get_rdma_devices_args()
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, get_rdma_devices_args.
**CN:** 该代码块通过 register_cuda_ci, get_rdma_devices_args 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 23-23: class TestTP declaration / 类 TestTP 声明
```python
class TestTP(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-24: class-level constants and configuration for `TestTP` / 类级常量与配置
```python
    extra_args = []
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 26-64: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST_MLA
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--tp",
                "4",
                "--elastic-ep-backend",
                "mooncake",
                "--mooncake-ib-device",
                ib_devices,
                "--moe-a2a-backend",
                "mooncake",
                "--deepep-mode",
                "low_latency",
                "--moe-dense-tp-size",
                "1",
                "--enable-dp-lm-head",
                "--enable-two-batch-overlap",
                "--disable-custom-all-reduce",
                "--enable-eplb",
                "--ep-num-redundant-experts",
                "72",
                "--chunked-prefill-size",
                "512",
                "--cuda-graph-max-bs",
                "128",
                "--max-running-requests",
                "512",
                "--mem-fraction-static",
                "0.5",
                *cls.extra_args,
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 66-68: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 70-83: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=200,
            num_threads=128,
        )
        metrics = run_eval(args)
        print(metrics)

        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 86-87: class TestPureDP declaration / 类 TestPureDP 声明
```python
@unittest.skipIf(is_in_ci(), "Skip since mooncake-ep fault-tolerant test is flaky.")
class TestPureDP(TestTP):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestTP`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestTP`。

### Lines 88-95: class-level constants and configuration for `TestPureDP` / 类级常量与配置
```python
    extra_args = [
        "--enable-dp-attention",
        "--dp",
        "4",
    ]

    pkill_process_1 = "sglang::scheduler_DP1_TP1_EP1"
    pkill_process_2 = "sglang::scheduler_DP3_TP3_EP3"
```
**EN:** This block defines shared names such as `extra_args`, `pkill_process_1`, `pkill_process_2`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args`, `pkill_process_1`, `pkill_process_2` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 97-102: test case gsm8k fault 1 / 测试用例 gsm8k fault 1
```python
    def test_gsm8k_fault_1(self):
        """
        Kill one rank and the system should remain operational.
        """
        os.system(f"pkill -f {self.pkill_process_1}")
        super().test_gsm8k()
```
**EN:** Kill one rank and the system should remain operational. This test exercises `test_gsm8k_fault_1` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Kill one rank and the system should remain operational. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_fault_1`。

### Lines 104-110: test case gsm8k fault 2 / 测试用例 gsm8k fault 2
```python
    @unittest.skipIf(is_in_ci(), "To reduce the CI execution time.")
    def test_gsm8k_fault_2(self):
        """
        Kill another rank and the system should remain operational.
        """
        os.system(f"pkill -f {self.pkill_process_2}")
        super().test_gsm8k()
```
**EN:** Kill another rank and the system should remain operational. This test exercises `test_gsm8k_fault_2` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Kill another rank and the system should remain operational. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_fault_2`。

### Lines 113-114: class TestHybridDPTP declaration / 类 TestHybridDPTP 声明
```python
@unittest.skipIf(is_in_ci(), "To reduce the CI execution time.")
class TestHybridDPTP(TestPureDP):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestPureDP`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestPureDP`。

### Lines 115-122: class-level constants and configuration for `TestHybridDPTP` / 类级常量与配置
```python
    extra_args = [
        "--enable-dp-attention",
        "--dp",
        "2",
    ]

    pkill_process_1 = "sglang::scheduler_DP1_TP2_EP2"
    pkill_process_2 = "sglang::scheduler_DP1_TP3_EP3"
```
**EN:** This block defines shared names such as `extra_args`, `pkill_process_1`, `pkill_process_2`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args`, `pkill_process_1`, `pkill_process_2` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 125-126: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestTP`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPureDP`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHybridDPTP`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTP.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestTP.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestTP.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestPureDP.test_gsm8k_fault_1`: Kill one rank and the system should remain operational. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_fault_1`。
- `TestPureDP.test_gsm8k_fault_2`: Kill another rank and the system should remain operational. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_fault_2`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 126
