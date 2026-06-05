# python_evt.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cutlass/python_evt.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `CutlassEVTOpsMixIn`, `MockCutlassHandler`, `_AssignmentFormatter`, and `CutlassEVTCodegen`. It exposes functions such as `scaled_mm_evt`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `CutlassEVTOpsMixIn`、`MockCutlassHandler`、`_AssignmentFormatter`、`CutlassEVTCodegen` 等类。同时提供 `scaled_mm_evt` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import itertools
from collections.abc import Generator, Iterable, Iterator, Sequence
from contextlib import contextmanager
from os import linesep
from typing import Any

import sympy

import torch
import torch._inductor.virtualized as virtualized
from torch._inductor.ir import ComputedBuffer, Pointwise
from torch._inductor.ops_handler import DefaultHandler, WrapperHandler
from torch._inductor.scheduler import BaseSchedulerNode
from torch._inductor.utils import DelayReplaceLine, IndentedBuffer, OrderedSet
from torch._inductor.virtualized import OpsValue

from ...virtualized import V


_ACCUMULATOR_ARG_NAME = "accum"
````
- **EN**: Imports dependencies such as `itertools`, `collections.abc`, `contextlib`, `os`, `typing`, `sympy`, and `...+7` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `_ACCUMULATOR_ARG_NAME`.
- **CN**: 这里导入了 `itertools`、`collections.abc`、`contextlib`、`os`、`typing`、`sympy`、`另有7项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `_ACCUMULATOR_ARG_NAME` 等值。

### Lines 21-40 / 第 21-40 行
````python


def scaled_mm_evt(
    scale_A_name: str, scale_B_name: str, bias_name: str | None, output_name: str
) -> tuple[list[str], dict[str, Any], str]:
    evt_read_names = [scale_A_name, scale_B_name]
    var_name_to_buffer_name = {n: n for n in [scale_A_name, scale_B_name]}
    var_name_to_buffer_name["D"] = output_name
    var_name_to_buffer_name[_ACCUMULATOR_ARG_NAME] = output_name
    expr = f"accum * {scale_A_name} * {scale_B_name}{linesep}"
    if bias_name:
        expr = f"({expr}) + {bias_name}"
        evt_read_names.append(bias_name)
        var_name_to_buffer_name[bias_name] = bias_name

    evt_py_code = f"def fn(accum, {','.join(evt_read_names)}):{linesep}\
    D = {expr}{linesep}\
    return D{linesep}"

    return evt_read_names, var_name_to_buffer_name, evt_py_code
````
- **EN**: Introduces function `scaled_mm_evt`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scale_A_name`, `evt_read_names`, `var_name_to_buffer_name`, `expr`, `evt_py_code`, and `D`.
- **CN**: 这里定义了函数`scaled_mm_evt`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scale_A_name`、`evt_read_names`、`var_name_to_buffer_name`、`expr`、`evt_py_code`、`D` 等值。

### Lines 41-60 / 第 41-60 行
````python


class CutlassEVTOpsMixIn:
    @staticmethod
    def _infix_bin_op(op: str, a: str, b: str) -> str:
        return f"{a} {op} {b}"

    @staticmethod
    def _prefix_bin_op(op: str, a: str, b: str) -> str:
        return f"{op}({a}, {b})"

    @staticmethod
    def _prefix_un_op(op: str, a: str) -> str:
        return f"{op}({a})"

    @staticmethod
    def to_dtype(
        x: str,
        dtype: Any,
        src_dtype: torch.dtype | None = None,
````
- **EN**: Introduces class `CutlassEVTOpsMixIn`, function `_infix_bin_op`, function `_prefix_bin_op`, function `_prefix_un_op`, function `to_dtype`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`CutlassEVTOpsMixIn`、函数`_infix_bin_op`、函数`_prefix_bin_op`、函数`_prefix_un_op`、函数`to_dtype`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python
        use_compute_types: bool = False,
    ) -> str:
        return x

    @staticmethod
    def constant(value: Any, dtype: Any) -> str:
        raise NotImplementedError

    @staticmethod
    def mul(x0: str, x1: str) -> str:
        return CutlassEVTOpsMixIn._infix_bin_op("*", x0, x1)

    @staticmethod
    def truediv(x0: str, x1: str) -> str:
        return CutlassEVTOpsMixIn._infix_bin_op("/", x0, x1)

    @staticmethod
    def ge(x0: str, x1: str) -> str:
        raise NotImplementedError

````
- **EN**: Introduces function `constant`, function `mul`, function `truediv`, function `ge`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`constant`、函数`mul`、函数`truediv`、函数`ge`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 81-100 / 第 81-100 行
````python
    @staticmethod
    def add(x0: str, x1: str) -> str:
        return CutlassEVTOpsMixIn._infix_bin_op("+", x0, x1)

    @staticmethod
    def relu(x0: str) -> str:
        return CutlassEVTOpsMixIn._prefix_un_op("relu", x0)

    @staticmethod
    def sigmoid(x0: str) -> str:
        return CutlassEVTOpsMixIn._prefix_un_op("sigmoid", x0)

    @staticmethod
    def sub(x0: str, x1: str) -> str:
        return CutlassEVTOpsMixIn._infix_bin_op("-", x0, x1)

    @staticmethod
    def tanh(x0: str) -> str:
        return CutlassEVTOpsMixIn._prefix_un_op("tanh", x0)

````
- **EN**: Introduces function `add`, function `relu`, function `sigmoid`, function `sub`, function `tanh`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`add`、函数`relu`、函数`sigmoid`、函数`sub`、函数`tanh`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
    @staticmethod
    def exp(x0: str) -> str:
        return CutlassEVTOpsMixIn._prefix_un_op("exp", x0)


class MockCutlassHandler(CutlassEVTOpsMixIn, WrapperHandler):
    """Passthrough handler for cutlass ops, used for running epilogue nodes for memory planning"""


class _AssignmentFormatter(DefaultHandler):
    def __init__(self, parent_handler: "CutlassEVTCodegen"):
        self.parent_handler = parent_handler

    def _default(self, name: str, args: tuple[Any, ...], kwargs: dict[str, Any]) -> Any:
        # Handle op dispatch here
        if hasattr(self.parent_handler, name):
            fn = getattr(self.parent_handler, name)
            line = fn(*args, **kwargs)
            if name in ("load", "store"):
                return OpsValue(line)
````
- **EN**: Introduces function `exp`, class `MockCutlassHandler`, class `_AssignmentFormatter`, function `__init__`, function `_default`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`exp`、类`MockCutlassHandler`、类`_AssignmentFormatter`、函数`__init__`、函数`_default`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
            else:
                var = self.parent_handler._tmp_var()
                line = DelayReplaceLine(
                    var,
                    lambda: "D"
                    if var == self.parent_handler.last_stored_var_name
                    else var,
                    f"{var} = {line}",
                )
                self.parent_handler.body.writeline(line)
                return OpsValue(var)
        else:
            raise NotImplementedError(name)


class CutlassEVTCodegen(CutlassEVTOpsMixIn):
    """
    Notes:
        * Used by CUTLASSGemmTemplate.
        * This class should not be instantiated by users, it is intended to be used
````
- **EN**: Introduces class `CutlassEVTCodegen`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `var`, `line`, `lambda`, and `Notes`.
- **CN**: 这里定义了类`CutlassEVTCodegen`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`var`、`line`、`lambda`、`Notes` 等值。

### Lines 141-160 / 第 141-160 行
````python
            by calling CutlassEVTCodegen.ir_to_evt_python_code(...)
            which instantiates this class as an ops handler for virtualized.V.ops.[op-name]
        * Extend this with more _op_<whatever> nodes to add support for new pointwise operations.
    """

    def __init__(self, accumulator_node_name: str, removed_buffers: OrderedSet[str]):
        """

        Initializes a CutlassEVTEpilogueArgumentFormatter object. Do not instantiate directly.
        Use the CutlassEVTCodegen.ir_to_evt_python_code static method.

        Args:
            accumulator_node_name: The name of the accumulator node which should contain
                                          the Matmul result before fusion according to the IR graph.
            epilogue_nodes: The list of scheduler nodes to be fused into the epilogue
        """
        self.accumulator_node_name: str = accumulator_node_name  #
        self.body: IndentedBuffer = IndentedBuffer(1)  # The body buffer for codegen
        self.var_counter: Iterator[int] = itertools.count()
        self.store_name_to_value: dict[str, OpsValue] = (
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `Args`, `accumulator_node_name`, and `epilogue_nodes`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `Args`、`accumulator_node_name`、`epilogue_nodes` 等值。

### Lines 161-180 / 第 161-180 行
````python
            dict()
        )  # Aliases for subexpression functors
        self.reads: OrderedSet[str] = OrderedSet([])
        # Used for creating example tensors
        self.var_name_to_buffer_name: dict[str, str] = {
            _ACCUMULATOR_ARG_NAME: accumulator_node_name
        }
        self.removed_buffers: OrderedSet[str] = removed_buffers
        self.cur_node: ComputedBuffer | None = None
        self.name_to_buffer = V.graph.name_to_buffer | V.graph.graph_inputs
        for name in V.graph.constants:
            # pyrefly: ignore [unsupported-operation]
            self.name_to_buffer[name] = V.graph.add_tensor_constant(
                V.graph.constants[name], name
            )
        self.is_D_assigned = False
        self.D_var_name = None

        if accumulator_node_name not in removed_buffers:
            # cannot return accumulator directly, so alias it
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_ACCUMULATOR_ARG_NAME`. This range continues the implementation of function `CutlassEVTCodegen.__init__`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_ACCUMULATOR_ARG_NAME` 等值。这一段延续了函数`CutlassEVTCodegen.__init__` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
            var = self._tmp_var()
            self.body.writeline(f"{var} = {_ACCUMULATOR_ARG_NAME}")
            self.store(accumulator_node_name, value=OpsValue(var))

    @staticmethod
    def ir_to_evt_python_code(
        cutlass_template_node_name: str,
        epilogue_nodes: list[BaseSchedulerNode],
        removed_buffers: OrderedSet[str],
    ) -> tuple[list[str], list[str], dict[str, Any], str]:
        codegen = CutlassEVTCodegen(cutlass_template_node_name, removed_buffers)
        handler = _AssignmentFormatter(codegen)

        with virtualized.V.set_ops_handler(handler):
            for s_node in epilogue_nodes:
                node = s_node.node
                assert isinstance(node, ComputedBuffer)
                with codegen.set_cur_node(node):
                    index_vars = CutlassEVTCodegen.get_index_vars(node)
                    node.get_store_function()(index_vars)
````
- **EN**: Introduces function `ir_to_evt_python_code`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`ir_to_evt_python_code`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 201-220 / 第 201-220 行
````python

        codegen.finalize()

        return (
            codegen.get_reads(),
            codegen.get_writes(),
            codegen.get_renames(),
            codegen.get_value(),
        )

    def get_value(self) -> str:
        return linesep.join(
            [
                self._render_input_signature(),
                self.body.getvalue(),
                self._render_return_statement(),
            ]
        )

    def finalize(self) -> None:
````
- **EN**: Introduces function `get_value`, function `finalize`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_value`、函数`finalize`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python
        # Rename the last store to D
        # no other code references this store
        # to workaround https://github.com/NVIDIA/cutlass/issues/2288
        # Note: the delayed line will automatically rewrite the last assignment to
        # be to D
        buffer_name = self.var_name_to_buffer_name[self.last_stored_var_name]
        self.var_name_to_buffer_name.pop(self.last_stored_var_name)
        self.var_name_to_buffer_name["D"] = buffer_name
        self.store_name_to_value[buffer_name] = OpsValue("D")

    @contextmanager
    def set_cur_node(self, node: ComputedBuffer) -> Generator[None, Any, Any]:
        prev_node = self.cur_node
        try:
            self.cur_node = node
            yield
        finally:
            self.cur_node = prev_node

    def get_renames(self) -> dict[str, str]:
````
- **EN**: Introduces function `set_cur_node`, function `get_renames`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`set_cur_node`、函数`get_renames`。保留了指向设计说明、规范或厂商数据手册的注释引用。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 241-260 / 第 241-260 行
````python
        return dict(self.var_name_to_buffer_name)

    def get_reads(self) -> list[str]:
        return list(self.reads.difference(self.store_name_to_value.keys()))

    def get_writes(self) -> list[str]:
        return list(self.store_name_to_value.keys())

    def load(self, name: str, index: Any) -> str:
        self._check_indexing(name, index)
        if name in self.store_name_to_value:
            return self.store_name_to_value[name].value
        elif name == self.accumulator_node_name:
            return _ACCUMULATOR_ARG_NAME
        else:
            self.reads.add(name)
            self.var_name_to_buffer_name[name] = name
            return name

    def store(
````
- **EN**: Introduces function `get_reads`, function `get_writes`, function `load`, function `store`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`get_reads`、函数`get_writes`、函数`load`、函数`store`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 261-280 / 第 261-280 行
````python
        self, name: Any, index: Any = None, value: Any = None, mode: Any = None
    ) -> None:
        if name not in self.removed_buffers:
            if index:
                self._check_indexing(name, index)
            assert value.value != _ACCUMULATOR_ARG_NAME, (
                "Cannot store accumulator arg name"
            )
            self.var_name_to_buffer_name[value.value] = name
            self.store_name_to_value[name] = value
            self.last_stored_var_name = value.value
        return None

    def _get_cur_node(self) -> ComputedBuffer:
        assert self.cur_node
        return self.cur_node

    @staticmethod
    def get_index_vars(node: ComputedBuffer) -> Sequence[sympy.Expr]:
        data = node.data
````
- **EN**: Introduces function `_get_cur_node`, function `get_index_vars`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_cur_node`、函数`get_index_vars`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-300 / 第 281-300 行
````python
        # TODO mlazos: relax this, cutlass supports reductions and other ops
        assert isinstance(data, Pointwise)
        return data._index(data.ranges)

    def _get_current_index_vars(self) -> Sequence[sympy.Expr]:
        return self.get_index_vars(self._get_cur_node())

    def _check_indexing(self, name: str, index: sympy.Expr) -> None:
        # We only support indexing that matches the layout today because
        # CUTLASS doesn't support arbitrary indexing
        buffer_name = (
            self.accumulator_node_name if name == _ACCUMULATOR_ARG_NAME else name
        )
        buffer = self.name_to_buffer[buffer_name]
        index_strides = V.graph.sizevars.stride_vars(
            index, self._get_current_index_vars()
        )
        stride = buffer.get_layout().stride
        if not self._stride_compatible(stride, index_strides):
            raise NotImplementedError(
````
- **EN**: Introduces function `_get_current_index_vars`, function `_check_indexing`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_current_index_vars`、函数`_check_indexing`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 301-320 / 第 301-320 行
````python
                f"Unsupported indexing for {name} with index {index}, index strides {index_strides}, and layout stride {stride}"
            )

    def _stride_compatible(
        self, left: Iterable[sympy.Expr], right: Iterable[sympy.Expr]
    ) -> bool:
        return all(
            sympy.Eq(l, r) or sympy.Eq(l, 0) or sympy.Eq(r, 0)
            for l, r in (zip(left, right))
        )

    def _render_input_signature(self) -> str:
        arguments = ", ".join(
            [_ACCUMULATOR_ARG_NAME]
            + [name for name in self.reads if name != self.accumulator_node_name]
        )
        return f"def fn({arguments}):"

    def _render_return_statement(self) -> str:
        return_vars = OrderedSet(
````
- **EN**: Introduces function `_stride_compatible`, function `_render_input_signature`, function `_render_return_statement`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_stride_compatible`、函数`_render_input_signature`、函数`_render_return_statement`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 321-327 / 第 321-327 行
````python
            op_v.value for op_v in self.store_name_to_value.values()
        )
        assert "D" in return_vars
        return f"return {', '.join(return_vars)}"

    def _tmp_var(self) -> str:
        return f"tmp_{next(self.var_counter)}"
````
- **EN**: Introduces function `_tmp_var`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_tmp_var`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `CutlassEVTOpsMixIn`, `MockCutlassHandler`, `_AssignmentFormatter`, and `CutlassEVTCodegen`  
  **CN**: 主要类：`CutlassEVTOpsMixIn`、`MockCutlassHandler`、`_AssignmentFormatter`、`CutlassEVTCodegen`
- **EN**: Primary functions: `scaled_mm_evt`  
  **CN**: 主要函数：`scaled_mm_evt`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `collections.abc`, `contextlib`, `os`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.virtualized`, `torch._inductor.ir`, `torch._inductor.ops_handler`, `torch._inductor.scheduler`, `torch._inductor.utils`, `...virtualized`
