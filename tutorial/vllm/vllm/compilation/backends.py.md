# backends.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/backends.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Compilation backend selection, graph splitting, and backend/CUDA-graph wrapping helpers. / 编译后端选择、图切分以及后端/CUDA 图封装辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-54)
```python
import ast
import dataclasses
import hashlib
import json
import operator
import os
import pprint
import time
from collections import defaultdict
from collections.abc import Callable, Generator, Sequence
from contextlib import contextmanager
from copy import deepcopy
from functools import partial
from typing import Any

import torch
import torch.fx as fx
from torch._dynamo.utils import dynamo_timed
from torch._logging._internal import trace_structured
from torch.fx._lazy_graph_module import _use_lazy_graph_module

import vllm.envs as envs
from vllm.compilation.codegen import (
    compile_execution_fn,
    generate_execution_code,
)
from vllm.config import CompilationConfig, CUDAGraphMode, VllmConfig
from vllm.config.compilation import DynamicShapesType
...
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in the compiler backend integration layer.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在编译后端集成层中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 56-777)
```python
logger = init_logger(__name__)
compilation_start_time = 0.0
model_tag: str = "backbone"
model_is_encoder: bool = False
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout the compiler backend integration layer.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在编译后端集成层中复用。

### Function `make_copy_and_call` (lines 59-93)
```python
def make_copy_and_call(
    sym_tensor_indices: list[int],
    input_buffers: list[torch.Tensor | None],
    callable_fn: Callable[..., Any],
) -> Callable[..., Any]:
    """Create a wrapper that copies inputs to static buffers before calling.

    This is used for cudagraph input copying where we need to copy dynamic
    tensors to static buffers before invoking the compiled graph.

    Args:
        sym_tensor_indices: Indices of tensors with symbolic shapes
        input_buffers: List of static buffers (can contain None for lazy init)
        callable_fn: The compiled function to call

    Returns:
...
```
**EN:** Function `make_copy_and_call` provides a focused helper used by the surrounding module. It fits into the compiler backend integration layer. Create a wrapper that copies inputs to static buffers before calling.
**CN:** 函数 `make_copy_and_call` 提供供周边模块使用的聚焦辅助逻辑，并嵌入编译后端集成层。 文档字符串说明：Create a wrapper that copies inputs to static buffers before calling.

### Function `make_compiler` (lines 96-121)
```python
def make_compiler(compilation_config: CompilationConfig) -> CompilerInterface:
    assert not envs.VLLM_USE_MEGA_AOT_ARTIFACT or envs.VLLM_USE_STANDALONE_COMPILE, (
        "VLLM_USE_MEGA_AOT_ARTIFACT=1 requires VLLM_USE_STANDALONE_COMPILE=1"
    )

    if compilation_config.backend == "inductor":
        # Use standalone compile only if requested, version is new enough,
        # and the symbol actually exists in this PyTorch build.
        if envs.VLLM_USE_STANDALONE_COMPILE and hasattr(
            torch._inductor, "standalone_compile"
        ):
            logger.debug("Using InductorStandaloneAdaptor")
            return InductorStandaloneAdaptor(
                compilation_config.compile_cache_save_format
            )
        else:
...
```
**EN:** Function `make_compiler` provides a focused helper used by the surrounding module. It fits into the compiler backend integration layer.
**CN:** 函数 `make_compiler` 提供供周边模块使用的聚焦辅助逻辑，并嵌入编译后端集成层。

### Class `CompilerManager` (lines 124-399)
```python
class CompilerManager:
    """
    A manager to manage the compilation process, including
    caching the compiled graph, loading the compiled graph,
    and compiling the graph.

    The cache is a dict mapping
    `(runtime_shape, graph_index, backend_name)`
    to `any_data` returned from the compiler.

    When serializing the cache, we save it to a Python file
    for readability. We don't use json here because json doesn't
    support int as key.
    """

    def __init__(self, compilation_config: CompilationConfig) -> None:
        self.cache: dict[tuple[Range, int, str], Any] = dict()
        self.is_cache_updated = False
        self.compilation_config = compilation_config
        self.compiler = make_compiler(compilation_config)
...
```
**EN:** Class `CompilerManager` coordinates multiple subcomponents and centralizes policy decisions. It operates inside the compiler backend integration layer. A manager to manage the compilation process, including caching the compiled graph, loading the compiled graph, and compiling the graph. Key methods include __init__, compute_hash, compile_context, initialize_cache, save_to_file.
**CN:** 类 `CompilerManager` 协调多个子组件并集中管理策略决策，并服务于编译后端集成层。 文档字符串强调：A manager to manage the compilation process, including caching the compiled graph, loading the compiled graph, and compiling the graph. 关键方法包括 __init__, compute_hash, compile_context, initialize_cache, save_to_file。

### Class `StopCompiling` (lines 402-403)
```python
class StopCompiling(BaseException):
    pass
```
**EN:** Class `StopCompiling` holds a main piece of file-specific logic. It operates inside the compiler backend integration layer.
**CN:** 类 `StopCompiling` 承载了该文件中的一块核心逻辑，并服务于编译后端集成层。

### Class `SplitItem` (lines 407-411)
```python
class SplitItem:
    submod_name: str
    graph_id: int
    is_splitting_graph: bool
    graph: fx.GraphModule
```
**EN:** Class `SplitItem` holds a main piece of file-specific logic. It operates inside the compiler backend integration layer.
**CN:** 类 `SplitItem` 承载了该文件中的一块核心逻辑，并服务于编译后端集成层。

### Function `_is_empty_allocation_node` (lines 414-434)
```python
def _is_empty_allocation_node(node: fx.Node) -> bool:
    if node.op == "call_method":
        return node.target == "new_empty"

    if node.op != "call_function":
        return False

    target = node.target
    if target in (torch.empty, torch.empty_like, torch.empty_strided):
        return True

    if isinstance(target, torch._ops.OpOverloadPacket):
        packet_name = target._qualified_op_name
    elif isinstance(target, torch._ops.OpOverload):
        packet_name = target.name()
    else:
...
```
**EN:** Function `_is_empty_allocation_node` provides a focused helper used by the surrounding module. It fits into the compiler backend integration layer.
**CN:** 函数 `_is_empty_allocation_node` 提供供周边模块使用的聚焦辅助逻辑，并嵌入编译后端集成层。

### Function `_merge_empty_only_subgraphs` (lines 437-476)
```python
def _merge_empty_only_subgraphs(
    node_to_subgraph_id: dict[fx.Node, int],
    split_op_graphs: list[int],
) -> None:
    """
    Merge a partition that only contains an empty allocation op into the
    previous partition. This avoids generating standalone empty submodules,
    which can lead to empty cudagraph captures.
    """

    nodes_by_subgraph_id: dict[int, list[fx.Node]] = defaultdict(list)
    for node, subgraph_id in node_to_subgraph_id.items():
        nodes_by_subgraph_id[subgraph_id].append(node)

    splitting_subgraphs = set(split_op_graphs)
    prev_non_splitting_subgraph_id: int | None = None
...
```
**EN:** Function `_merge_empty_only_subgraphs` provides a focused helper used by the surrounding module. It fits into the compiler backend integration layer. Merge a partition that only contains an empty allocation op into the previous partition.
**CN:** 函数 `_merge_empty_only_subgraphs` 提供供周边模块使用的聚焦辅助逻辑，并嵌入编译后端集成层。 文档字符串说明：Merge a partition that only contains an empty allocation op into the previous partition.

### Function `_decompose_size_nodes` (lines 479-545)
```python
def _decompose_size_nodes(graph: fx.GraphModule) -> None:
    """Decompose x.size() into per-dim sym_size.int calls.

    torch.Size objects cannot cross split boundaries because aot_autograd
    cannot handle them as submodule outputs. This replaces each size() call
    with individual sym_size.int(x, dim) nodes:
      - Dynamic dims (SymInt) → new sym_size.int node
      - Static dims (plain int) → inlined as literal constant
    """
    # Dynamo captures x.size()/x.shape as call_method target="size".
    size_nodes = list(graph.graph.find_nodes(op="call_method", target="size"))

    for node in size_nodes:
        tensor_node = node.args[0]
        ev = tensor_node.meta.get("example_value")
        assert ev is not None, (
...
```
**EN:** Function `_decompose_size_nodes` provides a focused helper used by the surrounding module. It fits into the compiler backend integration layer. Decompose x.size() into per-dim sym_size.int calls.
**CN:** 函数 `_decompose_size_nodes` 提供供周边模块使用的聚焦辅助逻辑，并嵌入编译后端集成层。 文档字符串说明：Decompose x.size() into per-dim sym_size.int calls.

### Function `split_graph` (lines 548-622)
```python
def split_graph(
    graph: fx.GraphModule, splitting_ops: list[str]
) -> tuple[fx.GraphModule, list[SplitItem]]:
    _decompose_size_nodes(graph)

    # split graph by ops
    subgraph_id = 0
    node_to_subgraph_id: dict[fx.Node, int] = {}
    split_op_graphs: list[int] = []
    for node in graph.graph.nodes:
        if node.op in ("output", "placeholder"):
            continue

        # Check if this is a getitem operation on a node from an earlier subgraph.
        # If so, assign it to the same subgraph as its input to avoid passing entire
        # tuple as input to submodules, which is against standalone_compile and
...
```
**EN:** Function `split_graph` rewrites graph structure to a form that later passes or backends handle more effectively. It fits into the compiler backend integration layer.
**CN:** 函数 `split_graph` 将图结构改写为后续 pass 或后端更易处理的形式，并嵌入编译后端集成层。

### Function `wrap_with_cudagraph_if_needed` (lines 628-679)
```python
def wrap_with_cudagraph_if_needed(
    piecewise_backend: Any,
    vllm_config: VllmConfig,
    compilation_config: CompilationConfig,
    is_first_graph: bool,
    is_last_graph: bool,
) -> Any:
    """
    Wrap a piecewise backend with CUDA graph wrapper if needed.
    This function is shared between VllmBackend and
    construct_serializable_fn_from_inductor_cache.

    Args:
        piecewise_backend: The backend to wrap
        vllm_config: The vLLM configuration
        compilation_config: The compilation configuration
...
```
**EN:** Function `wrap_with_cudagraph_if_needed` wraps a callable or graph with additional runtime-management behavior. It fits into the compiler backend integration layer. Wrap a piecewise backend with CUDA graph wrapper if needed.
**CN:** 函数 `wrap_with_cudagraph_if_needed` 为可调用对象或图增加额外的运行时管理封装，并嵌入编译后端集成层。 文档字符串说明：Wrap a piecewise backend with CUDA graph wrapper if needed.

### Class `PiecewiseCompileInterpreter` (lines 682-771)
```python
class PiecewiseCompileInterpreter(torch.fx.Interpreter):  # type: ignore[misc]
    """Code adapted from `torch.fx.passes.shape_prop.ShapeProp`.
    It runs the given split graph interpreter, and for each submodule in
    `compile_submod_names`, creates a PiecewiseBackend and compiles all
    ranges up front.

    NOTE: the order in `compile_submod_names` matters, because
    it will be used to determine the order of the compiled piecewise
    graphs. The first graph will handle logging, and the last graph
    has some special cudagraph output handling.

    Note: This class shares similar logic with
    reconstruct_serializable_fn_from_mega_artifact in caching.py.
    Both create PiecewiseBackend instances and wrap them with cudagraph.
    The key difference is:
    - reconstruct_serializable_fn_from_mega_artifact: PiecewiseBackend receives
      pre-compiled runnables (compiled_runnables is set, graph is None)
    - this class: PiecewiseBackend receives the FX graph to compile
      (graph is set, compiled_runnables is None)

...
```
**EN:** Class `PiecewiseCompileInterpreter` holds a main piece of file-specific logic. It operates inside the compiler backend integration layer. Code adapted from `torch.fx.passes.shape_prop.ShapeProp`. Key methods include __init__, run, call_module.
**CN:** 类 `PiecewiseCompileInterpreter` 承载了该文件中的一块核心逻辑，并服务于编译后端集成层。 文档字符串强调：Code adapted from `torch.fx.passes.shape_prop.ShapeProp`. 关键方法包括 __init__, run, call_module。

### Function `set_model_tag` (lines 781-797)
```python
def set_model_tag(tag: str, is_encoder: bool = False) -> Generator[None, None, None]:
    """Context manager to set the model tag."""
    global model_tag
    global model_is_encoder
    assert tag != model_tag, (
        f"Model tag {tag} is the same as the current tag {model_tag}."
    )
    old_tag = model_tag
    old_is_encoder = model_is_encoder

    model_tag = tag
    model_is_encoder = is_encoder
    try:
        yield
    finally:
        model_tag = old_tag
...
```
**EN:** Function `set_model_tag` mutates configuration or runtime flags that affect subsequent execution. It fits into the compiler backend integration layer. Context manager to set the model tag.
**CN:** 函数 `set_model_tag` 会修改影响后续执行的配置或运行时标志，并嵌入编译后端集成层。 文档字符串说明：Context manager to set the model tag.

### Class `VllmBackend` (lines 800-1331)
```python
class VllmBackend:
    """The compilation backend for `torch.compile` with vLLM.
    It is used for compilation mode of `CompilationMode.VLLM_COMPILE`,
    where we customize the compilation.

    The major work of this backend is to split the graph into
    piecewise graphs, and pass them to the piecewise backend.

    This backend also adds the PostGradPassManager to Inductor config,
    which handles the post-grad passes.
    """

    vllm_config: VllmConfig
    compilation_config: CompilationConfig
    _called: bool = False
    # the graph we compiled
    graph: fx.GraphModule
    # the stiching graph module for all the piecewise graphs
    split_gm: fx.GraphModule
    piecewise_graphs: list[SplitItem]
...
```
**EN:** Class `VllmBackend` holds a main piece of file-specific logic. It operates inside the compiler backend integration layer. The compilation backend for `torch.compile` with vLLM. Key methods include __init__, collect_standalone_compile_artifacts, configure_post_pass, _log_compilation_config, __call__.
**CN:** 类 `VllmBackend` 承载了该文件中的一块核心逻辑，并服务于编译后端集成层。 文档字符串强调：The compilation backend for `torch.compile` with vLLM. 关键方法包括 __init__, collect_standalone_compile_artifacts, configure_post_pass, _log_compilation_config, __call__。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。
- **Backend abstraction / 后端抽象**
  - **EN:** The file separates vLLM-facing policy from the concrete compiler/runtime used underneath.
  - **CN:** 该文件将 vLLM 侧策略与底层具体编译器/运行时实现解耦。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `import vllm.envs as envs`, `from vllm.compilation.codegen import compile_execution_fn, generate_execution_code`, `from vllm.config import CompilationConfig, CUDAGraphMode, VllmConfig`, `from vllm.config.compilation import DynamicShapesType`, `from vllm.config.utils import Range, hash_factors`, `from vllm.logger import init_logger`, `from vllm.logging_utils import lazy`, `from vllm.platforms import current_platform`, ...
- **PyTorch / Torch 栈**: `import torch`, `import torch.fx as fx`, `from torch._dynamo.utils import dynamo_timed`, `from torch._logging._internal import trace_structured`, `from torch.fx._lazy_graph_module import _use_lazy_graph_module`
- **Stdlib / 标准库**: `import ast`, `import dataclasses`, `import hashlib`, `import json`, `import operator`, `import os`, `import pprint`, `import time`, ...
