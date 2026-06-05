# PixelShuffle.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/PixelShuffle.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on pixel shuffle; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 pixel shuffle；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/native/TensorTransformations.h>
#include <ATen/native/cpu/PixelShuffleKernel.h>
#include <ATen/native/PixelShuffle.h>

#include <c10/util/Exception.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/empty.h>
#include <ATen/ops/pixel_shuffle_native.h>
#include <ATen/ops/pixel_unshuffle_native.h>
#endif

#include <algorithm>
#include <numeric>
#include <vector>

namespace at::native {

Tensor pixel_shuffle_cpu(const Tensor& self, int64_t upscale_factor) {
  check_pixel_shuffle_shapes(self, upscale_factor);

  // Format: (B1, ..., Bn), C, H, W
  std::vector<int64_t> output_sizes(self.sizes().begin(), self.sizes().end() - 3);
  output_sizes.insert(output_sizes.end(),
      {self.size(-3) / upscale_factor / upscale_factor,
       self.size(-2) * upscale_factor,
```
- EN: Lines 1-30 pull in 12 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are pixel_shuffle_cpu, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 12 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 pixel_shuffle_cpu，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
       self.size(-1) * upscale_factor});

  auto output = at::empty({0}, self.options());
  auto memory_format = self.suggest_memory_format();
  output.resize_(output_sizes, memory_format);

  if (output.numel() == 0) {
    return output;
  }

  auto input = self.contiguous(memory_format);

  pixel_shuffle_kernel(kCPU, output, input, upscale_factor);
  return output;
}

Tensor pixel_unshuffle_cpu(const Tensor& self, int64_t downscale_factor) {
  check_pixel_unshuffle_shapes(self, downscale_factor);

  if (self.numel() == 0) {
    return self.clone();
  }

  // Format: (B1, ..., Bn), C, H, W
  std::vector<int64_t> output_sizes(self.sizes().begin(), self.sizes().end() - 3);
  output_sizes.insert(output_sizes.end(),
      {self.size(-3) * downscale_factor * downscale_factor,
       self.size(-2) / downscale_factor,
       self.size(-1) / downscale_factor});
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are pixel_unshuffle_cpu, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 pixel_unshuffle_cpu，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
  auto output = at::empty({0}, self.options());
  auto memory_format = self.suggest_memory_format();
  output.resize_(output_sizes, memory_format);

  if (output.numel() == 0) {
    return output;
  }

  auto input = self.contiguous(memory_format);

  pixel_unshuffle_kernel(kCPU, output, input, downscale_factor);
  return output;
}

Tensor math_pixel_shuffle(const Tensor& self, int64_t upscale_factor) {
  check_pixel_shuffle_shapes(self, upscale_factor);

  // Format: (B1, ..., Bn), C, H, W
  int64_t c = self.size(-3);
  int64_t h = self.size(-2);
  int64_t w = self.size(-1);
  const auto NUM_NON_BATCH_DIMS = 3;
  const auto self_sizes_batch_end = self.sizes().end() - NUM_NON_BATCH_DIMS;

  int64_t upscale_factor_squared = upscale_factor * upscale_factor;
  int64_t oc = c / upscale_factor_squared;
  int64_t oh = h * upscale_factor;
  int64_t ow = w * upscale_factor;

  // First, reshape to split the channels dim from c into 3 separate dims: (oc,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are math_pixel_shuffle, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 math_pixel_shuffle，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 91-120
```cpp
  // upscale_factor, upscale_factor). This allows shuffling to be done next by
  // permuting dims.
  std::vector<int64_t> added_dims_shape(
      self.sizes().begin(), self_sizes_batch_end);
  added_dims_shape.insert(
      added_dims_shape.end(), {oc, upscale_factor, upscale_factor, h, w});
  const auto input_reshaped = self.reshape(added_dims_shape);

  // Next, shuffle by permuting the new upscale_factor dims alongside the height and width dims.
  std::vector<int64_t> permutation(self.sizes().begin(), self_sizes_batch_end);
  // std::iota is used to maintain the batch dims within the permutation.
  std::iota(permutation.begin(), permutation.end(), 0);
  permutation.insert(permutation.end(), {-5 /* oc */, -2 /* h */, -4 /* 1st upscale_factor */, -1 /* w */,
                                         -3 /* 2nd upscale_factor */});
  const auto input_permuted = input_reshaped.permute(permutation);

  // Finally, upscale by collapsing (h, upscale_factor) -> a single dim (oh)
  // and (w, upscale_factor) -> a single dim (ow).
  std::vector<int64_t> final_shape(self.sizes().begin(), self_sizes_batch_end);
  final_shape.insert(final_shape.end(), {oc, oh, ow});

  // pixel_shuffle expects to *never* return an alias of the input.
  return input_permuted.clone(at::MemoryFormat::Contiguous).view(final_shape);
}

Tensor math_pixel_unshuffle(const Tensor& self, int64_t downscale_factor) {
  check_pixel_unshuffle_shapes(self, downscale_factor);

  // Format: (B1, ..., Bn), C, H, W
  int64_t c = self.size(-3);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are collapsing, math_pixel_unshuffle, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 collapsing, math_pixel_unshuffle，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-150
```cpp
  int64_t h = self.size(-2);
  int64_t w = self.size(-1);
  constexpr auto NUM_NON_BATCH_DIMS = 3;
  const auto self_sizes_batch_end = self.sizes().end() - NUM_NON_BATCH_DIMS;

  int64_t downscale_factor_squared = downscale_factor * downscale_factor;
  int64_t oc = c * downscale_factor_squared;
  int64_t oh = h / downscale_factor;
  int64_t ow = w / downscale_factor;

  // First, reshape to split height dim into (oh, downscale_factor) dims and
  // width dim into (ow, downscale_factor) dims. This allows unshuffling to be
  // done next by permuting dims.
  std::vector<int64_t> added_dims_shape(
      self.sizes().begin(), self_sizes_batch_end);
  added_dims_shape.insert(
      added_dims_shape.end(), {c, oh, downscale_factor, ow, downscale_factor});
  const auto input_reshaped = self.reshape(added_dims_shape);

  // Next, unshuffle by permuting the downscale_factor dims alongside the channel dim.
  std::vector<int64_t> permutation(self.sizes().begin(), self_sizes_batch_end);
  // std::iota is used to maintain the batch dims within the permutation.
  std::iota(permutation.begin(), permutation.end(), 0);
  permutation.insert(permutation.end(), {-5 /* c */, -3 /* 1st downscale_factor */, -1 /*2nd downscale_factor */,
                                         -4 /* oh */, -2 /* ow */});
  const auto input_permuted = input_reshaped.permute(permutation);

  // Finally, downscale by collapsing (c, downscale_factor, downscale_factor) -> a single dim (oc),
  // resulting in height=oh and width=ow.
  std::vector<int64_t> final_shape(self.sizes().begin(), self_sizes_batch_end);
```
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 151-160
```cpp
  final_shape.insert(final_shape.end(), {oc, oh, ow});

  // pixel_unshuffle expects to *never* return an alias of the input.
  return input_permuted.clone(at::MemoryFormat::Contiguous).view(final_shape);
}

DEFINE_DISPATCH(pixel_shuffle_kernel);
DEFINE_DISPATCH(pixel_unshuffle_kernel);

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Notable symbols: pixel_shuffle_cpu, pixel_unshuffle_cpu, math_pixel_shuffle, math_pixel_unshuffle.
- CN: 重要符号：pixel_shuffle_cpu, pixel_unshuffle_cpu, math_pixel_shuffle, math_pixel_unshuffle。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/TensorTransformations.h, ATen/native/cpu/PixelShuffleKernel.h, ATen/native/PixelShuffle.h, c10/util/Exception.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/empty.h, ATen/ops/pixel_shuffle_native.h, ATen/ops/pixel_unshuffle_native.h`.
- CN: 主要内部头文件：`ATen/native/TensorTransformations.h, ATen/native/cpu/PixelShuffleKernel.h, ATen/native/PixelShuffle.h, c10/util/Exception.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/empty.h, ATen/ops/pixel_shuffle_native.h, ATen/ops/pixel_unshuffle_native.h`。
- EN: External/system headers: `algorithm, numeric, vector`.
- CN: 外部/系统头文件：`algorithm, numeric, vector`。
- EN: The implementation revolves around symbols such as `pixel_shuffle_cpu, pixel_unshuffle_cpu, math_pixel_shuffle, math_pixel_unshuffle`.
- CN: 实现围绕 `pixel_shuffle_cpu, pixel_unshuffle_cpu, math_pixel_shuffle, math_pixel_unshuffle` 等符号展开。
