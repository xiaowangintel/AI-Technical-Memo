# scaled_mm_blockwise_sm90_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_blockwise_sm90_fp8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides the public SM90 FP8 blockwise entry point and chooses the output accumulation type. / 提供公开的 SM90 FP8 blockwise 入口，并选择输出累积类型。

## Line-by-Line Analysis / 逐行分析
### Output type dispatch / 输出类型分发
```cpp
void cutlass_scaled_mm_blockwise_sm90_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales) {
  if (out.scalar_type() == torch::headeronly::ScalarType::BFloat16) {
    cutlass_gemm_blockwise_sm90_fp8_dispatch<cutlass::bfloat16_t>(
        out, a, b, a_scales, b_scales);

  } else {
    STD_TORCH_CHECK(out.scalar_type() == torch::headeronly::ScalarType::Half);
    cutlass_gemm_blockwise_sm90_fp8_dispatch<cutlass::half_t>(
        out, a, b, a_scales, b_scales);
```
**EN:** This thin wrapper dispatches to the SM90 blockwise kernel specialization for BF16 or FP16 output tensors.
**CN:** 这个轻量包装函数会把调用分发到适用于 BF16 或 FP16 输出张量的 SM90 blockwise 内核特化。

## Key Concepts / 关键概念
- **Thin front-end / 轻量前端**: The `.cu` file only exposes a stable API while the heavy template logic lives in the header. / 该 `.cu` 文件只暴露稳定 API，复杂模板逻辑则放在头文件中。

## Dependencies / 依赖关系
- **SM90 blockwise dispatch / SM90 blockwise 分发**: Delegates to `cutlass_gemm_blockwise_sm90_fp8_dispatch`. / 委托给 `cutlass_gemm_blockwise_sm90_fp8_dispatch`。
