# aoti.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/repro/aoti.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Contains reproduction, minification, and debugging helpers for compiler failures.
- **Purpose (CN)**: 包含面向编译失败的复现、最小化与调试辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
"""
Utilities for debugging and reproducing issues in Ahead of Time with Inductor (AOTI) compilation.

This file provides tools and utilities for:
- Generating minimal reproducible test cases (minification)
- Handling exported programs and graph modules
- Creating debug repros for AOTI compilation issues
- Supporting both accuracy testing and error reproduction
- Managing configuration and environment for repro cases

The main components include:
- Minification tools to reduce test cases while preserving errors
- Repro generation utilities for exported programs
- Error handling specific to AOTI compilation
- Command-line interface for running and managing repros
"""

import argparse
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 19-36
```python
import functools
import io
import logging
import os
import re
import shutil
import sys
import textwrap
from collections.abc import Sequence
from importlib import import_module
from typing import Any, IO

import torch
from torch._dynamo.debug_utils import (
    _cuda_system_info_comment,
    BuckTargetWriter,
    extra_imports,
    generate_config_string,
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 37-62
```python
    generate_env_vars_string,
    helper_for_dump_minify,
    InputReader,
    minifier_dir,
    NNModuleToString,
    NopInputReader,
)
from torch.export import ExportedProgram
from torch.hub import tqdm


log = logging.getLogger(__name__)


inductor_config = import_module("torch._inductor.config")
use_buck = inductor_config.is_fbcode()


class AOTIMinifierError(Exception):
    def __init__(self, original_exception: str | Exception) -> None:
        additional_message = "This error is caused by a bug in the AOTI minifier, please report a bug to PyTorch"
        full_message = f"{additional_message}: {str(original_exception)}"
        super().__init__(full_message)
        self.original_exception = original_exception
```
- **EN**: This module-level block helps organize reusable module behavior and state.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。

### Lines 63-80
```python
def dump_to_minify(
    exported_program: ExportedProgram,
    compiler_name: str,
    command: str = "minify",
    options: dict[str, Any] | None = None,
) -> None:
    """
    If command is "minify":
        Dump exported_program to `debug_dir/minifier/minifier_launcher.py`, with minify command.
    If command is "run":
        Dump exported_program to `cwd/repro.py`, with run command.
    """
    assert command in ["minify", "run"]

    subdir = os.path.join(minifier_dir(), "checkpoints")
    if not os.path.exists(subdir):
        os.makedirs(subdir, exist_ok=True)
```
- **EN**: Defines the `dump_to_minify` function; this block introduces logic that hand work to a compiler/backend pipeline.
- **CN**: 定义`dump_to_minify` 函数；该代码块引入了用于将工作移交给编译器或后端流水线的逻辑。

### Lines 81-98
```python
    if command == "minify":
        out = io.StringIO()
        save_graph_repro_ep(
            out,
            compiler_name,
            exported_program=exported_program,
            save_dir=subdir,
            command="minify",
            config_patches=options,
        )
        return helper_for_dump_minify(out.getvalue())
    else:
        curdir = os.getcwd()
        file_name = os.path.join(curdir, "repro.py")
        try:
            with open(file_name, "w") as fd:
                save_graph_repro_ep(
                    fd,
```
- **EN**: This block continues `dump_to_minify` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `dump_to_minify`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 99-116
```python
                    compiler_name,
                    exported_program=exported_program,
                    config_patches=options,
                    save_dir=subdir,
                    command="run",
                    module_in_comment=True,
                )
            log.warning("Writing repro file to %s", file_name)
            if use_buck:
                BuckTargetWriter(file_name).write()
        except OSError:
            log.warning("No write permissions for %s", file_name)


def get_module_string(gm: torch.fx.GraphModule) -> str:
    def _convert_to_comment(s_: str) -> str:
        s = s_.split("\n")
        if len(s) == 1:
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 117-134
```python
            return "# " + s_
        first = s.pop(0)
        for i in range(len(s)):
            line = s[i]
            if line.strip() != "":
                s[i] = "# " + line
            else:
                s[i] = ""
        s = "\n".join(s)
        s = first + "\n" + s
        return s

    module_string = NNModuleToString.convert(gm)
    return _convert_to_comment(module_string)


def save_graph_repro_ep(
    fd: IO[Any],
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 135-159
```python
    compiler_name: str,
    *,
    exported_program: ExportedProgram | None = None,
    gm: torch.nn.Module | None = None,
    args: tuple[Any] | None = None,
    config_patches: dict[str, str] | None = None,
    stable_output: bool = False,
    save_dir: str | None = None,
    command: str = "run",
    accuracy: str | bool | None = None,
    check_str: str | None = None,
    module_in_comment: bool = False,
    strict: bool = False,
) -> None:
    # Save graph for reproducing the error.
    # Either exported_program or gm will be saved, depending on which one is defined.
    # Only one of exported_program and gm should be defined.

    if exported_program is None and gm is None:
        raise AOTIMinifierError("One of exported_program and gm must be defined")
    if exported_program is not None and gm is not None:
        raise AOTIMinifierError("Only one of exported_program and gm can be defined")
    if gm is not None and args is None:
        raise AOTIMinifierError("If gm is defined, args should also be defined")
```
- **EN**: This block continues `save_graph_repro_ep` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `save_graph_repro_ep`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 160-177
```python
    if exported_program is None:
        assert gm is not None
        assert args is not None
        exported_program = torch.export.export(gm, args, strict=strict)
    elif gm is None:
        gm = exported_program.module(check_guards=False)

    # save a graph preview using gm
    module_string = get_module_string(gm)  # type: ignore[arg-type]
    fd.write(module_string)

    # save a graph repro using exported_program
    fd.write(
        generate_compiler_repro_exported_program(
            exported_program,
            options=config_patches,
            stable_output=stable_output,
            save_dir=save_dir,
```
- **EN**: This block continues `save_graph_repro_ep` and works to enforce guards that validate whether cached compiled code can be reused. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `save_graph_repro_ep`，用于实施守卫检查以判断缓存的编译代码能否复用。 同时它还会根据运行时条件分支处理。

### Lines 178-195
```python
        )
    )
    if accuracy is None:
        accuracy = "_accuracy" in compiler_name
    fd.write("if __name__ == '__main__':\n")
    fd.write("    from torch._dynamo.repro.aoti import run_repro\n")
    fd.write(
        f"    with torch.no_grad():\n"
        f"        run_repro(exported_program, config_patches=config_patches, accuracy={accuracy!r}, command={command!r}, "
        f"save_dir={save_dir!r}, check_str={check_str!r})\n"
    )


def dump_compiler_graph_state(
    gm: torch.fx.GraphModule,
    args: Sequence[Any],
    compiler_name: str,
    *,
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 196-213
```python
    config_patches: dict[str, str] | None = None,
    accuracy: str | bool | None = None,
    strict: bool = False,
) -> None:
    subdir = os.path.join(minifier_dir(), "checkpoints")
    if not os.path.exists(subdir):
        os.makedirs(subdir, exist_ok=True)
    file_name = os.path.join(subdir, f"{len(gm.graph.nodes)}.py")
    log.warning(
        "Writing checkpoint with %s nodes to %s", len(gm.graph.nodes), file_name
    )
    with open(file_name, "w") as fd:
        save_graph_repro_ep(
            fd,
            compiler_name,
            gm=gm,
            args=tuple(args),
            config_patches=config_patches,
```
- **EN**: This block continues `dump_compiler_graph_state` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `dump_compiler_graph_state`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 214-234
```python
            save_dir=subdir,
            accuracy=accuracy,
            module_in_comment=True,
            strict=strict,
        )
    curdir = os.getcwd()
    repro_path = os.path.join(curdir, "repro.py")
    try:
        shutil.copyfile(file_name, repro_path)
        log.warning("Copying repro file for convenience to %s", repro_path)
        if use_buck:
            BuckTargetWriter(file_name).write()
    except OSError:
        log.warning("No write permissions for %s", repro_path)


# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #
#                           DUMP REPROS
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ #
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 235-253
```python
def generate_compiler_repro_exported_program(
    exported_program: ExportedProgram,
    *,
    options: dict[str, str] | None = None,
    stable_output: bool = False,
    save_dir: str | None = None,
) -> str:
    model_str = textwrap.dedent(
        f"""
{generate_env_vars_string(stable_output=stable_output)}
import torch
import torch._inductor.inductor_prims

{generate_config_string(stable_output=stable_output)}

isolate_fails_code_str = None

{extra_imports}
```
- **EN**: Defines the `generate_compiler_repro_exported_program` function; this block introduces logic that hand work to a compiler/backend pipeline.
- **CN**: 定义`generate_compiler_repro_exported_program` 函数；该代码块引入了用于将工作移交给编译器或后端流水线的逻辑。

### Lines 254-274
```python
        """
    )
    if not stable_output:
        model_str += f"# torch version: {torch.version.__version__}\n"
        if hasattr(torch.version, "cuda"):
            model_str += f"# torch cuda version: {torch.version.cuda}\n"
        if hasattr(torch.version, "git_version"):
            model_str += f"# torch git version: {torch.version.git_version}\n\n\n"
        model_str += _cuda_system_info_comment()
    if save_dir:
        ep_path = os.path.join(save_dir, "exported_program.pt2")
    else:
        ep_path = "exported_program.pt2"
    torch.export.save(exported_program, ep_path)

    model_str += f"exported_program = torch.export.load('{ep_path}')\n"
    model_str += "# print(exported_program.graph)\n"
    model_str += f"config_patches={options}\n"
    return model_str
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 275-300
```python
def repro_load_args(load_args: Any, save_dir: str | None) -> tuple[Any]:
    if not hasattr(load_args, "_version"):
        log.warning(
            "load_args does not have a _version attribute, please file a bug to PyTorch "
            "and describe how you generate this repro script"
        )
    else:
        if load_args._version > 0:
            log.warning(
                "load_args is version %s, but this version of PyTorch only supports "
                "version 0.  We will try to run it anyway but there may be an incompatibility; "
                "if so, try upgrading your version of PyTorch.",
                load_args._version,
            )

    nop_reader = NopInputReader()
    load_args(nop_reader)

    with tqdm(desc="Loading inputs", total=nop_reader.total) as pbar:
        input_reader = InputReader(save_dir=save_dir, pbar=pbar)
        load_args(input_reader)
        args = input_reader.args

    return tuple(args)
```
- **EN**: This module-level block helps connect execution with differentiation-aware logic. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于将执行过程与可微分逻辑连接起来。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 301-326
```python
def repro_common(
    options: Any, exported_program: ExportedProgram
) -> tuple[torch.fx.GraphModule, Any, Any]:
    # pyrefly: ignore [bad-assignment]
    torch._inductor.config.generate_intermediate_hooks = True
    mod = exported_program.module(check_guards=False)
    args, kwargs = exported_program.example_inputs
    return mod, args, kwargs  # type: ignore[return-value]


def repro_get_args(
    options: Any,
    exported_program: ExportedProgram,
    config_patches: dict[str, Any] | None,
) -> tuple[torch.fx.GraphModule, Any, Any]:
    mod, args, kwargs = repro_common(options, exported_program)
    return mod, args, kwargs


def repro_run(
    options: Any,
    exported_program: ExportedProgram,
    config_patches: dict[str, Any] | None,
) -> None:
    from torch._inductor import _aoti_compile_and_package_inner
```
- **EN**: This module-level block helps enforce guards that validate whether cached compiled code can be reused. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于实施守卫检查以判断缓存的编译代码能否复用。 同时它还会计算并返回中间值或结果。

### Lines 327-350
```python
    gm, args, kwargs = repro_common(options, exported_program)

    from torch.cuda import synchronize

    _aoti_compile_and_package_inner(
        gm,
        args,
        kwargs,
        load_and_run=True,
        check_accuracy=options.accuracy,
        inductor_configs=config_patches,
    )

    need_sync = False

    for arg in args:
        if isinstance(arg, torch.Tensor) and arg.is_cuda:
            need_sync = True
            break

    if need_sync:
        synchronize()  # ensure segfaults are surfaced
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 351-369
```python
def export_for_aoti_minifier(
    gm: torch.nn.Module,
    tuple_inputs: tuple[Any],
    strict: bool = False,
    skip_export_error: bool = True,
) -> torch.nn.Module | None:
    # Some graphs cannot be used for AOTI/export (illegal graphs), these should be
    # considered as graphs that don't fail in the minifier, so the minifier keeps searching.
    # In these case, we return None. Otherwise, we return the exported graph module.
    # This won't affect the minifier result because the minifier is only responsible for catching
    # errors in AOTI, not export.
    #
    # Please add to this list of illegal graphs if you change the implementation here.
    # - graph output is not allowed by export
    #
    # If skip_export_error=True, then the errors in export will not be raised, and the minifier
    # will keep exploring and ignore this graph.
    from torch._dynamo.exc import UserError, UserErrorType
```
- **EN**: Defines the `export_for_aoti_minifier` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`export_for_aoti_minifier` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 370-390
```python
    try:
        ep = torch.export.export(gm, tuple_inputs, strict=strict)
        gm = ep.module(check_guards=False)
        return gm
    except Exception as e:
        if skip_export_error:
            return None
        if isinstance(e, UserError) and e.error_type == UserErrorType.INVALID_OUTPUT:
            # graph output is not allowed by export when strict=True
            return None
        if isinstance(e, RuntimeError):
            # graph output is not allowed by export when strict=False
            pattern = r"Found .* in output, which is not a known type\."
            if re.search(pattern, str(e)) is not None:
                return None
        raise AOTIMinifierError(e) from e
    # we should never reach here
    # pyrefly: ignore [unreachable]
    return None
```
- **EN**: This module-level block helps enforce guards that validate whether cached compiled code can be reused. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于实施守卫检查以判断缓存的编译代码能否复用。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 391-414
```python
def repro_minify(
    options: Any,
    exported_program: ExportedProgram,
    config_patches: dict[str, Any] | None,
) -> None:
    from functorch.compile import minifier
    from torch._inductor import _aoti_compile_and_package_inner
    from torch._inductor.compile_fx import _aoti_flatten_inputs

    mod, args, kwargs = repro_common(options, exported_program)

    # update serialized_in_spec and serialized_out_spec
    flat_example_inputs, inductor_configs = _aoti_flatten_inputs(
        mod, args, kwargs, options=config_patches
    )
    compiler_name = "aot_inductor"
    assert options.minifier_export_mode in ["dynamo", "python"]
    strict = options.minifier_export_mode == "dynamo"
    skip_export_error = options.skip_export_error

    from torch.cuda import synchronize

    need_sync = False
```
- **EN**: Defines the `repro_minify` function; this block introduces logic that hand work to a compiler/backend pipeline.
- **CN**: 定义`repro_minify` 函数；该代码块引入了用于将工作移交给编译器或后端流水线的逻辑。

### Lines 415-438
```python
    for arg in args:
        if isinstance(arg, torch.Tensor) and arg.is_cuda:
            need_sync = True
            break

    def module_fails(
        gm: torch.fx.GraphModule,
        flat_example_inputs: list[Any],
        check_str: str | None = None,
    ) -> bool:
        # Need to export first so the in_spec and out_spec are populated
        tuple_inputs = tuple(flat_example_inputs)
        # pyrefly: ignore [bad-assignment]
        gm = export_for_aoti_minifier(
            gm, tuple_inputs, strict=strict, skip_export_error=skip_export_error
        )

        # Some graphs cannot be used for AOTI/export (illegal graphs), these should be
        # considered as graphs that don't fail in the minifier, so the minifier keeps searching.
        if gm is None:
            return False

        assert isinstance(gm, torch.fx.GraphModule)
```
- **EN**: Defines the `repro_minify` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`repro_minify` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 439-456
```python
        try:
            _aoti_compile_and_package_inner(
                gm,
                tuple_inputs,
                load_and_run=True,
                check_accuracy=options.accuracy,
                inductor_configs=inductor_configs,
            )
            if need_sync:
                synchronize()  # ensure segfaults are surfaced
            return False
        except Exception as e:
            if check_str is not None and check_str not in repr(e):
                return False
            return True

    minifier(
        mod,
```
- **EN**: This block continues `repro_minify` and works to hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `repro_minify`，用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 457-474
```python
        flat_example_inputs,
        module_fails=functools.partial(module_fails, check_str=options.check_str),
        dump_state=functools.partial(
            dump_compiler_graph_state,
            compiler_name=compiler_name,
            config_patches=config_patches,
            accuracy=options.accuracy,
            strict=strict,
        ),
        save_dir=options.save_dir,
        offload_to_disk=options.offload_to_disk,
        skip_offload=options.skip_saving_eager_intermediates,
        skip_sanity=options.skip_sanity,
        max_granularity=options.max_granularity,
    )


def run_repro(
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 475-497
```python
    exported_program: ExportedProgram,
    *,
    config_patches: dict[str, str] | None = None,
    command: str = "run",
    accuracy: bool | str = "",
    save_dir: str | None = None,
    tracing_mode: str | None = None,
    check_str: str | None = None,
    minifier_export_mode: str = "python",
    skip_export_error: bool = True,
    **more_kwargs: Any,
) -> Any:
    for k in more_kwargs:
        log.warning(
            "Unrecognized kwarg %s; perhaps this repro was made on a newer version of PyTorch",
            k,
        )

    if accuracy is True:
        accuracy = "accuracy"
    elif accuracy is False:
        accuracy = ""
```
- **EN**: This block continues `run_repro` and works to validate invariants and surface meaningful failures. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `run_repro`，用于校验不变量并给出有意义的失败信息。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 498-515
```python
    parser = argparse.ArgumentParser(
        description=f"""\
An AOTI repro script, typically triggering a bug in PyTorch AOTInductor.
When run with no arguments, this script defaults to running '{command}'.
Extra flags may be available; to find out more, try '{command} --help'.
There are also alternate subcommands available, see below.

default settings on this script:
  {accuracy=}
  {tracing_mode=}
  {save_dir=}
  {check_str=}
""",
        formatter_class=argparse.RawTextHelpFormatter,
    )

    def common_flags(parser: argparse.ArgumentParser) -> None:
        accuracy_group = parser.add_mutually_exclusive_group()
```
- **EN**: Defines the `run_repro` function; this block introduces logic that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 定义`run_repro` 函数；该代码块引入了用于为 torch.compile 捕获 Python 执行并维护编译器状态的逻辑。

### Lines 516-533
```python
        accuracy_group.add_argument(
            "--no-accuracy",
            dest="accuracy",
            action="store_const",
            const="",
            default=accuracy,
            help="do not test accuracy, just run the module and see if it errors",
        )
        accuracy_group.add_argument(
            "--accuracy",
            action="store_const",
            const="accuracy",
            default=accuracy,
            help="""\
test if the RMSE between the compiled module and the fp64 reference is greater
than eager and the fp64 reference. This is usually more reliable than the
standard allclose test, as we expect numeric differences from compiling, often
improving accuracy over eager.  RMSE test allows for compiled module to
```
- **EN**: This block continues `run_repro.common_flags` and works to hand work to a compiler/backend pipeline.
- **CN**: 该代码块继续实现 `run_repro.common_flags`，用于将工作移交给编译器或后端流水线。

### Lines 534-551
```python
diverge greatly from eager, as long as this divergence moves it closer to the
'true' mathematical value of the network.  Caveats: (1) double precision can
still suffer from rounding error, so it is not a perfect reference (see for
example 'Herbie: Automatically Improving Floating Point Accuracy') for
approaches that detect the necessary working precision and compute it in
arbitrary precision floating point; unfortunately, this is not practical for
tensor computation; (2) if there are not enough samples in the output being
compared, we may get unlucky and have an unlucky greater RMSE than eager; this
could be overcome by applying a more rigorous statistical test at some
p-value, which we leave for future work.
""",
        )
        accuracy_group.add_argument(
            "--strict-accuracy",
            dest="accuracy",
            action="store_const",
            const="strict_accuracy",
            default=accuracy,
```
- **EN**: This block continues `run_repro.common_flags` and works to validate invariants and surface meaningful failures.
- **CN**: 该代码块继续实现 `run_repro.common_flags`，用于校验不变量并给出有意义的失败信息。

### Lines 552-578
```python
            help="""\
by default, when doing accuracy minification we will reject reductions which
change the divergence from a floating point divergence to a integral/boolean
divergence.  This is because some operations like ReLU involve temporarily
sharp boundaries that smooth out again afterwards; without requiring
divergence on floating point, the minifier will often fixate on divergent
boolean tensor even though this is not the true source of the divergence.
However, rejecting these reductions makes it more difficult for the minifier
to make process.  Using this option will let the minifier progress for ALL
divergences--you just might not end up with a useful repro in the end.""",
        )

        parser.add_argument(
            "--save-dir",
            type=str,
            default=save_dir,
            metavar="DIR",
            help="directory where saved inputs live",
        )
        parser.add_argument(
            "--no-save-dir",
            dest="save_dir",
            action="store_const",
            const=None,
            help="don't use any directory for saved inputs",
        )
```
- **EN**: This block continues `run_repro` and works to capture Python execution for torch.compile and maintain compiler state.
- **CN**: 该代码块继续实现 `run_repro`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 579-596
```python
    subparsers = parser.add_subparsers(
        dest="command", metavar="{run,minify}", required=True
    )

    parser_run = subparsers.add_parser(
        "run",
        help="just run the repro",
    )
    common_flags(parser_run)

    parser_minify = subparsers.add_parser(
        "minify", help="run the minifier on the repro"
    )
    common_flags(parser_minify)
    parser_get_args = subparsers.add_parser("get_args", help="get the args")
    common_flags(parser_get_args)
    parser_minify.add_argument(
        "--skip-saving-eager-intermediates",
```
- **EN**: This block continues `run_repro` and works to capture Python execution for torch.compile and maintain compiler state.
- **CN**: 该代码块继续实现 `run_repro`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 597-614
```python
        action="store_true",
        help="skip saving eager intermediates on --minify",
    )
    parser_minify.add_argument(
        "--offload-to-disk",
        action="store_true",
        help="during minification, offload delta debugging intermediates to disk.  Use if you're OOMing",
    )
    parser_minify.add_argument(
        "--skip-sanity",
        action="store_true",
        help="skip sanity check at beginning of minification on original graph",
    )
    parser_minify.add_argument(
        "--max-granularity",
        type=int,
        default=None,
        help="start at this granularity and work down; must be power of 2",
```
- **EN**: This block continues `run_repro` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `run_repro`，用于构建、遍历或改写图结构及其元数据。

### Lines 615-637
```python
    )
    parser_minify.add_argument(
        "--check-str",
        type=str,
        default=check_str,
        help="require minified program to fail with error containing this string",
    )
    parser_minify.add_argument(
        "--minifier-export-mode",
        type=str,
        default=minifier_export_mode,
        help=(
            "The export mode used in minifier, either dynamo or python."
            "`dynamo` corresponds to strict=True, and `python` corresponds to strict=False."
        ),
    )
    parser_minify.add_argument(
        "--skip-export-error",
        type=bool,
        default=skip_export_error,
        help="Skip intermediate graphs that cannot be exported.",
    )
```
- **EN**: This block continues `run_repro` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `run_repro`，用于构建、遍历或改写图结构及其元数据。

### Lines 638-655
```python
    # Run the repro in the context of minification, inverting exit code meaning
    parser_minifier_query = subparsers.add_parser(
        "minifier-query",
    )
    common_flags(parser_minifier_query)
    parser_minifier_query.add_argument(
        "--check-str",
        type=str,
        default=check_str,
        help="require minified program to fail with error containing this string",
    )

    args = None
    if len(sys.argv) <= 1:
        args = [command, *sys.argv[1:]]

    options = parser.parse_args(args)
    COMMAND_FNS = {
```
- **EN**: This block continues `run_repro` and works to validate invariants and surface meaningful failures. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `run_repro`，用于校验不变量并给出有意义的失败信息。 同时它还会根据运行时条件分支处理。

### Lines 656-662
```python
        "minify": repro_minify,
        "run": repro_run,
        "get_args": repro_get_args,
    }
    return COMMAND_FNS[options.command](
        options, exported_program, config_patches=config_patches
    )
```
- **EN**: This block continues `run_repro` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `run_repro`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._dynamo.debug_utils`, `torch.export`, `torch.hub`, `torch._inductor`, `torch.cuda`, `torch._dynamo.exc`, `torch._inductor.compile_fx`
- **Standard library / 标准库**: `argparse`, `functools`, `io`, `logging`, `os`, `re`, `shutil`, `sys`, `textwrap`, `collections.abc`, `importlib`, `typing`
- **Third-party packages / 第三方包**: `functorch.compile`
- **Primary symbols / 核心符号**: `AOTIMinifierError`, `dump_to_minify`, `get_module_string`, `save_graph_repro_ep`, `dump_compiler_graph_state`, `generate_compiler_repro_exported_program`, `repro_load_args`, `repro_common`, `repro_get_args`, `repro_run`, `export_for_aoti_minifier`, `repro_minify`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
