# Conv.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/detail/Conv.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on conv; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 conv；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <c10/xpu/XPUFunctions.h>

#include <ATen/ATen.h>
#include <ATen/core/grad_mode.h>
#include <ATen/record_function.h>
#include <c10/core/MemoryFormat.h>

#include <ATen/native/mkldnn/xpu/detail/Attr.h>
#include <ATen/native/mkldnn/xpu/detail/Utils.h>

#include <oneapi/dnnl/dnnl.hpp>

namespace at::native::onednn {

constexpr int src_batch_size_dim = 0;
constexpr int weight_dst_channels_dim = 0;

dnnl::memory::dims conv_dst_size(
    int64_t ndim,
    IntArrayRef src_size,
    IntArrayRef weight_size,
    IntArrayRef padding_front_top_left,
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation) {
  bool has_dilation = dilation.size() > 0;
  dnnl::memory::dims dst_size(ndim);
  dst_size[0] = src_size[src_batch_size_dim];
  dst_size[1] = weight_size[weight_dst_channels_dim];
  for (int d = 2; d < ndim; ++d) {
    auto dilate = has_dilation ? dilation[d - 2] : 1;
    auto kernel = dilate * (weight_size[d] - 1) + 1;
    dst_size[d] =
        (src_size[d] +
         (padding_front_top_left[d - 2] + padding_back_bottom_right[d - 2]) -
         kernel) /
            stride[d - 2] +
        1;
  }
  return dst_size;
}

static std::tuple<dnnl::memory::desc, dnnl::memory::desc, dnnl::memory::desc>
conv_get_md(
    const at::Tensor& src,
    const at::Tensor& weight,
    const at::Tensor& dst,
    int64_t groups,
    bool is_channels_last) {
  // create memory desc from the src/weight/dst tensors
  dnnl::memory::desc src_usr_md, weight_usr_md, dst_usr_md;
  auto ndim = src.ndimension();
  auto fmt_src = conv_src_fmt(ndim, is_channels_last);

  auto src_size = src.sizes().vec();
  auto src_data_t = get_onednn_dtype_include_double(src);
  src_usr_md = dnnl::memory::desc(src_size, src_data_t, fmt_src);

  auto dst_size = dst.sizes().vec();
  auto dst_data_t = get_onednn_dtype_include_double(dst);
```
- EN: Lines 1-60 pull in 8 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 第 1-60 行引入了 8 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 61-120
```cpp
  dst_usr_md = dnnl::memory::desc(dst_size, dst_data_t, fmt_src);

  auto ic = src.size(1);
  auto oc = dst.size(1);
  auto wei_data_t = get_onednn_dtype_include_double(weight);
  dnnl::memory::dims weight_size =
      compatible_weight_dims(ndim, groups, oc, ic, weight.sizes());
  auto fmt_weight = conv_weight_fmt(ndim, groups != 1, is_channels_last);
  weight_usr_md = dnnl::memory::desc(weight_size, wei_data_t, fmt_weight);

  return {src_usr_md, weight_usr_md, dst_usr_md};
}

sycl::event convolution(
    at::Tensor& dst,
    const at::Tensor& src,
    const at::Tensor& weight,
    const at::Tensor& bia,
    IntArrayRef padding_front_top_left,
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    Attr& attr,
    const std::vector<sycl::event>& deps) {
  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();

  bool is_channels_last = use_channels_last_for_conv(src, weight);

  // create usr_md for tensors, and md for conv primitive
  auto [src_md, weight_md, dst_md] =
      conv_get_md(src, weight, dst, groups, is_channels_last);

  auto bia_fmt = dnnl::memory::format_tag::x;
  auto bia_md = bia.defined()
      ? dnnl::memory::desc(
            {dst.size(1)}, get_onednn_dtype_include_double(bia), bia_fmt)
      : dnnl::memory::desc();

  // create conv primitive descriptor
  dnnl::memory::dims _stride = stride.vec();
  dnnl::memory::dims _padding_front_top_left = padding_front_top_left.vec();
  dnnl::memory::dims _padding_back_bottom_right =
      padding_back_bottom_right.vec();
  dnnl::memory::dims _dilation = compatible_dilation(dilation);

  // extract post ops
  dnnl::primitive_attr pattr;
  dnnl::post_ops po = attr.extract_post_ops(dst);
  pattr.set_post_ops(po);

  pattr.set_scratchpad_mode(dnnl::scratchpad_mode::user);

#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn()) {
    pattr.set_deterministic(true);
  }
#endif
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are defined, get_onednn_dtype_include_double, globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 defined, get_onednn_dtype_include_double, globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-180
```cpp

  at::native::onednn::apply_tf32_if_allowed(pattr);

  auto conv_fwd_pd = dnnl::convolution_forward::primitive_desc(
      engine,
      dnnl::prop_kind::forward,
      dnnl::algorithm::convolution_direct,
      src_md,
      weight_md,
      bia_md,
      dst_md,
      _stride,
      _dilation,
      _padding_front_top_left,
      _padding_back_bottom_right,
      pattr);

  dnnl::memory src_m, weight_m, dst_m, bia_m;
  at::Tensor src_blocked, weight_blocked, dst_blocked = dst;

  src_m = make_onednn_memory(src_md, engine, src.data_ptr());
  weight_m = make_onednn_memory(weight_md, engine, weight.data_ptr());
  dst_m = make_onednn_memory(dst_md, engine, dst.data_ptr());

  std::unordered_map<int, dnnl::memory> args;
  if (bia.defined()) {
    bia_m = make_onednn_memory(bia_md, engine, bia.data_ptr());
    args.insert({DNNL_ARG_BIAS, bia_m});
  }
  auto expected_dst_md = conv_fwd_pd.dst_desc();
  if (attr.with_binary())
    attr.construct_post_binary(conv_fwd_pd, args);

  args.insert({DNNL_ARG_SRC, src_m});
  args.insert({DNNL_ARG_WEIGHTS, weight_m});
  args.insert({DNNL_ARG_DST, dst_m});

  size_t scratchpad_size = conv_fwd_pd.scratchpad_desc().get_size();
  at::Tensor scratchpad_tensor = at::empty(
      {static_cast<int64_t>(scratchpad_size)},
      src.options().dtype(at::kByte),
      std::nullopt);
  auto scratchpad_m = make_onednn_memory(
      conv_fwd_pd.scratchpad_desc(), engine, scratchpad_tensor.data_ptr());
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad_m});

  auto conv_forward = dnnl::convolution_forward(conv_fwd_pd);
  auto conv_fwd_event =
      dnnl::sycl_interop::execute(conv_forward, stream, args, deps);

  return conv_fwd_event;
}

sycl::event convolution_backward_weights(
    at::Tensor& diff_weight,
    at::Tensor& diff_bia,
    const at::Tensor& diff_dst,
    const at::Tensor& src,
    IntArrayRef diff_weight_aten_size,
    IntArrayRef padding_front_top_left,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-240
```cpp
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    const std::vector<sycl::event>& deps) {
  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();

  bool is_channels_last = use_channels_last_for_conv(src, diff_dst);

  // create dnnl::memory desc
  auto [src_md, weight_md, dst_md] =
      conv_get_md(src, diff_weight, diff_dst, groups, is_channels_last);
  dnnl::memory::format_tag bia_fmt = dnnl::memory::format_tag::x;
  auto bia_md = diff_bia.defined()
      ? dnnl::memory::desc({diff_dst.size(1)}, src_md.get_data_type(), bia_fmt)
      : dnnl::memory::desc();

  // create fwd primitive hint
  dnnl::memory::dims _stride = stride.vec();
  dnnl::memory::dims _padding_front_top_left = padding_front_top_left.vec();
  dnnl::memory::dims _padding_back_bottom_right =
      padding_back_bottom_right.vec();
  dnnl::memory::dims _dilation = compatible_dilation(dilation);
  dnnl::primitive_attr pattr;

#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn()) {
    pattr.set_deterministic(true);
  }
#endif

  at::native::onednn::apply_tf32_if_allowed(pattr);

  pattr.set_scratchpad_mode(dnnl::scratchpad_mode::user);
  auto conv_fwd_pd = dnnl::convolution_forward::primitive_desc(
      engine,
      dnnl::prop_kind::forward,
      dnnl::algorithm::convolution_direct,
      src_md,
      weight_md,
      bia_md,
      dst_md,
      _stride,
      _dilation,
      _padding_front_top_left,
      _padding_back_bottom_right,
      pattr);

  // create bwd weight primitive
  auto conv_bwd_w_pd = dnnl::convolution_backward_weights::primitive_desc(
      engine,
      dnnl::algorithm::convolution_direct,
      src_md,
      weight_md,
      bia_md,
      dst_md,
      _stride,
      _dilation,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are defined, get_data_type, globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 defined, get_data_type, globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-300
```cpp
      _padding_front_top_left,
      _padding_back_bottom_right,
      conv_fwd_pd,
      pattr);

  // create bwd memory
  at::Tensor expected_src, expected_diff_dst, expected_diff_weight;
  dnnl::memory src_m, diff_dst_m, diff_weight_m;

  src_m = make_onednn_memory(src_md, engine, src.data_ptr());
  diff_dst_m = make_onednn_memory(dst_md, engine, diff_dst.data_ptr());
  diff_weight_m = make_onednn_memory(weight_md, engine, diff_weight.data_ptr());

  // insert args
  std::unordered_map<int, dnnl::memory> args;
  args.insert({DNNL_ARG_DIFF_DST, diff_dst_m});
  args.insert({DNNL_ARG_SRC, src_m});
  args.insert({DNNL_ARG_DIFF_WEIGHTS, diff_weight_m});
  if (diff_bia.defined()) {
    dnnl::memory diff_bia_m =
        make_onednn_memory(bia_md, engine, diff_bia.data_ptr());
    args.insert({DNNL_ARG_DIFF_BIAS, diff_bia_m});
  }

  size_t scratchpad_size = conv_bwd_w_pd.scratchpad_desc().get_size();
  at::Tensor scratchpad_tensor = at::empty(
      {static_cast<int64_t>(scratchpad_size)},
      src.options().dtype(at::kByte),
      std::nullopt);
  auto scratchpad_m = make_onednn_memory(
      conv_bwd_w_pd.scratchpad_desc(), engine, scratchpad_tensor.data_ptr());
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad_m});

  // execute primitive
  auto conv_bwd_w = dnnl::convolution_backward_weights(conv_bwd_w_pd);
  sycl::event conv_bwd_w_event =
      dnnl::sycl_interop::execute(conv_bwd_w, stream, args, deps);

  return conv_bwd_w_event;
}

sycl::event convolution_backward_data(
    at::Tensor& diff_src,
    const at::Tensor& diff_dst,
    const at::Tensor& weight,
    IntArrayRef padding_front_top_left,
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    bool bias_defined,
    const std::vector<sycl::event>& deps) {
  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();

  bool is_channels_last = use_channels_last_for_conv(diff_dst, weight);

  // create memory desc
  auto [src_md, weight_md, dst_md] =
      conv_get_md(diff_src, weight, diff_dst, groups, is_channels_last);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 301-360
```cpp
  dnnl::memory::format_tag bia_fmt = dnnl::memory::format_tag::x;
  auto bia_md = bias_defined
      ? dnnl::memory::desc(
            {diff_dst.size(1)}, weight_md.get_data_type(), bia_fmt)
      : dnnl::memory::desc();

  // create fwd primitive desc hint
  dnnl::primitive_attr pattr;

#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn()) {
    pattr.set_deterministic(true);
  }
#endif

  pattr.set_scratchpad_mode(dnnl::scratchpad_mode::user);
  dnnl::memory::dims _stride = stride.vec();
  dnnl::memory::dims _padding_front_top_left = padding_front_top_left.vec();
  dnnl::memory::dims _padding_back_bottom_right =
      padding_back_bottom_right.vec();
  dnnl::memory::dims _dilation = compatible_dilation(dilation);

  at::native::onednn::apply_tf32_if_allowed(pattr);

  auto conv_forward_pd = dnnl::convolution_forward::primitive_desc(
      engine,
      dnnl::prop_kind::forward,
      dnnl::algorithm::convolution_direct,
      src_md,
      weight_md,
      bia_md,
      dst_md,
      _stride,
      _dilation,
      _padding_front_top_left,
      _padding_back_bottom_right,
      pattr);

  auto conv_backward_data_pd = dnnl::convolution_backward_data::primitive_desc(
      engine,
      dnnl::algorithm::convolution_direct,
      src_md,
      weight_md,
      dst_md,
      _stride,
      _dilation,
      _padding_front_top_left,
      _padding_back_bottom_right,
      conv_forward_pd,
      pattr);

  // create memory
  at::Tensor expected_src, expected_wei, expected_dst;
  dnnl::memory diff_dst_m, wei_m, diff_src_m;

  diff_src_m = make_onednn_memory(src_md, engine, diff_src.data_ptr());
  wei_m = make_onednn_memory(weight_md, engine, weight.data_ptr());
  diff_dst_m = make_onednn_memory(dst_md, engine, diff_dst.data_ptr());
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are get_data_type, globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 get_data_type, globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 361-385
```cpp
  // insert args
  std::unordered_map<int, dnnl::memory> args;
  size_t scratchpad_size = conv_backward_data_pd.scratchpad_desc().get_size();
  at::Tensor scratchpad_tensor = at::empty(
      {static_cast<int64_t>(scratchpad_size)},
      diff_dst.options().dtype(at::kByte),
      std::nullopt);
  auto scratchpad_memory = make_onednn_memory(
      conv_backward_data_pd.scratchpad_desc(),
      engine,
      scratchpad_tensor.data_ptr());
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad_memory});
  args.insert({DNNL_ARG_DIFF_DST, diff_dst_m});
  args.insert({DNNL_ARG_WEIGHTS, wei_m});
  args.insert({DNNL_ARG_DIFF_SRC, diff_src_m});

  // execute primitive
  auto conv_backward_data =
      dnnl::convolution_backward_data(conv_backward_data_pd);
  auto conv_backward_data_event =
      dnnl::sycl_interop::execute(conv_backward_data, stream, args, deps);
  return conv_backward_data_event;
}

} // namespace at::native::onednn
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Notable symbols: conv_dst_size, conv_get_md, convolution, convolution_backward_weights, convolution_backward_data.
- CN: 重要符号：conv_dst_size, conv_get_md, convolution, convolution_backward_weights, convolution_backward_data。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/xpu/XPUFunctions.h, ATen/ATen.h, ATen/core/grad_mode.h, ATen/record_function.h, c10/core/MemoryFormat.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/Utils.h`.
- CN: 主要内部头文件：`c10/xpu/XPUFunctions.h, ATen/ATen.h, ATen/core/grad_mode.h, ATen/record_function.h, c10/core/MemoryFormat.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/Utils.h`。
- EN: External/system headers: `oneapi/dnnl/dnnl.hpp`.
- CN: 外部/系统头文件：`oneapi/dnnl/dnnl.hpp`。
- EN: The implementation revolves around symbols such as `conv_dst_size, conv_get_md, convolution, convolution_backward_weights, convolution_backward_data`.
- CN: 实现围绕 `conv_dst_size, conv_get_md, convolution, convolution_backward_weights, convolution_backward_data` 等符号展开。
