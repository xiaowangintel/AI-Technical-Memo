# download_data.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `benchmark/mmlu/download_data.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script targets mmlu download data. It primarily launches experiments and captures logs. / 该 shell 脚本面向 mmlu download data。它主要用于启动实验并收集日志。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Shell execution block / Shell 执行块
```bash
wget https://people.eecs.berkeley.edu/~hendrycks/data.tar
tar xf data.tar
```
**EN:** This block groups related shell commands and environment variables for one experiment stage.
**CN:** 该代码块把同一实验阶段相关的 shell 命令与环境变量组织在一起。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。

## Dependencies / 依赖关系
- **Commands / 命令**: `wget`, `tar`
