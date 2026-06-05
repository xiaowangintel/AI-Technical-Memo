# log_extract.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/jit/log_extract.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Provides JIT-oriented debugging or log-processing helpers used during compiler investigation.
- **用途 (CN)**: 提供面向 JIT 的调试或日志处理辅助脚本，用于编译器问题分析。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行

````python
import argparse
import functools
import traceback
from typing import Callable, List, Tuple

from torch.utils.jit.log_extract import (
    extract_ir,
    load_graph_and_inputs,
    run_baseline_no_fusion,
    run_nnc,
    run_nvfuser,
)
````

- EN: This block imports dependencies such as `argparse`, `functools`, `traceback`, `typing`; parses command-line arguments for script entry points.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数。

### Lines 15-26 / 第 15-26 行

````python
"""
Usage:
1. Run your script and pipe into a log file
  PYTORCH_JIT_LOG_LEVEL=">>graph_fuser" python3 my_test.py &> log.txt
2. Run log_extract:
  log_extract.py log.txt --nvfuser --nnc-dynamic --nnc-static

You can also extract the list of extracted IR:
  log_extract.py log.txt --output

Passing in --graphs 0 2 will only run graphs 0 and 2
"""
````

- EN: This block implements local helper logic for log extract.
- CN: 该代码块实现与 log extract 相关的局部辅助逻辑。

### Lines 29-37 / 第 29-37 行

````python
def test_runners(
    graphs: List[str],
    runners: List[Tuple[str, Callable]],
    graph_set: List[int] | None,
):
    for i, ir in enumerate(graphs):
        _, inputs = load_graph_and_inputs(ir)
        if graph_set and i not in graph_set:
            continue
````

- EN: This block defines callable units such as `test_runners`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 39-52 / 第 39-52 行

````python
        print(f"Running Graph {i}")
        prev_result = None
        prev_runner_name = None
        for runner in runners:
            runner_name, runner_fn = runner
            try:
                result = runner_fn(ir, inputs)
                if prev_result:
                    improvement = (prev_result / result - 1) * 100
                    print(
                        f"{runner_name} : {result:.6f} ms improvement over {prev_runner_name}: improvement: {improvement:.2f}%"
                    )
                else:
                    print(f"{runner_name} : {result:.6f} ms")
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 53-66 / 第 53-66 行

````python
                prev_result = result
                prev_runner_name = runner_name
            except RuntimeError:
                print(f"  Graph {i} failed for {runner_name} :", traceback.format_exc())


def run():
    parser = argparse.ArgumentParser(
        description="Extracts torchscript IR from log files and, optionally, benchmarks it or outputs the IR"
    )
    parser.add_argument("filename", help="Filename of log file")
    parser.add_argument(
        "--nvfuser", dest="nvfuser", action="store_true", help="benchmark nvfuser"
    )
````

- EN: This block defines callable units such as `run`; parses command-line arguments for script entry points; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；遍历集合、记录或生成项。

### Lines 67-80 / 第 67-80 行

````python
    parser.add_argument(
        "--no-nvfuser",
        dest="nvfuser",
        action="store_false",
        help="DON'T benchmark nvfuser",
    )
    parser.set_defaults(nvfuser=False)
    parser.add_argument(
        "--nnc-static",
        dest="nnc_static",
        action="store_true",
        help="benchmark nnc static",
    )
    parser.add_argument(
````

- EN: This block parses command-line arguments for script entry points.
- CN: 该代码块为脚本入口解析命令行参数。

### Lines 81-94 / 第 81-94 行

````python
        "--no-nnc-static",
        dest="nnc_static",
        action="store_false",
        help="DON'T benchmark nnc static",
    )
    parser.set_defaults(nnc_static=False)

    parser.add_argument(
        "--nnc-dynamic",
        dest="nnc_dynamic",
        action="store_true",
        help="nnc with dynamic shapes",
    )
    parser.add_argument(
````

- EN: This block parses command-line arguments for script entry points.
- CN: 该代码块为脚本入口解析命令行参数。

### Lines 95-108 / 第 95-108 行

````python
        "--no-nnc-dynamic",
        dest="nnc_dynamic",
        action="store_false",
        help="don't benchmark nnc with dynamic shapes",
    )
    parser.set_defaults(nnc_dynamic=False)

    parser.add_argument(
        "--baseline", dest="baseline", action="store_true", help="benchmark baseline"
    )
    parser.add_argument(
        "--no-baseline",
        dest="baseline",
        action="store_false",
````

- EN: This block parses command-line arguments for script entry points.
- CN: 该代码块为脚本入口解析命令行参数。

### Lines 109-119 / 第 109-119 行

````python
        help="DON'T benchmark baseline",
    )
    parser.set_defaults(baseline=False)

    parser.add_argument(
        "--output", dest="output", action="store_true", help="Output graph IR"
    )
    parser.add_argument(
        "--no-output", dest="output", action="store_false", help="DON'T output graph IR"
    )
    parser.set_defaults(output=False)
````

- EN: This block parses command-line arguments for script entry points.
- CN: 该代码块为脚本入口解析命令行参数。

### Lines 121-129 / 第 121-129 行

````python
    parser.add_argument(
        "--graphs", nargs="+", type=int, help="Run only specified graph indices"
    )

    args = parser.parse_args()
    graphs = extract_ir(args.filename)

    graph_set = args.graphs
    graph_set = graph_set if graph_set else None
````

- EN: This block parses command-line arguments for script entry points; branches on runtime conditions or configuration choices.
- CN: 该代码块为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支。

### Lines 131-141 / 第 131-141 行

````python
    options = []
    if args.baseline:
        options.append(("Baseline no fusion", run_baseline_no_fusion))
    if args.nnc_dynamic:
        options.append(("NNC Dynamic", functools.partial(run_nnc, dynamic=True)))
    if args.nnc_static:
        options.append(("NNC Static", functools.partial(run_nnc, dynamic=False)))
    if args.nvfuser:
        options.append(("NVFuser", run_nvfuser))

    test_runners(graphs, options, graph_set)
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 143-149 / 第 143-149 行

````python
    if args.output:
        quoted = []
        for i, ir in enumerate(graphs):
            if graph_set and i not in graph_set:
                continue
            quoted.append('"""' + ir + '"""')
        print("[" + ", ".join(quoted) + "]")
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 152-153 / 第 152-153 行

````python
if __name__ == "__main__":
    run()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: CLI entry points — the module exposes command-line arguments so developers or CI jobs can drive automation predictably.
  CN: 命令行入口——该模块暴露命令行参数，使开发者或 CI 作业能够以可预测方式驱动自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `argparse`, `functools`, `traceback`, `typing`
- Internal imports / 内部导入: `torch.utils.jit.log_extract`
- External imports / 外部导入: none
- Classes / 类: none
- Functions / 函数: `test_runners`, `run`
- Test entry points / 测试入口: `test_runners`
