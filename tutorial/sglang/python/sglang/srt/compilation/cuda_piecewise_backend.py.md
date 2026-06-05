# cuda_piecewise_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/compilation/cuda_piecewise_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the compilation and graph lowering part of the SRT runtime and implements logic centered on `cuda_piecewise_backend`. It exposes primary entry points such as `ConcreteSizeEntry`, `CUDAPiecewiseBackend`. / 该模块属于 SRT 运行时的编译与图优化部分，主要实现围绕 `cuda_piecewise_backend` 的逻辑。 它对外提供的主要入口包括 `ConcreteSizeEntry`, `CUDAPiecewiseBackend`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.10.0/vllm/compilation/cuda_piecewise_backend.py

import dataclasses
import logging
from contextlib import ExitStack
from typing import Any, Callable, Optional
from unittest.mock import patch

import torch
import torch.fx as fx

from sglang.srt.compilation.compilation_config import CompilationConfig
from sglang.srt.compilation.compilation_counter import compilation_counter
from sglang.srt.compilation.piecewise_context_manager import (
    get_pcg_capture_stream,
    is_in_pcg_torch_compile,
)
from sglang.srt.compilation.weak_ref_tensor import weak_ref_tensors

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; prepares compilation-related behavior.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理与编译相关的行为。

### Lines 25-39: Class ConcreteSizeEntry
```python
@dataclasses.dataclass
class ConcreteSizeEntry:
    runtime_shape: int
    need_to_compile: bool  # the size is in compile_sizes
    use_cudagraph: bool  # the size is in cudagraph_capture_sizes

    compiled: bool = False
    runnable: Callable = None  # type: ignore
    num_finished_warmup: int = 0
    cudagraph: Optional[torch.cuda.CUDAGraph] = None
    output: Optional[Any] = None

    # for cudagraph debugging, track the input addresses
    # during capture, and check if they are the same during replay
    input_addresses: Optional[list[int]] = None
```
**EN:** This range introduces `ConcreteSizeEntry` and defines the structure or metadata that its methods rely on. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一段引入 `ConcreteSizeEntry`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 42-43: Class CUDAPiecewiseBackend
```python
class CUDAPiecewiseBackend:

```
**EN:** This range introduces `CUDAPiecewiseBackend` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `CUDAPiecewiseBackend`，并定义其后续方法依赖的结构或元数据。

### Lines 44-101: Method CUDAPiecewiseBackend.__init__
```python
    def __init__(
        self,
        graph: fx.GraphModule,
        compile_config: CompilationConfig,
        inductor_config: dict[str, Any],
        graph_pool: Any,
        piecewise_compile_index: int,
        total_piecewise_compiles: int,
        sym_shape_indices: list[int],
        compiled_graph_for_general_shape: Callable,
        sglang_backend,
    ):
        """
        The backend for piecewise compilation.
        It mainly handles the compilation and cudagraph capturing.

        We will compile `self.graph` once for the general shape,
        and then compile for different shapes specified in
        `compilation_config.compile_sizes`.

        Independently, we will capture cudagraph for different shapes.

        If a shape needs both compilation and cudagraph, we will
        compile it first, and then capture cudagraph.
        """
        self.graph = graph
        self.inductor_config = inductor_config
        self.graph_pool = graph_pool
        self.piecewise_compile_index = piecewise_compile_index
        self.total_piecewise_compiles = total_piecewise_compiles
        self.sglang_backend = sglang_backend

        self.is_first_graph = piecewise_compile_index == 0
        self.is_last_graph = piecewise_compile_index == total_piecewise_compiles - 1

        self.compile_sizes: set[int] = set([])
        self.compile_config = compile_config
        self.cudagraph_capture_sizes: set[int] = set(compile_config.get_capture_sizes())

        self.first_run_finished = False

        self.compiled_graph_for_general_shape = compiled_graph_for_general_shape  # noqa

        self.sym_shape_indices = sym_shape_indices

        # the entries for different shapes that we need to either
        # compile or capture cudagraph
        self.concrete_size_entries: dict[int, ConcreteSizeEntry] = {}

        # to_be_compiled_sizes tracks the remaining sizes to compile,
        # and updates during the compilation process, so we need to copy it
        self.to_be_compiled_sizes: set[int] = self.compile_sizes.copy()
        for shape in self.compile_sizes.union(self.cudagraph_capture_sizes):
            self.concrete_size_entries[shape] = ConcreteSizeEntry(
                runtime_shape=shape,
                need_to_compile=shape in self.compile_sizes,
                use_cudagraph=shape in self.cudagraph_capture_sizes,
            )
```
**EN:** This callable implements `CUDAPiecewiseBackend.__init__`. It takes `graph`, `compile_config`, `inductor_config`, `graph_pool` and mainly initializes instance state and defaults. The docstring states: "The backend for piecewise compilation." In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CUDAPiecewiseBackend.__init__`。它接收 `graph`, `compile_config`, `inductor_config`, `graph_pool`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 103-107: Method CUDAPiecewiseBackend.check_for_ending_compilation
```python
    def check_for_ending_compilation(self):
        if self.is_last_graph and not self.to_be_compiled_sizes:
            # no specific sizes to compile
            # save the hash of the inductor graph for the next run
            self.sglang_backend.compiler_manager.save_to_file()
```
**EN:** This callable implements `CUDAPiecewiseBackend.check_for_ending_compilation` and mainly checks preconditions and compatibility. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CUDAPiecewiseBackend.check_for_ending_compilation`，主要用于检查前置条件与兼容性。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 109-168: Method CUDAPiecewiseBackend.__call__ (part 1/2)
```python
    def __call__(self, *args) -> Any:
        if not self.first_run_finished:
            self.first_run_finished = True
            self.check_for_ending_compilation()
            return self.compiled_graph_for_general_shape(*args)

        if len(self.sym_shape_indices) == 0:
            return self.compiled_graph_for_general_shape(*args)

        runtime_shape = args[self.sym_shape_indices[0]]
        if runtime_shape not in self.concrete_size_entries:
            # we don't need to do anything for this shape
            return self.compiled_graph_for_general_shape(*args)

        entry = self.concrete_size_entries[runtime_shape]

        if entry.runnable is None:
            entry.runnable = self.compiled_graph_for_general_shape

        if entry.need_to_compile and not entry.compiled:
            entry.compiled = True
            self.to_be_compiled_sizes.remove(runtime_shape)
            # args are real arguments
            entry.runnable = self.sglang_backend.compiler_manager.compile(
                self.graph,
                args,
                self.inductor_config,
                graph_index=self.piecewise_compile_index,
                num_graphs=self.total_piecewise_compiles,
                runtime_shape=runtime_shape,
            )

            # finished compilations for all required shapes
            if self.is_last_graph and not self.to_be_compiled_sizes:
                self.check_for_ending_compilation()

        if is_in_pcg_torch_compile():
            return entry.runnable(*args)

        if entry.cudagraph is None:
            if entry.num_finished_warmup < 1:  # noqa
                entry.num_finished_warmup += 1
                return entry.runnable(*args)

            if self.compile_config.get_enable_debug_mode():
                input_addresses = [
                    x.data_ptr() for x in args if isinstance(x, torch.Tensor)
                ]
                entry.input_addresses = input_addresses
            cudagraph = torch.cuda.CUDAGraph()

            with ExitStack() as stack:
                if not self.is_first_graph:
                    # during every model forward, we will capture
                    # many pieces of cudagraphs (roughly one per layer).
                    # running gc again and again across layers will
                    # make the cudagraph capture very slow.
                    # therefore, we only run gc for the first graph,
                    # and disable gc for the rest of the graphs.
                    stack.enter_context(patch("gc.collect", lambda: None))
```
**EN:** This callable implements `CUDAPiecewiseBackend.__call__`. It takes `*args` and mainly handles invocation-time behavior. This chunk is part 1 of 2 for the same logical block. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CUDAPiecewiseBackend.__call__`。它接收 `*args`，主要用于处理调用时的行为。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 169-208: Method CUDAPiecewiseBackend.__call__ (part 2/2)
```python
                    stack.enter_context(patch("torch.cuda.empty_cache", lambda: None))
                # mind-exploding: carefully manage the reference and memory.
                stream = get_pcg_capture_stream()
                assert (
                    stream is not None
                ), "PCG capture stream is not set, please check if runtime recompilation happened"
                with torch.cuda.graph(cudagraph, pool=self.graph_pool, stream=stream):
                    # `output` is managed by pytorch's cudagraph pool
                    output = entry.runnable(*args)
                    if self.is_last_graph:
                        # by converting it to weak ref,
                        # the original `output` will immediately be released
                        # to save memory. It is only safe to do this for
                        # the last graph, because the output of the last graph
                        # will not be used by any other cuda graph.
                        output = weak_ref_tensors(output)

            # here we always use weak ref for the output
            # to save memory
            entry.output = weak_ref_tensors(output)
            entry.cudagraph = cudagraph

            compilation_counter.num_cudagraph_captured += 1

            # important: we need to return the output, rather than
            # the weak ref of the output, so that pytorch can correctly
            # manage the memory during cuda graph capture
            return output

        if self.compile_config.get_enable_debug_mode():
            # check if the input addresses are the same
            new_input_addresses = [
                x.data_ptr() for x in args if isinstance(x, torch.Tensor)
            ]
            assert new_input_addresses == entry.input_addresses, (
                "Input addresses for cudagraphs are different during replay."
                f" Expected {entry.input_addresses}, got {new_input_addresses}"
            )
        entry.cudagraph.replay()
        return entry.output
```
**EN:** This callable implements `CUDAPiecewiseBackend.__call__`. It takes `*args` and mainly handles invocation-time behavior. This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CUDAPiecewiseBackend.__call__`。它接收 `*args`，主要用于处理调用时的行为。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查；处理与编译相关的行为；管理图捕获或回放逻辑。

## Key Concepts / 关键概念
- `ConcreteSizeEntry`: core class or state container / 核心类或状态容器
- `CUDAPiecewiseBackend`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `logging`, `contextlib`, `typing`, `unittest.mock`
- **Third-party / 第三方**: `torch`, `torch.fx`
- **Internal modules / 内部模块**: `sglang.srt.compilation.compilation_config`, `sglang.srt.compilation.compilation_counter`, `sglang.srt.compilation.piecewise_context_manager`, `sglang.srt.compilation.weak_ref_tensor`
