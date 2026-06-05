# test_large_max_new_tokens.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/openai_server/validation/test_large_max_new_tokens.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates large max new tokens behavior in SGLang's openai server / validation area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 openai server / validation 领域中与 large max new tokens 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
python3 -m unittest openai_server.validation.test_large_max_new_tokens.TestLargeMaxNewTokens.test_chat_completion
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-23: module imports and dependencies / 模块导入与依赖
```python
import os
import time
import unittest
from concurrent.futures import ThreadPoolExecutor

import openai

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    STDERR_FILENAME,
    STDOUT_FILENAME,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `time`, `unittest`, `concurrent.futures`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `time`, `unittest`, `concurrent.futures`。

### Lines 25-26: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=58, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=41, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 29-29: class TestLargeMaxNewTokens declaration / 类 TestLargeMaxNewTokens 声明
```python
class TestLargeMaxNewTokens(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 30-56: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        cls.stdout = open(STDOUT_FILENAME, "w")
        cls.stderr = open(STDERR_FILENAME, "w")

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=(
                "--max-total-token",
                "1536",
                "--context-len",
                "8192",
                "--decode-log-interval",
                "2",
            ),
            env={"SGLANG_CLIP_MAX_NEW_TOKENS_ESTIMATION": "256", **os.environ},
            return_stdout_stderr=(cls.stdout, cls.stderr),
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 58-64: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        cls.stdout.close()
        cls.stderr.close()
        os.remove(STDOUT_FILENAME)
        os.remove(STDERR_FILENAME)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 66-79: helper routine run chat completion / 辅助流程 run chat completion
```python
    def run_chat_completion(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": "Please repeat the world 'hello' for 10000 times.",
                },
            ],
            temperature=0,
        )
        return response
```
**EN:** This helper encapsulates `run_chat_completion` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_chat_completion`，以便周围测试复用准备、执行或校验逻辑。

### Lines 81-103: test case chat completion / 测试用例 chat completion
```python
    def test_chat_completion(self):
        num_requests = 4

        futures = []
        with ThreadPoolExecutor(num_requests) as executor:
            # Send multiple requests
            for i in range(num_requests):
                futures.append(executor.submit(self.run_chat_completion))

            # Ensure that they are running concurrently
            pt = 0
            while pt >= 0:
                time.sleep(5)
                lines = open(STDERR_FILENAME).readlines()
                for line in lines[pt:]:
                    print(line, end="", flush=True)
                    if f"#running-req: {num_requests}" in line:
                        all_requests_running = True
                        pt = -1
                        break
                    pt += 1

        assert all_requests_running
```
**EN:** This test exercises `test_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion`。

### Lines 106-107: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestLargeMaxNewTokens`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLargeMaxNewTokens.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLargeMaxNewTokens.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestLargeMaxNewTokens.run_chat_completion`: This helper encapsulates `run_chat_completion` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_chat_completion`，以便周围测试复用准备、执行或校验逻辑。
- `TestLargeMaxNewTokens.test_chat_completion`: This test exercises `test_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `time`, `unittest`, `concurrent.futures`
- **Third-party modules / 第三方模块**: `openai`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 107
