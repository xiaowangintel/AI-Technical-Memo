# qwen3_vl_reranker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/qwen3_vl_reranker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Example usage of Qwen3-VL-Reranker with SGLang. This example demonstrates how to use the Qwen3-VL-Reranker model for multimodal reranking tasks, supporting text, images, and videos. / 该文件的顶部说明概述了此示例的目标与使用场景。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Module overview and usage
````python
"""
Example usage of Qwen3-VL-Reranker with SGLang.

This example demonstrates how to use the Qwen3-VL-Reranker model for multimodal
reranking tasks, supporting text, images, and videos.

Server Launch:
    python -m sglang.launch_server \
        --model-path Qwen/Qwen3-VL-Reranker-2B \
        --served-model-name Qwen3-VL-Reranker-2B \
        --trust-remote-code \
        --disable-radix-cache \
        --chat-template examples/chat_template/qwen3_vl_reranker.jinja

Client Usage:
    python examples/runtime/qwen3_vl_reranker.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 19-19: Import dependencies and runtime symbols
````python
import requests
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 21-22: Set top-level configuration
````python
# Server URL
BASE_URL = "http://localhost:30000"
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 25-48: Score or rerank inputs
````python
def rerank_text_only():
    """Example: Text-only reranking (backward compatible)."""
    print("=" * 60)
    print("Text-only reranking example")
    print("=" * 60)

    request_data = {
        "query": "What is machine learning?",
        "documents": [
            "Machine learning is a branch of artificial intelligence that enables computers to learn from data.",
            "The weather in Paris is usually mild with occasional rain.",
            "Deep learning is a subset of machine learning using neural networks with many layers.",
        ],
        "instruct": "Retrieve passages that answer the question.",
        "return_documents": True,
    }

    response = requests.post(f"{BASE_URL}/v1/rerank", json=request_data)
    results = response.json()

    print("Results (sorted by relevance):")
    for i, result in enumerate(results):
        print(f"  {i+1}. Score: {result['score']:.4f} - {result['document'][:60]}...")
    print()
````
**EN:** This function computes scores for candidate inputs and exposes them in a format suitable for ranking or reranking.
**CN:** 该函数会为候选输入计算分数，并以适合排序或重排序的形式输出。

### Lines 51-103: Score or rerank inputs
````python
def rerank_with_images():
    """Example: Query is text, documents contain images."""
    print("=" * 60)
    print("Image reranking example")
    print("=" * 60)

    request_data = {
        "query": "A woman playing with her dog on a beach at sunset.",
        "documents": [
            # Document 1: Text description
            "A woman shares a joyful moment with her golden retriever on a sun-drenched beach at sunset.",
            # Document 2: Image URL
            [
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"
                    },
                }
            ],
            # Document 3: Text + Image (mixed)
            [
                {
                    "type": "text",
                    "text": "A joyful scene at the beach:",
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"
                    },
                },
            ],
        ],
        "instruct": "Retrieve images or text relevant to the user's query.",
        "return_documents": False,
    }

    response = requests.post(f"{BASE_URL}/v1/rerank", json=request_data)
    results = response.json()

    # Debug: print raw response if it's an error
    if isinstance(results, dict) and "message" in results:
        print(f"Error: {results['message']}")
        return
    if isinstance(results, str):
        print(f"Error: {results}")
        return

    print("Results (sorted by relevance):")
    for i, result in enumerate(results):
        print(f"  {i+1}. Index: {result['index']}, Score: {result['score']:.4f}")
    print()
````
**EN:** This function computes scores for candidate inputs and exposes them in a format suitable for ranking or reranking.
**CN:** 该函数会为候选输入计算分数，并以适合排序或重排序的形式输出。

### Lines 106-153: Score or rerank inputs
````python
def rerank_multimodal_query():
    """Example: Query contains both text and image."""
    print("=" * 60)
    print("Multimodal query reranking example")
    print("=" * 60)

    request_data = {
        # Query with text and image
        "query": [
            {"type": "text", "text": "Find similar images to this:"},
            {
                "type": "image_url",
                "image_url": {
                    "url": "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"
                },
            },
        ],
        "documents": [
            "A cat sleeping on a couch.",
            "A woman and her dog enjoying the sunset at the beach.",
            "A busy city street with cars and pedestrians.",
            [
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-VL/assets/demo.jpeg"
                    },
                }
            ],
        ],
        "instruct": "Find images or descriptions similar to the query image.",
    }

    response = requests.post(f"{BASE_URL}/v1/rerank", json=request_data)
    results = response.json()

    # Debug: print raw response if it's an error
    if isinstance(results, dict) and "message" in results:
        print(f"Error: {results['message']}")
        return
    if isinstance(results, str):
        print(f"Error: {results}")
        return

    print("Results (sorted by relevance):")
    for i, result in enumerate(results):
        print(f"  {i+1}. Index: {result['index']}, Score: {result['score']:.4f}")
    print()
````
**EN:** This function computes scores for candidate inputs and exposes them in a format suitable for ranking or reranking.
**CN:** 该函数会为候选输入计算分数，并以适合排序或重排序的形式输出。

### Lines 156-181: Parse arguments and run the workflow
````python
def main():
    """Run all examples."""
    print("\nQwen3-VL-Reranker Examples")
    print("Make sure the server is running with the correct model and template.\n")

    # Check if server is available
    try:
        response = requests.get(f"{BASE_URL}/health")
        if response.status_code != 200:
            print(f"Server health check failed: {response.status_code}")
            return
    except requests.exceptions.ConnectionError:
        print(f"Cannot connect to server at {BASE_URL}")
        print("Please start the server first with:")
        print("  python -m sglang.launch_server \\")
        print("      --model-path Qwen/Qwen3-VL-Reranker-2B \\")
        print("      --served-model-name Qwen3-VL-Reranker-2B \\")
        print("      --trust-remote-code \\")
        print("      --disable-radix-cache \\")
        print("      --chat-template examples/chat_template/qwen3_vl_reranker.jinja")
        return

    # Run examples
    rerank_text_only()
    rerank_with_images()
    rerank_multimodal_query()
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 184-185: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Multimodal inputs / 多模态输入**: The example mixes text with image or video inputs. / 该示例将文本与图像或视频输入结合起来。
- **Scoring and reranking / 打分与重排序**: The model returns scores used for ranking candidates. / 模型返回可用于候选项排序的分数。

## Dependencies / 依赖关系
- **Third-party / 第三方**: requests
