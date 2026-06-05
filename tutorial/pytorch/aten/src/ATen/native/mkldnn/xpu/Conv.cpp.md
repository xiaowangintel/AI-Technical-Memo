# Conv.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/Conv.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on conv; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 conv；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <vector>

#include <ATen/core/ATen_fwd.h>
#include <ATen/core/interned_strings.h>
#include <ATen/native/ConvUtils.h>
#include <ATen/native/mkldnn/xpu/Conv.h>
#include <ATen/native/mkldnn/xpu/FusionUtils.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNN.h>
#include <ATen/native/utils/ParamUtils.h>
#include <ATen/ops/full.h>
#include <ATen/ops/neg.h>
#include <c10/core/Scalar.h>
#include <c10/util/Exception.h>
#include <torch/library.h>
#include <optional>

using namespace dnnl;
using namespace at::native;
using namespace at::native::onednn;

namespace at::native::xpu {
namespace impl {

struct ConvParams {
  std::vector<int64_t> stride;
  std::vector<int64_t> padding;
  std::vector<int64_t> dilation;
  bool transposed{};
  std::vector<int64_t> output_padding;
  int64_t groups{};

  bool is_output_padding_neg() const;
  bool is_padding_neg() const;
  bool is_stride_nonpos() const;
  void view1d_as_2d();
};

std::ostream& operator<<(std::ostream& out, const ConvParams& params) {
  out << "ConvParams {"
      << "  stride = " << IntArrayRef{params.stride}
      << "  padding = " << IntArrayRef{params.padding}
      << "  dilation = " << IntArrayRef{params.dilation}
      << "  transposed = " << params.transposed
      << "  output_padding = " << IntArrayRef{params.output_padding}
      << "  groups = " << params.groups << '}';
  return out;
}

bool ConvParams::is_output_padding_neg() const {
  bool is_non_neg = false;
  for (auto p : output_padding) {
    is_non_neg |= (p < 0);
  }
  return is_non_neg;
}

bool ConvParams::is_padding_neg() const {
  bool is_non_neg = false;
  for (auto p : padding) {
    is_non_neg |= (p < 0);
```
- EN: Lines 1-60 pull in 14 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are is_output_padding_neg, is_padding_neg, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 14 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 is_output_padding_neg, is_padding_neg，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
  }
  return is_non_neg;
}

bool ConvParams::is_stride_nonpos() const {
  bool is_nonpos = false;
  for (auto s : stride) {
    is_nonpos |= (s <= 0);
  }
  return is_nonpos;
}

void ConvParams::view1d_as_2d() {
  if (stride.size() == 1) {
    stride.insert(stride.begin(), 1);
    padding.insert(padding.begin(), 0);
    dilation.insert(dilation.begin(), 1);
    output_padding.insert(output_padding.begin(), 0);
  }
}

static void check_shape_forward(
    const at::Tensor& input,
    const at::Tensor& weight,
    const at::Tensor& bias,
    const ConvParams& params) {
  int64_t k = input.ndimension();
  int64_t weight_dim = weight.ndimension();
  std::vector<int64_t> weight_sizes(weight_dim);
  if (weight_dim == k + 1) {
    weight_sizes[0] = weight.size(0) * weight.size(1);
    std::copy_n(weight.sizes().cbegin() + 2, k - 1, weight_sizes.begin() + 1);
    weight_dim = k;
  } else {
    std::copy_n(weight.sizes().cbegin(), weight_dim, weight_sizes.begin());
  }
  int64_t groups = params.groups;
  auto padding = params.padding;
  auto output_padding = params.output_padding;
  auto stride = params.stride;
  auto dilation = params.dilation;
  bool transposed = params.transposed;

  TORCH_CHECK(!params.is_padding_neg(), "negative padding is not supported");
  TORCH_CHECK(
      !params.is_output_padding_neg(),
      "negative output_padding is not supported");
  TORCH_CHECK(
      !params.is_stride_nonpos(), "non-positive stride is not supported");

  TORCH_CHECK(
      weight_dim == k,
      "Expected ",
      weight_dim,
      "-dimensional input for ",
      weight_dim,
      "-dimensional weight ",
      weight_sizes,
      ", but got ",
      k,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are is_stride_nonpos, view1d_as_2d, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 is_stride_nonpos, view1d_as_2d，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp
      "-dimensional input of size ",
      input.sizes(),
      " instead");
  TORCH_CHECK(
      weight_sizes[0] >= groups,
      "Given groups=",
      groups,
      ", expected weight to be at least ",
      groups,
      " at dimension 0, but got weight of size ",
      weight_sizes,
      " instead");
  TORCH_CHECK(
      weight_sizes[0] % groups == 0,
      "Given groups=",
      groups,
      ", expected weight to be divisible by ",
      groups,
      " at dimension 0, but got weight of size ",
      weight_sizes,
      " instead");

  if (!transposed) {
    std::vector<int64_t> input_shape;
    std::vector<int64_t> kernel_shape;
    bool kernel_size_correct = true;

    TORCH_CHECK(
        input.size(1) == (weight_sizes[1] * groups),
        "Given groups=",
        groups,
        ", weight of size ",
        weight_sizes,
        ", expected input",
        input.sizes(),
        " to have ",
        (weight_sizes[1] * groups),
        " channels, but got ",
        input.size(1),
        " channels instead");
    TORCH_CHECK(
        !bias.defined() ||
            (bias.ndimension() == 1 && bias.size(0) == weight_sizes[0]),
        "Given weight of size ",
        weight_sizes,
        ", expected bias to be 1-dimensional with ",
        weight_sizes[0],
        " elements",
        ", but got bias of size ",
        bias.sizes(),
        " instead");

    for (int i = 2; i < k; ++i) {
      input_shape.push_back(input.size(i) + 2 * padding[i - 2]);
      kernel_shape.push_back(dilation[i - 2] * (weight_sizes[i] - 1) + 1);
      if (input_shape.back() < kernel_shape.back()) {
        kernel_size_correct = false;
      }
    }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-240
```cpp
    TORCH_CHECK(
        input_shape.size() == kernel_shape.size(),
        "Inconsistent shape between Input and Kernel");

    if (!kernel_size_correct) {
      std::ostringstream input_ss;
      std::ostringstream kernel_ss;
      std::ostringstream output_ss;
      std::string separator = "";

      for (size_t i = 0, len = input_shape.size(); i < len; ++i) {
        input_ss << separator << input_shape[i];
        kernel_ss << separator << kernel_shape[i];
        separator = " x ";
      }

      TORCH_CHECK(
          0,
          "Calculated padded input size per channel: (",
          input_ss.str(),
          "). "
          "Kernel size: (",
          kernel_ss.str(),
          "). Kernel size can't be greater than actual input size");
    }
  } else {
    TORCH_CHECK(
        input.size(1) == weight_sizes[0],
        "Given transposed=",
        transposed,
        ", weight of size ",
        weight_sizes,
        ", expected input",
        input.sizes(),
        " to have ",
        weight_sizes[0],
        " channels, but got ",
        input.size(1),
        " channels instead");
    TORCH_CHECK(
        !bias.defined() ||
            (bias.ndimension() == 1 &&
             bias.size(0) == weight_sizes[1] * groups),
        "Given transposed=",
        transposed,
        ", weight of size ",
        weight_sizes,
        ", expected bias to be 1-dimensional with ",
        weight_sizes[1] * groups,
        " elements",
        ", but got bias of size ",
        bias.sizes(),
        " instead");
  }
}

static at::Tensor view4d(const at::Tensor& tensor) {
  TORCH_CHECK(
      tensor.ndimension() == 3,
      "expected 3D tensor, got tensor with ",
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are view4d, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 view4d，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-300
```cpp
      tensor.ndimension(),
      " dimensions instead");
  return tensor.unsqueeze(2);
}

static at::Tensor view3d(const at::Tensor& tensor) {
  TORCH_CHECK(
      tensor.ndimension() == 4,
      "expected 4D tensor, got tensor with ",
      tensor.ndimension(),
      " dimensions instead");
  return tensor.squeeze(2);
}

} // namespace impl

using namespace impl;

Tensor _convolution_out(
    Tensor& output_r,
    const Tensor& input_r,
    const Tensor& weight_r,
    const Tensor& bias_r,
    IntArrayRef stride_,
    IntArrayRef padding_,
    IntArrayRef dilation_,
    bool transposed_,
    IntArrayRef output_padding_,
    int64_t groups_,
    Attr attr,
    IntArrayRef pad_nd = IntArrayRef({})) {
  CheckedFrom c = "xpu_convolution";
  TensorArg input_t{input_r, "input", 1}, weight_t{weight_r, "weight", 2};
  checkAllSameType(c, {input_t, weight_t});
  checkAllSameGPU(c, {input_t, weight_t});
  c10::DeviceGuard device_guard(input_r.device());
  auto ndim = input_r.ndimension();
  TORCH_CHECK(
      3 == ndim || 4 == ndim || 5 == ndim,
      "convolution only supports 3D, 4D, 5D tensor");
  Tensor input = input_r, weight = weight_r;
  // PyTorch does not support ChannelsLast1D case,
  // thus we need the transformation here
  if (ndim == 3) {
    input = view4d(input_r);
    weight = view4d(weight_r);
  }
  // get computation format for Conv/TransposedConv
  bool is_channels_last_suggested = use_channels_last_for_conv(input, weight);

  auto k = weight.ndimension();
  if (k == input.ndimension() + 1) {
    k = input.ndimension();
  }
  int64_t dim = k - 2;
  TORCH_CHECK(dim > 0, "weight should have at least three dimensions");

  ConvParams params;
  if (ndim == 3) {
    // PyTorch does not support ChannelsLast1D case,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are view3d, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 view3d，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-360
```cpp
    // thus we need the transformation here
    params.stride = stride_.vec();
    params.padding = padding_.vec();
    params.dilation = dilation_.vec();
    params.transposed = transposed_;
    params.output_padding = output_padding_.vec();
    params.groups = groups_;
    params.view1d_as_2d();
  } else {
    params.stride = expand_param_if_needed(stride_, "stride", dim);
    // PyTorch default Conv padding should be a single integer value
    // or a list of values to match the conv dimensions
    // conv2d, the number of padding values should be 1 or 2
    // conv3d, the number of padding values should be 1 or 3
    // the padding value will be padded into both side of Conv input (D, H, W)
    params.padding = expand_param_if_needed(padding_, "padding", dim);
    params.dilation = expand_param_if_needed(dilation_, "dilation", dim);
    params.transposed = transposed_;
    params.output_padding =
        expand_param_if_needed(output_padding_, "output_padding", dim);
    params.groups = groups_;
  }

  // ensure the input/weight/bias/output are congituous in desired format
  at::MemoryFormat mfmt = is_channels_last_suggested
      ? get_cl_tag_by_ndim(input.ndimension())
      : at::MemoryFormat::Contiguous;

  auto bias = bias_r.defined() ? make_contiguous_and_aligned(bias_r) : bias_r;
  input = make_contiguous_and_aligned(input, mfmt);
  weight = make_contiguous_and_aligned(weight, mfmt);
  check_shape_forward(input, weight, bias, params);

  Tensor output;
  if (transposed_) {
    // create output and propagate memory format
    if (!output_r.defined()) {
      auto dst_tz = deconv_dst_size(
          input.sizes(),
          weight.sizes(),
          params.padding,
          params.stride,
          params.dilation,
          params.output_padding,
          params.groups);
      output = at::empty(dst_tz, input.options(), mfmt);
    } else {
      output = output_r;
    }

    onednn::deconvolution(
        output,
        input,
        weight,
        bias,
        params.stride,
        params.padding,
        params.output_padding,
        params.dilation,
        params.groups,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are input, get_cl_tag_by_ndim, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 input, get_cl_tag_by_ndim，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 361-420
```cpp
        attr);
  } else {
    // oneDNN supports padding the two sides of src with different values
    // the padding order should be front_top_left and back_bottom_right
    auto padding_front_top_left = params.padding;
    auto padding_back_bottom_right = params.padding;

    // PyTorch constant_pad_nd:
    // can pad different value to the two sides of Conv input (W, H, D)
    // (padding_left, padding_right,
    //  padding_top, padding_bottom,
    //  padding_front, padding_back)
    if (!pad_nd.vec().empty()) {
      for (int64_t i = 0; i < dim; ++i) {
        padding_front_top_left[i] += pad_nd[2 * dim - 2 * i - 2]; // 4, 2, 0
        padding_back_bottom_right[i] += pad_nd[2 * dim - 2 * i - 1]; // 5, 3, 1
      }
    }

    // create output and propagate memory format
    if (!output_r.defined()) {
      auto dst_tz = conv_dst_size(
          input.ndimension(),
          input.sizes(),
          weight.sizes(),
          padding_front_top_left,
          padding_back_bottom_right,
          params.stride,
          params.dilation);
      output = at::empty(dst_tz, input.options(), mfmt);
    } else {
      output = output_r;
    }
    onednn::convolution(
        output,
        input,
        weight,
        bias,
        padding_front_top_left,
        padding_back_bottom_right,
        params.stride,
        params.dilation,
        params.groups,
        attr);
  }

  if (ndim == 3) {
    output = view3d(output);
  }
  if (output_r.defined() && !output_r.is_same(output)) {
    output_r.copy_(output);
  } else {
    output_r = output;
  }
  return output_r;
}

Tensor _convolution(
    const Tensor& input_r,
    const Tensor& weight_r,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are input, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 input，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 421-480
```cpp
    const Tensor& bias_r,
    IntArrayRef stride_,
    IntArrayRef padding_,
    IntArrayRef dilation_,
    bool transposed_,
    IntArrayRef output_padding_,
    int64_t groups_,
    Attr attr) {
  Tensor output_r;
  return _convolution_out(
      output_r,
      input_r,
      weight_r,
      bias_r,
      stride_,
      padding_,
      dilation_,
      transposed_,
      output_padding_,
      groups_,
      attr);
}

Tensor convolution_overrideable(
    const Tensor& input_r,
    const Tensor& weight_r,
    const std::optional<at::Tensor>& bias_r_opt,
    IntArrayRef stride_,
    IntArrayRef padding_,
    IntArrayRef dilation_,
    bool transposed_,
    IntArrayRef output_padding_,
    int64_t groups_) {
  c10::MaybeOwned<Tensor> bias_r_maybe_owned =
      at::borrow_from_optional_tensor(bias_r_opt);
  const Tensor& bias_r = *bias_r_maybe_owned;

  return _convolution(
      input_r,
      weight_r,
      bias_r,
      stride_,
      padding_,
      dilation_,
      transposed_,
      output_padding_,
      groups_,
      Attr());
}

std::tuple<Tensor, Tensor, Tensor> convolution_backward_overrideable(
    const Tensor& grad_output,
    const Tensor& input,
    const Tensor& weight,
    IntArrayRef stride,
    IntArrayRef padding,
    IntArrayRef dilation,
    bool transposed,
    IntArrayRef output_padding,
    int64_t groups,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

### Lines 481-540
```cpp
    std::array<bool, 3> output_mask) {
  CheckedFrom c = "xpu_convolution_backward";
  c10::DeviceGuard device_guard(grad_output.device());
  auto ndim = input.ndimension();
  TORCH_CHECK(
      3 == ndim || 4 == ndim || 5 == ndim,
      "convolution bwd only supports 3D, 4D, 5D tensor");
  TORCH_CHECK(
      grad_output.scalar_type() == ScalarType::Float ||
          grad_output.scalar_type() == ScalarType::BFloat16 ||
          grad_output.scalar_type() == ScalarType::Double ||
          grad_output.scalar_type() == ScalarType::Half,
      "so far only support float, bfloat16, half and double convolution backward in XPU backend, your data type is ",
      grad_output.scalar_type());

  bool is_channels_last_suggested = use_channels_last_for_conv(input, weight);

  Tensor grad_output_, input_, weight_;
  IntArrayRef stride_, padding_, dilation_, output_padding_;
  bool transposed_ = false;
  int64_t groups_ = 0;
  ConvParams params;
  if (3 == ndim) {
    grad_output_ = view4d(grad_output);
    input_ = view4d(input);
    weight_ = view4d(weight);
    params.stride = stride.vec();
    params.padding = padding.vec();
    params.dilation = dilation.vec();
    params.transposed = transposed;
    params.output_padding = output_padding.vec();
    params.groups = groups;
    params.view1d_as_2d();
    stride_ = params.stride;
    padding_ = params.padding;
    dilation_ = params.dilation;
    transposed_ = params.transposed;
    output_padding_ = params.output_padding;
    groups_ = params.groups;
  } else {
    grad_output_ = grad_output;
    input_ = input;
    weight_ = weight;
    stride_ = stride;
    padding_ = padding;
    dilation_ = dilation;
    transposed_ = transposed;
    output_padding_ = output_padding;
    groups_ = groups;
  }

  // ensure the tensors are contiguous
  auto mfmt = is_channels_last_suggested
      ? get_cl_tag_by_ndim(input_.ndimension())
      : at::MemoryFormat::Contiguous;
  grad_output_ = make_contiguous_and_aligned(grad_output_, mfmt);
  weight_ = make_contiguous_and_aligned(weight_, mfmt);
  input_ = make_contiguous_and_aligned(input_, mfmt);

  auto opt = grad_output_.options();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are get_cl_tag_by_ndim, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 get_cl_tag_by_ndim，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 541-600
```cpp
  Tensor grad_input;
  Tensor grad_weight;
  Tensor grad_bias;
  if (output_mask[2]) {
    grad_bias = at::empty({grad_output_.size(1)}, opt);
  }

  if (output_mask[0]) {
    grad_input = at::empty(input_.sizes(), opt, mfmt);
    TensorArg grad_output_t{grad_output, "grad_output", 1},
        input_t{input, "input", 2};
    checkAllSameType(c, {grad_output_t, input_t});
    checkAllSameGPU(c, {grad_output_t, input_t});
    if (input.numel() > 0) {
      if (transposed_) {
        onednn::deconvolution_backward_data(
            grad_input,
            grad_output_,
            weight_,
            stride_,
            padding_,
            output_padding_,
            dilation_,
            groups_,
            output_mask[2]);
      } else {
        onednn::convolution_backward_data(
            grad_input,
            grad_output_,
            weight_,
            padding_,
            padding_,
            stride_,
            dilation_,
            groups_,
            output_mask[2]);
      }
    }
  }
  if (output_mask[1] || output_mask[2]) {
    grad_weight = at::empty(weight_.sizes(), opt, mfmt);
    TensorArg grad_output_t{grad_output, "grad_output", 1},
        weight_t{weight, "weight", 2};
    checkAllSameType(c, {grad_output_t, weight_t});
    checkAllSameGPU(c, {grad_output_t, weight_t});
    if (input.numel() > 0) {
      if (transposed_) {
        onednn::deconvolution_backward_weights(
            grad_weight,
            grad_bias,
            grad_output_,
            input_,
            stride_,
            padding_,
            output_padding_,
            dilation_,
            groups_);
      } else {
        onednn::convolution_backward_weights(
            grad_weight,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 601-660
```cpp
            grad_bias,
            grad_output_,
            input_,
            weight_.sizes(),
            padding_,
            padding_,
            stride_,
            dilation_,
            groups_);
      }
    }
    if (!output_mask[1]) {
      grad_weight = at::Tensor();
    }
  }

  if (3 == ndim) {
    if (output_mask[0])
      grad_input = view3d(grad_input);
    if (output_mask[1])
      grad_weight = view3d(grad_weight);
  }
  return std::tuple<Tensor, Tensor, Tensor>{grad_input, grad_weight, grad_bias};
}

Tensor convolution_pointwise(
    const Tensor& input_t,
    const Tensor& weight_t,
    const std::optional<Tensor>& bias_opt,
    IntArrayRef padding,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    std::string_view attr,
    torch::List<std::optional<at::Scalar>> scalars,
    std::optional<std::string_view> algorithm) {
  c10::DeviceGuard device_guard(input_t.device());
  Attr att;
  att = construct_unary_attr(att, attr, scalars, algorithm);
  const Tensor bias = bias_opt.has_value() ? bias_opt.value() : at::Tensor();

  return _convolution(
      input_t,
      weight_t,
      bias,
      stride,
      padding,
      dilation,
      /*transposed*/ false,
      /*output_padding*/ {0},
      groups,
      att);
}

Tensor convolution_pointwise_binary(
    const Tensor& input_t,
    const Tensor& other_t,
    const Tensor& weight_t,
    const std::optional<Tensor>& bias_opt,
    IntArrayRef padding,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 661-720
```cpp
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    std::string_view binary_attr,
    std::optional<at::Scalar> alpha,
    std::optional<std::string_view> unary_attr,
    torch::List<std::optional<at::Scalar>> unary_scalars,
    std::optional<std::string_view> unary_algorithm) {
  c10::DeviceGuard device_guard(input_t.device());
  Tensor output;
  Tensor bias = bias_opt.has_value() ? bias_opt.value() : at::Tensor();
  // Step1: Construct binary attr
  Attr attr;
  attr = construct_binary_attr(attr, binary_attr, other_t);
  // Step2: Append unary attr
  if (unary_attr.has_value())
    attr = construct_unary_attr(
        attr, unary_attr.value(), unary_scalars, unary_algorithm);

  Tensor res = _convolution_out(
      output,
      input_t,
      weight_t,
      bias,
      stride,
      padding,
      dilation,
      /*transposed*/ false,
      /*output_padding*/ {0},
      groups,
      attr);

  // Step3: Run conv
  return res;
}

Tensor& convolution_pointwise_binary_(
    Tensor& other_t,
    const Tensor& input_t,
    const Tensor& weight_t,
    const std::optional<Tensor>& bias_opt,
    IntArrayRef padding,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    std::string_view binary_attr,
    std::optional<at::Scalar> alpha,
    std::optional<std::string_view> unary_attr,
    torch::List<std::optional<at::Scalar>> unary_scalars,
    std::optional<std::string_view> unary_algorithm) {
  c10::DeviceGuard device_guard(input_t.device());
  Tensor bias = bias_opt.has_value() ? bias_opt.value() : at::Tensor();
  // Step1: Construct binary attr
  Attr attr;
  attr = construct_binary_attr(attr, binary_attr, other_t);

  // Step2: Append unary attr
  if (unary_attr.has_value())
    attr = construct_unary_attr(
        attr, unary_attr.value(), unary_scalars, unary_algorithm);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 721-758
```cpp

  _convolution_out(
      other_t,
      input_t,
      weight_t,
      bias,
      stride,
      padding,
      dilation,
      /*transposed*/ false,
      /*output_padding*/ {0},
      groups,
      attr);

  // Step3: Run conv
  return other_t;
}

TORCH_LIBRARY_IMPL(aten, XPU, m) {
  m.impl("convolution_overrideable", TORCH_FN(convolution_overrideable));
  m.impl(
      "convolution_backward_overrideable",
      TORCH_FN(convolution_backward_overrideable));
}

TORCH_LIBRARY_IMPL(mkldnn, XPU, m) {
  m.impl(
      TORCH_SELECTIVE_NAME("mkldnn::_convolution_pointwise"),
      TORCH_FN(convolution_pointwise));
  m.impl(
      TORCH_SELECTIVE_NAME("mkldnn::_convolution_pointwise.binary"),
      TORCH_FN(convolution_pointwise_binary));
  m.impl(
      TORCH_SELECTIVE_NAME("mkldnn::_convolution_pointwise_.binary"),
      TORCH_FN(convolution_pointwise_binary_));
}

} // namespace at::native::xpu
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_LIBRARY_IMPL, concentrating a specific part of the operator behavior.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_LIBRARY_IMPL，它们承载了某一部分算子行为的核心逻辑。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: is_output_padding_neg, is_padding_neg, is_stride_nonpos, view1d_as_2d, check_shape_forward, view4d, view3d, _convolution.
- CN: 重要符号：is_output_padding_neg, is_padding_neg, is_stride_nonpos, view1d_as_2d, check_shape_forward, view4d, view3d, _convolution。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/ATen_fwd.h, ATen/core/interned_strings.h, ATen/native/ConvUtils.h, ATen/native/mkldnn/xpu/Conv.h, ATen/native/mkldnn/xpu/FusionUtils.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/utils/ParamUtils.h, ATen/ops/full.h, ATen/ops/neg.h, c10/core/Scalar.h, c10/util/Exception.h, torch/library.h`.
- CN: 主要内部头文件：`ATen/core/ATen_fwd.h, ATen/core/interned_strings.h, ATen/native/ConvUtils.h, ATen/native/mkldnn/xpu/Conv.h, ATen/native/mkldnn/xpu/FusionUtils.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/utils/ParamUtils.h, ATen/ops/full.h, ATen/ops/neg.h, c10/core/Scalar.h, c10/util/Exception.h, torch/library.h`。
- EN: External/system headers: `vector, optional`.
- CN: 外部/系统头文件：`vector, optional`。
- EN: The implementation revolves around symbols such as `is_output_padding_neg, is_padding_neg, is_stride_nonpos, view1d_as_2d, check_shape_forward, view4d, view3d, _convolution, convolution_overrideable, convolution_backward_overrideable`.
- CN: 实现围绕 `is_output_padding_neg, is_padding_neg, is_stride_nonpos, view1d_as_2d, check_shape_forward, view4d, view3d, _convolution, convolution_overrideable, convolution_backward_overrideable` 等符号展开。
