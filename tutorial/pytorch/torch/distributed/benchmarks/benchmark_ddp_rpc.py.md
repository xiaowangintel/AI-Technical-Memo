# benchmark_ddp_rpc.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/benchmarks/benchmark_ddp_rpc.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed runtime helpers, APIs, and package wiring. Its main entry points include HybridModel, _retrieve_embedding_parameters, _print_header.
- **用途 (CN)**: 该模块聚焦于分布式运行时辅助逻辑、API 与包级导出，其主要入口包括 HybridModel, _retrieve_embedding_parameters, _print_header。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs

# pyre-unsafe
import argparse
import io
import os
import random
import shlex
import subprocess
import time

import numpy as np

import torch
import torch.distributed as dist
import torch.distributed.autograd as dist_autograd
import torch.distributed.rpc as rpc
import torch.multiprocessing as mp
import torch.nn as nn
import torch.optim as optim
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L3** EN: Keeps the inline comment or directive: pyre-unsafe | CN: 保留这一行注释或指令：pyre-unsafe
- **L4** EN: Imports module dependencies: `argparse`. | CN: 导入模块依赖：`argparse`。
- **L5** EN: Imports module dependencies: `io`. | CN: 导入模块依赖：`io`。
- **L6** EN: Imports module dependencies: `os`. | CN: 导入模块依赖：`os`。
- **L7** EN: Imports module dependencies: `random`. | CN: 导入模块依赖：`random`。
- **L8** EN: Imports module dependencies: `shlex`. | CN: 导入模块依赖：`shlex`。
- **L9** EN: Imports module dependencies: `subprocess`. | CN: 导入模块依赖：`subprocess`。
- **L10** EN: Imports module dependencies: `time`. | CN: 导入模块依赖：`time`。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Imports module dependencies: `numpy as np`. | CN: 导入模块依赖：`numpy as np`。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L15** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L16** EN: Imports module dependencies: `torch.distributed.autograd as dist_autograd`. | CN: 导入模块依赖：`torch.distributed.autograd as dist_autograd`。
- **L17** EN: Imports module dependencies: `torch.distributed.rpc as rpc`. | CN: 导入模块依赖：`torch.distributed.rpc as rpc`。
- **L18** EN: Imports module dependencies: `torch.multiprocessing as mp`. | CN: 导入模块依赖：`torch.multiprocessing as mp`。
- **L19** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L20** EN: Imports module dependencies: `torch.optim as optim`. | CN: 导入模块依赖：`torch.optim as optim`。

### Lines 21-40 / 第 21-40 行

````python
from torch.distributed.optim import DistributedOptimizer
from torch.distributed.rpc import RRef, TensorPipeRpcBackendOptions
from torch.distributed.rpc.backend_registry import BackendType
from torch.nn.parallel import DistributedDataParallel as DDP


# Config
NUM_TRAINERS = 8
NUM_PS = 8

NUM_EMBEDDINGS = 300
EMBEDDING_DIM = 64

WARMUP_CYCLES = 5


class HybridModel(torch.nn.Module):
    r"""
    The model consists of a sparse part and a dense part.

````

- **L21** EN: Imports selected names from `torch.distributed.optim`. | CN: 从 `torch.distributed.optim` 导入指定名称。
- **L22** EN: Imports selected names from `torch.distributed.rpc`. | CN: 从 `torch.distributed.rpc` 导入指定名称。
- **L23** EN: Imports selected names from `torch.distributed.rpc.backend_registry`. | CN: 从 `torch.distributed.rpc.backend_registry` 导入指定名称。
- **L24** EN: Imports selected names from `torch.nn.parallel`. | CN: 从 `torch.nn.parallel` 导入指定名称。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Keeps the inline comment or directive: Config | CN: 保留这一行注释或指令：Config
- **L28** EN: Assigns or updates `NUM_TRAINERS`. | CN: 对 `NUM_TRAINERS` 进行赋值或更新。
- **L29** EN: Assigns or updates `NUM_PS`. | CN: 对 `NUM_PS` 进行赋值或更新。
- **L30** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L31** EN: Assigns or updates `NUM_EMBEDDINGS`. | CN: 对 `NUM_EMBEDDINGS` 进行赋值或更新。
- **L32** EN: Assigns or updates `EMBEDDING_DIM`. | CN: 对 `EMBEDDING_DIM` 进行赋值或更新。
- **L33** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L34** EN: Assigns or updates `WARMUP_CYCLES`. | CN: 对 `WARMUP_CYCLES` 进行赋值或更新。
- **L35** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L36** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L37** EN: Defines class `HybridModel`. | CN: 定义类 `HybridModel`。
- **L38** EN: Starts the docstring for the class HybridModel. | CN: 开始定义 class HybridModel 的文档字符串。
- **L39** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。
- **L40** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
    The dense part is an nn.Linear module that is replicated across all trainers using
    DistributedDataParallel. The sparse part has nn.EmbeddingBags stored on multiple
    parameter servers.

    The model holds a Remote Reference to the embedding tables on the parameter
    servers.
    """

    def __init__(self, emb_rref_list, device):
        super().__init__()
        self.emb_rref_list = emb_rref_list
        fc1 = torch.nn.Linear(512, 256)
        fc2 = torch.nn.Linear(256, 128)
        relu = torch.nn.ReLU()
        fc3 = torch.nn.Linear(128, 64)
        fc4 = torch.nn.Linear(64, 32)
        fc5 = torch.nn.Linear(32, 8)
        sec = nn.Sequential(fc1, fc2, relu, fc3, fc4, fc5)
        self.ddp = DDP(sec.to(device), device_ids=[device])
        self.device = device
````

- **L41** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。
- **L42** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。
- **L43** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。
- **L44** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。
- **L45** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。
- **L46** EN: Continues the docstring text for the class HybridModel. | CN: 继续补充 class HybridModel 的文档字符串内容。
- **L47** EN: Closes the docstring for the class HybridModel. | CN: 结束 class HybridModel 的文档字符串。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L50** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L51** EN: Assigns or updates `self.emb_rref_list`. | CN: 对 `self.emb_rref_list` 进行赋值或更新。
- **L52** EN: Assigns or updates `fc1`. | CN: 对 `fc1` 进行赋值或更新。
- **L53** EN: Assigns or updates `fc2`. | CN: 对 `fc2` 进行赋值或更新。
- **L54** EN: Assigns or updates `relu`. | CN: 对 `relu` 进行赋值或更新。
- **L55** EN: Assigns or updates `fc3`. | CN: 对 `fc3` 进行赋值或更新。
- **L56** EN: Assigns or updates `fc4`. | CN: 对 `fc4` 进行赋值或更新。
- **L57** EN: Assigns or updates `fc5`. | CN: 对 `fc5` 进行赋值或更新。
- **L58** EN: Assigns or updates `sec`. | CN: 对 `sec` 进行赋值或更新。
- **L59** EN: Assigns or updates `self.ddp`. | CN: 对 `self.ddp` 进行赋值或更新。
- **L60** EN: Assigns or updates `self.device`. | CN: 对 `self.device` 进行赋值或更新。

### Lines 61-80 / 第 61-80 行

````python

    def forward(self, indices, offsets):
        emb_lookups = []

        for emb_rref in self.emb_rref_list:
            emb_lookups.append(
                emb_rref.rpc_sync().forward(
                    indices, offsets
                )  # embedding_sum(input, offsets)
            )
            emb_lookups_cat = torch.cat(emb_lookups, dim=1)

        # Make sure combined PS dimension is always bigger or equal than the FC input
        if NUM_PS * EMBEDDING_DIM < 512:
            raise AssertionError
        dim_normalizer = int(NUM_PS * EMBEDDING_DIM / 512)
        emb_lookups_reshaped = emb_lookups_cat.reshape(  # type: ignore[possibly-undefined]
            # pyrefly: ignore [unbound-name]
            [emb_lookups_cat.shape[0] * dim_normalizer, 512]
        )
````

- **L61** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L62** EN: Defines function `forward`. | CN: 定义函数 `forward`。
- **L63** EN: Assigns or updates `emb_lookups`. | CN: 对 `emb_lookups` 进行赋值或更新。
- **L64** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L65** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L66** EN: Calls `emb_lookups.append` as part of the current workflow. | CN: 在当前流程中调用 `emb_lookups.append`。
- **L67** EN: Calls `emb_rref.rpc_sync` as part of the current workflow. | CN: 在当前流程中调用 `emb_rref.rpc_sync`。
- **L68** EN: Continues the implementation inside function `forward`. | CN: 继续说明函数 `forward` 内部的实现。
- **L69** EN: Continues the implementation inside function `forward`. | CN: 继续说明函数 `forward` 内部的实现。
- **L70** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L71** EN: Assigns or updates `emb_lookups_cat`. | CN: 对 `emb_lookups_cat` 进行赋值或更新。
- **L72** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L73** EN: Keeps the inline comment or directive: Make sure combined PS dimension is always bigger or equal than the FC input | CN: 保留这一行注释或指令：Make sure combined PS dimension is always bigger or equal than the FC input
- **L74** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L75** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L76** EN: Assigns or updates `dim_normalizer`. | CN: 对 `dim_normalizer` 进行赋值或更新。
- **L77** EN: Assigns or updates `emb_lookups_reshaped`. | CN: 对 `emb_lookups_reshaped` 进行赋值或更新。
- **L78** EN: Keeps the inline comment or directive: pyrefly: ignore [unbound-name] | CN: 保留这一行注释或指令：pyrefly: ignore [unbound-name]
- **L79** EN: Continues the implementation inside function `forward`. | CN: 继续说明函数 `forward` 内部的实现。
- **L80** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 81-100 / 第 81-100 行

````python

        return self.ddp(emb_lookups_reshaped)


def _retrieve_embedding_parameters(emb_rref):
    return [RRef(p) for p in emb_rref.local_value().parameters()]


def _print_header():
    _print_cont("\n")
    _print_cont(" " * 10)
    for _ in [50, 75, 90, 95]:
        _print_cont(f"{'sec/epoch':14s}{'epoch/sec':10s}")
    _print_cont("\n")


def _print_benchmark(prefix, nelem, measurements):
    measurements = sorted(measurements)
    _print_cont(f"{prefix:8s}:")
    for p in [50, 75, 90, 95]:
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L83** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Defines function `_retrieve_embedding_parameters`. | CN: 定义函数 `_retrieve_embedding_parameters`。
- **L86** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L87** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L88** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L89** EN: Defines function `_print_header`. | CN: 定义函数 `_print_header`。
- **L90** EN: Calls `_print_cont` as part of the current workflow. | CN: 在当前流程中调用 `_print_cont`。
- **L91** EN: Calls `_print_cont` as part of the current workflow. | CN: 在当前流程中调用 `_print_cont`。
- **L92** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L93** EN: Calls `_print_cont` as part of the current workflow. | CN: 在当前流程中调用 `_print_cont`。
- **L94** EN: Calls `_print_cont` as part of the current workflow. | CN: 在当前流程中调用 `_print_cont`。
- **L95** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L96** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L97** EN: Defines function `_print_benchmark`. | CN: 定义函数 `_print_benchmark`。
- **L98** EN: Assigns or updates `measurements`. | CN: 对 `measurements` 进行赋值或更新。
- **L99** EN: Calls `_print_cont` as part of the current workflow. | CN: 在当前流程中调用 `_print_cont`。
- **L100** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 101-120 / 第 101-120 行

````python
        v = np.percentile(measurements, p)
        _print_cont(f"  p{p:02d}:  {v:1.3f}s  {nelem / v:6d}/s")
    _print_cont("\n")


def _print_cont(msg):
    print(msg, end="", flush=True)


def _run_printable(cmd):
    proc = subprocess.run(shlex.split(cmd), capture_output=True, check=False)  # type: ignore[call-overload]
    if proc.returncode != 0:
        raise AssertionError

    buffer = io.BytesIO()
    torch.save(proc.stdout.decode("utf-8"), buffer)
    input_tensor = torch.ByteTensor(list(buffer.getvalue()))

    output = []
    buffer = io.BytesIO(np.asarray(input_tensor).tobytes())
````

- **L101** EN: Assigns or updates `v`. | CN: 对 `v` 进行赋值或更新。
- **L102** EN: Calls `_print_cont` as part of the current workflow. | CN: 在当前流程中调用 `_print_cont`。
- **L103** EN: Calls `_print_cont` as part of the current workflow. | CN: 在当前流程中调用 `_print_cont`。
- **L104** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L105** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L106** EN: Defines function `_print_cont`. | CN: 定义函数 `_print_cont`。
- **L107** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L108** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L109** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L110** EN: Defines function `_run_printable`. | CN: 定义函数 `_run_printable`。
- **L111** EN: Assigns or updates `proc`. | CN: 对 `proc` 进行赋值或更新。
- **L112** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L113** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L114** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L115** EN: Assigns or updates `buffer`. | CN: 对 `buffer` 进行赋值或更新。
- **L116** EN: Calls `torch.save` as part of the current workflow. | CN: 在当前流程中调用 `torch.save`。
- **L117** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L118** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L119** EN: Assigns or updates `output`. | CN: 对 `output` 进行赋值或更新。
- **L120** EN: Assigns or updates `buffer`. | CN: 对 `buffer` 进行赋值或更新。

### Lines 121-140 / 第 121-140 行

````python
    output.append(torch.load(buffer))
    return output


def _run_trainer(emb_rref_list, rank):
    r"""
    Each trainer runs a forward pass which involves an embedding lookup on the 8 parameter servers,
    and running nn.Linear locally.

    During the backward pass, DDP is responsible for aggregating the gradients for the dense part
    (nn.Linear) and distributed autograd ensures gradients updates are
    propagated to the parameter servers.
    """
    # Setup the model.
    model = HybridModel(emb_rref_list, rank)

    # Retrieve all model parameters as rrefs for DistributedOptimizer.

    # Retrieve parameters from all embedding tables for the current trainer.
    model_parameter_rrefs = []
````

- **L121** EN: Calls `output.append` as part of the current workflow. | CN: 在当前流程中调用 `output.append`。
- **L122** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L123** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L124** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L125** EN: Defines function `_run_trainer`. | CN: 定义函数 `_run_trainer`。
- **L126** EN: Starts the docstring for the function _run_trainer. | CN: 开始定义 function _run_trainer 的文档字符串。
- **L127** EN: Continues the docstring text for the function _run_trainer. | CN: 继续补充 function _run_trainer 的文档字符串内容。
- **L128** EN: Continues the docstring text for the function _run_trainer. | CN: 继续补充 function _run_trainer 的文档字符串内容。
- **L129** EN: Continues the docstring text for the function _run_trainer. | CN: 继续补充 function _run_trainer 的文档字符串内容。
- **L130** EN: Continues the docstring text for the function _run_trainer. | CN: 继续补充 function _run_trainer 的文档字符串内容。
- **L131** EN: Continues the docstring text for the function _run_trainer. | CN: 继续补充 function _run_trainer 的文档字符串内容。
- **L132** EN: Continues the docstring text for the function _run_trainer. | CN: 继续补充 function _run_trainer 的文档字符串内容。
- **L133** EN: Closes the docstring for the function _run_trainer. | CN: 结束 function _run_trainer 的文档字符串。
- **L134** EN: Keeps the inline comment or directive: Setup the model. | CN: 保留这一行注释或指令：Setup the model.
- **L135** EN: Assigns or updates `model`. | CN: 对 `model` 进行赋值或更新。
- **L136** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L137** EN: Keeps the inline comment or directive: Retrieve all model parameters as rrefs for DistributedOptimizer. | CN: 保留这一行注释或指令：Retrieve all model parameters as rrefs for DistributedOptimizer.
- **L138** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L139** EN: Keeps the inline comment or directive: Retrieve parameters from all embedding tables for the current trainer. | CN: 保留这一行注释或指令：Retrieve parameters from all embedding tables for the current trainer.
- **L140** EN: Assigns or updates `model_parameter_rrefs`. | CN: 对 `model_parameter_rrefs` 进行赋值或更新。

### Lines 141-160 / 第 141-160 行

````python
    for ind, emb_rref in enumerate(emb_rref_list):
        ps_name = f"ps{ind}"
        model_parameter_rrefs.extend(
            rpc.rpc_sync(ps_name, _retrieve_embedding_parameters, args=(emb_rref,))
        )

    # model.parameters() only includes local parameters.
    model_parameter_rrefs.extend(RRef(param) for param in model.parameters())

    # Setup distributed optimizer
    opt = DistributedOptimizer(optim.SGD, model_parameter_rrefs, lr=0.05)

    criterion = torch.nn.CrossEntropyLoss()

    def get_next_batch(rank):
        for _ in range(10):
            num_indices = random.randint(20, 50)
            indices = torch.LongTensor(num_indices).random_(0, NUM_EMBEDDINGS)

            # Generate offsets.
````

- **L141** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L142** EN: Assigns or updates `ps_name`. | CN: 对 `ps_name` 进行赋值或更新。
- **L143** EN: Calls `model_parameter_rrefs.extend` as part of the current workflow. | CN: 在当前流程中调用 `model_parameter_rrefs.extend`。
- **L144** EN: Calls `rpc.rpc_sync` as part of the current workflow. | CN: 在当前流程中调用 `rpc.rpc_sync`。
- **L145** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L146** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L147** EN: Keeps the inline comment or directive: model.parameters() only includes local parameters. | CN: 保留这一行注释或指令：model.parameters() only includes local parameters.
- **L148** EN: Calls `model_parameter_rrefs.extend` as part of the current workflow. | CN: 在当前流程中调用 `model_parameter_rrefs.extend`。
- **L149** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L150** EN: Keeps the inline comment or directive: Setup distributed optimizer | CN: 保留这一行注释或指令：Setup distributed optimizer
- **L151** EN: Assigns or updates `opt`. | CN: 对 `opt` 进行赋值或更新。
- **L152** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L153** EN: Assigns or updates `criterion`. | CN: 对 `criterion` 进行赋值或更新。
- **L154** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L155** EN: Defines function `get_next_batch`. | CN: 定义函数 `get_next_batch`。
- **L156** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L157** EN: Assigns or updates `num_indices`. | CN: 对 `num_indices` 进行赋值或更新。
- **L158** EN: Assigns or updates `indices`. | CN: 对 `indices` 进行赋值或更新。
- **L159** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L160** EN: Keeps the inline comment or directive: Generate offsets. | CN: 保留这一行注释或指令：Generate offsets.

### Lines 161-180 / 第 161-180 行

````python
            offsets = []
            start = 0
            batch_size = 0

            while start < num_indices:
                offsets.append(start)
                start += random.randint(1, 10)
                batch_size += 1

            offsets_tensor = torch.LongTensor(offsets)
            target = torch.LongTensor(batch_size).random_(8).cuda(rank)

            yield indices, offsets_tensor, target

    measurements = []
    # Include warm-up cycles during training
    for _ in range(100 + WARMUP_CYCLES):
        start = time.time()
        batch_size = 0

````

- **L161** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L162** EN: Assigns or updates `start`. | CN: 对 `start` 进行赋值或更新。
- **L163** EN: Assigns or updates `batch_size`. | CN: 对 `batch_size` 进行赋值或更新。
- **L164** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L165** EN: Starts a loop controlled by a condition. | CN: 开始一个由条件控制的循环。
- **L166** EN: Calls `offsets.append` as part of the current workflow. | CN: 在当前流程中调用 `offsets.append`。
- **L167** EN: Continues the implementation inside function `get_next_batch`. | CN: 继续说明函数 `get_next_batch` 内部的实现。
- **L168** EN: Continues the implementation inside function `get_next_batch`. | CN: 继续说明函数 `get_next_batch` 内部的实现。
- **L169** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L170** EN: Assigns or updates `offsets_tensor`. | CN: 对 `offsets_tensor` 进行赋值或更新。
- **L171** EN: Assigns or updates `target`. | CN: 对 `target` 进行赋值或更新。
- **L172** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L173** EN: Yields a value from a generator-style callable. | CN: 从生成器风格的可调用对象中产出一个值。
- **L174** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L175** EN: Assigns or updates `measurements`. | CN: 对 `measurements` 进行赋值或更新。
- **L176** EN: Keeps the inline comment or directive: Include warm-up cycles during training | CN: 保留这一行注释或指令：Include warm-up cycles during training
- **L177** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L178** EN: Assigns or updates `start`. | CN: 对 `start` 进行赋值或更新。
- **L179** EN: Assigns or updates `batch_size`. | CN: 对 `batch_size` 进行赋值或更新。
- **L180** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 181-200 / 第 181-200 行

````python
        # create distributed autograd context
        for indices, offsets, target in get_next_batch(rank):
            batch_size += len(target)

            with dist_autograd.context() as context_id:
                output = model(indices, offsets)
                loss = criterion(output, target)

                # Run distributed backward pass
                dist_autograd.backward(context_id, [loss])

                # Run distributed optimizer. Gradients propagated all the way to the parameter servers
                opt.step(context_id)

                # Not necessary to zero grads as each iteration creates a different
                # distributed autograd context which hosts different grads

        measurements.append(time.time() - start)
        # print("Training done for epoch {}".format(epoch))

````

- **L181** EN: Keeps the inline comment or directive: create distributed autograd context | CN: 保留这一行注释或指令：create distributed autograd context
- **L182** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L183** EN: Continues the implementation inside function `_run_trainer`. | CN: 继续说明函数 `_run_trainer` 内部的实现。
- **L184** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L185** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L186** EN: Assigns or updates `output`. | CN: 对 `output` 进行赋值或更新。
- **L187** EN: Assigns or updates `loss`. | CN: 对 `loss` 进行赋值或更新。
- **L188** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L189** EN: Keeps the inline comment or directive: Run distributed backward pass | CN: 保留这一行注释或指令：Run distributed backward pass
- **L190** EN: Calls `dist_autograd.backward` as part of the current workflow. | CN: 在当前流程中调用 `dist_autograd.backward`。
- **L191** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L192** EN: Keeps the inline comment or directive: Run distributed optimizer. Gradients propagated all the way to the parameter ser | CN: 保留这一行注释或指令：Run distributed optimizer. Gradients propagated all the way to the parameter ser
- **L193** EN: Calls `opt.step` as part of the current workflow. | CN: 在当前流程中调用 `opt.step`。
- **L194** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L195** EN: Keeps the inline comment or directive: Not necessary to zero grads as each iteration creates a different | CN: 保留这一行注释或指令：Not necessary to zero grads as each iteration creates a different
- **L196** EN: Keeps the inline comment or directive: distributed autograd context which hosts different grads | CN: 保留这一行注释或指令：distributed autograd context which hosts different grads
- **L197** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L198** EN: Calls `measurements.append` as part of the current workflow. | CN: 在当前流程中调用 `measurements.append`。
- **L199** EN: Keeps the inline comment or directive: print("Training done for epoch {}".format(epoch)) | CN: 保留这一行注释或指令：print("Training done for epoch {}".format(epoch))
- **L200** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 201-220 / 第 201-220 行

````python
    # Throw away warm-up measurements
    measurements = measurements[WARMUP_CYCLES:]
    return rank, measurements, batch_size  # type: ignore[possibly-undefined]


def run_worker(rank, world_size):
    r"""
    Initialize RPC, calls the function, and shuts down RPC.
    """
    # Using different port numbers in TCP init_method for init_rpc and
    # init_process_group to avoid port conflicts.
    rpc_backend_options = TensorPipeRpcBackendOptions()
    rpc_backend_options.init_method = "tcp://localhost:29500"

    # Rank 16. Master
    if rank == (NUM_TRAINERS + NUM_PS):
        rpc.init_rpc(
            "master",
            rank=rank,
            backend=BackendType.TENSORPIPE,  # type: ignore[attr-defined]
````

- **L201** EN: Keeps the inline comment or directive: Throw away warm-up measurements | CN: 保留这一行注释或指令：Throw away warm-up measurements
- **L202** EN: Assigns or updates `measurements`. | CN: 对 `measurements` 进行赋值或更新。
- **L203** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L204** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L205** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L206** EN: Defines function `run_worker`. | CN: 定义函数 `run_worker`。
- **L207** EN: Starts the docstring for the function run_worker. | CN: 开始定义 function run_worker 的文档字符串。
- **L208** EN: Continues the docstring text for the function run_worker. | CN: 继续补充 function run_worker 的文档字符串内容。
- **L209** EN: Closes the docstring for the function run_worker. | CN: 结束 function run_worker 的文档字符串。
- **L210** EN: Keeps the inline comment or directive: Using different port numbers in TCP init_method for init_rpc and | CN: 保留这一行注释或指令：Using different port numbers in TCP init_method for init_rpc and
- **L211** EN: Keeps the inline comment or directive: init_process_group to avoid port conflicts. | CN: 保留这一行注释或指令：init_process_group to avoid port conflicts.
- **L212** EN: Assigns or updates `rpc_backend_options`. | CN: 对 `rpc_backend_options` 进行赋值或更新。
- **L213** EN: Assigns or updates `rpc_backend_options.init_method`. | CN: 对 `rpc_backend_options.init_method` 进行赋值或更新。
- **L214** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L215** EN: Keeps the inline comment or directive: Rank 16. Master | CN: 保留这一行注释或指令：Rank 16. Master
- **L216** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L217** EN: Calls `rpc.init_rpc` as part of the current workflow. | CN: 在当前流程中调用 `rpc.init_rpc`。
- **L218** EN: Continues the implementation inside function `run_worker`. | CN: 继续说明函数 `run_worker` 内部的实现。
- **L219** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L220** EN: Assigns or updates `backend`. | CN: 对 `backend` 进行赋值或更新。

### Lines 221-240 / 第 221-240 行

````python
            world_size=world_size,
        )

        # Build the Embedding tables on the Parameter Servers.
        emb_rref_list = []
        index = 0
        while index < NUM_PS:
            ps_name = f"ps{index}"
            emb_rref = rpc.remote(
                ps_name,
                torch.nn.EmbeddingBag,
                args=(NUM_EMBEDDINGS, EMBEDDING_DIM),
                kwargs={"mode": "sum"},
            )
            emb_rref_list.append(emb_rref)
            index += 1

        # Run training loop on the trainers.
        futs = []
        for trainer_rank in range(NUM_TRAINERS):
````

- **L221** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L222** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L223** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L224** EN: Keeps the inline comment or directive: Build the Embedding tables on the Parameter Servers. | CN: 保留这一行注释或指令：Build the Embedding tables on the Parameter Servers.
- **L225** EN: Assigns or updates `emb_rref_list`. | CN: 对 `emb_rref_list` 进行赋值或更新。
- **L226** EN: Assigns or updates `index`. | CN: 对 `index` 进行赋值或更新。
- **L227** EN: Starts a loop controlled by a condition. | CN: 开始一个由条件控制的循环。
- **L228** EN: Assigns or updates `ps_name`. | CN: 对 `ps_name` 进行赋值或更新。
- **L229** EN: Assigns or updates `emb_rref`. | CN: 对 `emb_rref` 进行赋值或更新。
- **L230** EN: Continues the implementation inside function `run_worker`. | CN: 继续说明函数 `run_worker` 内部的实现。
- **L231** EN: Continues the implementation inside function `run_worker`. | CN: 继续说明函数 `run_worker` 内部的实现。
- **L232** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L233** EN: Assigns or updates `kwargs`. | CN: 对 `kwargs` 进行赋值或更新。
- **L234** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L235** EN: Calls `emb_rref_list.append` as part of the current workflow. | CN: 在当前流程中调用 `emb_rref_list.append`。
- **L236** EN: Continues the implementation inside function `run_worker`. | CN: 继续说明函数 `run_worker` 内部的实现。
- **L237** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L238** EN: Keeps the inline comment or directive: Run training loop on the trainers. | CN: 保留这一行注释或指令：Run training loop on the trainers.
- **L239** EN: Assigns or updates `futs`. | CN: 对 `futs` 进行赋值或更新。
- **L240** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 241-260 / 第 241-260 行

````python
            trainer_name = f"trainer{trainer_rank}"
            fut = rpc.rpc_async(
                trainer_name, _run_trainer, args=(emb_rref_list, trainer_rank)
            )
            futs.append(fut)

        _print_header()

        measurements_all_trainers = []
        batch_size_all_trainers = 0
        # Wait for all training to finish.
        for fut in futs:
            rank, measurements, batch_size = fut.wait()
            _print_benchmark(f"Trainer{rank}", batch_size, measurements)
            batch_size_all_trainers += batch_size
            measurements_all_trainers.append(measurements)

        _print_benchmark("All", batch_size_all_trainers, measurements_all_trainers)

    # Rank 0-7. Trainers
````

- **L241** EN: Assigns or updates `trainer_name`. | CN: 对 `trainer_name` 进行赋值或更新。
- **L242** EN: Assigns or updates `fut`. | CN: 对 `fut` 进行赋值或更新。
- **L243** EN: Assigns or updates `trainer_name, _run_trainer, args`. | CN: 对 `trainer_name, _run_trainer, args` 进行赋值或更新。
- **L244** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L245** EN: Calls `futs.append` as part of the current workflow. | CN: 在当前流程中调用 `futs.append`。
- **L246** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L247** EN: Calls `_print_header` as part of the current workflow. | CN: 在当前流程中调用 `_print_header`。
- **L248** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L249** EN: Assigns or updates `measurements_all_trainers`. | CN: 对 `measurements_all_trainers` 进行赋值或更新。
- **L250** EN: Assigns or updates `batch_size_all_trainers`. | CN: 对 `batch_size_all_trainers` 进行赋值或更新。
- **L251** EN: Keeps the inline comment or directive: Wait for all training to finish. | CN: 保留这一行注释或指令：Wait for all training to finish.
- **L252** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L253** EN: Assigns or updates `rank, measurements, batch_size`. | CN: 对 `rank, measurements, batch_size` 进行赋值或更新。
- **L254** EN: Calls `_print_benchmark` as part of the current workflow. | CN: 在当前流程中调用 `_print_benchmark`。
- **L255** EN: Continues the implementation inside function `run_worker`. | CN: 继续说明函数 `run_worker` 内部的实现。
- **L256** EN: Calls `measurements_all_trainers.append` as part of the current workflow. | CN: 在当前流程中调用 `measurements_all_trainers.append`。
- **L257** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L258** EN: Calls `_print_benchmark` as part of the current workflow. | CN: 在当前流程中调用 `_print_benchmark`。
- **L259** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L260** EN: Keeps the inline comment or directive: Rank 0-7. Trainers | CN: 保留这一行注释或指令：Rank 0-7. Trainers

### Lines 261-280 / 第 261-280 行

````python
    elif rank >= 0 and rank < NUM_PS:
        # Initialize process group for Distributed DataParallel on trainers.
        dist.init_process_group(
            backend=dist.Backend.GLOO,
            rank=rank,
            world_size=NUM_TRAINERS,
            init_method="tcp://localhost:29501",
        )

        # Initialize RPC. Trainer just waits for RPCs from master.
        trainer_name = f"trainer{rank}"
        rpc.init_rpc(
            trainer_name,
            rank=rank,
            world_size=world_size,
            rpc_backend_options=rpc_backend_options,
        )

    # Rank 8-15. Parameter Servers
    elif rank >= NUM_TRAINERS and rank < NUM_TRAINERS + NUM_PS:
````

- **L261** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L262** EN: Keeps the inline comment or directive: Initialize process group for Distributed DataParallel on trainers. | CN: 保留这一行注释或指令：Initialize process group for Distributed DataParallel on trainers.
- **L263** EN: Calls `dist.init_process_group` as part of the current workflow. | CN: 在当前流程中调用 `dist.init_process_group`。
- **L264** EN: Assigns or updates `backend`. | CN: 对 `backend` 进行赋值或更新。
- **L265** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L266** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L267** EN: Assigns or updates `init_method`. | CN: 对 `init_method` 进行赋值或更新。
- **L268** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L269** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L270** EN: Keeps the inline comment or directive: Initialize RPC. Trainer just waits for RPCs from master. | CN: 保留这一行注释或指令：Initialize RPC. Trainer just waits for RPCs from master.
- **L271** EN: Assigns or updates `trainer_name`. | CN: 对 `trainer_name` 进行赋值或更新。
- **L272** EN: Calls `rpc.init_rpc` as part of the current workflow. | CN: 在当前流程中调用 `rpc.init_rpc`。
- **L273** EN: Continues the implementation inside function `run_worker`. | CN: 继续说明函数 `run_worker` 内部的实现。
- **L274** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L275** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L276** EN: Assigns or updates `rpc_backend_options`. | CN: 对 `rpc_backend_options` 进行赋值或更新。
- **L277** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L278** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L279** EN: Keeps the inline comment or directive: Rank 8-15. Parameter Servers | CN: 保留这一行注释或指令：Rank 8-15. Parameter Servers
- **L280** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。

### Lines 281-300 / 第 281-300 行

````python
        ps_name = f"ps{rank - NUM_TRAINERS}"
        rpc.init_rpc(
            ps_name,
            rank=rank,
            world_size=world_size,
            backend=BackendType.TENSORPIPE,  # type: ignore[attr-defined]
            rpc_backend_options=rpc_backend_options,
        )
        # parameter server do nothing

    # block until all rpcs finish
    rpc.shutdown()


if __name__ == "__main__":
    """Initializing the distributed environment."""

    output = _run_printable("nvidia-smi topo -m")
    print("-------------------------------------------")
    print("                  Info                     ")
````

- **L281** EN: Assigns or updates `ps_name`. | CN: 对 `ps_name` 进行赋值或更新。
- **L282** EN: Calls `rpc.init_rpc` as part of the current workflow. | CN: 在当前流程中调用 `rpc.init_rpc`。
- **L283** EN: Continues the implementation inside function `run_worker`. | CN: 继续说明函数 `run_worker` 内部的实现。
- **L284** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L285** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L286** EN: Assigns or updates `backend`. | CN: 对 `backend` 进行赋值或更新。
- **L287** EN: Assigns or updates `rpc_backend_options`. | CN: 对 `rpc_backend_options` 进行赋值或更新。
- **L288** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L289** EN: Keeps the inline comment or directive: parameter server do nothing | CN: 保留这一行注释或指令：parameter server do nothing
- **L290** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L291** EN: Keeps the inline comment or directive: block until all rpcs finish | CN: 保留这一行注释或指令：block until all rpcs finish
- **L292** EN: Calls `rpc.shutdown` as part of the current workflow. | CN: 在当前流程中调用 `rpc.shutdown`。
- **L293** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L294** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L295** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L296** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L297** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L298** EN: Assigns or updates `output`. | CN: 对 `output` 进行赋值或更新。
- **L299** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L300** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。

### Lines 301-320 / 第 301-320 行

````python
    print("-------------------------------------------")
    print()
    print(f"* PyTorch version: {torch.__version__}")
    print(f"* CUDA version: {torch.version.cuda}")
    print()
    print("------------ nvidia-smi topo -m -----------")
    print()
    print(output[0])
    print("-------------------------------------------")
    print("PyTorch Distributed Benchmark (DDP and RPC)")
    print("-------------------------------------------")

    # Cmd arguments to enable automated runs (e.g. Chronos, SSH, etc).
    parser = argparse.ArgumentParser(description="PyTorch DDP and RPC Benchmark")
    parser.add_argument(
        "--master-addr", type=str, default="localhost", help="Address of master node."
    )
    parser.add_argument("--master-port", type=str, default="29500", help="Master port.")

    parser.add_argument(
````

- **L301** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L302** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L303** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L304** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L305** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L306** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L307** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L308** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L309** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L310** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L311** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L312** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L313** EN: Keeps the inline comment or directive: Cmd arguments to enable automated runs (e.g. Chronos, SSH, etc). | CN: 保留这一行注释或指令：Cmd arguments to enable automated runs (e.g. Chronos, SSH, etc).
- **L314** EN: Assigns or updates `parser`. | CN: 对 `parser` 进行赋值或更新。
- **L315** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。
- **L316** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L317** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L318** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。
- **L319** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L320** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。

### Lines 321-340 / 第 321-340 行

````python
        "--number-trainers",
        type=int,
        default=NUM_TRAINERS,
        help="Number of Trainer Nodes.",
    )
    parser.add_argument(
        "--number-ps", type=int, default=NUM_PS, help="Number of Parameter Servers."
    )
    parser.add_argument(
        "--number-embeddings",
        type=int,
        default=NUM_EMBEDDINGS,
        help="Number of test embeddings to be generated.",
    )
    parser.add_argument(
        "--embedding-dim",
        type=int,
        default=EMBEDDING_DIM,
        help="Number of embedding dimensions.",
    )
````

- **L321** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L322** EN: Assigns or updates `type`. | CN: 对 `type` 进行赋值或更新。
- **L323** EN: Assigns or updates `default`. | CN: 对 `default` 进行赋值或更新。
- **L324** EN: Assigns or updates `help`. | CN: 对 `help` 进行赋值或更新。
- **L325** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L326** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。
- **L327** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L328** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L329** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。
- **L330** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L331** EN: Assigns or updates `type`. | CN: 对 `type` 进行赋值或更新。
- **L332** EN: Assigns or updates `default`. | CN: 对 `default` 进行赋值或更新。
- **L333** EN: Assigns or updates `help`. | CN: 对 `help` 进行赋值或更新。
- **L334** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L335** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。
- **L336** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L337** EN: Assigns or updates `type`. | CN: 对 `type` 进行赋值或更新。
- **L338** EN: Assigns or updates `default`. | CN: 对 `default` 进行赋值或更新。
- **L339** EN: Assigns or updates `help`. | CN: 对 `help` 进行赋值或更新。
- **L340** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 341-360 / 第 341-360 行

````python
    parser.add_argument(
        "--warmup-cycles",
        type=int,
        default=WARMUP_CYCLES,
        help="Number of cycles to warm-up each process before running the benchmark.",
    )

    args = parser.parse_args()

    os.environ["MASTER_ADDR"] = args.master_addr
    os.environ["MASTER_PORT"] = args.master_port

    NUM_TRAINERS = args.number_trainers
    NUM_PS = args.number_ps

    NUM_EMBEDDINGS = args.number_embeddings
    EMBEDDING_DIM = args.embedding_dim

    WARMUP_CYCLES = args.warmup_cycles

````

- **L341** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。
- **L342** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L343** EN: Assigns or updates `type`. | CN: 对 `type` 进行赋值或更新。
- **L344** EN: Assigns or updates `default`. | CN: 对 `default` 进行赋值或更新。
- **L345** EN: Assigns or updates `help`. | CN: 对 `help` 进行赋值或更新。
- **L346** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L347** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L348** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L349** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L350** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L351** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L352** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L353** EN: Assigns or updates `NUM_TRAINERS`. | CN: 对 `NUM_TRAINERS` 进行赋值或更新。
- **L354** EN: Assigns or updates `NUM_PS`. | CN: 对 `NUM_PS` 进行赋值或更新。
- **L355** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L356** EN: Assigns or updates `NUM_EMBEDDINGS`. | CN: 对 `NUM_EMBEDDINGS` 进行赋值或更新。
- **L357** EN: Assigns or updates `EMBEDDING_DIM`. | CN: 对 `EMBEDDING_DIM` 进行赋值或更新。
- **L358** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L359** EN: Assigns or updates `WARMUP_CYCLES`. | CN: 对 `WARMUP_CYCLES` 进行赋值或更新。
- **L360** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 361-366 / 第 361-366 行

````python
    # Defaults:
    #  8 trainers (rank 0-7),
    #  8 parameter servers (rank 8-15),
    #  1 master (rank 16).
    world_size = NUM_TRAINERS + NUM_PS + 1  # Trainers + PS + Master
    mp.spawn(run_worker, args=(world_size,), nprocs=world_size, join=True)
````

- **L361** EN: Keeps the inline comment or directive: Defaults: | CN: 保留这一行注释或指令：Defaults:
- **L362** EN: Keeps the inline comment or directive: 8 trainers (rank 0-7), | CN: 保留这一行注释或指令：8 trainers (rank 0-7),
- **L363** EN: Keeps the inline comment or directive: 8 parameter servers (rank 8-15), | CN: 保留这一行注释或指令：8 parameter servers (rank 8-15),
- **L364** EN: Keeps the inline comment or directive: 1 master (rank 16). | CN: 保留这一行注释或指令：1 master (rank 16).
- **L365** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L366** EN: Calls `mp.spawn` as part of the current workflow. | CN: 在当前流程中调用 `mp.spawn`。

## Key Concepts / 关键概念

- **EN**: distributed runtime helpers, APIs, and package wiring  
  **CN**: 分布式运行时辅助逻辑、API 与包级导出
- **EN**: process groups  
  **CN**: 进程组
- **EN**: RPC  
  **CN**: RPC
- **EN**: remote references  
  **CN**: 远程引用
- **EN**: distributed stores  
  **CN**: 分布式存储
- **EN**: optimizer coordination  
  **CN**: 优化器协同
- **EN**: distributed autograd  
  **CN**: 分布式自动求导
- **EN**: Primary classes: HybridModel  
  **CN**: 主要类：HybridModel

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed`, `torch.distributed.autograd`, `torch.distributed.optim`, `torch.distributed.rpc`, `torch.distributed.rpc.backend_registry`
- **PyTorch / PyTorch**: `torch`, `torch.multiprocessing`, `torch.nn`, `torch.nn.parallel`, `torch.optim`
- **Python Stdlib / Python 标准库**: `argparse`, `io`, `os`, `random`, `shlex`, `subprocess`, `time`
- **Third-party / 第三方**: `numpy`

