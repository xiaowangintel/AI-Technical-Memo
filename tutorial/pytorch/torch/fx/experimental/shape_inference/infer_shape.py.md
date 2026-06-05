# infer_shape.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/shape_inference/infer_shape.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
import copy
from collections import defaultdict

import torch
from torch._dynamo.source import LocalSource
from torch._subclasses import FakeTensorMode
from torch.fx.experimental.proxy_tensor import make_fx
from torch.fx.experimental.shape_inference.infer_symbol_values import (
```
- **EN**: This import block pulls in the dependencies used by later definitions, including `copy`, `collections`, `torch`, `torch._dynamo.source`.
- **CN**: 这一导入块引入后续定义所需的依赖，其中包括 `copy`, `collections`, `torch`, `torch._dynamo.source`。

### Lines 9-20
```python
    infer_symbol_values,
)
from torch.fx.experimental.symbolic_shapes import DimDynamic, ShapeEnv
from torch.types import IntLikeType
from torch.utils import _pytree


"""
This is the function that runs shape inference. It will modify the input graph module so that shapes are annotated.
"""
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 21-29
```python
def infer_shape(
    gm: torch.fx.GraphModule, input_tensors: list[torch.Tensor]
) -> (
    tuple[torch.fx.GraphModule, list[torch.Tensor], FakeTensorMode, IntLikeType] | None
):
    # Prepare environments
    shape_env = ShapeEnv()
    fake_mode = FakeTensorMode(shape_env=shape_env, allow_non_fake_inputs=True)
```
- **EN**: Defines the `infer_shape` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`infer_shape` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 30-37
```python
    flatten_inputs, spec = _pytree.tree_flatten(input_tensors)
    dim_count = 1
    for input_tensor in flatten_inputs:
        dim_count += input_tensor.dim() - 1

    sample = {f"s{i}": 2 for i in range(dim_count)}
    init_symints: list[IntLikeType] = [
        mksym(shape_env, v, LocalSource(k), DimDynamic.DYNAMIC)
```
- **EN**: This block continues `infer_shape` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `infer_shape`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构。

### Lines 38-46
```python
        for k, v in sample.items()
    ]
    symints: list[IntLikeType] = copy.deepcopy(init_symints)
    symbol_to_idx_dict = {f"s{i}": i for i in range(dim_count)}
    padding_constraints = defaultdict(list)  # type: ignore[var-annotated]

    complete_flag = False
    allowed_try_times = dim_count * 2
```
- **EN**: This block continues `infer_shape` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `infer_shape`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构。

### Lines 47-54
```python
    while not complete_flag and allowed_try_times > 0:
        # Create symbolic input tensors
        with fake_mode:
            sym_tensors = []
            i = 1
            for input_tensor in flatten_inputs:
                curr_dim = input_tensor.dim()
                desired_size = [symints[0]] + [
```
- **EN**: This block continues `infer_shape` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `infer_shape`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；使用带作用域的辅助对象或上下文管理器。

### Lines 55-62
```python
                    symints[ii] for ii in range(i, i + curr_dim - 1)
                ]
                sym_tensor = torch.randn(desired_size)
                sym_tensors.append(sym_tensor)
                i += curr_dim - 1
            sym_tensors = _pytree.tree_unflatten(sym_tensors, spec)
        try:
            with fake_mode:
```
- **EN**: This block continues `infer_shape` and works to track symbolic shape constraints and shape-dependent reasoning. It also uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `infer_shape`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会使用带作用域的辅助对象或上下文管理器。

### Lines 63-70
```python
                make_fx(
                    gm,
                    tracing_mode="symbolic",
                    _allow_non_fake_inputs=True,
                    pre_dispatch=True,
                    _allow_fake_constant=True,
                )(*sym_tensors)
            complete_flag = True
```
- **EN**: This block continues `infer_shape` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `infer_shape`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 71-78
```python
            return (gm, input_tensors, fake_mode, symints[0])
        except RuntimeError as e:
            if e:
                infer_symbol_values(
                    symints,
                    init_symints,
                    symbol_to_idx_dict,
                    padding_constraints,
```
- **EN**: This block continues `infer_shape` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `infer_shape`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 79-86
```python
                    str(e),
                )
                allowed_try_times -= 1
        except ValueError as e:
            if e:
                infer_symbol_values(
                    symints,
                    init_symints,
```
- **EN**: This block continues `infer_shape` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `infer_shape`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 87-94
```python
                    symbol_to_idx_dict,
                    padding_constraints,
                    str(e),
                )
                allowed_try_times -= 1


def mksym(
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。

### Lines 95-102
```python
    shape_env: ShapeEnv, value: int, source: LocalSource, dynamic_dim: DimDynamic
) -> IntLikeType:
    return shape_env.create_symintnode(
        shape_env.create_symbol(
            value,
            source=source,
            dynamic_dim=dynamic_dim,
        ),
```
- **EN**: This block continues `mksym` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `mksym`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 103-105
```python
        hint=value,
        source=source,
    )
```
- **EN**: This block continues `mksym` and works to manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 该代码块继续实现 `mksym`，用于操作 FX 图、跟踪辅助逻辑或变换工具。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
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
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._dynamo.source`, `torch._subclasses`, `torch.fx.experimental.proxy_tensor`, `torch.fx.experimental.shape_inference.infer_symbol_values`, `torch.fx.experimental.symbolic_shapes`, `torch.types`, `torch.utils`
- **Standard library / 标准库**: `copy`, `collections`
- **Primary symbols / 核心符号**: `infer_shape`, `mksym`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
