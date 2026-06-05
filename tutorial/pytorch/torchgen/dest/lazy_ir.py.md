# lazy_ir.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/dest/lazy_ir.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements backends that render the in-memory torchgen model into concrete source files and registrations.
- **Purpose (CN)**: 实现把 torchgen 内存模型渲染成具体源文件与注册代码的后端。
## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```python
from __future__ import annotations

import itertools
from abc import ABC
from dataclasses import dataclass
from typing import Any

import torchgen.api.dispatcher as dispatcher
from torchgen.api.lazy import (
    getValueT,
    isValueType,
    LazyArgument,
    LazyIrProperties,
    LazyIrSchema,
    tensorListValueT,
)
from torchgen.api.translate import translate
from torchgen.api.types import (
    BaseCType,
    Binding,
    deviceT,
    DispatcherSignature,
    kernel_signature,
    NativeSignature,
    OptionalCType,
    VectorCType,
)
from torchgen.context import method_with_native_function
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.api.dispatcher, torchgen.api.lazy, torchgen.api.translate, torchgen.api.types, and 1 more; standard-library modules such as __future__, itertools, abc, dataclasses, and 1 more for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.api.dispatcher、torchgen.api.lazy、torchgen.api.translate、torchgen.api.types 等共 5 项；标准库模块，如 __future__、itertools、abc、dataclasses 等共 5 项组织在一起，供下方逻辑使用。

### Lines 29-48
```python
from torchgen.dest.lazy_ts_lowering import ts_lowering_body
from torchgen.model import (
    Argument,
    BackendIndex,
    BackendMetadata,
    BaseTy,
    BaseType,
    FunctionSchema,
    ListType,
    NativeFunction,
    NativeFunctionsGroup,
)


def node_ctor_arg_rvalue_string(arg: LazyArgument) -> str:
    """
    Given a LazyArgument,
    generate a c++ string for materializing an rvalue of that arg for passing into
    a lazy Node constructor.
    """
```
- **EN**: The import section wires together PyTorch-local modules such as torchgen.dest.lazy_ts_lowering, torchgen.model for the logic below. This chunk defines `node_ctor_arg_rvalue_string`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries.
- **CN**: 导入区把PyTorch 本地模块，如 torchgen.dest.lazy_ts_lowering、torchgen.model组织在一起，供下方逻辑使用。 这一段定义了 `node_ctor_arg_rvalue_string`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。

### Lines 50-77
```python
    # TODO: Matching on CType seems wrong; should be matching on Type
    if isValueType(arg.lazy_type):
        if isinstance(arg.lazy_type, BaseCType):
            if arg.is_wrapped_scalar:
                return f"node_{arg.name}"
            elif arg.lazy_type.type is tensorListValueT:
                return f"lazy_{arg.name}_tensorlist"
            elif arg.is_symint_or_list:
                return f"GetSymIntValue({arg.name})"
            return f"lazy_{arg.name}->GetIrValue()"
        elif isinstance(arg.lazy_type, OptionalCType):
            if arg.is_symint_or_list:
                # TODO: I don't understand when you should put lazy_ in the name
                # or not
                return f"{arg.name} ? std::make_optional(GetSymIntValue(*{arg.name})) : ::std::nullopt"
            elif arg.is_wrapped_scalar:
                return f"node_{arg.name}"
            return (
                f"lazy_{arg.name} ? "
                f"std::make_optional(lazy_{arg.name}->GetIrValue()) : "
                "::std::nullopt"
            )
        else:
            raise AssertionError(
                f"TODO not sure if there are other valid types to handle here ({arg.lazy_type})"
            )
    else:
        # NB: this is here because right now we aren't treating SymInt[] as a
```
- **EN**: This chunk continues `node_ctor_arg_rvalue_string` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `node_ctor_arg_rvalue_string`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 78-99
```python
        # value type; when we do this needs to move above
        # NB: we cannot test arg.lazy_type as we've already specified it is an
        # int64_t and so we cannot distinguish between SymInt and int64_t
        if isinstance(arg.orig_type, ListType) and arg.orig_type.elem == BaseType(
            BaseTy.SymInt
        ):
            if arg.symint:
                return f"GetSymIntArrayRefValue({arg.name})"
            else:
                return f"std::vector<int64_t>({arg.name}.begin(), {arg.name}.end())"
        elif isinstance(arg.lazy_type, VectorCType) and isinstance(
            arg.lazy_type.elem, BaseCType
        ):
            return f"std::vector<{arg.lazy_type.elem.type}>({arg.name}.begin(), {arg.name}.end())"
        elif (
            isinstance(arg.lazy_type, OptionalCType)
            and isinstance(arg.lazy_type.elem, VectorCType)
            and isinstance(arg.lazy_type.elem.elem, BaseCType)
        ):
            return f"torch::lazy::ToOptionalVector<{arg.lazy_type.elem.elem.type}>({arg.name})"
        else:
            return f"{arg.name}"
```
- **EN**: This chunk continues `node_ctor_arg_rvalue_string` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `node_ctor_arg_rvalue_string`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 102-129
```python
def node_ctor_inputs(schema: LazyIrSchema) -> str:
    """
    Produce a formatted string with the arguments as passed into the constructor of a node class.
    """
    node_ctor_values = [
        node_ctor_arg_rvalue_string(arg) for arg in schema.filtered_args()
    ]
    return ", ".join(node_ctor_values)


def gen_fallback_code(
    schema: LazyIrSchema,
    sig: DispatcherSignature | NativeSignature,
    overload_name: str,
) -> str:
    """
    Generate code that falls back to eager conditioned on a predicate
    """
    dispatcher_sig = DispatcherSignature.from_schema(schema.func)
    exprs = translate(sig.arguments(), dispatcher_sig.arguments())
    fallback_args = ",\n                ".join([a.expr for a in exprs])
    if len(overload_name):
        aten_op_str = f"ATEN_OP2({schema.aten_name}, {overload_name})"
    else:
        aten_op_str = f"ATEN_OP({schema.aten_name})"
    return f"""
        if (force_eager_fallback({aten_symbol(schema)})) {{
            return at::native::call_fallback_fn_symint<&ltc_eager_fallback, {aten_op_str}>::call(
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk defines `gen_fallback_code`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段定义了 `gen_fallback_code`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 130-151
```python
                {fallback_args}
            );
        }}
"""


def aten_symbol(schema: LazyIrSchema) -> str:
    missing_interned_strings = {
        "sigmoid_backward",
    }
    if schema.aten_name in missing_interned_strings:
        return f'c10::Symbol::fromQualString("aten::{schema.aten_name}")'

    if not schema.aten_name.startswith("at::"):
        return f"at::aten::{schema.aten_name}"
    else:
        return schema.aten_name


# converts  all tensor-like arguments to meta tensors. Returns:
# (1) a string containing all of the logic that does the conversions.
# (2) a context, to be used by translate(), with all of the relevant bindings.
```
- **EN**: This chunk defines `aten_symbol`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `aten_symbol`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 152-175
```python
def convert_to_meta_tensors(sig: DispatcherSignature) -> tuple[str, list[Binding]]:
    context: list[Binding] = []
    unwrapped_tensor_args: list[str] = []
    for arg in sig.arguments():
        if isinstance(arg.argument, Argument) and arg.argument.type.is_tensor_like():
            unwrapped_name = f"{arg.name}_meta"
            unwrapped_tensor_args.append(
                f"auto {unwrapped_name} = to_meta({arg.name});"
            )
            context.append(arg.with_name(unwrapped_name))
        else:
            context.append(arg)
    unwrap_tensor_args_str = "\n        ".join(unwrapped_tensor_args)
    return unwrap_tensor_args_str, context


@dataclass(frozen=True)
class GenLazyIR(ABC):
    backend_index: BackendIndex
    backend_name: str
    node_base: str
    use_lazy_shape: bool

    @method_with_native_function
```
- **EN**: Decorators such as @dataclass(frozen=True), @method_with_native_function modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends GenLazyIR, which hold the primary data model or public surface for this slice of the file. This chunk defines `convert_to_meta_tensors`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @dataclass(frozen=True)、@method_with_native_function 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 GenLazyIR，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `convert_to_meta_tensors`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 176-197
```python
    def __call__(self, f: NativeFunctionsGroup | NativeFunction) -> list[str]:
        func = f.functional.func if isinstance(f, NativeFunctionsGroup) else f.func
        metadata = self.backend_index.get_kernel(
            f.functional if isinstance(f, NativeFunctionsGroup) else f
        )
        schema = LazyIrSchema(
            func, symint=metadata is not None and metadata.supports_symint()
        )
        return self.gen(schema)

    # there is no lowering functionality generated unless this IR base class is subclassed and
    # implemented as a backend-specific node
    def lowering_function(self, schema: LazyIrSchema) -> str:
        return ""

    def create_function(self, schema: LazyIrSchema, node_ctor_args: str) -> str:
        return ""

    def can_be_reused_function(self, schema: LazyIrSchema, node_ctor_args: str) -> str:
        return f"""bool CanBeReused({node_ctor_args}) const {{
    return false;
    }}"""
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. It introduces or extends is, which hold the primary data model or public surface for this slice of the file. This chunk defines `can_be_reused_function`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 它引入或扩展了 is，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `can_be_reused_function`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 199-226
```python
    def node_base_ctor_call(self, schema: LazyIrSchema) -> str:
        value_args = schema.filtered_args(values=True, scalars=False)
        # backends can customize the way the node base class constructor is called,
        # as long as all of its arguments can be generated from information available from the schema
        base_ctor_value_args_list = []
        for arg in value_args:
            if isinstance(arg.lazy_type, (BaseCType, VectorCType)):
                base_ctor_value_args_list.append(f"{arg.name}")
            elif isinstance(arg.lazy_type, OptionalCType):
                base_ctor_value_args_list.append(f"{arg.name}.value_or(kNullValue)")
            else:
                raise AssertionError(
                    f"Unsupported type ({arg.lazy_type}) - add support if necessary"
                )
        base_ctor_value_args = ", ".join(base_ctor_value_args_list)

        scalar_args = schema.filtered_args(values=False, scalars=True)

        # Shape construction.
        # Conditionally build shape depending on specified shape property
        if schema.properties.ShapePrecompute:
            shape_ctor_arg = "std::move(shapes),"
        elif schema.properties.ShapeCompute:
            shape_args = [a.name for a in value_args]
            shape_args.extend(a.name for a in scalar_args)
            shape_ctor_arg = f"compute_shape_{schema.name}({', '.join(shape_args)}),"
        elif schema.properties.ShapeCache:
            shape_args = [f"operand({i})" for i in range(len(value_args))]
```
- **EN**: This region behaves like generated or templated source, so the important signal is the emitted schema/layout rather than bespoke control flow. It introduces or extends constructor, which hold the primary data model or public surface for this slice of the file. This chunk defines `node_base_ctor_call`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段更像生成代码或模板，因此重点在于最终发射的 schema/布局，而不是手写控制流。 它引入或扩展了 constructor，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `node_base_ctor_call`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 227-253
```python
            shape_args.extend(a.name for a in scalar_args)
            shape_ctor_arg = f"[&](){{ return compute_shape_{schema.name}({', '.join(shape_args)})[0]; }},"
        else:
            shape_ctor_arg = ""

        scalar_hashes = ", ".join(f"{a.name}" for a in scalar_args)

        return f"""{self.node_base}(
              {schema.node_name}::ClassOpKind(),
              OpList{{{base_ctor_value_args}}},
              {shape_ctor_arg}
              /* num_outputs */ {len(schema.returns)},
              torch::lazy::MHash({scalar_hashes}))"""

    def gen(self, schema: LazyIrSchema) -> list[str]:
        opkind = schema.opkind or aten_symbol(schema)

        # for now, we just want one IR class decl and soon after also the method defs
        # and we use the functional version not out/inplace.
        all_args = schema.filtered_args()
        scalar_args = schema.filtered_args(values=False, scalars=True)

        ctor_args = [f"const {i.lazy_type.cpp_type()}& {i.name}" for i in all_args]
        reuse_ctor_args = ", ".join(ctor_args)
        if self.use_lazy_shape and schema.properties.ShapePrecompute:
            ctor_args.append("std::vector<torch::lazy::Shape>&& shapes")
        node_ctor_args = ", ".join(ctor_args)
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. It introduces or extends decl, which hold the primary data model or public surface for this slice of the file. This chunk defines `gen`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 它引入或扩展了 decl，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `gen`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 255-282
```python
        scalar_initializers = ",\n        ".join(
            [
                # This code is just special casing the mapping from string_view -> strings
                f"{a.name}({a.name}.has_value() ? ::std::make_optional(std::string(*{a.name})) : ::std::nullopt)"
                if a.lazy_type.cpp_type() == "::std::optional<c10::string_view>"
                else f"{a.name}({a.name})"
                for a in scalar_args
            ]
        )
        if len(scalar_initializers):
            scalar_initializers = f",\n        {scalar_initializers}"
        scalar_decls = "\n  ".join(
            [
                f"std::string {a.name};"
                if a.lazy_type.cpp_type() == "c10::string_view"
                else f"::std::optional<std::string> {a.name};"
                if a.lazy_type.cpp_type() == "::std::optional<c10::string_view>"
                else f"{a.lazy_type.cpp_type()} {a.name};"
                for a in scalar_args
            ]
        )
        optional_values = [
            arg.name
            for arg in schema.filtered_args(values=True, scalars=False)
            if isinstance(arg.lazy_type, OptionalCType)
        ]
        has_optional_decls = "\n  ".join(
            [f"bool has_{value}: 1;" for value in optional_values]
```
- **EN**: This chunk continues `gen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `gen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 283-305
```python
        )
        has_optional_defs = "\n    ".join(
            [f"has_{value} = !!{value};" for value in optional_values]
        )
        members_to_string = []
        for arg in scalar_args:
            if isinstance(arg.lazy_type, OptionalCType):
                value = f"{arg.name}.value()"
                if arg.is_generator:
                    value = '"torch.Generator()"'
                members_to_string.append(
                    f"""if ({arg.name}.has_value()) {{
      ss << ", {arg.name}=" << {value};
    }} else {{
      ss << ", {arg.name}=null";
    }}"""
                )
            else:
                members_to_string.append(f'ss << ", {arg.name}=" << {arg.name};')
        members_to_string_str = "\n    ".join(members_to_string)

        return [
            f"""\
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk continues `gen` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段延续了 `gen`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 306-332
```python
class {schema.node_name} : public {self.node_base} {{
 public:
  static torch::lazy::OpKind ClassOpKind() {{
    return torch::lazy::OpKind({opkind});
  }}

  {schema.node_name}({node_ctor_args})
      : {self.node_base_ctor_call(schema)}{scalar_initializers}
  {{
    {has_optional_defs}
  }}

  std::string ToString() const override {{
    std::stringstream ss;
    ss << {self.node_base}::ToString();
    {members_to_string_str}
    return ss.str();
  }}

  {self.create_function(schema, reuse_ctor_args)}

  {self.can_be_reused_function(schema, reuse_ctor_args)}

  {self.lowering_function(schema)}

  {scalar_decls}
  {has_optional_decls}
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk defines `ToString`, which implements one step in the operator code-generation pipeline. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段定义了 `ToString`，其作用是实现算子代码生成流水线中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 334-356
```python
}};

""",
        ]


@dataclass(frozen=True)
class GenTSLazyIR(GenLazyIR):
    def lowering_function(self, schema: LazyIrSchema) -> str:
        signature = """
  torch::lazy::TSOpVector Lower(
      std::shared_ptr<torch::jit::GraphFunction> function,
      torch::lazy::TSLoweringContext* loctx) const override"""

        if schema.properties.LowerDeclOnly:
            return f"{signature};"
        elif schema.properties.Lower:
            return f"""{signature} {{
    {ts_lowering_body(schema)}
  }}
            """
        else:
            return ""
```
- **EN**: Decorators such as @dataclass(frozen=True) modify how the following Python definitions are exported, wrapped, or dataclass-managed. Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. It introduces or extends GenTSLazyIR, which hold the primary data model or public surface for this slice of the file. This chunk defines `lowering_function`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @dataclass(frozen=True) 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 它引入或扩展了 GenTSLazyIR，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `lowering_function`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 358-385
```python
    def create_function(self, schema: LazyIrSchema, node_ctor_args: str) -> str:
        signature = f"static NodePtr Create({node_ctor_args})"
        if schema.properties.CreateFnDeclOnly:
            return f"{signature};"
        elif not schema.properties.CreateFn:
            return ""
        return f"""{signature} {{
    return ReuseOrMakeNode<{schema.node_name}>(data);
  }}"""

    def can_be_reused_function(self, schema: LazyIrSchema, node_ctor_args: str) -> str:
        signature = f"bool CanBeReused({node_ctor_args}) const"
        if schema.properties.CanBeReusedDeclOnly:
            return f"{signature};"
        elif not schema.properties.CanBeReused:
            return ""
        value_comparison = []
        for arg in itertools.chain(schema.positional_values, schema.keyword_values):
            if isinstance(arg.lazy_type, OptionalCType):
                value_comparison.append(
                    f"nullable_operand(i++) == {arg.name}.value_or(kNullValue)"
                )
            else:
                value_comparison.append(f"operand(i++) == {arg.name}")
        for arg in itertools.chain(schema.positional_scalars, schema.keyword_scalars):
            if isinstance(arg.lazy_type, OptionalCType):
                value_comparison.append(
                    f"((!this->{arg.name}&&!{arg.name}) || (this->{arg.name}&&{arg.name} && *(this->{arg.name}) == *{arg.name}))"
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk defines `can_be_reused_function`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段定义了 `can_be_reused_function`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 386-413
```python
                )
            else:
                value_comparison.append(f"this->{arg.name} == {arg.name}")
        value_comparison_str = " &&\n        ".join(value_comparison)

        return f"""{signature} {{
    size_t i = 0;
    return ({value_comparison_str});
  }}"""


@dataclass(frozen=True)
class GenLazyNativeFuncDefinition:
    class_method_name: str
    backend_index: BackendIndex
    tensor_class: str
    gen_forced_fallback_code: bool
    backend_namespace: str
    get_tensorlist: str
    get_tensor_or_wrap_number: str
    try_get_tensor: str
    metrics_counter: str
    create_tensor: str
    create_from_first_tensor: bool
    create_aten_from_ltc_tensor: str
    tuple_aten_from_ltc_tensors: str
    lazy_tensor_ptr: str
    get_device_fn: str
```
- **EN**: Decorators such as @dataclass(frozen=True) modify how the following Python definitions are exported, wrapped, or dataclass-managed. Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. It introduces or extends GenLazyNativeFuncDefinition, which hold the primary data model or public surface for this slice of the file. This chunk continues `GenLazyNativeFuncDefinition` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @dataclass(frozen=True) 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 它引入或扩展了 GenLazyNativeFuncDefinition，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `GenLazyNativeFuncDefinition`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 415-442
```python
    def lazy_tensor_decls(self, func: NativeFunction, schema: LazyIrSchema) -> str:
        value_args = schema.filtered_args(values=True, scalars=False)
        # Generates lazy_{name} variables for LazyTensors wrapping input tensors
        lazy_tensor_decls: list[str] = []
        for arg in value_args:
            if arg.is_wrapped_scalar:
                if isinstance(arg.lazy_type, OptionalCType):
                    lazy_tensor_decls.append(
                        f"""auto node_{arg.name} = {arg.name} ?
                std::make_optional(torch::lazy::LazyGraphExecutor::Get()->
                    GetIrValueForScalarFromCodegen(*{arg.name}, *common_device)):
                ::std::nullopt;"""
                    )
                else:
                    lazy_tensor_decls.append(
                        f"""auto node_{arg.name} = torch::lazy::LazyGraphExecutor::Get()->
                            GetIrValueForScalarFromCodegen({arg.name}, *common_device);"""
                    )
            elif arg.is_symint_or_list:
                continue  # values are extracted in isValueType
            elif isinstance(arg.lazy_type, BaseCType):
                if arg.lazy_type.type is tensorListValueT:
                    lazy_tensor_decls.append(
                        f"auto lazy_{arg.name}_tensorlist = "
                        f"{self.backend_namespace}::{self.get_tensorlist}({arg.name});"
                    )
                else:
                    lazy_tensor_decls.append(
```
- **EN**: This chunk defines `lazy_tensor_decls`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `lazy_tensor_decls`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 443-462
```python
                        f"{self.lazy_tensor_ptr} lazy_{arg.name} = "
                        f"{self.backend_namespace}::{self.get_tensor_or_wrap_number}({arg.name}, *common_device);"
                    )
            elif isinstance(arg.lazy_type, OptionalCType):
                if arg.lazy_type.elem != BaseCType(getValueT()):
                    raise AssertionError(
                        f"Expected OptionalCType elem to be {BaseCType(getValueT())}, "
                        f"got {arg.lazy_type.elem}"
                    )
                # TODO(alanwaketan): Maybe we want to apply GetLtcTensorOrCreateForWrappedNumber here, but hold it
                # until we encounter a real world example.
                lazy_tensor_decls.append(
                    f"{self.lazy_tensor_ptr} lazy_{arg.name} = "
                    f"{self.backend_namespace}::{self.try_get_tensor}({arg.name}.value_or(at::Tensor()));"
                )
            else:
                raise AssertionError(
                    f"TODO not sure if there are other valid types to handle here ({arg.lazy_type})"
                )
        return ("\n        ").join(lazy_tensor_decls)
```
- **EN**: This chunk continues `lazy_tensor_decls` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `lazy_tensor_decls`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 464-491
```python
    def force_eager_fallback(
        self,
        func: NativeFunction,
        schema: LazyIrSchema,
        metadata: BackendMetadata,
        sig: DispatcherSignature | NativeSignature,
    ) -> str:
        if self.gen_forced_fallback_code:
            return gen_fallback_code(
                schema, sig, overload_name=func.func.name.overload_name
            )
        return ""

    def metrics(self, func: NativeFunction, schema: LazyIrSchema) -> str:
        return f"{self.metrics_counter};"

    def get_device(self, func: NativeFunction, schema: LazyIrSchema) -> str:
        value_args = schema.filtered_args(values=True, scalars=False)
        scalar_args = schema.filtered_args(values=False, scalars=True)
        value_types_names = [f"{a.name}" for a in value_args if not a.is_wrapped_scalar]
        optional_device = OptionalCType(BaseCType(deviceT))
        optional_devices = [
            a.name for a in scalar_args if a.lazy_type == optional_device
        ]
        if len(value_types_names) == 0 and len(optional_devices) == 0:
            raise AssertionError("Expected at least one Value or Device type")
        get_device_str = (
            f"{self.get_device_fn}({', '.join(value_types_names + optional_devices)})"
```
- **EN**: This chunk defines `get_device`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_device`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 492-518
```python
        )
        return f"""auto common_device = {get_device_str};
        TORCH_INTERNAL_ASSERT(common_device);
        """

    def shape_inference(self, func: NativeFunction, schema: LazyIrSchema) -> str:
        metadata = self.backend_index.get_kernel(func)
        if metadata is None:
            raise AssertionError(f"No kernel metadata found for {func.func.name}")
        all_args = schema.filtered_args()
        returns_length = len(schema.returns)
        # call the meta kernel if it exists, to compute output shape/dtype for our IR
        # Note [Generated LTC Shape Functions]
        # LTC uses meta tensors from core to do shape inference when possible, and otherwise
        # we generate a shape function declaration that needs to be manually implemented.
        # How do we detect which ops are eligible to use meta tensors?
        # In general we should be able to use meta tensors not just on structured operators,
        # but also on composite operators that are implemented in terms of structured kernels.
        # We don't currently have a way of knowing at codegen time which ops are implemented that way.
        # This is the case for all view and view_copy operators however, so we're going to
        # use them specifically for all of the view_copy ops (instead of manually writing shape rules for all of them).
        is_view_copy_op = "view_copy" in func.tags
        is_structured = func.structured or func.structured_delegate is not None
        if is_structured or is_view_copy_op:
            meta_out = """
std::vector<torch::lazy::Shape> shapes{torch::lazy::Shape(out_meta.scalar_type(), out_meta.sizes().vec())};"""
            if returns_length > 1:
```
- **EN**: This chunk defines `TORCH_INTERNAL_ASSERT`, which verifies invariants and catches incorrect states early. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `TORCH_INTERNAL_ASSERT`，其作用是验证不变量，并尽早捕获错误状态。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 520-547
```python
                def this_shape(i: int) -> str:
                    return f"torch::lazy::Shape(std::get<{i}>(out_meta).scalar_type(), std::get<{i}>(out_meta).sizes().vec())"

                shapes_str = ",".join([this_shape(i) for i in range(returns_length)])
                meta_out = "std::vector<torch::lazy::Shape> shapes{" + shapes_str + "};"

            # Convert tensor args to the meta device and call it.
            # (We can't pass in the input tensors directly, because they are "functional wrappers".
            # If any of the meta kernels call a tensor op and redispatch, we don't want to hit the functionalize kernels.)
            # Even at::meta:: functions might redispatch, e.g. if they call into view ops.
            dispatcher_sig = DispatcherSignature.from_schema(func.func)
            meta_conversion_str, meta_call_ctx = convert_to_meta_tensors(dispatcher_sig)
            meta_call_args = [
                e.expr
                for e in translate(
                    meta_call_ctx, dispatcher_sig.arguments(), method=False
                )
            ]
            if is_view_copy_op:
                # view_copy ops always have a CompositeExplicitAutogradNonFunctional kernel
                if not func.has_composite_explicit_autograd_non_functional_kernel:
                    raise AssertionError(
                        f"view_copy op {func.func.name} must have "
                        "CompositeExplicitAutogradNonFunctional kernel"
                    )
                dispatch_ns = "compositeexplicitautogradnonfunctional"
            else:
                dispatch_ns = "meta"
```
- **EN**: This chunk defines `this_shape`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `this_shape`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 548-575
```python
            aten_name = schema.aten_name
            # TODO: this is trolling
            if func.func.has_symint() and metadata.supports_symint():
                aten_name += "_symint"
            shape_str = f"""\
        {meta_conversion_str}
        auto out_meta = at::{dispatch_ns}::{aten_name}({", ".join(meta_call_args)});
        {meta_out}"""
        else:
            shape_sig = ComputeShapeSignature(
                metadata.kernel, func, symint=metadata.supports_symint()
            )
            shape_str = f"""
            auto shapes = {shape_sig.shape_call};"""

        shape_str += f"""
            TORCH_INTERNAL_ASSERT(shapes.size() == {returns_length});"""

        # Calculating which dimensions are symbolic
        func_schema_str = "aten::" + str(func.func)
        shape_str += f"""
            if(torch::lazy::symbolicShapeEnabled()){{
                std::vector<torch::jit::IValue> inputs = {{ {", ".join(str(a.name) for a in all_args)} }};
                const char* schema_str = "{func_schema_str}";
                applySymbolicShapesOnLT(schema_str, inputs, shapes);
            }}
        """
        return shape_str
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk defines `applySymbolicShapesOnLT`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段定义了 `applySymbolicShapesOnLT`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 577-602
```python
    def build_ir_node(self, func: NativeFunction, schema: LazyIrSchema) -> str:
        node_ctor_input_str = node_ctor_inputs(schema)
        return f"""torch::lazy::NodePtr node = torch::lazy::ReuseNode<{schema.node_name}>({node_ctor_input_str});
        if (!node) {{
            {self.shape_inference(func, schema)}
            node = torch::lazy::MakeNode<{schema.node_name}>({node_ctor_input_str}, std::move(shapes));
            CacheNode(node);
        }}
        """

    def create_lazy_tensor(self, first_tensor_name: str | None = None) -> str:
        # xla uses an instance method for tensor creation, for the time being
        if self.create_from_first_tensor:
            # TODO(whc) remove this if XLA switches to using static method for creation
            if first_tensor_name is None:
                raise AssertionError("Requires first tensor to create lazy tensor")
            return f"{first_tensor_name}.{self.create_tensor}"
        return f"{self.backend_namespace}::{self.create_tensor}"

    def return_aten_tensor(self, func: NativeFunction, schema: LazyIrSchema) -> str:
        returns_length = len(schema.returns)
        value_args = schema.filtered_args(values=True, scalars=False)
        value_types_names = [f"{a.name}" for a in value_args if not a.is_wrapped_scalar]
        first_tensor_name = value_types_names[0] if len(value_types_names) > 0 else None
        bridge_str = f"""auto result = {self.create_aten_from_ltc_tensor}(
                {self.create_lazy_tensor(first_tensor_name)}(std::move(node), *common_device));"""
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk defines `CacheNode`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段定义了 `CacheNode`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 604-628
```python
        if returns_length > 1:
            if len(value_types_names) == 0:
                raise AssertionError(
                    "Code below assumes there is at least one tensor arg"
                )
            bridge_str = f"""std::vector<{self.lazy_tensor_ptr}> lazy_tensors;
        for (int i = 0; i < {returns_length}; i++) {{
            lazy_tensors.push_back({self.create_lazy_tensor(first_tensor_name)}({getValueT()}(node, i), *common_device));
        }}
        auto result = {self.tuple_aten_from_ltc_tensors}<{returns_length}>(lazy_tensors);"""

        if schema.name.name.inplace or func.func.is_out_fn():
            if returns_length != 1:
                raise AssertionError(
                    "We assumed there was no such case where an op is an in-place variant "
                    f"and has tuple outputs, but got tuple of len {returns_length}."
                )
            bridge_str = f"""lazy_{first_tensor_name}->SetInPlaceIrValue(node);
        auto& result = {first_tensor_name};"""

        bridge_str += """
        return result;"""
        return bridge_str

    @method_with_native_function
```
- **EN**: Decorators such as @method_with_native_function modify how the following Python definitions are exported, wrapped, or dataclass-managed. Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. This chunk continues `CacheNode` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Branch dispatch chooses specialized behavior for different cases, node kinds, or runtime situations. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @method_with_native_function 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 这一段延续了 `CacheNode`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 分支分发会为不同情况、节点类型或运行时场景选择专门行为。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 629-652
```python
    def __call__(self, func: NativeFunction) -> list[str]:
        sig = kernel_signature(func, self.backend_index)
        metadata = self.backend_index.get_kernel(func)
        if metadata is None:
            raise AssertionError(f"No kernel metadata found for {func.func.name}")
        schema = LazyIrSchema(func.func, symint=metadata.supports_symint())
        return [
            f"""\
    {sig.decl(name=f"{self.class_method_name}::{metadata.kernel}")} {{
        {self.force_eager_fallback(func, schema, metadata, sig)}
        {self.metrics(func, schema)}
        {self.get_device(func, schema)}
        {self.lazy_tensor_decls(func, schema)}
        {self.build_ir_node(func, schema)}
        {self.return_aten_tensor(func, schema)}
    }}\n
    """
        ]


class ComputeShapeSignature:
    """
    Here we use the base name as the suffix of the signature to avoid generating for in-place variants.
    """
```
- **EN**: Template placeholders indicate that torchgen or another emitter will substitute concrete operator-specific values later. It introduces or extends ComputeShapeSignature, which hold the primary data model or public surface for this slice of the file. This chunk defines `__call__`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 模板占位符表明 torchgen 或其他生成器会在稍后填入具体的算子相关值。 它引入或扩展了 ComputeShapeSignature，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `__call__`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 654-679
```python
    def __init__(self, kernel_name: str, f: NativeFunction, *, symint: bool) -> None:
        self.__schema = LazyIrSchema(f.func, symint=symint)
        self.__dispatch_args = ", ".join(
            [a.decl() for a in dispatcher.arguments(f.func, symint=symint)]
        )
        self.__call_args = ", ".join(
            [f"{arg.name}" for arg in self.__schema.filtered_args(generator=True)]
        )
        self.__kernel_name = kernel_name

    def __decl_suffix(self) -> str:
        return f"{self.__kernel_name}({self.__dispatch_args})"

    def __call_suffix(self) -> str:
        return f"{self.__kernel_name}({self.__call_args})"

    @property
    def shape_decl(self) -> str:
        return f"TORCH_API std::vector<torch::lazy::Shape> compute_shape_{self.__decl_suffix()}"

    @property
    def shape_call(self) -> str:
        return f"torch::lazy::compute_shape_{self.__call_suffix()}"


@dataclass(frozen=True)
```
- **EN**: Decorators such as @property, @property, and 1 more modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `shape_call`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @property、@property 等共 3 项 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `shape_call`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 680-699
```python
class GenLazyShapeInferenceDefinition:
    backend_index: BackendIndex
    tensor_class: str

    @method_with_native_function
    def __call__(self, f: NativeFunction) -> list[str]:
        metadata = self.backend_index.get_kernel(f)
        if metadata is None:
            raise AssertionError(f"No kernel metadata found for {f.func.name}")

        # See Note [Generated LTC Shape Functions]
        is_view_copy_op = "view_copy" in f.tags
        is_structured = f.structured or f.structured_delegate is not None
        if is_structured or is_view_copy_op:
            return []
        else:
            shape_sig = ComputeShapeSignature(
                metadata.kernel, f, symint=metadata.supports_symint()
            )
            return ["\n".join([f"{shape_sig.shape_decl};"])]
```
- **EN**: Decorators such as @method_with_native_function modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends GenLazyShapeInferenceDefinition, which hold the primary data model or public surface for this slice of the file. This chunk defines `__call__`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @method_with_native_function 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 GenLazyShapeInferenceDefinition，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `__call__`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 702-718
```python
def generate_non_native_lazy_ir_nodes(
    non_native: list[dict[str, Any]], gen_lazy_ir: GenLazyIR
) -> list[str]:
    """Generate the non-native lazy IR node classes"""
    nodes = []
    for op in non_native:
        # Set default properties for Non-Native IRs
        properties = LazyIrProperties("ShapeCache", "CanBeReused", "LowerDeclOnly")
        for p in op.get("properties", []):
            setattr(properties, p, True)

        # non-native is assumed to want symint bindings if you wrote symint
        schema = LazyIrSchema(FunctionSchema.parse(op["func"]), properties, symint=True)
        schema.opkind = op.get("opkind")
        nodes.append(gen_lazy_ir.gen(schema)[0])

    return nodes
```
- **EN**: This chunk defines `generate_non_native_lazy_ir_nodes`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `generate_non_native_lazy_ir_nodes`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Torch operator code generation**
  - EN: Builds derived code artifacts from declarative operator metadata.
  - CN: 从声明式算子元数据构建派生代码产物。
- **Dispatch-key plumbing**
  - EN: Models dispatch keys, backend selection, or registration routing.
  - CN: 建模 dispatch key、后端选择或注册路由。
- **Lazy tensor support**
  - EN: Builds lazy IR or lowering support for deferred execution backends.
  - CN: 为延迟执行后端构建 lazy IR 或降级支持。
- **node_ctor_arg_rvalue_string**
  - EN: `node_ctor_arg_rvalue_string` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `node_ctor_arg_rvalue_string` 是本文件声明、导出或驱动的显著符号之一。
- **node_ctor_inputs**
  - EN: `node_ctor_inputs` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `node_ctor_inputs` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torchgen.api.dispatcher`, `torchgen.api.lazy`, `torchgen.api.translate`, `torchgen.api.types`, `torchgen.context`, `torchgen.dest.lazy_ts_lowering`, `torchgen.model`
- **Standard library / 标准库**: `__future__`, `itertools`, `abc`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `node_ctor_arg_rvalue_string`, `node_ctor_inputs`, `gen_fallback_code`, `aten_symbol`, `convert_to_meta_tensors`, `GenLazyIR`, `__call__`, `lowering_function`, `create_function`, `can_be_reused_function`
