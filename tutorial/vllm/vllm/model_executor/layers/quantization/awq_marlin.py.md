# awq_marlin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/awq_marlin.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AWQMarlinConfig`, `AWQMarlinLinearMethod`, `AWQMarlinMoEMethod` for quantization backends, schemes, and utilities. / 实现 `AWQMarlinConfig`, `AWQMarlinLinearMethod`, `AWQMarlinMoEMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-60)
```python
from typing import TYPE_CHECKING, Any

import torch
from safetensors.torch import _TYPES as _SAFETENSORS_TO_TORCH_DTYPE
from torch.nn import Parameter
from transformers import PretrainedConfig

import vllm.model_executor.layers.fused_moe  # noqa
from vllm import _custom_ops as ops
from vllm import envs
from vllm.logger import init_logger
from vllm.model_executor.kernels.linear import (
    MPLinearLayerConfig,
    choose_mp_linear_kernel,
)
from vllm.model_executor.layers.fused_moe import (
    FusedMoEMethodBase,
    FusedMoeWeightScaleSupported,
    RoutedExperts,
    SharedExperts,
    UnquantizedFusedMoEMethod,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.experts.marlin_moe import fused_marlin_moe
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
    set_weight_attrs,
)
from vllm.model_executor.layers.quantization.awq import AWQConfig
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from vllm.model_executor.layers.quantization.utils import replace_parameter
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    check_marlin_supported,
    check_marlin_supports_layer,
    check_moe_marlin_supports_layer,
    get_marlin_input_dtype,
    marlin_act_int8_process_scales,
    marlin_make_workspace_new,
    marlin_moe_permute_scales,
    marlin_permute_bias,
    moe_awq_to_marlin_zero_points,
    verify_marlin_supported,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import is_layer_skipped
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.model_executor.parameter import GroupQuantScaleParameter, PackedvLLMParameter
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
from vllm.transformers_utils.config import get_safetensors_params_metadata
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch`, `safetensors`, `transformers` and internal modules such as `vllm.model_executor.layers.fused_moe`, `vllm`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.marlin_moe`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`, `safetensors`, `transformers`）以及内部模块（如 `vllm.model_executor.layers.fused_moe`, `vllm`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.marlin_moe`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 66-72)
```python
logger = init_logger(__name__)

# AWQ uses a non-standard packing order within int32 values.
# For 4-bit: standard order stores values at bit positions [0,4,8,12,16,20,24,28]
# for indices [0,1,2,3,4,5,6,7], while AWQ stores them for indices
# [0,4,1,5,2,6,3,7]. This permutation reverses that ordering.
_REVERSE_AWQ_PACK_ORDER = [0, 4, 1, 5, 2, 6, 3, 7]
```
**EN:** This block defines module-level metadata or constants such as `logger`, `_REVERSE_AWQ_PACK_ORDER`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `_REVERSE_AWQ_PACK_ORDER`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `_convert_awq_to_standard_format` (lines 75-150)
```python
def _convert_awq_to_standard_format(
    layer: torch.nn.Module,
    w_q_name: str,
    w_zp_name: str,
    size_bits: int,
) -> None:
    """Convert AWQ weight and zero-point tensors to standard GPTQ-like format.

    AWQ packs qweight along the output dim with a non-standard bit order.
    This converts to standard bit order and repacks qweight along the input
    dim, matching the format expected by the MPLinearKernel framework.
    """
    pack_factor = 32 // size_bits
    mask = (1 << size_bits) - 1
    device = getattr(layer, w_q_name).device
    reverse_order = torch.tensor(
        _REVERSE_AWQ_PACK_ORDER, dtype=torch.long, device=device
    )
    shifts = torch.arange(0, 32, size_bits, dtype=torch.int32, device=device)

    # --- Convert qweight: (K, N // pack) packed_dim=1 → (K // pack, N) packed_dim=0
    qw = getattr(layer, w_q_name).data
    K, N_packed = qw.shape
    N = N_packed * pack_factor

    # Unpack int32 → individual values, fix AWQ ordering
    unpacked = (qw.unsqueeze(-1) >> shifts) & mask  # (K, N_packed, pack_factor)
    unpacked = unpacked[:, :, reverse_order]
    unpacked = unpacked.reshape(K, N)  # (K, N)

    # Repack along input dim (dim 0)
    unpacked = unpacked.reshape(K // pack_factor, pack_factor, N)
    new_qw = (unpacked.to(torch.int32) << shifts[None, :, None]).sum(
        dim=1, dtype=torch.int32
    )

    def _noop_loader(*args, **kwargs):
        pass
# ... truncated for analysis ...
    unpacked_zp = unpacked_zp.reshape(N // pack_factor, pack_factor, G)
    new_qz = (unpacked_zp.to(torch.int32) << shifts[None, :, None]).sum(
        dim=1, dtype=torch.int32
    )

    new_zp_param = PackedvLLMParameter(
        data=new_qz.contiguous(),
        output_dim=0,
        input_dim=1,
        packed_dim=0,
        packed_factor=pack_factor,
        weight_loader=_noop_loader,
    )
    setattr(layer, w_zp_name, new_zp_param)
```
**EN:** Defines function `_convert_awq_to_standard_format` with signature `_convert_awq_to_standard_format(layer: torch.nn.Module, w_q_name: str, w_zp_name: str, size_bits: int) -> None`. It mainly works with `layer`, `w_q_name`, `w_zp_name`, `size_bits`; implements one step in the quantized-weight execution flow. The body uses tensor/kernel operations. Key calls include `torch.tensor`, `torch.arange`, `unpacked.reshape`, `sum`, `PackedvLLMParameter`, `setattr`.
**CN:** 定义函数 `_convert_awq_to_standard_format`，其签名为 `_convert_awq_to_standard_format(layer: torch.nn.Module, w_q_name: str, w_zp_name: str, size_bits: int) -> None`。它主要围绕 `layer`, `w_q_name`, `w_zp_name`, `size_bits` 展开；实现量化权重执行流程中的一个步骤。函数体包含张量或内核操作。关键调用包括 `torch.tensor`, `torch.arange`, `unpacked.reshape`, `sum`, `PackedvLLMParameter`, `setattr`。

### Class `AWQMarlinConfig` overview (lines 153-361)
```python
class AWQMarlinConfig(QuantizationConfig):
    """Config class for AWQ Marlin"""

    # num_bits -> type
    TYPE_MAP = {
        4: scalar_types.uint4,
    }

    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        zero_point: bool,
        lm_head_quantized: bool,
        modules_to_not_convert: list[str] | None,
        full_config: dict[str, Any],
    ) -> None:
        super().__init__()
        self.pack_factor = 32 // weight_bits  # packed into int32
        self.group_size = group_size
        self.zero_point = zero_point
        self.lm_head_quantized = lm_head_quantized
        self.weight_bits = weight_bits
        self.modules_to_not_convert = modules_to_not_convert or []
        self.full_config = full_config
```
**EN:** Defines class `AWQMarlinConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 12 direct methods, with notable entries `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`. Its docstring says: Config class for AWQ Marlin
**CN:** 定义类 `AWQMarlinConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 12 个方法，较重要的包括 `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`。 文档字符串进一步说明了该类的定位。

### Method `AWQMarlinConfig.__init__` (lines 161-189)
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        zero_point: bool,
        lm_head_quantized: bool,
        modules_to_not_convert: list[str] | None,
        full_config: dict[str, Any],
    ) -> None:
        super().__init__()
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
**EN:** Defines function `AWQMarlinConfig.__init__` with signature `__init__(self, weight_bits: int, group_size: int, zero_point: bool, lm_head_quantized: bool, modules_to_not_convert: list[str] | None, full_config: dict[str, Any]) -> None`. It mainly works with `weight_bits`, `group_size`, `zero_point`, `lm_head_quantized`, `modules_to_not_convert`, `full_config`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `verify_marlin_supported`, `ValueError`, `super`, `self.TYPE_MAP.keys`.
**CN:** 定义函数 `AWQMarlinConfig.__init__`，其签名为 `__init__(self, weight_bits: int, group_size: int, zero_point: bool, lm_head_quantized: bool, modules_to_not_convert: list[str] | None, full_config: dict[str, Any]) -> None`。它主要围绕 `weight_bits`, `group_size`, `zero_point`, `lm_head_quantized`, `modules_to_not_convert`, `full_config` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `verify_marlin_supported`, `ValueError`, `super`, `self.TYPE_MAP.keys`。

### Method `AWQMarlinConfig.from_config` (lines 217-232)
```python
    def from_config(cls, config: dict[str, Any]) -> "AWQMarlinConfig":
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
**EN:** Defines function `AWQMarlinConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'AWQMarlinConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`.
**CN:** 定义函数 `AWQMarlinConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'AWQMarlinConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`。

### Method `AWQMarlinConfig.get_quant_method` (lines 265-310)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> "QuantizeMethodBase | None":
        if isinstance(layer, LinearBase) or (
            isinstance(layer, ParallelLMHead) and self.lm_head_quantized
        ):
            if is_layer_skipped(
                prefix,
                self.modules_to_not_convert,
                self.packed_modules_mapping,
                skip_with_substr=True,
            ):
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
            quant_method = AWQMarlinLinearMethod(self)
            quant_method.input_dtype = get_marlin_input_dtype(prefix)
            return quant_method
        elif isinstance(layer, RoutedExperts):
            from vllm.model_executor.layers.quantization.moe_wna16 import MoeWNA16Config

            if is_layer_skipped(
                prefix,
                getattr(self, "modules_to_not_convert", []),
                skip_with_substr=True,
            ):
                return UnquantizedFusedMoEMethod(layer.moe_config)
            if not check_moe_marlin_supports_layer(layer, self.group_size):
                logger.warning_once(
                    f"Layer '{prefix}' is not supported by AWQMoeMarlin. "
                    "Falling back to Moe WNA16 kernels."
                )
                return MoeWNA16Config.from_config(self.full_config).get_quant_method(
                    layer, prefix
                )
            moe_quant_method = AWQMarlinMoEMethod(self, layer.moe_config)
            moe_quant_method.input_dtype = get_marlin_input_dtype(prefix)
            return moe_quant_method
        return None
```
**EN:** Defines function `AWQMarlinConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching. Key calls include `isinstance`, `is_layer_skipped`, `AWQMarlinLinearMethod`, `get_marlin_input_dtype`, `UnquantizedLinearMethod`, `check_marlin_supports_layer`.
**CN:** 定义函数 `AWQMarlinConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `isinstance`, `is_layer_skipped`, `AWQMarlinLinearMethod`, `get_marlin_input_dtype`, `UnquantizedLinearMethod`, `check_marlin_supports_layer`。

### Method `AWQMarlinConfig.apply_vllm_mapper` (lines 337-341)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper: "WeightsMapper"):
        if self.modules_to_not_convert:
            self.modules_to_not_convert = hf_to_vllm_mapper.apply_list(
                self.modules_to_not_convert
            )
```
**EN:** Defines function `AWQMarlinConfig.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `hf_to_vllm_mapper.apply_list`.
**CN:** 定义函数 `AWQMarlinConfig.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `hf_to_vllm_mapper.apply_list`。

### Class `AWQMarlinLinearMethod` overview (lines 364-492)
```python
class AWQMarlinLinearMethod(LinearMethodBase):
    """Linear method for AWQ Marlin.

    Uses choose_mp_linear_kernel to select the best available kernel
    (Conch, Exllama, or Marlin) for the current platform.

    Args:
        quant_config: The AWQ Marlin quantization config.
    """

    _kernel_backends_being_used: set[str] = set()

    def __init__(self, quant_config: AWQMarlinConfig) -> None:
        self.quant_config = quant_config
        self.quant_type = scalar_types.uint4
        self.input_dtype = None

        verify_marlin_supported(
            quant_type=self.quant_config.quant_type,
            group_size=self.quant_config.group_size,
            has_zp=self.quant_config.zero_point,
        )

    def create_weights(
        self,
```
**EN:** Defines class `AWQMarlinLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for AWQ Marlin.
**CN:** 定义类 `AWQMarlinLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `AWQMarlinLinearMethod.__init__` (lines 376-385)
```python
    def __init__(self, quant_config: AWQMarlinConfig) -> None:
        self.quant_config = quant_config
        self.quant_type = scalar_types.uint4
        self.input_dtype = None

        verify_marlin_supported(
            quant_type=self.quant_config.quant_type,
            group_size=self.quant_config.group_size,
            has_zp=self.quant_config.zero_point,
        )
```
**EN:** Defines function `AWQMarlinLinearMethod.__init__` with signature `__init__(self, quant_config: AWQMarlinConfig) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `verify_marlin_supported`.
**CN:** 定义函数 `AWQMarlinLinearMethod.__init__`，其签名为 `__init__(self, quant_config: AWQMarlinConfig) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `verify_marlin_supported`。

### Method `AWQMarlinLinearMethod.create_weights` (lines 387-474)
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
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        if self.quant_config.group_size != -1:
            group_size = self.quant_config.group_size
        else:
            group_size = input_size

        mp_linear_kernel_config = MPLinearLayerConfig(
            full_weight_shape=(input_size, output_size),
            partition_weight_shape=(
                input_size_per_partition,
                output_size_per_partition,
            ),
            weight_type=self.quant_config.quant_type,
            act_type=params_dtype if self.input_dtype is None else self.input_dtype,
            group_size=self.quant_config.group_size,
            zero_points=self.quant_config.zero_point,
            has_g_idx=False,
        )

        kernel_type = choose_mp_linear_kernel(mp_linear_kernel_config)

        if kernel_type.__name__ not in self._kernel_backends_being_used:
            logger.info("Using %s for AWQMarlinLinearMethod", kernel_type.__name__)
            self._kernel_backends_being_used.add(kernel_type.__name__)

        # Weights are loaded in AWQ checkpoint format (packed along output dim).
# ... truncated for analysis ...
            output_dim=1,
            weight_loader=weight_loader,
        )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("qzeros", qzeros)
        layer.register_parameter("scales", scales)

        self.kernel = kernel_type(
            mp_linear_kernel_config,
            w_q_param_name="qweight",
            w_s_param_name="scales",
            w_zp_param_name="qzeros",
        )
```
**EN:** Defines function `AWQMarlinLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs) -> None`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `MPLinearLayerConfig`, `choose_mp_linear_kernel`, `PackedvLLMParameter`, `GroupQuantScaleParameter`.
**CN:** 定义函数 `AWQMarlinLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs) -> None`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `MPLinearLayerConfig`, `choose_mp_linear_kernel`, `PackedvLLMParameter`, `GroupQuantScaleParameter`。

### Method `AWQMarlinLinearMethod.process_weights_after_loading` (lines 476-484)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # AWQ checkpoints use a non-standard packing order and pack qweight
        # along the output dimension. Convert to the standard format
        # (GPTQ-like: standard bit order, qweight packed along input dim)
        # before handing off to the kernel.
        _convert_awq_to_standard_format(
            layer, "qweight", "qzeros", self.quant_config.quant_type.size_bits
        )
        self.kernel.process_weights_after_loading(layer)
```
**EN:** Defines function `AWQMarlinLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses mostly straightforward data movement and object wiring. Key calls include `_convert_awq_to_standard_format`, `self.kernel.process_weights_after_loading`.
**CN:** 定义函数 `AWQMarlinLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `_convert_awq_to_standard_format`, `self.kernel.process_weights_after_loading`。

### Method `AWQMarlinLinearMethod.apply` (lines 486-492)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.kernel.apply_weights(layer, x, bias)
```
**EN:** Defines function `AWQMarlinLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.kernel.apply_weights`.
**CN:** 定义函数 `AWQMarlinLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.kernel.apply_weights`。

### Class `AWQMarlinMoEMethod` overview (lines 495-844)
```python
class AWQMarlinMoEMethod(FusedMoEMethodBase):
    def __init__(
        self,
        quant_config: AWQMarlinConfig,
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.quant_config = quant_config
        if self.quant_config.weight_bits != 4:
            raise ValueError("AWQMarlinMoEMethod only supports 4bit now.")
        self.quant_type = scalar_types.uint4
        self.input_dtype = None
        self.use_marlin = True

    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        layer.input_dtype = self.input_dtype
        extra_weight_attrs.update(
```
**EN:** Defines class `AWQMarlinMoEMethod` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 6 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `select_gemm_impl`, `apply`.
**CN:** 定义类 `AWQMarlinMoEMethod`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 6 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `select_gemm_impl`, `apply`。

### Method `AWQMarlinMoEMethod.__init__` (lines 496-507)
```python
    def __init__(
        self,
        quant_config: AWQMarlinConfig,
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.quant_config = quant_config
        if self.quant_config.weight_bits != 4:
            raise ValueError("AWQMarlinMoEMethod only supports 4bit now.")
        self.quant_type = scalar_types.uint4
        self.input_dtype = None
        self.use_marlin = True
```
**EN:** Defines function `AWQMarlinMoEMethod.__init__` with signature `__init__(self, quant_config: AWQMarlinConfig, moe: FusedMoEConfig)`. It mainly works with `quant_config`, `moe`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `ValueError`, `super`.
**CN:** 定义函数 `AWQMarlinMoEMethod.__init__`，其签名为 `__init__(self, quant_config: AWQMarlinConfig, moe: FusedMoEConfig)`。它主要围绕 `quant_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `ValueError`, `super`。

### Method `AWQMarlinMoEMethod.create_weights` (lines 509-608)
```python
    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        layer.input_dtype = self.input_dtype
        extra_weight_attrs.update(
            {
                "is_transposed": True,
                "quant_method": FusedMoeWeightScaleSupported.GROUP.value,
            }
        )

        intermediate_size_full = extra_weight_attrs.pop(
            "intermediate_size_full", intermediate_size_per_partition
        )
        self.is_k_full = intermediate_size_per_partition == intermediate_size_full

        w13_qweight = Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                2 * intermediate_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qweight", w13_qweight)
        set_weight_attrs(w13_qweight, extra_weight_attrs)

        w2_qweight = Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
# ... truncated for analysis ...
        w2_qzeros = Parameter(
            torch.empty(
                num_experts,
                num_groups_w2,
                hidden_size // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qzeros", w2_qzeros)
        set_weight_attrs(w2_qzeros, extra_weight_attrs)

        device = layer.w13_qweight.device
        layer.workspace = marlin_make_workspace_new(device, 4)
```
**EN:** Defines function `AWQMarlinMoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses tensor/kernel operations. Key calls include `extra_weight_attrs.update`, `extra_weight_attrs.pop`, `Parameter`, `layer.register_parameter`, `set_weight_attrs`, `marlin_make_workspace_new`.
**CN:** 定义函数 `AWQMarlinMoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含张量或内核操作。关键调用包括 `extra_weight_attrs.update`, `extra_weight_attrs.pop`, `Parameter`, `layer.register_parameter`, `set_weight_attrs`, `marlin_make_workspace_new`。

### Method `AWQMarlinMoEMethod.process_weights_after_loading` (lines 610-724)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        num_experts = layer.w13_qweight.shape[0]
        device = layer.w13_qweight.device
        is_a_8bit = self.input_dtype is not None and self.input_dtype.itemsize == 1

        if self.input_dtype == torch.float8_e4m3fn:
            ops.marlin_int4_fp8_preprocess(
                layer.w13_qweight.view(-1, layer.w13_qweight.size(2)),
                layer.w13_qzeros.view(-1, layer.w13_qzeros.size(2)),
                inplace=True,
            )
            ops.marlin_int4_fp8_preprocess(
                layer.w2_qweight.view(-1, layer.w2_qweight.size(2)),
                layer.w2_qzeros.view(-1, layer.w2_qzeros.size(2)),
                inplace=True,
            )
            layer.w13_scales.data = layer.w13_scales.data * 512
            layer.w2_scales.data = layer.w2_scales.data * 512

        layer.w13_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty((num_experts, 0), dtype=torch.int32, device=device),
            requires_grad=False,
        )
        layer.w2_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty((num_experts, 0), dtype=torch.int32, device=device),
            requires_grad=False,
        )

        marlin_w13_qweight = ops.awq_marlin_moe_repack(
            layer.w13_qweight,
            layer.w13_g_idx_sort_indices,
            size_k=layer.w13_qweight.shape[1],
            size_n=layer.w13_qweight.shape[2] * self.quant_config.pack_factor,
            num_bits=self.quant_config.weight_bits,
            is_a_8bit=is_a_8bit,
        )
        replace_parameter(layer, "w13_qweight", marlin_w13_qweight)

# ... truncated for analysis ...
        marlin_w2_zp = moe_awq_to_marlin_zero_points(
            layer.w2_qzeros,
            size_k=layer.w2_qzeros.shape[1],
            size_n=layer.w2_qzeros.shape[2] * self.quant_config.pack_factor,
            num_bits=self.quant_config.weight_bits,
            is_a_8bit=is_a_8bit,
        )
        replace_parameter(layer, "w2_qzeros", marlin_w2_zp)

        if hasattr(layer, "w13_bias") and layer.w13_bias is not None:
            layer.w13_bias.data = marlin_permute_bias(layer.w13_bias)

        if hasattr(layer, "w2_bias") and layer.w2_bias is not None:
            layer.w2_bias.data = marlin_permute_bias(layer.w2_bias)
```
**EN:** Defines function `AWQMarlinMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `ops.awq_marlin_moe_repack`, `replace_parameter`, `marlin_moe_permute_scales`, `moe_awq_to_marlin_zero_points`, `ops.marlin_int4_fp8_preprocess`.
**CN:** 定义函数 `AWQMarlinMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `ops.awq_marlin_moe_repack`, `replace_parameter`, `marlin_moe_permute_scales`, `moe_awq_to_marlin_zero_points`, `ops.marlin_int4_fp8_preprocess`。

### Method `AWQMarlinMoEMethod.apply` (lines 814-844)
```python
    def apply(
        self,
        layer: RoutedExperts,
        x: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        shared_experts: SharedExperts | None,
        shared_experts_input: torch.Tensor | None,
    ) -> torch.Tensor:
        return fused_marlin_moe(
            x,
            layer.w13_qweight,
            layer.w2_qweight,
            getattr(layer, "w13_bias", None),
            getattr(layer, "w2_bias", None),
            layer.w13_scales,
            layer.w2_scales,
            topk_weights,
            topk_ids,
            input_global_scale1=getattr(layer, "w13_input_global_scale", None),
            input_global_scale2=getattr(layer, "w2_input_global_scale", None),
            quant_type_id=self.quant_type.id,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            w1_zeros=layer.w13_qzeros,
            w2_zeros=layer.w2_qzeros,
            workspace=layer.workspace,
            input_dtype=self.input_dtype,
            inplace=not self.moe.disable_inplace,
        )
```
**EN:** Defines function `AWQMarlinMoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `fused_marlin_moe`, `getattr`.
**CN:** 定义函数 `AWQMarlinMoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `fused_marlin_moe`, `getattr`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `AWQMarlinConfig`, `AWQMarlinLinearMethod`, `AWQMarlinMoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `AWQMarlinConfig`, `AWQMarlinLinearMethod`, `AWQMarlinMoEMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `_convert_awq_to_standard_format` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_convert_awq_to_standard_format` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`, `safetensors`, `transformers`
- **Internal / 内部**: `vllm.model_executor.layers.fused_moe`, `vllm`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.marlin_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.awq`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.vocab_parallel_embedding`, `vllm.model_executor.parameter`, `vllm.platforms`, `vllm.scalar_type`, `vllm.transformers_utils.config`
