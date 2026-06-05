# cpu_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/cpu_worker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for cpu worker in the multimodal generation stack. Key symbols include `CPUWorker`. / 该模块包含多模态生成体系中与 cpu worker 相关的运行时支持代码。 关键符号包括 `CPUWorker`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
import os

import torch

from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import (
    init_logger,
)
from sglang.srt.utils import cpu_has_amx_support, get_cpu_ids_by_node

from .gpu_worker import GPUWorker

_is_cpu_amx_available = cpu_has_amx_support()

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-80: Class `CPUWorker` / 类 `CPUWorker`
```python
class CPUWorker(GPUWorker):
    """
    A worker that executes the model on pure CPU platforms
    """

    def __init__(
        self,
        local_rank: int,
        rank: int,
        master_port: int,
        server_args: ServerArgs,
    ):
        super().__init__(local_rank, rank, master_port, server_args)
        if _is_cpu_amx_available:
# ...

        @torch.library.register_fake("sgl_kernel::shm_allgather")
        def _(data, dim):
            return torch.cat([data] * self.server_args.tp_size, dim=dim)
```
**EN:** This class models `CPUWorker` as a specialization of `GPUWorker`. A worker that executes the model on pure CPU platforms Important methods include `__init__`, `init_cpu_threads_binding`.
**CN:** 该类实现 `CPUWorker`，并继承/扩展 `GPUWorker`。 文档字符串指出：A worker that executes the model on pure CPU platforms 其中较重要的方法包括 `__init__`, `init_cpu_threads_binding`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Video generation flow / 视频生成流程
- Symbol `CPUWorker` anchors the module API / 符号 `CPUWorker` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.utils`, `.gpu_worker`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `os`
