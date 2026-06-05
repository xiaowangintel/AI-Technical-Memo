# cutlass_moe_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/cutlass_moe_params.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `CutlassMoEType` and `CutlassMoEParams` and connects them to backend-specific paths such as `CUTLASS` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合线性投影与 GEMM 后端集成。它提供了 `CutlassMoEType` 和 `CutlassMoEParams` 等符号，并把这些符号连接到 `CUTLASS` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module imports and dependency wiring
```python
from dataclasses import dataclass
from enum import Enum, auto
from typing import Optional

import torch
```
**EN:** This section prepares the module namespace. It imports `dataclasses.dataclass`, `enum.Enum`, `enum.auto`, `typing.Optional`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `dataclasses.dataclass`、`enum.Enum`、`enum.auto`、`typing.Optional` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 8-17: Class `CutlassMoEType` declaration and shared state
```python
class CutlassMoEType(Enum):
    """
    Enum for the different types of cutlass moe operations
    that are currently supported in SGLang.
    """

    BlockscaledFP8 = auto()
    BlockscaledFP4 = auto()
```
**EN:** This block introduces class `CutlassMoEType` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Enum for the different types of cutlass moe operations that are currently supported in SGLang.
**CN:** 该代码块引入类 `CutlassMoEType`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 18-94: Class `CutlassMoEParams` declaration and shared state
```python
@dataclass
class CutlassMoEParams:
    """
    Parameters for the cutlass moe operation.
    """

    #  Type as defined above
    cutlass_moe_type: CutlassMoEType

    # Strides for activations, weights and output in logical number of elements.
    # The activations & output stride is the number of elements to the next row.
    # The weights stride is the number of elements to the next row per expert.
    # For example, if the weight is [e, n, k], then the b_stride is a tensor of
    # shape [e] with each element being k. Similarly for activations, if the
    # shape is [m, k], then the a_stride has shape [e] with each value k.
    # Similarly for output, if the output is [m, n], then the c_stride is a
    # tensor of shape [e] with each element being k.

    # Note: cutlass_fp4_group_mm is designed to accept the strides of
    # activations and weights to be the same, so it is passed in as a single
    # tensor.
    # ab_strides_13: [e] dtype: int64 [Gemm 1: Activation / Weight strides]
    # ab_strides_2: [e] dtype: int64 [Gemm 2: Activation / Weight strides]
    # c_strides_13: [e] dtype: int64 [Gemm 1: Output Strides]
    # c_strides_2: [e] dtype: int64 [Gemm 2: Output Strides]
    ab_strides_13: torch.Tensor
    ab_strides_2: torch.Tensor
    c_strides_13: torch.Tensor
    c_strides_2: torch.Tensor

    # m: Total number of tokens
    # n: intermediate size per partition
    # k: hidden size per expert
    # e: Number of experts
    # device: Device to run computation on and store tensors
    m: int
    intermediate_size_per_partition: int
    hidden_size: int
    num_experts: int
    device: torch.device

    # Pointers container for calculating offsets of the input activations for each expert
    # a_ptrs: [e] dtype: int64
    a_ptrs: torch.Tensor

    # Pointers container for calculating offsets of the input weights for each expert
    # b_ptrs: [e] dtype: int64
    b_ptrs: torch.Tensor

    # Pointers container for calculating offsets of the output activations for each expert
    # out_ptrs: [e] dtype: int64
    out_ptrs: torch.Tensor
    # Pointers container for calculating offsets of the input scales for each expert
    # a_scales_ptrs: [e] dtype: int64
    # b_scales_ptrs: [e] dtype: int64
    a_scales_ptrs: torch.Tensor
    b_scales_ptrs: torch.Tensor
    # Pointers for per-expert alpha values
    alpha_ptrs: torch.Tensor
    # CUTLASS blockscale layouts for A and B operands
    layout_sfa: torch.Tensor
    layout_sfb: torch.Tensor

    # Offsets that mark at which token index each expert begins its computation
    # The number of tokens computed with expert E is expert_offsets[E + 1] - expert_offsets[E]
    # expert_offsets: [e+1] dtype: int32
    expert_offsets: torch.Tensor

    # Problem size: (num_experts, (m,2n,k)) for first GEMM
    # problem_sizes1: [e, 3] dtype: int32
    # Problem size: (num_experts, (m,n,k)) for second GEMM
    # problem_sizes2: [e, 3] dtype: int32
    problem_sizes1: torch.Tensor
    problem_sizes2: torch.Tensor
    # Similar to expert_offsets, but for blockscales for FP4 blockscaled Group GEMM
    blockscale_offsets: Optional[torch.Tensor] = None
```
**EN:** This block introduces class `CutlassMoEParams` and the state shared by its methods. The class docstring summarizes its role: Parameters for the cutlass moe operation.
**CN:** 该代码块引入类 `CutlassMoEParams`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 95-154: `CutlassMoEParams` initialization and state setup
```python
    def __init__(
        self,
        cutlass_moe_type: CutlassMoEType,
        device: torch.device,
        num_experts: int,
        intermediate_size_per_partition: int,
        hidden_size: int,
    ):
        self.cutlass_moe_type = cutlass_moe_type
        self.device = device
        self.num_experts = num_experts
        self.intermediate_size_per_partition = intermediate_size_per_partition
        self.hidden_size = hidden_size
        self.n = self.intermediate_size_per_partition
        self.k = self.hidden_size
        self.e = self.num_experts
        self.ab_strides_13 = torch.full(
            (self.e,), self.k, dtype=torch.int64, device=self.device
        )
        self.ab_strides_2 = torch.full(
            (self.e,), self.n, dtype=torch.int64, device=self.device
        )
        self.c_strides_13 = torch.full(
            (self.e,), 2 * self.n, dtype=torch.int64, device=self.device
        )
        self.c_strides_2 = torch.full(
            (self.e,), self.k, dtype=torch.int64, device=self.device
        )
        self.expert_offsets = torch.empty(
            (self.e + 1,), dtype=torch.int32, device=self.device
        )
        self.problem_sizes1 = torch.empty(
            (self.e, 3), dtype=torch.int32, device=self.device
        )
        self.problem_sizes2 = torch.empty(
            (self.e, 3), dtype=torch.int32, device=self.device
        )
        if self.cutlass_moe_type == CutlassMoEType.BlockscaledFP4:
            self.blockscale_offsets = torch.empty(
                (self.e + 1,), dtype=torch.int32, device=self.device
            )
        else:
            self.blockscale_offsets = None
        self.a_ptrs = torch.empty((self.e,), dtype=torch.int64, device=self.device)
        self.b_ptrs = torch.empty((self.e,), dtype=torch.int64, device=self.device)
        self.out_ptrs = torch.empty((self.e,), dtype=torch.int64, device=self.device)
        self.a_scales_ptrs = torch.empty(
            (self.e,), dtype=torch.int64, device=self.device
        )
        self.b_scales_ptrs = torch.empty(
            (self.e,), dtype=torch.int64, device=self.device
        )
        self.alpha_ptrs = torch.empty((self.e,), dtype=torch.int64, device=self.device)
        self.layout_sfa = torch.empty(
            (self.e, 5), dtype=torch.int64, device=self.device
        )
        self.layout_sfb = torch.empty(
            (self.e, 5), dtype=torch.int64, device=self.device
        )
```
**EN:** This block defines `CutlassMoEParams.__init__` and contains the main logic for this step. It mainly invokes `torch.full` and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.cutlass_moe_type`, `self.device`, `self.num_experts`, `self.intermediate_size_per_partition`, and `self.hidden_size` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `CutlassMoEParams.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `torch.full` 和 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `self.cutlass_moe_type`、`self.device`、`self.num_experts`、`self.intermediate_size_per_partition` 以及 `self.hidden_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 155-171: Function `CutlassMoEParams.to_gemm1_args` and its core logic
```python
    def to_gemm1_args(self) -> dict:
        return {
            "ab_strides": self.ab_strides_13,
            "c_strides": self.c_strides_13,
            "problem_sizes": self.problem_sizes1,
            "expert_offsets": self.expert_offsets[:-1],
            "blockscale_offsets": self.blockscale_offsets[:-1],
            "a_ptrs": self.a_ptrs,
            "b_ptrs": self.b_ptrs,
            "out_ptrs": self.out_ptrs,
            "a_scales_ptrs": self.a_scales_ptrs,
            "b_scales_ptrs": self.b_scales_ptrs,
            "alpha_ptrs": self.alpha_ptrs,
            "layout_sfa": self.layout_sfa,
            "layout_sfb": self.layout_sfb,
        }
```
**EN:** This block defines `CutlassMoEParams.to_gemm1_args` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CutlassMoEParams.to_gemm1_args`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 172-187: Function `CutlassMoEParams.to_gemm2_args` and its core logic
```python
    def to_gemm2_args(self) -> dict:
        return {
            "ab_strides": self.ab_strides_2,
            "c_strides": self.c_strides_2,
            "problem_sizes": self.problem_sizes2,
            "expert_offsets": self.expert_offsets[:-1],
            "blockscale_offsets": self.blockscale_offsets[:-1],
            "a_ptrs": self.a_ptrs,
            "b_ptrs": self.b_ptrs,
            "out_ptrs": self.out_ptrs,
            "a_scales_ptrs": self.a_scales_ptrs,
            "b_scales_ptrs": self.b_scales_ptrs,
            "alpha_ptrs": self.alpha_ptrs,
            "layout_sfa": self.layout_sfa,
            "layout_sfb": self.layout_sfb,
        }
```
**EN:** This block defines `CutlassMoEParams.to_gemm2_args` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CutlassMoEParams.to_gemm2_args`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `CutlassMoEType` and `CutlassMoEParams`. / **主要符号**：核心入口包括 `CutlassMoEType` 和 `CutlassMoEParams`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `dataclasses.dataclass`, `enum.Enum`, `enum.auto`, and `typing.Optional` / **标准库**：`dataclasses.dataclass`、`enum.Enum`、`enum.auto` 以及 `typing.Optional`
- **Third-party**: `torch` / **第三方依赖**：`torch`
