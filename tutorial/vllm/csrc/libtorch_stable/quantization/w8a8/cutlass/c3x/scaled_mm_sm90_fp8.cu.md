# scaled_mm_sm90_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm90_fp8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides the public SM90 FP8 entry point and selects the correct epilogue path based on bias and batch-invariant mode. / 提供公开的 SM90 FP8 入口，并根据 bias 与 batch-invariant 模式选择正确的 epilogue 路径。

## Line-by-Line Analysis / 逐行分析
### Public SM90 FP8 entry / 公开的 SM90 FP8 入口
```cpp
void cutlass_scaled_mm_sm90_fp8(
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
      return cutlass_scaled_mm_sm90_fp8_batch_invariant_epilogue<true>(
          out, a, b, a_scales, b_scales, *bias);
    }
    return cutlass_scaled_mm_sm90_fp8_epilogue<true>(out, a, b, a_scales,
                                                     b_scales, *bias);
  } else {
    if (vllm_is_batch_invariant()) {
      return cutlass_scaled_mm_sm90_fp8_batch_invariant_epilogue<false>(
          out, a, b, a_scales, b_scales);
    }
    return cutlass_scaled_mm_sm90_fp8_epilogue<false>(out, a, b, a_scales,
                                                      b_scales);
```
**EN:** The function validates scale tensor layout, checks bias dtype compatibility, and then chooses among bias/no-bias and batch-invariant/non-batch-invariant helper templates.
**CN:** 该函数会验证缩放张量布局、检查 bias 数据类型兼容性，然后在“有/无 bias”和“batch-invariant/非 batch-invariant”的辅助模板之间做选择。

## Key Concepts / 关键概念
- **Bias-aware front-end / 感知 bias 的前端**: Bias only changes epilogue selection; the GEMM core remains in the dispatch header. / bias 只影响 epilogue 选择，GEMM 核心仍保留在 dispatch 头文件中。

## Dependencies / 依赖关系
- **SM90 FP8 dispatch / SM90 FP8 分发**: Uses helpers from `scaled_mm_sm90_fp8_dispatch.cuh`. / 使用 `scaled_mm_sm90_fp8_dispatch.cuh` 中的辅助逻辑。
