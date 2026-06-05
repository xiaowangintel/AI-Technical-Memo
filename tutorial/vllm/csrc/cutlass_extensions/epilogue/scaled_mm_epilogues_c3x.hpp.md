# scaled_mm_epilogues_c3x.hpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `csrc/cutlass_extensions/epilogue/scaled_mm_epilogues_c3x.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines Hopper-and-later CUTLASS 3.x fused epilogues for scaled GEMM, optional bias, activation zero-point corrections, and grouped scale arrays. / [CN] 为 Hopper 及更新架构上的 CUTLASS 3.x 定义融合 epilogue，支持 scaled GEMM、可选 bias、激活零点修正以及分组 scale 指针数组。

## Line-by-Line Analysis / 逐行分析
### File contract and trivial epilogue / 文件约定与最简 epilogue
```cpp
/*
   This file defines custom epilogues for fusing channel scales, token scales,
   bias, and activation zero-points onto a GEMM operation using the
   CUTLASS 3.x API, for NVIDIA GPUs with sm90a (Hopper) or later.

   Epilogues must contain a public type named EVTCompute of type Sm90EVT,
   as well as a static prepare_args function that constructs an
   EVTCompute::Arguments struct.
*/

namespace vllm::c3x {

#ifdef TORCH_TARGET_VERSION
using TensorType = torch::stable::Tensor;
#else
using TensorType = torch::Tensor;
#endif

using namespace cute;

template <typename T>
struct identity {
  CUTLASS_HOST_DEVICE
  T operator()(T lhs) const { return lhs; }
};

template <typename ElementAcc, typename ElementD, typename TileShape>
struct TrivialEpilogue {
 private:
  using Accum = cutlass::epilogue::fusion::Sm90AccFetch;
  using Compute = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::epilogue::thread::Identity, ElementD, ElementAcc,
      cutlass::FloatRoundStyle::round_to_nearest>;

 public:
  using EVTCompute = cutlass::epilogue::fusion::Sm90EVT<Compute, Accum>;
  using ArgumentType = typename EVTCompute::Arguments;

  template <typename... Args>
  static ArgumentType prepare_args(Args... args) {
    return {};
  }
};
```
**EN:** The comment states the contract every custom epilogue must satisfy: expose an `EVTCompute` type and a `prepare_args` function. `TensorType` abstracts over stable and unstable Torch ABIs, `identity` is a tiny callable helper, and `TrivialEpilogue` shows the minimal Sm90 EVT stack: fetch accumulator, apply identity conversion, and return an empty argument pack.
**CN:** 注释说明了每个自定义 epilogue 必须满足的接口约定：暴露 `EVTCompute` 类型，并提供 `prepare_args` 函数。`TensorType` 用来兼容稳定/非稳定 Torch ABI，`identity` 是一个极小的可调用辅助对象，而 `TrivialEpilogue` 则展示了最简 Sm90 EVT 结构：取出累加值、做恒等转换，并返回空参数包。

### Shared descriptor and argument helpers / 共享描述符与参数辅助函数
```cpp
template <typename ElementAcc, typename ElementD, typename TileShape>
struct ScaledEpilogueBase {
 protected:
  using Accum = cutlass::epilogue::fusion::Sm90AccFetch;

  template <typename T>
  using ColOrScalarLoad = cutlass::epilogue::fusion::Sm90ColOrScalarBroadcast<
      0 /*Stages*/, TileShape, T, Stride<Int<1>, Int<0>, Int<0>>>;

  template <typename T>
  using RowOrScalarLoad = cutlass::epilogue::fusion::Sm90RowOrScalarBroadcast<
      0 /*Stages*/, TileShape, T, Stride<Int<0>, Int<1>, Int<0>>>;

  // Don't want to support nullptr by default
  template <typename T, bool EnableNullPtr = false>
  using ColLoad = cutlass::epilogue::fusion::Sm90ColBroadcast<
      0 /*Stages*/, TileShape, T, T, Stride<Int<1>, Int<0>, Int<0>>,
      128 / sizeof_bits_v<T>, EnableNullPtr>;

  // Don't want to support nullptr by default
  template <typename T, bool EnableNullPtr = false>
  using RowLoad = cutlass::epilogue::fusion::Sm90RowBroadcast<
      0 /*Stages*/, TileShape, T, T, Stride<Int<0>, Int<1>, Int<0>>,
      128 / sizeof_bits_v<T>, EnableNullPtr>;

  template <typename T>
  using ColOrScalarLoadArray =
      cutlass::epilogue::fusion::Sm90ColOrScalarBroadcastArray<
          0 /*Stages*/, TileShape, T, Stride<Int<1>, Int<0>, Int<0>>>;

  template <typename T>
  using RowOrScalarLoadArray =
      cutlass::epilogue::fusion::Sm90RowOrScalarBroadcastArray<
          0 /*Stages*/, TileShape, T, Stride<Int<0>, Int<1>, Int<0>>>;

  // This utility function constructs the arguments for the load descriptors
  // from a tensor. It can handle both row and column, as well as row/column or
  // scalar cases.
  template <typename Descriptor, typename T>
  static auto args_from_tensor(TensorType const& tensor) {
    using Arguments = typename Descriptor::Arguments;
    auto* data_ptr = static_cast<T*>(tensor.data_ptr());
    if constexpr (std::is_same_v<Descriptor, ColOrScalarLoad<T>> ||
                  std::is_same_v<Descriptor, RowOrScalarLoad<T>>) {
      return Arguments{data_ptr, tensor.numel() != 1};
    } else {
      static_assert(!std::is_same_v<Descriptor, ColLoad<T, true>> &&
                    !std::is_same_v<Descriptor, RowLoad<T, true>>);
      return Arguments{data_ptr};
    }
  }

  // This overload handles the case where there might not be a tensor, in which
  // case a nullptr is passed and a constant (0) is used.
  template <typename Descriptor, typename T>
  static auto args_from_tensor(std::optional<TensorType> const& tensor) {
    using Arguments = typename Descriptor::Arguments;
    auto* data_ptr = tensor ? static_cast<T*>(tensor->data_ptr()) : nullptr;
    static_assert(std::is_same_v<Descriptor, ColLoad<T, true>> ||
                  std::is_same_v<Descriptor, RowLoad<T, true>>);
    return Arguments{data_ptr};
  }

  template <typename Descriptor, typename T>
  static auto args_from_tensor(const T* const* data_ptr, bool do_broadcast) {
    using Arguments = typename Descriptor::Arguments;
    static_assert(std::is_same_v<Descriptor, ColOrScalarLoadArray<T>> ||
                  std::is_same_v<Descriptor, RowOrScalarLoadArray<T>>);
    return Arguments{data_ptr, do_broadcast};
  }
};
```
**EN:** `ScaledEpilogueBase` centralizes descriptor aliases for row, column, scalar, nullable, and array-based broadcasts. Its overloaded `args_from_tensor` helpers turn Torch tensors, optional tensors, or pointer arrays into the exact CUTLASS descriptor argument structs expected by the EVT tree. This keeps the later epilogue definitions focused on math composition instead of plumbing.
**CN:** `ScaledEpilogueBase` 集中定义了行、列、标量、可空以及数组式广播描述符别名。它提供的多组 `args_from_tensor` 重载可把 Torch 张量、可选张量或指针数组转换成 EVT 树所需的 CUTLASS 描述符参数结构。这样后续 epilogue 定义就能专注于数学组合，而不必重复处理参数装配细节。

### Scaled GEMM without bias / 不带 bias 的 scaled GEMM
```cpp
template <typename ElementAcc, typename ElementD, typename TileShape>
struct ScaledEpilogue
    : private ScaledEpilogueBase<ElementAcc, ElementD, TileShape> {
 private:
  using SUPER = ScaledEpilogueBase<ElementAcc, ElementD, TileShape>;
  using Accum = typename SUPER::Accum;
  using ScaleA = typename SUPER::template ColOrScalarLoad<float>;
  using ScaleB = typename SUPER::template RowOrScalarLoad<float>;

  using Compute0 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, float, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTCompute0 =
      cutlass::epilogue::fusion::Sm90EVT<Compute0, ScaleB, Accum>;

  using Compute1 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, ElementD, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

 public:
  using EVTCompute =
      cutlass::epilogue::fusion::Sm90EVT<Compute1, ScaleA, EVTCompute0>;
  using ArgumentType = typename EVTCompute::Arguments;

  static ArgumentType prepare_args(TensorType const& a_scales,
                                   TensorType const& b_scales) {
    auto a_args = SUPER::template args_from_tensor<ScaleA, float>(a_scales);
    auto b_args = SUPER::template args_from_tensor<ScaleB, float>(b_scales);

    typename EVTCompute0::Arguments evt0_args{b_args, {}, {}};
    return ArgumentType{a_args, evt0_args, {}};
  }
};
```
**EN:** `ScaledEpilogue` composes two multiply stages: `ScaleB * Accum`, then `ScaleA * (...)`, with the final output rounded into `ElementD`. `prepare_args` builds the nested EVT argument tree in the same order, first for B-side scales and then for A-side scales. Because the broadcast descriptors can represent scalars or vectors, the same epilogue supports per-tensor and per-row/per-column quantization.
**CN:** `ScaledEpilogue` 组合了两层乘法：先计算 `ScaleB * Accum`，再计算 `ScaleA * (...)`，最后把结果按 `ElementD` 的类型进行舍入输出。`prepare_args` 也按照相同顺序构造嵌套 EVT 参数树：先装配 B 侧 scale，再装配 A 侧 scale。由于广播描述符既能表示标量也能表示向量，这个 epilogue 同时支持 per-tensor 与 per-row/per-column 量化。

### Scaled GEMM with row bias / 带行 bias 的 scaled GEMM
```cpp
template <typename ElementAcc, typename ElementD, typename TileShape>
struct ScaledEpilogueBias
    : private ScaledEpilogueBase<ElementAcc, ElementD, TileShape> {
 private:
  using SUPER = ScaledEpilogueBase<ElementAcc, ElementD, TileShape>;
  using Accum = typename SUPER::Accum;
  using ScaleA = typename SUPER::template ColOrScalarLoad<float>;
  using ScaleB = typename SUPER::template RowOrScalarLoad<float>;
  using Bias = typename SUPER::template RowLoad<ElementD>;

  using Compute0 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, float, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTCompute0 =
      cutlass::epilogue::fusion::Sm90EVT<Compute0, ScaleB, Accum>;

  using Compute1 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::homogeneous_multiply_add, ElementD, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

 public:
  using EVTCompute =
      cutlass::epilogue::fusion::Sm90EVT<Compute1, ScaleA, EVTCompute0, Bias>;

  using ArgumentType = typename EVTCompute::Arguments;
  static ArgumentType prepare_args(TensorType const& a_scales,
                                   TensorType const& b_scales,
                                   TensorType const& bias) {
    auto a_args = SUPER::template args_from_tensor<ScaleA, float>(a_scales);
    auto b_args = SUPER::template args_from_tensor<ScaleB, float>(b_scales);
    auto bias_args = SUPER::template args_from_tensor<Bias, ElementD>(bias);

    typename EVTCompute0::Arguments evt0_args{b_args, {}, {}};
    return ArgumentType{a_args, evt0_args, bias_args, {}};
  }
```
**EN:** `ScaledEpilogueBias` adds a row-wise bias descriptor and switches the final compute op to `homogeneous_multiply_add`, meaning it multiplies by `ScaleA` and then adds bias in one fused stage. This is the natural form for dense linear layers whose bias is indexed by output channel.
**CN:** `ScaledEpilogueBias` 新增了按行广播的 bias 描述符，并把最终计算节点切换成 `homogeneous_multiply_add`，也就是在同一个融合阶段里先乘 `ScaleA` 再加 bias。这正是常规线性层按输出通道索引 bias 的自然形式。

### Scaled GEMM with column bias / 带列 bias 的 scaled GEMM
```cpp
template <typename ElementAcc, typename ElementD, typename TileShape>
struct ScaledEpilogueColumnBias
    : private ScaledEpilogueBase<ElementAcc, ElementD, TileShape> {
 private:
  using SUPER = ScaledEpilogueBase<ElementAcc, ElementD, TileShape>;
  using Accum = typename SUPER::Accum;
  using ScaleA = typename SUPER::template ColOrScalarLoad<float>;
  using ScaleB = typename SUPER::template RowOrScalarLoad<float>;
  using Bias = typename SUPER::template ColLoad<ElementD>;

  using Compute0 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, float, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTCompute0 =
      cutlass::epilogue::fusion::Sm90EVT<Compute0, ScaleB, Accum>;

  using Compute1 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::homogeneous_multiply_add, ElementD, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

 public:
  using EVTCompute =
      cutlass::epilogue::fusion::Sm90EVT<Compute1, ScaleA, EVTCompute0, Bias>;

  using ArgumentType = typename EVTCompute::Arguments;
  static ArgumentType prepare_args(TensorType const& a_scales,
                                   TensorType const& b_scales,
                                   TensorType const& bias) {
    auto a_args = SUPER::template args_from_tensor<ScaleA, float>(a_scales);
    auto b_args = SUPER::template args_from_tensor<ScaleB, float>(b_scales);
    auto bias_args = SUPER::template args_from_tensor<Bias, ElementD>(bias);

    typename EVTCompute0::Arguments evt0_args{b_args, {}, {}};
    return ArgumentType{a_args, evt0_args, bias_args, {}};
  }
```
**EN:** `ScaledEpilogueColumnBias` is almost identical to the row-bias version except that its bias is loaded as a column vector. The comment calls out the main use case: transposed or sparse formulations where the output tile orientation makes column-wise bias the correct broadcast direction.
**CN:** `ScaledEpilogueColumnBias` 与行 bias 版本几乎完全相同，只是 bias 改为按列向量加载。注释中指出了主要用途：在转置或稀疏形式下，输出 tile 的朝向发生变化，此时列方向广播才是正确的 bias 语义。

### Per-tensor AZP correction / 每张量 AZP 修正
```cpp
template <typename ElementAcc, typename ElementD, typename TileShape>
struct ScaledEpilogueBiasAzp
    : private ScaledEpilogueBase<ElementAcc, ElementD, TileShape> {
 private:
  using SUPER = ScaledEpilogueBase<ElementAcc, ElementD, TileShape>;
  using Accum = typename SUPER::Accum;
  using ScaleA = typename SUPER::template ColOrScalarLoad<float>;
  using ScaleB = typename SUPER::template RowOrScalarLoad<float>;
  using Bias = typename SUPER::template RowLoad<ElementD, true>;

  // This is the full AZP term, azp * J @ B, shape (1,n)
  using AzpWithAdj = typename SUPER::template RowLoad<int32_t>;

  // Compute float(accum - azp_adj), both operands are int32_t
  using ComputeAzp = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::minus, float, int32_t,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTComputeAzp =
      cutlass::epilogue::fusion::Sm90EVT<ComputeAzp, Accum, AzpWithAdj>;

  using ComputeScaleB = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, float, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTComputeScaleB =
      cutlass::epilogue::fusion::Sm90EVT<ComputeScaleB, ScaleB, EVTComputeAzp>;

  using ComputeScaleBiasA = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::homogeneous_multiply_add, ElementD, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

 public:
  using EVTCompute =
      cutlass::epilogue::fusion::Sm90EVT<ComputeScaleBiasA, ScaleA,
                                         EVTComputeScaleB, Bias>;
  using ArgumentType = typename EVTCompute::Arguments;

  static ArgumentType prepare_args(TensorType const& a_scales,
                                   TensorType const& b_scales,
                                   TensorType const& azp_adj,
                                   std::optional<TensorType> const& bias) {
    auto a_args = SUPER::template args_from_tensor<ScaleA, float>(a_scales);
    auto b_args = SUPER::template args_from_tensor<ScaleB, float>(b_scales);
    auto bias_args = SUPER::template args_from_tensor<Bias, ElementD>(bias);
    auto azp_adj_args =
        SUPER::template args_from_tensor<AzpWithAdj, int32_t>(azp_adj);

    typename EVTComputeAzp::Arguments evt_azp_args{{}, azp_adj_args, {}};
    typename EVTComputeScaleB::Arguments evt_scale_b_args{
        b_args, evt_azp_args, {}};
    return ArgumentType{a_args, evt_scale_b_args, bias_args, {}};
  }
```
**EN:** `ScaledEpilogueBiasAzp` handles activation zero-point correction when a single scalar azp has already been folded into a precomputed row vector `azp_adj = azp * J @ B`. The EVT tree first subtracts that correction from the accumulator, then applies B scales, then A scales plus optional bias. Bias is nullable here, so the base helper is used with `EnableNullPtr = true`.
**CN:** `ScaledEpilogueBiasAzp` 处理每张量激活零点修正：标量 azp 预先折叠进一个行向量 `azp_adj = azp * J @ B` 中。其 EVT 树会先从累加值中减去这个修正项，再乘 B 侧 scale，最后乘 A 侧 scale 并加上可选 bias。这里的 bias 是可空的，因此基类辅助函数使用了 `EnableNullPtr = true` 的版本。

### Per-token AZP correction / 每 token 的 AZP 修正
```cpp
template <typename ElementAcc, typename ElementD, typename TileShape>
struct ScaledEpilogueBiasAzpToken
    : private ScaledEpilogueBase<ElementAcc, ElementD, TileShape> {
 private:
  using SUPER = ScaledEpilogueBase<ElementAcc, ElementD, TileShape>;
  using Accum = typename SUPER::Accum;
  using ScaleA = typename SUPER::template ColOrScalarLoad<float>;
  using ScaleB = typename SUPER::template RowOrScalarLoad<float>;
  using Bias = typename SUPER::template RowLoad<ElementD, true>;

  // Per-token azp term, shape (m,1)
  using Azp = typename SUPER::template ColLoad<int32_t>;

  // This is the AZP adjustment term, J @ B, shape (1,n)
  using AzpAdj = typename SUPER::template RowLoad<int32_t>;

  // Compute azp * azp_adj
  using ComputeAzp = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, int32_t, int32_t,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTComputeAzp =
      cutlass::epilogue::fusion::Sm90EVT<ComputeAzp, Azp, AzpAdj>;

  // Compute float(accum - azp*azp_adj), all operands are int32_t
  using ComputeAcc = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::minus, float, int32_t,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTComputeAcc =
      cutlass::epilogue::fusion::Sm90EVT<ComputeAcc, Accum, EVTComputeAzp>;

  using ComputeScaleB = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, float, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTComputeScaleB =
      cutlass::epilogue::fusion::Sm90EVT<ComputeScaleB, ScaleB, EVTComputeAcc>;

  using ComputeScaleBiasA = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::homogeneous_multiply_add, ElementD, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

 public:
  using EVTCompute =
      cutlass::epilogue::fusion::Sm90EVT<ComputeScaleBiasA, ScaleA,
                                         EVTComputeScaleB, Bias>;
  using ArgumentType = typename EVTCompute::Arguments;

  static ArgumentType prepare_args(TensorType const& a_scales,
                                   TensorType const& b_scales,
                                   TensorType const& azp_adj,
                                   TensorType const& azp,
                                   std::optional<TensorType> const& bias) {
    auto a_args = SUPER::template args_from_tensor<ScaleA, float>(a_scales);
    auto b_args = SUPER::template args_from_tensor<ScaleB, float>(b_scales);
    auto bias_args = SUPER::template args_from_tensor<Bias, ElementD>(bias);
    auto azp_args = SUPER::template args_from_tensor<Azp, int32_t>(azp);
    auto azp_adj_args =
        SUPER::template args_from_tensor<AzpAdj, int32_t>(azp_adj);

    typename EVTComputeAzp::Arguments evt_azp_args{azp_args, azp_adj_args, {}};
    typename EVTComputeAcc::Arguments evt_acc_args{{}, evt_azp_args, {}};
    typename EVTComputeScaleB::Arguments evt_scale_b_args{
        b_args, evt_acc_args, {}};
    return ArgumentType{a_args, evt_scale_b_args, bias_args, {}};
  }
```
**EN:** `ScaledEpilogueBiasAzpToken` supports a stronger form of quantization where each input row has its own zero point. Instead of materializing the full correction matrix, it computes the rank-1 term `azp * azp_adj` inside the epilogue, subtracts it from the accumulator, and then applies the same scale-and-bias pipeline. This reduces the extra storage from O(m*n) to O(m+n).
**CN:** `ScaledEpilogueBiasAzpToken` 支持更细粒度的量化形式：每个输入行都有自己的零点。它不会显式物化完整修正矩阵，而是在 epilogue 内部计算秩 1 修正项 `azp * azp_adj`，再从累加值中减去，最后继续走相同的 scale 与 bias 流水线。这样额外存储从 O(m*n) 降到了 O(m+n)。

### Group GEMM scale arrays / Group GEMM 的 scale 指针数组
```cpp
template <typename ElementAcc, typename ElementD, typename EpilogueDescriptor>
struct ScaledEpilogueArray
    : private ScaledEpilogueBase<ElementAcc, ElementD, EpilogueDescriptor> {
 private:
  using SUPER = ScaledEpilogueBase<ElementAcc, ElementD, EpilogueDescriptor>;
  using Accum = typename SUPER::Accum;
  using ScaleA = typename SUPER::template ColOrScalarLoadArray<float>;
  using ScaleB = typename SUPER::template RowOrScalarLoadArray<float>;

  using Compute0 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, float, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTCompute0 =
      cutlass::epilogue::fusion::Sm90EVT<Compute0, ScaleB, Accum>;

  using Compute1 = cutlass::epilogue::fusion::Sm90Compute<
      cutlass::multiplies, ElementD, float,
      cutlass::FloatRoundStyle::round_to_nearest>;

 public:
  using EVTCompute =
      cutlass::epilogue::fusion::Sm90EVT<Compute1, ScaleA, EVTCompute0>;
  using ArgumentType = typename EVTCompute::Arguments;

  using ScaleAArray = typename SUPER::template ColOrScalarLoadArray<float>;
  using ScaleBArray = typename SUPER::template RowOrScalarLoadArray<float>;

  static ArgumentType prepare_args(float const* const* a_scales_ptr,
                                   float const* const* b_scales_ptr,
                                   bool a_col_broadcast, bool b_row_broadcast) {
    auto a_args = SUPER::template args_from_tensor<ScaleAArray, float>(
        a_scales_ptr, a_col_broadcast);
    auto b_args = SUPER::template args_from_tensor<ScaleBArray, float>(
        b_scales_ptr, b_row_broadcast);

    typename EVTCompute0::Arguments evt0_args{b_args, {}, {}};
    return ArgumentType{a_args, evt0_args, {}};
```
**EN:** `ScaledEpilogueArray` adapts the same scale-only structure to group GEMM, where each group has different scale tensors and the kernel receives arrays of pointers. The broadcast-or-scalar booleans are preserved because each pointed-to tensor may still be either a true vector or a scalar.
**CN:** `ScaledEpilogueArray` 把同样的“仅 scale”结构扩展到了 group GEMM：每个 group 都可能对应不同的 scale 张量，因此 kernel 接收的是指针数组。是否按广播处理的布尔标记也被保留下来，因为每个指针指向的对象仍可能是真正的向量，也可能是标量。

## Key Concepts / 关键概念
- CUTLASS 3.x EVT trees for composing epilogue math / 使用 CUTLASS 3.x EVT 树组合 epilogue 数学操作
- Unified handling of scalar, row, column, optional, and grouped scale/bias inputs / 统一处理标量、行、列、可选以及分组的 scale/bias 输入
- In-epilogue activation-zero-point correction for quantized GEMM / 在 epilogue 内完成量化 GEMM 的激活零点修正

## Dependencies / 依赖关系
- Depends on custom broadcast descriptors from `broadcast_load_epilogue_c3x.hpp` and `broadcast_load_epilogue_array_c3x.hpp` / 依赖 `broadcast_load_epilogue_c3x.hpp` 与 `broadcast_load_epilogue_array_c3x.hpp` 中的自定义广播描述符
- Uses CUTLASS Sm90 fusion primitives such as `Sm90EVT`, `Sm90Compute`, and broadcast loaders / 使用 CUTLASS 的 `Sm90EVT`、`Sm90Compute` 与广播加载器等 Sm90 融合原语
- Consumes Torch tensor handles or pointer arrays to build runtime epilogue arguments / 消费 Torch 张量句柄或指针数组来构建运行时 epilogue 参数
