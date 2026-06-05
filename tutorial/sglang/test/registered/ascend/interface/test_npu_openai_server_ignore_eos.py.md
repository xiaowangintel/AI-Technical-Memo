# test_npu_openai_server_ignore_eos.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/interface/test_npu_openai_server_ignore_eos.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on interface npu openai server ignore eos in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 interface npu openai server ignore eos 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ascend.test_ascend_utils import LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
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

### Lines 16-16: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-2-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 19-19: Define class TestOpenAIServerIgnoreEOS
```python
class TestOpenAIServerIgnoreEOS(CustomTestCase):
```
**EN:** This declaration introduces the `TestOpenAIServerIgnoreEOS` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOpenAIServerIgnoreEOS` 测试类，并说明它通过继承承担的职责。

### Lines 20-24: Document the class `TestOpenAIServerIgnoreEOS`
```python
    """Testcase: Test 'ignore_eos' is True, the EOS is ignored and continue reasoning

    [Test Category] Interface
    [Test Target] ignore_eos
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestOpenAIServerIgnoreEOS`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestOpenAIServerIgnoreEOS`的设计意图。

### Lines 26-44: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.other_args = [
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
        ]
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

### Lines 46-48: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 50-97: Run test: ignore eos
```python
    def test_ignore_eos(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        max_tokens = 200

        response_default = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": "Count from 1 to 20."},
            ],
            temperature=0,
            max_tokens=max_tokens,
            extra_body={"ignore_eos": False},
        )

        response_ignore_eos = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": "Count from 1 to 20."},
            ],
            temperature=0,
            max_tokens=max_tokens,
            extra_body={"ignore_eos": True},
        )

        default_tokens = len(
            self.tokenizer.encode(response_default.choices[0].message.content)
        )
        ignore_eos_tokens = len(
            self.tokenizer.encode(response_ignore_eos.choices[0].message.content)
        )

        # Check if ignore_eos resulted in more tokens or exactly max_tokens
        # The ignore_eos response should either:
        # 1. Have more tokens than the default response (if default stopped at EOS before max_tokens)
        # 2. Have exactly max_tokens (if it reached the max_tokens limit)
        self.assertTrue(
            ignore_eos_tokens > default_tokens or ignore_eos_tokens >= max_tokens,
            f"ignore_eos did not generate more tokens: {ignore_eos_tokens} vs {default_tokens}",
        )

        self.assertEqual(
            response_ignore_eos.choices[0].finish_reason,
            "length",
            f"Expected finish_reason='length' for ignore_eos=True, got {response_ignore_eos.choices[0].finish_reason}",
        )
```
**EN:** This test method exercises ignore eos and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ignore eos 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 100-101: Expose unittest entrypoint
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
