# int4fp8_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/int4fp8_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `quantize_fp8_scale_tensorwise`, `quantize_int4_scale_columnwise`, and `pack_int4_to_int32` and organizes the supporting helpers, abstractions, and runtime decisions around them. / 该模块为 SGLang 的 SRT 运行时实现了低精度量化数据路径。它提供了 `quantize_fp8_scale_tensorwise`、`quantize_int4_scale_columnwise` 以及 `pack_int4_to_int32` 等符号，并围绕它们组织辅助函数、抽象层以及运行时决策。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports, constants, and runtime setup
```python
"""
Common utilities for quark.
"""

import logging
from typing import Tuple

import torch

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `logging`, `typing.Tuple`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`typing.Tuple` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 13-19: Function `quantize_fp8_scale_tensorwise` and its core logic
```python
def quantize_fp8_scale_tensorwise(w: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    FP8_MAX = 448.0
    scale = w.abs().amax().float() / FP8_MAX
    scaled = (w / scale).clamp(-FP8_MAX, FP8_MAX).to(torch.float8_e4m3fn)
    return scaled, scale
```
**EN:** This block defines `quantize_fp8_scale_tensorwise` and contains the main logic for this step. It mainly invokes `clamp.to`, `w.abs.amax.float`, `clamp`, `w.abs.amax`, and `w.abs`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `FP8_MAX`, `scale`, and `scaled` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `quantize_fp8_scale_tensorwise`，并承载这一阶段的核心逻辑。 它主要调用 `clamp.to`、`w.abs.amax.float`、`clamp`、`w.abs.amax` 以及 `w.abs`，说明该流程会编排底层辅助函数或计算内核。 像 `FP8_MAX`、`scale` 以及 `scaled` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 20-29: Function `quantize_int4_scale_columnwise` and its core logic
```python
def quantize_int4_scale_columnwise(
    w: torch.Tensor,
) -> Tuple[torch.Tensor, torch.Tensor]:
    S4_MAX = 7
    w_flat = w.reshape(-1, w.shape[-1]).float()
    scale = w_flat.abs().amax(axis=-1) / S4_MAX
    scaled = torch.round(w_flat / scale[:, None]).to(torch.int8).clamp(-S4_MAX, S4_MAX)
    return scaled.reshape(w.shape), scale.reshape(w.shape[:-1])
```
**EN:** This block defines `quantize_int4_scale_columnwise` and contains the main logic for this step. It mainly invokes `w.reshape.float`, `torch.round.to.clamp`, `w_flat.abs.amax`, `scaled.reshape`, and `scale.reshape`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `S4_MAX`, `w_flat`, `scale`, and `scaled` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `quantize_int4_scale_columnwise`，并承载这一阶段的核心逻辑。 它主要调用 `w.reshape.float`、`torch.round.to.clamp`、`w_flat.abs.amax`、`scaled.reshape` 以及 `scale.reshape`，说明该流程会编排底层辅助函数或计算内核。 像 `S4_MAX`、`w_flat`、`scale` 以及 `scaled` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 30-73: Function `pack_int4_to_int32` and its core logic
```python
def pack_int4_to_int32(to_pack: torch.Tensor, reorder: bool = True) -> torch.Tensor:
    if to_pack.ndim > 2:
        raise ValueError(
            "Pack: Only supports tensors with dimensions not greater than 2."
        )

    if reorder:
        order_map = [0, 2, 4, 6, 1, 3, 5, 7]
    else:
        order_map = [0, 1, 2, 3, 4, 5, 6, 7]
    pack_num = 8
    if to_pack.ndim == 2:
        packed = torch.zeros(
            to_pack.shape[0],
            to_pack.shape[1] // pack_num,
            dtype=torch.int32,
            device=to_pack.device,
        )
        new_c = to_pack.shape[1] // pack_num
        for c in range(new_c):
            for i in range(pack_num):
                # Use -3 as an example, high_position is 11111111,cause bit_or generate errors, so we can't use int4 directly
                packed_col = to_pack[:, c * pack_num + order_map[i]].to(torch.int32)
                packed_col = packed_col & 0x0F
                packed[:, c] = torch.bitwise_or(
                    packed[:, c], torch.bitwise_left_shift(packed_col, i * 4)
                )
    elif to_pack.ndim == 0:
        packed = to_pack.to(torch.int32)
    else:
        packed = torch.zeros(
            to_pack.shape[0] // pack_num, dtype=torch.int32, device=to_pack.device
        )
        new_c = to_pack.shape[0] // pack_num
        for c in range(new_c):
            for i in range(pack_num):
                # Use -3 as an example, high_position is 11111111,cause bit_or generate errors, so we can't use int4 directly
                packed_col = to_pack[c * pack_num + order_map[i]]
                packed_col = packed_col & 0x0F
                packed[c] = torch.bitwise_or(
                    packed[c], torch.bitwise_left_shift(packed_col, i * 4)
                )

    return packed.view(torch.uint32)
```
**EN:** This block defines `pack_int4_to_int32` and contains the main logic for this step. It mainly invokes `packed.view`, `ValueError`, `torch.zeros`, `range`, and `to_pack.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pack_num`, `order_map`, `packed`, `new_c`, and `packed_col` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pack_int4_to_int32`，并承载这一阶段的核心逻辑。 它主要调用 `packed.view`、`ValueError`、`torch.zeros`、`range` 以及 `to_pack.to`，说明该流程会编排底层辅助函数或计算内核。 像 `pack_num`、`order_map`、`packed`、`new_c` 以及 `packed_col` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `quantize_fp8_scale_tensorwise`, `quantize_int4_scale_columnwise`, and `pack_int4_to_int32`. / **主要符号**：核心入口包括 `quantize_fp8_scale_tensorwise`、`quantize_int4_scale_columnwise` 以及 `pack_int4_to_int32`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。

## Dependencies / 依赖关系
- **Standard library**: `logging` and `typing.Tuple` / **标准库**：`logging` 和 `typing.Tuple`
- **Third-party**: `torch` / **第三方依赖**：`torch`
