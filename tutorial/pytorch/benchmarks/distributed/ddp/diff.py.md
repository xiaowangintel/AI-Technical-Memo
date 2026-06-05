# diff.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/distributed/ddp/diff.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: #!/usr/bin/env python3
 2: #
 3: # Computes difference between measurements produced by ./benchmark.py.
 4: #
 5: 
 6: import argparse
 7: import json
 8: 
 9: import numpy as np
10: 
11: 
12: def load(path):
13:     with open(path) as f:
14:         return json.load(f)
15: 
16: 
17: def main():
18:     parser = argparse.ArgumentParser(description="PyTorch distributed benchmark diff")
19:     parser.add_argument("file", nargs=2)
20:     args = parser.parse_args()
21: 
22:     if len(args.file) != 2:
23:         raise RuntimeError("Must specify 2 files to diff")
24: 
25:     ja = load(args.file[0])
26:     jb = load(args.file[1])
27: 
28:     keys = (set(ja.keys()) | set(jb.keys())) - {"benchmark_results"}
29:     print(f"{'':20s} {'baseline':>20s}      {'test':>20s}")
30:     print(f"{'':20s} {'-' * 20:>20s}      {'-' * 20:>20s}")
31:     for key in sorted(keys):
32:         va = str(ja.get(key, "-"))
33:         vb = str(jb.get(key, "-"))
34:         print(f"{key + ':':20s} {va:>20s}  vs  {vb:>20s}")
35:     print()
36: 
37:     ba = ja["benchmark_results"]
38:     bb = jb["benchmark_results"]
39:     for ra, rb in zip(ba, bb):
40:         if ra["model"] != rb["model"]:
````
- EN: Handles module imports such as `argparse`, `json`, `numpy`.
- CN: 处理模块导入，例如 `argparse`, `json`, `numpy`。
- EN: Implements callable logic such as `load`, `main`.
- CN: 实现可调用逻辑，例如 `load`, `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:             continue
42:         if ra["batch_size"] != rb["batch_size"]:
43:             continue
44: 
45:         model = ra["model"]
46:         batch_size = int(ra["batch_size"])
47:         name = f"{model} with batch size {batch_size}"
48:         print(f"Benchmark: {name}")
49: 
50:         # Print header
51:         print()
52:         print(f"{'':>10s}", end="")
53:         for _ in [75, 95]:
54:             print(f"{'sec/iter':>16s}{'ex/sec':>10s}{'diff':>10s}", end="")
55:         print()
56: 
57:         # Print measurements
58:         for i, (xa, xb) in enumerate(zip(ra["result"], rb["result"])):
59:             # Ignore round without ddp
60:             if i == 0:
61:                 continue
62:             # Sanity check: ignore if number of ranks is not equal
63:             if len(xa["ranks"]) != len(xb["ranks"]):
64:                 continue
65: 
66:             ngpus = len(xa["ranks"])
67:             ma = sorted(xa["measurements"])
68:             mb = sorted(xb["measurements"])
69:             print(f"{ngpus:>4d} GPUs:", end="")
70:             for p in [75, 95]:
71:                 va = np.percentile(ma, p)
72:                 vb = np.percentile(mb, p)
73:                 # We're measuring time, so lower is better (hence the negation)
74:                 delta = -100 * ((vb - va) / va)
75:                 print(
76:                     f"  p{p:02d}: {vb:8.3f}s {int(batch_size / vb):7d}/s {delta:+8.1f}%",
77:                     end="",
78:                 )
79:             print()
80:         print()
````
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-84
````python
81: 
82: 
83: if __name__ == "__main__":
84:     main()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `load` / 符号 `load`
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `argparse`, `json`, `numpy`
- Python 导入: `argparse`, `json`, `numpy`
