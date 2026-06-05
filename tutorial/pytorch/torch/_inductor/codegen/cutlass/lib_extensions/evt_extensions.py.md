# evt_extensions.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cutlass/lib_extensions/evt_extensions.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from collections.abc import Callable
from typing import Any

from sympy import Expr

from torch._inductor.ir import (
    ComputedBuffer,
    InputBuffer,
    is_contiguous_strides_for_shape,
)
from torch.utils._ordered_set import OrderedSet

from ..utils import cutlass_arch, torch_dtype_to_cutlass_type, try_import_cutlass


EpilogueFunctor = Any  # EpilogueFunctor local class defined in _trace
Buffer = ComputedBuffer | InputBuffer
CutlassTupleType = Any  # cutlass.backend.c_types.tuple_factory_.<locals>.TupleType
CutlassVisitorType = Any  # cutlass.backend.c_types.visitor_factory.<locals>.VisitorType
CutlassArgType = (
````
- **EN**: Imports dependencies such as `collections.abc`, `typing`, `sympy`, `torch._inductor.ir`, `torch.utils._ordered_set`, and `..utils` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `EpilogueFunctor`, `Buffer`, `CutlassTupleType`, `CutlassVisitorType`, and `CutlassArgType`.
- **CN**: 这里导入了 `collections.abc`、`typing`、`sympy`、`torch._inductor.ir`、`torch.utils._ordered_set`、`..utils` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `EpilogueFunctor`、`Buffer`、`CutlassTupleType`、`CutlassVisitorType`、`CutlassArgType` 等值。

### Lines 21-40 / 第 21-40 行
````python
    Any  # Can be a CutlassTupleType, CutlassVisitorType, EmptyByte, or ctype.c_void_p
)


if try_import_cutlass():
    import ast
    import ctypes
    import textwrap

    from cutlass_cppgen.backend.c_types import (  # type: ignore[import-not-found]
        EmptyByte,
    )
    from cutlass_cppgen.backend.epilogue import (  # type: ignore[import-not-found]
        dtype2ctype,
    )
    from cutlass_cppgen.backend.evt import (  # type: ignore[import-not-found]
        backend as evt_backend,
        EpilogueFunctorVisitor,
    )
    from cutlass_cppgen.backend.evt.backend.emitter_base import (  # type: ignore[import-not-found]
````
- **EN**: Imports dependencies such as `ast`, `ctypes`, `textwrap`, `cutlass_cppgen.backend.c_types`, `cutlass_cppgen.backend.epilogue`, `cutlass_cppgen.backend.evt`, and `...+1` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `ast`、`ctypes`、`textwrap`、`cutlass_cppgen.backend.c_types`、`cutlass_cppgen.backend.epilogue`、`cutlass_cppgen.backend.evt`、`另有1项` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
        FusionCallbacks,
    )
    from cutlass_cppgen.backend.evt.backend.sm100_emitter import (  # type: ignore[import-not-found]
        Sm100CollectiveEpilogue,
    )
    from cutlass_cppgen.backend.evt.frontend import (  # type: ignore[import-not-found]
        PythonASTFrontend,
    )
    from cutlass_cppgen.backend.evt.ir.tensor import (  # type: ignore[import-not-found]
        Tensor as CutlassTensor,
    )
    from cutlass_library import (
        DataType,
        EpilogueScheduleType,
        LayoutType,
        TileDescription,
    )

    from torch._inductor.utils import IndentedBuffer

````
- **EN**: Imports dependencies such as `cutlass_cppgen.backend.evt.backend.sm100_emitter`, `cutlass_cppgen.backend.evt.frontend`, `cutlass_cppgen.backend.evt.ir.tensor`, `cutlass_library`, and `torch._inductor.utils` for the logic in this range.
- **CN**: 这里导入了 `cutlass_cppgen.backend.evt.backend.sm100_emitter`、`cutlass_cppgen.backend.evt.frontend`、`cutlass_cppgen.backend.evt.ir.tensor`、`cutlass_library`、`torch._inductor.utils` 等依赖，为后续逻辑提供基础能力。

### Lines 61-80 / 第 61-80 行
````python
    _CUTLASS_C_DTYPES = OrderedSet(dtype2ctype.values())  # type: ignore[var-annotated]

    class EVTArgRenames:
        """Handles mapping buffer names to variable names in the cpp kernel signature and body"""

        def __init__(self) -> None:
            self.buf_renames: dict[str, str] = {}

        def new_name(self, name: str) -> str:
            if name in self.buf_renames:
                return self.buf_renames[name]
            else:
                new_name = f"ptr_{len(self.buf_renames)}"
                self.buf_renames[name] = new_name
                return new_name

        def get(self, name: str) -> str:
            return self.buf_renames.get(name, name)

    def create_example_tensors(
````
- **EN**: Introduces class `EVTArgRenames`, function `__init__`, function `new_name`, function `get`, function `create_example_tensors`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_CUTLASS_C_DTYPES`, `else`, and `new_name`.
- **CN**: 这里定义了类`EVTArgRenames`、函数`__init__`、函数`new_name`、函数`get`、函数`create_example_tensors`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_CUTLASS_C_DTYPES`、`else`、`new_name` 等值。

### Lines 81-100 / 第 81-100 行
````python
        var_name_to_buffer_name: dict[str, str],
        name_to_buffer: dict[str, Buffer],
        size_hint_fn: Callable[[Expr | int], int],
    ) -> dict[str, CutlassTensor]:
        def cutlass_tensor_from_buffer(
            buffer: Buffer,
        ) -> CutlassTensor:
            shape = buffer.get_layout().size
            stride = buffer.get_layout().stride
            shape = tuple(size_hint_fn(x) for x in shape)
            stride = tuple(size_hint_fn(x) for x in stride)

            is_row_major = is_contiguous_strides_for_shape(stride, shape)
            is_column_major = is_contiguous_strides_for_shape(stride[::-1], shape[::-1])

            if not is_row_major and not is_column_major:
                raise RuntimeError(
                    f"Cannot create example tensor for {buffer.get_name()} with \
non-contiguous layout, received stride: {stride} and shape: {shape}"
                )
````
- **EN**: Introduces function `cutlass_tensor_from_buffer`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`cutlass_tensor_from_buffer`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python

            return CutlassTensor(
                shape=shape,
                layout_tag=(
                    LayoutType.RowMajor if is_row_major else LayoutType.ColumnMajor
                ),
                element=torch_dtype_to_cutlass_type(buffer.get_layout().dtype),
            )

        return {
            key: cutlass_tensor_from_buffer(name_to_buffer[name])
            for key, name in var_name_to_buffer_name.items()
        }

    def trace(
        fn_src: str,
        example_tensors: dict[str, CutlassTensor],
        accum_type: DataType,
        output_type: DataType,
        tile_description: TileDescription,
````
- **EN**: Introduces function `trace`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`trace`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
        epilogue_schedule: EpilogueScheduleType,
        name_to_buffer: dict[str, Buffer],
        size_hint_fn: Callable[[Expr | int], int],
        kernel_schedule: Any | None = None,
        device_type: str = "cuda",
        **kwargs: dict[str, Any],
    ) -> tuple[str, str, str, EVTArgRenames]:
        arch = int(cutlass_arch(device_type))
        assert device_type != "cuda" or arch >= 90, (
            "For CUDA, only SM90+ is supported for EVT"
        )
        epilogue_functor = _trace(fn_src, example_tensors, arch, **kwargs)
        visitor = EpilogueFunctorVisitor(arch, epilogue_functor)
        fusion_callbacks = FusionCallbacks(visitor.graph, arch, emit_CD=False)
        arch_prefix = "xe" if device_type == "xpu" else "sm"

        if device_type == "xpu" or arch < 100:
            try:
                evt_emitter = getattr(evt_backend, f"{arch_prefix}{arch}_emitter")
                CollectiveEpilogue = evt_emitter.CollectiveEpilogue
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
            except AttributeError as e:
                raise NotImplementedError(
                    f"EVT backend is not supported on Arch {arch_prefix}{arch}."
                ) from e

            collective_epilogue = CollectiveEpilogue(
                tile_description,
                epilogue_schedule,
                accum_type,
                output_type,
                fusion_callbacks,
            )
        else:
            collective_epilogue = Sm100CollectiveEpilogue(
                tile_description=tile_description,
                kernel_schedule=kernel_schedule,
                epilogue_schedule=epilogue_schedule,
                element_accumulator=accum_type,
                element_d=output_type,
                fusion_callbacks=fusion_callbacks,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `collective_epilogue`, `else`, `tile_description`, `kernel_schedule`, `epilogue_schedule`, `element_accumulator`, and `...+2`. This range continues the implementation of function `trace`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `collective_epilogue`、`else`、`tile_description`、`kernel_schedule`、`epilogue_schedule`、`element_accumulator`、`另有2项` 等值。这一段延续了函数`trace` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
            )
        evt_name, evt_code = collective_epilogue.emit()
        evt_args, arg_renames = _render_argument_type(
            epilogue_functor, name_to_buffer, size_hint_fn
        )
        return evt_name, evt_args, evt_code, arg_renames

    # Based off of
    # https://github.com/NVIDIA/cutlass/blob/df18f5e4f5de76bed8be1de8e4c245f2f5ec3020/python/cutlass/epilogue/epilogue.py#L117
    # This is modified to enable directly passing the source code of the epilogue vs getting it from a bona-fide python function
    # The reason for this is that inspect.getsource does not work with functions defined at runtime via exec/eval
    def _trace(
        fn_src: str,
        example_tensors: dict[str, CutlassTensor],
        cc: int,
        **kwargs: Any,
    ) -> EpilogueFunctor:
        class EpilogueFunctor(PythonASTFrontend):
            def __init__(self, cc: int, **kwargs: Any):
                self.source = textwrap.dedent(fn_src)
````
- **EN**: Introduces function `_trace`, class `EpilogueFunctor`, function `__init__`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `fn_src`, `example_tensors`, and `cc`.
- **CN**: 这里定义了函数`_trace`、类`EpilogueFunctor`、函数`__init__`。保留了指向设计说明、规范或厂商数据手册的注释引用。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `fn_src`、`example_tensors`、`cc` 等值。

### Lines 181-200 / 第 181-200 行
````python
                super().__init__(cc, **kwargs)

            def parse(
                self,
                example_inputs: dict[str, CutlassTensor],
            ) -> None:
                self.example_inputs = example_inputs
                self.ast = ast.parse(self.source)
                # pyrefly: ignore [missing-attribute]
                self.visit(self.ast)

        epilogue_functor = EpilogueFunctor(cc=cc, **kwargs)
        epilogue_functor.trace(example_tensors)
        return epilogue_functor

    def _render_argument_type(
        epilogue_functor: EpilogueFunctor,
        name_to_buffer: dict[str, Buffer],
        size_hint_fn: Callable[[Expr | int], int],
    ) -> tuple[str, EVTArgRenames]:
````
- **EN**: Introduces function `parse`, function `_render_argument_type`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `example_inputs`, `epilogue_functor`, `name_to_buffer`, and `size_hint_fn`.
- **CN**: 这里定义了函数`parse`、函数`_render_argument_type`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `example_inputs`、`epilogue_functor`、`name_to_buffer`、`size_hint_fn` 等值。

### Lines 201-220 / 第 201-220 行
````python
        epilogue_thread_type = epilogue_functor.epilogue_thread_type
        arg_renames = EVTArgRenames()

        # Fragile, but this is the only way to guarantee t is expected type because t is a local class
        def is_nested_visitor_type(t: type) -> bool:
            return (
                ".".join([t.__module__, t.__qualname__])
                == "cutlass_cppgen.backend.c_types.visitor_factory.<locals>.VisitorType"
            )

        buffer = IndentedBuffer()
        with buffer.set_tabwidth(2):

            def render_argument_type(name: str, t: CutlassArgType) -> None:
                if issubclass(t, ctypes.c_byte):
                    buffer.writeline(f"{{}}, /* {name} */")
                else:
                    fields = [
                        (
                            fname,
````
- **EN**: Introduces function `is_nested_visitor_type`, function `render_argument_type`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `epilogue_thread_type`, `arg_renames`, `buffer`, `else`, and `fields`.
- **CN**: 这里定义了函数`is_nested_visitor_type`、函数`render_argument_type`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `epilogue_thread_type`、`arg_renames`、`buffer`、`else`、`fields` 等值。

### Lines 221-240 / 第 221-240 行
````python
                            _get_arg_from_node(
                                ty, name_to_buffer[name], size_hint_fn, arg_renames
                            ),
                        )
                        for fname, ty in t._fields_
                    ]
                    field_strs = [
                        f"/* {fname} */ {str(field)}" for fname, field in fields
                    ]
                    buffer.writeline(f"{{{', '.join(field_strs)}}}, /* {name} */")

            def render_thread_type(name: str, t: CutlassArgType) -> None:
                if is_nested_visitor_type(t):
                    buffer.writeline(f"{{ /* {name} */")
                    with buffer.indent():
                        for name, inner_t in t._fields_:
                            render_thread_type(name, inner_t)
                    buffer.writeline("},")
                else:
                    render_argument_type(name, t)
````
- **EN**: Introduces function `render_thread_type`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `field_strs`, and `else`.
- **CN**: 这里定义了函数`render_thread_type`。包含分支、循环或上下文管理等控制流。初始化或更新了 `field_strs`、`else` 等值。

### Lines 241-260 / 第 241-260 行
````python

            # unroll the recursion once to address special case formatting
            # namely, no ending comma and no indentation for the outermost thread type
            buffer.writeline("{ /* thread */")
            with buffer.indent(3):
                if is_nested_visitor_type(epilogue_thread_type):
                    with buffer.indent():
                        for name, inner_t in epilogue_thread_type._fields_:
                            render_thread_type(name, inner_t)
                else:
                    render_argument_type("thread", epilogue_thread_type)
                buffer.writeline("}")

        return buffer.getvalue(), arg_renames

    def _get_arg_from_node(
        arg_ty: type,
        node: Buffer,
        size_hint_fn: Callable[[Expr | int], int],
        arg_renames: EVTArgRenames,
````
- **EN**: Introduces function `_get_arg_from_node`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_arg_from_node`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 261-280 / 第 261-280 行
````python
    ) -> str:
        from ..template import CUTLASSTemplate

        # Today, arguments are either a pointer to the
        # node's memory, a stride tuple, the datatype
        # Once again, need to check for local class type for stride tuple
        if (
            str(arg_ty)
            == "<class 'cutlass_cppgen.backend.c_types.tuple_factory_.<locals>.TupleType'>"
        ):
            DEFAULT_STRIDE_LEN = 3
            assert len(node.get_layout().stride) <= DEFAULT_STRIDE_LEN
            stride = [size_hint_fn(x) for x in node.get_layout().stride]
            for _ in range(DEFAULT_STRIDE_LEN - len(stride)):
                stride.append(0)

            def render_stride(x: int) -> str:
                # Handle EBO for 0 and 1
                if x == 0:
                    return "_0{}"
````
- **EN**: Imports dependencies such as `..template` for the logic in this range. Introduces function `render_stride`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..template` 等依赖，为后续逻辑提供基础能力。这里定义了函数`render_stride`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 281-298 / 第 281-298 行
````python
                elif x == 1:
                    return "_1{}"
                else:
                    return str(x)

            return f"{{{', '.join([render_stride(x) for x in stride])}}}"

        elif issubclass(arg_ty, ctypes.c_void_p):
            name = arg_renames.new_name(node.get_name())
            return f"({CUTLASSTemplate._DTYPE_TO_CUTLASS[node.get_layout().dtype]}*) ({name} + {name}_offset)"
        elif (
            arg_ty in _CUTLASS_C_DTYPES
        ):  # Assumption: this is the element dtype, this holds for all cutlass ir nodes currently
            return f"{CUTLASSTemplate._DTYPE_TO_CUTLASS[node.get_layout().dtype]}(0)"
        elif issubclass(arg_ty, EmptyByte):
            return "{}"

        raise NotImplementedError(f"Unsupported arg type: {arg_ty}")
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `name`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`name` 等值。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `typing`, `ast`, `ctypes`, `textwrap`
- **Third-party / 第三方**: `sympy`, `cutlass_cppgen.backend.c_types`, `cutlass_cppgen.backend.epilogue`, `cutlass_cppgen.backend.evt`, `cutlass_cppgen.backend.evt.backend.emitter_base`, `cutlass_cppgen.backend.evt.backend.sm100_emitter`, `cutlass_cppgen.backend.evt.frontend`, `cutlass_cppgen.backend.evt.ir.tensor`, `cutlass_library`
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.ir`, `torch.utils._ordered_set`, `..utils`, `torch._inductor.utils`, `..template`
