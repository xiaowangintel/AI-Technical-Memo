# extract_compiled_graph.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_lazy/extract_compiled_graph.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides lazy-backend hooks and Python helpers for deferred execution.
- **Purpose (CN)**: 提供 lazy 后端钩子以及用于延迟执行的 Python 辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
````python
# mypy: allow-untyped-defs
import copy
import dataclasses
import itertools
import os
from collections.abc import Callable
from typing import Any

import torch
import torch._lazy as lazy
import torch._lazy.metrics as metrics
from torch import fx
from torch._lazy import computation, debug as lazy_debug
from torch._lazy.tensor_factory_functions import tensor_factory_functions


debug = os.environ.get("debug_extract_compiled_graph") is not None
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._lazy, torch._lazy.metrics, ...; standard-library helpers such as copy, dataclasses, itertools, ....
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._lazy、torch._lazy.metrics、...；标准库辅助模块，如 copy、dataclasses、itertools、...。

### Lines 20-39 / 第 20-39 行
````python
@dataclasses.dataclass
class GraphInputMatcher:
    """
    The GraphInputMatcher class setup the graph inputs for future calls after lazy tracing.
    Specifically, those graph inputs corresponding to method parameters should be replaced with the
    arguments for the current call.

    tensor_id_to_arg_idx maps the tensor id to the parameter index.
    graph_input_tensor_ids, graph_input_ivalues list the tensor_id and ivalue for each of the
    TS/XLA graph inputs.
    """

    tensor_id_to_arg_idx: dict[int, int]
    graph_input_tensor_ids: list[int]
    # there are 2 categories of graph_input_tensors.
    # Category 1: those whose id are not found in tensor_id_to_arg_idx. These are
    # most likely const tensors and we can get its content from graph_input_tensors
    # Category 2: those whose id are found in tensor_id_to_arg_idx. We should get
    #  the tensor from method arguments
    graph_input_ivalues: list[Any]
````
- **EN**: It introduces or extends `GraphInputMatcher`, which hold the main object-oriented state for this portion of the file. Decorators such as `dataclasses.dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 它引入或扩展了 `GraphInputMatcher`，这些类承载了本段涉及的主要面向对象状态。 像 `dataclasses.dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 41-53 / 第 41-53 行
````python
    # get the real graph input tensors
    def __call__(self, args):
        real_input = []
        for tensor_id, traced_ivalue in zip(
            self.graph_input_tensor_ids, self.graph_input_ivalues
        ):
            arg_idx = self.tensor_id_to_arg_idx.get(tensor_id, None)
            if arg_idx is None:
                inp = traced_ivalue
            else:
                inp = args[arg_idx]
            real_input.append(inp)
        return real_input
````
- **EN**: This chunk defines `__call__`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__call__`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 56-72 / 第 56-72 行
````python
class ReturnValueHandler:
    r"""
    When ltc_sync_multi is called on multi tensors, the compiled graph
    will contain output only for unique tensors - if a tensor appears multiple
    times in the input to _ltc_sync_multi, only the first occurrence matters.

    However from python level, we still expect multi tensors returned with duplication
    even if the TS graph dedup the output. e.g. for method:

      def forward(self, a):
        return a, a

    the TS graph captured by LTC will return a single tensor, but Python method expects 2.

    This class dedup the lazy tensors first to get the index that will be used
    to duplicate the eager tensors later.
    """
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. It introduces or extends `ReturnValueHandler`, which hold the main object-oriented state for this portion of the file. This chunk defines `forward`, which implements a focused helper used by the surrounding module.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 它引入或扩展了 `ReturnValueHandler`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 74-86 / 第 74-86 行
````python
    def __init__(self, lazy_out_list):
        self.index: list[list[int]] = []
        self.total_count = len(lazy_out_list)

        tensor_id_to_idx: dict[int, int] = {}
        for dup_idx, lazy_tensor in enumerate(lazy_out_list):
            uniq_idx = tensor_id_to_idx.get(id(lazy_tensor))
            if uniq_idx is not None:
                self.index[uniq_idx].append(dup_idx)
            else:
                uniq_idx = len(self.index)
                self.index.append([dup_idx])
                tensor_id_to_idx[id(lazy_tensor)] = uniq_idx
````
- **EN**: This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 88-106 / 第 88-106 行
````python
    def duplicate_eager_tensors(self, eager_tensor_list):
        duplicated_list = [None] * self.total_count
        if len(eager_tensor_list) != len(self.index):
            raise AssertionError(
                f"eager_tensor_list length {len(eager_tensor_list)} != index length {len(self.index)}"
            )

        for uniq_idx, eager_tensor in enumerate(eager_tensor_list):
            for dup_idx in self.index[uniq_idx]:
                duplicated_list[dup_idx] = eager_tensor
        return duplicated_list


def force_lazy_device(model: fx.GraphModule):
    """
    Factory methods in a Fx graph may create tensors for a specific eager devices.
    If we take no actions, those eager tensors will be mixed with lazy tensors and
    cause crash. This method overwrite those eager device to lazy device.
    """
````
- **EN**: This chunk defines `force_lazy_device`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `force_lazy_device`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 108-121 / 第 108-121 行
````python
    def tolazydevice(dev):
        if isinstance(dev, torch.device):
            return torch.device("lazy", index=dev.index)
        return dev

    def hasDeviceArg(args, kwargs):
        return any(
            isinstance(arg, torch.device)
            for arg in itertools.chain(args, kwargs.values())
        )

    for nd in model.graph.nodes:
        nd.args = tuple(tolazydevice(arg) for arg in nd.args)
        nd.kwargs = {k: tolazydevice(v) for k, v in nd.kwargs.items()}
````
- **EN**: This chunk defines `hasDeviceArg`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `hasDeviceArg`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 123-141 / 第 123-141 行
````python
        # For torchbench like yolov3, hf_Bart, dynamo generates Fx graph that return
        # eager tensors on the default device
        # (check https://gist.github.com/shunting314/eabdf6c769c59bc384469717b8f9bb7f for yolove,
        # and https://gist.github.com/shunting314/8d5e2d9348a3258959d3954186c48814 for hf_Bart).
        # To force those tensors on the lazy device, we can not simply override
        # the device argument since there is no explicit device argument.
        # What we are doing here is, for the list of covered tensor factory methods
        # we add a lazy device argument explicitly.
        #
        # TODO: This solution is no ideal since we may miss some factory methods. In future
        # when we support lazy mode, this method can be replaced by that.
        if nd.target in tensor_factory_functions and not hasDeviceArg(
            nd.args, nd.kwargs
        ):
            kwargs = dict(nd.kwargs)  # nd.kwargs is immutable. make a mutable copy.
            kwargs["device"] = torch.device("lazy")
            nd.kwargs = kwargs

    model.recompile()
````
- **EN**: This chunk continues `hasDeviceArg` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `hasDeviceArg`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 144-163 / 第 144-163 行
````python
def get_fallback_ops():
    fallback_ops = []
    for opname in metrics.counter_names():
        if "aten::" not in opname:
            continue
        val = int(metrics.counter_value(opname))
        if val > 0:
            fallback_ops.append(f"{opname}={val}")

    return fallback_ops


def extract_compiled_graph(model: fx.GraphModule, example_inputs) -> Callable:
    """
    Optimize an eager model with LTC and returns a wrapper to execute the
    compiled graph directly without retracing. It depends on other mechanisms
    like TorchDynamo guards to guarantee the returned wrapper is only called
    when it's safe.
    """
    lazy_args = [arg.to(device="lazy") for arg in example_inputs]
````
- **EN**: This chunk defines `extract_compiled_graph`, which coordinates graph-oriented state needed for tracing, capture, or compilation. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `extract_compiled_graph`，其作用是协调 tracing、捕获或编译所需的图相关状态。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 164-181 / 第 164-181 行
````python
    args_tensor_ids = [lazy.get_tensor_id(lazy_arg) for lazy_arg in lazy_args]
    tensor_id_to_arg_idx = {tensor_id: i for i, tensor_id in enumerate(args_tensor_ids)}
    lazy_model = copy.deepcopy(model).to(device=torch.device("lazy"))
    force_lazy_device(lazy_model)

    # This line executes lazy tracing and enable us extracting compiled graph later
    metrics.reset()
    lazy_out = lazy_model(*lazy_args)
    fallback_ops = get_fallback_ops()
    metrics.reset()

    if len(fallback_ops) > 0:
        raise RuntimeError(
            f"Fail to extract the compiled graph because of fallback: {','.join(fallback_ops)}"
        )

    if not isinstance(lazy_out, (tuple, list)):
        lazy_out = (lazy_out,)
````
- **EN**: This chunk continues `extract_compiled_graph` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `extract_compiled_graph`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 183-202 / 第 183-202 行
````python
    args_and_out = tuple(lazy_args) + tuple(lazy_out)
    return_value_handler = ReturnValueHandler(args_and_out)
    if debug:
        print("Fx code:\n", model.code)
        print("LTC IR:", lazy_debug.dump_ir(args_and_out, "text"))

    # TODO: this part is TS backend specific for now and will be generalized to
    # support XLA
    (
        graph_input_tensor_ids,
        graph_input_ivalues,
    ) = computation.get_tensors_ts_device_data_node(args_and_out)
    if len(graph_input_tensor_ids) != len(graph_input_ivalues):
        raise AssertionError(
            f"graph_input_tensor_ids length {len(graph_input_tensor_ids)} "
            f"!= graph_input_ivalues length {len(graph_input_ivalues)}"
        )
    graph_input_matcher = GraphInputMatcher(
        tensor_id_to_arg_idx, graph_input_tensor_ids, graph_input_ivalues
    )
````
- **EN**: This chunk continues `extract_compiled_graph` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `extract_compiled_graph`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 204-222 / 第 204-222 行
````python
    graph_hash = computation.get_graph_hash(args_and_out)

    if debug:
        print("graph_hash", graph_hash)
        print(f"args_tensor_ids {args_tensor_ids}")
        print("tensor ids from device data:", graph_input_tensor_ids)

    # sync the list of output tensors so the computation graph for these
    # tensors will be cached. Those computation graphs can be retrieved
    # by graph hash later.
    lazy.sync_multi(args_and_out, [])

    def optimized_mod(*args):
        if len(args_and_out) == 0:
            return ()
        graph_input = graph_input_matcher(args)
        res = return_value_handler.duplicate_eager_tensors(
            computation.run_cached_graph(graph_hash, graph_input)
        )
````
- **EN**: This chunk defines `optimized_mod`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `optimized_mod`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 224-236 / 第 224-236 行
````python
        if len(res) != len(args_and_out):
            raise AssertionError(
                f"result length {len(res)} != args_and_out length {len(args_and_out)}"
            )
        for i, arg in enumerate(args):
            # only copy those tensors that get inplace updated
            if arg is not res[i]:
                arg.copy_(res[i])

        # skip the args
        return res[len(args) :]

    return optimized_mod
````
- **EN**: This chunk continues `optimized_mod` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `optimized_mod`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Lazy execution**
  - EN: Defers work and records intent until a later backend-specific execution stage.
  - CN: 延迟实际工作并记录执行意图，直到后续后端阶段再真正执行。
- **GraphInputMatcher**
  - EN: `GraphInputMatcher` is one of the main symbols declared or implemented in this file.
  - CN: `GraphInputMatcher` 是本文件声明或实现的主要符号之一。
- **ReturnValueHandler**
  - EN: `ReturnValueHandler` is one of the main symbols declared or implemented in this file.
  - CN: `ReturnValueHandler` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Deferred execution**
  - EN: The file records intent or metadata now so execution can be materialized later.
  - CN: 该文件先记录意图或元数据，等待后续阶段再真正执行。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._lazy`, `torch._lazy.metrics`, `torch._lazy.tensor_factory_functions`
- **Standard library / 标准库**: `copy`, `dataclasses`, `itertools`, `os`, `collections.abc`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `GraphInputMatcher`, `ReturnValueHandler`, `force_lazy_device`, `get_fallback_ops`, `extract_compiled_graph`
