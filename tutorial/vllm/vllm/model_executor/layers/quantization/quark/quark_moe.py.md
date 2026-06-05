# quark_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/quark/quark_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `QuarkMoEMethod`, `QuarkW8A8Fp8MoEMethod`, `QuarkW8A8Int8MoEMethod` for quantization backends, schemes, and utilities. / 实现 `QuarkMoEMethod`, `QuarkW8A8Fp8MoEMethod`, `QuarkW8A8Int8MoEMethod`，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-68)
```python
from typing import Any

import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import _custom_ops as ops
from vllm import envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import get_current_vllm_config
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe import (
    FusedMoEConfig,
    FusedMoEMethodBase,
    FusedMoeWeightScaleSupported,
    MoEActivation,
    RoutedExperts,
    SharedExperts,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    fp8_w8a8_moe_quant_config,
    int8_w8a8_moe_quant_config,
    mxfp4_w4a8_moe_quant_config,
    mxfp4_w4a16_moe_quant_config,
    ocp_mx_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.experts.marlin_moe import fused_marlin_moe
from vllm.model_executor.layers.fused_moe.oracle.mxfp4 import (
    TRITON_BACKENDS,
    Mxfp4MoeBackend,
    backend_to_kernel_cls,
    convert_gpt_oss_weight_to_mxfp4_moe_kernel_format,
    make_mxfp4_moe_kernel,
    make_mxfp4_moe_quant_config,
    mxfp4_round_up_hidden_size_and_intermediate_size,
    select_mxfp4_moe_backend,
)
from vllm.model_executor.layers.fused_moe.oracle.nvfp4 import (
    convert_to_nvfp4_moe_kernel_format,
    make_nvfp4_moe_kernel,
    make_nvfp4_moe_quant_config,
    select_nvfp4_moe_backend,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_fp8 import (
    prepare_fp8_moe_layer_for_marlin,
)
from vllm.model_executor.layers.quantization.utils.ocp_mx_utils import (
    OCP_MX_BLOCK_SIZE,
    OCP_MX_Scheme,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
    kNvfp4Static,
)
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    all_close_1d,
    normalize_e4m3fn_to_e4m3fnuz,
    per_tensor_dequantize,
)
from vllm.model_executor.utils import replace_parameter, set_weight_attrs
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
```
**EN:** This opening block pulls in external dependencies such as `typing`, `torch` and internal modules such as `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm._aiter_ops`, `vllm.config`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `torch`）以及内部模块（如 `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm._aiter_ops`, `vllm.config`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 70-77)
```python
logger = init_logger(__name__)

__all__ = [
    "QuarkMoEMethod",
    "QuarkW8A8Fp8MoEMethod",
    "QuarkOCP_MX_MoEMethod",
    "QuarkNvfp4MoEMethod",
]
```
**EN:** This block defines module-level metadata or constants such as `logger`, `__all__`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `__all__`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Class `QuarkMoEMethod` overview (lines 80-122)
```python
class QuarkMoEMethod(FusedMoEMethodBase):
    def __init__(self, moe: FusedMoEConfig):
        super().__init__(moe)
        self.has_bias = self.moe.has_bias

    @staticmethod
    def get_moe_method(
        quant_config: "QuarkConfig",  # type: ignore # noqa E501 # noqa F821
        module: RoutedExperts,
        layer_name: str,
    ) -> "QuarkMoEMethod":
        layer_quant_config = quant_config._find_matched_config(layer_name, module)

        if layer_quant_config.get("output_tensors") or layer_quant_config.get("bias"):
            raise NotImplementedError(
                "Currently, Quark models with "
                "output_tensors and bias "
                "quantized are not supported"
            )

        weight_config = layer_quant_config.get("weight")
        input_config = layer_quant_config.get("input_tensors")

        if quant_config._is_fp8_w4a8(weight_config, input_config):
            return QuarkW4A8Fp8MoEMethod(weight_config, input_config, module.moe_config)
```
**EN:** Defines class `QuarkMoEMethod` with base classes `FusedMoEMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 2 direct methods, with notable entries `__init__`, `get_moe_method`.
**CN:** 定义类 `QuarkMoEMethod`，其基类为 `FusedMoEMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 2 个方法，较重要的包括 `__init__`, `get_moe_method`。

### Method `QuarkMoEMethod.__init__` (lines 81-83)
```python
    def __init__(self, moe: FusedMoEConfig):
        super().__init__(moe)
        self.has_bias = self.moe.has_bias
```
**EN:** Defines function `QuarkMoEMethod.__init__` with signature `__init__(self, moe: FusedMoEConfig)`. It mainly works with `moe`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `QuarkMoEMethod.__init__`，其签名为 `__init__(self, moe: FusedMoEConfig)`。它主要围绕 `moe` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `QuarkMoEMethod.get_moe_method` (lines 86-122)
```python
    def get_moe_method(
        quant_config: "QuarkConfig",  # type: ignore # noqa E501 # noqa F821
        module: RoutedExperts,
        layer_name: str,
    ) -> "QuarkMoEMethod":
        layer_quant_config = quant_config._find_matched_config(layer_name, module)

        if layer_quant_config.get("output_tensors") or layer_quant_config.get("bias"):
            raise NotImplementedError(
                "Currently, Quark models with "
                "output_tensors and bias "
                "quantized are not supported"
            )

        weight_config = layer_quant_config.get("weight")
        input_config = layer_quant_config.get("input_tensors")

        if quant_config._is_fp8_w4a8(weight_config, input_config):
            return QuarkW4A8Fp8MoEMethod(weight_config, input_config, module.moe_config)
        elif quant_config._is_nvfp4(weight_config, input_config):
            return QuarkNvfp4MoEMethod(
                weight_config, input_config, module.moe_config, quant_config
            )
        elif quant_config._is_fp8_w8a8(weight_config, input_config):
            return QuarkW8A8Fp8MoEMethod(weight_config, input_config, module.moe_config)
        elif quant_config._is_w_ocp_mx_a_x(weight_config, input_config):
            # All OCP MX schemes (W4A16, W4A8, etc.) handled by QuarkOCP_MX_MoEMethod
            # Backend selection happens inside via oracle
            return QuarkOCP_MX_MoEMethod(weight_config, input_config, module.moe_config)
        elif quant_config._is_static_tensor_w8a8(
            weight_config, input_config
        ) or quant_config._is_dynamic_per_token_w8a8(weight_config, input_config):
            return QuarkW8A8Int8MoEMethod(
                weight_config, input_config, module.moe_config
            )
        else:
            raise RuntimeError("Unsupported FusedMoe scheme")
```
**EN:** Defines function `QuarkMoEMethod.get_moe_method` with signature `get_moe_method(quant_config: 'QuarkConfig', module: RoutedExperts, layer_name: str) -> 'QuarkMoEMethod'`. It mainly works with `quant_config`, `module`, `layer_name`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `quant_config._find_matched_config`, `layer_quant_config.get`, `quant_config._is_fp8_w4a8`, `NotImplementedError`, `QuarkW4A8Fp8MoEMethod`, `quant_config._is_nvfp4`.
**CN:** 定义函数 `QuarkMoEMethod.get_moe_method`，其签名为 `get_moe_method(quant_config: 'QuarkConfig', module: RoutedExperts, layer_name: str) -> 'QuarkMoEMethod'`。它主要围绕 `quant_config`, `module`, `layer_name` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `quant_config._find_matched_config`, `layer_quant_config.get`, `quant_config._is_fp8_w4a8`, `NotImplementedError`, `QuarkW4A8Fp8MoEMethod`, `quant_config._is_nvfp4`。

### Class `QuarkW8A8Fp8MoEMethod` overview (lines 125-516)
```python
class QuarkW8A8Fp8MoEMethod(QuarkMoEMethod):
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config

        self.weight_qscheme = self.weight_quant.get("qscheme")
        self.input_qscheme = self.input_quant.get("qscheme")
        self.weight_dtype = self.weight_quant.get("dtype", "").replace(
            "fp8_e4m3", "fp8"
        )
        self.input_dtype = self.input_quant.get("dtype", "").replace("fp8_e4m3", "fp8")
        per_tensor = (
            self.weight_qscheme == "per_tensor" and self.input_qscheme == "per_tensor"
        )
        per_channel = (
            self.weight_qscheme == "per_channel" and self.input_qscheme == "per_channel"
        )
        self.act_quant_group_shape = (
            GroupShape.PER_TOKEN if per_channel else GroupShape.PER_TENSOR
```
**EN:** Defines class `QuarkW8A8Fp8MoEMethod` with base classes `QuarkMoEMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`.
**CN:** 定义类 `QuarkW8A8Fp8MoEMethod`，其基类为 `QuarkMoEMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`。

### Method `QuarkW8A8Fp8MoEMethod.__init__` (lines 126-179)
```python
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config

        self.weight_qscheme = self.weight_quant.get("qscheme")
        self.input_qscheme = self.input_quant.get("qscheme")
        self.weight_dtype = self.weight_quant.get("dtype", "").replace(
            "fp8_e4m3", "fp8"
        )
        self.input_dtype = self.input_quant.get("dtype", "").replace("fp8_e4m3", "fp8")
        per_tensor = (
            self.weight_qscheme == "per_tensor" and self.input_qscheme == "per_tensor"
        )
        per_channel = (
            self.weight_qscheme == "per_channel" and self.input_qscheme == "per_channel"
        )
        self.act_quant_group_shape = (
            GroupShape.PER_TOKEN if per_channel else GroupShape.PER_TENSOR
        )
        if not (per_tensor or per_channel):
            raise ValueError(
                "For FP8 Fused MoE layers, only per-tensor and per-channel "
                "scales for weights and activations are supported. Found "
                f"{self.weight_qscheme}, {self.input_qscheme}"
            )  # noqa E501

        self.static_input_scales = not self.input_quant.get("is_dynamic")
        if self.static_input_scales and per_channel:
            raise ValueError(
                "For FP8 Fused MoE layer, we require either per tensor or "
                "channelwise, dynamic per token quantization."
            )

        # For GPUs that lack FP8 hardware support, we can leverage the Marlin
        # kernel for fast weight-only FP8 quantization
        self.use_marlin = (
            not current_platform.has_device_capability(89)
            or envs.VLLM_TEST_FORCE_FP8_MARLIN
        )
        # Disable marlin for rocm
        if current_platform.is_rocm():
            self.use_marlin = False

        self.rocm_aiter_moe_enabled = rocm_aiter_ops.is_fused_moe_enabled()

        self.model_type = getattr(
            get_current_vllm_config().model_config.hf_config, "model_type", None
        )
```
**EN:** Defines function `QuarkW8A8Fp8MoEMethod.__init__` with signature `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig)`. It mainly works with `weight_config`, `input_config`, `moe`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `self.weight_quant.get`, `self.input_quant.get`, `self.weight_quant.get.replace`, `self.input_quant.get.replace`, `current_platform.is_rocm`.
**CN:** 定义函数 `QuarkW8A8Fp8MoEMethod.__init__`，其签名为 `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig)`。它主要围绕 `weight_config`, `input_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `self.weight_quant.get`, `self.input_quant.get`, `self.weight_quant.get.replace`, `self.input_quant.get.replace`, `current_platform.is_rocm`。

### Method `QuarkW8A8Fp8MoEMethod.create_weights` (lines 181-304)
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
        params_dtype = torch.float8_e4m3fn

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                hidden_size,
                intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)
# ... truncated for analysis ...
                ),
                requires_grad=False,
            )
            layer.register_parameter("w13_bias", w13_bias)
            set_weight_attrs(w13_bias, extra_weight_attrs)

            w2_bias = torch.nn.Parameter(
                torch.zeros(num_experts, hidden_size, dtype=torch.float32),
                requires_grad=False,
            )
            layer.register_parameter("w2_bias", w2_bias)
            set_weight_attrs(w2_bias, extra_weight_attrs)
        else:
            layer.w13_bias, layer.w2_bias = None, None
```
**EN:** Defines function `QuarkW8A8Fp8MoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.zeros`, `extra_weight_attrs.update`, `torch.ones`.
**CN:** 定义函数 `QuarkW8A8Fp8MoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.zeros`, `extra_weight_attrs.update`, `torch.ones`。

### Method `QuarkW8A8Fp8MoEMethod.process_weights_after_loading` (lines 306-438)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        # Fp8 moe kernels require a single activation scale.
        # We take the max of all the scales in case they differ.
        if self.static_input_scales:
            if layer.w13_input_scale is None or layer.w2_input_scale is None:
                raise ValueError(
                    "QuantConfig has static quantization, but found "
                    "activation scales are None."
                )
            if not all_close_1d(layer.w13_input_scale) or not all_close_1d(
                layer.w2_input_scale
            ):
                logger.warning_once(
                    "Found input_scales that are not equal for "
                    "fp8 MoE layer. Using the maximum across experts "
                    "for each layer. "
                )
            layer.w13_input_scale = torch.nn.Parameter(
                layer.w13_input_scale.max(), requires_grad=False
            )
            layer.w2_input_scale = torch.nn.Parameter(
                layer.w2_input_scale.max(), requires_grad=False
            )

        if current_platform.is_fp8_fnuz():
            # Normalize the weights and scales
            w13_weight, w13_weight_scale, w13_input_scale = (
                normalize_e4m3fn_to_e4m3fnuz(
                    layer.w13_weight, layer.w13_weight_scale, layer.w13_input_scale
                )
            )
            w2_weight, w2_weight_scale, w2_input_scale = normalize_e4m3fn_to_e4m3fnuz(
                layer.w2_weight, layer.w2_weight_scale, layer.w2_input_scale
            )
            # Reset the parameter
            layer.w13_weight = torch.nn.Parameter(w13_weight, requires_grad=False)
            layer.w13_weight_scale = torch.nn.Parameter(
                w13_weight_scale, requires_grad=False
# ... truncated for analysis ...
                    layer.w13_weight_scale,
                    layer.w2_weight_scale,
                )
            )
            # TODO(rob): once we apply refactor to Quark, switch to using
            # replace_parameter for compatibility with reloading in RL.
            layer.w13_weight = torch.nn.Parameter(w13_weight, requires_grad=False)
            layer.w2_weight = torch.nn.Parameter(w2_weight, requires_grad=False)
            layer.w13_weight_scale = torch.nn.Parameter(
                w13_weight_scale, requires_grad=False
            )
            layer.w2_weight_scale = torch.nn.Parameter(
                w2_weight_scale, requires_grad=False
            )
```
**EN:** Defines function `QuarkW8A8Fp8MoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `current_platform.is_fp8_fnuz`, `torch.nn.Parameter`, `normalize_e4m3fn_to_e4m3fnuz`, `rocm_aiter_ops.shuffle_weights`, `ValueError`, `logger.warning_once`.
**CN:** 定义函数 `QuarkW8A8Fp8MoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `current_platform.is_fp8_fnuz`, `torch.nn.Parameter`, `normalize_e4m3fn_to_e4m3fnuz`, `rocm_aiter_ops.shuffle_weights`, `ValueError`, `logger.warning_once`。

### Method `QuarkW8A8Fp8MoEMethod.apply` (lines 455-516)
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
        if self.rocm_aiter_moe_enabled:
            from vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe import (
                rocm_aiter_fused_experts,
            )

            return rocm_aiter_fused_experts(
                hidden_states=x,
                w1=layer.w13_weight,
                w2=layer.w2_weight,
                topk_weights=topk_weights,
                topk_ids=topk_ids,
                activation=layer.activation,
                apply_router_weight_on_input=layer.apply_router_weight_on_input,
                quant_config=self.moe_quant_config,
                moe_config=layer.moe_config,
                expert_map=layer.expert_map,
            )
        elif self.use_marlin:
            assert layer.activation == MoEActivation.SILU, (
                f"{layer.activation} not supported for Marlin MoE."
            )
            return fused_marlin_moe(
                x,
                layer.w13_weight,
                layer.w2_weight,
                None,
                None,
                layer.w13_weight_scale,
                layer.w2_weight_scale,
# ... truncated for analysis ...

            return fused_experts(
                hidden_states=x,
                w1=layer.w13_weight,
                w2=layer.w2_weight,
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
**EN:** Defines function `QuarkW8A8Fp8MoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `rocm_aiter_fused_experts`, `fused_marlin_moe`, `fused_experts`.
**CN:** 定义函数 `QuarkW8A8Fp8MoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `rocm_aiter_fused_experts`, `fused_marlin_moe`, `fused_experts`。

### Class `QuarkW8A8Int8MoEMethod` overview (lines 519-817)
```python
class QuarkW8A8Int8MoEMethod(QuarkMoEMethod):
    """Quark W8A8 INT8 MoE method."""

    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config
        self.weight_qscheme = self.weight_quant.get("qscheme", "per_tensor")
        self.static_input_scales = not self.input_quant.get("is_dynamic", False)

    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        layer.num_experts = num_experts
```
**EN:** Defines class `QuarkW8A8Int8MoEMethod` with base classes `QuarkMoEMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`. Its docstring says: Quark W8A8 INT8 MoE method.
**CN:** 定义类 `QuarkW8A8Int8MoEMethod`，其基类为 `QuarkMoEMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`。 文档字符串进一步说明了该类的定位。

### Method `QuarkW8A8Int8MoEMethod.__init__` (lines 522-532)
```python
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config
        self.weight_qscheme = self.weight_quant.get("qscheme", "per_tensor")
        self.static_input_scales = not self.input_quant.get("is_dynamic", False)
```
**EN:** Defines function `QuarkW8A8Int8MoEMethod.__init__` with signature `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig)`. It mainly works with `weight_config`, `input_config`, `moe`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `self.weight_quant.get`, `self.input_quant.get`, `super`.
**CN:** 定义函数 `QuarkW8A8Int8MoEMethod.__init__`，其签名为 `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig)`。它主要围绕 `weight_config`, `input_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `self.weight_quant.get`, `self.input_quant.get`, `super`。

### Method `QuarkW8A8Int8MoEMethod.create_weights` (lines 534-692)
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
        layer.num_experts = num_experts
        layer.orig_dtype = params_dtype
        layer.weight_block_size = None
        params_dtype = torch.int8

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size,
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
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w2_weight, extra_weight_attrs)
# ... truncated for analysis ...
                    dtype=torch.float32,
                ),
                requires_grad=False,
            )
            layer.register_parameter("w13_bias", w13_bias)
            set_weight_attrs(w13_bias, extra_weight_attrs)
            w2_bias = torch.nn.Parameter(
                torch.zeros(num_experts, hidden_size, dtype=torch.float32),
                requires_grad=False,
            )
            layer.register_parameter("w2_bias", w2_bias)
            set_weight_attrs(w2_bias, extra_weight_attrs)
        else:
            layer.w13_bias, layer.w2_bias = None, None
```
**EN:** Defines function `QuarkW8A8Int8MoEMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.empty`, `extra_weight_attrs.update`, `torch.zeros`.
**CN:** 定义函数 `QuarkW8A8Int8MoEMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.empty`, `extra_weight_attrs.update`, `torch.zeros`。

### Method `QuarkW8A8Int8MoEMethod.process_weights_after_loading` (lines 694-764)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # Discard zero points (INT8 fused MoE kernel uses symmetric quant)
        for attr in (
            "w13_input_zero_point",
            "w2_input_zero_point",
            "w13_weight_zero_point",
            "w2_weight_zero_point",
        ):
            if hasattr(layer, attr):
                delattr(layer, attr)

        # For static input scales, collapse per-expert scales to single max
        if self.static_input_scales:
            if layer.w13_input_scale is None or layer.w2_input_scale is None:
                raise ValueError(
                    "QuantConfig has static quantization, but found "
                    "activation scales are None."
                )
            if not all_close_1d(layer.w13_input_scale) or not all_close_1d(
                layer.w2_input_scale
            ):
                logger.warning_once(
                    "Found input_scales that are not equal for "
                    "INT8 MoE layer. Using the maximum across experts "
                    "for each layer."
                )
            layer.w13_input_scale = torch.nn.Parameter(
                layer.w13_input_scale.max(), requires_grad=False
            )
            layer.w2_input_scale = torch.nn.Parameter(
                layer.w2_input_scale.max(), requires_grad=False
            )

        # Per-channel scales: 2D [E, N] -> 3D [E, N, 1] for the int8 MoE kernel.
        if self.weight_qscheme == "per_channel":
            for attr in ("w13_weight_scale", "w2_weight_scale"):
                param = getattr(layer, attr, None)
                if param is not None and param.dim() == 2:
# ... truncated for analysis ...
                        layer.w13_weight[expert_id][start : start + shard_size, :],
                        layer.w13_weight_scale[expert_id][shard_id],
                    )
                    layer.w13_weight[expert_id][start : start + shard_size, :], _, _ = (
                        ops.scaled_int8_quant(
                            dq_weight,
                            scale=max_w13_scales[expert_id],
                        )
                    )
                    start += shard_size

            layer.w13_weight_scale = torch.nn.Parameter(
                max_w13_scales, requires_grad=False
            )
```
**EN:** Defines function `QuarkW8A8Int8MoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `hasattr`, `torch.nn.Parameter`, `range`, `delattr`, `ValueError`, `logger.warning_once`.
**CN:** 定义函数 `QuarkW8A8Int8MoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `hasattr`, `torch.nn.Parameter`, `range`, `delattr`, `ValueError`, `logger.warning_once`。

### Method `QuarkW8A8Int8MoEMethod.apply` (lines 794-817)
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

        return fused_experts(
            hidden_states=x,
            w1=layer.w13_weight,
            w2=layer.w2_weight,
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
**EN:** Defines function `QuarkW8A8Int8MoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `fused_experts`.
**CN:** 定义函数 `QuarkW8A8Int8MoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `fused_experts`。

### Class `QuarkW4A8Fp8MoEMethod` overview (lines 820-972)
```python
class QuarkW4A8Fp8MoEMethod(QuarkMoEMethod):
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config

        assert rocm_aiter_ops.is_fused_moe_enabled(), (
            "W4A8 FP8 MoE requires ROCm AITER fused MoE support."
        )

    def create_weights(
        self,
        layer: RoutedExperts,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        params_dtype = torch.uint32
```
**EN:** Defines class `QuarkW4A8Fp8MoEMethod` with base classes `QuarkMoEMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`.
**CN:** 定义类 `QuarkW4A8Fp8MoEMethod`，其基类为 `QuarkMoEMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`。

### Method `QuarkW4A8Fp8MoEMethod.__init__` (lines 821-833)
```python
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config

        assert rocm_aiter_ops.is_fused_moe_enabled(), (
            "W4A8 FP8 MoE requires ROCm AITER fused MoE support."
        )
```
**EN:** Defines function `QuarkW4A8Fp8MoEMethod.__init__` with signature `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig)`. It mainly works with `weight_config`, `input_config`, `moe`; initializes the object state and cached resources. The body uses validation/error handling. Key calls include `super.__init__`, `rocm_aiter_ops.is_fused_moe_enabled`, `super`.
**CN:** 定义函数 `QuarkW4A8Fp8MoEMethod.__init__`，其签名为 `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig)`。它主要围绕 `weight_config`, `input_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含校验或报错逻辑。关键调用包括 `super.__init__`, `rocm_aiter_ops.is_fused_moe_enabled`, `super`。

### Method `QuarkW4A8Fp8MoEMethod.create_weights` (lines 835-902)
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
        params_dtype = torch.uint32
        w13_weight = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // 8,  # INT32 packing for W4
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        w2_weight = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // 8,  # INT32 packing for W4
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        layer.register_parameter("w2_weight", w2_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)
        set_weight_attrs(w2_weight, extra_weight_attrs)

        # Per-tensor fp8 weight scales
        w13_weight_scale = torch.nn.Parameter(
            torch.ones(num_experts, 2, dtype=torch.float32), requires_grad=False
        )
        w2_weight_scale = torch.nn.Parameter(
# ... truncated for analysis ...
            ),
            requires_grad=False,
        )
        w2_weight_scale_2 = torch.nn.Parameter(
            torch.ones(num_experts, hidden_size, dtype=torch.float32),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight_scale_2", w13_weight_scale_2)
        layer.register_parameter("w2_weight_scale_2", w2_weight_scale_2)
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.CHANNEL.value}
        )
        set_weight_attrs(w13_weight_scale_2, extra_weight_attrs)
        set_weight_attrs(w2_weight_scale_2, extra_weight_attrs)
```
**EN:** Defines function `QuarkW4A8Fp8MoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses tensor/kernel operations. Key calls include `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `extra_weight_attrs.update`, `torch.zeros`, `torch.ones`.
**CN:** 定义函数 `QuarkW4A8Fp8MoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含张量或内核操作。关键调用包括 `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `extra_weight_attrs.update`, `torch.zeros`, `torch.ones`。

### Method `QuarkW4A8Fp8MoEMethod.process_weights_after_loading` (lines 904-938)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        shuffled_w13, shuffled_w2 = rocm_aiter_ops.shuffle_weights(
            layer.w13_weight.data, layer.w2_weight.data
        )
        layer.w13_weight = torch.nn.Parameter(shuffled_w13, requires_grad=False)
        layer.w2_weight = torch.nn.Parameter(shuffled_w2, requires_grad=False)

        # INT4-FP8 : offset INT4 w13_weight_scale1 to single w13_weight_scale
        # Fp8 moe kernel needs single fp8 w13_weight_scale for w13 per expert.
        # We won't do requant each expert's fp8 weight (not direct available),
        # instead we adjust half of INT4 w13_weight_scale1 numbers
        shard_size = layer.intermediate_size_per_partition
        max_w13_scales = layer.w13_weight_scale.max(dim=1).values
        assert torch.all(max_w13_scales != 0), "fp8 weight scale cannot be zero."
        for expert_id in range(layer.local_num_experts):
            start = 0
            max_w13_scale_fp8 = max_w13_scales[expert_id]
            for shard_id in range(2):
                if layer.w13_weight_scale[expert_id][shard_id] != max_w13_scale_fp8:
                    int4_rescale = (
                        layer.w13_weight_scale[expert_id][shard_id] / max_w13_scale_fp8
                    )
                    layer.w13_weight_scale_2[expert_id][start : start + shard_size] *= (
                        int4_rescale
                    )
                start += shard_size

        layer.w13_weight_scale = torch.nn.Parameter(max_w13_scales, requires_grad=False)

        # special hack to asm_moe, which takes (weight_scale1 * weight_scale) as post
        # GEMM scaling optimal design - shall apply per-column weight_scale1 before
        # GEMM, and weight_scale post
        for expert_id in range(layer.local_num_experts):
            layer.w13_weight_scale_2[expert_id] *= max_w13_scales[expert_id]
            layer.w2_weight_scale_2[expert_id] *= layer.w2_weight_scale[expert_id]
```
**EN:** Defines function `QuarkW4A8Fp8MoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `rocm_aiter_ops.shuffle_weights`, `torch.nn.Parameter`, `torch.all`, `range`, `layer.w13_weight_scale.max`.
**CN:** 定义函数 `QuarkW4A8Fp8MoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `rocm_aiter_ops.shuffle_weights`, `torch.nn.Parameter`, `torch.all`, `range`, `layer.w13_weight_scale.max`。

### Method `QuarkW4A8Fp8MoEMethod.apply` (lines 948-972)
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
        from vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe import (
            rocm_aiter_fused_experts,
        )

        return rocm_aiter_fused_experts(
            hidden_states=x,
            w1=layer.w13_weight,
            w2=layer.w2_weight,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            activation=layer.activation,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
            quant_config=self.moe_quant_config,
            moe_config=layer.moe_config,
            expert_map=layer.expert_map,
        )
```
**EN:** Defines function `QuarkW4A8Fp8MoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `rocm_aiter_fused_experts`.
**CN:** 定义函数 `QuarkW4A8Fp8MoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `rocm_aiter_fused_experts`。

### Class `QuarkOCP_MX_MoEMethod` overview (lines 975-1519)
```python
class QuarkOCP_MX_MoEMethod(QuarkMoEMethod):
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any] | None,
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config

        weight_qscheme = self.weight_quant.get("qscheme")
        if not weight_qscheme == "per_group":
            raise ValueError(
                "For MX(FP4) Fused MoE layers, only per-group scales "
                f"for weights are supported. Found {weight_qscheme}."
            )  # noqa E501

        self.weight_dtype = self.weight_quant["dtype"].replace("fp", "mxfp")
        if self.input_quant is not None:
            input_quant = self.input_quant["dtype"]
            if input_quant in ["fp4", "fp6_e3m2", "fp6_e2m3"]:
                self.input_dtype = input_quant.replace("fp", "mxfp")
            elif input_quant == "fp8_e4m3":
                self.input_dtype = input_quant.replace("fp8_e4m3", "fp8")
```
**EN:** Defines class `QuarkOCP_MX_MoEMethod` with base classes `QuarkMoEMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 10 direct methods, with notable entries `__init__`, `maybe_roundup_sizes`, `get_packed_dim`, `create_weights`, `process_weights_after_loading`, `_setup_kernel`.
**CN:** 定义类 `QuarkOCP_MX_MoEMethod`，其基类为 `QuarkMoEMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 10 个方法，较重要的包括 `__init__`, `maybe_roundup_sizes`, `get_packed_dim`, `create_weights`, `process_weights_after_loading`, `_setup_kernel`。

### Method `QuarkOCP_MX_MoEMethod.__init__` (lines 976-1102)
```python
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any] | None,
        moe: FusedMoEConfig,
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config

        weight_qscheme = self.weight_quant.get("qscheme")
        if not weight_qscheme == "per_group":
            raise ValueError(
                "For MX(FP4) Fused MoE layers, only per-group scales "
                f"for weights are supported. Found {weight_qscheme}."
            )  # noqa E501

        self.weight_dtype = self.weight_quant["dtype"].replace("fp", "mxfp")
        if self.input_quant is not None:
            input_quant = self.input_quant["dtype"]
            if input_quant in ["fp4", "fp6_e3m2", "fp6_e2m3"]:
                self.input_dtype = input_quant.replace("fp", "mxfp")
            elif input_quant == "fp8_e4m3":
                self.input_dtype = input_quant.replace("fp8_e4m3", "fp8")
            else:
                raise NotImplementedError(
                    f"Current input dtype {input_quant} is not compatible \
                        with OCP MX (weight) MoE quantization. Please open an issue"
                )
        else:
            self.input_dtype = None

        self.fp4_dtype = getattr(torch, "float4_e2m1fn_x2", None)

        self.ocp_mx_scheme = OCP_MX_Scheme.from_quant_dtype(
            self.input_dtype, self.weight_dtype
        )

# ... truncated for analysis ...
        if self.mxfp4_backend != Mxfp4MoeBackend.NONE:
            logger.info_once(
                f"Using {self.mxfp4_backend.value} backend for {self.ocp_mx_scheme}"
            )
        elif self.emulate:
            logger.warning_once(
                f"The current mode (supports_mx={current_platform.supports_mx()}, "
                f"use_rocm_aiter_moe={self.use_rocm_aiter_moe}, "
                f"ocp_mx_scheme={self.ocp_mx_scheme}) "
                "does not support native MXFP4/MXFP6 "
                "computation. Simulated weight dequantization and activation "
                "QDQ (quantize and dequantize) will be used, with the linear "
                "layers computed in high precision."
            )
```
**EN:** Defines function `QuarkOCP_MX_MoEMethod.__init__` with signature `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any] | None, moe: FusedMoEConfig)`. It mainly works with `weight_config`, `input_config`, `moe`; initializes the object state and cached resources. The body uses branching, comprehensions, validation/error handling. Key calls include `super.__init__`, `self.weight_quant.get`, `self.weight_quant.replace`, `getattr`, `OCP_MX_Scheme.from_quant_dtype`, `any`.
**CN:** 定义函数 `QuarkOCP_MX_MoEMethod.__init__`，其签名为 `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any] | None, moe: FusedMoEConfig)`。它主要围绕 `weight_config`, `input_config`, `moe` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、推导式、校验或报错逻辑。关键调用包括 `super.__init__`, `self.weight_quant.get`, `self.weight_quant.replace`, `getattr`, `OCP_MX_Scheme.from_quant_dtype`, `any`。

### Method `QuarkOCP_MX_MoEMethod.create_weights` (lines 1139-1244)
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
        # Add the quantization method used (per tensor/grouped/channel)
        # to ensure the weight scales are loaded in properly
        extra_weight_attrs.update(
            {"quant_method": FusedMoeWeightScaleSupported.BLOCK.value}
        )

        params_dtype = torch.uint8

        # WEIGHTS
        w13_weight = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                2 * intermediate_size_per_partition,
                self.get_packed_dim(hidden_size, self.weight_dtype),
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)

        set_weight_attrs(w13_weight, extra_weight_attrs)

        w2_weight = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                hidden_size,
                self.get_packed_dim(intermediate_size_per_partition, self.weight_dtype),
                dtype=params_dtype,
            ),
# ... truncated for analysis ...
            w13_input_scale = torch.nn.Parameter(
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
**EN:** Defines function `QuarkOCP_MX_MoEMethod.create_weights` with signature `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `extra_weight_attrs.update`, `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.zeros`, `torch.ones`.
**CN:** 定义函数 `QuarkOCP_MX_MoEMethod.create_weights`，其签名为 `create_weights(self, layer: RoutedExperts, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `extra_weight_attrs.update`, `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `torch.zeros`, `torch.ones`。

### Method `QuarkOCP_MX_MoEMethod.apply` (lines 1458-1499)
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
        # For oracle-based kernels (W4A16, W4A8) or emulation kernel
        if self.moe_kernel is not None:
            return self.moe_kernel.apply(
                hidden_states=x,
                w1=layer.w13_weight,
                w2=layer.w2_weight,
                topk_weights=topk_weights,
                topk_ids=topk_ids,
                activation=layer.activation,
                global_num_experts=layer.global_num_experts,
                apply_router_weight_on_input=layer.apply_router_weight_on_input,
                expert_map=layer.expert_map,
                shared_experts=shared_experts,
                shared_experts_input=shared_experts_input,
            )

        # AITER path
        # TODO: Refactor this to use modular MOE kernel as well.
        from vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe import (
            rocm_aiter_fused_experts,
        )

        return rocm_aiter_fused_experts(
            x,
            layer.w13_weight,
            layer.w2_weight,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            activation=layer.activation,
            quant_config=self.moe_quant_config,
            moe_config=layer.moe_config,
            expert_map=layer.expert_map,
        )
```
**EN:** Defines function `QuarkOCP_MX_MoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `rocm_aiter_fused_experts`, `self.moe_kernel.apply`.
**CN:** 定义函数 `QuarkOCP_MX_MoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `rocm_aiter_fused_experts`, `self.moe_kernel.apply`。

### Method `QuarkOCP_MX_MoEMethod.apply_monolithic` (lines 1501-1519)
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
            hidden_states=x,
            w1=layer.w13_weight,
            w2=layer.w2_weight,
            router_logits=router_logits,
            activation=layer.activation,
            global_num_experts=layer.global_num_experts,
            expert_map=layer.expert_map,
            apply_router_weight_on_input=layer.apply_router_weight_on_input,
        )
```
**EN:** Defines function `QuarkOCP_MX_MoEMethod.apply_monolithic` with signature `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `router_logits`, `input_ids`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply_monolithic`.
**CN:** 定义函数 `QuarkOCP_MX_MoEMethod.apply_monolithic`，其签名为 `apply_monolithic(self, layer: RoutedExperts, x: torch.Tensor, router_logits: torch.Tensor, input_ids: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `router_logits`, `input_ids` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply_monolithic`。

### Class `QuarkNvfp4MoEMethod` overview (lines 1522-1745)
```python
class QuarkNvfp4MoEMethod(QuarkMoEMethod):
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
        quant_config: "QuarkConfig",  # type: ignore # noqa E501 # noqa F821
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config
        self.quant_config = quant_config
        self.group_size = 16

        # Select experts implementation.
        self.nvfp4_backend, self.experts_cls = select_nvfp4_moe_backend(
            config=self.moe,
            weight_key=kNvfp4Static,
            activation_key=kNvfp4Dynamic,
        )

    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
```
**EN:** Defines class `QuarkNvfp4MoEMethod` with base classes `QuarkMoEMethod` and decorators none. It acts as a backend or execution-method adapter and exposes 5 direct methods, with notable entries `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`.
**CN:** 定义类 `QuarkNvfp4MoEMethod`，其基类为 `QuarkMoEMethod`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 5 个方法，较重要的包括 `__init__`, `create_weights`, `process_weights_after_loading`, `get_fused_moe_quant_config`, `apply`。

### Method `QuarkNvfp4MoEMethod.__init__` (lines 1523-1541)
```python
    def __init__(
        self,
        weight_config: dict[str, Any],
        input_config: dict[str, Any],
        moe: FusedMoEConfig,
        quant_config: "QuarkConfig",  # type: ignore # noqa E501 # noqa F821
    ):
        super().__init__(moe)
        self.weight_quant = weight_config
        self.input_quant = input_config
        self.quant_config = quant_config
        self.group_size = 16

        # Select experts implementation.
        self.nvfp4_backend, self.experts_cls = select_nvfp4_moe_backend(
            config=self.moe,
            weight_key=kNvfp4Static,
            activation_key=kNvfp4Dynamic,
        )
```
**EN:** Defines function `QuarkNvfp4MoEMethod.__init__` with signature `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig, quant_config: 'QuarkConfig')`. It mainly works with `weight_config`, `input_config`, `moe`, `quant_config`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `select_nvfp4_moe_backend`, `super`.
**CN:** 定义函数 `QuarkNvfp4MoEMethod.__init__`，其签名为 `__init__(self, weight_config: dict[str, Any], input_config: dict[str, Any], moe: FusedMoEConfig, quant_config: 'QuarkConfig')`。它主要围绕 `weight_config`, `input_config`, `moe`, `quant_config` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `select_nvfp4_moe_backend`, `super`。

### Method `QuarkNvfp4MoEMethod.create_weights` (lines 1543-1647)
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
        layer.num_experts = num_experts
        layer.params_dtype = params_dtype
        layer.quant_config = self.quant_config
        weight_dtype = torch.uint8
        weight_scale_dtype = torch.float8_e4m3fn
        w13_num_shards = 2 if self.moe.is_act_and_mul else 1

        # GEMM 1 - w13 weight
        w13_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                w13_num_shards * intermediate_size_per_partition,
                # 2 fp4 items are packed in the input dimension
                hidden_size // 2,
                dtype=weight_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_weight", w13_weight)
        set_weight_attrs(w13_weight, extra_weight_attrs)

        # GEMM 2 - w2 weight
        w2_weight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                # 2 fp4 items are packed in the input dimension
                intermediate_size_per_partition // 2,
                dtype=weight_dtype,
# ... truncated for analysis ...
        # Input global scales (per-tensor FP32 scales)
        w13_input_scale_2 = torch.nn.Parameter(
            torch.empty(num_experts, w13_num_shards, dtype=torch.float32),
            requires_grad=False,
        )
        layer.register_parameter("w13_input_scale_2", w13_input_scale_2)
        set_weight_attrs(w13_input_scale_2, extra_weight_attrs)

        w2_input_scale_2 = torch.nn.Parameter(
            torch.empty(num_experts, dtype=torch.float32),
            requires_grad=False,
        )
        layer.register_parameter("w2_input_scale_2", w2_input_scale_2)
        set_weight_attrs(w2_input_scale_2, extra_weight_attrs)
```
**EN:** Defines function `QuarkNvfp4MoEMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses branching, tensor/kernel operations. Key calls include `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `extra_weight_attrs.update`, `torch.empty`.
**CN:** 定义函数 `QuarkNvfp4MoEMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, num_experts: int, hidden_size: int, intermediate_size_per_partition: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `num_experts`, `hidden_size`, `intermediate_size_per_partition`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.nn.Parameter`, `layer.register_parameter`, `set_weight_attrs`, `extra_weight_attrs.update`, `torch.empty`。

### Method `QuarkNvfp4MoEMethod.process_weights_after_loading` (lines 1649-1708)
```python
    def process_weights_after_loading(self, layer: RoutedExperts) -> None:
        """
        Convert NVFP4 MoE weights into kernel format and setup the kernel.
        """

        if not torch.allclose(
            layer.w13_weight_scale_2[:, 0], layer.w13_weight_scale_2[:, 1]
        ):
            raise ValueError("Different global scales for w1 and w3 is not supported.")

        # Use a single gscale for w13
        w13_weight_scale_2 = torch.maximum(
            layer.w13_weight_scale_2[:, 0], layer.w13_weight_scale_2[:, 1]
        ).contiguous()

        w2_weight_scale_2 = layer.w2_weight_scale_2

        (
            w13,
            w13_scale,
            w13_scale_2,
            a13_scale,
            w2,
            w2_scale,
            w2_scale_2,
            a2_scale,
        ) = convert_to_nvfp4_moe_kernel_format(
            nvfp4_backend=self.nvfp4_backend,
            layer=layer,
            w13=layer.w13_weight,
            w13_scale=layer.w13_weight_scale,
            w13_scale_2=w13_weight_scale_2,
            a13_scale=layer.w13_input_scale_2,
            w2=layer.w2_weight,
            w2_scale=layer.w2_weight_scale,
            w2_scale_2=w2_weight_scale_2,
            a2_scale=layer.w2_input_scale_2,
            is_act_and_mul=self.moe.is_act_and_mul,
        )

        replace_parameter(layer, "w13_weight", w13)
        replace_parameter(layer, "w13_weight_scale", w13_scale)
        replace_parameter(layer, "w13_weight_scale_2", w13_scale_2)
        replace_parameter(layer, "w13_input_scale_2", a13_scale)

        replace_parameter(layer, "w2_weight", w2)
        replace_parameter(layer, "w2_weight_scale", w2_scale)
        replace_parameter(layer, "w2_weight_scale_2", w2_scale_2)
        replace_parameter(layer, "w2_input_scale_2", a2_scale)

        # Setup modular kernel.
        self.moe_quant_config = self.get_fused_moe_quant_config(layer)
        if self.moe_quant_config:
            assert self.experts_cls is not None
            self.moe_kernel = make_nvfp4_moe_kernel(
                moe_quant_config=self.moe_quant_config,
                moe_config=self.moe,
                experts_cls=self.experts_cls,
                routing_tables=layer._expert_routing_tables(),
            )
```
**EN:** Defines function `QuarkNvfp4MoEMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: RoutedExperts) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `torch.maximum.contiguous`, `convert_to_nvfp4_moe_kernel_format`, `replace_parameter`, `self.get_fused_moe_quant_config`, `torch.allclose`, `ValueError`.
**CN:** 定义函数 `QuarkNvfp4MoEMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: RoutedExperts) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `torch.maximum.contiguous`, `convert_to_nvfp4_moe_kernel_format`, `replace_parameter`, `self.get_fused_moe_quant_config`, `torch.allclose`, `ValueError`。

### Method `QuarkNvfp4MoEMethod.apply` (lines 1723-1745)
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
**EN:** Defines function `QuarkNvfp4MoEMethod.apply` with signature `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`. It mainly works with `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input`; implements one step in the quantized-weight execution flow. The body uses validation/error handling. Key calls include `self.moe_kernel.apply`.
**CN:** 定义函数 `QuarkNvfp4MoEMethod.apply`，其签名为 `apply(self, layer: RoutedExperts, x: torch.Tensor, topk_weights: torch.Tensor, topk_ids: torch.Tensor, shared_experts: SharedExperts | None, shared_experts_input: torch.Tensor | None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `topk_weights`, `topk_ids`, `shared_experts`, `shared_experts_input` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 `self.moe_kernel.apply`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `QuarkMoEMethod`, `QuarkW8A8Fp8MoEMethod`, `QuarkW8A8Int8MoEMethod`, `QuarkW4A8Fp8MoEMethod`, `QuarkOCP_MX_MoEMethod`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `QuarkMoEMethod`, `QuarkW8A8Fp8MoEMethod`, `QuarkW8A8Int8MoEMethod`, `QuarkW4A8Fp8MoEMethod`, `QuarkOCP_MX_MoEMethod`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `torch`
- **Internal / 内部**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm._aiter_ops`, `vllm.config`, `vllm.logger`, `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.marlin_moe`, `vllm.model_executor.layers.fused_moe.oracle.mxfp4`, `vllm.model_executor.layers.fused_moe.oracle.nvfp4`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.scalar_type`
