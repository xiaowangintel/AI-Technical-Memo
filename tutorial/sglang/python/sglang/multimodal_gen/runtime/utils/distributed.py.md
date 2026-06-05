# distributed.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/utils/distributed.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for distributed in the multimodal generation stack. Key symbols include `broadcast_pyobj`, `generate_masked_orthogonal_rank_groups`, `RankGenerator`. / 该模块包含多模态生成体系中与 distributed 相关的运行时支持代码。 关键符号包括 `broadcast_pyobj`, `generate_masked_orthogonal_rank_groups`, `RankGenerator`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

import pickle
from typing import Any, List, Optional

import numpy as np
import torch
import torch.distributed as dist

from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 13-58: Function `broadcast_pyobj` / 函数 `broadcast_pyobj`
```python
def broadcast_pyobj(
    data: List[Any],
    rank: int,
    dist_group: Optional[torch.distributed.ProcessGroup] = None,
    src: int = 0,
    force_cpu_device: bool = True,
):
    """Broadcast inputs from src rank to all other ranks with torch.dist backend.
    The `rank` here refer to the source rank on global process group (regardless
    of dist_group argument).
    """

    device = torch.device(
        current_platform.device_type if not force_cpu_device else "cpu"
# ...

        serialized_data = bytes(tensor_data.cpu().numpy())
        data = pickle.loads(serialized_data)
        return data
```
**EN:** This function drives `broadcast_pyobj` with inputs such as `data`, `rank`, `dist_group`, `src`. Broadcast inputs from src rank to all other ranks with torch.dist backend.
**CN:** 这个函数负责 `broadcast_pyobj`，主要处理 `data`, `rank`, `dist_group`, `src` 等输入。 文档字符串说明：Broadcast inputs from src rank to all other ranks with torch.dist backend.

### Lines 61-162: Function `generate_masked_orthogonal_rank_groups` / 函数 `generate_masked_orthogonal_rank_groups`
```python
def generate_masked_orthogonal_rank_groups(
    world_size: int, parallel_size: list[int], mask: list[bool]
) -> list[list[int]]:
    """Generate orthogonal parallel groups based on the parallel size and mask.

    Arguments:
        world_size (int): world size

        parallel_size (List[int]):
            The parallel size of each orthogonal parallel type. For example, if
            tensor_parallel_size = 2, pipeline_model_parallel_group = 3, data_parallel_size = 4,
            and the parallel mapping order is tp-pp-dp, then the parallel_size = [2, 3, 4].

        mask (List[bool]):
# ...
                + inner_product(decomposed_group_idx, unmasked_stride)
            )
        ranks.append(rank)
    return ranks
```
**EN:** This function drives `generate_masked_orthogonal_rank_groups` with inputs such as `world_size`, `parallel_size`, `mask`. Generate orthogonal parallel groups based on the parallel size and mask.
**CN:** 这个函数负责 `generate_masked_orthogonal_rank_groups`，主要处理 `world_size`, `parallel_size`, `mask` 等输入。 文档字符串说明：Generate orthogonal parallel groups based on the parallel size and mask.

### Lines 165-234: Class `RankGenerator` / 类 `RankGenerator`
```python
class RankGenerator(object):
    def __init__(
        self,
        tp: int,
        sp: int,
        pp: int,
        cfg: int,
        dp: int,
        order: str,
        rank_offset: int = 0,
    ) -> None:
        self.tp = tp
        self.sp = sp
        self.pp = pp
# ...
            for rank_group in ranks:
                for i in range(len(rank_group)):
                    rank_group[i] += self.rank_offset
        return ranks
```
**EN:** This class models `RankGenerator` as a specialization of `object`. Important methods include `__init__`, `get_mask`, `get_ranks`.
**CN:** 该类实现 `RankGenerator`，并继承/扩展 `object`。 其中较重要的方法包括 `__init__`, `get_mask`, `get_ranks`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程
- Symbol `broadcast_pyobj` anchors the module API / 符号 `broadcast_pyobj` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `numpy`, `torch`, `torch.distributed`
- **Stdlib / 标准库**: `pickle`, `typing`
