# compressed_tensors.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/compressed_tensors.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for compressed tensors quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 压缩 张量 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-69: module imports and setup
```python
# Adapted from https://github.com/vllm-project/vllm/tree/main/vllm/model_executor/layers/quantization/compressed_tensors
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from __future__ import annotations

import logging
from contextlib import suppress
from typing import (
    TYPE_CHECKING,
    Any,
    Dict,
    List,
    Literal,
    NamedTuple,
    Optional,
    Tuple,
    cast,
)

import torch
from compressed_tensors.config import (
    CompressionFormat,
    SparsityCompressionConfig,
    SparsityStructure,
)
from compressed_tensors.quantization import (
    QuantizationArgs,
    QuantizationStrategy,
    QuantizationType,
)
from pydantic import BaseModel

from sglang.srt.layers.moe import MoeRunnerConfig, get_moe_runner_backend
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.compressed_tensors.schemes import (
    WNA16_SUPPORTED_BITS,
    CompressedTensorsLinearScheme,
    CompressedTensorsMoEScheme,
    CompressedTensorsMxInt4MoE,
    CompressedTensorsW4A4Fp4,
    CompressedTensorsW4A4Nvfp4MoE,
    CompressedTensorsW8A8Fp8,
    CompressedTensorsW8A8Fp8MoE,
    CompressedTensorsW8A8Int8,
    CompressedTensorsW8A16Fp8,
    CompressedTensorsWNA16,
    CompressedTensorsWNA16MoE,
    CompressedTensorsWNA16TritonMoE,
    NPUCompressedTensorsW4A8Int8DynamicMoE,
    NPUCompressedTensorsW4A16Int4DynamicMoE,
    NPUCompressedTensorsW8A8Int8,
    NPUCompressedTensorsW8A8Int8DynamicMoE,
)
from sglang.srt.layers.quantization.compressed_tensors.utils import (
    find_matched_target,
    is_activation_quantization_format,
    should_ignore_layer,
)
from sglang.srt.layers.quantization.fp8 import Fp8LinearMethod
from sglang.srt.layers.quantization.unquant import (
    UnquantizedFusedMoEMethod,
    UnquantizedLinearMethod,
)
from sglang.srt.utils import is_cuda, is_hip, is_npu
```
**EN:** This block imports __future__, compressed_tensors.config, compressed_tensors.quantization, contextlib, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, compressed_tensors.config, compressed_tensors.quantization, contextlib, sglang.srt.layers.linear, sglang.srt.layers.moe, sglang.srt.layers.moe.fused_moe_triton, sglang.srt.layers.moe.token_dispatcher 等依赖，并为当前量化实现准备模块命名空间。

### Lines 71-71: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 72-72: initialize _is_npu
```python
_is_npu = is_npu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_npu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_npu。

### Lines 73-73: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 75-80: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
    from sglang.srt.models.utils import WeightsMapper
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 82-82: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 84-84: initialize __all__
```python
__all__ = ["CompressedTensorsLinearMethod"]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as __all__.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 __all__。

### Lines 86-86: annotate SPARSITY_CONFIG_NAME
```python
SPARSITY_CONFIG_NAME: Literal["sparsity_config"] = "sparsity_config"
```
**EN:** This block declares and initializes the annotated symbol `SPARSITY_CONFIG_NAME`.
**CN:** 该代码块声明并初始化带类型注解的符号 `SPARSITY_CONFIG_NAME`。

### Lines 87-87: initialize QUANTIZATION_SCHEME_MAP_TYPE
```python
QUANTIZATION_SCHEME_MAP_TYPE = Dict[str, Optional[Dict[str, QuantizationArgs]]]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as QUANTIZATION_SCHEME_MAP_TYPE.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 QUANTIZATION_SCHEME_MAP_TYPE。

### Lines 90-90: class DeviceCapability: definition
```python
class DeviceCapability(NamedTuple):
```
**EN:** This block declares `DeviceCapability`, a supporting class for the quantization stack. It organizes behaviors such as as_version_str, to_int.
**CN:** 该代码块声明 `DeviceCapability`，它是量化栈中的支撑类，组织了 as_version_str, to_int 等行为。

### Lines 91-91: DeviceCapability member: annotate major
```python
    major: int
```
**EN:** This block declares and initializes the annotated symbol `major`.
**CN:** 该代码块声明并初始化带类型注解的符号 `major`。

### Lines 92-92: DeviceCapability member: annotate minor
```python
    minor: int
```
**EN:** This block declares and initializes the annotated symbol `minor`.
**CN:** 该代码块声明并初始化带类型注解的符号 `minor`。

### Lines 94-95: DeviceCapability.as_version_str()
```python
    def as_version_str(self) -> str:
        return f"{self.major}.{self.minor}"
```
**EN:** This block defines `DeviceCapability.as_version_str()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `DeviceCapability.as_version_str()`，用于实现量化栈中的可复用模块逻辑。

### Lines 97-104: DeviceCapability.to_int()
```python
    def to_int(self) -> int:
        """
        Express device capability as an integer ``<major><minor>``.

        It is assumed that the minor version is always a single digit.
        """
        assert 0 <= self.minor < 10
        return self.major * 10 + self.minor
```
**EN:** This block defines `DeviceCapability.to_int()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `DeviceCapability.to_int()`，用于实现量化栈中的可复用模块逻辑。

### Lines 107-107: class CompressedTensorsConfig: definition
```python
class CompressedTensorsConfig(QuantizationConfig):
```
**EN:** This block declares `CompressedTensorsConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_linear_method, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `CompressedTensorsConfig`，它是量化栈中的配置类，组织了 __init__, get_linear_method, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 108-130: CompressedTensorsConfig.__init__()
```python
    def __init__(
        self,
        target_scheme_map: Dict[str, Any],
        ignore: List[str],
        quant_format: str,
        sparsity_scheme_map: Dict[str, SparsityCompressionConfig],
        sparsity_ignore_list: List[str],
        kv_cache_scheme: Optional[Dict[str, Any]] = None,
        config: Optional[Dict[str, Any]] = None,
        packed_modules_mapping: Optional[Dict[str, List[str]]] = None,
        linear_fp8_config: Optional[Any] = None,
    ):
        super().__init__()
        self.ignore = ignore
        self.quant_format = quant_format
        # Map from [target -> scheme]
        self.target_scheme_map = target_scheme_map
        self.kv_cache_scheme = kv_cache_scheme
        self.sparsity_scheme_map = sparsity_scheme_map
        self.sparsity_ignore_list = sparsity_ignore_list
        self.config = config
        self.packed_modules_mapping = packed_modules_mapping or {}
        self.linear_fp8_config = linear_fp8_config
```
**EN:** This block defines `CompressedTensorsConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 132-133: CompressedTensorsConfig.get_linear_method()
```python
    def get_linear_method(self) -> CompressedTensorsLinearMethod:
        return CompressedTensorsLinearMethod(self)
```
**EN:** This block defines `CompressedTensorsConfig.get_linear_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_linear_method()`，用于为调用方获取或计算派生值。

### Lines 135-136: CompressedTensorsConfig.get_supported_act_dtypes()
```python
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.float16, torch.bfloat16]
```
**EN:** This block defines `CompressedTensorsConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 138-140: CompressedTensorsConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines `CompressedTensorsConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 142-143: CompressedTensorsConfig.get_name()
```python
    def get_name(self) -> str:
        return "compressed_tensors"
```
**EN:** This block defines `CompressedTensorsConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 145-146: CompressedTensorsConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `CompressedTensorsConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 148-158: CompressedTensorsConfig.apply_weight_name_mapper()
```python
    def apply_weight_name_mapper(self, hf_to_sglang_mapper: "WeightsMapper"):
        self.target_scheme_map = hf_to_sglang_mapper.apply_dict(self.target_scheme_map)
        self.ignore = hf_to_sglang_mapper.apply_list(self.ignore)
        self.sparsity_scheme_map = hf_to_sglang_mapper.apply_dict(
            self.sparsity_scheme_map
        )
        self.sparsity_ignore_list = hf_to_sglang_mapper.apply_list(
            self.sparsity_ignore_list
        )
        if self.kv_cache_scheme is not None:
            self.kv_cache_scheme = hf_to_sglang_mapper.apply_dict(self.kv_cache_scheme)
```
**EN:** This block defines `CompressedTensorsConfig.apply_weight_name_mapper()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsConfig.apply_weight_name_mapper()`，用于将量化计算应用到运行时输入上。

### Lines 160-191: CompressedTensorsConfig.get_quant_method()
```python
    def get_quant_method(
        self,
        layer: torch.nn.Module,
        prefix: str,
    ) -> Optional[QuantizeMethodBase]:
        from sglang.srt.layers.linear import LinearBase

        if isinstance(layer, LinearBase):
            # If linear_fp8_config is set, use FP8 for linear layers
            # This allows mixed quantization: experts with int4, linear layers with fp8
            if self.linear_fp8_config is not None:
                return Fp8LinearMethod(self.linear_fp8_config)
            scheme = self.get_linear_scheme(layer=layer, layer_name=prefix)
            if scheme is None:
                return UnquantizedLinearMethod()
            layer.scheme = scheme
            return CompressedTensorsLinearMethod(self)
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, FusedMoE):
            layer.scheme = self.get_moe_scheme(layer=layer, layer_name=prefix)
            if layer.scheme is None:  # ignored layer
                use_triton_kernels = get_moe_runner_backend().is_triton_kernels()
                use_flashinfer_trtllm_moe = (
                    get_moe_runner_backend().is_flashinfer_trtllm()
                )
                use_deep_gemm = get_moe_runner_backend().is_deep_gemm()
                return UnquantizedFusedMoEMethod(
                    use_triton_kernels, use_flashinfer_trtllm_moe, use_deep_gemm
                )
            return CompressedTensorsFusedMoEMethod(self)
        return None
```
**EN:** This block defines `CompressedTensorsConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 193-206: CompressedTensorsConfig._add_fused_moe_to_target_scheme_map()
```python
    def _add_fused_moe_to_target_scheme_map(self):
        """
        Helper function to update target_scheme_map
        since linear layers get fused into FusedMoE
        targeting 'Linear' needs to also match
        FusedMoE modules.
        """
        if (
            "Linear" not in self.target_scheme_map
            or "FusedMoE" in self.target_scheme_map
        ):
            return
        self.target_scheme_map["FusedMoE"] = self.target_scheme_map["Linear"]
        self.target_scheme_map["DeepEPMoE"] = self.target_scheme_map["Linear"]
```
**EN:** This block defines `CompressedTensorsConfig._add_fused_moe_to_target_scheme_map()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._add_fused_moe_to_target_scheme_map()`，用于实现量化栈中的可复用模块逻辑。

### Lines 208-215: CompressedTensorsConfig.weight_block_size()
```python
    @property
    def weight_block_size(self) -> Optional[List[int]]:
        """Get the weight block size from the quantization config."""
        if "Linear" in self.target_scheme_map:
            weights_config = self.target_scheme_map["Linear"].get("weights")
            if weights_config and hasattr(weights_config, "block_structure"):
                return weights_config.block_structure
        return None
```
**EN:** This block defines `CompressedTensorsConfig.weight_block_size()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig.weight_block_size()`，用于实现量化栈中的可复用模块逻辑。

### Lines 217-253: CompressedTensorsConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> CompressedTensorsConfig:
        ignore: List[str] = cast(List[str], config.get("ignore", []))
        quant_format = cast(str, config.get("format"))
        target_scheme_map = cls._quantization_scheme_map_from_config(config=config)
        sparsity_scheme_map, sparsity_ignore_list = cls._parse_sparsity_config(
            config=config
        )
        packed_modules_mapping = config.get("packed_modules_mapping", {})

        # Parse linear_fp8_config if present (for mixed quantization scenarios)
        # Format: {"activation_scheme": "dynamic", "fmt": "e4m3",
        #          "quant_method": "fp8", "weight_block_size": [128, 128]}
        linear_fp8_config = None
        if "linear_fp8_config" in config:
            from sglang.srt.layers.quantization.fp8 import Fp8Config

            fp8_cfg = config["linear_fp8_config"]
            # Check if it's fp8 format based on quant_method field
            is_fp8 = fp8_cfg.get("quant_method") == "fp8"
            linear_fp8_config = Fp8Config(
                is_checkpoint_fp8_serialized=is_fp8,
                activation_scheme=fp8_cfg.get("activation_scheme", "dynamic"),
                ignored_layers=fp8_cfg.get("ignored_layers"),
                weight_block_size=fp8_cfg.get("weight_block_size"),
            )

        return cls(
            target_scheme_map=target_scheme_map,
            ignore=ignore,
            quant_format=quant_format,
            sparsity_scheme_map=sparsity_scheme_map,
            sparsity_ignore_list=sparsity_ignore_list,
            config=config,
            packed_modules_mapping=packed_modules_mapping,
            linear_fp8_config=linear_fp8_config,
        )
```
**EN:** This block defines `CompressedTensorsConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `CompressedTensorsConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 255-274: CompressedTensorsConfig._parse_sparsity_config()
```python
    @classmethod
    def _parse_sparsity_config(
        cls, config: Dict[str, Any]
    ) -> Tuple[Dict[str, SparsityCompressionConfig], List[str]]:
        """
        :param config: The `quantization_config` dictionary from config.json
        :return: A tuple with two elements
            1. A dictionary mapping target layer names to their corresponding
                sparsity_config
            2. A list of layer names to ignore for sparsity
        """
        if not (sparsity_config := config.get(SPARSITY_CONFIG_NAME)):
            return dict(), []

        sparsity_config = SparsityCompressionConfig.model_validate(sparsity_config)
        sparse_scheme_map: Dict[str, SparsityCompressionConfig] = {
            target: sparsity_config for target in sparsity_config.targets or list()
        }
        sparsity_ignore_list = sparsity_config.ignore or list()
        return sparse_scheme_map, sparsity_ignore_list
```
**EN:** This block defines `CompressedTensorsConfig._parse_sparsity_config()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._parse_sparsity_config()`，用于实现量化栈中的可复用模块逻辑。

### Lines 276-324: CompressedTensorsConfig._quantization_scheme_map_from_config()
```python
    @classmethod
    def _quantization_scheme_map_from_config(
        cls, config: Dict[str, Any]
    ) -> QUANTIZATION_SCHEME_MAP_TYPE:
        """
        :param config: The `quantization_config` dictionary from config.json
        :return: A dictionary mapping target layer names to their corresponding
            quantization_args for weights and input activations
        """
        target_scheme_map: Dict[str, Any] = dict()
        quant_format = cast(str, config.get("format"))

        # The quant_config has multiple config_groups, each containing
        # an input_activations key with details about how the activations are
        # quantized, a weights key indicating how the weights are quantized,
        # and a list of targets under the `targets` key, dictating which
        # layers are impacted by the quantization details. The quantization
        # details follow the structure defined by the QuantizationArgs
        # pydantic model, which is used to verify the structure of the
        # quant_config and also store the details for later use.

        config_groups = config.get("config_groups", dict())
        for _, quant_config in config_groups.items():
            targets = quant_config.get("targets")
            for target in targets:
                target_scheme_map[target] = {}
                target_scheme_map[target]["weights"] = QuantizationArgs.model_validate(
                    quant_config.get("weights")
                )

                target_scheme_map[target]["input_activations"] = None
                if is_activation_quantization_format(quant_format):
                    input_activations = quant_config.get("input_activations")
                    # The only case where we have activation quant supported
                    # but no input_activations provided in the config
                    # should be w8a16fp8 w8a16fp8 can also run for cases where
                    # there is an input_quant but it is ignored
                    if not input_activations:
                        assert (
                            target_scheme_map[target]["weights"].type
                            == QuantizationType.FLOAT
                        )
                    else:
                        target_scheme_map[target]["input_activations"] = (
                            QuantizationArgs.model_validate(  # noqa: E501
                                quant_config.get("input_activations")
                            )
                        )
        return target_scheme_map
```
**EN:** This block defines `CompressedTensorsConfig._quantization_scheme_map_from_config()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `CompressedTensorsConfig._quantization_scheme_map_from_config()`，用于处理张量或权重量化逻辑。

### Lines 326-328: CompressedTensorsConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines `CompressedTensorsConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 330-344: CompressedTensorsConfig._check_scheme_supported()
```python
    def _check_scheme_supported(self, min_capability: int, error: bool = True) -> bool:
        capability_tuple = DeviceCapability(*torch.cuda.get_device_capability())

        if capability_tuple is not None:
            capability = capability_tuple.to_int()
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
**EN:** This block defines `CompressedTensorsConfig._check_scheme_supported()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._check_scheme_supported()`，用于实现量化栈中的可复用模块逻辑。

### Lines 346-366: CompressedTensorsConfig._is_dynamic_token_w4a8()
```python
    def _is_dynamic_token_w4a8(
        self, weight_quant: BaseModel, input_quant: BaseModel
    ) -> bool:
        is_weight_4_bits = weight_quant.num_bits == 4
        is_activation_8_bits = input_quant.num_bits == 8
        weight_strategy = (
            weight_quant.strategy == QuantizationStrategy.GROUP.value
            or weight_quant.strategy == QuantizationStrategy.CHANNEL.value
        )
        is_token = (
            weight_strategy and input_quant.strategy == QuantizationStrategy.TOKEN.value
        )
        is_dynamic = not weight_quant.dynamic and input_quant.dynamic

        return (
            is_weight_4_bits
            and is_activation_8_bits
            and is_token
            and weight_quant.symmetric
            and is_dynamic
        )
```
**EN:** This block defines `CompressedTensorsConfig._is_dynamic_token_w4a8()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_dynamic_token_w4a8()`，用于实现量化栈中的可复用模块逻辑。

### Lines 368-384: CompressedTensorsConfig._is_static_tensor_w8a8()
```python
    def _is_static_tensor_w8a8(
        self, weight_quant: BaseModel, input_quant: BaseModel
    ) -> bool:
        is_8_bits = weight_quant.num_bits == input_quant.num_bits == 8
        weight_strategy = (
            weight_quant.strategy == QuantizationStrategy.TENSOR.value
            or weight_quant.strategy == QuantizationStrategy.CHANNEL.value
        )
        is_tensor = (
            weight_strategy
            and input_quant.strategy == QuantizationStrategy.TENSOR.value
        )
        is_static = not weight_quant.dynamic and not input_quant.dynamic

        # Both symmetric and asymmetric input quantization supported.
        # Only symmetric weight quantization supported.
        return is_8_bits and is_tensor and weight_quant.symmetric and is_static
```
**EN:** This block defines `CompressedTensorsConfig._is_static_tensor_w8a8()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_static_tensor_w8a8()`，用于实现量化栈中的可复用模块逻辑。

### Lines 386-401: CompressedTensorsConfig._is_dynamic_token_w8a8()
```python
    def _is_dynamic_token_w8a8(
        self, weight_quant: BaseModel, input_quant: BaseModel
    ) -> bool:
        is_8_bits = weight_quant.num_bits == input_quant.num_bits == 8
        weight_strategy = (
            weight_quant.strategy == QuantizationStrategy.TENSOR.value
            or weight_quant.strategy == QuantizationStrategy.CHANNEL.value
        )
        is_token = (
            weight_strategy and input_quant.strategy == QuantizationStrategy.TOKEN.value
        )
        is_dynamic = not weight_quant.dynamic and input_quant.dynamic

        # Both symmetric and asymmetric input quantization supported.
        # Only symmetric weight quantization supported.
        return is_8_bits and is_token and weight_quant.symmetric and is_dynamic
```
**EN:** This block defines `CompressedTensorsConfig._is_dynamic_token_w8a8()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_dynamic_token_w8a8()`，用于实现量化栈中的可复用模块逻辑。

### Lines 403-437: CompressedTensorsConfig._is_fp8_w8a8()
```python
    def _is_fp8_w8a8(
        self, weight_quant: QuantizationArgs, input_quant: QuantizationArgs
    ) -> bool:
        # Confirm weights and activations quantized.
        if weight_quant is None or input_quant is None:
            return False

        # Confirm weight scheme is supported.
        is_floating_point = (
            weight_quant.type == QuantizationType.FLOAT
            and input_quant.type == QuantizationType.FLOAT
        )
        is_symmetric_weight = weight_quant.symmetric
        is_static_weight = not weight_quant.dynamic
        is_tensor_or_channel_or_block_weight = weight_quant.strategy in [
            QuantizationStrategy.TENSOR,
            QuantizationStrategy.CHANNEL,
            QuantizationStrategy.BLOCK,
        ]
        if not (
            is_floating_point
            and is_symmetric_weight
            and is_static_weight
            and is_tensor_or_channel_or_block_weight
        ):
            return False

        # Dynamic quantization is always supported if weights supported.
        if input_quant.dynamic:
            return True

        # Confirm activation scheme is supported.
        is_symmetric_activation = input_quant.symmetric
        is_per_tensor_activation = input_quant.strategy == QuantizationStrategy.TENSOR
        return is_symmetric_activation and is_per_tensor_activation
```
**EN:** This block defines `CompressedTensorsConfig._is_fp8_w8a8()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_fp8_w8a8()`，用于实现量化栈中的可复用模块逻辑。

### Lines 439-463: CompressedTensorsConfig._is_fp8_w8a16()
```python
    def _is_fp8_w8a16(self, weight_quant: BaseModel, input_quant: BaseModel) -> bool:
        # Confirm weights quantized.
        if weight_quant is None:
            return False

        # Confirm we have floating points.
        if weight_quant.type != QuantizationType.FLOAT:
            return False

        # Confirm weight scheme is supported.
        is_symmetric_weight = weight_quant.symmetric
        is_static_weight = not weight_quant.dynamic
        is_per_tensor_or_channel_weight = weight_quant.strategy in [
            QuantizationStrategy.TENSOR,
            QuantizationStrategy.CHANNEL,
        ]
        if not (
            is_symmetric_weight
            and is_static_weight  # noqa: SIM103
            and is_per_tensor_or_channel_weight
        ):
            return False

        # All conditions satisfied.
        return True
```
**EN:** This block defines `CompressedTensorsConfig._is_fp8_w8a16()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_fp8_w8a16()`，用于实现量化栈中的可复用模块逻辑。

### Lines 465-492: CompressedTensorsConfig._is_fp4a4_nvfp4()
```python
    def _is_fp4a4_nvfp4(
        self, weight_quant: QuantizationArgs, input_quant: QuantizationArgs
    ):
        if weight_quant is None or input_quant is None:
            return False

        is_tensor_group_quant = (
            weight_quant.strategy == QuantizationStrategy.TENSOR_GROUP.value
            and input_quant.strategy == QuantizationStrategy.TENSOR_GROUP.value
        )
        is_symmetric = weight_quant.symmetric and input_quant.symmetric

        is_group_size_16 = (
            weight_quant.group_size == 16 and input_quant.group_size == 16
        )
        is_float_type = (
            weight_quant.type == QuantizationType.FLOAT
            and input_quant.type == QuantizationType.FLOAT
        )
        is_4_bits = weight_quant.num_bits == 4 and input_quant.num_bits == 4

        return (
            is_tensor_group_quant
            and is_float_type
            and is_4_bits
            and is_group_size_16
            and is_symmetric
        )
```
**EN:** This block defines `CompressedTensorsConfig._is_fp4a4_nvfp4()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_fp4a4_nvfp4()`，用于实现量化栈中的可复用模块逻辑。

### Lines 494-505: CompressedTensorsConfig._is_wNa16_group_channel()
```python
    def _is_wNa16_group_channel(
        self, weight_quant: BaseModel, input_quant: BaseModel
    ) -> bool:
        input_quant_none = input_quant is None
        is_symmetric = weight_quant.symmetric
        is_channel_group = (
            weight_quant.strategy == QuantizationStrategy.CHANNEL.value
            or weight_quant.strategy == QuantizationStrategy.GROUP.value
        )
        is_static = not weight_quant.dynamic

        return is_channel_group and input_quant_none and is_symmetric and is_static
```
**EN:** This block defines `CompressedTensorsConfig._is_wNa16_group_channel()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_wNa16_group_channel()`，用于实现量化栈中的可复用模块逻辑。

### Lines 507-518: CompressedTensorsConfig._is_mxint4a16()
```python
    def _is_mxint4a16(self, weight_quant: BaseModel, input_quant: BaseModel) -> bool:
        input_quant_none = input_quant is None
        is_symmetric = weight_quant.symmetric
        is_mxint4 = (
            weight_quant.num_bits == 4
            and weight_quant.type == QuantizationType.INT
            and weight_quant.strategy == QuantizationStrategy.GROUP.value
            and weight_quant.group_size == 32
        )
        is_static = not weight_quant.dynamic

        return is_mxint4 and input_quant_none and is_symmetric and is_static
```
**EN:** This block defines `CompressedTensorsConfig._is_mxint4a16()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_mxint4a16()`，用于实现量化栈中的可复用模块逻辑。

### Lines 520-541: CompressedTensorsConfig._is_dynamic_token_w4()
```python
    def _is_dynamic_token_w4(
        self, weight_quant: BaseModel, input_quant: BaseModel
    ) -> bool:
        is_w4 = weight_quant.num_bits == 4
        weight_strategy = (
            weight_quant.strategy == QuantizationStrategy.TENSOR.value
            or weight_quant.strategy == QuantizationStrategy.CHANNEL.value
            or weight_quant.strategy == QuantizationStrategy.GROUP.value
        )
        if input_quant is not None:
            is_token = (
                weight_strategy
                and input_quant.strategy == QuantizationStrategy.TOKEN.value
            )
            is_dynamic = not weight_quant.dynamic and input_quant.dynamic
        else:
            is_token = weight_strategy
            is_dynamic = not weight_quant.dynamic

        # Both symmetric and asymmetric input quantization supported.
        # Only symmetric weight quantization supported.
        return is_w4 and weight_quant.symmetric and is_token and is_dynamic
```
**EN:** This block defines `CompressedTensorsConfig._is_dynamic_token_w4()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig._is_dynamic_token_w4()`，用于实现量化栈中的可复用模块逻辑。

### Lines 543-587: CompressedTensorsConfig._get_scheme_from_parts() (part 1/2)
```python
    def _get_scheme_from_parts(
        self, weight_quant: BaseModel, input_quant: BaseModel
    ) -> CompressedTensorsLinearScheme:

        # Detect If Mixed Precision
        if self._is_wNa16_group_channel(weight_quant, input_quant):
            if (
                self.quant_format == CompressionFormat.pack_quantized.value
                and weight_quant.num_bits in WNA16_SUPPORTED_BITS
            ):
                return CompressedTensorsWNA16(
                    num_bits=weight_quant.num_bits,
                    strategy=weight_quant.strategy,
                    group_size=weight_quant.group_size,
                    actorder=weight_quant.actorder,
                )
            else:
                raise ImportError(
                    "Other method (CompressedTensorsW4A16Sparse24) is not supported now"
                )

        if is_activation_quantization_format(self.quant_format):
            if self._is_fp4a4_nvfp4(weight_quant, input_quant):
                is_fp4a4_nvfp4_supported = self._check_scheme_supported(
                    CompressedTensorsW4A4Fp4.get_min_capability(), error=False
                )
                if is_fp4a4_nvfp4_supported:
                    return CompressedTensorsW4A4Fp4()
                else:
                    raise NotImplementedError(
                        "Current platform does not support w4a4 nvfp4 quantization."
                    )

            if self._is_fp8_w8a8(weight_quant, input_quant):
                is_fp8_w8a8_supported = self._check_scheme_supported(
                    CompressedTensorsW8A8Fp8.get_min_capability(), error=False
                )
                if is_fp8_w8a8_supported:
                    return CompressedTensorsW8A8Fp8(
                        weight_quant=weight_quant,
                        is_static_input_scheme=(
                            input_quant and not input_quant.dynamic
                        ),
                    )
                else:
```
**EN:** This segment of `CompressedTensorsConfig._get_scheme_from_parts()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `CompressedTensorsConfig._get_scheme_from_parts()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 588-631: CompressedTensorsConfig._get_scheme_from_parts() (part 2/2)
```python
                    # note: input_quant will be present for converted models;
                    # will be ignored during inference post loading
                    return CompressedTensorsW8A16Fp8(
                        strategy=weight_quant.strategy,
                        is_static_input_scheme=not input_quant.dynamic,
                    )

            # note: input_quant can be None
            if self._is_fp8_w8a16(weight_quant, input_quant):
                is_static_input_scheme = input_quant and not input_quant.dynamic
                return CompressedTensorsW8A16Fp8(
                    strategy=weight_quant.strategy,
                    is_static_input_scheme=is_static_input_scheme,
                )

            if self._is_static_tensor_w8a8(weight_quant, input_quant):
                if not _is_npu:
                    return CompressedTensorsW8A8Int8(
                        strategy=weight_quant.strategy,
                        is_static_input_scheme=True,
                        input_symmetric=input_quant.symmetric,
                    )
                else:
                    return NPUCompressedTensorsW8A8Int8(
                        strategy=weight_quant.strategy,
                        is_static_input_scheme=True,
                        input_symmetric=input_quant.symmetric,
                    )

            if self._is_dynamic_token_w8a8(weight_quant, input_quant):
                if not _is_npu:
                    return CompressedTensorsW8A8Int8(
                        strategy=weight_quant.strategy,
                        is_static_input_scheme=False,
                        input_symmetric=input_quant.symmetric,
                    )
                else:
                    return NPUCompressedTensorsW8A8Int8(
                        strategy=weight_quant.strategy,
                        is_static_input_scheme=False,
                        input_symmetric=input_quant.symmetric,
                    )

        raise NotImplementedError("No compressed-tensors compatible scheme was found.")
```
**EN:** This segment of `CompressedTensorsConfig._get_scheme_from_parts()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `CompressedTensorsConfig._get_scheme_from_parts()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 633-677: CompressedTensorsConfig.get_moe_scheme() (part 1/3)
```python
    def get_moe_scheme(
        self, layer: torch.nn.Module, layer_name: Optional[str] = None
    ) -> Optional[CompressedTensorsMoEScheme]:
        """
        compressed-tensors supports non uniform in the following way:

        targets of config_groups: There can be N config_groups which each
            have a quantization scheme. Each config_group has a list of targets
            which can be a full layer_name, a regex for a layer_name, or
            an nn.Module name.

        Detect whether a layer_name is found in any target and
        use the quantization scheme corresponding to the matched target
        to select the CompressedTensorsMoEScheme used for infernece.
        """

        # FusedMoE was made by combining multiple Linears so need to
        # make sure quantization config for Linear can target it
        self._add_fused_moe_to_target_scheme_map()
        unfused_names = [
            layer_name + proj_name
            for proj_name in [".0.gate_proj", ".0.up_proj", ".0.down_proj"]
        ]
        # TODO: refactor this to use expert_mapping and check all layer numbers
        all_scheme_dicts = [self.get_scheme_dict(layer, name) for name in unfused_names]
        scheme_dict = all_scheme_dicts[0] if all_scheme_dicts else None

        # multiple schemes found
        if not all(d == scheme_dict for d in all_scheme_dicts):
            raise ValueError(
                "All MoE projections need to have same "
                "quantization scheme but found multiple"
            )

        if scheme_dict is None:  # ignored layer
            return None

        weight_quant = scheme_dict.get("weights")
        input_quant = scheme_dict.get("input_activations")

        if self._is_wNa16_group_channel(weight_quant, input_quant):
            if not _is_npu:
                if (
                    self._is_mxint4a16(weight_quant, input_quant)
                    and get_moe_runner_backend().is_flashinfer_trtllm()
```
**EN:** This segment of `CompressedTensorsConfig.get_moe_scheme()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `CompressedTensorsConfig.get_moe_scheme()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 678-722: CompressedTensorsConfig.get_moe_scheme() (part 2/3)
```python
                ):
                    logger.info_once(
                        "Using CompressedTensorsMxInt4MoE with flashinfer_trtllm backend"
                    )
                    return CompressedTensorsMxInt4MoE(self)
                elif _is_hip:
                    logger.info_once("Using CompressedTensorsWNA16TritonMoE (ROCm)")
                    return CompressedTensorsWNA16TritonMoE(self)
                else:
                    moe_backend = get_moe_runner_backend()
                    if moe_backend.is_triton():
                        logger.info_once(
                            "Using CompressedTensorsWNA16TritonMoE "
                            "(moe_runner_backend=triton)"
                        )
                        return CompressedTensorsWNA16TritonMoE(self)
                    logger.info_once("Using CompressedTensorsWNA16MarlinMoEMethod")
                    return CompressedTensorsWNA16MoE(self)
            else:
                if (
                    self._is_dynamic_token_w4(weight_quant, input_quant)
                    and input_quant is None
                ):
                    logger.info_once("Using NPUCompressedTensorsW4A16Int4DynamicMoE")
                    return NPUCompressedTensorsW4A16Int4DynamicMoE(self)
        elif self._is_fp4a4_nvfp4(weight_quant, input_quant):
            logger.info_once("Using CompressedTensorsW4A4Nvfp4MoE")
            return CompressedTensorsW4A4Nvfp4MoE()
        elif self._is_fp8_w8a8(weight_quant, input_quant):
            logger.info_once("Using CompressedTensorsW8A8Fp8MoE")
            return CompressedTensorsW8A8Fp8MoE(weight_quant, input_quant)
        elif self._is_dynamic_token_w8a8(weight_quant, input_quant):
            if _is_npu:
                logger.info_once("Using NPUCompressedTensorsW8A8Int8DynamicMoE")
                return NPUCompressedTensorsW8A8Int8DynamicMoE(weight_quant, input_quant)
            else:
                raise NotImplementedError(
                    f"The W8A8Int8 Fused MoE scheme is implemented only for NPU for now."
                )
        elif self._is_dynamic_token_w4a8(weight_quant, input_quant):
            if _is_npu:
                logger.info_once("Using NPUCompressedTensorsW4A8Int8DynamicMoE")
                return NPUCompressedTensorsW4A8Int8DynamicMoE(self)
            else:
                raise NotImplementedError(
```
**EN:** This segment of `CompressedTensorsConfig.get_moe_scheme()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `CompressedTensorsConfig.get_moe_scheme()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 723-728: CompressedTensorsConfig.get_moe_scheme() (part 3/3)
```python
                    f"The W4A8Int8 Fused MoE scheme is implemented only for NPU for now."
                )
        else:
            raise RuntimeError(
                f"Unsupported FusedMoe scheme: {weight_quant}, {input_quant}"
            )
```
**EN:** This segment of `CompressedTensorsConfig.get_moe_scheme()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `CompressedTensorsConfig.get_moe_scheme()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 730-774: CompressedTensorsConfig.get_linear_scheme() (part 1/2)
```python
    def get_linear_scheme(
        self, layer: torch.nn.Module, layer_name: Optional[str] = None
    ) -> Optional[CompressedTensorsLinearScheme]:
        """
        compressed-tensors supports non uniform in the following way:

        targets of config_groups: There can be N config_groups which each
            have a quantization scheme. Each config_group has a list of targets
            which can be a full layer_name, a regex for a layer_name, or
            an nn.Module name.

        Detect whether a layer_name is found in any target and
        use the quantization scheme corresponding to the matched target
        to select the CompressedTensorsScheme used for infernece.
        """

        # Find the "target" in the compressed-tensors config
        # that our layer conforms to.
        # TODO : add compressed-tensors as dep
        # so we do not have to re-write these functions
        # need to make accelerate optional in ct to do this

        # Use the new get_scheme_dict method to extract QuantizationArgs
        scheme_dict = self.get_scheme_dict(layer, layer_name)
        weight_quant = None
        input_quant = None
        if scheme_dict:
            weight_quant = scheme_dict.get("weights")
            input_quant = scheme_dict.get("input_activations")

        # Find the sparsity scheme of the layer
        # assume that fused layers inerhit first component's sparsity scheme
        sparsity_targets = self.sparsity_scheme_map.keys() - set(
            self.sparsity_ignore_list
        )
        sparsity_scheme: Optional[SparsityCompressionConfig] = None
        with suppress(ValueError):
            matched_target = find_matched_target(
                layer_name=layer_name,
                module=layer,
                targets=sparsity_targets,
                fused_mapping=self.packed_modules_mapping,
            )
            sparsity_scheme = self.sparsity_scheme_map[matched_target]
```
**EN:** This segment of `CompressedTensorsConfig.get_linear_scheme()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `CompressedTensorsConfig.get_linear_scheme()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 775-802: CompressedTensorsConfig.get_linear_scheme() (part 2/2)
```python
        if self.supports_cutlass_24(
            weight_quant=weight_quant,
            input_quant=input_quant,
            sparsity_scheme=sparsity_scheme,
        ):
            raise ImportError("CompressedTensors24 is not supported now")
        elif weight_quant is None:
            logger.warning_once(
                "Acceleration for non-quantized schemes is "
                "not supported by Compressed Tensors. "
                "Falling back to UnquantizedLinearMethod"
            )
            return None

        else:
            # Find the quant_scheme
            scheme = self._get_scheme_from_parts(  # type: ignore
                weight_quant=weight_quant,
                input_quant=input_quant,
            )

        # Raise error if device does not support the scheme
        # (e.g. fp8 needs ada lovelace)
        # Note: NPU devices do not support min_capability function
        if not _is_npu:
            self._check_scheme_supported(scheme.get_min_capability())
        logger.debug("Using scheme: %s for %s", scheme.__class__.__name__, layer_name)
        return scheme
```
**EN:** This segment of `CompressedTensorsConfig.get_linear_scheme()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `CompressedTensorsConfig.get_linear_scheme()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 804-833: CompressedTensorsConfig.get_scheme_dict()
```python
    def get_scheme_dict(
        self, layer: torch.nn.Module, layer_name: str | None = None
    ) -> dict[str, QuantizationArgs | str | None] | None:
        """
        Extract the QuantizationArgs for a given layer.

        Returns:
            dict with {
                "weights": QuantizationArgs,
                "input_activations": QuantizationArgs | None,
                "format": str | None
            } | None
        """
        if should_ignore_layer(
            layer_name, ignore=self.ignore, fused_mapping=self.packed_modules_mapping
        ):
            return None

        # Will be empty for models with only sparsity
        if self.target_scheme_map:
            matched_target = find_matched_target(
                layer_name=layer_name,
                module=layer,
                targets=self.target_scheme_map.keys(),
                fused_mapping=self.packed_modules_mapping,
            )

            return self.target_scheme_map[matched_target]

        return None
```
**EN:** This block defines `CompressedTensorsConfig.get_scheme_dict()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_scheme_dict()`，用于为调用方获取或计算派生值。

### Lines 835-849: CompressedTensorsConfig.get_cache_scale()
```python
    def get_cache_scale(self, name: str) -> Optional[str]:
        """
        Check whether the param name matches the format for k/v cache scales
        in compressed-tensors. If this is the case, return its equivalent
        param name expected by vLLM

        :param name: param name
        :return: matching param name for KV cache scale in vLLM
        """
        if name.endswith(".output_scale") and ".k_proj" in name:
            return name.replace(".k_proj.output_scale", ".attn.k_scale")
        if name.endswith(".output_scale") and ".v_proj" in name:
            return name.replace(".v_proj.output_scale", ".attn.v_scale")
        # If no matches, return None
        return None
```
**EN:** This block defines `CompressedTensorsConfig.get_cache_scale()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsConfig.get_cache_scale()`，用于为调用方获取或计算派生值。

### Lines 851-915: CompressedTensorsConfig.supports_cutlass_24()
```python
    @staticmethod
    def supports_cutlass_24(
        weight_quant: Optional[QuantizationArgs],
        input_quant: Optional[QuantizationArgs],
        sparsity_scheme: Optional[SparsityCompressionConfig] = None,
    ) -> bool:
        """
        Check if the layer is supported by the Cutlass 2:4 Kernel
        Conditions:
            - Overarching condition: Sparsity Structure is 2:4
            - Unquantized cases are supported
            - Weight only quantization is not-supported
            - Supported weight quantization strategies are TENSOR and CHANNEL
            - Supported input quantization strategies are TENSOR and TOKEN
            - Only 8 bit quantization is supported

        :return: True if the layer is supported by the Cutlass 2:4 Kernel
            False otherwise
        """
        if sparsity_scheme is None:
            return False

        is_valid_sparsity_structure: bool = (
            sparsity_scheme.sparsity_structure == SparsityStructure.TWO_FOUR.value
        )

        valid_compressors = {
            CompressionFormat.dense.value,
            CompressionFormat.sparse_24_bitmask.value,
        }

        is_valid_sparsity = (
            is_valid_sparsity_structure and sparsity_scheme.format in valid_compressors
        )

        if not is_valid_sparsity:
            return False

        # Unquantized cases are supported
        if weight_quant is None and input_quant is None:
            return True

        # Weight only quantization is not-supported
        if weight_quant is not None and input_quant is None:
            return False

        supported_weight_quant_strategies = [
            QuantizationStrategy.TENSOR.value,
            QuantizationStrategy.CHANNEL.value,
        ]

        assert weight_quant is not None
        assert input_quant is not None
        if weight_quant.strategy not in supported_weight_quant_strategies:
            return False

        supported_input_quant_strategies = [
            QuantizationStrategy.TENSOR.value,
            QuantizationStrategy.TOKEN.value,
        ]

        if input_quant.strategy not in supported_input_quant_strategies:
            return False

        return weight_quant.num_bits == input_quant.num_bits == 8
```
**EN:** This block defines `CompressedTensorsConfig.supports_cutlass_24()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `CompressedTensorsConfig.supports_cutlass_24()`，用于实现量化栈中的可复用模块逻辑。

### Lines 918-919: class CompressedTensorsLinearMethod: definition
```python
class CompressedTensorsLinearMethod(LinearMethodBase):
```
**EN:** This block declares `CompressedTensorsLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, create_weights, apply.
**CN:** 该代码块声明 `CompressedTensorsLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, process_weights_after_loading, create_weights, apply 等行为。

### Lines 920-922: CompressedTensorsLinearMethod.__init__()
```python
    def __init__(self, quantization_config: CompressedTensorsConfig):
        self.quantization_config = quantization_config
        self.quant_config = quantization_config
```
**EN:** This block defines `CompressedTensorsLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 924-925: CompressedTensorsLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** This block defines `CompressedTensorsLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `CompressedTensorsLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 927-951: CompressedTensorsLinearMethod.create_weights()
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
        """
        Use the CompressedTensorsScheme associated with each layer to create
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
**EN:** This block defines `CompressedTensorsLinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsLinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 953-969: CompressedTensorsLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ):
        """
        Use the output of create_weights and the CompressedTensorsScheme
        associated with the layer to apply the forward pass with the
        layer input.  See LinearMethodBase for param details

        """

        scheme = layer.scheme
        if scheme is None:
            raise ValueError("A scheme must be defined for each layer")
        return scheme.apply_weights(layer, x, bias=bias)
```
**EN:** This block defines `CompressedTensorsLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 972-973: class CompressedTensorsFusedMoEMethod: definition
```python
class CompressedTensorsFusedMoEMethod(FusedMoEMethodBase):
```
**EN:** This block declares `CompressedTensorsFusedMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, process_weights_after_loading, create_weights, create_moe_runner.
**CN:** 该代码块声明 `CompressedTensorsFusedMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, process_weights_after_loading, create_weights, create_moe_runner 等行为。

### Lines 974-976: CompressedTensorsFusedMoEMethod.__init__()
```python
    def __init__(self, quantization_config: CompressedTensorsConfig):
        self.quantization_config = quantization_config
        self.quant_config = quantization_config
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 978-979: CompressedTensorsFusedMoEMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 981-1002: CompressedTensorsFusedMoEMethod.create_weights()
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
        Use the CompressedTensorsScheme associated with each layer to create
        the necessary parameters for the layer. See LinearMethodBase for param
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
**EN:** This block defines `CompressedTensorsFusedMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 1004-1007: CompressedTensorsFusedMoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        return layer.scheme.create_moe_runner(layer, moe_runner_config)
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 1009-1010: CompressedTensorsFusedMoEMethod.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer: torch.nn.Module):
        return layer.scheme.get_triton_quant_info(layer)
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 1012-1013: CompressedTensorsFusedMoEMethod.get_marlin_quant_info()
```python
    def get_marlin_quant_info(self, layer: torch.nn.Module):
        return layer.scheme.get_marlin_quant_info(layer)
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.get_marlin_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.get_marlin_quant_info()`，用于为调用方获取或计算派生值。

### Lines 1015-1030: CompressedTensorsFusedMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        """
        Use the output of create_weights and the CompressedTensorsScheme
        associated with the layer to apply the forward pass with the
        layer input.  See LinearMethodBase for param details

        """

        scheme = layer.scheme
        if scheme is None:
            raise ValueError("A scheme must be defined for each layer")
        return scheme.apply_weights(layer, dispatch_output)
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 1032-1040: CompressedTensorsFusedMoEMethod.apply_weights_with_router_logits()
```python
    def apply_weights_with_router_logits(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> torch.Tensor:
        scheme = layer.scheme
        if scheme is None:
            raise ValueError("A scheme must be defined for each layer")
        return scheme.apply_weights_with_router_logits(layer, dispatch_output)
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.apply_weights_with_router_logits()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.apply_weights_with_router_logits()`，用于将量化计算应用到运行时输入上。

### Lines 1042-1058: CompressedTensorsFusedMoEMethod.apply_without_routing_weights()
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        return layer.scheme.apply_without_routing_weights(
            layer,
            hidden_states,
            hidden_states_scale,
            group_list_type,
            group_list,
            output_dtype,
        )
```
**EN:** This block defines `CompressedTensorsFusedMoEMethod.apply_without_routing_weights()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `CompressedTensorsFusedMoEMethod.apply_without_routing_weights()`，用于将量化计算应用到运行时输入上。

## Key Concepts / 关键概念
- `QUANTIZATION_SCHEME_MAP_TYPE`: A module-level constant or registry. / `QUANTIZATION_SCHEME_MAP_TYPE`：模块级常量或注册表。
- `DeviceCapability`: A supporting class that structures file-level quantization behavior. / `DeviceCapability` 是一个支撑类，用于组织该文件中的量化行为。
- `CompressedTensorsConfig`: A configuration class that structures file-level quantization behavior. / `CompressedTensorsConfig` 是一个配置类，用于组织该文件中的量化行为。
- `CompressedTensorsLinearMethod`: A runtime method class that structures file-level quantization behavior. / `CompressedTensorsLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `CompressedTensorsFusedMoEMethod`: A runtime method class that structures file-level quantization behavior. / `CompressedTensorsFusedMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `compressed_tensors.config`, `compressed_tensors.quantization`, `contextlib`, `logging`, `pydantic`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.compressed_tensors.schemes`, `sglang.srt.layers.quantization.compressed_tensors.utils`, `sglang.srt.layers.quantization.fp8`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.models.utils`, `sglang.srt.utils`
