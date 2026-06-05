# server_args.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/server_args.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for server args in the multimodal generation stack. Key symbols include `_normalize_ltx2_two_stage_device_mode`, `is_ltx2_two_stage_pipeline_name`, `Backend`. / 该模块包含多模态生成体系中与 server args 相关的运行时支持代码。 关键符号包括 `_normalize_ltx2_two_stage_device_mode`, `is_ltx2_two_stage_pipeline_name`, `Backend`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Inspired by SGLang: https://github.com/sgl-project/sglang/blob/main/python/sglang/srt/server_args.py
"""The arguments of sglang-diffusion Inference."""

import argparse
import dataclasses
import json
import math
import os
import random
import sys
import tempfile
# ...
LTX2_TWO_STAGE_PIPELINE_NAMES = ("LTX2TwoStagePipeline", "LTX2TwoStageHQPipeline")
# H200-class GPUs (>=130 GiB total) can usually keep both LTX2 DiTs resident.
LTX2_RESIDENT_AUTO_ENABLE_MEM_GB = 130
LORA_MERGE_MODES = ("auto", "merge", "dynamic")
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 83-87: Function `_normalize_ltx2_two_stage_device_mode` / 函数 `_normalize_ltx2_two_stage_device_mode`
```python
def _normalize_ltx2_two_stage_device_mode(mode: str | None) -> str | None:
    if mode is None:
        return None
    mode = mode.lower()
    return mode
```
**EN:** This function drives `_normalize_ltx2_two_stage_device_mode` with inputs such as `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_normalize_ltx2_two_stage_device_mode`，主要处理 `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 90-91: Function `is_ltx2_two_stage_pipeline_name` / 函数 `is_ltx2_two_stage_pipeline_name`
```python
def is_ltx2_two_stage_pipeline_name(pipeline_class_name: str | None) -> bool:
    return pipeline_class_name in LTX2_TWO_STAGE_PIPELINE_NAMES
```
**EN:** This function drives `is_ltx2_two_stage_pipeline_name` with inputs such as `pipeline_class_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_ltx2_two_stage_pipeline_name`，主要处理 `pipeline_class_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 94-119: Class `Backend` / 类 `Backend`
```python
class Backend(str, Enum):
    """
    Enumeration for different model backends.
    - AUTO: Automatically select backend (prefer sglang native, fallback to diffusers)
    - SGLANG: Use sglang's native optimized implementation
    - DIFFUSERS: Use vanilla diffusers pipeline (supports all diffusers models)
    """

    AUTO = "auto"
    SGLANG = "sglang"
    DIFFUSERS = "diffusers"

    @classmethod
    def from_string(cls, value: str) -> "Backend":
# ...
    @classmethod
    def choices(cls) -> list[str]:
        """Get all available choices as strings for argparse."""
        return [backend.value for backend in cls]
```
**EN:** This class models `Backend` as a specialization of `str`, `Enum`. Enumeration for different model backends. Important methods include `from_string`, `choices`.
**CN:** 该类实现 `Backend`，并继承/扩展 `str`, `Enum`。 文档字符串指出：Enumeration for different model backends. 其中较重要的方法包括 `from_string`, `choices`。

### Lines 122-1894: Class `ServerArgs` / 类 `ServerArgs`
```python
@dataclasses.dataclass
class ServerArgs(DisaggArgsMixin):
    # Model and path configuration (for convenience)
    model_path: str

    # explicit model ID override (e.g. "Qwen-Image")
    model_id: str | None = None

    # Model backend (sglang native or diffusers)
    backend: Backend = Backend.AUTO

    # Attention
    attention_backend: str = None
    attention_backend_config: addict.Dict | None = None
# ...
                "Attention backend not specified. Using '%s' by default on ROCm "
                "to match SGLang SRT defaults.",
                default_backend,
            )
```
**EN:** This class models `ServerArgs` as a specialization of `DisaggArgsMixin`. Important methods include `broker_port`, `is_local_mode`, `_adjust_path`, `_adjust_parameters`.
**CN:** 该类实现 `ServerArgs`，并继承/扩展 `DisaggArgsMixin`。 其中较重要的方法包括 `broker_port`, `is_local_mode`, `_adjust_path`, `_adjust_parameters`。

### Lines 1897-1937: Class `PortArgs` / 类 `PortArgs`
```python
@dataclasses.dataclass
class PortArgs:
    # The ipc filename for scheduler (rank 0) to receive inputs from tokenizer (zmq)
    scheduler_input_ipc_name: str

    # The port for nccl initialization (torch.dist)
    nccl_port: int

    # The ipc filename for rpc call between Engine and Scheduler
    rpc_ipc_name: str

    # The ipc filename for Scheduler to send metrics
    metrics_ipc_name: str

# ...
            rpc_ipc_name=f"ipc://{tempfile.NamedTemporaryFile(delete=False).name}",
            metrics_ipc_name=f"ipc://{tempfile.NamedTemporaryFile(delete=False).name}",
            master_port=server_args.master_port,
        )
```
**EN:** This class models `PortArgs`. Important methods include `from_server_args`.
**CN:** 该类实现 `PortArgs`。 其中较重要的方法包括 `from_server_args`。

### Lines 1938-1940: Top-level configuration / 顶层配置
```python


_global_server_args = None
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1943-1951: Function `prepare_server_args` / 函数 `prepare_server_args`
```python
def prepare_server_args(argv: list[str]) -> ServerArgs:
    """
    Prepare the inference arguments from the command line arguments.
    """
    parser = FlexibleArgumentParser()
    ServerArgs.add_cli_args(parser)
    raw_args, unknown_args = parser.parse_known_args(argv)
    server_args = ServerArgs.from_cli_args(raw_args, unknown_args)
    return server_args
```
**EN:** This function drives `prepare_server_args` with inputs such as `argv`. Prepare the inference arguments from the command line arguments.
**CN:** 这个函数负责 `prepare_server_args`，主要处理 `argv` 等输入。 文档字符串说明：Prepare the inference arguments from the command line arguments.

### Lines 1954-1959: Function `set_global_server_args` / 函数 `set_global_server_args`
```python
def set_global_server_args(server_args: ServerArgs):
    """
    Set the global sgl_diffusion config for each process
    """
    global _global_server_args
    _global_server_args = server_args
```
**EN:** This function drives `set_global_server_args` with inputs such as `server_args`. Set the global sgl_diffusion config for each process
**CN:** 这个函数负责 `set_global_server_args`，主要处理 `server_args` 等输入。 文档字符串说明：Set the global sgl_diffusion config for each process

### Lines 1962-1969: Function `get_global_server_args` / 函数 `get_global_server_args`
```python
def get_global_server_args() -> ServerArgs:
    if _global_server_args is None:
        # in ci, usually when we test custom ops/modules directly,
        # we don't set the sgl_diffusion config. In that case, we set a default
        # config.
        # TODO(will): may need to handle this for CI.
        raise ValueError("Global sgl_diffusion args is not set.")
    return _global_server_args
```
**EN:** This function drives `get_global_server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_global_server_args`。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.ltx_2`, `sglang.multimodal_gen.configs.quantization.nunchaku`, `sglang.multimodal_gen.runtime.disaggregation.disagg_args`, `sglang.multimodal_gen.runtime.disaggregation.roles`, `sglang.multimodal_gen.runtime.layers.quantization.configs.nunchaku_config`
- **External / 外部**: `addict`, `yaml`
- **Stdlib / 标准库**: `argparse`, `dataclasses`, `json`, `math`, `os`, `random`, `sys`, `tempfile`
