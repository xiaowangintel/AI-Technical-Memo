# gpu_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/gpu_worker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for gpu worker in the multimodal generation stack. Key symbols include `_ExpandedOutputParts`, `GPUWorker`, `_oom_exceptions`. / 该模块包含多模态生成体系中与 gpu worker 相关的运行时支持代码。 关键符号包括 `_ExpandedOutputParts`, `GPUWorker`, `_oom_exceptions`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-79: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
import gc
import logging
import multiprocessing as mp
import os
import time
from contextlib import ExitStack
from dataclasses import dataclass, field
from typing import Any, Callable, List, Union

import numpy as np
import torch
# ...
    "text_encoder",
    "text_encoder_2",
    "transformer",
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 82-91: Class `_ExpandedOutputParts` / 类 `_ExpandedOutputParts`
```python
@dataclass
class _ExpandedOutputParts:
    tensor_outputs: list[torch.Tensor] = field(default_factory=list)
    list_outputs: list[Any] = field(default_factory=list)
    tensor_audio: list[torch.Tensor] = field(default_factory=list)
    trajectory_latents: list[torch.Tensor] = field(default_factory=list)
    noise_preds: list[torch.Tensor] = field(default_factory=list)
    output_file_paths: list[str] = field(default_factory=list)
    metrics_list: list[Any] = field(default_factory=list)
    trajectory_decoded_parts: list[list[torch.Tensor]] | None = None
```
**EN:** This class models `_ExpandedOutputParts`.
**CN:** 该类实现 `_ExpandedOutputParts`。

### Lines 94-867: Class `GPUWorker` / 类 `GPUWorker`
```python
class GPUWorker:
    """
    A worker that executes the model on a single GPU.
    """

    def __init__(
        self,
        local_rank: int,
        rank: int,
        master_port: int,
        server_args: ServerArgs,
    ):
        self.local_rank = local_rank
        self.rank = rank
# ...
            checksums[name] = compute_weights_checksum(
                iter_materialized_weights(module)
            )
        return checksums
```
**EN:** This class models `GPUWorker`. A worker that executes the model on a single GPU. Important methods include `__init__`, `init_device_and_model`, `do_mem_analysis`, `_format_offload_disable_suggestions`.
**CN:** 该类实现 `GPUWorker`。 文档字符串指出：A worker that executes the model on a single GPU. 其中较重要的方法包括 `__init__`, `init_device_and_model`, `do_mem_analysis`, `_format_offload_disable_suggestions`。

### Lines 868-887: Module documentation and setup / 模块文档与初始化
```python


OOM_MSG = """
OOM detected. Possible solutions:
  - If the OOM occurs during loading:
    1. Check available memory on every selected GPU, not only total capacity.
       In multi-GPU runs, the least-free selected GPU is the bottleneck.
    2. For single-GPU deployment, use `--performance-mode memory`, component CPU offload,
       or `--dit-layerwise-offload` for supported Wan/MOVA DiTs.
    3. For multi-GPU deployment, keep the default `--performance-mode auto` or set
       `--use-fsdp-inference true` to shard DiT weights with FSDP. FSDP is not a
       single-GPU substitute for CPU offload.
  - If the OOM occurs during runtime:
    1. Reduce resolution, `--num-frames`, or batch size.
# ...
    4. Use FSDP, with CFG parallelism when supported, for validated multi-GPU workloads.
    5. Use a lower-memory attention backend or quantization when available.
  Or, open an issue on GitHub https://github.com/sgl-project/sglang/issues/new/choose
"""
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 890-895: Function `_oom_exceptions` / 函数 `_oom_exceptions`
```python
def _oom_exceptions():
    # torch.OutOfMemoryError exists only in some PyTorch builds
    types = [torch.cuda.OutOfMemoryError]
    if hasattr(torch, "OutOfMemoryError"):
        types.append(torch.OutOfMemoryError)
    return tuple(types)
```
**EN:** This function drives `_oom_exceptions`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_oom_exceptions`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 898-964: Function `run_scheduler_process` / 函数 `run_scheduler_process`
```python
def run_scheduler_process(
    local_rank: int,
    rank: int,
    master_port: int,
    server_args: ServerArgs,
    pipe_writer: mp.connection.Connection,
    # For all workers: pipe to receive tasks from rank 0
    task_pipe_r: mp.connection.Connection,
    # For slave workers: pipe to send results back to rank 0
    result_pipe_w: mp.connection.Connection | None,
    # For rank 0 worker only: pipes to send tasks to slaves
    task_pipes_to_slaves: list[mp.connection.Connection] | None = None,
    # For rank 0 worker only: pipes to receive results from slaves
    result_pipes_from_slaves: list[mp.connection.Connection] | None = None,
# ...
            torch.cuda.empty_cache()
        if torch.distributed.is_available() and torch.distributed.is_initialized():
            torch.distributed.destroy_process_group()
        logger.info(f"Worker {rank}: Shutdown complete.")
```
**EN:** This function drives `run_scheduler_process` with inputs such as `local_rank`, `rank`, `master_port`, `server_args`. The entry point for the worker process.
**CN:** 这个函数负责 `run_scheduler_process`，主要处理 `local_rank`, `rank`, `master_port`, `server_args` 等输入。 文档字符串说明：The entry point for the worker process.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.loader.weight_utils`, `sglang.multimodal_gen.runtime.loader.weights_updater`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.pipelines_core`
- **External / 外部**: `multiprocessing`, `numpy`, `torch`, `setproctitle`
- **Stdlib / 标准库**: `gc`, `logging`, `os`, `time`, `contextlib`, `dataclasses`, `typing`
