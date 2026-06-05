# cpp_wrapper_gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/cpp_wrapper_gpu.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `DeferredTritonCallWrapper`, `CppWrapperGpu`, and `UnwrapUnspecArg`. It exposes functions such as `cpp_string_literal`, `signature_is_tma_desc`, and `_unpack_tma_descriptor_args`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `DeferredTritonCallWrapper`、`CppWrapperGpu`、`UnwrapUnspecArg` 等类。同时提供 `cpp_string_literal`、`signature_is_tma_desc`、`_unpack_tma_descriptor_args` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
from __future__ import annotations

import dataclasses
import re
import sys
from itertools import count, zip_longest
from typing import Any
from typing_extensions import Self

import sympy

import torch
from torch import dtype as torch_dtype
from torch._inductor.codecache import get_cpp_wrapper_cubin_path_name
from torch._inductor.runtime.runtime_utils import dynamo_timed
from torch.utils._ordered_set import OrderedSet

from .. import config
from ..codecache import CudaKernelParamCache
from ..ir import (
    GraphPartitionSignature,
    TensorBox,
    TMADescriptorExperimental,
    TMADescriptorStable,
)
from ..utils import (
    cache_on_self,
````
- **EN**: Imports dependencies such as `__future__`, `dataclasses`, `re`, `sys`, `itertools`, `typing`, and `...+10` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `__future__`、`dataclasses`、`re`、`sys`、`itertools`、`typing`、`另有10项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 29-56 / 第 29-56 行
````python
    get_gpu_type,
    GPU_ALIGN_BYTES,
    IndentedBuffer,
    XPU_KERNEL_FORMAT,
)
from ..virtualized import V
from .aoti_hipify_utils import maybe_hipify_code_wrapper
from .common import get_device_op_overrides, TritonScratchWorkspace
from .cpp_utils import cexpr
from .cpp_wrapper_cpu import CppWrapperCpu
from .multi_kernel import MultiKernelCall
from .triton_utils import should_unwrap_unspec_arg
from .wrapper import PythonWrapperCodegen, SymbolicCallArg


_cpp_string_literal_escapes = {
    "\\": "\\\\",
    '"': '\\"',
    "\n": "\\n",
    "\t": "\\t",
    "\r": "\\r",
}
_cpp_string_literal_pattern = re.compile(r'["\\\n\t\r]')


def cpp_string_literal(s: str) -> str:
    escaped = _cpp_string_literal_pattern.sub(
        lambda match: _cpp_string_literal_escapes[match.group(0)], s
````
- **EN**: Imports dependencies such as `..virtualized`, `.aoti_hipify_utils`, `.common`, `.cpp_utils`, `.cpp_wrapper_cpu`, `.multi_kernel`, and `...+2` for the logic in this range. Introduces function `cpp_string_literal`. Initializes or updates values such as `_cpp_string_literal_escapes`, `_cpp_string_literal_pattern`, and `escaped`.
- **CN**: 这里导入了 `..virtualized`、`.aoti_hipify_utils`、`.common`、`.cpp_utils`、`.cpp_wrapper_cpu`、`.multi_kernel`、`另有2项` 等依赖，为后续逻辑提供基础能力。这里定义了函数`cpp_string_literal`。初始化或更新了 `_cpp_string_literal_escapes`、`_cpp_string_literal_pattern`、`escaped` 等值。

### Lines 57-84 / 第 57-84 行
````python
    )
    return f'"{escaped}"'


TRITON_SIGNATURE_TO_CPP = {
    "i32": "int32_t",
    "i64": "int64_t",
    "fp32": "float",
    "fp64": "double",
}


def signature_is_tma_desc(sig: str | None) -> bool:
    """Check if a Triton signature represents a TMA descriptor."""
    if not sig:
        return False
    if sig == "nvTmaDesc":
        return True
    if sig.startswith("tensordesc<"):
        return True
    return False


def _unpack_tma_descriptor_args(var_name: str, sig_type: str) -> list[str]:
    """Unpack a StableTMADescriptor into kernel launch args.

    Given a variable name holding a StableTMADescriptor and its tensordesc<...>
    signature, returns the list of pointer args: &var.m, &var.block_shape[i]...,
````
- **EN**: Introduces function `signature_is_tma_desc`, function `_unpack_tma_descriptor_args`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`signature_is_tma_desc`、函数`_unpack_tma_descriptor_args`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 85-112 / 第 85-112 行
````python
    &var.strides[i]...
    """
    match = re.match(r"tensordesc<[^[]*\[([^\]]*)\]", sig_type)
    assert match is not None, f"Cannot parse tensordesc signature: {sig_type}"
    ndim = match.group(1).count(",") + 1
    result = [f"&{var_name}.m"]
    for i in range(ndim):
        result.append(f"&{var_name}.block_shape[{i}]")
    for i in range(ndim):
        result.append(f"&{var_name}.strides[{i}]")
    return result


@dataclasses.dataclass
class DeferredTritonCallWrapper:
    """
    When using cpp wrapper, GPU kernel load and launch needs to wait for Triton kernels
    to be tuned and stored as cubin files, so use a deferred generating the final wrapper around
    the triton kernel until right before the prefix is written.
    """

    wrapper_name: str
    kernel_name: str
    kernel_name_to_body: dict[str, str]
    arg_types: list[Any]
    triton_meta: dict[str, Any] | None = None
    inductor_meta: dict[str, Any] | None = None
    tma_tensor_args: dict[str, str] = dataclasses.field(default_factory=dict)
````
- **EN**: Introduces class `DeferredTritonCallWrapper`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了类`DeferredTritonCallWrapper`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 113-140 / 第 113-140 行
````python

    @cache_on_self
    def _get_tma_args(self) -> dict[str, str]:
        """Get mapping of TMA descriptor arg names to their signature types."""
        triton_meta = self.triton_meta or {}
        signature = triton_meta.get("signature", {})
        for name, sig_type in signature.items():
            if sig_type == "nvTmaDesc":
                raise RuntimeError(
                    f"nvTmaDesc (experimental TMA API) is not supported in lazy compile "
                    f"for arg '{name}'. Use the stable tensordesc API instead."
                )
        return {
            name: sig_type
            for name, sig_type in signature.items()
            if isinstance(sig_type, str) and sig_type.startswith("tensordesc<")
        }

    def _get_cpp_param_type(
        self, name: str, arg_type: Any, signature: dict[str, str] | None = None
    ) -> str:
        """Get the C++ parameter declaration for a given arg type."""
        if isinstance(arg_type, (torch_dtype, UnwrapUnspecArg)):
            # TMA descriptors need non-const references since their fields
            # are passed as void* pointers to kernel launch args
            if signature and signature_is_tma_desc(signature.get(name)):
                return f"{name}_type_& {name}"
            return f"const {name}_type_& {name}"
````
- **EN**: Introduces function `_get_tma_args`, function `_get_cpp_param_type`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_tma_args`、函数`_get_cpp_param_type`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-168 / 第 141-168 行
````python
        elif issubclass(arg_type, (SymbolicCallArg, sympy.Expr, int)):
            return f"int64_t {name}"
        elif arg_type is float:
            return f"float {name}"
        elif arg_type is bool:
            return f"bool {name}"
        else:
            raise ValueError(f"Unexpected arg type {arg_type}")

    def _write_wrapper_signature(
        self,
        prefix: IndentedBuffer,
        wrapper: CppWrapperGpu,
        arg_names: list[str],
        arg_types: list[Any] | None = None,
        signature: dict[str, str] | None = None,
    ) -> None:
        """Write the wrapper function signature including template and parameters."""
        if arg_types is None:
            arg_types = self.arg_types

        # Generate template types for tensor arguments
        template_types = [
            f"typename {name}_type_"
            for name, arg_type in zip(arg_names, arg_types)
            if isinstance(arg_type, (torch_dtype, UnwrapUnspecArg))
        ]
        if V.graph.aot_mode:
````
- **EN**: Introduces function `_write_wrapper_signature`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_write_wrapper_signature`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 169-196 / 第 169-196 行
````python
            template_types.append("typename kernels_type_")

        if template_types:
            prefix.writeline(f"template <{', '.join(template_types)}>")

        # Build parameter list
        param_lines = [
            self._get_cpp_param_type(name, arg_type, signature)
            for name, arg_type in zip(arg_names, arg_types)
        ]
        param_lines.append("int32_t device_idx_")
        param_lines.append(
            maybe_hipify_code_wrapper(
                f"{wrapper.device_codegen.cpp_stream_type()} stream_"
            )
        )
        if V.graph.aot_mode:
            param_lines.append("kernels_type_& kernels_")
        param_lines.append(
            "const std::optional<std::string>& cubin_dir_ = std::nullopt"
        )

        # Write function signature
        prefix.writeline(f"static __attribute__((noinline)) void {self.wrapper_name}(")
        with prefix.indent():
            for i, param in enumerate(param_lines):
                comma = "," if i < len(param_lines) - 1 else ""
                prefix.writeline(f"{param}{comma}")
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `param_lines`, and `comma`. This range continues the implementation of function `DeferredTritonCallWrapper._write_wrapper_signature`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `param_lines`、`comma` 等值。这一段延续了函数`DeferredTritonCallWrapper._write_wrapper_signature` 的具体实现。

### Lines 197-224 / 第 197-224 行
````python
        prefix.writeline("){")

    def generate(self, wrapper: CppWrapperGpu):
        """
        Generate the GPU kernel definition, as well as load and launch code.
        """
        prefix = wrapper.prefix
        if self.kernel_name.startswith("multi_kernel_"):
            # MultiKernel will select one kernel after running the autotune block
            self.kernel_name = MultiKernelCall.lookup_choice(self.kernel_name)

        # Defer compilation to runtime if autotune_at_compile_time is False (JIT only)
        if not V.graph.aot_mode and config.triton.autotune_at_compile_time is False:
            return self.generate_lazy(wrapper)

        params = CudaKernelParamCache.get(self.kernel_name)
        assert params, f"CudaKernelParamCache not populated for {self.kernel_name}"
        def_args = params["def_args"]
        arg_types = self.arg_types
        inductor_meta = params["inductor_meta"]

        if "extra_launcher_args" in inductor_meta and len(def_args) > len(arg_types):
            # extra_launcher_args should already be in def_args
            assert len(def_args) == len(arg_types) - len(
                inductor_meta["extra_launcher_args"]
            )
            arg_types = arg_types + [SymbolicCallArg] * len(
                inductor_meta["extra_launcher_args"]
````
- **EN**: Introduces function `generate`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`generate`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 225-252 / 第 225-252 行
````python
            )

        if not V.graph.aot_mode:
            prefix.writeline(
                maybe_hipify_code_wrapper(
                    f"static {wrapper.device_codegen.cpp_kernel_type()} {self.kernel_name} = nullptr;"
                )
            )
            kernel_var_name = self.kernel_name
        else:
            kernel_var_name = f"kernels_.{self.kernel_name}"

        # Write wrapper function signature
        self._write_wrapper_signature(prefix, wrapper, def_args, arg_types)

        with prefix.indent():
            if V.graph.aot_mode:
                # Emit the original Triton kernel for debugging purposes
                prefix.writeline("/*")
                prefix.splice(self.kernel_name_to_body[self.kernel_name])
                prefix.writeline("*/")
            self.generate_grid(prefix, inductor_meta, params)
            self.generate_load_kernel(prefix, kernel_var_name, params)
            self.generate_launch_kernel(prefix, wrapper, kernel_var_name, params)
        prefix.writeline("}")

        if not config.aot_inductor.embed_kernel_binary:
            # Ensure the cubin file is included in the package
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_var_name`, and `else`. This range continues the implementation of function `DeferredTritonCallWrapper.generate`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_var_name`、`else` 等值。这一段延续了函数`DeferredTritonCallWrapper.generate` 的具体实现。

### Lines 253-280 / 第 253-280 行
````python
            V.graph.wrapper_code.additional_files.append(
                params[get_cpp_wrapper_cubin_path_name()]
            )

    def _resolve_lazy_arg_names(self) -> tuple[list[str], list[str]]:
        """Compute wrapper and kernel arg names from triton_meta signature.

        Returns (wrapper_arg_names, kernel_arg_names) where:
        - wrapper_arg_names: params accepted by the C++ wrapper function
        - kernel_arg_names: params passed to the GPU kernel launch (non-constexpr only)
        """
        assert self.triton_meta is not None, (
            f"triton_meta is required for lazy compile of {self.kernel_name}"
        )
        signature = self.triton_meta.get("signature", {})
        inductor_meta = self.inductor_meta or {}
        extra_launcher_args_count = len(inductor_meta.get("extra_launcher_args", []))
        tma_tensor_args = self.tma_tensor_args
        num_tma_tensor_args = len(tma_tensor_args)

        internal_config_suffixes = ("BLOCK", "RSPLIT", "RSPLIT_SIZE")
        # Declared constexpr params (tl.constexpr in kernel signature) are excluded
        # from arg_types for user-defined kernels, while value-based constexpr params
        # (e.g. numel=1, arg=None) are still in arg_types.
        declared_constexpr_names = OrderedSet(
            inductor_meta.get("declared_constexpr_names", [])
        )
        wrapper_arg_names = []
````
- **EN**: Introduces function `_resolve_lazy_arg_names`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `signature`, `inductor_meta`, `extra_launcher_args_count`, `tma_tensor_args`, `num_tma_tensor_args`, `internal_config_suffixes`, and `...+2`.
- **CN**: 这里定义了函数`_resolve_lazy_arg_names`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `signature`、`inductor_meta`、`extra_launcher_args_count`、`tma_tensor_args`、`num_tma_tensor_args`、`internal_config_suffixes`、`另有2项` 等值。

### Lines 281-308 / 第 281-308 行
````python
        kernel_arg_names = []
        for name, sig_type in signature.items():
            if name.endswith(internal_config_suffixes):
                continue
            if sig_type != "constexpr":
                kernel_arg_names.append(name)
            if name not in declared_constexpr_names:
                wrapper_arg_names.append(name)

        num_wrapper_args = (
            len(self.arg_types) - extra_launcher_args_count - num_tma_tensor_args
        )
        if num_wrapper_args != len(wrapper_arg_names):
            raise AssertionError(
                f"Mismatch between ({num_wrapper_args}) arg_types and "
                f"{len(wrapper_arg_names)} wrapper_arg_names for {self.kernel_name}."
            )

        # Append grid args: passed to wrapper. Kernel args will handle grids separately.
        for i in range(extra_launcher_args_count):
            wrapper_arg_names.append(f"_grid_{i}")

        # Add TMA tensor args after grid args
        if tma_tensor_args:
            sig_tma_keys = list(self._get_tma_args().keys())
            assert list(tma_tensor_args.keys()) == sig_tma_keys, (
                f"TMA tensor args order mismatch for {self.kernel_name}: "
                f"{list(tma_tensor_args.keys())} vs signature order {sig_tma_keys}"
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_arg_names`, `num_wrapper_args`, and `sig_tma_keys`. This range continues the implementation of function `DeferredTritonCallWrapper._resolve_lazy_arg_names`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_arg_names`、`num_wrapper_args`、`sig_tma_keys` 等值。这一段延续了函数`DeferredTritonCallWrapper._resolve_lazy_arg_names` 的具体实现。

### Lines 309-336 / 第 309-336 行
````python
            )
        for desc_name in tma_tensor_args:
            wrapper_arg_names.append(f"_tma_tensor_{desc_name}")

        return wrapper_arg_names, kernel_arg_names

    def _generate_lazy_grid(self, prefix: IndentedBuffer) -> None:
        """Generate grid computation code for lazy-compiled kernels."""
        kernel_name = self.kernel_name
        grid_type = self.inductor_meta.get("grid_type") if self.inductor_meta else None

        # For PrecomputedGrid, generate switch statement on config_index
        if grid_type == "PrecomputedGrid":
            assert self.inductor_meta is not None
            precomputed_grids = self.inductor_meta.get("precomputed_grids", [])
            extra_launcher_args = self.inductor_meta.get("extra_launcher_args", [])

            switch_cases = []
            for idx, entry in enumerate(precomputed_grids):
                cpp_grids = list(entry.get("cpp", ["1L", "1L", "1L"]))
                # Replace internal arg names with C++ parameter names
                # e.g., _launcher_s0 -> _grid_0
                for i, arg_name in enumerate(extra_launcher_args):
                    cpp_grids = [g.replace(arg_name, f"_grid_{i}") for g in cpp_grids]
                g0 = cpp_grids[0]
                g1 = cpp_grids[1] if len(cpp_grids) > 1 else "1"
                g2 = cpp_grids[2] if len(cpp_grids) > 2 else "1"
                switch_cases.append(
````
- **EN**: Introduces function `_generate_lazy_grid`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_name`, `grid_type`, `precomputed_grids`, `extra_launcher_args`, `switch_cases`, `cpp_grids`, and `...+3`.
- **CN**: 这里定义了函数`_generate_lazy_grid`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_name`、`grid_type`、`precomputed_grids`、`extra_launcher_args`、`switch_cases`、`cpp_grids`、`另有3项` 等值。

### Lines 337-364 / 第 337-364 行
````python
                    f"case {idx}: grid_0 = {g0}; grid_1 = {g1}; grid_2 = {g2}; break;"
                )
            switch_cases.append("default: grid_0 = 1; grid_1 = 1; grid_2 = 1; break;")
            switch_body = "\n                        ".join(switch_cases)

            prefix.splice(
                f"""\
                uint32_t grid_0, grid_1, grid_2;
                switch ({kernel_name}_result.config_index) {{
                    {switch_body}
                }}
                if (grid_0 == 0) return;
                """
            )
        else:
            from ..runtime.triton_heuristics import GridExpr

            grid = GridExpr.from_meta_lazy(self.inductor_meta, kernel_name)
            for line in grid.prefix:
                prefix.writeline(line)

            prefix.splice(
                f"""\
                uint32_t grid_0 = {grid.x_grid};
                uint32_t grid_1 = {grid.y_grid};
                uint32_t grid_2 = {grid.z_grid};
                if (grid_0 == 0) return;
                """
````
- **EN**: Imports dependencies such as `..runtime.triton_heuristics` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `switch_body`, `else`, and `grid`.
- **CN**: 这里导入了 `..runtime.triton_heuristics` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `switch_body`、`else`、`grid` 等值。

### Lines 365-392 / 第 365-392 行
````python
            )

    def _generate_lazy_tma_args(
        self,
        prefix: IndentedBuffer,
        call_args_str: str,
        kernel_arg_names: list[str],
        tma_arg_names: OrderedSet[str],
        signature: dict[str, str],
    ) -> str:
        """Unpack TMA descriptor args into kernel launch args."""
        for arg_name in kernel_arg_names:
            if arg_name in tma_arg_names:
                tma_parts = _unpack_tma_descriptor_args(arg_name, signature[arg_name])
                tma_str = ", ".join(tma_parts)
                call_args_str = (
                    f"{call_args_str}, {tma_str}" if call_args_str else tma_str
                )
        return call_args_str

    def _generate_lazy_scratch(
        self,
        prefix: IndentedBuffer,
        wrapper: CppWrapperGpu,
        call_args_str: str,
    ) -> str:
        """Generate scratch space allocations with runtime-known sizes."""
        kernel_name = self.kernel_name
````
- **EN**: Introduces function `_generate_lazy_tma_args`, function `_generate_lazy_scratch`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `prefix`, `call_args_str`, `kernel_arg_names`, `tma_arg_names`, `signature`, `tma_parts`, and `...+3`.
- **CN**: 这里定义了函数`_generate_lazy_tma_args`、函数`_generate_lazy_scratch`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `prefix`、`call_args_str`、`kernel_arg_names`、`tma_arg_names`、`signature`、`tma_parts`、`另有3项` 等值。

### Lines 393-420 / 第 393-420 行
````python
        dtype_str = wrapper.codegen_dtype(torch.uint8)
        device_type, _ = wrapper.codegen_device(torch.device(get_gpu_type())).split(
            ", "
        )
        device_ptr_type = wrapper.device_codegen.cpp_device_ptr()
        for scratch_name in ("global_scratch", "profile_scratch"):
            size_expr = f"{kernel_name}_result.{scratch_name}"
            var = f"{scratch_name}_ptr"
            prefix.splice(
                maybe_hipify_code_wrapper(
                    f"""\
                {device_ptr_type} {var} = 0;
                RAIIAtenTensorHandle {var}_tensor;
                if ({size_expr} > 0) {{
                    int64_t {var}_size[] = {{{size_expr}}};
                    int64_t {var}_stride[] = {{1}};
                    AtenTensorHandle {var}_handle;
                    AOTI_TORCH_ERROR_CODE_CHECK(aoti_torch_empty_strided(
                        1, {var}_size, {var}_stride, {dtype_str},
                        {device_type}, device_idx_, &{var}_handle));
                    {var}_tensor = RAIIAtenTensorHandle({var}_handle);
                    {var} = reinterpret_cast<{device_ptr_type}>({var}_tensor.data_ptr());
                }}
            """
                )
            )
            call_args_str += f", &{var}"
        return call_args_str
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dtype_str`, `device_ptr_type`, `size_expr`, and `var`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dtype_str`、`device_ptr_type`、`size_expr`、`var` 等值。

### Lines 421-448 / 第 421-448 行
````python

    def _generate_lazy_launch(
        self,
        prefix: IndentedBuffer,
        wrapper: CppWrapperGpu,
        wrapper_arg_names: list[str],
        kernel_arg_names: list[str],
    ) -> None:
        """Generate kernel launch code for lazy-compiled kernels."""
        kernel_name = self.kernel_name
        signature = (self.triton_meta or {}).get("signature", {})
        tma_tensor_args = self.tma_tensor_args
        num_tma_tensor_args = len(tma_tensor_args)

        # wrapper_arg_names may include grid and TMA tensor args at the end;
        # only the leading portion maps 1:1 to kernel signature params.
        num_signature_args = len(wrapper_arg_names) - num_tma_tensor_args
        inductor_meta = self.inductor_meta or {}
        num_signature_args -= len(inductor_meta.get("extra_launcher_args", []))

        arg_type_lookup = dict(
            zip(wrapper_arg_names, self.arg_types[:num_signature_args])
        )

        # Identify TMA args — they are already passed as StableTMADescriptor params,
        # so we just unpack them directly (no need to reconstruct from tensors).
        tma_arg_names = OrderedSet(self._get_tma_args().keys())

````
- **EN**: Introduces function `_generate_lazy_launch`. Initializes or updates values such as `prefix`, `wrapper`, `wrapper_arg_names`, `kernel_arg_names`, `kernel_name`, `signature`, and `...+6`.
- **CN**: 这里定义了函数`_generate_lazy_launch`。初始化或更新了 `prefix`、`wrapper`、`wrapper_arg_names`、`kernel_arg_names`、`kernel_name`、`signature`、`另有6项` 等值。

### Lines 449-476 / 第 449-476 行
````python
        # Non-TMA args go through generate_args_decl
        non_tma_arg_names = [n for n in kernel_arg_names if n not in tma_arg_names]
        non_tma_arg_types = [
            arg_type_lookup[n] for n in non_tma_arg_names if n in arg_type_lookup
        ]
        non_tma_arg_sigs = [signature.get(n) for n in non_tma_arg_names]

        call_args_str = wrapper.generate_args_decl(
            prefix,
            non_tma_arg_names,
            non_tma_arg_types,
            non_tma_arg_sigs,
        )

        call_args_str = self._generate_lazy_tma_args(
            prefix, call_args_str, kernel_arg_names, tma_arg_names, signature
        )
        call_args_str = self._generate_lazy_scratch(prefix, wrapper, call_args_str)

        launch_args = (
            f"{kernel_name}, grid_0, grid_1, grid_2,"
            f" {kernel_name}_result.num_warps,"
            f" {kernel_name}_result.shared_mem,"
            f" kernel_args_, stream_"
        )

        prefix.splice(
            f"""\
````
- **EN**: Initializes or updates values such as `non_tma_arg_names`, `non_tma_arg_types`, `non_tma_arg_sigs`, `call_args_str`, and `launch_args`. This range continues the implementation of function `DeferredTritonCallWrapper._generate_lazy_launch`.
- **CN**: 初始化或更新了 `non_tma_arg_names`、`non_tma_arg_types`、`non_tma_arg_sigs`、`call_args_str`、`launch_args` 等值。这一段延续了函数`DeferredTritonCallWrapper._generate_lazy_launch` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
            void* kernel_args_[] = {{{call_args_str}}};
            launchKernel({launch_args});
            """
        )

    def generate_lazy(self, wrapper: CppWrapperGpu):
        """
        Generate C++ code that embeds Triton source and compiles it at runtime.
        """
        prefix = wrapper.prefix
        kernel_name = self.kernel_name
        # Track kernel names for parallel initialization
        wrapper._lazy_kernel_names.append(kernel_name)

        # Include TMA helpers if any args use TMA descriptors
        tma_signature_types = self._get_tma_args()
        if tma_signature_types:
            wrapper.write_tma_descriptor_helpers_once()

        kernel_var_decl = maybe_hipify_code_wrapper(
            f"static {wrapper.device_codegen.cpp_kernel_type()} {kernel_name} = nullptr;"
        )
        prefix.writeline(kernel_var_decl)
        # Use delimited raw string to handle )" in kernel source
        kernel_source_str = self.kernel_name_to_body.get(kernel_name, "")
        kernel_body = f'R"TRITON(\n{kernel_source_str}\n)TRITON"'
        prefix.writeline(f"static const char* {kernel_name}_source = {kernel_body};")
        prefix.writeline(f"static LazyKernelCompileResult {kernel_name}_result;")
````
- **EN**: Introduces function `generate_lazy`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `prefix`, `kernel_name`, `tma_signature_types`, `kernel_var_decl`, `kernel_source_str`, and `kernel_body`.
- **CN**: 这里定义了函数`generate_lazy`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `prefix`、`kernel_name`、`tma_signature_types`、`kernel_var_decl`、`kernel_source_str`、`kernel_body` 等值。

### Lines 505-532 / 第 505-532 行
````python

        wrapper_arg_names, kernel_arg_names = self._resolve_lazy_arg_names()
        signature = (self.triton_meta or {}).get("signature", {})
        self._write_wrapper_signature(
            prefix, wrapper, wrapper_arg_names, self.arg_types, signature
        )

        # Build autotune args - for TMA, pass tensors instead of descriptors.
        # Only iterate over signature params and grid args, not the trailing
        # TMA tensor params (those are only in the C++ wrapper signature).
        tma_tensor_args = self.tma_tensor_args
        num_autotune_args = len(wrapper_arg_names) - len(tma_tensor_args)
        autotune_arg_list = []
        # Track which args need scalar extraction for the autotune call.
        # UnwrapUnspecArg args are 0-dim tensors in C++ that Triton expects
        # as Python scalars; we use codegen_tensor_item to extract them.
        scalar_extractions: list[tuple[str, str, torch_dtype]] = []
        for idx, name in enumerate(wrapper_arg_names[:num_autotune_args]):
            if name in tma_signature_types:
                autotune_arg_list.append(f"_tma_tensor_{name}")
            elif isinstance(self.arg_types[idx], UnwrapUnspecArg):
                scalar_var = f"_autotune_scalar_{name}"
                scalar_extractions.append((name, scalar_var, self.arg_types[idx].dtype))
                autotune_arg_list.append(scalar_var)
            else:
                autotune_arg_list.append(name)
        autotune_args = ", ".join(autotune_arg_list)
        # Lazy compile with autotuning on first invocation
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `signature`, `tma_tensor_args`, `num_autotune_args`, `autotune_arg_list`, `scalar_extractions`, `scalar_var`, and `...+2`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `signature`、`tma_tensor_args`、`num_autotune_args`、`autotune_arg_list`、`scalar_extractions`、`scalar_var`、`另有2项` 等值。

### Lines 533-560 / 第 533-560 行
````python
        with prefix.indent():
            prefix.writeline(f"if ({kernel_name} == nullptr) {{")
            with prefix.indent():
                for tensor_name, scalar_var, dtype in scalar_extractions:
                    wrapper.codegen_tensor_item(
                        dtype, tensor_name, scalar_var, indented_buffer=prefix
                    )
                prefix.splice(
                    f"""\
                    {kernel_name}_result = runTritonKernelWithAutotune(
                        _module_pending_kernels, "{kernel_name}", stream_, {autotune_args});

                    {kernel_name} = loadKernel(
                        {kernel_name}_result.cubin_path,
                        {kernel_name}_result.mangled_name,
                        {kernel_name}_result.shared_mem);

                    // First invocation already ran the kernel, so return early
                    return;
                    """
                )
            prefix.writeline("}")

            self._generate_lazy_grid(prefix)
            self._generate_lazy_launch(
                prefix,
                wrapper,
                wrapper_arg_names,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `DeferredTritonCallWrapper.generate_lazy`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`DeferredTritonCallWrapper.generate_lazy` 的具体实现。

### Lines 561-588 / 第 561-588 行
````python
                kernel_arg_names,
            )
        prefix.writeline("}")

    def generate_grid(
        self,
        prefix: IndentedBuffer,
        inductor_meta: dict[str, Any],
        params: dict[str, Any],
    ):
        from ..runtime.triton_heuristics import GridExpr

        grid = GridExpr.from_meta(inductor_meta, params["config"], mode="cpp")
        for line in grid.prefix:
            prefix.writeline(line)
        prefix.splice(
            f"""\
            uint32_t grid_0 = {grid.x_grid};
            uint32_t grid_1 = {grid.y_grid};
            uint32_t grid_2 = {grid.z_grid};
            """
        )
        prefix.writeline("if (grid_0 == 0 || grid_1 == 0 || grid_2 == 0) return;")

    def generate_load_kernel(self, prefix, kernel_var_name, params):
        prefix.writeline(f"if ({kernel_var_name} == nullptr) {{")
        with prefix.indent():
            embed_kernel_args = [f"__{params['inductor_meta']['kernel_name']}_start"]
````
- **EN**: Imports dependencies such as `..runtime.triton_heuristics` for the logic in this range. Introduces function `generate_grid`, function `generate_load_kernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..runtime.triton_heuristics` 等依赖，为后续逻辑提供基础能力。这里定义了函数`generate_grid`、函数`generate_load_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 589-616 / 第 589-616 行
````python
            if torch.xpu.is_available():
                # XPU needs the end address of the kernel to calculate the size of the kernel binary.
                embed_kernel_args.append(
                    f"__{params['inductor_meta']['kernel_name']}_end"
                )

            if V.graph.aot_mode and config.aot_inductor.embed_kernel_binary:
                load_kernel_args = [
                    *embed_kernel_args,
                    cpp_string_literal(params["mangled_name"]),
                    str(params["shared_mem"]),
                ]
                if torch.xpu.is_available():
                    is_spv = "true" if XPU_KERNEL_FORMAT == "spv" else "false"
                    if config.aot_inductor.emit_multi_arch_kernel:
                        is_spv = "true"
                    load_kernel_args.append(is_spv)
            else:
                load_kernel_args = [
                    cpp_string_literal(params[get_cpp_wrapper_cubin_path_name()]),
                    cpp_string_literal(params["mangled_name"]),
                    str(params["shared_mem"]),
                    "cubin_dir_",
                ]

            prefix.writeline(
                f"{kernel_var_name} = loadKernel({', '.join(load_kernel_args)}); "
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `load_kernel_args`, `is_spv`, and `else`. This range continues the implementation of function `DeferredTritonCallWrapper.generate_load_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `load_kernel_args`、`is_spv`、`else` 等值。这一段延续了函数`DeferredTritonCallWrapper.generate_load_kernel` 的具体实现。

### Lines 617-644 / 第 617-644 行
````python
        prefix.writeline("}")

    def generate_launch_kernel(self, prefix, wrapper, kernel_var_name, params):
        """
        Generate the GPU kernel launching code.
        This is where all the call args being sorted out and generated.
        If enable_kernel_profile is enabled, all args related information would be packed in this function.
        """
        triton_meta = params["triton_meta"]
        assert len(self.arg_types) == len(params["def_args"]), (
            self.arg_types,
            params["def_args"],
        )
        arg_type_lookup = dict(zip(params["def_args"], self.arg_types))
        # difference between Python and C++ wrapper: C++ wrapper strips out equal_to_1 constants
        call_args = [
            name for name in params["call_args"] if name not in triton_meta["constants"]
        ]
        arg_types = [arg_type_lookup[name] for name in call_args]
        arg_signatures = [triton_meta["signature"][name] for name in call_args]
        num_ctas = params.get("config", {}).get("num_ctas", 1)
        scratch_spaces = {
            name: params[name] * num_ctas
            for name in ["global_scratch", "profile_scratch"]
            if params.get(name, None) is not None
        }
        call_args_str = wrapper.generate_args_decl(
            prefix,
````
- **EN**: Introduces function `generate_launch_kernel`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`generate_launch_kernel`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 645-672 / 第 645-672 行
````python
            call_args,
            arg_types,
            arg_signatures,
            scratch_spaces=scratch_spaces,
        )
        prefix.writeline(f"void* kernel_args_[] = {{{call_args_str}}};")
        launch_kernel_args = [
            kernel_var_name,
            "grid_0",
            "grid_1",
            "grid_2",
            str(params["num_warps"]),
            str(params["shared_mem"]),
            "kernel_args_",
            "stream_",
        ]

        enable_kernel_profile = config.cpp.enable_kernel_profile and sys.platform in [
            "linux",
            "win32",
        ]
        if enable_kernel_profile:
            normalized_kernel_name = re.sub(r"[^a-zA-Z0-9_]", "_", f"{kernel_var_name}")
            prefix.writeline("{")
            with prefix.indent():
                prefix.writelines(
                    [
                        f"std::unordered_map<std::string, C10IValueHandle> kwargs_{normalized_kernel_name};",
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scratch_spaces`, `launch_kernel_args`, `enable_kernel_profile`, and `normalized_kernel_name`. This range continues the implementation of function `DeferredTritonCallWrapper.generate_launch_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `scratch_spaces`、`launch_kernel_args`、`enable_kernel_profile`、`normalized_kernel_name` 等值。这一段延续了函数`DeferredTritonCallWrapper.generate_launch_kernel` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
                        "",
                    ]
                )
                # Add launch args info
                record_launch_kernel_args = [
                    ("grid_0", "grid_0"),
                    ("grid_1", "grid_1"),
                    ("grid_2", "grid_2"),
                    ("num_warps", str(params["num_warps"])),
                    ("shared_mem", str(params["shared_mem"])),
                ]
                for k, v in record_launch_kernel_args:
                    arg_name = f"{normalized_kernel_name}_{k}"
                    prefix.writelines(
                        [
                            f"// Create c10::IValue for {k}",
                            f"C10IValueHandle tmp_{arg_name};",
                            f"aoti_torch_int64_to_ivalue({v}, &tmp_{arg_name});",
                            f"RAIIC10IValueHandle RAII_{arg_name}(tmp_{arg_name});",
                            f'kwargs_{normalized_kernel_name}.emplace("{k}", RAII_{arg_name});',
                        ]
                    )

                # Add input info (This copies the logic from args_decl)
                curr_arg_id = -1
                total_args = []
                ordered_argsname = []

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `record_launch_kernel_args`, `arg_name`, `curr_arg_id`, `total_args`, and `ordered_argsname`. This range continues the implementation of function `DeferredTritonCallWrapper.generate_launch_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `record_launch_kernel_args`、`arg_name`、`curr_arg_id`、`total_args`、`ordered_argsname` 等值。这一段延续了函数`DeferredTritonCallWrapper.generate_launch_kernel` 的具体实现。

### Lines 701-728 / 第 701-728 行
````python
                def write_dummy_scalar_ivalue(arg_name):
                    # We only care about the shape, therefore we create a dummy scalar here.
                    prefix.writelines(
                        [
                            f"// Create c10::IValue for arg_{curr_arg_id}",
                            f"C10IValueHandle tmp_{arg_name};",
                            f"aoti_torch_int64_to_ivalue(0, &tmp_{arg_name});",
                            f"RAIIC10IValueHandle RAII_{arg_name}(tmp_{arg_name});",
                        ]
                    )
                    # pyrefly: ignore [bad-argument-type]
                    total_args.append(f"tmp_{arg_name}")

                def process_args_for_input_shape(arg, arg_type, arg_signature=None):
                    nonlocal curr_arg_id
                    curr_arg_id += 1
                    arg_name = f"{normalized_kernel_name}_arg_{curr_arg_id}"
                    # ignore tma descriptors, as host-side TMA descriptors need
                    # to be passed to the compiled Triton kernel by value
                    if isinstance(
                        arg_type, UnwrapUnspecArg
                    ) and not signature_is_tma_desc(arg_signature):
                        write_dummy_scalar_ivalue(arg_name)
                    elif isinstance(
                        arg_type, torch_dtype
                    ) and not signature_is_tma_desc(arg_signature):
                        # This is an at::Tensor.
                        prefix.writelines(
````
- **EN**: Introduces function `write_dummy_scalar_ivalue`, function `process_args_for_input_shape`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `arg_name`.
- **CN**: 这里定义了函数`write_dummy_scalar_ivalue`、函数`process_args_for_input_shape`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `arg_name` 等值。

### Lines 729-756 / 第 729-756 行
````python
                            [
                                f"// Create c10::IValue for arg_{curr_arg_id}",
                                f"C10IValueHandle tmp_{arg_name};",
                                f"aoti_torch_tensor_to_ivalue({arg}, &tmp_{arg_name});",
                                f"RAIIC10IValueHandle RAII_{arg_name}(tmp_{arg_name});",
                            ]
                        )
                        # pyrefly: ignore [bad-argument-type]
                        total_args.append(f"tmp_{arg_name}")
                    elif (
                        isinstance(arg_type, type(SymbolicCallArg))
                        and arg_signature is not None
                        and arg_signature in TRITON_SIGNATURE_TO_CPP
                    ) or arg_type in (sympy.Integer, int, sympy.Float, float):
                        write_dummy_scalar_ivalue(arg_name)
                    elif arg_signature and arg_signature.startswith("tensordesc<"):
                        # Skip tma related args
                        pass
                    else:
                        write_dummy_scalar_ivalue(arg_name)

                # Add input name and shape information
                for arg, arg_type, arg_signature in zip_longest(
                    call_args, arg_types, arg_signatures
                ):
                    # pyrefly: ignore [bad-argument-type]
                    ordered_argsname.append(f'"{arg}"')
                    process_args_for_input_shape(arg, arg_type, arg_signature)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`. This range continues the implementation of function `DeferredTritonCallWrapper.generate_launch_kernel.process_args_for_input_shape`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。这一段延续了函数`DeferredTritonCallWrapper.generate_launch_kernel.process_args_for_input_shape` 的具体实现。

### Lines 757-784 / 第 757-784 行
````python

                # Add input name into kwargs
                name_var = f"{normalized_kernel_name}_input_names"
                prefix.writelines(
                    [
                        "// Create c10::IValue for input names",
                        f"C10IValueHandle tmp_{name_var};",
                        f"std::vector<const char*> {name_var}({{{', '.join(ordered_argsname)}}});",
                        f"aoti_torch_strlist_to_ivalue({name_var}.data(), {len(ordered_argsname)}, &tmp_{name_var});",
                        f"RAIIC10IValueHandle RAII_{name_var}(tmp_{name_var});",
                        f'kwargs_{normalized_kernel_name}.emplace("Input Args", RAII_{name_var});',
                    ]
                )

                inputs_info_ = f"{normalized_kernel_name}_inputs_info_"
                # We pass in the non-RAII handles, since C10 doesn't automatically free them.
                # The RAII will make sure they get freed when they are out of scope.
                tmp_args = ",".join(total_args)
                prefix.writelines(
                    [
                        "// Aggregate all c10::IValue for inputs",
                        f"std::vector<C10IValueHandle> {inputs_info_}({{{tmp_args}}});",
                    ]
                )

                # Start recording Function
                prefix.writelines(
                    [
````
- **EN**: Initializes or updates values such as `name_var`, `inputs_info_`, and `tmp_args`. This range continues the implementation of function `DeferredTritonCallWrapper.generate_launch_kernel`.
- **CN**: 初始化或更新了 `name_var`、`inputs_info_`、`tmp_args` 等值。这一段延续了函数`DeferredTritonCallWrapper.generate_launch_kernel` 的具体实现。

### Lines 785-812 / 第 785-812 行
````python
                        "",
                        (
                            "torch::aot_inductor::RAIIAtenRecordFunctionHandle "
                            f"record_{normalized_kernel_name}_"
                            f'("{kernel_var_name}", '
                            f"reinterpret_cast<IValueMapHandle>(&kwargs_{normalized_kernel_name}), "
                            f"{inputs_info_});"
                        ),
                        "",
                        f"launchKernel({', '.join(launch_kernel_args)});",
                    ]
                )
            prefix.writeline("}")
        else:
            prefix.writeline(f"launchKernel({', '.join(launch_kernel_args)});")


class CppWrapperGpu(CppWrapperCpu):
    """
    Generates cpp wrapper for running on GPU and calls CUDA kernels
    """

    def __init__(self) -> None:
        self.device = get_gpu_type()
        self.device_codegen = get_device_op_overrides(self.device)
        super().__init__()
        self.grid_id = count()
        self._kernel_name_to_body: dict[str, str] = {}
````
- **EN**: Introduces class `CppWrapperGpu`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了类`CppWrapperGpu`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 813-840 / 第 813-840 行
````python
        self._triton_call_wrappers: dict[str, DeferredTritonCallWrapper] = {}
        self.autotune_input_prefix = "_REAL_AUTOTUNE_INPUT"
        self._lazy_kernel_names: list[str] = []

    @staticmethod
    def create(
        is_subgraph: bool,
        subgraph_name: str | None,
        parent_wrapper: PythonWrapperCodegen | None,
        partition_signatures: GraphPartitionSignature | None = None,
    ):
        # TODO - support subgraph codegen by lifting functions. Check the
        # comment at CppWrapperCpu `codegen_subgraph` function.
        return CppWrapperGpu()

    def write_header(self):
        if V.graph.is_const_graph:
            # We do not write header for constant graph, it will be written by main module.
            return

        super().write_header()
        self.header.splice(
            maybe_hipify_code_wrapper(self.device_codegen.kernel_driver())
        )

    @cache_on_self
    def write_tma_descriptor_helpers_once(self):
        self.header.splice(self.device_codegen.tma_descriptor_helpers())
````
- **EN**: Introduces function `create`, function `write_header`, function `write_tma_descriptor_helpers_once`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`create`、函数`write_header`、函数`write_tma_descriptor_helpers_once`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 841-868 / 第 841-868 行
````python

    def write_get_raw_stream(self, device_idx: int, graph_name: str) -> str:
        name = f"stream{device_idx}"
        self.writeline(
            maybe_hipify_code_wrapper(
                f"{self.device_codegen.cpp_stream_type()} {name};"
            )
        )
        self.writeline(
            f"AOTI_TORCH_ERROR_CODE_CHECK({self.device_codegen.aoti_get_stream()}({device_idx}, (void**)&{name}));"
        )
        return name

    def get_autotuning_input_name(self, idx):
        return f"{self.autotune_input_prefix}_{idx}"

    def codegen_inputs(self):
        # See Note: [Input Alignment handling in Inductor]
        #
        # JIT Inductor does not guard on input alignment. It relies on copy_misaligned_inputs to
        # copy misaligned inputs to aligned buffers. For AOTInductor, we need to do the same in cpp.

        if config.is_fbcode():
            # TODO: This is added because FC. Remove this once the newly added shim symbols,
            # e.g. aoti_torch_clone_preserve_strides, have landed
            return super().codegen_inputs()

        if V.graph.aot_mode and V.graph.inputs_to_check:
````
- **EN**: Introduces function `write_get_raw_stream`, function `get_autotuning_input_name`, function `codegen_inputs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`.
- **CN**: 这里定义了函数`write_get_raw_stream`、函数`get_autotuning_input_name`、函数`codegen_inputs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name` 等值。

### Lines 869-896 / 第 869-896 行
````python
            for idx in V.graph.inputs_to_check:
                input_name = V.graph.graph_input_names[idx]
                assert input_name in V.graph.graph_inputs, (
                    f"{input_name} not found in graph inputs"
                )
                value = V.graph.graph_inputs[input_name]
                assert isinstance(value, TensorBox), (
                    f"{input_name} is expected to be tensor but found as {type(value)}"
                )
                warn_msg = (
                    f"Input {idx} was compiled as {GPU_ALIGN_BYTES}-bytes aligned, "
                    "but it is not aligned at run time. Copying to an aligned tensor "
                    "to guarantee correctness, but expect a performance hit."
                )
                self.prefix.splice(
                    f"""
                    if ((reinterpret_cast<std::uintptr_t>({input_name}.data_ptr()) & ({GPU_ALIGN_BYTES} -1)) != 0) {{
                        AOTI_TORCH_WARN("{warn_msg}");
                        AtenTensorHandle {input_name}_aligned;
                        aoti_torch_clone_preserve_strides({input_name}, &{input_name}_aligned);
                        {input_name} = std::move(RAIIAtenTensorHandle({input_name}_aligned));
                    }}
                    """
                )

        super().codegen_inputs()

    def _define_kernel_helper(
````
- **EN**: Introduces function `_define_kernel_helper`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_name`, `value`, and `warn_msg`.
- **CN**: 这里定义了函数`_define_kernel_helper`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_name`、`value`、`warn_msg` 等值。

### Lines 897-924 / 第 897-924 行
````python
        self,
        kernel_name: str,
        kernel_body: str,
        metadata: str | None = None,
        gpu: bool = True,
        cpp_definition: str | None = None,
    ):
        if gpu:
            self._kernel_name_to_body[kernel_name] = kernel_body
            if config.triton.autotune_at_compile_time:
                # Call PythonWrapperCodegen to create the autotune code block
                PythonWrapperCodegen._define_kernel_helper(
                    self, kernel_name, kernel_body, metadata, gpu, cpp_definition
                )
        else:
            return CppWrapperCpu._define_kernel_helper(
                self, kernel_name, kernel_body, metadata, gpu, cpp_definition
            )

    def generate(self, is_inference):
        with dynamo_timed("CppWrapperGpu.generate", log_pt2_compile_event=True):
            return super().generate(is_inference)

    def finalize_prefix(self):
        """Define the triton kernels now that autotuning is finished"""
        old_prefix = self.prefix  # new content should go at start of prefix

        # Generating triton kernel callers can modify the prefix (cached dtypes),
````
- **EN**: Introduces function `generate`, function `finalize_prefix`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`generate`、函数`finalize_prefix`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 925-952 / 第 925-952 行
````python
        # so do this before running finalize_prefix(), but put the generated code
        # after the finalize_prefix() code.
        self.prefix = IndentedBuffer()
        for kernel in self._triton_call_wrappers.values():
            self.prefix.writeline("\n")
            kernel.generate(self)

        # Generate parallel kernel compilation initialization function
        if self._lazy_kernel_names:
            start_compile_calls = "\n    ".join(
                f'startKernelCompile(_module_pending_kernels, "{name}", {name}_source);'
                for name in self._lazy_kernel_names
            )
            self.prefix.splice(
                f"""\
// Start parallel compilation of all Triton kernels
static inline void start_all_triton_kernel_compiles() {{
    loadLazyCompileFuncs();
    _module_pending_kernels = PyDict_New();
    AOTI_TORCH_CHECK(_module_pending_kernels, "Failed to create pending kernels dict");
    {start_compile_calls}
}}

// Static initializer to start kernel compilation on module load
static struct TritonKernelCompileInit {{
    TritonKernelCompileInit() {{
        start_all_triton_kernel_compiles();
    }}
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `start_compile_calls`, and `_module_pending_kernels`. This range continues the implementation of function `CppWrapperGpu.finalize_prefix`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `start_compile_calls`、`_module_pending_kernels` 等值。这一段延续了函数`CppWrapperGpu.finalize_prefix` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
}} __triton_kernel_compile_init;
"""
            )

        triton_prefix = self.prefix

        self.prefix = IndentedBuffer()
        super().finalize_prefix()

        self.prefix.splice(triton_prefix)

        self.prefix.writeline("\n")
        self.prefix.splice(old_prefix)

    def generate_tma_descriptor(self, desc):
        self.write_tma_descriptor_helpers_once()

        if isinstance(desc, TMADescriptorExperimental):
            self._generate_experimental_tma_descriptor(desc)
        else:
            assert isinstance(desc, TMADescriptorStable)
            self._generate_stable_tma_descriptor(desc)

    def _generate_experimental_tma_descriptor(self, desc):
        # generate data pointer for the source tensor
        source = self.generate_args_decl(
            code=self,
            call_args=[self.val_to_arg_str(desc.tensor)],
````
- **EN**: Introduces function `generate_tma_descriptor`, function `_generate_experimental_tma_descriptor`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `triton_prefix`, `else`, `source`, `code`, and `call_args`.
- **CN**: 这里定义了函数`generate_tma_descriptor`、函数`_generate_experimental_tma_descriptor`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `triton_prefix`、`else`、`source`、`code`、`call_args` 等值。

### Lines 981-1008 / 第 981-1008 行
````python
            arg_types=[desc.tensor.get_dtype()],
            arg_signatures=[None],
            # these args are passed to initNDTMADescriptor, which is NOT a triton kernel
            is_triton_kernel=False,
        )

        desc_name = desc.name
        self.writeline(f"alignas(64) CUtensorMap {desc_name};")

        # `source` is in the form of `&var_x`, where `var_x` is the data pointer
        # (CUdeviceptr); we dereference `source` and cast to `void*` to pass to
        # the data pointer of the source tensor to the helper function
        # `init{1,2}DTMADescriptor`
        ptr = f"reinterpret_cast<void*>(*({source}))"
        dims = ", ".join(self.val_to_arg_str(dim) for dim in desc.dims)
        block_dims = ", ".join(self.val_to_arg_str(dim) for dim in desc.block_dims)
        element_size = self.val_to_arg_str(desc.element_size)
        fn = f"init{desc.rank}DTMADescriptor"
        args = f"&{desc_name}, {ptr}, {dims}, {block_dims}, {element_size}"
        self.writeline(f"{fn}({args});")

    def _generate_stable_tma_descriptor(self, desc):
        source = self.generate_args_decl(
            code=self,
            call_args=[self.val_to_arg_str(desc.tensor)],
            arg_types=[desc.tensor.get_dtype()],
            arg_signatures=[None],
            # these args are passed to initNDTMADescriptor, which is NOT a triton kernel
````
- **EN**: Introduces function `_generate_stable_tma_descriptor`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Initializes or updates values such as `arg_types`, `arg_signatures`, `is_triton_kernel`, `desc_name`, `ptr`, `dims`, and `...+7`.
- **CN**: 这里定义了函数`_generate_stable_tma_descriptor`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。初始化或更新了 `arg_types`、`arg_signatures`、`is_triton_kernel`、`desc_name`、`ptr`、`dims`、`另有7项` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
            is_triton_kernel=False,
        )

        desc_name = desc.name
        # Pack the relevant information into a StableTMADescriptor struct.
        # See [Note: AOTI TMA Stable handling] for more details.
        self.writeline(f"alignas(64) StableTMADescriptor {desc_name};")

        def fill_array(name, values):
            for i, val in enumerate(values):
                self.writeline(f"{name}[{i}] = {val};")

        ptr = f"reinterpret_cast<void*>(*({source}))"
        rank = len(desc.tensor.get_size())

        fill_array(f"{desc_name}.block_shape", desc.block_shape)
        fill_array(f"{desc_name}.global_shape", desc.tensor.get_size())
        fill_array(f"{desc_name}.strides", desc.tensor.get_stride())

        element_size = self.val_to_arg_str(desc.tensor.get_dtype().itemsize)
        fn = "initTMADescriptor"
        args = ", ".join(
            str(x)
            for x in [
                f"&{desc_name}.m",
                ptr,
                element_size,
                rank,
````
- **EN**: Introduces function `fill_array`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_triton_kernel`, `desc_name`, `ptr`, `rank`, `element_size`, `fn`, and `...+1`.
- **CN**: 这里定义了函数`fill_array`。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_triton_kernel`、`desc_name`、`ptr`、`rank`、`element_size`、`fn`、`另有1项` 等值。

### Lines 1037-1064 / 第 1037-1064 行
````python
                f"{desc_name}.block_shape",
                f"{desc_name}.global_shape",
                f"{desc_name}.strides",
            ]
        )
        self.writeline(f"{fn}({args});")

    def generate_args_decl(
        self,
        code: IndentedBuffer | Self,
        call_args,
        arg_types,
        arg_signatures,
        is_triton_kernel=True,
        scratch_spaces: dict[str, int] | None = None,
    ):
        """
        Generates any declarations of args to pass into a kernel call, and then returns the arg names.

        In more detail:
        * declarations: e.g. this function has a side effect of generating lines like `auto var_0 = ...;`
        * returns: a string with the list of args, e.g. "var_0, var_1"

        call_args: list of call arguments
        arg_types: list of argument types
        arg_signatures: list with signatures of all the args
        is_triton_kernel: whether these are passed into a triton kernel or not. In particular,
                          calls to triton kernels will have an additional global scratch space
````
- **EN**: Introduces function `generate_args_decl`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Initializes or updates values such as `code`, `is_triton_kernel`, `scratch_spaces`, `call_args`, `arg_types`, and `arg_signatures`.
- **CN**: 这里定义了函数`generate_args_decl`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。初始化或更新了 `code`、`is_triton_kernel`、`scratch_spaces`、`call_args`、`arg_types`、`arg_signatures` 等值。

### Lines 1065-1092 / 第 1065-1092 行
````python
                          arg injected at the front of the arg list.
        """
        new_args: list[str] = []

        def process_tma_stable_arg(arg, arg_type, arg_signature, var_name):
            # [Note: AOTI TMA Stable handling]
            # For most args, a single arg passed to the python triton interface
            # maps to a single arg in the cubin interface. However, for host-side
            # TMA descriptors, a single python arg turns into 1 + 2 * N args in the
            # cubin interface (where N is the rank).
            #
            # To do this: at TMA codegen time (for aoti), we generate a struct
            # (StableTMADescriptor) containing the necessary information; and then
            # when we call the function (i.e. here), we unpack the struct members.
            code.writeline(f"auto {var_name} = {cexpr(arg)};")
            return _unpack_tma_descriptor_args(var_name, arg_signature)

        def process_args(arg, arg_type, arg_signature=None):
            var_name = f"var_{next(self.arg_var_id)}"
            # ignore tma descriptors, as host-side TMA descriptors need
            # to be passed to the compiled Triton kernel by value
            if isinstance(arg_type, UnwrapUnspecArg) and not signature_is_tma_desc(
                arg_signature
            ):
                self.codegen_tensor_item(
                    arg_type.dtype,
                    arg,
                    var_name,
````
- **EN**: Introduces function `process_tma_stable_arg`, function `process_args`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`process_tma_stable_arg`、函数`process_args`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1093-1120 / 第 1093-1120 行
````python
                    indented_buffer=code,
                )
                new_args.append(f"&{var_name}")
            elif isinstance(arg_type, torch_dtype) and not signature_is_tma_desc(
                arg_signature
            ):
                device_ptr_type = self.device_codegen.cpp_device_ptr()
                code.writeline(
                    maybe_hipify_code_wrapper(
                        f"{device_ptr_type} {var_name} = reinterpret_cast<{device_ptr_type}>({arg}.data_ptr());"
                    )
                )
                new_args.append(f"&{var_name}")
            # For symbolic call arguments, examine the arg signatures from triton meta
            # to explicitly cast to the right type
            # Reason: `auto` can infer unexpected type against kernel input signature.
            elif (
                isinstance(arg_type, type(SymbolicCallArg))
                and arg_signature is not None
                and arg_signature in TRITON_SIGNATURE_TO_CPP
            ):
                code.writeline(
                    f"{TRITON_SIGNATURE_TO_CPP[arg_signature]} {var_name} = {cexpr(arg)};"
                )
                new_args.append(f"&{var_name}")
            elif arg_type in (sympy.Integer, int):
                code.writeline(f"int {var_name} = {cexpr(arg)};")
                new_args.append(f"&{var_name}")
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `indented_buffer`, and `device_ptr_type`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `indented_buffer`、`device_ptr_type` 等值。

### Lines 1121-1148 / 第 1121-1148 行
````python
            elif arg_type in (sympy.Float, float):
                # Use signature type if available, otherwise default to float
                cpp_type = TRITON_SIGNATURE_TO_CPP.get(  # pyrefly: ignore[no-matching-overload]
                    arg_signature, "float"
                )
                code.writeline(f"{cpp_type} {var_name} = {cexpr(arg)};")
                new_args.append(f"&{var_name}")
            elif arg_signature and arg_signature.startswith("tensordesc<"):
                new_args.extend(
                    process_tma_stable_arg(arg, arg_type, arg_signature, var_name)
                )
            else:
                code.writeline(f"auto {var_name} = {cexpr(arg)};")
                new_args.append(f"&{var_name}")

        for arg, arg_type, arg_signature in zip_longest(
            call_args, arg_types, arg_signatures
        ):
            process_args(arg, arg_type, arg_signature)

        for scratch_name, workspace_size in (scratch_spaces or {}).items():
            if (
                is_triton_kernel
                and (
                    scratch := self.device_codegen.cpp_scratch(
                        next(self.arg_var_id),
                        workspace=TritonScratchWorkspace(
                            size=workspace_size,
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cpp_type`, `else`, `scratch`, `workspace`, and `size`. This range continues the implementation of function `CppWrapperGpu.generate_args_decl.process_args`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `cpp_type`、`else`、`scratch`、`workspace`、`size` 等值。这一段延续了函数`CppWrapperGpu.generate_args_decl.process_args` 的具体实现。

### Lines 1149-1176 / 第 1149-1176 行
````python
                            generate_dtype_str=(
                                lambda: self.codegen_dtype(torch.uint8)
                            ),
                        ),
                        prefix=scratch_name,
                    )
                )
                is not None
            ):
                scratch_def, scratch_var = scratch
                code.writelines([maybe_hipify_code_wrapper(x) for x in scratch_def])
                new_args.append(f"&{scratch_var}")

        return ", ".join(new_args)

    def _generate_kernel_call_helper(
        self,
        kernel_name: str,
        call_args,
        *,
        device=None,
        triton=True,
        arg_types=None,
        raw_keys=None,
        raw_args=None,
        triton_meta=None,
        inductor_meta=None,
        graph_name="",
````
- **EN**: Introduces function `_generate_kernel_call_helper`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_generate_kernel_call_helper`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1177-1204 / 第 1177-1204 行
````python
        original_fxnode_name=None,
        current_stream_idx=None,
    ):
        """
        Override the default value of argument 'gpu' to True here.
        generate_kernel_call can still be called with gpu=False because of
        a mix of cpu kernels and gpu kernels.
        """
        device = device or V.graph.get_current_device_or_throw()
        if device.type == "cpu":
            # Even in CppWrapperGpu, we may see cpp kernels
            return CppWrapperCpu._generate_kernel_call_helper(
                self,
                kernel_name,
                call_args,
                device=device,
                triton=triton,
                arg_types=arg_types,
                raw_keys=raw_keys,
                raw_args=raw_args,
                triton_meta=triton_meta,
                inductor_meta=inductor_meta,
            )

        if (
            triton
            and config.triton.autotune_at_compile_time
            and kernel_name not in self.kernel_autotune_names
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1205-1232 / 第 1205-1232 行
````python
        ):
            # Call PythonWrapperCodegen to create the autotune code block
            PythonWrapperCodegen._generate_kernel_call_helper(
                self,
                kernel_name,
                call_args,
                device=device,
                triton=triton,
                arg_types=arg_types,
                raw_keys=raw_keys,
                raw_args=raw_args,
                triton_meta=triton_meta,
                inductor_meta=inductor_meta,
                original_fxnode_name=original_fxnode_name,
            )

        stream = (
            "stream"
            if V.graph.aot_mode
            else self.write_get_raw_stream(device.index, graph_name)
        )

        if triton:
            call_args, arg_types = self.prepare_triton_wrapper_args(
                call_args,
                # pyrefly: ignore [bad-argument-type]
                arg_types,
            )
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `device`, `triton`, `arg_types`, `raw_keys`, `raw_args`, `triton_meta`, and `...+3`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `device`、`triton`、`arg_types`、`raw_keys`、`raw_args`、`triton_meta`、`另有3项` 等值。

### Lines 1233-1260 / 第 1233-1260 行
````python

            # For lazy compile mode with TMA, extract underlying tensor names
            tma_tensor_args: dict[str, str] = {}
            is_lazy_compile = (
                not V.graph.aot_mode and config.triton.autotune_at_compile_time is False
            )
            if is_lazy_compile and raw_args and triton_meta:
                signature = triton_meta.get("signature", {})
                raw_keys_list = raw_keys or []
                for key, raw_arg in zip(raw_keys_list, raw_args):
                    sig_type = signature.get(key, "")
                    if isinstance(sig_type, str) and signature_is_tma_desc(sig_type):
                        if isinstance(raw_arg, TMADescriptorStable):
                            # Get the underlying tensor name
                            tensor_name = raw_arg.get_tensor().get_name()
                            tma_tensor_args[key] = tensor_name
                        else:
                            raise AssertionError("Unsupported TMA descriptor type")

            wrapper_name = f"call_{kernel_name}"
            if wrapper_name not in self._triton_call_wrappers:
                self._triton_call_wrappers[wrapper_name] = DeferredTritonCallWrapper(
                    wrapper_name,
                    kernel_name,
                    self._kernel_name_to_body,
                    arg_types,
                    triton_meta=triton_meta,
                    inductor_meta=inductor_meta,
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tma_tensor_args`, `is_lazy_compile`, `signature`, `raw_keys_list`, `sig_type`, `tensor_name`, and `...+4`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tma_tensor_args`、`is_lazy_compile`、`signature`、`raw_keys_list`、`sig_type`、`tensor_name`、`另有4项` 等值。

### Lines 1261-1288 / 第 1261-1288 行
````python
                    tma_tensor_args=tma_tensor_args,
                )

            # For TMA in lazy compile mode, add tensor args to the call
            if is_lazy_compile and tma_tensor_args:
                for tensor_name in tma_tensor_args.values():
                    call_args.append(tensor_name)
                    arg_types.append(
                        torch.float32
                    )  # dtype doesn't matter, just need tensor type

            device_idx = "this->device_idx_" if V.graph.aot_mode else str(device.index)
            call_args.append(device_idx)
            call_args.append(stream)
            if V.graph.aot_mode:
                call_args.append("kernels")
                call_args.append("this->cubin_dir_")
            debug_printer_manager = V.graph.wrapper_code.debug_printer
            debug_printer_manager.set_printer_args(
                call_args[: len(arg_types)], kernel_name, arg_types, None
            )
            with debug_printer_manager:
                self.writeline(f"{wrapper_name}({', '.join(call_args)});")
        else:
            casted = []
            # pyrefly: ignore [bad-argument-type, no-matching-overload]
            for arg_type, arg in zip(arg_types, call_args):
                new_arg = arg
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tma_tensor_args`, `device_idx`, `debug_printer_manager`, `else`, `casted`, and `new_arg`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `tma_tensor_args`、`device_idx`、`debug_printer_manager`、`else`、`casted`、`new_arg` 等值。

### Lines 1289-1316 / 第 1289-1316 行
````python
                if arg_type.endswith("*") and arg != "nullptr":
                    new_arg = f"{arg}.data_ptr()"
                # pyrefly: ignore [bad-argument-type]
                casted.append(f"({arg_type}){cexpr(new_arg)}")
            call_args_str = ", ".join(casted)
            self.writeline(f"kernels.{kernel_name}({call_args_str}, {stream});")

    def prepare_triton_wrapper_args(
        self, call_args: list[Any], arg_types: list[Any]
    ) -> tuple[list[Any], list[Any]]:
        assert len(call_args) == len(arg_types), (call_args, arg_types)
        new_args = []
        new_args_types = []
        for arg, arg_type in zip(call_args, arg_types):
            if isinstance(arg, str):
                if isinstance(arg_type, torch_dtype) and should_unwrap_unspec_arg(arg):
                    # dynamo wraps unspec variable as 0d CPU tensor, need convert to scalar
                    arg_type = UnwrapUnspecArg(dtype=arg_type)
                new_args.append(arg)
            elif isinstance(arg, bool):
                new_args.append(str(arg).lower())
            elif isinstance(arg, (int, float, SymbolicCallArg)):
                if isinstance(arg, float):
                    new_args.append(self.generate_float_value(arg))
                else:
                    new_args.append(str(arg))
            else:
                new_args.append(cexpr(V.graph.sizevars.simplify(arg)))
````
- **EN**: Introduces function `prepare_triton_wrapper_args`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`prepare_triton_wrapper_args`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1317-1328 / 第 1317-1328 行
````python
            new_args_types.append(arg_type)
        return new_args, new_args_types

    def make_zero_buffer(self, name):
        return f"AOTI_TORCH_ERROR_CODE_CHECK(aoti_torch_zero_({name}.get()));"


@dataclasses.dataclass
class UnwrapUnspecArg:
    """Marker that we need to call .item() on the tensor"""

    dtype: torch_dtype
````
- **EN**: Introduces function `make_zero_buffer`, class `UnwrapUnspecArg`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`make_zero_buffer`、类`UnwrapUnspecArg`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `re`, `sys`, `itertools`, `typing`
- **Third-party / 第三方**: `typing_extensions`, `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.codecache`, `torch._inductor.runtime.runtime_utils`, `torch.utils._ordered_set`, `..`, `..codecache`, `..ir`, `..utils`, `..virtualized`, `.aoti_hipify_utils`, `.common`, `.cpp_utils`, `.cpp_wrapper_cpu`, `.multi_kernel`, `.triton_utils`, `.wrapper`, `..runtime.triton_heuristics`
