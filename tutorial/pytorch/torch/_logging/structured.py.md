# structured.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_logging/structured.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides structured logging helpers, configuration surfaces, and log-registration utilities.
- **Purpose (CN)**: 提供结构化日志辅助逻辑、配置接口以及日志注册工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行
````python
"""
Utilities for converting data types into structured JSON for dumping.
"""

import inspect
import os
import traceback
from collections.abc import Sequence
from typing import Any

import torch._logging._internal
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._logging._internal; standard-library helpers such as inspect, os, traceback, .... The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._logging._internal；标准库辅助模块，如 inspect、os、traceback、...。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 14-22 / 第 14-22 行
````python
INTERN_TABLE: dict[str, int] = {}


DUMPED_FILES: set[str] = set()


def intern_string(s: str | None) -> int:
    if s is None:
        return -1
````
- **EN**: This chunk defines `intern_string`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `intern_string`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 24-31 / 第 24-31 行
````python
    r = INTERN_TABLE.get(s)
    if r is None:
        r = len(INTERN_TABLE)
        INTERN_TABLE[s] = r
        torch._logging._internal.trace_structured(
            "str", lambda: (s, r), suppress_context=True
        )
    return r
````
- **EN**: This chunk continues `intern_string` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `intern_string`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 34-47 / 第 34-47 行
````python
def dump_file(filename: str) -> None:
    if "eval_with_key" not in filename:
        return
    if filename in DUMPED_FILES:
        return
    DUMPED_FILES.add(filename)
    from torch.fx.graph_module import _loader

    torch._logging._internal.trace_structured(
        "dump_file",
        metadata_fn=lambda: {
            "name": filename,
        },
        payload_fn=lambda: _loader.get_source(filename),
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.graph_module. This chunk defines `dump_file`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.graph_module。 这一段定义了 `dump_file`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 48-61 / 第 48-61 行
````python
    )


def from_traceback(tb: Sequence[traceback.FrameSummary]) -> list[dict[str, Any]]:
    # dict naming convention here coincides with
    # python/combined_traceback.cpp
    r = [
        {
            "line": frame.lineno,
            "name": frame.name,
            "filename": intern_string(frame.filename),
            "loc": frame.line,
        }
        for frame in tb
````
- **EN**: This chunk defines `from_traceback`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `from_traceback`，其作用是协调 tracing、捕获或编译所需的图相关状态。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 62-74 / 第 62-74 行
````python
    ]
    return r


def get_user_stack(num_frames: int) -> list[dict[str, Any]]:
    from torch._guards import TracingContext
    from torch.utils._traceback import CapturedTraceback

    user_tb = TracingContext.extract_stack()
    if user_tb:
        return from_traceback(user_tb[-1 * num_frames :])

    tb = CapturedTraceback.extract().summary()
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._guards, torch.utils._traceback. This chunk defines `get_user_stack`, which retrieves runtime state and exposes it through a Python-friendly accessor. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._guards、torch.utils._traceback。 这一段定义了 `get_user_stack`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 76-84 / 第 76-84 行
````python
    # Filter out frames that are within the torch/ codebase
    torch_filepath = os.path.dirname(inspect.getfile(torch)) + os.path.sep
    for i, frame in enumerate(reversed(tb)):
        if torch_filepath not in frame.filename:
            # Only display `num_frames` frames in the traceback
            filtered_tb = tb[len(tb) - i - num_frames : len(tb) - i]
            return from_traceback(filtered_tb)

    return from_traceback(tb[-1 * num_frames :])
````
- **EN**: This chunk continues `get_user_stack` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_user_stack`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 87-100 / 第 87-100 行
````python
def get_framework_stack(
    num_frames: int = 25, cpp: bool = False
) -> list[dict[str, Any]]:
    """
    Returns the traceback for the user stack and the framework stack
    """
    from torch.fx.experimental.symbolic_shapes import uninteresting_files
    from torch.utils._traceback import CapturedTraceback

    tb = CapturedTraceback.extract(cpp=cpp).summary()
    tb = [
        frame
        for frame in tb
        if (
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch.fx.experimental.symbolic_shapes, torch.utils._traceback. This chunk defines `get_framework_stack`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch.fx.experimental.symbolic_shapes、torch.utils._traceback。 这一段定义了 `get_framework_stack`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 101-109 / 第 101-109 行
````python
            (
                frame.filename.endswith(".py")
                and frame.filename not in uninteresting_files()
            )
            or ("at::" in frame.name or "torch::" in frame.name)
        )
    ]

    return from_traceback(tb[-1 * num_frames :])
````
- **EN**: This chunk continues `get_framework_stack` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_framework_stack`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Structured logging**
  - EN: Routes events through configurable logging surfaces instead of ad hoc prints.
  - CN: 通过可配置日志接口路由事件，而不是零散打印。
- **INTERN_TABLE**
  - EN: `INTERN_TABLE` is one of the main symbols declared or implemented in this file.
  - CN: `INTERN_TABLE` 是本文件声明或实现的主要符号之一。
- **DUMPED_FILES**
  - EN: `DUMPED_FILES` is one of the main symbols declared or implemented in this file.
  - CN: `DUMPED_FILES` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch._logging._internal`, `torch.fx.graph_module`, `torch._guards`, `torch.utils._traceback`, `torch.fx.experimental.symbolic_shapes`
- **Standard library / 标准库**: `inspect`, `os`, `traceback`, `collections.abc`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `INTERN_TABLE`, `DUMPED_FILES`, `intern_string`, `dump_file`, `from_traceback`, `get_user_stack`, `get_framework_stack`
