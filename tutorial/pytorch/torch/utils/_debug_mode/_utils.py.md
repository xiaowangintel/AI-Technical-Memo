# _utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/_debug_mode/_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `_utils.py`. The implementation is performance-facing and focuses on measurement fidelity or result reporting.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `_utils.py` 展开。 该实现面向性能分析，重点在于测量可信度或结果报告。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23 / 第 1-23 行
```python
# mypy: allow-untyped-defs
"""
Utility functions for DebugMode: tensor formatting, hashing, stack traces, and hook runners.
"""

import inspect
import os
import traceback
import weakref
from collections.abc import Callable
from typing import TYPE_CHECKING

import torch
from torch._subclasses.fake_tensor import FakeTensor
from torch.fx.graph import _parse_stack_trace
from torch.utils._dtype_abbrs import dtype_abbrs
from torch.utils._pytree import tree_map
from torch.utils._traceback import CapturedTraceback
from torch.utils.weak import WeakIdRef


if TYPE_CHECKING:
    from torch.utils._debug_mode._calls import _DebugCall
```
- **EN**: The opening docstring frames the module contract, telling readers what subsystem responsibilities the rest of the file implements. This block establishes the module dependencies, pulling in internal torch modules such as torch, torch._subclasses.fake_tensor:FakeTensor, torch.fx.graph:_parse_stack_trace, torch.utils._dtype_abbrs:dtype_abbrs; standard-library helpers such as inspect, os, traceback, weakref. Type-checking-only branches keep static analyzers informed without changing runtime behavior. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The implementation reasons about nested container structure, not just individual tensor values.
- **CN**: 开头的模块文档字符串给出了整体契约，说明后续代码要实现的子系统职责。 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch._subclasses.fake_tensor:FakeTensor, torch.fx.graph:_parse_stack_trace, torch.utils._dtype_abbrs:dtype_abbrs；标准库辅助模块，如 inspect, os, traceback, weakref。 仅用于类型检查的分支在不改变运行时行为的前提下为静态分析器提供信息。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该实现推理的是嵌套容器结构，而不只是单个张量值。

### Lines 26-46 / 第 26-46 行
```python
REDISTRIBUTE_FUNC = "redistribute_input"

# Tracks if we're in inductor benchmarking, and temporarily disables logging
# (for ignoring autotuning kernel launches which don't affect the user-facing result)
_IN_INDUCTOR_BENCHMARK: bool = False
# For record_outputs, log_tensor_hashes hooks for triton kernels.
# Stores kernel outputs in call.record["output"]
_RECORD_TRITON_OUTPUTS: bool = False
# Annotates kernel output hashes, and stores them in call.post_hashes
_TRITON_OUTPUT_HASH_FN: Callable | None = None
# Annotates kernel input hashes, and stores them in call.pre_hashes
_TRITON_INPUT_HASH_FN: Callable | None = None

# registered dispatch call hooks
_DISPATCH_RECORD_HOOKS: list[Callable] = []
_DISPATCH_LOG_HOOKS: list[Callable] = []
_DISPATCH_PRE_LOG_HOOKS: list[Callable] = []


def _stringify_shape(shape) -> str:
    return f"[{', '.join([str(x) for x in shape])}]"
```
- **EN**: Key callable entry points in this range include `_stringify_shape`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `REDISTRIBUTE_FUNC` centralize shared configuration or sentinel values. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `_stringify_shape`，它们把聚焦的行为封装成具名辅助函数或 API。 `REDISTRIBUTE_FUNC` 等具名常量把共享配置或哨兵值集中定义在一起。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 49-70 / 第 49-70 行
```python
def _stringify_device_mesh(mesh) -> str:
    return f"DM({', '.join([str(s) for s in mesh.shape])})"


def _stringify_placement(placement) -> str:
    return f"[{', '.join([str(p) for p in placement])}]"


def _stringify_attributes(tensor, attributes) -> str:
    pairs = {}
    for attr in attributes:
        if hasattr(tensor, attr):
            pairs[attr] = getattr(tensor, attr)
    if len(pairs) == 0:
        return ""
    return f"{{{', '.join([f'{k}={v}' for k, v in pairs.items()])}}}"


def _stringify_dtensor_spec(spec) -> str:
    from torch.distributed.tensor._dtensor_spec import DTensorSpec

    return DTensorSpec.format_shard_order_str(spec.placements, spec.shard_order)
```
- **EN**: Key callable entry points in this range include `_stringify_device_mesh`, `_stringify_placement`, `_stringify_attributes`, `_stringify_dtensor_spec`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_stringify_device_mesh`, `_stringify_placement`, `_stringify_attributes`, `_stringify_dtensor_spec`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 73-93 / 第 73-93 行
```python
class TensorIdTracker:
    def __init__(self) -> None:
        self.tensor_memo: dict[WeakIdRef, int] = {}
        self.next_tensor_id = 0

    def _id(self, tensor) -> int:
        with torch._C._DisablePythonDispatcher():
            o = WeakIdRef(tensor)

            def del_memo() -> None:
                self.tensor_memo.pop(o, None)

            weakref.finalize(tensor, del_memo)
            if o not in self.tensor_memo:
                self.tensor_memo[o] = self.next_tensor_id
                self.next_tensor_id += 1
            return self.tensor_memo[o]


def _tensor_debug_string(tensor, attributes, tensor_memo=None) -> str:
    """Convert tensor to debug string representation."""
```
- **EN**: It introduces or extends class-level abstractions such as `TensorIdTracker`, which organize state and behavior for this subsystem. Key callable entry points in this range include `_tensor_debug_string`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `TensorIdTracker` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `_tensor_debug_string`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 95-117 / 第 95-117 行
```python
    if isinstance(tensor, torch.Tensor):
        tensor_debug_str = f"{dtype_abbrs[tensor.dtype]}{_stringify_shape(tensor.shape)}{_stringify_attributes(tensor, attributes)}"
        id_str = f"${tensor_memo._id(tensor)}" if tensor_memo is not None else ""
        if isinstance(tensor, torch.distributed.tensor.DTensor):
            # omitted device mesh
            return f"dt{id_str}: {tensor_debug_str}| {_stringify_dtensor_spec(tensor._spec)}"
        elif isinstance(tensor, FakeTensor):
            return f"ft{id_str}: {tensor_debug_str}"
        else:
            return f"t{id_str}: {tensor_debug_str}"
    else:
        raise RuntimeError(f"Unsupported tensor type: {type(tensor)}")


def _arg_to_str(arg, attributes, tensor_memo=None) -> str:
    from torch.distributed.tensor._dtensor_spec import DTensorSpec

    def to_str(x):
        if isinstance(x, torch.Tensor):
            return _tensor_debug_string(x, attributes, tensor_memo)
        elif isinstance(x, DTensorSpec):
            return _stringify_dtensor_spec(x)
        return x
```
- **EN**: Key callable entry points in this range include `_tensor_debug_string`, `_arg_to_str`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_tensor_debug_string`, `_arg_to_str`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 119-137 / 第 119-137 行
```python
    arg = tree_map(to_str, arg)
    return str(arg)


def norm_hash_fn(t: torch.Tensor, use_scalar: bool = False) -> torch.Tensor | float:
    """
    from Observer. Computes a hash for a tensor by converting it to float (if needed), making it contiguous,
    replacing NaN/inf values with fixed numbers, and then computing the L1 norm in float64 or complex128.
    This is used to generate a deterministic summary value for tensor comparison.
    """
    with torch._C._DisablePythonDispatcher():
        if not (t.is_floating_point() or t.is_complex()):
            t = t.float()
        t = t.contiguous()

        if t.is_complex():
            t_float = t.to(dtype=torch.complex128)
        else:
            t_float = t.to(dtype=torch.float64)
```
- **EN**: Key callable entry points in this range include `_arg_to_str`, `norm_hash_fn`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_arg_to_str`, `norm_hash_fn`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 139-157 / 第 139-157 行
```python
        out = t_float.norm(p=1)
        if use_scalar:
            return out.item()
        return out


def _compute_rel_diff(hash1, hash2):
    # Relative difference: |hash1 - hash2| / max(|hash1|, |hash2|, eps)
    numerator = abs(hash1 - hash2)
    denominator = max(abs(hash1), abs(hash2), 1e-10)
    return numerator / denominator


def hash_tensor_fn(t: torch.Tensor, use_scalar: bool = False) -> torch.Tensor | int:
    """
    wrapper over torch.hash_tensor
    """
    if isinstance(t, torch.distributed.tensor.DTensor):
        t = t.to_local()
```
- **EN**: Key callable entry points in this range include `norm_hash_fn`, `_compute_rel_diff`, `hash_tensor_fn`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `norm_hash_fn`, `_compute_rel_diff`, `hash_tensor_fn`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 159-177 / 第 159-177 行
```python
    if t.is_floating_point():
        t_clean = t.to(dtype=torch.float64)
    elif t.is_complex():
        t_clean = t.to(dtype=torch.complex128).view(torch.float64)
    else:
        t_clean = t.to(dtype=torch.int64)

    if t.numel() > 0:
        out = torch.hash_tensor(t_clean)
    else:
        out = torch.zeros((), device=t_clean.device, dtype=torch.uint64)

    if use_scalar:
        return out.item()  # type: ignore[attribute]
    return out


def _get_stack_trace() -> str:
    from torch.fx.experimental.symbolic_shapes import uninteresting_files
```
- **EN**: Key callable entry points in this range include `hash_tensor_fn`, `_get_stack_trace`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `hash_tensor_fn`, `_get_stack_trace`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 179-195 / 第 179-195 行
```python
    summary = CapturedTraceback.extract().summary()
    summary = summary[:-4]  # filter out DebugMode frames
    summary = [
        frame for frame in summary if frame.filename not in uninteresting_files()
    ]
    summary = traceback.StackSummary.from_list(summary)
    return "".join(summary.format())


def _get_user_stack_trace(stack_trace_str: str) -> str | None:
    # Extract user code stack trace, filtering out torch internals.
    torch_dir = os.path.dirname(inspect.getfile(torch))
    filter_fn = lambda file, name, code: not file.startswith(torch_dir + os.path.sep)  # noqa: E731
    trace = _parse_stack_trace(stack_trace_str, filter_fn=filter_fn)
    if trace:
        return f"File: {trace.file}:{trace.lineno} in {trace.name}, code: {trace.code}"
    return None
```
- **EN**: Key callable entry points in this range include `_get_stack_trace`, `_get_user_stack_trace`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_get_stack_trace`, `_get_user_stack_trace`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 198-220 / 第 198-220 行
```python
def _maybe_get_autograd_trace() -> str | None:
    if torch._C._current_autograd_node() is not None:
        tb = torch._C._current_autograd_node().metadata.get("traceback_")  # type: ignore[attr-defined]
        if tb:
            return "".join(tb)
    return None


def _get_op_name(op) -> str:
    if isinstance(op, torch._ops.OpOverload):
        op_name = op.__qualname__
    elif hasattr(op, "__module__") and hasattr(op, "__name__"):
        op_name = f"{op.__module__}.{op.__name__}"
    else:
        op_name = str(op)
    return op_name


def _run_hook(hook, *args):
    out = hook(*args)
    if out is not None and not isinstance(out, dict):
        raise AssertionError(f"hook must return None or dict, got {type(out).__name__}")
    return out
```
- **EN**: Key callable entry points in this range include `_maybe_get_autograd_trace`, `_get_op_name`, `_run_hook`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. Hook registration/order is important because the value comes from when callbacks fire, not just what they compute.
- **CN**: 这一段的重要可调用入口包括 `_maybe_get_autograd_trace`, `_get_op_name`, `_run_hook`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 钩子的价值不仅在于计算什么，更在于何时触发以及触发顺序。

### Lines 223-242 / 第 223-242 行
```python
def _run_dispatch_pre_log_hooks(call: "_DebugCall", func, types, args, kwargs) -> None:
    if _DISPATCH_PRE_LOG_HOOKS:
        for hook in _DISPATCH_PRE_LOG_HOOKS:
            hook_out = _run_hook(hook, func, types, args, kwargs, call)
            if hook_out is not None:
                # Store pre-hook results in call.log
                if call.log is None:
                    call.log = {}
                call.log.update(hook_out)


def _run_dispatch_hooks(call: "_DebugCall", func, types, args, kwargs, result) -> None:
    if _DISPATCH_RECORD_HOOKS:
        record = {}
        for hook in _DISPATCH_RECORD_HOOKS:
            hook_out = _run_hook(hook, func, types, args, kwargs, result)
            if hook_out is not None:
                record.update(hook_out)
        if record:
            call.record = record
```
- **EN**: Key callable entry points in this range include `_run_dispatch_pre_log_hooks`, `_run_dispatch_hooks`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Hook registration/order is important because the value comes from when callbacks fire, not just what they compute.
- **CN**: 这一段的重要可调用入口包括 `_run_dispatch_pre_log_hooks`, `_run_dispatch_hooks`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 钩子的价值不仅在于计算什么，更在于何时触发以及触发顺序。

### Lines 244-251 / 第 244-251 行
```python
    if _DISPATCH_LOG_HOOKS:
        # Preserve existing log from pre-hooks (e.g., input_hash)
        if call.log is None:
            call.log = {}
        for hook in _DISPATCH_LOG_HOOKS:
            hook_out = _run_hook(hook, func, types, args, kwargs, result)
            if hook_out is not None:
                call.log.update(hook_out)
```
- **EN**: Key callable entry points in this range include `_run_dispatch_hooks`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Hook registration/order is important because the value comes from when callbacks fire, not just what they compute.
- **CN**: 这一段的重要可调用入口包括 `_run_dispatch_hooks`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 钩子的价值不仅在于计算什么，更在于何时触发以及触发顺序。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Observer statistics**
  - EN: Collects runtime ranges or histograms so later code can derive quantization settings.
  - CN: 收集运行时范围或直方图，以便后续逻辑推导量化设置。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **Pytree processing**
  - EN: Manipulates nested Python container structure while preserving leaves and shape metadata.
  - CN: 在保留叶子节点与结构元数据的同时处理嵌套 Python 容器。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch._subclasses.fake_tensor:FakeTensor`, `torch.fx.graph:_parse_stack_trace`, `torch.utils._dtype_abbrs:dtype_abbrs`, `torch.utils._pytree:tree_map`, `torch.utils._traceback:CapturedTraceback`, `torch.utils.weak:WeakIdRef`
- **Python standard library / Python 标准库**: `inspect`, `os`, `traceback`, `weakref`, `collections.abc:Callable`, `typing:TYPE_CHECKING`
- **Primary symbols / 核心符号**: `TensorIdTracker`, `_stringify_shape`, `_stringify_device_mesh`, `_stringify_placement`, `_stringify_attributes`, `_stringify_dtensor_spec`, `_tensor_debug_string`, `_arg_to_str`, `norm_hash_fn`, `_compute_rel_diff`, `hash_tensor_fn`, `_get_stack_trace`, `_get_user_stack_trace`, `_maybe_get_autograd_trace`, `_get_op_name`
