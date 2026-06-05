# client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/embed/openai_embedding_long_text/client.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example script demonstrating long text embedding with chunked processing in vLLM / 演示使用池化模型生成嵌入。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example script demonstrating long text embedding with chunked processing in vLLM.

This example shows how to use vLLM's chunked processing feature to handle text
inputs that exceed the model's maximum token length. The feature automatically
splits long text into chunks and handles different pooling types optimally.

Prerequisites:
1. Start vLLM server with chunked processing enabled:
   
   # MEAN pooling (processes all chunks, recommended for complete coverage)
   vllm serve intfloat/multilingual-e5-large \
# ... key logic omitted for brevity ...
     --served-model-name bge-large-en-v1.5 \
     --trust-remote-code \
     --port 31090 \
     --api-key your-api-key

2. Install required dependencies:
   pip install openai requests
"""
```
**EN:** Example script demonstrating long text embedding with chunked processing in vLLM.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import time

import numpy as np
from openai import OpenAI
```
**EN:** This block loads helper libraries such as time, numpy, and openai.
**CN:** 这一部分加载 time、numpy，以及 openai 等辅助库。

### Top-level setup
```python
API_KEY = "your-api-key"  # Replace with your actual API key
BASE_URL = "http://localhost:31090/v1"
MODEL_NAME = "multilingual-e5-large"
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as API_KEY, BASE_URL, and MODEL_NAME.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 API_KEY、BASE_URL，以及 MODEL_NAME 等变量。

### Function: generate_long_text
```python
def generate_long_text(base_text: str, repeat_count: int) -> str:
    """Generate long text by repeating base text."""
    return base_text * repeat_count
```
**EN:** Generate long text by repeating base text.. It works with parameters such as base_text and repeat_count. The return value feeds the next stage of the example pipeline.
**CN:** 该函数触发模型推理并收集输出。它会处理 base_text 和 repeat_count 等参数。其返回值会继续传给示例管线的下一阶段。

### Function: test_embedding_with_different_lengths
```python
def test_embedding_with_different_lengths():
    """Test embedding generation with different text lengths."""
    client = OpenAI(api_key=API_KEY, base_url=BASE_URL)

    # Test cases with different text lengths
    test_cases = [
        {
            "name": "Short Text",
            "text": "Hello, this is a short text for embedding.",
            "expected_chunks": 1,
        },
        {
            "name": "Medium Text",
            "text": generate_long_text(
                "This is a medium-length text that should fit within the "
                "model's context window. " * 20,
                2,
            ),
            "expected_chunks": 1,
        },
    # ... key logic omitted for brevity ...
            processing_time = end_time - start_time

            # Extract embedding data
            embedding = response.data[0].embedding
            embedding_dim = len(embedding)

            print("✅ Success!")
            print(f"   - Embedding dimension: {embedding_dim}")
            print(f"   - Processing time: {processing_time:.2f}s")
            print(f"   - Expected chunks: ~{test_case['expected_chunks']}")
            print(f"   - First 5 values: {embedding[:5]}")

        except Exception as e:
            print(f"❌ Failed: {str(e)}")
```
**EN:** Test embedding generation with different text lengths.. Key operations include print, generate_long_text, time.time, len, and OpenAI.
**CN:** 该函数从模型输出中计算池化表示。关键操作包括 print、generate_long_text、time.time、len，以及 OpenAI。

### Function: test_batch_embedding
```python
def test_batch_embedding():
    """Test batch embedding with mixed-length inputs."""
    client = OpenAI(api_key=API_KEY, base_url=BASE_URL)

    print("\n🔄 Testing Batch Embedding with Mixed Lengths")
    print("=" * 50)

    # Mix of short and long texts
    batch_inputs = [
        "Short text 1",
        generate_long_text("Medium length text that fits in one chunk. " * 20, 1),
        "Another short text",
        generate_long_text("Long text requiring chunked processing. " * 100, 5),
    ]

    try:
        start_time = time.time()

        response = client.embeddings.create(
            input=batch_inputs, model=MODEL_NAME, encoding_format="float"
        )

        end_time = time.time()
        processing_time = end_time - start_time

        print("✅ Batch processing successful!")
        print(f"   - Number of inputs: {len(batch_inputs)}")
        print(f"   - Number of embeddings: {len(response.data)}")
        print(f"   - Total processing time: {processing_time:.2f}s")
        print(
            f"   - Average time per input: {processing_time / len(batch_inputs):.2f}s"
        )

        for i, data in enumerate(response.data):
            input_length = len(batch_inputs[i])
            embedding_dim = len(data.embedding)
            print(
                f"   - Input {i + 1}: {input_length} chars → {embedding_dim}D embedding"
            )

    except Exception as e:
        print(f"❌ Batch processing failed: {str(e)}")
```
**EN:** Test batch embedding with mixed-length inputs.. Key operations include print, len, generate_long_text, time.time, and OpenAI.
**CN:** 该函数从模型输出中计算池化表示。关键操作包括 print、len、generate_long_text、time.time，以及 OpenAI。

### Function: test_multiple_long_texts_batch
```python
def test_multiple_long_texts_batch():
    """Test batch processing with multiple long texts to verify chunk ID uniqueness."""
    client = OpenAI(api_key=API_KEY, base_url=BASE_URL)

    print("\n🔧 Testing Multiple Long Texts in Batch (Chunk ID Fix Verification)")
    print("=" * 70)

    # Create multiple distinct long texts that will all require chunking
    # Note: All pooling types now use MEAN aggregation across chunks:
    # - Native pooling (MEAN/CLS/LAST) is used within each chunk
    # - MEAN aggregation combines results across all chunks
    # - Full semantic coverage for all pooling types
    long_texts = [
        generate_long_text(
            "First long document about artificial intelligence and machine learning. "
            * 80,
            6,
        ),
        generate_long_text(
            "Second long document about natural language processing and transformers. "
    # ... key logic omitted for brevity ...
                f"   - Input {i + 1}: {input_length} chars → {embedding_dim}D "
                f"embedding (norm: {embedding_norm:.4f})"
            )

        print(
            "\n✅ This test verifies the fix for chunk ID collisions in "
            "batch processing"
        )
        print("   - Before fix: Multiple long texts would have conflicting chunk IDs")
        print("   - After fix: Each prompt's chunks have unique IDs with prompt index")

    except Exception as e:
        print(f"❌ Multiple long texts batch test failed: {str(e)}")
        print("   This might indicate the chunk ID collision bug is present!")
```
**EN:** Test batch processing with multiple long texts to verify chunk ID uniqueness.. Key operations include print, len, generate_long_text, np.linalg.norm, and np.array.
**CN:** 该函数封装示例中的可复用步骤。关键操作包括 print、len、generate_long_text、np.linalg.norm，以及 np.array。

### Function: test_embedding_consistency
```python
def test_embedding_consistency():
    """Test that chunked processing produces consistent results."""
    client = OpenAI(api_key=API_KEY, base_url=BASE_URL)

    print("\n🔍 Testing Embedding Consistency")
    print("=" * 40)

    # Use the same long text multiple times
    long_text = generate_long_text(
        "Consistency test text for chunked processing validation. " * 50, 3
    )

    embeddings = []

    try:
        for i in range(3):
            response = client.embeddings.create(
                input=long_text, model=MODEL_NAME, encoding_format="float"
            )
            embeddings.append(response.data[0].embedding)
    # ... key logic omitted for brevity ...
                np.linalg.norm(emb1) * np.linalg.norm(emb2)
            )

            print("✅ Consistency test completed!")
            print(f"   - Cosine similarity between runs: {cosine_sim:.6f}")
            print("   - Expected: ~1.0 (identical embeddings)")

            if cosine_sim > 0.999:
                print("   - ✅ High consistency achieved!")
            else:
                print("   - ⚠️ Consistency may vary due to numerical precision")

    except Exception as e:
        print(f"❌ Consistency test failed: {str(e)}")
```
**EN:** Test that chunked processing produces consistent results.. Key operations include print, np.array, np.linalg.norm, OpenAI, and generate_long_text.
**CN:** 该函数从模型输出中计算池化表示。关键操作包括 print、np.array、np.linalg.norm、OpenAI，以及 generate_long_text。

### Function: main
```python
def main():
    """Main function to run all tests."""
    print("🚀 vLLM Long Text Embedding Client")
    print(f"📡 Connecting to: {BASE_URL}")
    print(f"🤖 Model: {MODEL_NAME}")
    masked_key = "*" * (len(API_KEY) - 4) + API_KEY[-4:] if len(API_KEY) > 4 else "****"
    print(f"🔑 API Key: {masked_key}")

    # Run all test cases
    test_embedding_with_different_lengths()
    test_batch_embedding()
    test_multiple_long_texts_batch()
    test_embedding_consistency()

    print("\n" + "=" * 70)
    print("🎉 All tests completed!")
    print("\n💡 Key Features Demonstrated:")
    print("   - ✅ Automatic chunked processing for long text")
    print("   - ✅ Seamless handling of mixed-length batches")
    print("   - ✅ Multiple long texts in single batch (chunk ID fix)")
    print("   - ✅ Unified chunked processing:")
    print("     • Native pooling used within each chunk")
    print("     • MEAN aggregation across all chunks")
    print("     • Complete semantic coverage for all pooling types")
    print("   - ✅ Consistent embedding generation")
    print("   - ✅ Backward compatibility with short text")
    print("\n📚 For more information, see:")
    print(
        "   - Documentation: https://docs.vllm.ai/en/latest/models/pooling_models.html"
    )
    print("   - Chunked Processing Guide: openai_embedding_long_text.md")
```
**EN:** Main function to run all tests.. Key operations include print, len, test_embedding_with_different_lengths, test_batch_embedding, and test_multiple_long_texts_batch.
**CN:** 该函数编排端到端工作流。关键操作包括 print、len、test_embedding_with_different_lengths、test_batch_embedding，以及 test_multiple_long_texts_batch。

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

## Dependencies / 依赖关系
- **External libraries / 外部库**: `time`, `numpy`, `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `generate_long_text`, `test_embedding_with_different_lengths`, `test_batch_embedding`, `test_multiple_long_texts_batch`, `test_embedding_consistency`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `generate_long_text`, `time.time`, `len`, `OpenAI`, `enumerate`, `client.embeddings.create`, `str` reveal the main execution path / 这些调用体现了主要执行链路。
