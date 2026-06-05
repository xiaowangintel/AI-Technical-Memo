# test_constrained_decoding_spec_reasoning.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/test_constrained_decoding_spec_reasoning.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates constrained decoding spec reasoning behavior in SGLang's spec area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 领域中与 constrained decoding spec reasoning 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import openai

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `openai`, `sglang.srt.environ`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `openai`, `sglang.srt.environ`。

### Lines 15-16: supporting source context / 辅助源码上下文
```python

# Constrained decoding with EAGLE3 speculative reasoning (tp=2)
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 17-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=137, stage="base-b", runner_config="2-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class ServerWithGrammar declaration / 类 ServerWithGrammar 声明
```python
class ServerWithGrammar(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 21-37: class-level constants and configuration for `ServerWithGrammar` / 类级常量与配置
```python
    json_schema = json.dumps(
        {
            "type": "object",
            "properties": {
                "name": {"type": "string", "pattern": "^[\\w]+$"},
                "population": {"type": "integer"},
                "languages": {
                    "type": "array",
                    "items": {"type": "string"},
                    "minItems": 1,
                },
                "has_held_olympics": {"type": "boolean"},
            },
            "required": ["name", "population", "languages", "has_held_olympics"],
            "additionalProperties": False,
        }
    )
```
**EN:** This block defines shared names such as `json_schema`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `json_schema` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 39-63: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "openai/gpt-oss-120b"
        cls.base_url = DEFAULT_URL_FOR_TEST
        launch_args = [
            "--trust-remote-code",
            "--tp=2",
            "--reasoning-parser=gpt-oss",
            "--speculative-algorithm=EAGLE3",
            "--speculative-draft-model-path=lmsys/EAGLE3-gpt-oss-120b-bf16",
            "--speculative-num-steps=5",
            "--speculative-eagle-topk=4",
            "--speculative-num-draft-tokens=8",
        ]

        with (
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

### Lines 65-67: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 69-105: test case json openai / 测试用例 json openai
```python
    def test_json_openai(self):
        client = openai.Client(api_key="EMPTY", base_url=f"{self.base_url}/v1")

        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": "Introduce the capital of France. Return in a JSON format. "
                    "The JSON Schema is: " + json.dumps(self.json_schema),
                },
            ],
            temperature=0,
            max_tokens=1024,
            response_format={
                "type": "json_schema",
                "json_schema": {"name": "foo", "schema": json.loads(self.json_schema)},
            },
        )
        text = response.choices[0].message.content

        print("\n=== Reasoning Content ===")
        reasoning_content = response.choices[0].message.reasoning_content
        assert reasoning_content is not None and len(reasoning_content) > 0
        print(reasoning_content)

        try:
            js_obj = json.loads(text)
            print("\n=== Parsed JSON Content ===")
            print(json.dumps(js_obj))
        except (TypeError, json.decoder.JSONDecodeError):
            print("JSONDecodeError", text)
            raise

        self.assertIsInstance(js_obj["name"], str)
        self.assertIsInstance(js_obj["population"], int)
```
**EN:** This test exercises `test_json_openai` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_openai`。

### Lines 108-109: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `ServerWithGrammar`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `ServerWithGrammar.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `ServerWithGrammar.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `ServerWithGrammar.test_json_openai`: This test exercises `test_json_openai` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_json_openai`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 109
