# custom_all_reduce_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/custom_all_reduce_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `custom_all_reduce_ops`. It mainly organizes module-level constants, imports, and helper logic. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `custom_all_reduce_ops` 的逻辑。 它主要组织模块级常量、导入项与辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup (part 1/4)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/_custom_ops.py
import logging
from typing import List, Optional, Tuple

import torch

from sglang.srt.utils import is_cuda, is_hip, is_musa

logger = logging.getLogger(__name__)

_is_cuda = is_cuda()
_is_hip = is_hip()
_is_musa = is_musa()

IS_CUSTOM_AR_AVAILABLE = _is_cuda or _is_hip or _is_musa
IS_QUICK_AR_AVAILABLE = _is_hip
# TODO(zyksir): mscclpp is untested on AMD and therefore disabled.
IS_MSCCLPP_AR_AVAILABLE = _is_cuda

try:
    import sgl_kernel.allreduce as _custom_ar
except ImportError as e:
    if _is_cuda or _is_hip:
        logger.warning("Failed to import from custom_ar with %r", e)
    IS_CUSTOM_AR_AVAILABLE = False
    IS_QUICK_AR_AVAILABLE = False
    IS_MSCCLPP_AR_AVAILABLE = False

# region IS_CUSTOM_AR_AVAILABLE

if not IS_CUSTOM_AR_AVAILABLE:
    pass

elif _is_cuda or _is_musa:
    # CUDA custom allreduce

    def init_custom_ar(
        ipc_tensors: List[torch.Tensor],
        rank_data: torch.Tensor,
        rank: int,
        full_nvlink: bool,
    ) -> int:
        return _custom_ar.init_custom_ar(ipc_tensors, rank_data, rank, full_nvlink)

    def all_reduce(
        fa: int,
        inp: torch.Tensor,
        out: torch.Tensor,
        reg_buffer: int,
        reg_buffer_sz_bytes: int,
    ) -> None:
        _custom_ar.all_reduce(fa, inp, out, reg_buffer, reg_buffer_sz_bytes)

    def dispose(fa: int) -> None:
        _custom_ar.dispose(fa)

    def meta_size() -> int:
        return _custom_ar.meta_size()
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 1 of 4 for the same logical block. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 1/4 部分。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 61-120: Module-level logic (part 2/4)
```python

    def register_buffer(fa: int, ipc_tensors: List[int]) -> None:
        return _custom_ar.register_buffer(fa, ipc_tensors)

    def get_graph_buffer_ipc_meta(fa: int) -> Tuple[List[int], List[int]]:
        return _custom_ar.get_graph_buffer_ipc_meta(fa)

    def register_graph_buffers(
        fa: int, handles: List[List[int]], offsets: List[List[int]]
    ) -> None:
        _custom_ar.register_graph_buffers(fa, handles, offsets)

elif _is_hip:
    # ROCM custom allreduce

    def init_custom_ar(
        meta: torch.Tensor,
        rank_data: torch.Tensor,
        handles: List[str],
        offsets: List[int],
        rank: int,
        full_nvlink: bool,
    ) -> int:
        return _custom_ar.init_custom_ar(
            meta, rank_data, handles, offsets, rank, full_nvlink
        )

    def all_reduce_reg(fa: int, inp: torch.Tensor, out: torch.Tensor) -> None:
        _custom_ar.all_reduce_reg(fa, inp, out)

    def all_reduce_unreg(
        fa: int, inp: torch.Tensor, reg_buffer: torch.Tensor, out: torch.Tensor
    ) -> None:
        _custom_ar.all_reduce_unreg(fa, inp, reg_buffer, out)

    def deterministic_all_reduce_reg(
        fa: int, inp: torch.Tensor, out: torch.Tensor
    ) -> None:
        _custom_ar.deterministic_all_reduce_reg(fa, inp, out)

    def deterministic_all_reduce_unreg(
        fa: int, inp: torch.Tensor, reg_buffer: torch.Tensor, out: torch.Tensor
    ) -> None:
        _custom_ar.deterministic_all_reduce_unreg(fa, inp, reg_buffer, out)

    def dispose(fa: int) -> None:
        _custom_ar.dispose(fa)

    def meta_size() -> int:
        return _custom_ar.meta_size()

    def register_buffer(
        fa: int, t: torch.Tensor, handles: List[str], offsets: List[int]
    ) -> None:
        return _custom_ar.register_buffer(fa, t, handles, offsets)

    def get_graph_buffer_ipc_meta(fa: int) -> Tuple[torch.Tensor, List[int]]:
        return _custom_ar.get_graph_buffer_ipc_meta(fa)

    def register_graph_buffers(
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 2 of 4 for the same logical block. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 2/4 部分。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 121-180: Module-level constants and helpers (part 3/4)
```python
        fa: int, handles: List[str], offsets: List[List[int]]
    ) -> None:
        _custom_ar.register_graph_buffers(fa, handles, offsets)

    def allocate_meta_buffer(size: int) -> torch.Tensor:
        return _custom_ar.allocate_meta_buffer(size)

    def get_meta_buffer_ipc_handle(inp: torch.Tensor) -> torch.Tensor:
        return _custom_ar.get_meta_buffer_ipc_handle(inp)


# endregion

# region IS_QUICK_AR_AVAILABLE

if not IS_QUICK_AR_AVAILABLE:
    pass

elif _is_hip:
    # ROCM custom quick allreduce

    def init_custom_qr(
        rank: int, world_size: int, qr_max_size: Optional[int] = None
    ) -> int:
        return _custom_ar.init_custom_qr(world_size, rank, qr_max_size)

    def qr_get_handle(fa: int) -> torch.Tensor:
        return _custom_ar.qr_get_handle(fa)

    def qr_open_handles(fa: int, handles: list[torch.Tensor]) -> None:
        _custom_ar.qr_open_handles(fa, handles)

    def qr_all_reduce(
        fa: int,
        inp: torch.Tensor,
        out: torch.Tensor,
        quant_level: int,
        cast_bf2half: bool,
    ) -> None:
        _custom_ar.qr_all_reduce(fa, inp, out, quant_level, cast_bf2half)

    def qr_destroy(fa: int) -> None:
        _custom_ar.qr_destroy(fa)

    def qr_max_size() -> int:
        return _custom_ar.qr_max_size()


# endregion

# region IS_MSCCLPP_AR_AVAILABLE

if not IS_MSCCLPP_AR_AVAILABLE:
    pass

elif _is_cuda:

    def mscclpp_generate_unique_id() -> bytes:
        return _custom_ar.mscclpp_generate_unique_id()

```
**EN:** This range organizes module-level state and shared setup. This chunk is part 3 of 4 for the same logical block. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 3/4 部分。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 181-210: Module-level logic (part 4/4)
```python
    def mscclpp_init_context(
        unique_id: bytes,
        rank: int,
        world_size: int,
        scratch: torch.Tensor,
        put_buffer: torch.Tensor,
        nranks_per_node: int,
        rank_to_node: List[int],
        rank_to_ib: List[int],
        context_selection: int,
    ) -> int:
        return _custom_ar.mscclpp_init_context(
            unique_id,
            rank,
            world_size,
            scratch,
            put_buffer,
            nranks_per_node,
            rank_to_node,
            rank_to_ib,
            context_selection,
        )

    def mscclpp_allreduce(
        context: int, inp: torch.Tensor, out: torch.Tensor, nthreads: int, nblocks: int
    ) -> None:
        return _custom_ar.mscclpp_allreduce(context, inp, out, nthreads, nblocks)


# endregion
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 4 of 4 for the same logical block.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 4/4 部分。

## Key Concepts / 关键概念
- `IS_CUSTOM_AR_AVAILABLE`: module constant or capability flag / 模块常量或能力标记
- `IS_QUICK_AR_AVAILABLE`: module constant or capability flag / 模块常量或能力标记
- `IS_MSCCLPP_AR_AVAILABLE`: module constant or capability flag / 模块常量或能力标记

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: `torch`, `sgl_kernel.allreduce`
- **Internal modules / 内部模块**: `sglang.srt.utils`
