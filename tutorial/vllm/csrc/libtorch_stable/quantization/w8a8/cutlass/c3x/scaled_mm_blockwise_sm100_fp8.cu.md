# scaled_mm_blockwise_sm100_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_blockwise_sm100_fp8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Routes blockwise-scaled FP8 GEMM on SM100-class hardware to the correct output dtype specialization. / 在 SM100 类硬件上，把 blockwise 缩放的 FP8 GEMM 路由到正确的输出数据类型特化。

## Line-by-Line Analysis / 逐行分析
### Output type dispatch / 输出类型分发
```cpp
void cutlass_scaled_mm_blockwise_sm100_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales) {
  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    cutlass_gemm_blockwise_sm100_fp8_dispatch<cutlass::bfloat16_t>(
        out, a, b, a_scales, b_scales);

  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    cutlass_gemm_blockwise_sm100_fp8_dispatch<cutlass::half_t>(
        out, a, b, a_scales, b_scales);
```
**EN:** The wrapper selects either the BF16 or FP16 specialization of the blockwise SM100 dispatcher based on `out.scalar_type()`, and rejects unsupported output types.
**CN:** 该包装函数根据 `out.scalar_type()` 在 BF16 与 FP16 的 blockwise SM100 分发特化之间做选择，并拒绝不受支持的输出类型。

## Key Concepts / 关键概念
- **Typed dispatch / 类型化分发**: Keeps kernel templates strongly typed while exposing a simple runtime API. / 在保持模板强类型的同时，对外提供简洁的运行时接口。

## Dependencies / 依赖关系
- **scaled_mm epilogues / scaled_mm epilogues**: Connects kernel dispatch to epilogue templates that apply scaling, optional bias, and related post-processing. / 将内核分发逻辑连接到执行缩放、可选 bias 与相关后处理的 epilogue 模板。
- **SM100 blockwise dispatch / SM100 blockwise 分发**: Calls `cutlass_gemm_blockwise_sm100_fp8_dispatch` for the real kernel choice. / 调用 `cutlass_gemm_blockwise_sm100_fp8_dispatch` 执行真正的内核选择。
