# nfc-stat-parser.py — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/utils/nfc-stat-parser.py`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: !/usr/bin/env python3. It also sits in code that belongs to the BOLT post-link optimizer subsystem. / 该文件属于 BOLT 链接后优化器子系统。 源码头部说明其职责是：!/usr/bin/env python3。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```python
#!/usr/bin/env python3
import argparse
import csv
import re
import sys
import os
from statistics import geometric_mean
```

- EN: Imports Python modules such as `argparse`, `csv`, `re`, `sys`, and 2 more to support scripting logic. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `argparse`, `csv`, `re`, `sys`, `os`, `from statistics import geometric_mean`.
- CN: 这里导入 Python 模块，例如 `argparse`, `csv`, `re`, `sys`, and 2 more，以支撑脚本逻辑。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `argparse`, `csv`, `re`, `sys`, `os`, `from statistics import geometric_mean`。

### Lines 9-26

```python
TIMING_LOG_RE = re.compile(r"(.*)/(.*).tmp(.*)")


def main():
    parser = argparse.ArgumentParser(
        description="BOLT NFC stat parser",
        formatter_class=argparse.ArgumentDefaultsHelpFormatter,
    )
    parser.add_argument(
        "input", nargs="+", help="timing.log files produced by llvm-bolt-wrapper"
    )
    parser.add_argument(
        "--check_longer_than",
        default=2,
        type=float,
        help="Only warn on tests longer than X seconds for at least one side",
    )
    parser.add_argument(
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 27-40

```python
        "--threshold_single",
        default=10,
        type=float,
        help="Threshold for a single test result swing, abs percent",
    ),
    parser.add_argument(
        "--threshold_agg",
        default=5,
        type=float,
        help="Threshold for geomean test results swing, abs percent",
    ),
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 41-48

```python
    def fmt_delta(value, exc_threshold, above_bound=True):
        formatted_value = format(value, "+.2%")
        if not above_bound:
            formatted_value += "?"
        elif exc_threshold and sys.stdout.isatty():  # terminal supports colors
            return f"\033[1m{formatted_value}\033[0m"
        return formatted_value
```

- EN: Declares or implements routines including `format`. Notable symbols here include `format`.
- CN: 这里声明或实现函数，例如 `format`。这里较值得关注的符号包括 `format`。

### Lines 49-56

```python
    # Ratios for geomean computation
    time_ratios = []
    mem_ratios = []
    # Whether any test exceeds the single test threshold (mem or time)
    threshold_single = False
    # Whether geomean exceeds aggregate test threshold (mem or time)
    threshold_agg = False
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 57-65

```python
    if args.verbose:
        print(f"# Individual test threshold: +-{args.threshold_single}%")
        print(f"# Aggregate (geomean) test threshold: +-{args.threshold_agg}%")
        print(
            f"# Checking time swings for tests with runtime >"
            f"{args.check_longer_than}s - otherwise marked as ?"
        )
        print("Test/binary BOLT_wall_time BOLT_max_rss")
```

- EN: Declares or implements routines including `print`. Notable symbols here include `print`.
- CN: 这里声明或实现函数，例如 `print`。这里较值得关注的符号包括 `print`。

### Lines 66-83

```python
    for input_file in args.input:
        input_dir = os.path.dirname(input_file)
        with open(input_file) as timing_file:
            timing_reader = csv.reader(timing_file, delimiter=";")
            for row in timing_reader:
                test_name = row[0]
                m = TIMING_LOG_RE.match(row[0])
                if m:
                    test_name = f"{input_dir}/{m.groups()[1]}/{m.groups()[2]}"
                else:
                    # Prepend input dir to unparsed test name
                    test_name = input_dir + "#" + test_name
                time_a, time_b = float(row[1]), float(row[3])
                mem_a, mem_b = int(row[2]), int(row[4])
                # Check if time is above bound for at least one side
                time_above_bound = any(
                    [x > args.check_longer_than for x in [time_a, time_b]]
                )
```

- EN: Declares or implements routines including `float`, `int`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `float`, `int`.
- CN: 这里声明或实现函数，例如 `float`, `int`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `float`, `int`。

### Lines 84-101

```python
                # Compute B/A ratios (for % delta and geomean)
                time_ratio = time_b / time_a if time_a else float('nan')
                mem_ratio = mem_b / mem_a if mem_a else float('nan')
                # Keep ratios for geomean
                if time_above_bound and time_ratio > 0:  # must be >0 for gmean
                    time_ratios += [time_ratio]
                mem_ratios += [mem_ratio]
                # Deltas: (B/A)-1 = (B-A)/A
                time_delta = time_ratio - 1
                mem_delta = mem_ratio - 1
                # Check individual test results vs single test threshold
                time_exc = (
                    100.0 * abs(time_delta) > args.threshold_single and time_above_bound
                )
                mem_exc = 100.0 * abs(mem_delta) > args.threshold_single
                if time_exc or mem_exc:
                    threshold_single = True
                # Print deltas with formatting in verbose mode
```

- EN: Declares or implements routines including `float`, `abs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `float`, `abs`.
- CN: 这里声明或实现函数，例如 `float`, `abs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `float`, `abs`。

### Lines 102-119

```python
                if args.verbose or time_exc or mem_exc:
                    print(
                        test_name,
                        fmt_delta(time_delta, time_exc, time_above_bound),
                        fmt_delta(mem_delta, mem_exc),
                    )

    time_gmean_delta = geometric_mean(time_ratios) - 1
    mem_gmean_delta = geometric_mean(mem_ratios) - 1
    time_agg_threshold = 100.0 * abs(time_gmean_delta) > args.threshold_agg
    mem_agg_threshold = 100.0 * abs(mem_gmean_delta) > args.threshold_agg
    if time_agg_threshold or mem_agg_threshold:
        threshold_agg = True
    if time_agg_threshold or mem_agg_threshold or args.verbose:
        print(
            "Geomean",
            fmt_delta(time_gmean_delta, time_agg_threshold),
            fmt_delta(mem_gmean_delta, mem_agg_threshold),
```

- EN: Declares or implements routines including `fmt_delta`, `geometric_mean`, `abs`. Notable symbols here include `fmt_delta`, `geometric_mean`, `abs`.
- CN: 这里声明或实现函数，例如 `fmt_delta`, `geometric_mean`, `abs`。这里较值得关注的符号包括 `fmt_delta`, `geometric_mean`, `abs`。

### Lines 120-125

```python
        )
    exit(threshold_single or threshold_agg)


if __name__ == "__main__":
    main()
```

- EN: Declares or implements routines including `exit`, `main`. Notable symbols here include `exit`, `main`.
- CN: 这里声明或实现函数，例如 `exit`, `main`。这里较值得关注的符号包括 `exit`, `main`。

## Key Concepts / 关键概念

- `format`: function or method entry point / 函数或方法入口
- `print`: function or method entry point / 函数或方法入口
- `float`: function or method entry point / 函数或方法入口
- `int`: function or method entry point / 函数或方法入口
- `abs`: function or method entry point / 函数或方法入口
- `argparse`: imported dependency / 导入的依赖
- `csv`: imported dependency / 导入的依赖

## Dependencies / 依赖关系

- Python imports / Python 导入: `argparse`, `csv`, `re`, `sys`, `os`, `from statistics import geometric_mean`
- Directory context / 目录上下文: `bolt/utils` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/utils` 下的相邻文件通常与本文件协作组成对应子系统
