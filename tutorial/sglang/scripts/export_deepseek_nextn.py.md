# export_deepseek_nextn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/export_deepseek_nextn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `export_deepseek_nextn` workflow in SGLang. It mainly handles CI orchestration. / 该Python 模块用于支撑 SGLang 中的 `export_deepseek_nextn` 流程，主要负责CI 编排。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Export NextN layer for DeepSeek-V3/R1 model. The exported model can be used for speculative decoding.

Usage:
python3 export_deepseek_nextn.py --input-dir /path/to/DeepSeek-V3 --output-dir /path/to/DeepSeek-V3-NextN
"""
```
**EN:** Export NextN layer for DeepSeek-V3/R1 model.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 8-15: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import json
import os
import shutil

from safetensors import safe_open
from safetensors.torch import save_file
from transformers import AutoConfig
```
**EN:** This block loads argparse, json, os, shutil, safetensors, safetensors.torch, transformers. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, json, os, shutil, safetensors, safetensors.torch, transformers。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 18-21: Defines the `get_nextn_layer_id` routine / 定义 `get_nextn_layer_id` 例程
```python
def get_nextn_layer_id(config):
    if not hasattr(config, "num_hidden_layers"):
        raise ValueError("'num_hidden_layers' not found in model config.")
    return config.num_hidden_layers
```
**EN:** This block defines `get_nextn_layer_id`. It accepts 1 parameter(s): config. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `get_nextn_layer_id`。它接收 1 个参数：config。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 24-33: Defines the `update_and_save_config` routine / 定义 `update_and_save_config` 例程
```python
def update_and_save_config(config, output_dir):
    new_config = config.to_dict()
    new_config.update(
        {
            "num_hidden_layers": 1,
            "architectures": ["DeepseekV3ForCausalLMNextN"],
        }
    )
    with open(os.path.join(output_dir, "config.json"), "w") as f:
        json.dump(new_config, f, indent=2, ensure_ascii=False, sort_keys=True)
```
**EN:** This block defines `update_and_save_config`. It accepts 2 parameter(s): config, output_dir. Internally it reads or writes files.
**CN:** 该代码块定义了 `update_and_save_config`。它接收 2 个参数：config, output_dir。内部会读写文件。

### Lines 36-42: Defines the `copy_non_safetensors_files` routine / 定义 `copy_non_safetensors_files` 例程
```python
def copy_non_safetensors_files(input_dir, output_dir):
    for filename in os.listdir(input_dir):
        src_file_path = os.path.join(input_dir, filename)
        if os.path.isfile(src_file_path) and not filename.endswith(".safetensors"):
            dst_file_path = os.path.join(output_dir, filename)
            shutil.copy2(src_file_path, dst_file_path)
    print(f"All non-safetensors files have been copied to {output_dir}")
```
**EN:** This block defines `copy_non_safetensors_files`. It accepts 2 parameter(s): input_dir, output_dir. Internally it emits status messages.
**CN:** 该代码块定义了 `copy_non_safetensors_files`。它接收 2 个参数：input_dir, output_dir。内部会输出状态信息。

### Lines 45-88: Defines the `export_nextn_layer_parameters` routine / 定义 `export_nextn_layer_parameters` 例程
```python
def export_nextn_layer_parameters(input_dir, output_dir, nextn_layer_id):
    prefix = f"model.layers.{nextn_layer_id}"
    output_path = os.path.join(output_dir, "nextn_layer_parameters.safetensors")
    params = {}
    for filename in os.listdir(input_dir):
        if not filename.endswith(".safetensors"):
            continue

        file_path = os.path.join(input_dir, filename)
        print(f"Processing: {filename}")

        try:
            with safe_open(file_path, framework="pt") as f:
                matching_keys = [k for k in f.keys() if k.startswith(prefix)]

                if not matching_keys:
                    print(f"  No parameters starting with '{prefix}' found")
                    continue

                for key in matching_keys:
                    if "embed_tokens" in key or "shared_head.head" in key:
                        continue
                    new_key = key.replace(prefix, "model.layers.0")
                    params[new_key] = f.get_tensor(key)

        except Exception as e:
            print(f"  Error processing {filename}: {str(e)}")

    if params:
        print(f"Saving {len(params)} parameters to {output_path}")
        save_file(params, output_path)
    else:
        print("No matching parameters found.")

    # Update safetensors index
    index_path = os.path.join(output_dir, "model.safetensors.index.json")
    print(f"Updating safetensors index to {index_path}")
    index_data = {"weight_map": {}}
    for key in params:
        index_data["weight_map"][key] = "nextn_layer_parameters.safetensors"
    with open(index_path, "w") as f:
        json.dump(index_data, f, indent=4)

    print("All done.")
```
**EN:** This block defines `export_nextn_layer_parameters`. It accepts 3 parameter(s): input_dir, output_dir, nextn_layer_id. Internally it reads or writes files, emits status messages.
**CN:** 该代码块定义了 `export_nextn_layer_parameters`。它接收 3 个参数：input_dir, output_dir, nextn_layer_id。内部会读写文件、输出状态信息。

### Lines 91-115: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Export NextN layer parameters for DeepSeek-V3/R1"
    )
    parser.add_argument(
        "--input-dir",
        type=str,
        required=True,
        help="Input HF model directory.",
    )
    parser.add_argument(
        "--output-dir",
        type=str,
        required=True,
        help="Output nextn model directory.",
    )
    args = parser.parse_args()

    config = AutoConfig.from_pretrained(args.input_dir, trust_remote_code=True)
    assert config.num_nextn_predict_layers == 1, "Only 1 nextn layer is supported."
    nextn_layer_id = get_nextn_layer_id(config)
    os.makedirs(args.output_dir, exist_ok=True)
    copy_non_safetensors_files(args.input_dir, args.output_dir)
    update_and_save_config(config, args.output_dir)
    export_nextn_layer_parameters(args.input_dir, args.output_dir, nextn_layer_id)
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It parses CLI arguments.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会解析命令行参数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `os`, `shutil`
- **Third-party modules / 第三方模块**: `safetensors`, `safetensors.torch`, `transformers`
