# PadNd.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/PadNd.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on pad nd; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 pad nd；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/PadNd.h>
#include <ATen/core/Tensor.h>

#include <c10/util/irange.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_empty_affine_quantized.h>
#include <ATen/ops/_pad_circular.h>
#include <ATen/ops/_pad_circular_native.h>
#include <ATen/ops/_pad_enum_native.h>
#include <ATen/ops/constant_pad_nd.h>
#include <ATen/ops/constant_pad_nd_native.h>
#include <ATen/ops/empty.h>
#include <ATen/ops/pad_native.h>
#include <ATen/ops/reflection_pad1d.h>
#include <ATen/ops/reflection_pad2d.h>
#include <ATen/ops/reflection_pad3d.h>
#include <ATen/ops/replication_pad1d.h>
#include <ATen/ops/replication_pad2d.h>
#include <ATen/ops/replication_pad3d.h>
#endif

namespace at::native {

Tensor constant_pad_nd(const Tensor& self, IntArrayRef pad, const Scalar& value) {
    TORCH_CHECK(pad.size() % 2 == 0, "Length of pad must be even but instead it equals ",
```
- EN: Lines 1-30 pull in 19 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are constant_pad_nd, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 19 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 constant_pad_nd，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
             pad.size());

    auto input_sizes = self.sizes();
    auto l_inp = self.dim();

    auto l_pad = pad.size() / 2;
    auto l_diff = l_inp - l_pad;
    TORCH_CHECK(l_inp >= (int64_t)l_pad, "Length of pad should be no more than twice the number of "
             "dimensions of the input. Pad length is ", pad.size(), "while the input has ",
             l_inp, "dimensions.");


    bool all_pads_non_positive = true;

    auto c_input = self;
    for (const auto i : c10::irange(l_diff, l_inp)) {
        auto pad_idx = 2 * (l_inp - i - 1);
        if (pad[pad_idx] < 0) {
            c_input = c_input.narrow(i, -pad[pad_idx], c_input.size(i) + pad[pad_idx]);
        } else if (pad[pad_idx] != 0) {
            all_pads_non_positive = false;
        }
        if (pad[pad_idx + 1] < 0) {
            c_input = c_input.narrow(i, 0, c_input.size(i) + pad[pad_idx + 1]);
        } else if (pad[pad_idx + 1] != 0) {
            all_pads_non_positive = false;
        }
    }

    // if none of the pads are positive we can optimize and just return the result
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 61-90
```cpp
    // of calling .narrow() on the input
    if (all_pads_non_positive) {
        return c_input.clone();
    }


    std::vector<int64_t> new_shape;
    new_shape.reserve(l_diff);
    for (size_t i = 0; i < l_diff; i ++) {
        new_shape.emplace_back(input_sizes[i]);
    }

    for (const auto i : c10::irange(l_pad)) {
        auto pad_idx = pad.size() - ((i + 1) * 2);
        auto new_dim = input_sizes[l_diff + i] + pad[pad_idx] + pad[pad_idx + 1];
        TORCH_CHECK(new_dim >= 0, "The input size ", input_sizes[l_diff + i], ", plus negative padding ",
                 pad[pad_idx], " and ", pad[pad_idx + 1], " resulted in a negative output size, "
                 "which is invalid. Check dimension ", l_diff + i, " of your input.");
        new_shape.emplace_back(new_dim);
    }

    at::Tensor output;
    const auto memory_format = self.suggest_memory_format();
    if (self.is_quantized()) {
        const auto qscheme = self.qscheme();
        TORCH_CHECK(qscheme == kPerTensorAffine || qscheme == kPerTensorSymmetric,
                    "Only per-tensor padding is supported.");
        output = at::_empty_affine_quantized(
            new_shape, self.options().memory_format(memory_format),
            self.q_scale(), self.q_zero_point(), std::nullopt);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 91-120
```cpp
    } else {
        output = at::empty(new_shape, self.options().memory_format(memory_format));
    }
    output.fill_(value);

    auto c_output = output;
    for (const auto i : c10::irange(l_diff, l_inp)) {
        auto pad_idx = 2 * (l_inp - i - 1);
        if (pad[pad_idx] > 0) {
            c_output = c_output.narrow(i, pad[pad_idx], c_output.size(i) - pad[pad_idx]);
        }
        if (pad[pad_idx + 1] > 0) {
            c_output = c_output.narrow(i, 0, c_output.size(i) - pad[pad_idx + 1]);
        }
    }
    c_output.copy_(c_input);
    return output;
}

Tensor _pad_circular_symint(const Tensor &self, c10::SymIntArrayRef padding) {
  const auto in_shape = self.sym_sizes();
  const auto self_ndim = static_cast<int64_t>(in_shape.size());

  // number of dimensions that are padded
  const auto ndim_padded = padding.size() / 2;
  // number of preceding non_padded dimensions (1 for no_batch_dim case or 2)
  const auto ndim_nonpadded = self_ndim - ndim_padded;

  TORCH_CHECK(ndim_nonpadded == 1 || ndim_nonpadded == 2,
              "Invalid padding size, expected 1 or 2 non-padded dimensions, ",
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _pad_circular_symint, dimensions, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _pad_circular_symint, dimensions，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-150
```cpp
              "which would be equivalent to padding of length ",
              (self_ndim - 1) * 2,
              " or ",
              (self_ndim - 2) * 2,
              " respectively but got ",
              padding.size());

  c10::SymDimVector out_shape(in_shape.size());
  for (const auto i: c10::irange(ndim_nonpadded)) {
    out_shape[i] = in_shape[i];
  }

  // Get shape of padded tensor
  for (const auto i : c10::irange(ndim_padded)) {
    const auto& pad_l = padding[2 * (ndim_padded - i - 1) + 0];
    const auto& pad_r = padding[2 * (ndim_padded - i - 1) + 1];
    const auto& size = in_shape[ndim_nonpadded + i];
    out_shape[ndim_nonpadded + i] = size + pad_l + pad_r;

    TORCH_CHECK(
        pad_l <= size && pad_r <= size,
        "Padding value causes wrapping around more than once.");
    TORCH_CHECK(
        out_shape[ndim_nonpadded + i] >= 0,
        "Negative padding value is resulting in an empty dimension");
  }

  auto out = self.new_empty_symint(out_shape, self.options());

  // Put original array into the padded array
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 151-180
```cpp
  Tensor out_slice = out;
  Tensor in_slice = self;
  const SymInt zero = 0;
  for (const auto i : c10::irange(ndim_padded)) {
    const auto dim = ndim_padded - i + ndim_nonpadded - 1;
    const auto& pad_l = padding[2*i + 0];
    const auto& pad_r = padding[2*i + 1];
    out_slice = out_slice.slice_symint(dim, std::max(pad_l, zero), out_shape[dim] - std::max(pad_r, zero));
    in_slice = in_slice.slice_symint(dim, std::max(-pad_l, zero), in_shape[dim] - std::max(-pad_r, zero));
  }
  out_slice.copy_(in_slice);

  // The following steps first pad the beginning of the tensor (left side),
  // and then pad the end of the tensor (right side).
  // Note: Corners will be written more than once when ndim_padded > 1.
  //
  // Only in cases where padding values are > 0 are when additional copying
  // is required.
  for (const auto i : c10::irange(ndim_padded)) {
    const auto dim = ndim_padded - i + ndim_nonpadded - 1;
    const auto& pad_l = padding[2*i + 0];
    const auto& pad_r = padding[2*i + 1];

    if (pad_l > 0) {
      out_slice = out.slice_symint(dim, 0, pad_l);
      in_slice = out.slice_symint(dim,
                           out_shape[dim] - pad_l - std::max(pad_r, zero),
                           out_shape[dim] - std::max(pad_r, zero));
      out_slice.copy_(in_slice);
    }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-210
```cpp

    if (pad_r > 0) {
      out_slice = out.slice_symint(dim, out_shape[dim] - pad_r, out_shape[dim]);
      in_slice = out.slice_symint(dim, std::max(pad_l, zero), std::max(pad_l, zero) + pad_r);
      out_slice.copy_(in_slice);
    }
  }

  return out;
}

static std::string_view padding_mode_string(padding_mode m) {
  switch (m) {
    case padding_mode::reflect:
      return "reflect";
    case padding_mode::replicate:
      return "replicate";
    case padding_mode::circular:
      return "circular";
    case padding_mode::constant:
      return "constant";
  }
  TORCH_CHECK(false, "Invalid padding mode (", static_cast<int64_t>(m), ")");
}


Tensor _pad_enum_symint(const Tensor &self, c10::SymIntArrayRef pad, int64_t mode_int, std::optional<double> value) {
  const auto input_dim = self.dim();
  TORCH_CHECK(pad.size() % 2 == 0, "Padding length must be divisible by 2");
  TORCH_CHECK(static_cast<int64_t>(pad.size()) <= input_dim * 2,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are padding_mode_string, _pad_enum_symint, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 padding_mode_string, _pad_enum_symint，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 211-240
```cpp
              "Padding length should be less than or equal to two times the input dimension but got padding length ", pad.size(), " and input of dimension ", input_dim);
  auto mode = static_cast<at::padding_mode>(mode_int);

  if (mode == at::padding_mode::constant) {
    return at::constant_pad_nd_symint(self, pad, value.value_or(0.0));
  }
  TORCH_CHECK(!value.has_value() || *value == 0,
              "Padding mode \"", padding_mode_string(mode),
              "\" doesn't take in value argument");

  if (pad.size() == 2 && (input_dim == 2 || input_dim == 3)) {
    switch (mode) {
      case at::padding_mode::reflect: return at::reflection_pad1d_symint(self, pad);
      case at::padding_mode::replicate: return at::replication_pad1d_symint(self, pad);
      case at::padding_mode::circular: return at::_pad_circular_symint(self, pad);
      default: {}
    }
  } else if(pad.size() == 4 && (input_dim == 3 || input_dim == 4)) {
    switch (mode) {
      case at::padding_mode::reflect: return at::reflection_pad2d_symint(self, pad);
      case at::padding_mode::replicate: return at::replication_pad2d_symint(self, pad);
      case at::padding_mode::circular: return at::_pad_circular_symint(self, pad);
      default: {}
    }
  } else if (pad.size() == 6 && (input_dim == 4 || input_dim == 5)) {
    switch (mode) {
      case at::padding_mode::reflect: return at::reflection_pad3d_symint(self, pad);
      case at::padding_mode::replicate: return at::replication_pad3d_symint(self, pad);
      case at::padding_mode::circular: return at::_pad_circular_symint(self, pad);
      default: {}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-270
```cpp
    }
  }

  std::ostringstream error_msg;
  error_msg << "Padding size " << pad.size() << " is not supported for " << input_dim << "D input tensor.\n";
  error_msg << "Supported combinations for non-constant padding:\n";
  error_msg << "  - 2D or 3D input: padding size = 2 (pads last dimension)\n";
  error_msg << "  - 3D or 4D input: padding size = 4 (pads last 2 dimensions)\n";
  error_msg << "  - 4D or 5D input: padding size = 6 (pads last 3 dimensions)";

  C10_THROW_ERROR(NotImplementedError, error_msg.str());
}

Tensor pad_symint(const Tensor &self, c10::SymIntArrayRef pad, std::string_view mode, std::optional<double> value) {
  const auto mode_enum = [&] {
    if (mode == "reflect") {
      return at::padding_mode::reflect;
    } else if (mode == "constant") {
      return at::padding_mode::constant;
    } else if (mode == "replicate") {
      return at::padding_mode::replicate;
    } else if (mode == "circular") {
      return at::padding_mode::circular;
    }
    C10_THROW_ERROR(NotImplementedError,
                    c10::str("Unrecognised padding mode ", mode));
  }();
  return at::native::_pad_enum_symint(self, pad, static_cast<int64_t>(mode_enum), value);
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are pad_symint, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 pad_symint，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 271-271
```cpp
}  // namespace at::native
```
- EN: This range provides supporting glue code: declarations, simple forwarding logic, or structural scaffolding reused by nearby operator code.
- CN: 这一段提供了辅助性胶水代码：声明、简单转发逻辑，或被邻近算子代码复用的结构骨架。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: constant_pad_nd, _pad_circular_symint, padding_mode_string, _pad_enum_symint, pad_symint.
- CN: 重要符号：constant_pad_nd, _pad_circular_symint, padding_mode_string, _pad_enum_symint, pad_symint。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/PadNd.h, ATen/core/Tensor.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_empty_affine_quantized.h, ATen/ops/_pad_circular.h, ATen/ops/_pad_circular_native.h, ATen/ops/_pad_enum_native.h, ATen/ops/constant_pad_nd.h, ATen/ops/constant_pad_nd_native.h, ATen/ops/empty.h`.
- CN: 主要内部头文件：`ATen/PadNd.h, ATen/core/Tensor.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_empty_affine_quantized.h, ATen/ops/_pad_circular.h, ATen/ops/_pad_circular_native.h, ATen/ops/_pad_enum_native.h, ATen/ops/constant_pad_nd.h, ATen/ops/constant_pad_nd_native.h, ATen/ops/empty.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `constant_pad_nd, _pad_circular_symint, padding_mode_string, _pad_enum_symint, pad_symint`.
- CN: 实现围绕 `constant_pad_nd, _pad_circular_symint, padding_mode_string, _pad_enum_symint, pad_symint` 等符号展开。
