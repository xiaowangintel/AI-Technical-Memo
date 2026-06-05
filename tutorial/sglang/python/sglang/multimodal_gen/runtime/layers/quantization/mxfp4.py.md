# mxfp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/mxfp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `Mxfp4Config`, and `Mxfp4LinearMethod`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `Mxfp4Config` 和 `Mxfp4LinearMethod` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module setup and imports / 模块初始化与导入
```python
import logging
from typing import Dict, List, Optional

import torch
from torch.nn.parameter import Parameter

from sglang.multimodal_gen.runtime.layers.linear import (
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
)
from sglang.multimodal_gen.runtime.models.parameter import (
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.srt.layers.quantization.utils import is_layer_skipped
from sglang.srt.utils import is_hip, mxfp_supported
```
**EN:** This block establishes the module context and imports `logging`, `typing`, `torch`, `torch.nn.parameter`, `sglang.multimodal_gen.runtime.layers.linear`, and `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `logging`、`typing`、`torch`、`torch.nn.parameter`、`sglang.multimodal_gen.runtime.layers.linear` 和 `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`。这些依赖为后续实现提供所需符号。

### Lines 21-40: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
_is_hip = is_hip()

if _is_hip:
    try:
        import aiter
        from aiter.ops.gemm_op_a4w4 import gemm_a4w4
        from aiter.ops.shuffle import shuffle_weight
        from aiter.utility.fp4_utils import dynamic_mxfp4_quant
    except ImportError as e:
        logger.warning(f"aiter MXFP4 kernels not available: {e}")
        aiter = None
        shuffle_weight = None
        dynamic_mxfp4_quant = None
        gemm_a4w4 = None

# The gemm_a4w4 ASM kernel has degraded precision when the output
# dimension (N) is smaller than its minimum tile size.
# Layers with output_size falls below this threshold will stay unquantized
_MXFP4_MIN_OUTPUT_DIM = 256
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `_is_hip`, and `_MXFP4_MIN_OUTPUT_DIM`. The code collaborates with `logging.getLogger`, `is_hip`, and `logger.warning`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`_is_hip` 和 `_MXFP4_MIN_OUTPUT_DIM` 等名称。 代码会与 `logging.getLogger`、`is_hip` 和 `logger.warning` 协同工作。

### Lines 43-50: `Mxfp4Config` class overview / `Mxfp4Config` 类概览
```python
class Mxfp4Config(QuantizationConfig):
    """
    MXFP4 quantization config for diffusion models.

    Supports online quantization from unquantized BF16/FP16 checkpoints.
    Note: MXFP4 requires ROCm and MI350+ (gfx95x).
    """
```
**EN:** This block defines class `Mxfp4Config`. MXFP4 quantization config for diffusion models. Supports online quantization from unquantized BF16/FP16 checkpoints. It inherits from `QuantizationConfig`.
**CN:** 该代码块定义了类 `Mxfp4Config`。 它用于封装 mxfp4 config 相关行为。 它继承自 `QuantizationConfig`。

### Lines 51-60: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        is_checkpoint_mxfp4_serialized: bool = False,
        ignored_layers: Optional[List[str]] = None,
        packed_modules_mapping: Optional[Dict[str, List[str]]] = None,
    ):
        super().__init__()
        self.is_checkpoint_mxfp4_serialized = is_checkpoint_mxfp4_serialized
        self.ignored_layers = ignored_layers or []
        self.packed_modules_mapping = packed_modules_mapping or {}
```
**EN:** This block defines method `__init__` on `Mxfp4Config`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `is_checkpoint_mxfp4_serialized`, `ignored_layers`, and `packed_modules_mapping` drive the behavior in this section.
**CN:** 该代码块定义了 `Mxfp4Config` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `is_checkpoint_mxfp4_serialized`、`ignored_layers` 和 `packed_modules_mapping` 等参数驱动。

### Lines 62-64: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "mxfp4"
```
**EN:** This block defines method `get_name` on `Mxfp4Config`. It retrieves name.
**CN:** 该代码块定义了 `Mxfp4Config` 的方法 `get_name`。 它用于获取name。

### Lines 66-68: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.bfloat16, torch.float16]
```
**EN:** This block defines method `get_supported_act_dtypes` on `Mxfp4Config`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `Mxfp4Config` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 70-72: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 95  # gfx95x, Note: mxfp_supported() is a better check
```
**EN:** This block defines method `get_min_capability` on `Mxfp4Config`. It retrieves min capability.
**CN:** 该代码块定义了 `Mxfp4Config` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 74-76: `get_config_filenames` implementation / `get_config_filenames` 实现
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return []  # No config file needed for online quantization
```
**EN:** This block defines method `get_config_filenames` on `Mxfp4Config`. It retrieves config filenames.
**CN:** 该代码块定义了 `Mxfp4Config` 的方法 `get_config_filenames`。 它用于获取config filenames。

### Lines 78-82: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(cls, config: dict) -> "Mxfp4Config":
        """Create from model config (for pre-quantized checkpoints)."""
        is_serialized = config.get("quant_method") == "mxfp4"
        return cls(is_checkpoint_mxfp4_serialized=is_serialized)
```
**EN:** This block defines method `from_config` on `Mxfp4Config`. Create from model config (for pre-quantized checkpoints). Key calls include `cls`, and `config.get`. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了 `Mxfp4Config` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `cls` 和 `config.get`。 本段逻辑主要由 `config` 等参数驱动。

### Lines 84-109: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(self, layer, prefix: str):
        from sglang.multimodal_gen.runtime.layers.linear import LinearBase

        if isinstance(layer, LinearBase):
            if is_layer_skipped(
                prefix,
                self.ignored_layers,
                fused_mapping=self.packed_modules_mapping,
            ):
                logger.debug(
                    f"MXFP4: Keeping layer {prefix} unquantized (in ignored_layers)"
                )
                return UnquantizedLinearMethod()
            # Skip layers whose output dims are too small, see ASM kernel comment above
            output_size = getattr(layer, "output_size", None)
            if output_size is not None and output_size < _MXFP4_MIN_OUTPUT_DIM:
                logger.info(
                    f"MXFP4: Keeping layer {prefix} unquantized "
                    f"(output_size={output_size} < {_MXFP4_MIN_OUTPUT_DIM})"
                )
                return UnquantizedLinearMethod()
            logger.debug(f"MXFP4: Replacing layer {prefix} with MXFP4 linear method")
            return Mxfp4LinearMethod(self)
        else:
            logger.debug(f"MXFP4: Skipping layer {prefix} (not a LinearBase)")
        return None
```
**EN:** This block defines method `get_quant_method` on `Mxfp4Config`. It retrieves quant method. Key calls include `isinstance`, `is_layer_skipped`, `getattr`, `logger.debug`, and `Mxfp4LinearMethod`. The implementation branches on conditions. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `Mxfp4Config` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `isinstance`、`is_layer_skipped`、`getattr`、`logger.debug` 和 `Mxfp4LinearMethod`。 实现中包含条件分支。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 112-119: `Mxfp4LinearMethod` class overview / `Mxfp4LinearMethod` 类概览
```python
class Mxfp4LinearMethod(LinearMethodBase):
    """
    MXFP4 online quantization method for linear layers.

    Quantizes unquantized BF16/FP16 weights to MXFP4 format during
    process_weights_after_loading().
    """
```
**EN:** This block defines class `Mxfp4LinearMethod`. MXFP4 online quantization method for linear layers. Quantizes unquantized BF16/FP16 weights to MXFP4 format during process_weights_after_loading(). It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `Mxfp4LinearMethod`。 它用于封装 mxfp4 linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 120-121: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, quant_config: Mxfp4Config):
        self.quant_config = quant_config
```
**EN:** This block defines method `__init__` on `Mxfp4LinearMethod`. It initializes the instance state. Parameters such as `quant_config` drive the behavior in this section.
**CN:** 该代码块定义了 `Mxfp4LinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `quant_config` 等参数驱动。

### Lines 123-157: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        """
        Creates BF16/FP16 parameters that will be
        quantized to MXFP4 in process_weights_after_loading().
        """
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=params_dtype,
            ),
            weight_loader=weight_loader,
            input_dim=1,
            output_dim=0,
        )
        layer.register_parameter("weight", weight)

        # Placeholder scale (will be created during quantization)
        weight_scale = PerTensorScaleParameter(
            data=torch.empty(1, dtype=torch.float32),
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_scale", weight_scale)
```
**EN:** This block defines method `create_weights` on `Mxfp4LinearMethod`. Creates BF16/FP16 parameters that will be quantized to MXFP4 in process_weights_after_loading(). Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, and `PerTensorScaleParameter`. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `Mxfp4LinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`extra_weight_attrs.get`、`ModelWeightParameter`、`layer.register_parameter` 和 `PerTensorScaleParameter`。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 159-210: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        """
        Quantize BF16/FP16 weights to MXFP4 after loading from checkpoint.

        Converts weights from unquantized format to:
        - Packed uint8 (2 FP4 values per byte)
        - E8M0 scales (one per 32-element block)
        """
        if not mxfp_supported():
            platform = "unknown"
            if _is_hip:
                try:
                    platform = torch.cuda.get_device_properties(0).gcnArchName
                except:
                    platform = "ROCm (unknown arch)"
            raise RuntimeError(
                f"MXFP4 quantization requires ROCm and MI350+ (gfx95x). "
                f"Current platform: {platform}."
            )

        # Check if weights are already quantized
        if layer.weight.dtype not in [torch.bfloat16, torch.float16]:
            # Already quantized or unexpected dtype
            logger.info("Weights are quantized or unexpected dtype")
            return

        if any(fn is None for fn in (dynamic_mxfp4_quant, shuffle_weight, gemm_a4w4)):
            raise RuntimeError(
                "aiter MXFP4 kernels not available. "
                "Install aiter with MXFP4 support."
            )

        weight_data = layer.weight.data
        was_on_cpu = weight_data.device.type == "cpu"
        if was_on_cpu:
            weight_data = weight_data.cuda()

        w_quant, mx_scales = dynamic_mxfp4_quant(weight_data, shuffle=True)

        w_quant_shuffled = shuffle_weight(w_quant)

        if was_on_cpu:
            w_quant_shuffled = w_quant_shuffled.cpu()
            mx_scales = mx_scales.cpu()

        layer.weight = Parameter(w_quant_shuffled, requires_grad=False)
        layer.weight_scale = Parameter(mx_scales, requires_grad=False)

        logger.debug(
            f"MXFP4: Quantized layer weights - weight {layer.weight.shape} {layer.weight.dtype}, "
            f"scale {layer.weight_scale.shape}"
        )
```
**EN:** This block defines method `process_weights_after_loading` on `Mxfp4LinearMethod`. Quantize BF16/FP16 weights to MXFP4 after loading from checkpoint. Converts weights from unquantized format to: - Packed uint8 (2 FP4 values per byte) - E8M0 scales (one per 32-element block) Key calls include `any`, `dynamic_mxfp4_quant`, `shuffle_weight`, `Parameter`, and `logger.debug`. The implementation branches on conditions, handles exceptional paths. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `Mxfp4LinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `any`、`dynamic_mxfp4_quant`、`shuffle_weight`、`Parameter` 和 `logger.debug`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 212-237: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        if not mxfp_supported():
            raise RuntimeError(
                "MXFP4 inference requires ROCm and MI350+ (gfx95x). "
                "Current platform not supported."
            )

        # Handle 3D input tensors [batch, seq, hidden]
        original_shape = x.shape
        if x.dim() == 3:
            x = x.view(-1, x.shape[-1])

        x_fp4, x_scale = dynamic_mxfp4_quant(x, shuffle=True)

        y = gemm_a4w4(x_fp4, layer.weight, x_scale, layer.weight_scale)

        if bias is not None:
            y = y + bias

        return y.view(*original_shape[:-1], layer.weight.shape[0])
```
**EN:** This block defines method `apply` on `Mxfp4LinearMethod`. It applies function. Key calls include `dynamic_mxfp4_quant`, `gemm_a4w4`, `y.view`, `mxfp_supported`, and `RuntimeError`. The implementation branches on conditions. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `Mxfp4LinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `dynamic_mxfp4_quant`、`gemm_a4w4`、`y.view`、`mxfp_supported` 和 `RuntimeError`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `Mxfp4Config`: MXFP4 quantization config for diffusion models. / 核心类，用于封装 mxfp4 config 相关行为。
- `Mxfp4LinearMethod`: MXFP4 online quantization method for linear layers. / 核心类，用于封装 mxfp4 linear method 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.nn.parameter`, `aiter`, `aiter.ops.gemm_op_a4w4`, `aiter.ops.shuffle`, `aiter.utility.fp4_utils`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`

- **Total lines / 总行数**: 237
