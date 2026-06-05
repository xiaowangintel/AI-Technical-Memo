# accuracy_hooks.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/accuracy_hooks.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates accuracy hooks with focused assertions and fixtures. Key symbols include `TransformerHookCompat`, `_resolve_transformer_hook_compat`, `HookCall`. / 该测试模块通过有针对性的断言与夹具，验证 accuracy hooks 的实现。 关键符号包括 `TransformerHookCompat`, `_resolve_transformer_hook_compat`, `HookCall`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Imports and module setup / 导入与模块初始化
```python
from __future__ import annotations

import inspect
from dataclasses import dataclass, field
from typing import Any, Callable, Dict, Optional

import torch
import torch.nn as nn

from sglang.multimodal_gen.test.server.accuracy_config import (
    DEFAULT_TIMESTEP,
    I2V_IMAGE_DIM,
    TIMESTEP_NORMALIZATION_FACTOR,
    ComponentType,
# ...
DEFAULT_TRANSFORMER_POOLED_CHANNELS = 768
DEFAULT_VAE_LATENT_CHANNELS = 16
DEFAULT_VAE_LATENT_SPATIAL_SIZE = 32
LARGE_CHANNEL_LAYOUT_THRESHOLD = 128
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 44-49: Class `TransformerHookCompat` / 类 `TransformerHookCompat`
```python
@dataclass(frozen=True)
class TransformerHookCompat:
    normalize_reference_timestep: bool = False
    negate_reference_output: bool = False
    omit_reference_guidance: bool = False
    use_2d_hidden_states: bool = False
```
**EN:** This class models `TransformerHookCompat`.
**CN:** 该类实现 `TransformerHookCompat`。

### Lines 52-71: Function `_resolve_transformer_hook_compat` / 函数 `_resolve_transformer_hook_compat`
```python
def _resolve_transformer_hook_compat(case: Any) -> TransformerHookCompat:
    model_path = case.server_args.model_path.lower()
    if "z-image" in model_path:
        return TransformerHookCompat(
            normalize_reference_timestep=True,
            negate_reference_output=True,
        )
    if "qwen" in model_path:
        return TransformerHookCompat(
            normalize_reference_timestep=True,
            omit_reference_guidance=True,
        )
    if "sana" in model_path:
        return TransformerHookCompat(
# ...
        )
    if "flux" in model_path:
        return TransformerHookCompat(normalize_reference_timestep=True)
    return TransformerHookCompat()
```
**EN:** This function drives `_resolve_transformer_hook_compat` with inputs such as `case`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_transformer_hook_compat`，主要处理 `case` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 74-79: Class `HookCall` / 类 `HookCall`
```python
@dataclass
class HookCall:
    module: nn.Module
    args: tuple[Any, ...] = ()
    kwargs: Dict[str, Any] = field(default_factory=dict)
    negate_output: bool = False
```
**EN:** This class models `HookCall`.
**CN:** 该类实现 `HookCall`。

### Lines 82-88: Class `NativeHookProfile` / 类 `NativeHookProfile`
```python
@dataclass(frozen=True)
class NativeHookProfile:
    build_inputs: BuildInputsFn
    prepare_sglang_call: PrepareCallFn
    prepare_reference_call: PrepareCallFn
    normalize_sglang_output: NormalizeFn = extract_output_tensor
    normalize_reference_output: NormalizeFn = extract_output_tensor
```
**EN:** This class models `NativeHookProfile`.
**CN:** 该类实现 `NativeHookProfile`。

### Lines 91-102: Class `_DeterministicRNG` / 类 `_DeterministicRNG`
```python
class _DeterministicRNG:
    def __init__(self, seed: int = 42) -> None:
        self._seed = seed

    def randn(
        self, shape: tuple[int, ...], device: str, dtype: torch.dtype
    ) -> torch.Tensor:
        torch.manual_seed(self._seed)
        tensor = torch.randn(shape, device="cpu", dtype=dtype).to(device)
        seed_and_broadcast(self._seed, tensor)
        self._seed += 1
        return tensor
```
**EN:** This class models `_DeterministicRNG`. Important methods include `__init__`, `randn`.
**CN:** 该类实现 `_DeterministicRNG`。 其中较重要的方法包括 `__init__`, `randn`。

### Lines 105-111: Function `_resolve_nested_attr` / 函数 `_resolve_nested_attr`
```python
def _resolve_nested_attr(obj: Any, path: str) -> Any:
    current = obj
    for name in path.split("."):
        if current is None or not hasattr(current, name):
            return None
        current = getattr(current, name)
    return current
```
**EN:** This function drives `_resolve_nested_attr` with inputs such as `obj`, `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_nested_attr`，主要处理 `obj`, `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 114-121: Function `_read_config_value` / 函数 `_read_config_value`
```python
def _read_config_value(model: nn.Module, keys: list[str], default: int) -> int:
    config = getattr(model, "config", None)
    for key in keys:
        for root in (model, config):
            value = _resolve_nested_attr(root, key) if root is not None else None
            if isinstance(value, int) and value > 0:
                return value
    return default
```
**EN:** This function drives `_read_config_value` with inputs such as `model`, `keys`, `default`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_read_config_value`，主要处理 `model`, `keys`, `default` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 124-125: Function `_forward_parameter_names` / 函数 `_forward_parameter_names`
```python
def _forward_parameter_names(module: nn.Module) -> set[str]:
    return set(inspect.signature(module.forward).parameters.keys())
```
**EN:** This function drives `_forward_parameter_names` with inputs such as `module`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_forward_parameter_names`，主要处理 `module` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 128-135: Function `_infer_transformer_layout` / 函数 `_infer_transformer_layout`
```python
def _infer_transformer_layout(param_names: set[str]) -> str:
    if "img_shapes" in param_names or "txt_seq_lens" in param_names:
        return "token_shapes"
    if "img_ids" in param_names or "txt_ids" in param_names:
        return "token_ids"
    if "x" in param_names or "cap_feats" in param_names:
        return "alias"
    return "video"
```
**EN:** This function drives `_infer_transformer_layout` with inputs such as `param_names`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_infer_transformer_layout`，主要处理 `param_names` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 138-153: Function `_build_position_ids` / 函数 `_build_position_ids`
```python
def _build_position_ids(
    height: int, width: int, dims: int, device: str
) -> tuple[torch.Tensor, torch.Tensor]:
    img_len = height * width
    txt_len = DEFAULT_TEXT_SEQ_LEN
    if dims == 4:
        img_ids = torch.zeros(img_len, 4, device=device, dtype=torch.bfloat16)
        img_ids[:, 1] = torch.arange(height).repeat_interleave(width)
        img_ids[:, 2] = torch.arange(width).repeat(height)
        txt_ids = torch.zeros(txt_len, 4, device=device, dtype=torch.bfloat16)
    else:
        img_ids = torch.zeros(img_len, 3, device=device, dtype=torch.bfloat16)
        img_ids[:, 0] = torch.arange(height).repeat_interleave(width)
        img_ids[:, 1] = torch.arange(width).repeat(height)
        txt_ids = torch.zeros(txt_len, 3, device=device, dtype=torch.bfloat16)
    return img_ids, txt_ids
```
**EN:** This function drives `_build_position_ids` with inputs such as `height`, `width`, `dims`, `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_build_position_ids`，主要处理 `height`, `width`, `dims`, `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 156-189: Function `_build_alias_rotary_freqs` / 函数 `_build_alias_rotary_freqs`
```python
def _build_alias_rotary_freqs(
    model: nn.Module, device: str, height: int, width: int
) -> tuple[tuple[torch.Tensor, torch.Tensor], tuple[torch.Tensor, torch.Tensor]]:
    cap_len = DEFAULT_TEXT_SEQ_LEN
    cap_pad_len = (-cap_len) % ALIAS_ROTARY_TEXT_PAD_MULTIPLE
    cap_ids = (
        torch.stack(
            torch.meshgrid(
                torch.arange(cap_len + cap_pad_len),
                torch.arange(1),
                torch.arange(1),
                indexing="ij",
            ),
            dim=-1,
# ...
    )
    cos_cap, sin_cap = model.rotary_emb(cap_ids)
    cos_img, sin_img = model.rotary_emb(img_ids)
    return ((cos_cap, sin_cap), (cos_img, sin_img))
```
**EN:** This function drives `_build_alias_rotary_freqs` with inputs such as `model`, `device`, `height`, `width`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_build_alias_rotary_freqs`，主要处理 `model`, `device`, `height`, `width` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 192-199: Function `_supports_image_conditioning` / 函数 `_supports_image_conditioning`
```python
def _supports_image_conditioning(module: nn.Module) -> bool:
    image_embedder = _resolve_nested_attr(module, "condition_embedder.image_embedder")
    if image_embedder is not None:
        return True
    image_dim = _read_config_value(
        module, ["arch_config.image_dim", "image_dim"], default=0
    )
    return image_dim > 0
```
**EN:** This function drives `_supports_image_conditioning` with inputs such as `module`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_supports_image_conditioning`，主要处理 `module` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 202-418: Function `_build_transformer_hook_inputs` / 函数 `_build_transformer_hook_inputs`
```python
def _build_transformer_hook_inputs(
    case: Any, model: nn.Module, device: str, ref_model: Optional[nn.Module] = None
) -> Inputs:
    """Build one synthetic input bundle that both transformer variants can consume."""
    compat = _resolve_transformer_hook_compat(case)
    param_names = _forward_parameter_names(model)
    if ref_model is not None:
        # The input bundle has to satisfy both call signatures.
        param_names.update(_forward_parameter_names(ref_model))

    rng = _DeterministicRNG()
    layout = _infer_transformer_layout(param_names)
    requires_audio_stream_inputs = (
        "audio_hidden_states" in param_names
# ...
            )

    inputs["hook_compat"] = compat
    return inputs
```
**EN:** This function drives `_build_transformer_hook_inputs` with inputs such as `case`, `model`, `device`, `ref_model`. Build one synthetic input bundle that both transformer variants can consume.
**CN:** 这个函数负责 `_build_transformer_hook_inputs`，主要处理 `case`, `model`, `device`, `ref_model` 等输入。 文档字符串说明：Build one synthetic input bundle that both transformer variants can consume.

### Lines 421-424: Function `_get_transformer_hook_compat` / 函数 `_get_transformer_hook_compat`
```python
def _get_transformer_hook_compat(inputs: Inputs) -> TransformerHookCompat:
    compat = inputs.get("hook_compat")
    assert isinstance(compat, TransformerHookCompat)
    return compat
```
**EN:** This function drives `_get_transformer_hook_compat` with inputs such as `inputs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_transformer_hook_compat`，主要处理 `inputs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 427-448: Function `_supports_guidance_embedding` / 函数 `_supports_guidance_embedding`
```python
def _supports_guidance_embedding(module: nn.Module) -> bool:
    time_text_embed = getattr(module, "time_text_embed", None)
    if time_text_embed is None:
        return True

    parameters = list(inspect.signature(time_text_embed.forward).parameters.values())

    if any(param.kind is inspect.Parameter.VAR_POSITIONAL for param in parameters):
        return True

    accepted_args = [
        param
        for param in parameters
        if param.name != "self"
# ...
            inspect.Parameter.KEYWORD_ONLY,
        )
    ]
    return len(accepted_args) >= 3
```
**EN:** This function drives `_supports_guidance_embedding` with inputs such as `module`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_supports_guidance_embedding`，主要处理 `module` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 451-545: Function `_prepare_transformer_hook_call` / 函数 `_prepare_transformer_hook_call`
```python
def _prepare_transformer_hook_call(
    module: nn.Module, inputs: Inputs, side: str
) -> HookCall:
    param_names = _forward_parameter_names(module)
    signature = inspect.signature(module.forward)
    compat = _get_transformer_hook_compat(inputs)
    kwargs: Dict[str, Any] = {}
    negate_output = side == "reference" and compat.negate_reference_output

    if "hidden_states" in param_names:
        kwargs["hidden_states"] = inputs["hidden_states"]
    if "x" in param_names:
        kwargs["x"] = [inputs["hidden_states"].squeeze(0)]
    if "encoder_hidden_states" in param_names:
# ...
    if "return_dict" in param_names:
        kwargs["return_dict"] = True

    return HookCall(module=module, kwargs=kwargs, negate_output=negate_output)
```
**EN:** This function drives `_prepare_transformer_hook_call` with inputs such as `module`, `inputs`, `side`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_prepare_transformer_hook_call`，主要处理 `module`, `inputs`, `side` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 548-549: Function `_prepare_transformer_sglang_call` / 函数 `_prepare_transformer_sglang_call`
```python
def _prepare_transformer_sglang_call(module: nn.Module, inputs: Inputs) -> HookCall:
    return _prepare_transformer_hook_call(module, inputs, side="sglang")
```
**EN:** This function drives `_prepare_transformer_sglang_call` with inputs such as `module`, `inputs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_prepare_transformer_sglang_call`，主要处理 `module`, `inputs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 552-553: Function `_prepare_transformer_reference_call` / 函数 `_prepare_transformer_reference_call`
```python
def _prepare_transformer_reference_call(module: nn.Module, inputs: Inputs) -> HookCall:
    return _prepare_transformer_hook_call(module, inputs, side="reference")
```
**EN:** This function drives `_prepare_transformer_reference_call` with inputs such as `module`, `inputs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_prepare_transformer_reference_call`，主要处理 `module`, `inputs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 556-564: Function `_normalize_transformer_reference_output` / 函数 `_normalize_transformer_reference_output`
```python
def _normalize_transformer_reference_output(output: Any) -> torch.Tensor:
    sample = getattr(output, "sample", None)
    if (
        isinstance(sample, (list, tuple))
        and sample
        and all(isinstance(item, torch.Tensor) for item in sample)
    ):
        return torch.stack(list(sample), dim=0)
    return extract_output_tensor(output)
```
**EN:** This function drives `_normalize_transformer_reference_output` with inputs such as `output`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_normalize_transformer_reference_output`，主要处理 `output` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 567-585: Class `_VAEDecodeModule` / 类 `_VAEDecodeModule`
```python
class _VAEDecodeModule(nn.Module):
    def __init__(self, vae: nn.Module):
        super().__init__()
        self.vae = vae

    def forward(self, z: torch.Tensor) -> torch.Tensor:
        if (
            any(
                isinstance(module, (nn.Conv3d, nn.ConvTranspose3d))
                for module in self.vae.modules()
            )
            and z.ndim == 4
        ):
            z = z.unsqueeze(2)
        output = self.vae.decode(z)
        tensor = output.sample if hasattr(output, "sample") else output
        if isinstance(tensor, (list, tuple)):
            tensor = tensor[0]
        return tensor.squeeze(2) if tensor.ndim == 5 else tensor
```
**EN:** This class models `_VAEDecodeModule` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `_VAEDecodeModule`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 588-607: Function `_infer_vae_latent_channels` / 函数 `_infer_vae_latent_channels`
```python
def _infer_vae_latent_channels(model: nn.Module) -> int:
    for path in ("post_quant_conv.in_channels", "post_quant_conv.conv.in_channels"):
        value = _resolve_nested_attr(model, path)
        if isinstance(value, int) and value > 0:
            return value
    return _read_config_value(
        model,
        [
            "z_dim",
            "arch_config.z_dim",
            "latent_channels",
            "arch_config.latent_channels",
            "num_channels_latents",
            "arch_config.num_channels_latents",
# ...
            "arch_config.z_channels",
        ],
        default=DEFAULT_VAE_LATENT_CHANNELS,
    )
```
**EN:** This function drives `_infer_vae_latent_channels` with inputs such as `model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_infer_vae_latent_channels`，主要处理 `model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 610-627: Function `_build_vae_hook_inputs` / 函数 `_build_vae_hook_inputs`
```python
def _build_vae_hook_inputs(
    case: Any, model: nn.Module, device: str, ref_model: Optional[nn.Module] = None
) -> Inputs:
    del case, ref_model
    latent_channels = _infer_vae_latent_channels(model)
    rng = _DeterministicRNG()
    return {
        "z": rng.randn(
            (
                1,
                latent_channels,
                DEFAULT_VAE_LATENT_SPATIAL_SIZE,
                DEFAULT_VAE_LATENT_SPATIAL_SIZE,
            ),
            device,
            torch.bfloat16,
        )
    }
```
**EN:** This function drives `_build_vae_hook_inputs` with inputs such as `case`, `model`, `device`, `ref_model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_build_vae_hook_inputs`，主要处理 `case`, `model`, `device`, `ref_model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 630-631: Function `_prepare_vae_decode_call` / 函数 `_prepare_vae_decode_call`
```python
def _prepare_vae_decode_call(module: nn.Module, inputs: Inputs) -> HookCall:
    return HookCall(module=_VAEDecodeModule(module), args=(inputs["z"],))
```
**EN:** This function drives `_prepare_vae_decode_call` with inputs such as `module`, `inputs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_prepare_vae_decode_call`，主要处理 `module`, `inputs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 632-645: Top-level configuration / 顶层配置
```python


TRANSFORMER_NATIVE_PROFILE = NativeHookProfile(
    build_inputs=_build_transformer_hook_inputs,
    prepare_sglang_call=_prepare_transformer_sglang_call,
    prepare_reference_call=_prepare_transformer_reference_call,
    normalize_reference_output=_normalize_transformer_reference_output,
)

VAE_NATIVE_PROFILE = NativeHookProfile(
    build_inputs=_build_vae_hook_inputs,
    prepare_sglang_call=_prepare_vae_decode_call,
    prepare_reference_call=_prepare_vae_decode_call,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 648-653: Function `resolve_component_native_profile` / 函数 `resolve_component_native_profile`
```python
def resolve_component_native_profile(component: ComponentType) -> NativeHookProfile:
    if component == ComponentType.TRANSFORMER:
        return TRANSFORMER_NATIVE_PROFILE
    if component == ComponentType.VAE:
        return VAE_NATIVE_PROFILE
    raise KeyError(f"Unsupported native accuracy component: {component.value}")
```
**EN:** This function drives `resolve_component_native_profile` with inputs such as `component`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `resolve_component_native_profile`，主要处理 `component` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Automated verification / 自动化验证
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test.server.accuracy_config`, `sglang.multimodal_gen.test.server.accuracy_utils`
- **External / 外部**: `__future__`, `torch`, `torch.nn`
- **Stdlib / 标准库**: `inspect`, `dataclasses`, `typing`
