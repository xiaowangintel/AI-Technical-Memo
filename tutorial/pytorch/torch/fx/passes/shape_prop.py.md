# shape_prop.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/shape_prop.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
import traceback
from typing import Any, NamedTuple

import torch
import torch.fx
from torch._dispatch.python import enable_python_dispatcher
from torch._guards import detect_fake_mode
from torch._prims_common import is_contiguous_for_memory_format_or_false
from torch._subclasses.meta_utils import is_sparse_any
from torch.fx._compatibility import compatibility
from torch.fx.node import map_aggregate, Node


__all__ = ["TensorMetadata", "ShapeProp"]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 17-33
```python
@compatibility(is_backward_compatible=True)
class TensorMetadata(NamedTuple):
    # TensorMetadata is a structure containing pertinent information
    # about a tensor within a PyTorch program.

    # General Tensor metadata
    shape: torch.Size
    dtype: torch.dtype
    requires_grad: bool
    stride: tuple[int, ...]
    memory_format: torch.memory_format | None

    # Quantization metadata
    is_quantized: bool
    qparams: dict[str, Any]
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 34-50
```python
# When include_contiguity is True, we will set contiguity when its always true for the tensor.
# Some tensors can represent both contiguous and non-contiguous tensors. e.g: (u0, u1) with (u2, u3).
# In such situation contiguity is not set. We could also make it a tri-state i.e: (def_contiguous,
# def_not_contiguous and unknown).
def _extract_tensor_metadata(
    result: torch.Tensor, include_contiguity=True
) -> TensorMetadata:
    """
    Extract a TensorMetadata NamedTuple describing `result`.
    """
    shape = result.shape
    dtype = result.dtype
    requires_grad = result.requires_grad
    stride = result.stride() if not is_sparse_any(result) else ()

    memory_format = None
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 51-63
```python
    if include_contiguity and not is_sparse_any(result):
        memory_formats = (
            torch.contiguous_format,
            torch.channels_last,
            torch.channels_last_3d,
        )
        for query_format in memory_formats:
            if is_contiguous_for_memory_format_or_false(
                result, memory_format=query_format
            ):
                memory_format = query_format
                break
```
- **EN**: This block continues `_extract_tensor_metadata` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_extract_tensor_metadata`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 64-75
```python
    is_quantized = result.is_quantized
    qparams: dict[str, Any] = {}
    if is_quantized:
        qscheme = result.qscheme()
        qparams["qscheme"] = qscheme
        if qscheme in (torch.per_tensor_affine, torch.per_tensor_symmetric):
            qparams["scale"] = result.q_scale()  # type: ignore[assignment]
            qparams["zero_point"] = result.q_zero_point()  # type: ignore[assignment]
        elif qscheme in (
            torch.per_channel_affine,
            torch.per_channel_affine_float_qparams,
            torch.per_channel_symmetric,
```
- **EN**: This block continues `_extract_tensor_metadata` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `_extract_tensor_metadata`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会根据运行时条件分支处理。

### Lines 76-88
```python
        ):
            # In this branch, scale and zero_point are expected to be tensors,
            # we store the values as immutable_list in TensorMetadata for
            # easier serialization downstream
            qparams["scale"] = result.q_per_channel_scales().tolist()  # type: ignore[assignment]
            qparams["zero_point"] = result.q_per_channel_zero_points().tolist()  # type: ignore[assignment]
            qparams["axis"] = result.q_per_channel_axis()  # type: ignore[assignment]

    return TensorMetadata(
        shape, dtype, requires_grad, stride, memory_format, is_quantized, qparams
    )
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 89-101
```python
@compatibility(is_backward_compatible=True)
class ShapeProp(torch.fx.Interpreter):
    """
    Execute an FX graph Node-by-Node and
    record the shape and type of the result
    into the corresponding node.

    Example:
         In this example, we record the shape
         and data type of a module given
         an example input ``torch.randn(50, D_in)``.
         We print the name, shape and dtype of each node.
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 102-118
```python
        class TwoLayerNet(torch.nn.Module):
            def __init__(self, D_in, H, D_out):
                super().__init__()
                self.linear1 = torch.nn.Linear(D_in, H)
                self.linear2 = torch.nn.Linear(H, D_out)
            def forward(self, x):
                h_relu = self.linear1(x).clamp(min=0)
                y_pred = self.linear2(h_relu)
                return y_pred
        N, D_in, H, D_out = 64, 1000, 100, 10
        x = torch.randn(N, D_in)
        y = torch.randn(N, D_out)
        model = TwoLayerNet(D_in, H, D_out)
        gm = torch.fx.symbolic_trace(model)
        sample_input = torch.randn(50, D_in)
        ShapeProp(gm).propagate(sample_input)
```
- **EN**: Declares `ShapeProp(torch.fx.Interpreter)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `ShapeProp(torch.fx.Interpreter)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 119-136
```python
        for node in gm.graph.nodes:
            print(node.name, node.meta['tensor_meta'].dtype,
                node.meta['tensor_meta'].shape)

        The output of this code is:

        x torch.float32 torch.Size([50, 1000])
        linear1 torch.float32 torch.Size([50, 100])
        clamp_1 torch.float32 torch.Size([50, 100])
        linear2 torch.float32 torch.Size([50, 10])
        output torch.float32 torch.Size([50, 10])

    Args:
         module (GraphModule): The module to be executed
         fake_mode (FakeTensorMode): A fake mode for copying the gm

    """
```
- **EN**: This block continues `ShapeProp` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `ShapeProp`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构。

### Lines 137-148
```python
    def __init__(self, gm, fake_mode=None):
        super().__init__(gm)
        if fake_mode is None:
            fake_mode = detect_fake_mode()
        if fake_mode is not None:
            from torch._dynamo.utils import deepcopy_to_fake_tensor

            # Note:
            # We need fake execution cause the inputs are fake, however, we cannot fakify the module
            # - because we need to write to the tensor_meta of the real module. So we fakify to
            # produce a result (L131 below), to extract tensor meta, and then keep going.
            #
```
- **EN**: Defines the `ShapeProp.__init__` method; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`ShapeProp.__init__` 方法；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 149-166
```python
            # If we were to fakify, we would write to the wrong node, and then downstream fusion
            # would be missing the tensor_meta.
            #
            # See torch/_inductor/overrides.py for where this is called upstream of fusion.
            self.fake_module = deepcopy_to_fake_tensor(self.module, fake_mode)
            self.fake_mode = fake_mode
        else:
            self.fake_module = None
            self.fake_mode = None

        self.real_module = self.module

    def run_node(self, n: Node) -> Any:
        from torch.fx.experimental.symbolic_shapes import (
            compute_unbacked_bindings,
            rebind_unbacked,
        )
```
- **EN**: Declares `ShapeProp(torch.fx.Interpreter)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `ShapeProp(torch.fx.Interpreter)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 167-178
```python
        try:
            if self.fake_module is not None:
                # Hacky swap. Alternatively, we could do this with overriding
                # call_module and get_attr.
                self.module = self.fake_module
            try:
                if self.fake_mode is not None:
                    with self.fake_mode, enable_python_dispatcher():
                        result = super().run_node(n)
                        rebind_unbacked(self.fake_mode.shape_env, n, result)
                else:
                    result = super().run_node(n)
```
- **EN**: This block continues `ShapeProp.run_node` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `ShapeProp.run_node`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 179-196
```python
            finally:
                self.module = self.real_module
        except Exception as e:
            traceback.print_exc()
            raise RuntimeError(
                f"ShapeProp error for: node={n.format_node()} with meta={n.meta}"
            ) from e

        found_tensor = False

        def extract_tensor_meta(obj):
            if isinstance(obj, torch.Tensor):
                nonlocal found_tensor
                found_tensor = True
                return _extract_tensor_metadata(obj)
            else:
                return obj
```
- **EN**: Defines the `ShapeProp.run_node` method; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`ShapeProp.run_node` 方法；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 197-214
```python
        meta = map_aggregate(result, extract_tensor_meta)
        if found_tensor:
            n.meta["tensor_meta"] = meta

        if self.fake_mode:
            if (shape_env := self.fake_mode.shape_env) and (
                symbol_to_path := compute_unbacked_bindings(shape_env, result)
            ):
                n.meta["unbacked_bindings"] = symbol_to_path

        n.meta["type"] = type(result)
        return result

    def propagate(self, *args):
        """
        Run `module` via interpretation and return the result and
        record the shape and type of each node.
```
- **EN**: Declares `ShapeProp(torch.fx.Interpreter)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `ShapeProp(torch.fx.Interpreter)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 215-226
```python
        Args:
            *args (Tensor): the sample input.

        Returns:
            Any: The value returned from executing the Module
        """
        if self.fake_mode is not None:
            fake_args = [
                self.fake_mode.from_tensor(t) if isinstance(t, torch.Tensor) else t
                for t in args
            ]
        else:
```
- **EN**: This block continues `ShapeProp.propagate` and works to organize reusable module behavior and state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `ShapeProp.propagate`，用于组织可复用的模块行为与状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 227-228
```python
            fake_args = args
        return super().run(*fake_args)
```
- **EN**: This block continues `ShapeProp.propagate` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `ShapeProp.propagate`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx`, `torch._dispatch.python`, `torch._guards`, `torch._prims_common`, `torch._subclasses.meta_utils`, `torch.fx._compatibility`, `torch.fx.node`, `torch.fx.experimental.symbolic_shapes`, `torch._dynamo.utils`
- **Standard library / 标准库**: `traceback`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `TensorMetadata`, `_extract_tensor_metadata`, `ShapeProp`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
