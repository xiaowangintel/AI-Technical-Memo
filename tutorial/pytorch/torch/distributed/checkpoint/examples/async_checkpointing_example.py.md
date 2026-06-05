# async_checkpointing_example.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/examples/async_checkpointing_example.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include InjectedException, Model, _init_model, _print.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 InjectedException, Model, _init_model, _print。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
# Owner(s): ["oncall: distributed"]

import os
import shutil
import traceback
from concurrent.futures import Future

import torch
import torch.distributed as dist
import torch.distributed.checkpoint as dcp
import torch.multiprocessing as mp
import torch.nn as nn
import torch.nn.functional as F
from torch.distributed.checkpoint.state_dict import (
    _patch_model_state_dict,
    _patch_optimizer_state_dict,
)
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
from torch.distributed.tensor.device_mesh import init_device_mesh
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Keeps the inline comment or directive: Owner(s): ["oncall: distributed"] | CN: 保留这一行注释或指令：Owner(s): ["oncall: distributed"]
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Imports module dependencies: `os`. | CN: 导入模块依赖：`os`。
- **L5** EN: Imports module dependencies: `shutil`. | CN: 导入模块依赖：`shutil`。
- **L6** EN: Imports module dependencies: `traceback`. | CN: 导入模块依赖：`traceback`。
- **L7** EN: Imports selected names from `concurrent.futures`. | CN: 从 `concurrent.futures` 导入指定名称。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L10** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L11** EN: Imports module dependencies: `torch.distributed.checkpoint as dcp`. | CN: 导入模块依赖：`torch.distributed.checkpoint as dcp`。
- **L12** EN: Imports module dependencies: `torch.multiprocessing as mp`. | CN: 导入模块依赖：`torch.multiprocessing as mp`。
- **L13** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L14** EN: Imports module dependencies: `torch.nn.functional as F`. | CN: 导入模块依赖：`torch.nn.functional as F`。
- **L15** EN: Imports selected names from `torch.distributed.checkpoint.state_dict`. | CN: 从 `torch.distributed.checkpoint.state_dict` 导入指定名称。
- **L16** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L17** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L18** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L19** EN: Imports selected names from `torch.distributed.fsdp`. | CN: 从 `torch.distributed.fsdp` 导入指定名称。
- **L20** EN: Imports selected names from `torch.distributed.tensor.device_mesh`. | CN: 从 `torch.distributed.tensor.device_mesh` 导入指定名称。

### Lines 21-40 / 第 21-40 行

````python


DEVICE = "cuda"
NUM_EPOCHS = 1000
SAVE_PERIOD = 10
FAULT_PERIOD = 25
CHECKPOINT_DIR = f"~/{os.environ.get('LOGNAME', '')}/checkpoint"


class InjectedException(Exception):
    pass


class Model(torch.nn.Module):
    def __init__(self) -> None:
        super().__init__()
        self.net1 = nn.Linear(8, 32)
        self.net2 = nn.Linear(32, 128)
        self.net3 = nn.Linear(128, 64)
        self.net4 = nn.Linear(64, 8)
````

- **L21** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Assigns or updates `DEVICE`. | CN: 对 `DEVICE` 进行赋值或更新。
- **L24** EN: Assigns or updates `NUM_EPOCHS`. | CN: 对 `NUM_EPOCHS` 进行赋值或更新。
- **L25** EN: Assigns or updates `SAVE_PERIOD`. | CN: 对 `SAVE_PERIOD` 进行赋值或更新。
- **L26** EN: Assigns or updates `FAULT_PERIOD`. | CN: 对 `FAULT_PERIOD` 进行赋值或更新。
- **L27** EN: Assigns or updates `CHECKPOINT_DIR`. | CN: 对 `CHECKPOINT_DIR` 进行赋值或更新。
- **L28** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Defines class `InjectedException`. | CN: 定义类 `InjectedException`。
- **L31** EN: Keeps an intentionally empty block. | CN: 保留一个有意为空的代码块。
- **L32** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L33** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L34** EN: Defines class `Model`. | CN: 定义类 `Model`。
- **L35** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L36** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L37** EN: Assigns or updates `self.net1`. | CN: 对 `self.net1` 进行赋值或更新。
- **L38** EN: Assigns or updates `self.net2`. | CN: 对 `self.net2` 进行赋值或更新。
- **L39** EN: Assigns or updates `self.net3`. | CN: 对 `self.net3` 进行赋值或更新。
- **L40** EN: Assigns or updates `self.net4`. | CN: 对 `self.net4` 进行赋值或更新。

### Lines 41-60 / 第 41-60 行

````python
        self.net5 = nn.Linear(8, 1)

    def forward(self, x):
        x = F.relu(self.net1(x))
        x = F.relu(self.net2(x))
        x = F.relu(self.net3(x))
        x = F.relu(self.net4(x))
        x = F.sigmoid(self.net5(x))
        return x


def _init_model(rank, world_size):
    device_mesh = init_device_mesh(DEVICE, (world_size,))

    # Create a dummy model and wrap it in FSDP
    model = Model().cuda()
    device_mesh = init_device_mesh(DEVICE, (world_size,))
    model = FSDP(model, device_mesh=device_mesh, use_orig_params=True)

    optim = torch.optim.Adam(model.parameters(), lr=0.0001)
````

- **L41** EN: Assigns or updates `self.net5`. | CN: 对 `self.net5` 进行赋值或更新。
- **L42** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L43** EN: Defines function `forward`. | CN: 定义函数 `forward`。
- **L44** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L45** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L46** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L47** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L48** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L49** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L50** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L51** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L52** EN: Defines function `_init_model`. | CN: 定义函数 `_init_model`。
- **L53** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L54** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L55** EN: Keeps the inline comment or directive: Create a dummy model and wrap it in FSDP | CN: 保留这一行注释或指令：Create a dummy model and wrap it in FSDP
- **L56** EN: Assigns or updates `model`. | CN: 对 `model` 进行赋值或更新。
- **L57** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L58** EN: Assigns or updates `model`. | CN: 对 `model` 进行赋值或更新。
- **L59** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L60** EN: Assigns or updates `optim`. | CN: 对 `optim` 进行赋值或更新。

### Lines 61-80 / 第 61-80 行

````python

    _patch_model_state_dict(model)
    # pyrefly: ignore [bad-argument-type]
    _patch_optimizer_state_dict(model, optimizers=optim)

    return model, optim


def _print(msg):
    if dist.get_rank() == 0:
        print(msg)


def _input():
    x = torch.rand(128, 8, device="cuda")
    y = torch.zeros(128, 1, device="cuda")

    y[torch.sum(x, dim=1) >= 4] = 1.0

    return x, y
````

- **L61** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L62** EN: Calls `_patch_model_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `_patch_model_state_dict`。
- **L63** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-type] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-type]
- **L64** EN: Calls `_patch_optimizer_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `_patch_optimizer_state_dict`。
- **L65** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L66** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L67** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L68** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L69** EN: Defines function `_print`. | CN: 定义函数 `_print`。
- **L70** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L71** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L72** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L73** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L74** EN: Defines function `_input`. | CN: 定义函数 `_input`。
- **L75** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L76** EN: Assigns or updates `y`. | CN: 对 `y` 进行赋值或更新。
- **L77** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L78** EN: Continues the implementation inside function `_input`. | CN: 继续说明函数 `_input` 内部的实现。
- **L79** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L80** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 81-100 / 第 81-100 行

````python


def run(rank, world_size):
    # Set up world pg
    os.environ["MASTER_ADDR"] = "localhost"
    os.environ["MASTER_PORT"] = "12355"

    dist.init_process_group("cpu:gloo,cuda:nccl", rank=rank, world_size=world_size)
    torch.cuda.set_device(rank)

    model, optim = _init_model(rank, world_size)
    state_dict = {"model": model, "optim": optim}
    loss_calc = torch.nn.BCELoss()

    f = None
    # pyrefly: ignore [bad-assignment]
    for epoch in range(NUM_EPOCHS):
        try:
            torch.manual_seed(epoch)
            x, y = _input()
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L83** EN: Defines function `run`. | CN: 定义函数 `run`。
- **L84** EN: Keeps the inline comment or directive: Set up world pg | CN: 保留这一行注释或指令：Set up world pg
- **L85** EN: Continues the implementation inside function `run`. | CN: 继续说明函数 `run` 内部的实现。
- **L86** EN: Continues the implementation inside function `run`. | CN: 继续说明函数 `run` 内部的实现。
- **L87** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L88** EN: Calls `dist.init_process_group` as part of the current workflow. | CN: 在当前流程中调用 `dist.init_process_group`。
- **L89** EN: Calls `torch.cuda.set_device` as part of the current workflow. | CN: 在当前流程中调用 `torch.cuda.set_device`。
- **L90** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L91** EN: Assigns or updates `model, optim`. | CN: 对 `model, optim` 进行赋值或更新。
- **L92** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L93** EN: Assigns or updates `loss_calc`. | CN: 对 `loss_calc` 进行赋值或更新。
- **L94** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L95** EN: Assigns or updates `f`. | CN: 对 `f` 进行赋值或更新。
- **L96** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-assignment] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-assignment]
- **L97** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L98** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L99** EN: Calls `torch.manual_seed` as part of the current workflow. | CN: 在当前流程中调用 `torch.manual_seed`。
- **L100** EN: Assigns or updates `x, y`. | CN: 对 `x, y` 进行赋值或更新。

### Lines 101-120 / 第 101-120 行

````python

            loss = loss_calc(model(x), y)

            _print(f"{epoch=} {loss=}")

            loss.backward()
            optim.step()
            optim.zero_grad()

            if epoch % SAVE_PERIOD == 0:
                if f is not None:
                    if not isinstance(f, Future):
                        raise AssertionError("f should be a Future instance")
                    f.result()
                f = dcp.state_dict_saver.async_save(
                    state_dict, checkpoint_id=CHECKPOINT_DIR
                )

            if FAULT_PERIOD > 0 and epoch % FAULT_PERIOD == 0:
                raise InjectedException("Fault injection!")
````

- **L101** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L102** EN: Assigns or updates `loss`. | CN: 对 `loss` 进行赋值或更新。
- **L103** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L104** EN: Calls `_print` as part of the current workflow. | CN: 在当前流程中调用 `_print`。
- **L105** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L106** EN: Calls `loss.backward` as part of the current workflow. | CN: 在当前流程中调用 `loss.backward`。
- **L107** EN: Calls `optim.step` as part of the current workflow. | CN: 在当前流程中调用 `optim.step`。
- **L108** EN: Calls `optim.zero_grad` as part of the current workflow. | CN: 在当前流程中调用 `optim.zero_grad`。
- **L109** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L110** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L111** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L112** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L113** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L114** EN: Calls `f.result` as part of the current workflow. | CN: 在当前流程中调用 `f.result`。
- **L115** EN: Assigns or updates `f`. | CN: 对 `f` 进行赋值或更新。
- **L116** EN: Assigns or updates `state_dict, checkpoint_id`. | CN: 对 `state_dict, checkpoint_id` 进行赋值或更新。
- **L117** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L118** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L119** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L120** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 121-140 / 第 121-140 行

````python

        except InjectedException as e:
            dist.barrier()

            _print("Trainer encountered exception:")
            traceback.print_tb(e.__traceback__)

            _print("Reloading model from last checkpoint!")
            if f is not None:
                if not isinstance(f, Future):
                    raise AssertionError("f should be a Future instance") from None
                f.result()
            dcp.load(state_dict)


if __name__ == "__main__":
    world_size = torch.cuda.device_count()
    print(f"Running an example of Async Checkpointing on {world_size} devices.")
    shutil.rmtree(CHECKPOINT_DIR, ignore_errors=True)

````

- **L121** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L122** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L123** EN: Calls `dist.barrier` as part of the current workflow. | CN: 在当前流程中调用 `dist.barrier`。
- **L124** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L125** EN: Calls `_print` as part of the current workflow. | CN: 在当前流程中调用 `_print`。
- **L126** EN: Calls `traceback.print_tb` as part of the current workflow. | CN: 在当前流程中调用 `traceback.print_tb`。
- **L127** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L128** EN: Calls `_print` as part of the current workflow. | CN: 在当前流程中调用 `_print`。
- **L129** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L130** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L131** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L132** EN: Calls `f.result` as part of the current workflow. | CN: 在当前流程中调用 `f.result`。
- **L133** EN: Calls `dcp.load` as part of the current workflow. | CN: 在当前流程中调用 `dcp.load`。
- **L134** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L135** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L136** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L137** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L138** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L139** EN: Calls `shutil.rmtree` as part of the current workflow. | CN: 在当前流程中调用 `shutil.rmtree`。
- **L140** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 141-146 / 第 141-146 行

````python
    mp.spawn(
        run,
        args=(world_size,),
        nprocs=world_size,
        join=True,
    )
````

- **L141** EN: Calls `mp.spawn` as part of the current workflow. | CN: 在当前流程中调用 `mp.spawn`。
- **L142** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L143** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L144** EN: Assigns or updates `nprocs`. | CN: 对 `nprocs` 进行赋值或更新。
- **L145** EN: Assigns or updates `join`. | CN: 对 `join` 进行赋值或更新。
- **L146** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

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
- **EN**: Primary classes: InjectedException, Model  
  **CN**: 主要类：InjectedException, Model

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed`, `torch.distributed.checkpoint`, `torch.distributed.checkpoint.state_dict`, `torch.distributed.fsdp`, `torch.distributed.tensor.device_mesh`
- **PyTorch / PyTorch**: `torch`, `torch.multiprocessing`, `torch.nn`, `torch.nn.functional`
- **Python Stdlib / Python 标准库**: `concurrent.futures`, `os`, `shutil`, `traceback`
- **Third-party / 第三方**: None detected / 未检测到

