# scaled_mm_helper.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/scaled_mm_helper.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the top-level runtime dispatcher that decides between standard FP8, int8, and blockwise scaled GEMM paths. / 实现顶层运行时分发器，用于在标准 FP8、int8 和 blockwise 缩放 GEMM 路径之间做选择。

## Line-by-Line Analysis / 逐行分析
### Scale tensor validation / 缩放张量校验
```cpp
template <typename Fp8Func, typename Int8Func, typename BlockwiseFunc>
void dispatch_scaled_mm(torch::stable::Tensor& c,
                        torch::stable::Tensor const& a,
                        torch::stable::Tensor const& b,
                        torch::stable::Tensor const& a_scales,
                        torch::stable::Tensor const& b_scales,
                        std::optional<torch::stable::Tensor> const& bias,
                        Fp8Func fp8_func, Int8Func int8_func,
                        BlockwiseFunc blockwise_func) {
  STD_TORCH_CHECK(a_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
  STD_TORCH_CHECK(b_scales.scalar_type() ==
                  torch::headeronly::ScalarType::Float);
```
**EN:** Before dispatch, the helper enforces that both scale tensors use float storage, which matches the downstream CUTLASS epilogue expectations.
**CN:** 在分发前，该辅助函数强制要求两个缩放张量都使用 float 存储，这与下游 CUTLASS epilogue 的预期一致。

### Standard scaling path / 标准缩放路径
```cpp
  if ((a_scales.numel() == 1 || a_scales.numel() == a.size(0)) &&
      (b_scales.numel() == 1 || b_scales.numel() == b.size(1))) {
    // Standard per-tensor/per-token/per-channel scaling
    STD_TORCH_CHECK(a_scales.is_contiguous() && b_scales.is_contiguous());
    if (a.scalar_type() == torch::headeronly::ScalarType::Float8_e4m3fn) {
      fp8_func(c, a, b, a_scales, b_scales, bias);
    } else {
      STD_TORCH_CHECK(a.scalar_type() == torch::headeronly::ScalarType::Char);
      if constexpr (!std::is_same_v<Int8Func, std::nullptr_t>) {
        int8_func(c, a, b, a_scales, b_scales, bias);
      } else {
        int32_t version_num = get_sm_version_num();
        STD_TORCH_CHECK(
            false, "Int8 not supported on SM", version_num,
            ". Use FP8 quantization instead, or run on older arch (SM < 100).");
      }
    }
```
**EN:** If the scale tensors are scalar/per-token/per-channel shapes, the code takes the standard path: FP8 tensors call `fp8_func`, while int8 tensors call `int8_func` when supported.
**CN:** 如果缩放张量是标量、按 token 或按通道的形状，代码会进入标准路径：FP8 张量调用 `fp8_func`，而受支持时 int8 张量调用 `int8_func`。

### Blockwise path / 分块路径
```cpp
  } else {
    STD_TORCH_CHECK(a_scales.dim() == 2, "a scale must be 2d tensor.");
    STD_TORCH_CHECK(b_scales.dim() == 2, "b scale must be 2d tensor.");
    int32_t version_num = get_sm_version_num();
    if (version_num >= 90) {
      STD_TORCH_CHECK(
          a.size(0) == a_scales.size(0) &&
              cuda_utils::ceil_div(a.size(1), int64_t(128)) == a_scales.size(1),
          "a_scale_group_shape must be [1, 128].");
      STD_TORCH_CHECK(
          cuda_utils::ceil_div(b.size(0), int64_t(128)) == b_scales.size(0) &&
              cuda_utils::ceil_div(b.size(1), int64_t(128)) == b_scales.size(1),
          "b_scale_group_shape must be [128, 128].");
    }

    STD_TORCH_CHECK(!bias, "Bias not yet supported blockwise scaled_mm");
    blockwise_func(c, a, b, a_scales, b_scales);
```
**EN:** Otherwise the helper interprets scales as blockwise tensors, validates their 2D group layout, forbids bias for now, and forwards execution to `blockwise_func`.
**CN:** 否则，该辅助函数会把缩放解释为 blockwise 张量，验证其二维分组布局，当前禁止 bias，并把执行转发给 `blockwise_func`。

## Key Concepts / 关键概念
- **Runtime feature gating / 运行时功能门控**: The helper hides hardware- and quantization-specific branching behind one call site. / 该辅助函数把硬件相关与量化相关的分支隐藏在单一调用点之后。
- **Scale-shape semantics / 缩放形状语义**: The number and dimensionality of scale elements decide which quantization mode is active. / 缩放元素的数量与维度决定了当前启用哪种量化模式。

## Dependencies / 依赖关系
- **cuda_utils / cuda_utils**: Uses `ceil_div` for validating blockwise scale grid shapes. / 使用 `ceil_div` 验证 blockwise 缩放网格形状。
- **SM version helpers / SM 版本辅助函数**: Calls `get_sm_version_num()` from CUTLASS extension utilities to gate blockwise support and int8 availability. / 调用 CUTLASS 扩展工具中的 `get_sm_version_num()` 来限制 blockwise 支持范围与 int8 可用性。
