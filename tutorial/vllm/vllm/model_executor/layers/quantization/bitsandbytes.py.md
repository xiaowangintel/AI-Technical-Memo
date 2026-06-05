# bitsandbytes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/bitsandbytes.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `BitsAndBytesConfig`, `BitsAndBytesLinearMethod`, `BitsAndBytesMoEMethod` for quantization backends, schemes, and utilities. / 实现 `BitsAndBytesConfig`, `BitsAndBytesLinearMethod`, `BitsAndBytesMoEMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-27)
```python
from typing import Any, Union

import torch
from packaging import version

from vllm.model_executor.layers.fused_moe import (
    FusedMoEConfig,
    FusedMoEMethodBase,
    FusedMoEQuantConfig,
    RoutedExperts,
    SharedExperts,
)
from vllm.model_executor.layers.linear import (
    LinearBase,
    LinearMethodBase,
    UnquantizedLinearMethod,
    set_weight_attrs,
)
from vllm.model_executor.layers.quantization import (
    QuantizationConfig,
    QuantizationMethods,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch`, `packaging` and internal modules such as `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.platforms`, `vllm.utils.torch_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`, `packaging`）以及内部模块（如 `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.platforms`, `vllm.utils.torch_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `_check_bitsandbytes_version` (lines 30-45)
```python
def _check_bitsandbytes_version():
    min_version = "0.49.2" if current_platform.is_rocm() else "0.48.1"
    try:
        import bitsandbytes

        if version.parse(bitsandbytes.__version__) < version.parse(min_version):
            raise ImportError(
                "bitsandbytes version is wrong. Please "
                f"install bitsandbytes>={min_version}."
            )
    except ImportError as err:
        raise ImportError(
            f"Please install bitsandbytes>={min_version} via "
            f"`pip install bitsandbytes>={min_version}` to use "
            "bitsandbytes quantizer."
        ) from err
```
**EN:** Defines function `_check_bitsandbytes_version` with signature `_check_bitsandbytes_version()`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `current_platform.is_rocm`, `version.parse`, `ImportError`.
**CN:** 定义函数 `_check_bitsandbytes_version`，其签名为 `_check_bitsandbytes_version()`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `current_platform.is_rocm`, `version.parse`, `ImportError`。

### Function `is_layer_skipped_bnb` (lines 171-185)
```python
def is_layer_skipped_bnb(prefix: str, llm_int8_skip_modules: list[str]):
    # Split the prefix into its dot-separated components
    components = prefix.split(".")

    # Check if any of the skip modules exactly matches any component
    substr_check = any(
        module_name in components for module_name in llm_int8_skip_modules
    )

    # Allow certain layers to not be quantized
    set_components = set(".".join(components[: i + 1]) for i in range(len(components)))
    set_llm_int8_skip_modules = set(llm_int8_skip_modules)
    prefix_check = len(set_llm_int8_skip_modules & set_components) != 0

    return substr_check or prefix_check
```
**EN:** Defines function `is_layer_skipped_bnb` with signature `is_layer_skipped_bnb(prefix: str, llm_int8_skip_modules: list[str])`. It mainly works with `prefix`, `llm_int8_skip_modules`; returns a derived property or capability check. The body uses comprehensions. Key calls include `prefix.split`, `any`, `set`, `len`, `'.'.join`, `range`.
**CN:** 定义函数 `is_layer_skipped_bnb`，其签名为 `is_layer_skipped_bnb(prefix: str, llm_int8_skip_modules: list[str])`。它主要围绕 `prefix`, `llm_int8_skip_modules` 展开；返回派生属性或能力判断结果。函数体包含推导式。关键调用包括 `prefix.split`, `any`, `set`, `len`, `'.'.join`, `range`。

### Function `calculate_quant_ratio` (lines 188-192)
```python
def calculate_quant_ratio(dtype):
    if dtype.is_floating_point:
        return torch.finfo(dtype).bits // torch.iinfo(torch.uint8).bits
    else:
        return torch.iinfo(dtype).bits // torch.iinfo(torch.uint8).bits
```
**EN:** Defines function `calculate_quant_ratio` with signature `calculate_quant_ratio(dtype)`. It mainly works with `dtype`; handles quantization-related transformation logic. The body uses branching, tensor/kernel operations. Key calls include `torch.finfo`, `torch.iinfo`.
**CN:** 定义函数 `calculate_quant_ratio`，其签名为 `calculate_quant_ratio(dtype)`。它主要围绕 `dtype` 展开；处理量化相关的变换逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.finfo`, `torch.iinfo`。

### Function `_apply_bnb_4bit` (lines 389-409)
```python
def _apply_bnb_4bit(
    x: torch.Tensor,
    weight: torch.Tensor,
    offsets: torch.Tensor,
    out: torch.Tensor,
) -> None:
    # only load the bitsandbytes module when needed
    from bitsandbytes import matmul_4bit

    quant_states = weight.bnb_quant_state
    current_index = 0
    for i in range(len(quant_states)):
        output_size = quant_states[i].shape[0]
        # It is more efficient to use out kwarg like
        # matmul_4bit(..., out = ...).  Infeasible now due to the bug
        # https://github.com/TimDettmers/bitsandbytes/issues/1235.
        # Need to change  after the bug is fixed.
        out[:, current_index : current_index + output_size] = matmul_4bit(
            x, weight[offsets[i] : offsets[i + 1]].t(), quant_states[i]
        )
        current_index += output_size
```
**EN:** Defines function `_apply_bnb_4bit` with signature `_apply_bnb_4bit(x: torch.Tensor, weight: torch.Tensor, offsets: torch.Tensor, out: torch.Tensor) -> None`. It mainly works with `x`, `weight`, `offsets`, `out`; implements one step in the quantized-weight execution flow. The body uses iteration. Key calls include `range`, `len`, `matmul_4bit`, `weight.t`.
**CN:** 定义函数 `_apply_bnb_4bit`，其签名为 `_apply_bnb_4bit(x: torch.Tensor, weight: torch.Tensor, offsets: torch.Tensor, out: torch.Tensor) -> None`。它主要围绕 `x`, `weight`, `offsets`, `out` 展开；实现量化权重执行流程中的一个步骤。函数体包含循环处理。关键调用包括 `range`, `len`, `matmul_4bit`, `weight.t`。

### Function `_apply_bnb_4bit_fake` (lines 412-418)
```python
def _apply_bnb_4bit_fake(
    x: torch.Tensor,
    weight: torch.Tensor,
    offsets: torch.Tensor,
    out: torch.Tensor,
) -> None:
    return
```
**EN:** Defines function `_apply_bnb_4bit_fake` with signature `_apply_bnb_4bit_fake(x: torch.Tensor, weight: torch.Tensor, offsets: torch.Tensor, out: torch.Tensor) -> None`. It mainly works with `x`, `weight`, `offsets`, `out`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `_apply_bnb_4bit_fake`，其签名为 `_apply_bnb_4bit_fake(x: torch.Tensor, weight: torch.Tensor, offsets: torch.Tensor, out: torch.Tensor) -> None`。它主要围绕 `x`, `weight`, `offsets`, `out` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `BitsAndBytesConfig` overview (lines 48-168)
```python
class BitsAndBytesConfig(QuantizationConfig):
    """Config class for BitsAndBytes Quantization.

    Reference: https://arxiv.org/abs/2305.14314
    """

    def __init__(
        self,
        load_in_8bit: bool = False,
        load_in_4bit: bool = True,
        bnb_4bit_compute_dtype: str = "float32",
        bnb_4bit_quant_storage: str = "uint8",
        bnb_4bit_quant_type: str = "fp4",
        bnb_4bit_use_double_quant: bool = False,
        llm_int8_enable_fp32_cpu_offload: bool = False,
        llm_int8_has_fp16_weight: bool = False,
        llm_int8_skip_modules: list[str] | None = None,
        llm_int8_threshold: float = 6.0,
    ) -> None:
        super().__init__()
        self.load_in_8bit = load_in_8bit
        self.load_in_4bit = load_in_4bit
        self.bnb_4bit_compute_dtype = bnb_4bit_compute_dtype
        self.bnb_4bit_quant_storage = bnb_4bit_quant_storage
        self.bnb_4bit_quant_type = bnb_4bit_quant_type
```
**EN:** Defines class `BitsAndBytesConfig` with base classes `QuantizationConfig` and decorators none. It acts as a configuration holder and backend selector and exposes 8 direct methods, with notable entries `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`. Its docstring says: Config class for BitsAndBytes Quantization.
**CN:** 定义类 `BitsAndBytesConfig`，其基类为 `QuantizationConfig`，装饰器为 无。它在整体实现中充当配置容器与后端选择器，并直接暴露 8 个方法，较重要的包括 `__init__`, `__repr__`, `get_name`, `get_supported_act_dtypes`, `get_min_capability`, `get_config_filenames`。 文档字符串进一步说明了该类的定位。

### Method `BitsAndBytesConfig.__init__` (lines 54-82)
```python
    def __init__(
        self,
        load_in_8bit: bool = False,
        load_in_4bit: bool = True,
        bnb_4bit_compute_dtype: str = "float32",
        bnb_4bit_quant_storage: str = "uint8",
        bnb_4bit_quant_type: str = "fp4",
        bnb_4bit_use_double_quant: bool = False,
        llm_int8_enable_fp32_cpu_offload: bool = False,
        llm_int8_has_fp16_weight: bool = False,
        llm_int8_skip_modules: list[str] | None = None,
        llm_int8_threshold: float = 6.0,
    ) -> None:
        super().__init__()
        self.load_in_8bit = load_in_8bit
        self.load_in_4bit = load_in_4bit
        self.bnb_4bit_compute_dtype = bnb_4bit_compute_dtype
        self.bnb_4bit_quant_storage = bnb_4bit_quant_storage
        self.bnb_4bit_quant_type = bnb_4bit_quant_type
        self.bnb_4bit_use_double_quant = bnb_4bit_use_double_quant
        self.llm_int8_enable_fp32_cpu_offload = llm_int8_enable_fp32_cpu_offload
        self.llm_int8_has_fp16_weight = llm_int8_has_fp16_weight
        self.llm_int8_skip_modules = llm_int8_skip_modules or []
        self.llm_int8_threshold = llm_int8_threshold

        if self.bnb_4bit_quant_storage not in ["uint8"]:
            raise ValueError(
                f"Unsupported bnb_4bit_quant_storage: {self.bnb_4bit_quant_storage}"
            )
```
**EN:** Defines function `BitsAndBytesConfig.__init__` with signature `__init__(self, load_in_8bit: bool=False, load_in_4bit: bool=True, bnb_4bit_compute_dtype: str='float32', bnb_4bit_quant_storage: str='uint8', bnb_4bit_quant_type: str='fp4', bnb_4bit_use_double_quant: bool=False, llm_int8_enable_fp32_cpu_offload: bool=False, llm_int8_has_fp16_weight: bool=False, llm_int8_skip_modules: list[str] | None=None, llm_int8_threshold: float=6.0) -> None`. It mainly works with `load_in_8bit`, `load_in_4bit`, `bnb_4bit_compute_dtype`, `bnb_4bit_quant_storage`, `bnb_4bit_quant_type`, `bnb_4bit_use_double_quant`, `llm_int8_enable_fp32_cpu_offload`, `llm_int8_has_fp16_weight`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `ValueError`, `super`.
**CN:** 定义函数 `BitsAndBytesConfig.__init__`，其签名为 `__init__(self, load_in_8bit: bool=False, load_in_4bit: bool=True, bnb_4bit_compute_dtype: str='float32', bnb_4bit_quant_storage: str='uint8', bnb_4bit_quant_type: str='fp4', bnb_4bit_use_double_quant: bool=False, llm_int8_enable_fp32_cpu_offload: bool=False, llm_int8_has_fp16_weight: bool=False, llm_int8_skip_modules: list[str] | None=None, llm_int8_threshold: float=6.0) -> None`。它主要围绕 `load_in_8bit`, `load_in_4bit`, `bnb_4bit_compute_dtype`, `bnb_4bit_quant_storage`, `bnb_4bit_quant_type`, `bnb_4bit_use_double_quant`, `llm_int8_enable_fp32_cpu_offload`, `llm_int8_has_fp16_weight` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `ValueError`, `super`。

### Method `BitsAndBytesConfig.get_name` (lines 95-96)
```python
    def get_name(self) -> QuantizationMethods:
        return "bitsandbytes"
```
**EN:** Defines function `BitsAndBytesConfig.get_name` with signature `get_name(self) -> QuantizationMethods`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `BitsAndBytesConfig.get_name`，其签名为 `get_name(self) -> QuantizationMethods`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `BitsAndBytesConfig.from_config` (lines 111-157)
```python
    def from_config(cls, config: dict[str, Any]) -> "BitsAndBytesConfig":
        def get_safe_value(config, keys, default_value=None):
            try:
                value = cls.get_from_keys(config, keys)
                return value if value is not None else default_value
            except ValueError:
                return default_value

        load_in_8bit = get_safe_value(config, ["load_in_8bit"], default_value=False)
        load_in_4bit = get_safe_value(config, ["load_in_4bit"], default_value=True)
        bnb_4bit_compute_dtype = get_safe_value(
            config, ["bnb_4bit_compute_dtype"], default_value="float32"
        )
        bnb_4bit_quant_storage = get_safe_value(
            config, ["bnb_4bit_quant_storage"], default_value="uint8"
        )
        bnb_4bit_quant_type = get_safe_value(
            config, ["bnb_4bit_quant_type"], default_value="fp4"
        )
        bnb_4bit_use_double_quant = get_safe_value(
            config, ["bnb_4bit_use_double_quant"], default_value=False
        )
        llm_int8_enable_fp32_cpu_offload = get_safe_value(
            config, ["llm_int8_enable_fp32_cpu_offload"], default_value=False
        )
        llm_int8_has_fp16_weight = get_safe_value(
            config, ["llm_int8_has_fp16_weight"], default_value=False
        )
        llm_int8_skip_modules = get_safe_value(
            config, ["llm_int8_skip_modules"], default_value=[]
        )
        llm_int8_threshold = get_safe_value(
            config, ["llm_int8_threshold"], default_value=6.0
        )

        return cls(
            load_in_8bit=load_in_8bit,
            load_in_4bit=load_in_4bit,
            bnb_4bit_compute_dtype=bnb_4bit_compute_dtype,
            bnb_4bit_quant_storage=bnb_4bit_quant_storage,
            bnb_4bit_quant_type=bnb_4bit_quant_type,
            bnb_4bit_use_double_quant=bnb_4bit_use_double_quant,
            llm_int8_enable_fp32_cpu_offload=llm_int8_enable_fp32_cpu_offload,
            llm_int8_has_fp16_weight=llm_int8_has_fp16_weight,
            llm_int8_skip_modules=llm_int8_skip_modules,
            llm_int8_threshold=llm_int8_threshold,
        )
```
**EN:** Defines function `BitsAndBytesConfig.from_config` with signature `from_config(cls, config: dict[str, Any]) -> 'BitsAndBytesConfig'`. It mainly works with `config`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `get_safe_value`, `cls`, `cls.get_from_keys`.
**CN:** 定义函数 `BitsAndBytesConfig.from_config`，其签名为 `from_config(cls, config: dict[str, Any]) -> 'BitsAndBytesConfig'`。它主要围绕 `config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `get_safe_value`, `cls`, `cls.get_from_keys`。

### Method `BitsAndBytesConfig.get_quant_method` (lines 159-168)
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Union["LinearMethodBase", "BitsAndBytesMoEMethod"] | None:
        if isinstance(layer, LinearBase):
            if is_layer_skipped_bnb(prefix, self.llm_int8_skip_modules):
                return UnquantizedLinearMethod()
            return BitsAndBytesLinearMethod(self)
        elif isinstance(layer, RoutedExperts):
            return BitsAndBytesMoEMethod(self, layer.moe_config)
        return None
```
**EN:** Defines function `BitsAndBytesConfig.get_quant_method` with signature `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> Union['LinearMethodBase', 'BitsAndBytesMoEMethod'] | None`. It mainly works with `layer`, `prefix`; handles quantization-related transformation logic. The body uses branching. Key calls include `isinstance`, `is_layer_skipped_bnb`, `BitsAndBytesLinearMethod`, `UnquantizedLinearMethod`, `BitsAndBytesMoEMethod`.
**CN:** 定义函数 `BitsAndBytesConfig.get_quant_method`，其签名为 `get_quant_method(self, layer: torch.nn.Module, prefix: str) -> Union['LinearMethodBase', 'BitsAndBytesMoEMethod'] | None`。它主要围绕 `layer`, `prefix` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `isinstance`, `is_layer_skipped_bnb`, `BitsAndBytesLinearMethod`, `UnquantizedLinearMethod`, `BitsAndBytesMoEMethod`。

### Class `BitsAndBytesLinearMethod` overview (lines 195-386)
```python
class BitsAndBytesLinearMethod(LinearMethodBase):
    """Linear method for BitsAndBytes.

    Args:
       quant_config: The BitsAndBytes quantization config.
    """

    def __init__(self, quant_config: BitsAndBytesConfig):
        _check_bitsandbytes_version()
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
        from bitsandbytes.nn import Int8Params

        def create_qweight_for_8bit():
            qweight = Int8Params(
```
**EN:** Defines class `BitsAndBytesLinearMethod` with base classes `LinearMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `create_weights`, `apply`, `_apply_8bit_weight`, `_apply_4bit_weight`. Its docstring says: Linear method for BitsAndBytes.
**CN:** 定义类 `BitsAndBytesLinearMethod`，其基类为 `LinearMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `create_weights`, `apply`, `_apply_8bit_weight`, `_apply_4bit_weight`。 文档字符串进一步说明了该类的定位。

### Method `BitsAndBytesLinearMethod.__init__` (lines 202-204)
```python
    def __init__(self, quant_config: BitsAndBytesConfig):
        _check_bitsandbytes_version()
        self.quant_config = quant_config
```
**EN:** Defines function `BitsAndBytesLinearMethod.__init__` with signature `__init__(self, quant_config: BitsAndBytesConfig)`. It mainly works with `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `_check_bitsandbytes_version`.
**CN:** 定义函数 `BitsAndBytesLinearMethod.__init__`，其签名为 `__init__(self, quant_config: BitsAndBytesConfig)`。它主要围绕 `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `_check_bitsandbytes_version`。

### Method `BitsAndBytesLinearMethod.create_weights` (lines 206-271)
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
        from bitsandbytes.nn import Int8Params

        def create_qweight_for_8bit():
            qweight = Int8Params(
                data=torch.empty(
                    sum(output_partition_sizes),
                    input_size_per_partition,
                    dtype=torch.int8,
                ),
                has_fp16_weights=self.quant_config.llm_int8_has_fp16_weight,
                requires_grad=False,
            )
            set_weight_attrs(
                qweight,
                {
                    "input_dim": 0,
                    "output_dim": 0,
                    "pack_factor": 1,
                    "use_bitsandbytes_8bit": True,
                    "generation": 0,
                },
            )
            return qweight

        def create_qweight_for_4bit():
            quant_ratio = calculate_quant_ratio(params_dtype)

            total_size = input_size_per_partition * sum(output_partition_sizes)
# ... truncated for analysis ...
                    "pack_factor": quant_ratio,
                    "use_bitsandbytes_4bit": True,
                },
            )
            return qweight

        if self.quant_config.load_in_8bit:
            qweight = create_qweight_for_8bit()
        else:
            qweight = create_qweight_for_4bit()
        # Enable parameters to have the same name as in the BNB
        # checkpoint format.
        layer.register_parameter("weight", qweight)
        set_weight_attrs(qweight, extra_weight_attrs)
```
**EN:** Defines function `BitsAndBytesLinearMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `layer.register_parameter`, `set_weight_attrs`, `Int8Params`, `calculate_quant_ratio`, `torch.nn.Parameter`, `create_qweight_for_8bit`.
**CN:** 定义函数 `BitsAndBytesLinearMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `layer.register_parameter`, `set_weight_attrs`, `Int8Params`, `calculate_quant_ratio`, `torch.nn.Parameter`, `create_qweight_for_8bit`。

### Method `BitsAndBytesLinearMethod.apply` (lines 273-282)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if self.quant_config.load_in_8bit:
            return self._apply_8bit_weight(layer, x, bias)
        else:
            return self._apply_4bit_weight(layer, x, bias)
```
**EN:** Defines function `BitsAndBytesLinearMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `self._apply_8bit_weight`, `self._apply_4bit_weight`.
**CN:** 定义函数 `BitsAndBytesLinearMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `self._apply_8bit_weight`, `self._apply_4bit_weight`。

### Method `BitsAndBytesLinearMethod._apply_8bit_weight` (lines 284-352)
```python
    def _apply_8bit_weight(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # only load the bitsandbytes module when needed
        from bitsandbytes import MatmulLtState, matmul

        original_type = x.dtype
        original_shape = x.shape
        reshape_after_matmul = False
        if x.ndim > 2:
            x = x.reshape(-1, x.size(-1))
            reshape_after_matmul = True
        bf_x = x.to(torch.bfloat16)

        qweight = layer.weight
        offsets = qweight.bnb_shard_offsets
        quant_states = qweight.bnb_quant_state
        matmul_states = qweight.matmul_state
        generation = qweight.generation

        out_dim_0 = x.shape[0]
        out_dim_1 = sum(
            [quant_state[1].shape[0] for quant_state in quant_states.items()]
        )
        out = torch.empty(out_dim_0, out_dim_1, dtype=torch.float16, device=x.device)

        current_index = 0
        for i in range(len(quant_states)):
            output_size = quant_states[i].shape[0]

            # in profile_run or the first generation of inference,
            # create new matmul_states
            if generation == 0 or generation == 1:
                matmul_states[i] = MatmulLtState()
                matmul_states[i].CB = qweight[offsets[i] : offsets[i + 1]]
# ... truncated for analysis ...

            current_index += output_size

        out = out.to(original_type)

        if reshape_after_matmul:
            out = out.view(*original_shape[:-1], out.size(-1))

        if bias is not None:
            out += bias

        qweight.generation += 1

        return out
```
**EN:** Defines function `BitsAndBytesLinearMethod._apply_8bit_weight` with signature `_apply_8bit_weight(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, comprehensions, tensor/kernel operations. Key calls include `x.to`, `sum`, `torch.empty`, `range`, `out.to`, `x.reshape`.
**CN:** 定义函数 `BitsAndBytesLinearMethod._apply_8bit_weight`，其签名为 `_apply_8bit_weight(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、推导式、张量或内核操作。关键调用包括 `x.to`, `sum`, `torch.empty`, `range`, `out.to`, `x.reshape`。

### Class `BitsAndBytesMoEMethod` overview (lines 435-608)
```python
class BitsAndBytesMoEMethod(FusedMoEMethodBase):
    """MoE method for BitsAndBytes.

    Args:
       quant_config: The BitsAndBytes quantization config.
    """

    def __init__(
        self,
        quant_config: BitsAndBytesConfig,
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        _check_bitsandbytes_version()
        self.quant_config = quant_config

    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
```
**EN:** Defines class `BitsAndBytesMoEMethod` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 8 direct methods, with notable entries `__init__`, `create_weights`, `get_fused_moe_quant_config`, `apply`, `_create_weights_4bit`, `_create_weights_8bit`. Its docstring says: MoE method for BitsAndBytes.
**CN:** 定义类 `BitsAndBytesMoEMethod`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 8 个方法，较重要的包括 `__init__`, `create_weights`, `get_fused_moe_quant_config`, `apply`, `_create_weights_4bit`, `_create_weights_8bit`。 文档字符串进一步说明了该类的定位。

### Method `BitsAndBytesMoEMethod.__init__` (lines 442-449)
```python
    def __init__(
        self,
        quant_config: BitsAndBytesConfig,
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        _check_bitsandbytes_version()
        self.quant_config = quant_config
```
**EN:** Defines function `BitsAndBytesMoEMethod.__init__` with signature `__init__(self, quant_config: BitsAndBytesConfig, moe: FusedMoEConfig)`. It mainly works with `quant_config`, `moe`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `_check_bitsandbytes_version`, `super`.
**CN:** 定义函数 `BitsAndBytesMoEMethod.__init__`，其签名为 `__init__(self, quant_config: BitsAndBytesConfig, moe: FusedMoEConfig)`。它主要围绕 `quant_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `_check_bitsandbytes_version`, `super`。

### Method `BitsAndBytesMoEMethod.create_weights` (lines 451-471)
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
        if self.quant_config.load_in_8bit:
            call_fun = self._create_weights_8bit
        else:
            call_fun = self._create_weights_4bit
        call_fun(
            layer,
            num_experts,
            hidden_size,
            intermediate_size_per_partition,
            params_dtype,
            **extra_weight_attrs,
        )
```
**EN:** Defines function `BitsAndBytesMoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching. Key calls include `call_fun`.
**CN:** 定义函数 `BitsAndBytesMoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断。关键调用包括 `call_fun`。

### Method `BitsAndBytesMoEMethod.get_fused_moe_quant_config` (lines 473-476)
```python
    def get_fused_moe_quant_config(
        self, layer: RoutedExperts
    ) -> FusedMoEQuantConfig | None:
        return None
```
**EN:** Defines function `BitsAndBytesMoEMethod.get_fused_moe_quant_config` with signature `get_fused_moe_quant_config(self, layer: RoutedExperts) -> FusedMoEQuantConfig | None`. It mainly works with `layer`; handles quantization-related transformation logic. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `BitsAndBytesMoEMethod.get_fused_moe_quant_config`，其签名为 `get_fused_moe_quant_config(self, layer: RoutedExperts) -> FusedMoEQuantConfig | None`。它主要围绕 `layer` 展开；处理量化相关的变换逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `BitsAndBytesMoEMethod.apply` (lines 478-506)
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
        from vllm.model_executor.layers.fused_moe import fused_experts

        # TODO(bnell): Do these need to be called on the hot path?
        if self.quant_config.load_in_8bit:
            w13, w2 = self._apply_8bit_dequant(layer)
        else:
            w13, w2 = self._apply_4bit_dequnt(layer)
        return fused_experts(
            hidden_states=x,
            w1=w13,
            w2=w2,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            inplace=not self.moe.disable_inplace,
            activation=layer.activation,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            quant_config=self.moe_quant_config,
        )
```
**EN:** Defines function `BitsAndBytesMoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `fused_experts`, `self._apply_8bit_dequant`, `self._apply_4bit_dequnt`.
**CN:** 定义函数 `BitsAndBytesMoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `fused_experts`, `self._apply_8bit_dequant`, `self._apply_4bit_dequnt`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `BitsAndBytesConfig`, `BitsAndBytesLinearMethod`, `BitsAndBytesMoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `BitsAndBytesConfig`, `BitsAndBytesLinearMethod`, `BitsAndBytesMoEMethod`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `_check_bitsandbytes_version`, `is_layer_skipped_bnb`, `calculate_quant_ratio`, `_apply_bnb_4bit`, `_apply_bnb_4bit_fake` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_check_bitsandbytes_version`, `is_layer_skipped_bnb`, `calculate_quant_ratio`, `_apply_bnb_4bit`, `_apply_bnb_4bit_fake` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`, `packaging`
- **Internal / 内部**: `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.platforms`, `vllm.utils.torch_utils`
