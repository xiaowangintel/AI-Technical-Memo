# colbert_rerank_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/score/colbert_rerank_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example of using ColBERT late interaction models for reranking and scoring / 演示打分与重排序流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example of using ColBERT late interaction models for reranking and scoring.

ColBERT (Contextualized Late Interaction over BERT) uses per-token embeddings
and MaxSim scoring for document reranking, providing better accuracy than
single-vector models while being more efficient than cross-encoders.

vLLM supports ColBERT with multiple encoder backbones. Start the server
with one of the following:

    # BERT backbone (works out of the box)
    vllm serve answerdotai/answerai-colbert-small-v1

    # ModernBERT backbone
    vllm serve lightonai/GTE-ModernColBERT-v1 \
        --hf-overrides '{"architectures": ["ColBERTModernBertModel"]}'

    # Jina XLM-RoBERTa backbone
    vllm serve jinaai/jina-colbert-v2 \
        --hf-overrides '{"architectures": ["ColBERTJinaRobertaModel"]}' \
        --trust-remote-code

Then run this script:
    python colbert_rerank_online.py
"""
```
**EN:** Example of using ColBERT late interaction models for reranking and scoring.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import json

import requests
```
**EN:** This block loads helper libraries such as json and requests.
**CN:** 这一部分加载 json 和 requests 等辅助库。

### Top-level setup
```python
MODEL = "answerdotai/answerai-colbert-small-v1"
BASE_URL = "http://127.0.0.1:8000"

headers = {"accept": "application/json", "Content-Type": "application/json"}

documents = [
    "Machine learning is a subset of artificial intelligence.",
    "Python is a programming language.",
    "Deep learning uses neural networks for complex tasks.",
    "The weather today is sunny.",
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as MODEL, BASE_URL, headers, and documents.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 MODEL、BASE_URL、headers，以及 documents 等变量。

### Function: rerank_example
```python
def rerank_example():
    """Use the /rerank endpoint to rank documents by query relevance."""
    print("=== Rerank Example ===")

    data = {
        "model": MODEL,
        "query": "What is machine learning?",
        "documents": documents,
    }

    response = requests.post(f"{BASE_URL}/rerank", headers=headers, json=data)
    result = response.json()
    print(json.dumps(result, indent=2))

    print("\nRanked documents (most relevant first):")
    for item in result["results"]:
        doc_idx = item["index"]
        score = item["relevance_score"]
        print(f"  Score {score:.4f}: {documents[doc_idx]}")
```
**EN:** Use the /rerank endpoint to rank documents by query relevance.. Key operations include print, requests.post, response.json, and json.dumps.
**CN:** 该函数编排端到端工作流。关键操作包括 print、requests.post、response.json，以及 json.dumps。

### Function: score_example
```python
def score_example():
    """Use the /score endpoint for pairwise query-document scoring."""
    print("\n=== Score Example ===")

    data = {
        "model": MODEL,
        "text_1": "What is machine learning?",
        "text_2": [
            "Machine learning is a subset of AI.",
            "The weather is sunny.",
        ],
    }

    response = requests.post(f"{BASE_URL}/score", headers=headers, json=data)
    result = response.json()
    print(json.dumps(result, indent=2))
```
**EN:** Use the /score endpoint for pairwise query-document scoring.. Key operations include print, requests.post, response.json, and json.dumps.
**CN:** 该函数编排端到端工作流。关键操作包括 print、requests.post、response.json，以及 json.dumps。

### Function: main
```python
def main():
    rerank_example()
    score_example()
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include rerank_example and score_example.
**CN:** 该函数编排端到端工作流。关键操作包括 rerank_example 和 score_example。

### Entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。
- **Task-specific scoring / 任务特定打分**: Model outputs are converted into scores, labels, or reward values. / 模型输出会被转换为分数、标签或奖励值。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `json`, `requests` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `rerank_example`, `score_example`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `requests.post`, `response.json`, `json.dumps`, `rerank_example`, `score_example`, `main` reveal the main execution path / 这些调用体现了主要执行链路。
