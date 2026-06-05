# modelslim_mxfp8_scheme.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/modelslim_mxfp8_scheme.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `ModelSlimMXFP8Scheme`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: ModelSlim MXFP8 scheme for pre-quantized weight inference on Ascend NPU. Loads weights pre-quantized by msmodelslim (float8_e4m3fn weights, uint8 scales) and runs MXFP8 matmul at inference. / 该文件属于运行时算子层。它围绕 `ModelSlimMXFP8Scheme` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module setup and imports / 模块初始化与导入
```python
"""ModelSlim MXFP8 scheme for pre-quantized weight inference on Ascend NPU.

Loads weights pre-quantized by msmodelslim (float8_e4m3fn weights,
uint8 scales) and runs MXFP8 matmul at inference.
"""

from typing import List, Optional

import torch

from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes the module context and imports `typing`, `torch`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `typing`、`torch` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 13-24: supporting statements / 辅助语句
```python
_is_npu = current_platform.is_npu()

if _is_npu:
    import torch_npu

from sglang.multimodal_gen.runtime.models.parameter import (
    GroupQuantScaleParameter,
    ModelWeightParameter,
)
from sglang.srt.layers.quantization.modelslim.schemes import ModelSlimLinearScheme

MXFP8_BLOCK_SIZE = 32
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_npu`, and `MXFP8_BLOCK_SIZE`. The code collaborates with `current_platform.is_npu`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_npu` 和 `MXFP8_BLOCK_SIZE` 等名称。 代码会与 `current_platform.is_npu` 协同工作。

### Lines 27-28: `ModelSlimMXFP8Scheme` class overview / `ModelSlimMXFP8Scheme` 类概览
```python
class ModelSlimMXFP8Scheme(ModelSlimLinearScheme):
```
**EN:** This block defines class `ModelSlimMXFP8Scheme`. It encapsulates model slim mxfp8 scheme behavior. It inherits from `ModelSlimLinearScheme`.
**CN:** 该代码块定义了类 `ModelSlimMXFP8Scheme`。 它用于封装 model slim mxfp8 scheme 相关行为。 它继承自 `ModelSlimLinearScheme`。

### Lines 29-69: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: List[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        weight_loader = extra_weight_attrs.get("weight_loader")
        output_size_per_partition = sum(output_partition_sizes)

        # msmodelslim exports weight as float8_e4m3fn, shape [out, in]
        weight = ModelWeightParameter(
            data=torch.empty(
                (output_size_per_partition, input_size_per_partition),
                dtype=torch.float8_e4m3fn,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        # msmodelslim exports weight_scale as uint8, shape [out, in/32].
        # NOTE: This parameter is intentionally named "weight_scale" (not
        # "weight_scale_inv" as used in mxfp8_npu.py) because the weight loader
        # matches parameter names to checkpoint keys, and msmodelslim checkpoints
        # store this tensor under the key "<layer>.weight_scale".
        scale_dim = input_size_per_partition // MXFP8_BLOCK_SIZE
        weight_scale = GroupQuantScaleParameter(
            data=torch.empty(
                (output_size_per_partition, scale_dim),
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_scale", weight_scale)
```
**EN:** This block defines method `create_weights` on `ModelSlimMXFP8Scheme`. It creates weights. Key calls include `extra_weight_attrs.get`, `sum`, `ModelWeightParameter`, `layer.register_parameter`, and `GroupQuantScaleParameter`. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimMXFP8Scheme` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `extra_weight_attrs.get`、`sum`、`ModelWeightParameter`、`layer.register_parameter` 和 `GroupQuantScaleParameter`。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 71-79: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        # weight is already float8_e4m3fn, no cast needed
        weight = layer.weight.data
        layer.weight = torch.nn.Parameter(weight, requires_grad=False)

        # Reshape weight_scale: [out, in/32] -> [out, in/32//2, 2]
        weight_scale = layer.weight_scale.data
        weight_scale = weight_scale.reshape(weight_scale.shape[0], -1, 2)
        layer.weight_scale = torch.nn.Parameter(weight_scale, requires_grad=False)
```
**EN:** This block defines method `process_weights_after_loading` on `ModelSlimMXFP8Scheme`. It processes weights after loading. Key calls include `torch.nn.Parameter`, and `weight_scale.reshape`. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimMXFP8Scheme` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `torch.nn.Parameter` 和 `weight_scale.reshape`。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 81-124: `apply_weights` implementation / `apply_weights` 实现
```python
    def apply_weights(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        original_dtype = x.dtype
        if original_dtype not in (torch.float16, torch.bfloat16):
            # npu_dynamic_mx_quant only accepts fp16/bf16 activations
            x = x.to(torch.bfloat16)
            original_dtype = torch.bfloat16

        # npu_dynamic_mx_quant requires a 2D input [tokens, hidden_size].
        # Diffusion transformer inputs are typically 3D [batch, seq, hidden] or
        # higher. Flattening to 2D merges all leading dimensions into a single
        # token axis so the NPU kernel can compute per-token MXFP8 scales, then
        # we restore the original shape from the output.
        input_shape = x.shape
        x_2d = x.reshape(-1, x.shape[-1])

        # Dynamic MXFP8 activation quantisation
        qx, input_scale = torch_npu.npu_dynamic_mx_quant(
            x_2d, dst_type=torch_npu.float8_e4m3fn
        )

        # MXFP8 matmul
        output = torch_npu.npu_quant_matmul(
            qx,
            layer.weight.transpose(0, 1),
            layer.weight_scale.transpose(0, 1),
            scale_dtype=torch_npu.float8_e8m0fnu,
            pertoken_scale=input_scale,
            pertoken_scale_dtype=torch_npu.float8_e8m0fnu,
            bias=bias.to(torch.float32) if bias is not None else None,
            output_dtype=original_dtype,
            group_sizes=[1, 1, MXFP8_BLOCK_SIZE],
        )

        # Restore original shape
        output_shape = list(input_shape[:-1]) + [output.shape[-1]]
        output = output.reshape(output_shape)

        return output
```
**EN:** This block defines method `apply_weights` on `ModelSlimMXFP8Scheme`. It applies weights. Key calls include `x.reshape`, `torch_npu.npu_dynamic_mx_quant`, `torch_npu.npu_quant_matmul`, `output.reshape`, and `x.to`. The implementation branches on conditions. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimMXFP8Scheme` 的方法 `apply_weights`。 它用于应用weights。 关键调用包括 `x.reshape`、`torch_npu.npu_dynamic_mx_quant`、`torch_npu.npu_quant_matmul`、`output.reshape` 和 `x.to`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `ModelSlimMXFP8Scheme`: Primary class that encapsulates model slim mxfp8 scheme behavior. / 核心类，用于封装 model slim mxfp8 scheme 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方依赖**: `torch`, `torch_npu`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.srt.layers.quantization.modelslim.schemes`

- **Total lines / 总行数**: 124
