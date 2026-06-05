# build_modelopt_nvfp4_transformer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/tools/build_modelopt_nvfp4_transformer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This script provides developer tooling for build modelopt nvfp4 transformer, typically for conversion, inspection, or offline preparation. Key symbols include `_resolve_transformer_dir`, `_find_index_file`, `_load_weight_map`. / 该脚本提供与 build modelopt nvfp4 transformer 相关的开发工具，通常用于转换、检查或离线准备。 关键符号包括 `_resolve_transformer_dir`, `_find_index_file`, `_load_weight_map`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-49: Imports and module setup / 导入与模块初始化
```python
"""Build an SGLang-loadable ModelOpt NVFP4 diffusion transformer.

This tool keeps the ModelOpt-exported NVFP4 tensors for most transformer
modules, but can replace a validated subset of numerically sensitive modules
with their original BF16 tensors from the base transformer checkpoint.

It is primarily intended for FLUX.1-dev style ModelOpt NVFP4 exports where:
- the base pipeline should remain separate from the quantized transformer
- fallback BF16 modules are model-family specific
- the serialized FP4 weight byte order may already match the runtime kernel
"""

from __future__ import annotations

# ...
    ".input_scale",
    ".weight",
    ".bias",
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 52-59: Function `_resolve_transformer_dir` / 函数 `_resolve_transformer_dir`
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

### Lines 62-73: Function `_find_index_file` / 函数 `_find_index_file`
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

### Lines 76-100: Function `_load_weight_map` / 函数 `_load_weight_map`
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

### Lines 103-106: Function `_load_config` / 函数 `_load_config`
```python
def _load_config(model_dir: str) -> dict:
    config_path = os.path.join(model_dir, "config.json")
    with open(config_path, encoding="utf-8") as f:
        return json.load(f)
```
**EN:** This function drives `_load_config` with inputs such as `model_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_load_config`，主要处理 `model_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 109-112: Function `_write_config` / 函数 `_write_config`
```python
def _write_config(model_dir: Path, config: Mapping[str, object]) -> None:
    with open(model_dir / "config.json", "w", encoding="utf-8") as f:
        json.dump(config, f, indent=2, sort_keys=True)
        f.write("\n")
```
**EN:** This function drives `_write_config` with inputs such as `model_dir`, `config`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_write_config`，主要处理 `model_dir`, `config` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 115-125: Function `_copy_non_shard_files` / 函数 `_copy_non_shard_files`
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

### Lines 128-143: Function `_load_selected_tensors` / 函数 `_load_selected_tensors`
```python
def _load_selected_tensors(
    model_dir: str,
    weight_map: Mapping[str, str],
    tensor_names: Iterable[str],
):
    tensors = {}
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

### Lines 146-150: Function `_module_name_for_tensor` / 函数 `_module_name_for_tensor`
```python
def _module_name_for_tensor(tensor_name: str) -> str:
    for suffix in _TENSOR_MODULE_SUFFIXES:
        if tensor_name.endswith(suffix):
            return tensor_name[: -len(suffix)]
    return tensor_name
```
**EN:** This function drives `_module_name_for_tensor` with inputs such as `tensor_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_module_name_for_tensor`，主要处理 `tensor_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 153-160: Function `_matches_any_pattern` / 函数 `_matches_any_pattern`
```python
def _matches_any_pattern(module_name: str, patterns: Sequence[str]) -> bool:
    if not patterns:
        return False
    for pattern in patterns:
        regex_str = pattern.replace(".", r"\.").replace("*", r".*")
        if re.fullmatch(regex_str, module_name):
            return True
    return False
```
**EN:** This function drives `_matches_any_pattern` with inputs such as `module_name`, `patterns`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_matches_any_pattern`，主要处理 `module_name`, `patterns` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 163-168: Function `_preset_patterns` / 函数 `_preset_patterns`
```python
def _preset_patterns(pattern_preset: str) -> list[str]:
    if pattern_preset == "none":
        return []
    if pattern_preset == "flux1-nvfp4":
        return list(DEFAULT_FLUX1_NVFP4_FALLBACK_PATTERNS)
    raise ValueError(f"Unsupported pattern preset: {pattern_preset}")
```
**EN:** This function drives `_preset_patterns` with inputs such as `pattern_preset`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_preset_patterns`，主要处理 `pattern_preset` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 171-200: Function `_updated_quant_config` / 函数 `_updated_quant_config`
```python
def _updated_quant_config(
    source_config: Mapping[str, object],
    *,
    fallback_patterns: Sequence[str],
    swap_weight_nibbles: bool,
) -> dict[str, object]:
    output_config = json.loads(json.dumps(source_config))
    quant_config = output_config.get("quantization_config")
    if not isinstance(quant_config, dict):
        raise ValueError("Expected a flat quantization_config dict in config.json.")
    if (
        quant_config.get("quant_method") != "modelopt"
        or "FP4" not in str(quant_config.get("quant_algo", "")).upper()
    ):
# ...
        "quant_type", str(quant_config.get("quant_algo", "")).upper()
    )
    quant_config["swap_weight_nibbles"] = swap_weight_nibbles
    return output_config
```
**EN:** This function drives `_updated_quant_config` with inputs such as `source_config`, `fallback_patterns`, `swap_weight_nibbles`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_updated_quant_config`，主要处理 `source_config`, `fallback_patterns`, `swap_weight_nibbles` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 203-330: Function `build_modelopt_nvfp4_transformer` / 函数 `build_modelopt_nvfp4_transformer`
```python
def build_modelopt_nvfp4_transformer(
    *,
    base_transformer_dir: str,
    modelopt_hf_dir: str,
    output_dir: str,
    pattern_preset: str = "none",
    keep_bf16_patterns: Sequence[str] | None = None,
    swap_weight_nibbles: bool | None = None,
    overwrite: bool = False,
) -> dict[str, int | bool]:
    source_dir = _resolve_transformer_dir(modelopt_hf_dir)
    base_dir = _resolve_transformer_dir(base_transformer_dir)

    patterns = _preset_patterns(pattern_preset)
# ...
        "removed_aux_tensors": removed_aux_tensor_count,
        "output_shards": len(weights_by_file),
        "swap_weight_nibbles": resolved_swap_weight_nibbles,
    }
```
**EN:** This function drives `build_modelopt_nvfp4_transformer` with inputs such as `base_transformer_dir`, `modelopt_hf_dir`, `output_dir`, `pattern_preset`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_modelopt_nvfp4_transformer`，主要处理 `base_transformer_dir`, `modelopt_hf_dir`, `output_dir`, `pattern_preset` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 333-384: Function `_parse_args` / 函数 `_parse_args`
```python
def _parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description=(
            "Build an SGLang-loadable ModelOpt NVFP4 diffusion transformer and "
            "optionally keep selected modules in BF16."
        )
    )
    parser.add_argument(
        "--base-transformer-dir",
        required=True,
        help="Original BF16 transformer directory, or a parent model directory.",
    )
    parser.add_argument(
        "--modelopt-hf-dir",
# ...
        action="store_true",
        help="Replace --output-dir if it already exists.",
    )
    return parser.parse_args()
```
**EN:** This function drives `_parse_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_parse_args`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 387-398: Function `main` / 函数 `main`
```python
def main() -> None:
    args = _parse_args()
    stats = build_modelopt_nvfp4_transformer(
        base_transformer_dir=args.base_transformer_dir,
        modelopt_hf_dir=args.modelopt_hf_dir,
        output_dir=args.output_dir,
        pattern_preset=args.pattern_preset,
        keep_bf16_patterns=args.keep_bf16_pattern,
        swap_weight_nibbles=args.swap_weight_nibbles,
        overwrite=args.overwrite,
    )
    print(json.dumps(stats, indent=2, sort_keys=True))
```
**EN:** This function drives `main`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `main`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 399-402: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    main()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Quantization workflow / 量化工作流
- Pipeline orchestration / 流水线编排
- Symbol `_resolve_transformer_dir` anchors the module API / 符号 `_resolve_transformer_dir` 构成该模块的核心 API
- Symbol `_find_index_file` anchors the module API / 符号 `_find_index_file` 构成该模块的核心 API

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `safetensors`, `safetensors.torch`
- **Stdlib / 标准库**: `argparse`, `json`, `os`, `re`, `shutil`, `collections`, `pathlib`, `typing`
