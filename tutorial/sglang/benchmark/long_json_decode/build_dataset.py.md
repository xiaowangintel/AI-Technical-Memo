# build_dataset.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/long_json_decode/build_dataset.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on long json decode build dataset. It primarily loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and reads or writes local files and intermediate artifacts. / 该 Python 模块聚焦于 long json decode build dataset 相关流程。它主要用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、读写本地文件及中间产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Imports and setup / 导入与初始化
```python
import json

import transformers
import wikipedia

name = "meta-llama/Llama-2-7b-chat-hf"
t = transformers.AutoTokenizer.from_pretrained(name)
city_names = ["los angles", "london", "tokyo", "beijing", "singapore"]


for city_name in city_names:
    content = str(wikipedia.page(city_name).content)
    content = content.replace("\n\n", "\n")

    tokens = t.encode(content)

    truncate_len = int((10000 / len(tokens)) * len(content))
    truncate_content = content[:truncate_len]
    truncate_tokens = t.encode(truncate_content)

    # Count token
    print(
        f"city_name: {city_name}, #tokens: {len(tokens)}, #truncate tokens: {len(truncate_tokens)}"
    )

    with open("questions.jsonl", "a") as fout:
        fout.write(json.dumps({"document": truncate_content}) + "\n")
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and reads or writes local files and intermediate artifacts.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、读写本地文件及中间产物。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Third-party / 第三方依赖**: `transformers`, `wikipedia`
