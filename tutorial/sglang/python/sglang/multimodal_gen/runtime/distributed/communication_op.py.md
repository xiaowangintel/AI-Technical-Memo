# communication_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/communication_op.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `tensor_model_parallel_all_reduce`, `tensor_model_parallel_all_gather`, and `sequence_model_parallel_all_to_all_4D`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `tensor_model_parallel_all_reduce`、`tensor_model_parallel_all_gather` 和 `sequence_model_parallel_all_to_all_4D` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 7-14: module setup and imports / 模块初始化与导入
```python
import torch
import torch.distributed as dist

from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_cfg_group,
    get_sp_group,
    get_tp_group,
)
```
**EN:** This block establishes the module context and imports `torch`, `torch.distributed`, and `sglang.multimodal_gen.runtime.distributed.parallel_state`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`、`torch.distributed` 和 `sglang.multimodal_gen.runtime.distributed.parallel_state`。这些依赖为后续实现提供所需符号。

### Lines 17-22: `tensor_model_parallel_all_reduce` implementation / `tensor_model_parallel_all_reduce` 实现
```python
def tensor_model_parallel_all_reduce(
    input_: torch.Tensor, tp_group: dist.ProcessGroup = None
) -> torch.Tensor:
    """All-reduce the input tensor across model parallel group."""
    tp_group = tp_group or get_tp_group()
    return tp_group.all_reduce(input_)
```
**EN:** This block defines function `tensor_model_parallel_all_reduce`. All-reduce the input tensor across model parallel group. Key calls include `tp_group.all_reduce`, and `get_tp_group`. Parameters such as `input_`, and `tp_group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `tensor_model_parallel_all_reduce`。 它用于处理 tensor model parallel all reduce 相关逻辑。 关键调用包括 `tp_group.all_reduce` 和 `get_tp_group`。 本段逻辑主要由 `input_` 和 `tp_group` 等参数驱动。

### Lines 25-30: `tensor_model_parallel_all_gather` implementation / `tensor_model_parallel_all_gather` 实现
```python
def tensor_model_parallel_all_gather(
    input_: torch.Tensor, dim: int = -1, tp_group: dist.ProcessGroup = None
) -> torch.Tensor:
    """All-gather the input tensor across model parallel group."""
    tp_group = tp_group or get_tp_group()
    return tp_group.all_gather(input_, dim)
```
**EN:** This block defines function `tensor_model_parallel_all_gather`. All-gather the input tensor across model parallel group. Key calls include `tp_group.all_gather`, and `get_tp_group`. Parameters such as `input_`, `dim`, and `tp_group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `tensor_model_parallel_all_gather`。 它用于处理 tensor model parallel all gather 相关逻辑。 关键调用包括 `tp_group.all_gather` 和 `get_tp_group`。 本段逻辑主要由 `input_`、`dim` 和 `tp_group` 等参数驱动。

### Lines 34-38: `sequence_model_parallel_all_to_all_4D` implementation / `sequence_model_parallel_all_to_all_4D` 实现
```python
def sequence_model_parallel_all_to_all_4D(
    input_: torch.Tensor, scatter_dim: int = 2, gather_dim: int = 1
) -> torch.Tensor:
    """All-to-all communication of 4D tensors (e.g. QKV matrices) across sequence parallel group."""
    return get_sp_group().all_to_all_4D(input_, scatter_dim, gather_dim)
```
**EN:** This block defines function `sequence_model_parallel_all_to_all_4D`. All-to-all communication of 4D tensors (e.g. QKV matrices) across sequence parallel group. Key calls include `get_sp_group.all_to_all_4D`, and `get_sp_group`. Parameters such as `input_`, `scatter_dim`, and `gather_dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `sequence_model_parallel_all_to_all_4D`。 它用于处理 sequence model parallel all to all 4 d 相关逻辑。 关键调用包括 `get_sp_group.all_to_all_4D` 和 `get_sp_group`。 本段逻辑主要由 `input_`、`scatter_dim` 和 `gather_dim` 等参数驱动。

### Lines 41-45: `sequence_model_parallel_all_gather` implementation / `sequence_model_parallel_all_gather` 实现
```python
def sequence_model_parallel_all_gather(
    input_: torch.Tensor, dim: int = -1
) -> torch.Tensor:
    """All-gather the input tensor across model parallel group."""
    return get_sp_group().all_gather(input_, dim)
```
**EN:** This block defines function `sequence_model_parallel_all_gather`. All-gather the input tensor across model parallel group. Key calls include `get_sp_group.all_gather`, and `get_sp_group`. Parameters such as `input_`, and `dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `sequence_model_parallel_all_gather`。 它用于处理 sequence model parallel all gather 相关逻辑。 关键调用包括 `get_sp_group.all_gather` 和 `get_sp_group`。 本段逻辑主要由 `input_` 和 `dim` 等参数驱动。

### Lines 48-50: `sequence_model_parallel_all_reduce` implementation / `sequence_model_parallel_all_reduce` 实现
```python
def sequence_model_parallel_all_reduce(input_: torch.Tensor) -> torch.Tensor:
    """All-reduce the input tensor across model parallel group."""
    return get_sp_group().all_reduce(input_)
```
**EN:** This block defines function `sequence_model_parallel_all_reduce`. All-reduce the input tensor across model parallel group. Key calls include `get_sp_group.all_reduce`, and `get_sp_group`. Parameters such as `input_` drive the behavior in this section.
**CN:** 该代码块定义了函数 `sequence_model_parallel_all_reduce`。 它用于处理 sequence model parallel all reduce 相关逻辑。 关键调用包括 `get_sp_group.all_reduce` 和 `get_sp_group`。 本段逻辑主要由 `input_` 等参数驱动。

### Lines 53-57: `cfg_model_parallel_all_gather` implementation / `cfg_model_parallel_all_gather` 实现
```python
def cfg_model_parallel_all_gather(
    input_: torch.Tensor, dim: int = -1, separate_tensors: bool = False
) -> torch.Tensor:
    """All-gather the input tensor across model parallel group."""
    return get_cfg_group().all_gather(input_, dim, separate_tensors)
```
**EN:** This block defines function `cfg_model_parallel_all_gather`. All-gather the input tensor across model parallel group. Key calls include `get_cfg_group.all_gather`, and `get_cfg_group`. Parameters such as `input_`, `dim`, and `separate_tensors` drive the behavior in this section.
**CN:** 该代码块定义了函数 `cfg_model_parallel_all_gather`。 它用于处理 cfg model parallel all gather 相关逻辑。 关键调用包括 `get_cfg_group.all_gather` 和 `get_cfg_group`。 本段逻辑主要由 `input_`、`dim` 和 `separate_tensors` 等参数驱动。

### Lines 60-67: `cfg_model_parallel_all_reduce` implementation / `cfg_model_parallel_all_reduce` 实现
```python
def cfg_model_parallel_all_reduce(
    input_: torch.Tensor,
    op: torch._C._distributed_c10d.ReduceOp = torch._C._distributed_c10d.ReduceOp.SUM,
) -> torch.Tensor:
    """All-reduce the input tensor across CFG parallel group."""
    if not input_.is_contiguous():
        input_ = input_.contiguous()
    return get_cfg_group().all_reduce(input_, op=op)
```
**EN:** This block defines function `cfg_model_parallel_all_reduce`. All-reduce the input tensor across CFG parallel group. Key calls include `get_cfg_group.all_reduce`, `input_.is_contiguous`, `input_.contiguous`, and `get_cfg_group`. The implementation branches on conditions. Parameters such as `input_`, and `op` drive the behavior in this section.
**CN:** 该代码块定义了函数 `cfg_model_parallel_all_reduce`。 它用于处理 cfg model parallel all reduce 相关逻辑。 关键调用包括 `get_cfg_group.all_reduce`、`input_.is_contiguous`、`input_.contiguous` 和 `get_cfg_group`。 实现中包含条件分支。 本段逻辑主要由 `input_` 和 `op` 等参数驱动。

## Key Concepts / 关键概念
- `tensor_model_parallel_all_reduce`: All-reduce the input tensor across model parallel group. / 顶层函数，用于处理 tensor model parallel all reduce 相关逻辑。
- `tensor_model_parallel_all_gather`: All-gather the input tensor across model parallel group. / 顶层函数，用于处理 tensor model parallel all gather 相关逻辑。
- `sequence_model_parallel_all_to_all_4D`: All-to-all communication of 4D tensors (e.g. / 顶层函数，用于处理 sequence model parallel all to all 4 d 相关逻辑。
- `sequence_model_parallel_all_gather`: All-gather the input tensor across model parallel group. / 顶层函数，用于处理 sequence model parallel all gather 相关逻辑。
- `sequence_model_parallel_all_reduce`: All-reduce the input tensor across model parallel group. / 顶层函数，用于处理 sequence model parallel all reduce 相关逻辑。
- `cfg_model_parallel_all_gather`: All-gather the input tensor across model parallel group. / 顶层函数，用于处理 cfg model parallel all gather 相关逻辑。
- `cfg_model_parallel_all_reduce`: All-reduce the input tensor across CFG parallel group. / 顶层函数，用于处理 cfg model parallel all reduce 相关逻辑。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed.parallel_state`

- **Total lines / 总行数**: 67
