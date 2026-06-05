# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/cli/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `RaiseNotImplementedAction`, and `launch_distributed`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于服务入口层。它围绕 `RaiseNotImplementedAction` 和 `launch_distributed` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-11: module setup and imports / 模块初始化与导入
```python
import argparse
import os
import shlex
import subprocess
import sys

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `argparse`, `os`, `shlex`, `subprocess`, `sys`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `argparse`、`os`、`shlex`、`subprocess`、`sys` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 13-13: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 16-17: `RaiseNotImplementedAction` class overview / `RaiseNotImplementedAction` 类概览
```python
class RaiseNotImplementedAction(argparse.Action):
```
**EN:** This block defines class `RaiseNotImplementedAction`. It encapsulates raise not implemented action behavior. It inherits from `argparse.Action`.
**CN:** 该代码块定义了类 `RaiseNotImplementedAction`。 它用于封装 raise not implemented action 相关行为。 它继承自 `argparse.Action`。

### Lines 18-19: `__call__` implementation / `__call__` 实现
```python
    def __call__(self, parser, namespace, values, option_string=None):
        raise NotImplementedError(f"The {option_string} option is not yet implemented")
```
**EN:** This block defines method `__call__` on `RaiseNotImplementedAction`. It makes the object callable. Key calls include `NotImplementedError`. Parameters such as `parser`, `namespace`, `values`, and `option_string` drive the behavior in this section.
**CN:** 该代码块定义了 `RaiseNotImplementedAction` 的方法 `__call__`。 它用于让对象可以像函数一样被调用。 关键调用包括 `NotImplementedError`。 本段逻辑主要由 `parser`、`namespace`、`values` 和 `option_string` 等参数驱动。

### Lines 22-75: `launch_distributed` implementation / `launch_distributed` 实现
```python
def launch_distributed(
    num_gpus: int, args: list[str], master_port: int | None = None
) -> int:
    """
    Launch a distributed job with the given arguments

    Args:
        num_gpus: Number of GPUs to use
        args: Arguments to pass to v1_sgl_diffusion_inference.py (defaults to sys.argv[1:])
        master_port: Port for the master process (default: random)
    """

    current_env = os.environ.copy()
    python_executable = sys.executable
    project_root = os.path.abspath(
        os.path.join(os.path.dirname(__file__), "../../../..")
    )
    main_script = os.path.join(
        project_root, "sgl_diffusion/sample/v1_sgl_diffusion_inference.py"
    )

    cmd = [
        python_executable,
        "-m",
        "torch.distributed.run",
        f"--nproc_per_node={num_gpus}",
    ]

    if master_port is not None:
        cmd.append(f"--master_port={master_port}")

    cmd.append(main_script)
    cmd.extend(args)

    logger.info("Running inference with %d GPU(s)", num_gpus)
    logger.info("Launching command: %s", shlex.join(cmd))

    current_env["PYTHONIOENCODING"] = "utf-8"
    process = subprocess.Popen(
        cmd,
        env=current_env,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        universal_newlines=True,
        bufsize=1,
        encoding="utf-8",
        errors="replace",
    )

    if process.stdout:
        for line in iter(process.stdout.readline, ""):
            print(line.strip())

    return process.wait()
```
**EN:** This block defines function `launch_distributed`. Launch a distributed job with the given arguments Args: num_gpus: Number of GPUs to use args: Arguments to pass to v1_sgl_diffusion_inference.py (defaults to sys.argv[1:]) master_port: Port for the master process (default: random) Key calls include `os.environ.copy`, `os.path.abspath`, `os.path.join`, `cmd.append`, and `cmd.extend`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `num_gpus`, `args`, and `master_port` drive the behavior in this section.
**CN:** 该代码块定义了函数 `launch_distributed`。 它用于处理 launch distributed 相关逻辑。 关键调用包括 `os.environ.copy`、`os.path.abspath`、`os.path.join`、`cmd.append` 和 `cmd.extend`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `num_gpus`、`args` 和 `master_port` 等参数驱动。

## Key Concepts / 关键概念
- `RaiseNotImplementedAction`: Primary class that encapsulates raise not implemented action behavior. / 核心类，用于封装 raise not implemented action 相关行为。
- `launch_distributed`: Launch a distributed job with the given arguments Args: num_gpus: Number of GPUs to use args: Arguments to pass to v1_sgl_diffusion_inference.py (defaults to sys.argv[1:]) master_port: Port for the master process (default: random) / 顶层函数，用于处理 launch distributed 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `os`, `shlex`, `subprocess`, `sys`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 75
