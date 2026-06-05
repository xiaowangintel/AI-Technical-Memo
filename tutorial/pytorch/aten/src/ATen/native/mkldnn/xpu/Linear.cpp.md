# Linear.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/Linear.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on linear; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 linear；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <ATen/DeviceGuard.h>
#include <torch/library.h>

#include <FusionUtils.h>

namespace at::native::xpu {

std::tuple<std::vector<int64_t>, std::vector<int64_t>, std::vector<int64_t>>
collapse_in_out_dim(at::Tensor input, int64_t dim, at::Tensor weight) {
  // dim collapse, e.g. [B, M, K] -> [BM, K]
  std::vector<int64_t> input_reshaped_size = (dim == 2)
      ? std::vector<int64_t>(input.size(0), input.size(1))
      : std::vector<int64_t>{
            input.numel() / (input.size(input.dim() - 1)),
            input.size(input.dim() - 1)};
  // [B, M, K] -> [B, M]
  std::vector<int64_t> output_size(
      input.sizes().begin(), input.sizes().end() - 1);
  // [B, M, N]
  output_size.push_back(weight.size(0));

  // [BM, N]
  std::vector<int64_t> output_reshaped_size{
      input_reshaped_size[0], weight.size(0)};
  return {
      std::move(input_reshaped_size),
      std::move(output_size),
      std::move(output_reshaped_size)};
}
```
- EN: Lines 1-30 pull in 3 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are collapse_in_out_dim, size, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 3 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 collapse_in_out_dim, size，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
Tensor linear_pointwise(
    const Tensor& input_t, // [M, K] or [B, M, K]
    const Tensor& weight_t, // [N, K]
    const std::optional<Tensor>& bias_opt,
    std::string_view attr,
    torch::List<std::optional<at::Scalar>> scalars,
    std::optional<std::string_view> algorithm) {
  onednn::Attr att;
  const OptionalDeviceGuard device_guard(device_of(input_t));
  att = construct_unary_attr(att, attr, scalars, algorithm);
  auto input = input_t.contiguous();

  const int64_t dim = input.dim();

  auto [input_reshaped_size, output_size, output_reshaped_size] =
      collapse_in_out_dim(input, dim, weight_t);
  Tensor output = at::empty(output_size, input.options());
  Tensor input_reshaped = input;
  if (dim != 2) {
    output = output.reshape(output_reshaped_size);
    input_reshaped = input_reshaped.reshape(input_reshaped_size);
  }

  auto bias = bias_opt.has_value() ? bias_opt.value() : at::Tensor();
  at::native::onednn::matmul(
      output, input_reshaped, weight_t, bias, /*m2_trans*/ false, att);

  if (dim != 2) {
    output = output.reshape(output_size);
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp

  return output;
}

Tensor linear_pointwise_binary(
    const Tensor& input_t,
    const Tensor& other_t,
    const Tensor& weight_t,
    const std::optional<Tensor>& bias_opt,
    std::string_view binary_attr) {
  const OptionalDeviceGuard device_guard(device_of(input_t));
  onednn::Attr attr;
  attr = construct_binary_attr<true>(attr, binary_attr, other_t);
  auto input = input_t.contiguous();

  const int64_t dim = input.dim();

  // dim collapse
  auto [input_reshaped_size, output_size, output_reshaped_size] =
      collapse_in_out_dim(input, dim, weight_t);
  Tensor output = at::empty(output_size, input.options());
  Tensor input_reshaped = input;

  if (dim != 2) {
    // input [m, k], weight [n, k], output [m, n]
    output = output.reshape(output_reshaped_size);
    input_reshaped = input_reshaped.reshape(input_reshaped_size);
  } else {
    TORCH_CHECK(
        output.dim() == other_t.dim(),
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 91-113
```cpp
        "linear_binary_run expects the dimension of output and other tensor to be the same");
  }

  auto bias = bias_opt.has_value() ? bias_opt.value() : at::Tensor();
  at::native::onednn::matmul(
      output, input_reshaped, weight_t, bias, /*m2_trans*/ false, attr);

  if (dim != 2) {
    output = output.reshape(output_size);
  }
  return output;
}

TORCH_LIBRARY_IMPL(mkldnn, XPU, m) {
  m.impl(
      TORCH_SELECTIVE_NAME("mkldnn::_linear_pointwise"),
      TORCH_FN(linear_pointwise));
  m.impl(
      TORCH_SELECTIVE_NAME("mkldnn::_linear_pointwise.binary"),
      TORCH_FN(linear_pointwise_binary));
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
- EN: Notable symbols: collapse_in_out_dim, linear_pointwise, linear_pointwise_binary, TORCH_LIBRARY_IMPL.
- CN: 重要符号：collapse_in_out_dim, linear_pointwise, linear_pointwise_binary, TORCH_LIBRARY_IMPL。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/DeviceGuard.h, torch/library.h`.
- CN: 主要内部头文件：`ATen/DeviceGuard.h, torch/library.h`。
- EN: External/system headers: `FusionUtils.h`.
- CN: 外部/系统头文件：`FusionUtils.h`。
- EN: The implementation revolves around symbols such as `collapse_in_out_dim, linear_pointwise, linear_pointwise_binary, TORCH_LIBRARY_IMPL`.
- CN: 实现围绕 `collapse_in_out_dim, linear_pointwise, linear_pointwise_binary, TORCH_LIBRARY_IMPL` 等符号展开。
