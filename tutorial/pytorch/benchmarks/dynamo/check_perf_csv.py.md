# check_perf_csv.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/dynamo/check_perf_csv.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: import argparse
 2: import sys
 3: import textwrap
 4: 
 5: import pandas as pd
 6: 
 7: 
 8: def check_perf_csv(filename, threshold, threshold_scale):
 9:     """
10:     Basic performance checking.
11:     """
12:     try:
13:         df = pd.read_csv(filename)
14:     except FileNotFoundError:
15:         print(f"Error: File {filename} not found")
16:         sys.exit(1)
17: 
18:     effective_threshold = threshold * threshold_scale
19:     print(f"Checking {filename} (speedup threshold >= {effective_threshold:.2f}x)\n")
20: 
21:     failed = []
22:     for _, row in df.iterrows():
23:         model_name = row["name"]
24:         speedup = float(row["speedup"])
25:         abs_latency = float(row["abs_latency"])
26:         compilation_latency = float(row["compilation_latency"])
27:         compression_ratio = float(row["compression_ratio"])
28:         eager_peak_mem = float(row["eager_peak_mem"])
29:         dynamo_peak_mem = float(row["dynamo_peak_mem"])
30: 
31:         perf_summary = f"{model_name:34} speedup={speedup:.3f}x"
32:         if pd.notna(abs_latency):
33:             perf_summary += f", latency={abs_latency:.1f} ms/iter"
34:         if pd.notna(compilation_latency):
35:             perf_summary += f", compile={compilation_latency:.3f}s"
36:         if pd.notna(compression_ratio):
37:             perf_summary += f", mem_ratio={1 / compression_ratio:.2f}x"
38:             if pd.notna(eager_peak_mem) and pd.notna(dynamo_peak_mem):
39:                 perf_summary += (
40:                     f" (eager={eager_peak_mem:.1f} GB, dynamo={dynamo_peak_mem:.1f} GB)"
````
- EN: Handles module imports such as `argparse`, `sys`, `textwrap`, `pandas`.
- CN: 处理模块导入，例如 `argparse`, `sys`, `textwrap`, `pandas`。
- EN: Implements callable logic such as `check_perf_csv`.
- CN: 实现可调用逻辑，例如 `check_perf_csv`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:                 )
42: 
43:         if speedup < effective_threshold:
44:             failed.append((model_name, speedup))
45: 
46:         print(perf_summary)
47: 
48:     if failed:
49:         print(
50:             textwrap.dedent(
51:                 f"""
52:                 Error {len(failed)} model(s) performance regressed
53:                     {" ".join([name for name, _ in failed])}
54:                 """
55:             )
56:         )
57:         for name, sp in sorted(failed, key=lambda x: x[1]):
58:             pct_from_target = (sp / effective_threshold - 1.0) * 100.0
59:             print(
60:                 f"  - {name}: {sp:.3f}x (< {effective_threshold:.2f}x; {pct_from_target:.1f}% from target)"
61:             )
62:         sys.exit(1)
63:     else:
64:         print(
65:             f"\nAll {len(df)} model(s) passed threshold check (>= {effective_threshold:.2f}x)"
66:         )
67: 
68: 
69: if __name__ == "__main__":
70:     parser = argparse.ArgumentParser()
71:     parser.add_argument("--file", "-f", type=str, help="csv file name")
72:     parser.add_argument(
73:         "--threshold", "-t", type=float, help="threshold speedup value to check against"
74:     )
75:     parser.add_argument(
76:         "--threshold-scale",
77:         "-s",
78:         type=float,
79:         default=1.0,
80:         help="multiply threshold by this value to relax the check",
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-83
````python
81:     )
82:     args = parser.parse_args()
83:     check_perf_csv(args.file, args.threshold, args.threshold_scale)
````
- EN: This range contributes implementation details for the file goal: Implements benchmark definitions, helpers, or runners for performance measurement.
- CN: 该范围为文件目标提供实现细节：实现用于性能测量的基准定义、辅助工具或运行器。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `check_perf_csv` / 符号 `check_perf_csv`

## Dependencies / 依赖关系
- Python imports: `argparse`, `sys`, `textwrap`, `pandas`
- Python 导入: `argparse`, `sys`, `textwrap`, `pandas`
