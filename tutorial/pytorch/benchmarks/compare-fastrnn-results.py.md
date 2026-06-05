# compare-fastrnn-results.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/compare-fastrnn-results.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
````python
 1: import argparse
 2: import json
 3: from collections import namedtuple
 4: 
 5: 
 6: Result = namedtuple("Result", ["name", "base_time", "diff_time"])
 7: 
 8: 
 9: def construct_name(fwd_bwd, test_name):
10:     bwd = "backward" in fwd_bwd
11:     suite_name = fwd_bwd.replace("-backward", "")
12:     return f"{suite_name}[{test_name}]:{'bwd' if bwd else 'fwd'}"
13: 
14: 
15: def get_times(json_data):
16:     r = {}
17:     for fwd_bwd in json_data:
18:         for test_name in json_data[fwd_bwd]:
19:             name = construct_name(fwd_bwd, test_name)
20:             r[name] = json_data[fwd_bwd][test_name]
````
- EN: Handles module imports such as `argparse`, `json`, `collections`.
- CN: 处理模块导入，例如 `argparse`, `json`, `collections`。
- EN: Implements callable logic such as `construct_name`, `get_times`.
- CN: 实现可调用逻辑，例如 `construct_name`, `get_times`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 21-40
````python
21:     return r
22: 
23: 
24: parser = argparse.ArgumentParser("compare two pytest jsons")
25: parser.add_argument("base", help="base json file")
26: parser.add_argument("diff", help="diff json file")
27: parser.add_argument(
28:     "--format", default="md", type=str, help="output format (csv, md, json, table)"
29: )
30: args = parser.parse_args()
31: 
32: with open(args.base) as base:
33:     base_times = get_times(json.load(base))
34: with open(args.diff) as diff:
35:     diff_times = get_times(json.load(diff))
36: 
37: all_keys = set(base_times.keys()).union(diff_times.keys())
38: results = [
39:     Result(name, base_times.get(name, float("nan")), diff_times.get(name, float("nan")))
40:     for name in sorted(all_keys)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 41-60
````python
41: ]
42: 
43: header_fmt = {
44:     "table": "{:48s} {:>13s} {:>15s} {:>10s}",
45:     "md": "| {:48s} | {:>13s} | {:>15s} | {:>10s} |",
46:     "csv": "{:s}, {:s}, {:s}, {:s}",
47: }
48: data_fmt = {
49:     "table": "{:48s} {:13.6f} {:15.6f} {:9.1f}%",
50:     "md": "| {:48s} | {:13.6f} | {:15.6f} | {:9.1f}% |",
51:     "csv": "{:s}, {:.6f}, {:.6f}, {:.2f}%",
52: }
53: 
54: if args.format in ["table", "md", "csv"]:
55:     header_fmt_str = header_fmt[args.format]
56:     data_fmt_str = data_fmt[args.format]
57:     print(header_fmt_str.format("name", "base time (s)", "diff time (s)", "% change"))
58:     if args.format == "md":
59:         print(header_fmt_str.format(":---", "---:", "---:", "---:"))
60:     for r in results:
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-72
````python
61:         print(
62:             data_fmt_str.format(
63:                 r.name,
64:                 r.base_time,
65:                 r.diff_time,
66:                 (r.diff_time / r.base_time - 1.0) * 100.0,
67:             )
68:         )
69: elif args.format == "json":
70:     print(json.dumps(results))
71: else:
72:     raise ValueError("Unknown output format: " + args.format)
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `construct_name` / 符号 `construct_name`
- Symbol `get_times` / 符号 `get_times`

## Dependencies / 依赖关系
- Python imports: `argparse`, `json`, `collections`
- Python 导入: `argparse`, `json`, `collections`
