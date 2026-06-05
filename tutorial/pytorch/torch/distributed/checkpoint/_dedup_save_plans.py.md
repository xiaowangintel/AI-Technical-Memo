# _dedup_save_plans.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/_dedup_save_plans.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include dedup_save_plans.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 dedup_save_plans。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# Copyright (c) Meta Platforms, Inc. and affiliates
import dataclasses
from collections import defaultdict
from typing import TYPE_CHECKING

from torch.distributed.checkpoint.planner import SavePlan, WriteItem


if TYPE_CHECKING:
    from torch.distributed.checkpoint.metadata import MetadataIndex

__all__ = ["dedup_save_plans"]


def dedup_save_plans(
    all_plans: list[SavePlan],
    save_to_lowest_rank: bool = False,
) -> list[SavePlan]:
    """
    Removes duplicate entries from appearing on multiple SavePlans. For each duplicate across
````

- **L1** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L2** EN: Imports module dependencies: `dataclasses`. | CN: 导入模块依赖：`dataclasses`。
- **L3** EN: Imports selected names from `collections`. | CN: 从 `collections` 导入指定名称。
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
- **L15** EN: Defines function `dedup_save_plans`. | CN: 定义函数 `dedup_save_plans`。
- **L16** EN: Continues the implementation inside function `dedup_save_plans`. | CN: 继续说明函数 `dedup_save_plans` 内部的实现。
- **L17** EN: Assigns or updates `save_to_lowest_rank`. | CN: 对 `save_to_lowest_rank` 进行赋值或更新。
- **L18** EN: Continues the implementation inside function `dedup_save_plans`. | CN: 继续说明函数 `dedup_save_plans` 内部的实现。
- **L19** EN: Starts the docstring for the function dedup_save_plans. | CN: 开始定义 function dedup_save_plans 的文档字符串。
- **L20** EN: Continues the docstring text for the function dedup_save_plans. | CN: 继续补充 function dedup_save_plans 的文档字符串内容。

### Lines 21-40 / 第 21-40 行

````python
    a set of SavePlans, only the smallest SavePlan in terms of planned storage keeps the entry.

    Please note that this function does not modify the original SavePlans, but rather returns
    """

    # Map to query the plan indices that a write item is duplicated in
    write_item_to_plan_indices: dict[MetadataIndex, set[int]] = defaultdict(set)
    # Map to query the write item from its index
    write_item_idx_to_write_item: dict[MetadataIndex, WriteItem] = {}
    # Set of write item indices that are present in each plan
    # After deduplication, this will be the set of write item indices that are present in the final plans
    plan_to_item_indices: list[set[MetadataIndex]] = [
        {item.index for item in plan.items} for plan in all_plans
    ]

    for plan_idx, plan in enumerate(all_plans):
        for write_item in plan.items:
            # map each write item to its plan
            write_item_to_plan_indices[write_item.index].add(plan_idx)
            write_item_idx_to_write_item[write_item.index] = write_item
````

- **L21** EN: Continues the docstring text for the function dedup_save_plans. | CN: 继续补充 function dedup_save_plans 的文档字符串内容。
- **L22** EN: Continues the docstring text for the function dedup_save_plans. | CN: 继续补充 function dedup_save_plans 的文档字符串内容。
- **L23** EN: Continues the docstring text for the function dedup_save_plans. | CN: 继续补充 function dedup_save_plans 的文档字符串内容。
- **L24** EN: Closes the docstring for the function dedup_save_plans. | CN: 结束 function dedup_save_plans 的文档字符串。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Keeps the inline comment or directive: Map to query the plan indices that a write item is duplicated in | CN: 保留这一行注释或指令：Map to query the plan indices that a write item is duplicated in
- **L27** EN: Assigns or updates `write_item_to_plan_indices`. | CN: 对 `write_item_to_plan_indices` 进行赋值或更新。
- **L28** EN: Keeps the inline comment or directive: Map to query the write item from its index | CN: 保留这一行注释或指令：Map to query the write item from its index
- **L29** EN: Assigns or updates `write_item_idx_to_write_item`. | CN: 对 `write_item_idx_to_write_item` 进行赋值或更新。
- **L30** EN: Keeps the inline comment or directive: Set of write item indices that are present in each plan | CN: 保留这一行注释或指令：Set of write item indices that are present in each plan
- **L31** EN: Keeps the inline comment or directive: After deduplication, this will be the set of write item indices that are present | CN: 保留这一行注释或指令：After deduplication, this will be the set of write item indices that are present
- **L32** EN: Assigns or updates `plan_to_item_indices`. | CN: 对 `plan_to_item_indices` 进行赋值或更新。
- **L33** EN: Continues the implementation inside function `dedup_save_plans`. | CN: 继续说明函数 `dedup_save_plans` 内部的实现。
- **L34** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L35** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L36** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L37** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L38** EN: Keeps the inline comment or directive: map each write item to its plan | CN: 保留这一行注释或指令：map each write item to its plan
- **L39** EN: Continues the implementation inside function `dedup_save_plans`. | CN: 继续说明函数 `dedup_save_plans` 内部的实现。
- **L40** EN: Assigns or updates `write_item_idx_to_write_item[write_item.index]`. | CN: 对 `write_item_idx_to_write_item[write_item.index]` 进行赋值或更新。

### Lines 41-60 / 第 41-60 行

````python
    plan_to_size = [0] * len(all_plans)
    for write_item_idx, plan_indices in write_item_to_plan_indices.items():
        if save_to_lowest_rank:
            select_plan_idx = min(plan_indices)
        else:
            select_plan_idx = min(
                plan_indices, key=lambda plan_idx: plan_to_size[plan_idx]
            )

        write_item = write_item_idx_to_write_item[write_item_idx]
        # Ignore the storage size of anything that is not a tensor, since
        # we don't know how much storage they represent
        plan_to_size[select_plan_idx] += write_item.tensor_storage_size() or 1
        for plan_idx in plan_indices - {select_plan_idx}:
            plan_to_item_indices[plan_idx].discard(write_item_idx)
    # Sanity check
    if len(all_plans) != len(plan_to_item_indices):
        raise AssertionError("len(all_plans) != len(plan_to_item_indices)")
    # Create new plans with the updated write items post deduplication
    return [
````

- **L41** EN: Assigns or updates `plan_to_size`. | CN: 对 `plan_to_size` 进行赋值或更新。
- **L42** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L43** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L44** EN: Assigns or updates `select_plan_idx`. | CN: 对 `select_plan_idx` 进行赋值或更新。
- **L45** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L46** EN: Assigns or updates `select_plan_idx`. | CN: 对 `select_plan_idx` 进行赋值或更新。
- **L47** EN: Assigns or updates `plan_indices, key`. | CN: 对 `plan_indices, key` 进行赋值或更新。
- **L48** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Assigns or updates `write_item`. | CN: 对 `write_item` 进行赋值或更新。
- **L51** EN: Keeps the inline comment or directive: Ignore the storage size of anything that is not a tensor, since | CN: 保留这一行注释或指令：Ignore the storage size of anything that is not a tensor, since
- **L52** EN: Keeps the inline comment or directive: we don't know how much storage they represent | CN: 保留这一行注释或指令：we don't know how much storage they represent
- **L53** EN: Continues the implementation inside function `dedup_save_plans`. | CN: 继续说明函数 `dedup_save_plans` 内部的实现。
- **L54** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L55** EN: Continues the implementation inside function `dedup_save_plans`. | CN: 继续说明函数 `dedup_save_plans` 内部的实现。
- **L56** EN: Keeps the inline comment or directive: Sanity check | CN: 保留这一行注释或指令：Sanity check
- **L57** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L58** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L59** EN: Keeps the inline comment or directive: Create new plans with the updated write items post deduplication | CN: 保留这一行注释或指令：Create new plans with the updated write items post deduplication
- **L60** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 61-65 / 第 61-65 行

````python
        dataclasses.replace(
            plan, items=[item for item in plan.items if item.index in item_indexes]
        )
        for plan, item_indexes in zip(all_plans, plan_to_item_indices)
    ]
````

- **L61** EN: Calls `dataclasses.replace` as part of the current workflow. | CN: 在当前流程中调用 `dataclasses.replace`。
- **L62** EN: Assigns or updates `plan, items`. | CN: 对 `plan, items` 进行赋值或更新。
- **L63** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L64** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L65** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: Core callables: dedup_save_plans  
  **CN**: 核心可调用对象：dedup_save_plans

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.checkpoint.metadata`, `torch.distributed.checkpoint.planner`
- **PyTorch / PyTorch**: None detected / 未检测到
- **Python Stdlib / Python 标准库**: `collections`, `dataclasses`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

