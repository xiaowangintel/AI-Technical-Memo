# dce_extra_outputs.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/dce_extra_outputs.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: DCE pass for unused extra outputs in HOP subgraphs.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
"""
DCE pass for unused extra outputs in HOP subgraphs.

When enable_side_effects_with_extra_outputs is True, HOPs like invoke_subgraph and
checkpoint (tag_activation_checkpoint)
return all intermediate tensors/symints as extra outputs to support side effects.
However, many of these extra outputs may not actually be used in the parent graph.

This pass removes unused extra outputs by:
1. Collecting all callers for each subgraph
2. Checking if each output is used by all callers
3. Removing unused outputs from the subgraph's output node
4. Updating the HOP call and getitem indices in all call sites

"""
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 17-34
```python
import collections
import operator

import torch


# HOPs that may have extra outputs that can be DCE'd
_HOPS_WITH_EXTRA_OUTPUTS = {
    torch.ops.higher_order.invoke_subgraph,
    torch.ops.higher_order.tag_activation_checkpoint,
    # torch.ops.higher_order.autograd_function_apply,
}


def dce_hop_extra_outputs(gm: torch.fx.GraphModule) -> bool:
    """
    Remove unused extra outputs from HOP calls in all submodules.
```
- **EN**: This range initializes module-level constants or registries that later code reuses to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以构建、遍历或改写图结构及其元数据。

### Lines 35-51
```python
    For each subgraph output, check if any caller has a getitem for that index
    with users. If no caller uses it, remove the output.
    If the user in caller is an output node, to simply the algorithm, we do not recursively check
    if the caller's output is used further up in the call chain.

    Args:
        gm: The GraphModule to optimize

    Returns:
        True if any modifications were made, False otherwise
    """
    # Collect all subgraph usages: subgraph_id -> list of (parent_gm, subgraph_name, hop_node)
    subgraph_id_to_callers: dict[
        int, list[tuple[torch.fx.GraphModule, str, torch.fx.Node]]
    ] = collections.defaultdict(list)
    _collect_all_subgraph_usages(gm, subgraph_id_to_callers)
```
- **EN**: This block continues `dce_hop_extra_outputs` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `dce_hop_extra_outputs`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 52-68
```python
    if not subgraph_id_to_callers:
        return False

    modified = False

    for callers in subgraph_id_to_callers.values():
        parent_gm, subgraph_name, _ = callers[0]
        subgraph = getattr(parent_gm, subgraph_name)

        if not isinstance(subgraph, torch.fx.GraphModule):
            continue

        output_node = next(n for n in subgraph.graph.nodes if n.op == "output")
        output_args = output_node.args[0]
        if not isinstance(output_args, (tuple, list)):
            continue
```
- **EN**: This block continues `dce_hop_extra_outputs` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `dce_hop_extra_outputs`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 69-84
```python
        num_outputs = len(output_args)
        used_indices: set[int] = set()

        # Check which outputs are used by any caller
        for idx in range(num_outputs):
            if _is_output_used(idx, callers):
                used_indices.add(idx)

        # DCE if some outputs are unused
        if 0 < len(used_indices) < num_outputs:
            if _dce_subgraph(subgraph, callers, used_indices):
                modified = True

    return modified
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 85-96
```python
def _collect_all_subgraph_usages(
    gm: torch.fx.GraphModule,
    subgraph_id_to_callers: dict[
        int, list[tuple[torch.fx.GraphModule, str, torch.fx.Node]]
    ],
) -> None:
    """Recursively collect all HOP usages across the graph tree."""
    for node in gm.graph.nodes:
        if node.op == "call_function" and node.target in _HOPS_WITH_EXTRA_OUTPUTS:
            subgraph_attr = node.args[0]
            if (
                isinstance(subgraph_attr, torch.fx.Node)
```
- **EN**: Defines the `_collect_all_subgraph_usages` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`_collect_all_subgraph_usages` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 97-109
```python
                and subgraph_attr.op == "get_attr"
            ):
                subgraph_name = subgraph_attr.target
                assert isinstance(subgraph_name, str)
                subgraph = getattr(gm, subgraph_name, None)
                if isinstance(subgraph, torch.fx.GraphModule):
                    subgraph_id = id(subgraph)
                    subgraph_id_to_callers[subgraph_id].append(
                        (gm, subgraph_name, node)
                    )
                    _collect_all_subgraph_usages(subgraph, subgraph_id_to_callers)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 110-122
```python
def _is_output_used(
    output_idx: int,
    callers: list[tuple[torch.fx.GraphModule, str, torch.fx.Node]],
) -> bool:
    """Check if output_idx is used by ANY caller (has a getitem with users)."""
    for _parent_gm, _subgraph_name, hop_node in callers:
        for user in hop_node.users:
            if user.op == "call_function" and user.target == operator.getitem:
                if user.args[1] == output_idx and len(user.users) > 0:
                    return True
    return False
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 123-136
```python
def _dce_subgraph(
    subgraph: torch.fx.GraphModule,
    callers: list[tuple[torch.fx.GraphModule, str, torch.fx.Node]],
    used_indices: set[int],
) -> bool:
    """
    DCE a subgraph by removing unused output indices.

    Updates the subgraph's output node, all getitem nodes in callers,
    and example_value metadata on HOP nodes.
    """
    output_node = next(n for n in subgraph.graph.nodes if n.op == "output")
    old_outputs = list(output_node.args[0])
```
- **EN**: Defines the `_dce_subgraph` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`_dce_subgraph` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 137-151
```python
    # Check if this is the forward subgraph of autograd_function_apply
    # For autograd_function_apply, the fwd subgraph must return (output, saved_values, ...)
    # where indices 0 and 1 are ALWAYS required by the runtime
    # is_autograd_fwd = any(
    #     node.target == torch.ops.higher_order.autograd_function_apply
    #     for node in hop_nodes
    # )
    is_autograd_fwd = False

    # For autograd_function_apply forward subgraph, indices 0 (output) and 1 (saved_values)
    # are ALWAYS used by the runtime, even if not explicitly accessed via getitem
    if is_autograd_fwd and len(old_outputs) >= 2:
        used_indices.add(0)  # output
        used_indices.add(1)  # saved_values
```
- **EN**: This block continues `_dce_subgraph` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_dce_subgraph`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 152-166
```python
    # Nothing to DCE if all outputs are used or no outputs are used
    if len(used_indices) >= len(old_outputs) or len(used_indices) == 0:
        return False

    # Build mapping from old indices to new indices
    old_to_new: dict[int, int] = {}
    new_outputs = []
    new_idx = 0

    for old_idx in range(len(old_outputs)):
        if old_idx in used_indices:
            old_to_new[old_idx] = new_idx
            new_outputs.append(old_outputs[old_idx])
            new_idx += 1
```
- **EN**: This block continues `_dce_subgraph` and works to capture Python execution for torch.compile and maintain compiler state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_dce_subgraph`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 167-180
```python
    # Update subgraph output node
    # Create a new output node with the filtered outputs
    with subgraph.graph.inserting_before(output_node):
        new_output_node = subgraph.graph.output(tuple(new_outputs))
    output_node.replace_all_uses_with(new_output_node)
    subgraph.graph.erase_node(output_node)

    for parent_gm, _, hop_node in callers:
        # Update getitem nodes to use new indices
        for user in list(hop_node.users):
            if user.op == "call_function" and user.target == operator.getitem:
                old_idx = user.args[1]
                assert isinstance(old_idx, int)
```
- **EN**: This block continues `_dce_subgraph` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `_dce_subgraph`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 181-196
```python
                if old_idx not in old_to_new:
                    assert len(list(user.users)) == 0
                    parent_gm.graph.erase_node(user)
                    continue

                new_idx = old_to_new[old_idx]
                # Create a new getitem node with the new index
                with parent_gm.graph.inserting_before(user):
                    new_getitem = parent_gm.graph.call_function(
                        operator.getitem, args=(user.args[0], new_idx)
                    )
                    # Copy metadata from old node
                    new_getitem.meta = user.meta.copy()
                user.replace_all_uses_with(new_getitem)
                parent_gm.graph.erase_node(user)
```
- **EN**: This block continues `_dce_subgraph` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `_dce_subgraph`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 197-211
```python
        # Update example_value metadata on hop_node
        if "example_value" in hop_node.meta:
            old_example = hop_node.meta["example_value"]
            assert isinstance(old_example, (tuple, list))
            new_example = tuple(
                old_example[old_idx]
                for old_idx in range(len(old_outputs))
                if old_idx in used_indices
            )
            hop_node.meta["example_value"] = new_example

    # Recompile subgraph and all modified parent graphs
    subgraph.graph.lint()
    subgraph.recompile()
```
- **EN**: This block continues `_dce_subgraph` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_dce_subgraph`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 212-216
```python
    for parent_gm in {caller[0] for caller in callers}:
        parent_gm.graph.lint()
        parent_gm.recompile()

    return True
```
- **EN**: This block continues `_dce_subgraph` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `_dce_subgraph`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

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
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`
- **Standard library / 标准库**: `collections`, `operator`
- **Primary symbols / 核心符号**: `_HOPS_WITH_EXTRA_OUTPUTS`, `dce_hop_extra_outputs`, `_collect_all_subgraph_usages`, `_is_output_used`, `_dce_subgraph`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
