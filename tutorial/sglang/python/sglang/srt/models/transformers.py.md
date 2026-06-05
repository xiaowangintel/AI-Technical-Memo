# transformers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/transformers.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Wrapper around `transformers` models. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 19-19: Module docstring
```python
"""Wrapper around `transformers` models."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 21-72: Module imports
```python
import inspect
import logging
import re
from collections.abc import Iterable, Mapping
from contextlib import contextmanager
from typing import List, Literal, Optional, Tuple, Union

import torch
import transformers
from torch import nn
from transformers import AutoModel, PretrainedConfig, PreTrainedModel
from transformers.dynamic_module_utils import get_class_from_dynamic_module
from transformers.modeling_utils import ALL_ATTENTION_FUNCTIONS

from sglang.srt.distributed import (
    divide,
    get_moe_expert_parallel_world_size,
    get_pp_group,
    get_pp_indices,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.layers.layernorm import GemmaRMSNorm, RMSNorm
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.moe.ep_moe.layer import get_moe_impl_class
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.moe.topk import StandardTopKOutput
from sglang.srt.layers.moe.utils import filter_moe_weight_param_global_expert
from sglang.srt.layers.pooler import EmbeddingPoolerOutput, Pooler, PoolingType
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.utils import PPMissingLayer
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 75-95: Function `can_enable_torch_compile`
```python
def can_enable_torch_compile(config: PretrainedConfig) -> bool:
    """Check whether the model config is compatible with torch.compile.

    Dynamic rope scaling triggers data-dependent control flow that prevents
    capturing a single computation graph, so we disable compilation for it.
    """
    text_config = getattr(config, "text_config", config)
    rope_scaling = getattr(text_config, "rope_scaling", None)
    if isinstance(rope_scaling, dict):
        rope_type = rope_scaling.get("rope_type", rope_scaling.get("type", ""))
        if rope_type == "dynamic":
            return False
    rope_params = getattr(text_config, "rope_parameters", None)
    if isinstance(rope_params, dict):
        if isinstance(next(iter(rope_params.values()), None), dict):
            return not any(
                rp.get("rope_type") == "dynamic" for rp in rope_params.values()
            )
        if rope_params.get("rope_type") == "dynamic":
            return False
    return True
```
**EN:** This function implements `can_enable_torch_compile(config: ...)` and Check whether the model config is compatible with torch.compile.
**CN:** 这个函数实现了 `can_enable_torch_compile(config: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 98-98: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 100-100: Top-level annassign
```python
_TRANSFORMERS_MOE_LAYERS: dict[str, "TransformersFusedMoE"] = {}
```
**EN:** Defines or updates _TRANSFORMERS_MOE_LAYERS, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _TRANSFORMERS_MOE_LAYERS，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 103-104: Function `maybe_prefix`
```python
def maybe_prefix(prefix: str, name: str) -> str:
    return name if not prefix else f"{prefix}.{name}"
```
**EN:** This function implements `maybe_prefix(prefix: ..., name: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `maybe_prefix(prefix: ..., name: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 107-108: Function `log_replacement`
```python
def log_replacement(name: str, old_module: nn.Module, new_module: nn.Module):
    logger.debug("%s: %s -> %s", name, old_module, new_module)
```
**EN:** This function implements `log_replacement(name: ..., old_module: ..., new_module: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `log_replacement(name: ..., old_module: ..., new_module: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 111-117: Function `_getattr_first`
```python
def _getattr_first(obj, names, default=None):
    """Return the first existing attribute from *names*, else *default*."""
    for name in names:
        value = getattr(obj, name, None)
        if value is not None:
            return value
    return default
```
**EN:** This function implements `_getattr_first(obj, names, default=...)` and Return the first existing attribute from *names*, else *default*.
**CN:** 这个函数实现了 `_getattr_first(obj, names, default=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 120-140: Function `_resolve_attention_backend_model_cls`
```python
def _resolve_attention_backend_model_cls(config: PretrainedConfig):
    model_cls = getattr(transformers, getattr(config, "architectures", [""])[0], None)
    if model_cls is not None:
        return model_cls

    auto_map = getattr(config, "auto_map", {}) or {}
    for key in ("AutoModel", "AutoModelForCausalLM"):
        if key not in auto_map:
            continue
        try:
            return get_class_from_dynamic_module(
                auto_map[key],
                getattr(config, "_name_or_path", ""),
            )
        except Exception as e:
            logger.warning(
                "Failed to load dynamic module from auto_map[%s]: %s.",
                key,
                e,
            )
    return None
```
**EN:** This function implements `_resolve_attention_backend_model_cls(config: ...)` and implements attention-related tensor preparation or execution.
**CN:** 这个函数实现了 `_resolve_attention_backend_model_cls(config: ...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 143-165: Function `_encoder_accepts_feature_kwarg`
```python
def _encoder_accepts_feature_kwarg(encoder, feature_kwarg: str) -> bool:
    try:
        sig = inspect.signature(encoder)
    except (TypeError, ValueError):
        return False

    if feature_kwarg in sig.parameters:
        return True

    has_var_keyword = any(
        p.kind == inspect.Parameter.VAR_KEYWORD for p in sig.parameters.values()
    )
    if not has_var_keyword:
        return False

    required_positional_params = [
        p
        for p in sig.parameters.values()
        if p.kind
        in (inspect.Parameter.POSITIONAL_ONLY, inspect.Parameter.POSITIONAL_OR_KEYWORD)
        and p.default is inspect.Parameter.empty
    ]
    return len(required_positional_params) == 0
```
**EN:** This function implements `_encoder_accepts_feature_kwarg(encoder, feature_kwarg: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_encoder_accepts_feature_kwarg(encoder, feature_kwarg: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 168-188: Function `_init_on_device_without_buffers`
```python
@contextmanager
def _init_on_device_without_buffers(device: torch.device):
    """Initialize model parameters on *device* while leaving buffers on CPU.
    Adapted from ``accelerate``."""
    old_register_parameter = nn.Module.register_parameter

    def register_empty_parameter(module, name, param):
        old_register_parameter(module, name, param)
        if param is not None:
            param_cls = type(module._parameters[name])
            kwargs = module._parameters[name].__dict__
            kwargs["requires_grad"] = param.requires_grad
            module._parameters[name] = param_cls(
                module._parameters[name].to(device), **kwargs
            )

    try:
        nn.Module.register_parameter = register_empty_parameter
        yield
    finally:
        nn.Module.register_parameter = old_register_parameter
```
**EN:** This function implements `_init_on_device_without_buffers(device: ...)` and Initialize model parameters on *device* while leaving buffers on CPU. Decorators: contextmanager.
**CN:** 这个函数实现了 `_init_on_device_without_buffers(device: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：contextmanager。

### Lines 191-191: Top-level assign
```python
Style = Literal["colwise", "colwise_rep", "rowwise", "rowwise_rep", "replicate"]
```
**EN:** Defines or updates Style, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 Style，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 194-227: Function `replace_linear_class`
```python
def replace_linear_class(
    linear: nn.Linear,
    style: Style = "replicate",
    quant_config: Optional[QuantizationConfig] = None,
    *,
    prefix: str = "",
) -> Union[ColumnParallelLinear, RowParallelLinear, ReplicatedLinear]:
    if not isinstance(style, str):
        raise ValueError(f"Unsupported parallel style type {type(style)}, expected str")

    sglang_linear_cls, linear_kwargs = {
        "colwise": (ColumnParallelLinear, {}),
        "colwise_rep": (ColumnParallelLinear, {"gather_output": True}),
        "rowwise": (RowParallelLinear, {}),
        "rowwise_rep": (RowParallelLinear, {"input_is_parallel": False}),
        "replicate": (ReplicatedLinear, {}),
    }.get(style, (ReplicatedLinear, {}))

    class HFCompatibleLinear(sglang_linear_cls):
        @property
        def parent_cls(self) -> type:
            return sglang_linear_cls

        def forward(self, input: torch.Tensor) -> torch.Tensor:
            return super().forward(input)[0]

    return HFCompatibleLinear(
        input_size=linear.in_features,
        output_size=linear.out_features,
        bias=linear.bias is not None,
        quant_config=quant_config,
        prefix=prefix,
        **linear_kwargs,
    )
```
**EN:** This function implements `replace_linear_class(linear: ..., style: ...=..., quant_config: ...=..., *, prefix: ...=...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `replace_linear_class(linear: ..., style: ...=..., quant_config: ...=..., *, prefix: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 230-247: Function `_normalize_tp_style`
```python
def _normalize_tp_style(style: str) -> Style:
    style = style.lower().replace("-", "_")
    style = {
        "colwiseparallel": "colwise",
        "packed_colwise": "colwise",
        "local_colwise": "colwise",
        "rowwiseparallel": "rowwise",
        "packed_rowwise": "rowwise",
        "local_rowwise": "rowwise",
        "local_packed_rowwise": "rowwise",
        "isolated": "replicate",
        "local": "replicate",
        "replicated_with_grad_allreduce": "replicate",
        "moe_tp_experts": "replicate",
    }.get(style, style)
    if style not in {"colwise", "colwise_rep", "rowwise", "rowwise_rep", "replicate"}:
        raise ValueError(f"Unsupported TP style '{style}' for Transformers backend.")
    return style
```
**EN:** This function implements `_normalize_tp_style(style: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_normalize_tp_style(style: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 250-302: Function `replace_rms_norm_class`
```python
def replace_rms_norm_class(rms_norm: nn.Module, hidden_size: int) -> nn.Module:
    eps = _getattr_first(rms_norm, ("eps", "variance_epsilon"), 1e-6)
    kwargs = {"hidden_size": hidden_size, "eps": eps}
    weight_meta = getattr(rms_norm, "weight", None)
    if weight_meta is not None:
        kwargs["hidden_size"] = weight_meta.size(0)

    try:
        with torch.device("cpu"):
            weight_test = getattr(rms_norm.__class__(1), "weight", None)
    except Exception:
        weight_test = None
    is_gemma = weight_test is not None and torch.all(weight_test == 0)

    if is_gemma:
        base_cls = GemmaRMSNorm
        norm = base_cls(
            **{k: v for k, v in kwargs.items() if k in ("hidden_size", "eps")}
        )
    else:
        kwargs["has_weight"] = getattr(rms_norm, "with_scale", True)
        if weight_meta is not None:
            kwargs["weight_dtype"] = weight_meta.dtype
        else:
            kwargs["has_weight"] = False
        kwargs["cast_x_before_out_mul"] = (
            True  # match HF fp16-weight-multiply semantics
        )
        base_cls = RMSNorm
        norm = base_cls(**kwargs)

    # Wrap to handle 3D inputs from Transformers backbone (batch dim)
    class HFCompatibleRMSNorm(norm.__class__):
        def forward(self, x, *args, **kwargs):
            orig_shape = x.shape
            if x.ndim > 2:
                x = x.reshape(-1, x.shape[-1]).contiguous()
            result = super().forward(x, *args, **kwargs)
            if isinstance(result, tuple):
                return tuple(
# ... truncated for brevity ...
```
**EN:** This function implements `replace_rms_norm_class(rms_norm: ..., hidden_size: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `replace_rms_norm_class(rms_norm: ..., hidden_size: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 305-322: Function `sglang_flash_attention_forward`
```python
def sglang_flash_attention_forward(
    module: torch.nn.Module,
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    attention_mask: torch.Tensor,
    scaling: float = None,
    attention_instances: Optional[Mapping[int, RadixAttention]] = None,
    forward_batch: Optional[ForwardBatch] = None,
    **kwargs,
):
    self_attn: RadixAttention = attention_instances[module.layer_idx]
    if scaling is not None:
        self_attn.scaling = float(scaling)
    hidden = query.shape[-2]
    query, key, value = (x.transpose(1, 2) for x in (query, key, value))
    query, key, value = (x.reshape(hidden, -1) for x in (query, key, value))
    return self_attn.forward(query, key, value, forward_batch=forward_batch), None
```
**EN:** This function implements `sglang_flash_attention_forward(module: ..., query: ..., key: ..., value: ..., attention_mask: ..., scaling: ...=..., ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个函数实现了 `sglang_flash_attention_forward(module: ..., query: ..., key: ..., value: ..., attention_mask: ..., scaling: ...=..., ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 325-325: Top-level assign
```python
ALL_ATTENTION_FUNCTIONS["sglang"] = sglang_flash_attention_forward
```
**EN:** Defines or updates ALL_ATTENTION_FUNCTIONS['sglang'], providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 ALL_ATTENTION_FUNCTIONS['sglang']，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 328-337: Class `TransformersFusedMoE` overview
```python
class TransformersFusedMoE(nn.Module):
    """FusedMoE wrapper for the Transformers modeling backend.

    Wraps SGLang's native MoE implementation and exposes the
    ``(hidden_states, topk_ids, topk_weights)`` signature expected by
    Transformers' ``experts.forward()``.  A registered custom op
    (``torch.ops.sglang.transformers_moe_forward``) is used so that
    ``torch.compile`` can properly graph-break around the MoE kernel.
    """
```
**EN:** Defines `TransformersFusedMoE` as a reusable runtime type derived from nn.Module. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersFusedMoE`，其继承关系为 nn.Module。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 338-372: Method `TransformersFusedMoE.__init__`
```python
    def __init__(
        self,
        *,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        layer_id: int,
        reduce_results: bool,
        quant_config: Optional[QuantizationConfig],
        prefix: str,
        activation: str,
        with_bias: bool,
        expert_mapping: list,
    ) -> None:
        super().__init__()
        num_redundant = get_global_server_args().ep_num_redundant_experts
        experts_cls = get_moe_impl_class(quant_config)
        self.experts = experts_cls(
            num_experts=num_experts + num_redundant,
            top_k=top_k,
            layer_id=layer_id,
            hidden_size=hidden_size,
            intermediate_size=intermediate_size,
            reduce_results=reduce_results,
            quant_config=quant_config,
            activation=activation,
            with_bias=with_bias,
            prefix=prefix,
        )
        self.layer_name = prefix
        self.num_experts = num_experts
        self.top_k = top_k
        self._expert_mapping = expert_mapping
        _TRANSFORMERS_MOE_LAYERS[prefix] = self
```
**EN:** This method implements `__init__(*, num_experts: ..., top_k: ..., hidden_size: ..., intermediate_size: ..., layer_id: ..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, num_experts: ..., top_k: ..., hidden_size: ..., intermediate_size: ..., layer_id: ..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 374-376: Method `TransformersFusedMoE.tp_size`
```python
    @property
    def tp_size(self) -> int:
        return getattr(self.experts, "moe_tp_size", 1)
```
**EN:** This method implements `tp_size()` and implements a focused helper that supports the surrounding runtime flow inside `TransformersFusedMoE` Decorators: property.
**CN:** 这个方法实现了 `tp_size()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersFusedMoE` 内部调用 装饰器：property。

### Lines 378-380: Method `TransformersFusedMoE.ep_size`
```python
    @property
    def ep_size(self) -> int:
        return getattr(self.experts, "moe_ep_size", 1)
```
**EN:** This method implements `ep_size()` and implements a focused helper that supports the surrounding runtime flow inside `TransformersFusedMoE` Decorators: property.
**CN:** 这个方法实现了 `ep_size()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersFusedMoE` 内部调用 装饰器：property。

### Lines 382-387: Method `TransformersFusedMoE.maybe_all_reduce_tensor_model_parallel`
```python
    def maybe_all_reduce_tensor_model_parallel(
        self, output: torch.Tensor
    ) -> torch.Tensor:
        if self.tp_size > 1:
            return tensor_model_parallel_all_reduce(output)
        return output
```
**EN:** This method implements `maybe_all_reduce_tensor_model_parallel(output: ...)` and implements a focused helper that supports the surrounding runtime flow inside `TransformersFusedMoE`.
**CN:** 这个方法实现了 `maybe_all_reduce_tensor_model_parallel(output: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersFusedMoE` 内部调用。

### Lines 389-390: Method `TransformersFusedMoE.get_expert_weights`
```python
    def get_expert_weights(self):
        return getattr(self.experts, "get_expert_weights", lambda: None)()
```
**EN:** This method implements `get_expert_weights()` and handles weight mapping, filtering, or loading for this model component.
**CN:** 这个方法实现了 `get_expert_weights()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑。

### Lines 392-399: Method `TransformersFusedMoE.get_moe_weights`
```python
    def get_moe_weights(self) -> list[torch.Tensor]:
        num_local = getattr(self.experts, "num_local_experts", self.num_experts)
        return [
            x.data
            for name, x in self.experts.named_parameters()
            if name not in ("correction_bias",)
            and filter_moe_weight_param_global_expert(name, x, num_local)
        ]
```
**EN:** This method implements `get_moe_weights()` and handles weight mapping, filtering, or loading for this model component.
**CN:** 这个方法实现了 `get_moe_weights()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑。

### Lines 401-422: Method `TransformersFusedMoE.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        topk_ids = topk_ids.to(torch.int32)
        topk_weights = topk_weights.to(torch.float32)
        if hidden_states.is_cuda:
            return torch.ops.sglang.transformers_moe_forward(
                hidden_states,
                topk_ids,
                topk_weights,
                self.layer_name,
            )
        return _transformers_moe_forward(
            hidden_states,
            topk_ids,
            topk_weights,
            self.layer_name,
        )
```
**EN:** This method implements `forward(hidden_states: ..., topk_ids: ..., topk_weights: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., topk_ids: ..., topk_weights: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 424-469: Method `TransformersFusedMoE.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loaded: set[str] = set()
        param_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            matched = False
            for param_name, weight_name, expert_id, shard_id in self._expert_mapping:
                if weight_name not in name:
                    continue
                mapped_name = name.replace(weight_name, param_name)
                param = param_dict.get(mapped_name)
                if param is None:
                    continue
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                try:
                    weight_loader(
                        param,
                        loaded_weight,
                        name,
                        shard_id=shard_id,
                        expert_id=expert_id,
                    )
                except TypeError:
                    weight_loader(param, loaded_weight)
                loaded.add(name)
                matched = True
                break
            if not matched:
                direct_name = name if name in param_dict else f"experts.{name}"
                if direct_name in param_dict:
                    param = param_dict[direct_name]
                    weight_loader = getattr(
                        param, "weight_loader", default_weight_loader
                    )
                    try:
                        weight_loader(param, loaded_weight)
                    except TypeError:
                        default_weight_loader(param, loaded_weight)
                    loaded.add(name)
                else:
                    logger.warning(
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 472-492: Function `_transformers_moe_forward`
```python
def _transformers_moe_forward(
    hidden_states: torch.Tensor,
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    layer_name: str,
) -> torch.Tensor:
    self = _TRANSFORMERS_MOE_LAYERS[layer_name]
    # Record expert distribution for EPLB
    from sglang.srt.eplb.expert_distribution import (
        get_global_expert_distribution_recorder,
    )

    recorder = get_global_expert_distribution_recorder()
    with recorder.with_current_layer(self.experts.layer_id):
        recorder.on_select_experts(topk_ids=topk_ids)
    topk_output = StandardTopKOutput(
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        router_logits=topk_weights,
    )
    return self.experts(hidden_states.clone(), topk_output)
```
**EN:** This function implements `_transformers_moe_forward(hidden_states: ..., topk_ids: ..., topk_weights: ..., layer_name: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个函数实现了 `_transformers_moe_forward(hidden_states: ..., topk_ids: ..., topk_weights: ..., layer_name: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 495-501: Function `_transformers_moe_forward_fake`
```python
def _transformers_moe_forward_fake(
    hidden_states: torch.Tensor,
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    layer_name: str,
) -> torch.Tensor:
    return torch.empty_like(hidden_states)
```
**EN:** This function implements `_transformers_moe_forward_fake(hidden_states: ..., topk_ids: ..., topk_weights: ..., layer_name: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个函数实现了 `_transformers_moe_forward_fake(hidden_states: ..., topk_ids: ..., topk_weights: ..., layer_name: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 504-509: Top-level expr
```python
direct_register_custom_op(
    op_name="transformers_moe_forward",
    op_func=_transformers_moe_forward,
    mutates_args=["hidden_states"],
    fake_impl=_transformers_moe_forward_fake,
)
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 511-518: Top-level try
```python
try:
    from sglang.srt.compilation.compilation_config import SPLIT_OPS

    _MOE_SPLIT_OP = "sglang.transformers_moe_forward"
    if _MOE_SPLIT_OP not in SPLIT_OPS:
        SPLIT_OPS.append(_MOE_SPLIT_OP)
except ImportError:
    pass
```
**EN:** Wraps top-level setup in a guarded block to handle optional imports, resource management, or fallback behavior safely.
**CN:** 使用受保护的顶层代码块安全处理可选导入、资源管理或回退行为。

### Lines 521-525: Top-level annassign
```python
_BASE_DYNAMIC_ARG_DIMS: dict[str, int] = {
    "input_ids": 0,
    "positions": 0,
    "input_embeds": 0,
}
```
**EN:** Defines or updates _BASE_DYNAMIC_ARG_DIMS, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _BASE_DYNAMIC_ARG_DIMS，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 527-531: Top-level annassign
```python
_MULTIMODAL_DYNAMIC_ARG_DIMS: dict[str, int] = {
    "input_ids": 0,
    "positions": -1,  # last dim to support M-RoPE (Qwen2.5-VL 3×seq layout)
    "input_embeds": 0,
}
```
**EN:** Defines or updates _MULTIMODAL_DYNAMIC_ARG_DIMS, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _MULTIMODAL_DYNAMIC_ARG_DIMS，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 534-534: Class `TransformersBase` overview
```python
class TransformersBase(nn.Module):
```
**EN:** Defines `TransformersBase` as a reusable runtime type derived from nn.Module. The class groups 22 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersBase`，其继承关系为 nn.Module。这个类组织了 22 个方法，用于实现模型相关行为。

### Lines 535-552: Class `TransformersBase` attributes
```python
    torch_compile_dynamic_arg_dims: dict[str, int] = _BASE_DYNAMIC_ARG_DIMS

    hf_to_sglang_mapper = WeightsMapper(
        orig_to_new_prefix={
            "language_model.model.": "model.language_model.",
            "model.transformer.": "model.",
            "model.model.": "model.",
            "model.lm_head.": "lm_head.",
            "model.score.": "classifier.",
            "model.classifier.": "classifier.",
            "transformer.": "model.",
            "model.": "model.",
            "lm_head.": "lm_head.",
            "score.": "classifier.",
            "classifier.": "classifier.",
            "": "model.",
        }
    )
```
**EN:** Defines class-level attributes and metadata that shape how `TransformersBase` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `TransformersBase` 在运行时的行为。

### Lines 554-561: Method `TransformersBase.__init_subclass__`
```python
    def __init_subclass__(cls, *args, **kwargs):
        super().__init_subclass__(*args, **kwargs)
        mapper = WeightsMapper()
        for base in cls.__mro__:
            base_mapper = getattr(base, "hf_to_sglang_mapper", None)
            if base_mapper is not None:
                mapper = mapper | base_mapper
        cls.hf_to_sglang_mapper = mapper
```
**EN:** This method implements `__init_subclass__(*args, **kwargs)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `__init_subclass__(*args, **kwargs)`，其作用是执行后续运行时所需的初始化工作。

### Lines 563-658: Method `TransformersBase.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        logger.info("Using Transformers backend.")

        self.quant_config = quant_config
        self.config = config
        self.text_config = get_hf_text_config(config)
        self.weight_mapper = self.hf_to_sglang_mapper
        self.pp_group = get_pp_group()

        # Weight loading attrs
        self.skip_prefixes: list[str] = []
        self.skip_substrs: list[str] = []
        self.ignore_unexpected_prefixes: list[str] = []
        self.ignore_unexpected_suffixes: list[str] = []
        self.skip_substrs.extend([".attn.bias", ".attn.masked_bias", ".masked_bias"])
        self.ignore_unexpected_prefixes.extend(["classifier.", "score."])

        if self.quant_config is not None:
            quant_method_name = self.quant_config.get_name()
            if "gptq" in quant_method_name:
                self.ignore_unexpected_suffixes.append(".bias")
            if "fp8" in quant_method_name:
                fp8_suffix_map = {".activation_scale": ".input_scale"}
                use_mxfp8 = bool(getattr(self.quant_config, "use_mxfp8", False))
                weight_block_size = getattr(
                    self.quant_config, "weight_block_size", None
                )
                if not use_mxfp8 and weight_block_size is None:
                    fp8_suffix_map[".weight_scale_inv"] = ".weight_scale"
                self.weight_mapper = self.weight_mapper | WeightsMapper(
                    orig_to_new_suffix=fp8_suffix_map
                )

        # Resolve model class for _supports_attention_backend check
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 660-663: Method `TransformersBase._can_torch_compile`
```python
    @property
    def _can_torch_compile(self) -> bool:
        """Whether this model instance is safe to wrap with torch.compile."""
        return self._compile_compatible
```
**EN:** This method implements `_can_torch_compile()` and Whether this model instance is safe to wrap with torch.compile. Decorators: property.
**CN:** 这个方法实现了 `_can_torch_compile()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：property。

### Lines 665-677: Method `TransformersBase._init_parameters`
```python
    def _init_parameters(self, module: nn.Module):
        """Materialize any parameters still on the meta device."""
        for name, param in module.named_parameters(recurse=False):
            if param.device == torch.device("meta"):
                new_param = nn.Parameter(
                    torch.empty_like(
                        param.data,
                        device="cuda",
                    )
                )
                setattr(module, name, new_param)
        for child in module.children():
            self._init_parameters(child)
```
**EN:** This method implements `_init_parameters(module: ...)` and Materialize any parameters still on the meta device.
**CN:** 这个方法实现了 `_init_parameters(module: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 679-680: Method `TransformersBase.log_replacement`
```python
    def log_replacement(self, name: str, old_module: nn.Module, new_module: nn.Module):
        logger.debug("%s: %s -> %s", name, old_module, new_module)
```
**EN:** This method implements `log_replacement(name: ..., old_module: ..., new_module: ...)` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `log_replacement(name: ..., old_module: ..., new_module: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 683-694: Method `TransformersBase._get_model_tp_plan`
```python
    def _get_model_tp_plan(self) -> Mapping[str, str]:
        plan = (
            getattr(self.model, "tp_plan", None)
            or getattr(self.model, "_tp_plan", None)
            or getattr(self.model.config, "base_model_tp_plan", None)
            or getattr(self.text_config, "base_model_tp_plan", None)
        )
        if plan:
            return plan

        plan = self._infer_tp_plan_from_children()
        return plan if plan else {}
```
**EN:** This method implements `_get_model_tp_plan()` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_model_tp_plan()`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 696-698: Class `TransformersBase` attributes
```python
    _LANGUAGE_MODEL_CHILD_NAMES = frozenset(
        {"language_model", "text_model", "model", "lm"}
    )
```
**EN:** Defines class-level attributes and metadata that shape how `TransformersBase` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `TransformersBase` 在运行时的行为。

### Lines 700-741: Method `TransformersBase._infer_tp_plan_from_children`
```python
    def _infer_tp_plan_from_children(self) -> dict[str, str]:
        plan: dict[str, str] = {}
        for child_name, child_module in self.model.named_children():
            child_plan = getattr(child_module, "_tp_plan", None)
            if child_plan:
                plan.update({f"{child_name}.{k}": v for k, v in child_plan.items()})
                continue

            child_config = getattr(child_module, "config", None)
            if child_config is not None:
                child_tp = getattr(child_config, "base_model_tp_plan", None)
                if child_tp:
                    plan.update({f"{child_name}.{k}": v for k, v in child_tp.items()})
                    continue

            if child_name not in self._LANGUAGE_MODEL_CHILD_NAMES:
                continue
            if child_config is None:
                continue
            model_type = getattr(child_config, "model_type", "")
            base_type = (
                model_type.replace("_vl_text", "")
                .replace("_vl", "")
                .replace("_text", "")
            )
            if base_type and base_type != model_type:
                try:
                    from transformers import AutoConfig

                    base_cfg = AutoConfig.for_model(base_type)
                    base_tp = getattr(base_cfg, "base_model_tp_plan", None)
                    if base_tp:
                        plan.update(
                            {f"{child_name}.{k}": v for k, v in base_tp.items()}
                        )
                except Exception as e:
                    logger.debug(
                        "Could not infer TP plan from base model type '%s': %s",
                        base_type,
                        e,
# ... truncated for brevity ...
```
**EN:** This method implements `_infer_tp_plan_from_children()` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `_infer_tp_plan_from_children()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 743-753: Method `TransformersBase._normalize_tp_plan`
```python
    def _normalize_tp_plan(self, tp_plan: Mapping[str, str]) -> dict[str, Style]:
        normalized = {}
        for pattern, style in tp_plan.items():
            if pattern.startswith("^model\\."):
                pattern = "^" + pattern[len("^model\\.") :]
            elif pattern.startswith("model\\."):
                pattern = pattern[len("model\\.") :]
            elif pattern.startswith("model."):
                pattern = pattern[len("model.") :]
            normalized[pattern] = _normalize_tp_style(style)
        return normalized
```
**EN:** This method implements `_normalize_tp_plan(tp_plan: ...)` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `_normalize_tp_plan(tp_plan: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 756-797: Method `TransformersBase.recursive_replace`
```python
    def recursive_replace(self):
        tp_size = get_tensor_model_parallel_world_size()
        tp_plan = self._normalize_tp_plan(self._get_model_tp_plan())

        if not tp_plan and tp_size > 1:
            raise ValueError(
                f"{type(self.model)} does not support tensor parallel yet!"
            )

        # Prefix patterns to match from `self.model`
        prefixed_plan = {maybe_prefix("model", k): v for k, v in tp_plan.items()}

        def _recursive_replace(module: nn.Module, prefix: str):
            for child_name, child_module in module.named_children():
                qual_name = maybe_prefix(prefix, child_name)
                new_module = child_module

                if isinstance(child_module, nn.Linear):
                    pattern = next(
                        (p for p in prefixed_plan if re.match(p, qual_name)),
                        None,
                    )
                    style = prefixed_plan.get(pattern, "replicate")
                    new_module = replace_linear_class(
                        child_module,
                        style,
                        self.quant_config,
                        prefix=qual_name,
                    )
                elif child_module.__class__.__name__.endswith("RMSNorm"):
                    new_module = replace_rms_norm_class(
                        child_module,
                        self.text_config.hidden_size,
                    )
                else:
                    _recursive_replace(child_module, prefix=qual_name)

                if new_module is not child_module:
                    setattr(module, child_name, new_module)
                    log_replacement(qual_name, child_module, new_module)
# ... truncated for brevity ...
```
**EN:** This method implements `recursive_replace()` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `recursive_replace()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 800-807: Method `TransformersBase._get_model_pp_plan`
```python
    def _get_model_pp_plan(self) -> Mapping[str, object]:
        return (
            getattr(self.model, "_pp_plan", None)
            or getattr(self.model, "pp_plan", None)
            or getattr(self.model.config, "base_model_pp_plan", None)
            or getattr(self.text_config, "base_model_pp_plan", None)
            or {}
        )
```
**EN:** This method implements `_get_model_pp_plan()` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_model_pp_plan()`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 809-813: Method `TransformersBase._register_missing_prefix`
```python
    def _register_missing_prefix(self, prefix: str):
        if not prefix.endswith("."):
            prefix += "."
        if prefix not in self.skip_prefixes:
            self.skip_prefixes.append(prefix)
```
**EN:** This method implements `_register_missing_prefix(prefix: ...)` and registers metadata or implementation classes into a global lookup table.
**CN:** 这个方法实现了 `_register_missing_prefix(prefix: ...)`，其作用是把元数据或实现类注册到全局查找表中。

### Lines 815-827: Method `TransformersBase._make_pp_missing_layer`
```python
    @staticmethod
    def _make_pp_missing_layer(original: nn.Module) -> PPMissingLayer:
        """Create a PPMissingLayer that preserves plain attributes from
        *original* so that the HF forward loop can still access per-layer
        metadata (e.g. ``attention_type`` on Qwen2 decoder layers)."""
        replacement = PPMissingLayer()
        for key, value in original.__dict__.items():
            if key.startswith("_"):
                continue
            if isinstance(value, (nn.Module, nn.Parameter, torch.Tensor)):
                continue
            setattr(replacement, key, value)
        return replacement
```
**EN:** This method implements `_make_pp_missing_layer()` and Create a PPMissingLayer that preserves plain attributes from. Decorators: staticmethod.
**CN:** 这个方法实现了 `_make_pp_missing_layer()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 829-833: Method `TransformersBase._get_submodule_or_none`
```python
    def _get_submodule_or_none(self, name: str) -> Optional[nn.Module]:
        try:
            return self.model.get_submodule(name)
        except AttributeError:
            return None
```
**EN:** This method implements `_get_submodule_or_none(name: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_submodule_or_none(name: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 835-842: Method `TransformersBase._set_submodule`
```python
    def _set_submodule(self, name: str, module: nn.Module):
        if "." in name:
            parent_name, child_name = name.rsplit(".", 1)
            parent_module = self.model.get_submodule(parent_name)
        else:
            parent_module = self.model
            child_name = name
        setattr(parent_module, child_name, module)
```
**EN:** This method implements `_set_submodule(name: ..., module: ...)` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `_set_submodule(name: ..., module: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 844-897: Method `TransformersBase.pipeline_parallel`
```python
    def pipeline_parallel(self):
        if self.pp_group.world_size <= 1:
            return

        pp_plan = self._get_model_pp_plan()
        if not pp_plan:
            raise ValueError(
                f"{type(self.model)} does not support pipeline parallel yet!"
            )

        pp_keys = [re.sub(r"^model\.", "", name) for name in pp_plan.keys()]
        module_list_idx = None
        module_list_name = None
        for idx, name in enumerate(pp_keys):
            if isinstance(self._get_submodule_or_none(name), nn.ModuleList):
                if module_list_idx is not None:
                    raise ValueError(
                        "Pipeline parallel with multiple ModuleList blocks is not supported."
                    )
                module_list_idx = idx
                module_list_name = name

        if module_list_idx is None or module_list_name is None:
            raise ValueError(f"Could not find ModuleList in {type(self.model)}.")

        keep_prefix_modules = self.pp_group.is_first_rank or (
            getattr(self.text_config, "tie_word_embeddings", False)
            and self.pp_group.is_last_rank
        )
        for name in pp_keys[:module_list_idx]:
            if keep_prefix_modules:
                continue
            self._set_submodule(name, PPMissingLayer())
            self._register_missing_prefix(maybe_prefix("model", name))

        layers = self.model.get_submodule(module_list_name)
        self.start_layer, self.end_layer = get_pp_indices(
            len(layers),
            self.pp_group.rank_in_group,
            self.pp_group.world_size,
# ... truncated for brevity ...
```
**EN:** This method implements `pipeline_parallel()` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `pipeline_parallel()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 900-950: Method `TransformersBase._create_attention_instances`
```python
    def _create_attention_instances(self, tp_size: int) -> dict[int, RadixAttention]:
        num_heads = self.text_config.num_attention_heads
        num_kv_heads = getattr(self.text_config, "num_key_value_heads", num_heads)
        hidden_size = self.text_config.hidden_size
        head_dim = getattr(self.text_config, "head_dim", hidden_size // num_heads)

        layer_types = getattr(self.text_config, "layer_types", None) or getattr(
            self.config, "layer_types", None
        )
        global_sliding_window = getattr(
            self.text_config, "sliding_window", None
        ) or getattr(self.config, "sliding_window", None)

        # Detect encoder-only models (non-causal attention everywhere)
        is_encoder_only = any(
            not getattr(m, "is_causal", True)
            for m in self.model.modules()
            if hasattr(m, "is_causal")
        )
        if is_encoder_only and self.config != self.text_config:
            is_encoder_only = False
        if is_encoder_only:
            logger.info(
                "Detected encoder-only model (non-causal attention). "
                "Using RadixAttention with is_cross_attention=True."
            )

        instances = {}
        for idx in range(self.start_layer, self.end_layer):
            # Per-layer sliding window (e.g. Gemma2, Cohere)
            per_layer_sliding_window = -1
            if (
                layer_types is not None
                and idx < len(layer_types)
                and layer_types[idx] == "sliding_attention"
                and global_sliding_window is not None
            ):
                per_layer_sliding_window = global_sliding_window

            instances[idx] = RadixAttention(
# ... truncated for brevity ...
```
**EN:** This method implements `_create_attention_instances(tp_size: ...)` and implements attention-related tensor preparation or execution.
**CN:** 这个方法实现了 `_create_attention_instances(tp_size: ...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 953-985: Method `TransformersBase.replace_vocab_embed_class`
```python
    def replace_vocab_embed_class(self, module: nn.Module):
        old_module = self.model.get_input_embeddings()
        if old_module is None or isinstance(old_module, PPMissingLayer):
            return
        embedding_dim = getattr(old_module, "embedding_dim", None)
        if embedding_dim is None:
            embedding_dim = _getattr_first(
                self.text_config,
                ("embedding_size", "hidden_size"),
                None,
            )
        assert embedding_dim is not None
        new_module = VocabParallelEmbedding(
            self.vocab_size,
            embedding_dim,
            org_num_embeddings=self.vocab_size,
            quant_config=None,
        )

        old_embed_scale = getattr(old_module, "embed_scale", None)
        if old_embed_scale is not None:
            base_cls = new_module.__class__

            class ScaledEmbedding(base_cls):
                def forward(self, input_):
                    return base_cls.forward(self, input_) * self.embed_scale

            new_module.__class__ = ScaledEmbedding
            new_module.embed_scale = old_embed_scale
            self.embed_scale = None

        self.log_replacement("input embedding", old_module, new_module)
        self.model.set_input_embeddings(new_module)
```
**EN:** This method implements `replace_vocab_embed_class(module: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `replace_vocab_embed_class(module: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 988-993: Method `TransformersBase._format_position_ids`
```python
    def _format_position_ids(self, positions: torch.Tensor) -> torch.Tensor:
        if positions.ndim == 2 and positions.shape[0] == 3:
            return positions[:, None, ...]
        if positions.ndim == 1:
            return positions[None, ...]
        return positions
```
**EN:** This method implements `_format_position_ids(positions: ...)` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `_format_position_ids(positions: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 995-1029: Method `TransformersBase._run_hf_backbone`
```python
    def _run_hf_backbone(
        self,
        input_ids: Optional[torch.Tensor],
        input_embeds: Optional[torch.Tensor],
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs,
    ) -> torch.Tensor:
        hf_input_ids = None if input_ids is None else input_ids[None, ...]
        hf_input_embeds = None
        if input_embeds is not None:
            hf_input_embeds = input_embeds[None, ...]
            hf_input_ids = None

        # Scale embeddings if needed
        if (
            self.embed_scale is not None
            and hf_input_ids is not None
            and hf_input_embeds is None
        ):
            hf_input_embeds = (
                self.model.get_input_embeddings()(hf_input_ids) * self.embed_scale
            )
            hf_input_ids = None

        return self.model(
            input_ids=hf_input_ids,
            inputs_embeds=hf_input_embeds,
            use_cache=False,
            position_ids=self._format_position_ids(positions),
            return_dict=False,
            forward_batch=forward_batch,
            attention_instances=self.attention_instances,
            **kwargs,
        )[0][0, ...]
```
**EN:** This method implements `_run_hf_backbone(input_ids: ..., input_embeds: ..., positions: ..., forward_batch: ..., **kwargs)` and implements a focused helper that supports the surrounding runtime flow inside `TransformersBase`.
**CN:** 这个方法实现了 `_run_hf_backbone(input_ids: ..., input_embeds: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `TransformersBase` 内部调用。

### Lines 1031-1043: Method `TransformersBase._forward_hidden_states`
```python
    def _forward_hidden_states(
        self,
        input_ids: Optional[torch.Tensor],
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        return self._run_hf_backbone(
            input_ids=input_ids,
            input_embeds=input_embeds,
            positions=positions,
            forward_batch=forward_batch,
        )
```
**EN:** This method implements `_forward_hidden_states(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_hidden_states(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1045-1083: Method `TransformersBase.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = False,
    ) -> Union[LogitsProcessorOutput, EmbeddingPoolerOutput, PPProxyTensors]:
        runtime_input_ids: Optional[torch.Tensor] = input_ids
        runtime_input_embeds = input_embeds
        if not self.pp_group.is_first_rank:
            assert pp_proxy_tensors is not None
            runtime_input_ids = None
            runtime_input_embeds = pp_proxy_tensors["hidden_states"]

        hidden_states = self._forward_hidden_states(
            input_ids=runtime_input_ids,
            positions=positions,
            forward_batch=forward_batch,
            input_embeds=runtime_input_embeds,
        )

        if not self.pp_group.is_last_rank:
            return PPProxyTensors(
                {"hidden_states": hidden_states, "residual": hidden_states}
            )

        if get_embedding:
            assert (
                self.pooler is not None
            ), "pooling is not enabled for this model class"
            return self.pooler(hidden_states, forward_batch)

        assert self.logits_processor is not None and self.lm_head is not None
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch, None
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., pp_proxy_tensors: ...=..., input_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., pp_proxy_tensors: ...=..., input_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1086-1094: Method `TransformersBase.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=self.skip_prefixes,
            skip_substrs=self.skip_substrs,
            ignore_unexpected_prefixes=self.ignore_unexpected_prefixes,
            ignore_unexpected_suffixes=self.ignore_unexpected_suffixes,
        )
        return loader.load_weights(weights, mapper=self.weight_mapper)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1097-1098: Class `CausalMixin` overview
```python
class CausalMixin:
```
**EN:** Defines `CausalMixin` as a reusable runtime type derived from no explicit base class. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `CausalMixin`，其继承关系为 no explicit base class。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 1099-1122: Method `CausalMixin.__init__`
```python
    def __init__(self, *args, prefix: str = "", **kwargs):
        super().__init__(*args, prefix=prefix, **kwargs)

        tie_word_embeddings = getattr(self.text_config, "tie_word_embeddings", False)
        if tie_word_embeddings:
            self.skip_prefixes.append("lm_head.")

        if not self.pp_group.is_last_rank:
            self._register_missing_prefix("lm_head")
            return

        self.lm_head = ParallelLMHead(
            self.vocab_size,
            self.text_config.hidden_size,
            quant_config=self.quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if tie_word_embeddings:
            self.lm_head.weight = self.model.get_input_embeddings().weight

        logit_scale = getattr(self.text_config, "logit_scale", 1.0)
        self.logits_processor = LogitsProcessor(
            self.text_config, logit_scale=logit_scale
        )
```
**EN:** This method implements `__init__(*args, prefix: ...=..., **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*args, prefix: ...=..., **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1125-1126: Class `EmbeddingMixin` overview
```python
class EmbeddingMixin:
```
**EN:** Defines `EmbeddingMixin` as a reusable runtime type derived from no explicit base class. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `EmbeddingMixin`，其继承关系为 no explicit base class。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 1127-1135: Method `EmbeddingMixin.__init__`
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.ignore_unexpected_prefixes.append("lm_head.")
        if not self.pp_group.is_last_rank:
            return
        pooling_name = str(getattr(self.config, "pooling_type", "LAST")).upper()
        pooling_type = PoolingType.CLS if pooling_name == "CLS" else PoolingType.LAST
        normalize = bool(getattr(self.config, "normalize", True))
        self.pooler = Pooler(pooling_type=pooling_type, normalize=normalize)
```
**EN:** This method implements `__init__(*args, **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*args, **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1138-1139: Class `MoEMixin` overview
```python
class MoEMixin:
```
**EN:** Defines `MoEMixin` as a reusable runtime type derived from no explicit base class. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MoEMixin`，其继承关系为 no explicit base class。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 1140-1141: Method `MoEMixin.__init__`
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
```
**EN:** This method implements `__init__(*args, **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*args, **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1143-1159: Method `MoEMixin.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(
        cls, config
    ) -> Optional[ModelConfigForExpertLocation]:
        text_config = getattr(config, "text_config", config)
        num_experts = _getattr_first(
            text_config,
            ("num_local_experts", "num_experts", "n_routed_experts"),
        )
        if num_experts is None:
            return None
        num_groups = getattr(text_config, "n_group", None)
        return ModelConfigForExpertLocation(
            num_layers=text_config.num_hidden_layers,
            num_logical_experts=num_experts,
            num_groups=num_groups,
        )
```
**EN:** This method implements `get_model_config_for_expert_location(config)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 1161-1165: Method `MoEMixin.routed_experts_weights_of_layer`
```python
    @property
    def routed_experts_weights_of_layer(self) -> dict[int, list[torch.Tensor]]:
        return {
            fused.experts.layer_id: fused.get_moe_weights() for fused in self.moe_layers
        }
```
**EN:** This method implements `routed_experts_weights_of_layer()` and handles weight mapping, filtering, or loading for this model component Decorators: property.
**CN:** 这个方法实现了 `routed_experts_weights_of_layer()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑 装饰器：property。

### Lines 1167-1189: Method `MoEMixin._get_expert_mapping`
```python
    def _get_expert_mapping(self, num_experts: int) -> List[Tuple[str, str, int, str]]:
        ckpt_names = [
            ("gate_proj", "down_proj", "up_proj"),
            ("w1", "w2", "w3"),
            ("linear", "linear_1", "linear_v"),
        ]
        mapping: list = []
        for gate, down, up in ckpt_names:
            mapping.extend(
                FusedMoE.make_expert_params_mapping(
                    ckpt_gate_proj_name=gate,
                    ckpt_down_proj_name=down,
                    ckpt_up_proj_name=up,
                    num_experts=num_experts,
                )
            )
        # AutoWeightsLoader dispatches to TransformersFusedMoE (which IS the
        # ``experts`` module) so the incoming weight names have the "experts."
        # prefix already stripped.  Remove it from weight_name in the mapping.
        mapping = [
            (pn, wn.removeprefix("experts."), eid, sid) for pn, wn, eid, sid in mapping
        ]
        return mapping
```
**EN:** This method implements `_get_expert_mapping(num_experts: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_expert_mapping(num_experts: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1191-1303: Method `MoEMixin.recursive_replace`
```python
    def recursive_replace(self):
        """Replace experts modules with TransformersFusedMoE, then call
        super().recursive_replace() for Linear/RMSNorm replacement."""
        text_config = self.text_config

        num_experts = _getattr_first(
            text_config,
            ("num_local_experts", "num_experts", "n_routed_experts"),
        )
        assert num_experts is not None, "Cannot determine num_experts from config."

        top_k = _getattr_first(text_config, ("num_experts_per_tok", "top_k"))
        assert top_k is not None, "Cannot determine top_k from config."

        hidden_size = text_config.hidden_size
        intermediate_size = _getattr_first(
            text_config,
            ("moe_intermediate_size", "intermediate_size"),
        )
        assert intermediate_size is not None, "Cannot determine intermediate_size."

        num_shared_experts = _getattr_first(
            text_config,
            ("n_shared_experts", "moe_num_shared_experts"),
            0,
        )
        reduce_results = num_shared_experts == 0

        renormalize = getattr(text_config, "norm_topk_prob", top_k > 1)

        # Activation function
        activation = "silu"
        wrapped_arch = self.config.architectures[0].lower()
        if "gptoss" in wrapped_arch:
            activation = "swigluoai"
        elif "grok1" in wrapped_arch:
            activation = "gelu"

        # Expert mapping for AutoWeightsLoader
        expert_mapping = self._get_expert_mapping(num_experts)
# ... truncated for brevity ...
```
**EN:** This method implements `recursive_replace()` and Replace experts modules with TransformersFusedMoE, then call.
**CN:** 这个方法实现了 `recursive_replace()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1306-1306: Class `MultiModalMixin` overview
```python
class MultiModalMixin:
```
**EN:** Defines `MultiModalMixin` as a reusable runtime type derived from no explicit base class. The class groups 13 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MultiModalMixin`，其继承关系为 no explicit base class。这个类组织了 13 个方法，用于实现模型相关行为。

### Lines 1307-1341: Class `MultiModalMixin` attributes
```python
    torch_compile_dynamic_arg_dims: dict[str, int] = _MULTIMODAL_DYNAMIC_ARG_DIMS

    # Older VL checkpoints (e.g. Qwen2.5-VL) store text weights as
    # "model.layers.*" but transformers >=5.0 nests the text model under
    # "model.language_model.*".  Map explicitly so these load correctly.
    hf_to_sglang_mapper = WeightsMapper(
        orig_to_new_prefix={
            "language_model.model.": "model.language_model.",
            "text_model.model.": "model.text_model.",
            "text_model.lm_head.": "lm_head.",
            "language_model.lm_head.": "lm_head.",
            "vision_tower.": "model.vision_tower.",
            "vision_model.": "model.vision_model.",
            "vision_embed_tokens.": "model.vision_embed_tokens.",
            "image_newline.": "model.image_newline.",
            "vqmodel.": "model.vqmodel.",
            "multi_modal_projector.": "model.multi_modal_projector.",
            "visual.": "model.visual.",
            "model.layers.": "model.language_model.layers.",
            "model.embed_tokens.": "model.language_model.embed_tokens.",
            "model.norm.": "model.language_model.norm.",
            "model.rotary_emb.": "model.language_model.rotary_emb.",
        }
    )

    _mm_feature_kwarg = {
        "image": "pixel_values",
        "video": "pixel_values_videos",
        "audio": "input_features",
    }
    _mm_encoder_candidates = {
        "image": ("get_image_features", "get_image_feature"),
        "video": ("get_video_features", "get_video_feature"),
        "audio": ("get_audio_features", "get_audio_feature"),
    }
```
**EN:** Defines class-level attributes and metadata that shape how `MultiModalMixin` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MultiModalMixin` 在运行时的行为。

### Lines 1343-1345: Method `MultiModalMixin.__init__`
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._mm_padding_pattern = MultiModalityDataPaddingPatternMultimodalTokens()
```
**EN:** This method implements `__init__(*args, **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*args, **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1347-1352: Method `MultiModalMixin._uses_mrope_positions`
```python
    def _uses_mrope_positions(self) -> bool:
        rope_scaling = getattr(self.text_config, "rope_scaling", None)
        if isinstance(rope_scaling, Mapping) and "mrope_section" in rope_scaling:
            return True
        rope_type = str(getattr(self.text_config, "rope_type", "")).lower()
        return "mrope" in rope_type
```
**EN:** This method implements `_uses_mrope_positions()` and implements a focused helper that supports the surrounding runtime flow inside `MultiModalMixin`.
**CN:** 这个方法实现了 `_uses_mrope_positions()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MultiModalMixin` 内部调用。

### Lines 1354-1355: Method `MultiModalMixin.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: list[int], mm_inputs: MultimodalInputs):
        return input_ids
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MultiModalMixin`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MultiModalMixin` 内部调用。

### Lines 1357-1362: Method `MultiModalMixin._get_modality_encoder`
```python
    def _get_modality_encoder(self, modality_name: str):
        for name in self._mm_encoder_candidates[modality_name]:
            fn = getattr(self.model, name, None)
            if fn is not None:
                return fn
        raise AttributeError(f"No encoder method found for modality '{modality_name}'")
```
**EN:** This method implements `_get_modality_encoder(modality_name: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_modality_encoder(modality_name: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1364-1386: Method `MultiModalMixin._get_modality_dtype_device`
```python
    def _get_modality_dtype_device(
        self, modality_name: str
    ) -> tuple[Optional[torch.dtype], Optional[torch.device]]:
        module_candidates = {
            "image": ("vision_tower", "vision_model"),
            "video": ("video_tower", "vision_tower", "vision_model"),
            "audio": ("audio_tower", "audio_model", "audio_encoder"),
        }
        modules = []
        for name in module_candidates.get(modality_name, ()):
            module = getattr(self.model, name, None)
            if module is not None:
                modules.append(module)
        modules.append(self.model)

        for module in modules:
            for param in module.parameters():
                if torch.is_floating_point(param):
                    return param.dtype, param.device
            for buf in module.buffers():
                if torch.is_floating_point(buf):
                    return buf.dtype, buf.device
        return None, None
```
**EN:** This method implements `_get_modality_dtype_device(modality_name: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_modality_dtype_device(modality_name: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1388-1399: Method `MultiModalMixin._cast_mm_value`
```python
    def _cast_mm_value(self, value, dtype, device):
        if torch.is_tensor(value):
            if value.is_floating_point() and dtype is not None:
                return value.to(dtype=dtype, device=device)
            return value
        if isinstance(value, dict):
            return {k: self._cast_mm_value(v, dtype, device) for k, v in value.items()}
        if isinstance(value, list):
            return [self._cast_mm_value(v, dtype, device) for v in value]
        if isinstance(value, tuple):
            return tuple(self._cast_mm_value(v, dtype, device) for v in value)
        return value
```
**EN:** This method implements `_cast_mm_value(value, dtype, device)` and implements a focused helper that supports the surrounding runtime flow inside `MultiModalMixin`.
**CN:** 这个方法实现了 `_cast_mm_value(value, dtype, device)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MultiModalMixin` 内部调用。

### Lines 1401-1439: Method `MultiModalMixin._to_tensor_output`
```python
    def _to_tensor_output(self, output) -> torch.Tensor:
        if hasattr(output, "pooler_output") and output.pooler_output is not None:
            output = output.pooler_output
        if isinstance(output, tuple):
            output = output[0]
        if isinstance(output, (list, tuple)):
            if len(output) == 0:
                raise ValueError("Empty multimodal encoder output.")
            if all(torch.is_tensor(x) for x in output):
                output = torch.cat(
                    [x.reshape(-1, x.shape[-1]) if x.ndim > 2 else x for x in output],
                    dim=0,
                )
            else:
                output = output[0]
        elif hasattr(output, "last_hidden_state"):
            output = output.last_hidden_state
        elif isinstance(output, dict):
            if output.get("pooler_output", None) is not None:
                output = output["pooler_output"]
            else:
                output = next(v for v in output.values() if torch.is_tensor(v))
            if isinstance(output, (list, tuple)):
                if len(output) == 0:
                    raise ValueError("Empty multimodal encoder output.")
                if all(torch.is_tensor(x) for x in output):
                    output = torch.cat(
                        [
                            x.reshape(-1, x.shape[-1]) if x.ndim > 2 else x
                            for x in output
                        ],
                        dim=0,
                    )
                else:
                    output = output[0]

        if output.ndim > 2:
            output = output.reshape(-1, output.shape[-1])
        return output
```
**EN:** This method implements `_to_tensor_output(output)` and implements a focused helper that supports the surrounding runtime flow inside `MultiModalMixin`.
**CN:** 这个方法实现了 `_to_tensor_output(output)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MultiModalMixin` 内部调用。

### Lines 1441-1465: Method `MultiModalMixin._encode_modality_items`
```python
    def _encode_modality_items(
        self, modality_name: str, items: list[MultimodalDataItem]
    ) -> torch.Tensor:
        encoder = self._get_modality_encoder(modality_name)
        feature_kwarg = self._mm_feature_kwarg[modality_name]
        target_dtype, target_device = self._get_modality_dtype_device(modality_name)
        outputs = []
        for item in items:
            kwargs = self._cast_mm_value(
                dict(item.model_specific_data),
                dtype=target_dtype,
                device=target_device,
            )
            feature = self._cast_mm_value(
                item.feature,
                dtype=target_dtype,
                device=target_device,
            )
            if _encoder_accepts_feature_kwarg(encoder, feature_kwarg):
                kwargs[feature_kwarg] = feature
                result = encoder(**kwargs)
            else:
                result = encoder(feature, **kwargs)
            outputs.append(self._to_tensor_output(result))
        return torch.cat(outputs, dim=0)
```
**EN:** This method implements `_encode_modality_items(modality_name: ..., items: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MultiModalMixin`.
**CN:** 这个方法实现了 `_encode_modality_items(modality_name: ..., items: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MultiModalMixin` 内部调用。

### Lines 1467-1468: Method `MultiModalMixin.get_image_feature`
```python
    def get_image_feature(self, items: list[MultimodalDataItem]) -> torch.Tensor:
        return self._encode_modality_items("image", items)
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1470-1471: Method `MultiModalMixin.get_video_feature`
```python
    def get_video_feature(self, items: list[MultimodalDataItem]) -> torch.Tensor:
        return self._encode_modality_items("video", items)
```
**EN:** This method implements `get_video_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_video_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1473-1474: Method `MultiModalMixin.get_audio_feature`
```python
    def get_audio_feature(self, items: list[MultimodalDataItem]) -> torch.Tensor:
        return self._encode_modality_items("audio", items)
```
**EN:** This method implements `get_audio_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1476-1530: Method `MultiModalMixin._collect_mm_kwargs`
```python
    def _collect_mm_kwargs(self, forward_batch: ForwardBatch) -> dict:
        """Collect multimodal tensors from the forward batch and return them
        as kwargs suitable for the HF model's forward method."""
        kwargs = {}

        if getattr(forward_batch, "token_type_ids", None) is not None:
            tti = forward_batch.token_type_ids
            if tti.ndim == 1:
                tti = tti.unsqueeze(0)
            token_type_key = (
                "mm_token_type_ids"
                if "mm_token_type_ids"
                in inspect.signature(self.model.forward).parameters
                else "token_type_ids"
            )
            kwargs[token_type_key] = tti

        if (
            not forward_batch.forward_mode.is_decode()
            and forward_batch.contains_mm_inputs()
        ):
            mm_inputs = forward_batch.mm_inputs
            target_device = next(self.model.parameters()).device

            for batch_idx in range(len(mm_inputs or [])):
                mm_input = mm_inputs[batch_idx]
                if mm_input is None:
                    continue
                for item in mm_input.mm_items or []:
                    for key, value in (item.model_specific_data or {}).items():
                        if isinstance(value, torch.Tensor):
                            value = value.to(device=target_device)
                        if key not in kwargs:
                            kwargs[key] = value
                        elif isinstance(value, torch.Tensor) and isinstance(
                            kwargs[key], torch.Tensor
                        ):
                            kwargs[key] = torch.cat([kwargs[key], value], dim=0)
                    if item.feature is not None:
                        feature_key = self._mm_feature_kwarg.get(
# ... truncated for brevity ...
```
**EN:** This method implements `_collect_mm_kwargs(forward_batch: ...)` and Collect multimodal tensors from the forward batch and return them.
**CN:** 这个方法实现了 `_collect_mm_kwargs(forward_batch: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1532-1561: Method `MultiModalMixin._forward_hidden_states`
```python
    def _forward_hidden_states(
        self,
        input_ids: Optional[torch.Tensor],
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if input_embeds is not None:
            return super()._forward_hidden_states(
                input_ids=input_ids,
                positions=positions,
                forward_batch=forward_batch,
                input_embeds=input_embeds,
            )

        if (
            self._uses_mrope_positions()
            and getattr(forward_batch, "mrope_positions", None) is not None
        ):
            positions = forward_batch.mrope_positions

        mm_kwargs = self._collect_mm_kwargs(forward_batch)

        return self._run_hf_backbone(
            input_ids=input_ids,
            input_embeds=None,
            positions=positions,
            forward_batch=forward_batch,
            **mm_kwargs,
        )
```
**EN:** This method implements `_forward_hidden_states(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_hidden_states(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1564-1564: Class `TransformersForCausalLM` overview
```python
class TransformersForCausalLM(CausalMixin, TransformersBase):
```
**EN:** Defines `TransformersForCausalLM` as a reusable runtime type derived from CausalMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersForCausalLM`，其继承关系为 CausalMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1565-1565: Class `TransformersForCausalLM` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1568-1568: Class `TransformersMoEForCausalLM` overview
```python
class TransformersMoEForCausalLM(MoEMixin, CausalMixin, TransformersBase):
```
**EN:** Defines `TransformersMoEForCausalLM` as a reusable runtime type derived from MoEMixin, CausalMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMoEForCausalLM`，其继承关系为 MoEMixin, CausalMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1569-1569: Class `TransformersMoEForCausalLM` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1572-1572: Class `TransformersMultiModalForCausalLM` overview
```python
class TransformersMultiModalForCausalLM(MultiModalMixin, CausalMixin, TransformersBase):
```
**EN:** Defines `TransformersMultiModalForCausalLM` as a reusable runtime type derived from MultiModalMixin, CausalMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMultiModalForCausalLM`，其继承关系为 MultiModalMixin, CausalMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1573-1573: Class `TransformersMultiModalForCausalLM` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1576-1578: Class `TransformersMultiModalMoEForCausalLM` overview
```python
class TransformersMultiModalMoEForCausalLM(
    MultiModalMixin, MoEMixin, CausalMixin, TransformersBase
):
```
**EN:** Defines `TransformersMultiModalMoEForCausalLM` as a reusable runtime type derived from MultiModalMixin, MoEMixin, CausalMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMultiModalMoEForCausalLM`，其继承关系为 MultiModalMixin, MoEMixin, CausalMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1579-1579: Class `TransformersMultiModalMoEForCausalLM` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1582-1582: Class `TransformersEmbeddingModel` overview
```python
class TransformersEmbeddingModel(EmbeddingMixin, TransformersBase):
```
**EN:** Defines `TransformersEmbeddingModel` as a reusable runtime type derived from EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersEmbeddingModel`，其继承关系为 EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1583-1583: Class `TransformersEmbeddingModel` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1586-1586: Class `TransformersMoEEmbeddingModel` overview
```python
class TransformersMoEEmbeddingModel(MoEMixin, EmbeddingMixin, TransformersBase):
```
**EN:** Defines `TransformersMoEEmbeddingModel` as a reusable runtime type derived from MoEMixin, EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMoEEmbeddingModel`，其继承关系为 MoEMixin, EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1587-1587: Class `TransformersMoEEmbeddingModel` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1590-1592: Class `TransformersMultiModalEmbeddingModel` overview
```python
class TransformersMultiModalEmbeddingModel(
    MultiModalMixin, EmbeddingMixin, TransformersBase
):
```
**EN:** Defines `TransformersMultiModalEmbeddingModel` as a reusable runtime type derived from MultiModalMixin, EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMultiModalEmbeddingModel`，其继承关系为 MultiModalMixin, EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1593-1593: Class `TransformersMultiModalEmbeddingModel` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1596-1598: Class `TransformersMultiModalMoEEmbeddingModel` overview
```python
class TransformersMultiModalMoEEmbeddingModel(
    MultiModalMixin, MoEMixin, EmbeddingMixin, TransformersBase
):
```
**EN:** Defines `TransformersMultiModalMoEEmbeddingModel` as a reusable runtime type derived from MultiModalMixin, MoEMixin, EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMultiModalMoEEmbeddingModel`，其继承关系为 MultiModalMixin, MoEMixin, EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1599-1599: Class `TransformersMultiModalMoEEmbeddingModel` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1602-1602: Class `TransformersForSequenceClassification` overview
```python
class TransformersForSequenceClassification(EmbeddingMixin, TransformersBase):
```
**EN:** Defines `TransformersForSequenceClassification` as a reusable runtime type derived from EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersForSequenceClassification`，其继承关系为 EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1603-1603: Class `TransformersForSequenceClassification` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1606-1608: Class `TransformersMoEForSequenceClassification` overview
```python
class TransformersMoEForSequenceClassification(
    MoEMixin, EmbeddingMixin, TransformersBase
):
```
**EN:** Defines `TransformersMoEForSequenceClassification` as a reusable runtime type derived from MoEMixin, EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMoEForSequenceClassification`，其继承关系为 MoEMixin, EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1609-1609: Class `TransformersMoEForSequenceClassification` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1612-1614: Class `TransformersMultiModalForSequenceClassification` overview
```python
class TransformersMultiModalForSequenceClassification(
    MultiModalMixin, EmbeddingMixin, TransformersBase
):
```
**EN:** Defines `TransformersMultiModalForSequenceClassification` as a reusable runtime type derived from MultiModalMixin, EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMultiModalForSequenceClassification`，其继承关系为 MultiModalMixin, EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1615-1615: Class `TransformersMultiModalForSequenceClassification` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1618-1620: Class `TransformersMultiModalMoEForSequenceClassification` overview
```python
class TransformersMultiModalMoEForSequenceClassification(
    MultiModalMixin, MoEMixin, EmbeddingMixin, TransformersBase
):
```
**EN:** Defines `TransformersMultiModalMoEForSequenceClassification` as a reusable runtime type derived from MultiModalMixin, MoEMixin, EmbeddingMixin, TransformersBase. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `TransformersMultiModalMoEForSequenceClassification`，其继承关系为 MultiModalMixin, MoEMixin, EmbeddingMixin, TransformersBase。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1621-1621: Class `TransformersMultiModalMoEForSequenceClassification` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1624-1637: Top-level assign
```python
EntryClass = [
    TransformersForCausalLM,
    TransformersMoEForCausalLM,
    TransformersMultiModalForCausalLM,
    TransformersMultiModalMoEForCausalLM,
    TransformersEmbeddingModel,
    TransformersMoEEmbeddingModel,
    TransformersMultiModalEmbeddingModel,
    TransformersMultiModalMoEEmbeddingModel,
    TransformersForSequenceClassification,
    TransformersMoEForSequenceClassification,
    TransformersMultiModalForSequenceClassification,
    TransformersMultiModalMoEForSequenceClassification,
]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `inspect`
- `logging`
- `re`
- `collections.abc.Iterable`
- `collections.abc.Mapping`
- `contextlib.contextmanager`
- `typing.List`
- `typing.Literal`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `transformers`
- `torch.nn`
- `transformers.AutoModel`
- `transformers.PretrainedConfig`
- `transformers.PreTrainedModel`
- `transformers.dynamic_module_utils.get_class_from_dynamic_module`
- `transformers.modeling_utils.ALL_ATTENTION_FUNCTIONS`
- `sglang.srt.distributed.divide`
- `sglang.srt.distributed.get_moe_expert_parallel_world_size`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_pp_indices`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.layers.layernorm.GemmaRMSNorm`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.moe.topk.StandardTopKOutput`
- `sglang.srt.layers.moe.utils.filter_moe_weight_param_global_expert`
- `sglang.srt.layers.pooler.EmbeddingPoolerOutput`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.utils.AutoWeightsLoader`
- `sglang.srt.models.utils.WeightsMapper`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.common.direct_register_custom_op`
- `sglang.srt.utils.hf_transformers_utils.get_hf_text_config`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.compilation.compilation_config.SPLIT_OPS`
- `transformers.AutoConfig`
