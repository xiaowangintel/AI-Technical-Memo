# build_dataset.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/multi_document_qa/build_dataset.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on multi document qa build dataset. It primarily loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. / 该 Python 模块聚焦于 multi document qa build dataset 相关流程。它主要用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-70: Imports and setup / 导入与初始化
```python
import json

import transformers

content = "\n".join(
    open("llama2.txt", "r", encoding="utf-8", errors="ignore").readlines()
)
content = content.replace("\n\n", "\n")

# Count token
name = "meta-llama/Llama-2-7b-chat-hf"
t = transformers.AutoTokenizer.from_pretrained(name)
print(f"num tokens: {len(t.encode(content))}")

# Segment
SEP = "\n\n"
parts = content.split(SEP)
print(f"num segments: {len(parts)}")

segment_len = 1100

segments = []
tmp = []
tmp_len = 0
for i in range(len(parts)):
    tmp.append(parts[i])
    tmp_len += len(t.encode(parts[i]))

    if tmp_len > segment_len:
        segments.append(SEP.join(tmp))
        tmp = []
        tmp_len = 0

for i, s in enumerate(segments):
    print(i, len(t.encode(segments[i])))

# Dump
with open("questions.jsonl", "w") as fout:
    fout.write(
        json.dumps(
            {
                "documents": segments[:30],
                "questions": [
                    "What is the name of the fine-tuned LLMs?",
                    "Which figure shows the helpfulness human evaluation results for Llama 2-Chat?",
                    "What is the number of parameters in the largest Llama 2 model?",
                    "What is the batch size of fine-tuning?",
                    "Where can we find the details of potential data contamination?",
                    "What is the full name of MPT?",
                    "What is the power consumption of RSC in Watt?",
                    "How many tokens of data do they train on?",
                    "Which model's release is delayed due to a lack of time to sufficiently red team?",
                    "Which activation function is used in Llama?",
                ],
                "answers": [
                    "Llama 2 Chat",
                    "1",
                    "70 B",
                    "64",
                    "A 6",
                    "MosaicML",
                    "400",
                    "2 trillion",
                    "34 B",
                    "SwiGLU",
                ],
            }
        )
        + "\n"
    )
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it loads, filters, or serializes benchmark datasets, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`
- **Third-party / 第三方依赖**: `transformers`
