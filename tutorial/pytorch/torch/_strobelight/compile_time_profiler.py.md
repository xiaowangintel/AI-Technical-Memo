# compile_time_profiler.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_strobelight/compile_time_profiler.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides general-purpose torch Python helpers, package wiring, or small runtime-facing utilities.
- **Purpose (CN)**: 提供通用的 torch Python 辅助逻辑、包级连接代码或小型运行时工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
````python
# mypy: disallow-untyped-defs

import json
import logging
import os
import re
import subprocess
from datetime import datetime
from socket import gethostname
from typing import Any

from torch._strobelight.cli_function_profiler import StrobelightCLIFunctionProfiler


logger = logging.getLogger("strobelight_compile_time_profiler")
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._strobelight.cli_function_profiler; standard-library helpers such as json, logging, os, ....
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._strobelight.cli_function_profiler；标准库辅助模块，如 json、logging、os、...。

### Lines 17-36 / 第 17-36 行
````python
console_handler = logging.StreamHandler()
formatter = logging.Formatter(
    "%(name)s, line %(lineno)d, %(asctime)s, %(levelname)s: %(message)s"
)
console_handler.setFormatter(formatter)

logger.addHandler(console_handler)
logger.setLevel(logging.INFO)
logger.propagate = False


def get_fburl(url: str) -> str:
    short_url = url
    # Attempt to shorten the URL
    try:
        result = subprocess.run(
            ["fburl", url], capture_output=True, stdin=subprocess.DEVNULL
        )
        if result.returncode == 0:
            short_url = result.stdout.decode("utf-8")
````
- **EN**: This chunk defines `get_fburl`, which retrieves runtime state and exposes it through a Python-friendly accessor. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `get_fburl`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 37-56 / 第 37-56 行
````python
    except Exception as e:
        logger.warning("URL shortening failed: %s, using long URL", repr(e))
    return short_url


def get_strobelight_url(identifier: str) -> str:
    scuba_json = {
        "aggregateList": [],
        "aggregation_field": "async_stack_complete",
        "b_constraints": [[]],
        "c_constraints": [[]],
        "cols": ["namespace_id", "namespace_process_id"],
        "compare": "none",
        "constraints": [
            [{"column": "sample_tags", "op": "all", "value": [f'["{identifier}"]']}]
        ],
        "derivedCols": [],
        "end": "now",
        "enumCols": [],
        "filterMode": "DEFAULT",
````
- **EN**: This chunk defines `get_strobelight_url`, which retrieves runtime state and exposes it through a Python-friendly accessor. Exception-handling paths normalize backend/runtime failures into predictable Python-visible behavior. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_strobelight_url`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 异常处理路径会把后端/运行时失败规范化为可预期的 Python 可见行为。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 57-76 / 第 57-76 行
````python
        "hideEmptyColumns": "false",
        "ignoreGroupByInComparison": "false",
        "is_timeseries": "false",
        "mappedCols": [],
        "metric": "count",
        "modifiers": [],
        "order": "weight",
        "order_desc": "true",
        "param_dimensions": [
            {"dim": "py_async_stack", "op": "edge", "param": "0", "anchor": "0"}
        ],
        "purposes": [],
        "return_remainder": "false",
        "samplingRatio": "1",
        "should_pivot": "false",
        "start": "-30 days",
        "timezone": "America/Los_Angeles",
        "top": 10000,
    }
    scuba_url_prefix = "https://www.internalfb.com/intern/scuba/query/?dataset=pyperf_experimental/on_demand&drillstate="
````
- **EN**: This chunk continues `get_strobelight_url` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `get_strobelight_url`，进一步展开其内部控制流或状态更新。

### Lines 77-94 / 第 77-94 行
````python
    scuba_url_suff = "&view=GraphProfilerView&&normalized=1726332703&pool=uber"
    long_url = scuba_url_prefix + json.dumps(scuba_json) + scuba_url_suff
    return get_fburl(long_url)


class StrobelightCompileTimeProfiler:
    success_profile_count: int = 0
    failed_profile_count: int = 0
    ignored_profile_runs: int = 0
    inside_profile_compile_time: bool = False
    enabled: bool = False

    # A regex that can be used to filter out what frames to profile. ex: "1/.*"
    frame_id_filter: str | None = os.environ.get("COMPILE_STROBELIGHT_FRAME_FILTER")

    # A unique identifier that is used as the run_user_name in the strobelight profile to
    # associate all compile time profiles together.
    identifier: str | None = None
````
- **EN**: It introduces or extends `StrobelightCompileTimeProfiler`, which hold the main object-oriented state for this portion of the file. This chunk continues `StrobelightCompileTimeProfiler` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `StrobelightCompileTimeProfiler`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `StrobelightCompileTimeProfiler`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 96-113 / 第 96-113 行
````python
    current_phase: str | None = None

    profiler: Any | None = None

    max_stack_length: int = int(
        os.environ.get("COMPILE_STROBELIGHT_MAX_STACK_LENGTH", 500)
    )
    max_profile_time: int = int(
        os.environ.get("COMPILE_STROBELIGHT_MAX_PROFILE_TIME", 60 * 30)
    )
    # Collect sample each x cycles.
    sample_each: int = int(
        float(os.environ.get("COMPILE_STROBELIGHT_SAMPLE_RATE", 1e7))
    )

    @classmethod
    def get_frame(cls) -> str:
        from torch._guards import CompileContext
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._guards. This chunk defines `get_frame`, which retrieves runtime state and exposes it through a Python-friendly accessor. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._guards。 这一段定义了 `get_frame`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 115-133 / 第 115-133 行
````python
        return (str)(CompileContext.current_trace_id())

    @classmethod
    def enable(cls, profiler_class: Any = StrobelightCLIFunctionProfiler) -> None:
        if cls.enabled:
            logger.info("compile time strobelight profiling already enabled")
            return

        logger.info("compile time strobelight profiling enabled")

        if profiler_class is StrobelightCLIFunctionProfiler:
            import shutil

            if not shutil.which("strobeclient"):
                logger.info(
                    "strobeclient not found, can't enable compile time strobelight profiling, seems"
                    "like you are not on a FB machine."
                )
                return
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as shutil. This chunk defines `enable`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 shutil。 这一段定义了 `enable`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 135-149 / 第 135-149 行
````python
        cls.enabled = True
        cls._cls_init()
        # profiler_class should have public API similar to that of StrobelightCLIFunctionProfiler.
        # we have pass different functionProfilerClass for meta-internal fbcode targets.
        # NB: the actual implementation in Meta is at
        # fbcode/caffe2/fb/strobelight/function_profiler.py
        cls.profiler = profiler_class(
            sample_each=cls.sample_each,
            max_profile_duration_sec=cls.max_profile_time,
            stack_max_len=cls.max_stack_length,
            async_stack_max_len=cls.max_stack_length,
            run_user_name="pt2-profiler/"
            + os.environ.get("USER", os.environ.get("USERNAME", "")),
            sample_tags={cls.identifier},
        )
````
- **EN**: This chunk continues `enable` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `enable`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 151-163 / 第 151-163 行
````python
    @classmethod
    def _cls_init(cls) -> None:
        cls.identifier = "{date}{pid}{hostname}".format(
            date=datetime.now().strftime("%Y-%m-%d-%H:%M:%S"),
            pid=os.getpid(),
            hostname=gethostname(),
        )

        logger.info("Unique sample tag for this run is: %s", cls.identifier)
        logger.info(
            "URL to access the strobelight profile at the end of the run: %s",
            get_strobelight_url(cls.identifier),
        )
````
- **EN**: This chunk defines `_cls_init`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `_cls_init`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 165-182 / 第 165-182 行
````python
    @classmethod
    def _log_stats(cls) -> None:
        logger.info(
            "%s strobelight success runs out of %s non-recursive compilation events.",
            cls.success_profile_count,
            cls.success_profile_count + cls.failed_profile_count,
        )

    # TODO use threadlevel meta data to tags to record phases.
    @classmethod
    def profile_compile_time(
        cls, func: Any, phase_name: str, *args: Any, **kwargs: Any
    ) -> Any:
        def skip() -> Any:
            return func(*args, **kwargs)

        if not cls.enabled:
            return skip()
````
- **EN**: This chunk defines `skip`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `skip`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 184-200 / 第 184-200 行
````python
        if cls.profiler is None:
            logger.error("profiler is not set")
            return

        frame_id = cls.get_frame()

        if cls.inside_profile_compile_time:
            cls.ignored_profile_runs += 1
            logger.info(
                "profile_compile_time is requested for phase: %s, frame %s, while already in running phase: %s,"
                "frame %s, recursive call ignored",
                phase_name,
                frame_id,
                cls.current_phase,
                frame_id,
            )
            return skip()
````
- **EN**: This chunk continues `skip` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `skip`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 202-220 / 第 202-220 行
````python
        if cls.frame_id_filter is not None:
            should_run = re.match(cls.frame_id_filter, frame_id) is not None
            if not should_run:
                logger.info(
                    "profiling frame %s is skipped due to frame_id_filter %s",
                    frame_id,
                    cls.frame_id_filter,
                )
                return skip()

        cls.inside_profile_compile_time = True
        cls.current_phase = phase_name
        logger.info("profiling frame %s", frame_id)
        work_result = cls.profiler.profile(func, *args, **kwargs)

        if cls.profiler.profile_result is not None:
            cls.success_profile_count += 1
        else:
            cls.failed_profile_count += 1
````
- **EN**: This chunk continues `skip` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `skip`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 222-224 / 第 222-224 行
````python
        cls._log_stats()
        cls.inside_profile_compile_time = False
        return work_result
````
- **EN**: This chunk continues `skip` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `skip`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **get_fburl**
  - EN: `get_fburl` is one of the main symbols declared or implemented in this file.
  - CN: `get_fburl` 是本文件声明或实现的主要符号之一。
- **get_strobelight_url**
  - EN: `get_strobelight_url` is one of the main symbols declared or implemented in this file.
  - CN: `get_strobelight_url` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Profiler integration**
  - EN: The file records, surfaces, or configures trace information for later performance analysis.
  - CN: 该文件会记录、暴露或配置跟踪信息，以便后续性能分析。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch._strobelight.cli_function_profiler`, `torch._guards`
- **Standard library / 标准库**: `json`, `logging`, `os`, `re`, `subprocess`, `datetime`, `socket`, `typing`, `shutil`
- **Primary symbols in this file / 本文件核心符号**: `get_fburl`, `get_strobelight_url`, `StrobelightCompileTimeProfiler`
