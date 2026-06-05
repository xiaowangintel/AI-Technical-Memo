# scaled_mm_sm100_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm100_fp8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the SM100 FP8 public entry point and selects bias-aware versus bias-free epilogues, including batch-invariant mode. / 实现 SM100 FP8 公开入口，并在带 bias / 不带 bias 的 epilogue 之间选择，同时处理 batch-invariant 模式。

## Line-by-Line Analysis / 逐行分析
### Public SM100 FP8 entry / 公开的 SM100 FP8 入口
```cpp
void cutlass_scaled_mm_sm100_fp8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales,
    std::optional<torch::stable::Tensor> const& bias) {
  STD_TORCH_CHECK(a_scales.is_contiguous() && b_scales.is_contiguous());
  if (bias) {
    STD_TORCH_CHECK(bias->scalar_type() == out.scalar_type(),
                    "currently bias dtype must match output dtype ",
                    out.scalar_type());
    if (vllm_is_batch_invariant()) {
      return cutlass_scaled_mm_sm100_fp8_batch_invariant_epilogue<true>(
          out, a, b, a_scales, b_scales, *bias);
    }
    return cutlass_scaled_mm_sm100_fp8_epilogue<true>(out, a, b, a_scales,
                                                      b_scales, *bias);
  } else {
    if (vllm_is_batch_invariant()) {
      return cutlass_scaled_mm_sm100_fp8_batch_invariant_epilogue<false>(
          out, a, b, a_scales, b_scales);
    }
    return cutlass_scaled_mm_sm100_fp8_epilogue<false>(out, a, b, a_scales,
                                                       b_scales);
```
**EN:** The function first validates contiguous scale tensors, checks bias dtype compatibility, and then selects one of four paths: bias/no-bias crossed with batch-invariant/non-batch-invariant execution.
**CN:** 该函数先验证缩放张量连续性、检查 bias 数据类型兼容性，然后在四条路径之间选择：有/无 bias 与 batch-invariant/非 batch-invariant 的组合。

## Key Concepts / 关键概念
- **Batch invariance / 批不变性**: When enabled, the code uses a fixed kernel choice so output behavior does not depend on batch size. / 启用后，代码会使用固定的内核选择，使输出行为不依赖 batch 大小。

## Dependencies / 依赖关系
- **scaled_mm epilogues / scaled_mm epilogues**: Connects kernel dispatch to epilogue templates that apply scaling, optional bias, and related post-processing. / 将内核分发逻辑连接到执行缩放、可选 bias 与相关后处理的 epilogue 模板。
- **SM100 FP8 dispatch / SM100 FP8 分发**: Calls helper templates from `scaled_mm_sm100_fp8_dispatch.cuh`. / 调用 `scaled_mm_sm100_fp8_dispatch.cuh` 中的辅助模板。
