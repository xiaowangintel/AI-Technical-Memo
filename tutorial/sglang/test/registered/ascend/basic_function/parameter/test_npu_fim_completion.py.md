# test_npu_fim_completion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/parameter/test_npu_fim_completion.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on parameter npu fim completion in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 parameter npu fim completion 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Import dependencies
```python
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ascend.test_ascend_utils import (
    DEEPSEEK_CODER_1_3_B_BASE_PATH,
    DEEPSEEK_CODER_JSON_PATH,
)
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 19-24: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-1-npu-a3",
    nightly=True,
    disabled="run failed",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 27-27: Define class TestFimCompletion
```python
class TestFimCompletion(CustomTestCase):
```
**EN:** This declaration introduces the `TestFimCompletion` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFimCompletion` 测试类，并说明它通过继承承担的职责。

### Lines 28-32: Document the class `TestFimCompletion`
```python
    """Testcase：Verify set --completion-template, the model's FIM (Fill-in-the-Middle) completion function work correctly.

    [Test Category] Parameter
    [Test Target] --completion-template
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFimCompletion`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFimCompletion`的设计意图。

### Lines 34-43: Declare TestFimCompletion configuration
```python
    model = DEEPSEEK_CODER_1_3_B_BASE_PATH
    other_args = [
        "--completion-template",
        "deepseek_coder",
        "--attention-backend",
        "ascend",
        "--disable-cuda-graph",
        "--mem-fraction-static",
        0.8,
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestFimCompletion` test methods.
**CN:** 该代码块定义了 `TestFimCompletion` 各测试方法共享的类级配置。

### Lines 45-57: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=cls.other_args,
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 59-61: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 63-88: Define helper: run fim completion
```python
    def run_fim_completion(self, number_of_completion):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        prompt = "function sum(a: number, b: number): number{\n"
        suffix = "}"

        prompt_input = self.tokenizer.encode(prompt) + self.tokenizer.encode(suffix)
        num_prompt_tokens = len(prompt_input) + 2

        response = client.completions.create(
            model=self.model,
            prompt=prompt,
            suffix=suffix,
            temperature=0.3,
            max_tokens=32,
            stream=False,
            n=number_of_completion,
        )
        assert len(response.choices) == number_of_completion
        assert response.id
        assert response.created
        assert response.object == "text_completion"
        assert (
            response.usage.prompt_tokens == num_prompt_tokens
        ), f"{response.usage.prompt_tokens} vs {num_prompt_tokens}"
        assert response.usage.completion_tokens > 0
        assert response.usage.total_tokens > 0
```
**EN:** This helper function encapsulates reusable logic inside `TestFimCompletion` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFimCompletion` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 90-92: Run test: fim completion
```python
    def test_fim_completion(self):
        for number_of_completion in [1, 3]:
            self.run_fim_completion(number_of_completion)
```
**EN:** This test method exercises fim completion and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fim completion 场景，并验证观测到的行为是否符合预期契约。

### Lines 95-95: Define class TestFimCompletionJson
```python
class TestFimCompletionJson(TestFimCompletion):
```
**EN:** This declaration introduces the `TestFimCompletionJson` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFimCompletionJson` 测试类，并说明它通过继承承担的职责。

### Lines 96-104: Declare TestFimCompletionJson configuration
```python
    other_args = [
        "--completion-template",
        DEEPSEEK_CODER_JSON_PATH,
        "--attention-backend",
        "ascend",
        "--disable-cuda-graph",
        "--mem-fraction-static",
        0.8,
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestFimCompletionJson` test methods.
**CN:** 该代码块定义了 `TestFimCompletionJson` 各测试方法共享的类级配置。

### Lines 107-108: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `openai`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
