# joint_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/joint_graph.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `UniformValueConstantFolder`. It exposes functions such as `_is_lossless_fp_widening_cast`, `lazy_init`, `remove_no_ops`, `remove_redundant_views`, `_has_self_referential_shape`, `constant_fold_uniform_value`, and `...+16`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `UniformValueConstantFolder` 等类。同时提供 `_is_lossless_fp_widening_cast`、`lazy_init`、`remove_no_ops`、`remove_redundant_views`、`_has_self_referential_shape`、`constant_fold_uniform_value`、`另有16项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import functools
import itertools
import logging
import operator
import typing
from collections import Counter
from collections.abc import Sequence
from typing import Any

import torch
import torch._guards
import torch.utils._pytree as pytree
from torch._dynamo.utils import counters
from torch._inductor.constant_folding import ConstantFolder
from torch._inductor.fx_passes.dedupe_symint_uses import _SymHashingDict
from torch._inductor.utils import get_gpu_type
from torch.fx.experimental.symbolic_shapes import (
    guard_or_false,
    guard_or_true,
    statically_known_true,
)
from torch.multiprocessing.reductions import StorageWeakRef
from torch.utils._ordered_set import OrderedSet

from .. import config
from ..pattern_matcher import (
    Arg,
````
- **EN**: Imports dependencies such as `functools`, `itertools`, `logging`, `operator`, `typing`, `collections`, and `...+13` for the logic in this range.
- **CN**: 这里导入了 `functools`、`itertools`、`logging`、`operator`、`typing`、`collections`、`另有13项` 等依赖，为后续逻辑提供基础能力。

### Lines 29-56 / 第 29-56 行
````python
    CallFunction,
    init_once_fakemode,
    KeywordArg,
    Match,
    MULTIPLE,
    PatternMatcherPass as PatternMatcherPassBase,
    register_graph_pattern,
    stable_topological_sort,
)
from .decompose_mem_bound_mm import check_device
from .replace_random import replace_random_passes


PatternMatcherPass = functools.partial(
    PatternMatcherPassBase, subsystem="joint_graph_passes"
)

log = logging.getLogger(__name__)
early_patterns = PatternMatcherPass()
patterns = PatternMatcherPass()
aten = torch.ops.aten
prims = torch.ops.prims

pass_patterns = [
    patterns,
    PatternMatcherPass(),
]

````
- **EN**: Imports dependencies such as `.decompose_mem_bound_mm`, and `.replace_random` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `PatternMatcherPass`, `log`, `early_patterns`, `patterns`, `aten`, `prims`, and `...+1`.
- **CN**: 这里导入了 `.decompose_mem_bound_mm`、`.replace_random` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `PatternMatcherPass`、`log`、`early_patterns`、`patterns`、`aten`、`prims`、`另有1项` 等值。

### Lines 57-84 / 第 57-84 行
````python

def _is_lossless_fp_widening_cast(
    src_dtype: torch.dtype, dst_dtype: torch.dtype
) -> bool:
    if src_dtype == dst_dtype:
        return True

    if not (src_dtype.is_floating_point and dst_dtype.is_floating_point):
        return False

    src_info = torch.finfo(src_dtype)
    dst_info = torch.finfo(dst_dtype)

    # A floating-point cast is only pointless if the first conversion cannot
    # discard precision or range from the source values.
    return (
        dst_info.eps <= src_info.eps
        and dst_info.max >= src_info.max
        and dst_info.tiny <= src_info.tiny
    )


@init_once_fakemode
def lazy_init(input_device: torch.device | None = None):
    from .fuse_attention import _sfdp_init
    from .misc_patterns import _misc_patterns_init
    from .pad_mm import _pad_mm_init

````
- **EN**: Imports dependencies such as `.fuse_attention`, `.misc_patterns`, and `.pad_mm` for the logic in this range. Introduces function `_is_lossless_fp_widening_cast`, function `lazy_init`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里导入了 `.fuse_attention`、`.misc_patterns`、`.pad_mm` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_is_lossless_fp_widening_cast`、函数`lazy_init`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。

### Lines 85-112 / 第 85-112 行
````python
    _pad_mm_init(input_device)
    _sfdp_init(input_device)
    _misc_patterns_init(input_device)


def remove_no_ops(
    gm: torch.fx.GraphModule,
    zeros: OrderedSet[torch.fx.Node],
    ones: OrderedSet[torch.fx.Node],
):
    """Remove identity arithmetic operations: (+ 0, - 0, * 1, / 1)."""
    with torch.utils._python_dispatch._disable_current_modes():
        graph = gm.graph

        def fake_tensors_eq(t1, t2, fields=("shape", "dtype", "device")):
            if any(not isinstance(t, torch.Tensor) for t in (t1, t2)):
                return False
            for field in fields:
                if getattr(t1, field) != getattr(t2, field):
                    return False
            return True

        def is_mutated(n):
            """Check if a node is mutated by any in-place operation."""
            for user in n.users:
                if user.op != "call_function" or not hasattr(user.target, "_schema"):
                    continue
                for i, arg in enumerate(user.args):
````
- **EN**: Introduces function `remove_no_ops`, function `fake_tensors_eq`, function `is_mutated`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`remove_no_ops`、函数`fake_tensors_eq`、函数`is_mutated`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 113-140 / 第 113-140 行
````python
                    if arg is n:
                        schema_arg = user.target._schema.arguments[i]
                        if schema_arg.alias_info and schema_arg.alias_info.is_write:
                            return True
            return False

        def replace_no_op(node, replace_input_index):
            replacement = node.args[replace_input_index]

            # https://github.com/pytorch/pytorch/issues/86128 causes
            # non-Tensor inputs even for ops with only Tensor inputs.
            # TODO - decompose/type promote to avoid this
            if not all(isinstance(arg, torch.fx.Node) for arg in node.args):
                return

            # https://github.com/pytorch/pytorch/issues/174187
            # Don't replace if the replacement value is mutated in-place.
            # The original node acts as an implicit copy; removing it would
            # cause users to observe the post-mutation value instead.
            if is_mutated(replacement):
                return

            if not fake_tensors_eq(node.meta["val"], replacement.meta["val"]):
                if fake_tensors_eq(
                    node.meta["val"],
                    replacement.meta["val"],
                    ("shape", "device"),
                ):
````
- **EN**: Introduces function `replace_no_op`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`replace_no_op`。保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-168 / 第 141-168 行
````python
                    with graph.inserting_after(node):
                        replacement = graph.call_function(
                            torch.ops.prims.convert_element_type.default,
                            args=(replacement, node.meta["val"].dtype),
                        )
                else:
                    return

            node.replace_all_uses_with(replacement)
            replacement.meta.update(node.meta)
            graph.erase_node(node)

        for node in graph.find_nodes(op="call_function", target=aten.add.Tensor):
            # TODO handle Tensor-Scalar adds, it's a different schema
            if len(node.args) == 2:
                if (
                    not any(e in zeros for e in node.args)
                    or node.kwargs.get("alpha", 1) != 1
                ):
                    continue

                replace_index = 1 if node.args[0] in zeros else 0
                replace_no_op(node, replace_index)

        for node in graph.find_nodes(op="call_function", target=aten.sub.Tensor):
            if len(node.args) == 2:
                if node.args[1] not in zeros or node.kwargs.get("alpha", 1) != 1:
                    continue
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `replacement`, `args`, `else`, and `replace_index`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `replacement`、`args`、`else`、`replace_index` 等值。

### Lines 169-196 / 第 169-196 行
````python

                replace_no_op(node, 0)

        for node in graph.find_nodes(op="call_function", target=aten.mul.Tensor):
            if len(node.args) == 2:
                if not any(e in ones for e in node.args):
                    continue

                replace_input_index = 1 if node.args[0] in ones else 0
                replace_no_op(node, replace_input_index)

        for node in graph.find_nodes(op="call_function", target=aten.div.Tensor):
            if len(node.args) == 2 and node.args[1] in ones:
                replace_no_op(node, 0)

        # meta tensors returned from the graph have no data and can be replaced with empty_strided
        for output_node in graph.find_nodes(op="output"):
            had_meta_return = False

            def visit(n):
                nonlocal had_meta_return
                val = n.meta.get("val")
                if isinstance(val, torch.Tensor) and val.device.type == "meta":
                    with graph.inserting_before(output_node):
                        n.replace_all_uses_with(
                            graph.call_function(
                                torch.ops.aten.empty_strided.default,
                                args=(val.size(), val.stride()),
````
- **EN**: Introduces function `visit`. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `replace_input_index`, `had_meta_return`, `val`, and `args`.
- **CN**: 这里定义了函数`visit`。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `replace_input_index`、`had_meta_return`、`val`、`args` 等值。

### Lines 197-224 / 第 197-224 行
````python
                                kwargs={"dtype": val.dtype, "device": val.device},
                            )
                        )
                    had_meta_return = True

            torch.fx.map_arg(output_node.args, visit)
            if had_meta_return:
                graph.eliminate_dead_code()


def remove_redundant_views(gm: torch.fx.GraphModule):
    """
    Removes redundant views by reusing existing ones.
    """
    with torch.utils._python_dispatch._disable_current_modes():
        # A dictionary mapping a tensor to all aliased views.
        views: dict[torch.fx.Node, dict[torch.dtype, torch.fx.Node]] = {}
        graph = gm.graph

        for node in graph.find_nodes(
            op="call_function", target=torch.ops.aten.view.dtype
        ):
            src = node.args[0]
            to_type = node.args[1]
            existing_views = views.get(src)
            is_needed = True

            if existing_views:
````
- **EN**: Introduces function `remove_redundant_views`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`remove_redundant_views`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 225-252 / 第 225-252 行
````python
                # Replace the view with the an existing view if available.
                alias = existing_views.get(to_type)
                if alias:
                    is_needed = False
                    node.replace_all_uses_with(alias)
                    alias.meta.update(node.meta)
                    graph.erase_node(node)
            else:
                from_type = src.meta["val"].dtype
                existing_views = {from_type: src}
                views[src] = existing_views

            if is_needed:
                # Save the new alias but do not replace existing one.
                existing_views.setdefault(to_type, node)
                views[node] = existing_views

        # Clean up unused views.
        while True:
            unused_views = [alias for alias in views if not alias.users]
            if len(unused_views) == 0:
                break
            for unused in unused_views:
                views.pop(unused)
                graph.erase_node(unused)


class UniformValueConstantFolder(ConstantFolder):
````
- **EN**: Introduces class `UniformValueConstantFolder`. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `alias`, `is_needed`, `else`, `from_type`, `existing_views`, and `unused_views`.
- **CN**: 这里定义了类`UniformValueConstantFolder`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `alias`、`is_needed`、`else`、`from_type`、`existing_views`、`unused_views` 等值。

### Lines 253-280 / 第 253-280 行
````python
    """
    Runs constant folding and replaces tensors that have a uniform value
    with a tensor constructor call: aten.full([shape], value, ...)
    """

    def __init__(self, gm, skip_constructors=False) -> None:
        super().__init__(gm, skip_constructors)
        self.node_storages_ptrs: dict[torch.fx.Node, int] = {}
        self.constant_data_ptrs: dict[torch.fx.Node, StorageWeakRef] = {}
        # we may constant fold a tensor which in the graph has a sym size
        # see: [constant folding refining of symints]
        self.node_replacements_shapes: dict[torch.fx.Node, list[int]] = {}

        # initialize symint -> node mapping so that we can
        # use symint nodes in full constructors
        self.symint_nodes = _SymHashingDict()
        for n in self.module.graph.nodes:  # type: ignore[union-attr]
            if "val" in n.meta and isinstance(n.meta["val"], torch.SymInt):
                if n.meta["val"] not in self.symint_nodes:
                    self.symint_nodes[n.meta["val"]] = n

        # reference from torch/_funtorch/partitioners.py:get_default_op_list
        self.view_op_packets = [
            aten.squeeze,
            aten.unsqueeze,
            aten.alias,
            aten.view,
            aten.slice,
````
- **EN**: Introduces function `__init__`. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`__init__`。包含分支、循环或上下文管理等控制流。

### Lines 281-308 / 第 281-308 行
````python
            aten.t,
            prims.broadcast_in_dim,
            aten.expand,
            aten.as_strided,
            aten.permute,
        ]

        self.indexing_op_packets = OrderedSet(
            [
                aten.slice,
            ]
        )

        self._add_peephole_patterns()

    def _add_peephole_patterns(self) -> None:
        """
        Add peephole patterns for nodes where we can infer constant value even if some inputs
        of the node are unknown.
        """
        for op in itertools.chain(
            self.module.graph.find_nodes(  # type: ignore[operator, union-attr]
                op="call_function", target=torch.ops.aten.mul.Tensor
            ),
            self.module.graph.find_nodes(  # type: ignore[operator, union-attr]
                op="call_function", target=torch.ops.aten.mul.Scalar
            ),
        ):
````
- **EN**: Introduces function `_add_peephole_patterns`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `op`.
- **CN**: 这里定义了函数`_add_peephole_patterns`。包含分支、循环或上下文管理等控制流。初始化或更新了 `op` 等值。

### Lines 309-336 / 第 309-336 行
````python
            tensor_val = op.meta.get("val", None)
            if not isinstance(tensor_val, torch.Tensor):
                continue

            def is_zero_int(arg: Any) -> bool:
                return isinstance(arg, int) and arg == 0

            if not any(is_zero_int(a) for a in op.args):
                continue

            t = torch.full(
                [1],  # shape
                0,  # value
                dtype=tensor_val.dtype,
                device=tensor_val.device,
                pin_memory=False,
            )
            self.add_node_replacement(op, t)

    def _support_dynamic_shape(self):
        return True

    def insertable_tensor_check(self, t: torch.Tensor) -> bool:
        return True

    def add_node_replacement(self, node: torch.fx.Node, tensor: torch.Tensor) -> None:
        self.node_replacements[node] = tensor.flatten()[0].item()
        self.node_replacements_shapes[node] = node.meta["val"].shape
````
- **EN**: Introduces function `is_zero_int`, function `_support_dynamic_shape`, function `insertable_tensor_check`, function `add_node_replacement`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`is_zero_int`、函数`_support_dynamic_shape`、函数`insertable_tensor_check`、函数`add_node_replacement`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 337-364 / 第 337-364 行
````python
        self.constant_data_ptrs[node] = StorageWeakRef(tensor.untyped_storage())

    def insert_placerholder_values(self, env: dict[torch.fx.Node, Any]) -> None:
        for n in self.module.graph.find_nodes(op="placeholder"):  # type: ignore[operator, union-attr]
            if "val" in n.meta and isinstance(n.meta["val"], torch.SymInt):
                env[n] = n.meta["val"]
            else:
                env[n] = self.unknown_value

    def _deduce_value(self, node: torch.fx.Node):
        # deduce value for full-like nodes
        # 1. for constructors, substitute value is a tensor of size [1]
        # 2. for view ops/indexing, substitute value is the same as the input
        # 3. for pointwise ops, run node to get the substitute value
        # 4. deal with some special ops
        # otherwise, stop deduce value and return unknown value

        # TODO: cat, more indexing
        # TODO - do on cpu to avoid syncs

        # single-elem attrs
        if node.op == "get_attr" or (
            node.op == "call_function"
            and node.target is torch.ops.aten.lift_fresh_copy.default
        ):
            out = super(ConstantFolder, self).run_node(node)
            if isinstance(out, torch.Tensor) and out.numel() == 1:
                return out
````
- **EN**: Introduces function `insert_placerholder_values`, function `_deduce_value`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`insert_placerholder_values`、函数`_deduce_value`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 365-392 / 第 365-392 行
````python

        # handle device_put op
        if node.target == prims.device_put.default:
            return super(ConstantFolder, self).run_node(node)

        # constructors ops
        if (
            node.op == "call_function"
            and node.target is aten.full.default
            and len(node.args) == 2
        ):
            args, kwargs = self.fetch_args_kwargs_from_env(node)
            value = args[1]
            # Don't specialize symbolic value.
            if not isinstance(value, (torch.SymInt, torch.SymFloat, torch.SymBool)):
                new_args = [[1], value]
                return aten.full.default(*new_args, **node.kwargs)

        # handle before view ops because this changes value
        if node.target is aten.view.dtype:
            (input_tensor, output_dtype), kwargs = self.fetch_args_kwargs_from_env(node)
            # view.dtype with different element sizes changes element count
            # (e.g., complex64 [1+0j] viewed as float32 becomes [1.0, 0.0]),
            # making uniform values non-uniform. Also crashes on 0-d tensors.
            if input_tensor.element_size() != output_dtype.itemsize:
                return self.unknown_value
            return super(ConstantFolder, self).run_node(node)

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `value`, and `new_args`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `value`、`new_args` 等值。

### Lines 393-420 / 第 393-420 行
````python
        # view ops, return input tensor, the first argument
        if hasattr(node.target, "overloadpacket") and (
            node.target.overloadpacket in self.view_op_packets
            or node.target.overloadpacket in self.indexing_op_packets
        ):
            assert isinstance(node.args[0], torch.fx.Node)
            return self.env[node.args[0]]

        # we don't want to return unknown value for symints so that we can
        # still constant fold through their use in constructors or views
        # if we see them in a pointwise node (e.g., tensor * symint)
        # we will bail
        if "val" in node.meta and isinstance(node.meta["val"], torch.SymInt):
            return node.meta["val"]

        # pointwise ops
        if isinstance(node.target, torch._ops.OpOverload) and (
            torch.Tag.pointwise in node.target.tags
            or node.target is torch.ops.aten.scalar_tensor.default
        ):
            args, kwargs = self.fetch_args_kwargs_from_env(node)
            flattened_inputs = pytree.arg_tree_leaves(*args, **kwargs)

            if any(isinstance(inp, torch.SymInt) for inp in flattened_inputs):
                return self.unknown_value

            # we run the ops with dim 1, so remove memory_format to avoid error
            kwargs = dict(kwargs)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `flattened_inputs`, and `kwargs`. This range continues the implementation of function `UniformValueConstantFolder._deduce_value`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `flattened_inputs`、`kwargs` 等值。这一段延续了函数`UniformValueConstantFolder._deduce_value` 的具体实现。

### Lines 421-448 / 第 421-448 行
````python
            kwargs.pop("memory_format", None)

            return node.target(*args, **kwargs)

        return self.unknown_value


def _has_self_referential_shape(
    shapes: list[int | torch.fx.Node], node: torch.fx.Node
) -> bool:
    """
    Check if any shape in `shapes` depends on `node`.

    This is used to detect cycles when constant_fold_uniform_value creates a
    replacement full() node whose shape includes a sym_size computed from the
    original tensor being replaced.

    Checks direct args only - shape nodes typically come from sym_size(tensor, dim)
    where tensor is a direct arg.
    """
    for shape_node in shapes:
        if isinstance(shape_node, torch.fx.Node):
            if node in shape_node.args:
                return True
    return False


def constant_fold_uniform_value(gm: torch.fx.GraphModule):
````
- **EN**: Introduces function `_has_self_referential_shape`, function `constant_fold_uniform_value`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `shapes`.
- **CN**: 这里定义了函数`_has_self_referential_shape`、函数`constant_fold_uniform_value`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `shapes` 等值。

### Lines 449-476 / 第 449-476 行
````python
    """Runs constant folding and replaces constants which can be constructed with a single `full` call. Calls into remove_no_ops."""
    with torch.utils._python_dispatch._disable_current_modes():
        aten = torch.ops.aten

        # Constant folding can leak memory, especially with repeated compilation, so we are only going to
        # remove constants which can be replaced with a constructor.
        cf = UniformValueConstantFolder(gm)
        cf.run()

        node_replacements = cf.node_replacements

        # note: [constant folding refining of symints]
        # constant folding will partially evaluate a graph such that values which have dependencies which
        # are entirely known at compile time may also become compile time constants. in some cases,
        # this will include symints which we had not yet previously deduced are guaranteed a
        # constant value and is then deduced in constant folding. an example is:
        # unbacked_symint_eq_11 = torch.full((), 11).item()
        # torch.full((unbacked_symint_eq_11,), 0)
        node_replacements_shapes = cf.node_replacements_shapes

        graph = gm.graph

        zeros = OrderedSet[Any]()
        ones = OrderedSet[Any]()

        # Got failures in `test_is_set_to_cuda` if we change aliasing on constants,
        # so just constant-ify if a Tensor is unaliased
        constant_data_ptr_count: typing.Counter[StorageWeakRef] = Counter()
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `aten`, `cf`, `node_replacements`, `node_replacements_shapes`, `graph`, `zeros`, and `...+2`. This range continues the implementation of function `constant_fold_uniform_value`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `aten`、`cf`、`node_replacements`、`node_replacements_shapes`、`graph`、`zeros`、`另有2项` 等值。这一段延续了函数`constant_fold_uniform_value` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python

        for node in cf.node_replacements:
            constant_data_ptr_count[cf.constant_data_ptrs[node]] += 1

        for node, value in node_replacements.items():
            # we dont have a functional way right now of instantiating a non-contiguous tensor with full/zeros/ones right now
            # hasn't shown up to be important yet
            if "val" not in node.meta:
                # This can only happen in AOTI
                continue

            fake_tensor = node.meta["val"]
            if not fake_tensor.is_contiguous(memory_format=torch.contiguous_format):
                continue

            # TODO - not sure about lossy uint->python value->uint conversions
            if fake_tensor.dtype in (
                torch.uint8,
                torch.uint16,
                torch.uint32,
                torch.uint64,
            ):
                continue

            if constant_data_ptr_count[cf.constant_data_ptrs[node]] > 1:
                continue

            with graph.inserting_after(node):
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fake_tensor`. This range continues the implementation of function `constant_fold_uniform_value`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `fake_tensor` 等值。这一段延续了函数`constant_fold_uniform_value` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python
                # the conversion from tensor and back to value can be lossy, just use the original full ctor value
                if (
                    node.op == "call_function"
                    and node.target is aten.full.default
                    and len(node.args) == 2
                ):
                    value = node.args[1]

                # refines symints, see [constant folding refining of symints] above
                for runtime_size, compile_time_size in zip(
                    node_replacements_shapes[node], fake_tensor.shape
                ):
                    torch._check(runtime_size == compile_time_size)

                # replace SymInt as Node before creating a new full node
                # e.g. (1, s0) -> (1, arg0_1)
                node_shape = node_replacements_shapes[node]
                if not all(
                    not isinstance(s, torch.SymInt) or s in cf.symint_nodes
                    for s in node_shape
                ):
                    continue

                shapes = [
                    cf.symint_nodes[s] if isinstance(s, torch.SymInt) else s
                    for s in node_replacements_shapes[node]
                ]

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `value`, `node_shape`, and `shapes`. This range continues the implementation of function `constant_fold_uniform_value`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `value`、`node_shape`、`shapes` 等值。这一段延续了函数`constant_fold_uniform_value` 的具体实现。

### Lines 533-560 / 第 533-560 行
````python
                # Check if any shape depends on a symint that was computed from
                # the node being replaced - this would create a cycle
                if _has_self_referential_shape(shapes, node):
                    continue

                # zeros and ones just get traced into full, so we insert those
                new_node = graph.call_function(
                    aten.full.default,
                    args=(shapes, value),
                    kwargs={
                        "dtype": fake_tensor.dtype,
                        "layout": torch.strided,
                        "device": fake_tensor.device,
                        "pin_memory": node.kwargs.get("pin_memory", False),
                    },
                )

                new_node.meta.update(node.meta)
                node.replace_all_uses_with(new_node)
                graph.erase_node(node)

                if value == 0:
                    zeros.add(new_node)
                elif value == 1:
                    ones.add(new_node)

        remove_no_ops(gm, zeros, ones)
        remove_redundant_views(gm)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_node`, `args`, and `kwargs`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_node`、`args`、`kwargs` 等值。

### Lines 561-588 / 第 561-588 行
````python


def canonicalize_quant_mapping(gm: torch.fx.GraphModule):
    """


    torch.ops.higher_order.invoke_quant_packed(repeated_subgraph0, 'quant_invoke_0_0', (arg0_1, arg1_1));
    ->
    torch.ops.higher_order.invoke_quant(repeated_subgraph0, arg0_1, arg1_1, scheme = 'nf4');
    """
    graph = gm.graph
    invoke_quant_invocations = graph.find_nodes(
        op="call_function", target=torch.ops.higher_order.invoke_quant_packed
    )
    for invoke_quant in invoke_quant_invocations:
        kwargs = dict(invoke_quant.kwargs)

        quant_options_node = kwargs.pop("quant_options", None)
        if quant_options_node is not None:
            assert isinstance(quant_options_node, torch.fx.Node)
            quant_options = torch._higher_order_ops.InvokeQuant(
                *invoke_quant.kwargs["quant_options"].args,
                **invoke_quant.kwargs["quant_options"].kwargs,
            )
        else:
            quant_options = torch._higher_order_ops.InvokeQuant()

        subgraph, *args = invoke_quant.args
````
- **EN**: Introduces function `canonicalize_quant_mapping`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`canonicalize_quant_mapping`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 589-616 / 第 589-616 行
````python
        with gm.graph.inserting_before(invoke_quant):
            invoke_quant_replacement = graph.call_function(
                torch._higher_order_ops.invoke_quant,
                (subgraph, *args),
                # pyrefly: ignore [bad-argument-type]
                kwargs,
            )
            invoke_quant_replacement.meta.update(subgraph.meta)
            invoke_quant_replacement.meta["quant_options"] = quant_options

            invoke_quant.replace_all_uses_with(invoke_quant_replacement)
            graph.erase_node(invoke_quant)

            if quant_options_node and len(quant_options_node.users) == 0:
                graph.erase_node(quant_options_node)

            first_user = next(iter(invoke_quant_replacement.users))

            if (
                len(invoke_quant_replacement.users) == 1
                and len(subgraph.users) == 1
                and first_user.target is operator.getitem
                and first_user.args[1] == 0
            ):
                subgraph_graph = getattr(gm, subgraph.target)
                output_node = torch._inductor.utils.output_node(subgraph_graph)
                assert (
                    isinstance(output_node.args[0], (list, tuple))
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `invoke_quant_replacement`, `first_user`, `subgraph_graph`, and `output_node`. This range continues the implementation of function `canonicalize_quant_mapping`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `invoke_quant_replacement`、`first_user`、`subgraph_graph`、`output_node` 等值。这一段延续了函数`canonicalize_quant_mapping` 的具体实现。

### Lines 617-644 / 第 617-644 行
````python
                    and len(output_node.args[0]) == 1
                )

                unpacked_output = output_node.args[0][0]
                # pyrefly: ignore [bad-argument-type]
                output_node.args = (unpacked_output,)
                if "val" in output_node.meta:
                    output_node.meta["val"] = output_node.meta["val"][0]
                subgraph_graph.recompile()

                invoke_quant_replacement.meta.update(first_user.meta)
                first_user.replace_all_uses_with(invoke_quant_replacement)
                graph.erase_node(first_user)


def canonicalize_aten_ir_passes(gm: torch.fx.GraphModule):
    """
    Canonicalization passes that will run immediately after aot autograd
    tracing. Thsis must be run before all other graph passes.
    """
    canonicalize_quant_mapping(gm)


def joint_graph_passes(
    graph: torch.fx.GraphModule,
    input_device: torch.device | None = None,
):
    """
````
- **EN**: Introduces function `canonicalize_aten_ir_passes`, function `joint_graph_passes`. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`canonicalize_aten_ir_passes`、函数`joint_graph_passes`。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 645-672 / 第 645-672 行
````python
    Run FX transformations on the joint forwards+backwards graph.
    """
    GraphTransformObserver = functools.partial(
        torch.fx.passes.graph_transform_observer.GraphTransformObserver,
        subsystem="joint_graph_passes",
    )

    lazy_init(input_device)
    count = 0

    # must occur before other passes
    canonicalize_aten_ir_passes(graph)

    if config.joint_custom_pre_pass is not None:
        GraphTransformObserver(graph, "joint_custom_pre_pass").apply_graph_pass(
            config.joint_custom_pre_pass
        )
        count += 1

    from .post_grad import remove_noop_ops

    GraphTransformObserver(graph, "remove_noop_ops").apply_graph_pass(remove_noop_ops)

    if config.joint_graph_constant_folding:
        GraphTransformObserver(graph, "constant_fold_uniform_value").apply_gm_pass(
            constant_fold_uniform_value
        )

````
- **EN**: Imports dependencies such as `.post_grad` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `GraphTransformObserver`, `subsystem`, and `count`. This range continues the implementation of function `joint_graph_passes`.
- **CN**: 这里导入了 `.post_grad` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `GraphTransformObserver`、`subsystem`、`count` 等值。这一段延续了函数`joint_graph_passes` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
    if config.pattern_matcher:
        count += early_patterns.apply(graph.graph)

    # Make sure AutoChunker happens before pad_mm so we don't need
    # to handle padding when searching for chunking patterns.
    if config.auto_chunker.enable:
        from .auto_chunker import CantChunk, chunk

        try:
            graph = chunk(graph)
        except CantChunk:
            auto_chunker_log = torch._logging.getArtifactLogger(
                __name__, "auto_chunker"
            )
            auto_chunker_log.debug("AutoChunker fail.", exc_info=True)

    if config.pattern_matcher:
        for i, patterns in enumerate(pass_patterns):
            maybe_count = GraphTransformObserver(
                graph, f"pass_pattern_{i}"
            ).apply_graph_pass(patterns.apply)
            count += maybe_count if maybe_count is not None else 0

    if not config.fallback_random:
        # not trying into the bisector because decomps may have already affected rng reproducibility
        # we'll instead explicitly turn off the config
        count += replace_random_passes(graph)

````
- **EN**: Imports dependencies such as `.auto_chunker` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, `graph`, `auto_chunker_log`, and `maybe_count`. This range continues the implementation of function `joint_graph_passes`.
- **CN**: 这里导入了 `.auto_chunker` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`graph`、`auto_chunker_log`、`maybe_count` 等值。这一段延续了函数`joint_graph_passes` 的具体实现。

### Lines 701-728 / 第 701-728 行
````python
    if config.joint_custom_post_pass is not None:
        GraphTransformObserver(graph, "joint_custom_post_pass").apply_graph_pass(
            config.joint_custom_post_pass
        )
        count += 1

    if count:
        stable_topological_sort(graph.graph)
        graph.graph.lint()
        graph.recompile()
    return graph


@register_graph_pattern(
    CallFunction(
        torch.ops.prims.iota.default,
        KeywordArg("length"),
        start=KeywordArg("start"),
        step=KeywordArg("step"),
        dtype=KeywordArg("dtype"),
        device=KeywordArg("device"),
        requires_grad=KeywordArg("requires_grad"),
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=patterns,
)
def fix_iota_device(match: Match, length, start, step, dtype, device, requires_grad):
    """
````
- **EN**: Introduces function `fix_iota_device`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`fix_iota_device`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 729-756 / 第 729-756 行
````python
    Eager supports:

        aten.index(cuda_tensor, torch.arange(..., device="cpu"))

    But this results in an implicit host-device-copy and breaks cudagraphs.
    Rewrite the arange to use CUDA.
    """
    (node,) = match.nodes
    user_devices = OrderedSet[torch.device]()
    for user in node.users:
        if (
            user.op == "call_function"
            and user.target in (aten.index.Tensor, aten.index_put.default)
            and hasattr(user.meta.get("val"), "device")
        ):
            user_devices.add(user.meta["val"].device)  # type: ignore[union-attr]
        else:
            return  # bail out

    if len(user_devices) == 1 and "val" in node.meta:
        (user_device,) = user_devices
        if device.type != user_device.type:
            repl = match.graph.call_function(
                torch.ops.prims.iota.default,
                (length,),
                {
                    "start": start,
                    "step": step,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `user_devices`, `else`, and `repl`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `user_devices`、`else`、`repl` 等值。

### Lines 757-784 / 第 757-784 行
````python
                    "dtype": dtype,
                    "device": user_device,
                    "requires_grad": requires_grad,
                },
            )
            repl.meta.update(node.meta)
            repl.meta["val"] = repl.meta["val"].to(user_device)
            node.replace_all_uses_with(repl)
            match.erase_nodes()


@register_graph_pattern(
    CallFunction(
        torch.ops.prims.convert_element_type.default,
        CallFunction(
            torch.ops.prims.convert_element_type.default,
            KeywordArg("arg"),
            KeywordArg("dtype1"),
        ),
        KeywordArg("dtype2"),
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=patterns,
)
def pointless_convert(match: Match, arg, dtype1: torch.dtype, dtype2: torch.dtype):
    """Remove chain of dtype conversions often created by AMP"""
    graph = match.graph
    node = match.output_node()
````
- **EN**: Introduces function `pointless_convert`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`pointless_convert`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 785-812 / 第 785-812 行
````python
    allowed = torch.float16, torch.bfloat16, torch.float32, torch.float64
    arg_val = arg.meta.get("val", None)
    if not isinstance(arg_val, torch.Tensor):
        return

    if arg_val.dtype in allowed and dtype1 in allowed and dtype2 in allowed:
        if config.emulate_precision_casts and not _is_lossless_fp_widening_cast(
            arg_val.dtype, dtype1
        ):
            return
        repl = graph.call_function(
            torch.ops.prims.convert_element_type.default, (arg, dtype2)
        )
        repl.meta.update(node.meta)
        node.replace_all_uses_with(repl)
        match.erase_nodes()


def definitely_equal(
    old_sizes: Sequence[torch.SymInt | int],
    new_sizes: Sequence[torch.SymInt | torch.fx.Node | int],
) -> bool:
    """
    Leverage guard_or_true/false to compare if two lists of int/symint are equal.
    Useful to compare sizes, strides etc.

    Can handle -1 in new_sizes which happens in the size arguments of a
    view op. old_sizes is supposed to be the tensor shape and should not
````
- **EN**: Introduces function `definitely_equal`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`definitely_equal`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 813-840 / 第 813-840 行
````python
    contain -1.

    new_sizes can contains fx.Node when dynamic shape is enabled. In that
    case new_sizes[i].meta['val'] contains the real torch.SymInt.
    """

    num_neg1 = 0

    if len(old_sizes) != len(new_sizes):
        return False

    for lhs_item, rhs_item in zip(old_sizes, new_sizes):
        if isinstance(rhs_item, torch.fx.Node):
            rhs_item = rhs_item.meta["val"]

        assert isinstance(lhs_item, (int, torch.SymInt)), type(lhs_item)
        assert isinstance(rhs_item, (int, torch.SymInt)), type(rhs_item)

        # It still makes sense to call guard_or_true/false since lhs_item
        # rhs_item are torch.SymInt rather than sympy expressions when
        # dynamic shape is enabled.
        if guard_or_false(lhs_item == rhs_item):
            continue

        if guard_or_true(rhs_item != -1):
            return False

        num_neg1 += 1
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_neg1`, and `rhs_item`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `num_neg1`、`rhs_item` 等值。

### Lines 841-868 / 第 841-868 行
````python

        if num_neg1 > 1:
            return False
    return True


@register_graph_pattern(
    CallFunction(torch.ops.aten.view.default, KeywordArg("arg"), KeywordArg("size")),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=early_patterns,
)
def pointless_view(match: Match, arg, size):
    """Remove no-op view"""
    node = match.output_node()
    arg_size = list(node.args[0].meta["val"].shape)  # type: ignore[union-attr]
    if definitely_equal(arg_size, size):
        node.replace_all_uses_with(node.args[0])  # type: ignore[arg-type]
        match.erase_nodes()


@register_graph_pattern(
    CallFunction(
        aten.view.default,
        CallFunction(aten.view.default, KeywordArg("arg"), KeywordArg("size1")),
        KeywordArg("size2"),
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=early_patterns,
````
- **EN**: Introduces function `pointless_view`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`pointless_view`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 869-896 / 第 869-896 行
````python
)
def pointless_view_pair(match: Match, arg, size1, size2):
    """
    Remove a pair of views that are pointless.
    """
    node = match.output_node()
    arg_size = list(arg.meta["val"].shape)
    if definitely_equal(arg_size, size2):
        node.replace_all_uses_with(arg)
        match.erase_nodes()
        counters["inductor"]["removed_pointless_view_pair"] += 1


@register_graph_pattern(
    CallFunction(
        aten.permute.default,
        CallFunction(aten.permute.default, KeywordArg("arg"), KeywordArg("perm1")),
        KeywordArg("perm2"),
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=early_patterns,
)
def pointless_permute_pair(match: Match, arg, perm1, perm2):
    rank = len(perm1)
    assert len(perm2) == rank

    for i in range(rank):
        if perm1[perm2[i]] != i:
````
- **EN**: Introduces function `pointless_view_pair`, function `pointless_permute_pair`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`pointless_view_pair`、函数`pointless_permute_pair`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 897-924 / 第 897-924 行
````python
            return  # bail out
    node = match.output_node()
    node.replace_all_uses_with(arg)
    match.erase_nodes()


@register_graph_pattern(
    CallFunction(
        aten.bmm,
        Arg(),
        Arg(),
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=patterns,
)
def bmm_to_mm(match: Match, mat1: torch.fx.Node, mat2: torch.fx.Node):
    """Convert bmm to mm when batch size is 1"""

    def repl(a, b):
        return torch.mm(a.squeeze(0), b.squeeze(0)).unsqueeze(0)

    if (
        check_device(mat1.meta["val"], mat2.meta["val"], get_gpu_type())
        and statically_known_true(mat1.meta["val"].shape[0] == 1)
        and statically_known_true(mat2.meta["val"].shape[0] == 1)
    ):
        # pyrefly: ignore [bad-argument-type]
        match.replace_by_example(repl, [mat1, mat2])
````
- **EN**: Introduces function `bmm_to_mm`, function `repl`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`bmm_to_mm`、函数`repl`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 925-952 / 第 925-952 行
````python


# When softmax is used with temperature or other scaling, we get the pattern
#
#   scale(x) - scale(x).amax(dim, keepdim=True)
#
# which is expected to be at most zero, but we may end up with numerical
# discrepancies # between the recomputed values of scale(x) inside and out
# of the reduction, # depending on compiler optimizations, e.g. use of fma
# instructions.
#
# Here we replace it with the mathematically equivalent,
#
#   scale(x - x.amax(dim, keepdim=True))
#
# which is more stable as we only compute the scaling once.
#
# NOTE: This pattern must come after fused attention matching!


def _partial_softmax_pattern(linear_func, reverse=False, to_dtype=False):
    # Allow matching inp * other and other * input
    if reverse:
        scaled = CallFunction(
            linear_func, KeywordArg("other"), KeywordArg("inp"), _users=MULTIPLE
        )
    else:
        scaled = CallFunction(
````
- **EN**: Introduces function `_partial_softmax_pattern`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scaled`, and `else`.
- **CN**: 这里定义了函数`_partial_softmax_pattern`。包含分支、循环或上下文管理等控制流。初始化或更新了 `scaled`、`else` 等值。

### Lines 953-980 / 第 953-980 行
````python
            linear_func, KeywordArg("inp"), KeywordArg("other"), _users=MULTIPLE
        )
    if to_dtype:
        scaled = CallFunction(
            prims.convert_element_type, scaled, KeywordArg("dtype"), _users=MULTIPLE
        )
    amax = CallFunction(
        aten.amax.default, scaled, KeywordArg("dim"), KeywordArg("keepdim")
    )
    return CallFunction(aten.sub.Tensor, scaled, amax)


def _other_is_broadcasted_in_dim(match):
    # Check that the scaling factor is constant across the reduction dim,
    # so scaling doesn't change which index corresponds to the maximum value
    other = match.kwargs["other"]
    if isinstance(other, (int, float)):
        return True

    inp = match.kwargs["inp"]
    if not all(isinstance(x, torch.fx.Node) for x in (inp, other)):
        return False

    inp_example = inp.meta["val"]
    other_example = other.meta["val"]
    if isinstance(other_example, (torch.SymInt, torch.SymFloat)):
        return True

````
- **EN**: Introduces function `_other_is_broadcasted_in_dim`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_other_is_broadcasted_in_dim`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 981-1008 / 第 981-1008 行
````python
    if not all(isinstance(x, torch.Tensor) for x in (inp_example, other_example)):
        return False

    inp_ndim = inp_example.ndim
    other_shape = other_example.shape
    if inp_ndim < len(other_shape):
        return False

    # Pad other_shape to the same ndim as inp
    other_shape = [1] * (inp_ndim - len(other_shape)) + list(other_shape)

    dim = match.kwargs["dim"]
    if isinstance(dim, int):
        dim = (dim,)

    if any(d >= len(other_shape) for d in dim):
        return False

    return all(statically_known_true(other_shape[d] == 1) for d in dim)


def mul_softmax_pattern(match: Match, *, inp, other, dim, keepdim, dtype=None):
    def repl(inp, other):
        if dtype is not None:
            inp = inp.to(dtype)

        sign: int | float | torch.Tensor
        if isinstance(other, (int, float, torch.SymInt, torch.SymFloat)):
````
- **EN**: Introduces function `mul_softmax_pattern`, function `repl`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`mul_softmax_pattern`、函数`repl`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1009-1036 / 第 1009-1036 行
````python
            sign = 1 if other >= 0 else -1
        else:
            one = torch.scalar_tensor(1, dtype=inp.dtype, device=inp.device)
            sign = torch.where(other >= 0, one, -one)

        inp = inp * sign
        max_ = torch.amax(inp, dim=dim, keepdim=keepdim)

        return (inp - max_) * (sign * other)

    # pyrefly: ignore [bad-argument-type]
    match.replace_by_example(repl, [inp, other])


for reverse, to_dtype in itertools.product((False, True), repeat=2):
    register_graph_pattern(
        _partial_softmax_pattern(aten.mul.Tensor, reverse=reverse, to_dtype=to_dtype),
        # pyrefly: ignore [bad-argument-type]
        pass_dict=pass_patterns[1],
        extra_check=_other_is_broadcasted_in_dim,
    )(mul_softmax_pattern)


def div_softmax_pattern(match: Match, *, inp, other, dim, keepdim, dtype=None):
    def repl(inp, other):
        if dtype is not None:
            inp = inp.to(dtype)

````
- **EN**: Introduces function `div_softmax_pattern`, function `repl`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`div_softmax_pattern`、函数`repl`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1037-1064 / 第 1037-1064 行
````python
        sign: int | float | torch.Tensor
        if isinstance(other, (int, float, torch.SymInt, torch.SymFloat)):
            sign = 1 if other >= 0 else -1
        else:
            one = torch.scalar_tensor(1, dtype=inp.dtype, device=inp.device)
            sign = torch.where(other >= 0, one, -one)

        inp = inp * sign
        max_ = torch.amax(inp, dim=dim, keepdim=keepdim)

        return (inp - max_) / (sign * other)

    # pyrefly: ignore [bad-argument-type]
    match.replace_by_example(repl, [inp, other])


for to_dtype in (False, True):
    register_graph_pattern(
        _partial_softmax_pattern(aten.div.Tensor, to_dtype=to_dtype),
        # pyrefly: ignore [bad-argument-type]
        pass_dict=pass_patterns[1],
        extra_check=_other_is_broadcasted_in_dim,
    )(div_softmax_pattern)


def scatter_upon_const_tensor_extra_check(m):
    if not config.optimize_scatter_upon_const_tensor:
        return False
````
- **EN**: Introduces function `scatter_upon_const_tensor_extra_check`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`scatter_upon_const_tensor_extra_check`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1065-1092 / 第 1065-1092 行
````python
    full_shape = m.kwargs["shape"]
    selector = m.kwargs["selector"]
    dim = m.kwargs["dim"]
    if dim < 0:
        dim += len(full_shape)

    selector_ft = selector.meta["val"]
    assert selector_ft.dim() == len(full_shape)

    for idx, select_sz, full_sz in zip(
        itertools.count(), selector_ft.shape, full_shape
    ):
        if idx == dim:
            continue

        # TODO: the pattern can be updated to support the case that index tensor
        # is shorter. But that will need a more complex condition expression
        # especially for multi-dimensional tensors.
        # Skip it for now.
        if isinstance(full_sz, torch.fx.Node):
            full_sz = full_sz.meta["val"]
        if select_sz < full_sz:
            return False

    # Actually we can support small size larger than 1. It would be a bit
    # tedious. E.g., we load all the index values (not many) and compare
    # them with the position in tensor to decide what value to return.
    return selector_ft.size(dim) == 1
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `full_shape`, `selector`, `dim`, `selector_ft`, and `full_sz`. This range continues the implementation of function `scatter_upon_const_tensor_extra_check`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `full_shape`、`selector`、`dim`、`selector_ft`、`full_sz` 等值。这一段延续了函数`scatter_upon_const_tensor_extra_check` 的具体实现。

### Lines 1093-1120 / 第 1093-1120 行
````python


@register_graph_pattern(
    CallFunction(
        aten.scatter.value,
        CallFunction(
            aten.full,
            KeywordArg("shape"),
            KeywordArg("background_val"),
            dtype=KeywordArg("dtype"),
        ),
        KeywordArg("dim"),
        KeywordArg("selector"),
        KeywordArg("val"),  # scalar value
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=patterns,
    extra_check=scatter_upon_const_tensor_extra_check,
)
def scatter_upon_const_tensor(
    match: Match, shape, background_val, dtype, dim, selector, val
):
    """
    Match the pattern of full+scatter into a pointwise operation in joint graph.

    TODO: Right now the scatter value must be a scalar. But we could support it
    when it is a tensor as well.
    """
````
- **EN**: Introduces function `scatter_upon_const_tensor`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`scatter_upon_const_tensor`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1121-1148 / 第 1121-1148 行
````python
    from torch._inductor import metrics

    # pyrefly: ignore  # bad-assignment
    metrics.num_matches_for_scatter_upon_const_tensor += 1

    # Create a replacement that uses torch.where for the pointwise operation
    def repl_fn(shape, background_val, dim, selector, val):
        # Create a tensor of indices for the scatter dimension
        length = shape[dim]
        indices = torch.arange(length, device=selector.device, dtype=torch.int64)

        # Reshape indices to have size 'length' at dim, then broadcast
        view_shape = [1] * len(shape)
        view_shape[dim] = length
        indices_view = indices.view(*view_shape)

        # Broadcast selector to match full tensor shape
        selector_expanded = selector.expand(shape)

        # Create a mask for where to scatter
        mask = selector_expanded == indices_view

        # Use torch.where to implement the scatter pointwise operation
        return torch.where(mask, val, background_val)

    # replace the scatter operation with pointwise equivalent
    # pyrefly: ignore [bad-argument-type]
    match.replace_by_example(repl_fn, [shape, background_val, dim, selector, val])
````
- **EN**: Imports dependencies such as `torch._inductor` for the logic in this range. Introduces function `repl_fn`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `torch._inductor` 等依赖，为后续逻辑提供基础能力。这里定义了函数`repl_fn`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `UniformValueConstantFolder`  
  **CN**: 主要类：`UniformValueConstantFolder`
- **EN**: Primary functions: `_is_lossless_fp_widening_cast`, `lazy_init`, `remove_no_ops`, `remove_redundant_views`, `_has_self_referential_shape`, `constant_fold_uniform_value`, and `...+16`  
  **CN**: 主要函数：`_is_lossless_fp_widening_cast`、`lazy_init`、`remove_no_ops`、`remove_redundant_views`、`_has_self_referential_shape`、`constant_fold_uniform_value`、`另有16项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `itertools`, `logging`, `operator`, `typing`, `collections`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._guards`, `torch.utils._pytree`, `torch._dynamo.utils`, `torch._inductor.constant_folding`, `torch._inductor.fx_passes.dedupe_symint_uses`, `torch._inductor.utils`, `torch.fx.experimental.symbolic_shapes`, `torch.multiprocessing.reductions`, `torch.utils._ordered_set`, `..`, `..pattern_matcher`, `.decompose_mem_bound_mm`, `.replace_random`, `.fuse_attention`, `.misc_patterns`, `.pad_mm`, `.post_grad`, `torch._inductor`, `.auto_chunker`
