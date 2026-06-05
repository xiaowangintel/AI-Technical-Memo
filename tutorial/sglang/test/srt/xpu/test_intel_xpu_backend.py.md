# test_intel_xpu_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/xpu/test_intel_xpu_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates intel xpu backend behavior in SGLang's srt / xpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / xpu 领域中与 intel xpu backend 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: supporting statements / 辅助语句
```python
"""
Usage:
python3 -m unittest test_intel_xpu_backend.TestIntelXPUBackend.test_latency_qwen_model
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 6-16: module imports and dependencies / 模块导入与依赖
```python
import unittest
from functools import wraps

from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST_FP8_WITH_MOE,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST_BASE,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST_QWEN,
    CustomTestCase,
    is_in_ci,
    run_bench_one_batch,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `functools`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `functools`, `sglang.test.test_utils`。

### Lines 19-53: function intel xpu benchmark / 函数 intel xpu benchmark
```python
def intel_xpu_benchmark(
    extra_args=None, min_throughput=None, mem_fraction_static="0.4"
):
    def decorator(test_func):
        @wraps(test_func)
        def wrapper(self):
            common_args = [
                "--disable-radix-cache",
                "--trust-remote-code",
                "--mem-fraction-static",
                str(mem_fraction_static),
                "--batch-size",
                "1",
                "--device",
                "xpu",
            ]
            ci_args = ["--input", "64", "--output", "4"] if is_in_ci() else []
            full_args = common_args + ci_args + (extra_args or [])

            model = test_func(self)
            prefill_latency, decode_throughput, decode_latency = run_bench_one_batch(
                model, full_args
            )

            print(f"{model=}")
            print(f"{prefill_latency=}")
            print(f"{decode_throughput=}")
            print(f"{decode_latency=}")

            if is_in_ci() and min_throughput is not None:
                self.assertGreater(decode_throughput, min_throughput)

        return wrapper

    return decorator
```
**EN:** This block implements `intel_xpu_benchmark` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `intel_xpu_benchmark`，承担模块行为中的一个聚焦逻辑片段。

### Lines 56-57: class TestIntelXPUBackend declaration / 类 TestIntelXPUBackend 声明
```python
class TestIntelXPUBackend(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 58-60: test case latency qwen model / 测试用例 latency qwen model
```python
    @intel_xpu_benchmark(min_throughput=10, mem_fraction_static="0.3")
    def test_latency_qwen_model(self):
        return DEFAULT_SMALL_MODEL_NAME_FOR_TEST_QWEN
```
**EN:** This test exercises `test_latency_qwen_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_latency_qwen_model`。

### Lines 62-67: test case attention backend / 测试用例 attention backend
```python
    @intel_xpu_benchmark(
        ["--attention-backend", "intel_xpu", "--page-size", "128"],
        mem_fraction_static="0.5",
    )
    def test_attention_backend(self):
        return DEFAULT_SMALL_MODEL_NAME_FOR_TEST_BASE
```
**EN:** This test exercises `test_attention_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_attention_backend`。

### Lines 69-79: test case mla decode attention backend / 测试用例 mla decode attention backend
```python
    @intel_xpu_benchmark(
        [
            "--json-model-override-args",
            '{"num_hidden_layers": 4}',
            "--decode-attention-backend",
            "intel_xpu",
        ],
        min_throughput=32,
    )
    def test_mla_decode_attention_backend(self):
        return DEFAULT_MODEL_NAME_FOR_TEST_FP8_WITH_MOE
```
**EN:** This test exercises `test_mla_decode_attention_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mla_decode_attention_backend`。

### Lines 82-83: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `intel_xpu_benchmark`: This block implements `intel_xpu_benchmark` and captures one focused piece of the module's behavior. / 该代码块实现 `intel_xpu_benchmark`，承担模块行为中的一个聚焦逻辑片段。
- `TestIntelXPUBackend`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestIntelXPUBackend.test_latency_qwen_model`: This test exercises `test_latency_qwen_model` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_latency_qwen_model`。
- `TestIntelXPUBackend.test_attention_backend`: This test exercises `test_attention_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_attention_backend`。
- `TestIntelXPUBackend.test_mla_decode_attention_backend`: This test exercises `test_mla_decode_attention_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mla_decode_attention_backend`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `functools`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 83
