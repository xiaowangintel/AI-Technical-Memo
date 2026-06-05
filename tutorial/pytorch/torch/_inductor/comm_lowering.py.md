# comm_lowering.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/comm_lowering.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `can_realize_as_comm_buffer`, `realize_as_comm_buffer`, `_get_data`, `mark_as_skip_wait`, `should_skip_wait`, `_should_lower_as_one_shot_all_reduce`, and `...+4`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `can_realize_as_comm_buffer`、`realize_as_comm_buffer`、`_get_data`、`mark_as_skip_wait`、`should_skip_wait`、`_should_lower_as_one_shot_all_reduce`、`另有4项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import logging

import torch
import torch.utils._pytree as pytree
from torch._inductor.utils import is_symbolic
from torch.utils._ordered_set import OrderedSet

from . import config, ir
from .virtualized import V


log = logging.getLogger(__name__)


# NOTE [lowering-time collective optimization]
#
# In collective communication libraries such as NCCL, every rank maintains
# communication buffers that are remotely accessible by some peers. Depending
# on the underlying transport, remote accessibility may be established via
# mechanisms such as ib_reg_mr, CUDA P2P, or CUDA multicast. Typically, these
# buffers are private to the communication library by default, and
# communication ops copy user data in and out of these buffers.
#
# To prevent these copies, an optimization commonly known as "user buffer
# registration" can be employed. This allows direct establishment of remote
# accessibility on user buffers, eliminating the need for copying. However,
# this optimization introduces stringent usage requirements, which are
````
- **EN**: Imports dependencies such as `logging`, `torch`, `torch.utils._pytree`, `torch._inductor.utils`, `torch.utils._ordered_set`, `.`, and `...+1` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `logging`、`torch`、`torch.utils._pytree`、`torch._inductor.utils`、`torch.utils._ordered_set`、`.`、`另有1项` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `log` 等值。

### Lines 29-56 / 第 29-56 行
````python
# typically hard to satisfy without being intrusive to the user code:
#
# - Establishing remote accessibility is expensive and often done ahead of
# time. In such implementations, all ranks must agree on the set of allocations
# used for every collective op. Failing to meet this requirement can
# lead to runtime errors or even silent correctness issues.
# - Even if the collective communication library supports gracefully falling
# back to "unregistered" implementations, the fallback mechanism would nullify
# the optimization.
# - Some communication mechanisms impose stricter requirements than others. For
# example, CUDA's multicast + multi-mem instructions require all ranks to agree
# not only on the allocations used for every collective but also on the offsets
# within these allocations.
#
# To support all different mechanisms with optimal results, we aim to satisfy
# the strictest requirement for this family of optimizations - we ensures that
# every collective op invocation is guaranteed to operate on the same
# allocation, at the same offset, in every iteration.
#
# For eligible collective ops, we identify communication buffers at lowering
# time and optionally choose to lower the op to a different kernel
# (communication libraries like NCCL handle both registered and non-registered
# buffers transparently within the same op, though some may require different
# ops for different cases). Later, the codegen will perform "persistent
# allocation" to satisfy the aforementioned constraints, and optionally,
# perform buffer planning to optimize overall memory usage.
def can_realize_as_comm_buffer(
    x: ir.TensorBox, comm_buffer_type: ir.CommBufferType
````
- **EN**: Introduces function `can_realize_as_comm_buffer`. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `x`.
- **CN**: 这里定义了函数`can_realize_as_comm_buffer`。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `x` 等值。

### Lines 57-84 / 第 57-84 行
````python
) -> bool:
    """
    Check if an input can be realized as a comm buffer of the specified
    `comm_buffer_type`.
    """
    data = _get_data(x)

    if isinstance(data, ir.Loops):
        return True

    # We cannot realize buffers as comm buffers if we don't control their
    # allocation.
    if isinstance(data, ir.Buffer) and not data.should_allocate():
        return False

    layout = data.get_output_spec()
    if isinstance(layout, ir.CommBufferLayout):
        return True

    if isinstance(layout, ir.FixedLayout):
        return True

    if isinstance(layout, ir.FlexibleLayout) and not is_symbolic(data.get_numel()):
        return True

    return False


````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `data`, and `layout`. This range continues the implementation of function `can_realize_as_comm_buffer`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `data`、`layout` 等值。这一段延续了函数`can_realize_as_comm_buffer` 的具体实现。

### Lines 85-112 / 第 85-112 行
````python
def realize_as_comm_buffer(
    x: ir.TensorBox,
    comm_buffer_type: ir.CommBufferType,
    group_name: "torch.distributed.distributed_c10d.GroupName",
) -> None:
    """
    Realize an input as a comm buffer of the specified `comm_buffer_type`.

    Specifically, this realizes the underlying buffer if it's still unrealized
    and changes the layout of the buffer to `ir.CommBufferLayout`.
    """
    x.realize()
    buffer = _get_data(x)
    assert isinstance(buffer, ir.Buffer)

    layout = buffer.get_output_spec()
    if isinstance(layout, ir.CommBufferLayout):
        return

    # The buffer may have already been frozen to FixedLayout if it was used
    # by another operation before the comm operation.
    if not isinstance(layout, (ir.FlexibleLayout, ir.FixedLayout)):
        raise AssertionError(
            "A buffer can only be realized as a comm buffer if it "
            f"has `FlexibleLayout` or `FixedLayout` (got {layout})."
        )

    if is_symbolic(buffer.get_numel()):
````
- **EN**: Introduces function `realize_as_comm_buffer`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `x`, `comm_buffer_type`, `group_name`, `buffer`, and `layout`.
- **CN**: 这里定义了函数`realize_as_comm_buffer`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `x`、`comm_buffer_type`、`group_name`、`buffer`、`layout` 等值。

### Lines 113-140 / 第 113-140 行
````python
        raise AssertionError(
            "A buffer with symbolic shape cannot be converted to "
            f"a comm buffer (got {layout})."
        )

    buffer.layout = ir.CommBufferLayout(
        layout=layout,
        comm_buffer_type=comm_buffer_type,
        group_name=group_name,
    )


def _get_data(x: ir.TensorBox) -> ir.IRNode:
    if isinstance(x.data, ir.BaseView):
        # TensorBox -> *View -> StorageBox -> IRNode
        node = x.data.unwrap_view()
        assert isinstance(node, (ir.BaseView, ir.MutableBox))
        return node.data
    elif isinstance(x.data, ir.StorageBox):
        # TensorBox -> StorageBox -> IRNode
        return x.data.data
    else:
        raise AssertionError(
            "Expect the data attr of a `TensorBox` to be either "
            f"an `ir.BaseView` or `ir.StorageBox` (got {x.data})."
        )


````
- **EN**: Introduces function `_get_data`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `layout`, `comm_buffer_type`, `group_name`, `node`, and `else`.
- **CN**: 这里定义了函数`_get_data`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `layout`、`comm_buffer_type`、`group_name`、`node`、`else` 等值。

### Lines 141-168 / 第 141-168 行
````python
_bufs_to_skip_wait = OrderedSet[tuple[int, str]]()


def mark_as_skip_wait(x: ir.IRNode) -> None:
    """
    If a non-blocking collective is lowered as a blocking collective, the wait
    node in the original graph becomes useless and we can skip the lowering it.
    """
    _bufs_to_skip_wait.add((id(V.graph), x.get_name()))


def should_skip_wait(x: ir.IRNode) -> bool:
    return (id(V.graph), x.get_name()) in _bufs_to_skip_wait


def _should_lower_as_one_shot_all_reduce(
    inp: ir.TensorBox,
    reduce_op: str,
    group_name: "torch.distributed.distributed_c10d.GroupName",
):
    from torch.distributed._symmetric_memory import is_symm_mem_enabled_for_group

    inp_size = inp.get_numel() * inp.get_dtype().itemsize
    return (
        config._collective.auto_select
        and is_symm_mem_enabled_for_group(group_name)
        and can_realize_as_comm_buffer(inp, ir.CommBufferType.SYMM_MEM)
        and reduce_op == "sum"
````
- **EN**: Imports dependencies such as `torch.distributed._symmetric_memory` for the logic in this range. Introduces function `mark_as_skip_wait`, function `should_skip_wait`, function `_should_lower_as_one_shot_all_reduce`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `_bufs_to_skip_wait`, `inp`, `reduce_op`, `group_name`, and `inp_size`.
- **CN**: 这里导入了 `torch.distributed._symmetric_memory` 等依赖，为后续逻辑提供基础能力。这里定义了函数`mark_as_skip_wait`、函数`should_skip_wait`、函数`_should_lower_as_one_shot_all_reduce`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `_bufs_to_skip_wait`、`inp`、`reduce_op`、`group_name`、`inp_size` 等值。

### Lines 169-196 / 第 169-196 行
````python
        and inp_size <= config._collective.one_shot_all_reduce_threshold_bytes
    )


def _one_shot_all_reduce(inp: ir.TensorBox, reduce_op, group_name):
    realize_as_comm_buffer(inp, ir.CommBufferType.SYMM_MEM, group_name)
    return pytree.tree_map(
        ir.TensorBox.create,
        ir.FallbackKernel.create(
            torch.ops.symm_mem.one_shot_all_reduce.default,
            inp,
            reduce_op,
            group_name,
        ),
    )


def _create_out_of_place(kernel, inputs, *args) -> ir.IRNode:
    node = ir._CollectiveKernel.create_out_of_place(kernel, inputs, *args)
    assert isinstance(node, ir.IRNode)
    return ir.TensorBox.create(node)


def register_comm_lowerings():
    """
    Register lowerings for the comm subsystem.
    """
    try:
````
- **EN**: Introduces function `_one_shot_all_reduce`, function `_create_out_of_place`, function `register_comm_lowerings`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_one_shot_all_reduce`、函数`_create_out_of_place`、函数`register_comm_lowerings`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 197-224 / 第 197-224 行
````python
        torch.ops._c10d_functional.all_reduce
    except AttributeError:
        log.info(
            "Inductor support for distributed collectives depends on building "
            "torch.distributed"
        )
        return

    from .lowering import (
        add_layout_constraint,
        clone,
        constrain_to_fx_strides,
        copy_,
        register_lowering,
    )

    def register_comm_lowering(fn):
        add_layout_constraint(fn, constrain_to_fx_strides)
        return register_lowering(fn)

    c10d = torch.ops._c10d_functional

    @register_comm_lowering(c10d.all_reduce)  # type: ignore[misc]
    def _all_reduce(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: "torch.distributed.distributed_c10d.GroupName",
    ) -> ir.TensorBox:
````
- **EN**: Imports dependencies such as `.lowering` for the logic in this range. Introduces function `register_comm_lowering`, function `_all_reduce`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `.lowering` 等依赖，为后续逻辑提供基础能力。这里定义了函数`register_comm_lowering`、函数`_all_reduce`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 225-252 / 第 225-252 行
````python
        if _should_lower_as_one_shot_all_reduce(inp, reduce_op, group_name):
            return _one_shot_all_reduce(inp, reduce_op, group_name)

        # Lower as c10d.all_reduce_
        inp = clone(inp)
        if config.reorder_for_compute_comm_overlap:
            # The horizontal fusion of this clone often severely delays the
            # scheduling of the all_reduce_ node. Horizontally fusing this
            # clone can almost never out-perform scheduling the all_reduce_
            # earlier. Also in most cases, this clone is eliminated via
            # in-place reuse. Therefore, we tell the scheduler to not fuse it.
            inp.realize()
            V.graph.no_fuse_buffer_names.add(inp.get_name())
        # pyrefly: ignore [bad-assignment]
        inp = ir.ExternKernel.require_contiguous(inp)
        # Because we are lowering as inplace c10d.all_reduce_, we should generate
        # _AllReduce_Kernel instead of _AllReduceKernel.
        ir._AllReduce_Kernel.create_inplace(
            c10d.all_reduce_.default,
            inp,  # type: ignore[arg-type]
            reduce_op,
            group_name,  # type: ignore[arg-type]
        )
        return inp  # type: ignore[return-value]

    @register_comm_lowering(c10d.all_reduce_)  # type: ignore[misc]
    def _all_reduce_(
        inp: ir.TensorBox,
````
- **EN**: Introduces function `_all_reduce_`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_all_reduce_`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 253-280 / 第 253-280 行
````python
        reduce_op: str,
        group_name: "torch.distributed.distributed_c10d.GroupName",
    ) -> ir.TensorBox:
        if _should_lower_as_one_shot_all_reduce(inp, reduce_op, group_name):
            ret = copy_(
                inp,
                _one_shot_all_reduce(inp, reduce_op, group_name),
            )
            mark_as_skip_wait(ret)
            return inp

        # Lower as c10d.all_reduce_
        # pyrefly: ignore [bad-assignment]
        inp = ir.ExternKernel.require_contiguous(inp)
        ir._AllReduce_Kernel.create_inplace(
            c10d.all_reduce_.default,
            inp,  # type: ignore[arg-type]
            reduce_op,
            group_name,  # type: ignore[arg-type]
        )
        return inp  # type: ignore[return-value]

    @register_comm_lowering(c10d.all_reduce_coalesced)
    def _all_reduce_coalesced(inputs, reduce_op, group_name):
        inputs = [clone(inp) for inp in inputs]
        ir._CollectiveKernel.create_inplace(
            c10d.all_reduce_coalesced_.default,
            inputs,
````
- **EN**: Introduces function `_all_reduce_coalesced`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_all_reduce_coalesced`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-308 / 第 281-308 行
````python
            reduce_op,
            group_name,
        )
        return inputs

    @register_comm_lowering(c10d.all_reduce_coalesced_)
    def _all_reduce_coalesced_(inputs, reduce_op, group_name):
        ir._CollectiveKernel.create_inplace(
            c10d.all_reduce_coalesced_.default,
            inputs,
            reduce_op,
            group_name,
        )
        return inputs

    @register_comm_lowering(c10d.all_gather_into_tensor)
    def _all_gather_into_tensor(inp, group_size, group_name):
        return _create_out_of_place(
            c10d.all_gather_into_tensor.default,
            inp,
            group_size,
            group_name,
        )

    @register_comm_lowering(c10d.all_gather_into_tensor_coalesced)
    def _all_gather_into_tensor_coalesced(inputs, group_size, group_name):
        return pytree.tree_map(
            ir.TensorBox.create,
````
- **EN**: Introduces function `_all_reduce_coalesced_`, function `_all_gather_into_tensor`, function `_all_gather_into_tensor_coalesced`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_all_reduce_coalesced_`、函数`_all_gather_into_tensor`、函数`_all_gather_into_tensor_coalesced`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 309-336 / 第 309-336 行
````python
            ir._CollectiveKernel.create_out_of_place(
                c10d.all_gather_into_tensor_coalesced.default,
                inputs,
                group_size,
                group_name,
            ),
        )

    @register_comm_lowering(c10d.all_gather_into_tensor_out)
    def _all_gather_into_tensor_out(inp, group_size, group_name, *, out):
        ir._CollectiveKernel.create_inplace(
            c10d.all_gather_into_tensor_out.default,
            inp,
            group_size,
            group_name,
            out=out,
        )
        return out

    @register_comm_lowering(c10d.reduce_scatter_tensor)
    def _reduce_scatter_tensor(inp, reduce_op, group_size, group_name):
        return _create_out_of_place(
            c10d.reduce_scatter_tensor.default,
            inp,
            reduce_op,
            group_size,
            group_name,
        )
````
- **EN**: Introduces function `_all_gather_into_tensor_out`, function `_reduce_scatter_tensor`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_all_gather_into_tensor_out`、函数`_reduce_scatter_tensor`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 337-364 / 第 337-364 行
````python

    @register_comm_lowering(c10d.reduce_scatter_tensor_out)
    def _reduce_scatter_tensor_out(inp, reduce_op, group_size, group_name, *, out):
        ir._CollectiveKernel.create_inplace(
            c10d.reduce_scatter_tensor_out.default,
            inp,
            reduce_op,
            group_size,
            group_name,
            out=out,
        )
        return out

    @register_comm_lowering(c10d.reduce_scatter_tensor_coalesced)
    def _reduce_scatter_tensor_coalesced(inputs, reduce_op, group_size, group_name):
        return pytree.tree_map(
            ir.TensorBox.create,
            ir._CollectiveKernel.create_out_of_place(
                c10d.reduce_scatter_tensor_coalesced.default,
                inputs,
                reduce_op,
                group_size,
                group_name,
            ),
        )

    @register_comm_lowering(c10d.all_to_all_single)
    def _all_to_all_single(inp, output_split_sizes, input_split_sizes, group_name):
````
- **EN**: Introduces function `_reduce_scatter_tensor_out`, function `_reduce_scatter_tensor_coalesced`, function `_all_to_all_single`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_reduce_scatter_tensor_out`、函数`_reduce_scatter_tensor_coalesced`、函数`_all_to_all_single`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 365-392 / 第 365-392 行
````python
        return _create_out_of_place(
            c10d.all_to_all_single.default,
            inp,
            output_split_sizes,
            input_split_sizes,
            group_name,
        )

    @register_comm_lowering(c10d.broadcast)
    def _broadcast(inp, src, group_name):
        inp = clone(inp)
        ir._CollectiveKernel.create_inplace(
            c10d.broadcast_.default, inp, src, group_name
        )
        return inp

    @register_comm_lowering(c10d.broadcast_)
    def _broadcast_(inp, src, group_name):
        ir._CollectiveKernel.create_inplace(
            c10d.broadcast_.default, inp, src, group_name
        )
        return inp

    @register_comm_lowering(torch.ops._dtensor.shard_dim_alltoall)
    def _shard_dim_alltoall(inp, gather_dim, shard_dim, group_name):
        return _create_out_of_place(
            torch.ops._dtensor.shard_dim_alltoall.default,
            inp,
````
- **EN**: Introduces function `_broadcast`, function `_broadcast_`, function `_shard_dim_alltoall`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_broadcast`、函数`_broadcast_`、函数`_shard_dim_alltoall`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 393-420 / 第 393-420 行
````python
            gather_dim,
            shard_dim,
            group_name,
        )

    @register_comm_lowering(c10d.wait_tensor)
    def _wait_tensor(inp):
        if should_skip_wait(inp):
            return inp

        ir._WaitKernel.create_wait(c10d.wait_tensor.default, inp)
        return inp

    @register_comm_lowering(c10d.isend)  # type: ignore[misc]
    def _isend(inp, dst, tag, group_name):
        inp = ir.ExternKernel.require_contiguous(inp)
        return _create_out_of_place(c10d.isend.default, inp, dst, tag, group_name)

    @register_comm_lowering(c10d.irecv)  # type: ignore[misc]
    def _irecv(inp, src, tag, group_name):
        inp = ir.ExternKernel.require_contiguous(inp)
        ir._CollectiveKernel.create_inplace(
            c10d.irecv.default, inp, src, tag, group_name
        )
        return inp

    @register_comm_lowering(c10d.batch_p2p_ops)  # type: ignore[misc]
    def _batch_p2p_ops(op_list, peer_list, tag_list, tensors, group_name):
````
- **EN**: Introduces function `_wait_tensor`, function `_isend`, function `_irecv`, function `_batch_p2p_ops`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_wait_tensor`、函数`_isend`、函数`_irecv`、函数`_batch_p2p_ops`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 421-448 / 第 421-448 行
````python
        tensors = [ir.ExternKernel.require_contiguous(t) for t in tensors]
        kernel = c10d.batch_p2p_ops.default
        with V.graph.fake_mode:
            (
                example_output,
                tensor_args,
                non_tensor_args,
                unflatten_args,
                unbacked_bindings,
            ) = ir._CollectiveKernel.process_kernel(
                kernel,
                op_list,
                peer_list,
                tag_list,
                tensors,
                group_name,
            )
        assert not unbacked_bindings, f"{kernel} {unbacked_bindings}"
        for op, tensor_arg in zip(op_list, tensor_args):
            tensor_arg.realize()
            if op == "irecv":
                V.graph.mark_buffer_mutated(tensor_arg.get_name())

        device = tensor_args[0].get_device()
        packed = ir._CollectiveKernel(
            ir.MultiOutputLayout(device=device),
            kernel,
            tensor_args,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tensors`, `kernel`, `device`, and `packed`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tensors`、`kernel`、`device`、`packed` 等值。

### Lines 449-476 / 第 449-476 行
````python
            non_tensor_args,
            unflatten_args,
        )

        results = []
        for i, (op, t, ex_out) in enumerate(zip(op_list, tensors, example_output)):
            if op == "irecv":
                packed.mutation_outputs.append(
                    ir.MutationOutput(ir.NoneLayout(device=device), t, packed)
                )
                packed.alias_names.append(t.get_name())
                results.append(t)
            else:
                # isend: 0-element placeholder output connected to the collective
                placeholder = ir.MultiOutput(
                    ir._CollectiveKernel.tensor_to_layout(ex_out),
                    packed,
                    [(list, i)],
                )
                results.append(ir.TensorBox.create(placeholder))
        return results


def register_symm_mem_lowerings():
    """
    Register lowerings for symmetric memory (symm_mem) operations.
    """
    try:
````
- **EN**: Introduces function `register_symm_mem_lowerings`. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`register_symm_mem_lowerings`。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 477-504 / 第 477-504 行
````python
        symm_mem = torch.ops.symm_mem
        # Check for an actual operation, not just the namespace.
        # torch.ops.symm_mem is a lazy namespace that always exists,
        # but the operations may not exist on non-CUDA platforms or
        # when USE_DISTRIBUTED is disabled.
        symm_mem.one_shot_all_reduce
    except AttributeError:
        log.info("symm_mem ops not available, skipping symm_mem lowerings")
        return

    from torch._library._out_variant import register_out_variant

    # Register manual out variant mappings for symm_mem ops.
    register_out_variant(
        symm_mem.one_shot_all_reduce.default,
        symm_mem.one_shot_all_reduce_out.default,
    )
    register_out_variant(
        symm_mem.one_shot_all_reduce_copy.default,
        symm_mem.one_shot_all_reduce_copy_out.default,
    )

    from .lowering import register_lowering

    def _copy_input_to_comm_buffer(
        inp: ir.TensorBox,
        comm_buffer_type: ir.CommBufferType,
        group_name: "torch.distributed.distributed_c10d.GroupName",
````
- **EN**: Imports dependencies such as `torch._library._out_variant`, and `.lowering` for the logic in this range. Introduces function `_copy_input_to_comm_buffer`. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `torch._library._out_variant`、`.lowering` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_copy_input_to_comm_buffer`。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 505-532 / 第 505-532 行
````python
    ) -> ir.TensorBox:
        """
        Fallback: insert a Pointwise identity copy allocated in P2P via
        CommBufferLayout.  Used when we don't control the input's allocation.
        """
        inp.realize()
        copy = ir.Pointwise.create(
            device=inp.get_device(),
            dtype=inp.get_dtype(),
            inner_fn=inp.make_loader(),
            ranges=inp.get_size(),
        )
        realize_as_comm_buffer(copy, comm_buffer_type, group_name)
        return copy

    def _maybe_realize_symm_mem(
        inp: ir.TensorBox,
        group_name: str,  # type: ignore[arg-type]
    ) -> ir.TensorBox:
        """
        Ensure inp is in P2P memory for a symm_mem collective.

        If inductor controls the buffer's allocation (ComputedBuffer,
        or any buffer with FlexibleLayout/FixedLayout), switch its
        layout to CommBufferLayout in-place, zero-copy.

        If inductor does not control allocation (e.g. InputBuffer),
        insert a Pointwise identity copy into a new CommBufferLayout buffer.
````
- **EN**: Introduces function `_maybe_realize_symm_mem`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_maybe_realize_symm_mem`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 533-560 / 第 533-560 行
````python
        This adds an extra Triton kernel. Returns the possibly new TensorBox.

        TODO(tianrengao): eliminate the extra kernel for static-shape
        InputBuffers by pre-allocating P2P memory in the wrapper and DMA .copy_()
        """
        if can_realize_as_comm_buffer(inp, ir.CommBufferType.SYMM_MEM):
            realize_as_comm_buffer(inp, ir.CommBufferType.SYMM_MEM, group_name)  # type: ignore[arg-type]
            return inp
        else:
            return _copy_input_to_comm_buffer(
                inp,
                ir.CommBufferType.SYMM_MEM,
                group_name,  # type: ignore[arg-type]
            )

    @register_lowering(symm_mem.one_shot_all_reduce)
    def _symm_mem_one_shot_all_reduce(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.one_shot_all_reduce.default,
                inp,
                reduce_op,
````
- **EN**: Introduces function `_symm_mem_one_shot_all_reduce`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_symm_mem_one_shot_all_reduce`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 561-588 / 第 561-588 行
````python
                group_name,
            ),
        )

    @register_lowering(symm_mem.one_shot_all_reduce_out)
    def _symm_mem_one_shot_all_reduce_out(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
        out: ir.TensorBox,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.one_shot_all_reduce_out.default,
                inp,
                reduce_op,
                group_name,
                out,
            ),
        )

    @register_lowering(symm_mem.one_shot_all_reduce_copy)
    def _symm_mem_one_shot_all_reduce_copy(
        symm_buffer: ir.TensorBox,
        local_input: ir.TensorBox,
        reduce_op: str,
````
- **EN**: Introduces function `_symm_mem_one_shot_all_reduce_out`, function `_symm_mem_one_shot_all_reduce_copy`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_one_shot_all_reduce_out`、函数`_symm_mem_one_shot_all_reduce_copy`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 589-616 / 第 589-616 行
````python
        group_name: str,
    ):
        symm_buffer = _maybe_realize_symm_mem(symm_buffer, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.one_shot_all_reduce_copy.default,
                symm_buffer,
                local_input,
                reduce_op,
                group_name,
            ),
        )

    @register_lowering(symm_mem.one_shot_all_reduce_copy_out)
    def _symm_mem_one_shot_all_reduce_copy_out(
        symm_buffer: ir.TensorBox,
        local_input: ir.TensorBox,
        reduce_op: str,
        group_name: str,
        out: ir.TensorBox,
    ):
        symm_buffer = _maybe_realize_symm_mem(symm_buffer, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.one_shot_all_reduce_copy_out.default,
                symm_buffer,
````
- **EN**: Introduces function `_symm_mem_one_shot_all_reduce_copy_out`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_one_shot_all_reduce_copy_out`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 617-644 / 第 617-644 行
````python
                local_input,
                reduce_op,
                group_name,
                out,
            ),
        )

    @register_lowering(symm_mem.two_shot_all_reduce_)
    def _symm_mem_two_shot_all_reduce_(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        ir.FallbackKernel.create(
            symm_mem.two_shot_all_reduce_.default,
            inp,
            reduce_op,
            group_name,
        )
        return inp

    @register_lowering(symm_mem.two_shot_all_reduce_out)
    def _symm_mem_two_shot_all_reduce_out(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
        output: ir.TensorBox,
````
- **EN**: Introduces function `_symm_mem_two_shot_all_reduce_`, function `_symm_mem_two_shot_all_reduce_out`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_two_shot_all_reduce_`、函数`_symm_mem_two_shot_all_reduce_out`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 645-672 / 第 645-672 行
````python
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.two_shot_all_reduce_out.default,
                inp,
                reduce_op,
                group_name,
                output,
            ),
        )

    @register_lowering(symm_mem.multimem_all_reduce_)
    def _symm_mem_multimem_all_reduce_(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        ir.FallbackKernel.create(
            symm_mem.multimem_all_reduce_.default,
            inp,
            reduce_op,
            group_name,
        )
        return inp

````
- **EN**: Introduces function `_symm_mem_multimem_all_reduce_`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_multimem_all_reduce_`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 673-700 / 第 673-700 行
````python
    @register_lowering(symm_mem.multimem_one_shot_all_reduce)
    def _symm_mem_multimem_one_shot_all_reduce(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.multimem_one_shot_all_reduce.default,
                inp,
                reduce_op,
                group_name,
            ),
        )

    @register_lowering(symm_mem.multimem_one_shot_all_reduce_out)
    def _symm_mem_multimem_one_shot_all_reduce_out(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
        out: ir.TensorBox,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
````
- **EN**: Introduces function `_symm_mem_multimem_one_shot_all_reduce`, function `_symm_mem_multimem_one_shot_all_reduce_out`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_multimem_one_shot_all_reduce`、函数`_symm_mem_multimem_one_shot_all_reduce_out`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 701-728 / 第 701-728 行
````python
                symm_mem.multimem_one_shot_all_reduce_out.default,
                inp,
                reduce_op,
                group_name,
                out,
            ),
        )

    @register_lowering(symm_mem.multimem_one_shot_reduce_out)
    def _symm_mem_multimem_one_shot_reduce_out(
        inp: ir.TensorBox,
        reduce_op: str,
        root: int,
        group_name: str,
        out: ir.TensorBox,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.multimem_one_shot_reduce_out.default,
                inp,
                reduce_op,
                root,
                group_name,
                out,
            ),
        )
````
- **EN**: Introduces function `_symm_mem_multimem_one_shot_reduce_out`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_multimem_one_shot_reduce_out`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 729-756 / 第 729-756 行
````python

    @register_lowering(symm_mem.multimem_all_gather_out)
    def _symm_mem_multimem_all_gather_out(
        inp: ir.TensorBox,
        group_name: str,
        out: ir.TensorBox,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
            ir.FallbackKernel.create(
                symm_mem.multimem_all_gather_out.default,
                inp,
                group_name,
                out,
            ),
        )

    @register_lowering(symm_mem.reduce_scatter_out)
    def _symm_mem_reduce_scatter_out(
        inp: ir.TensorBox,
        group_name: str,
        split_last_dim: bool,
        output: ir.TensorBox,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        return pytree.tree_map(
            ir.TensorBox.create,
````
- **EN**: Introduces function `_symm_mem_multimem_all_gather_out`, function `_symm_mem_reduce_scatter_out`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_multimem_all_gather_out`、函数`_symm_mem_reduce_scatter_out`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 757-784 / 第 757-784 行
````python
            ir.FallbackKernel.create(
                symm_mem.reduce_scatter_out.default,
                inp,
                group_name,
                split_last_dim,
                output,
            ),
        )

    @register_lowering(symm_mem.all_to_all_vdev)
    def _symm_mem_all_to_all_vdev(
        inp: ir.TensorBox,
        out: ir.TensorBox,
        in_splits: ir.TensorBox,
        out_splits_offsets: ir.TensorBox,
        group_name: str,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        out = _maybe_realize_symm_mem(out, group_name)
        ir.FallbackKernel.create(
            symm_mem.all_to_all_vdev.default,
            inp,
            out,
            in_splits,
            out_splits_offsets,
            group_name,
        )
        return None
````
- **EN**: Introduces function `_symm_mem_all_to_all_vdev`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_all_to_all_vdev`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 785-812 / 第 785-812 行
````python

    @register_lowering(symm_mem.all_to_all_vdev_2d)
    def _symm_mem_all_to_all_vdev_2d(
        inp: ir.TensorBox,
        out: ir.TensorBox,
        in_splits: ir.TensorBox,
        out_splits_offsets: ir.TensorBox,
        group_name: str,
        major_align=None,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        out = _maybe_realize_symm_mem(out, group_name)
        ir.FallbackKernel.create(
            symm_mem.all_to_all_vdev_2d.default,
            inp,
            out,
            in_splits,
            out_splits_offsets,
            group_name,
            major_align,
        )
        return None

    @register_lowering(symm_mem.all_to_all_vdev_2d_offset)
    def _symm_mem_all_to_all_vdev_2d_offset(
        inp: ir.TensorBox,
        out: ir.TensorBox,
        in_splits_offsets: ir.TensorBox,
````
- **EN**: Introduces function `_symm_mem_all_to_all_vdev_2d`, function `_symm_mem_all_to_all_vdev_2d_offset`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_all_to_all_vdev_2d`、函数`_symm_mem_all_to_all_vdev_2d_offset`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 813-840 / 第 813-840 行
````python
        out_splits_offsets: ir.TensorBox,
        group_name: str,
    ):
        inp = _maybe_realize_symm_mem(inp, group_name)
        out = _maybe_realize_symm_mem(out, group_name)
        ir.FallbackKernel.create(
            symm_mem.all_to_all_vdev_2d_offset.default,
            inp,
            out,
            in_splits_offsets,
            out_splits_offsets,
            group_name,
        )
        return None

    @register_lowering(symm_mem.tile_reduce)
    def _symm_mem_tile_reduce(
        in_tile: ir.TensorBox,
        out_tile: ir.TensorBox,
        root: int,
        group_name: str,
        reduce_op: str = "sum",
    ):
        in_tile = _maybe_realize_symm_mem(in_tile, group_name)
        out_tile = _maybe_realize_symm_mem(out_tile, group_name)
        ir.FallbackKernel.create(
            symm_mem.tile_reduce.default,
            in_tile,
````
- **EN**: Introduces function `_symm_mem_tile_reduce`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_tile_reduce`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 841-868 / 第 841-868 行
````python
            out_tile,
            root,
            group_name,
            reduce_op,
        )
        return None

    @register_lowering(symm_mem.multi_root_tile_reduce)
    def _symm_mem_multi_root_tile_reduce(
        in_tiles,  # list of TensorBox
        out_tile: ir.TensorBox,
        roots,  # list of int
        group_name: str,
        reduce_op: str = "sum",
    ):
        for i, in_tile in enumerate(in_tiles):
            in_tiles[i] = _maybe_realize_symm_mem(in_tile, group_name)
        out_tile = _maybe_realize_symm_mem(out_tile, group_name)
        ir.FallbackKernel.create(
            symm_mem.multi_root_tile_reduce.default,
            in_tiles,
            out_tile,
            roots,
            group_name,
            reduce_op,
        )
        return None

````
- **EN**: Introduces function `_symm_mem_multi_root_tile_reduce`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_multi_root_tile_reduce`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 869-896 / 第 869-896 行
````python
    @register_lowering(symm_mem._low_contention_all_gather)
    def _symm_mem_low_contention_all_gather(
        inp: ir.TensorBox,
        group_name: str,
    ):
        # Use _CollectiveKernel so that _WaitKernel.get_volatile_reads()
        # can track the input's lifetime through wait_tensor, preventing
        # the memory planner from reusing the input buffer while the
        # backend stream is still reading it.
        return _create_out_of_place(
            symm_mem._low_contention_all_gather.default,
            inp,
            group_name,
        )

    @register_lowering(symm_mem._low_contention_reduce_scatter)
    def _symm_mem_low_contention_reduce_scatter(
        inp: ir.TensorBox,
        reduce_op: str,
        group_name: str,
    ):
        # Use _CollectiveKernel so that _WaitKernel.get_volatile_reads()
        # can track the input's lifetime through wait_tensor, preventing
        # the memory planner from reusing the input buffer while the
        # backend stream is still reading it.
        return _create_out_of_place(
            symm_mem._low_contention_reduce_scatter.default,
            inp,
````
- **EN**: Introduces function `_symm_mem_low_contention_all_gather`, function `_symm_mem_low_contention_reduce_scatter`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_symm_mem_low_contention_all_gather`、函数`_symm_mem_low_contention_reduce_scatter`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 897-899 / 第 897-899 行
````python
            reduce_op,
            group_name,
        )
````
- **EN**: This range continues the implementation of function `register_symm_mem_lowerings._symm_mem_low_contention_reduce_scatter`.
- **CN**: 这一段延续了函数`register_symm_mem_lowerings._symm_mem_low_contention_reduce_scatter` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `can_realize_as_comm_buffer`, `realize_as_comm_buffer`, `_get_data`, `mark_as_skip_wait`, `should_skip_wait`, `_should_lower_as_one_shot_all_reduce`, and `...+4`  
  **CN**: 主要函数：`can_realize_as_comm_buffer`、`realize_as_comm_buffer`、`_get_data`、`mark_as_skip_wait`、`should_skip_wait`、`_should_lower_as_one_shot_all_reduce`、`另有4项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._pytree`, `torch._inductor.utils`, `torch.utils._ordered_set`, `.`, `.virtualized`, `torch.distributed._symmetric_memory`, `.lowering`, `torch._library._out_variant`
