# fsdp_checkpoint_example.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/examples/fsdp_checkpoint_example.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include opt_at, init_model.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 opt_at, init_model。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates

"""
The following example demonstrates how to use Pytorch Distributed Checkpoint to save a FSDP model.

This is the current recommended way to checkpoint FSDP.
torch.save() and torch.load() is not recommended when checkpointing sharded models.
"""

import os
import shutil

import torch
import torch.distributed as dist
import torch.distributed.checkpoint as dist_cp
import torch.multiprocessing as mp
from torch.distributed.checkpoint.optimizer import load_sharded_optimizer_state_dict
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
from torch.distributed.fsdp.fully_sharded_data_parallel import StateDictType
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Starts the docstring for the module. | CN: 开始定义 module 的文档字符串。
- **L5** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L6** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L7** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L8** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L9** EN: Closes the docstring for the module. | CN: 结束 module 的文档字符串。
- **L10** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L11** EN: Imports module dependencies: `os`. | CN: 导入模块依赖：`os`。
- **L12** EN: Imports module dependencies: `shutil`. | CN: 导入模块依赖：`shutil`。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L15** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L16** EN: Imports module dependencies: `torch.distributed.checkpoint as dist_cp`. | CN: 导入模块依赖：`torch.distributed.checkpoint as dist_cp`。
- **L17** EN: Imports module dependencies: `torch.multiprocessing as mp`. | CN: 导入模块依赖：`torch.multiprocessing as mp`。
- **L18** EN: Imports selected names from `torch.distributed.checkpoint.optimizer`. | CN: 从 `torch.distributed.checkpoint.optimizer` 导入指定名称。
- **L19** EN: Imports selected names from `torch.distributed.fsdp`. | CN: 从 `torch.distributed.fsdp` 导入指定名称。
- **L20** EN: Imports selected names from `torch.distributed.fsdp.fully_sharded_data_parallel`. | CN: 从 `torch.distributed.fsdp.fully_sharded_data_parallel` 导入指定名称。

### Lines 21-40 / 第 21-40 行

````python


CHECKPOINT_DIR = f"/scratch/{os.environ.get('LOGNAME', '')}/checkpoint"


def opt_at(opt, idx):
    return list(opt.state.values())[idx]


def init_model():
    model = FSDP(torch.nn.Linear(4, 4).cuda(dist.get_rank()))
    optim = torch.optim.Adam(model.parameters(), lr=0.1)
    model(torch.rand(4, 4)).sum().backward()
    optim.step()

    return model, optim


def print_params(stage, model_1, model_2, optim_1, optim_2):
    with FSDP.summon_full_params(model_1), FSDP.summon_full_params(model_2):
````

- **L21** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Assigns or updates `CHECKPOINT_DIR`. | CN: 对 `CHECKPOINT_DIR` 进行赋值或更新。
- **L24** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Defines function `opt_at`. | CN: 定义函数 `opt_at`。
- **L27** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L28** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Defines function `init_model`. | CN: 定义函数 `init_model`。
- **L31** EN: Assigns or updates `model`. | CN: 对 `model` 进行赋值或更新。
- **L32** EN: Assigns or updates `optim`. | CN: 对 `optim` 进行赋值或更新。
- **L33** EN: Calls `model` as part of the current workflow. | CN: 在当前流程中调用 `model`。
- **L34** EN: Calls `optim.step` as part of the current workflow. | CN: 在当前流程中调用 `optim.step`。
- **L35** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L36** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L37** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L38** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L39** EN: Defines function `print_params`. | CN: 定义函数 `print_params`。
- **L40** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。

### Lines 41-60 / 第 41-60 行

````python
        print(
            f"{stage} --- rank: {dist.get_rank()}\n"
            f"model.weight: {model_1.weight}\n"
            f"model_2.weight:{model_2.weight}\n"
            f"model.bias: {model_1.bias}\n"
            f"model_2.bias: {model_2.bias}\n"
        )

    print(
        f"{stage} --- rank: {dist.get_rank()}\n"
        f"optim exp_avg:{opt_at(optim_1, 0)['exp_avg']}\n"
        f"optim_2 exp_avg:{opt_at(optim_2, 0)['exp_avg']}\n"
        f"optim exp_avg_sq:{opt_at(optim_1, 0)['exp_avg_sq']}\n"
        f"optim_2 exp_avg_sq:{opt_at(optim_2, 0)['exp_avg_sq']}\n"
    )


def run_fsdp_checkpoint_example(rank, world_size):
    # Set up world pg
    os.environ["MASTER_ADDR"] = "localhost"
````

- **L41** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L42** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L43** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L44** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L45** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L46** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L47** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L50** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L51** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L52** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L53** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L54** EN: Continues the implementation inside function `print_params`. | CN: 继续说明函数 `print_params` 内部的实现。
- **L55** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L56** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L57** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L58** EN: Defines function `run_fsdp_checkpoint_example`. | CN: 定义函数 `run_fsdp_checkpoint_example`。
- **L59** EN: Keeps the inline comment or directive: Set up world pg | CN: 保留这一行注释或指令：Set up world pg
- **L60** EN: Continues the implementation inside function `run_fsdp_checkpoint_example`. | CN: 继续说明函数 `run_fsdp_checkpoint_example` 内部的实现。

### Lines 61-80 / 第 61-80 行

````python
    os.environ["MASTER_PORT"] = "12355"

    # Initialize the process group
    dist.init_process_group("cpu:gloo,cuda:nccl", rank=rank, world_size=world_size)
    torch.cuda.set_device(rank)

    # Create a model
    model_1, optim_1 = init_model()

    # Save the model to CHECKPOINT_DIR
    with FSDP.state_dict_type(model_1, StateDictType.SHARDED_STATE_DICT):
        state_dict = {
            "model": model_1.state_dict(),
            "optim": FSDP.optim_state_dict(model_1, optim_1),
        }

        dist_cp.save_state_dict(
            state_dict=state_dict,
            storage_writer=dist_cp.FileSystemWriter(CHECKPOINT_DIR),
        )
````

- **L61** EN: Continues the implementation inside function `run_fsdp_checkpoint_example`. | CN: 继续说明函数 `run_fsdp_checkpoint_example` 内部的实现。
- **L62** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L63** EN: Keeps the inline comment or directive: Initialize the process group | CN: 保留这一行注释或指令：Initialize the process group
- **L64** EN: Calls `dist.init_process_group` as part of the current workflow. | CN: 在当前流程中调用 `dist.init_process_group`。
- **L65** EN: Calls `torch.cuda.set_device` as part of the current workflow. | CN: 在当前流程中调用 `torch.cuda.set_device`。
- **L66** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L67** EN: Keeps the inline comment or directive: Create a model | CN: 保留这一行注释或指令：Create a model
- **L68** EN: Assigns or updates `model_1, optim_1`. | CN: 对 `model_1, optim_1` 进行赋值或更新。
- **L69** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L70** EN: Keeps the inline comment or directive: Save the model to CHECKPOINT_DIR | CN: 保留这一行注释或指令：Save the model to CHECKPOINT_DIR
- **L71** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L72** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L73** EN: Continues the implementation inside function `run_fsdp_checkpoint_example`. | CN: 继续说明函数 `run_fsdp_checkpoint_example` 内部的实现。
- **L74** EN: Continues the implementation inside function `run_fsdp_checkpoint_example`. | CN: 继续说明函数 `run_fsdp_checkpoint_example` 内部的实现。
- **L75** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L76** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L77** EN: Calls `dist_cp.save_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `dist_cp.save_state_dict`。
- **L78** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L79** EN: Assigns or updates `storage_writer`. | CN: 对 `storage_writer` 进行赋值或更新。
- **L80** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 81-100 / 第 81-100 行

````python

    # Create a second model
    model_2, optim_2 = init_model()

    # Print the model parameters for both models.
    # Before loading, the parameters should be different.
    print_params("Before loading", model_1, model_2, optim_1, optim_2)

    # Load model_2 with parameters saved in CHECKPOINT_DIR
    with FSDP.state_dict_type(model_2, StateDictType.SHARDED_STATE_DICT):
        state_dict = {
            "model": model_2.state_dict(),
            # cannot load the optimizer state_dict together with the model state_dict
        }

        dist_cp.load_state_dict(
            state_dict=state_dict,
            storage_reader=dist_cp.FileSystemReader(CHECKPOINT_DIR),
        )
        model_2.load_state_dict(state_dict["model"])
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Keeps the inline comment or directive: Create a second model | CN: 保留这一行注释或指令：Create a second model
- **L83** EN: Assigns or updates `model_2, optim_2`. | CN: 对 `model_2, optim_2` 进行赋值或更新。
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Keeps the inline comment or directive: Print the model parameters for both models. | CN: 保留这一行注释或指令：Print the model parameters for both models.
- **L86** EN: Keeps the inline comment or directive: Before loading, the parameters should be different. | CN: 保留这一行注释或指令：Before loading, the parameters should be different.
- **L87** EN: Calls `print_params` as part of the current workflow. | CN: 在当前流程中调用 `print_params`。
- **L88** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L89** EN: Keeps the inline comment or directive: Load model_2 with parameters saved in CHECKPOINT_DIR | CN: 保留这一行注释或指令：Load model_2 with parameters saved in CHECKPOINT_DIR
- **L90** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L91** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L92** EN: Continues the implementation inside function `run_fsdp_checkpoint_example`. | CN: 继续说明函数 `run_fsdp_checkpoint_example` 内部的实现。
- **L93** EN: Keeps the inline comment or directive: cannot load the optimizer state_dict together with the model state_dict | CN: 保留这一行注释或指令：cannot load the optimizer state_dict together with the model state_dict
- **L94** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L95** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L96** EN: Calls `dist_cp.load_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `dist_cp.load_state_dict`。
- **L97** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L98** EN: Assigns or updates `storage_reader`. | CN: 对 `storage_reader` 进行赋值或更新。
- **L99** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L100** EN: Calls `model_2.load_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `model_2.load_state_dict`。

### Lines 101-120 / 第 101-120 行

````python

        optim_state = load_sharded_optimizer_state_dict(
            model_state_dict=state_dict["model"],
            optimizer_key="optim",
            storage_reader=dist_cp.FileSystemReader(CHECKPOINT_DIR),
        )

        flattened_osd = FSDP.optim_state_dict_to_load(
            model_2, optim_2, optim_state["optim"]
        )
        optim_2.load_state_dict(flattened_osd)

    # Print the model parameters for both models.
    # After loading, the parameters should be the same.
    print_params("After loading", model_1, model_2, optim_1, optim_2)

    # Shut down world pg
    dist.destroy_process_group()


````

- **L101** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L102** EN: Assigns or updates `optim_state`. | CN: 对 `optim_state` 进行赋值或更新。
- **L103** EN: Assigns or updates `model_state_dict`. | CN: 对 `model_state_dict` 进行赋值或更新。
- **L104** EN: Assigns or updates `optimizer_key`. | CN: 对 `optimizer_key` 进行赋值或更新。
- **L105** EN: Assigns or updates `storage_reader`. | CN: 对 `storage_reader` 进行赋值或更新。
- **L106** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L107** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L108** EN: Assigns or updates `flattened_osd`. | CN: 对 `flattened_osd` 进行赋值或更新。
- **L109** EN: Continues the implementation inside function `run_fsdp_checkpoint_example`. | CN: 继续说明函数 `run_fsdp_checkpoint_example` 内部的实现。
- **L110** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L111** EN: Calls `optim_2.load_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `optim_2.load_state_dict`。
- **L112** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L113** EN: Keeps the inline comment or directive: Print the model parameters for both models. | CN: 保留这一行注释或指令：Print the model parameters for both models.
- **L114** EN: Keeps the inline comment or directive: After loading, the parameters should be the same. | CN: 保留这一行注释或指令：After loading, the parameters should be the same.
- **L115** EN: Calls `print_params` as part of the current workflow. | CN: 在当前流程中调用 `print_params`。
- **L116** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L117** EN: Keeps the inline comment or directive: Shut down world pg | CN: 保留这一行注释或指令：Shut down world pg
- **L118** EN: Calls `dist.destroy_process_group` as part of the current workflow. | CN: 在当前流程中调用 `dist.destroy_process_group`。
- **L119** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L120** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 121-130 / 第 121-130 行

````python
if __name__ == "__main__":
    world_size = torch.cuda.device_count()
    print(f"Running fsdp checkpoint example on {world_size} devices.")
    shutil.rmtree(CHECKPOINT_DIR, ignore_errors=True)
    mp.spawn(
        run_fsdp_checkpoint_example,
        args=(world_size,),
        nprocs=world_size,
        join=True,
    )
````

- **L121** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L122** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L123** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L124** EN: Calls `shutil.rmtree` as part of the current workflow. | CN: 在当前流程中调用 `shutil.rmtree`。
- **L125** EN: Calls `mp.spawn` as part of the current workflow. | CN: 在当前流程中调用 `mp.spawn`。
- **L126** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L127** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L128** EN: Assigns or updates `nprocs`. | CN: 对 `nprocs` 进行赋值或更新。
- **L129** EN: Assigns or updates `join`. | CN: 对 `join` 进行赋值或更新。
- **L130** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: process groups  
  **CN**: 进程组
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: optimizer coordination  
  **CN**: 优化器协同
- **EN**: state dict handling  
  **CN**: state_dict 处理
- **EN**: Core callables: opt_at, init_model, print_params, run_fsdp_checkpoint_example  
  **CN**: 核心可调用对象：opt_at, init_model, print_params, run_fsdp_checkpoint_example

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed`, `torch.distributed.checkpoint`, `torch.distributed.checkpoint.optimizer`, `torch.distributed.fsdp`, `torch.distributed.fsdp.fully_sharded_data_parallel`
- **PyTorch / PyTorch**: `torch`, `torch.multiprocessing`
- **Python Stdlib / Python 标准库**: `os`, `shutil`
- **Third-party / 第三方**: None detected / 未检测到

