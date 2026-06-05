# colqwen3_token_embed_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/token_embed/colqwen3_token_embed_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example online usage of Pooling API for ColQwen3 multi-vector retrieval / 演示词元级嵌入提取。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example online usage of Pooling API for ColQwen3 multi-vector retrieval.

ColQwen3 is a multi-modal late interaction model based on Qwen3-VL that
produces per-token embeddings (320-dim, L2-normalized) for both text and
image inputs. Similarity is computed via MaxSim scoring.

This example mirrors the official TomoroAI inference code
(https://huggingface.co/TomoroAI/tomoro-colqwen3-embed-4b) but uses the
vLLM serving API instead of local HuggingFace model loading.

Start the server with:
    vllm serve TomoroAI/tomoro-colqwen3-embed-4b --max-model-len 4096

Then run this script:
    python colqwen3_token_embed_online.py
"""
```
**EN:** Example online usage of Pooling API for ColQwen3 multi-vector retrieval.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
from io import BytesIO

import numpy as np
import pybase64 as base64
import requests
from PIL import Image
```
**EN:** This block loads helper libraries such as argparse, io, numpy, pybase64, and requests.
**CN:** 这一部分加载 argparse、io、numpy、pybase64，以及 requests 等辅助库。

### Function: post_http_request
```python
def post_http_request(payload: dict, api_url: str) -> requests.Response:
    headers = {"User-Agent": "Test Client"}
    return requests.post(api_url, headers=headers, json=payload)
```
**EN:** This function sends a request and handles the reply. It works with parameters such as payload and api_url. Key operations include requests.post. The return value feeds the next stage of the example pipeline.
**CN:** 该函数发送请求并处理返回结果。它会处理 payload 和 api_url 等参数。关键操作包括 requests.post。其返回值会继续传给示例管线的下一阶段。

### Function: load_image
```python
def load_image(url: str) -> Image.Image:
    """Download an image from URL (handles Wikimedia 403)."""
    for hdrs in ({}, {"User-Agent": "Mozilla/5.0 (compatible; ColQwen3-demo/1.0)"}):
        resp = requests.get(url, headers=hdrs, timeout=10)
        if resp.status_code == 403:
            continue
        resp.raise_for_status()
        return Image.open(BytesIO(resp.content)).convert("RGB")
    raise RuntimeError(f"Could not fetch image from {url}")
```
**EN:** Download an image from URL (handles Wikimedia 403).. It works with parameters such as url. Key operations include RuntimeError, requests.get, resp.raise_for_status, convert, and Image.open. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 url 等参数。关键操作包括 RuntimeError、requests.get、resp.raise_for_status、convert，以及 Image.open。其返回值会继续传给示例管线的下一阶段。

### Function: encode_image_base64
```python
def encode_image_base64(image: Image.Image) -> str:
    """Encode a PIL image to a base64 data URI."""
    buf = BytesIO()
    image.save(buf, format="PNG")
    return "data:image/png;base64," + base64.b64encode(buf.getvalue()).decode()
```
**EN:** Encode a PIL image to a base64 data URI.. It works with parameters such as image. Key operations include BytesIO, image.save, decode, base64.b64encode, and buf.getvalue. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 image 等参数。关键操作包括 BytesIO、image.save、decode、base64.b64encode，以及 buf.getvalue。其返回值会继续传给示例管线的下一阶段。

### Function: compute_maxsim
```python
def compute_maxsim(q_emb: np.ndarray, d_emb: np.ndarray) -> float:
    """Compute ColBERT-style MaxSim score between query and document."""
    sim = q_emb @ d_emb.T
    return float(sim.max(axis=-1).sum())
```
**EN:** Compute ColBERT-style MaxSim score between query and document.. It works with parameters such as q_emb and d_emb. Key operations include float, sum, and sim.max. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 q_emb 和 d_emb 等参数。关键操作包括 float、sum，以及 sim.max。其返回值会继续传给示例管线的下一阶段。

### Function: encode_queries
```python
def encode_queries(texts: list[str], model: str, api_url: str) -> list[np.ndarray]:
    """Encode text queries → list of multi-vector embeddings."""
    resp = post_http_request({"model": model, "input": texts}, api_url)
    return [np.array(item["data"]) for item in resp.json()["data"]]
```
**EN:** Encode text queries → list of multi-vector embeddings.. It works with parameters such as texts, model, and api_url. Key operations include post_http_request, np.array, and resp.json. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 texts、model，以及 api_url 等参数。关键操作包括 post_http_request、np.array，以及 resp.json。其返回值会继续传给示例管线的下一阶段。

### Function: encode_images
```python
def encode_images(image_urls: list[str], model: str, api_url: str) -> list[np.ndarray]:
    """Encode image documents → list of multi-vector embeddings.

    Images are sent via the chat-style `messages` field so that the
    vLLM multimodal processor handles them correctly.
    """
    embeddings = []
    for url in image_urls:
        print(f"  Loading: {url.split('/')[-1]}...")
        image = load_image(url)
        image_uri = encode_image_base64(image)
        resp = post_http_request(
            {
                "model": model,
                "messages": [
                    {
                        "role": "user",
                        "content": [
                            {"type": "image_url", "image_url": {"url": image_uri}},
                            {"type": "text", "text": "Describe the image."},
                        ],
                    }
                ],
            },
            api_url,
        )
        result = resp.json()
        if resp.status_code != 200 or "data" not in result:
            print(f"    Error ({resp.status_code}): {str(result)[:200]}")
            continue
        embeddings.append(np.array(result["data"][0]["data"]))
    return embeddings
```
**EN:** Encode image documents → list of multi-vector embeddings.. It works with parameters such as image_urls, model, and api_url. Key operations include print, load_image, encode_image_base64, post_http_request, and resp.json. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 image_urls、model，以及 api_url 等参数。关键操作包括 print、load_image、encode_image_base64、post_http_request，以及 resp.json。其返回值会继续传给示例管线的下一阶段。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", type=str, default="localhost")
    parser.add_argument("--port", type=int, default=8000)
    parser.add_argument(
        "--model",
        type=str,
        default="TomoroAI/tomoro-colqwen3-embed-4b",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    pooling_url = f"http://{args.host}:{args.port}/pooling"
    score_url = f"http://{args.host}:{args.port}/score"
    model = args.model

    # Same sample data as the official TomoroAI example
    queries = [
        "Retrieve the city of Singapore",
        "Retrieve the city of Beijing",
        "Retrieve the city of London",
    ]
    image_urls = [
        "https://upload.wikimedia.org/wikipedia/commons/2/27/Singapore_skyline_2022.jpg",
        "https://upload.wikimedia.org/wikipedia/commons/6/61/Beijing_skyline_at_night.JPG",
        "https://upload.wikimedia.org/wikipedia/commons/4/49/London_skyline.jpg",
    ]

    # ── 1) Text query embeddings ────────────────────────────
    print("=" * 60)
    print("1. Encode text queries (multi-vector)")
    # ... key logic omitted for brevity ...
    text_query = "What is the capital of France?"
    text_docs = [
        "The capital of France is Paris.",
        "Berlin is the capital of Germany.",
        "Python is a programming language.",
    ]
    resp = post_http_request(
        {"model": model, "text_1": text_query, "text_2": text_docs},
        score_url,
    )
    print(f'  Query: "{text_query}"\n')
    for item in resp.json()["data"]:
        idx = item["index"]
        print(f"  Doc {idx} (score={item['score']:.4f}): {text_docs[idx]}")
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, enumerate, encode_queries, encode_images, and post_http_request.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、enumerate、encode_queries、encode_images，以及 post_http_request。

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
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `argparse`, `io`, `numpy`, `pybase64`, `requests`, `PIL` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `post_http_request`, `load_image`, `encode_image_base64`, `compute_maxsim`, `encode_queries`, `encode_images`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `requests.post`, `RuntimeError`, `requests.get`, `resp.raise_for_status`, `convert`, `Image.open`, `BytesIO`, `image.save` reveal the main execution path / 这些调用体现了主要执行链路。
