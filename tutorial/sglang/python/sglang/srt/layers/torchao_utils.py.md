# torchao_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/torchao_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements convolutional layer support for the SGLang SRT runtime. It exposes symbols such as `proj_filter`, `proj_filter_conv3d`, and `apply_torchao_config_to_model` and connects them to backend-specific paths such as `CUDA`. / 该模块为 SGLang 的 SRT 运行时实现了卷积层支持。它提供了 `proj_filter`、`proj_filter_conv3d` 以及 `apply_torchao_config_to_model` 等符号，并把这些符号连接到 `CUDA` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports, constants, and runtime setup
```python
"""
Common utilities for torchao.
"""

import logging
from typing import Callable, Optional

import torch

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `logging`, `typing.Callable`, `typing.Optional`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`typing.Callable`、`typing.Optional` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 13-21: Function `proj_filter` and its core logic
```python
def proj_filter(
    module: torch.nn.Module,
    fqn: str,
):
    """Filter function for quantizing projection layers."""
    return "proj" in fqn


# TODO: implement a more general filter function
```
**EN:** This block defines `proj_filter` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `proj_filter`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 22-31: Function `proj_filter_conv3d` and its core logic
```python
def proj_filter_conv3d(
    module: torch.nn.Module,
    fqn: str,
):
    if isinstance(module, torch.nn.Conv3d):
        logger.warning(f"Quantize: skipping {fqn} because it's a Conv3d")
        return False
    return "proj" in fqn
```
**EN:** This block defines `proj_filter_conv3d` and contains the main logic for this step. It mainly invokes `isinstance` and `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `proj_filter_conv3d`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance` 和 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 32-95: `apply_torchao_config_to_model` apply step for torchao config to model
```python
def apply_torchao_config_to_model(
    model: torch.nn.Module,
    torchao_config: str,
    filter_fn: Optional[Callable] = proj_filter,
):
    """Quantize a modelwith torchao quantization specified by torchao_config

    Args:
       `model`: a model to be quantized based on torchao_config
       `torchao_config` (str): type of quantization and their arguments we want to use to
        quantize the model, e.g. int4wo-128 means int4 weight only quantization with group_size
        128
    """
    if torchao_config == "" or torchao_config is None:
        return model

    # Lazy import to suppress some warnings
    from torchao.quantization import (
        float8_dynamic_activation_float8_weight,
        float8_weight_only,
        int4_weight_only,
        int8_dynamic_activation_int8_weight,
        int8_weight_only,
        quantize_,
    )
    from torchao.quantization.observer import PerRow, PerTensor

    if "int8wo" in torchao_config:
        quantize_(model, int8_weight_only(), filter_fn=proj_filter_conv3d)
    elif "int8dq" in torchao_config:
        quantize_(model, int8_dynamic_activation_int8_weight(), filter_fn=filter_fn)
    elif "int4wo" in torchao_config:
        group_size = int(torchao_config.split("-")[-1])
        assert group_size in [
            32,
            64,
            128,
            256,
        ], f"int4wo groupsize needs to be one of [32, 64, 128, 256] but got {group_size}"
        quantize_(model, int4_weight_only(group_size=group_size), filter_fn=filter_fn)
    elif "fp8wo" in torchao_config:
        # this requires newer hardware
        # [rank0]: AssertionError: fp8e4nv data type is not supported on CUDA arch < 89
        quantize_(model, float8_weight_only(), filter_fn=proj_filter_conv3d)
    elif "fp8dq" in torchao_config:
        granularity = torchao_config.split("-")[-1]
        GRANULARITY_MAP = {
            "per_row": PerRow(),
            "per_tensor": PerTensor(),
        }
        assert (
            granularity in GRANULARITY_MAP
        ), f"Supported granularity are: {GRANULARITY_MAP.keys()}, got {granularity}"
        quantize_(
            model,
            float8_dynamic_activation_float8_weight(
                granularity=GRANULARITY_MAP[granularity]
            ),
            filter_fn=proj_filter_conv3d,
        )
    else:
        raise ValueError(f"Unexpected config: {torchao_config}")

    return model
```
**EN:** This block defines `apply_torchao_config_to_model` and contains the main logic for this step. It mainly invokes `quantize_`, `int8_weight_only`, `int8_dynamic_activation_int8_weight`, `int`, and `int4_weight_only`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `group_size`, `granularity`, and `GRANULARITY_MAP` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_torchao_config_to_model`，并承载这一阶段的核心逻辑。 它主要调用 `quantize_`、`int8_weight_only`、`int8_dynamic_activation_int8_weight`、`int` 以及 `int4_weight_only`，说明该流程会编排底层辅助函数或计算内核。 像 `group_size`、`granularity` 以及 `GRANULARITY_MAP` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `proj_filter`, `proj_filter_conv3d`, and `apply_torchao_config_to_model`. / **主要符号**：核心入口包括 `proj_filter`、`proj_filter_conv3d` 以及 `apply_torchao_config_to_model`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。
- **Convolution support**: Tracks convolution-specific tensor layout, parameter handling, or fused compute. / **卷积支持**：跟踪卷积特有的张量布局、参数处理或融合计算。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `typing.Callable`, and `typing.Optional` / **标准库**：`logging`、`typing.Callable` 以及 `typing.Optional`
- **Third-party**: `torch`, `torchao.quantization.float8_dynamic_activation_float8_weight`, `torchao.quantization.float8_weight_only`, `torchao.quantization.int4_weight_only`, `torchao.quantization.int8_dynamic_activation_int8_weight`, `torchao.quantization.int8_weight_only`, `torchao.quantization.quantize_`, `torchao.quantization.observer.PerRow`, and `torchao.quantization.observer.PerTensor` / **第三方依赖**：`torch`、`torchao.quantization.float8_dynamic_activation_float8_weight`、`torchao.quantization.float8_weight_only`、`torchao.quantization.int4_weight_only`、`torchao.quantization.int8_dynamic_activation_int8_weight`、`torchao.quantization.int8_weight_only`、`torchao.quantization.quantize_`、`torchao.quantization.observer.PerRow` 以及 `torchao.quantization.observer.PerTensor`
