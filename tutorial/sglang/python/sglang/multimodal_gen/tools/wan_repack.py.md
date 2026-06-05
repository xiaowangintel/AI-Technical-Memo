# wan_repack.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/tools/wan_repack.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This script provides developer tooling for wan repack, typically for conversion, inspection, or offline preparation. Key symbols include `get_transformer_config`, `get_transformer_dirs`, `get_quant_subpath`. / 该脚本提供与 wan repack 相关的开发工具，通常用于转换、检查或离线准备。 关键符号包括 `get_transformer_config`, `get_transformer_dirs`, `get_quant_subpath`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Imports and module setup / 导入与模块初始化
```python
### Based on https://github.com/huggingface/diffusers/blob/main/scripts/convert_wan_to_diffusers.py

import argparse
import json
import pathlib
import shutil
from typing import Any, Dict, List

from safetensors.torch import load_file, save_file

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)

# ...
SUPPORTED_MODEL_TYPES = ["Wan2.2-T2V-A14B", "Wan2.2-I2V-A14B", "Wan2.2-TI2V-5B"]

# Cascade models have two transformers (high_noise + low_noise)
CASCADE_MODEL_TYPES = {"Wan2.2-T2V-A14B", "Wan2.2-I2V-A14B"}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 63-69: Function `get_transformer_config` / 函数 `get_transformer_config`
```python
def get_transformer_config(model_type: str) -> Dict[str, Any]:
    if model_type in SUPPORTED_MODEL_TYPES:
        return TRANSFORMER_KEYS_RENAME_DICT
    else:
        raise ValueError(
            f"Unsupported model_type: {model_type}. Supported: {SUPPORTED_MODEL_TYPES}"
        )
```
**EN:** This function drives `get_transformer_config` with inputs such as `model_type`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_transformer_config`，主要处理 `model_type` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 72-76: Function `get_transformer_dirs` / 函数 `get_transformer_dirs`
```python
def get_transformer_dirs(model_type: str) -> List[str]:
    """Return the list of transformer directory names for a given model type."""
    if model_type in CASCADE_MODEL_TYPES:
        return ["transformer", "transformer_2"]
    return ["transformer"]
```
**EN:** This function drives `get_transformer_dirs` with inputs such as `model_type`. Return the list of transformer directory names for a given model type.
**CN:** 这个函数负责 `get_transformer_dirs`，主要处理 `model_type` 等输入。 文档字符串说明：Return the list of transformer directory names for a given model type.

### Lines 79-90: Function `get_quant_subpath` / 函数 `get_quant_subpath`
```python
def get_quant_subpath(
    model_type: str, quant_path: pathlib.Path, transformer_dir: str
) -> pathlib.Path:
    """Return the quant weights subdirectory for a given transformer."""
    if model_type in CASCADE_MODEL_TYPES:
        sub = (
            "high_noise_model"
            if transformer_dir == "transformer"
            else "low_noise_model"
        )
        return quant_path / sub
    return quant_path
```
**EN:** This function drives `get_quant_subpath` with inputs such as `model_type`, `quant_path`, `transformer_dir`. Return the quant weights subdirectory for a given transformer.
**CN:** 这个函数负责 `get_quant_subpath`，主要处理 `model_type`, `quant_path`, `transformer_dir` 等输入。 文档字符串说明：Return the quant weights subdirectory for a given transformer.

### Lines 93-94: Function `update_dict_` / 函数 `update_dict_`
```python
def update_dict_(d: Dict[str, Any], old_key: str, new_key: str) -> None:
    d[new_key] = d.pop(old_key)
```
**EN:** This function drives `update_dict_` with inputs such as `d`, `old_key`, `new_key`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `update_dict_`，主要处理 `d`, `old_key`, `new_key` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 97-108: Function `load_sharded_safetensors` / 函数 `load_sharded_safetensors`
```python
def load_sharded_safetensors(directory: pathlib.Path, pattern: str) -> dict:
    candidates = sorted(directory.glob(pattern))
    if not candidates:
        raise FileNotFoundError(f"No file matching '{pattern}' found in {directory}")
    if len(candidates) > 1:
        raise FileNotFoundError(
            f"Multiple files matching '{pattern}' found in {directory}: {candidates}"
        )

    state_dict = {}
    state_dict.update(load_file(candidates[0]))
    return state_dict
```
**EN:** This function drives `load_sharded_safetensors` with inputs such as `directory`, `pattern`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `load_sharded_safetensors`，主要处理 `directory`, `pattern` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 111-143: Function `convert_transformer` / 函数 `convert_transformer`
```python
def convert_transformer(
    model_type: str, model_dir: pathlib.Path, output_dir: pathlib.Path
) -> None:
    """Convert a single quantized transformer directory into Diffusers format."""
    model_path = pathlib.Path(model_dir)
    out_path = pathlib.Path(output_dir)
    out_path.mkdir(parents=True, exist_ok=True)
    RENAME_DICT = get_transformer_config(model_type)

    state_dict = load_sharded_safetensors(model_path, "quant_model_weight*.safetensors")

    json_candidates = sorted(model_path.glob("quant_model_description*.json"))
    if not json_candidates:
        raise FileNotFoundError(
# ...
    save_file(state_dict, out_path / "diffusion_pytorch_model.safetensors")

    with open(out_path / "quant_model_description.json", "w") as f:
        json.dump(quant_config, f, indent=2)
```
**EN:** This function drives `convert_transformer` with inputs such as `model_type`, `model_dir`, `output_dir`. Convert a single quantized transformer directory into Diffusers format.
**CN:** 这个函数负责 `convert_transformer`，主要处理 `model_type`, `model_dir`, `output_dir` 等输入。 文档字符串说明：Convert a single quantized transformer directory into Diffusers format.

### Lines 146-183: Function `repack` / 函数 `repack`
```python
def repack(
    model_type: str,
    original_model_path: pathlib.Path,
    quant_path: pathlib.Path,
    output_path: pathlib.Path,
) -> None:
    """
    Full one-step repack workflow:
      1. Copy the original HF Diffusers model to output_path, excluding transformer dir(s).
      2. For each transformer: convert quant weights and copy config.json from original.
    """
    transformer_dirs = get_transformer_dirs(model_type)

    # Step 1: Copy original model, skipping transformer dirs (they will be replaced)
# ...
            shutil.copy2(str(src_config), str(out_tdir / "config.json"))
            logger.debug(f"  Copied config.json from original {tdir}/")

    logger.info(f"\nDone! Repacked model saved to: {output_path}")
```
**EN:** This function drives `repack` with inputs such as `model_type`, `original_model_path`, `quant_path`, `output_path`. Full one-step repack workflow:
**CN:** 这个函数负责 `repack`，主要处理 `model_type`, `original_model_path`, `quant_path`, `output_path` 等输入。 文档字符串说明：Full one-step repack workflow:

### Lines 186-215: Function `get_args` / 函数 `get_args`
```python
def get_args():
    parser = argparse.ArgumentParser(
        description="Repack msmodelslim quantized Wan2.2 weights into HF Diffusers format"
    )
    parser.add_argument(
        "--model-type",
        type=str,
        required=True,
        choices=SUPPORTED_MODEL_TYPES,
        help="Model type to convert",
    )
    parser.add_argument(
        "--original-model-path",
        type=str,
# ...
        required=True,
        help="Output path for the repacked model (e.g., /weights/Wan2.2-TI2V-5B-Diffusers-MXFP8)",
    )
    return parser.parse_args()
```
**EN:** This function drives `get_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_args`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 216-225: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    args = get_args()
    repack(
        model_type=args.model_type,
        original_model_path=pathlib.Path(args.original_model_path),
        quant_path=pathlib.Path(args.quant_path),
        output_path=pathlib.Path(args.output_path),
    )
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Image generation flow / 图像生成流程
- Symbol `get_transformer_config` anchors the module API / 符号 `get_transformer_config` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `safetensors.torch`
- **Stdlib / 标准库**: `argparse`, `json`, `pathlib`, `shutil`, `typing`
