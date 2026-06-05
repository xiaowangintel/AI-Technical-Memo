# colqwen3_rerank_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/score/colqwen3_rerank_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example of using ColQwen3 late interaction model for reranking and scoring / 演示打分与重排序流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example of using ColQwen3 late interaction model for reranking and scoring.

ColQwen3 is a multi-modal ColBERT-style model based on Qwen3-VL.
It produces per-token embeddings and uses MaxSim scoring for retrieval
and reranking. Supports both text and image inputs.

Start the server with:
    vllm serve TomoroAI/tomoro-colqwen3-embed-4b --max-model-len 50000

Then run this script:
    python colqwen3_rerank_online.py
"""
```
**EN:** Example of using ColQwen3 late interaction model for reranking and scoring.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from io import BytesIO

import pybase64 as base64
import requests
from PIL import Image
```
**EN:** This block loads helper libraries such as io, pybase64, requests, and PIL.
**CN:** 这一部分加载 io、pybase64、requests，以及 PIL 等辅助库。

### Top-level setup
```python
MODEL = "TomoroAI/tomoro-colqwen3-embed-4b"
BASE_URL = "http://127.0.0.1:8000"

headers = {"accept": "application/json", "Content-Type": "application/json"}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as MODEL, BASE_URL, and headers.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 MODEL、BASE_URL，以及 headers 等变量。

### Function: load_image
```python
def load_image(url: str) -> Image.Image:
    """Download an image from URL (handles Wikimedia 403)."""
    for hdrs in (
        {},
        {"User-Agent": "Mozilla/5.0 (compatible; ColQwen3-demo/1.0)"},
    ):
        resp = requests.get(url, headers=hdrs, timeout=15)
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

### Function: make_image_content
```python
def make_image_content(image_url: str, text: str = "Describe the image.") -> dict:
    """Build a ScoreMultiModalParam dict from an image URL."""
    image = load_image(image_url)
    return {
        "content": [
            {
                "type": "image_url",
                "image_url": {"url": encode_image_base64(image)},
            },
            {"type": "text", "text": text},
        ]
    }
```
**EN:** Build a ScoreMultiModalParam dict from an image URL.. It works with parameters such as image_url and text. Key operations include load_image and encode_image_base64. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 image_url 和 text 等参数。关键操作包括 load_image 和 encode_image_base64。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
IMAGE_URLS = {
    "beijing": "https://upload.wikimedia.org/wikipedia/commons/6/61/Beijing_skyline_at_night.JPG",
    "london": "https://upload.wikimedia.org/wikipedia/commons/4/49/London_skyline.jpg",
    "singapore": "https://upload.wikimedia.org/wikipedia/commons/2/27/Singapore_skyline_2022.jpg",
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as IMAGE_URLS.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 IMAGE_URLS 等变量。

### Function: rerank_text
```python
def rerank_text():
    """Text-only reranking via /rerank endpoint."""
    print("=" * 60)
    print("1. Text reranking (/rerank)")
    print("=" * 60)

    data = {
        "model": MODEL,
        "query": "What is machine learning?",
        "documents": [
            "Machine learning is a subset of artificial intelligence.",
            "Python is a programming language.",
            "Deep learning uses neural networks for complex tasks.",
            "The weather today is sunny.",
        ],
    }

    response = requests.post(f"{BASE_URL}/rerank", headers=headers, json=data)

    if response.status_code == 200:
        result = response.json()
        print("\n  Ranked documents (most relevant first):")
        for item in result["results"]:
            doc_idx = item["index"]
            score = item["relevance_score"]
            print(f"    [{score:.4f}] {data['documents'][doc_idx]}")
    else:
        print(f"  Request failed: {response.status_code}")
        print(f"  {response.text[:300]}")
```
**EN:** Text-only reranking via /rerank endpoint.. Key operations include print, requests.post, and response.json.
**CN:** 该函数生成任务相关的分数或标签。关键操作包括 print、requests.post，以及 response.json。

### Function: score_text
```python
def score_text():
    """Text-only scoring via /score endpoint."""
    print()
    print("=" * 60)
    print("2. Text scoring (/score)")
    print("=" * 60)

    query = "What is the capital of France?"
    documents = [
        "The capital of France is Paris.",
        "Berlin is the capital of Germany.",
        "Python is a programming language.",
    ]

    data = {
        "model": MODEL,
        "text_1": query,
        "text_2": documents,
    }

    response = requests.post(f"{BASE_URL}/score", headers=headers, json=data)

    if response.status_code == 200:
        result = response.json()
        print(f"\n  Query: {query}\n")
        for item in result["data"]:
            idx = item["index"]
            score = item["score"]
            print(f"    Doc {idx} (score={score:.4f}): {documents[idx]}")
    else:
        print(f"  Request failed: {response.status_code}")
        print(f"  {response.text[:300]}")
```
**EN:** Text-only scoring via /score endpoint.. Key operations include print, requests.post, and response.json.
**CN:** 该函数生成任务相关的分数或标签。关键操作包括 print、requests.post，以及 response.json。

### Function: score_text_top_n
```python
def score_text_top_n():
    """Text reranking with top_n filtering via /rerank endpoint."""
    print()
    print("=" * 60)
    print("3. Text reranking with top_n=2 (/rerank)")
    print("=" * 60)

    data = {
        "model": MODEL,
        "query": "What is the capital of France?",
        "documents": [
            "The capital of France is Paris.",
            "Berlin is the capital of Germany.",
            "Python is a programming language.",
            "The Eiffel Tower is in Paris.",
        ],
        "top_n": 2,
    }

    response = requests.post(f"{BASE_URL}/rerank", headers=headers, json=data)

    if response.status_code == 200:
        result = response.json()
        print(f"\n  Top {data['top_n']} results:")
        for item in result["results"]:
            doc_idx = item["index"]
            score = item["relevance_score"]
            print(f"    [{score:.4f}] {data['documents'][doc_idx]}")
    else:
        print(f"  Request failed: {response.status_code}")
        print(f"  {response.text[:300]}")
```
**EN:** Text reranking with top_n filtering via /rerank endpoint.. Key operations include print, requests.post, and response.json.
**CN:** 该函数生成任务相关的分数或标签。关键操作包括 print、requests.post，以及 response.json。

### Function: score_text_vs_images
```python
def score_text_vs_images():
    """Score a text query against image documents via /score."""
    print()
    print("=" * 60)
    print("4. Multi-modal scoring: text query vs image docs (/score)")
    print("=" * 60)

    query = "Retrieve the city of Beijing"
    labels = list(IMAGE_URLS.keys())
    print(f"\n  Loading {len(labels)} images...")
    image_contents = [make_image_content(IMAGE_URLS[name]) for name in labels]

    data = {
        "model": MODEL,
        "data_1": query,
        "data_2": image_contents,
    }

    response = requests.post(f"{BASE_URL}/score", headers=headers, json=data)

    if response.status_code == 200:
        result = response.json()
        print(f'\n  Query: "{query}"\n')
        for item in result["data"]:
            idx = item["index"]
            print(f"    Doc {idx} [{labels[idx]}] score={item['score']:.4f}")
    else:
        print(f"  Request failed: {response.status_code}")
        print(f"  {response.text[:300]}")
```
**EN:** Score a text query against image documents via /score.. Key operations include print, list, requests.post, IMAGE_URLS.keys, and make_image_content.
**CN:** 该函数生成任务相关的分数或标签。关键操作包括 print、list、requests.post、IMAGE_URLS.keys，以及 make_image_content。

### Function: rerank_text_vs_images
```python
def rerank_text_vs_images():
    """Rerank image documents by a text query via /rerank."""
    print()
    print("=" * 60)
    print("5. Multi-modal reranking: text query vs image docs (/rerank)")
    print("=" * 60)

    query = "Retrieve the city of London"
    labels = list(IMAGE_URLS.keys())
    print(f"\n  Loading {len(labels)} images...")
    image_contents = [make_image_content(IMAGE_URLS[name]) for name in labels]

    data = {
        "model": MODEL,
        "query": query,
        "documents": image_contents,
        "top_n": 2,
    }

    response = requests.post(f"{BASE_URL}/rerank", headers=headers, json=data)

    if response.status_code == 200:
        result = response.json()
        print(f'\n  Query: "{query}"')
        print(f"  Top {data['top_n']} results:\n")
        for item in result["results"]:
            idx = item["index"]
            print(f"    [{item['relevance_score']:.4f}] {labels[idx]}")
    else:
        print(f"  Request failed: {response.status_code}")
        print(f"  {response.text[:300]}")
```
**EN:** Rerank image documents by a text query via /rerank.. Key operations include print, list, requests.post, IMAGE_URLS.keys, and make_image_content.
**CN:** 该函数生成任务相关的分数或标签。关键操作包括 print、list、requests.post、IMAGE_URLS.keys，以及 make_image_content。

### Function: main
```python
def main():
    # Text-only
    rerank_text()
    score_text()
    score_text_top_n()

    # Multi-modal (text query × image documents)
    score_text_vs_images()
    rerank_text_vs_images()
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include rerank_text, score_text, score_text_top_n, score_text_vs_images, and rerank_text_vs_images.
**CN:** 该函数编排端到端工作流。关键操作包括 rerank_text、score_text、score_text_top_n、score_text_vs_images，以及 rerank_text_vs_images。

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
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `io`, `pybase64`, `requests`, `PIL` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `load_image`, `encode_image_base64`, `make_image_content`, `rerank_text`, `score_text`, `score_text_top_n`, `score_text_vs_images`, `rerank_text_vs_images` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `RuntimeError`, `requests.get`, `resp.raise_for_status`, `convert`, `Image.open`, `BytesIO`, `image.save`, `decode` reveal the main execution path / 这些调用体现了主要执行链路。
