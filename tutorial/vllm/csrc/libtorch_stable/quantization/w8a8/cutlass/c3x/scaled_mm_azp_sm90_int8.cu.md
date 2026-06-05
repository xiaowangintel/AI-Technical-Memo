# scaled_mm_azp_sm90_int8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_azp_sm90_int8.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Selects the SM90 int8 scaled-matmul epilogue for asymmetric zero-point (AZP) execution. / 为带非对称零点（AZP）的 SM90 int8 缩放矩阵乘选择合适的 epilogue。

## Line-by-Line Analysis / 逐行分析
### AZP epilogue selection / AZP 后处理选择
```cpp
void cutlass_scaled_mm_azp_sm90_int8(
    torch::stable::Tensor& out, torch::stable::Tensor const& a,
    torch::stable::Tensor const& b, torch::stable::Tensor const& a_scales,
    torch::stable::Tensor const& b_scales, torch::stable::Tensor const& azp_adj,
    std::optional<torch::stable::Tensor> const& azp,
    std::optional<torch::stable::Tensor> const& bias) {
  if (azp) {
    return cutlass_scaled_mm_sm90_int8_epilogue<
        c3x::ScaledEpilogueBiasAzpToken>(out, a, b, a_scales, b_scales, azp_adj,
                                         *azp, bias);
  } else {
    return cutlass_scaled_mm_sm90_int8_epilogue<c3x::ScaledEpilogueBiasAzp>(
        out, a, b, a_scales, b_scales, azp_adj, bias);
```
**EN:** This wrapper checks whether an explicit `azp` tensor is present. If it is, it picks the token-aware `ScaledEpilogueBiasAzpToken`; otherwise it falls back to `ScaledEpilogueBiasAzp`, while forwarding the shared scale, adjustment, and optional bias tensors.
**CN:** 该包装函数检查是否提供了显式的 `azp` 张量；若存在，则选择按 token 处理的 `ScaledEpilogueBiasAzpToken`，否则回退到 `ScaledEpilogueBiasAzp`，同时转发公共缩放、校正项以及可选 bias。

## Key Concepts / 关键概念
- **Runtime epilogue choice / 运行时后处理选择**: Keeps AZP handling in a thin front-end while reusing the common SM90 int8 dispatcher. / 将 AZP 处理保留在轻量前端中，同时复用通用的 SM90 int8 分发器。

## Dependencies / 依赖关系
- **scaled_mm epilogues / scaled_mm epilogues**: Connects kernel dispatch to epilogue templates that apply scaling, optional bias, and related post-processing. / 将内核分发逻辑连接到执行缩放、可选 bias 与相关后处理的 epilogue 模板。
- **SM90 int8 dispatch / SM90 int8 分发**: Delegates the real kernel selection to `scaled_mm_sm90_int8_dispatch.cuh`. / 把真正的内核选择交给 `scaled_mm_sm90_int8_dispatch.cuh`。
