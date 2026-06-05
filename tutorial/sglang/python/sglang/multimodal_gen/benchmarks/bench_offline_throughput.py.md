# bench_offline_throughput.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/benchmarks/bench_offline_throughput.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `BatchOutput`, `BenchArgs`, and `initialize_engine`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Benchmark offline throughput for multimodal generation models (Image/Video Generation). This script benchmarks generation throughput without running a server, using low-level APIs. / 该文件属于多模态生成模块。它围绕 `BatchOutput`、`BenchArgs` 和 `initialize_engine` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: module setup and imports / 模块初始化与导入
```python
"""
Benchmark offline throughput for multimodal generation models (Image/Video Generation).

This script benchmarks generation throughput without running a server, using low-level APIs.
It provides detailed metrics on throughput, latency, and resource utilization.

# Usage Examples

## Text-to-Video with VBench dataset
python -m sglang.multimodal_gen.benchmarks.bench_offline_throughput \\
    --model-path Wan-AI/Wan2.1-T2V-1.3B-Diffusers \\
    --dataset vbench \\
    --num-prompts 20 \\
    --batch-size 1 \\
    --width 512 --height 512 --num-frames 16

## Random dataset for stress testing
python -m sglang.multimodal_gen.benchmarks.bench_offline_throughput \\
    --model-path Wan-AI/Wan2.1-T2V-1.3B-Diffusers \\
    --dataset random \\
    --num-prompts 100 \\
    --batch-size 1 \\
    --num-inference-steps 20 \\
    --output-file results.json
"""

import argparse
import dataclasses
import json
import time
from dataclasses import dataclass
from typing import Any, Dict, List, Optional, Tuple

import torch
from tqdm import tqdm

from sglang.multimodal_gen.benchmarks.datasets import RandomDataset, VBenchDataset
from sglang.multimodal_gen.runtime.entrypoints.diffusion_generator import DiffGenerator
from sglang.multimodal_gen.runtime.server_args import ServerArgs, set_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import (
    configure_logger,
    init_logger,
)
from sglang.multimodal_gen.test.test_utils import print_divider, print_value_formatted
```
**EN:** This block establishes the module context and imports `argparse`, `dataclasses`, `json`, `time`, `typing`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `argparse`、`dataclasses`、`json`、`time`、`typing` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 46-46: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 50-52: `BatchOutput` class overview / `BatchOutput` 类概览
```python
class BatchOutput:
    """Container for batch generation results."""
```
**EN:** This block defines class `BatchOutput`. Container for batch generation results.
**CN:** 该代码块定义了类 `BatchOutput`。 它用于封装 batch output 相关行为。

### Lines 53-59: supporting statements / 辅助语句
```python
    latency: float = 0.0
    latency_per_sample: float = 0.0
    num_samples: int = 0
    total_frames: int = 0
    peak_memory_mb: float = 0.0
    success: bool = False
    error: str = ""
```
**EN:** This block gathers supporting statements inside `BatchOutput`. It updates names such as `latency`, `latency_per_sample`, `num_samples`, `total_frames`, `peak_memory_mb`, and `success`.
**CN:** 该代码块汇集了位于 `BatchOutput` 内部的辅助语句。 它会更新 `latency`、`latency_per_sample`、`num_samples`、`total_frames`、`peak_memory_mb` 和 `success` 等名称。

### Lines 63-66: `BenchArgs` class overview / `BenchArgs` 类概览
```python
class BenchArgs:
    """Benchmark configuration for multimodal generation."""

    # Diffusion Model Configuration
```
**EN:** This block defines class `BenchArgs`. Benchmark configuration for multimodal generation.
**CN:** 该代码块定义了类 `BenchArgs`。 它用于封装 bench args 相关行为。

### Lines 67-90: supporting statements / 辅助语句
```python
    num_inference_steps: int = 20
    guidance_scale: float = 7.5
    seed: int = 42
    disable_safety_checker: bool = False

    # Output Configuration
    width: int = 32
    height: int = 32
    num_frames: int = 1
    fps: int = 24

    # Dataset & Benchmark
    dataset: str = "random"
    dataset_path: str = ""
    task_name: str = "unknown"
    num_prompts: int = 10
    batch_size: int = 1
    random_request_config: str = None
    random_request_seed: int = 42

    # Benchmark Execution
    skip_warmup: bool = False
    output_file: str = ""
    disable_tqdm: bool = False
```
**EN:** This block gathers supporting statements inside `BenchArgs`. It updates names such as `num_inference_steps`, `guidance_scale`, `seed`, `disable_safety_checker`, `width`, and `height`.
**CN:** 该代码块汇集了位于 `BenchArgs` 内部的辅助语句。 它会更新 `num_inference_steps`、`guidance_scale`、`seed`、`disable_safety_checker`、`width` 和 `height` 等名称。

### Lines 92-187: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(parser: argparse.ArgumentParser):
        """Add benchmark-specific CLI arguments."""
        # Diffusion Model Configuration
        parser.add_argument(
            "--num-inference-steps",
            type=int,
            default=20,
            help="Number of denoising steps",
        )
        parser.add_argument(
            "--guidance-scale",
            type=float,
            default=7.5,
            help="Classifier-free guidance scale",
        )
        parser.add_argument("--seed", type=int, default=42, help="Random seed")
        parser.add_argument(
            "--disable-safety-checker",
            action="store_true",
            help="Disable NSFW detection",
        )

        # Output Configuration
        parser.add_argument("--width", type=int, default=32, help="Image/video width")
        parser.add_argument("--height", type=int, default=32, help="Image/video height")
        parser.add_argument(
            "--num-frames", type=int, default=1, help="Number of frames for video"
        )
        parser.add_argument("--fps", type=int, default=24, help="FPS for video")

        # Dataset & Benchmark
        parser.add_argument(
            "--dataset",
            type=str,
            default="random",
            choices=["vbench", "random"],
            help="Dataset to use",
        )
        parser.add_argument(
            "--dataset-path",
            type=str,
            default="",
            help="Path to dataset (prompts file or image directory)",
        )
        parser.add_argument(
            "--task-name",
            type=str,
            default="unknown",
            help="Task name for benchmark identification",
        )
        parser.add_argument(
            "--num-prompts",
            type=int,
            default=10,
            help="Total number of prompts to benchmark",
        )
        parser.add_argument(
            "--batch-size",
            type=int,
            default=1,
            help="Batch size per generation call (currently only bs=1 is supported)",
        )

        parser.add_argument(
            "--random-request-config",
            type=str,
            default=None,
            help=(
                "JSON string defining random request profiles. "
                "Each profile may contain: width, height, num_inference_steps, etc. "
                "The 'weight' field controls sampling probability (relative weight)."
            ),
        )
        parser.add_argument(
            "--random-request-seed",
            type=int,
            default=42,
            help="Random seed for sampling request profiles (default: 42).",
        )

        # Benchmark Execution
        parser.add_argument(
            "--skip-warmup", action="store_true", help="Skip warmup batch"
        )
        parser.add_argument(
            "--output-file",
            type=str,
            default="",
            help="Output JSON file for results (append mode)",
        )
        parser.add_argument(
            "--disable-tqdm",
            action="store_true",
            help="Disable progress bar",
        )
```
**EN:** This block defines method `add_cli_args` on `BenchArgs`. Add benchmark-specific CLI arguments. Key calls include `parser.add_argument`. Parameters such as `parser` drive the behavior in this section.
**CN:** 该代码块定义了 `BenchArgs` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `parser.add_argument`。 本段逻辑主要由 `parser` 等参数驱动。

### Lines 189-193: `from_cli_args` implementation / `from_cli_args` 实现
```python
    @classmethod
    def from_cli_args(cls, args: argparse.Namespace):
        """Create BenchArgs from parsed CLI arguments."""
        attrs = [attr.name for attr in dataclasses.fields(cls)]
        return cls(**{attr: getattr(args, attr) for attr in attrs})
```
**EN:** This block defines method `from_cli_args` on `BenchArgs`. Create BenchArgs from parsed CLI arguments. Key calls include `cls`, `dataclasses.fields`, and `getattr`. Parameters such as `args` drive the behavior in this section.
**CN:** 该代码块定义了 `BenchArgs` 的方法 `from_cli_args`。 它用于从…构造cli args。 关键调用包括 `cls`、`dataclasses.fields` 和 `getattr`。 本段逻辑主要由 `args` 等参数驱动。

### Lines 196-201: `initialize_engine` implementation / `initialize_engine` 实现
```python
def initialize_engine(server_args: ServerArgs) -> DiffGenerator:
    """Initialize diffusion pipeline engine."""
    logger.info("Initializing engine...")
    engine = DiffGenerator.from_server_args(server_args, local_mode=True)
    logger.info("Engine initialized successfully")
    return engine
```
**EN:** This block defines function `initialize_engine`. Initialize diffusion pipeline engine. Key calls include `logger.info`, and `DiffGenerator.from_server_args`. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `initialize_engine`。 它用于初始化engine。 关键调用包括 `logger.info` 和 `DiffGenerator.from_server_args`。 本段逻辑主要由 `server_args` 等参数驱动。

### Lines 204-246: `generate_batch` implementation / `generate_batch` 实现
```python
def generate_batch(
    engine: DiffGenerator,
    bench_args: BenchArgs,
    prompts: List[str],
    user_sampling_params: List[Dict[str, Any]],
) -> BatchOutput:
    """Generate batch of images/videos synchronously."""
    assert len(user_sampling_params) == len(prompts), (
        f"user_sampling_params length ({len(user_sampling_params)}) must match "
        f"prompts length ({len(prompts)})"
    )

    output = BatchOutput()
    start_time = time.perf_counter()

    torch.cuda.reset_peak_memory_stats()

    for prompt, params in zip(prompts, user_sampling_params):
        try:
            sampling_params_kwargs = dict(params)
            sampling_params_kwargs["prompt"] = prompt
            result = engine.generate(sampling_params_kwargs=sampling_params_kwargs)

            if result is not None:
                if isinstance(result, list):
                    output.total_frames += len(result)
                else:
                    output.total_frames += 1
            output.num_samples += 1
        except Exception as e:
            logger.error(f"Generation failed for prompt '{prompt[:50]}...': {e}")
            output.error = str(e)

    output.latency = time.perf_counter() - start_time
    output.latency_per_sample = output.latency / len(prompts) if prompts else 0.0
    output.success = output.num_samples > 0
    output.peak_memory_mb = torch.cuda.max_memory_allocated() / (1024 * 1024)

    logger.debug(
        f"Batch generated: {output.num_samples}/{len(prompts)} samples in {output.latency:.2f}s"
    )

    return output
```
**EN:** This block defines function `generate_batch`. Generate batch of images/videos synchronously. Key calls include `BatchOutput`, `time.perf_counter`, `torch.cuda.reset_peak_memory_stats`, `zip`, and `logger.debug`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `engine`, `bench_args`, `prompts`, and `user_sampling_params` drive the behavior in this section.
**CN:** 该代码块定义了函数 `generate_batch`。 它用于生成batch。 关键调用包括 `BatchOutput`、`time.perf_counter`、`torch.cuda.reset_peak_memory_stats`、`zip` 和 `logger.debug`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `engine`、`bench_args`、`prompts` 和 `user_sampling_params` 等参数驱动。

### Lines 249-294: `calculate_metrics` implementation / `calculate_metrics` 实现
```python
def calculate_metrics(
    outputs: List[BatchOutput],
    total_duration: float,
    resolution: Tuple[int, int, int],
    num_requests: int,
    all_sampling_params: Optional[List[Dict[str, Any]]] = None,
) -> Dict[str, Any]:
    """Calculate generation-specific throughput metrics."""
    successful = [o for o in outputs if o.success]
    num_success = sum(o.num_samples for o in successful)
    total_frames = sum(o.total_frames for o in successful)
    peak_memory = max((o.peak_memory_mb for o in outputs), default=0)

    width, height, frames = resolution
    if all_sampling_params:
        total_pixels = sum(
            p.get("width", width)
            * p.get("height", height)
            * p.get("num_frames", frames)
            for p in all_sampling_params[:num_success]
        )
    else:
        total_pixels = num_success * width * height * frames

    metrics = {
        "num_requests": num_requests,
        "successful_requests": num_success,
        "failed_requests": num_requests - num_success,
        "total_duration_seconds": total_duration,
        "total_frames_generated": total_frames,
        "total_pixels_generated": total_pixels,
        "images_per_second": num_success / total_duration if total_duration > 0 else 0,
        "frames_per_second": total_frames / total_duration if total_duration > 0 else 0,
        "megapixels_per_second": (
            total_pixels / (total_duration * 1e6) if total_duration > 0 else 0
        ),
        "requests_per_second": (
            num_success / total_duration if total_duration > 0 else 0
        ),
        "latency_per_request_seconds": (
            total_duration / num_success if num_success > 0 else 0
        ),
        "peak_memory_mb": peak_memory,
    }

    return metrics
```
**EN:** This block defines function `calculate_metrics`. Calculate generation-specific throughput metrics. Key calls include `sum`, `max`, and `p.get`. The implementation branches on conditions. Parameters such as `outputs`, `total_duration`, `resolution`, `num_requests`, and `all_sampling_params` drive the behavior in this section.
**CN:** 该代码块定义了函数 `calculate_metrics`。 它用于处理 calculate metrics 相关逻辑。 关键调用包括 `sum`、`max` 和 `p.get`。 实现中包含条件分支。 本段逻辑主要由 `outputs`、`total_duration`、`resolution`、`num_requests` 和 `all_sampling_params` 等参数驱动。

### Lines 297-396: `throughput_test` implementation / `throughput_test` 实现
```python
def throughput_test(
    server_args: ServerArgs,
    bench_args: BenchArgs,
) -> Dict[str, Any]:
    """Main throughput benchmark function."""
    configure_logger(server_args=server_args)
    logger.info("Starting offline throughput benchmark...")

    engine = initialize_engine(server_args)

    if bench_args.random_request_config and bench_args.dataset != "random":
        raise ValueError(
            "--random-request-config can only be used with --dataset random"
        )

    logger.info(f"Loading {bench_args.dataset} dataset...")
    if bench_args.dataset == "vbench":
        bench_args.task_name = engine.server_args.pipeline_config.task_type
        dataset = VBenchDataset(bench_args)
    elif bench_args.dataset == "random":
        dataset = RandomDataset(bench_args)
    else:
        raise ValueError(f"Unknown dataset: {bench_args.dataset}")

    _sampling_params = {
        "guidance_scale": bench_args.guidance_scale,
        "num_inference_steps": bench_args.num_inference_steps,
        "height": bench_args.height,
        "width": bench_args.width,
        "num_frames": bench_args.num_frames,
        "seed": bench_args.seed,
    }
    if bench_args.disable_safety_checker:
        _sampling_params["safety_checker"] = None

    total_count = min(bench_args.num_prompts, len(dataset))
    all_prompts = [dataset[i].prompt for i in range(total_count)]

    if bench_args.random_request_config:
        all_sampling_params = []
        for i in range(total_count):
            params = dict(_sampling_params)
            params.update(dataset.get_sampling_params(i))
            all_sampling_params.append(params)
    else:
        all_sampling_params = [_sampling_params] * total_count

    if not bench_args.skip_warmup:
        logger.info("Running warmup batch...")
        warmup_count = min(bench_args.batch_size, total_count)
        warmup_prompts = all_prompts[:warmup_count]
        warmup_sampling_params = all_sampling_params[:warmup_count]
        generate_batch(engine, bench_args, warmup_prompts, warmup_sampling_params)

    logger.info(f"Running benchmark with {bench_args.num_prompts} prompts...")
    outputs: List[BatchOutput] = []

    start_time = time.perf_counter()

    num_batches = (total_count + bench_args.batch_size - 1) // bench_args.batch_size
    pbar = tqdm(
        total=num_batches,
        disable=bench_args.disable_tqdm,
        desc="Benchmark",
    )

    for batch_start in range(0, total_count, bench_args.batch_size):
        batch_end = min(batch_start + bench_args.batch_size, total_count)
        batch_prompts = all_prompts[batch_start:batch_end]
        batch_sampling_params = all_sampling_params[batch_start:batch_end]

        batch_output = generate_batch(
            engine, bench_args, batch_prompts, batch_sampling_params
        )
        outputs.append(batch_output)

        pbar.update(1)

    pbar.close()
    total_duration = time.perf_counter() - start_time

    resolution = (bench_args.width, bench_args.height, bench_args.num_frames)
    metrics = calculate_metrics(
        outputs,
        total_duration,
        resolution=resolution,
        num_requests=total_count,
        all_sampling_params=all_sampling_params,
    )

    display_results(
        metrics,
        bench_args,
        model_path=server_args.model_path,
    )

    if bench_args.output_file:
        save_results(metrics, bench_args, server_args)

    return metrics
```
**EN:** This block defines function `throughput_test`. Main throughput benchmark function. Key calls include `configure_logger`, `logger.info`, `initialize_engine`, `min`, and `time.perf_counter`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `server_args`, and `bench_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `throughput_test`。 它用于处理 throughput test 相关逻辑。 关键调用包括 `configure_logger`、`logger.info`、`initialize_engine`、`min` 和 `time.perf_counter`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `server_args` 和 `bench_args` 等参数驱动。

### Lines 399-437: `display_results` implementation / `display_results` 实现
```python
def display_results(
    metrics: Dict[str, Any],
    bench_args: BenchArgs,
    model_path: str,
):
    """Display benchmark results in console."""
    print(
        "\n{s:{c}^{n}}".format(s=" Offline Throughput Benchmark Result ", n=110, c="=")
    )
    print_value_formatted("Model:", model_path)
    print_value_formatted("Dataset:", bench_args.dataset)
    print_value_formatted(
        "Resolution:",
        f"{bench_args.width}x{bench_args.height}x{bench_args.num_frames}",
    )
    print_value_formatted("Num Inference Steps:", bench_args.num_inference_steps)
    print_divider(75)
    print_value_formatted("Total Requests:", metrics["num_requests"])
    print_value_formatted("Successful Requests:", metrics["successful_requests"])
    print_value_formatted("Failed Requests:", metrics["failed_requests"])
    print_value_formatted(
        "Total Duration (seconds):", metrics["total_duration_seconds"]
    )
    print_divider(75)
    print_value_formatted("Frames Generated:", metrics["total_frames_generated"])
    print_value_formatted(
        "Megapixels Generated:", metrics["total_pixels_generated"] / 1e6
    )
    print_divider(75)
    print_value_formatted(
        "Frame Throughput (frames/sec):", metrics["frames_per_second"]
    )
    print_value_formatted("MP Throughput (MP/sec):", metrics["megapixels_per_second"])
    print_value_formatted("Requests Per Second:", metrics["requests_per_second"])
    print_value_formatted(
        "Latency Per Request (sec):", metrics["latency_per_request_seconds"]
    )
    print_value_formatted("Peak Memory (MB):", metrics["peak_memory_mb"])
    print_divider(110, "=")
```
**EN:** This block defines function `display_results`. Display benchmark results in console. Key calls include `print`, `print_value_formatted`, `print_divider`, and `format`. Parameters such as `metrics`, `bench_args`, and `model_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `display_results`。 它用于处理 display results 相关逻辑。 关键调用包括 `print`、`print_value_formatted`、`print_divider` 和 `format`。 本段逻辑主要由 `metrics`、`bench_args` 和 `model_path` 等参数驱动。

### Lines 440-468: `save_results` implementation / `save_results` 实现
```python
def save_results(
    metrics: Dict[str, Any],
    bench_args: BenchArgs,
    server_args: ServerArgs,
):
    """Save benchmark results to JSON file."""
    result = {
        "metadata": {
            "timestamp": time.strftime("%Y-%m-%dT%H:%M:%S"),
            "model_path": server_args.model_path,
            "task_type": bench_args.task_name,
            "backend": "engine",
        },
        "configuration": {
            "num_inference_steps": bench_args.num_inference_steps,
            "guidance_scale": bench_args.guidance_scale,
            "seed": bench_args.seed,
            "batch_size": bench_args.batch_size,
            "num_prompts": bench_args.num_prompts,
            "resolution": f"{bench_args.width}x{bench_args.height}x{bench_args.num_frames}",
            "dataset": bench_args.dataset,
        },
        "results": metrics,
    }

    with open(bench_args.output_file, "a") as f:
        f.write(json.dumps(result) + "\n")

    logger.info(f"Results saved to {bench_args.output_file}")
```
**EN:** This block defines function `save_results`. Save benchmark results to JSON file. Key calls include `logger.info`, `open`, `f.write`, `time.strftime`, and `json.dumps`. The implementation uses context-managed resources. Parameters such as `metrics`, `bench_args`, and `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `save_results`。 它用于保存results。 关键调用包括 `logger.info`、`open`、`f.write`、`time.strftime` 和 `json.dumps`。 实现中使用上下文管理资源。 本段逻辑主要由 `metrics`、`bench_args` 和 `server_args` 等参数驱动。

### Lines 471-489: `main` implementation / `main` 实现
```python
def main():
    """Main entry point."""
    parser = argparse.ArgumentParser(
        description="Offline throughput benchmark for multimodal generation models"
    )

    ServerArgs.add_cli_args(parser)
    BenchArgs.add_cli_args(parser)

    args, unknown_args = parser.parse_known_args()

    server_args = ServerArgs.from_cli_args(args, unknown_args)
    bench_args = BenchArgs.from_cli_args(args)

    set_global_server_args(server_args)

    result = throughput_test(server_args, bench_args)

    return result
```
**EN:** This block defines function `main`. Main entry point. Key calls include `argparse.ArgumentParser`, `ServerArgs.add_cli_args`, `BenchArgs.add_cli_args`, `parser.parse_known_args`, and `ServerArgs.from_cli_args`.
**CN:** 该代码块定义了函数 `main`。 它用于处理 main 相关逻辑。 关键调用包括 `argparse.ArgumentParser`、`ServerArgs.add_cli_args`、`BenchArgs.add_cli_args`、`parser.parse_known_args` 和 `ServerArgs.from_cli_args`。

### Lines 492-493: supporting statements / 辅助语句
```python
if __name__ == "__main__":
    main()
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `main`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `main` 协同工作。

## Key Concepts / 关键概念
- `BatchOutput`: Container for batch generation results. / 核心类，用于封装 batch output 相关行为。
- `BenchArgs`: Benchmark configuration for multimodal generation. / 核心类，用于封装 bench args 相关行为。
- `initialize_engine`: Initialize diffusion pipeline engine. / 顶层函数，用于初始化engine。
- `generate_batch`: Generate batch of images/videos synchronously. / 顶层函数，用于生成batch。
- `calculate_metrics`: Calculate generation-specific throughput metrics. / 顶层函数，用于处理 calculate metrics 相关逻辑。
- `throughput_test`: Main throughput benchmark function. / 顶层函数，用于处理 throughput test 相关逻辑。
- `display_results`: Display benchmark results in console. / 顶层函数，用于处理 display results 相关逻辑。
- `save_results`: Save benchmark results to JSON file. / 顶层函数，用于保存results。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `dataclasses`, `json`, `time`, `typing`
- **Third-party / 第三方依赖**: `torch`, `tqdm`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.benchmarks.datasets`, `sglang.multimodal_gen.runtime.entrypoints.diffusion_generator`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.test.test_utils`

- **Total lines / 总行数**: 493
