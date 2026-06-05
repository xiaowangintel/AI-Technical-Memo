# benchmark_lora.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_lora.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, LoRA-related kernel experiments, centered around `DEFAULT_MODELS`, `DEFAULT_TP_SIZES`, `DEFAULT_BATCH_SIZES`, `DEFAULT_HIDDEN_SIZES`. / 实现与基准测试编排、LoRA 相关内核实验相关的逻辑，核心符号包括 `DEFAULT_MODELS`, `DEFAULT_TP_SIZES`, `DEFAULT_BATCH_SIZES`, `DEFAULT_HIDDEN_SIZES`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-23)
```python
import argparse
import copy
import json
import pickle
import time
from collections.abc import Callable
from dataclasses import dataclass
from enum import Enum, auto
from itertools import product
from pathlib import Path
from typing import Any

import torch
import torch.utils.benchmark as TBenchmark
from torch.utils.benchmark import Measurement as TMeasurement
from utils import ArgPool, Bench, CudaGraphBenchParams
from weight_shapes import WEIGHT_SHAPES

from vllm.lora.ops.triton_ops.utils import get_lora_op_configs
from vllm.triton_utils import HAS_TRITON, triton
```
**EN:** This block gathers standard-library helpers such as `argparse`, `copy`, `json`, `pickle`, `time`, `collections.abc`; third-party packages such as `torch`, `torch.utils.benchmark`; project-local modules such as `utils`, `weight_shapes`, `vllm.lora.ops.triton_ops.utils`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `copy`, `json`, `pickle`, `time`, `collections.abc`；第三方依赖，如 `torch`, `torch.utils.benchmark`；项目内部模块，如 `utils`, `weight_shapes`, `vllm.lora.ops.triton_ops.utils`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 25-36)
```python
if HAS_TRITON:
    from vllm.lora.ops.triton_ops import (  ## added fused_moe_lora
        LoRAKernelMeta,
        fused_moe_lora_expand,
        fused_moe_lora_shrink,
        lora_expand,
        lora_shrink,
    )
    from vllm.lora.ops.triton_ops.fused_moe_lora_op import (
        _LORA_PTR_DICT,  ## added _LORA_PTR_DICT for fused_moe_lora
    )
    from vllm.lora.ops.triton_ops.utils import _LORA_A_PTR_DICT, _LORA_B_PTR_DICT
```
**EN:** This top-level block prepares shared state such as module-level state. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 module-level state。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Additional imports (lines 37-37)
```python
from vllm import _custom_ops as ops
```
**EN:** This block gathers project-local modules such as `vllm`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 38-38)
```python
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers project-local modules such as `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 39-39)
```python
from vllm.utils.math_utils import round_up
```
**EN:** This block gathers project-local modules such as `vllm.utils.math_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.utils.math_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 41-74)
```python
DEFAULT_MODELS = list(WEIGHT_SHAPES.keys())
DEFAULT_TP_SIZES = [1]
DEFAULT_BATCH_SIZES = [
    1,
    16,
    32,
    64,
    128,
    192,
    256,
    320,
    384,
    448,
    512,
    640,
    768,
    896,
    1024,
    2048,
    3072,
    4096,
    5120,
    6144,
    7168,
    8192,
]
DEFAULT_HIDDEN_SIZES = [1024, 2048, 4096, 8192, 16384]
DEFAULT_LORA_RANKS = [16]
DEFAULT_NUM_LORAS = [1, 2, 3, 4]
DEFAULT_SORT_BY_LORA_IDS = [False, True]
DEFAULT_SEQ_LENGTHS = [1]
DEFAULT_EXPAND_FN_ADD_INPUTS = [True, False]
DEFAULT_TOP_K_NUMS = [1]  # Added for MoE LoRA top_k
DEFAULT_NUM_EXPERTS = [8]  # Added for MoE LoRA num_experts
```
**EN:** This top-level block prepares shared state such as `DEFAULT_MODELS`, `DEFAULT_TP_SIZES`, `DEFAULT_BATCH_SIZES`, `DEFAULT_HIDDEN_SIZES`, `DEFAULT_LORA_RANKS`, `DEFAULT_NUM_LORAS`. It uses `list`, `WEIGHT_SHAPES.keys` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `DEFAULT_MODELS`, `DEFAULT_TP_SIZES`, `DEFAULT_BATCH_SIZES`, `DEFAULT_HIDDEN_SIZES`, `DEFAULT_LORA_RANKS`, `DEFAULT_NUM_LORAS`。它借助 `list`, `WEIGHT_SHAPES.keys` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `dtype_to_str` (lines 78-85)
```python
def dtype_to_str(dtype: torch.dtype):
    if dtype == torch.float16:
        return "f16"
    if dtype == torch.bfloat16:
        return "bf16"
    if dtype == torch.float32:
        return "f32"
    raise ValueError(f"Unsupported dtype {dtype}")
```
**EN:** `dtype_to_str` implements a helper used by `benchmark_lora.py`. It mainly works with `dtype` and relies on `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `dtype_to_str` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `dtype`，并结合 `ValueError` 以及 条件分支 来完成这一段基准测试流程。

### Function `make_rand_lora_weight_tensor` (lines 88-92)
```python
def make_rand_lora_weight_tensor(
    k: int, n: int, num_loras: int, dtype: torch.dtype, device: str = "cuda"
) -> torch.Tensor:
    # LoRA weights column major
    return torch.rand((num_loras, n, k), dtype=dtype).to(device)
```
**EN:** `make_rand_lora_weight_tensor` implements LoRA-related kernel experiments. It mainly works with `k`, `n`, `num_loras`, `dtype`, `device` and relies on `torch.rand.to`, `torch.rand` plus value production to move data through this part of the benchmark pipeline.
**CN:** `make_rand_lora_weight_tensor` 负责实现LoRA 相关内核实验。 它主要处理 `k`, `n`, `num_loras`, `dtype`, `device`，并结合 `torch.rand.to`, `torch.rand` 以及 结果返回 来完成这一段基准测试流程。

### Function `make_rand_tensors` (lines 95-114)
```python
def make_rand_tensors(
    a_shape: tuple[int, ...],
    b_shape: tuple[int, ...],
    c_shape: tuple[int, ...],
    a_dtype: torch.dtype,
    b_dtype: torch.dtype,
    c_dtype: torch.dtype,
    num_slices: int,
    device: str = "cuda",
) -> tuple[torch.Tensor, list[torch.Tensor], torch.Tensor]:
    """
    Make LoRA input/output matrices.
    """
    A = torch.rand(a_shape, dtype=a_dtype).to(device)

    # LoRA weights column major
    Bs = [torch.rand(b_shape, dtype=b_dtype).to(device) for _ in range(num_slices)]

    C = torch.zeros(c_shape, dtype=c_dtype).to(device)
    return A, Bs, C
```
**EN:** `make_rand_tensors` Make LoRA input/output matrices. It mainly works with `a_shape`, `b_shape`, `c_shape`, `a_dtype`, `b_dtype`, ... and relies on `torch.rand.to`, `torch.rand`, `range`, `torch.zeros.to`, `torch.zeros` plus value production to move data through this part of the benchmark pipeline.
**CN:** `make_rand_tensors` 的职责是：Make LoRA input/output matrices。 它主要处理 `a_shape`, `b_shape`, `c_shape`, `a_dtype`, `b_dtype`, ...，并结合 `torch.rand.to`, `torch.rand`, `range`, `torch.zeros.to`, `torch.zeros` 以及 结果返回 来完成这一段基准测试流程。

### Function `make_prompt_lora_mapping` (lines 117-140)
```python
def make_prompt_lora_mapping(
    num_prompts: int, num_active_loras: int, sort_by_lora_id: bool, device: str
) -> torch.Tensor:
    """
    All prompts are mapped to a LoRA ID in range [0, num_active_loras).
    where 0 refers to first lora, 1 refers to second lora and so on.
    """
    assert num_active_loras > 0

    if not sort_by_lora_id:
        return torch.randint(0, num_active_loras, (num_prompts,), dtype=torch.long)

    # Divide LoRAs equally and in order.
    part_size = num_prompts // num_active_loras
    part_size = max(part_size, 1)

    lora_id = 0
    prompt_lora_mapping = []
    while len(prompt_lora_mapping) < num_prompts:
        prompt_lora_mapping.extend([lora_id] * part_size)
        lora_id = lora_id + 1 if lora_id + 1 < num_active_loras else lora_id
    return torch.tensor(
        prompt_lora_mapping[:num_prompts], dtype=torch.long, device=device
    )
```
**EN:** `make_prompt_lora_mapping` All prompts are mapped to a LoRA ID in range [0, num_active_loras). It mainly works with `num_prompts`, `num_active_loras`, `sort_by_lora_id`, `device` and relies on `torch.randint`, `max`, `len`, `prompt_lora_mapping.extend`, `torch.tensor` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `make_prompt_lora_mapping` 的职责是：All prompts are mapped to a LoRA ID in range [0, num_active_loras)。 它主要处理 `num_prompts`, `num_active_loras`, `sort_by_lora_id`, `device`，并结合 `torch.randint`, `max`, `len`, `prompt_lora_mapping.extend`, `torch.tensor` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `make_token_lora_mapping` (lines 143-165)
```python
def make_token_lora_mapping(
    num_tokens: int,
    num_prompts: int,
    prompt_lora_mapping: torch.Tensor,
    seq_len_tensor: torch.Tensor,
    device: str,
):
    """
    Make token_lora_mapping from prompt_lora_mapping and seq_lens_tensor
    """
    assert prompt_lora_mapping.shape[0] == num_prompts

    # token to lora index mapping
    token_lora_mapping = [0] * num_tokens
    current_offset = 0
    for b_id in range(num_prompts):
        lora_index = prompt_lora_mapping[b_id].item()
        s = current_offset
        e = s + seq_len_tensor[b_id].item()
        token_lora_mapping[s:e] = [lora_index] * (e - s)
        current_offset += seq_len_tensor[b_id].item()

    return torch.tensor(token_lora_mapping, dtype=torch.long, device=device)
```
**EN:** `make_token_lora_mapping` Make token_lora_mapping from prompt_lora_mapping and seq_lens_tensor. It mainly works with `num_tokens`, `num_prompts`, `prompt_lora_mapping`, `seq_len_tensor`, `device` and relies on `range`, `prompt_lora_mapping.item`, `seq_len_tensor.item`, `torch.tensor` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `make_token_lora_mapping` 的职责是：Make token_lora_mapping from prompt_lora_mapping and seq_lens_tensor。 它主要处理 `num_tokens`, `num_prompts`, `prompt_lora_mapping`, `seq_len_tensor`, `device`，并结合 `range`, `prompt_lora_mapping.item`, `seq_len_tensor.item`, `torch.tensor` 以及 循环迭代 来完成这一段基准测试流程。

### Function `ref_group_gemm` (lines 168-197)
```python
def ref_group_gemm(
    ref_out: torch.Tensor,
    input: torch.Tensor,
    lora_weights: list[torch.Tensor],
    seq_lens_cpu: torch.Tensor,
    prompt_lora_mapping_cpu: torch.Tensor,
    scaling: float,
    add_inputs: bool | None,
):
    """
    Torch group gemm reference implementation to test correctness of
    benchmarking operations.
    """
    batches = seq_lens_cpu.size(0)
    out_list = []
    current_offset = 0
    for lora_index, b_length in zip(range(batches), seq_lens_cpu):
        x = input[current_offset : b_length + current_offset, :]
        current_offset += b_length
        w = lora_weights[prompt_lora_mapping_cpu[lora_index]]
        result = torch.nn.functional.linear(x, w)
        result *= scaling
        out_list.append(result)

    cat_result = torch.cat(out_list, dim=0)

    if add_inputs:
        ref_out += cat_result
    else:
        ref_out.copy_(cat_result)
```
**EN:** `ref_group_gemm` Torch group gemm reference implementation to test correctness of benchmarking operations. It mainly works with `ref_out`, `input`, `lora_weights`, `seq_lens_cpu`, `prompt_lora_mapping_cpu`, ... and relies on `seq_lens_cpu.size`, `zip`, `range`, `torch.nn.functional.linear`, `out_list.append`, `torch.cat` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `ref_group_gemm` 的职责是：Torch group gemm reference implementation to test correctness of benchmarking operations。 它主要处理 `ref_out`, `input`, `lora_weights`, `seq_lens_cpu`, `prompt_lora_mapping_cpu`, ...，并结合 `seq_lens_cpu.size`, `zip`, `range`, `torch.nn.functional.linear`, `out_list.append`, `torch.cat` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Class `OpType` (lines 200-417)
```python
class OpType(Enum):
    """
    LoRA Ops to benchmark and its properties.
    """

    LORA_SHRINK = auto()
    LORA_EXPAND = auto()
    ## Adding support for fused moe lora
    FUSED_MOE_LORA_GATE_UP_SHRINK = auto()  ## Gate/Up projection variant with shrink
    FUSED_MOE_LORA_GATE_UP_EXPAND = auto()  ## Gate/Up projection variant with expand
    FUSED_MOE_LORA_DOWN_SHRINK = auto()  ## Down projection variant with shrink
    FUSED_MOE_LORA_DOWN_EXPAND = auto()  ## Down projection variant with expand

    @staticmethod
    def from_str(s: str) -> "OpType":
        if s.lower() == "lora_shrink":
            return OpType.LORA_SHRINK
        if s.lower() == "lora_expand":
            return OpType.LORA_EXPAND
        # Adding support for fused moe lora, both in gate_up and down
        if s.lower() == "fused_moe_lora_gate_up_shrink":  ## Gate/Up variant with shrink
            return OpType.FUSED_MOE_LORA_GATE_UP_SHRINK
        if s.lower() == "fused_moe_lora_gate_up_expand":  ## Gate/Up variant with expand
            return OpType.FUSED_MOE_LORA_GATE_UP_EXPAND
        if s.lower() == "fused_moe_lora_down_shrink":  ## Down variant with shrink
            return OpType.FUSED_MOE_LORA_DOWN_SHRINK
        if s.lower() == "fused_moe_lora_down_expand":  ## Down variant with expand
            return OpType.FUSED_MOE_LORA_DOWN_EXPAND
        raise ValueError(f"Unrecognized str {s} to convert to OpType")

    def is_shrink_fn(self) -> bool:
        return self in [OpType.LORA_SHRINK]

    def is_expand_fn(self) -> bool:
    # ... omitted for brevity ...
                ref_group_gemm(
                    ref_out=output[:, slice_offset : slice_offset + hidden_size],
                    input=input[slice_idx].clone().to(dtype=w_dtype),
                    lora_weights=lora_weights[slice_idx],
                    **kwargs,
                )
        else:
            raise ValueError(f"Unrecognized optype {self}")
```
**EN:** Class `OpType` is the main object-oriented wrapper for this module. Its docstring says: LoRA Ops to benchmark and its properties. It extends `Enum` and organizes behavior through `from_str`, `is_shrink_fn`, `is_expand_fn`, `is_fused_moe_lora_fn`, `is_fused_moe_lora_gate_up_fn`, `is_fused_moe_lora_down_fn`.
**CN:** 类 `OpType` 是该模块中的主要面向对象封装。文档字符串指出：LoRA Ops to benchmark and its properties。它继承自 `Enum`，并通过 `from_str`, `is_shrink_fn`, `is_expand_fn`, `is_fused_moe_lora_fn`, `is_fused_moe_lora_gate_up_fn`, `is_fused_moe_lora_down_fn` 组织行为。

### Method `OpType.from_str` (lines 214-228)
```python
    def from_str(s: str) -> "OpType":
        if s.lower() == "lora_shrink":
            return OpType.LORA_SHRINK
        if s.lower() == "lora_expand":
            return OpType.LORA_EXPAND
        # Adding support for fused moe lora, both in gate_up and down
        if s.lower() == "fused_moe_lora_gate_up_shrink":  ## Gate/Up variant with shrink
            return OpType.FUSED_MOE_LORA_GATE_UP_SHRINK
        if s.lower() == "fused_moe_lora_gate_up_expand":  ## Gate/Up variant with expand
            return OpType.FUSED_MOE_LORA_GATE_UP_EXPAND
        if s.lower() == "fused_moe_lora_down_shrink":  ## Down variant with shrink
            return OpType.FUSED_MOE_LORA_DOWN_SHRINK
        if s.lower() == "fused_moe_lora_down_expand":  ## Down variant with expand
            return OpType.FUSED_MOE_LORA_DOWN_EXPAND
        raise ValueError(f"Unrecognized str {s} to convert to OpType")
```
**EN:** `from_str` implements a helper used by `benchmark_lora.py`. It mainly works with `s` and relies on `s.lower`, `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `from_str` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `s`，并结合 `s.lower`, `ValueError` 以及 条件分支 来完成这一段基准测试流程。

### Method `OpType.is_shrink_fn` (lines 230-231)
```python
    def is_shrink_fn(self) -> bool:
        return self in [OpType.LORA_SHRINK]
```
**EN:** `is_shrink_fn` implements a helper used by `benchmark_lora.py`. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `is_shrink_fn` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `OpType.is_expand_fn` (lines 233-234)
```python
    def is_expand_fn(self) -> bool:
        return self in [OpType.LORA_EXPAND]
```
**EN:** `is_expand_fn` implements a helper used by `benchmark_lora.py`. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `is_expand_fn` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `OpType.is_fused_moe_lora_fn` (lines 236-242)
```python
    def is_fused_moe_lora_fn(self) -> bool:  ## adding for fused MoE LoRA
        return self in [
            OpType.FUSED_MOE_LORA_GATE_UP_SHRINK,
            OpType.FUSED_MOE_LORA_DOWN_SHRINK,
            OpType.FUSED_MOE_LORA_GATE_UP_EXPAND,
            OpType.FUSED_MOE_LORA_DOWN_EXPAND,
        ]
```
**EN:** `is_fused_moe_lora_fn` implements LoRA-related kernel experiments. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `is_fused_moe_lora_fn` 负责实现LoRA 相关内核实验。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Class `BenchmarkContext` (lines 421-465)
```python
class BenchmarkContext:
    """
    LoRA benchmark context
    """

    batch_size: int
    hidden_size: int
    num_loras: int
    num_active_loras: int
    lora_rank: int
    sort_by_lora_id: bool
    dtype: torch.dtype
    seq_length: int | None = None
    num_experts: int | None = None  # num_experts for MoE based ops
    top_k_num: int | None = None  # top_k for MoE based ops
    num_slices: int | None = None  # num_slices for slice based ops

    def with_seq_length(self, seq_length: int) -> "BenchmarkContext":
        ctx = copy.copy(self)
        ctx.seq_length = seq_length
        return ctx

    def with_num_slices(self, num_slices: int) -> "BenchmarkContext":
        ctx = copy.copy(self)
        ctx.num_slices = num_slices
        return ctx

    def bench_label(self) -> str:
        return f"lora-{self.dtype}"

    def bench_sublabel(self, op_type: OpType) -> str:
        m, k, n = op_type.mkn(
            self.batch_size, self.seq_length, self.hidden_size, self.lora_rank
        )
        desc = {
            "bs": self.batch_size,
            "sl": self.seq_length,
            "m": m,
            "k": k,
            "n": n,
            "num_loras": self.num_loras,
            "sort_by_lora": self.sort_by_lora_id,
            "num_slices": self.num_slices,
        }
        return json.dumps(desc)
```
**EN:** Class `BenchmarkContext` is the main object-oriented wrapper for this module. Its docstring says: LoRA benchmark context. It extends `object` and organizes behavior through `with_seq_length`, `with_num_slices`, `bench_label`, `bench_sublabel`.
**CN:** 类 `BenchmarkContext` 是该模块中的主要面向对象封装。文档字符串指出：LoRA benchmark context。它继承自 `object`，并通过 `with_seq_length`, `with_num_slices`, `bench_label`, `bench_sublabel` 组织行为。

### Method `BenchmarkContext.with_seq_length` (lines 438-441)
```python
    def with_seq_length(self, seq_length: int) -> "BenchmarkContext":
        ctx = copy.copy(self)
        ctx.seq_length = seq_length
        return ctx
```
**EN:** `with_seq_length` implements a helper used by `benchmark_lora.py`. It mainly works with `seq_length` and relies on `copy.copy` plus value production to move data through this part of the benchmark pipeline.
**CN:** `with_seq_length` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `seq_length`，并结合 `copy.copy` 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkContext.with_num_slices` (lines 443-446)
```python
    def with_num_slices(self, num_slices: int) -> "BenchmarkContext":
        ctx = copy.copy(self)
        ctx.num_slices = num_slices
        return ctx
```
**EN:** `with_num_slices` implements a helper used by `benchmark_lora.py`. It mainly works with `num_slices` and relies on `copy.copy` plus value production to move data through this part of the benchmark pipeline.
**CN:** `with_num_slices` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `num_slices`，并结合 `copy.copy` 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkContext.bench_label` (lines 448-449)
```python
    def bench_label(self) -> str:
        return f"lora-{self.dtype}"
```
**EN:** `bench_label` implements a helper used by `benchmark_lora.py`. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `bench_label` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkContext.bench_sublabel` (lines 451-465)
```python
    def bench_sublabel(self, op_type: OpType) -> str:
        m, k, n = op_type.mkn(
            self.batch_size, self.seq_length, self.hidden_size, self.lora_rank
        )
        desc = {
            "bs": self.batch_size,
            "sl": self.seq_length,
            "m": m,
            "k": k,
            "n": n,
            "num_loras": self.num_loras,
            "sort_by_lora": self.sort_by_lora_id,
            "num_slices": self.num_slices,
        }
        return json.dumps(desc)
```
**EN:** `bench_sublabel` implements a helper used by `benchmark_lora.py`. It mainly works with `op_type` and relies on `op_type.mkn`, `json.dumps` plus value production to move data through this part of the benchmark pipeline.
**CN:** `bench_sublabel` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `op_type`，并结合 `op_type.mkn`, `json.dumps` 以及 结果返回 来完成这一段基准测试流程。

### Class `BenchmarkTensors` (lines 469-989)
```python
class BenchmarkTensors:
    """
    Input/Output tensors used for benchmarks
    """

    # matmul tensors
    input: torch.Tensor
    lora_weights_lst: list[torch.Tensor]
    output: torch.Tensor
    # LoRA kernel metadata
    lora_kernel_meta: LoRAKernelMeta
    # Metadata tensors used in testing correctness
    seq_lens: torch.Tensor
    prompt_lora_mapping: torch.Tensor

    def io_types(self) -> str:
        return (
            f"{dtype_to_str(self.input.dtype)}x"
            f"{dtype_to_str(self.lora_weights_lst[0].dtype)}=>"
            f"{dtype_to_str(self.output.dtype)}"
        )

    def get_num_tokens(self, size: int, top_k_num: int, op_type: OpType):
        return (
            size * top_k_num if op_type in [OpType.FUSED_MOE_LORA_DOWN_SHRINK] else size
        )

    @staticmethod
    def make(
        ctx: BenchmarkContext, op_type: OpType, device: str = "cuda"
    ) -> "BenchmarkTensors":
        # Make input / output matmul tensors.
        a_shape, b_shape, c_shape = op_type.matmul_shapes(
            ctx.batch_size,
    # ... omitted for brevity ...

        rtol, atol = {
            torch.float16: (6e-2, 6e-2),
            torch.bfloat16: (6e-2, 6e-2),
            torch.float32: (1e-2, 1e-2),
        }[self.output.dtype]

        return torch.allclose(ref_output, self.output, rtol=rtol, atol=atol)
```
**EN:** Class `BenchmarkTensors` is the main object-oriented wrapper for this module. Its docstring says: Input/Output tensors used for benchmarks. It extends `object` and organizes behavior through `io_types`, `get_num_tokens`, `make`, `sanity_check`, `to_device`, `metadata`.
**CN:** 类 `BenchmarkTensors` 是该模块中的主要面向对象封装。文档字符串指出：Input/Output tensors used for benchmarks。它继承自 `object`，并通过 `io_types`, `get_num_tokens`, `make`, `sanity_check`, `to_device`, `metadata` 组织行为。

### Method `BenchmarkTensors.io_types` (lines 484-489)
```python
    def io_types(self) -> str:
        return (
            f"{dtype_to_str(self.input.dtype)}x"
            f"{dtype_to_str(self.lora_weights_lst[0].dtype)}=>"
            f"{dtype_to_str(self.output.dtype)}"
        )
```
**EN:** `io_types` implements a helper used by `benchmark_lora.py`. It mainly works with no explicit parameters and relies on `dtype_to_str` plus value production to move data through this part of the benchmark pipeline.
**CN:** `io_types` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `dtype_to_str` 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkTensors.get_num_tokens` (lines 491-494)
```python
    def get_num_tokens(self, size: int, top_k_num: int, op_type: OpType):
        return (
            size * top_k_num if op_type in [OpType.FUSED_MOE_LORA_DOWN_SHRINK] else size
        )
```
**EN:** `get_num_tokens` implements a helper used by `benchmark_lora.py`. It mainly works with `size`, `top_k_num`, `op_type` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_num_tokens` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `size`, `top_k_num`, `op_type`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkTensors.make` (lines 497-555)
```python
    def make(
        ctx: BenchmarkContext, op_type: OpType, device: str = "cuda"
    ) -> "BenchmarkTensors":
        # Make input / output matmul tensors.
        a_shape, b_shape, c_shape = op_type.matmul_shapes(
            ctx.batch_size,
            ctx.seq_length,
            ctx.hidden_size,
            ctx.lora_rank,
            ctx.num_loras,
            ctx.num_slices,
            ctx.top_k_num,
            ctx.num_experts,
        )
        a_type, b_type, c_type = op_type.matmul_dtypes(ctx.dtype)
        input_tensor, lora_weights, output_tensor = make_rand_tensors(
            a_shape, b_shape, c_shape, a_type, b_type, c_type, num_slices=ctx.num_slices
        )

        # Make metadata tensors.
        # Keep the metadata tensors in the CPU for further processing if needed.
        # The tensors get moved to the GPU before benchmarking.
        assert ctx.num_active_loras <= ctx.num_loras
        total_tokens = ctx.batch_size * ctx.seq_length

        # Make metadata tensors involved in correctness testing.
        # Prepare seq lens tensor
        seq_len_tensor = torch.randint(
            ctx.seq_length, ctx.seq_length + 1, (ctx.batch_size,)
        )
        assert total_tokens == seq_len_tensor.sum()
        # Prepare prompt lora indices tensor
        prompt_lora_indices_tensor = make_prompt_lora_mapping(
            ctx.batch_size, ctx.num_active_loras, ctx.sort_by_lora_id, "cpu"
        # ... omitted for brevity ...
        return BenchmarkTensors(
            input_tensor,
            lora_weights,
            output_tensor,
            lora_kernel_meta,
            seq_len_tensor,
            prompt_lora_indices_tensor,
        )
```
**EN:** `make` implements a helper used by `benchmark_lora.py`. It mainly works with `ctx`, `op_type`, `device` and relies on `op_type.matmul_shapes`, `op_type.matmul_dtypes`, `make_rand_tensors`, `torch.randint`, `seq_len_tensor.sum`, `make_prompt_lora_mapping` plus value production to move data through this part of the benchmark pipeline.
**CN:** `make` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `ctx`, `op_type`, `device`，并结合 `op_type.matmul_shapes`, `op_type.matmul_dtypes`, `make_rand_tensors`, `torch.randint`, `seq_len_tensor.sum`, `make_prompt_lora_mapping` 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkTensors.sanity_check` (lines 557-577)
```python
    def sanity_check(self, ctx: BenchmarkContext, op_type: OpType) -> None:
        """
        Fails asserts when non-conformality is detected.
        """
        num_tokens = (
            self.input.shape[1]
            if op_type.is_fused_moe_lora_expand_fn()
            else self.input.shape[-2]
        )
        # check metadata tensors
        ## In down shrink case, each token is repeated top_k_num times
        assert num_tokens == self.get_num_tokens(
            torch.sum(self.seq_lens), ctx.top_k_num, op_type
        ), f"Expected {num_tokens} tokens, but got {torch.sum(self.seq_lens)}"
        num_seqs = self.seq_lens.shape[0]
        # assert self.seq_start_loc.shape[0] == num_seqs
        ## In down shrink case, each prompt corresponds to top_k_num sequences
        assert self.prompt_lora_mapping.shape[0] == num_seqs
        assert self.get_num_tokens(
            self.lora_kernel_meta.token_lora_mapping.shape[0], ctx.top_k_num, op_type
        )
```
**EN:** `sanity_check` Fails asserts when non-conformality is detected. It mainly works with `ctx`, `op_type` and relies on `op_type.is_fused_moe_lora_expand_fn`, `self.get_num_tokens`, `torch.sum` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `sanity_check` 的职责是：Fails asserts when non-conformality is detected。 它主要处理 `ctx`, `op_type`，并结合 `op_type.is_fused_moe_lora_expand_fn`, `self.get_num_tokens`, `torch.sum` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `bench_optype` (lines 992-1064)
```python
def bench_optype(
    ctx: BenchmarkContext,
    arg_pool_size: int,
    op_type: OpType,
    cuda_graph_nops: int | None = None,
    expand_fn_add_inputs: bool | None = None,
    test_correctness: bool = False,
) -> TMeasurement:
    assert arg_pool_size >= 1
    if op_type.is_shrink_fn() or op_type.is_fused_moe_lora_fn():
        assert expand_fn_add_inputs is None
    else:
        assert expand_fn_add_inputs is not None

    # BenchmarkContext -> BenchmarkTensors
    bench_tensors: list[BenchmarkTensors] = [
        BenchmarkTensors.make(ctx, op_type) for _ in range(arg_pool_size)
    ]
    for bt in bench_tensors:
        bt.sanity_check(ctx, op_type)

    # Test correctness of our implementation.
    if test_correctness:
        assert op_type in [OpType.LORA_SHRINK, OpType.LORA_EXPAND], (
            f"Correctness testing is not supported for {op_type.name}."
        )
        assert all(
            [
                bt.test_correctness(ctx, op_type, expand_fn_add_inputs)
                for bt in bench_tensors
            ]
        )

    # BenchmarkTensors -> dict (kwargs)
    # ... omitted for brevity ...
        ctx.bench_label(),
        ctx.bench_sublabel(op_type),
        description,
        op_type.bench_fn(),
        **kwargs,
    ) as bench:
        timer = bench.run()
    return timer
```
**EN:** `bench_optype` implements a helper used by `benchmark_lora.py`. It mainly works with `ctx`, `arg_pool_size`, `op_type`, `cuda_graph_nops`, `expand_fn_add_inputs`, ... and relies on `op_type.is_shrink_fn`, `op_type.is_fused_moe_lora_fn`, `BenchmarkTensors.make`, `range`, `bt.sanity_check`, `all` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `bench_optype` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `ctx`, `arg_pool_size`, `op_type`, `cuda_graph_nops`, `expand_fn_add_inputs`, ...，并结合 `op_type.is_shrink_fn`, `op_type.is_fused_moe_lora_fn`, `BenchmarkTensors.make`, `range`, `bt.sanity_check`, `all` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `bench_torch_mm` (lines 1067-1120)
```python
def bench_torch_mm(
    ctx: BenchmarkContext,
    arg_pool_size: int,
    op_type: OpType,
    cuda_graph_nops: int | None = None,
) -> TMeasurement:
    """
    Benchmark basic torch.mm as a roofline.

    When all the input tokens have the same LoRA ID, the LoRA kernels are just
    a matmul. This torch.mm benchmark serves as a roofline for that case.

    input op_type is used in determining the m, k, n dimensions for the matmul.
    """

    batch_size, hidden_size, lora_rank, seq_length, dtype = (
        ctx.batch_size,
        ctx.hidden_size,
        ctx.lora_rank,
        ctx.seq_length,
        ctx.dtype,
    )

    m, k, n = op_type.mkn(batch_size, seq_length, hidden_size, lora_rank)
    # For a fairer comparison.
    n = n * ctx.num_slices

    # Get matmul input and output tensors for A x B = C
    As, Bs, Cs = [], [], []
    for _ in range(arg_pool_size):
        As.append(torch.rand((m, k), dtype=dtype).to("cuda"))
        Bs.append(torch.rand((n, k), dtype=dtype).to("cuda").t())
        Cs.append(torch.rand((m, n), dtype=dtype).to("cuda"))

    # ... omitted for brevity ...
        cuda_graph_params,
        ctx.bench_label(),
        ctx.bench_sublabel(op_type),
        description,
        torch.mm,
        **mm_kwargs,
    ) as bench:
        return bench.run()
```
**EN:** `bench_torch_mm` Benchmark basic torch.mm as a roofline. It mainly works with `ctx`, `arg_pool_size`, `op_type`, `cuda_graph_nops` and relies on `op_type.mkn`, `range`, `As.append`, `torch.rand.to`, `torch.rand`, `Bs.append` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `bench_torch_mm` 的职责是：Benchmark basic torch.mm as a roofline。 它主要处理 `ctx`, `arg_pool_size`, `op_type`, `cuda_graph_nops`，并结合 `op_type.mkn`, `range`, `As.append`, `torch.rand.to`, `torch.rand`, `Bs.append` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `use_cuda_graph_recommendation` (lines 1124-1133)
```python
def use_cuda_graph_recommendation() -> str:
    return """
            Triton kernels have a significant launch overhead with
            launched directly via python. This overhead is more noticeable
            for small the problem sizes. For these cases, it is recommended
            to use the script with `--cuda-graph-nops N` to benchmark N
            consecutive invocations of the benchmarking operations from 
            inside a CUDA Graph. Note that the returned measurement is for N 
            invocations of the operation.
            """
```
**EN:** `use_cuda_graph_recommendation` implements multimodal processing benchmarks. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `use_cuda_graph_recommendation` 负责实现多模态处理基准。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `print_timers` (lines 1136-1155)
```python
def print_timers(timers: list[TMeasurement], args: argparse.Namespace | None = None):
    compare = TBenchmark.Compare(timers)
    compare.print()

    if args and args.cuda_graph_nops:
        print(
            f"Note : The timings reported above is for {args.cuda_graph_nops} "
            "consecutive invocations of the benchmarking functions. "
            f"Please divide by {args.cuda_graph_nops} for single invocation "
            "timings."
        )

    print(
        "Note on Comparison with torch.mm : The torch.mm numbers are "
        "benchmark numbers of a simple matmul emulating the single lora "
        "case. It is provided as a roofline for comparing our LoRA Kernel "
        "implementations. It is expected that the LoRA kernels will be "
        "slower than torch.mm in cases where num_loras is big. But for "
        "small num_loras the goal should be to match the torch.mm numbers."
    )
```
**EN:** `print_timers` formats results for display or export. It mainly works with `timers`, `args` and relies on `TBenchmark.Compare`, `compare.print`, `print` plus branching to move data through this part of the benchmark pipeline.
**CN:** `print_timers` 负责格式化结果以便展示或导出。 它主要处理 `timers`, `args`，并结合 `TBenchmark.Compare`, `compare.print`, `print` 以及 条件分支 来完成这一段基准测试流程。

### Function `run` (lines 1158-1217)
```python
def run(args: argparse.Namespace, bench_ctxs: list[BenchmarkContext]):
    if args.cuda_graph_nops is not None:
        assert args.cuda_graph_nops > 0
        print(f"Benchmarking {args.cuda_graph_nops} invocations inside a CUDA Graph")
    else:
        print(f"CUDA Graphs not enabled.\n{use_cuda_graph_recommendation()}")

    timers = []
    for bench_ctx in bench_ctxs:
        for seq_len in args.seq_lengths:
            bench_ops: list[OpType] = args.op_types
            seq_len_timers = []
            for bench_op in bench_ops:
                for num_slices in bench_op.num_slices():
                    _ctx = bench_ctx.with_seq_length(seq_len).with_num_slices(
                        num_slices
                    )
                    # Benchmark torch.mm as a roofline
                    seq_len_timers.append(
                        bench_torch_mm(
                            _ctx, args.arg_pool_size, bench_op, args.cuda_graph_nops
                        )
                    )

                    # Benchmark bench_op
                    expand_fn_add_inputs = (
                        [None]
                        if bench_op.is_shrink_fn() or bench_op.is_fused_moe_lora_fn()
                        else args.expand_fn_add_inputs
                    )
                    for add_input_arg in expand_fn_add_inputs:
                        seq_len_timers.append(
                            bench_optype(
                                _ctx,
    # ... omitted for brevity ...
        if not od.exists():
            od.mkdir()

        timestamp = int(time.time())
        pkl_file = od / f"lora_bench-{timestamp}.pkl"
        print(f"Writing benchmarks to {pkl_file}")
        with open(pkl_file, "wb") as f:
            pickle.dump(timers, f)
```
**EN:** `run` implements a helper used by `benchmark_lora.py`. It mainly works with `args`, `bench_ctxs` and relies on `print`, `use_cuda_graph_recommendation`, `bench_op.num_slices`, `bench_ctx.with_seq_length.with_num_slices`, `bench_ctx.with_seq_length`, `seq_len_timers.append` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `run` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `args`, `bench_ctxs`，并结合 `print`, `use_cuda_graph_recommendation`, `bench_op.num_slices`, `bench_ctx.with_seq_length.with_num_slices`, `bench_ctx.with_seq_length`, `seq_len_timers.append` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `as_benchmark_contexts` (lines 1220-1261)
```python
def as_benchmark_contexts(
    hidden_sizes: list[int], lora_ranks: list[int], args: argparse.Namespace
) -> list[BenchmarkContext]:
    ctxs: list[BenchmarkContext] = []
    for (
        batch_size,
        hidden_size,
        lora_rank,
        num_loras,
        sort_by_lora_id,
        top_k_num,
        num_experts,
    ) in product(  # noqa
        args.batch_sizes,
        list(hidden_sizes),
        lora_ranks,
        args.num_loras,
        args.sort_by_lora_id,
        args.top_k_nums,
        args.num_experts,
    ):
        ctxs.append(
            BenchmarkContext(
                batch_size=batch_size,
                hidden_size=hidden_size,
                lora_rank=lora_rank,
                num_loras=num_loras,
                num_active_loras=args.num_active_loras
                if args.num_active_loras
                else num_loras,
                # To be filled based on the OpType to benchmark
                seq_length=None,
                sort_by_lora_id=sort_by_lora_id,
                dtype=args.dtype,
                top_k_num=top_k_num,
                num_experts=num_experts,
                # To be filled based on the OpType to benchmark
                num_slices=None,
            )
        )

    return ctxs
```
**EN:** `as_benchmark_contexts` coordinates or measures benchmark orchestration. It mainly works with `hidden_sizes`, `lora_ranks`, `args` and relies on `product`, `list`, `ctxs.append`, `BenchmarkContext` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `as_benchmark_contexts` 负责协调或测量基准测试编排。 它主要处理 `hidden_sizes`, `lora_ranks`, `args`，并结合 `product`, `list`, `ctxs.append`, `BenchmarkContext` 以及 循环迭代 来完成这一段基准测试流程。

### Function `run_list_bench` (lines 1264-1278)
```python
def run_list_bench(args: argparse.Namespace):
    print(args)

    print(
        "List bench :\n"
        f"  Hidden Sizes {args.hidden_sizes}"
        f"  LoRA Ranks {args.lora_ranks}"
    )

    # Get all benchmarking contexts
    bench_contexts: list[BenchmarkContext] = as_benchmark_contexts(
        hidden_sizes=args.hidden_sizes, lora_ranks=args.lora_ranks, args=args
    )

    run(args, bench_contexts)
```
**EN:** `run_list_bench` implements a helper used by `benchmark_lora.py`. It mainly works with `args` and relies on `print`, `as_benchmark_contexts`, `run` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `run_list_bench` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `print`, `as_benchmark_contexts`, `run` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run_range_bench` (lines 1281-1302)
```python
def run_range_bench(args: argparse.Namespace):
    print(args)

    hidden_sizes = list(
        range(
            args.hidden_sizes_start,
            args.hidden_sizes_end + 1,
            args.hidden_sizes_increment,
        )
    )
    lora_ranks = list(
        range(args.lora_ranks_start, args.lora_ranks_end + 1, args.lora_ranks_increment)
    )

    print(f"Range bench :\n Hidden Sizes {hidden_sizes} LoRA Ranks {lora_ranks}")

    # Get all benchmarking contexts
    bench_contexts: list[BenchmarkContext] = as_benchmark_contexts(
        hidden_sizes=hidden_sizes, lora_ranks=lora_ranks, args=args
    )

    run(args, bench_contexts)
```
**EN:** `run_range_bench` implements a helper used by `benchmark_lora.py`. It mainly works with `args` and relies on `print`, `list`, `range`, `as_benchmark_contexts`, `run` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `run_range_bench` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `print`, `list`, `range`, `as_benchmark_contexts`, `run` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run_model_bench` (lines 1305-1327)
```python
def run_model_bench(args: argparse.Namespace):
    print(args)

    def hidden_sizes_from_model(model: str, tp_size: int) -> set[int]:
        hidden_sizes = set()
        for KN, tp_split_dim in WEIGHT_SHAPES[model]:
            KN[tp_split_dim] = KN[tp_split_dim] // tp_size
            hidden_sizes.add(KN[1])
        return hidden_sizes

    # Get all hidden sizes
    hidden_sizes: set[int] = set()
    for model_name, tp_size in product(args.models, args.tp_sizes):
        hidden_sizes = hidden_sizes.union(hidden_sizes_from_model(model_name, tp_size))

    print(f"Model bench :\n Hidden Sizes {hidden_sizes} LoRA Ranks {args.lora_ranks}")

    # Get all benchmarking contexts
    bench_contexts: list[BenchmarkContext] = as_benchmark_contexts(
        hidden_sizes=hidden_sizes, lora_ranks=args.lora_ranks, args=args
    )

    run(args, bench_contexts)
```
**EN:** `run_model_bench` implements a helper used by `benchmark_lora.py`. It mainly works with `args` and relies on `print`, `set`, `hidden_sizes.add`, `product`, `hidden_sizes.union`, `hidden_sizes_from_model` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `run_model_bench` 负责实现 `benchmark_lora.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `print`, `set`, `hidden_sizes.add`, `product`, `hidden_sizes.union`, `hidden_sizes_from_model` 以及 循环迭代 来完成这一段基准测试流程。

### Entry point (lines 1330-1490)
```python
if __name__ == "__main__":

    def to_torch_dtype(dt):
        if dt == "torch.float16":
            return torch.float16
        if dt == "torch.bfloat16":
            return torch.bfloat16
        raise ValueError("unsupported dtype")

    def get_bool(s: str) -> bool:
        return s.lower() in ["true", "1"]

    def add_common_command_args(p: argparse.ArgumentParser):
        p.add_argument(
            "--dtype",
            type=to_torch_dtype,
            required=True,
            help="Available options are ['torch.float16', 'torch.bfloat16']",
        )

        p.add_argument(
            "--arg-pool-size",
            type=int,
            default=32,
            help="Run profiles with a pool of input/output/meta tensors instead"
            "of simply reusing the same tensors for all runs. A bigger arg-pool"
            "mitigates hardware caching effects during benchmarking.",
        )

        p.add_argument(
            "--cuda-graph-nops",
            type=int,
            help=(
                "when set profiling is done using cudagraph, "
# ... omitted for brevity ...
    model_parser.add_argument(
        "--lora-ranks", nargs="+", type=int, default=DEFAULT_LORA_RANKS
    )
    add_common_command_args(model_parser)
    model_parser.set_defaults(func=run_model_bench)

    args = parser.parse_args()
    args.func(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `ValueError`, `s.lower`, `p.add_argument`, `list`, `FlexibleArgumentParser`, `use_cuda_graph_recommendation` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `ValueError`, `s.lower`, `p.add_argument`, `list`, `FlexibleArgumentParser`, `use_cuda_graph_recommendation` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `copy`, `json`, `pickle`, `time`, `collections.abc`.
- **CN:** 标准库依赖：`argparse`, `copy`, `json`, `pickle`, `time`, `collections.abc`。
- **EN:** Third-party packages: `torch`, `torch.utils.benchmark`.
- **CN:** 第三方依赖：`torch`, `torch.utils.benchmark`。
- **EN:** Internal modules: `utils`, `weight_shapes`, `vllm.lora.ops.triton_ops.utils`, `vllm.triton_utils`, `vllm`, `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`utils`, `weight_shapes`, `vllm.lora.ops.triton_ops.utils`, `vllm.triton_utils`, `vllm`, `vllm.utils.argparse_utils`。
