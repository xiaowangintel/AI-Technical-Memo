# awq.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/awq.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AWQConfig`, `AWQLinearMethod` for quantization backends, schemes, and utilities. / 实现 `AWQConfig`, `AWQLinearMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-25)
```python
from typing import TYPE_CHECKING, Any, Union

import torch
from safetensors.torch import _TYPES as _SAFETENSORS_TO_TORCH_DTYPE
from transformers import PretrainedConfig

from vllm import _custom_ops as ops
from vllm import envs
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe import RoutedExperts
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import is_layer_skipped
from vllm.model_executor.parameter import GroupQuantScaleParameter, PackedvLLMParameter
from vllm.transformers_utils.config import get_safetensors_params_metadata
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch`, `safetensors`, `transformers` and internal modules such as `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils.quant_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`, `safetensors`, `transformers`）以及内部模块（如 `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils.quant_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 31-31)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Class `AWQConfig` overview (lines 34-169)
```python
class AWQConfig(QuantizationConfig):
    """Config class for AWQ.

    Reference: https://arxiv.org/abs/2306.00978
    """

    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        zero_point: bool,
        modules_to_not_convert: list[str] | None = None,
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
**EN:** Defines class `AWQConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 10 direct methods, with notable entries `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`. Its docstring says: Config class for AWQ.
**CN:** 定义类 `AWQConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 10 个方法，较重要的包括 `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`。 文档字符串进一步说明了该类的定位。

### Method `AWQConfig.__init__` (lines 40-58)
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        zero_point: bool,
        modules_to_not_convert: list[str] | None = None,
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
**EN:** Defines function `AWQConfig.__init__` with signature `__init__(self, weight_bits: int, group_size: int, zero_point: bool, modules_to_not_convert: list[str] | None=None) -> None`. It mainly works with `weight_bits`, `group_size`, `zero_point`, `modules_to_not_convert`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `ValueError`, `super`.
**CN:** 定义函数 `AWQConfig.__init__`，其签名为 `__init__(self, weight_bits: int, group_size: int, zero_point: bool, modules_to_not_convert: list[str] | None=None) -> None`。它主要围绕 `weight_bits`, `group_size`, `zero_point`, `modules_to_not_convert` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `ValueError`, `super`。

### Method `AWQConfig.get_min_capability` (lines 75-77)
```python
    def get_min_capability(cls) -> int:
        # The AWQ kernel only supports Turing or newer GPUs.
        return 75
```
**EN:** Defines function `AWQConfig.get_min_capability` with signature `get_min_capability(cls) -> int`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `AWQConfig.get_min_capability`，其签名为 `get_min_capability(cls) -> int`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `AWQConfig.get_config_filenames` (lines 80-85)
```python
    def get_config_filenames() -> list[str]:
        return [
            "quant_config.json",  # E.g., casperhansen/vicuna-7b-v1.5-awq
            # E.g., abhinavkulkarni/mosaicml-mpt-7b-instruct-w4-g128-awq
            "quantize_config.json",
        ]
```
**EN:** Defines function `AWQConfig.get_config_filenames` with signature `get_config_filenames() -> list[str]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `AWQConfig.get_config_filenames`，其签名为 `get_config_filenames() -> list[str]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `AWQConfig.from_config` (lines 88-95)
```python
    def from_config(cls, config: dict[str, Any]) -> "AWQConfig":
        weight_bits = cls.get_from_keys(config, ["w_bit", "bits"])
        group_size = cls.get_from_keys(config, ["q_group_size", "group_size"])
        zero_point = cls.get_from_keys(config, ["zero_point"])
        modules_to_not_convert = cls.get_from_keys_or(
            config, ["modules_to_not_convert"], None
        )
        return cls(weight_bits, group_size, zero_point, modules_to_not_convert)
```
**EN:** Defines function `AWQConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'AWQConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`.
**CN:** 定义函数 `AWQConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'AWQConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`。

### Method `AWQConfig.get_quant_method` (lines 97-143)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Union["LinearMethodBase", "QuantizeMethodBase"] | None:
        if isinstance(layer, LinearBase):
            if is_layer_skipped(
                prefix,
                self.modules_to_not_convert,
                self.packed_modules_mapping,
                skip_with_substr=True,
            ):
                return UnquantizedLinearMethod()
            return AWQLinearMethod(self)
        elif isinstance(layer, RoutedExperts):
            # Lazy import to avoid circular import.
            from .awq_marlin import AWQMarlinConfig
            from .moe_wna16 import MoeWNA16Config
            from .utils.marlin_utils import check_moe_marlin_supports_layer

            if not check_moe_marlin_supports_layer(layer, self.group_size):
                logger.warning_once(
                    f"Layer '{prefix}' is not supported by AWQMoeMarlin. "
                    "Falling back to Moe WNA16 kernels."
                )
                config = {
                    "quant_method": "awq",
                    "bits": self.weight_bits,
                    "group_size": self.group_size,
                    "zero_point": self.zero_point,
                    "lm_head": False,
                    "modules_to_not_convert": self.modules_to_not_convert,
                }
                return MoeWNA16Config.from_config(config).get_quant_method(
                    layer, prefix
                )
            marlin_compatible_config_dict = {
                "quant_method": "awq",
                "bits": self.weight_bits,
                "group_size": self.group_size,
                "zero_point": self.zero_point,
                "lm_head": False,
                "modules_to_not_convert": self.modules_to_not_convert,
            }
            awq_marlin_config = AWQMarlinConfig.from_config(
                marlin_compatible_config_dict
            )
            return awq_marlin_config.get_quant_method(layer, prefix)
        return None
```
**EN:** Defines function `AWQConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> Union['LinearMethodBase', 'QuantizeMethodBase'] | None`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching. Key calls include `isinstance`, `is_layer_skipped`, `AWQLinearMethod`, `UnquantizedLinearMethod`, `AWQMarlinConfig.from_config`, `awq_marlin_config.get_quant_method`.
**CN:** 定义函数 `AWQConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> Union['LinearMethodBase', 'QuantizeMethodBase'] | None`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `isinstance`, `is_layer_skipped`, `AWQLinearMethod`, `UnquantizedLinearMethod`, `AWQMarlinConfig.from_config`, `awq_marlin_config.get_quant_method`。

### Method `AWQConfig.apply_vllm_mapper` (lines 145-149)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper: "WeightsMapper"):
        if self.modules_to_not_convert:
            self.modules_to_not_convert = hf_to_vllm_mapper.apply_list(
                self.modules_to_not_convert
            )
```
**EN:** Defines function `AWQConfig.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `hf_to_vllm_mapper.apply_list`.
**CN:** 定义函数 `AWQConfig.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `hf_to_vllm_mapper.apply_list`。

### Class `AWQLinearMethod` overview (lines 172-286)
```python
class AWQLinearMethod(LinearMethodBase):
    """Linear method for AWQ.

    Args:
        quant_config: The AWQ quantization config.
    """

    def __init__(self, quant_config: AWQConfig):
        self.quant_config = quant_config

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
        # Normalize group_size
        if self.quant_config.group_size != -1:
            group_size = self.quant_config.group_size
        else:
            group_size = input_size
```
**EN:** Defines class `AWQLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for AWQ.
**CN:** 定义类 `AWQLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `AWQLinearMethod.__init__` (lines 179-180)
```python
    def __init__(self, quant_config: AWQConfig):
        self.quant_config = quant_config
```
**EN:** Defines function `AWQLinearMethod.__init__` with signature `__init__(self, quant_config: AWQConfig)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `AWQLinearMethod.__init__`，其签名为 `__init__(self, quant_config: AWQConfig)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `AWQLinearMethod.create_weights` (lines 182-255)
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
        # Normalize group_size
        if self.quant_config.group_size != -1:
            group_size = self.quant_config.group_size
        else:
            group_size = input_size

        if input_size_per_partition % group_size != 0:
            raise ValueError(
                "The input size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
            )

        output_size_per_partition = sum(output_partition_sizes)
        if output_size_per_partition % self.quant_config.pack_factor != 0:
            raise ValueError(
                "The output size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
            )

        weight_loader = extra_weight_attrs.get("weight_loader")
        qweight = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition,
                output_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
# ... truncated for analysis ...
        scales = GroupQuantScaleParameter(
            data=torch.empty(
                num_groups,
                output_size_per_partition,
                dtype=params_dtype,
            ),
            input_dim=0,
            output_dim=1,
            weight_loader=weight_loader,
        )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("qzeros", qzeros)
        layer.register_parameter("scales", scales)
```
**EN:** Defines function `AWQLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `PackedvLLMParameter`, `GroupQuantScaleParameter`, `layer.register_parameter`, `ValueError`.
**CN:** 定义函数 `AWQLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `PackedvLLMParameter`, `GroupQuantScaleParameter`, `layer.register_parameter`, `ValueError`。

### Method `AWQLinearMethod.process_weights_after_loading` (lines 257-260)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.qweight = torch.nn.Parameter(layer.qweight.data, requires_grad=False)
        layer.qzeros = torch.nn.Parameter(layer.qzeros.data, requires_grad=False)
        layer.scales = torch.nn.Parameter(layer.scales.data, requires_grad=False)
```
**EN:** Defines function `AWQLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses tensor/kernel operations. Key calls include `torch.nn.Parameter`.
**CN:** 定义函数 `AWQLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含张量或内核操作。关键调用包括 `torch.nn.Parameter`。

### Method `AWQLinearMethod.apply` (lines 262-286)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        qweight = layer.qweight
        scales = layer.scales
        qzeros = layer.qzeros
        pack_factor = self.quant_config.pack_factor
        out_shape = x.shape[:-1] + (qweight.shape[-1] * pack_factor,)
        reshaped_x = x.reshape(-1, x.shape[-1])

        # num_tokens >= threshold
        FP16_MATMUL_HEURISTIC_CONDITION = x.shape[:-1].numel() >= 256
        # Batch invariant mode requires torch.matmul path
        # for Triton override
        if FP16_MATMUL_HEURISTIC_CONDITION or envs.VLLM_BATCH_INVARIANT:
            out = ops.awq_dequantize(qweight, scales, qzeros, 0, 0, 0)
            out = torch.matmul(reshaped_x, out)
        else:
            out = ops.awq_gemm(reshaped_x, qweight, scales, qzeros, pack_factor)
        if bias is not None:
            out.add_(bias)
        return out.reshape(out_shape)
```
**EN:** Defines function `AWQLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, tensor/kernel operations. Key calls include `x.reshape`, `out.reshape`, `x.shape.numel`, `ops.awq_dequantize`, `torch.matmul`, `ops.awq_gemm`.
**CN:** 定义函数 `AWQLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `x.reshape`, `out.reshape`, `x.shape.numel`, `ops.awq_dequantize`, `torch.matmul`, `ops.awq_gemm`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `AWQConfig`, `AWQLinearMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `AWQConfig`, `AWQLinearMethod`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`, `safetensors`, `transformers`
- **Internal / 内部**: `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`, `vllm.transformers_utils.config`
