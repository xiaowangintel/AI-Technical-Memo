# cutlass_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/cutlass_moe.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `CUTLASS MoE`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `CUTLASS MoE` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Imports and module setup
````python
import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 4-43: `get_cutlass_w4a8_moe_mm_data` definition
````python
def get_cutlass_w4a8_moe_mm_data(
    topk_ids: torch.Tensor,
    expert_offsets: torch.Tensor,
    problem_sizes1: torch.Tensor,
    problem_sizes2: torch.Tensor,
    input_permutation: torch.Tensor,
    output_permutation: torch.Tensor,
    num_experts: int,
    n: int,
    k: int,
):
    """
    Prepare data necessary to perform CUTLASS grouped matrix multiplications
    used in CUTLASS-based fused MoE.

    The function takes in topk_ids (token-expert mapping) and uses it to
    compute:
    - expert_offsets: Indices that mark at which token index each expert begins
                      its computation after the input is sorted with
                      input_permutation. The number of tokens computed with
                      expert E is expert_offsets[E + 1] - expert_offsets[E]
    - problem_sizes1, problem_sizes2: MxNxK sizes of each expert's
                                      multiplication in two grouped MMs used in
                                      the fused MoE operation.
    - input_permutation: Permutation that must be used to shuffle the input
                         before executing the MMs.
    - output_permutation: Permutation that must be used to shuffle the output
                          after executing the MMs.
    """
    torch.ops.sgl_kernel.get_cutlass_w4a8_moe_mm_data.default(
        topk_ids,
        expert_offsets,
        problem_sizes1,
        problem_sizes2,
        input_permutation,
        output_permutation,
        num_experts,
        n,
        k,
    )
````
**EN:** This section defines `get_cutlass_w4a8_moe_mm_data`. It retrieves or computes the `cutlass w4a8 MoE mm data` path used by the module. Docstring summary: Prepare data necessary to perform CUTLASS grouped matrix multiplications used in CUTLASS-based fused MoE. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `get_cutlass_w4a8_moe_mm_data`。它负责获取或计算模块中与 `cutlass w4a8 MoE mm data` 相关的处理路径。 文档字符串摘要：Prepare data necessary to perform CUTLASS grouped matrix multiplications used in CUTLASS-based fused MoE. 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 46-112: `cutlass_w4a8_moe_mm` definition
````python
def cutlass_w4a8_moe_mm(
    d: torch.Tensor,
    a: torch.Tensor,
    b: torch.Tensor,
    a_scales: torch.Tensor,
    b_scales: torch.Tensor,
    experts_offsets: torch.tensor,
    problem_sizes: torch.tensor,
    a_strides: torch.tensor,
    b_strides: torch.tensor,
    d_strides: torch.tensor,
    s_strides: torch.tensor,
    chunk_size: int = 128,
    topk: int = 8,
):
    """
    Perform grouped matrix multiplication between int4 weights and fp8 activations.

    This function executes multiple GEMM operations in parallel, which is useful for
    scenarios like Mixture of Experts (MoE) where different inputs go through different
    experts. The implementation leverages NVIDIA Hopper architecture features for
    optimal performance with quantized weights.

    Args:
        d: Output matrices of shape [total_m, total_n]
        a: Activation matrices in FP8 (float_e4m3_t) format
           Each tensor should be of shape [total_m, K] in row-major layout
        b: Weight matrices in packed int4 format
           Each tensor should be of shape [E, N, K//2] in column-major layout
           where each byte contains two 4-bit integers
        a_scales: Scale factors for the inputs
        b_scales: Scale factors for the quantized weights
           Each tensor should be of shape [E, K//512, N*8]
        experts_offsets: Tensor containing expert offsets for determining group boundaries
        problem_sizes: with shape [num_experts, 3] (M, N, K for each group) (int32)
        a_strides: Strides information for A matrices
        b_strides: Strides information for B matrices
        d_strides: Strides information for D matrices
        s_strides: Strides information for b_scales matrices
        chunk_size: Number of elements each scale value applies to (K//512), default to 128

    Requirements:
        - All tensors must be on a CUDA device
        - Requires an NVIDIA Hopper GPU (H100)
        - A tensors must be in float8_e4m3fn format
        - B tensors must contain packed int4 values (stored as int8)

    Note:
        The function computes: D = (A * (B * scales))
        for each group of tensors in parallel
    """

    torch.ops.sgl_kernel.cutlass_w4a8_moe_mm.default(
        d,
        a,
        b,
        a_scales,
        b_scales,
        experts_offsets,
        problem_sizes,
        a_strides,
        b_strides,
        d_strides,
        s_strides,
        chunk_size,
        topk,
    )
````
**EN:** This section defines `cutlass_w4a8_moe_mm` and implements the core logic associated with cutlass w4a8 MoE mm. Docstring summary: Perform grouped matrix multiplication between int4 weights and fp8 activations. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `cutlass_w4a8_moe_mm`，并实现与 cutlass w4a8 MoE mm 相关的核心逻辑。 文档字符串摘要：Perform grouped matrix multiplication between int4 weights and fp8 activations. 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `get_cutlass_w4a8_moe_mm_data`, `cutlass_w4a8_moe_mm`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
