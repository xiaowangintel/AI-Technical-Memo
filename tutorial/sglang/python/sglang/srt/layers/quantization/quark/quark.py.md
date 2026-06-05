# quark.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/quark/quark.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for quark quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 Quark 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0

import fnmatch
import logging
from typing import TYPE_CHECKING, Any, List, Optional, cast

import torch

from sglang.srt.layers.linear import LinearBase
from sglang.srt.layers.moe import MoeRunnerConfig
from sglang.srt.layers.quantization.base_config import (  # noqa: E501
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.kv_cache import BaseKVCacheMethod
from sglang.srt.layers.quantization.quark.schemes import (
    QuarkLinearScheme,
    QuarkMoEScheme,
    QuarkW4A4MXFP4,
    QuarkW4A4MXFp4MoE,
    QuarkW8A8Fp8,
    QuarkW8A8FP8MoE,
)
from sglang.srt.layers.quantization.quark.utils import deep_compare, should_ignore_layer
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.utils import get_device_capability
```
**EN:** This block imports fnmatch, logging, torch, typing, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton.layer, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 fnmatch, logging, torch, typing, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton.layer, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 31-32: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import StandardDispatchOutput
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 34-34: initialize __all__
```python
__all__ = ["QuarkLinearMethod", "QuarkFusedMoEMethod"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 36-36: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 39-40: class QuarkConfig: definition
```python
class QuarkConfig(QuantizationConfig):
```
**EN:** This block declares `QuarkConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_linear_method, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `QuarkConfig`，它是量化栈中的配置类，组织了 __init__, get_linear_method, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 41-57: QuarkConfig.__init__()
```python
    def __init__(
        self,
        quant_config: dict[str, Any],
        kv_cache_group: Optional[list[str]] = None,
        kv_cache_config: Optional[dict[str, Any]] = None,
        pack_method: str = "reorder",
    ):
        super().__init__()
        if kv_cache_group is None:
            kv_cache_group = []
        self.quant_config = quant_config
        self.kv_cache_group = kv_cache_group
        self.kv_cache_config = kv_cache_config
        self.pack_method = pack_method
        self.exclude_layers = cast(list[str], self.quant_config.get("exclude", []))

        self.packed_modules_mapping = self.quant_config["packed_modules_mapping"]
```
**EN:** This block defines `QuarkConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 59-60: QuarkConfig.get_linear_method()
```python
    def get_linear_method(self) -> "QuarkLinearMethod":
        return QuarkLinearMethod(self)
```
**EN:** This block defines `QuarkConfig.get_linear_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_linear_method()`，用于为调用方获取或计算派生值。

### Lines 62-64: QuarkConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.float16, torch.bfloat16]
```
**EN:** This block defines `QuarkConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 66-68: QuarkConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines `QuarkConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 70-71: QuarkConfig.get_name()
```python
    def get_name(self) -> str:
        return "quark"
```
**EN:** This block defines `QuarkConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 73-80: QuarkConfig.apply_weight_name_mapper()
```python
    def apply_weight_name_mapper(self, hf_to_sglang_mapper):
        mapped = hf_to_sglang_mapper.apply_list(self.exclude_layers)
        expanded = []
        for name in mapped:
            expanded.append(name)
            if name.startswith("language_model."):
                expanded.append(name.removeprefix("language_model."))
        self.exclude_layers = list(dict.fromkeys(expanded))
```
**EN:** This block defines `QuarkConfig.apply_weight_name_mapper()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuarkConfig.apply_weight_name_mapper()`，用于将量化计算应用到运行时输入上。

### Lines 82-111: QuarkConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional["QuantizeMethodBase"]:
        # Check if the layer is skipped for quantization.
        if should_ignore_layer(
            prefix,
            ignore=self.exclude_layers,
            fused_mapping=self.packed_modules_mapping,
        ):
            if isinstance(layer, LinearBase):
                return UnquantizedLinearMethod()
            elif isinstance(layer, RadixAttention):
                return QuarkKVCacheMethod(self)
            return None

        if isinstance(layer, LinearBase):
            scheme = self.get_linear_scheme(layer=layer, layer_name=prefix)
            layer.scheme = scheme
            return QuarkLinearMethod(self)

        if isinstance(layer, RadixAttention):
            return QuarkKVCacheMethod(self)

        from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE

        if isinstance(layer, FusedMoE):
            layer.scheme = self.get_moe_scheme(layer, prefix)
            return QuarkFusedMoEMethod(self)

        return None
```
**EN:** This block defines `QuarkConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 113-178: QuarkConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> "QuarkConfig":
        export_config = config.get("export")
        if export_config is None:
            raise ValueError(
                "The export key should be included in "
                "the configurations of Quark quantized model"
            )

        kv_cache_group = cast(list[str], export_config.get("kv_cache_group"))
        pack_method = cast(str, export_config.get("pack_method"))

        # In the export model of quark, the quantization configuration
        # of kv_cache is stored in layer_quant_config. First, it is
        # judged whether kv_cache_group exists, and then it is judged
        # whether layer_quant_config has a quantization configuration
        # that matches kv_cache.
        if len(kv_cache_group) == 0:
            kv_cache_config = None
        else:
            kv_cache_set = set(kv_cache_group)
            layer_quant_config = cast(dict[str, Any], config.get("layer_quant_config"))
            layer_quant_names = list(layer_quant_config.keys())
            layer_quant_set = set(layer_quant_names)

            if not kv_cache_set.issubset(layer_quant_set):
                raise ValueError(
                    "The Quark quantized model has the "
                    "kv_cache_group parameter setting, "
                    "but no kv_cache quantization settings "
                    "were found in the quantization "
                    "configuration."
                )

            q_configs = [
                cast(dict[str, Any], layer_quant_config.get(name))
                for name in kv_cache_group
            ]
            if not all(deep_compare(q_config, q_configs[0]) for q_config in q_configs):
                raise ValueError(
                    "The quantization method used for kv_cache should "
                    "be the same, but the quantization method for the "
                    "kv_cache layer in the config is different."
                )
            kv_cache_config = q_configs[0].get("output_tensors")
            if kv_cache_config is None:
                raise ValueError("The kv_cache quantization configuration is empty.")

            # Since we have already set kv_cache quantization configurations,
            # we will remove the quantization configuration for the
            # output_tensors corresponding to the kv_cache layer.
            for q_config in q_configs:
                q_config["output_tensors"] = None

            # In case q_proj output is also quantized, remove the configuration
            # to keep qkv consistency.
            q_proj_q_config = cast(dict[str, Any], layer_quant_config.get("*q_proj"))
            if q_proj_q_config is not None:
                q_proj_q_config["output_tensors"] = None

        return cls(
            quant_config=config,
            kv_cache_group=kv_cache_group,
            kv_cache_config=kv_cache_config,
            pack_method=pack_method,
        )
```
**EN:** This block defines `QuarkConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `QuarkConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 180-182: QuarkConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return []
```
**EN:** This block defines `QuarkConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 184-200: QuarkConfig._check_scheme_supported()
```python
    def _check_scheme_supported(self, min_capability: int, error: bool = True) -> bool:
        capability_tuple = get_device_capability()

        if capability_tuple is not None:
            assert 0 <= capability_tuple[1] < 10
            capability = capability_tuple[0] * 10 + capability_tuple[1]

            supported = capability >= min_capability
            if error and not supported:
                raise RuntimeError(
                    "Quantization scheme is not supported for ",
                    f"the current GPU. Min capability: {min_capability}. ",
                    f"Current capability: {capability}.",
                )
            return supported
        else:
            return False
```
**EN:** This block defines `QuarkConfig._check_scheme_supported()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `QuarkConfig._check_scheme_supported()`，用于实现量化栈中的可复用模块逻辑。

### Lines 202-231: QuarkConfig._is_fp8_w8a8()
```python
    def _is_fp8_w8a8(
        self,
        weight_quant: Optional[dict[str, Any]],
        input_quant: Optional[dict[str, Any]],
    ) -> bool:
        # Confirm weights and input quantized.
        if weight_quant is None or input_quant is None:
            return False

        # Confirm weight scheme is supported
        is_fp8_dtype = (
            weight_quant.get("dtype") == "fp8_e4m3"
            and input_quant.get("dtype") == "fp8_e4m3"
        )
        is_static_weight = not weight_quant.get("is_dynamic")
        is_per_tensor_or_channel_weight = weight_quant.get("qscheme") in [
            "per_tensor",
            "per_channel",
        ]

        if not (is_fp8_dtype and is_static_weight and is_per_tensor_or_channel_weight):
            return False

        # Dynamic quantization is always supported if weights supported.
        if input_quant.get("is_dynamic"):
            return True

        # Confirm activation scheme is supported.
        is_per_tensor_activation = input_quant.get("qscheme") == "per_tensor"
        return is_per_tensor_activation
```
**EN:** This block defines `QuarkConfig._is_fp8_w8a8()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `QuarkConfig._is_fp8_w8a8()`，用于实现量化栈中的可复用模块逻辑。

### Lines 233-282: QuarkConfig._is_mx_fp4()
```python
    def _is_mx_fp4(
        self,
        weight_quant: Optional[dict[str, Any]],
        input_quant: Optional[dict[str, Any]],
    ) -> bool:
        # Confirm weights and input quantized.
        if weight_quant is None or input_quant is None:
            logger.debug(
                "Quark model is not in MX-FP4 format: "
                "weight_quant or input_quant not set"
            )
            return False

        # Input and weight dtype needs to be fp4.
        if weight_quant.get("dtype") != "fp4" or input_quant.get("dtype") != "fp4":
            logger.debug("Quark model is not in MX-FP4 format: dtype not fp4")
            return False

        # Input and weight qscheme needs to be per group.
        if (
            weight_quant.get("qscheme") != "per_group"
            or input_quant.get("qscheme") != "per_group"
        ):
            logger.debug("Quark model is not in MX-FP4 format: not per_group")
            return False

        # Input and weight group size needs to be 32.
        if weight_quant.get("group_size") != 32 or input_quant.get("group_size") != 32:
            logger.debug("Quark model is not in MX-FP4 format: not group_size=32")
            return False

        # Weights need to use static quantization.
        if weight_quant.get("is_dynamic") is True:
            logger.debug("Quark model is not in MX-FP4 format: not weight static")
            return False

        # Activations need to use dynamic quantization.
        if input_quant.get("is_dynamic") is False:
            logger.debug("Quark model is not in MX-FP4 format: not activation dynamic")
            return False

        # Activations and weight scales need to be in e8m0 format.
        if (
            weight_quant.get("scale_format") != "e8m0"
            or input_quant.get("scale_format") != "e8m0"
        ):
            logger.debug("Quark model is not in MX-FP4 format: not scale_format e8m0")
            return False

        return True
```
**EN:** This block defines `QuarkConfig._is_mx_fp4()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `QuarkConfig._is_mx_fp4()`，用于实现量化栈中的可复用模块逻辑。

### Lines 284-328: QuarkConfig._find_matched_config()
```python
    def _find_matched_config(
        self, layer_name: str, module: torch.nn.Module
    ) -> dict[str, Any]:

        proj_name = layer_name.split(".")[-1]
        if proj_name in self.packed_modules_mapping:
            shard_proj_names = self.packed_modules_mapping[proj_name]

            # Convert fused_name --> [shard_names]
            shard_names = [
                layer_name.replace(proj_name, shard_proj_name)
                for shard_proj_name in shard_proj_names
            ]
            shard_configs = [
                self._find_matched_config(shard_name, module)
                for shard_name in shard_names
            ]
            if not all(
                deep_compare(q_config, shard_configs[0]) for q_config in shard_configs
            ):
                raise ValueError(
                    f"Found a different quantization configuration for "
                    f"{shard_proj_names} in {layer_name}. vLLM "
                    "requires all to use the same scheme."
                )
            return shard_configs[0]
        else:
            layer_quant_config = cast(
                dict[str, Any], self.quant_config.get("layer_quant_config")
            )
            for name_pattern in layer_quant_config:
                if fnmatch.fnmatch(layer_name, name_pattern):
                    return layer_quant_config[name_pattern]

            layer_type = type(module).__name__
            layer_type_quant_config = cast(
                dict[str, Any], self.quant_config.get("layer_type_quant_config")
            )
            if layer_type in layer_type_quant_config:
                return layer_type_quant_config[layer_type]

            global_quant_config = cast(
                dict[str, Any], self.quant_config.get("global_quant_config")
            )
            return global_quant_config
```
**EN:** This block defines `QuarkConfig._find_matched_config()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `QuarkConfig._find_matched_config()`，用于实现量化栈中的可复用模块逻辑。

### Lines 330-352: QuarkConfig._get_scheme_from_config()
```python
    def _get_scheme_from_config(self, config: dict[str, Any]) -> "QuarkLinearScheme":
        if config.get("output_tensors") or config.get("bias"):
            raise NotImplementedError(
                "Currently, Quark models with output_tensors "
                "and bias quantized are not supported"
            )
        weight_config = cast(dict[str, Any], config.get("weight"))
        input_config = cast(dict[str, Any], config.get("input_tensors"))

        if self._is_mx_fp4(weight_config, input_config):
            return QuarkW4A4MXFP4(weight_config, input_config)
        if self._is_fp8_w8a8(weight_config, input_config):
            is_fp8_w8a8_supported = self._check_scheme_supported(
                QuarkW8A8Fp8.get_min_capability(), error=False
            )
            if is_fp8_w8a8_supported:
                return QuarkW8A8Fp8(weight_config, input_config)

        raise NotImplementedError(
            "No quark compatible scheme was found. "
            f"Weight config: {weight_config}, "
            f"Input config: {input_config}"
        )
```
**EN:** This block defines `QuarkConfig._get_scheme_from_config()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `QuarkConfig._get_scheme_from_config()`，用于实现量化栈中的可复用模块逻辑。

### Lines 354-367: QuarkConfig.get_linear_scheme()
```python
    def get_linear_scheme(
        self, layer: torch.nn.Module, layer_name: str
    ) -> "QuarkLinearScheme":

        layer_quant_config = self._find_matched_config(layer_name, layer)

        # Find the quant_scheme
        scheme = self._get_scheme_from_config(layer_quant_config)

        # Raise error if device does not support the scheme
        # (e.g. fp8 needs ada lovelace)
        self._check_scheme_supported(scheme.get_min_capability())

        return scheme
```
**EN:** This block defines `QuarkConfig.get_linear_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_linear_scheme()`，用于为调用方获取或计算派生值。

### Lines 369-390: QuarkConfig.get_moe_scheme()
```python
    def get_moe_scheme(
        self,
        module: torch.nn.Module,
        layer_name: str,
    ) -> "QuarkMoEScheme":
        layer_quant_config = self._find_matched_config(layer_name, module)

        if layer_quant_config.get("output_tensors") or layer_quant_config.get("bias"):
            raise NotImplementedError(
                "Currently, Quark models with "
                "output_tensors and bias "
                "quantized are not supported"
            )
        weight_config = layer_quant_config.get("weight")
        input_config = layer_quant_config.get("input_tensors")

        if self._is_mx_fp4(weight_config, input_config):
            return QuarkW4A4MXFp4MoE(weight_config, input_config)
        elif self._is_fp8_w8a8(weight_config, input_config):
            return QuarkW8A8FP8MoE(weight_config, input_config)
        else:
            raise RuntimeError("Unsupported FusedMoe scheme")
```
**EN:** This block defines `QuarkConfig.get_moe_scheme()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_moe_scheme()`，用于为调用方获取或计算派生值。

### Lines 392-393: QuarkConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `QuarkConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuarkConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 396-397: class QuarkLinearMethod: definition
```python
class QuarkLinearMethod(LinearMethodBase):
```
**EN:** This block declares `QuarkLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, create_weights, apply.
**CN:** 该代码块声明 `QuarkLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, process_weights_after_loading, create_weights, apply 等行为。

### Lines 398-399: QuarkLinearMethod.__init__()
```python
    def __init__(self, quantization_config: QuarkConfig):
        self.quantization_config = quantization_config
```
**EN:** This block defines `QuarkLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 401-402: QuarkLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** This block defines `QuarkLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QuarkLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 404-428: QuarkLinearMethod.create_weights()
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
        Use the QuarkLinearScheme associated with the layer to create
        the necessary parameters for the layer. See LinearMethodBase for param
        details
        """
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.scheme.create_weights(
            layer=layer,
            input_size=input_size,
            input_size_per_partition=input_size_per_partition,
            output_partition_sizes=output_partition_sizes,
            output_size=output_size,
            params_dtype=params_dtype,
            weight_loader=weight_loader,
        )
```
**EN:** This block defines `QuarkLinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkLinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 430-445: QuarkLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ):
        """
        Use the output of create_weights and the QuarkLinearScheme
        associated with the layer to apply the forward pass with the
        layer input.  See LinearMethodBase for param details

        """
        scheme = layer.scheme
        if scheme is None:
            raise ValueError("A scheme must be defined for each layer")
        return scheme.apply_weights(layer, x, bias=bias)
```
**EN:** This block defines `QuarkLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuarkLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 448-449: class QuarkFusedMoEMethod: definition
```python
class QuarkFusedMoEMethod(FusedMoEMethodBase):
```
**EN:** This block declares `QuarkFusedMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, create_weights, create_moe_runner.
**CN:** 该代码块声明 `QuarkFusedMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, process_weights_after_loading, create_weights, create_moe_runner 等行为。

### Lines 450-451: QuarkFusedMoEMethod.__init__()
```python
    def __init__(self, quantization_config: QuarkConfig):
        self.quantization_config = quantization_config
```
**EN:** This block defines `QuarkFusedMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkFusedMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 453-454: QuarkFusedMoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** This block defines `QuarkFusedMoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QuarkFusedMoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 456-477: QuarkFusedMoEMethod.create_weights()
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
        """
        Use the QuarkMoEScheme associated with the layer to create
        the necessary parameters for the layer. See FusedMoEMethodBase for param
        details
        """
        layer.scheme.create_weights(
            layer=layer,
            num_experts=num_experts,
            hidden_size=hidden_size,
            intermediate_size_per_partition=intermediate_size_per_partition,
            params_dtype=params_dtype,
            **extra_weight_attrs,
        )
```
**EN:** This block defines `QuarkFusedMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkFusedMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 479-482: QuarkFusedMoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        layer.scheme.create_moe_runner(layer, moe_runner_config)
```
**EN:** This block defines `QuarkFusedMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuarkFusedMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 484-498: QuarkFusedMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ):
        """
        Use the output of create_weights and the QuarkMoEScheme
        associated with the layer to apply the forward pass with the
        fused MoE layer. See FusedMoEMethodBase for param details

        """
        scheme = layer.scheme
        if scheme is None:
            raise ValueError("A scheme must be defined for each layer")
        return scheme.apply_weights(layer, dispatch_output)
```
**EN:** This block defines `QuarkFusedMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuarkFusedMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 501-505: class QuarkKVCacheMethod: definition
```python
class QuarkKVCacheMethod(BaseKVCacheMethod):
    """
    Supports loading kv-cache scaling factors from quark checkpoints.
    """
```
**EN:** This block declares `QuarkKVCacheMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, validate_kv_cache_config.
**CN:** 该代码块声明 `QuarkKVCacheMethod`，它是量化栈中的运行方法类，组织了 __init__, validate_kv_cache_config 等行为。

### Lines 506-508: QuarkKVCacheMethod.__init__()
```python
    def __init__(self, quant_config: QuarkConfig):
        self.validate_kv_cache_config(quant_config.kv_cache_config)
        super().__init__(quant_config)
```
**EN:** This block defines `QuarkKVCacheMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuarkKVCacheMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 510-533: QuarkKVCacheMethod.validate_kv_cache_config()
```python
    @staticmethod
    def validate_kv_cache_config(kv_cache_config: Optional[dict[str, Any]]):
        """
        Validator for the kv cache configuration. Useful for controlling the
        kv cache quantization schemes, that are being supported in vLLM
        :param kv_cache_config: the quark kv cache scheme
        """
        if kv_cache_config is None:
            return

        dtype = kv_cache_config.get("dtype")
        if dtype != "fp8_e4m3":
            raise NotImplementedError(
                "Currently supported kv cache quantization is "
                f"dtype=fp8_e4m3, however received {dtype}"
            )

        qscheme = kv_cache_config.get("qscheme")
        if qscheme != "per_tensor":
            raise NotImplementedError(
                "Only support per-tensor scaling factor "
                "for quark KV cache. "
                f"Expected qscheme: per_tensor, found qscheme: {qscheme}"
            )
```
**EN:** This block defines `QuarkKVCacheMethod.validate_kv_cache_config()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `QuarkKVCacheMethod.validate_kv_cache_config()`，用于实现量化栈中的可复用模块逻辑。

## Key Concepts / 关键概念
- `QuarkConfig`: A configuration class that structures file-level quantization behavior. / `QuarkConfig` 是一个配置类，用于组织该文件中的量化行为。
- `QuarkLinearMethod`: A runtime method class that structures file-level quantization behavior. / `QuarkLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `QuarkFusedMoEMethod`: A runtime method class that structures file-level quantization behavior. / `QuarkFusedMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `QuarkKVCacheMethod`: A runtime method class that structures file-level quantization behavior. / `QuarkKVCacheMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `fnmatch`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton.layer`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.kv_cache`, `sglang.srt.layers.quantization.quark.schemes`, `sglang.srt.layers.quantization.quark.utils`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.radix_attention`, `sglang.srt.utils`
