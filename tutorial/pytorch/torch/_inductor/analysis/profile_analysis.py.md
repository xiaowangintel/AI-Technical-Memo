# profile_analysis.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/analysis/profile_analysis.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module analyzes compiler, runtime, or profiling state for TorchInductor. It defines classes such as `ProfileEvent`, `KernelStats`, `Device`, `JsonProfile`, and `ParseException`. It exposes functions such as `parse_list`, `register_adapter`, `_slow_conv2d_adapter`, `conv_adapter`, `default_adapter`, `addmm_adapter`, and `...+11`.
- **用途（中文）**: 该模块分析 TorchInductor 的编译、运行期或性能画像状态。其中定义了 `ProfileEvent`、`KernelStats`、`Device`、`JsonProfile`、`ParseException` 等类。同时提供 `parse_list`、`register_adapter`、`_slow_conv2d_adapter`、`conv_adapter`、`default_adapter`、`addmm_adapter`、`另有11项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
import json
import logging
import math
from collections import defaultdict
from collections.abc import Callable
from dataclasses import dataclass
from typing import Any, Optional

import torch
from torch._inductor.analysis.device_info import DeviceInfo, lookup_device_info
from torch._inductor.utils import tabulate_2d, zip_dicts
from torch.utils import _pytree as pytree
from torch.utils._ordered_set import OrderedSet
from torch.utils.flop_counter import flop_registry


log = logging.getLogger(__name__)


ATEN_PREFIX = "aten::"


@dataclass
class ProfileEvent:
    category: str
    key: str
    self_device_time_ms: float
    # the benchmark is run multiple times and we average the count across all the
````
- **EN**: Imports dependencies such as `json`, `logging`, `math`, `collections`, `collections.abc`, `dataclasses`, and `...+7` for the logic in this range. Introduces class `ProfileEvent`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `json`、`logging`、`math`、`collections`、`collections.abc`、`dataclasses`、`另有7项` 等依赖，为后续逻辑提供基础能力。这里定义了类`ProfileEvent`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 29-56 / 第 29-56 行
````python
    # runs. It should be an integer but define a float just in case.
    count: float


# adapters convert the json trace into a format that works with flops_counter
ArgsType = tuple[tuple[Any, ...], dict[Any, Any]]
AdapterType = Callable[[tuple[Any, ...], tuple[Any, ...]], ArgsType]
adapters_map: dict[str, AdapterType] = {}


def parse_list(lst: str) -> list[int]:
    lst = lst.replace("[", "").replace("]", "")
    substrings = lst.split(",")

    return [int(substring.strip()) for substring in substrings]


def register_adapter(
    aten: str | list[str],
) -> Callable[
    [AdapterType],
    AdapterType,
]:
    def decorator(func: AdapterType) -> AdapterType:
        # pyrefly: ignore [unknown-name]
        global _adapters_map

        if isinstance(aten, str):
````
- **EN**: Introduces function `parse_list`, function `register_adapter`, function `decorator`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`parse_list`、函数`register_adapter`、函数`decorator`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 57-84 / 第 57-84 行
````python
            adapters_map[aten] = func
        else:
            for at in aten:
                adapters_map[at] = func
        return func

    return decorator


@register_adapter(["_slow_conv2d_forward"])
def _slow_conv2d_adapter(
    shapes: tuple[Any, ...], concrete: tuple[Any, ...]
) -> tuple[tuple[Any], dict[Any, Any]]:
    tmp = list(shapes)
    tmp.append(False)
    tmp2 = list(concrete)
    if len(tmp2) < 5:
        raise ParseException("slow conv2d has less than 5 concrete inputs")
    tmp2[3] = tmp2[4]
    return conv_adapter(tuple(tmp), tuple(tmp2))


@register_adapter(
    ["convolution", "_convolution", "cudnn_convolution", "convolution_overrideable"]
)
def conv_adapter(
    shapes: tuple[Any, ...], concrete: tuple[Any, ...]
) -> tuple[tuple[Any], dict[Any, Any]]:
````
- **EN**: Introduces function `_slow_conv2d_adapter`, function `conv_adapter`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_slow_conv2d_adapter`、函数`conv_adapter`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 85-112 / 第 85-112 行
````python
    tmp = list(shapes)
    if len(tmp) == 4:
        transposed = False
    elif len(tmp) > 6:
        transposed = bool(tmp[6])
        tmp[6] = transposed
    else:
        raise ParseException(f"Convolution has the wrong number of inputs: {len(tmp)}")

    kwargs: dict[Any, Any] = {}
    if not transposed:
        # calculate output shape if not transposed.
        def conv_out_dims(x: int, kernel: int, stride: int) -> int:
            return (x - kernel) // stride + 1

        stride = parse_list(concrete[3])
        inp = shapes[0]
        w = shapes[1]
        out_x_y = [conv_out_dims(*args) for args in zip(inp[2:], w[2:], stride)]
        out = [inp[0], w[0]] + out_x_y  # we only need the xy values
        kwargs["out_val"] = out

    return tuple(tmp), kwargs


def default_adapter(
    shapes: tuple[Any], concrete: tuple[Any]
) -> tuple[tuple[Any], dict[Any, Any]]:
````
- **EN**: Introduces function `conv_out_dims`, function `default_adapter`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tmp`, `transposed`, `else`, `kwargs`, `stride`, `inp`, and `...+4`.
- **CN**: 这里定义了函数`conv_out_dims`、函数`default_adapter`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tmp`、`transposed`、`else`、`kwargs`、`stride`、`inp`、`另有4项` 等值。

### Lines 113-140 / 第 113-140 行
````python
    return shapes, {}


@register_adapter("addmm")
def addmm_adapter(
    shapes: tuple[Any], concrete: tuple[Any]
) -> tuple[tuple[Any], dict[Any, Any]]:
    tmp = list(shapes)[:3]
    return tuple(tmp), {}


@register_adapter("bmm")
def bmm_adapter(
    shapes: tuple[Any], concrete: tuple[Any]
) -> tuple[tuple[Any], dict[Any, Any]]:
    tmp = list(shapes)
    return tuple(tmp[:2]), {}


@register_adapter("baddbmm")
def baddbmm_adapter(
    shapes: tuple[Any], concrete: tuple[Any]
) -> tuple[tuple[Any], dict[Any, Any]]:
    tmp = list(shapes)[:3]
    return tuple(tmp), {}


@register_adapter("mm")
````
- **EN**: Introduces function `addmm_adapter`, function `bmm_adapter`, function `baddbmm_adapter`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`addmm_adapter`、函数`bmm_adapter`、函数`baddbmm_adapter`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-168 / 第 141-168 行
````python
def mm_adapter(
    shapes: tuple[Any], concrete: tuple[Any]
) -> tuple[tuple[Any], dict[Any, Any]]:
    return shapes, {}


def _parse_kernel_name(name: str) -> str | None:
    """
    parse the name of the kernel from the event name.
    """
    if name.startswith(ATEN_PREFIX):
        return name[len(ATEN_PREFIX) :]
    elif "conv" in name:
        return "convolution"
    elif "addmm" in name:
        return "addmm"
    elif "bmm" in name:
        return "bmm"
    elif "baddbmm" in name:
        return "baddbmm"
    elif "_mm" in name:
        return "mm"
    else:
        return None


def _calculate_flops(event: dict[str, Any]) -> int:
    """
````
- **EN**: Introduces function `mm_adapter`, function `_parse_kernel_name`, function `_calculate_flops`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `shapes`, and `else`.
- **CN**: 这里定义了函数`mm_adapter`、函数`_parse_kernel_name`、函数`_calculate_flops`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `shapes`、`else` 等值。

### Lines 169-196 / 第 169-196 行
````python
    This function has to parse the kernel name, which is error prone. There doesn't seem to be another solution that
    will support all the different backends that can generate kernels, so make sure to update this function when new
    ops and backends are desired.
    """
    name = event["name"]
    if "kernel_flop" in event["args"] and event["args"]["kernel_flop"] != 0:
        return event["args"]["kernel_flop"]
    op_name = _parse_kernel_name(name)
    if op_name is None:
        return 0

    op_obj = getattr(torch.ops.aten, op_name, None)
    if op_obj is None or op_obj not in flop_registry:
        return 0

    flop_function = flop_registry[op_obj]

    if "Input Dims" not in event["args"] or "Concrete Inputs" not in event["args"]:
        return 0
    input_shapes = event["args"]["Input Dims"]
    concrete = event["args"]["Concrete Inputs"]
    if op_name in adapters_map:
        try:
            args, kwargs = adapters_map[op_name](input_shapes, concrete)
        except ParseException as e:
            msg = f"Failed to parse {op_name} with {e}"
            log.warning(msg)
            return 0
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `op_name`, `op_obj`, `flop_function`, `input_shapes`, `concrete`, and `...+2`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`op_name`、`op_obj`、`flop_function`、`input_shapes`、`concrete`、`另有2项` 等值。

### Lines 197-224 / 第 197-224 行
````python
    else:
        try:
            args, kwargs = default_adapter(input_shapes, concrete)
        except ParseException as e:
            msg = f"Failed to parse {op_name} with {e}"
            log.warning(msg)
            return 0
    return flop_function(*args, **kwargs)


def _get_size_from_string(type_string: str) -> int:
    if not hasattr(torch, type_string):
        return 1
    else:
        return getattr(torch, type_string).itemsize


def _default_estimate_gb(event: dict[str, Any]) -> float:
    sizes_and_types = zip(event["args"]["Input Dims"], event["args"]["Input type"])
    bw = 0
    for size, typ in sizes_and_types:
        isize = _get_size_from_string(typ)
        bw += isize * math.prod(pytree.tree_flatten(size)[0])
    return bw / 1e9


def _estimate_gb(event: dict[str, Any]) -> float:
    """
````
- **EN**: Introduces function `_get_size_from_string`, function `_default_estimate_gb`, function `_estimate_gb`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `try`, `msg`, `sizes_and_types`, `bw`, and `isize`.
- **CN**: 这里定义了函数`_get_size_from_string`、函数`_default_estimate_gb`、函数`_estimate_gb`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`try`、`msg`、`sizes_and_types`、`bw`、`isize` 等值。

### Lines 225-252 / 第 225-252 行
````python
    Our best effort to estimate the gb, should be refactored soon with MemoryCounter.
    """
    name = event["name"]
    if "kernel_num_gb" in event["args"] and event["args"]["kernel_num_gb"] != 0:
        return event["args"]["kernel_num_gb"]
    if "Input type" not in event["args"] or "Input Dims" not in event["args"]:
        return 0
    op_name = _parse_kernel_name(name)
    if op_name is None:
        return _default_estimate_gb(event)

    op_obj = getattr(torch.ops.aten, op_name, None)
    if op_obj is None:
        return _default_estimate_gb(event)

    if "Input Dims" not in event["args"] or "Concrete Inputs" not in event["args"]:
        return _default_estimate_gb(event)
    input_shapes = event["args"]["Input Dims"]

    # NOTE these will be refactored into a similar object to FlopCounter soon
    def mm_formula(M: int, N: int, K: int, size: int) -> int:
        return 2 * (M * K + N * K + M * N) * size

    if op_name == "addmm":
        add_in_size = math.prod(pytree.tree_flatten(input_shapes[0])[0])
        add_type_size = _get_size_from_string(event["args"]["Input type"][0])
        M = input_shapes[1][0]
        N = input_shapes[1][1]
````
- **EN**: Introduces function `mm_formula`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `op_name`, `op_obj`, `input_shapes`, `add_in_size`, `add_type_size`, and `...+2`.
- **CN**: 这里定义了函数`mm_formula`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`op_name`、`op_obj`、`input_shapes`、`add_in_size`、`add_type_size`、`另有2项` 等值。

### Lines 253-280 / 第 253-280 行
````python
        assert input_shapes[1][1] == input_shapes[2][0]
        K = input_shapes[2][1]
        mul_type_size = _get_size_from_string(event["args"]["Input type"][1])
        return (mm_formula(M, N, K, mul_type_size) + add_in_size * add_type_size) / 1e9
    elif op_name == "mm":
        M = input_shapes[0][0]
        N = input_shapes[0][1]
        assert input_shapes[0][1] == input_shapes[1][0]
        K = input_shapes[1][1]
        type_size = _get_size_from_string(event["args"]["Input type"][0])
        return mm_formula(M, N, K, type_size) / 1e9
    elif op_name == "baddbmm":
        add_in_size = math.prod(pytree.tree_flatten(input_shapes[0])[0])
        add_type_size = _get_size_from_string(event["args"]["Input type"][0])
        B = input_shapes[0][0]
        M = input_shapes[1][1]
        N = input_shapes[1][2]
        K = input_shapes[2][2]
        mul_type_size = _get_size_from_string(event["args"]["Input type"][1])
        return (
            B * mm_formula(M, N, K, mul_type_size) + add_in_size * add_type_size
        ) / 1e9
    elif op_name == "bmm":
        add_in_size = math.prod(pytree.tree_flatten(input_shapes[0])[0])
        add_type_size = _get_size_from_string(event["args"]["Input type"][0])
        B = input_shapes[0][0]
        M = input_shapes[0][1]
        N = input_shapes[0][2]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `K`, `mul_type_size`, `M`, `N`, `type_size`, `add_in_size`, and `...+2`. This range continues the implementation of function `_estimate_gb`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `K`、`mul_type_size`、`M`、`N`、`type_size`、`add_in_size`、`另有2项` 等值。这一段延续了函数`_estimate_gb` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
        K = input_shapes[1][2]
        mul_type_size = _get_size_from_string(event["args"]["Input type"][1])
        return (
            B * mm_formula(M, N, K, mul_type_size) + add_in_size * add_type_size
        ) / 1e9
    elif op_name in [
        "convolution",
        "_convolution",
        "cudnn_convolution",
        "_slow_conv2d_forward",
    ]:
        concrete = event["args"]["Concrete Inputs"]

        def conv_out_dim(x: int, kernel: int, stride: int) -> int:
            return (x - kernel) // stride + 1

        stride = parse_list(
            concrete[3] if op_name != "_slow_conv2d_forward" else concrete[4]
        )
        inp = input_shapes[0]
        w = input_shapes[1]
        out_x_y = [conv_out_dim(*args) for args in zip(inp[2:], w[2:], stride)]
        out = [inp[0], w[0]] + out_x_y
        # each output element reads in * w * w chunk
        input_reads = out[0] * out[1] * out[2] * out[3] * inp[1] * w[2] * w[3]
        # Assume weights are in cache, so only read once
        weight_reads = w[0] * w[1] * w[2] * w[3]
        return (input_reads + weight_reads) / 1e9
````
- **EN**: Introduces function `conv_out_dim`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`conv_out_dim`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 309-336 / 第 309-336 行
````python

    return _default_estimate_gb(event)


def _create_extern_mapping(
    data: dict[str, Any],
) -> defaultdict[int, list[dict[str, Any]]]:
    """
    compute a mapping from external ids to non kernels, which contain the information we need to estimate flops etc
    """
    extern_mapping: defaultdict[int, list[dict[str, Any]]] = defaultdict(list)
    for event in data["traceEvents"]:
        if (
            "args" not in event
            or "External id" not in event["args"]
            or event["cat"] != "cpu_op"
        ):
            continue
        if len(extern_mapping[event["args"]["External id"]]) > 0:
            raise ParseException("duplicate external id in event")
        extern_mapping[event["args"]["External id"]].append(event)
    return extern_mapping


def _augment_trace_helper(data: dict[str, Any]) -> dict[str, Any]:
    extern_mapping = _create_extern_mapping(data)

    for event in data["traceEvents"]:
````
- **EN**: Introduces function `_create_extern_mapping`, function `_augment_trace_helper`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_create_extern_mapping`、函数`_augment_trace_helper`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 337-364 / 第 337-364 行
````python
        if "cat" not in event or event["cat"] != "kernel":
            continue
        if "args" not in event:
            raise ParseException(f"kernel has no args: {event}")
        if "External id" not in event["args"]:
            event_str = f"kernel has no External id: {event}"
            log.info(event_str)
            continue

        external_op = extern_mapping[event["args"]["External id"]][0]
        flops = _calculate_flops(external_op)
        if flops == 0:
            flops = _calculate_flops(event)
        external_op["args"]["kernel_flop"] = flops
        external_op["args"]["kernel_num_gb"] = _estimate_gb(external_op)
        event["args"]["kernel_flop"] = external_op["args"]["kernel_flop"]
        event["args"]["kernel_num_gb"] = external_op["args"]["kernel_num_gb"]
    return data


_dtype_map = {
    "float": torch.float,
    "float32": torch.float,
    "int": torch.int,
    "int8": torch.int8,
    "int16": torch.int16,
    "int32": torch.int,
    "long": torch.long,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 365-392 / 第 365-392 行
````python
    "long int": torch.long,
    "bfloat16": torch.bfloat16,
    "float16": torch.float16,
    "float64": torch.double,
}


@dataclass(frozen=True)
class KernelStats:
    flops: int
    bw: float
    latency: float  # us
    achieved_flops: float
    achieved_bandwidth: float


KernelNameMap = defaultdict[str, OrderedSet[KernelStats]]


@dataclass(frozen=False)
class Device:
    name: str
    index: int
    info: DeviceInfo | None
    stats: KernelNameMap

    def __repr__(self) -> str:
        return f"Device({self.name}, {self.index}): {self.info}"
````
- **EN**: Introduces class `KernelStats`, class `Device`, function `__repr__`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了类`KernelStats`、类`Device`、函数`__repr__`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。处理与设备相关的行为、能力信息或派发决策。

### Lines 393-420 / 第 393-420 行
````python


DeviceMap = dict[int, Device]
Table = tuple[list[str], dict[str, list[str]]]


class JsonProfile:
    _devices: DeviceMap

    def __init__(
        self,
        path: str,
        benchmark_name: str | None = None,
        dtype: torch.dtype | str | None = None,
    ):
        """
        Convenience class for running common operations on chrome/perfetto json traces.
        """
        self.path = path
        with open(path) as f:
            self.data = json.load(f)
            self.events = self.data["traceEvents"]
        self.benchmark_name = benchmark_name
        if dtype is None:
            self.dtype = None
        elif isinstance(dtype, torch.dtype):
            # pyrefly: ignore [bad-assignment]
            self.dtype = dtype
````
- **EN**: Introduces class `JsonProfile`, function `__init__`. Serializes or deserializes JSON data as part of persistence or interchange. Touches the filesystem to load, validate, or store compiler artifacts. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了类`JsonProfile`、函数`__init__`。通过 JSON 的序列化或反序列化来完成持久化或数据交换。这一段会访问文件系统，用于加载、校验或保存编译产物。处理与设备相关的行为、能力信息或派发决策。

### Lines 421-448 / 第 421-448 行
````python
        else:
            # pyrefly: ignore [bad-assignment]
            self.dtype = _dtype_map.get(dtype)
        self._create_devices()

    def convert_dtype(self, event: dict[str, Any]) -> torch.dtype | None:
        """
        Each op has a list of dtypes for each input arg. We need to convert these into a single dtype for flop estimation.
        Issues:
         - converting the strings to concrete torch.dtypes
         - What if we have float32, float, float16 all in the inputs? Our choice is to use the largest buffer dtype.
        """

        if (
            "Input Dims" not in event["args"]
            or "Input type" not in event["args"]
            or "Concrete Inputs" not in event["args"]
        ):
            if "bfloat16" in event["name"]:
                return torch.bfloat16
            elif "float16" in event["name"]:
                return torch.float16
            else:
                return None

        input_sizes = event["args"]["Input Dims"]
        input_types = event["args"]["Input type"]
        concrete_inputs = event["args"]["Concrete Inputs"]
````
- **EN**: Introduces function `convert_dtype`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`convert_dtype`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 449-476 / 第 449-476 行
````python
        assert len(input_sizes) == len(input_types)
        assert len(input_types) == len(concrete_inputs)

        if len(input_sizes) == 0:
            raise RuntimeError("Empty input_sizes and input_types")

        biggest_size = 0
        biggest_index = 0
        for i in range(len(input_sizes)):
            if concrete_inputs[i] != "":
                # concrete inputs are usually small tensors, so we can just skip
                continue
            my_size = input_sizes[i]
            total_size = sum(parse_list(my_size))
            if total_size > biggest_size:
                biggest_size = total_size
                biggest_index = i
        ret_type = input_types[biggest_index]
        if ret_type in _dtype_map:
            return _dtype_map[ret_type]
        raise RuntimeError(f"Unknown type: {ret_type}. Please add to _dtype_map.")

    def _create_devices(self) -> None:
        self._devices = {}
        for dev in self.data["deviceProperties"]:
            name = dev["name"]
            device_info = lookup_device_info(name)

````
- **EN**: Introduces function `_create_devices`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `biggest_size`, `biggest_index`, `my_size`, `total_size`, `ret_type`, `name`, and `...+1`.
- **CN**: 这里定义了函数`_create_devices`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `biggest_size`、`biggest_index`、`my_size`、`total_size`、`ret_type`、`name`、`另有1项` 等值。

### Lines 477-504 / 第 477-504 行
````python
            if device_info is None:
                log.info(
                    "Unsupported device in profile: %s, please consider contributing to _device_mapping.",
                    name,
                )
            self._devices[dev["id"]] = Device(
                name, dev["id"], device_info, defaultdict(OrderedSet)
            )

    def calculate_flops(self, event: dict[str, Any]) -> int:
        return _calculate_flops(event)

    def estimate_gb(self, event: dict[str, Any]) -> float:
        return _estimate_gb(event)

    def augment_trace(self) -> None:
        self.data = _augment_trace_helper(self.data)

    def _compute_stats(self) -> None:
        """populates the name -> stats map"""
        for event in self.events:
            if "cat" not in event or "args" not in event or event["cat"] != "kernel":
                continue
            if "device" not in event["args"]:
                continue
            dev_tmp = event["args"]["device"]
            if dev_tmp not in self._devices:
                continue
````
- **EN**: Introduces function `calculate_flops`, function `estimate_gb`, function `augment_trace`, function `_compute_stats`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`calculate_flops`、函数`estimate_gb`、函数`augment_trace`、函数`_compute_stats`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 505-532 / 第 505-532 行
````python
            dev = self._devices[event["args"]["device"]]

            dur = event["dur"]  # us
            if "kernel_flop" in event["args"]:
                assert dur != 0
                # 1,000,000us/s * flop / us
                op_flops = event["args"]["kernel_flop"] / (dur / 1e6)
            else:
                op_flops = 0

            if "kernel_num_gb" in event["args"]:
                assert dur != 0
                # 1,000,000us/s * gb  = gb/s
                op_gbps = event["args"]["kernel_num_gb"] / (dur / 1e6)
            else:
                op_gbps = 0

            if dev.info is not None:
                dtype = self.convert_dtype(event) or self.dtype
                if dtype is None:
                    raise RuntimeError(
                        "dtype is not found on tensor and default dtype is not set"
                    )
                achieved_flops = 100 * op_flops / (1e12 * dev.info.tops[dtype])
                achieved_bandwidth = 100 * op_gbps / dev.info.dram_bw_gbs
            else:
                achieved_flops = 0
                achieved_bandwidth = 0
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 533-560 / 第 533-560 行
````python

            if "name" not in event["args"]:
                continue
            dev.stats[event["name"]].add(
                KernelStats(
                    flops=op_flops,
                    bw=op_gbps,
                    latency=dur,
                    achieved_bandwidth=achieved_bandwidth,
                    achieved_flops=achieved_flops,
                )
            )

    def _create_single_table(self, dev: Device) -> Table:
        """Create a table with the devices mapped to indices."""
        headers = [
            "Kernel Name",
            "Kernel Count",
            "FLOPS",
            "Kernel Reads (GB)",
            "Dur (us)",
            "Achieved FLOPS %",
            "Achieved Bandwidth %",
        ]
        rows: dict[str, list[str]] = {}

        def safe_div_format(x: float, y: float) -> str:
            if y == 0:
````
- **EN**: Introduces function `_create_single_table`, function `safe_div_format`. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `flops`, `bw`, `latency`, `achieved_bandwidth`, `achieved_flops`, `headers`, and `...+1`.
- **CN**: 这里定义了函数`_create_single_table`、函数`safe_div_format`。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `flops`、`bw`、`latency`、`achieved_bandwidth`、`achieved_flops`、`headers`、`另有1项` 等值。

### Lines 561-588 / 第 561-588 行
````python
                return "0.0"
            return f"{x / y:.4f}"

        for kernel_name, stats_set in dev.stats.items():
            ker_count = 0
            flops = 0
            flops_count = 0
            achieved_flops = 0.0
            bw = 0.0
            bw_count = 0
            achieved_bandwidth = 0.0
            latency = 0.0
            for stats in stats_set:
                if stats.flops != 0:
                    flops += stats.flops
                    achieved_flops += stats.achieved_flops
                    flops_count += 1
                if stats.bw != 0:
                    bw += stats.bw
                    achieved_bandwidth += stats.achieved_bandwidth
                    bw_count += 1
                latency += stats.latency
                ker_count += 1
            assert ker_count != 0
            rows[kernel_name] = [
                str(ker_count),
                safe_div_format(flops, flops_count),
                safe_div_format(bw, bw_count),
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ker_count`, `flops`, `flops_count`, `achieved_flops`, `bw`, `bw_count`, and `...+2`. This range continues the implementation of function `JsonProfile._create_single_table`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ker_count`、`flops`、`flops_count`、`achieved_flops`、`bw`、`bw_count`、`另有2项` 等值。这一段延续了函数`JsonProfile._create_single_table` 的具体实现。

### Lines 589-616 / 第 589-616 行
````python
                safe_div_format(latency, ker_count),
                safe_div_format(achieved_flops, flops_count),
                safe_div_format(achieved_bandwidth, bw_count),
            ]

        return headers, rows

    def _create_tables(self, devs: DeviceMap) -> dict[int, Table]:
        return {idx: self._create_single_table(dev) for idx, dev in devs.items()}

    def _combine_tables(
        self, table1: Table, table1_name: str, table2: Table, table2_name: str
    ) -> Table:
        new_headers = (
            ["Kernel Name"]
            + [f"{table1_name} {head}" for head in table1[0][1:]]
            + [f"{table2_name} {head}" for head in table2[0][1:]]
        )
        t1_length = len(table1[0][1:])
        t2_length = len(table2[0][1:])
        new_rows = {}

        for key, row1, row2 in zip_dicts(
            table1[1],
            table2[1],
            d1_default=["Empty"] * t1_length,
            d2_default=["Empty"] * t2_length,
        ):
````
- **EN**: Introduces function `_create_tables`, function `_combine_tables`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_headers`, `t1_length`, `t2_length`, `new_rows`, `d1_default`, and `d2_default`.
- **CN**: 这里定义了函数`_create_tables`、函数`_combine_tables`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_headers`、`t1_length`、`t2_length`、`new_rows`、`d1_default`、`d2_default` 等值。

### Lines 617-644 / 第 617-644 行
````python
            assert row1 is not None
            assert row2 is not None
            new_rows[key] = row1 + row2
        return new_headers, new_rows

    def report(
        self, other: Optional["JsonProfile"] = None, name_limit: int = 40
    ) -> str:
        def create_ret(
            table_headers: list[str], table_rows: dict[str, list[str]]
        ) -> str:
            table_flattened = [
                [kernel_name[:name_limit], *kernel_vals]
                for kernel_name, kernel_vals in table_rows.items()
            ]
            return tabulate_2d(table_flattened, headers=table_headers)

        if other is not None:
            self._compute_stats()
            other._compute_stats()

            self_tables = self._create_tables(self._devices)
            other_tables = self._create_tables(other._devices)

            self_name = (
                self.benchmark_name if self.benchmark_name is not None else "Table 1"
            )
            other_name = (
````
- **EN**: Introduces function `report`, function `create_ret`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `table_headers`, `table_flattened`, `self_tables`, `other_tables`, `self_name`, and `other_name`.
- **CN**: 这里定义了函数`report`、函数`create_ret`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `table_headers`、`table_flattened`、`self_tables`、`other_tables`、`self_name`、`other_name` 等值。

### Lines 645-672 / 第 645-672 行
````python
                other.benchmark_name if other.benchmark_name is not None else "Table 2"
            )

            ret = []
            assert self._devices.keys() == other._devices.keys()
            for device_idx, t1, t2 in zip_dicts(
                self_tables, other_tables, d1_default=None, d2_default=None
            ):
                assert t1 is not None
                assert t2 is not None
                table_headers, table_rows = self._combine_tables(
                    t1, self_name, t2, other_name
                )
                tab_string = create_ret(table_headers, table_rows)
                # pyrefly: ignore [bad-argument-type]
                ret.append(f"{self._devices[device_idx]}:\n{tab_string}")
            return "\n".join(ret)
        self._compute_stats()

        self_tables = self._create_tables(self._devices)

        ret = []
        for idx, table in self_tables.items():
            table_headers, table_rows = table
            tab_string = create_ret(table_headers, table_rows)
            # pyrefly: ignore [bad-argument-type]
            ret.append(f"{self._devices[idx]}:\n{tab_string}")
        return "\n".join(ret)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ret`, `tab_string`, and `self_tables`. This range continues the implementation of function `JsonProfile.report`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ret`、`tab_string`、`self_tables` 等值。这一段延续了函数`JsonProfile.report` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python

    def dump(self, out: str) -> None:
        with open(out, "w") as f:
            json.dump(self.data, f)

    def combine_with(self, other: "JsonProfile") -> "JsonProfile":
        """
        Combine this profile with another profile by merging their trace events.
        Returns a new JsonProfile object with combined data.
        """
        # Create a new combined data structure
        combined_data = {
            "traceEvents": self.data["traceEvents"] + other.data["traceEvents"],
            "deviceProperties": self.data.get("deviceProperties", []),
        }

        # Merge device properties, avoiding duplicates
        other_device_props = other.data.get("deviceProperties", [])
        existing_device_ids = OrderedSet(
            [dev["id"] for dev in combined_data["deviceProperties"]]
        )

        for device_prop in other_device_props:
            if device_prop["id"] not in existing_device_ids:
                combined_data["deviceProperties"].append(device_prop)

        # Copy any other top-level properties from the first profile
        for key, value in self.data.items():
````
- **EN**: Introduces function `dump`, function `combine_with`. Serializes or deserializes JSON data as part of persistence or interchange. Touches the filesystem to load, validate, or store compiler artifacts. Processes profiling, FLOP, or benchmark information to characterize execution cost.
- **CN**: 这里定义了函数`dump`、函数`combine_with`。通过 JSON 的序列化或反序列化来完成持久化或数据交换。这一段会访问文件系统，用于加载、校验或保存编译产物。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。

### Lines 701-728 / 第 701-728 行
````python
            if key not in combined_data:
                combined_data[key] = value

        import os

        # Create a temporary file to write the combined data
        import tempfile

        with tempfile.NamedTemporaryFile(
            mode="w", suffix=".json", delete=False
        ) as tmp_file:
            json.dump(combined_data, tmp_file)
            tmp_path = tmp_file.name

        try:
            # Create new JsonProfile from the combined data
            combined_profile = JsonProfile(
                tmp_path,
                benchmark_name=f"{self.benchmark_name or 'Profile1'}_+_{other.benchmark_name or 'Profile2'}",
                dtype=self.dtype or other.dtype,
            )
            return combined_profile
        finally:
            # Clean up temporary file
            os.unlink(tmp_path)


class ParseException(RuntimeError):
````
- **EN**: Imports dependencies such as `os`, and `tempfile` for the logic in this range. Introduces class `ParseException`. Serializes or deserializes JSON data as part of persistence or interchange. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `os`、`tempfile` 等依赖，为后续逻辑提供基础能力。这里定义了类`ParseException`。通过 JSON 的序列化或反序列化来完成持久化或数据交换。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 729-756 / 第 729-756 行
````python
    pass


def main() -> None:
    """
    Main function for the profile analysis script.
    """
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--diff",
        nargs=5,
        metavar=(
            "input_file1",
            "name1",
            "input_file2",
            "name2",
            "dtype",
        ),
        help="Two json traces to compare with, specified as <file1> <name1> <file2> <name2> <dtype>",
    )
    parser.add_argument(
        "--name_limit",
        type=int,
        help="the maximum name size in the final report",
    )
    parser.add_argument(
````
- **EN**: Imports dependencies such as `argparse` for the logic in this range. Introduces function `main`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `argparse` 等依赖，为后续逻辑提供基础能力。这里定义了函数`main`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 757-784 / 第 757-784 行
````python
        "--augment_trace",
        "-a",
        nargs=3,
        metavar=("input_file", "output_file", "dtype"),
        help="Augment a trace with inductor meta information. Provide input and output file paths.",
    )
    parser.add_argument(
        "--analysis",
        nargs=2,
        metavar=("input_file", "dtype"),
        help="Run analysis on a single trace, specified as <file> <dtype>",
    )
    parser.add_argument(
        "--combine",
        nargs="+",
        metavar=("input_files", "output_file"),
        help="Combine multiple profiles into a single profile by merging trace events. Specify as <input_file1> \
<input_file2> [input_file3 ...] <output_file>. The last argument is the output file, all preceding arguments are \
input files to combine.",
    )
    args = parser.parse_args()

    if args.diff:
        p1 = JsonProfile(args.diff[0], args.diff[1], dtype=args.diff[4])
        p1.augment_trace()
        p2 = JsonProfile(args.diff[2], args.diff[3], dtype=args.diff[4])
        p2.augment_trace()
        if args.name_limit:
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nargs`, `metavar`, `help`, `args`, `p1`, and `p2`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `nargs`、`metavar`、`help`、`args`、`p1`、`p2` 等值。

### Lines 785-812 / 第 785-812 行
````python
            print(p1.report(p2, name_limit=args.name_limit))
        else:
            print(p1.report(p2))
    if args.analysis:
        p1 = JsonProfile(
            args.analysis[0],
            dtype=args.analysis[1],
        )
        p1.augment_trace()
        if args.name_limit:
            print(p1.report(name_limit=args.name_limit))
        else:
            print(p1.report())
    if args.augment_trace:
        p = JsonProfile(args.augment_trace[0], dtype=args.augment_trace[2])
        p.augment_trace()
        p.dump(args.augment_trace[1])
    if args.combine:
        input_files = args.combine[:-1]  # All arguments except the last one
        output_file = args.combine[-1]  # Last argument is the output file

        if len(input_files) < 2:
            print("Error: At least 2 input files are required for combining")
            return

        # Load the first profile
        combined = JsonProfile(input_files[0], dtype=None)

````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 813-823 / 第 813-823 行
````python
        # Iteratively combine with all other profiles
        for input_file in input_files[1:]:
            profile = JsonProfile(input_file, dtype=None)
            combined = combined.combine_with(profile)

        combined.dump(output_file)
        print(f"Successfully combined {', '.join(input_files)} into {output_file}")


if __name__ == "__main__":
    main()
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `profile`, and `combined`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `profile`、`combined` 等值。

## Key Concepts / 关键概念
- **EN**: Analyzes compiler, runtime, or profiling state for TorchInductor  
  **CN**: 分析 TorchInductor 的编译、运行期或性能画像状态
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `ProfileEvent`, `KernelStats`, `Device`, `JsonProfile`, and `ParseException`  
  **CN**: 主要类：`ProfileEvent`、`KernelStats`、`Device`、`JsonProfile`、`ParseException`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `math`, `collections`, `collections.abc`, `dataclasses`, `typing`, `argparse`, `os`, `tempfile`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.analysis.device_info`, `torch._inductor.utils`, `torch.utils`, `torch.utils._ordered_set`, `torch.utils.flop_counter`
