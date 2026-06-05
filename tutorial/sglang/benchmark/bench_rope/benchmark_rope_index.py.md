# benchmark_rope_index.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/bench_rope/benchmark_rope_index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on rope index. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 rope index 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 9-18: Imports and setup / 导入与初始化
```python
import argparse
import math
import time
from dataclasses import dataclass, field
from typing import Any

import numpy as np
import torch

from sglang.srt.layers.rotary_embedding import MRotaryEmbedding
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、准备张量并调用 GPU 内核。

### Lines 25-25: Class `DummyVisionConfig` declaration / 类 `DummyVisionConfig` 声明
```python
class DummyVisionConfig:
```
**EN:** This block introduces class `DummyVisionConfig`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `DummyVisionConfig`，用于通过统一接口组织相关行为。

### Lines 26-26: Class-level state / 类级状态
```python
    spatial_merge_size: int = 2
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。

### Lines 30-30: Class `DummyHFConfig` declaration / 类 `DummyHFConfig` 声明
```python
class DummyHFConfig:
```
**EN:** This block introduces class `DummyHFConfig`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `DummyHFConfig`，用于通过统一接口组织相关行为。

### Lines 31-36: Class-level state / 类级状态
```python
    image_token_id: int = 32000
    video_start_token_id: int = 32001
    video_end_token_id: int = 32002
    vision_config: DummyVisionConfig = field(
        default_factory=lambda: DummyVisionConfig(spatial_merge_size=2)
    )
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it processes tokenized prompts or decoded outputs.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会处理分词后的提示词或解码后的输出。

### Lines 42-51: Function `calculate_stats` / 函数 `calculate_stats`
```python
def calculate_stats(times: list[float]) -> dict[str, float]:
    """Calculate statistics from a list of times."""
    times_array = np.array(times, dtype=np.float64)
    return {
        "mean": float(np.mean(times_array)),
        "median": float(np.median(times_array)),
        "p99": float(np.percentile(times_array, 99)),
        "min": float(np.min(times_array)),
        "max": float(np.max(times_array)),
    }
```
**EN:** `calculate_stats` is a function that implements the core logic for this scope. The docstring frames it as: Calculate statistics from a list of times. It returns `{'mean': float(np.mean(times_array)), 'median': float(np.median(times_array)), 'p99': float(np.pe...` to the caller. Notable calls include `np.array`, `float`, `np.mean`.
**CN:** `calculate_stats` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `{'mean': float(np.mean(times_array)), 'median': float(np.median(times_array)), 'p99': float(np.pe...`。其中较关键的调用包括 `np.array`, `float`, `np.mean`。

### Lines 54-56: Function `_sync` / 函数 `_sync`
```python
def _sync(device: torch.device):
    if device.type == "cuda":
        torch.cuda.synchronize()
```
**EN:** `_sync` is a function that prepares tensors and invokes GPU kernels. Notable calls include `torch.cuda.synchronize`.
**CN:** `_sync` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `torch.cuda.synchronize`。

### Lines 59-63: Function `_approx_hw` / 函数 `_approx_hw`
```python
def _approx_hw(patches: int, merge: int) -> tuple[int, int]:
    # want (h/merge)*(w/merge) ~= patches
    gh = int(math.sqrt(max(1, patches)))
    gw = max(1, patches // max(1, gh))
    return gh * merge, gw * merge
```
**EN:** `_approx_hw` is a function that implements the core logic for this scope. It returns `(gh * merge, gw * merge)` to the caller. Notable calls include `int`, `max`, `math.sqrt`.
**CN:** `_approx_hw` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `(gh * merge, gw * merge)`。其中较关键的调用包括 `int`, `max`, `math.sqrt`。

### Lines 66-221: Function `generate_test_data` / 函数 `generate_test_data`
```python
def generate_test_data(
    num_tokens: int,
    batch_size: int,
    hf_config: DummyHFConfig,
    dtype: torch.dtype,
    device: torch.device,
    pad_ratio: float,
    num_images_per_sample: int,
    image_patch_tokens: int,
    num_videos_per_sample: int,
    video_patch_tokens: int,
    seed: int,
):
    """
    Generate synthetic (input_ids, attention_mask, image_grid_thw, video_grid_thw).

    NOTE:
      - image_grid_thw / video_grid_thw are global lists across the entire batch in encounter order,
        matching the function's image_index/video_index behavior.
      - image patches are represented by repeated image_token_id.
      - video patches are represented by image_token_id wrapped with start/end tokens.
    """
    torch.manual_seed(seed)

    forbidden = {
        0,
        hf_config.image_token_id,
        hf_config.video_start_token_id,
        hf_config.video_end_token_id,
    }
    vocab_size = 50000

    def rand_text(n: int) -> torch.Tensor:
        # generate random ids not in forbidden
        out = torch.randint(1, vocab_size, (n,), device=device, dtype=torch.long)
        # fix forbidden by +1 until ok (cheap, deterministic enough for benchmark data)
        for bad in forbidden:
            out = torch.where(out == bad, out + 1, out)
        return out

    image_grids: list[list[int]] = []
    video_grids: list[list[int]] = []

    input_ids = torch.zeros((batch_size, num_tokens), device=device, dtype=torch.long)
    attention_mask = torch.zeros(
        (batch_size, num_tokens), device=device, dtype=torch.long
    )

    eff_len = int(round(num_tokens * (1.0 - pad_ratio)))
    eff_len = max(1, min(num_tokens, eff_len))

    min_needed = 1
    min_needed += num_images_per_sample * image_patch_tokens
    min_needed += num_videos_per_sample * (2 + video_patch_tokens)
    if eff_len < min_needed:
        num_images_per_sample = 0
        num_videos_per_sample = 0

    for b in range(batch_size):
        blocks: list[torch.Tensor] = []

        reserved = (
            num_images_per_sample * image_patch_tokens
            + num_videos_per_sample * (2 + video_patch_tokens)
        )
        reserved = min(reserved, max(0, eff_len - 1))
        text_budget = max(1, eff_len - reserved)

        n_text_chunks = num_images_per_sample + num_videos_per_sample + 1
        base = text_budget // n_text_chunks
        rem = text_budget % n_text_chunks
        text_chunks = [base + (1 if i < rem else 0) for i in range(n_text_chunks)]

        tci = 0
        for _ in range(num_images_per_sample):
            blocks.append(rand_text(text_chunks[tci]))
            tci += 1
            blocks.append(
                torch.full(
                    (image_patch_tokens,),
                    hf_config.image_token_id,
                    device=device,
                    dtype=torch.long,
                )
            )

            h, w = _approx_hw(
                image_patch_tokens, hf_config.vision_config.spatial_merge_size
            )
            image_grids.append([1, h, w])

        for _ in range(num_videos_per_sample):
            blocks.append(rand_text(text_chunks[tci]))
            tci += 1
            blocks.append(
                torch.tensor(
                    [hf_config.video_start_token_id], device=device, dtype=torch.long
                )
            )
            blocks.append(
                torch.full(
                    (video_patch_tokens,),
                    hf_config.image_token_id,
                    device=device,
                    dtype=torch.long,
                )
            )
            blocks.append(
                torch.tensor(
                    [hf_config.video_end_token_id], device=device, dtype=torch.long
                )
            )

            h, w = _approx_hw(
                video_patch_tokens, hf_config.vision_config.spatial_merge_size
            )
            # first field = group count used by code; set to 1
            video_grids.append([1, h, w])

        blocks.append(rand_text(text_chunks[tci]))

        tokens = torch.cat(blocks, dim=0)[:eff_len]
        pad = torch.zeros(
            (num_tokens - tokens.numel(),), device=device, dtype=torch.long
        )
        ids = torch.cat([tokens, pad], dim=0)

        mask = torch.cat(
            [
                torch.ones((tokens.numel(),), device=device, dtype=torch.long),
                torch.zeros(
                    (num_tokens - tokens.numel(),), device=device, dtype=torch.long
                ),
            ],
            dim=0,
        )

        input_ids[b] = ids
        attention_mask[b] = mask

    image_grid_thw = (
        torch.tensor(image_grids, device=device, dtype=torch.long)
        if len(image_grids)
        else None
    )
    video_grid_thw = (
        torch.tensor(video_grids, device=device, dtype=torch.long)
        if len(video_grids)
        else None
    )
    return (
        input_ids.to(dtype=torch.long),
        attention_mask.to(dtype=torch.long),
        image_grid_thw,
        video_grid_thw,
    )
```
**EN:** `generate_test_data` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. The docstring frames it as: Generate synthetic (input_ids, attention_mask, image_grid_thw, video_grid_thw). It returns `(input_ids.to(dtype=torch.long), attention_mask.to(dtype=torch.long), image_grid_thw, video_grid_...` to the caller. Notable calls include `torch.manual_seed`, `torch.zeros`, `int`.
**CN:** `generate_test_data` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(input_ids.to(dtype=torch.long), attention_mask.to(dtype=torch.long), image_grid_thw, video_grid_...`。其中较关键的调用包括 `torch.manual_seed`, `torch.zeros`, `int`。

### Lines 224-365: Function `benchmark_rope_index` / 函数 `benchmark_rope_index`
```python
def benchmark_rope_index(
    model_name: str,
    tp_size: int,
    num_tokens: int,
    batch_size: int,
    pad_ratio: float,
    spatial_merge_size: int,
    num_images: int,
    image_patch_tokens: int,
    num_videos: int,
    video_patch_tokens: int,
    dtype: torch.dtype,
    seed: int,
    warmup_iter: int,
    benchmark_iter: int,
    device: torch.device,
):
    torch.manual_seed(seed)
    hf_config = DummyHFConfig(
        image_token_id=32000,
        video_start_token_id=32001,
        video_end_token_id=32002,
        vision_config=DummyVisionConfig(spatial_merge_size=spatial_merge_size),
    )

    print(80 * "=")
    print(
        f"Evaluating: {model_name} tp_size={tp_size} "
        f"num_tokens={num_tokens} batch={batch_size} pad_ratio={pad_ratio} "
        f"images/sample={num_images} image_patch_tokens={image_patch_tokens} "
        f"videos/sample={num_videos} video_patch_tokens={video_patch_tokens} "
        f"dtype={dtype} device={device}"
    )

    input_ids, attention_mask, image_grid_thw, video_grid_thw = generate_test_data(
        num_tokens=num_tokens,
        batch_size=batch_size,
        hf_config=hf_config,
        dtype=dtype,
        device=device,
        pad_ratio=pad_ratio,
        num_images_per_sample=num_images,
        image_patch_tokens=image_patch_tokens,
        num_videos_per_sample=num_videos,
        video_patch_tokens=video_patch_tokens,
        seed=seed,
    )

    # Validate output shapes before benchmarking.
    has_mm = (image_grid_thw is not None) or (video_grid_thw is not None)
    if has_mm:
        pos, delta = MRotaryEmbedding.get_rope_index_glm4v(
            input_ids=input_ids,
            hf_config=hf_config,
            image_grid_thw=image_grid_thw,
            video_grid_thw=video_grid_thw,
            attention_mask=attention_mask,
        )
        assert pos.shape == (3, batch_size, num_tokens)
        assert delta.shape == (batch_size, 1)

    # Warm up
    for _ in range(warmup_iter):
        if has_mm:
            MRotaryEmbedding.get_rope_index_glm4v(
                input_ids=input_ids,
                hf_config=hf_config,
                image_grid_thw=image_grid_thw,
                video_grid_thw=video_grid_thw,
                attention_mask=attention_mask,
            )
        MRotaryEmbedding.get_rope_index_glm4v(
            input_ids=input_ids,
            hf_config=hf_config,
            image_grid_thw=None,
            video_grid_thw=None,
            attention_mask=attention_mask,
        )

    _sync(device)

    # Time multimodal branch
    multimodal_times = []
    for _ in range(benchmark_iter):
        _sync(device)
        start = time.time()
        MRotaryEmbedding.get_rope_index_glm4v(
            input_ids=input_ids,
            hf_config=hf_config,
            image_grid_thw=image_grid_thw,
            video_grid_thw=video_grid_thw,
            attention_mask=attention_mask,
        )
        _sync(device)
        multimodal_times.append(time.time() - start)

    # Time fallback branch
    fallback_times = []
    for _ in range(benchmark_iter):
        _sync(device)
        start = time.time()
        MRotaryEmbedding.get_rope_index_glm4v(
            input_ids=input_ids,
            hf_config=hf_config,
            image_grid_thw=None,
            video_grid_thw=None,
            attention_mask=attention_mask,
        )
        _sync(device)
        fallback_times.append(time.time() - start)

    multimodal_stats = calculate_stats(multimodal_times)
    fallback_stats = calculate_stats(fallback_times)

    print(f"\nPerformance for config (B={batch_size}, T={num_tokens}):")
    print(
        f"Multimodal: mean={multimodal_stats['mean']:.8f}s, "
        f"median={multimodal_stats['median']:.8f}s, "
        f"p99={multimodal_stats['p99']:.8f}s"
    )
    print(
        f"Fallback:   mean={fallback_stats['mean']:.8f}s, "
        f"median={fallback_stats['median']:.8f}s, "
        f"p99={fallback_stats['p99']:.8f}s"
    )

    if has_mm:
        speedup = (
            multimodal_stats["mean"] / fallback_stats["mean"]
            if fallback_stats["mean"] > 0
            else float("inf")
        )
        print(f"Fallback Speedup over Multimodal: {speedup:.8f}x")
    else:
        speedup = float("nan")
        print(
            "[INFO] num_tokens too small for multimodal segments; skip multimodal benchmark."
        )

    print(f"Fallback Speedup over Multimodal: {speedup:.8f}x")

    return multimodal_stats, fallback_stats, speedup
```
**EN:** `benchmark_rope_index` is a function that measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. It returns `(multimodal_stats, fallback_stats, speedup)` to the caller. Notable calls include `torch.manual_seed`, `DummyHFConfig`, `print`.
**CN:** `benchmark_rope_index` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(multimodal_stats, fallback_stats, speedup)`。其中较关键的调用包括 `torch.manual_seed`, `DummyHFConfig`, `print`。

### Lines 368-425: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Benchmark GLM4V get_rope_index_glm4v."
    )
    parser.add_argument("--model-name", type=str, default="GLM4V")
    parser.add_argument("--tp-size", type=int, default=1)
    parser.add_argument(
        "--device", type=str, default="cuda" if torch.cuda.is_available() else "cpu"
    )
    parser.add_argument("--warmup-iter", type=int, default=10)
    parser.add_argument("--benchmark-iter", type=int, default=100)
    parser.add_argument("--dtype", type=str, choices=["int64"], default="int64")
    parser.add_argument("--seed", type=int, default=0)

    # token length sweep
    parser.add_argument("--num-tokens", type=int, nargs="+", required=False)

    # data shape knobs
    parser.add_argument("--batch-size", type=int, default=1)
    parser.add_argument("--pad-ratio", type=float, default=0.0)
    parser.add_argument("--spatial-merge-size", type=int, default=2)
    parser.add_argument("--num-images", type=int, default=1)
    parser.add_argument("--image-patch-tokens", type=int, default=256)
    parser.add_argument("--num-videos", type=int, default=1)
    parser.add_argument("--video-patch-tokens", type=int, default=256)

    # output
    parser.add_argument("--out-dir", type=str, default=".")
    args = parser.parse_args()
    print(args)

    device = torch.device(args.device)

    if args.num_tokens is None:
        num_tokens_list = [2**i for i in range(0, 18)]
    else:
        num_tokens_list = args.num_tokens

    rows: list[dict[str, Any]] = []

    for num_tokens in num_tokens_list:
        multimodal_stats, fallback_stats, speedup = benchmark_rope_index(
            model_name=args.model_name,
            tp_size=args.tp_size,
            num_tokens=num_tokens,
            batch_size=args.batch_size,
            pad_ratio=args.pad_ratio,
            spatial_merge_size=args.spatial_merge_size,
            num_images=args.num_images,
            image_patch_tokens=args.image_patch_tokens,
            num_videos=args.num_videos,
            video_patch_tokens=args.video_patch_tokens,
            dtype=getattr(torch, args.dtype),
            seed=args.seed,
            warmup_iter=args.warmup_iter,
            benchmark_iter=args.benchmark_iter,
            device=device,
        )
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `math`, `time`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `numpy`, `torch`
- **Internal / 项目内部依赖**: `sglang.srt.layers.rotary_embedding`
