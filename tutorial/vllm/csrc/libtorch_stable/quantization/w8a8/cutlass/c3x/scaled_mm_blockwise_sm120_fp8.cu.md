# scaled_mm_blockwise_sm120_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_blockwise_sm120_fp8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Routes blockwise-scaled FP8 GEMM on SM120-family hardware to BF16 or FP16 implementations. / 在 SM120 系列硬件上，把 blockwise 缩放 FP8 GEMM 路由到 BF16 或 FP16 实现。

## Line-by-Line Analysis / 逐行分析
### Output type dispatch / 输出类型分发
```cpp
void cutlass_scaled_mm_blockwise_sm120_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales) {
  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    cutlass_gemm_blockwise_sm120_fp8_dispatch<cutlass::bfloat16_t>(
        out, a, b, a_scales, b_scales);

  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    cutlass_gemm_blockwise_sm120_fp8_dispatch<cutlass::half_t>(
        out, a, b, a_scales, b_scales);
```
**EN:** Like the SM100 wrapper, this function chooses the blockwise SM120 dispatcher specialization that matches the output tensor type.
**CN:** 与 SM100 包装函数类似，这里根据输出张量类型选择匹配的 blockwise SM120 分发特化。

## Key Concepts / 关键概念
- **Output specialization / 输出特化**: The runtime API stays simple while the actual kernels remain templated by output type. / 运行时接口保持简洁，而实际内核继续按输出类型模板化。

## Dependencies / 依赖关系
- **SM120 blockwise dispatch / SM120 blockwise 分发**: Calls `cutlass_gemm_blockwise_sm120_fp8_dispatch` to pick the concrete kernel. / 调用 `cutlass_gemm_blockwise_sm120_fp8_dispatch` 选择具体内核。
