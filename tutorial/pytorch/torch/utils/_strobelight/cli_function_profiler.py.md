# cli_function_profiler.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/_strobelight/cli_function_profiler.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `cli_function_profiler.py`. Runtime configuration, environment overrides, or feature gating are important in this module. It bridges PyTorch values or graphs into visualization-friendly event formats.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `cli_function_profiler.py` 展开。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。 它把 PyTorch 数值或计算图桥接到适合可视化的事件格式。
## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行
```python
# mypy: disallow-untyped-defs

import functools
import logging
import os
import re
import subprocess
import time
from collections.abc import Callable, Sequence
from threading import Lock
from typing import Any, TypeVar
from typing_extensions import ParamSpec


logger = logging.getLogger("strobelight_function_profiler")

console_handler = logging.StreamHandler()
formatter = logging.Formatter(
    "%(name)s, line %(lineno)d, %(asctime)s, %(levelname)s: %(message)s"
)
console_handler.setFormatter(formatter)
```
- **EN**: This block establishes the module dependencies, pulling in standard-library helpers such as functools, logging, os, re; external packages such as typing_extensions:ParamSpec.
- **CN**: 这一段建立模块依赖，引入了标准库辅助模块，如 functools, logging, os, re；外部包，如 typing_extensions:ParamSpec。

### Lines 23-41 / 第 23-41 行
```python
logger.addHandler(console_handler)
logger.setLevel(logging.INFO)
logger.propagate = False

_P = ParamSpec("_P")
_R = TypeVar("_R")


class StrobelightCLIProfilerError(Exception):
    """
    Raised when an error happens during strobelight profiling
    """


def _pid_namespace_link(pid: int | None = None) -> str:
    """Returns the link to the process's namespace, example: pid:[4026531836]"""
    PID_NAMESPACE_PATH = "/proc/{}/ns/pid"
    pid = pid or os.getpid()
    return os.readlink(PID_NAMESPACE_PATH.format(pid))
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIProfilerError`, which organize state and behavior for this subsystem. Key callable entry points in this range include `_pid_namespace_link`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `_P`, `_R`, `PID_NAMESPACE_PATH` centralize shared configuration or sentinel values. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIProfilerError` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `_pid_namespace_link`，它们把聚焦的行为封装成具名辅助函数或 API。 `_P, _R, PID_NAMESPACE_PATH` 等具名常量把共享配置或哨兵值集中定义在一起。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 44-66 / 第 44-66 行
```python
def _pid_namespace(pid: int | None = None) -> int:
    """Returns the process's namespace id"""
    pid = pid or os.getpid()
    link = _pid_namespace_link(pid)
    return int(link[link.find("[") + 1 : -1])


def _command_to_string(command: Sequence[str]) -> str:
    return " ".join(command)


class StrobelightCLIFunctionProfiler:
    """
    Note: this is a meta only tool.

    StrobelightCLIFunctionProfiler can be used to profile a python function and
    generate a strobelight link with the results. It works on meta servers but
    does not requires an fbcode target.
    When stop_at_error is false(default), error during profiling does not prevent
    the work function from running.

    Check function_profiler_example.py for an example.
    """
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Key callable entry points in this range include `_pid_namespace`, `_command_to_string`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `_pid_namespace`, `_command_to_string`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 68-91 / 第 68-91 行
```python
    # This lock is used to make sure only one thread is running the profiler at any point.
    _lock = Lock()

    def __init__(
        self,
        *,
        stop_at_error: bool = False,
        max_profile_duration_sec: int = 60 * 10,
        sample_each: float = 1e7,  # sample each sample_each cycles.
        run_user_name: str = "pytorch-strobelight-ondemand",
        timeout_wait_for_running_sec: int = 60,
        timeout_wait_for_finished_sec: int = 60,
        recorded_env_variables: list[str] | None = None,
        sample_tags: list[str] | None = None,
        stack_max_len: int = 127,
        async_stack_max_len: int = 127,
    ) -> None:
        self.stop_at_error = stop_at_error
        self.max_profile_duration_sec = max_profile_duration_sec
        self.sample_each = sample_each
        self.run_user_name = run_user_name
        self.timeout_wait_for_running_sec = timeout_wait_for_running_sec
        self.timeout_wait_for_finished_sec = timeout_wait_for_finished_sec
        # Results of the most recent run.
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。

### Lines 92-113 / 第 92-113 行
```python
        # Tracks the strobelight run id of the most recent run
        self.current_run_id: int | None = None
        self.sample_tags = sample_tags

    def _run_async(self) -> None:
        processId = os.getpid()
        namespace = _pid_namespace(processId)
        command = [
            "strobeclient",
            "run",
            "--profiler",
            "pyperf",
            "--event",
            "cycles",
            "--async",
            "--sample-interval",
            f"{int(self.sample_each)}",
            "--duration-ms",
            f"{int(self.max_profile_duration_sec * 1000)}",
            "--pid",
            f"{namespace}:{processId}",
        ]
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。

### Lines 115-135 / 第 115-135 行
```python
        if self.sample_tags:
            command.append("--sample-tags")
            command.append(",".join(self.sample_tags))

        logger.debug("running command: %s", _command_to_string(command))
        result = subprocess.run(command, capture_output=True)
        output = result.stderr.decode("utf-8")
        logger.debug("output:\n{%s}", output)

        if result.returncode != 0:
            raise StrobelightCLIProfilerError(
                f"failed to start strobelight profiling, error in run_async:{output}"
            )

        if match := re.search(r"INFO Run Id: (-?\d+)", output):
            self.current_run_id = int(match.group(1))
            return

        raise StrobelightCLIProfilerError(
            f"failed to start strobelight profiling, unexpected result {output}"
        )
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 137-152 / 第 137-152 行
```python
    def _wait_for_running(self, counter: int = 0) -> None:
        if counter > 20:
            raise StrobelightCLIProfilerError(
                "wait_for_running called more than 20 times"
            )

        command = ["strobeclient", "getRunStatus", "--run-id", f"{self.current_run_id}"]
        logger.debug("running command: %s", _command_to_string(command))
        result = subprocess.run(command, capture_output=True)
        output = result.stderr.decode("utf-8")
        logger.debug("output:\n{%s}", output)

        if result.returncode != 0:
            raise StrobelightCLIProfilerError(
                f"failed to start strobelight profiling, error in wait_for_running:{output}"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 154-172 / 第 154-172 行
```python
        if match := re.search("Profile run status: (.*)", output):
            current_status = match.group(1)
            if current_status == "RUNNING":
                return
            elif current_status == "PREPARING":
                time.sleep(10)
                self._wait_for_running(counter + 1)
                return
            else:
                raise StrobelightCLIProfilerError(f"unexpected {current_status} phase")

        raise StrobelightCLIProfilerError(f"unexpected output\n: {output} ")

    def _stop_run(self) -> None:
        command = ["strobeclient", "stopRun", "--run-id", str(self.current_run_id)]
        logger.debug("running command: %s", _command_to_string(command))
        result = subprocess.run(command, capture_output=True)
        output = result.stderr.decode("utf-8")
        logger.debug("output:\n{%s}", output)
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 174-195 / 第 174-195 行
```python
        if result.returncode != 0:
            raise StrobelightCLIProfilerError(
                f"failed to stop strobelight profiling, return code is not 0 :{output}"
            )

        if match := re.search("INFO ::1:(.*)", output):
            current_status = match.group(1)
            if current_status.__contains__("Success!"):
                return
            else:
                raise StrobelightCLIProfilerError(
                    f"failed to stop strobelight profiling, got {current_status} result"
                )

        raise StrobelightCLIProfilerError(f"unexpected output\n: {output} ")

    def _get_results(self) -> None:
        command = ["strobeclient", "getRunStatus", "--run-id", str(self.current_run_id)]
        logger.debug("running command: %s", _command_to_string(command))
        result = subprocess.run(command, capture_output=True)
        output = result.stderr.decode("utf-8")
        logger.debug("output:\n{%s}", output)
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 197-217 / 第 197-217 行
```python
        if result.returncode != 0:
            raise StrobelightCLIProfilerError(
                f"failed to extract profiling results, return code is not 0 : {output}"
            )

        if match := re.search("INFO ::1:(.*)", output):
            current_status = match.group(1)
            if current_status.__contains__("Profile run status: PROCESSING"):
                time.sleep(10)
                self._get_results()
                return
            elif not current_status.__contains__("Profile run finished with SUCCESS"):
                raise StrobelightCLIProfilerError(
                    f"failed to extract profiling results, unexpected response {output}"
                )

        for item in re.findall(
            r"(Total samples(.*)|GraphProfiler(.*)|Icicle view \(python stack\)(.*))",
            output,
        ):
            logger.info(item[0])
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 219-235 / 第 219-235 行
```python
    def _stop_strobelight_no_throw(
        self,
        collect_results: bool,
    ) -> None:
        try:
            # call stop run
            self._stop_run()
            logger.info("strobelight profiling stopped")

            logger.debug("collection stopped")

            if not collect_results:
                return

            self._get_results()
        except Exception:
            logger.warning("error during stop_strobelight", exc_info=True)
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 237-257 / 第 237-257 行
```python
    # Return true if strobelight started and is running. Never throw.
    def _start_strobelight(self) -> bool:
        strobelight_started = False
        try:
            self._run_async()
            strobelight_started = True
            logger.info("strobelight run id is: %s", self.current_run_id)
            self._wait_for_running()
            logger.info("strobelight profiling running")
            return True

        except Exception:
            logger.warning("error during start_strobelight:", exc_info=True)
            if strobelight_started:
                self._stop_strobelight_no_throw(collect_results=False)
            return False

    def profile(
        self, work_function: Callable[_P, _R], *args: _P.args, **kwargs: _P.kwargs
    ) -> _R | None:
        self.current_run_id = None
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 259-276 / 第 259-276 行
```python
        if locked := StrobelightCLIFunctionProfiler._lock.acquire(False):
            if not locked:
                if self.stop_at_error:
                    raise StrobelightCLIProfilerError("concurrent runs not supported")

                logger.warning("concurrent runs not supported")
                return work_function(*args, **kwargs)

            started = self._start_strobelight()
            if not started:
                if self.stop_at_error:
                    StrobelightCLIFunctionProfiler._lock.release()
                    raise StrobelightCLIProfilerError(
                        "failed to start strobelight profiling"
                    )
                result = work_function(*args, **kwargs)
                StrobelightCLIFunctionProfiler._lock.release()
                return result
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 278-301 / 第 278-301 行
```python
            try:
                logger.debug("collection started")
                result = work_function(*args, **kwargs)
                self._stop_strobelight_no_throw(collect_results=True)
                StrobelightCLIFunctionProfiler._lock.release()
                return result
            except Exception as error:
                logger.warning("work function throw exception", exc_info=True)
                self._stop_strobelight_no_throw(collect_results=False)
                StrobelightCLIFunctionProfiler._lock.release()
                raise error
        return None


# A function decorator that wraps profile, if no profiler is provided one with
# default args is created. A function can be annotated as:
# @strobelight()
# @strobelight(profiler = StrobelightFunctionProfiler(stop_at_error=True,..))
# @strobelight(stop_at_error=True,...)
def strobelight(
    profiler: StrobelightCLIFunctionProfiler | None = None, **kwargs: Any
) -> Callable[[Callable[_P, _R]], Callable[_P, _R | None]]:
    if not profiler:
        profiler = StrobelightCLIFunctionProfiler(**kwargs)
```
- **EN**: It introduces or extends class-level abstractions such as `StrobelightCLIFunctionProfiler`, which organize state and behavior for this subsystem. Key callable entry points in this range include `strobelight`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `StrobelightCLIFunctionProfiler` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `strobelight`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 303-313 / 第 303-313 行
```python
    def strobelight_inner(
        work_function: Callable[_P, _R],
    ) -> Callable[_P, _R | None]:
        @functools.wraps(work_function)
        def wrapper_function(*args: _P.args, **kwargs: _P.kwargs) -> _R | None:
            # pyrefly: ignore [bad-argument-type]
            return profiler.profile(work_function, *args, **kwargs)

        return wrapper_function

    return strobelight_inner
```
- **EN**: Key callable entry points in this range include `strobelight`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `strobelight`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **TensorBoard integration**
  - EN: Exports model, scalar, and graph information into TensorBoard-consumable formats.
  - CN: 把模型、标量与图信息导出为 TensorBoard 可消费的格式。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `functools`, `logging`, `os`, `re`, `subprocess`, `time`, `collections.abc:Callable`, `collections.abc:Sequence`, `threading:Lock`, `typing:Any`, `typing:TypeVar`
- **Third-party packages / 第三方包**: `typing_extensions:ParamSpec`
- **Primary symbols / 核心符号**: `StrobelightCLIProfilerError`, `StrobelightCLIFunctionProfiler`, `_pid_namespace_link`, `_pid_namespace`, `_command_to_string`, `strobelight`
