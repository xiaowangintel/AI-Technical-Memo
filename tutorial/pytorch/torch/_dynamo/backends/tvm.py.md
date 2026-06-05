# tvm.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/backends/tvm.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Connects captured graphs to compiler backends and post-processing pipelines.
- **Purpose (CN)**: 将捕获到的图连接到编译后端及后处理流水线。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
"""
This module provides TVM backend integration for TorchDynamo.

Apache TVM is a deep learning compiler framework that can optimize and execute
models on various hardware backends. This module enables:

- Compilation of PyTorch models to TVM's computation graphs
- Multiple scheduling options:
  - Default scheduler
  - Auto-scheduler for automatic optimization
  - Meta-schedule for evolutionary search-based tuning
- Hardware-specific optimizations:
  - CUDA GPU support
  - CPU support with LLVM targeting and architecture-specific tuning
  - Automatic detection of CPU capabilities (AVX2, AVX512)
- Tensor conversion utilities between PyTorch and TVM formats
- Configurable optimization levels and tuning trials
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 19-36
```python
The backend can be used with torch.compile():
    model = torch.compile(model, backend="tvm")
"""

import functools
import importlib
import logging
import os
import sys
import tempfile
from collections.abc import Callable
from pathlib import Path
from types import MappingProxyType
from typing import Any

import torch
from torch import fx
```
- **EN**: This module-level block helps proxy tensor-like values through symbolic execution helpers.
- **CN**: 这个模块级代码块用于借助符号执行辅助逻辑代理类张量值。

### Lines 37-48
```python
from .common import device_from_inputs, fake_tensor_unsupported
from .registry import register_backend


log = logging.getLogger(__name__)


@register_backend
@fake_tensor_unsupported  # type: ignore[arg-type]
def tvm(
    gm: fx.GraphModule,
    example_inputs: list[torch.Tensor],
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 49-60
```python
    *,
    options: MappingProxyType[str, Any] | None = None,
) -> Callable[..., Any]:
    if options is None:
        options = MappingProxyType({"scheduler": None, "trials": 20000, "opt_level": 3})
    assert options is not None
    import tvm  # type: ignore[import]
    from tvm import relay  # type: ignore[import]
    from tvm.contrib import graph_executor  # type: ignore[import]

    jit_mod = torch.jit.trace(gm, example_inputs)
    device = device_from_inputs(example_inputs)
```
- **EN**: This block continues `tvm` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `tvm`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 61-77
```python
    shape_list = [(f"inp_{idx}", i.shape) for idx, i in enumerate(example_inputs)]
    example_outputs = gm(*example_inputs)
    if len(example_outputs) == 0:
        log.warning("Explicitly fall back to eager due to zero output")
        return gm.forward
    mod, params = relay.frontend.from_pytorch(jit_mod, shape_list)
    if device.type == "cuda":
        dev = tvm.cuda(device.index)
        target = tvm.target.cuda()
    else:
        dev = tvm.cpu(0)
        target = tvm.target.Target(llvm_target())

    scheduler = options.get("scheduler", None)
    if scheduler is None:
        scheduler = os.environ.get("TVM_SCHEDULER", None)
```
- **EN**: This block continues `tvm` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tvm`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 78-89
```python
    trials = options.get("trials", 20000)
    opt_level = options.get("opt_level", 3)

    if scheduler == "auto_scheduler":
        # pyrefly: ignore [missing-import]
        from tvm import auto_scheduler

        with (
            tempfile.NamedTemporaryFile() as log_file,
            auto_scheduler.ApplyHistoryBest(log_file),
            tvm.transform.PassContext(
                opt_level=opt_level, config={"relay.backend.use_auto_scheduler": True}
```
- **EN**: This block continues `tvm` and works to hand work to a compiler/backend pipeline. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `tvm`，用于将工作移交给编译器或后端流水线。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 90-101
```python
            ),
        ):
            lib = relay.build(mod, target=target, params=params)
    elif scheduler == "meta_schedule":
        # pyrefly: ignore [missing-import]
        from tvm import meta_schedule as ms

        with tempfile.TemporaryDirectory() as work_dir:
            if device.type != "cuda":
                # meta_schedule needs num-cores to be specified
                # here we use the maximum core count
                target = tvm.target.Target(
```
- **EN**: This block continues `tvm` and works to normalize dtype/device related arguments and behavior. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `tvm`，用于规范化 dtype/device 相关参数与行为。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 102-113
```python
                    f"{llvm_target()} --num-cores {ms.utils.cpu_count(logical=False)}"
                )
            # TODO(shingjan): This could be replaced by tvm.contrib.torch.optimize_torch
            # once USE_PT_TVMDSOOP is updated and turned on by default in TVM.
            assert trials > 0
            database = ms.relay_integration.tune_relay(
                mod=mod,
                target=target,
                work_dir=work_dir,
                max_trials_global=trials,
                num_trials_per_iter=64,
                params=params,
```
- **EN**: This block continues `tvm` and works to capture Python execution for torch.compile and maintain compiler state.
- **CN**: 该代码块继续实现 `tvm`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 114-125
```python
                strategy="evolutionary",
                opt_level=opt_level,
            )
            lib = ms.relay_integration.compile_relay(
                database=database,
                mod=mod,
                target=target,
                params=params,
                opt_level=opt_level,
            )
    elif scheduler == "default" or not scheduler:
        # no autotuning
```
- **EN**: This block continues `tvm` and works to hand work to a compiler/backend pipeline.
- **CN**: 该代码块继续实现 `tvm`，用于将工作移交给编译器或后端流水线。

### Lines 126-143
```python
        with tvm.transform.PassContext(opt_level=opt_level):
            lib = relay.build(mod, target=target, params=params)
    else:
        raise NotImplementedError(
            "This tuning option is invalid/not implemented for torchdynamo's TVM-related backend. "
            "There are three available options: default, auto_scheduler and meta_schedule."
        )
    m = graph_executor.GraphModule(lib["default"](dev))

    def to_torch_tensor(nd_tensor: tvm.nd.array) -> torch.Tensor:
        """A helper function to transfer a NDArray to torch.tensor."""
        if nd_tensor.dtype == "bool":
            # DLPack does not support boolean so it can't be handled by
            # torch.utils.dlpack.from_pack. Workaround by going through
            # numpy, although this brings additional data copy overhead.
            return torch.from_numpy(nd_tensor.numpy())
        return torch.utils.dlpack.from_dlpack(nd_tensor.to_dlpack())
```
- **EN**: Defines the `tvm` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`tvm` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 144-155
```python
    def to_tvm_tensor(torch_tensor: torch.Tensor) -> tvm.nd.array:
        """A helper function to transfer a torch.tensor to NDArray."""
        if torch_tensor.dtype == torch.bool:
            # same reason as above, fallback to numpy conversion which
            # could introduce data copy overhead
            return tvm.nd.array(torch_tensor.cpu().numpy())
        return tvm.nd.from_dlpack(torch_tensor)

    def exec_tvm(*i_args: torch.Tensor) -> list[torch.Tensor]:
        args = [a.contiguous() for a in i_args]
        shape_info, _ = m.get_input_info()
        active_inputs = {name for name, _ in shape_info.items()}
```
- **EN**: Defines the `tvm` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`tvm` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 156-173
```python
        for idx, arg in enumerate(args, 0):
            if arg.dim() != 0:
                if arg.requires_grad:
                    arg = arg.detach()
                inp_name = f"inp_{idx}"
                if inp_name not in active_inputs:
                    log.warning(
                        "input %s skipped as not found in tvm's runtime library",
                        inp_name,
                    )
                    continue
                m.set_input(
                    inp_name,
                    to_tvm_tensor(arg),
                )
        m.run()
        return [to_torch_tensor(m.get_output(i)) for i in range(m.get_num_outputs())]
```
- **EN**: This block continues `tvm` and works to connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `tvm`，用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 174-188
```python
    return exec_tvm


tvm_meta_schedule = functools.partial(tvm, scheduler="meta_schedule")
tvm_auto_scheduler = functools.partial(tvm, scheduler="auto_scheduler")


def has_tvm() -> bool:
    try:
        importlib.import_module("tvm")
        return True
    except ImportError:
        return False
```
- **EN**: This block adds optional-import fallback behavior so the module can degrade gracefully when an extra dependency is absent.
- **CN**: 该代码块为可选依赖提供降级回退逻辑，从而在额外依赖缺失时保持模块可用。

### Lines 189-197
```python
@functools.cache
def llvm_target() -> str:
    if sys.platform == "linux":
        cpuinfo = Path("/proc/cpuinfo").read_text()
        if "avx512" in cpuinfo:
            return "llvm -mcpu=skylake-avx512"
        elif "avx2" in cpuinfo:
            return "llvm -mcpu=core-avx2"
    return "llvm"
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `.common`, `.registry`
- **Standard library / 标准库**: `functools`, `importlib`, `logging`, `os`, `sys`, `tempfile`, `collections.abc`, `pathlib`, `types`, `typing`
- **Third-party packages / 第三方包**: `tvm`, `tvm.contrib`
- **Primary symbols / 核心符号**: `tvm`, `has_tvm`, `llvm_target`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
