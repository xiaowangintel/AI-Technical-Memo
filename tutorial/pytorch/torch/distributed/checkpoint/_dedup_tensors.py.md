# _dedup_tensors.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/_dedup_tensors.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include init_logger, dedup_tensors.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 init_logger, dedup_tensors。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# Copyright (c) Meta Platforms, Inc. and affiliates
import dataclasses
import logging
from typing import TYPE_CHECKING

from torch.distributed.checkpoint.planner import SavePlan


if TYPE_CHECKING:
    from torch.distributed.checkpoint.metadata import MetadataIndex

__all__ = ["dedup_tensors"]


def init_logger() -> logging.Logger:
    logger = logging.getLogger(__name__)
    level = logging.INFO
    logger.setLevel(level)
    console = logging.StreamHandler()
    formatter = logging.Formatter(
````

- **L1** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L2** EN: Imports module dependencies: `dataclasses`. | CN: 导入模块依赖：`dataclasses`。
- **L3** EN: Imports module dependencies: `logging`. | CN: 导入模块依赖：`logging`。
- **L4** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Imports selected names from `torch.distributed.checkpoint.planner`. | CN: 从 `torch.distributed.checkpoint.planner` 导入指定名称。
- **L7** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L10** EN: Imports selected names from `torch.distributed.checkpoint.metadata`. | CN: 从 `torch.distributed.checkpoint.metadata` 导入指定名称。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Defines function `init_logger`. | CN: 定义函数 `init_logger`。
- **L16** EN: Assigns or updates `logger`. | CN: 对 `logger` 进行赋值或更新。
- **L17** EN: Assigns or updates `level`. | CN: 对 `level` 进行赋值或更新。
- **L18** EN: Calls `logger.setLevel` as part of the current workflow. | CN: 在当前流程中调用 `logger.setLevel`。
- **L19** EN: Assigns or updates `console`. | CN: 对 `console` 进行赋值或更新。
- **L20** EN: Assigns or updates `formatter`. | CN: 对 `formatter` 进行赋值或更新。

### Lines 21-40 / 第 21-40 行

````python
        "%(asctime)s %(filename)s:%(lineno)s %(levelname)s p:%(processName)s t:%(threadName)s: %(message)s"
    )
    console.setFormatter(formatter)
    console.setLevel(level)
    logger.addHandler(console)
    logger.propagate = False
    return logger


logger = init_logger()


# TODO add docstring for dedup_tensors
def dedup_tensors(all_plans: list[SavePlan]) -> list[SavePlan]:
    all_plans = list(all_plans)
    key_to_plan: dict[MetadataIndex, list[int]] = {}
    for plan_idx, plan in enumerate(all_plans):
        for write_item in plan.items:
            key_to_plan.setdefault(write_item.index, []).append(plan_idx)

````

- **L21** EN: Continues the implementation inside function `init_logger`. | CN: 继续说明函数 `init_logger` 内部的实现。
- **L22** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L23** EN: Calls `console.setFormatter` as part of the current workflow. | CN: 在当前流程中调用 `console.setFormatter`。
- **L24** EN: Calls `console.setLevel` as part of the current workflow. | CN: 在当前流程中调用 `console.setLevel`。
- **L25** EN: Calls `logger.addHandler` as part of the current workflow. | CN: 在当前流程中调用 `logger.addHandler`。
- **L26** EN: Assigns or updates `logger.propagate`. | CN: 对 `logger.propagate` 进行赋值或更新。
- **L27** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L28** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Assigns or updates `logger`. | CN: 对 `logger` 进行赋值或更新。
- **L31** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L32** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L33** EN: Keeps the inline comment or directive: TODO add docstring for dedup_tensors | CN: 保留这一行注释或指令：TODO add docstring for dedup_tensors
- **L34** EN: Defines function `dedup_tensors`. | CN: 定义函数 `dedup_tensors`。
- **L35** EN: Assigns or updates `all_plans`. | CN: 对 `all_plans` 进行赋值或更新。
- **L36** EN: Assigns or updates `key_to_plan`. | CN: 对 `key_to_plan` 进行赋值或更新。
- **L37** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L38** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L39** EN: Calls `key_to_plan.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `key_to_plan.setdefault`。
- **L40** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 41-60 / 第 41-60 行

````python
    replicated_items = {k: v for k, v in key_to_plan.items() if len(v) > 1}

    # Remove duplicates by always keeping the first entry.
    # Compute the per-rank remove set.
    plan_to_keys: dict[int, list[MetadataIndex]] = {}
    for key, plans in replicated_items.items():
        for plan_idx in plans[1:]:
            plan_to_keys.setdefault(plan_idx, []).append(key)
    if len(plan_to_keys) > 0:
        logger.info("Duplicate keys to remove: %s", plan_to_keys)

    for plan_idx, keys in plan_to_keys.items():
        key_set = set(keys)
        # rewrite items and remove elements
        new_items = [
            write_item
            for write_item in all_plans[plan_idx].items
            if write_item.index not in key_set
        ]
        all_plans[plan_idx] = dataclasses.replace(all_plans[plan_idx], items=new_items)
````

- **L41** EN: Assigns or updates `replicated_items`. | CN: 对 `replicated_items` 进行赋值或更新。
- **L42** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L43** EN: Keeps the inline comment or directive: Remove duplicates by always keeping the first entry. | CN: 保留这一行注释或指令：Remove duplicates by always keeping the first entry.
- **L44** EN: Keeps the inline comment or directive: Compute the per-rank remove set. | CN: 保留这一行注释或指令：Compute the per-rank remove set.
- **L45** EN: Assigns or updates `plan_to_keys`. | CN: 对 `plan_to_keys` 进行赋值或更新。
- **L46** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L47** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L48** EN: Calls `plan_to_keys.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `plan_to_keys.setdefault`。
- **L49** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L50** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L51** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L52** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L53** EN: Assigns or updates `key_set`. | CN: 对 `key_set` 进行赋值或更新。
- **L54** EN: Keeps the inline comment or directive: rewrite items and remove elements | CN: 保留这一行注释或指令：rewrite items and remove elements
- **L55** EN: Assigns or updates `new_items`. | CN: 对 `new_items` 进行赋值或更新。
- **L56** EN: Continues the implementation inside function `dedup_tensors`. | CN: 继续说明函数 `dedup_tensors` 内部的实现。
- **L57** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L58** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L59** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L60** EN: Assigns or updates `all_plans[plan_idx]`. | CN: 对 `all_plans[plan_idx]` 进行赋值或更新。

### Lines 61-62 / 第 61-62 行

````python

    return all_plans
````

- **L61** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L62** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: Core callables: init_logger, dedup_tensors  
  **CN**: 核心可调用对象：init_logger, dedup_tensors

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.checkpoint.metadata`, `torch.distributed.checkpoint.planner`
- **PyTorch / PyTorch**: None detected / 未检测到
- **Python Stdlib / Python 标准库**: `dataclasses`, `logging`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

