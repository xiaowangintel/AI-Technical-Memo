# stateful_example.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/examples/stateful_example.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include Model, _make_stateful, _train.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 Model, _make_stateful, _train。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
# Owner(s): ["oncall: distributed"]

# pyre-unsafe


import os
import shutil

import torch
import torch.distributed as dist
import torch.distributed.checkpoint as dcp
import torch.multiprocessing as mp
import torch.nn as nn
from torch.distributed.checkpoint.state_dict import (
    _patch_model_state_dict,
    _patch_optimizer_state_dict,
)
from torch.distributed.device_mesh import init_device_mesh
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Keeps the inline comment or directive: Owner(s): ["oncall: distributed"] | CN: 保留这一行注释或指令：Owner(s): ["oncall: distributed"]
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Keeps the inline comment or directive: pyre-unsafe | CN: 保留这一行注释或指令：pyre-unsafe
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L7** EN: Imports module dependencies: `os`. | CN: 导入模块依赖：`os`。
- **L8** EN: Imports module dependencies: `shutil`. | CN: 导入模块依赖：`shutil`。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L11** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L12** EN: Imports module dependencies: `torch.distributed.checkpoint as dcp`. | CN: 导入模块依赖：`torch.distributed.checkpoint as dcp`。
- **L13** EN: Imports module dependencies: `torch.multiprocessing as mp`. | CN: 导入模块依赖：`torch.multiprocessing as mp`。
- **L14** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L15** EN: Imports selected names from `torch.distributed.checkpoint.state_dict`. | CN: 从 `torch.distributed.checkpoint.state_dict` 导入指定名称。
- **L16** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L17** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L18** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L19** EN: Imports selected names from `torch.distributed.device_mesh`. | CN: 从 `torch.distributed.device_mesh` 导入指定名称。
- **L20** EN: Imports selected names from `torch.distributed.fsdp`. | CN: 从 `torch.distributed.fsdp` 导入指定名称。

### Lines 21-40 / 第 21-40 行

````python


CHECKPOINT_DIR = f"~/{os.environ.get('LOGNAME', '')}/checkpoint"


class Model(torch.nn.Module):
    def __init__(self) -> None:
        super().__init__()
        torch.manual_seed(0)
        self.net1 = nn.Sequential(nn.Linear(8, 16), nn.ReLU())
        self.net2 = nn.Sequential(nn.Linear(16, 32), nn.ReLU())
        self.net3 = nn.Linear(32, 64)
        self.net4 = nn.Sequential(nn.ReLU(), nn.Linear(64, 8))

    def forward(self, x):
        return self.net4(self.net3(self.net2(self.net1(x))))

    def get_input(self):
        return torch.rand(8, 8, device="cuda")

````

- **L21** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Assigns or updates `CHECKPOINT_DIR`. | CN: 对 `CHECKPOINT_DIR` 进行赋值或更新。
- **L24** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Defines class `Model`. | CN: 定义类 `Model`。
- **L27** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L28** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L29** EN: Calls `torch.manual_seed` as part of the current workflow. | CN: 在当前流程中调用 `torch.manual_seed`。
- **L30** EN: Assigns or updates `self.net1`. | CN: 对 `self.net1` 进行赋值或更新。
- **L31** EN: Assigns or updates `self.net2`. | CN: 对 `self.net2` 进行赋值或更新。
- **L32** EN: Assigns or updates `self.net3`. | CN: 对 `self.net3` 进行赋值或更新。
- **L33** EN: Assigns or updates `self.net4`. | CN: 对 `self.net4` 进行赋值或更新。
- **L34** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L35** EN: Defines function `forward`. | CN: 定义函数 `forward`。
- **L36** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L37** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L38** EN: Defines function `get_input`. | CN: 定义函数 `get_input`。
- **L39** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L40** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 41-60 / 第 41-60 行

````python

def _make_stateful(model, optim):
    _patch_model_state_dict(model)
    _patch_optimizer_state_dict(model, optimizers=optim)


def _train(model, optim, train_steps=1):
    torch.manual_seed(0)
    loss = None
    for _ in range(train_steps):
        loss = model(model.get_input()).sum()
        loss.backward()
        optim.step()
        optim.zero_grad()

    return loss


def _init_model(device, world_size):
    device_mesh = init_device_mesh(device, (world_size,))
````

- **L41** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L42** EN: Defines function `_make_stateful`. | CN: 定义函数 `_make_stateful`。
- **L43** EN: Calls `_patch_model_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `_patch_model_state_dict`。
- **L44** EN: Calls `_patch_optimizer_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `_patch_optimizer_state_dict`。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L47** EN: Defines function `_train`. | CN: 定义函数 `_train`。
- **L48** EN: Calls `torch.manual_seed` as part of the current workflow. | CN: 在当前流程中调用 `torch.manual_seed`。
- **L49** EN: Assigns or updates `loss`. | CN: 对 `loss` 进行赋值或更新。
- **L50** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L51** EN: Assigns or updates `loss`. | CN: 对 `loss` 进行赋值或更新。
- **L52** EN: Calls `loss.backward` as part of the current workflow. | CN: 在当前流程中调用 `loss.backward`。
- **L53** EN: Calls `optim.step` as part of the current workflow. | CN: 在当前流程中调用 `optim.step`。
- **L54** EN: Calls `optim.zero_grad` as part of the current workflow. | CN: 在当前流程中调用 `optim.zero_grad`。
- **L55** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L56** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L57** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L58** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L59** EN: Defines function `_init_model`. | CN: 定义函数 `_init_model`。
- **L60** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。

### Lines 61-80 / 第 61-80 行

````python
    model = Model().cuda()
    model = FSDP(
        model,
        device_mesh=device_mesh,
        use_orig_params=True,
    )
    optim = torch.optim.Adam(model.parameters(), lr=0.1)
    _make_stateful(model, optim)

    return model, optim


def run(rank, world_size, device="cuda"):
    # Set up world pg
    os.environ["MASTER_ADDR"] = "localhost"
    os.environ["MASTER_PORT"] = "12355"

    dist.init_process_group("cpu:gloo,cuda:nccl", rank=rank, world_size=world_size)
    torch.cuda.set_device(rank)

````

- **L61** EN: Assigns or updates `model`. | CN: 对 `model` 进行赋值或更新。
- **L62** EN: Assigns or updates `model`. | CN: 对 `model` 进行赋值或更新。
- **L63** EN: Continues the implementation inside function `_init_model`. | CN: 继续说明函数 `_init_model` 内部的实现。
- **L64** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L65** EN: Assigns or updates `use_orig_params`. | CN: 对 `use_orig_params` 进行赋值或更新。
- **L66** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L67** EN: Assigns or updates `optim`. | CN: 对 `optim` 进行赋值或更新。
- **L68** EN: Calls `_make_stateful` as part of the current workflow. | CN: 在当前流程中调用 `_make_stateful`。
- **L69** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L70** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L71** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L72** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L73** EN: Defines function `run`. | CN: 定义函数 `run`。
- **L74** EN: Keeps the inline comment or directive: Set up world pg | CN: 保留这一行注释或指令：Set up world pg
- **L75** EN: Continues the implementation inside function `run`. | CN: 继续说明函数 `run` 内部的实现。
- **L76** EN: Continues the implementation inside function `run`. | CN: 继续说明函数 `run` 内部的实现。
- **L77** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L78** EN: Calls `dist.init_process_group` as part of the current workflow. | CN: 在当前流程中调用 `dist.init_process_group`。
- **L79** EN: Calls `torch.cuda.set_device` as part of the current workflow. | CN: 在当前流程中调用 `torch.cuda.set_device`。
- **L80** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 81-100 / 第 81-100 行

````python
    model, optim = _init_model(device, world_size)
    _train(model, optim, train_steps=2)

    dcp.save(
        state_dict={"model": model, "optimizer": optim},
        checkpoint_id=CHECKPOINT_DIR,
    )

    # presumably do something else
    model, optim = _init_model(device, world_size)
    dcp.load(
        state_dict={"model": model, "optimizer": optim},
        checkpoint_id=CHECKPOINT_DIR,
    )
    _train(model, optim, train_steps=2)


if __name__ == "__main__":
    world_size = torch.cuda.device_count()
    print(f"Running stateful checkpoint example on {world_size} devices.")
````

- **L81** EN: Assigns or updates `model, optim`. | CN: 对 `model, optim` 进行赋值或更新。
- **L82** EN: Calls `_train` as part of the current workflow. | CN: 在当前流程中调用 `_train`。
- **L83** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L84** EN: Calls `dcp.save` as part of the current workflow. | CN: 在当前流程中调用 `dcp.save`。
- **L85** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L86** EN: Assigns or updates `checkpoint_id`. | CN: 对 `checkpoint_id` 进行赋值或更新。
- **L87** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L88** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L89** EN: Keeps the inline comment or directive: presumably do something else | CN: 保留这一行注释或指令：presumably do something else
- **L90** EN: Assigns or updates `model, optim`. | CN: 对 `model, optim` 进行赋值或更新。
- **L91** EN: Calls `dcp.load` as part of the current workflow. | CN: 在当前流程中调用 `dcp.load`。
- **L92** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L93** EN: Assigns or updates `checkpoint_id`. | CN: 对 `checkpoint_id` 进行赋值或更新。
- **L94** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L95** EN: Calls `_train` as part of the current workflow. | CN: 在当前流程中调用 `_train`。
- **L96** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L97** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L98** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L99** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L100** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。

### Lines 101-107 / 第 101-107 行

````python
    shutil.rmtree(CHECKPOINT_DIR, ignore_errors=True)
    mp.spawn(
        run,
        args=(world_size,),
        nprocs=world_size,
        join=True,
    )
````

- **L101** EN: Calls `shutil.rmtree` as part of the current workflow. | CN: 在当前流程中调用 `shutil.rmtree`。
- **L102** EN: Calls `mp.spawn` as part of the current workflow. | CN: 在当前流程中调用 `mp.spawn`。
- **L103** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L104** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L105** EN: Assigns or updates `nprocs`. | CN: 对 `nprocs` 进行赋值或更新。
- **L106** EN: Assigns or updates `join`. | CN: 对 `join` 进行赋值或更新。
- **L107** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: process groups  
  **CN**: 进程组
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: optimizer coordination  
  **CN**: 优化器协同
- **EN**: state dict handling  
  **CN**: state_dict 处理
- **EN**: Primary classes: Model  
  **CN**: 主要类：Model

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed`, `torch.distributed.checkpoint`, `torch.distributed.checkpoint.state_dict`, `torch.distributed.device_mesh`, `torch.distributed.fsdp`
- **PyTorch / PyTorch**: `torch`, `torch.multiprocessing`, `torch.nn`
- **Python Stdlib / Python 标准库**: `os`, `shutil`
- **Third-party / 第三方**: None detected / 未检测到

