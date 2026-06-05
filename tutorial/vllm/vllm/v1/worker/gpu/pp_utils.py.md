# pp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/pp_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pipeline Parallelism utils for V2 Model Runner. / 该模块位于 `worker/gpu` 子系统，主要围绕 `pp_broadcast`, `pp_receive` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Pipeline Parallelism utils for V2 Model Runner."""

import torch

from vllm.distributed.parallel_state import get_pp_group
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `pp_broadcast` function / `pp_broadcast` 函数
```python
def pp_broadcast(
    sampled_token_ids: torch.Tensor,
    num_sampled: torch.Tensor,
    num_rejected: torch.Tensor,
) -> None:
    pp = get_pp_group()
    assert pp.is_last_rank

    assert sampled_token_ids.dtype == torch.int64
    torch.distributed.broadcast(
        sampled_token_ids.contiguous(), src=pp.last_rank, group=pp.device_group
    )

    combined = torch.stack((num_sampled, num_rejected), dim=0)
    torch.distributed.broadcast(combined, src=pp.last_rank, group=pp.device_group)
```
**EN:** This function implements `pp_broadcast` within the module. Key calls include `get_pp_group`, `broadcast`, `stack`, `contiguous`.
**CN:** 该函数会实现 `pp_broadcast`，其作用域位于the module。 关键调用包括 `get_pp_group`, `broadcast`, `stack`, `contiguous`。

### `pp_receive` function / `pp_receive` 函数
```python
def pp_receive(
    num_reqs: int, max_sample_len: int = 1
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    pp = get_pp_group()
    assert not pp.is_last_rank

    sampled_tokens = torch.empty(
        num_reqs, max_sample_len, dtype=torch.int64, device=pp.device
    )
    torch.distributed.broadcast(sampled_tokens, src=pp.last_rank, group=pp.device_group)

    combined = torch.empty(2, num_reqs, dtype=torch.int32, device=pp.device)
    torch.distributed.broadcast(combined, src=pp.last_rank, group=pp.device_group)
    num_sampled, num_rejected = combined.unbind(dim=0)
    return sampled_tokens, num_sampled, num_rejected
```
**EN:** This function implements `pp_receive` within the module. Key calls include `get_pp_group`, `empty`, `broadcast`, `unbind`.
**CN:** 该函数会实现 `pp_receive`，其作用域位于the module。 关键调用包括 `get_pp_group`, `empty`, `broadcast`, `unbind`。

## Key Concepts / 关键概念
- `pp_broadcast`: top-level helper or orchestration entry point. / `pp_broadcast`：顶层辅助函数或编排入口。
- `pp_receive`: top-level helper or orchestration entry point. / `pp_receive`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.distributed.parallel_state`
