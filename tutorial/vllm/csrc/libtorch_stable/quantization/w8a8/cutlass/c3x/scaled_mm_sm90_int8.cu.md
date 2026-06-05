# scaled_mm_sm90_int8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_sm90_int8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the public SM90 int8 scaled-matmul entry point and chooses bias-aware epilogues. / 实现公开的 SM90 int8 scaled matmul 入口，并选择感知 bias 的 epilogue。

## Line-by-Line Analysis / 逐行分析
### Public SM90 int8 entry / 公开的 SM90 int8 入口
```cpp
void cutlass_scaled_mm_sm90_int8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales,
    std::optional<torch::stable::Tensor> const& bias) {
  STD_TORCH_CHECK(a_scales.is_contiguous() && b_scales.is_contiguous());
  if (bias) {
    STD_TORCH_CHECK(bias->scalar_type() == out.scalar_type(),
                    "currently bias dtype must match output dtype ",
                    out.scalar_type());
    return cutlass_scaled_mm_sm90_int8_epilogue<c3x::ScaledEpilogueBias>(
        out, a, b, a_scales, b_scales, *bias);
  } else {
    return cutlass_scaled_mm_sm90_int8_epilogue<c3x::ScaledEpilogue>(
```
**EN:** The wrapper validates contiguous scales, checks optional bias dtype, and chooses between `ScaledEpilogueBias` and `ScaledEpilogue` before forwarding to the SM90 int8 dispatcher.
**CN:** 该包装函数会验证缩放张量连续性、检查可选 bias 的数据类型，然后在 `ScaledEpilogueBias` 与 `ScaledEpilogue` 之间做选择，并转发给 SM90 int8 分发器。

## Key Concepts / 关键概念
- **Epilogue-only front-end / 仅负责 epilogue 的前端**: The `.cu` file only decides the post-processing template; kernel selection stays in the dispatch header. / 该 `.cu` 文件只决定后处理模板，真正的内核选择仍在 dispatch 头文件中。

## Dependencies / 依赖关系
- **SM90 int8 dispatch / SM90 int8 分发**: Calls `cutlass_scaled_mm_sm90_int8_epilogue` defined in `scaled_mm_sm90_int8_dispatch.cuh`. / 调用 `scaled_mm_sm90_int8_dispatch.cuh` 中定义的 `cutlass_scaled_mm_sm90_int8_epilogue`。
