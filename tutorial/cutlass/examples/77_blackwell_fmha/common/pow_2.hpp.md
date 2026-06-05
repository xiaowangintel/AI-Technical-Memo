# pow_2.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/77_blackwell_fmha/common/pow_2.hpp`  
**Purpose / 用途**: Defines a tiny power-of-two helper type used in FMHA shape math so multiplications, divisions, and modulus operations can be expressed cheaply and in a CuTe-friendly way. / 定义一个小型 2 的幂辅助类型，用于 FMHA 的 shape 计算，使乘法、除法和取模能以更便宜且对 CuTe 友好的方式表达。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include <cute/config.hpp>
#include <cute/numeric/integral_constant.hpp>

#include <cuda_runtime.h>

namespace cutlass::fmha {
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
struct Pow2 {                                                                   
  int n;                                                                        
  int log2_n;                                                                   
                                                                                
  explicit CUTE_DEVICE Pow2(int n) : n(n) {
#ifdef __CUDA_ARCH__
    log2_n = __ffs(n) - 1;
#endif
  }                    
                                                                                
  template<class T>  
  CUTE_HOST_DEVICE T operator *(T const& b) const {
    return n * b;
  }

  template<int N>
  CUTE_HOST_DEVICE auto operator *(Int<N> const&) const {
    if constexpr (N & (N - 1) == 0) {
      return Pow2{n * N};
    }
    return n * N;
  }

};                                                                              

template<class T>
CUTE_HOST_DEVICE auto operator/(T const& a, Pow2 const& b) {
  return a >> b.log2_n;
}

template<class T>
CUTE_HOST_DEVICE auto operator%(T const& a, Pow2 const& b) {
  return a & (b.n - 1);
}

template<class T>
CUTE_HOST_DEVICE bool operator<(T const& a, Pow2 const& b) {
  return a < b.n;
}
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

```cpp
CUTE_HOST_DEVICE void print(Pow2 const& a) {
  printf("2^%d", a.log2_n);
}

} // end namespace cutlass::fmha

namespace cute {

template <>
struct is_integral<cutlass::fmha::Pow2> : true_type {};

} // end namespace cute
```

**EN**: This code block advances the file's control flow or type system in a focused step. In CUTLASS examples, even seemingly local code usually exists to reconcile runtime problem metadata with a heavily templated compile-time kernel configuration. Belongs to a fused-attention pipeline, so problem shape, sequence handling, and softmax state are central.  
**CN**: 这一代码块以一个较集中的步骤推进文件的控制流或类型系统。在 CUTLASS 示例中，即便看似局部的代码，通常也是为了把运行期问题元数据和高度模板化的编译期内核配置对齐起来。属于融合注意力流水线，因此问题形状、序列处理和 softmax 状态是核心。

---

## Key Concepts / 关键概念

- Architecture-tagged specialization (`Sm100`/`Sm103`/`Sm120`) / 基于架构标签的特化（`Sm100`/`Sm103`/`Sm120`）
- Warp-specialized FMHA layering across collective, kernel, and device wrappers / 跨 collective、kernel 与 device wrapper 的 warp-specialized FMHA 分层

## Dependencies / 依赖项

- `cute/config.hpp` — CuTe tensor/layout support / CuTe 张量/布局支持
- `cute/numeric/integral_constant.hpp` — CuTe tensor/layout support / CuTe 张量/布局支持
- `cuda_runtime.h` — included dependency used by this example / 此示例使用的包含依赖
