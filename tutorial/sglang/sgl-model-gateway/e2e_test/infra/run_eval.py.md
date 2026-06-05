# run_eval.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/run_eval.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises run eval behavior in the end-to-end infrastructure layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试基础设施 中与 run eval 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Module docstring
```python
"""MMLU evaluation runner for E2E tests.

Simplified evaluation runner that uses local eval implementations
with cleaner logging for CI/CD environments.

Usage:
    from infra.run_eval import run_eval
    from types import SimpleNamespace

    args = SimpleNamespace(
        base_url="http://127.0.0.1:30000",
        model="meta-llama/Llama-3.1-8B-Instruct",
        eval_name="mmlu",
        num_examples=64,
        num_threads=32,
        temperature=0.1,
    )
    metrics = run_eval(args)
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 21-27: Imports and dependencies
```python
from __future__ import annotations

import logging
import os
import time
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any
```
**EN:** This block imports `__future__`, `logging`, `os`, `time`, and 2 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 29-30: If block
```python
if TYPE_CHECKING:
    from .simple_eval_common import Eval
```
**EN:** This top-level block contributes supporting logic to the module and is executed or defined as part of file initialization.
**CN:** 这个顶层代码块为模块提供辅助逻辑，并在文件初始化阶段被执行或定义。

### Lines 32-33: Imports and dependencies
```python
from .simple_eval_common import ChatCompletionSampler, set_ulimit
from .simple_eval_mmlu import MMLU_DATASET_URL
```
**EN:** This block imports `.simple_eval_common`, `.simple_eval_mmlu`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 35-35: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 36-50: Class `EvalConfig`
```python


@dataclass
class EvalConfig:
    """Configuration for running an evaluation."""

    base_url: str
    model: str | None = None
    eval_name: str = "mmlu"
    num_examples: int = 64
    num_threads: int = 32
    temperature: float = 0.0
    max_tokens: int = 2048
    host: str = "127.0.0.1"
    port: int = 30000
```
**EN:** Class `EvalConfig` groups related state and behavior. It exposes 0 method(s) that implement the module's primary abstraction.
**CN:** 类 `EvalConfig` 将相关状态与行为封装在一起，提供 0 个方法来实现本模块的核心抽象。

### Lines 53-60: Helper function `_get_eval`
```python
def _get_eval(eval_name: str, num_examples: int, num_threads: int) -> "Eval":
    """Get the evaluation object by name."""
    if eval_name == "mmlu":
        from .simple_eval_mmlu import MMLUEval

        return MMLUEval(MMLU_DATASET_URL, num_examples, num_threads)
    else:
        raise ValueError(f"Unknown eval: {eval_name}. Supported: mmlu")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 63-138: Helper function `run_eval`
```python
def run_eval(args: Any) -> dict:
    """Run an evaluation and return metrics.

    Args:
        args: Configuration object with attributes:
            - base_url: Base URL of the server (e.g., "http://127.0.0.1:30000")
            - model: Model name/path (optional, will be auto-detected)
            - eval_name: Evaluation name ("mmlu")
            - num_examples: Number of examples to evaluate
            - num_threads: Number of parallel threads
            - temperature: Sampling temperature

    Returns:
        Dict with metrics including 'score' key.
    """
    set_ulimit()

    if "OPENAI_API_KEY" not in os.environ:
        os.environ["OPENAI_API_KEY"] = "EMPTY"

    # Build base URL
    base_url = getattr(args, "base_url", None)
    if base_url:
        base_url = base_url.rstrip("/")  # Remove trailing slashes
        if not base_url.endswith("/v1"):
            base_url = f"{base_url}/v1"
    else:
        host = getattr(args, "host", "127.0.0.1")
        port = getattr(args, "port", 30000)
        base_url = f"http://{host}:{port}/v1"

    eval_name = getattr(args, "eval_name", "mmlu")
    num_examples = getattr(args, "num_examples", 64)
    num_threads = getattr(args, "num_threads", 32)
    temperature = getattr(args, "temperature", 0.0)
    max_tokens = getattr(args, "max_tokens", 2048)
    model = getattr(args, "model", None)

    logger.info(
        "Starting %s eval: %d examples, %d threads, temp=%.2f",
        eval_name,
        num_examples,
        num_threads,
        temperature,
    )

    # Create sampler
    sampler = ChatCompletionSampler(
        model=model,
        max_tokens=max_tokens,
        base_url=base_url,
        temperature=temperature,
    )

    # Get eval object
    eval_obj = _get_eval(eval_name, num_examples, num_threads)

    # Run evaluation
    start_time = time.perf_counter()
    result = eval_obj(sampler)
    latency = time.perf_counter() - start_time

    # Build metrics
    metrics = result.metrics.copy() if result.metrics else {}
    metrics["score"] = result.score
    metrics["latency"] = latency

    logger.info(
        "%s eval complete: score=%.3f, latency=%.1fs, model=%s",
        eval_name,
        result.score,
        latency,
        sampler.model,
    )

    return metrics
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排
- Primary classes: `EvalConfig` / 主要类：`EvalConfig`
- Reusable functions: `_get_eval`, `run_eval` / 可复用函数：`_get_eval`, `run_eval`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `logging`, `os`, `time`, `typing`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: `.simple_eval_common`, `.simple_eval_mmlu`
