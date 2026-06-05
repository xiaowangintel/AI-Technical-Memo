# test_profile_merger_http_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_profile_merger_http_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates profile merger http api behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 profile merger http api 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

from sglang.srt.managers.io_struct import ProfileReqInput
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `sglang.srt.managers.io_struct`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `sglang.srt.managers.io_struct`, `sglang.test.ci.ci_register`。

### Lines 7-8: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=8, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=9, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 11-11: class TestProfileMergerHTTPAPI declaration / 类 TestProfileMergerHTTPAPI 声明
```python
class TestProfileMergerHTTPAPI(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 12-39: test case profile req input merge profiles json serialization / 测试用例 profile req input merge profiles json serialization
```python
    def test_profile_req_input_merge_profiles_json_serialization(self):
        # Test with merge_profiles=True
        req_input = ProfileReqInput(
            output_dir="/tmp/test",
            num_steps=5,
            activities=["CPU", "GPU"],
            profile_by_stage=True,
            merge_profiles=True,
        )

        # Convert to dict (as would happen in HTTP request)
        req_dict = {
            "output_dir": req_input.output_dir,
            "num_steps": req_input.num_steps,
            "activities": req_input.activities,
            "profile_by_stage": req_input.profile_by_stage,
            "merge_profiles": req_input.merge_profiles,
        }

        # Test JSON serialization
        json_str = json.dumps(req_dict)
        parsed_data = json.loads(json_str)

        self.assertTrue(parsed_data["merge_profiles"])
        self.assertEqual(parsed_data["output_dir"], "/tmp/test")
        self.assertEqual(parsed_data["num_steps"], 5)
        self.assertEqual(parsed_data["activities"], ["CPU", "GPU"])
        self.assertTrue(parsed_data["profile_by_stage"])
```
**EN:** This test exercises `test_profile_req_input_merge_profiles_json_serialization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_json_serialization`。

### Lines 41-58: test case profile req input merge profiles json deserialization / 测试用例 profile req input merge profiles json deserialization
```python
    def test_profile_req_input_merge_profiles_json_deserialization(self):
        # Test JSON data as would come from HTTP request
        json_data = {
            "output_dir": "/tmp/test",
            "num_steps": 10,
            "activities": ["CPU", "GPU", "MEM"],
            "profile_by_stage": False,
            "merge_profiles": True,
        }

        # Create ProfileReqInput from dict (as HTTP server would do)
        req_input = ProfileReqInput(**json_data)

        self.assertTrue(req_input.merge_profiles)
        self.assertEqual(req_input.output_dir, "/tmp/test")
        self.assertEqual(req_input.num_steps, 10)
        self.assertEqual(req_input.activities, ["CPU", "GPU", "MEM"])
        self.assertFalse(req_input.profile_by_stage)
```
**EN:** This test exercises `test_profile_req_input_merge_profiles_json_deserialization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_json_deserialization`。

### Lines 60-65: test case profile req input merge profiles default value / 测试用例 profile req input merge profiles default value
```python
    def test_profile_req_input_merge_profiles_default_value(self):
        # Test with minimal data
        json_data = {"output_dir": "/tmp/test"}

        req_input = ProfileReqInput(**json_data)
        self.assertFalse(req_input.merge_profiles)
```
**EN:** This test exercises `test_profile_req_input_merge_profiles_default_value` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_default_value`。

### Lines 67-71: test case profile req input merge profiles explicit false / 测试用例 profile req input merge profiles explicit false
```python
    def test_profile_req_input_merge_profiles_explicit_false(self):
        json_data = {"output_dir": "/tmp/test", "merge_profiles": False}

        req_input = ProfileReqInput(**json_data)
        self.assertFalse(req_input.merge_profiles)
```
**EN:** This test exercises `test_profile_req_input_merge_profiles_explicit_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_explicit_false`。

### Lines 73-91: test case http api parameter flow / 测试用例 http api parameter flow
```python
    def test_http_api_parameter_flow(self):
        # Simulate HTTP request data
        request_data = {
            "output_dir": "/tmp/test",
            "num_steps": 5,
            "activities": ["CPU", "GPU"],
            "profile_by_stage": True,
            "merge_profiles": True,
        }

        # Create ProfileReqInput as HTTP server would
        obj = ProfileReqInput(**request_data)

        # Verify the parameter is set correctly
        self.assertTrue(obj.merge_profiles)
        self.assertEqual(obj.output_dir, "/tmp/test")
        self.assertEqual(obj.num_steps, 5)
        self.assertEqual(obj.activities, ["CPU", "GPU"])
        self.assertTrue(obj.profile_by_stage)
```
**EN:** This test exercises `test_http_api_parameter_flow` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_parameter_flow`。

### Lines 93-107: test case http api parameter validation / 测试用例 http api parameter validation
```python
    def test_http_api_parameter_validation(self):
        # Test with True
        json_data = {"merge_profiles": True}
        req_input = ProfileReqInput(**json_data)
        self.assertTrue(req_input.merge_profiles)

        # Test with False
        json_data = {"merge_profiles": False}
        req_input = ProfileReqInput(**json_data)
        self.assertFalse(req_input.merge_profiles)

        # Test with string "true" (should be converted by JSON parser)
        json_data = {"merge_profiles": "true"}
        req_input = ProfileReqInput(**json_data)
        self.assertEqual(req_input.merge_profiles, "true")  # String, not boolean
```
**EN:** This test exercises `test_http_api_parameter_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_parameter_validation`。

### Lines 109-122: test case http api backward compatibility / 测试用例 http api backward compatibility
```python
    def test_http_api_backward_compatibility(self):
        # Test minimal request (no merge_profiles)
        json_data = {}
        req_input = ProfileReqInput(**json_data)
        self.assertFalse(req_input.merge_profiles)  # Should default to False

        # Test with other parameters but no merge_profiles
        json_data = {
            "output_dir": "/tmp/test",
            "num_steps": 5,
            "activities": ["CPU", "GPU"],
        }
        req_input = ProfileReqInput(**json_data)
        self.assertFalse(req_input.merge_profiles)  # Should default to False
```
**EN:** This test exercises `test_http_api_backward_compatibility` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_backward_compatibility`。

### Lines 124-162: test case http api parameter combinations / 测试用例 http api parameter combinations
```python
    def test_http_api_parameter_combinations(self):
        test_cases = [
            {
                "name": "minimal with merge_profiles",
                "data": {"merge_profiles": True},
                "expected_merge": True,
            },
            {
                "name": "full parameters with merge_profiles=True",
                "data": {
                    "output_dir": "/tmp/test",
                    "num_steps": 10,
                    "activities": ["CPU", "GPU", "MEM"],
                    "profile_by_stage": True,
                    "with_stack": True,
                    "record_shapes": True,
                    "merge_profiles": True,
                },
                "expected_merge": True,
            },
            {
                "name": "full parameters with merge_profiles=False",
                "data": {
                    "output_dir": "/tmp/test",
                    "num_steps": 10,
                    "activities": ["CPU", "GPU", "MEM"],
                    "profile_by_stage": False,
                    "with_stack": False,
                    "record_shapes": False,
                    "merge_profiles": False,
                },
                "expected_merge": False,
            },
        ]

        for test_case in test_cases:
            with self.subTest(test_case["name"]):
                req_input = ProfileReqInput(**test_case["data"])
                self.assertEqual(req_input.merge_profiles, test_case["expected_merge"])
```
**EN:** This test exercises `test_http_api_parameter_combinations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_parameter_combinations`。

### Lines 165-166: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestProfileMergerHTTPAPI`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestProfileMergerHTTPAPI.test_profile_req_input_merge_profiles_json_serialization`: This test exercises `test_profile_req_input_merge_profiles_json_serialization` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_json_serialization`。
- `TestProfileMergerHTTPAPI.test_profile_req_input_merge_profiles_json_deserialization`: This test exercises `test_profile_req_input_merge_profiles_json_deserialization` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_json_deserialization`。
- `TestProfileMergerHTTPAPI.test_profile_req_input_merge_profiles_default_value`: This test exercises `test_profile_req_input_merge_profiles_default_value` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_default_value`。
- `TestProfileMergerHTTPAPI.test_profile_req_input_merge_profiles_explicit_false`: This test exercises `test_profile_req_input_merge_profiles_explicit_false` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_profile_req_input_merge_profiles_explicit_false`。
- `TestProfileMergerHTTPAPI.test_http_api_parameter_flow`: This test exercises `test_http_api_parameter_flow` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_parameter_flow`。
- `TestProfileMergerHTTPAPI.test_http_api_parameter_validation`: This test exercises `test_http_api_parameter_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_parameter_validation`。
- `TestProfileMergerHTTPAPI.test_http_api_backward_compatibility`: This test exercises `test_http_api_backward_compatibility` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_backward_compatibility`。
- `TestProfileMergerHTTPAPI.test_http_api_parameter_combinations`: This test exercises `test_http_api_parameter_combinations` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_api_parameter_combinations`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.managers.io_struct`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 166
