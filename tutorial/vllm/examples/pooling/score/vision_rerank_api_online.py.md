# vision_rerank_api_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/score/vision_rerank_api_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example Python client for multimodal rerank API which is compatible with Jina and Cohere https://jina.ai/reranker Run `vllm serve <model> --runner pooling` to start up the server in vLLM / 演示打分与重排序流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example Python client for multimodal rerank API which is compatible with
Jina and Cohere https://jina.ai/reranker

Run `vllm serve <model> --runner pooling` to start up the server in vLLM.
e.g.
    vllm serve jinaai/jina-reranker-m0 --runner pooling

    vllm serve Qwen/Qwen3-VL-Reranker-2B \
        --runner pooling \
        --max-model-len 4096 \
        --hf_overrides '{"architectures": ["Qwen3VLForSequenceClassification"],"classifier_from_token": ["no", "yes"],"is_original_qwen3_reranker": true}' \
        --chat-template examples/pooling/score/template/qwen3_vl_reranker.jinja
"""
```
**EN:** Example Python client for multimodal rerank API which is compatible with Jina and Cohere https://jina.ai/reranker Run `vllm serve <model> --runner pooling` to start up the server in vLLM.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import pprint

import requests

from vllm.multimodal.utils import encode_image_url, fetch_image
```
**EN:** This block loads helper libraries such as argparse, pprint, and requests and pulls in vLLM APIs like vllm.multimodal.utils.
**CN:** 这一部分加载 argparse、pprint，以及 requests 等辅助库，并引入 vllm.multimodal.utils 等 vLLM API。

### Top-level setup
```python
query = "A woman playing with her dog on a beach at sunset."
document = (
    "A woman shares a joyful moment with her golden retriever on a sun-drenched beach at sunset, "
    "as the dog offers its paw in a heartwarming display of companionship and trust."
)
image_url = "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"
video_url = "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen3-Omni/demo/draw.mp4"
documents = [
    {
        "type": "text",
        "text": document,
    },
    {
        "type": "image_url",
        "image_url": {"url": image_url},
    },
    {
        "type": "image_url",
        "image_url": {"url": encode_image_url(fetch_image(image_url))},
    },
    {
        "type": "video_url",
        "video_url": {"url": video_url},
    },
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as query, document, image_url, video_url, and documents. It also performs early helper calls such as encode_image_url and fetch_image.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 query、document、image_url、video_url，以及 documents 等变量。它还会提前执行 encode_image_url 和 fetch_image 等辅助调用。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", type=str, default="localhost")
    parser.add_argument("--port", type=int, default=8000)
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    base_url = f"http://{args.host}:{args.port}"
    models_url = base_url + "/v1/models"
    rerank_url = base_url + "/rerank"

    response = requests.get(models_url)
    model = response.json()["data"][0]["id"]

    print("Query: string & Document: list of string")
    prompt = {"model": model, "query": query, "documents": [document]}
    response = requests.post(rerank_url, json=prompt)
    pprint.pprint(response.json())

    print("Query: string & Document: text")
    prompt = {"model": model, "query": query, "documents": {"content": [documents[0]]}}
    response = requests.post(rerank_url, json=prompt)
    pprint.pprint(response.json())

    print("Query: string & Document: image url")
    prompt = {
    # ... key logic omitted for brevity ...

    print("Query: string & Document: list")
    prompt = {
        "model": model,
        "query": query,
        "documents": [
            document,
            {"content": [documents[0]]},
            {"content": [documents[1]]},
            {"content": [documents[0], documents[1]]},
        ],
    }
    response = requests.post(rerank_url, json=prompt)
    pprint.pprint(response.json())
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include response.json, print, requests.post, pprint.pprint, and requests.get.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 response.json、print、requests.post、pprint.pprint，以及 requests.get。

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
- **RAG orchestration / RAG 编排**: Documents are loaded, indexed, retrieved, and injected into prompts. / 文档会被加载、索引、检索，并注入到提示词中。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm.multimodal.utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `pprint`, `requests` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `encode_image_url`, `fetch_image`, `parser.add_argument`, `argparse.ArgumentParser`, `parser.parse_args`, `response.json`, `print`, `requests.post` reveal the main execution path / 这些调用体现了主要执行链路。
