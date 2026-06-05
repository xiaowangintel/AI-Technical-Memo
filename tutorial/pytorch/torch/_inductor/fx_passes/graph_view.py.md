# graph_view.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/graph_view.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `GraphView`. It exposes functions such as `_get_module_stack`, `_addindent`, `_clean_stack_name`, `_is_root`, `make_graph_view`, and `get_subgraph_by_path`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `GraphView` 等类。同时提供 `_get_module_stack`、`_addindent`、`_clean_stack_name`、`_is_root`、`make_graph_view`、`get_subgraph_by_path` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
from __future__ import annotations

import itertools
import re
from typing import Any, TYPE_CHECKING

import torch.fx as fx  # noqa: TC001
from torch.utils._ordered_set import OrderedSet


if TYPE_CHECKING:
    from collections.abc import Callable


````
- **EN**: Imports dependencies such as `__future__`, `itertools`, `re`, `typing`, `torch.fx`, `torch.utils._ordered_set`, and `...+1` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `__future__`、`itertools`、`re`、`typing`、`torch.fx`、`torch.utils._ordered_set`、`另有1项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含分支、循环或上下文管理等控制流。

### Lines 15-28 / 第 15-28 行
````python
def _get_module_stack(node: fx.Node) -> list[tuple[str, type[Any]]]:
    nn_stack = node.meta.get("nn_module_stack", "")
    if nn_stack:
        return list(nn_stack.values())

    fwd_nn_stack = node.meta.get("fwd_nn_module_stack", "")
    if fwd_nn_stack:
        return list(fwd_nn_stack.values())

    return []


def _addindent(s_: str, num_spaces: int) -> str:
    s: list[str] = s_.split("\n")
````
- **EN**: Introduces function `_get_module_stack`, function `_addindent`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nn_stack`, `fwd_nn_stack`, and `s`.
- **CN**: 这里定义了函数`_get_module_stack`、函数`_addindent`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `nn_stack`、`fwd_nn_stack`、`s` 等值。

### Lines 29-42 / 第 29-42 行
````python
    # don't do anything for single-line stuff
    if len(s) == 1:
        return s_
    first: str = s.pop(0)
    s: list[str] = [(num_spaces * " ") + line for line in s]
    joint_s: str = "\n".join(s)
    joint_s = first + "\n" + joint_s
    return joint_s


class GraphView:
    """
    A hierarchical class for organizing and managing torch.fx nodes by their module stack.

````
- **EN**: Introduces class `GraphView`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `first`, `s`, and `joint_s`.
- **CN**: 这里定义了类`GraphView`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `first`、`s`、`joint_s` 等值。

### Lines 43-56 / 第 43-56 行
````python
    This class provides a tree-like structure where each node in the hierarchy corresponds
    to a module or submodule in a traced FX graph. Each `GraphView` instance can hold a list
    of FX nodes (`self.data`) belonging to that module scope, maintain a unique set of nodes
    (`self.unique_nodes`), and manage its child containers (`self.children`).

    Attributes:
        name (str): The name of the module or container scope.
        klass (type[Any]): The class type associated with this module/container.
        data (list[fx.Node]): A list of FX graph nodes belonging to this module.
        unique_nodes (OrderedSet[fx.Node]): A deduplicated set of nodes to ensure no duplicates.
        children (dict[str, GraphView]): A mapping of child module names to their corresponding GraphView instances.
    """

    def __init__(self, name: str, klass: type[Any]) -> None:
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `Attributes`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `Attributes` 等值。

### Lines 57-70 / 第 57-70 行
````python
        self.name: str = name
        self.klass: type[Any] = klass
        self.data: list[fx.Node] = []
        self.unique_nodes: OrderedSet[fx.Node] = OrderedSet()
        self.children: dict[str, GraphView] = {}

    def add(self, data: fx.Node) -> None:
        if data not in self.unique_nodes:
            self.data.append(data)
            self.unique_nodes.add(data)

    def get_child(self, module_stack: str, klass: type[Any] | None = None) -> GraphView:
        if module_stack not in self.children:
            new_stack = GraphView(module_stack, klass or self.klass)
````
- **EN**: Introduces function `add`, function `get_child`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_stack`.
- **CN**: 这里定义了函数`add`、函数`get_child`。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_stack` 等值。

### Lines 71-84 / 第 71-84 行
````python
            self.children[module_stack] = new_stack
        return self.children[module_stack]

    def __getitem__(self, name: str) -> GraphView:
        return self.children[name]

    def __getattr__(self, name: str) -> GraphView:
        return self.children[name]

    def __repr__(self) -> str:
        child_lines: list[str] = []
        for name, child in self.children.items():
            mod_str = repr(child)
            mod_str = _addindent(mod_str, 2)
````
- **EN**: Introduces function `__getitem__`, function `__getattr__`, function `__repr__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `child_lines`, and `mod_str`.
- **CN**: 这里定义了函数`__getitem__`、函数`__getattr__`、函数`__repr__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `child_lines`、`mod_str` 等值。

### Lines 85-98 / 第 85-98 行
````python
            child_lines.append(f"({name}): {mod_str}")
        main_str = f"{self.klass.__name__}("
        if child_lines:
            main_str += "\n  " + "\n  ".join(child_lines) + "\n"
        main_str += ")"
        return main_str


def _clean_stack_name(stack_name: str) -> str:
    """
    Clean up FX node's nn_module_stack metadata string to match the module name hierarchies

    Example:
        Input: "L['self']._modules['layers']['0']._modules['attention']"
````
- **EN**: Introduces function `_clean_stack_name`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `main_str`, `Example`, and `Input`.
- **CN**: 这里定义了函数`_clean_stack_name`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `main_str`、`Example`、`Input` 等值。

### Lines 99-112 / 第 99-112 行
````python
        Output: "layers.0.attention"
    """
    cleaned = re.sub(r"^L\['self'\]\.?", "", stack_name)
    parts = re.findall(r"\['([^']+)'\]", cleaned)
    return ".".join(parts) if parts else cleaned


def _is_root(stack: str) -> bool:
    return stack == ""


def make_graph_view(
    graph: fx.Graph,
    module_stack_fn: Callable[[fx.Node], list[tuple[str, type[Any]]]] | None = None,
````
- **EN**: Introduces function `_is_root`, function `make_graph_view`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Output`, `cleaned`, `parts`, `graph`, and `module_stack_fn`.
- **CN**: 这里定义了函数`_is_root`、函数`make_graph_view`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Output`、`cleaned`、`parts`、`graph`、`module_stack_fn` 等值。

### Lines 113-126 / 第 113-126 行
````python
) -> GraphView | None:
    """
    Code from: https://github.com/meta-pytorch/autoparallel/pull/158

    Make a graph view from the fx.Graph. This is a tree structure that
    represents the module hierarchy of the graph, and enables us to
    easily find the nodes that belong to each module, and gives a slightly
    easier way of visualize different parts of the graph by extracting
    subgraphs that belong to a particular module FQN.

    For example, if we have the following model with module hierarchy:

    Transformer(
        (tok_embeddings): Embedding(128256, 4096)
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. This range continues the implementation of function `make_graph_view`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。这一段延续了函数`make_graph_view` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
        (layers): ModuleDict(
            (0): TransformerBlock(
            (attention): Attention(
                (wq): Linear(in_features=4096, out_features=4096, bias=False)
                (wk): Linear(in_features=4096, out_features=1024, bias=False)
                (wv): Linear(in_features=4096, out_features=1024, bias=False)
                (wo): Linear(in_features=4096, out_features=4096, bias=False)
                (sdpa): ScaledDotProductAttention()
            )
            (feed_forward): FeedForward(
                (w1): Linear(in_features=4096, out_features=14336, bias=False)
                (w2): Linear(in_features=14336, out_features=4096, bias=False)
                (w3): Linear(in_features=4096, out_features=14336, bias=False)
            )
````
- **EN**: This range continues the implementation of function `make_graph_view`.
- **CN**: 这一段延续了函数`make_graph_view` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
            (attention_norm): RMSNorm((4096,), eps=1e-05, elementwise_affine=True)
            (ffn_norm): RMSNorm((4096,), eps=1e-05, elementwise_affine=True)
            )
        )
        (norm): RMSNorm((4096,), eps=1e-05, elementwise_affine=True)
        (output): Linear(in_features=4096, out_features=128256, bias=False)
    )

    Then we can get a GraphView for the fx.Graph that enables us to do

    graph_view = make_graph_view(graph)
    subgraph = get_subgraph_by_path(graph_view, "layers.0")

    where subgraph contains all the nodes that belong to this region
````
- **EN**: Initializes or updates values such as `graph_view`, and `subgraph`. This range continues the implementation of function `make_graph_view`.
- **CN**: 初始化或更新了 `graph_view`、`subgraph` 等值。这一段延续了函数`make_graph_view` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python

    module_stack_fn: Optional callable for extracting module hierarchy information from nodes.

        Signature: Callable[[fx.Node], list[tuple[str, type[Any]]]]

        Takes an FX node and returns a list of (module_path, module_class) tuples representing
        the nested module hierarchy for that node, ordered from outermost to innermost scope.

        - module_path (str): Dot-separated path identifying the module in the hierarchy
          (e.g., "layers.0.attention.wq")
        - module_class (type): The Python class type of the module

        This enables custom logic for determining module membership, useful for:
        - Graphs without standard nn_module_stack metadata
````
- **EN**: Initializes or updates values such as `module_stack_fn`, and `Signature`. This range continues the implementation of function `make_graph_view`.
- **CN**: 初始化或更新了 `module_stack_fn`、`Signature` 等值。这一段延续了函数`make_graph_view` 的具体实现。

### Lines 169-182 / 第 169-182 行
````python
        - Filtering or grouping nodes by custom criteria

        Example of getting the module stack from annotation:

        def module_stack_fn(node):
            module_stack = node.meta.get("custom", {}).get("module_path", "")
            return [(module_stack, torch.nn.Module)]

        If None, defaults to extracting from node.meta["nn_module_stack"] or
        node.meta["fwd_nn_module_stack"].
    """

    def nn_module_stack_meta(node: fx.Node) -> list[tuple[str, type[Any]]]:
        result = []
````
- **EN**: Introduces function `module_stack_fn`, function `nn_module_stack_meta`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `module_stack`, and `result`.
- **CN**: 这里定义了函数`module_stack_fn`、函数`nn_module_stack_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `module_stack`、`result` 等值。

### Lines 183-196 / 第 183-196 行
````python
        for module_stack, module_class in _get_module_stack(node):
            module_stack = _clean_stack_name(module_stack)
            result.append((module_stack, module_class))
        return result

    if module_stack_fn is None:
        module_stack_fn = nn_module_stack_meta
    nodes: list[fx.Node] = list(graph.nodes)
    nodes_by_module_stack_root: GraphView | None = None
    for node in nodes:
        for module_stack, module_class in module_stack_fn(node):
            nodes_by_module_stack: GraphView | None = nodes_by_module_stack_root
            for name in module_stack.split("."):
                if nodes_by_module_stack is None:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `module_stack`, `module_stack_fn`, `nodes`, `nodes_by_module_stack_root`, and `nodes_by_module_stack`. This range continues the implementation of function `make_graph_view`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `module_stack`、`module_stack_fn`、`nodes`、`nodes_by_module_stack_root`、`nodes_by_module_stack` 等值。这一段延续了函数`make_graph_view` 的具体实现。

### Lines 197-210 / 第 197-210 行
````python
                    nodes_by_module_stack = GraphView(name, module_class)
                    nodes_by_module_stack_root = nodes_by_module_stack
                if _is_root(module_stack):
                    new_stack: GraphView = nodes_by_module_stack
                else:
                    new_stack = nodes_by_module_stack.get_child(name, module_class)
                nodes_by_module_stack = new_stack
                nodes_by_module_stack.add(node)

    return nodes_by_module_stack_root


def get_subgraph_by_path(
    graph_view: GraphView, paths: str | list[str]
````
- **EN**: Introduces function `get_subgraph_by_path`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nodes_by_module_stack`, `nodes_by_module_stack_root`, `new_stack`, `else`, and `graph_view`.
- **CN**: 这里定义了函数`get_subgraph_by_path`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `nodes_by_module_stack`、`nodes_by_module_stack_root`、`new_stack`、`else`、`graph_view` 等值。

### Lines 211-224 / 第 211-224 行
````python
) -> list[fx.Node]:
    """
    Get subgraph by path(s).
    Args:
        graph_view (object): Root graph view object.
        paths (str or list of str): Path(s) to subgraph.
    Returns:
        list[fx.Node]: fx nodes belong to the subgraph
    """

    def get_node_by_path(node: GraphView, path: str) -> GraphView:
        for p in path.split("."):
            if p in node.children:
                node = node.children[p]
````
- **EN**: Introduces function `get_node_by_path`. Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `Returns`, and `node`.
- **CN**: 这里定义了函数`get_node_by_path`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`Returns`、`node` 等值。

### Lines 225-238 / 第 225-238 行
````python
            else:
                return GraphView("", object)
        return node

    if isinstance(paths, list):
        nodes = list(
            itertools.chain.from_iterable(
                get_node_by_path(graph_view, p).data for p in paths
            )
        )
        return nodes
    else:
        node = get_node_by_path(graph_view, paths)
        return node.data
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `nodes`, and `node`. This range continues the implementation of function `get_subgraph_by_path`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`nodes`、`node` 等值。这一段延续了函数`get_subgraph_by_path` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary classes: `GraphView`  
  **CN**: 主要类：`GraphView`
- **EN**: Primary functions: `_get_module_stack`, `_addindent`, `_clean_stack_name`, `_is_root`, `make_graph_view`, and `get_subgraph_by_path`  
  **CN**: 主要函数：`_get_module_stack`、`_addindent`、`_clean_stack_name`、`_is_root`、`make_graph_view`、`get_subgraph_by_path`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `itertools`, `re`, `typing`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch.fx`, `torch.utils._ordered_set`
