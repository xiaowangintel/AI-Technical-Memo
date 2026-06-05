# rlhf_nccl_fsdp_ep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/rl/rlhf_nccl_fsdp_ep.py`
- **Repository**: vllm-project/vllm
- **Purpose**: RLHF with FSDP2 training (4 GPUs) and vLLM expert-parallel inference (4 GPUs) / 演示 vLLM 示例目录中与 rlhf nccl fsdp ep 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
RLHF with FSDP2 training (4 GPUs) and vLLM expert-parallel inference (4 GPUs).

8-GPU layout:
  Training  — 4 GPUs, PyTorch FSDP2 (fully_shard)
  Inference — 4 GPUs, vLLM AsyncLLMEngine with expert parallelism +
              data parallelism (TP=1, DP=4, enable_expert_parallel
              → EP_SIZE = TP×DP = 4)

FSDP workers are Ray actors that form a single FSDP2 process group.
Rank 0 gathers full parameters via DTensor.full_tensor() and broadcasts
them to the vLLM inference engine through the NCCL weight-transfer API.

The inference engine uses AsyncLLMEngine which automatically spawns
DP worker processes (no manual placement group needed).  Weight sync
uses pause_generation / resume_generation.

Steps:
  1. Launch 4 FSDP training workers.
  2. Launch AsyncLLMEngine with EP+DP (dummy weights).
  3. Generate from prompts → gibberish (random weights).
  4. Pause generation, transfer weights from FSDP, resume.
  5. Generate from prompts → sensible output (synced weights).

Assumes a single-node cluster with 8 GPUs.
"""
```
**EN:** RLHF with FSDP2 training (4 GPUs) and vLLM expert-parallel inference (4 GPUs).
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import asyncio
import os
import uuid
from dataclasses import asdict

import ray
import torch
import torch.distributed as dist
from huggingface_hub import snapshot_download
from torch.distributed.fsdp import fully_shard
from transformers import AutoModelForCausalLM

import vllm
from vllm import SamplingParams
from vllm.config import WeightTransferConfig
from vllm.distributed.weight_transfer.base import (
    WeightTransferInitRequest,
    WeightTransferUpdateRequest,
)
from vllm.distributed.weight_transfer.nccl_engine import (
    NCCLTrainerSendWeightsArgs,
    NCCLWeightTransferEngine,
    NCCLWeightTransferInitInfo,
    NCCLWeightTransferUpdateInfo,
)
from vllm.utils.network_utils import get_ip, get_open_port
from vllm.v1.executor import Executor
```
**EN:** This block loads helper libraries such as asyncio, os, uuid, dataclasses, and ray and pulls in vLLM APIs like vllm, vllm.config, vllm.distributed.weight_transfer.base, vllm.distributed.weight_transfer.nccl_engine, and vllm.utils.network_utils.
**CN:** 这一部分加载 asyncio、os、uuid、dataclasses，以及 ray 等辅助库，并引入 vllm、vllm.config、vllm.distributed.weight_transfer.base、vllm.distributed.weight_transfer.nccl_engine，以及 vllm.utils.network_utils 等 vLLM API。

### Top-level setup
```python
MODEL_NAME = "Qwen/Qwen3-30B-A3B"

FSDP_WORLD_SIZE = 4
INFERENCE_TP_SIZE = 1
INFERENCE_DP_SIZE = 4
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as MODEL_NAME, FSDP_WORLD_SIZE, INFERENCE_TP_SIZE, and INFERENCE_DP_SIZE.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 MODEL_NAME、FSDP_WORLD_SIZE、INFERENCE_TP_SIZE，以及 INFERENCE_DP_SIZE 等变量。

### Class: FSDPTrainWorker
```python
class FSDPTrainWorker:
    """
    One FSDP2 training worker per GPU.  Four of these form the FSDP group.
    Rank 0 additionally handles weight transfer to the vLLM engine.
    """

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
    # ... key logic omitted for brevity ...
                for name, param in self.model.named_parameters():
                    yield name, param.full_tensor()

            trainer_args = NCCLTrainerSendWeightsArgs(
                group=self.model_update_group,
                packed=packed,
            )
            NCCLWeightTransferEngine.trainer_send_weights(
                iterator=_full_param_iter(),
                trainer_args=trainer_args,
            )
        else:
            for _, param in self.model.named_parameters():
                param.full_tensor()
```
**EN:** One FSDP2 training worker per GPU.. Notable methods include __init__, get_rank, setup_transfer_endpoint, init_weight_transfer_group, get_weight_metadata, and gather_and_broadcast_weights.
**CN:** 该类封装了示例中使用的 FSDPTrainWorker 抽象。较重要的方法包括 __init__、get_rank、setup_transfer_endpoint、init_weight_transfer_group、get_weight_metadata，以及 gather_and_broadcast_weights。

### Function: create_async_engine
```python
def create_async_engine(**kwargs):
    """Create an AsyncLLMEngine directly (no subclass needed)."""
    engine_args = vllm.AsyncEngineArgs(**kwargs)
    vllm_config = engine_args.create_engine_config()
    executor_class = Executor.get_class(vllm_config)
    return vllm.AsyncLLMEngine(
        vllm_config=vllm_config,
        executor_class=executor_class,
        log_requests=engine_args.enable_log_requests,
        log_stats=not engine_args.disable_log_stats,
    )
```
**EN:** Create an AsyncLLMEngine directly (no subclass needed).. Key operations include vllm.AsyncEngineArgs, engine_args.create_engine_config, Executor.get_class, and vllm.AsyncLLMEngine. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。关键操作包括 vllm.AsyncEngineArgs、engine_args.create_engine_config、Executor.get_class，以及 vllm.AsyncLLMEngine。其返回值会继续传给示例管线的下一阶段。

### Async function: generate_batch
```python
async def generate_batch(engine, prompts, sampling_params):
    """Generate completions for a batch of prompts."""

    async def gen_one(prompt):
        output = None
        async for request_output in engine.generate(
            {"prompt": prompt},
            sampling_params,
            request_id=str(uuid.uuid4()),
        ):
            output = request_output
        return output

    return await asyncio.gather(*[gen_one(p) for p in prompts])
```
**EN:** Generate completions for a batch of prompts.. It works with parameters such as engine, prompts, and sampling_params. Key operations include engine.generate, asyncio.gather, str, uuid.uuid4, and gen_one. The return value feeds the next stage of the example pipeline.
**CN:** 该函数以异步方式触发模型推理并收集输出。它会处理 engine、prompts，以及 sampling_params 等参数。关键操作包括 engine.generate、asyncio.gather、str、uuid.uuid4，以及 gen_one。其返回值会继续传给示例管线的下一阶段。

### Async function: main
```python
async def main():
    ray.init()

    # Download model weights to local/shared disk once.
    local_model_path = snapshot_download(MODEL_NAME)
    print(f"[init] Model downloaded to {local_model_path}")

    # FSDP rendezvous address (single-node)
    fsdp_master_addr = get_ip()
    fsdp_master_port = get_open_port()

    # Launch 4 FSDP training workers.
    # Ray allocates 1 GPU per worker; AsyncLLMEngine's internal DP
    # placement groups will land on the remaining 4 GPUs.
    fsdp_workers = [
        FSDPTrainWorker.remote(
            local_model_path,
            rank,
            FSDP_WORLD_SIZE,
            fsdp_master_addr,
    # ... key logic omitted for brevity ...
    print("[sync] Generation resumed.")

    # Generate with synced weights — expect sensible output.
    print("[generate] Starting generation with synced weights...")
    outputs_updated = await generate_batch(engine, prompts, sampling_params)
    print("[generate] Generation complete.")

    print("-" * 60)
    print("AFTER weight sync (real weights):")
    print("-" * 60)
    for output in outputs_updated:
        print(f"Prompt: {output.prompt!r}")
        print(f"Generated: {output.outputs[0].text!r}")
        print("-" * 60)
```
**EN:** This function asynchronously orchestrates the end-to-end workflow. Key operations include print, ray.get, generate_batch, asdict, and ray.init.
**CN:** 该函数以异步方式编排端到端工作流。关键操作包括 print、ray.get、generate_batch、asdict，以及 ray.init。

### Entry point
```python
if __name__ == "__main__":
    asyncio.run(main())
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to asyncio.run and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 asyncio.run 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.config`, `vllm.distributed.weight_transfer.base`, `vllm.distributed.weight_transfer.nccl_engine`, `vllm.utils.network_utils`, `vllm.v1.executor` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `asyncio`, `os`, `uuid`, `dataclasses`, `ray`, `torch`, `torch.distributed`, `huggingface_hub` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `FSDPTrainWorker`, `create_async_engine`, `generate_batch`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `model.named_parameters`, `str`, `fully_shard`, `self.model.named_parameters`, `param.full_tensor`, `ray.remote`, `dist.init_process_group`, `torch.accelerator.set_device_index` reveal the main execution path / 这些调用体现了主要执行链路。
