# _graph_annotations.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/cuda/_graph_annotations.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-facing Python wrappers for streams, memory, random state, and runtime management.
- **Purpose (CN)**: 实现面向 CUDA 的 Python 包装层，用于流、内存、随机状态以及运行时管理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
````python
"""Annotate CUDA graph kernel nodes during capture.

During CUDA graph capture, ``mark_kernels`` uses ``cudaGraphGetNodes``
to count nodes before and after the wrapped region.  Nodes at indices
``[before, after)`` are the ones added within the scope.  Each kernel
or memcpy node found is annotated by its ``toolsId`` so it can later
be matched to profiler trace events.

The annotations can be pickled and later merged into a Chrome profiler
trace using ``torch.cuda._annotate_cuda_graph_trace``.

Requires ``cuda.bindings`` package and a CUDA driver that supports
``cudaGraphNodeGetToolsId`` (CUDA >= 13.1 or appropriate cuda-compat).
When unavailable, ``mark_kernels`` silently becomes a no-op.

Usage during capture::
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 18-35 / 第 18-35 行
````python
    from torch.cuda._graph_annotations import (
        enable_annotations,
        mark_kernels,
        resolve_pending_annotations,
        remap_to_exec_graph,
    )

    enable_annotations()

    with torch.cuda.graph(graph):
        with mark_kernels("phase_A"):
            y = workload_a(x)
        with mark_kernels("phase_B"):
            z = workload_b(y)
        resolve_pending_annotations()

    remap_to_exec_graph(graph)
"""
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.cuda._graph_annotations. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.cuda._graph_annotations。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 37-54 / 第 37-54 行
````python
from collections import defaultdict
from contextlib import contextmanager
from logging import getLogger
from typing import Any

import torch
from torch.cuda._utils import _check_cuda_bindings, _HAS_CUDA_BINDINGS


try:
    from cuda.bindings import (  # pyrefly: ignore[missing-import]
        runtime as _cuda_runtime,
    )
except ImportError:
    _cuda_runtime = None  # type: ignore[assignment]


logger = getLogger(__name__)
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.cuda._utils; standard-library helpers such as collections, contextlib, logging, ...; other helper packages such as cuda.bindings. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.cuda._utils；标准库辅助模块，如 collections、contextlib、logging、...；其他辅助包，如 cuda.bindings。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。

### Lines 57-74 / 第 57-74 行
````python
# Tri-state: None = not probed, True = available, False = unavailable.
# Deferred to first use to avoid premature CUDA initialization.
_tools_id_available: bool | None = None

# Global kill switch. When False, mark_kernels and mark_stream are no-ops.
_annotations_enabled: bool = False


def enable_annotations() -> None:
    """Enable kernel annotation recording."""
    global _annotations_enabled
    _annotations_enabled = True


def disable_annotations() -> None:
    """Disable kernel annotation recording."""
    global _annotations_enabled
    _annotations_enabled = False
````
- **EN**: This chunk defines `disable_annotations`, which mutates configuration or backend state that affects later execution.
- **CN**: 这一段定义了 `disable_annotations`，其作用是修改会影响后续执行的配置或后端状态。

### Lines 77-96 / 第 77-96 行
````python
def _is_tools_id_unavailable() -> bool:
    """Return True if we already know cudaGraphNodeGetToolsId is missing."""
    if not _HAS_CUDA_BINDINGS:
        return True
    if _tools_id_available is False:
        return True
    if not hasattr(_cuda_runtime, "cudaGraphNodeGetToolsId"):
        return True
    return False


def _get_tools_id(node: Any) -> int | None:
    """Return the toolsId for a graph node, or None if unavailable."""
    global _tools_id_available
    if _tools_id_available is None:
        try:
            tools_id = _check_cuda_bindings(
                _cuda_runtime.cudaGraphNodeGetToolsId(  # pyrefly: ignore[missing-attribute]
                    node
                )
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_get_tools_id`, which implements a focused helper used by the surrounding module. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_get_tools_id`，其作用是实现周边模块使用的关键辅助逻辑。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 97-111 / 第 97-111 行
````python
            )
        except Exception:
            _tools_id_available = False
            logger.info(
                "cudaGraphNodeGetToolsId not available; "
                "CUDA graph kernel annotations will be disabled"
            )
            return None
        _tools_id_available = True
        return tools_id
    return _check_cuda_bindings(
        _cuda_runtime.cudaGraphNodeGetToolsId(  # pyrefly: ignore[missing-attribute]
            node
        )
    )
````
- **EN**: This chunk continues `_get_tools_id` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `_get_tools_id`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 114-126 / 第 114-126 行
````python
def _get_capture_graph(stream: Any) -> Any:
    """Return the graph handle for the active capture, or None."""
    status, _id, graph, _deps, _edge_data, _num_deps = _check_cuda_bindings(
        _cuda_runtime.cudaStreamGetCaptureInfo(  # pyrefly: ignore[missing-attribute]
            stream
        )
    )
    if (
        status
        != _cuda_runtime.cudaStreamCaptureStatus.cudaStreamCaptureStatusActive  # pyrefly: ignore[missing-attribute]
    ):
        return None
    return graph
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_get_capture_graph`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_get_capture_graph`，其作用是协调 tracing、捕获或编译所需的图相关状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 129-144 / 第 129-144 行
````python
def _get_node_count(graph: Any) -> int:
    """Return the number of nodes currently in the graph."""
    _, num = _check_cuda_bindings(
        _cuda_runtime.cudaGraphGetNodes(  # pyrefly: ignore[missing-attribute]
            graph, numNodes=0
        )
    )
    return num


# toolsId -> list of annotation objects.
_kernel_annotations: defaultdict[int, list[Any]] = defaultdict(list)

# Node types we annotate. Initialized lazily to avoid touching cuda.bindings
# at import time.
_ANNOTATABLE_TYPES: set[Any] | None = None
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_get_node_count`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_get_node_count`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 147-164 / 第 147-164 行
````python
def _get_annotatable_types() -> set[Any]:
    global _ANNOTATABLE_TYPES
    if _ANNOTATABLE_TYPES is None:
        _ANNOTATABLE_TYPES = {
            _cuda_runtime.cudaGraphNodeType.cudaGraphNodeTypeKernel,  # pyrefly: ignore[missing-attribute]
            _cuda_runtime.cudaGraphNodeType.cudaGraphNodeTypeMemcpy,  # pyrefly: ignore[missing-attribute]
        }
    return _ANNOTATABLE_TYPES


# Pending scopes: (annotation, start_node_index, end_node_index).
_pending_scopes: list[tuple[Any, int, int]] = []

# Graph handle saved during capture for post-capture resolution.
_capture_graph: Any = None

# Capture graph ID saved by resolve_pending_annotations for remap_to_exec_graph.
_last_capture_graph_id: int | None = None
````
- **EN**: This chunk defines `_get_annotatable_types`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_get_annotatable_types`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 167-186 / 第 167-186 行
````python
@contextmanager  # type: ignore[arg-type]
def mark_kernels(annotation: str | dict[str, Any]):
    """Context manager that records node index ranges for later annotation.

    During capture, calls ``cudaGraphGetNodes`` to count graph nodes before
    and after the scope.  Nodes at indices ``[before, after)`` were added
    inside the scope.  After capture, ``resolve_pending_annotations``
    enumerates all nodes and annotates kernel/memcpy nodes in those ranges.

    Must be called inside an active ``torch.cuda.graph()`` capture.  If the
    current stream is not capturing, or if ``cudaGraphNodeGetToolsId`` is not
    available, the context manager is a no-op.

    Args:
        annotation: Arbitrary object appended to the annotation list for
            every kernel/memcpy node whose index falls within this scope.
    """
    if not _annotations_enabled or _is_tools_id_unavailable():
        yield
        return
````
- **EN**: This chunk defines `mark_kernels`, which implements a focused helper used by the surrounding module. Decorators such as `contextmanager` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `mark_kernels`，其作用是实现周边模块使用的关键辅助逻辑。 像 `contextmanager` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 188-206 / 第 188-206 行
````python
    if isinstance(annotation, str):
        annotation = {"str": annotation}

    stream = _cuda_runtime.cudaStream_t(  # pyrefly: ignore[missing-attribute]
        init_value=torch.cuda.current_stream().cuda_stream
    )
    graph = _get_capture_graph(stream)
    if graph is None:
        yield
        return

    global _capture_graph
    _capture_graph = graph

    start_count = _get_node_count(graph)

    yield

    end_count = _get_node_count(graph)
````
- **EN**: This chunk continues `mark_kernels` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `mark_kernels`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 208-222 / 第 208-222 行
````python
    if end_count > start_count:
        _pending_scopes.append((annotation, start_count, end_count))


def resolve_pending_annotations() -> None:
    """Resolve pending scope index ranges into kernel annotations.

    Enumerates all graph nodes and annotates kernel/memcpy nodes whose
    indices fall within recorded scope ranges. Must be called while still
    inside the ``torch.cuda.graph()`` capture context.
    """
    global _capture_graph
    if not _pending_scopes:
        _capture_graph = None
        return
````
- **EN**: This chunk defines `resolve_pending_annotations`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `resolve_pending_annotations`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 224-241 / 第 224-241 行
````python
    # Get a fresh graph handle from the active capture.
    stream = _cuda_runtime.cudaStream_t(  # pyrefly: ignore[missing-attribute]
        init_value=torch.cuda.current_stream().cuda_stream
    )
    graph = _get_capture_graph(stream)
    if graph is None:
        graph = _capture_graph
    if graph is None:
        logger.warning("resolve_pending_annotations: no graph handle available")
        _pending_scopes.clear()
        return

    try:
        num = _get_node_count(graph)
        if num == 0:
            _pending_scopes.clear()
            _capture_graph = None
            return
````
- **EN**: This chunk continues `resolve_pending_annotations` and expands its internal control flow or state updates. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `resolve_pending_annotations`，进一步展开其内部控制流或状态更新。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 243-255 / 第 243-255 行
````python
        nodes, num = _check_cuda_bindings(
            _cuda_runtime.cudaGraphGetNodes(  # pyrefly: ignore[missing-attribute]
                graph, numNodes=num
            )
        )

        # Save capture graph ID for remap_to_exec_graph.
        global _last_capture_graph_id
        if num > 0:
            first_tid = _get_tools_id(nodes[0])
            _last_capture_graph_id = (first_tid >> 32) if first_tid else None

        annotatable = _get_annotatable_types()
````
- **EN**: This chunk continues `resolve_pending_annotations` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `resolve_pending_annotations`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 257-271 / 第 257-271 行
````python
        # Sort by (start, -end, -append_index). The append index encodes
        # nesting depth: inner context managers exit first, so they are
        # appended to _pending_scopes first (smaller index). Using
        # -append_index as tiebreaker ensures that for same-range scopes
        # the outer scope (larger index) sorts first and is pushed onto
        # the stack first, leaving the inner scope on top.
        sorted_scopes = sorted(
            (
                (ann, start, end, i)
                for i, (ann, start, end) in enumerate(_pending_scopes)
            ),
            key=lambda s: (s[1], -s[2], -s[3]),
        )
        scope_ptr = 0
        active_stack: list[tuple[int, Any]] = []  # (end_idx, annotation)
````
- **EN**: This chunk continues `resolve_pending_annotations` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `resolve_pending_annotations`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 273-286 / 第 273-286 行
````python
        for i in range(num):
            # Pop scopes whose range ended.
            while active_stack and active_stack[-1][0] <= i:
                active_stack.pop()

            # Push scopes that start at or before this index.
            while scope_ptr < len(sorted_scopes) and sorted_scopes[scope_ptr][1] <= i:
                ann, _start_idx, end_idx, _idx = sorted_scopes[scope_ptr]
                if end_idx > i:
                    active_stack.append((end_idx, ann))
                scope_ptr += 1

            if not active_stack:
                continue
````
- **EN**: This chunk continues `resolve_pending_annotations` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `resolve_pending_annotations`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 288-304 / 第 288-304 行
````python
            node = nodes[i]
            node_type = _check_cuda_bindings(
                _cuda_runtime.cudaGraphNodeGetType(  # pyrefly: ignore[missing-attribute]
                    node
                )
            )
            if node_type not in annotatable:
                continue

            tools_id = _get_tools_id(node)
            if tools_id is None:
                logger.warning(
                    "resolve_pending_annotations: toolsId unavailable, aborting"
                )
                _pending_scopes.clear()
                _capture_graph = None
                return
````
- **EN**: This chunk continues `resolve_pending_annotations` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `resolve_pending_annotations`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 306-325 / 第 306-325 行
````python
            if len(active_stack) == 1:
                _kernel_annotations[tools_id].append(active_stack[0][1])
            else:
                # Merge all active scopes into one dict. Inner scopes sit
                # on top of the stack. Iterating reversed (inner first)
                # with setdefault lets the inner scope's values win for
                # overlapping keys (e.g. name, stream) while outer scopes
                # fill in any missing keys.
                merged: dict[str, Any] = {}
                for _, ann in reversed(active_stack):
                    if isinstance(ann, dict):
                        for ak, av in ann.items():
                            merged.setdefault(ak, av)
                    else:
                        merged.setdefault("name", ann)
                _kernel_annotations[tools_id].append(merged)
    except Exception:
        logger.exception("resolve_pending_annotations failed")
    finally:
        _pending_scopes.clear()
````
- **EN**: This chunk continues `resolve_pending_annotations` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `resolve_pending_annotations`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 326-339 / 第 326-339 行
````python
        _capture_graph = None


def remap_to_exec_graph(torch_cuda_graph: torch.cuda.CUDAGraph) -> None:
    """Remap annotation keys from capture graph ID to exec graph ID.

    During capture, toolsId encodes the capture graph's ID in the upper
    32 bits. After instantiation, the profiler uses the exec graph's ID.
    This function rewrites the keys so annotations match the trace.

    Must be called after the ``torch.cuda.graph()`` context exits.
    """
    if not _kernel_annotations:
        return
````
- **EN**: This chunk defines `remap_to_exec_graph`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `remap_to_exec_graph`，其作用是协调 tracing、捕获或编译所需的图相关状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 341-360 / 第 341-360 行
````python
    exec_handle = _cuda_runtime.cudaGraphExec_t(  # pyrefly: ignore[missing-attribute]
        init_value=torch_cuda_graph.raw_cuda_graph_exec()
    )
    exec_graph_id = _check_cuda_bindings(
        _cuda_runtime.cudaGraphExecGetId(  # pyrefly: ignore[missing-attribute]
            exec_handle
        )
    )

    # Only remap annotations from the most recent capture graph.
    # Previously remapped annotations (from earlier captures) keep their
    # correct exec graph IDs.
    capture_graph_id = _last_capture_graph_id
    remapped: dict[int, list[Any]] = {}
    for tools_id, ann_list in _kernel_annotations.items():
        graph_id = tools_id >> 32
        if capture_graph_id is not None and graph_id != capture_graph_id:
            # Belongs to a different graph — keep as-is.
            remapped[tools_id] = ann_list
            continue
````
- **EN**: This chunk continues `remap_to_exec_graph` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `remap_to_exec_graph`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 361-374 / 第 361-374 行
````python
        node_id = tools_id & 0xFFFFFFFF
        new_tools_id = (exec_graph_id << 32) | node_id
        if new_tools_id in remapped:
            remapped[new_tools_id].extend(ann_list)
        else:
            remapped[new_tools_id] = list(ann_list)

    _kernel_annotations.clear()
    _kernel_annotations.update(remapped)


def get_kernel_annotations() -> dict[int, list[Any]]:
    """Return the current kernel annotation map (toolsId -> annotations)."""
    return _kernel_annotations
````
- **EN**: This chunk defines `get_kernel_annotations`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_kernel_annotations`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 377-389 / 第 377-389 行
````python
def clear_kernel_annotations() -> None:
    """Clear all recorded kernel annotations and pending scopes."""
    global _capture_graph
    _kernel_annotations.clear()
    _pending_scopes.clear()
    _capture_graph = None


# Counter-based stream ID registry. IDs start at 60 (above the highest
# observed non-graphed CUDA stream ID) so every assigned lane is visually
# distinct in Perfetto and doesn't collide with real streams.
_stream_id_counter: int = 60
_stream_id_map: dict[int, int] = {}
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `clear_kernel_annotations`, which drops cached state so a later execution phase can rebuild it cleanly. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `clear_kernel_annotations`，其作用是清除缓存状态，以便后续执行阶段干净地重建。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 392-408 / 第 392-408 行
````python
def _get_stream_id(stream: torch.cuda.Stream) -> int:
    """Return a small, stable stream ID for the given CUDA stream."""
    global _stream_id_counter
    key = stream.cuda_stream
    if key not in _stream_id_map:
        _stream_id_map[key] = _stream_id_counter
        _stream_id_counter += 1
    return _stream_id_map[key]


def get_stream_for_pg(pg_key: str) -> int:
    """Return a unique stream ID for the given process group key."""
    global _stream_id_counter
    if pg_key not in _stream_id_map:
        _stream_id_map[pg_key] = _stream_id_counter  # type: ignore[assignment]
        _stream_id_counter += 1
    return _stream_id_map[pg_key]  # type: ignore[return-value]
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `get_stream_for_pg`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `get_stream_for_pg`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 411-430 / 第 411-430 行
````python
@contextmanager  # type: ignore[arg-type]
def mark_stream(stream: torch.cuda.Stream, annotation: str | dict[str, Any]):
    """Switch to stream, inject its ID into annotation, and mark kernels.

    If *stream* is already the current stream, no stream switch or stream ID
    injection happens — the kernels stay on whatever stream is active (which
    keeps the trace faithful when e.g. FSDP uses the current stream for
    copy-in instead of a separate one).
    """
    if not _annotations_enabled:
        with torch.cuda.stream(stream):
            yield
        return
    if stream.cuda_stream == torch.cuda.current_stream().cuda_stream:
        with mark_kernels(annotation):
            yield
    else:
        if isinstance(annotation, str):
            annotation = {"str": annotation}
        if isinstance(annotation, dict):
````
- **EN**: This chunk defines `mark_stream`, which implements a focused helper used by the surrounding module. Decorators such as `contextmanager` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `mark_stream`，其作用是实现周边模块使用的关键辅助逻辑。 像 `contextmanager` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 431-434 / 第 431-434 行
````python
            annotation["stream"] = _get_stream_id(stream)
        with torch.cuda.stream(stream):
            with mark_kernels(annotation):
                yield
````
- **EN**: This chunk continues `mark_stream` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `mark_stream`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **CUDA runtime**
  - EN: Wraps streams, devices, memory state, and CUDA-specific coordination points.
  - CN: 封装流、设备、内存状态以及 CUDA 专用协同点。
- **enable_annotations**
  - EN: `enable_annotations` is one of the main symbols declared or implemented in this file.
  - CN: `enable_annotations` 是本文件声明或实现的主要符号之一。
- **disable_annotations**
  - EN: `disable_annotations` is one of the main symbols declared or implemented in this file.
  - CN: `disable_annotations` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Profiler integration**
  - EN: The file records, surfaces, or configures trace information for later performance analysis.
  - CN: 该文件会记录、暴露或配置跟踪信息，以便后续性能分析。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch.cuda._graph_annotations`, `torch`, `torch.cuda._utils`
- **Standard library / 标准库**: `collections`, `contextlib`, `logging`, `typing`
- **Other helper packages / 其他辅助包**: `cuda.bindings`
- **Primary symbols in this file / 本文件核心符号**: `enable_annotations`, `disable_annotations`, `_is_tools_id_unavailable`, `_get_tools_id`, `_get_capture_graph`, `_get_node_count`, `_ANNOTATABLE_TYPES`, `_get_annotatable_types`, `mark_kernels`, `resolve_pending_annotations`
