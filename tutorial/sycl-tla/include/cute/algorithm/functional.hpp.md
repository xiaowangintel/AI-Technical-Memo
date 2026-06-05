# functional.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/algorithm/functional.hpp`
- **EN:** Defines reusable function objects and predicates used by CuTe algorithms.
- **CN:** 定义 CuTe 算法复用的函数对象与谓词。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * Copyright (C) 2025 Intel Corporation, All rights reserved.
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
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 34-36
```cpp
#include <cute/config.hpp>          // CUTE_HOST_DEVICE
#include <cute/numeric/math.hpp>    // cute::max, cute::min
#include <cute/numeric/complex.hpp> // cute::conj
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/numeric/math.hpp` (related definitions from `cute/numeric/math.hpp`); `cute/numeric/complex.hpp` (related definitions from `cute/numeric/complex.hpp`).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/numeric/math.hpp`（来自 `cute/numeric/math.hpp` 的相关定义）；`cute/numeric/complex.hpp`（来自 `cute/numeric/complex.hpp` 的相关定义）。

### Lines 38-292
```cpp
/** C++14 <functional> extensions */

namespace cute {

/**************/
/** Identity **/
/**************/

struct identity {
  template <class T>
  CUTE_HOST_DEVICE constexpr
  decltype(auto) operator()(T&& arg) const {
    return static_cast<T&&>(arg);
  }
};

template <class R>
struct constant_fn {
  template <class... T>
  CUTE_HOST_DEVICE constexpr
  decltype(auto) operator()(T&&...) const {
    return r_;
  }
  R r_;
};

/***********/
/** Unary **/
/***********/

#define CUTE_LEFT_UNARY_OP(NAME,OP)                                  \
  struct NAME {                                                      \
    template <class T>                                               \
    CUTE_HOST_DEVICE constexpr                                       \
    decltype(auto) operator()(T&& arg) const {                       \
      return OP static_cast<T&&>(arg);                                \
    }                                                                \
  }
#define CUTE_RIGHT_UNARY_OP(NAME,OP)                                 \
  struct NAME {                                                      \
    template <class T>                                               \
    CUTE_HOST_DEVICE constexpr                                       \
    decltype(auto) operator()(T&& arg) const {                       \
      return static_cast<T&&>(arg) OP ;                               \
    }                                                                \
  }
#define CUTE_NAMED_UNARY_OP(NAME,OP)                                 \
  struct NAME {                                                      \
    template <class T>                                               \
    CUTE_HOST_DEVICE constexpr                                       \
    decltype(auto) operator()(T&& arg) const {                       \
      return OP (static_cast<T&&>(arg));                              \
    }                                                                \
  }

CUTE_LEFT_UNARY_OP(unary_plus,       +);
CUTE_LEFT_UNARY_OP(negate,           -);
CUTE_LEFT_UNARY_OP(bit_not,          ~);
CUTE_LEFT_UNARY_OP(logical_not,      !);
CUTE_LEFT_UNARY_OP(dereference,      *);
CUTE_LEFT_UNARY_OP(address_of,       &);
CUTE_LEFT_UNARY_OP(pre_increment,   ++);
CUTE_LEFT_UNARY_OP(pre_decrement,   --);

CUTE_RIGHT_UNARY_OP(post_increment, ++);
CUTE_RIGHT_UNARY_OP(post_decrement, --);

CUTE_NAMED_UNARY_OP(abs_fn,           abs);
CUTE_NAMED_UNARY_OP(conjugate, cute::conj);

#undef CUTE_LEFT_UNARY_OP
#undef CUTE_RIGHT_UNARY_OP
#undef CUTE_NAMED_UNARY_OP

template <int Shift_>
struct shift_right_const {
  static constexpr int Shift = Shift_;

  template <class T>
  CUTE_HOST_DEVICE constexpr
  decltype(auto) operator()(T&& arg) const {
    return static_cast<T&&>(arg) >> Shift;
  }
};

template <int Shift_>
struct shift_left_const {
  static constexpr int Shift = Shift_;

  template <class T>
  CUTE_HOST_DEVICE constexpr
  decltype(auto) operator()(T&& arg) const {
    return static_cast<T&&>(arg) << Shift;
  }
};

/************/
/** Binary **/
/************/

#define CUTE_BINARY_OP(NAME,OP)                                      \
  struct NAME {                                                      \
    template <class T, class U>                                      \
    CUTE_HOST_DEVICE constexpr                                       \
    decltype(auto) operator()(T&& lhs, U&& rhs) const {              \
      return static_cast<T&&>(lhs) OP static_cast<U&&>(rhs);           \
    }                                                                \
  }
#define CUTE_NAMED_BINARY_OP(NAME,OP)                                \
  struct NAME {                                                      \
    template <class T, class U>                                      \
    CUTE_HOST_DEVICE constexpr                                       \
    decltype(auto) operator()(T&& lhs, U&& rhs) const {              \
      return OP (static_cast<T&&>(lhs), static_cast<U&&>(rhs));        \
    }                                                                \
  }

CUTE_BINARY_OP(plus,                 +);
CUTE_BINARY_OP(minus,                -);
CUTE_BINARY_OP(multiplies,           *);
CUTE_BINARY_OP(divides,              /);
CUTE_BINARY_OP(modulus,              %);

CUTE_BINARY_OP(plus_assign,         +=);
CUTE_BINARY_OP(minus_assign,        -=);
CUTE_BINARY_OP(multiplies_assign,   *=);
CUTE_BINARY_OP(divides_assign,      /=);
CUTE_BINARY_OP(modulus_assign,      %=);

CUTE_BINARY_OP(bit_and,              &);
CUTE_BINARY_OP(bit_or,               |);
CUTE_BINARY_OP(bit_xor,              ^);
CUTE_BINARY_OP(left_shift,          <<);
CUTE_BINARY_OP(right_shift,         >>);

CUTE_BINARY_OP(bit_and_assign,      &=);
CUTE_BINARY_OP(bit_or_assign,       |=);
CUTE_BINARY_OP(bit_xor_assign,      ^=);
CUTE_BINARY_OP(left_shift_assign,  <<=);
CUTE_BINARY_OP(right_shift_assign, >>=);

CUTE_BINARY_OP(logical_and,         &&);
CUTE_BINARY_OP(logical_or,          ||);

CUTE_BINARY_OP(equal_to,            ==);
CUTE_BINARY_OP(not_equal_to,        !=);
CUTE_BINARY_OP(greater,              >);
CUTE_BINARY_OP(less,                 <);
CUTE_BINARY_OP(greater_equal,       >=);
CUTE_BINARY_OP(less_equal,          <=);

CUTE_NAMED_BINARY_OP(max_fn, cute::max);
CUTE_NAMED_BINARY_OP(min_fn, cute::min);
CUTE_NAMED_BINARY_OP(gcd_fn, cute::gcd);

#undef CUTE_BINARY_OP
#undef CUTE_NAMED_BINARY_OP

/**********/
/** Fold **/
/**********/

#define CUTE_FOLD_OP(NAME,OP)                                        \
  struct NAME##_unary_rfold {                                        \
    template <class... T>                                            \
    CUTE_HOST_DEVICE constexpr                                       \
    auto operator()(T&&... t) const {                                \
      return (t OP ...);                                             \
    }                                                                \
  };                                                                 \
  struct NAME##_unary_lfold {                                        \
    template <class... T>                                            \
    CUTE_HOST_DEVICE constexpr                                       \
    auto operator()(T&&... t) const {                                \
      return (... OP t);                                             \
    }                                                                \
  };                                                                 \
  struct NAME##_binary_rfold {                                       \
    template <class U, class... T>                                   \
    CUTE_HOST_DEVICE constexpr                                       \
    auto operator()(U&& u, T&&... t) const {                         \
      return (t OP ... OP u);                                        \
    }                                                                \
  };                                                                 \
  struct NAME##_binary_lfold {                                       \
    template <class U, class... T>                                   \
    CUTE_HOST_DEVICE constexpr                                       \
    auto operator()(U&& u, T&&... t) const {                         \
      return (u OP ... OP t);                                        \
    }                                                                \
  }

CUTE_FOLD_OP(plus,                 +);
CUTE_FOLD_OP(minus,                -);
CUTE_FOLD_OP(multiplies,           *);
CUTE_FOLD_OP(divides,              /);
CUTE_FOLD_OP(modulus,              %);

CUTE_FOLD_OP(plus_assign,         +=);
CUTE_FOLD_OP(minus_assign,        -=);
CUTE_FOLD_OP(multiplies_assign,   *=);
CUTE_FOLD_OP(divides_assign,      /=);
CUTE_FOLD_OP(modulus_assign,      %=);

CUTE_FOLD_OP(bit_and,              &);
CUTE_FOLD_OP(bit_or,               |);
CUTE_FOLD_OP(bit_xor,              ^);
CUTE_FOLD_OP(left_shift,          <<);
CUTE_FOLD_OP(right_shift,         >>);

CUTE_FOLD_OP(bit_and_assign,      &=);
CUTE_FOLD_OP(bit_or_assign,       |=);
CUTE_FOLD_OP(bit_xor_assign,      ^=);
CUTE_FOLD_OP(left_shift_assign,  <<=);
CUTE_FOLD_OP(right_shift_assign, >>=);

CUTE_FOLD_OP(logical_and,         &&);
CUTE_FOLD_OP(logical_or,          ||);

CUTE_FOLD_OP(equal_to,            ==);
CUTE_FOLD_OP(not_equal_to,        !=);
CUTE_FOLD_OP(greater,              >);
CUTE_FOLD_OP(less,                 <);
CUTE_FOLD_OP(greater_equal,       >=);
CUTE_FOLD_OP(less_equal,          <=);

#undef CUTE_FOLD_OP

/**********/
/** Meta **/
/**********/

template <class Fn, class Arg>
struct bound_fn {

  template <class T>
  CUTE_HOST_DEVICE constexpr
  decltype(auto)
  operator()(T&& arg) {
    return fn_(arg_, static_cast<T&&>(arg));
  }

  Fn fn_;
  Arg arg_;
};

template <class Fn, class Arg>
CUTE_HOST_DEVICE constexpr
auto
bind(Fn const& fn, Arg const& arg) {
  return bound_fn<Fn,Arg>{fn, arg};
}

} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute, cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute, cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** This header mainly contributes targeted infrastructure for the surrounding CuTe subsystem.
  **CN:** 该头文件主要为周边 CuTe 子系统提供定向基础设施。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/numeric/math.hpp` supplies related definitions from `cute/numeric/math.hpp`.
  **CN:** `cute/numeric/math.hpp` 提供了来自 `cute/numeric/math.hpp` 的相关定义。
- **EN:** `cute/numeric/complex.hpp` supplies related definitions from `cute/numeric/complex.hpp`.
  **CN:** `cute/numeric/complex.hpp` 提供了来自 `cute/numeric/complex.hpp` 的相关定义。
