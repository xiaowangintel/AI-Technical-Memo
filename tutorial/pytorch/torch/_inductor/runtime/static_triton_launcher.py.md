# static_triton_launcher.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/runtime/static_triton_launcher.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module provides runtime helpers used after compilation. It defines classes such as `StaticallyLaunchedTritonKernel`, `StaticallyLaunchedCudaKernel`, and `StaticallyLaunchedXpuKernel`. It exposes functions such as `statically_launched_kernel_by_device`.
- **用途（中文）**: 该模块提供编译完成后使用的运行期辅助逻辑。其中定义了 `StaticallyLaunchedTritonKernel`、`StaticallyLaunchedCudaKernel`、`StaticallyLaunchedXpuKernel` 等类。同时提供 `statically_launched_kernel_by_device` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import functools
import os
from functools import cached_property
from typing import Any
from typing_extensions import Unpack

from ..utils import is_rocm
from .triton_compat import ASTSource, CompiledKernel, knobs as triton_knobs
from .triton_helpers import get_constexprs


class StaticallyLaunchedTritonKernel:
    """
    Parses the metadata of a CompiledKernel from Triton into a structure that can
    launch the cuda kernel directly. Only works for triton kernels compiled to cubin.

    Doing this avoids C++ codegen and compilation during compile, since we can use a
    statically compiled library to launch the kernel. To avoid mallocing for the arguments,
    we have a launcher for different numbers of arguments up to a max. StaticCudaLauncher
    only supports # of arguments up until 10 for now.
````
- **EN**: Imports dependencies such as `functools`, `os`, `typing`, `typing_extensions`, `..utils`, `.triton_compat`, and `...+1` for the logic in this range. Introduces class `StaticallyLaunchedTritonKernel`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `functools`、`os`、`typing`、`typing_extensions`、`..utils`、`.triton_compat`、`另有1项` 等依赖，为后续逻辑提供基础能力。这里定义了类`StaticallyLaunchedTritonKernel`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 21-40 / 第 21-40 行
````python

    Workflow:
    Compile time:
    1. Compile a kernel with triton and get a CompiledKernel
    2. Instantiate kernel = StaticallyLaunchedTritonKernel(triton_kernel)
    3. Write to a cubin file: kernel.write_cubin_to_file(filepath)
    4. Call kernel.load_kernel() (CUDA should be initialized by this point) to load the cubin
    Runtime:
    5. Call kernel.run(grid, stream, args) to launch the kernel

    Note that after step 3, StaticallyLaunchedTritonKernel is fully pickleable/serializable.
    This allows it to be cached by FXGraphCache/TritonBundler, as well as sent from the worker
    to the parent process in inductor.

    There are two main versions of triton that we wish to support: 3.3 and 3.2. Triton makes considerable changes
    to how it handles constants in 3.3, so there's some special logic necessary to handle both versions.
    """

    @cached_property
    def C_impl(self):
````
- **EN**: Introduces function `C_impl`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了函数`C_impl`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 41-60 / 第 41-60 行
````python
        raise NotImplementedError

    def __init__(self, kernel: CompiledKernel) -> None:
        # pyrefly: ignore [missing-attribute]
        self.name = kernel.src.fn.__name__
        # pyrefly: ignore [missing-attribute]
        self.cubin_path = kernel._cubin_path

        # Used by torch.compile to filter constants in older triton versions
        # pyrefly: ignore [missing-attribute]
        self.arg_names = kernel.src.fn.arg_names

        # Const exprs that are declared by the triton kernel directly
        # Used to generate the kernel launcher's def args
        # pyrefly: ignore [missing-attribute]
        self.declared_constexprs = get_constexprs(kernel.src.fn)

        # pyrefly: ignore [missing-attribute]
        self.hash = kernel.hash

````
- **EN**: Introduces function `__init__`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`__init__`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python
        if triton_knobs is None:
            # pyrefly: ignore [missing-attribute]
            launch_enter = kernel.__class__.launch_enter_hook
            # pyrefly: ignore [missing-attribute]
            launch_exit = kernel.__class__.launch_exit_hook
        else:
            launch_enter = triton_knobs.runtime.launch_enter_hook
            launch_exit = triton_knobs.runtime.launch_exit_hook

        def hook_is_empty(hook: Any) -> bool:
            if hook is None:
                return True
            if (
                triton_knobs
                and (HookChain := getattr(triton_knobs, "HookChain", None)) is not None
                and isinstance(hook, HookChain)
            ):
                # Support hooks after https://github.com/triton-lang/triton/pull/7866
                return len(hook.calls) == 0
            return False
````
- **EN**: Introduces function `hook_is_empty`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`hook_is_empty`。保留了指向设计说明、规范或厂商数据手册的注释引用。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 81-100 / 第 81-100 行
````python

        if not hook_is_empty(launch_enter) or not hook_is_empty(launch_exit):
            raise NotImplementedError(
                "We don't support launch enter or launch exit hooks"
            )
        # pyrefly: ignore [missing-attribute]
        self.num_warps = kernel.metadata.num_warps
        self.shared = (
            # pyrefly: ignore [missing-attribute]
            kernel.shared if hasattr(kernel, "shared") else kernel.metadata.shared
        )

        def needs_scratch_arg(scratch_name: str, param_name: str) -> bool:
            # pyrefly: ignore [missing-attribute]
            if hasattr(kernel.metadata, param_name):
                # pyrefly: ignore [missing-attribute]
                if getattr(kernel.metadata, param_name) > 0:
                    raise NotImplementedError(
                        f"{scratch_name} scratch not yet supported"
                    )
````
- **EN**: Introduces function `needs_scratch_arg`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`needs_scratch_arg`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python
                return True
            return False

        # Newer triton versions pass an extra global scratch parameter to the compiled cuda kernel.
        # Inductor never uses this field or enables it, but we still have to pass
        # an extra None into the set of params if its enabled
        self.has_global_scratch = needs_scratch_arg("Global", "global_scratch_size")
        # same situation for profile scratch - triton-lang/triton#7258
        self.has_profile_scratch = needs_scratch_arg("Profile", "profile_scratch_size")

        # pyrefly: ignore [missing-attribute]
        self.arg_tys = self.arg_ty_from_signature(kernel.src)
        self.function: int | None = None  # Loaded by load_kernel(on the parent process)
        num_ctas = 1
        if hasattr(kernel, "num_ctas"):
            num_ctas = kernel.num_ctas
        elif hasattr(kernel, "metadata"):
            num_ctas = kernel.metadata.num_ctas

        if num_ctas != 1:
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python
            raise NotImplementedError(
                "Static cuda launcher only supports num_ctas == 1"
            )

    def reload_cubin_from_raw(self, filepath: str) -> str:
        """
        If the cubin file triton generated gets deleted under us, we can
        reload it from the raw cubin file.
        """
        if self.cubin_path is None:
            assert self.cubin_raw is not None
            os.makedirs(os.path.dirname(filepath), exist_ok=True)
            with open(filepath, "wb") as f:
                f.write(self.cubin_raw)
                self.cubin_path = filepath  # pyre-ignore
        return self.cubin_path

    def load_kernel(self, device: int) -> None:
        if self.function is not None:
            return
````
- **EN**: Introduces function `reload_cubin_from_raw`, function `load_kernel`. Touches the filesystem to load, validate, or store compiler artifacts. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`reload_cubin_from_raw`、函数`load_kernel`。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 141-160 / 第 141-160 行
````python

        assert hasattr(self, "cubin_path")
        assert self.cubin_path is not None
        (self.function, self.n_regs, self.n_spills) = self.C_impl._load_kernel(
            self.cubin_path, self.name, self.shared, device
        )
        # Don't need the cubin path anymore now that we've loaded
        self.cubin_path = None
        self.cubin_raw = None

    @staticmethod
    @functools.lru_cache
    def type_mappings() -> dict[str, str]:
        return {
            "i1": "i",
            "i8": "b",
            "i16": "h",
            "i32": "i",
            "i64": "l",
            "u1": "I",
````
- **EN**: Introduces function `type_mappings`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`type_mappings`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 161-180 / 第 161-180 行
````python
            "u8": "B",
            "u16": "H",
            "u32": "I",
            "u64": "K",
            "fp16": "f",
            "bf16": "f",
            "fp32": "f",
            "f32": "f",
            "fp64": "d",
            # TODO handle nvTmaDesc/CUtensormap
        }

    def extract_type(self, ty: str) -> str:
        """
        Takes a triton type from CompiledKernel.signature and
        converts it into a single char encoding. _StaticCudaLauncher
        will switch on this char to figure out what type the underlying
        value should be passed to the triton kernel as.
        """
        if ty[0] == "*":
````
- **EN**: Introduces function `extract_type`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`extract_type`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python
            return "O"
        elif ty == "nvTmaDesc":
            raise NotImplementedError("nvTmaDesc kernels are not yet supported")
        return StaticallyLaunchedTritonKernel.type_mappings()[ty]

    def arg_ty_from_signature(self, src: ASTSource) -> str:
        def index_key(i: Any) -> int:
            if isinstance(i, str):
                # pyrefly: ignore [missing-attribute]
                return src.fn.arg_names.index(i)
            elif isinstance(i, tuple):
                # In triton 3.3, src.fn.constants has tuples as a key
                return i[0]
            else:
                return i

        # pyrefly: ignore [missing-attribute]
        signature = {index_key(key): value for key, value in src.signature.items()}
        # Triton uses these as the main way to filter out constants passed to their cubin
        constants = [index_key(key) for key in getattr(src, "constants", dict())]
````
- **EN**: Introduces function `arg_ty_from_signature`, function `index_key`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`arg_ty_from_signature`、函数`index_key`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 201-220 / 第 201-220 行
````python
        # This value is always a superset of kernel.fn.constexprs: kernel.fn.constexprs are
        # constants declared by the triton kernel directly, whereas this list can have
        # constants that are unused by the triton kernel that triton figured out during
        # compilation.
        self.full_constexprs = constants
        # Despite requiring them to be passed in, the triton CUDA launcher
        # completely ignores the constexprs passed into it when generating code.
        # So we can ignore them here too
        params = []

        for i in sorted(signature.keys()):
            ty = signature[i]
            # In newer triton versions, constants are passed in to signature with type `constexpr`
            # In older triton versions, there can be constants in src.constants that are not `constexpr` in signature
            # so we check both here
            if ty == "constexpr" or i in constants:
                pass
            else:
                # pyrefly: ignore [bad-argument-type]
                params.append(self.extract_type(ty))
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `params`, `ty`, and `else`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `params`、`ty`、`else` 等值。

### Lines 221-240 / 第 221-240 行
````python
        return "".join(params)

    def __getstate__(self) -> dict[str, Any]:
        # Remove objects that are no longer valid for pickling
        state = self.__dict__.copy()
        state["function"] = None
        # Cubin paths aren't consistent across processes, so we clear
        # and reload them.
        state["cubin_path"] = None
        return state

    def run(
        self,
        grid_x: int,
        grid_y: int,
        grid_z: int,
        stream: int,
        *args: Unpack[tuple[object, ...]],
    ) -> None:
        """Actually run the kernel at runtime. This function is the hot codepath."""
````
- **EN**: Introduces function `__getstate__`, function `run`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `state`, `grid_x`, `grid_y`, `grid_z`, and `stream`.
- **CN**: 这里定义了函数`__getstate__`、函数`run`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `state`、`grid_x`、`grid_y`、`grid_z`、`stream` 等值。

### Lines 241-260 / 第 241-260 行
````python

        # Assert load_kernel() has been called and args match
        assert self.function is not None

        # TODO: actually, if the args *don't* match, we probably should
        # throw an exception. But if inductor is the only one calling this
        # thing, it should always match.
        # Get rid of constants before passing to cubin launcher

        arg_tys = self.arg_tys

        if is_rocm():
            # ROCm/HIP kernel ABI: The Triton HIP backend ALWAYS includes both
            # global_scratch and profile_scratch parameters in the kernel signature,
            # even when the kernel doesn't use them (i.e., when has_*_scratch is False).
            #
            # This differs fundamentally from CUDA, where these parameters are only
            # present in the signature if the corresponding has_*_scratch flag is True.
            #
            # The flags indicate whether memory will be allocated/used:
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 261-280 / 第 261-280 行
````python
            # - has_global_scratch: Whether global scratch workspace is needed
            # - has_profile_scratch: Whether profiling instrumentation is enabled
            #
            # However, regardless of flag values, we MUST always pass both parameters
            # to match the HIP kernel ABI. Passing None is safe:
            #
            # - If scratch is not needed (has_*_scratch=False or scratch_size=0):
            #   The None becomes nullptr, which the kernel never dereferences
            #
            # - If scratch is needed (has_*_scratch=True and scratch_size>0):
            #   The None becomes nullptr initially, but the HIP runtime intercepts
            #   the kernel launch, allocates the required scratch memory based on
            #   kernel metadata, and replaces the nullptr with a valid pointer before
            #   the kernel actually executes
            #
            # Not passing both parameters causes segmentation faults because the kernel
            # expects them at specific positions in the argument array.
            arg_tys = arg_tys + "OO"
            args = (*args, None, None)

````
- **EN**: Initializes or updates values such as `arg_tys`, and `args`. This range continues the implementation of function `StaticallyLaunchedTritonKernel.run`.
- **CN**: 初始化或更新了 `arg_tys`、`args` 等值。这一段延续了函数`StaticallyLaunchedTritonKernel.run` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
        else:
            for has_scratch in [self.has_global_scratch, self.has_profile_scratch]:
                if has_scratch:
                    arg_tys = arg_tys + "O"
                    args = (*args, None)
        # pyrefly: ignore [bad-argument-type]
        assert len(args) == len(arg_tys)

        # TODO: can handle grid functions here or in C++, so
        # that we don't need the grid handler above.
        self.C_impl._launch_kernel(
            self.function,
            grid_x,
            grid_y,
            grid_z,
            self.num_warps,
            self.shared,
            arg_tys,
            args,
            stream,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `arg_tys`, and `args`. This range continues the implementation of function `StaticallyLaunchedTritonKernel.run`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`arg_tys`、`args` 等值。这一段延续了函数`StaticallyLaunchedTritonKernel.run` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
        )


class StaticallyLaunchedCudaKernel(StaticallyLaunchedTritonKernel):
    @cached_property
    def C_impl(self):
        from torch._C import _StaticCudaLauncher

        return _StaticCudaLauncher

    def __init__(self, kernel: CompiledKernel) -> None:
        # pyrefly: ignore [missing-attribute]
        if "hsaco" in kernel.asm:
            # pyrefly: ignore [missing-attribute]
            self.cubin_raw = kernel.asm["hsaco"]

        # pyrefly: ignore [missing-attribute]
        elif "cubin" in kernel.asm:
            # pyrefly: ignore [missing-attribute]
            self.cubin_raw = kernel.asm["cubin"]
````
- **EN**: Imports dependencies such as `torch._C` for the logic in this range. Introduces class `StaticallyLaunchedCudaKernel`, function `C_impl`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `torch._C` 等依赖，为后续逻辑提供基础能力。这里定义了类`StaticallyLaunchedCudaKernel`、函数`C_impl`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 321-340 / 第 321-340 行
````python
        else:
            raise RuntimeError(
                "Expected either 'hsaco' (ROCm) or 'cubin' (CUDA) in kernel.asm"
            )
        super().__init__(kernel)


class StaticallyLaunchedXpuKernel(StaticallyLaunchedTritonKernel):
    @cached_property
    def C_impl(self):
        from torch._C import _StaticXpuLauncher

        return _StaticXpuLauncher

    def __init__(self, kernel: CompiledKernel) -> None:
        # pyrefly: ignore [missing-attribute]
        self.cubin_raw = kernel.asm.get("zebin", None)
        super().__init__(kernel)


````
- **EN**: Imports dependencies such as `torch._C` for the logic in this range. Introduces class `StaticallyLaunchedXpuKernel`, function `C_impl`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `torch._C` 等依赖，为后续逻辑提供基础能力。这里定义了类`StaticallyLaunchedXpuKernel`、函数`C_impl`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 341-351 / 第 341-351 行
````python
def statically_launched_kernel_by_device(
    kernel: CompiledKernel, device_type: str = "cuda"
) -> StaticallyLaunchedTritonKernel:
    if device_type in ("cuda", "hip"):
        return StaticallyLaunchedCudaKernel(kernel)
    elif device_type == "xpu":
        return StaticallyLaunchedXpuKernel(kernel)
    else:
        raise NotImplementedError(
            f"Device type {device_type} is not supported for static launcher"
        )
````
- **EN**: Introduces function `statically_launched_kernel_by_device`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`statically_launched_kernel_by_device`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

## Key Concepts / 关键概念
- **EN**: Provides runtime helpers used after compilation  
  **CN**: 提供编译完成后使用的运行期辅助逻辑
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Primary classes: `StaticallyLaunchedTritonKernel`, `StaticallyLaunchedCudaKernel`, and `StaticallyLaunchedXpuKernel`  
  **CN**: 主要类：`StaticallyLaunchedTritonKernel`、`StaticallyLaunchedCudaKernel`、`StaticallyLaunchedXpuKernel`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `os`, `typing`
- **Third-party / 第三方**: `typing_extensions`
- **PyTorch/Internal / PyTorch 内部**: `..utils`, `.triton_compat`, `.triton_helpers`, `torch._C`
