# using_template_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/score/using_template_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example of using the rerank API with template / 演示打分与重排序流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example of using the rerank API with template.

This script demonstrates how to interact with a vLLM server running
a reranking model via the REST API.
Before running this script, start the vLLM server with one of the
supported reranking models using the commands below.

note:
    Some reranking models require special configuration overrides to work correctly
    with vLLM's score API.
    Reference: https://github.com/vllm-project/vllm/blob/main/examples/pooling/score/qwen3_reranker_online.py
    Reference: https://github.com/vllm-project/vllm/blob/main/examples/pooling/score/convert_model_to_seq_cls.py

run:
    vllm serve BAAI/bge-reranker-v2-gemma --hf_overrides '{"architectures": ["GemmaForSequenceClassification"],"classifier_from_token": ["Yes"],"method": "no_post_processing"}' --chat-template examples/pooling/score/template/bge-reranker-v2-gemma.jinja
    vllm serve tomaarsen/Qwen3-Reranker-0.6B-seq-cls --chat-template examples/pooling/score/template/qwen3_reranker.jinja
    vllm serve mixedbread-ai/mxbai-rerank-base-v2 --hf_overrides '{"architectures": ["Qwen2ForSequenceClassification"],"classifier_from_token": ["0", "1"], "method": "from_2_way_softmax"}' --chat-template examples/pooling/score/template/mxbai_rerank_v2.jinja
    vllm serve nvidia/llama-nemotron-rerank-1b-v2 --runner pooling --trust-remote-code --chat-template examples/pooling/score/template/nemotron-rerank.jinja
    vllm serve Qwen/Qwen3-Reranker-0.6B --runner pooling --hf_overrides '{"architectures": ["Qwen3ForSequenceClassification"],"classifier_from_token": ["no", "yes"],"is_original_qwen3_reranker": true}' --chat-template examples/pooling/score/template/qwen3_reranker.jinja
"""
```
**EN:** Example of using the rerank API with template.
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
url = "http://127.0.0.1:8000/rerank"

# HTTP headers for the request
headers = {"accept": "application/json", "Content-Type": "application/json"}

# Example query & documents
query = "how much protein should a female eat?"
documents = [
    "As a general guideline, the CDC's average requirement of protein for women ages 19 to 70 is 46 grams per day. But, as you can see from this chart, you'll need to increase that if you're expecting or training for a marathon. Check out the chart below to see how much protein you should be eating each day.",
    "Definition of summit for English Language Learners. : 1  the highest point of a mountain : the top of a mountain. : 2  the highest level. : 3  a meeting or series of meetings between the leaders of two or more governments.",
    "Calorie intake should not fall below 1,200 a day in women or 1,500 a day in men, except under the supervision of a health professional.",
]

# Request payload for the rerank API
data = {
    "model": "nvidia/llama-nemotron-rerank-1b-v2",  # Model to use for reranking
    "query": query,  # The query to score documents against
    "documents": documents,  # List of documents to be scored
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as url, headers, query, documents, and data.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 url、headers、query、documents，以及 data 等变量。

### Function: main
```python
def main():
    """Main function to send a rerank request to the vLLM server.

    This function sends a POST request to the /rerank endpoint with
    the query and documents, then prints the relevance scores.
    """
    # Send POST request to the vLLM server's rerank endpoint
    response = requests.post(url, headers=headers, json=data)

    # Check if the request was successful
    if response.status_code == 200:
        print("Request successful!")
        # Pretty print the JSON response containing relevance scores
        # The response includes scores for each document's relevance to the query
        print(json.dumps(response.json(), indent=2))
    else:
        # Handle request failure
        print(f"Request failed with status code: {response.status_code}")
        print(response.text)
```
**EN:** Main function to send a rerank request to the vLLM server.. Key operations include print, requests.post, json.dumps, and response.json.
**CN:** 该函数编排端到端工作流。关键操作包括 print、requests.post、json.dumps，以及 response.json。

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
- **Internal structure / 内部结构**: top-level blocks such as `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `requests.post`, `json.dumps`, `response.json`, `main` reveal the main execution path / 这些调用体现了主要执行链路。
