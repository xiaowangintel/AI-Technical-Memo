# modelopt.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/modelopt.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `ModelOptKVCacheMethod`, `ModelOptQuantConfigBase`, `ModelOptFp8Config` for quantization backends, schemes, and utilities. / 实现 `ModelOptKVCacheMethod`, `ModelOptQuantConfigBase`, `ModelOptFp8Config`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-96)
```python
from fnmatch import fnmatch
from typing import TYPE_CHECKING, Any

import torch
from torch.nn.parameter import Parameter

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.config import get_current_vllm_config
from vllm.logger import init_logger
from vllm.model_executor.kernels.linear import (
    MarlinNvFp4LinearKernel,
    NvFp4LinearLayerConfig,
    init_fp8_linear_kernel,
    init_mxfp8_linear_kernel,
    init_nvfp4_linear_kernel,
)
from vllm.model_executor.layers.attention import Attention, MLAAttention
from vllm.model_executor.layers.fused_moe import (
    FusedMoEConfig,
    FusedMoEMethodBase,
    FusedMoEQuantConfig,
    FusedMoeWeightScaleSupported,
    MoEActivation,
    RoutedExperts,
    RoutingMethodType,
    SharedExperts,
)
from vllm.model_executor.layers.fused_moe.oracle.fp8 import (
    Fp8MoeBackend,
    convert_to_fp8_moe_kernel_format,
    make_fp8_moe_kernel,
    make_fp8_moe_quant_config,
    select_fp8_moe_backend,
)
from vllm.model_executor.layers.fused_moe.oracle.mxfp8 import (
    select_mxfp8_moe_backend,
)
from vllm.model_executor.layers.fused_moe.oracle.nvfp4 import (
    convert_to_nvfp4_moe_kernel_format,
    is_global_sf_supported_for_nvfp4_backend,
    make_nvfp4_moe_kernel,
    make_nvfp4_moe_quant_config,
    select_nvfp4_moe_backend,
)
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from vllm.model_executor.layers.quantization import QuantizationMethods
# ... truncated for analysis ...
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    requantize_with_max_scale,
)
from vllm.model_executor.parameter import (
    BlockQuantScaleParameter,
    ChannelQuantScaleParameter,
    GroupQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from vllm.model_executor.utils import replace_parameter, set_weight_attrs
from vllm.utils.flashinfer import flashinfer_trtllm_fp8_block_scale_moe
```
**EN:** This opening block pulls in external dependencies such as `fnmatch`, `typing`, `torch` and internal modules such as `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.config`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.fused_moe`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `fnmatch`, `typing`, `torch`）以及内部模块（如 `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.config`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.fused_moe`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 101-1002)
```python
logger = init_logger(__name__)

QUANT_ALGOS = [
    # FP8 (per-tensor weight + optional static activation scale).
    "FP8",
    # FP8 per-channel weight scale + per-token activation scale.
    "FP8_PER_CHANNEL_PER_TOKEN",
    # FP8 per-block weight-only (ModelOpt may emit this as lowercase).
    "FP8_PB_WO",
    # NVFP4 W4A4 (4-bit float weights AND 4-bit float activations).
    "NVFP4",
    # W4A16 NVFP4 (4-bit float weights, fp16/bf16 activations).
    "W4A16_NVFP4",
    # MXFP8
    "MXFP8",
    # MIXED_PRECISION,
    "MIXED_PRECISION",
]
KV_CACHE_QUANT_ALGOS = ["FP8", "NVFP4"]


class ModelOptKVCacheMethod(BaseKVCacheMethod):
    """
    Supports loading kv-cache scaling factors from FP8 or NVFP4 checkpoints.
    """

    def __init__(self, quant_config: "ModelOptQuantConfigBase"):
        super().__init__(quant_config)


class ModelOptQuantConfigBase(QuantizationConfig):
    LinearMethodCls: type = LinearMethodBase
    FusedMoEMethodCls: type = FusedMoEMethodBase
    KVCacheMethodCls: type = BaseKVCacheMethod

    def __init__(
        self,
        exclude_modules: list[str],
    ):
        super().__init__()
        self.exclude_modules: list[str] = exclude_modules

    def is_layer_excluded(self, prefix: str) -> bool:
        """
        Check if a layer should be excluded from quantization.
# ... truncated for analysis ...
            activation=layer.activation,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            shared_experts=shared_experts,
            shared_experts_input=shared_experts_input,
        )


ModelOptFp8Config.LinearMethodCls = ModelOptFp8LinearMethod
ModelOptFp8Config.FusedMoEMethodCls = ModelOptFp8MoEMethod
ModelOptFp8Config.KVCacheMethodCls = ModelOptKVCacheMethod
```
**EN:** This block defines module-level metadata or constants such as `logger`, `QUANT_ALGOS`, `KV_CACHE_QUANT_ALGOS`, `LinearMethodCls`, `FusedMoEMethodCls`, `KVCacheMethodCls`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `QUANT_ALGOS`, `KV_CACHE_QUANT_ALGOS`, `LinearMethodCls`, `FusedMoEMethodCls`, `KVCacheMethodCls`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Class `ModelOptKVCacheMethod` overview (lines 122-128)
```python
class ModelOptKVCacheMethod(BaseKVCacheMethod):
    """
    Supports loading kv-cache scaling factors from FP8 or NVFP4 checkpoints.
    """

    def __init__(self, quant_config: "ModelOptQuantConfigBase"):
        super().__init__(quant_config)
```
**EN:** Defines class `ModelOptKVCacheMethod` with base classes `BaseKVCacheMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 1 direct methods, with notable entries `__init__`. Its docstring says: Supports loading kv-cache scaling factors from FP8 or NVFP4 checkpoints.
**CN:** 定义类 `ModelOptKVCacheMethod`，其基类为 `BaseKVCacheMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 1 个方法，较重要的包括 `__init__`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptKVCacheMethod.__init__` (lines 127-128)
```python
    def __init__(self, quant_config: "ModelOptQuantConfigBase"):
        super().__init__(quant_config)
```
**EN:** Defines function `ModelOptKVCacheMethod.__init__` with signature `__init__(self, quant_config: 'ModelOptQuantConfigBase')`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `ModelOptKVCacheMethod.__init__`，其签名为 `__init__(self, quant_config: 'ModelOptQuantConfigBase')`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Class `ModelOptQuantConfigBase` overview (lines 131-361)
```python
class ModelOptQuantConfigBase(QuantizationConfig):
    LinearMethodCls: type = LinearMethodBase
    FusedMoEMethodCls: type = FusedMoEMethodBase
    KVCacheMethodCls: type = BaseKVCacheMethod

    def __init__(
        self,
        exclude_modules: list[str],
    ):
        super().__init__()
        self.exclude_modules: list[str] = exclude_modules

    def is_layer_excluded(self, prefix: str) -> bool:
        """
        Check if a layer should be excluded from quantization.

        Handles both exact matching (for fused layers) and ModelOpt wildcard matching.

        The ModelOpt exclude_modules list is a list of wildcards.
        """
        if len(self.exclude_modules) == 0:
            return False

        # First check exact matching with fused layer support
        if is_layer_skipped(prefix, self.exclude_modules, self.packed_modules_mapping):
```
**EN:** Defines class `ModelOptQuantConfigBase` with base classes `QuantizationConfig` and decorators none. It acts as a quantization-oriented module building block and exposes 8 direct methods, with notable entries `__init__`, `is_layer_excluded`, `get_quant_method`, `apply_vllm_mapper`, `_extract_modelopt_quant_algo`, `get_config_filenames`.
**CN:** 定义类 `ModelOptQuantConfigBase`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 8 个方法，较重要的包括 `__init__`, `is_layer_excluded`, `get_quant_method`, `apply_vllm_mapper`, `_extract_modelopt_quant_algo`, `get_config_filenames`。

### Method `ModelOptQuantConfigBase.__init__` (lines 136-141)
```python
    def __init__(
        self,
        exclude_modules: list[str],
    ):
        super().__init__()
        self.exclude_modules: list[str] = exclude_modules
```
**EN:** Defines function `ModelOptQuantConfigBase.__init__` with signature `__init__(self, exclude_modules: list[str])`. It mainly works with `exclude_modules`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `ModelOptQuantConfigBase.__init__`，其签名为 `__init__(self, exclude_modules: list[str])`。它主要围绕 `exclude_modules` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `ModelOptQuantConfigBase.get_quant_method` (lines 181-216)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> "QuantizeMethodBase | None":
        # handle kv-cache first so we can focus only on weight quantization thereafter
        if isinstance(layer, (Attention, MLAAttention)):
            return self.KVCacheMethodCls(self)

        # handle exclusion
        if self.is_layer_excluded(prefix):
            if isinstance(layer, LinearBase):
                return UnquantizedLinearMethod()
            return None

        # TODO: This special hard coded logic is not needed for quantized checkpoints
        # generated by ModelOpt >= 0.39.0 where they are handled natually by the
        # exclude_modules config. But need to keep them for loading quantized
        # checkpoints generated by older versions. Then check substring matching
        # for patterns not caught by exact match
        if "vision_tower" in prefix or "vision_model" in prefix:
            return UnquantizedLinearMethod()

        # now, the layer is quantized, handle it here
        if isinstance(layer, LinearBase):
            quant_method = self.LinearMethodCls(self)
            if getattr(quant_method, "backend", "") == "marlin":
                quant_method.marlin_input_dtype = get_marlin_input_dtype(prefix)
            return quant_method
        elif isinstance(layer, RoutedExperts):
            quant_method = self.FusedMoEMethodCls(
                quant_config=self, moe_config=layer.moe_config
            )
            if getattr(quant_method, "backend", "") == "marlin":
                quant_method.marlin_input_dtype = get_marlin_input_dtype(prefix)
            return quant_method

        return None
```
**EN:** Defines function `ModelOptQuantConfigBase.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching. Key calls include `isinstance`, `self.is_layer_excluded`, `self.KVCacheMethodCls`, `UnquantizedLinearMethod`, `self.LinearMethodCls`, `getattr`.
**CN:** 定义函数 `ModelOptQuantConfigBase.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `isinstance`, `self.is_layer_excluded`, `self.KVCacheMethodCls`, `UnquantizedLinearMethod`, `self.LinearMethodCls`, `getattr`。

### Method `ModelOptQuantConfigBase.apply_vllm_mapper` (lines 218-237)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper: "WeightsMapper"):
        if len(self.exclude_modules) > 0:
            # This is a workaround for the weights remapping issue:
            # https://github.com/vllm-project/vllm/issues/28072
            # Right now, the Nvidia ModelOpt library use just one wildcard pattern:
            #        module_path*
            # It gets applied if the whole tree of modules rooted at module_path
            # is not quantized. Here we replace such pattern by 2 patterns that are
            # collectively equivalent to the original pattern:
            #        module_path
            #        module_path.*
            new_exclude_modules = []
            for exclude in self.exclude_modules:
                if len(exclude) >= 2 and exclude[-1] == "*" and exclude[-2] != ".":
                    new_exclude_modules.append(exclude[:-1])
                    new_exclude_modules.append(exclude[:-1] + ".*")
                else:
                    new_exclude_modules.append(exclude)

            self.exclude_modules = hf_to_vllm_mapper.apply_list(new_exclude_modules)
```
**EN:** Defines function `ModelOptQuantConfigBase.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses branching, iteration. Key calls include `len`, `hf_to_vllm_mapper.apply_list`, `new_exclude_modules.append`.
**CN:** 定义函数 `ModelOptQuantConfigBase.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `len`, `hf_to_vllm_mapper.apply_list`, `new_exclude_modules.append`。

### Method `ModelOptQuantConfigBase.from_config` (lines 276-361)
```python
    def from_config(cls, config: dict[str, Any]) -> "ModelOptQuantConfigBase":
        # Handle both ModelOpt format and compressed-tensors style format
        if "quantization" in config:
            # Traditional ModelOpt format:
            # {"quantization": {"quant_algo": "..."}}
            quant_config = cls.get_from_keys(config, ["quantization"])
            if not isinstance(quant_config, dict):
                raise ValueError("Expected 'quantization' to be a dictionary in config")

            quant_method = quant_config.get("quant_algo")

            # Handle kv_cache_quant_algo with proper type validation
            kv_cache_quant_method = quant_config.get("kv_cache_quant_algo")

            # Handle group_size with proper type validation
            group_size_raw = quant_config.get("group_size")

            # "exclude_modules" is the key in the legacy hf_quant_config.json
            exclude_modules = quant_config.get("exclude_modules", [])
        else:
            # Compressed-tensors style format (config.json quantization_config):
            # {"quant_algo": "...", "quant_method": "modelopt"}
            quant_method = config.get("quant_algo")

            # "kv_cache_scheme" (a dict) instead of "kv_cache_quant_algo" (a string).
            kv_cache_scheme = config.get("kv_cache_scheme")
            if isinstance(kv_cache_scheme, dict) and (
                kv_cache_scheme.get("type") == "float"
                and kv_cache_scheme.get("num_bits") == 8
            ):
                kv_cache_quant_method = "FP8"
            else:
                kv_cache_quant_method = None

            # "ignore" is the key in config.json
            exclude_modules = config.get("ignore", [])
            group_size_raw = config.get("group_size")

# ... truncated for analysis ...
        if quant_method not in QUANT_ALGOS:
            raise ValueError(
                f"ModelOpt currently only supports: {QUANT_ALGOS} "
                "quantizations in vLLM. Please check the "
                "`hf_quant_config.json` file for your model's "
                "quant configuration."
            )
        return cls._from_config(
            quant_method=quant_method,
            kv_cache_quant_method=kv_cache_quant_method,
            exclude_modules=exclude_modules,
            group_size=group_size,
            original_config=config,
        )
```
**EN:** Defines function `ModelOptQuantConfigBase.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'ModelOptQuantConfigBase'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `str.upper`, `cls._from_config`, `cls.get_from_keys`, `quant_config.get`, `config.get`, `ValueError`.
**CN:** 定义函数 `ModelOptQuantConfigBase.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'ModelOptQuantConfigBase'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `str.upper`, `cls._from_config`, `cls.get_from_keys`, `quant_config.get`, `config.get`, `ValueError`。

### Class `ModelOptFp8Config` overview (lines 364-435)
```python
class ModelOptFp8Config(ModelOptQuantConfigBase):
    """Config class for ModelOpt FP8."""

    def __init__(
        self,
        quant_method: str,
        is_checkpoint_fp8_serialized: bool,
        kv_cache_quant_method: str | None,
        exclude_modules: list[str],
    ) -> None:
        super().__init__(exclude_modules)
        self.quant_method = quant_method
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
        self.kv_cache_quant_method = kv_cache_quant_method
        if is_checkpoint_fp8_serialized:
            logger.warning(
                "Detected ModelOpt fp8 checkpoint (quant_algo=%s). Please note "
                "that the format is experimental and could change.",
                quant_method,
            )

        # Select LinearMethod implementation based on quant_algo.
        if self.quant_method == "FP8":
            self.LinearMethodCls = ModelOptFp8LinearMethod
        elif self.quant_method == "FP8_PER_CHANNEL_PER_TOKEN":
```
**EN:** Defines class `ModelOptFp8Config` with base classes `ModelOptQuantConfigBase` and decorators none. It acts as a configuration holder and backend selector and exposes 6 direct methods, with notable entries `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`. Its docstring says: Config class for ModelOpt FP8.
**CN:** 定义类 `ModelOptFp8Config`，其基类为 `ModelOptQuantConfigBase`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 6 个方法，较重要的包括 `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptFp8Config.__init__` (lines 367-397)
```python
    def __init__(
        self,
        quant_method: str,
        is_checkpoint_fp8_serialized: bool,
        kv_cache_quant_method: str | None,
        exclude_modules: list[str],
    ) -> None:
        super().__init__(exclude_modules)
        self.quant_method = quant_method
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
        self.kv_cache_quant_method = kv_cache_quant_method
        if is_checkpoint_fp8_serialized:
            logger.warning(
                "Detected ModelOpt fp8 checkpoint (quant_algo=%s). Please note "
                "that the format is experimental and could change.",
                quant_method,
            )

        # Select LinearMethod implementation based on quant_algo.
        if self.quant_method == "FP8":
            self.LinearMethodCls = ModelOptFp8LinearMethod
        elif self.quant_method == "FP8_PER_CHANNEL_PER_TOKEN":
            self.LinearMethodCls = ModelOptFp8PcPtLinearMethod
        elif self.quant_method == "FP8_PB_WO":
            self.LinearMethodCls = ModelOptFp8PbWoLinearMethod
        else:
            raise ValueError(
                "Unsupported ModelOpt FP8 quant_algo for vLLM: "
                f"{self.quant_method}. Supported: FP8 / "
                "FP8_PER_CHANNEL_PER_TOKEN / FP8_PB_WO."
            )
```
**EN:** Defines function `ModelOptFp8Config.__init__` with signature `__init__(self, quant_method: str, is_checkpoint_fp8_serialized: bool, kv_cache_quant_method: str | None, exclude_modules: list[str]) -> None`. It mainly works with `quant_method`, `is_checkpoint_fp8_serialized`, `kv_cache_quant_method`, `exclude_modules`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `logger.warning`, `super`, `ValueError`.
**CN:** 定义函数 `ModelOptFp8Config.__init__`，其签名为 `__init__(self, quant_method: str, is_checkpoint_fp8_serialized: bool, kv_cache_quant_method: str | None, exclude_modules: list[str]) -> None`。它主要围绕 `quant_method`, `is_checkpoint_fp8_serialized`, `kv_cache_quant_method`, `exclude_modules` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `logger.warning`, `super`, `ValueError`。

### Method `ModelOptFp8Config.get_name` (lines 399-400)
```python
    def get_name(self) -> QuantizationMethods:
        return "modelopt"
```
**EN:** Defines function `ModelOptFp8Config.get_name` with signature `get_name(self) -> QuantizationMethods`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptFp8Config.get_name`，其签名为 `get_name(self) -> QuantizationMethods`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ModelOptFp8Config.get_supported_act_dtypes` (lines 402-403)
```python
    def get_supported_act_dtypes(self) -> list[torch.dtype]:
        return [torch.bfloat16, torch.half]
```
**EN:** Defines function `ModelOptFp8Config.get_supported_act_dtypes` with signature `get_supported_act_dtypes(self) -> list[torch.dtype]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptFp8Config.get_supported_act_dtypes`，其签名为 `get_supported_act_dtypes(self) -> list[torch.dtype]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ModelOptFp8Config.get_min_capability` (lines 406-407)
```python
    def get_min_capability(cls) -> int:
        return 89
```
**EN:** Defines function `ModelOptFp8Config.get_min_capability` with signature `get_min_capability(cls) -> int`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptFp8Config.get_min_capability`，其签名为 `get_min_capability(cls) -> int`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `ModelOptFp8LinearMethod` overview (lines 438-530)
```python
class ModelOptFp8LinearMethod(LinearMethodBase):
    """Linear method for Model Optimizer static quantization.
    Supports loading FP8 checkpoints with static weight scale and
    activation scale. Future support might be added for dynamic
    scales.

    Limitations:
    1. Only support per-tensor quantization due to torch._scaled_mm support.
    2. Only support float8_e4m3fn datatype
        Args: quant_config: The ModelOpt quantization config.
    """

    def __init__(self, quant_config: ModelOptFp8Config) -> None:
        self.quant_config = quant_config
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype

    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
```
**EN:** Defines class `ModelOptFp8LinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for Model Optimizer static quantization.
**CN:** 定义类 `ModelOptFp8LinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptFp8LinearMethod.__init__` (lines 450-453)
```python
    def __init__(self, quant_config: ModelOptFp8Config) -> None:
        self.quant_config = quant_config
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype
```
**EN:** Defines function `ModelOptFp8LinearMethod.__init__` with signature `__init__(self, quant_config: ModelOptFp8Config) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `torch.get_default_dtype`, `get_current_vllm_config`.
**CN:** 定义函数 `ModelOptFp8LinearMethod.__init__`，其签名为 `__init__(self, quant_config: ModelOptFp8Config) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `torch.get_default_dtype`, `get_current_vllm_config`。

### Method `ModelOptFp8LinearMethod.create_weights` (lines 455-510)
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
        del input_size, output_size
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        weight_dtype = (
            torch.float8_e4m3fn
            if self.quant_config.is_checkpoint_fp8_serialized
            else params_dtype
        )
        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition, input_size_per_partition, dtype=weight_dtype
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        if self.quant_config.is_checkpoint_fp8_serialized:
            # WEIGHT SCALE
            weight_scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )
            weight_scale[:] = torch.finfo(torch.float32).min
            layer.register_parameter("weight_scale", weight_scale)
            # INPUT SCALE
            scale = PerTensorScaleParameter(
                data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                weight_loader=weight_loader,
            )

            scale[:] = torch.finfo(torch.float32).min
            layer.register_parameter("input_scale", scale)

        self.fp8_linear = init_fp8_linear_kernel(
            activation_quant_key=kFp8StaticTensorSym,
            weight_quant_key=kFp8StaticTensorSym,
            weight_shape=layer.weight.shape,
            input_dtype=self.input_dtype,
            out_dtype=self.out_dtype,
            module_name=self.__class__.__name__,
        )
```
**EN:** Defines function `ModelOptFp8LinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `init_fp8_linear_kernel`, `PerTensorScaleParameter`.
**CN:** 定义函数 `ModelOptFp8LinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `init_fp8_linear_kernel`, `PerTensorScaleParameter`。

### Method `ModelOptFp8LinearMethod.process_weights_after_loading` (lines 512-522)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        weight = layer.weight
        max_w_scale = layer.weight_scale.max()
        if not (layer.weight_scale == layer.weight_scale[0]).all():
            max_w_scale, weight = requantize_with_max_scale(
                layer.weight, layer.weight_scale, layer.logical_widths
            )
        layer.weight = Parameter(weight.t(), requires_grad=False)
        layer.weight_scale = Parameter(max_w_scale, requires_grad=False)
        layer.input_scale = Parameter(layer.input_scale.max(), requires_grad=False)
        self.fp8_linear.process_weights_after_loading(layer)
```
**EN:** Defines function `ModelOptFp8LinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching. Key calls include `layer.weight_scale.max`, `Parameter`, `self.fp8_linear.process_weights_after_loading`, `all`, `requantize_with_max_scale`, `weight.t`.
**CN:** 定义函数 `ModelOptFp8LinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断。关键调用包括 `layer.weight_scale.max`, `Parameter`, `self.fp8_linear.process_weights_after_loading`, `all`, `requantize_with_max_scale`, `weight.t`。

### Method `ModelOptFp8LinearMethod.apply` (lines 524-530)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.fp8_linear.apply_weights(layer, x, bias)
```
**EN:** Defines function `ModelOptFp8LinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.fp8_linear.apply_weights`.
**CN:** 定义函数 `ModelOptFp8LinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.fp8_linear.apply_weights`。

### Class `ModelOptFp8PcPtLinearMethod` overview (lines 533-611)
```python
class ModelOptFp8PcPtLinearMethod(LinearMethodBase):
    """Linear method for ModelOpt FP8_PER_CHANNEL_PER_TOKEN checkpoints.

    Expected checkpoint structure (per Linear):
    - weight: fp8-e4m3fn, shape [out, in]
    - weight_scale: fp32, shape [out] (per-output-channel)
    - no input_scale (activations are dynamically quantized per-token)
    """

    def __init__(self, quant_config: ModelOptFp8Config) -> None:
        self.quant_config = quant_config
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype

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
        del input_size, output_size
```
**EN:** Defines class `ModelOptFp8PcPtLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for ModelOpt FP8_PER_CHANNEL_PER_TOKEN checkpoints.
**CN:** 定义类 `ModelOptFp8PcPtLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptFp8PcPtLinearMethod.__init__` (lines 542-545)
```python
    def __init__(self, quant_config: ModelOptFp8Config) -> None:
        self.quant_config = quant_config
        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype
```
**EN:** Defines function `ModelOptFp8PcPtLinearMethod.__init__` with signature `__init__(self, quant_config: ModelOptFp8Config) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `torch.get_default_dtype`, `get_current_vllm_config`.
**CN:** 定义函数 `ModelOptFp8PcPtLinearMethod.__init__`，其签名为 `__init__(self, quant_config: ModelOptFp8Config) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `torch.get_default_dtype`, `get_current_vllm_config`。

### Method `ModelOptFp8PcPtLinearMethod.create_weights` (lines 547-598)
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
        del input_size, output_size

        if not self.quant_config.is_checkpoint_fp8_serialized:
            raise ValueError(
                "FP8_PER_CHANNEL_PER_TOKEN currently only supports "
                "FP8-serialized checkpoints."
            )

        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=torch.float8_e4m3fn,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        weight_scale = ChannelQuantScaleParameter(
            data=torch.empty(output_size_per_partition, dtype=torch.float32),
            output_dim=0,
            weight_loader=weight_loader,
        )
        weight_scale[:] = torch.finfo(torch.float32).min
        layer.register_parameter("weight_scale", weight_scale)

        self.fp8_linear = init_fp8_linear_kernel(
            activation_quant_key=kFp8DynamicTokenSym,
            weight_quant_key=kFp8StaticTokenSym,
            weight_shape=layer.weight.shape,
            input_dtype=self.input_dtype,
            out_dtype=self.out_dtype,
            module_name=self.__class__.__name__,
        )
```
**EN:** Defines function `ModelOptFp8PcPtLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `ChannelQuantScaleParameter`, `init_fp8_linear_kernel`.
**CN:** 定义函数 `ModelOptFp8PcPtLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `ChannelQuantScaleParameter`, `init_fp8_linear_kernel`。

### Method `ModelOptFp8PcPtLinearMethod.process_weights_after_loading` (lines 600-603)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.weight = Parameter(layer.weight.t(), requires_grad=False)
        layer.weight_scale = Parameter(layer.weight_scale.data, requires_grad=False)
        self.fp8_linear.process_weights_after_loading(layer)
```
**EN:** Defines function `ModelOptFp8PcPtLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses mostly straightforward data movement and object wiring. Key calls include `Parameter`, `self.fp8_linear.process_weights_after_loading`, `layer.weight.t`.
**CN:** 定义函数 `ModelOptFp8PcPtLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `Parameter`, `self.fp8_linear.process_weights_after_loading`, `layer.weight.t`。

### Method `ModelOptFp8PcPtLinearMethod.apply` (lines 605-611)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.fp8_linear.apply_weights(layer, x, bias)
```
**EN:** Defines function `ModelOptFp8PcPtLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.fp8_linear.apply_weights`.
**CN:** 定义函数 `ModelOptFp8PcPtLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.fp8_linear.apply_weights`。

### Class `ModelOptFp8PbWoLinearMethod` overview (lines 614-740)
```python
class ModelOptFp8PbWoLinearMethod(LinearMethodBase):
    """Linear method for ModelOpt FP8_PB_WO checkpoints.

    ModelOpt exports `weight_scale` as a 4D tensor:
      [out_blk, 1, in_blk, 1]
    where block size is typically 128 for both dims.

    vLLM executes it as FP8 GEMM with *dynamic per-token* activation quant.
    """

    _WEIGHT_BLOCK_SIZE: tuple[int, int] = (128, 128)

    def __init__(self, quant_config: ModelOptFp8Config) -> None:
        self.quant_config = quant_config
        block_n, block_k = self._WEIGHT_BLOCK_SIZE
        self.weight_block_size = list(self._WEIGHT_BLOCK_SIZE)

        self.activation_quant_key = create_fp8_quant_key(
            static=False, group_shape=GroupShape(1, block_k)
        )
        self.weight_quant_key = create_fp8_quant_key(
            static=True, group_shape=GroupShape(block_n, block_k)
        )

        self.out_dtype = torch.get_default_dtype()
```
**EN:** Defines class `ModelOptFp8PbWoLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for ModelOpt FP8_PB_WO checkpoints.
**CN:** 定义类 `ModelOptFp8PbWoLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptFp8PbWoLinearMethod.__init__` (lines 626-639)
```python
    def __init__(self, quant_config: ModelOptFp8Config) -> None:
        self.quant_config = quant_config
        block_n, block_k = self._WEIGHT_BLOCK_SIZE
        self.weight_block_size = list(self._WEIGHT_BLOCK_SIZE)

        self.activation_quant_key = create_fp8_quant_key(
            static=False, group_shape=GroupShape(1, block_k)
        )
        self.weight_quant_key = create_fp8_quant_key(
            static=True, group_shape=GroupShape(block_n, block_k)
        )

        self.out_dtype = torch.get_default_dtype()
        self.input_dtype = get_current_vllm_config().model_config.dtype
```
**EN:** Defines function `ModelOptFp8PbWoLinearMethod.__init__` with signature `__init__(self, quant_config: ModelOptFp8Config) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `list`, `create_fp8_quant_key`, `torch.get_default_dtype`, `GroupShape`, `get_current_vllm_config`.
**CN:** 定义函数 `ModelOptFp8PbWoLinearMethod.__init__`，其签名为 `__init__(self, quant_config: ModelOptFp8Config) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `list`, `create_fp8_quant_key`, `torch.get_default_dtype`, `GroupShape`, `get_current_vllm_config`。

### Method `ModelOptFp8PbWoLinearMethod.create_weights` (lines 641-713)
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
        del input_size, output_size

        if not self.quant_config.is_checkpoint_fp8_serialized:
            raise ValueError(
                "FP8_PB_WO currently only supports FP8-serialized checkpoints."
            )

        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        # Expose block size so the v2 weight loaders can translate offsets from
        # element-space -> block-space for BlockQuantScaleParameter.
        layer.weight_block_size = self.weight_block_size

        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=torch.float8_e4m3fn,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)
# ... truncated for analysis ...
            output_dim=0,
            weight_loader=weight_loader,
        )
        weight_scale[:] = torch.finfo(torch.float32).min
        layer.register_parameter("weight_scale", weight_scale)

        self.w8a8_block_fp8_linear = init_fp8_linear_kernel(
            activation_quant_key=self.activation_quant_key,
            weight_quant_key=self.weight_quant_key,
            weight_shape=layer.weight.shape,
            input_dtype=self.input_dtype,
            out_dtype=self.out_dtype,
            module_name=self.__class__.__name__,
        )
```
**EN:** Defines function `ModelOptFp8PbWoLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `BlockQuantScaleParameter`, `init_fp8_linear_kernel`.
**CN:** 定义函数 `ModelOptFp8PbWoLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `BlockQuantScaleParameter`, `init_fp8_linear_kernel`。

### Method `ModelOptFp8PbWoLinearMethod.process_weights_after_loading` (lines 715-732)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Keep weight in [out, in] layout for Fp8BlockScaledMMLinearKernel.
        layer.weight = Parameter(layer.weight.data, requires_grad=False)

        scale = layer.weight_scale
        if scale.dim() == 4:
            # [out_blk, 1, in_blk, 1] -> [out_blk, in_blk]
            scale = scale.squeeze(1).squeeze(-1)
        elif scale.dim() != 2:
            raise ValueError(
                "Unexpected ModelOpt FP8_PB_WO weight_scale shape: "
                f"{tuple(scale.shape)}."
            )

        layer.weight_scale = Parameter(scale.contiguous(), requires_grad=False)

        if hasattr(self, "fp8_linear"):
            self.fp8_linear.process_weights_after_loading(layer)
```
**EN:** Defines function `ModelOptFp8PbWoLinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, validation/error handling. Key calls include `Parameter`, `hasattr`, `scale.dim`, `scale.squeeze.squeeze`, `scale.contiguous`, `self.fp8_linear.process_weights_after_loading`.
**CN:** 定义函数 `ModelOptFp8PbWoLinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、校验或报错逻辑。关键调用包括 `Parameter`, `hasattr`, `scale.dim`, `scale.squeeze.squeeze`, `scale.contiguous`, `self.fp8_linear.process_weights_after_loading`。

### Method `ModelOptFp8PbWoLinearMethod.apply` (lines 734-740)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.w8a8_block_fp8_linear.apply_weights(layer, x, bias)
```
**EN:** Defines function `ModelOptFp8PbWoLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.w8a8_block_fp8_linear.apply_weights`.
**CN:** 定义函数 `ModelOptFp8PbWoLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.w8a8_block_fp8_linear.apply_weights`。

### Class `ModelOptFp8MoEMethod` overview (lines 743-997)
```python
class ModelOptFp8MoEMethod(FusedMoEMethodBase):
    """MoE method for ModelOpt FP8.
    Supports loading FP8 checkpoints with static weight scale and
    activation scale.
    Args:
        quant_config: The ModelOpt quantization config.
    """

    def __init__(
        self,
        quant_config: ModelOptFp8Config,
        moe_config: FusedMoEConfig,
    ) -> None:
        super().__init__(moe_config)
        self.quant_config = quant_config
        assert self.quant_config.is_checkpoint_fp8_serialized

        # Select Fp8 MoE backend
        self.fp8_backend, self.experts_cls = select_fp8_moe_backend(
            config=self.moe,
            weight_key=kFp8StaticTensorSym,
            activation_key=kFp8StaticTensorSym,
        )

    def maybe_make_prepare_finalize(
```
**EN:** Defines class `ModelOptFp8MoEMethod` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 9 direct methods, with notable entries `__init__`, `maybe_make_prepare_finalize`, `select_gemm_impl`, `create_weights`, `_setup_kernel`, `process_weights_after_loading`. Its docstring says: MoE method for ModelOpt FP8.
**CN:** 定义类 `ModelOptFp8MoEMethod`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 9 个方法，较重要的包括 `__init__`, `maybe_make_prepare_finalize`, `select_gemm_impl`, `create_weights`, `_setup_kernel`, `process_weights_after_loading`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptFp8MoEMethod.__init__` (lines 751-765)
```python
    def __init__(
        self,
        quant_config: ModelOptFp8Config,
        moe_config: FusedMoEConfig,
    ) -> None:
        super().__init__(moe_config)
        self.quant_config = quant_config
        assert self.quant_config.is_checkpoint_fp8_serialized

        # Select Fp8 MoE backend
        self.fp8_backend, self.experts_cls = select_fp8_moe_backend(
            config=self.moe,
            weight_key=kFp8StaticTensorSym,
            activation_key=kFp8StaticTensorSym,
        )
```
**EN:** Defines function `ModelOptFp8MoEMethod.__init__` with signature `__init__(self, quant_config: ModelOptFp8Config, moe_config: FusedMoEConfig) -> None`. It mainly works with `quant_config`, `moe_config`; initializes the object state and cached resources. The body uses validation/error handling. Key calls include `super.__init__`, `select_fp8_moe_backend`, `super`.
**CN:** 定义函数 `ModelOptFp8MoEMethod.__init__`，其签名为 `__init__(self, quant_config: ModelOptFp8Config, moe_config: FusedMoEConfig) -> None`。它主要围绕 `quant_config`, `moe_config` 展开；负责初始化对象状态和缓存资源。函数体包含校验或报错逻辑。关键调用包括 `super.__init__`, `select_fp8_moe_backend`, `super`。

### Method `ModelOptFp8MoEMethod.create_weights` (lines 786-863)
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
        layer.orig_dtype = params_dtype
        layer.num_experts = num_experts

        # Use FP8 dtype if checkpoint is serialized
        weight_dtype = (
            torch.float8_e4m3fn
            if self.quant_config.is_checkpoint_fp8_serialized
            else params_dtype
        )
        weight_loader = extra_weight_attrs.get("weight_loader")

        w13_num_shards = 2 if self.moe.is_act_and_mul else 1

        w13_weight = ModelWeightParameter(
            data=torch.empty(
                num_experts,
                w13_num_shards * intermediate_size_per_partition,
                hidden_size,
                dtype=weight_dtype,
            ),
            input_dim=2,
            output_dim=1,
            weight_loader=weight_loader,
        )
        layer.register_parameter("w13_weight", w13_weight)

        w2_weight = ModelWeightParameter(
            data=torch.empty(
                num_experts,
# ... truncated for analysis ...
        layer.register_parameter("w13_weight_scale", w13_weight_scale)
        layer.register_parameter("w2_weight_scale", w2_weight_scale)

        # INPUT SCALES - Per-tensor scaling for ModelOpt
        w13_input_scale = PerTensorScaleParameter(
            data=torch.full((num_experts,), 1.0, dtype=torch.float32),
            weight_loader=weight_loader,
        )
        w2_input_scale = PerTensorScaleParameter(
            data=torch.full((num_experts,), 1.0, dtype=torch.float32),
            weight_loader=weight_loader,
        )
        layer.register_parameter("w13_input_scale", w13_input_scale)
        layer.register_parameter("w2_input_scale", w2_input_scale)
```
**EN:** Defines function `ModelOptFp8MoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `PerTensorScaleParameter`, `torch.empty`, `torch.full`.
**CN:** 定义函数 `ModelOptFp8MoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `PerTensorScaleParameter`, `torch.empty`, `torch.full`。

### Method `ModelOptFp8MoEMethod.apply_monolithic` (lines 950-972)
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
**EN:** Defines function `ModelOptFp8MoEMethod.apply_monolithic` with signature `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `router_logits`, `input_ids`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply_monolithic`.
**CN:** 定义函数 `ModelOptFp8MoEMethod.apply_monolithic`，其签名为 `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `router_logits`, `input_ids` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply_monolithic`。

### Method `ModelOptFp8MoEMethod.apply` (lines 974-997)
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
**EN:** Defines function `ModelOptFp8MoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply`.
**CN:** 定义函数 `ModelOptFp8MoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply`。

### Class `ModelOptNvFp4Config` overview (lines 1005-1100)
```python
class ModelOptNvFp4Config(ModelOptQuantConfigBase):
    """Config class for ModelOpt FP4."""

    def __init__(
        self,
        quant_method: str = "NVFP4",
        is_checkpoint_nvfp4_serialized: bool = False,
        kv_cache_quant_algo: str | None = None,
        exclude_modules: list[str] | None = None,
        group_size: int = 16,
    ) -> None:
        if exclude_modules is None:
            exclude_modules = []
        super().__init__(exclude_modules)
        self.quant_method = quant_method
        self.is_checkpoint_nvfp4_serialized = is_checkpoint_nvfp4_serialized
        if is_checkpoint_nvfp4_serialized:
            logger.warning(
                "Detected ModelOpt NVFP4 checkpoint (quant_algo=%s). Please "
                "note that the format is experimental and could change in "
                "future.",
                quant_method,
            )

            self.group_size = group_size
```
**EN:** Defines class `ModelOptNvFp4Config` with base classes `ModelOptQuantConfigBase` and decorators none. It acts as a configuration holder and backend selector and exposes 6 direct methods, with notable entries `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`. Its docstring says: Config class for ModelOpt FP4.
**CN:** 定义类 `ModelOptNvFp4Config`，其基类为 `ModelOptQuantConfigBase`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 6 个方法，较重要的包括 `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptNvFp4Config.__init__` (lines 1008-1043)
```python
    def __init__(
        self,
        quant_method: str = "NVFP4",
        is_checkpoint_nvfp4_serialized: bool = False,
        kv_cache_quant_algo: str | None = None,
        exclude_modules: list[str] | None = None,
        group_size: int = 16,
    ) -> None:
        if exclude_modules is None:
            exclude_modules = []
        super().__init__(exclude_modules)
        self.quant_method = quant_method
        self.is_checkpoint_nvfp4_serialized = is_checkpoint_nvfp4_serialized
        if is_checkpoint_nvfp4_serialized:
            logger.warning(
                "Detected ModelOpt NVFP4 checkpoint (quant_algo=%s). Please "
                "note that the format is experimental and could change in "
                "future.",
                quant_method,
            )

            self.group_size = group_size
            self.kv_cache_quant_algo = kv_cache_quant_algo

        # Select LinearMethod implementation based on quant_algo (FP8 pattern).
        # NVFP4         -> W4A4: cutlass NVFP4 GEMM with input quantization
        # W4A16_NVFP4   -> W4A16: FP4 Marlin GEMM with bf16/fp16 activations
        if quant_method == "NVFP4":
            self.LinearMethodCls = ModelOptNvFp4LinearMethod
        elif quant_method == "W4A16_NVFP4":
            self.LinearMethodCls = ModelOptNvFp4W4A16LinearMethod
        else:
            raise ValueError(
                f"Unsupported ModelOpt NVFP4 quant_algo: {quant_method}. "
                "Supported: NVFP4 / W4A16_NVFP4."
            )
```
**EN:** Defines function `ModelOptNvFp4Config.__init__` with signature `__init__(self, quant_method: str='NVFP4', is_checkpoint_nvfp4_serialized: bool=False, kv_cache_quant_algo: str | None=None, exclude_modules: list[str] | None=None, group_size: int=16) -> None`. It mainly works with `quant_method`, `is_checkpoint_nvfp4_serialized`, `kv_cache_quant_algo`, `exclude_modules`, `group_size`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `logger.warning`, `super`, `ValueError`.
**CN:** 定义函数 `ModelOptNvFp4Config.__init__`，其签名为 `__init__(self, quant_method: str='NVFP4', is_checkpoint_nvfp4_serialized: bool=False, kv_cache_quant_algo: str | None=None, exclude_modules: list[str] | None=None, group_size: int=16) -> None`。它主要围绕 `quant_method`, `is_checkpoint_nvfp4_serialized`, `kv_cache_quant_algo`, `exclude_modules`, `group_size` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `logger.warning`, `super`, `ValueError`。

### Method `ModelOptNvFp4Config.get_name` (lines 1045-1046)
```python
    def get_name(self) -> QuantizationMethods:
        return "modelopt_fp4"
```
**EN:** Defines function `ModelOptNvFp4Config.get_name` with signature `get_name(self) -> QuantizationMethods`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptNvFp4Config.get_name`，其签名为 `get_name(self) -> QuantizationMethods`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ModelOptNvFp4Config.get_supported_act_dtypes` (lines 1048-1049)
```python
    def get_supported_act_dtypes(self) -> list[torch.dtype]:
        return [torch.bfloat16, torch.half, torch.float8_e4m3fn]
```
**EN:** Defines function `ModelOptNvFp4Config.get_supported_act_dtypes` with signature `get_supported_act_dtypes(self) -> list[torch.dtype]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptNvFp4Config.get_supported_act_dtypes`，其签名为 `get_supported_act_dtypes(self) -> list[torch.dtype]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ModelOptNvFp4Config.get_min_capability` (lines 1052-1053)
```python
    def get_min_capability(cls) -> int:
        return 75
```
**EN:** Defines function `ModelOptNvFp4Config.get_min_capability` with signature `get_min_capability(cls) -> int`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptNvFp4Config.get_min_capability`，其签名为 `get_min_capability(cls) -> int`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `ModelOptNvFp4LinearMethod` overview (lines 1103-1232)
```python
class ModelOptNvFp4LinearMethod(LinearMethodBase):
    """Linear method for Model Optimizer NVFP4.
    Supports loading NVFP4 checkpoints with the following structure:

    input_scale: torch.float32, scalar ,
    weight: NVFP4(represented as byte) Shape: [1, X, y/2]
    weight_scale: FP8-E4M3, Shape: [X, Y], aka per block scale,
    weight_scale_2: torch.float32, scalar,
    Args: quant_config: The ModelOpt quantization config.
    """

    def __init__(self, quant_config: ModelOptNvFp4Config) -> None:
        self.quant_config = quant_config
        self.marlin_input_dtype = None
        self.kernel = init_nvfp4_linear_kernel()

    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
```
**EN:** Defines class `ModelOptNvFp4LinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for Model Optimizer NVFP4.
**CN:** 定义类 `ModelOptNvFp4LinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptNvFp4LinearMethod.__init__` (lines 1114-1117)
```python
    def __init__(self, quant_config: ModelOptNvFp4Config) -> None:
        self.quant_config = quant_config
        self.marlin_input_dtype = None
        self.kernel = init_nvfp4_linear_kernel()
```
**EN:** Defines function `ModelOptNvFp4LinearMethod.__init__` with signature `__init__(self, quant_config: ModelOptNvFp4Config) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `init_nvfp4_linear_kernel`.
**CN:** 定义函数 `ModelOptNvFp4LinearMethod.__init__`，其签名为 `__init__(self, quant_config: ModelOptNvFp4Config) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `init_nvfp4_linear_kernel`。

### Method `ModelOptNvFp4LinearMethod.create_weights` (lines 1119-1191)
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
        del input_size, output_size
        if not self.quant_config.is_checkpoint_nvfp4_serialized:
            raise ValueError(
                "NVFP4 quantization was selected, "
                " dynamic quantization is not supported."
            )
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        if input_size_per_partition % 16 != 0:
            raise ValueError(
                "Unsupported model when in features size is not multiple of 16"
            )
        # The nvfp4 weight is still represented as
        weight_dtype = (
            torch.float8_e4m3fn
            if self.quant_config.is_checkpoint_nvfp4_serialized
            else params_dtype
        )
        # Weight
        weight = ModelWeightParameter(
            data=torch.empty(
                # 2 fp4 items are packed in the input dimension
                layer.output_size_per_partition,
                layer.input_size_per_partition // 2,
# ... truncated for analysis ...

        # Per Block Weight Scale
        weight_scale = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // self.quant_config.group_size,
                dtype=weight_dtype,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )

        layer.register_parameter("weight_scale", weight_scale)
```
**EN:** Defines function `ModelOptNvFp4LinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `PerTensorScaleParameter`, `ValueError`.
**CN:** 定义函数 `ModelOptNvFp4LinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `PerTensorScaleParameter`, `ValueError`。

### Method `ModelOptNvFp4LinearMethod.process_weights_after_loading` (lines 1193-1224)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if (
            torch.unique(layer.input_scale).numel() != 1
            or torch.unique(layer.weight_scale_2).numel() != 1
        ):
            logger.warning_once(
                "In NVFP4 linear, the global scale for input or weight are different"
                " for parallel layers (e.g. q_proj, k_proj, v_proj). This "
                " will likely results in reduce accuracy. Please verify the model"
                " accuracy. Consider using a checkpoint with a shared global NVFP4"
                " scale for parallel layers."
            )

        # Rename ModelOpt checkpoint names to standardized names
        input_global_scale = layer.input_scale.max().to(torch.float32)
        layer.input_global_scale = Parameter(input_global_scale, requires_grad=False)
        del layer.input_scale

        weight_global_scale = layer.weight_scale_2.max().to(torch.float32)
        layer.weight_global_scale = Parameter(weight_global_scale, requires_grad=False)
        del layer.weight_scale_2

        # Pre-compute alpha and inverse for runtime quantization
        layer.alpha = Parameter(
            layer.input_global_scale * layer.weight_global_scale, requires_grad=False
        )
        layer.input_global_scale_inv = Parameter(
            (1.0 / layer.input_global_scale).to(torch.float32), requires_grad=False
        )

        # Convert layer to NVFP4 linear kernel format
        self.kernel.process_weights_after_loading(layer)
```
**EN:** Defines function `ModelOptNvFp4LinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `layer.input_scale.max.to`, `Parameter`, `layer.weight_scale_2.max.to`, `self.kernel.process_weights_after_loading`, `logger.warning_once`, `to`.
**CN:** 定义函数 `ModelOptNvFp4LinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `layer.input_scale.max.to`, `Parameter`, `layer.weight_scale_2.max.to`, `self.kernel.process_weights_after_loading`, `logger.warning_once`, `to`。

### Method `ModelOptNvFp4LinearMethod.apply` (lines 1226-1232)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.kernel.apply_weights(layer=layer, x=x, bias=bias)
```
**EN:** Defines function `ModelOptNvFp4LinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.kernel.apply_weights`.
**CN:** 定义函数 `ModelOptNvFp4LinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.kernel.apply_weights`。

### Class `ModelOptNvFp4W4A16LinearMethod` overview (lines 1235-1378)
```python
class ModelOptNvFp4W4A16LinearMethod(LinearMethodBase):
    """Linear method for ModelOpt NVFP4 W4A16.

    4-bit NVFP4 weights, fp16/bf16 activations. Loads ModelOpt-style names
    directly (no on-disk conversion) and dispatches to the FP4 Marlin GEMM:

        weight          uint8     packed NVFP4 (2 nibbles/byte along input dim)
        weight_scale    fp8-e4m3  per 16-elem group along input dim
        weight_scale_2  fp32      per-tensor global scale = amax / (6.0 * 448.0)

    No activation quantization. Marlin expects the global scale in the same
    form ModelOpt stores (amax/2688), so we rename weight_scale_2 ->
    weight_global_scale **without reciprocation** -- the CT W4A16 path
    reciprocates only because CT stores the inverse on disk.

    We also register a placeholder input_scale parameter so that W4A4-shaped
    checkpoints (which contain *_proj.input_scale tensors) can be loaded
    under this method without the per-shard loader hitting a KeyError on
    the merged-name lookup. The placeholder is discarded in
    process_weights_after_loading -- its value is never used.
    """

    def __init__(self, quant_config: ModelOptNvFp4Config) -> None:
        self.quant_config = quant_config
        # Vestigial slot mirrored from ModelOptNvFp4LinearMethod: the parent
```
**EN:** Defines class `ModelOptNvFp4W4A16LinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for ModelOpt NVFP4 W4A16.
**CN:** 定义类 `ModelOptNvFp4W4A16LinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptNvFp4W4A16LinearMethod.__init__` (lines 1257-1269)
```python
    def __init__(self, quant_config: ModelOptNvFp4Config) -> None:
        self.quant_config = quant_config
        # Vestigial slot mirrored from ModelOptNvFp4LinearMethod: the parent
        # config's get_quant_method only fills marlin_input_dtype when
        # backend == "marlin"; we don't set that since we pin the kernel
        # below, but we keep the attribute for shape parity.
        self.marlin_input_dtype = None
        # Direct-instantiate the Marlin NVFP4 adapter rather than going through
        # init_nvfp4_linear_kernel(): the latter's priority list returns a
        # cutlass W4A4 kernel as first-pick on this hardware, which would
        # silently try to quantize activations (we have no input_scale). For
        # W4A16 there is exactly one valid kernel, so we pin it.
        self.kernel = MarlinNvFp4LinearKernel(NvFp4LinearLayerConfig())
```
**EN:** Defines function `ModelOptNvFp4W4A16LinearMethod.__init__` with signature `__init__(self, quant_config: ModelOptNvFp4Config) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `MarlinNvFp4LinearKernel`, `NvFp4LinearLayerConfig`.
**CN:** 定义函数 `ModelOptNvFp4W4A16LinearMethod.__init__`，其签名为 `__init__(self, quant_config: ModelOptNvFp4Config) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `MarlinNvFp4LinearKernel`, `NvFp4LinearLayerConfig`。

### Method `ModelOptNvFp4W4A16LinearMethod.create_weights` (lines 1271-1344)
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
        del input_size, output_size
        if not self.quant_config.is_checkpoint_nvfp4_serialized:
            raise ValueError(
                "W4A16_NVFP4 quantization was selected; "
                "dynamic quantization is not supported."
            )
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        if input_size_per_partition % 16 != 0:
            raise ValueError(
                "Unsupported model: input feature size is not a multiple of 16."
            )

        # Packed NVFP4 weights: uint8, 2 nibbles per byte along the input dim.
        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // 2,
                dtype=torch.uint8,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
# ... truncated for analysis ...
        layer.register_parameter("weight_scale", weight_scale)

        # Placeholder input_scale param so W4A4-shaped checkpoints can be
        # loaded under this method without KeyError on the merged-name
        # lookup (qwen2-style stacked-loader path renames *_proj.input_scale
        # to e.g. qkv_proj.input_scale and looks it up unconditionally).
        # Discarded in process_weights_after_loading; never read by the kernel.
        # For native W4A16 checkpoints (no input_scale on disk) the param
        # stays uninitialized and is simply deleted.
        input_scale = PerTensorScaleParameter(
            data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
            weight_loader=weight_loader,
        )
        layer.register_parameter("input_scale", input_scale)
```
**EN:** Defines function `ModelOptNvFp4W4A16LinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `PerTensorScaleParameter`, `GroupQuantScaleParameter`.
**CN:** 定义函数 `ModelOptNvFp4W4A16LinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `PerTensorScaleParameter`, `GroupQuantScaleParameter`。

### Method `ModelOptNvFp4W4A16LinearMethod.process_weights_after_loading` (lines 1346-1370)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Discard the input_scale placeholder. Whether it carries values
        # (W4A4 ckpt loaded as W4A16) or is uninitialized (native W4A16
        # ckpt), W4A16 mode does not quantize activations, so this is unused.
        if hasattr(layer, "input_scale"):
            del layer.input_scale

        if torch.unique(layer.weight_scale_2).numel() != 1:
            logger.warning_once(
                "In W4A16_NVFP4 linear, the global weight scale "
                "(weight_scale_2) differs across fused parallel layers "
                "(e.g. q/k/v_proj). This will likely reduce accuracy. "
                "Consider a checkpoint with a shared global scale."
            )

        # Rename weight_scale_2 -> weight_global_scale. NO reciprocation:
        # ModelOpt already stores amax/2688, which is exactly what Marlin
        # consumes via nvfp4_marlin_process_global_scale (called inside the
        # Marlin adapter's process_weights_after_loading).
        layer.weight_global_scale = Parameter(
            layer.weight_scale_2.max().to(torch.float32), requires_grad=False
        )
        del layer.weight_scale_2

        self.kernel.process_weights_after_loading(layer)
```
**EN:** Defines function `ModelOptNvFp4W4A16LinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, tensor/kernel operations. Key calls include `hasattr`, `Parameter`, `self.kernel.process_weights_after_loading`, `torch.unique.numel`, `logger.warning_once`, `layer.weight_scale_2.max.to`.
**CN:** 定义函数 `ModelOptNvFp4W4A16LinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、张量或内核操作。关键调用包括 `hasattr`, `Parameter`, `self.kernel.process_weights_after_loading`, `torch.unique.numel`, `logger.warning_once`, `layer.weight_scale_2.max.to`。

### Method `ModelOptNvFp4W4A16LinearMethod.apply` (lines 1372-1378)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.kernel.apply_weights(layer=layer, x=x, bias=bias)
```
**EN:** Defines function `ModelOptNvFp4W4A16LinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.kernel.apply_weights`.
**CN:** 定义函数 `ModelOptNvFp4W4A16LinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.kernel.apply_weights`。

### Class `ModelOptNvFp4FusedMoE` overview (lines 1381-1657)
```python
class ModelOptNvFp4FusedMoE(FusedMoEMethodBase):
    """
    MoE Method for FP4 Quantization.
    Args:
        quant_config: NVFP4 Quant Config
    """

    def __init__(
        self,
        quant_config: ModelOptNvFp4Config,
        moe_config: FusedMoEConfig,
    ) -> None:
        super().__init__(moe_config)
        self.quant_config = quant_config
        # Select experts implementation.
        self.nvfp4_backend, self.experts_cls = select_nvfp4_moe_backend(
            config=self.moe,
            weight_key=kNvfp4Static,
            activation_key=kNvfp4Dynamic,
        )

        self.use_global_sf = is_global_sf_supported_for_nvfp4_backend(
            self.nvfp4_backend
        )
```
**EN:** Defines class `ModelOptNvFp4FusedMoE` with base classes `FusedMoEMethodBase` and decorators none. It acts as a quantization-oriented module building block and exposes 9 direct methods, with notable entries `__init__`, `maybe_make_prepare_finalize`, `uses_weight_scale_2_pattern`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`. Its docstring says: MoE Method for FP4 Quantization.
**CN:** 定义类 `ModelOptNvFp4FusedMoE`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 9 个方法，较重要的包括 `__init__`, `maybe_make_prepare_finalize`, `uses_weight_scale_2_pattern`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptNvFp4FusedMoE.__init__` (lines 1388-1404)
```python
    def __init__(
        self,
        quant_config: ModelOptNvFp4Config,
        moe_config: FusedMoEConfig,
    ) -> None:
        super().__init__(moe_config)
        self.quant_config = quant_config
        # Select experts implementation.
        self.nvfp4_backend, self.experts_cls = select_nvfp4_moe_backend(
            config=self.moe,
            weight_key=kNvfp4Static,
            activation_key=kNvfp4Dynamic,
        )

        self.use_global_sf = is_global_sf_supported_for_nvfp4_backend(
            self.nvfp4_backend
        )
```
**EN:** Defines function `ModelOptNvFp4FusedMoE.__init__` with signature `__init__(self, quant_config: ModelOptNvFp4Config, moe_config: FusedMoEConfig) -> None`. It mainly works with `quant_config`, `moe_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `select_nvfp4_moe_backend`, `is_global_sf_supported_for_nvfp4_backend`, `super`.
**CN:** 定义函数 `ModelOptNvFp4FusedMoE.__init__`，其签名为 `__init__(self, quant_config: ModelOptNvFp4Config, moe_config: FusedMoEConfig) -> None`。它主要围绕 `quant_config`, `moe_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `select_nvfp4_moe_backend`, `is_global_sf_supported_for_nvfp4_backend`, `super`。

### Method `ModelOptNvFp4FusedMoE.create_weights` (lines 1421-1535)
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
        assert self.quant_config.is_checkpoint_nvfp4_serialized

        layer.num_experts = num_experts
        layer.params_dtype = params_dtype
        layer.quant_config = self.quant_config
        weight_dtype = torch.uint8
        weight_scale_dtype = torch.float8_e4m3fn
        weight_loader = extra_weight_attrs.get("weight_loader")
        global_num_experts = extra_weight_attrs.get("global_num_experts")
        w13_num_shards = 2 if self.moe.is_act_and_mul else 1
        # GEMM 1
        w13_weight = ModelWeightParameter(
            data=torch.empty(
                num_experts,
                w13_num_shards * intermediate_size_per_partition,
                # 2 fp4 items are packed in the input dimension
                hidden_size // 2,
                dtype=weight_dtype,
            ),
            input_dim=1,
            output_dim=2,
            weight_loader=weight_loader,
        )
        layer.register_parameter("w13_weight", w13_weight)

        # GEMM 2
        w2_weight = ModelWeightParameter(
            data=torch.empty(
                num_experts,
# ... truncated for analysis ...
            data=torch.empty(
                global_sf_num_experts,
                w13_num_shards,
                dtype=torch.float32,
            ),
            weight_loader=weight_loader,
        )
        layer.register_parameter("w13_input_scale", w13_input_scale)

        w2_input_scale = PerTensorScaleParameter(
            data=torch.empty(global_sf_num_experts, dtype=torch.float32),
            weight_loader=weight_loader,
        )
        layer.register_parameter("w2_input_scale", w2_input_scale)
```
**EN:** Defines function `ModelOptNvFp4FusedMoE.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `extra_weight_attrs.update`, `PerTensorScaleParameter`, `torch.empty`.
**CN:** 定义函数 `ModelOptNvFp4FusedMoE.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `extra_weight_attrs.update`, `PerTensorScaleParameter`, `torch.empty`。

### Method `ModelOptNvFp4FusedMoE.apply_monolithic` (lines 1610-1632)
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
**EN:** Defines function `ModelOptNvFp4FusedMoE.apply_monolithic` with signature `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `router_logits`, `input_ids`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply_monolithic`.
**CN:** 定义函数 `ModelOptNvFp4FusedMoE.apply_monolithic`，其签名为 `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `router_logits`, `input_ids` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply_monolithic`。

### Method `ModelOptNvFp4FusedMoE.apply` (lines 1634-1657)
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
**EN:** Defines function `ModelOptNvFp4FusedMoE.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply`.
**CN:** 定义函数 `ModelOptNvFp4FusedMoE.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply`。

### Class `ModelOptMxFp8Config` overview (lines 1665-1739)
```python
class ModelOptMxFp8Config(ModelOptQuantConfigBase):
    """Config class for ModelOpt MXFP8."""

    def __init__(
        self,
        is_checkpoint_mxfp8_serialized: bool,
        kv_cache_quant_algo: str | None,
        exclude_modules: list[str],
    ) -> None:
        super().__init__(exclude_modules)
        self.is_checkpoint_mxfp8_serialized = is_checkpoint_mxfp8_serialized

        if not is_checkpoint_mxfp8_serialized:
            raise ValueError(
                "MXFP8 quantization requires a serialized checkpoint. "
                "Dynamic quantization is not supported."
            )

        logger.warning(
            "Detected ModelOpt MXFP8 checkpoint. Please note that "
            "the format is experimental and could change in future."
        )

        self.kv_cache_quant_algo = kv_cache_quant_algo
```
**EN:** Defines class `ModelOptMxFp8Config` with base classes `ModelOptQuantConfigBase` and decorators none. It acts as a configuration holder and backend selector and exposes 6 direct methods, with notable entries `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`. Its docstring says: Config class for ModelOpt MXFP8.
**CN:** 定义类 `ModelOptMxFp8Config`，其基类为 `ModelOptQuantConfigBase`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 6 个方法，较重要的包括 `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptMxFp8Config.__init__` (lines 1668-1688)
```python
    def __init__(
        self,
        is_checkpoint_mxfp8_serialized: bool,
        kv_cache_quant_algo: str | None,
        exclude_modules: list[str],
    ) -> None:
        super().__init__(exclude_modules)
        self.is_checkpoint_mxfp8_serialized = is_checkpoint_mxfp8_serialized

        if not is_checkpoint_mxfp8_serialized:
            raise ValueError(
                "MXFP8 quantization requires a serialized checkpoint. "
                "Dynamic quantization is not supported."
            )

        logger.warning(
            "Detected ModelOpt MXFP8 checkpoint. Please note that "
            "the format is experimental and could change in future."
        )

        self.kv_cache_quant_algo = kv_cache_quant_algo
```
**EN:** Defines function `ModelOptMxFp8Config.__init__` with signature `__init__(self, is_checkpoint_mxfp8_serialized: bool, kv_cache_quant_algo: str | None, exclude_modules: list[str]) -> None`. It mainly works with `is_checkpoint_mxfp8_serialized`, `kv_cache_quant_algo`, `exclude_modules`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `logger.warning`, `ValueError`, `super`.
**CN:** 定义函数 `ModelOptMxFp8Config.__init__`，其签名为 `__init__(self, is_checkpoint_mxfp8_serialized: bool, kv_cache_quant_algo: str | None, exclude_modules: list[str]) -> None`。它主要围绕 `is_checkpoint_mxfp8_serialized`, `kv_cache_quant_algo`, `exclude_modules` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `logger.warning`, `ValueError`, `super`。

### Method `ModelOptMxFp8Config.get_name` (lines 1690-1691)
```python
    def get_name(self) -> QuantizationMethods:
        return "modelopt_mxfp8"
```
**EN:** Defines function `ModelOptMxFp8Config.get_name` with signature `get_name(self) -> QuantizationMethods`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptMxFp8Config.get_name`，其签名为 `get_name(self) -> QuantizationMethods`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ModelOptMxFp8Config.get_supported_act_dtypes` (lines 1693-1694)
```python
    def get_supported_act_dtypes(self) -> list[torch.dtype]:
        return [torch.bfloat16]
```
**EN:** Defines function `ModelOptMxFp8Config.get_supported_act_dtypes` with signature `get_supported_act_dtypes(self) -> list[torch.dtype]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptMxFp8Config.get_supported_act_dtypes`，其签名为 `get_supported_act_dtypes(self) -> list[torch.dtype]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ModelOptMxFp8Config.get_min_capability` (lines 1697-1699)
```python
    def get_min_capability(cls) -> int:
        # Marlin kernel supports MXFP8 on SM80+
        return 80
```
**EN:** Defines function `ModelOptMxFp8Config.get_min_capability` with signature `get_min_capability(cls) -> int`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptMxFp8Config.get_min_capability`，其签名为 `get_min_capability(cls) -> int`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `ModelOptMxFp8LinearMethod` overview (lines 1742-1844)
```python
class ModelOptMxFp8LinearMethod(LinearMethodBase):
    """Linear method for ModelOpt MXFP8 quantization."""

    def __init__(self, quant_config: ModelOptMxFp8Config) -> None:
        self.quant_config = quant_config

        if not self.quant_config.is_checkpoint_mxfp8_serialized:
            raise ValueError(
                "MXFP8 currently only supports serialized checkpoints. "
                "Dynamic quantization is not supported."
            )

        self.kernel = init_mxfp8_linear_kernel()

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
        del input_size, output_size
```
**EN:** Defines class `ModelOptMxFp8LinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `apply`. Its docstring says: Linear method for ModelOpt MXFP8 quantization.
**CN:** 定义类 `ModelOptMxFp8LinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptMxFp8LinearMethod.__init__` (lines 1745-1754)
```python
    def __init__(self, quant_config: ModelOptMxFp8Config) -> None:
        self.quant_config = quant_config

        if not self.quant_config.is_checkpoint_mxfp8_serialized:
            raise ValueError(
                "MXFP8 currently only supports serialized checkpoints. "
                "Dynamic quantization is not supported."
            )

        self.kernel = init_mxfp8_linear_kernel()
```
**EN:** Defines function `ModelOptMxFp8LinearMethod.__init__` with signature `__init__(self, quant_config: ModelOptMxFp8Config) -> None`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `init_mxfp8_linear_kernel`, `ValueError`.
**CN:** 定义函数 `ModelOptMxFp8LinearMethod.__init__`，其签名为 `__init__(self, quant_config: ModelOptMxFp8Config) -> None`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `init_mxfp8_linear_kernel`, `ValueError`。

### Method `ModelOptMxFp8LinearMethod.create_weights` (lines 1756-1810)
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
        del input_size, output_size

        if not self.quant_config.is_checkpoint_mxfp8_serialized:
            raise ValueError(
                "MXFP8 quantization was selected, but checkpoint is not "
                "MXFP8 serialized. Dynamic quantization is not supported."
            )

        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")
        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition

        if input_size_per_partition % MXFP8_BLOCK_SIZE != 0:
            raise ValueError(
                f"MXFP8 requires input dimension to be divisible by "
                f"{MXFP8_BLOCK_SIZE}, got {input_size_per_partition}"
            )

        # Weight tensor: FP8 E4M3 format
        weight = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition,
                dtype=MXFP8_VALUE_DTYPE,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight", weight)

        # Weight scale tensor (E8M0 encoded as uint8), one scale per block of 32 along K
        weight_scale = ModelWeightParameter(
            data=torch.empty(
                output_size_per_partition,
                input_size_per_partition // MXFP8_BLOCK_SIZE,
                dtype=MXFP8_SCALE_DTYPE,
            ),
            input_dim=1,
            output_dim=0,
            weight_loader=weight_loader,
        )
        layer.register_parameter("weight_scale", weight_scale)
```
**EN:** Defines function `ModelOptMxFp8LinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `ValueError`, `torch.empty`.
**CN:** 定义函数 `ModelOptMxFp8LinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `sum`, `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `ValueError`, `torch.empty`。

### Method `ModelOptMxFp8LinearMethod.process_weights_after_loading` (lines 1812-1836)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Validate weight tensor
        if layer.weight.ndim != 2:
            raise ValueError(
                f"MXFP8 weight must be 2D tensor [N, K], got {layer.weight.ndim}D "
                f"with shape {tuple(layer.weight.shape)}"
            )

        if layer.weight.dtype != MXFP8_VALUE_DTYPE:
            raise ValueError(
                f"MXFP8 weight must be {MXFP8_VALUE_DTYPE} (FP8 E4M3), "
                f"got {layer.weight.dtype}. The checkpoint may not be properly "
                f"quantized with MXFP8."
            )

        # Validate weight scale tensor (should be 2D, not swizzled)
        assert layer.weight_scale.ndim == 2, (
            f"MXFP8 weight scale must be 2D, got {layer.weight_scale.ndim}D"
        )
        assert layer.weight_scale.dtype == MXFP8_SCALE_DTYPE, (
            f"MXFP8 weight scale must be {MXFP8_SCALE_DTYPE},"
            f" got {layer.weight_scale.dtype}"
        )

        self.kernel.process_weights_after_loading(layer)
```
**EN:** Defines function `ModelOptMxFp8LinearMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, validation/error handling. Key calls include `self.kernel.process_weights_after_loading`, `ValueError`, `tuple`.
**CN:** 定义函数 `ModelOptMxFp8LinearMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、校验或报错逻辑。关键调用包括 `self.kernel.process_weights_after_loading`, `ValueError`, `tuple`。

### Method `ModelOptMxFp8LinearMethod.apply` (lines 1838-1844)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.kernel.apply_weights(layer, x, bias)
```
**EN:** Defines function `ModelOptMxFp8LinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.kernel.apply_weights`.
**CN:** 定义函数 `ModelOptMxFp8LinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.kernel.apply_weights`。

### Class `ModelOptMxFp8FusedMoE` overview (lines 1847-2187)
```python
class ModelOptMxFp8FusedMoE(FusedMoEMethodBase):
    """FlashInfer TRTLLM MXFP8 block-scale MoE for ModelOpt checkpoints."""

    def __init__(
        self,
        quant_config: ModelOptMxFp8Config,
        moe_config: FusedMoEConfig,
    ) -> None:
        super().__init__(moe_config)
        self.quant_config = quant_config
        assert self.quant_config.is_checkpoint_mxfp8_serialized

        self.mxfp8_backend, _ = select_mxfp8_moe_backend(self.moe)

    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        assert layer.intermediate_size_per_partition == intermediate_size_per_partition
        assert layer.hidden_size == hidden_size
```
**EN:** Defines class `ModelOptMxFp8FusedMoE` with base classes `FusedMoEMethodBase` and decorators none. It acts as a quantization-oriented module building block and exposes 11 direct methods, with notable entries `__init__`, `create_weights`, `_check_weight_dtypes`, `_shuffle_weights_for_trtllm`, `process_weights_after_loading`, `maybe_make_prepare_finalize`. Its docstring says: FlashInfer TRTLLM MXFP8 block-scale MoE for ModelOpt checkpoints.
**CN:** 定义类 `ModelOptMxFp8FusedMoE`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当面向量化的模块构件，并直接暴露 11 个方法，较重要的包括 `__init__`, `create_weights`, `_check_weight_dtypes`, `_shuffle_weights_for_trtllm`, `process_weights_after_loading`, `maybe_make_prepare_finalize`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptMxFp8FusedMoE.__init__` (lines 1850-1859)
```python
    def __init__(
        self,
        quant_config: ModelOptMxFp8Config,
        moe_config: FusedMoEConfig,
    ) -> None:
        super().__init__(moe_config)
        self.quant_config = quant_config
        assert self.quant_config.is_checkpoint_mxfp8_serialized

        self.mxfp8_backend, _ = select_mxfp8_moe_backend(self.moe)
```
**EN:** Defines function `ModelOptMxFp8FusedMoE.__init__` with signature `__init__(self, quant_config: ModelOptMxFp8Config, moe_config: FusedMoEConfig) -> None`. It mainly works with `quant_config`, `moe_config`; initializes the object state and cached resources. The body uses validation/error handling. Key calls include `super.__init__`, `select_mxfp8_moe_backend`, `super`.
**CN:** 定义函数 `ModelOptMxFp8FusedMoE.__init__`，其签名为 `__init__(self, quant_config: ModelOptMxFp8Config, moe_config: FusedMoEConfig) -> None`。它主要围绕 `quant_config`, `moe_config` 展开；负责初始化对象状态和缓存资源。函数体包含校验或报错逻辑。关键调用包括 `super.__init__`, `select_mxfp8_moe_backend`, `super`。

### Method `ModelOptMxFp8FusedMoE.create_weights` (lines 1861-1952)
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
        assert layer.intermediate_size_per_partition == intermediate_size_per_partition
        assert layer.hidden_size == hidden_size
        layer.orig_dtype = params_dtype

        if hidden_size % MXFP8_BLOCK_SIZE != 0:
            raise ValueError(
                f"MXFP8 MoE requires hidden_size divisible by {MXFP8_BLOCK_SIZE}, "
                f"got {hidden_size}."
            )
        if intermediate_size_per_partition % MXFP8_BLOCK_SIZE != 0:
            raise ValueError(
                "MXFP8 MoE requires intermediate_size_per_partition divisible by "
                f"{MXFP8_BLOCK_SIZE}, got {intermediate_size_per_partition}."
            )

        layer.num_experts = num_experts
        weight_loader = extra_weight_attrs.get("weight_loader")
        w13_num_shards = 2 if self.moe.is_act_and_mul else 1

        # GEMM 1 weights: [E, (2I or I), H]
        w13_weight = ModelWeightParameter(
            data=torch.empty(
                num_experts,
                w13_num_shards * intermediate_size_per_partition,
                hidden_size,
                dtype=MXFP8_VALUE_DTYPE,
            ),
            input_dim=2,
            output_dim=1,
# ... truncated for analysis ...
            output_dim=1,
            weight_loader=weight_loader,
        )
        layer.register_parameter("w2_weight_scale", w2_weight_scale)

        # Ensure the generic MoE weight-loader treats these as block scales.
        set_weight_attrs(
            layer.w13_weight_scale,
            {"quant_method": FusedMoeWeightScaleSupported.BLOCK.value},
        )
        set_weight_attrs(
            layer.w2_weight_scale,
            {"quant_method": FusedMoeWeightScaleSupported.BLOCK.value},
        )
```
**EN:** Defines function `ModelOptMxFp8FusedMoE.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `ValueError`, `torch.empty`.
**CN:** 定义函数 `ModelOptMxFp8FusedMoE.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `extra_weight_attrs.get`, `ModelWeightParameter`, `layer.register_parameter`, `set_weight_attrs`, `ValueError`, `torch.empty`。

### Method `ModelOptMxFp8FusedMoE.apply_monolithic` (lines 2091-2173)
```python
    def apply_monolithic(
        self,
        layer: RoutedExperts,
        x: torch.Tensor,
        router_logits: torch.Tensor,
        input_ids: torch.Tensor | None = None,
    ) -> torch.Tensor:
        from flashinfer.fused_moe.core import (
            ActivationType,
            Fp8QuantizationType,
        )

        assert self.mxfp8_backend == Fp8MoeBackend.FLASHINFER_TRTLLM

        if layer.eplb_state is not None:
            raise NotImplementedError(
                "EPLB is not supported for FlashInfer TRTLLM MXFP8 MoE backend."
            )

        supported_activations = [MoEActivation.SILU]
        if layer.activation not in supported_activations:
            raise NotImplementedError(
                "FlashInfer TRTLLM MXFP8 MoE supports only "
                f"{supported_activations}, got {layer.activation}."
            )

        # Map vLLM MoEActivation to FlashInfer ActivationType.
        activation_map = {
            MoEActivation.SILU: ActivationType.Swiglu,
            MoEActivation.RELU2_NO_MUL: ActivationType.Relu2,
        }
        fi_activation_type: ActivationType = activation_map[layer.activation]

        # DeepSeekV3 routing requires float32 logits; others expect bfloat16.
        if layer.routing_method_type == RoutingMethodType.DeepSeekV3:
            assert router_logits.dtype == torch.float32, (
                "DeepSeekV3 routing requires float32 router_logits, "
                f"got {router_logits.dtype}."
# ... truncated for analysis ...
            routed_scaling_factor=layer.routed_scaling_factor,
            routing_method_type=layer.routing_method_type,
            use_shuffled_weight=True,
            weight_layout=0,
            fp8_quantization_type=Fp8QuantizationType.MxFp8,
        )

        if fi_activation_type != ActivationType.Swiglu:
            raise NotImplementedError(
                "FlashInfer TRTLLM MXFP8 MoE supports only Swiglu activation, "
                f"got {fi_activation_type}."
            )

        return flashinfer_trtllm_fp8_block_scale_moe(**kwargs)
```
**EN:** Defines function `ModelOptMxFp8FusedMoE.apply_monolithic` with signature `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `router_logits`, `input_ids`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `mxfp8_e4m3_quantize`, `dict`, `flashinfer_trtllm_fp8_block_scale_moe`, `NotImplementedError`, `router_logits.to`.
**CN:** 定义函数 `ModelOptMxFp8FusedMoE.apply_monolithic`，其签名为 `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `router_logits`, `input_ids` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `mxfp8_e4m3_quantize`, `dict`, `flashinfer_trtllm_fp8_block_scale_moe`, `NotImplementedError`, `router_logits.to`。

### Method `ModelOptMxFp8FusedMoE.apply` (lines 2175-2187)
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
        raise NotImplementedError(
            "Non-monolithic MXFP8 MoE path is not yet implemented."
        )
```
**EN:** Defines function `ModelOptMxFp8FusedMoE.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `NotImplementedError`.
**CN:** 定义函数 `ModelOptMxFp8FusedMoE.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `NotImplementedError`。

### Class `ModelOptMixedPrecisionConfig` overview (lines 2196-2378)
```python
class ModelOptMixedPrecisionConfig(ModelOptQuantConfigBase):
    """Config class for ModelOpt MIXED_PRECISION.

    Supports checkpoints where different layers use different quantization
    algorithms (e.g., FP8 for dense layers and NVFP4 for MoE experts).
    The per-layer algorithm is specified in the ``quantized_layers`` dict
    inside ``config.json``'s ``quantization_config`` (preferred) or the
    legacy ``hf_quant_config.json``.
    """

    def __init__(
        self,
        kv_cache_quant_method: str | None,
        exclude_modules: list[str],
        quantized_layers: dict[str, dict[str, Any]],
        fp8_config: ModelOptFp8Config,
        nvfp4_config: ModelOptNvFp4Config,
    ) -> None:
        super().__init__(exclude_modules)
        self.kv_cache_quant_method = kv_cache_quant_method
        self.quantized_layers = quantized_layers
        self.fp8_config = fp8_config
        self.nvfp4_config = nvfp4_config

    def get_name(self) -> QuantizationMethods:
```
**EN:** Defines class `ModelOptMixedPrecisionConfig` with base classes `ModelOptQuantConfigBase` and decorators none. It acts as a configuration holder and backend selector and exposes 9 direct methods, with notable entries `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`. Its docstring says: Config class for ModelOpt MIXED_PRECISION.
**CN:** 定义类 `ModelOptMixedPrecisionConfig`，其基类为 `ModelOptQuantConfigBase`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 9 个方法，较重要的包括 `__init__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `override_quantization_method`, `_from_config`。 文档字符串进一步说明了该类的定位。

### Method `ModelOptMixedPrecisionConfig.__init__` (lines 2206-2218)
```python
    def __init__(
        self,
        kv_cache_quant_method: str | None,
        exclude_modules: list[str],
        quantized_layers: dict[str, dict[str, Any]],
        fp8_config: ModelOptFp8Config,
        nvfp4_config: ModelOptNvFp4Config,
    ) -> None:
        super().__init__(exclude_modules)
        self.kv_cache_quant_method = kv_cache_quant_method
        self.quantized_layers = quantized_layers
        self.fp8_config = fp8_config
        self.nvfp4_config = nvfp4_config
```
**EN:** Defines function `ModelOptMixedPrecisionConfig.__init__` with signature `__init__(self, kv_cache_quant_method: str | None, exclude_modules: list[str], quantized_layers: dict[str, dict[str, Any]], fp8_config: ModelOptFp8Config, nvfp4_config: ModelOptNvFp4Config) -> None`. It mainly works with `kv_cache_quant_method`, `exclude_modules`, `quantized_layers`, `fp8_config`, `nvfp4_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `ModelOptMixedPrecisionConfig.__init__`，其签名为 `__init__(self, kv_cache_quant_method: str | None, exclude_modules: list[str], quantized_layers: dict[str, dict[str, Any]], fp8_config: ModelOptFp8Config, nvfp4_config: ModelOptNvFp4Config) -> None`。它主要围绕 `kv_cache_quant_method`, `exclude_modules`, `quantized_layers`, `fp8_config`, `nvfp4_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `ModelOptMixedPrecisionConfig.get_name` (lines 2220-2221)
```python
    def get_name(self) -> QuantizationMethods:
        return "modelopt_mixed"
```
**EN:** Defines function `ModelOptMixedPrecisionConfig.get_name` with signature `get_name(self) -> QuantizationMethods`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ModelOptMixedPrecisionConfig.get_name`，其签名为 `get_name(self) -> QuantizationMethods`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ModelOptMixedPrecisionConfig.get_quant_method` (lines 2334-2373)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> "QuantizeMethodBase | None":
        """Return quantize-method based on layer."""
        # KV-cache quantization
        if isinstance(layer, Attention):
            if self.kv_cache_quant_method:
                return ModelOptKVCacheMethod(self)
            return None

        # Excluded layers
        if self.is_layer_excluded(prefix):
            if isinstance(layer, LinearBase):
                return UnquantizedLinearMethod()
            return None

        quant_algo = self._resolve_quant_algo(prefix)

        if isinstance(layer, LinearBase):
            if quant_algo == "FP8":
                return ModelOptFp8LinearMethod(self.fp8_config)
            if quant_algo == "NVFP4":
                return ModelOptNvFp4LinearMethod(self.nvfp4_config)
            # Layer not in quantized_layers — leave unquantized
            return UnquantizedLinearMethod()

        if isinstance(layer, RoutedExperts):
            if quant_algo == "FP8":
                return ModelOptFp8MoEMethod(
                    quant_config=self.fp8_config,
                    moe_config=layer.moe_config,
                )
            if quant_algo == "NVFP4":
                return ModelOptNvFp4FusedMoE(
                    quant_config=self.nvfp4_config,
                    moe_config=layer.moe_config,
                )
            return None

        return None
```
**EN:** Defines function `ModelOptMixedPrecisionConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching. Key calls include `isinstance`, `self.is_layer_excluded`, `self._resolve_quant_algo`, `UnquantizedLinearMethod`, `ModelOptKVCacheMethod`, `ModelOptFp8LinearMethod`.
**CN:** 定义函数 `ModelOptMixedPrecisionConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> 'QuantizeMethodBase | None'`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `isinstance`, `self.is_layer_excluded`, `self._resolve_quant_algo`, `UnquantizedLinearMethod`, `ModelOptKVCacheMethod`, `ModelOptFp8LinearMethod`。

### Method `ModelOptMixedPrecisionConfig.apply_vllm_mapper` (lines 2375-2378)
```python
    def apply_vllm_mapper(self, hf_to_vllm_mapper: "WeightsMapper"):
        super().apply_vllm_mapper(hf_to_vllm_mapper)
        if self.quantized_layers:
            self.quantized_layers = hf_to_vllm_mapper.apply_dict(self.quantized_layers)
```
**EN:** Defines function `ModelOptMixedPrecisionConfig.apply_vllm_mapper` with signature `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`. It mainly works with `hf_to_vllm_mapper`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `super.apply_vllm_mapper`, `hf_to_vllm_mapper.apply_dict`, `super`.
**CN:** 定义函数 `ModelOptMixedPrecisionConfig.apply_vllm_mapper`，其签名为 `apply_vllm_mapper(self, hf_to_vllm_mapper: 'WeightsMapper')`。它主要围绕 `hf_to_vllm_mapper` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `super.apply_vllm_mapper`, `hf_to_vllm_mapper.apply_dict`, `super`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `ModelOptKVCacheMethod`, `ModelOptQuantConfigBase`, `ModelOptFp8Config`, `ModelOptFp8LinearMethod`, `ModelOptFp8PcPtLinearMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `ModelOptKVCacheMethod`, `ModelOptQuantConfigBase`, `ModelOptFp8Config`, `ModelOptFp8LinearMethod`, `ModelOptFp8PcPtLinearMethod`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `fnmatch`, `typing`, `torch`
- **Internal / 内部**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.config`, `vllm.logger`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.attention`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.oracle.fp8`, `vllm.model_executor.layers.fused_moe.oracle.mxfp8`, `vllm.model_executor.layers.fused_moe.oracle.nvfp4`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.quantization.kv_cache`, `vllm.model_executor.layers.quantization.utils.flashinfer_utils`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.mxfp8_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.parameter`
