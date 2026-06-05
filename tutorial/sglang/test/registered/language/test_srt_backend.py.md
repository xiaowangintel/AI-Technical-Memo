# test_srt_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/language/test_srt_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates srt backend behavior in SGLang's language area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 language 领域中与 srt backend 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module imports and dependencies / 模块导入与依赖
```python
import unittest

import sglang as sgl
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_programs import (
    test_decode_int,
    test_decode_json_regex,
    test_dtype_gen,
    test_expert_answer,
    test_few_shot_qa,
    test_gen_min_new_tokens,
    test_hellaswag_select,
    test_mt_bench,
    test_parallel_decoding,
    test_regex,
    test_select,
    test_stream,
    test_stream_logprobs,
    test_tool_use,
)
from sglang.test.test_utils import DEFAULT_MODEL_NAME_FOR_TEST, CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_programs`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_programs`。

### Lines 23-24: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=79, stage="base-a", runner_config="1-gpu-small")
register_amd_ci(est_time=120, suite="stage-a-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-27: class TestSRTBackend declaration / 类 TestSRTBackend 声明
```python
class TestSRTBackend(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 28-28: class-level constants and configuration for `TestSRTBackend` / 类级常量与配置
```python
    backend = None
```
**EN:** This block defines shared names such as `backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 30-40: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.backend = sgl.Runtime(
            model_path=DEFAULT_MODEL_NAME_FOR_TEST,
            cuda_graph_max_bs=4,
            mem_fraction_static=0.7,
            incremental_streaming_output=True,
            log_level="info",
            enable_metrics=True,
        )
        sgl.set_default_backend(cls.backend)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 42-44: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        cls.backend.shutdown()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 46-47: test case few shot qa / 测试用例 few shot qa
```python
    def test_few_shot_qa(self):
        test_few_shot_qa()
```
**EN:** This test exercises `test_few_shot_qa` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_few_shot_qa`。

### Lines 49-50: test case mt bench / 测试用例 mt bench
```python
    def test_mt_bench(self):
        test_mt_bench()
```
**EN:** This test exercises `test_mt_bench` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mt_bench`。

### Lines 52-53: test case select / 测试用例 select
```python
    def test_select(self):
        test_select(check_answer=False)
```
**EN:** This test exercises `test_select` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_select`。

### Lines 55-56: test case decode int / 测试用例 decode int
```python
    def test_decode_int(self):
        test_decode_int()
```
**EN:** This test exercises `test_decode_int` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_int`。

### Lines 58-60: test case decode json regex / 测试用例 decode json regex
```python
    @unittest.skip("Skip this flaky test.")
    def test_decode_json_regex(self):
        test_decode_json_regex()
```
**EN:** This test exercises `test_decode_json_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_json_regex`。

### Lines 62-63: test case expert answer / 测试用例 expert answer
```python
    def test_expert_answer(self):
        test_expert_answer()
```
**EN:** This test exercises `test_expert_answer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_expert_answer`。

### Lines 65-66: test case tool use / 测试用例 tool use
```python
    def test_tool_use(self):
        test_tool_use()
```
**EN:** This test exercises `test_tool_use` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_use`。

### Lines 68-69: test case parallel decoding / 测试用例 parallel decoding
```python
    def test_parallel_decoding(self):
        test_parallel_decoding()
```
**EN:** This test exercises `test_parallel_decoding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_decoding`。

### Lines 71-72: test case stream / 测试用例 stream
```python
    def test_stream(self):
        test_stream()
```
**EN:** This test exercises `test_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream`。

### Lines 74-75: test case stream logprobs / 测试用例 stream logprobs
```python
    def test_stream_logprobs(self):
        test_stream_logprobs()
```
**EN:** This test exercises `test_stream_logprobs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_logprobs`。

### Lines 77-78: test case regex / 测试用例 regex
```python
    def test_regex(self):
        test_regex()
```
**EN:** This test exercises `test_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_regex`。

### Lines 80-81: test case dtype gen / 测试用例 dtype gen
```python
    def test_dtype_gen(self):
        test_dtype_gen()
```
**EN:** This test exercises `test_dtype_gen` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dtype_gen`。

### Lines 83-87: test case hellaswag select / 测试用例 hellaswag select
```python
    def test_hellaswag_select(self):
        # Run twice to capture more bugs
        for _ in range(2):
            accuracy, latency = test_hellaswag_select()
            self.assertGreater(accuracy, 0.60)
```
**EN:** This test exercises `test_hellaswag_select` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hellaswag_select`。

### Lines 89-90: test case gen min new tokens / 测试用例 gen min new tokens
```python
    def test_gen_min_new_tokens(self):
        test_gen_min_new_tokens()
```
**EN:** This test exercises `test_gen_min_new_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gen_min_new_tokens`。

### Lines 93-94: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestSRTBackend`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSRTBackend.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSRTBackend.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestSRTBackend.test_few_shot_qa`: This test exercises `test_few_shot_qa` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_few_shot_qa`。
- `TestSRTBackend.test_mt_bench`: This test exercises `test_mt_bench` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mt_bench`。
- `TestSRTBackend.test_select`: This test exercises `test_select` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_select`。
- `TestSRTBackend.test_decode_int`: This test exercises `test_decode_int` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_int`。
- `TestSRTBackend.test_decode_json_regex`: This test exercises `test_decode_json_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_json_regex`。
- `TestSRTBackend.test_expert_answer`: This test exercises `test_expert_answer` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_expert_answer`。
- `TestSRTBackend.test_tool_use`: This test exercises `test_tool_use` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_use`。
- `TestSRTBackend.test_parallel_decoding`: This test exercises `test_parallel_decoding` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_decoding`。
- `TestSRTBackend.test_stream`: This test exercises `test_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang`, `sglang.test.ci.ci_register`, `sglang.test.test_programs`, `sglang.test.test_utils`

- **Total lines / 总行数**: 94
