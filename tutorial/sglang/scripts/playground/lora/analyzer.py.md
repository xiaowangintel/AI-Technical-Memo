# analyzer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/playground/lora/analyzer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `analyzer` workflow in SGLang. It mainly handles scripted automation. / 该Python 模块用于支撑 SGLang 中的 `analyzer` 流程，主要负责脚本化自动化。它属于 `lora` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import glob
import json
import os
import re
import sys

from tqdm import tqdm
```
**EN:** This block loads glob, json, os, re, sys, tqdm. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 glob, json, os, re, sys, tqdm。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 9-9: Top-level expr logic / 顶层 expr 逻辑
```python
sys.path.append("../../")
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 10-10: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
from fix_corrupted_json import clean_json_file
```
**EN:** This block loads fix_corrupted_json. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 fix_corrupted_json。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 12-17: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
dirpath = "/Users/ying"
output_file_prefix = "analyzed_log"

time = {}
tot_time = {}
size = {}
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 19-19: Top-level expr logic / 顶层 expr 逻辑
```python
os.system(f"rm {output_file_prefix}*")
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 21-73: Top-level for logic / 顶层 for 逻辑
```python
for dirname in glob.glob(os.path.join(dirpath, "trace*")):
    print(dirname)
    trace_name = dirname.split("/")[-1]
    time[trace_name] = {}
    size[trace_name] = {}
    total_time = 0
    for filename in tqdm(glob.glob(os.path.join(dirname, "*.json"))):
        step_name = filename.split("/")[-1].split(".")[0]
        step_name = "_".join(step_name.split("_")[1:])
        if "prefill" not in filename and "decode" not in filename:
            continue

        match = re.search(r"(prefill|decode)_step_(\d+)\.json", filename)
        if match:
            phase = match.group(1)
            step = match.group(2)
        else:
            raise Exception(f"Cannot parse {filename}")

        try:
            with open(filename, "r") as f:
                trace = json.load(f)
        except:
            clean_json_file(filename, filename)
            with open(filename, "r") as f:
                trace = json.load(f)

        for event in trace["traceEvents"]:
            name = event["name"]
            if name in ["profile_prefill_step", "profile_decode_step"]:
                dur = event["dur"] / 1e3
                time[trace_name][step_name] = dur
                break
        total_time += dur

        step = int(step_name.split("_")[-1])
        with open(os.path.join(dirname, f"size_{step}.json"), "r") as f:
            size_info = json.load(f)
        size[trace_name][step_name] = size_info["size"]

    tot_time[trace_name] = total_time
    time[trace_name] = dict(
        sorted(time[trace_name].items(), key=lambda x: int(x[0].split("_")[-1]))
    )
    size[trace_name] = dict(
        sorted(size[trace_name].items(), key=lambda x: int(x[0].split("_")[-1]))
    )

    with open(f"{output_file_prefix}_{trace_name}", "a") as f:
        for k, v in time[trace_name].items():
            size_v = size[trace_name][k]
            print(f"{k:>15}{v:10.2f}\t{size_v}")
            f.write(f"{k:>15}{v:10.2f}\t{size_v}\n")
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

### Lines 75-77: Top-level with logic / 顶层 with 逻辑
```python
with open(f"{output_file_prefix}_total_time", "w") as f:
    print(tot_time)
    json.dump(tot_time, f)
```
**EN:** This top-level block performs module setup that does not fit into a standalone helper, but still affects the surrounding control flow.
**CN:** 该顶层代码块执行了一部分无法单独抽成辅助函数的初始化逻辑，并会影响后续控制流。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `glob`, `json`, `os`, `re`, `sys`
- **Third-party modules / 第三方模块**: `fix_corrupted_json`, `tqdm`
