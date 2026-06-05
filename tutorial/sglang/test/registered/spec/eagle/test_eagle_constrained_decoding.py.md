# test_eagle_constrained_decoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/eagle/test_eagle_constrained_decoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates eagle constrained decoding behavior in SGLang's spec / eagle area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 / eagle 领域中与 eagle constrained decoding 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.json_constrained_kit import JSONConstrainedMixin
from sglang.test.kits.regex_constrained_kit import RegexConstrainedMixin
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE,
    DEFAULT_TARGET_MODEL_EAGLE,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`。

### Lines 17-18: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=116, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=165, stage="stage-b", runner_config="1-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-23: class TestEagleConstrainedDecoding declaration / 类 TestEagleConstrainedDecoding 声明
```python
class TestEagleConstrainedDecoding(
    CustomTestCase, RegexConstrainedMixin, JSONConstrainedMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `RegexConstrainedMixin`, `JSONConstrainedMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `RegexConstrainedMixin`, `JSONConstrainedMixin`。

### Lines 24-34: class-level constants and configuration for `TestEagleConstrainedDecoding` / 类级常量与配置
```python
    max_running_requests = 64
    attention_backend = "triton"
    spec_steps = 5
    spec_topk = 1
    spec_draft_tokens = 6
    page_size = 1
    other_launch_args = []
    model = DEFAULT_TARGET_MODEL_EAGLE
    draft_model = DEFAULT_DRAFT_MODEL_EAGLE
    grammar_backend = "xgrammar"
    spec_v2 = False
```
**EN:** This block defines shared names such as `max_running_requests`, `attention_backend`, `spec_steps`, `spec_topk`, `spec_draft_tokens`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `max_running_requests`, `attention_backend`, `spec_steps`, `spec_topk`, `spec_draft_tokens` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 36-73: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        launch_args = [
            "--trust-remote-code",
            "--attention-backend",
            cls.attention_backend,
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-draft-model",
            cls.draft_model,
            "--speculative-num-steps",
            cls.spec_steps,
            "--speculative-eagle-topk",
            cls.spec_topk,
            "--speculative-num-draft-tokens",
            cls.spec_draft_tokens,
            "--page-size",
            str(cls.page_size),
            "--mem-fraction-static",
            "0.75",
            "--max-running-requests",
            str(cls.max_running_requests),
            "--grammar-backend",
            cls.grammar_backend,
        ]
        launch_args.extend(cls.other_launch_args)
        with (
            envs.SGLANG_ENABLE_SPEC_V2.override(cls.spec_v2),
            envs.SGLANG_SPEC_NAN_DETECTION.override(True),
            envs.SGLANG_SPEC_OOB_DETECTION.override(True),
        ):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=launch_args,
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 75-77: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 80-80: class TestEagleConstrainedDecodingV2 declaration / 类 TestEagleConstrainedDecodingV2 声明
```python
class TestEagleConstrainedDecodingV2(TestEagleConstrainedDecoding):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEagleConstrainedDecoding`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEagleConstrainedDecoding`。

### Lines 81-81: class-level constants and configuration for `TestEagleConstrainedDecodingV2` / 类级常量与配置
```python
    spec_v2 = True
```
**EN:** This block defines shared names such as `spec_v2`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `spec_v2` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 84-85: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEagleConstrainedDecoding`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEagleConstrainedDecodingV2`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEagleConstrainedDecoding.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEagleConstrainedDecoding.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.json_constrained_kit`, `sglang.test.kits.regex_constrained_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 85
