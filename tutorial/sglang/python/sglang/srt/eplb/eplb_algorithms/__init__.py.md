# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/eplb/eplb_algorithms/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the expert-parallel load balancing part of the SRT runtime and implements logic centered on `__init__`. It exposes primary entry points such as `EplbAlgorithm`, `rebalance_experts`, `compute_algorithm`. / 该模块属于 SRT 运行时的专家并行负载均衡部分，主要实现围绕 `__init__` 的逻辑。 它对外提供的主要入口包括 `EplbAlgorithm`, `rebalance_experts`, `compute_algorithm`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module imports, constants, and setup
```python
from enum import Enum, auto
from typing import Optional

import torch

from sglang.srt.eplb.eplb_algorithms import deepseek, deepseek_vec, elasticity_aware


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 9-15: Class EplbAlgorithm
```python
class EplbAlgorithm(Enum):
    deepseek = auto()
    deepseek_hierarchical = auto()
    deepseek_vec = auto()
    deepseek_vec_hierarchical = auto()
    elasticity_aware = auto()
    elasticity_aware_hierarchical = auto()
```
**EN:** This range introduces `EplbAlgorithm` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `EplbAlgorithm`，并定义其后续方法依赖的结构或元数据。

### Lines 16-18: Module-level logic
```python
    # TODO may have more algorithm later


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 19-72: Function rebalance_experts
```python
def rebalance_experts(
    tokens_per_expert: torch.Tensor,
    num_physical_experts: int,
    num_local_physical_experts: int,
    num_groups: Optional[int],
    num_nodes: int,
    algorithm: EplbAlgorithm,
):
    if algorithm in [EplbAlgorithm.deepseek, EplbAlgorithm.deepseek_hierarchical]:
        return deepseek.rebalance_experts(
            weight=tokens_per_expert.sum(dim=0),
            num_replicas=num_physical_experts,
            num_groups=num_groups,
            num_nodes=num_nodes,
            num_gpus=num_physical_experts // num_local_physical_experts,
            enable_hierarchical=algorithm == EplbAlgorithm.deepseek_hierarchical,
        )

    if algorithm in [
        EplbAlgorithm.deepseek_vec,
        EplbAlgorithm.deepseek_vec_hierarchical,
    ]:
        return deepseek_vec.rebalance_experts(
            tokens_per_expert=tokens_per_expert,
            num_physical_experts=num_physical_experts,
            num_local_physical_experts=num_local_physical_experts,
            num_groups=num_groups,
            num_nodes=num_nodes,
            enable_hierarchical=algorithm == EplbAlgorithm.deepseek_vec_hierarchical,
        )

    if algorithm in [
        EplbAlgorithm.elasticity_aware,
        EplbAlgorithm.elasticity_aware_hierarchical,
    ]:
        from sglang.srt.elastic_ep.elastic_ep import ElasticEPStateManager

        return elasticity_aware.rebalance_experts(
            weight=tokens_per_expert.sum(dim=0),
            num_replicas=num_physical_experts,
            num_groups=num_groups,
            num_nodes=num_nodes,
            num_gpus=num_physical_experts // num_local_physical_experts,
            enable_hierarchical=(
                algorithm == EplbAlgorithm.elasticity_aware_hierarchical
            ),
            active_ranks=(
                ElasticEPStateManager.instance().active_ranks
                if ElasticEPStateManager.instance() is not None
                else ElasticEPStateManager.healthy_rank_state()
            ),
        )

    raise NotImplementedError
```
**EN:** This callable implements `rebalance_experts`. It takes `tokens_per_expert`, `num_physical_experts`, `num_local_physical_experts`, `num_groups` and mainly implements rebalance experts. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `rebalance_experts`。它接收 `tokens_per_expert`, `num_physical_experts`, `num_local_physical_experts`, `num_groups`，主要用于实现 rebalance experts 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；管理模型权重或检查点。

### Lines 75-87: Function compute_algorithm
```python
def compute_algorithm(
    raw_algorithm: str,
    num_groups: Optional[int],
    num_nodes: int,
) -> EplbAlgorithm:
    if raw_algorithm != "auto":
        return EplbAlgorithm[raw_algorithm]

    # TODO test on real scenarios and know which ones perform better
    if (num_groups is not None) and (num_groups % num_nodes == 0):
        return EplbAlgorithm.deepseek_hierarchical
    else:
        return EplbAlgorithm.deepseek
```
**EN:** This callable implements `compute_algorithm`. It takes `raw_algorithm`, `num_groups`, `num_nodes` and mainly implements compute algorithm.
**CN:** 这一可调用对象实现了 `compute_algorithm`。它接收 `raw_algorithm`, `num_groups`, `num_nodes`，主要用于实现 compute algorithm 相关逻辑。

## Key Concepts / 关键概念
- `EplbAlgorithm`: core class or state container / 核心类或状态容器
- `rebalance_experts`: implements rebalance experts / 实现 rebalance experts 相关逻辑
- `compute_algorithm`: implements compute algorithm / 实现 compute algorithm 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `enum`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.eplb.eplb_algorithms`, `sglang.srt.elastic_ep.elastic_ep`
