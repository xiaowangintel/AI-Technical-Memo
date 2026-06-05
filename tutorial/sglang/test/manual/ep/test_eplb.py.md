# test_eplb.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/ep/test_eplb.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `eplb` scenario in `test/manual/ep`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/ep` 中的 `eplb` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and shared helpers / 导入与共享辅助项
```python
import tempfile
import unittest
from pathlib import Path
from types import SimpleNamespace

import sglang as sgl
from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range imports `tempfile`, `unittest`, `pathlib` and `types`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。

### Lines 19-22: Class definition for _BaseTestDynamicEPLB / 类定义
```python
class _BaseTestDynamicEPLB(CustomTestCase):
    extra_args = []

    @classmethod
```
**EN:** This range declares `_BaseTestDynamicEPLB`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 23-43: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEFAULT_MLA_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        with (
            envs.SGLANG_ENABLE_JIT_DEEPGEMM.override(False),
            envs.SGLANG_EXPERT_LOCATION_UPDATER_CANARY.override(True),
        ):
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
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `with`, `override` and `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 44-47: Scenario logic / 场景逻辑
```python
                    "normal",
                    "--disable-cuda-graph",
                    "--enable-eplb",
                    "--ep-num-redundant-experts",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 48-62: Scenario logic / 场景逻辑
```python
                    "4",
                    "--eplb-rebalance-num-iterations",
                    "50",
                    "--expert-distribution-recorder-buffer-size",
                    "50",
                    # TODO pr-chain: enable later
                    # "--enable-expert-distribution-metrics",
                    # TODO auto determine these flags
                    "--expert-distribution-recorder-mode",
                    "stat",
                    "--ep-dispatch-algorithm",
                    "static",
                    *cls.extra_args,
                ],
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 63-64: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 65-66: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 68-68: Test routines around test_mmlu / 测试例程
```python
    def test_mmlu(self):
```
**EN:** This range defines concrete test routine(s) `test_mmlu`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 69-78: Assertions and result checks / 断言与结果检查
```python
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=64,
            num_threads=32,
        )

        metrics = run_eval(args)
        self.assertGreater(metrics["score"], 0.5)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `SimpleNamespace`, `run_eval` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 81-82: Class definition for TestDynamicEPLBSimple / 类定义
```python
class TestDynamicEPLBSimple(_BaseTestDynamicEPLB):
    pass
```
**EN:** This range declares `TestDynamicEPLBSimple`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 85-86: Class definition for TestDynamicEPLBMultiChunk / 类定义
```python
class TestDynamicEPLBMultiChunk(_BaseTestDynamicEPLB):
    extra_args = ["--eplb-rebalance-layers-per-chunk", "1"]
```
**EN:** This range declares `TestDynamicEPLBMultiChunk`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 89-89: Class definition for TestStaticEPLB / 类定义
```python
class TestStaticEPLB(CustomTestCase):
```
**EN:** This range declares `TestStaticEPLB`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 90-113: Test routines around test_save_expert_distribution_and_init_expert_location / 测试例程
```python
    def test_save_expert_distribution_and_init_expert_location(self):
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        with tempfile.TemporaryDirectory() as tmp_dir:
            engine_kwargs = dict(
                model_path=DEFAULT_MLA_MODEL_NAME_FOR_TEST,
                trust_remote_code=True,
                ep_num_redundant_experts=4,
                enable_dp_attention=True,
                moe_a2a_backend="deepep",
                disable_cuda_graph=True,
                expert_distribution_recorder_mode="stat",
                tp_size=2,
                dp_size=2,
                log_level="info",
                # TODO pr-chain: enable later
                # enable_expert_distribution_metrics=True,
            )

            print(f"Action: start engine")
            envs.SGLANG_EXPERT_DISTRIBUTION_RECORDER_DIR.set(tmp_dir)
            engine = sgl.Engine(
                **engine_kwargs,
                disable_overlap_schedule=True,
```
**EN:** This range defines concrete test routine(s) `test_save_expert_distribution_and_init_expert_location`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `TemporaryDirectory` and `Engine`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 114-114: Scenario logic / 场景逻辑
```python
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 115-138: Assertions and result checks / 断言与结果检查
```python
            engine.start_expert_distribution_record()
            self._assert_engine_generate_correct(engine)

            print(f"Action: dump_expert_distribution_record")
            engine.dump_expert_distribution_record()
            snapshot_path = list(Path(tmp_dir).glob("*.pt"))[0]
            assert snapshot_path is not None
            print(f"{snapshot_path=}")

            print(f"Action: shutdown engine")
            engine.shutdown()
            del engine

            print(f"Action: start engine with init_expert_location")
            engine = sgl.Engine(
                **engine_kwargs,
                init_expert_location=str(snapshot_path),
                port=21000,
                # TODO auto determine these flags
                ep_dispatch_algorithm="static",
            )
            self._assert_engine_generate_correct(engine)
            print(f"Action: shutdown engine")
            engine.shutdown()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `start_expert_distribution_record`, `_assert_engine_generate_correct`, `dump_expert_distribution_record` and `Path`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 139-139: Scenario logic / 场景逻辑
```python
            del engine
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 141-150: Helper routines around _assert_engine_generate_correct / 辅助例程
```python
    def _assert_engine_generate_correct(self, engine: sgl.Engine):
        output = engine.generate(
            prompt=["1+1=2, 2+2=4", "One plus one is two, two plus two is four"],
            sampling_params=dict(max_new_tokens=8, temperature=0.0),
        )
        print(f"engine.generate {output=}")
        self.assertEqual(
            [x["text"] for x in output],
            [", 4+4=8,", ", four plus four is eight, eight"],
        )
```
**EN:** This range implements helper routine(s) `_assert_engine_generate_correct` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `generate` and `assertEqual`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 151-154: Script entry point / 脚本入口
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
- **Standard Library / 标准库**: `pathlib`, `tempfile`, `types`, `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
