# fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Fp8Config`, `CopyNumelCounter`, `Fp8LinearMethod` for quantization backends, schemes, and utilities. / 实现 `Fp8Config`, `CopyNumelCounter`, `Fp8LinearMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-90)
```python
from typing import TYPE_CHECKING, Any

import torch
from torch.utils._python_dispatch import TorchDispatchMode

import vllm.envs as envs
import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import _custom_ops as ops
from vllm.config import get_current_vllm_config
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.logger import init_logger
from vllm.model_executor.kernels.linear import (
    init_fp8_linear_kernel,
)
from vllm.model_executor.kernels.linear.scaled_mm import (
    CutlassFP8ScaledMMLinearKernel,
    MarlinFP8ScaledMMLinearKernel,
)
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.fused_moe import (
    FusedMoEMethodBase,
    FusedMoeWeightScaleSupported,
    RoutedExperts,
    SharedExperts,
    UnquantizedFusedMoEMethod,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.oracle.fp8 import (
    Fp8MoeBackend,
    convert_to_fp8_moe_kernel_format,
    make_fp8_moe_kernel,
    make_fp8_moe_quant_config,
    select_fp8_moe_backend,
)
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
from vllm.model_executor.layers.quantization.kv_cache import BaseKVCacheMethod
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    create_fp8_input_scale,
    create_fp8_scale_parameter,
# ... truncated for analysis ...
    initialize_online_processing,
)
from vllm.model_executor.parameter import (
    BlockQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from vllm.model_executor.utils import replace_parameter, set_weight_attrs
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import (
    is_deep_gemm_supported,
)
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch` and internal modules such as `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.config`, `vllm.distributed`, `vllm.logger`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`）以及内部模块（如 `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.config`, `vllm.distributed`, `vllm.logger`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 95-97)
```python
ACTIVATION_SCHEMES = ["static", "dynamic"]

logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `ACTIVATION_SCHEMES`, `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `ACTIVATION_SCHEMES`, `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `_copy_missing_attrs` (lines 251-258)
```python
def _copy_missing_attrs(old: torch.Tensor, new: torch.Tensor) -> None:
    """Copies any attrs present in `old` but not in `new` to `new`"""
    new_attrs = set(dir(new))
    attrs_to_set = {}
    for attr in dir(old):
        if attr not in new_attrs:
            attrs_to_set[attr] = getattr(old, attr)
    set_weight_attrs(new, attrs_to_set)
```
**EN:** Defines function `_copy_missing_attrs` with signature `_copy_missing_attrs(old: torch.Tensor, new: torch.Tensor) -> None`. It mainly works with `old`, `new`; implements one step in the quantized-weight execution flow. The body uses branching, iteration. Key calls include `set`, `dir`, `set_weight_attrs`, `getattr`.
**CN:** 定义函数 `_copy_missing_attrs`，其签名为 `_copy_missing_attrs(old: torch.Tensor, new: torch.Tensor) -> None`。它主要围绕 `old`, `new` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `set`, `dir`, `set_weight_attrs`, `getattr`。

### Class `Fp8Config` overview (lines 100-228)
```python
class Fp8Config(QuantizationConfig):
    """Config class for FP8."""

    def __init__(
        self,
        is_checkpoint_fp8_serialized: bool = False,
        activation_scheme: str = "dynamic",
        ignored_layers: list[str] | None = None,
        weight_block_size: list[int] | None = None,
    ) -> None:
        super().__init__()

        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized

        if activation_scheme not in ACTIVATION_SCHEMES:
            raise ValueError(f"Unsupported activation scheme {activation_scheme}")
        self.activation_scheme = activation_scheme
        self.ignored_layers = ignored_layers or []
        if weight_block_size is not None:
            if not is_checkpoint_fp8_serialized:
                raise ValueError(
                    "The block-wise quantization only supports fp8-serialized "
                    "checkpoint for now."
                )
            if len(weight_block_size) != 2:
```
**EN:** Defines class `Fp8Config` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 9 direct methods, with notable entries `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`, `apply_vllm_mapper`. Its docstring says: Config class for FP8.
**CN:** 定义类 `Fp8Config`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 9 个方法，较重要的包括 `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`, `apply_vllm_mapper`。 文档字符串进一步说明了该类的定位。

### Method `Fp8Config.__init__` (lines 103-136)
```python
    def __init__(
        self,
        is_checkpoint_fp8_serialized: bool = False,
        activation_scheme: str = "dynamic",
        ignored_layers: list[str] | None = None,
        weight_block_size: list[int] | None = None,
    ) -> None:
        super().__init__()

        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized

        if activation_scheme not in ACTIVATION_SCHEMES:
            raise ValueError(f"Unsupported activation scheme {activation_scheme}")
        self.activation_scheme = activation_scheme
        self.ignored_layers = ignored_layers or []
        if weight_block_size is not None:
            if not is_checkpoint_fp8_serialized:
                raise ValueError(
                    "The block-wise quantization only supports fp8-serialized "
                    "checkpoint for now."
                )
            if len(weight_block_size) != 2:
                raise ValueError(
                    "The quantization block size of weight must have 2 "
                    f"dimensions, but got {len(weight_block_size)} dimensions"
                )
            if activation_scheme != "dynamic":
                raise ValueError(
                    "The block-wise quantization only supports "
                    "dynamic activation scheme for now, but got "
                    f"{activation_scheme} activation scheme."
                )
        self.weight_block_size = weight_block_size
        self.use_deep_gemm: bool | None = None
```
**EN:** Defines function `Fp8Config.__init__` with signature `__init__(self, is_checkpoint_fp8_serialized: bool=False, activation_scheme: str='dynamic', ignored_layers: list[str] | None=None, weight_block_size: list[int] | None=None) -> None`. It mainly works with `is_checkpoint_fp8_serialized`, `activation_scheme`, `ignored_layers`, `weight_block_size`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `ValueError`, `super`, `len`.
**CN:** 定义函数 `Fp8Config.__init__`，其签名为 `__init__(self, is_checkpoint_fp8_serialized: bool=False, activation_scheme: str='dynamic', ignored_layers: list[str] | None=None, weight_block_size: list[int] | None=None) -> None`。它主要围绕 `is_checkpoint_fp8_serialized`, `activation_scheme`, `ignored_layers`, `weight_block_size` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `ValueError`, `super`, `len`。

### Method `Fp8Config.apply_vllm_mapper` (lines 154-156)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper: "WeightsMapper"):
        if self.ignored_layers is not None:
            self.ignored_layers = hf_to_vllm_mapper.apply_list(self.ignored_layers)
```
**EN:** Defines function `Fp8Config.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `hf_to_vllm_mapper.apply_list`.
**CN:** 定义函数 `Fp8Config.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `hf_to_vllm_mapper.apply_list`。

### Method `Fp8Config.from_config` (lines 159-174)
```python
    def from_config(cls, config: dict[str, Any]) -> "Fp8Config":
        quant_method = cls.get_from_keys(config, ["quant_method"])
        is_checkpoint_fp8_serialized = "fp8" in quant_method
        activation_scheme = cls.get_from_keys(config, ["activation_scheme"])
        ignored_layers = cls.get_from_keys_or(config, ["ignored_layers"], None)
        weight_block_size = cls.get_from_keys_or(config, ["weight_block_size"], None)
        if not ignored_layers:
            ignored_layers = cls.get_from_keys_or(
                config, ["modules_to_not_convert"], None
            )
        return cls(
            is_checkpoint_fp8_serialized=is_checkpoint_fp8_serialized,
            activation_scheme=activation_scheme,
            ignored_layers=ignored_layers,
            weight_block_size=weight_block_size,
        )
```
**EN:** Defines function `Fp8Config.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'Fp8Config'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`.
**CN:** 定义函数 `Fp8Config.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'Fp8Config'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`。

### Method `Fp8Config.get_quant_method` (lines 176-208)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> "QuantizeMethodBase | None":
        if isinstance(layer, LinearBase):
            if is_layer_skipped(
                prefix=prefix,
                ignored_layers=self.ignored_layers,
                fused_mapping=self.packed_modules_mapping,
            ):
                return UnquantizedLinearMethod()
            if not self.is_checkpoint_fp8_serialized:
                online_method = Fp8OnlineLinearMethod(self)
                online_method.marlin_input_dtype = get_marlin_input_dtype(prefix)
                return online_method
            else:
                offline_method = Fp8LinearMethod(self)
                offline_method.marlin_input_dtype = get_marlin_input_dtype(prefix)
                return offline_method
        elif isinstance(layer, RoutedExperts):
            if is_layer_skipped(
                prefix=prefix,
                ignored_layers=self.ignored_layers,
                fused_mapping=self.packed_modules_mapping,
            ):
                return UnquantizedFusedMoEMethod(layer.moe_config)
            if self.is_checkpoint_fp8_serialized:
                moe_quant_method = Fp8MoEMethod(self, layer)
            else:
                moe_quant_method = Fp8OnlineMoEMethod(self, layer)
            return moe_quant_method
        elif isinstance(layer, Attention):
            return Fp8KVCacheMethod(self)
        return None
```
**EN:** Defines function `Fp8Config.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching. Key calls include `isinstance`, `is_layer_skipped`, `UnquantizedLinearMethod`, `Fp8OnlineLinearMethod`, `get_marlin_input_dtype`, `Fp8LinearMethod`.
**CN:** 定义函数 `Fp8Config.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `isinstance`, `is_layer_skipped`, `UnquantizedLinearMethod`, `Fp8OnlineLinearMethod`, `get_marlin_input_dtype`, `Fp8LinearMethod`。

### Class `CopyNumelCounter` overview (lines 231-248)
```python
class CopyNumelCounter(TorchDispatchMode):
    """
    Tracks total number of elements modified with `copy_`. Useful for keeping
    track of weight loading where underlying weights can be arbitrarily
    transformed (such as with `narrow`) before calling copy.
    """

    def __init__(self):
        super().__init__()
        self.copied_numel = 0

    def __torch_dispatch__(self, func, types, args=(), kwargs=None):
        if kwargs is None:
            kwargs = {}
        out = func(*args, **kwargs)
        if func == torch.ops.aten.copy_.default:
            self.copied_numel += args[0].numel()
        return out
```
**EN:** Defines class `CopyNumelCounter` with base classes `TorchDispatchMode` and decorators none. It acts as a quantization-oriented module building block and exposes 2 direct methods, with notable entries `__init__`, `__torch_dispatch__`. Its docstring says: Tracks total number of elements modified with `copy_`.
**CN:** 定义类 `CopyNumelCounter`，其基类为 `TorchDispatchMode`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 2 个方法，较重要的包括 `__init__`, `__torch_dispatch__`。 文档字符串进一步说明了该类的定位。

### Method `CopyNumelCounter.__init__` (lines 238-240)
```python
    def __init__(self):
        super().__init__()
        self.copied_numel = 0
```
**EN:** Defines function `CopyNumelCounter.__init__` with signature `__init__(self)`. It mainly works with object context only; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `CopyNumelCounter.__init__`，其签名为 `__init__(self)`。它主要围绕 仅依赖对象上下文 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `CopyNumelCounter.__torch_dispatch__` (lines 242-248)
```python
    def __torch_dispatch__(self, func, types, args=(), kwargs=None):
        if kwargs is None:
            kwargs = {}
        out = func(*args, **kwargs)
        if func == torch.ops.aten.copy_.default:
            self.copied_numel += args[0].numel()
        return out
```
**EN:** Defines function `CopyNumelCounter.__torch_dispatch__` with signature `__torch_dispatch__(self, func, types, args=(), kwargs=None)`. It mainly works with `func`, `types`, `args`, `kwargs`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `func`, `args.numel`.
**CN:** 定义函数 `CopyNumelCounter.__torch_dispatch__`，其签名为 `__torch_dispatch__(self, func, types, args=(), kwargs=None)`。它主要围绕 `func`, `types`, `args`, `kwargs` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `func`, `args.numel`。

### Class `Fp8LinearMethod` overview (lines 261-483)
```python
class Fp8LinearMethod(LinearMethodBase):
    """Linear method for FP8.
    Supports loading FP8 checkpoints with static weight scale and
    dynamic/static activation scale.

    Limitations:
    1. Only support float8_e4m3fn data type due to the limitation of
       torch._scaled_mm (https://github.com/pytorch/pytorch/blob/2e48b39603411a41c5025efbe52f89560b827825/aten/src/ATen/native/cuda/Blas.cpp#L854-L856)

    Args:
        quant_config: The quantization config.
    """

    def __init__(self, quant_config: Fp8Config):
        self.quant_config = quant_config
        self.is_scale_e8m0 = getattr(quant_config, "is_scale_e8m0", False)
        self.cutlass_block_fp8_supported = cutlass_block_fp8_supported()
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype

        # For GPUs that lack FP8 hardware support, we can leverage the Marlin
        # kernel for fast weight-only FP8 quantization
        self.marlin_input_dtype = None
        self.use_marlin = False
```
**EN:** Defines class `Fp8LinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for FP8.
**CN:** 定义类 `Fp8LinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `Fp8LinearMethod.__init__` (lines 274-314)
```python
    def __init__(self, quant_config: Fp8Config):
        self.quant_config = quant_config
        self.is_scale_e8m0 = getattr(quant_config, "is_scale_e8m0", False)
        self.cutlass_block_fp8_supported = cutlass_block_fp8_supported()
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype

        # For GPUs that lack FP8 hardware support, we can leverage the Marlin
        # kernel for fast weight-only FP8 quantization
        self.marlin_input_dtype = None
        self.use_marlin = False

        if self.quant_config.use_deep_gemm is not None:
            self.use_deep_gemm = self.quant_config.use_deep_gemm
        else:
            self.use_deep_gemm = is_deep_gemm_supported()

        self.weight_block_size = self.quant_config.weight_block_size
        self.block_quant = self.weight_block_size is not None
        self.act_q_static = self.quant_config.activation_scheme == "static"

        if self.block_quant:
            assert not self.act_q_static
            assert self.weight_block_size is not None

            self.activation_quant_key = create_fp8_quant_key(
                static=self.act_q_static,
                group_shape=GroupShape(1, self.weight_block_size[0]),
            )
            self.weight_quant_key = create_fp8_quant_key(
                static=True, group_shape=GroupShape(*self.weight_block_size)
            )
        else:
            self.weight_quant_key = kFp8StaticTensorSym
            # Use per-token quantization for better perf if dynamic and cutlass
            if self.act_q_static:
                self.activation_quant_key = kFp8StaticTensorSym
            elif cutlass_fp8_supported():
                self.activation_quant_key = kFp8DynamicTokenSym
            else:
                self.activation_quant_key = kFp8DynamicTensorSym
```
**EN:** Defines function `Fp8LinearMethod.__init__` with signature `__init__(self, quant_config: Fp8Config)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `getattr`, `cutlass_block_fp8_supported`, `torch.get_default_dtype`, `is_deep_gemm_supported`, `create_fp8_quant_key`, `get_current_vllm_config`.
**CN:** 定义函数 `Fp8LinearMethod.__init__`，其签名为 `__init__(self, quant_config: Fp8Config)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `getattr`, `cutlass_block_fp8_supported`, `torch.get_default_dtype`, `is_deep_gemm_supported`, `create_fp8_quant_key`, `get_current_vllm_config`。

### Method `Fp8LinearMethod.create_weights` (lines 316-390)
```python
    def create_weights(
        self,
        layer: RoutedExperts,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype
        layer.weight_block_size = None

        if self.block_quant:
            assert self.weight_block_size is not None
            layer.weight_block_size = self.weight_block_size
            validate_fp8_block_shape(
                layer,
                input_size,
                output_size,
                input_size_per_partition,
                output_partition_sizes,
                self.weight_block_size,
            )

        weight = create_fp8_weight_parameter(
            output_size_per_partition, input_size_per_partition, weight_loader
        )
        layer.register_parameter("weight", weight)

        # WEIGHT SCALE
        if not self.block_quant:
            scale = create_fp8_scale_parameter(
# ... truncated for analysis ...
            scale = create_fp8_input_scale(output_partition_sizes, weight_loader)
            set_weight_attrs(scale, {"scale_type": "input_scale"})
            layer.register_parameter("input_scale", scale)

        self.fp8_linear = init_fp8_linear_kernel(
            activation_quant_key=self.activation_quant_key,
            weight_quant_key=self.weight_quant_key,
            weight_shape=layer.weight.shape,
            input_dtype=self.input_dtype,
            out_dtype=self.out_dtype,
            module_name=self.__class__.__name__,
        )

        self.use_marlin = isinstance(self.fp8_linear, MarlinFP8ScaledMMLinearKernel)
```
**EN:** Defines function `Fp8LinearMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling. Key calls include `sum`, `extra_weight_attrs.get`, `create_fp8_weight_parameter`, `layer.register_parameter`, `init_fp8_linear_kernel`, `isinstance`.
**CN:** 定义函数 `Fp8LinearMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑。关键调用包括 `sum`, `extra_weight_attrs.get`, `create_fp8_weight_parameter`, `layer.register_parameter`, `init_fp8_linear_kernel`, `isinstance`。

### Method `Fp8LinearMethod.process_weights_after_loading` (lines 392-435)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        if self.use_marlin:
            # Only Marlin kernels support `marlin_input_dtype`; guard to avoid
            # AttributeError if backend selection changes.
            if hasattr(self.fp8_linear, "marlin_input_dtype"):
                self.fp8_linear.marlin_input_dtype = self.marlin_input_dtype
            self.fp8_linear.process_weights_after_loading(layer)
            return

        input_scale = None
        # TODO(rob): refactor block quant into separate class.
        if self.block_quant:
            assert not self.act_q_static

        # If checkpoint not serialized fp8, quantize the weights.
        else:
            # If checkpoint is fp8 per-tensor, handle that there are N scales for N
            # shards in a fused module
            weight = layer.weight
            weight_scale = layer.weight_scale

            # If using w8a8, torch._scaled_mm needs per tensor, so
            # requantize the logical shards as a single weight.
            weight, weight_scale, input_scale = process_fp8_weight_tensor_strategy(
                weight,
                weight_scale,
                layer.logical_widths,
                getattr(layer, "input_scale", None),
            )
            if self.act_q_static:
                assert input_scale is not None
                input_scale = input_scale.max()
            weight = weight.t()

            # Update layer with new values.
            replace_parameter(layer, "weight", weight.data)
            replace_parameter(layer, "weight_scale", weight_scale.data)

        if input_scale is not None:
            replace_parameter(layer, "input_scale", input_scale)
        else:
            layer.input_scale = None

        self.fp8_linear.process_weights_after_loading(layer)
```
**EN:** Defines function `Fp8LinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, validation/error handling. Key calls include `self.fp8_linear.process_weights_after_loading`, `hasattr`, `process_fp8_weight_tensor_strategy`, `weight.t`, `replace_parameter`, `getattr`.
**CN:** 定义函数 `Fp8LinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、校验或报错逻辑。关键调用包括 `self.fp8_linear.process_weights_after_loading`, `hasattr`, `process_fp8_weight_tensor_strategy`, `weight.t`, `replace_parameter`, `getattr`。

### Method `Fp8LinearMethod.apply` (lines 437-483)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # if batch invariant mode is enabled, prefer direct FP8 path
        # we will use BF16 dequant when direct FP8 is not supported.
        if envs.VLLM_BATCH_INVARIANT:
            if self.block_quant:
                assert self.weight_block_size is not None
                return self.fp8_linear.apply_weights(
                    layer,
                    x,
                    bias,
                )
            else:
                if isinstance(self.fp8_linear, CutlassFP8ScaledMMLinearKernel):
                    return self.fp8_linear.apply_weights(layer, x, bias)

                # per-tensor/channel: dequant to BF16 and run GEMM
                weight_fp8 = layer.weight.to(torch.bfloat16)
                weight_scale = layer.weight_scale.to(torch.bfloat16)
                if weight_scale.numel() == 1:
                    # Per-tensor: simple scalar multiplication
                    weight_bf16 = weight_fp8 * weight_scale
                else:
                    # Multiple scales (fused modules like QKV)
                    # Try to infer correct broadcasting
                    # weight is [K, N], scale could be [num_logical_weights]
                    # Need to figure out how to broadcast - for now just try
                    # direct multiplication
                    if (
                        weight_scale.dim() == 1
                        and weight_scale.shape[0] == weight_fp8.shape[0]
                    ):
                        # Per-row scaling
                        weight_bf16 = weight_fp8 * weight_scale.unsqueeze(1)
                    else:
                        # Fallback
                        weight_bf16 = weight_fp8 * weight_scale
                return torch.nn.functional.linear(x, weight_bf16.t(), bias)

        if self.use_marlin:
            return self.fp8_linear.apply_weights(layer, x, bias)

        return self.fp8_linear.apply_weights(layer, x, bias)
```
**EN:** Defines function `Fp8LinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `self.fp8_linear.apply_weights`, `isinstance`, `layer.weight.to`, `layer.weight_scale.to`, `torch.nn.functional.linear`, `weight_scale.numel`.
**CN:** 定义函数 `Fp8LinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `self.fp8_linear.apply_weights`, `isinstance`, `layer.weight.to`, `layer.weight_scale.to`, `torch.nn.functional.linear`, `weight_scale.numel`。

### Class `Fp8OnlineLinearMethod` overview (lines 488-562)
```python
class Fp8OnlineLinearMethod(Fp8LinearMethod):
    """Online version of Fp8LinearMethod which loads a full precision checkpoint
    and quantizes weights during loading."""

    uses_meta_device: bool = True

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
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype
        layer.weight_block_size = None

        weight = ModelWeightParameter(
```
**EN:** Defines class `Fp8OnlineLinearMethod` with base classes `Fp8LinearMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 2 direct methods, with notable entries `create_weights`, `process_weights_after_loading`. Its docstring says: Online version of Fp8LinearMethod which loads a full precision checkpoint and quantizes weights during loading.
**CN:** 定义类 `Fp8OnlineLinearMethod`，其基类为 `Fp8LinearMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 2 个方法，较重要的包括 `create_weights`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `Fp8OnlineLinearMethod.create_weights` (lines 494-535)
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
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype
        layer.weight_block_size = None

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                device="meta",  # materialized and processed during loading
                dtype=params_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        initialize_online_processing(layer)

        self.fp8_linear = init_fp8_linear_kernel(
            activation_quant_key=self.activation_quant_key,
            weight_quant_key=self.weight_quant_key,
            weight_shape=layer.weight.shape,
            input_dtype=self.input_dtype,
            out_dtype=self.out_dtype,
            module_name=self.__class__.__name__,
        )
        self.use_marlin = isinstance(self.fp8_linear, MarlinFP8ScaledMMLinearKernel)
```
**EN:** Defines function `Fp8OnlineLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `initialize_online_processing`, `init_fp8_linear_kernel`.
**CN:** 定义函数 `Fp8OnlineLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `initialize_online_processing`, `init_fp8_linear_kernel`。

### Method `Fp8OnlineLinearMethod.process_weights_after_loading` (lines 537-562)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        # TODO(future): support block_quant in online quant path
        assert not self.block_quant

        layer.input_scale = None
        qweight, weight_scale = ops.scaled_fp8_quant(layer.weight, scale=None)

        # Update layer with new values.
        replace_parameter(layer, "weight", qweight.data)
        replace_parameter(layer, "weight_scale", weight_scale.data)

        if self.use_marlin:
            # Only Marlin kernels support `marlin_input_dtype`; guard to avoid
            # AttributeError if backend selection changes.
            if hasattr(self.fp8_linear, "marlin_input_dtype"):
                self.fp8_linear.marlin_input_dtype = self.marlin_input_dtype
            self.fp8_linear.process_weights_after_loading(layer)
        else:
            weight = qweight.t()
            replace_parameter(layer, "weight", weight.data)

        # Prevent duplicate processing (e.g., during weight reload)
        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Fp8OnlineLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `getattr`, `ops.scaled_fp8_quant`, `replace_parameter`, `hasattr`, `self.fp8_linear.process_weights_after_loading`, `qweight.t`.
**CN:** 定义函数 `Fp8OnlineLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `getattr`, `ops.scaled_fp8_quant`, `replace_parameter`, `hasattr`, `self.fp8_linear.process_weights_after_loading`, `qweight.t`。

### Class `Fp8MoEMethod` overview (lines 565-925)
```python
class Fp8MoEMethod(FusedMoEMethodBase):
    """MoE method for FP8.
    Supports loading FP8 checkpoints with static weight scale and
    dynamic/static activation scale.

    Also supports loading quantized FP16/BF16 model checkpoints with dynamic
    activation scaling. The weight scaling factor will be initialized after
    the model weights are loaded.

    Args:
        quant_config: The quantization config.
    """

    def __init__(self, quant_config: Fp8Config, layer: RoutedExperts):
        super().__init__(layer.moe_config)
        self.quant_config = quant_config
        self.weight_block_size = self.quant_config.weight_block_size
        self.block_quant: bool = self.weight_block_size is not None
        self.weight_scale_name = (
            "weight_scale_inv" if self.block_quant else "weight_scale"
        )

        # Set weight key and activation key for kernel compatibility
        if self.block_quant:
            weight_key = kFp8Static128BlockSym
```
**EN:** Defines class `Fp8MoEMethod` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 9 direct methods, with notable entries `__init__`, `create_weights`, `_setup_kernel`, `process_weights_after_loading`, `maybe_make_prepare_finalize`, `get_fused_moe_quant_config`. Its docstring says: MoE method for FP8.
**CN:** 定义类 `Fp8MoEMethod`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 9 个方法，较重要的包括 `__init__`, `create_weights`, `_setup_kernel`, `process_weights_after_loading`, `maybe_make_prepare_finalize`, `get_fused_moe_quant_config`。 文档字符串进一步说明了该类的定位。

### Method `Fp8MoEMethod.__init__` (lines 578-605)
```python
    def __init__(self, quant_config: Fp8Config, layer: RoutedExperts):
        super().__init__(layer.moe_config)
        self.quant_config = quant_config
        self.weight_block_size = self.quant_config.weight_block_size
        self.block_quant: bool = self.weight_block_size is not None
        self.weight_scale_name = (
            "weight_scale_inv" if self.block_quant else "weight_scale"
        )

        # Set weight key and activation key for kernel compatibility
        if self.block_quant:
            weight_key = kFp8Static128BlockSym
            activation_key = kFp8Dynamic128Sym
        else:
            weight_key = kFp8StaticTensorSym
            activation_key = (
                kFp8StaticTensorSym
                if self.quant_config.activation_scheme == "static"
                else kFp8DynamicTensorSym
            )

        # Select Fp8 MoE backend
        self.fp8_backend, self.experts_cls = select_fp8_moe_backend(
            config=self.moe,
            weight_key=weight_key,
            activation_key=activation_key,
            allow_vllm_cutlass=False,
        )
```
**EN:** Defines function `Fp8MoEMethod.__init__` with signature `__init__(self, quant_config: Fp8Config, layer: RoutedExperts)`. It mainly works with `quant_config`, `layer`; initializes the object state and cached resources. The body uses branching. Key calls include `super.__init__`, `select_fp8_moe_backend`, `super`.
**CN:** 定义函数 `Fp8MoEMethod.__init__`，其签名为 `__init__(self, quant_config: Fp8Config, layer: RoutedExperts)`。它主要围绕 `quant_config`, `layer` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `super.__init__`, `select_fp8_moe_backend`, `super`。

### Method `Fp8MoEMethod.create_weights` (lines 607-745)
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
        layer.num_experts = num_experts
        layer.orig_dtype = params_dtype
        layer.weight_block_size = None

        assert self.quant_config.is_checkpoint_fp8_serialized
        params_dtype = torch.float8_e4m3fn

        if self.block_quant:
            assert self.weight_block_size is not None
            layer.weight_block_size = self.weight_block_size
            tp_size = get_tensor_model_parallel_world_size()
            block_n, block_k = (
                self.weight_block_size[0],
                self.weight_block_size[1],
            )
            # NOTE: To ensure proper alignment of the block-wise quantization
            # scales, the output_size of the weights for both the gate and up
            # layers must be divisible by block_n.
            # Required by column parallel or enabling merged weights
            if intermediate_size_per_partition % block_n != 0:
                raise ValueError(
                    f"The output_size of gate's and up's weight = "
                    f"{intermediate_size_per_partition} is not divisible by "
                    f"weight quantization block_n = {block_n}."
                )
            if tp_size > 1 and intermediate_size_per_partition % block_k != 0:
                # Required by row parallel
                raise ValueError(
                    f"The input_size of down's weight = "
# ... truncated for analysis ...
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
            )
            layer.register_parameter("w13_input_scale", w13_input_scale)
            set_weight_attrs(w13_input_scale, extra_weight_attrs)

            w2_input_scale = torch.nn.Parameter(
                torch.ones(num_experts, dtype=torch.float32), requires_grad=False
            )
            layer.register_parameter("w2_input_scale", w2_input_scale)
            set_weight_attrs(w2_input_scale, extra_weight_attrs)

        else:
            layer.w13_input_scale = None
            layer.w2_input_scale = None
```
**EN:** Defines function `Fp8MoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `extra_weight_attrs.update`, `get_tensor_model_parallel_world_size`, `torch.empty`.
**CN:** 定义函数 `Fp8MoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `extra_weight_attrs.update`, `get_tensor_model_parallel_world_size`, `torch.empty`。

### Method `Fp8MoEMethod.apply_monolithic` (lines 878-900)
```python
    def apply_monolithic(
        self,
        layer: RoutedExperts,
        x: torch.Tensor,
        router_logits: torch.Tensor,
        input_ids: torch.Tensor | None = None,
    ) -> torch.Tensor:
        assert self.is_monolithic
        assert self.moe_kernel is not None
        return self.moe_kernel.apply_monolithic(
            x,
            layer.w13_weight,
            layer.w2_weight,
            router_logits,
            activation=layer.activation,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            num_expert_group=layer.num_expert_group,
            topk_group=layer.topk_group,
            e_score_correction_bias=layer.e_score_correction_bias,
            routed_scaling_factor=layer.routed_scaling_factor,
        )
```
**EN:** Defines function `Fp8MoEMethod.apply_monolithic` with signature `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `router_logits`, `input_ids`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply_monolithic`.
**CN:** 定义函数 `Fp8MoEMethod.apply_monolithic`，其签名为 `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `router_logits`, `input_ids` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply_monolithic`。

### Method `Fp8MoEMethod.apply` (lines 902-925)
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
        assert not self.is_monolithic
        assert self.moe_kernel is not None
        return self.moe_kernel.apply(
            x,
            layer.w13_weight,
            layer.w2_weight,
            topk_weights,
            topk_ids,
            activation=layer.activation,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            shared_experts=shared_experts,
            shared_experts_input=shared_experts_input,
        )
```
**EN:** Defines function `Fp8MoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply`.
**CN:** 定义函数 `Fp8MoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply`。

### Class `Fp8OnlineMoEMethod` overview (lines 930-1051)
```python
class Fp8OnlineMoEMethod(Fp8MoEMethod):
    """MoE method for online FP8 quantization.
    Supports loading quantized FP16/BF16 model checkpoints with dynamic
    activation scaling. The weight scaling factor will be initialized after
    the model weights are loaded.

    Args:
        quant_config: The quantization config.
    """

    uses_meta_device: bool = True

    def __init__(self, quant_config: Fp8Config, layer: RoutedExperts):
        super().__init__(quant_config, layer)
        assert not quant_config.is_checkpoint_fp8_serialized
        assert quant_config.activation_scheme == "dynamic"
        assert quant_config.weight_block_size is None

    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
```
**EN:** Defines class `Fp8OnlineMoEMethod` with base classes `Fp8MoEMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 3 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`. Its docstring says: MoE method for online FP8 quantization.
**CN:** 定义类 `Fp8OnlineMoEMethod`，其基类为 `Fp8MoEMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 3 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `Fp8OnlineMoEMethod.__init__` (lines 942-946)
```python
    def __init__(self, quant_config: Fp8Config, layer: RoutedExperts):
        super().__init__(quant_config, layer)
        assert not quant_config.is_checkpoint_fp8_serialized
        assert quant_config.activation_scheme == "dynamic"
        assert quant_config.weight_block_size is None
```
**EN:** Defines function `Fp8OnlineMoEMethod.__init__` with signature `__init__(self, quant_config: Fp8Config, layer: RoutedExperts)`. It mainly works with `quant_config`, `layer`; initializes the object state and cached resources. The body uses validation/error handling. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `Fp8OnlineMoEMethod.__init__`，其签名为 `__init__(self, quant_config: Fp8Config, layer: RoutedExperts)`。它主要围绕 `quant_config`, `layer` 展开；负责初始化对象状态和缓存资源。函数体包含校验或报错逻辑。关键调用包括 `super.__init__`, `super`。

### Method `Fp8OnlineMoEMethod.create_weights` (lines 948-1014)
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
        layer.num_experts = num_experts
        layer.orig_dtype = params_dtype
        layer.weight_block_size = None

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                device="meta",
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
                device="meta",  # materialized and processed during loading
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
# ... truncated for analysis ...

            w2_bias = torch.nn.Parameter(
                torch.zeros(
                    num_experts,
                    hidden_size,
                    device="meta",  # materialized and processed during loading
                    dtype=layer.orig_dtype,
                ),
                requires_grad=False,
            )
            layer.register_parameter("w2_bias", w2_bias)
            set_weight_attrs(w2_bias, extra_weight_attrs)

        initialize_online_processing(layer)
```
**EN:** Defines function `Fp8OnlineMoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `initialize_online_processing`, `torch.empty`, `torch.zeros`.
**CN:** 定义函数 `Fp8OnlineMoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `initialize_online_processing`, `torch.empty`, `torch.zeros`。

### Method `Fp8OnlineMoEMethod.process_weights_after_loading` (lines 1016-1051)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        # TODO(@ksayers): inplace fp8 quant kernel, initialize scales with ones
        if getattr(layer, "_already_called_process_weights_after_loading", False):
            return

        fp8_dtype = current_platform.fp8_dtype()
        w13 = torch.empty_like(layer.w13_weight, dtype=fp8_dtype)
        w2 = torch.empty_like(layer.w2_weight, dtype=fp8_dtype)
        w13_scale = torch.ones(
            layer.num_experts, device=w13.device, dtype=torch.float32
        )
        w2_scale = torch.ones(layer.num_experts, device=w2.device, dtype=torch.float32)
        layer.w13_input_scale = None
        layer.w2_input_scale = None

        for expert in range(layer.local_num_experts):
            w13[expert, :, :], w13_scale[expert] = ops.scaled_fp8_quant(
                layer.w13_weight[expert, :, :]
            )
            w2[expert, :, :], w2_scale[expert] = ops.scaled_fp8_quant(
                layer.w2_weight[expert, :, :]
            )

        # Shuffle weights to runtime format and setup kernel.
        self._setup_kernel(
            layer,
            w13,
            w2,
            w13_scale,
            w2_scale,
            w13_input_scale=layer.w13_input_scale,
            w2_input_scale=layer.w2_input_scale,
        )

        # Prevent duplicate processing (e.g., during weight reload)
        layer._already_called_process_weights_after_loading = True
```
**EN:** Defines function `Fp8OnlineMoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, tensor/kernel operations. Key calls include `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `torch.ones`, `range`, `self._setup_kernel`.
**CN:** 定义函数 `Fp8OnlineMoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `getattr`, `current_platform.fp8_dtype`, `torch.empty_like`, `torch.ones`, `range`, `self._setup_kernel`。

### Class `Fp8KVCacheMethod` overview (lines 1054-1060)
```python
class Fp8KVCacheMethod(BaseKVCacheMethod):
    """
    Supports loading kv-cache scaling factors from FP8 checkpoints.
    """

    def __init__(self, quant_config: Fp8Config):
        super().__init__(quant_config)
```
**EN:** Defines class `Fp8KVCacheMethod` with base classes `BaseKVCacheMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 1 direct methods, with notable entries `__init__`. Its docstring says: Supports loading kv-cache scaling factors from FP8 checkpoints.
**CN:** 定义类 `Fp8KVCacheMethod`，其基类为 `BaseKVCacheMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 1 个方法，较重要的包括 `__init__`。 文档字符串进一步说明了该类的定位。

### Method `Fp8KVCacheMethod.__init__` (lines 1059-1060)
```python
    def __init__(self, quant_config: Fp8Config):
        super().__init__(quant_config)
```
**EN:** Defines function `Fp8KVCacheMethod.__init__` with signature `__init__(self, quant_config: Fp8Config)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `Fp8KVCacheMethod.__init__`，其签名为 `__init__(self, quant_config: Fp8Config)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `Fp8Config`, `CopyNumelCounter`, `Fp8LinearMethod`, `Fp8OnlineLinearMethod`, `Fp8MoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `Fp8Config`, `CopyNumelCounter`, `Fp8LinearMethod`, `Fp8OnlineLinearMethod`, `Fp8MoEMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `_copy_missing_attrs` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_copy_missing_attrs` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`
- **Internal / 内部**: `vllm.envs`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.config`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.kernels.linear.scaled_mm`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.oracle.fp8`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.kv_cache`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`
