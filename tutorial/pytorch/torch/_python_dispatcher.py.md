# _python_dispatcher.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_python_dispatcher.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides general-purpose torch Python helpers, package wiring, or small runtime-facing utilities.
- **Purpose (CN)**: 提供通用的 torch Python 辅助逻辑、包级连接代码或小型运行时工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行
````python
# mypy: allow-untyped-defs
import re

import torch._C as C


"""
PythonDispatcher class is a thin python-binding to C++ dispatcher and it
is designed to show how dispatcher precompute works. In particular,
it shows for a certain op `foo`, what the computed dispatch table looks
like after user register their kernels to certains dispatch keys.
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch._C; standard-library helpers such as re. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch._C；标准库辅助模块，如 re。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 13-26 / 第 13-26 行
````python
In the real C++ dispatcher we support many dispatch keys for different
functionalities. For simplicity PythonDispatcher only supports dispatch
keys for a single example of each use case. These use cases are listed below:

- CPU/AutogradCPU: represents in-tree backends which we usually have dedicated inference &
    autograd kernel in pytorch core library.
    E.g. CPU, CUDA
- FPGA/AutogradOther: represents in-tree backends which we usually have backend specific
    inference kernels, but they share the same autograd kernel specified in AutogradOther.
    E.g. FPGA, SparseCsrCPU
- XLA/AutogradXLA: represents out-of-tree backends which we don't have either inference or autograd
    kernel defined in pytorch core library. Backend owner is responsible for registering both
    inference & autograd kernels in their extensions(e.g. torch-xla) for the operators they support.
    E.g. XLA, XPU, MPS
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 27-38 / 第 27-38 行
````python
- CompositeExplicitAutograd: alias key mapped to inference kernels of all backends like CPU, CUDA, XLA etc.
    Kernels registered to this key MUST work for inference for all backends.
- Autograd: alias key mapped to autograd of all backends like AutogradCPU, AutogradXLA, AutogradOther.
    Kernels registered to this key MUST work for autograd for all backends.
- CompositeImplicitAutograd: alias key CompositeImplicitAutograd = CompositeExplicitAutograd + Autograd
    Kernels registered to this key MUST work for both inference + autograd for all backends.

Note we only allow registrations to alias keys inside pytorch core library. E.g
you shouldn't register a CompositeImplicitAutograd or CompositeExplicitAutograd
kernel from torch-xla extension, instead you should upstream the kernel into
pytorch/pytorch repo so that it's available for all backends and continuously
tested even without the extension.
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 40-52 / 第 40-52 行
````python
Usage:
  dispatcher = PythonDispatcher()
  dispatcher.register(["CPU", "XLA", "CompositeImplicitAutograd"])
  print(dispatcher.dispatchTable()) # This tells you exactly which kernel is used for certain backend.
  # For more debugging information
  # print(dispatcher.keys())
  # print(dispatcher.registrations())
  # print(dispatcher.rawRegistrations())
  # print(dispatcher.rawDispatchTable())
PythonDispatcher calls C++ dispatcher under the hood for to precompute dispatch table.
This file only provides the simplified API for developers, relevant test code is located in
test/test_dispatch.py
"""
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 55-68 / 第 55-68 行
````python
class PythonDispatcher:
    namespace = "__test__"
    name = "foo"
    # fmt: off
    runtime_keys = [
        "CPU", "AutogradCPU",
        "FPGA", "AutogradOther",
        "XLA", "AutogradXLA",
        "Lazy", "AutogradLazy",
    ]
    # fmt: on
    alias_keys = [
        "CompositeExplicitAutograd",
        "Autograd",
````
- **EN**: It introduces or extends `PythonDispatcher`, which hold the main object-oriented state for this portion of the file.
- **CN**: 它引入或扩展了 `PythonDispatcher`，这些类承载了本段涉及的主要面向对象状态。

### Lines 69-81 / 第 69-81 行
````python
        "CompositeImplicitAutograd",
    ]
    supported_keys = runtime_keys + alias_keys

    def __init__(self) -> None:
        C._dispatch_check_invariants(self.name)  # type: ignore[attr-defined]
        self.ref = C._dispatch_library("FRAGMENT", self.namespace, "")
        self.ref.def_("foo(Tensor x) -> Tensor")

    """
    Returns a list of dispatch keys supported by PythonDispatcher.
    You can register kernels to these keys.
    """
````
- **EN**: This chunk defines `__init__`, which initializes object state and binds construction-time configuration.
- **CN**: 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。

### Lines 83-92 / 第 83-92 行
````python
    def keys(self):
        return self.supported_keys

    """
    Register kernels to the target dispatchKeys.
    dispatchKeys(list[str]): a list of dispatch keys that you want to register
      your own kernel. Note that you don't need to write the kernel yourself in
      this PythonDispatcher.E.g. for CPU key, a kernel(e.g fn_CPU for CPU) is
      automatically generated and registered.
    """
````
- **EN**: This chunk defines `keys`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `keys`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 94-107 / 第 94-107 行
````python
    def register(self, dispatchKeys):
        # Overridden is not supported and triggers a warning in C++ dispatcher.
        if len(set(dispatchKeys)) != len(dispatchKeys):
            raise RuntimeError(
                f"Overridden is not allowed but found duplicates in {dispatchKeys}."
            )
        # We currently forbid this in codegen instead of C++ dispatcher.
        if (
            "CompositeImplicitAutograd" in dispatchKeys
            and "CompositeExplicitAutograd" in dispatchKeys
        ):
            raise RuntimeError(
                "Registration to both CompositeImplicitAutograd and CompositeExplicitAutograd is not allowed."
            )
````
- **EN**: This chunk defines `register`, which registers a hook, schema, operator, or callback with surrounding infrastructure. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段定义了 `register`，其作用是向周边基础设施注册钩子、schema、算子或回调。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 108-120 / 第 108-120 行
````python
        for key in dispatchKeys:
            if key not in self.supported_keys:
                raise RuntimeError(
                    f"{key} is not supported, please select a dispatch key in {self.supported_keys}."
                )
            self.ref.impl_t_t("foo", dispatch=key, debug="fn_" + key)

    """
    Helper function to format (key, kernel).
    """

    def _format_line(self, key, kernel):
        return f"{key:<15} {kernel}\n"
````
- **EN**: This chunk defines `_format_line`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_format_line`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 122-132 / 第 122-132 行
````python
    """
    Helper function to print a table header.
    """

    def _format_header(self, header):
        s = f"""
{header}
"""
        s += self._format_line("key", "kernel")
        s += "---------------------------\n"
        return s
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_format_header`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_format_header`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 134-145 / 第 134-145 行
````python
    """
    Returns raw output of all registration info for debugging only.
    Use registrations() for a simplified version.
    """

    def rawRegistrations(self):
        return C._dispatch_dump(f"{self.namespace}::{self.name}")  # type: ignore[attr-defined]

    """
    Returns raw output of computed dispatch table for debugging only.
    Use dispatchTable() for a simplified version.
    """
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `rawRegistrations`, which implements a focused helper used by the surrounding module. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `rawRegistrations`，其作用是实现周边模块使用的关键辅助逻辑。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 147-160 / 第 147-160 行
````python
    def rawDispatchTable(self):
        return C._dispatch_dump_table(f"{self.namespace}::{self.name}")  # type: ignore[attr-defined]

    """
    Returns a table(str) including all the registrations from users.
    Note this includes registrations to both runtime keys and alias keys.
    """

    def registrations(self):
        output = self._format_header("Registered Kernels")
        state = self.rawRegistrations()
        state_entries = state.split("\n")
        for line in state_entries:
            first = line.split(":")[0]
````
- **EN**: This chunk defines `registrations`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `registrations`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 161-170 / 第 161-170 行
````python
            if any(first.startswith(k) for k in self.supported_keys):
                kernel = line.split("::")[0].split(" ")[1]
                output += self._format_line(first, kernel)
        return output

    """
    Returns the computed dispatch table(str). Note this only include
    runtime keys, registrations to alias keys have been decoded to their
    mapped runtime keys.
    """
````
- **EN**: This chunk continues `registrations` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `registrations`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 172-182 / 第 172-182 行
````python
    def dispatchTable(self):
        output = self._format_header("Computed Dispatch Table")
        table = self.rawDispatchTable()
        table_entries = table.split("\n")
        regex = re.compile(r"registered at .*FallbackKernel\.cpp.*(\[)")
        for line in table_entries:
            k = line.split(":")[0]
            if k in self.runtime_keys:
                entry = regex.sub("[", line)
                output += self._format_line(k, entry.split(": ")[1])
        return output
````
- **EN**: This chunk defines `dispatchTable`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `dispatchTable`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **PythonDispatcher**
  - EN: `PythonDispatcher` is one of the main symbols declared or implemented in this file.
  - CN: `PythonDispatcher` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Gradient semantics**
  - EN: The implementation preserves gradient correctness and explains how backward/JVP state is tracked.
  - CN: 实现会维护梯度正确性，并说明如何跟踪 backward/JVP 状态。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
- **Deferred execution**
  - EN: The file records intent or metadata now so execution can be materialized later.
  - CN: 该文件先记录意图或元数据，等待后续阶段再真正执行。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch._C`
- **Standard library / 标准库**: `re`
- **Primary symbols in this file / 本文件核心符号**: `PythonDispatcher`
