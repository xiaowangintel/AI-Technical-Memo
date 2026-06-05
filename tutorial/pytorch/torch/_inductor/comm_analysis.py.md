# comm_analysis.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/comm_analysis.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `NCCL_COLL`, `NVIDIA_GPU_TYPE`, `NCCL_HW`, `NCCL_ALGO`, and `NCCL_PROTO`. It exposes functions such as `get_gpu_type`, `get_collective_type_from_kernel_name`, `get_collective_type`, `get_ir_node_size_numel`, `get_fx_node_size_numel`, `get_collective_input_size_bytes`, and `...+7`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `NCCL_COLL`、`NVIDIA_GPU_TYPE`、`NCCL_HW`、`NCCL_ALGO`、`NCCL_PROTO` 等类。同时提供 `get_gpu_type`、`get_collective_type_from_kernel_name`、`get_collective_type`、`get_ir_node_size_numel`、`get_fx_node_size_numel`、`get_collective_input_size_bytes`、`另有7项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import functools
import logging
import math
import operator
from enum import IntEnum
from typing import Any

import sympy

import torch
import torch.utils._pytree as pytree
from torch.fx.experimental.symbolic_shapes import optimization_hint
from torch.fx.operator_schemas import normalize_function

from . import ir
from .utils import get_dtype_size, snode_args_kwargs, sympy_product
from .virtualized import V


log = logging.getLogger(__name__)
````
- **EN**: Imports dependencies such as `functools`, `logging`, `math`, `operator`, `enum`, `typing`, and `...+8` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `functools`、`logging`、`math`、`operator`、`enum`、`typing`、`另有8项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `log` 等值。

### Lines 21-40 / 第 21-40 行
````python


class NCCL_COLL(IntEnum):
    ALL_REDUCE = 0
    ALL_GATHER = 1
    REDUCE_SCATTER = 2
    ALL_TO_ALL = 3
    UNSUPPORTED = 4
    P2P = 5


class NVIDIA_GPU_TYPE(IntEnum):
    VOLTA = 0
    AMPERE = 1
    HOPPER = 2
    BLACKWELL = 3


@functools.lru_cache
def get_gpu_type() -> NVIDIA_GPU_TYPE:
````
- **EN**: Introduces class `NCCL_COLL`, class `NVIDIA_GPU_TYPE`, function `get_gpu_type`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Initializes or updates values such as `ALL_REDUCE`, `ALL_GATHER`, `REDUCE_SCATTER`, `ALL_TO_ALL`, `UNSUPPORTED`, `P2P`, and `...+4`.
- **CN**: 这里定义了类`NCCL_COLL`、类`NVIDIA_GPU_TYPE`、函数`get_gpu_type`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。初始化或更新了 `ALL_REDUCE`、`ALL_GATHER`、`REDUCE_SCATTER`、`ALL_TO_ALL`、`UNSUPPORTED`、`P2P`、`另有4项` 等值。

### Lines 41-60 / 第 41-60 行
````python
    gpu_info = torch.utils.collect_env.get_gpu_info(torch.utils.collect_env.run) or ""
    if "V100" in gpu_info:
        return NVIDIA_GPU_TYPE.VOLTA
    elif "A100" in gpu_info:
        return NVIDIA_GPU_TYPE.AMPERE
    elif "H100" in gpu_info:
        return NVIDIA_GPU_TYPE.HOPPER
    elif any(gpu in gpu_info for gpu in ("B100", "B200", "B300")):
        return NVIDIA_GPU_TYPE.BLACKWELL
    else:
        # for other gpu types, assume Ampere
        return NVIDIA_GPU_TYPE.AMPERE


def get_collective_type_from_kernel_name(kernel_name: str) -> NCCL_COLL:
    assert kernel_name is not None
    if "all_reduce" in kernel_name:
        return NCCL_COLL.ALL_REDUCE
    elif "all_gather" in kernel_name:
        return NCCL_COLL.ALL_GATHER
````
- **EN**: Introduces function `get_collective_type_from_kernel_name`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_collective_type_from_kernel_name`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 61-80 / 第 61-80 行
````python
    elif "reduce_scatter" in kernel_name:
        return NCCL_COLL.REDUCE_SCATTER
    elif any(comm in kernel_name for comm in ("all_to_all", "alltoall")):
        return NCCL_COLL.ALL_TO_ALL
    elif any(comm in kernel_name for comm in ("isend", "irecv", "batch_p2p")):
        return NCCL_COLL.P2P
    else:
        return NCCL_COLL.UNSUPPORTED


def get_collective_type(node: ir.IRNode) -> NCCL_COLL:
    if not isinstance(node, ir._CollectiveKernel):
        raise ValueError(f"node is not a collective kernel: {node}")

    name = node.python_kernel_name
    assert name is not None
    return get_collective_type_from_kernel_name(name)


def get_ir_node_size_numel(size: torch.Size, fallback: int = 4096 * 4096) -> int:
````
- **EN**: Introduces function `get_collective_type`, function `get_ir_node_size_numel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `name`.
- **CN**: 这里定义了函数`get_collective_type`、函数`get_ir_node_size_numel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`name` 等值。

### Lines 81-100 / 第 81-100 行
````python
    numel = sympy_product(size)
    if isinstance(numel, sympy.Integer):
        return int(numel)
    return V.graph.sizevars.optimization_hint(numel, fallback=fallback)


def get_fx_node_size_numel(size: torch.Size, fallback: int = 4096 * 4096) -> int:
    numel = functools.reduce(operator.mul, size, 1)
    result = optimization_hint(numel, fallback=fallback)
    return result


def get_collective_input_size_bytes(node: ir.IRNode) -> int:
    sz_bytes = 0
    for inp in node.inputs:  # type: ignore[attr-defined]
        numel = get_ir_node_size_numel(inp.layout.size)
        sz_bytes += numel * get_dtype_size(inp.layout.dtype)
    return sz_bytes


````
- **EN**: Introduces function `get_fx_node_size_numel`, function `get_collective_input_size_bytes`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_fx_node_size_numel`、函数`get_collective_input_size_bytes`。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
def get_collective_group_size(node: ir.IRNode) -> int:
    if isinstance(node, ir._CollectiveKernel) and not isinstance(node, ir._WaitKernel):
        from torch.distributed.distributed_c10d import _get_group_size_by_name

        return _get_group_size_by_name(node.constant_args[-1])
    else:
        raise TypeError(f"Unsupported collective type: {node}")


####################################################################################################################
# The following code and constants are adapted from https://github.com/NVIDIA/nccl/blob/master/src/graph/tuning.cc #
####################################################################################################################


class NCCL_HW(IntEnum):
    NVLINK = 0
    PCI = 1
    NET = 2


````
- **EN**: Imports dependencies such as `torch.distributed.distributed_c10d` for the logic in this range. Introduces function `get_collective_group_size`, class `NCCL_HW`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch.distributed.distributed_c10d` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_collective_group_size`、类`NCCL_HW`。保留了指向设计说明、规范或厂商数据手册的注释引用。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python
class NCCL_ALGO(IntEnum):
    TREE = 0
    RING = 1


class NCCL_PROTO(IntEnum):
    # The ordering and enum values here matches original in
    # https://github.com/NVIDIA/nccl/blob/0b083e52096c387bad7a5c5c65b26a9dca54de8c/src/include/devcomm.h#L28
    # For difference between these protocols, see https://github.com/NVIDIA/nccl/issues/281#issuecomment-571816990
    LL = 0  # Low-latency
    # LL128 = 1   # Low-latency 128-byte
    # SIMPLE = 2


# Latencies in us
# len(NCCL_ALGO) x len(NCCL_PROTO)
# NOTE: use array instead of tensor to prevent incompatibility with fake mode
baseLat = [
    # Tree
    [
````
- **EN**: Introduces class `NCCL_ALGO`, class `NCCL_PROTO`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Initializes or updates values such as `TREE`, `RING`, `LL`, and `baseLat`.
- **CN**: 这里定义了类`NCCL_ALGO`、类`NCCL_PROTO`。保留了指向设计说明、规范或厂商数据手册的注释引用。初始化或更新了 `TREE`、`RING`、`LL`、`baseLat` 等值。

### Lines 141-160 / 第 141-160 行
````python
        6.8,  # LL
    ],
    # Ring
    [
        6.6,  # LL
    ],
]

# Latencies in us
# len(NCCL_HW) x len(NCCL_ALGO) x len(NCCL_PROTO)
hwLat = [
    # NVLINK
    [
        [0.6],  # Tree (LL)
        [0.6],  # Ring (LL)
    ],
    # PCI
    [
        [1.0],  # Tree (LL)
        [1.0],  # Ring (LL)
````
- **EN**: Initializes or updates values such as `hwLat`.
- **CN**: 初始化或更新了 `hwLat` 等值。

### Lines 161-180 / 第 161-180 行
````python
    ],
    # NET
    [
        [5.0],  # Tree (LL)
        [2.7],  # Ring (LL)
    ],
]


# LL128 max BW per channel
llMaxBws = [
    # Volta-N1/Intel-N2/Intel-N4
    [
        39.0,
        39.0,
        20.4,
    ],
    # Ampere-N1/AMD-N2/AMD-N4
    [
        87.7,
````
- **EN**: Initializes or updates values such as `llMaxBws`.
- **CN**: 初始化或更新了 `llMaxBws` 等值。

### Lines 181-200 / 第 181-200 行
````python
        22.5,  # avg of ring & tree
        19.0,
    ],
    # Hopper-N1/AMD-N2/AMD-N4
    [
        141.0,
        45.0,  # avg of ring & tree
        35.0,
    ],
    # Blackwell-N1/AMD-N2/AMD-N4
    [
        282.0,
        90.0,  # avg of ring & tree
        70.0,
    ],
]


def estimate_nccl_collective_runtime_nccl_estimator(snode) -> float | None:  # type: ignore[no-untyped-def]
    kernel = snode.node
````
- **EN**: Introduces function `estimate_nccl_collective_runtime_nccl_estimator`. Initializes or updates values such as `kernel`.
- **CN**: 这里定义了函数`estimate_nccl_collective_runtime_nccl_estimator`。初始化或更新了 `kernel` 等值。

### Lines 201-220 / 第 201-220 行
````python
    assert kernel is not None
    py_kernel_name = getattr(kernel, "python_kernel_name", "")
    pg_name = kernel.constant_args[-1]  # type: ignore[attr-defined]
    from torch.distributed.distributed_c10d import _resolve_process_group

    pg = _resolve_process_group(pg_name)
    rank: int = torch.distributed.get_rank(pg)
    # TODO(ivankobzarev): Figure out how we can use time estimations,
    # without cuda allocations.
    device = torch.device(f"cuda:{rank}")

    fn = eval(py_kernel_name)
    args, kwargs = snode_args_kwargs(snode)

    # TODO(ivankobzarev): fix out variants snode_args_kwargs
    if "all_gather_into_tensor_out" in py_kernel_name:
        args = args[1:] + args[0]

    with torch.distributed._time_estimator(group=pg, device=device) as time_estimator:
        w = fn(*args, **kwargs)
````
- **EN**: Imports dependencies such as `torch.distributed.distributed_c10d` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch.distributed.distributed_c10d` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
        torch.ops._c10d_functional.wait_tensor.default(w)

    est_time_us = time_estimator.estimated_time
    # -1000 constant is NCCL return in case of error during estimations.
    # Observed it for all_to_all estimations.
    if est_time_us < 0:
        return None
    est_time_ms = est_time_us / 1e3
    return est_time_ms


def estimate_nccl_collective_runtime_impl(
    tensor_storage_size_bytes: int, group_size: int, coll: NCCL_COLL
) -> float:
    """
    Returns estimated NCCL collective runtime in milliseconds (ms).

    The following heuristics are copied from https://github.com/NVIDIA/nccl/blob/master/src/graph/tuning.cc.
    We aim to estimate the runtime as accurately as possible.

````
- **EN**: Introduces function `estimate_nccl_collective_runtime_impl`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`estimate_nccl_collective_runtime_impl`。保留了指向设计说明、规范或厂商数据手册的注释引用。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 241-260 / 第 241-260 行
````python
    Assumptions:
    - only ring algorithm (NCCL_ALGO_RING) is used
    - only Low-Latency protocol (NCCL_PROTO_LL) is used, i.e. Simple or LL128 is not used
    - 8 gpus per node  # TODO: Need to find a way to get accurate "gpus per node" and "# nodes" info.
    - collective is one of: allreduce, reducescatter, allgather
    """
    # Convert bytes to GB
    tensor_storage_size_GB = tensor_storage_size_bytes / 1024 / 1024 / 1024

    # Currently assumes each node has 8 gpus. And when >1 node is used, assumes each node uses all 8 gpus.
    # TODO: Need to find a way to get accurate "gpus per node" and "# nodes" info.
    num_gpus_per_node = 8
    nNodes = math.ceil(group_size / num_gpus_per_node)
    nRanks = group_size  # this is total # of gpus globally that participate in this collective op

    if nRanks <= 1:
        return 0

    if coll == NCCL_COLL.UNSUPPORTED:
        return 0
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Assumptions`, `tensor_storage_size_GB`, `num_gpus_per_node`, `nNodes`, and `nRanks`. This range continues the implementation of function `estimate_nccl_collective_runtime_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Assumptions`、`tensor_storage_size_GB`、`num_gpus_per_node`、`nNodes`、`nRanks` 等值。这一段延续了函数`estimate_nccl_collective_runtime_impl` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python

    # Assumes ring algorithm
    nccl_algo = NCCL_ALGO.RING
    nccl_proto = NCCL_PROTO.LL

    # =============== bandwidth computation ===============
    # First compute bandwidth in GB/s; then at the end, convert it to GB/ns

    bwIntra = torch._inductor.config.intra_node_bw
    bwInter = torch._inductor.config.inter_node_bw

    compCapIndex = get_gpu_type()
    index2 = nNodes - 1 if nNodes <= 2 else 2
    # LL: for single node, we look at GPU type; for multi-node, we look at CPU type
    index1 = compCapIndex if nNodes == 1 else 0
    llMaxBw = llMaxBws[index1][index2]

    # NOTE: each step of ring algorithm is synchronized,
    # and is bottlenecked by the slowest link which is the inter-node interconnect.
    # hence when nNodes >= 2, bw is inter-node bandwidth.
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `nccl_algo`, `nccl_proto`, `bwIntra`, `bwInter`, `compCapIndex`, `index2`, and `...+2`. This range continues the implementation of function `estimate_nccl_collective_runtime_impl`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `nccl_algo`、`nccl_proto`、`bwIntra`、`bwInter`、`compCapIndex`、`index2`、`另有2项` 等值。这一段延续了函数`estimate_nccl_collective_runtime_impl` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
    # NOTE: the original code in https://github.com/NVIDIA/nccl/blob/master/src/graph/tuning.cc
    # have this as `if nNodes <= 2` which seems wrong. Corrected it here.
    bw = bwIntra if nNodes == 1 else bwInter
    nChannels = 2  # Assume # channels is 2
    busBw = nChannels * bw

    # Various model refinements
    busBw = min(
        llMaxBw,
        busBw
        * (1.0 / 4.0 if (nNodes > 1 or coll == NCCL_COLL.ALL_REDUCE) else 1.0 / 3.0),
    )

    if coll == NCCL_COLL.ALL_REDUCE:
        nsteps = 2 * (nRanks - 1)
    elif coll == NCCL_COLL.ALL_TO_ALL:
        nsteps = 2 * (nRanks - 1)
    elif coll in (NCCL_COLL.REDUCE_SCATTER, NCCL_COLL.ALL_GATHER):
        nsteps = nRanks - 1
    elif coll == NCCL_COLL.P2P:
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bw`, `nChannels`, `busBw`, and `nsteps`. This range continues the implementation of function `estimate_nccl_collective_runtime_impl`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。包含分支、循环或上下文管理等控制流。初始化或更新了 `bw`、`nChannels`、`busBw`、`nsteps` 等值。这一段延续了函数`estimate_nccl_collective_runtime_impl` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
        # assume 1 hop per pair
        nsteps = 1

    # Convert bus BW to algorithm BW (tensor bytes / algoBW = actual execution time)
    ratio = (1.0 * nRanks) / nsteps  # type: ignore[possibly-undefined]
    bandwidth = busBw * ratio
    # Convert GB/s to GB/ns
    bandwidth_GB_per_ns = bandwidth / 1e9

    # =============== latency computation ===============
    intraHw = NCCL_HW.NVLINK

    if coll == NCCL_COLL.ALL_REDUCE:
        if nNodes > 1:
            nInterSteps = 2 * nNodes
        else:
            nInterSteps = 0
    elif coll in (NCCL_COLL.REDUCE_SCATTER, NCCL_COLL.ALL_GATHER, NCCL_COLL.ALL_TO_ALL):
        nInterSteps = nNodes - 1
    elif coll == NCCL_COLL.P2P:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nsteps`, `ratio`, `bandwidth`, `bandwidth_GB_per_ns`, `intraHw`, `nInterSteps`, and `...+1`. This range continues the implementation of function `estimate_nccl_collective_runtime_impl`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `nsteps`、`ratio`、`bandwidth`、`bandwidth_GB_per_ns`、`intraHw`、`nInterSteps`、`另有1项` 等值。这一段延续了函数`estimate_nccl_collective_runtime_impl` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
        # p2p may cross node bdry
        nInterSteps = 1 if nNodes > 1 else 0

    # First compute latency in us; then at the end, convert it to ns
    latency = baseLat[nccl_algo][nccl_proto]
    intraLat = hwLat[intraHw][nccl_algo][nccl_proto]
    interLat = hwLat[NCCL_HW.NET][nccl_algo][nccl_proto]

    # Inter-node rings still have to launch nsteps * net overhead.
    netOverhead = 0.0
    if nNodes > 1:
        netOverhead = 1.0  # getNetOverhead(comm);
    intraLat = max(intraLat, netOverhead)
    latency += (nsteps - nInterSteps) * intraLat + nInterSteps * interLat  # type: ignore[possibly-undefined]
    # Convert us to ns
    latency_ns = latency * 1e3

    # =============== final result ===============
    transport_ns = tensor_storage_size_GB / bandwidth_GB_per_ns
    ns = transport_ns + latency_ns
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nInterSteps`, `latency`, `intraLat`, `interLat`, `netOverhead`, `latency_ns`, and `...+2`. This range continues the implementation of function `estimate_nccl_collective_runtime_impl`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `nInterSteps`、`latency`、`intraLat`、`interLat`、`netOverhead`、`latency_ns`、`另有2项` 等值。这一段延续了函数`estimate_nccl_collective_runtime_impl` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
    ms = ns / 1e6
    return ms


################################################################################################################
# The above code and constants are adapted from https://github.com/NVIDIA/nccl/blob/master/src/graph/tuning.cc #
################################################################################################################


def estimate_nccl_collective_runtime(node: ir.IRNode) -> float:
    """
    Returns estimated NCCL collective runtime in nanoseconds (ms).

    The following heuristics are copied from https://github.com/NVIDIA/nccl/blob/master/src/graph/tuning.cc.
    We aim to estimate the runtime as accurately as possible.

    Assumptions:
    - only ring algorithm (NCCL_ALGO_RING) is used
    - only Low-Latency protocol (NCCL_PROTO_LL) is used, i.e. Simple or LL128 is not used
    - 8 gpus per node  # TODO: Need to find a way to get accurate "gpus per node" and "# nodes" info.
````
- **EN**: Introduces function `estimate_nccl_collective_runtime`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `ms`, and `Assumptions`.
- **CN**: 这里定义了函数`estimate_nccl_collective_runtime`。保留了指向设计说明、规范或厂商数据手册的注释引用。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `ms`、`Assumptions` 等值。

### Lines 361-380 / 第 361-380 行
````python
    - collective is one of: allreduce, reducescatter, allgather
    """
    tensor_storage_size_bytes = get_collective_input_size_bytes(node)
    group_size = get_collective_group_size(node)
    coll = get_collective_type(node)
    return estimate_nccl_collective_runtime_impl(
        tensor_storage_size_bytes, group_size, coll
    )


def estimate_fx_collective_size(fx_node: torch.fx.Node) -> int:
    """Estimate the size of a collective operation in bytes, including inputs and outputs."""
    input_bytes = None

    args, kwargs = fx_node.args, fx_node.kwargs
    kwargs = dict(kwargs)

    # dont double count pre-allocated buffer passed in
    kwargs.pop("out", None)

````
- **EN**: Introduces function `estimate_fx_collective_size`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `tensor_storage_size_bytes`, `group_size`, `coll`, `input_bytes`, and `kwargs`.
- **CN**: 这里定义了函数`estimate_fx_collective_size`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `tensor_storage_size_bytes`、`group_size`、`coll`、`input_bytes`、`kwargs` 等值。

### Lines 381-400 / 第 381-400 行
````python
    def tensor_bytes(t: torch.Tensor) -> int:
        return get_fx_node_size_numel(t.size()) * get_dtype_size(t.dtype)

    def add_inp_bytes(inp: torch.fx.Node):
        inp_val = inp.meta.get("val", None)
        if not isinstance(inp_val, torch.Tensor):
            return

        nonlocal input_bytes
        if input_bytes is None:
            input_bytes = 0
        input_bytes += tensor_bytes(inp_val)

    pytree.tree_map_only(
        torch.fx.Node,
        add_inp_bytes,
        (args, kwargs),
    )

    output_val = fx_node.meta.get("val", None)
````
- **EN**: Introduces function `tensor_bytes`, function `add_inp_bytes`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`tensor_bytes`、函数`add_inp_bytes`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 401-420 / 第 401-420 行
````python

    if input_bytes is None or output_val is None:
        return 0

    # Coalesced collectives return a list of tensors
    if isinstance(output_val, (list, tuple)):
        output_bytes = sum(
            tensor_bytes(t) for t in output_val if isinstance(t, torch.Tensor)
        )
    elif isinstance(output_val, torch.Tensor):
        output_bytes = tensor_bytes(output_val)
    else:
        return 0

    return input_bytes + output_bytes


def estimate_fx_collective_memory_footprint(fx_node: torch.fx.Node) -> int:
    """Estimate the memory footprint of a collective operation in bytes.

````
- **EN**: Introduces function `estimate_fx_collective_memory_footprint`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_bytes`, and `else`.
- **CN**: 这里定义了函数`estimate_fx_collective_memory_footprint`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_bytes`、`else` 等值。

### Lines 421-440 / 第 421-440 行
````python
    This returns the total bytes that need to be live concurrently in memory.
    For all_reduce, we divide by 2 since it can be done in-place.
    """
    from torch._inductor.fx_passes.bucketing import (
        is_all_reduce_tensor as is_all_reduce,
    )

    size = estimate_fx_collective_size(fx_node)
    return size if not is_all_reduce(fx_node) else size // 2


def estimate_nccl_collective_runtime_from_fx_node(
    fx_node: torch.fx.Node,
    override_size: int | None = None,
    use_nccl_estimator: bool = True,
) -> float:
    """
    Returns estimated NCCL collective runtime in nanoseconds (ms).

    The following heuristics are copied from https://github.com/NVIDIA/nccl/blob/master/src/graph/tuning.cc.
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.bucketing` for the logic in this range. Introduces function `estimate_nccl_collective_runtime_from_fx_node`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.fx_passes.bucketing` 等依赖，为后续逻辑提供基础能力。这里定义了函数`estimate_nccl_collective_runtime_from_fx_node`。保留了指向设计说明、规范或厂商数据手册的注释引用。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 441-460 / 第 441-460 行
````python
    We aim to estimate the runtime as accurately as possible.

    Assumptions:
    - only ring algorithm (NCCL_ALGO_RING) is used
    - only Low-Latency protocol (NCCL_PROTO_LL) is used, i.e. Simple or LL128 is not used
    - 8 gpus per node  # TODO: Need to find a way to get accurate "gpus per node" and "# nodes" info.
    - collective is one of: allreduce, reducescatter, allgather
    """
    from torch.distributed.distributed_c10d import _get_group_size_by_name

    if fx_node.target is torch.ops._c10d_functional.all_to_all_single.default:
        # TODO(ivankobzarev): Temporarily disabled - NCCL estimator returns internal error.
        # for all_to_all during inductor compilation. Falls back to heuristic estimation.
        use_nccl_estimator = False

    if override_size is None:
        tensor_storage_size_bytes = estimate_fx_collective_size(fx_node)
    else:
        tensor_storage_size_bytes = override_size

````
- **EN**: Imports dependencies such as `torch.distributed.distributed_c10d` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Assumptions`, `use_nccl_estimator`, `tensor_storage_size_bytes`, and `else`. This range continues the implementation of function `estimate_nccl_collective_runtime_from_fx_node`.
- **CN**: 这里导入了 `torch.distributed.distributed_c10d` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `Assumptions`、`use_nccl_estimator`、`tensor_storage_size_bytes`、`else` 等值。这一段延续了函数`estimate_nccl_collective_runtime_from_fx_node` 的具体实现。

### Lines 461-480 / 第 461-480 行
````python
    assert not isinstance(fx_node.target, str)
    opt_args_kwargs = normalize_function(
        fx_node.target,
        args=fx_node.args,
        kwargs=fx_node.kwargs,
        normalize_to_only_use_kwargs=True,
    )
    assert opt_args_kwargs is not None
    args, kwargs = opt_args_kwargs

    group_name = kwargs["group_name"]
    group_size = _get_group_size_by_name(group_name)
    assert isinstance(fx_node.target, torch._ops.OpOverload)
    coll = get_collective_type_from_kernel_name(fx_node.target.name())

    def _nccl_estimate() -> float | None:
        # TODO: Refactor with estimate_nccl_collective_runtime_nccl_estimator
        from torch.distributed.distributed_c10d import _resolve_process_group, Backend

        pg = _resolve_process_group(group_name)
````
- **EN**: Imports dependencies such as `torch.distributed.distributed_c10d` for the logic in this range. Introduces function `_nccl_estimate`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `opt_args_kwargs`, `args`, `kwargs`, `normalize_to_only_use_kwargs`, `group_name`, `group_size`, and `...+2`.
- **CN**: 这里导入了 `torch.distributed.distributed_c10d` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_nccl_estimate`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `opt_args_kwargs`、`args`、`kwargs`、`normalize_to_only_use_kwargs`、`group_name`、`group_size`、`另有2项` 等值。

### Lines 481-500 / 第 481-500 行
````python
        if torch.distributed.distributed_c10d.get_backend(pg) == Backend.FAKE:
            # nccl estimator requires real process group
            return None

        device = torch.device("cuda")
        try:
            backend = pg._get_backend(device)
        except RuntimeError:
            return None
        if not backend.supports_time_estimate:
            return None

        flat_args, flat_args_pytree_spec = pytree.tree_flatten((args, kwargs))

        def _tensor(size, dtype, device) -> torch.Tensor:  # type: ignore[no-untyped-def]
            return torch.empty(
                size if override_size is None else [override_size],
                dtype=dtype,
                device=device,
            )
````
- **EN**: Introduces function `_tensor`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_tensor`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 501-520 / 第 501-520 行
````python

        def to_real_tensor(e: Any) -> Any:
            if isinstance(e, torch.fx.Node):
                return to_real_tensor(e.meta["val"])
            if isinstance(e, torch.Tensor):
                return _tensor([get_fx_node_size_numel(e.size())], e.dtype, e.device)
            return e

        flat_args = [to_real_tensor(a) for a in flat_args]
        real_args, real_kwargs = pytree.tree_unflatten(flat_args, flat_args_pytree_spec)

        fn = fx_node.target
        assert isinstance(fn, torch._ops.OpOverload)
        with torch.distributed._time_estimator(
            group=pg, device=device
        ) as time_estimator:
            w = fn(*real_args, **real_kwargs)
            # Coalesced collectives return a list of tensors
            if isinstance(w, (list, tuple)):
                for t in w:
````
- **EN**: Introduces function `to_real_tensor`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`to_real_tensor`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 521-539 / 第 521-539 行
````python
                    torch.ops._c10d_functional.wait_tensor.default(t)
            else:
                torch.ops._c10d_functional.wait_tensor.default(w)
        est_time_us = time_estimator.estimated_time
        # -1000 constant is NCCL return in case of error during estimations.
        # Observed it for all_to_all estimations.
        if est_time_us < 0:
            return None
        est_time_ms = est_time_us / 1e3
        return est_time_ms

    if use_nccl_estimator:
        est_time_ms = _nccl_estimate()
        if est_time_ms is not None:
            return est_time_ms

    return estimate_nccl_collective_runtime_impl(
        tensor_storage_size_bytes, group_size, coll
    )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `est_time_us`, and `est_time_ms`. This range continues the implementation of function `estimate_nccl_collective_runtime_from_fx_node._nccl_estimate`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`est_time_us`、`est_time_ms` 等值。这一段延续了函数`estimate_nccl_collective_runtime_from_fx_node._nccl_estimate` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `NCCL_COLL`, `NVIDIA_GPU_TYPE`, `NCCL_HW`, `NCCL_ALGO`, and `NCCL_PROTO`  
  **CN**: 主要类：`NCCL_COLL`、`NVIDIA_GPU_TYPE`、`NCCL_HW`、`NCCL_ALGO`、`NCCL_PROTO`
- **EN**: Primary functions: `get_gpu_type`, `get_collective_type_from_kernel_name`, `get_collective_type`, `get_ir_node_size_numel`, `get_fx_node_size_numel`, `get_collective_input_size_bytes`, and `...+7`  
  **CN**: 主要函数：`get_gpu_type`、`get_collective_type_from_kernel_name`、`get_collective_type`、`get_ir_node_size_numel`、`get_fx_node_size_numel`、`get_collective_input_size_bytes`、`另有7项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `logging`, `math`, `operator`, `enum`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._pytree`, `torch.fx.experimental.symbolic_shapes`, `torch.fx.operator_schemas`, `.`, `.utils`, `.virtualized`, `torch.distributed.distributed_c10d`, `torch._inductor.fx_passes.bucketing`
