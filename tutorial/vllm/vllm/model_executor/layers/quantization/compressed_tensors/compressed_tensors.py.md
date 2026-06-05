# compressed_tensors.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/compressed_tensors/compressed_tensors.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CompressedTensorsConfig`, `CompressedTensorsLinearMethod`, `CompressedTensorsKVCacheMethod` for quantization backends, schemes, and utilities. / 实现 `CompressedTensorsConfig`, `CompressedTensorsLinearMethod`, `CompressedTensorsKVCacheMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-66)
```python
from functools import partial
from typing import TYPE_CHECKING, Any, Literal, cast

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
from compressed_tensors.transform import TransformConfig

from vllm.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.fused_moe import RoutedExperts
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from vllm.model_executor.layers.quantization import QuantizationMethods
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors_moe import (  # noqa: E501
    CompressedTensorsMoEMethod,
)
from vllm.model_executor.layers.quantization.compressed_tensors.schemes import (
    WNA16_SUPPORTED_BITS,
    CompressedTensors24,
    CompressedTensorsScheme,
    CompressedTensorsW4A4Fp4,
    CompressedTensorsW4A4Mxfp4,
    CompressedTensorsW4A8Fp8,
    CompressedTensorsW4A8Int,
    CompressedTensorsW4A16Fp4,
    CompressedTensorsW8A8Fp8,
    CompressedTensorsW8A8Int8,
    CompressedTensorsW8A8Mxfp8,
    CompressedTensorsW8A16Fp8,
    CompressedTensorsWNA16,
)
from vllm.model_executor.layers.quantization.compressed_tensors.transform.linear import (  # noqa: E501
    CompressedTensorsLinearTransformMethod,
    get_linear_transform_schemes,
)
from vllm.model_executor.layers.quantization.compressed_tensors.utils import (
    find_matched_target,
    is_activation_quantization_format,
    should_ignore_layer,
)
from vllm.model_executor.layers.quantization.kv_cache import BaseKVCacheMethod
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.platforms import current_platform
```
**EN:** This opening block pulls in external dependencies such as `functools`, `typing`, `torch`, `compressed_tensors` and internal modules such as `vllm.distributed`, `vllm.logger`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `functools`, `typing`, `torch`, `compressed_tensors`）以及内部模块（如 `vllm.distributed`, `vllm.logger`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 71-76)
```python
logger = init_logger(__name__)

__all__ = ["CompressedTensorsLinearMethod"]

SPARSITY_CONFIG_NAME: Literal["sparsity_config"] = "sparsity_config"
QUANTIZATION_SCHEME_MAP_TYPE = dict[str, dict[str, QuantizationArgs] | None]
```
**EN:** This block defines module-level metadata or constants such as `logger`, `__all__`, `SPARSITY_CONFIG_NAME`, `QUANTIZATION_SCHEME_MAP_TYPE`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `__all__`, `SPARSITY_CONFIG_NAME`, `QUANTIZATION_SCHEME_MAP_TYPE`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Class `CompressedTensorsConfig` overview (lines 79-905)
```python
class CompressedTensorsConfig(QuantizationConfig):
    def __init__(
        self,
        target_scheme_map: dict[str, Any],
        ignore: list[str],
        quant_format: str,
        sparsity_scheme_map: dict[str, SparsityCompressionConfig],
        sparsity_ignore_list: list[str],
        kv_cache_scheme: dict[str, Any] | None = None,
        config: dict[str, Any] | None = None,
        transform_config: dict[str, Any] | None = None,
        total_num_heads: int | None = None,
        total_num_kv_heads: int | None = None,
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
        self.total_num_heads = total_num_heads
        self.total_num_kv_heads = total_num_kv_heads
```
**EN:** Defines class `CompressedTensorsConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 31 direct methods, with notable entries `__init__`, `get_linear_method`, `get_supported_act_dtypes`, `get_min_capability`, `get_name`, `apply_vllm_mapper`.
**CN:** 定义类 `CompressedTensorsConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 31 个方法，较重要的包括 `__init__`, `get_linear_method`, `get_supported_act_dtypes`, `get_min_capability`, `get_name`, `apply_vllm_mapper`。

### Method `CompressedTensorsConfig.__init__` (lines 80-108)
```python
    def __init__(
        self,
        target_scheme_map: dict[str, Any],
        ignore: list[str],
        quant_format: str,
        sparsity_scheme_map: dict[str, SparsityCompressionConfig],
        sparsity_ignore_list: list[str],
        kv_cache_scheme: dict[str, Any] | None = None,
        config: dict[str, Any] | None = None,
        transform_config: dict[str, Any] | None = None,
        total_num_heads: int | None = None,
        total_num_kv_heads: int | None = None,
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
        self.total_num_heads = total_num_heads
        self.total_num_kv_heads = total_num_kv_heads

        if transform_config:
            self.transform_config = TransformConfig.model_validate(transform_config)
        else:
            self.transform_config = None
```
**EN:** Defines function `CompressedTensorsConfig.__init__` with signature `__init__(self, target_scheme_map: dict[str, Any], ignore: list[str], quant_format: str, sparsity_scheme_map: dict[str, SparsityCompressionConfig], sparsity_ignore_list: list[str], kv_cache_scheme: dict[str, Any] | None=None, config: dict[str, Any] | None=None, transform_config: dict[str, Any] | None=None, total_num_heads: int | None=None, total_num_kv_heads: int | None=None)`. It mainly works with `target_scheme_map`, `ignore`, `quant_format`, `sparsity_scheme_map`, `sparsity_ignore_list`, `kv_cache_scheme`, `config`, `transform_config`; initializes the object state and cached resources. The body uses branching. Key calls include `super.__init__`, `TransformConfig.model_validate`, `super`.
**CN:** 定义函数 `CompressedTensorsConfig.__init__`，其签名为 `__init__(self, target_scheme_map: dict[str, Any], ignore: list[str], quant_format: str, sparsity_scheme_map: dict[str, SparsityCompressionConfig], sparsity_ignore_list: list[str], kv_cache_scheme: dict[str, Any] | None=None, config: dict[str, Any] | None=None, transform_config: dict[str, Any] | None=None, total_num_heads: int | None=None, total_num_kv_heads: int | None=None)`。它主要围绕 `target_scheme_map`, `ignore`, `quant_format`, `sparsity_scheme_map`, `sparsity_ignore_list`, `kv_cache_scheme`, `config`, `transform_config` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `super.__init__`, `TransformConfig.model_validate`, `super`。

### Method `CompressedTensorsConfig.apply_vllm_mapper` (lines 123-154)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper: "WeightsMapper"):
        """
        Transform layer paths in config targets to match vLLM's naming.

        The WeightsMapper is designed for weight paths, but some backends
        (e.g. transformers) use broad prefix mappings like "" -> "model."
        which would incorrectly transform non-path targets.

        compressed-tensors targets can be:
        - Layer paths: "layers.0.self_attn.q_proj" -> transformed
        - Module class names: "Linear" -> preserved (no ".")
        - Regex patterns: "re:.*proj" -> preserved (starts with "re:")
        """

        def _map_target(target: str) -> str | None:
            is_layer_path = "." in target and not target.startswith("re:")
            if is_layer_path:
                return hf_to_vllm_mapper._map_name(target)
            return target

        def _apply_dict(d: dict) -> dict:
            return {k: v for t, v in d.items() if (k := _map_target(t)) is not None}

        def _apply_list(lst: list) -> list:
            return [t for x in lst if (t := _map_target(x)) is not None]

        self.target_scheme_map = _apply_dict(self.target_scheme_map)
        self.ignore = _apply_list(self.ignore)
        self.sparsity_scheme_map = _apply_dict(self.sparsity_scheme_map)
        self.sparsity_ignore_list = _apply_list(self.sparsity_ignore_list)
        if self.kv_cache_scheme is not None:
            self.kv_cache_scheme = _apply_dict(self.kv_cache_scheme)
```
**EN:** Defines function `CompressedTensorsConfig.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses branching, comprehensions. Key calls include `_apply_dict`, `_apply_list`, `hf_to_vllm_mapper._map_name`, `target.startswith`, `d.items`, `_map_target`.
**CN:** 定义函数 `CompressedTensorsConfig.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、推导式。关键调用包括 `_apply_dict`, `_apply_list`, `hf_to_vllm_mapper._map_name`, `target.startswith`, `d.items`, `_map_target`。

### Method `CompressedTensorsConfig.from_config` (lines 215-252)
```python
    def from_config(cls, config: dict[str, Any]) -> "CompressedTensorsConfig":
        # We keep only config groups which are not doing Attention quantization
        # because Attention quantization on its own is not supported by vLLM.
        # It is coupled with KV-cache quantization, and if scales are present in the
        # checkpoint, they will be used properly.
        if "config_groups" in config:
            grps_without_attn_quant = {}
            for k, v in config["config_groups"].items():
                # e.g. LlamaAttention, Qwen3Attention, etc.
                if len(v["targets"]) == 1 and v["targets"][0].endswith("Attention"):
                    logger.warning(
                        "Skipping CompressedTensors config group for %s. Attention "
                        "quant is coupled with KV-cache quantization in vLLM.",
                        v["targets"][0],
                    )
                    continue
                grps_without_attn_quant[k] = v
            config["config_groups"] = grps_without_attn_quant

        ignore: list[str] = cast(list[str], config.get("ignore", []))
        quant_format = cast(str, config.get("format"))
        target_scheme_map = cls._quantization_scheme_map_from_config(config=config)
        sparsity_scheme_map, sparsity_ignore_list = cls._parse_sparsity_config(
            config=config
        )

        return cls(
            target_scheme_map=target_scheme_map,
            ignore=ignore,
            quant_format=quant_format,
            sparsity_scheme_map=sparsity_scheme_map,
            sparsity_ignore_list=sparsity_ignore_list,
            config=config,
            transform_config=config.get("transform_config"),
            kv_cache_scheme=config.get("kv_cache_scheme"),
            total_num_heads=config.get("total_num_heads"),
            total_num_kv_heads=config.get("total_num_kv_heads"),
        )
```
**EN:** Defines function `CompressedTensorsConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'CompressedTensorsConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses branching, iteration. Key calls include `cast`, `cls._quantization_scheme_map_from_config`, `cls._parse_sparsity_config`, `cls`, `config.items`, `config.get`.
**CN:** 定义函数 `CompressedTensorsConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'CompressedTensorsConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `cast`, `cls._quantization_scheme_map_from_config`, `cls._parse_sparsity_config`, `cls`, `config.items`, `config.get`。

### Method `CompressedTensorsConfig.get_scheme` (lines 716-794)
```python
    def get_scheme(
        self, layer: torch.nn.Module, layer_name: str | None = None
    ) -> "CompressedTensorsScheme | None":
        """
        compressed-tensors supports non uniform in the following way:

        targets of config_groups: There can be N config_groups which each
            have a quantization scheme. Each config_group has a list of targets
            which can be a full layer_name, a regex for a layer_name, or
            an nn.Module name.

        Detect whether a layer_name is found in any target and
        use the quantization scheme corresponding to the matched target
        to select the CompressedTensorsScheme used for inference.
        """

        # Use the new get_quant_args method to extract QuantizationArgs
        scheme_dict = self.get_scheme_dict(layer, layer_name)

        weight_quant = None
        input_quant = None
        format = None
        if scheme_dict:
            weight_quant = scheme_dict.get("weights")
            input_quant = scheme_dict.get("input_activations")
            format = scheme_dict.get("format")

        # Find the sparsity scheme of the layer
        # assume that fused layers inherit first component's sparsity scheme
        sparsity_targets = self.sparsity_scheme_map.keys() - set(
            self.sparsity_ignore_list
        )
        sparsity_scheme: SparsityCompressionConfig | None = None
        matched_target = find_matched_target(
            layer_name=layer_name,
            module=layer,
            targets=sparsity_targets,
            fused_mapping=self.packed_modules_mapping,
# ... truncated for analysis ...
        else:
            # Find the quant_scheme
            scheme = self._get_scheme_from_parts(  # type: ignore
                weight_quant=weight_quant,
                input_quant=input_quant,
                format=format,
                layer_name=layer_name,
            )

        # Raise error if device does not support the scheme
        # (e.g. fp8 needs ada lovelace)
        self._check_scheme_supported(scheme.get_min_capability())
        logger.debug("Using scheme: %s for %s", scheme.__class__.__name__, layer_name)
        return scheme
```
**EN:** Defines function `CompressedTensorsConfig.get_scheme` with signature `get_scheme(self, layer: torch.nn.Module, layer_name: str | None=None) -> 'CompressedTensorsScheme | None'`. It mainly works with `layer`, `layer_name`; returns a derived property or capability check. The body uses branching. Key calls include `self.get_scheme_dict`, `find_matched_target`, `self.supports_cutlass_24`, `self._check_scheme_supported`, `logger.debug`, `scheme_dict.get`.
**CN:** 定义函数 `CompressedTensorsConfig.get_scheme`，其签名为 `get_scheme(self, layer: torch.nn.Module, layer_name: str | None=None) -> 'CompressedTensorsScheme | None'`。它主要围绕 `layer`, `layer_name` 展开；返回派生属性或能力判断结果。函数体包含分支判断。关键调用包括 `self.get_scheme_dict`, `find_matched_target`, `self.supports_cutlass_24`, `self._check_scheme_supported`, `logger.debug`, `scheme_dict.get`。

### Class `CompressedTensorsLinearMethod` overview (lines 908-956)
```python
class CompressedTensorsLinearMethod(LinearMethodBase):
    def __init__(self, quantization_config: CompressedTensorsConfig):
        self.quantization_config = quantization_config

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)

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
        Use the CompressedTensorsScheme associated with each layer to create
        the necessary parameters for the layer. See LinearMethodBase for param
        details
        """
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.scheme.create_weights(
            layer=layer,
```
**EN:** Defines class `CompressedTensorsLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `process_weights_after_loading`, `create_weights`, `apply`.
**CN:** 定义类 `CompressedTensorsLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `process_weights_after_loading`, `create_weights`, `apply`。

### Method `CompressedTensorsLinearMethod.__init__` (lines 909-910)
```python
    def __init__(self, quantization_config: CompressedTensorsConfig):
        self.quantization_config = quantization_config
```
**EN:** Defines function `CompressedTensorsLinearMethod.__init__` with signature `__init__(self, quantization_config: CompressedTensorsConfig)`. It mainly works with `quantization_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `CompressedTensorsLinearMethod.__init__`，其签名为 `__init__(self, quantization_config: CompressedTensorsConfig)`。它主要围绕 `quantization_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `CompressedTensorsLinearMethod.process_weights_after_loading` (lines 912-913)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** Defines function `CompressedTensorsLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses mostly straightforward data movement and object wiring. Key calls include `layer.scheme.process_weights_after_loading`.
**CN:** 定义函数 `CompressedTensorsLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `layer.scheme.process_weights_after_loading`。

### Method `CompressedTensorsLinearMethod.create_weights` (lines 915-939)
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
**EN:** Defines function `CompressedTensorsLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses mostly straightforward data movement and object wiring. Key calls include `extra_weight_attrs.get`, `layer.scheme.create_weights`.
**CN:** 定义函数 `CompressedTensorsLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `extra_weight_attrs.get`, `layer.scheme.create_weights`。

### Method `CompressedTensorsLinearMethod.apply` (lines 941-956)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
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
**EN:** Defines function `CompressedTensorsLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None)`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `scheme.apply_weights`, `ValueError`.
**CN:** 定义函数 `CompressedTensorsLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None)`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `scheme.apply_weights`, `ValueError`。

### Class `CompressedTensorsKVCacheMethod` overview (lines 959-1143)
```python
class CompressedTensorsKVCacheMethod(BaseKVCacheMethod):
    """
    Supports loading kv-cache scaling factors from compressed-tensors
    checkpoints.
    """

    def __init__(self, quant_config: CompressedTensorsConfig):
        self.validate_kv_cache_scheme(quant_config.kv_cache_scheme)
        super().__init__(quant_config)

    @staticmethod
    def validate_kv_cache_scheme(kv_cache_scheme: dict[str, Any] | None):
        """
        Validator for the kv cache scheme. Useful for controlling the
        kv cache quantization schemes, that are being supported in vLLM
        :param kv_cache_scheme: the compressed-tensors kv cache scheme
        """
        if kv_cache_scheme is None:
            return

        type_ = kv_cache_scheme.get("type")
        num_bits = kv_cache_scheme.get("num_bits")

        if type_ != "float" and num_bits != 8:
            raise NotImplementedError(
```
**EN:** Defines class `CompressedTensorsKVCacheMethod` with base classes `BaseKVCacheMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `validate_kv_cache_scheme`, `create_weights`, `process_weights_after_loading`. Its docstring says: Supports loading kv-cache scaling factors from compressed-tensors checkpoints.
**CN:** 定义类 `CompressedTensorsKVCacheMethod`，其基类为 `BaseKVCacheMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `validate_kv_cache_scheme`, `create_weights`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `CompressedTensorsKVCacheMethod.__init__` (lines 965-967)
```python
    def __init__(self, quant_config: CompressedTensorsConfig):
        self.validate_kv_cache_scheme(quant_config.kv_cache_scheme)
        super().__init__(quant_config)
```
**EN:** Defines function `CompressedTensorsKVCacheMethod.__init__` with signature `__init__(self, quant_config: CompressedTensorsConfig)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `self.validate_kv_cache_scheme`, `super.__init__`, `super`.
**CN:** 定义函数 `CompressedTensorsKVCacheMethod.__init__`，其签名为 `__init__(self, quant_config: CompressedTensorsConfig)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.validate_kv_cache_scheme`, `super.__init__`, `super`。

### Method `CompressedTensorsKVCacheMethod.validate_kv_cache_scheme` (lines 970-1007)
```python
    def validate_kv_cache_scheme(kv_cache_scheme: dict[str, Any] | None):
        """
        Validator for the kv cache scheme. Useful for controlling the
        kv cache quantization schemes, that are being supported in vLLM
        :param kv_cache_scheme: the compressed-tensors kv cache scheme
        """
        if kv_cache_scheme is None:
            return

        type_ = kv_cache_scheme.get("type")
        num_bits = kv_cache_scheme.get("num_bits")

        if type_ != "float" and num_bits != 8:
            raise NotImplementedError(
                "Currently supported kv cache quantization is "
                "num_bits=8, type=float, however "
                f"received num_bits={num_bits}, type={type_}"
            )

        strategy = QuantizationStrategy(kv_cache_scheme.get("strategy"))
        supported_strategies = (
            QuantizationStrategy.TENSOR,
            QuantizationStrategy.ATTN_HEAD,
        )
        if strategy not in supported_strategies:
            raise NotImplementedError(
                "Invalid strategy for compressed-tensors KV cache. "
                f"Expected strategies: {supported_strategies}, found strategy:"
                f" {strategy}"
            )

        is_symmetric = kv_cache_scheme.get("symmetric")
        if not is_symmetric:
            raise NotImplementedError(
                "Only support symmetric scaling factor "
                "for compressed-tensors KV cache. "
                f"However found symmetric: {is_symmetric}"
            )
```
**EN:** Defines function `CompressedTensorsKVCacheMethod.validate_kv_cache_scheme` with signature `validate_kv_cache_scheme(kv_cache_scheme: dict[str, Any] | None)`. It mainly works with `kv_cache_scheme`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `kv_cache_scheme.get`, `QuantizationStrategy`, `NotImplementedError`.
**CN:** 定义函数 `CompressedTensorsKVCacheMethod.validate_kv_cache_scheme`，其签名为 `validate_kv_cache_scheme(kv_cache_scheme: dict[str, Any] | None)`。它主要围绕 `kv_cache_scheme` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `kv_cache_scheme.get`, `QuantizationStrategy`, `NotImplementedError`。

### Method `CompressedTensorsKVCacheMethod.create_weights` (lines 1009-1115)
```python
    def create_weights(self, layer: torch.nn.Module):
        """
        Initialize placeholder scales and zero points to enable loading of
        quantized params from compressed-tensors checkpoints.
        """
        strategy = None  # for backward compatibility
        if (
            hasattr(self.quant_config, "kv_cache_scheme")
            and self.quant_config.kv_cache_scheme is not None
        ):
            strategy = QuantizationStrategy(
                self.quant_config.kv_cache_scheme["strategy"]
            )

        n_scales = int(layer.num_kv_heads) if strategy == "attn_head" else 1

        layer.k_scale = torch.nn.Parameter(
            torch.ones(n_scales, requires_grad=False, dtype=torch.float32)
        )
        layer.v_scale = torch.nn.Parameter(
            torch.ones(n_scales, requires_grad=False, dtype=torch.float32)
        )
        layer.q_scale = torch.nn.Parameter(
            torch.ones(n_scales, requires_grad=False, dtype=torch.float32)
        )

        # Zero points are not used in vLLM as currently only symmetric quantization is
        # supported. We need to create them here to enable loading of llm-compressor
        # checkpoints which contain them irrespective of the symmetric/asymmetric
        # scheme used during quantization.
        layer.k_zero_point = torch.nn.Parameter(
            torch.zeros(n_scales, requires_grad=False)
        )
        layer.v_zero_point = torch.nn.Parameter(
            torch.zeros(n_scales, requires_grad=False)
        )
        layer.q_zero_point = torch.nn.Parameter(
            torch.zeros(n_scales, requires_grad=False)
# ... truncated for analysis ...
            )
            layer.v_scale.weight_loader = partial(
                _tp_aware_loader, kind="v", param_type="scale"
            )

            layer.q_zero_point.weight_loader = partial(
                _tp_aware_loader, kind="q", param_type="zero_point"
            )
            layer.k_zero_point.weight_loader = partial(
                _tp_aware_loader, kind="k", param_type="zero_point"
            )
            layer.v_zero_point.weight_loader = partial(
                _tp_aware_loader, kind="v", param_type="zero_point"
            )
```
**EN:** Defines function `CompressedTensorsKVCacheMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module)`. It mainly works with `layer`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `hasattr`, `QuantizationStrategy`, `int`, `torch.ones`, `torch.zeros`.
**CN:** 定义函数 `CompressedTensorsKVCacheMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module)`。它主要围绕 `layer` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `hasattr`, `QuantizationStrategy`, `int`, `torch.ones`, `torch.zeros`。

### Method `CompressedTensorsKVCacheMethod.process_weights_after_loading` (lines 1117-1143)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        """
        Override the default vLLM placeholder scales with the llm-compressor loaded
        scales. Zero points are not used as only symmetric quantization is supported.
        """
        layer._k_scale = layer.k_scale
        layer._v_scale = layer.v_scale
        layer._q_scale = layer.q_scale

        # Set the _float variants that the attention backend uses.
        def _to_scalar(tensor: torch.Tensor) -> float:
            # For n_scales > 1 (e.g., ATTN_HEAD strategy), take max
            if tensor.numel() > 1:
                return tensor.max().item()
            return tensor.item()

        layer._k_scale_float = _to_scalar(layer.k_scale)
        layer._v_scale_float = _to_scalar(layer.v_scale)
        layer._q_scale_float = _to_scalar(layer.q_scale)

        # Discard all placeholders.
        del layer.k_scale
        del layer.v_scale
        del layer.q_scale
        del layer.k_zero_point
        del layer.v_zero_point
        del layer.q_zero_point
```
**EN:** Defines function `CompressedTensorsKVCacheMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching. Key calls include `_to_scalar`, `tensor.item`, `tensor.numel`, `tensor.max.item`, `tensor.max`.
**CN:** 定义函数 `CompressedTensorsKVCacheMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断。关键调用包括 `_to_scalar`, `tensor.item`, `tensor.numel`, `tensor.max.item`, `tensor.max`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `CompressedTensorsConfig`, `CompressedTensorsLinearMethod`, `CompressedTensorsKVCacheMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `CompressedTensorsConfig`, `CompressedTensorsLinearMethod`, `CompressedTensorsKVCacheMethod`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `functools`, `typing`, `torch`, `compressed_tensors`
- **Internal / 内部**: `vllm.distributed`, `vllm.logger`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.compressed_tensors.compressed_tensors_moe`, `vllm.model_executor.layers.quantization.compressed_tensors.schemes`, `vllm.model_executor.layers.quantization.compressed_tensors.transform.linear`, `vllm.model_executor.layers.quantization.compressed_tensors.utils`, `vllm.model_executor.layers.quantization.kv_cache`, `vllm.model_executor.layers.vocab_parallel_embedding`, `vllm.platforms`
