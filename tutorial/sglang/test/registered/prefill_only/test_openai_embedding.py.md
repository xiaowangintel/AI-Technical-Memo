# test_openai_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/prefill_only/test_openai_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates openai embedding behavior in SGLang's prefill only area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 prefill only 领域中与 openai embedding 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `openai`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `openai`, `sglang.srt.utils`。

### Lines 16-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=91, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=141, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class TestOpenAIEmbedding declaration / 类 TestOpenAIEmbedding 声明
```python
class TestOpenAIEmbedding(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 21-36: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        # Configure embedding-specific args
        other_args = ["--is-embedding", "--enable-metrics"]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=other_args,
        )
        cls.base_url += "/v1"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 38-40: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 42-47: test case embedding single / 测试用例 embedding single
```python
    def test_embedding_single(self):
        """Test single embedding request"""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.embeddings.create(model=self.model, input="Hello world")
        self.assertEqual(len(response.data), 1)
        self.assertTrue(len(response.data[0].embedding) > 0)
```
**EN:** Test single embedding request This test exercises `test_embedding_single` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test single embedding request 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_single`。

### Lines 49-57: test case embedding batch / 测试用例 embedding batch
```python
    def test_embedding_batch(self):
        """Test batch embedding request"""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.embeddings.create(
            model=self.model, input=["Hello world", "Test text"]
        )
        self.assertEqual(len(response.data), 2)
        self.assertTrue(len(response.data[0].embedding) > 0)
        self.assertTrue(len(response.data[1].embedding) > 0)
```
**EN:** Test batch embedding request This test exercises `test_embedding_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test batch embedding request 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_batch`。

### Lines 59-64: test case embedding single batch str / 测试用例 embedding single batch str
```python
    def test_embedding_single_batch_str(self):
        """Test embedding with a List[str] and length equals to 1"""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.embeddings.create(model=self.model, input=["Hello world"])
        self.assertEqual(len(response.data), 1)
        self.assertTrue(len(response.data[0].embedding) > 0)
```
**EN:** Test embedding with a List[str] and length equals to 1 This test exercises `test_embedding_single_batch_str` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embedding with a List[str] and length equals to 1 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_single_batch_str`。

### Lines 66-81: test case embedding single int list / 测试用例 embedding single int list
```python
    def test_embedding_single_int_list(self):
        """Test embedding with a List[int] or List[List[int]]]"""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.embeddings.create(
            model=self.model,
            input=[[15339, 314, 703, 284, 612, 262, 10658, 10188, 286, 2061]],
        )
        self.assertEqual(len(response.data), 1)
        self.assertTrue(len(response.data[0].embedding) > 0)

        response = client.embeddings.create(
            model=self.model,
            input=[15339, 314, 703, 284, 612, 262, 10658, 10188, 286, 2061],
        )
        self.assertEqual(len(response.data), 1)
        self.assertTrue(len(response.data[0].embedding) > 0)
```
**EN:** Test embedding with a List[int] or List[List[int]]] This test exercises `test_embedding_single_int_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embedding with a List[int] or List[List[int]]] 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_single_int_list`。

### Lines 83-97: test case empty string embedding / 测试用例 empty string embedding
```python
    def test_empty_string_embedding(self):
        """Test embedding an empty string."""

        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        # Text embedding example with empty string
        text = ""
        # Expect a BadRequestError for empty input
        with self.assertRaises(openai.BadRequestError) as cm:
            client.embeddings.create(
                model=self.model,
                input=text,
            )
        # check the status code
        self.assertEqual(cm.exception.status_code, 400)
```
**EN:** Test embedding an empty string. This test exercises `test_empty_string_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embedding an empty string. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_string_embedding`。

### Lines 99-109: test case embedding with dimensions parameter / 测试用例 embedding with dimensions parameter
```python
    def test_embedding_with_dimensions_parameter(self):
        """Test that non-Matryoshka models reject dimensions parameter."""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        # Test that specifying dimensions fails for non-Matryoshka models
        with self.assertRaises(openai.BadRequestError) as cm:
            client.embeddings.create(
                model=self.model, input="Hello world", dimensions=512
            )

        self.assertEqual(cm.exception.status_code, 400)
```
**EN:** Test that non-Matryoshka models reject dimensions parameter. This test exercises `test_embedding_with_dimensions_parameter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that non-Matryoshka models reject dimensions parameter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_with_dimensions_parameter`。

### Lines 112-112: class TestMatryoshkaEmbeddingModel declaration / 类 TestMatryoshkaEmbeddingModel 声明
```python
class TestMatryoshkaEmbeddingModel(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 113-113: supporting statements / 辅助语句
```python
    """Test class for Model that supports Matryoshka embedding functionality, using OpenAI API."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 115-140: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.matryoshka_dims = [128, 256, 512, 768, 1024]

        # Configure embedding-specific args with Matryoshka support via json_model_override_args
        matryoshka_config = {
            "is_matryoshka": True,
            "matryoshka_dimensions": cls.matryoshka_dims,
        }
        other_args = [
            "--is-embedding",
            "--enable-metrics",
            "--json-model-override-args",
            json.dumps(matryoshka_config),
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=other_args,
        )
        cls.base_url += "/v1"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 142-145: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process"):
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 147-158: test case matryoshka embedding valid dimensions / 测试用例 matryoshka embedding valid dimensions
```python
    def test_matryoshka_embedding_valid_dimensions(self):
        """Test Matryoshka embedding with valid dimensions."""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        # Test with various valid dimensions
        for dimensions in self.matryoshka_dims:
            with self.subTest(dimensions=dimensions):
                response = client.embeddings.create(
                    model=self.model, input="Hello world", dimensions=dimensions
                )
                self.assertEqual(len(response.data), 1)
                self.assertEqual(len(response.data[0].embedding), dimensions)
```
**EN:** Test Matryoshka embedding with valid dimensions. This test exercises `test_matryoshka_embedding_valid_dimensions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Matryoshka embedding with valid dimensions. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_matryoshka_embedding_valid_dimensions`。

### Lines 160-172: test case matryoshka embedding batch same dimensions / 测试用例 matryoshka embedding batch same dimensions
```python
    def test_matryoshka_embedding_batch_same_dimensions(self):
        """Test Matryoshka embedding with batch input and same dimensions."""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        response = client.embeddings.create(
            model=self.model,
            input=["Hello world", "Test text", "Another example"],
            dimensions=256,
        )

        self.assertEqual(len(response.data), 3)
        for embedding_data in response.data:
            self.assertEqual(len(embedding_data.embedding), 256)
```
**EN:** Test Matryoshka embedding with batch input and same dimensions. This test exercises `test_matryoshka_embedding_batch_same_dimensions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Matryoshka embedding with batch input and same dimensions. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_matryoshka_embedding_batch_same_dimensions`。

### Lines 174-183: test case matryoshka embedding no dimensions / 测试用例 matryoshka embedding no dimensions
```python
    def test_matryoshka_embedding_no_dimensions(self):
        """Test embedding without specifying dimensions (should use full size)."""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        response = client.embeddings.create(model=self.model, input="Hello world")

        self.assertEqual(len(response.data), 1)

        # Should return full embedding size when no dimensions specified
        self.assertEqual(len(response.data[0].embedding), 1536)
```
**EN:** Test embedding without specifying dimensions (should use full size). This test exercises `test_matryoshka_embedding_no_dimensions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embedding without specifying dimensions (should use full size). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_matryoshka_embedding_no_dimensions`。

### Lines 185-196: test case matryoshka embedding invalid dimensions / 测试用例 matryoshka embedding invalid dimensions
```python
    def test_matryoshka_embedding_invalid_dimensions(self):
        """Test Matryoshka embedding with invalid dimensions."""
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        for dimensions in [100, 0, -1, 10000]:
            with self.assertRaises(openai.BadRequestError) as cm:
                client.embeddings.create(
                    model=self.model,
                    input="Hello world",
                    dimensions=dimensions,
                )
            self.assertEqual(cm.exception.status_code, 400)
```
**EN:** Test Matryoshka embedding with invalid dimensions. This test exercises `test_matryoshka_embedding_invalid_dimensions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Matryoshka embedding with invalid dimensions. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_matryoshka_embedding_invalid_dimensions`。

### Lines 199-200: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestOpenAIEmbedding`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMatryoshkaEmbeddingModel`: Test class for Model that supports Matryoshka embedding functionality, using OpenAI API. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIEmbedding.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestOpenAIEmbedding.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestOpenAIEmbedding.test_embedding_single`: Test single embedding request / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_single`。
- `TestOpenAIEmbedding.test_embedding_batch`: Test batch embedding request / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_batch`。
- `TestOpenAIEmbedding.test_embedding_single_batch_str`: Test embedding with a List[str] and length equals to 1 / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_single_batch_str`。
- `TestOpenAIEmbedding.test_embedding_single_int_list`: Test embedding with a List[int] or List[List[int]]] / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_single_int_list`。
- `TestOpenAIEmbedding.test_empty_string_embedding`: Test embedding an empty string. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_string_embedding`。
- `TestOpenAIEmbedding.test_embedding_with_dimensions_parameter`: Test that non-Matryoshka models reject dimensions parameter. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_with_dimensions_parameter`。
- `TestMatryoshkaEmbeddingModel.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMatryoshkaEmbeddingModel.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 200
