# awq.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/awq/awq.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for awq quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 AWQ 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
from __future__ import annotations

import logging
import warnings
from typing import TYPE_CHECKING, Any, Dict, List, Optional

import torch

from sglang.srt.layers.linear import LinearBase
from sglang.srt.layers.moe import MoeRunnerConfig
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.marlin_utils import (
    check_marlin_supported,
    check_marlin_supports_layer,
    check_moe_marlin_supports_layer,
    verify_marlin_supported,
)
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.layers.quantization.utils import get_scalar_types
from sglang.srt.utils.patch_torch import register_fake_if_exists

from .schemes import (
    AWQAscendLinearScheme,
    AWQAscendMoEScheme,
    AWQIntelAMXLinearScheme,
    AWQIntelAMXMoEScheme,
    AWQLinearScheme,
    AWQMarlinLinearScheme,
    AWQMoEScheme,
)
```
**EN:** This block imports __future__, logging, torch, typing, .schemes, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, typing, .schemes, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton 等依赖，并为当前量化实现准备模块命名空间。

### Lines 38-42: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 44-44: module imports and setup
```python
from sglang.srt.utils import is_cuda, is_hip, is_npu, is_xpu
```
**EN:** This block imports __future__, logging, torch, typing, .schemes, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, torch, typing, .schemes, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton 等依赖，并为当前量化实现准备模块命名空间。

### Lines 46-46: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 47-47: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 48-48: initialize _is_xpu
```python
_is_xpu = is_xpu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_xpu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_xpu。

### Lines 49-49: initialize _is_npu
```python
_is_npu = is_npu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_npu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_npu。

### Lines 51-52: conditional logic for not (_is_cuda or _is_hip or _is_xpu or _is_npu)
```python
if not (_is_cuda or _is_hip or _is_xpu or _is_npu):
    warnings.warn(f"Only CUDA, HIP and XPU support AWQ currently.")
```
**EN:** This block applies conditional logic controlled by `not (_is_cuda or _is_hip or _is_xpu or _is_npu)`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `not (_is_cuda or _is_hip or _is_xpu or _is_npu)` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 54-54: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 57-57: initialize (ScalarType, scalar_types)
```python
ScalarType, scalar_types = get_scalar_types()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as (ScalarType, scalar_types).
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 (ScalarType, scalar_types)。

### Lines 60-61: is_layer_skipped_awq()
```python
def is_layer_skipped_awq(prefix: str, modules_to_not_convert: List[str]):
    return any(module_name in prefix for module_name in modules_to_not_convert)
```
**EN:** This block defines `is_layer_skipped_awq()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `is_layer_skipped_awq()`，用于检查供上层控制流使用的条件。

### Lines 64-69: class AWQConfig: definition
```python
class AWQConfig(QuantizationConfig):
    """Config class for AWQ.

    Reference: https://arxiv.org/abs/2306.00978
    """
```
**EN:** This block declares `AWQConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_scaled_act_names, get_name.
**CN:** 该代码块声明 `AWQConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_scaled_act_names, get_name 等行为。

### Lines 70-88: AWQConfig.__init__()
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        zero_point: bool,
        modules_to_not_convert: Optional[List[str]] = None,
    ) -> None:
        super().__init__()
        self.weight_bits = weight_bits
        self.group_size = group_size
        self.zero_point = zero_point
        self.modules_to_not_convert = modules_to_not_convert or []

        if self.weight_bits != 4:
            raise ValueError(
                "Currently, only 4-bit weight quantization is supported for "
                f"AWQ, but got {self.weight_bits} bits."
            )
        self.pack_factor = 32 // self.weight_bits
```
**EN:** This block defines `AWQConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 90-96: AWQConfig.__repr__()
```python
    def __repr__(self) -> str:
        return (
            f"AWQConfig(weight_bits={self.weight_bits}, "
            f"group_size={self.group_size}, "
            f"zero_point={self.zero_point}, "
            f"modules_to_not_convert={self.modules_to_not_convert})"
        )
```
**EN:** This block defines `AWQConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `AWQConfig.__repr__()`，用于构造可读的调试表示。

### Lines 98-99: AWQConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `AWQConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 101-102: AWQConfig.get_name()
```python
    def get_name(self) -> str:
        return "awq"
```
**EN:** This block defines `AWQConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 104-105: AWQConfig.get_supported_act_dtypes()
```python
    def get_supported_act_dtypes(self) -> List[torch.dtype]:
        return [torch.float16] if not _is_npu else [torch.float16, torch.bfloat16]
```
**EN:** This block defines `AWQConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 107-115: AWQConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # The AWQ kernel only supports Turing or newer GPUs.
        if _is_npu:
            raise NotImplementedError(
                'NPU hardware does not support "get_min_capability" feature.'
            )
        else:
            return 75
```
**EN:** This block defines `AWQConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 117-123: AWQConfig.get_config_filenames()
```python
    @staticmethod
    def get_config_filenames() -> List[str]:
        return [
            "quant_config.json",  # E.g., casperhansen/vicuna-7b-v1.5-awq
            # E.g., abhinavkulkarni/mosaicml-mpt-7b-instruct-w4-g128-awq
            "quantize_config.json",
        ]
```
**EN:** This block defines `AWQConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 125-133: AWQConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> AWQConfig:
        weight_bits = cls.get_from_keys(config, ["w_bit", "bits"])
        group_size = cls.get_from_keys(config, ["q_group_size", "group_size"])
        zero_point = cls.get_from_keys(config, ["zero_point"])
        modules_to_not_convert = cls.get_from_keys_or(
            config, ["modules_to_not_convert"], None
        )
        return cls(weight_bits, group_size, zero_point, modules_to_not_convert)
```
**EN:** This block defines `AWQConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `AWQConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 135-157: AWQConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[LinearMethodBase]:
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if _is_npu:
            if isinstance(layer, LinearBase):
                if is_layer_skipped_awq(prefix, self.modules_to_not_convert):
                    return UnquantizedLinearMethod()
                layer.scheme = self.get_linear_scheme(layer)
                return AWQLinearMethod(self)
            elif isinstance(layer, FusedMoE):
                layer.scheme = self.get_moe_scheme(layer)
                return AWQMoEMethod(self)
            return None

        if isinstance(layer, LinearBase):
            if is_layer_skipped_awq(prefix, self.modules_to_not_convert):
                return UnquantizedLinearMethod()
            layer.scheme = self.get_linear_scheme(layer)
            return AWQLinearMethod(self)
        return None
```
**EN:** This block defines `AWQConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 159-165: AWQConfig.get_linear_scheme()
```python
    def get_linear_scheme(self, layer: torch.nn.Module):
        assert isinstance(layer, LinearBase)
        # TODO: move platform-specific AWQ scheme selection into the platform
        # plugin factory once quantization hooks are available there.
        if _is_npu:
            return AWQAscendLinearScheme(self)
        return AWQLinearScheme(self)
```
**EN:** This block defines `AWQConfig.get_linear_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_linear_scheme()`，用于为调用方获取或计算派生值。

### Lines 167-174: AWQConfig.get_moe_scheme()
```python
    def get_moe_scheme(self, layer: torch.nn.Module):
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        assert isinstance(layer, FusedMoE)
        # This is currently only reached by the NPU path in get_quant_method.
        if _is_npu:
            return AWQAscendMoEScheme(self)
        raise NotImplementedError("AWQConfig only supports MoE scheme on NPU.")
```
**EN:** This block defines `AWQConfig.get_moe_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQConfig.get_moe_scheme()`，用于为调用方获取或计算派生值。

### Lines 177-179: class AWQCPUConfig: definition
```python
class AWQCPUConfig(AWQConfig):
    """CPU Config class for AWQ, inherit from AWQConfig"""
```
**EN:** This block declares `AWQCPUConfig`, a configuration class for the quantization stack. It organizes behaviors such as get_supported_act_dtypes, get_quant_method, get_linear_scheme, get_moe_scheme.
**CN:** 该代码块声明 `AWQCPUConfig`，它是量化栈中的配置类，组织了 get_supported_act_dtypes, get_quant_method, get_linear_scheme, get_moe_scheme 等行为。

### Lines 180-181: AWQCPUConfig.get_supported_act_dtypes()
```python
    def get_supported_act_dtypes(self) -> List[torch.dtype]:
        return [torch.float16, torch.bfloat16]
```
**EN:** This block defines `AWQCPUConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQCPUConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 183-197: AWQCPUConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[LinearMethodBase]:
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, LinearBase):
            if is_layer_skipped_awq(prefix, self.modules_to_not_convert):
                return UnquantizedLinearMethod()
            layer.scheme = self.get_linear_scheme(layer)
            return AWQLinearMethod(self)
        elif isinstance(layer, FusedMoE):
            layer.scheme = self.get_moe_scheme(layer)
            return AWQMoEMethod(self)
        return None
```
**EN:** This block defines `AWQCPUConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQCPUConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 199-203: AWQCPUConfig.get_linear_scheme()
```python
    def get_linear_scheme(self, layer: torch.nn.Module):
        from sglang.srt.layers.linear import LinearBase

        assert isinstance(layer, LinearBase)
        return AWQIntelAMXLinearScheme(self)
```
**EN:** This block defines `AWQCPUConfig.get_linear_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQCPUConfig.get_linear_scheme()`，用于为调用方获取或计算派生值。

### Lines 205-209: AWQCPUConfig.get_moe_scheme()
```python
    def get_moe_scheme(self, layer: torch.nn.Module):
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        assert isinstance(layer, FusedMoE)
        return AWQIntelAMXMoEScheme(self)
```
**EN:** This block defines `AWQCPUConfig.get_moe_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQCPUConfig.get_moe_scheme()`，用于为调用方获取或计算派生值。

### Lines 212-215: class AWQMarlinConfig: definition
```python
class AWQMarlinConfig(QuantizationConfig):
    """Config class for AWQ Marlin"""

    # num_bits -> type
```
**EN:** This block declares `AWQMarlinConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_scaled_act_names, get_name.
**CN:** 该代码块声明 `AWQMarlinConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_scaled_act_names, get_name 等行为。

### Lines 216-219: AWQMarlinConfig member: initialize TYPE_MAP
```python
    TYPE_MAP = {
        4: scalar_types.uint4,
        8: scalar_types.uint8,
    }
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as TYPE_MAP.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 TYPE_MAP。

### Lines 221-251: AWQMarlinConfig.__init__()
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        zero_point: bool,
        lm_head_quantized: bool,
        modules_to_not_convert: Optional[list[str]],
        full_config: dict[str, Any],
    ) -> None:
        super().__init__()
        if _is_hip:
            warnings.warn(f"HIP does not support fused_marlin_moe currently.")
        self.pack_factor = 32 // weight_bits  # packed into int32
        self.group_size = group_size
        self.zero_point = zero_point
        self.lm_head_quantized = lm_head_quantized
        self.weight_bits = weight_bits
        self.modules_to_not_convert = modules_to_not_convert or []
        self.full_config = full_config

        if self.weight_bits not in self.TYPE_MAP:
            raise ValueError(
                f"Unsupported num_bits = {self.weight_bits}. "
                f"Supported num_bits = {self.TYPE_MAP.keys()}"
            )

        self.quant_type = self.TYPE_MAP[self.weight_bits]

        verify_marlin_supported(
            self.quant_type, group_size=self.group_size, has_zp=self.zero_point
        )
```
**EN:** This block defines `AWQMarlinConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQMarlinConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 253-260: AWQMarlinConfig.__repr__()
```python
    def __repr__(self) -> str:
        return (
            f"AWQMarlinConfig(quant_type={self.quant_type}, "
            f"group_size={self.group_size}, "
            f"zero_point={self.zero_point}, "
            f"lm_head_quantized={self.lm_head_quantized}, "
            f"modules_to_not_convert={self.modules_to_not_convert})"
        )
```
**EN:** This block defines `AWQMarlinConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `AWQMarlinConfig.__repr__()`，用于构造可读的调试表示。

### Lines 262-263: AWQMarlinConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `AWQMarlinConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 265-267: AWQMarlinConfig.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "awq_marlin"
```
**EN:** This block defines `AWQMarlinConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 269-271: AWQMarlinConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.half, torch.bfloat16]
```
**EN:** This block defines `AWQMarlinConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 273-275: AWQMarlinConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines `AWQMarlinConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 277-279: AWQMarlinConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return ["quantize_config.json"]
```
**EN:** This block defines `AWQMarlinConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 281-297: AWQMarlinConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> AWQMarlinConfig:
        weight_bits = cls.get_from_keys(config, ["bits"])
        group_size = cls.get_from_keys(config, ["group_size"])
        zero_point = cls.get_from_keys(config, ["zero_point"])
        lm_head_quantized = cls.get_from_keys_or(config, ["lm_head"], default=False)
        modules_to_not_convert = cls.get_from_keys_or(
            config, ["modules_to_not_convert"], None
        )
        return cls(
            weight_bits,
            group_size,
            zero_point,
            lm_head_quantized,
            modules_to_not_convert,
            config,
        )
```
**EN:** This block defines `AWQMarlinConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `AWQMarlinConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 299-321: AWQMarlinConfig.override_quantization_method()
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_cfg, user_quant) -> Optional[str]:
        can_convert = cls.is_awq_marlin_compatible(hf_quant_cfg)
        is_valid_user_quant = (
            user_quant is None or user_quant == "marlin" or user_quant == "awq_marlin"
        )

        if can_convert and is_valid_user_quant:
            msg = (
                "The model is convertible to {} during runtime."
                " Using {} kernel.".format(cls.get_name(), cls.get_name())
            )
            logger.info(msg)
            return cls.get_name()

        if can_convert and user_quant == "awq":
            logger.info(
                "Detected that the model can run with awq_marlin"
                ", however you specified quantization=awq explicitly,"
                " so forcing awq. Use quantization=awq_marlin for"
                " faster inference"
            )
        return None
```
**EN:** This block defines `AWQMarlinConfig.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `AWQMarlinConfig.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 323-358: AWQMarlinConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE
        from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead

        if isinstance(layer, LinearBase) or (
            isinstance(layer, ParallelLMHead) and self.lm_head_quantized
        ):
            if is_layer_skipped_awq(prefix, self.modules_to_not_convert):
                return UnquantizedLinearMethod()
            # Check if the layer is supported by AWQMarlin.
            if not check_marlin_supports_layer(layer, self.group_size):
                logger.warning_once(
                    "Layer '%s' is not supported by AWQMarlin. Falling back to unoptimized AWQ kernels.",  # noqa: E501
                    prefix,
                )
                return AWQConfig.from_config(self.full_config).get_quant_method(
                    layer, prefix
                )
            layer.scheme = self.get_linear_scheme(layer)
            return AWQLinearMethod(self)
        elif isinstance(layer, FusedMoE):
            from sglang.srt.layers.quantization.moe_wna16 import MoeWNA16Config

            if not check_moe_marlin_supports_layer(layer, self.group_size):
                logger.warning_once(
                    f"Layer '{prefix}' is not supported by AWQMoeMarlin. "
                    "Falling back to Moe WNA16 kernels."
                )
                return MoeWNA16Config.from_config(self.full_config).get_quant_method(
                    layer, prefix
                )
            layer.scheme = self.get_moe_scheme(layer)
            return AWQMoEMethod(self)
        return None
```
**EN:** This block defines `AWQMarlinConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 360-361: AWQMarlinConfig.get_linear_scheme()
```python
    def get_linear_scheme(self, layer: torch.nn.Module):
        return AWQMarlinLinearScheme(self)
```
**EN:** This block defines `AWQMarlinConfig.get_linear_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_linear_scheme()`，用于为调用方获取或计算派生值。

### Lines 363-364: AWQMarlinConfig.get_moe_scheme()
```python
    def get_moe_scheme(self, layer: torch.nn.Module):
        return AWQMoEScheme(self)
```
**EN:** This block defines `AWQMarlinConfig.get_moe_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `AWQMarlinConfig.get_moe_scheme()`，用于为调用方获取或计算派生值。

### Lines 366-389: AWQMarlinConfig.is_awq_marlin_compatible()
```python
    @classmethod
    def is_awq_marlin_compatible(cls, quant_config: dict[str, Any]):
        # Extract data from quant config.
        quant_method = quant_config.get("quant_method", "").lower()
        num_bits = quant_config.get("bits")
        group_size = quant_config.get("group_size")
        zero_point = quant_config.get("zero_point")

        if not _is_cuda:
            return False

        if quant_method != "awq":
            return False

        # If we cannot find the info needed in the config, cannot convert.
        if num_bits is None or group_size is None or zero_point is None:
            return False

        if num_bits not in cls.TYPE_MAP:
            return False

        return check_marlin_supported(
            quant_type=cls.TYPE_MAP[num_bits], group_size=group_size, has_zp=zero_point
        )
```
**EN:** This block defines `AWQMarlinConfig.is_awq_marlin_compatible()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `AWQMarlinConfig.is_awq_marlin_compatible()`，用于检查供上层控制流使用的条件。

### Lines 392-398: class AWQLinearMethod: definition
```python
class AWQLinearMethod(LinearMethodBase):
    """Linear method for AWQ.

    Args:
        quant_config: The AWQ quantization config.
    """
```
**EN:** This block declares `AWQLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `AWQLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 399-400: AWQLinearMethod.__init__()
```python
    def __init__(self, quant_config: AWQConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `AWQLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 402-421: AWQLinearMethod.create_weights()
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
        layer.scheme.create_weights(
            layer=layer,
            input_size_per_partition=input_size_per_partition,
            output_partition_sizes=output_partition_sizes,
            input_size=input_size,
            output_size=output_size,
            params_dtype=params_dtype,
            weight_loader=weight_loader,
        )
```
**EN:** This block defines `AWQLinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQLinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 423-424: AWQLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** This block defines `AWQLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `AWQLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 426-432: AWQLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return layer.scheme.apply_weights(layer, x, bias)
```
**EN:** This block defines `AWQLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `AWQLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 435-436: class AWQMoEMethod: definition
```python
class AWQMoEMethod(FusedMoEMethodBase):
```
**EN:** This block declares `AWQMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `AWQMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 437-441: AWQMoEMethod.__init__()
```python
    def __init__(self, quant_config: AWQMarlinConfig):
        self.quant_config = quant_config
        self.quant_type = scalar_types.uint4
        if self.quant_config.weight_bits != 4:
            raise ValueError("AWQMoEMethod only supports 4bit now.")
```
**EN:** This block defines `AWQMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `AWQMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 443-459: AWQMoEMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        layer.scheme.create_weights(
            layer=layer,
            num_experts=num_experts,
            hidden_size=hidden_size,
            intermediate_size_per_partition=intermediate_size_per_partition,
            params_dtype=params_dtype,
            **extra_weight_attrs,
        )
```
**EN:** This block defines `AWQMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 461-462: AWQMoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** This block defines `AWQMoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `AWQMoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 464-467: AWQMoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        layer.scheme.create_moe_runner(layer, moe_runner_config)
```
**EN:** This block defines `AWQMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `AWQMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 469-474: AWQMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        return layer.scheme.apply_weights(layer, dispatch_output)
```
**EN:** This block defines `AWQMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `AWQMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 478-484: conditional logic for _is_cuda
```python
if _is_cuda:

    @register_fake_if_exists("sgl_kernel::awq_marlin_repack")
    def _(b_q_weight, size_k, size_n, num_bits):
        return b_q_weight.new_empty(
            (size_k // 16, size_n * (num_bits // 2)), dtype=b_q_weight.dtype
        )
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

## Key Concepts / 关键概念
- `is_layer_skipped_awq()` : A public function that checks a condition used by higher-level control flow. / `is_layer_skipped_awq()`：一个公开函数，用于检查供上层控制流使用的条件。
- `AWQConfig`: A configuration class that structures file-level quantization behavior. / `AWQConfig` 是一个配置类，用于组织该文件中的量化行为。
- `AWQCPUConfig`: A configuration class that structures file-level quantization behavior. / `AWQCPUConfig` 是一个配置类，用于组织该文件中的量化行为。
- `AWQMarlinConfig`: A configuration class that structures file-level quantization behavior. / `AWQMarlinConfig` 是一个配置类，用于组织该文件中的量化行为。
- `AWQLinearMethod`: A runtime method class that structures file-level quantization behavior. / `AWQLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `AWQMoEMethod`: A runtime method class that structures file-level quantization behavior. / `AWQMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `torch`, `typing`, `warnings`
- **Internal / 内部**: `.schemes`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.moe_wna16`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.quantization.utils`, `sglang.srt.layers.vocab_parallel_embedding`, `sglang.srt.utils`, `sglang.srt.utils.patch_torch`
