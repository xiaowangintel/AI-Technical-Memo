# Deconv.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/detail/Deconv.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on deconv; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 deconv；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <ATen/ATen.h>
#include <c10/xpu/XPUFunctions.h>

#include <ATen/native/mkldnn/xpu/detail/Attr.h>
#include <ATen/native/mkldnn/xpu/detail/Utils.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNNContext.h>
#include <oneapi/dnnl/dnnl.hpp>

namespace at::native::onednn {

static inline dnnl::memory::dims deconv_compatible_dilation(
    IntArrayRef& dilation) {
  dnnl::memory::dims ret = dilation.vec();
  for (auto it = ret.begin(); it != ret.end(); it++) {
    *it -= 1;
  }
  return ret;
}

static inline std::vector<int64_t> compatible_groups_deconv_strides(
    const at::Tensor& weight,
    dnnl::memory::dims group_size) {
  std::vector<int64_t> strides = weight.strides().vec();
  strides[0] = weight.strides()[1];
  strides[1] = weight.strides()[0];
  strides.insert(strides.begin(), group_size[2] * weight.strides()[0]);
  return strides;
}

dnnl::memory::dims deconv_dst_size(
    IntArrayRef src_size,
    IntArrayRef weight_size,
    IntArrayRef padding,
    IntArrayRef stride,
    IntArrayRef dilation,
    IntArrayRef dst_padding,
    int64_t groups) {
  auto dim = src_size.size();
  dnnl::memory::dims dst_size(dim);
  auto kernel_size = weight_size.slice(2);

  dst_size[0] = src_size[0];
  dst_size[1] = weight_size[1] * groups;
  for (size_t d = 2; d < dim; ++d) {
    dst_size[d] = (src_size[d] - 1) * stride[d - 2] - 2 * padding[d - 2] +
        (dilation[d - 2] * (kernel_size[d - 2] - 1) + 1) + dst_padding[d - 2];
  }
  return dst_size;
}

static inline dnnl::memory::format_tag deconv_src_fmt(
    const int64_t ndim,
    const bool is_channels_last = false) {
  // 3D: n/c/w (n/w/c)         [a/b/c (a/c/b)]
  // 4D: n/c/h/w (n/h/w/c)     [a/b/c/d (a/c/d/b)]
  // 5D: n/c/d/h/w (n/d/h/w/c) [a/b/c/d/e (a/c/d/e/b)]
  if (!is_channels_last) {
    return (ndim == 3)
        ? dnnl::memory::format_tag::ncw
        : ((ndim == 4) ? dnnl::memory::format_tag::nchw
```
- EN: Lines 1-60 pull in 6 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 第 1-60 行引入了 6 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 61-120
```cpp
                       : ((ndim == 5) ? dnnl::memory::format_tag::ncdhw
                                      : dnnl::memory::format_tag::undef));
  } else {
    return (ndim == 3)
        ? dnnl::memory::format_tag::nwc
        : ((ndim == 4) ? dnnl::memory::format_tag::nhwc
                       : ((ndim == 5) ? dnnl::memory::format_tag::ndhwc
                                      : dnnl::memory::format_tag::undef));
  }
}

static inline std::vector<int64_t> deconv_weight_fmt(
    const at::Tensor& weight,
    const int64_t ndim,
    dnnl::memory::dims weight_size,
    const bool grouped = false,
    const bool is_channels_last = false) {
  // 3D fmt: (g)i/o/w ((g)i/w/o)  [b/a/c  (b/c/a)]
  // 4D fmt: (g)i/o/h/w ((g)i/h/w/o) [b/a/c/d (b/c/d/a)]
  // 5D fmt: (g)i/o/d/h/w ((g)i/d/h/w/o) [b/a/c/d/e (b/c/d/e/a)]
  auto strides_ = weight.strides().vec();
  std::vector<int64_t> strides;
  if (grouped) {
    strides = compatible_groups_deconv_strides(weight, weight_size);
  } else {
    strides = strides_;
    std::swap(strides[0], strides[1]);
  }
  return strides;
}

static inline dnnl::memory::dims deconv_compatible_weight_dims(
    int64_t ndim,
    int64_t groups,
    int64_t oc,
    int64_t ic,
    IntArrayRef weight_size) {
  if (ndim == 3) {
    auto kw = weight_size[2];
    return (groups != 1)
        ? dnnl::memory::dims({groups, oc / groups, ic / groups, kw})
        : dnnl::memory::dims({oc, ic, kw});
  } else if (ndim == 4) {
    auto kh = weight_size[2];
    auto kw = weight_size[3];
    return (groups != 1)
        ? dnnl::memory::dims({groups, oc / groups, ic / groups, kh, kw})
        : dnnl::memory::dims({oc, ic, kh, kw});
  } else if (ndim == 5) {
    auto kd = weight_size[2];
    auto kh = weight_size[3];
    auto kw = weight_size[4];
    return (groups != 1)
        ? dnnl::memory::dims({groups, oc / groups, ic / groups, kd, kh, kw})
        : dnnl::memory::dims({oc, ic, kd, kh, kw});
  } else {
    TORCH_CHECK(0, "unsupported dimension in xpu oneDNN deconvolution...");
  }
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-180
```cpp
static std::tuple<dnnl::memory::desc, dnnl::memory::desc, dnnl::memory::desc>
deconv_get_plain_md(
    const at::Tensor& src,
    const at::Tensor& weight,
    const at::Tensor& dst,
    int64_t groups,
    bool is_channels_last_suggested) {
  auto ndim = src.ndimension();
  auto src_data_t = get_onednn_dtype_include_double(src);
  auto fmt_src = deconv_src_fmt(ndim, is_channels_last_suggested);
  auto src_usr_md = dnnl::memory::desc(src.sizes().vec(), src_data_t, fmt_src);

  auto dst_data_t = get_onednn_dtype_include_double(dst);
  auto dst_usr_md = dnnl::memory::desc(dst.sizes().vec(), dst_data_t, fmt_src);

  auto ic = src.size(1);
  auto oc = dst.size(1);
  dnnl::memory::dims weight_size =
      deconv_compatible_weight_dims(ndim, groups, oc, ic, weight.sizes());
  auto weight_dt = get_onednn_dtype_include_double(weight);
  auto fmt_weight = deconv_weight_fmt(
      weight, ndim, weight_size, groups != 1, is_channels_last_suggested);
  dnnl::memory::desc weight_usr_md =
      dnnl::memory::desc(weight_size, weight_dt, fmt_weight);

  return {src_usr_md, weight_usr_md, dst_usr_md};
}

sycl::event deconvolution(
    at::Tensor& dst,
    const at::Tensor& src,
    const at::Tensor& weight,
    const at::Tensor& bia,
    IntArrayRef stride,
    IntArrayRef padding,
    IntArrayRef dst_padding,
    IntArrayRef dilation,
    int64_t groups,
    Attr& attr,
    const std::vector<sycl::event>& deps) {
  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();

  bool is_channels_last_suggested = use_channels_last_for_conv(src, weight);

  // create usr_md for tensors, and md for conv primitive
  auto [src_md, weight_md, dst_md] =
      deconv_get_plain_md(src, weight, dst, groups, is_channels_last_suggested);

  dnnl::memory::format_tag bia_fmt = dnnl::memory::format_tag::x;
  auto bia_md = bia.defined()
      ? dnnl::memory::desc(
            {dst.size(1)}, get_onednn_dtype_include_double(bia), bia_fmt)
      : dnnl::memory::desc();

  // create primitive desc
  dnnl::memory::dims _stride = stride.vec();
  dnnl::memory::dims _padding_l = padding.vec();
  dnnl::memory::dims _padding_r = padding_r(padding, dst_padding);
  dnnl::memory::dims _dilation = deconv_compatible_dilation(dilation);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are defined, get_onednn_dtype_include_double, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 defined, get_onednn_dtype_include_double，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-240
```cpp

  // construct primitive attr
  dnnl::primitive_attr pattr;
  dnnl::post_ops po = attr.extract_post_ops(dst);
  pattr.set_post_ops(po);
#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn())
    pattr.set_deterministic(true);
#endif

  pattr.set_scratchpad_mode(dnnl::scratchpad_mode::user);

  auto deconv_fwd_pd = dnnl::deconvolution_forward::primitive_desc(
      engine,
      dnnl::prop_kind::forward,
      dnnl::algorithm::deconvolution_direct,
      src_md,
      weight_md,
      bia_md,
      dst_md,
      _stride,
      _dilation,
      _padding_l,
      _padding_r,
      pattr);

  dnnl::memory src_m, weight_m, dst_m, bia_m;
  at::Tensor src_blocked, weight_blocked, dst_blocked = dst;

  src_m = make_onednn_memory(src_md, engine, src.data_ptr());
  weight_m = make_onednn_memory(weight_md, engine, weight.data_ptr());
  dst_m = make_onednn_memory(dst_md, engine, dst.data_ptr());

  std::unordered_map<int, dnnl::memory> args;
  args.insert({DNNL_ARG_SRC, src_m});
  args.insert({DNNL_ARG_WEIGHTS, weight_m});
  args.insert({DNNL_ARG_DST, dst_m});

  if (bia.defined()) {
    auto bia_m = make_onednn_memory(bia_md, engine, bia.data_ptr());
    args.insert({DNNL_ARG_BIAS, bia_m});
  }
  if (attr.with_binary())
    attr.construct_post_binary(deconv_fwd_pd, args);

  size_t scratchpad_size = deconv_fwd_pd.scratchpad_desc().get_size();
  at::Tensor scratchpad_tensor = at::empty(
      {static_cast<int64_t>(scratchpad_size)},
      src.options().dtype(at::kByte),
      std::nullopt);
  auto scratchpad_m = make_onednn_memory(
      deconv_fwd_pd.scratchpad_desc(), engine, scratchpad_tensor.data_ptr());
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad_m});

  auto deconv_fwd = dnnl::deconvolution_forward(deconv_fwd_pd);
  sycl::event deconv_event =
      dnnl::sycl_interop::execute(deconv_fwd, stream, args, deps);
  return deconv_event;
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-300
```cpp

sycl::event deconvolution_backward_data(
    at::Tensor& diff_src,
    const at::Tensor& diff_dst,
    const at::Tensor& weight,
    IntArrayRef stride,
    IntArrayRef padding,
    IntArrayRef dst_padding,
    IntArrayRef dilation,
    int64_t groups,
    bool bias_defined,
    const std::vector<sycl::event>& deps) {
  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();

  bool is_channels_last_suggested =
      use_channels_last_for_conv(diff_dst, weight);
  // create memory desc
  auto [src_md, weight_md, dst_md] = deconv_get_plain_md(
      diff_src, weight, diff_dst, groups, is_channels_last_suggested);

  dnnl::memory::format_tag bia_fmt = dnnl::memory::format_tag::x;
  auto bias_md = bias_defined
      ? dnnl::memory::desc(
            {diff_dst.size(1)}, weight_md.get_data_type(), bia_fmt)
      : dnnl::memory::desc();

  // create fwd primitive desc hint
  dnnl::primitive_attr pattr;
  pattr.set_scratchpad_mode(dnnl::scratchpad_mode::user);
#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn())
    pattr.set_deterministic(true);
#endif

  dnnl::memory::dims _stride = stride.vec();
  dnnl::memory::dims _padding_l = padding.vec();
  dnnl::memory::dims _padding_r = padding_r(padding, dst_padding);
  dnnl::memory::dims _dilation = deconv_compatible_dilation(dilation);
  auto deconv_fwd_pd = dnnl::deconvolution_forward::primitive_desc(
      engine,
      dnnl::prop_kind::forward,
      dnnl::algorithm::deconvolution_direct,
      src_md,
      weight_md,
      bias_md,
      dst_md,
      _stride,
      _dilation,
      _padding_l,
      _padding_r,
      pattr);

  // create bwd primitive desc
  auto deconv_backward_data_pd =
      dnnl::deconvolution_backward_data::primitive_desc(
          engine,
          dnnl::algorithm::deconvolution_direct,
          src_md,
```
- EN: The main callable definitions or declarations in this block are get_data_type, globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段的主要可调用定义或声明包括 get_data_type, globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 301-360
```cpp
          weight_md,
          dst_md,
          _stride,
          _dilation,
          _padding_l,
          _padding_r,
          deconv_fwd_pd,
          pattr);

  // create memory
  dnnl::memory diff_dst_m, wei_m, diff_src_m;

  diff_src_m = make_onednn_memory(src_md, engine, diff_src.data_ptr());
  wei_m = make_onednn_memory(weight_md, engine, weight.data_ptr());
  diff_dst_m = make_onednn_memory(dst_md, engine, diff_dst.data_ptr());

  // insert args
  std::unordered_map<int, dnnl::memory> args;
  size_t scratchpad_size = deconv_backward_data_pd.scratchpad_desc().get_size();
  at::Tensor scratchpad_tensor = at::empty(
      {static_cast<int64_t>(scratchpad_size)},
      diff_dst.options().dtype(at::kByte),
      std::nullopt);
  auto scratchpad_memory = make_onednn_memory(
      deconv_backward_data_pd.scratchpad_desc(),
      engine,
      scratchpad_tensor.data_ptr());
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad_memory});
  args.insert({DNNL_ARG_DIFF_DST, diff_dst_m});
  args.insert({DNNL_ARG_WEIGHTS, wei_m});
  args.insert({DNNL_ARG_DIFF_SRC, diff_src_m});

  // execute primitive
  auto deconv_backward_data =
      dnnl::deconvolution_backward_data(deconv_backward_data_pd);
  sycl::event deconv_bwd_data_event =
      dnnl::sycl_interop::execute(deconv_backward_data, stream, args, deps);
  return deconv_bwd_data_event;
}

sycl::event deconvolution_backward_weights(
    at::Tensor& diff_weight,
    at::Tensor& diff_bia,
    const at::Tensor& diff_dst,
    const at::Tensor& src,
    IntArrayRef stride,
    IntArrayRef padding,
    IntArrayRef dst_padding,
    IntArrayRef dilation,
    int64_t groups,
    const std::vector<sycl::event>& deps) {
  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();

  bool is_channels_last_suggested = use_channels_last_for_conv(src, diff_dst);

  // create memory desc
  auto [src_md, weight_md, dst_md] = deconv_get_plain_md(
      src, diff_weight, diff_dst, groups, is_channels_last_suggested);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 361-420
```cpp
  dnnl::memory::format_tag bia_fmt = dnnl::memory::format_tag::x;
  auto bia_md = diff_bia.defined()
      ? dnnl::memory::desc({diff_dst.size(1)}, src_md.get_data_type(), bia_fmt)
      : dnnl::memory::desc();

  // create fwd primitive desc hint
  dnnl::memory::dims _stride = stride.vec();
  dnnl::memory::dims _padding_l = padding.vec();
  dnnl::memory::dims _padding_r = padding_r(padding, dst_padding);
  dnnl::memory::dims _dilation = deconv_compatible_dilation(dilation);
  dnnl::primitive_attr pattr;

#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn())
    pattr.set_deterministic(true);
#endif
  pattr.set_scratchpad_mode(dnnl::scratchpad_mode::user);
  auto deconv_fwd_pd = dnnl::deconvolution_forward::primitive_desc(
      engine,
      dnnl::prop_kind::forward,
      dnnl::algorithm::deconvolution_direct,
      src_md,
      weight_md,
      bia_md,
      dst_md,
      _stride,
      _dilation,
      _padding_l,
      _padding_r,
      pattr);

  auto deconv_bwd_w_pd = dnnl::deconvolution_backward_weights::primitive_desc(
      engine,
      dnnl::algorithm::deconvolution_direct,
      src_md,
      weight_md,
      bia_md,
      dst_md,
      _stride,
      _dilation,
      _padding_l,
      _padding_r,
      deconv_fwd_pd,
      pattr);

  // create bwd dnnl::memory
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
```
- EN: The main callable definitions or declarations in this block are defined, get_data_type, globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段的主要可调用定义或声明包括 defined, get_data_type, globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 421-443
```cpp
    dnnl::memory diff_bia_m =
        make_onednn_memory(bia_md, engine, diff_bia.data_ptr());
    args.insert({DNNL_ARG_DIFF_BIAS, diff_bia_m});
  }

  size_t scratchpad_size = deconv_bwd_w_pd.scratchpad_desc().get_size();
  at::Tensor scratchpad_tensor = at::empty(
      {static_cast<int64_t>(scratchpad_size)},
      src.options().dtype(at::kByte),
      std::nullopt);
  auto scratchpad_m = make_onednn_memory(
      deconv_bwd_w_pd.scratchpad_desc(), engine, scratchpad_tensor.data_ptr());
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad_m});

  // execute primitive
  auto deconv_bwd_w = dnnl::deconvolution_backward_weights(deconv_bwd_w_pd);

  sycl::event deconv_bwd_w_event =
      dnnl::sycl_interop::execute(deconv_bwd_w, stream, args, deps);
  return deconv_bwd_w_event;
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
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: deconv_compatible_dilation, compatible_groups_deconv_strides, deconv_dst_size, deconv_src_fmt, deconv_weight_fmt, deconv_compatible_weight_dims, deconv_get_plain_md, deconvolution.
- CN: 重要符号：deconv_compatible_dilation, compatible_groups_deconv_strides, deconv_dst_size, deconv_src_fmt, deconv_weight_fmt, deconv_compatible_weight_dims, deconv_get_plain_md, deconvolution。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/ATen.h, c10/xpu/XPUFunctions.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/Utils.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h`.
- CN: 主要内部头文件：`ATen/ATen.h, c10/xpu/XPUFunctions.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/Utils.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h`。
- EN: External/system headers: `oneapi/dnnl/dnnl.hpp`.
- CN: 外部/系统头文件：`oneapi/dnnl/dnnl.hpp`。
- EN: The implementation revolves around symbols such as `deconv_compatible_dilation, compatible_groups_deconv_strides, deconv_dst_size, deconv_src_fmt, deconv_weight_fmt, deconv_compatible_weight_dims, deconv_get_plain_md, deconvolution, deconvolution_backward_data, deconvolution_backward_weights`.
- CN: 实现围绕 `deconv_compatible_dilation, compatible_groups_deconv_strides, deconv_dst_size, deconv_src_fmt, deconv_weight_fmt, deconv_compatible_weight_dims, deconv_get_plain_md, deconvolution, deconvolution_backward_data, deconvolution_backward_weights` 等符号展开。
