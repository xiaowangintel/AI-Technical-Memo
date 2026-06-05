# build_modelopt_fp8_transformer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/tools/build_modelopt_fp8_transformer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This script provides developer tooling for build modelopt fp8 transformer, typically for conversion, inspection, or offline preparation. Key symbols include `_resolve_transformer_dir`, `_resolve_backbone_ckpt`, `_find_index_file`. / 该脚本提供与 build modelopt fp8 transformer 相关的开发工具，通常用于转换、检查或离线准备。 关键符号包括 `_resolve_transformer_dir`, `_resolve_backbone_ckpt`, `_find_index_file`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-209: Imports and module setup / 导入与模块初始化
```python
"""Build an SGLang-loadable ModelOpt FP8 diffusion transformer.

The core conversion path is model-agnostic:
- read the ModelOpt diffusers transformer export
- rebuild per-layer `weight_scale` / `input_scale` tensors from `backbone.pt`
- materialize SGLang-native `float8_e4m3fn` weights
- preserve ModelOpt `ignore` layers in their original dtype

Some models still benefit from a small validated BF16 fallback set. Those
fallback profiles are intentionally isolated so the generic FP8 conversion path
remains reusable across future diffusion backbones.

Example:

# ...
    r"^proj_out$",
    r"^transformer_blocks\.\d+\.img_mlp\.net\.2$",
    r"^transformer_blocks\.\d+\.(img_mod|txt_mod)$",
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 212-219: Function `_resolve_transformer_dir` / 函数 `_resolve_transformer_dir`
```python
def _resolve_transformer_dir(path: str) -> str:
    candidate = Path(path).expanduser().resolve()
    if (candidate / "config.json").is_file():
        return str(candidate)
    transformer_dir = candidate / "transformer"
    if (transformer_dir / "config.json").is_file():
        return str(transformer_dir)
    raise FileNotFoundError(f"Could not resolve a transformer directory from: {path}")
```
**EN:** This function drives `_resolve_transformer_dir` with inputs such as `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_transformer_dir`，主要处理 `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 222-229: Function `_resolve_backbone_ckpt` / 函数 `_resolve_backbone_ckpt`
```python
def _resolve_backbone_ckpt(path: str) -> str:
    candidate = Path(path).expanduser().resolve()
    if candidate.is_file():
        return str(candidate)
    backbone_path = candidate / "backbone.pt"
    if backbone_path.is_file():
        return str(backbone_path)
    raise FileNotFoundError(f"Could not resolve backbone.pt from: {path}")
```
**EN:** This function drives `_resolve_backbone_ckpt` with inputs such as `path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_backbone_ckpt`，主要处理 `path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 232-243: Function `_find_index_file` / 函数 `_find_index_file`
```python
def _find_index_file(model_dir: str) -> str | None:
    for filename in INDEX_FILENAMES:
        candidate = os.path.join(model_dir, filename)
        if os.path.isfile(candidate):
            return filename

    matches = sorted(
        filename
        for filename in os.listdir(model_dir)
        if filename.endswith(".safetensors.index.json")
    )
    return matches[0] if matches else None
```
**EN:** This function drives `_find_index_file` with inputs such as `model_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_find_index_file`，主要处理 `model_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 246-270: Function `_load_weight_map` / 函数 `_load_weight_map`
```python
def _load_weight_map(model_dir: str) -> tuple[dict[str, str], str | None]:
    index_filename = _find_index_file(model_dir)
    if index_filename is not None:
        with open(os.path.join(model_dir, index_filename), encoding="utf-8") as f:
            index_data = json.load(f)
        return dict(index_data["weight_map"]), index_filename

    safetensors_files = sorted(
        filename
        for filename in os.listdir(model_dir)
        if filename.endswith(".safetensors")
    )
    if len(safetensors_files) != 1:
        raise ValueError(
# ...
    ) as f:
        weight_map = {key: shard_name for key in f.keys()}
    index_filename = f"{Path(shard_name).stem}.safetensors.index.json"
    return weight_map, index_filename
```
**EN:** This function drives `_load_weight_map` with inputs such as `model_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_load_weight_map`，主要处理 `model_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 273-276: Function `_load_config` / 函数 `_load_config`
```python
def _load_config(model_dir: str) -> dict:
    config_path = os.path.join(model_dir, "config.json")
    with open(config_path, encoding="utf-8") as f:
        return json.load(f)
```
**EN:** This function drives `_load_config` with inputs such as `model_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_load_config`，主要处理 `model_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 279-288: Function `_load_first_shard_metadata` / 函数 `_load_first_shard_metadata`
```python
def _load_first_shard_metadata(
    model_dir: str, weight_map: Mapping[str, str]
) -> dict[str, str]:
    if not weight_map:
        return {}
    first_shard = next(iter(weight_map.values()))
    with safe_open(
        os.path.join(model_dir, first_shard), framework="pt", device="cpu"
    ) as f:
        return dict(f.metadata() or {})
```
**EN:** This function drives `_load_first_shard_metadata` with inputs such as `model_dir`, `weight_map`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_load_first_shard_metadata`，主要处理 `model_dir`, `weight_map` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 291-297: Function `_map_hunyuanvideo_runtime_module_name` / 函数 `_map_hunyuanvideo_runtime_module_name`
```python
def _map_hunyuanvideo_runtime_module_name(module_name: str) -> list[str]:
    mapped_names: list[str] = []
    for pattern, replacement in HUNYUANVIDEO_RUNTIME_NAME_REPLACEMENTS:
        mapped = re.sub(pattern, replacement, module_name)
        if mapped != module_name:
            mapped_names.append(mapped)
    return mapped_names
```
**EN:** This function drives `_map_hunyuanvideo_runtime_module_name` with inputs such as `module_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_map_hunyuanvideo_runtime_module_name`，主要处理 `module_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 300-305: Function `_get_runtime_module_name_mapper` / 函数 `_get_runtime_module_name_mapper`
```python
def _get_runtime_module_name_mapper(
    *, model_type: str, class_name: str | None
) -> Callable[[str], list[str]] | None:
    if model_type == "hunyuan-video" or class_name == "HunyuanVideoTransformer3DModel":
        return _map_hunyuanvideo_runtime_module_name
    return None
```
**EN:** This function drives `_get_runtime_module_name_mapper` with inputs such as `model_type`, `class_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_get_runtime_module_name_mapper`，主要处理 `model_type`, `class_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 308-339: Function `_module_name_variants` / 函数 `_module_name_variants`
```python
def _module_name_variants(
    weight_name: str,
    runtime_name_mapper: Callable[[str], list[str]] | None = None,
) -> list[str]:
    module_name = weight_name[:-7] if weight_name.endswith(".weight") else weight_name
    variants = [module_name]

    for prefix in ("model.diffusion_model.", "velocity_model."):
        if module_name.startswith(prefix):
            variants.append(module_name[len(prefix) :])

    canonicalized: list[str] = []
    for variant in variants:
        canonicalized.append(
# ...
    for variant in variants:
        if variant not in deduped:
            deduped.append(variant)
    return deduped
```
**EN:** This function drives `_module_name_variants` with inputs such as `weight_name`, `runtime_name_mapper`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_module_name_variants`，主要处理 `weight_name`, `runtime_name_mapper` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 342-346: Function `_preferred_module_name` / 函数 `_preferred_module_name`
```python
def _preferred_module_name(
    weight_name: str,
    runtime_name_mapper: Callable[[str], list[str]] | None = None,
) -> str:
    return _module_name_variants(weight_name, runtime_name_mapper)[-1]
```
**EN:** This function drives `_preferred_module_name` with inputs such as `weight_name`, `runtime_name_mapper`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_preferred_module_name`，主要处理 `weight_name`, `runtime_name_mapper` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 349-355: Function `_scale_key_candidates` / 函数 `_scale_key_candidates`
```python
def _scale_key_candidates(weight_name: str) -> list[str]:
    candidates = [weight_name]
    if weight_name.startswith("model.diffusion_model."):
        candidates.append(
            "velocity_model." + weight_name[len("model.diffusion_model.") :]
        )
    return candidates
```
**EN:** This function drives `_scale_key_candidates` with inputs such as `weight_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_scale_key_candidates`，主要处理 `weight_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 358-365: Function `_resolve_scale_key` / 函数 `_resolve_scale_key`
```python
def _resolve_scale_key(
    weight_name: str,
    scale_map: Mapping[str, Mapping[str, torch.Tensor]],
) -> str | None:
    for candidate in _scale_key_candidates(weight_name):
        if candidate in scale_map:
            return candidate
    return None
```
**EN:** This function drives `_resolve_scale_key` with inputs such as `weight_name`, `scale_map`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_scale_key`，主要处理 `weight_name`, `scale_map` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 368-382: Function `_is_ltx2_x0_export` / 函数 `_is_ltx2_x0_export`
```python
def _is_ltx2_x0_export(
    *,
    config: Mapping[str, object],
    source_metadata: Mapping[str, str],
    source_weight_map: Mapping[str, str],
) -> bool:
    if config.get("_class_name") != "X0Model":
        return False
    if not any(name.startswith("model.diffusion_model.") for name in source_weight_map):
        return False
    try:
        metadata_config = json.loads(str(source_metadata.get("config", "")))
    except json.JSONDecodeError:
        return False
    return isinstance(metadata_config.get("transformer"), dict)
```
**EN:** This function drives `_is_ltx2_x0_export` with inputs such as `config`, `source_metadata`, `source_weight_map`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_is_ltx2_x0_export`，主要处理 `config`, `source_metadata`, `source_weight_map` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 385-400: Function `_build_output_config` / 函数 `_build_output_config`
```python
def _build_output_config(
    *,
    source_config: Mapping[str, object],
    source_metadata: Mapping[str, str],
    quant_config: Mapping[str, object],
    is_ltx2_x0_export: bool,
) -> dict[str, object]:
    if is_ltx2_x0_export:
        metadata_config = json.loads(str(source_metadata["config"]))
        output_config = dict(metadata_config["transformer"])
        output_config["_class_name"] = "LTX2VideoTransformer3DModel"
    else:
        output_config = dict(source_config)

    output_config["quantization_config"] = dict(quant_config)
    return output_config
```
**EN:** This function drives `_build_output_config` with inputs such as `source_config`, `source_metadata`, `quant_config`, `is_ltx2_x0_export`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_build_output_config`，主要处理 `source_config`, `source_metadata`, `quant_config`, `is_ltx2_x0_export` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 403-410: Function `_should_keep_ltx2_transformer_key` / 函数 `_should_keep_ltx2_transformer_key`
```python
def _should_keep_ltx2_transformer_key(weight_name: str) -> bool:
    if not weight_name.startswith("model.diffusion_model."):
        return False
    connector_prefixes = (
        "model.diffusion_model.audio_embeddings_connector.",
        "model.diffusion_model.video_embeddings_connector.",
    )
    return not weight_name.startswith(connector_prefixes)
```
**EN:** This function drives `_should_keep_ltx2_transformer_key` with inputs such as `weight_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_should_keep_ltx2_transformer_key`，主要处理 `weight_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 413-436: Function `get_default_keep_bf16_patterns` / 函数 `get_default_keep_bf16_patterns`
```python
def get_default_keep_bf16_patterns(
    *, model_type: str, class_name: str | None
) -> list[str]:
    if model_type == "ltx2":
        return list(DEFAULT_LTX2_KEEP_BF16_PATTERNS)
    if model_type == "flux1":
        return list(DEFAULT_FLUX1_KEEP_BF16_PATTERNS)
    if model_type == "flux2":
        return list(DEFAULT_FLUX2_KEEP_BF16_PATTERNS)
    if model_type == "hunyuan-video":
        return list(DEFAULT_HUNYUANVIDEO_KEEP_BF16_PATTERNS)
    if model_type == "qwen-image":
        return list(DEFAULT_QWEN_IMAGE_KEEP_BF16_PATTERNS)
    if model_type == "none":
# ...
        return list(DEFAULT_HUNYUANVIDEO_KEEP_BF16_PATTERNS)
    if class_name == "QwenImageTransformer2DModel":
        return list(DEFAULT_QWEN_IMAGE_KEEP_BF16_PATTERNS)
    return []
```
**EN:** This function drives `get_default_keep_bf16_patterns` with inputs such as `model_type`, `class_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_default_keep_bf16_patterns`，主要处理 `model_type`, `class_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 439-451: Function `should_keep_bf16` / 函数 `should_keep_bf16`
```python
def should_keep_bf16(
    weight_name: str,
    keep_bf16_patterns: Sequence[str],
    runtime_name_mapper: Callable[[str], list[str]] | None = None,
) -> bool:
    if not keep_bf16_patterns:
        return False

    return any(
        re.search(pattern, module_name)
        for pattern in keep_bf16_patterns
        for module_name in _module_name_variants(weight_name, runtime_name_mapper)
    )
```
**EN:** This function drives `should_keep_bf16` with inputs such as `weight_name`, `keep_bf16_patterns`, `runtime_name_mapper`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `should_keep_bf16`，主要处理 `weight_name`, `keep_bf16_patterns`, `runtime_name_mapper` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 454-469: Function `is_ignored_by_modelopt` / 函数 `is_ignored_by_modelopt`
```python
def is_ignored_by_modelopt(
    weight_name: str,
    ignore_patterns: Sequence[str],
    runtime_name_mapper: Callable[[str], list[str]] | None = None,
) -> bool:
    if not ignore_patterns:
        return False

    for pattern in ignore_patterns:
        regex_str = pattern.replace(".", r"\.").replace("*", r".*")
        if any(
            re.fullmatch(regex_str, module_name)
            for module_name in _module_name_variants(weight_name, runtime_name_mapper)
        ):
            return True
    return False
```
**EN:** This function drives `is_ignored_by_modelopt` with inputs such as `weight_name`, `ignore_patterns`, `runtime_name_mapper`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_ignored_by_modelopt`，主要处理 `weight_name`, `ignore_patterns`, `runtime_name_mapper` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 472-494: Function `build_fp8_scale_map` / 函数 `build_fp8_scale_map`
```python
def build_fp8_scale_map(
    model_state_dict: Mapping[str, torch.Tensor],
    *,
    maxbound: float = FP8_E4M3_MAXBOUND,
) -> dict[str, dict[str, torch.Tensor]]:
    scale_map: dict[str, dict[str, torch.Tensor]] = {}
    for key, value in model_state_dict.items():
        if key.endswith(".weight_quantizer._amax"):
            layer_name = key[: -len(".weight_quantizer._amax")]
            scale_map.setdefault(f"{layer_name}.weight", {})["weight_scale"] = (
                value.detach().to(torch.float32).reshape(1).cpu() / maxbound
            )
        elif key.endswith(".input_quantizer._amax"):
            layer_name = key[: -len(".input_quantizer._amax")]
# ...
        weight_name: scale_tensors
        for weight_name, scale_tensors in scale_map.items()
        if {"weight_scale", "input_scale"} <= set(scale_tensors)
    }
```
**EN:** This function drives `build_fp8_scale_map` with inputs such as `model_state_dict`, `maxbound`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_fp8_scale_map`，主要处理 `model_state_dict`, `maxbound` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 497-512: Function `quantize_fp8_weight` / 函数 `quantize_fp8_weight`
```python
def quantize_fp8_weight(
    weight: torch.Tensor,
    weight_scale: torch.Tensor,
) -> torch.Tensor:
    if weight.dtype == torch.float8_e4m3fn:
        return weight.contiguous()

    scale = weight_scale.to(weight.device, dtype=torch.float32)
    if scale.numel() != 1:
        raise ValueError(
            "Only per-tensor FP8 scales are supported for diffusion checkpoints, "
            f"got shape {tuple(scale.shape)}."
        )

    quantized = (weight.to(torch.float32) / scale.reshape(1)).to(torch.float8_e4m3fn)
    return quantized.cpu().contiguous()
```
**EN:** This function drives `quantize_fp8_weight` with inputs such as `weight`, `weight_scale`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `quantize_fp8_weight`，主要处理 `weight`, `weight_scale` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 515-525: Function `_copy_non_shard_files` / 函数 `_copy_non_shard_files`
```python
def _copy_non_shard_files(source_dir: str, output_dir: str) -> None:
    ignored = set(INDEX_FILENAMES)
    for entry in os.listdir(source_dir):
        if entry.endswith(".safetensors") or entry in ignored:
            continue
        source_path = os.path.join(source_dir, entry)
        output_path = os.path.join(output_dir, entry)
        if os.path.isdir(source_path):
            shutil.copytree(source_path, output_path, dirs_exist_ok=True)
        else:
            shutil.copy2(source_path, output_path)
```
**EN:** This function drives `_copy_non_shard_files` with inputs such as `source_dir`, `output_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_copy_non_shard_files`，主要处理 `source_dir`, `output_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 528-543: Function `_load_selected_tensors` / 函数 `_load_selected_tensors`
```python
def _load_selected_tensors(
    model_dir: str,
    weight_map: Mapping[str, str],
    tensor_names: Iterable[str],
) -> dict[str, torch.Tensor]:
    tensors: dict[str, torch.Tensor] = {}
    names_by_file: dict[str, list[str]] = defaultdict(list)
    for name in tensor_names:
        names_by_file[weight_map[name]].append(name)

    for filename, names in names_by_file.items():
        shard_path = os.path.join(model_dir, filename)
        with safe_open(shard_path, framework="pt", device="cpu") as f:
            for name in names:
                tensors[name] = f.get_tensor(name).contiguous()
    return tensors
```
**EN:** This function drives `_load_selected_tensors` with inputs such as `model_dir`, `weight_map`, `tensor_names`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_load_selected_tensors`，主要处理 `model_dir`, `weight_map`, `tensor_names` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 546-792: Function `build_modelopt_fp8_transformer` / 函数 `build_modelopt_fp8_transformer`
```python
def build_modelopt_fp8_transformer(
    *,
    modelopt_hf_dir: str,
    modelopt_backbone_ckpt: str,
    output_dir: str,
    base_transformer_dir: str | None = None,
    model_type: str = "auto",
    keep_bf16_patterns: Sequence[str] | None = None,
    maxbound: float = FP8_E4M3_MAXBOUND,
    overwrite: bool = False,
) -> dict[str, int]:
    source_dir = _resolve_transformer_dir(modelopt_hf_dir)
    backbone_ckpt_path = _resolve_backbone_ckpt(modelopt_backbone_ckpt)
    base_dir = (
# ...
        "preserved_ignored_weights": preserved_ignored_weight_count,
        "added_scale_tensors": added_scale_count,
        "output_shards": len(weights_by_file),
    }
```
**EN:** This function drives `build_modelopt_fp8_transformer` with inputs such as `modelopt_hf_dir`, `modelopt_backbone_ckpt`, `output_dir`, `base_transformer_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_modelopt_fp8_transformer`，主要处理 `modelopt_hf_dir`, `modelopt_backbone_ckpt`, `output_dir`, `base_transformer_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 795-864: Function `_parse_args` / 函数 `_parse_args`
```python
def _parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description=(
            "Build an SGLang-loadable ModelOpt FP8 diffusion transformer from a "
            "ModelOpt diffusers export."
        )
    )
    parser.add_argument(
        "--modelopt-hf-dir",
        required=True,
        help="ModelOpt --hf-ckpt-dir output, or its transformer subdirectory.",
    )
    parser.add_argument(
        "--modelopt-backbone-ckpt",
# ...
        action="store_true",
        help="Replace --output-dir if it already exists.",
    )
    return parser.parse_args()
```
**EN:** This function drives `_parse_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_parse_args`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 867-879: Function `main` / 函数 `main`
```python
def main() -> None:
    args = _parse_args()
    stats = build_modelopt_fp8_transformer(
        modelopt_hf_dir=args.modelopt_hf_dir,
        modelopt_backbone_ckpt=args.modelopt_backbone_ckpt,
        output_dir=args.output_dir,
        base_transformer_dir=args.base_transformer_dir,
        model_type=args.model_type,
        keep_bf16_patterns=args.keep_bf16_pattern,
        maxbound=args.maxbound,
        overwrite=args.overwrite,
    )
    print(json.dumps(stats, indent=2, sort_keys=True))
```
**EN:** This function drives `main`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `main`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 880-883: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    main()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Quantization workflow / 量化工作流
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Audio generation flow / 音频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.quantization_utils`
- **External / 外部**: `__future__`, `torch`, `safetensors`, `safetensors.torch`
- **Stdlib / 标准库**: `argparse`, `gc`, `json`, `os`, `re`, `shutil`, `collections`, `pathlib`
