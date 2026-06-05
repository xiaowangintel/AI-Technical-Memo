# test_vlms_piecewise_cuda_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/nightly/test_vlms_piecewise_cuda_graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `vlms piecewise cuda graph` scenario in `test/manual/nightly`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/nightly` 中的 `vlms piecewise cuda graph` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports and shared helpers / 导入与共享辅助项
```python
import argparse
import glob
import json
import os
import random
import sys
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.kits.mmmu_vlm_kit import _run_lmms_eval_with_retry
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)

MODELS = [
    SimpleNamespace(model="Qwen/Qwen2.5-VL-7B-Instruct", mmmu_accuracy=0.60),
]


# Set default mem_fraction_static to 0.8
DEFAULT_MEM_FRACTION_STATIC = 0.8
```
**EN:** This range imports `argparse`, `glob`, `json` and `os`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `SimpleNamespace`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 29-32: Class definition for TestVLMPiecewiseCudaGraph / 类定义
```python
class TestVLMPiecewiseCudaGraph(CustomTestCase):
    parsed_args = None  # Class variable to store args

    @classmethod
```
**EN:** This range declares `TestVLMPiecewiseCudaGraph`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 33-46: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        # Removed argument parsing from here
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.time_out = DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH

        if cls.parsed_args is None:
            cls.parsed_args = SimpleNamespace(
                mem_fraction_static=DEFAULT_MEM_FRACTION_STATIC
            )

        # Set OpenAI API key and base URL environment variables. Needed for lmm-evals to work.
        os.environ["OPENAI_API_KEY"] = cls.api_key
        os.environ["OPENAI_API_BASE"] = f"{cls.base_url}/v1"
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `SimpleNamespace`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 48-53: Helper routines around run_mmmu_eval / 辅助例程
```python
    def run_mmmu_eval(
        self,
        model_version: str,
        output_path: str,
        *,
        env: dict | None = None,
```
**EN:** This range implements helper routine(s) `run_mmmu_eval` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 54-72: Request and response handling / 请求与响应处理
```python
    ):
        """
        Evaluate a VLM on the MMMU validation set with lmms‑eval.
        Only `model_version` (checkpoint) and `chat_template` vary;
        We are focusing only on the validation set due to resource constraints.
        """
        # -------- fixed settings --------
        model = "openai_compatible"
        tp = 1
        tasks = "mmmu_val"
        batch_size = 32
        log_suffix = "openai_compatible"
        os.makedirs(output_path, exist_ok=True)

        # -------- compose --model_args --------
        model_args = f'model_version="{model_version}",' f"tp={tp}"

        # -------- build command list --------
        cmd = [
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Representative call sites include `makedirs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 73-78: Scenario logic / 场景逻辑
```python
            "python3",
            "-m",
            "lmms_eval",
            "--model",
            model,
            "--model_args",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 79-88: Scenario logic / 场景逻辑
```python
            model_args,
            "--tasks",
            tasks,
            "--batch_size",
            str(batch_size),
            "--output_path",
            str(output_path),
        ]

        _run_lmms_eval_with_retry(cmd, timeout=3600)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_run_lmms_eval_with_retry`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 90-103: Helper routines around _run_vlm_mmmu_test / 辅助例程
```python
    def _run_vlm_mmmu_test(
        self,
        model,
        output_path,
        test_name="",
        custom_env=None,
        log_level="info",
        capture_output=False,
    ):
        """
        Common method to run VLM MMMU benchmark test.
        Args:
            model: Model to test
            output_path: Path for output logs
```
**EN:** This range implements helper routine(s) `_run_vlm_mmmu_test` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 104-114: Environment-driven configuration / 环境驱动配置
```python
            test_name: Optional test name for logging
            custom_env: Optional custom environment variables
            log_level: Log level for server (default: "info")
            capture_output: Whether to capture server stdout/stderr
        """
        print(f"\nTesting model: {model.model}{test_name}")

        process = None
        mmmu_accuracy = 0  # Initialize to handle potential exceptions
        server_output = ""
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 115-128: Environment-driven configuration / 环境驱动配置
```python
        try:
            # Prepare environment variables
            process_env = os.environ.copy()
            if custom_env:
                process_env.update(custom_env)
            # if test vlm with cuda_ipc feature, open this env_var
            process_env["SGLANG_USE_CUDA_IPC_TRANSPORT"] = "1"

            # Prepare stdout/stderr redirection if needed
            stdout_file = None
            stderr_file = None
            if capture_output:
                stdout_file = open("/tmp/server_stdout.log", "w")
                stderr_file = open("/tmp/server_stderr.log", "w")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `copy` and `update`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 129-139: Process control logic / 进程控制逻辑
```python

            # Launch server for testing
            process = popen_launch_server(
                model.model,
                base_url=self.base_url,
                timeout=self.time_out,
                api_key=self.api_key,
                other_args=[
                    "--trust-remote-code",
                    "--piecewise-cuda-graph-max-tokens",
                    "8192",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 140-153: Scenario logic / 场景逻辑
```python
                    "--enforce-piecewise-cuda-graph",
                    "--tp=8",
                    "--piecewise-cuda-graph-compiler=eager",
                    "--disable-radix-cache",
                    "--log-level",
                    log_level,
                ],
                env=process_env,
                return_stdout_stderr=(
                    (stdout_file, stderr_file) if capture_output else None
                ),
            )

            # Run evaluation
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 154-164: Scenario logic / 场景逻辑
```python
            self.run_mmmu_eval(model.model, output_path)

            # Get the result file
            # Search recursively for JSON result files (lmms-eval v0.4.1+ creates subdirectories)
            result_files = glob.glob(f"{output_path}/**/*.json", recursive=True)
            if not result_files:
                result_files = glob.glob(f"{output_path}/*.json")

            if not result_files:
                raise FileNotFoundError(f"No JSON result files found in {output_path}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `run_mmmu_eval`, `files`, `glob` and `FileNotFoundError`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 165-178: Scenario logic / 场景逻辑
```python
            result_file_path = result_files[0]

            with open(result_file_path, "r") as f:
                result = json.load(f)
                print(f"Result{test_name}\n: {result}")

            # Process the result
            mmmu_accuracy = result["results"]["mmmu_val"]["mmmu_acc,none"]
            print(
                f"Model {model.model} achieved accuracy{test_name}: {mmmu_accuracy:.4f}"
            )

            # Capture server output if requested
            if capture_output and process:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `load`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 179-189: Assertions and result checks / 断言与结果检查
```python
                server_output = self._read_output_from_files()

            # Assert performance meets expected threshold
            self.assertGreaterEqual(
                mmmu_accuracy,
                model.mmmu_accuracy,
                f"Model {model.model} accuracy ({mmmu_accuracy:.4f}) below expected threshold ({model.mmmu_accuracy:.4f}){test_name}",
            )

            return server_output
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_read_output_from_files`, `assertGreaterEqual`, `accuracy` and `threshold`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 190-203: Process control logic / 进程控制逻辑
```python
        except Exception as e:
            print(f"Error testing {model.model}{test_name}: {e}")
            self.fail(f"Test failed for {model.model}{test_name}: {e}")

        finally:
            # Ensure process cleanup happens regardless of success/failure
            if process is not None and process.poll() is None:
                print(f"Cleaning up process {process.pid}")
                try:
                    kill_process_tree(process.pid)
                except Exception as e:
                    print(f"Error killing process: {e}")

            # clean up temporary files
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `fail`, `poll` and `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 204-214: Scenario logic / 场景逻辑
```python
            if capture_output:
                if stdout_file:
                    stdout_file.close()
                if stderr_file:
                    stderr_file.close()
                for filename in ["/tmp/server_stdout.log", "/tmp/server_stderr.log"]:
                    try:
                        if os.path.exists(filename):
                            os.remove(filename)
                    except Exception as e:
                        print(f"Error removing {filename}: {e}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `close`, `exists` and `remove`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 216-228: Helper routines around _read_output_from_files / 辅助例程
```python
    def _read_output_from_files(self):
        output_lines = []

        log_files = [
            ("/tmp/server_stdout.log", "[STDOUT]"),
            ("/tmp/server_stderr.log", "[STDERR]"),
        ]
        for filename, tag in log_files:
            try:
                if os.path.exists(filename):
                    with open(filename, "r") as f:
                        for line in f:
                            output_lines.append(f"{tag} {line.rstrip()}")
```
**EN:** This range implements helper routine(s) `_read_output_from_files` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `exists`, `append` and `rstrip`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 229-232: Scenario logic / 场景逻辑
```python
            except Exception as e:
                print(f"Error reading {tag.lower()} file: {e}")

        return "\n".join(output_lines)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `lower` and `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 234-242: Test routines around test_vlm_mmmu_benchmark / 测试例程
```python
    def test_vlm_mmmu_benchmark(self):
        """Test VLM models against MMMU benchmark."""
        models_to_test = MODELS

        if is_in_ci():
            models_to_test = [random.choice(MODELS)]

        for model in models_to_test:
            self._run_vlm_mmmu_test(model, "./logs")
```
**EN:** This range defines concrete test routine(s) `test_vlm_mmmu_benchmark`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. Representative call sites include `is_in_ci`, `choice` and `_run_vlm_mmmu_test`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 243-262: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    # Define and parse arguments here, before unittest.main
    parser = argparse.ArgumentParser(description="Test VLM models")
    parser.add_argument(
        "--mem-fraction-static",
        type=float,
        help="Static memory fraction for the model",
        default=DEFAULT_MEM_FRACTION_STATIC,
    )

    # Parse args intended for unittest
    args = parser.parse_args()

    # Store the parsed args object on the class
    TestVLMPiecewiseCudaGraph.parsed_args = args

    # Pass args to unittest
    unittest.main(argv=[sys.argv[0]])
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `ArgumentParser`, `add_argument`, `parse_args` and `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `argparse`, `glob`, `json`, `os`, `random`, `sys`, `types`, `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.kits.mmmu_vlm_kit`, `sglang.test.test_utils`
