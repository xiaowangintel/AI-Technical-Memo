# model_truncator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/model_truncator.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on shared debugging utilities. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于共享调试工具。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-11: Import dependencies and shared types / 导入依赖与共享类型
```python
import json
import os
import re
from argparse import ArgumentParser
from pathlib import Path
from typing import Dict

import torch
from huggingface_hub import snapshot_download
from safetensors.torch import load_file, save_file
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 14-51: Implement function `main` / 实现函数 `main`
```python
def main(args):
    dir_input = Path(_maybe_snapshot_download(args.input))
    dir_output = Path(args.output)
    print(f"{dir_input=} {dir_output=}")

    dir_output.mkdir(parents=True, exist_ok=True)

    for pattern in ["generation_config.json", "*.py", "tokenizer*"]:
        os.system(f"cp -rf {dir_input}/{pattern} {dir_output}")

    _transform_json(
        dir_input,
        dir_output,
        "config.json",
        lambda data: _transform_config(args, data),
    )

    safetensors_index = _transform_json(
        dir_input,
        dir_output,
        "model.safetensors.index.json",
        lambda data: _transform_safetensors_index(args, data),
    )

    for path_input_safetensors in sorted(list(dir_input.glob("*.safetensors"))):
        path_output_safetensors = dir_output / path_input_safetensors.relative_to(
            dir_input
        )

        state_dict = load_file(path_input_safetensors)
        _transform_safetensors_file(
            state_dict, safetensors_index, debug_name=str(path_output_safetensors)
        )
        if len(state_dict) > 0:
            print(f"Save {len(state_dict)} tensors to {path_output_safetensors}")
            save_file(state_dict, path_output_safetensors)
        else:
            print(f"Skip saving {path_output_safetensors} since it is empty")
```
**EN:** Function `main` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `main` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 54-57: Implement helper `_maybe_snapshot_download` / 实现辅助函数 `_maybe_snapshot_download`
```python
def _maybe_snapshot_download(path):
    if Path(path).exists():
        return path
    return snapshot_download(path)
```
**EN:** Function `_maybe_snapshot_download` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_maybe_snapshot_download` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 60-64: Implement helper `_transform_json` / 实现辅助函数 `_transform_json`
```python
def _transform_json(dir_input, dir_output, filename, fn):
    data = json.loads((dir_input / filename).read_text())
    fn(data)
    (dir_output / filename).write_text(json.dumps(data, indent=4))
    return data
```
**EN:** Function `_transform_json` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_transform_json` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 67-68: Implement helper `_transform_config` / 实现辅助函数 `_transform_config`
```python
def _transform_config(args, config_json):
    config_json["num_hidden_layers"] = args.keep_num_layers
```
**EN:** Function `_transform_config` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_transform_config` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 71-76: Implement helper `_transform_safetensors_index` / 实现辅助函数 `_transform_safetensors_index`
```python
def _transform_safetensors_index(args, safetensors_index):
    weight_map = safetensors_index["weight_map"]
    weight_map = {
        name: loc for name, loc in weight_map.items() if _filter_tensor_name(args, name)
    }
    safetensors_index["weight_map"] = weight_map
```
**EN:** Function `_transform_safetensors_index` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_transform_safetensors_index` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 79-85: Implement helper `_transform_safetensors_file` / 实现辅助函数 `_transform_safetensors_file`
```python
def _transform_safetensors_file(
    state_dict: Dict[str, torch.Tensor], safetensors_index, debug_name: str
):
    names_to_remove = set(state_dict) - set(safetensors_index["weight_map"])
    print(f"Remove {list(names_to_remove)} in {debug_name}")
    for name in names_to_remove:
        del state_dict[name]
```
**EN:** Function `_transform_safetensors_file` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_transform_safetensors_file` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 88-95: Implement helper `_filter_tensor_name` / 实现辅助函数 `_filter_tensor_name`
```python
def _filter_tensor_name(args, tensor_name: str):
    # We focus on DeepSeek-like names currently, but can be easily extended to more kinds of models
    m = re.match(r"^model.layers.(\d+).*", tensor_name)
    if m is None:
        return True

    layer_id = int(m.group(1))
    return layer_id < args.keep_num_layers
```
**EN:** Function `_filter_tensor_name` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_filter_tensor_name` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 98-112: Provide a command-line entrypoint / 提供命令行入口
```python
if __name__ == "__main__":
    """
    Example:
    python -m sglang.srt.debug_utils.model_truncator --input deepseek-ai/DeepSeek-V3-0324 --output /tmp/DeepSeek-V3-0324-5layer
    hf upload my_name/DeepSeek-V3-0324-5layer /tmp/DeepSeek-V3-0324-5layer

    Alternatively, the following may be used on-the-fly.
    But this may not be useful to test RL frameworks, and sometimes it may have issues.
        --json-model-override-args '{"num_hidden_layers": 5}'
    """
    parser = ArgumentParser(description="Create truncated model for fast debugging.")
    parser.add_argument("--input", type=str, required=True)
    parser.add_argument("--output", type=str, required=True)
    parser.add_argument("--keep-num-layers", type=int, default=5)
    main(parser.parse_args())
```
**EN:** This conditional branch selects behavior based on runtime state, optional inputs, or developer-facing entry conditions.
**CN:** 该条件分支会根据运行时状态、可选输入或面向开发者的入口条件来选择不同的行为。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `main`, `_maybe_snapshot_download`, `_transform_json`, `_transform_config`, `_transform_safetensors_index`, `_transform_safetensors_file`, `_filter_tensor_name`
- **Module role / 模块角色**: Shared debugging utilities / 共享调试工具
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `re`, `argparse`, `pathlib`, `typing`
- **Third-party / 第三方**: `torch`, `huggingface_hub`, `safetensors`
- **Internal / 内部**: None / 无
