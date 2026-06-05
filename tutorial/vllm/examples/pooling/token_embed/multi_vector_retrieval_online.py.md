# multi_vector_retrieval_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/token_embed/multi_vector_retrieval_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example online usage of Pooling API for multi vector retrieval / 演示词元级嵌入提取。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example online usage of Pooling API for multi vector retrieval.

Run `vllm serve <model> --runner pooling`
to start up the server in vLLM. e.g.

vllm serve BAAI/bge-m3 --pooler-config.task token_embed
"""
```
**EN:** Example online usage of Pooling API for multi vector retrieval.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import pprint

import requests
import torch
```
**EN:** This block loads helper libraries such as argparse, pprint, requests, and torch.
**CN:** 这一部分加载 argparse、pprint、requests，以及 torch 等辅助库。

### Function: post_http_request
```python
def post_http_request(prompt: dict, api_url: str) -> requests.Response:
    headers = {"User-Agent": "Test Client"}
    response = requests.post(api_url, headers=headers, json=prompt)
    return response
```
**EN:** This function sends a request and handles the reply. It works with parameters such as prompt and api_url. Key operations include requests.post. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。它会处理 prompt 和 api_url 等参数。关键操作包括 requests.post。其返回值会继续传给示例管线的下一阶段。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", type=str, default="localhost")
    parser.add_argument("--port", type=int, default=8000)
    parser.add_argument("--model", type=str, default="BAAI/bge-m3")

    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    pooling_url = f"http://{args.host}:{args.port}/pooling"
    score_url = f"http://{args.host}:{args.port}/score"
    model_name = args.model

    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]
    prompt = {"model": model_name, "input": prompts}

    pooling_response = post_http_request(prompt=prompt, api_url=pooling_url)
    for output in pooling_response.json()["data"]:
        multi_vector = torch.tensor(output["data"])
        print(multi_vector.shape)

    queries = "What is the capital of France?"
    documents = [
        "The capital of Brazil is Brasilia.",
        "The capital of France is Paris.",
    ]
    prompt = {"model": model_name, "queries": queries, "documents": documents}
    score_response = post_http_request(prompt=prompt, api_url=score_url)
    print("\nPrompt when queries is string and documents is a list:")
    pprint.pprint(prompt)
    print("\nScore Response:")
    pprint.pprint(score_response.json())
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, post_http_request, pprint.pprint, pooling_response.json, and torch.tensor.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、post_http_request、pprint.pprint、pooling_response.json，以及 torch.tensor。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    main(args)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。
- **Task-specific scoring / 任务特定打分**: Model outputs are converted into scores, labels, or reward values. / 模型输出会被转换为分数、标签或奖励值。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `argparse`, `pprint`, `requests`, `torch` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `post_http_request`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `requests.post`, `parser.add_argument`, `argparse.ArgumentParser`, `parser.parse_args`, `print`, `post_http_request`, `pprint.pprint`, `pooling_response.json` reveal the main execution path / 这些调用体现了主要执行链路。
