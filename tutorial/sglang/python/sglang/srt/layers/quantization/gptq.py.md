# gptq.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/gptq.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for gptq quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 GPTQ 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-56: module imports and setup
```python
from __future__ import annotations

import logging
from dataclasses import dataclass
from fractions import Fraction
from typing import TYPE_CHECKING, Any, Callable, Dict, List, Optional, Union

import torch

from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
    npu_fused_experts,
)
from sglang.srt.layers.moe import (
    MoeRunner,
    MoeRunnerBackend,
    MoeRunnerConfig,
    get_moe_runner_backend,
)
from sglang.srt.layers.moe.moe_runner.marlin import MarlinMoeQuantInfo
from sglang.srt.layers.parameter import (
    BasevLLMParameter,
    ChannelQuantScaleParameter,
    GroupQuantScaleParameter,
    PackedColumnParameter,
    PackedvLLMParameter,
    RowvLLMParameter,
    permute_param_layout_,
)
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.marlin_utils import (
    apply_gptq_marlin_linear,
    check_marlin_supported,
    check_marlin_supports_shape,
    marlin_is_k_full,
    marlin_make_empty_g_idx,
    marlin_make_workspace,
    marlin_moe_permute_scales,
    marlin_permute_scales,
    marlin_repeat_scales_on_all_ranks,
    marlin_sort_g_idx,
    marlin_zero_points,
    verify_marlin_supported,
)
from sglang.srt.layers.quantization.utils import (
    get_linear_quant_method,
    get_scalar_types,
    replace_parameter,
    unpack_cols,
)
from sglang.srt.utils import is_cuda, is_npu, set_weight_attrs
from sglang.srt.utils.patch_torch import register_fake_if_exists
```
**EN:** This block imports __future__, dataclasses, fractions, logging, sglang.jit_kernel.gptq_marlin_repack, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.linear, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, dataclasses, fractions, logging, sglang.jit_kernel.gptq_marlin_repack, sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu, sglang.srt.layers.linear, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 58-62: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 64-64: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 66-69: conditional logic for _is_cuda
```python
if _is_cuda:
    from sgl_kernel import gptq_gemm, gptq_shuffle

    from sglang.jit_kernel.gptq_marlin_repack import gptq_marlin_repack
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 71-71: initialize _is_npu
```python
_is_npu = is_npu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_npu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_npu。

### Lines 73-74: conditional logic for _is_npu
```python
if _is_npu:
    import torch_npu
```
**EN:** This block applies conditional logic controlled by `_is_npu`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_npu` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 76-76: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 77-77: initialize (ScalarType, scalar_types)
```python
ScalarType, scalar_types = get_scalar_types()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as (ScalarType, scalar_types).
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 (ScalarType, scalar_types)。

### Lines 80-85: check_marlin_format()
```python
def check_marlin_format(hf_quant_cfg: Dict[str, Any]) -> bool:
    # compat: gptqmodel and autogptq (eol) main use checkpoint_format: str
    # compat: autogptq <=0.7.1 is_marlin_format: bool
    return hf_quant_cfg.get("checkpoint_format") == "marlin" or hf_quant_cfg.get(
        "is_marlin_format", False
    )
```
**EN:** This block defines `check_marlin_format()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `check_marlin_format()`，用于实现量化栈中的可复用模块逻辑。

### Lines 88-104: gptq_marlin_moe_repack()
```python
def gptq_marlin_moe_repack(
    b_q_weight: torch.Tensor,
    perm: torch.Tensor,
    size_k: int,
    size_n: int,
    num_bits: int,
) -> torch.Tensor:
    num_experts = b_q_weight.shape[0]
    assert size_k % 16 == 0
    output = torch.empty(
        (num_experts, size_k // 16, size_n * (num_bits // 2)),
        device=b_q_weight.device,
        dtype=b_q_weight.dtype,
    )
    for e in range(num_experts):
        output[e] = gptq_marlin_repack(b_q_weight[e], perm[e], size_k, size_n, num_bits)
    return output
```
**EN:** This block defines `gptq_marlin_moe_repack()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `gptq_marlin_moe_repack()`，用于实现量化栈中的可复用模块逻辑。

### Lines 107-108: class MarlinLinearLayerConfig: definition
```python
@dataclass
class MarlinLinearLayerConfig:
```
**EN:** This block declares `MarlinLinearLayerConfig`, a configuration class for the quantization stack. It organizes behaviors such as class state.
**CN:** 该代码块声明 `MarlinLinearLayerConfig`，它是量化栈中的配置类，组织了 class state 等行为。

### Lines 109-109: MarlinLinearLayerConfig member: annotate full_weight_shape
```python
    full_weight_shape: tuple[int, int]  # [in, out]
```
**EN:** This block declares and initializes the annotated symbol `full_weight_shape`.
**CN:** 该代码块声明并初始化带类型注解的符号 `full_weight_shape`。

### Lines 110-110: MarlinLinearLayerConfig member: annotate partition_weight_shape
```python
    partition_weight_shape: tuple[int, int]
```
**EN:** This block declares and initializes the annotated symbol `partition_weight_shape`.
**CN:** 该代码块声明并初始化带类型注解的符号 `partition_weight_shape`。

### Lines 111-111: MarlinLinearLayerConfig member: annotate weight_type
```python
    weight_type: ScalarType
```
**EN:** This block declares and initializes the annotated symbol `weight_type`.
**CN:** 该代码块声明并初始化带类型注解的符号 `weight_type`。

### Lines 112-112: MarlinLinearLayerConfig member: annotate act_type
```python
    act_type: torch.dtype
```
**EN:** This block declares and initializes the annotated symbol `act_type`.
**CN:** 该代码块声明并初始化带类型注解的符号 `act_type`。

### Lines 113-113: MarlinLinearLayerConfig member: annotate group_size
```python
    group_size: int
```
**EN:** This block declares and initializes the annotated symbol `group_size`.
**CN:** 该代码块声明并初始化带类型注解的符号 `group_size`。

### Lines 114-114: MarlinLinearLayerConfig member: annotate zero_points
```python
    zero_points: bool
```
**EN:** This block declares and initializes the annotated symbol `zero_points`.
**CN:** 该代码块声明并初始化带类型注解的符号 `zero_points`。

### Lines 115-115: MarlinLinearLayerConfig member: annotate has_g_idx
```python
    has_g_idx: bool
```
**EN:** This block declares and initializes the annotated symbol `has_g_idx`.
**CN:** 该代码块声明并初始化带类型注解的符号 `has_g_idx`。

### Lines 118-123: class GPTQConfig: definition
```python
class GPTQConfig(QuantizationConfig):
    """Config class for GPTQ.

    Reference: https://arxiv.org/abs/2210.17323
    """
```
**EN:** This block declares `GPTQConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_scaled_act_names, get_name.
**CN:** 该代码块声明 `GPTQConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_scaled_act_names, get_name 等行为。

### Lines 124-176: GPTQConfig.__init__()
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        desc_act: bool,
        lm_head_quantized: bool,
        dynamic: Dict[str, Dict[str, Union[int, bool]]],
        checkpoint_format: str = "",
        true_sequential: bool = False,
        static_groups: bool = False,
    ) -> None:
        # GPTQModel use `dynamic` config property to allow per module
        # quantization config so each module can be individually optimized.
        # Format is Dict[str, Dict] where key is a regex string that can
        # perform both positive ("+:" prefixed) or negative ("-:" prefixed)
        # matching of a module.
        # Default to positive match, override base quant config mode, if no
        # prefix is used. Value is in dict format of field key and override
        # value.
        # Negative matching will skip quantization init for this module
        # entirely:
        # non-quantized inference. More details and quantization examples can be
        # found at: https://github.com/ModelCloud/GPTQModel
        # Example:
        #  # last 1/2 of the layers 10-21 has 8bit vs 4bit for 0-9
        #  # last 1/4 of the layers 16-21 has 8bit and group_size 64
        # dynamic = {
        #  #`.*\.` matches the layers_node prefix
        #  # positive match layer 10-15
        #  r"+:.*\.(?:1[0-5])\..*": {"bits": 8,},
        #  # positive match layer 16-21
        #  r"+:.*\.(?:1[6-9]|20|21)\..*": {"bits": 8, "group_size": 64,},
        #  r"-:.*\.moe\..*": {}, # negative match (skip) all `moe` layers
        # }
        super().__init__()
        self.dynamic = dynamic

        self.weight_bits = weight_bits
        self.group_size = group_size
        self.desc_act = desc_act
        self.lm_head_quantized = lm_head_quantized
        self.pack_factor = Fraction(32, self.weight_bits)
        # GPTQ v1 and v2 format deals with zero points differently.
        # Currently GPTQModel stores v1 format checkpoints by default,
        # but provides the option to set `format="gptq_v2"` in `QuantizeConfig`.
        self.checkpoint_format = checkpoint_format
        self.true_sequential = true_sequential
        self.static_groups = static_groups
        if self.weight_bits not in [2, 3, 4, 8]:
            raise ValueError(
                "Currently, only 2/3/4/8-bit weight quantization is "
                f"supported for GPTQ, but got {self.weight_bits} bits."
            )
```
**EN:** This block defines `GPTQConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 178-186: GPTQConfig.__repr__()
```python
    def __repr__(self) -> str:
        return (
            f"GPTQConfig(weight_bits={self.weight_bits}, "
            f"group_size={self.group_size}, "
            f"desc_act={self.desc_act}),"
            f"lm_head_quantized={self.lm_head_quantized}), "
            f"dynamic={self.dynamic},"
            f"checkpoint_format={self.checkpoint_format})"
        )
```
**EN:** This block defines `GPTQConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `GPTQConfig.__repr__()`，用于构造可读的调试表示。

### Lines 188-193: GPTQConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        """Returns the activation function names that should be post-scaled.

        For now, this is only used by AWQ.
        """
        raise NotImplementedError
```
**EN:** This block defines `GPTQConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 195-197: GPTQConfig.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "gptq"
```
**EN:** This block defines `GPTQConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 199-201: GPTQConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.half] if not _is_npu else [torch.half, torch.bfloat16]
```
**EN:** This block defines `GPTQConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 203-211: GPTQConfig.get_min_capability()
```python
    @classmethod
    # Need to figure it out
    def get_min_capability(cls) -> int:
        if _is_npu:
            raise NotImplementedError(
                'NPU hardware does not support "get_min_capability" feature.'
            )
        else:
            return 60
```
**EN:** This block defines `GPTQConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 213-215: GPTQConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return ["quantize_config.json"]
```
**EN:** This block defines `GPTQConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 217-242: GPTQConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> GPTQConfig:
        dynamic = cls.get_from_keys_or(config, ["dynamic"], default={})
        dynamic = {} if dynamic is None else dynamic

        weight_bits = cls.get_from_keys(config, ["bits"])
        group_size = cls.get_from_keys(config, ["group_size"])
        desc_act = cls.get_from_keys(config, ["desc_act"])
        lm_head_quantized = cls.get_from_keys_or(config, ["lm_head"], default=False)
        checkpoint_format = cls.get_from_keys_or(
            config, ["checkpoint_format"], default=""
        )
        true_sequential = cls.get_from_keys_or(
            config, ["true_sequential"], default=False
        )
        static_groups = cls.get_from_keys_or(config, ["static_groups"], default=False)
        return cls(
            weight_bits,
            group_size,
            desc_act,
            lm_head_quantized,
            dynamic,
            checkpoint_format,
            true_sequential,
            static_groups,
        )
```
**EN:** This block defines `GPTQConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `GPTQConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 244-263: GPTQConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[LinearMethodBase]:
        # Delay the import to avoid circular dependency
        from sglang.srt.layers.linear import LinearBase
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if _is_npu:
            if isinstance(layer, FusedMoE):
                return GPTQMoEAscendMethod(self)
            if isinstance(layer, LinearBase):
                return GPTQLinearAscendMethod(self)
            return None

        if isinstance(layer, FusedMoE):
            raise TypeError("GPTQ Method does not support MoE, please use gptq_marlin")
        else:
            return get_linear_quant_method(
                self, layer, prefix=prefix, linear_method_cls=GPTQLinearMethod
            )
```
**EN:** This block defines `GPTQConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 266-269: class GPTQMarlinConfig: definition
```python
class GPTQMarlinConfig(QuantizationConfig):
    """Config class for GPTQ Marlin"""

    # (num_bits, is_sym) -> quant_type
```
**EN:** This block declares `GPTQMarlinConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_scaled_act_names, get_name.
**CN:** 该代码块声明 `GPTQMarlinConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_scaled_act_names, get_name 等行为。

### Lines 270-273: GPTQMarlinConfig member: initialize TYPE_MAP
```python
    TYPE_MAP = {
        (4, True): scalar_types.uint4b8,
        (8, True): scalar_types.uint8b128,
    }
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as TYPE_MAP.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 TYPE_MAP。

### Lines 275-331: GPTQMarlinConfig.__init__()
```python
    def __init__(
        self,
        weight_bits: int,
        group_size: int,
        desc_act: bool,
        is_sym: bool,
        lm_head_quantized: bool,
        dynamic: Dict[str, Dict[str, Union[int, bool]]],
        full_config: Dict[str, Any],
    ) -> None:
        super().__init__()
        if desc_act and group_size == -1:
            # In this case, act_order == True is the same as act_order == False
            # (since we have only one group per output channel)
            desc_act = False

        # GPTQModel use `dynamic` config property to allow per module
        # quantization config so each module can be individually optimized.
        # Format is Dict[str, Dict] where key is a regex string that can
        # perform both positive ("+:" prefixed) or negative ("-:" prefixed)
        # matching of a module.
        # Default to positive match, override base quant config mode, if no
        # prefix is used. Value is in dict format of field key and override
        # value.
        # Negative matching will skip quantization init for this module
        # entirely:
        # non-quantized inference. More details and quantization examples can be
        # found at: https://github.com/ModelCloud/GPTQModel
        # Example:
        #  # last 1/2 of the layers 10-21 has 8bit vs 4bit for 0-9
        #  # last 1/4 of the layers 16-21 has 8bit and group_size 64
        # dynamic = {
        #  #`.*\.` matches the layers_node prefix
        #  # positive match layer 10-15
        #  r"+:.*\.(?:1[0-5])\..*": {"bits": 8,},
        #  # positive match layer 16-21
        #  r"+:.*\.(?:1[6-9]|20|21)\..*": {"bits": 8, "group_size": 64,},
        #  r"-:.*\.moe\..*": {}, # negative match (skip) all `moe` layers
        # }
        self.dynamic = dynamic

        self.weight_bits = weight_bits
        self.is_sym = is_sym

        self.pack_factor = 32 // weight_bits  # packed into int32
        self.group_size = group_size
        self.desc_act = desc_act
        self.lm_head_quantized = lm_head_quantized
        self.full_config = full_config

        if (weight_bits, is_sym) not in self.TYPE_MAP:
            raise ValueError(
                "Unsupported quantization config: " f"bits={weight_bits}, sym={is_sym}"
            )

        # (num_bits, is_sym) -> quant_type
        self.quant_type = self.TYPE_MAP[(weight_bits, is_sym)]
```
**EN:** This block defines `GPTQMarlinConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQMarlinConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 333-340: GPTQMarlinConfig.__repr__()
```python
    def __repr__(self) -> str:
        return (
            f"GPTQMarlinConfig(quant_type={self.quant_type}, "
            f"group_size={self.group_size}, "
            f"desc_act={self.desc_act}, "
            f"lm_head_quantized={self.lm_head_quantized}), "
            f"dynamic={self.dynamic}"
        )
```
**EN:** This block defines `GPTQMarlinConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `GPTQMarlinConfig.__repr__()`，用于构造可读的调试表示。

### Lines 342-347: GPTQMarlinConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        """Returns the activation function names that should be post-scaled.

        For now, this is only used by AWQ.
        """
        raise NotImplementedError
```
**EN:** This block defines `GPTQMarlinConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQMarlinConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 349-351: GPTQMarlinConfig.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "gptq_marlin"
```
**EN:** This block defines `GPTQMarlinConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQMarlinConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 353-355: GPTQMarlinConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return [torch.half, torch.bfloat16]
```
**EN:** This block defines `GPTQMarlinConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQMarlinConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 357-359: GPTQMarlinConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 80
```
**EN:** This block defines `GPTQMarlinConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQMarlinConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 361-363: GPTQMarlinConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return ["quantize_config.json"]
```
**EN:** This block defines `GPTQMarlinConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQMarlinConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 365-383: GPTQMarlinConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> GPTQMarlinConfig:
        dynamic = cls.get_from_keys_or(config, ["dynamic"], default={})
        dynamic = {} if dynamic is None else dynamic

        weight_bits = cls.get_from_keys(config, ["bits"])
        group_size = cls.get_from_keys(config, ["group_size"])
        desc_act = cls.get_from_keys(config, ["desc_act"])
        is_sym = cls.get_from_keys(config, ["sym"])
        lm_head_quantized = cls.get_from_keys_or(config, ["lm_head"], default=False)
        return cls(
            weight_bits,
            group_size,
            desc_act,
            is_sym,
            lm_head_quantized,
            dynamic,
            config,
        )
```
**EN:** This block defines `GPTQMarlinConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `GPTQMarlinConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 385-410: GPTQMarlinConfig.override_quantization_method()
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_cfg, user_quant) -> Optional[str]:
        is_marlin_format = check_marlin_format(hf_quant_cfg)

        can_convert = cls.is_gptq_marlin_compatible(hf_quant_cfg)

        is_valid_user_quant = (
            user_quant is None or user_quant == "marlin" or user_quant == "gptq_marlin"
        )

        if not is_marlin_format and can_convert and is_valid_user_quant:
            msg = (
                "The model is convertible to {} during runtime."
                " Using {} kernel.".format(cls.get_name(), cls.get_name())
            )
            logger.info(msg)
            return cls.get_name()

        if not is_marlin_format and can_convert and user_quant == "gptq":
            logger.info(
                "Detected that the model can run with gptq_marlin"
                ", however you specified quantization=gptq explicitly,"
                " so forcing gptq. Use quantization=gptq_marlin for"
                " faster inference"
            )
        return None
```
**EN:** This block defines `GPTQMarlinConfig.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `GPTQMarlinConfig.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 412-420: GPTQMarlinConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        # Delay the import to avoid circular dependency
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE

        if isinstance(layer, FusedMoE):
            return GPTQMarlinMoEMethod(self)
        return get_linear_quant_method(self, layer, prefix, GPTQMarlinLinearMethod)
```
**EN:** This block defines `GPTQMarlinConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GPTQMarlinConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 422-445: GPTQMarlinConfig.is_gptq_marlin_compatible()
```python
    @classmethod
    def is_gptq_marlin_compatible(cls, quant_config: Dict[str, Any]):
        quant_method = quant_config.get("quant_method", "").lower()
        num_bits = quant_config.get("bits")
        group_size = quant_config.get("group_size")
        sym = quant_config.get("sym")
        desc_act = quant_config.get("desc_act")

        if not _is_cuda:
            return False

        if quant_method != "gptq":
            return False

        # Marlin conversion is only valid if required properties are found
        if num_bits is None or group_size is None or sym is None or desc_act is None:
            return False

        if (num_bits, sym) not in cls.TYPE_MAP:
            return False

        return check_marlin_supported(
            quant_type=cls.TYPE_MAP[(num_bits, sym)], group_size=group_size
        )
```
**EN:** This block defines `GPTQMarlinConfig.is_gptq_marlin_compatible()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `GPTQMarlinConfig.is_gptq_marlin_compatible()`，用于检查供上层控制流使用的条件。

### Lines 448-454: class GPTQLinearMethod: definition
```python
class GPTQLinearMethod(LinearMethodBase):
    """Linear method for GPTQ.

    Args:
        quant_config: The GPTQ quantization config.
    """
```
**EN:** This block declares `GPTQLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `GPTQLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 455-458: GPTQLinearMethod.__init__()
```python
    def __init__(self, quant_config: GPTQConfig):
        self.quant_config = quant_config
        # GPTQ v1 and v2 format deals with zero points differently
        self.use_v2_format = quant_config.checkpoint_format == "gptq_v2"
```
**EN:** This block defines `GPTQLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 460-504: GPTQLinearMethod.create_weights() (part 1/3)
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
        del output_size  # Unused.
        weight_loader = extra_weight_attrs.get("weight_loader")
        if input_size_per_partition % self.quant_config.group_size != 0:
            raise ValueError(
                "The input size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
            )
        output_size_per_partition = sum(output_partition_sizes)
        if output_size_per_partition % self.quant_config.pack_factor.numerator != 0:
            raise ValueError(
                "The output size is not aligned with the quantized "
                "weight shape. This can be caused by too large "
                "tensor parallel size."
            )

        if self.quant_config.group_size != -1:
            group_size = self.quant_config.group_size
        else:
            group_size = input_size
        self.use_shuffle = True
        scale_and_zero_size = input_size // group_size
        scale_and_zero_input_dim = None
        if (
            input_size != input_size_per_partition
            and self.quant_config.group_size != -1
        ):
            if self.quant_config.desc_act:
                self.use_shuffle = False
            else:
                # we need to partition qzeros and scales for exllama kernel
                scale_and_zero_size = input_size_per_partition // group_size
                scale_and_zero_input_dim = 0

        qweight = PackedvLLMParameter(
```
**EN:** This segment of `GPTQLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 505-549: GPTQLinearMethod.create_weights() (part 2/3)
```python
            data=torch.empty(
                input_size_per_partition // self.quant_config.pack_factor,
                output_size_per_partition,
                dtype=torch.int32,
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=0,
            packed_factor=self.quant_config.pack_factor,
            weight_loader=weight_loader,
        )

        g_idx = RowvLLMParameter(
            data=torch.tensor(
                [
                    i // self.quant_config.group_size
                    for i in range(input_size_per_partition)
                ],
                dtype=torch.int32,
            ),
            input_dim=0,
            weight_loader=weight_loader,
        )
        qzeros_args = {
            "data": torch.empty(
                scale_and_zero_size,
                output_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            "weight_loader": weight_loader,
        }
        weight_scale_args = {
            "data": torch.empty(
                scale_and_zero_size,
                output_size_per_partition,
                dtype=params_dtype,
            ),
            "weight_loader": weight_loader,
        }
        if scale_and_zero_input_dim is None:
            scales = ChannelQuantScaleParameter(output_dim=1, **weight_scale_args)
            qzeros = PackedColumnParameter(
                output_dim=1,
                packed_dim=1,
                packed_factor=self.quant_config.pack_factor,
```
**EN:** This segment of `GPTQLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 550-568: GPTQLinearMethod.create_weights() (part 3/3)
```python
                **qzeros_args,
            )

        else:
            scales = GroupQuantScaleParameter(
                output_dim=1, input_dim=0, **weight_scale_args
            )
            qzeros = PackedvLLMParameter(
                input_dim=0,
                output_dim=1,
                packed_dim=1,
                packed_factor=self.quant_config.pack_factor,
                **qzeros_args,
            )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("g_idx", g_idx)
        layer.register_parameter("qzeros", qzeros)
        layer.register_parameter("scales", scales)
```
**EN:** This segment of `GPTQLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 570-586: GPTQLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        # for torch.compile
        layer.qzeros = torch.nn.Parameter(layer.qzeros.data, requires_grad=False)
        layer.qweight = torch.nn.Parameter(layer.qweight.data, requires_grad=False)
        layer.g_idx = torch.nn.Parameter(layer.g_idx.data, requires_grad=False)
        layer.scales = torch.nn.Parameter(layer.scales.data, requires_grad=False)

        # exllama needs to shuffle the weight after the weight is loaded
        # here we do the shuffle on first forward pass
        if self.use_shuffle:
            if self.quant_config.desc_act:
                layer.g_idx.data = torch.argsort(layer.g_idx).to(torch.int)
            else:
                layer.g_idx.data = torch.empty(
                    (0,), dtype=torch.int, device=layer.g_idx.device
                )
            gptq_shuffle(layer.qweight, layer.g_idx, self.quant_config.weight_bits)
```
**EN:** This block defines `GPTQLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `GPTQLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 588-608: GPTQLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        out_shape = x.shape[:-1] + (layer.qweight.shape[-1],)
        reshaped_x = x.reshape(-1, x.shape[-1])

        output = gptq_gemm(
            reshaped_x,
            layer.qweight,
            layer.qzeros,
            layer.scales,
            layer.g_idx,
            self.use_shuffle,
            self.quant_config.weight_bits,
        )
        if bias is not None:
            output.add_(bias)
        return output.reshape(out_shape)
```
**EN:** This block defines `GPTQLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GPTQLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 611-612: class GPTQMoEAscendMethod: definition
```python
class GPTQMoEAscendMethod(FusedMoEMethodBase):
```
**EN:** This block declares `GPTQMoEAscendMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, create_moe_runner, process_weights_after_loading.
**CN:** 该代码块声明 `GPTQMoEAscendMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, create_moe_runner, process_weights_after_loading 等行为。

### Lines 613-617: GPTQMoEAscendMethod.__init__()
```python
    def __init__(self, quant_config: GPTQConfig):
        super().__init__()
        self.quant_config = quant_config
        self.use_v2_format = quant_config.checkpoint_format == "gptq_v2"
        self.moe_runner_config: Optional[MoeRunnerConfig] = None
```
**EN:** This block defines `GPTQMoEAscendMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQMoEAscendMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 619-663: GPTQMoEAscendMethod.create_weights() (part 1/3)
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

        pack_factor = self.quant_config.pack_factor

        num_groups_w13 = hidden_size // self.quant_config.group_size
        num_groups_w2 = intermediate_size_per_partition // self.quant_config.group_size

        extra_weight_attrs.update(
            {
                "is_transposed": True,
                "quant_method": FusedMoeWeightScaleSupported.GROUP.value,
            }
        )

        w13_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size // pack_factor,
                2 * intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qweight", w13_qweight)
        set_weight_attrs(w13_qweight, extra_weight_attrs)

        w2_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition // pack_factor,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qweight", w2_qweight)
```
**EN:** This segment of `GPTQMoEAscendMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMoEAscendMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 664-708: GPTQMoEAscendMethod.create_weights() (part 2/3)
```python
        set_weight_attrs(w2_qweight, extra_weight_attrs)

        w13_scales = torch.nn.Parameter(
            torch.empty(
                num_experts,
                num_groups_w13,
                2 * intermediate_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_scales", w13_scales)
        set_weight_attrs(w13_scales, extra_weight_attrs)

        w2_scales = torch.nn.Parameter(
            torch.empty(
                num_experts,
                num_groups_w2,
                hidden_size,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_scales", w2_scales)
        set_weight_attrs(w2_scales, extra_weight_attrs)

        w13_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
                num_groups_w13,
                2 * intermediate_size_per_partition // pack_factor,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qzeros", w13_qzeros)
        set_weight_attrs(w13_qzeros, extra_weight_attrs)

        w2_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
                num_groups_w2,
                hidden_size // pack_factor,
                dtype=torch.int32,
            ),
```
**EN:** This segment of `GPTQMoEAscendMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMoEAscendMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 709-712: GPTQMoEAscendMethod.create_weights() (part 3/3)
```python
            requires_grad=False,
        )
        layer.register_parameter("w2_qzeros", w2_qzeros)
        set_weight_attrs(w2_qzeros, extra_weight_attrs)
```
**EN:** This segment of `GPTQMoEAscendMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMoEAscendMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 714-720: GPTQMoEAscendMethod.create_moe_runner()
```python
    def create_moe_runner(
        self,
        layer: torch.nn.Module,
        moe_runner_config: MoeRunnerConfig,
        **extra_weight_attrs,
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `GPTQMoEAscendMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GPTQMoEAscendMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 722-766: GPTQMoEAscendMethod.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w13_qzeros_2d = layer.w13_qzeros.data.contiguous().reshape(
            -1, layer.w13_qzeros.shape[-1]
        )
        layer.w13_qzeros = torch.nn.Parameter(
            unpack_from_int32(
                w13_qzeros_2d,
                self.quant_config.weight_bits,
                packed_dim=1,
            )
            .reshape(layer.w13_qzeros.shape[0], layer.w13_qzeros.shape[1], -1)
            .to(layer.w13_scales.dtype),
            requires_grad=False,
        )
        if not self.use_v2_format:
            layer.w13_qzeros += 1

        w2_qzeros_2d = layer.w2_qzeros.data.contiguous().reshape(
            -1, layer.w2_qzeros.shape[-1]
        )
        layer.w2_qzeros = torch.nn.Parameter(
            unpack_from_int32(
                w2_qzeros_2d,
                self.quant_config.weight_bits,
                packed_dim=1,
            )
            .reshape(layer.w2_qzeros.shape[0], layer.w2_qzeros.shape[1], -1)
            .to(layer.w2_scales.dtype),
            requires_grad=False,
        )
        if not self.use_v2_format:
            layer.w2_qzeros += 1

        w13_qweight_2d = (
            layer.w13_qweight.data.transpose(-1, -2)
            .contiguous()
            .reshape(-1, layer.w13_qweight.shape[-2])
        )
        w13_qweight_tmp = unpack_from_int32(
            w13_qweight_2d, self.quant_config.weight_bits, packed_dim=1
        )

        if self.quant_config.weight_bits == 4:
            group_size = self.quant_config.group_size
            scale_expanded = layer.w13_scales.data.repeat_interleave(group_size, dim=1)
```
**EN:** This segment of `GPTQMoEAscendMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMoEAscendMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 767-811: GPTQMoEAscendMethod.process_weights_after_loading() (part 2/3)
```python

            neg_mask = scale_expanded < 0

            if neg_mask.any():
                neg_mask = neg_mask.transpose(-1, -2)
                neg_mask = neg_mask.contiguous().reshape(w13_qweight_tmp.shape)
                w13_qweight_tmp[neg_mask] = -w13_qweight_tmp[neg_mask]

                if w13_qweight_tmp.max() > 7:
                    w13_qweight_tmp.clamp_(max=7)

                layer.w13_scales.data.abs_()

            layer.w13_qweight = torch.nn.Parameter(
                torch_npu.npu_convert_weight_to_int4pack(
                    w13_qweight_tmp.reshape(
                        layer.w13_qweight.shape[0], layer.w13_qweight.shape[2], -1
                    )
                    .transpose(-1, -2)
                    .contiguous()
                    .reshape(-1, layer.w13_qweight.shape[2])
                    .to(torch.int32)
                )
                .reshape(layer.w13_qweight.shape[0], layer.w13_qweight.shape[1] * 8, -1)
                .contiguous(),
                requires_grad=False,
            )
        # use int8 to store weight by default
        else:
            layer.w13_qweight = torch.nn.Parameter(
                w13_qweight_tmp.reshape(
                    layer.w13_qweight.shape[0], layer.w13_qweight.shape[2], -1
                )
                .transpose(-1, -2)
                .contiguous(),
                requires_grad=False,
            )

        w2_qweight_2d = (
            layer.w2_qweight.data.transpose(-1, -2)
            .contiguous()
            .reshape(-1, layer.w2_qweight.shape[-2])
        )
        w2_qweight_tmp = unpack_from_int32(
            w2_qweight_2d, self.quant_config.weight_bits, packed_dim=1
```
**EN:** This segment of `GPTQMoEAscendMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMoEAscendMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 812-853: GPTQMoEAscendMethod.process_weights_after_loading() (part 3/3)
```python
        )

        if self.quant_config.weight_bits == 4:
            group_size = self.quant_config.group_size
            scale_expanded = layer.w2_scales.data.repeat_interleave(group_size, dim=1)

            neg_mask = scale_expanded < 0

            if neg_mask.any():
                neg_mask = neg_mask.transpose(-1, -2)
                neg_mask = neg_mask.contiguous().reshape(w2_qweight_tmp.shape)
                w2_qweight_tmp[neg_mask] = -w2_qweight_tmp[neg_mask]

                if w2_qweight_tmp.max() > 7:
                    w2_qweight_tmp.clamp_(max=7)

                layer.w2_scales.data.abs_()

            layer.w2_qweight = torch.nn.Parameter(
                torch_npu.npu_convert_weight_to_int4pack(
                    w2_qweight_tmp.reshape(
                        layer.w2_qweight.shape[0], layer.w2_qweight.shape[2], -1
                    )
                    .transpose(-1, -2)
                    .contiguous()
                    .reshape(-1, layer.w2_qweight.shape[2])
                    .to(torch.int32)
                )
                .reshape(layer.w2_qweight.shape[0], layer.w2_qweight.shape[1] * 8, -1)
                .contiguous(),
                requires_grad=False,
            )
        # use int8 to store weight by default
        else:
            layer.w2_qweight = torch.nn.Parameter(
                w2_qweight_tmp.reshape(
                    layer.w2_qweight.shape[0], layer.w2_qweight.shape[2], -1
                )
                .transpose(-1, -2)
                .contiguous(),
                requires_grad=False,
            )
```
**EN:** This segment of `GPTQMoEAscendMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMoEAscendMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 855-892: GPTQMoEAscendMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> torch.Tensor:
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        assert (
            self.moe_runner_config is not None
        ), "moe_runner_config is not set. Did you forget to call create_weights/create_moe_runner?"

        assert self.moe_runner_config.activation in ("silu", "swiglu"), (
            f"Only SiLU/Swiglu activation is supported, "
            f"got {self.moe_runner_config.activation!r}."
        )

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output
        topk_weights, topk_ids, _ = topk_output

        topk_ids = topk_ids.to(torch.int32)
        topk_weights = topk_weights.to(x.dtype)

        output = npu_fused_experts(
            hidden_states=x,
            w13=layer.w13_qweight,
            w13_scale=layer.w13_scales,
            w13_offset=layer.w13_qzeros,
            w2=layer.w2_qweight,
            w2_scale=layer.w2_scales,
            w2_offset=layer.w2_qzeros,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            top_k=topk_ids.shape[1],
            use_wna16=True,
        )

        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `GPTQMoEAscendMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GPTQMoEAscendMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 895-901: class GPTQMarlinLinearMethod: definition
```python
class GPTQMarlinLinearMethod(LinearMethodBase):
    """Linear method for GPTQ Marlin.

    Args:
        quant_config: The GPTQ Marlin quantization config.
    """
```
**EN:** This block declares `GPTQMarlinLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `GPTQMarlinLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, apply 等行为。

### Lines 902-902: GPTQMarlinLinearMethod member: annotate _kernel_backends_being_used
```python
    _kernel_backends_being_used: set[str] = set()
```
**EN:** This block declares and initializes the annotated symbol `_kernel_backends_being_used`.
**CN:** 该代码块声明并初始化带类型注解的符号 `_kernel_backends_being_used`。

### Lines 904-911: GPTQMarlinLinearMethod.__init__()
```python
    def __init__(self, quant_config: GPTQMarlinConfig) -> None:
        self.quant_config = quant_config

        # Verify supported on platform.
        verify_marlin_supported(
            quant_type=self.quant_config.quant_type,
            group_size=self.quant_config.group_size,
        )
```
**EN:** This block defines `GPTQMarlinLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQMarlinLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 913-957: GPTQMarlinLinearMethod.create_weights() (part 1/3)
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
        is_row_parallel = input_size != input_size_per_partition
        weight_loader = extra_weight_attrs.get("weight_loader")

        self.kernel_config = MarlinLinearLayerConfig(
            full_weight_shape=(input_size, output_size),
            partition_weight_shape=(
                input_size_per_partition,
                output_size_per_partition,
            ),
            weight_type=self.quant_config.quant_type,
            act_type=params_dtype,
            group_size=self.quant_config.group_size,
            zero_points=False,
            has_g_idx=self.quant_config.desc_act,
        )
        # Normalize group_size
        if self.quant_config.group_size != -1:
            group_size = self.quant_config.group_size
        else:
            group_size = input_size

        # Determine sharding
        if marlin_repeat_scales_on_all_ranks(
            self.quant_config.desc_act, self.quant_config.group_size, is_row_parallel
        ):
            # By setting scale_dim == None, weight_loader will
            # repeat the scales on each GPU in TP>1 case.
            scales_and_zp_input_dim = None
            scales_and_zp_size = input_size // group_size
        else:
            # By setting scale_dim == 0, weight_loader will
            # shard the scales in TP>1 case.
            scales_and_zp_input_dim = 0
            scales_and_zp_size = input_size_per_partition // group_size
```
**EN:** This segment of `GPTQMarlinLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMarlinLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 958-1002: GPTQMarlinLinearMethod.create_weights() (part 2/3)
```python

        # Quantized weights
        qweight = PackedvLLMParameter(
            data=torch.empty(
                input_size_per_partition // self.quant_config.pack_factor,
                output_size_per_partition,
                dtype=torch.int32,
            ),
            input_dim=0,
            output_dim=1,
            packed_dim=0,
            packed_factor=self.quant_config.pack_factor,
            weight_loader=weight_loader,
        )

        # Activation order
        g_idx = RowvLLMParameter(
            data=torch.empty(
                input_size_per_partition,
                dtype=torch.int32,
            ),
            input_dim=0,
            weight_loader=weight_loader,
        )

        qzeros_args = {
            "data": torch.empty(
                scales_and_zp_size,
                output_size_per_partition // self.quant_config.pack_factor,
                dtype=torch.int32,
            ),
            "weight_loader": weight_loader,
        }
        weight_scale_args = {
            "data": torch.empty(
                scales_and_zp_size,
                output_size_per_partition,
                dtype=params_dtype,
            ),
            "weight_loader": weight_loader,
        }

        if scales_and_zp_input_dim is None:
            scales = ChannelQuantScaleParameter(output_dim=1, **weight_scale_args)
            qzeros = PackedColumnParameter(
```
**EN:** This segment of `GPTQMarlinLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMarlinLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 1003-1024: GPTQMarlinLinearMethod.create_weights() (part 3/3)
```python
                output_dim=1,
                packed_dim=1,
                packed_factor=self.quant_config.pack_factor,
                **qzeros_args,
            )

        else:
            scales = GroupQuantScaleParameter(
                output_dim=1, input_dim=0, **weight_scale_args
            )
            qzeros = PackedvLLMParameter(
                input_dim=0,
                output_dim=1,
                packed_dim=1,
                packed_factor=self.quant_config.pack_factor,
                **qzeros_args,
            )

        layer.register_parameter("qweight", qweight)
        layer.register_parameter("g_idx", g_idx)
        layer.register_parameter("scales", scales)
        layer.register_parameter("qzeros", qzeros)
```
**EN:** This segment of `GPTQMarlinLinearMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMarlinLinearMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 1026-1070: GPTQMarlinLinearMethod.process_weights_after_loading() (part 1/3)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        device = getattr(layer, "qweight").device
        c = self.kernel_config

        check_marlin_supports_shape(
            c.partition_weight_shape[1],  # out_features
            c.partition_weight_shape[0],  # in_features
            c.full_weight_shape[0],  # in_features
            c.group_size,
        )

        row_parallel = c.partition_weight_shape[0] != c.full_weight_shape[0]
        self.is_k_full = marlin_is_k_full(c.has_g_idx, row_parallel)

        # Allocate marlin workspace.
        self.workspace = marlin_make_workspace(device)

        # Default names since marlin requires empty parameters for these,
        # TODO: remove this requirement from marlin (allow optional tensors)
        self.w_q_name = "qweight"
        self.w_s_name = "scales"
        self.w_zp_name = "qzeros"
        self.w_gidx_name = "g_idx"

        def _transform_param(
            layer: torch.nn.Module, name: Optional[str], fn: Callable
        ) -> None:
            if name is not None and getattr(layer, name, None) is not None:

                old_param = getattr(layer, name)
                new_param = fn(old_param)
                # replace the parameter with torch.nn.Parameter for TorchDynamo
                # compatibility
                replace_parameter(
                    layer, name, torch.nn.Parameter(new_param.data, requires_grad=False)
                )

        def transform_w_q(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1, packed_dim=0)
            x.data = gptq_marlin_repack(
                x.data.contiguous(),
                perm=layer.g_idx_sort_indices,
                size_k=c.partition_weight_shape[0],
                size_n=c.partition_weight_shape[1],
```
**EN:** This segment of `GPTQMarlinLinearMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMarlinLinearMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 1071-1115: GPTQMarlinLinearMethod.process_weights_after_loading() (part 2/3)
```python
                num_bits=c.weight_type.size_bits,
            )
            return x

        def transform_w_s(x):
            assert isinstance(x, BasevLLMParameter)
            permute_param_layout_(x, input_dim=0, output_dim=1)
            x.data = marlin_permute_scales(
                x.data.contiguous(),
                size_k=c.partition_weight_shape[0],
                size_n=c.partition_weight_shape[1],
                group_size=c.group_size,
            )
            return x

        if c.has_g_idx:
            g_idx, g_idx_sort_indices = marlin_sort_g_idx(
                getattr(layer, self.w_gidx_name)
            )
            _transform_param(layer, self.w_gidx_name, lambda _: g_idx)
            layer.g_idx_sort_indices = g_idx_sort_indices
        else:
            setattr(layer, self.w_gidx_name, marlin_make_empty_g_idx(device))
            layer.g_idx_sort_indices = marlin_make_empty_g_idx(device)

        if c.zero_points:
            grouped_k = (
                c.partition_weight_shape[0] // c.group_size if c.group_size != -1 else 1
            )
            _transform_param(
                layer,
                self.w_zp_name,
                lambda x: marlin_zero_points(
                    unpack_cols(
                        x.t(),
                        c.weight_type.size_bits,
                        grouped_k,
                        c.partition_weight_shape[1],
                    ),
                    size_k=grouped_k,
                    size_n=c.partition_weight_shape[1],
                    num_bits=c.weight_type.size_bits,
                ),
            )
        else:
```
**EN:** This segment of `GPTQMarlinLinearMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMarlinLinearMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 1116-1118: GPTQMarlinLinearMethod.process_weights_after_loading() (part 3/3)
```python
            setattr(layer, self.w_zp_name, marlin_make_empty_g_idx(device))
        _transform_param(layer, self.w_q_name, transform_w_q)
        _transform_param(layer, self.w_s_name, transform_w_s)
```
**EN:** This segment of `GPTQMarlinLinearMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMarlinLinearMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 1120-1160: GPTQMarlinLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        c = self.kernel_config

        def _get_weight_params(
            layer: torch.nn.Module,
        ) -> tuple[
            torch.Tensor,  # w_q
            torch.Tensor,  # w_s
            Optional[torch.Tensor],  # w_zp,
            Optional[torch.Tensor],  # w_gidx
        ]:
            return (
                getattr(layer, self.w_q_name),
                getattr(layer, self.w_s_name),
                getattr(layer, self.w_zp_name or "", None),
                getattr(layer, self.w_gidx_name or "", None),
            )

        w_q, w_s, w_zp, w_gidx = _get_weight_params(layer)

        # `process_weights_after_loading` will ensure w_zp and w_gidx are not
        #  None for marlin
        return apply_gptq_marlin_linear(
            input=x,
            weight=w_q,
            weight_scale=w_s,
            weight_zp=w_zp,  # type: ignore
            g_idx=w_gidx,  # type: ignore
            g_idx_sort_indices=layer.g_idx_sort_indices,
            workspace=self.workspace,
            wtype=c.weight_type,
            input_size_per_partition=c.partition_weight_shape[0],
            output_size_per_partition=c.partition_weight_shape[1],
            is_k_full=self.is_k_full,
            bias=bias,
        )
```
**EN:** This block defines `GPTQMarlinLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GPTQMarlinLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 1163-1204: unpack_from_int32()
```python
def unpack_from_int32(
    weight: torch.Tensor,
    num_bits: int,
    packed_dim: int = 1,
) -> torch.Tensor:
    """
    Unpacks quantized weights from int32 format back to original bits.

    :param weight: The packed int32 tensor containing quantized weights
    :param num_bits: The number of bits used for quantization (<= 8)
    :param packed_dim: Dimension along which weights are packed (0 or 1), defaults to 1
    :return: Unpacked tensor with int8 dtype after applying offset correction
    """
    assert (
        weight.dtype == torch.int32
    ), f"Expecting `weight.dtype` is torch.int32 but got {weight.dtype}."
    assert (
        num_bits <= 8
    ), f"Expecting `num_bits` should not be larger than 8 but got {num_bits}."

    pack_factor = 32 // num_bits
    mask = (1 << num_bits) - 1

    if packed_dim == 1:
        unpacked_weight = torch.zeros(
            (weight.shape[0], weight.shape[1] * pack_factor),
            device=weight.device,
            dtype=torch.int32,
        )
        for i in range(pack_factor):
            unpacked_weight[:, i::pack_factor] = (weight >> (num_bits * i)) & mask
    else:
        unpacked_weight = torch.zeros(
            (weight.shape[0] * pack_factor, weight.shape[1]),
            device=weight.device,
            dtype=torch.int32,
        )
        for i in range(pack_factor):
            unpacked_weight[i::pack_factor, :] = (weight >> (num_bits * i)) & mask
    offset = pow(2, num_bits) // 2
    unpacked_weight = (unpacked_weight - offset).to(torch.int8)
    return unpacked_weight
```
**EN:** This block defines `unpack_from_int32()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `unpack_from_int32()`，用于实现量化栈中的可复用模块逻辑。

### Lines 1207-1209: class GPTQLinearAscendMethod: definition
```python
class GPTQLinearAscendMethod(GPTQLinearMethod):
    """Linear method for GPTQ on Ascend NPU."""
```
**EN:** This block declares `GPTQLinearAscendMethod`, a runtime method class for the quantization stack. It organizes behaviors such as create_weights, process_weights_after_loading, apply.
**CN:** 该代码块声明 `GPTQLinearAscendMethod`，它是量化栈中的运行方法类，组织了 create_weights, process_weights_after_loading, apply 等行为。

### Lines 1210-1235: GPTQLinearAscendMethod.create_weights()
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
            layer,
            input_size_per_partition,
            output_partition_sizes,
            input_size,
            output_size,
            params_dtype,
            **extra_weight_attrs,
        )
        set_weight_attrs(layer.qzeros, {"pack_factor": self.quant_config.pack_factor})
        set_weight_attrs(layer.qweight, {"pack_factor": self.quant_config.pack_factor})

        if self.quant_config.desc_act:
            raise ValueError(
                "Currently, desc_act (True) is not supported by GPTQ quantization on npu."
            )
```
**EN:** This block defines `GPTQLinearAscendMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GPTQLinearAscendMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 1237-1265: GPTQLinearAscendMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:

        layer.qzeros = torch.nn.Parameter(
            unpack_from_int32(
                layer.qzeros.data.contiguous(),
                self.quant_config.weight_bits,
                packed_dim=1,
            ).to(layer.scales.dtype),
            requires_grad=False,
        )
        if not self.use_v2_format:
            layer.qzeros += 1

        qweight_tmp = unpack_from_int32(
            layer.qweight.data.contiguous(), self.quant_config.weight_bits, packed_dim=0
        )
        # use int8 to store weight by default
        if self.quant_config.weight_bits != 4:
            layer.qweight = torch.nn.Parameter(
                qweight_tmp,
                requires_grad=False,
            )
            return

        # for 4bit case we need to pack 4bit weight to int32 to save memory
        layer.qweight = torch.nn.Parameter(
            torch_npu.npu_convert_weight_to_int4pack(qweight_tmp.to(torch.int32)),
            requires_grad=False,
        )
```
**EN:** This block defines `GPTQLinearAscendMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `GPTQLinearAscendMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 1267-1297: GPTQLinearAscendMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        qweight = layer.qweight
        scales = layer.scales
        qzeros = layer.qzeros

        reshaped_x = x.reshape(-1, x.shape[-1])

        if bias is not None and bias.dtype == torch.bfloat16:
            bias = bias.float()

        # 4bit weight is packed to int32(8 x int4)
        if self.quant_config.weight_bits == 4:
            out_shape = x.shape[:-1] + (qweight.shape[-1] * 8,)
        else:
            out_shape = x.shape[:-1] + (qweight.shape[-1],)

        out = torch_npu.npu_weight_quant_batchmatmul(
            reshaped_x,
            qweight,
            antiquant_scale=scales,
            antiquant_offset=qzeros,
            antiquant_group_size=self.quant_config.group_size,
            bias=bias,
        )

        return out.reshape(out_shape)
```
**EN:** This block defines `GPTQLinearAscendMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GPTQLinearAscendMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 1300-1302: class GPTQMarlinMoEMethod: definition
```python
class GPTQMarlinMoEMethod(FusedMoEMethodBase):
    """MoE Marlin method with quantization."""
```
**EN:** This block declares `GPTQMarlinMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `GPTQMarlinMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 1303-1304: GPTQMarlinMoEMethod.__init__()
```python
    def __init__(self, quant_config: GPTQMarlinConfig) -> None:
        self.quant_config = quant_config
```
**EN:** This block defines `GPTQMarlinMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GPTQMarlinMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 1306-1350: GPTQMarlinMoEMethod.create_weights() (part 1/4)
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
        # Delay the import to avoid circular dependency
        from sglang.srt.layers.linear import set_weight_attrs
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoeWeightScaleSupported

        self.is_k_full = (not self.quant_config.desc_act) or layer.moe_tp_size == 1

        if self.quant_config.group_size != -1:
            scales_size13 = hidden_size // self.quant_config.group_size
            if self.quant_config.desc_act:
                w2_scales_size = intermediate_size_per_partition
            else:
                w2_scales_size = intermediate_size_per_partition * layer.moe_tp_size
            scales_size2 = w2_scales_size // self.quant_config.group_size
            strategy = FusedMoeWeightScaleSupported.GROUP.value
        else:
            scales_size13 = 1
            scales_size2 = 1
            strategy = FusedMoeWeightScaleSupported.CHANNEL.value

        extra_weight_attrs.update({"quant_method": strategy, "is_transposed": True})
        # Fused gate_up_proj (column parallel)
        w13_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size // self.quant_config.pack_factor,
                2 * intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qweight", w13_qweight)
        set_weight_attrs(w13_qweight, extra_weight_attrs)
        # down_proj (row parallel)
        w2_qweight = torch.nn.Parameter(
            torch.empty(
                num_experts,
```
**EN:** This segment of `GPTQMarlinMoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMarlinMoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 1351-1395: GPTQMarlinMoEMethod.create_weights() (part 2/4)
```python
                intermediate_size_per_partition // self.quant_config.pack_factor,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qweight", w2_qweight)
        set_weight_attrs(w2_qweight, extra_weight_attrs)
        # up_proj scales
        w13_scales = torch.nn.Parameter(
            torch.empty(
                num_experts,
                scales_size13,
                2 * intermediate_size_per_partition,
                dtype=torch.half,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_scales", w13_scales)
        set_weight_attrs(w13_scales, extra_weight_attrs)
        # down_proj scales
        w2_scales = torch.nn.Parameter(
            torch.empty(num_experts, scales_size2, hidden_size, dtype=torch.half),
            requires_grad=False,
        )
        layer.register_parameter("w2_scales", w2_scales)
        set_weight_attrs(w2_scales, extra_weight_attrs)
        # dont shard the w2 scales when running act order
        set_weight_attrs(w2_scales, {"load_full_w2": self.quant_config.desc_act})
        # up_proj scales
        w13_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
                scales_size13,
                2 * intermediate_size_per_partition // self.quant_config.pack_factor,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_qzeros", w13_qzeros)
        set_weight_attrs(w13_qzeros, extra_weight_attrs)
        # down_proj scales
        w2_qzeros = torch.nn.Parameter(
            torch.empty(
                num_experts,
```
**EN:** This segment of `GPTQMarlinMoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMarlinMoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 1396-1440: GPTQMarlinMoEMethod.create_weights() (part 3/4)
```python
                scales_size2,
                hidden_size // self.quant_config.pack_factor,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_qzeros", w2_qzeros)
        set_weight_attrs(w2_qzeros, extra_weight_attrs)
        # dont shard the w2 scales when running act order
        set_weight_attrs(w2_qzeros, {"load_full_w2": self.quant_config.desc_act})
        w13_g_idx = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_g_idx", w13_g_idx)
        set_weight_attrs(w13_g_idx, extra_weight_attrs)
        w2_g_idx = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_g_idx", w2_g_idx)
        set_weight_attrs(w2_g_idx, extra_weight_attrs)
        w13_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty(
                num_experts,
                hidden_size,
                dtype=torch.int32,
            ),
            requires_grad=False,
        )
        layer.register_parameter("w13_g_idx_sort_indices", w13_g_idx_sort_indices)
        set_weight_attrs(w13_g_idx_sort_indices, extra_weight_attrs)
        w2_g_idx_sort_indices = torch.nn.Parameter(
            torch.empty(
                num_experts,
                intermediate_size_per_partition,
                dtype=torch.int32,
```
**EN:** This segment of `GPTQMarlinMoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMarlinMoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 1441-1445: GPTQMarlinMoEMethod.create_weights() (part 4/4)
```python
            ),
            requires_grad=False,
        )
        layer.register_parameter("w2_g_idx_sort_indices", w2_g_idx_sort_indices)
        set_weight_attrs(w2_g_idx_sort_indices, extra_weight_attrs)
```
**EN:** This segment of `GPTQMarlinMoEMethod.create_weights()` creates parameters, buffers, or helper objects on demand. It covers one portion of the full implementation.
**CN:** `GPTQMarlinMoEMethod.create_weights()` 的这一段代码用于按需创建参数、缓冲区或辅助对象，展示了完整实现中的一部分。

### Lines 1447-1491: GPTQMarlinMoEMethod.process_weights_after_loading() (part 1/2)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:

        # Process act_order
        if self.quant_config.desc_act:
            # Get sorting based on g_idx
            num_experts = layer.w13_g_idx.shape[0]
            w13_g_idx_sort_indices = torch.empty_like(layer.w13_g_idx)
            w2_g_idx_sort_indices = torch.empty_like(layer.w2_g_idx)
            w13_sorted_g_idx = torch.empty_like(layer.w13_g_idx)
            w2_sorted_g_idx = torch.empty_like(layer.w2_g_idx)
            for e in range(num_experts):
                w13_g_idx_sort_indices[e] = torch.argsort(layer.w13_g_idx[e]).to(
                    torch.int32
                )
                w2_g_idx_sort_indices[e] = torch.argsort(layer.w2_g_idx[e]).to(
                    torch.int32
                )
                w13_sorted_g_idx[e] = layer.w13_g_idx[e][w13_g_idx_sort_indices[e]]
                w2_sorted_g_idx[e] = layer.w2_g_idx[e][w2_g_idx_sort_indices[e]]
            replace_parameter(layer, "w13_g_idx", w13_sorted_g_idx)
            replace_parameter(layer, "w2_g_idx", w2_sorted_g_idx)
            replace_parameter(layer, "w13_g_idx_sort_indices", w13_g_idx_sort_indices)
            replace_parameter(layer, "w2_g_idx_sort_indices", w2_g_idx_sort_indices)
        else:
            # Reset g_idx related tensors
            num_experts = layer.w13_g_idx.shape[0]
            device = layer.w13_g_idx.device
            layer.w13_g_idx = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )
            layer.w2_g_idx = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )
            layer.w13_g_idx_sort_indices = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )
            layer.w2_g_idx_sort_indices = torch.nn.Parameter(
                torch.empty((num_experts, 0), dtype=torch.int32, device=device),
                requires_grad=False,
            )
        # Repack weights
        marlin_w13_qweight = gptq_marlin_moe_repack(
```
**EN:** This segment of `GPTQMarlinMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMarlinMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 1492-1526: GPTQMarlinMoEMethod.process_weights_after_loading() (part 2/2)
```python
            layer.w13_qweight,
            layer.w13_g_idx_sort_indices,
            layer.w13_qweight.shape[1] * self.quant_config.pack_factor,
            layer.w13_qweight.shape[2],
            self.quant_config.weight_bits,
        )
        replace_parameter(layer, "w13_qweight", marlin_w13_qweight)
        marlin_w2_qweight = gptq_marlin_moe_repack(
            layer.w2_qweight,
            layer.w2_g_idx_sort_indices,
            layer.w2_qweight.shape[1] * self.quant_config.pack_factor,
            layer.w2_qweight.shape[2],
            self.quant_config.weight_bits,
        )
        replace_parameter(layer, "w2_qweight", marlin_w2_qweight)
        # Repack scales
        marlin_w13_scales = marlin_moe_permute_scales(
            s=layer.w13_scales,
            size_k=layer.intermediate_size_per_partition,
            size_n=layer.w13_scales.shape[2],
            group_size=self.quant_config.group_size,
        )
        replace_parameter(layer, "w13_scales", marlin_w13_scales)
        marlin_w2_scales = marlin_moe_permute_scales(
            s=layer.w2_scales,
            size_k=layer.w2_scales.shape[1]
            * (
                self.quant_config.group_size
                if self.quant_config.group_size != -1
                else self.quant_config.pack_factor
            ),
            size_n=layer.w2_scales.shape[2],
            group_size=self.quant_config.group_size,
        )
        replace_parameter(layer, "w2_scales", marlin_w2_scales)
```
**EN:** This segment of `GPTQMarlinMoEMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GPTQMarlinMoEMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 1528-1533: GPTQMarlinMoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        assert get_moe_runner_backend().is_auto()
        self.moe_runner_config = moe_runner_config
        self.runner = MoeRunner(MoeRunnerBackend.MARLIN, moe_runner_config)
```
**EN:** This block defines `GPTQMarlinMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GPTQMarlinMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 1535-1553: GPTQMarlinMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        quant_info = MarlinMoeQuantInfo(
            w13_qweight=layer.w13_qweight,
            w2_qweight=layer.w2_qweight,
            w13_scales=layer.w13_scales,
            w2_scales=layer.w2_scales,
            w13_g_idx=layer.w13_g_idx,
            w2_g_idx=layer.w2_g_idx,
            w13_g_idx_sort_indices=layer.w13_g_idx_sort_indices,
            w2_g_idx_sort_indices=layer.w2_g_idx_sort_indices,
            weight_bits=self.quant_config.weight_bits,
            is_k_full=self.is_k_full,
        )

        return self.runner.run(dispatch_output, quant_info)
```
**EN:** This block defines `GPTQMarlinMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GPTQMarlinMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 1557-1571: conditional logic for _is_cuda
```python
if _is_cuda:

    @register_fake_if_exists("sgl_kernel::gptq_gemm")
    def _(a, b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, use_shuffle, bit):
        return a.new_empty((a.shape[0], b_q_weight.shape[-1]), dtype=a.dtype)

    @register_fake_if_exists("sgl_kernel::gptq_marlin_repack")
    def _(b_q_weight, perm, size_k, size_n, num_bits):
        return b_q_weight.new_empty(
            (size_k // 16, size_n * (num_bits // 2)), dtype=b_q_weight.dtype
        )

    @register_fake_if_exists("sgl_kernel::gptq_shuffle")
    def _(q_weight, q_perm, bit):
        return
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

## Key Concepts / 关键概念
- `check_marlin_format()` : A public function that implements reusable module logic for the quantization stack. / `check_marlin_format()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `gptq_marlin_moe_repack()` : A public function that implements reusable module logic for the quantization stack. / `gptq_marlin_moe_repack()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `MarlinLinearLayerConfig`: A configuration class that structures file-level quantization behavior. / `MarlinLinearLayerConfig` 是一个配置类，用于组织该文件中的量化行为。
- `GPTQConfig`: A configuration class that structures file-level quantization behavior. / `GPTQConfig` 是一个配置类，用于组织该文件中的量化行为。
- `GPTQMarlinConfig`: A configuration class that structures file-level quantization behavior. / `GPTQMarlinConfig` 是一个配置类，用于组织该文件中的量化行为。
- `GPTQLinearMethod`: A runtime method class that structures file-level quantization behavior. / `GPTQLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GPTQMoEAscendMethod`: A runtime method class that structures file-level quantization behavior. / `GPTQMoEAscendMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GPTQMarlinLinearMethod`: A runtime method class that structures file-level quantization behavior. / `GPTQMarlinLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `unpack_from_int32()` : A public function that implements reusable module logic for the quantization stack. / `unpack_from_int32()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `GPTQLinearAscendMethod`: A runtime method class that structures file-level quantization behavior. / `GPTQLinearAscendMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GPTQMarlinMoEMethod`: A runtime method class that structures file-level quantization behavior. / `GPTQMarlinMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `dataclasses`, `fractions`, `logging`, `sgl_kernel`, `torch`, `torch_npu`, `typing`
- **Internal / 内部**: `sglang.jit_kernel.gptq_marlin_repack`, `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.moe_runner.marlin`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.parameter`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.marlin_utils`, `sglang.srt.layers.quantization.utils`, `sglang.srt.utils`, `sglang.srt.utils.patch_torch`
