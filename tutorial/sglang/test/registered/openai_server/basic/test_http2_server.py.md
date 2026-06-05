# test_http2_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/openai_server/basic/test_http2_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates http2 server behavior in SGLang's openai server / basic area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 openai server / basic 领域中与 http2 server 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
Test HTTP/2 server (Granian) with basic OpenAI-compatible endpoints.

Verifies that --enable-http2 launches successfully and serves requests
via both HTTP/1.1 and HTTP/2 (h2c).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-21: module imports and dependencies / 模块导入与依赖
```python
import subprocess
import unittest

import requests

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
**EN:** This block imports the modules needed by the rest of the file, including `subprocess`, `unittest`, `requests`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `subprocess`, `unittest`, `requests`, `sglang.srt.utils`。

### Lines 23-31: CI registration and metadata / CI 注册与元数据
```python
try:
    import granian  # noqa: F401

    _HAS_GRANIAN = True
except ImportError:
    _HAS_GRANIAN = False

register_cuda_ci(est_time=52, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=52, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 34-35: class TestHTTP2Server declaration / 类 TestHTTP2Server 声明
```python
@unittest.skipUnless(_HAS_GRANIAN, "granian not installed (pip install sglang[http2])")
class TestHTTP2Server(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 36-45: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--enable-http2"],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 47-49: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 51-53: test case health / 测试用例 health
```python
    def test_health(self):
        resp = requests.get(f"{self.base_url}/health")
        self.assertEqual(resp.status_code, 200)
```
**EN:** This test exercises `test_health` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health`。

### Lines 55-58: test case get model info / 测试用例 get model info
```python
    def test_get_model_info(self):
        resp = requests.get(f"{self.base_url}/get_model_info")
        self.assertEqual(resp.status_code, 200)
        self.assertIn("model_path", resp.json())
```
**EN:** This test exercises `test_get_model_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_model_info`。

### Lines 60-73: test case completion / 测试用例 completion
```python
    def test_completion(self):
        resp = requests.post(
            f"{self.base_url}/v1/completions",
            json={
                "model": self.model,
                "prompt": "The capital of France is",
                "max_tokens": 8,
                "temperature": 0,
            },
        )
        self.assertEqual(resp.status_code, 200)
        data = resp.json()
        self.assertIn("choices", data)
        self.assertGreater(len(data["choices"][0]["text"]), 0)
```
**EN:** This test exercises `test_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion`。

### Lines 75-88: test case chat completion / 测试用例 chat completion
```python
    def test_chat_completion(self):
        resp = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Say hello"}],
                "max_tokens": 16,
                "temperature": 0,
            },
        )
        self.assertEqual(resp.status_code, 200)
        data = resp.json()
        self.assertIn("choices", data)
        self.assertGreater(len(data["choices"][0]["message"]["content"]), 0)
```
**EN:** This test exercises `test_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion`。

### Lines 90-109: test case h2c with curl / 测试用例 h2c with curl
```python
    def test_h2c_with_curl(self):
        """Verify the server actually speaks HTTP/2 via h2c."""
        result = subprocess.run(
            [
                "curl",
                "--http2-prior-knowledge",
                "-s",
                "-o",
                "/dev/null",
                "-w",
                "%{http_version}",
                f"{self.base_url}/health",
            ],
            capture_output=True,
            text=True,
            timeout=10,
        )
        self.assertEqual(
            result.stdout.strip(), "2", "Server should respond with HTTP/2"
        )
```
**EN:** Verify the server actually speaks HTTP/2 via h2c. This test exercises `test_h2c_with_curl` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify the server actually speaks HTTP/2 via h2c. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_h2c_with_curl`。

### Lines 112-113: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestHTTP2Server`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestHTTP2Server.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestHTTP2Server.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestHTTP2Server.test_health`: This test exercises `test_health` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_health`。
- `TestHTTP2Server.test_get_model_info`: This test exercises `test_get_model_info` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_model_info`。
- `TestHTTP2Server.test_completion`: This test exercises `test_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion`。
- `TestHTTP2Server.test_chat_completion`: This test exercises `test_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion`。
- `TestHTTP2Server.test_h2c_with_curl`: Verify the server actually speaks HTTP/2 via h2c. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_h2c_with_curl`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `subprocess`, `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 113
