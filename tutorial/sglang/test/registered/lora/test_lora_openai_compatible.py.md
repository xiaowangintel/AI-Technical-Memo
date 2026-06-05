# test_lora_openai_compatible.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_openai_compatible.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora openai compatible behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora openai compatible 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting statements / 辅助语句
```python
"""
End-to-end tests for OpenAI-compatible LoRA adapter usage.

Tests the model:adapter syntax and backward compatibility with explicit lora_path.

Usage:
    python3 -m unittest openai_server.features.test_lora_openai_compatible.TestLoRAOpenAICompatible.test_model_adapter_syntax
    python3 -m unittest openai_server.features.test_lora_openai_compatible.TestLoRAOpenAICompatible.test_explicit_lora_path
    python3 -m unittest openai_server.features.test_lora_openai_compatible.TestLoRAOpenAICompatible.test_priority_model_over_explicit
    python3 -m unittest openai_server.features.test_lora_openai_compatible.TestLoRAOpenAICompatible.test_base_model_no_adapter
    python3 -m unittest openai_server.features.test_lora_openai_compatible.TestLoRAOpenAICompatible.test_completions_api_with_adapter
    python3 -m unittest openai_server.features.test_lora_openai_compatible.TestLoRAOpenAICompatible.test_streaming_with_adapter
    python3 -m unittest openai_server.features.test_lora_openai_compatible.TestLoRADisabledError.test_lora_disabled_error
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-28: module imports and dependencies / 模块导入与依赖
```python
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `openai`, `sglang.srt.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `openai`, `sglang.srt.utils`, `sglang.test.ci.ci_register`。

### Lines 30-31: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=150, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=150, suite="nightly-amd-1-gpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 34-36: helper routine get real lora adapter / 辅助流程 get real lora adapter
```python
def get_real_lora_adapter() -> str:
    """Use a real LoRA adapter from Hugging Face."""
    return "codelion/Llama-3.2-1B-Instruct-tool-calling-lora"
```
**EN:** Use a real LoRA adapter from Hugging Face. This helper encapsulates `get_real_lora_adapter` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Use a real LoRA adapter from Hugging Face. 该辅助函数封装了 `get_real_lora_adapter`，以便周围测试复用准备、执行或校验逻辑。

### Lines 39-68: function setup class / 函数 setup class
```python
def setup_class(cls, enable_lora=True):
    """Setup test class with LoRA-enabled server."""
    cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
    cls.base_url = DEFAULT_URL_FOR_TEST

    # Use real LoRA adapter
    cls.lora_adapter_path = get_real_lora_adapter()

    other_args = [
        "--max-running-requests",
        "10",
        "--disable-radix-cache",  # Disable cache for cleaner tests
    ]

    if enable_lora:
        other_args.extend(
            [
                "--enable-lora",
                "--lora-paths",
                f"tool_calling={cls.lora_adapter_path}",
            ]
        )

    cls.process = popen_launch_server(
        cls.model,
        cls.base_url,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        other_args=other_args,
    )
    cls.client = openai.Client(api_key="EMPTY", base_url=f"{cls.base_url}/v1")
```
**EN:** Setup test class with LoRA-enabled server. This block implements `setup_class` and captures one focused piece of the module's behavior.
**CN:** Setup test class with LoRA-enabled server. 该代码块实现 `setup_class`，承担模块行为中的一个聚焦逻辑片段。

### Lines 71-71: class TestLoRAOpenAICompatible declaration / 类 TestLoRAOpenAICompatible 声明
```python
class TestLoRAOpenAICompatible(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 72-72: supporting statements / 辅助语句
```python
    """Test OpenAI-compatible LoRA adapter usage."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 74-76: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        setup_class(cls, enable_lora=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 78-80: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 82-94: test case model adapter syntax / 测试用例 model adapter syntax
```python
    def test_model_adapter_syntax(self):
        """Test the new model:adapter syntax works correctly."""
        response = self.client.chat.completions.create(
            # ← New OpenAI-compatible syntax
            model=f"{self.model}:tool_calling",
            messages=[{"role": "user", "content": "What tools do you have available?"}],
            max_tokens=50,
            temperature=0,
        )

        self.assertIsNotNone(response.choices[0].message.content)
        self.assertGreater(len(response.choices[0].message.content), 0)
        print(f"Model adapter syntax response: {response.choices[0].message.content}")
```
**EN:** Test the new model:adapter syntax works correctly. This test exercises `test_model_adapter_syntax` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the new model:adapter syntax works correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_adapter_syntax`。

### Lines 96-109: test case explicit lora path / 测试用例 explicit lora path
```python
    def test_explicit_lora_path(self):
        """Test backward compatibility with explicit lora_path via extra_body."""
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": "What tools do you have available?"}],
            # ← Legacy explicit method
            extra_body={"lora_path": "tool_calling"},
            max_tokens=50,
            temperature=0,
        )

        self.assertIsNotNone(response.choices[0].message.content)
        self.assertGreater(len(response.choices[0].message.content), 0)
        print(f"Explicit lora_path response: {response.choices[0].message.content}")
```
**EN:** Test backward compatibility with explicit lora_path via extra_body. This test exercises `test_explicit_lora_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test backward compatibility with explicit lora_path via extra_body. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_explicit_lora_path`。

### Lines 111-127: test case priority model over explicit / 测试用例 priority model over explicit
```python
    def test_priority_model_over_explicit(self):
        """Test that model:adapter syntax takes precedence over explicit lora_path."""
        # This test verifies the priority logic in _resolve_lora_path
        response = self.client.chat.completions.create(
            # ← Model specifies tool_calling adapter
            model=f"{self.model}:tool_calling",
            messages=[{"role": "user", "content": "What tools do you have available?"}],
            # ← Both specify same adapter
            extra_body={"lora_path": "tool_calling"},
            max_tokens=50,
            temperature=0,
        )

        # Should use tool_calling adapter (model parameter takes precedence)
        self.assertIsNotNone(response.choices[0].message.content)
        self.assertGreater(len(response.choices[0].message.content), 0)
        print(f"Priority test response: {response.choices[0].message.content}")
```
**EN:** Test that model:adapter syntax takes precedence over explicit lora_path. This test exercises `test_priority_model_over_explicit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that model:adapter syntax takes precedence over explicit lora_path. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_model_over_explicit`。

### Lines 129-140: test case base model no adapter / 测试用例 base model no adapter
```python
    def test_base_model_no_adapter(self):
        """Test using base model without any adapter."""
        response = self.client.chat.completions.create(
            model=self.model,  # ← No adapter specified
            messages=[{"role": "user", "content": "Hello, how are you?"}],
            max_tokens=30,
            temperature=0,
        )

        self.assertIsNotNone(response.choices[0].message.content)
        self.assertGreater(len(response.choices[0].message.content), 0)
        print(f"Base model response: {response.choices[0].message.content}")
```
**EN:** Test using base model without any adapter. This test exercises `test_base_model_no_adapter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test using base model without any adapter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_base_model_no_adapter`。

### Lines 142-153: test case completions api with adapter / 测试用例 completions api with adapter
```python
    def test_completions_api_with_adapter(self):
        """Test completions API with LoRA adapter."""
        response = self.client.completions.create(
            model=f"{self.model}:tool_calling",  # ← Using model:adapter syntax
            prompt="What tools do you have available?",
            max_tokens=50,
            temperature=0,
        )

        self.assertIsNotNone(response.choices[0].text)
        self.assertGreater(len(response.choices[0].text), 0)
        print(f"Completions API response: {response.choices[0].text}")
```
**EN:** Test completions API with LoRA adapter. This test exercises `test_completions_api_with_adapter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test completions API with LoRA adapter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completions_api_with_adapter`。

### Lines 155-171: test case streaming with adapter / 测试用例 streaming with adapter
```python
    def test_streaming_with_adapter(self):
        """Test streaming with LoRA adapter."""
        stream = self.client.chat.completions.create(
            model=f"{self.model}:tool_calling",
            messages=[{"role": "user", "content": "What tools do you have available?"}],
            max_tokens=50,
            temperature=0,
            stream=True,
        )

        collected_content = ""
        for chunk in stream:
            if chunk.choices[0].delta.content:
                collected_content += chunk.choices[0].delta.content

        self.assertGreater(len(collected_content), 0)
        print(f"Streaming response: {collected_content}")
```
**EN:** Test streaming with LoRA adapter. This test exercises `test_streaming_with_adapter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with LoRA adapter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_with_adapter`。

### Lines 173-196: test case multiple adapters / 测试用例 multiple adapters
```python
    def test_multiple_adapters(self):
        """Test using different adapters in sequence."""
        # Test tool_calling adapter
        tool_response = self.client.chat.completions.create(
            model=f"{self.model}:tool_calling",
            messages=[{"role": "user", "content": "What tools do you have available?"}],
            max_tokens=30,
            temperature=0,
        )

        # Test base model without adapter
        base_response = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": "Hello, how are you?"}],
            max_tokens=30,
            temperature=0,
        )

        self.assertIsNotNone(tool_response.choices[0].message.content)
        self.assertIsNotNone(base_response.choices[0].message.content)
        print(
            f"Tool calling adapter response: {tool_response.choices[0].message.content}"
        )
        print(f"Base model response: {base_response.choices[0].message.content}")
```
**EN:** Test using different adapters in sequence. This test exercises `test_multiple_adapters` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test using different adapters in sequence. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_adapters`。

### Lines 199-199: class TestLoRADisabledError declaration / 类 TestLoRADisabledError 声明
```python
class TestLoRADisabledError(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 200-200: supporting statements / 辅助语句
```python
    """Test error handling when LoRA is disabled."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 202-204: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        setup_class(cls, enable_lora=False)  # ← LoRA disabled
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 206-208: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 210-225: test case lora disabled error / 测试用例 lora disabled error
```python
    def test_lora_disabled_error(self):
        """Test that using LoRA adapter when LoRA is disabled raises appropriate error."""
        with self.assertRaises(openai.APIError) as context:
            self.client.chat.completions.create(
                model=f"{self.model}:tool_calling",  # ← Trying to use adapter
                messages=[
                    {"role": "user", "content": "What tools do you have available?"}
                ],
                max_tokens=50,
            )

        # Verify the error message contains helpful guidance
        error_message = str(context.exception)
        self.assertIn("LoRA", error_message)
        self.assertIn("not enabled", error_message)
        print(f"Expected error message: {error_message}")
```
**EN:** Test that using LoRA adapter when LoRA is disabled raises appropriate error. This test exercises `test_lora_disabled_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that using LoRA adapter when LoRA is disabled raises appropriate error. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_disabled_error`。

### Lines 228-228: class TestLoRAEdgeCases declaration / 类 TestLoRAEdgeCases 声明
```python
class TestLoRAEdgeCases(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 229-229: supporting statements / 辅助语句
```python
    """Test edge cases for LoRA adapter usage."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 231-233: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        setup_class(cls, enable_lora=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 235-237: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 239-250: test case model with colon no adapter / 测试用例 model with colon no adapter
```python
    def test_model_with_colon_no_adapter(self):
        """Test model parameter ending with colon (empty adapter)."""
        response = self.client.chat.completions.create(
            model=f"{self.model}:",  # ← Model ends with colon
            messages=[{"role": "user", "content": "Hello!"}],
            max_tokens=30,
            temperature=0,
        )

        # Should work as base model (no adapter)
        self.assertIsNotNone(response.choices[0].message.content)
        print(f"Model with colon response: {response.choices[0].message.content}")
```
**EN:** Test model parameter ending with colon (empty adapter). This test exercises `test_model_with_colon_no_adapter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test model parameter ending with colon (empty adapter). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_with_colon_no_adapter`。

### Lines 252-266: test case explicit lora path none / 测试用例 explicit lora path none
```python
    def test_explicit_lora_path_none(self):
        """Test explicit lora_path set to None."""
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": "Hello!"}],
            extra_body={"lora_path": None},  # ← Explicitly None
            max_tokens=30,
            temperature=0,
        )

        # Should work as base model
        self.assertIsNotNone(response.choices[0].message.content)
        print(
            f"Explicit None lora_path response: {response.choices[0].message.content}"
        )
```
**EN:** Test explicit lora_path set to None. This test exercises `test_explicit_lora_path_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test explicit lora_path set to None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_explicit_lora_path_none`。

### Lines 268-278: test case invalid adapter name / 测试用例 invalid adapter name
```python
    def test_invalid_adapter_name(self):
        """Test using non-existent adapter name."""
        with self.assertRaises(openai.APIError) as context:
            self.client.chat.completions.create(
                model=f"{self.model}:nonexistent",  # ← Non-existent adapter
                messages=[{"role": "user", "content": "Hello!"}],
                max_tokens=30,
            )

        error_message = str(context.exception)
        print(f"Invalid adapter error: {error_message}")
```
**EN:** Test using non-existent adapter name. This test exercises `test_invalid_adapter_name` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test using non-existent adapter name. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_adapter_name`。

### Lines 281-282: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `get_real_lora_adapter`: Use a real LoRA adapter from Hugging Face. / 该辅助函数封装了 `get_real_lora_adapter`，以便周围测试复用准备、执行或校验逻辑。
- `setup_class`: Setup test class with LoRA-enabled server. / 该代码块实现 `setup_class`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoRAOpenAICompatible`: Test OpenAI-compatible LoRA adapter usage. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRADisabledError`: Test error handling when LoRA is disabled. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRAEdgeCases`: Test edge cases for LoRA adapter usage. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRAOpenAICompatible.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLoRAOpenAICompatible.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestLoRAOpenAICompatible.test_model_adapter_syntax`: Test the new model:adapter syntax works correctly. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_adapter_syntax`。
- `TestLoRAOpenAICompatible.test_explicit_lora_path`: Test backward compatibility with explicit lora_path via extra_body. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_explicit_lora_path`。
- `TestLoRAOpenAICompatible.test_priority_model_over_explicit`: Test that model:adapter syntax takes precedence over explicit lora_path. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_model_over_explicit`。
- `TestLoRAOpenAICompatible.test_base_model_no_adapter`: Test using base model without any adapter. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_base_model_no_adapter`。
- `TestLoRAOpenAICompatible.test_completions_api_with_adapter`: Test completions API with LoRA adapter. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completions_api_with_adapter`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 282
