# default.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/eplb/policy/default.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Expert parallelism load balancer (EPLB) for vLLM / 实现专家并行负载均衡的策略、状态或工作进程逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""
Expert parallelism load balancer (EPLB) for vLLM.

This module implements the core rearrangement algorithm.

The rearrangement algorithm is adapted from
[DeepSeek EPLB](https://github.com/deepseek-ai/eplb).

Please find at [#12](https://github.com/deepseek-ai/EPLB/issues/12) an example
on how the EPLB algorithm works.
"""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Expert parallelism load balancer (EPLB) for vLLM.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
import numpy as np
import torch

from .abstract import AbstractEplbPolicy
```
**EN:** This block imports `numpy`, `torch`, `.abstract` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `numpy`, `torch`, `.abstract`，为后续实现准备运行时、类型与辅助 API。

### Class `DefaultEplbPolicy` / 类 `DefaultEplbPolicy`
```python
class DefaultEplbPolicy(AbstractEplbPolicy):
    @classmethod
    def balanced_packing(
        cls, weight: np.ndarray, num_packs: int
    ) -> tuple[np.ndarray, np.ndarray]:
        """
        Pack n weighted objects to m packs, such that each bin contains exactly
        n/m objects and the weights of all packs are as balanced as possible.

        Parameters:
            weight: [X, n], the weight of each item
            num_packs: number of packs

        Returns:
            pack_index: [X, n], the pack index of each item
            rank_in_pack: [X, n], the rank of the item in the pack
        """
        num_layers, num_groups = weight.shape
        assert num_groups % num_packs == 0
        groups_per_pack = num_groups // num_packs

        if groups_per_pack == 1:
            pack_index = np.tile(np.arange(num_groups, dtype=np.int64), (num_layers, 1))
            rank_in_pack = np.zeros_like(pack_index, dtype=np.int64)
            return pack_index, rank_in_pack

        # Sort and get indices in descending order
        indices = np.argsort(-weight, axis=-1)

        pack_index = np.full((num_layers, num_groups), -1, dtype=np.int64)
        rank_in_pack = np.full((num_layers, num_groups), -1, dtype=np.int64)

        pack_weights = np.zeros((num_layers, num_packs), dtype=np.float64)
        pack_items = np.zeros((num_layers, num_packs), dtype=np.int64)

        # Run the packing algorithm
        for layer_idx in range(num_layers):
            weights_row = pack_weights[layer_idx]
            items_row = pack_items[layer_idx]

            for group in indices[layer_idx]:
                # Pick the lightest pack; full packs are masked out by inf.
                pack = int(np.argmin(weights_row))

                pack_index[layer_idx, group] = pack
# ... truncated for analysis ...
        # Only apply when the number of GPUs and slots per GPU remain unchanged.
        # Helps to avoid unnecessary weight copying when experts move
        # within the same GPU.
        if old_phy2log_np is not None:
            phy2log_np = cls.preserve_intragpu_slots(
                phy2log_np, num_ranks, old_phy2log_np
            )

        phy2log = torch.from_numpy(phy2log_np)
        return phy2log
```
**EN:** Declares `DefaultEplbPolicy`, a class derived from `AbstractEplbPolicy`. Key methods include `balanced_packing`, `replicate_experts`, `rebalance_experts_hierarchical`, `preserve_intragpu_slots`, `rebalance_experts`.
**CN:** 声明 `DefaultEplbPolicy`，它是一个类，继承自 `AbstractEplbPolicy`。 关键方法包括 `balanced_packing`, `replicate_experts`, `rebalance_experts_hierarchical`, `preserve_intragpu_slots`, `rebalance_experts`。

## Key Concepts / 关键概念
- Expert-parallel load balancing / 专家并行负载均衡
- `DefaultEplbPolicy`: class interface or data carrier / `DefaultEplbPolicy`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: `numpy`, `torch`
- **Internal modules / 内部模块**: `.abstract`
