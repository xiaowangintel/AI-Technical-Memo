# ReplicationPadding.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/ReplicationPadding.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on replication padding; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 replication padding；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>
#include <ATen/Dispatch.h>
#include <ATen/Parallel.h>
#include <ATen/TensorMeta.h>
#include <ATen/native/Padding.h>
#include <c10/util/irange.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/replication_pad1d_backward_native.h>
#include <ATen/ops/replication_pad1d_native.h>
#include <ATen/ops/replication_pad2d_backward_native.h>
#include <ATen/ops/replication_pad2d_native.h>
#include <ATen/ops/replication_pad3d_backward_native.h>
#include <ATen/ops/replication_pad3d_native.h>
#include <ATen/ops/zeros_like.h>
#endif

namespace at::meta {

TORCH_META_FUNC(replication_pad1d) (
  const Tensor& input, IntArrayRef paddingSize  // no out argument!
) {
  TORCH_CHECK(paddingSize.size() == 2, "padding size is expected to be 2");

  int64_t dimw = 1;
  int64_t dimslices = 0;
  int64_t nbatch = 1;

  int64_t pad_l = paddingSize[0];
  int64_t pad_r = paddingSize[1];

  at::native::padding::check_valid_input<1>(input, paddingSize);

  if (input.ndimension() == 3) {
    nbatch = input.size(0);
    dimw++;
    dimslices++;
  }

  /* sizes */
  int64_t nslices = input.size(dimslices);
  int64_t iwidth = input.size(dimw);
  int64_t owidth = iwidth + pad_l + pad_r;

  TORCH_CHECK(owidth >= 1,
      "input (W: ", iwidth, ") is too small."
      " Calculated output W: ", owidth);

  if (input.ndimension() == 2) {
    set_output_raw_strided(0, {nslices, owidth}, {}, input.options());
  } else {
    set_output_raw_strided(0, {nbatch, nslices, owidth}, {}, input.options());
  }
}

TORCH_META_FUNC(replication_pad1d_backward) (
```
- EN: Lines 1-60 pull in 15 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 第 1-60 行引入了 15 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-120
```cpp
  const Tensor& gradOutput,
  const Tensor& input,
  IntArrayRef paddingSize
) {
  int64_t dimw = 1;
  TORCH_CHECK(paddingSize.size() == 2, "padding size is expected to be 2");
  int64_t pad_l = paddingSize[0];
  int64_t pad_r = paddingSize[1];

  if (input.ndimension() == 3) {
    dimw++;
  }

  /* sizes */
  int64_t iwidth = input.size(dimw);
  int64_t owidth  = iwidth + pad_l + pad_r;

  TORCH_CHECK(owidth == gradOutput.size(dimw),
      "gradOutput width unexpected. Expected: ", owidth,
      " Got: ", gradOutput.size(dimw));

  set_output_raw_strided(0, input.sizes(), {}, input.options());
}

TORCH_META_FUNC(replication_pad2d) (
  const Tensor& input, IntArrayRef paddingSize
) {
  TORCH_CHECK(paddingSize.size() == 4, "padding size is expected to be 4");
  int64_t pad_l = paddingSize[0];
  int64_t pad_r = paddingSize[1];
  int64_t pad_t = paddingSize[2];
  int64_t pad_b = paddingSize[3];
  int64_t dimw = 2;
  int64_t dimh = 1;
  int64_t dimslices = 0;
  int64_t nbatch = 1;

  at::native::padding::check_valid_input<2>(input, paddingSize);

  if (input.dim() == 4) {
    nbatch = input.size(0);
    dimw++;
    dimh++;
    dimslices++;
  }

  /* sizes */
  int64_t nslices = input.size(dimslices);
  int64_t iheight = input.size(dimh);
  int64_t iwidth = input.size(dimw);
  int64_t oheight = iheight + pad_t + pad_b;
  int64_t owidth  = iwidth + pad_l + pad_r;

  TORCH_CHECK(owidth >= 1 || oheight >= 1,
      "input (H: ", iheight, ", W: ", iwidth, " ) is too small."
      " Calculated output H: ", oheight, " W: ", owidth);

  if (input.dim() == 3) {
    set_output_raw_strided(0, {nslices, oheight, owidth}, {}, input.options());
  } else {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-180
```cpp
    set_output_raw_strided(0, {nbatch, nslices, oheight, owidth}, {}, input.options());
  }
}

TORCH_META_FUNC(replication_pad3d) (
  const Tensor& input, IntArrayRef paddingSize
) {
  TORCH_CHECK(paddingSize.size() == 6, "padding size is expected to be 6");
  int64_t pleft = paddingSize[0];
  int64_t pright = paddingSize[1];
  int64_t ptop = paddingSize[2];
  int64_t pbottom = paddingSize[3];
  int64_t pfront = paddingSize[4];
  int64_t pback = paddingSize[5];
  int64_t dimw = 3;
  int64_t dimh = 2;
  int64_t dimd = 1;
  int64_t dimslices = 0;
  int64_t nbatch = 1;

  at::native::padding::check_valid_input<3>(input, paddingSize);

  if (input.dim() == 5) {
    nbatch = input.size(0);
    dimw++;
    dimh++;
    dimd++;
    dimslices++;
  }

  /* sizes */
  int64_t nslices = input.size(dimslices);
  int64_t idepth = input.size(dimd);
  int64_t iheight = input.size(dimh);
  int64_t iwidth = input.size(dimw);
  int64_t odepth = idepth + pfront + pback;
  int64_t oheight = iheight + ptop + pbottom;
  int64_t owidth  = iwidth + pleft + pright;

  TORCH_CHECK(owidth >= 1 || oheight >= 1 || odepth >= 1,
      "input (D: ", idepth, " H: ", iheight, ", W: ", iwidth,
      ") is too small."
      " Calculated output D: ", odepth, " H: ", oheight, " W: ", owidth);

  /* resize output */
  if (input.dim() == 4) {
    set_output_raw_strided(0, {nslices, odepth, oheight, owidth}, {}, input.options());
  } else {
    set_output_raw_strided(0, {nbatch, nslices, odepth, oheight, owidth}, {}, input.options());
  }
}

} // namespace at::meta

namespace at::native {

namespace {

void replication_pad2d_backward_out_cpu_template(
    Tensor& gradInput,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-240
```cpp
    const Tensor& gradOutput,
    const Tensor& input,
    IntArrayRef paddingSize)
{
  TORCH_CHECK(paddingSize.size() == 4, "padding size is expected to be 4");
  int pad_l = paddingSize[0];
  int pad_r = paddingSize[1];
  int pad_t = paddingSize[2];
  int pad_b = paddingSize[3];
  int dimw = 2;
  int dimh = 1;

  if (input.dim() == 4) {
    dimw++;
    dimh++;
  }

  /* sizes */
  int64_t iheight = input.size(dimh);
  int64_t iwidth = input.size(dimw);
  int64_t oheight = iheight + pad_t + pad_b;
  int64_t owidth  = iwidth + pad_l + pad_r;

  TORCH_CHECK(owidth == gradOutput.size(dimw),
      "gradOutput width unexpected. Expected: ", owidth, ", Got: ",
      gradOutput.size(dimw));
  TORCH_CHECK(oheight == gradOutput.size(dimh),
      "gradOutput height unexpected. Expected: ", oheight, ", Got: ",
      gradOutput.size(dimh));

  if (gradInput.numel() == 0) {
    return;
  }

  replication_pad2d_backward_kernel(kCPU, gradInput, gradOutput, paddingSize);
}

void replication_pad3d_backward_out_cpu_template(
    Tensor& gradInput,
    const Tensor& gradOutput,
    const Tensor& input,
    IntArrayRef paddingSize)
{
  TORCH_CHECK(paddingSize.size() == 6, "padding size is expected to be 6");
  int pleft = paddingSize[0];
  int pright = paddingSize[1];
  int ptop = paddingSize[2];
  int pbottom = paddingSize[3];
  int pfront = paddingSize[4];
  int pback = paddingSize[5];
  int dimc = 0;
  int dimw = 3;
  int dimh = 2;
  int dimd = 1;

  if (input.dim() == 5) {
    dimc++;
    dimw++;
    dimh++;
    dimd++;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-300
```cpp
  }

  /* sizes */
  int64_t ichannel = input.size(dimc);
  int64_t idepth = input.size(dimd);
  int64_t iheight = input.size(dimh);
  int64_t iwidth = input.size(dimw);
  int64_t odepth = idepth + pfront + pback;
  int64_t oheight = iheight + ptop + pbottom;
  int64_t owidth  = iwidth + pleft + pright;

  at::native::padding::check_valid_input<3>(input, paddingSize);

  TORCH_CHECK(ichannel == gradOutput.size(dimc),
      "gradOutput channel unexpected. Expected: ", ichannel, ", Got: ",
      gradOutput.size(dimc));
  TORCH_CHECK(owidth == gradOutput.size(dimw),
      "gradOutput width unexpected. Expected: ", owidth, ", Got: ",
      gradOutput.size(dimw));
  TORCH_CHECK(oheight == gradOutput.size(dimh),
      "gradOutput height unexpected. Expected: ", oheight, ", Got: ",
      gradOutput.size(dimh));
  TORCH_CHECK(odepth == gradOutput.size(dimd),
      "gradOutput depth unexpected. Expected: ", odepth, ", Got: ",
      gradOutput.size(dimd));

  if (gradInput.numel() == 0) {
    return;
  }

  replication_pad3d_backward_kernel(kCPU, gradInput, gradOutput, paddingSize);
}

} // anonymous namespace

TORCH_IMPL_FUNC(replication_pad1d_out_cpu) (
  const Tensor& input, IntArrayRef paddingSize, const Tensor& output
) {
  replication_pad1d_kernel(kCPU, output, input, paddingSize);
}

TORCH_IMPL_FUNC(replication_pad1d_backward_out_cpu) (
  const Tensor& gradOutput, const Tensor& input, IntArrayRef paddingSize, const Tensor& gradInput
) {
  if (gradInput.numel() == 0) {
    return;
  }
  gradInput.zero_();

  replication_pad1d_backward_kernel(kCPU, gradInput, gradOutput, paddingSize);
}

TORCH_IMPL_FUNC(replication_pad2d_out_cpu) (
  const Tensor& input, IntArrayRef paddingSize, const Tensor& output
) {
  // TODO: move this to TORCH_META_FUNC when CUDA has channels last support
  output.resize_(output.sizes(), input.suggest_memory_format());

  replication_pad2d_kernel(kCPU, output, input, paddingSize);
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 301-360
```cpp

Tensor& replication_pad2d_backward_out_cpu(const Tensor& gradOutput,
    const Tensor& input,
    IntArrayRef paddingSize,
    Tensor& gradInput)
{
  gradInput.resize_as_(input, input.suggest_memory_format());
  gradInput.zero_();
  replication_pad2d_backward_out_cpu_template(
      gradInput, gradOutput, input, paddingSize);
  return gradInput;
}

Tensor replication_pad2d_backward_cpu(
    const Tensor& gradOutput,
    const Tensor& input,
    IntArrayRef paddingSize)
{
  auto gradInput = at::zeros_like(input, input.suggest_memory_format());
  replication_pad2d_backward_out_cpu_template(
      gradInput, gradOutput, input, paddingSize);
  return gradInput;
}

TORCH_IMPL_FUNC(replication_pad3d_out_cpu) (
  const Tensor& input, IntArrayRef paddingSize, const Tensor& output
) {
  // TODO: move this to TORCH_META_FUNC when CUDA has channels last support
  output.resize_(output.sizes(), input.suggest_memory_format());

  replication_pad3d_kernel(kCPU, output, input, paddingSize);
}

Tensor& replication_pad3d_backward_out_cpu(const Tensor& gradOutput,
    const Tensor& input,
    IntArrayRef paddingSize,
    Tensor& gradInput)
{
  gradInput.resize_as_(input, input.suggest_memory_format());
  gradInput.zero_();
  replication_pad3d_backward_out_cpu_template(
      gradInput, gradOutput, input, paddingSize);
  return gradInput;
}

Tensor replication_pad3d_backward_cpu(
    const Tensor& gradOutput,
    const Tensor& input,
    IntArrayRef paddingSize)
{
  auto gradInput = at::zeros_like(input, input.suggest_memory_format());
  replication_pad3d_backward_out_cpu_template(
      gradInput, gradOutput, input, paddingSize);
  return gradInput;
}

DEFINE_DISPATCH(replication_pad1d_kernel);
DEFINE_DISPATCH(replication_pad1d_backward_kernel);
DEFINE_DISPATCH(replication_pad2d_kernel);
DEFINE_DISPATCH(replication_pad2d_backward_kernel);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 361-364
```cpp
DEFINE_DISPATCH(replication_pad3d_kernel);
DEFINE_DISPATCH(replication_pad3d_backward_kernel);

} // namespace at::native
```
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

## Key Concepts / 关键概念

- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: TORCH_META_FUNC, replication_pad2d_backward_out_cpu_template, replication_pad3d_backward_out_cpu_template, TORCH_IMPL_FUNC, replication_pad2d_backward_out_cpu, replication_pad2d_backward_cpu, replication_pad3d_backward_out_cpu, replication_pad3d_backward_cpu.
- CN: 重要符号：TORCH_META_FUNC, replication_pad2d_backward_out_cpu_template, replication_pad3d_backward_out_cpu_template, TORCH_IMPL_FUNC, replication_pad2d_backward_out_cpu, replication_pad2d_backward_cpu, replication_pad3d_backward_out_cpu, replication_pad3d_backward_cpu。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/TensorMeta.h, ATen/native/Padding.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/replication_pad1d_backward_native.h, ATen/ops/replication_pad1d_native.h, ATen/ops/replication_pad2d_backward_native.h, ATen/ops/replication_pad2d_native.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/TensorMeta.h, ATen/native/Padding.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/replication_pad1d_backward_native.h, ATen/ops/replication_pad1d_native.h, ATen/ops/replication_pad2d_backward_native.h, ATen/ops/replication_pad2d_native.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `TORCH_META_FUNC, replication_pad2d_backward_out_cpu_template, replication_pad3d_backward_out_cpu_template, TORCH_IMPL_FUNC, replication_pad2d_backward_out_cpu, replication_pad2d_backward_cpu, replication_pad3d_backward_out_cpu, replication_pad3d_backward_cpu`.
- CN: 实现围绕 `TORCH_META_FUNC, replication_pad2d_backward_out_cpu_template, replication_pad3d_backward_out_cpu_template, TORCH_IMPL_FUNC, replication_pad2d_backward_out_cpu, replication_pad2d_backward_cpu, replication_pad3d_backward_out_cpu, replication_pad3d_backward_cpu` 等符号展开。
