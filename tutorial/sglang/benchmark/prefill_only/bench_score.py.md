# bench_score.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/prefill_only/bench_score.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on prefill only score. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 Python 模块聚焦于 prefill only score 相关流程。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-59: Imports and setup / 导入与初始化
```python
"""
SGLang Scoring Benchmark Script

This script benchmarks SGLang's scoring API performance using HTTP requests.

Current Features:
- HTTP-only implementation (open source compatible)
- Uses /v1/score API endpoint directly
- Single item scoring with batching support
- Configurable RPS, duration, and batch sizes
- Progress tracking and detailed metrics
- Poisson and constant request distributions

Usage:
- Update configuration variables at the top of the file
- Ensure SGLang server is running on the configured HTTP_URL
- Run: python bench_score.py
- Each request will contain ITEM_COUNT_VALUES items for batch scoring

"""

import asyncio

from transformers import AutoTokenizer
from util import (
    BenchmarkConfig,
    generate_text_with_token_count,
    run_benchmark_main,
    run_generic_benchmark,
)

###############################################################################
# CONFIG
###############################################################################
# Create benchmark configuration
config = BenchmarkConfig()
config.rps_values = [160]
config.duration_secs_values = [60]
config.num_unique_requests = 100
config.distribution = "POISSON"
config.profile = False
config.freeze_gc = True  # Enable GC freeze functionality
# Profiler output directory - by default uses present working directory (pwd)
# Uncomment and customize the line below to override the default location:
# config.profiler_dir = "/sglang-oss-trace"

# HTTP Configuration
HTTP_URL = "http://localhost:30000/v1/score"  # Use score API directly

# Score API Config
# ITEM_COUNT_VALUES determines number of items per score request (batch size)
SCORE_QUERY_TOKENS = 120
SCORE_ITEM_TOKENS = 180
SCORE_MODEL_PATH = "Qwen/Qwen3-0.6B"
SCORE_LABEL_TOKEN_IDS = [9454, 2753]  # Yes/No token IDs
ITEM_COUNT_VALUES = [10]  # Number of items per request

# Special token to replicate for precise token counting
SPECIAL_REPLICATED_TOKEN = "<|im_start|>"
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

### Lines 65-112: Function `create_score_request_builder` / 函数 `create_score_request_builder`
```python
def create_score_request_builder():
    """Create a score request builder function with shared tokenizer."""
    # Load tokenizer once here to verify special token and get precise counts
    print("Loading tokenizer...")
    tokenizer = AutoTokenizer.from_pretrained(SCORE_MODEL_PATH)

    # Verify that our special token produces exactly 1 token
    special_token_count = len(
        tokenizer.encode(config.special_replicated_token, add_special_tokens=False)
    )
    print(
        f"Special token '{config.special_replicated_token}' produces "
        f"{special_token_count} token(s)"
    )

    def generate_text_with_token_count_local(num_toks):
        """Generate text with precise token count using replicated token."""
        return generate_text_with_token_count(
            SCORE_MODEL_PATH,
            num_toks,
            config.special_replicated_token,
            tokenizer=tokenizer,
        )

    def build_score_request(index: int, item_count: int) -> tuple:
        """Build a single score request."""
        try:
            # Generate query and items for score API
            query = generate_text_with_token_count_local(SCORE_QUERY_TOKENS)
            items = [
                generate_text_with_token_count_local(SCORE_ITEM_TOKENS)
                for _ in range(item_count)
            ]

            # Return as dict for score API format
            score_data = {
                "query": query,
                "items": items,
                "label_token_ids": SCORE_LABEL_TOKEN_IDS,
                "model": SCORE_MODEL_PATH,
            }
            return (index, score_data)

        except Exception as e:
            print(f"Error building request {index}: {e}")
            return (index, None)

    return build_score_request
```
**EN:** `create_score_request_builder` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. The docstring frames it as: Create a score request builder function with shared tokenizer. It returns `build_score_request` to the caller. Notable calls include `print`, `AutoTokenizer.from_pretrained`, `len`.
**CN:** `create_score_request_builder` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `build_score_request`。其中较关键的调用包括 `print`, `AutoTokenizer.from_pretrained`, `len`。

### Lines 115-117: Function `validate_score_response` / 函数 `validate_score_response`
```python
def validate_score_response(response_data: dict) -> bool:
    """Validate score API response."""
    return "scores" in response_data or "logprobs" in response_data
```
**EN:** `validate_score_response` is a function that computes evaluation scores and aggregate statistics. The docstring frames it as: Validate score API response. It returns `'scores' in response_data or 'logprobs' in response_data` to the caller.
**CN:** `validate_score_response` 是一个函数，用于计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `'scores' in response_data or 'logprobs' in response_data`。

### Lines 120-149: Function `build_warmup_score_request` / 函数 `build_warmup_score_request`
```python
def build_warmup_score_request() -> dict:
    """Build a warmup request for the score API."""
    # Load tokenizer once for warmup generation
    tokenizer = AutoTokenizer.from_pretrained(SCORE_MODEL_PATH)

    warmup_query = generate_text_with_token_count(
        SCORE_MODEL_PATH,
        SCORE_QUERY_TOKENS,
        config.special_replicated_token,
        tokenizer=tokenizer,
    )
    warmup_items = [
        generate_text_with_token_count(
            SCORE_MODEL_PATH,
            SCORE_ITEM_TOKENS,
            config.special_replicated_token,
            tokenizer=tokenizer,
        )
        for _ in range(3)
    ]

    return {
        "query": warmup_query,
        "items": warmup_items,
        "label_token_ids": SCORE_LABEL_TOKEN_IDS,
        "model": SCORE_MODEL_PATH,
        # Add missing parameters for consistency with the original warmup
        "apply_softmax": True,
        "item_first": False,
    }
```
**EN:** `build_warmup_score_request` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. The docstring frames it as: Build a warmup request for the score API. It returns `{'query': warmup_query, 'items': warmup_items, 'label_token_ids': SCORE_LABEL_TOKEN_IDS, 'model':...` to the caller. Notable calls include `AutoTokenizer.from_pretrained`, `generate_text_with_token_count`, `range`.
**CN:** `build_warmup_score_request` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `{'query': warmup_query, 'items': warmup_items, 'label_token_ids': SCORE_LABEL_TOKEN_IDS, 'model':...`。其中较关键的调用包括 `AutoTokenizer.from_pretrained`, `generate_text_with_token_count`, `range`。

### Lines 155-170: Async function `run_benchmark` / 异步函数 `run_benchmark`
```python
async def run_benchmark(rps, duration_secs, item_count):
    """Run a single benchmark with the given RPS value."""
    # Create the request builder function with shared tokenizer
    build_request_func = create_score_request_builder()

    return await run_generic_benchmark(
        rps=rps,
        duration_secs=duration_secs,
        item_count=item_count,
        config=config,
        http_url=HTTP_URL,
        build_request_func=build_request_func,
        response_validator=validate_score_response,
        api_name="SINGLE_ITEM_SCORING",
        request_description="score requests",
    )
```
**EN:** `run_benchmark` is a async function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. The docstring frames it as: Run a single benchmark with the given RPS value. It returns `await run_generic_benchmark(rps=rps, duration_secs=duration_secs, item_count=item_count, config=c...` to the caller. Notable calls include `create_score_request_builder`, `run_generic_benchmark`.
**CN:** `run_benchmark` 是一个异步函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `await run_generic_benchmark(rps=rps, duration_secs=duration_secs, item_count=item_count, config=c...`。其中较关键的调用包括 `create_score_request_builder`, `run_generic_benchmark`。

### Lines 173-188: Async function `main` / 异步函数 `main`
```python
async def main():
    """Main function that runs benchmarks for all RPS values."""
    additional_info = {
        "Query tokens per request": SCORE_QUERY_TOKENS,
        "Item tokens per item": SCORE_ITEM_TOKENS,
    }

    await run_benchmark_main(
        config,
        run_benchmark,
        "SINGLE_ITEM_SCORING",
        HTTP_URL,
        ITEM_COUNT_VALUES,
        additional_info,
        build_warmup_score_request,
    )
```
**EN:** `main` is a async function that measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and processes tokenized prompts or decoded outputs. The docstring frames it as: Main function that runs benchmarks for all RPS values. Notable calls include `run_benchmark_main`.
**CN:** `main` 是一个异步函数，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `run_benchmark_main`。

### Lines 191-192: CLI entry point / 命令行入口
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
- **Standard library / 标准库**: `asyncio`
- **Third-party / 第三方依赖**: `transformers`, `util`
