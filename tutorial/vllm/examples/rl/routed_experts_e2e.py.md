# routed_experts_e2e.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/rl/routed_experts_e2e.py`
- **Repository**: vllm-project/vllm
- **Purpose**: End-to-end example for routed experts capture with hybrid models / 演示 vLLM 示例目录中与 routed experts e2e 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
End-to-end example for routed experts capture with hybrid models.

Validates that:
1. routed_experts is returned in CompletionOutput for MoE models.
2. Expert IDs are within valid range.
3. Results are deterministic across runs (baseline vs reference).

Usage:
    python examples/rl/routed_experts_e2e.py \
        --model Qwen/Qwen3-30B-A3B \
        --tp 4 \
        --max-model-len 4096 \
        --num-prompts 20 \
        --max-new-tokens 50
"""
```
**EN:** End-to-end example for routed experts capture with hybrid models.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from __future__ import annotations

import argparse
import asyncio
import logging
import os
import uuid
from dataclasses import dataclass, field

import numpy as np

from vllm.engine.arg_utils import AsyncEngineArgs
```
**EN:** This block loads helper libraries such as __future__, argparse, asyncio, logging, and os and pulls in vLLM APIs like vllm.engine.arg_utils.
**CN:** 这一部分加载 __future__、argparse、asyncio、logging，以及 os 等辅助库，并引入 vllm.engine.arg_utils 等 vLLM API。

### Top-level setup
```python
logger = logging.getLogger(__name__)

DEFAULT_MODEL = "Qwen/Qwen3-30B-A3B"

TEST_PROMPTS = [
    "Hello, my name is",
    "The capital of France is",
    "Explain quantum computing in simple terms:",
    "Write a Python function that sorts a list:",
    "The meaning of life is",
    "In a distant galaxy, there was a",
    "The best way to learn programming is",
    "Once upon a time in a land far away,",
    "The theory of relativity states that",
    "How does photosynthesis work?",
    "Describe the process of machine learning:",
    "What are the benefits of exercise?",
    "The history of artificial intelligence began",
    "Translate the following to French: Hello world",
    "Summarize the plot of Romeo and Juliet:",
    "What is the difference between TCP and UDP?",
    "The water cycle consists of",
    "Explain how a neural network learns:",
    "The periodic table organizes elements by",
    "Write a haiku about the ocean:",
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as logger, DEFAULT_MODEL, and TEST_PROMPTS. It also performs early helper calls such as logging.getLogger.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 logger、DEFAULT_MODEL，以及 TEST_PROMPTS 等变量。它还会提前执行 logging.getLogger 等辅助调用。

### Class: InferenceResult
```python
class InferenceResult:
    """Result from a single inference run."""

    experts_list: list[np.ndarray] = field(default_factory=list)
    token_ids_list: list[list[int]] = field(default_factory=list)
    num_experts: int = 0
```
**EN:** Result from a single inference run..
**CN:** 该类封装了示例中使用的 InferenceResult 抽象。

### Async function: _run_async_inference
```python
async def _run_async_inference(
    engine_args: AsyncEngineArgs,
    prompts: list[str],
    max_new_tokens: int,
) -> InferenceResult:
    """Run inference using AsyncLLM."""
    from vllm.sampling_params import SamplingParams
    from vllm.v1.engine.async_llm import AsyncLLM

    engine = AsyncLLM.from_engine_args(engine_args)

    hf_config = engine.model_config.hf_text_config
    num_experts: int = getattr(hf_config, "num_experts", 0) or getattr(
        hf_config, "num_local_experts", 0
    )
    assert num_experts > 0, "Could not determine num_experts from model config"

    sampling_params = SamplingParams(
        temperature=0,
        max_tokens=max_new_tokens,
    # ... key logic omitted for brevity ...

    tasks = [_generate_one(p, i) for i, p in enumerate(prompts)]
    outputs = await asyncio.gather(*tasks)

    # Sort by original index to maintain prompt order
    outputs.sort(key=lambda x: x[0])

    result = InferenceResult(num_experts=num_experts)
    for _, routed, token_ids in outputs:
        result.experts_list.append(routed)
        result.token_ids_list.append(token_ids)

    engine.shutdown()
    return result
```
**EN:** Run inference using AsyncLLM.. It works with parameters such as engine_args, prompts, and max_new_tokens. Key operations include getattr, len, AsyncLLM.from_engine_args, SamplingParams, and outputs.sort. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式编排端到端工作流。它会处理 engine_args、prompts，以及 max_new_tokens 等参数。关键操作包括 getattr、len、AsyncLLM.from_engine_args、SamplingParams，以及 outputs.sort。其返回值会继续传给示例管线的下一阶段。

### Function: run_inference
```python
def run_inference(
    model: str,
    prompts: list[str],
    max_new_tokens: int = 50,
    tp: int = 1,
    max_model_len: int = 4096,
) -> InferenceResult:
    """Run inference with routed experts capture enabled via AsyncLLM."""
    engine_args = AsyncEngineArgs(
        model=model,
        enable_return_routed_experts=True,
        tensor_parallel_size=tp,
        max_model_len=max_model_len,
        disable_log_stats=True,
        attention_backend="FLASH_ATTN",
    )

    result = asyncio.run(_run_async_inference(engine_args, prompts, max_new_tokens))

    from vllm.platforms import current_platform

    if current_platform.is_cuda_alike():
        current_platform.empty_cache()

    return result
```
**EN:** Run inference with routed experts capture enabled via AsyncLLM.. It works with parameters such as model, prompts, max_new_tokens, tp, and max_model_len. Key operations include AsyncEngineArgs, asyncio.run, current_platform.is_cuda_alike, _run_async_inference, and current_platform.empty_cache. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 model、prompts、max_new_tokens、tp，以及 max_model_len 等参数。关键操作包括 AsyncEngineArgs、asyncio.run、current_platform.is_cuda_alike、_run_async_inference，以及 current_platform.empty_cache。其返回值会继续传给示例管线的下一阶段。

### Function: validate_expert_ids
```python
def validate_expert_ids(
    experts_list: list[np.ndarray],
    num_experts: int,
) -> None:
    """Check that all expert IDs are within valid range [0, num_experts)."""
    for i, experts in enumerate(experts_list):
        assert np.all(experts >= 0), (
            f"Prompt {i}: negative expert IDs found, min={experts.min()}"
        )
        assert np.all(experts < num_experts), (
            f"Prompt {i}: expert ID out of range [0, {num_experts}), "
            f"max={experts.max()}"
        )
```
**EN:** Check that all expert IDs are within valid range [0, num_experts).. It works with parameters such as experts_list and num_experts. Key operations include np.all, enumerate, experts.min, and experts.max.
**CN:** 该函数封装示例中的可复用步骤。它会处理 experts_list 和 num_experts 等参数。关键操作包括 np.all、enumerate、experts.min，以及 experts.max。

### Function: validate_shapes
```python
def validate_shapes(experts_list: list[np.ndarray]) -> None:
    """Check that all routed_experts arrays have at least 2 dimensions."""
    for i, experts in enumerate(experts_list):
        assert experts.ndim >= 2, (
            f"Prompt {i}: expected at least 2D array, got shape {experts.shape}"
        )
        logger.info("Prompt %d: routed_experts shape = %s", i, experts.shape)
```
**EN:** Check that all routed_experts arrays have at least 2 dimensions.. It works with parameters such as experts_list. Key operations include enumerate and logger.info.
**CN:** 该函数封装示例中的可复用步骤。它会处理 experts_list 等参数。关键操作包括 enumerate 和 logger.info。

### Function: compare_token_ids
```python
def compare_token_ids(
    baseline: list[list[int]],
    reference: list[list[int]],
) -> float:
    """Compare token IDs from two runs. Returns mismatch ratio."""
    assert len(baseline) == len(reference), (
        f"Length mismatch: {len(baseline)} vs {len(reference)}"
    )

    total_tokens = 0
    total_mismatches = 0

    for i, (base, ref) in enumerate(zip(baseline, reference)):
        min_len = min(len(base), len(ref))
        max_len = max(len(base), len(ref))
        matches = 0
        for a, b in zip(base[:min_len], ref[:min_len]):
            if a != b:
                break
            matches += 1

        total_mismatches += max_len - matches
        total_tokens += max_len

        if matches < min_len or len(base) != len(ref):
            print(
                f"  Prompt {i}: token_ids len={len(base)} vs {len(ref)}, "
                f"mismatches={max_len - matches}/{max_len}"
            )

    if total_tokens == 0:
        raise ValueError("No tokens to compare")

    mismatch_ratio = total_mismatches / total_tokens
    print(
        f"Token ID mismatches: {total_mismatches}/{total_tokens} ({mismatch_ratio:.4%})"
    )
    return mismatch_ratio
```
**EN:** Compare token IDs from two runs.. It works with parameters such as baseline and reference. Key operations include len, print, zip, enumerate, and min. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 baseline 和 reference 等参数。关键操作包括 len、print、zip、enumerate，以及 min。其返回值会继续传给示例管线的下一阶段。

### Function: compare_routed_experts
```python
def compare_routed_experts(
    baseline: list[np.ndarray],
    reference: list[np.ndarray],
    threshold: float = 0.05,
) -> float:
    """Compare two runs of routed experts. Returns mismatch ratio.

    Raises AssertionError if ratio exceeds threshold.
    """
    assert len(baseline) == len(reference), (
        f"Length mismatch: {len(baseline)} vs {len(reference)}"
    )

    total_elements = 0
    total_mismatches = 0

    for i, (base, ref) in enumerate(zip(baseline, reference)):
        min_len = min(len(base), len(ref))
        max_len = max(len(base), len(ref))
        if min_len == 0:
    # ... key logic omitted for brevity ...
        raise ValueError("No elements to compare")

    mismatch_ratio = total_mismatches / total_elements
    print(
        f"Routed experts mismatches: {total_mismatches}/{total_elements} "
        f"({mismatch_ratio:.4%})"
    )

    assert mismatch_ratio < threshold, (
        f"Too many mismatches: {total_mismatches}/{total_elements} "
        f"({mismatch_ratio:.4%}) exceeds threshold {threshold:.4%}"
    )

    return mismatch_ratio
```
**EN:** Compare two runs of routed experts.. It works with parameters such as baseline, reference, and threshold. Key operations include len, print, zip, enumerate, and min. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 baseline、reference，以及 threshold 等参数。关键操作包括 len、print、zip、enumerate，以及 min。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    os.environ.setdefault("VLLM_BATCH_INVARIANT", "1")

    parser = argparse.ArgumentParser(
        description="Test routed experts capture for MoE models"
    )
    parser.add_argument("--model", type=str, default=DEFAULT_MODEL)
    parser.add_argument("--tp", type=int, default=1)
    parser.add_argument("--max-model-len", type=int, default=4096)
    parser.add_argument("--num-prompts", type=int, default=20)
    parser.add_argument("--max-new-tokens", type=int, default=50)
    parser.add_argument(
        "--deterministic",
        action="store_true",
        help="Run twice and compare results for determinism check",
    )
    parser.add_argument(
        "--threshold",
        type=float,
        default=0.05,
    # ... key logic omitted for brevity ...
        print("\n--- Routed Experts ---")
        expert_mismatch = compare_routed_experts(
            baseline.experts_list,
            reference.experts_list,
            threshold=args.threshold,
        )

        print(
            f"\nDeterminism check passed. "
            f"Token mismatch: {token_mismatch:.4%}, "
            f"Expert mismatch: {expert_mismatch:.4%}"
        )

    print("\nAll tests passed!")
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print, parser.add_argument, run_inference, validate_expert_ids, and len.
**CN:** 该函数编排端到端工作流。关键操作包括 print、parser.add_argument、run_inference、validate_expert_ids，以及 len。

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
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm.engine.arg_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `__future__`, `argparse`, `asyncio`, `logging`, `os`, `uuid`, `dataclasses`, `numpy` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `InferenceResult`, `_run_async_inference`, `run_inference`, `validate_expert_ids`, `validate_shapes`, `compare_token_ids`, `compare_routed_experts`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `logging.getLogger`, `field`, `getattr`, `len`, `AsyncLLM.from_engine_args`, `SamplingParams`, `outputs.sort`, `InferenceResult` reveal the main execution path / 这些调用体现了主要执行链路。
