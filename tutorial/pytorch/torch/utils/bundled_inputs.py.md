# bundled_inputs.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/bundled_inputs.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `bundled_inputs.py`. The implementation is performance-facing and focuses on measurement fidelity or result reporting. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `bundled_inputs.py` 展开。 该实现面向性能分析，重点在于测量可信度或结果报告。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
```python
#!/usr/bin/env python3
# mypy: allow-untyped-defs
from typing import Any, TypeVar, NamedTuple
from collections.abc import Callable, Sequence
import textwrap
import torch
from torch._C import TupleType, ListType
from torch.jit._recursive import wrap_cpp_module


T = TypeVar("T")

MAX_RAW_TENSOR_SIZE = 16

class InflatableArg(NamedTuple):
    """Helper type for bundled inputs.

    'value' is the compressed/deflated input that is stored in the model. Value
    must be of the same type as the argument to the function that it is a deflated
    input for.
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch._C:TupleType, torch._C:ListType, torch.jit._recursive:wrap_cpp_module; standard-library helpers such as typing:Any, typing:TypeVar, typing:NamedTuple, collections.abc:Callable. It introduces or extends class-level abstractions such as `InflatableArg`, which organize state and behavior for this subsystem. Named constants such as `T`, `MAX_RAW_TENSOR_SIZE` centralize shared configuration or sentinel values. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch._C:TupleType, torch._C:ListType, torch.jit._recursive:wrap_cpp_module；标准库辅助模块，如 typing:Any, typing:TypeVar, typing:NamedTuple, collections.abc:Callable。 它引入或扩展了 `InflatableArg` 等类级抽象，用于组织该子系统的状态与行为。 `T, MAX_RAW_TENSOR_SIZE` 等具名常量把共享配置或哨兵值集中定义在一起。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 22-38 / 第 22-38 行
```python
    'fmt' is a formattable code string that is executed to inflate the compressed data into
    the appropriate input. It can use 'value' as an input to the format str. It must result
    in a value of the same type as 'value'.

    'fmt_fn' is a formattable function code string that is executed to inflate the compressed
    data into the appropriate input. It must result in a value of the same type as 'value'.
    The function name should be the formattable part of the string.

    Note: Only top level InflatableArgs can be inflated. i.e. you cannot place
    an inflatable arg inside of some other structure. You should instead create
    an inflatable arg such that the fmt code string returns the full structure
    of your input.
    """

    value: Any
    fmt: str = "{}"
    fmt_fn: str = ""
```
- **EN**: It introduces or extends class-level abstractions such as `InflatableArg`, which organize state and behavior for this subsystem.
- **CN**: 它引入或扩展了 `InflatableArg` 等类级抽象，用于组织该子系统的状态与行为。

### Lines 41-60 / 第 41-60 行
```python
def bundle_inputs(
        model: torch.jit.ScriptModule,
        inputs: Sequence[tuple[Any, ...]] | dict[Callable, Sequence[tuple[Any, ...]] | None] | None,
        info: list[str] | dict[Callable, list[str]] | None = None,
        *,
        _receive_inflate_expr: list[str] | None = None,
) -> torch.jit.ScriptModule:
    """Create and return a copy of the specified model with inputs attached.

    The original model is not mutated or changed in any way.

    Models with bundled inputs can be invoked in a uniform manner by
    benchmarking and code coverage tools.

    If inputs is passed in as a list then the inputs will be bundled for 'forward'.
    If inputs is instead passed in as a map then all the methods specified in the map
    will have their corresponding inputs bundled. Info should match watchever type is
    chosen for the inputs.

    The returned model will support the following methods:
```
- **EN**: Key callable entry points in this range include `bundle_inputs`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `bundle_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 62-83 / 第 62-83 行
```python
        `get_all_bundled_inputs_for_<function_name>() -> List[Tuple[Any, ...]]`
            Returns a list of tuples suitable for passing to the model like
            `for inp in model.get_all_bundled_inputs_for_foo(): model.foo(*inp)`

        `get_bundled_inputs_functions_and_info() -> Dict[str, Dict[str: List[str]]]`
            Returns a dictionary mapping function names to a metadata dictionary.
            This nested dictionary maps preset strings like:
                'get_inputs_function_name' -> the name of a function attribute in this model that can be
                    run to get back a list of inputs corresponding to that function.
                'info' -> the user provided extra information about the bundled inputs

    If forward has bundled inputs then these following functions will also be defined on the returned module:

        `get_all_bundled_inputs() -> List[Tuple[Any, ...]]`
            Returns a list of tuples suitable for passing to the model like
            `for inp in model.get_all_bundled_inputs(): model(*inp)`

        `get_num_bundled_inputs() -> int`
            Equivalent to `len(model.get_all_bundled_inputs())`,
            but slightly easier to call from C++.

    Inputs can be specified in one of two ways:
```
- **EN**: Key callable entry points in this range include `bundle_inputs`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 这一段的重要可调用入口包括 `bundle_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 85-101 / 第 85-101 行
```python
      - The model can define `_generate_bundled_inputs_for_<function_name>`.
        If the user chooses this method inputs[<function>] should map to None

      - The `inputs` argument to this function can be a dictionary mapping functions to a
        list of inputs, of the same form that will be returned by get_all_bundled_inputs_for_<function_name>.
        Alternatively if only bundling inputs for forward the map can be omitted and a singular list of inputs
        can be provided instead.

        The type of the inputs is List[Tuple[Any, ...]]. The outer list corresponds with a
        list of inputs, the inner tuple is the list of args that together make up one input.
        For inputs of functions that take one arg, this will be a tuple of length one. The Any, ...
        is the actual data that makes up the args, e.g. a tensor.

    Info is an optional parameter that maps functions to a list of strings providing extra information about that
    function's bundled inputs. Alternatively if only bundling inputs for forward the map can be omitted and
    a singular list of information can be provided instead. This could be descriptions, expected outputs, etc.
        - Ex: info={model.forward : ['man eating icecream', 'an airplane', 'a dog']}
```
- **EN**: Key callable entry points in this range include `bundle_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 这一段的重要可调用入口包括 `bundle_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 103-126 / 第 103-126 行
```python
    This function will attempt to optimize arguments so that (e.g.)
    arguments like `torch.zeros(1000)` will be represented compactly.
    Only top-level arguments will be optimized.
    Tensors in lists or tuples will not.
    """
    if not isinstance(model, torch.jit.ScriptModule):
        raise Exception("Only ScriptModule is supported.")  # noqa: TRY002

    ignored_methods, ignored_attrs = _get_bundled_inputs_attributes_and_methods(model)
    clone = torch._C._hack_do_not_use_clone_module_with_class(  # type: ignore[attr-defined]
        model._c,
        ignored_methods,
        ignored_attrs,
    )

    # The above cloning function returns a torch._C.scriptmodule and we need a torch.jit.scriptmodule.
    # Fortunately there is a function in _recursive that does exactly that conversion.
    cloned_module = wrap_cpp_module(clone)
    if isinstance(inputs, dict):
        if not isinstance(info, dict) and info is not None:
            raise AssertionError("If inputs is a dict, info must be a dict or None")
        augment_many_model_functions_with_bundled_inputs(cloned_module, inputs, _receive_inflate_expr, info)
    else:
        if not isinstance(info, list) and info is not None:
```
- **EN**: Key callable entry points in this range include `bundle_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `bundle_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 127-147 / 第 127-147 行
```python
            raise AssertionError("If inputs is a list, info must be a list or None")
        augment_model_with_bundled_inputs(cloned_module, inputs, _receive_inflate_expr, info)
    return cloned_module

def augment_model_with_bundled_inputs(
        model: torch.jit.ScriptModule,
        inputs: Sequence[tuple[Any, ...]] | None = None,
        _receive_inflate_expr: list[str] | None = None,  # For debugging.
        info: list[str] | None = None,  # Optional argument to provide info about forward or its inputs
        skip_size_check=False,
) -> None:
    """Add bundled sample inputs to a model for the forward function.

    Models with bundled inputs can be invoked in a uniform manner by
    benchmarking and code coverage tools.

    Augmented models will support the following methods:

        `get_all_bundled_inputs() -> List[Tuple[Any, ...]]`
            Returns a list of tuples suitable for passing to the model like
            `for inp in model.get_all_bundled_inputs(): model(*inp)`
```
- **EN**: Key callable entry points in this range include `bundle_inputs`, `augment_model_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `bundle_inputs`, `augment_model_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 149-171 / 第 149-171 行
```python
        `get_num_bundled_inputs() -> int`
            Equivalent to `len(model.get_all_bundled_inputs())`,
            but slightly easier to call from C++.

        `get_bundled_inputs_functions_and_info() -> Dict[str, Dict[str: List[str]]]`
            Returns a dictionary mapping function names to a metadata dictionary.
            This nested dictionary maps preset strings like:
                'get_inputs_function_name' -> the name of a function attribute in this model that can be
                    run to get back a list of inputs corresponding to that function.
                'info' -> the user provided extra information about the bundled inputs

    Inputs can be specified in one of two ways:

      - The model can define `_generate_bundled_inputs_for_forward`.
        If the user chooses this method inputs should be None

      - `inputs` is a list of inputs of form List[Tuple[Any, ...]]. A list of tuples where the elements
        of each tuple are the args that make up one input.
    """
    if not isinstance(model, torch.jit.ScriptModule):
        raise Exception("Only ScriptModule is supported.")  # noqa: TRY002

    forward: Callable = model.forward
```
- **EN**: Key callable entry points in this range include `augment_model_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `augment_model_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 173-195 / 第 173-195 行
```python
    # Sometimes forward won't have a name attached so just in case
    if not hasattr(forward, "__name__"):
        forward.__name__ = 'forward'
    augment_many_model_functions_with_bundled_inputs(
        model,
        inputs={forward : inputs},
        _receive_inflate_expr=_receive_inflate_expr,
        info={forward : info} if info else None,
        skip_size_check=skip_size_check,
    )


def augment_many_model_functions_with_bundled_inputs(
        model: torch.jit.ScriptModule,
        inputs: dict[Callable, Sequence[tuple[Any, ...]] | None],
        _receive_inflate_expr: list[str] | None = None,  # For debugging.
        info: dict[Callable, list[str]] | None = None,  # Optional argument to provide info about the function or its inputs
        skip_size_check=False,
) -> None:
    """Add bundled sample inputs to a model for an arbitrary list of public functions.

    Models with bundled inputs can be invoked in a uniform manner by
    benchmarking and code coverage tools.
```
- **EN**: Key callable entry points in this range include `augment_model_with_bundled_inputs`, `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `augment_model_with_bundled_inputs`, `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 197-218 / 第 197-218 行
```python
    Augmented models will support the following methods:

        `get_all_bundled_inputs_for_<function_name>() -> List[Tuple[Any, ...]]`
            Returns a list of tuples suitable for passing to the model like
            `for inp in model.get_all_bundled_inputs_for_foo(): model.foo(*inp)`

        `get_bundled_inputs_functions_and_info() -> Dict[str, Dict[str: List[str]]]`
            Returns a dictionary mapping function names to a metadata dictionary.
            This nested dictionary maps preset strings like:
                'get_inputs_function_name' -> the name of a function attribute in this model that can be
                    run to get back a list of inputs corresponding to that function.
                'info' -> the user provided extra information about the bundled inputs

    If forward has bundled inputs then these following functions are also defined:

        `get_all_bundled_inputs() -> List[Tuple[Any, ...]]`
            Returns a list of tuples suitable for passing to the model like
            `for inp in model.get_all_bundled_inputs(): model(*inp)`

        `get_num_bundled_inputs() -> int`
            Equivalent to `len(model.get_all_bundled_inputs())`,
            but slightly easier to call from C++.
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 220-242 / 第 220-242 行
```python
    Inputs can be specified in one of two ways:

      - The model can define `_generate_bundled_inputs_for_<function_name>`.
        If the user chooses this method inputs[<function>] should map to None

      - The `inputs` argument to this function can be a dictionary mapping functions to a
        list of inputs, of the same form that will be returned by get_all_bundled_inputs_for_<function_name>.
        The type of the inputs is List[Tuple[Any, ...]]. The outer list corresponds with a
        list of inputs, the inner tuple is the list of args that together make up one input.
        For inputs of functions that take one arg, this will be a tuple of length one. The Any, ...
        is the actual data that makes up the args, e.g. a tensor.

    Info is an optional parameter that maps functions to a list of strings providing extra information about that
    function's bundled inputs. This could be descriptions, expected outputs, etc.
        - Ex: info={model.forward : ['man eating icecream', 'an airplane', 'a dog']}

    This function will attempt to optimize arguments so that (e.g.)
    arguments like `torch.zeros(1000)` will be represented compactly.
    Only top-level arguments will be optimized.
    Tensors in lists or tuples will not.
    """
    if not isinstance(model, torch.jit.ScriptModule):
        raise Exception("Only ScriptModule is supported.")  # noqa: TRY002
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 244-265 / 第 244-265 行
```python
    if not inputs:
        raise Exception("Please provide inputs for at least 1 function")  # noqa: TRY002

    if hasattr(model, "get_all_bundled_inputs") or hasattr(model, "get_bundled_inputs_functions_and_info"):
        raise Exception(  # noqa: TRY002
            "Models can only be augmented with bundled inputs once. "
            "This Model seems to have already been augmented with "
            "bundled inputs. Please start afresh with one that "
            "doesn't have bundled inputs.",
        )

    get_bundled_inputs_functions_and_info_template = ""

    for function, input_list in inputs.items():
        if hasattr(function, "__name__"):
            function_name = function.__name__
        else:
            if hasattr(function, "name"):
                function_name = function.name  # type: ignore[attr-defined]
            else:
                raise Exception(  # noqa: TRY002
                    'At least one of your functions has no attribute name please ensure all have one. m.foo.name = "foo"')
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 268-291 / 第 268-291 行
```python
        if input_list is not None and not isinstance(input_list, Sequence):
            raise TypeError(f"Error inputs for function {function_name} is not a Sequence")

        function_arg_types = [arg.type for arg in function.schema.arguments[1:]]  # type: ignore[attr-defined]
        deflated_inputs_type: ListType = ListType(TupleType(function_arg_types))
        model._c._register_attribute(f"_bundled_inputs_deflated_{function_name}", deflated_inputs_type, [])

        if hasattr(model, "_generate_bundled_inputs_for_" + function_name):
            if input_list is not None:
                raise Exception(  # noqa: TRY002
                    f"inputs[{function_name}] is not None, but _generate_bundled_inputs_for_{function_name} is already defined"
                )
            # Model author already defined _generate_bundled_inputs_for_<function_name>.
        elif input_list is None or len(input_list) == 0:
            raise Exception(  # noqa: TRY002
                f"inputs for {function_name} must be specified if "
                f"_generate_bundled_inputs_for_{function_name} is not already defined"
            )
        else:
            # Iterate over the inputs and args in each input.
            # Accumulate `deflated_inputs` as (possibly) compressed values
            # and `parts` to be joined into the expression that unpacks them.
            deflated_inputs = []
            parts = []
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 292-314 / 第 292-314 行
```python
            for inp_idx, args in enumerate(input_list):
                if not isinstance(args, tuple) and not isinstance(args, list):  # type: ignore[arg-type]
                    raise TypeError(
                        f"Error bundled input for function {function_name} idx: {inp_idx} is not a Tuple or a List"
                    )
                deflated_args = []
                parts.append("(")
                for arg_idx, arg in enumerate(args):
                    inflate_helper_fn_name = _get_inflate_helper_fn_name(arg_idx, inp_idx, function_name)
                    deflated, inflater, helper_definition = _inflate_expr(
                        arg,
                        f"deflated[{inp_idx}][{arg_idx}]",
                        inflate_helper_fn_name,
                        skip_size_check=skip_size_check,
                    )
                    deflated_args.append(deflated)
                    parts.append(f"    {inflater},")
                    if helper_definition:
                        model.define(textwrap.dedent(helper_definition))
                deflated_inputs.append(tuple(deflated_args))
                parts.append("),")
            parts.append("")
            expr = "\n".join(parts)
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 316-335 / 第 316-335 行
```python
            # Back-channel return this expr for debugging.
            if _receive_inflate_expr is not None:
                _receive_inflate_expr.append(expr)
            setattr(model, f"_bundled_inputs_deflated_{function_name}", deflated_inputs)
            definition = textwrap.dedent("""
                def _generate_bundled_inputs_for_{name}(self):
                    deflated = self._bundled_inputs_deflated_{name}
                    return [
                {expr}
                    ]
                """).format(expr=expr, name=function_name)
            model.define(definition)

        # Define get_all_bundled_inputs_for_<function_name> that caches the generated inputs.
        model.define(textwrap.dedent("""
            def get_all_bundled_inputs_for_{name}(self):
                all_inputs = self._generate_bundled_inputs_for_{name}()
                assert all_inputs is not None
                return all_inputs
            """).format(name=function_name))
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 337-357 / 第 337-357 行
```python
        # Add to the high level helper methods
        inputs_info = repr(info[function]) if info and function in info else '[]'
        get_bundled_inputs_functions_and_info_template += f"""
            temp_dict : Dict[str,List[str]] = {{}}
            info: List[str] = {inputs_info}

            temp_dict['info'] = info
            temp_dict['get_inputs_function_name'] = ['get_all_bundled_inputs_for_{function_name}']
            all_inputs['{function_name}'] = temp_dict
            """

        # To ensure backwards compatibility and a streamlined api for forward these wrappers are provided
        if function_name == 'forward':
            model.define(textwrap.dedent("""
                def get_all_bundled_inputs(self):
                    return self.get_all_bundled_inputs_for_forward()
                """))
            model.define(textwrap.dedent("""
                def get_num_bundled_inputs(self):
                    return len(self.get_all_bundled_inputs_for_forward())
                """))
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 359-382 / 第 359-382 行
```python
    # Define some high level helper methods that act on all bundled inputs
    model.define(textwrap.dedent(f"""
        def get_bundled_inputs_functions_and_info(self):
            all_inputs : Dict[str, Dict[str,List[str]]] = {{}}
            {get_bundled_inputs_functions_and_info_template}
            return all_inputs
        """))

def _inflate_expr(
    arg: T, ref: str, inflate_helper_fn_name: str, skip_size_check: bool = False
) -> tuple[T | torch.Tensor, str, str | None]:
    # Allow custom inflation expressions any object.
    # For example, calling custom image-decoding ops.
    # Or just use "{}" as the format string to ignore size limits.
    if isinstance(arg, InflatableArg):
        if arg.fmt_fn:
            if arg.fmt not in ["{}", ""]:
                raise Exception(  # noqa: TRY002
                    f"Bundled input argument at position '{ref}' has "
                    f"both arg.fmt_fn => \n{arg.fmt_fn} "
                    f"\n and arg.fmt  => {arg.fmt}. "
                    "Please choose `arg.fmt` if the deflater is straightforward or "
                    "`arg.fmt_fn` if you need a function."
                )
```
- **EN**: Key callable entry points in this range include `augment_many_model_functions_with_bundled_inputs`, `_inflate_expr`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `augment_many_model_functions_with_bundled_inputs`, `_inflate_expr`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 384-407 / 第 384-407 行
```python
            helper_definition = arg.fmt_fn.format(inflate_helper_fn_name)
            expr = f"self.{inflate_helper_fn_name}({ref})"

            return arg.value, expr, helper_definition
        else:
            return arg.value, arg.fmt.format(ref), None

    if isinstance(arg, torch.Tensor):
        # Small-storage tensors can just be saved directly.
        if arg._typed_storage().size() <= MAX_RAW_TENSOR_SIZE or skip_size_check:
            return arg, ref, None
        # Small contiguous tensors can be cloned to have small storage.
        # TODO: Should we do this even for non-contiguous tensors?
        if arg.is_contiguous() and arg.numel() <= MAX_RAW_TENSOR_SIZE:
            return arg.clone(), ref, None
        # Example inputs commonly come from torch.zeros, torch.ones, or torch.full.
        # These can be represented compactly.
        for fmt in [torch.contiguous_format, torch.channels_last]:
            if arg.is_contiguous(memory_format=fmt) and (arg == arg.flatten()[0]).all().item():
                return (arg.flatten()[0].clone().expand(*arg.size()),
                        f"{ref}.contiguous(memory_format={fmt})", None)
        # Prevent big tensors from being bundled by default.
        # TODO: Provide more useful diagnostics.
        raise Exception(  # noqa: TRY002
```
- **EN**: Key callable entry points in this range include `_inflate_expr`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_inflate_expr`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 408-423 / 第 408-423 行
```python
            f"Bundled input argument at position '{ref}' is "
            f"a tensor with storage size {arg._typed_storage().size()}. "
            f"You probably don't want to bundle this as an input. "
        )
    else:
        return arg, ref, None

def _get_bundled_inputs_attributes_and_methods(script_module: torch.jit.ScriptModule) -> tuple[list[str], list[str]]:
    methods: list[str] = []
    attributes: list[str] = []

    # Has bundled inputs for forward
    if hasattr(script_module, 'get_all_bundled_inputs'):
        methods.append('get_all_bundled_inputs')
        methods.append('get_num_bundled_inputs')
        methods.append('run_on_bundled_input')
```
- **EN**: Key callable entry points in this range include `_inflate_expr`, `_get_bundled_inputs_attributes_and_methods`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_inflate_expr`, `_get_bundled_inputs_attributes_and_methods`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 425-448 / 第 425-448 行
```python
    if hasattr(script_module, 'get_bundled_inputs_functions_and_info'):
        methods.append('get_bundled_inputs_functions_and_info')
        all_info = script_module.get_bundled_inputs_functions_and_info()
        for function_name in all_info:
            methods.append("get_all_bundled_inputs_for_" + function_name)
            methods.append("_generate_bundled_inputs_for_" + function_name)
            attributes.append("_bundled_inputs_deflated_" + function_name)

            bundled_inputs_fn = getattr(
                script_module,
                f"get_all_bundled_inputs_for_{function_name}"
            )
            num_bundled_inputs: int = len(bundled_inputs_fn())

            # Check inflate helper functions for each function, argument and bundled input
            func = getattr(script_module, function_name)
            for arg_idx in range(len(func.schema.arguments) - 1):
                for input_idx in range(num_bundled_inputs):
                    helper_fn_name = _get_inflate_helper_fn_name(
                        arg_idx=arg_idx,
                        input_idx=input_idx,
                        function_name=function_name
                    )
                    # if the arg has an InflatableArg with fmt_fn, add the helper function name
```
- **EN**: Key callable entry points in this range include `_get_bundled_inputs_attributes_and_methods`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 这一段的重要可调用入口包括 `_get_bundled_inputs_attributes_and_methods`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 449-472 / 第 449-472 行
```python
                    if hasattr(script_module, helper_fn_name):
                        methods.append(helper_fn_name)

    return (methods, attributes)


def _get_inflate_helper_fn_name(
    arg_idx: int,
    input_idx: int,
    function_name: str,
) -> str:
    return f"_inflate_helper_for_{function_name}_input_{input_idx}_arg_{arg_idx}"



def bundle_randn(*size, dtype=None):
    """Generate a tensor that will be inflated with torch.randn."""
    stub = torch.zeros(1, dtype=dtype).expand(*size)
    return InflatableArg(value=stub, fmt="torch.randn_like({})")


def bundle_large_tensor(t):
    """Wrap a tensor to allow bundling regardless of size."""
    return InflatableArg(value=t, fmt="{}")
```
- **EN**: Key callable entry points in this range include `_get_bundled_inputs_attributes_and_methods`, `_get_inflate_helper_fn_name`, `bundle_randn`, `bundle_large_tensor`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_get_bundled_inputs_attributes_and_methods`, `_get_inflate_helper_fn_name`, `bundle_randn`, `bundle_large_tensor`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **InflatableArg**
  - EN: `InflatableArg` is one of the main classes that structures the file's behavior.
  - CN: `InflatableArg` 是组织该文件行为的核心类之一。
- **bundle_inputs**
  - EN: `bundle_inputs` is a representative function that exposes or coordinates an important action in this module.
  - CN: `bundle_inputs` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
- **augment_model_with_bundled_inputs**
  - EN: `augment_model_with_bundled_inputs` is a representative function that exposes or coordinates an important action in this module.
  - CN: `augment_model_with_bundled_inputs` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch._C:TupleType`, `torch._C:ListType`, `torch.jit._recursive:wrap_cpp_module`
- **Python standard library / Python 标准库**: `typing:Any`, `typing:TypeVar`, `typing:NamedTuple`, `collections.abc:Callable`, `collections.abc:Sequence`, `textwrap`
- **Primary symbols / 核心符号**: `InflatableArg`, `bundle_inputs`, `augment_model_with_bundled_inputs`, `augment_many_model_functions_with_bundled_inputs`, `_inflate_expr`, `_get_bundled_inputs_attributes_and_methods`, `_get_inflate_helper_fn_name`, `bundle_randn`, `bundle_large_tensor`
