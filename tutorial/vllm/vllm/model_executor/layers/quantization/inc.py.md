# inc.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/inc.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `INCConfig`, `INCXPULinearBase`, `INCXPULinearMethod` for quantization backends, schemes, and utilities. / 实现 `INCConfig`, `INCXPULinearBase`, `INCXPULinearMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-30)
```python
from fractions import Fraction
from functools import lru_cache
from typing import TYPE_CHECKING, Any

import regex as re
import torch
from torch.nn.parameter import Parameter

from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe import RoutedExperts
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from vllm.model_executor.layers.quantization import (
    QuantizationConfig,
    QuantizationMethods,
)
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.model_executor.parameter import (
    GroupQuantScaleParameter,
    PackedvLLMParameter,
    RowvLLMParameter,
)
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
```
**EN:** This opening block pulls in external dependencies such as `fractions`, `functools`, `typing`, `regex`, `torch` and internal modules such as `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.vocab_parallel_embedding`, `vllm.model_executor.parameter`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `fractions`, `functools`, `typing`, `regex`, `torch`）以及内部模块（如 `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.vocab_parallel_embedding`, `vllm.model_executor.parameter`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 35-35)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `get_ark_state` (lines 613-635)
```python
def get_ark_state() -> tuple[bool, str | None, Any | None, Any | None]:
    """Return ARK availability, error details, cached instance, and QuantLinear."""
    try:
        import auto_round_kernel
        from auto_round_kernel.qlinear import QuantLinear

        logger.info("Successfully imported auto_round_kernel.")
    except ImportError as error:
        return False, str(error), None, None

    ark_loader = getattr(auto_round_kernel, "_ark_instance", None)
    if not callable(ark_loader):
        return False, "auto_round_kernel does not expose _ark_instance().", None, None

    try:
        ark_instance = ark_loader()
    except Exception as error:
        return False, str(error), None, None

    if ark_instance is None:
        return False, "auto_round_kernel._ark_instance() returned None.", None, None

    return True, None, ark_instance, QuantLinear
```
**EN:** Defines function `get_ark_state` with signature `get_ark_state() -> tuple[bool, str | None, Any | None, Any | None]`. It mainly works with object context only; returns a derived property or capability check. The body uses branching. Key calls include `lru_cache`, `getattr`, `logger.info`, `callable`, `ark_loader`, `str`.
**CN:** 定义函数 `get_ark_state`，其签名为 `get_ark_state() -> tuple[bool, str | None, Any | None, Any | None]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含分支判断。关键调用包括 `lru_cache`, `getattr`, `logger.info`, `callable`, `ark_loader`, `str`。

### Class `INCConfig` overview (lines 38-517)
```python
class INCConfig(QuantizationConfig):
    """Config class for Intel Neural Compressor (INC).
    Repo: https://github.com/intel/neural-compressor
    """

    SUPPORTED_BITS = {2, 3, 4, 8}
    SUPPORTED_DTYPES = {"int"}
    SUPPORTED_FORMATS = {"auto_round:auto_gptq", "auto_round:auto_awq"}
    SUPPORTED_BACKENDS = {
        "auto",
        "gptq",
        "gptq:marlin",
        "awq",
        "awq:marlin",
        "marlin",
    }

    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        sym: bool = True,
        packing_format: str = "auto_round:auto_gptq",
        block_name_to_quantize: str | list[str] | None = None,
        extra_config: dict[str, Any] | None = None,
```
**EN:** Defines class `INCConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 16 direct methods, with notable entries `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`. Its docstring says: Config class for Intel Neural Compressor (INC).
**CN:** 定义类 `INCConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 16 个方法，较重要的包括 `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`。 文档字符串进一步说明了该类的定位。

### Method `INCConfig.__init__` (lines 55-100)
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        sym: bool = True,
        packing_format: str = "auto_round:auto_gptq",
        block_name_to_quantize: str | list[str] | None = None,
        extra_config: dict[str, Any] | None = None,
        data_type: str = "int",
        backend: str = "auto",
    ) -> None:
        super().__init__()
        if weight_bits not in self.SUPPORTED_BITS:
            raise ValueError(
                f"Unsupported weight_bits: {weight_bits}, "
                f"currently only support {self.SUPPORTED_BITS}."
            )
        if data_type not in self.SUPPORTED_DTYPES:
            raise ValueError(
                f"Unsupported data_type: {data_type},"
                f" currently only support  {self.SUPPORTED_DTYPES}."
            )
        if packing_format not in self.SUPPORTED_FORMATS:
            raise ValueError(
                f"Unsupported packing_format: {packing_format}, "
                f"currently only support {self.SUPPORTED_FORMATS}."
            )
        if backend not in self.SUPPORTED_BACKENDS:
            raise ValueError(
                f"Unsupported backend: {backend},  "
                f"currently only support {self.SUPPORTED_BACKENDS}."
            )

        self.weight_bits = weight_bits
        self.group_size = group_size
        self.sym = sym
        self.packing_format = packing_format
        self.block_name_to_quantize = (
            block_name_to_quantize.split(",")
            if isinstance(block_name_to_quantize, str)
            else block_name_to_quantize
        )
        self.extra_config = extra_config
        self.data_type = data_type
        self.backend = backend
        self.pack_factor = Fraction(32, weight_bits)
```
**EN:** Defines function `INCConfig.__init__` with signature `__init__(self, weight_bits: int, group_size: int, sym: bool=True, packing_format: str='auto_round:auto_gptq', block_name_to_quantize: str | list[str] | None=None, extra_config: dict[str, Any] | None=None, data_type: str='int', backend: str='auto') -> None`. It mainly works with `weight_bits`, `group_size`, `sym`, `packing_format`, `block_name_to_quantize`, `extra_config`, `data_type`, `backend`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `Fraction`, `ValueError`, `isinstance`, `block_name_to_quantize.split`, `super`.
**CN:** 定义函数 `INCConfig.__init__`，其签名为 `__init__(self, weight_bits: int, group_size: int, sym: bool=True, packing_format: str='auto_round:auto_gptq', block_name_to_quantize: str | list[str] | None=None, extra_config: dict[str, Any] | None=None, data_type: str='int', backend: str='auto') -> None`。它主要围绕 `weight_bits`, `group_size`, `sym`, `packing_format`, `block_name_to_quantize`, `extra_config`, `data_type`, `backend` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `Fraction`, `ValueError`, `isinstance`, `block_name_to_quantize.split`, `super`。

### Method `INCConfig.apply_awq_quant_layer` (lines 238-321)
```python
    def apply_awq_quant_layer(self, layer, prefix: str, backend: str = "auto"):
        from vllm.model_executor.layers.quantization.utils.marlin_utils import (
            check_marlin_supported,
            check_moe_marlin_supports_layer,
        )

        weight_bits, group_size, sym = self.get_layer_config(layer, prefix)
        if not self.check_quantized(weight_bits):
            if isinstance(layer, (LinearBase, ParallelLMHead)):
                return UnquantizedLinearMethod()
            else:
                return None

        logger.debug(
            "[%s] Type: %s, Bits: %s, Group Size: %s, Sym: %s",
            prefix,
            layer.__class__.__name__,
            weight_bits,
            group_size,
            sym,
        )
        if backend == "auto" or "marlin" in backend:
            AWQ_TYPE_MAP = {
                4: scalar_types.uint4,
                8: scalar_types.uint8,
            }
            use_marlin = (weight_bits in AWQ_TYPE_MAP) and check_marlin_supported(
                AWQ_TYPE_MAP[weight_bits], group_size, not sym
            )

            if isinstance(layer, RoutedExperts):
                use_marlin = use_marlin and check_moe_marlin_supports_layer(
                    layer, group_size
                )

        else:
            use_marlin = False
        if use_marlin:
# ... truncated for analysis ...
                "quant_method": "awq",
                "bits": weight_bits,
                "group_size": group_size,
                "zero_point": not sym,
                "lm_head": False,
            }
            return MoeWNA16Config.from_config(config).get_quant_method(layer, prefix)

        if isinstance(layer, (LinearBase, ParallelLMHead)):
            if use_marlin:
                return AWQMarlinLinearMethod(quant_args_marlin)
            else:
                return AWQLinearMethod(quant_args)
        return None
```
**EN:** Defines function `INCConfig.apply_awq_quant_layer` with signature `apply_awq_quant_layer(self, layer, prefix: str, backend: str='auto')`. It mainly works with `layer`, `prefix`, `backend`; handles quantization-related transformation logic. The body uses branching. Key calls include `self.get_layer_config`, `logger.debug`, `isinstance`, `self.check_quantized`, `AWQMarlinConfig`, `AWQConfig`.
**CN:** 定义函数 `INCConfig.apply_awq_quant_layer`，其签名为 `apply_awq_quant_layer(self, layer, prefix: str, backend: str='auto')`。它主要围绕 `layer`, `prefix`, `backend` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `self.get_layer_config`, `logger.debug`, `isinstance`, `self.check_quantized`, `AWQMarlinConfig`, `AWQConfig`。

### Method `INCConfig.apply_gptq_quant_layer` (lines 323-404)
```python
    def apply_gptq_quant_layer(self, layer, prefix: str, backend: str = "auto"):
        from vllm.model_executor.layers.quantization.utils.marlin_utils import (
            check_marlin_supported,
            check_moe_marlin_supports_layer,
        )

        weight_bits, group_size, sym = self.get_layer_config(layer, prefix)
        if not self.check_quantized(weight_bits):
            if isinstance(layer, (LinearBase, ParallelLMHead)):
                return UnquantizedLinearMethod()
            else:
                return None

        logger.debug(
            "[%s] Type: %s, Bits: %s, Group Size: %s, Sym: %s",
            prefix,
            layer.__class__.__name__,
            weight_bits,
            group_size,
            sym,
        )
        if backend == "auto" or "marlin" in backend:
            GPTQ_TYPE_MAP = {
                (4, True): scalar_types.uint4b8,
                (8, True): scalar_types.uint8b128,
            }
            use_marlin = (weight_bits, sym) in GPTQ_TYPE_MAP and check_marlin_supported(
                GPTQ_TYPE_MAP[(weight_bits, sym)], group_size, has_zp=not sym
            )
            if isinstance(layer, RoutedExperts):
                use_marlin = use_marlin and check_moe_marlin_supports_layer(
                    layer, group_size
                )
        else:
            use_marlin = False
        if use_marlin:
            from vllm.model_executor.layers.quantization.auto_gptq import (
                AutoGPTQConfig,
# ... truncated for analysis ...
                    layer, prefix
                )

        if isinstance(layer, (LinearBase, ParallelLMHead)):
            if use_marlin:
                return AutoGPTQLinearMethod(quant_args_marlin)
            else:
                raise NotImplementedError(
                    f"INC quantization with bits={weight_bits}, sym={sym} "
                    "is not supported. Only 4-bit and 8-bit symmetric "
                    "quantization is supported with Marlin kernels."
                )

        return None
```
**EN:** Defines function `INCConfig.apply_gptq_quant_layer` with signature `apply_gptq_quant_layer(self, layer, prefix: str, backend: str='auto')`. It mainly works with `layer`, `prefix`, `backend`; handles quantization-related transformation logic. The body uses branching, validation/error handling. Key calls include `self.get_layer_config`, `logger.debug`, `isinstance`, `self.check_quantized`, `AutoGPTQConfig`, `UnquantizedLinearMethod`.
**CN:** 定义函数 `INCConfig.apply_gptq_quant_layer`，其签名为 `apply_gptq_quant_layer(self, layer, prefix: str, backend: str='auto')`。它主要围绕 `layer`, `prefix`, `backend` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑。关键调用包括 `self.get_layer_config`, `logger.debug`, `isinstance`, `self.check_quantized`, `AutoGPTQConfig`, `UnquantizedLinearMethod`。

### Method `INCConfig.apply_xpu_w4a16_quant_layer` (lines 406-446)
```python
    def apply_xpu_w4a16_quant_layer(self, layer, prefix: str):
        weight_bits, group_size, sym = self.get_layer_config(layer, prefix)

        if not self.check_quantized(weight_bits):
            if isinstance(layer, (LinearBase, ParallelLMHead)):
                return UnquantizedLinearMethod()
            else:
                return None

        if weight_bits != 4:
            raise NotImplementedError(
                f"INC on XPU only supports 4-bit quantization, "
                f"got weight_bits={weight_bits}."
            )
        if not sym:
            raise NotImplementedError(
                "INC W4A16 on XPU only supports symmetric quantization for now."
            )

        if isinstance(layer, (LinearBase, ParallelLMHead)):
            is_ark_available, ark_error, _, _ = get_ark_state()
            if is_ark_available:
                return INCARKLinearMethod(
                    weight_bits=weight_bits,
                    group_size=group_size,
                    sym=sym,
                )

            logger.debug(
                "ARK backend is unavailable for layer %s; "
                "falling back to the default XPU INC path. Error: %s",
                prefix,
                ark_error or "unknown error",
            )

            return INCXPULinearMethod(
                weight_bits=weight_bits,
                group_size=group_size,
                sym=sym,
            )
        return None
```
**EN:** Defines function `INCConfig.apply_xpu_w4a16_quant_layer` with signature `apply_xpu_w4a16_quant_layer(self, layer, prefix: str)`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching, validation/error handling. Key calls include `self.get_layer_config`, `isinstance`, `self.check_quantized`, `NotImplementedError`, `get_ark_state`, `logger.debug`.
**CN:** 定义函数 `INCConfig.apply_xpu_w4a16_quant_layer`，其签名为 `apply_xpu_w4a16_quant_layer(self, layer, prefix: str)`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断、校验或报错逻辑。关键调用包括 `self.get_layer_config`, `isinstance`, `self.check_quantized`, `NotImplementedError`, `get_ark_state`, `logger.debug`。

### Class `INCXPULinearBase` overview (lines 520-609)
```python
class INCXPULinearBase(LinearMethodBase):
    def __init__(self, weight_bits: int, group_size: int, sym: bool):
        self.weight_bits = weight_bits
        self.group_size = group_size
        self.sym = sym
        self.pack_factor = 32 // weight_bits

    def _create_inc_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        params_dtype: torch.dtype,
        weight_loader: Any,
        group_size: int,
        pack_factor: int,
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)
        scales_and_zp_size = input_size_per_partition // group_size

        qweight = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition // pack_factor,
                output_size_per_partition,
                dtype=torch.int32,
```
**EN:** Defines class `INCXPULinearBase` with base classes `LinearMethodBase` and decorators none. It acts as a quantization-oriented module building block and exposes 3 direct methods, with notable entries `__init__`, `_create_inc_weights`, `create_weights`.
**CN:** 定义类 `INCXPULinearBase`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 3 个方法，较重要的包括 `__init__`, `_create_inc_weights`, `create_weights`。

### Method `INCXPULinearBase.__init__` (lines 521-525)
```python
    def __init__(self, weight_bits: int, group_size: int, sym: bool):
        self.weight_bits = weight_bits
        self.group_size = group_size
        self.sym = sym
        self.pack_factor = 32 // weight_bits
```
**EN:** Defines function `INCXPULinearBase.__init__` with signature `__init__(self, weight_bits: int, group_size: int, sym: bool)`. It mainly works with `weight_bits`, `group_size`, `sym`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `INCXPULinearBase.__init__`，其签名为 `__init__(self, weight_bits: int, group_size: int, sym: bool)`。它主要围绕 `weight_bits`, `group_size`, `sym` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `INCXPULinearBase._create_inc_weights` (lines 527-589)
```python
    def _create_inc_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        params_dtype: torch.dtype,
        weight_loader: Any,
        group_size: int,
        pack_factor: int,
    ) -> None:
        output_size_per_partition = sum(output_partition_sizes)
        scales_and_zp_size = input_size_per_partition // group_size

        qweight = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition // pack_factor,
                output_size_per_partition,
                dtype=torch.int32,
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=0,
            packed_factor=pack_factor,
            weight_loader=weight_loader,
        )

        scales = GroupQuantScaleParameter(
            data=torch.empty(
                scales_and_zp_size,
                output_size_per_partition,
                dtype=params_dtype,
            ),
            input_dim=0,
            output_dim=1,
            weight_loader=weight_loader,
        )

        qzeros = PackedvLLMParameter(
# ... truncated for analysis ...

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("scales", scales)
        layer.register_parameter("qzeros", qzeros)

        g_idx = RowvLLMParameter(
            data=torch.tensor(
                [i // group_size for i in range(input_size_per_partition)],
                dtype=torch.int32,
            ),
            input_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("g_idx", g_idx)
```
**EN:** Defines function `INCXPULinearBase._create_inc_weights` with signature `_create_inc_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], params_dtype: torch.dtype, weight_loader: Any, group_size: int, pack_factor: int) -> None`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `params_dtype`, `weight_loader`, `group_size`, `pack_factor`; implements one step in the quantized-weight execution flow. The body uses comprehensions, tensor/kernel operations. Key calls include `sum`, `PackedvLLMParameter`, `GroupQuantScaleParameter`, `layer.register_parameter`, `RowvLLMParameter`, `torch.empty`.
**CN:** 定义函数 `INCXPULinearBase._create_inc_weights`，其签名为 `_create_inc_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], params_dtype: torch.dtype, weight_loader: Any, group_size: int, pack_factor: int) -> None`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `params_dtype`, `weight_loader`, `group_size`, `pack_factor` 展开；实现量化权重执行流程中的一个步骤。函数体包含推导式、张量或内核操作。关键调用包括 `sum`, `PackedvLLMParameter`, `GroupQuantScaleParameter`, `layer.register_parameter`, `RowvLLMParameter`, `torch.empty`。

### Method `INCXPULinearBase.create_weights` (lines 591-609)
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
        self._create_inc_weights(
            layer=layer,
            input_size_per_partition=input_size_per_partition,
            output_partition_sizes=output_partition_sizes,
            params_dtype=params_dtype,
            weight_loader=extra_weight_attrs.get("weight_loader"),
            group_size=self.group_size,
            pack_factor=self.pack_factor,
        )
```
**EN:** Defines function `INCXPULinearBase.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses mostly straightforward data movement and object wiring. Key calls include `self._create_inc_weights`, `extra_weight_attrs.get`.
**CN:** 定义函数 `INCXPULinearBase.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self._create_inc_weights`, `extra_weight_attrs.get`。

### Class `INCXPULinearMethod` overview (lines 638-693)
```python
class INCXPULinearMethod(INCXPULinearBase):
    """XPU linear method for INC w4a16 GPTQ quantization (symmetric only).

    Repacks GPTQ weights from [in_packed, out] to oneDNN [out, in_packed]
    layout and calls torch.ops._xpu_C.int4_gemm_w4a16.

    GPTQ format: qweight [in_packed, out] with sequential nibble order.

    Note: Asymmetric quantization (sym=false) is not for now.

    FIXME(yiliu30): Refine the implementation to reuse XPUwNa16LinearKernel.
    """

    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        """Repack GPTQ weights into kernel-ready NT layout."""
        device = layer.qweight.data.device

        # oneDNN int4 kernel requires strides[0]==1 ("NT format"), but GPTQ
        # checkpoint is [K_packed, N] contiguous with strides (N, 1).
        # Two transposes are needed — neither alone can achieve this:
        #   1. .t().contiguous() → [N, K_packed] contiguous in memory
        #   2. .t()              → [K_packed, N] view with strides (1, K_packed)
        # The result has the same logical shape but strides[0]==1 as required.
        qweight_ct = layer.qweight.data.t().contiguous()
        layer.qweight = Parameter(qweight_ct.t(), requires_grad=False)
```
**EN:** Defines class `INCXPULinearMethod` with base classes `INCXPULinearBase` and decorators none. It acts as a backend or execution-method adapter and exposes 2 direct methods, with notable entries `process_weights_after_loading`, `apply`. Its docstring says: XPU linear method for INC w4a16 GPTQ quantization (symmetric only).
**CN:** 定义类 `INCXPULinearMethod`，其基类为 `INCXPULinearBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 2 个方法，较重要的包括 `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `INCXPULinearMethod.process_weights_after_loading` (lines 651-672)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        """Repack GPTQ weights into kernel-ready NT layout."""
        device = layer.qweight.data.device

        # oneDNN int4 kernel requires strides[0]==1 ("NT format"), but GPTQ
        # checkpoint is [K_packed, N] contiguous with strides (N, 1).
        # Two transposes are needed — neither alone can achieve this:
        #   1. .t().contiguous() → [N, K_packed] contiguous in memory
        #   2. .t()              → [K_packed, N] view with strides (1, K_packed)
        # The result has the same logical shape but strides[0]==1 as required.
        qweight_ct = layer.qweight.data.t().contiguous()
        layer.qweight = Parameter(qweight_ct.t(), requires_grad=False)

        # Scales: [num_groups, out] — no change needed
        layer.scales = Parameter(layer.scales.data, requires_grad=False)

        # Symmetric: GPTQ v1 stores qzeros=7, effective zp = 7+1 = 8
        # Kernel expects int8 scalar = 8
        layer.qzeros = Parameter(
            torch.tensor([8], dtype=torch.int8, device=device),
            requires_grad=False,
        )
```
**EN:** Defines function `INCXPULinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses tensor/kernel operations. Key calls include `layer.qweight.data.t.contiguous`, `Parameter`, `qweight_ct.t`, `torch.tensor`, `layer.qweight.data.t`.
**CN:** 定义函数 `INCXPULinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含张量或内核操作。关键调用包括 `layer.qweight.data.t.contiguous`, `Parameter`, `qweight_ct.t`, `torch.tensor`, `layer.qweight.data.t`。

### Method `INCXPULinearMethod.apply` (lines 674-693)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # qweight is already in NT layout [K_packed, N] (strides (1, K_packed))
        # from process_weights_after_loading — pass directly to kernel.
        out_shape = x.shape[:-1] + (layer.qweight.shape[1],)
        reshaped_x = x.reshape(-1, x.shape[-1])
        out = torch.ops._xpu_C.int4_gemm_w4a16(
            reshaped_x,
            layer.qweight,
            bias,
            layer.scales,
            layer.qzeros,
            self.group_size,
            None,  # g_idx not needed: desc_act is always False for INC models
        )
        return out.reshape(out_shape)
```
**EN:** Defines function `INCXPULinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses tensor/kernel operations. Key calls include `x.reshape`, `torch.ops._xpu_C.int4_gemm_w4a16`, `out.reshape`.
**CN:** 定义函数 `INCXPULinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含张量或内核操作。关键调用包括 `x.reshape`, `torch.ops._xpu_C.int4_gemm_w4a16`, `out.reshape`。

### Class `INCARKLinearMethod` overview (lines 696-794)
```python
class INCARKLinearMethod(INCXPULinearBase):
    """XPU & CPU w4a16 linear method for INC quantization utilizing the ARK backend.

    See: https://github.com/intel/auto-round/blob/main/auto_round_extension/ark/README.md

    Repacks GPTQ/INC weights into ARK's layout.
    """

    def __init__(self, weight_bits: int, group_size: int, sym: bool):
        super().__init__(weight_bits=weight_bits, group_size=group_size, sym=sym)

        is_available, error_str, _, quant_linear_cls = get_ark_state()
        if not is_available or quant_linear_cls is None:
            reason = error_str or "unknown error"
            raise ImportError(f"Failed to import auto_round_kernel. {reason}")

        self.QuantLinear = quant_linear_cls

    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
```
**EN:** Defines class `INCARKLinearMethod` with base classes `INCXPULinearBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: XPU & CPU w4a16 linear method for INC quantization utilizing the ARK backend.
**CN:** 定义类 `INCARKLinearMethod`，其基类为 `INCXPULinearBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `INCARKLinearMethod.__init__` (lines 704-712)
```python
    def __init__(self, weight_bits: int, group_size: int, sym: bool):
        super().__init__(weight_bits=weight_bits, group_size=group_size, sym=sym)

        is_available, error_str, _, quant_linear_cls = get_ark_state()
        if not is_available or quant_linear_cls is None:
            reason = error_str or "unknown error"
            raise ImportError(f"Failed to import auto_round_kernel. {reason}")

        self.QuantLinear = quant_linear_cls
```
**EN:** Defines function `INCARKLinearMethod.__init__` with signature `__init__(self, weight_bits: int, group_size: int, sym: bool)`. It mainly works with `weight_bits`, `group_size`, `sym`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `get_ark_state`, `ImportError`, `super`.
**CN:** 定义函数 `INCARKLinearMethod.__init__`，其签名为 `__init__(self, weight_bits: int, group_size: int, sym: bool)`。它主要围绕 `weight_bits`, `group_size`, `sym` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `get_ark_state`, `ImportError`, `super`。

### Method `INCARKLinearMethod.create_weights` (lines 714-735)
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
        super().create_weights(
            layer=layer,
            input_size_per_partition=input_size_per_partition,
            output_partition_sizes=output_partition_sizes,
            input_size=input_size,
            output_size=output_size,
            params_dtype=params_dtype,
            **extra_weight_attrs,
        )
        layer.in_features = input_size_per_partition
        layer.out_features = sum(output_partition_sizes)
        layer.params_dtype = params_dtype
```
**EN:** Defines function `INCARKLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses mostly straightforward data movement and object wiring. Key calls include `super.create_weights`, `sum`, `super`.
**CN:** 定义函数 `INCARKLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.create_weights`, `sum`, `super`。

### Method `INCARKLinearMethod.process_weights_after_loading` (lines 737-786)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if hasattr(layer, "input_size_per_partition"):
            in_features = layer.input_size_per_partition
        elif hasattr(layer, "input_size"):
            in_features = layer.input_size
        else:
            raise AttributeError("Cannot determine in_features for layer.")

        if hasattr(layer, "output_partition_sizes"):
            out_features = sum(layer.output_partition_sizes)
        elif hasattr(layer, "output_size_per_partition"):
            out_features = layer.output_size_per_partition
        elif hasattr(layer, "output_size"):
            out_features = layer.output_size
        else:
            out_features = layer.scales.shape[-1]

        ark_linear = self.QuantLinear(
            bits=self.weight_bits,
            group_size=self.group_size,
            sym=self.sym,
            in_features=in_features,
            out_features=out_features,
            bias=layer.bias is not None,
            weight_dtype=layer.params_dtype,
        )

        ark_linear.to(layer.qweight.device)

        with torch.no_grad():
            ark_linear.qweight.copy_(layer.qweight.detach())

            if hasattr(layer, "qzeros") and layer.qzeros is not None:
                ark_linear.qzeros.copy_(layer.qzeros.detach())
            else:
                ark_linear.qzeros = None

            ark_linear.scales.copy_(layer.scales.detach())

            if hasattr(layer, "bias") and layer.bias is not None:
                ark_linear.bias.copy_(layer.bias.detach())

        ark_linear.post_init()

        layer.ark_linear = ark_linear

        del layer.qweight
        if hasattr(layer, "qzeros"):
            del layer.qzeros
        del layer.scales
```
**EN:** Defines function `INCARKLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, context-managed execution, validation/error handling, tensor/kernel operations. Key calls include `hasattr`, `self.QuantLinear`, `ark_linear.to`, `ark_linear.post_init`, `sum`, `torch.no_grad`.
**CN:** 定义函数 `INCARKLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、上下文管理、校验或报错逻辑、张量或内核操作。关键调用包括 `hasattr`, `self.QuantLinear`, `ark_linear.to`, `ark_linear.post_init`, `sum`, `torch.no_grad`。

### Method `INCARKLinearMethod.apply` (lines 788-794)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return layer.ark_linear.forward(x)
```
**EN:** Defines function `INCARKLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `layer.ark_linear.forward`.
**CN:** 定义函数 `INCARKLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `layer.ark_linear.forward`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `INCConfig`, `INCXPULinearBase`, `INCXPULinearMethod`, `INCARKLinearMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `INCConfig`, `INCXPULinearBase`, `INCXPULinearMethod`, `INCARKLinearMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_ark_state` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_ark_state` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `fractions`, `functools`, `typing`, `regex`, `torch`
- **Internal / 内部**: `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.vocab_parallel_embedding`, `vllm.model_executor.parameter`, `vllm.platforms`, `vllm.scalar_type`
