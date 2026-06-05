# mrcr_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/evals/mrcr/mrcr_eval.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MRCR long-context evaluation for vLLM's OpenAI-compatible server. / 该文件主要围绕 MRCR Eval 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-33)
```python
#!/usr/bin/env python3
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""MRCR long-context evaluation for vLLM's OpenAI-compatible server.

Streams samples from `openai/mrcr` on HuggingFace, sends chat completions to
the server, and scores each response with a prefix-gated SequenceMatcher ratio
against the reference answer.
"""

import argparse
import asyncio
import json
import time
from difflib import SequenceMatcher

import aiohttp
import numpy as np
import requests
from tqdm.asyncio import tqdm

DATASET_REPO = "openai/mrcr"
NEEDLE_SHARDS = {
    2: "2needle/2needle_0.parquet",
    4: "4needle/4needle_0.parquet",
    8: "8needle/8needle_0.parquet",
}
# Reserve headroom for chat-template tokens on top of the messages.
PROMPT_SAFETY_BUFFER = 256
# Pre-filter heuristic before the authoritative /tokenize check.
CHARS_PER_TOKEN = 4
# Skip chain-of-thought on reasoning models; ignored by non-reasoning templates.
DEFAULT_EXTRA_BODY: dict = {"chat_template_kwargs": {"enable_thinking": False}}
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `argparse`, `asyncio`, `aiohttp`, `numpy`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: discover_server_model (lines 36-44)
```python
def discover_server_model(base_url: str) -> tuple[str, int | None]:
    """Return (model_id, max_model_len) from /v1/models."""
    resp = requests.get(f"{base_url}/v1/models", timeout=30)
    resp.raise_for_status()
    data = resp.json().get("data", [])
    if not data:
        raise RuntimeError(f"No models advertised at {base_url}/v1/models")
    entry = data[0]
    return entry["id"], entry.get("max_model_len")
```
**EN:** Return (model_id, max_model_len) from /v1/models. It coordinates operations such as `requests.get`, `resp.raise_for_status`, `resp.json().get`.
**CN:** 该辅助函数为 Discover Server Model 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `requests.get`, `resp.raise_for_status`, `resp.json().get` 等操作。

### Helper: count_chat_tokens (lines 47-55)
```python
def count_chat_tokens(base_url: str, model: str, messages: list[dict]) -> int:
    """Return the chat-template-rendered token count via /tokenize."""
    resp = requests.post(
        f"{base_url}/tokenize",
        json={"model": model, "messages": messages, "add_generation_prompt": True},
        timeout=120,
    )
    resp.raise_for_status()
    return int(resp.json()["count"])
```
**EN:** Return the chat-template-rendered token count via /tokenize. It coordinates operations such as `requests.post`, `resp.raise_for_status`, `int`.
**CN:** 该辅助函数为 Count Chat Tokens 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `requests.post`, `resp.raise_for_status`, `int` 等操作。

### Helper: _load_mrcr_samples (lines 58-120)
```python
def _load_mrcr_samples(
    needles: list[int],
    max_prompt_tokens: int,
    num_samples: int,
    seed: int,
    base_url: str,
    model_name: str,
) -> list[dict]:
    """Stream MRCR samples balanced across needle buckets, token-verified."""
    try:
        from datasets import load_dataset
    except ImportError as e:
        raise ImportError(
            "MRCR eval requires `datasets`. Install with: uv pip install datasets"
        ) from e

    max_chars = max_prompt_tokens * CHARS_PER_TOKEN
    per_bucket = num_samples // len(needles)
    leftover = num_samples - per_bucket * len(needles)
# ... omitted for brevity ...
                    "n_needles": int(row["n_needles"]),
                    "n_tokens": n_tokens,
                }
            )
            taken += 1
            if taken >= target:
                break

        if taken < target:
            print(f"Warning: only {taken}/{target} samples for n_needles={n}")

    if not samples:
        raise RuntimeError("No MRCR samples fit; loosen max_prompt_tokens.")
    return samples
```
**EN:** Stream MRCR samples balanced across needle buckets, token-verified. It coordinates operations such as `enumerate`, `len`, `load_dataset(DATASET_REPO, data_files=NEEDLE_SHARDS[n], split='train', streaming=True).shuffle`.
**CN:** 该辅助函数为 Load MRCR Samples 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `enumerate`, `len`, `load_dataset(DATASET_REPO, data_files=NEEDLE_SHARDS[n], split='train', streaming=True).shuffle` 等操作。

### Helper: score_mrcr (lines 123-128)
```python
def score_mrcr(response: str, answer: str, random_prefix: str) -> float:
    """Prefix-gated SequenceMatcher ratio; 0 if the prefix is missing."""
    if not response.startswith(random_prefix):
        return 0.0
    stripped = response[len(random_prefix) :]
    return SequenceMatcher(a=answer, b=stripped, autojunk=False).ratio()
```
**EN:** Prefix-gated SequenceMatcher ratio; 0 if the prefix is missing. It coordinates operations such as `SequenceMatcher(a=answer, b=stripped, autojunk=False).ratio`, `response.startswith`, `len`.
**CN:** 该辅助函数为 Score MRCR 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `SequenceMatcher(a=answer, b=stripped, autojunk=False).ratio`, `response.startswith`, `len` 等操作。

### Helper: _call_chat (lines 131-158)
```python
async def _call_chat(
    session: aiohttp.ClientSession,
    url: str,
    model: str,
    messages: list[dict],
    max_tokens: int,
    temperature: float,
    seed: int | None,
    extra_body: dict,
) -> tuple[str, int]:
    data = {
        "model": model,
        "messages": messages,
        "temperature": temperature,
        "max_tokens": max_tokens,
        **extra_body,
    }
    if seed is not None:
        data["seed"] = seed
    try:
        async with session.post(f"{url}/v1/chat/completions", json=data) as resp:
            resp.raise_for_status()
            result = await resp.json()
            text = result["choices"][0]["message"]["content"] or ""
            return text, result.get("usage", {}).get("completion_tokens", 0)
    except Exception as e:
        print(f"chat request failed: {e}")
        return "", 0
```
**EN:** Async Implements a reusable helper for Call Chat, reducing duplication across related tests. It coordinates operations such as `session.post`, `resp.raise_for_status`, `print`.
**CN:** 该辅助函数为 Call Chat 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `session.post`, `resp.raise_for_status`, `print` 等操作。

### Helper: evaluate_mrcr (lines 161-271)
```python
def evaluate_mrcr(
    model_name: str | None = None,
    num_samples: int = 40,
    needles: list[int] | None = None,
    max_prompt_tokens: int | None = None,
    max_tokens: int = 2048,
    host: str = "http://127.0.0.1",
    port: int = 8000,
    temperature: float = 0.0,
    seed: int | None = 42,
    concurrency: int = 8,
    extra_body: dict | None = None,
) -> dict:
    """Run MRCR against a vLLM server; auto-discovers model and context."""
    needles = needles or [2, 4, 8]
    extra_body = DEFAULT_EXTRA_BODY if extra_body is None else extra_body
    base_url = f"{host}:{port}"

    discovered_model, server_max_len = discover_server_model(base_url)
# ... omitted for brevity ...
    total_out = int(sum(out_tokens))
    return {
        "model": model_name,
        "match_ratio": float(scores.mean()),
        "prefix_hit_rate": float(prefix_hits.mean()),
        "per_needle": per_needle,
        "num_samples": len(samples),
        "latency": latency,
        "total_output_tokens": total_out,
        "tokens_per_second": total_out / latency if latency > 0 else 0.0,
        "max_tokens": max_tokens,
        "needles": needles,
        "max_prompt_tokens": max_prompt_tokens,
    }
```
**EN:** Run MRCR against a vLLM server; auto-discovers model and context. It coordinates operations such as `discover_server_model`, `print`, `_load_mrcr_samples`.
**CN:** 该辅助函数为 Evaluate MRCR 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `discover_server_model`, `print`, `_load_mrcr_samples` 等操作。

### Helper: main (lines 274-329)
```python
def main() -> None:
    p = argparse.ArgumentParser(description="MRCR evaluation for vLLM serve")
    p.add_argument("--model", default=None, help="Default: discovered from /v1/models")
    p.add_argument("--num-samples", type=int, default=40)
    p.add_argument(
        "--needles", type=int, nargs="+", default=[2, 4, 8], choices=[2, 4, 8]
    )
    p.add_argument(
        "--max-prompt-tokens",
        type=int,
        default=None,
        help="Default: server max_model_len - max_tokens - buffer",
    )
    p.add_argument("--max-tokens", type=int, default=2048)
    p.add_argument("--host", default="http://127.0.0.1")
    p.add_argument("--port", type=int, default=8000)
    p.add_argument("--temperature", type=float, default=0.0)
    p.add_argument("--seed", type=int, default=42)
    p.add_argument("--concurrency", type=int, default=8)
# ... omitted for brevity ...
    )

    print("\nResults:")
    print(f"  match_ratio:     {result['match_ratio']:.4f}")
    print(f"  prefix_hit_rate: {result['prefix_hit_rate']:.4f}")
    for k, v in result["per_needle"].items():
        print(f"  {k}: {v:.4f}")
    print(f"  samples:         {result['num_samples']}")
    print(f"  latency:         {result['latency']:.1f}s")
    print(f"  output tok/s:    {result['tokens_per_second']:.1f}")

    if args.save_results:
        with open(args.save_results, "w") as f:
            json.dump(result, f, indent=2)
```
**EN:** Implements a reusable helper for Main, reducing duplication across related tests. It coordinates operations such as `argparse.ArgumentParser`, `p.add_argument`, `p.parse_args`.
**CN:** 该辅助函数为 Main 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `argparse.ArgumentParser`, `p.add_argument`, `p.parse_args` 等操作。

### Conditional block (lines 332-333)
```python
if __name__ == "__main__":
    main()
```
**EN:** Handles a conditional branch that adjusts test behavior for a specific environment, backend, or runtime constraint.
**CN:** 这里处理条件分支，用于针对特定环境、后端或运行时限制调整测试行为。

## Key Concepts / 关键概念
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Benchmarking / 基准测试**
  - **EN:** The tests validate CLI arguments, datasets, or metrics used by benchmark workflows.
  - **CN:** 测试验证基准流程使用的命令行参数、数据集或指标。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `time`, `difflib`
- **Third-party / 第三方依赖**: `aiohttp`, `numpy`, `requests`, `tqdm.asyncio`, `datasets`
