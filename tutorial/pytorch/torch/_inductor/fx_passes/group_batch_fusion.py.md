# group_batch_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/group_batch_fusion.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `GroupBatchFusionBase`, `GroupFusion`, `BatchFusion`, `BatchPointwiseOpsFusionFactory`, `PostGradBatchLinearFusion`, `GroupLinearFusion`, and `...+22`. It exposes functions such as `update_stack_example_value`, `update_pointwise_example_value`, `register_fusion`, `list_group_batch_fusions`, `decompose_stack`, `_is_mutable_node`, and `...+6`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `GroupBatchFusionBase`、`GroupFusion`、`BatchFusion`、`BatchPointwiseOpsFusionFactory`、`PostGradBatchLinearFusion`、`GroupLinearFusion`、`另有22项` 等类。同时提供 `update_stack_example_value`、`update_pointwise_example_value`、`register_fusion`、`list_group_batch_fusions`、`decompose_stack`、`_is_mutable_node`、`另有6项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import collections
import logging
import operator
from collections import OrderedDict
from collections.abc import Iterable, Iterator
from typing import Any

import torch
from torch._dynamo.utils import counters, is_node_meta_valid
from torch._logging import trace_structured
from torch.fx.passes.graph_transform_observer import GraphTransformObserver
from torch.utils._ordered_set import OrderedSet

from .. import config
from ..pattern_matcher import (
    CallFunctionVarArgs,
    CallMethodVarArgs,
    get_arg_value,
    MatchResult,
    stable_topological_sort,
)
from ..utils import OPTIMUS_EXCLUDE_POST_GRAD


try:
    # importing this will register fbgemm lowerings for inductor
    import deeplearning.fbgemm.fbgemm_gpu.fb.inductor_lowerings  # noqa: F401
````
- **EN**: Imports dependencies such as `collections`, `logging`, `operator`, `collections.abc`, `typing`, `torch`, and `...+8` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`.
- **CN**: 这里导入了 `collections`、`logging`、`operator`、`collections.abc`、`typing`、`torch`、`另有8项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `try` 等值。

### Lines 29-56 / 第 29-56 行
````python

    has_fbgemm = True
except Exception:
    has_fbgemm = False

aten = torch.ops.aten

log = logging.getLogger(__name__)

DEFAULT_BETA = 1
DEFAULT_ALPHA = 1

MIN_FUSE_SET_SIZE = 5
MAX_FUSE_SET_SIZE = 300
MAX_FUSE_SEARCH_DEPTH = 5
# The maximum tensor size that can go into the fusion group
MAX_FUSE_TENSOR_SIZE_GROUP_LINEAR = 4096
# Whether we only fuse nodes with same parent node
FUSE_NODES_WITH_SAME_PARENT = False
# Whether we enable the add broadcast in batch linear
SHAPE_BROADCAST_BATCH_LINEAR = False
# Whether we enable the fuse nodes with same users
Fuse_NODES_WITH_SAME_USERS = False

# exclude these nodes from BFS
# excluding get item improves optimizer compilation time by 60s
SEARCH_EXCLUSIONS = OrderedSet([operator.getitem])

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_fbgemm`, `aten`, `log`, `DEFAULT_BETA`, `DEFAULT_ALPHA`, `MIN_FUSE_SET_SIZE`, and `...+7`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `has_fbgemm`、`aten`、`log`、`DEFAULT_BETA`、`DEFAULT_ALPHA`、`MIN_FUSE_SET_SIZE`、`另有7项` 等值。

### Lines 57-84 / 第 57-84 行
````python

default_graph_search_options = {
    "min_fuse_set_size": MIN_FUSE_SET_SIZE,
    "max_fuse_set_size": MAX_FUSE_SET_SIZE,
    "max_fuse_search_depth": MAX_FUSE_SEARCH_DEPTH,
    "max_fuse_tensor_size_group_linear": MAX_FUSE_TENSOR_SIZE_GROUP_LINEAR,
    "fuse_nodes_with_same_parent": FUSE_NODES_WITH_SAME_PARENT,
    "shape_broadcast_batch_linear": SHAPE_BROADCAST_BATCH_LINEAR,
    "fuse_nodes_with_same_users": Fuse_NODES_WITH_SAME_USERS,
}

graph_search_options = default_graph_search_options


def update_stack_example_value(node, metadata, dim=0, op=torch.stack):
    """
    Update the example value of the node in the graph to enable followup split cat opt.
    """
    if node is not None and hasattr(node, "meta"):
        if op is torch.stack:
            example_value = torch.stack(metadata, dim=dim)
        elif op is torch.unbind:
            example_value = torch.unbind(metadata, dim=dim)  # type: ignore[assignment]
        else:
            return
        node.meta["example_value"] = example_value


````
- **EN**: Introduces function `update_stack_example_value`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `default_graph_search_options`, `graph_search_options`, `example_value`, and `else`.
- **CN**: 这里定义了函数`update_stack_example_value`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `default_graph_search_options`、`graph_search_options`、`example_value`、`else` 等值。

### Lines 85-112 / 第 85-112 行
````python
def update_pointwise_example_value(pointwise_node, input, other, op):
    """
    Update the example value of the add node in the graph to enable followup split cat opt.
    """
    if pointwise_node is not None and hasattr(pointwise_node, "meta"):
        if op is torch.add:
            example_value = torch.add(input, other)
        elif op is torch.mul:
            example_value = torch.mul(input, other)
        else:
            return
        pointwise_node.meta["example_value"] = example_value


class GroupBatchFusionBase:
    def __init__(self, **kwargs) -> None:
        self.graph_search_options = kwargs.pop(
            "graph_search_options", default_graph_search_options
        )

    def match(self, node):
        raise NotImplementedError("match called on base")

    def fuse(self, graph, subset):
        raise NotImplementedError("fuse called on base")


PRE_GRAD_FUSIONS: dict[str, GroupBatchFusionBase] = {}
````
- **EN**: Introduces function `update_pointwise_example_value`, class `GroupBatchFusionBase`, function `__init__`, function `match`, function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `example_value`, `else`, and `PRE_GRAD_FUSIONS`.
- **CN**: 这里定义了函数`update_pointwise_example_value`、类`GroupBatchFusionBase`、函数`__init__`、函数`match`、函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `example_value`、`else`、`PRE_GRAD_FUSIONS` 等值。

### Lines 113-140 / 第 113-140 行
````python
POST_GRAD_FUSIONS: dict[str, GroupBatchFusionBase] = {}


def register_fusion(name: str, pre_grad=True):
    def decorator(fusion_cls: GroupBatchFusionBase):
        if pre_grad:
            PRE_GRAD_FUSIONS[name] = fusion_cls
        else:
            POST_GRAD_FUSIONS[name] = fusion_cls
        return fusion_cls

    return decorator


def list_group_batch_fusions(pre_grad=True) -> list[str]:
    if pre_grad:
        return list(PRE_GRAD_FUSIONS.keys())
    else:
        return list(POST_GRAD_FUSIONS.keys())


def decompose_stack(graph: torch.fx.GraphModule, input_tensors: list[Any]) -> Any:
    unsqueezed_inputs = []
    unsqueezed_inputs_meta = []
    for input_tensor in input_tensors:
        unsqueezed_input = graph.call_function(  # type: ignore[operator]
            aten.unsqueeze, args=(input_tensor,), kwargs={"dim": 0}
        )
````
- **EN**: Introduces function `register_fusion`, function `decorator`, function `list_group_batch_fusions`, function `decompose_stack`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`register_fusion`、函数`decorator`、函数`list_group_batch_fusions`、函数`decompose_stack`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-168 / 第 141-168 行
````python
        unsqueezed_inputs.append(unsqueezed_input)
        unsqueezed_input.meta["val"] = aten.unsqueeze(input_tensor.meta["val"], dim=0)  # type: ignore[assignment]
        unsqueezed_inputs_meta.append(unsqueezed_input.meta["val"])
    stacked_inputs = graph.call_function(  # type: ignore[operator]
        aten.cat, args=(unsqueezed_inputs,), kwargs={"dim": 0}
    )
    stacked_inputs.meta["val"] = aten.cat(unsqueezed_inputs_meta, dim=0)  # type: ignore[assignment]
    return stacked_inputs


class GroupFusion(GroupBatchFusionBase):
    """
    Fuse ops in a group way, e.g, fuse mm/addmm of arbitrary input shapes with fbgemm.gmm.
    """


class BatchFusion(GroupBatchFusionBase):
    """
    Fuse ops in a batch way, e.g, fuse mm/addmm of same input shapes with bmm.
    """


class BatchPointwiseOpsFusionFactory(BatchFusion):
    def __init__(self, op, **kwargs) -> None:
        super().__init__(**kwargs)
        self.op = op


````
- **EN**: Introduces class `GroupFusion`, class `BatchFusion`, class `BatchPointwiseOpsFusionFactory`, function `__init__`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `stacked_inputs`.
- **CN**: 这里定义了类`GroupFusion`、类`BatchFusion`、类`BatchPointwiseOpsFusionFactory`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `stacked_inputs` 等值。

### Lines 169-196 / 第 169-196 行
````python
@register_fusion("batch_linear_post_grad", pre_grad=False)
class PostGradBatchLinearFusion(BatchFusion):
    """
    Fuse ops in a batch way in post grad (aten level).
    """

    def _addmm_node_can_be_fused(self, node: torch.fx.Node) -> bool:
        # pyre-fixme[7]: Incompatible return type
        return (
            # pyrefly: ignore [bad-return]
            node.kwargs.get("beta", DEFAULT_BETA) == DEFAULT_BETA
            and node.kwargs.get("alpha", DEFAULT_ALPHA) == DEFAULT_ALPHA  # type: ignore[return-value]
        )

    def _is_input_2d(self, input: torch.fx.Node) -> bool:
        input_shapes = input.meta["val"].shape
        return (
            len(input_shapes) == 2
            and isinstance(input_shapes[0], int)
            and isinstance(input_shapes[1], int)
        )

    def match(self, node: torch.fx.Node) -> tuple[str, int, int, int, bool, str] | None:
        if CallFunctionVarArgs(aten.mm).match(node):
            input_m, weight_m = node.args
            bias_m = None

        elif CallFunctionVarArgs(aten.addmm.default).match(
````
- **EN**: Introduces class `PostGradBatchLinearFusion`, function `_addmm_node_can_be_fused`, function `_is_input_2d`, function `match`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`PostGradBatchLinearFusion`、函数`_addmm_node_can_be_fused`、函数`_is_input_2d`、函数`match`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 197-224 / 第 197-224 行
````python
            node
        ) and self._addmm_node_can_be_fused(node):
            bias_m, input_m, weight_m = node.args
        else:
            return None
        # get the user of the node
        if self.graph_search_options.get("fuse_nodes_with_same_users", False):
            users = [user.target for user in node.users]
        else:
            users = ""  # type: ignore[assignment]
        # only handle the cases where inputs are 2D tensors
        if not self._is_input_2d(input_m) or not self._is_input_2d(weight_m):  # type: ignore[arg-type]
            return None
        m, k = input_m.meta["val"].shape  # type: ignore[union-attr]
        n = weight_m.meta["val"].shape[1]  # type: ignore[union-attr]
        batch_key = ("batch_linear_post_grad", m, k, n, bias_m is not None, str(users))
        return batch_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        batch_inputs = []
        batch_weights = []
        batch_biases = []
        batch_nodes = []
        batch_inputs_meta = []
        batch_weights_meta = []
        batch_biases_meta = []

        for node in subset:
````
- **EN**: Introduces function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `users`, `n`, `batch_key`, `batch_inputs`, `batch_weights`, and `...+5`.
- **CN**: 这里定义了函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`users`、`n`、`batch_key`、`batch_inputs`、`batch_weights`、`另有5项` 等值。

### Lines 225-252 / 第 225-252 行
````python
            if CallFunctionVarArgs(aten.addmm.default).match(node):
                bias, input, weight = node.args
            elif CallFunctionVarArgs(aten.mm.default).match(node):
                input, weight = node.args
                bias = None
            batch_nodes.append(node)
            batch_inputs.append(input)  # type: ignore[possibly-undefined]
            batch_weights.append(weight)  # type: ignore[possibly-undefined]
            batch_biases.append(bias)  # type: ignore[possibly-undefined]
            batch_inputs_meta.append(input.meta)  # type: ignore[possibly-undefined, union-attr]
            batch_weights_meta.append(weight.meta)  # type: ignore[possibly-undefined, union-attr]
            if bias is not None:  # type: ignore[possibly-undefined]
                batch_biases_meta.append(bias.meta)  # type: ignore[possibly-undefined, union-attr]
            else:
                batch_biases_meta.append(None)

        with graph.inserting_before(subset[-1]):  # type: ignore[operator]
            fused_inputs = decompose_stack(graph, batch_inputs)
            fused_weights = decompose_stack(graph, batch_weights)
            fused_inputs_meta_val = torch.stack(
                [input["val"] for input in batch_inputs_meta]
            )
            fused_weights_meta_val = torch.stack(
                [weight["val"] for weight in batch_weights_meta]
            )
            fused_bmm = graph.call_function(  # type: ignore[operator]
                aten.bmm,
                args=(fused_inputs, fused_weights),
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bias`, `else`, `fused_inputs`, `fused_weights`, `fused_inputs_meta_val`, `fused_weights_meta_val`, and `...+2`. This range continues the implementation of function `PostGradBatchLinearFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bias`、`else`、`fused_inputs`、`fused_weights`、`fused_inputs_meta_val`、`fused_weights_meta_val`、`另有2项` 等值。这一段延续了函数`PostGradBatchLinearFusion.fuse` 的具体实现。

### Lines 253-280 / 第 253-280 行
````python
            )
            fused_bmm.meta["val"] = aten.bmm(
                fused_inputs_meta_val, fused_weights_meta_val
            )
        for i, original_mm in enumerate(batch_nodes):
            has_bias = False
            with graph.inserting_after(fused_bmm):  # type: ignore[operator]
                new_mm = graph.call_function(aten.select, args=((fused_bmm, 0, i)))  # type: ignore[operator]
                new_mm.meta["val"] = aten.select(fused_bmm.meta["val"], 0, i)
                if batch_biases[i]:
                    has_bias = True
                    # broadcast the bias to the same shape as the mm output
                    if self.graph_search_options.get(
                        "shape_broadcast_batch_linear", False
                    ):
                        broadcast_shape = torch.broadcast_shapes(
                            batch_biases_meta[i]["val"].shape, new_mm.meta["val"].shape
                        )
                        broadcast_bias = graph.call_function(  # type: ignore[operator]
                            aten.broadcast_to.default,
                            args=(batch_biases[i],),
                            kwargs={"size": broadcast_shape},
                        )
                        broadcast_bias.meta["val"] = aten.broadcast_to(
                            batch_biases_meta[i]["val"], broadcast_shape
                        )  # type: ignore[assignment]
                        new_bias_add = graph.call_function(  # type: ignore[operator]
                            aten.add.Tensor, args=((broadcast_bias, new_mm))
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_bias`, `new_mm`, `broadcast_shape`, `broadcast_bias`, `args`, `kwargs`, and `...+1`. This range continues the implementation of function `PostGradBatchLinearFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `has_bias`、`new_mm`、`broadcast_shape`、`broadcast_bias`、`args`、`kwargs`、`另有1项` 等值。这一段延续了函数`PostGradBatchLinearFusion.fuse` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
                        )
                        new_bias_add.meta["val"] = aten.add.Tensor(
                            broadcast_bias.meta["val"], new_mm.meta["val"]
                        )
                    else:
                        new_bias_add = graph.call_function(  # type: ignore[operator]
                            aten.add, args=((batch_biases[i], new_mm))
                        )
                        new_bias_add.meta["val"] = aten.add.Tensor(
                            batch_biases_meta[i]["val"], new_mm.meta["val"]
                        )
            new_mm_cont = new_bias_add if has_bias else new_mm  # type: ignore[possibly-undefined]
            original_mm.replace_all_uses_with(new_mm_cont)
            new_mm_cont.meta.update(original_mm.meta)
            graph.erase_node(original_mm)  # type: ignore[operator]
        counters["inductor"]["batch_linear_post_grad"] += 1


@register_fusion("group_linear", pre_grad=False)
class GroupLinearFusion(GroupFusion):
    def _addmm_node_can_be_fused(self, node: torch.fx.Node):
        input_shape = node.args[1].meta["val"].shape  # type: ignore[union-attr]
        weight_shape = node.args[2].meta["val"].shape  # type: ignore[union-attr]
        return (
            node.kwargs.get("beta", DEFAULT_BETA) == DEFAULT_BETA
            and node.kwargs.get("alpha", DEFAULT_ALPHA) == DEFAULT_ALPHA
            and len(input_shape) == 2
            and len(weight_shape) == 2
````
- **EN**: Introduces class `GroupLinearFusion`, function `_addmm_node_can_be_fused`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`GroupLinearFusion`、函数`_addmm_node_can_be_fused`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 309-336 / 第 309-336 行
````python
            and all(x % 2 == 0 for x in input_shape + weight_shape)
            and all(
                shape <= self.graph_search_options["max_fuse_tensor_size_group_linear"]
                for shape in input_shape + weight_shape
            )
        )

    def _mm_node_can_be_fused(self, node: torch.fx.Node):
        input_shape = node.args[0].meta["val"].shape  # type: ignore[union-attr]
        weight_shape = node.args[1].meta["val"].shape  # type: ignore[union-attr]
        return (
            len(input_shape) == 2
            and len(weight_shape) == 2
            and all(x % 2 == 0 for x in input_shape + weight_shape)
            and all(
                shape <= self.graph_search_options["max_fuse_tensor_size_group_linear"]
                for shape in input_shape + weight_shape
            )
        )

    def match(self, node: torch.fx.Node) -> tuple[str, bool] | None:
        if CallFunctionVarArgs(aten.mm.default).match(
            node
        ) and self._mm_node_can_be_fused(node):
            group_key = ("group_linear", True)
        elif CallFunctionVarArgs(aten.addmm.default).match(
            node
        ) and self._addmm_node_can_be_fused(node):
````
- **EN**: Introduces function `_mm_node_can_be_fused`, function `match`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_shape`, `weight_shape`, and `group_key`.
- **CN**: 这里定义了函数`_mm_node_can_be_fused`、函数`match`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_shape`、`weight_shape`、`group_key` 等值。

### Lines 337-364 / 第 337-364 行
````python
            bias = node.args[0]
            group_key = ("group_linear", bias is None)
        else:
            group_key = None
        return group_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        group_inputs = []
        group_weights = []
        group_biases = []
        group_nodes = []
        for node in subset:
            if CallFunctionVarArgs(aten.addmm.default).match(node):
                bias, input, weight = node.args
            else:
                assert CallFunctionVarArgs(aten.mm.default).match(node)
                input, weight = node.args
                bias = None

            group_nodes.append(node)
            group_inputs.append(input)
            group_weights.append(weight)
            group_biases.append(bias)

        if all(bias is None for bias in group_biases):
            group_biases = None  # type: ignore[assignment]

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
````
- **EN**: Introduces function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bias`, `group_key`, `else`, `group_inputs`, `group_weights`, `group_biases`, and `...+1`.
- **CN**: 这里定义了函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `bias`、`group_key`、`else`、`group_inputs`、`group_weights`、`group_biases`、`另有1项` 等值。

### Lines 365-392 / 第 365-392 行
````python
            fused_mm = graph.call_function(  # type: ignore[operator]
                torch.ops.fbgemm.gmm.default,
                args=(group_inputs, group_weights, group_biases),
                kwargs={"smart_fused": True},
            )

        for i, original_mm in enumerate(group_nodes):
            with graph.inserting_after(fused_mm):  # type: ignore[operator]
                new_mm = graph.call_function(operator.getitem, args=(fused_mm, i))  # type: ignore[operator]
            original_mm.replace_all_uses_with(new_mm)
            new_mm.meta.update(original_mm.meta)
            graph.erase_node(original_mm)  # type: ignore[operator]
        counters["inductor"]["group_linear"] += 1


class BatchPointwiseMathOpsPostGradFusion(BatchPointwiseOpsFusionFactory):
    """
    Batch pointwise math operator (e.g., add, mul) in post grad pass.
    """

    def __init__(self, op, **kwargs) -> None:
        super().__init__(op, **kwargs)
        self.op = op

    def _pointwise_node_can_be_fused(self, node: torch.fx.Node):
        # note: we only consider the case where the inputs are tensors
        # for mixed precision training, we need to make sure the inputs
        # of the aten.cat when do the stack should be the same dtype
````
- **EN**: Introduces class `BatchPointwiseMathOpsPostGradFusion`, function `__init__`, function `_pointwise_node_can_be_fused`. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fused_mm`, `args`, `kwargs`, and `new_mm`.
- **CN**: 这里定义了类`BatchPointwiseMathOpsPostGradFusion`、函数`__init__`、函数`_pointwise_node_can_be_fused`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `fused_mm`、`args`、`kwargs`、`new_mm` 等值。

### Lines 393-420 / 第 393-420 行
````python
        # otherwise, the output of the aten.cat may be not the same as
        # its inputs, and cause dtype not same error in mm or addmm
        input, other = node.args
        return (
            input.meta["val"].shape == other.meta["val"].shape  # type: ignore[union-attr]
            # input and other can be scalars, where they have no attribute 'meta'
            if hasattr(input, "meta")
            and hasattr(other, "meta")
            and is_node_meta_valid(input)  # type: ignore[arg-type, union-attr]
            and is_node_meta_valid(other)  # type: ignore[arg-type, union-attr]
            # torch.SymInt or torch.SymFloat object has no attribute 'shape'
            and isinstance(input.meta["val"], torch.Tensor)  # type: ignore[union-attr]
            and isinstance(other.meta["val"], torch.Tensor)  # type: ignore[union-attr]
            else False
        )

    def match(self, node: torch.fx.Node):
        if CallFunctionVarArgs(self.op).match(
            node
        ) and self._pointwise_node_can_be_fused(node):
            alpha = node.kwargs.get("alpha", DEFAULT_ALPHA)
            rounding_mode = node.kwargs.get("rounding_mode", None)
            input, other = node.args
            shape = list(input.meta["val"].shape)  # type: ignore[union-attr]
            if self.graph_search_options.get("fuse_nodes_with_same_parent", False):
                # only consider the linear case so far
                # pyre-fixme[16]
                if input.target is aten.select or other.target is aten.select:  # type: ignore[union-attr]
````
- **EN**: Introduces function `match`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`match`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 421-448 / 第 421-448 行
````python
                    parent = (
                        # pyre-fixme[16]
                        input.args[0]  # type: ignore[union-attr]
                        # pyre-fixme[16]
                        if input.target is aten.select  # type: ignore[union-attr]
                        else other.args[0]  # type: ignore[union-attr]
                    )
                else:
                    parent = ""
            else:
                parent = ""
            group_key = (
                "batch_aten_" + self.op.__name__.lower().split(".")[0],
                str(shape),
                str(input.meta["val"].dtype),  # type: ignore[union-attr]
                str(other.meta["val"].dtype),  # type: ignore[union-attr]
                str(alpha),
                str(rounding_mode),
                str(parent),
            )
        else:
            group_key = None
        return group_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        batch_inputs, batch_others = [], []
        alpha = subset[0].kwargs.get("alpha", DEFAULT_ALPHA)
        batch_inputs_meta, batch_others_meta = [], []
````
- **EN**: Introduces function `fuse`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`fuse`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 449-476 / 第 449-476 行
````python

        for node in subset:
            input, other = node.args
            batch_inputs.append(input)
            batch_others.append(other)
            batch_inputs_meta.append(input.meta)  # type: ignore[possibly-undefined, union-attr]
            batch_others_meta.append(other.meta)  # type: ignore[possibly-undefined, union-attr]

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
            stack_inputs = decompose_stack(graph, batch_inputs)
            stack_others = decompose_stack(graph, batch_others)
            stack_inputs_meta = torch.stack(
                [input["val"] for input in batch_inputs_meta]
            )
            stack_others_meta = torch.stack(
                [other["val"] for other in batch_others_meta]
            )

            batch_op = graph.call_function(  # type: ignore[operator]
                self.op,
                args=(stack_inputs, stack_others),
                kwargs={"alpha": alpha} if self.op == aten.add.Tensor else {},
            )
            batch_op.meta["val"] = self.op(stack_inputs_meta, stack_others_meta)
            for i, original_add in enumerate(subset):
                with graph.inserting_after(batch_op):  # type: ignore[operator]
                    new_add = graph.call_function(  # type: ignore[operator]
                        torch.ops.aten.select, args=((batch_op, 0, i))
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `stack_inputs`, `stack_others`, `stack_inputs_meta`, `stack_others_meta`, `batch_op`, `args`, and `...+2`. This range continues the implementation of function `BatchPointwiseMathOpsPostGradFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `stack_inputs`、`stack_others`、`stack_inputs_meta`、`stack_others_meta`、`batch_op`、`args`、`另有2项` 等值。这一段延续了函数`BatchPointwiseMathOpsPostGradFusion.fuse` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
                    )
                original_add.replace_all_uses_with(new_add)
                new_add.meta.update(original_add.meta)
                graph.erase_node(original_add)  # type: ignore[operator]
        counters["inductor"][
            "batch_aten_" + self.op.__name__.lower().split(".")[0]
        ] += 1


@register_fusion("batch_linear_lhs")
class BatchLinearLHSFusion(BatchFusion):
    """
    Batch linear left-hand side fusion. This pass tries to fuse the following patterns:

        torch.nn.functional.linear(x, w1), linear(x, w2),... * linear(x, wn)
        -> torch.mm(x, torch.cat([w1, w2,... * wn]).transpose(0, 1))

    We have a separate pass to eliminate contiguous transpose in a generic way.
    """

    def match(self, node: torch.fx.Node) -> tuple[str, bool, Any] | None:
        if CallFunctionVarArgs(torch.nn.functional.linear).match(
            node
        ) and is_linear_node_can_be_fused(node):
            input = get_arg_value(node, 0, "input")
            bias = get_arg_value(node, 2, "bias")
            group_key = ("batch_linear_lhs", bias is None, input)
        else:
````
- **EN**: Introduces class `BatchLinearLHSFusion`, function `match`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`BatchLinearLHSFusion`、函数`match`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 505-532 / 第 505-532 行
````python
            group_key = None
        return group_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        batch_nodes = []
        batch_input = None
        batch_weights, batch_weights_meta = [], []
        batch_biases, batch_biases_meta = [], []
        split_sections = []
        for node in subset:
            input = get_arg_value(node, 0, "input")
            weight = get_arg_value(node, 1, "weight")
            bias = get_arg_value(node, 2, "bias")
            batch_nodes.append(node)
            if batch_input is None:
                batch_input = input
            else:
                assert batch_input is input
            batch_weights.append(weight)
            batch_weights_meta.append(weight.meta["example_value"])
            if bias:
                batch_biases.append(bias)
                batch_biases_meta.append(bias.meta["example_value"])
            split_sections.append(weight.meta["example_value"].shape[0])

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
            cat_weights = graph.call_function(  # type: ignore[operator]
                torch.cat, args=(batch_weights,), kwargs={"dim": 0}
````
- **EN**: Introduces function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `group_key`, `batch_nodes`, `batch_input`, `split_sections`, `input`, `weight`, and `...+3`.
- **CN**: 这里定义了函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `group_key`、`batch_nodes`、`batch_input`、`split_sections`、`input`、`weight`、`另有3项` 等值。

### Lines 533-560 / 第 533-560 行
````python
            )
            cat_weights.meta["example_value"] = torch.cat(batch_weights_meta, dim=0)
            transposed_weights = graph.call_function(  # type: ignore[operator]
                torch.transpose, args=(cat_weights, 0, 1)
            )
            transposed_weights.meta["example_value"] = torch.transpose(
                cat_weights.meta["example_value"], 0, 1
            )
            if len(batch_biases) > 0:
                cat_biases = graph.call_function(  # type: ignore[operator]
                    torch.cat, args=(batch_biases,), kwargs={"dim": 0}
                )
                cat_biases.meta["example_value"] = torch.cat(batch_biases_meta, dim=0)
                fused_lhs = graph.call_function(  # type: ignore[operator]
                    torch.addmm,
                    args=(cat_biases, batch_input, transposed_weights),
                )
                fused_lhs.meta["example_value"] = torch.addmm(
                    cat_biases.meta["example_value"],
                    batch_input.meta["example_value"],  # type: ignore[union-attr]
                    transposed_weights.meta["example_value"],
                )
            else:
                fused_lhs = graph.call_function(  # type: ignore[operator]
                    torch.mm,
                    args=(batch_input, transposed_weights),
                )
                fused_lhs.meta["example_value"] = torch.mm(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `transposed_weights`, `cat_biases`, `fused_lhs`, `args`, and `else`. This range continues the implementation of function `BatchLinearLHSFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `transposed_weights`、`cat_biases`、`fused_lhs`、`args`、`else` 等值。这一段延续了函数`BatchLinearLHSFusion.fuse` 的具体实现。

### Lines 561-588 / 第 561-588 行
````python
                    batch_input.meta["example_value"],  # type: ignore[union-attr]
                    transposed_weights.meta["example_value"],
                )
            fused_lhs_list = graph.call_function(  # type: ignore[operator]
                torch.split, args=(fused_lhs, split_sections), kwargs={"dim": 1}
            )

        for i, node in enumerate(batch_nodes):
            with graph.inserting_after(fused_lhs_list):  # type: ignore[operator]
                new_node = graph.call_function(  # type: ignore[operator]
                    operator.getitem, args=(fused_lhs_list, i)
                )
            node.replace_all_uses_with(new_node)
            new_node.meta.update(node.meta)
            graph.erase_node(node)  # type: ignore[operator]
        counters["inductor"]["batch_linear_lhs"] += 1


# Poor person's check for if a node in the graph mutates its input.
# (the graph is torch IR, so we will see torch fns and python operators)
def _is_mutable_node(tgt):
    if str(tgt).endswith("_"):
        # e.g. torch.mul_, torch.Tensor.mul_
        return True
    if (
        hasattr(tgt, "__module__")
        and tgt.__module__ == "_operator"
        and tgt.__name__.startswith("i")
````
- **EN**: Introduces function `_is_mutable_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fused_lhs_list`, and `new_node`.
- **CN**: 这里定义了函数`_is_mutable_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fused_lhs_list`、`new_node` 等值。

### Lines 589-616 / 第 589-616 行
````python
    ):
        # e.g. operator.iand, operator.imul
        return True
    return False


def is_linear_node_can_be_fused(node: torch.fx.Node):
    input = get_arg_value(node, 0, "input")
    weight = get_arg_value(node, 1, "weight")
    return (
        is_node_meta_valid(node)
        and is_node_meta_valid(input)
        and is_node_meta_valid(weight)
        and len(input.meta["example_value"].shape) == 2
        and len(weight.meta["example_value"].shape) == 2
        # the mm -> bmm transform adds an unbind() op,
        # which is not safe for autograd when the output of the mm is mutated.
        # don't pattern match if any users of the mm mutate the input.
        and not any(_is_mutable_node(user.target) for user in node.users)
    )


@register_fusion("batch_linear")
class PreGradBatchLinearFusion(BatchFusion):
    """
    Batch linear fusion in pre grad pass.
    Fuse linear with same size with torch.baddmm
    """
````
- **EN**: Introduces function `is_linear_node_can_be_fused`, class `PreGradBatchLinearFusion`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`is_linear_node_can_be_fused`、类`PreGradBatchLinearFusion`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 617-644 / 第 617-644 行
````python

    def _getitem_args(self, getitem_node: torch.fx.Node):
        if getitem_node.target != operator.__getitem__ or (
            getitem_node.op != "call_function"
        ):
            return None
        return getitem_node.args[0]

    def match(self, node: torch.fx.Node):
        if CallFunctionVarArgs([torch.nn.functional.linear, torch._C._nn.linear]).match(
            node
        ) and is_linear_node_can_be_fused(node):
            input = get_arg_value(node, 0, "input")
            weight = get_arg_value(node, 1, "weight")
            bias = get_arg_value(node, 2, "bias")
            if self.graph_search_options.get("fuse_nodes_with_same_users", False):
                users = [user.target for user in node.users]
            else:
                users = ""  # type: ignore[assignment]
            group_key = (
                "batch_linear",
                self._getitem_args(input),
                str(input.meta["example_value"].shape),
                str(weight.meta["example_value"].shape),
                bias is None,
                str(users),
            )
        else:
````
- **EN**: Introduces function `_getitem_args`, function `match`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input`, `weight`, `bias`, `users`, `else`, and `group_key`.
- **CN**: 这里定义了函数`_getitem_args`、函数`match`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input`、`weight`、`bias`、`users`、`else`、`group_key` 等值。

### Lines 645-672 / 第 645-672 行
````python
            group_key = None
        return group_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        batch_nodes = []
        batch_inputs = []
        batch_weights = []
        batch_biases = []
        batch_inputs_metadata = []
        batch_weights_metadata = []
        batch_biases_metadata = []
        for node in subset:
            batch_nodes.append(node)
            input = get_arg_value(node, 0, "input")
            batch_inputs.append(input)
            batch_inputs_metadata.append(input.meta["example_value"])
            weight = get_arg_value(node, 1, "weight")
            batch_weights.append(weight)
            batch_weights_metadata.append(weight.meta["example_value"])
            bias = get_arg_value(node, 2, "bias")
            batch_biases.append(bias)
            if bias is not None and hasattr(bias, "meta"):
                batch_biases_metadata.append(bias.meta["example_value"])

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
            stack_inputs = graph.call_function(  # type: ignore[operator]
                torch.stack, args=(batch_inputs,), kwargs={"dim": 0}
            )
````
- **EN**: Introduces function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `group_key`, `batch_nodes`, `batch_inputs`, `batch_weights`, `batch_biases`, `batch_inputs_metadata`, and `...+6`.
- **CN**: 这里定义了函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `group_key`、`batch_nodes`、`batch_inputs`、`batch_weights`、`batch_biases`、`batch_inputs_metadata`、`另有6项` 等值。

### Lines 673-700 / 第 673-700 行
````python
            update_stack_example_value(stack_inputs, batch_inputs_metadata)
            stack_weights = graph.call_function(  # type: ignore[operator]
                torch.stack, args=(batch_weights,), kwargs={"dim": 0}
            )
            update_stack_example_value(stack_weights, batch_weights_metadata)
            transpose_weight = graph.call_function(  # type: ignore[operator]
                torch.transpose, args=(stack_weights, 1, 2)
            )
            transpose_weight.meta["example_value"] = torch.transpose(
                stack_weights.meta["example_value"], 1, 2
            )
            if all(bias is None for bias in batch_biases):
                bmm = graph.call_function(  # type: ignore[operator]
                    torch.bmm,
                    args=(stack_inputs, transpose_weight),
                )
                bmm.meta["example_value"] = torch.bmm(
                    stack_inputs.meta["example_value"],
                    transpose_weight.meta["example_value"],
                )
                bmm_meta = bmm.meta["example_value"]
            else:
                stack_biases = graph.call_function(  # type: ignore[operator]
                    torch.stack, args=(batch_biases,), kwargs={"dim": 0}
                )
                update_stack_example_value(stack_biases, batch_biases_metadata)
                unsqueeze_biases = graph.call_function(  # type: ignore[operator]
                    torch.unsqueeze, args=(stack_biases, 1)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `stack_weights`, `transpose_weight`, `bmm`, `args`, `bmm_meta`, `else`, and `...+2`. This range continues the implementation of function `PreGradBatchLinearFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `stack_weights`、`transpose_weight`、`bmm`、`args`、`bmm_meta`、`else`、`另有2项` 等值。这一段延续了函数`PreGradBatchLinearFusion.fuse` 的具体实现。

### Lines 701-728 / 第 701-728 行
````python
                )
                unsqueeze_biases.meta["example_value"] = torch.unsqueeze(
                    stack_biases.meta["example_value"], 1
                )
                bmm = graph.call_function(  # type: ignore[operator]
                    torch.baddbmm,
                    args=(unsqueeze_biases, stack_inputs, transpose_weight),
                )
                try:
                    # it will have runtime error to broadcast when it has dynamic shape included
                    # in the meta data, so we need to skip the update meta data
                    bmm.meta["example_value"] = torch.baddbmm(
                        unsqueeze_biases.meta["example_value"],
                        stack_inputs.meta["example_value"],
                        transpose_weight.meta["example_value"],
                    )
                    bmm_meta = bmm.meta["example_value"]
                except Exception as e:
                    log.debug(
                        f" exception when update bmm meta data with stack error tracekey {e}"  # noqa: G004
                    )
                    bmm_meta = None

            bmm = graph.call_function(torch.unbind, args=(bmm,), kwargs={"dim": 0})  # type: ignore[operator]
            if bmm_meta is not None:
                bmm.meta["example_value"] = torch.unbind(bmm_meta, dim=0)
            for i, linear in enumerate(batch_nodes):
                with graph.inserting_after(bmm):  # type: ignore[operator]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bmm`, `args`, `try`, and `bmm_meta`. This range continues the implementation of function `PreGradBatchLinearFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bmm`、`args`、`try`、`bmm_meta` 等值。这一段延续了函数`PreGradBatchLinearFusion.fuse` 的具体实现。

### Lines 729-756 / 第 729-756 行
````python
                    getitem = graph.call_function(operator.getitem, args=(bmm, i))  # type: ignore[operator]
                linear.replace_all_uses_with(getitem)
                getitem.meta.update(linear.meta)
                graph.erase_node(linear)  # type: ignore[operator]
        counters["inductor"]["batch_linear"] += 1


@register_fusion("batch_layernorm")
class BatchLayernormFusion(BatchFusion):
    """
    Batch layer norm fusion in pre grad pass
    """

    def match(self, node: torch.fx.Node):
        if CallFunctionVarArgs(torch.nn.functional.layer_norm).match(node):
            input = get_arg_value(node, 0, "input")
            weight = get_arg_value(node, 2, "weight")
            bias = get_arg_value(node, 3, "bias")
            if self.graph_search_options.get("fuse_nodes_with_same_users", False):
                users = [user.target for user in node.users]
            else:
                users = ""  # type: ignore[assignment]
            group_key = (
                (
                    "batch_layernorm",
                    str(input.meta["example_value"].shape),
                    str(weight.meta["example_value"].shape)
                    if weight is not None
````
- **EN**: Introduces class `BatchLayernormFusion`, function `match`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`BatchLayernormFusion`、函数`match`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 757-784 / 第 757-784 行
````python
                    else "",
                    str(bias.meta["example_value"].shape) if bias is not None else "",
                    str(get_arg_value(node, 1, "normalized_shape")),
                    str(get_arg_value(node, 4, "eps")),
                    str(users),
                )
                if "example_value" in input.meta
                and is_node_meta_valid(weight)
                and is_node_meta_valid(bias)
                else None
            )
        else:
            group_key = None
        return group_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        group_inputs = []
        group_shapes = []
        group_weights = []
        group_biases = []
        group_epss = []
        group_nodes = []
        group_inputs_metadata = []
        group_biases_metadata = []
        group_weights_metadata = []
        for node in subset:
            group_nodes.append(node)
            input = get_arg_value(node, 0, "input")
````
- **EN**: Introduces function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `group_key`, `group_inputs`, `group_shapes`, `group_weights`, `group_biases`, and `...+6`.
- **CN**: 这里定义了函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`group_key`、`group_inputs`、`group_shapes`、`group_weights`、`group_biases`、`另有6项` 等值。

### Lines 785-812 / 第 785-812 行
````python
            group_inputs.append(input)
            group_inputs_metadata.append(input.meta["example_value"])
            group_shapes.append(get_arg_value(node, 1, "normalized_shape"))
            weight = get_arg_value(node, 2, "weight")
            group_weights.append(weight)
            if weight is not None and hasattr(weight, "meta"):
                group_weights_metadata.append(weight.meta["example_value"])
            bias = get_arg_value(node, 3, "bias")
            group_biases.append(bias)
            if bias is not None and hasattr(bias, "meta"):
                group_biases_metadata.append(bias.meta["example_value"])
            eps = get_arg_value(node, 4, "eps")
            if eps is None:
                eps = 1e-5
            group_epss.append(eps)
        stack_dim = -1 - len(group_shapes[-1])

        if all(bias is None for bias in group_biases):
            group_biases = None  # type: ignore[assignment]
        if all(weight is None for weight in group_weights):
            group_weights = None  # type: ignore[assignment]
        assert all(eps == group_epss[0] for eps in group_epss), (
            "all epsilon values must be equal"
        )

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
            stack_input = graph.call_function(  # type: ignore[operator]
                torch.stack, args=(group_inputs,), kwargs={"dim": stack_dim}
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `weight`, `bias`, `eps`, `stack_dim`, `group_biases`, `group_weights`, and `...+1`. This range continues the implementation of function `BatchLayernormFusion.fuse`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `weight`、`bias`、`eps`、`stack_dim`、`group_biases`、`group_weights`、`另有1项` 等值。这一段延续了函数`BatchLayernormFusion.fuse` 的具体实现。

### Lines 813-840 / 第 813-840 行
````python
            )
            update_stack_example_value(stack_input, group_inputs_metadata, stack_dim)
            if group_weights is not None:
                stack_weight = graph.call_function(  # type: ignore[operator]
                    torch.stack, args=(group_weights,), kwargs={"dim": 0}
                )
                update_stack_example_value(stack_weight, group_weights_metadata)
            else:
                stack_weight = None
            if group_biases is not None:
                stack_bias = graph.call_function(  # type: ignore[operator]
                    torch.stack, args=(group_biases,), kwargs={"dim": 0}
                )
                update_stack_example_value(stack_bias, group_biases_metadata)
            else:
                stack_bias = None

            batch_layer_norm = graph.call_function(  # type: ignore[operator]
                torch.nn.functional.layer_norm,
                args=(stack_input, group_shapes[-1]),
                kwargs={"eps": group_epss[-1]},
            )
            batch_layer_norm.meta["example_value"] = stack_input.meta["example_value"]

            if group_weights is not None and group_biases is not None:
                previous_batch_layer_norm_meta = batch_layer_norm.meta["example_value"]
                batch_layer_norm = graph.call_function(  # type: ignore[operator]
                    torch.mul, args=(stack_weight, batch_layer_norm)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `stack_weight`, `else`, `stack_bias`, `batch_layer_norm`, `args`, `kwargs`, and `...+1`. This range continues the implementation of function `BatchLayernormFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `stack_weight`、`else`、`stack_bias`、`batch_layer_norm`、`args`、`kwargs`、`另有1项` 等值。这一段延续了函数`BatchLayernormFusion.fuse` 的具体实现。

### Lines 841-868 / 第 841-868 行
````python
                )
                update_pointwise_example_value(
                    batch_layer_norm,
                    # pyrefly: ignore [missing-attribute]
                    stack_weight.meta["example_value"],
                    previous_batch_layer_norm_meta,
                    torch.mul,
                )
                previous_batch_layer_norm_meta = batch_layer_norm.meta["example_value"]
                batch_layer_norm = graph.call_function(  # type: ignore[operator]
                    torch.add, args=(stack_bias, batch_layer_norm)
                )
                update_pointwise_example_value(
                    batch_layer_norm,
                    # pyrefly: ignore [missing-attribute]
                    stack_bias.meta["example_value"],
                    previous_batch_layer_norm_meta,
                    torch.add,
                )
            elif group_weights is not None and group_biases is None:
                previous_batch_layer_norm_meta = batch_layer_norm.meta["example_value"]
                # pyrefly: ignore [not-callable]
                batch_layer_norm = graph.call_function(
                    torch.mul, args=(stack_weight, batch_layer_norm)
                )
                update_pointwise_example_value(
                    batch_layer_norm,
                    # pyrefly: ignore [missing-attribute]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `previous_batch_layer_norm_meta`, and `batch_layer_norm`. This range continues the implementation of function `BatchLayernormFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `previous_batch_layer_norm_meta`、`batch_layer_norm` 等值。这一段延续了函数`BatchLayernormFusion.fuse` 的具体实现。

### Lines 869-896 / 第 869-896 行
````python
                    stack_weight.meta["example_value"],
                    previous_batch_layer_norm_meta,
                    torch.mul,
                )
            elif group_weights is None and group_biases is not None:
                previous_batch_layer_norm_meta = batch_layer_norm.meta["example_value"]
                # pyrefly: ignore [not-callable]
                batch_layer_norm = graph.call_function(
                    torch.add, args=(stack_bias, batch_layer_norm)
                )
                update_pointwise_example_value(
                    batch_layer_norm,
                    # pyrefly: ignore [missing-attribute]
                    stack_bias.meta["example_value"],
                    previous_batch_layer_norm_meta,
                    torch.add,
                )

            batch_layer_norm_unbind = graph.call_function(  # type: ignore[operator]
                torch.unbind,
                args=(batch_layer_norm,),
                kwargs={"dim": stack_dim},
            )
            update_stack_example_value(
                batch_layer_norm_unbind,
                batch_layer_norm.meta["example_value"],
                op=torch.unbind,
                dim=stack_dim,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `previous_batch_layer_norm_meta`, `batch_layer_norm`, `batch_layer_norm_unbind`, `args`, `kwargs`, `op`, and `...+1`. This range continues the implementation of function `BatchLayernormFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `previous_batch_layer_norm_meta`、`batch_layer_norm`、`batch_layer_norm_unbind`、`args`、`kwargs`、`op`、`另有1项` 等值。这一段延续了函数`BatchLayernormFusion.fuse` 的具体实现。

### Lines 897-924 / 第 897-924 行
````python
            )

        for i, node in enumerate(group_nodes):
            with graph.inserting_after(batch_layer_norm_unbind):  # type: ignore[operator]
                new_node = graph.call_function(  # type: ignore[operator]
                    operator.getitem, args=(batch_layer_norm_unbind, i)
                )
            node.replace_all_uses_with(new_node)
            new_node.meta.update(node.meta)
            graph.erase_node(node)  # type: ignore[operator]
        counters["inductor"]["batch_layernorm"] += 1


class BatchPointwiseOpsPreGradFusion(BatchPointwiseOpsFusionFactory):
    """
    Batch pointwise ops (e.g., sigmoid, relu, tanh) fusion in pre grad pass.
    We fuse it in random place, and the introduced stack node may be merged in split cat.
    """

    def __init__(self, op, **kwargs) -> None:
        super().__init__(op, **kwargs)
        self.op = op

    def match(self, node: torch.fx.Node):
        input = get_arg_value(node, 0, "input")
        if CallFunctionVarArgs(self.op).match(node) and is_node_meta_valid(node):
            if self.graph_search_options.get("fuse_nodes_with_same_parent", False):
                # pyre-fixme[16]
````
- **EN**: Introduces class `BatchPointwiseOpsPreGradFusion`, function `__init__`, function `match`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_node`, and `input`.
- **CN**: 这里定义了类`BatchPointwiseOpsPreGradFusion`、函数`__init__`、函数`match`。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_node`、`input` 等值。

### Lines 925-952 / 第 925-952 行
````python
                parent = node.args[0]
                parent = parent.target if parent is not None else ""  # type: ignore[union-attr]
            else:
                parent = ""
            # for relu op, we also use the inplace to construct the key
            group_key = (
                "batch_" + self.op.__name__.lower().split(".")[0],
                str(input.meta["example_value"].shape),
                str(node.kwargs.get("inplace", False)),
                str(parent),
            )
        else:
            group_key = None
        return group_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        batch_nodes = []
        batch_inputs = []
        batch_inputs_metadata = []

        for node in subset:
            batch_nodes.append(node)
            input = get_arg_value(node, 0, "input")
            batch_inputs.append(input)
            batch_inputs_metadata.append(input.meta["example_value"])

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
            stack_inputs = graph.call_function(  # type: ignore[operator]
````
- **EN**: Introduces function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `parent`, `else`, `group_key`, `batch_nodes`, `batch_inputs`, `batch_inputs_metadata`, and `...+2`.
- **CN**: 这里定义了函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `parent`、`else`、`group_key`、`batch_nodes`、`batch_inputs`、`batch_inputs_metadata`、`另有2项` 等值。

### Lines 953-980 / 第 953-980 行
````python
                torch.stack, args=(batch_inputs,), kwargs={"dim": 0}
            )
            update_stack_example_value(stack_inputs, batch_inputs_metadata)
            if self.op is torch.nn.functional.relu:
                batch_op = graph.call_function(  # type: ignore[operator]
                    self.op,
                    args=(stack_inputs,),
                    kwargs={"inplace": subset[0].kwargs.get("inplace", False)},
                )
                batch_op.meta["example_value"] = self.op(
                    stack_inputs.meta["example_value"],
                    # pyrefly: ignore [bad-argument-type]
                    inplace=subset[0].kwargs.get("inplace", False),
                )
            else:
                batch_op = graph.call_function(  # type: ignore[operator]
                    self.op,
                    args=(stack_inputs,),
                )
                batch_op.meta["example_value"] = self.op(
                    stack_inputs.meta["example_value"]
                )
            unbind_op = graph.call_function(  # type: ignore[operator]
                torch.unbind, args=(batch_op,), kwargs={"dim": 0}
            )
            unbind_op.meta["example_value"] = torch.unbind(
                batch_op.meta["example_value"], dim=0
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `batch_op`, `args`, `kwargs`, `inplace`, `else`, and `unbind_op`. This range continues the implementation of function `BatchPointwiseOpsPreGradFusion.fuse`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `batch_op`、`args`、`kwargs`、`inplace`、`else`、`unbind_op` 等值。这一段延续了函数`BatchPointwiseOpsPreGradFusion.fuse` 的具体实现。

### Lines 981-1008 / 第 981-1008 行
````python
            for i, node in enumerate(batch_nodes):
                with graph.inserting_after(unbind_op):  # type: ignore[operator]
                    getitem = graph.call_function(operator.getitem, args=(unbind_op, i))  # type: ignore[operator]
                node.replace_all_uses_with(getitem)
                getitem.meta.update(node.meta)
                graph.erase_node(node)  # type: ignore[operator]
        counters["inductor"]["batch_" + self.op.__name__.lower().split(".")[0]] += 1


class BatchPointwiseOpsPostGradFusion(BatchPointwiseOpsFusionFactory):
    """
    Batch pointwise ops (e.g., sigmoid, relu, tanh) fusion in post grad pass.
    The introduced stack node may be merged in split cat.
    """

    def __init__(self, op, **kwargs) -> None:
        super().__init__(op, **kwargs)
        self.op = op

    def match(self, node: torch.fx.Node):
        input = get_arg_value(node, 0, "input")
        if CallFunctionVarArgs(self.op).match(node) and is_node_meta_valid(node):
            # for relu op, we also use the inplace to construct the key
            # we batch the ops with same parent to enable followup split cat
            parent = node.args[0]
            parent = (
                parent.target  # type: ignore[union-attr]
                if self.graph_search_options.get("fuse_nodes_with_same_parent", False)
````
- **EN**: Introduces class `BatchPointwiseOpsPostGradFusion`, function `__init__`, function `match`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `getitem`, `input`, and `parent`.
- **CN**: 这里定义了类`BatchPointwiseOpsPostGradFusion`、函数`__init__`、函数`match`。包含分支、循环或上下文管理等控制流。初始化或更新了 `getitem`、`input`、`parent` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
                else ""
            )
            group_key = (
                "batch_aten_" + self.op.__name__.lower().split(".")[0],
                str(input.meta["val"].shape),
                str(node.kwargs.get("inplace", False)),
                # pyre-fixme[16]
                str(parent),
            )
        else:
            group_key = None
        return group_key

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        batch_nodes = []
        batch_inputs = []
        batch_inputs_metadata = []

        for node in subset:
            batch_nodes.append(node)
            input = get_arg_value(node, 0, "input")
            batch_inputs.append(input)
            batch_inputs_metadata.append(input.meta["val"])

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
            stack_inputs = decompose_stack(graph, batch_inputs)
            update_stack_example_value(stack_inputs, batch_inputs_metadata)
            batch_op = graph.call_function(  # type: ignore[operator]
````
- **EN**: Introduces function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `group_key`, `else`, `batch_nodes`, `batch_inputs`, `batch_inputs_metadata`, `input`, and `...+2`.
- **CN**: 这里定义了函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `group_key`、`else`、`batch_nodes`、`batch_inputs`、`batch_inputs_metadata`、`input`、`另有2项` 等值。

### Lines 1037-1064 / 第 1037-1064 行
````python
                self.op,
                args=(stack_inputs,),
            )
            for i, node in enumerate(batch_nodes):
                with graph.inserting_after(batch_op):  # type: ignore[operator]
                    getitem = graph.call_function(aten.select, args=(batch_op, 0, i))  # type: ignore[operator]
                node.replace_all_uses_with(getitem)
                getitem.meta.update(node.meta)
                graph.erase_node(node)  # type: ignore[operator]
        counters["inductor"][
            "batch_aten_" + self.op.__name__.lower().split(".")[0]
        ] += 1


class BatchMathOpsPreGradFusion(BatchPointwiseOpsFusionFactory):
    """
    Batch simple match related ops such as nan_to_num in pre grad pass.
    """

    def __init__(self, op, **kwargs):
        super().__init__(op, **kwargs)
        self.op = op

    def match(self, node: torch.fx.Node):
        input = get_arg_value(node, 0, "input")
        if self._match_op(node) and is_node_meta_valid(node):
            # check the input has the same shape and its users have the same target
            # check all clamp operators have the same min and max values, and
````
- **EN**: Introduces class `BatchMathOpsPreGradFusion`, function `__init__`, function `match`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args`, `getitem`, and `input`.
- **CN**: 这里定义了类`BatchMathOpsPreGradFusion`、函数`__init__`、函数`match`。包含分支、循环或上下文管理等控制流。初始化或更新了 `args`、`getitem`、`input` 等值。

### Lines 1065-1092 / 第 1065-1092 行
````python
            # nan_to_num operators use the same default value.
            child = next(iter(node.users.keys()))
            group_key = (
                str(input.meta["example_value"].shape)
                + str(node.kwargs)
                + str(child.target)
            )
        else:
            group_key = None
        return group_key

    def _match_op(self, node: torch.fx.Node) -> MatchResult:
        return CallFunctionVarArgs(self.op).match(node)

    def fuse(self, graph: torch.fx.GraphModule, subset: list[torch.fx.Node]):
        batch_nodes = []
        batch_inputs = []
        batch_inputs_metadata = []
        kwargs = subset[0].kwargs

        for node in subset:
            batch_nodes.append(node)
            input = get_arg_value(node, 0, "input")
            batch_inputs.append(input)
            batch_inputs_metadata.append(input.meta["example_value"])

        with graph.inserting_before(subset[0]):  # type: ignore[operator]
            stack_inputs = graph.call_function(  # type: ignore[operator]
````
- **EN**: Introduces function `_match_op`, function `fuse`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `child`, `group_key`, `else`, `batch_nodes`, `batch_inputs`, `batch_inputs_metadata`, and `...+3`.
- **CN**: 这里定义了函数`_match_op`、函数`fuse`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `child`、`group_key`、`else`、`batch_nodes`、`batch_inputs`、`batch_inputs_metadata`、`另有3项` 等值。

### Lines 1093-1120 / 第 1093-1120 行
````python
                torch.stack, args=(batch_inputs,), kwargs={"dim": 0}
            )
            update_stack_example_value(stack_inputs, batch_inputs_metadata)
            batch_op = graph.call_function(  # type: ignore[operator]
                self.op,
                args=(stack_inputs,),
                kwargs=kwargs,
            )
            batch_op.meta["example_value"] = self.op(
                stack_inputs.meta["example_value"], **kwargs
            )
            unbind_op = graph.call_function(  # type: ignore[operator]
                torch.unbind, args=(batch_op,), kwargs={"dim": 0}
            )
            unbind_op.meta["example_value"] = torch.unbind(
                batch_op.meta["example_value"], dim=0
            )
            for i, node in enumerate(batch_nodes):
                with graph.inserting_after(unbind_op):  # type: ignore[operator]
                    getitem = graph.call_function(operator.getitem, args=(unbind_op, i))  # type: ignore[operator]
                node.replace_all_uses_with(getitem)
                getitem.meta.update(node.meta)
                graph.erase_node(node)  # type: ignore[operator]
        counters["inductor"]["batch_" + self.op.__name__.lower().split(".")[0]] += 1


@register_fusion("batch_tanh")
class BatchTanhPreGradFusion(BatchPointwiseOpsPreGradFusion):
````
- **EN**: Introduces class `BatchTanhPreGradFusion`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`BatchTanhPreGradFusion`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 1121-1148 / 第 1121-1148 行
````python
    def __init__(self, **kwargs) -> None:
        super().__init__(torch.tanh, **kwargs)


@register_fusion("batch_sigmoid")
class BatchSigmoidPreGradFusion(BatchPointwiseOpsPreGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(torch.sigmoid, **kwargs)


@register_fusion("batch_relu")
class BatchReLuPreGradFusion(BatchPointwiseOpsPreGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(torch.nn.functional.relu, **kwargs)


@register_fusion("batch_detach")
class BatchDetachPreGradFusion(BatchMathOpsPreGradFusion):
    def __init__(self, **kwargs):
        super().__init__(torch.detach, **kwargs)

    def _match_op(self, node: torch.fx.Node) -> MatchResult:
        return CallFunctionVarArgs(torch.detach).match(node) or CallMethodVarArgs(
            "detach"
        ).match(node)


@register_fusion("batch_nan_to_num")
````
- **EN**: Introduces function `__init__`, class `BatchSigmoidPreGradFusion`, function `__init__`, class `BatchReLuPreGradFusion`, function `__init__`, class `BatchDetachPreGradFusion`, function `__init__`, function `_match_op`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`__init__`、类`BatchSigmoidPreGradFusion`、函数`__init__`、类`BatchReLuPreGradFusion`、函数`__init__`、类`BatchDetachPreGradFusion`、函数`__init__`、函数`_match_op`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1149-1176 / 第 1149-1176 行
````python
class BatchNanToNumPreGradFusion(BatchMathOpsPreGradFusion):
    def __init__(self, **kwargs):
        super().__init__(torch.nan_to_num, **kwargs)


@register_fusion("batch_clamp")
class BatchClampPreGradFusion(BatchMathOpsPreGradFusion):
    def __init__(self, **kwargs):
        super().__init__(torch.clamp, **kwargs)


@register_fusion("batch_dropout")
class BatchDropoutPreGradFusion(BatchMathOpsPreGradFusion):
    def __init__(self, **kwargs):
        super().__init__(torch.nn.functional.dropout, **kwargs)


@register_fusion("batch_aten_tanh", pre_grad=False)
class BatchTanhPostGradFusion(BatchPointwiseOpsPostGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(aten.tanh.default, **kwargs)


@register_fusion("batch_aten_sigmoid", pre_grad=False)
class BatchSigmoidPostGradFusion(BatchPointwiseOpsPostGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(aten.sigmoid.default, **kwargs)

````
- **EN**: Introduces class `BatchNanToNumPreGradFusion`, function `__init__`, class `BatchClampPreGradFusion`, function `__init__`, class `BatchDropoutPreGradFusion`, function `__init__`, class `BatchTanhPostGradFusion`, function `__init__`, class `BatchSigmoidPostGradFusion`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`BatchNanToNumPreGradFusion`、函数`__init__`、类`BatchClampPreGradFusion`、函数`__init__`、类`BatchDropoutPreGradFusion`、函数`__init__`、类`BatchTanhPostGradFusion`、函数`__init__`、类`BatchSigmoidPostGradFusion`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1177-1204 / 第 1177-1204 行
````python

@register_fusion("batch_aten_relu", pre_grad=False)
class BatchReLuPostGradFusion(BatchPointwiseOpsPostGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(aten.relu.default, **kwargs)


@register_fusion("batch_aten_add", pre_grad=False)
class BatchAddPostGradFusion(BatchPointwiseMathOpsPostGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(aten.add.Tensor, **kwargs)


@register_fusion("batch_aten_sub", pre_grad=False)
class BatchSubPostGradFusion(BatchPointwiseMathOpsPostGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(aten.sub.Tensor, **kwargs)


@register_fusion("batch_aten_div", pre_grad=False)
class BatchDivPostGradFusion(BatchPointwiseMathOpsPostGradFusion):
    def __init__(self, **kwargs) -> None:
        super().__init__(aten.div.Tensor, **kwargs)


@register_fusion("batch_aten_mul", pre_grad=False)
class BatchMulPostGradFusion(BatchPointwiseMathOpsPostGradFusion):
    def __init__(self, **kwargs) -> None:
````
- **EN**: Introduces class `BatchReLuPostGradFusion`, function `__init__`, class `BatchAddPostGradFusion`, function `__init__`, class `BatchSubPostGradFusion`, function `__init__`, class `BatchDivPostGradFusion`, function `__init__`, class `BatchMulPostGradFusion`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`BatchReLuPostGradFusion`、函数`__init__`、类`BatchAddPostGradFusion`、函数`__init__`、类`BatchSubPostGradFusion`、函数`__init__`、类`BatchDivPostGradFusion`、函数`__init__`、类`BatchMulPostGradFusion`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1205-1232 / 第 1205-1232 行
````python
        super().__init__(aten.mul.Tensor, **kwargs)


class _OrderedSet:
    def __init__(self, param=None) -> None:
        if param:
            self.rep = OrderedDict(dict.fromkeys(param))
        else:
            self.rep = OrderedDict()

    def __contains__(self, o) -> bool:
        return o in self.rep

    def __len__(self) -> int:
        return self.rep.__len__()

    def append(self, o):
        self.rep[o] = None

    def __iter__(self):
        return self.rep.keys().__iter__()


def find_independent_subset_greedy(
    node_list: Iterable[torch.fx.Node],
    graph_search_options: dict[str, Any],
) -> Iterator[Iterable[torch.fx.Node]]:
    """
````
- **EN**: Introduces class `_OrderedSet`, function `__init__`, function `__contains__`, function `__len__`, function `append`, function `__iter__`, function `find_independent_subset_greedy`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `node_list`, and `graph_search_options`.
- **CN**: 这里定义了类`_OrderedSet`、函数`__init__`、函数`__contains__`、函数`__len__`、函数`append`、函数`__iter__`、函数`find_independent_subset_greedy`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`node_list`、`graph_search_options` 等值。

### Lines 1233-1260 / 第 1233-1260 行
````python
    Yields a list of subsets of `node_list` where no element in the subset
    depends on any other element in the subset. This results in a set of
    independent nodes which can be fused together.

    The order of `node_list` is preserved within each subset so we can benefit
    from split-cat elimination in later passes.

    During iteration it is only safe to mutate the graph by changing the nodes
    that have been returned.

    graph_search_options:
      - min_fuse_set_size: Minimum size of the subset to consider. Subsets below
        this size will be ignored.
      - max_fuse_set_size: Maximum size of the subset to consider. Subsets will
        be broken to be at most this size.
    """

    # Compute all the children of `node` which are members of
    # `interesting_nodes`.
    def find_dependent_nodes(node, interesting_nodes):
        visited_node_set = OrderedSet[torch.fx.Node]()
        dep_set = OrderedSet[torch.fx.Node]()

        work = [node]
        while work:
            node = work.pop()
            for input_node in node.all_input_nodes:
                if input_node in interesting_nodes:
````
- **EN**: Introduces function `find_dependent_nodes`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph_search_options`, `visited_node_set`, `dep_set`, `work`, and `node`.
- **CN**: 这里定义了函数`find_dependent_nodes`。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph_search_options`、`visited_node_set`、`dep_set`、`work`、`node` 等值。

### Lines 1261-1288 / 第 1261-1288 行
````python
                    dep_set.add(input_node)

                if input_node not in visited_node_set:
                    visited_node_set.add(input_node)
                    work.append(input_node)

        return dep_set

    min_fuse_set_size = graph_search_options["min_fuse_set_size"]
    max_fuse_set_size = graph_search_options["max_fuse_set_size"]

    # node_list needs to be a set because we only track the nodes that are left
    # in it (and we want to do the `in` on a set, not a list). But we want to
    # keep the correct order.
    node_list = _OrderedSet(node_list)

    cache: dict[torch.fx.Node, OrderedSet[torch.fx.Node]] = {}
    while node_list:
        subset: list[torch.fx.Node] = []
        subset_deps = OrderedSet[torch.fx.Node]()

        next_round_node_list = _OrderedSet()
        for node in node_list:
            if len(subset) >= max_fuse_set_size or node in subset_deps:
                next_round_node_list.append(node)
                continue

            dep_set = cache.pop(node, None)
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `min_fuse_set_size`, `max_fuse_set_size`, `node_list`, `cache`, `subset`, `subset_deps`, and `...+2`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `min_fuse_set_size`、`max_fuse_set_size`、`node_list`、`cache`、`subset`、`subset_deps`、`另有2项` 等值。

### Lines 1289-1316 / 第 1289-1316 行
````python
            if dep_set is None:
                dep_set = find_dependent_nodes(node, node_list)

            if not dep_set.intersection(subset):
                subset.append(node)
                subset_deps.update(dep_set)
            else:
                next_round_node_list.append(node)
                cache[node] = dep_set

        if len(subset) >= min_fuse_set_size:
            # Careful here - the caller uses the subsets to fuse nodes together
            # so we need to clear any cache entry that contains one of the
            # returned nodes because the dependency list could be different
            # (larger) after the merge.
            cache = {k: v for k, v in cache.items() if v.isdisjoint(subset)}
            yield subset

        node_list = next_round_node_list


def get_fusion_candidates(
    rule: GroupBatchFusionBase,
    root_node: torch.fx.Node,
    fused_set: OrderedSet[torch.fx.Node],
) -> collections.defaultdict[Any, list[torch.fx.Node]]:
    """
    Search fusion candidates for a specific rule using BFS starting from the root node.
````
- **EN**: Introduces function `get_fusion_candidates`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_fusion_candidates`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1317-1344 / 第 1317-1344 行
````python
    We only search the subgraph within graph_search_options["max_fuse_search_depth"].
    """
    q: collections.deque[tuple[int, torch.fx.Node]] = collections.deque()

    candidate_dict: collections.defaultdict[Any, list[torch.fx.Node]] = (
        collections.defaultdict(list)
    )

    if root_node.target in SEARCH_EXCLUSIONS:
        return candidate_dict

    visited_set = OrderedSet[torch.fx.Node]()

    for next_node in root_node.all_input_nodes:
        q.append((1, next_node))
        visited_set.add(next_node)

    while len(q) > 0:
        depth, node = q.popleft()

        if node in fused_set:
            continue

        key = rule.match(node)
        if key is not None:
            candidate_nodes = candidate_dict[key]
            if node not in candidate_nodes:
                candidate_nodes.append(node)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `q`, `candidate_dict`, `visited_set`, `key`, and `candidate_nodes`. This range continues the implementation of function `get_fusion_candidates`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `q`、`candidate_dict`、`visited_set`、`key`、`candidate_nodes` 等值。这一段延续了函数`get_fusion_candidates` 的具体实现。

### Lines 1345-1372 / 第 1345-1372 行
````python
        else:
            if depth < rule.graph_search_options["max_fuse_search_depth"]:
                for next_node in node.all_input_nodes:
                    if next_node not in visited_set:
                        visited_set.add(next_node)
                        q.append((depth + 1, next_node))

    return candidate_dict


def apply_group_batch_fusion(graph: torch.fx.GraphModule, rule: GroupBatchFusionBase):
    stable_topological_sort(graph)  # type: ignore[arg-type]
    fused_set = OrderedSet[torch.fx.Node]()
    log_to_scuba = False

    for node in reversed(graph.nodes):  # type: ignore[arg-type]
        candidates = get_fusion_candidates(rule, node, fused_set)

        for key, candidate_nodes in candidates.items():
            if len(candidate_nodes) < rule.graph_search_options["min_fuse_set_size"]:
                continue

            for subset in find_independent_subset_greedy(
                candidate_nodes, rule.graph_search_options
            ):
                rule.fuse(graph, subset)
                fused_set.update(subset)
                log.debug(
````
- **EN**: Introduces function `apply_group_batch_fusion`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `fused_set`, `log_to_scuba`, and `candidates`.
- **CN**: 这里定义了函数`apply_group_batch_fusion`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`fused_set`、`log_to_scuba`、`candidates` 等值。

### Lines 1373-1400 / 第 1373-1400 行
````python
                    f"{rule.__class__.__name__}: key = {key}; subset size = {len(list(subset))}"  # noqa: G004
                )
                log_to_scuba = True
    if log_to_scuba:
        from torch.fx._lazy_graph_module import _LazyGraphModule

        # Force graph to re-compile otherwise the output python code may be broken
        gm = graph._owning_module
        if isinstance(gm, _LazyGraphModule):
            _LazyGraphModule.recompile()
        else:
            assert isinstance(gm, torch.fx.GraphModule)
            gm.recompile()
        graph_str = gm.print_readable(
            print_output=False, include_stride=True, include_device=True
        )

        name = f"optimus_{str(rule.__class__.__name__)}"
        if "MTIA" in name:
            name = f"cff_{str(rule.__class__.__name__)}"
        trace_structured(
            "artifact",
            metadata_fn=lambda: {
                "name": name,
                "encoding": "string",
            },
            payload_fn=lambda: graph_str,
        )
````
- **EN**: Imports dependencies such as `torch.fx._lazy_graph_module` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `log_to_scuba`, `gm`, `else`, `graph_str`, `print_output`, `name`, and `...+2`.
- **CN**: 这里导入了 `torch.fx._lazy_graph_module` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `log_to_scuba`、`gm`、`else`、`graph_str`、`print_output`、`name`、`另有2项` 等值。

### Lines 1401-1428 / 第 1401-1428 行
````python


def generate_fusion_from_config(config_options: dict[str, Any], pre_grad=True):
    fusions: list[GroupBatchFusionBase] = []
    for name, options in config_options.items():
        # we skip all patterns from pattern_matcher passes (e.g., split_cat)
        if name not in PRE_GRAD_FUSIONS and name not in POST_GRAD_FUSIONS:
            continue
        fusion_cls = PRE_GRAD_FUSIONS[name] if pre_grad else POST_GRAD_FUSIONS[name]
        _options = graph_search_options.copy()
        _options.update(options)
        fusions.append(fusion_cls(graph_search_options=_options))  # type: ignore[operator]
    return fusions


def group_batch_fusion_passes(graph: torch.fx.Graph, pre_grad=True):
    fusions: list[GroupBatchFusionBase] = []
    # we keep all current pre grad fusions to keep
    # current implementation, will remove this later
    if pre_grad:
        fusions += generate_fusion_from_config(
            config.pre_grad_fusion_options, pre_grad=True
        )
    else:
        fbgemm_fusion_keys = [
            x
            for x in config.post_grad_fusion_options
            if (
````
- **EN**: Introduces function `generate_fusion_from_config`, function `group_batch_fusion_passes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fusions`, `fusion_cls`, `_options`, `else`, and `fbgemm_fusion_keys`.
- **CN**: 这里定义了函数`generate_fusion_from_config`、函数`group_batch_fusion_passes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fusions`、`fusion_cls`、`_options`、`else`、`fbgemm_fusion_keys` 等值。

### Lines 1429-1451 / 第 1429-1451 行
````python
                x not in OPTIMUS_EXCLUDE_POST_GRAD
                and config.post_grad_fusion_options[x].get("require_fbgemm", False)
            )
        ]
        fbgemm_fusions = {
            fusion: config.post_grad_fusion_options[fusion]
            for fusion in fbgemm_fusion_keys
        }
        non_fbgemm_fusions = {
            fusion: config.post_grad_fusion_options[fusion]
            for fusion in config.post_grad_fusion_options
            if fusion not in fbgemm_fusion_keys
        }
        fusions += generate_fusion_from_config(non_fbgemm_fusions, pre_grad=False)
        if has_fbgemm:
            fusions += generate_fusion_from_config(fbgemm_fusions, pre_grad=False)

    for i, rule in enumerate(fusions):
        with GraphTransformObserver(
            graph.owning_module,
            f"group_batch_fusion_{i}",
        ):
            apply_group_batch_fusion(graph, rule)  # type: ignore[arg-type]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fbgemm_fusions`, `fusion`, and `non_fbgemm_fusions`. This range continues the implementation of function `group_batch_fusion_passes`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `fbgemm_fusions`、`fusion`、`non_fbgemm_fusions` 等值。这一段延续了函数`group_batch_fusion_passes` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `GroupBatchFusionBase`, `GroupFusion`, `BatchFusion`, `BatchPointwiseOpsFusionFactory`, `PostGradBatchLinearFusion`, `GroupLinearFusion`, and `...+22`  
  **CN**: 主要类：`GroupBatchFusionBase`、`GroupFusion`、`BatchFusion`、`BatchPointwiseOpsFusionFactory`、`PostGradBatchLinearFusion`、`GroupLinearFusion`、`另有22项`
- **EN**: Primary functions: `update_stack_example_value`, `update_pointwise_example_value`, `register_fusion`, `list_group_batch_fusions`, `decompose_stack`, `_is_mutable_node`, and `...+6`  
  **CN**: 主要函数：`update_stack_example_value`、`update_pointwise_example_value`、`register_fusion`、`list_group_batch_fusions`、`decompose_stack`、`_is_mutable_node`、`另有6项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `logging`, `operator`, `collections.abc`, `typing`
- **Third-party / 第三方**: `deeplearning.fbgemm.fbgemm_gpu.fb.inductor_lowerings`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch._logging`, `torch.fx.passes.graph_transform_observer`, `torch.utils._ordered_set`, `..`, `..pattern_matcher`, `..utils`, `torch.fx._lazy_graph_module`
