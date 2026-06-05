# constant_fold.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/csrc/jit/passes/onnx/constant_fold.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements the TorchScript JIT pass logic for constant fold, including graph analysis and rewrites.
- 用途 (CN): 实现与 constant fold 相关的 TorchScript JIT pass 逻辑，包括图分析与改写。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11
```cpp
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/onnx/constant_fold.h>
#include <torch/csrc/jit/passes/onnx/helper.h>

#include <ATen/Functions.h>

#include <c10/util/Exception.h>
#include <c10/util/irange.h>
#include <algorithm>
#include <optional>

```
- EN: Pulls in the headers needed by the constant fold logic. Internal dependencies: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/constant_fold.h`, `torch/csrc/jit/passes/onnx/helper.h`, `ATen/Functions.h`, `c10/util/Exception.h`, `...`; external dependencies: `algorithm`, `optional`.
- CN: 为 constant fold 相关逻辑引入所需头文件。内部依赖：`torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/constant_fold.h`, `torch/csrc/jit/passes/onnx/helper.h`, `ATen/Functions.h`, `c10/util/Exception.h`, `...`；外部依赖：`algorithm`, `optional`。

### Lines 12-20
```cpp
namespace torch::jit {

namespace onnx {
using namespace ::c10::onnx;
}

namespace onnx_constant_fold {

enum OnnxType : int {
```
- EN: Declares core types or data containers for this file. Prominent symbols: `namespace`, `OnnxType`.
- CN: 声明该文件使用的核心类型或数据容器。主要符号：`namespace`, `OnnxType`。

### Lines 21-31
```cpp
  ONNX_FLOAT = 1,
  ONNX_UINT8,
  ONNX_INT8,
  ONNX_UINT16,
  ONNX_INT16,
  ONNX_INT32,
  ONNX_INT64,
  ONNX_FLOAT16 = 10,
  ONNX_DOUBLE,
  ONNX_UINT32,
};
```
- EN: This block implements local helper logic for constant fold. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 constant fold 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 32-47
```cpp

static std::unordered_map<int, at::ScalarType> onnxTypeToScalarTypeMap = {
    // Only conversion of ONNX numeric types is included here.
    // Unsigned ONNX types are mapped to the next higher signed
    // ScalarType type.
    {ONNX_FLOAT, at::kFloat},
    {ONNX_UINT8, at::kByte},
    {ONNX_INT8, at::kChar},
    {ONNX_UINT16, at::kInt},
    {ONNX_INT16, at::kShort},
    {ONNX_INT32, at::kInt},
    {ONNX_INT64, at::kLong},
    {ONNX_FLOAT16, at::kFloat},
    {ONNX_DOUBLE, at::kDouble},
    {ONNX_UINT32, at::kLong},
};
```
- EN: This block implements local helper logic for constant fold. Key symbols: no dominant local symbol names.
- CN: 该代码块实现与 constant fold 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 48-57
```cpp

static void handleNegativeStartEndIndex(
    int64_t& start,
    int64_t& end,
    int64_t& axis,
    c10::IntArrayRef tensorSizes) {
  if (start < 0) {
    start = tensorSizes[axis] + start;
  }
  if (end < 0) {
```
- EN: This block handles conditional branches. Key symbols: `handleNegativeStartEndIndex`.
- CN: 该代码块处理条件分支。关键符号：`handleNegativeStartEndIndex`。

### Lines 58-66
```cpp
    end = tensorSizes[axis] + end;
  }
  // index higher than dimension is treated as the end.
  if (end > tensorSizes[axis]) {
    end = tensorSizes[axis];
  }
}

static std::optional<at::Tensor> runTorchSlice_opset9(
```
- EN: This block handles conditional branches. Key symbols: `runTorchSlice_opset9`.
- CN: 该代码块处理条件分支。关键符号：`runTorchSlice_opset9`。

### Lines 67-76
```cpp
    const Node* node,
    std::vector<at::Tensor>& inputTensorValues) {
  assert(inputTensorValues.size() == 1);
  if (inputTensorValues.size() != 1) {
    TORCH_WARN(
        "Constant folding - Invalid number of inputs found for opset 9 "
        "onnx::Slice op. Constant folding not applied.");
    return std::nullopt;
  }
  if (!(node->hasAttributeS("starts") && node->hasAttributeS("ends"))) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `assert`, `size`, `hasAttributeS`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`assert`, `size`, `hasAttributeS`。

### Lines 77-85
```cpp
    return std::nullopt;
  }
  auto const& startsAttr = node->is(attr::starts);
  auto const& endsAttr = node->is(attr::ends);
  if (startsAttr.size() != endsAttr.size()) {
    return std::nullopt;
  }
  std::vector<int64_t> axesAttr;
  if (node->hasAttributeS("axes")) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `is`, `size`, `hasAttributeS`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`is`, `size`, `hasAttributeS`。

### Lines 86-99
```cpp
    axesAttr = node->is(attr::axes);
  } else {
    axesAttr.resize(startsAttr.size());
    std::iota(axesAttr.begin(), axesAttr.end(), 0);
  }
  auto updated_val = inputTensorValues[0];
  for (const auto i : c10::irange(axesAttr.size())) {
    // ONNX slice accepts negative starts and ends values.
    int64_t axis = axesAttr[i], start = startsAttr[i], end = endsAttr[i];
    // ONNX slice accepts negative axis, fix this for aten op
    axis += axis < 0 ? inputTensorValues[0].sizes().size() : 0;
    handleNegativeStartEndIndex(start, end, axis, updated_val.sizes());
    int64_t length = end - start;
    if (length < 0 || start > updated_val.sizes()[axis] - length)
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `is`, `resize`, `size`, `iota`, `begin`, `end`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`is`, `resize`, `size`, `iota`, `begin`, `end`, `...`。

### Lines 100-108
```cpp
      return std::nullopt;
    updated_val = at::narrow(updated_val, axis, start, length);
  }
  return std::optional<at::Tensor>(updated_val);
}

static std::optional<at::Tensor> runTorchSlice_opset10(
    const Node* node,
    std::vector<at::Tensor>& inputTensorValues) {
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `narrow`, `runTorchSlice_opset10`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`narrow`, `runTorchSlice_opset10`。

### Lines 109-119
```cpp
  const int maxSliceInputCount = 5;
  const int minSliceInputCount = 3;
  if (inputTensorValues.size() < minSliceInputCount ||
      inputTensorValues.size() > maxSliceInputCount) {
    TORCH_WARN(
        "Constant folding - Invalid number of inputs found for opset opset >= 10 onnx::Slice op. "
        "Constant folding not applied.");
    return std::nullopt;
  }
  // Checking validity of 'starts' and 'ends' input
  if (inputTensorValues[1].sizes().size() != 1 ||
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `size`, `sizes`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`size`, `sizes`。

### Lines 120-129
```cpp
      inputTensorValues[2].sizes().size() != 1) {
    TORCH_WARN(
        "Constant folding - Invalid 'starts' or 'ends' inputs found for opset >= 10 onnx::Slice op. "
        "Constant folding not applied.");
    return std::nullopt;
  }
  if (inputTensorValues[1].sizes()[0] != inputTensorValues[2].sizes()[0]) {
    // Number of elements of 'starts' and 'ends' 1-D input tensors should be the
    // same
    return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `sizes`, `size`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`sizes`, `size`。

### Lines 130-138
```cpp
  }
  // Checking 'axes' input, if available.
  std::vector<int64_t> axes;
  if (inputTensorValues.size() > 3) {
    if (inputTensorValues[3].sizes().size() != 1) {
      TORCH_WARN(
          "Constant folding - Invalid 'axes' input found for opset >= 10 onnx::Slice op. "
          "Constant folding not applied.");
      return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `size`, `sizes`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`size`, `sizes`。

### Lines 139-148
```cpp
    }
    if (inputTensorValues[3].sizes()[0] != inputTensorValues[1].sizes()[0]) {
      // Number of elements of 'axes' and 'ends' 1-D input tensors should be the
      // same
      TORCH_WARN(
          "Constant folding - Invalid 'axes' or 'ends' inputs found for opset >= 10 onnx::Slice op. "
          "Constant folding not applied.");
      return std::nullopt;
    }
    auto axes_a = inputTensorValues[3].accessor<int64_t, 1>();
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `sizes`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`sizes`。

### Lines 149-159
```cpp
    axes.resize(inputTensorValues[3].sizes()[0]);
    // ONNX slice accepts negative axis, fix this for aten op
    for (const auto i : c10::irange(inputTensorValues[3].sizes()[0])) {
      axes[i] = axes_a[i] < 0 ? axes_a[i] + inputTensorValues[0].sizes().size()
                              : axes_a[i];
    }
  } else {
    axes = std::vector<int64_t>(inputTensorValues[1].sizes()[0], 0);
  }
  // Checking 'steps' input, if available.
  if (inputTensorValues.size() > 4) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `resize`, `sizes`, `irange`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`resize`, `sizes`, `irange`, `size`。

### Lines 160-172
```cpp
    if (inputTensorValues[4].sizes().size() != 1) {
      TORCH_WARN(
          "Constant folding - Invalid 'steps' input found for opset >= 10 onnx::Slice op. "
          "Constant folding not applied.");
      return std::nullopt;
    }
    if (inputTensorValues[4].sizes()[0] != inputTensorValues[1].sizes()[0]) {
      // Number of elements of 'steps' and 'ends' 1-D input tensors should be
      // the same
      TORCH_WARN(
          "Constant folding - Invalid 'steps' or 'ends' inputs found for opset >= 10 onnx::Slice op. "
          "Constant folding not applied.");
      return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `sizes`, `size`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`sizes`, `size`。

### Lines 173-181
```cpp
    }
    auto steps_a = inputTensorValues[4].accessor<int64_t, 1>();
    for (const auto i : c10::irange(inputTensorValues[4].sizes()[0])) {
      // Only steps == 1 are supported for constant-folding.
      if (steps_a[i] != 1) {
        TORCH_WARN(
            "Constant folding - Only steps=1 can be constant folded for opset >= 10 onnx::Slice op. "
            "Constant folding not applied.");
        return std::nullopt;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `irange`, `sizes`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`irange`, `sizes`。

### Lines 182-193
```cpp
      }
    }
  }
  auto starts_a = inputTensorValues[1].accessor<int64_t, 1>();
  auto ends_a = inputTensorValues[2].accessor<int64_t, 1>();
  auto updated_val = inputTensorValues[0];
  for (const auto i : c10::irange(inputTensorValues[1].sizes()[0])) {
    // ONNX slice accepts negative starts and ends values.
    int64_t start = starts_a[i], end = ends_a[i], axis = axes[i];
    handleNegativeStartEndIndex(start, end, axis, updated_val.sizes());
    int64_t length = end - start;
    if (length < 0 || start > updated_val.sizes()[axis] - length)
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `irange`, `sizes`, `handleNegativeStartEndIndex`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`irange`, `sizes`, `handleNegativeStartEndIndex`。

### Lines 194-205
```cpp
      return std::nullopt;
    updated_val = at::narrow(updated_val, axis, start, length);
  }
  return std::optional<at::Tensor>(updated_val);
}

// Refer to AT_FORALL_SCALAR_TYPES_WITH_COMPLEX_EXCEPT_COMPLEX_HALF
static at::Tensor runTorchArange_opset11(
    const Node* node,
    const std::vector<at::Tensor>& inputTensorValues) {
  TORCH_INTERNAL_ASSERT(inputTensorValues.size() == 3);
  auto dtype = inputTensorValues[0].scalar_type();
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `narrow`, `runTorchArange_opset11`, `size`, `scalar_type`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`narrow`, `runTorchArange_opset11`, `size`, `scalar_type`。

### Lines 206-216
```cpp
  at::Tensor updated_val;
  switch (dtype) {
    case at::ScalarType::Float: {
      auto start = inputTensorValues[0].item<float>();
      auto end = inputTensorValues[1].item<float>();
      auto step = inputTensorValues[2].item<float>();
      updated_val = at::arange(start, end, step);
      break;
    }
    case at::ScalarType::Double: {
      auto start = inputTensorValues[0].item<double>();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `arange`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`arange`。

### Lines 217-225
```cpp
      auto end = inputTensorValues[1].item<double>();
      auto step = inputTensorValues[2].item<double>();
      updated_val = at::arange(start, end, step);
      break;
    }
    case at::ScalarType::Short: {
      auto start = inputTensorValues[0].item<int16_t>();
      auto end = inputTensorValues[1].item<int16_t>();
      auto step = inputTensorValues[2].item<int16_t>();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `arange`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`arange`。

### Lines 226-237
```cpp
      updated_val = at::arange(start, end, step);
      break;
    }
    case at::ScalarType::Int: {
      auto start = inputTensorValues[0].item<int>();
      auto end = inputTensorValues[1].item<int>();
      auto step = inputTensorValues[2].item<int>();
      updated_val = at::arange(start, end, step);
      break;
    }
    case at::ScalarType::Long: {
      auto start = inputTensorValues[0].item<int64_t>();
```
- EN: This block manipulates TorchScript IR objects. Key symbols: `arange`.
- CN: 该代码块操作 TorchScript IR 对象。关键符号：`arange`。

### Lines 238-248
```cpp
      auto end = inputTensorValues[1].item<int64_t>();
      auto step = inputTensorValues[2].item<int64_t>();
      updated_val = at::arange(start, end, step);
      break;
    }
    default: {
      TORCH_WARN(
          "Constant folding - ONNX Range type: ", dtype, " is not supported.");
    }
  }
  return updated_val;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `arange`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`arange`。

### Lines 249-258
```cpp
}

at::Tensor IntToTensor(int64_t value) {
  auto options = c10::TensorOptions().dtype(at::kLong).device(at::kCPU);
  std::vector<int64_t> size_data = {value};
  auto f = at::from_blob(size_data.data(), {1}, at::kLong).to(at::kCPU);
  // Need copy here
  at::Tensor f_copy = at::empty({1}, options);
  f_copy.copy_(f);
  return at::squeeze(f_copy, 0);
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `IntToTensor`, `TensorOptions`, `dtype`, `device`, `from_blob`, `data`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`IntToTensor`, `TensorOptions`, `dtype`, `device`, `from_blob`, `data`, `...`。

### Lines 259-267
```cpp
}

std::optional<at::Tensor> runTorchBackendForOnnx(
    const Node* node,
    std::vector<at::Tensor>& inputTensorValues,
    int opset_version) {
  at::Tensor updated_val;
  if (node->kind() == onnx::Slice) {
    if (opset_version == ONNX_OPSET_9) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `runTorchBackendForOnnx`, `kind`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`runTorchBackendForOnnx`, `kind`。

### Lines 268-277
```cpp
      return runTorchSlice_opset9(node, inputTensorValues);
    } else if (opset_version >= ONNX_OPSET_10) {
      return runTorchSlice_opset10(node, inputTensorValues);
    } else {
      TORCH_WARN(
          "Constant folding - unsupported opset version. Constant folding not applied.");
      return std::nullopt;
    }
  } else if (node->kind() == onnx::Concat) {
    if (!node->hasAttributeS("axis")) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `runTorchSlice_opset9`, `runTorchSlice_opset10`, `kind`, `hasAttributeS`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`runTorchSlice_opset9`, `runTorchSlice_opset10`, `kind`, `hasAttributeS`。

### Lines 278-290
```cpp
      return std::nullopt;
    }
    updated_val =
        at::cat(at::TensorList(inputTensorValues), node->i(attr::axis));
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Sqrt) {
    updated_val = at::sqrt(inputTensorValues[0]);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Div) {
    // One example shows at::div(CPULongType, CPULongType) = CPUFloatType,
    // So we add a cast below.
    updated_val = at::div(inputTensorValues[0], inputTensorValues[1]);
    if (inputTensorValues[0].scalar_type() ==
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `cat`, `TensorList`, `i`, `kind`, `sqrt`, `div`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`cat`, `TensorList`, `i`, `kind`, `sqrt`, `div`, `...`。

### Lines 291-300
```cpp
        inputTensorValues[1].scalar_type()) {
      updated_val = updated_val.to(inputTensorValues[0].scalar_type());
    }
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Mul) {
    updated_val = at::mul(inputTensorValues[0], inputTensorValues[1]);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Sub) {
    updated_val = at::sub(inputTensorValues[0], inputTensorValues[1]);
    return std::optional<at::Tensor>(updated_val);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `scalar_type`, `to`, `kind`, `mul`, `sub`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`scalar_type`, `to`, `kind`, `mul`, `sub`。

### Lines 301-312
```cpp
  } else if (node->kind() == onnx::Add) {
    updated_val = at::add(inputTensorValues[0], inputTensorValues[1]);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Unsqueeze) {
    if (opset_version >= ONNX_OPSET_13) {
      assert(inputTensorValues.size() == 2);
      // Checking validity of 'axes' input
      if (inputTensorValues[1].sizes().size() != 1) {
        TORCH_WARN(
            "Constant folding - Invalid 'axes' inputs found for opset 13 onnx::Unsqueeze op. "
            "Constant folding not applied.");
        return std::nullopt;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `kind`, `add`, `assert`, `size`, `sizes`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`kind`, `add`, `assert`, `size`, `sizes`。

### Lines 313-325
```cpp
      }
      auto axes_a = inputTensorValues[1].accessor<int64_t, 1>();
      std::vector<int64_t> axes;
      for (int64_t i = 0; i < inputTensorValues[1].sizes()[0]; ++i) {
        // ONNX unsqueeze accepts negative axes
        // From https://pytorch.org/docs/stable/generated/torch.unsqueeze.html
        // Negative dim will correspond to unsqueeze() applied at dim = dim +
        // input.dim() + 1.
        axes_a[i] +=
            axes_a[i] < 0 ? inputTensorValues[0].sizes().size() + 1 : 0;
        axes.push_back(axes_a[i]);
      }
      std::sort(axes.begin(), axes.end());
```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `sizes`, `size`, `push_back`, `sort`, `begin`, `end`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`sizes`, `size`, `push_back`, `sort`, `begin`, `end`。

### Lines 326-334
```cpp
      updated_val = inputTensorValues[0];
      for (int64_t i = 0; i < inputTensorValues[1].sizes()[0]; ++i) {
        updated_val = at::unsqueeze(updated_val, axes[i]);
      }
      return std::optional<at::Tensor>(updated_val);
    } else if (opset_version >= ONNX_OPSET_9) {
      assert(inputTensorValues.size() == 1);
      if (!node->hasAttributeS("axes")) {
        return std::nullopt;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `sizes`, `unsqueeze`, `assert`, `size`, `hasAttributeS`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`sizes`, `unsqueeze`, `assert`, `size`, `hasAttributeS`。

### Lines 335-347
```cpp
      }
      updated_val = inputTensorValues[0];
      std::vector<int64_t> axesAttr = node->is(attr::axes);
      std::sort(axesAttr.begin(), axesAttr.end());
      for (auto axis : axesAttr) {
        updated_val = at::unsqueeze(updated_val, axis);
      }
      return std::optional<at::Tensor>(updated_val);
    } else {
      TORCH_WARN(
          "Constant folding - unsupported opset version. "
          "Constant folding not applied.");
      return std::nullopt;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `is`, `sort`, `begin`, `end`, `unsqueeze`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`is`, `sort`, `begin`, `end`, `unsqueeze`。

### Lines 348-357
```cpp
    }
  } else if (node->kind() == onnx::Squeeze) {
    assert(inputTensorValues.size() == 2 || inputTensorValues.size() == 1);
    if (opset_version >= ONNX_OPSET_13) {
      // Squeeze version 13 input axes is optional, inputTensorValues.size() ==
      // 1 means axes equal to None
      updated_val = inputTensorValues[0];
      if (inputTensorValues.size() == 2) {
        // Checking validity of 'axes' input
        if (inputTensorValues[1].sizes().size() != 1) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `kind`, `assert`, `size`, `sizes`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`kind`, `assert`, `size`, `sizes`。

### Lines 358-370
```cpp
          TORCH_WARN(
              "Constant folding - Invalid 'axes' inputs found for opset 13 onnx::Squeeze op. "
              "Constant folding not applied.");
          return std::nullopt;
        }
        auto axes_a = inputTensorValues[1].accessor<int64_t, 1>();
        std::vector<int64_t> axes;
        for (int64_t i = 0; i < inputTensorValues[1].sizes()[0]; ++i) {
          // ONNX Squeeze accepts negative axes
          axes_a[i] += axes_a[i] < 0 ? inputTensorValues[0].sizes().size() : 0;
          axes.push_back(axes_a[i]);
        }
        std::sort(axes.begin(), axes.end());
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `sizes`, `size`, `push_back`, `sort`, `begin`, `end`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`sizes`, `size`, `push_back`, `sort`, `begin`, `end`。

### Lines 371-379
```cpp
        for (int64_t i = 0; i < inputTensorValues[1].sizes()[0]; ++i) {
          updated_val = at::squeeze(updated_val, axes[i]);
        }
      }
      return std::optional<at::Tensor>(updated_val);
    } else if (opset_version >= ONNX_OPSET_9) {
      assert(inputTensorValues.size() == 1);
      updated_val = inputTensorValues[0];
      if (node->hasAttributeS("axes")) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `sizes`, `squeeze`, `assert`, `size`, `hasAttributeS`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`sizes`, `squeeze`, `assert`, `size`, `hasAttributeS`。

### Lines 380-391
```cpp
        std::vector<int64_t> axesAttr = node->is(attr::axes);
        std::sort(axesAttr.begin(), axesAttr.end());
        for (auto axis : axesAttr) {
          updated_val = at::squeeze(updated_val, axis);
        }
      }
      return std::optional<at::Tensor>(updated_val);
    } else {
      TORCH_WARN(
          "Constant folding - unsupported opset version. "
          "Constant folding not applied.");
      return std::nullopt;
```
- EN: This block iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `is`, `sort`, `begin`, `end`, `squeeze`.
- CN: 该代码块遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`is`, `sort`, `begin`, `end`, `squeeze`。

### Lines 392-402
```cpp
    }
  } else if (node->kind() == onnx::Transpose) {
    assert(inputTensorValues.size() == 1);
    if (!node->hasAttributeS("perm")) {
      return std::nullopt;
    }
    updated_val = inputTensorValues[0].permute(node->is(attr::perm));
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Cast) {
    assert(inputTensorValues.size() == 1);
    if (node->hasAttributeS("to") && ONNXTypeToATenType(node->i(attr::to))) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `assert`, `size`, `hasAttributeS`, `permute`, `is`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `assert`, `size`, `hasAttributeS`, `permute`, `is`, `...`。

### Lines 403-411
```cpp
      updated_val = inputTensorValues[0].to(
          ONNXTypeToATenType(node->i(attr::to)).value());
      return std::optional<at::Tensor>(updated_val);
    }
    return std::nullopt;
  } else if (node->kind() == onnx::Reshape) {
    assert(inputTensorValues.size() == 2);
    updated_val = inputTensorValues[0];
    std::vector<int64_t> shape(inputTensorValues[1].sizes()[0], 0);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `to`, `ONNXTypeToATenType`, `i`, `value`, `kind`, `assert`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`to`, `ONNXTypeToATenType`, `i`, `value`, `kind`, `assert`, `...`。

### Lines 412-425
```cpp
    auto shape_a = inputTensorValues[1].accessor<int64_t, 1>();
    assert(inputTensorValues[1].sizes()[0] >= 0);
    // Set value of allowzero
    int64_t allowzero = 0;
    if (node->hasAttributeS("allowzero")) {
      allowzero = node->i(attr::allowzero);
    }
    for (size_t i = 0; i < (size_t)(inputTensorValues[1].sizes()[0]); ++i) {
      // All shape dim values should be >= -1
      // onnx::Reshape supports a shape dim value to be zero, in
      // which case the actual dim value remains unchanged. However,
      // at::reshape does not support shape dim value to be zero
      assert(shape_a[i] >= -1);
      if (shape_a[i] == 0 && !allowzero) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `assert`, `sizes`, `hasAttributeS`, `i`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`assert`, `sizes`, `hasAttributeS`, `i`。

### Lines 426-435
```cpp
        if (i >= inputTensorValues[0].sizes().size()) {
          throw std::runtime_error(
              "Dimension with value 0 exceeds the input size dimensions.");
        }
        shape[i] = inputTensorValues[0].sizes()[i];
      } else {
        shape[i] = shape_a[i];
      }
    }
    return std::optional<at::Tensor>(at::reshape(updated_val, shape));
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `sizes`, `size`, `runtime_error`, `reshape`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`sizes`, `size`, `runtime_error`, `reshape`。

### Lines 436-445
```cpp
  } else if (node->kind() == onnx::Shape) {
    TORCH_INTERNAL_ASSERT(inputTensorValues.size() == 1);
    updated_val = at::_shape_as_tensor(inputTensorValues[0]);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::ReduceL1 || node->kind() == onnx::ReduceL2) {
    assert(inputTensorValues.size() == 1);
    if (!node->hasAttributeS("axes")) {
      return std::nullopt;
    }
    if (!node->hasAttributeS("keepdims")) {
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `size`, `_shape_as_tensor`, `assert`, `hasAttributeS`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `size`, `_shape_as_tensor`, `assert`, `hasAttributeS`。

### Lines 446-454
```cpp
      return std::nullopt;
    }
    int p = node->kind() == onnx::ReduceL1 ? 1 : 2;
    updated_val = at::norm(
        inputTensorValues[0], p, node->is(attr::axes), node->i(attr::keepdims));
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::ReduceProd) {
    int64_t rank = inputTensorValues[0].sizes().size();
    std::vector<int64_t> axes;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `norm`, `is`, `i`, `sizes`, `size`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `norm`, `is`, `i`, `sizes`, `size`。

### Lines 455-464
```cpp
    if (!node->hasAttributeS("axes")) {
      axes = std::vector<int64_t>(rank);
      std::iota(axes.rbegin(), axes.rend(), 0);
    } else {
      for (const auto& axis : node->is(attr::axes)) {
        axes.emplace_back(axis < 0 ? axis + rank : axis);
      }
      std::sort(axes.begin(), axes.end(), std::greater<>());
    }

```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `hasAttributeS`, `iota`, `rbegin`, `rend`, `is`, `emplace_back`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`hasAttributeS`, `iota`, `rbegin`, `rend`, `is`, `emplace_back`, `...`。

### Lines 465-476
```cpp
    bool keepdims =
        node->hasAttributeS("keepdims") ? node->i(attr::keepdims) : true;
    updated_val = inputTensorValues[0];
    for (const auto& axis : axes) {
      updated_val = at::prod(updated_val, axis, keepdims);
    }
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Gather) {
    assert(inputTensorValues.size() == 2);
    // default axis = 0
    int64_t axis = 0;
    if (node->hasAttributeS("axis")) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `hasAttributeS`, `i`, `prod`, `kind`, `assert`, `size`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`hasAttributeS`, `i`, `prod`, `kind`, `assert`, `size`。

### Lines 477-486
```cpp
      axis = node->i(attr::axis);
    }
    // If axis attribute for onnx::Gather has a value less than 0,
    // It needs to be adjusted (+= dim sizes) for aten op
    axis += axis < 0 ? inputTensorValues[0].sizes().size() : 0;
    at::Tensor indices = inputTensorValues[1];
    auto q = indices.dim();
    // at::index_select only supports indices with rank <= 1.
    // See https://pytorch.org/docs/main/generated/torch.index_select.html
    if (q > 1) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects. Key symbols: `i`, `sizes`, `size`, `dim`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象。关键符号：`i`, `sizes`, `size`, `dim`。

### Lines 487-496
```cpp
      return std::nullopt;
    }
    // If the device of indices tensor is not the same with it of the input
    // tensor, move it to the device of the input tensor
    if (inputTensorValues[0].device() != indices.device()) {
      indices = indices.to(inputTensorValues[0].device());
    }
    // If indices input for onnx::Gather has a value less than 0,
    // It needs to be adjusted (+= dim value) for aten op
    auto less_mask = at::lt(indices, 0);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `device`, `to`, `lt`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`device`, `to`, `lt`。

### Lines 497-505
```cpp
    auto indices_corr = at::add(indices, inputTensorValues[0].sizes()[axis]);
    auto indices_masked = at::where(less_mask, indices_corr, indices);
    updated_val = at::index_select(inputTensorValues[0], axis, indices_masked);
    // If rank of indices is 0, rank of output tensor should be
    // rank_of_input - 1.
    if (q < 1) {
      updated_val = updated_val.squeeze(axis);
    }
    return std::optional<at::Tensor>(updated_val);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `add`, `sizes`, `where`, `index_select`, `squeeze`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`add`, `sizes`, `where`, `index_select`, `squeeze`。

### Lines 506-515
```cpp
  } else if (node->kind() == onnx::Range) {
    updated_val = runTorchArange_opset11(node, inputTensorValues);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Where) {
    updated_val = at::where(
        inputTensorValues[0], inputTensorValues[1], inputTensorValues[2]);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Equal) {
    updated_val = at::eq(inputTensorValues[0], inputTensorValues[1]);
    return std::optional<at::Tensor>(updated_val);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `runTorchArange_opset11`, `where`, `eq`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `runTorchArange_opset11`, `where`, `eq`。

### Lines 516-524
```cpp
  } else if (node->kind() == onnx::Greater) {
    updated_val = at::greater(inputTensorValues[0], inputTensorValues[1]);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Less) {
    updated_val = at::less(inputTensorValues[0], inputTensorValues[1]);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Neg) {
    updated_val = at::neg(inputTensorValues[0]);
    return std::optional<at::Tensor>(updated_val);
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `greater`, `less`, `neg`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `greater`, `less`, `neg`。

### Lines 525-535
```cpp
  } else if (node->kind() == onnx::Not) {
    auto ones =
        at::ones(inputTensorValues[0].sizes(), inputTensorValues[0].dtype());
    updated_val = at::ne(inputTensorValues[0], ones);
    return std::optional<at::Tensor>(updated_val);
  } else if (node->kind() == onnx::Size) {
    int64_t total_size = 1;
    for (auto size : inputTensorValues[0].sizes()) {
      total_size *= size;
    }
    return std::optional<at::Tensor>(IntToTensor(total_size));
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `ones`, `sizes`, `dtype`, `ne`, `IntToTensor`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `ones`, `sizes`, `dtype`, `ne`, `IntToTensor`。

### Lines 536-544
```cpp
  } else if (node->kind() == onnx::Softmax) {
    int64_t axis = node->hasAttributeS("axis") ? node->i(attr::axis) : -1;
    updated_val = at::softmax(inputTensorValues[0], axis);
    return std::optional<at::Tensor>(updated_val);
  } else {
    return std::nullopt;
  }
}

```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `kind`, `hasAttributeS`, `i`, `softmax`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`kind`, `hasAttributeS`, `i`, `softmax`。

### Lines 545-555
```cpp
static bool isConstant(Value* val, const ValueToParamPairMap& valsToParamsMap) {
  auto parentNode = val->node();
  return (parentNode->kind() == prim::Param &&
          valsToParamsMap.find(val) !=
              valsToParamsMap
                  .end()) || // Checks val is a parameter and not a real input
      (parentNode->kind() == onnx::Constant && !parentNode->mustBeNone() &&
       parentNode->kindOf(attr::value) ==
           AttributeKind::t); // Check other types?
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `isConstant`, `node`, `kind`, `find`, `end`, `mustBeNone`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`isConstant`, `node`, `kind`, `find`, `end`, `mustBeNone`, `...`。

### Lines 556-564
```cpp
static bool hasParamInput(Node* n, const ValueToParamPairMap& valsToParamsMap) {
  for (auto input : n->inputs()) {
    if (valsToParamsMap.find(input) != valsToParamsMap.end()) {
      return true;
    }
  }
  return false;
}

```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects. Key symbols: `hasParamInput`, `inputs`, `find`, `end`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象。关键符号：`hasParamInput`, `inputs`, `find`, `end`。

### Lines 565-573
```cpp
static std::vector<at::Tensor> getValues(
    Node* node,
    const ValueToParamPairMap& valsToParamsMap) {
  size_t numInputs = node->inputs().size();
  std::vector<at::Tensor> inputTensorValues;
  inputTensorValues.reserve(numInputs);
  for (auto val : node->inputs()) {
    if (val->node()->kind() == prim::Param) {
      auto itr = valsToParamsMap.find(val);
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects. Key symbols: `getValues`, `inputs`, `size`, `reserve`, `node`, `kind`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象。关键符号：`getValues`, `inputs`, `size`, `reserve`, `node`, `kind`, `...`。

### Lines 574-587
```cpp
      if (itr == valsToParamsMap.end()) {
        throw std::runtime_error(
            "getValues: Input value not found amongst constant parameters.");
      }
      inputTensorValues.push_back(itr->second.second.toTensor());
    } else if (val->node()->kind() == onnx::Constant) {
      inputTensorValues.push_back(val->node()->t(attr::value));
    } else {
      throw std::runtime_error(
          "getValues: Unsupported kind of constant node found.");
    }
  }
  TORCH_INTERNAL_ASSERT(inputTensorValues.size() == numInputs);
  return inputTensorValues;
```
- EN: This block handles conditional branches; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `end`, `runtime_error`, `push_back`, `toTensor`, `node`, `kind`, `...`.
- CN: 该代码块处理条件分支；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`end`, `runtime_error`, `push_back`, `toTensor`, `node`, `kind`, `...`。

### Lines 588-598
```cpp
}

static bool areNodeInputsConstant(
    Node* node,
    const ValueToParamPairMap& valsToParamsMap) {
  return std::all_of(
      node->inputs().begin(),
      node->inputs().end(),
      [&valsToParamsMap](Value* v) { return isConstant(v, valsToParamsMap); });
}

```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `areNodeInputsConstant`, `all_of`, `inputs`, `begin`, `end`, `isConstant`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`areNodeInputsConstant`, `all_of`, `inputs`, `begin`, `end`, `isConstant`。

### Lines 599-609
```cpp
static std::vector<Node*> getOnnxConstParentsToRemove(Node* node) {
  std::vector<Node*> parentNodes;
  for (auto val : node->inputs()) {
    // If the parent of 'node' is an onnx::Constant node,
    // and 'node' is the only downstream node it serves (this
    // is important), then push it in the list to remove.
    if (val->node()->kind() == onnx::Constant && val->uses().size() == 1) {
      parentNodes.push_back(val->node());
    }
  }
  return parentNodes;
```
- EN: This block handles conditional branches; iterates over collections or graph structures; produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getOnnxConstParentsToRemove`, `inputs`, `node`, `kind`, `uses`, `size`, `...`.
- CN: 该代码块处理条件分支；遍历集合或图结构；返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getOnnxConstParentsToRemove`, `inputs`, `node`, `kind`, `uses`, `size`, `...`。

### Lines 610-622
```cpp
}

} // namespace onnx_constant_fold

// This method updates the block in-place to fold all the one-time
// constant-based computations/ops into an initializer node.
//
// NB: This is not constant folding in the traditional sense, as we
// don't try particularly hard to evaluate operations on constant nodes.
// This is more of a partial evaluation analysis, where operations on constant
// nodes can be lifted so we run them earlier, before the usual parameters are
// known.
static void ConstantFoldONNX(
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `ConstantFoldONNX`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`ConstantFoldONNX`。

### Lines 623-633
```cpp
    Block* b,
    ParamMap& paramsDict,
    int opset_version) {
  if (opset_version < ONNX_OPSET_9) {
    TORCH_WARN(
        "Constant folding supported for only opsets >= 9. "
        "Constant folding not applied.");
    return;
  }
  TORCH_INTERNAL_ASSERT(b->param_node());
  auto valsToParamsMap = buildValueToParamsMap(b, paramsDict);
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `param_node`, `buildValueToParamsMap`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`param_node`, `buildValueToParamsMap`。

### Lines 634-642
```cpp
  // Only the root block is constant-folded. Folding nested blocks is
  // not supported for now.
  for (auto it = b->nodes().begin(), end = b->nodes().end(); it != end; ++it) {
    auto node = *it;
    if (node->outputs().size() > 1) {
      // Constant folding for multiple-output nodes not supported. Skip it.
      continue;
    }
    if (!onnx_constant_fold::areNodeInputsConstant(node, valsToParamsMap)) {
```
- EN: This block handles conditional branches; iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `nodes`, `begin`, `end`, `outputs`, `size`, `areNodeInputsConstant`.
- CN: 该代码块处理条件分支；遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`nodes`, `begin`, `end`, `outputs`, `size`, `areNodeInputsConstant`。

### Lines 643-655
```cpp
      // If all the inputs to this node are not either parameter or
      // onnx::Constant, then skip this node.
      continue;
    }

    auto inputTensorValues =
        onnx_constant_fold::getValues(node, valsToParamsMap);
    if (inputTensorValues.empty()) {
      // This is a terminal node with no inputs, such as onnx::Constant. Skip
      // it.
      continue;
    }
    auto updatedValWrapped = onnx_constant_fold::runTorchBackendForOnnx(
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `getValues`, `empty`, `runTorchBackendForOnnx`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`getValues`, `empty`, `runTorchBackendForOnnx`。

### Lines 656-664
```cpp
        node, inputTensorValues, opset_version);
    if (updatedValWrapped == std::nullopt) {
      // Constant folding is not supported for this op. Skip it.
      continue;
    }

    at::Tensor updatedVal = *updatedValWrapped;
    auto newSourceNodeOutput = [&]() -> Value* {
      if (onnx_constant_fold::hasParamInput(node, valsToParamsMap)) {
```
- EN: This block handles conditional branches; manipulates TorchScript IR objects; performs optimization-oriented rewriting. Key symbols: `hasParamInput`.
- CN: 该代码块处理条件分支；操作 TorchScript IR 对象；执行面向优化的改写。关键符号：`hasParamInput`。

### Lines 665-673
```cpp
        // Create a new input to the block (prim::Param node output). Add a
        // corresponding entry in valToParamMap. Replace the downstream inputs
        // with this value, and disconnect all the input values of the folded
        // node.
        auto newSourceNodeOutput = b->addInput();
        valsToParamsMap.insert(
            {newSourceNodeOutput,
             std::make_pair(newSourceNodeOutput->debugName(), updatedVal)});
        return newSourceNodeOutput;
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `addInput`, `insert`, `make_pair`, `debugName`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`addInput`, `insert`, `make_pair`, `debugName`。

### Lines 674-690
```cpp
      } else {
        auto newSourceNode =
            createONNXConstant(node->owningGraph(), node, updatedVal);
        newSourceNode->copyMetadata(node);
        return newSourceNode->output();
      }
    }();
    newSourceNodeOutput->inferTypeFrom(updatedVal);
    node->outputs().at(0)->replaceAllUsesWith(newSourceNodeOutput);
    // Next we remove the current node that has been replaced by
    // an initializer. But before we start de-wiring this node,
    // we check if any parents of this nodes were onnx::Constant
    // and remove them first, and then remove the current node.
    // If the parent was an initializer (not onnx::Constant) then
    // they are all removed by the eraseUnusedBlockInputs() call
    // (below) outside the loop.
    auto onnxConstParents =
```
- EN: This block produces a result or forwards a computed value; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `createONNXConstant`, `owningGraph`, `copyMetadata`, `output`, `inferTypeFrom`, `outputs`, `...`.
- CN: 该代码块返回结果或转发已计算的值；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`createONNXConstant`, `owningGraph`, `copyMetadata`, `output`, `inferTypeFrom`, `outputs`, `...`。

### Lines 691-703
```cpp
        onnx_constant_fold::getOnnxConstParentsToRemove(node);
    node->removeAllInputs();
    for (auto* n : onnxConstParents) {
      n->destroy();
    }
    it.destroyCurrent();
  }
  eraseUnusedValuesFromMap(valsToParamsMap);
  eraseUnusedBlockInputs(b);
  buildParamsMapFromValueToParamsMap(valsToParamsMap, paramsDict);
  return;
}

```
- EN: This block iterates over collections or graph structures; manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `getOnnxConstParentsToRemove`, `removeAllInputs`, `destroy`, `destroyCurrent`, `eraseUnusedValuesFromMap`, `eraseUnusedBlockInputs`, `...`.
- CN: 该代码块遍历集合或图结构；操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`getOnnxConstParentsToRemove`, `removeAllInputs`, `destroy`, `destroyCurrent`, `eraseUnusedValuesFromMap`, `eraseUnusedBlockInputs`, `...`。

### Lines 704-712
```cpp
void ConstantFoldONNX(
    std::shared_ptr<Graph>& g,
    ParamMap& paramsDict,
    int opset_version) {
  ConstantFoldONNX(g->block(), paramsDict, opset_version);
  GRAPH_DUMP("After ConstantFoldONNX:", g);
}

} // namespace torch::jit
```
- EN: This block manipulates TorchScript IR objects; performs optimization-oriented rewriting; protects shared state or ordering assumptions. Key symbols: `ConstantFoldONNX`, `block`.
- CN: 该代码块操作 TorchScript IR 对象；执行面向优化的改写；保护共享状态或执行顺序假设。关键符号：`ConstantFoldONNX`, `block`。


## Key Concepts / 关键概念
- EN: IR graph manipulation — the file inspects or rewrites TorchScript graphs and nodes.
  CN: IR 图操作——该文件会检查或改写 TorchScript 的图、节点与块。
- EN: Optimization pass — the code simplifies graphs by removing redundant work or folding values.
  CN: 优化 pass——代码通过删除冗余计算或常量折叠来简化图。
- EN: Type and shape reasoning — the code propagates static information across the IR.
  CN: 类型与形状推理——代码会在 IR 中传播静态信息。
- EN: Export interop — the implementation helps bridge TorchScript graphs to ONNX-style representations.
  CN: 导出互操作——实现帮助 TorchScript 图与 ONNX 风格表示之间建立桥接。
- EN: Subsystem integration — the implementation collaborates with nearby JIT/frontend headers and runtime helpers.
  CN: 子系统集成——实现会与附近的 JIT/前端头文件以及运行时辅助模块协同工作。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/onnx/constant_fold.h`, `torch/csrc/jit/passes/onnx/helper.h`, `ATen/Functions.h`, `c10/util/Exception.h`, `c10/util/irange.h`
- External includes / 外部头文件: `algorithm`, `optional`
- Namespaces / 命名空间: `torch::jit`, `onnx`, `onnx_constant_fold`
- Representative symbols / 代表性符号: `namespace`, `OnnxType`, `handleNegativeStartEndIndex`, `runTorchSlice_opset9`, `assert`, `size`, `hasAttributeS`, `is`, `resize`, `iota`, `...`
