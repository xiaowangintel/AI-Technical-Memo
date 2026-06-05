# wrapper_benchmark.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/wrapper_benchmark.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `BenchmarkCallableType`, and `ProfileEvent`. It exposes functions such as `get_kernel_category_by_source_code`, `get_kernel_category`, `get_triton_kernel`, `benchmark_all_kernels`, `parse_profile_event_list`, `perf_profile`, and `...+3`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `BenchmarkCallableType`、`ProfileEvent` 等类。同时提供 `get_kernel_category_by_source_code`、`get_kernel_category`、`get_triton_kernel`、`benchmark_all_kernels`、`parse_profile_event_list`、`perf_profile`、`另有3项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import argparse
import datetime
import tempfile
from collections import defaultdict
from dataclasses import dataclass
from types import ModuleType
from typing import Any, Protocol

import torch
from torch.autograd import DeviceType
from torch.utils._ordered_set import OrderedSet

from .runtime.benchmarking import benchmarker
from .runtime.runtime_utils import create_bandwidth_info_str, get_num_bytes


class BenchmarkCallableType(Protocol):
    def __call__(self, times: int, repeat: int) -> float: ...


````
- **EN**: Imports dependencies such as `argparse`, `datetime`, `tempfile`, `collections`, `dataclasses`, `types`, and `...+6` for the logic in this range. Introduces class `BenchmarkCallableType`, function `__call__`.
- **CN**: 这里导入了 `argparse`、`datetime`、`tempfile`、`collections`、`dataclasses`、`types`、`另有6项` 等依赖，为后续逻辑提供基础能力。这里定义了类`BenchmarkCallableType`、函数`__call__`。

### Lines 21-40 / 第 21-40 行
````python
_kernel_category_choices = [
    "foreach",
    "persistent_reduction",
    "pointwise",
    "reduction",
    "split_scan",
    "template",
]


def get_kernel_category_by_source_code(src_code: str) -> str:
    """
    Similar to get_kernel_category but use the source code. Call this API
    if we have not compile the src_code to module yet.
    """
    choices = [
        ch for ch in _kernel_category_choices if f"@triton_heuristics.{ch}" in src_code
    ]
    if len(choices) == 1:
        return choices[0]
````
- **EN**: Introduces function `get_kernel_category_by_source_code`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_kernel_category_choices`, and `choices`.
- **CN**: 这里定义了函数`get_kernel_category_by_source_code`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_kernel_category_choices`、`choices` 等值。

### Lines 41-60 / 第 41-60 行
````python
    else:
        return "unknown"


def get_kernel_category(kernel_mod: ModuleType) -> str:
    """
    Given the module defining a triton kernel, return the category of the kernel.
    Category can be one of:
    - pointwise
    - reduction
    - persistent_reduction

    Currently we simply decide the category depending on what decorator is imported
    by the kernel.
    """
    choices = [ch for ch in _kernel_category_choices if ch in kernel_mod.__dict__]
    if len(choices) == 1:
        return choices[0]
    else:
        return "unknown"
````
- **EN**: Introduces function `get_kernel_category`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_kernel_category`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 61-80 / 第 61-80 行
````python


def get_triton_kernel(mod: ModuleType):  # type: ignore[no-untyped-def]
    from torch._inductor.runtime.triton_heuristics import CachingAutotuner

    cand_list = [
        v
        for k, v in mod.__dict__.items()
        if k.startswith("triton_") and isinstance(v, CachingAutotuner)
    ]
    assert len(cand_list) == 1
    return cand_list[0]


def benchmark_all_kernels(
    benchmark_name: str, benchmark_all_configs: dict[Any, Any] | None
) -> None:
    """
    An experimental API used only when config.benchmark_kernel is true.

````
- **EN**: Imports dependencies such as `torch._inductor.runtime.triton_heuristics` for the logic in this range. Introduces function `get_triton_kernel`, function `benchmark_all_kernels`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.runtime.triton_heuristics` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_triton_kernel`、函数`benchmark_all_kernels`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
    Run the kernel benchmarks for all the kernels cached in PyCodeCache.
    Used in the compiled modules.

    Put this method here rather than codegen it for convenience since its implementation
    does not change based on different graph modules being compiled.
    """
    from torch._inductor.codecache import PyCodeCache

    nfound = 0
    for kernel_mod in PyCodeCache.modules:
        kernel_key = kernel_mod.key
        if not hasattr(kernel_mod, "get_args") or not hasattr(kernel_mod, "call"):
            continue

        triton_kernel = get_triton_kernel(kernel_mod)
        device_type = triton_kernel.device_props.type
        kernel_category = get_kernel_category(kernel_mod)
        args = kernel_mod.get_args()
        num_in_out_ptrs = len(
            [
````
- **EN**: Imports dependencies such as `torch._inductor.codecache` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nfound`, `kernel_key`, `triton_kernel`, `device_type`, `kernel_category`, `args`, and `...+1`.
- **CN**: 这里导入了 `torch._inductor.codecache` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `nfound`、`kernel_key`、`triton_kernel`、`device_type`、`kernel_category`、`args`、`另有1项` 等值。

### Lines 101-120 / 第 101-120 行
````python
                arg_name
                for arg_name in triton_kernel.fn.arg_names
                if arg_name.startswith("in_out_ptr")
            ]
        )
        num_gb = triton_kernel.inductor_meta.get("kernel_num_gb", None)
        if num_gb is None:
            num_gb = get_num_bytes(*args, num_in_out_args=num_in_out_ptrs) / 1e9

        def get_info_str(
            ms: float,
            n_regs: Any | None,
            n_spills: Any | None,
            shared: Any | None,
            prefix: str = "",
        ) -> str:
            if not any(x is None for x in [n_regs, n_spills, shared]):
                kernel_detail_str = (
                    f"  {n_regs:3} regs  {n_spills:3} spills  {shared:8} shared mem"
                )
````
- **EN**: Introduces function `get_info_str`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_gb`, `ms`, `n_regs`, `n_spills`, `shared`, `prefix`, and `...+1`.
- **CN**: 这里定义了函数`get_info_str`。包含分支、循环或上下文管理等控制流。初始化或更新了 `num_gb`、`ms`、`n_regs`、`n_spills`、`shared`、`prefix`、`另有1项` 等值。

### Lines 121-140 / 第 121-140 行
````python
            else:
                kernel_detail_str = ""

            gb_per_s = num_gb / (ms / 1e3)
            return create_bandwidth_info_str(
                ms, num_gb, gb_per_s, prefix=prefix, suffix=kernel_detail_str
            )

        kernel_desc = (
            f"{benchmark_name:20} {kernel_category[:3].upper()} {kernel_key[:10]}"
        )
        if benchmark_all_configs:
            assert hasattr(kernel_mod, "benchmark_all_configs")
            bench_result = kernel_mod.benchmark_all_configs(args)
            print(kernel_desc)
            for launcher, ms in bench_result.items():
                print(
                    f"  {get_info_str(ms, launcher.n_regs, launcher.n_spills, launcher.shared)} @ {launcher.config}"
                )
        else:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `kernel_detail_str`, `gb_per_s`, `kernel_desc`, and `bench_result`. This range continues the implementation of function `benchmark_all_kernels`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`kernel_detail_str`、`gb_per_s`、`kernel_desc`、`bench_result` 等值。这一段延续了函数`benchmark_all_kernels` 的具体实现。

### Lines 141-160 / 第 141-160 行
````python
            ms = benchmarker.benchmark(
                lambda: kernel_mod.call(args),
                device=device_type,
                rep=40,
            )
            assert len(triton_kernel.launchers) == 1, (
                "Autotuner should have selected the best config"
            )
            launcher = triton_kernel.launchers[0]
            print(
                get_info_str(
                    ms,
                    launcher.n_regs,
                    launcher.n_spills,
                    launcher.shared,
                    prefix=f"{kernel_desc} ",
                )
            )

        nfound += 1
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `ms`, `lambda`, `device`, `rep`, `launcher`, and `prefix`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `ms`、`lambda`、`device`、`rep`、`launcher`、`prefix` 等值。

### Lines 161-180 / 第 161-180 行
````python
    if nfound == 0:
        print(
            "No kernel with benchmark functionality found. Make sure you run inductor with config.benchmark_kernel being True"
        )


@dataclass
class ProfileEvent:
    category: str
    key: str
    self_device_time_ms: float
    # the benchmark is run multiple times and we average the count across all the
    # runs. It should be an integer but define a float just in case.
    count: float


def parse_profile_event_list(
    benchmark_name: str,
    event_list: torch.autograd.profiler_util.EventList,
    wall_time_ms: float,
````
- **EN**: Introduces class `ProfileEvent`, function `parse_profile_event_list`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Processes profiling, FLOP, or benchmark information to characterize execution cost.
- **CN**: 这里定义了类`ProfileEvent`、函数`parse_profile_event_list`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。

### Lines 181-200 / 第 181-200 行
````python
    nruns: int,
    device_name: str,
) -> None:
    """
    Parse and generate a report for an event_list.
    """

    def get_self_device_time(
        ev: torch.autograd.profiler_util.EventList,
    ) -> float:
        """
        ev.self_device_time_total is in microsecond. Convert to millisecond.
        """
        return ev.self_device_time_total / 1000 / nruns  # type: ignore[attr-defined]

    all_events: dict[str, list[ProfileEvent]] = defaultdict(list)

    def add_event(
        ev: torch.autograd.profiler_util.EventList,
        category: str,
````
- **EN**: Introduces function `get_self_device_time`, function `add_event`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `nruns`, `device_name`, `ev`, `all_events`, and `category`.
- **CN**: 这里定义了函数`get_self_device_time`、函数`add_event`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `nruns`、`device_name`、`ev`、`all_events`、`category` 等值。

### Lines 201-220 / 第 201-220 行
````python
    ) -> None:
        profile_ev = ProfileEvent(
            category=category,
            key=ev.key,  # type: ignore[attr-defined]
            self_device_time_ms=get_self_device_time(ev),
            count=ev.count / nruns,  # type: ignore[operator] # average across all runs
        )
        all_events[category].append(profile_ev)

    for ev in event_list:
        assert not ev.is_legacy, "Don't support the legacy profiler"
        if ev.device_type == DeviceType.CPU:
            # ignore the event on CPU side
            continue

        category = "unknown"
        if ev.key.startswith("triton_"):
            if ev.key.startswith("triton_poi"):
                category = "triton_pointwise"
            elif ev.key.startswith("triton_red"):
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
                category = "triton_reduction"
            elif ev.key.startswith("triton_per"):
                category = "triton_persistent_reduction"
            else:
                category = "triton_unknown"

        add_event(ev, category)

    def report_category(category: str, profile_events: list[ProfileEvent]) -> float:
        if not device_name:
            return 0.0

        from tabulate import tabulate

        profile_events.sort(key=lambda ev: ev.self_device_time_ms, reverse=True)

        rows = []
        total_time = 0.0
        print(f"\n  == {category} category kernels == ")
        for ev in profile_events:
````
- **EN**: Imports dependencies such as `tabulate` for the logic in this range. Introduces function `report_category`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `tabulate` 等依赖，为后续逻辑提供基础能力。这里定义了函数`report_category`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 241-260 / 第 241-260 行
````python
            total_time += ev.self_device_time_ms
            percent = f"{ev.self_device_time_ms / wall_time_ms * 100:.2f}%"
            rows.append([ev.key[:120], ev.self_device_time_ms, ev.count, percent])
        rows.append(
            ["Total", total_time, "", f"{total_time / wall_time_ms * 100:.2f}%"]
        )
        print(
            tabulate(
                rows,
                headers=[
                    "Kernel",
                    f"Self {device_name.upper()} TIME (ms)",
                    "Count",
                    "Percent",
                ],
            )
        )
        return total_time

    def report() -> None:
````
- **EN**: Introduces function `report`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `percent`, and `headers`.
- **CN**: 这里定义了函数`report`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `percent`、`headers` 等值。

### Lines 261-280 / 第 261-280 行
````python
        category_list = [
            "triton_pointwise",
            "triton_reduction",
            "triton_persistent_reduction",
            "triton_unknown",
            "unknown",
        ]
        assert OrderedSet(all_events.keys()).issubset(OrderedSet(category_list)), (
            f"{list(all_events.keys())}"
        )

        per_category_wall_time = {}
        total_device_ms = 0.0
        for category in category_list:
            if category in all_events:
                _time = report_category(category, all_events[category])
                per_category_wall_time[category] = _time
                total_device_ms += _time

        device_busy_percent = f"{total_device_ms / wall_time_ms * 100:.2f}%"
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `category_list`, `per_category_wall_time`, `total_device_ms`, `_time`, and `device_busy_percent`. This range continues the implementation of function `parse_profile_event_list.report`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `category_list`、`per_category_wall_time`、`total_device_ms`、`_time`、`device_busy_percent` 等值。这一段延续了函数`parse_profile_event_list.report` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
        if device_name:
            print(
                f"\nPercent of time when {device_name.upper()} is busy: {device_busy_percent}"
            )
        else:
            print("No device detected")

        print(f"Total wall time {wall_time_ms:.3f} ms")

        # output such a line so we can gather such line from all compiled modules from all
        # benchmarks and tabulate it!
        # Columns: benchmark_name, pointwise_percent, reduction_percent, persistent_reduction_percent,
        #   unknown_category_percent, device_busy_percent, wall_time_ms
        tabulate_line = f"Output for tabulate: {benchmark_name}"
        for category in category_list:
            percent = (
                f"{per_category_wall_time.get(category, 0.0) / wall_time_ms * 100:.2f}%"
            )
            tabulate_line += f", {percent}"
        tabulate_line += f", {device_busy_percent}, {wall_time_ms:.3f}ms"
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `tabulate_line`, and `percent`. This range continues the implementation of function `parse_profile_event_list.report`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`tabulate_line`、`percent` 等值。这一段延续了函数`parse_profile_event_list.report` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python

        print(tabulate_line)

    report()


PROFILE_DIR = tempfile.gettempdir()
PROFILE_PATH = f"{PROFILE_DIR}/compiled_module_profile.json"


def perf_profile(
    wall_time_ms: float,
    times: int,
    repeat: int,
    benchmark_name: str,
    benchmark_compiled_module_fn: BenchmarkCallableType,
) -> None:
    with torch.profiler.profile(record_shapes=True) as p:
        benchmark_compiled_module_fn(times=times, repeat=repeat)

````
- **EN**: Introduces function `perf_profile`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `PROFILE_DIR`, `PROFILE_PATH`, `wall_time_ms`, `times`, `repeat`, `benchmark_name`, and `...+1`.
- **CN**: 这里定义了函数`perf_profile`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含分支、循环或上下文管理等控制流。初始化或更新了 `PROFILE_DIR`、`PROFILE_PATH`、`wall_time_ms`、`times`、`repeat`、`benchmark_name`、`另有1项` 等值。

### Lines 321-340 / 第 321-340 行
````python
    path = PROFILE_PATH
    p.export_chrome_trace(path)
    print(f"Profiling result for a compiled module of benchmark {benchmark_name}:")
    print(f"Chrome trace for the profile is written to {path}")
    event_list = p.key_averages(group_by_input_shape=True)
    print(event_list.table(sort_by="self_device_time_total", row_limit=10))
    parse_profile_event_list(
        benchmark_name, event_list, wall_time_ms, times * repeat, p.use_device or ""
    )


def ncu_analyzer(
    benchmark_name: str,
    benchmark_compiled_module_fn: BenchmarkCallableType,
    args: argparse.Namespace,
) -> None:
    import inspect
    import os
    import subprocess

````
- **EN**: Imports dependencies such as `inspect`, `os`, and `subprocess` for the logic in this range. Introduces function `ncu_analyzer`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Initializes or updates values such as `path`, `event_list`, `benchmark_name`, `benchmark_compiled_module_fn`, and `args`.
- **CN**: 这里导入了 `inspect`、`os`、`subprocess` 等依赖，为后续逻辑提供基础能力。这里定义了函数`ncu_analyzer`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。初始化或更新了 `path`、`event_list`、`benchmark_name`、`benchmark_compiled_module_fn`、`args` 等值。

### Lines 341-360 / 第 341-360 行
````python
    kernel_regex = args.ncu_kernel_regex
    metrics = args.ncu_metrics

    module_file = inspect.getfile(benchmark_compiled_module_fn)
    module_dir = os.path.dirname(module_file)
    module_name = os.path.splitext(os.path.basename(module_file))[0]

    ncu_dir = tempfile.gettempdir()
    timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
    ncu_output = os.path.join(ncu_dir, f"ncu_output_{timestamp}.ncu-rep")
    python_cmd = (
        f"""import sys; sys.path.insert(0, '{module_dir}'); """
        f"""from {module_name} import benchmark_compiled_module; """
        """benchmark_compiled_module(times=1, repeat=1)"""
    )

    ncu_cmd = [
        "ncu",
        "--target-processes",
        "all",
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Initializes or updates values such as `kernel_regex`, `metrics`, `module_file`, `module_dir`, `module_name`, `ncu_dir`, and `...+4`. This range continues the implementation of function `ncu_analyzer`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。初始化或更新了 `kernel_regex`、`metrics`、`module_file`、`module_dir`、`module_name`、`ncu_dir`、`另有4项` 等值。这一段延续了函数`ncu_analyzer` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
        "--replay-mode",
        "kernel",
        "--kernel-name-base",
        "function",
        "--print-units",
        "base",
        "--import-source",
        "yes",
        "--force-overwrite",
        "--export",
        ncu_output,
    ]

    if kernel_regex:
        ncu_cmd.extend(["--kernel-name", f"regex:{kernel_regex}"])

    if metrics:
        ncu_cmd.extend(["--metrics", metrics])
    else:
        ncu_cmd.extend(["--set", "full"])
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`. This range continues the implementation of function `ncu_analyzer`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。这一段延续了函数`ncu_analyzer` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python

    ncu_cmd.extend(
        [
            "python",
            "-c",
            python_cmd,
        ]
    )

    try:
        subprocess.run(ncu_cmd, check=True)
        print(f"\nNCU profiling results for benchmark {benchmark_name}:")
        print(f"NCU report has been written to {ncu_output}")

    except subprocess.CalledProcessError as e:
        print(f"NCU profiling failed with error: {e}")
        return


def collect_memory_snapshot(
````
- **EN**: Introduces function `collect_memory_snapshot`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`collect_memory_snapshot`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 401-420 / 第 401-420 行
````python
    benchmark_compiled_module_fn: BenchmarkCallableType,
) -> None:
    assert torch.cuda.is_available()

    torch.cuda.memory._record_memory_history(max_entries=100000)
    benchmark_compiled_module_fn(times=10, repeat=1)  # run 10 times
    snapshot_path = f"{tempfile.gettempdir()}/memory_snapshot.pickle"
    torch.cuda.memory._dump_snapshot(snapshot_path)
    torch.cuda.memory._record_memory_history(enabled=None)
    print(f"The collect memory snapshot has been written to {snapshot_path}")


# With AOTAutograd cache, we directly call the compiled module. So prevent
# Dynamo from reentering
@torch.compiler.disable  # type: ignore[misc]
def compiled_module_main(
    benchmark_name: str, benchmark_compiled_module_fn: BenchmarkCallableType
) -> None:
    """
    This is the function called in __main__ block of a compiled module.
````
- **EN**: Introduces function `compiled_module_main`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`compiled_module_main`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 421-440 / 第 421-440 行
````python
    """
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--benchmark-kernels",
        "-k",
        action="store_true",
        help="Whether to benchmark each individual kernels",
    )
    parser.add_argument(
        "--benchmark-all-configs",
        "-c",
        action="store_true",
        help="Whether to benchmark each individual config for a kernel",
    )
    parser.add_argument(
        "--profile",
        "-p",
        action="store_true",
````
- **EN**: Imports dependencies such as `argparse` for the logic in this range. Processes profiling, FLOP, or benchmark information to characterize execution cost. Initializes or updates values such as `parser`, `action`, and `help`. This range continues the implementation of function `compiled_module_main`.
- **CN**: 这里导入了 `argparse` 等依赖，为后续逻辑提供基础能力。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。初始化或更新了 `parser`、`action`、`help` 等值。这一段延续了函数`compiled_module_main` 的具体实现。

### Lines 441-460 / 第 441-460 行
````python
        help="Whether to profile the compiled module",
    )
    parser.add_argument(
        "--cuda-memory-snapshot",
        action="store_true",
        help="""
            Whether to collect CUDA memory snapshot. Refer to
            "https://pytorch.org/blog/understanding-gpu-memory-1/
            for details about how to visualize the collected snapshot
        """,
    )
    parser.add_argument(
        "--ncu",
        action="store_true",
        help="Whether to run ncu analysis",
    )
    parser.add_argument(
        "--ncu-kernel-regex",
        type=str,
        default=None,
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。

### Lines 461-480 / 第 461-480 行
````python
        help=(
            "Filter kernels profiled by NCU using a regex (e.g., '^triton_.*'). "
            "Maps to '--kernel-name regex:<regex>'. "
            "If None, NCU will profile all kernels."
        ),
    )
    parser.add_argument(
        "--ncu-metrics",
        type=str,
        default=None,
        help=(
            "Comma-separated list of NCU metrics to collect (e.g., 'dram__bytes.sum.per_second'). "
            "If None, NCU will use '--set full'."
        ),
    )
    parser.add_argument(
        "--times",
        type=int,
        default=10,
        help="Number of times to run each benchmark iteration",
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Initializes or updates values such as `help`, `type`, and `default`. This range continues the implementation of function `compiled_module_main`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。初始化或更新了 `help`、`type`、`default` 等值。这一段延续了函数`compiled_module_main` 的具体实现。

### Lines 481-500 / 第 481-500 行
````python
    )
    parser.add_argument(
        "--repeat",
        type=int,
        default=10,
        help="Number of repetitions of each benchmark run",
    )

    args = parser.parse_args()

    if args.benchmark_kernels:
        benchmark_all_kernels(benchmark_name, args.benchmark_all_configs)
    else:
        times = args.times
        repeat = args.repeat

        if torch.cuda.is_available():
            torch.cuda.reset_peak_memory_stats()
        wall_time_ms = benchmark_compiled_module_fn(times=times, repeat=repeat) * 1000

````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `type`, `default`, `help`, `args`, `else`, `times`, and `...+2`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `type`、`default`、`help`、`args`、`else`、`times`、`另有2项` 等值。

### Lines 501-520 / 第 501-520 行
````python
        if torch.cuda.is_available():
            peak_mem = torch.cuda.max_memory_allocated()
            print(f"Peak GPU memory usage {peak_mem / 1e6:.3f} MB")

        if torch.cuda.is_available() and args.cuda_memory_snapshot:
            collect_memory_snapshot(benchmark_compiled_module_fn)

        if args.profile:
            perf_profile(
                wall_time_ms,
                times,
                repeat,
                benchmark_name,
                benchmark_compiled_module_fn,
            )
        if args.ncu:
            ncu_analyzer(
                benchmark_name,
                benchmark_compiled_module_fn,
                args=args,
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `peak_mem`, and `args`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `peak_mem`、`args` 等值。

### Lines 521-521 / 第 521-521 行
````python
            )
````
- **EN**: This range continues the implementation of function `compiled_module_main`.
- **CN**: 这一段延续了函数`compiled_module_main` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `datetime`, `tempfile`, `collections`, `dataclasses`, `types`, `typing`, `inspect`, `os`, `subprocess`
- **Third-party / 第三方**: `tabulate`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.autograd`, `torch.utils._ordered_set`, `.runtime.benchmarking`, `.runtime.runtime_utils`, `torch._inductor.runtime.triton_heuristics`, `torch._inductor.codecache`
