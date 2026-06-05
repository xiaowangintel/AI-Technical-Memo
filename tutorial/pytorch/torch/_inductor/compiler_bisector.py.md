# compiler_bisector.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/compiler_bisector.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `Subsystem`, `BisectSubsystem`, `BinarySubsystem`, `ConfigChange`, `BisectionResult`, and `CompilerBisector`. It exposes functions such as `reset_counters`, `get_env_val`, `command_line_usage`, and `get_is_bisection_enabled`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `Subsystem`、`BisectSubsystem`、`BinarySubsystem`、`ConfigChange`、`BisectionResult`、`CompilerBisector` 等类。同时提供 `reset_counters`、`get_env_val`、`command_line_usage`、`get_is_bisection_enabled` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import atexit
import collections
import dataclasses
import functools
import os
import shutil
import sys
import tempfile
from collections.abc import Callable
from dataclasses import dataclass, field

from torch._inductor.runtime.cache_dir_utils import cache_dir


# Set the subdirectory name
SUBDIR_NAME = "bisect"


@dataclass
class Subsystem:
````
- **EN**: Imports dependencies such as `atexit`, `collections`, `dataclasses`, `functools`, `os`, `shutil`, and `...+4` for the logic in this range. Introduces class `Subsystem`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `atexit`、`collections`、`dataclasses`、`functools`、`os`、`shutil`、`另有4项` 等依赖，为后续逻辑提供基础能力。这里定义了类`Subsystem`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 21-40 / 第 21-40 行
````python
    name: str


@dataclass
class BisectSubsystem(Subsystem):
    pass


@dataclass
class BinarySubsystem(Subsystem):
    pass


@dataclass
class ConfigChange(BinarySubsystem):
    name: str = field(init=False)
    config_name: str
    config_field: str
    config_value: object

````
- **EN**: Introduces class `BisectSubsystem`, class `BinarySubsystem`, class `ConfigChange`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `name`, `config_name`, `config_field`, and `config_value`.
- **CN**: 这里定义了类`BisectSubsystem`、类`BinarySubsystem`、类`ConfigChange`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `name`、`config_name`、`config_field`、`config_value` 等值。

### Lines 41-60 / 第 41-60 行
````python
    def __post_init__(self) -> None:
        self.name = f"{self.config_name}_{self.config_field}"


# Dictionary of backend -> subsystems
BACKENDS: dict[str, list[Subsystem]] = {
    # run dynamo without aot_autograd
    "eager": [],
    # run dynamo with aot_autograd, but no partitioner or decomps
    "aot_eager": [],
    # run dynamo with aot autograd, decompositions and partitioner
    "aot_eager_decomp_partition": [
        ConfigChange("aot_eager_decomp_partition", "cse", False),
        BisectSubsystem(
            "decomposition"
        ),  # number of decompositions we apply in tracing
    ],  # TODO - add cse ?
    # applies CrossRefFakeMode on invocation
    "aot_eager_decomp_partition_crossref": [],
    "inductor": [
````
- **EN**: Introduces function `__post_init__`. Initializes or updates values such as `BACKENDS`.
- **CN**: 这里定义了函数`__post_init__`。初始化或更新了 `BACKENDS` 等值。

### Lines 61-80 / 第 61-80 行
````python
        BisectSubsystem("pre_grad_passes"),  # passes applied on pre-grad IR
        BisectSubsystem("joint_graph_passes"),  # passes applied on joint graph
        BisectSubsystem(
            "post_grad_passes"
        ),  # passes applied individually on forward, and backward in inductor
        ConfigChange("inductor", "fallback_random", True),
        ConfigChange("inductor", "emulate_precision_casts", True),
        ConfigChange("inductor", "layout_optimization", False),
        ConfigChange("inductor", "comprehensive_padding", False),
        BisectSubsystem("cudagraphs"),  # cudagraph wrapping of compiled graphs
        BisectSubsystem("lowerings"),  # lowering aten operators to inductor
    ],  # TODO - add more - fusions ?
}

subsystem_call_counter: dict[str, int] = collections.Counter()
call_counter_debug_info: dict[int, str] = {}


def reset_counters() -> None:
    subsystem_call_counter.clear()
````
- **EN**: Introduces function `reset_counters`. Initializes or updates values such as `subsystem_call_counter`, and `call_counter_debug_info`.
- **CN**: 这里定义了函数`reset_counters`。初始化或更新了 `subsystem_call_counter`、`call_counter_debug_info` 等值。

### Lines 81-100 / 第 81-100 行
````python
    call_counter_debug_info.clear()


@functools.cache
def get_env_val(env_str: str) -> str | None:
    return os.environ.get(env_str, None)


@dataclasses.dataclass
class BisectionResult:
    """
    backend: torch.compile backend responsible for failure
    subsystem: optional, registered component identified for failure
    bisect_number: optional, number of times the subsystem needed to be applied to trigger failure
    debug_info: associated info of the triggering bisect application of subsystem
    """

    backend: str
    subsystem: str | None = None
    bisect_number: int | None = None
````
- **EN**: Introduces function `get_env_val`, class `BisectionResult`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里定义了函数`get_env_val`、类`BisectionResult`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 101-120 / 第 101-120 行
````python
    debug_info: str | None = None


class CompilerBisector:
    """
    This class iteratively runs torch.compile backends (eager, aot_eager, inductor) to find the
    first backend that can repro an issue.

    Once it discovers the offending backend it will iteratively disable subsystems within the backend.
    For subsystems which are applied repeatedly, such as the number of post grad passes or number
    of lowering of nodes to inductor ir, it will bisect to find the offending application.

    The idiomatic way to run it is with `do_bisect`. You can also use it by setting the env flags
    `TORCH_BISECT_BACKEND`, `TORCH_BISECT_SUBSYSTEM` and `TORCH_BISECT_MAX`.

    It also supports a CLI interface:

        python -m torch._inductor.compiler_bisector start
        python -m torch._inductor.compiler_bisector good
        python -m torch._inductor.compiler_bisector bad
````
- **EN**: Introduces class `CompilerBisector`. Initializes or updates values such as `debug_info`.
- **CN**: 这里定义了类`CompilerBisector`。初始化或更新了 `debug_info` 等值。

### Lines 121-140 / 第 121-140 行
````python
        python -m torch._inductor.compiler_bisector end

    Or use `run` to automatically bisect by running a command repeatedly:

        python -m torch._inductor.compiler_bisector run <command>

    The command's exit code determines the result: 0 = good, non-zero = bad.
    The TORCH_COMPILE_BACKEND env var is set to the backend being tested.
    """

    bisection_enabled: bool = False

    in_process_cache: str | None = None

    @classmethod
    def get_dir(cls) -> str:
        return f"{cache_dir() if not cls.in_process_cache else cls.in_process_cache}/{SUBDIR_NAME}"

    @classmethod
    def write_lines_to_file(cls, file_path: str, lines: list[str]) -> None:
````
- **EN**: Introduces function `get_dir`, function `write_lines_to_file`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_dir`、函数`write_lines_to_file`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-160 / 第 141-160 行
````python
        os.makedirs(os.path.dirname(file_path), exist_ok=True)
        with open(file_path, "w") as file:
            file.writelines(lines)

    @classmethod
    def read_lines_from_file(cls, file_path: str) -> list[str]:
        if os.path.exists(file_path):
            with open(file_path) as file:
                return file.readlines()
        return []

    @classmethod
    def update_run_state(
        cls, backend_name: str, subsystem: Subsystem, run_state: str
    ) -> None:
        file_path = os.path.join(
            cls.get_dir(), backend_name, f"{subsystem.name}_run_state.txt"
        )
        if isinstance(subsystem, ConfigChange):
            assert run_state == "test_disable"
````
- **EN**: Introduces function `read_lines_from_file`, function `update_run_state`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`read_lines_from_file`、函数`update_run_state`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 161-180 / 第 161-180 行
````python
            cls.set_config_values(
                backend_name,
                subsystem.name,
                {subsystem.config_field: subsystem.config_value},
            )

        cls.write_lines_to_file(file_path, [run_state])

    @classmethod
    def set_config_values(
        cls, backend: str, subsystem: str, config_data: dict[str, object]
    ) -> None:
        file_path = os.path.join(cls.get_dir(), backend, f"{subsystem}_config.txt")
        lines = [f"{k}={v}\n" for k, v in config_data.items()]
        cls.write_lines_to_file(file_path, lines)

    @classmethod
    def update_bisect_status(cls, backend_name: str, subsystem_name: str) -> None:
        assert isinstance(subsystem_name, str)
        file_path = os.path.join(cls.get_dir(), "bisect_status.txt")
````
- **EN**: Introduces function `set_config_values`, function `update_bisect_status`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`set_config_values`、函数`update_bisect_status`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 181-200 / 第 181-200 行
````python
        lines = [f"backend={backend_name}\n", f"subsystem={subsystem_name}\n"]
        cls.write_lines_to_file(file_path, lines)

    @classmethod
    def update_bisect_range(
        cls, backend_name: str, subsystem_name: str, low: int, high: int
    ) -> None:
        assert isinstance(subsystem_name, str)
        file_path = os.path.join(
            cls.get_dir(), backend_name, f"{subsystem_name}_bisect_range.txt"
        )
        lines = [f"low={low}\n", f"high={high}\n"]
        cls.write_lines_to_file(file_path, lines)

    @classmethod
    def get_backend(cls) -> str | None:
        """
        Returns the active backend, if any
        """
        if val := get_env_val("TORCH_BISECT_BACKEND"):
````
- **EN**: Introduces function `update_bisect_range`, function `get_backend`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`update_bisect_range`、函数`get_backend`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 201-220 / 第 201-220 行
````python
            return val

        file_path = os.path.join(cls.get_dir(), "bisect_status.txt")
        lines = cls.read_lines_from_file(file_path)
        for line in lines:
            if line.startswith("backend="):
                return line.strip().split("=")[1]
        return None

    @classmethod
    def get_subsystem(cls) -> str | None:
        """
        Returns the active subsystem, if any
        """

        if val := get_env_val("TORCH_BISECT_SUBSYSTEM"):
            return val

        file_path = os.path.join(cls.get_dir(), "bisect_status.txt")
        lines = cls.read_lines_from_file(file_path)
````
- **EN**: Introduces function `get_subsystem`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_subsystem`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python
        for line in lines:
            if line.startswith("subsystem="):
                out = line.strip().split("=")[1]
                return out if out else None
        return None

    @classmethod
    def get_subsystem_object(cls, backend_name: str, subsystem_name: str) -> Subsystem:
        return next(obj for obj in BACKENDS[backend_name] if obj.name == subsystem_name)

    @classmethod
    def get_run_state(cls, backend_name: str, subsystem_name: str) -> str | None:
        """
        Returns the current stage of bisecting, if Any
        """

        file_path = os.path.join(
            cls.get_dir(), backend_name, f"{subsystem_name}_run_state.txt"
        )
        lines = cls.read_lines_from_file(file_path)
````
- **EN**: Introduces function `get_subsystem_object`, function `get_run_state`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_subsystem_object`、函数`get_run_state`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 241-260 / 第 241-260 行
````python
        if lines:
            out = lines[0].strip()
            assert out in ("test_disable", "find_max_bounds", "bisect")
            return out
        return None

    @classmethod
    def get_bisect_range(
        cls, backend_name: str, subsystem_name: str
    ) -> tuple[int, int]:
        file_path = os.path.join(
            cls.get_dir(), backend_name, f"{subsystem_name}_bisect_range.txt"
        )
        lines = cls.read_lines_from_file(file_path)
        low = None
        high = None
        # pyrefly: ignore [bad-assignment]
        for line in reversed(lines):
            if line.startswith("low="):
                low = int(line.strip().split("=")[1])
````
- **EN**: Introduces function `get_bisect_range`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_bisect_range`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 261-280 / 第 261-280 行
````python
            elif line.startswith("high="):
                high = int(line.strip().split("=")[1])

            if low is not None and high is not None:
                break

        if low is None or high is None:
            raise RuntimeError(
                f"Trying to get bisect range when it is not set: subsystem {subsystem_name}"
            )

        return low, high

    @classmethod
    def update_config_change(cls, backend: str, subsystem: ConfigChange) -> None:
        file_path = os.path.join(cls.get_dir(), backend, f"{subsystem.name}_config.txt")
        lines = [
            f"config_name={subsystem.config_name}\n",
            f"config_field={subsystem.config_field}\n",
            f"config_value={subsystem.config_value}\n",
````
- **EN**: Introduces function `update_config_change`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`update_config_change`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-300 / 第 281-300 行
````python
        ]
        cls.write_lines_to_file(file_path, lines)

    @classmethod
    def get_config_change(cls, config_name: str) -> dict[str, object] | None:
        backend = cls.get_backend()
        subsystem = cls.get_subsystem()

        if not backend or not subsystem:
            return None

        file_path = os.path.join(cls.get_dir(), backend, f"{subsystem}_config.txt")

        if not os.path.exists(file_path):
            return None

        lines = cls.read_lines_from_file(file_path)
        config_data = {}
        for line in lines:
            key, value = line.strip().split("=", 1)
````
- **EN**: Introduces function `get_config_change`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_config_change`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 301-320 / 第 301-320 行
````python
            config_data[key] = eval(value)

        return config_data

    @classmethod
    def delete_bisect_status(cls) -> None:
        # in process_cache we have created if it exists, just the subdirectory of non created dir
        dir_name = cls.in_process_cache if cls.in_process_cache else cls.get_dir()
        if os.path.exists(dir_name):
            shutil.rmtree(dir_name)
            print("Bisection status deleted.")
        else:
            print("No bisection status found.")

    @classmethod
    def get_system_counter(cls, name: str, increment: bool = True) -> int:
        global subsystem_call_counter
        curr = subsystem_call_counter[name]
        if increment:
            subsystem_call_counter[name] += 1
````
- **EN**: Introduces function `delete_bisect_status`, function `get_system_counter`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里定义了函数`delete_bisect_status`、函数`get_system_counter`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 321-340 / 第 321-340 行
````python
        return curr

    @classmethod
    def disable_subsystem(
        cls,
        backend: str,
        subsystem: str,
        debug_info: Callable[[], str] | None = None,
    ) -> bool:
        if not cls.bisection_enabled:
            return False

        if cls.get_backend() != backend:
            return False

        if cls.get_subsystem() != subsystem:
            return False

        if val := get_env_val("TORCH_BISECT_MAX"):
            counter = cls.get_system_counter(subsystem, increment=True)
````
- **EN**: Introduces function `disable_subsystem`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`disable_subsystem`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 341-360 / 第 341-360 行
````python
            return counter > int(val)

        run_state = cls.get_run_state(backend, subsystem)
        if run_state == "test_disable":
            # First run, disable completely
            return True
        elif run_state == "find_max_bounds":
            # Second run, update bisection range and return True to enable the subsystem
            cls.update_bisect_range(
                backend,
                subsystem,
                0,
                cls.get_system_counter(subsystem, increment=True),
            )
            return False
        else:
            assert run_state == "bisect"
            # If the environment variable is not set, use the bisection range midpoint
            low, high = cls.get_bisect_range(backend, subsystem)
            # if high - low <= 2:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `run_state`, and `else`. This range continues the implementation of function `CompilerBisector.disable_subsystem`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `run_state`、`else` 等值。这一段延续了函数`CompilerBisector.disable_subsystem` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
            midpoint = (low + high) // 2
            call_counter = cls.get_system_counter(subsystem)

            if (
                call_counter >= low
                and call_counter <= high
                and (low - high) <= 2
                and debug_info is not None
            ):
                call_counter_debug_info[call_counter] = debug_info()

            return call_counter > midpoint

    @classmethod
    def advance_subsystem(
        cls, curr_backend: str, curr_subsystem: Subsystem
    ) -> Subsystem | None:
        """
        Tries to move to the next subsystem within the current system.
        """
````
- **EN**: Introduces function `advance_subsystem`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`advance_subsystem`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 381-400 / 第 381-400 行
````python
        print(f"Disabling {curr_subsystem.name} did not fix the issue.")

        current_subsystems = BACKENDS[curr_backend]
        current_subsystem_index = next(
            i
            for i, subsystem in enumerate(current_subsystems)
            if subsystem.name == curr_subsystem.name
        )

        if current_subsystem_index < len(current_subsystems) - 1:
            next_subsystem = current_subsystems[current_subsystem_index + 1]
            cls.update_bisect_status(curr_backend, next_subsystem.name)
            cls.update_run_state(curr_backend, next_subsystem, "test_disable")
            print(
                f"Moving to the next subsystem: {curr_backend} - {next_subsystem.name}"
            )
            return next_subsystem
        else:
            print(
                f"All subsystems in {curr_backend} have been checked. The issue is not in this system."
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `current_subsystems`, `current_subsystem_index`, `next_subsystem`, and `else`. This range continues the implementation of function `CompilerBisector.advance_subsystem`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `current_subsystems`、`current_subsystem_index`、`next_subsystem`、`else` 等值。这一段延续了函数`CompilerBisector.advance_subsystem` 的具体实现。

### Lines 401-420 / 第 401-420 行
````python
            )
            return None

    @classmethod
    def advance_backend(cls, curr_backend: str) -> str | None:
        """
        Tries Move to the next backend.
        """
        current_system_index = list(BACKENDS.keys()).index(curr_backend)

        if current_system_index < len(BACKENDS) - 1:
            curr_backend = list(BACKENDS.keys())[current_system_index + 1]
            cls.update_bisect_status(curr_backend, "")
            print(f"Moving to the next system: {curr_backend}")
            return curr_backend
        else:
            return None

    @classmethod
    def process_subsystem(
````
- **EN**: Introduces function `advance_backend`, function `process_subsystem`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`advance_backend`、函数`process_subsystem`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 421-440 / 第 421-440 行
````python
        cls,
        curr_backend: str,
        curr_subsystem: Subsystem,
        fn: Callable[[], bool],
        cli_interface: bool = True,
    ) -> bool:
        """
        Process the current subsystem. Returns True if the issue is found, False otherwise.
        """
        assert isinstance(curr_subsystem, Subsystem)
        while True:
            run_state = cls.get_run_state(curr_backend, curr_subsystem.name)
            reset_counters()
            if run_state == "test_disable":
                if not fn():
                    next_subsystem = cls.advance_subsystem(curr_backend, curr_subsystem)
                    if not next_subsystem:
                        return False
                    curr_subsystem = next_subsystem
                else:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `curr_backend`, `curr_subsystem`, `fn`, `cli_interface`, `run_state`, `next_subsystem`, and `...+1`. This range continues the implementation of function `CompilerBisector.process_subsystem`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `curr_backend`、`curr_subsystem`、`fn`、`cli_interface`、`run_state`、`next_subsystem`、`另有1项` 等值。这一段延续了函数`CompilerBisector.process_subsystem` 的具体实现。

### Lines 441-460 / 第 441-460 行
````python
                    if isinstance(curr_subsystem, ConfigChange):
                        print(
                            f"Setting config {curr_subsystem.config_name} field {curr_subsystem.config_field} "
                            f"to {curr_subsystem.config_value} fixed the issue"
                        )
                    else:
                        print(f"Disabling {curr_subsystem.name} fixed the issue.")
                    if isinstance(curr_subsystem, BinarySubsystem):
                        return True
                    print("Starting bisect by getting upper bound.")
                    cls.update_run_state(
                        curr_backend, curr_subsystem, "find_max_bounds"
                    )
            elif run_state == "find_max_bounds":
                if fn():
                    raise RuntimeError(
                        f"Function succeeded with 'find_max_bounds' status for {curr_backend} - {curr_subsystem.name}."
                    )
                else:
                    _, high = cls.get_bisect_range(curr_backend, curr_subsystem.name)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`. This range continues the implementation of function `CompilerBisector.process_subsystem`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。这一段延续了函数`CompilerBisector.process_subsystem` 的具体实现。

### Lines 461-480 / 第 461-480 行
````python
                    print(f"Upper bound of {high} found for {curr_backend}.")
                    cls.update_run_state(curr_backend, curr_subsystem, "bisect")
            elif run_state == "bisect":
                low, high = cls.get_bisect_range(curr_backend, curr_subsystem.name)
                midpoint = (low + high) // 2
                print(
                    f"Bisecting {curr_backend} - {curr_subsystem.name} (Range: [{low}, {high}], Midpoint: {midpoint})"
                )
                if fn():
                    cls.update_bisect_range(
                        curr_backend, curr_subsystem.name, midpoint + 1, high
                    )
                else:
                    cls.update_bisect_range(
                        curr_backend, curr_subsystem.name, low, midpoint
                    )
                low, high = cls.get_bisect_range(curr_backend, curr_subsystem.name)
                if low == high:
                    print(
                        f"Binary search completed for {curr_backend} - {curr_subsystem.name}. The bisect number is {low}. "
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `midpoint`, and `else`. This range continues the implementation of function `CompilerBisector.process_subsystem`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `midpoint`、`else` 等值。这一段延续了函数`CompilerBisector.process_subsystem` 的具体实现。

### Lines 481-500 / 第 481-500 行
````python
                        f"Debug info: {call_counter_debug_info.get(low, 'not found')}"
                    )
                    return True
            else:
                raise RuntimeError(f"Unexpected run_state {run_state}")

            if cli_interface:
                sys.exit(0)

    @classmethod
    def initialize_system(cls) -> None:
        curr_backend = next(iter(BACKENDS.keys()))
        curr_subsystem = ""
        cls.update_bisect_status(curr_backend, curr_subsystem)
        print(f"Starting bisection process with system: {curr_backend}")

    @classmethod
    def do_bisect(
        cls, fn: Callable[[], bool], cli_interface: bool = False
    ) -> BisectionResult | None:
````
- **EN**: Introduces function `initialize_system`, function `do_bisect`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`initialize_system`、函数`do_bisect`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 501-520 / 第 501-520 行
````python
        """
        Run fn repeatedly attempting to bisect torch.compile. fn should return True on success and False on failure.
        """

        # TODO graph bisecting is not well composed with lowering
        # bisector so far. Use a config to opt-in
        import torch._inductor.config as inductor_config

        if inductor_config.test_configs.bisect_pre_grad_graph:
            BACKENDS["inductor"].insert(0, BisectSubsystem("pre_grad_graph"))

        if not cli_interface:
            bisection_enabled_orig = cls.bisection_enabled
            cls.delete_bisect_status()
            cls.bisection_enabled = True
            # Only create temp dir if not already set (CLI run mode pre-sets it)
            if not cls.in_process_cache:
                cls.in_process_cache = tempfile.mkdtemp()

            def cleanup() -> None:
````
- **EN**: Imports dependencies such as `torch._inductor.config` for the logic in this range. Introduces function `cleanup`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.config` 等依赖，为后续逻辑提供基础能力。这里定义了函数`cleanup`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 521-540 / 第 521-540 行
````python
                cls.bisection_enabled = bisection_enabled_orig
                cls.delete_bisect_status()
                cls.in_process_cache = None

                if BACKENDS["inductor"][0].name == "pre_grad_graph":
                    del BACKENDS["inductor"][0]

            cleanup_handler = atexit.register(cleanup)

            class DisableBisect:
                def __del__(self) -> None:
                    cleanup()
                    atexit.unregister(cleanup_handler)

            _cleanup = DisableBisect()

        curr_backend = cls.get_backend()
        curr_subsystem_name = cls.get_subsystem()

        if not curr_backend:
````
- **EN**: Introduces class `DisableBisect`, function `__del__`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`DisableBisect`、函数`__del__`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 541-560 / 第 541-560 行
````python
            cls.initialize_system()
            curr_backend = cls.get_backend()
            assert curr_backend is not None
            curr_subsystem_name = cls.get_subsystem()

        curr_subsystem = (
            cls.get_subsystem_object(curr_backend, curr_subsystem_name)
            if curr_subsystem_name is not None
            else None
        )
        while True:
            assert curr_backend is not None
            reset_counters()
            if curr_subsystem:
                result = cls.process_subsystem(
                    curr_backend, curr_subsystem, fn, cli_interface=cli_interface
                )
                if result:
                    curr_subsystem = cls.get_subsystem_object(
                        curr_backend,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `curr_backend`, `curr_subsystem_name`, `curr_subsystem`, and `result`. This range continues the implementation of function `CompilerBisector.do_bisect`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `curr_backend`、`curr_subsystem_name`、`curr_subsystem`、`result` 等值。这一段延续了函数`CompilerBisector.do_bisect` 的具体实现。

### Lines 561-580 / 第 561-580 行
````python
                        cls.get_subsystem(),  # type: ignore[arg-type]
                    )

                    if isinstance(curr_subsystem, BinarySubsystem):
                        return BisectionResult(
                            curr_backend,
                            curr_subsystem.name,
                            0,
                            curr_subsystem.name,
                        )

                    low, _ = cls.get_bisect_range(curr_backend, curr_subsystem.name)
                    return BisectionResult(
                        curr_backend,
                        curr_subsystem.name,
                        low,
                        call_counter_debug_info.get(low),
                    )

                next_subsystem = cls.advance_subsystem(curr_backend, curr_subsystem)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `next_subsystem`. This range continues the implementation of function `CompilerBisector.do_bisect`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `next_subsystem` 等值。这一段延续了函数`CompilerBisector.do_bisect` 的具体实现。

### Lines 581-600 / 第 581-600 行
````python
                if not next_subsystem:
                    print(
                        f"The issue is in the {curr_backend} system, but could not identify subsystem."
                    )
                    assert curr_backend is not None
                    return BisectionResult(curr_backend)

                curr_subsystem = next_subsystem
            else:
                if fn():
                    next_backend = cls.advance_backend(curr_backend)
                    if not next_backend:
                        print("All systems have been checked.")
                        return None

                    curr_backend = next_backend
                else:
                    current_subsystems = BACKENDS[curr_backend]
                    if current_subsystems:
                        curr_subsystem = current_subsystems[0]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `curr_subsystem`, `else`, `next_backend`, `curr_backend`, and `current_subsystems`. This range continues the implementation of function `CompilerBisector.do_bisect`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `curr_subsystem`、`else`、`next_backend`、`curr_backend`、`current_subsystems` 等值。这一段延续了函数`CompilerBisector.do_bisect` 的具体实现。

### Lines 601-620 / 第 601-620 行
````python
                        cls.update_bisect_status(curr_backend, curr_subsystem.name)
                        cls.update_run_state(
                            curr_backend, curr_subsystem, "test_disable"
                        )
                        print(
                            f"The issue is in the {curr_backend} system. Moving to the first subsystem: {curr_subsystem}"
                        )
                    else:
                        print(f"The issue is in the {curr_backend} system.")
                        return BisectionResult(curr_backend)

            if cli_interface:
                sys.exit(0)


HELP_TEXT = """\
Usage: python -m torch._inductor.compiler_bisector <command>

Commands:
  start       Start a new bisection session (manual mode)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `HELP_TEXT`, `Usage`, and `Commands`. This range continues the implementation of function `CompilerBisector.do_bisect`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`HELP_TEXT`、`Usage`、`Commands` 等值。这一段延续了函数`CompilerBisector.do_bisect` 的具体实现。

### Lines 621-640 / 第 621-640 行
````python
  good        Mark the current state as good (no issue)
  bad         Mark the current state as bad (issue present)
  end         End the bisection session and clean up
  run <cmd>   Automatically bisect by running a command repeatedly
              Exit code 0 = good, non-zero = bad

Example - using 'run' for automatic bisection:

  1. Create a test script (e.g., test_bug.py):

      import os, sys, torch

      def main():
          torch._dynamo.reset()
          # Use the backend the bisector is testing
          backend = os.environ.get("TORCH_COMPILE_BACKEND", "inductor")

          @torch.compile(backend=backend)
          def fn(x):
              return x.sin()  # or your failing operation
````
- **EN**: Imports dependencies such as `os`, `sys`, and `torch` for the logic in this range. Introduces function `main`, function `fn`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `os`、`sys`、`torch` 等依赖，为后续逻辑提供基础能力。这里定义了函数`main`、函数`fn`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 641-660 / 第 641-660 行
````python

          x = torch.randn(10)
          compiled = fn(x)
          expected = x.sin()  # eager reference

          if torch.allclose(compiled, expected, rtol=1e-4, atol=1e-4):
              return 0  # PASS
          else:
              print(f"FAIL: got {compiled}, expected {expected}")
              return 1  # FAIL

      if __name__ == "__main__":
          sys.exit(main())

  2. Run the bisector:

      python -m torch._inductor.compiler_bisector run python test_bug.py

  3. The bisector will:
      - Test backends (eager, aot_eager, inductor) to find which fails
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `x`, `compiled`, `expected`, and `else`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `x`、`compiled`、`expected`、`else` 等值。

### Lines 661-680 / 第 661-680 行
````python
      - Test subsystems within that backend (passes, lowerings, etc.)
      - Binary search to find the exact operation causing the issue

  Output:
      Disabling lowerings fixed the issue.
      Binary search completed for inductor - lowerings. The bisect number is 42.
      Bisection complete: BisectionResult(backend='inductor', subsystem='lowerings', ...)

Environment variables set for your test script:
  TORCH_COMPILE_BACKEND  - The backend being tested (use this in torch.compile)
"""


def command_line_usage() -> None:
    """Entry point for the compiler bisector command-line interface."""
    if len(sys.argv) < 2:
        print(HELP_TEXT)
        sys.exit(1)

    bisection_manager = CompilerBisector()
````
- **EN**: Introduces function `command_line_usage`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Output`, and `bisection_manager`.
- **CN**: 这里定义了函数`command_line_usage`。包含分支、循环或上下文管理等控制流。初始化或更新了 `Output`、`bisection_manager` 等值。

### Lines 681-700 / 第 681-700 行
````python
    command = sys.argv[1]

    if command == "end":
        bisection_manager.delete_bisect_status()
        sys.exit(0)

    if command == "start":
        bisection_manager.delete_bisect_status()
        bisection_manager.initialize_system()
        sys.exit(0)

    if command == "run":
        if len(sys.argv) < 3:
            print(
                "Usage: python -m torch._inductor.compiler_bisector run <command> [args...]"
            )
            sys.exit(1)

        import subprocess

````
- **EN**: Imports dependencies such as `subprocess` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `command`. This range continues the implementation of function `command_line_usage`.
- **CN**: 这里导入了 `subprocess` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `command` 等值。这一段延续了函数`command_line_usage` 的具体实现。

### Lines 701-720 / 第 701-720 行
````python
        run_cmd = sys.argv[2:]

        def test_function() -> bool:
            # Pass bisection state to subprocess via environment variables
            env = os.environ.copy()
            backend = bisection_manager.get_backend()
            subsystem = bisection_manager.get_subsystem()

            if backend:
                # For test script to select the right backend
                env["TORCH_COMPILE_BACKEND"] = backend
                # For bisector in subprocess to know which backend we're testing
                env["TORCH_BISECT_BACKEND"] = backend

            if subsystem:
                assert backend is not None  # subsystem requires a backend
                env["TORCH_BISECT_SUBSYSTEM"] = subsystem
                # Get run_state to determine TORCH_BISECT_MAX
                run_state = bisection_manager.get_run_state(backend, subsystem)
                if run_state == "test_disable":
````
- **EN**: Introduces function `test_function`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `run_cmd`, `env`, `backend`, `subsystem`, and `run_state`.
- **CN**: 这里定义了函数`test_function`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `run_cmd`、`env`、`backend`、`subsystem`、`run_state` 等值。

### Lines 721-740 / 第 721-740 行
````python
                    # -1 means always disable (counter > -1 is always True)
                    env["TORCH_BISECT_MAX"] = "-1"
                # For find_max_bounds and bisect, let the subprocess use file-based
                # mechanisms. The subprocess reads run_state and bisect_range from
                # files, and writes the actual count during find_max_bounds.

            result = subprocess.run(run_cmd, env=env)
            return result.returncode == 0

        bisection_manager.delete_bisect_status()
        bisection_manager.bisection_enabled = True
        # Use shared cache_dir instead of temp dir so subprocesses can access files
        CompilerBisector.in_process_cache = cache_dir()
        result = bisection_manager.do_bisect(test_function, cli_interface=False)
        if result:
            print(f"\nBisection complete: {result}")
        else:
            print("\nBisection complete: no issue found")
        sys.exit(0)

````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `result`, and `else`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `result`、`else` 等值。

### Lines 741-760 / 第 741-760 行
````python
    if command not in ["good", "bad"]:
        print(f"Invalid command: {command}")
        print("Must be 'good', 'bad', 'start', 'end', or 'run'.")
        sys.exit(1)

    def test_function() -> bool:
        return command == "good"

    if not bisection_manager.get_backend():
        raise ValueError("Must call start prior to good or bad")

    bisection_manager.do_bisect(test_function, cli_interface=True)


def get_is_bisection_enabled() -> bool:
    return (
        CompilerBisector.get_subsystem() is not None
        or CompilerBisector.get_backend() is not None
    )

````
- **EN**: Introduces function `test_function`, function `get_is_bisection_enabled`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`test_function`、函数`get_is_bisection_enabled`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 761-765 / 第 761-765 行
````python

CompilerBisector.bisection_enabled = get_is_bisection_enabled()

if __name__ == "__main__":
    command_line_usage()
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 包含分支、循环或上下文管理等控制流。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Primary classes: `Subsystem`, `BisectSubsystem`, `BinarySubsystem`, `ConfigChange`, `BisectionResult`, and `CompilerBisector`  
  **CN**: 主要类：`Subsystem`、`BisectSubsystem`、`BinarySubsystem`、`ConfigChange`、`BisectionResult`、`CompilerBisector`
- **EN**: Primary functions: `reset_counters`, `get_env_val`, `command_line_usage`, and `get_is_bisection_enabled`  
  **CN**: 主要函数：`reset_counters`、`get_env_val`、`command_line_usage`、`get_is_bisection_enabled`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `atexit`, `collections`, `dataclasses`, `functools`, `os`, `shutil`, `sys`, `tempfile`, `collections.abc`, `subprocess`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.runtime.cache_dir_utils`, `torch._inductor.config`
