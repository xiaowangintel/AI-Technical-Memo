# reader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/eplb/eplb_simulator/reader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the expert-parallel load balancing part of the SRT runtime and implements logic centered on `reader`. It exposes primary entry points such as `read_mode_per_pass`. / 该模块属于 SRT 运行时的专家并行负载均衡部分，主要实现围绕 `reader` 的逻辑。 它对外提供的主要入口包括 `read_mode_per_pass`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Module imports, constants, and setup
```python
from collections import defaultdict
from pathlib import Path

import torch
from tqdm import tqdm

from sglang.srt.eplb.expert_distribution import (
    _convert_global_physical_count_to_logical_count,
)

convert_global_physical_count_to_logical_count = (
    _convert_global_physical_count_to_logical_count
)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 16-51: Function read_mode_per_pass
```python
def read_mode_per_pass(dir_data: Path):
    """Read data from ExpertDistributionRecorder when recorded with mode `per_pass`"""

    # gpc := global_physical_count
    gpc_of_forward_pass_and_rank = defaultdict(lambda: defaultdict())
    for path in tqdm(list(dir_data.glob("*.pt"))):
        data_pack = torch.load(path, weights_only=True)
        last_physical_to_logical_map = data_pack["last_physical_to_logical_map"]
        for record in data_pack["records"]:
            forward_pass_id = record["forward_pass_id"]
            rank = record["rank"]
            assert (
                gpc_of_forward_pass_and_rank[forward_pass_id].get(rank) is None
            ), f"Duplicated {forward_pass_id=} {rank=}"
            gpc_of_forward_pass_and_rank[forward_pass_id][rank] = record[
                "global_physical_count"
            ]

    forward_pass_ids = sorted(gpc_of_forward_pass_and_rank.keys())
    print(f"Make {forward_pass_ids=} into array")

    items = []
    for forward_pass_id, gpc_of_rank in sorted(gpc_of_forward_pass_and_rank.items()):
        gpc_of_rank_tensor = torch.stack(
            [gpc for rank, gpc in sorted(gpc_of_rank.items())]
        ).sum(dim=0)
        items.append(gpc_of_rank_tensor)

    gpc_of_forward_pass = torch.stack(items)
    print(f"{gpc_of_forward_pass.shape=}")

    return dict(
        global_physical_count_of_forward_pass=gpc_of_forward_pass,
        last_physical_to_logical_map=last_physical_to_logical_map,
        forward_pass_ids=forward_pass_ids,
    )
```
**EN:** This callable implements `read_mode_per_pass`. It takes `dir_data` and mainly implements read mode per pass. The docstring states: "Read data from ExpertDistributionRecorder when recorded with mode `per_pass`" In this range it sets up imports and shared symbols; performs defensive checks on invalid state; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `read_mode_per_pass`。它接收 `dir_data`，主要用于实现 read mode per pass 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；管理模型权重或检查点。

## Key Concepts / 关键概念
- `read_mode_per_pass`: implements read mode per pass / 实现 read mode per pass 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `pathlib`
- **Third-party / 第三方**: `torch`, `tqdm`
- **Internal modules / 内部模块**: `sglang.srt.eplb.expert_distribution`
