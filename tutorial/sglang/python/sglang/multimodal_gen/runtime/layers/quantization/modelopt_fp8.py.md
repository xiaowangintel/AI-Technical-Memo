# modelopt_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/modelopt_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `ModelOptFp8Config`, and `ModelOptFp8LinearMethod`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: ModelOpt FP8 quantization support for diffusion models. Handles checkpoints produced by NVIDIA Model Optimizer (ModelOpt) with ``quant_algo: "FP8"`` and ``quant_method: "modelopt"``. / 该文件属于运行时算子层。它围绕 `ModelOptFp8Config` 和 `ModelOptFp8LinearMethod` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: module setup and imports / 模块初始化与导入
```python
"""ModelOpt FP8 quantization support for diffusion models.

Handles checkpoints produced by NVIDIA Model Optimizer (ModelOpt) with
``quant_algo: "FP8"`` and ``quant_method: "modelopt"``.

Per quantized linear layer the checkpoint contains:
    .weight         float8_e4m3fn  [out, in]   FP8 quantized weight
    .weight_scale   float32        scalar       per-tensor weight scale
    .input_scale    float32        scalar       per-tensor static activation scale
    .bias           bfloat16       [out]        bias (unquantized)
    ._amax          (ignored)                   calibration artifact

Layers listed in the ``ignore`` field of the quantization config remain in
bfloat16 and use the standard unquantized linear method.
"""

from __future__ import annotations

import fnmatch
import logging
from typing import Any, Dict, List, Optional

import torch

from sglang.multimodal_gen.runtime.layers.linear import (
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.multimodal_gen.runtime.models.parameter import (
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.srt.layers.quantization.fp8_utils import (
    apply_fp8_linear,
    cutlass_fp8_supported,
)
from sglang.srt.layers.quantization.utils import convert_to_channelwise
```
**EN:** This block establishes the module context and imports `__future__`, `fnmatch`, `logging`, `typing`, `torch`, and `sglang.multimodal_gen.runtime.layers.linear`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`fnmatch`、`logging`、`typing`、`torch` 和 `sglang.multimodal_gen.runtime.layers.linear`。这些依赖为后续实现提供所需符号。

### Lines 43-43: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 46-48: `ModelOptFp8Config` class overview / `ModelOptFp8Config` 类概览
```python
class ModelOptFp8Config(QuantizationConfig):
    """Config for ModelOpt static per-tensor FP8 quantization."""
```
**EN:** This block defines class `ModelOptFp8Config`. Config for ModelOpt static per-tensor FP8 quantization. It inherits from `QuantizationConfig`.
**CN:** 该代码块定义了类 `ModelOptFp8Config`。 它用于封装 model opt fp8 config 相关行为。 它继承自 `QuantizationConfig`。

### Lines 49-56: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        is_checkpoint_fp8_serialized: bool = True,
        ignore: Optional[List[str]] = None,
    ) -> None:
        super().__init__()
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
        self.ignore = ignore or []
```
**EN:** This block defines method `__init__` on `ModelOptFp8Config`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `is_checkpoint_fp8_serialized`, and `ignore` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `is_checkpoint_fp8_serialized` 和 `ignore` 等参数驱动。

### Lines 60-62: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "modelopt"
```
**EN:** This block defines method `get_name` on `ModelOptFp8Config`. It retrieves name.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_name`。 它用于获取name。

### Lines 64-66: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.bfloat16, torch.half]
```
**EN:** This block defines method `get_supported_act_dtypes` on `ModelOptFp8Config`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 68-70: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 89
```
**EN:** This block defines method `get_min_capability` on `ModelOptFp8Config`. It retrieves min capability.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 72-74: `get_config_filenames` implementation / `get_config_filenames` 实现
```python
    @staticmethod
    def get_config_filenames() -> list[str]:
        return []
```
**EN:** This block defines method `get_config_filenames` on `ModelOptFp8Config`. It retrieves config filenames.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_config_filenames`。 它用于获取config filenames。

### Lines 76-88: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> "ModelOptFp8Config":
        quant_algo = config.get("quant_algo")
        if quant_algo is None:
            raise ValueError(
                "ModelOptFp8Config requires 'quant_algo' in the quantization config."
            )
        if "FP8" not in quant_algo:
            raise ValueError(
                f"ModelOptFp8Config only supports FP8, got quant_algo={quant_algo!r}."
            )
        ignore = config.get("ignore", [])
        return cls(is_checkpoint_fp8_serialized=True, ignore=ignore)
```
**EN:** This block defines method `from_config` on `ModelOptFp8Config`. It constructs from config. Key calls include `config.get`, `cls`, and `ValueError`. The implementation branches on conditions. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `config.get`、`cls` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `config` 等参数驱动。

### Lines 90-104: `_is_layer_ignored` implementation / `_is_layer_ignored` 实现
```python
    def _is_layer_ignored(self, prefix: str) -> bool:
        """Check whether *prefix* matches any pattern in the ignore list.

        ModelOpt ignore patterns are matched against the full prefix as a glob
        (e.g. ``"norm_out*"`` matches ``"norm_out.linear"``) **and** against the
        first path component (e.g. ``"proj_out"`` matches only the top-level
        ``proj_out``, not ``single_transformer_blocks.0.proj_out``).
        """
        first_component = prefix.split(".")[0]
        for pattern in self.ignore:
            if fnmatch.fnmatch(prefix, pattern):
                return True
            if fnmatch.fnmatch(first_component, pattern):
                return True
        return False
```
**EN:** This block defines method `_is_layer_ignored` on `ModelOptFp8Config`. Check whether *prefix* matches any pattern in the ignore list. ModelOpt ignore patterns are matched against the full prefix as a glob (e.g. Key calls include `prefix.split`, and `fnmatch.fnmatch`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `_is_layer_ignored`。 它用于处理 is layer ignored 相关逻辑。 关键调用包括 `prefix.split` 和 `fnmatch.fnmatch`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `prefix` 等参数驱动。

### Lines 106-115: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        from sglang.multimodal_gen.runtime.layers.linear import LinearBase

        if isinstance(layer, LinearBase):
            if self._is_layer_ignored(prefix):
                return UnquantizedLinearMethod()
            return ModelOptFp8LinearMethod(self)
        return None
```
**EN:** This block defines method `get_quant_method` on `ModelOptFp8Config`. It retrieves quant method. Key calls include `isinstance`, `self._is_layer_ignored`, `ModelOptFp8LinearMethod`, and `UnquantizedLinearMethod`. The implementation branches on conditions. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `isinstance`、`self._is_layer_ignored`、`ModelOptFp8LinearMethod` 和 `UnquantizedLinearMethod`。 实现中包含条件分支。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 117-118: `get_scaled_act_names` implementation / `get_scaled_act_names` 实现
```python
    def get_scaled_act_names(self) -> list[str]:
        return []
```
**EN:** This block defines method `get_scaled_act_names` on `ModelOptFp8Config`. It retrieves scaled act names.
**CN:** 该代码块定义了 `ModelOptFp8Config` 的方法 `get_scaled_act_names`。 它用于获取scaled act names。

### Lines 121-127: `ModelOptFp8LinearMethod` class overview / `ModelOptFp8LinearMethod` 类概览
```python
class ModelOptFp8LinearMethod(LinearMethodBase):
    """Linear method for ModelOpt static per-tensor FP8 quantization.

    Uses ``torch._scaled_mm`` (or CUTLASS FP8 GEMM when available) for
    the FP8 matrix multiply — the same kernels used by the LLM runtime.
    """
```
**EN:** This block defines class `ModelOptFp8LinearMethod`. Linear method for ModelOpt static per-tensor FP8 quantization. Uses ``torch._scaled_mm`` (or CUTLASS FP8 GEMM when available) for the FP8 matrix multiply — the same kernels used by the LLM runtime. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `ModelOptFp8LinearMethod`。 它用于封装 model opt fp8 linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 128-130: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, quant_config: ModelOptFp8Config):
        self.quant_config = quant_config
        self.cutlass_fp8_supported = cutlass_fp8_supported()
```
**EN:** This block defines method `__init__` on `ModelOptFp8LinearMethod`. It initializes the instance state. Key calls include `cutlass_fp8_supported`. Parameters such as `quant_config` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `cutlass_fp8_supported`。 本段逻辑主要由 `quant_config` 等参数驱动。

### Lines 132-170: `create_weights` implementation / `create_weights` 实现
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
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=torch.float8_e4m3fn,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        for scale_name in ("weight_scale", "input_scale"):
            scale = PerTensorScaleParameter(
                data=torch.full(
                    (len(output_partition_sizes),),
                    torch.finfo(torch.float32).min,
                    dtype=torch.float32,
                ),
                weight_loader=weight_loader,
            )
            layer.register_parameter(scale_name, scale)
```
**EN:** This block defines method `create_weights` on `ModelOptFp8LinearMethod`. It creates weights. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, and `PerTensorScaleParameter`. The implementation iterates over collections or steps. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`extra_weight_attrs.get`、`ModelWeightParameter`、`layer.register_parameter` 和 `PerTensorScaleParameter`。 实现中会遍历集合或步骤。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 172-189: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Diffusion models use single-partition layers (no TP, no fused QKV),
        # so we just take the max scale directly without the
        # dequantize-requantize round-trip that the LLM path does (which
        # requires CUDA kernels that are unavailable during CPU-phase loading).
        max_w_scale = layer.weight_scale.max()

        # Transpose weight to [in, out] column-major layout for
        # apply_fp8_linear / CUTLASS fp8_scaled_mm.  Do NOT call
        # .contiguous() — the kernel requires column-major stride.
        layer.weight = torch.nn.Parameter(layer.weight.data.t(), requires_grad=False)

        if self.cutlass_fp8_supported:
            max_w_scale = convert_to_channelwise(max_w_scale, layer.logical_widths)
        layer.weight_scale = torch.nn.Parameter(max_w_scale, requires_grad=False)
        layer.input_scale = torch.nn.Parameter(
            layer.input_scale.max(), requires_grad=False
        )
```
**EN:** This block defines method `process_weights_after_loading` on `ModelOptFp8LinearMethod`. It processes weights after loading. Key calls include `layer.weight_scale.max`, `torch.nn.Parameter`, `layer.weight.data.t`, `convert_to_channelwise`, and `layer.input_scale.max`. The implementation branches on conditions. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `layer.weight_scale.max`、`torch.nn.Parameter`、`layer.weight.data.t`、`convert_to_channelwise` 和 `layer.input_scale.max`。 实现中包含条件分支。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 191-204: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return apply_fp8_linear(
            input=x,
            weight=layer.weight,
            weight_scale=layer.weight_scale,
            input_scale=layer.input_scale,
            bias=bias,
            cutlass_fp8_supported=self.cutlass_fp8_supported,
        )
```
**EN:** This block defines method `apply` on `ModelOptFp8LinearMethod`. It applies function. Key calls include `apply_fp8_linear`. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelOptFp8LinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `apply_fp8_linear`。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `ModelOptFp8Config`: Config for ModelOpt static per-tensor FP8 quantization. / 核心类，用于封装 model opt fp8 config 相关行为。
- `ModelOptFp8LinearMethod`: Linear method for ModelOpt static per-tensor FP8 quantization. / 核心类，用于封装 model opt fp8 linear method 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `fnmatch`, `logging`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.utils`

- **Total lines / 总行数**: 204
