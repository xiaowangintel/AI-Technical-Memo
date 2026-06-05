# multi_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/multi_kernel.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `MultiKernelState`, `MultiKernel`, `MultiKernelCall`, `SizeHintMultiKernel`, and `SizeHintMultiKernelCall`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `MultiKernelState`、`MultiKernel`、`MultiKernelCall`、`SizeHintMultiKernel`、`SizeHintMultiKernelCall` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import functools
import logging
import math
import os
import pathlib
from typing import Any

from torch._inductor.ir import MultiTemplateBuffer
from torch._inductor.metrics import get_metric_table, is_metric_table_enabled
from torch.utils._ordered_set import OrderedSet

from .. import config
from ..codecache import code_hash, CodeCacheFuture, get_path, write_atomic
from ..runtime.benchmarking import benchmarker
from ..utils import cache_on_self, IndentedBuffer
from ..virtualized import V
from .common import TensorArg, WorkspaceArg


````
- **EN**: Imports dependencies such as `functools`, `logging`, `math`, `os`, `pathlib`, `typing`, and `...+9` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `functools`、`logging`、`math`、`os`、`pathlib`、`typing`、`另有9项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 21-40 / 第 21-40 行
````python
log = logging.getLogger(__name__)


class MultiKernelState:
    """
    Maintain state of multi-kernel compilation so we don't define duplicated
    multi-kernel for the same set of sub-kernels.

    V.graph.wrapper_code has a reference to MultiKernelState instance.
    """

    def __init__(self):
        self.subkernel_to_kernel_name = {}
        self.kernel_defs = IndentedBuffer()

    def define_kernel(
        self,
        kernels: list[Any],
        kernel_shape_keys: list[None | tuple[tuple[int, ...], ...]] | None = None,
    ) -> str:
````
- **EN**: Introduces class `MultiKernelState`, function `__init__`, function `define_kernel`. Initializes or updates values such as `log`, `kernels`, and `kernel_shape_keys`.
- **CN**: 这里定义了类`MultiKernelState`、函数`__init__`、函数`define_kernel`。初始化或更新了 `log`、`kernels`、`kernel_shape_keys` 等值。

### Lines 41-60 / 第 41-60 行
````python
        """
        Previously we name the multi kernel as "multi_kernel_{kernel_names[0]}".
        This has some minor issue.

        E.g. for persistent reduction https://gist.github.com/shunting314/39e7c00ff8bb2055942ed5a3255d61ca ,
        there are 2 flavors of non-persistent reduction:
          https://gist.github.com/shunting314/056d43d35907e87efb883970b35c17d4
        and
          https://gist.github.com/shunting314/02ee753b65c513c54e695626afe682bd

        The only different is cache eviction policy.

        We should name the multi-kernel differently in these 2 cases.

        kernels:
            A list of kernels
        kernel_shape_keys:
            Specified for size-hint multi-kernels.
            Each list element is a shape key, corresponding to the concrete input & output size hints each kernel was tuned for.
        """
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Initializes or updates values such as `https`, `kernels`, and `kernel_shape_keys`. This range continues the implementation of function `MultiKernelState.define_kernel`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。初始化或更新了 `https`、`kernels`、`kernel_shape_keys` 等值。这一段延续了函数`MultiKernelState.define_kernel` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python
        # Prevent circular import
        from ..select_algorithm import TritonTemplateKernel

        kernel_names = tuple(k.kernel_name for k in kernels)
        if kernel_names in self.subkernel_to_kernel_name:
            return self.subkernel_to_kernel_name[kernel_names]

        # name the multi kernel based on the first kernel
        multi_kernel_name = f"multi_kernel_{len(self.subkernel_to_kernel_name)}"
        self.subkernel_to_kernel_name[kernel_names] = multi_kernel_name

        if V.graph.cpp_wrapper and not config.triton.autotune_at_compile_time:
            # we should not generate any python code for multi-kernel during
            # the second pass of cpp-wrapper.
            return multi_kernel_name

        arg_index: dict[int, list[slice]] = {}
        _, call_args, _, arg_types = kernels[0].args.python_argdefs()
        if isinstance(kernels[0], TritonTemplateKernel) and isinstance(
            kernels[0].output_node, MultiTemplateBuffer
````
- **EN**: Imports dependencies such as `..select_algorithm` for the logic in this range. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..select_algorithm` 等依赖，为后续逻辑提供基础能力。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
        ):
            for i, kernel in enumerate(kernels):
                additional_call_args, _ = kernel.additional_call_args_and_types()
                if i not in arg_index:
                    arg_index[i] = []
                arg_index[i].append(slice(0, len(call_args)))
                arg_index[i].append(
                    slice(
                        len(call_args) + i * len(additional_call_args),
                        len(call_args) + (i + 1) * len(additional_call_args),
                    )
                )
        else:
            kernels[0].add_numel_to_call_args(multi_kernel_name, call_args, arg_types)
            for i in range(len(kernels)):
                arg_index[i] = [slice(0, len(call_args))]

        keyed_by_sizes = kernel_shape_keys is not None
        buf = self.kernel_defs
        buf.writeline("")
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `keyed_by_sizes`, and `buf`. This range continues the implementation of function `MultiKernelState.define_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`keyed_by_sizes`、`buf` 等值。这一段延续了函数`MultiKernelState.define_kernel` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
        buf.writeline("arg_index = {")
        for key, slice_list in arg_index.items():
            slice_reprs = ", ".join(repr(s) for s in slice_list)
            buf.writeline(f"    {key}: [{slice_reprs}],")
        buf.writeline("}")

        if not keyed_by_sizes:  # no size hint keys, just call with list of kernels
            buf.writeline(
                f"{multi_kernel_name} = async_compile.multi_kernel({multi_kernel_name!r}, ["
            )
            with buf.indent():
                for name in kernel_names:
                    buf.writeline(f"{name},")
            buf.writeline("], arg_index=arg_index)")
        else:  # call with dict[size hint key, kernel]
            assert isinstance(kernels[0], TritonTemplateKernel)
            assert isinstance(kernel_shape_keys, list)
            assert len(kernels) == len(kernel_shape_keys)
            buf.writeline(
                f"{multi_kernel_name} = async_compile.size_hint_multi_kernel({multi_kernel_name!r}, {{"
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `slice_reprs`, and `else`. This range continues the implementation of function `MultiKernelState.define_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `slice_reprs`、`else` 等值。这一段延续了函数`MultiKernelState.define_kernel` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
            )
            with buf.indent():
                for shape_key, name in zip(kernel_shape_keys, kernel_names):
                    buf.writeline(f"{shape_key}: {name},")
            buf.writeline("}, arg_index=arg_index)")

        if config.triton.autotune_at_compile_time:
            V.graph.wrapper_code.src_to_kernel["\n".join(kernel_names)] = (
                multi_kernel_name
            )

        return multi_kernel_name


class MultiKernel:
    """
    This class maintains the compile time state for multi kernels.

    Assume we do codegen for a MultiKernel encapsulating kernel1 and kernel2.
    The generated definition for the multi-kernel will looks like:
````
- **EN**: Introduces class `MultiKernel`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`MultiKernel`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
    ```
    multi_kernel_kernel1 = MultiKernelCall(
        [kernel1, kernel2], multi_kernel_definition_code
    )
    ```

    Here is a concrete example: https://gist.github.com/shunting314/d9f3fb6bc6cee3dbae005825ca196d39
    """

    def __init__(self, kernels):
        assert len(kernels) >= 2

        self.kernels = kernels
        self.kernel_name = V.graph.wrapper_code.multi_kernel_state.define_kernel(
            kernels
        )

        # need this since some code in inductor check if the kernel object has an args
        # attribute to decide if it's a non-null kernel.
        self.args = object()
````
- **EN**: Introduces function `__init__`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `multi_kernel_kernel1`.
- **CN**: 这里定义了函数`__init__`。保留了指向设计说明、规范或厂商数据手册的注释引用。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `multi_kernel_kernel1` 等值。

### Lines 161-180 / 第 161-180 行
````python

    @staticmethod
    def _merge_workspace_args(left: list[WorkspaceArg], right: list[WorkspaceArg]):
        if left == right:
            return left
        result = {x.inner_name: x for x in left}
        for arg in right:
            if arg.inner_name in result:
                result[arg.inner_name] = WorkspaceArg.maximum(
                    result[arg.inner_name], arg
                )
            else:
                result[arg.inner_name] = arg
        return [*result.values()]

    @staticmethod
    def merge_workspaces_inplace(kernels):
        if len(kernels) < 2:
            return
        # All kernels must share the same workspace
````
- **EN**: Introduces function `_merge_workspace_args`, function `merge_workspaces_inplace`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_merge_workspace_args`、函数`merge_workspaces_inplace`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python
        workspace_args = functools.reduce(
            MultiKernel._merge_workspace_args,
            [kernel.args.workspace_args for kernel in kernels],
        )
        for kernel in kernels:
            kernel.args.workspace_args = workspace_args
        return workspace_args

    def call_kernel(self, kernel_name):
        """
        Collect the union of arguments from all subkernels as the arguments
        for the multi-kernel.
        """
        # Prevent circular import
        from ..select_algorithm import TritonTemplateKernel

        assert kernel_name == self.kernel_name
        V.graph.wrapper_code.write_triton_header_once()
        _, call_args, _, arg_types = self.kernels[0].args.python_argdefs()
        for kernel in self.kernels[1:]:
````
- **EN**: Imports dependencies such as `..select_algorithm` for the logic in this range. Introduces function `call_kernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..select_algorithm` 等依赖，为后续逻辑提供基础能力。这里定义了函数`call_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 201-220 / 第 201-220 行
````python
            _, other_call_args, _, other_arg_types = kernel.args.python_argdefs()
            assert call_args == other_call_args, (call_args, other_call_args)
            assert arg_types == other_arg_types

        if V.graph.cpp_wrapper and not config.triton.autotune_at_compile_time:
            # for the second pass of cpp-wrapper codegen, we should call
            # the fast kernel directly
            kernel_name = MultiKernelCall.lookup_choice(self.kernel_name)

        if isinstance(self.kernels[0], TritonTemplateKernel) and isinstance(
            self.kernels[0].output_node, MultiTemplateBuffer
        ):
            # For matmuls the grid arguments are passed in as additional arguments
            # to the kernel run method. These grids change based on the various
            # parameters of the matmul. So we need to pass each kernel's grid into
            # the multi call kernel.
            multi_call_args = call_args
            multi_call_arg_types = arg_types
            for kernel in self.kernels:
                additional_call_args, additional_arg_types = (
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_name`, `multi_call_args`, and `multi_call_arg_types`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_name`、`multi_call_args`、`multi_call_arg_types` 等值。

### Lines 221-240 / 第 221-240 行
````python
                    kernel.additional_call_args_and_types()
                )
                multi_call_args.extend(list(additional_call_args))
                multi_call_arg_types.extend(list(additional_arg_types))
        else:
            # numels for all subkernels should be the same. Use kernels[0] here
            self.kernels[0].add_numel_to_call_args(kernel_name, call_args, arg_types)
            multi_call_args = call_args
            multi_call_arg_types = arg_types

        for ws in self.kernels[0].args.workspace_args:
            V.graph.wrapper_code.generate_workspace_allocation(ws)

        if V.graph.cpp_wrapper:
            # We have already selected the best kernel at compile time
            # so we only have one set of call args. NB: this currently
            # doesn't work with MultiTemplateBuffer kernels. @bobrenjc93
            # will add it in a subsequent PR.
            V.graph.wrapper_code.generate_kernel_call(
                kernel_name, call_args, arg_types=arg_types
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `multi_call_args`, and `multi_call_arg_types`. This range continues the implementation of function `MultiKernel.call_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`multi_call_args`、`multi_call_arg_types` 等值。这一段延续了函数`MultiKernel.call_kernel` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
            )
        else:
            V.graph.wrapper_code.generate_kernel_call(
                kernel_name, multi_call_args, arg_types=multi_call_arg_types
            )

        for ws in reversed(self.kernels[0].args.workspace_args):
            V.graph.wrapper_code.generate_workspace_deallocation(ws)

    def codegen_nan_check(self):
        wrapper = V.graph.wrapper_code
        seen: OrderedSet[str] = OrderedSet()
        for k in self.kernels:
            _, call_args, precompile_args, _ = k.args.python_argdefs()
            for arg, precompile_arg in zip(call_args, precompile_args):
                if arg in seen:
                    continue
                seen.add(arg)
                if isinstance(precompile_arg, TensorArg):
                    line = f"assert not {arg}.isnan().any().item()"
````
- **EN**: Introduces function `codegen_nan_check`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `wrapper`, `seen`, and `line`.
- **CN**: 这里定义了函数`codegen_nan_check`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`wrapper`、`seen`、`line` 等值。

### Lines 261-280 / 第 261-280 行
````python
                    wrapper.writeline(line)
                    line = f"assert not {arg}.isinf().any().item()"
                    wrapper.writeline(line)

    @property
    def removed_buffers(self):
        return OrderedSet.intersection(*[k.removed_buffers for k in self.kernels])

    @property
    def inplaced_to_remove(self):
        return OrderedSet.intersection(*[k.inplaced_to_remove for k in self.kernels])

    @property
    @cache_on_self
    def inplace_update_buffers(self):
        """
        Make sure all kernels have the same inplace update mappings.
        """
        for k in self.kernels[1:]:
            assert k.inplace_update_buffers == self.kernels[0].inplace_update_buffers
````
- **EN**: Introduces function `removed_buffers`, function `inplaced_to_remove`, function `inplace_update_buffers`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`removed_buffers`、函数`inplaced_to_remove`、函数`inplace_update_buffers`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-300 / 第 281-300 行
````python
        return self.kernels[0].inplace_update_buffers

    def warn_mix_layout(self, kernel_name: str):
        pass


class MultiKernelCall:
    """
    This class is called at run time to actually run the kernel
    """

    def __init__(self, multi_kernel_name, kernels, arg_index):
        assert len(kernels) >= 1
        self._kernels = kernels
        self.multi_kernel_name = multi_kernel_name

        self.disable_cache = os.environ.get(
            "TORCHINDUCTOR_DISABLE_MULTI_KERNEL_CACHE"
        ) == "1" or is_metric_table_enabled("persistent_red_perf")

````
- **EN**: Introduces function `warn_mix_layout`, class `MultiKernelCall`, function `__init__`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`warn_mix_layout`、类`MultiKernelCall`、函数`__init__`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 301-320 / 第 301-320 行
````python
        self.picked_kernel = None
        self.arg_index = arg_index
        if config.triton.multi_kernel > 1:
            # manually force a subkernel to ease perf testing
            picked_by_config = config.triton.multi_kernel - 2
            assert picked_by_config < len(self._kernels)
            # pyrefly: ignore [bad-assignment]
            self.picked_kernel = picked_by_config
        elif not self.disable_cache:
            self.load_cache()

        self._recorded = False

    def cache_file_path(self):
        key = code_hash(
            ",".join(
                [
                    f"{k.fn.cache_key}{k.size_hints!r}{k.triton_meta!r}"
                    for k in self.kernels
                ]
````
- **EN**: Introduces function `cache_file_path`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`cache_file_path`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 321-340 / 第 321-340 行
````python
            )
        )
        _, _, path = get_path(key, "picked_kernel")
        return pathlib.Path(path)

    def load_cache(self):
        assert self.picked_kernel is None
        path = self.cache_file_path()
        if path.exists():
            with path.open() as fd:
                # pyrefly: ignore [bad-assignment]
                self.picked_kernel = int(fd.read())
                # pyrefly: ignore [unsupported-operation]
                assert self.picked_kernel >= 0 and self.picked_kernel < len(
                    self._kernels
                )
                log.debug(
                    "Load picked kernel %d from cache file %s", self.picked_kernel, path
                )

````
- **EN**: Introduces function `load_cache`. Touches the filesystem to load, validate, or store compiler artifacts. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`load_cache`。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 341-360 / 第 341-360 行
````python
    def store_cache(self):
        assert self.picked_kernel is not None
        path = self.cache_file_path()
        path.parent.mkdir(parents=True, exist_ok=True)

        write_atomic(path, str(self.picked_kernel))
        log.debug("Store picked kernel %d to cache file %s", self.picked_kernel, path)

    @property
    def kernels(self):
        """
        Read results from future.

        This should be called after parallel compilation is done.
        In case you call this before compilation is done,
        it may slow down the parallel compilation.
        """
        for i, kernel in enumerate(self._kernels):
            if isinstance(kernel, CodeCacheFuture):
                self._kernels[i] = kernel.result()
````
- **EN**: Introduces function `store_cache`, function `kernels`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里定义了函数`store_cache`、函数`kernels`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 361-380 / 第 361-380 行
````python

        return self._kernels

    def benchmark_sub_kernels(self, *args, **kwargs):
        """
        Benchmark all the sub kernels and return the execution time
        (in milliseconds) for each of time.

        Unit test may mock this method to force a specific kernel to
        be picked.
        """

        def wrap_fn(kernel, index):
            def inner():
                filtered_args = self._get_filtered_args(args, index)
                args_clone, kwargs_clone = kernel.clone_args(*filtered_args, **kwargs)
                return kernel.run(*args_clone, **kwargs_clone)

            return inner

````
- **EN**: Introduces function `benchmark_sub_kernels`, function `wrap_fn`, function `inner`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `filtered_args`.
- **CN**: 这里定义了函数`benchmark_sub_kernels`、函数`wrap_fn`、函数`inner`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `filtered_args` 等值。

### Lines 381-400 / 第 381-400 行
````python
        return [
            benchmarker.benchmark(
                wrap_fn(kernel, index),
                # Currently the kernel type must be a CachingAutotuner
                device=kernel.device_props.type,
                rep=40,
            )
            for index, kernel in enumerate(self.kernels)
        ]

    def _get_filtered_args(self, args, index):
        """
        We pass in all arguments to all kernels into the MultiKernelCall
        so when invoking a particular kernel we need to filter to only the
        arguments for that specific kernel.
        """

        # This is sometimes invoked at runtime where V.graph is
        # a NullHandler
        if hasattr(V.graph, "cpp_wrapper") and V.graph.cpp_wrapper:
````
- **EN**: Introduces function `_get_filtered_args`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_filtered_args`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 401-420 / 第 401-420 行
````python
            # for cpp-wrapper, we should not filter the args since
            # we already have chosen a single kernel and arg set.
            return args
        return [item for s in self.arg_index[index] for item in args[s]]

    # record_choice and lookup_choice are helper functions for cpp-wrapper
    # codegen. The first pass use record_choice to keep the choice and
    # the second pass do lookup by calling lookup_choice.
    #
    # An alternative that reused the multi-kernel cache does not work well
    # since during codegen of the second pass, it's very hard to know the
    # path for the cache file. Also reading the cache file need do some IO
    # which can be slower.
    @staticmethod
    def record_choice(multi_kernel_name: str, picked_kernel_name: str):
        """
        Record the multi-kernel choice for cpp-wrapper after autotuning

        We should do nothing if this function is not called during codegen.
        """
````
- **EN**: Introduces function `record_choice`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`record_choice`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 421-440 / 第 421-440 行
````python
        from torch._inductor.graph import GraphLowering

        if not isinstance(V.graph, GraphLowering):
            return

        if not V.graph.record_multi_kernel_choice:
            return

        V.graph.multi_kernel_to_choice[multi_kernel_name] = picked_kernel_name

    @staticmethod
    def lookup_choice(multi_kernel_name: str) -> str:
        # this should always been done during cpp-wrapper codegen
        assert (
            V.graph.record_multi_kernel_choice
            and multi_kernel_name in V.graph.multi_kernel_to_choice
        )
        # there should be no miss
        return V.graph.multi_kernel_to_choice[multi_kernel_name]

````
- **EN**: Imports dependencies such as `torch._inductor.graph` for the logic in this range. Introduces function `lookup_choice`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.graph` 等依赖，为后续逻辑提供基础能力。这里定义了函数`lookup_choice`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 441-460 / 第 441-460 行
````python
    def run(self, *args, **kwargs):
        if self.picked_kernel is None:
            timings = self.benchmark_sub_kernels(*args, **kwargs)
            self.picked_kernel = timings.index(min(timings))
            k0 = self.kernels[0]
            log.debug(
                "pick %dth sub-kernel in %s. Size hints %s. Reduction hint %s. Timings %s",
                self.picked_kernel,
                [k.inductor_meta.get("kernel_name") for k in self.kernels],
                k0.size_hints,
                k0.inductor_meta.get("reduction_hint"),
                timings,
            )
            get_metric_table("persistent_red_perf").add_row(
                functools.partial(self._metrics_table_row, timings)
            )

            if not self.disable_cache:
                self.store_cache()

````
- **EN**: Introduces function `run`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `timings`, and `k0`.
- **CN**: 这里定义了函数`run`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `timings`、`k0` 等值。

### Lines 461-480 / 第 461-480 行
````python
        if not self._recorded:
            self._recorded = True
            picked_kernel_name = self.kernels[self.picked_kernel].inductor_meta.get(
                "kernel_name"
            )
            assert picked_kernel_name is not None
            self.record_choice(self.multi_kernel_name, picked_kernel_name)

        run = self.kernels[self.picked_kernel].run  # type: ignore[method-assign]
        filtered_args = self._get_filtered_args(args, self.picked_kernel)
        run(*filtered_args, **kwargs)

    def _metrics_table_row(self, timings):
        def get_kernel_path(k):
            return k.fn.fn.__code__.co_filename

        k0 = self.kernels[0]
        row = {
            "size_hints": k0.size_hints,
            "reduction_hint": k0.inductor_meta.get("reduction_hint"),
````
- **EN**: Introduces function `_metrics_table_row`, function `get_kernel_path`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `picked_kernel_name`, `run`, `filtered_args`, `k0`, and `row`.
- **CN**: 这里定义了函数`_metrics_table_row`、函数`get_kernel_path`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `picked_kernel_name`、`run`、`filtered_args`、`k0`、`row` 等值。

### Lines 481-500 / 第 481-500 行
````python
        }
        max_kernels = 4
        assert len(timings) <= max_kernels
        for i in range(max_kernels):
            if i < len(self.kernels):
                row[f"kernel{i}_path"] = get_kernel_path(self.kernels[i])
                row[f"kernel{i}_latency"] = timings[i]
            else:
                row[f"kernel{i}_path"] = ""
                row[f"kernel{i}_latency"] = ""
        return row


class SizeHintMultiKernel(MultiKernel):
    """
    Version of multi-kernel that generates kernels based on specified size hints.
    Currently only performs 1-d search over hints; doesn't perform combinatorial n-d search
    if n > 1 dynamic dimensions are specified.

    e.g. matmul([s0, s1], [s1, s2]) with size-hints [64, 256] only generates 2 kernels,
````
- **EN**: Introduces class `SizeHintMultiKernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `max_kernels`, and `else`.
- **CN**: 这里定义了类`SizeHintMultiKernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `max_kernels`、`else` 等值。

### Lines 501-520 / 第 501-520 行
````python
    based on tuning shapes ([64, 64], [64, 64]) and ([256, 256], [256, 256])
    """

    def __init__(self, kernels):
        assert isinstance(kernels, dict) and len(kernels) >= 1

        self.kernels, self.kernel_shape_keys = [], []
        for shape_key, kernel in kernels.items():
            self.kernels.append(kernel)
            self.kernel_shape_keys.append(shape_key)
        self.kernel_name = V.graph.wrapper_code.multi_kernel_state.define_kernel(
            self.kernels, self.kernel_shape_keys
        )

        # need this since some code in inductor check if the kernel object has an args
        # attribute to decide if it's a non-null kernel.
        self.args = object()


class SizeHintMultiKernelCall(MultiKernelCall):
````
- **EN**: Introduces function `__init__`, class `SizeHintMultiKernelCall`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`__init__`、类`SizeHintMultiKernelCall`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 521-540 / 第 521-540 行
````python
    """
    Runtime class for size-hint multi-kernels.
    Instead of having a plain list of kernels to benchmark over, keys them by input & output shapes,
    and optionally perform shape-based selection. The pre-generated kernel is chosen based on the shape keys,
    with the heuristic being log2 l1 distance between the pre-generated / runtime input & output shapes.
    """

    def __init__(self, multi_kernel_name, kernels, arg_index):
        super().__init__(multi_kernel_name, list(kernels.values()), arg_index)
        self._kernel_hints = list(kernels.keys())

        # Caches results for unique shapes.
        self._shape_cache = {}

    def _get_shape_cache_key(self, *args, **kwargs):
        """
        Generate a cache key based on tensor shapes for shape-specialized dispatch.
        """
        shapes = []
        for arg in args:
````
- **EN**: Introduces function `__init__`, function `_get_shape_cache_key`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Processes profiling, FLOP, or benchmark information to characterize execution cost. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`__init__`、函数`_get_shape_cache_key`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含分支、循环或上下文管理等控制流。

### Lines 541-560 / 第 541-560 行
````python
            if hasattr(arg, "shape"):
                shapes.append(tuple(arg.shape))
        return tuple(shapes)

    def _get_cached_shape_choice(self, cache_key):
        """
        Get cached kernel choice for a specific shape.
        """
        return self._shape_cache.get(cache_key)

    def _cache_shape_choice(self, cache_key, kernel_idx):
        """
        Cache kernel choice for a specific shape.
        """
        self._shape_cache[cache_key] = kernel_idx

    def _dist_heuristic(self, k1, k2):
        """
        log2 L1 distance heuristic for kernel selection.
        """
````
- **EN**: Introduces function `_get_cached_shape_choice`, function `_cache_shape_choice`, function `_dist_heuristic`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_cached_shape_choice`、函数`_cache_shape_choice`、函数`_dist_heuristic`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 561-580 / 第 561-580 行
````python

        def dist(x, y):
            lx = math.log2(x) if x > 0 else -1
            ly = math.log2(y) if y > 0 else -1
            return abs(lx - ly)

        out = 0
        for s1, s2 in zip(k1, k2):
            out += sum(dist(x, y) for x, y in zip(s1, s2))
        return out

    def run(self, *args, **kwargs):
        cache_key = self._get_shape_cache_key(*args, **kwargs)
        cached_choice = self._get_cached_shape_choice(cache_key)
        if cached_choice is not None:
            self.picked_kernel = cached_choice
            log.debug(
                "using cached shape-specialized choice %dth sub-kernel in %s. Cache key: %s",
                self.picked_kernel,
                [k.inductor_meta.get("kernel_name") for k in self.kernels],
````
- **EN**: Introduces function `dist`, function `run`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`dist`、函数`run`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 581-600 / 第 581-600 行
````python
                cache_key,
            )
        else:
            self._select_kernel_by_shape(*args, **kwargs)

        if not self._recorded:
            self._recorded = True
            picked_kernel_name = self.kernels[self.picked_kernel].inductor_meta.get(
                "kernel_name"
            )
            assert picked_kernel_name is not None
            self.record_choice(self.multi_kernel_name, picked_kernel_name)

        run = self.kernels[self.picked_kernel].run  # type: ignore[method-assign]
        filtered_args = self._get_filtered_args(args, self.picked_kernel)
        run(*filtered_args, **kwargs)

    def _select_kernel_by_shape(self, *args, **kwargs):
        """
        Benchmark kernels for a particular shape and return the
````
- **EN**: Introduces function `_select_kernel_by_shape`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_select_kernel_by_shape`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 601-610 / 第 601-610 行
````python
        best kernel for this shape.
        """
        shape_key = self._get_shape_cache_key(*args, **kwargs)
        dists = [
            self._dist_heuristic(shape_key, key) if key is not None else 2**62
            for key in self._kernel_hints
        ]
        # pyrefly: ignore [bad-assignment]
        self.picked_kernel = dists.index(min(dists))
        self._cache_shape_choice(shape_key, self.picked_kernel)
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `shape_key`, and `dists`. This range continues the implementation of function `SizeHintMultiKernelCall._select_kernel_by_shape`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `shape_key`、`dists` 等值。这一段延续了函数`SizeHintMultiKernelCall._select_kernel_by_shape` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `logging`, `math`, `os`, `pathlib`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.ir`, `torch._inductor.metrics`, `torch.utils._ordered_set`, `..`, `..codecache`, `..runtime.benchmarking`, `..utils`, `..virtualized`, `.common`, `..select_algorithm`, `torch._inductor.graph`
