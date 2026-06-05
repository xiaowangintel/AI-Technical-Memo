# build_dataset.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/json_decode_regex/build_dataset.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on json decode regex build dataset. It primarily loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and reads or writes local files and intermediate artifacts. / 该 Python 模块聚焦于 json decode regex build dataset 相关流程。它主要用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、读写本地文件及中间产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and setup / 导入与初始化
```python
import json

import transformers
import wikipedia

model_path = "meta-llama/Llama-2-7b-chat-hf"
t = transformers.AutoTokenizer.from_pretrained(model_path)
city_names = [
    "los angles",
    "london",
    "tokyo",
    "beijing",
    "singapore",
    "paris",
    "dubai",
    "sydney",
    "moscow",
    "rome",
    "toronto",
    "rio de janeiro",
    "istanbul",
    "berlin",
    "auckland",
    "buenos aires",
    "mexico city",
    "mumbai",
    "seoul",
    "bangkok",
    "cairo",
    "athens",
    "jerusalem",
]
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。

### Lines 35-51: Function `get_content` / 函数 `get_content`
```python
def get_content(city_name):
    content = str(wikipedia.page(city_name).content)
    content = content.replace("\n\n", "\n")

    tokens = t.encode(content)

    expected_tokens = 3000
    truncate_len = int((expected_tokens / len(tokens)) * len(content))
    truncate_content = content[:truncate_len]
    truncate_tokens = t.encode(truncate_content)

    # Count token
    print(
        f"city_name: {city_name}, #tokens: {len(tokens)}, #truncate tokens: {len(truncate_tokens)}"
    )

    return truncate_content
```
**EN:** `get_content` is a function that processes tokenized prompts or decoded outputs. It returns `truncate_content` to the caller. Notable calls include `str`, `content.replace`, `t.encode`.
**CN:** `get_content` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `truncate_content`。其中较关键的调用包括 `str`, `content.replace`, `t.encode`。

### Lines 54-58: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    with open("questions.jsonl", "w") as fout:
        for city_name in city_names:
            truncate_content = get_content(city_name)
            fout.write(json.dumps({"document": truncate_content}) + "\n")
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Third-party / 第三方依赖**: `transformers`, `wikipedia`
