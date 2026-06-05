# rlhf_async_new_apis.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/rl/rlhf_async_new_apis.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates async reinforcement learning using vLLM and Ray, with native weight syncing APIs and batch-invariant generation / 演示 vLLM 示例目录中与 rlhf async new apis 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Demonstrates async reinforcement learning using vLLM and Ray,
with native weight syncing APIs and batch-invariant generation.

The script separates training and inference workloads onto distinct GPUs
so that Ray can manage process placement and inter-process communication.
A Hugging Face Transformer model occupies one GPU for training, and a
vLLM AsyncLLMEngine occupies another GPU for inference.

Batch invariance is enabled so that generation output is deterministic
regardless of how many requests are batched together. This is required
for the validation phase to succeed. Batch invariance currently requires
# ... key logic omitted for brevity ...
  directly with the training model and comparing its output to the
  post-swap tokens from the weight-synced engine.

This example assumes a single-node cluster with two GPUs, but Ray
supports multi-node clusters. vLLM expects the GPUs are only used for vLLM
workloads. Residual GPU activity interferes with vLLM memory profiling and
causes unexpected behavior.
"""
```
**EN:** Demonstrates async reinforcement learning using vLLM and Ray, with native weight syncing APIs and batch-invariant generation.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import asyncio
import uuid
from dataclasses import asdict

import ray
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

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
from vllm.platforms import current_platform
from vllm.utils.network_utils import get_ip, get_open_port
from vllm.v1.executor import Executor
```
**EN:** This block loads helper libraries such as asyncio, uuid, dataclasses, ray, and torch and pulls in vLLM APIs like vllm, vllm.config, vllm.distributed.weight_transfer.base, vllm.distributed.weight_transfer.nccl_engine, and vllm.platforms.
**CN:** 这一部分加载 asyncio、uuid、dataclasses、ray，以及 torch 等辅助库，并引入 vllm、vllm.config、vllm.distributed.weight_transfer.base、vllm.distributed.weight_transfer.nccl_engine，以及 vllm.platforms 等 vLLM API。

### Top-level setup
```python
MODEL_NAME_V1 = "Qwen/Qwen3-1.7B-Base"
MODEL_NAME_V2 = "Qwen/Qwen3-1.7B"
PAUSE_TOKEN_THRESHOLD = 10
ATTN_BACKEND = "TRITON_ATTN" if current_platform.is_rocm() else "FLASH_ATTN"
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as MODEL_NAME_V1, MODEL_NAME_V2, PAUSE_TOKEN_THRESHOLD, and ATTN_BACKEND. It also performs early helper calls such as current_platform.is_rocm.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 MODEL_NAME_V1、MODEL_NAME_V2、PAUSE_TOKEN_THRESHOLD，以及 ATTN_BACKEND 等变量。它还会提前执行 current_platform.is_rocm 等辅助调用。

### Class: MyLLM
```python
class MyLLM(vllm.AsyncLLMEngine):
    """Configure the vLLM worker for Ray placement group execution."""

    def __init__(self, **kwargs):
        engine_args = vllm.AsyncEngineArgs(**kwargs)
        vllm_config = engine_args.create_engine_config()
        executor_class = Executor.get_class(vllm_config)
        super().__init__(
            vllm_config=vllm_config,
            executor_class=executor_class,
            log_requests=engine_args.enable_log_requests,
            log_stats=not engine_args.disable_log_stats,
        )
        self._generation_paused = False
        self._request_pause_flag = False

    async def do_generate(
        self, prompt_token_ids: list[int], sampling_params: vllm.SamplingParams
    ) -> tuple[vllm.RequestOutput, int]:
        """Generate a single request, setting the request pause flag once the
    # ... key logic omitted for brevity ...
            ):
                self._request_pause_flag = True
            if self._generation_paused and pause_token_index == -1:
                pause_token_index = prev_token_count
            prev_token_count = cur_token_count
        return output, pause_token_index

    async def pause_after_n_tokens(self):
        """Wait for any request to set the pause flag, then pause."""
        while not self._request_pause_flag:
            await asyncio.sleep(0)
        await super().pause_generation(mode="keep")
        await asyncio.sleep(5)
        self._generation_paused = True
```
**EN:** Configure the vLLM worker for Ray placement group execution.. It extends vllm.AsyncLLMEngine. Notable methods include __init__, do_generate, and pause_after_n_tokens.
**CN:** 该类封装了示例中使用的 MyLLM 抽象。它继承自 vllm.AsyncLLMEngine。较重要的方法包括 __init__、do_generate，以及 pause_after_n_tokens。

### Class: TrainModel
```python
class TrainModel:
    """Ray actor that wraps the training model on a dedicated GPU."""

    def __init__(self, model_name: str):
        from vllm.model_executor.layers.batch_invariant import (
            init_batch_invariance,
        )

        # need to init all env vars for batch invariance which affect nccl ops
        init_batch_invariance()

        self.model = AutoModelForCausalLM.from_pretrained(
            model_name, dtype=torch.bfloat16
        ).to("cuda:0")
        self.port = get_open_port()
        self.master_address = get_ip()

    def get_master_address_and_port(self):
        return self.master_address, self.port

    # ... key logic omitted for brevity ...
            trainer_args=trainer_args,
        )

    @torch.inference_mode()
    def generate(self, token_ids: list[int], max_new_tokens: int) -> list[int]:
        """Greedy-decode max_new_tokens from the given context."""
        input_ids = torch.tensor([token_ids], device="cuda:0")
        output = self.model.generate(
            input_ids,
            max_new_tokens=max_new_tokens,
            do_sample=False,
        )
        new_token_ids = output[0, len(token_ids) :].tolist()
        return new_token_ids
```
**EN:** Ray actor that wraps the training model on a dedicated GPU.. Notable methods include __init__, get_master_address_and_port, get_weight_metadata, init_weight_transfer_group, broadcast_weights, and generate.
**CN:** 该类封装了示例中使用的 TrainModel 抽象。较重要的方法包括 __init__、get_master_address_and_port、get_weight_metadata、init_weight_transfer_group、broadcast_weights，以及 generate。

### Top-level setup
```python
ray_env_vars = {
    # Prevent Ray from setting CUDA_VISIBLE_DEVICES
    "RAY_EXPERIMENTAL_NOSET_CUDA_ENV_VAR": "1",
}

if current_platform.is_rocm():
    # For ROCm, BATCH_INVARIANT vllm is not supported
    ray_env_vars["VLLM_ROCM_USE_SKINNY_GEMM"] = "0"
else:
    # Enable batch invariance for deterministic outputs on NVIDIA
    ray_env_vars["VLLM_BATCH_INVARIANT"] = "1"

    # ... key logic omitted for brevity ...
print(f"  Required: >= {MIN_PASS_RATE:.0%}")

assert pass_rate >= MIN_PASS_RATE, (
    f"Validation pass rate {pass_rate:.0%} ({num_pass}/{num_total}) "
    f"is below the required {MIN_PASS_RATE:.0%} threshold. "
    f"See failures above for details."
)
print("=" * 50)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as ray_env_vars, train_model, rocm_determinism_kwargs, llm_kwargs, llm, and PROMPTS. It also performs early helper calls such as current_platform.is_rocm, ray.init, TrainModel.remote, dict, and WeightTransferConfig.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 ray_env_vars、train_model、rocm_determinism_kwargs、llm_kwargs、llm，以及 PROMPTS 等变量。它还会提前执行 current_platform.is_rocm、ray.init、TrainModel.remote、dict，以及 WeightTransferConfig 等辅助调用。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.config`, `vllm.distributed.weight_transfer.base`, `vllm.distributed.weight_transfer.nccl_engine`, `vllm.platforms`, `vllm.utils.network_utils`, `vllm.v1.executor` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `asyncio`, `uuid`, `dataclasses`, `ray`, `torch`, `transformers` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `MyLLM`, `TrainModel` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `current_platform.is_rocm`, `asyncio.sleep`, `super`, `vllm.AsyncEngineArgs`, `engine_args.create_engine_config`, `Executor.get_class`, `__init__`, `self.generate` reveal the main execution path / 这些调用体现了主要执行链路。
