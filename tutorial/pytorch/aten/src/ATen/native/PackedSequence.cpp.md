# PackedSequence.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/PackedSequence.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on packed sequence; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 packed sequence；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_pack_padded_sequence_backward_native.h>
#include <ATen/ops/_pack_padded_sequence_native.h>
#include <ATen/ops/_pad_packed_sequence_native.h>
#include <ATen/ops/cat.h>
#include <ATen/ops/empty.h>
#include <ATen/ops/full.h>
#include <ATen/ops/pad_sequence_native.h>
#include <ATen/ops/zeros.h>
#include <ATen/ops/zeros_like_ops.h>
#endif

#include <c10/util/irange.h>

namespace at::native {

static void checkLongTensor(const Tensor& tensor) {
  TORCH_CHECK(tensor.dim() == 1 && tensor.device().type() == at::kCPU && tensor.scalar_type() == at::kLong,
           "'lengths' argument should be a 1D CPU int64 tensor, but got ",
            tensor.dim(), "D ", tensor.device().str(), " ", tensor.scalar_type(), " tensor");
}

// This method returns `(data, batch_sizes)`, which are then passed into a
// `PackedSequence` constructor.
```
- EN: Lines 1-30 pull in 13 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are checkLongTensor, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 13 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 checkLongTensor，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
// `data` can be on arbitrary device and of arbitrary dtype, but `batch_sizes`
// must be a CPU int64 tensor.
// See NOTE [ device and dtype of a PackedSequence ]
std::tuple<Tensor, Tensor> _pack_padded_sequence(const Tensor& _input, const Tensor& _lengths, bool batch_first) {
  TORCH_CHECK(_input.numel() > 0, "Cannot pack empty tensors.");
  auto input = batch_first ? _input.transpose(0, 1) : _input;
  auto lengths_t = _lengths.contiguous();
  checkLongTensor(lengths_t);

  int64_t batch_size = input.size(1);
  const int64_t * lengths = lengths_t.const_data_ptr<int64_t>();

  TORCH_CHECK(lengths_t.size(0) == batch_size,
           "Expected `len(lengths)` to be equal to batch_size, but got ", lengths_t.size(0),
           " (batch_size=", batch_size, ")");
  TORCH_CHECK(lengths[batch_size - 1] > 0,
           "Length of all samples has to be greater than 0, but found an element "
           "in 'lengths' that is <= 0");
  for (const auto i : c10::irange(batch_size - 1)) {
    if (lengths[batch_size - 1 - i] > lengths[batch_size - 2 - i]) {
      // NB: enforce_sorted is implemented at a Python level, but the sortedness
      // check lives here. If enforce_sorted=False then this error should never
      // get called.
      TORCH_CHECK(false, "`lengths` array must be sorted in decreasing order when "
               "`enforce_sorted` is True. You can pass `enforce_sorted=False` "
               "to pack_padded_sequence and/or pack_sequence to sidestep this "
               "requirement if you do not need ONNX exportability.");
    }
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _pack_padded_sequence, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _pack_padded_sequence，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-90
```cpp
  std::vector<at::Tensor> steps;
  steps.reserve(batch_size);
  at::Tensor batch_sizes_t = at::empty(lengths[0], _lengths.options());
  int64_t * batch_sizes = batch_sizes_t.mutable_data_ptr<int64_t>();

  std::vector<int64_t> step_shape; // == [-1, *input.shape[2:]]
  {
    auto input_sizes = input.sizes();
    step_shape.reserve(input_sizes.size());
    auto s_input_sizes = input_sizes.slice(2);
    step_shape.push_back(-1);
    step_shape.insert(step_shape.end(), s_input_sizes.begin(), s_input_sizes.end());
  }

  // To understand what's going on in this loop imagine that the input is a padded 2D
  // array that looks like this (x = valid entry, . = padding)
  //
  //  1 1 1 1 1
  //  2 2 2 . .
  //  2 2 2 . .
  //  4 . . . .
  //  4 . . . .
  //
  // Where the vertical dimension corresponds to time, and horizontal dim to batch.
  // In this example, the lengths array will be equal to [5, 3, 3, 1, 1], and we will
  // iterate over them in reverse order (from the rightmost column to the left).
  // We want to avoid eager slicing of the input at every time step, and wait for
  // the moments where the length increases. In this example, that will happen at the
  // first, second and fourth steps. Then, we slice out the whole block of the input
  // that corresponds to this length, and hasn't been sliced yet (the steps at which each
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are this, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 this，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
  // element is sliced are annotated in the array above).  You can think of this as if we
  // were scanning the sequences from the shortest one, and every time we realize there's
  // more elements below in our column, we lower the counter (prev_l), and append the new
  // block to the output.
  int64_t prev_l = 0;
  for (const auto i : c10::irange(batch_size)) {
    int64_t l = lengths[batch_size - 1 - i];
    if (l > prev_l) {
      auto current_batch_size = batch_size - i;
      steps.push_back(input.slice(0, prev_l, l).slice(1, 0, current_batch_size).contiguous().view(step_shape));
      for (int64_t j = 0; j < (l - prev_l); ++j) {
        (*batch_sizes++) = current_batch_size;
      }
      prev_l = l;
    }
    TORCH_CHECK(l >= prev_l);
  }

  return std::make_tuple(at::cat(steps), std::move(batch_sizes_t));
}

// `grad` could be on arbitrary device and of arbitrary dtype, but `_batch_sizes`
// is guaranteed to be a CPU int64 tensor.
// See NOTE [ device and dtype of a PackedSequence ]
Tensor _pack_padded_sequence_backward_symint(const Tensor& grad, c10::SymIntArrayRef input_size, const Tensor& _batch_sizes, bool batch_first) {
  std::vector<c10::SymInt> input_size_after_t = input_size.vec();
  if (batch_first) {
    TORCH_CHECK(input_size.size() >= 2);
    std::swap(input_size_after_t[0], input_size_after_t[1]);
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _pack_padded_sequence_backward_symint, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _pack_padded_sequence_backward_symint，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-150
```cpp
  auto grad_input = at::zeros_symint(input_size_after_t, grad.options());
  auto batch_sizes_t = _batch_sizes.contiguous();
  checkLongTensor(batch_sizes_t);

  int64_t offset = 0;
  // NOTE: this op advertises as CompositeImplicitAutograd, but uses data_ptr().
  // we should fix this.
  auto max_seq_len = batch_sizes_t.size(0);
  const int64_t * batch_sizes = batch_sizes_t.const_data_ptr<int64_t>();
  for (const auto i : c10::irange(max_seq_len)) {
    grad_input[i].slice(0, 0, batch_sizes[i]).copy_(grad.slice(0, offset, offset + batch_sizes[i]));
    offset += batch_sizes[i];
  }

  if (batch_first) {
    grad_input = grad_input.transpose(0, 1);
  }

  return grad_input;
}

std::tuple<Tensor, Tensor> _pad_packed_sequence(const Tensor& data, const Tensor& _batch_sizes, bool batch_first, const Scalar& padding_value, int64_t total_length) {
  auto batch_sizes_t = _batch_sizes.contiguous();
  checkLongTensor(batch_sizes_t);
  TORCH_CHECK(batch_sizes_t.numel() > 0, "batch_sizes can not be empty");

  const int64_t * batch_sizes = batch_sizes_t.const_data_ptr<int64_t>();
  int64_t max_batch_size = batch_sizes[0];
  int64_t max_real_seq_length = batch_sizes_t.size(0);
  int64_t max_seq_length = max_real_seq_length;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _pad_packed_sequence, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _pad_packed_sequence，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 151-180
```cpp
  if (total_length > 0) {
    TORCH_CHECK(total_length >= max_seq_length,
             "Expected total_length to be at least the length of the longest "
             "sequence in input, but got total_length=", total_length, " and "
             "max sequence length being ", max_seq_length);
    max_seq_length = total_length;
  }

  std::vector<int64_t> output_size; // == [max_seq_length, max_batch_size, *var_data.size()[1:]]
  {
    output_size.reserve(data.dim() + 1);
    output_size.push_back(max_seq_length);
    output_size.push_back(max_batch_size);
    auto s_data_size = data.sizes().slice(1);
    output_size.insert(output_size.end(), s_data_size.begin(), s_data_size.end());
  }
  auto output = at::full(output_size, padding_value, data.options());

  // This will be modified at every iteration, but we reserve memory for it now.
  std::vector<int64_t> tmp_view_size = std::move(output_size); // == [-1, -1, *var_data.size()[1:]]

  at::Tensor lengths_t = at::empty(max_batch_size, batch_sizes_t.options());
  int64_t * lengths = lengths_t.mutable_data_ptr<int64_t>() + max_batch_size - 1;
  int64_t data_offset = 0;
  int64_t prev_batch_size = max_batch_size;
  int64_t prev_i = 0;
  for (int64_t i = 0; i <= max_real_seq_length; ++i) {
    int64_t batch_size = i != max_real_seq_length ? batch_sizes[i] : 0;
    if (batch_size != prev_batch_size) {
      int64_t l = prev_batch_size * (i - prev_i);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-210
```cpp
      // The lines below are equivalent to this:
      // output[prev_i:i, :prev_batch_size] = tmp.view(i - prev_i, prev_batch_size, *input.shape[2:])
      auto tmp = data.slice(0, data_offset, data_offset + l);
      tmp_view_size[0] = i - prev_i;
      tmp_view_size[1] = prev_batch_size;
      output.slice(0, prev_i, i).slice(1, 0, prev_batch_size).copy_(tmp.view(tmp_view_size));
      data_offset += l;
      prev_i = i;
    }
    int64_t dec = prev_batch_size - batch_size;
    if (dec > 0) {
      for ([[maybe_unused]] const auto j : c10::irange(dec)) {
        (*lengths--) = i;
      }
    }
    prev_batch_size = batch_size;
  }

  if (batch_first) {
    output = output.transpose(0, 1);
  }

  return std::make_tuple(std::move(output), std::move(lengths_t));
}

Tensor pad_sequence(TensorList sequences, bool batch_first, double padding_value, const std::string_view padding_side) {
  const int64_t sequences_size = sequences.size();
  TORCH_CHECK(sequences_size > 0, "received an empty list of sequences");
  TORCH_CHECK(padding_side == "left" || padding_side == "right",
              "Expected padding_side to be one of left or right, but got ", padding_side, ".");
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are view, pad_sequence, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 view, pad_sequence，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 211-240
```cpp
  IntArrayRef max_size = sequences[0].sizes();
  IntArrayRef trailing_dims = max_size.slice(1);
  int64_t max_len = std::max_element(
    sequences.begin(),
    sequences.end(),
    [](const Tensor &a, const Tensor &b) {
      return a.size(0) < b.size(0);
    }
  )->size(0);

  DimVector out_dims;
  if (batch_first) {
    out_dims = {sequences_size, max_len};
  } else {
    out_dims = {max_len, sequences_size};
  }
  out_dims.insert(out_dims.end(), trailing_dims.begin(), trailing_dims.end());

  Tensor out = at::full(out_dims, padding_value, sequences[0].options());
  for (const auto i : c10::irange(sequences_size)) {
    const Tensor& currseq = sequences[i];
    const int64_t length_i = currseq.size(0);
    const int64_t start = padding_side == "left" ? max_len - length_i : 0;
    // use index notation to prevent duplicate references to the tensor
    if (batch_first) {
      out.select(0, i).narrow(0, start, length_i).copy_(currseq);
    } else {
      out.narrow(0, start, length_i).select(1, i).copy_(currseq);
    }
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-244
```cpp
  return out;
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: checkLongTensor, _pack_padded_sequence, _pack_padded_sequence_backward_symint, _pad_packed_sequence, pad_sequence.
- CN: 重要符号：checkLongTensor, _pack_padded_sequence, _pack_padded_sequence_backward_symint, _pad_packed_sequence, pad_sequence。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_pack_padded_sequence_backward_native.h, ATen/ops/_pack_padded_sequence_native.h, ATen/ops/_pad_packed_sequence_native.h, ATen/ops/cat.h, ATen/ops/empty.h, ATen/ops/full.h, ATen/ops/pad_sequence_native.h, ATen/ops/zeros.h, ATen/ops/zeros_like_ops.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_pack_padded_sequence_backward_native.h, ATen/ops/_pack_padded_sequence_native.h, ATen/ops/_pad_packed_sequence_native.h, ATen/ops/cat.h, ATen/ops/empty.h, ATen/ops/full.h, ATen/ops/pad_sequence_native.h, ATen/ops/zeros.h, ATen/ops/zeros_like_ops.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `checkLongTensor, _pack_padded_sequence, _pack_padded_sequence_backward_symint, _pad_packed_sequence, pad_sequence`.
- CN: 实现围绕 `checkLongTensor, _pack_padded_sequence, _pack_padded_sequence_backward_symint, _pad_packed_sequence, pad_sequence` 等符号展开。
