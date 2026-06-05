# convert_parquet_to_json.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/boolq/convert_parquet_to_json.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on boolq convert parquet to json. It primarily loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. / 该 Python 模块聚焦于 boolq convert parquet to json 相关流程。它主要用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and setup / 导入与初始化
```python
import sys

import pyarrow.parquet as pq
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会加载、筛选或序列化基准测试数据集。

### Lines 6-18: Function `convert_parquet_to_json` / 函数 `convert_parquet_to_json`
```python
def convert_parquet_to_json(input_file, output_file):
    # read parquet file
    table = pq.read_table(input_file)

    # turn parquet data to dataframe
    df = table.to_pandas()

    # turn dataframe to json form
    json_data = df.to_json(orient="records", lines=True)

    # write json to file
    with open(output_file, "w") as f:
        f.write(json_data)
```
**EN:** `convert_parquet_to_json` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. Notable calls include `pq.read_table`, `table.to_pandas`, `df.to_json`.
**CN:** `convert_parquet_to_json` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。其中较关键的调用包括 `pq.read_table`, `table.to_pandas`, `df.to_json`。

### Lines 21-28: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage:python convert_parquet_to_json.py <input_file> <output_file>")

    input_file = sys.argv[1]
    output_file = sys.argv[2]

    convert_parquet_to_json(input_file, output_file)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it loads, filters, or serializes benchmark datasets.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会加载、筛选或序列化基准测试数据集。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `sys`
- **Third-party / 第三方依赖**: `pyarrow.parquet`
