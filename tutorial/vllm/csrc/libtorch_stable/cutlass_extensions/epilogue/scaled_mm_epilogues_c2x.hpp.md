# scaled_mm_epilogues_c2x.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/cutlass_extensions/epilogue/scaled_mm_epilogues_c2x.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines CUTLASS 2.x epilogue visitor trees for scaled matrix multiplication with optional bias and activation zero-point corrections. / 为带可选 bias 与激活零点修正的 scaled matrix multiplication 定义 CUTLASS 2.x 的 epilogue visitor tree。

## Line-by-Line Analysis / 逐行分析
### Shared descriptor utilities
```cpp
template <typename ElementD, typename OutputTileThreadMap>
struct ScaledEpilogueBase {
 protected:
  using Accum = cutlass::epilogue::threadblock::VisitorAccFetch;
  ...
  template <typename Descriptor, typename T>
  static auto args_from_tensor(torch::stable::Tensor const& tensor) {
    using Arguments = typename Descriptor::Arguments;
    auto* data_ptr = static_cast<T*>(tensor.data_ptr());
    if constexpr (std::is_same_v<Descriptor, ColOrScalarLoad<T>> ||
                  std::is_same_v<Descriptor, RowOrScalarLoad<T>>) {
      return Arguments{data_ptr, tensor.numel() != 1};
```
**EN:** The base class abstracts how epilogues load per-row, per-column, scalar, or optional tensors. This lets all derived epilogues build visitor arguments from `torch::stable::Tensor` without duplicating pointer/shape rules.
**CN:** 基类抽象了 epilogue 如何加载按行、按列、标量或可选张量的数据。这样所有派生 epilogue 都能从 `torch::stable::Tensor` 构造 visitor 参数，而不必重复处理指针和形状规则。

### Plain scaled epilogue
```cpp
using ScaleA = typename SUPER::template ColOrScalarLoad<float>;
using ScaleB = typename SUPER::template RowOrScalarLoad<float>;
...
using EVTCompute =
    cutlass::epilogue::threadblock::Sm80EVT<Compute1, ScaleA, EVTCompute0>;
...
static ArgumentType prepare_args(torch::stable::Tensor const& a_scales,
                                 torch::stable::Tensor const& b_scales) {
  auto a_args = SUPER::template args_from_tensor<ScaleA, float>(a_scales);
  auto b_args = SUPER::template args_from_tensor<ScaleB, float>(b_scales);
```
**EN:** `ScaledEpilogue` implements the basic `_scaled_mm` formula. It multiplies the accumulator by B-scales first, then applies A-scales, matching the broadcast semantics of per-token/per-channel quantization.
**CN:** `ScaledEpilogue` 实现了基础 `_scaled_mm` 公式。它先用 B-scale 处理累加值，再应用 A-scale，对应逐 token / 逐通道量化的广播语义。

### Bias fusion
```cpp
using Bias = typename SUPER::template RowLoad<ElementD>;
...
using EVTCompute = cutlass::epilogue::threadblock::Sm80EVT<Compute1, ScaleA,
                                                           EVTCompute0, Bias>;
...
return ArgumentType{a_args, evt0_args, bias_args, {}};
```
**EN:** `ScaledEpilogueBias` extends the basic path with a per-output-channel bias term. The final visitor uses `homogeneous_multiply_add`, so bias is fused directly into the epilogue instead of requiring a separate kernel.
**CN:** `ScaledEpilogueBias` 在基础路径上加入了按输出通道广播的 bias 项。最终 visitor 使用 `homogeneous_multiply_add`，因此 bias 会直接融合进 epilogue，而无需额外核函数。

### Per-tensor AZP correction
```cpp
using AzpWithAdj = typename SUPER::template RowLoad<int32_t>;
using ComputeAzp = cutlass::epilogue::threadblock::VisitorCompute<
    cutlass::minus, float, int32_t,
    cutlass::FloatRoundStyle::round_to_nearest>;
...
typename EVTComputeAzp::Arguments evt_azp_args{{}, azp_adj_args, {}};
typename EVTComputeScaleB::Arguments evt_scale_b_args{
    b_args, evt_azp_args, {}};
return ArgumentType{a_args, evt_scale_b_args, bias_args, {}};
```
**EN:** `ScaledEpilogueBiasAzp` handles symmetric scales plus an already materialized activation-zero-point correction term `azp * J @ B`. It subtracts that correction from the accumulator before applying scales and bias.
**CN:** `ScaledEpilogueBiasAzp` 处理对称 scale 外加已经物化好的激活零点修正项 `azp * J @ B`。它会在应用 scale 和 bias 之前，先从累加值中减去这部分修正。

### Per-token AZP rank-1 update
```cpp
using Azp = typename SUPER::template ColLoad<int32_t>;
using AzpAdj = typename SUPER::template RowLoad<int32_t>;
...
using EVTComputeAzp =
    cutlass::epilogue::threadblock::Sm80EVT<ComputeAzp, Azp, AzpAdj>;
using EVTComputeAcc =
    cutlass::epilogue::threadblock::Sm80EVT<ComputeAcc, Accum, EVTComputeAzp>;
```
**EN:** `ScaledEpilogueBiasAzpToken` avoids materializing an `m x n` correction matrix. Instead it computes a rank-1 product from per-token AZP and per-channel `J @ B`, then subtracts that from the accumulator on the fly.
**CN:** `ScaledEpilogueBiasAzpToken` 避免显式构造 `m x n` 的修正矩阵。它通过逐 token 的 AZP 与逐通道的 `J @ B` 在线计算一个 rank-1 乘积，再将其从累加值中扣除。

## Key Concepts / 关键概念
- **EVT composition / EVT 组合**: Each epilogue is a visitor tree composed from broadcast loads and compute nodes.
- **Broadcasted quantization metadata / 广播量化元数据**: A-scales, B-scales, bias, and AZP terms can be scalar/row/column tensors.
- **Fused post-processing / 融合后处理**: Scaling, bias, and zero-point correction happen in the GEMM epilogue.

## Dependencies / 依赖关系
- Builds on `broadcast_load_epilogue_c2x.hpp` descriptors.
- Consumes `torch::stable::Tensor` metadata to prepare CUTLASS epilogue arguments.
- Used by stable-ABI scaled GEMM implementations elsewhere in `libtorch_stable`.
