# fsdp_load.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/fsdp_load.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module handles loading, adaptation, or initialization for fsdp load components used by multimodal generation. Key symbols include `_format_dtype_mismatch_summary`, `_make_param_like`, `_get_param_for_weight_loading`. / 该模块负责多模态生成中 fsdp load 组件的加载、适配或初始化。 关键符号包括 `_format_dtype_mismatch_summary`, `_make_param_like`, `_get_param_for_weight_loading`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-51: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0

# Adapted from torchtune
# Copyright 2024 The TorchTune Authors.
# Copyright 2025 The sglang-diffusion Authors.

from collections import Counter, defaultdict
from collections.abc import Callable, Generator
from itertools import chain
from typing import Any

import torch
# ...
    torch.float8_e5m2,
    torch.int8,
)
_DTYPE_MISMATCH_EXAMPLE_LIMIT = 3
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 54-65: Function `_format_dtype_mismatch_summary` / 函数 `_format_dtype_mismatch_summary`
```python
def _format_dtype_mismatch_summary(
    mismatch_counts: Counter[tuple[torch.dtype, torch.dtype]],
    mismatch_examples: dict[tuple[torch.dtype, torch.dtype], list[str]],
) -> str:
    parts: list[str] = []
    for (checkpoint_dtype, target_dtype), count in mismatch_counts.items():
        examples = mismatch_examples[(checkpoint_dtype, target_dtype)]
        part = f"{checkpoint_dtype}->{target_dtype} x{count}"
        if examples:
            part += f" (e.g. {', '.join(examples)})"
        parts.append(part)
    return "; ".join(parts)
```
**EN:** This function drives `_format_dtype_mismatch_summary` with inputs such as `mismatch_counts`, `mismatch_examples`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_format_dtype_mismatch_summary`，主要处理 `mismatch_counts`, `mismatch_examples` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 68-80: Function `_make_param_like` / 函数 `_make_param_like`
```python
def _make_param_like(
    actual_param: torch.nn.Parameter, tensor: torch.Tensor
) -> torch.nn.Parameter:
    cls = actual_param.__class__
    # nn.Parameter defaults to requires_grad=True, which is illegal for non-floating/complex dtypes (e.g., int8/FP8
    # quantized weights).
    try:
        new_param = cls.__new__(cls, tensor, requires_grad=False)
    except TypeError:
        new_param = cls.__new__(cls, tensor)
    new_param.__dict__.update(actual_param.__dict__)
    new_param.requires_grad = False
    return new_param
```
**EN:** This function drives `_make_param_like` with inputs such as `actual_param`, `tensor`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_make_param_like`，主要处理 `actual_param`, `tensor` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 83-97: Function `_get_param_for_weight_loading` / 函数 `_get_param_for_weight_loading`
```python
def _get_param_for_weight_loading(
    model: torch.nn.Module,
    param_dict: dict[str, torch.nn.Parameter],
    param_name: str,
) -> torch.nn.Parameter | None:
    actual_param = param_dict.get(param_name)
    if actual_param is not None and getattr(actual_param, "weight_loader", None):
        return actual_param

    pre_fsdp_weight_loader_params = getattr(model, "_pre_fsdp_weight_loader_params", {})
    pre_fsdp_param = pre_fsdp_weight_loader_params.get(param_name)
    if pre_fsdp_param is not None:
        return pre_fsdp_param

    return actual_param
```
**EN:** This function drives `_get_param_for_weight_loading` with inputs such as `model`, `param_dict`, `param_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_param_for_weight_loading`，主要处理 `model`, `param_dict`, `param_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 100-104: Function `_make_class_name_shard_condition` / 函数 `_make_class_name_shard_condition`
```python
def _make_class_name_shard_condition(class_names: set[str]):
    def shard_condition(n: str, m: nn.Module) -> bool:
        return type(m).__name__ in class_names

    return shard_condition
```
**EN:** This function drives `_make_class_name_shard_condition` with inputs such as `class_names`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_make_class_name_shard_condition`，主要处理 `class_names` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 107-121: Function `_is_common_numbered_block` / 函数 `_is_common_numbered_block`
```python
def _is_common_numbered_block(n: str, m: nn.Module) -> bool:
    return is_module_list_entry_in(
        n,
        (
            "blocks",
            "layers",
            "double_blocks",
            "single_blocks",
            "refiner_blocks",
            "noise_refiner",
            "context_refiner",
            "transformer_blocks",
            "single_transformer_blocks",
        ),
    )
```
**EN:** This function drives `_is_common_numbered_block` with inputs such as `n`, `m`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_is_common_numbered_block`，主要处理 `n`, `m` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 124-136: Function `_resolve_fsdp_shard_conditions` / 函数 `_resolve_fsdp_shard_conditions`
```python
def _resolve_fsdp_shard_conditions(
    model: torch.nn.Module,
    fsdp_shard_conditions: list[Callable[[str, nn.Module], bool]] | None,
) -> tuple[list[Callable[[str, nn.Module], bool]], str]:
    if fsdp_shard_conditions:
        return fsdp_shard_conditions, "explicit"

    block_class_names = set(getattr(model, "_repeated_blocks", []) or [])
    block_class_names.update(getattr(model, "_no_split_modules", []) or [])
    if block_class_names:
        return [_make_class_name_shard_condition(block_class_names)], "block-class"

    return [_is_common_numbered_block], "common-numbered-block"
```
**EN:** This function drives `_resolve_fsdp_shard_conditions` with inputs such as `model`, `fsdp_shard_conditions`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_fsdp_shard_conditions`，主要处理 `model`, `fsdp_shard_conditions` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 139-166: Function `_maybe_dequantize_fp8` / 函数 `_maybe_dequantize_fp8`
```python
def _maybe_dequantize_fp8(
    full_tensor: torch.Tensor,
    target_dtype: torch.dtype,
    target_param_name: str,
    param_sd: dict[str, torch.Tensor],
) -> torch.Tensor:
    """Auto-dequantize an FP8 checkpoint weight when the model parameter expects a higher-precision type.

    Some modules (e.g. AdaLayerNormZero) don't accept quant_config, so their
    parameters remain in higher precision even when the checkpoint stores FP8
    weights.  In that case we multiply by the per-tensor weight_scale to
    recover the original unquantized value.
    """
    if not (
# ...
            target_param_name,
            scale_key,
        )
    return full_tensor
```
**EN:** This function drives `_maybe_dequantize_fp8` with inputs such as `full_tensor`, `target_dtype`, `target_param_name`, `param_sd`. Auto-dequantize an FP8 checkpoint weight when the model parameter expects a higher-precision type.
**CN:** 这个函数负责 `_maybe_dequantize_fp8`，主要处理 `full_tensor`, `target_dtype`, `target_param_name`, `param_sd` 等输入。 文档字符串说明：Auto-dequantize an FP8 checkpoint weight when the model parameter expects a higher-precision type.

### Lines 170-279: Function `maybe_load_fsdp_model` / 函数 `maybe_load_fsdp_model`
```python
def maybe_load_fsdp_model(
    model_cls: type[nn.Module],
    init_params: dict[str, Any],
    weight_dir_list: list[str],
    device: torch.device,
    hsdp_replicate_dim: int,
    hsdp_shard_dim: int,
    param_dtype: torch.dtype,
    reduce_dtype: torch.dtype,
    cpu_offload: bool = False,
    fsdp_inference: bool = False,
    output_dtype: torch.dtype | None = None,
    pin_cpu_memory: bool = True,
    strict: bool = True,
# ...
        # Avoid unintended computation graph accumulation during inference
        if isinstance(p, torch.nn.Parameter):
            p.requires_grad = False
    return model
```
**EN:** This function drives `maybe_load_fsdp_model` with inputs such as `model_cls`, `init_params`, `weight_dir_list`, `device`. Load a model with optional FSDP (Fully Sharded Data Parallel) support.
**CN:** 这个函数负责 `maybe_load_fsdp_model`，主要处理 `model_cls`, `init_params`, `weight_dir_list`, `device` 等输入。 文档字符串说明：Load a model with optional FSDP (Fully Sharded Data Parallel) support.

### Lines 282-353: Function `shard_model` / 函数 `shard_model`
```python
def shard_model(
    model,
    *,
    cpu_offload: bool,
    reshard_after_forward: bool = True,
    mp_policy: MixedPrecisionPolicy | None = MixedPrecisionPolicy(),  # noqa
    mesh: DeviceMesh | None = None,
    fsdp_shard_conditions: list[Callable[[str, nn.Module], bool]] | None = None,
    pin_cpu_memory: bool = True,
) -> None:
    """
    Utility to shard a model with FSDP using the PyTorch Distributed fully_shard API.

    This method will over the model's named modules from the bottom-up and apply shard modules
# ...
        num_layers_sharded,
        type(model).__name__,
        condition_source,
    )
```
**EN:** This function drives `shard_model` with inputs such as `model`, `cpu_offload`, `reshard_after_forward`, `mp_policy`. Utility to shard a model with FSDP using the PyTorch Distributed fully_shard API.
**CN:** 这个函数负责 `shard_model`，主要处理 `model`, `cpu_offload`, `reshard_after_forward`, `mp_policy` 等输入。 文档字符串说明：Utility to shard a model with FSDP using the PyTorch Distributed fully_shard API.

### Lines 357-682: Function `load_model_from_full_model_state_dict` / 函数 `load_model_from_full_model_state_dict`
```python
def load_model_from_full_model_state_dict(
    model: FSDPModule | torch.nn.Module,
    full_sd_iterator: Generator[tuple[str, torch.Tensor], None, None],
    device: torch.device,
    param_dtype: torch.dtype | None,
    strict: bool = False,
    cpu_offload: bool = False,
    param_names_mapping: Callable[[str], tuple[str, Any, Any]] | None = None,
) -> _IncompatibleKeys:
    """
    Converting full state dict into a sharded state dict
    and loading it into FSDP model (if training) or normal huggingface model
    Args:
        model (Union[FSDPModule, torch.nn.Module]): Model to generate fully qualified names for cpu_state_dict
# ...
        sharded_sd[new_param_name] = nn.Parameter(sharded_tensor)

    # choose `assign=True` since we cannot call `copy_` on meta tensor
    return model.load_state_dict(sharded_sd, strict=strict, assign=True)
```
**EN:** This function drives `load_model_from_full_model_state_dict` with inputs such as `model`, `full_sd_iterator`, `device`, `param_dtype`. Converting full state dict into a sharded state dict
**CN:** 这个函数负责 `load_model_from_full_model_state_dict`，主要处理 `model`, `full_sd_iterator`, `device`, `param_dtype` 等输入。 文档字符串说明：Converting full state dict into a sharded state dict

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.fsdp`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.loader.weight_utils`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`, `sglang.srt.utils`
- **External / 外部**: `torch`, `torch.distributed`, `torch.distributed._tensor`, `torch.distributed.fsdp`, `torch.nn.modules.module`
- **Stdlib / 标准库**: `collections`, `collections.abc`, `itertools`, `typing`
