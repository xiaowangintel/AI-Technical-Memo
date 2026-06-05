# shape_propagation.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/shape_propagation.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `ShapeVar`, and `ShapePropagationOpsHandler`. It exposes functions such as `get_broadcasted_shape`, and `broadcast_shapes_for_args`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `ShapeVar`、`ShapePropagationOpsHandler` 等类。同时提供 `get_broadcasted_shape`、`broadcast_shapes_for_args` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import functools
from collections.abc import Callable, Sequence
from typing import Protocol

import sympy

import torch

from .virtualized import OpsValue, V


BlockShapeType = Sequence[int | str] | None


````
- **EN**: Imports dependencies such as `functools`, `collections.abc`, `typing`, `sympy`, `torch`, and `.virtualized` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `BlockShapeType`.
- **CN**: 这里导入了 `functools`、`collections.abc`、`typing`、`sympy`、`torch`、`.virtualized` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `BlockShapeType` 等值。

### Lines 15-28 / 第 15-28 行
````python
class ShapeVar(Protocol):
    @property
    def shape(self) -> BlockShapeType: ...


ShapeArg = ShapeVar | torch.types.Number | str | OpsValue | torch.dtype

# Inputs need to be cacheable (e.g., not a CSEVar) in order for the cache to be effective
# So first decompose CSEVars -> tuple before calling this


@functools.lru_cache(None)
def get_broadcasted_shape(a: BlockShapeType, b: BlockShapeType) -> BlockShapeType:
    assert isinstance(a, Sequence)
````
- **EN**: Introduces class `ShapeVar`, function `shape`, function `get_broadcasted_shape`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了类`ShapeVar`、函数`shape`、函数`get_broadcasted_shape`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 29-42 / 第 29-42 行
````python
    assert isinstance(b, Sequence)
    if len(a) > len(b):
        return get_broadcasted_shape(a, (*[1] * (len(a) - len(b)), *b))
    elif len(a) < len(b):
        b, a = a, b
        return get_broadcasted_shape(a, (*[1] * (len(a) - len(b)), *b))
    else:

        def _get_broadcasted_dim(d1: int | str, d2: int | str) -> int | str:
            if str(d1) == "1":
                return d2
            elif str(d2) == "1":
                return d1
            assert str(d1) == str(d2)
````
- **EN**: Introduces function `_get_broadcasted_dim`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`_get_broadcasted_dim`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 43-56 / 第 43-56 行
````python
            return d1

        return tuple(_get_broadcasted_dim(d1, d2) for d1, d2 in zip(a, b))


def broadcast_shapes_for_args(args: Sequence[ShapeArg]) -> BlockShapeType:
    result_shape: BlockShapeType = None

    for arg in args:
        if hasattr(arg, "shape"):
            shape = arg.shape
            if shape is None:
                return None
            elif result_shape is None:
````
- **EN**: Introduces function `broadcast_shapes_for_args`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `result_shape`, and `shape`.
- **CN**: 这里定义了函数`broadcast_shapes_for_args`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `result_shape`、`shape` 等值。

### Lines 57-70 / 第 57-70 行
````python
                result_shape = tuple(shape)
            else:
                result_shape = get_broadcasted_shape(result_shape, tuple(shape))
        elif isinstance(arg, (int, float)):
            if result_shape is None:
                result_shape = ()
        elif isinstance(arg, torch.dtype):
            continue
        else:
            from torch._inductor.loop_body import LoopBody, LoopBodyBlock

            if isinstance(arg, (LoopBodyBlock, LoopBody, OpsValue)):
                # TODO: fix me
                return None
````
- **EN**: Imports dependencies such as `torch._inductor.loop_body` for the logic in this range. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.loop_body` 等依赖，为后续逻辑提供基础能力。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 71-84 / 第 71-84 行
````python
            raise TypeError(f"Unknown type: {type(arg)}")

    return result_shape


class ShapePropagationOpsHandler:
    """
    Propagate shape from args to output
    """

    @staticmethod
    def constant(value: torch.types.Number, dtype: torch.dtype) -> BlockShapeType:
        # TritonKernelOverrides.constant uses tl.full with shape=[1]*ndim for all types
        from torch._inductor.codegen.triton import TritonKernel
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.triton` for the logic in this range. Introduces class `ShapePropagationOpsHandler`, function `constant`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `torch._inductor.codegen.triton` 等依赖，为后续逻辑提供基础能力。这里定义了类`ShapePropagationOpsHandler`、函数`constant`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 85-98 / 第 85-98 行
````python

        if isinstance(V.kernel, TritonKernel):
            ndim = V.kernel.triton_tensor_ndim()
            return tuple([1] * ndim)
        else:
            return ()

    @staticmethod
    def store_reduction(name: str, index: int, value: ShapeArg) -> None:
        return None

    @staticmethod
    def reduction(
        dtype: torch.dtype,
````
- **EN**: Introduces function `store_reduction`, function `reduction`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`store_reduction`、函数`reduction`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 99-112 / 第 99-112 行
````python
        src_dtype: torch.dtype,
        reduction_type: str,
        value: ShapeArg | tuple[ShapeArg, ...],
    ) -> BlockShapeType | tuple[BlockShapeType, ...]:
        raise NotImplementedError

    @staticmethod
    def store(name: str, index: int, value: ShapeArg, mode: str | None = None) -> None:
        return None

    @staticmethod
    def to_dtype(
        value: ShapeVar,
        dtype: torch.dtype,
````
- **EN**: Introduces function `store`, function `to_dtype`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`store`、函数`to_dtype`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 113-126 / 第 113-126 行
````python
        src_dtype: torch.dtype | None = None,
        use_compute_types: bool = True,
    ) -> BlockShapeType:
        return value.shape

    @staticmethod
    def dot(a: sympy.Expr, b: sympy.Expr) -> BlockShapeType:
        from torch._inductor.codegen.triton import TritonKernel

        assert isinstance(V.kernel, TritonKernel), "dot supports Triton only"
        return ("YBLOCK", "XBLOCK")

    @staticmethod
    def index_expr(expr: sympy.Expr, dtype: torch.dtype) -> BlockShapeType:
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.triton` for the logic in this range. Introduces function `dot`, function `index_expr`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `torch._inductor.codegen.triton` 等依赖，为后续逻辑提供基础能力。这里定义了函数`dot`、函数`index_expr`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。

### Lines 127-140 / 第 127-140 行
````python
        # shape is implicitly embedded in expr.
        return None

    @staticmethod
    def load_seed(name: str, offset: int) -> BlockShapeType:
        return ()

    @staticmethod
    def indirect_indexing(
        var: ShapeArg,
        size: sympy.Expr | int,
        check: bool = True,
        wrap_neg: bool = True,
    ) -> None:
````
- **EN**: Introduces function `load_seed`, function `indirect_indexing`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`load_seed`、函数`indirect_indexing`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-148 / 第 141-148 行
````python
        return None

    def __getattr__(self, name: str) -> Callable[..., BlockShapeType]:
        return lambda *args, **kwargs: broadcast_shapes_for_args(args)

    @staticmethod
    def device_assert_async(cond: ShapeArg, msg: str) -> None:
        return None
````
- **EN**: Introduces function `__getattr__`, function `device_assert_async`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`__getattr__`、函数`device_assert_async`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `ShapeVar`, and `ShapePropagationOpsHandler`  
  **CN**: 主要类：`ShapeVar`、`ShapePropagationOpsHandler`
- **EN**: Primary functions: `get_broadcasted_shape`, and `broadcast_shapes_for_args`  
  **CN**: 主要函数：`get_broadcasted_shape`、`broadcast_shapes_for_args`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `collections.abc`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `.virtualized`, `torch._inductor.codegen.triton`, `torch._inductor.loop_body`
