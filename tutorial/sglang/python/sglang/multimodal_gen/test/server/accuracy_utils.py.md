# accuracy_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/accuracy_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates accuracy utils with focused assertions and fixtures. Key symbols include `ComponentSelection`, `ParameterShardContext`, `seed_and_broadcast`. / 该测试模块通过有针对性的断言与夹具，验证 accuracy utils 的实现。 关键符号包括 `ComponentSelection`, `ParameterShardContext`, `seed_and_broadcast`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-62: Imports and module setup / 导入与模块初始化
```python
from __future__ import annotations

import json
import os
import shlex
from contextlib import nullcontext
from dataclasses import dataclass
from typing import Any, Dict, List, Optional, Tuple

import torch
import torch.nn as nn
from safetensors.torch import load_file as safetensors_load_file
from torch.distributed.tensor import distribute_tensor

# ...
    "image_encoder.",
    "encoder.",
    "decoder.",
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 65-70: Class `ComponentSelection` / 类 `ComponentSelection`
```python
@dataclass(frozen=True)
class ComponentSelection:
    base_model_id: str
    base_model_root: str
    component_paths: Dict[str, str]
    source_path: str
```
**EN:** This class models `ComponentSelection`.
**CN:** 该类实现 `ComponentSelection`。

### Lines 73-76: Class `ParameterShardContext` / 类 `ParameterShardContext`
```python
@dataclass(frozen=True)
class ParameterShardContext:
    world_size: int
    rank: int
```
**EN:** This class models `ParameterShardContext`.
**CN:** 该类实现 `ParameterShardContext`。

### Lines 79-84: Function `seed_and_broadcast` / 函数 `seed_and_broadcast`
```python
def seed_and_broadcast(seed: int, tensor: torch.Tensor) -> torch.Tensor:
    """Seed and broadcast tensor across ranks for determinism."""
    torch.manual_seed(seed)
    if torch.distributed.is_initialized() and torch.distributed.get_world_size() > 1:
        torch.distributed.broadcast(tensor, src=0)
    return tensor
```
**EN:** This function drives `seed_and_broadcast` with inputs such as `seed`, `tensor`. Seed and broadcast tensor across ranks for determinism.
**CN:** 这个函数负责 `seed_and_broadcast`，主要处理 `seed`, `tensor` 等输入。 文档字符串说明：Seed and broadcast tensor across ranks for determinism.

### Lines 87-91: Function `read_json_file` / 函数 `read_json_file`
```python
def read_json_file(path: str) -> Dict[str, Any]:
    if not os.path.exists(path):
        return {}
    with open(path) as f:
        return json.load(f)
```
**EN:** This function drives `read_json_file` with inputs such as `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `read_json_file`，主要处理 `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 94-102: Function `has_component_files` / 函数 `has_component_files`
```python
def has_component_files(path: str) -> bool:
    if not os.path.isdir(path):
        return False
    if os.path.exists(os.path.join(path, "config.json")):
        return True
    for ext in (".safetensors", ".bin", ".pth"):
        if any(name.endswith(ext) for name in os.listdir(path)):
            return True
    return False
```
**EN:** This function drives `has_component_files` with inputs such as `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `has_component_files`，主要处理 `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 105-112: Function `list_safetensor_files` / 函数 `list_safetensor_files`
```python
def list_safetensor_files(path: str) -> List[str]:
    if not os.path.isdir(path):
        return []
    return sorted(
        os.path.join(path, name)
        for name in os.listdir(path)
        if name.endswith(".safetensors")
    )
```
**EN:** This function drives `list_safetensor_files` with inputs such as `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `list_safetensor_files`，主要处理 `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 115-122: Function `is_text_encoder_config` / 函数 `is_text_encoder_config`
```python
def is_text_encoder_config(path: str) -> bool:
    cfg_path = os.path.join(path, "config.json")
    if not os.path.exists(cfg_path):
        return False
    cfg = read_json_file(cfg_path)
    if cfg.get("model_type") == "i2v" or cfg.get("dim") == I2V_TEXT_ENCODER_DIM:
        return False
    return True
```
**EN:** This function drives `is_text_encoder_config` with inputs such as `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_text_encoder_config`，主要处理 `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 125-135: Function `_resolve_component_subfolder` / 函数 `_resolve_component_subfolder`
```python
def _resolve_component_subfolder(
    model_index: Dict[str, Any], key: str
) -> Optional[str]:
    entry = model_index.get(key)
    if isinstance(entry, dict):
        return entry.get("path") or entry.get("subfolder")
    if isinstance(entry, str):
        return entry
    if entry is not None:
        return key
    return None
```
**EN:** This function drives `_resolve_component_subfolder` with inputs such as `model_index`, `key`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_component_subfolder`，主要处理 `model_index`, `key` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 138-166: Function `resolve_component_path` / 函数 `resolve_component_path`
```python
def resolve_component_path(
    local_root: str, component: ComponentType, model_index_keys: Tuple[str, ...]
) -> str:
    model_index_path = os.path.join(local_root, "model_index.json")
    model_index = read_json_file(model_index_path)

    if model_index:
        for key in model_index_keys:
            subfolder = _resolve_component_subfolder(model_index, key)
            if not subfolder:
                continue
            candidate = os.path.join(local_root, subfolder)
            if not has_component_files(candidate):
                continue
# ...

    raise FileNotFoundError(
        f"Could not resolve {component.value} from model_index.json under {local_root}"
    )
```
**EN:** This function drives `resolve_component_path` with inputs such as `local_root`, `component`, `model_index_keys`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `resolve_component_path`，主要处理 `local_root`, `component`, `model_index_keys` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 169-192: Function `extract_component_path_overrides` / 函数 `extract_component_path_overrides`
```python
def extract_component_path_overrides(extra_args: List[str]) -> Dict[str, str]:
    normalized_args = []
    for arg in extra_args:
        normalized_args.extend(shlex.split(arg))

    component_paths: Dict[str, str] = {}
    index = 0
    while index < len(normalized_args):
        arg = normalized_args[index]
        key_part = arg.split("=", 1)[0] if "=" in arg else arg
        if key_part.startswith("--") and key_part.endswith("-path"):
            component = key_part[2:-5].replace("-", "_")
            if "=" in arg:
                component_paths[component] = arg.split("=", 1)[1]
# ...

    for component, path in component_paths.items():
        component_paths[component] = os.path.expanduser(path)
    return component_paths
```
**EN:** This function drives `extract_component_path_overrides` with inputs such as `extra_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `extract_component_path_overrides`，主要处理 `extra_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 195-211: Function `load_checkpoint_weights` / 函数 `load_checkpoint_weights`
```python
def load_checkpoint_weights(
    module: nn.Module, model_path: str
) -> tuple[list[str], list[str]]:
    safetensors_files = list_safetensor_files(model_path)
    assert safetensors_files, f"Found no safetensors files in {model_path}"

    loaded_state: Dict[str, torch.Tensor] = {}
    for safetensor_path in safetensors_files:
        loaded_state.update(safetensors_load_file(safetensor_path))

    module.load_state_dict(loaded_state, strict=False)

    state_keys = set(module.state_dict().keys())
    loaded_keys = set(loaded_state.keys())
    missing_keys = sorted(state_keys - loaded_keys)
    unexpected_keys = sorted(loaded_keys - state_keys)
    return missing_keys, unexpected_keys
```
**EN:** This function drives `load_checkpoint_weights` with inputs such as `module`, `model_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `load_checkpoint_weights`，主要处理 `module`, `model_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 214-265: Function `select_component_source` / 函数 `select_component_source`
```python
def select_component_source(
    model_id: str,
    extra_args: List[str],
    component: ComponentType,
    model_index_keys: Tuple[str, ...],
) -> ComponentSelection:
    component_paths = extract_component_path_overrides(extra_args)
    force_diffusers_model = resolve_model_overlay_target(model_id) is not None or (
        os.path.exists(model_id)
        and load_overlay_manifest_if_present(model_id) is not None
    )
    base_model_root = maybe_download_model(
        model_id, force_diffusers_model=force_diffusers_model
    )
# ...
        base_model_root=base_model_root,
        component_paths=component_paths,
        source_path=source_path,
    )
```
**EN:** This function drives `select_component_source` with inputs such as `model_id`, `extra_args`, `component`, `model_index_keys`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `select_component_source`，主要处理 `model_id`, `extra_args`, `component`, `model_index_keys` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 268-279: Function `ensure_distributed_env_defaults` / 函数 `ensure_distributed_env_defaults`
```python
def ensure_distributed_env_defaults() -> None:
    if "WORLD_SIZE" in os.environ:
        return
    os.environ.update(
        {
            "MASTER_ADDR": os.getenv("MASTER_ADDR", "127.0.0.1"),
            "MASTER_PORT": os.getenv("MASTER_PORT", "29505"),
            "RANK": "0",
            "LOCAL_RANK": "0",
            "WORLD_SIZE": "1",
        }
    )
```
**EN:** This function drives `ensure_distributed_env_defaults`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `ensure_distributed_env_defaults`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 282-332: Function `initialize_parallel_runtime` / 函数 `initialize_parallel_runtime`
```python
def initialize_parallel_runtime(sgl_args: ServerArgs) -> None:
    tp_size = sgl_args.tp_size
    sp_degree = sgl_args.sp_degree
    ulysses_degree = sgl_args.ulysses_degree
    ring_degree = sgl_args.ring_degree
    dp_size = sgl_args.dp_size
    cfg_degree = sgl_args.cfg_parallel_degree or 1

    if (
        tp_size is None
        or sp_degree is None
        or ulysses_degree is None
        or ring_degree is None
    ):
# ...
        dp_size=dp_size,
    )
    if torch.distributed.is_initialized():
        torch.distributed.barrier()
```
**EN:** This function drives `initialize_parallel_runtime` with inputs such as `sgl_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `initialize_parallel_runtime`，主要处理 `sgl_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 335-371: Function `build_accuracy_server_args` / 函数 `build_accuracy_server_args`
```python
def build_accuracy_server_args(
    base_model_id: str,
    base_model_root: str,
    case: Any,
    component: ComponentType,
    num_gpus: int,
    component_paths: Dict[str, str],
) -> ServerArgs:
    cfg_parallel = bool(case.server_args.cfg_parallel)
    kwargs = {
        "model_path": base_model_root,
        "model_id": base_model_id,
        "num_gpus": num_gpus,
        "trust_remote_code": True,
# ...
    sgl_args.enable_cache_dit = case.server_args.enable_cache_dit
    sgl_args.dit_layerwise_offload = case.server_args.dit_layerwise_offload
    sgl_args.dit_offload_prefetch_size = case.server_args.dit_offload_prefetch_size
    return sgl_args
```
**EN:** This function drives `build_accuracy_server_args` with inputs such as `base_model_id`, `base_model_root`, `case`, `component`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_accuracy_server_args`，主要处理 `base_model_id`, `base_model_root`, `case`, `component` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 374-389: Function `set_module_attr` / 函数 `set_module_attr`
```python
def set_module_attr(module: nn.Module, name: str, value: Any) -> None:
    """Assign to a nested parameter/buffer path such as `blocks.0.attn.to_q.weight`."""
    attrs = name.split(".")
    parent = module
    for attr in attrs[:-1]:
        if hasattr(parent, attr):
            parent = getattr(parent, attr)
        elif isinstance(parent, (nn.ModuleList, nn.Sequential)):
            parent = parent[int(attr)]
        elif isinstance(parent, nn.ModuleDict):
            parent = parent[attr]
        else:
            raise AttributeError(
                f"Cannot resolve {name} on {module.__class__.__name__}"
            )
    setattr(parent, attrs[-1], value)
```
**EN:** This function drives `set_module_attr` with inputs such as `module`, `name`, `value`. Assign to a nested parameter/buffer path such as `blocks.0.attn.to_q.weight`.
**CN:** 这个函数负责 `set_module_attr`，主要处理 `module`, `name`, `value` 等输入。 文档字符串说明：Assign to a nested parameter/buffer path such as `blocks.0.attn.to_q.weight`.

### Lines 392-416: Function `materialize_module` / 函数 `materialize_module`
```python
def materialize_module(
    module: nn.Module, device: torch.device, dtype: torch.dtype
) -> None:
    """Materialize meta tensors and cast floating tensors onto one target device/dtype."""
    for name, param in module.named_parameters():
        if param.device.type == "meta":
            new_data = torch.zeros(param.shape, device=device, dtype=dtype)
            if hasattr(param, "device_mesh") and param.device_mesh is not None:
                new_data = distribute_tensor(
                    new_data, param.device_mesh, param.placements
                )
            set_module_attr(
                module, name, nn.Parameter(new_data, requires_grad=param.requires_grad)
            )
# ...
                new_buf = distribute_tensor(new_buf, buf.device_mesh, buf.placements)
            set_module_attr(module, name, new_buf)
        elif torch.is_floating_point(buf):
            buf.data = buf.data.to(device=device, dtype=dtype)
```
**EN:** This function drives `materialize_module` with inputs such as `module`, `device`, `dtype`. Materialize meta tensors and cast floating tensors onto one target device/dtype.
**CN:** 这个函数负责 `materialize_module`，主要处理 `module`, `device`, `dtype` 等输入。 文档字符串说明：Materialize meta tensors and cast floating tensors onto one target device/dtype.

### Lines 419-443: Function `build_parameter_shard_contexts` / 函数 `build_parameter_shard_contexts`
```python
def build_parameter_shard_contexts(
    module: nn.Module,
) -> Dict[str, ParameterShardContext]:
    """Record TP shard world/rank for each parameter owned by a TP-aware submodule."""
    shard_contexts: Dict[str, ParameterShardContext] = {}
    for module_name, submodule in module.named_modules():
        tp_group = getattr(submodule, "tp_group", None)
        if tp_group is None:
            continue

        context = ParameterShardContext(
            world_size=get_group_size(tp_group),
            rank=get_group_rank(tp_group),
        )
# ...
            qualified_name = f"{module_name}.{name}" if module_name else name
            shard_contexts[qualified_name] = context

    return shard_contexts
```
**EN:** This function drives `build_parameter_shard_contexts` with inputs such as `module`. Record TP shard world/rank for each parameter owned by a TP-aware submodule.
**CN:** 这个函数负责 `build_parameter_shard_contexts`，主要处理 `module` 等输入。 文档字符串说明：Record TP shard world/rank for each parameter owned by a TP-aware submodule.

### Lines 446-454: Function `build_state_lookup` / 函数 `build_state_lookup`
```python
def build_state_lookup(state: Dict[str, torch.Tensor]) -> Dict[str, torch.Tensor]:
    """Index a source state dict under both original and prefix-stripped names."""
    lookup: Dict[str, torch.Tensor] = {}
    for key, val in state.items():
        lookup[key] = val
        for prefix in SOURCE_PREFIXES:
            if key.startswith(prefix):
                lookup[key[len(prefix) :]] = val
    return lookup
```
**EN:** This function drives `build_state_lookup` with inputs such as `state`. Index a source state dict under both original and prefix-stripped names.
**CN:** 这个函数负责 `build_state_lookup`，主要处理 `state` 等输入。 文档字符串说明：Index a source state dict under both original and prefix-stripped names.

### Lines 457-466: Function `normalize_state_key` / 函数 `normalize_state_key`
```python
def normalize_state_key(name: str) -> str:
    """Normalize common naming differences between source and target state dicts."""
    return (
        name.replace("_fsdp_wrapped_module.", "")
        .replace("_orig_mod.", "")
        .replace("gamma", "weight")
        .replace("beta", "bias")
        .replace("scale", "weight")
        .replace("shift", "bias")
    )
```
**EN:** This function drives `normalize_state_key` with inputs such as `name`. Normalize common naming differences between source and target state dicts.
**CN:** 这个函数负责 `normalize_state_key`，主要处理 `name` 等输入。 文档字符串说明：Normalize common naming differences between source and target state dicts.

### Lines 469-479: Function `fuse_qkv` / 函数 `fuse_qkv`
```python
def fuse_qkv(lookup: Dict[str, torch.Tensor], name: str) -> Optional[torch.Tensor]:
    if "qkv_proj" not in name:
        return None
    variants = ["q_proj", "q"]
    for repl in variants:
        q_name = name.replace("qkv_proj", repl)
        k_name = q_name.replace(".q_proj", ".k_proj").replace(".q", ".k")
        v_name = q_name.replace(".q_proj", ".v_proj").replace(".q", ".v")
        if q_name in lookup and k_name in lookup and v_name in lookup:
            return torch.cat([lookup[q_name], lookup[k_name], lookup[v_name]], dim=0)
    return None
```
**EN:** This function drives `fuse_qkv` with inputs such as `lookup`, `name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `fuse_qkv`，主要处理 `lookup`, `name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 482-493: Function `fuse_gate_up_proj` / 函数 `fuse_gate_up_proj`
```python
def fuse_gate_up_proj(
    lookup: Dict[str, torch.Tensor], name: str
) -> Optional[torch.Tensor]:
    if "gate_up_proj" not in name:
        return None

    for gate_token, up_token in (("gate_proj", "up_proj"), ("wi_0", "wi_1")):
        gate_name = name.replace("gate_up_proj", gate_token)
        up_name = name.replace("gate_up_proj", up_token)
        if gate_name in lookup and up_name in lookup:
            return torch.cat([lookup[gate_name], lookup[up_name]], dim=0)
    return None
```
**EN:** This function drives `fuse_gate_up_proj` with inputs such as `lookup`, `name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `fuse_gate_up_proj`，主要处理 `lookup`, `name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 496-524: Function `generate_name_candidates` / 函数 `generate_name_candidates`
```python
def generate_name_candidates(
    name: str, reverse_mapping: Optional[Dict[str, Tuple[str, Any, Any]]]
) -> List[str]:
    candidates: List[str] = []
    clean = normalize_state_key(name)

    for cand in (name, clean):
        if cand not in candidates:
            candidates.append(cand)

    if reverse_mapping:
        for key in (name, clean):
            entry = reverse_mapping.get(key)
            if entry and entry[0] not in candidates:
# ...
        if cand not in candidates:
            candidates.append(cand)

    return candidates
```
**EN:** This function drives `generate_name_candidates` with inputs such as `name`, `reverse_mapping`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `generate_name_candidates`，主要处理 `name`, `reverse_mapping` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 527-577: Function `copy_tensor` / 函数 `copy_tensor`
```python
def copy_tensor(
    dest: torch.Tensor,
    src: torch.Tensor,
    tp_world: int,
    rank: int,
) -> bool:
    if src.numel() == 0:
        return False
    src = src.to(device=dest.device, dtype=dest.dtype)

    if hasattr(dest, "device_mesh") and dest.device_mesh is not None:
        if src.numel() == dest.numel():
            with torch.no_grad():
                dt = distribute_tensor(
# ...
            dest.copy_(src.unsqueeze(2).repeat(1, 1, dest.shape[2], 1, 1))
        return True

    return False
```
**EN:** This function drives `copy_tensor` with inputs such as `dest`, `src`, `tp_world`, `rank`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `copy_tensor`，主要处理 `dest`, `src`, `tp_world`, `rank` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 580-585: Function `_config_to_dict` / 函数 `_config_to_dict`
```python
def _config_to_dict(config: Any) -> Dict[str, Any]:
    to_dict = getattr(config, "to_dict", None)
    if not callable(to_dict):
        return {}
    config_dict = to_dict()
    return config_dict if isinstance(config_dict, dict) else {}
```
**EN:** This function drives `_config_to_dict` with inputs such as `config`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_config_to_dict`，主要处理 `config` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 588-600: Function `resolve_text_encoder_vocab_size` / 函数 `resolve_text_encoder_vocab_size`
```python
def resolve_text_encoder_vocab_size(config: Any) -> int:
    config_dict = _config_to_dict(config)
    vocab_size = config_dict.get("vocab_size")
    if isinstance(vocab_size, int) and vocab_size > 0:
        return vocab_size

    text_config = config_dict.get("text_config")
    if isinstance(text_config, dict):
        nested_vocab_size = text_config.get("vocab_size")
        if isinstance(nested_vocab_size, int) and nested_vocab_size > 0:
            return nested_vocab_size

    return DEFAULT_TEXT_ENCODER_VOCAB_SIZE
```
**EN:** This function drives `resolve_text_encoder_vocab_size` with inputs such as `config`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `resolve_text_encoder_vocab_size`，主要处理 `config` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 603-621: Function `build_deterministic_text_encoder_inputs` / 函数 `build_deterministic_text_encoder_inputs`
```python
def build_deterministic_text_encoder_inputs(
    config: Any, device: str
) -> tuple[torch.Tensor, torch.Tensor]:
    """Build one stable token batch that works across text-encoder implementations."""
    vocab_size = resolve_text_encoder_vocab_size(config)
    max_token_id = max(
        TEXT_ENCODER_TOKEN_MIN + 1, min(vocab_size, TEXT_ENCODER_TOKEN_MAX)
    )

    torch.manual_seed(TEXT_ENCODER_INPUT_SEED)
    input_ids = torch.randint(
        TEXT_ENCODER_TOKEN_MIN,
        max_token_id,
        (1, TEXT_ENCODER_TOKEN_LENGTH),
        device="cpu",
        dtype=torch.long,
    ).to(device)
    attention_mask = torch.ones_like(input_ids)
    return input_ids, attention_mask
```
**EN:** This function drives `build_deterministic_text_encoder_inputs` with inputs such as `config`, `device`. Build one stable token batch that works across text-encoder implementations.
**CN:** 这个函数负责 `build_deterministic_text_encoder_inputs`，主要处理 `config`, `device` 等输入。 文档字符串说明：Build one stable token batch that works across text-encoder implementations.

### Lines 624-626: Function `resolve_text_encoder_forward_module` / 函数 `resolve_text_encoder_forward_module`
```python
def resolve_text_encoder_forward_module(model: nn.Module) -> nn.Module:
    get_encoder = getattr(model, "get_encoder", None)
    return get_encoder() if callable(get_encoder) else model
```
**EN:** This function drives `resolve_text_encoder_forward_module` with inputs such as `model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `resolve_text_encoder_forward_module`，主要处理 `model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 629-638: Function `_module_device` / 函数 `_module_device`
```python
def _module_device(module: nn.Module) -> torch.device:
    param = next(module.parameters(), None)
    if param is not None:
        return param.device

    buf = next(module.buffers(), None)
    if buf is not None:
        return buf.device

    return torch.device("cpu")
```
**EN:** This function drives `_module_device` with inputs such as `module`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_module_device`，主要处理 `module` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 641-675: Function `extract_output_tensor` / 函数 `extract_output_tensor`
```python
def extract_output_tensor(output: Any) -> torch.Tensor:
    """Best-effort extraction of a tensor from model outputs."""
    if isinstance(output, torch.Tensor):
        return output

    sample = getattr(output, "sample", None)
    if sample is not None:
        if isinstance(sample, (list, tuple)):
            sample = sample[0]
        if isinstance(sample, torch.Tensor):
            return sample

    last_hidden_state = getattr(output, "last_hidden_state", None)
    if last_hidden_state is not None:
# ...
        and isinstance(output[0], torch.Tensor)
    ):
        return output[0]
    raise ValueError(f"Could not extract tensor from output of type {type(output)}")
```
**EN:** This function drives `extract_output_tensor` with inputs such as `output`. Best-effort extraction of a tensor from model outputs.
**CN:** 这个函数负责 `extract_output_tensor`，主要处理 `output` 等输入。 文档字符串说明：Best-effort extraction of a tensor from model outputs.

### Lines 678-687: Function `run_text_encoder_accuracy_pair` / 函数 `run_text_encoder_accuracy_pair`
```python
def run_text_encoder_accuracy_pair(
    sgl: nn.Module, ref: nn.Module
) -> tuple[torch.Tensor, torch.Tensor]:
    input_ids, attention_mask = build_deterministic_text_encoder_inputs(
        ref.config, "cpu"
    )
    return (
        _run_single_text_encoder_forward(sgl, input_ids, attention_mask),
        _run_single_text_encoder_forward(ref, input_ids, attention_mask),
    )
```
**EN:** This function drives `run_text_encoder_accuracy_pair` with inputs such as `sgl`, `ref`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `run_text_encoder_accuracy_pair`，主要处理 `sgl`, `ref` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 690-702: Function `_run_single_text_encoder_forward` / 函数 `_run_single_text_encoder_forward`
```python
def _run_single_text_encoder_forward(
    model: nn.Module, input_ids: torch.Tensor, attention_mask: torch.Tensor
) -> torch.Tensor:
    """Run one encoder forward and normalize its output into a tensor."""
    with torch.no_grad():
        forward_model = resolve_text_encoder_forward_module(model)
        model_device = _module_device(forward_model)
        output = forward_model(
            input_ids.to(device=model_device),
            attention_mask=attention_mask.to(device=model_device),
            output_hidden_states=True,
        )
    return extract_output_tensor(output)
```
**EN:** This function drives `_run_single_text_encoder_forward` with inputs such as `model`, `input_ids`, `attention_mask`. Run one encoder forward and normalize its output into a tensor.
**CN:** 这个函数负责 `_run_single_text_encoder_forward`，主要处理 `model`, `input_ids`, `attention_mask` 等输入。 文档字符串说明：Run one encoder forward and normalize its output into a tensor.

### Lines 705-801: Function `_run_staged_native_component_accuracy_case` / 函数 `_run_staged_native_component_accuracy_case`
```python
def _run_staged_native_component_accuracy_case(
    engine_cls: Any,
    case: Any,
    component: ComponentType,
    library: str,
    num_gpus: int,
) -> None:
    from sglang.multimodal_gen.test.server.accuracy_hooks import (
        resolve_component_native_profile,
    )

    sgl = None
    ref = None
    try:
# ...
        if ref is not None:
            del ref
        engine_cls.reset_parallel_runtime()
        engine_cls.clear_memory()
```
**EN:** This function drives `_run_staged_native_component_accuracy_case` with inputs such as `engine_cls`, `case`, `component`, `library`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_staged_native_component_accuracy_case`，主要处理 `engine_cls`, `case`, `component`, `library` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 804-852: Function `_run_staged_text_encoder_accuracy_case` / 函数 `_run_staged_text_encoder_accuracy_case`
```python
def _run_staged_text_encoder_accuracy_case(
    engine_cls: Any, case: Any, num_gpus: int
) -> None:
    sgl = None
    ref = None
    try:
        sgl, ref, device = engine_cls.load_component_pair(
            case,
            ComponentType.TEXT_ENCODER,
            "transformers",
            num_gpus,
            materialize_sgl_on_device=False,
            materialize_ref_on_device=False,
        )
# ...
        if ref is not None:
            del ref
        engine_cls.reset_parallel_runtime()
        engine_cls.clear_memory()
```
**EN:** This function drives `_run_staged_text_encoder_accuracy_case` with inputs such as `engine_cls`, `case`, `num_gpus`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_run_staged_text_encoder_accuracy_case`，主要处理 `engine_cls`, `case`, `num_gpus` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 855-864: Function `run_native_component_accuracy_case` / 函数 `run_native_component_accuracy_case`
```python
def run_native_component_accuracy_case(
    engine_cls: Any,
    case: Any,
    component: ComponentType,
    library: str,
    num_gpus: int,
) -> None:
    _run_staged_native_component_accuracy_case(
        engine_cls, case, component, library, num_gpus
    )
```
**EN:** This function drives `run_native_component_accuracy_case` with inputs such as `engine_cls`, `case`, `component`, `library`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `run_native_component_accuracy_case`，主要处理 `engine_cls`, `case`, `component`, `library` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 867-868: Function `run_text_encoder_accuracy_case` / 函数 `run_text_encoder_accuracy_case`
```python
def run_text_encoder_accuracy_case(engine_cls: Any, case: Any, num_gpus: int) -> None:
    _run_staged_text_encoder_accuracy_case(engine_cls, case, num_gpus)
```
**EN:** This function drives `run_text_encoder_accuracy_case` with inputs such as `engine_cls`, `case`, `num_gpus`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `run_text_encoder_accuracy_case`，主要处理 `engine_cls`, `case`, `num_gpus` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Model/component loading / 模型/组件加载
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.utils`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.model_overlay`, `sglang.multimodal_gen.test.server.accuracy_config`, `sglang.multimodal_gen.test.server.accuracy_hooks`, `sglang.multimodal_gen`
- **External / 外部**: `__future__`, `torch`, `torch.nn`, `safetensors.torch`, `torch.distributed.tensor`
- **Stdlib / 标准库**: `json`, `os`, `shlex`, `contextlib`, `dataclasses`, `typing`
