# process_result.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/slurm/process_result.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `process_result` workflow in SGLang. It mainly handles CI orchestration, benchmark automation. / 该Python 模块用于支撑 SGLang 中的 `process_result` 流程，主要负责CI 编排、基准测试自动化。它属于 `slurm` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Process a raw srt-slurm benchmark result JSON into an aggregated format.

Usage (called once per result file):
    RESULT_FILENAME=<path_without_.json> PREFILL_GPUS=<n> DECODE_GPUS=<n> \\
        RECIPE_FILE=<path_to_recipe.yaml> python3 process_result.py

Required env vars:
    RESULT_FILENAME   - path to the result file without the .json extension
    FRAMEWORK         - e.g. dynamo-sglang
    PRECISION         - e.g. fp8, fp4
    MODEL_PREFIX      - short model label, e.g. dsr1
    ISL               - input sequence length
    OSL               - output sequence length
    PREFILL_GPUS      - number of prefill GPUs (extracted from result filename)
    DECODE_GPUS       - number of decode GPUs (extracted from result filename)

Optional env vars:
    RECIPE_FILE       - path to the srt-slurm recipe YAML; if set, topology
                        fields (TP, EP, DP, workers) are parsed from it
"""
```
**EN:** Process a raw srt-slurm benchmark result JSON into an aggregated format.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 22-25: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import json
import os
import sys
from pathlib import Path
```
**EN:** This block loads json, os, sys, pathlib. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 json, os, sys, pathlib。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 28-33: Defines the `require` routine / 定义 `require` 例程
```python
def require(var):
    val = os.environ.get(var)
    if val is None:
        print(f"ERROR: Missing required env var: {var}", file=sys.stderr)
        sys.exit(1)
    return val
```
**EN:** This block defines `require`. It accepts 1 parameter(s): var. Internally it emits status messages, controls process exit status.
**CN:** 该代码块定义了 `require`。它接收 1 个参数：var。内部会输出状态信息、控制进程退出状态。

### Lines 36-43: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
result_filename = require("RESULT_FILENAME")
framework = require("FRAMEWORK")
precision = require("PRECISION")
model_prefix = require("MODEL_PREFIX")
isl = int(require("ISL"))
osl = int(require("OSL"))
prefill_gpus = int(require("PREFILL_GPUS"))
decode_gpus = int(require("DECODE_GPUS"))
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 45-46: Top-level with logic / 顶层 with 逻辑
```python
with open(f"{result_filename}.json") as f:
    raw = json.load(f)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 51-54: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
prefill_tp = prefill_ep = prefill_dp_attn = 0
prefill_num_workers = decode_tp = decode_ep = decode_dp_attn = decode_num_workers = 0

recipe_file = os.environ.get("RECIPE_FILE")
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 55-74: Top-level if logic / 顶层 if 逻辑
```python
if recipe_file and Path(recipe_file).exists():
    import yaml

    with open(recipe_file) as f:
        recipe = yaml.safe_load(f)

    res = recipe.get("resources", {})
    prefill_num_workers = res.get("prefill_workers", 0)
    decode_num_workers = res.get("decode_workers", 0)

    sgl = recipe.get("backend", {}).get("sglang_config", {})
    p = sgl.get("prefill", {})
    d = sgl.get("decode", {})

    prefill_tp = p.get("tensor-parallel-size", 0)
    prefill_ep = p.get("expert-parallel-size", 0)
    prefill_dp_attn = p.get("data-parallel-size", "-")
    decode_tp = d.get("tensor-parallel-size", 0)
    decode_ep = d.get("expert-parallel-size", 0)
    decode_dp_attn = d.get("data-parallel-size", "-")
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 76-105: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
total_gpus = prefill_gpus + decode_gpus

data = {
    "hw": "gb200",
    "conc": int(raw["max_concurrency"]),
    "model": raw["model_id"],
    "infmax_model_prefix": model_prefix,
    "framework": framework,
    "precision": precision,
    "isl": isl,
    "osl": osl,
    "is_multinode": True,
    "disagg": True,
    "num_prefill_gpu": prefill_gpus,
    "num_decode_gpu": decode_gpus,
    "prefill_num_workers": prefill_num_workers,
    "prefill_tp": prefill_tp,
    "prefill_ep": prefill_ep,
    "prefill_dp_attention": prefill_dp_attn,
    "decode_num_workers": decode_num_workers,
    "decode_tp": decode_tp,
    "decode_ep": decode_ep,
    "decode_dp_attention": decode_dp_attn,
    "tput_per_gpu": float(raw["total_token_throughput"]) / total_gpus,
    "output_tput_per_gpu": float(raw["output_throughput"]) / decode_gpus,
    "input_tput_per_gpu": (
        float(raw["total_token_throughput"]) - float(raw["output_throughput"])
    )
    / prefill_gpus,
}
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 107-111: Top-level for logic / 顶层 for 逻辑
```python
for key, value in raw.items():
    if key.endswith("_ms"):
        data[key.replace("_ms", "")] = float(value) / 1000.0
    if "tpot" in key:
        data[key.replace("_ms", "").replace("tpot", "intvty")] = 1000.0 / float(value)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 113-113: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
out_path = Path(result_filename).parent / f"agg_{Path(result_filename).name}.json"
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 114-115: Top-level with logic / 顶层 with 逻辑
```python
with open(out_path, "w") as f:
    json.dump(data, f, indent=2)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 117-117: Top-level expr logic / 顶层 expr 逻辑
```python
print(f"Written: {out_path}")
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **YAML configuration** / YAML 配置
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `pathlib`, `sys`
- **Third-party modules / 第三方模块**: `yaml`
