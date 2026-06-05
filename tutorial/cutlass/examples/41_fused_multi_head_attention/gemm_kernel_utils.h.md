# gemm_kernel_utils.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/gemm_kernel_utils.h`
**Purpose / 用途**: Shared utility layer for FMHA GEMM dispatch. It provides launch-time validation macros, architecture/type dispatch helpers, small integer utilities, default GEMM-type selection, and warp-uniform helpers. / 这是 FMHA GEMM 派发使用的公共工具层，提供启动期校验宏、架构/类型派发辅助、整数小工具、默认 GEMM 类型选择以及 warp-uniform 辅助函数。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1-36 / 第 1-36 行
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

#pragma once

#include "cutlass/arch/mma.h"

////////////////////////////////////////////////////////////////////////////////
```
**EN**: License, include, and namespace setup.
**CN**: 许可证、头文件以及命名空间准备。

### Lines 37-131 / 第 37-131 行
```cpp
// Some helper functions
////////////////////////////////////////////////////////////////////////////////
#define DISPATCH_TYPES(tensor, func)                                           \
  {                                                                            \
    if (query.scalar_type() == at::ScalarType::Float) {                        \
      using scalar_t = float;                                                  \
      func();                                                                  \
    } else if (query.scalar_type() == at::ScalarType::Half) {                  \
      using scalar_t = cutlass::half_t;                                        \
      func();                                                                  \
    } else if (query.scalar_type() == at::ScalarType::BFloat16) {              \
      using scalar_t = cutlass::bfloat16_t;                                    \
      func();                                                                  \
    } else {                                                                   \
      XFORMERS_CHECK(false, "Only fp32, half & bf16 supported at the moment"); \
    }                                                                          \
  }

#define DISPATCH_BOOL(BOOL_V, BOOL_NAME, F) \
  {                                         \
    if (BOOL_V) {                           \
      using BOOL_NAME = std::true_type;      \
      F();                                  \
    } else {                                \
      using BOOL_NAME = std::false_type;      \
      F();                                  \
    }                                       \
  }

#define DISPATCH_ARCHTAG(CC, func)                                        \
  {                                                                       \
    if (CC >= 80) {                                                       \
      using ArchTag = cutlass::arch::Sm80;                                \
      func();                                                             \
    } else if (CC >= 75) {                                                \
      using ArchTag = cutlass::arch::Sm75;                                \
      func();                                                             \
    } else if (CC >= 70) {                                                \
      using ArchTag = cutlass::arch::Sm70;                                \
      func();                                                             \
    } else if (CC >= 50) {                                                \
      using ArchTag = cutlass::arch::Sm50;                                \
      func();                                                             \
    } else {                                                              \
      XFORMERS_CHECK(                                                     \
          false,                                                          \
          "Your device is too old. We require compute capability >= 50"); \
    }                                                                     \
  }

#define CHECK_NOSPARSE_CONTIGUOUS_CUDA(TENSOR)                            \
  XFORMERS_CHECK(TENSOR.is_cuda(), #TENSOR " must be a CUDA tensor");     \
  XFORMERS_CHECK(!TENSOR.is_sparse(), #TENSOR " must be a dense tensor"); \
  XFORMERS_CHECK(TENSOR.is_contiguous());

#define CHECK_NOSPARSE_LASTCONTIGUOUS_CUDA(TENSOR)                        \
  XFORMERS_CHECK(TENSOR.is_cuda(), #TENSOR " must be a CUDA tensor");     \
  XFORMERS_CHECK(!TENSOR.is_sparse(), #TENSOR " must be a dense tensor"); \
  XFORMERS_CHECK(                                                         \
      TENSOR.stride(-1) == 1, #TENSOR ": last dimension must be contiguous");

#ifdef TORCH_CHECK
#define CHECK_ALIGNED_PTR(PTR, ALIGNMENT) \
  XFORMERS_CHECK(                         \
      uint64_t(PTR) % ALIGNMENT == 0, #PTR " is not correctly aligned")
#define XFORMERS_CHECK TORCH_CHECK
#elif defined(__CUDACC_RTC__)
#define CHECK_ALIGNED_PTR(PTR, ALIGNMENT)  \
  if (!(uint64_t(PTR) % ALIGNMENT == 0)) { \
    return false;                          \
  }
#define XFORMERS_CHECK(COND, ERR) \
  if (!(COND)) {                  \
    return false;                 \
  }
#else
#include <iostream>
#define CHECK_ALIGNED_PTR(PTR, ALIGNMENT)            \
  if (!(uint64_t(PTR) % ALIGNMENT == 0)) {           \
    std::cerr << #PTR " is not correctly aligned\n"; \
    return false;                                    \
  }
#define XFORMERS_CHECK(COND, ERR)                       \
  if (!(COND)) {                                        \
    std::cerr << "'" #COND "' failed: " << ERR << "\n"; \
    return false;                                       \
  }
#endif

#define ASSIGN_CHECK_OVERFLOW(A, B)                                    \
  {                                                                    \
    A = B;                                                             \
    XFORMERS_CHECK(                                                    \
        B < std::numeric_limits<decltype(A)>::max(), #B " overflows"); \
  }
```
**EN**: Dispatch and validation macros for datatype selection, architecture selection, tensor checks, alignment checks, and overflow checks.
**CN**: 类型派发、架构派发、张量检查、对齐检查与溢出检查等宏。

### Lines 132-144 / 第 132-144 行
```cpp

namespace gemm_kernel_utils {

template <typename integer>
constexpr CUTLASS_HOST_DEVICE integer ceil_div(integer n, integer m) {
  return (n + m - 1) / m;
}

template <typename integer>
constexpr CUTLASS_HOST_DEVICE integer align_up(integer n, integer m) {
  return ((n + m - 1) / m) * m;
}
```
**EN**: `ceil_div` and `align_up` constexpr helpers.
**CN**: `ceil_div` 与 `align_up` 两个 constexpr 小工具。

### Lines 145-201 / 第 145-201 行
```cpp
////////////////////////////////////////////////////////////////////////////////
// Determine the type of GEMM we do (TensorCores or not, Shapes ...)
// TODO: Maybe we could rely on Cutlass's DefaultGemm templates
////////////////////////////////////////////////////////////////////////////////

// Fallback to Simt (FMA on cuda cores) if not in a special case below
template <typename ArchTag, typename scalar_t_, typename Enable = void>
struct DefaultGemmType {
  static constexpr int ThreadK = 8;
  static constexpr int WarpK = 8;
  static constexpr int kMinimumAlignment = 1;
  using InstructionShape = cutlass::gemm::GemmShape<1, 1, 1>;
  using OpClass = cutlass::arch::OpClassSimt;
  using Operator = cutlass::arch::OpMultiplyAdd;
};

// Specialization for tensorcores with f32
template <typename ArchTag>
struct DefaultGemmType<
    ArchTag,
    float,
    typename cutlass::platform::enable_if<
        ArchTag::kMinComputeCapability >= 80>::type> {
  static constexpr int ThreadK = 32;
  static constexpr int WarpK = 32;
  static constexpr int kMinimumAlignment = 4;
  using OpClass = cutlass::arch::OpClassTensorOp;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 8>;
  using Operator = cutlass::arch::OpMultiplyAddFastF32;
};

// Specialization for tensorcores with f16/bf16 - Sm75+
template <typename ArchTag, typename scalar_t>
struct DefaultGemmType<
    ArchTag,
    scalar_t,
    typename cutlass::platform::enable_if<
        ArchTag::kMinComputeCapability >= 75 &&
        cutlass::sizeof_bits<scalar_t>::value == 16>::type> {
  static constexpr int ThreadK = 32;
  static constexpr int WarpK = 32;
  static constexpr int kMinimumAlignment = 4;
  using OpClass = cutlass::arch::OpClassTensorOp;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 8>;
  using Operator = cutlass::arch::OpMultiplyAdd;
};

// Specialization for tensorcores with f16 - Volta
template <>
struct DefaultGemmType<cutlass::arch::Sm70, cutlass::half_t, void> {
  static constexpr int ThreadK = 32;
  static constexpr int WarpK = 32;
  static constexpr int kMinimumAlignment = 2;
  using OpClass = cutlass::arch::OpClassTensorOp;
  using InstructionShape = cutlass::gemm::GemmShape<8, 8, 4>;
  using Operator = cutlass::arch::OpMultiplyAdd;
};
```
**EN**: `DefaultGemmType` primary template and architecture/datatype specializations for SIMT, fp32 tensor cores, fp16/bf16 tensor cores, and Volta fp16 tensor cores.
**CN**: `DefaultGemmType` 主模板，以及针对 SIMT、fp32 Tensor Core、fp16/bf16 Tensor Core、Volta fp16 Tensor Core 的特化。

### Lines 202-225 / 第 202-225 行
```cpp

// Enables to do
// `auto x = kCondition ? fa(arg) : fb(arg)`
// when `fa` and `fb` have different types
template <bool kVal, typename TA, typename TB>
struct call_conditional;

template <typename TA, typename TB>
struct call_conditional<true, TA, TB> {
  template <typename Arg>
  static CUTLASS_HOST_DEVICE auto apply(TA ta, TB tb, Arg arg)
      -> decltype(ta(arg)) {
    return ta(arg);
  }
};

template <typename TA, typename TB>
struct call_conditional<false, TA, TB> {
  template <typename Arg>
  static CUTLASS_HOST_DEVICE auto apply(TA ta, TB tb, Arg arg)
      -> decltype(tb(arg)) {
    return tb(arg);
  }
};
```
**EN**: `call_conditional` helper for compile-time branch selection with different callable types.
**CN**: `call_conditional` 辅助模板，用于不同可调用类型之间的编译期分支选择。

### Lines 226-257 / 第 226-257 行
```cpp

////////////////////////////////////////////////////////////////////////////////
// Mark a variable as warp-uniform - enables some compiler optimizations
// The cheapest way to do it is just to broadcast it from lane 0
////////////////////////////////////////////////////////////////////////////////

template <typename T>
CUTLASS_DEVICE T warp_uniform(T value) {
  struct {
    union {
      T value;
      uint32_t asInt;
    };
  } p;
  p.value = value;
  p.asInt = __shfl_sync(0xffffffff, (unsigned)p.asInt, 0);
  return p.value;
}

template <typename T>
CUTLASS_DEVICE T* warp_uniform(T* ptr) {
  struct {
    union {
      T* ptr;
      uint32_t asInt[2];
    };
  } p;
  p.ptr = ptr;
  p.asInt[0] = warp_uniform(p.asInt[0]);
  p.asInt[1] = warp_uniform(p.asInt[1]);
  return p.ptr;
}
```
**EN**: `warp_uniform` helpers for scalars and pointers.
**CN**: 标量与指针版本的 `warp_uniform` 辅助函数。

### Lines 258-258 / 第 258-258 行
```cpp
} // namespace gemm_kernel_utils
```
**EN**: Namespace closure.
**CN**: 命名空间收尾。

---
## Key Concepts / 关键概念
- **EN:** Grouped FMHA relies on aggressive template dispatch because the best kernel depends on datatype, alignment, and GPU generation.
  **CN:** grouped FMHA 高度依赖模板派发，因为最佳内核实现取决于数据类型、对齐状态以及 GPU 架构代际。
- **EN:** `DefaultGemmType` is especially important: it decides whether FMHA can use tensor cores and what instruction shape the back-to-back GEMMs should target.
  **CN:** `DefaultGemmType` 尤其关键：它决定 FMHA 能否使用 Tensor Core，以及背靠背 GEMM 应该选择何种指令形状。
- **EN:** `call_conditional` is useful inside fused kernels because output iterators or epilogue destinations may have different static types for first/last/intermediate iterations.
  **CN:** `call_conditional` 在融合内核中很有用，因为首块/尾块/中间块的输出 iterator 或 epilogue 目标常常具有不同的静态类型。
- **EN:** `warp_uniform` helps CUTLASS-generated code stay efficient by making per-warp invariants explicit.
  **CN:** `warp_uniform` 通过显式声明“warp 内不变”的值，帮助 CUTLASS 生成的代码保持高效。
- **EN:** `default_fmha_grouped.h` and `fmha_grouped.h` depend on this file for almost every compile-time dispatch decision and many host-side validation checks.
  **CN:** `default_fmha_grouped.h` 与 `fmha_grouped.h` 在几乎所有编译期派发决策以及大量主机侧校验上都依赖本文件。
- **EN:** It is the infrastructure layer that turns one FMHA algorithm description into architecture-specific CUTLASS kernels.
  **CN:** 它是把“一个 FMHA 算法描述”转化为“面向特定架构的 CUTLASS 内核”的基础设施层。
## Dependencies / 依赖项
- `"cutlass/arch/mma.h"` — architecture-specific MMA operator traits / 架构相关的 MMA 算子 traits
- `<iostream>` — standard C++ stream support used by host-side checks/debug code / 主机侧检查 / 调试代码使用的标准 C++ 流支持
