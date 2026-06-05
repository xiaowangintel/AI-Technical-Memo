# test_json_response.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/utils/test_json_response.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates json response behavior in SGLang's unit / utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 工具 领域中与 json response 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import unittest

import numpy as np
import orjson

from sglang.srt.utils.json_response import (
    SGLangORJSONResponse,
    dumps_json,
    orjson_response,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `numpy`, `orjson`, `sglang.srt.utils.json_response`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `numpy`, `orjson`, `sglang.srt.utils.json_response`。

### Lines 13-13: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-16: class TestJSONResponseUtils declaration / 类 TestJSONResponseUtils 声明
```python
class TestJSONResponseUtils(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 17-27: test case dumps json maps non finite values to null / 测试用例 dumps json maps non finite values to null
```python
    def test_dumps_json_maps_non_finite_values_to_null(self):
        payload = {
            "neg_inf": float("-inf"),
            "pos_inf": float("inf"),
            "nan": float("nan"),
        }
        parsed = orjson.loads(dumps_json(payload))

        self.assertIsNone(parsed["neg_inf"])
        self.assertIsNone(parsed["pos_inf"])
        self.assertIsNone(parsed["nan"])
```
**EN:** This test exercises `test_dumps_json_maps_non_finite_values_to_null` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dumps_json_maps_non_finite_values_to_null`。

### Lines 29-37: test case dumps json supports numpy and non string keys / 测试用例 dumps json supports numpy and non string keys
```python
    def test_dumps_json_supports_numpy_and_non_string_keys(self):
        payload = {
            1: np.array([1, 2, 3], dtype=np.int64),
            "scalar": np.float32(1.5),
        }
        parsed = orjson.loads(dumps_json(payload))

        self.assertEqual(parsed["1"], [1, 2, 3])
        self.assertAlmostEqual(parsed["scalar"], 1.5)
```
**EN:** This test exercises `test_dumps_json_supports_numpy_and_non_string_keys` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dumps_json_supports_numpy_and_non_string_keys`。

### Lines 39-45: test case orjson response uses expected media type / 测试用例 orjson response uses expected media type
```python
    def test_orjson_response_uses_expected_media_type(self):
        response = orjson_response({"value": float("-inf")}, status_code=201)
        parsed = orjson.loads(response.body)

        self.assertEqual(response.status_code, 201)
        self.assertEqual(response.media_type, "application/json")
        self.assertIsNone(parsed["value"])
```
**EN:** This test exercises `test_orjson_response_uses_expected_media_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_orjson_response_uses_expected_media_type`。

### Lines 47-51: test case sglang orjson response serializes with shared options / 测试用例 sglang orjson response serializes with shared options
```python
    def test_sglang_orjson_response_serializes_with_shared_options(self):
        response = SGLangORJSONResponse(content={"value": float("-inf")})
        parsed = orjson.loads(response.body)

        self.assertIsNone(parsed["value"])
```
**EN:** This test exercises `test_sglang_orjson_response_serializes_with_shared_options` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sglang_orjson_response_serializes_with_shared_options`。

### Lines 54-55: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestJSONResponseUtils`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestJSONResponseUtils.test_dumps_json_maps_non_finite_values_to_null`: This test exercises `test_dumps_json_maps_non_finite_values_to_null` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dumps_json_maps_non_finite_values_to_null`。
- `TestJSONResponseUtils.test_dumps_json_supports_numpy_and_non_string_keys`: This test exercises `test_dumps_json_supports_numpy_and_non_string_keys` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dumps_json_supports_numpy_and_non_string_keys`。
- `TestJSONResponseUtils.test_orjson_response_uses_expected_media_type`: This test exercises `test_orjson_response_uses_expected_media_type` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_orjson_response_uses_expected_media_type`。
- `TestJSONResponseUtils.test_sglang_orjson_response_serializes_with_shared_options`: This test exercises `test_sglang_orjson_response_serializes_with_shared_options` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sglang_orjson_response_serializes_with_shared_options`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `numpy`, `orjson`
- **Internal modules / 内部模块**: `sglang.srt.utils.json_response`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 55
