# modelslim.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/modelslim.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `ModelSlimConfig`, and `ModelSlimLinearMethod`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `ModelSlimConfig` 和 `ModelSlimLinearMethod` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import logging
from types import MappingProxyType
from typing import TYPE_CHECKING, Any, Dict, List, Mapping, Optional, cast

import torch

from sglang.multimodal_gen.runtime.layers.linear import (
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.compressed_tensors.utils import should_ignore_layer
from sglang.srt.layers.quantization.modelslim.schemes import (
    ModelSlimW4A4Int4,
    ModelSlimW8A8Int8,
)
```
**EN:** This block establishes the module context and imports `__future__`, `logging`, `types`, `typing`, `torch`, and `sglang.multimodal_gen.runtime.layers.linear`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`logging`、`types`、`typing`、`torch` 和 `sglang.multimodal_gen.runtime.layers.linear`。这些依赖为后续实现提供所需符号。

### Lines 23-30: supporting statements / 辅助语句
```python
if TYPE_CHECKING:
    from sglang.srt.layers.quantization.modelslim.schemes import (
        ModelSlimLinearScheme,
    )

from sglang.multimodal_gen.runtime.loader.utils import get_param_names_mapping

logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 33-43: `ModelSlimConfig` class overview / `ModelSlimConfig` 类概览
```python
class ModelSlimConfig(QuantizationConfig):
    """
    Config class for ModelSlim Quantization of Diffusion models https://gitcode.com/Ascend/msmodelslim, a NPU-specific quantization type.
    The quantization method (W8A8, W4A4, etc.) will be automatically parsed from the `quant_model_description.json` config.

    ModelSlim for Diffusion models includes support for various quantization schemes, such as:
    - W4A4 dynamic linear
    - W8A8 static linear
    - W8A8 dynamic linear
    """
```
**EN:** This block defines class `ModelSlimConfig`. Config class for ModelSlim Quantization of Diffusion models https://gitcode.com/Ascend/msmodelslim, a NPU-specific quantization type. The quantization method (W8A8, W4A4, etc.) will be automatically parsed from the `quant_model_description.json` config. It inherits from `QuantizationConfig`.
**CN:** 该代码块定义了类 `ModelSlimConfig`。 它用于封装 model slim config 相关行为。 它继承自 `QuantizationConfig`。

### Lines 44-61: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        quant_config: Dict[str, Any] = {},
        reverse_param_names_mapping: dict = None,
    ):
        super().__init__()
        self.quant_description = quant_config
        ignore = cast(List[str], quant_config.get("ignore", []))
        self.ignore = ignore
        packed_modules_mapping = quant_config.get("packed_modules_mapping", {})
        self.packed_modules_mapping = (
            packed_modules_mapping if packed_modules_mapping is not None else {}
        )
        self._name_mapper = (
            get_param_names_mapping(reverse_param_names_mapping)
            if reverse_param_names_mapping is not None
            else None
        )
```
**EN:** This block defines method `__init__` on `ModelSlimConfig`. It initializes the instance state. Key calls include `super.__init__`, `cast`, `quant_config.get`, `get_param_names_mapping`, and `super`. Parameters such as `quant_config`, and `reverse_param_names_mapping` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`cast`、`quant_config.get`、`get_param_names_mapping` 和 `super`。 本段逻辑主要由 `quant_config` 和 `reverse_param_names_mapping` 等参数驱动。

### Lines 63-64: `get_linear_method` implementation / `get_linear_method` 实现
```python
    def get_linear_method(self) -> ModelSlimLinearMethod:
        return ModelSlimLinearMethod(self)
```
**EN:** This block defines method `get_linear_method` on `ModelSlimConfig`. It retrieves linear method. Key calls include `ModelSlimLinearMethod`.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_linear_method`。 它用于获取linear method。 关键调用包括 `ModelSlimLinearMethod`。

### Lines 66-68: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.int8, torch.float16, torch.bfloat16]
```
**EN:** This block defines method `get_supported_act_dtypes` on `ModelSlimConfig`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 70-72: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 0
```
**EN:** This block defines method `get_min_capability` on `ModelSlimConfig`. It retrieves min capability.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 74-76: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "modelslim"
```
**EN:** This block defines method `get_name` on `ModelSlimConfig`. It retrieves name.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_name`。 它用于获取name。

### Lines 78-81: `get_config_filenames` implementation / `get_config_filenames` 实现
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        filenames = ["quant_model_description.json"]
        return filenames
```
**EN:** This block defines method `get_config_filenames` on `ModelSlimConfig`. It retrieves config filenames.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_config_filenames`。 它用于获取config filenames。

### Lines 83-87: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(
        cls, config: Dict[str, Any], reverse_param_names_mapping: dict = None
    ) -> ModelSlimConfig:
        return cls(config, reverse_param_names_mapping)
```
**EN:** This block defines method `from_config` on `ModelSlimConfig`. It constructs from config. Key calls include `cls`. Parameters such as `config`, and `reverse_param_names_mapping` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `cls`。 本段逻辑主要由 `config` 和 `reverse_param_names_mapping` 等参数驱动。

### Lines 89-118: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(
        self,
        layer: torch.nn.Module,
        prefix: str,
    ) -> Optional[QuantizeMethodBase]:
        from sglang.multimodal_gen.runtime.layers.linear import LinearBase

        if isinstance(layer, LinearBase):
            if should_ignore_layer(
                prefix,
                ignore=self.ignore,
                fused_mapping=self.packed_modules_mapping,
            ):
                return UnquantizedLinearMethod()
            key = "model"
            packed_modules_mapping_subset = self.packed_modules_mapping.get(key, {})
            prefix_in_quant_config = prefix
            proj_name = prefix.split(".")[-1]
            if proj_name in packed_modules_mapping_subset:
                prefix_in_quant_config = prefix.replace(
                    proj_name, packed_modules_mapping_subset[proj_name][0]
                )

            if self.is_layer_skipped(prefix, packed_modules_mapping_subset):
                return UnquantizedLinearMethod()
            scheme = self.get_scheme(layer=layer, layer_name=prefix_in_quant_config)
            layer.scheme = scheme
            return ModelSlimLinearMethod(self)
        else:
            return None
```
**EN:** This block defines method `get_quant_method` on `ModelSlimConfig`. It retrieves quant method. Key calls include `isinstance`, `should_ignore_layer`, `self.packed_modules_mapping.get`, `self.is_layer_skipped`, and `self.get_scheme`. The implementation branches on conditions. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `isinstance`、`should_ignore_layer`、`self.packed_modules_mapping.get`、`self.is_layer_skipped` 和 `self.get_scheme`。 实现中包含条件分支。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 120-142: `_get_scheme_from_parts` implementation / `_get_scheme_from_parts` 实现
```python
    def _get_scheme_from_parts(
        self,
        layer_name: str,
    ) -> ModelSlimLinearScheme:
        full_weight_name = layer_name + ".weight"
        if self._name_mapper is not None:
            mapped_name, _, _ = self._name_mapper(full_weight_name)
        else:
            mapped_name = full_weight_name

        quant_type = self.quant_description.get(mapped_name, "")
        prefix = mapped_name.removesuffix(".weight")
        if quant_type == "W8A8_DYNAMIC" or quant_type == "W8A8":
            return ModelSlimW8A8Int8(quant_config=self.quant_description, prefix=prefix)
        elif quant_type == "W4A4_DYNAMIC":
            return ModelSlimW4A4Int4(quant_config=self.quant_description, prefix=prefix)
        elif quant_type == "W8A8_MXFP8":
            from sglang.multimodal_gen.runtime.layers.quantization.modelslim_mxfp8_scheme import (
                ModelSlimMXFP8Scheme,
            )

            return ModelSlimMXFP8Scheme()
        raise NotImplementedError("No modelslim compatible scheme was found.")
```
**EN:** This block defines method `_get_scheme_from_parts` on `ModelSlimConfig`. It retrieves scheme from parts. Key calls include `self.quant_description.get`, `mapped_name.removesuffix`, `NotImplementedError`, `self._name_mapper`, and `ModelSlimW8A8Int8`. The implementation branches on conditions. Parameters such as `layer_name` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `_get_scheme_from_parts`。 它用于获取scheme from parts。 关键调用包括 `self.quant_description.get`、`mapped_name.removesuffix`、`NotImplementedError`、`self._name_mapper` 和 `ModelSlimW8A8Int8`。 实现中包含条件分支。 本段逻辑主要由 `layer_name` 等参数驱动。

### Lines 144-157: `get_scheme` implementation / `get_scheme` 实现
```python
    def get_scheme(
        self, layer: torch.nn.Module, layer_name: Optional[str] = None
    ) -> Optional[ModelSlimLinearScheme]:
        """
        get_scheme method adjusted for modelslim, taken from
        python/sglang/srt/layers/quantization/compressed_tensors/compressed_tensors.py
        """
        scheme = self._get_scheme_from_parts(
            layer_name=layer_name,
        )

        # Ascend doesn't support device capability
        logger.debug("Using scheme: %s for %s", scheme.__class__.__name__, layer_name)
        return scheme
```
**EN:** This block defines method `get_scheme` on `ModelSlimConfig`. get_scheme method adjusted for modelslim, taken from python/sglang/srt/layers/quantization/compressed_tensors/compressed_tensors.py Key calls include `self._get_scheme_from_parts`, and `logger.debug`. Parameters such as `layer`, and `layer_name` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_scheme`。 它用于获取scheme。 关键调用包括 `self._get_scheme_from_parts` 和 `logger.debug`。 本段逻辑主要由 `layer` 和 `layer_name` 等参数驱动。

### Lines 159-188: `is_layer_skipped` implementation / `is_layer_skipped` 实现
```python
    def is_layer_skipped(
        self, prefix: str, fused_mapping: Mapping[str, List[str]] = MappingProxyType({})
    ):
        # adapted from vllm.model_executor.layers.quantization.utils.quant_utils.is_layer_skipped
        proj_name = prefix.split(".")[-1]
        if proj_name in fused_mapping:
            shard_prefixes = [
                prefix.replace(proj_name, shard_proj_name)
                for shard_proj_name in fused_mapping[proj_name]
            ]

            is_skipped = None
            for shard_prefix in shard_prefixes:
                is_shard_skipped = (
                    self.quant_description.get(shard_prefix + ".weight", "") == "FLOAT"
                )

                if is_skipped is None:
                    is_skipped = is_shard_skipped
                elif is_shard_skipped != is_skipped:
                    raise ValueError(
                        f"Detected some but not all shards of {prefix} "
                        "are quantized. All shards of fused layers "
                        "to have the same precision."
                    )
        else:
            is_skipped = self.quant_description.get(prefix + ".weight", "") == "FLOAT"

        assert is_skipped is not None
        return is_skipped
```
**EN:** This block defines method `is_layer_skipped` on `ModelSlimConfig`. It handles is layer skipped logic. Key calls include `MappingProxyType`, `prefix.split`, `prefix.replace`, `self.quant_description.get`, and `ValueError`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `prefix`, and `fused_mapping` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `is_layer_skipped`。 它用于处理 is layer skipped 相关逻辑。 关键调用包括 `MappingProxyType`、`prefix.split`、`prefix.replace`、`self.quant_description.get` 和 `ValueError`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `prefix` 和 `fused_mapping` 等参数驱动。

### Lines 190-191: `get_scaled_act_names` implementation / `get_scaled_act_names` 实现
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines method `get_scaled_act_names` on `ModelSlimConfig`. It retrieves scaled act names.
**CN:** 该代码块定义了 `ModelSlimConfig` 的方法 `get_scaled_act_names`。 它用于获取scaled act names。

### Lines 194-195: `ModelSlimLinearMethod` class overview / `ModelSlimLinearMethod` 类概览
```python
class ModelSlimLinearMethod(LinearMethodBase):
```
**EN:** This block defines class `ModelSlimLinearMethod`. It encapsulates model slim linear method behavior. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `ModelSlimLinearMethod`。 它用于封装 model slim linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 196-197: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, quantization_config: ModelSlimConfig):
        self.quantization_config = quantization_config
```
**EN:** This block defines method `__init__` on `ModelSlimLinearMethod`. It initializes the instance state. Parameters such as `quantization_config` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimLinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `quantization_config` 等参数驱动。

### Lines 199-200: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.scheme.process_weights_after_loading(layer)
```
**EN:** This block defines method `process_weights_after_loading` on `ModelSlimLinearMethod`. It processes weights after loading. Key calls include `layer.scheme.process_weights_after_loading`. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimLinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `layer.scheme.process_weights_after_loading`。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 202-226: `create_weights` implementation / `create_weights` 实现
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
        Use the ModelSlimLinearScheme associated with each layer to create
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
**EN:** This block defines method `create_weights` on `ModelSlimLinearMethod`. Use the ModelSlimLinearScheme associated with each layer to create the necessary parameters for the layer. See LinearMethodBase for param details Key calls include `extra_weight_attrs.get`, and `layer.scheme.create_weights`. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimLinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `extra_weight_attrs.get` 和 `layer.scheme.create_weights`。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 228-244: `apply` implementation / `apply` 实现
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
**EN:** This block defines method `apply` on `ModelSlimLinearMethod`. Use the output of create_weights and the CompressedTensorsScheme associated with the layer to apply the forward pass with the layer input. See LinearMethodBase for param details Key calls include `scheme.apply_weights`, and `ValueError`. The implementation branches on conditions. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `ModelSlimLinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `scheme.apply_weights` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `ModelSlimConfig`: Config class for ModelSlim Quantization of Diffusion models https://gitcode.com/Ascend/msmodelslim, a NPU-specific quantization type. / 核心类，用于封装 model slim config 相关行为。
- `ModelSlimLinearMethod`: Primary class that encapsulates model slim linear method behavior. / 核心类，用于封装 model slim linear method 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `types`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.srt.layers.quantization.compressed_tensors.utils`, `sglang.srt.layers.quantization.modelslim.schemes`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.layers.quantization.modelslim_mxfp8_scheme`

- **Total lines / 总行数**: 244
