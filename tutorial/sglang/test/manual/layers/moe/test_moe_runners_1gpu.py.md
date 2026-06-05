# test_moe_runners_1gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/layers/moe/test_moe_runners_1gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `moe runners 1gpu` scenario in `test/manual/layers/moe`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/layers/moe` 中的 `moe runners 1gpu` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Process control logic / 进程控制逻辑
```python
import os
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST_FP8_WITH_MOE,
    DEFAULT_MODEL_NAME_FOR_TEST_MOE_NVFP4,
    DEFAULT_MODEL_NAME_FOR_TEST_MXFP4_WITH_MOE,
    DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 18-42: Class definition for TestMoERunner / 类定义
```python
class TestMoERunner(CustomTestCase):
    BASE_URL = DEFAULT_URL_FOR_TEST
    TIMEOUT = 6000
    DEFAULT_EVAL_KWARGS = {
        "eval_name": "mmlu",
        "num_examples": 5,
        "num_threads": 1,
    }

    CONFIGS = {
        "moe_runner_auto": {
            "model": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "triton",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_triton": {
            "model": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
            "other_args": [
```
**EN:** This range declares `TestMoERunner`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 43-67: Scenario logic / 场景逻辑
```python
                "--trust-remote-code",
                "--moe-runner-backend",
                "triton",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_triton_kernel": {
            "model": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "triton_kernel",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_flashinfer_cutlass": {
            "model": DEFAULT_MODEL_NAME_FOR_TEST_MOE_NVFP4,  # requires model with modelopt_fp4 quantization
            "other_args": [
                "--trust-remote-code",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 68-92: Scenario logic / 场景逻辑
```python
                "--moe-runner-backend",
                "flashinfer_cutlass",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_deep_gemm": {
            "model": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "deep_gemm",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_flashinfer_trtllm": {
            "model": DEFAULT_MODEL_NAME_FOR_TEST_FP8_WITH_MOE,  # modelopt_fp4 or fp8 quantization is required for Flashinfer trtllm MOE
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 93-117: Scenario logic / 场景逻辑
```python
                "flashinfer_trtllm",
            ],
        },
        "moe_runner_flashinfer_mxfp4": {
            "model": DEFAULT_MODEL_NAME_FOR_TEST_MXFP4_WITH_MOE,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "flashinfer_mxfp4",
                "--quantization",
                "mxfp4",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_flashinfer_cutedsl": {
            "model": DEFAULT_MODEL_NAME_FOR_TEST_MOE_NVFP4,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "flashinfer_cutedsl",
                "--attention-backend",
                "torch_native",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 118-142: Scenario logic / 场景逻辑
```python
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_cutlass": {
            "model": DEFAULT_MODEL_NAME_FOR_TEST_MOE_NVFP4,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "cutlass",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
            ],
        },
        "moe_runner_cutlass_fp8": {
            "model": DEFAULT_MODEL_NAME_FOR_TEST_FP8_WITH_MOE,
            "timeout": 3600,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "cutlass",
                "--attention-backend",
                "triton",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 143-167: Scenario logic / 场景逻辑
```python
                "--sampling-backend",
                "pytorch",
                "--disable-cuda-graph",
            ],
        },
        "moe_runner_speculative": {
            "model": DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
            "other_args": [
                "--trust-remote-code",
                "--moe-runner-backend",
                "triton",
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-draft-model-path",
                DEFAULT_SMALL_MOE_MODEL_NAME_FOR_TEST_CHAT,
                "--speculative-moe-runner-backend",
                "triton",
                "--speculative-num-steps",
                "2",
                "--speculative-num-draft-tokens",
                "4",
                "--attention-backend",
                "torch_native",
                "--sampling-backend",
                "pytorch",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 168-171: Scenario logic / 场景逻辑
```python
            ],
        },
    }
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 172-192: Helper routines around _run_config / 辅助例程
```python
    def _run_config(self, config: dict) -> None:
        model = config["model"]
        other_args = config.get("other_args", [])
        eval_kwargs = self.DEFAULT_EVAL_KWARGS
        env = dict(os.environ)
        env["SGLANG_ENABLE_JIT_DEEPGEMM"] = "1"
        env["SGLANG_JIT_DEEPGEMM_PRECOMPILE"] = "0"
        env.update(config.get("env_overrides", {}))
        timeout = config.get("timeout", self.TIMEOUT)

        process = popen_launch_server(
            model,
            self.BASE_URL,
            timeout=timeout,
            other_args=other_args,
            env=env,
        )
        try:
            args = SimpleNamespace(
                base_url=self.BASE_URL,
                model=model,
```
**EN:** This range implements helper routine(s) `_run_config` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `get`, `update`, `popen_launch_server` and `SimpleNamespace`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 193-196: Scenario logic / 场景逻辑
```python
                **eval_kwargs,
            )
            metrics = run_eval(args)
            print(f"{metrics=}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Representative call sites include `run_eval`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 197-199: Assertions and result checks / 断言与结果检查
```python
            self.assertGreaterEqual(metrics["score"], 0.48)
        finally:
            kill_process_tree(process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreaterEqual` and `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 200-211: Script entry point / 脚本入口
```python


for _name, _cfg in TestMoERunner.CONFIGS.items():
    setattr(
        TestMoERunner,
        f"test_{_name}",
        (lambda self, cfg=_cfg: self._run_config(cfg)),
    )


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `items`, `setattr`, `_run_config` and `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `types`, `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`
