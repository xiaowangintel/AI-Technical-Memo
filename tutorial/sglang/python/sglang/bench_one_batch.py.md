# bench_one_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/bench_one_batch.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Benchmark the latency of running a single static batch without a server. / 该文件围绕上述模块职责展开实现，组织核心流程、辅助逻辑以及与相邻组件的协作。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48: Module-level supporting statements
```python
"""
Benchmark the latency of running a single static batch without a server.

This script does not launch a server and uses the low-level APIs.
It accepts server arguments (the same as launch_server.py) and benchmark arguments (e.g., batch size, input lengths).

# Usage (latency test)
## with dummy weights:
python -m sglang.bench_one_batch --model-path meta-llama/Meta-Llama-3-8B-Instruct --load-format dummy
## sweep through multiple data points and store (append) the results in a jsonl file:
python -m sglang.bench_one_batch --model-path meta-llama/Meta-Llama-3-8B-Instruct --batch-size 1 12 14 --input-len 256 512 --output-len 32 256 --run-name test_run
## run with profiling:
python -m sglang.bench_one_batch --model-path meta-llama/Meta-Llama-3-8B-Instruct --batch-size 1 12 14 --input-len 256 512 --profile
## run with profiling to custom directory:
export SGLANG_TORCH_PROFILER_DIR=/root/sglang/profile_log
python -m sglang.bench_one_batch --model-path meta-llama/Meta-Llama-3-8B-Instruct --batch-size 1 --input-len 256 --profile
## run with CUDA profiler (nsys):
nsys profile --force-overwrite=true -o bench_one_batch python -m sglang.bench_one_batch --model-path meta-llama/Meta-Llama-3-8B-Instruct --batch-size 1 --input-len 256 --profile --profile-activities CUDA_PROFILER
# Usage (correctness test):
python -m sglang.bench_one_batch --model-path TinyLlama/TinyLlama-1.1B-Chat-v0.4 --correct

## Reference output (of the correctness test above, can be gpu dependent):
input_ids=[[1, 450, 7483, 310, 3444, 338], [1, 450, 7483, 310, 278, 3303, 13187, 290, 338], [1, 20628, 338, 263, 6575, 1460, 2462, 322, 306, 763]]

prefill logits (first half): tensor([[-10.0312,  -9.5000,   0.8931,  ...,  -4.9414,  -3.2422,  -3.3633],
        [-10.0312,  -9.5000,   0.8931,  ...,  -4.9414,  -3.2422,  -3.3633],
        [ -9.1875, -10.2500,   2.7129,  ...,  -4.3359,  -4.0664,  -4.1328]],
       device='cuda:0')

prefill logits (final): tensor([[-8.3125, -7.1172,  3.3457,  ..., -4.9570, -4.1328, -3.4141],
        [-8.9141, -9.0156,  4.1445,  ..., -4.9922, -4.4961, -4.0781],
        [-9.6328, -9.0547,  4.0195,  ..., -5.3047, -4.7148, -4.4570]],
       device='cuda:0')

========== Prompt 0 ==========
<s> The capital of France is Paris.
The capital of the United States is Washington, D.C.


========== Prompt 1 ==========
<s> The capital of the United Kindom is London.
The capital of the United Kingdom is London.
The capital of the

========== Prompt 2 ==========
<s> Today is a sunny day and I like to go for a walk in the park.
I'm going to the park
"""
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 49-49: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 50-50: Module-level supporting statements
```python
import argparse
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 51-51: Module-level supporting statements
```python
import copy
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 52-52: Module-level supporting statements
```python
import dataclasses
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 53-53: Module-level supporting statements
```python
import itertools
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 54-54: Module-level supporting statements
```python
import json
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 55-55: Module-level supporting statements
```python
import logging
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 56-56: Module-level supporting statements
```python
import multiprocessing
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 57-57: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 58-58: Module-level supporting statements
```python
import time
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 59-59: Module-level supporting statements
```python
from types import SimpleNamespace
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 60-60: Module-level supporting statements
```python
from typing import Optional, Tuple
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 61-61: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 62-62: Module-level supporting statements
```python
import numpy as np
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 63-63: Module-level supporting statements
```python
import torch
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 64-64: Module-level supporting statements
```python
import torch.distributed as dist
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 65-65: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 66-66: Module-level supporting statements
```python
from sglang.srt.configs.model_config import ModelConfig
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 67-67: Module-level supporting statements
```python
from sglang.srt.distributed.parallel_state import destroy_distributed_environment
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 68-68: Module-level supporting statements
```python
from sglang.srt.entrypoints.engine import _set_envs_and_config
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 69-69: Module-level supporting statements
```python
from sglang.srt.layers.dp_attention import get_attention_tp_size
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 70-70: Module-level supporting statements
```python
from sglang.srt.layers.moe import initialize_moe_config
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 71-71: Module-level supporting statements
```python
from sglang.srt.layers.quantization.fp4_utils import initialize_fp4_gemm_config
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 72-72: Module-level supporting statements
```python
from sglang.srt.layers.quantization.fp8_utils import initialize_fp8_gemm_config
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 73-73: Module-level supporting statements
```python
from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 74-74: Module-level supporting statements
```python
from sglang.srt.managers.scheduler_components.dp_attn import prepare_mlp_sync_batch_raw
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 75-75: Module-level supporting statements
```python
from sglang.srt.mem_cache.base_prefix_cache import EvictParams
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 76-76: Module-level supporting statements
```python
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 77-77: Module-level supporting statements
```python
from sglang.srt.model_executor.model_runner import ModelRunner
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 78-78: Module-level supporting statements
```python
from sglang.srt.sampling.sampling_params import SamplingParams
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 79-79: Module-level supporting statements
```python
from sglang.srt.server_args import PortArgs, ServerArgs
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 80-80: Module-level supporting statements
```python
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 81-90: Module-level supporting statements
```python
from sglang.srt.utils import (
    configure_logger,
    get_bool_env_var,
    kill_process_tree,
    maybe_reindex_device_id,
    require_mlp_sync,
    require_mlp_tp_gather,
    set_gpu_proc_affinity,
    suppress_other_loggers,
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 91-91: Module-level supporting statements
```python
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 92-92: Module-level supporting statements
```python
from sglang.srt.utils.tensor_bridge import use_mlx
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 93-94: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 95-137: start profile function
```python
def start_profile(
    profile_activities,
    profile_record_shapes=False,
    rank_print=print,
    trace_filename=None,
):
    """
    Abstracted function to start profiling based on profile_activities.
    Returns profiler object (or None).
    """
    if use_mlx():
        import mlx.core as mx

        if trace_filename:
            mlx_trace_filename = trace_filename.replace(".trace.json.gz", ".gputrace")
            mx.metal.start_capture(mlx_trace_filename)
            rank_print(f"MLX Metal capture started directly to {mlx_trace_filename}")
        return "mlx"

    if "CUDA_PROFILER" in profile_activities:
        try:
            torch.cuda.cudart().cudaProfilerStart()
            rank_print("CUDA Profiler started (nsys will begin capturing)")
        except Exception as e:
            rank_print(f"Failed to start CUDA profiler: {e}")
        return None
    else:
        activities = []
        if "CPU" in profile_activities:
            activities.append(torch.profiler.ProfilerActivity.CPU)
        if "GPU" in profile_activities:
            activities.append(torch.profiler.ProfilerActivity.CUDA)
        if "XPU" in profile_activities:
            activities.append(torch.profiler.ProfilerActivity.XPU)
        if activities:
            profiler = torch.profiler.profile(
                activities=activities,
                with_stack=True,
                record_shapes=profile_record_shapes,
            )
            profiler.start()
            return profiler
        return None
```
**EN:** This block uses `start_profile` to collect profiling data. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `start_profile` 来收集剖析数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 138-139: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 140-185: stop profile function
```python
def stop_profile(
    profiler,
    profile_activities,
    rank_print=print,
    save_trace=False,
    trace_filename=None,
    stage=None,
):
    """
    Abstracted function to stop profiling based on profile_activities.
    Optionally saves trace results and prints completion messages.
    """
    if profiler == "mlx":
        import mlx.core as mx

        mx.metal.stop_capture()

        if save_trace and trace_filename:
            # Change SGLang's default torch extension to Apple's .gputrace extension
            mlx_trace_filename = trace_filename.replace(".trace.json.gz", ".gputrace")

            stage_desc = f"for {stage}" if stage else ""
            rank_print(f"MLX Metal gputrace {stage_desc} saved to {mlx_trace_filename}")
        return

    if "CUDA_PROFILER" in profile_activities:
        try:
            torch.cuda.cudart().cudaProfilerStop()
            rank_print("CUDA Profiler stopped (nsys should dump traces)")
        except Exception as e:
            rank_print(f"Failed to stop CUDA profiler: {e}")
    elif profiler is not None:
        profiler.stop()

    if save_trace:
        if profiler is not None:
            if trace_filename:
                _save_profile_trace_results(
                    profiler, profile_activities, trace_filename
                )
                stage_desc = f"for {stage}" if stage else ""
                rank_print(
                    f"torch profiler chrome trace {stage_desc} saved to {trace_filename}"
                )
        if "CUDA_PROFILER" in profile_activities:
            rank_print(f"CUDA profiler trace for {stage} completed")
```
**EN:** This block uses `stop_profile` to collect profiling data. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `stop_profile` 来收集剖析数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 186-188: Module-level supporting statements
```python


@dataclasses.dataclass
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 189-189: BenchArgs class declaration
```python
class BenchArgs:
```
**EN:** This block declares the `BenchArgs` class, which exists to benchmark a specific workload. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `BenchArgs` 类，其职责是对特定负载进行基准测试。它定义了本文件其余部分使用的结构与成员布局。

### Lines 190-190: Class-level supporting statements
```python
    run_name: str = "default"
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 191-191: Class-level supporting statements
```python
    batch_size: Tuple[int] = (1,)
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 192-192: Class-level supporting statements
```python
    input_len: Tuple[int] = (1024,)
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 193-193: Class-level supporting statements
```python
    output_len: Tuple[int] = (16,)
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 194-194: Class-level supporting statements
```python
    prompt_filename: str = ""
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 195-195: Class-level supporting statements
```python
    result_filename: str = "result.jsonl"
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 196-196: Class-level supporting statements
```python
    correctness_test: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 197-197: Class-level supporting statements
```python
    # This is only used for correctness test
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 198-198: Class-level supporting statements
```python
    cut_len: int = 4
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 199-199: Class-level supporting statements
```python
    log_decode_step: int = 0
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 200-200: Class-level supporting statements
```python
    profile: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 201-201: Class-level supporting statements
```python
    profile_record_shapes: bool = False
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 202-202: Class-level supporting statements
```python
    profile_activities: Tuple[str] = ("CPU", "GPU")
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 203-203: Class-level supporting statements
```python
    profile_stage: str = "all"
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 204-204: Class-level supporting statements
```python
    profile_filename_prefix: str = "profile"
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 205-205: Class-level supporting statements
```python
    profile_start_step: Optional[int] = None
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 206-206: Class-level supporting statements
```python
    profile_steps: Optional[int] = None
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 207-208: Class-level supporting statements
```python

    @staticmethod
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 209-268: BenchArgs.add_cli_args method (part 1/2)
```python
    def add_cli_args(parser: argparse.ArgumentParser):
        parser.add_argument("--run-name", type=str, default=BenchArgs.run_name)
        parser.add_argument(
            "--batch-size", type=int, nargs="+", default=BenchArgs.batch_size
        )
        parser.add_argument(
            "--input-len", type=int, nargs="+", default=BenchArgs.input_len
        )
        parser.add_argument(
            "--output-len", type=int, nargs="+", default=BenchArgs.output_len
        )
        parser.add_argument(
            "--prompt-filename", type=str, default=BenchArgs.prompt_filename
        )
        parser.add_argument(
            "--result-filename", type=str, default=BenchArgs.result_filename
        )
        parser.add_argument("--correctness-test", action="store_true")
        parser.add_argument("--cut-len", type=int, default=BenchArgs.cut_len)
        parser.add_argument(
            "--log-decode-step",
            type=int,
            default=BenchArgs.log_decode_step,
            help="Log decode latency by step, default is set to zero to disable.",
        )
        parser.add_argument("--profile", action="store_true", help="Enable profiling.")
        parser.add_argument(
            "--profile-record-shapes",
            action="store_true",
            help="Record tensor shapes in profiling results.",
        )
        parser.add_argument(
            "--profile-activities",
            type=str,
            nargs="+",
            default=["CPU", "GPU"],
            choices=["CPU", "GPU", "CUDA_PROFILER", "XPU"],
            help="Profiler activities: CPU, GPU, XPU, CUDA_PROFILER. If CPU/GPU/XPU, use torch profiler. If CUDA_PROFILER, use CUDA profiler.",
        )
        parser.add_argument(
            "--profile-stage",
            type=str,
            default=BenchArgs.profile_stage,
            choices=["all", "prefill", "decode"],
            help="Which stage to profile: all, prefill, or decode only.",
        )
        parser.add_argument(
            "--profile-filename-prefix",
            type=str,
            default=BenchArgs.profile_filename_prefix,
            help="Prefix of the profiling file names. The full profiling result file(s) be "
            '"[profile_filename_prefix]_batch[batch_size]_input[input_len]_output[output_len].trace.json.gz"',
        )
        parser.add_argument(
            "--profile-start-step",
            type=int,
            default=None,
            help="Decode step at which to start profiling (0-indexed). If not specified, defaults to output_len // 2.",
        )
        parser.add_argument(
```
**EN:** This block uses `BenchArgs.add_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `BenchArgs.add_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 269-273: BenchArgs.add_cli_args method (part 2/2)
```python
            "--profile-steps",
            type=int,
            default=None,
            help="Number of decode steps to profile starting from profile-start-step. If not specified, profiles only one step.",
        )
```
**EN:** This block uses `BenchArgs.add_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `BenchArgs.add_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 274-275: Class-level supporting statements
```python

    @classmethod
```
**EN:** This block contains supporting statements for the `BenchArgs` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BenchArgs` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 276-287: BenchArgs.from_cli_args method
```python
    def from_cli_args(cls, args: argparse.Namespace):
        # use the default value's type to cast the args into correct types.
        attrs = [(attr.name, type(attr.default)) for attr in dataclasses.fields(cls)]
        result = {}
        for attr, attr_type in attrs:
            value = getattr(args, attr)
            # Handle None values - don't try to cast them
            if value is None or attr_type == type(None):
                result[attr] = value
            else:
                result[attr] = attr_type(value)
        return cls(**result)
```
**EN:** This block uses `BenchArgs.from_cli_args` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BenchArgs.from_cli_args` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 288-289: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 290-333: load model function
```python
def load_model(server_args, port_args, gpu_id, tp_rank):
    suppress_other_loggers()
    rank_print = print if tp_rank == 0 else lambda *args, **kwargs: None
    moe_ep_rank = tp_rank // (server_args.tp_size // server_args.ep_size)

    model_config = ModelConfig.from_server_args(server_args)
    runner_kwargs = dict(
        model_config=model_config,
        mem_fraction_static=server_args.mem_fraction_static,
        gpu_id=gpu_id,
        tp_rank=tp_rank,
        tp_size=server_args.tp_size,
        moe_ep_rank=moe_ep_rank,
        moe_ep_size=server_args.ep_size,
        pp_rank=0,
        pp_size=1,
        nccl_port=port_args.nccl_port,
        server_args=server_args,
    )

    _use_mlx = use_mlx()
    if _use_mlx:
        from sglang.srt.hardware_backend.mlx.model_runner_stub import (
            MlxModelRunnerStub,
        )

        model_runner = MlxModelRunnerStub(**runner_kwargs)
    else:
        model_runner = ModelRunner(**runner_kwargs)
    rank_print(f"max_total_num_tokens={model_runner.max_total_num_tokens}")
    tokenizer = get_tokenizer(
        server_args.tokenizer_path,
        tokenizer_mode=server_args.tokenizer_mode,
        trust_remote_code=server_args.trust_remote_code,
    )
    if server_args.tp_size > 1:
        dist.barrier()

    if _use_mlx:
        model_runner = _MlxBenchRunner(model_runner, server_args)
    else:
        model_runner = _TorchBenchRunner(model_runner)

    return model_runner, tokenizer
```
**EN:** This block uses `load_model` to load resources or configuration. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `load_model` 来加载资源或配置。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 334-335: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 336-378: prepare inputs for correctness test function
```python
def prepare_inputs_for_correctness_test(bench_args, tokenizer, custom_prompts):
    if custom_prompts:
        custom_input_len = len(custom_prompts)
        bs = bench_args.batch_size[0]
        if custom_input_len > bs:
            logging.warning(
                f"Custom input size ({custom_input_len}) is larger than batch_size ({bs}). "
                f"Using the first {bs} prompts."
            )
            custom_prompts = custom_prompts[:bs]

    prompts = (
        custom_prompts
        if custom_prompts
        else [
            "The capital of France is",
            "The capital of the United Kindom is",
            "Today is a sunny day and I like",
        ]
    )
    input_ids = [tokenizer.encode(p) for p in prompts]
    sampling_params = SamplingParams(
        temperature=0,
        max_new_tokens=BenchArgs.output_len,
    )

    reqs = []
    for i in range(len(prompts)):
        assert len(input_ids[i]) > bench_args.cut_len

        tmp_input_ids = input_ids[i][: bench_args.cut_len]
        req = Req(
            rid=i,
            origin_input_text=prompts[i],
            origin_input_ids=tmp_input_ids,
            sampling_params=sampling_params,
        )
        req.fill_ids = req.origin_input_ids
        req.logprob_start_len = -1
        req.set_extend_input_len(len(req.fill_ids) - len(req.prefix_indices))
        reqs.append(req)

    return input_ids, reqs
```
**EN:** This block uses `prepare_inputs_for_correctness_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `prepare_inputs_for_correctness_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 379-380: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 381-393: prepare extend inputs for correctness test function
```python
def prepare_extend_inputs_for_correctness_test(
    bench_args, input_ids, reqs, model_runner
):
    for i in range(len(reqs)):
        req: Req = reqs[i]
        req.fill_ids += input_ids[i][bench_args.cut_len :]
        if model_runner is not None:
            req.prefix_indices = model_runner.req_to_token_pool.req_to_token[
                i, : bench_args.cut_len
            ].to(req.prefix_indices.dtype)
            req.logprob_start_len = -1
            req.set_extend_input_len(len(req.fill_ids) - len(req.prefix_indices))
    return reqs
```
**EN:** This block uses `prepare_extend_inputs_for_correctness_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `prepare_extend_inputs_for_correctness_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 394-395: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 396-422: prepare synthetic inputs for latency test function
```python
def prepare_synthetic_inputs_for_latency_test(
    batch_size, input_len, custom_inputs=None
):
    input_ids = (
        custom_inputs
        if custom_inputs
        else np.random.randint(0, 10000, (batch_size, input_len), dtype=np.int32)
    )
    sampling_params = SamplingParams(
        temperature=0,
        max_new_tokens=BenchArgs.output_len,
    )

    reqs = []
    for i in range(len(input_ids)):
        req = Req(
            rid=i,
            origin_input_text="",
            origin_input_ids=list(input_ids[i]),
            sampling_params=sampling_params,
        )
        req.fill_ids = req.origin_input_ids
        req.logprob_start_len = -1
        req.set_extend_input_len(len(req.fill_ids) - len(req.prefix_indices))
        reqs.append(req)

    return reqs
```
**EN:** This block uses `prepare_synthetic_inputs_for_latency_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `prepare_synthetic_inputs_for_latency_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 423-424: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 425-425: TreeCacheNamespace class declaration
```python
class TreeCacheNamespace(SimpleNamespace):
```
**EN:** This block declares the `TreeCacheNamespace` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `TreeCacheNamespace` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 426-427: TreeCacheNamespace.supports_swa method
```python
    def supports_swa(self) -> bool:
        return False
```
**EN:** This block uses `TreeCacheNamespace.supports_swa` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TreeCacheNamespace.supports_swa` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 428-428: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `TreeCacheNamespace` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`TreeCacheNamespace` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 429-430: TreeCacheNamespace.supports_mamba method
```python
    def supports_mamba(self) -> bool:
        return False
```
**EN:** This block uses `TreeCacheNamespace.supports_mamba` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TreeCacheNamespace.supports_mamba` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 431-431: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `TreeCacheNamespace` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`TreeCacheNamespace` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 432-433: TreeCacheNamespace.is_chunk_cache method
```python
    def is_chunk_cache(self) -> bool:
        return False
```
**EN:** This block uses `TreeCacheNamespace.is_chunk_cache` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TreeCacheNamespace.is_chunk_cache` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 434-434: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `TreeCacheNamespace` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`TreeCacheNamespace` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 435-436: TreeCacheNamespace.is_tree_cache method
```python
    def is_tree_cache(self) -> bool:
        return not self.is_chunk_cache()
```
**EN:** This block uses `TreeCacheNamespace.is_tree_cache` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TreeCacheNamespace.is_tree_cache` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 437-437: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `TreeCacheNamespace` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`TreeCacheNamespace` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 438-439: TreeCacheNamespace.evict method
```python
    def evict(self, params: EvictParams):
        pass
```
**EN:** This block uses `TreeCacheNamespace.evict` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `TreeCacheNamespace.evict` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 440-442: Module-level supporting statements
```python


@torch.no_grad
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 443-465: extend function
```python
def extend(reqs, model_runner):
    # Create dummy tree_cache for benchmarks (no prefix caching, just allocation)
    dummy_tree_cache = TreeCacheNamespace(
        page_size=model_runner.server_args.page_size,
        device=model_runner.device,
        token_to_kv_pool_allocator=model_runner.token_to_kv_pool_allocator,
    )

    batch = ScheduleBatch.init_new(
        reqs=reqs,
        req_to_token_pool=model_runner.req_to_token_pool,
        token_to_kv_pool_allocator=model_runner.token_to_kv_pool_allocator,
        tree_cache=dummy_tree_cache,
        model_config=model_runner.model_config,
        enable_overlap=False,
        spec_algorithm=SpeculativeAlgorithm.NONE,
    )
    batch.prepare_for_extend()
    _maybe_prepare_mlp_sync_batch(batch, model_runner)
    forward_batch = ForwardBatch.init_new(batch, model_runner)
    logits_output = model_runner.forward(forward_batch).logits_output
    next_token_ids = model_runner.sample(logits_output, forward_batch)
    return next_token_ids, logits_output.next_token_logits, batch
```
**EN:** This block uses `extend` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `extend` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 466-468: Module-level supporting statements
```python


@torch.no_grad
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 469-476: decode function
```python
def decode(input_token_ids, batch, model_runner):
    batch.output_ids = input_token_ids
    batch.prepare_for_decode()
    _maybe_prepare_mlp_sync_batch(batch, model_runner)
    forward_batch = ForwardBatch.init_new(batch, model_runner)
    logits_output = model_runner.forward(forward_batch).logits_output
    next_token_ids = model_runner.sample(logits_output, forward_batch)
    return next_token_ids, logits_output.next_token_logits
```
**EN:** This block uses `decode` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `decode` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 477-478: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 479-492: maybe prepare mlp sync batch function
```python
def _maybe_prepare_mlp_sync_batch(batch: ScheduleBatch, model_runner):
    if require_mlp_sync(model_runner.server_args):
        prepare_mlp_sync_batch_raw(
            batch,
            dp_size=model_runner.server_args.dp_size,
            attn_tp_size=get_attention_tp_size(),
            attn_cp_size=model_runner.attn_cp_size,
            tp_group=model_runner.tp_group,
            get_idle_batch=None,
            disable_cuda_graph=model_runner.server_args.disable_cuda_graph,
            require_mlp_tp_gather=require_mlp_tp_gather(model_runner.server_args),
            disable_overlap_schedule=model_runner.server_args.disable_overlap_schedule,
            offload_tags=set(),
        )
```
**EN:** This block uses `_maybe_prepare_mlp_sync_batch` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_maybe_prepare_mlp_sync_batch` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 493-494: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 495-495: _TorchBenchRunner class declaration
```python
class _TorchBenchRunner:
```
**EN:** This block declares the `_TorchBenchRunner` class, which exists to benchmark a specific workload. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `_TorchBenchRunner` 类，其职责是对特定负载进行基准测试。它定义了本文件其余部分使用的结构与成员布局。

### Lines 496-496: Class-level supporting statements
```python
    """Wraps ModelRunner for the standard PyTorch benchmark path."""
```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 497-497: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 498-499: _TorchBenchRunner initializer
```python
    def __init__(self, model_runner):
        self.torch_runner = model_runner
```
**EN:** This block initializes the `_TorchBenchRunner` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `_TorchBenchRunner` 对象，连接后续方法使用的状态与依赖。

### Lines 500-500: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 501-503: _TorchBenchRunner.clear method
```python
    def clear(self):
        self.torch_runner.req_to_token_pool.clear()
        self.torch_runner.token_to_kv_pool_allocator.clear()
```
**EN:** This block uses `_TorchBenchRunner.clear` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_TorchBenchRunner.clear` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 504-504: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 505-506: _TorchBenchRunner.extend method
```python
    def extend(self, reqs):
        return extend(reqs, self.torch_runner)
```
**EN:** This block uses `_TorchBenchRunner.extend` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_TorchBenchRunner.extend` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 507-507: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 508-509: _TorchBenchRunner.decode method
```python
    def decode(self, next_token_ids, batch):
        return decode(next_token_ids, batch, self.torch_runner)
```
**EN:** This block uses `_TorchBenchRunner.decode` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_TorchBenchRunner.decode` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 510-510: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 511-512: _TorchBenchRunner.cleanup method
```python
    def cleanup(self, batch):
        pass
```
**EN:** This block uses `_TorchBenchRunner.cleanup` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_TorchBenchRunner.cleanup` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 513-513: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 514-515: _TorchBenchRunner.synchronize method
```python
    def synchronize(self):
        synchronize(self.torch_runner.device)
```
**EN:** This block uses `_TorchBenchRunner.synchronize` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_TorchBenchRunner.synchronize` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 516-516: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_TorchBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_TorchBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 517-518: _TorchBenchRunner.max_batch_size method
```python
    def max_batch_size(self, input_len, output_len):
        return self.torch_runner.max_total_num_tokens // (input_len + output_len)
```
**EN:** This block uses `_TorchBenchRunner.max_batch_size` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_TorchBenchRunner.max_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 519-520: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 521-521: _MlxBenchRunner class declaration
```python
class _MlxBenchRunner:
```
**EN:** This block declares the `_MlxBenchRunner` class, which exists to benchmark a specific workload. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `_MlxBenchRunner` 类，其职责是对特定负载进行基准测试。它定义了本文件其余部分使用的结构与成员布局。

### Lines 522-522: Class-level supporting statements
```python
    """Wraps MlxModelRunner for the MLX benchmark path."""
```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 523-523: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 524-540: _MlxBenchRunner initializer
```python
    def __init__(self, model_runner, server_args):
        from sglang.srt.hardware_backend.mlx.model_runner import MlxModelRunner

        # Radix cache requires the scheduler's allocator/trie; disable in
        # standalone bench mode where no scheduler is present.
        init_kwargs = dict(
            model_path=server_args.model_path,
            trust_remote_code=server_args.trust_remote_code,
            disable_radix_cache=True,
            mem_fraction_static=server_args.mem_fraction_static,
            quantization=server_args.quantization,
        )
        if server_args.max_total_tokens is not None:
            init_kwargs["pool_size"] = server_args.max_total_tokens
        self.mlx_runner = MlxModelRunner(**init_kwargs)
        self.mlx_runner.init_kv_pool(req_to_token_pool=None)
        self.fake_torch_runner = model_runner
```
**EN:** This block initializes the `_MlxBenchRunner` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `_MlxBenchRunner` 对象，连接后续方法使用的状态与依赖。

### Lines 541-541: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 542-543: _MlxBenchRunner.clear method
```python
    def clear(self):
        self.mlx_runner.clear()
```
**EN:** This block uses `_MlxBenchRunner.clear` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_MlxBenchRunner.clear` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 544-544: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 545-559: _MlxBenchRunner.extend method
```python
    def extend(self, reqs):
        req_ids = [str(req.rid) for req in reqs]
        results = []
        for rid, req in zip(req_ids, reqs):
            token_ids = [int(t) for t in req.fill_ids]
            next_token = self.mlx_runner.prefill(
                req_id=rid,
                new_token_ids=token_ids,
                full_token_ids=token_ids,
                prefix_slot_ids=[],
                new_slot_ids=[],
                req_pool_idx=0,
            )
            results.append(next_token)
        return torch.tensor(results), None, req_ids
```
**EN:** This block uses `_MlxBenchRunner.extend` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_MlxBenchRunner.extend` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 560-560: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 561-563: _MlxBenchRunner.decode method
```python
    def decode(self, next_token_ids, req_ids):
        next_token_ids = self.mlx_runner.decode_batch(req_ids)
        return torch.tensor(next_token_ids), None
```
**EN:** This block uses `_MlxBenchRunner.decode` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_MlxBenchRunner.decode` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 564-564: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 565-568: _MlxBenchRunner.cleanup method
```python
    def cleanup(self, batch):
        if isinstance(batch, list):
            for req_id in batch:
                self.mlx_runner.remove_request(req_id)
```
**EN:** This block uses `_MlxBenchRunner.cleanup` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_MlxBenchRunner.cleanup` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 569-569: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 570-571: _MlxBenchRunner.synchronize method
```python
    def synchronize(self):
        pass
```
**EN:** This block uses `_MlxBenchRunner.synchronize` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_MlxBenchRunner.synchronize` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 572-572: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `_MlxBenchRunner` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`_MlxBenchRunner` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 573-574: _MlxBenchRunner.max_batch_size method
```python
    def max_batch_size(self, input_len, output_len):
        return self.fake_torch_runner.max_total_num_tokens // (input_len + output_len)
```
**EN:** This block uses `_MlxBenchRunner.max_batch_size` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_MlxBenchRunner.max_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 575-576: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 577-587: read prompts from file function
```python
def _read_prompts_from_file(prompt_file, rank_print):
    """Read custom prompts from the file specified by `--prompt-filename`."""
    if not prompt_file:
        return []
    if not os.path.exists(prompt_file):
        rank_print(
            f"Custom prompt file {prompt_file} not found. Using default inputs..."
        )
        return []
    with open(prompt_file, "r") as pf:
        return pf.readlines()
```
**EN:** This block uses `_read_prompts_from_file` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_read_prompts_from_file` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 588-589: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 590-591: get torch profiler output dir function
```python
def _get_torch_profiler_output_dir():
    return os.environ.get("SGLANG_TORCH_PROFILER_DIR", "/tmp")
```
**EN:** This block uses `_get_torch_profiler_output_dir` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_get_torch_profiler_output_dir` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 592-593: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 594-599: create torch profiler filename function
```python
def _create_torch_profiler_filename(
    profile_filename_prefix, batch_size, input_len, output_len, stage
):
    output_dir = _get_torch_profiler_output_dir()
    filename = f"{profile_filename_prefix}_batch{batch_size}_input{input_len}_output{output_len}_{stage}.trace.json.gz"
    return os.path.join(output_dir, filename)
```
**EN:** This block uses `_create_torch_profiler_filename` to create runtime objects or artifacts. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_create_torch_profiler_filename` 来创建运行时对象或产物。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 600-601: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 602-612: save profile trace results function
```python
def _save_profile_trace_results(profiler, profile_activities, filename):
    parent_dir = os.path.dirname(os.path.abspath(filename))
    os.makedirs(parent_dir, exist_ok=True)
    profiler.export_chrome_trace(filename)
    if "GPU" in profile_activities:
        sort_by = "self_cuda_time_total"
    elif "XPU" in profile_activities:
        sort_by = "self_xpu_time_total"
    else:
        sort_by = "self_cpu_time_total"
    print(profiler.key_averages(group_by_input_shape=True).table(sort_by=sort_by))
```
**EN:** This block uses `_save_profile_trace_results` to persist generated data. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_save_profile_trace_results` 来持久化生成的数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 613-614: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 615-665: correctness test function
```python
def correctness_test(
    server_args,
    port_args,
    bench_args,
    gpu_id,
    tp_rank,
):
    # Configure the logger
    configure_logger(server_args, prefix=f" TP{tp_rank}")
    rank_print = print if tp_rank == 0 else lambda *args, **kwargs: None

    # Load the model
    model_runner, tokenizer = load_model(server_args, port_args, gpu_id, tp_rank)

    # Prepare inputs
    custom_prompts = _read_prompts_from_file(bench_args.prompt_filename, rank_print)
    input_ids, reqs = prepare_inputs_for_correctness_test(
        bench_args, tokenizer, custom_prompts
    )
    rank_print(f"\n{input_ids=}\n")

    if bench_args.cut_len > 0:
        # Prefill
        next_token_ids, next_token_logits, batch = model_runner.extend(reqs)
        rank_print(f"prefill logits (first half): {next_token_logits} \n")

        # Prepare extend inputs
        torch_runner = getattr(model_runner, "torch_runner", None)
        reqs = prepare_extend_inputs_for_correctness_test(
            bench_args, input_ids, reqs, torch_runner
        )

    # Extend (prefill w/ KV cache)
    next_token_ids, next_token_logits, batch = model_runner.extend(reqs)
    rank_print(f"prefill logits (final): {next_token_logits} \n")

    # Decode
    output_ids = [input_ids[i] + [next_token_ids[i]] for i in range(len(input_ids))]
    for _ in range(bench_args.output_len[0] - 1):
        next_token_ids, _ = model_runner.decode(next_token_ids, batch)
        next_token_ids_list = next_token_ids.tolist()
        for i in range(len(reqs)):
            output_ids[i].append(next_token_ids_list[i])

    # Clean up
    model_runner.cleanup(batch)

    # Print output texts
    for i in range(len(reqs)):
        rank_print(f"========== Prompt {i} ==========")
        rank_print(tokenizer.decode(output_ids[i]), "\n")
```
**EN:** This block uses `correctness_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `correctness_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 666-667: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 668-669: synchronize function
```python
def synchronize(device):
    torch.get_device_module(device).synchronize()
```
**EN:** This block uses `synchronize` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `synchronize` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 670-671: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 672-731: latency test run once function (part 1/3)
```python
def latency_test_run_once(
    run_name,
    model_runner,
    rank_print,
    reqs,
    batch_size,
    input_len,
    output_len,
    log_decode_step,
    profile,
    profile_record_shapes,
    profile_activities,
    profile_filename_prefix,
    profile_stage,
    tp_rank,
    profile_start_step=None,
    profile_steps=None,
):
    max_batch_size = model_runner.max_batch_size(input_len, output_len)
    if batch_size > max_batch_size:
        rank_print(
            f"skipping ({batch_size}, {input_len}, {output_len}) due to max batch size limit"
        )
        return

    model_runner.clear()

    measurement_results = {
        "run_name": run_name,
        "batch_size": batch_size,
        "input_len": input_len,
        "output_len": output_len,
    }

    tot_latency = 0

    profiler = None
    enable_profile_prefill = profile and profile_stage in ["all", "prefill"]
    trace_filename_prefill = None
    if enable_profile_prefill:
        trace_filename_prefill = _create_torch_profiler_filename(
            profile_filename_prefix, batch_size, input_len, output_len, "prefill"
        )
        profiler = start_profile(
            profile_activities,
            profile_record_shapes=profile_record_shapes,
            rank_print=rank_print,
            trace_filename=trace_filename_prefill,  # pass it in here for the MLX path only
        )

    model_runner.synchronize()
    tic = time.perf_counter()
    next_token_ids, _, batch = model_runner.extend(reqs)
    model_runner.synchronize()
    prefill_latency = time.perf_counter() - tic

    if enable_profile_prefill:
        stop_profile(
            profiler,
            profile_activities,
```
**EN:** This block uses `latency_test_run_once` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `latency_test_run_once` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 732-791: latency test run once function (part 2/3)
```python
            rank_print=rank_print,
            save_trace=True,
            trace_filename=trace_filename_prefill,
            stage="prefill",
        )

    tot_latency += prefill_latency
    throughput = input_len * batch_size / prefill_latency
    rank_print(
        f"Prefill. latency: {prefill_latency:6.5f} s, throughput: {throughput:9.2f} token/s"
    )
    measurement_results["prefill_latency"] = prefill_latency
    measurement_results["prefill_throughput"] = throughput

    decode_latencies = []
    # Determine profiling start step and end step
    profile_start = (
        profile_start_step if profile_start_step is not None else (output_len // 2)
    )
    profile_end = profile_start + (profile_steps if profile_steps is not None else 1)
    enable_profile_decode = profile and profile_stage in ["all", "decode"]
    trace_filename_decode = None
    profiler = None
    for i in range(output_len - 1):
        model_runner.synchronize()
        # Start profiler at the specified step
        if enable_profile_decode and i == profile_start:
            trace_filename_decode = _create_torch_profiler_filename(
                profile_filename_prefix, batch_size, input_len, output_len, "decode"
            )
            profiler = start_profile(
                profile_activities,
                profile_record_shapes=profile_record_shapes,
                rank_print=rank_print,
                trace_filename=trace_filename_decode,
            )

        tic = time.perf_counter()
        next_token_ids, _ = model_runner.decode(next_token_ids, batch)
        model_runner.synchronize()
        latency = time.perf_counter() - tic

        # Stop profiler after the specified number of steps
        if enable_profile_decode and profiler is not None and i >= profile_end - 1:
            stop_profile(
                profiler,
                profile_activities,
                rank_print=rank_print,
                save_trace=True,
                trace_filename=trace_filename_decode,
                stage="decode",
            )
            profiler = None

        tot_latency += latency
        throughput = batch_size / latency
        decode_latencies.append(latency)
        if i < 5 or (log_decode_step > 0 and i % log_decode_step == 0):
            rank_print(
                f"Decode {i}. Batch size: {batch_size}, latency: {latency:6.5f} s, throughput: {throughput:9.2f} token/s"
```
**EN:** This block uses `latency_test_run_once` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the middle-stage logic and data flow of the routine.
**CN:** 该代码块通过 `latency_test_run_once` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流。

### Lines 792-812: latency test run once function (part 3/3)
```python
            )

    # Record decode timing from 2nd output
    if output_len > 1:
        med_decode_latency = np.median(decode_latencies)
        med_decode_throughput = batch_size / med_decode_latency
        rank_print(
            f"Decode.  median latency: {med_decode_latency:6.5f} s, median throughput: {med_decode_throughput:9.2f} token/s"
        )
        measurement_results["median_decode_latency"] = med_decode_latency
        measurement_results["median_decode_throughput"] = med_decode_throughput

    throughput = (input_len + output_len) * batch_size / tot_latency
    rank_print(
        f"Total. latency: {tot_latency:6.3f} s, throughput: {throughput:9.2f} token/s"
    )
    measurement_results["total_latency"] = tot_latency
    measurement_results["overall_throughput"] = throughput

    model_runner.cleanup(batch)
    return measurement_results
```
**EN:** This block uses `latency_test_run_once` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `latency_test_run_once` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 813-814: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 815-874: latency test function (part 1/2)
```python
def latency_test(
    server_args,
    port_args,
    bench_args,
    gpu_id,
    tp_rank,
):
    initialize_moe_config(server_args)
    initialize_fp8_gemm_config(server_args)
    initialize_fp4_gemm_config(server_args)

    # Set CPU affinity
    if get_bool_env_var("SGLANG_SET_CPU_AFFINITY"):
        set_gpu_proc_affinity(
            server_args.pp_size, server_args.tp_size, server_args.nnodes, tp_rank
        )

    # Configure the logger
    configure_logger(server_args, prefix=f" TP{tp_rank}")
    rank_print = print if tp_rank == 0 else lambda *args, **kwargs: None

    # Load the model
    model_runner, tokenizer = load_model(server_args, port_args, gpu_id, tp_rank)

    # Prepare inputs for warm up
    reqs = prepare_synthetic_inputs_for_latency_test(
        bench_args.batch_size[0], bench_args.input_len[0]
    )

    # Warm up
    rank_print("Warmup ...")
    latency_test_run_once(
        bench_args.run_name,
        model_runner,
        rank_print,
        reqs,
        bench_args.batch_size[0],
        bench_args.input_len[0],
        min(32, bench_args.output_len[0]),  # shorter decoding to speed up the warmup
        log_decode_step=0,
        profile=False,
        profile_record_shapes=False,
        profile_activities=("CPU", "GPU"),
        profile_filename_prefix="",
        profile_stage="all",
        tp_rank=tp_rank,
        profile_start_step=None,
        profile_steps=None,
    )

    rank_print("Benchmark ...")

    custom_inputs = _read_prompts_from_file(bench_args.prompt_filename, rank_print)
    custom_inputs = [tokenizer.encode(p.strip()) for p in custom_inputs]
    custom_input_len = len(custom_inputs)

    # Run the sweep
    result_list = []
    for bs, il, ol in itertools.product(
        bench_args.batch_size, bench_args.input_len, bench_args.output_len
```
**EN:** This block uses `latency_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `latency_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 875-925: latency test function (part 2/2)
```python
    ):
        bs_aligned_inputs = []
        if custom_inputs:
            if custom_input_len == bs:
                bs_aligned_inputs = custom_inputs
            elif custom_input_len > bs:
                rank_print(
                    f"Custom input size ({custom_input_len}) is larger than batch_size ({bs}). "
                    f"Using the first {bs} prompts."
                )
                bs_aligned_inputs = copy.deepcopy(custom_inputs[:bs])
            else:
                rank_print(
                    f"Custom input size ({custom_input_len}) is smaller than batch_size ({bs}). "
                    f"Pad to the desired batch_size with the last prompt."
                )
                bs_aligned_inputs = copy.deepcopy(custom_inputs)
                bs_aligned_inputs.extend(
                    [bs_aligned_inputs[-1]] * (bs - custom_input_len)
                )

        reqs = prepare_synthetic_inputs_for_latency_test(bs, il, bs_aligned_inputs)
        ret = latency_test_run_once(
            bench_args.run_name,
            model_runner,
            rank_print,
            reqs,
            bs,
            il,
            ol,
            bench_args.log_decode_step,
            bench_args.profile if tp_rank == 0 else None,
            bench_args.profile_record_shapes if tp_rank == 0 else None,
            bench_args.profile_activities,
            bench_args.profile_filename_prefix,
            bench_args.profile_stage,
            tp_rank,
            bench_args.profile_start_step,
            bench_args.profile_steps,
        )
        if ret is not None:
            result_list.append(ret)

    # Write results in jsonlines format on rank 0.
    if tp_rank == 0 and bench_args.result_filename:
        with open(bench_args.result_filename, "a") as fout:
            for result in result_list:
                fout.write(json.dumps(result) + "\n")

    if server_args.tp_size > 1:
        destroy_distributed_environment()
```
**EN:** This block uses `latency_test` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `latency_test` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 926-927: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 928-968: main function
```python
def main(server_args, bench_args):
    server_args.cuda_graph_max_bs = max(bench_args.batch_size)

    _set_envs_and_config(server_args)

    if server_args.model_path:
        if bench_args.correctness_test:
            work_func = correctness_test
        else:
            work_func = latency_test
    else:
        raise ValueError(
            "Provide --model-path for running the tests or "
            "provide --result-filename for plotting the results"
        )

    port_args = PortArgs.init_new(server_args)

    if server_args.tp_size == 1:
        work_func(server_args, port_args, bench_args, 0, 0)
    else:
        workers = []
        for tp_rank in range(server_args.tp_size):
            with maybe_reindex_device_id(tp_rank) as gpu_id:
                proc = multiprocessing.Process(
                    target=work_func,
                    args=(
                        server_args,
                        port_args,
                        bench_args,
                        gpu_id,
                        tp_rank,
                    ),
                )
                proc.start()
                workers.append(proc)

        for proc in workers:
            proc.join()

        proc.terminate()
```
**EN:** This block uses `main` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `main` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 969-970: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 971-988: Module-level supporting statements
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    ServerArgs.add_cli_args(parser)
    BenchArgs.add_cli_args(parser)
    args = parser.parse_args()
    server_args = ServerArgs.from_cli_args(args)
    bench_args = BenchArgs.from_cli_args(args)

    logging.basicConfig(
        level=getattr(logging, server_args.log_level.upper()),
        format="%(message)s",
    )

    try:
        main(server_args, bench_args)
    finally:
        if server_args.tp_size != 1:
            kill_process_tree(os.getpid(), include_parent=False)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- Performance measurement / 性能测量
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Numerical data processing / 数值数据处理
- Runtime coordination / 运行时协同

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.configs.model_config`
- `sglang.srt.distributed.parallel_state`
- `sglang.srt.entrypoints.engine`
- `sglang.srt.hardware_backend.mlx.model_runner`
- `sglang.srt.hardware_backend.mlx.model_runner_stub`
- `sglang.srt.layers.dp_attention`
- `sglang.srt.layers.moe`
- `sglang.srt.layers.quantization.fp4_utils`
- `sglang.srt.layers.quantization.fp8_utils`
- `sglang.srt.managers.schedule_batch`
- `sglang.srt.managers.scheduler_components.dp_attn`
- `sglang.srt.mem_cache.base_prefix_cache`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.model_executor.model_runner`
- `sglang.srt.sampling.sampling_params`
- `sglang.srt.server_args`
- `sglang.srt.speculative.spec_info`
- `sglang.srt.utils`
- `sglang.srt.utils.hf_transformers_utils`
- `sglang.srt.utils.tensor_bridge`
### External / 外部
- `mlx`
- `multiprocessing`
- `numpy`
- `torch`
- `argparse` (stdlib)
- `copy` (stdlib)
- `dataclasses` (stdlib)
- `itertools` (stdlib)
- `json` (stdlib)
- `logging` (stdlib)
- `os` (stdlib)
- `time` (stdlib)
- `types` (stdlib)
- `typing` (stdlib)
