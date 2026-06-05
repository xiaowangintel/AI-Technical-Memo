# compile_deep_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/compile_deep_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Compile DeepGEMM Kernels for a model with specify server arguments This script launches a server for capturing DeepGEMM calls and then compiles the kernels. / 该文件围绕上述模块职责展开实现，组织核心流程、辅助逻辑以及与相邻组件的协作。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10: Module-level supporting statements
```python
"""
Compile DeepGEMM Kernels for a model with specify server arguments

This script launches a server for capturing DeepGEMM calls and then compiles the kernels.
It accepts server arguments (the same as launch_server.py).

Usage:
python3 -m sglang.compile_deep_gemm --model deepseek-ai/DeepSeek-V3 --tp 8 --trust-remote-code

"""
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python
import argparse
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
import dataclasses
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python
import multiprocessing
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-16: Module-level supporting statements
```python
import time
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 17-17: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-18: Module-level supporting statements
```python
import requests
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 19-19: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 20-20: Module-level supporting statements
```python
from sglang.srt.disaggregation.utils import FAKE_BOOTSTRAP_HOST
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 21-21: Module-level supporting statements
```python
from sglang.srt.entrypoints.http_server import launch_server
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 22-22: Module-level supporting statements
```python
from sglang.srt.entrypoints.warmup import warmup
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 23-23: Module-level supporting statements
```python
from sglang.srt.environ import envs
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 24-24: Module-level supporting statements
```python
from sglang.srt.managers.io_struct import GenerateReqInput
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 25-25: Module-level supporting statements
```python
from sglang.srt.managers.tokenizer_manager import TokenizerManager
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 26-26: Module-level supporting statements
```python
from sglang.srt.server_args import ServerArgs
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 27-27: Module-level supporting statements
```python
from sglang.srt.utils import kill_process_tree
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 28-28: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 29-29: Module-level supporting statements
```python
multiprocessing.set_start_method("spawn", force=True)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 30-31: Module-level supporting statements
```python

# Reduce warning
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 32-32: Module-level supporting statements
```python
envs.SGLANG_IN_DEEPGEMM_PRECOMPILE_STAGE.set(True)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 33-33: Module-level supporting statements
```python
# Force enable deep gemm
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 34-34: Module-level supporting statements
```python
envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(True)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 35-35: Module-level supporting statements
```python
# Force enable mha chunked kv for DeepSeek V3 to avoid missing kv_b_proj DeepGEMM case
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 36-36: Module-level supporting statements
```python
envs.SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD.set(0)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 37-39: Module-level supporting statements
```python


@dataclasses.dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 40-40: CompileArgs class declaration
```python
class CompileArgs:
```
**EN:** This block declares the `CompileArgs` class, which exists to compile kernels or build artifacts. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `CompileArgs` 类，其职责是编译内核或构建产物。它定义了本文件其余部分使用的结构与成员布局。

### Lines 41-41: Class-level supporting statements
```python
    timeout: int = 3600
```
**EN:** This block contains supporting statements for the `CompileArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`CompileArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 42-43: Class-level supporting statements
```python

    @staticmethod
```
**EN:** This block contains supporting statements for the `CompileArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`CompileArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 44-45: CompileArgs.add_cli_args method
```python
    def add_cli_args(parser: argparse.ArgumentParser):
        parser.add_argument("--timeout", type=int, default=CompileArgs.timeout)
```
**EN:** This block uses `CompileArgs.add_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CompileArgs.add_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 46-47: Class-level supporting statements
```python

    @classmethod
```
**EN:** This block contains supporting statements for the `CompileArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`CompileArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 48-53: CompileArgs.from_cli_args method
```python
    def from_cli_args(cls, args: argparse.Namespace):
        # use the default value's type to cast the args into correct types.
        attrs = [(attr.name, type(attr.default)) for attr in dataclasses.fields(cls)]
        return cls(
            **{attr: attr_type(getattr(args, attr)) for attr, attr_type in attrs}
        )
```
**EN:** This block uses `CompileArgs.from_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `CompileArgs.from_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 54-56: Module-level supporting statements
```python


@warmup("compile-deep-gemm")
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 57-89: warm up compile function
```python
async def warm_up_compile(
    disaggregation_mode: str, tokenizer_manager: TokenizerManager
):
    print("\nGenerate warm up request for compiling DeepGEMM...\n")
    server_args = tokenizer_manager.server_args
    dp_size = server_args.dp_size
    base_ids = [0, 1, 2, 3]
    sampling_params = {
        "temperature": 0.0,
        "max_new_tokens": 8,
        "ignore_eos": True,
    }

    if disaggregation_mode != "null":
        input_ids = [list(base_ids) for _ in range(dp_size)]
        generate_req_input = GenerateReqInput(
            input_ids=input_ids,
            sampling_params=sampling_params,
        )
        generate_req_input.bootstrap_host = [FAKE_BOOTSTRAP_HOST] * dp_size
        generate_req_input.bootstrap_room = [
            i * (2**63 // dp_size) + (i % server_args.tp_size) for i in range(dp_size)
        ]
    else:
        input_ids = (
            base_ids if dp_size == 1 else [list(base_ids) for _ in range(dp_size)]
        )
        generate_req_input = GenerateReqInput(
            input_ids=input_ids,
            sampling_params=sampling_params,
        )

    await tokenizer_manager.generate_request(generate_req_input, None).__anext__()
```
**EN:** This block uses `warm_up_compile` to compile kernels or build artifacts. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `warm_up_compile` 来编译内核或构建产物。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 90-91: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 92-98: launch server internal function
```python
def launch_server_internal(server_args):
    try:
        launch_server(server_args)
    except Exception as e:
        raise e
    finally:
        kill_process_tree(os.getpid(), include_parent=False)
```
**EN:** This block uses `launch_server_internal` to launch a service or execution flow. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `launch_server_internal` 来启动服务或执行流程。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 99-100: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 101-160: launch server process and send one request function (part 1/2)
```python
def launch_server_process_and_send_one_request(
    server_args: ServerArgs, compile_args: CompileArgs
):
    proc = multiprocessing.Process(target=launch_server_internal, args=(server_args,))
    proc.start()
    base_url = f"http://{server_args.host}:{server_args.port}"
    timeout = compile_args.timeout

    start_time = time.perf_counter()
    while time.perf_counter() - start_time < timeout:
        try:
            headers = {
                "Content-Type": "application/json; charset=utf-8",
            }
            if server_args.node_rank == 0:
                response = requests.get(f"{base_url}/v1/models", headers=headers)
            else:
                # This http api is created by launch_dummy_health_check_server for none-rank0 node.
                response = requests.get(f"{base_url}/health", headers=headers)
            if response.status_code == 200:
                # Rank-0 node send a request to sync with other node and then return.
                if server_args.node_rank == 0:
                    dp_size = server_args.dp_size
                    base_ids = [0, 1, 2, 3]
                    payload = {
                        "sampling_params": {
                            "max_new_tokens": 8,
                            "temperature": 0,
                        },
                    }
                    if server_args.disaggregation_mode != "null":
                        payload["input_ids"] = [list(base_ids) for _ in range(dp_size)]
                        payload["bootstrap_host"] = [FAKE_BOOTSTRAP_HOST] * dp_size
                        payload["bootstrap_room"] = [
                            i * (2**63 // dp_size) + (i % server_args.tp_size)
                            for i in range(dp_size)
                        ]
                    else:
                        payload["input_ids"] = (
                            base_ids
                            if dp_size == 1
                            else [list(base_ids) for _ in range(dp_size)]
                        )

                    response = requests.post(
                        f"{base_url}/generate",
                        json=payload,
                        timeout=600,
                    )
                    if response.status_code != 200:
                        error = response.json()
                        raise RuntimeError(f"Sync request failed: {error}")
                # Other nodes should wait for the exit signal from Rank-0 node.
                else:
                    start_time_waiting = time.perf_counter()
                    while proc.is_alive():
                        if time.perf_counter() - start_time_waiting < timeout:
                            time.sleep(10)
                        else:
                            raise TimeoutError("Waiting for main node timeout!")
```
**EN:** This block uses `launch_server_process_and_send_one_request` to launch a service or execution flow. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `launch_server_process_and_send_one_request` 来启动服务或执行流程。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 161-168: launch server process and send one request function (part 2/2)
```python
                return proc
        except requests.RequestException:
            pass
        time.sleep(10)
    raise TimeoutError(
        "DeepGEMM Kernels compilation timeout."
        "\n\nFeel free and please restart the command."
    )
```
**EN:** This block uses `launch_server_process_and_send_one_request` to launch a service or execution flow. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `launch_server_process_and_send_one_request` 来启动服务或执行流程。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 169-170: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 171-179: refine server args function
```python
def refine_server_args(server_args: ServerArgs, compile_args: CompileArgs):
    # Disable cuda graph and torch compile to save time
    server_args.disable_cuda_graph = True
    server_args.enable_torch_compile = False
    print(f"Disable CUDA Graph and Torch Compile to save time...")

    # Set watchdog timeout to compile_args.timeout because compilation will take a long time
    server_args.watchdog_timeout = compile_args.timeout
    server_args.warmups = "compile-deep-gemm"
```
**EN:** This block uses `refine_server_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `refine_server_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 180-181: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 182-204: run compile function
```python
def run_compile(server_args: ServerArgs, compile_args: CompileArgs):
    print(
        "Begin DeepGEMM Kernels compilation...\n"
        "It may take a long time and timeout maybe raised "
        "while the compilation is still in progress.\n"
        "Just feel free to restart the command "
        "until the compilation is fully finished.\n"
    )

    proc = launch_server_process_and_send_one_request(server_args, compile_args)

    print("\nDeepGEMM Kernels compilation finished successfully.")

    # Sleep for safety
    time.sleep(10)
    if proc.is_alive():
        # This is the rank0 node.
        kill_process_tree(proc.pid)
    else:
        try:
            kill_process_tree(proc.pid)
        except Exception:
            pass
```
**EN:** This block uses `run_compile` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `run_compile` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 205-206: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 207-217: Module-level supporting statements
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    ServerArgs.add_cli_args(parser)
    CompileArgs.add_cli_args(parser)
    args = parser.parse_args()
    server_args = ServerArgs.from_cli_args(args)
    compile_args = CompileArgs.from_cli_args(args)

    refine_server_args(server_args, compile_args)

    run_compile(server_args, compile_args)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- External API integration / 外部 API 集成
- Runtime coordination / 运行时协同

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.disaggregation.utils`
- `sglang.srt.entrypoints.http_server`
- `sglang.srt.entrypoints.warmup`
- `sglang.srt.environ`
- `sglang.srt.managers.io_struct`
- `sglang.srt.managers.tokenizer_manager`
- `sglang.srt.server_args`
- `sglang.srt.utils`
### External / 外部
- `multiprocessing`
- `requests`
- `argparse` (stdlib)
- `dataclasses` (stdlib)
- `os` (stdlib)
- `time` (stdlib)
