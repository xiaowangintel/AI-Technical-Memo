# mxfp8_npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/mxfp8_npu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `MXFP8Config`, and `NPUMXFP8DiffusionLinearMethod`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Online MXFP8 quantization for Diffusion models on Ascend NPU. Provides ``MXFP8Config`` (registered as ``"mxfp8"``) and ``NPUMXFP8DiffusionLinearMethod`` which quantise FP16/BF16 weights to MXFP8 at load time and use ``npu_dynamic_mx_quant`` + ``npu_quant_matmul`` for inference, mirroring the LLM-side ``NPUMXFP8LinearMethod``. / 该文件属于运行时算子层。它围绕 `MXFP8Config` 和 `NPUMXFP8DiffusionLinearMethod` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module setup and imports / 模块初始化与导入
```python
"""Online MXFP8 quantization for Diffusion models on Ascend NPU.

Provides ``MXFP8Config`` (registered as ``"mxfp8"``) and
``NPUMXFP8DiffusionLinearMethod`` which quantise FP16/BF16 weights to MXFP8
at load time and use ``npu_dynamic_mx_quant`` + ``npu_quant_matmul`` for
inference, mirroring the LLM-side ``NPUMXFP8LinearMethod``.
"""

from __future__ import annotations

from typing import Any, Dict, List, Optional

import torch
from torch.nn.parameter import Parameter

from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes the module context and imports `__future__`, `typing`, `torch`, `torch.nn.parameter`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`typing`、`torch`、`torch.nn.parameter` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 18-33: supporting statements / 辅助语句
```python
_is_npu = current_platform.is_npu()

if _is_npu:
    import torch_npu

from sglang.multimodal_gen.runtime.layers.linear import LinearBase, LinearMethodBase
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.multimodal_gen.runtime.models.parameter import ModelWeightParameter
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)

MXFP8_BLOCK_SIZE = 32
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_npu`, `logger`, and `MXFP8_BLOCK_SIZE`. The code collaborates with `current_platform.is_npu`, and `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_npu`、`logger` 和 `MXFP8_BLOCK_SIZE` 等名称。 代码会与 `current_platform.is_npu` 和 `init_logger` 协同工作。

### Lines 36-38: `MXFP8Config` class overview / `MXFP8Config` 类概览
```python
class MXFP8Config(QuantizationConfig):
    """Config for online MXFP8 quantization on Ascend NPU (Diffusion)."""
```
**EN:** This block defines class `MXFP8Config`. Config for online MXFP8 quantization on Ascend NPU (Diffusion). It inherits from `QuantizationConfig`.
**CN:** 该代码块定义了类 `MXFP8Config`。 它用于封装 mxfp8 config 相关行为。 它继承自 `QuantizationConfig`。

### Lines 39-40: `__init__` implementation / `__init__` 实现
```python
    def __init__(self) -> None:
        super().__init__()
```
**EN:** This block defines method `__init__` on `MXFP8Config`. It initializes the instance state. Key calls include `super.__init__`, and `super`.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。

### Lines 42-44: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "mxfp8"
```
**EN:** This block defines method `get_name` on `MXFP8Config`. It retrieves name.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `get_name`。 它用于获取name。

### Lines 46-48: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.float16]
```
**EN:** This block defines method `get_supported_act_dtypes` on `MXFP8Config`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 50-52: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 0  # NPU, not CUDA
```
**EN:** This block defines method `get_min_capability` on `MXFP8Config`. It retrieves min capability.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 54-56: `get_config_filenames` implementation / `get_config_filenames` 实现
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines method `get_config_filenames` on `MXFP8Config`. It retrieves config filenames.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `get_config_filenames`。 它用于获取config filenames。

### Lines 58-60: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> "MXFP8Config":
        return cls()
```
**EN:** This block defines method `from_config` on `MXFP8Config`. It constructs from config. Key calls include `cls`. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `cls`。 本段逻辑主要由 `config` 等参数驱动。

### Lines 62-67: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        if isinstance(layer, LinearBase):
            return NPUMXFP8DiffusionLinearMethod(self)
        return None
```
**EN:** This block defines method `get_quant_method` on `MXFP8Config`. It retrieves quant method. Key calls include `isinstance`, and `NPUMXFP8DiffusionLinearMethod`. The implementation branches on conditions. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `isinstance` 和 `NPUMXFP8DiffusionLinearMethod`。 实现中包含条件分支。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 69-70: `get_scaled_act_names` implementation / `get_scaled_act_names` 实现
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines method `get_scaled_act_names` on `MXFP8Config`. It retrieves scaled act names.
**CN:** 该代码块定义了 `MXFP8Config` 的方法 `get_scaled_act_names`。 它用于获取scaled act names。

### Lines 73-79: `NPUMXFP8DiffusionLinearMethod` class overview / `NPUMXFP8DiffusionLinearMethod` 类概览
```python
class NPUMXFP8DiffusionLinearMethod(LinearMethodBase):
    """Ascend NPU MXFP8 linear method for Diffusion models.

    Online mode: loads FP16/BF16 weights → quantises to MXFP8 at load time.
    Inference: dynamic MXFP8 activation quant + MXFP8 matmul (block_size=32).
    """
```
**EN:** This block defines class `NPUMXFP8DiffusionLinearMethod`. Ascend NPU MXFP8 linear method for Diffusion models. Online mode: loads FP16/BF16 weights → quantises to MXFP8 at load time. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `NPUMXFP8DiffusionLinearMethod`。 它用于封装 npumxfp8 diffusion linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 80-81: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, quant_config: MXFP8Config):
        self.quant_config = quant_config
```
**EN:** This block defines method `__init__` on `NPUMXFP8DiffusionLinearMethod`. It initializes the instance state. Parameters such as `quant_config` drive the behavior in this section.
**CN:** 该代码块定义了 `NPUMXFP8DiffusionLinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `quant_config` 等参数驱动。

### Lines 83-112: `create_weights` implementation / `create_weights` 实现
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
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype

        # Load weights in original dtype; quantise later in process_weights_after_loading
        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=params_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)
```
**EN:** This block defines method `create_weights` on `NPUMXFP8DiffusionLinearMethod`. It creates weights. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, and `torch.empty`. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `NPUMXFP8DiffusionLinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`extra_weight_attrs.get`、`ModelWeightParameter`、`layer.register_parameter` 和 `torch.empty`。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 114-137: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:

        weight_fp = layer.weight.data
        if weight_fp.dtype not in (torch.float16, torch.bfloat16):
            weight_fp = weight_fp.to(torch.bfloat16)

        # Move weight to NPU if needed. We intentionally use a conditional
        # move rather than an assert because `dit_cpu_offload` defaults to
        # True in ServerArgs, which causes fsdp_load to move every parameter
        # back to CPU after loading (even when the target device is NPU).
        # npu_dynamic_mx_quant requires an NPU tensor, so we must transfer
        # here. The quantized fp8 weights produced below will remain on NPU
        # for inference; if the model still needs to be offloaded after
        # quantization (e.g. very large model on a small NPU), a higher-level
        # offload pass can move them back afterwards.
        if not weight_fp.is_npu:
            weight_fp = weight_fp.to(f"npu:{torch.npu.current_device()}")

        # Online MXFP8 quantisation of weights (block_size=32)
        qw, w_scale = torch_npu.npu_dynamic_mx_quant(
            weight_fp, dst_type=torch_npu.float8_e4m3fn
        )
        layer.weight = Parameter(qw, requires_grad=False)
        layer.weight_scale_inv = Parameter(w_scale, requires_grad=False)
```
**EN:** This block defines method `process_weights_after_loading` on `NPUMXFP8DiffusionLinearMethod`. It processes weights after loading. Key calls include `torch_npu.npu_dynamic_mx_quant`, `Parameter`, `weight_fp.to`, and `torch.npu.current_device`. The implementation branches on conditions. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `NPUMXFP8DiffusionLinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `torch_npu.npu_dynamic_mx_quant`、`Parameter`、`weight_fp.to` 和 `torch.npu.current_device`。 实现中包含条件分支。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 139-176: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        original_dtype = x.dtype
        if original_dtype not in (torch.float16, torch.bfloat16):
            x = x.to(torch.bfloat16)
            original_dtype = torch.bfloat16

        # Flatten to 2D [tokens, hidden] so npu_dynamic_mx_quant returns 3D scale
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
            layer.weight_scale_inv.transpose(0, 1),
            scale_dtype=torch_npu.float8_e8m0fnu,
            pertoken_scale=input_scale,
            pertoken_scale_dtype=torch_npu.float8_e8m0fnu,
            bias=bias.to(torch.float32) if bias is not None else None,
            output_dtype=original_dtype,
            group_sizes=[1, 1, MXFP8_BLOCK_SIZE],
        )

        # Restore original shape (replace last dim with output features)
        output_shape = list(input_shape[:-1]) + [output.shape[-1]]
        output = output.reshape(output_shape)

        return output
```
**EN:** This block defines method `apply` on `NPUMXFP8DiffusionLinearMethod`. It applies function. Key calls include `x.reshape`, `torch_npu.npu_dynamic_mx_quant`, `torch_npu.npu_quant_matmul`, `output.reshape`, and `x.to`. The implementation branches on conditions. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `NPUMXFP8DiffusionLinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `x.reshape`、`torch_npu.npu_dynamic_mx_quant`、`torch_npu.npu_quant_matmul`、`output.reshape` 和 `x.to`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `MXFP8Config`: Config for online MXFP8 quantization on Ascend NPU (Diffusion). / 核心类，用于封装 mxfp8 config 相关行为。
- `NPUMXFP8DiffusionLinearMethod`: Ascend NPU MXFP8 linear method for Diffusion models. / 核心类，用于封装 npumxfp8 diffusion linear method 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.nn.parameter`, `torch_npu`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 176
