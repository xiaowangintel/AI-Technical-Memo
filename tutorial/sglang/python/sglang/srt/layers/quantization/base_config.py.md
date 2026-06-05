# base_config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/base_config.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for base config quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 基础 配置 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://raw.githubusercontent.com/vllm-project/vllm/v0.5.5/vllm/model_executor/layers/quantization/base_config.py
from __future__ import annotations

import inspect
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Type

import torch
from torch import nn
```
**EN:** This block imports __future__, abc, inspect, torch, sglang.srt.layers.moe.moe_runner, sglang.srt.layers.moe.moe_runner.triton, sglang.srt.layers.moe.token_dispatcher, sglang.srt.models.utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, abc, inspect, torch, sglang.srt.layers.moe.moe_runner, sglang.srt.layers.moe.moe_runner.triton, sglang.srt.layers.moe.token_dispatcher, sglang.srt.models.utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 13-17: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.moe_runner import MoeRunnerConfig
    from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
    from sglang.srt.layers.moe.token_dispatcher import CombineInput, DispatchOutput
    from sglang.srt.models.utils import WeightsMapper
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 20-22: class QuantizeMethodBase: definition
```python
class QuantizeMethodBase(ABC):
    """Base class for different quantized methods."""
```
**EN:** This block declares `QuantizeMethodBase`, a supporting class for the quantization stack. It organizes behaviors such as create_weights, apply, process_weights_after_loading.
**CN:** 该代码块声明 `QuantizeMethodBase`，它是量化栈中的支撑类，组织了 create_weights, apply, process_weights_after_loading 等行为。

### Lines 23-29: QuantizeMethodBase.create_weights()
```python
    def create_weights(
        self, layer: torch.nn.Module, *weight_args, **extra_weight_attrs
    ):
        """Create weights for a layer.

        The weights will be set as attributes of the layer."""
        raise NotImplementedError()
```
**EN:** This block defines `QuantizeMethodBase.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `QuantizeMethodBase.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 31-36: QuantizeMethodBase.apply()
```python
    @abstractmethod
    def apply(self, layer: torch.nn.Module, *args, **kwargs) -> torch.Tensor:
        """Apply the weights in layer to the input tensor.

        Expects create_weights to have been called before on the layer."""
        raise NotImplementedError()
```
**EN:** This block defines `QuantizeMethodBase.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuantizeMethodBase.apply()`，用于将量化计算应用到运行时输入上。

### Lines 38-43: QuantizeMethodBase.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: nn.Module) -> None:
        """Process the weight after loading.

        This can be used for example, to transpose weights for computation.
        """
        return
```
**EN:** This block defines `QuantizeMethodBase.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `QuantizeMethodBase.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 46-48: class LinearMethodBase: definition
```python
class LinearMethodBase(QuantizeMethodBase):
    """Base class for different (maybe quantized) linear methods."""
```
**EN:** This block declares `LinearMethodBase`, a runtime method class for the quantization stack. It organizes behaviors such as create_weights, apply.
**CN:** 该代码块声明 `LinearMethodBase`，它是量化栈中的运行方法类，组织了 create_weights, apply 等行为。

### Lines 49-72: LinearMethodBase.create_weights()
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
        """Create weights for a linear layer.
           The weights will be set as attributes of the layer.

        Args:
            layer: The layer that is using the LinearMethodBase factory.
            input_size_per_partition: Size of the weight input dim on rank X.
            output_partition_sizes: Sizes of the output dim of each logical
                weight on rank X. E.g., output_partition_sizes for QKVLinear
                is a list contains the width of Wq, Wk, Wv on rank X.
            input_size: Size of the input dim of the weight across all ranks.
            output_size: Size of the output dim of the weight across all ranks.
            params_dtype: Datatype of the parameters.
        """
        raise NotImplementedError()
```
**EN:** This block defines `LinearMethodBase.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `LinearMethodBase.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 74-83: LinearMethodBase.apply()
```python
    @abstractmethod
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        """Apply the weights in layer to the input tensor.
        Expects create_weights to have been called before on the layer."""
        raise NotImplementedError()
```
**EN:** This block defines `LinearMethodBase.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `LinearMethodBase.apply()`，用于将量化计算应用到运行时输入上。

### Lines 86-87: class FusedMoEMethodBase: definition
```python
class FusedMoEMethodBase(QuantizeMethodBase):
```
**EN:** This block declares `FusedMoEMethodBase`, a runtime method class for the quantization stack. It organizes behaviors such as create_weights, create_moe_runner, apply, get_triton_quant_info.
**CN:** 该代码块声明 `FusedMoEMethodBase`，它是量化栈中的运行方法类，组织了 create_weights, create_moe_runner, apply, get_triton_quant_info 等行为。

### Lines 88-97: FusedMoEMethodBase.create_weights()
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
        raise NotImplementedError
```
**EN:** This block defines `FusedMoEMethodBase.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `FusedMoEMethodBase.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 99-102: FusedMoEMethodBase.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        raise NotImplementedError
```
**EN:** This block defines `FusedMoEMethodBase.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `FusedMoEMethodBase.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 104-110: FusedMoEMethodBase.apply()
```python
    @abstractmethod
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: DispatchOutput,
    ) -> CombineInput:
        raise NotImplementedError
```
**EN:** This block defines `FusedMoEMethodBase.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `FusedMoEMethodBase.apply()`，用于将量化计算应用到运行时输入上。

### Lines 112-123: FusedMoEMethodBase.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer: torch.nn.Module) -> "TritonMoeQuantInfo":
        """Return a ``TritonMoeQuantInfo`` describing the quantisation state
        stored on *layer*.

        The LoRA MoE runner calls this so that ``invoke_fused_moe_kernel``
        receives the correct flags / scales / block-shape for the base
        weights.  Each quantisation method must override this with the
        same construction it already uses inside ``apply()``.
        """
        raise NotImplementedError(
            f"{type(self).__name__} must implement get_triton_quant_info()"
        )
```
**EN:** This block defines `FusedMoEMethodBase.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `FusedMoEMethodBase.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 126-128: class QuantizationConfig: definition
```python
class QuantizationConfig(ABC):
    """Base class for quantization configs."""
```
**EN:** This block declares `QuantizationConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_name, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `QuantizationConfig`，它是量化栈中的配置类，组织了 __init__, get_name, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 129-132: QuantizationConfig.__init__()
```python
    def __init__(self):
        super().__init__()
        # mapping is updated by models as they initialize
        self.packed_modules_mapping: Dict[str, List[str]] = dict()
```
**EN:** This block defines `QuantizationConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `QuantizationConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 134-137: QuantizationConfig.get_name()
```python
    @abstractmethod
    def get_name(self) -> str:
        """Name of the quantization method."""
        raise NotImplementedError()
```
**EN:** This block defines `QuantizationConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 139-142: QuantizationConfig.get_supported_act_dtypes()
```python
    @abstractmethod
    def get_supported_act_dtypes(self) -> List[torch.dtype]:
        """List of supported activation dtypes."""
        raise NotImplementedError()
```
**EN:** This block defines `QuantizationConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 144-153: QuantizationConfig.get_min_capability()
```python
    @classmethod
    @abstractmethod
    def get_min_capability(cls) -> int:
        """Minimum GPU capability to support the quantization method.

        E.g., 70 for Volta, 75 for Turing, 80 for Ampere.
        This requirement is due to the custom CUDA kernels used by the
        quantization method.
        """
        raise NotImplementedError()
```
**EN:** This block defines `QuantizationConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 155-159: QuantizationConfig.get_config_filenames()
```python
    @staticmethod
    @abstractmethod
    def get_config_filenames() -> List[str]:
        """List of filenames to search for in the model directory."""
        raise NotImplementedError()
```
**EN:** This block defines `QuantizationConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 161-165: QuantizationConfig.from_config()
```python
    @classmethod
    @abstractmethod
    def from_config(cls, config: Dict[str, Any]) -> "QuantizationConfig":
        """Create a config class from the model's quantization config."""
        raise NotImplementedError()
```
**EN:** This block defines `QuantizationConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `QuantizationConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 167-175: QuantizationConfig.override_quantization_method()
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_cfg, user_quant) -> Optional[str]:
        """
        Detects if this quantization method can support a given checkpoint
        format by overriding the user specified quantization method --
        this method should only be overwritten by subclasses in exceptional
        circumstances
        """
        return None
```
**EN:** This block defines `QuantizationConfig.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `QuantizationConfig.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 177-202: QuantizationConfig._modelopt_override_quantization_method()
```python
    @classmethod
    def _modelopt_override_quantization_method(
        cls, hf_quant_config, user_quant
    ) -> Optional[str]:
        """Shared ModelOpt quantization method override logic."""
        if hf_quant_config is None:
            return None

        # Check if this is a ModelOpt config
        quant_algo = hf_quant_config.get("quant_algo", "").upper()

        # If user specified generic "modelopt", auto-detect the specific method
        if user_quant == "modelopt":
            if "FP8" in quant_algo:
                return "modelopt_fp8"
            elif "NVFP4" in quant_algo or "FP4" in quant_algo:
                return "modelopt_fp4"

        # The hf_quant_config may be a parsed quant config, so we need to check the
        # quant_method.
        if hf_quant_config.get("quant_method", "") == "modelopt_fp8":
            return "modelopt_fp8"
        elif hf_quant_config.get("quant_method", "") == "modelopt_fp4":
            return "modelopt_fp4"

        return None
```
**EN:** This block defines `QuantizationConfig._modelopt_override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `QuantizationConfig._modelopt_override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 204-212: QuantizationConfig.get_from_keys()
```python
    @staticmethod
    def get_from_keys(config: Dict[str, Any], keys: List[str]) -> Any:
        """Get a value from the model's quantization config."""
        for key in keys:
            if key in config:
                return config[key]
        raise ValueError(
            f"Cannot find any of {keys} in the model's " "quantization config."
        )
```
**EN:** This block defines `QuantizationConfig.get_from_keys()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_from_keys()`，用于为调用方获取或计算派生值。

### Lines 214-220: QuantizationConfig.get_from_keys_or()
```python
    @staticmethod
    def get_from_keys_or(config: Dict[str, Any], keys: List[str], default: Any) -> Any:
        """Get a optional value from the model's quantization config."""
        try:
            return QuantizationConfig.get_from_keys(config, keys)
        except ValueError:
            return default
```
**EN:** This block defines `QuantizationConfig.get_from_keys_or()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_from_keys_or()`，用于为调用方获取或计算派生值。

### Lines 222-235: QuantizationConfig.get_quant_method()
```python
    @abstractmethod
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        """Get the quantize method to use for the quantized layer.

        Args:
            layer: The layer for the quant method.
            prefix: The full name of the layer in the state dict
        Returns:
            The quantize method. None if the given layer doesn't support quant
            method.
        """
        raise NotImplementedError()
```
**EN:** This block defines `QuantizationConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 237-243: QuantizationConfig.get_scaled_act_names()
```python
    @abstractmethod
    def get_scaled_act_names(self) -> List[str]:
        """Returns the activation function names that should be post-scaled.

        For now, this is only used by AWQ.
        """
        raise NotImplementedError()
```
**EN:** This block defines `QuantizationConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `QuantizationConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 245-254: QuantizationConfig.apply_weight_name_mapper()
```python
    def apply_weight_name_mapper(
        self, hf_to_sglang_mapper: "WeightsMapper"
    ):  # noqa: B027
        """
        Interface for models to update module names referenced in
        quantization configs in order to reflect the sglang model structure
        :param hf_to_sglang_mapper: maps from hf model structure (the assumed
            structure of the qconfig) to sglang model structure
        """
        pass
```
**EN:** This block defines `QuantizationConfig.apply_weight_name_mapper()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `QuantizationConfig.apply_weight_name_mapper()`，用于将量化计算应用到运行时输入上。

### Lines 257-266: method_has_implemented_embedding()
```python
def method_has_implemented_embedding(method_class: Type[QuantizeMethodBase]) -> bool:
    """
    Not all quant methods have embedding implemented, so we need to check that
    it exists for our given method. We check this by making sure the function
    has been changed from the base implementation.
    """
    base_embedding = inspect.getattr_static(QuantizeMethodBase, "embedding", None)
    class_embedding = inspect.getattr_static(method_class, "embedding", None)

    return class_embedding is not None and class_embedding is not base_embedding
```
**EN:** This block defines `method_has_implemented_embedding()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `method_has_implemented_embedding()`，用于实现量化栈中的可复用模块逻辑。

## Key Concepts / 关键概念
- `QuantizeMethodBase`: A supporting class that structures file-level quantization behavior. / `QuantizeMethodBase` 是一个支撑类，用于组织该文件中的量化行为。
- `LinearMethodBase`: A runtime method class that structures file-level quantization behavior. / `LinearMethodBase` 是一个运行方法类，用于组织该文件中的量化行为。
- `FusedMoEMethodBase`: A runtime method class that structures file-level quantization behavior. / `FusedMoEMethodBase` 是一个运行方法类，用于组织该文件中的量化行为。
- `QuantizationConfig`: A configuration class that structures file-level quantization behavior. / `QuantizationConfig` 是一个配置类，用于组织该文件中的量化行为。
- `method_has_implemented_embedding()` : A public function that implements reusable module logic for the quantization stack. / `method_has_implemented_embedding()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `abc`, `inspect`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.moe.moe_runner`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.models.utils`
