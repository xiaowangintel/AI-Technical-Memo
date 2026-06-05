# test_structural_tag.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/openai_server/features/test_structural_tag.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `structural tag` scenario in `test/manual/openai_server/features`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/openai_server/features` 中的 `structural tag` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Process control logic / 进程控制逻辑
```python
"""
python3 -m unittest test.srt.openai_server.features.test_structural_tag
"""

import json
import unittest
from typing import Any

import openai

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 21-37: Helper routines around setup_class / 辅助例程
```python
def setup_class(cls, backend: str):
    cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
    cls.base_url = DEFAULT_URL_FOR_TEST

    other_args = [
        "--max-running-requests",
        "10",
        "--grammar-backend",
        backend,
    ]

    cls.process = popen_launch_server(
        cls.model,
        cls.base_url,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        other_args=other_args,
    )
```
**EN:** This range implements helper routine(s) `setup_class` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 40-45: Class definition for TestStructuralTagXGrammarBackend / 类定义
```python
class TestStructuralTagXGrammarBackend(CustomTestCase):
    model: str
    base_url: str
    process: Any

    @classmethod
```
**EN:** This range declares `TestStructuralTagXGrammarBackend`, which organizes the scenario as a reusable test-oriented class. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 46-47: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        setup_class(cls, backend="xgrammar")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `setup_class`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 48-49: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 50-51: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 53-64: Test routines around test_stag_constant_str_openai / 测试例程
```python
    def test_stag_constant_str_openai(self):
        client = openai.Client(api_key="EMPTY", base_url=f"{self.base_url}/v1")

        # even when the answer is ridiculous, the model should follow the instruction
        answer = "The capital of France is Berlin."

        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
```
**EN:** This range defines concrete test routine(s) `test_stag_constant_str_openai`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `Client` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-77: Scenario logic / 场景逻辑
```python
                    "content": "Introduce the capital of France. Return in a JSON format.",
                },
            ],
            temperature=0,
            max_tokens=128,
            response_format={
                "type": "structural_tag",
                "format": {
                    "type": "const_string",
                    "value": answer,
                },
            },
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 78-80: Assertions and result checks / 断言与结果检查
```python

        text = response.choices[0].message.content
        self.assertEqual(text, answer)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-89: Test routines around test_stag_json_schema_openai / 测试例程
```python
    def test_stag_json_schema_openai(self):
        client = openai.Client(api_key="EMPTY", base_url=f"{self.base_url}/v1")
        json_schema = {
            "type": "object",
            "properties": {
                "name": {"type": "string", "pattern": "^[\\w]+$"},
                "population": {"type": "integer"},
            },
```
**EN:** This range defines concrete test routine(s) `test_stag_json_schema_openai`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `Client`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-106: Request and response handling / 请求与响应处理
```python
            "required": ["name", "population"],
            "additionalProperties": False,
        }

        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": "Introduce the capital of France. Return in a JSON format.",
                },
            ],
            temperature=0,
            max_tokens=128,
            response_format={
                "type": "structural_tag",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `create`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 107-114: Scenario logic / 场景逻辑
```python
                "format": {
                    "type": "json_schema",
                    "json_schema": json_schema,
                },
            },
        )

        text = response.choices[0].message.content
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 115-122: Assertions and result checks / 断言与结果检查
```python
        try:
            js_obj = json.loads(text)
        except (TypeError, json.decoder.JSONDecodeError):
            print("JSONDecodeError", text)
            raise

        self.assertIsInstance(js_obj["name"], str)
        self.assertIsInstance(js_obj["population"], int)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `loads`, `except` and `assertIsInstance`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 123-126: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `typing`, `unittest`
- **Third-party / 第三方库**: `openai`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.test_utils`
