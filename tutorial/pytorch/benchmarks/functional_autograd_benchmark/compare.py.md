# compare.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/functional_autograd_benchmark/compare.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
````python
 1: import argparse
 2: from collections import defaultdict
 3: 
 4: from utils import from_markdown_table, to_markdown_table
 5: 
 6: 
 7: def main():
 8:     parser = argparse.ArgumentParser(
 9:         "Main script to compare results from the benchmarks"
10:     )
11:     parser.add_argument(
12:         "--before",
13:         type=str,
14:         default="before.txt",
15:         help="Text file containing the times to use as base",
16:     )
17:     parser.add_argument(
18:         "--after",
19:         type=str,
20:         default="after.txt",
````
- EN: Handles module imports such as `argparse`, `collections`, `utils`.
- CN: 处理模块导入，例如 `argparse`, `collections`, `utils`。
- EN: Implements callable logic such as `main`.
- CN: 实现可调用逻辑，例如 `main`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。

### Lines 21-40
````python
21:         help="Text file containing the times to use as new version",
22:     )
23:     parser.add_argument(
24:         "--output", type=str, default="", help="Text file where to write the output"
25:     )
26:     args = parser.parse_args()
27: 
28:     with open(args.before) as f:
29:         content = f.read()
30:     res_before = from_markdown_table(content)
31: 
32:     with open(args.after) as f:
33:         content = f.read()
34:     res_after = from_markdown_table(content)
35: 
36:     diff = defaultdict(defaultdict)
37:     for model in res_before:
38:         for task in res_before[model]:
39:             mean_before, var_before = res_before[model][task]
40:             if task not in res_after[model]:
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-60
````python
41:                 diff[model][task] = (None, mean_before, var_before, None, None)
42:             else:
43:                 mean_after, var_after = res_after[model][task]
44:                 diff[model][task] = (
45:                     mean_before / mean_after,
46:                     mean_before,
47:                     var_before,
48:                     mean_after,
49:                     var_after,
50:                 )
51:     for model in res_after:
52:         for task in res_after[model]:
53:             if task not in res_before[model]:
54:                 mean_after, var_after = res_after[model][task]
55:                 diff[model][task] = (None, None, None, mean_after, var_after)
56: 
57:     header = (
58:         "model",
59:         "task",
60:         "speedup",
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-75
````python
61:         "mean (before)",
62:         "var (before)",
63:         "mean (after)",
64:         "var (after)",
65:     )
66:     out = to_markdown_table(diff, header=header)
67: 
68:     print(out)
69:     if args.output:
70:         with open(args.output, "w") as f:
71:             f.write(out)
72: 
73: 
74: if __name__ == "__main__":
75:     main()
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `main` / 符号 `main`

## Dependencies / 依赖关系
- Python imports: `argparse`, `collections`, `utils`
- Python 导入: `argparse`, `collections`, `utils`
