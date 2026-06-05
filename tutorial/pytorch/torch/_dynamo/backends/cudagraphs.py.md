# cudagraphs.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/backends/cudagraphs.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Connects captured graphs to compiler backends and post-processing pipelines.
- **Purpose (CN)**: 将捕获到的图连接到编译后端及后处理流水线。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
"""
This module implements CUDA graphs support for TorchDynamo backends.

CUDA graphs allow for capturing and replaying GPU operations, which can significantly
reduce CPU overhead in GPU-accelerated PyTorch models. This module provides:

- CUDA graph creation and management for both forward and backward passes
- Input mutation detection and handling
- Device compatibility checking
- Stack trace management for debugging
- Integration with TorchInductor's cudagraph trees

The backend supports two main modes:
1. cudagraphs: Full CUDA graph support with both forward and backward pass optimization
2. cudagraphs_inner: Lower-level CUDA graph implementation used for benchmarking
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 17-28
```python
Key components:
- CudagraphsBackend: Main backend class for CUDA graph integration
- Mutation detection utilities to ensure graph safety
- Device mapping and compatibility checks
- Stack trace collection for debugging
"""

import functools
from collections import defaultdict
from collections.abc import Callable, Sequence
from typing import Any
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 29-40
```python
import torch
import torch.fx
from torch._dynamo import config
from torch._dynamo.backends.common import aot_autograd
from torch._dynamo.backends.debugging import boxed_nop
from torch._inductor.cudagraph_utils import (
    BoxedDeviceIndex,
    check_multiple_devices_or_any_cpu_nodes,
    format_default_skip_message,
    get_mutation_stack_trace,
    get_placeholder_info,
    log_cudagraph_skip_and_bump_counter,
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 41-57
```python
)
from torch._inductor.utils import (
    BoxedBool,
    count_tangents,
    get_first_incompatible_cudagraph_node,
    num_fw_fixed_arguments,
    output_node,
)
from torch.multiprocessing.reductions import StorageWeakRef

from .registry import register_backend


def find_input_mutations(g: torch.fx.Graph) -> set[int]:
    def meta_fk(meta: dict[str, Any]) -> Any:
        return meta["val"] if "val" in meta else meta["fake_result"]
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 58-69
```python
    inputs = defaultdict(set)
    input_idx = 0
    mutated_inputs = set()
    for n in g.nodes:
        if n.op == "placeholder":
            if isinstance(meta_fk(n.meta), torch.Tensor):
                inputs[StorageWeakRef(meta_fk(n.meta)._typed_storage())].add(input_idx)
            input_idx += 1
        elif n.op == "call_function":
            if not hasattr(n.target, "_schema"):
                continue
```
- **EN**: This block continues `find_input_mutations` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `find_input_mutations`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 70-81
```python
            schema = n.target._schema
            for i, arg in enumerate(schema.arguments):
                if i < len(n.args):
                    argument = n.args[i]
                else:
                    if arg.name not in n.kwargs:
                        continue
                    argument = n.kwargs[arg.name]
                mut_arg = False
                if arg.alias_info:
                    if arg.alias_info.is_write:
                        mut_arg = True
```
- **EN**: This block continues `find_input_mutations` and works to capture Python execution for torch.compile and maintain compiler state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `find_input_mutations`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 82-93
```python
                if mut_arg:
                    # TODO: not correct for args that contain tensors in a struct
                    # like list
                    mutated_inputs |= inputs[
                        StorageWeakRef(meta_fk(argument.meta)._typed_storage())
                    ]

        # TODO: error on unrecognized nodes
    return mutated_inputs


def get_device_node_mapping(
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 94-110
```python
    gm: torch.fx.GraphModule,
) -> dict[torch.device, torch.fx.Node]:
    device_node_mapping: dict[torch.device, torch.fx.Node] = {}
    for n in gm.graph.nodes:
        t = n.meta.get("val", None)
        if isinstance(t, torch.Tensor) and t.device not in device_node_mapping:
            device_node_mapping[t.device] = n
    return device_node_mapping


def check_for_mutation_ignore_cuda_graph_managed_tensor(
    aot_model: torch.fx.GraphModule, num_fixed: int
) -> str | None:
    mutation_indices = find_input_mutations(aot_model.graph) - set(range(num_fixed))
    if not mutation_indices:
        return None
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 111-126
```python
    placeholders = get_placeholder_info(aot_model.graph)
    return get_mutation_stack_trace(placeholders, mutation_indices)


def check_for_skip(aot_model: torch.fx.GraphModule, num_fixed: int) -> str | None:
    if not config.cudagraph_backend_support_input_mutation:
        if mut_skip := check_for_mutation_ignore_cuda_graph_managed_tensor(
            aot_model, num_fixed
        ):
            return mut_skip

    if skip := check_multiple_devices_or_any_cpu_nodes(
        get_device_node_mapping(aot_model)
    ):
        return skip
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 127-138
```python
    if node := get_first_incompatible_cudagraph_node(aot_model):
        return format_default_skip_message(f"incompatible op ({node.name})")

    return None


def get_device_index(gm: torch.fx.GraphModule) -> int:
    device = next(iter(get_device_node_mapping(gm)))
    assert device.type == "cuda"
    return device.index
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 139-156
```python
def get_stack_traces(gm: torch.fx.GraphModule) -> list[str | None]:
    output = output_node(gm)
    assert len(output.args) == 1
    args = output.args[0]
    if not hasattr(args, "__iter__"):
        return []
    return [
        (arg.stack_trace if isinstance(arg, torch.fx.node.Node) else None)
        for arg in args  # type: ignore[union-attr]
    ]


def cudagraphs(dynamo_model: torch.fx.GraphModule, dynamo_inputs: Sequence[Any]) -> Any:
    from torch._inductor.cudagraph_trees import cudagraphify_impl

    do_cudagraphs = BoxedBool(True)
    boxed_device_index = BoxedDeviceIndex(None)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 157-170
```python
    def forward_cudagraphs(
        aot_model: torch.fx.GraphModule,
        aot_inputs: list[Any],
        is_inference: bool = False,
    ) -> Any:
        interp = boxed_nop(aot_model, aot_inputs)
        fixed = num_fw_fixed_arguments(len(dynamo_inputs), len(aot_inputs))
        if skip_msg := check_for_skip(aot_model, fixed):
            BoxedBool.disable(do_cudagraphs)
            log_cudagraph_skip_and_bump_counter(
                f"skipping cudagraphs due to {skip_msg}"
            )
            return interp
```
- **EN**: Defines the `cudagraphs.forward_cudagraphs` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`cudagraphs.forward_cudagraphs` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 171-185
```python
        boxed_device_index.set(get_device_index(aot_model))
        out = cudagraphify_impl(
            interp,
            aot_inputs,
            range(fixed),
            device_index=boxed_device_index.value,
            is_backward=False,
            is_inference=is_inference,
            stack_traces=get_stack_traces(aot_model),
            placeholders=get_placeholder_info(aot_model.graph),
            mutated_input_idxs=find_input_mutations(aot_model.graph),
        )
        out._boxed_call = True  # type: ignore[attr-defined]
        return out
```
- **EN**: This block continues `cudagraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `cudagraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 186-198
```python
    def backward_cudagraphs(
        aot_model: torch.fx.GraphModule, aot_inputs: list[Any]
    ) -> Any:
        interp = boxed_nop(aot_model, aot_inputs)
        if not do_cudagraphs:
            return aot_model

        fixed = count_tangents(aot_model)
        if skip_msg := check_for_skip(aot_model, fixed):
            log_cudagraph_skip_and_bump_counter(
                f"skipping cudagraphs due to {skip_msg}"
            )
```
- **EN**: Defines the `cudagraphs.backward_cudagraphs` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`cudagraphs.backward_cudagraphs` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 199-215
```python
            # See [Backward Generation Handling]
            device_idx = boxed_device_index.value
            if device_idx is None:
                device_idx = 0  # Default to device 0 if not set
            manager = torch._inductor.cudagraph_trees.get_manager(
                device_idx, create_if_none_exists=False
            )
            assert manager is not None

            def fn(inputs: list[Any]) -> Any:
                # pyrefly: ignore [missing-attribute]
                manager.set_to_running_backward()
                return aot_model(inputs)

            fn._boxed_call = True  # type: ignore[attr-defined]
            return fn
```
- **EN**: Defines the `cudagraphs.backward_cudagraphs` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`cudagraphs.backward_cudagraphs` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 216-229
```python
        out = cudagraphify_impl(
            interp,
            aot_inputs,
            range(fixed),
            device_index=get_device_index(aot_model),
            is_backward=True,
            is_inference=False,
            stack_traces=get_stack_traces(aot_model),
            placeholders=get_placeholder_info(aot_model.graph),
            mutated_input_idxs=find_input_mutations(aot_model.graph),
        )
        out._boxed_call = True  # type: ignore[attr-defined]
        return out
```
- **EN**: This block continues `cudagraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `cudagraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 230-247
```python
    aot_cudagraphs = aot_autograd(
        fw_compiler=forward_cudagraphs,
        bw_compiler=backward_cudagraphs,
        inference_compiler=functools.partial(forward_cudagraphs, is_inference=True),
        keep_inference_input_mutations=torch._dynamo.config.cudagraph_backend_keep_input_mutation,
    )
    return aot_cudagraphs(dynamo_model, dynamo_inputs)


class CudagraphsBackend:
    compiler_name = "cudagraphs"

    @staticmethod
    def reset() -> None:
        from torch._inductor.cudagraph_trees import reset_cudagraph_trees

        reset_cudagraph_trees()
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 248-259
```python
    @staticmethod
    def __call__(model: torch.fx.GraphModule, inputs: Sequence[Any]) -> Any:
        return cudagraphs(model, inputs)


# aot_cudagraphs only applies CUDA graphs to the graph.  It is also helpful
# for debugging and can serve as a perf baseline.
register_backend(name="cudagraphs", compiler_fn=CudagraphsBackend())


def cudagraphs_inner(
    model: Callable[..., Any],
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 260-271
```python
    inputs: Sequence[Any],
    copy_outputs: bool = True,
    copy_inputs: bool = True,
) -> Callable[..., Sequence[Any]]:
    """This isn't registered as a backend, but is used in some benchmarks"""
    assert isinstance(inputs, (list, tuple))
    if copy_inputs:
        static_inputs = [torch.zeros_like(x) for x in inputs]
    else:
        static_inputs = list(inputs)

    # warmup
```
- **EN**: This block continues `cudagraphs_inner` and works to hand work to a compiler/backend pipeline. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `cudagraphs_inner`，用于将工作移交给编译器或后端流水线。 同时它还会根据运行时条件分支处理。

### Lines 272-287
```python
    torch.cuda.synchronize()
    stream = torch.cuda.Stream()
    stream.wait_stream(torch.cuda.current_stream())
    with torch.cuda.stream(stream):
        model(*inputs)
    stream.synchronize()
    torch.cuda.current_stream().wait_stream(stream)
    torch.cuda.synchronize()

    # record
    graph = torch.cuda.CUDAGraph()
    with torch.cuda.graph(graph, stream=stream):
        static_outputs = model(*static_inputs)
    if not isinstance(static_outputs, (list, tuple)):
        static_outputs = (static_outputs,)
```
- **EN**: This block continues `cudagraphs_inner` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `cudagraphs_inner`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 288-299
```python
    def run(*new_inputs: Any) -> Sequence[Any]:
        assert len(static_inputs) == len(new_inputs)
        if copy_inputs:
            for dst, src in zip(static_inputs, new_inputs):
                dst.copy_(src)
        graph.replay()
        if copy_outputs:
            return [x.clone() for x in static_outputs]
        else:
            return static_outputs

    return run
```
- **EN**: Defines the `cudagraphs_inner` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`cudagraphs_inner` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
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
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx`, `torch._dynamo`, `torch._dynamo.backends.common`, `torch._dynamo.backends.debugging`, `torch._inductor.cudagraph_utils`, `torch._inductor.utils`, `torch.multiprocessing.reductions`, `.registry`, `torch._inductor.cudagraph_trees`
- **Standard library / 标准库**: `functools`, `collections`, `collections.abc`, `typing`
- **Primary symbols / 核心符号**: `find_input_mutations`, `get_device_node_mapping`, `check_for_mutation_ignore_cuda_graph_managed_tensor`, `check_for_skip`, `get_device_index`, `get_stack_traces`, `cudagraphs`, `CudagraphsBackend`, `cudagraphs_inner`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
