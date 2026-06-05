# native_function_generation.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/native_function_generation.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements part of the torchgen pipeline that converts operator metadata into generated source code or helper data.
- **Purpose (CN)**: 实现 torchgen 流水线中的一部分，把算子元数据转换为生成源码或辅助数据。
## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```python
from __future__ import annotations

import string
from collections import defaultdict
from typing import TYPE_CHECKING

import torchgen.api.dispatcher as dispatcher
from torchgen.api.translate import translate
from torchgen.api.types import Binding, DispatcherSignature, Expr
from torchgen.context import with_native_function
from torchgen.model import (
    Annotation,
    Argument,
    BackendIndex,
    BackendMetadata,
    BaseOperatorName,
    BaseTy,
    BaseType,
    DEFAULT_KERNEL_NAMESPACE,
    DeviceCheckType,
    DispatchKey,
    FunctionSchema,
    NativeFunction,
    NativeFunctionsGroup,
    OperatorName,
    Return,
    SchemaKind,
    Variant,
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.api.dispatcher, torchgen.api.translate, torchgen.api.types, torchgen.context, and 1 more; standard-library modules such as __future__, string, collections, typing for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.api.dispatcher、torchgen.api.translate、torchgen.api.types、torchgen.context 等共 5 项；标准库模块，如 __future__、string、collections、typing组织在一起，供下方逻辑使用。

### Lines 29-54
```python
)
from torchgen.utils import concatMap


if TYPE_CHECKING:
    from collections.abc import Sequence


# See Note: [Out ops with functional variants that don't get grouped properly]
OUT_OPS_THAT_DONT_GET_GROUPED_PROPERLY = [
    # This has a functional variant, but it's currently marked private.
    # This function should be marked private as well (*_backward ops aren't exposed to python anyway).
    "adaptive_avg_pool3d_backward.grad_input",
    # There's a functional variant, _slow_conv2d_backward.output_mask, that isn't grouped properly.
    # Maybe we can kill this operator in favor of convolution_backward?
    "_slow_conv2d_backward.grad_input",
]


# See Note: [Mutable ops that cannot get an out variant]
MUTABLE_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT = [
    # should be out=?
    "_cummax_helper",
    # should be out=?
    "_cummin_helper",
]
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.utils; standard-library modules such as collections.abc for the logic below. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.utils；标准库模块，如 collections.abc组织在一起，供下方逻辑使用。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 56-83
```python
# All of these operators don't have any tensor like returns
FUNCTIONAL_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT = [
    "_assert_async",  # no return
    "_assert_async.msg",  # no return
    "_assert_tensor_metadata",  # no return
    "_cslt_sparse_mm_search",  # returns an int
    "_assert_scalar",  # no return
    "_dimI",  # returns an int
    "_dimV",  # returns an int
    "_has_same_storage_numel",  # returns a boolean
    "_linalg_check_errors",  # no return
    "_local_scalar_dense",  # returns a Scalar
    "_nested_tensor_from_mask_left_aligned",  # returns a boolean
    "_nnz",  # returns an int
    "_use_cudnn_ctc_loss",  # returns a boolean
    "_use_cudnn_ctc_loss.Tensor",  # returns a boolean
    "_use_miopen_ctc_loss",  # returns a boolean
    "_use_miopen_ctc_loss.Tensor",  # returns a boolean
    "_validate_compressed_sparse_indices",  # no return
    "allclose",  # returns a boolean
    "dense_dim",  # returns an int
    "equal",  # returns a boolean
    "is_coalesced",  # returns an boolean
    "is_pinned",  # returns a boolean
    "is_same_size",  # returns a boolean
    "is_set_to",  # returns a boolean
    "q_per_channel_axis",  # returns an int
    "q_scale",  # returns a float
```
- **EN**: Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 84-105
```python
    "q_zero_point",  # returns an int
    "qscheme",  # returns a QScheme
    "record_stream",  # no return
    "sparse_dim",  # returns an int
    "sym_constrain_range",  # no return
    "sym_constrain_range_for_size",  # no return
    "_nested_tensor_storage_offsets",  # returns a vector of ints
    "_chunk_grad_outputs_efficient_attention",  # returns a bool
    "_fused_sdp_choice",  # returns an int
    "_print",  # no return
    "_sink_tokens",  # no return
    "_nested_get_ragged_idx",  # returns an int
]

INPLACE_OPS_THAT_DONT_GET_GROUPED_PROPERLY = [
    # polygamma and polygamma.out both exist, but have a
    # pre-self arg (while polygamma_ does not)
    # We should either fix this schema so it can be grouped properly,
    # or allow the codegen to generate new functional/out= NativeFunctions for this op
    # (which would require changing its overload name to prevent overload ambiguity).
    "polygamma_"
]
```
- **EN**: Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 108-128
```python
# Groups "similar" NativeFunctions together
# example add.Tensor, add_.Tensor, add.out
# "similar" NativeFunctions are all expected to have an identical `signature()`,
# But have differing SchemaKinds.
def pre_group_native_functions(
    native_functions: Sequence[NativeFunction],
) -> dict[FunctionSchema, dict[SchemaKind, NativeFunction]]:
    pre_grouped_native_functions: dict[
        FunctionSchema, dict[SchemaKind, NativeFunction]
    ] = defaultdict(dict)
    for f in native_functions:
        d = pre_grouped_native_functions[f.func.signature()]
        if f.func.kind() in d:
            raise AssertionError(f"Duplicate schema kind {f.func.kind()} for {f.func}")
        d[f.func.kind()] = f
    return pre_grouped_native_functions


# Returns the out variant overload name given a base function overload name
def get_expected_out_variant_overload_name(overload_name: str | None) -> str:
    return "out" if not overload_name else f"{overload_name}_out"
```
- **EN**: This chunk defines `get_expected_out_variant_overload_name`, which loads external data or stored state into the active pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_expected_out_variant_overload_name`，其作用是把外部数据或已存储状态加载到当前流水线中。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 131-158
```python
# Helper function: given an inplace FunctionSchema, generate its corresponding out= variant
# Example before:
#   _add_relu_.Scalar(Tensor(a!) self, Scalar other, Scalar alpha=1) -> Tensor(a!)
# Example after:
#   _add_relu.Scalar_out(Tensor self, Scalar other, Scalar alpha=1, *, Tensor(a!) out)
def self_to_out_signature(func: FunctionSchema) -> FunctionSchema:
    # Generating an out= schema from an inplace schema.
    if func.kind() != SchemaKind.inplace:
        raise AssertionError(f"Expected inplace schema, got {func.kind()}")
    if func.arguments.self_arg is None:
        raise AssertionError("Expected self_arg to be non-None")
    # The new out= schema has:
    # - a new out argument with the same type as "func" (but with a mutable annotation)
    # - The returns (if any) now alias the out= argument instead of "func"
    # - an "out" overload name
    return FunctionSchema(
        name=func.name.remove_inplace().with_overload(
            get_expected_out_variant_overload_name(func.name.overload_name)
        ),
        arguments=func.arguments.remove_self_annotation().with_out_args(
            [
                Argument(
                    name="out",
                    type=func.arguments.self_arg.argument.type,
                    default=None,
                    annotation=func.arguments.self_arg.argument.annotation,
                )
            ]
```
- **EN**: This chunk defines `self_to_out_signature`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `self_to_out_signature`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 159-186
```python
        ),
        returns=func.returns,
    )


# Helper function: given a functional FunctionSchema, generate its corresponding out= variant
# Example before:
#   _to_copy(Tensor self, *, ScalarType? dtype=None, Layout? layout=None, Device? device=None,
#       bool? pin_memory=None, bool non_blocking=False, MemoryFormat? memory_format=None) -> Tensor
# Example after:
#   _to_copy._out(Tensor self, *, bool non_blocking=False, MemoryFormat? memory_format=None,
#       Tensor(a!) out) -> Tensor(a!)
def functional_to_out_signature(func: FunctionSchema) -> FunctionSchema:
    # Generating an out= schema from a functional schema.
    if func.kind() != SchemaKind.functional:
        raise AssertionError(f"Expected functional schema, got {func.kind()}")

    new_returns, new_out_args = generate_out_args_from_schema(func)
    # The new out= schema has:
    # - one or more new out argument(s) with the same type as returns (but with a mutable annotation)
    # - The returns now alias the out= arguments
    # - an "_out" overload name
    return FunctionSchema(
        name=func.name.with_overload(
            get_expected_out_variant_overload_name(func.name.overload_name)
        ),
        arguments=func.arguments.signature().with_out_args(
            new_out_args,
```
- **EN**: This chunk defines `functional_to_out_signature`, which wraps lower-level behavior in a transform-friendly interface. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `functional_to_out_signature`，其作用是把底层行为包装为更适合变换组合的接口。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 187-211
```python
        ),
        returns=tuple(new_returns),
    )


# Helper function: given a function schema, generate corresponding out arguments, also the updated return annotations.
def generate_out_args_from_schema(
    func: FunctionSchema,
) -> tuple[list[Return], list[Argument]]:
    # More of a sanity check - our existing restrictions on schemas should enforce that
    # mutable schema kinds never return their mutable arguments.
    if any(r.annotation is not None and r.annotation.is_write for r in func.returns):
        raise AssertionError("Mutable schema kinds should not return mutable arguments")

    tensorlike_rets = [r for r in func.returns if r.type.is_tensor_like()]
    if len(tensorlike_rets) == 0:
        raise AssertionError("Expected at least one tensor-like return")

    used_annotations = concatMap(
        lambda a: [] if a.annotation is None else a.annotation.alias_set,
        func.arguments.flat_all,
    )
    valid_annotations = [x for x in string.ascii_lowercase if x not in used_annotations]

    all_rets_are_tensors = all(r.type == BaseType(BaseTy.Tensor) for r in func.returns)
```
- **EN**: This chunk defines `generate_out_args_from_schema`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `generate_out_args_from_schema`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 213-236
```python
    new_out_args: list[Argument] = []
    # The end result of new_returns is that:
    # - If every return is a plain tensor, then the new returns == the old returns, but with the out= alias annotations added.
    # - Otherwise, none of the out arguments show up in the returns (and we're only left with non-tensor-like returns, if any).
    new_returns: list[Return] = []
    for i, r in enumerate(func.returns):
        if r.type.is_tensor_like():
            new_out = Argument(
                name="out" if len(func.returns) == 1 else f"out{i}",
                type=r.type,
                default=None,
                annotation=Annotation.parse(f"{valid_annotations[i]}!"),
            )
            new_out_args.append(new_out)
            if all_rets_are_tensors:
                # The convention for out= schemas is that they only return their out arguments
                # if the return is a plain Tensor (or if it's a tuple of plain Tensors)
                new_ret = Return(
                    name=None, type=new_out.type, annotation=new_out.annotation
                )
                new_returns.append(new_ret)
        else:
            new_returns.append(r)
    return new_returns, new_out_args
```
- **EN**: This chunk continues `generate_out_args_from_schema` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `generate_out_args_from_schema`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 239-261
```python
# Helper function: given a mutable FunctionSchema, generate its corresponding out= variant
# Example before:
#   _fused_moving_avg_obs_fq_helper(Tensor self, Tensor observer_on, Tensor fake_quant_on, Tensor(a!) running_min, Tensor(b!) running_max, Tensor(c!) scale, Tensor(d!) zero_point, float averaging_const, int quant_min, int quant_max, int ch_axis, bool per_row_fake_quant=False, bool symmetric_quant=False) -> (Tensor output, Tensor mask)
# Example after:
#   _fused_moving_avg_obs_fq_helper._out(Tensor self, Tensor observer_on, Tensor fake_quant_on, Tensor(a!) running_min, Tensor(b!) running_max, Tensor(c!) scale, Tensor(d!) zero_point, float averaging_const, int quant_min, int quant_max, int ch_axis, bool per_row_fake_quant=False, bool symmetric_quant=False, *, Tensor(e!) out0, Tensor(f!) out1) -> (Tensor(e!), Tensor(f!))
def mutable_to_out_signature(func: FunctionSchema) -> FunctionSchema:
    # Generating an out= schema from a mutable schema.
    if func.kind() != SchemaKind.mutable:
        raise AssertionError(f"Expected mutable schema, got {func.kind()}")
    # The new out= schema has:
    # - Any non-aliased tensor-like returns are converted to mutable, aliased out= arguments
    #   (if the argument is a tensor then we also return it for method chaining,
    #   otherwise we return nothing)
    # - an "out" overload name
    #
    # Note that:
    # (1) This also means that we can *only* generate an out= variant from a mutable schema
    #     if the mutable schema has at least one tensor-like non-aliasing return.
    # (2) The generated out= variant still has mutable positional arguments,
    #     but if necessary we could probably add another out= variant that also
    #     functionalizes the mutable arguments (a functional_out variant)

    new_returns, new_out_args = generate_out_args_from_schema(func)
```
- **EN**: This chunk defines `mutable_to_out_signature`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `mutable_to_out_signature`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 263-283
```python
    return FunctionSchema(
        name=func.name.remove_inplace().with_overload(
            get_expected_out_variant_overload_name(func.name.overload_name)
        ),
        arguments=func.arguments.with_out_args(new_out_args),
        returns=tuple(new_returns),
    )


# This function, given function of one SchemaKind, as well as a target SchemaKind,
# generates a new NativeFunction with the same properties, but using the target SchemaKind.
# We only actually generate functions for either functional or out= SchemaKinds.
# This function returns a tuple, with:
# - The generated NativeFunction
# - a dictionary of `BackendIndex` objects, describing which dispatch keys
#   we will generate kernels for, for the new NativeFunction.
#   Details are in the function, but we only generate composite kernels (in some cases) today.
def generate_function(
    f: NativeFunction, k: SchemaKind
) -> tuple[NativeFunction, dict[DispatchKey, dict[OperatorName, BackendMetadata]]]:
    from torchgen.api import cpp
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.api for the logic below. This chunk defines `generate_function`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.api组织在一起，供下方逻辑使用。 这一段定义了 `generate_function`，其作用是根据高层元数据生成派生代码、表或分析产物。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 285-312
```python
    if k == SchemaKind.functional:
        if f.func.kind() == SchemaKind.functional:
            raise AssertionError("Cannot generate functional from functional schema")
        # The new "functional" NativeFunction has:
        # - any mutable arguments have been converted into (immutable) returns.
        #   (if a mutable argument was not also a return, it gets converted to one)
        # - "_functional" appended to the base name, ONLY IF this op has a mutable variant.
        #   See Note [Overload Ambiguity With Functional Variants]
        # The default grouping logic in signature() actually already does this,
        # so we can piggy-back off it (but we still want return names)
        func = f.func.signature(keep_return_names=True).with_name(
            OperatorName(
                name=BaseOperatorName(
                    base=f.func.name.name.base,
                    inplace=False,
                    dunder_method=f.func.name.name.dunder_method,
                    # See Note [Overload Ambiguity With Functional Variants]
                    functional_overload=f.func.kind() == SchemaKind.mutable,
                ),
                overload_name=f.func.name.overload_name,
            )
        )
    elif k == SchemaKind.out:
        # We generate out= ops mostly just so that we can pair up NativeFunctions into groups easily,
        # but at least today, there is no good reason to actually use them.
        # we'll generate a dispatcher entry for them, but won't actually register any kernels for them.
        if f.func.kind() == SchemaKind.inplace:
            func = self_to_out_signature(f.func)
```
- **EN**: This chunk continues `generate_function` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `generate_function`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 313-340
```python
        elif f.func.kind() == SchemaKind.mutable:
            func = mutable_to_out_signature(f.func)
        elif f.func.kind() == SchemaKind.functional:
            func = functional_to_out_signature(f.func)
        else:
            raise AssertionError(
                "We only bother generating out= functions from either inplace or mutable or functional variants"
            )
    else:
        raise AssertionError(
            "We currently only generate either functional or out= NativeFunctions"
        )

    # Generated kernel naming convention for out: <op_name>_<overload_name>. The reason for this is to
    # disambiguate operator with the same name but different overload name, e.g., `randn.names_out` and
    # `randn.generator_with_names_out`.
    kernel_name = (
        func.name.unambiguous_name()
        if func.kind() == SchemaKind.out
        else cpp.name(func)
    )
    if f.func.has_symint():
        kernel_name += "_symint"
    backend_metadata = {
        DispatchKey.CompositeExplicitAutograd: {
            func.name: BackendMetadata(
                kernel=kernel_name,
                structured=False,
```
- **EN**: This chunk continues `generate_function` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `generate_function`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 341-368
```python
                cpp_namespace=DEFAULT_KERNEL_NAMESPACE,
            )
        }
    }
    tags = {"generated"} | set(
        f.tags & {"nondeterministic_seeded", "view_copy", "pt2_compliant_tag"}
    )

    return (
        NativeFunction(
            func=func,
            use_const_ref_for_mutable_tensors=f.use_const_ref_for_mutable_tensors,
            # These generated fn's aren't meant to be user friendly- don't generate methods.
            variants={Variant.function},
            structured=False,
            structured_delegate=None,
            structured_inherits=None,
            precomputed=None,
            autogen=[],
            ufunc_inner_loop={},
            manual_kernel_registration=False,
            manual_cpp_binding=False,
            python_module=None,
            category_override=None,
            device_guard=False,
            device_check=DeviceCheckType.NoCheck,
            loc=f.loc,
            cpp_no_default_args=set(),
```
- **EN**: This chunk continues `generate_function` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `generate_function`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 369-390
```python
            is_abstract=f.is_abstract,
            has_composite_implicit_autograd_kernel=False,
            has_composite_implicit_autograd_nested_tensor_kernel=False,
            has_composite_explicit_autograd_kernel=True,
            has_composite_explicit_autograd_non_functional_kernel=False,
            # Every generated NativeFunction gets a "generated" tag, so it's easy to tell
            # which NativeFunction objects did not come directly from native_functions.yaml.
            tags=tags,
            namespace=f.namespace,
        ),
        backend_metadata,
    )


# This function is responsible for adding generated NativeFunctions which don't appear
# explicitly in the codegen.
# You can inspect the full list of NativeFunctions yourself with the torchgen package, by running
# torchgen.parse_native_yaml("aten/src/ATen/native/native_functions.yaml", "aten/src/ATen/native/tags.yaml")
# (Maybe we should make a friendly API for this)
#
# Note: this function *mutates* its two inputs,
# adding the new NativeFunctions / BackendMetadata to them
```
- **EN**: This chunk continues `generate_function` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `generate_function`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 391-418
```python
def add_generated_native_functions(
    rs: list[NativeFunction],
    indices: dict[DispatchKey, dict[OperatorName, BackendMetadata]],
) -> None:
    # The main code for generating new NativeFunctions
    # First we group of NativeFunctions by schema kind,
    # then we detect which ones are missing and generate them.
    pre_grouped_native_functions = pre_group_native_functions(rs)
    for d in pre_grouped_native_functions.values():
        has_functional = SchemaKind.functional in d
        has_inplace = SchemaKind.inplace in d
        has_mutable = SchemaKind.mutable in d
        has_out = SchemaKind.out in d
        is_core = any("core" in variant.tags for variant in d.values())

        # We automatically generate a few native functions that don't exist in the yaml, for a few reasons:
        # (1) If an operator has an inplace/out= variant but no functional variant, we can generate
        #     a simple functional variant that the functionalization pass can consume.
        # (2) If an operator has an inplace or functional but no out= variant, we generate an out=
        #     variant, mostly so we can easily pair up functions into NativeFunctionsGroup,
        #     while maintaining the constraint that the out= variant is "required".
        if has_mutable or has_inplace or has_out or has_functional:
            # Don't bother generating functions trio's for native functions that bypass the dispatcher.
            are_manual = all(f.manual_cpp_binding for f in d.values())
            # Don't bother generating functional + out= variants for view operators
            # set_ is technically an inplace_view, but for now it is treated
            # as a normal inplace op in the codegen
            has_view_ops = any(
```
- **EN**: This chunk defines `add_generated_native_functions`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `add_generated_native_functions`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 419-446
```python
                f.is_view_op and str(f.func.name.name) != "set_" for f in d.values()
            )
            # Don't generate the other variants for non-core CompositeImplicitAutograd operators.
            # We could probably do this, but the main benefit of generating the function triplets
            # is for transforms that need them, and transforms don't need to act directly
            # on CompositeImplicitAutograd operators (since we let them decompose).
            are_composite_implicit = all(
                f.has_composite_implicit_autograd_kernel for f in d.values()
            )
            if are_manual or has_view_ops or are_composite_implicit and not is_core:
                continue
            if has_out and len(d.values()) == 1:
                # Note: [Out ops with functional variants that don't get grouped properly]
                # In theory we could validly have an out= operator in native_functions.yaml
                # that has no other variants.
                # But today, all of the operators where that's the case actually do have
                # functional variants, that we are just unable to pair up properly.
                # I think banning this all together is probably safer
                # (you can always add a functional variant yourself if you want to add a new out= operator).
                #
                # We should probably fix the existing cases; this check is to prevent us from adding more over time.
                if (
                    str(d[SchemaKind.out].func.name)
                    not in OUT_OPS_THAT_DONT_GET_GROUPED_PROPERLY
                ):
                    raise AssertionError(
                        f"Found an out= operator that we could not find any other variants of: {str(d[SchemaKind.out].func)}"
                    )
```
- **EN**: This chunk continues `add_generated_native_functions` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `add_generated_native_functions`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 447-466
```python
                continue

            # Some inplace ops that have problematic schemas (that we should fix), which prevent us
            # from generating out= and functional variants
            if (
                has_inplace
                and str(d[SchemaKind.inplace].func.name)
                in INPLACE_OPS_THAT_DONT_GET_GROUPED_PROPERLY
            ):
                continue

            base_fn = (
                d[SchemaKind.mutable]
                if has_mutable
                else d[SchemaKind.inplace]
                if has_inplace
                else d[SchemaKind.out]
                if has_out
                else d[SchemaKind.functional]
            )
```
- **EN**: This chunk continues `add_generated_native_functions` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `add_generated_native_functions`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 468-495
```python
            # Note: [Mutable ops that cannot get an out variant]
            # We can only generate an out= variant if either:
            # - the original function has tensor-like returns (since we can convert them to out kwargs)
            # - or it's inplace (since we can convert `self` to an out kwarg)
            # There are only two functions that don't fit this criteria today though,
            # and they both look like they should be fixed to be out= variants,
            # so if feels safer to ban this schema all-together
            base_fn_valid = base_fn.func.kind() == SchemaKind.inplace or any(
                r.type.is_tensor_like() for r in base_fn.func.returns
            )
            # Note: [Loosen the assertion that all functional should have out variant]
            # By design all functional operators should have our variants. The needs_out check
            # is loosening this requirement, changing it to only generate out variant if there's
            # an `autogen` block in the native function, in the long run it should be removed.
            # FIXME: Remove this after figuring out CI job failures related to min, max, mean
            needs_out = any("out" in str(op_name) for op_name in base_fn.autogen)
            gets_out_variant = not has_out and base_fn_valid and needs_out
            if not has_out and not base_fn_valid:
                if (
                    str(base_fn.func.name)
                    not in MUTABLE_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT
                    and str(base_fn.func.name)
                    not in FUNCTIONAL_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT
                ):
                    raise AssertionError(
                        f"""Found an operator that we could not generate an out= variant for: {str(base_fn.func)}.
This type of operators don't have tensor-like return, making it difficult to generate a proper out= variant. If
out= variant is not needed, please add the function name into FUNCTIONAL_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT list."""
```
- **EN**: This chunk continues `add_generated_native_functions` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `add_generated_native_functions`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 496-512
```python
                    )

            # Generate an out= variant
            if gets_out_variant:
                fn, metadata = generate_function(base_fn, SchemaKind.out)
                d[SchemaKind.out] = fn
                BackendIndex.grow_index(indices, metadata)
                rs.append(fn)

            # Generate a functional variant, but only do it if the operator got an out= variant
            # (Functional variants are only useful if we can group up the variants,
            # which we can only do if they have an out= variant)
            if not has_functional and (has_out or gets_out_variant):
                fn, metadata = generate_function(base_fn, SchemaKind.functional)
                d[SchemaKind.functional] = fn
                BackendIndex.grow_index(indices, metadata)
                rs.append(fn)
```
- **EN**: This chunk continues `add_generated_native_functions` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `add_generated_native_functions`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 515-539
```python
def return_str(rets: tuple[Return, ...], names: list[str]) -> str:
    if len(rets) != len(names):
        raise AssertionError(
            f"Returns and names length mismatch: {len(rets)} vs {len(names)}"
        )
    if len(rets) == 0:
        return ""
    elif len(rets) == 1:
        return f"return {names[0]};"
    else:
        return f"return {dispatcher.returns_type(rets).cpp_type()}({', '.join(names)});"


# Given a function, and the name of a variable corresponding to the output of that function,
# gather up all of the individual returns that are not aliased
def gather_nonaliased_inner_rets(func: FunctionSchema, out_var: str) -> list[str]:
    aliased_rets = func.aliased_return_names()
    non_aliased_names = []
    is_out_var_a_tuple = len(func.returns) > 1
    for i, r in enumerate(aliased_rets):
        if r is None:
            non_aliased_names.append(
                f"std::get<{i}>({out_var})" if is_out_var_a_tuple else out_var
            )
    return non_aliased_names
```
- **EN**: This chunk defines `gather_nonaliased_inner_rets`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `gather_nonaliased_inner_rets`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 542-560
```python
# Generates functional kernels in terms of their inplace.mutable counterparts.
# We only do this for "generated" NativeFunctions
@with_native_function
def gen_composite_functional_kernel(g: NativeFunctionsGroup) -> str | None:
    # We should only be generating these for code-generated NativeFunctions
    if "generated" not in g.functional.tags:
        return None
    # And we always write the kernel for a generated op in terms of a non-generated op.
    if g.inplace is not None and "generated" not in g.inplace.tags:
        target_f = g.inplace
    elif g.mutable is not None and "generated" not in g.mutable.tags:
        target_f = g.mutable
    else:
        # We should be guaranteed to have a valid inplace/mutable variant to call into.
        # See Note: [Mutable Ops Not Using Functionalization]
        raise AssertionError(str(g.functional.func))

    sig = DispatcherSignature(g.functional.func)
    target_sig = DispatcherSignature(target_f.func)
```
- **EN**: Decorators such as @with_native_function modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `gen_composite_functional_kernel`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @with_native_function 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `gen_composite_functional_kernel`，其作用是根据高层元数据生成派生代码、表或分析产物。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 562-586
```python
    context: list[Binding | Expr] = []
    clone_mutable_inputs = []
    cloned_return_names = []
    # We can't just directly pass all of the arguments from the functional op into the mutating op.
    # We need to check for which inputs to the mutating operator are mutable,
    # and clone those inputs first.
    for a_curr, a_tgt in zip(
        dispatcher.jit_arguments(g.functional.func),
        dispatcher.jit_arguments(target_f.func),
    ):
        if a_tgt.annotation is not None and a_tgt.annotation.is_write:
            clone_mutable_inputs.append(
                f"auto {a_curr.name}_clone = clone_arg({a_curr.name});"
            )
            context.append(
                Expr(
                    expr=f"{a_curr.name}_clone",
                    type=dispatcher.argument_type(a_curr, binds=a_curr.name),
                )
            )
            # Invariant: mutable arguments on the inner mutable op are always returns on the functional op.
            cloned_return_names.append(f"{a_curr.name}_clone")
        else:
            context.append(dispatcher.argument(a_curr))
    exprs = ", ".join([e.expr for e in translate(context, target_sig.arguments())])
```
- **EN**: This chunk continues `gen_composite_functional_kernel` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `gen_composite_functional_kernel`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 588-607
```python
    out_name = "output"
    maybe_assign = f"auto {out_name} = " if len(target_f.func.returns) > 0 else ""
    inner_return_names = gather_nonaliased_inner_rets(target_f.func, out_name)
    ret_str = return_str(
        g.functional.func.returns, inner_return_names + cloned_return_names
    )

    clone_mutable_inputs_str = "\n".join(clone_mutable_inputs)
    return f"""
{sig.defn(name=sig.name() + ("_symint" if g.out.func.has_symint() else ""))} {{
  {clone_mutable_inputs_str}
  {maybe_assign}at::_ops::{target_f.func.name.unambiguous_name()}::call({exprs});
  {ret_str}
}}
"""


# Generates out= kernels in terms of their functional counterparts.
# We only do this for "generated" NativeFunctions
@with_native_function
```
- **EN**: Decorators such as @with_native_function modify how the following Python definitions are exported, wrapped, or dataclass-managed. Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk continues `gen_composite_functional_kernel` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @with_native_function 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段延续了 `gen_composite_functional_kernel`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 608-635
```python
def gen_composite_out_kernel(g: NativeFunctionsGroup) -> str | None:
    # We should only be generating these for code-generated NativeFunctions
    if "generated" not in g.out.tags:
        return None
    # And we always write the kernel for the out= op in terms of the functional.
    # Note that the functional op might have also been generated, but we don't have to
    # worry about cycles, because the generated functional kernels are always implemented
    # in terms of non-generated kernels (see gen_composite_functional_kernel).

    sig = DispatcherSignature(g.out.func)
    target_sig = DispatcherSignature(g.functional.func)

    exprs = ", ".join(
        [e.expr for e in translate(sig.arguments(), target_sig.arguments())]
    )

    copy_outs = []
    out_name = "tmp_output"
    for i, out_arg in enumerate(g.out.func.arguments.out):
        functional_return_name = (
            out_name
            if len(g.functional.func.returns) == 1
            else f"std::get<{i}>({out_name})"
        )
        copy_outs.append(
            f"""\
  resize_out_helper({out_arg.name}, {functional_return_name});
  copy_arg({out_arg.name}, {functional_return_name});"""
```
- **EN**: This chunk defines `resize_out_helper`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `resize_out_helper`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 636-662
```python
        )

    rets = []
    # For each return arg in the calling (out=) operator,
    # If it corresponds to an aliased input, return the input.
    # Otherwise, return the corresponding output from calling the functional operator.
    for i, ret_name in enumerate(g.out.func.aliased_return_names()):
        if ret_name is not None:
            rets.append(ret_name)
        else:
            functional_return_name = (
                out_name
                if len(g.functional.func.returns) == 1
                else f"std::get<{i}>({out_name})"
            )
            rets.append(functional_return_name)

    copy_outs_str = "\n".join(copy_outs)

    # Kernel name needs to follow the naming convention defined in `generate_function()`
    return f"""
{sig.defn(name=g.out.func.name.unambiguous_name() + ("_symint" if g.out.func.has_symint() else ""))} {{
  auto {out_name} = at::_ops::{g.functional.func.name.unambiguous_name()}::call({exprs});
  {copy_outs_str}
  {return_str(g.out.func.returns, rets)}
}}
"""
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk continues `resize_out_helper` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段延续了 `resize_out_helper`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Torch operator code generation**
  - EN: Builds derived code artifacts from declarative operator metadata.
  - CN: 从声明式算子元数据构建派生代码产物。
- **Dispatch-key plumbing**
  - EN: Models dispatch keys, backend selection, or registration routing.
  - CN: 建模 dispatch key、后端选择或注册路由。
- **Model training**
  - EN: Runs data-driven training loops to fit heuristic or regression models.
  - CN: 运行数据驱动的训练循环，以拟合启发式或回归模型。
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
- **OUT_OPS_THAT_DONT_GET_GROUPED_PROPERLY**
  - EN: `OUT_OPS_THAT_DONT_GET_GROUPED_PROPERLY` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `OUT_OPS_THAT_DONT_GET_GROUPED_PROPERLY` 是本文件声明、导出或驱动的显著符号之一。
- **MUTABLE_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT**
  - EN: `MUTABLE_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `MUTABLE_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torchgen.api.dispatcher`, `torchgen.api.translate`, `torchgen.api.types`, `torchgen.context`, `torchgen.model`, `torchgen.utils`, `torchgen.api`
- **Standard library / 标准库**: `__future__`, `string`, `collections`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `OUT_OPS_THAT_DONT_GET_GROUPED_PROPERLY`, `MUTABLE_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT`, `FUNCTIONAL_OPS_THAT_CANNOT_GET_AN_OUT_VARIANT`, `INPLACE_OPS_THAT_DONT_GET_GROUPED_PROPERLY`, `pre_group_native_functions`, `get_expected_out_variant_overload_name`, `self_to_out_signature`, `functional_to_out_signature`, `generate_out_args_from_schema`, `mutable_to_out_signature`
