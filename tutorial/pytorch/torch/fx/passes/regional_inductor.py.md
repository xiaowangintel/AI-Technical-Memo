# regional_inductor.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/regional_inductor.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import contextlib
import functools
import logging
from collections.abc import Callable, Iterator, Mapping
from typing import Any, ParamSpec, TypeVar


_P = ParamSpec("_P")
_R = TypeVar("_R")

import torch
from torch.fx._compatibility import compatibility


logger = logging.getLogger(__name__)

__all__ = ["regional_inductor"]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 19-31
```python

# standalone_inductor returns a callable class object - this does not sit well
# with Fx graph node op call_function which expects a function. So this is just
# a wrapper function to make Fx graph codegen happy.
def _dummy_wrapper(fn: Callable[_P, _R]) -> Callable[_P, _R]:
    @functools.wraps(fn)
    def inner(*args: _P.args, **kwargs: _P.kwargs) -> _R:
        return fn(*args, **kwargs)

    return inner


@contextlib.contextmanager
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 32-44
```python
def _disable_remat_for_regional_subcompile() -> Iterator[None]:
    # In torch.compile, regional_inductor subcompiles run after the enclosing
    # non-strict full graph has already been partitioned, so any graph-SAC
    # remat pass has already run before we reach this nested compile.
    # Rerunning remat here can see stage-2-reordered backward nodes that
    # violate remat's contiguous-backward-region assumption.
    with torch._functorch.config.patch(remat_using_tags_for_fwd_loss_bwd_graph=False):
        yield


def _compile_submod(gm: torch.fx.GraphModule, prefix: str) -> torch.fx.GraphModule:
    from torch._inductor.standalone_compile import AOTCompiledArtifact
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会使用带作用域的辅助对象或上下文管理器。

### Lines 45-57
```python
    for node in gm.graph.nodes:
        if node.op == "call_module" and node.target.startswith(prefix):
            fake_inputs = []
            for inp_node in node.all_input_nodes:
                if hasattr(inp_node, "meta") and "val" in inp_node.meta:
                    fake_inputs.append(inp_node.meta["val"])
                else:
                    raise RuntimeError(
                        f"Partition is bad because non fake tensor value is seen {inp_node}"
                    )

            submod = getattr(gm, node.target)
```
- **EN**: This block continues `_compile_submod` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_compile_submod`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 58-73
```python
            # Get inductor configs from annotation
            # TODO we should change partition when there are multiple differently
            # annotated regions.
            inductor_options: dict[str, Any] = {}
            for sub_node in submod.graph.nodes:
                if hasattr(sub_node, "meta") and sub_node.meta.get("custom", None):
                    custom = sub_node.meta["custom"]
                    if isinstance(custom, dict) and "compile_with_inductor" in custom:
                        compile_value = custom["compile_with_inductor"]
                        if (
                            isinstance(compile_value, dict)
                            and "inductor_configs" in compile_value
                        ):
                            inductor_options = compile_value["inductor_configs"]
                            break
```
- **EN**: This block continues `_compile_submod` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_compile_submod`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 74-91
```python
            # Log the options being used
            logger.info(
                "Compiling submodule %s with inductor options: %s",
                node.target,
                inductor_options,
            )

            # Apply config patches before compilation
            import torch._inductor.config as inductor_config

            # Validate that all config keys exist
            for key in inductor_options:
                if not hasattr(inductor_config, key):
                    raise ValueError(
                        f"Invalid inductor config key '{key}' in regional_inductor annotation. "
                        f"Available config keys can be found in torch._inductor.config"
                    )
```
- **EN**: This block continues `_compile_submod` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_compile_submod`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 92-103
```python
            with (
                inductor_config.patch(inductor_options),
                _disable_remat_for_regional_subcompile(),
            ):
                compiled_fn = torch._inductor.standalone_compile(
                    submod,
                    fake_inputs,
                    dynamic_shapes="from_tracing_context",
                    aot=True,
                )
            if not isinstance(compiled_fn, AOTCompiledArtifact):
                raise AssertionError(
```
- **EN**: This block continues `_compile_submod` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `_compile_submod`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 104-120
```python
                    f"Expected AOTCompiledArtifact, got {type(compiled_fn)}"
                )
            # _dummy_wrapper is to make call_function happy
            compiled_submod = _dummy_wrapper(compiled_fn)
            with gm.graph.inserting_after(node):
                new_node = gm.graph.call_function(
                    compiled_submod, args=node.args, kwargs=node.kwargs
                )
                new_node.meta = node.meta
                node.replace_all_uses_with(new_node)
                gm.graph.erase_node(node)
                del gm._modules[node.target]

    gm.recompile()
    return gm
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；使用带作用域的辅助对象或上下文管理器。

### Lines 121-134
```python
def _needs_inductor_compile(node: torch.fx.Node) -> bool:
    return bool(
        node.op not in ("placeholder", "output")
        and hasattr(node, "meta")
        and node.meta.get("custom", None)
        and "compile_with_inductor" in node.meta["custom"]
    )


class _RegionScooper:
    """
    Scoops out the inductor marked regions. It does NOT compile them.
    """
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 135-146
```python
    @staticmethod
    def scoop_regions(gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
        from torch.fx.passes.infra.partitioner import CapabilityBasedPartitioner
        from torch.fx.passes.operator_support import create_op_support
        from torch.fx.passes.utils.fuser_utils import fuse_by_partitions

        # Group tagged nodes by region ID.  The region ID comes from the
        # optional "inductor_region" key inside the compile_with_inductor
        # annotation. When absent, all tagged nodes share a single default region
        _DEFAULT_REGION = object()
        regions: dict[object, set[torch.fx.Node]] = {}
        for node in gm.graph.nodes:
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 147-161
```python
            if _needs_inductor_compile(node):
                compile_value = node.meta["custom"]["compile_with_inductor"]
                if (
                    isinstance(compile_value, dict)
                    and "inductor_region" in compile_value
                ):
                    rid = compile_value["inductor_region"]
                else:
                    rid = _DEFAULT_REGION
                regions.setdefault(rid, set()).add(node)

        if not regions:
            logger.info("No inductor marked nodes found")
            return gm
```
- **EN**: This block continues `_RegionScooper.scoop_regions` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_RegionScooper.scoop_regions`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 162-173
```python
        # Run CapabilityBasedPartitioner per region to get cycle-safe partitions
        # without merging across region boundaries.
        def _is_in_region(
            region_nodes: set[torch.fx.Node],
        ) -> Callable[[Mapping[str, torch.nn.Module], torch.fx.Node], bool]:
            def is_node_supported(
                _submodules: Mapping[str, torch.nn.Module], node: torch.fx.Node
            ) -> bool:
                return node in region_nodes

            return is_node_supported
```
- **EN**: Defines the `_RegionScooper.scoop_regions` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`_RegionScooper.scoop_regions` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 174-189
```python
        all_partitions: list[dict[torch.fx.Node, int | None]] = []
        for region_nodes in regions.values():
            support = create_op_support(_is_in_region(region_nodes))
            partitioner = CapabilityBasedPartitioner(
                gm, support, allows_single_node_partition=True
            )
            for partition in partitioner.propose_partitions():
                all_partitions.append(partition.nodes)

        return fuse_by_partitions(
            gm,
            all_partitions,
            prefix="__marked_inductor_submod",
            always_return_tuple=True,
        )
```
- **EN**: This block continues `_RegionScooper` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `_RegionScooper`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 190-201
```python
    @staticmethod
    def recursively_scoop_regions(
        gm: torch.fx.GraphModule, _processed: set[int] | None = None
    ) -> torch.fx.GraphModule:
        if _processed is None:
            _processed = set()
        for node in gm.graph.find_nodes(op="get_attr"):
            if _needs_inductor_compile(node):
                # If the get_attr itself is marked for compile, the outer graph will
                # take care of it. If we dont do that, we end up with nested
                # regional inductor compiles that do not work well.
                continue
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 202-217
```python
            submod = getattr(gm, node.target)
            # Track by id: multiple get_attr nodes may reference the same GraphModule
            if (
                isinstance(submod, torch.fx.GraphModule)
                and id(submod) not in _processed
            ):
                _processed.add(id(submod))
                _RegionScooper.recursively_scoop_regions(submod, _processed)

        return _RegionScooper.scoop_regions(gm)

    def __call__(self, gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
        with torch.fx.traceback.preserve_node_meta(enable=False):
            return _RegionScooper.recursively_scoop_regions(gm)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 218-231
```python
class _RegionCompiler:
    """
    Compiles the scooped out regions.
    """

    @staticmethod
    def compile_region(gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
        from torch.fx.graph import _BoxedCodeGen

        gm = _compile_submod(gm, "__marked_inductor_submod")
        gm.graph.set_codegen(_BoxedCodeGen())
        gm.recompile()
        return gm
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 232-247
```python
    @staticmethod
    def recursively_compile_regions(gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
        # Find if the graph module has a scooped out region
        found_region = False
        for node in gm.graph.find_nodes(op="call_module"):
            submod = getattr(gm, node.target)
            if isinstance(submod, torch.fx.GraphModule):
                if node.target.startswith("__marked_inductor_submod"):
                    found_region = True

        # Recurse through the subgraphs
        for node in gm.graph.find_nodes(op="get_attr"):
            submod = getattr(gm, node.target)
            if isinstance(submod, torch.fx.GraphModule):
                _RegionCompiler.recursively_compile_regions(submod)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 248-265
```python
        if found_region:
            return _RegionCompiler.compile_region(gm)
        return gm

    def __call__(self, gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
        with torch.fx.traceback.preserve_node_meta(enable=False):
            return _RegionCompiler.recursively_compile_regions(gm)


def _create_inductor_marked_regions(gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
    with torch.fx.traceback.preserve_node_meta(enable=False):
        return _RegionScooper()(gm)


def _compile_inductor_marked_regions(gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
    with torch.fx.traceback.preserve_node_meta(enable=False):
        return _RegionCompiler()(gm)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 266-277
```python

@compatibility(is_backward_compatible=False)
def regional_inductor(
    gm: torch.fx.GraphModule, *example_args: object
) -> torch.fx.GraphModule:
    """
    Scoops out inductor marked regions and compiles them with inductor.

    Inductor options should be provided via the annotation API::

        with fx_traceback.annotate(
            {
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 278-291
```python
                "compile_with_inductor": {
                    "inductor_configs": {
                        "max_autotune": True,
                        "triton.cudagraphs": False,
                    }
                }
            }
        ):
            ...
    """

    # fuser utils create new nodes using create_proxy which retains the seq_nr
    # metadata and cause issues
```
- **EN**: This block continues `regional_inductor` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `regional_inductor`，用于构建、遍历或改写图结构及其元数据。

### Lines 292-299
```python
    with torch.fx.traceback.preserve_node_meta(enable=False):
        gm = _create_inductor_marked_regions(gm)
        gm = _compile_inductor_marked_regions(gm)
        if torch._functorch.config.force_autograd_cache:
            from torch._inductor.output_code import RegionalOutputCode

            return RegionalOutputCode(gm)  # type: ignore[return-value]
        return gm
```
- **EN**: This block continues `regional_inductor` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `regional_inductor`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx._compatibility`, `torch._inductor.standalone_compile`, `torch.fx.passes.infra.partitioner`, `torch.fx.passes.operator_support`, `torch.fx.passes.utils.fuser_utils`, `torch.fx.graph`, `torch._inductor.config`, `torch._inductor.output_code`
- **Standard library / 标准库**: `contextlib`, `functools`, `logging`, `collections.abc`, `typing`
- **Primary symbols / 核心符号**: `_P`, `_R`, `__all__`, `_dummy_wrapper`, `_disable_remat_for_regional_subcompile`, `_compile_submod`, `_needs_inductor_compile`, `_RegionScooper`, `_RegionCompiler`, `_create_inductor_marked_regions`, `_compile_inductor_marked_regions`, `regional_inductor`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
