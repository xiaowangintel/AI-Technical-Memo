# config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/config.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-23 — imports and setup
```python
from dataclasses import dataclass
from enum import IntEnum
from typing import Union

import torch

from vllm.config import ParallelConfig, SchedulerConfig
from vllm.config.kernel import MoEBackend
from vllm.distributed import get_dp_group, get_pcp_group, get_tensor_model_parallel_rank
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.quantization.utils.ocp_mx_utils import (
    OCP_MX_DTYPES,
    OCP_MX_Scheme,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape
from vllm.platforms import current_platform
from vllm.utils.import_utils import has_triton_kernels
from vllm.utils.math_utils import cdiv

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 36-66 — function `_get_config_dtype_str`
```python
def _get_config_dtype_str(
    dtype: torch.dtype,
    use_fp8_w8a8: bool = False,
    use_fp8_w8a16: bool = False,
    use_int8_w8a16: bool = False,
    use_int4_w4a16: bool = False,
    ocp_mx_scheme: str | None = None,
) -> str | None:
    """
    Return a string used to construct the filename that contains the
    tuning info for a particular quantization scheme.  See
    try_get_optimal_moe_config in fused_moe.py.
    """
    if use_fp8_w8a8:
        return "fp8_w8a8"
    elif use_fp8_w8a16:
        return "fp8_w8a16"
    elif use_int8_w8a16:
        return "int8_w8a16"
    elif use_int4_w4a16:
        return "int4_w4a16"
    elif ocp_mx_scheme is not None:
        # The output of this function is passed to `try_get_optimal_moe_config`,
        # and as we only simulate OCP MX execution in fused_moe for now,
        # we will NOT look for `*,dtype=w_mxfp4_a_mxfp4.json` for now.
        return None
    elif dtype == torch.float:
        # avoiding cases where kernel fails when float32 MoE
        # use fp16/bfloat16 configs
        return "float32"
    return None
```
**EN:** This function defines `_get_config_dtype_str`. Return a string used to construct the filename that contains the tuning info for a particular quantization scheme. The main inputs are `dtype`, `use_fp8_w8a8`, `use_fp8_w8a16`, `use_int8_w8a16`, `use_int4_w4a16`, `ocp_mx_scheme`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_get_config_dtype_str`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `dtype`, `use_fp8_w8a8`, `use_fp8_w8a16`, `use_int8_w8a16`, `use_int4_w4a16`, `ocp_mx_scheme`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 69-97 — function `_quant_flags_to_group_shape`
```python
def _quant_flags_to_group_shape(
    quant_dtype: torch.dtype | str | None,
    per_act_token_quant: bool,
    per_out_ch_quant: bool,
    block_shape: list[int] | None,
) -> tuple[GroupShape | None, GroupShape | None]:
    """
    Convert MoE quantization flags into more generic GroupShapes.
    """
    a_shape: GroupShape | None
    w_shape: GroupShape | None
    if block_shape is not None:
        assert not per_act_token_quant
        assert not per_out_ch_quant
        # TODO(bnell): this is not quite right for activations since first
        # dim should be 1.
        a_shape = GroupShape(row=block_shape[0], col=block_shape[1])
        w_shape = GroupShape(row=block_shape[0], col=block_shape[1])
    else:
        w_shape = None
        a_shape = None if quant_dtype is None else GroupShape.PER_TENSOR

        if per_act_token_quant:
            a_shape = GroupShape.PER_TOKEN

        if per_out_ch_quant:
            w_shape = GroupShape.PER_TOKEN

    return a_shape, w_shape
```
**EN:** This function defines `_quant_flags_to_group_shape`. Convert MoE quantization flags into more generic GroupShapes. The main inputs are `quant_dtype`, `per_act_token_quant`, `per_out_ch_quant`, `block_shape`. Key calls include `GroupShape`. It writes or updates `a_shape`, `w_shape`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_quant_flags_to_group_shape`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `quant_dtype`, `per_act_token_quant`, `per_out_ch_quant`, `block_shape`。 关键调用包括 `GroupShape`。 它会写入或更新 `a_shape`, `w_shape`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 102-126 — class `RoutingMethodType`
```python
class RoutingMethodType(IntEnum):
    # Default: Softmax -> TopK
    Default = (0,)
    # Renormalize: TopK -> Softmax/Sigmoid
    Renormalize = (1,)
    # DeepSeekV3: Sigmoid -> RoutingBiasAdd -> Top2 in group -> Top4 groups
    # -> Top8 experts from the Top4 groups
    DeepSeekV3 = (2,)
    # Llama4: Top1 -> Sigmoid
    Llama4 = (3,)
    # RenormalizeNaive: Softmax/Sigmoid -> TopK -> Renormalize
    RenormalizeNaive = (4,)
    # TopK: TopK (no softmax)
    TopK = (5,)
# ... omitted for brevity ...
    Custom = (101,)
    Simulated = (102,)
```
**EN:** This class defines `RoutingMethodType`. It inherits from `IntEnum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `Default`, `Renormalize`, `DeepSeekV3`, `Llama4`, `RenormalizeNaive`, `TopK`.
**CN:** 该类定义了 `RoutingMethodType`。 它继承自 `IntEnum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `Default`, `Renormalize`, `DeepSeekV3`, `Llama4`, `RenormalizeNaive`, `TopK`。

### Lines 129-166 — function `get_routing_method_type`
```python
def get_routing_method_type(
    scoring_func: str,
    top_k: int,
    renormalize: bool,
    num_expert_group: int | None,
    has_e_score_bias: bool,
) -> RoutingMethodType:
    if scoring_func == "sqrtsoftplus":
        # DeepSeek V4 uses sqrtsoftplus routing with optional routing bias
        # and top-k renormalization.
        if renormalize:
            return RoutingMethodType.DeepseekV4
        else:
            return RoutingMethodType.Unspecified

    if has_e_score_bias:
        if (num_expert_group or 0) > 0 and scoring_func == "sigmoid":
            return RoutingMethodType.DeepSeekV3
        elif scoring_func == "sigmoid":
            return RoutingMethodType.MiniMax2
        else:
            return RoutingMethodType.Unspecified

    if scoring_func == "sigmoid":
        if top_k == 1:
            return RoutingMethodType.Llama4
        elif renormalize:
            return RoutingMethodType.SigmoidRenorm
        else:
            return RoutingMethodType.Unspecified

    if scoring_func == "softmax":
        if renormalize:
            return RoutingMethodType.Renormalize
        else:
            return RoutingMethodType.Default

    return RoutingMethodType.Unspecified
```
**EN:** This function defines `get_routing_method_type`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `scoring_func`, `top_k`, `renormalize`, `num_expert_group`, `has_e_score_bias`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `get_routing_method_type`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `scoring_func`, `top_k`, `renormalize`, `num_expert_group`, `has_e_score_bias`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 169-203 — class `FusedMoEQuantDesc`
```python
@dataclass
class FusedMoEQuantDesc:
    """
    A quantization descriptor for fused MoE ops. This class can describe
    either activations or weights.
    """

    # The quantized type of this parameters.  None means unquantized or
    # already quantized.
    # TODO (bnell): use scalar_type instead of Union.
    dtype: torch.dtype | str | None = None

    # A field that describes the quantization group shape, from quant_utils.py.
    #  * (-1, -1)   for per-tensor quantization
# ... omitted for brevity ...
    # Biases for GPT triton MoE
    bias: torch.Tensor | None = None
```
**EN:** This class defines `FusedMoEQuantDesc`. A quantization descriptor for fused MoE ops. It writes or updates `dtype`, `shape`, `scale`, `alpha_or_gscale`, `zp`, `bias`.
**CN:** 该类定义了 `FusedMoEQuantDesc`。 该类的文档字符串说明了它在当前模块中的职责。 它会写入或更新 `dtype`, `shape`, `scale`, `alpha_or_gscale`, `zp`, `bias`。

### Lines 208-584 — class `FusedMoEQuantConfig`
```python
@dataclass
class FusedMoEQuantConfig:
    """
    The FusedMoEQuantConfig contains all the quantization parameters for
    a single FusedMoEMethodBase operation.  It consists of four
    FusedMoEQuantDescs, one for each activation and set of weights.

    Each FusedMoEMethodBase must implement a get_fused_moe_quant_config
    method to construct a FusedMoEQuantConfig for use with that class.

    FusedMoEQuant configs are only used for modular kernels, fused_experts
    (from fused_moe.py), cutlass_moe_fp[48], rocm_aiter_fused_experts and
    triton_kernel_moe_forward.  Other MoE methods can ignore the
    FusedMoEQuantConfig (for now) and hardcode it to None.
# ... omitted for brevity ...
        assert quant_config.block_shape == block_shape
        return quant_config
```
**EN:** This class defines `FusedMoEQuantConfig`. The FusedMoEQuantConfig contains all the quantization parameters for a single FusedMoEMethodBase operation. Important methods include `quant_dtype`, `weight_quant_dtype`, `is_quantized`, `is_per_act_token`, `per_act_token_quant`, `per_out_ch_quant`. Key calls include `_get_config_dtype_str`, `_quant_flags_to_group_shape`, `FusedMoEQuantConfig`, `isinstance`, `current_platform.fp8_dtype`, `hasattr`. It writes or updates `_a1`, `_a2`, `_w1`, `_w2`, `is_scale_swizzled`, `gemm1_alpha`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `FusedMoEQuantConfig`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `quant_dtype`, `weight_quant_dtype`, `is_quantized`, `is_per_act_token`, `per_act_token_quant`, `per_out_ch_quant`。 关键调用包括 `_get_config_dtype_str`, `_quant_flags_to_group_shape`, `FusedMoEQuantConfig`, `isinstance`, `current_platform.fp8_dtype`, `hasattr`。 它会写入或更新 `_a1`, `_a2`, `_w1`, `_w2`, `is_scale_swizzled`, `gemm1_alpha`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 268-270 — method `FusedMoEQuantConfig.quant_dtype`
```python
    @property
    def quant_dtype(self) -> torch.dtype | str | None:
        return self._a1.dtype
```
**EN:** This method defines `quant_dtype`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `quant_dtype`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 272-274 — method `FusedMoEQuantConfig.weight_quant_dtype`
```python
    @property
    def weight_quant_dtype(self) -> torch.dtype | str | None:
        return self._w1.dtype
```
**EN:** This method defines `weight_quant_dtype`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `weight_quant_dtype`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 276-278 — method `FusedMoEQuantConfig.is_quantized`
```python
    @property
    def is_quantized(self) -> bool:
        return self.quant_dtype is not None
```
**EN:** This method defines `is_quantized`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `is_quantized`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 280-282 — method `FusedMoEQuantConfig.is_per_act_token`
```python
    @property
    def is_per_act_token(self) -> bool:
        return self._a1.shape == GroupShape.PER_TOKEN
```
**EN:** This method defines `is_per_act_token`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `is_per_act_token`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 284-286 — method `FusedMoEQuantConfig.per_act_token_quant`
```python
    @property
    def per_act_token_quant(self) -> bool:
        return self._a1.shape == GroupShape.PER_TOKEN
```
**EN:** This method defines `per_act_token_quant`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `per_act_token_quant`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 288-290 — method `FusedMoEQuantConfig.per_out_ch_quant`
```python
    @property
    def per_out_ch_quant(self) -> bool:
        return self._w1.shape == GroupShape.PER_TOKEN
```
**EN:** This method defines `per_out_ch_quant`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `per_out_ch_quant`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 587-622 — function `fp8_w8a8_moe_quant_config`
```python
def fp8_w8a8_moe_quant_config(
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    a1_scale: torch.Tensor | None = None,
    a2_scale: torch.Tensor | None = None,
    w1_bias: torch.Tensor | None = None,
    w2_bias: torch.Tensor | None = None,
    per_act_token_quant: bool = False,
    per_out_ch_quant: bool = False,
    block_shape: list[int] | None = None,
    a1_gscale: torch.Tensor | None = None,
    a2_gscale: torch.Tensor | None = None,
    g1_alphas: torch.Tensor | None = None,
    g2_alphas: torch.Tensor | None = None,
    gemm1_clamp_limit: float | None = None,
) -> FusedMoEQuantConfig:
    """
    Construct a quant config for fp8 activations and fp8 weights.
    """
    return FusedMoEQuantConfig.make(
        current_platform.fp8_dtype(),
        w1_scale=w1_scale,
        g1_alphas=g1_alphas,
        w2_scale=w2_scale,
        g2_alphas=g2_alphas,
        w1_bias=w1_bias,
        w2_bias=w2_bias,
        a1_scale=a1_scale,
        a1_gscale=a1_gscale,
        a2_scale=a2_scale,
        a2_gscale=a2_gscale,
        per_act_token_quant=per_act_token_quant,
        per_out_ch_quant=per_out_ch_quant,
        block_shape=block_shape,
        gemm1_clamp_limit=gemm1_clamp_limit,
    )
```
**EN:** This function defines `fp8_w8a8_moe_quant_config`. Construct a quant config for fp8 activations and fp8 weights. The main inputs are `w1_scale`, `w2_scale`, `a1_scale`, `a2_scale`, `w1_bias`, `w2_bias`. Key calls include `FusedMoEQuantConfig.make`, `current_platform.fp8_dtype`.
**CN:** 该函数定义 `fp8_w8a8_moe_quant_config`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `w1_scale`, `w2_scale`, `a1_scale`, `a2_scale`, `w1_bias`, `w2_bias`。 关键调用包括 `FusedMoEQuantConfig.make`, `current_platform.fp8_dtype`。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_get_config_dtype_str`, `_quant_flags_to_group_shape`, `RoutingMethodType`, `get_routing_method_type` / [CN] 核心符号：`_get_config_dtype_str`, `_quant_flags_to_group_shape`, `RoutingMethodType`, `get_routing_method_type`

## Dependencies / 依赖关系
- **External**: `dataclasses`, `enum`, `typing`, `torch` / **外部依赖**: `dataclasses`, `enum`, `typing`, `torch`
- **Internal**: `vllm.config`, `vllm.config.kernel`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.import_utils`, `vllm.utils.math_utils` / **内部依赖**: `vllm.config`, `vllm.config.kernel`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.quantization.utils.ocp_mx_utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.utils.import_utils`, `vllm.utils.math_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
