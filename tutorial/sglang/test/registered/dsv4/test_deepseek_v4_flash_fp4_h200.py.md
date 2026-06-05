# test_deepseek_v4_flash_fp4_h200.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/dsv4/test_deepseek_v4_flash_fp4_h200.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates deepseek v4 flash fp4 h200 behavior in SGLang's dsv4 area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 dsv4 领域中与 deepseek v4 flash fp4 h200 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: supporting statements / 辅助语句
```python
"""H200 per-commit CI: DeepSeek-V4-Flash FP4 Marlin (LowLatency recipe).

Launches TP=4 with Marlin FP4 MoE runner + EAGLE speculative decoding.
Runs 12 ServerSanity probes (correctness, streaming, concurrency, determinism)
plus a GSM8K accuracy gate.

Registry: base-c-test-dsv4-8-gpu-h200 (per-commit, 8x H200 — only 4 used by TP=4)
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 10-22: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.server_sanity_kit import ServerSanityMixin
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
    try_cached_model,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `sglang.srt.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `sglang.srt.utils`, `sglang.test.ci.ci_register`。

### Lines 24-24: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=370, stage="base-c", runner_config="dsv4-8-gpu-h200")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-35: function flashinfer has sm90 cutlass mxfp4 / 函数 flashinfer has sm90 cutlass mxfp4
```python
def _flashinfer_has_sm90_cutlass_mxfp4() -> bool:
    try:
        from flashinfer.fused_moe import (  # noqa: F401
            interleave_moe_weights_for_sm90_mixed_gemm,
        )

        return True
    except ImportError:
        return False
```
**EN:** This block implements `_flashinfer_has_sm90_cutlass_mxfp4` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_flashinfer_has_sm90_cutlass_mxfp4`，承担模块行为中的一个聚焦逻辑片段。

### Lines 38-41: module-level constants and configuration / 模块级常量与配置
```python
MODEL = "deepseek-ai/DeepSeek-V4-Flash"
MODEL_FP8 = "sgl-project/DeepSeek-V4-Flash-FP8"
SERVER_LAUNCH_TIMEOUT = 3600
DEEPEP_CONFIG = '{"normal_dispatch":{"num_sms":96},"normal_combine":{"num_sms":96}}'
```
**EN:** This block defines shared names such as `MODEL`, `MODEL_FP8`, `SERVER_LAUNCH_TIMEOUT`, `DEEPEP_CONFIG`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `MODEL`, `MODEL_FP8`, `SERVER_LAUNCH_TIMEOUT`, `DEEPEP_CONFIG` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 44-44: class TestDSV4FlashFP4H200 declaration / 类 TestDSV4FlashFP4H200 声明
```python
class TestDSV4FlashFP4H200(ServerSanityMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ServerSanityMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ServerSanityMixin`, `CustomTestCase`。

### Lines 45-45: supporting statements / 辅助语句
```python
    """LowLatency recipe: TP=4, Marlin FP4, EAGLE spec decoding."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 47-72: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = try_cached_model(MODEL)
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=SERVER_LAUNCH_TIMEOUT,
            other_args=[
                "--trust-remote-code",
                "--tp",
                "4",
                "--moe-runner-backend",
                "marlin",
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-num-steps",
                "3",
                "--speculative-eagle-topk",
                "1",
                "--speculative-num-draft-tokens",
                "4",
                "--watchdog-timeout",
                "900",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 74-77: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 79-91: test case gsm8k / 测试用例 gsm8k
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
        print(f"[DSV4 Flash FP4 Marlin H200] GSM8K {metrics=}")
        self.assertGreater(metrics["score"], 0.93)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 94-98: class TestDSV4FlashFP4H200FlashInferCutlass declaration / 类 TestDSV4FlashFP4H200FlashInferCutlass 声明
```python
@unittest.skipUnless(
    _flashinfer_has_sm90_cutlass_mxfp4(),
    "FlashInfer build lacks SM90 mixed-input MXFP4 helpers (PR #3084, >= 0.6.11)",
)
class TestDSV4FlashFP4H200FlashInferCutlass(ServerSanityMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `ServerSanityMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `ServerSanityMixin`, `CustomTestCase`。

### Lines 99-104: supporting statements / 辅助语句
```python
    """FlashInfer SM90 mixed-input cutlass MXFP4 backend (this PR): TP=4 + EAGLE.

    Mirrors :class:`TestDSV4FlashFP4H200` but swaps `--moe-runner-backend marlin`
    for `flashinfer_mxfp4`, exercising the SM90 cutlass path from FlashInfer PR
    #3084 end-to-end on a real DSv4-Flash checkpoint.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 106-129: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = try_cached_model(MODEL)
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=SERVER_LAUNCH_TIMEOUT,
            other_args=[
                "--trust-remote-code",
                "--tp",
                "4",
                "--moe-runner-backend",
                "flashinfer_mxfp4",
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-num-steps",
                "3",
                "--speculative-eagle-topk",
                "1",
                "--speculative-num-draft-tokens",
                "4",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 131-134: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 136-148: test case gsm8k / 测试用例 gsm8k
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
        print(f"[DSV4 Flash FP4 FlashInfer Cutlass H200] GSM8K {metrics=}")
        self.assertGreater(metrics["score"], 0.93)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 151-152: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_flashinfer_has_sm90_cutlass_mxfp4`: This block implements `_flashinfer_has_sm90_cutlass_mxfp4` and captures one focused piece of the module's behavior. / 该代码块实现 `_flashinfer_has_sm90_cutlass_mxfp4`，承担模块行为中的一个聚焦逻辑片段。
- `TestDSV4FlashFP4H200`: LowLatency recipe: TP=4, Marlin FP4, EAGLE spec decoding. / 用于组织相关测试、夹具或辅助方法。
- `TestDSV4FlashFP4H200FlashInferCutlass`: FlashInfer SM90 mixed-input cutlass MXFP4 backend (this PR): TP=4 + EAGLE. / 用于组织相关测试、夹具或辅助方法。
- `TestDSV4FlashFP4H200.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDSV4FlashFP4H200.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestDSV4FlashFP4H200.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestDSV4FlashFP4H200FlashInferCutlass.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDSV4FlashFP4H200FlashInferCutlass.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestDSV4FlashFP4H200FlashInferCutlass.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.server_sanity_kit`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 152
