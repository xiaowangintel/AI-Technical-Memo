# abstract.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/eplb/policy/abstract.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements expert-parallel load-balancing policies, state, or workers. / 实现专家并行负载均衡的策略、状态或工作进程逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
from abc import ABC, abstractmethod

import torch
```
**EN:** This block imports `abc`, `torch` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `abc`, `torch`，为后续实现准备运行时、类型与辅助 API。

### Class `AbstractEplbPolicy` / 类 `AbstractEplbPolicy`
```python
class AbstractEplbPolicy(ABC):
    @classmethod
    @abstractmethod
    def rebalance_experts(
        cls,
        weight: torch.Tensor,
        num_replicas: int,
        num_groups: int,
        num_nodes: int,
        num_ranks: int,
        old_global_expert_indices: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """
        Entry point for expert-parallelism load balancer.

        Parameters:
            weight: [layers, num_logical_experts], the load statistics
                for all logical experts
            num_replicas: number of physical experts, must be a multiple of
                `num_ranks`
            num_groups: number of expert groups
            num_nodes: number of server nodes
            num_ranks: number of ranks, must be a multiple of `num_nodes`
            old_global_expert_indices: [layers, num_logical_experts], the old global
                expert indices. Used to avoid unnecessary weight copying
                for experts moving within one rank.
        Returns:
            physical_to_logical_map: [layers, num_replicas], the expert
                index of each replica
        """
        raise NotImplementedError
```
**EN:** Declares `AbstractEplbPolicy`, a abstract base class derived from `ABC`. Key methods include `rebalance_experts`.
**CN:** 声明 `AbstractEplbPolicy`，它是一个抽象基类，继承自 `ABC`。 关键方法包括 `rebalance_experts`。

## Key Concepts / 关键概念
- Expert-parallel load balancing / 专家并行负载均衡
- `AbstractEplbPolicy`: abstract base class interface or data carrier / `AbstractEplbPolicy`：抽象基类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `abc`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: None / 无
