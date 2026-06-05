# fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `Fp8Config`, and `Fp8LinearMethod`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `Fp8Config` 和 `Fp8LinearMethod` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-54: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Union

import torch
from torch.nn import Module
from torch.nn.parameter import Parameter

from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_tensor_model_parallel_world_size,
)
from sglang.multimodal_gen.runtime.layers.linear import (
    LinearMethodBase,
    UnquantizedLinearMethod,
)
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.multimodal_gen.runtime.models.parameter import (
    BlockQuantScaleParameter,
    ModelWeightParameter,
    PerTensorScaleParameter,
)
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.common import (
    cpu_has_amx_support,
    get_bool_env_var,
    use_intel_amx_backend,
)
from sglang.srt.layers.amx_utils import _amx_process_weight_after_loading
from sglang.srt.layers.quantization.fp8_kernel import (
    is_fp8_fnuz,
    per_token_group_quant_fp8,
)
from sglang.srt.layers.quantization.fp8_utils import (
    apply_fp8_linear,
    can_auto_enable_marlin_fp8,
    cutlass_fp8_supported,
    dispatch_w8a8_block_fp8_linear,
    input_to_float8,
    normalize_e4m3fn_to_e4m3fnuz,
    requant_weight_ue8m0_inplace,
)
from sglang.srt.layers.quantization.marlin_utils_fp8 import (
    apply_fp8_marlin_linear,
    prepare_fp8_layer_for_marlin,
)
from sglang.srt.layers.quantization.utils import (
    convert_to_channelwise,
    is_layer_skipped,
    requantize_with_max_scale,
)
```
**EN:** This block establishes the module context and imports `__future__`, `logging`, `typing`, `torch`, `torch.nn`, and `torch.nn.parameter`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`logging`、`typing`、`torch`、`torch.nn` 和 `torch.nn.parameter`。这些依赖为后续实现提供所需符号。

### Lines 56-74: supporting statements / 辅助语句
```python
if TYPE_CHECKING:
    from sglang.srt.layers.quantization.w4afp8 import W4AFp8Config

_is_hip = current_platform.is_hip()
_is_cuda = current_platform.is_cuda()
_is_npu = current_platform.is_npu()
_is_cpu_amx_available = cpu_has_amx_support()
_is_cpu = current_platform.is_cpu()
_is_fp8_fnuz = is_fp8_fnuz()
_use_hip_int4 = get_bool_env_var("SGLANG_INT4_WEIGHT") and _is_hip
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip

if _use_aiter or _use_hip_int4:
    pass


ACTIVATION_SCHEMES = ["static", "dynamic"]

logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_hip`, `_is_cuda`, `_is_npu`, `_is_cpu_amx_available`, `_is_cpu`, and `_is_fp8_fnuz`. The code collaborates with `current_platform.is_hip`, `current_platform.is_cuda`, `current_platform.is_npu`, `cpu_has_amx_support`, `current_platform.is_cpu`, and `is_fp8_fnuz`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_hip`、`_is_cuda`、`_is_npu`、`_is_cpu_amx_available`、`_is_cpu` 和 `_is_fp8_fnuz` 等名称。 代码会与 `current_platform.is_hip`、`current_platform.is_cuda`、`current_platform.is_npu`、`cpu_has_amx_support`、`current_platform.is_cpu` 和 `is_fp8_fnuz` 协同工作。

### Lines 77-79: `Fp8Config` class overview / `Fp8Config` 类概览
```python
class Fp8Config(QuantizationConfig):
    """Config class for FP8."""
```
**EN:** This block defines class `Fp8Config`. Config class for FP8. It inherits from `QuantizationConfig`.
**CN:** 该代码块定义了类 `Fp8Config`。 它用于封装 fp8 config 相关行为。 它继承自 `QuantizationConfig`。

### Lines 80-109: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        is_checkpoint_fp8_serialized: bool = False,
        activation_scheme: str = "dynamic",
        ignored_layers: Optional[List[str]] = None,
        weight_block_size: List[int] = None,
        packed_modules_mapping: Optional[Dict[str, List[str]]] = None,
    ) -> None:
        self.is_checkpoint_fp8_serialized = is_checkpoint_fp8_serialized
        if is_checkpoint_fp8_serialized:
            logger.info("Detected fp8 checkpoint.")
        if activation_scheme not in ACTIVATION_SCHEMES:
            raise ValueError(f"Unsupported activation scheme {activation_scheme}")
        self.activation_scheme = activation_scheme
        self.ignored_layers = ignored_layers or []
        self.packed_modules_mapping = packed_modules_mapping or {}
        if weight_block_size is not None:
            if not is_checkpoint_fp8_serialized:
                raise ValueError(
                    f"The block-wise quantization only supports fp8-serialized checkpoint for now."
                )
            if len(weight_block_size) != 2:
                raise ValueError(
                    f"The quantization block size of weight must have 2 dimensions, but got {len(weight_block_size)} dimensions."
                )
            if activation_scheme != "dynamic":
                raise ValueError(
                    f"The block-wise quantization only supports dynamic activation scheme for now, but got {activation_scheme} activation scheme."
                )
        self.weight_block_size = weight_block_size
```
**EN:** This block defines method `__init__` on `Fp8Config`. It initializes the instance state. Key calls include `logger.info`, `ValueError`, and `len`. The implementation branches on conditions. Parameters such as `is_checkpoint_fp8_serialized`, `activation_scheme`, `ignored_layers`, `weight_block_size`, and `packed_modules_mapping` drive the behavior in this section.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `logger.info`、`ValueError` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `is_checkpoint_fp8_serialized`、`activation_scheme`、`ignored_layers`、`weight_block_size` 和 `packed_modules_mapping` 等参数驱动。

### Lines 111-113: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "fp8"
```
**EN:** This block defines method `get_name` on `Fp8Config`. It retrieves name.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `get_name`。 它用于获取name。

### Lines 115-117: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.half]
```
**EN:** This block defines method `get_supported_act_dtypes` on `Fp8Config`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 119-121: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines method `get_min_capability` on `Fp8Config`. It retrieves min capability.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 123-125: `get_config_filenames` implementation / `get_config_filenames` 实现
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines method `get_config_filenames` on `Fp8Config`. It retrieves config filenames.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `get_config_filenames`。 它用于获取config filenames。

### Lines 127-144: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> Fp8Config:
        quant_method = cls.get_from_keys(config, ["quant_method"])
        is_checkpoint_fp8_serialized = "fp8" in quant_method
        activation_scheme = cls.get_from_keys(config, ["activation_scheme"])
        ignored_layers = cls.get_from_keys_or(
            config, ["ignored_layers", "modules_to_not_convert"], None
        )
        if ignored_layers:
            # hacking ministral
            ignored_layers = [layer.replace("model.", "") for layer in ignored_layers]
        weight_block_size = cls.get_from_keys_or(config, ["weight_block_size"], None)
        return cls(
            is_checkpoint_fp8_serialized=is_checkpoint_fp8_serialized,
            activation_scheme=activation_scheme,
            ignored_layers=ignored_layers,
            weight_block_size=weight_block_size,
        )
```
**EN:** This block defines method `from_config` on `Fp8Config`. It constructs from config. Key calls include `cls.get_from_keys`, `cls.get_from_keys_or`, `cls`, and `layer.replace`. The implementation branches on conditions. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `cls.get_from_keys`、`cls.get_from_keys_or`、`cls` 和 `layer.replace`。 实现中包含条件分支。 本段逻辑主要由 `config` 等参数驱动。

### Lines 146-159: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        from sglang.multimodal_gen.runtime.layers.linear import LinearBase

        if isinstance(layer, LinearBase):
            if is_layer_skipped(
                prefix,
                self.ignored_layers,
                fused_mapping=self.packed_modules_mapping,
            ):
                return UnquantizedLinearMethod()
            return Fp8LinearMethod(self)
        return None
```
**EN:** This block defines method `get_quant_method` on `Fp8Config`. It retrieves quant method. Key calls include `isinstance`, `is_layer_skipped`, `Fp8LinearMethod`, and `UnquantizedLinearMethod`. The implementation branches on conditions. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `isinstance`、`is_layer_skipped`、`Fp8LinearMethod` 和 `UnquantizedLinearMethod`。 实现中包含条件分支。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 161-162: `get_scaled_act_names` implementation / `get_scaled_act_names` 实现
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines method `get_scaled_act_names` on `Fp8Config`. It retrieves scaled act names.
**CN:** 该代码块定义了 `Fp8Config` 的方法 `get_scaled_act_names`。 它用于获取scaled act names。

### Lines 165-182: `Fp8LinearMethod` class overview / `Fp8LinearMethod` 类概览
```python
class Fp8LinearMethod(LinearMethodBase):
    """Linear method for FP8.
    Supports loading FP8 checkpoints with static weight scale and
    dynamic/static activation scale.

    Also supports loading quantized FP16/BF16 model checkpoints with dynamic
    activation scaling. The weight scaling factor will be initialized after
    the model weights are loaded.

    Limitations:
    1. Only support per-tensor quantization due to torch._scaled_mm support.
    2. Only support float8_e4m3fn data type due to the limitation of
       torch._scaled_mm (https://github.com/pytorch/pytorch/blob/2e48b39603411a41c5025efbe52f89560b827825/aten/src/ATen/native/cuda/Blas.cpp#L854-L856)

    Args:
        quant_config: The quantization config.
    """
```
**EN:** This block defines class `Fp8LinearMethod`. Linear method for FP8. Supports loading FP8 checkpoints with static weight scale and dynamic/static activation scale. It inherits from `LinearMethodBase`.
**CN:** 该代码块定义了类 `Fp8LinearMethod`。 它用于封装 fp8 linear method 相关行为。 它继承自 `LinearMethodBase`。

### Lines 183-197: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, quant_config: Union[Fp8Config, W4AFp8Config]):
        self.quant_config = quant_config
        self.cutlass_fp8_supported = cutlass_fp8_supported()

        # For GPUs that lack FP8 hardware support, we can leverage the Marlin
        # kernel for fast weight-only FP8 quantization
        self.use_marlin = False
        if _is_cuda:
            force_marlin = get_bool_env_var("SGLANG_FORCE_FP8_MARLIN")
            auto_enable = can_auto_enable_marlin_fp8()
            self.use_marlin = force_marlin or auto_enable

        self.block_quant = self.quant_config.weight_block_size is not None

        self.w8a8_block_fp8_linear = dispatch_w8a8_block_fp8_linear()
```
**EN:** This block defines method `__init__` on `Fp8LinearMethod`. It initializes the instance state. Key calls include `cutlass_fp8_supported`, `dispatch_w8a8_block_fp8_linear`, `get_bool_env_var`, and `can_auto_enable_marlin_fp8`. The implementation branches on conditions. Parameters such as `quant_config` drive the behavior in this section.
**CN:** 该代码块定义了 `Fp8LinearMethod` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `cutlass_fp8_supported`、`dispatch_w8a8_block_fp8_linear`、`get_bool_env_var` 和 `can_auto_enable_marlin_fp8`。 实现中包含条件分支。 本段逻辑主要由 `quant_config` 等参数驱动。

### Lines 199-306: `create_weights` implementation / `create_weights` 实现
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
        output_size_per_partition = sum(output_partition_sizes)
        weight_loader = extra_weight_attrs.get("weight_loader")

        tp_size = get_tensor_model_parallel_world_size()
        if self.block_quant:
            block_n, block_k = (
                self.quant_config.weight_block_size[0],
                self.quant_config.weight_block_size[1],
            )
            # Required by row parallel
            if tp_size > 1 and input_size // input_size_per_partition == tp_size:
                if input_size_per_partition % block_k != 0:
                    raise ValueError(
                        f"Weight input_size_per_partition = "
                        f"{input_size_per_partition} is not divisible by "
                        f"weight quantization block_k = {block_k}."
                    )
            # Required by column parallel or enabling merged weights
            if (
                tp_size > 1 and output_size // output_size_per_partition == tp_size
            ) or len(output_partition_sizes) > 1:
                for output_partition_size in output_partition_sizes:
                    if output_partition_size % block_n != 0:
                        raise ValueError(
                            f"Weight output_partition_size = "
                            f"{output_partition_size} is not divisible by "
                            f"weight quantization block_n = {block_n}."
                        )

        layer.logical_widths = output_partition_sizes
        layer.input_size_per_partition = input_size_per_partition
        layer.output_size_per_partition = output_size_per_partition
        layer.orig_dtype = params_dtype

        # WEIGHT
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

        # If checkpoint is serialized fp8, load them.
        # Otherwise, wait until process_weights_after_loading.
        if self.quant_config.is_checkpoint_fp8_serialized:
            # WEIGHT SCALE
            if self.block_quant:
                if hasattr(self.quant_config, "activation_scheme"):
                    assert self.quant_config.activation_scheme == "dynamic"
                elif hasattr(self.quant_config, "linear_activation_scheme"):
                    assert self.quant_config.linear_activation_scheme == "dynamic"
                scale = BlockQuantScaleParameter(
                    data=torch.empty(
                        (output_size_per_partition + block_n - 1) // block_n,
                        (input_size_per_partition + block_k - 1) // block_k,
                        dtype=torch.float32,
                    ),
                    input_dim=1,
                    output_dim=0,
                    weight_loader=weight_loader,
                )
                scale.format_ue8m0 = False
                scale[:] = torch.finfo(torch.float32).min
                layer.register_parameter("weight_scale_inv", scale)
            else:
                scale = PerTensorScaleParameter(
                    data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                    weight_loader=weight_loader,
                )
                scale[:] = torch.finfo(torch.float32).min
                layer.register_parameter("weight_scale", scale)

            # INPUT ACTIVATION SCALE
            if (
                hasattr(self.quant_config, "activation_scheme")
                and self.quant_config.activation_scheme == "static"
            ) or (
                hasattr(self.quant_config, "linear_activation_scheme")
                and self.quant_config.linear_activation_scheme == "static"
            ):
                scale = PerTensorScaleParameter(
                    data=torch.empty(len(output_partition_sizes), dtype=torch.float32),
                    weight_loader=weight_loader,
                )

                scale[:] = torch.finfo(torch.float32).min
                layer.register_parameter("input_scale", scale)
            else:
                layer.register_parameter("input_scale", None)
```
**EN:** This block defines method `create_weights` on `Fp8LinearMethod`. It creates weights. Key calls include `sum`, `extra_weight_attrs.get`, `get_tensor_model_parallel_world_size`, `ModelWeightParameter`, and `layer.register_parameter`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `Fp8LinearMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `sum`、`extra_weight_attrs.get`、`get_tensor_model_parallel_world_size`、`ModelWeightParameter` 和 `layer.register_parameter`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 308-446: `process_weights_after_loading` implementation / `process_weights_after_loading` 实现
```python
    def process_weights_after_loading(self, layer: Module) -> None:
        if self.block_quant:
            # If ROCm, normalize the weights and scales to e4m3fnuz
            if _is_fp8_fnuz:
                # activation_scheme: dynamic
                weight, weight_scale, _ = normalize_e4m3fn_to_e4m3fnuz(
                    weight=layer.weight,
                    weight_scale=layer.weight_scale_inv,
                    input_scale=None,
                )
                layer.input_scale = None
            elif _is_cpu:
                assert (
                    _is_cpu_amx_available
                ), "Fp8LinearMethod on CPU requires that CPU has AMX support"
                _amx_process_weight_after_loading(layer, ["weight"])
                layer.weight_scale_inv = torch.nn.Parameter(
                    layer.weight_scale_inv.data, requires_grad=False
                )
                return
            else:
                # For fp8 linear weights run with deepgemm, the weights and scales need be requantized to ue8m0
                from sglang.srt.layers.quantization.fp8_utils import (
                    deepgemm_w8a8_block_fp8_linear_with_fallback,
                )
                from sglang.srt.model_loader.utils import (
                    should_deepgemm_weight_requant_ue8m0,
                )

                if (
                    should_deepgemm_weight_requant_ue8m0(
                        weight_block_size=getattr(
                            self.quant_config, "weight_block_size", None
                        ),
                    )
                    and (
                        self.w8a8_block_fp8_linear
                        is deepgemm_w8a8_block_fp8_linear_with_fallback
                    )
                    and (not layer.weight_scale_inv.format_ue8m0)
                ):
                    requant_weight_ue8m0_inplace(
                        layer.weight,
                        layer.weight_scale_inv,
                        self.quant_config.weight_block_size,
                    )
                    layer.weight_scale_inv.format_ue8m0 = True
                weight, weight_scale = layer.weight.data, layer.weight_scale_inv.data

            layer.weight.data = weight.data
            layer.weight_scale_inv.data = weight_scale.data
        else:
            layer.weight = Parameter(layer.weight.data, requires_grad=False)

            # If checkpoint not serialized fp8, quantize the weights.
            if not self.quant_config.is_checkpoint_fp8_serialized:
                if self.cutlass_fp8_supported or self.use_marlin:
                    # apply per-channel quantization default as
                    # cutlass sgl-kernel and marlin only support per-channel scale
                    qweight, weight_scale = per_token_group_quant_fp8(
                        layer.weight, layer.weight.shape[-1]
                    )
                    weight_scale = weight_scale.t().contiguous()
                else:
                    # per-tensor quantization
                    qweight, weight_scale = input_to_float8(layer.weight)

                # Update the layer with the new values.
                layer.weight = Parameter(qweight.t(), requires_grad=False)
                layer.weight_scale = Parameter(weight_scale, requires_grad=False)
                layer.input_scale = None

            # If checkpoint is fp8, handle that there are N scales for N
            # shards in a fused module
            else:
                layer.weight_scale = Parameter(
                    layer.weight_scale.data, requires_grad=False
                )
                if (
                    hasattr(self.quant_config, "activation_scheme")
                    and self.quant_config.activation_scheme == "static"
                ) or (
                    hasattr(self.quant_config, "linear_activation_scheme")
                    and self.quant_config.linear_activation_scheme == "static"
                ):
                    layer.input_scale = Parameter(
                        layer.input_scale.data, requires_grad=False
                    )

                # cutlass sgl-kernel and marlin only support per-channel scale
                if self.cutlass_fp8_supported or self.use_marlin:
                    weight = layer.weight
                    weight_scale = convert_to_channelwise(
                        layer.weight_scale, layer.logical_widths
                    )
                else:
                    # Dequant -> Quant with max scale so we can run per tensor.
                    weight = layer.weight
                    weight_scale = layer.weight_scale
                    # If ROCm, normalize the weights and scales to e4m3fnuz
                    if _is_fp8_fnuz:
                        weight, weight_scale, input_scale = (
                            normalize_e4m3fn_to_e4m3fnuz(
                                weight=weight,
                                weight_scale=weight_scale,
                                input_scale=layer.input_scale,
                            )
                        )
                        if input_scale is not None:
                            layer.input_scale = Parameter(
                                input_scale, requires_grad=False
                            )

                    weight_scale, weight = requantize_with_max_scale(
                        weight=weight,
                        weight_scale=weight_scale,
                        logical_widths=layer.logical_widths,
                    )

                # Update layer with new values.
                layer.weight = Parameter(weight.t(), requires_grad=False)
                layer.weight_scale = Parameter(weight_scale, requires_grad=False)
                if (
                    hasattr(self.quant_config, "activation_scheme")
                    and self.quant_config.activation_scheme == "static"
                ) or (
                    hasattr(self.quant_config, "linear_activation_scheme")
                    and self.quant_config.linear_activation_scheme == "static"
                ):
                    layer.input_scale = Parameter(
                        layer.input_scale.max(), requires_grad=False
                    )

        if self.use_marlin:
            if self.block_quant:
                layer.weight_block_size = self.quant_config.weight_block_size
            prepare_fp8_layer_for_marlin(layer, not self.block_quant)
            # Activations not quantized for marlin.
            del layer.input_scale
```
**EN:** This block defines method `process_weights_after_loading` on `Fp8LinearMethod`. It processes weights after loading. Key calls include `Parameter`, `prepare_fp8_layer_for_marlin`, `normalize_e4m3fn_to_e4m3fnuz`, `_amx_process_weight_after_loading`, and `torch.nn.Parameter`. The implementation branches on conditions. Parameters such as `layer` drive the behavior in this section.
**CN:** 该代码块定义了 `Fp8LinearMethod` 的方法 `process_weights_after_loading`。 它用于处理weights after loading。 关键调用包括 `Parameter`、`prepare_fp8_layer_for_marlin`、`normalize_e4m3fn_to_e4m3fnuz`、`_amx_process_weight_after_loading` 和 `torch.nn.Parameter`。 实现中包含条件分支。 本段逻辑主要由 `layer` 等参数驱动。

### Lines 448-504: `apply` implementation / `apply` 实现
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if self.use_marlin:
            return apply_fp8_marlin_linear(
                input=x,
                weight=layer.weight,
                weight_scale=layer.weight_scale,
                workspace=layer.workspace,
                size_n=layer.output_size_per_partition,
                size_k=layer.input_size_per_partition,
                bias=bias,
            )

        if self.block_quant:
            if use_intel_amx_backend(layer):
                return torch.ops.sgl_kernel.fp8_scaled_mm_cpu(
                    x,
                    layer.weight,
                    layer.weight_scale_inv,
                    self.quant_config.weight_block_size,
                    bias,
                    x.dtype,
                    True,  # is_vnni
                )

            if isinstance(x, tuple):
                return self.w8a8_block_fp8_linear(
                    input=x[0],
                    weight=layer.weight,
                    block_size=self.quant_config.weight_block_size,
                    weight_scale=layer.weight_scale_inv,
                    input_scale=x[1],
                    bias=bias,
                )

            return self.w8a8_block_fp8_linear(
                input=x,
                weight=layer.weight,
                block_size=self.quant_config.weight_block_size,
                weight_scale=layer.weight_scale_inv,
                input_scale=None,
                bias=bias,
            )

        return apply_fp8_linear(
            input=x,
            weight=layer.weight,
            weight_scale=layer.weight_scale,
            input_scale=layer.input_scale,
            bias=bias,
            cutlass_fp8_supported=self.cutlass_fp8_supported,
            use_per_token_if_dynamic=False,
        )
```
**EN:** This block defines method `apply` on `Fp8LinearMethod`. It applies function. Key calls include `apply_fp8_linear`, `apply_fp8_marlin_linear`, `use_intel_amx_backend`, `isinstance`, and `self.w8a8_block_fp8_linear`. The implementation branches on conditions. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `Fp8LinearMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `apply_fp8_linear`、`apply_fp8_marlin_linear`、`use_intel_amx_backend`、`isinstance` 和 `self.w8a8_block_fp8_linear`。 实现中包含条件分支。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `Fp8Config`: Config class for FP8. / 核心类，用于封装 fp8 config 相关行为。
- `Fp8LinearMethod`: Linear method for FP8. / 核心类，用于封装 fp8 linear method 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.nn`, `torch.nn.parameter`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.common`, `sglang.srt.layers.amx_utils`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.quantization.marlin_utils_fp8`, `sglang.srt.layers.quantization.utils`, `sglang.srt.layers.quantization.w4afp8`

- **Total lines / 总行数**: 504
