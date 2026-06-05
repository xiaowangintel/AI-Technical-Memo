# prepare.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/backends/_nnapi/prepare.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Exposes backend-specific switches, capability probes, and preprocessing helpers for optional runtimes.
- **Purpose (CN)**: 暴露后端相关的开关、能力探测以及面向可选运行时的预处理辅助逻辑。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
````python
# mypy: allow-untyped-decorators
# mypy: allow-untyped-defs

from typing import Optional

import torch
from torch.backends._nnapi.serializer import _NnapiSerializer


ANEURALNETWORKS_PREFER_LOW_POWER = 0
ANEURALNETWORKS_PREFER_FAST_SINGLE_ANSWER = 1
ANEURALNETWORKS_PREFER_SUSTAINED_SPEED = 2


class NnapiModule(torch.nn.Module):
    """Torch Module that wraps an NNAPI Compilation.
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.backends._nnapi.serializer; standard-library helpers such as typing. It introduces or extends `NnapiModule`, which hold the main object-oriented state for this portion of the file.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.backends._nnapi.serializer；标准库辅助模块，如 typing。 它引入或扩展了 `NnapiModule`，这些类承载了本段涉及的主要面向对象状态。

### Lines 18-37 / 第 18-37 行
````python
    This module handles preparing the weights, initializing the
    NNAPI TorchBind object, and adjusting the memory formats
    of all inputs and outputs.
    """

    # _nnapi.Compilation is defined
    comp: Optional[torch.classes._nnapi.Compilation]  # type: ignore[name-defined]  # noqa: UP045
    weights: list[torch.Tensor]
    out_templates: list[torch.Tensor]

    def __init__(
        self,
        shape_compute_module: torch.nn.Module,
        ser_model: torch.Tensor,
        weights: list[torch.Tensor],
        inp_mem_fmts: list[int],
        out_mem_fmts: list[int],
        compilation_preference: int,
        relax_f32_to_f16: bool,
    ):
````
- **EN**: This chunk defines `__init__`, which initializes object state and binds construction-time configuration.
- **CN**: 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。

### Lines 38-57 / 第 38-57 行
````python
        super().__init__()
        self.shape_compute_module = shape_compute_module
        self.ser_model = ser_model
        self.weights = weights
        self.inp_mem_fmts = inp_mem_fmts
        self.out_mem_fmts = out_mem_fmts
        self.out_templates = []
        self.comp = None
        self.compilation_preference = compilation_preference
        self.relax_f32_to_f16 = relax_f32_to_f16

    @torch.jit.export
    def init(self, args: list[torch.Tensor]):
        if self.comp is not None:
            raise AssertionError("comp must be None before initialization")
        self.out_templates = self.shape_compute_module.prepare(self.ser_model, args)  # type: ignore[operator]
        self.weights = [w.contiguous() for w in self.weights]
        comp = torch.classes._nnapi.Compilation()
        comp.init2(
            self.ser_model,
````
- **EN**: This chunk defines `init`, which implements a focused helper used by the surrounding module. Decorators such as `torch.jit.export` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `init`，其作用是实现周边模块使用的关键辅助逻辑。 像 `torch.jit.export` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 58-71 / 第 58-71 行
````python
            self.weights,
            self.compilation_preference,
            self.relax_f32_to_f16,
        )

        self.comp = comp

    def forward(self, args: list[torch.Tensor]) -> list[torch.Tensor]:
        if self.comp is None:
            self.init(args)
        comp = self.comp
        if comp is None:
            raise AssertionError("comp must not be None")
        outs = [torch.empty_like(out) for out in self.out_templates]
````
- **EN**: This chunk defines `forward`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `forward`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 73-92 / 第 73-92 行
````python
        if len(args) != len(self.inp_mem_fmts):
            raise AssertionError(
                f"args length {len(args)} != inp_mem_fmts length {len(self.inp_mem_fmts)}"
            )
        fixed_args = []
        for idx in range(len(args)):
            fmt = self.inp_mem_fmts[idx]
            # These constants match the values in DimOrder in serializer.py
            # TODO: See if it's possible to use those directly.
            if fmt == 0:
                fixed_args.append(args[idx].contiguous())
            elif fmt == 1:
                fixed_args.append(args[idx].permute(0, 2, 3, 1).contiguous())
            else:
                raise ValueError("Invalid mem_fmt")
        comp.run(fixed_args, outs)
        if len(outs) != len(self.out_mem_fmts):
            raise AssertionError(
                f"outs length {len(outs)} != out_mem_fmts length {len(self.out_mem_fmts)}"
            )
````
- **EN**: This chunk continues `forward` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `forward`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 93-112 / 第 93-112 行
````python
        for idx in range(len(self.out_templates)):
            fmt = self.out_mem_fmts[idx]
            # These constants match the values in DimOrder in serializer.py
            # TODO: See if it's possible to use those directly.
            if fmt in (0, 2):
                pass
            elif fmt == 1:
                outs[idx] = outs[idx].permute(0, 3, 1, 2)
            else:
                raise ValueError("Invalid mem_fmt")
        return outs


def convert_model_to_nnapi(
    model,
    inputs,
    serializer=None,
    return_shapes=None,
    use_int16_for_qint16=False,
    compilation_preference=ANEURALNETWORKS_PREFER_SUSTAINED_SPEED,
````
- **EN**: This chunk defines `convert_model_to_nnapi`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `convert_model_to_nnapi`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 113-132 / 第 113-132 行
````python
    relax_f32_to_f16=False,
):
    (
        shape_compute_module,
        ser_model_tensor,
        used_weights,
        inp_mem_fmts,
        out_mem_fmts,
        retval_count,
    ) = process_for_nnapi(
        model, inputs, serializer, return_shapes, use_int16_for_qint16
    )

    nnapi_model = NnapiModule(
        shape_compute_module,
        ser_model_tensor,
        used_weights,
        inp_mem_fmts,
        out_mem_fmts,
        compilation_preference,
````
- **EN**: This chunk continues `convert_model_to_nnapi` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `convert_model_to_nnapi`，进一步展开其内部控制流或状态更新。

### Lines 133-147 / 第 133-147 行
````python
        relax_f32_to_f16,
    )

    class NnapiInterfaceWrapper(torch.nn.Module):
        """NNAPI list-ifying and de-list-ifying wrapper.

        NNAPI always expects a list of inputs and provides a list of outputs.
        This module allows us to accept inputs as separate arguments.
        It returns results as either a single tensor or tuple,
        matching the original module.
        """

        def __init__(self, mod):
            super().__init__()
            self.mod = mod
````
- **EN**: It introduces or extends `NnapiInterfaceWrapper`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics.
- **CN**: 它引入或扩展了 `NnapiInterfaceWrapper`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。

### Lines 149-162 / 第 149-162 行
````python
    wrapper_model_py = NnapiInterfaceWrapper(nnapi_model)
    wrapper_model = torch.jit.script(wrapper_model_py)
    # TODO: Maybe make these names match the original.
    arg_list = ", ".join(f"arg_{idx}" for idx in range(len(inputs)))
    if retval_count < 0:
        ret_expr = "retvals[0]"
    else:
        ret_expr = "".join(f"retvals[{idx}], " for idx in range(retval_count))
    wrapper_model.define(
        f"def forward(self, {arg_list}):\n"
        f"    retvals = self.mod([{arg_list}])\n"
        f"    return {ret_expr}\n"
    )
    return wrapper_model
````
- **EN**: This chunk continues `NnapiInterfaceWrapper` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `NnapiInterfaceWrapper`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 165-184 / 第 165-184 行
````python
def process_for_nnapi(
    model, inputs, serializer=None, return_shapes=None, use_int16_for_qint16=False
):
    model = torch.jit.freeze(model)

    if isinstance(inputs, torch.Tensor):
        inputs = [inputs]

    serializer = serializer or _NnapiSerializer(
        config=None, use_int16_for_qint16=use_int16_for_qint16
    )
    (
        ser_model,
        used_weights,
        inp_mem_fmts,
        out_mem_fmts,
        shape_compute_lines,
        retval_count,
    ) = serializer.serialize_model(model, inputs, return_shapes)
    ser_model_tensor = torch.tensor(ser_model, dtype=torch.int32)
````
- **EN**: This chunk defines `process_for_nnapi`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `process_for_nnapi`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 186-199 / 第 186-199 行
````python
    # We have to create a new class here every time this function is called
    # because module.define adds a method to the *class*, not the instance.
    class ShapeComputeModule(torch.nn.Module):
        """Code-gen-ed module for tensor shape computation.

        module.prepare will mutate ser_model according to the computed operand
        shapes, based on the shapes of args.  Returns a list of output templates.
        """

    shape_compute_module = torch.jit.script(ShapeComputeModule())
    real_shape_compute_lines = [
        "def prepare(self, ser_model: torch.Tensor, args: List[torch.Tensor]) -> List[torch.Tensor]:\n",
    ] + [f"    {line}\n" for line in shape_compute_lines]
    shape_compute_module.define("".join(real_shape_compute_lines))
````
- **EN**: It introduces or extends `ShapeComputeModule`, which hold the main object-oriented state for this portion of the file. This chunk continues `ShapeComputeModule` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `ShapeComputeModule`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `ShapeComputeModule`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 201-208 / 第 201-208 行
````python
    return (
        shape_compute_module,
        ser_model_tensor,
        used_weights,
        inp_mem_fmts,
        out_mem_fmts,
        retval_count,
    )
````
- **EN**: This chunk continues `ShapeComputeModule` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `ShapeComputeModule`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Backend configuration**
  - EN: Exposes optional-runtime toggles and capability checks that alter low-level execution behavior.
  - CN: 暴露可选运行时开关与能力检查，以改变底层执行行为。
- **ANEURALNETWORKS_PREFER_LOW_POWER**
  - EN: `ANEURALNETWORKS_PREFER_LOW_POWER` is one of the main symbols declared or implemented in this file.
  - CN: `ANEURALNETWORKS_PREFER_LOW_POWER` 是本文件声明或实现的主要符号之一。
- **ANEURALNETWORKS_PREFER_FAST_SINGLE_ANSWER**
  - EN: `ANEURALNETWORKS_PREFER_FAST_SINGLE_ANSWER` is one of the main symbols declared or implemented in this file.
  - CN: `ANEURALNETWORKS_PREFER_FAST_SINGLE_ANSWER` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.backends._nnapi.serializer`
- **Standard library / 标准库**: `typing`
- **Primary symbols in this file / 本文件核心符号**: `ANEURALNETWORKS_PREFER_LOW_POWER`, `ANEURALNETWORKS_PREFER_FAST_SINGLE_ANSWER`, `ANEURALNETWORKS_PREFER_SUSTAINED_SPEED`, `NnapiModule`, `convert_model_to_nnapi`, `process_for_nnapi`
