# test_mla_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/mla/test_mla_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mla fp8 behavior in SGLang's mla area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 mla 领域中与 mla fp8 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import MGSMEnMixin
from sglang.test.test_utils import (
    DEFAULT_MLA_FP8_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`。

### Lines 14-15: supporting source context / 辅助源码上下文
```python

# MLA FP8 KV cache test with MGSM evaluation
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 16-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=104, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=800, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class TestMLA declaration / 类 TestMLA 声明
```python
class TestMLA(CustomTestCase, MGSMEnMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `MGSMEnMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `MGSMEnMixin`。

### Lines 21-21: class-level constants and configuration for `TestMLA` / 类级常量与配置
```python
    mgsm_en_score_threshold = 0.8
```
**EN:** This block defines shared names such as `mgsm_en_score_threshold`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `mgsm_en_score_threshold` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 23-47: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MLA_FP8_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--kv-cache-dtype",
            "fp8_e5m2",
            # Pin MoE expert dispatch and kernel reduction order so MGSM
            # scores don't drift across runs. The eval already uses greedy
            # decoding, but FP8 dequant + non-deterministic MoE top-k
            # tie-breaks produce ~1–3 point swings without this flag and
            # straddle the 0.8 threshold. With deterministic inference,
            # the score becomes a fixed function of (model, weights, CUDA
            # stack), so threshold-edge flakes stop being random noise.
        ]
        if not is_in_amd_ci():
            # On AMD, the default attention backend (aiter) is not in the deterministic-inference allowlist, so the server fails to start, disable it.
            other_args.append("--enable-deterministic-inference")
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 49-51: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 54-55: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMLA`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMLA.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMLA.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 55
