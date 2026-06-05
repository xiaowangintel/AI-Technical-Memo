# constant_folding.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/constant_folding.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `ConstantFolder`. It exposes functions such as `add_dont_constant_fold`, `clear_dont_constant_fold`, `replace_node_with_constant`, `is_const_source`, `constant_fold`, `constant_graph_tag`, and `...+1`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `ConstantFolder` 等类。同时提供 `add_dont_constant_fold`、`clear_dont_constant_fold`、`replace_node_with_constant`、`is_const_source`、`constant_fold`、`constant_graph_tag`、`另有1项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import collections
import typing_extensions
from collections.abc import Callable
from typing import Any

import torch
import torch.utils._pytree as pytree
from torch._inductor.freezing_utils import maybe_set_is_frozen_param
from torch.utils._ordered_set import OrderedSet


aten = torch.ops.aten

# We would like to split modules into two subgraphs for runtime weight updates to work correctly.
# The use case and more information could be found at:
# https://docs.google.com/document/d/1inZC-8KarJ6gKB7G9egmYLx1V_dKX_apxon0w4zPC0Q/edit?usp=sharing
META_TAG = "MODULE_TYPE"
MODULE_TAG = "_MAIN_MODULE"
CONST_MODULE_TAG = "_CONST_MODULE"

````
- **EN**: Imports dependencies such as `collections`, `typing_extensions`, `collections.abc`, `typing`, `torch`, `torch.utils._pytree`, and `...+2` for the logic in this range. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Initializes or updates values such as `aten`, `META_TAG`, `MODULE_TAG`, and `CONST_MODULE_TAG`.
- **CN**: 这里导入了 `collections`、`typing_extensions`、`collections.abc`、`typing`、`torch`、`torch.utils._pytree`、`另有2项` 等依赖，为后续逻辑提供基础能力。保留了指向设计说明、规范或厂商数据手册的注释引用。初始化或更新了 `aten`、`META_TAG`、`MODULE_TAG`、`CONST_MODULE_TAG` 等值。

### Lines 21-40 / 第 21-40 行
````python
_dont_constant_fold: list[torch.fx.node.Target] = []


def add_dont_constant_fold(op: torch.fx.node.Target) -> None:
    global _dont_constant_fold
    _dont_constant_fold.append(op)


def clear_dont_constant_fold() -> None:
    global _dont_constant_fold
    _dont_constant_fold.clear()


def replace_node_with_constant(
    gm: torch.fx.GraphModule,
    node: torch.fx.Node,
    constant: torch.Tensor | None = None,
    name: str | None = None,
) -> None:
    g = gm.graph
````
- **EN**: Introduces function `add_dont_constant_fold`, function `clear_dont_constant_fold`, function `replace_node_with_constant`. Initializes or updates values such as `_dont_constant_fold`, `gm`, `node`, `constant`, `name`, and `g`.
- **CN**: 这里定义了函数`add_dont_constant_fold`、函数`clear_dont_constant_fold`、函数`replace_node_with_constant`。初始化或更新了 `_dont_constant_fold`、`gm`、`node`、`constant`、`name`、`g` 等值。

### Lines 41-60 / 第 41-60 行
````python

    if name:
        qualname = name
    else:
        if not hasattr(gm, "_frozen_param_count"):
            gm._frozen_param_count = 0  # type: ignore[assignment]
        i = gm._frozen_param_count
        # pyrefly: ignore [bad-assignment]
        while True:
            qualname = f"_frozen_param{i}"
            if not hasattr(gm, qualname):
                break
            i += 1  # type: ignore[assignment, operator]

        gm._frozen_param_count = i + 1  # type: ignore[assignment, operator]

    with g.inserting_before(node):
        if constant is not None:
            new_input_node = g.create_node("get_attr", qualname, (), {})
        else:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `qualname`, `else`, `i`, and `new_input_node`. This range continues the implementation of function `replace_node_with_constant`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `qualname`、`else`、`i`、`new_input_node` 等值。这一段延续了函数`replace_node_with_constant` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python
            # this is the case for lifted constants
            new_input_node = g.create_node("placeholder", qualname, (), {})
        node.replace_all_uses_with(new_input_node)
        new_input_node.meta.update(node.meta)
        g.erase_node(node)
        new_input_node.name = node.name

    if constant is not None:
        # needed to suppress `does not reference an nn.Module, nn.Parameter, or buffer` warning
        gm.register_buffer(qualname, constant)
        setattr(gm, qualname, constant)
        # mark any constants created during freezing
        maybe_set_is_frozen_param(constant)


def is_const_source(
    node: torch.fx.Node, lifted_constant_names: list[str] | None
) -> bool:
    return node.op == "get_attr" or node.name in (lifted_constant_names or ())

````
- **EN**: Introduces function `is_const_source`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`is_const_source`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python

class ConstantFolder(torch.fx.Interpreter):
    def __init__(
        self,
        gm: torch.fx.GraphModule,
        skip_constructors: bool = False,
        lifted_constant_names: list[str] | None = None,
        skip_folding_node_fn: Callable[[torch.fx.Node], bool] | None = None,
    ) -> None:
        super().__init__(gm)
        self.node_replacements: dict[torch.fx.Node, Any] = {}
        self.replaced_uses: dict[torch.fx.Node, int] = collections.Counter()
        self.unknown_value = object()
        self.skip_constructors: bool = skip_constructors

        # overwrite this to deallocate env values if their only remaining use
        # is the output
        self.user_to_last_uses = self.node_to_last_non_output_use()
        self.lifted_constant_names = lifted_constant_names
        self.deferred_value = object()
````
- **EN**: Introduces class `ConstantFolder`, function `__init__`. Initializes or updates values such as `gm`, `skip_constructors`, `lifted_constant_names`, and `skip_folding_node_fn`.
- **CN**: 这里定义了类`ConstantFolder`、函数`__init__`。初始化或更新了 `gm`、`skip_constructors`、`lifted_constant_names`、`skip_folding_node_fn` 等值。

### Lines 101-120 / 第 101-120 行
````python
        self.skip_folding_node_fn = skip_folding_node_fn

    def _support_dynamic_shape(self) -> bool:
        # ConstantFolder not support dynamic shape now
        return False

    def _deduce_value(self, node: torch.fx.Node) -> Any:
        if self.lifted_constant_names is None:
            return super().run_node(node)
        # if lifted_constant_names is passed in, no concrete value is available
        # so we just check if all inputs have values
        if self.skip_folding_node_fn is not None and self.skip_folding_node_fn(node):
            return self.unknown_value
        flattened_node_inps = pytree.arg_tree_leaves(*node.args, **node.kwargs)
        for inp in flattened_node_inps:
            if (
                isinstance(inp, torch.fx.Node)
                and inp.name not in (self.lifted_constant_names or ())
                and self.env[inp] is not self.deferred_value
            ):
````
- **EN**: Introduces function `_support_dynamic_shape`, function `_deduce_value`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `flattened_node_inps`.
- **CN**: 这里定义了函数`_support_dynamic_shape`、函数`_deduce_value`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `flattened_node_inps` 等值。

### Lines 121-140 / 第 121-140 行
````python
                return self.unknown_value
        return self.deferred_value

    def is_impure(self, node: torch.fx.node.Node) -> bool:
        def is_woq_int8_pattern(node: torch.fx.node.Node) -> bool:
            return (
                node.target is torch.ops.prims.convert_element_type.default  # type: ignore[return-value]
                and isinstance(node.args[0], torch.fx.Node)
                and "val" in node.args[0].meta
                and node.args[0].meta["val"].dtype == torch.int8  # type: ignore[union-attr]
                and node.args[1] == torch.bfloat16
            )

        if (
            is_woq_int8_pattern(node)
            or (
                node.target is torch.ops.aten.permute.default
                and len(node.users) == 1
                and is_woq_int8_pattern(next(iter(node.users)))
            )
````
- **EN**: Introduces function `is_impure`, function `is_woq_int8_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`is_impure`、函数`is_woq_int8_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
        ) and is_const_source(
            node.args[0],  # type: ignore[arg-type]
            self.lifted_constant_names,
        ):
            # Case 1: int8_weight -> dq -> bf16_weight
            # Case 2: int8_weight -> permute -> dq -> bf16_weight
            return True

        quant_registered = (
            getattr(torch.ops.quantized_decomposed, "dequantize_per_channel", None)
            is not None
        )
        if quant_registered and node.target in [
            torch.ops.quantized_decomposed.dequantize_per_channel.default,
            torch.ops.quantized_decomposed.dequantize_per_tensor.default,
            torch.ops.quantized_decomposed.dequantize_per_tensor.tensor,
            torch.ops.quantized_decomposed.convert_element_type.no_fuse,
        ]:
            # For the pattern fp32_weight -> q -> dq
            # We only folding fp32_weight -> q
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `quant_registered`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `quant_registered` 等值。

### Lines 161-180 / 第 161-180 行
````python
            # int8_weight and leave dq in graph to be fused
            return True

        if node.target in _dont_constant_fold:
            return True
        return False

    def node_to_last_non_output_use(self) -> dict[torch.fx.Node, list[torch.fx.Node]]:
        last_non_output_use = collections.defaultdict(list)
        seen_uses = OrderedSet[torch.fx.Node]()
        output_node = next(iter(reversed(self.module.graph.nodes)))  # type: ignore[arg-type, union-attr]

        for node in reversed(self.module.graph.nodes):  # type: ignore[arg-type, union-attr]
            if node.target == "output":
                continue

            def add_use(inp: torch.fx.Node) -> None:
                if inp in seen_uses:
                    return

````
- **EN**: Introduces function `node_to_last_non_output_use`, function `add_use`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `last_non_output_use`, `seen_uses`, and `output_node`.
- **CN**: 这里定义了函数`node_to_last_non_output_use`、函数`add_use`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `last_non_output_use`、`seen_uses`、`output_node` 等值。

### Lines 181-200 / 第 181-200 行
````python
                seen_uses.add(inp)
                last_non_output_use[node].append(inp)

            # In-place is fine since we don't mutate
            pytree.tree_map_only_(torch.fx.Node, add_use, (node.args, node.kwargs))

            # if this node is only used in output, we want to gc it right away
            if len(node.users) == 1 and output_node in node.users:
                last_non_output_use[node].append(node)

        return last_non_output_use

    @typing_extensions.override
    def run_node(self, node: torch.fx.Node) -> Any:
        if node.target == "output":
            # because we remove nodes from env on last non output use,
            # re-define them now or we'll get error in interpreter
            def set_env(arg: torch.fx.Node) -> None:
                self.env[arg] = self.unknown_value

````
- **EN**: Introduces function `run_node`, function `set_env`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`run_node`、函数`set_env`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 201-220 / 第 201-220 行
````python
            # In-place is fine since we don't mutate
            pytree.tree_map_only_(torch.fx.Node, set_env, node.args)
            return super().run_node(node)

        args, kwargs = self.fetch_args_kwargs_from_env(node)
        flattened_inputs = pytree.arg_tree_leaves(*args, **kwargs)

        # We need to do this weird thing because in cases where flattened_inputs
        # contains a ScriptObject, equality checking results in a type error if
        # the types are different.
        if any(
            type(self.unknown_value) is type(input_) and self.unknown_value == input_
            for input_ in flattened_inputs
        ):
            return self.unknown_value

        # TODO - fix errors with this
        if (
            node.op == "call_function"
            and node.target is aten._efficientzerotensor.default
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `flattened_inputs`. This range continues the implementation of function `ConstantFolder.run_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `flattened_inputs` 等值。这一段延续了函数`ConstantFolder.run_node` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
        ):
            return self.unknown_value

        # TODO - constant folding triton kernel returns the inputs -- fix this
        if (
            node.op == "call_function"
            and node.name == "triton_kernel_wrapper_functional_proxy"
        ):
            return self.unknown_value

        # skip constructors, since inductor generates optimal code for them already
        # and turning into tensor would result in an additional global memory read
        # TODO - more complicated strategy
        if (
            self.skip_constructors
            and not is_const_source(node, self.lifted_constant_names)
            and not any(isinstance(e, torch.Tensor) for e in flattened_inputs)
        ):
            return self.unknown_value

````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `ConstantFolder.run_node`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`ConstantFolder.run_node` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
        # All mutations should either be removed or on inputs which we did not make constant
        if (
            isinstance(node.target, torch._ops.OpOverload)
            and torch.Tag.nondeterministic_seeded in node.target.tags
        ):
            return self.unknown_value

        if node.op == "call_function" and isinstance(
            node.target, torch._ops.HigherOrderOperator
        ):
            return self.unknown_value

        out = self._deduce_value(node)

        if isinstance(
            out,
            (
                torch._C.ScriptObject,
                torch._library.fake_class_registry.FakeScriptObject,
            ),
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `out` 等值。

### Lines 261-280 / 第 261-280 行
````python
        ):
            return out

        if out is self.unknown_value:
            return self.unknown_value

        if not is_const_source(node, self.lifted_constant_names) and (
            isinstance(out, torch.Tensor) or out is self.deferred_value
        ):
            if (
                out is not self.deferred_value
                and out.device.type == "meta"  # pyrefly: ignore[missing-attribute]
            ):
                return out

            if not self.insertable_tensor_check(
                out  # pyrefly: ignore[bad-argument-type]
            ):
                return out

````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `ConstantFolder.run_node`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`ConstantFolder.run_node` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
            if self.is_impure(node):
                return self.unknown_value

            self.add_node_replacement(node, out)  # pyrefly: ignore[bad-argument-type]

            flattened_node_inps = pytree.arg_tree_leaves(*node.args, **node.kwargs)

            for n in flattened_node_inps:
                if not isinstance(n, torch.fx.Node):
                    continue

                self.replaced_uses[n] += 1

            for to_delete in self.user_to_last_uses.get(node, []):
                if self.replaced_uses[to_delete] == len(to_delete.users):
                    self.node_replacements.pop(to_delete, None)

        return out

    def insertable_tensor_check(self, tensor: torch.Tensor) -> bool:
````
- **EN**: Introduces function `insertable_tensor_check`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `flattened_node_inps`.
- **CN**: 这里定义了函数`insertable_tensor_check`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `flattened_node_inps` 等值。

### Lines 301-320 / 第 301-320 行
````python
        return True

    def add_node_replacement(self, node: torch.fx.Node, tensor: torch.Tensor) -> None:
        self.node_replacements[node] = tensor

    def run(self) -> Any:  # type: ignore[override]
        env: dict[torch.fx.Node, Any] = {}
        self.insert_placerholder_values(env)
        return super().run(initial_env=env)

    def insert_placerholder_values(self, env: dict[torch.fx.Node, Any]) -> None:
        for n in self.module.graph.find_nodes(op="placeholder"):  # type: ignore[operator, union-attr]
            env[n] = self.unknown_value  # type: ignore[assignment]
        if self.lifted_constant_names is None:
            return
        for n in self.module.graph.nodes:  # type: ignore[union-attr]
            if n.name in (self.lifted_constant_names or ()):
                env[n] = self.deferred_value


````
- **EN**: Introduces function `add_node_replacement`, function `run`, function `insert_placerholder_values`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `env`.
- **CN**: 这里定义了函数`add_node_replacement`、函数`run`、函数`insert_placerholder_values`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `env` 等值。

### Lines 321-340 / 第 321-340 行
````python
def constant_fold(
    gm: torch.fx.GraphModule,
    constraint_fn: Callable[[torch.fx.Node], bool] | None = None,
) -> None:
    with torch.utils._python_dispatch._disable_current_modes():
        cf = ConstantFolder(gm, skip_constructors=True)
        cf.run()

        for node, constant in cf.node_replacements.items():
            if constraint_fn is not None and not constraint_fn(node):
                continue
            replace_node_with_constant(gm, node, constant)

        erased_params = []
        for node in gm.graph.find_nodes(op="get_attr"):
            if len(node.users) == 0:
                if hasattr(gm, node.target):
                    delattr(gm, node.target)
                erased_params.append(node)

````
- **EN**: Introduces function `constant_fold`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `gm`, `constraint_fn`, `cf`, and `erased_params`.
- **CN**: 这里定义了函数`constant_fold`。包含分支、循环或上下文管理等控制流。初始化或更新了 `gm`、`constraint_fn`、`cf`、`erased_params` 等值。

### Lines 341-360 / 第 341-360 行
````python
        for node in erased_params:
            gm.graph.erase_node(node)

        gm.graph.eliminate_dead_code()
        gm.graph.lint()
        gm.recompile()


def constant_graph_tag(
    gm: torch.fx.GraphModule,
    skip_constructors: bool = True,
    lifted_constant_names: list[str] | None = None,
    skip_folding_node_fn: Callable[[torch.fx.Node], bool] | None = None,
) -> None:
    with torch.utils._python_dispatch._disable_current_modes():
        cf = ConstantFolder(
            gm,
            skip_constructors=skip_constructors,
            lifted_constant_names=lifted_constant_names,
            skip_folding_node_fn=skip_folding_node_fn,
````
- **EN**: Introduces function `constant_graph_tag`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `gm`, `skip_constructors`, `lifted_constant_names`, `skip_folding_node_fn`, and `cf`.
- **CN**: 这里定义了函数`constant_graph_tag`。包含分支、循环或上下文管理等控制流。初始化或更新了 `gm`、`skip_constructors`、`lifted_constant_names`、`skip_folding_node_fn`、`cf` 等值。

### Lines 361-380 / 第 361-380 行
````python
        )
        cf.run()

        for node in gm.graph.nodes:
            if skip_folding_node_fn is not None and skip_folding_node_fn(node):
                node.meta[META_TAG] = MODULE_TAG
                continue
            if (
                is_const_source(node, lifted_constant_names)
                or node in cf.node_replacements
                or node in cf.replaced_uses
            ):
                node.meta[META_TAG] = CONST_MODULE_TAG
            else:
                node.meta[META_TAG] = MODULE_TAG


def run_and_get_constant_graph(
    gm: torch.fx.GraphModule,
    skip_constructors: bool = True,
````
- **EN**: Introduces function `run_and_get_constant_graph`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `gm`, and `skip_constructors`.
- **CN**: 这里定义了函数`run_and_get_constant_graph`。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`gm`、`skip_constructors` 等值。

### Lines 381-400 / 第 381-400 行
````python
    lifted_constant_names: list[str] | None = None,
    skip_folding_node_fn: Callable[[torch.fx.Node], bool] | None = None,
) -> torch.fx.GraphModule:
    """
    Construct a GraphModule which corresponds to the part which could be
    constant folded in provided gm.
    """

    constant_graph_tag(
        gm, skip_constructors, lifted_constant_names, skip_folding_node_fn
    )

    def untag(node: torch.fx.Node) -> bool:
        used_to_fold = False
        for u in node.users:
            if u.meta[META_TAG] == CONST_MODULE_TAG:
                used_to_fold = True
                break
        if not used_to_fold:
            node.meta[META_TAG] = MODULE_TAG
````
- **EN**: Introduces function `untag`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `lifted_constant_names`, `skip_folding_node_fn`, and `used_to_fold`.
- **CN**: 这里定义了函数`untag`。包含分支、循环或上下文管理等控制流。初始化或更新了 `lifted_constant_names`、`skip_folding_node_fn`、`used_to_fold` 等值。

### Lines 401-420 / 第 401-420 行
````python
        return used_to_fold

    # We rewrite the tags, if it's a constant being directly consumed, without
    # any folding opportunity, we keep it in main gm.
    for node in gm.graph.nodes:
        if node.op == "get_attr" or (node.name in (lifted_constant_names or ())):
            untag(node)

    new_graph = torch.fx.Graph()

    node_remapping: dict[torch.fx.Node, torch.fx.Node] = {}
    output_nodes = []
    for node in gm.graph.nodes:
        if node.meta[META_TAG] == MODULE_TAG:
            continue

        new_node = new_graph.node_copy(node, lambda x: node_remapping[x])
        node_remapping[node] = new_node

        for user in node.users:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_graph`, `node_remapping`, `output_nodes`, and `new_node`. This range continues the implementation of function `run_and_get_constant_graph`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_graph`、`node_remapping`、`output_nodes`、`new_node` 等值。这一段延续了函数`run_and_get_constant_graph` 的具体实现。

### Lines 421-429 / 第 421-429 行
````python
            if user.meta[META_TAG] == MODULE_TAG:
                output_nodes.append(new_node)
                break

    new_graph.output(tuple(output_nodes))
    new_graph.lint()
    new_gm = torch.fx.GraphModule(gm, new_graph)

    return new_gm
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_gm`. This range continues the implementation of function `run_and_get_constant_graph`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_gm` 等值。这一段延续了函数`run_and_get_constant_graph` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `ConstantFolder`  
  **CN**: 主要类：`ConstantFolder`
- **EN**: Primary functions: `add_dont_constant_fold`, `clear_dont_constant_fold`, `replace_node_with_constant`, `is_const_source`, `constant_fold`, `constant_graph_tag`, and `...+1`  
  **CN**: 主要函数：`add_dont_constant_fold`、`clear_dont_constant_fold`、`replace_node_with_constant`、`is_const_source`、`constant_fold`、`constant_graph_tag`、`另有1项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `collections.abc`, `typing`
- **Third-party / 第三方**: `typing_extensions`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._pytree`, `torch._inductor.freezing_utils`, `torch.utils._ordered_set`
