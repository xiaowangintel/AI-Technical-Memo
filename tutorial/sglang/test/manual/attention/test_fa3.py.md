# test_fa3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/attention/test_fa3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `fa3` scenario in `test/manual/attention`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/attention` 中的 `fa3` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-59: Constants and scenario settings / 常量与场景配置
```python
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import get_device_sm, kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE3,
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_MODEL_NAME_FOR_TEST_MLA,
    DEFAULT_MODEL_NAME_FOR_TEST_MLA_NEXTN,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

# FlashAttention3 integration tests (requires SM 90+ / H100)
# Multiple test classes: FA3, FA3+MLA, FA3+SpecDecode variants
GSM_DATASET_PATH = None

# In case of some machine lack internet connection, we can set OFFLINE_MODE to True.
OFFLINE_MODE = False

# Change the path below when OFFLINE_MODE is True.
OFFLINE_PATH_DICT = {
    DEFAULT_MODEL_NAME_FOR_TEST: "/shared/public/elr-models/meta-llama/Meta-Llama-3.1-8B-Instruct",
    DEFAULT_DRAFT_MODEL_EAGLE3: "/shared/public/elr-models/jamesliu1/sglang-EAGLE3-Llama-3.1-Instruct-8B",
    DEFAULT_MODEL_NAME_FOR_TEST_MLA: "/shared/public/sharing/deepseek/dsv3-test/snapshots/",
    DEFAULT_MODEL_NAME_FOR_TEST_MLA_NEXTN: "/shared/public/sharing/deepseek/dsv3-test-NextN/snapshots/",
    GSM_DATASET_PATH: "/shared/public/data/gsm8k/test.jsonl",
}

if OFFLINE_MODE:
    DEFAULT_MODEL_NAME_FOR_TEST = OFFLINE_PATH_DICT[DEFAULT_MODEL_NAME_FOR_TEST]
    DEFAULT_DRAFT_MODEL_EAGLE3 = OFFLINE_PATH_DICT[DEFAULT_DRAFT_MODEL_EAGLE3]
    DEFAULT_MODEL_NAME_FOR_TEST_MLA = OFFLINE_PATH_DICT[DEFAULT_MODEL_NAME_FOR_TEST_MLA]
    DEFAULT_MODEL_NAME_FOR_TEST_MLA_NEXTN = OFFLINE_PATH_DICT[
        DEFAULT_MODEL_NAME_FOR_TEST_MLA_NEXTN
    ]
    GSM_DATASET_PATH = OFFLINE_PATH_DICT[GSM_DATASET_PATH]

# Default server arguments shared across all tests
DEFAULT_SERVER_ARGS = [
    "--trust-remote-code",
    "--cuda-graph-max-bs",
    "8",
    "--attention-backend",
    "fa3",
]

"""
Integration test for python/sglang/srt/layers/attention/flashattention_backend.py
"""


@unittest.skipIf(get_device_sm() < 90, "Test requires CUDA SM 90 or higher")
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `tests`, `skipIf` and `get_device_sm`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 60-69: Class definition for BaseFlashAttentionTest / 类定义
```python
class BaseFlashAttentionTest(CustomTestCase):
    """Base class for testing FlashAttention3."""

    model = DEFAULT_MODEL_NAME_FOR_TEST
    base_url = DEFAULT_URL_FOR_TEST
    accuracy_threshold = 0.65  # derived tests need to override this
    speculative_decode = False
    spec_decode_threshold = 1.0  # derived spec decoding tests need to override this

    @classmethod
```
**EN:** This range declares `BaseFlashAttentionTest`, which organizes the scenario as a reusable test-oriented class. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 70-72: Helper routines around get_server_args / 辅助例程
```python
    def get_server_args(cls):
        """Return the arguments for the server launch. Override in subclasses."""
        return DEFAULT_SERVER_ARGS
```
**EN:** This range implements helper routine(s) `get_server_args` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 73-74: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 75-84: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        # disable deep gemm precompile to make launch server faster
        # please don't do this if you want to make your inference workload faster
        with (
            envs.SGLANG_JIT_DEEPGEMM_PRECOMPILE.override(False),
            envs.SGLANG_ENABLE_JIT_DEEPGEMM.override(False),
        ):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `with`, `override` and `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 85-87: Scenario logic / 场景逻辑
```python
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=cls.get_server_args(),
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_server_args`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 88-89: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 90-91: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 93-109: Test routines around test_gsm8k / 测试例程
```python
    def test_gsm8k(self):
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=100,
            num_threads=128,
            num_shots=4,
            gsm8k_data_path=GSM_DATASET_PATH,
        )
        metrics = run_eval(args)
        print(f"{metrics=}")
```
**EN:** This range defines concrete test routine(s) `test_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get`, `SimpleNamespace` and `run_eval`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 110-117: Assertions and result checks / 断言与结果检查
```python
        # Use the appropriate metric key based on the test class
        metric_key = "score"
        self.assertGreater(metrics[metric_key], self.accuracy_threshold)

        if self.speculative_decode:
            server_info = requests.get(self.base_url + "/server_info").json()
            avg_spec_accept_length = server_info["internal_states"][0][
                "avg_spec_accept_length"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`, `get` and `json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 118-120: Assertions and result checks / 断言与结果检查
```python
            ]
            print(f"{avg_spec_accept_length=}")
            self.assertGreater(avg_spec_accept_length, self.spec_decode_threshold)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 123-129: Class definition for TestFlashAttention3MLA / 类定义
```python
class TestFlashAttention3MLA(BaseFlashAttentionTest):
    """Test FlashAttention3 with MLA, e.g. deepseek v3 test model"""

    accuracy_threshold = 0.60
    model = DEFAULT_MODEL_NAME_FOR_TEST_MLA

    @classmethod
```
**EN:** This range declares `TestFlashAttention3MLA`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 130-131: Helper routines around get_server_args / 辅助例程
```python
    def get_server_args(cls):
        return DEFAULT_SERVER_ARGS
```
**EN:** This range implements helper routine(s) `get_server_args` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 134-142: Class definition for TestFlashAttention3SpeculativeDecode / 类定义
```python
class TestFlashAttention3SpeculativeDecode(BaseFlashAttentionTest):
    """Test FlashAttention3 with speculative decode enabled with Llama 3.1 8B and its eagle3 model"""

    model = DEFAULT_MODEL_NAME_FOR_TEST
    accuracy_threshold = 0.65
    speculative_decode = True
    spec_decode_threshold = 1.5

    @classmethod
```
**EN:** This range declares `TestFlashAttention3SpeculativeDecode`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 143-158: Helper routines around get_server_args / 辅助例程
```python
    def get_server_args(cls):
        args = DEFAULT_SERVER_ARGS
        args.extend(
            [
                "--cuda-graph-max-bs",
                "4",
                "--speculative-algorithm",
                "EAGLE3",
                "--speculative-draft-model-path",
                DEFAULT_DRAFT_MODEL_EAGLE3,
                "--speculative-num-steps",
                "3",
                "--speculative-eagle-topk",
                "1",
                "--speculative-num-draft-tokens",
                "4",
```
**EN:** This range implements helper routine(s) `get_server_args` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `extend`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 159-163: Scenario logic / 场景逻辑
```python
                "--dtype",
                "float16",
            ]
        )
        return args
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 166-176: Class definition for TestFlashAttention3SpeculativeDecodeTopk / 类定义
```python
class TestFlashAttention3SpeculativeDecodeTopk(BaseFlashAttentionTest):
    """Tests FlashAttention3 with enhanced speculative decoding using Llama 3.1 8B and EAGLE3.
    This test will be using top-k value > 1 which would verify the other branches of the FA3 code
    """

    model = DEFAULT_MODEL_NAME_FOR_TEST
    accuracy_threshold = 0.65
    speculative_decode = True
    spec_decode_threshold = 1.6

    @classmethod
```
**EN:** This range declares `TestFlashAttention3SpeculativeDecodeTopk`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 177-190: Helper routines around get_server_args / 辅助例程
```python
    def get_server_args(cls):
        args = DEFAULT_SERVER_ARGS
        args.extend(
            [
                "--cuda-graph-max-bs",
                "4",
                "--speculative-algorithm",
                "EAGLE3",
                "--speculative-draft-model-path",
                DEFAULT_DRAFT_MODEL_EAGLE3,
                "--speculative-num-steps",
                "5",
                "--speculative-eagle-topk",
                "4",
```
**EN:** This range implements helper routine(s) `get_server_args` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `extend`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 191-197: Scenario logic / 场景逻辑
```python
                "--speculative-num-draft-tokens",
                "8",
                "--dtype",
                "float16",
            ]
        )
        return args
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 200-208: Class definition for TestFlashAttention3MLASpeculativeDecode / 类定义
```python
class TestFlashAttention3MLASpeculativeDecode(BaseFlashAttentionTest):
    """Test FlashAttention3 with speculative decode enabled with deepseek v3 test model and its nextN model"""

    model = DEFAULT_MODEL_NAME_FOR_TEST_MLA
    accuracy_threshold = 0.60
    speculative_decode = True
    spec_decode_threshold = 2.5

    @classmethod
```
**EN:** This range declares `TestFlashAttention3MLASpeculativeDecode`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 209-224: Helper routines around get_server_args / 辅助例程
```python
    def get_server_args(cls):
        args = DEFAULT_SERVER_ARGS
        args.extend(
            [
                "--cuda-graph-max-bs",
                "4",
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-draft-model-path",
                DEFAULT_MODEL_NAME_FOR_TEST_MLA_NEXTN,
                "--speculative-num-steps",
                "3",
                "--speculative-eagle-topk",
                "1",
                "--speculative-num-draft-tokens",
                "4",
```
**EN:** This range implements helper routine(s) `get_server_args` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `extend`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 225-227: Scenario logic / 场景逻辑
```python
            ]
        )
        return args
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 230-240: Class definition for TestFlashAttention3MLASpeculativeDecodeTopk / 类定义
```python
class TestFlashAttention3MLASpeculativeDecodeTopk(BaseFlashAttentionTest):
    """Test FlashAttention3 with speculative decode enabled with deepseek v3 test model and its nextN model
    This test will be using top-k value > 1 which would verify the other branches of the FA3 code
    """

    model = DEFAULT_MODEL_NAME_FOR_TEST_MLA
    accuracy_threshold = 0.60
    speculative_decode = True
    spec_decode_threshold = 2.95

    @classmethod
```
**EN:** This range declares `TestFlashAttention3MLASpeculativeDecodeTopk`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 241-254: Helper routines around get_server_args / 辅助例程
```python
    def get_server_args(cls):
        args = DEFAULT_SERVER_ARGS
        args.extend(
            [
                "--cuda-graph-max-bs",
                "4",
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-draft-model-path",
                DEFAULT_MODEL_NAME_FOR_TEST_MLA_NEXTN,
                "--speculative-num-steps",
                "5",
                "--speculative-eagle-topk",
                "4",
```
**EN:** This range implements helper routine(s) `get_server_args` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `extend`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 255-259: Scenario logic / 场景逻辑
```python
                "--speculative-num-draft-tokens",
                "8",
            ]
        )
        return args
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 260-263: Script entry point / 脚本入口
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
- **Project Modules / 项目模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
