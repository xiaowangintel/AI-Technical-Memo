# merge_matmul.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/merge_matmul.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```python
import itertools
import operator

import torch
from torch.fx._symbolic_trace import symbolic_trace
from torch.fx.node import Node
from torch.fx.passes.tools_common import legalize_graph


def split_result_tensors(
    result: torch.Tensor, inputs: list[torch.Tensor]
) -> tuple[torch.Tensor, ...]:
    """
    A free function for use in the merge_matmul graph transformation below that
    splits the output from a merged matmul into the individual results for each
    input tensor.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 18-34
```python
    Arguments:
        result: The merged matmul result tensor.
        inputs: The list of inputs that were merged into one for the matmul.

    Returns:
        List of matmul results for each input tensor.
    """
    # When fx tracer is running, x.shape[0] will be torch.fx.Attribute but we
    # need an int even when tracing
    if isinstance(result, torch.fx.Proxy):
        splits = [0] * len(inputs)
    else:
        splits = [x.shape[0] for x in inputs]

    return torch.split(result, splits)
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 35-47
```python
def may_depend_on(a: Node, b: Node, search_depth: int = 6) -> bool:
    """
    Determine if one node depends on another in a torch.fx.Graph.

    Arguments:
        a: The node that may have a dependency on b.
        b: The node that a may have a dependency on.
        search_depth: In the case of an indirect dependency, this function
                        searches upto this many nodes away in search of a
                        data dependency. If none is found, the function
                        makes the conservative assumption that there is a
                        dependency.
```
- **EN**: Defines the `may_depend_on` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`may_depend_on` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 48-63
```python
    Returns:
        True if a may depend on b, False if it definitely does not.
    """
    # Equivalence is defined as dependence.
    if a == b:
        return True

    # If a has no inputs, it cannot depend on b.
    if len(a.all_input_nodes) == 0:
        return False

    # If the search depth has been exhausted and no conclusion has been
    # reached, assume that there is a data dependency.
    if search_depth == 0:
        return True
```
- **EN**: This block continues `may_depend_on` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `may_depend_on`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 64-78
```python
    # Recursively check all inputs of a.
    for inp in a.all_input_nodes:
        if may_depend_on(inp, b, search_depth - 1):
            return True

    return False


def are_nodes_independent(nodes: list[Node]) -> bool:
    """
    Check if all of the given nodes are pairwise-data independent.

    Arguments:
        nodes: The nodes to check for data dependencies.
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 79-96
```python
    Returns:
        True if any pair in nodes has a data dependency.
    """
    # For each pair in nodes:
    for i, j in itertools.combinations(nodes, 2):
        if may_depend_on(i, j) or may_depend_on(j, i):
            return False

    return True


def merge_matmul(in_mod: torch.nn.Module) -> torch.fx.GraphModule:
    """
    A graph transformation that merges matrix multiplication operations that share the same right-hand
    side operand into one large matrix multiplication.

    ::
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 97-114
```python
                   ____      _________        _________
          ----    |    |    |         |     M|  A * C  |
        M| A  |  T| B  | * K|    C    | =    |---------|
          ---- ,  |    |    |         |     T|  B * C  |
           K       ----      ---------        ---------
                    K            R                R
    """
    gm = symbolic_trace(in_mod)

    rhs_users: dict[Node, list[Node]] = {}
    lhs_users: dict[Node, list[Node]] = {}

    # Populate rhs_users and lhs_users - maps from LHS/RHS matrix multiply operands to
    # the matmul of which they are the LHS/RHS.
    for node in gm.graph.nodes:
        if node.op != "call_function" or node.target is not torch.matmul:
            continue
```
- **EN**: This block continues `merge_matmul` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `merge_matmul`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 115-130
```python
        lhs, rhs = node.args

        # TODO: Properly handle aliasing caused by get_attr. For now,
        # use the attribute name as the operand if the node is a
        # get_attr.
        lhs = lhs.target if lhs.op == "get_attr" else lhs
        rhs = rhs.target if rhs.op == "get_attr" else rhs

        lhs_users.setdefault(lhs, []).append(node)
        rhs_users.setdefault(rhs, []).append(node)

    for rhs, mms in rhs_users.items():
        # There must be at least matmuls for a merge to make sense.
        if len(mms) < 2:
            continue
```
- **EN**: This block continues `merge_matmul` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `merge_matmul`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 131-145
```python
        # All matmuls must not depend on each other directly or indirectly
        # in order for the merge to be possible.
        if not are_nodes_independent(mms):
            continue

        lhs_vals = [mm.args[0] for mm in mms]

        # Merge the matmul.
        # Collect a list of LHS operands and the single RHS operand.
        lhs = [gm.graph.get_attr(l) if isinstance(l, str) else l for l in lhs_vals]
        rhs = gm.graph.get_attr(rhs) if isinstance(rhs, str) else rhs

        # Concatenate all the LHS operands.
        merge_mm_cat = gm.graph.call_function(torch.cat, (lhs,), {})
```
- **EN**: This block continues `merge_matmul` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `merge_matmul`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 146-157
```python
        # Multiply the concatenated LHS operands with the one RHS. This will produce
        # the same results as all the individual matmuls involving rhs in the original graph,
        # but they will all be concatenated together.
        merge_mm = gm.graph.call_function(
            torch.matmul,
            (
                merge_mm_cat,
                rhs,
            ),
            {},
        )
```
- **EN**: This block continues `merge_matmul` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `merge_matmul`，用于构建、遍历或改写图结构及其元数据。

### Lines 158-172
```python
        # Split the result of the merged matmul using the shapes of the LHS operands
        # to ascertain how large each chunk should be.
        merge_mm_split = gm.graph.call_function(
            split_result_tensors, (merge_mm, lhs), {}
        )
        merge_mm_res = [
            gm.graph.call_function(operator.getitem, (merge_mm_split, out), {})
            for out in range(len(lhs))
        ]

        # Replace all uses of the original, unmerged matmuls with the equivalent split chunk from the merged matmul.
        for old, new in zip(mms, merge_mm_res):
            old.replace_all_uses_with(new)
            gm.graph.erase_node(old)
```
- **EN**: This block continues `merge_matmul` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `merge_matmul`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构。

### Lines 173-179
```python
        # All of the new nodes created above were inserted at the end, so we need to sort
        # the nodes topologically to make sure all definitions precede uses.
        legalize_graph(gm)

    gm.recompile()
    gm.graph.lint()
    return gm
```
- **EN**: This block continues `merge_matmul` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `merge_matmul`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx._symbolic_trace`, `torch.fx.node`, `torch.fx.passes.tools_common`
- **Standard library / 标准库**: `itertools`, `operator`
- **Primary symbols / 核心符号**: `split_result_tensors`, `may_depend_on`, `are_nodes_independent`, `merge_matmul`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
