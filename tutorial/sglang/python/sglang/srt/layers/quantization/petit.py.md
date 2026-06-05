# petit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/petit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for petit quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 Petit 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/quantization/modelopt.py


import logging
from typing import Any, Dict, List, Optional

import regex as re
import torch
from torch.nn.parameter import Parameter

from sglang.srt.layers.linear import LinearBase
from sglang.srt.layers.parameter import ModelWeightParameter, PerTensorScaleParameter
from sglang.srt.layers.quantization.base_config import (
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.petit_utils import (
    apply_petit_nvfp4_linear,
    prepare_nvfp4_layer_for_petit,
    verify_petit_nvfp4_supported,
)
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.layers.quantization.utils import is_layer_skipped
from sglang.srt.utils import is_hip
```
**EN:** This block imports logging, regex, torch, torch.nn.parameter, sglang.srt.layers.linear, sglang.srt.layers.parameter, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.petit_utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 logging, regex, torch, torch.nn.parameter, sglang.srt.layers.linear, sglang.srt.layers.parameter, sglang.srt.layers.quantization.base_config, sglang.srt.layers.quantization.petit_utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 29-29: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 32-32: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 36-38: class PetitNvFp4Config: definition
```python
class PetitNvFp4Config(QuantizationConfig):
    """Config class for Petit FP4."""
```
**EN:** This block declares `PetitNvFp4Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_name, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `PetitNvFp4Config`，它是量化栈中的配置类，组织了 __init__, get_name, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 39-54: PetitNvFp4Config.__init__()
```python
    def __init__(
        self,
        is_checkpoint_nvfp4_serialized: bool = False,
        kv_cache_quant_algo: str = None,
        group_size: int = None,
        exclude_modules: List[str] = None,
    ) -> None:
        self.is_checkpoint_nvfp4_serialized = is_checkpoint_nvfp4_serialized
        if is_checkpoint_nvfp4_serialized:
            logger.warning(
                "Detected nvfp4 checkpoint. Please note that the "
                "format is experimental and subject to change."
            )
        self.group_size = group_size
        self.kv_cache_quant_algo = kv_cache_quant_algo
        self.exclude_modules = exclude_modules
```
**EN:** This block defines `PetitNvFp4Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `PetitNvFp4Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 56-58: PetitNvFp4Config.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "petit_nvfp4"
```
**EN:** This block defines `PetitNvFp4Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `PetitNvFp4Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 60-62: PetitNvFp4Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.half]
```
**EN:** This block defines `PetitNvFp4Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `PetitNvFp4Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 64-67: PetitNvFp4Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # Petit supports the gfx90a and gfx942 GPUs
        return 90
```
**EN:** This block defines `PetitNvFp4Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `PetitNvFp4Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 69-71: PetitNvFp4Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return ["hf_quant_config.json"]
```
**EN:** This block defines `PetitNvFp4Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `PetitNvFp4Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 73-101: PetitNvFp4Config.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> "PetitNvFp4Config":
        quant_config = cls.get_from_keys(config, ["quantization"])
        quant_method = quant_config["quant_algo"]
        group_size = quant_config.get("group_size", None)
        verify_petit_nvfp4_supported(quant_method, group_size)

        is_checkpoint_nvfp4_serialized = "NVFP4" in quant_method
        kv_cache_quant_algo = quant_config["kv_cache_quant_algo"]
        if not kv_cache_quant_algo:
            kv_cache_quant_algo = "auto"
        exclude_modules = quant_config.get("exclude_modules", None)
        if not (group_size and kv_cache_quant_algo and (exclude_modules is not None)):
            logger.warning(
                f"group_size: {group_size},"
                f"kv_cache_quant_algo: {kv_cache_quant_algo},"
                f"exclude_modules: {exclude_modules}"
            )
            raise ValueError(
                "NVFP4 quantization requires group size and "
                "kv_cache_quant_algo specified in "
                "hf_quant_config.json"
            )
        return cls(
            is_checkpoint_nvfp4_serialized,
            kv_cache_quant_algo,
            group_size,
            exclude_modules,
        )
```
**EN:** This block defines `PetitNvFp4Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `PetitNvFp4Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 103-108: PetitNvFp4Config.override_quantization_method()
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_cfg, user_quant) -> Optional[str]:
        can_convert = cls.is_petit_nvfp4_compatible(hf_quant_cfg)
        if can_convert:
            return cls.get_name()
        return None
```
**EN:** This block defines `PetitNvFp4Config.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `PetitNvFp4Config.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 110-113: PetitNvFp4Config.is_petit_nvfp4_compatible()
```python
    @classmethod
    def is_petit_nvfp4_compatible(cls, quant_config: Dict[str, Any]) -> bool:
        quant_method = quant_config.get("quant_method", "").lower()
        return _is_hip and quant_method == "modelopt"
```
**EN:** This block defines `PetitNvFp4Config.is_petit_nvfp4_compatible()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `PetitNvFp4Config.is_petit_nvfp4_compatible()`，用于检查供上层控制流使用的条件。

### Lines 115-120: PetitNvFp4Config.is_layer_excluded()
```python
    def is_layer_excluded(self, prefix: str, exclude_modules: list):
        for pattern in exclude_modules:
            regex_str = pattern.replace(".", r"\.").replace("*", r".*")
            if re.fullmatch(regex_str, prefix):
                return True
        return False
```
**EN:** This block defines `PetitNvFp4Config.is_layer_excluded()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `PetitNvFp4Config.is_layer_excluded()`，用于检查供上层控制流使用的条件。

### Lines 122-131: PetitNvFp4Config.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional["QuantizeMethodBase"]:
        if isinstance(layer, LinearBase):
            if is_layer_skipped(prefix, self.exclude_modules) or self.is_layer_excluded(
                prefix, self.exclude_modules
            ):
                return UnquantizedLinearMethod()
            return PetitNvFp4LinearMethod(self)
        return None
```
**EN:** This block defines `PetitNvFp4Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `PetitNvFp4Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 133-134: PetitNvFp4Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `PetitNvFp4Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `PetitNvFp4Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 137-151: class PetitNvFp4LinearMethod: definition
```python
class PetitNvFp4LinearMethod(LinearMethodBase):
    """Linear method for NVFP4.
    Supports loading NVFP4 checkpoints with the following structure:

    |Tensor Name           | datatype      |  shape      |
    |----------------------------------------------------|
    |input_scale           | torch.float32 | scalar      |
    |weight                | NVFP4(SE2M1)  | [1, X, y/2] |
    |weight_scale          | FP8-E4M3      | [X, Y]      |
    |weight_scale_2        | torch.float32 | scalar      |

    The weights are quantized per block of 16 elements.
    Args: quant_config: The ModelOpt quantization config.
    """
```
**EN:** This block declares `PetitNvFp4LinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `PetitNvFp4LinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 152-153: PetitNvFp4LinearMethod.__init__()
```python
    def __init__(self, quant_config: PetitNvFp4Config):
        self.quant_config = quant_config
```
**EN:** This block defines `PetitNvFp4LinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `PetitNvFp4LinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 155-199: PetitNvFp4LinearMethod.create_weights() (part 1/2)
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
        del input_size, output_size
        if not self.quant_config.is_checkpoint_nvfp4_serialized:
            raise ValueError(
                "NVFP4 quantization was selected, "
                " dynamic quantization is not supported."
            )

        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        layer.logical_widths = output_partition_sizes

        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        if input_size_per_partition % 16 != 0:
            raise ValueError(
                "Unsupported model when in features size is " "not multiple of 16"
            )

        weight_dtype = (
            torch.float8_e4m3fn
            if self.quant_config.is_checkpoint_nvfp4_serialized
            else params_dtype
        )

        weight = ModelWeightParameter(
            data=torch.empty(
                # 2 fp4 data is packed in one uint8 in the input dimension
                output_size_per_partition,
                input_size_per_partition // 2,
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
```
**EN:** This segment of `PetitNvFp4LinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `PetitNvFp4LinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 200-227: PetitNvFp4LinearMethod.create_weights() (part 2/2)
```python
        )
        layer.register_parameter("weight", weight)

        input_scale = PerTensorScaleParameter(
            data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
            weight_loader=weight_loader,
        )

        layer.register_parameter("input_scale", input_scale)

        weight_scale_2 = PerTensorScaleParameter(
            data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_scale_2", weight_scale_2)

        weight_scale = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // self.quant_config.group_size,
                dtype=weight_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )

        layer.register_parameter("weight_scale", weight_scale)
```
**EN:** This segment of `PetitNvFp4LinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `PetitNvFp4LinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 229-239: PetitNvFp4LinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        input_scale_2 = layer.input_scale.max().to(torch.float32)
        weight_scale_2 = layer.weight_scale_2.max().to(torch.float32)
        layer.input_scale = Parameter(input_scale_2, requires_grad=False)
        layer.weight_scale_2 = Parameter(weight_scale_2, requires_grad=False)
        layer.alpha = Parameter(
            layer.input_scale * layer.weight_scale_2, requires_grad=False
        )

        prepare_nvfp4_layer_for_petit(layer)
        del layer.input_scale
```
**EN:** This block defines `PetitNvFp4LinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `PetitNvFp4LinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 241-255: PetitNvFp4LinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return apply_petit_nvfp4_linear(
            input=x,
            weight=layer.weight,
            weight_scale=layer.weight_scale,
            weight_scale_2=layer.weight_scale_2,
            size_n=layer.output_size_per_partition,
            size_k=layer.input_size_per_partition,
            bias=bias,
        )
```
**EN:** This block defines `PetitNvFp4LinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `PetitNvFp4LinearMethod.apply()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `PetitNvFp4Config`: A configuration class that structures file-level quantization behavior. / `PetitNvFp4Config` 是一个配置类，用于组织该文件中的量化行为。
- `PetitNvFp4LinearMethod`: A runtime method class that structures file-level quantization behavior. / `PetitNvFp4LinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `logging`, `regex`, `torch`, `torch.nn.parameter`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.petit_utils`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`
