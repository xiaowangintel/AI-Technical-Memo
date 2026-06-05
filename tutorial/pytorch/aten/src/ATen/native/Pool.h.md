# Pool.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/Pool.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on pool; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 pool；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <ATen/core/Tensor.h>
#include <ATen/div_rtn.h>
#include <ATen/TensorUtils.h>
#include <ATen/native/DispatchStub.h>
#include <c10/util/irange.h>

#include <utility>

#pragma once

namespace at::native {

using max_pool2d_fn = void(*)(const Tensor& output, const Tensor& indices, const Tensor& input,
    int kW, int kH, int dW, int dH, int padW, int padH, int dilationW, int dilationH);
using max_pool2d_backward_fn = void(*)(const Tensor& grad_input, const Tensor& grad_output, const Tensor& indices);

DECLARE_DISPATCH(max_pool2d_fn, max_pool2d_kernel)
DECLARE_DISPATCH(max_pool2d_backward_fn, max_pool2d_backward_kernel)

// average pooling has same signature for forward and backward
using avg_pool2d_fn = void(*)(const Tensor& output, const Tensor& input, int64_t kW, int64_t kH,
    int64_t dW, int64_t dH, int64_t padW, int64_t padH, bool count_include_pad, std::optional<int64_t> divisor_override);
using avg_pool2d_backward_fn = void(*)(const Tensor& output, const Tensor& input, int kW, int kH,
    int dW, int dH, int padW, int padH, bool count_include_pad, std::optional<int64_t> divisor_override);

DECLARE_DISPATCH(avg_pool2d_fn, avg_pool2d_kernel)
DECLARE_DISPATCH(avg_pool2d_backward_fn, avg_pool2d_backward_kernel)

// average pooling has same signature for forward and backward
using avg_pool3d_fn = void(*)(const Tensor& output, const Tensor& input,
    int64_t kW, int64_t kH, int64_t kD, int64_t dW, int64_t dH, int64_t dD,
    int64_t padW, int64_t padH, int64_t padD, bool count_include_pad,
    std::optional<int64_t> divisor_override);
using avg_pool3d_backward_fn = void(*)(const Tensor& output, const Tensor& input,
    int kW, int kH, int kD, int dW, int dH, int dD,
    int padW, int padH, int padD, bool count_include_pad,
    std::optional<int64_t> divisor_override);

DECLARE_DISPATCH(avg_pool3d_fn, avg_pool3d_kernel)
DECLARE_DISPATCH(avg_pool3d_backward_fn, avg_pool3d_backward_kernel)

using max_pool3d_fn = void(*)(Tensor& output, Tensor& indices, const Tensor& input,
    int kW, int kH, int kD, int dW, int dH, int dD, int pW, int pH, int pD, int dilationW, int dilationH, int dilationD);
using max_pool3d_backward_fn = void(*)(Tensor& grad_input, const Tensor& grad_output, const Tensor& indices);

DECLARE_DISPATCH(max_pool3d_fn, max_pool3d_kernel)
DECLARE_DISPATCH(max_pool3d_backward_fn, max_pool3d_backward_kernel)
namespace {

template <typename dest_t, typename src_t>
inline dest_t
safe_downcast(src_t v)
{
  TORCH_CHECK(std::numeric_limits<dest_t>::min() <= v && v <= std::numeric_limits<dest_t>::max(),
              "integer out of range");

  return static_cast<dest_t>(v);
}

template<typename T>
```
- EN: Lines 1-60 pull in 6 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, safe_downcast, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 6 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH, safe_downcast，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
inline T pooling_output_shape_pad_lr(
        T inputSize, T kernelSize, T pad_l, T pad_r, T stride, T dilation,
        bool ceil_mode) {
    T outputSize = div_rtn<T>(
        inputSize + pad_l + pad_r - dilation * (kernelSize - 1) - 1 +
        (ceil_mode ? stride - 1 : 0), stride) + 1;
    if (ceil_mode) {
        // ensure that the last pooling starts inside the image
        // needed to avoid problems in ceil mode
        if ((outputSize - 1) * stride >= inputSize + pad_l) {
          --outputSize;
        }
    }
    return outputSize;
}

template<typename T>
inline T pooling_output_shape(
      T inputSize, T kernelSize, T pad, T stride, T dilation, bool ceil_mode) {
    TORCH_CHECK(stride != 0, "stride should not be zero");
    TORCH_CHECK(pad >= 0,
                "pad must be non-negative, but got pad: ", pad);
    TORCH_CHECK(pad <= ((kernelSize - 1) * dilation + 1) / 2,
                "pad should be at most half of effective kernel size, but got pad=",
                pad, ", kernel_size=", kernelSize, " and dilation=", dilation)
    return pooling_output_shape_pad_lr(
        inputSize, kernelSize, pad, pad, stride, dilation, ceil_mode);
}

template <typename T>
std::pair<T, T> _pooling_same_mode_padding_lr(
    T inputSize, T kernelSize, T stride, T dilation) {
  // NOTE: with strides, the output shape is ceil(inputSize/stride)
  auto total_padding = T(dilation) * (kernelSize - 1);

  // Prefer symmetric padding if possible
  if (stride > 2 && (total_padding % 2 == 1)) {
    // The floor in the output size calculation gives us a little wiggle room
    auto wiggle_room = inputSize % stride - 1;
    if (wiggle_room > 0) {
      total_padding = total_padding - 1;
    }
  }

  auto left = total_padding / 2;
  return {left, total_padding - left};
}

inline std::pair<int64_t, int64_t> pooling_same_mode_padding_lr(
    int64_t inputSize, int64_t kernelSize, int64_t stride, int64_t dilation) {
  return _pooling_same_mode_padding_lr(inputSize, kernelSize, stride, dilation);
}

inline std::pair<c10::SymInt, c10::SymInt> pooling_same_mode_padding_lr(
    c10::SymInt inputSize, c10::SymInt kernelSize, c10::SymInt stride, c10::SymInt dilation) {
  return _pooling_same_mode_padding_lr(std::move(inputSize), std::move(kernelSize), std::move(stride), std::move(dilation));
}

// AveragePool2d/DilatedMaxPool2d (forward)
inline void
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are ceil, DilatedMaxPool2d, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 ceil, DilatedMaxPool2d，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp
pool2d_shape_check(
  const Tensor& input,
  int64_t kH, int64_t kW, int64_t dH, int64_t dW, int64_t padH, int64_t padW, int64_t dilationH, int64_t dilationW,
  int64_t nInputPlane,
  int64_t inputHeight, int64_t inputWidth,
  int64_t outputHeight, int64_t outputWidth, MemoryFormat memory_format)
{
  const int64_t ndim = input.ndimension();
#ifndef STRIP_ERROR_MESSAGES
  const int64_t nOutputPlane = nInputPlane;
#endif

  TORCH_CHECK(kW > 0 && kH > 0,
              "kernel size should be greater than zero, but got ",
              "kH: ", kH, " kW: ", kW);
  TORCH_CHECK(dW > 0 && dH > 0,
              "stride should be greater than zero, but got "
              "dH: ", dH, " dW: ", dW);
  TORCH_CHECK(dilationH > 0 && dilationW > 0,
              "dilation should be greater than zero, but got ",
              "dilationH: ", dilationH, " dilationW: ", dilationW);

  bool valid_dims = input.size(1) != 0 && input.size(2) != 0;
  if (memory_format == at::MemoryFormat::ChannelsLast){
    // Expect tensor in NHWC format and allow 0-dim only for N.
    TORCH_CHECK((ndim == 4 && valid_dims && input.size(3) != 0),
      "Expected 4D (batch mode) tensor expected for input with channels_last layout"
      " with optional 0 dim batch size for input, but got: ", input.sizes());
  } else {
    TORCH_CHECK((ndim == 3 && input.size(0) != 0 && valid_dims) ||
      (ndim == 4 && valid_dims && input.size(3) != 0),
      "Expected 3D or 4D (batch mode) tensor with optional 0 dim batch size for input, but got:",
      input.sizes());
  }

  TORCH_CHECK(kW/2 >= padW && kH/2 >= padH,
              "pad should be smaller than or equal to half of kernel size, but got ",
              "padW = ", padW, ", padH = ", padH, ", kW = ", kW, ", kH = ", kH);

  TORCH_CHECK(outputWidth >= 1 && outputHeight >= 1,
              "Given input size: (",
              nInputPlane, "x", inputHeight, "x", inputWidth, "). ",
              "Calculated output size: (",
              nOutputPlane, "x", outputHeight, "x", outputWidth, "). ",
              "Output size is too small");
}

// DilatedMaxPool2d (backward)
inline void
max_pool2d_backward_shape_check(
  const Tensor& input,
  const Tensor& gradOutput,
  const Tensor& indices,
  int kH, int kW, int dH, int dW, int padH, int padW, int dilationH, int dilationW,
  int64_t nInputPlane,
  int64_t inputHeight, int64_t inputWidth,
  int64_t outputHeight, int64_t outputWidth, MemoryFormat memory_format)
{
  pool2d_shape_check(
    input,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are DilatedMaxPool2d, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 DilatedMaxPool2d，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-240
```cpp
    kH, kW, dH, dW, padH, padW, dilationH, dilationW,
    nInputPlane, inputHeight, inputWidth, outputHeight, outputWidth, memory_format);

  const int64_t ndim = input.ndimension();
  const int64_t nOutputPlane = nInputPlane;

  check_dim_size(gradOutput, ndim, ndim-3, nOutputPlane);
  check_dim_size(gradOutput, ndim, ndim-2, outputHeight);
  check_dim_size(gradOutput, ndim, ndim-1, outputWidth);

  check_dim_size(indices, ndim, ndim-3, nOutputPlane);
  check_dim_size(indices, ndim, ndim-2, outputHeight);
  check_dim_size(indices, ndim, ndim-1, outputWidth);

  if (ndim == 4) {
    const int64_t batchSize = input.size(0);
    check_dim_size(gradOutput, ndim, 0, batchSize);
    check_dim_size(indices, ndim, 0, batchSize);
  }
}

// AveragePool2d (backward)
inline void
avg_pool2d_backward_shape_check(
  const Tensor& input,
  const Tensor& gradOutput,
  int64_t /*nbatch*/,
  int kH, int kW, int dH, int dW, int padH, int padW,
  int64_t nInputPlane,
  int64_t inputHeight, int64_t inputWidth,
  int64_t outputHeight, int64_t outputWidth,
  MemoryFormat memory_format)
{
  pool2d_shape_check(
    input,
    kH, kW, dH, dW, padH, padW, 1, 1,
    nInputPlane, inputHeight, inputWidth, outputHeight, outputWidth,
    memory_format);

  const int64_t ndim = input.ndimension();
  const int64_t nOutputPlane = nInputPlane;

  check_dim_size(gradOutput, ndim, ndim-3, nOutputPlane);
  check_dim_size(gradOutput, ndim, ndim-2, outputHeight);
  check_dim_size(gradOutput, ndim, ndim-1, outputWidth);
}

// AveragePool3d/DilatedMaxPool3d (forward)
inline void
pool3d_shape_check(
  const Tensor& input,
  int64_t nslices,
  int kT, int kH, int kW,
  int dT, int dH, int dW,
  int pT, int pH, int pW,
  int dilationT, int dilationH, int dilationW,
  int64_t itime, int64_t iheight, int64_t iwidth,
  int64_t otime, int64_t oheight, int64_t owidth,
  const char *fn_name,
  bool check_input_size=false)
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AveragePool2d, DilatedMaxPool3d, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AveragePool2d, DilatedMaxPool3d，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-300
```cpp
{
  const int64_t ndim = input.ndimension();

  TORCH_CHECK(kT > 0 && kW > 0 && kH > 0,
              "kernel size should be greater than zero, but got ",
              "kT: ", kT, " kH: ", kH, " kW: ", kW);
  TORCH_CHECK(dT > 0 && dW > 0 && dH > 0,
              "stride should be greater than zero, but got ",
              "dT: ", dT, " dH: ", dH, " dW: ", dW);
  TORCH_CHECK(dilationT > 0 && dilationW > 0 && dilationH > 0,
              "dilation should be greater than zero, but got ",
              "dilationT: ", dilationT, " dilationH: ", dilationH, " dilationW: ", dilationW);

  TORCH_CHECK(ndim == 4 || ndim == 5,
              fn_name, ": Expected 4D or 5D tensor for input, but got: ", input.sizes());

  for (const auto i : c10::irange(ndim)) {
    if (ndim == 5 && i == 0) {
      // size of batch-dim can be 0.
      continue;
    }
    TORCH_CHECK(
        input.size(i) > 0,
        fn_name,
        ": Expected input's non-batch dimensions to have positive length,"
        " but input has a shape of ",
        input.sizes(),
        " and non-batch dimension ",
        input.size(i),
        " has length zero!")
  }

  if (check_input_size) { // AveragePool3d
    TORCH_CHECK(itime >= kT && iheight >= kH && iwidth >= kW,
                "input image ", "(T: ", itime, " H: ", iheight, " W: ", iwidth, ") smaller than ",
                "kernel size ", "(kT: ", kT, " kH: ", kH, " kW: ", kW, ")");
  }

  TORCH_CHECK(kT/2 >= pT && kW/2 >= pW && kH/2 >= pH,
              "pad should be smaller than or equal to half of kernel size, but got "
              "kT: ", kT, " kW: ", kW, " kH: ", kH, " padT: ", pT, " padW: ", pW, " padH: ", pH);

  TORCH_CHECK(otime >= 1 && owidth >= 1 && oheight >= 1,
              "Given input size: (",
              nslices,"x", itime, "x", iheight, "x", iwidth, "). ",
              "Calculated output size: (",
              nslices, "x", otime, "x", oheight, "x", owidth, "). ",
              "Output size is too small");
}

inline void
max_pool3d_backward_shape_check(
  const Tensor& input,
  const Tensor& gradOutput,
  const Tensor& indices,
  int64_t nslices,
  int kT, int kH, int kW,
  int dT, int dH, int dW,
  int pT, int pH, int pW,
  int dilationT, int dilationH, int dilationW,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 301-360
```cpp
  int64_t itime, int64_t iheight, int64_t iwidth,
  int64_t otime, int64_t oheight, int64_t owidth,
  const char* fn_name)
{
  const int64_t ndim = input.ndimension();

  pool3d_shape_check(
    input,
    nslices,
    kT, kH, kW,
    dT, dH, dW,
    pT, pH, pW,
    dilationT, dilationH, dilationW,
    itime, iheight, iwidth,
    otime, oheight, owidth, fn_name);

  check_dim_size(gradOutput, ndim, ndim-4, nslices);
  check_dim_size(gradOutput, ndim, ndim-3, otime);
  check_dim_size(gradOutput, ndim, ndim-2, oheight);
  check_dim_size(gradOutput, ndim, ndim-1, owidth);

  check_dim_size(indices, ndim, ndim-4, nslices);
  check_dim_size(indices, ndim, ndim-3, otime);
  check_dim_size(indices, ndim, ndim-2, oheight);
  check_dim_size(indices, ndim, ndim-1, owidth);
}

inline void
avg_pool3d_backward_shape_check(
  const Tensor& input,
  const Tensor& gradOutput,
  int64_t nslices,
  int kT, int kH, int kW,
  int dT, int dH, int dW,
  int pT, int pH, int pW,
  int64_t itime, int64_t iheight, int64_t iwidth,
  int64_t otime, int64_t oheight, int64_t owidth,
  const char *fn_name)
{
  const int64_t ndim = input.ndimension();

  pool3d_shape_check(
    input,
    nslices,
    kT, kH, kW,
    dT, dH, dW,
    pT, pH, pW,
    1, 1, 1,
    itime, iheight, iwidth,
    otime, oheight, owidth,
    fn_name, true);

  check_dim_size(gradOutput, ndim, ndim-4, nslices);
  check_dim_size(gradOutput, ndim, ndim-3, otime);
  check_dim_size(gradOutput, ndim, ndim-2, oheight);
  check_dim_size(gradOutput, ndim, ndim-1, owidth);
}

} // anonymous namespace
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 361-361
```cpp
} // namespace at::native
```
- EN: This range provides supporting glue code: declarations, simple forwarding logic, or structural scaffolding reused by nearby operator code.
- CN: 这一段提供了辅助性胶水代码：声明、简单转发逻辑，或被邻近算子代码复用的结构骨架。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: DECLARE_DISPATCH, safe_downcast, pooling_output_shape_pad_lr, pooling_output_shape, TORCH_CHECK, _pooling_same_mode_padding_lr, pooling_same_mode_padding_lr, pool2d_shape_check.
- CN: 重要符号：DECLARE_DISPATCH, safe_downcast, pooling_output_shape_pad_lr, pooling_output_shape, TORCH_CHECK, _pooling_same_mode_padding_lr, pooling_same_mode_padding_lr, pool2d_shape_check。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/div_rtn.h, ATen/TensorUtils.h, ATen/native/DispatchStub.h, c10/util/irange.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/div_rtn.h, ATen/TensorUtils.h, ATen/native/DispatchStub.h, c10/util/irange.h`。
- EN: External/system headers: `utility`.
- CN: 外部/系统头文件：`utility`。
- EN: The implementation revolves around symbols such as `DECLARE_DISPATCH, safe_downcast, pooling_output_shape_pad_lr, pooling_output_shape, TORCH_CHECK, _pooling_same_mode_padding_lr, pooling_same_mode_padding_lr, pool2d_shape_check, max_pool2d_backward_shape_check, avg_pool2d_backward_shape_check`.
- CN: 实现围绕 `DECLARE_DISPATCH, safe_downcast, pooling_output_shape_pad_lr, pooling_output_shape, TORCH_CHECK, _pooling_same_mode_padding_lr, pooling_same_mode_padding_lr, pool2d_shape_check, max_pool2d_backward_shape_check, avg_pool2d_backward_shape_check` 等符号展开。
