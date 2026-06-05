# classification_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/classify/classification_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example Python client for classification API using vLLM API server NOTE: start a supported classification model server with `vllm serve`, e.g / 演示池化模型的推理模式。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""Example Python client for classification API using vLLM API server
NOTE:
    start a supported classification model server with `vllm serve`, e.g.
    vllm serve jason9693/Qwen2.5-1.5B-apeach
"""
```
**EN:** Example Python client for classification API using vLLM API server NOTE: start a supported classification model server with `vllm serve`, e.g.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import pprint

import requests
```
**EN:** This block loads helper libraries such as argparse, pprint, and requests.
**CN:** 这一部分加载 argparse、pprint，以及 requests 等辅助库。

### Top-level setup
```python
headers = {"accept": "application/json", "Content-Type": "application/json"}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as headers.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 headers 等变量。

### Function: parse_args
```python
def parse_args():
    parse = argparse.ArgumentParser()
    parse.add_argument("--host", type=str, default="localhost")
    parse.add_argument("--port", type=int, default=8000)
    return parse.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parse.add_argument, argparse.ArgumentParser, and parse.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parse.add_argument、argparse.ArgumentParser，以及 parse.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    base_url = f"http://{args.host}:{args.port}"
    models_url = base_url + "/v1/models"
    classify_url = base_url + "/classify"
    tokenize_url = base_url + "/tokenize"

    response = requests.get(models_url, headers=headers)
    model = response.json()["data"][0]["id"]

    # /classify can accept str as input
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]

    payload = {
        "model": model,
        "input": prompts,
    }
    response = requests.post(classify_url, headers=headers, json=payload)
    pprint.pprint(response.json())

    # /classify can accept token ids as input
    token_ids = []
    for prompt in prompts:
        response = requests.post(
            tokenize_url,
            json={"model": model, "prompt": prompt},
        )
        token_ids.append(response.json()["tokens"])

    payload = {
        "model": model,
        "input": token_ids,
    }
    response = requests.post(classify_url, headers=headers, json=payload)
    pprint.pprint(response.json())
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include response.json, requests.post, pprint.pprint, requests.get, and token_ids.append.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 response.json、requests.post、pprint.pprint、requests.get，以及 token_ids.append。

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
- **External libraries / 外部库**: `argparse`, `pprint`, `requests` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `parse.add_argument`, `argparse.ArgumentParser`, `parse.parse_args`, `response.json`, `requests.post`, `pprint.pprint`, `requests.get`, `token_ids.append` reveal the main execution path / 这些调用体现了主要执行链路。
