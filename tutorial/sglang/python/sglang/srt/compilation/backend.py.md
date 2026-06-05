# backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/compilation/backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the compilation and graph lowering part of the SRT runtime and implements logic centered on `backend`. It exposes primary entry points such as `make_compiler`, `make_backend`, `CompilerManager`. / 该模块属于 SRT 运行时的编译与图优化部分，主要实现围绕 `backend` 的逻辑。 它对外提供的主要入口包括 `make_compiler`, `make_backend`, `CompilerManager`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.10.0/vllm/compilation/backend.py


import ast
import dataclasses
import logging
import os
import pprint
import time
from collections.abc import Sequence
from contextlib import contextmanager
from typing import Any, Callable, Optional

import torch
import torch.fx as fx
from torch._dispatch.python import enable_python_dispatcher

from sglang.srt.compilation.compilation_config import CompilationConfig
from sglang.srt.compilation.compilation_counter import compilation_counter
from sglang.srt.compilation.compiler_interface import EagerAdapter, InductorAdaptor
from sglang.srt.compilation.cuda_piecewise_backend import CUDAPiecewiseBackend
from sglang.srt.compilation.npu_piecewise_backend import NPUPiecewiseBackend
from sglang.srt.compilation.pass_manager import PostGradPassManager
from sglang.srt.environ import envs
from sglang.srt.platforms import current_platform
from sglang.srt.utils.common import is_npu

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；读取环境变量驱动的配置。

### Lines 33-39: Function make_compiler
```python
def make_compiler(config: CompilationConfig):
    if config.compiler == "eager":
        return EagerAdapter()
    elif config.compiler == "inductor":
        return InductorAdaptor()
    else:
        raise ValueError(f"Unknown compiler: {config.compiler}")
```
**EN:** This callable implements `make_compiler`. It takes `config` and mainly implements make compiler. In this range it performs defensive checks on invalid state; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `make_compiler`。它接收 `config`，主要用于实现 make compiler 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；处理与编译相关的行为。

### Lines 42-70: Function make_backend
```python
def make_backend(
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

    if current_platform.is_out_of_tree():
        backend_cls = current_platform.get_piecewise_backend_cls()
    elif is_npu():
        backend_cls = NPUPiecewiseBackend
    else:
        backend_cls = CUDAPiecewiseBackend
    return backend_cls(
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
**EN:** This callable implements `make_backend`. It takes `graph`, `compile_config`, `inductor_config`, `graph_pool` and mainly implements make backend. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `make_backend`。它接收 `graph`, `compile_config`, `inductor_config`, `graph_pool`，主要用于实现 make backend 相关逻辑。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 73-73: Class CompilerManager
```python
class CompilerManager:
```
**EN:** This range introduces `CompilerManager` and defines the structure or metadata that its methods rely on. In this range it prepares compilation-related behavior.
**CN:** 这一段引入 `CompilerManager`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理与编译相关的行为。

### Lines 74-80: Method CompilerManager.__init__
```python
    def __init__(
        self,
        config: CompilationConfig,
    ):
        self.cache = dict()
        self.is_cache_updated = False
        self.compiler = make_compiler(config)
```
**EN:** This callable implements `CompilerManager.__init__`. It takes `config` and mainly initializes instance state and defaults. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `CompilerManager.__init__`。它接收 `config`，主要用于初始化实例状态与默认值。 在这一范围内，它会处理与编译相关的行为。

### Lines 82-83: Method CompilerManager.compute_hash
```python
    def compute_hash(self):
        return self.compiler.compute_hash()
```
**EN:** This callable implements `CompilerManager.compute_hash` and mainly implements compute hash. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `CompilerManager.compute_hash`，主要用于实现 compute hash 相关逻辑。 在这一范围内，它会处理与编译相关的行为。

### Lines 85-98: Method CompilerManager.initialize_cache
```python
    def initialize_cache(
        self, cache_dir: str, disable_cache: bool = False, prefix: str = ""
    ):
        self.disable_cache = disable_cache
        self.cache_dir = cache_dir
        self.cache_file_path = os.path.join(cache_dir, "sglang_compile_cache.py")

        if not disable_cache and os.path.exists(self.cache_file_path):
            with open(self.cache_file_path) as f:
                self.cache = ast.literal_eval(f.read())

        self.compiler.initialize_cache(
            cache_dir=cache_dir, disable_cache=disable_cache, prefix=prefix
        )
```
**EN:** This callable implements `CompilerManager.initialize_cache`. It takes `cache_dir`, `disable_cache`, `prefix` and mainly implements initialize cache. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `CompilerManager.initialize_cache`。它接收 `cache_dir`, `disable_cache`, `prefix`，主要用于实现 initialize cache 相关逻辑。 在这一范围内，它会处理与编译相关的行为。

### Lines 100-106: Method CompilerManager.save_to_file
```python
    def save_to_file(self):
        if self.disable_cache or not self.is_cache_updated:
            return
        printer = pprint.PrettyPrinter(indent=4)
        data = printer.pformat(self.cache)
        with open(self.cache_file_path, "w") as f:
            f.write(data)
```
**EN:** This callable implements `CompilerManager.save_to_file` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `CompilerManager.save_to_file`，主要用于将数据转换为另一种表示。

### Lines 108-135: Method CompilerManager.load
```python
    def load(
        self,
        graph: fx.GraphModule,
        example_inputs: list[Any],
        graph_index: int,
        runtime_shape: Optional[int] = None,
    ) -> Optional[Callable]:
        handle = self.cache[(runtime_shape, graph_index, self.compiler.name)]
        compiled_graph = self.compiler.load(
            handle, graph, example_inputs, graph_index, runtime_shape
        )
        if runtime_shape is None:
            logger.debug(
                "Directly load the %s-th graph for dynamic shape from %s via "
                "handle %s",
                graph_index,
                self.compiler.name,
                handle,
            )
        else:
            logger.debug(
                "Directly load the %s-th graph for shape %s from %s via " "handle %s",
                graph_index,
                str(runtime_shape),
                self.compiler.name,
                handle,
            )
        return compiled_graph
```
**EN:** This callable implements `CompilerManager.load`. It takes `graph`, `example_inputs`, `graph_index`, `runtime_shape` and mainly loads external data or weights. In this range it sets up imports and shared symbols; emits logs for diagnostics; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `CompilerManager.load`。它接收 `graph`, `example_inputs`, `graph_index`, `runtime_shape`，主要用于加载外部数据或权重。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理与编译相关的行为。

### Lines 137-196: Method CompilerManager.compile (part 1/2)
```python
    def compile(
        self,
        graph: fx.GraphModule,
        example_inputs,
        inductor_config: dict[str, Any],
        graph_index: int = 0,
        num_graphs: int = 1,
        runtime_shape: Optional[int] = None,
    ) -> Any:
        if graph_index == 0:
            # before compiling the first graph, record the start time
            global compilation_start_time
            compilation_start_time = time.time()

        compilation_counter.num_backend_compilations += 1

        compiled_graph = None

        # TODO(Yuwei): support cache loading

        # no compiler cached the graph, or the cache is disabled,
        # we need to compile it
        if isinstance(self.compiler, InductorAdaptor):
            maybe_key = None
        else:
            maybe_key = f"artifact_shape_{runtime_shape}_subgraph_{graph_index}"
        compiled_graph, handle = self.compiler.compile(
            graph, example_inputs, inductor_config, runtime_shape, maybe_key
        )

        assert compiled_graph is not None, "Failed to compile the graph"

        # store the artifact in the cache
        if handle is not None:
            self.cache[(runtime_shape, graph_index, self.compiler.name)] = handle
            compilation_counter.num_cache_entries_updated += 1
            self.is_cache_updated = True
            if graph_index == 0:
                # adds some info logging for the first graph
                if runtime_shape is None:
                    logger.info("Cache the graph for dynamic shape for later use")
                else:
                    logger.info(
                        "Cache the graph of shape %s for later use", str(runtime_shape)
                    )
            if runtime_shape is None:
                logger.debug(
                    "Store the %s-th graph for dynamic shape from %s via " "handle %s",
                    graph_index,
                    self.compiler.name,
                    handle,
                )
            else:
                logger.debug(
                    "Store the %s-th graph for shape %s from %s via handle %s",
                    graph_index,
                    str(runtime_shape),
                    self.compiler.name,
                    handle,
                )
```
**EN:** This callable implements `CompilerManager.compile`. It takes `graph`, `example_inputs`, `inductor_config`, `graph_index` and mainly implements compile. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `CompilerManager.compile`。它接收 `graph`, `example_inputs`, `inductor_config`, `graph_index`，主要用于实现 compile 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 197-211: Method CompilerManager.compile (part 2/2)
```python

        # after compiling the last graph, record the end time
        if graph_index == num_graphs - 1:
            now = time.time()
            elapsed = now - compilation_start_time
            if runtime_shape is None:
                logger.info("Compiling a graph for dynamic shape takes %.2f s", elapsed)
            else:
                logger.info(
                    "Compiling a graph for shape %s takes %.2f s",
                    runtime_shape,
                    elapsed,
                )

        return compiled_graph
```
**EN:** This callable implements `CompilerManager.compile`. It takes `graph`, `example_inputs`, `inductor_config`, `graph_index` and mainly implements compile. This chunk is part 2 of 2 for the same logical block. In this range it emits logs for diagnostics; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CompilerManager.compile`。它接收 `graph`, `example_inputs`, `inductor_config`, `graph_index`，主要用于实现 compile 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会输出日志以便诊断；处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 214-219: Class SplitItem
```python
@dataclasses.dataclass
class SplitItem:
    submod_name: str
    graph_id: int
    is_splitting_graph: bool
    graph: fx.GraphModule
```
**EN:** This range introduces `SplitItem` and defines the structure or metadata that its methods rely on. In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `SplitItem`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 222-265: Function split_graph
```python
def split_graph(
    graph: fx.GraphModule, ops: list[str]
) -> tuple[fx.GraphModule, list[SplitItem]]:
    # split graph by ops
    subgraph_id = 0
    node_to_subgraph_id = {}
    split_op_graphs = []
    for node in graph.graph.nodes:
        if node.op in ("output", "placeholder"):
            continue
        if node.op == "call_function" and str(node.target) in ops:
            subgraph_id += 1
            node_to_subgraph_id[node] = subgraph_id
            split_op_graphs.append(subgraph_id)
            subgraph_id += 1
        else:
            node_to_subgraph_id[node] = subgraph_id

    # `keep_original_order` is important!
    # otherwise pytorch might reorder the nodes and
    # the semantics of the graph will change when we
    # have mutations in the graph
    split_gm = torch.fx.passes.split_module.split_module(
        graph, None, lambda node: node_to_subgraph_id[node], keep_original_order=True
    )

    outputs = []

    names = [name for (name, module) in split_gm.named_modules()]

    for name in names:
        if "." in name or name == "":
            # recursive child module or the root module
            continue

        module = getattr(split_gm, name)

        graph_id = int(name.replace("submod_", ""))
        outputs.append(SplitItem(name, graph_id, (graph_id in split_op_graphs), module))

    # sort by intetger graph_id, rather than string name
    outputs.sort(key=lambda x: x.graph_id)

    return split_gm, outputs
```
**EN:** This callable implements `split_graph`. It takes `graph`, `ops` and mainly implements split graph. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `split_graph`。它接收 `graph`, `ops`，主要用于实现 split graph 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 266-273: Module-level constants and helpers
```python


# we share the global graph pool among all the backends
global_graph_pool = None

compilation_start_time = 0.0


```
**EN:** This range organizes module-level state and shared setup. In this range it manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 274-274: Class PiecewiseCompileInterpreter
```python
class PiecewiseCompileInterpreter(torch.fx.Interpreter):
```
**EN:** This range introduces `PiecewiseCompileInterpreter` and defines the structure or metadata that its methods rely on. In this range it prepares compilation-related behavior.
**CN:** 这一段引入 `PiecewiseCompileInterpreter`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理与编译相关的行为。

### Lines 275-294: Method PiecewiseCompileInterpreter.__init__
```python
    def __init__(
        self,
        module: torch.fx.GraphModule,
        compile_submod_names: list[str],
        inductor_config: dict[str, Any],
        graph_pool,
        compile_config: CompilationConfig,
        sglang_backend: "SGLangBackend",
    ):
        super().__init__(module)
        from torch._guards import detect_fake_mode

        self.fake_mode = detect_fake_mode()
        self.compile_submod_names = compile_submod_names
        self.graph_pool = graph_pool
        self.sglang_backend = sglang_backend
        # When True, it annoyingly dumps the torch.fx.Graph on errors.
        self.extra_traceback = False
        self.inductor_config = inductor_config
        self.compile_config = compile_config
```
**EN:** This callable implements `PiecewiseCompileInterpreter.__init__`. It takes `module`, `compile_submod_names`, `inductor_config`, `graph_pool` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; records metrics or tracing signals; prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `PiecewiseCompileInterpreter.__init__`。它接收 `module`, `compile_submod_names`, `inductor_config`, `graph_pool`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号；处理与编译相关的行为。

### Lines 296-302: Method PiecewiseCompileInterpreter.run
```python
    def run(self, *args):
        fake_args = [
            self.fake_mode.from_tensor(t) if isinstance(t, torch.Tensor) else t
            for t in args
        ]
        with self.fake_mode, enable_python_dispatcher():
            return super().run(*fake_args)
```
**EN:** This callable implements `PiecewiseCompileInterpreter.run`. It takes `*args` and mainly executes the main workflow.
**CN:** 这一可调用对象实现了 `PiecewiseCompileInterpreter.run`。它接收 `*args`，主要用于执行主要流程。

### Lines 304-345: Method PiecewiseCompileInterpreter.call_module
```python
    def call_module(
        self,
        target: torch.fx.node.Target,
        args: tuple[torch.fx.node.Argument, ...],
        kwargs: dict[str, Any],
    ) -> Any:
        assert isinstance(target, str)
        output = super().call_module(target, args, kwargs)

        if target in self.compile_submod_names:
            index = self.compile_submod_names.index(target)
            submod = self.fetch_attr(target)
            sym_shape_indices = [
                i for i, x in enumerate(args) if isinstance(x, torch.SymInt)
            ]
            global compilation_start_time
            compiled_graph_for_dynamic_shape = (
                self.sglang_backend.compiler_manager.compile(
                    submod,
                    args,
                    self.inductor_config,
                    graph_index=index,
                    num_graphs=len(self.compile_submod_names),
                    runtime_shape=None,
                )
            )

            self.module.__dict__[target] = make_backend(
                submod,
                self.compile_config,
                self.inductor_config,
                self.graph_pool,
                index,
                len(self.compile_submod_names),
                sym_shape_indices,
                compiled_graph_for_dynamic_shape,
                self.sglang_backend,
            )

            compilation_counter.num_piecewise_capturable_graphs_seen += 1

        return output
```
**EN:** This callable implements `PiecewiseCompileInterpreter.call_module`. It takes `target`, `args`, `kwargs` and mainly implements call module. In this range it performs defensive checks on invalid state; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `PiecewiseCompileInterpreter.call_module`。它接收 `target`, `args`, `kwargs`，主要用于实现 call module 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 346-350: Module-level constants and helpers
```python


model_tag: str = "backbone"


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 351-363: Function set_model_tag
```python
@contextmanager
def set_model_tag(tag: str):
    """Context manager to set the model tag."""
    global model_tag
    assert (
        tag != model_tag
    ), f"Model tag {tag} is the same as the current tag {model_tag}."
    old_tag = model_tag
    model_tag = tag
    try:
        yield
    finally:
        model_tag = old_tag
```
**EN:** This callable implements `set_model_tag`. It takes `tag` and mainly applies configuration to mutable state. The docstring states: "Context manager to set the model tag." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `set_model_tag`。它接收 `tag`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 366-381: Class SGLangBackend
```python
class SGLangBackend:

    graph_pool: Any
    _called: bool = False
    # the graph we compiled
    graph: fx.GraphModule
    # the stiching graph module for all the piecewise graphs
    split_gm: fx.GraphModule
    piecewise_graphs: list[SplitItem]
    returned_callable: Callable
    # Inductor passes to run on the graph pre-defunctionalization
    post_grad_passes: Sequence[Callable]
    sym_tensor_indices: list[int]
    input_buffers: list[torch.Tensor]
    compiler_manager: CompilerManager

```
**EN:** This range introduces `SGLangBackend` and defines the structure or metadata that its methods rely on. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一段引入 `SGLangBackend`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 382-398: Method SGLangBackend.__init__
```python
    def __init__(
        self,
        config: CompilationConfig,
        graph_pool: Any,
    ):
        assert graph_pool is not None
        self.graph_pool = graph_pool

        self.post_grad_pass_manager = PostGradPassManager()
        self.sym_tensor_indices = []
        self.input_buffers = []

        self.compiler_manager = CompilerManager(config)
        self.inductor_config = {
            "enable_auto_functionalized_v2": False,
        }
        self.compile_config = config
```
**EN:** This callable implements `SGLangBackend.__init__`. It takes `config`, `graph_pool` and mainly initializes instance state and defaults. In this range it performs defensive checks on invalid state; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SGLangBackend.__init__`。它接收 `config`, `graph_pool`，主要用于初始化实例状态与默认值。 在这一范围内，它会对非法状态执行防御性检查；处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 400-402: Method SGLangBackend.configure_post_pass
```python
    def configure_post_pass(self):
        self.post_grad_pass_manager.configure()
        self.inductor_config["post_grad_custom_post_pass"] = self.post_grad_pass_manager
```
**EN:** This callable implements `SGLangBackend.configure_post_pass` and mainly implements configure post pass. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `SGLangBackend.configure_post_pass`，主要用于实现 configure post pass 相关逻辑。 在这一范围内，它会处理与编译相关的行为。

### Lines 404-463: Method SGLangBackend.__call__ (part 1/2)
```python
    def __call__(self, graph: fx.GraphModule, example_inputs) -> Callable:
        base_cache_dir = envs.SGLANG_CACHE_DIR.get()

        cache_hash = self.compiler_manager.compute_hash()
        cache_dir = os.path.join(
            base_cache_dir,
            "torch_compile_cache",
            cache_hash,
        )

        os.makedirs(cache_dir, exist_ok=True)
        rank = 0
        dp_rank = 0
        local_cache_dir = os.path.join(cache_dir, f"rank_{rank}_{dp_rank}", model_tag)
        os.makedirs(local_cache_dir, exist_ok=True)
        self.compiler_manager.initialize_cache(
            local_cache_dir, disable_cache=False, prefix=""
        )
        compilation_counter.num_graphs_seen += 1

        assert not self._called, "SGLangBackend can only be called once"

        self.graph = graph
        self.configure_post_pass()

        self.split_gm, self.piecewise_graphs = split_graph(
            graph,
            self.compile_config.split_ops,
        )
        from torch._dynamo.utils import lazy_format_graph_code

        # depyf will hook lazy_format_graph_code and dump the graph
        # for debugging, no need to print the graph here
        lazy_format_graph_code("before split", self.graph)
        lazy_format_graph_code("after split", self.split_gm)

        compilation_counter.num_piecewise_graphs_seen += len(self.piecewise_graphs)

        submod_names_to_compile = [
            item.submod_name
            for item in self.piecewise_graphs
            if not item.is_splitting_graph
        ]

        PiecewiseCompileInterpreter(
            self.split_gm,
            submod_names_to_compile,
            self.inductor_config,
            self.graph_pool,
            self.compile_config,
            self,
        ).run(*example_inputs)

        rank = torch.distributed.get_rank()

        if rank == 0:
            graph_path = os.path.join(
                local_cache_dir, f"computation_graph_{time.time()}.py"
            )
            if not os.path.exists(graph_path):
```
**EN:** This callable implements `SGLangBackend.__call__`. It takes `graph`, `example_inputs` and mainly handles invocation-time behavior. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `SGLangBackend.__call__`。它接收 `graph`, `example_inputs`，主要用于处理调用时的行为。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；协调分布式通信。

### Lines 464-475: Method SGLangBackend.__call__ (part 2/2)
```python
                # code adapted from https://github.com/thuml/depyf/blob/dab831108a752d1facc00acdd6d4243891845c37/depyf/explain/patched_lazy_format_graph_code.py#L30 # noqa
                # use `print_readable` because it can include submodules
                src = (
                    "from __future__ import annotations\nimport torch\n"
                    + self.split_gm.print_readable(print_output=False)
                )
                src = src.replace("<lambda>", "GraphModule")
                with open(graph_path, "w") as f:
                    f.write(src)

        self._called = True
        return self.split_gm
```
**EN:** This callable implements `SGLangBackend.__call__`. It takes `graph`, `example_inputs` and mainly handles invocation-time behavior. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SGLangBackend.__call__`。它接收 `graph`, `example_inputs`，主要用于处理调用时的行为。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；管理图捕获或回放逻辑。

## Key Concepts / 关键概念
- `make_compiler`: implements make compiler / 实现 make compiler 相关逻辑
- `make_backend`: implements make backend / 实现 make backend 相关逻辑
- `CompilerManager`: core class or state container / 核心类或状态容器
- `SplitItem`: core class or state container / 核心类或状态容器
- `split_graph`: implements split graph / 实现 split graph 相关逻辑
- `PiecewiseCompileInterpreter`: core class or state container / 核心类或状态容器
- `set_model_tag`: applies configuration to mutable state / 将配置写入可变状态
- `SGLangBackend`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ast`, `dataclasses`, `logging`, `os`, `pprint`, `time`, `collections.abc`, `contextlib`, `typing`
- **Third-party / 第三方**: `torch`, `torch.fx`, `torch._dispatch.python`, `torch._guards`, `torch._dynamo.utils`
- **Internal modules / 内部模块**: `sglang.srt.compilation.compilation_config`, `sglang.srt.compilation.compilation_counter`, `sglang.srt.compilation.compiler_interface`, `sglang.srt.compilation.cuda_piecewise_backend`, `sglang.srt.compilation.npu_piecewise_backend`, `sglang.srt.compilation.pass_manager`, `sglang.srt.environ`, `sglang.srt.platforms`, `sglang.srt.utils.common`
