# moe_wna16.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/moe_wna16.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for moe wna16 quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 MoE wna16 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/quantization/moe_wna16.py
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any, Dict, List, Optional

import numpy as np
import torch

from sglang.srt.distributed import get_tensor_model_parallel_rank
from sglang.srt.distributed.parallel_state import get_tp_group
from sglang.srt.layers.moe import MoeRunner, MoeRunnerBackend, MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton import TritonMoeQuantInfo
from sglang.srt.layers.quantization.awq import AWQConfig
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.gptq import GPTQConfig, GPTQMarlinConfig
from sglang.srt.layers.quantization.unquant import (
    UnquantizedFusedMoEMethod,
    UnquantizedLinearMethod,
)
from sglang.srt.utils import get_device_capability, set_weight_attrs
```
**EN:** This block imports __future__, logging, numpy, torch, sglang.srt.distributed, sglang.srt.distributed.parallel_state, sglang.srt.layers.linear, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, logging, numpy, torch, sglang.srt.distributed, sglang.srt.distributed.parallel_state, sglang.srt.layers.linear, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 29-29: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 31-35: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 38-65: get_weight_perm()
```python
def get_weight_perm(num_bits: int):
    perm_list: List[int] = []
    for i in range(32):
        perm1: List[int] = []
        col = i // 4
        for block in [0, 1]:
            for row in [
                2 * (i % 4),
                2 * (i % 4) + 1,
                2 * (i % 4 + 4),
                2 * (i % 4 + 4) + 1,
            ]:
                perm1.append(16 * row + col + 8 * block)
        for j in range(4):
            perm_list.extend([p + 256 * j for p in perm1])

    perm = np.array(perm_list)

    if num_bits == 4:
        interleave = np.array([0, 2, 4, 6, 1, 3, 5, 7])
    elif num_bits == 8:
        interleave = np.array([0, 2, 1, 3])
    else:
        raise Exception("num_bits must be 4 or 8, got {}".format(num_bits))

    perm = perm.reshape((-1, len(interleave)))[:, interleave].ravel()
    perm = torch.from_numpy(perm)
    return perm
```
**EN:** This block defines `get_weight_perm()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `get_weight_perm()`，用于为调用方获取或计算派生值。

### Lines 68-70: class MoeWNA16Config: definition
```python
class MoeWNA16Config(QuantizationConfig):
    """Config class for MOE WNA16 (W8A16/W4A16) quantization."""
```
**EN:** This block declares `MoeWNA16Config`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_name, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `MoeWNA16Config`，它是量化栈中的配置类，组织了 __init__, get_name, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 71-115: MoeWNA16Config.__init__()
```python
    def __init__(
        self,
        linear_quant_method: str,
        weight_bits: int,
        group_size: int,
        has_zp: bool,
        lm_head_quantized: bool,
        modules_to_not_convert: Optional[List[str]],
        full_config: Dict[str, Any],
    ) -> None:
        super().__init__()
        self.weight_bits = weight_bits
        self.group_size = group_size
        self.has_zp = has_zp
        self.bit8_pack_factor = 8 // self.weight_bits
        self.lm_head_quantized = lm_head_quantized
        self.linear_quant_method = linear_quant_method
        self.full_config = full_config
        self.use_marlin = False
        # Avoid circular import

        if self.linear_quant_method == "gptq":
            self.use_marlin = GPTQMarlinConfig.is_gptq_marlin_compatible(full_config)
        elif self.linear_quant_method == "awq":
            capability_tuple = get_device_capability()
            device_capability = (
                -1
                if capability_tuple is None
                else capability_tuple[0] * 10 + capability_tuple[1]
            )
            awq_min_capability = AWQConfig.get_min_capability()
            if device_capability < awq_min_capability:
                raise ValueError(
                    "The quantization method moe_wna16 + awq is not supported "
                    "for the current GPU. "
                    f"Minimum capability: {awq_min_capability}. "
                    f"Current capability: {device_capability}."
                )
        else:
            raise ValueError("moe_wna16 only support gptq and awq.")

        if modules_to_not_convert is None:
            self.modules_to_not_convert = []
        else:
            self.modules_to_not_convert = modules_to_not_convert
```
**EN:** This block defines `MoeWNA16Config.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `MoeWNA16Config.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 117-119: MoeWNA16Config.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "moe_wna16"
```
**EN:** This block defines `MoeWNA16Config.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MoeWNA16Config.get_name()`，用于为调用方获取或计算派生值。

### Lines 121-123: MoeWNA16Config.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.bfloat16, torch.half]
```
**EN:** This block defines `MoeWNA16Config.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MoeWNA16Config.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 125-127: MoeWNA16Config.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines `MoeWNA16Config.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MoeWNA16Config.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 129-131: MoeWNA16Config.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return ["quantize_config.json"]
```
**EN:** This block defines `MoeWNA16Config.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MoeWNA16Config.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 133-134: MoeWNA16Config.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        raise NotImplementedError
```
**EN:** This block defines `MoeWNA16Config.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MoeWNA16Config.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 136-161: MoeWNA16Config.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> MoeWNA16Config:
        quant_method = cls.get_from_keys(config, ["quant_method"])
        weight_bits = cls.get_from_keys(config, ["bits"])
        group_size = cls.get_from_keys(config, ["group_size"])
        lm_head_quantized = cls.get_from_keys_or(config, ["lm_head"], default=False)
        if quant_method == "gptq":
            has_zp = not cls.get_from_keys(config, ["sym"])
            modules_to_not_convert = []
        elif quant_method == "awq":
            has_zp = cls.get_from_keys(config, ["zero_point"])
            modules_to_not_convert = cls.get_from_keys_or(
                config, ["modules_to_not_convert"], None
            )
        else:
            raise ValueError("moe_wna16 only support gptq and awq.")

        return cls(
            quant_method,
            weight_bits,
            group_size,
            has_zp,
            lm_head_quantized,
            modules_to_not_convert,
            config,
        )
```
**EN:** This block defines `MoeWNA16Config.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `MoeWNA16Config.from_config()`，用于从序列化配置数据构造对象。

### Lines 163-167: MoeWNA16Config.override_quantization_method()
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_cfg, user_quant) -> Optional[str]:
        if user_quant == "moe_wna16" and cls.is_moe_wna16_compatible(hf_quant_cfg):
            return cls.get_name()
        return None
```
**EN:** This block defines `MoeWNA16Config.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `MoeWNA16Config.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 169-192: MoeWNA16Config.is_moe_wna16_compatible()
```python
    @classmethod
    def is_moe_wna16_compatible(cls, quant_config: Dict[str, Any]):
        # Extract data from quant config.
        quant_method = quant_config.get("quant_method", "").lower()
        num_bits = quant_config.get("bits")
        desc_act = quant_config.get("desc_act")

        capability_tuple = get_device_capability()
        device_capability = (
            -1
            if all(capability is None for capability in capability_tuple)
            else capability_tuple[0] * 10 + capability_tuple[1]
        )
        # Avoid circular import
        awq_min_capability = AWQConfig.get_min_capability()

        gptq_compatible = quant_method == "gptq" and not desc_act and num_bits in [4, 8]
        awq_compatible = (
            quant_method == "awq"
            and num_bits == 4
            and device_capability >= awq_min_capability
        )

        return gptq_compatible or awq_compatible
```
**EN:** This block defines `MoeWNA16Config.is_moe_wna16_compatible()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `MoeWNA16Config.is_moe_wna16_compatible()`，用于检查供上层控制流使用的条件。

### Lines 194-224: MoeWNA16Config.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        # avoid circular import
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE

        if is_layer_skipped_quant(prefix, self.modules_to_not_convert):
            if isinstance(layer, FusedMoE):
                return UnquantizedFusedMoEMethod()
            return UnquantizedLinearMethod()
        elif isinstance(layer, LinearBase):

            if self.linear_quant_method == "gptq":
                if self.use_marlin:
                    return GPTQMarlinConfig.from_config(
                        self.full_config
                    ).get_quant_method(layer, prefix)
                else:
                    return GPTQConfig.from_config(self.full_config).get_quant_method(
                        layer, prefix
                    )
            elif self.linear_quant_method == "awq":
                return AWQConfig.from_config(self.full_config).get_quant_method(
                    layer, prefix
                )
            else:
                raise ValueError("moe_wna16 only support gptq and awq.")
        elif isinstance(layer, FusedMoE):
            return MoeWNA16Method(self)
        return None
```
**EN:** This block defines `MoeWNA16Config.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MoeWNA16Config.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 227-228: is_layer_skipped_quant()
```python
def is_layer_skipped_quant(prefix: str, modules_to_not_convert: List[str]):
    return any(module_name in prefix for module_name in modules_to_not_convert)
```
**EN:** This block defines `is_layer_skipped_quant()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `is_layer_skipped_quant()`，用于检查供上层控制流使用的条件。

### Lines 231-237: class MoeWNA16Method: definition
```python
class MoeWNA16Method(FusedMoEMethodBase):
    """Linear method for MOE WNA16 (W8A16/W4A16) quantization.

    Args:
        quant_config: The MOE WNA16 (W8A16/W4A16) quantization config.
    """
```
**EN:** This block declares `MoeWNA16Method`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, create_moe_runner, get_triton_quant_info.
**CN:** 该代码块声明 `MoeWNA16Method`，它是量化栈中的运行方法类，组织了 __init__, create_weights, create_moe_runner, get_triton_quant_info 等行为。

### Lines 238-239: MoeWNA16Method.__init__()
```python
    def __init__(self, quant_config: MoeWNA16Config):
        self.quant_config = quant_config
```
**EN:** This block defines `MoeWNA16Method.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `MoeWNA16Method.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 241-285: MoeWNA16Method.create_weights() (part 1/3)
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
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        layer.quant_config = self.quant_config
        bit8_pack_factor = self.quant_config.bit8_pack_factor
        group_size = self.quant_config.group_size
        group_size_div_factor = 1

        # make intermediate_size and hidden_size diviable by group_size
        # we reduce the group size to ensure that
        # and we would repeat the loaded_weight later
        while intermediate_size_per_partition % group_size or hidden_size % group_size:
            group_size = group_size // 2
            group_size_div_factor *= 2
            assert group_size >= 32
        layer.group_size = group_size
        layer.group_size_div_factor = group_size_div_factor

        strategy = FusedMoeWeightScaleSupported.GROUP.value
        extra_weight_attrs.update({"quant_method": strategy, "is_transposed": False})

        assert "weight_loader" in extra_weight_attrs
        weight_loader = extra_weight_attrs["weight_loader"]
        wrapped_weight_loader = MoeWNA16Method.get_weight_loader(layer, weight_loader)
        extra_weight_attrs["weight_loader"] = wrapped_weight_loader

        # Fused gate_up_proj (column parallel)
        w13_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // bit8_pack_factor,
                dtype=torch.uint8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qweight", w13_qweight)
```
**EN:** This segment of `MoeWNA16Method.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `MoeWNA16Method.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 286-330: MoeWNA16Method.create_weights() (part 2/3)
```python
        set_weight_attrs(w13_qweight, extra_weight_attrs)

        # down_proj (row parallel)
        w2_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // bit8_pack_factor,
                dtype=torch.uint8,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qweight", w2_qweight)
        set_weight_attrs(w2_qweight, extra_weight_attrs)

        w13_scales = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                2 * intermediate_size_per_partition,
                hidden_size // group_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_scales", w13_scales)
        set_weight_attrs(w13_scales, extra_weight_attrs)

        w2_scales = torch.nn.Parameter(
            torch.zeros(
                num_experts,
                hidden_size,
                intermediate_size_per_partition // group_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_scales", w2_scales)
        set_weight_attrs(w2_scales, extra_weight_attrs)

        if self.quant_config.has_zp:
            w13_qzeros = torch.nn.Parameter(
                torch.zeros(
                    num_experts,
                    2 * intermediate_size_per_partition // bit8_pack_factor,
                    hidden_size // group_size,
```
**EN:** This segment of `MoeWNA16Method.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `MoeWNA16Method.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 331-361: MoeWNA16Method.create_weights() (part 3/3)
```python
                    dtype=torch.uint8,
                ),
                requires_grad=False,
            )
            layer.register_parameter("w13_qzeros", w13_qzeros)
            set_weight_attrs(w13_qzeros, extra_weight_attrs)

            w2_qzeros = torch.nn.Parameter(
                torch.zeros(
                    num_experts,
                    hidden_size // bit8_pack_factor,
                    intermediate_size_per_partition // group_size,
                    dtype=torch.uint8,
                ),
                requires_grad=False,
            )
            layer.register_parameter("w2_qzeros", w2_qzeros)
            set_weight_attrs(w2_qzeros, extra_weight_attrs)

        if self.quant_config.linear_quant_method == "gptq":
            # some param are unused, but we need to init them in order to
            # load weights
            invalid_param_keys = ["w13_g_idx", "w2_g_idx"]
            if not self.quant_config.has_zp:
                invalid_param_keys += ["w13_qzeros", "w2_qzeros"]
            for key in invalid_param_keys:
                param = torch.nn.Parameter(
                    torch.empty((0,), dtype=torch.int32), requires_grad=False
                )
                layer.register_parameter(key, param)
                set_weight_attrs(param, extra_weight_attrs)
```
**EN:** This segment of `MoeWNA16Method.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `MoeWNA16Method.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 363-367: MoeWNA16Method.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.TRITON, moe_runner_config)
```
**EN:** This block defines `MoeWNA16Method.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `MoeWNA16Method.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 369-382: MoeWNA16Method.get_triton_quant_info()
```python
    def get_triton_quant_info(self, layer: torch.nn.Module) -> TritonMoeQuantInfo:
        weight_bits = self.quant_config.weight_bits
        has_zp = self.quant_config.has_zp
        return TritonMoeQuantInfo(
            w13_weight=layer.w13_qweight,
            w2_weight=layer.w2_qweight,
            use_int4_w4a16=weight_bits == 4,
            use_int8_w8a16=weight_bits == 8,
            w13_scale=layer.w13_scales,
            w2_scale=layer.w2_scales,
            w13_zp=layer.w13_qzeros if has_zp else None,
            w2_zp=layer.w2_qzeros if has_zp else None,
            block_shape=[0, layer.group_size],
        )
```
**EN:** This block defines `MoeWNA16Method.get_triton_quant_info()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MoeWNA16Method.get_triton_quant_info()`，用于为调用方获取或计算派生值。

### Lines 384-394: MoeWNA16Method.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        assert (
            self.moe_runner_config.activation == "silu"
        ), "Only SiLU activation is supported."

        quant_info = self.get_triton_quant_info(layer)
        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `MoeWNA16Method.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `MoeWNA16Method.apply()`，用于将量化计算应用到运行时输入上。

### Lines 396-440: MoeWNA16Method.get_weight_loader() (part 1/3)
```python
    @staticmethod
    def get_weight_loader(layer, weight_loader):

        def convert_awq_tensor(tensor, tensor_type):
            # convert awq qweight/qzeros to a standard format (assume int4)
            # qweight: (k, n // pack_factor_bit32) -> (n, k // pack_factor_bit8)
            # qzeros: (k // group_size, n // pack_factor_bit32) ->
            #         (n // pack_factor_bit8, k // group_size)
            # pack_factor_bit32 = 32 // weight_bits
            # pack_factor_bit8 = 8 // weight_bits

            # 0. suppose origin shape (a, b), dtype int32
            # 1. convert to uint8, shape (a, b) -> (a, 4 * b)
            size0 = tensor.size(0)
            tensor = tensor.view(torch.uint8)

            # 2. unpack to uint4 (only when weight_bits == 4)
            #    shape (a, 4 * b) -> (a, 4 * b, 2)
            shifter = torch.tensor([0, 4], dtype=torch.uint8, device=tensor.device)
            tensor = (tensor[:, :, None] >> shifter) & 0xF

            # 3. change order, see
            # https://github.com/casper-hansen/AutoAWQ/blob/v0.2.8/awq/utils/quant_utils.py
            # shape -> (a, 4 * b * pack_factor_bit8)
            reverse_awq_pack_order = [0, 4, 1, 5, 2, 6, 3, 7]
            tensor = tensor.view(-1, 8)[:, reverse_awq_pack_order]
            tensor = tensor.view(size0, -1)

            # 4. transpose, shape -> (4 * b * pack_factor_bit8, a)
            tensor = tensor.T.contiguous()

            # 5. repack (only when weight_bits == 4)
            # qweight shape -> (4 * b * pack_factor_bit8, a // pack_factor_bit8)
            # qzeros shape -> (4 * b, a)

            if tensor_type == "qweight":
                tensor = tensor[:, 1::2] * 16 + tensor[:, ::2]
            elif tensor_type == "qzeros":
                tensor = tensor[1::2, :] * 16 + tensor[::2, :]
            return tensor

        def convert_gptq_int4_qzeros(tensor):
            tensor = tensor.view(torch.uint8)
            shifter = torch.tensor([0, 4], dtype=torch.uint8, device=tensor.device)
            tensor = (tensor[:, :, None] >> shifter) & 0xF
```
**EN:** This segment of `MoeWNA16Method.get_weight_loader()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `MoeWNA16Method.get_weight_loader()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 441-485: MoeWNA16Method.get_weight_loader() (part 2/3)
```python
            tensor = tensor + 1
            tensor = tensor[:, :, 0] + tensor[:, :, 1] * 16
            return tensor

        def moe_wna16_weight_loader(
            param: torch.nn.Parameter,
            loaded_weight: torch.Tensor,
            weight_name: str,
            shard_id: str,
            expert_id: int,
        ):
            if "g_idx" in weight_name:
                return
            if not layer.quant_config.has_zp and "qzeros" in weight_name:
                return

            device = get_tp_group().device
            tp_rank = get_tensor_model_parallel_rank()
            loaded_weight = loaded_weight.to(device)
            shard_size = layer.intermediate_size_per_partition

            # convert gptq and awq weight to a standard format
            if layer.quant_config.linear_quant_method == "awq":
                assert layer.quant_config.weight_bits == 4
                if "weight" in weight_name:
                    loaded_weight = convert_awq_tensor(loaded_weight, "qweight")
                elif "zeros" in weight_name:
                    loaded_weight = convert_awq_tensor(loaded_weight, "qzeros")
                else:
                    loaded_weight = loaded_weight.T
            elif layer.quant_config.linear_quant_method == "gptq":
                assert layer.quant_config.weight_bits in [4, 8]
                if "weight" in weight_name:
                    loaded_weight = loaded_weight.T.contiguous().view(torch.uint8)
                elif "zeros" in weight_name:
                    # add 1 to gptq qzeros to align with awq
                    loaded_weight = loaded_weight.view(torch.uint8)
                    if layer.quant_config.weight_bits == 4:
                        loaded_weight = convert_gptq_int4_qzeros(loaded_weight).T
                    else:
                        loaded_weight = loaded_weight.T + 1
                else:
                    loaded_weight = loaded_weight.T

            # repeat the qzeros/scales to fit new group size
```
**EN:** This segment of `MoeWNA16Method.get_weight_loader()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `MoeWNA16Method.get_weight_loader()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

### Lines 486-510: MoeWNA16Method.get_weight_loader() (part 3/3)
```python
            if (
                layer.group_size_div_factor > 1
                and "qzeros" in weight_name
                or "scales" in weight_name
            ):
                loaded_weight = loaded_weight.repeat_interleave(
                    layer.group_size_div_factor, 1
                )

            if "w13_qzeros" in weight_name:
                tensor = loaded_weight.view(
                    layer.moe_tp_size, -1, loaded_weight.size(1)
                )[tp_rank]
                if shard_id == "w1":
                    param.data[expert_id, : shard_size // 2] = tensor
                else:
                    param.data[expert_id, shard_size // 2 :] = tensor
            elif "w2_qzeros" in weight_name:
                param.data[expert_id] = loaded_weight.view(
                    loaded_weight.size(0), layer.moe_tp_size, -1
                )[:, tp_rank]
            else:
                weight_loader(param, loaded_weight, weight_name, shard_id, expert_id)

        return moe_wna16_weight_loader
```
**EN:** This segment of `MoeWNA16Method.get_weight_loader()` retrieves or computes a derived value for callers. It covers one portion of the full implementation.
**CN:** `MoeWNA16Method.get_weight_loader()` 的这一段代码用于为调用方获取或计算派生值，展示了完整实现中的一部分。

## Key Concepts / 关键概念
- `get_weight_perm()` : A public function that retrieves or computes a derived value for callers. / `get_weight_perm()`：一个公开函数，用于为调用方获取或计算派生值。
- `MoeWNA16Config`: A configuration class that structures file-level quantization behavior. / `MoeWNA16Config` 是一个配置类，用于组织该文件中的量化行为。
- `is_layer_skipped_quant()` : A public function that checks a condition used by higher-level control flow. / `is_layer_skipped_quant()`：一个公开函数，用于检查供上层控制流使用的条件。
- `MoeWNA16Method`: A runtime method class that structures file-level quantization behavior. / `MoeWNA16Method` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `numpy`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.distributed`, `sglang.srt.distributed.parallel_state`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.fused_moe_triton.layer`, `sglang.srt.layers.moe.moe_runner.triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.awq`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.gptq`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.utils`
