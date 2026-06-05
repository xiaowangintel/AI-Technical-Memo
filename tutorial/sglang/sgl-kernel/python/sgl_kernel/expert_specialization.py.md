# expert_specialization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/expert_specialization.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `expert specialization`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `expert specialization` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Imports and module setup
````python
import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 4-29: `es_fp8_blockwise_scaled_grouped_mm` definition
````python
def es_fp8_blockwise_scaled_grouped_mm(
    output,
    a,
    b,
    scales_a,
    scales_b,
    stride_a,
    stride_b,
    stride_d,
    problem_sizes,
    expert_offsets,
    workspace,
):
    torch.ops.sgl_kernel.es_fp8_blockwise_scaled_grouped_mm.default(
        output,
        a,
        b,
        scales_a,
        scales_b,
        stride_a,
        stride_b,
        stride_d,
        problem_sizes,
        expert_offsets,
        workspace,
    )
````
**EN:** This section defines `es_fp8_blockwise_scaled_grouped_mm` and implements the core logic associated with expert specialization FP8 blockwise scaled grouped mm. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `es_fp8_blockwise_scaled_grouped_mm`，并实现与 expert specialization FP8 blockwise scaled grouped mm 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 32-37: `es_sm100_mxfp8_blockscaled_grouped_mm` definition
````python
def es_sm100_mxfp8_blockscaled_grouped_mm(
    output, a, b, sfa, sfb, problem_sizes, expert_offsets, blockscale_offsets
):
    torch.ops.sgl_kernel.es_sm100_mxfp8_blockscaled_grouped_mm.default(
        a, b, sfa, sfb, output, problem_sizes, expert_offsets, blockscale_offsets
    )
````
**EN:** This section defines `es_sm100_mxfp8_blockscaled_grouped_mm` and implements the core logic associated with expert specialization sm100 MXFP8 blockscaled grouped mm. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `es_sm100_mxfp8_blockscaled_grouped_mm`，并实现与 expert specialization sm100 MXFP8 blockscaled grouped mm 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 40-50: `es_sm100_mxfp8_blockscaled_grouped_quant` definition
````python
def es_sm100_mxfp8_blockscaled_grouped_quant(
    input, problem_sizes, expert_offsets, blockscale_offsets, quant_output, scale_factor
):
    torch.ops.sgl_kernel.es_sm100_mxfp8_blockscaled_grouped_quant.default(
        input,
        problem_sizes,
        expert_offsets,
        blockscale_offsets,
        quant_output,
        scale_factor,
    )
````
**EN:** This section defines `es_sm100_mxfp8_blockscaled_grouped_quant` and implements the core logic associated with expert specialization sm100 MXFP8 blockscaled grouped quantization. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `es_sm100_mxfp8_blockscaled_grouped_quant`，并实现与 expert specialization sm100 MXFP8 blockscaled grouped quantization 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `es_fp8_blockwise_scaled_grouped_mm`, `es_sm100_mxfp8_blockscaled_grouped_mm`, `es_sm100_mxfp8_blockscaled_grouped_quant`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
