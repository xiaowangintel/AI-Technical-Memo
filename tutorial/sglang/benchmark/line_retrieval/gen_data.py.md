# gen_data.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/line_retrieval/gen_data.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on line retrieval gen data. It primarily builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and orchestrates external commands and log collection. / 该 Python 模块聚焦于 line retrieval gen data 相关流程。它主要用于构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、编排外部命令并收集日志。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and setup / 导入与初始化
```python
"""
Generate line data for line retrieval task.

Usage:
python3 gen_data.py --number 1000
"""

import argparse
import json
from collections import defaultdict

import numpy as np
from tqdm import tqdm
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and orchestrates external commands and log collection.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、编排外部命令并收集日志。

### Lines 16-120: Function `generate_lines` / 函数 `generate_lines`
```python
def generate_lines(random_words, num_lines, redirect_ratio):
    prefix = "Here is a list of lines, each with its corresponding REGISTER_CONTENT value. Please memorize them. Be prepared to provide the REGISTER_CONTENT value for a specific line index when I ask."
    suffix = "The list has ended. Please give the final REGISTER_CONTENT value for a specific line after resolving the redirections and references. For example, the REGISTER_CONTENT of Line __idx0__ is __val0__. The REGISTER_CONTENT of Line __idx1__ is __val1__. The REGISTER_CONTENT of Line __idx2__ is __val2__. The REGISTER_CONTENT of Line ??? is"

    # Raw lines
    visited_indices = set([None])
    visited_values = set([None])

    lines = []
    redirects = []
    indices = []
    values = []
    for i in tqdm(range(num_lines)):
        line_index = None
        while line_index in visited_indices:
            line_index = "-".join(np.random.choice(random_words, size=(2,)))
        visited_indices.add(line_index)

        line_value = np.random.randint(low=0, high=999999)
        line_value = f"{line_value:06}"

        line = f"Line {line_index}: The REGISTER_CONTENT is {line_value}."
        lines.append(line)
        redirects.append(None)
        indices.append(line_index)
        values.append(line_value)

    # Add redirect
    if redirect_ratio > 0:
        num_redirect_lines = int(len(lines) * redirect_ratio)
        redirect_indices = np.random.choice(
            np.arange(len(lines)), size=(num_redirect_lines,), replace=False
        )
        for i in redirect_indices:
            target_idx = np.random.choice(min(i * 2 + 100, num_lines))
            lines[i] = (
                f"Line {indices[i]}: The REGISTER_CONTENT is the same as Line {indices[target_idx]}."
            )
            redirects[i] = target_idx

    # Build links and find sources
    links = [[] for _ in range(num_lines)]
    contains_ring = set()
    for i in range(num_lines):
        if redirects[i] is None:
            continue

        tmp_link = []
        cur = i
        visited = set()
        while redirects[cur] is not None:
            visited.add(cur)
            tmp_link.append(redirects[cur])
            cur = redirects[cur]

            if cur in visited:
                contains_ring.add(i)
                tmp_link = None
                break
        values[i] = values[cur]
        links[i] = tmp_link

    # Group by num_links
    group_by_num_hoops = defaultdict(list)
    for i in range(num_lines):
        if i in contains_ring:
            continue
        group_by_num_hoops[len(links[i]) + 1].append(i)

    keys = sorted(list(group_by_num_hoops.keys()))
    for num_links in keys:
        print(f"#links: {num_links}, #lines: {len(group_by_num_hoops[num_links])}")

    # Append few-shot examples
    hoop1_candidates = list(group_by_num_hoops[1])
    hoop1_candidate_keys = {c: max([c] + links[c]) for c in hoop1_candidates}
    hoop1_candidates.sort(key=lambda c: hoop1_candidate_keys[c])
    hoop2_candidates = list(group_by_num_hoops[2])
    hoop2_candidate_keys = {c: max([c] + links[c]) for c in hoop2_candidates}
    hoop2_candidates.sort(key=lambda c: hoop2_candidate_keys[c])

    i = hoop1_candidates[5]
    suffix = suffix.replace("__idx0__", indices[i]).replace("__val0__", values[i])
    if len(hoop2_candidates):
        i = hoop2_candidates[0]
        suffix = suffix.replace("__idx1__", indices[i]).replace("__val1__", values[i])
        i = hoop2_candidates[1]
        suffix = suffix.replace("__idx2__", indices[i]).replace("__val2__", values[i])
    else:
        i = hoop1_candidates[1]
        suffix = suffix.replace("__idx1__", indices[i]).replace("__val1__", values[i])
        i = hoop1_candidates[10]
        suffix = suffix.replace("__idx2__", indices[i]).replace("__val2__", values[i])

    obj = {
        "prefix": prefix,
        "suffix": suffix,
        "lines": lines,
        "indices": indices,
        "values": values,
        "links": links,
        "group_by_num_hoops": group_by_num_hoops,
        "contains_ring": sorted(list(contains_ring)),
    }
    return obj
```
**EN:** `generate_lines` is a function that implements the core logic for this scope. It returns `obj` to the caller. Notable calls include `set`, `tqdm`, `range`.
**CN:** `generate_lines` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `obj`。其中较关键的调用包括 `set`, `tqdm`, `range`。

### Lines 123-139: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--number", type=int)
    parser.add_argument("--redirect-ratio", type=float, default=0.0)
    args = parser.parse_args()

    num_lines = args.number

    random_words_filename = "random_words.json"
    random_words = json.load(open(random_words_filename, "r"))

    np.random.seed(42)
    obj = generate_lines(random_words, num_lines, args.redirect_ratio)

    fout = f"lines_{num_lines}_{args.redirect_ratio:.1f}.json"
    with open(fout, "w") as fout:
        json.dump(obj, fout, indent=2)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, loads, filters, or serializes benchmark datasets, and reads or writes local files and intermediate artifacts.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `json`, `collections`
- **Third-party / 第三方依赖**: `numpy`, `tqdm`
