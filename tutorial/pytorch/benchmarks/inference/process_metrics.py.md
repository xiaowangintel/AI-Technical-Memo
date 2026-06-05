# process_metrics.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/inference/process_metrics.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
````python
 1: """
 2: This file will take the csv outputs from server.py, calculate the mean and
 3: variance of the warmup_latency, average_latency, throughput and gpu_util
 4: and write these to the corresponding `results/output_{batch_size}_{compile}.md`
 5: file, appending to the file if it exists or creatng a new one otherwise.
 6: """
 7: 
 8: import argparse
 9: import os
10: 
11: import pandas as pd
12: 
13: 
14: if __name__ == "__main__":
15:     parser = argparse.ArgumentParser(description="Parse output files")
16:     parser.add_argument("--csv", type=str, help="Path to csv file")
17:     parser.add_argument("--name", type=str, help="Name of experiment")
18:     args = parser.parse_args()
19: 
20:     input_csv = "./results/" + args.csv
````
- EN: Handles module imports such as `argparse`, `os`, `pandas`.
- CN: 处理模块导入，例如 `argparse`, `os`, `pandas`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 21-40
````python
21:     df = pd.read_csv(input_csv)
22: 
23:     batch_size = int(os.path.basename(args.csv).split("_")[1])
24:     compile = os.path.basename(args.csv).split("_")[-1].split(".")[0]
25: 
26:     # Calculate mean and standard deviation for a subset of metrics
27:     metrics = ["warmup_latency", "average_latency", "throughput", "gpu_util"]
28:     means = {}
29:     stds = {}
30: 
31:     for metric in metrics:
32:         means[metric] = df[metric].mean()
33:         stds[metric] = df[metric].std()
34: 
35:     output_md = f"results/output_{batch_size}_{compile}.md"
36:     write_header = os.path.isfile(output_md) is False
37: 
38:     with open(output_md, "a+") as f:
39:         if write_header:
40:             f.write(f"## Batch Size {batch_size} Compile {compile}\n\n")
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-51
````python
41:             f.write(
42:                 "| Experiment | Warmup_latency (s) | Average_latency (s) | Throughput (samples/sec) | GPU Utilization (%) |\n"
43:             )
44:             f.write(
45:                 "| ---------- | ------------------ | ------------------- | ------------------------ | ------------------- |\n"
46:             )
47: 
48:         line = f"| {args.name} |"
49:         for metric in metrics:
50:             line += f" {means[metric]:.3f} +/- {stds[metric]:.3f} |"
51:         f.write(line + "\n")
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试

## Dependencies / 依赖关系
- Python imports: `argparse`, `os`, `pandas`
- Python 导入: `argparse`, `os`, `pandas`
