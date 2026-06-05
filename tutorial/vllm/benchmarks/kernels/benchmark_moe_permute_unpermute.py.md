# benchmark_moe_permute_unpermute.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_moe_permute_unpermute.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, mixture-of-experts benchmarks, centered around `FP8_DTYPE`, `BenchmarkConfig`, `benchmark_permute`, `benchmark_unpermute`. / 实现与基准测试编排、混合专家基准相关的逻辑，核心符号包括 `FP8_DTYPE`, `BenchmarkConfig`, `benchmark_permute`, `benchmark_unpermute`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-19)
```python
import argparse
from typing import Any, TypedDict

import ray
import torch
from transformers import AutoConfig

from vllm.model_executor.layers.fused_moe import fused_topk
from vllm.model_executor.layers.fused_moe.moe_permute_unpermute import (
    moe_permute,
    moe_unpermute,
)
from vllm.model_executor.layers.fused_moe.utils import _fp8_quantize
from vllm.platforms import current_platform
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block gathers standard-library helpers such as `argparse`, `typing`; third-party packages such as `ray`, `torch`, `transformers`; project-local modules such as `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `typing`；第三方依赖，如 `ray`, `torch`, `transformers`；项目内部模块，如 `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 21-21)
```python
FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** This top-level block prepares shared state such as `FP8_DTYPE`. It uses `current_platform.fp8_dtype` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `FP8_DTYPE`。它借助 `current_platform.fp8_dtype` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Class `BenchmarkConfig` (lines 24-30)
```python
class BenchmarkConfig(TypedDict):
    BLOCK_SIZE_M: int
    BLOCK_SIZE_N: int
    BLOCK_SIZE_K: int
    GROUP_SIZE_M: int
    num_warps: int
    num_stages: int
```
**EN:** Class `BenchmarkConfig` packages shared state and related operations for this benchmark module. It extends `TypedDict` and exposes methods such as its methods.
**CN:** 类 `BenchmarkConfig` 用于封装该基准模块的共享状态与相关操作。它继承自 `TypedDict`，并提供 its methods 等方法。

### Function `benchmark_permute` (lines 33-101)
```python
def benchmark_permute(
    num_tokens: int,
    num_experts: int,
    hidden_size: int,
    topk: int,
    dtype: torch.dtype,
    use_fp8_w8a8: bool,
    use_int8_w8a16: bool,
    num_iters: int = 100,
) -> float:
    # init_dtype = torch.float16 if use_fp8_w8a8 else dtype
    hidden_states = torch.randn(num_tokens, hidden_size, dtype=dtype)
    # output_hidden_states = torch.empty_like(hidden_states)
    if use_fp8_w8a8:
        qhidden_states, scale = _fp8_quantize(hidden_states, None, None)
    else:
        qhidden_states = hidden_states

    gating_output = torch.randn(num_iters, num_tokens, num_experts, dtype=torch.float32)

    input_gating = torch.randn(num_tokens, num_experts, dtype=torch.float32)
    topk_weights, topk_ids, token_expert_indices = fused_topk(
        qhidden_states, input_gating, topk, False
    )

    def prepare(i: int):
        input_gating.copy_(gating_output[i])

    def run():
        moe_permute(
            qhidden_states,
            a1q_scale=None,
            topk_ids=topk_ids,
            n_expert=num_experts,
    # ... omitted for brevity ...
        start_event.record()
        graph.replay()
        end_event.record()
        end_event.synchronize()
        latencies.append(start_event.elapsed_time(end_event))
    avg = sum(latencies) / (num_iters * 10) * 1000  # us
    graph.reset()
    return avg
```
**EN:** `benchmark_permute` coordinates or measures benchmark orchestration. It mainly works with `num_tokens`, `num_experts`, `hidden_size`, `topk`, `dtype`, ... and relies on `torch.randn`, `_fp8_quantize`, `fused_topk`, `input_gating.copy_`, `moe_permute`, `run` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_permute` 负责协调或测量基准测试编排。 它主要处理 `num_tokens`, `num_experts`, `hidden_size`, `topk`, `dtype`, ...，并结合 `torch.randn`, `_fp8_quantize`, `fused_topk`, `input_gating.copy_`, `moe_permute`, `run` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `benchmark_unpermute` (lines 104-189)
```python
def benchmark_unpermute(
    num_tokens: int,
    num_experts: int,
    hidden_size: int,
    topk: int,
    dtype: torch.dtype,
    use_fp8_w8a8: bool,
    use_int8_w8a16: bool,
    num_iters: int = 100,
) -> float:
    # init_dtype = torch.float16 if use_fp8_w8a8 else dtype
    hidden_states = torch.randn(num_tokens, hidden_size, dtype=dtype)
    if use_fp8_w8a8:
        qhidden_states, scale = _fp8_quantize(hidden_states, None, None)
    else:
        qhidden_states = hidden_states

    input_gating = torch.randn(num_tokens, num_experts, dtype=torch.float32)

    topk_weights, topk_ids, token_expert_indices = fused_topk(
        qhidden_states, input_gating, topk, False
    )

    def prepare():
        (
            permuted_hidden_states,
            _,
            first_token_off,
            inv_perm_idx,
            _,
        ) = moe_permute(
            qhidden_states,
            a1q_scale=None,
            topk_ids=topk_ids,
    # ... omitted for brevity ...
        start_event.record()
        graph.replay()
        end_event.record()
        end_event.synchronize()
        latencies.append(start_event.elapsed_time(end_event))
    avg = sum(latencies) / (num_iters * 10) * 1000  # us
    graph.reset()
    return avg
```
**EN:** `benchmark_unpermute` coordinates or measures benchmark orchestration. It mainly works with `num_tokens`, `num_experts`, `hidden_size`, `topk`, `dtype`, ... and relies on `torch.randn`, `_fp8_quantize`, `fused_topk`, `moe_permute`, `permuted_hidden_states.to`, `torch.empty_like` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `benchmark_unpermute` 负责协调或测量基准测试编排。 它主要处理 `num_tokens`, `num_experts`, `hidden_size`, `topk`, `dtype`, ...，并结合 `torch.randn`, `_fp8_quantize`, `fused_topk`, `moe_permute`, `permuted_hidden_states.to`, `torch.empty_like` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Class `BenchmarkWorker` (lines 193-235)
```python
class BenchmarkWorker:
    def __init__(self, seed: int) -> None:
        torch.set_default_device("cuda")
        set_random_seed(seed)
        self.seed = seed
        # Get the device ID to allocate tensors and kernels
        # on the respective GPU. This is required for Ray to work
        # correctly with multi-GPU tuning on the ROCm platform.
        self.device_id = int(ray.get_gpu_ids()[0])

    def benchmark(
        self,
        num_tokens: int,
        num_experts: int,
        hidden_size: int,
        topk: int,
        dtype: torch.dtype,
        use_fp8_w8a8: bool,
        use_int8_w8a16: bool,
    ) -> tuple[float, float]:
        set_random_seed(self.seed)

        permute_time = benchmark_permute(
            num_tokens,
            num_experts,
            hidden_size,
            topk,
            dtype,
            use_fp8_w8a8,
            use_int8_w8a16,
            num_iters=100,
        )
        unpermute_time = benchmark_unpermute(
            num_tokens,
            num_experts,
            hidden_size,
            topk,
            dtype,
            use_fp8_w8a8,
            use_int8_w8a16,
            num_iters=100,
        )
        return permute_time, unpermute_time
```
**EN:** Class `BenchmarkWorker` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `benchmark`.
**CN:** 类 `BenchmarkWorker` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `benchmark` 等方法。

### Method `BenchmarkWorker.__init__` (lines 194-201)
```python
    def __init__(self, seed: int) -> None:
        torch.set_default_device("cuda")
        set_random_seed(seed)
        self.seed = seed
        # Get the device ID to allocate tensors and kernels
        # on the respective GPU. This is required for Ray to work
        # correctly with multi-GPU tuning on the ROCm platform.
        self.device_id = int(ray.get_gpu_ids()[0])
```
**EN:** `__init__` implements a helper used by `benchmark_moe_permute_unpermute.py`. It mainly works with `seed` and relies on `torch.set_default_device`, `set_random_seed`, `int`, `ray.get_gpu_ids` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_moe_permute_unpermute.py` 使用的辅助逻辑。 它主要处理 `seed`，并结合 `torch.set_default_device`, `set_random_seed`, `int`, `ray.get_gpu_ids` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `BenchmarkWorker.benchmark` (lines 203-235)
```python
    def benchmark(
        self,
        num_tokens: int,
        num_experts: int,
        hidden_size: int,
        topk: int,
        dtype: torch.dtype,
        use_fp8_w8a8: bool,
        use_int8_w8a16: bool,
    ) -> tuple[float, float]:
        set_random_seed(self.seed)

        permute_time = benchmark_permute(
            num_tokens,
            num_experts,
            hidden_size,
            topk,
            dtype,
            use_fp8_w8a8,
            use_int8_w8a16,
            num_iters=100,
        )
        unpermute_time = benchmark_unpermute(
            num_tokens,
            num_experts,
            hidden_size,
            topk,
            dtype,
            use_fp8_w8a8,
            use_int8_w8a16,
            num_iters=100,
        )
        return permute_time, unpermute_time
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `num_tokens`, `num_experts`, `hidden_size`, `topk`, `dtype`, ... and relies on `set_random_seed`, `benchmark_permute`, `benchmark_unpermute` plus value production to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `num_tokens`, `num_experts`, `hidden_size`, `topk`, `dtype`, ...，并结合 `set_random_seed`, `benchmark_permute`, `benchmark_unpermute` 以及 结果返回 来完成这一段基准测试流程。

### Function `get_weight_block_size_safety` (lines 238-242)
```python
def get_weight_block_size_safety(config, default_value=None):
    quantization_config = getattr(config, "quantization_config", {})
    if isinstance(quantization_config, dict):
        return quantization_config.get("weight_block_size", default_value)
    return default_value
```
**EN:** `get_weight_block_size_safety` implements a helper used by `benchmark_moe_permute_unpermute.py`. It mainly works with `config`, `default_value` and relies on `getattr`, `isinstance`, `quantization_config.get` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_weight_block_size_safety` 负责实现 `benchmark_moe_permute_unpermute.py` 使用的辅助逻辑。 它主要处理 `config`, `default_value`，并结合 `getattr`, `isinstance`, `quantization_config.get` 以及 条件分支 来完成这一段基准测试流程。

### Function `main` (lines 245-339)
```python
def main(args: argparse.Namespace):
    print(args)

    config = AutoConfig.from_pretrained(
        args.model, trust_remote_code=args.trust_remote_code
    )
    if config.architectures[0] == "DbrxForCausalLM":
        E = config.ffn_config.moe_num_experts
        topk = config.ffn_config.moe_top_k
    elif config.architectures[0] == "JambaForCausalLM":
        E = config.num_experts
        topk = config.num_experts_per_tok
    elif (
        config.architectures[0] == "DeepseekV3ForCausalLM"
        or config.architectures[0] == "DeepseekV2ForCausalLM"
        or config.architectures[0] == "Glm4MoeForCausalLM"
        or config.architectures[0] == "Glm4MoeLiteForCausalLM"
    ):
        E = config.n_routed_experts
        topk = config.num_experts_per_tok
    elif config.architectures[0] in ["Qwen2MoeForCausalLM", "Qwen3MoeForCausalLM"]:
        E = config.num_experts
        topk = config.num_experts_per_tok

    else:
        # Support for llama4
        config = config.get_text_config()
        # Default: Mixtral.
        E = config.num_local_experts
        topk = config.num_experts_per_tok

    hidden_size = config.hidden_size
    dtype = torch.float16 if current_platform.is_rocm() else config.dtype
    use_fp8_w8a8 = args.dtype == "fp8_w8a8"
    # ... omitted for brevity ...
            for batch_size in batch_sizes
        ],
    )

    for batch_size, (permute, unpermute) in zip(batch_sizes, outputs):
        print(f"Batch size: {batch_size}")
        print(f"Permute time: {permute:.2f} us")
        print(f"Unpermute time: {unpermute:.2f} us")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `print`, `AutoConfig.from_pretrained`, `config.get_text_config`, `current_platform.is_rocm`, `ray.init`, `int` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `print`, `AutoConfig.from_pretrained`, `config.get_text_config`, `current_platform.is_rocm`, `ray.init`, `int` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 342-355)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser()
    parser.add_argument(
        "--model", type=str, default="mistralai/Mixtral-8x7B-Instruct-v0.1"
    )
    parser.add_argument(
        "--dtype", type=str, choices=["auto", "fp8_w8a8", "int8_w8a16"], default="auto"
    )
    parser.add_argument("--seed", type=int, default=0)
    parser.add_argument("--batch-size", type=int, required=False)
    parser.add_argument("--trust-remote-code", action="store_true")
    args = parser.parse_args()

    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Distributed execution: the code coordinates work across processes, devices, or Ray workers.
- **CN:** 分布式执行：代码会在多个进程、设备或 Ray worker 之间协调工作。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `typing`.
- **CN:** 标准库依赖：`argparse`, `typing`。
- **EN:** Third-party packages: `ray`, `torch`, `transformers`.
- **CN:** 第三方依赖：`ray`, `torch`, `transformers`。
- **EN:** Internal modules: `vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.fused_moe`, `vllm.model_executor.layers.fused_moe.moe_permute_unpermute`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。
