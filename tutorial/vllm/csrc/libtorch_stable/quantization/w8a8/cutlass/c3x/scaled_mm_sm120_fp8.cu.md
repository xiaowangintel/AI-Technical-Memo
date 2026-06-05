# scaled_mm_sm120_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm120_fp8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides the public SM120 FP8 entry point and selects bias/no-bias plus batch-invariant execution modes. / 提供公开的 SM120 FP8 入口，并选择带/不带 bias 以及 batch-invariant 执行模式。

## Line-by-Line Analysis / 逐行分析
### Public SM120 FP8 entry / 公开的 SM120 FP8 入口
```cpp
void cutlass_scaled_mm_sm120_fp8(
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
      return cutlass_scaled_mm_sm120_fp8_batch_invariant_epilogue<
          c3x::ScaledEpilogueBias>(out, a, b, a_scales, b_scales, *bias);
    }
    return cutlass_scaled_mm_sm120_fp8_epilogue<c3x::ScaledEpilogueBias>(
        out, a, b, a_scales, b_scales, *bias);
  } else {
    if (vllm_is_batch_invariant()) {
      return cutlass_scaled_mm_sm120_fp8_batch_invariant_epilogue<
          c3x::ScaledEpilogue>(out, a, b, a_scales, b_scales);
    }
    return cutlass_scaled_mm_sm120_fp8_epilogue<c3x::ScaledEpilogue>(
        out, a, b, a_scales, b_scales);
```
**EN:** This wrapper mirrors the SM100 control flow: validate scale tensors, verify optional bias dtype, then pick the right epilogue helper depending on bias presence and batch invariance.
**CN:** 该包装函数延续了 SM100 的控制流：验证缩放张量、检查可选 bias 的数据类型，然后根据 bias 是否存在以及是否 batch-invariant 选择合适的 epilogue 辅助函数。

## Key Concepts / 关键概念
- **Shared front-end pattern / 共享前端模式**: The SM100 and SM120 `.cu` entry files share nearly identical control flow, keeping architecture-specific details in dispatch headers. / SM100 与 SM120 的 `.cu` 入口文件几乎共享同一控制流，把架构细节留在 dispatch 头文件中。

## Dependencies / 依赖关系
- **SM120 FP8 dispatch / SM120 FP8 分发**: Uses helpers from `scaled_mm_sm120_fp8_dispatch.cuh`. / 使用 `scaled_mm_sm120_fp8_dispatch.cuh` 中的辅助逻辑。
