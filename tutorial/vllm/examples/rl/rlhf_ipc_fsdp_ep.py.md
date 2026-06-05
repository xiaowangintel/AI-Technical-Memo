# rlhf_ipc_fsdp_ep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/rl/rlhf_ipc_fsdp_ep.py`
- **Repository**: vllm-project/vllm
- **Purpose**: RLHF with FSDP2 training and vLLM expert-parallel inference using **CUDA IPC** weight transfer and **packed** tensors / 演示 vLLM 示例目录中与 rlhf ipc fsdp ep 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
RLHF with FSDP2 training and vLLM expert-parallel inference using **CUDA IPC**
weight transfer and **packed** tensors.

Layout (4 GPUs, TP=1, DP=4, EP):
  * One Ray placement group per GPU.
  * Each PG holds one FSDP training worker and one vLLM ``LLM`` instance
    (sync API) using fractional GPUs so both fit on the same device.
  * The 4 ``LLM`` instances form a DP group via env-var-based SPMD
    coordination (``VLLM_DP_RANK``, ``VLLM_DP_SIZE``, etc.), the same
    mechanism used by ``examples/offline_inference/data_parallel.py``.
  * A ``DataParallelInferenceEngine`` actor spawns all 4 LLM actors,
    waits for initialization, and orchestrates generation / weight-sync.

Uses the built-in ``ray`` send_mode: each FSDP worker calls
``trainer_send_weights`` targeting its colocated LLM actor.

This example was run on 4xH100.
"""
```
**EN:** RLHF with FSDP2 training and vLLM expert-parallel inference using **CUDA IPC** weight transfer and **packed** tensors.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from __future__ import annotations

import os
from dataclasses import asdict

import ray
import torch
import torch.distributed as dist
from huggingface_hub import snapshot_download
from ray.util.placement_group import placement_group
from ray.util.scheduling_strategies import PlacementGroupSchedulingStrategy
from torch.distributed._tensor import DTensor
from torch.distributed.fsdp import fully_shard
from transformers import AutoModelForCausalLM

from vllm import LLM, SamplingParams
from vllm.config import WeightTransferConfig
from vllm.distributed.weight_transfer.ipc_engine import (
    IPCTrainerSendWeightsArgs,
    IPCWeightTransferEngine,
    IPCWeightTransferInitInfo,
)
from vllm.utils.network_utils import get_ip, get_open_port
```
**EN:** This block loads helper libraries such as __future__, os, dataclasses, ray, and torch and pulls in vLLM APIs like vllm, vllm.config, vllm.distributed.weight_transfer.ipc_engine, and vllm.utils.network_utils.
**CN:** 这一部分加载 __future__、os、dataclasses、ray，以及 torch 等辅助库，并引入 vllm、vllm.config、vllm.distributed.weight_transfer.ipc_engine，以及 vllm.utils.network_utils 等 vLLM API。

### Top-level setup
```python
TRAIN_GPU_FRACTION = float(os.environ.get("RLHF_IPC_TRAIN_GPU_FRACTION", "0.42"))
VLLM_GPU_FRACTION = float(os.environ.get("RLHF_IPC_VLLM_GPU_FRACTION", "0.42"))

MODEL_NAME = "Qwen/Qwen3-30B-A3B"

FSDP_WORLD_SIZE = 4
INFERENCE_TP_SIZE = 1
INFERENCE_DP_SIZE = 4
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as TRAIN_GPU_FRACTION, VLLM_GPU_FRACTION, MODEL_NAME, FSDP_WORLD_SIZE, INFERENCE_TP_SIZE, and INFERENCE_DP_SIZE. It also performs early helper calls such as float and os.environ.get.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 TRAIN_GPU_FRACTION、VLLM_GPU_FRACTION、MODEL_NAME、FSDP_WORLD_SIZE、INFERENCE_TP_SIZE，以及 INFERENCE_DP_SIZE 等变量。它还会提前执行 float 和 os.environ.get 等辅助调用。

### Class: MyLLM
```python
class MyLLM(LLM):
    """LLM subclass that configures DP env vars for SPMD coordination."""

    def __init__(
        self,
        *args,
        dp_rank: int = 0,
        dp_size: int = 1,
        dp_master_ip: str = "127.0.0.1",
        dp_master_port: int = 0,
        **kwargs,
    ):
        os.environ.pop("CUDA_VISIBLE_DEVICES", None)
        os.environ["VLLM_RAY_PER_WORKER_GPUS"] = str(VLLM_GPU_FRACTION)
        os.environ["VLLM_RAY_BUNDLE_INDICES"] = "0"
        os.environ["VLLM_ALLOW_INSECURE_SERIALIZATION"] = "1"

        os.environ["VLLM_DP_RANK"] = str(dp_rank)
        os.environ["VLLM_DP_RANK_LOCAL"] = str(dp_rank)
        os.environ["VLLM_DP_SIZE"] = str(dp_size)
        os.environ["VLLM_DP_MASTER_IP"] = dp_master_ip
        os.environ["VLLM_DP_MASTER_PORT"] = str(dp_master_port)

        super().__init__(*args, **kwargs)

    def ready(self):
        return True
```
**EN:** LLM subclass that configures DP env vars for SPMD coordination.. It extends LLM. Notable methods include __init__ and ready.
**CN:** 该类封装了示例中使用的 MyLLM 抽象。它继承自 LLM。较重要的方法包括 __init__ 和 ready。

### Class: FSDPTrainWorker
```python
class FSDPTrainWorker:
    """One FSDP2 worker per GPU; colocated with vLLM DP rank via placement group."""

    def __init__(
        self,
        model_name: str,
        rank: int,
        fsdp_world_size: int,
        fsdp_master_addr: str,
        fsdp_master_port: int,
    ):
        self.rank = rank

        os.environ["MASTER_ADDR"] = fsdp_master_addr
        os.environ["MASTER_PORT"] = str(fsdp_master_port)

        dist.init_process_group(backend="nccl", rank=rank, world_size=fsdp_world_size)
        torch.accelerator.set_device_index(0)

        model = AutoModelForCausalLM.from_pretrained(
    # ... key logic omitted for brevity ...
                    del tensor
                else:
                    yield name, tensor

        trainer_args = IPCTrainerSendWeightsArgs(
            send_mode="ray",
            llm_handle=llm_handle,
            packed=packed,
            packed_buffer_size_bytes=1024 * 1024 * 1024,  # 1 GB
        )
        IPCWeightTransferEngine.trainer_send_weights(
            iterator=_full_param_iter(),
            trainer_args=trainer_args,
        )
```
**EN:** One FSDP2 worker per GPU; colocated with vLLM DP rank via placement group.. Notable methods include __init__, get_rank, get_weight_metadata, and gather_and_broadcast_weights_ipc.
**CN:** 该类封装了示例中使用的 FSDPTrainWorker 抽象。较重要的方法包括 __init__、get_rank、get_weight_metadata，以及 gather_and_broadcast_weights_ipc。

### Class: DataParallelInferenceEngine
```python
class DataParallelInferenceEngine:
    """Manages a pool of DP-sharded vLLM LLM actors.

    Spawns one MyLLM actor per placement group, waits for all engines to
    finish initializing, and exposes generation / weight-sync helpers.
    """

    def __init__(
        self,
        model: str,
        pgs: list,
        dp_master_ip: str,
        dp_master_port: int,
    ):
        dp_size = len(pgs)
        self.llm_actors = []
        for r in range(dp_size):
            sched = PlacementGroupSchedulingStrategy(
                placement_group=pgs[r],
                placement_group_capture_child_tasks=True,
    # ... key logic omitted for brevity ...
                    is_checkpoint_format=is_checkpoint_format
                )
                for actor in self.llm_actors
            ]
        )

    def finish_weight_update(self):
        ray.get([actor.finish_weight_update.remote() for actor in self.llm_actors])

    def sleep(self, level: int = 0):
        ray.get([actor.sleep.remote(level=level) for actor in self.llm_actors])

    def wake_up(self, tags: list[str] | None = None):
        ray.get([actor.wake_up.remote(tags=tags) for actor in self.llm_actors])
```
**EN:** Manages a pool of DP-sharded vLLM LLM actors.. Notable methods include __init__, get_llm_actors, generate, init_weight_transfer, start_weight_update, finish_weight_update, sleep, and wake_up.
**CN:** 该类封装了示例中使用的 DataParallelInferenceEngine 抽象。较重要的方法包括 __init__、get_llm_actors、generate、init_weight_transfer、start_weight_update、finish_weight_update、sleep，以及 wake_up。

### Function: main
```python
def main():
    ray.init(
        runtime_env={
            "env_vars": {
                "VLLM_ALLOW_INSECURE_SERIALIZATION": "1",
            }
        }
    )

    assert TRAIN_GPU_FRACTION + VLLM_GPU_FRACTION <= 1.0, (
        "Train + vLLM GPU fractions must sum to at most 1.0 per bundle."
    )

    local_model_path = snapshot_download(MODEL_NAME)
    print(f"[init] Model downloaded to {local_model_path}")

    fsdp_master_addr = get_ip()
    fsdp_master_port = get_open_port()
    dp_master_port = get_open_port()
    dp_master_ip = get_ip()
    # ... key logic omitted for brevity ...
    print("[sync] Waking KV cache + scheduling...")
    ray.get(inference_engine.wake_up.remote(tags=["kv_cache", "scheduling"]))

    print("[generate] Generating with synced weights...")
    outputs_updated = ray.get(
        inference_engine.generate.remote(prompts, sampling_params)
    )
    print("-" * 60)
    print("AFTER weight sync (real weights):")
    print("-" * 60)
    for output in outputs_updated:
        print(f"Prompt: {output.prompt!r}")
        print(f"Generated: {output.outputs[0].text!r}")
        print("-" * 60)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print, ray.get, range, get_ip, and get_open_port.
**CN:** 该函数编排端到端工作流。关键操作包括 print、ray.get、range、get_ip，以及 get_open_port。

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

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.config`, `vllm.distributed.weight_transfer.ipc_engine`, `vllm.utils.network_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `__future__`, `os`, `dataclasses`, `ray`, `torch`, `torch.distributed`, `huggingface_hub`, `ray.util.placement_group` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `MyLLM`, `FSDPTrainWorker`, `DataParallelInferenceEngine`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `float`, `os.environ.get`, `str`, `os.environ.pop`, `__init__`, `super`, `contiguous`, `model.named_parameters` reveal the main execution path / 这些调用体现了主要执行链路。
