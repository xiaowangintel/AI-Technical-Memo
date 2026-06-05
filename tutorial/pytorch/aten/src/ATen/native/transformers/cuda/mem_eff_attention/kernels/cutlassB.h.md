# cutlassB.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/transformers/cuda/mem_eff_attention/kernels/cutlassB.h`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Declares interfaces, templates, constants, or helper types for Memory-efficient attention CUDA specialization, centered on cutlass B with emphasis on attention computation.
- 用途（中文）: 声明接口、模板、常量或辅助类型，属于高效注意力 CUDA 特化实现，核心主题是cutlass B，重点关注注意力计算。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-30

```cpp
   1: /*
   2:  * Copyright (c) Meta Platforms, Inc. and affiliates.
   3:  * All rights reserved.
   4:  *
   5:  * This source code is licensed under the BSD-style license found in the
   6:  * LICENSE file in the root directory of this source tree.
   7:  */
   8: // This file is auto-generated. See "generate_kernels.py"
   9: #pragma once
  10: #include <ATen/native/transformers/cuda/mem_eff_attention/kernel_backward.h>
  11: using namespace PyTorchMemEffAttention;
  12: // ======== f16 / sm70 ========
  13: __global__ void __launch_bounds__(
  14:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32, true>::kNumThreads,
  15:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32, true>::kMinBlocksPerSm)
  16: fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32, true>::Params p);
  17: __global__ void __launch_bounds__(
  18:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32>::kNumThreads,
  19:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32>::kMinBlocksPerSm)
  20: fmha_cutlassB_f16_aligned_64x64_k32_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32>::Params p);
  21: __global__ void __launch_bounds__(
  22:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64, true>::kNumThreads,
  23:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64, true>::kMinBlocksPerSm)
  24: fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64, true>::Params p);
  25: __global__ void __launch_bounds__(
  26:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64>::kNumThreads,
  27:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64>::kMinBlocksPerSm)
  28: fmha_cutlassB_f16_aligned_64x64_k64_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64>::Params p);
  29: __global__ void __launch_bounds__(
  30:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128, true>::kNumThreads,
```
- L1: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L2: Documents the nearby logic: Copyright (c) Meta Platforms, Inc. and affiliates. / 说明附近逻辑的作用：Copyright (c) Meta Platforms, Inc. and affiliates.
- L3: Documents the nearby logic: All rights reserved. / 说明附近逻辑的作用：All rights reserved.
- L4: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L5: Documents the nearby logic: This source code is licensed under the BSD-style license found in the / 说明附近逻辑的作用：This source code is licensed under the BSD-style license found in the
- L6: Documents the nearby logic: LICENSE file in the root directory of this source tree. / 说明附近逻辑的作用：LICENSE file in the root directory of this source tree.
- L7: Documents the nearby logic: / / 说明附近逻辑的作用：/
- L8: Documents the nearby logic: This file is auto-generated. See "generate_kernels.py" / 说明附近逻辑的作用：This file is auto-generated. See "generate_kernels.py"
- L9: Prevents repeated inclusion of this header in one translation unit. / 防止该头文件在同一翻译单元中被重复包含。
- L10: Includes `ATen/native/transformers/cuda/mem_eff_attention/kernel_backward.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/transformers/cuda/mem_eff_attention/kernel_backward.h`，为 ATen 的张量/算子基础设施提供支持。
- L11: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L12: Documents the nearby logic: ======== f16 / sm70 ======== / 说明附近逻辑的作用：======== f16 / sm70 ========
- L13: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L14: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L15: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L16: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm70`，作为本文件可调用接口的一部分。
- L17: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L18: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L19: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L20: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_sm70`，作为本文件可调用接口的一部分。
- L21: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L22: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L23: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L24: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm70`，作为本文件可调用接口的一部分。
- L25: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L26: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L27: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L28: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_sm70`，作为本文件可调用接口的一部分。
- L29: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L30: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 31-60

```cpp
  31:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128, true>::kMinBlocksPerSm)
  32: fmha_cutlassB_f16_aligned_128x64_k128_seqaligned_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128, true>::Params p);
  33: __global__ void __launch_bounds__(
  34:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128>::kNumThreads,
  35:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128>::kMinBlocksPerSm)
  36: fmha_cutlassB_f16_aligned_128x64_k128_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128>::Params p);
  37: __global__ void __launch_bounds__(
  38:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128, true>::kNumThreads,
  39:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128, true>::kMinBlocksPerSm)
  40: fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128, true>::Params p);
  41: __global__ void __launch_bounds__(
  42:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128>::kNumThreads,
  43:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
  44: fmha_cutlassB_f16_aligned_64x64_k128_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128>::Params p);
  45: __global__ void __launch_bounds__(
  46:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 65536>::kNumThreads,
  47:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 65536>::kMinBlocksPerSm)
  48: fmha_cutlassB_f16_aligned_128x64_k65536_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 65536>::Params p);
  49: __global__ void __launch_bounds__(
  50:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 65536>::kNumThreads,
  51:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
  52: fmha_cutlassB_f16_aligned_64x64_k65536_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 65536>::Params p);
  53: __global__ void __launch_bounds__(
  54:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 32>::kNumThreads,
  55:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 32>::kMinBlocksPerSm)
  56: fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 32>::Params p);
  57: __global__ void __launch_bounds__(
  58:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 64>::kNumThreads,
  59:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 64>::kMinBlocksPerSm)
  60: fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 64>::Params p);
```
- L31: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L32: Declares function `fmha_cutlassB_f16_aligned_128x64_k128_seqaligned_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k128_seqaligned_sm70`，作为本文件可调用接口的一部分。
- L33: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L34: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L35: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L36: Declares function `fmha_cutlassB_f16_aligned_128x64_k128_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k128_sm70`，作为本文件可调用接口的一部分。
- L37: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L38: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L39: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L40: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm70`，作为本文件可调用接口的一部分。
- L41: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L42: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L43: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L44: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_sm70`，作为本文件可调用接口的一部分。
- L45: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L46: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L47: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L48: Declares function `fmha_cutlassB_f16_aligned_128x64_k65536_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k65536_sm70`，作为本文件可调用接口的一部分。
- L49: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L50: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L51: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L52: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_sm70`，作为本文件可调用接口的一部分。
- L53: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L54: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L55: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L56: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm70`，作为本文件可调用接口的一部分。
- L57: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L58: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L59: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L60: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm70`，作为本文件可调用接口的一部分。

### Lines 61-90

```cpp
  61: __global__ void __launch_bounds__(
  62:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 128>::kNumThreads,
  63:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 128>::kMinBlocksPerSm)
  64: fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 128>::Params p);
  65: __global__ void __launch_bounds__(
  66:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 128>::kNumThreads,
  67:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
  68: fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 128>::Params p);
  69: __global__ void __launch_bounds__(
  70:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 65536>::kNumThreads,
  71:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 65536>::kMinBlocksPerSm)
  72: fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 65536>::Params p);
  73: __global__ void __launch_bounds__(
  74:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 65536>::kNumThreads,
  75:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
  76: fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 65536>::Params p);
  77: __global__ void __launch_bounds__(
  78:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 32>::kNumThreads,
  79:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 32>::kMinBlocksPerSm)
  80: fmha_cutlassB_f16_notaligned_64x64_k32_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 32>::Params p);
  81: __global__ void __launch_bounds__(
  82:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 64>::kNumThreads,
  83:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 64>::kMinBlocksPerSm)
  84: fmha_cutlassB_f16_notaligned_64x64_k64_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 64>::Params p);
  85: __global__ void __launch_bounds__(
  86:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 128>::kNumThreads,
  87:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 128>::kMinBlocksPerSm)
  88: fmha_cutlassB_f16_notaligned_128x64_k128_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 128>::Params p);
  89: __global__ void __launch_bounds__(
  90:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 128>::kNumThreads,
```
- L61: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L62: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L63: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L64: Declares function `fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm70`，作为本文件可调用接口的一部分。
- L65: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L66: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L67: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L68: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm70`，作为本文件可调用接口的一部分。
- L69: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L70: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L71: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L72: Declares function `fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm70`，作为本文件可调用接口的一部分。
- L73: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L74: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L75: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L76: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm70`，作为本文件可调用接口的一部分。
- L77: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L78: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L79: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L80: Declares function `fmha_cutlassB_f16_notaligned_64x64_k32_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k32_sm70`，作为本文件可调用接口的一部分。
- L81: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L82: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L83: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L84: Declares function `fmha_cutlassB_f16_notaligned_64x64_k64_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k64_sm70`，作为本文件可调用接口的一部分。
- L85: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L86: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L87: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L88: Declares function `fmha_cutlassB_f16_notaligned_128x64_k128_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k128_sm70`，作为本文件可调用接口的一部分。
- L89: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L90: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 91-120

```cpp
  91:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 128>::kMinBlocksPerSm)
  92: fmha_cutlassB_f16_notaligned_64x64_k128_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 128>::Params p);
  93: __global__ void __launch_bounds__(
  94:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 65536>::kNumThreads,
  95:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 65536>::kMinBlocksPerSm)
  96: fmha_cutlassB_f16_notaligned_128x64_k65536_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 65536>::Params p);
  97: __global__ void __launch_bounds__(
  98:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 65536>::kNumThreads,
  99:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 100: fmha_cutlassB_f16_notaligned_64x64_k65536_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 65536>::Params p);
 101: __global__ void __launch_bounds__(
 102:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 32>::kNumThreads,
 103:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 32>::kMinBlocksPerSm)
 104: fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 32>::Params p);
 105: __global__ void __launch_bounds__(
 106:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 64>::kNumThreads,
 107:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 64>::kMinBlocksPerSm)
 108: fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 64>::Params p);
 109: __global__ void __launch_bounds__(
 110:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 128>::kNumThreads,
 111:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 128>::kMinBlocksPerSm)
 112: fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 128>::Params p);
 113: __global__ void __launch_bounds__(
 114:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 128>::kNumThreads,
 115:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 128>::kMinBlocksPerSm)
 116: fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 128>::Params p);
 117: __global__ void __launch_bounds__(
 118:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 65536>::kNumThreads,
 119:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 65536>::kMinBlocksPerSm)
 120: fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 65536>::Params p);
```
- L91: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L92: Declares function `fmha_cutlassB_f16_notaligned_64x64_k128_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k128_sm70`，作为本文件可调用接口的一部分。
- L93: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L94: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L95: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L96: Declares function `fmha_cutlassB_f16_notaligned_128x64_k65536_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k65536_sm70`，作为本文件可调用接口的一部分。
- L97: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L98: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L99: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L100: Declares function `fmha_cutlassB_f16_notaligned_64x64_k65536_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k65536_sm70`，作为本文件可调用接口的一部分。
- L101: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L102: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L103: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L104: Declares function `fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm70`，作为本文件可调用接口的一部分。
- L105: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L106: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L107: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L108: Declares function `fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm70`，作为本文件可调用接口的一部分。
- L109: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L110: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L111: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L112: Declares function `fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm70`，作为本文件可调用接口的一部分。
- L113: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L114: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L115: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L116: Declares function `fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm70`，作为本文件可调用接口的一部分。
- L117: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L118: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L119: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L120: Declares function `fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm70`，作为本文件可调用接口的一部分。

### Lines 121-150

```cpp
 121: __global__ void __launch_bounds__(
 122:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 65536>::kNumThreads,
 123:     AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 124: fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 65536>::Params p);
 125: 
 126: template <typename T> void dispatch_cutlassB_f16_sm70(T cb, int cc) {
 127:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32, true>(), fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm70);
 128:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_sm70);
 129:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64, true>(), fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm70);
 130:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_sm70);
 131:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128, true>(), fmha_cutlassB_f16_aligned_128x64_k128_seqaligned_sm70);
 132:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 128>(), fmha_cutlassB_f16_aligned_128x64_k128_sm70);
 133:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128, true>(), fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm70);
 134:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_sm70);
 135:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 128, 64, 65536>(), fmha_cutlassB_f16_aligned_128x64_k65536_sm70);
 136:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_sm70);
 137:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm70);
 138:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm70);
 139:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 128>(), fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm70);
 140:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm70);
 141:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 128, 64, 65536>(), fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm70);
 142:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm70);
 143:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 32>(), fmha_cutlassB_f16_notaligned_64x64_k32_sm70);
 144:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 64>(), fmha_cutlassB_f16_notaligned_64x64_k64_sm70);
 145:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 128>(), fmha_cutlassB_f16_notaligned_128x64_k128_sm70);
 146:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 128>(), fmha_cutlassB_f16_notaligned_64x64_k128_sm70);
 147:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 128, 64, 65536>(), fmha_cutlassB_f16_notaligned_128x64_k65536_sm70);
 148:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, false, false, 64, 64, 65536>(), fmha_cutlassB_f16_notaligned_64x64_k65536_sm70);
 149:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 32>(), fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm70);
 150:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 64>(), fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm70);
```
- L121: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L122: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L123: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L124: Declares function `fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm70`，作为本文件可调用接口的一部分。
- L126: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L127: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L128: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L129: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L130: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L131: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L132: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L133: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L134: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L135: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L136: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L137: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L138: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L139: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L140: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L141: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L142: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L143: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L144: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L145: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L146: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L147: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L148: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L149: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L150: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。

### Lines 151-180

```cpp
 151:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 128>(), fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm70);
 152:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 128>(), fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm70);
 153:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 128, 64, 65536>(), fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm70);
 154:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, cutlass::half_t, false, true, false, 64, 64, 65536>(), fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm70);
 155: }
 156: 
 157: // ======== bf16 / sm80 ========
 158: __global__ void __launch_bounds__(
 159:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32, true>::kNumThreads,
 160:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32, true>::kMinBlocksPerSm)
 161: fmha_cutlassB_bf16_aligned_64x64_k32_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32, true>::Params p);
 162: __global__ void __launch_bounds__(
 163:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32>::kNumThreads,
 164:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32>::kMinBlocksPerSm)
 165: fmha_cutlassB_bf16_aligned_64x64_k32_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32>::Params p);
 166: __global__ void __launch_bounds__(
 167:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64, true>::kNumThreads,
 168:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64, true>::kMinBlocksPerSm)
 169: fmha_cutlassB_bf16_aligned_64x64_k64_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64, true>::Params p);
 170: __global__ void __launch_bounds__(
 171:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64>::kNumThreads,
 172:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64>::kMinBlocksPerSm)
 173: fmha_cutlassB_bf16_aligned_64x64_k64_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64>::Params p);
 174: __global__ void __launch_bounds__(
 175:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 64, 96>::kNumThreads,
 176:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 64, 96>::kMinBlocksPerSm)
 177: fmha_cutlassB_bf16_aligned_128x64_k96_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 64, 96>::Params p);
 178: __global__ void __launch_bounds__(
 179:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128, true>::kNumThreads,
 180:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128, true>::kMinBlocksPerSm)
```
- L151: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L152: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L153: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L154: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L155: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L157: Documents the nearby logic: ======== bf16 / sm80 ======== / 说明附近逻辑的作用：======== bf16 / sm80 ========
- L158: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L159: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L160: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L161: Declares function `fmha_cutlassB_bf16_aligned_64x64_k32_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k32_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L162: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L163: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L164: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L165: Declares function `fmha_cutlassB_bf16_aligned_64x64_k32_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k32_sm80`，作为本文件可调用接口的一部分。
- L166: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L167: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L168: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L169: Declares function `fmha_cutlassB_bf16_aligned_64x64_k64_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k64_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L170: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L171: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L172: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L173: Declares function `fmha_cutlassB_bf16_aligned_64x64_k64_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k64_sm80`，作为本文件可调用接口的一部分。
- L174: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L175: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L176: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L177: Declares function `fmha_cutlassB_bf16_aligned_128x64_k96_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_128x64_k96_sm80`，作为本文件可调用接口的一部分。
- L178: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L179: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L180: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 181-210

```cpp
 181: fmha_cutlassB_bf16_aligned_128x128_k128_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128, true>::Params p);
 182: __global__ void __launch_bounds__(
 183:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128>::kNumThreads,
 184:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128>::kMinBlocksPerSm)
 185: fmha_cutlassB_bf16_aligned_128x128_k128_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128>::Params p);
 186: __global__ void __launch_bounds__(
 187:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128, true>::kNumThreads,
 188:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128, true>::kMinBlocksPerSm)
 189: fmha_cutlassB_bf16_aligned_64x64_k128_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128, true>::Params p);
 190: __global__ void __launch_bounds__(
 191:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128>::kNumThreads,
 192:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 193: fmha_cutlassB_bf16_aligned_64x64_k128_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128>::Params p);
 194: __global__ void __launch_bounds__(
 195:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 128, 64, 65536>::kNumThreads,
 196:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 128, 64, 65536>::kMinBlocksPerSm)
 197: fmha_cutlassB_bf16_aligned_128x64_k65536_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 128, 64, 65536>::Params p);
 198: __global__ void __launch_bounds__(
 199:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 65536>::kNumThreads,
 200:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 201: fmha_cutlassB_bf16_aligned_64x64_k65536_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 65536>::Params p);
 202: __global__ void __launch_bounds__(
 203:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 32>::kNumThreads,
 204:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 32>::kMinBlocksPerSm)
 205: fmha_cutlassB_bf16_aligned_64x64_k32_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 32>::Params p);
 206: __global__ void __launch_bounds__(
 207:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 64>::kNumThreads,
 208:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 64>::kMinBlocksPerSm)
 209: fmha_cutlassB_bf16_aligned_64x64_k64_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 64>::Params p);
 210: __global__ void __launch_bounds__(
```
- L181: Declares function `fmha_cutlassB_bf16_aligned_128x128_k128_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_128x128_k128_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L182: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L183: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L184: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L185: Declares function `fmha_cutlassB_bf16_aligned_128x128_k128_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_128x128_k128_sm80`，作为本文件可调用接口的一部分。
- L186: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L187: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L188: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L189: Declares function `fmha_cutlassB_bf16_aligned_64x64_k128_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k128_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L190: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L191: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L192: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L193: Declares function `fmha_cutlassB_bf16_aligned_64x64_k128_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k128_sm80`，作为本文件可调用接口的一部分。
- L194: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L195: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L196: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L197: Declares function `fmha_cutlassB_bf16_aligned_128x64_k65536_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_128x64_k65536_sm80`，作为本文件可调用接口的一部分。
- L198: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L199: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L200: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L201: Declares function `fmha_cutlassB_bf16_aligned_64x64_k65536_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k65536_sm80`，作为本文件可调用接口的一部分。
- L202: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L203: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L204: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L205: Declares function `fmha_cutlassB_bf16_aligned_64x64_k32_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k32_dropout_sm80`，作为本文件可调用接口的一部分。
- L206: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L207: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L208: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L209: Declares function `fmha_cutlassB_bf16_aligned_64x64_k64_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k64_dropout_sm80`，作为本文件可调用接口的一部分。
- L210: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 211-240

```cpp
 211:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 128, 128, 128>::kNumThreads,
 212:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 128, 128, 128>::kMinBlocksPerSm)
 213: fmha_cutlassB_bf16_aligned_128x128_k128_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 128, 128, 128>::Params p);
 214: __global__ void __launch_bounds__(
 215:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 128>::kNumThreads,
 216:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 217: fmha_cutlassB_bf16_aligned_64x64_k128_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 128>::Params p);
 218: __global__ void __launch_bounds__(
 219:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 128, 64, 65536>::kNumThreads,
 220:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 128, 64, 65536>::kMinBlocksPerSm)
 221: fmha_cutlassB_bf16_aligned_128x64_k65536_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 128, 64, 65536>::Params p);
 222: __global__ void __launch_bounds__(
 223:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 65536>::kNumThreads,
 224:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 225: fmha_cutlassB_bf16_aligned_64x64_k65536_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 65536>::Params p);
 226: 
 227: template <typename T> void dispatch_cutlassB_bf16_sm80(T cb, int cc) {
 228:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32, true>(), fmha_cutlassB_bf16_aligned_64x64_k32_seqaligned_sm80);
 229:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 32>(), fmha_cutlassB_bf16_aligned_64x64_k32_sm80);
 230:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64, true>(), fmha_cutlassB_bf16_aligned_64x64_k64_seqaligned_sm80);
 231:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 64, 64, 64>(), fmha_cutlassB_bf16_aligned_64x64_k64_sm80);
 232:     if (cc == 86 || cc == 89) cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 64, 96>(), fmha_cutlassB_bf16_aligned_128x64_k96_sm80);
 233:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128, true>(), fmha_cutlassB_bf16_aligned_128x128_k128_seqaligned_sm80);
 234:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, true, 128, 128, 128>(), fmha_cutlassB_bf16_aligned_128x128_k128_sm80);
 235:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128, true>(), fmha_cutlassB_bf16_aligned_64x64_k128_seqaligned_sm80);
 236:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 128>(), fmha_cutlassB_bf16_aligned_64x64_k128_sm80);
 237:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 128, 64, 65536>(), fmha_cutlassB_bf16_aligned_128x64_k65536_sm80);
 238:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, false, false, 64, 64, 65536>(), fmha_cutlassB_bf16_aligned_64x64_k65536_sm80);
 239:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 32>(), fmha_cutlassB_bf16_aligned_64x64_k32_dropout_sm80);
 240:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 64, 64, 64>(), fmha_cutlassB_bf16_aligned_64x64_k64_dropout_sm80);
```
- L211: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L212: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L213: Declares function `fmha_cutlassB_bf16_aligned_128x128_k128_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_128x128_k128_dropout_sm80`，作为本文件可调用接口的一部分。
- L214: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L215: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L216: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L217: Declares function `fmha_cutlassB_bf16_aligned_64x64_k128_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k128_dropout_sm80`，作为本文件可调用接口的一部分。
- L218: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L219: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L220: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L221: Declares function `fmha_cutlassB_bf16_aligned_128x64_k65536_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_128x64_k65536_dropout_sm80`，作为本文件可调用接口的一部分。
- L222: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L223: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L224: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L225: Declares function `fmha_cutlassB_bf16_aligned_64x64_k65536_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_bf16_aligned_64x64_k65536_dropout_sm80`，作为本文件可调用接口的一部分。
- L227: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L228: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L229: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L230: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L231: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L232: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L233: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L234: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L235: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L236: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L237: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L238: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L239: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L240: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。

### Lines 241-270

```cpp
 241:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, true, 128, 128, 128>(), fmha_cutlassB_bf16_aligned_128x128_k128_dropout_sm80);
 242:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 128>(), fmha_cutlassB_bf16_aligned_64x64_k128_dropout_sm80);
 243:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 128, 64, 65536>(), fmha_cutlassB_bf16_aligned_128x64_k65536_dropout_sm80);
 244:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::bfloat16_t, true, true, false, 64, 64, 65536>(), fmha_cutlassB_bf16_aligned_64x64_k65536_dropout_sm80);
 245: }
 246: 
 247: // ======== f16 / sm80 ========
 248: __global__ void __launch_bounds__(
 249:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32, true>::kNumThreads,
 250:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32, true>::kMinBlocksPerSm)
 251: fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32, true>::Params p);
 252: __global__ void __launch_bounds__(
 253:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32>::kNumThreads,
 254:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32>::kMinBlocksPerSm)
 255: fmha_cutlassB_f16_aligned_64x64_k32_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32>::Params p);
 256: __global__ void __launch_bounds__(
 257:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64, true>::kNumThreads,
 258:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64, true>::kMinBlocksPerSm)
 259: fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64, true>::Params p);
 260: __global__ void __launch_bounds__(
 261:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64>::kNumThreads,
 262:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64>::kMinBlocksPerSm)
 263: fmha_cutlassB_f16_aligned_64x64_k64_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64>::Params p);
 264: __global__ void __launch_bounds__(
 265:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 64, 96>::kNumThreads,
 266:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 64, 96>::kMinBlocksPerSm)
 267: fmha_cutlassB_f16_aligned_128x64_k96_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 64, 96>::Params p);
 268: __global__ void __launch_bounds__(
 269:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128, true>::kNumThreads,
 270:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128, true>::kMinBlocksPerSm)
```
- L241: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L242: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L243: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L244: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L245: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L247: Documents the nearby logic: ======== f16 / sm80 ======== / 说明附近逻辑的作用：======== f16 / sm80 ========
- L248: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L249: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L250: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L251: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L252: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L253: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L254: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L255: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_sm80`，作为本文件可调用接口的一部分。
- L256: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L257: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L258: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L259: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L260: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L261: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L262: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L263: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_sm80`，作为本文件可调用接口的一部分。
- L264: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L265: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L266: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L267: Declares function `fmha_cutlassB_f16_aligned_128x64_k96_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k96_sm80`，作为本文件可调用接口的一部分。
- L268: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L269: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L270: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 271-300

```cpp
 271: fmha_cutlassB_f16_aligned_128x128_k128_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128, true>::Params p);
 272: __global__ void __launch_bounds__(
 273:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128>::kNumThreads,
 274:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128>::kMinBlocksPerSm)
 275: fmha_cutlassB_f16_aligned_128x128_k128_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128>::Params p);
 276: __global__ void __launch_bounds__(
 277:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128, true>::kNumThreads,
 278:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128, true>::kMinBlocksPerSm)
 279: fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128, true>::Params p);
 280: __global__ void __launch_bounds__(
 281:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128>::kNumThreads,
 282:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 283: fmha_cutlassB_f16_aligned_64x64_k128_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128>::Params p);
 284: __global__ void __launch_bounds__(
 285:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 128, 64, 65536>::kNumThreads,
 286:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 128, 64, 65536>::kMinBlocksPerSm)
 287: fmha_cutlassB_f16_aligned_128x64_k65536_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 128, 64, 65536>::Params p);
 288: __global__ void __launch_bounds__(
 289:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 65536>::kNumThreads,
 290:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 291: fmha_cutlassB_f16_aligned_64x64_k65536_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 65536>::Params p);
 292: __global__ void __launch_bounds__(
 293:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 32>::kNumThreads,
 294:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 32>::kMinBlocksPerSm)
 295: fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 32>::Params p);
 296: __global__ void __launch_bounds__(
 297:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 64>::kNumThreads,
 298:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 64>::kMinBlocksPerSm)
 299: fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 64>::Params p);
 300: __global__ void __launch_bounds__(
```
- L271: Declares function `fmha_cutlassB_f16_aligned_128x128_k128_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x128_k128_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L272: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L273: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L274: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L275: Declares function `fmha_cutlassB_f16_aligned_128x128_k128_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x128_k128_sm80`，作为本文件可调用接口的一部分。
- L276: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L277: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L278: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L279: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm80`，作为本文件可调用接口的一部分。
- L280: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L281: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L282: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L283: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_sm80`，作为本文件可调用接口的一部分。
- L284: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L285: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L286: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L287: Declares function `fmha_cutlassB_f16_aligned_128x64_k65536_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k65536_sm80`，作为本文件可调用接口的一部分。
- L288: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L289: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L290: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L291: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_sm80`，作为本文件可调用接口的一部分。
- L292: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L293: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L294: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L295: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm80`，作为本文件可调用接口的一部分。
- L296: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L297: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L298: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L299: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm80`，作为本文件可调用接口的一部分。
- L300: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 301-330

```cpp
 301:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 128, 128, 128>::kNumThreads,
 302:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 128, 128, 128>::kMinBlocksPerSm)
 303: fmha_cutlassB_f16_aligned_128x128_k128_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 128, 128, 128>::Params p);
 304: __global__ void __launch_bounds__(
 305:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 128>::kNumThreads,
 306:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 307: fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 128>::Params p);
 308: __global__ void __launch_bounds__(
 309:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 128, 64, 65536>::kNumThreads,
 310:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 128, 64, 65536>::kMinBlocksPerSm)
 311: fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 128, 64, 65536>::Params p);
 312: __global__ void __launch_bounds__(
 313:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 65536>::kNumThreads,
 314:     AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 315: fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 65536>::Params p);
 316: 
 317: template <typename T> void dispatch_cutlassB_f16_sm80(T cb, int cc) {
 318:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32, true>(), fmha_cutlassB_f16_aligned_64x64_k32_seqaligned_sm80);
 319:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_sm80);
 320:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64, true>(), fmha_cutlassB_f16_aligned_64x64_k64_seqaligned_sm80);
 321:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_sm80);
 322:     if (cc == 86 || cc == 89) cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 64, 96>(), fmha_cutlassB_f16_aligned_128x64_k96_sm80);
 323:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128, true>(), fmha_cutlassB_f16_aligned_128x128_k128_seqaligned_sm80);
 324:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, true, 128, 128, 128>(), fmha_cutlassB_f16_aligned_128x128_k128_sm80);
 325:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128, true>(), fmha_cutlassB_f16_aligned_64x64_k128_seqaligned_sm80);
 326:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_sm80);
 327:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 128, 64, 65536>(), fmha_cutlassB_f16_aligned_128x64_k65536_sm80);
 328:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_sm80);
 329:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm80);
 330:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm80);
```
- L301: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L302: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L303: Declares function `fmha_cutlassB_f16_aligned_128x128_k128_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x128_k128_dropout_sm80`，作为本文件可调用接口的一部分。
- L304: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L305: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L306: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L307: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm80`，作为本文件可调用接口的一部分。
- L308: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L309: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L310: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L311: Declares function `fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm80`，作为本文件可调用接口的一部分。
- L312: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L313: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L314: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L315: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm80`，作为本文件可调用接口的一部分。
- L317: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L318: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L319: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L320: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L321: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L322: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L323: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L324: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L325: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L326: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L327: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L328: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L329: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L330: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。

### Lines 331-360

```cpp
 331:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, true, 128, 128, 128>(), fmha_cutlassB_f16_aligned_128x128_k128_dropout_sm80);
 332:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm80);
 333:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 128, 64, 65536>(), fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm80);
 334:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, cutlass::half_t, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm80);
 335: }
 336: 
 337: // ======== f16 / sm50 ========
 338: __global__ void __launch_bounds__(
 339:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 32>::kNumThreads,
 340:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 32>::kMinBlocksPerSm)
 341: fmha_cutlassB_f16_aligned_64x64_k32_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 32>::Params p);
 342: __global__ void __launch_bounds__(
 343:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 64>::kNumThreads,
 344:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 64>::kMinBlocksPerSm)
 345: fmha_cutlassB_f16_aligned_64x64_k64_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 64>::Params p);
 346: __global__ void __launch_bounds__(
 347:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 128>::kNumThreads,
 348:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 349: fmha_cutlassB_f16_aligned_64x64_k128_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 128>::Params p);
 350: __global__ void __launch_bounds__(
 351:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 65536>::kNumThreads,
 352:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 353: fmha_cutlassB_f16_aligned_64x64_k65536_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 65536>::Params p);
 354: __global__ void __launch_bounds__(
 355:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 32>::kNumThreads,
 356:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 32>::kMinBlocksPerSm)
 357: fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 32>::Params p);
 358: __global__ void __launch_bounds__(
 359:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 64>::kNumThreads,
 360:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 64>::kMinBlocksPerSm)
```
- L331: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L332: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L333: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L334: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L335: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L337: Documents the nearby logic: ======== f16 / sm50 ======== / 说明附近逻辑的作用：======== f16 / sm50 ========
- L338: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L339: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L340: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L341: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_sm50`，作为本文件可调用接口的一部分。
- L342: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L343: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L344: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L345: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_sm50`，作为本文件可调用接口的一部分。
- L346: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L347: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L348: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L349: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_sm50`，作为本文件可调用接口的一部分。
- L350: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L351: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L352: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L353: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_sm50`，作为本文件可调用接口的一部分。
- L354: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L355: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L356: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L357: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm50`，作为本文件可调用接口的一部分。
- L358: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L359: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L360: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 361-390

```cpp
 361: fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 64>::Params p);
 362: __global__ void __launch_bounds__(
 363:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 128>::kNumThreads,
 364:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 365: fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 128>::Params p);
 366: __global__ void __launch_bounds__(
 367:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 65536>::kNumThreads,
 368:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 369: fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 65536>::Params p);
 370: __global__ void __launch_bounds__(
 371:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 32>::kNumThreads,
 372:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 32>::kMinBlocksPerSm)
 373: fmha_cutlassB_f16_notaligned_64x64_k32_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 32>::Params p);
 374: __global__ void __launch_bounds__(
 375:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 64>::kNumThreads,
 376:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 64>::kMinBlocksPerSm)
 377: fmha_cutlassB_f16_notaligned_64x64_k64_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 64>::Params p);
 378: __global__ void __launch_bounds__(
 379:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 128>::kNumThreads,
 380:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 128>::kMinBlocksPerSm)
 381: fmha_cutlassB_f16_notaligned_64x64_k128_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 128>::Params p);
 382: __global__ void __launch_bounds__(
 383:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 65536>::kNumThreads,
 384:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 385: fmha_cutlassB_f16_notaligned_64x64_k65536_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 65536>::Params p);
 386: __global__ void __launch_bounds__(
 387:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 32>::kNumThreads,
 388:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 32>::kMinBlocksPerSm)
 389: fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 32>::Params p);
 390: __global__ void __launch_bounds__(
```
- L361: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm50`，作为本文件可调用接口的一部分。
- L362: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L363: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L364: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L365: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm50`，作为本文件可调用接口的一部分。
- L366: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L367: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L368: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L369: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm50`，作为本文件可调用接口的一部分。
- L370: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L371: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L372: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L373: Declares function `fmha_cutlassB_f16_notaligned_64x64_k32_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k32_sm50`，作为本文件可调用接口的一部分。
- L374: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L375: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L376: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L377: Declares function `fmha_cutlassB_f16_notaligned_64x64_k64_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k64_sm50`，作为本文件可调用接口的一部分。
- L378: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L379: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L380: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L381: Declares function `fmha_cutlassB_f16_notaligned_64x64_k128_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k128_sm50`，作为本文件可调用接口的一部分。
- L382: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L383: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L384: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L385: Declares function `fmha_cutlassB_f16_notaligned_64x64_k65536_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k65536_sm50`，作为本文件可调用接口的一部分。
- L386: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L387: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L388: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L389: Declares function `fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm50`，作为本文件可调用接口的一部分。
- L390: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 391-420

```cpp
 391:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 64>::kNumThreads,
 392:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 64>::kMinBlocksPerSm)
 393: fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 64>::Params p);
 394: __global__ void __launch_bounds__(
 395:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 128>::kNumThreads,
 396:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 128>::kMinBlocksPerSm)
 397: fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 128>::Params p);
 398: __global__ void __launch_bounds__(
 399:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 65536>::kNumThreads,
 400:     AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 401: fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 65536>::Params p);
 402: 
 403: template <typename T> void dispatch_cutlassB_f16_sm50(T cb, int cc) {
 404:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_sm50);
 405:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_sm50);
 406:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_sm50);
 407:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_sm50);
 408:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm50);
 409:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm50);
 410:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm50);
 411:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm50);
 412:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 32>(), fmha_cutlassB_f16_notaligned_64x64_k32_sm50);
 413:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 64>(), fmha_cutlassB_f16_notaligned_64x64_k64_sm50);
 414:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 128>(), fmha_cutlassB_f16_notaligned_64x64_k128_sm50);
 415:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, false, false, 64, 64, 65536>(), fmha_cutlassB_f16_notaligned_64x64_k65536_sm50);
 416:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 32>(), fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm50);
 417:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 64>(), fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm50);
 418:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 128>(), fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm50);
 419:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, cutlass::half_t, false, true, false, 64, 64, 65536>(), fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm50);
 420: }
```
- L391: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L392: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L393: Declares function `fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm50`，作为本文件可调用接口的一部分。
- L394: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L395: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L396: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L397: Declares function `fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm50`，作为本文件可调用接口的一部分。
- L398: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L399: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L400: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L401: Declares function `fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm50`，作为本文件可调用接口的一部分。
- L403: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L404: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L405: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L406: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L407: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L408: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L409: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L410: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L411: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L412: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L413: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L414: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L415: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L416: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L417: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L418: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L419: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L420: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 421-450

```cpp
 421: 
 422: // ======== f32 / sm50 ========
 423: __global__ void __launch_bounds__(
 424:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 32>::kNumThreads,
 425:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 32>::kMinBlocksPerSm)
 426: fmha_cutlassB_f32_aligned_64x64_k32_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 32>::Params p);
 427: __global__ void __launch_bounds__(
 428:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 64>::kNumThreads,
 429:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 64>::kMinBlocksPerSm)
 430: fmha_cutlassB_f32_aligned_64x64_k64_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 64>::Params p);
 431: __global__ void __launch_bounds__(
 432:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 128>::kNumThreads,
 433:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 434: fmha_cutlassB_f32_aligned_64x64_k128_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 128>::Params p);
 435: __global__ void __launch_bounds__(
 436:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 65536>::kNumThreads,
 437:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 438: fmha_cutlassB_f32_aligned_64x64_k65536_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 65536>::Params p);
 439: __global__ void __launch_bounds__(
 440:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 32>::kNumThreads,
 441:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 32>::kMinBlocksPerSm)
 442: fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 32>::Params p);
 443: __global__ void __launch_bounds__(
 444:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 64>::kNumThreads,
 445:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 64>::kMinBlocksPerSm)
 446: fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 64>::Params p);
 447: __global__ void __launch_bounds__(
 448:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 128>::kNumThreads,
 449:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 450: fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 128>::Params p);
```
- L422: Documents the nearby logic: ======== f32 / sm50 ======== / 说明附近逻辑的作用：======== f32 / sm50 ========
- L423: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L424: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L425: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L426: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_sm50`，作为本文件可调用接口的一部分。
- L427: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L428: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L429: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L430: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_sm50`，作为本文件可调用接口的一部分。
- L431: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L432: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L433: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L434: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_sm50`，作为本文件可调用接口的一部分。
- L435: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L436: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L437: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L438: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_sm50`，作为本文件可调用接口的一部分。
- L439: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L440: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L441: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L442: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm50`，作为本文件可调用接口的一部分。
- L443: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L444: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L445: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L446: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm50`，作为本文件可调用接口的一部分。
- L447: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L448: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L449: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L450: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm50`，作为本文件可调用接口的一部分。

### Lines 451-480

```cpp
 451: __global__ void __launch_bounds__(
 452:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 65536>::kNumThreads,
 453:     AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 454: fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 65536>::Params p);
 455: __global__ void __launch_bounds__(
 456:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 32>::kNumThreads,
 457:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 32>::kMinBlocksPerSm)
 458: fmha_cutlassB_f32_notaligned_64x64_k32_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 32>::Params p);
 459: __global__ void __launch_bounds__(
 460:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 64>::kNumThreads,
 461:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 64>::kMinBlocksPerSm)
 462: fmha_cutlassB_f32_notaligned_64x64_k64_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 64>::Params p);
 463: __global__ void __launch_bounds__(
 464:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 128>::kNumThreads,
 465:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 128>::kMinBlocksPerSm)
 466: fmha_cutlassB_f32_notaligned_64x64_k128_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 128>::Params p);
 467: __global__ void __launch_bounds__(
 468:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 65536>::kNumThreads,
 469:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 470: fmha_cutlassB_f32_notaligned_64x64_k65536_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 65536>::Params p);
 471: __global__ void __launch_bounds__(
 472:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 32>::kNumThreads,
 473:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 32>::kMinBlocksPerSm)
 474: fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 32>::Params p);
 475: __global__ void __launch_bounds__(
 476:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 64>::kNumThreads,
 477:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 64>::kMinBlocksPerSm)
 478: fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 64>::Params p);
 479: __global__ void __launch_bounds__(
 480:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 128>::kNumThreads,
```
- L451: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L452: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L453: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L454: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm50`，作为本文件可调用接口的一部分。
- L455: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L456: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L457: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L458: Declares function `fmha_cutlassB_f32_notaligned_64x64_k32_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k32_sm50`，作为本文件可调用接口的一部分。
- L459: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L460: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L461: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L462: Declares function `fmha_cutlassB_f32_notaligned_64x64_k64_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k64_sm50`，作为本文件可调用接口的一部分。
- L463: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L464: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L465: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L466: Declares function `fmha_cutlassB_f32_notaligned_64x64_k128_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k128_sm50`，作为本文件可调用接口的一部分。
- L467: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L468: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L469: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L470: Declares function `fmha_cutlassB_f32_notaligned_64x64_k65536_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k65536_sm50`，作为本文件可调用接口的一部分。
- L471: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L472: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L473: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L474: Declares function `fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm50`，作为本文件可调用接口的一部分。
- L475: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L476: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L477: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L478: Declares function `fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm50`，作为本文件可调用接口的一部分。
- L479: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L480: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 481-510

```cpp
 481:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 128>::kMinBlocksPerSm)
 482: fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 128>::Params p);
 483: __global__ void __launch_bounds__(
 484:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 65536>::kNumThreads,
 485:     AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 486: fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm50(typename AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 65536>::Params p);
 487: 
 488: template <typename T> void dispatch_cutlassB_f32_sm50(T cb, int cc) {
 489:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_sm50);
 490:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_sm50);
 491:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_sm50);
 492:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_sm50);
 493:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm50);
 494:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm50);
 495:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm50);
 496:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm50);
 497:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 32>(), fmha_cutlassB_f32_notaligned_64x64_k32_sm50);
 498:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 64>(), fmha_cutlassB_f32_notaligned_64x64_k64_sm50);
 499:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 128>(), fmha_cutlassB_f32_notaligned_64x64_k128_sm50);
 500:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, false, false, 64, 64, 65536>(), fmha_cutlassB_f32_notaligned_64x64_k65536_sm50);
 501:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 32>(), fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm50);
 502:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 64>(), fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm50);
 503:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 128>(), fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm50);
 504:     cb(AttentionBackwardKernel<cutlass::arch::Sm50, float, false, true, false, 64, 64, 65536>(), fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm50);
 505: }
 506: 
 507: // ======== f32 / sm70 ========
 508: __global__ void __launch_bounds__(
 509:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 32>::kNumThreads,
 510:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 32>::kMinBlocksPerSm)
```
- L481: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L482: Declares function `fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm50`，作为本文件可调用接口的一部分。
- L483: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L484: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L485: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L486: Declares function `fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm50` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm50`，作为本文件可调用接口的一部分。
- L488: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L489: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L490: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L491: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L492: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L493: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L494: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L495: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L496: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L497: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L498: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L499: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L500: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L501: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L502: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L503: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L504: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L505: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L507: Documents the nearby logic: ======== f32 / sm70 ======== / 说明附近逻辑的作用：======== f32 / sm70 ========
- L508: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L509: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L510: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 511-540

```cpp
 511: fmha_cutlassB_f32_aligned_64x64_k32_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 32>::Params p);
 512: __global__ void __launch_bounds__(
 513:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 64>::kNumThreads,
 514:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 64>::kMinBlocksPerSm)
 515: fmha_cutlassB_f32_aligned_64x64_k64_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 64>::Params p);
 516: __global__ void __launch_bounds__(
 517:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 128>::kNumThreads,
 518:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 519: fmha_cutlassB_f32_aligned_64x64_k128_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 128>::Params p);
 520: __global__ void __launch_bounds__(
 521:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 65536>::kNumThreads,
 522:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 523: fmha_cutlassB_f32_aligned_64x64_k65536_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 65536>::Params p);
 524: __global__ void __launch_bounds__(
 525:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 32>::kNumThreads,
 526:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 32>::kMinBlocksPerSm)
 527: fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 32>::Params p);
 528: __global__ void __launch_bounds__(
 529:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 64>::kNumThreads,
 530:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 64>::kMinBlocksPerSm)
 531: fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 64>::Params p);
 532: __global__ void __launch_bounds__(
 533:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 128>::kNumThreads,
 534:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 535: fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 128>::Params p);
 536: __global__ void __launch_bounds__(
 537:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 65536>::kNumThreads,
 538:     AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 539: fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 65536>::Params p);
 540: __global__ void __launch_bounds__(
```
- L511: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_sm70`，作为本文件可调用接口的一部分。
- L512: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L513: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L514: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L515: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_sm70`，作为本文件可调用接口的一部分。
- L516: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L517: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L518: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L519: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_sm70`，作为本文件可调用接口的一部分。
- L520: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L521: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L522: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L523: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_sm70`，作为本文件可调用接口的一部分。
- L524: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L525: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L526: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L527: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm70`，作为本文件可调用接口的一部分。
- L528: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L529: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L530: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L531: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm70`，作为本文件可调用接口的一部分。
- L532: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L533: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L534: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L535: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm70`，作为本文件可调用接口的一部分。
- L536: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L537: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L538: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L539: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm70`，作为本文件可调用接口的一部分。
- L540: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 541-570

```cpp
 541:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 32>::kNumThreads,
 542:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 32>::kMinBlocksPerSm)
 543: fmha_cutlassB_f32_notaligned_64x64_k32_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 32>::Params p);
 544: __global__ void __launch_bounds__(
 545:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 64>::kNumThreads,
 546:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 64>::kMinBlocksPerSm)
 547: fmha_cutlassB_f32_notaligned_64x64_k64_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 64>::Params p);
 548: __global__ void __launch_bounds__(
 549:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 128>::kNumThreads,
 550:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 128>::kMinBlocksPerSm)
 551: fmha_cutlassB_f32_notaligned_64x64_k128_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 128>::Params p);
 552: __global__ void __launch_bounds__(
 553:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 65536>::kNumThreads,
 554:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 555: fmha_cutlassB_f32_notaligned_64x64_k65536_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 65536>::Params p);
 556: __global__ void __launch_bounds__(
 557:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 32>::kNumThreads,
 558:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 32>::kMinBlocksPerSm)
 559: fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 32>::Params p);
 560: __global__ void __launch_bounds__(
 561:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 64>::kNumThreads,
 562:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 64>::kMinBlocksPerSm)
 563: fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 64>::Params p);
 564: __global__ void __launch_bounds__(
 565:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 128>::kNumThreads,
 566:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 128>::kMinBlocksPerSm)
 567: fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 128>::Params p);
 568: __global__ void __launch_bounds__(
 569:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 65536>::kNumThreads,
 570:     AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 65536>::kMinBlocksPerSm)
```
- L541: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L542: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L543: Declares function `fmha_cutlassB_f32_notaligned_64x64_k32_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k32_sm70`，作为本文件可调用接口的一部分。
- L544: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L545: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L546: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L547: Declares function `fmha_cutlassB_f32_notaligned_64x64_k64_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k64_sm70`，作为本文件可调用接口的一部分。
- L548: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L549: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L550: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L551: Declares function `fmha_cutlassB_f32_notaligned_64x64_k128_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k128_sm70`，作为本文件可调用接口的一部分。
- L552: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L553: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L554: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L555: Declares function `fmha_cutlassB_f32_notaligned_64x64_k65536_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k65536_sm70`，作为本文件可调用接口的一部分。
- L556: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L557: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L558: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L559: Declares function `fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm70`，作为本文件可调用接口的一部分。
- L560: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L561: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L562: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L563: Declares function `fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm70`，作为本文件可调用接口的一部分。
- L564: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L565: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L566: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L567: Declares function `fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm70`，作为本文件可调用接口的一部分。
- L568: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L569: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L570: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 571-600

```cpp
 571: fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm70(typename AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 65536>::Params p);
 572: 
 573: template <typename T> void dispatch_cutlassB_f32_sm70(T cb, int cc) {
 574:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_sm70);
 575:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_sm70);
 576:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_sm70);
 577:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_sm70);
 578:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm70);
 579:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm70);
 580:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm70);
 581:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm70);
 582:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 32>(), fmha_cutlassB_f32_notaligned_64x64_k32_sm70);
 583:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 64>(), fmha_cutlassB_f32_notaligned_64x64_k64_sm70);
 584:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 128>(), fmha_cutlassB_f32_notaligned_64x64_k128_sm70);
 585:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, false, false, 64, 64, 65536>(), fmha_cutlassB_f32_notaligned_64x64_k65536_sm70);
 586:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 32>(), fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm70);
 587:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 64>(), fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm70);
 588:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 128>(), fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm70);
 589:     cb(AttentionBackwardKernel<cutlass::arch::Sm70, float, false, true, false, 64, 64, 65536>(), fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm70);
 590: }
 591: 
 592: // ======== f16 / sm75 ========
 593: __global__ void __launch_bounds__(
 594:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 32>::kNumThreads,
 595:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 32>::kMinBlocksPerSm)
 596: fmha_cutlassB_f16_aligned_64x64_k32_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 32>::Params p);
 597: __global__ void __launch_bounds__(
 598:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 64>::kNumThreads,
 599:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 64>::kMinBlocksPerSm)
 600: fmha_cutlassB_f16_aligned_64x64_k64_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 64>::Params p);
```
- L571: Declares function `fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm70` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm70`，作为本文件可调用接口的一部分。
- L573: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L574: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L575: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L576: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L577: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L578: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L579: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L580: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L581: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L582: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L583: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L584: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L585: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L586: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L587: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L588: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L589: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L590: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L592: Documents the nearby logic: ======== f16 / sm75 ======== / 说明附近逻辑的作用：======== f16 / sm75 ========
- L593: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L594: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L595: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L596: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_sm75`，作为本文件可调用接口的一部分。
- L597: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L598: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L599: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L600: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_sm75`，作为本文件可调用接口的一部分。

### Lines 601-630

```cpp
 601: __global__ void __launch_bounds__(
 602:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 128>::kNumThreads,
 603:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 128>::kMinBlocksPerSm)
 604: fmha_cutlassB_f16_aligned_128x64_k128_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 128>::Params p);
 605: __global__ void __launch_bounds__(
 606:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 128>::kNumThreads,
 607:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 608: fmha_cutlassB_f16_aligned_64x64_k128_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 128>::Params p);
 609: __global__ void __launch_bounds__(
 610:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 65536>::kNumThreads,
 611:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 65536>::kMinBlocksPerSm)
 612: fmha_cutlassB_f16_aligned_128x64_k65536_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 65536>::Params p);
 613: __global__ void __launch_bounds__(
 614:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 65536>::kNumThreads,
 615:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 616: fmha_cutlassB_f16_aligned_64x64_k65536_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 65536>::Params p);
 617: __global__ void __launch_bounds__(
 618:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 32>::kNumThreads,
 619:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 32>::kMinBlocksPerSm)
 620: fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 32>::Params p);
 621: __global__ void __launch_bounds__(
 622:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 64>::kNumThreads,
 623:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 64>::kMinBlocksPerSm)
 624: fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 64>::Params p);
 625: __global__ void __launch_bounds__(
 626:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 128>::kNumThreads,
 627:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 128>::kMinBlocksPerSm)
 628: fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 128>::Params p);
 629: __global__ void __launch_bounds__(
 630:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 128>::kNumThreads,
```
- L601: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L602: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L603: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L604: Declares function `fmha_cutlassB_f16_aligned_128x64_k128_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k128_sm75`，作为本文件可调用接口的一部分。
- L605: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L606: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L607: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L608: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_sm75`，作为本文件可调用接口的一部分。
- L609: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L610: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L611: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L612: Declares function `fmha_cutlassB_f16_aligned_128x64_k65536_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k65536_sm75`，作为本文件可调用接口的一部分。
- L613: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L614: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L615: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L616: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_sm75`，作为本文件可调用接口的一部分。
- L617: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L618: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L619: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L620: Declares function `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm75`，作为本文件可调用接口的一部分。
- L621: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L622: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L623: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L624: Declares function `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm75`，作为本文件可调用接口的一部分。
- L625: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L626: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L627: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L628: Declares function `fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm75`，作为本文件可调用接口的一部分。
- L629: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L630: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 631-660

```cpp
 631:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 632: fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 128>::Params p);
 633: __global__ void __launch_bounds__(
 634:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 65536>::kNumThreads,
 635:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 65536>::kMinBlocksPerSm)
 636: fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 65536>::Params p);
 637: __global__ void __launch_bounds__(
 638:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 65536>::kNumThreads,
 639:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 640: fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 65536>::Params p);
 641: __global__ void __launch_bounds__(
 642:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 32>::kNumThreads,
 643:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 32>::kMinBlocksPerSm)
 644: fmha_cutlassB_f16_notaligned_64x64_k32_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 32>::Params p);
 645: __global__ void __launch_bounds__(
 646:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 64>::kNumThreads,
 647:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 64>::kMinBlocksPerSm)
 648: fmha_cutlassB_f16_notaligned_64x64_k64_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 64>::Params p);
 649: __global__ void __launch_bounds__(
 650:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 128>::kNumThreads,
 651:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 128>::kMinBlocksPerSm)
 652: fmha_cutlassB_f16_notaligned_128x64_k128_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 128>::Params p);
 653: __global__ void __launch_bounds__(
 654:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 128>::kNumThreads,
 655:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 128>::kMinBlocksPerSm)
 656: fmha_cutlassB_f16_notaligned_64x64_k128_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 128>::Params p);
 657: __global__ void __launch_bounds__(
 658:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 65536>::kNumThreads,
 659:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 65536>::kMinBlocksPerSm)
 660: fmha_cutlassB_f16_notaligned_128x64_k65536_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 65536>::Params p);
```
- L631: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L632: Declares function `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm75`，作为本文件可调用接口的一部分。
- L633: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L634: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L635: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L636: Declares function `fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm75`，作为本文件可调用接口的一部分。
- L637: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L638: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L639: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L640: Declares function `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm75`，作为本文件可调用接口的一部分。
- L641: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L642: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L643: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L644: Declares function `fmha_cutlassB_f16_notaligned_64x64_k32_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k32_sm75`，作为本文件可调用接口的一部分。
- L645: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L646: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L647: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L648: Declares function `fmha_cutlassB_f16_notaligned_64x64_k64_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k64_sm75`，作为本文件可调用接口的一部分。
- L649: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L650: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L651: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L652: Declares function `fmha_cutlassB_f16_notaligned_128x64_k128_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k128_sm75`，作为本文件可调用接口的一部分。
- L653: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L654: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L655: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L656: Declares function `fmha_cutlassB_f16_notaligned_64x64_k128_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k128_sm75`，作为本文件可调用接口的一部分。
- L657: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L658: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L659: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L660: Declares function `fmha_cutlassB_f16_notaligned_128x64_k65536_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k65536_sm75`，作为本文件可调用接口的一部分。

### Lines 661-690

```cpp
 661: __global__ void __launch_bounds__(
 662:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 65536>::kNumThreads,
 663:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 664: fmha_cutlassB_f16_notaligned_64x64_k65536_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 65536>::Params p);
 665: __global__ void __launch_bounds__(
 666:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 32>::kNumThreads,
 667:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 32>::kMinBlocksPerSm)
 668: fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 32>::Params p);
 669: __global__ void __launch_bounds__(
 670:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 64>::kNumThreads,
 671:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 64>::kMinBlocksPerSm)
 672: fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 64>::Params p);
 673: __global__ void __launch_bounds__(
 674:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 128>::kNumThreads,
 675:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 128>::kMinBlocksPerSm)
 676: fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 128>::Params p);
 677: __global__ void __launch_bounds__(
 678:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 128>::kNumThreads,
 679:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 128>::kMinBlocksPerSm)
 680: fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 128>::Params p);
 681: __global__ void __launch_bounds__(
 682:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 65536>::kNumThreads,
 683:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 65536>::kMinBlocksPerSm)
 684: fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 65536>::Params p);
 685: __global__ void __launch_bounds__(
 686:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 65536>::kNumThreads,
 687:     AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 688: fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 65536>::Params p);
 689: 
 690: template <typename T> void dispatch_cutlassB_f16_sm75(T cb, int cc) {
```
- L661: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L662: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L663: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L664: Declares function `fmha_cutlassB_f16_notaligned_64x64_k65536_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k65536_sm75`，作为本文件可调用接口的一部分。
- L665: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L666: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L667: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L668: Declares function `fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm75`，作为本文件可调用接口的一部分。
- L669: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L670: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L671: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L672: Declares function `fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm75`，作为本文件可调用接口的一部分。
- L673: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L674: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L675: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L676: Declares function `fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm75`，作为本文件可调用接口的一部分。
- L677: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L678: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L679: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L680: Declares function `fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm75`，作为本文件可调用接口的一部分。
- L681: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L682: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L683: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L684: Declares function `fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm75`，作为本文件可调用接口的一部分。
- L685: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L686: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L687: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L688: Declares function `fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm75`，作为本文件可调用接口的一部分。
- L690: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。

### Lines 691-720

```cpp
 691:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_sm75);
 692:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_sm75);
 693:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 128>(), fmha_cutlassB_f16_aligned_128x64_k128_sm75);
 694:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_sm75);
 695:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 128, 64, 65536>(), fmha_cutlassB_f16_aligned_128x64_k65536_sm75);
 696:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_sm75);
 697:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 32>(), fmha_cutlassB_f16_aligned_64x64_k32_dropout_sm75);
 698:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 64>(), fmha_cutlassB_f16_aligned_64x64_k64_dropout_sm75);
 699:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 128>(), fmha_cutlassB_f16_aligned_128x64_k128_dropout_sm75);
 700:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 128>(), fmha_cutlassB_f16_aligned_64x64_k128_dropout_sm75);
 701:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 128, 64, 65536>(), fmha_cutlassB_f16_aligned_128x64_k65536_dropout_sm75);
 702:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f16_aligned_64x64_k65536_dropout_sm75);
 703:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 32>(), fmha_cutlassB_f16_notaligned_64x64_k32_sm75);
 704:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 64>(), fmha_cutlassB_f16_notaligned_64x64_k64_sm75);
 705:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 128>(), fmha_cutlassB_f16_notaligned_128x64_k128_sm75);
 706:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 128>(), fmha_cutlassB_f16_notaligned_64x64_k128_sm75);
 707:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 128, 64, 65536>(), fmha_cutlassB_f16_notaligned_128x64_k65536_sm75);
 708:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, false, false, 64, 64, 65536>(), fmha_cutlassB_f16_notaligned_64x64_k65536_sm75);
 709:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 32>(), fmha_cutlassB_f16_notaligned_64x64_k32_dropout_sm75);
 710:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 64>(), fmha_cutlassB_f16_notaligned_64x64_k64_dropout_sm75);
 711:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 128>(), fmha_cutlassB_f16_notaligned_128x64_k128_dropout_sm75);
 712:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 128>(), fmha_cutlassB_f16_notaligned_64x64_k128_dropout_sm75);
 713:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 128, 64, 65536>(), fmha_cutlassB_f16_notaligned_128x64_k65536_dropout_sm75);
 714:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, cutlass::half_t, false, true, false, 64, 64, 65536>(), fmha_cutlassB_f16_notaligned_64x64_k65536_dropout_sm75);
 715: }
 716: 
 717: // ======== f32 / sm75 ========
 718: __global__ void __launch_bounds__(
 719:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 32>::kNumThreads,
 720:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 32>::kMinBlocksPerSm)
```
- L691: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L692: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L693: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L694: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L695: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L696: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L697: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L698: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L699: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L700: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L701: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L702: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L703: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L704: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L705: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L706: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L707: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L708: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L709: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L710: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L711: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L712: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L713: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L714: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L715: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L717: Documents the nearby logic: ======== f32 / sm75 ======== / 说明附近逻辑的作用：======== f32 / sm75 ========
- L718: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L719: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L720: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 721-750

```cpp
 721: fmha_cutlassB_f32_aligned_64x64_k32_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 32>::Params p);
 722: __global__ void __launch_bounds__(
 723:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 64>::kNumThreads,
 724:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 64>::kMinBlocksPerSm)
 725: fmha_cutlassB_f32_aligned_64x64_k64_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 64>::Params p);
 726: __global__ void __launch_bounds__(
 727:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 128>::kNumThreads,
 728:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 729: fmha_cutlassB_f32_aligned_64x64_k128_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 128>::Params p);
 730: __global__ void __launch_bounds__(
 731:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 65536>::kNumThreads,
 732:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 733: fmha_cutlassB_f32_aligned_64x64_k65536_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 65536>::Params p);
 734: __global__ void __launch_bounds__(
 735:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 32>::kNumThreads,
 736:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 32>::kMinBlocksPerSm)
 737: fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 32>::Params p);
 738: __global__ void __launch_bounds__(
 739:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 64>::kNumThreads,
 740:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 64>::kMinBlocksPerSm)
 741: fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 64>::Params p);
 742: __global__ void __launch_bounds__(
 743:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 128>::kNumThreads,
 744:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 745: fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 128>::Params p);
 746: __global__ void __launch_bounds__(
 747:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 65536>::kNumThreads,
 748:     AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 749: fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 65536>::Params p);
 750: __global__ void __launch_bounds__(
```
- L721: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_sm75`，作为本文件可调用接口的一部分。
- L722: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L723: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L724: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L725: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_sm75`，作为本文件可调用接口的一部分。
- L726: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L727: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L728: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L729: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_sm75`，作为本文件可调用接口的一部分。
- L730: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L731: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L732: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L733: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_sm75`，作为本文件可调用接口的一部分。
- L734: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L735: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L736: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L737: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm75`，作为本文件可调用接口的一部分。
- L738: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L739: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L740: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L741: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm75`，作为本文件可调用接口的一部分。
- L742: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L743: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L744: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L745: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm75`，作为本文件可调用接口的一部分。
- L746: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L747: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L748: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L749: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm75`，作为本文件可调用接口的一部分。
- L750: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 751-780

```cpp
 751:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 32>::kNumThreads,
 752:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 32>::kMinBlocksPerSm)
 753: fmha_cutlassB_f32_notaligned_64x64_k32_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 32>::Params p);
 754: __global__ void __launch_bounds__(
 755:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 64>::kNumThreads,
 756:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 64>::kMinBlocksPerSm)
 757: fmha_cutlassB_f32_notaligned_64x64_k64_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 64>::Params p);
 758: __global__ void __launch_bounds__(
 759:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 128>::kNumThreads,
 760:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 128>::kMinBlocksPerSm)
 761: fmha_cutlassB_f32_notaligned_64x64_k128_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 128>::Params p);
 762: __global__ void __launch_bounds__(
 763:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 65536>::kNumThreads,
 764:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 765: fmha_cutlassB_f32_notaligned_64x64_k65536_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 65536>::Params p);
 766: __global__ void __launch_bounds__(
 767:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 32>::kNumThreads,
 768:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 32>::kMinBlocksPerSm)
 769: fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 32>::Params p);
 770: __global__ void __launch_bounds__(
 771:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 64>::kNumThreads,
 772:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 64>::kMinBlocksPerSm)
 773: fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 64>::Params p);
 774: __global__ void __launch_bounds__(
 775:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 128>::kNumThreads,
 776:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 128>::kMinBlocksPerSm)
 777: fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 128>::Params p);
 778: __global__ void __launch_bounds__(
 779:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 65536>::kNumThreads,
 780:     AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 65536>::kMinBlocksPerSm)
```
- L751: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L752: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L753: Declares function `fmha_cutlassB_f32_notaligned_64x64_k32_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k32_sm75`，作为本文件可调用接口的一部分。
- L754: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L755: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L756: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L757: Declares function `fmha_cutlassB_f32_notaligned_64x64_k64_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k64_sm75`，作为本文件可调用接口的一部分。
- L758: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L759: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L760: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L761: Declares function `fmha_cutlassB_f32_notaligned_64x64_k128_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k128_sm75`，作为本文件可调用接口的一部分。
- L762: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L763: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L764: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L765: Declares function `fmha_cutlassB_f32_notaligned_64x64_k65536_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k65536_sm75`，作为本文件可调用接口的一部分。
- L766: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L767: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L768: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L769: Declares function `fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm75`，作为本文件可调用接口的一部分。
- L770: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L771: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L772: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L773: Declares function `fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm75`，作为本文件可调用接口的一部分。
- L774: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L775: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L776: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L777: Declares function `fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm75`，作为本文件可调用接口的一部分。
- L778: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L779: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L780: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 781-810

```cpp
 781: fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm75(typename AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 65536>::Params p);
 782: 
 783: template <typename T> void dispatch_cutlassB_f32_sm75(T cb, int cc) {
 784:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_sm75);
 785:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_sm75);
 786:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_sm75);
 787:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_sm75);
 788:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm75);
 789:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm75);
 790:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm75);
 791:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm75);
 792:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 32>(), fmha_cutlassB_f32_notaligned_64x64_k32_sm75);
 793:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 64>(), fmha_cutlassB_f32_notaligned_64x64_k64_sm75);
 794:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 128>(), fmha_cutlassB_f32_notaligned_64x64_k128_sm75);
 795:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, false, false, 64, 64, 65536>(), fmha_cutlassB_f32_notaligned_64x64_k65536_sm75);
 796:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 32>(), fmha_cutlassB_f32_notaligned_64x64_k32_dropout_sm75);
 797:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 64>(), fmha_cutlassB_f32_notaligned_64x64_k64_dropout_sm75);
 798:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 128>(), fmha_cutlassB_f32_notaligned_64x64_k128_dropout_sm75);
 799:     cb(AttentionBackwardKernel<cutlass::arch::Sm75, float, false, true, false, 64, 64, 65536>(), fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm75);
 800: }
 801: 
 802: // ======== f32 / sm80 ========
 803: __global__ void __launch_bounds__(
 804:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 32>::kNumThreads,
 805:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 32>::kMinBlocksPerSm)
 806: fmha_cutlassB_f32_aligned_64x64_k32_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 32>::Params p);
 807: __global__ void __launch_bounds__(
 808:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 64>::kNumThreads,
 809:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 64>::kMinBlocksPerSm)
 810: fmha_cutlassB_f32_aligned_64x64_k64_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 64>::Params p);
```
- L781: Declares function `fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm75` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_notaligned_64x64_k65536_dropout_sm75`，作为本文件可调用接口的一部分。
- L783: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L784: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L785: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L786: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L787: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L788: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L789: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L790: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L791: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L792: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L793: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L794: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L795: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L796: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L797: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L798: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L799: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L800: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L802: Documents the nearby logic: ======== f32 / sm80 ======== / 说明附近逻辑的作用：======== f32 / sm80 ========
- L803: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L804: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L805: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L806: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_sm80`，作为本文件可调用接口的一部分。
- L807: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L808: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L809: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L810: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_sm80`，作为本文件可调用接口的一部分。

### Lines 811-840

```cpp
 811: __global__ void __launch_bounds__(
 812:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 128>::kNumThreads,
 813:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 128>::kMinBlocksPerSm)
 814: fmha_cutlassB_f32_aligned_128x64_k128_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 128>::Params p);
 815: __global__ void __launch_bounds__(
 816:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 128>::kNumThreads,
 817:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 128>::kMinBlocksPerSm)
 818: fmha_cutlassB_f32_aligned_64x64_k128_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 128>::Params p);
 819: __global__ void __launch_bounds__(
 820:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 65536>::kNumThreads,
 821:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 65536>::kMinBlocksPerSm)
 822: fmha_cutlassB_f32_aligned_128x64_k65536_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 65536>::Params p);
 823: __global__ void __launch_bounds__(
 824:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 65536>::kNumThreads,
 825:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 65536>::kMinBlocksPerSm)
 826: fmha_cutlassB_f32_aligned_64x64_k65536_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 65536>::Params p);
 827: __global__ void __launch_bounds__(
 828:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 32>::kNumThreads,
 829:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 32>::kMinBlocksPerSm)
 830: fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 32>::Params p);
 831: __global__ void __launch_bounds__(
 832:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 64>::kNumThreads,
 833:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 64>::kMinBlocksPerSm)
 834: fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 64>::Params p);
 835: __global__ void __launch_bounds__(
 836:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 128>::kNumThreads,
 837:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 128>::kMinBlocksPerSm)
 838: fmha_cutlassB_f32_aligned_128x64_k128_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 128>::Params p);
 839: __global__ void __launch_bounds__(
 840:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 128>::kNumThreads,
```
- L811: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L812: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L813: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L814: Declares function `fmha_cutlassB_f32_aligned_128x64_k128_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_128x64_k128_sm80`，作为本文件可调用接口的一部分。
- L815: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L816: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L817: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L818: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_sm80`，作为本文件可调用接口的一部分。
- L819: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L820: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L821: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L822: Declares function `fmha_cutlassB_f32_aligned_128x64_k65536_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_128x64_k65536_sm80`，作为本文件可调用接口的一部分。
- L823: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L824: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L825: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L826: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_sm80`，作为本文件可调用接口的一部分。
- L827: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L828: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L829: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L830: Declares function `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm80`，作为本文件可调用接口的一部分。
- L831: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L832: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L833: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L834: Declares function `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm80`，作为本文件可调用接口的一部分。
- L835: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L836: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L837: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L838: Declares function `fmha_cutlassB_f32_aligned_128x64_k128_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_128x64_k128_dropout_sm80`，作为本文件可调用接口的一部分。
- L839: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L840: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 841-870

```cpp
 841:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 128>::kMinBlocksPerSm)
 842: fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 128>::Params p);
 843: __global__ void __launch_bounds__(
 844:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 65536>::kNumThreads,
 845:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 65536>::kMinBlocksPerSm)
 846: fmha_cutlassB_f32_aligned_128x64_k65536_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 65536>::Params p);
 847: __global__ void __launch_bounds__(
 848:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 65536>::kNumThreads,
 849:     AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 65536>::kMinBlocksPerSm)
 850: fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm80(typename AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 65536>::Params p);
 851: 
 852: template <typename T> void dispatch_cutlassB_f32_sm80(T cb, int cc) {
 853:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_sm80);
 854:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_sm80);
 855:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 128>(), fmha_cutlassB_f32_aligned_128x64_k128_sm80);
 856:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_sm80);
 857:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 128, 64, 65536>(), fmha_cutlassB_f32_aligned_128x64_k65536_sm80);
 858:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, false, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_sm80);
 859:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 32>(), fmha_cutlassB_f32_aligned_64x64_k32_dropout_sm80);
 860:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 64>(), fmha_cutlassB_f32_aligned_64x64_k64_dropout_sm80);
 861:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 128>(), fmha_cutlassB_f32_aligned_128x64_k128_dropout_sm80);
 862:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 128>(), fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm80);
 863:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 128, 64, 65536>(), fmha_cutlassB_f32_aligned_128x64_k65536_dropout_sm80);
 864:     cb(AttentionBackwardKernel<cutlass::arch::Sm80, float, true, true, false, 64, 64, 65536>(), fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm80);
 865: }
 866: 
 867: 
 868: template <typename DT, typename T>
 869: void dispatch_cutlassB(T cb, int cc = 0) {
 870: 
```
- L841: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L842: Declares function `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k128_dropout_sm80`，作为本文件可调用接口的一部分。
- L843: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L844: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L845: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L846: Declares function `fmha_cutlassB_f32_aligned_128x64_k65536_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_128x64_k65536_dropout_sm80`，作为本文件可调用接口的一部分。
- L847: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L848: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L849: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L850: Declares function `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm80` as part of this file's callable surface. / 声明函数 `fmha_cutlassB_f32_aligned_64x64_k65536_dropout_sm80`，作为本文件可调用接口的一部分。
- L852: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L853: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L854: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L855: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L856: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L857: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L858: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L859: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L860: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L861: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L862: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L863: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L864: Declares function `cb` as part of this file's callable surface. / 声明函数 `cb`，作为本文件可调用接口的一部分。
- L865: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L868: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L869: Defines function `dispatch_cutlassB` and begins its implementation body. / 定义函数 `dispatch_cutlassB`，并开始其实现体。

### Lines 871-898

```cpp
 871:     if (std::is_same_v<DT, cutlass::half_t> && 70 <= cc && cc <= 74) {
 872:         dispatch_cutlassB_f16_sm70(cb, cc);
 873:     }
 874:     if (std::is_same_v<DT, cutlass::bfloat16_t> && 80 <= cc && cc <= 121) {
 875:         dispatch_cutlassB_bf16_sm80(cb, cc);
 876:     }
 877:     if (std::is_same_v<DT, cutlass::half_t> && 80 <= cc && cc <= 121) {
 878:         dispatch_cutlassB_f16_sm80(cb, cc);
 879:     }
 880:     if (std::is_same_v<DT, cutlass::half_t> && 50 <= cc && cc <= 69) {
 881:         dispatch_cutlassB_f16_sm50(cb, cc);
 882:     }
 883:     if (std::is_same_v<DT, float> && 50 <= cc && cc <= 69) {
 884:         dispatch_cutlassB_f32_sm50(cb, cc);
 885:     }
 886:     if (std::is_same_v<DT, float> && 70 <= cc && cc <= 74) {
 887:         dispatch_cutlassB_f32_sm70(cb, cc);
 888:     }
 889:     if (std::is_same_v<DT, cutlass::half_t> && 75 <= cc && cc <= 79) {
 890:         dispatch_cutlassB_f16_sm75(cb, cc);
 891:     }
 892:     if (std::is_same_v<DT, float> && 75 <= cc && cc <= 79) {
 893:         dispatch_cutlassB_f32_sm75(cb, cc);
 894:     }
 895:     if (std::is_same_v<DT, float> && 80 <= cc && cc <= 121) {
 896:         dispatch_cutlassB_f32_sm80(cb, cc);
 897:     }
 898: }
```
- L871: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L872: Declares function `dispatch_cutlassB_f16_sm70` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f16_sm70`，作为本文件可调用接口的一部分。
- L873: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L874: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L875: Declares function `dispatch_cutlassB_bf16_sm80` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_bf16_sm80`，作为本文件可调用接口的一部分。
- L876: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L877: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L878: Declares function `dispatch_cutlassB_f16_sm80` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f16_sm80`，作为本文件可调用接口的一部分。
- L879: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L880: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L881: Declares function `dispatch_cutlassB_f16_sm50` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f16_sm50`，作为本文件可调用接口的一部分。
- L882: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L883: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L884: Declares function `dispatch_cutlassB_f32_sm50` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f32_sm50`，作为本文件可调用接口的一部分。
- L885: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L886: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L887: Declares function `dispatch_cutlassB_f32_sm70` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f32_sm70`，作为本文件可调用接口的一部分。
- L888: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L889: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L890: Declares function `dispatch_cutlassB_f16_sm75` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f16_sm75`，作为本文件可调用接口的一部分。
- L891: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L892: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L893: Declares function `dispatch_cutlassB_f32_sm75` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f32_sm75`，作为本文件可调用接口的一部分。
- L894: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L895: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L896: Declares function `dispatch_cutlassB_f32_sm80` as part of this file's callable surface. / 声明函数 `dispatch_cutlassB_f32_sm80`，作为本文件可调用接口的一部分。
- L897: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L898: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

## Key Concepts / 关键概念

- Memory-efficient attention CUDA specialization / 高效注意力 CUDA 特化实现
- CUDA transformer kernels and dispatch / CUDA Transformer 内核与分发
- Transformer attention operators and helpers / Transformer 注意力算子与辅助逻辑
- Attention score computation and masking / 注意力分数计算与掩码处理
- Dispatch stubs and backend selection / 分发桩与后端选择
- CUDA execution and specialization / CUDA 执行与特化

## Dependencies / 依赖关系

- `ATen/native/transformers/cuda/mem_eff_attention/kernel_backward.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- Subsystem tie-in: transformer attention, masking, scaling, and backend-specific fused kernels. / 子系统关联：Transformer 注意力、掩码、缩放以及后端特化的融合内核。
