# FusionUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/FusionUtils.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on fusion utils; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 fusion utils；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <ATen/native/mkldnn/xpu/FusionUtils.h>

using namespace at::native::onednn;

namespace at::native::xpu {

onednn::Attr& handle_argument_less(std::string_view unary, onednn::Attr& attr) {
  static const std::unordered_map<
      std::string_view,
      std::function<onednn::Attr&(onednn::Attr&)>>
      unary_map = {
          {"relu",
           [](onednn::Attr& attr) -> onednn::Attr& {
             return attr.append_post_eltwise(
                 1.0f, 0.0f, 0.0f, attr.kind_with_relu);
           }},
          {"sigmoid",
           [](onednn::Attr& attr) -> onednn::Attr& {
             return attr.append_post_eltwise(
                 1.0f, 0.0f, 0.0f, attr.kind_with_sigmoid);
           }},
          {"tanh",
           [](onednn::Attr& attr) -> onednn::Attr& {
             return attr.append_post_eltwise(
                 1.0f, 0.0f, 0.0f, attr.kind_with_tanh);
           }},
          {"hardswish",
           [](onednn::Attr& attr) -> onednn::Attr& {
             return attr.append_post_eltwise(
                 1.0f, 1.0f / 6.0f, 1.0f / 2.0f, attr.kind_with_hardswish);
```
- EN: Lines 1-30 pull in 1 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are handle_argument_less, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 1 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 handle_argument_less，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
           }},
          {"swish",
           [](onednn::Attr& attr) -> onednn::Attr& {
             return attr.append_post_eltwise(
                 1.0f, 1.0f, 0.0f, attr.kind_with_swish);
           }},
          {"hardsigmoid",
           [](onednn::Attr& attr) -> onednn::Attr& {
             return attr.append_post_eltwise(
                 1.0f, 1.0f / 6.0f, 1.0f / 2.0f, attr.kind_with_hardsigmoid);
           }},
          {"none", [](onednn::Attr& attr) -> onednn::Attr& { return attr; }}};

  if (unary_map.find(unary) != unary_map.end()) {
    return unary_map.at(unary)(attr);
  }
  TORCH_CHECK(
      false,
      "Unary attr ",
      unary,
      " is not supported for conv/linear post unary fusion");
}

onednn::Attr& handle_need_sclars(
    std::string_view unary,
    onednn::Attr& attr,
    torch::List<std::optional<at::Scalar>> scalars) {
  static const std::unordered_map<
      std::string_view,
      std::function<onednn::Attr&(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
          onednn::Attr&, torch::List<std::optional<at::Scalar>>)>>
      unary_map = {
          {"leaky_relu",
           [](onednn::Attr& attr,
              torch::List<std::optional<at::Scalar>> scalars) -> onednn::Attr& {
             auto alpha =
                 scalars[0].get().toOptional<at::Scalar>().value().to<float>();
             return attr.append_post_eltwise(
                 1.0f, alpha, 0.f, attr.kind_with_relu);
           }},
          {"hardtanh",
           [](onednn::Attr& attr,
              torch::List<std::optional<at::Scalar>> scalars) -> onednn::Attr& {
             auto alpha =
                 scalars[0].get().toOptional<at::Scalar>().value().to<float>();
             auto beta =
                 scalars[1].get().toOptional<at::Scalar>().value().to<float>();
             return attr.append_post_eltwise(
                 1.0f, alpha, beta, attr.kind_with_clip);
           }}};

  if (unary_map.find(unary) != unary_map.end()) {
    return unary_map.at(unary)(attr, scalars);
  }
  TORCH_CHECK(
      false,
      "Unary attr ",
      unary,
      " is not supported for conv/linear post unary fusion");
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 91-120
```cpp

onednn::Attr& handle_need_algorithm(
    std::string_view unary,
    onednn::Attr& attr,
    std::optional<std::string_view> algorithm) {
  TORCH_CHECK(
      unary == "gelu",
      "GELU is the only unary operation that requires an algorithm currently");
  if (!algorithm.has_value()) {
    TORCH_CHECK(
        false,
        "GELU algorithm is not specified, please specify it as 'none' or 'tanh'");
  }
  enum dnnl::algorithm gelu_type;
  if (algorithm.value() == "none") {
    gelu_type = attr.kind_with_gelu_erf;
  } else {
    gelu_type = attr.kind_with_gelu_tanh;
  }
  return attr.append_post_eltwise(1.0f, 0.0f, 0.0f, gelu_type);
}

onednn::Attr& construct_unary_attr(
    onednn::Attr& attr,
    std::string_view unary,
    torch::List<std::optional<at::Scalar>> scalars,
    std::optional<std::string_view> algorithm) {
  // Define sets for unary operations based on their argument requirements.
  // Category `argument_less`: stateless operations
  // Category `need_scalars`: require alpha/beta
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-145
```cpp
  // Category `need_algorithm`: require algorithm specification, only gelu now.
  // If further unary operations required, they can be added to these sets or
  // add new sets according to their new categories.
  static const std::set<std::string_view> argument_less = {
      "none", "relu", "sigmoid", "tanh", "hardswish", "swish", "hardsigmoid"};
  static const std::set<std::string_view> need_scalars = {
      "leaky_relu", "hardtanh"};
  static const std::set<std::string_view> need_algorithm = {"gelu"};

  if (argument_less.find(unary) != argument_less.end()) {
    return handle_argument_less(unary, attr);
  } else if (need_scalars.find(unary) != need_scalars.end()) {
    return handle_need_sclars(unary, attr, scalars);
  } else if (need_algorithm.find(unary) != need_algorithm.end()) {
    return handle_need_algorithm(unary, attr, algorithm);
  } else {
    TORCH_CHECK(
        false,
        "Unary attr ",
        unary,
        " is not supported for conv/linear post unary fusion");
  }
}

} // namespace at::native::xpu
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Backend focus: utility layer.
- CN: 后端重点：工具层。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: handle_argument_less, handle_need_sclars, handle_need_algorithm, construct_unary_attr.
- CN: 重要符号：handle_argument_less, handle_need_sclars, handle_need_algorithm, construct_unary_attr。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/mkldnn/xpu/FusionUtils.h`.
- CN: 主要内部头文件：`ATen/native/mkldnn/xpu/FusionUtils.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `handle_argument_less, handle_need_sclars, handle_need_algorithm, construct_unary_attr`.
- CN: 实现围绕 `handle_argument_less, handle_need_sclars, handle_need_algorithm, construct_unary_attr` 等符号展开。
