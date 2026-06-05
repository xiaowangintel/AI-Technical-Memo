# summarize.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/slurm/summarize.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `summarize` workflow in SGLang. It mainly handles CI orchestration, benchmark automation. / 该Python 模块用于支撑 SGLang 中的 `summarize` 流程，主要负责CI 编排、基准测试自动化。它属于 `slurm` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Print a markdown summary table from processed benchmark results.

Usage:
    python3 summarize.py <results_dir>

Reads all agg_*.json files recursively from <results_dir> and prints a
markdown table to stdout (redirect to $GITHUB_STEP_SUMMARY to publish).
"""
```
**EN:** Print a markdown summary table from processed benchmark results.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 10-14: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import json
import sys
from pathlib import Path

from tabulate import tabulate
```
**EN:** This block loads json, sys, pathlib, tabulate. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 json, sys, pathlib, tabulate。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 16-42: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
HEADERS = [
    "Model",
    "Served Model",
    "Hardware",
    "Framework",
    "Precision",
    "ISL",
    "OSL",
    "Prefill TP",
    "Prefill EP",
    "Prefill DP Attn",
    "Prefill Workers",
    "Prefill GPUs",
    "Decode TP",
    "Decode EP",
    "Decode DP Attn",
    "Decode Workers",
    "Decode GPUs",
    "Conc",
    "TTFT (ms)",
    "TPOT (ms)",
    "Interactivity (tok/s/user)",
    "E2EL (s)",
    "TPUT per GPU",
    "Output TPUT per GPU",
    "Input TPUT per GPU",
]
```
**EN:** This section defines HEADERS, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 HEADERS，用于把可复用的默认值集中在模块顶部。

### Lines 45-50: Defines the `load_json` routine / 定义 `load_json` 例程
```python
def load_json(path):
    try:
        with open(path) as f:
            return json.load(f)
    except Exception:
        return None
```
**EN:** This block defines `load_json`. It accepts 1 parameter(s): path. Internally it loads structured data, reads or writes files.
**CN:** 该代码块定义了 `load_json`。它接收 1 个参数：path。内部会加载结构化数据、读写文件。

### Lines 53-118: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    if len(sys.argv) < 2:
        print("Usage: python3 summarize.py <results_dir>")
        sys.exit(1)

    results_dir = Path(sys.argv[1])
    results = [
        r
        for path in results_dir.rglob("agg_*.json")
        if (r := load_json(path)) and "is_multinode" in r
    ]

    if not results:
        print("No processed result files found.")
        return

    results.sort(
        key=lambda r: (
            r["infmax_model_prefix"],
            r["hw"],
            r["framework"],
            r["precision"],
            r["isl"],
            r["osl"],
            r["prefill_tp"],
            r["prefill_ep"],
            r["decode_tp"],
            r["decode_ep"],
            r["conc"],
        )
    )

    rows = [
        [
            r["infmax_model_prefix"],
            r["model"],
            r["hw"].upper(),
            r["framework"].upper(),
            r["precision"].upper(),
            r["isl"],
            r["osl"],
            r["prefill_tp"],
            r["prefill_ep"],
            r["prefill_dp_attention"],
            r["prefill_num_workers"],
            r["num_prefill_gpu"],
            r["decode_tp"],
            r["decode_ep"],
            r["decode_dp_attention"],
            r["decode_num_workers"],
            r["num_decode_gpu"],
            r["conc"],
            f"{r['median_ttft'] * 1000:.4f}",
            f"{r['median_tpot'] * 1000:.4f}",
            f"{r['median_intvty']:.4f}",
            f"{r['median_e2el']:.4f}",
            f"{r['tput_per_gpu']:.4f}",
            f"{r['output_tput_per_gpu']:.4f}",
            f"{r['input_tput_per_gpu']:.4f}",
        ]
        for r in results
    ]

    print("## GB200 Nightly Benchmark Results\n")
    print(tabulate(rows, headers=HEADERS, tablefmt="github"))
    print()
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it touches filesystem paths, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会处理文件系统路径、输出状态信息、控制进程退出状态。

### Lines 121-122: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `pathlib`, `sys`
- **Third-party modules / 第三方模块**: `tabulate`
