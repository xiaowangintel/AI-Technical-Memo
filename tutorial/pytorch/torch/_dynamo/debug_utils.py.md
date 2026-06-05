# debug_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/debug_utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: Debug utilities for TorchDynamo compilation and execution.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-38
```python
"""
Debug utilities for TorchDynamo compilation and execution.

This module provides various debugging tools and utilities for TorchDynamo, including:

- Minification support for reducing test cases while preserving bugs
- Input/output handling via InputReader and InputWriter for reproducible testing
- Accuracy checking between original and compiled models
- Neural network module string conversion via NNModuleToString
- Profiling tools and system information collection
- Buck build system integration for Meta-internal testing

Key classes:
- InputReader/InputWriter: Handle serialization of model inputs/outputs
- NNModuleToString: Converts nn.Modules to string representations
- BuckTargetWriter: Manages Buck build system integration
"""

from __future__ import annotations

import atexit
import copy
import cProfile
import functools
import getpass
import inspect
import itertools
import logging
import os
import re
import subprocess
import sys
import tempfile
import textwrap
from collections import Counter
from importlib import import_module
from typing import Any, TYPE_CHECKING, TypeVar
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 39-71
```python
import torch
import torch._prims_common as utils
import torch._subclasses.meta_utils
from torch import Tensor
from torch._dynamo.testing import rand_strided
from torch._inductor.cpp_builder import normalize_path_separator
from torch._prims_common import is_float_dtype
from torch.multiprocessing.reductions import StorageWeakRef
from torch.utils._content_store import ContentStoreReader, ContentStoreWriter

from . import config
from .utils import clone_inputs, get_debug_dir, warn_once


if TYPE_CHECKING:
    from collections.abc import Callable, Sequence

    from torch.hub import tqdm
    from torch.storage import UntypedStorage


log = logging.getLogger(__name__)

T = TypeVar("T")


inductor_config = import_module("torch._inductor.config")
use_buck = inductor_config.is_fbcode()

if use_buck:
    import libfb.py.build_info
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 72-110
```python
# pyrefly: ignore [implicit-any]
extra_deps = []
extra_imports = ""
cur_target = ""
if use_buck:
    extra_deps = [
        "//caffe2/torch/fb/sparsenn:sparsenn_operators_gpu",
        "//caffe2/torch/fb/sparsenn:sparsenn_operators",
        "//deeplearning/fbgemm/fbgemm_gpu:sparse_ops_cpu",
        "//deeplearning/fbgemm/fbgemm_gpu:sparse_ops",
    ]
    cur_target = libfb.py.build_info.BuildInfo.get_build_rule().replace("fbcode:", "//")  # type: ignore[possibly-undefined]
    extra_imports = "\n".join([f'torch.ops.load_library("{x}")' for x in extra_deps])


BUCK_CMD_PREFIX = ["buck2", "run", "@mode/dev-nosan"]


class BuckTargetWriter:
    def __init__(self, filename: str) -> None:
        self.subdir, self.py_file = os.path.split(os.path.abspath(filename))
        self.target = self.py_file.replace(".py", "")

        # Get main_module path from fbcode
        self.path = f"{self.subdir.replace('/', '.')}.{self.target}"
        self.path = self.path[self.path.find("fbcode.") :]
        self.path = self.path[7:]

        # Get cmd line path
        tmp = self.subdir
        tmp = tmp[tmp.find("fbcode/") :][7:]
        self.cmd_line_path = f"//{tmp}:{self.target}"

    def build(self) -> str:
        extra_cpp_deps = "\n".join([f'        "{x}",' for x in extra_deps])
        return textwrap.dedent(
            f"""
load("@fbcode_macros//build_defs:python_binary.bzl", "python_binary")
```
- **EN**: This range initializes module-level constants or registries that later code reuses to organize reusable module behavior and state.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以组织可复用的模块行为与状态。

### Lines 111-152
```python
python_binary(
    name="{self.target}",
    srcs = ["{self.py_file}"],
    compile = False,
    deps = [
        "//caffe2:torch",
        "//caffe2:libtorch",
        "//caffe2/functorch:functorch",
        "//triton:triton",
        "{cur_target}",
    ],
    cpp_deps = [
{extra_cpp_deps}
    ],
    main_module = "{self.path}",
    par_style = "xar",
)
"""
        )

    def write(self, print_msg: bool = True) -> list[str]:
        target_file = os.path.join(self.subdir, "TARGETS")
        with open(target_file, "w") as fd:
            fd.write(self.build())
        # log.warning("Wrote isolation TARGETS file at %s", target_file)
        cmd_split = BUCK_CMD_PREFIX + [self.cmd_line_path]
        if print_msg:
            log.warning(
                "Found an example that reproduces the error. Run this cmd to repro - %s",
                " ".join(cmd_split),
            )
        return cmd_split


def minifier_dir() -> str:
    path = os.path.join(get_debug_dir(), "minifier")
    if path is None:
        path = f"{tempfile.gettempdir()}/minifier_{getpass.getuser()}"
    if not os.path.exists(path):
        os.makedirs(path, exist_ok=True)
    return path
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 153-193
```python

MAX_CONSTANT_NUMEL_INLINE = 4


class NNModuleToString:
    safe_reprs = [
        torch.nn.Linear,
        torch.nn.Conv1d,
        torch.nn.Conv2d,
        torch.nn.Conv3d,
        torch.nn.BatchNorm1d,
        torch.nn.BatchNorm2d,
        torch.nn.BatchNorm3d,
        torch.nn.LayerNorm,
        torch.nn.Dropout,
        torch.nn.Softmax,
        torch.nn.ReLU,
        torch.nn.GELU,
        torch.nn.Identity,
        torch.nn.MaxPool2d,
        torch.nn.Embedding,
        torch.nn.Tanh,
        torch.nn.ConvTranspose1d,
        torch.nn.GLU,
        torch.nn.LSTM,
        torch.nn.Flatten,
        torch.nn.AdaptiveAvgPool2d,
    ]

    @staticmethod
    def can_convert_to_string(gm: torch.fx.GraphModule) -> bool:
        cant_convert = set()
        for _, module in gm.named_children():
            if type(module) not in NNModuleToString.safe_reprs:
                cant_convert.add(module)

        if len(cant_convert) > 0:
            log.warning("We have not tested reprs of some modules - %s", cant_convert)
        # TODO - Assuming that all modules can be safely repr'd. Check if that assumption is correct.
        return True
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 194-224
```python
    @staticmethod
    def convert(gm: torch.fx.GraphModule) -> str:
        from torch.nn.modules.module import _addindent

        tab = " " * 4

        model_str = textwrap.dedent(
            """
            from torch.nn import *
            class Repro(torch.nn.Module):
                def __init__(self) -> None:
                    super().__init__()
            """
        )

        for module_name, module in gm.named_children():
            module_str = f"{module.__repr__()}"
            # module should be a core torch.nn.Module, so all parameters
            # should be on the same device.
            example_param = next(module.parameters(), None)
            if example_param is not None and example_param.is_cuda:
                module_str = f"{module_str}.cuda()"
            model_str += f"{tab * 2}self.{module_name} = {module_str}\n"

        for buffer_name, buffer in gm._buffers.items():
            if buffer is None:
                continue
            # Serialize full data for small buffers
            if buffer.numel() <= MAX_CONSTANT_NUMEL_INLINE:
                from torch._tensor_str import PRINT_OPTS
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 225-263
```python
                assert PRINT_OPTS.threshold >= MAX_CONSTANT_NUMEL_INLINE
                tensor_str = repr(buffer)
            elif torch.is_floating_point(buffer):
                tensor_str = f"torch.randn({list(buffer.shape)}, dtype={buffer.dtype})"
            else:
                tensor_str = (
                    f"torch.randint(1, size={list(buffer.shape)}, dtype={buffer.dtype})"
                )
            if buffer.is_cuda:
                tensor_str = f"{tensor_str}.cuda()"
            model_str += (
                f"{tab * 2}self.register_buffer('{buffer_name}', {tensor_str})\n"
            )

        for param_name, param in gm._parameters.items():
            if param is None:
                continue
            maybe_device = ""
            if param.is_cuda:
                maybe_device = ', device="cuda"'
            tensor_str = f"torch.nn.Parameter(torch.randn({list(param.shape)}, dtype={param.dtype}{maybe_device}))"
            model_str += f"{tab * 2}self.{param_name} = {tensor_str}\n"

        # TODO - Keep this code for now. But, I don't think we will need this.
        # attrs = dir(gm)
        # for attr in attrs:
        #     if "_tensor_constant" in attr:
        #         val = getattr(gm, attr)
        #         model_str += f"    {attr} = {val!r}\n"

        model_str += f"{_addindent(gm.code, 4)}\n"
        return model_str


@functools.cache  # subprocess is expensive
def _cuda_system_info_comment() -> str:
    if not torch.cuda.is_available():
        return "# torch.cuda.is_available()==False, no GPU info collected\n"
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 264-299
```python
    model_str = "# CUDA Info: \n"
    try:
        if torch.version.hip is None:
            cuda_version_out = subprocess.check_output(["nvcc", "--version"])
            cuda_version_lines = cuda_version_out.decode().split("\n")
            comment = "".join([f"# {s} \n" for s in cuda_version_lines if s != ""])
            model_str += f"{comment}\n"
        else:
            model_str += "# Not searching for nvcc on ROCM setup\n"
    except (FileNotFoundError, subprocess.CalledProcessError):
        model_str += "# nvcc not found\n"

    gpu_names = Counter(
        torch.cuda.get_device_name(i) for i in range(torch.cuda.device_count())
    )

    model_str += "# GPU Hardware Info: \n"
    for name, count in gpu_names.items():
        model_str += f"# {name} : {count} \n"
    model_str += "\n"
    return model_str


def generate_env_vars_string(*, stable_output: bool = False) -> str:
    """
    Generate a string configuration for environment variables related to Dynamo, Inductor, and Triton.
    """
    if stable_output:
        return "# env var omitted due to stable_output=True"

    allow_list = ["TORCH", "DYNAMO", "INDUCTOR", "TRITON"]
    skip_list = ["TRITON_LIBDEVICE_PATH", "TRITON_PTXAS_PATH", "TRITON_LIBCUDA_PATH"]

    def filter(key: str) -> bool:
        return any(string in key for string in allow_list) and key not in skip_list
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 300-339
```python
    config_lines = [
        f"""os.environ['{key}'] = '{value.replace("'", '"')}'"""
        for key, value in os.environ.items()
        if filter(key)
    ]
    config_string = "\n".join(config_lines)
    return normalize_path_separator(f"""\
import os
{config_string}
    """)


def generate_config_string(*, stable_output: bool = False) -> str:
    import torch._functorch.config
    import torch._inductor.config

    if stable_output:
        return "# config omitted due to stable_output=True"

    experimental_config = torch.fx.experimental._config.codegen_config()  # type: ignore[attr-defined]
    return f"""\
import torch._dynamo.config
import torch._inductor.config
import torch._functorch.config
import torch.fx.experimental._config
{torch._dynamo.config.codegen_config()}
{torch._inductor.config.codegen_config()}
{torch._functorch.config.codegen_config()}
{experimental_config}
"""


def get_minifier_repro_path() -> str:
    return os.path.join(minifier_dir(), "minifier_launcher.py")


def helper_for_dump_minify(contents: str) -> None:
    minified_repro_path = get_minifier_repro_path()
    log.warning("Writing minified repro to:\n%s", minified_repro_path)
```
- **EN**: This module-level block helps capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 340-381
```python
    if use_buck:
        BuckTargetWriter(minified_repro_path).write()
    try:
        with open(minified_repro_path, "w") as fd:
            fd.write(contents)

    except OSError as e:
        log.exception("")
        raise NotImplementedError(f"Could not write to {minified_repro_path}") from e


class AccuracyError(Exception):
    pass


def clone_inputs_retaining_gradness(example_inputs: Sequence[Any]) -> list[Any]:
    """
    This clone inputs is different from utils clone_input. In case of minifier,
    all the tensors are leaf tensors while creating a new graph. So, we set the
    requires_grad field w/o checking the leafness of the tensor.
    """
    cloned_inputs = clone_inputs(example_inputs)
    for idx in range(len(example_inputs)):
        if isinstance(cloned_inputs[idx], torch.Tensor):
            cloned_inputs[idx].requires_grad_(example_inputs[idx].requires_grad)
    return cloned_inputs  # type: ignore[return-value]


def run_fwd_maybe_bwd(
    gm: torch.fx.GraphModule,
    args: Sequence[Any],
    only_fwd: bool = False,
    disable_clone: bool = False,
) -> Any:
    """
    Runs a forward and possibly backward iteration for a given mod and args.

    When disable_clone is True, we will use args as-is without cloning.
    This is higher fidelity but we may destroy the args in the process.
    """
    from .testing import collect_results, reduce_to_scalar_loss, requires_bwd_pass
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 382-420
```python
    gm = copy.deepcopy(gm)
    if not disable_clone:
        args = clone_inputs_retaining_gradness(args)

    if hasattr(gm, "zero_grad"):
        gm.zero_grad(True)

    # TorchInductor returned callable expects lists. So, may need a boxed calling convention.
    out = gm(args) if getattr(gm, "_boxed_call", False) else gm(*args)

    if only_fwd:
        return out
    if requires_bwd_pass(out):
        loss = reduce_to_scalar_loss(out)
        loss.backward()
    return collect_results(gm, out, None, args)


def same_two_models(
    gm: torch.fx.GraphModule,
    opt_gm: torch.fx.GraphModule,
    example_inputs: Sequence[Any],
    only_fwd: bool = False,
    *,
    require_fp64: bool = False,
    ignore_non_fp: bool = False,
) -> bool:
    """
    Check two models have same accuracy.

    require_fp64: if True, raise an error if we unable to calculate the fp64 reference
    ignore_non_fp: if True, do not compare outputs which are not floating point.  This
        is mostly useful for the minifier (which wants to avoid quantizing floating point
        error into integer/boolean error)
    """
    from .utils import same

    ref = run_fwd_maybe_bwd(gm, example_inputs, only_fwd)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 421-457
```python
    fp64_ref = None
    if config.same_two_models_use_fp64:
        try:
            fp64_model, fp64_examples = cast_to_fp64(
                copy.deepcopy(gm), clone_inputs_retaining_gradness(example_inputs)
            )
            fp64_ref = run_fwd_maybe_bwd(fp64_model, fp64_examples, only_fwd)
        except Exception:
            if require_fp64:
                raise RuntimeError(  # noqa: B904
                    "Could not generate fp64 outputs, workaround with torch._dynamo.config.same_two_models_use_fp64 = False"
                )
            log.warning("Could not generate fp64 outputs")

    try:
        res = run_fwd_maybe_bwd(opt_gm, example_inputs, only_fwd)
    except Exception:
        # This means that the minified graph is bad/exposes a different problem.
        # As we are checking accuracy here, lets log the exception and return True.
        log.exception(
            "While minifying the program in accuracy minification mode, "
            "ran into a runtime exception which is likely an unrelated issue."
            " Skipping this graph."
        )
        return True

    passing = same(
        ref,
        res,
        fp64_ref,
        tol=config.repro_tolerance,
        equal_nan=True,
        ignore_non_fp=ignore_non_fp,
    )
    return passing
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 458-498
```python
def cast_dtype_args_to_fp64(model: torch.fx.GraphModule) -> torch.fx.GraphModule:
    for node in model.graph.nodes:
        if (
            node.op == "call_function"
            and node.target is torch.ops.prims.convert_element_type.default
        ):
            assert len(node.args) == 2
            if is_float_dtype(node.args[1]) and node.args[1] != torch.float64:
                node.args = (node.args[0], torch.float64)
        if node.op == "call_function":
            dtype = node.kwargs.get("dtype")
            if dtype is not None and is_float_dtype(dtype):
                new_kwargs = dict(node.kwargs)
                new_kwargs["dtype"] = torch.float64
                node.kwargs = new_kwargs

    model.graph.lint()
    model.recompile()
    return model


def cast_to(
    dtype: torch.dtype, model: torch.fx.GraphModule, inputs: list[Any]
) -> tuple[torch.fx.GraphModule, list[Any]]:
    from torch.utils._pytree import tree_map

    model = model.to(dtype)
    if dtype == torch.float64:
        # If casting to fp64 for accuracy comparison, we need to
        # replace dtype arguments embedded in the graph with fp64
        model = cast_dtype_args_to_fp64(model)

    inputs = tree_map(
        lambda x: x.to(dtype)
        if isinstance(x, torch.Tensor) and x.is_floating_point()
        else x,
        inputs,
    )
    return model, inputs
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 499-540
```python
def cast_to_fp64(
    model: torch.fx.GraphModule, inputs: list[Any]
) -> tuple[torch.fx.GraphModule, list[Any]]:
    return cast_to(torch.float64, model, inputs)


def backend_accuracy_fails(
    gm: torch.fx.GraphModule,
    example_inputs: Sequence[Any],
    compiler_fn: Callable[[torch.fx.GraphModule, list[Any]], torch.fx.GraphModule],
    only_fwd: bool = False,
    *,
    require_fp64: bool = False,
    ignore_non_fp: bool = False,
) -> bool:
    try:
        compiled_gm = compiler_fn(
            copy.deepcopy(gm), clone_inputs_retaining_gradness(example_inputs)
        )
        return not same_two_models(
            gm,
            compiled_gm,
            example_inputs,
            only_fwd,
            require_fp64=require_fp64,
            ignore_non_fp=ignore_non_fp,
        )
    except Exception:
        # This means that the minified graph is bad/exposes a different problem.
        # As we are checking accuracy here, lets log the exception and return False.
        log.exception(
            "While minifying the program in accuracy minification mode, "
            "ran into a runtime exception which is likely an unrelated issue."
            " Skipping this graph"
        )
        return False


# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #
#                       REPRO SUPPORT CODE
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 541-581
```python

# Helper functions for computing what the default values of tensor
# values should be.  These all coincide with factory functions, e.g., torch.empty


def _stride_or_default(
    stride: torch._prims_common.StrideType | None,
    *,
    shape: torch._prims_common.ShapeType,
) -> torch._prims_common.StrideType:
    return stride if stride is not None else utils.make_contiguous_strides_for(shape)


def _mk_defaulter(d: T) -> Callable[[T | None], T]:
    return lambda x: x if x is not None else d


_dtype_or_default = _mk_defaulter(torch.float32)
_device_or_default = _mk_defaulter(torch.device("cpu"))
_storage_offset_or_default = _mk_defaulter(0)
_requires_grad_or_default = _mk_defaulter(False)
_is_leaf_or_default = _mk_defaulter(False)


class NopInputReader:
    def __init__(self) -> None:
        self.total = 0

    def storage(
        self,
        storage_hash: str | None,
        nbytes: int,
        *,
        device: torch._prims_common.DeviceLikeType | None = None,
        dtype_hint: torch.dtype | None = None,
    ) -> None:
        self.total += 1

    def tensor(self, *args: Any, **kwargs: Any) -> torch.Tensor | None:
        pass
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 582-613
```python
    def symint(self, *args: Any, **kwargs: Any) -> int | None:
        pass

    def const(self, name: str) -> None:
        pass

    def unsupported(self, name: str) -> None:
        pass

    def generator(self, device_type: str, device_index: int) -> None:
        pass

    def opaque(self, script_class_name: str) -> None:
        self.total += 1


# TODO: Support bundling the entire repro into a zip file for ease of
# transferring around
class InputReader:
    def __init__(
        self, save_dir: str | None = None, *, pbar: tqdm | None = None
    ) -> None:
        # If None, we will generate random data instead.  It's important
        # to natively support this use case as it will allow people to
        # share repros without including the real data, if the problem
        # reproduces even on random data.
        if save_dir is None:
            log.warning("no save_dir specified, will generate random data")
        self.store = ContentStoreReader(save_dir) if save_dir is not None else None
        self.args: list[Any] = []
        self.pbar = pbar
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 614-642
```python
    def storage(
        self,
        storage_hash: str | None,
        nbytes: int,
        *,
        device: torch._prims_common.DeviceLikeType | None = None,
        dtype_hint: torch.dtype | None = None,
    ) -> UntypedStorage:
        if self.pbar is not None:
            self.pbar.update(1)
        device = _device_or_default(device)  # type: ignore[arg-type]
        dtype_hint = _dtype_or_default(dtype_hint)
        if self.store is not None and storage_hash is not None:
            try:
                storage = self.store.read_storage(storage_hash)
            except FileNotFoundError:
                pass
            else:
                if device != storage.device:
                    log.warning("device mismatch: %s != %s", device, storage.device)
                    # TODO: transfer it to the right device?  But failing this
                    # way would be very mysterious!  Would have been better
                    # not to store device in the serialized format...
                return storage
        warn_once(f"could not load {storage_hash}, generating random data instead")
        shape = (nbytes // dtype_hint.itemsize,)
        stride = _stride_or_default(None, shape=shape)
        return rand_strided(shape, stride, dtype_hint, device).untyped_storage()
```
- **EN**: Declares `InputReader`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `InputReader`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 643-682
```python
    def tensor(
        self,
        storage: UntypedStorage,
        shape: torch._prims_common.ShapeType,
        stride: torch._prims_common.StrideType | None = None,
        *,
        storage_offset: int | None = None,
        dtype: torch.dtype | None = None,
        requires_grad: bool | None = None,
        is_leaf: bool | None = None,
        **metadata: Any,
    ) -> torch.Tensor:
        stride = _stride_or_default(stride, shape=shape)
        storage_offset = _storage_offset_or_default(storage_offset)
        dtype = _dtype_or_default(dtype)
        is_leaf = _is_leaf_or_default(is_leaf)
        requires_grad = _requires_grad_or_default(requires_grad)
        t = torch.tensor(
            [], dtype=dtype, device=storage.device, requires_grad=requires_grad
        )
        with torch.no_grad():
            t.set_(storage, storage_offset, shape, stride)
        if not is_leaf:
            # Fake up some autograd history in a very naughty way
            with torch.enable_grad():
                t = t.clone(memory_format=torch.preserve_format)
            with torch.no_grad():
                t.set_(storage, storage_offset, shape, stride)
        assert torch._subclasses.meta_utils.safe_is_leaf(t) == is_leaf
        torch._utils.set_tensor_metadata(t, metadata)
        self.args.append(t)
        return t  # for BC

    def symint(self, val: Any) -> Any:
        self.args.append(val)
        return val  # for BC

    def const(self, name: str) -> None:
        self.args.append(None)
```
- **EN**: Declares `InputReader`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `InputReader`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 683-719
```python
    def unsupported(self, name: str) -> None:
        self.args.append(None)

    def generator(self, device_type: str, device_index: int) -> torch._C.Generator:
        gen = torch.cuda.default_generators[device_index].clone_state()
        self.args.append(gen)
        return gen

    def opaque(self, script_class_name: str) -> None:
        self.args.append(None)


# Here is our writer strategy:
#  1. We will stream all of the inputs to disk
#  2. You can now deterministically randomize the inputs, or reload
#     the inputs from disk
#  3. You can YOLO run the script without the inputs, in which case
#     we'll fill the inputs with random data and pray.  This is the
#     legacy behavior, but it's also useful if you want to find out
#     if we're so broken even random inputs trigger it
#  4. We could offer an in process "check if the randomized thing
#     works too" but this is delicate so we don't do it


class InputWriter:
    def __init__(self, save_dir: str | None, *, stable_hash: bool = False) -> None:
        self._lines: list[str] = []
        # TODO: consider ensuring tensor and storage counters line up?
        self.storage_counter = itertools.count()
        self.save_dir = save_dir
        self.store = (
            ContentStoreWriter(save_dir, stable_hash=stable_hash)
            if save_dir is not None
            else None
        )
        self.seen_storages: dict[StorageWeakRef, str] = {}
```
- **EN**: This module-level block helps initialize learnable tensors and related state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于初始化可学习张量及相关状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 720-747
```python
    def lines(self) -> list[str]:
        r = [
            "def load_args(reader):",
        ]
        r.extend(f"    {l}" for l in self._lines)
        # In case we need to change the internal format of load_args
        # in an FC-breaking way
        r.append("load_args._version = 0")
        return r

    # Storages are untyped, but we need to initialize them with data if
    # we don't have the real data, so we give a hint saying what kind
    # of initialization may be appropriate
    #
    # If we had a FakeTensor, device_hint tells us what device should be
    def storage(
        self,
        untyped_storage: UntypedStorage,
        *,
        device_hint: torch._prims_common.DeviceLikeType | None = None,
        dtype_hint: torch.dtype | None = None,
    ) -> str:
        ws = StorageWeakRef(untyped_storage)
        v = self.seen_storages.get(ws)
        if v is not None:
            return v
        v = f"buf{next(self.storage_counter)}"
        maybe_dtype_hint = ""
```
- **EN**: Declares `InputWriter`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `InputWriter`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 748-775
```python
        if _dtype_or_default(None) != _dtype_or_default(dtype_hint):
            maybe_dtype_hint = f", dtype_hint={dtype_hint!r}"
        # TODO: being optional on device is kind of pointless as the default
        # is CPU but most repros we care about are CUDA
        maybe_device = ""
        device = untyped_storage.device
        if device.type == "meta":
            assert device_hint is not None
            device = device_hint  # type: ignore[assignment]
        if _device_or_default(None) != device:
            maybe_device = f", device={device!r}"
        nbytes = untyped_storage.nbytes()
        storage_hash = None
        if self.store is not None and untyped_storage.device.type != "meta":
            storage_hash = self.store.write_storage(untyped_storage)
        self._lines.append(
            f"{v} = reader.storage({storage_hash!r}, {nbytes!r}{maybe_device}{maybe_dtype_hint})"
        )
        self.seen_storages[ws] = v
        return v

    def tensor(self, name: str, t: torch.Tensor) -> None:
        from torch.fx.experimental.symbolic_shapes import statically_known_true, sym_eq

        storage = self.storage(
            t.untyped_storage(), dtype_hint=t.dtype, device_hint=t.device
        )
        args = []
```
- **EN**: Declares `InputWriter`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `InputWriter`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 776-803
```python
        # NB: this is positional, must come first
        if not statically_known_true(
            sym_eq(_stride_or_default(None, shape=t.shape), t.stride())
        ):
            args.append(str(tuple(t.stride())))
        if _dtype_or_default(None) != t.dtype:
            args.append(f"dtype={t.dtype!r}")
        if not statically_known_true(
            _storage_offset_or_default(None) == t.storage_offset()
        ):
            args.append(f"storage_offset={t.storage_offset()!r}")
        tensor_metadata = torch._utils.get_tensor_metadata(t)
        if tensor_metadata:
            args.extend(f"{k}={v!r}" for k, v in tensor_metadata.items())
        if _requires_grad_or_default(None) != t.requires_grad:
            args.append(f"requires_grad={t.requires_grad!r}")
        is_leaf = torch._subclasses.meta_utils.safe_is_leaf(t)
        if _is_leaf_or_default(None) != is_leaf:
            args.append(f"is_leaf={is_leaf!r}")
        self._lines.append(
            "reader.tensor("
            + ", ".join([storage, str(tuple(t.shape)), *args])
            + f")  # {name}"
        )

    def unsupported(self, name: str, arg: Any) -> None:
        # NB: Try hard not to /print/ a tensor, that will be very slow
        self._lines.append(
```
- **EN**: Declares `InputWriter`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `InputWriter`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 804-841
```python
            f"reader.unsupported({name!r})  # unsupported type for dumping: {type(arg)}"
        )
        # Best effort dump as much useful stuff we can lol, in case you want
        # to repair the repro
        if isinstance(arg, (list, tuple)):
            self._lines.append('"""')
            for i, a in enumerate(arg):
                name_i = f"{name}[{i}]"
                if isinstance(a, torch.Tensor):
                    self.tensor(name_i, a)
                elif isinstance(a, (int, torch.SymInt)):
                    self.symint(name_i, a)
                else:
                    self.unsupported(name_i, a)
            self._lines.append('"""')

    # write out that the arg was filtered out as it is constant
    def const(self, name: str) -> None:
        self._lines.append(
            f"reader.const({name!r})  # {name}, filtered out during compilation"
        )

    # TODO: this doesn't actually symint atm
    def symint(self, name: str, val: Any) -> None:
        if isinstance(val, torch.SymInt):
            val = val.node.hint
        self._lines.append(f"reader.symint({val!r})  # {name}")

    def generator(self, name: str, arg: torch._C.Generator) -> None:
        device = arg.device
        self._lines.append(
            f"reader.generator({device.type!r}, {device.index!r})  # {name}"
        )

    def opaque(self, name: str, script_class_name: str) -> None:
        self._lines.append(f"reader.opaque({script_class_name!r})  # {name}")
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 842-883
```python
def aot_graph_input_parser(
    func: Callable[[list[Tensor]], list[Tensor]],
    device: str = "cuda",
    sym_shapes: dict[str, int] | None = None,
    default_sym_shape: int | None = None,
) -> dict[str, Any]:
    """
    Takes in a function which has been printed with print_readable() and constructs kwargs to run it.

    Handles Tensor inputs, Symints, and a graph module which might have tensor constants.

    Consider a function `forward` defined as follows:

    def forward(self, primals_1: "f32[1001, 6]", primals_2: "f32[s0]", primals_3: "Sym(s0)",):
        _tensor_constant0: "i64[4190]" = self._tensor_constant0
        # Further implementation

    kwargs = aot_graph_input_parser(forward)
    forward(**kwargs)
    """

    from torch.utils._dtype_abbrs import dtype_abbrs

    dtype_map: dict[str, torch.dtype] = {
        value: key for key, value in dtype_abbrs.items()
    }
    dtype_pattern: str = "|".join(dtype_abbrs.values())

    # Extracting the source code from the function
    source = inspect.getsource(func)

    # Regular expressions
    tensor_assignment_regex = rf"(_tensor_constant\d+): \"({dtype_pattern})\[\s*(.*?)\s*\]\" = self\.(_tensor_constant\d+)"
    tensor_regex = rf"({dtype_pattern})\[\s*(.*?)\s*\]"
    sym_shape_regex = r"Sym\((s\d+)\)"

    class TensorContainer:
        "Container for tensors as attributes"

    # Dictionary for tensors from annotations
    kwargs: dict[str, Any] = {}
```
- **EN**: Defines the `aot_graph_input_parser` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`aot_graph_input_parser` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 884-919
```python
    sym_shapes_dict: dict[str, int] = sym_shapes or {}

    def get_sym_int(symint: str) -> int:
        torch._check(
            symint in sym_shapes_dict or default_sym_shape is not None,
            lambda: f"{symint} not in symbolic_shapes and default sym shape not passed in",
        )
        return sym_shapes_dict.get(symint, default_sym_shape)  # type: ignore[return-value]

    def gen_tensor(shape: torch._prims_common.ShapeType, dtype: torch.dtype) -> Tensor:
        # Resolve symbolic shapes to concrete values
        resolved_shape = []
        dynamic_dims = []
        for i, dim in enumerate(shape):
            dim = dim.strip()  # type: ignore[attr-defined]
            if "s" in dim:
                s = get_sym_int(dim)
                resolved_shape.append(s)
                dynamic_dims.append(i)
            else:
                if dim:
                    resolved_shape.append(int(dim))

        constructor = torch.randn if dtype.is_floating_point else torch.zeros
        out = constructor(resolved_shape, dtype=dtype, device=device)  # type: ignore[call-arg]
        for d in dynamic_dims:
            torch._dynamo.mark_dynamic(out, d)
        return out

    # Parse function annotations for tensor generation
    annotations = func.__annotations__
    for param, annotation in annotations.items():
        # Skip 'return' annotation
        if param == "return":
            continue
```
- **EN**: Defines the `aot_graph_input_parser` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`aot_graph_input_parser` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 920-954
```python
        match = re.search(tensor_regex, annotation)
        if match:
            data_type, shape_str = match.groups()
            shape = tuple(shape_str.split(","))
            dtype = dtype_map[data_type]
            # pyrefly: ignore [bad-argument-type]
            kwargs[param] = gen_tensor(shape, dtype)

        match = re.search(sym_shape_regex, annotation)
        if match:
            kwargs[param] = get_sym_int(match.group(1))

    if "self" in inspect.signature(func).parameters:
        container = TensorContainer()
        kwargs["self"] = container
        for match in re.finditer(tensor_assignment_regex, source):
            attr_name, data_type, shape_str, _ = match.groups()
            shape = tuple(shape_str.split(","))
            dtype = dtype_map[data_type]
            # pyrefly: ignore [bad-argument-type]
            setattr(container, attr_name, gen_tensor(shape, dtype))

    return kwargs


def profile_to_file(filename: str) -> Callable[[T], T]:
    """
    Decorator to cProfile a given function and save the result to disk on process exit.

    Args:
        filename: filename to save profile to
    """
    prof = cProfile.Profile()
    filename = os.path.abspath(os.path.expanduser(filename))
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 955-980
```python
    def decorator(fn: Any) -> Any:
        @functools.wraps(fn)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            prof.enable()
            try:
                return fn(*args, **kwargs)
            finally:
                prof.disable()

        return wrapper

    def save_it() -> None:
        prof.dump_stats(filename)
        sys.stderr.write(
            textwrap.dedent(
                f"""\
                Wrote profile to {filename}, view with:

                    snakeviz {filename}

                """
            )
        )

    atexit.register(save_it)
    return decorator
```
- **EN**: These decorators register or transform the following definition so it can capture Python execution for torch.compile and maintain compiler state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够为 torch.compile 捕获 Python 执行并维护编译器状态。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._prims_common`, `torch._subclasses.meta_utils`, `torch._dynamo.testing`, `torch._inductor.cpp_builder`, `torch.multiprocessing.reductions`, `torch.utils._content_store`, `.`, `.utils`, `torch.hub`, `torch.storage`, `torch._functorch.config`, `torch._inductor.config`, `.testing`, `torch.utils._pytree`
- **Standard library / 标准库**: `__future__`, `atexit`, `copy`, `cProfile`, `functools`, `getpass`, `inspect`, `itertools`, `logging`, `os`, `re`, `subprocess`, `sys`, `tempfile`, `textwrap`
- **Third-party packages / 第三方包**: `libfb.py.build_info`
- **Primary symbols / 核心符号**: `T`, `BUCK_CMD_PREFIX`, `BuckTargetWriter`, `minifier_dir`, `MAX_CONSTANT_NUMEL_INLINE`, `NNModuleToString`, `_cuda_system_info_comment`, `generate_env_vars_string`, `generate_config_string`, `get_minifier_repro_path`, `helper_for_dump_minify`, `AccuracyError`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
