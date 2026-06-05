# generate_matrix.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/slurm/generate_matrix.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `generate_matrix` workflow in SGLang. It mainly handles CI orchestration, nightly workflow scheduling. / 该Python 模块用于支撑 SGLang 中的 `generate_matrix` 流程，主要负责CI 编排、夜间任务调度。它属于 `slurm` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Reads nightly-configs.yaml and generates one matrix entry per recipe YAML,
where each srt-slurm recipe runs its full concurrency sweep as a single Slurm job.

conc-list in the config is documentation only and is not used to split jobs.

Output: JSON array written to stdout, consumed by the workflow setup job as
a dynamic matrix via fromJson(needs.setup.outputs.matrix).

Usage:
    python3 generate_matrix.py <path-to-nightly-configs.yaml> --runner <label> [--filter NAMES]

Example:
    python3 generate_matrix.py scripts/ci/slurm/nightly-configs.yaml --runner gb200
    python3 generate_matrix.py scripts/ci/slurm/nightly-configs.yaml --runner gb200 \\
        --filter dsr1-fp8-1k1k-max-tpt,dsr1-fp4-1k1k-mid-curve
"""
```
**EN:** Reads nightly-configs.yaml and generates one matrix entry per recipe YAML, where each srt-slurm recipe runs its full concurrency sweep as a single Slurm job.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 19-23: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import json
import sys

import yaml
```
**EN:** This block loads argparse, json, sys, yaml. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, json, sys, yaml。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 26-30: Defines the `seq_len_str` routine / 定义 `seq_len_str` 例程
```python
def seq_len_str(isl, osl):
    def fmt(n):
        return f"{n // 1024}k" if n % 1024 == 0 else str(n)

    return f"{fmt(isl)}{fmt(osl)}"
```
**EN:** This block defines `seq_len_str`. It accepts 2 parameter(s): isl, osl. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `seq_len_str`。它接收 2 个参数：isl, osl。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 33-93: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("config_file", help="Path to nightly-configs.yaml")
    parser.add_argument(
        "--runner",
        required=True,
        help="Filter configs by runner label (e.g. gb200, b200)",
    )
    parser.add_argument(
        "--filter",
        default="",
        help=(
            "Optional comma-separated list of matrix entry names to include "
            "(e.g. 'dsr1-fp8-1k1k-max-tpt'). Names must match exactly."
        ),
    )
    args = parser.parse_args()

    with open(args.config_file) as f:
        data = yaml.safe_load(f)

    matrix = []
    for exp_name, exp in data.items():
        if exp["runner"] != args.runner:
            continue

        for seq_cfg in exp["seq-len-configs"]:
            isl, osl = seq_cfg["isl"], seq_cfg["osl"]
            sl = seq_len_str(isl, osl)

            for entry in seq_cfg["search-space"]:
                config_file = entry["config_file"]
                topology = config_file.rsplit("/", 1)[-1].replace(".yaml", "")

                matrix.append(
                    {
                        "name": f"{exp['model-prefix']}-{exp['precision']}-{sl}-{topology}",
                        "exp_name": exp_name,
                        "model": exp["model"],
                        "model_prefix": exp["model-prefix"],
                        "precision": exp["precision"],
                        "isl": str(isl),
                        "osl": str(osl),
                        "config_file": config_file,
                    }
                )

    wanted = [n.strip() for n in args.filter.split(",") if n.strip()]
    if wanted:
        available = [e["name"] for e in matrix]
        unknown = [n for n in wanted if n not in available]
        if unknown:
            print(
                f"ERROR: unknown config name(s): {', '.join(unknown)}. "
                f"Available for runner '{args.runner}': {', '.join(available)}",
                file=sys.stderr,
            )
            sys.exit(1)
        matrix = [e for e in matrix if e["name"] in wanted]

    print(json.dumps(matrix))
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it reads or writes files, parses CLI arguments, emits status messages, controls process exit status.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会读写文件、解析命令行参数、输出状态信息、控制进程退出状态。

### Lines 96-97: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **YAML configuration** / YAML 配置
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `sys`
- **Third-party modules / 第三方模块**: `yaml`
