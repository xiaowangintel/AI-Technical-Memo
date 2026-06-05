# distributed_autotune.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/distributed_autotune.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `_DistributedAutotuneState`, `_DistributedAutotuneInfo`, `_DistributedAutotuneBuffer`, and `_SerializedChoice`. It exposes functions such as `get_autotune_pg`, `schedule`, `graph_context`, `maybe_autotune_remote`, `_sync`, `_autotune_local_nodes`, and `...+1`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `_DistributedAutotuneState`、`_DistributedAutotuneInfo`、`_DistributedAutotuneBuffer`、`_SerializedChoice` 等类。同时提供 `get_autotune_pg`、`schedule`、`graph_context`、`maybe_autotune_remote`、`_sync`、`_autotune_local_nodes`、`另有1项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

import contextlib
import dataclasses
from typing import Any, TYPE_CHECKING
from unittest.mock import patch

import sympy

import torch._logging
import torch.distributed as dist
import torch.fx
from torch.utils._ordered_set import OrderedSet

from . import config, select_algorithm
from .ir import (
    Buffer,
    ChoiceCaller,
    Layout,
    MultiTemplateBuffer,
````
- **EN**: Imports dependencies such as `__future__`, `contextlib`, `dataclasses`, `typing`, `unittest.mock`, `sympy`, and `...+6` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `__future__`、`contextlib`、`dataclasses`、`typing`、`unittest.mock`、`sympy`、`另有6项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
    OperationBuffer,
    StorageBox,
    TensorBox,
)
from .kernel_inputs import KernelInputs, MMKernelInputs
from .scheduler import SchedulerNode
from .virtualized import NullHandler, V


if TYPE_CHECKING:
    from collections.abc import Generator, Sequence


_DISTRIBUTED_AUTOTUNE_KEY = "distributed_autotune"

_AUTOTUNE_PG: dist.ProcessGroup | None = None


@dataclasses.dataclass
class _DistributedAutotuneState:
````
- **EN**: Imports dependencies such as `.kernel_inputs`, `.scheduler`, `.virtualized`, and `collections.abc` for the logic in this range. Introduces class `_DistributedAutotuneState`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Applies decorators to register behavior or alter how the following definition is constructed.
- **CN**: 这里导入了 `.kernel_inputs`、`.scheduler`、`.virtualized`、`collections.abc` 等依赖，为后续逻辑提供基础能力。这里定义了类`_DistributedAutotuneState`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。使用装饰器来注册行为，或改变后续定义的构造方式。

### Lines 41-60 / 第 41-60 行
````python
    """
    State used to track autotuning during a graph_context()
    """

    # This is the next operator index. Used to figure out which rank should do
    # the autotuning.
    autotuned_index: int = 0

    # For debugging - used to make sure that we autotune the same number of
    # local operators that we expected to.
    autotuned_local_count: int = 0


@dataclasses.dataclass
class _DistributedAutotuneInfo:
    index: int
    local: bool


def get_autotune_pg() -> dist.ProcessGroup | None:
````
- **EN**: Introduces class `_DistributedAutotuneInfo`, function `get_autotune_pg`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `autotuned_index`, `autotuned_local_count`, `index`, and `local`.
- **CN**: 这里定义了类`_DistributedAutotuneInfo`、函数`get_autotune_pg`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `autotuned_index`、`autotuned_local_count`、`index`、`local` 等值。

### Lines 61-80 / 第 61-80 行
````python
    if dist.is_available() and dist.is_initialized():
        global _AUTOTUNE_PG
        if _AUTOTUNE_PG is None:
            _AUTOTUNE_PG = dist.distributed_c10d._new_group_with_tag(
                pg_tag="pt2_distributed_autotune_pg"
            )
        return _AUTOTUNE_PG

    return None


def schedule(scheduler: torch._inductor.scheduler.Scheduler) -> None:
    """
    Finish the distributed autotuning by propagating the autotuning results
    between the ranks and then replacing the placeholder with the real Buffer.
    """
    assert config.distributed_max_autotune_gemm
    autotune_results = _autotune_local_nodes(scheduler)
    choices_by_index = _sync(autotune_results)
    _autotune_remote_nodes(scheduler, choices_by_index)
````
- **EN**: Introduces function `schedule`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_AUTOTUNE_PG`, `pg_tag`, `autotune_results`, and `choices_by_index`.
- **CN**: 这里定义了函数`schedule`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_AUTOTUNE_PG`、`pg_tag`、`autotune_results`、`choices_by_index` 等值。

### Lines 81-100 / 第 81-100 行
````python


@contextlib.contextmanager
def graph_context() -> Generator[None, None, None]:
    """
    Wrapped around processing a graph, sets up figuring out which ranks tune
    which shapes.
    """
    assert not isinstance(
        V.get_distributed_autotune_state(check_poisoned=False),  # type: ignore[call-arg]
        _DistributedAutotuneState,
    )
    V.set_distributed_autotune_state(_DistributedAutotuneState())
    try:
        yield
    finally:
        V.set_distributed_autotune_state(NullHandler())


def maybe_autotune_remote(
````
- **EN**: Introduces function `graph_context`, function `maybe_autotune_remote`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`graph_context`、函数`maybe_autotune_remote`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python
    name: str, choices: list[ChoiceCaller], inputs: list[Buffer], layout: Layout
) -> TensorBox | None:
    """
    Used by an op (like `mm`) to determine if the op should be autotuned
    locally (returns None) or remotely (returns a placeholder Buffer).
    """
    if not config.distributed_max_autotune_gemm:
        return None

    if not (autotune_pg := get_autotune_pg()):
        return None

    if len(choices) <= 1:
        return None

    state = V.distributed_autotune_state
    index = state.autotuned_index
    state.autotuned_index += 1
    local = index % autotune_pg.size() == autotune_pg.rank()

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `state`, `index`, and `local`. This range continues the implementation of function `maybe_autotune_remote`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`state`、`index`、`local` 等值。这一段延续了函数`maybe_autotune_remote` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
    V.current_node.meta[_DISTRIBUTED_AUTOTUNE_KEY] = _DistributedAutotuneInfo(
        index, local
    )
    if local:
        state.autotuned_local_count += 1
        return None

    return torch._inductor.ir.TensorBox.create(
        _DistributedAutotuneBuffer(name, inputs, layout)
    )


class _DistributedAutotuneBuffer(MultiTemplateBuffer):
    """
    A MultiTemplateBuffer which represents a kernel being autotuned on a
    different rank. When `schedule` is called this will be replaced by the
    "real" buffer.
    """

    # Name of the kernel being autotuned.
````
- **EN**: Introduces class `_DistributedAutotuneBuffer`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`_DistributedAutotuneBuffer`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
    _kernel_name: str

    def __init__(
        self,
        kernel_name: str,
        inputs: list[Buffer],
        layout: Layout,
    ) -> None:
        super().__init__(
            layout,
            inputs,
            choice_timings_fn=self._dummy_choice_timings,
            unfiltered_choices=[],
            allowed_prologue_inps=OrderedSet({}),
        )

        self._kernel_name = kernel_name

    def _dummy_choice_timings(
        self, _hint_override: int | None
````
- **EN**: Introduces function `__init__`, function `_dummy_choice_timings`. Initializes or updates values such as `_kernel_name`, `kernel_name`, `inputs`, `layout`, `choice_timings_fn`, `unfiltered_choices`, and `...+1`.
- **CN**: 这里定义了函数`__init__`、函数`_dummy_choice_timings`。初始化或更新了 `_kernel_name`、`kernel_name`、`inputs`、`layout`、`choice_timings_fn`、`unfiltered_choices`、`另有1项` 等值。

### Lines 161-180 / 第 161-180 行
````python
    ) -> dict[ChoiceCaller, float]:
        # This should never get called. It means that a remote autotune was
        # scheduled but never filled in.
        raise NotImplementedError

    def autotune(self, ser_choice: _SerializedChoice) -> TensorBox:
        """
        Given a _SerializedChoice (autotune results from another rank)
        compute the final TensorBox.
        """

        from .select_algorithm import autotune_select_algorithm

        with patch.object(V.graph, "scheduler", None):
            kernel_inputs = MMKernelInputs([*self.original_inputs])
            assert isinstance(self.layout, Layout)
            choice = ser_choice.get_choice(self.layout, kernel_inputs)
            buffer, _ = autotune_select_algorithm(
                self._kernel_name,
                [choice],
````
- **EN**: Imports dependencies such as `.select_algorithm` for the logic in this range. Introduces function `autotune`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.select_algorithm` 等依赖，为后续逻辑提供基础能力。这里定义了函数`autotune`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python
                kernel_inputs.nodes(),
                self.layout,
            )
            assert isinstance(buffer, TensorBox)
            return buffer


# Can we make this async?
def _sync(autotune_results: list[_SerializedChoice]) -> Sequence[_SerializedChoice]:
    """
    Perform the all_gather to collect the autotune results from all the ranks.
    """

    autotune_pg = get_autotune_pg()
    assert autotune_pg

    # Perform allgather
    all_states: list[list[_SerializedChoice]] = [None] * autotune_pg.size()  # type: ignore[list-item]
    torch.distributed.all_gather_object(all_states, autotune_results, group=autotune_pg)

````
- **EN**: Introduces function `_sync`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `autotune_pg`, and `all_states`.
- **CN**: 这里定义了函数`_sync`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `autotune_pg`、`all_states` 等值。

### Lines 201-220 / 第 201-220 行
````python
    node_count = sum(len(x) for x in all_states)
    # It's faster to briefly lie about the type than to unzip the results and append.
    choices_by_index: list[_SerializedChoice] = [None] * node_count  # type: ignore[list-item]

    check_count = 0
    for other_results in all_states:
        for choice in other_results:
            assert isinstance(choice, _SerializedChoice)
            assert choices_by_index[choice.index] is None
            choices_by_index[choice.index] = choice
            check_count += 1

    assert node_count == check_count, f"count mismatch: {node_count} != {check_count}"
    return choices_by_index


class _SerializedChoice:
    """
    This is a serializer for the autotune choice. KernelTemplateChoice can't
    be serialized directly (the template and inputs prevent this) so we need to
````
- **EN**: Introduces class `_SerializedChoice`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_count`, `choices_by_index`, and `check_count`.
- **CN**: 这里定义了类`_SerializedChoice`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node_count`、`choices_by_index`、`check_count` 等值。

### Lines 221-240 / 第 221-240 行
````python
    serialize it by parts and reconstruct later on.
    """

    def __init__(self, index: int, choice: ChoiceCaller) -> None:
        self.index = index
        self.template_uid = _SerializedChoice._template_uid_from_choice(choice)
        self.kwargs = self._compute_kwargs(choice.description)

    def get_choice(self, layout: Layout, inputs: KernelInputs) -> ChoiceCaller | None:
        """
        Deserialize the ChoiceCaller and return it.
        """

        template = self._template_from_uid()

        kwargs = {**self.kwargs}
        if "BLOCK_K" in kwargs:
            # TODO: Do we really need to externally compute this value? If it's
            # needed I'm surprised it's not just part of the original template
            # description.
````
- **EN**: Introduces function `__init__`, function `get_choice`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `template`, and `kwargs`.
- **CN**: 这里定义了函数`__init__`、函数`get_choice`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `template`、`kwargs` 等值。

### Lines 241-260 / 第 241-260 行
````python
            # This needs the actual 'k' to figure out the value.
            k = inputs.nodes()[0].get_size()[1]
            kwargs["EVEN_K"] = sympy.gcd(k, kwargs["BLOCK_K"]) == kwargs["BLOCK_K"]

        extra_kwargs: dict[str, Any] = {}
        from .kernel_template_choice import (
            DictKernelTemplateParams,
            KernelTemplateChoice,
        )

        params = DictKernelTemplateParams(kwargs)
        ktc = KernelTemplateChoice(template, params, extra_kwargs, layout, inputs)
        return ktc.choice

    @staticmethod
    def _compute_kwargs(description: str) -> dict[str, int | str | bool]:
        """
        Given a template description turn it into input kwargs.
        """
        if not description:
````
- **EN**: Imports dependencies such as `.kernel_template_choice` for the logic in this range. Introduces function `_compute_kwargs`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `.kernel_template_choice` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_compute_kwargs`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。

### Lines 261-280 / 第 261-280 行
````python
            return {}

        # TODO: It seems like it would be better if the template could provide
        # this directly instead of having to parse a string.
        kwargs: dict[str, int | str | bool] = {}
        for cfg in description.split(","):
            key, val = cfg.split("=", 1)
            key, val = key.strip(), val.strip()
            if val == "True":
                kwargs[key] = True
            elif val == "False":
                kwargs[key] = False
            elif val.isdigit():
                kwargs[key] = int(val)
            else:
                assert val.startswith("'") and val.endswith("'")
                kwargs[key] = val[1:-1]
        return kwargs

    @staticmethod
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kwargs`, and `else`.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kwargs`、`else` 等值。

### Lines 281-300 / 第 281-300 行
````python
    def _template_uid_from_choice(choice: ChoiceCaller) -> str:
        """
        Given a ChoiceCaller figure out which template represents it. This
        is reversed by _template_from_uid().
        """

        # We need a better way to do this - right now we need to add each
        # supported template directly.
        if isinstance(choice, select_algorithm.ExternKernelCaller):
            if choice.choice.name == "mm":
                return "torch._inductor.kernel.mm.aten_mm"
            else:
                raise RuntimeError(f"TODO: kernel {choice.choice.name!r}")
        elif isinstance(choice, select_algorithm.TritonTemplateCaller):
            return "torch._inductor.kernel.mm.mm_template"
        else:
            raise RuntimeError(f"TODO: {type(choice)}")

    def _template_from_uid(self) -> Any:
        """
````
- **EN**: Introduces function `_template_uid_from_choice`, function `_template_from_uid`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`_template_uid_from_choice`、函数`_template_from_uid`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 301-320 / 第 301-320 行
````python
        See _template_uid_from_choice().
        """
        parts = self.template_uid.split(".")
        obj = globals()[parts[0]]
        for k in parts[1:]:
            obj = getattr(obj, k)
        return obj


def _autotune_local_nodes(
    scheduler: torch._inductor.scheduler.Scheduler,
) -> list[_SerializedChoice]:
    """
    Go through the nodes in the scheduler and autotune the kernels which
    should be autotuned by this rank.
    """

    autotune_results: list[_SerializedChoice] = []

    for node in scheduler.nodes:
````
- **EN**: Introduces function `_autotune_local_nodes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `parts`, `obj`, `scheduler`, and `autotune_results`.
- **CN**: 这里定义了函数`_autotune_local_nodes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `parts`、`obj`、`scheduler`、`autotune_results` 等值。

### Lines 321-340 / 第 321-340 行
````python
        if not isinstance(node, SchedulerNode):
            continue

        if (inner_node := node.node) is None:
            continue

        if isinstance(inner_node, _DistributedAutotuneBuffer):
            # This is marked for remote autotuning.
            continue

        if not isinstance(inner_node, MultiTemplateBuffer):
            continue

        if (origin_node := inner_node.origin_node) is None:
            continue

        if (meta := origin_node.meta) is None:
            continue

        info = meta.get(_DISTRIBUTED_AUTOTUNE_KEY)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `info`. This range continues the implementation of function `_autotune_local_nodes`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `info` 等值。这一段延续了函数`_autotune_local_nodes` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
        if info is None:
            continue

        assert info.local

        # We force autotuning here
        # Still takes advantage of async precompile
        # We need all the configs before fusion
        min_choice, _ = inner_node.get_min_choice()

        choice = _SerializedChoice(info.index, min_choice)
        autotune_results.append(choice)

    state = V.distributed_autotune_state
    assert len(autotune_results) == state.autotuned_local_count, (
        f"incorrect local autotuned nodes found ({len(autotune_results)} != {state.autotuned_local_count})"
    )
    return autotune_results


````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `choice`, and `state`. This range continues the implementation of function `_autotune_local_nodes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `choice`、`state` 等值。这一段延续了函数`_autotune_local_nodes` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
def _autotune_remote_nodes(
    scheduler: torch._inductor.scheduler.Scheduler,
    choices_by_index: Sequence[_SerializedChoice],
) -> None:
    """
    Go through the nodes in the scheduler and autotune the nodes that were
    autotuned on remote ranks.
    """

    for i, node in enumerate(scheduler.nodes):
        if isinstance(node, SchedulerNode) and isinstance(
            (dist_node := node.node), _DistributedAutotuneBuffer
        ):
            assert dist_node.origin_node is not None
            info = dist_node.origin_node.meta[_DISTRIBUTED_AUTOTUNE_KEY]
            out_tensorbox = dist_node.autotune(choices_by_index[info.index])

            out_storage = out_tensorbox.data
            assert isinstance(out_storage, StorageBox)
            out_buffer = out_storage.data
````
- **EN**: Introduces function `_autotune_remote_nodes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scheduler`, `choices_by_index`, `info`, `out_tensorbox`, `out_storage`, and `out_buffer`.
- **CN**: 这里定义了函数`_autotune_remote_nodes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scheduler`、`choices_by_index`、`info`、`out_tensorbox`、`out_storage`、`out_buffer` 等值。

### Lines 381-385 / 第 381-385 行
````python
            assert isinstance(out_buffer, OperationBuffer)

            assert out_buffer.layout == dist_node.layout

            scheduler._replace_node(out_buffer, dist_node, i, node)
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `_autotune_remote_nodes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`_autotune_remote_nodes` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Primary classes: `_DistributedAutotuneState`, `_DistributedAutotuneInfo`, `_DistributedAutotuneBuffer`, and `_SerializedChoice`  
  **CN**: 主要类：`_DistributedAutotuneState`、`_DistributedAutotuneInfo`、`_DistributedAutotuneBuffer`、`_SerializedChoice`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `contextlib`, `dataclasses`, `typing`, `unittest.mock`, `collections.abc`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch._logging`, `torch.distributed`, `torch.fx`, `torch.utils._ordered_set`, `.`, `.ir`, `.kernel_inputs`, `.scheduler`, `.virtualized`, `.select_algorithm`, `.kernel_template_choice`
