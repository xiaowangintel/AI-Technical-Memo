# test_qwen3_next_models_fp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/models/test_qwen3_next_models_fp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates qwen3 next models fp4 behavior in SGLang's models area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 models 领域中与 qwen3 next models fp4 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.utils import get_device_sm
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.server_fixtures.default_fixture import DefaultServerBase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`。

### Lines 8-10: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=500, suite="nightly-4-gpu-b200", nightly=True)

QWEN3_NEXT_MODEL_FP4 = "nvidia/Qwen3-Next-80B-A3B-Instruct-NVFP4"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 13-16: class TestQwen3NextFp4 declaration / 类 TestQwen3NextFp4 声明
```python
@unittest.skipIf(
    get_device_sm() < 100, "Test requires CUDA SM 100 or higher (Blackwell)"
)
class TestQwen3NextFp4(GSM8KMixin, DefaultServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `GSM8KMixin`, `DefaultServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `GSM8KMixin`, `DefaultServerBase`。

### Lines 17-30: class-level constants and configuration for `TestQwen3NextFp4` / 类级常量与配置
```python
    model = QWEN3_NEXT_MODEL_FP4
    gsm8k_accuracy_thres = 0.93
    other_args = [
        "--tp-size",
        "4",
        "--chunked-prefill-size",
        "2048",
        "--quantization",
        "modelopt_fp4",
        "--mamba-scheduler-strategy",
        "extra_buffer",
        "--mamba-track-interval",
        "128",
    ]
```
**EN:** This block defines shared names such as `model`, `gsm8k_accuracy_thres`, `other_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `gsm8k_accuracy_thres`, `other_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 33-34: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestQwen3NextFp4`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.server_fixtures.default_fixture`

- **Total lines / 总行数**: 34
