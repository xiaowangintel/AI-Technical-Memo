# bench_embeddings.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/prefill_only/bench_embeddings.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on prefill only embeddings. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 prefill only embeddings 相关流程。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-75: Imports and setup / 导入与初始化
```python
"""
SGLang Embeddings Benchmark Script

This script benchmarks SGLang's /v1/embeddings API performance using HTTP requests.

Features:
- HTTP-only implementation
- Uses /v1/embeddings API endpoint directly
- Configurable RPS, duration, and batch sizes
- Progress tracking and detailed metrics
- Poisson and constant request distributions

Usage:
- Update configuration variables at the top of the file
- Ensure SGLang server is running on the configured HTTP_URL
- Run: python bench_embeddings.py
"""

import asyncio
import logging
from typing import Optional

from transformers import AutoTokenizer
from util import (
    BenchmarkConfig,
    generate_text_with_token_count,
    run_benchmark_main,
    run_generic_benchmark,
)

# Configure logging
logging.basicConfig(
    level=logging.INFO, format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
logger = logging.getLogger(__name__)

###############################################################################
# CONFIG
###############################################################################
# Create benchmark configuration
config = BenchmarkConfig()
config.rps_values = [500]
config.duration_secs_values = [60]
config.num_unique_requests = 100
config.distribution = "POISSON"
config.profile = False
config.freeze_gc = True  # Enable GC freeze functionality
# Profiler output directory - by default uses present working directory (pwd)
# Uncomment and customize the line below to override the default location:
# config.profiler_dir = "/sglang-oss-trace"

# HTTP Configuration
HTTP_URL = "http://localhost:30000/v1/embeddings"

# Embeddings API Config
EMBEDDINGS_MODEL_PATH = "Qwen/Qwen3-Embedding-0.6B"
BATCH_SIZE = [1]  # Number of items per request (batch size)

# Configurable input token length
EMBEDDINGS_INPUT_TOKENS = 500  # Default token length
MATRYOSHKA_DIMENSIONS: Optional[int] = (
    None  # Set to None to disable matryoshka embeddings
)

# Load tokenizer once for embeddings text generation
print("Loading tokenizer for embeddings input generation...")
embeddings_tokenizer = AutoTokenizer.from_pretrained(EMBEDDINGS_MODEL_PATH)

# Generate input text with the specified token length using pre-loaded tokenizer
EMBEDDINGS_INPUT_TEXT = generate_text_with_token_count(
    EMBEDDINGS_MODEL_PATH,
    EMBEDDINGS_INPUT_TOKENS,
    config.special_replicated_token,
    tokenizer=embeddings_tokenizer,
)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

### Lines 81-97: Function `build_embeddings_request` / 函数 `build_embeddings_request`
```python
def build_embeddings_request(index: int, item_count: int) -> tuple:
    """Build a single embeddings request."""
    try:
        # For embeddings, input can be a string or list of strings
        if item_count == 1:
            input_data = EMBEDDINGS_INPUT_TEXT
        else:
            input_data = [EMBEDDINGS_INPUT_TEXT for _ in range(item_count)]
        req = {
            "input": input_data,
            "model": EMBEDDINGS_MODEL_PATH,
            "dimensions": MATRYOSHKA_DIMENSIONS,
        }
        return (index, req)
    except Exception as e:
        logger.error(f"Error building request {index}: {e}")
        return (index, None)
```
**EN:** `build_embeddings_request` is a function that implements the core logic for this scope. The docstring frames it as: Build a single embeddings request. It returns `(index, req)` to the caller. Notable calls include `logger.error`, `range`.
**CN:** `build_embeddings_request` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(index, req)`。其中较关键的调用包括 `logger.error`, `range`。

### Lines 100-107: Function `validate_embeddings_response` / 函数 `validate_embeddings_response`
```python
def validate_embeddings_response(response_data: dict) -> bool:
    """Validate embeddings API response."""
    return (
        "data" in response_data
        and len(response_data["data"][0]["embedding"]) == MATRYOSHKA_DIMENSIONS
        if MATRYOSHKA_DIMENSIONS
        else True
    )
```
**EN:** `validate_embeddings_response` is a function that implements the core logic for this scope. The docstring frames it as: Validate embeddings API response. It returns `'data' in response_data and len(response_data['data'][0]['embedding']) == MATRYOSHKA_DIMENSIONS i...` to the caller. Notable calls include `len`.
**CN:** `validate_embeddings_response` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `'data' in response_data and len(response_data['data'][0]['embedding']) == MATRYOSHKA_DIMENSIONS i...`。其中较关键的调用包括 `len`。

### Lines 110-116: Function `build_warmup_embeddings_request` / 函数 `build_warmup_embeddings_request`
```python
def build_warmup_embeddings_request() -> dict:
    """Build a warmup request for the embeddings API."""
    return {
        "input": EMBEDDINGS_INPUT_TEXT,
        "model": EMBEDDINGS_MODEL_PATH,
        "dimensions": MATRYOSHKA_DIMENSIONS,
    }
```
**EN:** `build_warmup_embeddings_request` is a function that implements the core logic for this scope. The docstring frames it as: Build a warmup request for the embeddings API. It returns `{'input': EMBEDDINGS_INPUT_TEXT, 'model': EMBEDDINGS_MODEL_PATH, 'dimensions': MATRYOSHKA_DIMENSI...` to the caller.
**CN:** `build_warmup_embeddings_request` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `{'input': EMBEDDINGS_INPUT_TEXT, 'model': EMBEDDINGS_MODEL_PATH, 'dimensions': MATRYOSHKA_DIMENSI...`。

### Lines 122-134: Async function `run_benchmark` / 异步函数 `run_benchmark`
```python
async def run_benchmark(rps, duration_secs, item_count):
    """Run a single embeddings benchmark with the given RPS value."""
    return await run_generic_benchmark(
        rps=rps,
        duration_secs=duration_secs,
        item_count=item_count,
        config=config,
        http_url=HTTP_URL,
        build_request_func=build_embeddings_request,
        response_validator=validate_embeddings_response,
        api_name="EMBEDDINGS",
        request_description="embeddings requests",
    )
```
**EN:** `run_benchmark` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Run a single embeddings benchmark with the given RPS value. It returns `await run_generic_benchmark(rps=rps, duration_secs=duration_secs, item_count=item_count, config=c...` to the caller. Notable calls include `run_generic_benchmark`.
**CN:** `run_benchmark` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `await run_generic_benchmark(rps=rps, duration_secs=duration_secs, item_count=item_count, config=c...`。其中较关键的调用包括 `run_generic_benchmark`。

### Lines 137-155: Async function `main` / 异步函数 `main`
```python
async def main():
    additional_info = {
        "Input text length": f"{EMBEDDINGS_INPUT_TOKENS} tokens",
        "Input text preview": (
            EMBEDDINGS_INPUT_TEXT[:100] + "..."
            if len(EMBEDDINGS_INPUT_TEXT) > 100
            else EMBEDDINGS_INPUT_TEXT
        ),
    }

    await run_benchmark_main(
        config,
        run_benchmark,
        "EMBEDDINGS",
        HTTP_URL,
        BATCH_SIZE,
        additional_info,
        build_warmup_embeddings_request,
    )
```
**EN:** `main` is a async function that measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and processes tokenized prompts or decoded outputs. Notable calls include `run_benchmark_main`, `len`.
**CN:** `main` 是一个异步函数，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `run_benchmark_main`, `len`。

### Lines 158-159: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    asyncio.run(main())
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it coordinates asynchronous or parallel execution.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会协调异步或并行执行。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `logging`, `typing`
- **Third-party / 第三方依赖**: `transformers`, `util`
