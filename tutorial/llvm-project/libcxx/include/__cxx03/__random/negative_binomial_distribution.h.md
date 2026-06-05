# negative_binomial_distribution.h — Code Analysis / 代码分析

## Source / 来源
- File: `libcxx/include/__cxx03/__random/negative_binomial_distribution.h`
- Repository: `llvm-project`
- Purpose (EN): Defines `negative_binomial_distribution` as part of libc++ random-number facilities and probability distributions.
- 作用 (CN): 该文件定义了 `negative_binomial_distribution`，属于 libc++ 的随机数设施与概率分布实现。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```cpp
   1: //===----------------------------------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
```
- EN: The block is comment-only prologue material, documenting licensing, provenance, or API intent.
- CN: 该代码块仅包含注释性前言，用于说明许可证、来源或 API 设计意图。

### Lines 9-19
```cpp
   9: #ifndef _LIBCPP___CXX03___RANDOM_NEGATIVE_BINOMIAL_DISTRIBUTION_H
  10: #define _LIBCPP___CXX03___RANDOM_NEGATIVE_BINOMIAL_DISTRIBUTION_H
  11: 
  12: #include <__cxx03/__assert>
  13: #include <__cxx03/__config>
  14: #include <__cxx03/__random/bernoulli_distribution.h>
  15: #include <__cxx03/__random/gamma_distribution.h>
  16: #include <__cxx03/__random/is_valid.h>
  17: #include <__cxx03/__random/poisson_distribution.h>
  18: #include <__cxx03/iosfwd>
  19: #include <__cxx03/limits>
```
- EN: It imports `__cxx03/__assert`, `__cxx03/__config`, `__cxx03/__random/bernoulli_distribution.h`, `__cxx03/__random/gamma_distribution.h`, ... to make required declarations, traits, and utilities available. The header guard protects against multiple inclusion and preserves one-definition behavior. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 这里引入了 `__cxx03/__assert`, `__cxx03/__config`, `__cxx03/__random/bernoulli_distribution.h`, `__cxx03/__random/gamma_distribution.h`, ...，为后续实现提供所需声明、traits 与工具。 头文件保护宏可避免重复包含，并维持单一定义相关行为。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 20-24
```cpp
  20: 
  21: #if !defined(_LIBCPP_HAS_NO_PRAGMA_SYSTEM_HEADER)
  22: #  pragma GCC system_header
  23: #endif
  24: 
```
- EN: Conditional compilation gates this code on language mode, platform support, or libc++ configuration. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 条件编译依据语言版本、平台支持或 libc++ 配置来决定此处代码是否启用。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 25-29
```cpp
  25: _LIBCPP_PUSH_MACROS
  26: #include <__cxx03/__undef_macros>
  27: 
  28: _LIBCPP_BEGIN_NAMESPACE_STD
  29: 
```
- EN: It imports `__cxx03/__undef_macros` to make required declarations, traits, and utilities available. Namespace markers place the implementation into libc++'s configured namespace and ABI layout. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 这里引入了 `__cxx03/__undef_macros`，为后续实现提供所需声明、traits 与工具。 命名空间标记把实现放入 libc++ 约定的命名空间与 ABI 布局中。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 30-37
```cpp
  30: template <class _IntType = int>
  31: class _LIBCPP_TEMPLATE_VIS negative_binomial_distribution {
  32:   static_assert(__libcpp_random_is_valid_inttype<_IntType>::value, "IntType must be a supported integer type");
  33: 
  34: public:
  35:   // types
  36:   typedef _IntType result_type;
  37: 
```
- EN: This block introduces `negative_binomial_distribution` as the main type or helper abstraction in this area. Template parameters keep the facility generic across user-supplied types, iterators, or callables. Compile-time assertions reject unsupported template arguments before instantiation continues.
- CN: 这一段引入了 `negative_binomial_distribution`，作为该区域的主要类型或辅助抽象。 模板参数让该设施能够适配用户提供的类型、迭代器或可调用对象。 编译期断言会在模板继续实例化前拒绝不受支持的参数。

### Lines 38-44
```cpp
  38:   class _LIBCPP_TEMPLATE_VIS param_type {
  39:     result_type __k_;
  40:     double __p_;
  41: 
  42:   public:
  43:     typedef negative_binomial_distribution distribution_type;
  44: 
```
- EN: This block introduces `param_type` as the main type or helper abstraction in this area. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 这一段引入了 `param_type`，作为该区域的主要类型或辅助抽象。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 45-49
```cpp
  45:     _LIBCPP_HIDE_FROM_ABI explicit param_type(result_type __k = 1, double __p = 0.5) : __k_(__k), __p_(__p) {}
  46: 
  47:     _LIBCPP_HIDE_FROM_ABI result_type k() const { return __k_; }
  48:     _LIBCPP_HIDE_FROM_ABI double p() const { return __p_; }
  49: 
```
- EN: The code declares or defines `__p_`, `p` and wires parameter handling, annotations, or result propagation. The return path exposes the computed library value or forwards the user-visible operation result. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 该段声明或定义了 `__p_`, `p`，并串联参数处理、注解以及结果传递逻辑。 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 50-55
```cpp
  50:     friend _LIBCPP_HIDE_FROM_ABI bool operator==(const param_type& __x, const param_type& __y) {
  51:       return __x.__k_ == __y.__k_ && __x.__p_ == __y.__p_;
  52:     }
  53:     friend _LIBCPP_HIDE_FROM_ABI bool operator!=(const param_type& __x, const param_type& __y) { return !(__x == __y); }
  54:   };
  55: 
```
- EN: The return path exposes the computed library value or forwards the user-visible operation result. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 56-65
```cpp
  56: private:
  57:   param_type __p_;
  58: 
  59: public:
  60:   // constructor and reset functions
  61:   _LIBCPP_HIDE_FROM_ABI explicit negative_binomial_distribution(result_type __k = 1, double __p = 0.5)
  62:       : __p_(__k, __p) {}
  63:   _LIBCPP_HIDE_FROM_ABI explicit negative_binomial_distribution(const param_type& __p) : __p_(__p) {}
  64:   _LIBCPP_HIDE_FROM_ABI void reset() {}
  65: 
```
- EN: The code declares or defines `reset` and wires parameter handling, annotations, or result propagation. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 该段声明或定义了 `reset`，并串联参数处理、注解以及结果传递逻辑。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 66-73
```cpp
  66:   // generating functions
  67:   template <class _URNG>
  68:   _LIBCPP_HIDE_FROM_ABI result_type operator()(_URNG& __g) {
  69:     return (*this)(__g, __p_);
  70:   }
  71:   template <class _URNG>
  72:   _LIBCPP_HIDE_FROM_ABI result_type operator()(_URNG& __g, const param_type& __p);
  73: 
```
- EN: Template parameters keep the facility generic across user-supplied types, iterators, or callables. The return path exposes the computed library value or forwards the user-visible operation result. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 模板参数让该设施能够适配用户提供的类型、迭代器或可调用对象。 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 74-80
```cpp
  74:   // property functions
  75:   _LIBCPP_HIDE_FROM_ABI result_type k() const { return __p_.k(); }
  76:   _LIBCPP_HIDE_FROM_ABI double p() const { return __p_.p(); }
  77: 
  78:   _LIBCPP_HIDE_FROM_ABI param_type param() const { return __p_; }
  79:   _LIBCPP_HIDE_FROM_ABI void param(const param_type& __p) { __p_ = __p; }
  80: 
```
- EN: The code declares or defines `p`, `param` and wires parameter handling, annotations, or result propagation. The return path exposes the computed library value or forwards the user-visible operation result. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 该段声明或定义了 `p`, `param`，并串联参数处理、注解以及结果传递逻辑。 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 81-92
```cpp
  81:   _LIBCPP_HIDE_FROM_ABI result_type min() const { return 0; }
  82:   _LIBCPP_HIDE_FROM_ABI result_type max() const { return numeric_limits<result_type>::max(); }
  83: 
  84:   friend _LIBCPP_HIDE_FROM_ABI bool
  85:   operator==(const negative_binomial_distribution& __x, const negative_binomial_distribution& __y) {
  86:     return __x.__p_ == __y.__p_;
  87:   }
  88:   friend _LIBCPP_HIDE_FROM_ABI bool
  89:   operator!=(const negative_binomial_distribution& __x, const negative_binomial_distribution& __y) {
  90:     return !(__x == __y);
  91:   }
  92: };
```
- EN: The code declares or defines `max` and wires parameter handling, annotations, or result propagation. The return path exposes the computed library value or forwards the user-visible operation result. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 该段声明或定义了 `max`，并串联参数处理、注解以及结果传递逻辑。 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

### Lines 93-104
```cpp
  93: 
  94: template <class _IntType>
  95: template <class _URNG>
  96: _IntType negative_binomial_distribution<_IntType>::operator()(_URNG& __urng, const param_type& __pr) {
  97:   static_assert(__libcpp_random_is_valid_urng<_URNG>::value, "");
  98:   result_type __k = __pr.k();
  99:   double __p      = __pr.p();
 100:   // When the number of bits in _IntType is small, we are too likely to
 101:   // overflow __f below to use this technique.
 102:   if (__k <= 21 * __p && sizeof(_IntType) > 1) {
 103:     bernoulli_distribution __gen(__p);
 104:     result_type __f = 0;
```
- EN: Template parameters keep the facility generic across user-supplied types, iterators, or callables. Compile-time assertions reject unsupported template arguments before instantiation continues. The code declares or defines `k`, `p`, `__gen` and wires parameter handling, annotations, or result propagation.
- CN: 模板参数让该设施能够适配用户提供的类型、迭代器或可调用对象。 编译期断言会在模板继续实例化前拒绝不受支持的参数。 该段声明或定义了 `k`, `p`, `__gen`，并串联参数处理、注解以及结果传递逻辑。

### Lines 105-116
```cpp
 105:     result_type __s = 0;
 106:     while (__s < __k) {
 107:       if (__gen(__urng))
 108:         ++__s;
 109:       else
 110:         ++__f;
 111:     }
 112:     _LIBCPP_ASSERT_INTERNAL(__f >= 0,
 113:                             "std::negative_binomial_distribution should never produce negative values. "
 114:                             "This is almost certainly a signed integer overflow issue on __f.");
 115:     return __f;
 116:   }
```
- EN: The code declares or defines `__gen` and wires parameter handling, annotations, or result propagation. Local control flow performs the core algorithm and preserves invariants before exposing the result/state. The return path exposes the computed library value or forwards the user-visible operation result.
- CN: 该段声明或定义了 `__gen`，并串联参数处理、注解以及结果传递逻辑。 局部控制流在输出结果或状态前执行核心算法并维护必要不变量。 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。

### Lines 117-128
```cpp
 117:   return poisson_distribution<result_type>(gamma_distribution<double>(__k, (1 - __p) / __p)(__urng))(__urng);
 118: }
 119: 
 120: template <class _CharT, class _Traits, class _IntType>
 121: _LIBCPP_HIDE_FROM_ABI basic_ostream<_CharT, _Traits>&
 122: operator<<(basic_ostream<_CharT, _Traits>& __os, const negative_binomial_distribution<_IntType>& __x) {
 123:   __save_flags<_CharT, _Traits> __lx(__os);
 124:   typedef basic_ostream<_CharT, _Traits> _OStream;
 125:   __os.flags(_OStream::dec | _OStream::left | _OStream::fixed | _OStream::scientific);
 126:   _CharT __sp = __os.widen(' ');
 127:   __os.fill(__sp);
 128:   return __os << __x.k() << __sp << __x.p();
```
- EN: Template parameters keep the facility generic across user-supplied types, iterators, or callables. The code declares or defines `__lx`, `flags`, `widen`, `fill`, ... and wires parameter handling, annotations, or result propagation. The return path exposes the computed library value or forwards the user-visible operation result.
- CN: 模板参数让该设施能够适配用户提供的类型、迭代器或可调用对象。 该段声明或定义了 `__lx`, `flags`, `widen`, `fill`, ...，并串联参数处理、注解以及结果传递逻辑。 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。

### Lines 129-140
```cpp
 129: }
 130: 
 131: template <class _CharT, class _Traits, class _IntType>
 132: _LIBCPP_HIDE_FROM_ABI basic_istream<_CharT, _Traits>&
 133: operator>>(basic_istream<_CharT, _Traits>& __is, negative_binomial_distribution<_IntType>& __x) {
 134:   typedef negative_binomial_distribution<_IntType> _Eng;
 135:   typedef typename _Eng::result_type result_type;
 136:   typedef typename _Eng::param_type param_type;
 137:   __save_flags<_CharT, _Traits> __lx(__is);
 138:   typedef basic_istream<_CharT, _Traits> _Istream;
 139:   __is.flags(_Istream::dec | _Istream::skipws);
 140:   result_type __k;
```
- EN: Template parameters keep the facility generic across user-supplied types, iterators, or callables. The code declares or defines `__lx`, `flags` and wires parameter handling, annotations, or result propagation. Stream operators preserve or restore object state so the type follows standard-library serialization conventions.
- CN: 模板参数让该设施能够适配用户提供的类型、迭代器或可调用对象。 该段声明或定义了 `__lx`, `flags`，并串联参数处理、注解以及结果传递逻辑。 流运算符用于保存或恢复对象状态，使该类型符合标准库序列化约定。

### Lines 141-147
```cpp
 141:   double __p;
 142:   __is >> __k >> __p;
 143:   if (!__is.fail())
 144:     __x.param(param_type(__k, __p));
 145:   return __is;
 146: }
 147: 
```
- EN: The code declares or defines `param_type` and wires parameter handling, annotations, or result propagation. Local control flow performs the core algorithm and preserves invariants before exposing the result/state. The return path exposes the computed library value or forwards the user-visible operation result.
- CN: 该段声明或定义了 `param_type`，并串联参数处理、注解以及结果传递逻辑。 局部控制流在输出结果或状态前执行核心算法并维护必要不变量。 返回路径会产出计算所得的库内部值，或转发用户可见的操作结果。

### Lines 148-152
```cpp
 148: _LIBCPP_END_NAMESPACE_STD
 149: 
 150: _LIBCPP_POP_MACROS
 151: 
 152: #endif // _LIBCPP___CXX03___RANDOM_NEGATIVE_BINOMIAL_DISTRIBUTION_H
```
- EN: Namespace markers place the implementation into libc++'s configured namespace and ABI layout. libc++ visibility and configuration macros manage portability, ABI exposure, and inlining behavior here.
- CN: 命名空间标记把实现放入 libc++ 约定的命名空间与 ABI 布局中。 这里的 libc++ 可见性与配置宏负责管理可移植性、ABI 暴露方式以及内联行为。

## Key Concepts / 关键概念
- Probability state normalization and sampling / 概率状态归一化与采样
- Template-based generic programming / 基于模板的泛型编程
- Compile-time validation and trait composition / 编译期校验与 traits 组合
- Serializable library state / 可序列化的库状态
- libc++ ABI/configuration macro discipline / libc++ ABI/配置宏规范
- Primary symbols: `negative_binomial_distribution`, `param_type`, `__p_`, `p`, `reset`, `_IntType`, `basic_ostream` / 主要符号：`negative_binomial_distribution`, `param_type`, `__p_`, `p`, `reset`, `_IntType`, `basic_ostream`

## Dependencies / 依赖关系
### Direct includes / 直接包含
- `__cxx03/__assert`
- `__cxx03/__config`
- `__cxx03/__random/bernoulli_distribution.h`
- `__cxx03/__random/gamma_distribution.h`
- `__cxx03/__random/is_valid.h`
- `__cxx03/__random/poisson_distribution.h`
- `__cxx03/iosfwd`
- `__cxx03/limits`
- `__cxx03/__undef_macros`
### Internal hooks / 内部钩子
- libc++ macros: `_LIBCPP_BEGIN_NAMESPACE_STD`, `_LIBCPP_END_NAMESPACE_STD`, `_LIBCPP_HIDE_FROM_ABI`, `_LIBCPP_TEMPLATE_VIS`, `_LIBCPP_PUSH_MACROS`, `_LIBCPP_POP_MACROS`
- Related symbols / 相关符号: `negative_binomial_distribution`, `param_type`, `__p_`, `p`, `reset`, `param`
- Domain / 领域: random-number facilities and probability distributions / 随机数设施与概率分布实现
