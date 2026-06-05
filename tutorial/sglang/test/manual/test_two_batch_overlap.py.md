# test_two_batch_overlap.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_two_batch_overlap.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `two batch overlap` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual` 中的 `two batch overlap` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and shared helpers / 导入与共享辅助项
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.batch_overlap.two_batch_overlap import (
    compute_split_seq_index,
    compute_split_token_index,
)
from sglang.srt.environ import envs
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_ENABLE_THINKING_MODEL_NAME_FOR_TEST,
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    popen_launch_server,
)
```
**EN:** This range imports `unittest`, `types`, `requests` and `sglang.srt.batch_overlap.two_batch_overlap`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 23-24: Class definition for TestTwoBatchOverlap / 类定义
```python
class TestTwoBatchOverlap(unittest.TestCase):
    @classmethod
```
**EN:** This range declares `TestTwoBatchOverlap`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 25-47: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEFAULT_MLA_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        with envs.SGLANG_ENABLE_JIT_DEEPGEMM.override(False):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--trust-remote-code",
                    "--tp",
                    "2",
                    "--dp",
                    "2",
                    "--enable-dp-attention",
                    "--moe-a2a-backend",
                    "deepep",
                    "--deepep-mode",
                    "normal",
                    "--disable-cuda-graph",  # DeepEP normal does not support CUDA Graph
                    "--enable-two-batch-overlap",
                ],
            )
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `override` and `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

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

### Lines 53-63: Test routines around test_generate_single_prompt / 测试例程
```python
    def test_generate_single_prompt(self):
        response = requests.post(
            self.base_url + "/generate",
            # we use an uncommon start to minimise the chance that the cache is hit by chance
            json={
                "text": "_ 1+1=2, 1+2=3, 1+3=4, 1+4=",
                "sampling_params": {"temperature": 0, "max_new_tokens": 8},
            },
        )
        print(f"{response.json()=}")
        self.assertEqual(response.json()["text"], "5, 1+5=6")
```
**EN:** This range defines concrete test routine(s) `test_generate_single_prompt`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `post`, `json` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-72: Test routines around test_mmlu / 测试例程
```python
    def test_mmlu(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=64,
            num_threads=32,
        )
```
**EN:** This range defines concrete test routine(s) `test_mmlu`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Representative call sites include `SimpleNamespace`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 73-75: Assertions and result checks / 断言与结果检查
```python

        metrics = run_eval(args)
        self.assertGreater(metrics["score"], 0.5)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Assertions in this block enforce the intended outcome. Representative call sites include `run_eval` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 78-78: Class definition for TestTwoBatchOverlapUnitTest / 类定义
```python
class TestTwoBatchOverlapUnitTest(unittest.TestCase):
```
**EN:** This range declares `TestTwoBatchOverlapUnitTest`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 79-102: Test routines around test_compute_split_seq_and_token_index / 测试例程
```python
    def test_compute_split_seq_and_token_index(self):
        for num_tokens, expect in [
            (0, 0),
            (100, 50),
            (99, 49),
        ]:
            actual = compute_split_seq_index(
                forward_mode=ForwardMode.DECODE,
                num_tokens=num_tokens,
                extend_lens=None,
                token_num_per_seq=1,
            )
            self.assertEqual(actual, expect)

        for extend_lens, expect in [
            ([], (0, 0)),
            ([42], (0, 21)),
            ([42, 999], (1, 520)),
            ([999, 42], (0, 520)),
            ([498, 502], (1, 498)),
            ([4096, 4096, 4096, 4096], (2, 8192)),
            ([4095, 4096, 4096, 4096, 1], (2, 8191)),
            ([1, 4095, 4096, 4096, 4096], (3, 8192)),
            ([4097, 4096, 4096, 4095, 1], (2, 8193)),
```
**EN:** This range defines concrete test routine(s) `test_compute_split_seq_and_token_index`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `compute_split_seq_index` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 103-103: Scenario logic / 场景逻辑
```python
            ([1, 1, 1, 1, 99999], (4, 50001)),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 104-120: Assertions and result checks / 断言与结果检查
```python
            ([99999, 1, 1, 1, 1], (0, 50001)),
        ]:
            actual_seq_idx = compute_split_seq_index(
                forward_mode=ForwardMode.EXTEND,
                num_tokens=None,
                extend_lens=extend_lens,
                token_num_per_seq=None,
            )
            actual_token_idx = compute_split_token_index(
                split_seq_index=actual_seq_idx,
                forward_mode=ForwardMode.EXTEND,
                extend_seq_lens=extend_lens,
                token_num_per_seq=None,
            )
            actual = (actual_seq_idx, actual_token_idx)
            print(f"{extend_lens=} {expect=} {actual=}")
            self.assertEqual(actual, expect)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `compute_split_seq_index`, `compute_split_token_index` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 123-124: Class definition for TestQwen3TwoBatchOverlap / 类定义
```python
class TestQwen3TwoBatchOverlap(TestTwoBatchOverlap):
    @classmethod
```
**EN:** This range declares `TestQwen3TwoBatchOverlap`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 125-147: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEFAULT_ENABLE_THINKING_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-1234"
        with envs.SGLANG_ENABLE_JIT_DEEPGEMM.override(False):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--trust-remote-code",
                    "--tp",
                    "2",
                    "--dp",
                    "2",
                    "--enable-dp-attention",
                    "--moe-a2a-backend",
                    "deepep",
                    "--deepep-mode",
                    "normal",
                    "--disable-cuda-graph",  # DeepEP normal does not support CUDA Graph
                    "--enable-two-batch-overlap",
                ],
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `override` and `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 148-148: Scenario logic / 场景逻辑
```python
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 149-152: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Environment-aware configuration / 环境感知配置

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `types`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.batch_overlap.two_batch_overlap`, `sglang.srt.environ`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
