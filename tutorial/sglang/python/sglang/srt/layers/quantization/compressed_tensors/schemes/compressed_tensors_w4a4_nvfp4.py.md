# compressed_tensors_w4a4_nvfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/schemes/compressed_tensors_w4a4_nvfp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for compressed tensors w4a4 nvfp4 quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 压缩 张量 w4a4 NVFP4 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: module imports and setup
```python
# Adapted from https://github.com/vllm-project/vllm/tree/main/vllm/model_executor/layers/quantization/compressed_tensors
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import logging
from collections.abc import Callable
from typing import Optional

import torch
from torch.nn.parameter import Parameter

from sglang.srt.layers.parameter import (
    GroupQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    CompressedTensorsLinearScheme,
)
from sglang.srt.layers.quantization.fp4_utils import get_fp4_gemm_runner_backend
from sglang.srt.layers.quantization.modelopt_quant import (
    enable_flashinfer_fp4_gemm,
    fp4_gemm,
    fp4_quantize,
)
from sglang.srt.layers.quantization.utils import swizzle_blockscale
```
**EN:** This block imports collections.abc, flashinfer, logging, torch, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.fp4_utils, sglang.srt.layers.quantization.modelopt_quant and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 collections.abc, flashinfer, logging, torch, sglang.srt.layers.parameter, sglang.srt.layers.quantization.compressed_tensors.schemes, sglang.srt.layers.quantization.fp4_utils, sglang.srt.layers.quantization.modelopt_quant 等依赖，并为当前量化实现准备模块命名空间。

### Lines 27-27: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 29-29: initialize __all__
```python
__all__ = ["CompressedTensorsW4A4Fp4"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 32-32: class CompressedTensorsW4A4Fp4: definition
```python
class CompressedTensorsW4A4Fp4(CompressedTensorsLinearScheme):
```
**EN:** This block declares `CompressedTensorsW4A4Fp4`, a scheme class for the quantization stack. It organizes behaviors such as __init__, get_min_capability, create_weights, process_weights_after_loading.
**CN:** 该代码块声明 `CompressedTensorsW4A4Fp4`，它是量化栈中的执行方案类，组织了 __init__, get_min_capability, create_weights, process_weights_after_loading 等行为。

### Lines 33-34: CompressedTensorsW4A4Fp4.__init__()
```python
    def __init__(self):
        self.group_size = 16
```
**EN:** This block defines `CompressedTensorsW4A4Fp4.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Fp4.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 36-38: CompressedTensorsW4A4Fp4.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 100
```
**EN:** This block defines `CompressedTensorsW4A4Fp4.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Fp4.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 40-92: CompressedTensorsW4A4Fp4.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        output_partition_sizes: list[int],
        input_size_per_partition: int,
        params_dtype: torch.dtype,
        weight_loader: Callable,
        **kwargs,
    ):
        output_size_per_partition = sum(output_partition_sizes)
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        # Weight
        weight = ModelWeightParameter(
            data=torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition // 2,
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_packed", weight)

        # Global Weight Scale
        weight_global_scale = PerTensorScaleParameter(
            data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_global_scale", weight_global_scale)

        # Per Group Weight Scale
        weight_scale = GroupQuantScaleParameter(
            data=torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition // self.group_size,
                dtype=torch.float8_e4m3fn,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )

        layer.register_parameter("weight_scale", weight_scale)

        input_global_scale = PerTensorScaleParameter(
            data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
            weight_loader=weight_loader,
        )
        layer.register_parameter("input_global_scale", input_global_scale)
```
**EN:** This block defines `CompressedTensorsW4A4Fp4.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Fp4.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 94-132: CompressedTensorsW4A4Fp4.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer) -> None:
        global_input_scale = layer.input_global_scale.max().to(torch.float32)
        layer.input_global_scale = Parameter(global_input_scale, requires_grad=False)

        layer.weight_global_scale = Parameter(
            layer.weight_global_scale.max().to(torch.float32), requires_grad=False
        )

        if get_fp4_gemm_runner_backend().is_flashinfer_trtllm():
            # FlashInfer TRTLLM FP4 GEMM requires a different weight layout.
            # FlashInfer provides nvfp4_quantize to quantize + shuffle the
            # layout but we use our own quantization so we have to call
            # shuffles ourselves.
            from flashinfer import shuffle_matrix_a, shuffle_matrix_sf_a

            weight = layer.weight_packed.data
            weight_scale = layer.weight_scale.data

            epilogue_tile_m = 128
            weight = shuffle_matrix_a(weight.view(torch.uint8), epilogue_tile_m)
            weight_scale = (
                shuffle_matrix_sf_a(weight_scale.view(torch.uint8), epilogue_tile_m)
                .reshape(weight_scale.shape)
                .view(torch.float8_e4m3fn)
            )

            layer.weight_scale = Parameter(weight_scale, requires_grad=False)
            layer.weight_packed = Parameter(weight, requires_grad=False)
        else:
            swizzled_weight_scale = swizzle_blockscale(layer.weight_scale)
            layer.weight_scale = Parameter(swizzled_weight_scale, requires_grad=False)
            layer.weight_packed = Parameter(
                layer.weight_packed.data, requires_grad=False
            )

        layer.alpha = Parameter(
            1 / (layer.input_global_scale * layer.weight_global_scale),
            requires_grad=False,
        )
```
**EN:** This block defines `CompressedTensorsW4A4Fp4.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Fp4.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 134-172: CompressedTensorsW4A4Fp4.apply_weights()
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        output_dtype = x.dtype
        w_n, _ = layer.weight_packed.shape
        output_shape = [x.shape[0], w_n]

        # quantize BF16 or FP16 to (FP4 and interleaved block scale)
        x_fp4, x_blockscale = fp4_quantize(x, layer.input_global_scale)

        assert x_fp4.dtype == torch.uint8
        assert layer.weight_packed.dtype == torch.uint8
        assert layer.weight_scale.dtype == torch.float8_e4m3fn
        assert layer.alpha.dtype == torch.float32

        w = layer.weight_packed
        w_blockscale = layer.weight_scale
        if (
            enable_flashinfer_fp4_gemm
            and not get_fp4_gemm_runner_backend().is_cutlass()
        ):
            w = layer.weight_packed.T
            w_blockscale = layer.weight_scale.T

        out = fp4_gemm(
            x_fp4,
            w,
            x_blockscale,
            w_blockscale,
            layer.alpha,
            output_dtype,
            w_n,
        )
        if bias is not None:
            out = out + bias
        return out.view(*output_shape)
```
**EN:** This block defines `CompressedTensorsW4A4Fp4.apply_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsW4A4Fp4.apply_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `CompressedTensorsW4A4Fp4`: A scheme class that structures file-level quantization behavior. / `CompressedTensorsW4A4Fp4` 是一个执行方案类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `collections.abc`, `flashinfer`, `logging`, `torch`, `torch.nn.parameter`, `typing`
- **Internal / 内部**: `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.fp4_utils`, `sglang.srt.layers.quantization.modelopt_quant`, `sglang.srt.layers.quantization.utils`
