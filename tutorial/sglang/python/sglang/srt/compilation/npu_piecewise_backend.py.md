# npu_piecewise_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/compilation/npu_piecewise_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the compilation and graph lowering part of the SRT runtime and implements logic centered on `npu_piecewise_backend`. It exposes primary entry points such as `NPUPiecewiseBackend`. / 该模块属于 SRT 运行时的编译与图优化部分，主要实现围绕 `npu_piecewise_backend` 的逻辑。 它对外提供的主要入口包括 `NPUPiecewiseBackend`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Module imports, constants, and setup
```python
from contextlib import ExitStack
from typing import Any, Callable
from unittest.mock import patch

import torch
import torch.fx as fx

from sglang.srt.compilation.compilation_config import CompilationConfig
from sglang.srt.compilation.compilation_counter import compilation_counter
from sglang.srt.compilation.cuda_piecewise_backend import (
    CUDAPiecewiseBackend,
    weak_ref_tensors,
)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 16-16: Class NPUPiecewiseBackend
```python
class NPUPiecewiseBackend(CUDAPiecewiseBackend):
```
**EN:** This range introduces `NPUPiecewiseBackend` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `NPUPiecewiseBackend`，并定义其后续方法依赖的结构或元数据。

### Lines 17-39: Method NPUPiecewiseBackend.__init__
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
        super().__init__(
            graph,
            compile_config,
            inductor_config,
            graph_pool,
            piecewise_compile_index,
            total_piecewise_compiles,
            sym_shape_indices,
            compiled_graph_for_general_shape,
            sglang_backend,
        )
```
**EN:** This callable implements `NPUPiecewiseBackend.__init__`. It takes `graph`, `compile_config`, `inductor_config`, `graph_pool` and mainly initializes instance state and defaults. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `NPUPiecewiseBackend.__init__`。它接收 `graph`, `compile_config`, `inductor_config`, `graph_pool`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 41-100: Method NPUPiecewiseBackend.__call__ (part 1/2)
```python
    def __call__(self, *args):
        runtime_shape = args[self.sym_shape_indices[0]]
        if runtime_shape not in self.concrete_size_entries:
            # we don't need to do anything for this shape
            return self.compiled_graph_for_general_shape(*args)

        entry = self.concrete_size_entries[runtime_shape]

        if entry.runnable is None:
            entry.runnable = self.compiled_graph_for_general_shape

        if entry.cudagraph is None:
            if entry.num_finished_warmup < 1:  # noqa
                entry.num_finished_warmup += 1
                return entry.runnable(*args)

            if self.compile_config.get_enable_debug_mode():
                input_addresses = [
                    x.data_ptr() for x in args if isinstance(x, torch.Tensor)
                ]
                entry.input_addresses = input_addresses
            npugraph = torch.npu.NPUGraph()

            with ExitStack() as stack:
                if not self.is_first_graph:
                    # during every model forward, we will capture
                    # many pieces of cudagraphs (roughly one per layer).
                    # running gc again and again across layers will
                    # make the cudagraph capture very slow.
                    # therefore, we only run gc for the first graph,
                    # and disable gc for the rest of the graphs.
                    stack.enter_context(patch("gc.collect", lambda: None))
                    stack.enter_context(patch("torch.npu.empty_cache", lambda: None))

                # mind-exploding: carefully manage the reference and memory.
                with torch.npu.graph(npugraph, pool=self.graph_pool):
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
            entry.cudagraph = npugraph

            compilation_counter.num_cudagraph_captured += 1

            # important: we need to return the output, rather than
            # the weak ref of the output, so that pytorch can correctly
            # manage the memory during cuda graph capture
            return output

        if self.compile_config.get_enable_debug_mode():
            # check if the input addresses are the same
```
**EN:** This callable implements `NPUPiecewiseBackend.__call__`. It takes `*args` and mainly handles invocation-time behavior. This chunk is part 1 of 2 for the same logical block. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `NPUPiecewiseBackend.__call__`。它接收 `*args`，主要用于处理调用时的行为。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 101-109: Method NPUPiecewiseBackend.__call__ (part 2/2)
```python
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
**EN:** This callable implements `NPUPiecewiseBackend.__call__`. It takes `*args` and mainly handles invocation-time behavior. This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `NPUPiecewiseBackend.__call__`。它接收 `*args`，主要用于处理调用时的行为。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

## Key Concepts / 关键概念
- `NPUPiecewiseBackend`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `typing`, `unittest.mock`
- **Third-party / 第三方**: `torch`, `torch.fx`
- **Internal modules / 内部模块**: `sglang.srt.compilation.compilation_config`, `sglang.srt.compilation.compilation_counter`, `sglang.srt.compilation.cuda_piecewise_backend`
