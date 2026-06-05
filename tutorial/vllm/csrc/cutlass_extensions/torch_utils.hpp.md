# torch_utils.hpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `csrc/cutlass_extensions/torch_utils.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Bridges Torch tensor metadata and scalar types into CuTe/CUTLASS layouts while supporting both stable and unstable libtorch ABIs. / [CN] 在支持稳定与非稳定 libtorch ABI 的前提下，把 Torch 张量元数据和标量类型桥接到 CuTe/CUTLASS 布局体系。

## Line-by-Line Analysis / 逐行分析
### ABI-dependent Torch aliases / ABI 相关的 Torch 别名
```cpp
// This header is shared between _C (unstable ABI, used by machete) and
// _C_stable_libtorch (stable ABI, used by W4A8/sparse). TORCH_TARGET_VERSION
// is defined only for the stable target, so we switch includes and types
// accordingly. TorchTensor (not Tensor) avoids ambiguity with cute::Tensor.
#ifdef TORCH_TARGET_VERSION
  #include <torch/csrc/stable/tensor.h>
  #include <torch/headeronly/util/BFloat16.h>
  #include <torch/headeronly/util/Half.h>
  #include <torch/headeronly/util/shim_utils.h>  // for STD_TORCH_CHECK
using TorchTensor = torch::stable::Tensor;
  #define TORCH_UTILS_CHECK STD_TORCH_CHECK
#else
  #include <torch/all.h>
using TorchTensor = torch::Tensor;
  #define TORCH_UTILS_CHECK TORCH_CHECK
#endif
```
**EN:** The header switches between `torch::stable::Tensor` and `torch::Tensor` depending on `TORCH_TARGET_VERSION`, and maps the appropriate assertion macro to `TORCH_UTILS_CHECK`. This allows one helper header to serve both the unstable `_C` target and the stable-ABI extension target without code duplication.
**CN:** 该头文件会根据 `TORCH_TARGET_VERSION` 在 `torch::stable::Tensor` 与 `torch::Tensor` 之间切换，并把对应的断言宏统一映射成 `TORCH_UTILS_CHECK`。这样同一份辅助头文件就能同时服务于不稳定 ABI 的 `_C` 目标和稳定 ABI 的扩展目标，而无需复制代码。

### Indexed CuTe tuple helpers / 带索引的 CuTe 元组辅助函数
```cpp
namespace detail {

template <class T, class F, class G, int... I>
CUTE_HOST_DEVICE constexpr auto tapply_with_idx(T&& t, F&& f, G&& g,
                                                seq<I...>) {
  return g(f(cute::get<I>(static_cast<T&&>(t)), I)...);
}

template <class F, int... I>
CUTE_HOST_DEVICE constexpr auto make_shape_from_idx(F&& f, seq<I...>) {
  return make_shape(f(I)...);
}

};  // namespace detail

template <class T, class F>
CUTE_HOST_DEVICE constexpr auto transform_with_idx(T const& t, F&& f) {
  if constexpr (cute::is_tuple<T>::value) {
    return detail::tapply_with_idx(
        t, f, [](auto const&... a) { return cute::make_tuple(a...); },
        tuple_seq<T>{});
  } else {
    return f(t);
  }

  CUTE_GCC_UNREACHABLE;
}

// calls: make_shape(f(0), f(1), ..., f(N-1))
template <int N, class F>
CUTE_HOST_DEVICE constexpr auto make_shape_from_idx(F&& f) {
  return detail::make_shape_from_idx(f, make_seq<N>{});
}
```
**EN:** The `detail` helpers and `transform_with_idx`/`make_shape_from_idx` add index-aware transformations over CuTe tuples. Standard CuTe transforms do not always expose the dimension index, but this file needs the index to compare Torch strides and sizes against a compile-time stride descriptor.
**CN:** `detail` 命名空间中的辅助函数以及 `transform_with_idx`/`make_shape_from_idx` 为 CuTe 元组提供了“带维度索引”的变换能力。标准的 CuTe 变换并不总是暴露维度下标，但这里必须知道维度索引，才能把 Torch 的 stride/size 与编译期的 stride 描述符逐维比较。

### Torch tensor to CuTe layout / Torch 张量到 CuTe 布局
```cpp
template <typename Stride>
static inline auto make_cute_layout(TorchTensor const& tensor,
                                    std::string_view name = "tensor") {
  TORCH_UTILS_CHECK(tensor.dim() <= rank(Stride{}));
  auto stride = cute::transform_with_idx(Stride{}, [&](auto const& stride_ele,
                                                       auto const& idx) {
    using StrideEle = std::decay_t<decltype(stride_ele)>;

    if (idx < tensor.dim()) {
      if constexpr (cute::is_static_v<StrideEle>) {
        TORCH_UTILS_CHECK(StrideEle::value == tensor.stride(idx), "Expected ",
                          name, ".stride(", idx, ") to be ", StrideEle::value);
        return StrideEle{};
      } else {
        if (tensor.size(idx) == 1) {
          // use 0 stride for dim with size 1, this is easier for
          // cute/cutlass to optimize (helps the TMA code flatten dims)
          return StrideEle{0};
        } else {
          return tensor.stride(idx);
        }
      }
    } else {
      // Extra strides are assumed to be 0 or 1
      if constexpr (cute::is_static_v<StrideEle>) {
        static_assert(StrideEle::value == 0 || StrideEle::value == 1);
      }
      return StrideEle{};
    }
  });

  auto shape = cute::make_shape_from_idx<rank(Stride{})>([&](auto const& idx) {
    if (idx < tensor.dim())
      return tensor.size(idx);
    else
      return int64_t(1);
  });

  return make_layout(shape, stride);
}
```
**EN:** `make_cute_layout` is the core bridge. It verifies the tensor rank does not exceed the target stride rank, preserves static stride requirements, converts dynamic strides from the tensor, collapses size-1 dimensions to stride 0 for better CUTLASS/CuTe optimization, pads missing dimensions with extent 1, and finally returns `make_layout(shape, stride)`.
**CN:** `make_cute_layout` 是最核心的桥接函数。它先验证张量 rank 不超过目标 stride 的 rank，再保留静态 stride 约束、从张量读取动态 stride，并把 size 为 1 的维度改写成 stride 0，以便 CUTLASS/CuTe 做更好的优化；缺失维度则补成 extent 1，最后返回 `make_layout(shape, stride)`。

### Optional layout construction / 可选布局构造
```cpp
template <typename Stride>
static inline auto maybe_make_cute_layout(
    std::optional<TorchTensor> const& tensor,
    std::string_view name = "tensor") {
  using Layout = decltype(make_cute_layout<Stride>(*tensor));

  if (tensor) {
    return std::optional<Layout>{make_cute_layout<Stride>(*tensor, name)};
  } else {
    return std::optional<Layout>{};
  }
}
```
**EN:** `maybe_make_cute_layout` wraps the previous routine in `std::optional`, returning an empty optional when the tensor is absent. That pattern matches epilogues or kernels where an auxiliary operand such as bias may be optional.
**CN:** `maybe_make_cute_layout` 用 `std::optional` 封装了前面的逻辑；当张量不存在时直接返回空 optional。这非常适合 bias 等辅助操作数可选的 kernel/epilogue 场景。

### Torch-to-CUTLASS scalar mapping / Torch 到 CUTLASS 的标量映射
```cpp
template <typename T>
struct equivalent_cutlass_type {
  using type = T;
};

template <typename T>
using equivalent_cutlass_type_t = typename equivalent_cutlass_type<T>::type;

template <>
struct equivalent_cutlass_type<torch::headeronly::Half> {
  using type = cutlass::half_t;
};

template <>
struct equivalent_cutlass_type<torch::headeronly::BFloat16> {
  using type = cutlass::bfloat16_t;
```
**EN:** `equivalent_cutlass_type` maps Torch header-only scalar wrappers like `Half` and `BFloat16` to the CUTLASS scalar types expected by kernels. The primary template leaves unrelated types untouched.
**CN:** `equivalent_cutlass_type` 把 `Half`、`BFloat16` 这类 Torch header-only 标量包装类型映射成 kernel 所期望的 CUTLASS 标量类型。主模板则保持其他无关类型不变。

### CUTLASS-to-Torch scalar mapping / CUTLASS 到 Torch 的标量映射
```cpp
// Return a `torch::headeronly::CppTypeToScalarType<T>` compatible type, i.e.
// get the C++ type equivalent to T, e.g.: `cutlass::half_t -> Half`
template <typename T>
struct equivalent_scalar_type {
  using type = T;
};

template <typename T>
using equivalent_scalar_type_t = typename equivalent_scalar_type<T>::type;

template <>
struct equivalent_scalar_type<cutlass::half_t> {
  using type = torch::headeronly::Half;
};

template <>
struct equivalent_scalar_type<cutlass::bfloat16_t> {
  using type = torch::headeronly::BFloat16;
};

// get equivalent torch::headeronly::ScalarType tag from compile time type
template <typename T>
static inline constexpr torch::headeronly::ScalarType equivalent_scalar_type_v =
    torch::headeronly::CppTypeToScalarType<equivalent_scalar_type_t<T>>::value;
```
**EN:** `equivalent_scalar_type` performs the inverse mapping, and `equivalent_scalar_type_v` turns the result into a `torch::headeronly::ScalarType` tag. This is useful when template code knows the CUTLASS type at compile time but still needs to create or validate Torch tensors.
**CN:** `equivalent_scalar_type` 实现了反向映射，而 `equivalent_scalar_type_v` 则进一步把结果转成 `torch::headeronly::ScalarType` 标记。当模板代码只在编译期知道 CUTLASS 类型、但仍需创建或校验 Torch 张量时，这个工具就很实用。

## Key Concepts / 关键概念
- Stable-vs-unstable libtorch ABI compatibility / 稳定与非稳定 libtorch ABI 兼容层
- Conversion from runtime tensor metadata to compile-time-friendly CuTe layouts / 从运行时张量元数据转换到适合编译期模板处理的 CuTe 布局
- Bidirectional mapping between Torch scalar wrappers and CUTLASS scalar types / Torch 标量包装类型与 CUTLASS 标量类型之间的双向映射

## Dependencies / 依赖关系
- Torch stable or regular tensor headers provide `Tensor` and scalar wrappers / Torch 稳定或常规张量头提供 `Tensor` 及标量包装类型
- `cute/layout.hpp` and CUTLASS layout headers provide rank, shape, and matrix-layout types / `cute/layout.hpp` 与 CUTLASS 布局头提供 rank、shape 和矩阵布局类型
- Used by CUTLASS extension entry points that consume Torch tensors / 被使用 Torch 张量作为输入的 CUTLASS 扩展入口调用
