# kv_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/kv_cache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime method classes that apply kv cache quantized weights during inference. / 该模块实现了推理期运行方法类，用于应用 KV 缓存 量化权重。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/quantization/kv_cache.py

import logging

import torch

from sglang.srt.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
```
**EN:** This block imports logging, torch, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.fp8_kernel and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 logging, torch, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.fp8_kernel 等依赖，并为当前量化实现准备模块命名空间。

### Lines 15-15: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 18-28: class BaseKVCacheMethod: definition
```python
class BaseKVCacheMethod(QuantizeMethodBase):
    """
    Quant method that adds `k_scale` and `v_scale` attributes to the
    Attention layer to support loading those scaling factors from checkpoints.
    The k/v_scale will be used to:
        - quantize k/v_cache entries before saving them to the cache
        - dequantize k/v_cache entries before fetching them from the cache

    :param quant_config: the appropriate QuantizationConfig
    """
```
**EN:** This block declares `BaseKVCacheMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, apply, process_weights_after_loading.
**CN:** 该代码块声明 `BaseKVCacheMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, apply, process_weights_after_loading 等行为。

### Lines 29-30: BaseKVCacheMethod.__init__()
```python
    def __init__(self, quant_config: QuantizationConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `BaseKVCacheMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `BaseKVCacheMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 32-46: BaseKVCacheMethod.create_weights()
```python
    def create_weights(self, layer: torch.nn.Module):
        """
        Create "weight" (aka k_scale and v_scale) for an attention layer.
        """
        # Initialize the KV cache scales to -1.0, which is an invalid value.
        # If the k/v_scale appears in the checkpoint, it will be
        # overwritten when loading weights.
        layer.k_scale = torch.nn.Parameter(
            torch.tensor(-1.0, dtype=torch.float32), requires_grad=False
        )
        layer.v_scale = torch.nn.Parameter(
            torch.tensor(-1.0, dtype=torch.float32), requires_grad=False
        )
        layer.k_scale._skip_weight_check = True
        layer.v_scale._skip_weight_check = True
```
**EN:** This block defines `BaseKVCacheMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `BaseKVCacheMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 48-49: BaseKVCacheMethod.apply()
```python
    def apply(self, layer: torch.nn.Module) -> torch.Tensor:
        raise RuntimeError(f"{self.__class__.__name__}.apply should not be called.")
```
**EN:** This block defines `BaseKVCacheMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `BaseKVCacheMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 51-85: BaseKVCacheMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer) -> None:
        if layer.k_scale > 0.0 and layer.v_scale > 0.0:
            # We prefer to use separate k_scale and v_scale if present
            k_scale = layer.k_scale.to("cpu").tolist()
            v_scale = layer.v_scale.to("cpu").tolist()
            if is_fp8_fnuz():
                k_scale *= 2
                v_scale *= 2
        elif layer.k_scale < 0.0 and layer.v_scale < 0.0:
            # If no scales were loaded (both scales are invalid negative
            # values), use the default value of 1.0
            k_scale = 1.0
            v_scale = 1.0
        else:
            # If we find a single kv_scale in the checkpoint, we remap
            # kv_scale to k_scale during weight loading, and duplicate
            # k_scale to v_scale here
            assert layer.k_scale > 0.0
            scale_to_duplicate = max(layer.k_scale, layer.v_scale)
            k_scale = scale_to_duplicate.to("cpu").tolist()
            v_scale = scale_to_duplicate.to("cpu").tolist()
            if is_fp8_fnuz():
                k_scale *= 2
                v_scale *= 2

        if not isinstance(k_scale, float) or not isinstance(v_scale, float):
            raise ValueError(
                "Only support per-tensor scaling factor " "for fp8 KV cache"
            )

        # These are used in the final Attention.forward()
        layer.k_scale.copy_(k_scale)
        layer.v_scale.copy_(v_scale)
        layer.k_scale_float = k_scale
        layer.v_scale_float = v_scale
```
**EN:** This block defines `BaseKVCacheMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `BaseKVCacheMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

## Key Concepts / 关键概念
- `BaseKVCacheMethod`: A runtime method class that structures file-level quantization behavior. / `BaseKVCacheMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `logging`, `torch`
- **Internal / 内部**: `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.fp8_kernel`
