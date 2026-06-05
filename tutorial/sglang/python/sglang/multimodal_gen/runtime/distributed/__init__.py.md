# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `_get_folding_tp_group`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `_get_folding_tp_group` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-28: module setup and imports / 模块初始化与导入
```python
from functools import lru_cache

from sglang.multimodal_gen.configs.models.encoders import TextEncoderConfig
from sglang.multimodal_gen.runtime.distributed.communication_op import *
from sglang.multimodal_gen.runtime.distributed.group_coordinator import (
    get_local_torch_device,
)
from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    cleanup_dist_env_and_memory,
    get_dp_group,
    get_dp_rank,
    get_dp_world_size,
    get_sp_group,
    get_sp_parallel_rank,
    get_sp_world_size,
    get_tp_group,
    get_tp_rank,
    get_tp_world_size,
    get_world_group,
    get_world_rank,
    get_world_size,
    init_distributed_environment,
    initialize_model_parallel,
    maybe_init_distributed_environment_and_model_parallel,
    model_parallel_is_initialized,
)
from sglang.multimodal_gen.runtime.distributed.utils import *
```
**EN:** This block establishes the module context and imports `functools`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.distributed.group_coordinator`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, and `sglang.multimodal_gen.runtime.distributed.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `functools`、`sglang.multimodal_gen.configs.models.encoders`、`sglang.multimodal_gen.runtime.distributed.communication_op`、`sglang.multimodal_gen.runtime.distributed.group_coordinator`、`sglang.multimodal_gen.runtime.distributed.parallel_state` 和 `sglang.multimodal_gen.runtime.distributed.utils`。这些依赖为后续实现提供所需符号。

### Lines 33-58: supporting statements / 辅助语句
```python
__all__ = [
    # Initialization
    "init_distributed_environment",
    "initialize_model_parallel",
    "cleanup_dist_env_and_memory",
    "model_parallel_is_initialized",
    "maybe_init_distributed_environment_and_model_parallel",
    # World group
    "get_world_group",
    "get_world_rank",
    "get_world_size",
    # Data parallel group
    "get_dp_group",
    "get_dp_rank",
    "get_dp_world_size",
    # Sequence parallel group
    "get_sp_group",
    "get_sp_parallel_rank",
    "get_sp_world_size",
    # Tensor parallel group
    "get_tp_group",
    "get_tp_rank",
    "get_tp_world_size",
    # Get torch device
    "get_local_torch_device",
]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `__all__`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `__all__` 等名称。

### Lines 61-71: `_get_folding_tp_group` implementation / `_get_folding_tp_group` 实现
```python
def _get_folding_tp_group(
    config: TextEncoderConfig,
) -> torch.distributed.ProcessGroup | None:
    if config.parallel_folding:
        if config.parallel_folding_mode == "sp":
            return get_sp_group()
        elif config.parallel_folding_mode == "ulysses":
            return get_sp_group().ulysses_group
        elif config.parallel_folding_mode == "ring":
            return get_sp_group().ring_group
    return get_tp_group()
```
**EN:** This block defines function `_get_folding_tp_group`. It retrieves folding tp group. Key calls include `get_tp_group`, and `get_sp_group`. The implementation branches on conditions. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_get_folding_tp_group`。 它用于获取folding tp group。 关键调用包括 `get_tp_group` 和 `get_sp_group`。 实现中包含条件分支。 本段逻辑主要由 `config` 等参数驱动。

## Key Concepts / 关键概念
- `_get_folding_tp_group`: Top-level function that retrieves folding tp group. / 顶层函数，用于获取folding tp group。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.runtime.distributed.communication_op`, `sglang.multimodal_gen.runtime.distributed.group_coordinator`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.distributed.utils`

- **Total lines / 总行数**: 71
