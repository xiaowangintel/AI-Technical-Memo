# gamma.hpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/gamma.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/` (`llvm-project`)
- **Purpose / 目的**:
  - **EN**: Declares Boost.Math special functions, approximation kernels, and type-generic numeric entry points.
  - **CN**: 声明 Boost.Math 的特殊函数、近似计算内核以及按类型泛化的数值入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

````cpp
   1: //  Copyright John Maddock 2006-7, 2013-20.
   2: //  Copyright Paul A. Bristow 2007, 2013-14.
   3: //  Copyright Nikhar Agrawal 2013-14
   4: //  Copyright Christopher Kormanyos 2013-14, 2020, 2024
   5: //  Copyright Matt Borland 2024.
   6: //  Use, modification and distribution are subject to the
   7: //  Boost Software License, Version 1.0. (See accompanying file
   8: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   9: 
  10: #ifndef BOOST_MATH_SF_GAMMA_HPP
  11: #define BOOST_MATH_SF_GAMMA_HPP
  12: 
  13: #ifdef _MSC_VER
  14: #pragma once
  15: #endif
  16: 
  17: #include <boost/math/tools/config.hpp>
  18: #include <boost/math/tools/series.hpp>
  19: #include <boost/math/tools/fraction.hpp>
  20: #include <boost/math/tools/precision.hpp>
  21: #include <boost/math/tools/promotion.hpp>
  22: #include <boost/math/tools/type_traits.hpp>
  23: #include <boost/math/tools/numeric_limits.hpp>
  24: #include <boost/math/tools/cstdint.hpp>
  25: #include <boost/math/tools/assert.hpp>
  26: #include <boost/math/policies/error_handling.hpp>
  27: #include <boost/math/constants/constants.hpp>
  28: #include <boost/math/special_functions/math_fwd.hpp>
````
- **L1 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L1 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L2 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L2 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L3 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L3 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L4 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L4 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L5 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L5 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L6 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L6 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L7 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L7 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L8 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L8 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L9 EN**: Blank line separating nearby declarations or logic.
  - **L9 CN**: 空行，用于分隔相邻声明或逻辑。
- **L10 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_SF_GAMMA_HPP`.
  - **L10 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_SF_GAMMA_HPP`。
- **L11 EN**: Defines macro `BOOST_MATH_SF_GAMMA_HPP` for compile-time control, shorthand, or generated boilerplate.
  - **L11 CN**: 定义宏 `BOOST_MATH_SF_GAMMA_HPP`，用于编译期控制、简写或生成样板代码。
- **L12 EN**: Blank line separating nearby declarations or logic.
  - **L12 CN**: 空行，用于分隔相邻声明或逻辑。
- **L13 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L13 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L14 EN**: Uses `#pragma once` to prevent multiple inclusion of this header.
  - **L14 CN**: 使用 `#pragma once` 防止该头文件被重复包含。
- **L15 EN**: Closes the current preprocessor conditional block or header guard.
  - **L15 CN**: 结束当前预处理条件块或头文件保护。
- **L16 EN**: Blank line separating nearby declarations or logic.
  - **L16 CN**: 空行，用于分隔相邻声明或逻辑。
- **L17 EN**: Includes <boost/math/tools/config.hpp> to access Boost.Math numeric tool helpers.
  - **L17 CN**: 引入 <boost/math/tools/config.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L18 EN**: Includes <boost/math/tools/series.hpp> to access Boost.Math numeric tool helpers.
  - **L18 CN**: 引入 <boost/math/tools/series.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L19 EN**: Includes <boost/math/tools/fraction.hpp> to access Boost.Math numeric tool helpers.
  - **L19 CN**: 引入 <boost/math/tools/fraction.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L20 EN**: Includes <boost/math/tools/precision.hpp> to access Boost.Math numeric tool helpers.
  - **L20 CN**: 引入 <boost/math/tools/precision.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L21 EN**: Includes <boost/math/tools/promotion.hpp> to access Boost.Math numeric tool helpers.
  - **L21 CN**: 引入 <boost/math/tools/promotion.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L22 EN**: Includes <boost/math/tools/type_traits.hpp> to access Boost.Math numeric tool helpers.
  - **L22 CN**: 引入 <boost/math/tools/type_traits.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L23 EN**: Includes <boost/math/tools/numeric_limits.hpp> to access Boost.Math numeric tool helpers.
  - **L23 CN**: 引入 <boost/math/tools/numeric_limits.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L24 EN**: Includes <boost/math/tools/cstdint.hpp> to access Boost.Math numeric tool helpers.
  - **L24 CN**: 引入 <boost/math/tools/cstdint.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L25 EN**: Includes <boost/math/tools/assert.hpp> to access Boost.Math numeric tool helpers.
  - **L25 CN**: 引入 <boost/math/tools/assert.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L26 EN**: Includes <boost/math/policies/error_handling.hpp> to access Boost.Math policy configuration.
  - **L26 CN**: 引入 <boost/math/policies/error_handling.hpp> 以使用Boost.Math 策略配置。
- **L27 EN**: Includes <boost/math/constants/constants.hpp> to access Boost.Math numeric constants.
  - **L27 CN**: 引入 <boost/math/constants/constants.hpp> 以使用Boost.Math 数值常量。
- **L28 EN**: Includes <boost/math/special_functions/math_fwd.hpp> to access Boost.Math special-function declarations.
  - **L28 CN**: 引入 <boost/math/special_functions/math_fwd.hpp> 以使用Boost.Math 特殊函数声明。

### Lines 29-56 / 第 29-56 行

````cpp
  29: #include <boost/math/special_functions/log1p.hpp>
  30: #include <boost/math/special_functions/trunc.hpp>
  31: #include <boost/math/special_functions/powm1.hpp>
  32: #include <boost/math/special_functions/sqrt1pm1.hpp>
  33: #include <boost/math/special_functions/lanczos.hpp>
  34: #include <boost/math/special_functions/fpclassify.hpp>
  35: #include <boost/math/special_functions/detail/igamma_large.hpp>
  36: #include <boost/math/special_functions/detail/unchecked_factorial.hpp>
  37: #include <boost/math/special_functions/detail/lgamma_small.hpp>
  38: 
  39: // Only needed for types larger than double
  40: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
  41: #include <boost/math/special_functions/bernoulli.hpp>
  42: #include <boost/math/special_functions/polygamma.hpp>
  43: #endif
  44: 
  45: #ifdef _MSC_VER
  46: # pragma warning(push)
  47: # pragma warning(disable: 4702) // unreachable code (return after domain_error throw).
  48: # pragma warning(disable: 4127) // conditional expression is constant.
  49: # pragma warning(disable: 4100) // unreferenced formal parameter.
  50: # pragma warning(disable: 6326) // potential comparison of a constant with another constant
  51: // Several variables made comments,
  52: // but some difficulty as whether referenced on not may depend on macro values.
  53: // So to be safe, 4100 warnings suppressed.
  54: // TODO - revisit this?
  55: #endif
  56: 
````
- **L29 EN**: Includes <boost/math/special_functions/log1p.hpp> to access Boost.Math special-function declarations.
  - **L29 CN**: 引入 <boost/math/special_functions/log1p.hpp> 以使用Boost.Math 特殊函数声明。
- **L30 EN**: Includes <boost/math/special_functions/trunc.hpp> to access Boost.Math special-function declarations.
  - **L30 CN**: 引入 <boost/math/special_functions/trunc.hpp> 以使用Boost.Math 特殊函数声明。
- **L31 EN**: Includes <boost/math/special_functions/powm1.hpp> to access Boost.Math special-function declarations.
  - **L31 CN**: 引入 <boost/math/special_functions/powm1.hpp> 以使用Boost.Math 特殊函数声明。
- **L32 EN**: Includes <boost/math/special_functions/sqrt1pm1.hpp> to access Boost.Math special-function declarations.
  - **L32 CN**: 引入 <boost/math/special_functions/sqrt1pm1.hpp> 以使用Boost.Math 特殊函数声明。
- **L33 EN**: Includes <boost/math/special_functions/lanczos.hpp> to access Boost.Math special-function declarations.
  - **L33 CN**: 引入 <boost/math/special_functions/lanczos.hpp> 以使用Boost.Math 特殊函数声明。
- **L34 EN**: Includes <boost/math/special_functions/fpclassify.hpp> to access Boost.Math special-function declarations.
  - **L34 CN**: 引入 <boost/math/special_functions/fpclassify.hpp> 以使用Boost.Math 特殊函数声明。
- **L35 EN**: Includes <boost/math/special_functions/detail/igamma_large.hpp> to access Boost.Math special-function declarations.
  - **L35 CN**: 引入 <boost/math/special_functions/detail/igamma_large.hpp> 以使用Boost.Math 特殊函数声明。
- **L36 EN**: Includes <boost/math/special_functions/detail/unchecked_factorial.hpp> to access Boost.Math special-function declarations.
  - **L36 CN**: 引入 <boost/math/special_functions/detail/unchecked_factorial.hpp> 以使用Boost.Math 特殊函数声明。
- **L37 EN**: Includes <boost/math/special_functions/detail/lgamma_small.hpp> to access Boost.Math special-function declarations.
  - **L37 CN**: 引入 <boost/math/special_functions/detail/lgamma_small.hpp> 以使用Boost.Math 特殊函数声明。
- **L38 EN**: Blank line separating nearby declarations or logic.
  - **L38 CN**: 空行，用于分隔相邻声明或逻辑。
- **L39 EN**: Comment documents nearby intent or usage notes: `Only needed for types larger than double`.
  - **L39 CN**: 注释说明附近代码的意图或使用说明：`Only needed for types larger than double`。
- **L40 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_HAS_GPU_SUPPORT`.
  - **L40 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_HAS_GPU_SUPPORT`。
- **L41 EN**: Includes <boost/math/special_functions/bernoulli.hpp> to access Boost.Math special-function declarations.
  - **L41 CN**: 引入 <boost/math/special_functions/bernoulli.hpp> 以使用Boost.Math 特殊函数声明。
- **L42 EN**: Includes <boost/math/special_functions/polygamma.hpp> to access Boost.Math special-function declarations.
  - **L42 CN**: 引入 <boost/math/special_functions/polygamma.hpp> 以使用Boost.Math 特殊函数声明。
- **L43 EN**: Closes the current preprocessor conditional block or header guard.
  - **L43 CN**: 结束当前预处理条件块或头文件保护。
- **L44 EN**: Blank line separating nearby declarations or logic.
  - **L44 CN**: 空行，用于分隔相邻声明或逻辑。
- **L45 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L45 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L46 EN**: Continues logic associated with callable symbol `warning`.
  - **L46 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L47 EN**: Continues logic associated with callable symbol `warning`.
  - **L47 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L48 EN**: Continues logic associated with callable symbol `warning`.
  - **L48 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L49 EN**: Continues logic associated with callable symbol `warning`.
  - **L49 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L50 EN**: Continues logic associated with callable symbol `warning`.
  - **L50 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L51 EN**: Comment documents nearby intent or usage notes: `Several variables made comments,`.
  - **L51 CN**: 注释说明附近代码的意图或使用说明：`Several variables made comments,`。
- **L52 EN**: Comment documents nearby intent or usage notes: `but some difficulty as whether referenced on not may depend on macro values.`.
  - **L52 CN**: 注释说明附近代码的意图或使用说明：`but some difficulty as whether referenced on not may depend on macro values.`。
- **L53 EN**: Comment documents nearby intent or usage notes: `So to be safe, 4100 warnings suppressed.`.
  - **L53 CN**: 注释说明附近代码的意图或使用说明：`So to be safe, 4100 warnings suppressed.`。
- **L54 EN**: Comment documents nearby intent or usage notes: `TODO - revisit this?`.
  - **L54 CN**: 注释说明附近代码的意图或使用说明：`TODO - revisit this?`。
- **L55 EN**: Closes the current preprocessor conditional block or header guard.
  - **L55 CN**: 结束当前预处理条件块或头文件保护。
- **L56 EN**: Blank line separating nearby declarations or logic.
  - **L56 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 57-84 / 第 57-84 行

````cpp
  57: namespace boost{ namespace math{
  58: 
  59: namespace detail{
  60: 
  61: template <class T>
  62: BOOST_MATH_GPU_ENABLED inline bool is_odd(T v, const boost::math::true_type&)
  63: {
  64:    int i = static_cast<int>(v);
  65:    return i&1;
  66: }
  67: template <class T>
  68: BOOST_MATH_GPU_ENABLED inline bool is_odd(T v, const boost::math::false_type&)
  69: {
  70:    // Oh dear can't cast T to int!
  71:    BOOST_MATH_STD_USING
  72:    T modulus = v - 2 * floor(v/2);
  73:    return static_cast<bool>(modulus != 0);
  74: }
  75: template <class T>
  76: BOOST_MATH_GPU_ENABLED inline bool is_odd(T v)
  77: {
  78:    return is_odd(v, ::boost::math::is_convertible<T, int>());
  79: }
  80: 
  81: template <class T>
  82: BOOST_MATH_GPU_ENABLED T sinpx(T z)
  83: {
  84:    // Ad hoc function calculates x * sin(pi * x),
````
- **L57 EN**: Opens namespace scope `boost{ namespace math`.
  - **L57 CN**: 打开命名空间作用域 `boost{ namespace math`。
- **L58 EN**: Blank line separating nearby declarations or logic.
  - **L58 CN**: 空行，用于分隔相邻声明或逻辑。
- **L59 EN**: Opens namespace scope `detail`.
  - **L59 CN**: 打开命名空间作用域 `detail`。
- **L60 EN**: Blank line separating nearby declarations or logic.
  - **L60 CN**: 空行，用于分隔相邻声明或逻辑。
- **L61 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L61 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L62 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L62 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L63 EN**: Opens a new lexical scope or compound statement.
  - **L63 CN**: 打开一个新的词法作用域或复合语句块。
- **L64 EN**: Initializes variable `i` from the right-hand expression.
  - **L64 CN**: 使用右侧表达式初始化变量 `i`。
- **L65 EN**: Returns from the current function with `i&1`.
  - **L65 CN**: 以 `i&1` 从当前函数返回。
- **L66 EN**: Closes the current lexical scope or compound statement.
  - **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L67 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L68 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L68 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L69 EN**: Opens a new lexical scope or compound statement.
  - **L69 CN**: 打开一个新的词法作用域或复合语句块。
- **L70 EN**: Comment documents nearby intent or usage notes: `Oh dear can't cast T to int!`.
  - **L70 CN**: 注释说明附近代码的意图或使用说明：`Oh dear can't cast T to int!`。
- **L71 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L71 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L72 EN**: Executes a call or declaration centered on `floor`.
  - **L72 CN**: 执行以 `floor` 为核心的调用或声明。
- **L73 EN**: Returns from the current function with `static_cast<bool>(modulus != 0)`.
  - **L73 CN**: 以 `static_cast<bool>(modulus != 0)` 从当前函数返回。
- **L74 EN**: Closes the current lexical scope or compound statement.
  - **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L75 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L76 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L76 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L77 EN**: Opens a new lexical scope or compound statement.
  - **L77 CN**: 打开一个新的词法作用域或复合语句块。
- **L78 EN**: Returns from the current function with `is_odd(v, ::boost::math::is_convertible<T, int>())`.
  - **L78 CN**: 以 `is_odd(v, ::boost::math::is_convertible<T, int>())` 从当前函数返回。
- **L79 EN**: Closes the current lexical scope or compound statement.
  - **L79 CN**: 结束当前词法作用域或复合语句块。
- **L80 EN**: Blank line separating nearby declarations or logic.
  - **L80 CN**: 空行，用于分隔相邻声明或逻辑。
- **L81 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L81 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L82 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L82 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L83 EN**: Opens a new lexical scope or compound statement.
  - **L83 CN**: 打开一个新的词法作用域或复合语句块。
- **L84 EN**: Comment documents nearby intent or usage notes: `Ad hoc function calculates x * sin(pi * x),`.
  - **L84 CN**: 注释说明附近代码的意图或使用说明：`Ad hoc function calculates x * sin(pi * x),`。

### Lines 85-112 / 第 85-112 行

````cpp
  85:    // taking extra care near when x is near a whole number.
  86:    BOOST_MATH_STD_USING
  87:    int sign = 1;
  88:    if(z < 0)
  89:    {
  90:       z = -z;
  91:    }
  92:    T fl = floor(z);
  93:    T dist;  // LCOV_EXCL_LINE
  94:    if(is_odd(fl))
  95:    {
  96:       fl += 1;
  97:       dist = fl - z;
  98:       sign = -sign;
  99:    }
 100:    else
 101:    {
 102:       dist = z - fl;
 103:    }
 104:    BOOST_MATH_ASSERT(fl >= 0);
 105:    if(dist > T(0.5))
 106:       dist = 1 - dist;
 107:    T result = sin(dist*boost::math::constants::pi<T>());
 108:    return sign*z*result;
 109: } // template <class T> T sinpx(T z)
 110: //
 111: // tgamma(z), with Lanczos support:
 112: //
````
- **L85 EN**: Comment documents nearby intent or usage notes: `taking extra care near when x is near a whole number.`.
  - **L85 CN**: 注释说明附近代码的意图或使用说明：`taking extra care near when x is near a whole number.`。
- **L86 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L86 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L87 EN**: Initializes variable `sign` from the right-hand expression.
  - **L87 CN**: 使用右侧表达式初始化变量 `sign`。
- **L88 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L88 CN**: 开始 `if` 控制流语句并计算其条件。
- **L89 EN**: Opens a new lexical scope or compound statement.
  - **L89 CN**: 打开一个新的词法作用域或复合语句块。
- **L90 EN**: Executes a standalone statement or declaration: `z = -z;`.
  - **L90 CN**: 执行一条独立语句或声明：`z = -z;`。
- **L91 EN**: Closes the current lexical scope or compound statement.
  - **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Executes a call or declaration centered on `floor`.
  - **L92 CN**: 执行以 `floor` 为核心的调用或声明。
- **L93 EN**: Continues the surrounding expression or declaration: `T dist;  // LCOV_EXCL_LINE`.
  - **L93 CN**: 继续构造周围的表达式或声明：`T dist;  // LCOV_EXCL_LINE`。
- **L94 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L94 CN**: 开始 `if` 控制流语句并计算其条件。
- **L95 EN**: Opens a new lexical scope or compound statement.
  - **L95 CN**: 打开一个新的词法作用域或复合语句块。
- **L96 EN**: Executes a standalone statement or declaration: `fl += 1;`.
  - **L96 CN**: 执行一条独立语句或声明：`fl += 1;`。
- **L97 EN**: Executes a standalone statement or declaration: `dist = fl - z;`.
  - **L97 CN**: 执行一条独立语句或声明：`dist = fl - z;`。
- **L98 EN**: Executes a standalone statement or declaration: `sign = -sign;`.
  - **L98 CN**: 执行一条独立语句或声明：`sign = -sign;`。
- **L99 EN**: Closes the current lexical scope or compound statement.
  - **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Starts the alternative branch of the preceding conditional.
  - **L100 CN**: 开始前一个条件语句的备选分支。
- **L101 EN**: Opens a new lexical scope or compound statement.
  - **L101 CN**: 打开一个新的词法作用域或复合语句块。
- **L102 EN**: Executes a standalone statement or declaration: `dist = z - fl;`.
  - **L102 CN**: 执行一条独立语句或声明：`dist = z - fl;`。
- **L103 EN**: Closes the current lexical scope or compound statement.
  - **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L104 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L105 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L105 CN**: 开始 `if` 控制流语句并计算其条件。
- **L106 EN**: Executes a standalone statement or declaration: `dist = 1 - dist;`.
  - **L106 CN**: 执行一条独立语句或声明：`dist = 1 - dist;`。
- **L107 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L107 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L108 EN**: Returns from the current function with `sign*z*result`.
  - **L108 CN**: 以 `sign*z*result` 从当前函数返回。
- **L109 EN**: Continues logic associated with callable symbol `sinpx`.
  - **L109 CN**: 继续与可调用符号 `sinpx` 相关的逻辑。
- **L110 EN**: Separator comment used for visual grouping.
  - **L110 CN**: 分隔注释，用于视觉分组。
- **L111 EN**: Comment documents nearby intent or usage notes: `tgamma(z), with Lanczos support:`.
  - **L111 CN**: 注释说明附近代码的意图或使用说明：`tgamma(z), with Lanczos support:`。
- **L112 EN**: Separator comment used for visual grouping.
  - **L112 CN**: 分隔注释，用于视觉分组。

### Lines 113-140 / 第 113-140 行

````cpp
 113: template <class T, class Policy, class Lanczos>
 114: BOOST_MATH_GPU_ENABLED T gamma_imp_final(T z, const Policy& pol, const Lanczos& l)
 115: {
 116:    BOOST_MATH_STD_USING
 117:    
 118:    (void)l; // Suppresses unused variable warning when BOOST_MATH_INSTRUMENT is not defined
 119: 
 120:    T result = 1;  
 121: 
 122: #ifdef BOOST_MATH_INSTRUMENT
 123:    static bool b = false;
 124:    if(!b)
 125:    {
 126:       std::cout << "tgamma_imp called with " << typeid(z).name() << " " << typeid(l).name() << std::endl;
 127:       b = true;
 128:    }
 129: #endif
 130:    constexpr auto function = "boost::math::tgamma<%1%>(%1%)";
 131: 
 132:    if(z <= 0)
 133:    {
 134:       // shift z to > 1:
 135:       while(z < 0)
 136:       {
 137:          result /= z;
 138:          z += 1;
 139:       }
 140:    }
````
- **L113 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L113 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L114 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L114 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L115 EN**: Opens a new lexical scope or compound statement.
  - **L115 CN**: 打开一个新的词法作用域或复合语句块。
- **L116 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L116 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L117 EN**: Blank line separating nearby declarations or logic.
  - **L117 CN**: 空行，用于分隔相邻声明或逻辑。
- **L118 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L118 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L119 EN**: Blank line separating nearby declarations or logic.
  - **L119 CN**: 空行，用于分隔相邻声明或逻辑。
- **L120 EN**: Executes a standalone statement or declaration: `T result = 1;`.
  - **L120 CN**: 执行一条独立语句或声明：`T result = 1;`。
- **L121 EN**: Blank line separating nearby declarations or logic.
  - **L121 CN**: 空行，用于分隔相邻声明或逻辑。
- **L122 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_INSTRUMENT`.
  - **L122 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_INSTRUMENT`。
- **L123 EN**: Initializes variable `b` from the right-hand expression.
  - **L123 CN**: 使用右侧表达式初始化变量 `b`。
- **L124 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L124 CN**: 开始 `if` 控制流语句并计算其条件。
- **L125 EN**: Opens a new lexical scope or compound statement.
  - **L125 CN**: 打开一个新的词法作用域或复合语句块。
- **L126 EN**: Executes a call or declaration centered on `typeid`.
  - **L126 CN**: 执行以 `typeid` 为核心的调用或声明。
- **L127 EN**: Executes a standalone statement or declaration: `b = true;`.
  - **L127 CN**: 执行一条独立语句或声明：`b = true;`。
- **L128 EN**: Closes the current lexical scope or compound statement.
  - **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Closes the current preprocessor conditional block or header guard.
  - **L129 CN**: 结束当前预处理条件块或头文件保护。
- **L130 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L130 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L131 EN**: Blank line separating nearby declarations or logic.
  - **L131 CN**: 空行，用于分隔相邻声明或逻辑。
- **L132 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L132 CN**: 开始 `if` 控制流语句并计算其条件。
- **L133 EN**: Opens a new lexical scope or compound statement.
  - **L133 CN**: 打开一个新的词法作用域或复合语句块。
- **L134 EN**: Comment documents nearby intent or usage notes: `shift z to > 1:`.
  - **L134 CN**: 注释说明附近代码的意图或使用说明：`shift z to > 1:`。
- **L135 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L135 CN**: 开始 `while` 控制流语句并计算其条件。
- **L136 EN**: Opens a new lexical scope or compound statement.
  - **L136 CN**: 打开一个新的词法作用域或复合语句块。
- **L137 EN**: Executes a standalone statement or declaration: `result /= z;`.
  - **L137 CN**: 执行一条独立语句或声明：`result /= z;`。
- **L138 EN**: Executes a standalone statement or declaration: `z += 1;`.
  - **L138 CN**: 执行一条独立语句或声明：`z += 1;`。
- **L139 EN**: Closes the current lexical scope or compound statement.
  - **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Closes the current lexical scope or compound statement.
  - **L140 CN**: 结束当前词法作用域或复合语句块。

### Lines 141-168 / 第 141-168 行

````cpp
 141:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 142:    if((floor(z) == z) && (z < max_factorial<T>::value))
 143:    {
 144:       result *= unchecked_factorial<T>(static_cast<unsigned>(itrunc(z, pol) - 1));
 145:       BOOST_MATH_INSTRUMENT_VARIABLE(result);
 146:    }
 147:    else if (z < tools::root_epsilon<T>())
 148:    {
 149:       if (z < 1 / tools::max_value<T>())
 150:          result = policies::raise_overflow_error<T>(function, nullptr, pol);
 151:       result *= 1 / z - constants::euler<T>();
 152:    }
 153:    else
 154:    {
 155:       result *= Lanczos::lanczos_sum(z);
 156:       T zgh = (z + static_cast<T>(Lanczos::g()) - boost::math::constants::half<T>());
 157:       T lzgh = log(zgh);
 158:       BOOST_MATH_INSTRUMENT_VARIABLE(result);
 159:       BOOST_MATH_INSTRUMENT_VARIABLE(tools::log_max_value<T>());
 160:       if(z * lzgh > tools::log_max_value<T>())
 161:       {
 162:          // we're going to overflow unless this is done with care:
 163:          BOOST_MATH_INSTRUMENT_VARIABLE(zgh);
 164:          if(lzgh * z / 2 > tools::log_max_value<T>())
 165:             return boost::math::sign(result) * policies::raise_overflow_error<T>(function, "Result of tgamma is too large to represent.", pol);
 166:          T hp = pow(zgh, T((z / 2) - T(0.25)));
 167:          BOOST_MATH_INSTRUMENT_VARIABLE(hp);
 168:          result *= hp / exp(zgh);
````
- **L141 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L141 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L143 EN**: Opens a new lexical scope or compound statement.
  - **L143 CN**: 打开一个新的词法作用域或复合语句块。
- **L144 EN**: Executes a call or declaration centered on `unchecked_factorial<T>`.
  - **L144 CN**: 执行以 `unchecked_factorial<T>` 为核心的调用或声明。
- **L145 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L145 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L146 EN**: Closes the current lexical scope or compound statement.
  - **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Starts the alternative branch of the preceding conditional.
  - **L147 CN**: 开始前一个条件语句的备选分支。
- **L148 EN**: Opens a new lexical scope or compound statement.
  - **L148 CN**: 打开一个新的词法作用域或复合语句块。
- **L149 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L149 CN**: 开始 `if` 控制流语句并计算其条件。
- **L150 EN**: Executes a call or declaration centered on `policies::raise_overflow_error<T>`.
  - **L150 CN**: 执行以 `policies::raise_overflow_error<T>` 为核心的调用或声明。
- **L151 EN**: Executes a call or declaration centered on `constants::euler<T>`.
  - **L151 CN**: 执行以 `constants::euler<T>` 为核心的调用或声明。
- **L152 EN**: Closes the current lexical scope or compound statement.
  - **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Starts the alternative branch of the preceding conditional.
  - **L153 CN**: 开始前一个条件语句的备选分支。
- **L154 EN**: Opens a new lexical scope or compound statement.
  - **L154 CN**: 打开一个新的词法作用域或复合语句块。
- **L155 EN**: Executes a call or declaration centered on `Lanczos::lanczos_sum`.
  - **L155 CN**: 执行以 `Lanczos::lanczos_sum` 为核心的调用或声明。
- **L156 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L156 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L157 EN**: Executes a call or declaration centered on `log`.
  - **L157 CN**: 执行以 `log` 为核心的调用或声明。
- **L158 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L158 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L159 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L159 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L160 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L160 CN**: 开始 `if` 控制流语句并计算其条件。
- **L161 EN**: Opens a new lexical scope or compound statement.
  - **L161 CN**: 打开一个新的词法作用域或复合语句块。
- **L162 EN**: Comment documents nearby intent or usage notes: `we're going to overflow unless this is done with care:`.
  - **L162 CN**: 注释说明附近代码的意图或使用说明：`we're going to overflow unless this is done with care:`。
- **L163 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L163 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L164 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L164 CN**: 开始 `if` 控制流语句并计算其条件。
- **L165 EN**: Returns from the current function with `boost::math::sign(result) * policies::raise_overflow_error<T>(function, "Result of tgamma is too large to represent.", pol)`.
  - **L165 CN**: 以 `boost::math::sign(result) * policies::raise_overflow_error<T>(function, "Result of tgamma is too large to represent.", pol)` 从当前函数返回。
- **L166 EN**: Executes a call or declaration centered on `pow`.
  - **L166 CN**: 执行以 `pow` 为核心的调用或声明。
- **L167 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L167 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L168 EN**: Executes a call or declaration centered on `exp`.
  - **L168 CN**: 执行以 `exp` 为核心的调用或声明。

### Lines 169-196 / 第 169-196 行

````cpp
 169:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 170:          if(tools::max_value<T>() / hp < result)
 171:             return boost::math::sign(result) * policies::raise_overflow_error<T>(function, "Result of tgamma is too large to represent.", pol);
 172:          result *= hp;
 173:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 174:       }
 175:       else
 176:       {
 177:          BOOST_MATH_INSTRUMENT_VARIABLE(zgh);
 178:          BOOST_MATH_INSTRUMENT_VARIABLE(pow(zgh, T(z - boost::math::constants::half<T>())));
 179:          BOOST_MATH_INSTRUMENT_VARIABLE(exp(zgh));
 180:          result *= pow(zgh, T(z - boost::math::constants::half<T>())) / exp(zgh);
 181:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 182:       }
 183:    }
 184:    return result;
 185: }
 186: 
 187: #ifdef BOOST_MATH_ENABLE_CUDA
 188: #  pragma nv_diag_suppress 2190
 189: #endif
 190: 
 191: // SYCL compilers can not support recursion so we extract it into a dispatch function
 192: template <class T, class Policy, class Lanczos>
 193: BOOST_MATH_GPU_ENABLED BOOST_MATH_FORCEINLINE T gamma_imp(T z, const Policy& pol, const Lanczos& l)
 194: {
 195:    BOOST_MATH_STD_USING
 196: 
````
- **L169 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L169 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L170 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L170 CN**: 开始 `if` 控制流语句并计算其条件。
- **L171 EN**: Returns from the current function with `boost::math::sign(result) * policies::raise_overflow_error<T>(function, "Result of tgamma is too large to represent.", pol)`.
  - **L171 CN**: 以 `boost::math::sign(result) * policies::raise_overflow_error<T>(function, "Result of tgamma is too large to represent.", pol)` 从当前函数返回。
- **L172 EN**: Executes a standalone statement or declaration: `result *= hp;`.
  - **L172 CN**: 执行一条独立语句或声明：`result *= hp;`。
- **L173 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L173 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L174 EN**: Closes the current lexical scope or compound statement.
  - **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Starts the alternative branch of the preceding conditional.
  - **L175 CN**: 开始前一个条件语句的备选分支。
- **L176 EN**: Opens a new lexical scope or compound statement.
  - **L176 CN**: 打开一个新的词法作用域或复合语句块。
- **L177 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L177 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L178 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L178 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L179 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L179 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L180 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L180 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L181 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L181 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L182 EN**: Closes the current lexical scope or compound statement.
  - **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Closes the current lexical scope or compound statement.
  - **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Returns from the current function with `result`.
  - **L184 CN**: 以 `result` 从当前函数返回。
- **L185 EN**: Closes the current lexical scope or compound statement.
  - **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Blank line separating nearby declarations or logic.
  - **L186 CN**: 空行，用于分隔相邻声明或逻辑。
- **L187 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_ENABLE_CUDA`.
  - **L187 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_ENABLE_CUDA`。
- **L188 EN**: Continues the surrounding expression or declaration: `#  pragma nv_diag_suppress 2190`.
  - **L188 CN**: 继续构造周围的表达式或声明：`#  pragma nv_diag_suppress 2190`。
- **L189 EN**: Closes the current preprocessor conditional block or header guard.
  - **L189 CN**: 结束当前预处理条件块或头文件保护。
- **L190 EN**: Blank line separating nearby declarations or logic.
  - **L190 CN**: 空行，用于分隔相邻声明或逻辑。
- **L191 EN**: Comment documents nearby intent or usage notes: `SYCL compilers can not support recursion so we extract it into a dispatch function`.
  - **L191 CN**: 注释说明附近代码的意图或使用说明：`SYCL compilers can not support recursion so we extract it into a dispatch function`。
- **L192 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L192 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L193 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L193 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L194 EN**: Opens a new lexical scope or compound statement.
  - **L194 CN**: 打开一个新的词法作用域或复合语句块。
- **L195 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L195 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L196 EN**: Blank line separating nearby declarations or logic.
  - **L196 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 197-224 / 第 197-224 行

````cpp
 197:    T result = 1;
 198:    constexpr auto function = "boost::math::tgamma<%1%>(%1%)";
 199: 
 200:    if(z <= 0)
 201:    {
 202:       if(floor(z) == z)
 203:          return policies::raise_pole_error<T>(function, "Evaluation of tgamma at a negative integer %1%.", z, pol);
 204:       if(z <= -20)
 205:       {
 206: #ifndef BOOST_MATH_NO_EXCEPTIONS
 207:          try
 208: #endif
 209:          {
 210:             result = gamma_imp_final(T(-z), pol, l) * sinpx(z);
 211:          }
 212: #ifndef BOOST_MATH_NO_EXCEPTIONS
 213:          catch (const std::overflow_error&)
 214:          {
 215:             return policies::raise_underflow_error<T>(function, "Result of tgamma is too small to represent.", pol);
 216:          }
 217: #endif
 218:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 219:          BOOST_MATH_IF_CONSTEXPR(!boost::math::numeric_limits<T>::is_specialized || (boost::math::numeric_limits<T>::digits > 64))
 220:          {
 221:             if ((fabs(result) < 1) && (tools::max_value<T>() * fabs(result) < boost::math::constants::pi<T>()))
 222:             {
 223:                return policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE MP only.
 224: 
````
- **L197 EN**: Executes a standalone statement or declaration: `T result = 1;`.
  - **L197 CN**: 执行一条独立语句或声明：`T result = 1;`。
- **L198 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L198 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L199 EN**: Blank line separating nearby declarations or logic.
  - **L199 CN**: 空行，用于分隔相邻声明或逻辑。
- **L200 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L200 CN**: 开始 `if` 控制流语句并计算其条件。
- **L201 EN**: Opens a new lexical scope or compound statement.
  - **L201 CN**: 打开一个新的词法作用域或复合语句块。
- **L202 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L202 CN**: 开始 `if` 控制流语句并计算其条件。
- **L203 EN**: Returns from the current function with `policies::raise_pole_error<T>(function, "Evaluation of tgamma at a negative integer %1%.", z, pol)`.
  - **L203 CN**: 以 `policies::raise_pole_error<T>(function, "Evaluation of tgamma at a negative integer %1%.", z, pol)` 从当前函数返回。
- **L204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L205 EN**: Opens a new lexical scope or compound statement.
  - **L205 CN**: 打开一个新的词法作用域或复合语句块。
- **L206 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_NO_EXCEPTIONS`.
  - **L206 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_NO_EXCEPTIONS`。
- **L207 EN**: Starts an exception-handling region.
  - **L207 CN**: 开始一个异常处理区域。
- **L208 EN**: Closes the current preprocessor conditional block or header guard.
  - **L208 CN**: 结束当前预处理条件块或头文件保护。
- **L209 EN**: Opens a new lexical scope or compound statement.
  - **L209 CN**: 打开一个新的词法作用域或复合语句块。
- **L210 EN**: Executes a call or declaration centered on `gamma_imp_final`.
  - **L210 CN**: 执行以 `gamma_imp_final` 为核心的调用或声明。
- **L211 EN**: Closes the current lexical scope or compound statement.
  - **L211 CN**: 结束当前词法作用域或复合语句块。
- **L212 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_NO_EXCEPTIONS`.
  - **L212 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_NO_EXCEPTIONS`。
- **L213 EN**: Starts an exception handler: `catch (const std::overflow_error&)`.
  - **L213 CN**: 开始一个异常处理器：`catch (const std::overflow_error&)`。
- **L214 EN**: Opens a new lexical scope or compound statement.
  - **L214 CN**: 打开一个新的词法作用域或复合语句块。
- **L215 EN**: Returns from the current function with `policies::raise_underflow_error<T>(function, "Result of tgamma is too small to represent.", pol)`.
  - **L215 CN**: 以 `policies::raise_underflow_error<T>(function, "Result of tgamma is too small to represent.", pol)` 从当前函数返回。
- **L216 EN**: Closes the current lexical scope or compound statement.
  - **L216 CN**: 结束当前词法作用域或复合语句块。
- **L217 EN**: Closes the current preprocessor conditional block or header guard.
  - **L217 CN**: 结束当前预处理条件块或头文件保护。
- **L218 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L218 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L219 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L219 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L220 EN**: Opens a new lexical scope or compound statement.
  - **L220 CN**: 打开一个新的词法作用域或复合语句块。
- **L221 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L221 CN**: 开始 `if` 控制流语句并计算其条件。
- **L222 EN**: Opens a new lexical scope or compound statement.
  - **L222 CN**: 打开一个新的词法作用域或复合语句块。
- **L223 EN**: Returns from the current function with `policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE MP only.`.
  - **L223 CN**: 以 `policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE MP only.` 从当前函数返回。
- **L224 EN**: Blank line separating nearby declarations or logic.
  - **L224 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 225-252 / 第 225-252 行

````cpp
 225:             }
 226:          }
 227:          else
 228:          {
 229:             // Result can never be small: tgamma[-z] is always larger than sinpx[z] is small.
 230:             // Specifically, sinpx can never be larger than 1 / epsilon which is too small to
 231:             // ever generate a value less than one for `result`, unless T has a truely
 232:             // exceptional number of digits precision.
 233:             BOOST_MATH_ASSERT((fabs(result) > 1) || (tools::max_value<T>() * fabs(result) > boost::math::constants::pi<T>()));
 234:          }
 235:          result = -boost::math::constants::pi<T>() / result;
 236:          if (result == 0)
 237:             return policies::raise_underflow_error<T>(function, "Result of tgamma is too small to represent.", pol);
 238:          /*
 239:          * Result can never be subnormal as we have a value > 1 in the numerator:
 240:          if((boost::math::fpclassify)(result) == (int)FP_SUBNORMAL)
 241:             return policies::raise_denorm_error<T>(function, "Result of tgamma is denormalized.", result, pol);
 242:             */
 243:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 244:          return result;
 245:       }
 246:    }
 247: 
 248:    return gamma_imp_final(T(z), pol, l);
 249: }
 250: 
 251: #ifdef BOOST_MATH_ENABLE_CUDA
 252: #  pragma nv_diag_default 2190
````
- **L225 EN**: Closes the current lexical scope or compound statement.
  - **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Closes the current lexical scope or compound statement.
  - **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Starts the alternative branch of the preceding conditional.
  - **L227 CN**: 开始前一个条件语句的备选分支。
- **L228 EN**: Opens a new lexical scope or compound statement.
  - **L228 CN**: 打开一个新的词法作用域或复合语句块。
- **L229 EN**: Comment documents nearby intent or usage notes: `Result can never be small: tgamma[-z] is always larger than sinpx[z] is small.`.
  - **L229 CN**: 注释说明附近代码的意图或使用说明：`Result can never be small: tgamma[-z] is always larger than sinpx[z] is small.`。
- **L230 EN**: Comment documents nearby intent or usage notes: `Specifically, sinpx can never be larger than 1 / epsilon which is too small to`.
  - **L230 CN**: 注释说明附近代码的意图或使用说明：`Specifically, sinpx can never be larger than 1 / epsilon which is too small to`。
- **L231 EN**: Comment documents nearby intent or usage notes: `ever generate a value less than one for `result`, unless T has a truely`.
  - **L231 CN**: 注释说明附近代码的意图或使用说明：`ever generate a value less than one for `result`, unless T has a truely`。
- **L232 EN**: Comment documents nearby intent or usage notes: `exceptional number of digits precision.`.
  - **L232 CN**: 注释说明附近代码的意图或使用说明：`exceptional number of digits precision.`。
- **L233 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L233 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L234 EN**: Closes the current lexical scope or compound statement.
  - **L234 CN**: 结束当前词法作用域或复合语句块。
- **L235 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L235 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L236 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L236 CN**: 开始 `if` 控制流语句并计算其条件。
- **L237 EN**: Returns from the current function with `policies::raise_underflow_error<T>(function, "Result of tgamma is too small to represent.", pol)`.
  - **L237 CN**: 以 `policies::raise_underflow_error<T>(function, "Result of tgamma is too small to represent.", pol)` 从当前函数返回。
- **L238 EN**: Separator comment used for visual grouping.
  - **L238 CN**: 分隔注释，用于视觉分组。
- **L239 EN**: Comment documents nearby intent or usage notes: `Result can never be subnormal as we have a value > 1 in the numerator:`.
  - **L239 CN**: 注释说明附近代码的意图或使用说明：`Result can never be subnormal as we have a value > 1 in the numerator:`。
- **L240 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L240 CN**: 开始 `if` 控制流语句并计算其条件。
- **L241 EN**: Returns from the current function with `policies::raise_denorm_error<T>(function, "Result of tgamma is denormalized.", result, pol)`.
  - **L241 CN**: 以 `policies::raise_denorm_error<T>(function, "Result of tgamma is denormalized.", result, pol)` 从当前函数返回。
- **L242 EN**: Comment documents nearby intent or usage notes: `/`.
  - **L242 CN**: 注释说明附近代码的意图或使用说明：`/`。
- **L243 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L243 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L244 EN**: Returns from the current function with `result`.
  - **L244 CN**: 以 `result` 从当前函数返回。
- **L245 EN**: Closes the current lexical scope or compound statement.
  - **L245 CN**: 结束当前词法作用域或复合语句块。
- **L246 EN**: Closes the current lexical scope or compound statement.
  - **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Blank line separating nearby declarations or logic.
  - **L247 CN**: 空行，用于分隔相邻声明或逻辑。
- **L248 EN**: Returns from the current function with `gamma_imp_final(T(z), pol, l)`.
  - **L248 CN**: 以 `gamma_imp_final(T(z), pol, l)` 从当前函数返回。
- **L249 EN**: Closes the current lexical scope or compound statement.
  - **L249 CN**: 结束当前词法作用域或复合语句块。
- **L250 EN**: Blank line separating nearby declarations or logic.
  - **L250 CN**: 空行，用于分隔相邻声明或逻辑。
- **L251 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_ENABLE_CUDA`.
  - **L251 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_ENABLE_CUDA`。
- **L252 EN**: Continues the surrounding expression or declaration: `#  pragma nv_diag_default 2190`.
  - **L252 CN**: 继续构造周围的表达式或声明：`#  pragma nv_diag_default 2190`。

### Lines 253-280 / 第 253-280 行

````cpp
 253: #endif
 254: 
 255: //
 256: // lgamma(z) with Lanczos support:
 257: //
 258: template <class T, class Policy, class Lanczos>
 259: BOOST_MATH_GPU_ENABLED T lgamma_imp_final(T z, const Policy& pol, const Lanczos& l, int* sign = nullptr)
 260: {
 261: #ifdef BOOST_MATH_INSTRUMENT
 262:    static bool b = false;
 263:    if(!b)
 264:    {
 265:       std::cout << "lgamma_imp called with " << typeid(z).name() << " " << typeid(l).name() << std::endl;
 266:       b = true;
 267:    }
 268: #endif
 269: 
 270:    BOOST_MATH_STD_USING
 271: 
 272:    constexpr auto function = "boost::math::lgamma<%1%>(%1%)";
 273: 
 274:    T result = 0;
 275:    int sresult = 1;
 276:    
 277:    if (z < tools::root_epsilon<T>())
 278:    {
 279:       if (0 == z)
 280:          return policies::raise_pole_error<T>(function, "Evaluation of lgamma at %1%.", z, pol);
````
- **L253 EN**: Closes the current preprocessor conditional block or header guard.
  - **L253 CN**: 结束当前预处理条件块或头文件保护。
- **L254 EN**: Blank line separating nearby declarations or logic.
  - **L254 CN**: 空行，用于分隔相邻声明或逻辑。
- **L255 EN**: Separator comment used for visual grouping.
  - **L255 CN**: 分隔注释，用于视觉分组。
- **L256 EN**: Comment documents nearby intent or usage notes: `lgamma(z) with Lanczos support:`.
  - **L256 CN**: 注释说明附近代码的意图或使用说明：`lgamma(z) with Lanczos support:`。
- **L257 EN**: Separator comment used for visual grouping.
  - **L257 CN**: 分隔注释，用于视觉分组。
- **L258 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L258 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L259 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L259 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L260 EN**: Opens a new lexical scope or compound statement.
  - **L260 CN**: 打开一个新的词法作用域或复合语句块。
- **L261 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_INSTRUMENT`.
  - **L261 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_INSTRUMENT`。
- **L262 EN**: Initializes variable `b` from the right-hand expression.
  - **L262 CN**: 使用右侧表达式初始化变量 `b`。
- **L263 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L263 CN**: 开始 `if` 控制流语句并计算其条件。
- **L264 EN**: Opens a new lexical scope or compound statement.
  - **L264 CN**: 打开一个新的词法作用域或复合语句块。
- **L265 EN**: Executes a call or declaration centered on `typeid`.
  - **L265 CN**: 执行以 `typeid` 为核心的调用或声明。
- **L266 EN**: Executes a standalone statement or declaration: `b = true;`.
  - **L266 CN**: 执行一条独立语句或声明：`b = true;`。
- **L267 EN**: Closes the current lexical scope or compound statement.
  - **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Closes the current preprocessor conditional block or header guard.
  - **L268 CN**: 结束当前预处理条件块或头文件保护。
- **L269 EN**: Blank line separating nearby declarations or logic.
  - **L269 CN**: 空行，用于分隔相邻声明或逻辑。
- **L270 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L270 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L271 EN**: Blank line separating nearby declarations or logic.
  - **L271 CN**: 空行，用于分隔相邻声明或逻辑。
- **L272 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L272 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L273 EN**: Blank line separating nearby declarations or logic.
  - **L273 CN**: 空行，用于分隔相邻声明或逻辑。
- **L274 EN**: Executes a standalone statement or declaration: `T result = 0;`.
  - **L274 CN**: 执行一条独立语句或声明：`T result = 0;`。
- **L275 EN**: Initializes variable `sresult` from the right-hand expression.
  - **L275 CN**: 使用右侧表达式初始化变量 `sresult`。
- **L276 EN**: Blank line separating nearby declarations or logic.
  - **L276 CN**: 空行，用于分隔相邻声明或逻辑。
- **L277 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L277 CN**: 开始 `if` 控制流语句并计算其条件。
- **L278 EN**: Opens a new lexical scope or compound statement.
  - **L278 CN**: 打开一个新的词法作用域或复合语句块。
- **L279 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L279 CN**: 开始 `if` 控制流语句并计算其条件。
- **L280 EN**: Returns from the current function with `policies::raise_pole_error<T>(function, "Evaluation of lgamma at %1%.", z, pol)`.
  - **L280 CN**: 以 `policies::raise_pole_error<T>(function, "Evaluation of lgamma at %1%.", z, pol)` 从当前函数返回。

### Lines 281-308 / 第 281-308 行

````cpp
 281:       if (4 * fabs(z) < tools::epsilon<T>())
 282:          result = -log(fabs(z));
 283:       else
 284:          result = log(fabs(1 / z - constants::euler<T>()));
 285:       if (z < 0)
 286:          sresult = -1;
 287:    }
 288:    else if(z < 15)
 289:    {
 290:       typedef typename policies::precision<T, Policy>::type precision_type;
 291:       typedef boost::math::integral_constant<int,
 292:          precision_type::value <= 0 ? 0 :
 293:          precision_type::value <= 64 ? 64 :
 294:          precision_type::value <= 113 ? 113 : 0
 295:       > tag_type;
 296: 
 297:       result = lgamma_small_imp<T>(z, T(z - 1), T(z - 2), tag_type(), pol, l);
 298:    }
 299:    else if((z >= 3) && (z < 100) && (boost::math::numeric_limits<T>::max_exponent >= 1024))
 300:    {
 301:       // taking the log of tgamma reduces the error, no danger of overflow here:
 302:       result = log(gamma_imp(z, pol, l));
 303:    }
 304:    else
 305:    {
 306:       // regular evaluation:
 307:       T zgh = static_cast<T>(z + T(Lanczos::g()) - boost::math::constants::half<T>());
 308:       result = log(zgh) - 1;
````
- **L281 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L281 CN**: 开始 `if` 控制流语句并计算其条件。
- **L282 EN**: Executes a call or declaration centered on `-log`.
  - **L282 CN**: 执行以 `-log` 为核心的调用或声明。
- **L283 EN**: Starts the alternative branch of the preceding conditional.
  - **L283 CN**: 开始前一个条件语句的备选分支。
- **L284 EN**: Executes a call or declaration centered on `log`.
  - **L284 CN**: 执行以 `log` 为核心的调用或声明。
- **L285 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L285 CN**: 开始 `if` 控制流语句并计算其条件。
- **L286 EN**: Executes a standalone statement or declaration: `sresult = -1;`.
  - **L286 CN**: 执行一条独立语句或声明：`sresult = -1;`。
- **L287 EN**: Closes the current lexical scope or compound statement.
  - **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Starts the alternative branch of the preceding conditional.
  - **L288 CN**: 开始前一个条件语句的备选分支。
- **L289 EN**: Opens a new lexical scope or compound statement.
  - **L289 CN**: 打开一个新的词法作用域或复合语句块。
- **L290 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::precision<T, Policy>::type precision_type;`.
  - **L290 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::precision<T, Policy>::type precision_type;`。
- **L291 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int,`.
  - **L291 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int,`。
- **L292 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 0 ? 0 :`.
  - **L292 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 0 ? 0 :`。
- **L293 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 64 ? 64 :`.
  - **L293 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 64 ? 64 :`。
- **L294 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 113 ? 113 : 0`.
  - **L294 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 113 ? 113 : 0`。
- **L295 EN**: Executes a standalone statement or declaration: `> tag_type;`.
  - **L295 CN**: 执行一条独立语句或声明：`> tag_type;`。
- **L296 EN**: Blank line separating nearby declarations or logic.
  - **L296 CN**: 空行，用于分隔相邻声明或逻辑。
- **L297 EN**: Executes a call or declaration centered on `lgamma_small_imp<T>`.
  - **L297 CN**: 执行以 `lgamma_small_imp<T>` 为核心的调用或声明。
- **L298 EN**: Closes the current lexical scope or compound statement.
  - **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Starts the alternative branch of the preceding conditional.
  - **L299 CN**: 开始前一个条件语句的备选分支。
- **L300 EN**: Opens a new lexical scope or compound statement.
  - **L300 CN**: 打开一个新的词法作用域或复合语句块。
- **L301 EN**: Comment documents nearby intent or usage notes: `taking the log of tgamma reduces the error, no danger of overflow here:`.
  - **L301 CN**: 注释说明附近代码的意图或使用说明：`taking the log of tgamma reduces the error, no danger of overflow here:`。
- **L302 EN**: Executes a call or declaration centered on `log`.
  - **L302 CN**: 执行以 `log` 为核心的调用或声明。
- **L303 EN**: Closes the current lexical scope or compound statement.
  - **L303 CN**: 结束当前词法作用域或复合语句块。
- **L304 EN**: Starts the alternative branch of the preceding conditional.
  - **L304 CN**: 开始前一个条件语句的备选分支。
- **L305 EN**: Opens a new lexical scope or compound statement.
  - **L305 CN**: 打开一个新的词法作用域或复合语句块。
- **L306 EN**: Comment documents nearby intent or usage notes: `regular evaluation:`.
  - **L306 CN**: 注释说明附近代码的意图或使用说明：`regular evaluation:`。
- **L307 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L307 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L308 EN**: Executes a call or declaration centered on `log`.
  - **L308 CN**: 执行以 `log` 为核心的调用或声明。

### Lines 309-336 / 第 309-336 行

````cpp
 309:       result *= z - 0.5f;
 310:       //
 311:       // Only add on the lanczos sum part if we're going to need it:
 312:       //
 313:       if(result * tools::epsilon<T>() < 20)
 314:          result += log(Lanczos::lanczos_sum_expG_scaled(z));
 315:    }
 316: 
 317:    if(sign)
 318:       *sign = sresult;
 319:    return result;
 320: }
 321: 
 322: #ifdef BOOST_MATH_ENABLE_CUDA
 323: #  pragma nv_diag_suppress 2190
 324: #endif
 325: 
 326: template <class T, class Policy, class Lanczos>
 327: BOOST_MATH_GPU_ENABLED BOOST_MATH_FORCEINLINE T lgamma_imp(T z, const Policy& pol, const Lanczos& l, int* sign = nullptr)
 328: {
 329:    BOOST_MATH_STD_USING
 330: 
 331:    if(z <= -tools::root_epsilon<T>())
 332:    {
 333:       constexpr auto function = "boost::math::lgamma<%1%>(%1%)";
 334: 
 335:       T result = 0;
 336:       int sresult = 1;
````
- **L309 EN**: Executes a standalone statement or declaration: `result *= z - 0.5f;`.
  - **L309 CN**: 执行一条独立语句或声明：`result *= z - 0.5f;`。
- **L310 EN**: Separator comment used for visual grouping.
  - **L310 CN**: 分隔注释，用于视觉分组。
- **L311 EN**: Comment documents nearby intent or usage notes: `Only add on the lanczos sum part if we're going to need it:`.
  - **L311 CN**: 注释说明附近代码的意图或使用说明：`Only add on the lanczos sum part if we're going to need it:`。
- **L312 EN**: Separator comment used for visual grouping.
  - **L312 CN**: 分隔注释，用于视觉分组。
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Executes a call or declaration centered on `log`.
  - **L314 CN**: 执行以 `log` 为核心的调用或声明。
- **L315 EN**: Closes the current lexical scope or compound statement.
  - **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Blank line separating nearby declarations or logic.
  - **L316 CN**: 空行，用于分隔相邻声明或逻辑。
- **L317 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L317 CN**: 开始 `if` 控制流语句并计算其条件。
- **L318 EN**: Comment documents nearby intent or usage notes: `sign = sresult;`.
  - **L318 CN**: 注释说明附近代码的意图或使用说明：`sign = sresult;`。
- **L319 EN**: Returns from the current function with `result`.
  - **L319 CN**: 以 `result` 从当前函数返回。
- **L320 EN**: Closes the current lexical scope or compound statement.
  - **L320 CN**: 结束当前词法作用域或复合语句块。
- **L321 EN**: Blank line separating nearby declarations or logic.
  - **L321 CN**: 空行，用于分隔相邻声明或逻辑。
- **L322 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_ENABLE_CUDA`.
  - **L322 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_ENABLE_CUDA`。
- **L323 EN**: Continues the surrounding expression or declaration: `#  pragma nv_diag_suppress 2190`.
  - **L323 CN**: 继续构造周围的表达式或声明：`#  pragma nv_diag_suppress 2190`。
- **L324 EN**: Closes the current preprocessor conditional block or header guard.
  - **L324 CN**: 结束当前预处理条件块或头文件保护。
- **L325 EN**: Blank line separating nearby declarations or logic.
  - **L325 CN**: 空行，用于分隔相邻声明或逻辑。
- **L326 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L326 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L327 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L327 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L328 EN**: Opens a new lexical scope or compound statement.
  - **L328 CN**: 打开一个新的词法作用域或复合语句块。
- **L329 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L329 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L330 EN**: Blank line separating nearby declarations or logic.
  - **L330 CN**: 空行，用于分隔相邻声明或逻辑。
- **L331 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L331 CN**: 开始 `if` 控制流语句并计算其条件。
- **L332 EN**: Opens a new lexical scope or compound statement.
  - **L332 CN**: 打开一个新的词法作用域或复合语句块。
- **L333 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L333 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L334 EN**: Blank line separating nearby declarations or logic.
  - **L334 CN**: 空行，用于分隔相邻声明或逻辑。
- **L335 EN**: Executes a standalone statement or declaration: `T result = 0;`.
  - **L335 CN**: 执行一条独立语句或声明：`T result = 0;`。
- **L336 EN**: Initializes variable `sresult` from the right-hand expression.
  - **L336 CN**: 使用右侧表达式初始化变量 `sresult`。

### Lines 337-364 / 第 337-364 行

````cpp
 337: 
 338:       // reflection formula:
 339:       if(floor(z) == z)
 340:          return policies::raise_pole_error<T>(function, "Evaluation of lgamma at a negative integer %1%.", z, pol);
 341: 
 342:       T t = sinpx(z);
 343:       z = -z;
 344:       if(t < 0)
 345:       {
 346:          t = -t;
 347:       }
 348:       else
 349:       {
 350:          sresult = -sresult;
 351:       }
 352:       result = log(boost::math::constants::pi<T>()) - lgamma_imp_final(T(z), pol, l) - log(t);
 353: 
 354:       if(sign)
 355:       {
 356:          *sign = sresult;
 357:       }
 358: 
 359:       return result;
 360:    }
 361:    else
 362:    {
 363:       return lgamma_imp_final(T(z), pol, l, sign);
 364:    }
````
- **L337 EN**: Blank line separating nearby declarations or logic.
  - **L337 CN**: 空行，用于分隔相邻声明或逻辑。
- **L338 EN**: Comment documents nearby intent or usage notes: `reflection formula:`.
  - **L338 CN**: 注释说明附近代码的意图或使用说明：`reflection formula:`。
- **L339 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L339 CN**: 开始 `if` 控制流语句并计算其条件。
- **L340 EN**: Returns from the current function with `policies::raise_pole_error<T>(function, "Evaluation of lgamma at a negative integer %1%.", z, pol)`.
  - **L340 CN**: 以 `policies::raise_pole_error<T>(function, "Evaluation of lgamma at a negative integer %1%.", z, pol)` 从当前函数返回。
- **L341 EN**: Blank line separating nearby declarations or logic.
  - **L341 CN**: 空行，用于分隔相邻声明或逻辑。
- **L342 EN**: Executes a call or declaration centered on `sinpx`.
  - **L342 CN**: 执行以 `sinpx` 为核心的调用或声明。
- **L343 EN**: Executes a standalone statement or declaration: `z = -z;`.
  - **L343 CN**: 执行一条独立语句或声明：`z = -z;`。
- **L344 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L344 CN**: 开始 `if` 控制流语句并计算其条件。
- **L345 EN**: Opens a new lexical scope or compound statement.
  - **L345 CN**: 打开一个新的词法作用域或复合语句块。
- **L346 EN**: Executes a standalone statement or declaration: `t = -t;`.
  - **L346 CN**: 执行一条独立语句或声明：`t = -t;`。
- **L347 EN**: Closes the current lexical scope or compound statement.
  - **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Starts the alternative branch of the preceding conditional.
  - **L348 CN**: 开始前一个条件语句的备选分支。
- **L349 EN**: Opens a new lexical scope or compound statement.
  - **L349 CN**: 打开一个新的词法作用域或复合语句块。
- **L350 EN**: Executes a standalone statement or declaration: `sresult = -sresult;`.
  - **L350 CN**: 执行一条独立语句或声明：`sresult = -sresult;`。
- **L351 EN**: Closes the current lexical scope or compound statement.
  - **L351 CN**: 结束当前词法作用域或复合语句块。
- **L352 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L352 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L353 EN**: Blank line separating nearby declarations or logic.
  - **L353 CN**: 空行，用于分隔相邻声明或逻辑。
- **L354 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L354 CN**: 开始 `if` 控制流语句并计算其条件。
- **L355 EN**: Opens a new lexical scope or compound statement.
  - **L355 CN**: 打开一个新的词法作用域或复合语句块。
- **L356 EN**: Comment documents nearby intent or usage notes: `sign = sresult;`.
  - **L356 CN**: 注释说明附近代码的意图或使用说明：`sign = sresult;`。
- **L357 EN**: Closes the current lexical scope or compound statement.
  - **L357 CN**: 结束当前词法作用域或复合语句块。
- **L358 EN**: Blank line separating nearby declarations or logic.
  - **L358 CN**: 空行，用于分隔相邻声明或逻辑。
- **L359 EN**: Returns from the current function with `result`.
  - **L359 CN**: 以 `result` 从当前函数返回。
- **L360 EN**: Closes the current lexical scope or compound statement.
  - **L360 CN**: 结束当前词法作用域或复合语句块。
- **L361 EN**: Starts the alternative branch of the preceding conditional.
  - **L361 CN**: 开始前一个条件语句的备选分支。
- **L362 EN**: Opens a new lexical scope or compound statement.
  - **L362 CN**: 打开一个新的词法作用域或复合语句块。
- **L363 EN**: Returns from the current function with `lgamma_imp_final(T(z), pol, l, sign)`.
  - **L363 CN**: 以 `lgamma_imp_final(T(z), pol, l, sign)` 从当前函数返回。
- **L364 EN**: Closes the current lexical scope or compound statement.
  - **L364 CN**: 结束当前词法作用域或复合语句块。

### Lines 365-392 / 第 365-392 行

````cpp
 365: }
 366: 
 367: #ifdef BOOST_MATH_ENABLE_CUDA
 368: #  pragma nv_diag_default 2190
 369: #endif
 370: 
 371: //
 372: // Incomplete gamma functions follow:
 373: //
 374: template <class T>
 375: struct upper_incomplete_gamma_fract
 376: {
 377: private:
 378:    T z, a;
 379:    int k;
 380: public:
 381:    typedef boost::math::pair<T,T> result_type;
 382: 
 383:    BOOST_MATH_GPU_ENABLED upper_incomplete_gamma_fract(T a1, T z1)
 384:       : z(z1-a1+1), a(a1), k(0)
 385:    {
 386:    }
 387: 
 388:    BOOST_MATH_GPU_ENABLED result_type operator()()
 389:    {
 390:       ++k;
 391:       z += 2;
 392:       return result_type(k * (a - k), z);
````
- **L365 EN**: Closes the current lexical scope or compound statement.
  - **L365 CN**: 结束当前词法作用域或复合语句块。
- **L366 EN**: Blank line separating nearby declarations or logic.
  - **L366 CN**: 空行，用于分隔相邻声明或逻辑。
- **L367 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_ENABLE_CUDA`.
  - **L367 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_ENABLE_CUDA`。
- **L368 EN**: Continues the surrounding expression or declaration: `#  pragma nv_diag_default 2190`.
  - **L368 CN**: 继续构造周围的表达式或声明：`#  pragma nv_diag_default 2190`。
- **L369 EN**: Closes the current preprocessor conditional block or header guard.
  - **L369 CN**: 结束当前预处理条件块或头文件保护。
- **L370 EN**: Blank line separating nearby declarations or logic.
  - **L370 CN**: 空行，用于分隔相邻声明或逻辑。
- **L371 EN**: Separator comment used for visual grouping.
  - **L371 CN**: 分隔注释，用于视觉分组。
- **L372 EN**: Comment documents nearby intent or usage notes: `Incomplete gamma functions follow:`.
  - **L372 CN**: 注释说明附近代码的意图或使用说明：`Incomplete gamma functions follow:`。
- **L373 EN**: Separator comment used for visual grouping.
  - **L373 CN**: 分隔注释，用于视觉分组。
- **L374 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L374 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L375 EN**: Declares struct `upper_incomplete_gamma_fract`.
  - **L375 CN**: 声明 struct `upper_incomplete_gamma_fract`。
- **L376 EN**: Opens a new lexical scope or compound statement.
  - **L376 CN**: 打开一个新的词法作用域或复合语句块。
- **L377 EN**: Sets the following members to `private` access.
  - **L377 CN**: 将后续成员的访问级别设为 `private`。
- **L378 EN**: Executes a standalone statement or declaration: `T z, a;`.
  - **L378 CN**: 执行一条独立语句或声明：`T z, a;`。
- **L379 EN**: Executes a standalone statement or declaration: `int k;`.
  - **L379 CN**: 执行一条独立语句或声明：`int k;`。
- **L380 EN**: Sets the following members to `public` access.
  - **L380 CN**: 将后续成员的访问级别设为 `public`。
- **L381 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::pair<T,T> result_type;`.
  - **L381 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::pair<T,T> result_type;`。
- **L382 EN**: Blank line separating nearby declarations or logic.
  - **L382 CN**: 空行，用于分隔相邻声明或逻辑。
- **L383 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L383 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L384 EN**: Continues logic associated with callable symbol `z`.
  - **L384 CN**: 继续与可调用符号 `z` 相关的逻辑。
- **L385 EN**: Opens a new lexical scope or compound statement.
  - **L385 CN**: 打开一个新的词法作用域或复合语句块。
- **L386 EN**: Closes the current lexical scope or compound statement.
  - **L386 CN**: 结束当前词法作用域或复合语句块。
- **L387 EN**: Blank line separating nearby declarations or logic.
  - **L387 CN**: 空行，用于分隔相邻声明或逻辑。
- **L388 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L388 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L389 EN**: Opens a new lexical scope or compound statement.
  - **L389 CN**: 打开一个新的词法作用域或复合语句块。
- **L390 EN**: Executes a standalone statement or declaration: `++k;`.
  - **L390 CN**: 执行一条独立语句或声明：`++k;`。
- **L391 EN**: Executes a standalone statement or declaration: `z += 2;`.
  - **L391 CN**: 执行一条独立语句或声明：`z += 2;`。
- **L392 EN**: Returns from the current function with `result_type(k * (a - k), z)`.
  - **L392 CN**: 以 `result_type(k * (a - k), z)` 从当前函数返回。

### Lines 393-420 / 第 393-420 行

````cpp
 393:    }
 394: };
 395: 
 396: template <class T>
 397: BOOST_MATH_GPU_ENABLED inline T upper_gamma_fraction(T a, T z, T eps)
 398: {
 399:    // Multiply result by z^a * e^-z to get the full
 400:    // upper incomplete integral.  Divide by tgamma(z)
 401:    // to normalise.
 402:    upper_incomplete_gamma_fract<T> f(a, z);
 403:    return 1 / (z - a + 1 + boost::math::tools::continued_fraction_a(f, eps));
 404: }
 405: 
 406: template <class T>
 407: struct lower_incomplete_gamma_series
 408: {
 409: private:
 410:    T a, z, result;
 411: public:
 412:    typedef T result_type;
 413:    BOOST_MATH_GPU_ENABLED lower_incomplete_gamma_series(T a1, T z1) : a(a1), z(z1), result(1){}
 414: 
 415:    BOOST_MATH_GPU_ENABLED T operator()()
 416:    {
 417:       T r = result;
 418:       a += 1;
 419:       result *= z/a;
 420:       return r;
````
- **L393 EN**: Closes the current lexical scope or compound statement.
  - **L393 CN**: 结束当前词法作用域或复合语句块。
- **L394 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L394 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L395 EN**: Blank line separating nearby declarations or logic.
  - **L395 CN**: 空行，用于分隔相邻声明或逻辑。
- **L396 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L396 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L397 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L397 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L398 EN**: Opens a new lexical scope or compound statement.
  - **L398 CN**: 打开一个新的词法作用域或复合语句块。
- **L399 EN**: Comment documents nearby intent or usage notes: `Multiply result by z^a * e^-z to get the full`.
  - **L399 CN**: 注释说明附近代码的意图或使用说明：`Multiply result by z^a * e^-z to get the full`。
- **L400 EN**: Comment documents nearby intent or usage notes: `upper incomplete integral.  Divide by tgamma(z)`.
  - **L400 CN**: 注释说明附近代码的意图或使用说明：`upper incomplete integral.  Divide by tgamma(z)`。
- **L401 EN**: Comment documents nearby intent or usage notes: `to normalise.`.
  - **L401 CN**: 注释说明附近代码的意图或使用说明：`to normalise.`。
- **L402 EN**: Executes a call or declaration centered on `f`.
  - **L402 CN**: 执行以 `f` 为核心的调用或声明。
- **L403 EN**: Returns from the current function with `1 / (z - a + 1 + boost::math::tools::continued_fraction_a(f, eps))`.
  - **L403 CN**: 以 `1 / (z - a + 1 + boost::math::tools::continued_fraction_a(f, eps))` 从当前函数返回。
- **L404 EN**: Closes the current lexical scope or compound statement.
  - **L404 CN**: 结束当前词法作用域或复合语句块。
- **L405 EN**: Blank line separating nearby declarations or logic.
  - **L405 CN**: 空行，用于分隔相邻声明或逻辑。
- **L406 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L406 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L407 EN**: Declares struct `lower_incomplete_gamma_series`.
  - **L407 CN**: 声明 struct `lower_incomplete_gamma_series`。
- **L408 EN**: Opens a new lexical scope or compound statement.
  - **L408 CN**: 打开一个新的词法作用域或复合语句块。
- **L409 EN**: Sets the following members to `private` access.
  - **L409 CN**: 将后续成员的访问级别设为 `private`。
- **L410 EN**: Executes a standalone statement or declaration: `T a, z, result;`.
  - **L410 CN**: 执行一条独立语句或声明：`T a, z, result;`。
- **L411 EN**: Sets the following members to `public` access.
  - **L411 CN**: 将后续成员的访问级别设为 `public`。
- **L412 EN**: Introduces a legacy type alias or function typedef: `typedef T result_type;`.
  - **L412 CN**: 引入传统类型别名或函数 typedef：`typedef T result_type;`。
- **L413 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L413 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L414 EN**: Blank line separating nearby declarations or logic.
  - **L414 CN**: 空行，用于分隔相邻声明或逻辑。
- **L415 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L415 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L416 EN**: Opens a new lexical scope or compound statement.
  - **L416 CN**: 打开一个新的词法作用域或复合语句块。
- **L417 EN**: Executes a standalone statement or declaration: `T r = result;`.
  - **L417 CN**: 执行一条独立语句或声明：`T r = result;`。
- **L418 EN**: Executes a standalone statement or declaration: `a += 1;`.
  - **L418 CN**: 执行一条独立语句或声明：`a += 1;`。
- **L419 EN**: Executes a standalone statement or declaration: `result *= z/a;`.
  - **L419 CN**: 执行一条独立语句或声明：`result *= z/a;`。
- **L420 EN**: Returns from the current function with `r`.
  - **L420 CN**: 以 `r` 从当前函数返回。

### Lines 421-448 / 第 421-448 行

````cpp
 421:    }
 422: };
 423: 
 424: template <class T, class Policy>
 425: BOOST_MATH_GPU_ENABLED inline T lower_gamma_series(T a, T z, const Policy& pol, T init_value = 0)
 426: {
 427:    // Multiply result by ((z^a) * (e^-z) / a) to get the full
 428:    // lower incomplete integral. Then divide by tgamma(a)
 429:    // to get the normalised value.
 430:    lower_incomplete_gamma_series<T> s(a, z);
 431:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 432:    T factor = policies::get_epsilon<T, Policy>();
 433:    T result = boost::math::tools::sum_series(s, factor, max_iter, init_value);
 434:    policies::check_series_iterations<T>("boost::math::detail::lower_gamma_series<%1%>(%1%)", max_iter, pol);
 435:    return result;
 436: }
 437: 
 438: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 439: 
 440: //
 441: // Fully generic tgamma and lgamma use Stirling's approximation
 442: // with Bernoulli numbers.
 443: //
 444: template<class T>
 445: boost::math::size_t highest_bernoulli_index()
 446: {
 447:    const float digits10_of_type = (boost::math::numeric_limits<T>::is_specialized
 448:                                       ? static_cast<float>(boost::math::numeric_limits<T>::digits10)
````
- **L421 EN**: Closes the current lexical scope or compound statement.
  - **L421 CN**: 结束当前词法作用域或复合语句块。
- **L422 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L422 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L423 EN**: Blank line separating nearby declarations or logic.
  - **L423 CN**: 空行，用于分隔相邻声明或逻辑。
- **L424 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L424 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L425 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L425 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L426 EN**: Opens a new lexical scope or compound statement.
  - **L426 CN**: 打开一个新的词法作用域或复合语句块。
- **L427 EN**: Comment documents nearby intent or usage notes: `Multiply result by ((z^a) * (e^-z) / a) to get the full`.
  - **L427 CN**: 注释说明附近代码的意图或使用说明：`Multiply result by ((z^a) * (e^-z) / a) to get the full`。
- **L428 EN**: Comment documents nearby intent or usage notes: `lower incomplete integral. Then divide by tgamma(a)`.
  - **L428 CN**: 注释说明附近代码的意图或使用说明：`lower incomplete integral. Then divide by tgamma(a)`。
- **L429 EN**: Comment documents nearby intent or usage notes: `to get the normalised value.`.
  - **L429 CN**: 注释说明附近代码的意图或使用说明：`to get the normalised value.`。
- **L430 EN**: Executes a call or declaration centered on `s`.
  - **L430 CN**: 执行以 `s` 为核心的调用或声明。
- **L431 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L431 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L432 EN**: Executes a call or declaration centered on `Policy>`.
  - **L432 CN**: 执行以 `Policy>` 为核心的调用或声明。
- **L433 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L433 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L434 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L434 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L435 EN**: Returns from the current function with `result`.
  - **L435 CN**: 以 `result` 从当前函数返回。
- **L436 EN**: Closes the current lexical scope or compound statement.
  - **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Blank line separating nearby declarations or logic.
  - **L437 CN**: 空行，用于分隔相邻声明或逻辑。
- **L438 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_HAS_GPU_SUPPORT`.
  - **L438 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_HAS_GPU_SUPPORT`。
- **L439 EN**: Blank line separating nearby declarations or logic.
  - **L439 CN**: 空行，用于分隔相邻声明或逻辑。
- **L440 EN**: Separator comment used for visual grouping.
  - **L440 CN**: 分隔注释，用于视觉分组。
- **L441 EN**: Comment documents nearby intent or usage notes: `Fully generic tgamma and lgamma use Stirling's approximation`.
  - **L441 CN**: 注释说明附近代码的意图或使用说明：`Fully generic tgamma and lgamma use Stirling's approximation`。
- **L442 EN**: Comment documents nearby intent or usage notes: `with Bernoulli numbers.`.
  - **L442 CN**: 注释说明附近代码的意图或使用说明：`with Bernoulli numbers.`。
- **L443 EN**: Separator comment used for visual grouping.
  - **L443 CN**: 分隔注释，用于视觉分组。
- **L444 EN**: Introduces template parameters or specialization context: `template<class T>`.
  - **L444 CN**: 为后续声明引入模板参数或特化上下文：`template<class T>`。
- **L445 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L445 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L446 EN**: Opens a new lexical scope or compound statement.
  - **L446 CN**: 打开一个新的词法作用域或复合语句块。
- **L447 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L447 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L448 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L448 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 449-476 / 第 449-476 行

````cpp
 449:                                       : static_cast<float>(boost::math::tools::digits<T>() * 0.301F));
 450: 
 451:    // Find the high index n for Bn to produce the desired precision in Stirling's calculation.
 452:    return static_cast<boost::math::size_t>(18.0F + (0.6F * digits10_of_type));
 453: }
 454: 
 455: template<class T>
 456: int minimum_argument_for_bernoulli_recursion()
 457: {
 458:    BOOST_MATH_STD_USING
 459: 
 460:    const float digits10_of_type = (boost::math::numeric_limits<T>::is_specialized
 461:                                     ? (float) boost::math::numeric_limits<T>::digits10
 462:                                     : (float) (boost::math::tools::digits<T>() * 0.301F));
 463: 
 464:    int min_arg = (int) (digits10_of_type * 1.7F);
 465: 
 466:    if(digits10_of_type < 50.0F)
 467:    {
 468:       // The following code sequence has been modified
 469:       // within the context of issue 396.
 470: 
 471:       // The calculation of the test-variable limit has now
 472:       // been protected against overflow/underflow dangers.
 473: 
 474:       // The previous line looked like this and did, in fact,
 475:       // underflow ldexp when using certain multiprecision types.
 476: 
````
- **L449 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L449 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L450 EN**: Blank line separating nearby declarations or logic.
  - **L450 CN**: 空行，用于分隔相邻声明或逻辑。
- **L451 EN**: Comment documents nearby intent or usage notes: `Find the high index n for Bn to produce the desired precision in Stirling's calculation.`.
  - **L451 CN**: 注释说明附近代码的意图或使用说明：`Find the high index n for Bn to produce the desired precision in Stirling's calculation.`。
- **L452 EN**: Returns from the current function with `static_cast<boost::math::size_t>(18.0F + (0.6F * digits10_of_type))`.
  - **L452 CN**: 以 `static_cast<boost::math::size_t>(18.0F + (0.6F * digits10_of_type))` 从当前函数返回。
- **L453 EN**: Closes the current lexical scope or compound statement.
  - **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Blank line separating nearby declarations or logic.
  - **L454 CN**: 空行，用于分隔相邻声明或逻辑。
- **L455 EN**: Introduces template parameters or specialization context: `template<class T>`.
  - **L455 CN**: 为后续声明引入模板参数或特化上下文：`template<class T>`。
- **L456 EN**: Continues logic associated with callable symbol `minimum_argument_for_bernoulli_recursion`.
  - **L456 CN**: 继续与可调用符号 `minimum_argument_for_bernoulli_recursion` 相关的逻辑。
- **L457 EN**: Opens a new lexical scope or compound statement.
  - **L457 CN**: 打开一个新的词法作用域或复合语句块。
- **L458 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L458 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L459 EN**: Blank line separating nearby declarations or logic.
  - **L459 CN**: 空行，用于分隔相邻声明或逻辑。
- **L460 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L460 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L461 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L461 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L462 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L462 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L463 EN**: Blank line separating nearby declarations or logic.
  - **L463 CN**: 空行，用于分隔相邻声明或逻辑。
- **L464 EN**: Initializes variable `min_arg` from the right-hand expression.
  - **L464 CN**: 使用右侧表达式初始化变量 `min_arg`。
- **L465 EN**: Blank line separating nearby declarations or logic.
  - **L465 CN**: 空行，用于分隔相邻声明或逻辑。
- **L466 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L466 CN**: 开始 `if` 控制流语句并计算其条件。
- **L467 EN**: Opens a new lexical scope or compound statement.
  - **L467 CN**: 打开一个新的词法作用域或复合语句块。
- **L468 EN**: Comment documents nearby intent or usage notes: `The following code sequence has been modified`.
  - **L468 CN**: 注释说明附近代码的意图或使用说明：`The following code sequence has been modified`。
- **L469 EN**: Comment documents nearby intent or usage notes: `within the context of issue 396.`.
  - **L469 CN**: 注释说明附近代码的意图或使用说明：`within the context of issue 396.`。
- **L470 EN**: Blank line separating nearby declarations or logic.
  - **L470 CN**: 空行，用于分隔相邻声明或逻辑。
- **L471 EN**: Comment documents nearby intent or usage notes: `The calculation of the test-variable limit has now`.
  - **L471 CN**: 注释说明附近代码的意图或使用说明：`The calculation of the test-variable limit has now`。
- **L472 EN**: Comment documents nearby intent or usage notes: `been protected against overflow/underflow dangers.`.
  - **L472 CN**: 注释说明附近代码的意图或使用说明：`been protected against overflow/underflow dangers.`。
- **L473 EN**: Blank line separating nearby declarations or logic.
  - **L473 CN**: 空行，用于分隔相邻声明或逻辑。
- **L474 EN**: Comment documents nearby intent or usage notes: `The previous line looked like this and did, in fact,`.
  - **L474 CN**: 注释说明附近代码的意图或使用说明：`The previous line looked like this and did, in fact,`。
- **L475 EN**: Comment documents nearby intent or usage notes: `underflow ldexp when using certain multiprecision types.`.
  - **L475 CN**: 注释说明附近代码的意图或使用说明：`underflow ldexp when using certain multiprecision types.`。
- **L476 EN**: Blank line separating nearby declarations or logic.
  - **L476 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 477-504 / 第 477-504 行

````cpp
 477:       // const float limit = std::ceil(std::pow(1.0f / std::ldexp(1.0f, 1-boost::math::tools::digits<T>()), 1.0f / 20.0f));
 478: 
 479:       // The new safe version of the limit check is now here.
 480:       const float d2_minus_one = ((digits10_of_type / 0.301F) - 1.0F);
 481:       const float limit        = ceil(exp((d2_minus_one * log(2.0F)) / 20.0F));
 482: 
 483:       min_arg = (int) (BOOST_MATH_GPU_SAFE_MIN(digits10_of_type * 1.7F, limit));
 484:    }
 485: 
 486:    return min_arg;
 487: }
 488: 
 489: template <class T, class Policy>
 490: T scaled_tgamma_no_lanczos(const T& z, const Policy& pol, bool islog = false)
 491: {
 492:    BOOST_MATH_STD_USING
 493:    //
 494:    // Calculates tgamma(z) / (z/e)^z
 495:    // Requires that our argument is large enough for Sterling's approximation to hold.
 496:    // Used internally when combining gamma's of similar magnitude without logarithms.
 497:    //
 498:    BOOST_MATH_ASSERT(minimum_argument_for_bernoulli_recursion<T>() <= z);
 499: 
 500:    // Perform the Bernoulli series expansion of Stirling's approximation.
 501: 
 502:    const boost::math::size_t number_of_bernoullis_b2n = policies::get_max_series_iterations<Policy>();
 503: 
 504:    T one_over_x_pow_two_n_minus_one = 1 / z;
````
- **L477 EN**: Comment documents nearby intent or usage notes: `const float limit = std::ceil(std::pow(1.0f / std::ldexp(1.0f, 1-boost::math::tools::digits<T>()), 1.0f / 20.0f));`.
  - **L477 CN**: 注释说明附近代码的意图或使用说明：`const float limit = std::ceil(std::pow(1.0f / std::ldexp(1.0f, 1-boost::math::tools::digits<T>()), 1.0f / 20.0f));`。
- **L478 EN**: Blank line separating nearby declarations or logic.
  - **L478 CN**: 空行，用于分隔相邻声明或逻辑。
- **L479 EN**: Comment documents nearby intent or usage notes: `The new safe version of the limit check is now here.`.
  - **L479 CN**: 注释说明附近代码的意图或使用说明：`The new safe version of the limit check is now here.`。
- **L480 EN**: Initializes variable `d2_minus_one` from the right-hand expression.
  - **L480 CN**: 使用右侧表达式初始化变量 `d2_minus_one`。
- **L481 EN**: Initializes variable `limit` from the right-hand expression.
  - **L481 CN**: 使用右侧表达式初始化变量 `limit`。
- **L482 EN**: Blank line separating nearby declarations or logic.
  - **L482 CN**: 空行，用于分隔相邻声明或逻辑。
- **L483 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L483 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L484 EN**: Closes the current lexical scope or compound statement.
  - **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Blank line separating nearby declarations or logic.
  - **L485 CN**: 空行，用于分隔相邻声明或逻辑。
- **L486 EN**: Returns from the current function with `min_arg`.
  - **L486 CN**: 以 `min_arg` 从当前函数返回。
- **L487 EN**: Closes the current lexical scope or compound statement.
  - **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Blank line separating nearby declarations or logic.
  - **L488 CN**: 空行，用于分隔相邻声明或逻辑。
- **L489 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L489 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L490 EN**: Continues logic associated with callable symbol `scaled_tgamma_no_lanczos`.
  - **L490 CN**: 继续与可调用符号 `scaled_tgamma_no_lanczos` 相关的逻辑。
- **L491 EN**: Opens a new lexical scope or compound statement.
  - **L491 CN**: 打开一个新的词法作用域或复合语句块。
- **L492 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L492 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L493 EN**: Separator comment used for visual grouping.
  - **L493 CN**: 分隔注释，用于视觉分组。
- **L494 EN**: Comment documents nearby intent or usage notes: `Calculates tgamma(z) / (z/e)^z`.
  - **L494 CN**: 注释说明附近代码的意图或使用说明：`Calculates tgamma(z) / (z/e)^z`。
- **L495 EN**: Comment documents nearby intent or usage notes: `Requires that our argument is large enough for Sterling's approximation to hold.`.
  - **L495 CN**: 注释说明附近代码的意图或使用说明：`Requires that our argument is large enough for Sterling's approximation to hold.`。
- **L496 EN**: Comment documents nearby intent or usage notes: `Used internally when combining gamma's of similar magnitude without logarithms.`.
  - **L496 CN**: 注释说明附近代码的意图或使用说明：`Used internally when combining gamma's of similar magnitude without logarithms.`。
- **L497 EN**: Separator comment used for visual grouping.
  - **L497 CN**: 分隔注释，用于视觉分组。
- **L498 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L498 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L499 EN**: Blank line separating nearby declarations or logic.
  - **L499 CN**: 空行，用于分隔相邻声明或逻辑。
- **L500 EN**: Comment documents nearby intent or usage notes: `Perform the Bernoulli series expansion of Stirling's approximation.`.
  - **L500 CN**: 注释说明附近代码的意图或使用说明：`Perform the Bernoulli series expansion of Stirling's approximation.`。
- **L501 EN**: Blank line separating nearby declarations or logic.
  - **L501 CN**: 空行，用于分隔相邻声明或逻辑。
- **L502 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L502 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L503 EN**: Blank line separating nearby declarations or logic.
  - **L503 CN**: 空行，用于分隔相邻声明或逻辑。
- **L504 EN**: Executes a standalone statement or declaration: `T one_over_x_pow_two_n_minus_one = 1 / z;`.
  - **L504 CN**: 执行一条独立语句或声明：`T one_over_x_pow_two_n_minus_one = 1 / z;`。

### Lines 505-532 / 第 505-532 行

````cpp
 505:    const T one_over_x2 = one_over_x_pow_two_n_minus_one * one_over_x_pow_two_n_minus_one;
 506:    T sum = (boost::math::bernoulli_b2n<T>(1) / 2) * one_over_x_pow_two_n_minus_one;
 507:    const T target_epsilon_to_break_loop = sum * boost::math::tools::epsilon<T>();
 508:    const T half_ln_two_pi_over_z = sqrt(boost::math::constants::two_pi<T>() / z);
 509:    T last_term = 2 * sum;
 510: 
 511:    for (boost::math::size_t n = 2U;; ++n)
 512:    {
 513:       one_over_x_pow_two_n_minus_one *= one_over_x2;
 514: 
 515:       const boost::math::size_t n2 = static_cast<boost::math::size_t>(n * 2U);
 516: 
 517:       const T term = (boost::math::bernoulli_b2n<T>(static_cast<int>(n)) * one_over_x_pow_two_n_minus_one) / (n2 * (n2 - 1U));
 518: 
 519:       if ((n >= 3U) && (abs(term) < target_epsilon_to_break_loop))
 520:       {
 521:          // We have reached the desired precision in Stirling's expansion.
 522:          // Adding additional terms to the sum of this divergent asymptotic
 523:          // expansion will not improve the result.
 524: 
 525:          // Break from the loop.
 526:          break;
 527:       }
 528:       if (n > number_of_bernoullis_b2n)
 529:          // Safety net, we hope to never get here:
 530:          return policies::raise_evaluation_error("scaled_tgamma_no_lanczos<%1%>()", "Exceeded maximum series iterations without reaching convergence, best approximation was %1%", T(exp(sum) * half_ln_two_pi_over_z), pol); // LCOV_EXCL_LINE
 531: 
 532:       sum += term;
````
- **L505 EN**: Initializes variable `one_over_x2` from the right-hand expression.
  - **L505 CN**: 使用右侧表达式初始化变量 `one_over_x2`。
- **L506 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L506 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L507 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L507 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L508 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L508 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L509 EN**: Executes a standalone statement or declaration: `T last_term = 2 * sum;`.
  - **L509 CN**: 执行一条独立语句或声明：`T last_term = 2 * sum;`。
- **L510 EN**: Blank line separating nearby declarations or logic.
  - **L510 CN**: 空行，用于分隔相邻声明或逻辑。
- **L511 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L511 CN**: 开始 `for` 控制流语句并计算其条件。
- **L512 EN**: Opens a new lexical scope or compound statement.
  - **L512 CN**: 打开一个新的词法作用域或复合语句块。
- **L513 EN**: Executes a standalone statement or declaration: `one_over_x_pow_two_n_minus_one *= one_over_x2;`.
  - **L513 CN**: 执行一条独立语句或声明：`one_over_x_pow_two_n_minus_one *= one_over_x2;`。
- **L514 EN**: Blank line separating nearby declarations or logic.
  - **L514 CN**: 空行，用于分隔相邻声明或逻辑。
- **L515 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L515 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L516 EN**: Blank line separating nearby declarations or logic.
  - **L516 CN**: 空行，用于分隔相邻声明或逻辑。
- **L517 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L517 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L518 EN**: Blank line separating nearby declarations or logic.
  - **L518 CN**: 空行，用于分隔相邻声明或逻辑。
- **L519 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L519 CN**: 开始 `if` 控制流语句并计算其条件。
- **L520 EN**: Opens a new lexical scope or compound statement.
  - **L520 CN**: 打开一个新的词法作用域或复合语句块。
- **L521 EN**: Comment documents nearby intent or usage notes: `We have reached the desired precision in Stirling's expansion.`.
  - **L521 CN**: 注释说明附近代码的意图或使用说明：`We have reached the desired precision in Stirling's expansion.`。
- **L522 EN**: Comment documents nearby intent or usage notes: `Adding additional terms to the sum of this divergent asymptotic`.
  - **L522 CN**: 注释说明附近代码的意图或使用说明：`Adding additional terms to the sum of this divergent asymptotic`。
- **L523 EN**: Comment documents nearby intent or usage notes: `expansion will not improve the result.`.
  - **L523 CN**: 注释说明附近代码的意图或使用说明：`expansion will not improve the result.`。
- **L524 EN**: Blank line separating nearby declarations or logic.
  - **L524 CN**: 空行，用于分隔相邻声明或逻辑。
- **L525 EN**: Comment documents nearby intent or usage notes: `Break from the loop.`.
  - **L525 CN**: 注释说明附近代码的意图或使用说明：`Break from the loop.`。
- **L526 EN**: Exits the nearest loop or switch statement.
  - **L526 CN**: 退出最近的循环或 switch 语句。
- **L527 EN**: Closes the current lexical scope or compound statement.
  - **L527 CN**: 结束当前词法作用域或复合语句块。
- **L528 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L528 CN**: 开始 `if` 控制流语句并计算其条件。
- **L529 EN**: Comment documents nearby intent or usage notes: `Safety net, we hope to never get here:`.
  - **L529 CN**: 注释说明附近代码的意图或使用说明：`Safety net, we hope to never get here:`。
- **L530 EN**: Returns from the current function with `policies::raise_evaluation_error("scaled_tgamma_no_lanczos<%1%>()", "Exceeded maximum series iterations without reaching convergence, best approximation was %1%", T(exp(sum) * half_ln_two_pi_over_z), pol); // LCOV_EXCL_LINE`.
  - **L530 CN**: 以 `policies::raise_evaluation_error("scaled_tgamma_no_lanczos<%1%>()", "Exceeded maximum series iterations without reaching convergence, best approximation was %1%", T(exp(sum) * half_ln_two_pi_over_z), pol); // LCOV_EXCL_LINE` 从当前函数返回。
- **L531 EN**: Blank line separating nearby declarations or logic.
  - **L531 CN**: 空行，用于分隔相邻声明或逻辑。
- **L532 EN**: Executes a standalone statement or declaration: `sum += term;`.
  - **L532 CN**: 执行一条独立语句或声明：`sum += term;`。

### Lines 533-560 / 第 533-560 行

````cpp
 533: 
 534:       // Sanity check for divergence:
 535:       T fterm = fabs(term);
 536:       if(fterm > last_term)
 537:          // Safety net, we hope to never get here:
 538:          return policies::raise_evaluation_error("scaled_tgamma_no_lanczos<%1%>()", "Series became divergent without reaching convergence, best approximation was %1%", T(exp(sum) * half_ln_two_pi_over_z), pol);  // LCOV_EXCL_LINE
 539:       last_term = fterm;
 540:    }
 541: 
 542:    // Complete Stirling's approximation.
 543:    T scaled_gamma_value = islog ? T(sum + log(half_ln_two_pi_over_z)) : T(exp(sum) * half_ln_two_pi_over_z);
 544:    return scaled_gamma_value;
 545: }
 546: 
 547: // Forward declaration of the lgamma_imp template specialization.
 548: template <class T, class Policy>
 549: T lgamma_imp(T z, const Policy& pol, const lanczos::undefined_lanczos&, int* sign = nullptr);
 550: 
 551: template <class T, class Policy>
 552: T gamma_imp(T z, const Policy& pol, const lanczos::undefined_lanczos&)
 553: {
 554:    BOOST_MATH_STD_USING
 555: 
 556:    constexpr auto function = "boost::math::tgamma<%1%>(%1%)";
 557: 
 558:    // Check if the argument of tgamma is identically zero.
 559:    const bool is_at_zero = (z == 0);
 560: 
````
- **L533 EN**: Blank line separating nearby declarations or logic.
  - **L533 CN**: 空行，用于分隔相邻声明或逻辑。
- **L534 EN**: Comment documents nearby intent or usage notes: `Sanity check for divergence:`.
  - **L534 CN**: 注释说明附近代码的意图或使用说明：`Sanity check for divergence:`。
- **L535 EN**: Executes a call or declaration centered on `fabs`.
  - **L535 CN**: 执行以 `fabs` 为核心的调用或声明。
- **L536 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L536 CN**: 开始 `if` 控制流语句并计算其条件。
- **L537 EN**: Comment documents nearby intent or usage notes: `Safety net, we hope to never get here:`.
  - **L537 CN**: 注释说明附近代码的意图或使用说明：`Safety net, we hope to never get here:`。
- **L538 EN**: Returns from the current function with `policies::raise_evaluation_error("scaled_tgamma_no_lanczos<%1%>()", "Series became divergent without reaching convergence, best approximation was %1%", T(exp(sum) * half_ln_two_pi_over_z), pol);  // LCOV_EXCL_LINE`.
  - **L538 CN**: 以 `policies::raise_evaluation_error("scaled_tgamma_no_lanczos<%1%>()", "Series became divergent without reaching convergence, best approximation was %1%", T(exp(sum) * half_ln_two_pi_over_z), pol);  // LCOV_EXCL_LINE` 从当前函数返回。
- **L539 EN**: Executes a standalone statement or declaration: `last_term = fterm;`.
  - **L539 CN**: 执行一条独立语句或声明：`last_term = fterm;`。
- **L540 EN**: Closes the current lexical scope or compound statement.
  - **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Blank line separating nearby declarations or logic.
  - **L541 CN**: 空行，用于分隔相邻声明或逻辑。
- **L542 EN**: Comment documents nearby intent or usage notes: `Complete Stirling's approximation.`.
  - **L542 CN**: 注释说明附近代码的意图或使用说明：`Complete Stirling's approximation.`。
- **L543 EN**: Executes a call or declaration centered on `T`.
  - **L543 CN**: 执行以 `T` 为核心的调用或声明。
- **L544 EN**: Returns from the current function with `scaled_gamma_value`.
  - **L544 CN**: 以 `scaled_gamma_value` 从当前函数返回。
- **L545 EN**: Closes the current lexical scope or compound statement.
  - **L545 CN**: 结束当前词法作用域或复合语句块。
- **L546 EN**: Blank line separating nearby declarations or logic.
  - **L546 CN**: 空行，用于分隔相邻声明或逻辑。
- **L547 EN**: Comment documents nearby intent or usage notes: `Forward declaration of the lgamma_imp template specialization.`.
  - **L547 CN**: 注释说明附近代码的意图或使用说明：`Forward declaration of the lgamma_imp template specialization.`。
- **L548 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L548 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L549 EN**: Executes a call or declaration centered on `lgamma_imp`.
  - **L549 CN**: 执行以 `lgamma_imp` 为核心的调用或声明。
- **L550 EN**: Blank line separating nearby declarations or logic.
  - **L550 CN**: 空行，用于分隔相邻声明或逻辑。
- **L551 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L551 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L552 EN**: Continues logic associated with callable symbol `gamma_imp`.
  - **L552 CN**: 继续与可调用符号 `gamma_imp` 相关的逻辑。
- **L553 EN**: Opens a new lexical scope or compound statement.
  - **L553 CN**: 打开一个新的词法作用域或复合语句块。
- **L554 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L554 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L555 EN**: Blank line separating nearby declarations or logic.
  - **L555 CN**: 空行，用于分隔相邻声明或逻辑。
- **L556 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L556 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L557 EN**: Blank line separating nearby declarations or logic.
  - **L557 CN**: 空行，用于分隔相邻声明或逻辑。
- **L558 EN**: Comment documents nearby intent or usage notes: `Check if the argument of tgamma is identically zero.`.
  - **L558 CN**: 注释说明附近代码的意图或使用说明：`Check if the argument of tgamma is identically zero.`。
- **L559 EN**: Initializes variable `is_at_zero` from the right-hand expression.
  - **L559 CN**: 使用右侧表达式初始化变量 `is_at_zero`。
- **L560 EN**: Blank line separating nearby declarations or logic.
  - **L560 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 561-588 / 第 561-588 行

````cpp
 561:    if((boost::math::isnan)(z) || (is_at_zero) || ((boost::math::isinf)(z) && (z < 0)))
 562:       return policies::raise_domain_error<T>(function, "Evaluation of tgamma at %1%.", z, pol);
 563: 
 564:    const bool b_neg = (z < 0);
 565: 
 566:    const bool floor_of_z_is_equal_to_z = (floor(z) == z);
 567: 
 568:    // Special case handling of small factorials:
 569:    if((!b_neg) && floor_of_z_is_equal_to_z && (z < boost::math::max_factorial<T>::value))
 570:    {
 571:       return boost::math::unchecked_factorial<T>(static_cast<unsigned>(itrunc(z) - 1));
 572:    }
 573: 
 574:    // Make a local, unsigned copy of the input argument.
 575:    T zz((!b_neg) ? z : -z);
 576: 
 577:    // Special case for ultra-small z:
 578:    if(zz < tools::cbrt_epsilon<T>())
 579:    {
 580:       const T a0(1);
 581:       const T a1(boost::math::constants::euler<T>());
 582:       const T six_euler_squared((boost::math::constants::euler<T>() * boost::math::constants::euler<T>()) * 6);
 583:       const T a2((six_euler_squared -  boost::math::constants::pi_sqr<T>()) / 12);
 584: 
 585:       const T inverse_tgamma_series = z * ((a2 * z + a1) * z + a0);
 586: 
 587:       return 1 / inverse_tgamma_series;
 588:    }
````
- **L561 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L561 CN**: 开始 `if` 控制流语句并计算其条件。
- **L562 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Evaluation of tgamma at %1%.", z, pol)`.
  - **L562 CN**: 以 `policies::raise_domain_error<T>(function, "Evaluation of tgamma at %1%.", z, pol)` 从当前函数返回。
- **L563 EN**: Blank line separating nearby declarations or logic.
  - **L563 CN**: 空行，用于分隔相邻声明或逻辑。
- **L564 EN**: Initializes variable `b_neg` from the right-hand expression.
  - **L564 CN**: 使用右侧表达式初始化变量 `b_neg`。
- **L565 EN**: Blank line separating nearby declarations or logic.
  - **L565 CN**: 空行，用于分隔相邻声明或逻辑。
- **L566 EN**: Initializes variable `floor_of_z_is_equal_to_z` from the right-hand expression.
  - **L566 CN**: 使用右侧表达式初始化变量 `floor_of_z_is_equal_to_z`。
- **L567 EN**: Blank line separating nearby declarations or logic.
  - **L567 CN**: 空行，用于分隔相邻声明或逻辑。
- **L568 EN**: Comment documents nearby intent or usage notes: `Special case handling of small factorials:`.
  - **L568 CN**: 注释说明附近代码的意图或使用说明：`Special case handling of small factorials:`。
- **L569 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L569 CN**: 开始 `if` 控制流语句并计算其条件。
- **L570 EN**: Opens a new lexical scope or compound statement.
  - **L570 CN**: 打开一个新的词法作用域或复合语句块。
- **L571 EN**: Returns from the current function with `boost::math::unchecked_factorial<T>(static_cast<unsigned>(itrunc(z) - 1))`.
  - **L571 CN**: 以 `boost::math::unchecked_factorial<T>(static_cast<unsigned>(itrunc(z) - 1))` 从当前函数返回。
- **L572 EN**: Closes the current lexical scope or compound statement.
  - **L572 CN**: 结束当前词法作用域或复合语句块。
- **L573 EN**: Blank line separating nearby declarations or logic.
  - **L573 CN**: 空行，用于分隔相邻声明或逻辑。
- **L574 EN**: Comment documents nearby intent or usage notes: `Make a local, unsigned copy of the input argument.`.
  - **L574 CN**: 注释说明附近代码的意图或使用说明：`Make a local, unsigned copy of the input argument.`。
- **L575 EN**: Executes a call or declaration centered on `zz`.
  - **L575 CN**: 执行以 `zz` 为核心的调用或声明。
- **L576 EN**: Blank line separating nearby declarations or logic.
  - **L576 CN**: 空行，用于分隔相邻声明或逻辑。
- **L577 EN**: Comment documents nearby intent or usage notes: `Special case for ultra-small z:`.
  - **L577 CN**: 注释说明附近代码的意图或使用说明：`Special case for ultra-small z:`。
- **L578 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L578 CN**: 开始 `if` 控制流语句并计算其条件。
- **L579 EN**: Opens a new lexical scope or compound statement.
  - **L579 CN**: 打开一个新的词法作用域或复合语句块。
- **L580 EN**: Executes a call or declaration centered on `a0`.
  - **L580 CN**: 执行以 `a0` 为核心的调用或声明。
- **L581 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L581 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L582 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L582 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L583 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L583 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L584 EN**: Blank line separating nearby declarations or logic.
  - **L584 CN**: 空行，用于分隔相邻声明或逻辑。
- **L585 EN**: Initializes variable `inverse_tgamma_series` from the right-hand expression.
  - **L585 CN**: 使用右侧表达式初始化变量 `inverse_tgamma_series`。
- **L586 EN**: Blank line separating nearby declarations or logic.
  - **L586 CN**: 空行，用于分隔相邻声明或逻辑。
- **L587 EN**: Returns from the current function with `1 / inverse_tgamma_series`.
  - **L587 CN**: 以 `1 / inverse_tgamma_series` 从当前函数返回。
- **L588 EN**: Closes the current lexical scope or compound statement.
  - **L588 CN**: 结束当前词法作用域或复合语句块。

### Lines 589-616 / 第 589-616 行

````cpp
 589: 
 590:    // Scale the argument up for the calculation of lgamma,
 591:    // and use downward recursion later for the final result.
 592:    const int min_arg_for_recursion = minimum_argument_for_bernoulli_recursion<T>();
 593: 
 594:    int n_recur;
 595: 
 596:    if(zz < min_arg_for_recursion)
 597:    {
 598:       n_recur = boost::math::itrunc(min_arg_for_recursion - zz) + 1;
 599: 
 600:       zz += n_recur;
 601:    }
 602:    else
 603:    {
 604:       n_recur = 0;
 605:    }
 606:    if (!n_recur)
 607:    {
 608:       if (zz > tools::log_max_value<T>())
 609:          return b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only
 610:       if (log(zz) * zz / 2 > tools::log_max_value<T>())
 611:          return b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only
 612:    }
 613:    T gamma_value = scaled_tgamma_no_lanczos(zz, pol);
 614:    T power_term = pow(zz, zz / 2);
 615:    T exp_term = exp(-zz);
 616:    gamma_value *= (power_term * exp_term);
````
- **L589 EN**: Blank line separating nearby declarations or logic.
  - **L589 CN**: 空行，用于分隔相邻声明或逻辑。
- **L590 EN**: Comment documents nearby intent or usage notes: `Scale the argument up for the calculation of lgamma,`.
  - **L590 CN**: 注释说明附近代码的意图或使用说明：`Scale the argument up for the calculation of lgamma,`。
- **L591 EN**: Comment documents nearby intent or usage notes: `and use downward recursion later for the final result.`.
  - **L591 CN**: 注释说明附近代码的意图或使用说明：`and use downward recursion later for the final result.`。
- **L592 EN**: Initializes variable `min_arg_for_recursion` from the right-hand expression.
  - **L592 CN**: 使用右侧表达式初始化变量 `min_arg_for_recursion`。
- **L593 EN**: Blank line separating nearby declarations or logic.
  - **L593 CN**: 空行，用于分隔相邻声明或逻辑。
- **L594 EN**: Executes a standalone statement or declaration: `int n_recur;`.
  - **L594 CN**: 执行一条独立语句或声明：`int n_recur;`。
- **L595 EN**: Blank line separating nearby declarations or logic.
  - **L595 CN**: 空行，用于分隔相邻声明或逻辑。
- **L596 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L596 CN**: 开始 `if` 控制流语句并计算其条件。
- **L597 EN**: Opens a new lexical scope or compound statement.
  - **L597 CN**: 打开一个新的词法作用域或复合语句块。
- **L598 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L598 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L599 EN**: Blank line separating nearby declarations or logic.
  - **L599 CN**: 空行，用于分隔相邻声明或逻辑。
- **L600 EN**: Executes a standalone statement or declaration: `zz += n_recur;`.
  - **L600 CN**: 执行一条独立语句或声明：`zz += n_recur;`。
- **L601 EN**: Closes the current lexical scope or compound statement.
  - **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Starts the alternative branch of the preceding conditional.
  - **L602 CN**: 开始前一个条件语句的备选分支。
- **L603 EN**: Opens a new lexical scope or compound statement.
  - **L603 CN**: 打开一个新的词法作用域或复合语句块。
- **L604 EN**: Executes a standalone statement or declaration: `n_recur = 0;`.
  - **L604 CN**: 执行一条独立语句或声明：`n_recur = 0;`。
- **L605 EN**: Closes the current lexical scope or compound statement.
  - **L605 CN**: 结束当前词法作用域或复合语句块。
- **L606 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L606 CN**: 开始 `if` 控制流语句并计算其条件。
- **L607 EN**: Opens a new lexical scope or compound statement.
  - **L607 CN**: 打开一个新的词法作用域或复合语句块。
- **L608 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L608 CN**: 开始 `if` 控制流语句并计算其条件。
- **L609 EN**: Returns from the current function with `b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only`.
  - **L609 CN**: 以 `b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only` 从当前函数返回。
- **L610 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L610 CN**: 开始 `if` 控制流语句并计算其条件。
- **L611 EN**: Returns from the current function with `b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only`.
  - **L611 CN**: 以 `b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only` 从当前函数返回。
- **L612 EN**: Closes the current lexical scope or compound statement.
  - **L612 CN**: 结束当前词法作用域或复合语句块。
- **L613 EN**: Executes a call or declaration centered on `scaled_tgamma_no_lanczos`.
  - **L613 CN**: 执行以 `scaled_tgamma_no_lanczos` 为核心的调用或声明。
- **L614 EN**: Executes a call or declaration centered on `pow`.
  - **L614 CN**: 执行以 `pow` 为核心的调用或声明。
- **L615 EN**: Executes a call or declaration centered on `exp`.
  - **L615 CN**: 执行以 `exp` 为核心的调用或声明。
- **L616 EN**: Executes a call or declaration centered on `*=`.
  - **L616 CN**: 执行以 `*=` 为核心的调用或声明。

### Lines 617-644 / 第 617-644 行

````cpp
 617:    if (!n_recur && (tools::max_value<T>() / power_term < gamma_value))
 618:       return b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only
 619:    gamma_value *= power_term;
 620: 
 621:    // Rescale the result using downward recursion if necessary.
 622:    if(n_recur)
 623:    {
 624:       // The order of divides is important, if we keep subtracting 1 from zz
 625:       // we DO NOT get back to z (cancellation error).  Further if z < epsilon
 626:       // we would end up dividing by zero.  Also in order to prevent spurious
 627:       // overflow with the first division, we must save dividing by |z| till last,
 628:       // so the optimal order of divides is z+1, z+2, z+3...z+n_recur-1,z.
 629:       zz = fabs(z) + 1;
 630:       for(int k = 1; k < n_recur; ++k)
 631:       {
 632:          gamma_value /= zz;
 633:          zz += 1;
 634:       }
 635:       gamma_value /= fabs(z);
 636:    }
 637: 
 638:    // Return the result, accounting for possible negative arguments.
 639:    if(b_neg)
 640:    {
 641:       // Provide special error analysis for:
 642:       // * arguments in the neighborhood of a negative integer
 643:       // * arguments exactly equal to a negative integer.
 644: 
````
- **L617 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L617 CN**: 开始 `if` 控制流语句并计算其条件。
- **L618 EN**: Returns from the current function with `b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only`.
  - **L618 CN**: 以 `b_neg ? policies::raise_underflow_error<T>(function, nullptr, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only` 从当前函数返回。
- **L619 EN**: Executes a standalone statement or declaration: `gamma_value *= power_term;`.
  - **L619 CN**: 执行一条独立语句或声明：`gamma_value *= power_term;`。
- **L620 EN**: Blank line separating nearby declarations or logic.
  - **L620 CN**: 空行，用于分隔相邻声明或逻辑。
- **L621 EN**: Comment documents nearby intent or usage notes: `Rescale the result using downward recursion if necessary.`.
  - **L621 CN**: 注释说明附近代码的意图或使用说明：`Rescale the result using downward recursion if necessary.`。
- **L622 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L622 CN**: 开始 `if` 控制流语句并计算其条件。
- **L623 EN**: Opens a new lexical scope or compound statement.
  - **L623 CN**: 打开一个新的词法作用域或复合语句块。
- **L624 EN**: Comment documents nearby intent or usage notes: `The order of divides is important, if we keep subtracting 1 from zz`.
  - **L624 CN**: 注释说明附近代码的意图或使用说明：`The order of divides is important, if we keep subtracting 1 from zz`。
- **L625 EN**: Comment documents nearby intent or usage notes: `we DO NOT get back to z (cancellation error).  Further if z < epsilon`.
  - **L625 CN**: 注释说明附近代码的意图或使用说明：`we DO NOT get back to z (cancellation error).  Further if z < epsilon`。
- **L626 EN**: Comment documents nearby intent or usage notes: `we would end up dividing by zero.  Also in order to prevent spurious`.
  - **L626 CN**: 注释说明附近代码的意图或使用说明：`we would end up dividing by zero.  Also in order to prevent spurious`。
- **L627 EN**: Comment documents nearby intent or usage notes: `overflow with the first division, we must save dividing by |z| till last,`.
  - **L627 CN**: 注释说明附近代码的意图或使用说明：`overflow with the first division, we must save dividing by |z| till last,`。
- **L628 EN**: Comment documents nearby intent or usage notes: `so the optimal order of divides is z+1, z+2, z+3...z+n_recur-1,z.`.
  - **L628 CN**: 注释说明附近代码的意图或使用说明：`so the optimal order of divides is z+1, z+2, z+3...z+n_recur-1,z.`。
- **L629 EN**: Executes a call or declaration centered on `fabs`.
  - **L629 CN**: 执行以 `fabs` 为核心的调用或声明。
- **L630 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L630 CN**: 开始 `for` 控制流语句并计算其条件。
- **L631 EN**: Opens a new lexical scope or compound statement.
  - **L631 CN**: 打开一个新的词法作用域或复合语句块。
- **L632 EN**: Executes a standalone statement or declaration: `gamma_value /= zz;`.
  - **L632 CN**: 执行一条独立语句或声明：`gamma_value /= zz;`。
- **L633 EN**: Executes a standalone statement or declaration: `zz += 1;`.
  - **L633 CN**: 执行一条独立语句或声明：`zz += 1;`。
- **L634 EN**: Closes the current lexical scope or compound statement.
  - **L634 CN**: 结束当前词法作用域或复合语句块。
- **L635 EN**: Executes a call or declaration centered on `fabs`.
  - **L635 CN**: 执行以 `fabs` 为核心的调用或声明。
- **L636 EN**: Closes the current lexical scope or compound statement.
  - **L636 CN**: 结束当前词法作用域或复合语句块。
- **L637 EN**: Blank line separating nearby declarations or logic.
  - **L637 CN**: 空行，用于分隔相邻声明或逻辑。
- **L638 EN**: Comment documents nearby intent or usage notes: `Return the result, accounting for possible negative arguments.`.
  - **L638 CN**: 注释说明附近代码的意图或使用说明：`Return the result, accounting for possible negative arguments.`。
- **L639 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L639 CN**: 开始 `if` 控制流语句并计算其条件。
- **L640 EN**: Opens a new lexical scope or compound statement.
  - **L640 CN**: 打开一个新的词法作用域或复合语句块。
- **L641 EN**: Comment documents nearby intent or usage notes: `Provide special error analysis for:`.
  - **L641 CN**: 注释说明附近代码的意图或使用说明：`Provide special error analysis for:`。
- **L642 EN**: Comment documents nearby intent or usage notes: `arguments in the neighborhood of a negative integer`.
  - **L642 CN**: 注释说明附近代码的意图或使用说明：`arguments in the neighborhood of a negative integer`。
- **L643 EN**: Comment documents nearby intent or usage notes: `arguments exactly equal to a negative integer.`.
  - **L643 CN**: 注释说明附近代码的意图或使用说明：`arguments exactly equal to a negative integer.`。
- **L644 EN**: Blank line separating nearby declarations or logic.
  - **L644 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 645-672 / 第 645-672 行

````cpp
 645:       // Check if the argument of tgamma is exactly equal to a negative integer.
 646:       if(floor_of_z_is_equal_to_z)
 647:          return policies::raise_pole_error<T>(function, "Evaluation of tgamma at a negative integer %1%.", z, pol); // LCOV_EXCL_LINE  MP only
 648: 
 649:       T s = sinpx(z);
 650:       if ((gamma_value > 1) && (tools::max_value<T>() / gamma_value < fabs(s)))
 651:          return policies::raise_underflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only
 652:       gamma_value *= s;  // LCOV_EXCL_LINE  MP only
 653: 
 654:       BOOST_MATH_INSTRUMENT_VARIABLE(gamma_value);
 655:       //
 656:       // Result can never overflow, since sinpx(z) can never be smaller than machine epsilon and gamma_value > 1.
 657:       //
 658:       BOOST_MATH_ASSERT(   (abs(gamma_value) > 1) || ((tools::max_value<T>() * abs(gamma_value)) > boost::math::constants::pi<T>()));  // LCOV_EXCL_LINE  MP only
 659: 
 660:       gamma_value = -boost::math::constants::pi<T>() / gamma_value;
 661: 
 662:       BOOST_MATH_INSTRUMENT_VARIABLE(gamma_value);  // LCOV_EXCL_LINE  MP only
 663:       //
 664:       // We can never underflow since the numerator > 1 above and denominator is not infinite:
 665:       //
 666:       BOOST_MATH_ASSERT(gamma_value != 0);  // LCOV_EXCL_LINE  MP only
 667:    }
 668: 
 669:    return gamma_value;
 670: }
 671: 
 672: template <class T, class Policy>
````
- **L645 EN**: Comment documents nearby intent or usage notes: `Check if the argument of tgamma is exactly equal to a negative integer.`.
  - **L645 CN**: 注释说明附近代码的意图或使用说明：`Check if the argument of tgamma is exactly equal to a negative integer.`。
- **L646 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L646 CN**: 开始 `if` 控制流语句并计算其条件。
- **L647 EN**: Returns from the current function with `policies::raise_pole_error<T>(function, "Evaluation of tgamma at a negative integer %1%.", z, pol); // LCOV_EXCL_LINE  MP only`.
  - **L647 CN**: 以 `policies::raise_pole_error<T>(function, "Evaluation of tgamma at a negative integer %1%.", z, pol); // LCOV_EXCL_LINE  MP only` 从当前函数返回。
- **L648 EN**: Blank line separating nearby declarations or logic.
  - **L648 CN**: 空行，用于分隔相邻声明或逻辑。
- **L649 EN**: Executes a call or declaration centered on `sinpx`.
  - **L649 CN**: 执行以 `sinpx` 为核心的调用或声明。
- **L650 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L650 CN**: 开始 `if` 控制流语句并计算其条件。
- **L651 EN**: Returns from the current function with `policies::raise_underflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only`.
  - **L651 CN**: 以 `policies::raise_underflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE  MP only` 从当前函数返回。
- **L652 EN**: Continues the surrounding expression or declaration: `gamma_value *= s;  // LCOV_EXCL_LINE  MP only`.
  - **L652 CN**: 继续构造周围的表达式或声明：`gamma_value *= s;  // LCOV_EXCL_LINE  MP only`。
- **L653 EN**: Blank line separating nearby declarations or logic.
  - **L653 CN**: 空行，用于分隔相邻声明或逻辑。
- **L654 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L654 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L655 EN**: Separator comment used for visual grouping.
  - **L655 CN**: 分隔注释，用于视觉分组。
- **L656 EN**: Comment documents nearby intent or usage notes: `Result can never overflow, since sinpx(z) can never be smaller than machine epsilon and gamma_value > 1.`.
  - **L656 CN**: 注释说明附近代码的意图或使用说明：`Result can never overflow, since sinpx(z) can never be smaller than machine epsilon and gamma_value > 1.`。
- **L657 EN**: Separator comment used for visual grouping.
  - **L657 CN**: 分隔注释，用于视觉分组。
- **L658 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L658 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L659 EN**: Blank line separating nearby declarations or logic.
  - **L659 CN**: 空行，用于分隔相邻声明或逻辑。
- **L660 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L660 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L661 EN**: Blank line separating nearby declarations or logic.
  - **L661 CN**: 空行，用于分隔相邻声明或逻辑。
- **L662 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L662 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L663 EN**: Separator comment used for visual grouping.
  - **L663 CN**: 分隔注释，用于视觉分组。
- **L664 EN**: Comment documents nearby intent or usage notes: `We can never underflow since the numerator > 1 above and denominator is not infinite:`.
  - **L664 CN**: 注释说明附近代码的意图或使用说明：`We can never underflow since the numerator > 1 above and denominator is not infinite:`。
- **L665 EN**: Separator comment used for visual grouping.
  - **L665 CN**: 分隔注释，用于视觉分组。
- **L666 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L666 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L667 EN**: Closes the current lexical scope or compound statement.
  - **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Blank line separating nearby declarations or logic.
  - **L668 CN**: 空行，用于分隔相邻声明或逻辑。
- **L669 EN**: Returns from the current function with `gamma_value`.
  - **L669 CN**: 以 `gamma_value` 从当前函数返回。
- **L670 EN**: Closes the current lexical scope or compound statement.
  - **L670 CN**: 结束当前词法作用域或复合语句块。
- **L671 EN**: Blank line separating nearby declarations or logic.
  - **L671 CN**: 空行，用于分隔相邻声明或逻辑。
- **L672 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L672 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。

### Lines 673-700 / 第 673-700 行

````cpp
 673: inline T log_gamma_near_1(const T& z, Policy const& pol)
 674: {
 675:    //
 676:    // This is for the multiprecision case where there is
 677:    // no lanczos support, use a taylor series at z = 1,
 678:    // see https://www.wolframalpha.com/input/?i=taylor+series+lgamma(x)+at+x+%3D+1
 679:    //
 680:    BOOST_MATH_STD_USING // ADL of std names
 681: 
 682:    // For some reason, several lines aren't triggered for coverage even though
 683:    // adjacent lines are... weird!
 684: 
 685:    BOOST_MATH_ASSERT(fabs(z) < 1); // LCOV_EXCL_LINE
 686: 
 687:    T result = -constants::euler<T>() * z;
 688: 
 689:    T power_term = z * z / 2;
 690:    int n = 2;     // LCOV_EXCL_LINE
 691:    T term = 0;    // LCOV_EXCL_LINE
 692: 
 693:    do
 694:    {
 695:       term = power_term * boost::math::polygamma(n - 1, T(1), pol);
 696:       result += term;  // LCOV_EXCL_LINE
 697:       ++n;
 698:       power_term *= z / n;
 699:    } while (fabs(result) * tools::epsilon<T>() < fabs(term));
 700: 
````
- **L673 EN**: Continues logic associated with callable symbol `log_gamma_near_1`.
  - **L673 CN**: 继续与可调用符号 `log_gamma_near_1` 相关的逻辑。
- **L674 EN**: Opens a new lexical scope or compound statement.
  - **L674 CN**: 打开一个新的词法作用域或复合语句块。
- **L675 EN**: Separator comment used for visual grouping.
  - **L675 CN**: 分隔注释，用于视觉分组。
- **L676 EN**: Comment documents nearby intent or usage notes: `This is for the multiprecision case where there is`.
  - **L676 CN**: 注释说明附近代码的意图或使用说明：`This is for the multiprecision case where there is`。
- **L677 EN**: Comment documents nearby intent or usage notes: `no lanczos support, use a taylor series at z = 1,`.
  - **L677 CN**: 注释说明附近代码的意图或使用说明：`no lanczos support, use a taylor series at z = 1,`。
- **L678 EN**: Comment documents nearby intent or usage notes: `see https://www.wolframalpha.com/input/?i=taylor+series+lgamma(x)+at+x+%3D+1`.
  - **L678 CN**: 注释说明附近代码的意图或使用说明：`see https://www.wolframalpha.com/input/?i=taylor+series+lgamma(x)+at+x+%3D+1`。
- **L679 EN**: Separator comment used for visual grouping.
  - **L679 CN**: 分隔注释，用于视觉分组。
- **L680 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L680 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L681 EN**: Blank line separating nearby declarations or logic.
  - **L681 CN**: 空行，用于分隔相邻声明或逻辑。
- **L682 EN**: Comment documents nearby intent or usage notes: `For some reason, several lines aren't triggered for coverage even though`.
  - **L682 CN**: 注释说明附近代码的意图或使用说明：`For some reason, several lines aren't triggered for coverage even though`。
- **L683 EN**: Comment documents nearby intent or usage notes: `adjacent lines are... weird!`.
  - **L683 CN**: 注释说明附近代码的意图或使用说明：`adjacent lines are... weird!`。
- **L684 EN**: Blank line separating nearby declarations or logic.
  - **L684 CN**: 空行，用于分隔相邻声明或逻辑。
- **L685 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L685 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L686 EN**: Blank line separating nearby declarations or logic.
  - **L686 CN**: 空行，用于分隔相邻声明或逻辑。
- **L687 EN**: Executes a call or declaration centered on `-constants::euler<T>`.
  - **L687 CN**: 执行以 `-constants::euler<T>` 为核心的调用或声明。
- **L688 EN**: Blank line separating nearby declarations or logic.
  - **L688 CN**: 空行，用于分隔相邻声明或逻辑。
- **L689 EN**: Executes a standalone statement or declaration: `T power_term = z * z / 2;`.
  - **L689 CN**: 执行一条独立语句或声明：`T power_term = z * z / 2;`。
- **L690 EN**: Continues the surrounding expression or declaration: `int n = 2;     // LCOV_EXCL_LINE`.
  - **L690 CN**: 继续构造周围的表达式或声明：`int n = 2;     // LCOV_EXCL_LINE`。
- **L691 EN**: Continues the surrounding expression or declaration: `T term = 0;    // LCOV_EXCL_LINE`.
  - **L691 CN**: 继续构造周围的表达式或声明：`T term = 0;    // LCOV_EXCL_LINE`。
- **L692 EN**: Blank line separating nearby declarations or logic.
  - **L692 CN**: 空行，用于分隔相邻声明或逻辑。
- **L693 EN**: Continues the surrounding expression or declaration: `do`.
  - **L693 CN**: 继续构造周围的表达式或声明：`do`。
- **L694 EN**: Opens a new lexical scope or compound statement.
  - **L694 CN**: 打开一个新的词法作用域或复合语句块。
- **L695 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L695 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L696 EN**: Continues the surrounding expression or declaration: `result += term;  // LCOV_EXCL_LINE`.
  - **L696 CN**: 继续构造周围的表达式或声明：`result += term;  // LCOV_EXCL_LINE`。
- **L697 EN**: Executes a standalone statement or declaration: `++n;`.
  - **L697 CN**: 执行一条独立语句或声明：`++n;`。
- **L698 EN**: Executes a standalone statement or declaration: `power_term *= z / n;`.
  - **L698 CN**: 执行一条独立语句或声明：`power_term *= z / n;`。
- **L699 EN**: Executes a call or declaration centered on `while`.
  - **L699 CN**: 执行以 `while` 为核心的调用或声明。
- **L700 EN**: Blank line separating nearby declarations or logic.
  - **L700 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 701-728 / 第 701-728 行

````cpp
 701:    return result;
 702: }
 703: 
 704: template <class T, class Policy>
 705: T lgamma_imp(T z, const Policy& pol, const lanczos::undefined_lanczos&, int* sign)
 706: {
 707:    BOOST_MATH_STD_USING
 708: 
 709:    constexpr auto function = "boost::math::lgamma<%1%>(%1%)";
 710: 
 711:    // Check if the argument of lgamma is identically zero.
 712:    const bool is_at_zero = (z == 0);
 713: 
 714:    if(is_at_zero)
 715:       return policies::raise_domain_error<T>(function, "Evaluation of lgamma at zero %1%.", z, pol);
 716:    if((boost::math::isnan)(z))
 717:       return policies::raise_domain_error<T>(function, "Evaluation of lgamma at %1%.", z, pol);
 718:    if((boost::math::isinf)(z))
 719:       return policies::raise_overflow_error<T>(function, nullptr, pol);
 720: 
 721:    const bool b_neg = (z < 0);
 722: 
 723:    const bool floor_of_z_is_equal_to_z = (floor(z) == z);
 724: 
 725:    // Special case handling of small factorials:
 726:    if((!b_neg) && floor_of_z_is_equal_to_z && (z < boost::math::max_factorial<T>::value))
 727:    {
 728:       if (sign)
````
- **L701 EN**: Returns from the current function with `result`.
  - **L701 CN**: 以 `result` 从当前函数返回。
- **L702 EN**: Closes the current lexical scope or compound statement.
  - **L702 CN**: 结束当前词法作用域或复合语句块。
- **L703 EN**: Blank line separating nearby declarations or logic.
  - **L703 CN**: 空行，用于分隔相邻声明或逻辑。
- **L704 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L704 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L705 EN**: Continues logic associated with callable symbol `lgamma_imp`.
  - **L705 CN**: 继续与可调用符号 `lgamma_imp` 相关的逻辑。
- **L706 EN**: Opens a new lexical scope or compound statement.
  - **L706 CN**: 打开一个新的词法作用域或复合语句块。
- **L707 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L707 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L708 EN**: Blank line separating nearby declarations or logic.
  - **L708 CN**: 空行，用于分隔相邻声明或逻辑。
- **L709 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L709 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L710 EN**: Blank line separating nearby declarations or logic.
  - **L710 CN**: 空行，用于分隔相邻声明或逻辑。
- **L711 EN**: Comment documents nearby intent or usage notes: `Check if the argument of lgamma is identically zero.`.
  - **L711 CN**: 注释说明附近代码的意图或使用说明：`Check if the argument of lgamma is identically zero.`。
- **L712 EN**: Initializes variable `is_at_zero` from the right-hand expression.
  - **L712 CN**: 使用右侧表达式初始化变量 `is_at_zero`。
- **L713 EN**: Blank line separating nearby declarations or logic.
  - **L713 CN**: 空行，用于分隔相邻声明或逻辑。
- **L714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L715 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Evaluation of lgamma at zero %1%.", z, pol)`.
  - **L715 CN**: 以 `policies::raise_domain_error<T>(function, "Evaluation of lgamma at zero %1%.", z, pol)` 从当前函数返回。
- **L716 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L716 CN**: 开始 `if` 控制流语句并计算其条件。
- **L717 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Evaluation of lgamma at %1%.", z, pol)`.
  - **L717 CN**: 以 `policies::raise_domain_error<T>(function, "Evaluation of lgamma at %1%.", z, pol)` 从当前函数返回。
- **L718 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L718 CN**: 开始 `if` 控制流语句并计算其条件。
- **L719 EN**: Returns from the current function with `policies::raise_overflow_error<T>(function, nullptr, pol)`.
  - **L719 CN**: 以 `policies::raise_overflow_error<T>(function, nullptr, pol)` 从当前函数返回。
- **L720 EN**: Blank line separating nearby declarations or logic.
  - **L720 CN**: 空行，用于分隔相邻声明或逻辑。
- **L721 EN**: Initializes variable `b_neg` from the right-hand expression.
  - **L721 CN**: 使用右侧表达式初始化变量 `b_neg`。
- **L722 EN**: Blank line separating nearby declarations or logic.
  - **L722 CN**: 空行，用于分隔相邻声明或逻辑。
- **L723 EN**: Initializes variable `floor_of_z_is_equal_to_z` from the right-hand expression.
  - **L723 CN**: 使用右侧表达式初始化变量 `floor_of_z_is_equal_to_z`。
- **L724 EN**: Blank line separating nearby declarations or logic.
  - **L724 CN**: 空行，用于分隔相邻声明或逻辑。
- **L725 EN**: Comment documents nearby intent or usage notes: `Special case handling of small factorials:`.
  - **L725 CN**: 注释说明附近代码的意图或使用说明：`Special case handling of small factorials:`。
- **L726 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L726 CN**: 开始 `if` 控制流语句并计算其条件。
- **L727 EN**: Opens a new lexical scope or compound statement.
  - **L727 CN**: 打开一个新的词法作用域或复合语句块。
- **L728 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L728 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 729-756 / 第 729-756 行

````cpp
 729:          *sign = 1;  // LCOV_EXCL_LINE
 730:       return log(boost::math::unchecked_factorial<T>(itrunc(z) - 1));
 731:    }
 732: 
 733:    // Make a local, unsigned copy of the input argument.
 734:    T zz((!b_neg) ? z : -z);
 735: 
 736:    const int min_arg_for_recursion = minimum_argument_for_bernoulli_recursion<T>();
 737: 
 738:    T log_gamma_value;
 739: 
 740:    if (zz < min_arg_for_recursion)
 741:    {
 742:       // Here we simply take the logarithm of tgamma(). This is somewhat
 743:       // inefficient, but simple. The rationale is that the argument here
 744:       // is relatively small and overflow is not expected to be likely.
 745:       if (sign)
 746:          * sign = 1;  // // LCOV_EXCL_LINE
 747:       if(fabs(z - 1) < 0.25)
 748:       {
 749:          log_gamma_value = log_gamma_near_1(T(zz - 1), pol);
 750:       }
 751:       else if(fabs(z - 2) < 0.25)
 752:       {
 753:          log_gamma_value = log_gamma_near_1(T(zz - 2), pol) + log(zz - 1);
 754:       }
 755:       else if (z > -tools::root_epsilon<T>())
 756:       {
````
- **L729 EN**: Comment documents nearby intent or usage notes: `sign = 1;  // LCOV_EXCL_LINE`.
  - **L729 CN**: 注释说明附近代码的意图或使用说明：`sign = 1;  // LCOV_EXCL_LINE`。
- **L730 EN**: Returns from the current function with `log(boost::math::unchecked_factorial<T>(itrunc(z) - 1))`.
  - **L730 CN**: 以 `log(boost::math::unchecked_factorial<T>(itrunc(z) - 1))` 从当前函数返回。
- **L731 EN**: Closes the current lexical scope or compound statement.
  - **L731 CN**: 结束当前词法作用域或复合语句块。
- **L732 EN**: Blank line separating nearby declarations or logic.
  - **L732 CN**: 空行，用于分隔相邻声明或逻辑。
- **L733 EN**: Comment documents nearby intent or usage notes: `Make a local, unsigned copy of the input argument.`.
  - **L733 CN**: 注释说明附近代码的意图或使用说明：`Make a local, unsigned copy of the input argument.`。
- **L734 EN**: Executes a call or declaration centered on `zz`.
  - **L734 CN**: 执行以 `zz` 为核心的调用或声明。
- **L735 EN**: Blank line separating nearby declarations or logic.
  - **L735 CN**: 空行，用于分隔相邻声明或逻辑。
- **L736 EN**: Initializes variable `min_arg_for_recursion` from the right-hand expression.
  - **L736 CN**: 使用右侧表达式初始化变量 `min_arg_for_recursion`。
- **L737 EN**: Blank line separating nearby declarations or logic.
  - **L737 CN**: 空行，用于分隔相邻声明或逻辑。
- **L738 EN**: Executes a standalone statement or declaration: `T log_gamma_value;`.
  - **L738 CN**: 执行一条独立语句或声明：`T log_gamma_value;`。
- **L739 EN**: Blank line separating nearby declarations or logic.
  - **L739 CN**: 空行，用于分隔相邻声明或逻辑。
- **L740 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L740 CN**: 开始 `if` 控制流语句并计算其条件。
- **L741 EN**: Opens a new lexical scope or compound statement.
  - **L741 CN**: 打开一个新的词法作用域或复合语句块。
- **L742 EN**: Comment documents nearby intent or usage notes: `Here we simply take the logarithm of tgamma(). This is somewhat`.
  - **L742 CN**: 注释说明附近代码的意图或使用说明：`Here we simply take the logarithm of tgamma(). This is somewhat`。
- **L743 EN**: Comment documents nearby intent or usage notes: `inefficient, but simple. The rationale is that the argument here`.
  - **L743 CN**: 注释说明附近代码的意图或使用说明：`inefficient, but simple. The rationale is that the argument here`。
- **L744 EN**: Comment documents nearby intent or usage notes: `is relatively small and overflow is not expected to be likely.`.
  - **L744 CN**: 注释说明附近代码的意图或使用说明：`is relatively small and overflow is not expected to be likely.`。
- **L745 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L745 CN**: 开始 `if` 控制流语句并计算其条件。
- **L746 EN**: Comment documents nearby intent or usage notes: `sign = 1;  // // LCOV_EXCL_LINE`.
  - **L746 CN**: 注释说明附近代码的意图或使用说明：`sign = 1;  // // LCOV_EXCL_LINE`。
- **L747 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L747 CN**: 开始 `if` 控制流语句并计算其条件。
- **L748 EN**: Opens a new lexical scope or compound statement.
  - **L748 CN**: 打开一个新的词法作用域或复合语句块。
- **L749 EN**: Executes a call or declaration centered on `log_gamma_near_1`.
  - **L749 CN**: 执行以 `log_gamma_near_1` 为核心的调用或声明。
- **L750 EN**: Closes the current lexical scope or compound statement.
  - **L750 CN**: 结束当前词法作用域或复合语句块。
- **L751 EN**: Starts the alternative branch of the preceding conditional.
  - **L751 CN**: 开始前一个条件语句的备选分支。
- **L752 EN**: Opens a new lexical scope or compound statement.
  - **L752 CN**: 打开一个新的词法作用域或复合语句块。
- **L753 EN**: Executes a call or declaration centered on `log_gamma_near_1`.
  - **L753 CN**: 执行以 `log_gamma_near_1` 为核心的调用或声明。
- **L754 EN**: Closes the current lexical scope or compound statement.
  - **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Starts the alternative branch of the preceding conditional.
  - **L755 CN**: 开始前一个条件语句的备选分支。
- **L756 EN**: Opens a new lexical scope or compound statement.
  - **L756 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 757-784 / 第 757-784 行

````cpp
 757:          // Reflection formula may fail if z is very close to zero, let the series
 758:          // expansion for tgamma close to zero do the work:
 759:          if (sign)
 760:             *sign = z < 0 ? -1 : 1;  // LCOV_EXCL_LINE
 761:          return log(abs(gamma_imp(z, pol, lanczos::undefined_lanczos())));
 762:       }
 763:       else
 764:       {
 765:          // No issue with spurious overflow in reflection formula,
 766:          // just fall through to regular code:
 767:          T g = gamma_imp(zz, pol, lanczos::undefined_lanczos());
 768:          if (sign)
 769:          {
 770:             *sign = g < 0 ? -1 : 1;  // LCOV_EXCL_LINE  MP only
 771:          }
 772:          log_gamma_value = log(abs(g));
 773:       }
 774:    }
 775:    else
 776:    {
 777:       // Perform the Bernoulli series expansion of Stirling's approximation.
 778:       T sum = scaled_tgamma_no_lanczos(zz, pol, true);
 779:       log_gamma_value = zz * (log(zz) - 1) + sum;
 780:    }
 781: 
 782:    int sign_of_result = 1;
 783: 
 784:    if(b_neg)
````
- **L757 EN**: Comment documents nearby intent or usage notes: `Reflection formula may fail if z is very close to zero, let the series`.
  - **L757 CN**: 注释说明附近代码的意图或使用说明：`Reflection formula may fail if z is very close to zero, let the series`。
- **L758 EN**: Comment documents nearby intent or usage notes: `expansion for tgamma close to zero do the work:`.
  - **L758 CN**: 注释说明附近代码的意图或使用说明：`expansion for tgamma close to zero do the work:`。
- **L759 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L759 CN**: 开始 `if` 控制流语句并计算其条件。
- **L760 EN**: Comment documents nearby intent or usage notes: `sign = z < 0 ? -1 : 1;  // LCOV_EXCL_LINE`.
  - **L760 CN**: 注释说明附近代码的意图或使用说明：`sign = z < 0 ? -1 : 1;  // LCOV_EXCL_LINE`。
- **L761 EN**: Returns from the current function with `log(abs(gamma_imp(z, pol, lanczos::undefined_lanczos())))`.
  - **L761 CN**: 以 `log(abs(gamma_imp(z, pol, lanczos::undefined_lanczos())))` 从当前函数返回。
- **L762 EN**: Closes the current lexical scope or compound statement.
  - **L762 CN**: 结束当前词法作用域或复合语句块。
- **L763 EN**: Starts the alternative branch of the preceding conditional.
  - **L763 CN**: 开始前一个条件语句的备选分支。
- **L764 EN**: Opens a new lexical scope or compound statement.
  - **L764 CN**: 打开一个新的词法作用域或复合语句块。
- **L765 EN**: Comment documents nearby intent or usage notes: `No issue with spurious overflow in reflection formula,`.
  - **L765 CN**: 注释说明附近代码的意图或使用说明：`No issue with spurious overflow in reflection formula,`。
- **L766 EN**: Comment documents nearby intent or usage notes: `just fall through to regular code:`.
  - **L766 CN**: 注释说明附近代码的意图或使用说明：`just fall through to regular code:`。
- **L767 EN**: Executes a call or declaration centered on `gamma_imp`.
  - **L767 CN**: 执行以 `gamma_imp` 为核心的调用或声明。
- **L768 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L768 CN**: 开始 `if` 控制流语句并计算其条件。
- **L769 EN**: Opens a new lexical scope or compound statement.
  - **L769 CN**: 打开一个新的词法作用域或复合语句块。
- **L770 EN**: Comment documents nearby intent or usage notes: `sign = g < 0 ? -1 : 1;  // LCOV_EXCL_LINE  MP only`.
  - **L770 CN**: 注释说明附近代码的意图或使用说明：`sign = g < 0 ? -1 : 1;  // LCOV_EXCL_LINE  MP only`。
- **L771 EN**: Closes the current lexical scope or compound statement.
  - **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Executes a call or declaration centered on `log`.
  - **L772 CN**: 执行以 `log` 为核心的调用或声明。
- **L773 EN**: Closes the current lexical scope or compound statement.
  - **L773 CN**: 结束当前词法作用域或复合语句块。
- **L774 EN**: Closes the current lexical scope or compound statement.
  - **L774 CN**: 结束当前词法作用域或复合语句块。
- **L775 EN**: Starts the alternative branch of the preceding conditional.
  - **L775 CN**: 开始前一个条件语句的备选分支。
- **L776 EN**: Opens a new lexical scope or compound statement.
  - **L776 CN**: 打开一个新的词法作用域或复合语句块。
- **L777 EN**: Comment documents nearby intent or usage notes: `Perform the Bernoulli series expansion of Stirling's approximation.`.
  - **L777 CN**: 注释说明附近代码的意图或使用说明：`Perform the Bernoulli series expansion of Stirling's approximation.`。
- **L778 EN**: Executes a call or declaration centered on `scaled_tgamma_no_lanczos`.
  - **L778 CN**: 执行以 `scaled_tgamma_no_lanczos` 为核心的调用或声明。
- **L779 EN**: Executes a call or declaration centered on `*`.
  - **L779 CN**: 执行以 `*` 为核心的调用或声明。
- **L780 EN**: Closes the current lexical scope or compound statement.
  - **L780 CN**: 结束当前词法作用域或复合语句块。
- **L781 EN**: Blank line separating nearby declarations or logic.
  - **L781 CN**: 空行，用于分隔相邻声明或逻辑。
- **L782 EN**: Initializes variable `sign_of_result` from the right-hand expression.
  - **L782 CN**: 使用右侧表达式初始化变量 `sign_of_result`。
- **L783 EN**: Blank line separating nearby declarations or logic.
  - **L783 CN**: 空行，用于分隔相邻声明或逻辑。
- **L784 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L784 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 785-812 / 第 785-812 行

````cpp
 785:    {
 786:       // Provide special error analysis if the argument is exactly
 787:       // equal to a negative integer.
 788: 
 789:       // Check if the argument of lgamma is exactly equal to a negative integer.
 790:       if(floor_of_z_is_equal_to_z)
 791:          return policies::raise_pole_error<T>(function, "Evaluation of lgamma at a negative integer %1%.", z, pol);  // LCOV_EXCL_LINE  MP only
 792: 
 793:       T t = sinpx(z);
 794: 
 795:       if(t < 0)
 796:       {
 797:          t = -t;
 798:       }
 799:       else
 800:       {
 801:          sign_of_result = -sign_of_result;  // LCOV_EXCL_LINE  MP only
 802:       }
 803: 
 804:       log_gamma_value = - log_gamma_value + log(boost::math::constants::pi<T>()) - log(t);
 805:    }
 806: 
 807:    if(sign != static_cast<int*>(nullptr)) { *sign = sign_of_result; }
 808: 
 809:    return log_gamma_value;
 810: }
 811: 
 812: #endif // BOOST_MATH_HAS_GPU_SUPPORT
````
- **L785 EN**: Opens a new lexical scope or compound statement.
  - **L785 CN**: 打开一个新的词法作用域或复合语句块。
- **L786 EN**: Comment documents nearby intent or usage notes: `Provide special error analysis if the argument is exactly`.
  - **L786 CN**: 注释说明附近代码的意图或使用说明：`Provide special error analysis if the argument is exactly`。
- **L787 EN**: Comment documents nearby intent or usage notes: `equal to a negative integer.`.
  - **L787 CN**: 注释说明附近代码的意图或使用说明：`equal to a negative integer.`。
- **L788 EN**: Blank line separating nearby declarations or logic.
  - **L788 CN**: 空行，用于分隔相邻声明或逻辑。
- **L789 EN**: Comment documents nearby intent or usage notes: `Check if the argument of lgamma is exactly equal to a negative integer.`.
  - **L789 CN**: 注释说明附近代码的意图或使用说明：`Check if the argument of lgamma is exactly equal to a negative integer.`。
- **L790 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L790 CN**: 开始 `if` 控制流语句并计算其条件。
- **L791 EN**: Returns from the current function with `policies::raise_pole_error<T>(function, "Evaluation of lgamma at a negative integer %1%.", z, pol);  // LCOV_EXCL_LINE  MP only`.
  - **L791 CN**: 以 `policies::raise_pole_error<T>(function, "Evaluation of lgamma at a negative integer %1%.", z, pol);  // LCOV_EXCL_LINE  MP only` 从当前函数返回。
- **L792 EN**: Blank line separating nearby declarations or logic.
  - **L792 CN**: 空行，用于分隔相邻声明或逻辑。
- **L793 EN**: Executes a call or declaration centered on `sinpx`.
  - **L793 CN**: 执行以 `sinpx` 为核心的调用或声明。
- **L794 EN**: Blank line separating nearby declarations or logic.
  - **L794 CN**: 空行，用于分隔相邻声明或逻辑。
- **L795 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L795 CN**: 开始 `if` 控制流语句并计算其条件。
- **L796 EN**: Opens a new lexical scope or compound statement.
  - **L796 CN**: 打开一个新的词法作用域或复合语句块。
- **L797 EN**: Executes a standalone statement or declaration: `t = -t;`.
  - **L797 CN**: 执行一条独立语句或声明：`t = -t;`。
- **L798 EN**: Closes the current lexical scope or compound statement.
  - **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Starts the alternative branch of the preceding conditional.
  - **L799 CN**: 开始前一个条件语句的备选分支。
- **L800 EN**: Opens a new lexical scope or compound statement.
  - **L800 CN**: 打开一个新的词法作用域或复合语句块。
- **L801 EN**: Continues the surrounding expression or declaration: `sign_of_result = -sign_of_result;  // LCOV_EXCL_LINE  MP only`.
  - **L801 CN**: 继续构造周围的表达式或声明：`sign_of_result = -sign_of_result;  // LCOV_EXCL_LINE  MP only`。
- **L802 EN**: Closes the current lexical scope or compound statement.
  - **L802 CN**: 结束当前词法作用域或复合语句块。
- **L803 EN**: Blank line separating nearby declarations or logic.
  - **L803 CN**: 空行，用于分隔相邻声明或逻辑。
- **L804 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L804 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L805 EN**: Closes the current lexical scope or compound statement.
  - **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Blank line separating nearby declarations or logic.
  - **L806 CN**: 空行，用于分隔相邻声明或逻辑。
- **L807 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L807 CN**: 开始 `if` 控制流语句并计算其条件。
- **L808 EN**: Blank line separating nearby declarations or logic.
  - **L808 CN**: 空行，用于分隔相邻声明或逻辑。
- **L809 EN**: Returns from the current function with `log_gamma_value`.
  - **L809 CN**: 以 `log_gamma_value` 从当前函数返回。
- **L810 EN**: Closes the current lexical scope or compound statement.
  - **L810 CN**: 结束当前词法作用域或复合语句块。
- **L811 EN**: Blank line separating nearby declarations or logic.
  - **L811 CN**: 空行，用于分隔相邻声明或逻辑。
- **L812 EN**: Closes the current preprocessor conditional block or header guard.
  - **L812 CN**: 结束当前预处理条件块或头文件保护。

### Lines 813-840 / 第 813-840 行

````cpp
 813: 
 814: // In order for tgammap1m1_imp to compile we need a forward decl of boost::math::tgamma
 815: // The rub is that we can't just use math_fwd so we provide one here only in that circumstance
 816: #ifdef BOOST_MATH_HAS_NVRTC
 817: template <class RT>
 818: BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> tgamma(RT z);
 819: 
 820: template <class RT1, class RT2>
 821: BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> tgamma(RT1 a, RT2 z);
 822: 
 823: template <class RT1, class RT2, class Policy>
 824: BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> tgamma(RT1 a, RT2 z, const Policy& pol);
 825: #endif
 826: 
 827: //
 828: // This helper calculates tgamma(dz+1)-1 without cancellation errors,
 829: // used by the upper incomplete gamma with z < 1:
 830: //
 831: template <class T, class Policy, class Lanczos>
 832: BOOST_MATH_GPU_ENABLED T tgammap1m1_imp(T dz, Policy const& pol, const Lanczos& l)
 833: {
 834:    BOOST_MATH_STD_USING
 835: 
 836:    typedef typename policies::precision<T,Policy>::type precision_type;
 837: 
 838:    typedef boost::math::integral_constant<int,
 839:       precision_type::value <= 0 ? 0 :
 840:       precision_type::value <= 64 ? 64 :
````
- **L813 EN**: Blank line separating nearby declarations or logic.
  - **L813 CN**: 空行，用于分隔相邻声明或逻辑。
- **L814 EN**: Comment documents nearby intent or usage notes: `In order for tgammap1m1_imp to compile we need a forward decl of boost::math::tgamma`.
  - **L814 CN**: 注释说明附近代码的意图或使用说明：`In order for tgammap1m1_imp to compile we need a forward decl of boost::math::tgamma`。
- **L815 EN**: Comment documents nearby intent or usage notes: `The rub is that we can't just use math_fwd so we provide one here only in that circumstance`.
  - **L815 CN**: 注释说明附近代码的意图或使用说明：`The rub is that we can't just use math_fwd so we provide one here only in that circumstance`。
- **L816 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L816 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L817 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L817 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L818 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L818 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L819 EN**: Blank line separating nearby declarations or logic.
  - **L819 CN**: 空行，用于分隔相邻声明或逻辑。
- **L820 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>`.
  - **L820 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>`。
- **L821 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L821 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L822 EN**: Blank line separating nearby declarations or logic.
  - **L822 CN**: 空行，用于分隔相邻声明或逻辑。
- **L823 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class Policy>`.
  - **L823 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class Policy>`。
- **L824 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L824 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L825 EN**: Closes the current preprocessor conditional block or header guard.
  - **L825 CN**: 结束当前预处理条件块或头文件保护。
- **L826 EN**: Blank line separating nearby declarations or logic.
  - **L826 CN**: 空行，用于分隔相邻声明或逻辑。
- **L827 EN**: Separator comment used for visual grouping.
  - **L827 CN**: 分隔注释，用于视觉分组。
- **L828 EN**: Comment documents nearby intent or usage notes: `This helper calculates tgamma(dz+1)-1 without cancellation errors,`.
  - **L828 CN**: 注释说明附近代码的意图或使用说明：`This helper calculates tgamma(dz+1)-1 without cancellation errors,`。
- **L829 EN**: Comment documents nearby intent or usage notes: `used by the upper incomplete gamma with z < 1:`.
  - **L829 CN**: 注释说明附近代码的意图或使用说明：`used by the upper incomplete gamma with z < 1:`。
- **L830 EN**: Separator comment used for visual grouping.
  - **L830 CN**: 分隔注释，用于视觉分组。
- **L831 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L831 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L832 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L832 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L833 EN**: Opens a new lexical scope or compound statement.
  - **L833 CN**: 打开一个新的词法作用域或复合语句块。
- **L834 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L834 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L835 EN**: Blank line separating nearby declarations or logic.
  - **L835 CN**: 空行，用于分隔相邻声明或逻辑。
- **L836 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::precision<T,Policy>::type precision_type;`.
  - **L836 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::precision<T,Policy>::type precision_type;`。
- **L837 EN**: Blank line separating nearby declarations or logic.
  - **L837 CN**: 空行，用于分隔相邻声明或逻辑。
- **L838 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int,`.
  - **L838 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int,`。
- **L839 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 0 ? 0 :`.
  - **L839 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 0 ? 0 :`。
- **L840 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 64 ? 64 :`.
  - **L840 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 64 ? 64 :`。

### Lines 841-868 / 第 841-868 行

````cpp
 841:       precision_type::value <= 113 ? 113 : 0
 842:    > tag_type;
 843: 
 844:    T result{};
 845:    if(dz < 0)
 846:    {
 847:       if(dz < T(-0.5))
 848:       {
 849:          // Best method is simply to subtract 1 from tgamma:
 850:          #ifdef BOOST_MATH_HAS_NVRTC
 851:          result = ::tgamma(1+dz);
 852:          #else
 853:          result = boost::math::tgamma(1+dz, pol) - 1;
 854:          #endif
 855:          BOOST_MATH_INSTRUMENT_CODE(result);
 856:       }
 857:       else
 858:       {
 859:          // Use expm1 on lgamma:
 860:          result = boost::math::expm1(-boost::math::log1p(dz, pol)
 861:             + lgamma_small_imp<T>(dz+2, dz + 1, dz, tag_type(), pol, l), pol);
 862:          BOOST_MATH_INSTRUMENT_CODE(result);
 863:       }
 864:    }
 865:    else
 866:    {
 867:       if(dz < 2)
 868:       {
````
- **L841 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 113 ? 113 : 0`.
  - **L841 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 113 ? 113 : 0`。
- **L842 EN**: Executes a standalone statement or declaration: `> tag_type;`.
  - **L842 CN**: 执行一条独立语句或声明：`> tag_type;`。
- **L843 EN**: Blank line separating nearby declarations or logic.
  - **L843 CN**: 空行，用于分隔相邻声明或逻辑。
- **L844 EN**: Executes a standalone statement or declaration: `T result{};`.
  - **L844 CN**: 执行一条独立语句或声明：`T result{};`。
- **L845 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L845 CN**: 开始 `if` 控制流语句并计算其条件。
- **L846 EN**: Opens a new lexical scope or compound statement.
  - **L846 CN**: 打开一个新的词法作用域或复合语句块。
- **L847 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L847 CN**: 开始 `if` 控制流语句并计算其条件。
- **L848 EN**: Opens a new lexical scope or compound statement.
  - **L848 CN**: 打开一个新的词法作用域或复合语句块。
- **L849 EN**: Comment documents nearby intent or usage notes: `Best method is simply to subtract 1 from tgamma:`.
  - **L849 CN**: 注释说明附近代码的意图或使用说明：`Best method is simply to subtract 1 from tgamma:`。
- **L850 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L850 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L851 EN**: Executes a call or declaration centered on `::tgamma`.
  - **L851 CN**: 执行以 `::tgamma` 为核心的调用或声明。
- **L852 EN**: Continues the current preprocessor branch selection.
  - **L852 CN**: 继续当前的预处理分支选择。
- **L853 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L853 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L854 EN**: Closes the current preprocessor conditional block or header guard.
  - **L854 CN**: 结束当前预处理条件块或头文件保护。
- **L855 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L855 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L856 EN**: Closes the current lexical scope or compound statement.
  - **L856 CN**: 结束当前词法作用域或复合语句块。
- **L857 EN**: Starts the alternative branch of the preceding conditional.
  - **L857 CN**: 开始前一个条件语句的备选分支。
- **L858 EN**: Opens a new lexical scope or compound statement.
  - **L858 CN**: 打开一个新的词法作用域或复合语句块。
- **L859 EN**: Comment documents nearby intent or usage notes: `Use expm1 on lgamma:`.
  - **L859 CN**: 注释说明附近代码的意图或使用说明：`Use expm1 on lgamma:`。
- **L860 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L860 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L861 EN**: Executes a call or declaration centered on `lgamma_small_imp<T>`.
  - **L861 CN**: 执行以 `lgamma_small_imp<T>` 为核心的调用或声明。
- **L862 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L862 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L863 EN**: Closes the current lexical scope or compound statement.
  - **L863 CN**: 结束当前词法作用域或复合语句块。
- **L864 EN**: Closes the current lexical scope or compound statement.
  - **L864 CN**: 结束当前词法作用域或复合语句块。
- **L865 EN**: Starts the alternative branch of the preceding conditional.
  - **L865 CN**: 开始前一个条件语句的备选分支。
- **L866 EN**: Opens a new lexical scope or compound statement.
  - **L866 CN**: 打开一个新的词法作用域或复合语句块。
- **L867 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L867 CN**: 开始 `if` 控制流语句并计算其条件。
- **L868 EN**: Opens a new lexical scope or compound statement.
  - **L868 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 869-896 / 第 869-896 行

````cpp
 869:          // Use expm1 on lgamma:
 870:          result = boost::math::expm1(lgamma_small_imp<T>(dz+1, dz, dz-1, tag_type(), pol, l), pol);
 871:          BOOST_MATH_INSTRUMENT_CODE(result);
 872:       }
 873:       else
 874:       {
 875:          // Best method is simply to subtract 1 from tgamma:
 876:          #ifdef BOOST_MATH_HAS_NVRTC
 877:          result = ::tgamma(1+dz);
 878:          #else
 879:          result = boost::math::tgamma(1+dz, pol) - 1;
 880:          #endif
 881:          BOOST_MATH_INSTRUMENT_CODE(result);
 882:       }
 883:    }
 884: 
 885:    return result;
 886: }
 887: 
 888: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 889: 
 890: template <class T, class Policy>
 891: inline T tgammap1m1_imp(T z, Policy const& pol,
 892:                  const ::boost::math::lanczos::undefined_lanczos&)
 893: {
 894:    BOOST_MATH_STD_USING // ADL of std names
 895: 
 896:    if(fabs(z) < T(0.55))
````
- **L869 EN**: Comment documents nearby intent or usage notes: `Use expm1 on lgamma:`.
  - **L869 CN**: 注释说明附近代码的意图或使用说明：`Use expm1 on lgamma:`。
- **L870 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L870 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L871 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L871 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L872 EN**: Closes the current lexical scope or compound statement.
  - **L872 CN**: 结束当前词法作用域或复合语句块。
- **L873 EN**: Starts the alternative branch of the preceding conditional.
  - **L873 CN**: 开始前一个条件语句的备选分支。
- **L874 EN**: Opens a new lexical scope or compound statement.
  - **L874 CN**: 打开一个新的词法作用域或复合语句块。
- **L875 EN**: Comment documents nearby intent or usage notes: `Best method is simply to subtract 1 from tgamma:`.
  - **L875 CN**: 注释说明附近代码的意图或使用说明：`Best method is simply to subtract 1 from tgamma:`。
- **L876 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L876 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L877 EN**: Executes a call or declaration centered on `::tgamma`.
  - **L877 CN**: 执行以 `::tgamma` 为核心的调用或声明。
- **L878 EN**: Continues the current preprocessor branch selection.
  - **L878 CN**: 继续当前的预处理分支选择。
- **L879 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L879 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L880 EN**: Closes the current preprocessor conditional block or header guard.
  - **L880 CN**: 结束当前预处理条件块或头文件保护。
- **L881 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L881 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L882 EN**: Closes the current lexical scope or compound statement.
  - **L882 CN**: 结束当前词法作用域或复合语句块。
- **L883 EN**: Closes the current lexical scope or compound statement.
  - **L883 CN**: 结束当前词法作用域或复合语句块。
- **L884 EN**: Blank line separating nearby declarations or logic.
  - **L884 CN**: 空行，用于分隔相邻声明或逻辑。
- **L885 EN**: Returns from the current function with `result`.
  - **L885 CN**: 以 `result` 从当前函数返回。
- **L886 EN**: Closes the current lexical scope or compound statement.
  - **L886 CN**: 结束当前词法作用域或复合语句块。
- **L887 EN**: Blank line separating nearby declarations or logic.
  - **L887 CN**: 空行，用于分隔相邻声明或逻辑。
- **L888 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_HAS_GPU_SUPPORT`.
  - **L888 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_HAS_GPU_SUPPORT`。
- **L889 EN**: Blank line separating nearby declarations or logic.
  - **L889 CN**: 空行，用于分隔相邻声明或逻辑。
- **L890 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L890 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `inline T tgammap1m1_imp(T z, Policy const& pol,`.
  - **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`inline T tgammap1m1_imp(T z, Policy const& pol,`。
- **L892 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L892 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L893 EN**: Opens a new lexical scope or compound statement.
  - **L893 CN**: 打开一个新的词法作用域或复合语句块。
- **L894 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L894 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L895 EN**: Blank line separating nearby declarations or logic.
  - **L895 CN**: 空行，用于分隔相邻声明或逻辑。
- **L896 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L896 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 897-924 / 第 897-924 行

````cpp
 897:    {
 898:       return boost::math::expm1(log_gamma_near_1(z, pol));
 899:    }
 900:    return boost::math::expm1(boost::math::lgamma(1 + z, pol));
 901: }
 902: 
 903: #endif // BOOST_MATH_HAS_GPU_SUPPORT
 904: 
 905: //
 906: // Series representation for upper fraction when z is small:
 907: //
 908: template <class T>
 909: struct small_gamma2_series
 910: {
 911:    typedef T result_type;
 912: 
 913:    BOOST_MATH_GPU_ENABLED small_gamma2_series(T a_, T x_) : result(-x_), x(-x_), apn(a_+1), n(1){}
 914: 
 915:    BOOST_MATH_GPU_ENABLED T operator()()
 916:    {
 917:       T r = result / (apn);
 918:       result *= x;
 919:       result /= ++n;
 920:       apn += 1;
 921:       return r;
 922:    }
 923: 
 924: private:
````
- **L897 EN**: Opens a new lexical scope or compound statement.
  - **L897 CN**: 打开一个新的词法作用域或复合语句块。
- **L898 EN**: Returns from the current function with `boost::math::expm1(log_gamma_near_1(z, pol))`.
  - **L898 CN**: 以 `boost::math::expm1(log_gamma_near_1(z, pol))` 从当前函数返回。
- **L899 EN**: Closes the current lexical scope or compound statement.
  - **L899 CN**: 结束当前词法作用域或复合语句块。
- **L900 EN**: Returns from the current function with `boost::math::expm1(boost::math::lgamma(1 + z, pol))`.
  - **L900 CN**: 以 `boost::math::expm1(boost::math::lgamma(1 + z, pol))` 从当前函数返回。
- **L901 EN**: Closes the current lexical scope or compound statement.
  - **L901 CN**: 结束当前词法作用域或复合语句块。
- **L902 EN**: Blank line separating nearby declarations or logic.
  - **L902 CN**: 空行，用于分隔相邻声明或逻辑。
- **L903 EN**: Closes the current preprocessor conditional block or header guard.
  - **L903 CN**: 结束当前预处理条件块或头文件保护。
- **L904 EN**: Blank line separating nearby declarations or logic.
  - **L904 CN**: 空行，用于分隔相邻声明或逻辑。
- **L905 EN**: Separator comment used for visual grouping.
  - **L905 CN**: 分隔注释，用于视觉分组。
- **L906 EN**: Comment documents nearby intent or usage notes: `Series representation for upper fraction when z is small:`.
  - **L906 CN**: 注释说明附近代码的意图或使用说明：`Series representation for upper fraction when z is small:`。
- **L907 EN**: Separator comment used for visual grouping.
  - **L907 CN**: 分隔注释，用于视觉分组。
- **L908 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L908 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L909 EN**: Declares struct `small_gamma2_series`.
  - **L909 CN**: 声明 struct `small_gamma2_series`。
- **L910 EN**: Opens a new lexical scope or compound statement.
  - **L910 CN**: 打开一个新的词法作用域或复合语句块。
- **L911 EN**: Introduces a legacy type alias or function typedef: `typedef T result_type;`.
  - **L911 CN**: 引入传统类型别名或函数 typedef：`typedef T result_type;`。
- **L912 EN**: Blank line separating nearby declarations or logic.
  - **L912 CN**: 空行，用于分隔相邻声明或逻辑。
- **L913 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L913 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L914 EN**: Blank line separating nearby declarations or logic.
  - **L914 CN**: 空行，用于分隔相邻声明或逻辑。
- **L915 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L915 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L916 EN**: Opens a new lexical scope or compound statement.
  - **L916 CN**: 打开一个新的词法作用域或复合语句块。
- **L917 EN**: Executes a call or declaration centered on `/`.
  - **L917 CN**: 执行以 `/` 为核心的调用或声明。
- **L918 EN**: Executes a standalone statement or declaration: `result *= x;`.
  - **L918 CN**: 执行一条独立语句或声明：`result *= x;`。
- **L919 EN**: Executes a standalone statement or declaration: `result /= ++n;`.
  - **L919 CN**: 执行一条独立语句或声明：`result /= ++n;`。
- **L920 EN**: Executes a standalone statement or declaration: `apn += 1;`.
  - **L920 CN**: 执行一条独立语句或声明：`apn += 1;`。
- **L921 EN**: Returns from the current function with `r`.
  - **L921 CN**: 以 `r` 从当前函数返回。
- **L922 EN**: Closes the current lexical scope or compound statement.
  - **L922 CN**: 结束当前词法作用域或复合语句块。
- **L923 EN**: Blank line separating nearby declarations or logic.
  - **L923 CN**: 空行，用于分隔相邻声明或逻辑。
- **L924 EN**: Sets the following members to `private` access.
  - **L924 CN**: 将后续成员的访问级别设为 `private`。

### Lines 925-952 / 第 925-952 行

````cpp
 925:    T result, x, apn;
 926:    int n;
 927: };
 928: //
 929: // calculate power term prefix (z^a)(e^-z) used in the non-normalised
 930: // incomplete gammas:
 931: //
 932: template <class T, class Policy>
 933: BOOST_MATH_GPU_ENABLED T full_igamma_prefix(T a, T z, const Policy& pol)
 934: {
 935:    BOOST_MATH_STD_USING
 936: 
 937:    if (z > tools::max_value<T>() || (a > 0 && z == 0))
 938:       return 0;
 939: 
 940:    T alz = a * log(z);
 941: 
 942:    T prefix { };
 943: 
 944:    if(z >= 1)
 945:    {
 946:       if((alz < tools::log_max_value<T>()) && (-z > tools::log_min_value<T>()))
 947:       {
 948:          prefix = pow(z, a) * exp(-z);
 949:       }
 950:       else if(a >= 1)
 951:       {
 952:          prefix = pow(T(z / exp(z/a)), a);
````
- **L925 EN**: Executes a standalone statement or declaration: `T result, x, apn;`.
  - **L925 CN**: 执行一条独立语句或声明：`T result, x, apn;`。
- **L926 EN**: Executes a standalone statement or declaration: `int n;`.
  - **L926 CN**: 执行一条独立语句或声明：`int n;`。
- **L927 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L927 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L928 EN**: Separator comment used for visual grouping.
  - **L928 CN**: 分隔注释，用于视觉分组。
- **L929 EN**: Comment documents nearby intent or usage notes: `calculate power term prefix (z^a)(e^-z) used in the non-normalised`.
  - **L929 CN**: 注释说明附近代码的意图或使用说明：`calculate power term prefix (z^a)(e^-z) used in the non-normalised`。
- **L930 EN**: Comment documents nearby intent or usage notes: `incomplete gammas:`.
  - **L930 CN**: 注释说明附近代码的意图或使用说明：`incomplete gammas:`。
- **L931 EN**: Separator comment used for visual grouping.
  - **L931 CN**: 分隔注释，用于视觉分组。
- **L932 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L932 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L933 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L933 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L934 EN**: Opens a new lexical scope or compound statement.
  - **L934 CN**: 打开一个新的词法作用域或复合语句块。
- **L935 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L935 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L936 EN**: Blank line separating nearby declarations or logic.
  - **L936 CN**: 空行，用于分隔相邻声明或逻辑。
- **L937 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L937 CN**: 开始 `if` 控制流语句并计算其条件。
- **L938 EN**: Returns from the current function with `0`.
  - **L938 CN**: 以 `0` 从当前函数返回。
- **L939 EN**: Blank line separating nearby declarations or logic.
  - **L939 CN**: 空行，用于分隔相邻声明或逻辑。
- **L940 EN**: Executes a call or declaration centered on `log`.
  - **L940 CN**: 执行以 `log` 为核心的调用或声明。
- **L941 EN**: Blank line separating nearby declarations or logic.
  - **L941 CN**: 空行，用于分隔相邻声明或逻辑。
- **L942 EN**: Executes a standalone statement or declaration: `T prefix { };`.
  - **L942 CN**: 执行一条独立语句或声明：`T prefix { };`。
- **L943 EN**: Blank line separating nearby declarations or logic.
  - **L943 CN**: 空行，用于分隔相邻声明或逻辑。
- **L944 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L944 CN**: 开始 `if` 控制流语句并计算其条件。
- **L945 EN**: Opens a new lexical scope or compound statement.
  - **L945 CN**: 打开一个新的词法作用域或复合语句块。
- **L946 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L946 CN**: 开始 `if` 控制流语句并计算其条件。
- **L947 EN**: Opens a new lexical scope or compound statement.
  - **L947 CN**: 打开一个新的词法作用域或复合语句块。
- **L948 EN**: Executes a call or declaration centered on `pow`.
  - **L948 CN**: 执行以 `pow` 为核心的调用或声明。
- **L949 EN**: Closes the current lexical scope or compound statement.
  - **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Starts the alternative branch of the preceding conditional.
  - **L950 CN**: 开始前一个条件语句的备选分支。
- **L951 EN**: Opens a new lexical scope or compound statement.
  - **L951 CN**: 打开一个新的词法作用域或复合语句块。
- **L952 EN**: Executes a call or declaration centered on `pow`.
  - **L952 CN**: 执行以 `pow` 为核心的调用或声明。

### Lines 953-980 / 第 953-980 行

````cpp
 953:       }
 954:       else
 955:       {
 956:          prefix = exp(alz - z);  // LCOV_EXCL_LINE defensive programming, can probably never get here?
 957:       }
 958:    }
 959:    else
 960:    {
 961:       if(alz > tools::log_min_value<T>())
 962:       {
 963:          prefix = pow(z, a) * exp(-z);
 964:       }
 965:       // LCOV_EXCL_START
 966:       // Defensive programming, can probably never get here, very hard to prove though!
 967:       else if(z/a < tools::log_max_value<T>())
 968:       {
 969:          prefix = pow(T(z / exp(z/a)), a);
 970:       }
 971:       else
 972:       {
 973:          prefix = exp(alz - z);
 974:       }
 975:       // LCOV_EXCL_STOP
 976:    }
 977:    //
 978:    // This error handling isn't very good: it happens after the fact
 979:    // rather than before it...
 980:    // Typically though this method is used when the result is small, we should probably not overflow here...
````
- **L953 EN**: Closes the current lexical scope or compound statement.
  - **L953 CN**: 结束当前词法作用域或复合语句块。
- **L954 EN**: Starts the alternative branch of the preceding conditional.
  - **L954 CN**: 开始前一个条件语句的备选分支。
- **L955 EN**: Opens a new lexical scope or compound statement.
  - **L955 CN**: 打开一个新的词法作用域或复合语句块。
- **L956 EN**: Continues logic associated with callable symbol `exp`.
  - **L956 CN**: 继续与可调用符号 `exp` 相关的逻辑。
- **L957 EN**: Closes the current lexical scope or compound statement.
  - **L957 CN**: 结束当前词法作用域或复合语句块。
- **L958 EN**: Closes the current lexical scope or compound statement.
  - **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Starts the alternative branch of the preceding conditional.
  - **L959 CN**: 开始前一个条件语句的备选分支。
- **L960 EN**: Opens a new lexical scope or compound statement.
  - **L960 CN**: 打开一个新的词法作用域或复合语句块。
- **L961 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L961 CN**: 开始 `if` 控制流语句并计算其条件。
- **L962 EN**: Opens a new lexical scope or compound statement.
  - **L962 CN**: 打开一个新的词法作用域或复合语句块。
- **L963 EN**: Executes a call or declaration centered on `pow`.
  - **L963 CN**: 执行以 `pow` 为核心的调用或声明。
- **L964 EN**: Closes the current lexical scope or compound statement.
  - **L964 CN**: 结束当前词法作用域或复合语句块。
- **L965 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L965 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L966 EN**: Comment documents nearby intent or usage notes: `Defensive programming, can probably never get here, very hard to prove though!`.
  - **L966 CN**: 注释说明附近代码的意图或使用说明：`Defensive programming, can probably never get here, very hard to prove though!`。
- **L967 EN**: Starts the alternative branch of the preceding conditional.
  - **L967 CN**: 开始前一个条件语句的备选分支。
- **L968 EN**: Opens a new lexical scope or compound statement.
  - **L968 CN**: 打开一个新的词法作用域或复合语句块。
- **L969 EN**: Executes a call or declaration centered on `pow`.
  - **L969 CN**: 执行以 `pow` 为核心的调用或声明。
- **L970 EN**: Closes the current lexical scope or compound statement.
  - **L970 CN**: 结束当前词法作用域或复合语句块。
- **L971 EN**: Starts the alternative branch of the preceding conditional.
  - **L971 CN**: 开始前一个条件语句的备选分支。
- **L972 EN**: Opens a new lexical scope or compound statement.
  - **L972 CN**: 打开一个新的词法作用域或复合语句块。
- **L973 EN**: Executes a call or declaration centered on `exp`.
  - **L973 CN**: 执行以 `exp` 为核心的调用或声明。
- **L974 EN**: Closes the current lexical scope or compound statement.
  - **L974 CN**: 结束当前词法作用域或复合语句块。
- **L975 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L975 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L976 EN**: Closes the current lexical scope or compound statement.
  - **L976 CN**: 结束当前词法作用域或复合语句块。
- **L977 EN**: Separator comment used for visual grouping.
  - **L977 CN**: 分隔注释，用于视觉分组。
- **L978 EN**: Comment documents nearby intent or usage notes: `This error handling isn't very good: it happens after the fact`.
  - **L978 CN**: 注释说明附近代码的意图或使用说明：`This error handling isn't very good: it happens after the fact`。
- **L979 EN**: Comment documents nearby intent or usage notes: `rather than before it...`.
  - **L979 CN**: 注释说明附近代码的意图或使用说明：`rather than before it...`。
- **L980 EN**: Comment documents nearby intent or usage notes: `Typically though this method is used when the result is small, we should probably not overflow here...`.
  - **L980 CN**: 注释说明附近代码的意图或使用说明：`Typically though this method is used when the result is small, we should probably not overflow here...`。

### Lines 981-1008 / 第 981-1008 行

````cpp
 981:    //
 982:    if((boost::math::fpclassify)(prefix) == (int)BOOST_MATH_FP_INFINITE)
 983:       return policies::raise_overflow_error<T>("boost::math::detail::full_igamma_prefix<%1%>(%1%, %1%)", "Result of incomplete gamma function is too large to represent.", pol);  // LCOV_EXCL_LINE
 984: 
 985:    return prefix;
 986: }
 987: //
 988: // Compute (z^a)(e^-z)/tgamma(a)
 989: // most if the error occurs in this function:
 990: //
 991: template <class T, class Policy, class Lanczos>
 992: BOOST_MATH_GPU_ENABLED T regularised_gamma_prefix(T a, T z, const Policy& pol, const Lanczos& l)
 993: {
 994:    BOOST_MATH_STD_USING
 995:    if (z >= tools::max_value<T>() || (a > 0 && z == 0))
 996:       return 0;
 997:    T agh = a + static_cast<T>(Lanczos::g()) - T(0.5);
 998:    T prefix{};
 999:    T d = ((z - a) - static_cast<T>(Lanczos::g()) + T(0.5)) / agh;
1000: 
1001:    if(a < 1)
1002:    {
1003:       //
1004:       // We have to treat a < 1 as a special case because our Lanczos
1005:       // approximations are optimised against the factorials with a > 1,
1006:       // and for high precision types especially (128-bit reals for example)
1007:       // very small values of a can give rather erroneous results for gamma
1008:       // unless we do this:
````
- **L981 EN**: Separator comment used for visual grouping.
  - **L981 CN**: 分隔注释，用于视觉分组。
- **L982 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L982 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L983 EN**: Returns from the current function with `policies::raise_overflow_error<T>("boost::math::detail::full_igamma_prefix<%1%>(%1%, %1%)", "Result of incomplete gamma function is too large to represent.", pol);  // LCOV_EXCL_LINE`.
  - **L983 CN**: 以 `policies::raise_overflow_error<T>("boost::math::detail::full_igamma_prefix<%1%>(%1%, %1%)", "Result of incomplete gamma function is too large to represent.", pol);  // LCOV_EXCL_LINE` 从当前函数返回。
- **L984 EN**: Blank line separating nearby declarations or logic.
  - **L984 CN**: 空行，用于分隔相邻声明或逻辑。
- **L985 EN**: Returns from the current function with `prefix`.
  - **L985 CN**: 以 `prefix` 从当前函数返回。
- **L986 EN**: Closes the current lexical scope or compound statement.
  - **L986 CN**: 结束当前词法作用域或复合语句块。
- **L987 EN**: Separator comment used for visual grouping.
  - **L987 CN**: 分隔注释，用于视觉分组。
- **L988 EN**: Comment documents nearby intent or usage notes: `Compute (z^a)(e^-z)/tgamma(a)`.
  - **L988 CN**: 注释说明附近代码的意图或使用说明：`Compute (z^a)(e^-z)/tgamma(a)`。
- **L989 EN**: Comment documents nearby intent or usage notes: `most if the error occurs in this function:`.
  - **L989 CN**: 注释说明附近代码的意图或使用说明：`most if the error occurs in this function:`。
- **L990 EN**: Separator comment used for visual grouping.
  - **L990 CN**: 分隔注释，用于视觉分组。
- **L991 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L991 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L992 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L992 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L993 EN**: Opens a new lexical scope or compound statement.
  - **L993 CN**: 打开一个新的词法作用域或复合语句块。
- **L994 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L994 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L995 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L995 CN**: 开始 `if` 控制流语句并计算其条件。
- **L996 EN**: Returns from the current function with `0`.
  - **L996 CN**: 以 `0` 从当前函数返回。
- **L997 EN**: Executes a call or declaration centered on `static_cast<T>`.
  - **L997 CN**: 执行以 `static_cast<T>` 为核心的调用或声明。
- **L998 EN**: Executes a standalone statement or declaration: `T prefix{};`.
  - **L998 CN**: 执行一条独立语句或声明：`T prefix{};`。
- **L999 EN**: Executes a call or declaration centered on `=`.
  - **L999 CN**: 执行以 `=` 为核心的调用或声明。
- **L1000 EN**: Blank line separating nearby declarations or logic.
  - **L1000 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1001 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1001 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1002 EN**: Opens a new lexical scope or compound statement.
  - **L1002 CN**: 打开一个新的词法作用域或复合语句块。
- **L1003 EN**: Separator comment used for visual grouping.
  - **L1003 CN**: 分隔注释，用于视觉分组。
- **L1004 EN**: Comment documents nearby intent or usage notes: `We have to treat a < 1 as a special case because our Lanczos`.
  - **L1004 CN**: 注释说明附近代码的意图或使用说明：`We have to treat a < 1 as a special case because our Lanczos`。
- **L1005 EN**: Comment documents nearby intent or usage notes: `approximations are optimised against the factorials with a > 1,`.
  - **L1005 CN**: 注释说明附近代码的意图或使用说明：`approximations are optimised against the factorials with a > 1,`。
- **L1006 EN**: Comment documents nearby intent or usage notes: `and for high precision types especially (128-bit reals for example)`.
  - **L1006 CN**: 注释说明附近代码的意图或使用说明：`and for high precision types especially (128-bit reals for example)`。
- **L1007 EN**: Comment documents nearby intent or usage notes: `very small values of a can give rather erroneous results for gamma`.
  - **L1007 CN**: 注释说明附近代码的意图或使用说明：`very small values of a can give rather erroneous results for gamma`。
- **L1008 EN**: Comment documents nearby intent or usage notes: `unless we do this:`.
  - **L1008 CN**: 注释说明附近代码的意图或使用说明：`unless we do this:`。

### Lines 1009-1036 / 第 1009-1036 行

````cpp
1009:       //
1010:       // TODO: is this still required?  Lanczos approx should be better now?
1011:       //
1012:       if((z <= tools::log_min_value<T>()) || (a < 1 / tools::max_value<T>()))
1013:       {
1014:          // Oh dear, have to use logs, should be free of cancellation errors though:
1015:          return exp(a * log(z) - z - lgamma_imp(a, pol, l));
1016:       }
1017:       else
1018:       {
1019:          // direct calculation, no danger of overflow as gamma(a) < 1/a
1020:          // for small a.
1021:          return pow(z, a) * exp(-z) / gamma_imp(a, pol, l);
1022:       }
1023:    }
1024:    else if((fabs(d*d*a) <= 100) && (a > 150))
1025:    {
1026:       // special case for large a and a ~ z.
1027:       prefix = a * boost::math::log1pmx(d, pol) + z * static_cast<T>(0.5 - Lanczos::g()) / agh;
1028:       prefix = exp(prefix);
1029:    }
1030:    else
1031:    {
1032:       //
1033:       // general case.
1034:       // direct computation is most accurate, but use various fallbacks
1035:       // for different parts of the problem domain:
1036:       //
````
- **L1009 EN**: Separator comment used for visual grouping.
  - **L1009 CN**: 分隔注释，用于视觉分组。
- **L1010 EN**: Comment documents nearby intent or usage notes: `TODO: is this still required?  Lanczos approx should be better now?`.
  - **L1010 CN**: 注释说明附近代码的意图或使用说明：`TODO: is this still required?  Lanczos approx should be better now?`。
- **L1011 EN**: Separator comment used for visual grouping.
  - **L1011 CN**: 分隔注释，用于视觉分组。
- **L1012 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1012 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1013 EN**: Opens a new lexical scope or compound statement.
  - **L1013 CN**: 打开一个新的词法作用域或复合语句块。
- **L1014 EN**: Comment documents nearby intent or usage notes: `Oh dear, have to use logs, should be free of cancellation errors though:`.
  - **L1014 CN**: 注释说明附近代码的意图或使用说明：`Oh dear, have to use logs, should be free of cancellation errors though:`。
- **L1015 EN**: Returns from the current function with `exp(a * log(z) - z - lgamma_imp(a, pol, l))`.
  - **L1015 CN**: 以 `exp(a * log(z) - z - lgamma_imp(a, pol, l))` 从当前函数返回。
- **L1016 EN**: Closes the current lexical scope or compound statement.
  - **L1016 CN**: 结束当前词法作用域或复合语句块。
- **L1017 EN**: Starts the alternative branch of the preceding conditional.
  - **L1017 CN**: 开始前一个条件语句的备选分支。
- **L1018 EN**: Opens a new lexical scope or compound statement.
  - **L1018 CN**: 打开一个新的词法作用域或复合语句块。
- **L1019 EN**: Comment documents nearby intent or usage notes: `direct calculation, no danger of overflow as gamma(a) < 1/a`.
  - **L1019 CN**: 注释说明附近代码的意图或使用说明：`direct calculation, no danger of overflow as gamma(a) < 1/a`。
- **L1020 EN**: Comment documents nearby intent or usage notes: `for small a.`.
  - **L1020 CN**: 注释说明附近代码的意图或使用说明：`for small a.`。
- **L1021 EN**: Returns from the current function with `pow(z, a) * exp(-z) / gamma_imp(a, pol, l)`.
  - **L1021 CN**: 以 `pow(z, a) * exp(-z) / gamma_imp(a, pol, l)` 从当前函数返回。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  - **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Closes the current lexical scope or compound statement.
  - **L1023 CN**: 结束当前词法作用域或复合语句块。
- **L1024 EN**: Starts the alternative branch of the preceding conditional.
  - **L1024 CN**: 开始前一个条件语句的备选分支。
- **L1025 EN**: Opens a new lexical scope or compound statement.
  - **L1025 CN**: 打开一个新的词法作用域或复合语句块。
- **L1026 EN**: Comment documents nearby intent or usage notes: `special case for large a and a ~ z.`.
  - **L1026 CN**: 注释说明附近代码的意图或使用说明：`special case for large a and a ~ z.`。
- **L1027 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1027 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1028 EN**: Executes a call or declaration centered on `exp`.
  - **L1028 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1029 EN**: Closes the current lexical scope or compound statement.
  - **L1029 CN**: 结束当前词法作用域或复合语句块。
- **L1030 EN**: Starts the alternative branch of the preceding conditional.
  - **L1030 CN**: 开始前一个条件语句的备选分支。
- **L1031 EN**: Opens a new lexical scope or compound statement.
  - **L1031 CN**: 打开一个新的词法作用域或复合语句块。
- **L1032 EN**: Separator comment used for visual grouping.
  - **L1032 CN**: 分隔注释，用于视觉分组。
- **L1033 EN**: Comment documents nearby intent or usage notes: `general case.`.
  - **L1033 CN**: 注释说明附近代码的意图或使用说明：`general case.`。
- **L1034 EN**: Comment documents nearby intent or usage notes: `direct computation is most accurate, but use various fallbacks`.
  - **L1034 CN**: 注释说明附近代码的意图或使用说明：`direct computation is most accurate, but use various fallbacks`。
- **L1035 EN**: Comment documents nearby intent or usage notes: `for different parts of the problem domain:`.
  - **L1035 CN**: 注释说明附近代码的意图或使用说明：`for different parts of the problem domain:`。
- **L1036 EN**: Separator comment used for visual grouping.
  - **L1036 CN**: 分隔注释，用于视觉分组。

### Lines 1037-1064 / 第 1037-1064 行

````cpp
1037:       T alz = a * log(z / agh);
1038:       T amz = a - z;
1039:       if((BOOST_MATH_GPU_SAFE_MIN(alz, amz) <= tools::log_min_value<T>()) || (BOOST_MATH_GPU_SAFE_MAX(alz, amz) >= tools::log_max_value<T>()))
1040:       {
1041:          T amza = amz / a;
1042:          if((BOOST_MATH_GPU_SAFE_MIN(alz, amz)/2 > tools::log_min_value<T>()) && (BOOST_MATH_GPU_SAFE_MAX(alz, amz)/2 < tools::log_max_value<T>()))
1043:          {
1044:             // compute square root of the result and then square it:
1045:             T sq = pow(z / agh, a / 2) * exp(amz / 2);
1046:             prefix = sq * sq;
1047:          }
1048:          else if((BOOST_MATH_GPU_SAFE_MIN(alz, amz)/4 > tools::log_min_value<T>()) && (BOOST_MATH_GPU_SAFE_MAX(alz, amz)/4 < tools::log_max_value<T>()) && (z > a))
1049:          {
1050:             // compute the 4th root of the result then square it twice:
1051:             T sq = pow(z / agh, a / 4) * exp(amz / 4);
1052:             prefix = sq * sq;
1053:             prefix *= prefix;
1054:          }
1055:          else if((amza > tools::log_min_value<T>()) && (amza < tools::log_max_value<T>()))
1056:          {
1057:             prefix = pow(T((z * exp(amza)) / agh), a);
1058:          }
1059:          else
1060:          {
1061:             prefix = exp(alz + amz);
1062:          }
1063:       }
1064:       else
````
- **L1037 EN**: Executes a call or declaration centered on `log`.
  - **L1037 CN**: 执行以 `log` 为核心的调用或声明。
- **L1038 EN**: Executes a standalone statement or declaration: `T amz = a - z;`.
  - **L1038 CN**: 执行一条独立语句或声明：`T amz = a - z;`。
- **L1039 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1039 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1040 EN**: Opens a new lexical scope or compound statement.
  - **L1040 CN**: 打开一个新的词法作用域或复合语句块。
- **L1041 EN**: Executes a standalone statement or declaration: `T amza = amz / a;`.
  - **L1041 CN**: 执行一条独立语句或声明：`T amza = amz / a;`。
- **L1042 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1042 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1043 EN**: Opens a new lexical scope or compound statement.
  - **L1043 CN**: 打开一个新的词法作用域或复合语句块。
- **L1044 EN**: Comment documents nearby intent or usage notes: `compute square root of the result and then square it:`.
  - **L1044 CN**: 注释说明附近代码的意图或使用说明：`compute square root of the result and then square it:`。
- **L1045 EN**: Executes a call or declaration centered on `pow`.
  - **L1045 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1046 EN**: Executes a standalone statement or declaration: `prefix = sq * sq;`.
  - **L1046 CN**: 执行一条独立语句或声明：`prefix = sq * sq;`。
- **L1047 EN**: Closes the current lexical scope or compound statement.
  - **L1047 CN**: 结束当前词法作用域或复合语句块。
- **L1048 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1048 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1049 EN**: Opens a new lexical scope or compound statement.
  - **L1049 CN**: 打开一个新的词法作用域或复合语句块。
- **L1050 EN**: Comment documents nearby intent or usage notes: `compute the 4th root of the result then square it twice:`.
  - **L1050 CN**: 注释说明附近代码的意图或使用说明：`compute the 4th root of the result then square it twice:`。
- **L1051 EN**: Executes a call or declaration centered on `pow`.
  - **L1051 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1052 EN**: Executes a standalone statement or declaration: `prefix = sq * sq;`.
  - **L1052 CN**: 执行一条独立语句或声明：`prefix = sq * sq;`。
- **L1053 EN**: Executes a standalone statement or declaration: `prefix *= prefix;`.
  - **L1053 CN**: 执行一条独立语句或声明：`prefix *= prefix;`。
- **L1054 EN**: Closes the current lexical scope or compound statement.
  - **L1054 CN**: 结束当前词法作用域或复合语句块。
- **L1055 EN**: Starts the alternative branch of the preceding conditional.
  - **L1055 CN**: 开始前一个条件语句的备选分支。
- **L1056 EN**: Opens a new lexical scope or compound statement.
  - **L1056 CN**: 打开一个新的词法作用域或复合语句块。
- **L1057 EN**: Executes a call or declaration centered on `pow`.
  - **L1057 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1058 EN**: Closes the current lexical scope or compound statement.
  - **L1058 CN**: 结束当前词法作用域或复合语句块。
- **L1059 EN**: Starts the alternative branch of the preceding conditional.
  - **L1059 CN**: 开始前一个条件语句的备选分支。
- **L1060 EN**: Opens a new lexical scope or compound statement.
  - **L1060 CN**: 打开一个新的词法作用域或复合语句块。
- **L1061 EN**: Executes a call or declaration centered on `exp`.
  - **L1061 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  - **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Closes the current lexical scope or compound statement.
  - **L1063 CN**: 结束当前词法作用域或复合语句块。
- **L1064 EN**: Starts the alternative branch of the preceding conditional.
  - **L1064 CN**: 开始前一个条件语句的备选分支。

### Lines 1065-1092 / 第 1065-1092 行

````cpp
1065:       {
1066:          prefix = pow(T(z / agh), a) * exp(amz);
1067:       }
1068:    }
1069:    prefix *= sqrt(agh / boost::math::constants::e<T>()) / Lanczos::lanczos_sum_expG_scaled(a);
1070:    return prefix;
1071: }
1072: 
1073: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
1074: 
1075: //
1076: // And again, without Lanczos support:
1077: //
1078: template <class T, class Policy>
1079: T regularised_gamma_prefix(T a, T z, const Policy& pol, const lanczos::undefined_lanczos& l)
1080: {
1081:    BOOST_MATH_STD_USING
1082: 
1083:    if((a < 1) && (z < 1))
1084:    {
1085:       // No overflow possible since the power terms tend to unity as a,z -> 0
1086:       return pow(z, a) * exp(-z) / boost::math::tgamma(a, pol);
1087:    }
1088:    else if(a > minimum_argument_for_bernoulli_recursion<T>())
1089:    {
1090:       T scaled_gamma = scaled_tgamma_no_lanczos(a, pol);
1091:       T power_term = pow(z / a, a / 2);
1092:       T a_minus_z = a - z;
````
- **L1065 EN**: Opens a new lexical scope or compound statement.
  - **L1065 CN**: 打开一个新的词法作用域或复合语句块。
- **L1066 EN**: Executes a call or declaration centered on `pow`.
  - **L1066 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1067 EN**: Closes the current lexical scope or compound statement.
  - **L1067 CN**: 结束当前词法作用域或复合语句块。
- **L1068 EN**: Closes the current lexical scope or compound statement.
  - **L1068 CN**: 结束当前词法作用域或复合语句块。
- **L1069 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1069 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1070 EN**: Returns from the current function with `prefix`.
  - **L1070 CN**: 以 `prefix` 从当前函数返回。
- **L1071 EN**: Closes the current lexical scope or compound statement.
  - **L1071 CN**: 结束当前词法作用域或复合语句块。
- **L1072 EN**: Blank line separating nearby declarations or logic.
  - **L1072 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1073 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_HAS_GPU_SUPPORT`.
  - **L1073 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_HAS_GPU_SUPPORT`。
- **L1074 EN**: Blank line separating nearby declarations or logic.
  - **L1074 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1075 EN**: Separator comment used for visual grouping.
  - **L1075 CN**: 分隔注释，用于视觉分组。
- **L1076 EN**: Comment documents nearby intent or usage notes: `And again, without Lanczos support:`.
  - **L1076 CN**: 注释说明附近代码的意图或使用说明：`And again, without Lanczos support:`。
- **L1077 EN**: Separator comment used for visual grouping.
  - **L1077 CN**: 分隔注释，用于视觉分组。
- **L1078 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1078 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1079 EN**: Continues logic associated with callable symbol `regularised_gamma_prefix`.
  - **L1079 CN**: 继续与可调用符号 `regularised_gamma_prefix` 相关的逻辑。
- **L1080 EN**: Opens a new lexical scope or compound statement.
  - **L1080 CN**: 打开一个新的词法作用域或复合语句块。
- **L1081 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1081 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1082 EN**: Blank line separating nearby declarations or logic.
  - **L1082 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1083 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1083 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1084 EN**: Opens a new lexical scope or compound statement.
  - **L1084 CN**: 打开一个新的词法作用域或复合语句块。
- **L1085 EN**: Comment documents nearby intent or usage notes: `No overflow possible since the power terms tend to unity as a,z -> 0`.
  - **L1085 CN**: 注释说明附近代码的意图或使用说明：`No overflow possible since the power terms tend to unity as a,z -> 0`。
- **L1086 EN**: Returns from the current function with `pow(z, a) * exp(-z) / boost::math::tgamma(a, pol)`.
  - **L1086 CN**: 以 `pow(z, a) * exp(-z) / boost::math::tgamma(a, pol)` 从当前函数返回。
- **L1087 EN**: Closes the current lexical scope or compound statement.
  - **L1087 CN**: 结束当前词法作用域或复合语句块。
- **L1088 EN**: Starts the alternative branch of the preceding conditional.
  - **L1088 CN**: 开始前一个条件语句的备选分支。
- **L1089 EN**: Opens a new lexical scope or compound statement.
  - **L1089 CN**: 打开一个新的词法作用域或复合语句块。
- **L1090 EN**: Executes a call or declaration centered on `scaled_tgamma_no_lanczos`.
  - **L1090 CN**: 执行以 `scaled_tgamma_no_lanczos` 为核心的调用或声明。
- **L1091 EN**: Executes a call or declaration centered on `pow`.
  - **L1091 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1092 EN**: Executes a standalone statement or declaration: `T a_minus_z = a - z;`.
  - **L1092 CN**: 执行一条独立语句或声明：`T a_minus_z = a - z;`。

### Lines 1093-1120 / 第 1093-1120 行

````cpp
1093:       if ((0 == power_term) || (fabs(a_minus_z) > tools::log_max_value<T>()))
1094:       {
1095:          // The result is probably zero, but we need to be sure:
1096:          return exp(a * log(z / a) + a_minus_z - log(scaled_gamma));
1097:       }
1098:       return (power_term * exp(a_minus_z)) * (power_term / scaled_gamma);
1099:    }
1100:    else
1101:    {
1102:       //
1103:       // Usual case is to calculate the prefix at a+shift and recurse down
1104:       // to the value we want:
1105:       //
1106:       const int min_z = minimum_argument_for_bernoulli_recursion<T>();
1107:       long shift = 1 + ltrunc(min_z - a);
1108:       T result = regularised_gamma_prefix(T(a + shift), z, pol, l);
1109:       if (result != 0)
1110:       {
1111:          for (long i = 0; i < shift; ++i)
1112:          {
1113:             result /= z;
1114:             result *= a + i;
1115:          }
1116:          return result;
1117:       }
1118:       else
1119:       {
1120:          //
````
- **L1093 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1093 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1094 EN**: Opens a new lexical scope or compound statement.
  - **L1094 CN**: 打开一个新的词法作用域或复合语句块。
- **L1095 EN**: Comment documents nearby intent or usage notes: `The result is probably zero, but we need to be sure:`.
  - **L1095 CN**: 注释说明附近代码的意图或使用说明：`The result is probably zero, but we need to be sure:`。
- **L1096 EN**: Returns from the current function with `exp(a * log(z / a) + a_minus_z - log(scaled_gamma))`.
  - **L1096 CN**: 以 `exp(a * log(z / a) + a_minus_z - log(scaled_gamma))` 从当前函数返回。
- **L1097 EN**: Closes the current lexical scope or compound statement.
  - **L1097 CN**: 结束当前词法作用域或复合语句块。
- **L1098 EN**: Returns from the current function with `(power_term * exp(a_minus_z)) * (power_term / scaled_gamma)`.
  - **L1098 CN**: 以 `(power_term * exp(a_minus_z)) * (power_term / scaled_gamma)` 从当前函数返回。
- **L1099 EN**: Closes the current lexical scope or compound statement.
  - **L1099 CN**: 结束当前词法作用域或复合语句块。
- **L1100 EN**: Starts the alternative branch of the preceding conditional.
  - **L1100 CN**: 开始前一个条件语句的备选分支。
- **L1101 EN**: Opens a new lexical scope or compound statement.
  - **L1101 CN**: 打开一个新的词法作用域或复合语句块。
- **L1102 EN**: Separator comment used for visual grouping.
  - **L1102 CN**: 分隔注释，用于视觉分组。
- **L1103 EN**: Comment documents nearby intent or usage notes: `Usual case is to calculate the prefix at a+shift and recurse down`.
  - **L1103 CN**: 注释说明附近代码的意图或使用说明：`Usual case is to calculate the prefix at a+shift and recurse down`。
- **L1104 EN**: Comment documents nearby intent or usage notes: `to the value we want:`.
  - **L1104 CN**: 注释说明附近代码的意图或使用说明：`to the value we want:`。
- **L1105 EN**: Separator comment used for visual grouping.
  - **L1105 CN**: 分隔注释，用于视觉分组。
- **L1106 EN**: Initializes variable `min_z` from the right-hand expression.
  - **L1106 CN**: 使用右侧表达式初始化变量 `min_z`。
- **L1107 EN**: Initializes variable `shift` from the right-hand expression.
  - **L1107 CN**: 使用右侧表达式初始化变量 `shift`。
- **L1108 EN**: Executes a call or declaration centered on `regularised_gamma_prefix`.
  - **L1108 CN**: 执行以 `regularised_gamma_prefix` 为核心的调用或声明。
- **L1109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1110 EN**: Opens a new lexical scope or compound statement.
  - **L1110 CN**: 打开一个新的词法作用域或复合语句块。
- **L1111 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L1111 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1112 EN**: Opens a new lexical scope or compound statement.
  - **L1112 CN**: 打开一个新的词法作用域或复合语句块。
- **L1113 EN**: Executes a standalone statement or declaration: `result /= z;`.
  - **L1113 CN**: 执行一条独立语句或声明：`result /= z;`。
- **L1114 EN**: Executes a standalone statement or declaration: `result *= a + i;`.
  - **L1114 CN**: 执行一条独立语句或声明：`result *= a + i;`。
- **L1115 EN**: Closes the current lexical scope or compound statement.
  - **L1115 CN**: 结束当前词法作用域或复合语句块。
- **L1116 EN**: Returns from the current function with `result`.
  - **L1116 CN**: 以 `result` 从当前函数返回。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  - **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Starts the alternative branch of the preceding conditional.
  - **L1118 CN**: 开始前一个条件语句的备选分支。
- **L1119 EN**: Opens a new lexical scope or compound statement.
  - **L1119 CN**: 打开一个新的词法作用域或复合语句块。
- **L1120 EN**: Separator comment used for visual grouping.
  - **L1120 CN**: 分隔注释，用于视觉分组。

### Lines 1121-1148 / 第 1121-1148 行

````cpp
1121:          // We failed, most probably we have z << 1, try again, this time
1122:          // we calculate z^a e^-z / tgamma(a+shift), combining power terms
1123:          // as we go.  And again recurse down to the result.
1124:          //
1125:          T scaled_gamma = scaled_tgamma_no_lanczos(T(a + shift), pol);
1126:          T power_term_1 = pow(T(z / (a + shift)), a);
1127:          T power_term_2 = pow(T(a + shift), T(-shift));
1128:          T power_term_3 = exp(a + shift - z);
1129:          if ((0 == power_term_1) || (0 == power_term_2) || (0 == power_term_3) || (fabs(a + shift - z) > tools::log_max_value<T>()))
1130:          {
1131:             // We have no test case that gets here, most likely the type T
1132:             // has a high precision but low exponent range:
1133:             return exp(a * log(z) - z - boost::math::lgamma(a, pol));
1134:          }
1135:          result = power_term_1 * power_term_2 * power_term_3 / scaled_gamma;
1136:          for (long i = 0; i < shift; ++i)
1137:          {
1138:             result *= a + i;
1139:          }
1140:          return result;
1141:       }
1142:    }
1143: }
1144: 
1145: #endif // BOOST_MATH_HAS_GPU_SUPPORT
1146: 
1147: //
1148: // Upper gamma fraction for very small a:
````
- **L1121 EN**: Comment documents nearby intent or usage notes: `We failed, most probably we have z << 1, try again, this time`.
  - **L1121 CN**: 注释说明附近代码的意图或使用说明：`We failed, most probably we have z << 1, try again, this time`。
- **L1122 EN**: Comment documents nearby intent or usage notes: `we calculate z^a e^-z / tgamma(a+shift), combining power terms`.
  - **L1122 CN**: 注释说明附近代码的意图或使用说明：`we calculate z^a e^-z / tgamma(a+shift), combining power terms`。
- **L1123 EN**: Comment documents nearby intent or usage notes: `as we go.  And again recurse down to the result.`.
  - **L1123 CN**: 注释说明附近代码的意图或使用说明：`as we go.  And again recurse down to the result.`。
- **L1124 EN**: Separator comment used for visual grouping.
  - **L1124 CN**: 分隔注释，用于视觉分组。
- **L1125 EN**: Executes a call or declaration centered on `scaled_tgamma_no_lanczos`.
  - **L1125 CN**: 执行以 `scaled_tgamma_no_lanczos` 为核心的调用或声明。
- **L1126 EN**: Executes a call or declaration centered on `pow`.
  - **L1126 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1127 EN**: Executes a call or declaration centered on `pow`.
  - **L1127 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1128 EN**: Executes a call or declaration centered on `exp`.
  - **L1128 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1129 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1129 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1130 EN**: Opens a new lexical scope or compound statement.
  - **L1130 CN**: 打开一个新的词法作用域或复合语句块。
- **L1131 EN**: Comment documents nearby intent or usage notes: `We have no test case that gets here, most likely the type T`.
  - **L1131 CN**: 注释说明附近代码的意图或使用说明：`We have no test case that gets here, most likely the type T`。
- **L1132 EN**: Comment documents nearby intent or usage notes: `has a high precision but low exponent range:`.
  - **L1132 CN**: 注释说明附近代码的意图或使用说明：`has a high precision but low exponent range:`。
- **L1133 EN**: Returns from the current function with `exp(a * log(z) - z - boost::math::lgamma(a, pol))`.
  - **L1133 CN**: 以 `exp(a * log(z) - z - boost::math::lgamma(a, pol))` 从当前函数返回。
- **L1134 EN**: Closes the current lexical scope or compound statement.
  - **L1134 CN**: 结束当前词法作用域或复合语句块。
- **L1135 EN**: Executes a standalone statement or declaration: `result = power_term_1 * power_term_2 * power_term_3 / scaled_gamma;`.
  - **L1135 CN**: 执行一条独立语句或声明：`result = power_term_1 * power_term_2 * power_term_3 / scaled_gamma;`。
- **L1136 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L1136 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1137 EN**: Opens a new lexical scope or compound statement.
  - **L1137 CN**: 打开一个新的词法作用域或复合语句块。
- **L1138 EN**: Executes a standalone statement or declaration: `result *= a + i;`.
  - **L1138 CN**: 执行一条独立语句或声明：`result *= a + i;`。
- **L1139 EN**: Closes the current lexical scope or compound statement.
  - **L1139 CN**: 结束当前词法作用域或复合语句块。
- **L1140 EN**: Returns from the current function with `result`.
  - **L1140 CN**: 以 `result` 从当前函数返回。
- **L1141 EN**: Closes the current lexical scope or compound statement.
  - **L1141 CN**: 结束当前词法作用域或复合语句块。
- **L1142 EN**: Closes the current lexical scope or compound statement.
  - **L1142 CN**: 结束当前词法作用域或复合语句块。
- **L1143 EN**: Closes the current lexical scope or compound statement.
  - **L1143 CN**: 结束当前词法作用域或复合语句块。
- **L1144 EN**: Blank line separating nearby declarations or logic.
  - **L1144 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1145 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1145 CN**: 结束当前预处理条件块或头文件保护。
- **L1146 EN**: Blank line separating nearby declarations or logic.
  - **L1146 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1147 EN**: Separator comment used for visual grouping.
  - **L1147 CN**: 分隔注释，用于视觉分组。
- **L1148 EN**: Comment documents nearby intent or usage notes: `Upper gamma fraction for very small a:`.
  - **L1148 CN**: 注释说明附近代码的意图或使用说明：`Upper gamma fraction for very small a:`。

### Lines 1149-1176 / 第 1149-1176 行

````cpp
1149: //
1150: template <class T, class Policy>
1151: BOOST_MATH_GPU_ENABLED inline T tgamma_small_upper_part(T a, T x, const Policy& pol, T* pgam = 0, bool invert = false, T* pderivative = 0)
1152: {
1153:    BOOST_MATH_STD_USING  // ADL of std functions.
1154:    //
1155:    // Compute the full upper fraction (Q) when a is very small:
1156:    //
1157: 
1158:    #ifdef BOOST_MATH_HAS_NVRTC
1159:    typedef typename tools::promote_args<T>::type result_type;
1160:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1161:    typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
1162:    T result {detail::tgammap1m1_imp(static_cast<value_type>(a), pol, evaluation_type())};
1163:    #else
1164:    T result { boost::math::tgamma1pm1(a, pol) };
1165:    #endif
1166: 
1167:    if(pgam)
1168:       *pgam = (result + 1) / a;
1169:    T p = boost::math::powm1(x, a, pol);
1170:    result -= p;
1171:    result /= a;
1172:    detail::small_gamma2_series<T> s(a, x);
1173:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>() - 10;
1174:    p += 1;
1175:    if(pderivative)
1176:       *pderivative = p / (*pgam * exp(x));
````
- **L1149 EN**: Separator comment used for visual grouping.
  - **L1149 CN**: 分隔注释，用于视觉分组。
- **L1150 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1150 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1151 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1151 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1152 EN**: Opens a new lexical scope or compound statement.
  - **L1152 CN**: 打开一个新的词法作用域或复合语句块。
- **L1153 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1153 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1154 EN**: Separator comment used for visual grouping.
  - **L1154 CN**: 分隔注释，用于视觉分组。
- **L1155 EN**: Comment documents nearby intent or usage notes: `Compute the full upper fraction (Q) when a is very small:`.
  - **L1155 CN**: 注释说明附近代码的意图或使用说明：`Compute the full upper fraction (Q) when a is very small:`。
- **L1156 EN**: Separator comment used for visual grouping.
  - **L1156 CN**: 分隔注释，用于视觉分组。
- **L1157 EN**: Blank line separating nearby declarations or logic.
  - **L1157 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1158 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1158 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1159 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T>::type result_type;`.
  - **L1159 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T>::type result_type;`。
- **L1160 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L1160 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L1161 EN**: Introduces a legacy type alias or function typedef: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L1161 CN**: 引入传统类型别名或函数 typedef：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L1162 EN**: Executes a call or declaration centered on `{detail::tgammap1m1_imp`.
  - **L1162 CN**: 执行以 `{detail::tgammap1m1_imp` 为核心的调用或声明。
- **L1163 EN**: Continues the current preprocessor branch selection.
  - **L1163 CN**: 继续当前的预处理分支选择。
- **L1164 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1164 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1165 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1165 CN**: 结束当前预处理条件块或头文件保护。
- **L1166 EN**: Blank line separating nearby declarations or logic.
  - **L1166 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1167 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1167 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1168 EN**: Comment documents nearby intent or usage notes: `pgam = (result + 1) / a;`.
  - **L1168 CN**: 注释说明附近代码的意图或使用说明：`pgam = (result + 1) / a;`。
- **L1169 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1169 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1170 EN**: Executes a standalone statement or declaration: `result -= p;`.
  - **L1170 CN**: 执行一条独立语句或声明：`result -= p;`。
- **L1171 EN**: Executes a standalone statement or declaration: `result /= a;`.
  - **L1171 CN**: 执行一条独立语句或声明：`result /= a;`。
- **L1172 EN**: Executes a call or declaration centered on `s`.
  - **L1172 CN**: 执行以 `s` 为核心的调用或声明。
- **L1173 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1173 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1174 EN**: Executes a standalone statement or declaration: `p += 1;`.
  - **L1174 CN**: 执行一条独立语句或声明：`p += 1;`。
- **L1175 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1175 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1176 EN**: Comment documents nearby intent or usage notes: `pderivative = p / (*pgam * exp(x));`.
  - **L1176 CN**: 注释说明附近代码的意图或使用说明：`pderivative = p / (*pgam * exp(x));`。

### Lines 1177-1204 / 第 1177-1204 行

````cpp
1177:    T init_value = invert ? *pgam : 0;
1178:    result = -p * tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter, (init_value - result) / p);
1179:    policies::check_series_iterations<T>("boost::math::tgamma_small_upper_part<%1%>(%1%, %1%)", max_iter, pol);
1180:    if(invert)
1181:       result = -result;
1182:    return result;
1183: }
1184: //
1185: // Upper gamma fraction for integer a:
1186: //
1187: template <class T, class Policy>
1188: BOOST_MATH_GPU_ENABLED inline T finite_gamma_q(T a, T x, Policy const& pol, T* pderivative = 0)
1189: {
1190:    //
1191:    // Calculates normalised Q when a is an integer:
1192:    //
1193:    BOOST_MATH_STD_USING
1194:    T e = exp(-x);
1195:    T sum = e;
1196:    if(sum != 0)
1197:    {
1198:       T term = sum;
1199:       for(unsigned n = 1; n < a; ++n)
1200:       {
1201:          term /= n;
1202:          term *= x;
1203:          sum += term;
1204:       }
````
- **L1177 EN**: Executes a standalone statement or declaration: `T init_value = invert ? *pgam : 0;`.
  - **L1177 CN**: 执行一条独立语句或声明：`T init_value = invert ? *pgam : 0;`。
- **L1178 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1178 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1179 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1179 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1180 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1180 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1181 EN**: Executes a standalone statement or declaration: `result = -result;`.
  - **L1181 CN**: 执行一条独立语句或声明：`result = -result;`。
- **L1182 EN**: Returns from the current function with `result`.
  - **L1182 CN**: 以 `result` 从当前函数返回。
- **L1183 EN**: Closes the current lexical scope or compound statement.
  - **L1183 CN**: 结束当前词法作用域或复合语句块。
- **L1184 EN**: Separator comment used for visual grouping.
  - **L1184 CN**: 分隔注释，用于视觉分组。
- **L1185 EN**: Comment documents nearby intent or usage notes: `Upper gamma fraction for integer a:`.
  - **L1185 CN**: 注释说明附近代码的意图或使用说明：`Upper gamma fraction for integer a:`。
- **L1186 EN**: Separator comment used for visual grouping.
  - **L1186 CN**: 分隔注释，用于视觉分组。
- **L1187 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1187 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1188 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1188 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1189 EN**: Opens a new lexical scope or compound statement.
  - **L1189 CN**: 打开一个新的词法作用域或复合语句块。
- **L1190 EN**: Separator comment used for visual grouping.
  - **L1190 CN**: 分隔注释，用于视觉分组。
- **L1191 EN**: Comment documents nearby intent or usage notes: `Calculates normalised Q when a is an integer:`.
  - **L1191 CN**: 注释说明附近代码的意图或使用说明：`Calculates normalised Q when a is an integer:`。
- **L1192 EN**: Separator comment used for visual grouping.
  - **L1192 CN**: 分隔注释，用于视觉分组。
- **L1193 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1193 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1194 EN**: Executes a call or declaration centered on `exp`.
  - **L1194 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1195 EN**: Executes a standalone statement or declaration: `T sum = e;`.
  - **L1195 CN**: 执行一条独立语句或声明：`T sum = e;`。
- **L1196 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1196 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1197 EN**: Opens a new lexical scope or compound statement.
  - **L1197 CN**: 打开一个新的词法作用域或复合语句块。
- **L1198 EN**: Executes a standalone statement or declaration: `T term = sum;`.
  - **L1198 CN**: 执行一条独立语句或声明：`T term = sum;`。
- **L1199 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L1199 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1200 EN**: Opens a new lexical scope or compound statement.
  - **L1200 CN**: 打开一个新的词法作用域或复合语句块。
- **L1201 EN**: Executes a standalone statement or declaration: `term /= n;`.
  - **L1201 CN**: 执行一条独立语句或声明：`term /= n;`。
- **L1202 EN**: Executes a standalone statement or declaration: `term *= x;`.
  - **L1202 CN**: 执行一条独立语句或声明：`term *= x;`。
- **L1203 EN**: Executes a standalone statement or declaration: `sum += term;`.
  - **L1203 CN**: 执行一条独立语句或声明：`sum += term;`。
- **L1204 EN**: Closes the current lexical scope or compound statement.
  - **L1204 CN**: 结束当前词法作用域或复合语句块。

### Lines 1205-1232 / 第 1205-1232 行

````cpp
1205:    }
1206:    if(pderivative)
1207:    {
1208:       *pderivative = e * pow(x, a) / boost::math::unchecked_factorial<T>(itrunc(T(a - 1), pol));
1209:    }
1210:    return sum;
1211: }
1212: //
1213: // Upper gamma fraction for half integer a:
1214: //
1215: template <class T, class Policy>
1216: BOOST_MATH_GPU_ENABLED T finite_half_gamma_q(T a, T x, T* p_derivative, const Policy& pol)
1217: {
1218:    //
1219:    // Calculates normalised Q when a is a half-integer:
1220:    //
1221:    BOOST_MATH_STD_USING
1222: 
1223:    #ifdef BOOST_MATH_HAS_NVRTC
1224:    T e;
1225:    if (boost::math::is_same_v<T, float>)
1226:    {
1227:       e = ::erfcf(::sqrtf(x));
1228:    }
1229:    else
1230:    {
1231:       e = ::erfc(::sqrt(x));
1232:    }
````
- **L1205 EN**: Closes the current lexical scope or compound statement.
  - **L1205 CN**: 结束当前词法作用域或复合语句块。
- **L1206 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1206 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1207 EN**: Opens a new lexical scope or compound statement.
  - **L1207 CN**: 打开一个新的词法作用域或复合语句块。
- **L1208 EN**: Comment documents nearby intent or usage notes: `pderivative = e * pow(x, a) / boost::math::unchecked_factorial<T>(itrunc(T(a - 1), pol));`.
  - **L1208 CN**: 注释说明附近代码的意图或使用说明：`pderivative = e * pow(x, a) / boost::math::unchecked_factorial<T>(itrunc(T(a - 1), pol));`。
- **L1209 EN**: Closes the current lexical scope or compound statement.
  - **L1209 CN**: 结束当前词法作用域或复合语句块。
- **L1210 EN**: Returns from the current function with `sum`.
  - **L1210 CN**: 以 `sum` 从当前函数返回。
- **L1211 EN**: Closes the current lexical scope or compound statement.
  - **L1211 CN**: 结束当前词法作用域或复合语句块。
- **L1212 EN**: Separator comment used for visual grouping.
  - **L1212 CN**: 分隔注释，用于视觉分组。
- **L1213 EN**: Comment documents nearby intent or usage notes: `Upper gamma fraction for half integer a:`.
  - **L1213 CN**: 注释说明附近代码的意图或使用说明：`Upper gamma fraction for half integer a:`。
- **L1214 EN**: Separator comment used for visual grouping.
  - **L1214 CN**: 分隔注释，用于视觉分组。
- **L1215 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1215 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1216 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1216 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1217 EN**: Opens a new lexical scope or compound statement.
  - **L1217 CN**: 打开一个新的词法作用域或复合语句块。
- **L1218 EN**: Separator comment used for visual grouping.
  - **L1218 CN**: 分隔注释，用于视觉分组。
- **L1219 EN**: Comment documents nearby intent or usage notes: `Calculates normalised Q when a is a half-integer:`.
  - **L1219 CN**: 注释说明附近代码的意图或使用说明：`Calculates normalised Q when a is a half-integer:`。
- **L1220 EN**: Separator comment used for visual grouping.
  - **L1220 CN**: 分隔注释，用于视觉分组。
- **L1221 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1221 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1222 EN**: Blank line separating nearby declarations or logic.
  - **L1222 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1223 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1223 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1224 EN**: Executes a standalone statement or declaration: `T e;`.
  - **L1224 CN**: 执行一条独立语句或声明：`T e;`。
- **L1225 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1225 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1226 EN**: Opens a new lexical scope or compound statement.
  - **L1226 CN**: 打开一个新的词法作用域或复合语句块。
- **L1227 EN**: Executes a call or declaration centered on `::erfcf`.
  - **L1227 CN**: 执行以 `::erfcf` 为核心的调用或声明。
- **L1228 EN**: Closes the current lexical scope or compound statement.
  - **L1228 CN**: 结束当前词法作用域或复合语句块。
- **L1229 EN**: Starts the alternative branch of the preceding conditional.
  - **L1229 CN**: 开始前一个条件语句的备选分支。
- **L1230 EN**: Opens a new lexical scope or compound statement.
  - **L1230 CN**: 打开一个新的词法作用域或复合语句块。
- **L1231 EN**: Executes a call or declaration centered on `::erfc`.
  - **L1231 CN**: 执行以 `::erfc` 为核心的调用或声明。
- **L1232 EN**: Closes the current lexical scope or compound statement.
  - **L1232 CN**: 结束当前词法作用域或复合语句块。

### Lines 1233-1260 / 第 1233-1260 行

````cpp
1233:    #else
1234:    T e = boost::math::erfc(sqrt(x), pol);
1235:    #endif
1236: 
1237:    if((e != 0) && (a > 1))
1238:    {
1239:       T term = exp(-x) / sqrt(constants::pi<T>() * x);
1240:       term *= x;
1241:       static const T half = T(1) / 2; // LCOV_EXCL_LINE
1242:       term /= half;
1243:       T sum = term;
1244:       for(unsigned n = 2; n < a; ++n)
1245:       {
1246:          term /= n - half;
1247:          term *= x;
1248:          sum += term;
1249:       }
1250:       e += sum;
1251:       if(p_derivative)
1252:       {
1253:          *p_derivative = 0;
1254:       }
1255:    }
1256:    else if(p_derivative)
1257:    {
1258:       // We'll be dividing by x later, so calculate derivative * x:
1259:       *p_derivative = sqrt(x) * exp(-x) / constants::root_pi<T>();
1260:    }
````
- **L1233 EN**: Continues the current preprocessor branch selection.
  - **L1233 CN**: 继续当前的预处理分支选择。
- **L1234 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1234 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1235 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1235 CN**: 结束当前预处理条件块或头文件保护。
- **L1236 EN**: Blank line separating nearby declarations or logic.
  - **L1236 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1237 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1237 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1238 EN**: Opens a new lexical scope or compound statement.
  - **L1238 CN**: 打开一个新的词法作用域或复合语句块。
- **L1239 EN**: Executes a call or declaration centered on `exp`.
  - **L1239 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1240 EN**: Executes a standalone statement or declaration: `term *= x;`.
  - **L1240 CN**: 执行一条独立语句或声明：`term *= x;`。
- **L1241 EN**: Continues logic associated with callable symbol `T`.
  - **L1241 CN**: 继续与可调用符号 `T` 相关的逻辑。
- **L1242 EN**: Executes a standalone statement or declaration: `term /= half;`.
  - **L1242 CN**: 执行一条独立语句或声明：`term /= half;`。
- **L1243 EN**: Executes a standalone statement or declaration: `T sum = term;`.
  - **L1243 CN**: 执行一条独立语句或声明：`T sum = term;`。
- **L1244 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L1244 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1245 EN**: Opens a new lexical scope or compound statement.
  - **L1245 CN**: 打开一个新的词法作用域或复合语句块。
- **L1246 EN**: Executes a standalone statement or declaration: `term /= n - half;`.
  - **L1246 CN**: 执行一条独立语句或声明：`term /= n - half;`。
- **L1247 EN**: Executes a standalone statement or declaration: `term *= x;`.
  - **L1247 CN**: 执行一条独立语句或声明：`term *= x;`。
- **L1248 EN**: Executes a standalone statement or declaration: `sum += term;`.
  - **L1248 CN**: 执行一条独立语句或声明：`sum += term;`。
- **L1249 EN**: Closes the current lexical scope or compound statement.
  - **L1249 CN**: 结束当前词法作用域或复合语句块。
- **L1250 EN**: Executes a standalone statement or declaration: `e += sum;`.
  - **L1250 CN**: 执行一条独立语句或声明：`e += sum;`。
- **L1251 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1251 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1252 EN**: Opens a new lexical scope or compound statement.
  - **L1252 CN**: 打开一个新的词法作用域或复合语句块。
- **L1253 EN**: Comment documents nearby intent or usage notes: `p_derivative = 0;`.
  - **L1253 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = 0;`。
- **L1254 EN**: Closes the current lexical scope or compound statement.
  - **L1254 CN**: 结束当前词法作用域或复合语句块。
- **L1255 EN**: Closes the current lexical scope or compound statement.
  - **L1255 CN**: 结束当前词法作用域或复合语句块。
- **L1256 EN**: Starts the alternative branch of the preceding conditional.
  - **L1256 CN**: 开始前一个条件语句的备选分支。
- **L1257 EN**: Opens a new lexical scope or compound statement.
  - **L1257 CN**: 打开一个新的词法作用域或复合语句块。
- **L1258 EN**: Comment documents nearby intent or usage notes: `We'll be dividing by x later, so calculate derivative * x:`.
  - **L1258 CN**: 注释说明附近代码的意图或使用说明：`We'll be dividing by x later, so calculate derivative * x:`。
- **L1259 EN**: Comment documents nearby intent or usage notes: `p_derivative = sqrt(x) * exp(-x) / constants::root_pi<T>();`.
  - **L1259 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = sqrt(x) * exp(-x) / constants::root_pi<T>();`。
- **L1260 EN**: Closes the current lexical scope or compound statement.
  - **L1260 CN**: 结束当前词法作用域或复合语句块。

### Lines 1261-1288 / 第 1261-1288 行

````cpp
1261:    return e;
1262: }
1263: //
1264: // Asymptotic approximation for large argument, see: https://dlmf.nist.gov/8.11#E2
1265: //
1266: template <class T>
1267: struct incomplete_tgamma_large_x_series
1268: {
1269:    typedef T result_type;
1270:    BOOST_MATH_GPU_ENABLED incomplete_tgamma_large_x_series(const T& a, const T& x)
1271:       : a_poch(a - 1), z(x), term(1) {}
1272:    BOOST_MATH_GPU_ENABLED T operator()()
1273:    {
1274:       T result = term;
1275:       term *= a_poch / z;
1276:       a_poch -= 1;
1277:       return result;
1278:    }
1279:    T a_poch, z, term;
1280: };
1281: 
1282: template <class T, class Policy>
1283: BOOST_MATH_GPU_ENABLED T incomplete_tgamma_large_x(const T& a, const T& x, const Policy& pol)
1284: {
1285:    BOOST_MATH_STD_USING
1286:    incomplete_tgamma_large_x_series<T> s(a, x);
1287:    boost::math::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
1288:    T result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
````
- **L1261 EN**: Returns from the current function with `e`.
  - **L1261 CN**: 以 `e` 从当前函数返回。
- **L1262 EN**: Closes the current lexical scope or compound statement.
  - **L1262 CN**: 结束当前词法作用域或复合语句块。
- **L1263 EN**: Separator comment used for visual grouping.
  - **L1263 CN**: 分隔注释，用于视觉分组。
- **L1264 EN**: Comment documents nearby intent or usage notes: `Asymptotic approximation for large argument, see: https://dlmf.nist.gov/8.11#E2`.
  - **L1264 CN**: 注释说明附近代码的意图或使用说明：`Asymptotic approximation for large argument, see: https://dlmf.nist.gov/8.11#E2`。
- **L1265 EN**: Separator comment used for visual grouping.
  - **L1265 CN**: 分隔注释，用于视觉分组。
- **L1266 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1266 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1267 EN**: Declares struct `incomplete_tgamma_large_x_series`.
  - **L1267 CN**: 声明 struct `incomplete_tgamma_large_x_series`。
- **L1268 EN**: Opens a new lexical scope or compound statement.
  - **L1268 CN**: 打开一个新的词法作用域或复合语句块。
- **L1269 EN**: Introduces a legacy type alias or function typedef: `typedef T result_type;`.
  - **L1269 CN**: 引入传统类型别名或函数 typedef：`typedef T result_type;`。
- **L1270 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1270 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1271 EN**: Continues logic associated with callable symbol `a_poch`.
  - **L1271 CN**: 继续与可调用符号 `a_poch` 相关的逻辑。
- **L1272 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1272 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1273 EN**: Opens a new lexical scope or compound statement.
  - **L1273 CN**: 打开一个新的词法作用域或复合语句块。
- **L1274 EN**: Executes a standalone statement or declaration: `T result = term;`.
  - **L1274 CN**: 执行一条独立语句或声明：`T result = term;`。
- **L1275 EN**: Executes a standalone statement or declaration: `term *= a_poch / z;`.
  - **L1275 CN**: 执行一条独立语句或声明：`term *= a_poch / z;`。
- **L1276 EN**: Executes a standalone statement or declaration: `a_poch -= 1;`.
  - **L1276 CN**: 执行一条独立语句或声明：`a_poch -= 1;`。
- **L1277 EN**: Returns from the current function with `result`.
  - **L1277 CN**: 以 `result` 从当前函数返回。
- **L1278 EN**: Closes the current lexical scope or compound statement.
  - **L1278 CN**: 结束当前词法作用域或复合语句块。
- **L1279 EN**: Executes a standalone statement or declaration: `T a_poch, z, term;`.
  - **L1279 CN**: 执行一条独立语句或声明：`T a_poch, z, term;`。
- **L1280 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1280 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1281 EN**: Blank line separating nearby declarations or logic.
  - **L1281 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1282 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1282 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1283 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1283 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1284 EN**: Opens a new lexical scope or compound statement.
  - **L1284 CN**: 打开一个新的词法作用域或复合语句块。
- **L1285 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1285 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1286 EN**: Executes a call or declaration centered on `s`.
  - **L1286 CN**: 执行以 `s` 为核心的调用或声明。
- **L1287 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1287 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1288 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1288 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1289-1316 / 第 1289-1316 行

````cpp
1289:    boost::math::policies::check_series_iterations<T>("boost::math::tgamma<%1%>(%1%,%1%)", max_iter, pol);
1290:    return result;
1291: }
1292: 
1293: 
1294: //
1295: // Main incomplete gamma entry point, handles all four incomplete gamma's:
1296: //
1297: template <class T, class Policy>
1298: BOOST_MATH_GPU_ENABLED T gamma_incomplete_imp_final(T a, T x, bool normalised, bool invert,
1299:                        const Policy& pol, T* p_derivative)
1300: {
1301:    BOOST_MATH_STD_USING
1302: 
1303:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
1304: 
1305:    T result = 0; // Just to avoid warning C4701: potentially uninitialized local variable 'result' used
1306: 
1307:    BOOST_MATH_ASSERT((p_derivative == nullptr) || normalised);
1308: 
1309:    bool is_int, is_half_int;
1310:    bool is_small_a = (a < 30) && (a <= x + 1) && (x < tools::log_max_value<T>());
1311:    if(is_small_a)
1312:    {
1313:       T fa = floor(a);
1314:       is_int = (fa == a);
1315:       is_half_int = is_int ? false : (fabs(fa - a) == 0.5f);
1316:    }
````
- **L1289 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1289 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1290 EN**: Returns from the current function with `result`.
  - **L1290 CN**: 以 `result` 从当前函数返回。
- **L1291 EN**: Closes the current lexical scope or compound statement.
  - **L1291 CN**: 结束当前词法作用域或复合语句块。
- **L1292 EN**: Blank line separating nearby declarations or logic.
  - **L1292 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1293 EN**: Blank line separating nearby declarations or logic.
  - **L1293 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1294 EN**: Separator comment used for visual grouping.
  - **L1294 CN**: 分隔注释，用于视觉分组。
- **L1295 EN**: Comment documents nearby intent or usage notes: `Main incomplete gamma entry point, handles all four incomplete gamma's:`.
  - **L1295 CN**: 注释说明附近代码的意图或使用说明：`Main incomplete gamma entry point, handles all four incomplete gamma's:`。
- **L1296 EN**: Separator comment used for visual grouping.
  - **L1296 CN**: 分隔注释，用于视觉分组。
- **L1297 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1297 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1298 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1298 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1299 EN**: Continues the surrounding expression or declaration: `const Policy& pol, T* p_derivative)`.
  - **L1299 CN**: 继续构造周围的表达式或声明：`const Policy& pol, T* p_derivative)`。
- **L1300 EN**: Opens a new lexical scope or compound statement.
  - **L1300 CN**: 打开一个新的词法作用域或复合语句块。
- **L1301 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1301 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1302 EN**: Blank line separating nearby declarations or logic.
  - **L1302 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1303 EN**: Introduces a legacy type alias or function typedef: `typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;`.
  - **L1303 CN**: 引入传统类型别名或函数 typedef：`typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;`。
- **L1304 EN**: Blank line separating nearby declarations or logic.
  - **L1304 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1305 EN**: Continues the surrounding expression or declaration: `T result = 0; // Just to avoid warning C4701: potentially uninitialized local variable 'result' used`.
  - **L1305 CN**: 继续构造周围的表达式或声明：`T result = 0; // Just to avoid warning C4701: potentially uninitialized local variable 'result' used`。
- **L1306 EN**: Blank line separating nearby declarations or logic.
  - **L1306 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1307 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1307 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1308 EN**: Blank line separating nearby declarations or logic.
  - **L1308 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1309 EN**: Executes a standalone statement or declaration: `bool is_int, is_half_int;`.
  - **L1309 CN**: 执行一条独立语句或声明：`bool is_int, is_half_int;`。
- **L1310 EN**: Initializes variable `is_small_a` from the right-hand expression.
  - **L1310 CN**: 使用右侧表达式初始化变量 `is_small_a`。
- **L1311 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1311 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1312 EN**: Opens a new lexical scope or compound statement.
  - **L1312 CN**: 打开一个新的词法作用域或复合语句块。
- **L1313 EN**: Executes a call or declaration centered on `floor`.
  - **L1313 CN**: 执行以 `floor` 为核心的调用或声明。
- **L1314 EN**: Executes a call or declaration centered on `=`.
  - **L1314 CN**: 执行以 `=` 为核心的调用或声明。
- **L1315 EN**: Executes a call or declaration centered on `:`.
  - **L1315 CN**: 执行以 `:` 为核心的调用或声明。
- **L1316 EN**: Closes the current lexical scope or compound statement.
  - **L1316 CN**: 结束当前词法作用域或复合语句块。

### Lines 1317-1344 / 第 1317-1344 行

````cpp
1317:    else
1318:    {
1319:       is_int = is_half_int = false;
1320:    }
1321: 
1322:    int eval_method;
1323: 
1324:    if (x == 0)
1325:    {
1326:       eval_method = 2;
1327:    }
1328:    else if(is_int && (x > 0.6))
1329:    {
1330:       // calculate Q via finite sum:
1331:       invert = !invert;
1332:       eval_method = 0;
1333:    }
1334:    else if(is_half_int && (x > 0.2))
1335:    {
1336:       // calculate Q via finite sum for half integer a:
1337:       invert = !invert;
1338:       eval_method = 1;
1339:    }
1340:    else if((x < tools::root_epsilon<T>()) && (a > 1))
1341:    {
1342:       eval_method = 6;
1343:    }
1344:    else if ((x > 1000) && ((a < x) || (fabs(a - 50) / x < 1)))
````
- **L1317 EN**: Starts the alternative branch of the preceding conditional.
  - **L1317 CN**: 开始前一个条件语句的备选分支。
- **L1318 EN**: Opens a new lexical scope or compound statement.
  - **L1318 CN**: 打开一个新的词法作用域或复合语句块。
- **L1319 EN**: Executes a standalone statement or declaration: `is_int = is_half_int = false;`.
  - **L1319 CN**: 执行一条独立语句或声明：`is_int = is_half_int = false;`。
- **L1320 EN**: Closes the current lexical scope or compound statement.
  - **L1320 CN**: 结束当前词法作用域或复合语句块。
- **L1321 EN**: Blank line separating nearby declarations or logic.
  - **L1321 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1322 EN**: Executes a standalone statement or declaration: `int eval_method;`.
  - **L1322 CN**: 执行一条独立语句或声明：`int eval_method;`。
- **L1323 EN**: Blank line separating nearby declarations or logic.
  - **L1323 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1324 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1324 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1325 EN**: Opens a new lexical scope or compound statement.
  - **L1325 CN**: 打开一个新的词法作用域或复合语句块。
- **L1326 EN**: Executes a standalone statement or declaration: `eval_method = 2;`.
  - **L1326 CN**: 执行一条独立语句或声明：`eval_method = 2;`。
- **L1327 EN**: Closes the current lexical scope or compound statement.
  - **L1327 CN**: 结束当前词法作用域或复合语句块。
- **L1328 EN**: Starts the alternative branch of the preceding conditional.
  - **L1328 CN**: 开始前一个条件语句的备选分支。
- **L1329 EN**: Opens a new lexical scope or compound statement.
  - **L1329 CN**: 打开一个新的词法作用域或复合语句块。
- **L1330 EN**: Comment documents nearby intent or usage notes: `calculate Q via finite sum:`.
  - **L1330 CN**: 注释说明附近代码的意图或使用说明：`calculate Q via finite sum:`。
- **L1331 EN**: Executes a standalone statement or declaration: `invert = !invert;`.
  - **L1331 CN**: 执行一条独立语句或声明：`invert = !invert;`。
- **L1332 EN**: Executes a standalone statement or declaration: `eval_method = 0;`.
  - **L1332 CN**: 执行一条独立语句或声明：`eval_method = 0;`。
- **L1333 EN**: Closes the current lexical scope or compound statement.
  - **L1333 CN**: 结束当前词法作用域或复合语句块。
- **L1334 EN**: Starts the alternative branch of the preceding conditional.
  - **L1334 CN**: 开始前一个条件语句的备选分支。
- **L1335 EN**: Opens a new lexical scope or compound statement.
  - **L1335 CN**: 打开一个新的词法作用域或复合语句块。
- **L1336 EN**: Comment documents nearby intent or usage notes: `calculate Q via finite sum for half integer a:`.
  - **L1336 CN**: 注释说明附近代码的意图或使用说明：`calculate Q via finite sum for half integer a:`。
- **L1337 EN**: Executes a standalone statement or declaration: `invert = !invert;`.
  - **L1337 CN**: 执行一条独立语句或声明：`invert = !invert;`。
- **L1338 EN**: Executes a standalone statement or declaration: `eval_method = 1;`.
  - **L1338 CN**: 执行一条独立语句或声明：`eval_method = 1;`。
- **L1339 EN**: Closes the current lexical scope or compound statement.
  - **L1339 CN**: 结束当前词法作用域或复合语句块。
- **L1340 EN**: Starts the alternative branch of the preceding conditional.
  - **L1340 CN**: 开始前一个条件语句的备选分支。
- **L1341 EN**: Opens a new lexical scope or compound statement.
  - **L1341 CN**: 打开一个新的词法作用域或复合语句块。
- **L1342 EN**: Executes a standalone statement or declaration: `eval_method = 6;`.
  - **L1342 CN**: 执行一条独立语句或声明：`eval_method = 6;`。
- **L1343 EN**: Closes the current lexical scope or compound statement.
  - **L1343 CN**: 结束当前词法作用域或复合语句块。
- **L1344 EN**: Starts the alternative branch of the preceding conditional.
  - **L1344 CN**: 开始前一个条件语句的备选分支。

### Lines 1345-1372 / 第 1345-1372 行

````cpp
1345:    {
1346:       // calculate Q via asymptotic approximation:
1347:       invert = !invert;
1348:       eval_method = 7;
1349:    }
1350:    else if(x < T(0.5))
1351:    {
1352:       //
1353:       // Changeover criterion chosen to give a changeover at Q ~ 0.33
1354:       //
1355:       if(T(-0.4) / log(x) < a)
1356:       {
1357:          eval_method = 2;
1358:       }
1359:       else
1360:       {
1361:          eval_method = 3;
1362:       }
1363:    }
1364:    else if(x < T(1.1))
1365:    {
1366:       //
1367:       // Changeover here occurs when P ~ 0.75 or Q ~ 0.25:
1368:       //
1369:       if(x * 0.75f < a)
1370:       {
1371:          eval_method = 2;
1372:       }
````
- **L1345 EN**: Opens a new lexical scope or compound statement.
  - **L1345 CN**: 打开一个新的词法作用域或复合语句块。
- **L1346 EN**: Comment documents nearby intent or usage notes: `calculate Q via asymptotic approximation:`.
  - **L1346 CN**: 注释说明附近代码的意图或使用说明：`calculate Q via asymptotic approximation:`。
- **L1347 EN**: Executes a standalone statement or declaration: `invert = !invert;`.
  - **L1347 CN**: 执行一条独立语句或声明：`invert = !invert;`。
- **L1348 EN**: Executes a standalone statement or declaration: `eval_method = 7;`.
  - **L1348 CN**: 执行一条独立语句或声明：`eval_method = 7;`。
- **L1349 EN**: Closes the current lexical scope or compound statement.
  - **L1349 CN**: 结束当前词法作用域或复合语句块。
- **L1350 EN**: Starts the alternative branch of the preceding conditional.
  - **L1350 CN**: 开始前一个条件语句的备选分支。
- **L1351 EN**: Opens a new lexical scope or compound statement.
  - **L1351 CN**: 打开一个新的词法作用域或复合语句块。
- **L1352 EN**: Separator comment used for visual grouping.
  - **L1352 CN**: 分隔注释，用于视觉分组。
- **L1353 EN**: Comment documents nearby intent or usage notes: `Changeover criterion chosen to give a changeover at Q ~ 0.33`.
  - **L1353 CN**: 注释说明附近代码的意图或使用说明：`Changeover criterion chosen to give a changeover at Q ~ 0.33`。
- **L1354 EN**: Separator comment used for visual grouping.
  - **L1354 CN**: 分隔注释，用于视觉分组。
- **L1355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1356 EN**: Opens a new lexical scope or compound statement.
  - **L1356 CN**: 打开一个新的词法作用域或复合语句块。
- **L1357 EN**: Executes a standalone statement or declaration: `eval_method = 2;`.
  - **L1357 CN**: 执行一条独立语句或声明：`eval_method = 2;`。
- **L1358 EN**: Closes the current lexical scope or compound statement.
  - **L1358 CN**: 结束当前词法作用域或复合语句块。
- **L1359 EN**: Starts the alternative branch of the preceding conditional.
  - **L1359 CN**: 开始前一个条件语句的备选分支。
- **L1360 EN**: Opens a new lexical scope or compound statement.
  - **L1360 CN**: 打开一个新的词法作用域或复合语句块。
- **L1361 EN**: Executes a standalone statement or declaration: `eval_method = 3;`.
  - **L1361 CN**: 执行一条独立语句或声明：`eval_method = 3;`。
- **L1362 EN**: Closes the current lexical scope or compound statement.
  - **L1362 CN**: 结束当前词法作用域或复合语句块。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  - **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Starts the alternative branch of the preceding conditional.
  - **L1364 CN**: 开始前一个条件语句的备选分支。
- **L1365 EN**: Opens a new lexical scope or compound statement.
  - **L1365 CN**: 打开一个新的词法作用域或复合语句块。
- **L1366 EN**: Separator comment used for visual grouping.
  - **L1366 CN**: 分隔注释，用于视觉分组。
- **L1367 EN**: Comment documents nearby intent or usage notes: `Changeover here occurs when P ~ 0.75 or Q ~ 0.25:`.
  - **L1367 CN**: 注释说明附近代码的意图或使用说明：`Changeover here occurs when P ~ 0.75 or Q ~ 0.25:`。
- **L1368 EN**: Separator comment used for visual grouping.
  - **L1368 CN**: 分隔注释，用于视觉分组。
- **L1369 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1369 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1370 EN**: Opens a new lexical scope or compound statement.
  - **L1370 CN**: 打开一个新的词法作用域或复合语句块。
- **L1371 EN**: Executes a standalone statement or declaration: `eval_method = 2;`.
  - **L1371 CN**: 执行一条独立语句或声明：`eval_method = 2;`。
- **L1372 EN**: Closes the current lexical scope or compound statement.
  - **L1372 CN**: 结束当前词法作用域或复合语句块。

### Lines 1373-1400 / 第 1373-1400 行

````cpp
1373:       else
1374:       {
1375:          eval_method = 3;
1376:       }
1377:    }
1378:    else
1379:    {
1380:       //
1381:       // Begin by testing whether we're in the "bad" zone
1382:       // where the result will be near 0.5 and the usual
1383:       // series and continued fractions are slow to converge:
1384:       //
1385:       bool use_temme = false;
1386:       if(normalised && boost::math::numeric_limits<T>::is_specialized && (a > 20))
1387:       {
1388:          T sigma = fabs((x-a)/a);
1389:          if((a > 200) && (policies::digits<T, Policy>() <= 113))
1390:          {
1391:             //
1392:             // This limit is chosen so that we use Temme's expansion
1393:             // only if the result would be larger than about 10^-6.
1394:             // Below that the regular series and continued fractions
1395:             // converge OK, and if we use Temme's method we get increasing
1396:             // errors from the dominant erfc term as it's (inexact) argument
1397:             // increases in magnitude.
1398:             //
1399:             if(20 / a > sigma * sigma)
1400:                use_temme = true;
````
- **L1373 EN**: Starts the alternative branch of the preceding conditional.
  - **L1373 CN**: 开始前一个条件语句的备选分支。
- **L1374 EN**: Opens a new lexical scope or compound statement.
  - **L1374 CN**: 打开一个新的词法作用域或复合语句块。
- **L1375 EN**: Executes a standalone statement or declaration: `eval_method = 3;`.
  - **L1375 CN**: 执行一条独立语句或声明：`eval_method = 3;`。
- **L1376 EN**: Closes the current lexical scope or compound statement.
  - **L1376 CN**: 结束当前词法作用域或复合语句块。
- **L1377 EN**: Closes the current lexical scope or compound statement.
  - **L1377 CN**: 结束当前词法作用域或复合语句块。
- **L1378 EN**: Starts the alternative branch of the preceding conditional.
  - **L1378 CN**: 开始前一个条件语句的备选分支。
- **L1379 EN**: Opens a new lexical scope or compound statement.
  - **L1379 CN**: 打开一个新的词法作用域或复合语句块。
- **L1380 EN**: Separator comment used for visual grouping.
  - **L1380 CN**: 分隔注释，用于视觉分组。
- **L1381 EN**: Comment documents nearby intent or usage notes: `Begin by testing whether we're in the "bad" zone`.
  - **L1381 CN**: 注释说明附近代码的意图或使用说明：`Begin by testing whether we're in the "bad" zone`。
- **L1382 EN**: Comment documents nearby intent or usage notes: `where the result will be near 0.5 and the usual`.
  - **L1382 CN**: 注释说明附近代码的意图或使用说明：`where the result will be near 0.5 and the usual`。
- **L1383 EN**: Comment documents nearby intent or usage notes: `series and continued fractions are slow to converge:`.
  - **L1383 CN**: 注释说明附近代码的意图或使用说明：`series and continued fractions are slow to converge:`。
- **L1384 EN**: Separator comment used for visual grouping.
  - **L1384 CN**: 分隔注释，用于视觉分组。
- **L1385 EN**: Initializes variable `use_temme` from the right-hand expression.
  - **L1385 CN**: 使用右侧表达式初始化变量 `use_temme`。
- **L1386 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1386 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1387 EN**: Opens a new lexical scope or compound statement.
  - **L1387 CN**: 打开一个新的词法作用域或复合语句块。
- **L1388 EN**: Executes a call or declaration centered on `fabs`.
  - **L1388 CN**: 执行以 `fabs` 为核心的调用或声明。
- **L1389 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1389 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1390 EN**: Opens a new lexical scope or compound statement.
  - **L1390 CN**: 打开一个新的词法作用域或复合语句块。
- **L1391 EN**: Separator comment used for visual grouping.
  - **L1391 CN**: 分隔注释，用于视觉分组。
- **L1392 EN**: Comment documents nearby intent or usage notes: `This limit is chosen so that we use Temme's expansion`.
  - **L1392 CN**: 注释说明附近代码的意图或使用说明：`This limit is chosen so that we use Temme's expansion`。
- **L1393 EN**: Comment documents nearby intent or usage notes: `only if the result would be larger than about 10^-6.`.
  - **L1393 CN**: 注释说明附近代码的意图或使用说明：`only if the result would be larger than about 10^-6.`。
- **L1394 EN**: Comment documents nearby intent or usage notes: `Below that the regular series and continued fractions`.
  - **L1394 CN**: 注释说明附近代码的意图或使用说明：`Below that the regular series and continued fractions`。
- **L1395 EN**: Comment documents nearby intent or usage notes: `converge OK, and if we use Temme's method we get increasing`.
  - **L1395 CN**: 注释说明附近代码的意图或使用说明：`converge OK, and if we use Temme's method we get increasing`。
- **L1396 EN**: Comment documents nearby intent or usage notes: `errors from the dominant erfc term as it's (inexact) argument`.
  - **L1396 CN**: 注释说明附近代码的意图或使用说明：`errors from the dominant erfc term as it's (inexact) argument`。
- **L1397 EN**: Comment documents nearby intent or usage notes: `increases in magnitude.`.
  - **L1397 CN**: 注释说明附近代码的意图或使用说明：`increases in magnitude.`。
- **L1398 EN**: Separator comment used for visual grouping.
  - **L1398 CN**: 分隔注释，用于视觉分组。
- **L1399 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1399 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1400 EN**: Executes a standalone statement or declaration: `use_temme = true;`.
  - **L1400 CN**: 执行一条独立语句或声明：`use_temme = true;`。

### Lines 1401-1428 / 第 1401-1428 行

````cpp
1401:          }
1402:          else if(policies::digits<T, Policy>() <= 64)
1403:          {
1404:             // Note in this zone we can't use Temme's expansion for
1405:             // types longer than an 80-bit real:
1406:             // it would require too many terms in the polynomials.
1407:             if(sigma < 0.4)
1408:                use_temme = true;
1409:          }
1410:       }
1411:       if(use_temme)
1412:       {
1413:          eval_method = 5;
1414:       }
1415:       else
1416:       {
1417:          //
1418:          // Regular case where the result will not be too close to 0.5.
1419:          //
1420:          // Changeover here occurs at P ~ Q ~ 0.5
1421:          // Note that series computation of P is about x2 faster than continued fraction
1422:          // calculation of Q, so try and use the CF only when really necessary, especially
1423:          // for small x.
1424:          //
1425:          if(x - (1 / (3 * x)) < a)
1426:          {
1427:             eval_method = 2;
1428:          }
````
- **L1401 EN**: Closes the current lexical scope or compound statement.
  - **L1401 CN**: 结束当前词法作用域或复合语句块。
- **L1402 EN**: Starts the alternative branch of the preceding conditional.
  - **L1402 CN**: 开始前一个条件语句的备选分支。
- **L1403 EN**: Opens a new lexical scope or compound statement.
  - **L1403 CN**: 打开一个新的词法作用域或复合语句块。
- **L1404 EN**: Comment documents nearby intent or usage notes: `Note in this zone we can't use Temme's expansion for`.
  - **L1404 CN**: 注释说明附近代码的意图或使用说明：`Note in this zone we can't use Temme's expansion for`。
- **L1405 EN**: Comment documents nearby intent or usage notes: `types longer than an 80-bit real:`.
  - **L1405 CN**: 注释说明附近代码的意图或使用说明：`types longer than an 80-bit real:`。
- **L1406 EN**: Comment documents nearby intent or usage notes: `it would require too many terms in the polynomials.`.
  - **L1406 CN**: 注释说明附近代码的意图或使用说明：`it would require too many terms in the polynomials.`。
- **L1407 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1407 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1408 EN**: Executes a standalone statement or declaration: `use_temme = true;`.
  - **L1408 CN**: 执行一条独立语句或声明：`use_temme = true;`。
- **L1409 EN**: Closes the current lexical scope or compound statement.
  - **L1409 CN**: 结束当前词法作用域或复合语句块。
- **L1410 EN**: Closes the current lexical scope or compound statement.
  - **L1410 CN**: 结束当前词法作用域或复合语句块。
- **L1411 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1411 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1412 EN**: Opens a new lexical scope or compound statement.
  - **L1412 CN**: 打开一个新的词法作用域或复合语句块。
- **L1413 EN**: Executes a standalone statement or declaration: `eval_method = 5;`.
  - **L1413 CN**: 执行一条独立语句或声明：`eval_method = 5;`。
- **L1414 EN**: Closes the current lexical scope or compound statement.
  - **L1414 CN**: 结束当前词法作用域或复合语句块。
- **L1415 EN**: Starts the alternative branch of the preceding conditional.
  - **L1415 CN**: 开始前一个条件语句的备选分支。
- **L1416 EN**: Opens a new lexical scope or compound statement.
  - **L1416 CN**: 打开一个新的词法作用域或复合语句块。
- **L1417 EN**: Separator comment used for visual grouping.
  - **L1417 CN**: 分隔注释，用于视觉分组。
- **L1418 EN**: Comment documents nearby intent or usage notes: `Regular case where the result will not be too close to 0.5.`.
  - **L1418 CN**: 注释说明附近代码的意图或使用说明：`Regular case where the result will not be too close to 0.5.`。
- **L1419 EN**: Separator comment used for visual grouping.
  - **L1419 CN**: 分隔注释，用于视觉分组。
- **L1420 EN**: Comment documents nearby intent or usage notes: `Changeover here occurs at P ~ Q ~ 0.5`.
  - **L1420 CN**: 注释说明附近代码的意图或使用说明：`Changeover here occurs at P ~ Q ~ 0.5`。
- **L1421 EN**: Comment documents nearby intent or usage notes: `Note that series computation of P is about x2 faster than continued fraction`.
  - **L1421 CN**: 注释说明附近代码的意图或使用说明：`Note that series computation of P is about x2 faster than continued fraction`。
- **L1422 EN**: Comment documents nearby intent or usage notes: `calculation of Q, so try and use the CF only when really necessary, especially`.
  - **L1422 CN**: 注释说明附近代码的意图或使用说明：`calculation of Q, so try and use the CF only when really necessary, especially`。
- **L1423 EN**: Comment documents nearby intent or usage notes: `for small x.`.
  - **L1423 CN**: 注释说明附近代码的意图或使用说明：`for small x.`。
- **L1424 EN**: Separator comment used for visual grouping.
  - **L1424 CN**: 分隔注释，用于视觉分组。
- **L1425 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1425 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1426 EN**: Opens a new lexical scope or compound statement.
  - **L1426 CN**: 打开一个新的词法作用域或复合语句块。
- **L1427 EN**: Executes a standalone statement or declaration: `eval_method = 2;`.
  - **L1427 CN**: 执行一条独立语句或声明：`eval_method = 2;`。
- **L1428 EN**: Closes the current lexical scope or compound statement.
  - **L1428 CN**: 结束当前词法作用域或复合语句块。

### Lines 1429-1456 / 第 1429-1456 行

````cpp
1429:          else
1430:          {
1431:             eval_method = 4;
1432:             invert = !invert;
1433:          }
1434:       }
1435:    }
1436: 
1437:    switch(eval_method)
1438:    {
1439:    case 0:
1440:       {
1441:          result = finite_gamma_q(a, x, pol, p_derivative);
1442:          if(!normalised)
1443:          {
1444:             #ifdef BOOST_MATH_HAS_NVRTC
1445:             if (boost::math::is_same_v<T, float>)
1446:             {
1447:                result *= ::tgammaf(a);
1448:             }
1449:             else
1450:             {
1451:                result *= ::tgamma(a);
1452:             }
1453:             #else
1454:             result *= boost::math::tgamma(a, pol);
1455:             #endif
1456:          }
````
- **L1429 EN**: Starts the alternative branch of the preceding conditional.
  - **L1429 CN**: 开始前一个条件语句的备选分支。
- **L1430 EN**: Opens a new lexical scope or compound statement.
  - **L1430 CN**: 打开一个新的词法作用域或复合语句块。
- **L1431 EN**: Executes a standalone statement or declaration: `eval_method = 4;`.
  - **L1431 CN**: 执行一条独立语句或声明：`eval_method = 4;`。
- **L1432 EN**: Executes a standalone statement or declaration: `invert = !invert;`.
  - **L1432 CN**: 执行一条独立语句或声明：`invert = !invert;`。
- **L1433 EN**: Closes the current lexical scope or compound statement.
  - **L1433 CN**: 结束当前词法作用域或复合语句块。
- **L1434 EN**: Closes the current lexical scope or compound statement.
  - **L1434 CN**: 结束当前词法作用域或复合语句块。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  - **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Blank line separating nearby declarations or logic.
  - **L1436 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1437 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  - **L1437 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L1438 EN**: Opens a new lexical scope or compound statement.
  - **L1438 CN**: 打开一个新的词法作用域或复合语句块。
- **L1439 EN**: Introduces a switch dispatch label: `case 0:`.
  - **L1439 CN**: 引入一个 switch 分发标签：`case 0:`。
- **L1440 EN**: Opens a new lexical scope or compound statement.
  - **L1440 CN**: 打开一个新的词法作用域或复合语句块。
- **L1441 EN**: Executes a call or declaration centered on `finite_gamma_q`.
  - **L1441 CN**: 执行以 `finite_gamma_q` 为核心的调用或声明。
- **L1442 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1442 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1443 EN**: Opens a new lexical scope or compound statement.
  - **L1443 CN**: 打开一个新的词法作用域或复合语句块。
- **L1444 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1444 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1445 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1445 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1446 EN**: Opens a new lexical scope or compound statement.
  - **L1446 CN**: 打开一个新的词法作用域或复合语句块。
- **L1447 EN**: Executes a call or declaration centered on `::tgammaf`.
  - **L1447 CN**: 执行以 `::tgammaf` 为核心的调用或声明。
- **L1448 EN**: Closes the current lexical scope or compound statement.
  - **L1448 CN**: 结束当前词法作用域或复合语句块。
- **L1449 EN**: Starts the alternative branch of the preceding conditional.
  - **L1449 CN**: 开始前一个条件语句的备选分支。
- **L1450 EN**: Opens a new lexical scope or compound statement.
  - **L1450 CN**: 打开一个新的词法作用域或复合语句块。
- **L1451 EN**: Executes a call or declaration centered on `::tgamma`.
  - **L1451 CN**: 执行以 `::tgamma` 为核心的调用或声明。
- **L1452 EN**: Closes the current lexical scope or compound statement.
  - **L1452 CN**: 结束当前词法作用域或复合语句块。
- **L1453 EN**: Continues the current preprocessor branch selection.
  - **L1453 CN**: 继续当前的预处理分支选择。
- **L1454 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1454 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1455 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1455 CN**: 结束当前预处理条件块或头文件保护。
- **L1456 EN**: Closes the current lexical scope or compound statement.
  - **L1456 CN**: 结束当前词法作用域或复合语句块。

### Lines 1457-1484 / 第 1457-1484 行

````cpp
1457:          break;
1458:       }
1459:    case 1:
1460:       {
1461:          result = finite_half_gamma_q(a, x, p_derivative, pol);
1462:          if(!normalised)
1463:          {
1464:             #ifdef BOOST_MATH_HAS_NVRTC
1465:             if (boost::math::is_same_v<T, float>)
1466:             {
1467:                result *= ::tgammaf(a);
1468:             }
1469:             else
1470:             {
1471:                result *= ::tgamma(a);
1472:             }
1473:             #else
1474:             result *= boost::math::tgamma(a, pol);
1475:             #endif
1476:          }
1477:          if(p_derivative && (*p_derivative == 0))
1478:             *p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());
1479:          break;
1480:       }
1481:    case 2:
1482:       {
1483:          // Compute P:
1484:          result = normalised ? regularised_gamma_prefix(a, x, pol, lanczos_type()) : full_igamma_prefix(a, x, pol);
````
- **L1457 EN**: Exits the nearest loop or switch statement.
  - **L1457 CN**: 退出最近的循环或 switch 语句。
- **L1458 EN**: Closes the current lexical scope or compound statement.
  - **L1458 CN**: 结束当前词法作用域或复合语句块。
- **L1459 EN**: Introduces a switch dispatch label: `case 1:`.
  - **L1459 CN**: 引入一个 switch 分发标签：`case 1:`。
- **L1460 EN**: Opens a new lexical scope or compound statement.
  - **L1460 CN**: 打开一个新的词法作用域或复合语句块。
- **L1461 EN**: Executes a call or declaration centered on `finite_half_gamma_q`.
  - **L1461 CN**: 执行以 `finite_half_gamma_q` 为核心的调用或声明。
- **L1462 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1462 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1463 EN**: Opens a new lexical scope or compound statement.
  - **L1463 CN**: 打开一个新的词法作用域或复合语句块。
- **L1464 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1464 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1465 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1465 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1466 EN**: Opens a new lexical scope or compound statement.
  - **L1466 CN**: 打开一个新的词法作用域或复合语句块。
- **L1467 EN**: Executes a call or declaration centered on `::tgammaf`.
  - **L1467 CN**: 执行以 `::tgammaf` 为核心的调用或声明。
- **L1468 EN**: Closes the current lexical scope or compound statement.
  - **L1468 CN**: 结束当前词法作用域或复合语句块。
- **L1469 EN**: Starts the alternative branch of the preceding conditional.
  - **L1469 CN**: 开始前一个条件语句的备选分支。
- **L1470 EN**: Opens a new lexical scope or compound statement.
  - **L1470 CN**: 打开一个新的词法作用域或复合语句块。
- **L1471 EN**: Executes a call or declaration centered on `::tgamma`.
  - **L1471 CN**: 执行以 `::tgamma` 为核心的调用或声明。
- **L1472 EN**: Closes the current lexical scope or compound statement.
  - **L1472 CN**: 结束当前词法作用域或复合语句块。
- **L1473 EN**: Continues the current preprocessor branch selection.
  - **L1473 CN**: 继续当前的预处理分支选择。
- **L1474 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1474 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1475 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1475 CN**: 结束当前预处理条件块或头文件保护。
- **L1476 EN**: Closes the current lexical scope or compound statement.
  - **L1476 CN**: 结束当前词法作用域或复合语句块。
- **L1477 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1477 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1478 EN**: Comment documents nearby intent or usage notes: `p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());`.
  - **L1478 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());`。
- **L1479 EN**: Exits the nearest loop or switch statement.
  - **L1479 CN**: 退出最近的循环或 switch 语句。
- **L1480 EN**: Closes the current lexical scope or compound statement.
  - **L1480 CN**: 结束当前词法作用域或复合语句块。
- **L1481 EN**: Introduces a switch dispatch label: `case 2:`.
  - **L1481 CN**: 引入一个 switch 分发标签：`case 2:`。
- **L1482 EN**: Opens a new lexical scope or compound statement.
  - **L1482 CN**: 打开一个新的词法作用域或复合语句块。
- **L1483 EN**: Comment documents nearby intent or usage notes: `Compute P:`.
  - **L1483 CN**: 注释说明附近代码的意图或使用说明：`Compute P:`。
- **L1484 EN**: Executes a call or declaration centered on `regularised_gamma_prefix`.
  - **L1484 CN**: 执行以 `regularised_gamma_prefix` 为核心的调用或声明。

### Lines 1485-1512 / 第 1485-1512 行

````cpp
1485:          if(p_derivative)
1486:             *p_derivative = result;
1487:          if(result != 0)
1488:          {
1489:             //
1490:             // If we're going to be inverting the result then we can
1491:             // reduce the number of series evaluations by quite
1492:             // a few iterations if we set an initial value for the
1493:             // series sum based on what we'll end up subtracting it from
1494:             // at the end.
1495:             // Have to be careful though that this optimization doesn't
1496:             // lead to spurious numeric overflow.  Note that the
1497:             // scary/expensive overflow checks below are more often
1498:             // than not bypassed in practice for "sensible" input
1499:             // values:
1500:             //
1501:             T init_value = 0;
1502:             bool optimised_invert = false;
1503:             if(invert)
1504:             {
1505:                #ifdef BOOST_MATH_HAS_NVRTC
1506:                if (boost::math::is_same_v<T, float>)
1507:                {
1508:                   init_value = (normalised ? T(1) : ::tgammaf(a));
1509:                }
1510:                else
1511:                {
1512:                   init_value = (normalised ? T(1) : ::tgamma(a));
````
- **L1485 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1485 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1486 EN**: Comment documents nearby intent or usage notes: `p_derivative = result;`.
  - **L1486 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = result;`。
- **L1487 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1487 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1488 EN**: Opens a new lexical scope or compound statement.
  - **L1488 CN**: 打开一个新的词法作用域或复合语句块。
- **L1489 EN**: Separator comment used for visual grouping.
  - **L1489 CN**: 分隔注释，用于视觉分组。
- **L1490 EN**: Comment documents nearby intent or usage notes: `If we're going to be inverting the result then we can`.
  - **L1490 CN**: 注释说明附近代码的意图或使用说明：`If we're going to be inverting the result then we can`。
- **L1491 EN**: Comment documents nearby intent or usage notes: `reduce the number of series evaluations by quite`.
  - **L1491 CN**: 注释说明附近代码的意图或使用说明：`reduce the number of series evaluations by quite`。
- **L1492 EN**: Comment documents nearby intent or usage notes: `a few iterations if we set an initial value for the`.
  - **L1492 CN**: 注释说明附近代码的意图或使用说明：`a few iterations if we set an initial value for the`。
- **L1493 EN**: Comment documents nearby intent or usage notes: `series sum based on what we'll end up subtracting it from`.
  - **L1493 CN**: 注释说明附近代码的意图或使用说明：`series sum based on what we'll end up subtracting it from`。
- **L1494 EN**: Comment documents nearby intent or usage notes: `at the end.`.
  - **L1494 CN**: 注释说明附近代码的意图或使用说明：`at the end.`。
- **L1495 EN**: Comment documents nearby intent or usage notes: `Have to be careful though that this optimization doesn't`.
  - **L1495 CN**: 注释说明附近代码的意图或使用说明：`Have to be careful though that this optimization doesn't`。
- **L1496 EN**: Comment documents nearby intent or usage notes: `lead to spurious numeric overflow.  Note that the`.
  - **L1496 CN**: 注释说明附近代码的意图或使用说明：`lead to spurious numeric overflow.  Note that the`。
- **L1497 EN**: Comment documents nearby intent or usage notes: `scary/expensive overflow checks below are more often`.
  - **L1497 CN**: 注释说明附近代码的意图或使用说明：`scary/expensive overflow checks below are more often`。
- **L1498 EN**: Comment documents nearby intent or usage notes: `than not bypassed in practice for "sensible" input`.
  - **L1498 CN**: 注释说明附近代码的意图或使用说明：`than not bypassed in practice for "sensible" input`。
- **L1499 EN**: Comment documents nearby intent or usage notes: `values:`.
  - **L1499 CN**: 注释说明附近代码的意图或使用说明：`values:`。
- **L1500 EN**: Separator comment used for visual grouping.
  - **L1500 CN**: 分隔注释，用于视觉分组。
- **L1501 EN**: Executes a standalone statement or declaration: `T init_value = 0;`.
  - **L1501 CN**: 执行一条独立语句或声明：`T init_value = 0;`。
- **L1502 EN**: Initializes variable `optimised_invert` from the right-hand expression.
  - **L1502 CN**: 使用右侧表达式初始化变量 `optimised_invert`。
- **L1503 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1503 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1504 EN**: Opens a new lexical scope or compound statement.
  - **L1504 CN**: 打开一个新的词法作用域或复合语句块。
- **L1505 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1505 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1506 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1506 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1507 EN**: Opens a new lexical scope or compound statement.
  - **L1507 CN**: 打开一个新的词法作用域或复合语句块。
- **L1508 EN**: Executes a call or declaration centered on `=`.
  - **L1508 CN**: 执行以 `=` 为核心的调用或声明。
- **L1509 EN**: Closes the current lexical scope or compound statement.
  - **L1509 CN**: 结束当前词法作用域或复合语句块。
- **L1510 EN**: Starts the alternative branch of the preceding conditional.
  - **L1510 CN**: 开始前一个条件语句的备选分支。
- **L1511 EN**: Opens a new lexical scope or compound statement.
  - **L1511 CN**: 打开一个新的词法作用域或复合语句块。
- **L1512 EN**: Executes a call or declaration centered on `=`.
  - **L1512 CN**: 执行以 `=` 为核心的调用或声明。

### Lines 1513-1540 / 第 1513-1540 行

````cpp
1513:                }
1514:                #else
1515:                init_value = (normalised ? T(1) : boost::math::tgamma(a, pol));
1516:                #endif
1517: 
1518:                if(normalised || (result >= 1) || (tools::max_value<T>() * result > init_value))
1519:                {
1520:                   init_value /= result;
1521:                   if(normalised || (a < 1) || (tools::max_value<T>() / a > init_value))
1522:                   {
1523:                      init_value *= -a;
1524:                      optimised_invert = true;
1525:                   }
1526:                   else
1527:                      init_value = 0;  // LCOV_EXCL_LINE  Unreachable for any "sensible" floating point type.
1528:                }
1529:                else
1530:                   init_value = 0;
1531:             }
1532:             result *= detail::lower_gamma_series(a, x, pol, init_value) / a;
1533:             if(optimised_invert)
1534:             {
1535:                invert = false;
1536:                result = -result;
1537:             }
1538:          }
1539:          break;
1540:       }
````
- **L1513 EN**: Closes the current lexical scope or compound statement.
  - **L1513 CN**: 结束当前词法作用域或复合语句块。
- **L1514 EN**: Continues the current preprocessor branch selection.
  - **L1514 CN**: 继续当前的预处理分支选择。
- **L1515 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1515 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1516 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1516 CN**: 结束当前预处理条件块或头文件保护。
- **L1517 EN**: Blank line separating nearby declarations or logic.
  - **L1517 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1518 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1518 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1519 EN**: Opens a new lexical scope or compound statement.
  - **L1519 CN**: 打开一个新的词法作用域或复合语句块。
- **L1520 EN**: Executes a standalone statement or declaration: `init_value /= result;`.
  - **L1520 CN**: 执行一条独立语句或声明：`init_value /= result;`。
- **L1521 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1521 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1522 EN**: Opens a new lexical scope or compound statement.
  - **L1522 CN**: 打开一个新的词法作用域或复合语句块。
- **L1523 EN**: Executes a standalone statement or declaration: `init_value *= -a;`.
  - **L1523 CN**: 执行一条独立语句或声明：`init_value *= -a;`。
- **L1524 EN**: Executes a standalone statement or declaration: `optimised_invert = true;`.
  - **L1524 CN**: 执行一条独立语句或声明：`optimised_invert = true;`。
- **L1525 EN**: Closes the current lexical scope or compound statement.
  - **L1525 CN**: 结束当前词法作用域或复合语句块。
- **L1526 EN**: Starts the alternative branch of the preceding conditional.
  - **L1526 CN**: 开始前一个条件语句的备选分支。
- **L1527 EN**: Continues the surrounding expression or declaration: `init_value = 0;  // LCOV_EXCL_LINE  Unreachable for any "sensible" floating point type.`.
  - **L1527 CN**: 继续构造周围的表达式或声明：`init_value = 0;  // LCOV_EXCL_LINE  Unreachable for any "sensible" floating point type.`。
- **L1528 EN**: Closes the current lexical scope or compound statement.
  - **L1528 CN**: 结束当前词法作用域或复合语句块。
- **L1529 EN**: Starts the alternative branch of the preceding conditional.
  - **L1529 CN**: 开始前一个条件语句的备选分支。
- **L1530 EN**: Executes a standalone statement or declaration: `init_value = 0;`.
  - **L1530 CN**: 执行一条独立语句或声明：`init_value = 0;`。
- **L1531 EN**: Closes the current lexical scope or compound statement.
  - **L1531 CN**: 结束当前词法作用域或复合语句块。
- **L1532 EN**: Executes a call or declaration centered on `detail::lower_gamma_series`.
  - **L1532 CN**: 执行以 `detail::lower_gamma_series` 为核心的调用或声明。
- **L1533 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1533 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1534 EN**: Opens a new lexical scope or compound statement.
  - **L1534 CN**: 打开一个新的词法作用域或复合语句块。
- **L1535 EN**: Executes a standalone statement or declaration: `invert = false;`.
  - **L1535 CN**: 执行一条独立语句或声明：`invert = false;`。
- **L1536 EN**: Executes a standalone statement or declaration: `result = -result;`.
  - **L1536 CN**: 执行一条独立语句或声明：`result = -result;`。
- **L1537 EN**: Closes the current lexical scope or compound statement.
  - **L1537 CN**: 结束当前词法作用域或复合语句块。
- **L1538 EN**: Closes the current lexical scope or compound statement.
  - **L1538 CN**: 结束当前词法作用域或复合语句块。
- **L1539 EN**: Exits the nearest loop or switch statement.
  - **L1539 CN**: 退出最近的循环或 switch 语句。
- **L1540 EN**: Closes the current lexical scope or compound statement.
  - **L1540 CN**: 结束当前词法作用域或复合语句块。

### Lines 1541-1568 / 第 1541-1568 行

````cpp
1541:    case 3:
1542:       {
1543:          // Compute Q:
1544:          invert = !invert;
1545:          T g{};
1546:          result = tgamma_small_upper_part(a, x, pol, &g, invert, p_derivative);
1547:          invert = false;
1548:          if(normalised)
1549:             result /= g;
1550:          break;
1551:       }
1552:    case 4:
1553:       {
1554:          // Compute Q:
1555:          result = normalised ? regularised_gamma_prefix(a, x, pol, lanczos_type()) : full_igamma_prefix(a, x, pol);
1556:          if(p_derivative)
1557:             *p_derivative = result;
1558:          if(result != 0)
1559:             result *= upper_gamma_fraction(a, x, policies::get_epsilon<T, Policy>());
1560:          break;
1561:       }
1562:    case 5:
1563:       {
1564:          //
1565:          // Use compile time dispatch to the appropriate
1566:          // Temme asymptotic expansion.  This may be dead code
1567:          // if T does not have numeric limits support, or has
1568:          // too many digits for the most precise version of
````
- **L1541 EN**: Introduces a switch dispatch label: `case 3:`.
  - **L1541 CN**: 引入一个 switch 分发标签：`case 3:`。
- **L1542 EN**: Opens a new lexical scope or compound statement.
  - **L1542 CN**: 打开一个新的词法作用域或复合语句块。
- **L1543 EN**: Comment documents nearby intent or usage notes: `Compute Q:`.
  - **L1543 CN**: 注释说明附近代码的意图或使用说明：`Compute Q:`。
- **L1544 EN**: Executes a standalone statement or declaration: `invert = !invert;`.
  - **L1544 CN**: 执行一条独立语句或声明：`invert = !invert;`。
- **L1545 EN**: Executes a standalone statement or declaration: `T g{};`.
  - **L1545 CN**: 执行一条独立语句或声明：`T g{};`。
- **L1546 EN**: Executes a call or declaration centered on `tgamma_small_upper_part`.
  - **L1546 CN**: 执行以 `tgamma_small_upper_part` 为核心的调用或声明。
- **L1547 EN**: Executes a standalone statement or declaration: `invert = false;`.
  - **L1547 CN**: 执行一条独立语句或声明：`invert = false;`。
- **L1548 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1548 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1549 EN**: Executes a standalone statement or declaration: `result /= g;`.
  - **L1549 CN**: 执行一条独立语句或声明：`result /= g;`。
- **L1550 EN**: Exits the nearest loop or switch statement.
  - **L1550 CN**: 退出最近的循环或 switch 语句。
- **L1551 EN**: Closes the current lexical scope or compound statement.
  - **L1551 CN**: 结束当前词法作用域或复合语句块。
- **L1552 EN**: Introduces a switch dispatch label: `case 4:`.
  - **L1552 CN**: 引入一个 switch 分发标签：`case 4:`。
- **L1553 EN**: Opens a new lexical scope or compound statement.
  - **L1553 CN**: 打开一个新的词法作用域或复合语句块。
- **L1554 EN**: Comment documents nearby intent or usage notes: `Compute Q:`.
  - **L1554 CN**: 注释说明附近代码的意图或使用说明：`Compute Q:`。
- **L1555 EN**: Executes a call or declaration centered on `regularised_gamma_prefix`.
  - **L1555 CN**: 执行以 `regularised_gamma_prefix` 为核心的调用或声明。
- **L1556 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1556 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1557 EN**: Comment documents nearby intent or usage notes: `p_derivative = result;`.
  - **L1557 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = result;`。
- **L1558 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1558 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1559 EN**: Executes a call or declaration centered on `upper_gamma_fraction`.
  - **L1559 CN**: 执行以 `upper_gamma_fraction` 为核心的调用或声明。
- **L1560 EN**: Exits the nearest loop or switch statement.
  - **L1560 CN**: 退出最近的循环或 switch 语句。
- **L1561 EN**: Closes the current lexical scope or compound statement.
  - **L1561 CN**: 结束当前词法作用域或复合语句块。
- **L1562 EN**: Introduces a switch dispatch label: `case 5:`.
  - **L1562 CN**: 引入一个 switch 分发标签：`case 5:`。
- **L1563 EN**: Opens a new lexical scope or compound statement.
  - **L1563 CN**: 打开一个新的词法作用域或复合语句块。
- **L1564 EN**: Separator comment used for visual grouping.
  - **L1564 CN**: 分隔注释，用于视觉分组。
- **L1565 EN**: Comment documents nearby intent or usage notes: `Use compile time dispatch to the appropriate`.
  - **L1565 CN**: 注释说明附近代码的意图或使用说明：`Use compile time dispatch to the appropriate`。
- **L1566 EN**: Comment documents nearby intent or usage notes: `Temme asymptotic expansion.  This may be dead code`.
  - **L1566 CN**: 注释说明附近代码的意图或使用说明：`Temme asymptotic expansion.  This may be dead code`。
- **L1567 EN**: Comment documents nearby intent or usage notes: `if T does not have numeric limits support, or has`.
  - **L1567 CN**: 注释说明附近代码的意图或使用说明：`if T does not have numeric limits support, or has`。
- **L1568 EN**: Comment documents nearby intent or usage notes: `too many digits for the most precise version of`.
  - **L1568 CN**: 注释说明附近代码的意图或使用说明：`too many digits for the most precise version of`。

### Lines 1569-1596 / 第 1569-1596 行

````cpp
1569:          // these expansions, in that case we'll be calling
1570:          // an empty function.
1571:          //
1572:          typedef typename policies::precision<T, Policy>::type precision_type;
1573: 
1574:          typedef boost::math::integral_constant<int,
1575:             precision_type::value <= 0 ? 0 :
1576:             precision_type::value <= 53 ? 53 :
1577:             precision_type::value <= 64 ? 64 :
1578:             precision_type::value <= 113 ? 113 : 0
1579:          > tag_type;
1580: 
1581:          result = igamma_temme_large(a, x, pol, tag_type());
1582:          if(x >= a)
1583:             invert = !invert;
1584:          if(p_derivative)
1585:             *p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());
1586:          break;
1587:       }
1588:    case 6:
1589:       {
1590:          // x is so small that P is necessarily very small too,
1591:          // use http://functions.wolfram.com/GammaBetaErf/GammaRegularized/06/01/05/01/01/
1592:          if(!normalised)
1593:             result = pow(x, a) / (a);
1594:          else
1595:          {
1596: #ifndef BOOST_MATH_NO_EXCEPTIONS
````
- **L1569 EN**: Comment documents nearby intent or usage notes: `these expansions, in that case we'll be calling`.
  - **L1569 CN**: 注释说明附近代码的意图或使用说明：`these expansions, in that case we'll be calling`。
- **L1570 EN**: Comment documents nearby intent or usage notes: `an empty function.`.
  - **L1570 CN**: 注释说明附近代码的意图或使用说明：`an empty function.`。
- **L1571 EN**: Separator comment used for visual grouping.
  - **L1571 CN**: 分隔注释，用于视觉分组。
- **L1572 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::precision<T, Policy>::type precision_type;`.
  - **L1572 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::precision<T, Policy>::type precision_type;`。
- **L1573 EN**: Blank line separating nearby declarations or logic.
  - **L1573 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1574 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int,`.
  - **L1574 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int,`。
- **L1575 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 0 ? 0 :`.
  - **L1575 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 0 ? 0 :`。
- **L1576 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 53 ? 53 :`.
  - **L1576 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 53 ? 53 :`。
- **L1577 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 64 ? 64 :`.
  - **L1577 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 64 ? 64 :`。
- **L1578 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 113 ? 113 : 0`.
  - **L1578 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 113 ? 113 : 0`。
- **L1579 EN**: Executes a standalone statement or declaration: `> tag_type;`.
  - **L1579 CN**: 执行一条独立语句或声明：`> tag_type;`。
- **L1580 EN**: Blank line separating nearby declarations or logic.
  - **L1580 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1581 EN**: Executes a call or declaration centered on `igamma_temme_large`.
  - **L1581 CN**: 执行以 `igamma_temme_large` 为核心的调用或声明。
- **L1582 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1582 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1583 EN**: Executes a standalone statement or declaration: `invert = !invert;`.
  - **L1583 CN**: 执行一条独立语句或声明：`invert = !invert;`。
- **L1584 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1584 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1585 EN**: Comment documents nearby intent or usage notes: `p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());`.
  - **L1585 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());`。
- **L1586 EN**: Exits the nearest loop or switch statement.
  - **L1586 CN**: 退出最近的循环或 switch 语句。
- **L1587 EN**: Closes the current lexical scope or compound statement.
  - **L1587 CN**: 结束当前词法作用域或复合语句块。
- **L1588 EN**: Introduces a switch dispatch label: `case 6:`.
  - **L1588 CN**: 引入一个 switch 分发标签：`case 6:`。
- **L1589 EN**: Opens a new lexical scope or compound statement.
  - **L1589 CN**: 打开一个新的词法作用域或复合语句块。
- **L1590 EN**: Comment documents nearby intent or usage notes: `x is so small that P is necessarily very small too,`.
  - **L1590 CN**: 注释说明附近代码的意图或使用说明：`x is so small that P is necessarily very small too,`。
- **L1591 EN**: Comment documents nearby intent or usage notes: `use http://functions.wolfram.com/GammaBetaErf/GammaRegularized/06/01/05/01/01/`.
  - **L1591 CN**: 注释说明附近代码的意图或使用说明：`use http://functions.wolfram.com/GammaBetaErf/GammaRegularized/06/01/05/01/01/`。
- **L1592 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1592 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1593 EN**: Executes a call or declaration centered on `pow`.
  - **L1593 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1594 EN**: Starts the alternative branch of the preceding conditional.
  - **L1594 CN**: 开始前一个条件语句的备选分支。
- **L1595 EN**: Opens a new lexical scope or compound statement.
  - **L1595 CN**: 打开一个新的词法作用域或复合语句块。
- **L1596 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_NO_EXCEPTIONS`.
  - **L1596 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_NO_EXCEPTIONS`。

### Lines 1597-1624 / 第 1597-1624 行

````cpp
1597:             try
1598:             {
1599: #endif
1600:                #ifdef BOOST_MATH_HAS_NVRTC
1601:                if (boost::math::is_same_v<T, float>)
1602:                {
1603:                   result = ::powf(x, a) / ::tgammaf(a + 1);
1604:                }
1605:                else
1606:                {
1607:                   result = ::pow(x, a) / ::tgamma(a + 1);
1608:                }
1609:                #else
1610:                result = pow(x, a) / boost::math::tgamma(a + 1, pol);
1611:                #endif
1612: #ifndef BOOST_MATH_NO_EXCEPTIONS
1613:             }
1614:             catch (const std::overflow_error&)
1615:             {
1616:                result = 0;
1617:             }
1618: #endif
1619:          }
1620:          result *= 1 - a * x / (a + 1);
1621:          if (p_derivative)
1622:             *p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());
1623:          break;
1624:       }
````
- **L1597 EN**: Starts an exception-handling region.
  - **L1597 CN**: 开始一个异常处理区域。
- **L1598 EN**: Opens a new lexical scope or compound statement.
  - **L1598 CN**: 打开一个新的词法作用域或复合语句块。
- **L1599 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1599 CN**: 结束当前预处理条件块或头文件保护。
- **L1600 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1600 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1601 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1601 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1602 EN**: Opens a new lexical scope or compound statement.
  - **L1602 CN**: 打开一个新的词法作用域或复合语句块。
- **L1603 EN**: Executes a call or declaration centered on `::powf`.
  - **L1603 CN**: 执行以 `::powf` 为核心的调用或声明。
- **L1604 EN**: Closes the current lexical scope or compound statement.
  - **L1604 CN**: 结束当前词法作用域或复合语句块。
- **L1605 EN**: Starts the alternative branch of the preceding conditional.
  - **L1605 CN**: 开始前一个条件语句的备选分支。
- **L1606 EN**: Opens a new lexical scope or compound statement.
  - **L1606 CN**: 打开一个新的词法作用域或复合语句块。
- **L1607 EN**: Executes a call or declaration centered on `::pow`.
  - **L1607 CN**: 执行以 `::pow` 为核心的调用或声明。
- **L1608 EN**: Closes the current lexical scope or compound statement.
  - **L1608 CN**: 结束当前词法作用域或复合语句块。
- **L1609 EN**: Continues the current preprocessor branch selection.
  - **L1609 CN**: 继续当前的预处理分支选择。
- **L1610 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1610 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1611 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1611 CN**: 结束当前预处理条件块或头文件保护。
- **L1612 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_NO_EXCEPTIONS`.
  - **L1612 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_NO_EXCEPTIONS`。
- **L1613 EN**: Closes the current lexical scope or compound statement.
  - **L1613 CN**: 结束当前词法作用域或复合语句块。
- **L1614 EN**: Starts an exception handler: `catch (const std::overflow_error&)`.
  - **L1614 CN**: 开始一个异常处理器：`catch (const std::overflow_error&)`。
- **L1615 EN**: Opens a new lexical scope or compound statement.
  - **L1615 CN**: 打开一个新的词法作用域或复合语句块。
- **L1616 EN**: Executes a standalone statement or declaration: `result = 0;`.
  - **L1616 CN**: 执行一条独立语句或声明：`result = 0;`。
- **L1617 EN**: Closes the current lexical scope or compound statement.
  - **L1617 CN**: 结束当前词法作用域或复合语句块。
- **L1618 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1618 CN**: 结束当前预处理条件块或头文件保护。
- **L1619 EN**: Closes the current lexical scope or compound statement.
  - **L1619 CN**: 结束当前词法作用域或复合语句块。
- **L1620 EN**: Executes a call or declaration centered on `/`.
  - **L1620 CN**: 执行以 `/` 为核心的调用或声明。
- **L1621 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1621 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1622 EN**: Comment documents nearby intent or usage notes: `p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());`.
  - **L1622 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = regularised_gamma_prefix(a, x, pol, lanczos_type());`。
- **L1623 EN**: Exits the nearest loop or switch statement.
  - **L1623 CN**: 退出最近的循环或 switch 语句。
- **L1624 EN**: Closes the current lexical scope or compound statement.
  - **L1624 CN**: 结束当前词法作用域或复合语句块。

### Lines 1625-1652 / 第 1625-1652 行

````cpp
1625:    case 7:
1626:    {
1627:       // x is large,
1628:       // Compute Q:
1629:       result = normalised ? regularised_gamma_prefix(a, x, pol, lanczos_type()) : full_igamma_prefix(a, x, pol);
1630:       if (p_derivative)
1631:          *p_derivative = result;
1632:       result /= x;
1633:       if (result != 0)
1634:          result *= incomplete_tgamma_large_x(a, x, pol);
1635:       break;
1636:    }
1637:    }
1638: 
1639:    if(normalised && (result > 1))
1640:       result = 1;
1641:    if(invert)
1642:    {
1643:       #ifdef BOOST_MATH_HAS_NVRTC
1644:       T gam;
1645:       if (boost::math::is_same_v<T, float>)
1646:       {
1647:          gam = normalised ? T(1) : ::tgammaf(a);
1648:       }
1649:       else
1650:       {
1651:          gam = normalised ? T(1) : ::tgamma(a);
1652:       }
````
- **L1625 EN**: Introduces a switch dispatch label: `case 7:`.
  - **L1625 CN**: 引入一个 switch 分发标签：`case 7:`。
- **L1626 EN**: Opens a new lexical scope or compound statement.
  - **L1626 CN**: 打开一个新的词法作用域或复合语句块。
- **L1627 EN**: Comment documents nearby intent or usage notes: `x is large,`.
  - **L1627 CN**: 注释说明附近代码的意图或使用说明：`x is large,`。
- **L1628 EN**: Comment documents nearby intent or usage notes: `Compute Q:`.
  - **L1628 CN**: 注释说明附近代码的意图或使用说明：`Compute Q:`。
- **L1629 EN**: Executes a call or declaration centered on `regularised_gamma_prefix`.
  - **L1629 CN**: 执行以 `regularised_gamma_prefix` 为核心的调用或声明。
- **L1630 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1630 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1631 EN**: Comment documents nearby intent or usage notes: `p_derivative = result;`.
  - **L1631 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = result;`。
- **L1632 EN**: Executes a standalone statement or declaration: `result /= x;`.
  - **L1632 CN**: 执行一条独立语句或声明：`result /= x;`。
- **L1633 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1633 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1634 EN**: Executes a call or declaration centered on `incomplete_tgamma_large_x`.
  - **L1634 CN**: 执行以 `incomplete_tgamma_large_x` 为核心的调用或声明。
- **L1635 EN**: Exits the nearest loop or switch statement.
  - **L1635 CN**: 退出最近的循环或 switch 语句。
- **L1636 EN**: Closes the current lexical scope or compound statement.
  - **L1636 CN**: 结束当前词法作用域或复合语句块。
- **L1637 EN**: Closes the current lexical scope or compound statement.
  - **L1637 CN**: 结束当前词法作用域或复合语句块。
- **L1638 EN**: Blank line separating nearby declarations or logic.
  - **L1638 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1639 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1639 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1640 EN**: Executes a standalone statement or declaration: `result = 1;`.
  - **L1640 CN**: 执行一条独立语句或声明：`result = 1;`。
- **L1641 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1641 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1642 EN**: Opens a new lexical scope or compound statement.
  - **L1642 CN**: 打开一个新的词法作用域或复合语句块。
- **L1643 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1643 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1644 EN**: Executes a standalone statement or declaration: `T gam;`.
  - **L1644 CN**: 执行一条独立语句或声明：`T gam;`。
- **L1645 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1645 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1646 EN**: Opens a new lexical scope or compound statement.
  - **L1646 CN**: 打开一个新的词法作用域或复合语句块。
- **L1647 EN**: Executes a call or declaration centered on `T`.
  - **L1647 CN**: 执行以 `T` 为核心的调用或声明。
- **L1648 EN**: Closes the current lexical scope or compound statement.
  - **L1648 CN**: 结束当前词法作用域或复合语句块。
- **L1649 EN**: Starts the alternative branch of the preceding conditional.
  - **L1649 CN**: 开始前一个条件语句的备选分支。
- **L1650 EN**: Opens a new lexical scope or compound statement.
  - **L1650 CN**: 打开一个新的词法作用域或复合语句块。
- **L1651 EN**: Executes a call or declaration centered on `T`.
  - **L1651 CN**: 执行以 `T` 为核心的调用或声明。
- **L1652 EN**: Closes the current lexical scope or compound statement.
  - **L1652 CN**: 结束当前词法作用域或复合语句块。

### Lines 1653-1680 / 第 1653-1680 行

````cpp
1653:       #else
1654:       T gam = normalised ? T(1) : boost::math::tgamma(a, pol);
1655:       #endif
1656:       result = gam - result;
1657:    }
1658:    if(p_derivative)
1659:    {
1660:       if((x == 0) || ((x < 1) && (tools::max_value<T>() * x < *p_derivative)))
1661:       {
1662:          // overflow, just return an arbitrarily large value:
1663:          *p_derivative = tools::max_value<T>() / 2;
1664:       }
1665:       else
1666:          *p_derivative /= x;
1667:    }
1668: 
1669:    return result;
1670: }
1671: 
1672: // Need to implement this dispatch to avoid recursion for device compilers
1673: template <class T, class Policy>
1674: BOOST_MATH_GPU_ENABLED T gamma_incomplete_imp(T a, T x, bool normalised, bool invert,
1675:                        const Policy& pol, T* p_derivative)
1676: {
1677:    constexpr auto function = "boost::math::gamma_p<%1%>(%1%, %1%)";
1678:    if(a <= 0)
1679:       return policies::raise_domain_error<T>(function, "Argument a to the incomplete gamma function must be greater than zero (got a=%1%).", a, pol);
1680:    if(x < 0)
````
- **L1653 EN**: Continues the current preprocessor branch selection.
  - **L1653 CN**: 继续当前的预处理分支选择。
- **L1654 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1654 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1655 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1655 CN**: 结束当前预处理条件块或头文件保护。
- **L1656 EN**: Executes a standalone statement or declaration: `result = gam - result;`.
  - **L1656 CN**: 执行一条独立语句或声明：`result = gam - result;`。
- **L1657 EN**: Closes the current lexical scope or compound statement.
  - **L1657 CN**: 结束当前词法作用域或复合语句块。
- **L1658 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1658 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1659 EN**: Opens a new lexical scope or compound statement.
  - **L1659 CN**: 打开一个新的词法作用域或复合语句块。
- **L1660 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1660 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1661 EN**: Opens a new lexical scope or compound statement.
  - **L1661 CN**: 打开一个新的词法作用域或复合语句块。
- **L1662 EN**: Comment documents nearby intent or usage notes: `overflow, just return an arbitrarily large value:`.
  - **L1662 CN**: 注释说明附近代码的意图或使用说明：`overflow, just return an arbitrarily large value:`。
- **L1663 EN**: Comment documents nearby intent or usage notes: `p_derivative = tools::max_value<T>() / 2;`.
  - **L1663 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = tools::max_value<T>() / 2;`。
- **L1664 EN**: Closes the current lexical scope or compound statement.
  - **L1664 CN**: 结束当前词法作用域或复合语句块。
- **L1665 EN**: Starts the alternative branch of the preceding conditional.
  - **L1665 CN**: 开始前一个条件语句的备选分支。
- **L1666 EN**: Comment documents nearby intent or usage notes: `p_derivative /= x;`.
  - **L1666 CN**: 注释说明附近代码的意图或使用说明：`p_derivative /= x;`。
- **L1667 EN**: Closes the current lexical scope or compound statement.
  - **L1667 CN**: 结束当前词法作用域或复合语句块。
- **L1668 EN**: Blank line separating nearby declarations or logic.
  - **L1668 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1669 EN**: Returns from the current function with `result`.
  - **L1669 CN**: 以 `result` 从当前函数返回。
- **L1670 EN**: Closes the current lexical scope or compound statement.
  - **L1670 CN**: 结束当前词法作用域或复合语句块。
- **L1671 EN**: Blank line separating nearby declarations or logic.
  - **L1671 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1672 EN**: Comment documents nearby intent or usage notes: `Need to implement this dispatch to avoid recursion for device compilers`.
  - **L1672 CN**: 注释说明附近代码的意图或使用说明：`Need to implement this dispatch to avoid recursion for device compilers`。
- **L1673 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1673 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1674 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1674 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1675 EN**: Continues the surrounding expression or declaration: `const Policy& pol, T* p_derivative)`.
  - **L1675 CN**: 继续构造周围的表达式或声明：`const Policy& pol, T* p_derivative)`。
- **L1676 EN**: Opens a new lexical scope or compound statement.
  - **L1676 CN**: 打开一个新的词法作用域或复合语句块。
- **L1677 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1677 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1678 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1678 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1679 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Argument a to the incomplete gamma function must be greater than zero (got a=%1%).", a, pol)`.
  - **L1679 CN**: 以 `policies::raise_domain_error<T>(function, "Argument a to the incomplete gamma function must be greater than zero (got a=%1%).", a, pol)` 从当前函数返回。
- **L1680 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1680 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1681-1708 / 第 1681-1708 行

````cpp
1681:       return policies::raise_domain_error<T>(function, "Argument x to the incomplete gamma function must be >= 0 (got x=%1%).", x, pol);
1682: 
1683:    BOOST_MATH_STD_USING
1684: 
1685: 
1686:    T result = 0; // Just to avoid warning C4701: potentially uninitialized local variable 'result' used
1687: 
1688:    if(x > 0 && a >= max_factorial<T>::value && !normalised)
1689:    {
1690:       //
1691:       // When we're computing the non-normalized incomplete gamma
1692:       // and a is large the result is rather hard to compute unless
1693:       // we use logs.  There are really two options - if x is a long
1694:       // way from a in value then we can reliably use methods 2 and 4
1695:       // below in logarithmic form and go straight to the result.
1696:       // Otherwise we let the regularized gamma take the strain
1697:       // (the result is unlikely to underflow in the central region anyway)
1698:       // and combine with lgamma in the hopes that we get a finite result.
1699:       //
1700:       if(invert && (a * 4 < x))
1701:       {
1702:          // This is method 4 below, done in logs:
1703:          result = a * log(x) - x;
1704:          BOOST_MATH_ASSERT(p_derivative == nullptr);
1705:          // Not currently used for non-normalized igamma:
1706:          //if(p_derivative)
1707:          //   *p_derivative = exp(result);
1708:          result += log(upper_gamma_fraction(a, x, policies::get_epsilon<T, Policy>()));
````
- **L1681 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Argument x to the incomplete gamma function must be >= 0 (got x=%1%).", x, pol)`.
  - **L1681 CN**: 以 `policies::raise_domain_error<T>(function, "Argument x to the incomplete gamma function must be >= 0 (got x=%1%).", x, pol)` 从当前函数返回。
- **L1682 EN**: Blank line separating nearby declarations or logic.
  - **L1682 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1683 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1683 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1684 EN**: Blank line separating nearby declarations or logic.
  - **L1684 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1685 EN**: Blank line separating nearby declarations or logic.
  - **L1685 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1686 EN**: Continues the surrounding expression or declaration: `T result = 0; // Just to avoid warning C4701: potentially uninitialized local variable 'result' used`.
  - **L1686 CN**: 继续构造周围的表达式或声明：`T result = 0; // Just to avoid warning C4701: potentially uninitialized local variable 'result' used`。
- **L1687 EN**: Blank line separating nearby declarations or logic.
  - **L1687 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1688 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1688 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1689 EN**: Opens a new lexical scope or compound statement.
  - **L1689 CN**: 打开一个新的词法作用域或复合语句块。
- **L1690 EN**: Separator comment used for visual grouping.
  - **L1690 CN**: 分隔注释，用于视觉分组。
- **L1691 EN**: Comment documents nearby intent or usage notes: `When we're computing the non-normalized incomplete gamma`.
  - **L1691 CN**: 注释说明附近代码的意图或使用说明：`When we're computing the non-normalized incomplete gamma`。
- **L1692 EN**: Comment documents nearby intent or usage notes: `and a is large the result is rather hard to compute unless`.
  - **L1692 CN**: 注释说明附近代码的意图或使用说明：`and a is large the result is rather hard to compute unless`。
- **L1693 EN**: Comment documents nearby intent or usage notes: `we use logs.  There are really two options - if x is a long`.
  - **L1693 CN**: 注释说明附近代码的意图或使用说明：`we use logs.  There are really two options - if x is a long`。
- **L1694 EN**: Comment documents nearby intent or usage notes: `way from a in value then we can reliably use methods 2 and 4`.
  - **L1694 CN**: 注释说明附近代码的意图或使用说明：`way from a in value then we can reliably use methods 2 and 4`。
- **L1695 EN**: Comment documents nearby intent or usage notes: `below in logarithmic form and go straight to the result.`.
  - **L1695 CN**: 注释说明附近代码的意图或使用说明：`below in logarithmic form and go straight to the result.`。
- **L1696 EN**: Comment documents nearby intent or usage notes: `Otherwise we let the regularized gamma take the strain`.
  - **L1696 CN**: 注释说明附近代码的意图或使用说明：`Otherwise we let the regularized gamma take the strain`。
- **L1697 EN**: Comment documents nearby intent or usage notes: `(the result is unlikely to underflow in the central region anyway)`.
  - **L1697 CN**: 注释说明附近代码的意图或使用说明：`(the result is unlikely to underflow in the central region anyway)`。
- **L1698 EN**: Comment documents nearby intent or usage notes: `and combine with lgamma in the hopes that we get a finite result.`.
  - **L1698 CN**: 注释说明附近代码的意图或使用说明：`and combine with lgamma in the hopes that we get a finite result.`。
- **L1699 EN**: Separator comment used for visual grouping.
  - **L1699 CN**: 分隔注释，用于视觉分组。
- **L1700 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1700 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1701 EN**: Opens a new lexical scope or compound statement.
  - **L1701 CN**: 打开一个新的词法作用域或复合语句块。
- **L1702 EN**: Comment documents nearby intent or usage notes: `This is method 4 below, done in logs:`.
  - **L1702 CN**: 注释说明附近代码的意图或使用说明：`This is method 4 below, done in logs:`。
- **L1703 EN**: Executes a call or declaration centered on `log`.
  - **L1703 CN**: 执行以 `log` 为核心的调用或声明。
- **L1704 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1704 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1705 EN**: Comment documents nearby intent or usage notes: `Not currently used for non-normalized igamma:`.
  - **L1705 CN**: 注释说明附近代码的意图或使用说明：`Not currently used for non-normalized igamma:`。
- **L1706 EN**: Comment documents nearby intent or usage notes: `if(p_derivative)`.
  - **L1706 CN**: 注释说明附近代码的意图或使用说明：`if(p_derivative)`。
- **L1707 EN**: Comment documents nearby intent or usage notes: `p_derivative = exp(result);`.
  - **L1707 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = exp(result);`。
- **L1708 EN**: Executes a call or declaration centered on `log`.
  - **L1708 CN**: 执行以 `log` 为核心的调用或声明。

### Lines 1709-1736 / 第 1709-1736 行

````cpp
1709:       }
1710:       else if(!invert && (a > 4 * x))
1711:       {
1712:          // This is method 2 below, done in logs:
1713:          result = a * log(x) - x;
1714:          BOOST_MATH_ASSERT(p_derivative == nullptr);
1715:          // Not currently used for non-normalized igamma:
1716:          //if(p_derivative)
1717:          //   *p_derivative = exp(result);
1718:          T init_value = 0;
1719:          result += log(detail::lower_gamma_series(a, x, pol, init_value) / a);
1720:       }
1721:       else
1722:       {
1723:          result = gamma_incomplete_imp_final(T(a), T(x), true, invert, pol, p_derivative);
1724:          if(result == 0)
1725:          {
1726:             if(invert)
1727:             {
1728:                // Try http://functions.wolfram.com/06.06.06.0039.01
1729:                result = 1 + 1 / (12 * a) + 1 / (288 * a * a);
1730:                result = log(result) - a + (a - 0.5f) * log(a) + log(boost::math::constants::root_two_pi<T>());
1731:                BOOST_MATH_ASSERT(p_derivative == nullptr);
1732:                // Not currently used for non-normalized igamma:
1733:                //if(p_derivative)
1734:                //   *p_derivative = exp(a * log(x) - x);
1735:             }
1736:             else
````
- **L1709 EN**: Closes the current lexical scope or compound statement.
  - **L1709 CN**: 结束当前词法作用域或复合语句块。
- **L1710 EN**: Starts the alternative branch of the preceding conditional.
  - **L1710 CN**: 开始前一个条件语句的备选分支。
- **L1711 EN**: Opens a new lexical scope or compound statement.
  - **L1711 CN**: 打开一个新的词法作用域或复合语句块。
- **L1712 EN**: Comment documents nearby intent or usage notes: `This is method 2 below, done in logs:`.
  - **L1712 CN**: 注释说明附近代码的意图或使用说明：`This is method 2 below, done in logs:`。
- **L1713 EN**: Executes a call or declaration centered on `log`.
  - **L1713 CN**: 执行以 `log` 为核心的调用或声明。
- **L1714 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1714 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1715 EN**: Comment documents nearby intent or usage notes: `Not currently used for non-normalized igamma:`.
  - **L1715 CN**: 注释说明附近代码的意图或使用说明：`Not currently used for non-normalized igamma:`。
- **L1716 EN**: Comment documents nearby intent or usage notes: `if(p_derivative)`.
  - **L1716 CN**: 注释说明附近代码的意图或使用说明：`if(p_derivative)`。
- **L1717 EN**: Comment documents nearby intent or usage notes: `p_derivative = exp(result);`.
  - **L1717 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = exp(result);`。
- **L1718 EN**: Executes a standalone statement or declaration: `T init_value = 0;`.
  - **L1718 CN**: 执行一条独立语句或声明：`T init_value = 0;`。
- **L1719 EN**: Executes a call or declaration centered on `log`.
  - **L1719 CN**: 执行以 `log` 为核心的调用或声明。
- **L1720 EN**: Closes the current lexical scope or compound statement.
  - **L1720 CN**: 结束当前词法作用域或复合语句块。
- **L1721 EN**: Starts the alternative branch of the preceding conditional.
  - **L1721 CN**: 开始前一个条件语句的备选分支。
- **L1722 EN**: Opens a new lexical scope or compound statement.
  - **L1722 CN**: 打开一个新的词法作用域或复合语句块。
- **L1723 EN**: Executes a call or declaration centered on `gamma_incomplete_imp_final`.
  - **L1723 CN**: 执行以 `gamma_incomplete_imp_final` 为核心的调用或声明。
- **L1724 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1724 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1725 EN**: Opens a new lexical scope or compound statement.
  - **L1725 CN**: 打开一个新的词法作用域或复合语句块。
- **L1726 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1726 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1727 EN**: Opens a new lexical scope or compound statement.
  - **L1727 CN**: 打开一个新的词法作用域或复合语句块。
- **L1728 EN**: Comment documents nearby intent or usage notes: `Try http://functions.wolfram.com/06.06.06.0039.01`.
  - **L1728 CN**: 注释说明附近代码的意图或使用说明：`Try http://functions.wolfram.com/06.06.06.0039.01`。
- **L1729 EN**: Executes a call or declaration centered on `/`.
  - **L1729 CN**: 执行以 `/` 为核心的调用或声明。
- **L1730 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1730 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1731 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1731 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1732 EN**: Comment documents nearby intent or usage notes: `Not currently used for non-normalized igamma:`.
  - **L1732 CN**: 注释说明附近代码的意图或使用说明：`Not currently used for non-normalized igamma:`。
- **L1733 EN**: Comment documents nearby intent or usage notes: `if(p_derivative)`.
  - **L1733 CN**: 注释说明附近代码的意图或使用说明：`if(p_derivative)`。
- **L1734 EN**: Comment documents nearby intent or usage notes: `p_derivative = exp(a * log(x) - x);`.
  - **L1734 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = exp(a * log(x) - x);`。
- **L1735 EN**: Closes the current lexical scope or compound statement.
  - **L1735 CN**: 结束当前词法作用域或复合语句块。
- **L1736 EN**: Starts the alternative branch of the preceding conditional.
  - **L1736 CN**: 开始前一个条件语句的备选分支。

### Lines 1737-1764 / 第 1737-1764 行

````cpp
1737:             {
1738:                // This is method 2 below, done in logs, we're really outside the
1739:                // range of this method, but since the result is almost certainly
1740:                // infinite, we should probably be OK:
1741:                result = a * log(x) - x;
1742:                BOOST_MATH_ASSERT(p_derivative == nullptr);
1743:                // Not currently used for non-normalized igamma:
1744:                //if(p_derivative)
1745:                //   *p_derivative = exp(result);
1746:                T init_value = 0;
1747:                result += log(detail::lower_gamma_series(a, x, pol, init_value) / a);
1748:             }
1749:          }
1750:          else
1751:          {
1752:             #ifdef BOOST_MATH_HAS_NVRTC
1753:             if (boost::math::is_same_v<T, float>)
1754:             {
1755:                result = ::logf(result) + ::lgammaf(a);
1756:             }
1757:             else
1758:             {
1759:                result = ::log(result) + ::lgamma(a);
1760:             }
1761:             #else
1762:             result = log(result) + boost::math::lgamma(a, pol);
1763:             #endif
1764:          }
````
- **L1737 EN**: Opens a new lexical scope or compound statement.
  - **L1737 CN**: 打开一个新的词法作用域或复合语句块。
- **L1738 EN**: Comment documents nearby intent or usage notes: `This is method 2 below, done in logs, we're really outside the`.
  - **L1738 CN**: 注释说明附近代码的意图或使用说明：`This is method 2 below, done in logs, we're really outside the`。
- **L1739 EN**: Comment documents nearby intent or usage notes: `range of this method, but since the result is almost certainly`.
  - **L1739 CN**: 注释说明附近代码的意图或使用说明：`range of this method, but since the result is almost certainly`。
- **L1740 EN**: Comment documents nearby intent or usage notes: `infinite, we should probably be OK:`.
  - **L1740 CN**: 注释说明附近代码的意图或使用说明：`infinite, we should probably be OK:`。
- **L1741 EN**: Executes a call or declaration centered on `log`.
  - **L1741 CN**: 执行以 `log` 为核心的调用或声明。
- **L1742 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1742 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1743 EN**: Comment documents nearby intent or usage notes: `Not currently used for non-normalized igamma:`.
  - **L1743 CN**: 注释说明附近代码的意图或使用说明：`Not currently used for non-normalized igamma:`。
- **L1744 EN**: Comment documents nearby intent or usage notes: `if(p_derivative)`.
  - **L1744 CN**: 注释说明附近代码的意图或使用说明：`if(p_derivative)`。
- **L1745 EN**: Comment documents nearby intent or usage notes: `p_derivative = exp(result);`.
  - **L1745 CN**: 注释说明附近代码的意图或使用说明：`p_derivative = exp(result);`。
- **L1746 EN**: Executes a standalone statement or declaration: `T init_value = 0;`.
  - **L1746 CN**: 执行一条独立语句或声明：`T init_value = 0;`。
- **L1747 EN**: Executes a call or declaration centered on `log`.
  - **L1747 CN**: 执行以 `log` 为核心的调用或声明。
- **L1748 EN**: Closes the current lexical scope or compound statement.
  - **L1748 CN**: 结束当前词法作用域或复合语句块。
- **L1749 EN**: Closes the current lexical scope or compound statement.
  - **L1749 CN**: 结束当前词法作用域或复合语句块。
- **L1750 EN**: Starts the alternative branch of the preceding conditional.
  - **L1750 CN**: 开始前一个条件语句的备选分支。
- **L1751 EN**: Opens a new lexical scope or compound statement.
  - **L1751 CN**: 打开一个新的词法作用域或复合语句块。
- **L1752 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1752 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1753 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1753 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1754 EN**: Opens a new lexical scope or compound statement.
  - **L1754 CN**: 打开一个新的词法作用域或复合语句块。
- **L1755 EN**: Executes a call or declaration centered on `::logf`.
  - **L1755 CN**: 执行以 `::logf` 为核心的调用或声明。
- **L1756 EN**: Closes the current lexical scope or compound statement.
  - **L1756 CN**: 结束当前词法作用域或复合语句块。
- **L1757 EN**: Starts the alternative branch of the preceding conditional.
  - **L1757 CN**: 开始前一个条件语句的备选分支。
- **L1758 EN**: Opens a new lexical scope or compound statement.
  - **L1758 CN**: 打开一个新的词法作用域或复合语句块。
- **L1759 EN**: Executes a call or declaration centered on `::log`.
  - **L1759 CN**: 执行以 `::log` 为核心的调用或声明。
- **L1760 EN**: Closes the current lexical scope or compound statement.
  - **L1760 CN**: 结束当前词法作用域或复合语句块。
- **L1761 EN**: Continues the current preprocessor branch selection.
  - **L1761 CN**: 继续当前的预处理分支选择。
- **L1762 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1762 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1763 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1763 CN**: 结束当前预处理条件块或头文件保护。
- **L1764 EN**: Closes the current lexical scope or compound statement.
  - **L1764 CN**: 结束当前词法作用域或复合语句块。

### Lines 1765-1792 / 第 1765-1792 行

````cpp
1765:       }
1766:       if(result > tools::log_max_value<T>())
1767:          return policies::raise_overflow_error<T>(function, nullptr, pol);
1768:       return exp(result);
1769:    }
1770: 
1771:    // If no special handling is required then we proceeds as normal
1772:    return gamma_incomplete_imp_final(T(a), T(x), normalised, invert, pol, p_derivative);
1773: }
1774: 
1775: //
1776: // Ratios of two gamma functions:
1777: //
1778: template <class T, class Policy, class Lanczos>
1779: BOOST_MATH_GPU_ENABLED T tgamma_delta_ratio_imp_lanczos_final(T z, T delta, const Policy& pol, const Lanczos&)
1780: {
1781:    BOOST_MATH_STD_USING
1782: 
1783:    T zgh = static_cast<T>(z + T(Lanczos::g()) - constants::half<T>());
1784:    T result{};
1785:    if(z + delta == z)
1786:    {
1787:       // Given delta < z * eps
1788:       // and zgh > z
1789:       // Then this must follow:
1790:       BOOST_MATH_ASSERT(fabs(delta / zgh) < boost::math::tools::epsilon<T>());
1791:       // We have:
1792:       // result = exp((constants::half<T>() - z) * boost::math::log1p(delta / zgh, pol));
````
- **L1765 EN**: Closes the current lexical scope or compound statement.
  - **L1765 CN**: 结束当前词法作用域或复合语句块。
- **L1766 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1766 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1767 EN**: Returns from the current function with `policies::raise_overflow_error<T>(function, nullptr, pol)`.
  - **L1767 CN**: 以 `policies::raise_overflow_error<T>(function, nullptr, pol)` 从当前函数返回。
- **L1768 EN**: Returns from the current function with `exp(result)`.
  - **L1768 CN**: 以 `exp(result)` 从当前函数返回。
- **L1769 EN**: Closes the current lexical scope or compound statement.
  - **L1769 CN**: 结束当前词法作用域或复合语句块。
- **L1770 EN**: Blank line separating nearby declarations or logic.
  - **L1770 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1771 EN**: Comment documents nearby intent or usage notes: `If no special handling is required then we proceeds as normal`.
  - **L1771 CN**: 注释说明附近代码的意图或使用说明：`If no special handling is required then we proceeds as normal`。
- **L1772 EN**: Returns from the current function with `gamma_incomplete_imp_final(T(a), T(x), normalised, invert, pol, p_derivative)`.
  - **L1772 CN**: 以 `gamma_incomplete_imp_final(T(a), T(x), normalised, invert, pol, p_derivative)` 从当前函数返回。
- **L1773 EN**: Closes the current lexical scope or compound statement.
  - **L1773 CN**: 结束当前词法作用域或复合语句块。
- **L1774 EN**: Blank line separating nearby declarations or logic.
  - **L1774 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1775 EN**: Separator comment used for visual grouping.
  - **L1775 CN**: 分隔注释，用于视觉分组。
- **L1776 EN**: Comment documents nearby intent or usage notes: `Ratios of two gamma functions:`.
  - **L1776 CN**: 注释说明附近代码的意图或使用说明：`Ratios of two gamma functions:`。
- **L1777 EN**: Separator comment used for visual grouping.
  - **L1777 CN**: 分隔注释，用于视觉分组。
- **L1778 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L1778 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L1779 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1779 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1780 EN**: Opens a new lexical scope or compound statement.
  - **L1780 CN**: 打开一个新的词法作用域或复合语句块。
- **L1781 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1781 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1782 EN**: Blank line separating nearby declarations or logic.
  - **L1782 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1783 EN**: Executes a call or declaration centered on `static_cast<T>`.
  - **L1783 CN**: 执行以 `static_cast<T>` 为核心的调用或声明。
- **L1784 EN**: Executes a standalone statement or declaration: `T result{};`.
  - **L1784 CN**: 执行一条独立语句或声明：`T result{};`。
- **L1785 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1785 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1786 EN**: Opens a new lexical scope or compound statement.
  - **L1786 CN**: 打开一个新的词法作用域或复合语句块。
- **L1787 EN**: Comment documents nearby intent or usage notes: `Given delta < z * eps`.
  - **L1787 CN**: 注释说明附近代码的意图或使用说明：`Given delta < z * eps`。
- **L1788 EN**: Comment documents nearby intent or usage notes: `and zgh > z`.
  - **L1788 CN**: 注释说明附近代码的意图或使用说明：`and zgh > z`。
- **L1789 EN**: Comment documents nearby intent or usage notes: `Then this must follow:`.
  - **L1789 CN**: 注释说明附近代码的意图或使用说明：`Then this must follow:`。
- **L1790 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1790 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1791 EN**: Comment documents nearby intent or usage notes: `We have:`.
  - **L1791 CN**: 注释说明附近代码的意图或使用说明：`We have:`。
- **L1792 EN**: Comment documents nearby intent or usage notes: `result = exp((constants::half<T>() - z) * boost::math::log1p(delta / zgh, pol));`.
  - **L1792 CN**: 注释说明附近代码的意图或使用说明：`result = exp((constants::half<T>() - z) * boost::math::log1p(delta / zgh, pol));`。

### Lines 1793-1820 / 第 1793-1820 行

````cpp
1793:       // 0.5 - z == -z
1794:       // log1p(delta / zgh) = delta / zgh = delta / z
1795:       // multiplying we get -delta.
1796:       result = exp(-delta);
1797:    }
1798:    else
1799:    {
1800:       if(fabs(delta) < 10)
1801:       {
1802:          result = exp((constants::half<T>() - z) * boost::math::log1p(delta / zgh, pol));
1803:       }
1804:       else
1805:       {
1806:          result = pow(T(zgh / (zgh + delta)), T(z - constants::half<T>()));
1807:       }
1808:       // Split the calculation up to avoid spurious overflow:
1809:       result *= Lanczos::lanczos_sum(z) / Lanczos::lanczos_sum(T(z + delta));
1810:    }
1811:    result *= pow(T(constants::e<T>() / (zgh + delta)), delta);
1812:    return result;
1813: }
1814: 
1815: template <class T, class Policy, class Lanczos>
1816: BOOST_MATH_GPU_ENABLED T tgamma_delta_ratio_imp_lanczos(T z, T delta, const Policy& pol, const Lanczos& l)
1817: {
1818:    BOOST_MATH_STD_USING
1819: 
1820:    if(z < tools::epsilon<T>())
````
- **L1793 EN**: Comment documents nearby intent or usage notes: `0.5 - z == -z`.
  - **L1793 CN**: 注释说明附近代码的意图或使用说明：`0.5 - z == -z`。
- **L1794 EN**: Comment documents nearby intent or usage notes: `log1p(delta / zgh) = delta / zgh = delta / z`.
  - **L1794 CN**: 注释说明附近代码的意图或使用说明：`log1p(delta / zgh) = delta / zgh = delta / z`。
- **L1795 EN**: Comment documents nearby intent or usage notes: `multiplying we get -delta.`.
  - **L1795 CN**: 注释说明附近代码的意图或使用说明：`multiplying we get -delta.`。
- **L1796 EN**: Executes a call or declaration centered on `exp`.
  - **L1796 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1797 EN**: Closes the current lexical scope or compound statement.
  - **L1797 CN**: 结束当前词法作用域或复合语句块。
- **L1798 EN**: Starts the alternative branch of the preceding conditional.
  - **L1798 CN**: 开始前一个条件语句的备选分支。
- **L1799 EN**: Opens a new lexical scope or compound statement.
  - **L1799 CN**: 打开一个新的词法作用域或复合语句块。
- **L1800 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1800 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1801 EN**: Opens a new lexical scope or compound statement.
  - **L1801 CN**: 打开一个新的词法作用域或复合语句块。
- **L1802 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1802 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1803 EN**: Closes the current lexical scope or compound statement.
  - **L1803 CN**: 结束当前词法作用域或复合语句块。
- **L1804 EN**: Starts the alternative branch of the preceding conditional.
  - **L1804 CN**: 开始前一个条件语句的备选分支。
- **L1805 EN**: Opens a new lexical scope or compound statement.
  - **L1805 CN**: 打开一个新的词法作用域或复合语句块。
- **L1806 EN**: Executes a call or declaration centered on `pow`.
  - **L1806 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1807 EN**: Closes the current lexical scope or compound statement.
  - **L1807 CN**: 结束当前词法作用域或复合语句块。
- **L1808 EN**: Comment documents nearby intent or usage notes: `Split the calculation up to avoid spurious overflow:`.
  - **L1808 CN**: 注释说明附近代码的意图或使用说明：`Split the calculation up to avoid spurious overflow:`。
- **L1809 EN**: Executes a call or declaration centered on `Lanczos::lanczos_sum`.
  - **L1809 CN**: 执行以 `Lanczos::lanczos_sum` 为核心的调用或声明。
- **L1810 EN**: Closes the current lexical scope or compound statement.
  - **L1810 CN**: 结束当前词法作用域或复合语句块。
- **L1811 EN**: Executes a call or declaration centered on `pow`.
  - **L1811 CN**: 执行以 `pow` 为核心的调用或声明。
- **L1812 EN**: Returns from the current function with `result`.
  - **L1812 CN**: 以 `result` 从当前函数返回。
- **L1813 EN**: Closes the current lexical scope or compound statement.
  - **L1813 CN**: 结束当前词法作用域或复合语句块。
- **L1814 EN**: Blank line separating nearby declarations or logic.
  - **L1814 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1815 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Lanczos>`.
  - **L1815 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Lanczos>`。
- **L1816 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1816 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1817 EN**: Opens a new lexical scope or compound statement.
  - **L1817 CN**: 打开一个新的词法作用域或复合语句块。
- **L1818 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1818 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1819 EN**: Blank line separating nearby declarations or logic.
  - **L1819 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1820 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1820 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1821-1848 / 第 1821-1848 行

````cpp
1821:    {
1822:       //
1823:       // We get spurious numeric overflow unless we're very careful, this
1824:       // can occur either inside Lanczos::lanczos_sum(z) or in the
1825:       // final combination of terms, to avoid this, split the product up
1826:       // into 2 (or 3) parts:
1827:       //
1828:       // G(z) / G(L) = 1 / (z * G(L)) ; z < eps, L = z + delta = delta
1829:       //    z * G(L) = z * G(lim) * (G(L)/G(lim)) ; lim = largest factorial
1830:       //
1831:       if(boost::math::max_factorial<T>::value < delta)
1832:       {
1833:          T ratio = tgamma_delta_ratio_imp_lanczos_final(T(delta), T(boost::math::max_factorial<T>::value - delta), pol, l);
1834:          ratio *= z;
1835:          ratio *= boost::math::unchecked_factorial<T>(boost::math::max_factorial<T>::value - 1);
1836:          return 1 / ratio;
1837:       }
1838:       else
1839:       {
1840:          #ifdef BOOST_MATH_HAS_NVRTC
1841:          if (boost::math::is_same_v<T, float>)
1842:          {
1843:             return 1 / (z * ::tgammaf(z + delta));
1844:          }
1845:          else
1846:          {
1847:             return 1 / (z * ::tgamma(z + delta));
1848:          }
````
- **L1821 EN**: Opens a new lexical scope or compound statement.
  - **L1821 CN**: 打开一个新的词法作用域或复合语句块。
- **L1822 EN**: Separator comment used for visual grouping.
  - **L1822 CN**: 分隔注释，用于视觉分组。
- **L1823 EN**: Comment documents nearby intent or usage notes: `We get spurious numeric overflow unless we're very careful, this`.
  - **L1823 CN**: 注释说明附近代码的意图或使用说明：`We get spurious numeric overflow unless we're very careful, this`。
- **L1824 EN**: Comment documents nearby intent or usage notes: `can occur either inside Lanczos::lanczos_sum(z) or in the`.
  - **L1824 CN**: 注释说明附近代码的意图或使用说明：`can occur either inside Lanczos::lanczos_sum(z) or in the`。
- **L1825 EN**: Comment documents nearby intent or usage notes: `final combination of terms, to avoid this, split the product up`.
  - **L1825 CN**: 注释说明附近代码的意图或使用说明：`final combination of terms, to avoid this, split the product up`。
- **L1826 EN**: Comment documents nearby intent or usage notes: `into 2 (or 3) parts:`.
  - **L1826 CN**: 注释说明附近代码的意图或使用说明：`into 2 (or 3) parts:`。
- **L1827 EN**: Separator comment used for visual grouping.
  - **L1827 CN**: 分隔注释，用于视觉分组。
- **L1828 EN**: Comment documents nearby intent or usage notes: `G(z) / G(L) = 1 / (z * G(L)) ; z < eps, L = z + delta = delta`.
  - **L1828 CN**: 注释说明附近代码的意图或使用说明：`G(z) / G(L) = 1 / (z * G(L)) ; z < eps, L = z + delta = delta`。
- **L1829 EN**: Comment documents nearby intent or usage notes: `z * G(L) = z * G(lim) * (G(L)/G(lim)) ; lim = largest factorial`.
  - **L1829 CN**: 注释说明附近代码的意图或使用说明：`z * G(L) = z * G(lim) * (G(L)/G(lim)) ; lim = largest factorial`。
- **L1830 EN**: Separator comment used for visual grouping.
  - **L1830 CN**: 分隔注释，用于视觉分组。
- **L1831 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1831 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1832 EN**: Opens a new lexical scope or compound statement.
  - **L1832 CN**: 打开一个新的词法作用域或复合语句块。
- **L1833 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1833 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1834 EN**: Executes a standalone statement or declaration: `ratio *= z;`.
  - **L1834 CN**: 执行一条独立语句或声明：`ratio *= z;`。
- **L1835 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1835 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1836 EN**: Returns from the current function with `1 / ratio`.
  - **L1836 CN**: 以 `1 / ratio` 从当前函数返回。
- **L1837 EN**: Closes the current lexical scope or compound statement.
  - **L1837 CN**: 结束当前词法作用域或复合语句块。
- **L1838 EN**: Starts the alternative branch of the preceding conditional.
  - **L1838 CN**: 开始前一个条件语句的备选分支。
- **L1839 EN**: Opens a new lexical scope or compound statement.
  - **L1839 CN**: 打开一个新的词法作用域或复合语句块。
- **L1840 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1840 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1841 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1841 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1842 EN**: Opens a new lexical scope or compound statement.
  - **L1842 CN**: 打开一个新的词法作用域或复合语句块。
- **L1843 EN**: Returns from the current function with `1 / (z * ::tgammaf(z + delta))`.
  - **L1843 CN**: 以 `1 / (z * ::tgammaf(z + delta))` 从当前函数返回。
- **L1844 EN**: Closes the current lexical scope or compound statement.
  - **L1844 CN**: 结束当前词法作用域或复合语句块。
- **L1845 EN**: Starts the alternative branch of the preceding conditional.
  - **L1845 CN**: 开始前一个条件语句的备选分支。
- **L1846 EN**: Opens a new lexical scope or compound statement.
  - **L1846 CN**: 打开一个新的词法作用域或复合语句块。
- **L1847 EN**: Returns from the current function with `1 / (z * ::tgamma(z + delta))`.
  - **L1847 CN**: 以 `1 / (z * ::tgamma(z + delta))` 从当前函数返回。
- **L1848 EN**: Closes the current lexical scope or compound statement.
  - **L1848 CN**: 结束当前词法作用域或复合语句块。

### Lines 1849-1876 / 第 1849-1876 行

````cpp
1849:          #else
1850:          return 1 / (z * boost::math::tgamma(z + delta, pol));
1851:          #endif
1852:       }
1853:    }
1854: 
1855:    return tgamma_delta_ratio_imp_lanczos_final(T(z), T(delta), pol, l);
1856: }
1857: 
1858: //
1859: // And again without Lanczos support this time:
1860: //
1861: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
1862: 
1863: template <class T, class Policy>
1864: T tgamma_delta_ratio_imp_lanczos(T z, T delta, const Policy& pol, const lanczos::undefined_lanczos& l)
1865: {
1866:    BOOST_MATH_STD_USING
1867: 
1868:    //
1869:    // We adjust z and delta so that both z and z+delta are large enough for
1870:    // Sterling's approximation to hold.  We can then calculate the ratio
1871:    // for the adjusted values, and rescale back down to z and z+delta.
1872:    //
1873:    // Get the required shifts first:
1874:    //
1875:    long numerator_shift = 0;
1876:    long denominator_shift = 0;
````
- **L1849 EN**: Continues the current preprocessor branch selection.
  - **L1849 CN**: 继续当前的预处理分支选择。
- **L1850 EN**: Returns from the current function with `1 / (z * boost::math::tgamma(z + delta, pol))`.
  - **L1850 CN**: 以 `1 / (z * boost::math::tgamma(z + delta, pol))` 从当前函数返回。
- **L1851 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1851 CN**: 结束当前预处理条件块或头文件保护。
- **L1852 EN**: Closes the current lexical scope or compound statement.
  - **L1852 CN**: 结束当前词法作用域或复合语句块。
- **L1853 EN**: Closes the current lexical scope or compound statement.
  - **L1853 CN**: 结束当前词法作用域或复合语句块。
- **L1854 EN**: Blank line separating nearby declarations or logic.
  - **L1854 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1855 EN**: Returns from the current function with `tgamma_delta_ratio_imp_lanczos_final(T(z), T(delta), pol, l)`.
  - **L1855 CN**: 以 `tgamma_delta_ratio_imp_lanczos_final(T(z), T(delta), pol, l)` 从当前函数返回。
- **L1856 EN**: Closes the current lexical scope or compound statement.
  - **L1856 CN**: 结束当前词法作用域或复合语句块。
- **L1857 EN**: Blank line separating nearby declarations or logic.
  - **L1857 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1858 EN**: Separator comment used for visual grouping.
  - **L1858 CN**: 分隔注释，用于视觉分组。
- **L1859 EN**: Comment documents nearby intent or usage notes: `And again without Lanczos support this time:`.
  - **L1859 CN**: 注释说明附近代码的意图或使用说明：`And again without Lanczos support this time:`。
- **L1860 EN**: Separator comment used for visual grouping.
  - **L1860 CN**: 分隔注释，用于视觉分组。
- **L1861 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_HAS_GPU_SUPPORT`.
  - **L1861 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_HAS_GPU_SUPPORT`。
- **L1862 EN**: Blank line separating nearby declarations or logic.
  - **L1862 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1863 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1863 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1864 EN**: Continues logic associated with callable symbol `tgamma_delta_ratio_imp_lanczos`.
  - **L1864 CN**: 继续与可调用符号 `tgamma_delta_ratio_imp_lanczos` 相关的逻辑。
- **L1865 EN**: Opens a new lexical scope or compound statement.
  - **L1865 CN**: 打开一个新的词法作用域或复合语句块。
- **L1866 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1866 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1867 EN**: Blank line separating nearby declarations or logic.
  - **L1867 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1868 EN**: Separator comment used for visual grouping.
  - **L1868 CN**: 分隔注释，用于视觉分组。
- **L1869 EN**: Comment documents nearby intent or usage notes: `We adjust z and delta so that both z and z+delta are large enough for`.
  - **L1869 CN**: 注释说明附近代码的意图或使用说明：`We adjust z and delta so that both z and z+delta are large enough for`。
- **L1870 EN**: Comment documents nearby intent or usage notes: `Sterling's approximation to hold.  We can then calculate the ratio`.
  - **L1870 CN**: 注释说明附近代码的意图或使用说明：`Sterling's approximation to hold.  We can then calculate the ratio`。
- **L1871 EN**: Comment documents nearby intent or usage notes: `for the adjusted values, and rescale back down to z and z+delta.`.
  - **L1871 CN**: 注释说明附近代码的意图或使用说明：`for the adjusted values, and rescale back down to z and z+delta.`。
- **L1872 EN**: Separator comment used for visual grouping.
  - **L1872 CN**: 分隔注释，用于视觉分组。
- **L1873 EN**: Comment documents nearby intent or usage notes: `Get the required shifts first:`.
  - **L1873 CN**: 注释说明附近代码的意图或使用说明：`Get the required shifts first:`。
- **L1874 EN**: Separator comment used for visual grouping.
  - **L1874 CN**: 分隔注释，用于视觉分组。
- **L1875 EN**: Initializes variable `numerator_shift` from the right-hand expression.
  - **L1875 CN**: 使用右侧表达式初始化变量 `numerator_shift`。
- **L1876 EN**: Initializes variable `denominator_shift` from the right-hand expression.
  - **L1876 CN**: 使用右侧表达式初始化变量 `denominator_shift`。

### Lines 1877-1904 / 第 1877-1904 行

````cpp
1877:    const int min_z = minimum_argument_for_bernoulli_recursion<T>();
1878: 
1879:    if (min_z > z)
1880:       numerator_shift = 1 + ltrunc(min_z - z);
1881:    if (min_z > z + delta)
1882:       denominator_shift = 1 + ltrunc(min_z - z - delta);
1883:    //
1884:    // If the shifts are zero, then we can just combine scaled tgamma's
1885:    // and combine the remaining terms:
1886:    //
1887:    if (numerator_shift == 0 && denominator_shift == 0)
1888:    {
1889:       T scaled_tgamma_num = scaled_tgamma_no_lanczos(z, pol);
1890:       T scaled_tgamma_denom = scaled_tgamma_no_lanczos(T(z + delta), pol);
1891:       T result = scaled_tgamma_num / scaled_tgamma_denom;
1892:       result *= exp(z * boost::math::log1p(-delta / (z + delta), pol)) * pow(T((delta + z) / constants::e<T>()), -delta);
1893:       return result;
1894:    }
1895:    //
1896:    // We're going to have to rescale first, get the adjusted z and delta values,
1897:    // plus the ratio for the adjusted values:
1898:    //
1899:    T zz = z + numerator_shift;
1900:    T dd = delta - (numerator_shift - denominator_shift);
1901:    T ratio = tgamma_delta_ratio_imp_lanczos(zz, dd, pol, l);
1902:    //
1903:    // Use gamma recurrence relations to get back to the original
1904:    // z and z+delta:
````
- **L1877 EN**: Initializes variable `min_z` from the right-hand expression.
  - **L1877 CN**: 使用右侧表达式初始化变量 `min_z`。
- **L1878 EN**: Blank line separating nearby declarations or logic.
  - **L1878 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1879 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1879 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1880 EN**: Executes a call or declaration centered on `ltrunc`.
  - **L1880 CN**: 执行以 `ltrunc` 为核心的调用或声明。
- **L1881 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1881 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1882 EN**: Executes a call or declaration centered on `ltrunc`.
  - **L1882 CN**: 执行以 `ltrunc` 为核心的调用或声明。
- **L1883 EN**: Separator comment used for visual grouping.
  - **L1883 CN**: 分隔注释，用于视觉分组。
- **L1884 EN**: Comment documents nearby intent or usage notes: `If the shifts are zero, then we can just combine scaled tgamma's`.
  - **L1884 CN**: 注释说明附近代码的意图或使用说明：`If the shifts are zero, then we can just combine scaled tgamma's`。
- **L1885 EN**: Comment documents nearby intent or usage notes: `and combine the remaining terms:`.
  - **L1885 CN**: 注释说明附近代码的意图或使用说明：`and combine the remaining terms:`。
- **L1886 EN**: Separator comment used for visual grouping.
  - **L1886 CN**: 分隔注释，用于视觉分组。
- **L1887 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1887 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1888 EN**: Opens a new lexical scope or compound statement.
  - **L1888 CN**: 打开一个新的词法作用域或复合语句块。
- **L1889 EN**: Executes a call or declaration centered on `scaled_tgamma_no_lanczos`.
  - **L1889 CN**: 执行以 `scaled_tgamma_no_lanczos` 为核心的调用或声明。
- **L1890 EN**: Executes a call or declaration centered on `scaled_tgamma_no_lanczos`.
  - **L1890 CN**: 执行以 `scaled_tgamma_no_lanczos` 为核心的调用或声明。
- **L1891 EN**: Executes a standalone statement or declaration: `T result = scaled_tgamma_num / scaled_tgamma_denom;`.
  - **L1891 CN**: 执行一条独立语句或声明：`T result = scaled_tgamma_num / scaled_tgamma_denom;`。
- **L1892 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1892 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1893 EN**: Returns from the current function with `result`.
  - **L1893 CN**: 以 `result` 从当前函数返回。
- **L1894 EN**: Closes the current lexical scope or compound statement.
  - **L1894 CN**: 结束当前词法作用域或复合语句块。
- **L1895 EN**: Separator comment used for visual grouping.
  - **L1895 CN**: 分隔注释，用于视觉分组。
- **L1896 EN**: Comment documents nearby intent or usage notes: `We're going to have to rescale first, get the adjusted z and delta values,`.
  - **L1896 CN**: 注释说明附近代码的意图或使用说明：`We're going to have to rescale first, get the adjusted z and delta values,`。
- **L1897 EN**: Comment documents nearby intent or usage notes: `plus the ratio for the adjusted values:`.
  - **L1897 CN**: 注释说明附近代码的意图或使用说明：`plus the ratio for the adjusted values:`。
- **L1898 EN**: Separator comment used for visual grouping.
  - **L1898 CN**: 分隔注释，用于视觉分组。
- **L1899 EN**: Executes a standalone statement or declaration: `T zz = z + numerator_shift;`.
  - **L1899 CN**: 执行一条独立语句或声明：`T zz = z + numerator_shift;`。
- **L1900 EN**: Executes a call or declaration centered on `-`.
  - **L1900 CN**: 执行以 `-` 为核心的调用或声明。
- **L1901 EN**: Executes a call or declaration centered on `tgamma_delta_ratio_imp_lanczos`.
  - **L1901 CN**: 执行以 `tgamma_delta_ratio_imp_lanczos` 为核心的调用或声明。
- **L1902 EN**: Separator comment used for visual grouping.
  - **L1902 CN**: 分隔注释，用于视觉分组。
- **L1903 EN**: Comment documents nearby intent or usage notes: `Use gamma recurrence relations to get back to the original`.
  - **L1903 CN**: 注释说明附近代码的意图或使用说明：`Use gamma recurrence relations to get back to the original`。
- **L1904 EN**: Comment documents nearby intent or usage notes: `z and z+delta:`.
  - **L1904 CN**: 注释说明附近代码的意图或使用说明：`z and z+delta:`。

### Lines 1905-1932 / 第 1905-1932 行

````cpp
1905:    //
1906:    for (long long i = 0; i < numerator_shift; ++i)
1907:    {
1908:       ratio /= (z + i);
1909:       if (i < denominator_shift)
1910:          ratio *= (z + delta + i);
1911:    }
1912:    for (long long i = numerator_shift; i < denominator_shift; ++i)
1913:    {
1914:       ratio *= (z + delta + i);
1915:    }
1916:    return ratio;
1917: }
1918: 
1919: #endif
1920: 
1921: template <class T, class Policy>
1922: BOOST_MATH_GPU_ENABLED T tgamma_delta_ratio_imp(T z, T delta, const Policy& pol)
1923: {
1924:    BOOST_MATH_STD_USING
1925: 
1926:    if((z <= 0) || (z + delta <= 0))
1927:    {
1928:       // This isn't very sophisticated, or accurate, but it does work:
1929:       #ifdef BOOST_MATH_HAS_NVRTC
1930:       if (boost::math::is_same_v<T, float>)
1931:       {
1932:          return ::tgammaf(z) / ::tgammaf(z + delta);
````
- **L1905 EN**: Separator comment used for visual grouping.
  - **L1905 CN**: 分隔注释，用于视觉分组。
- **L1906 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L1906 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1907 EN**: Opens a new lexical scope or compound statement.
  - **L1907 CN**: 打开一个新的词法作用域或复合语句块。
- **L1908 EN**: Executes a call or declaration centered on `/=`.
  - **L1908 CN**: 执行以 `/=` 为核心的调用或声明。
- **L1909 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1909 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1910 EN**: Executes a call or declaration centered on `*=`.
  - **L1910 CN**: 执行以 `*=` 为核心的调用或声明。
- **L1911 EN**: Closes the current lexical scope or compound statement.
  - **L1911 CN**: 结束当前词法作用域或复合语句块。
- **L1912 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L1912 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1913 EN**: Opens a new lexical scope or compound statement.
  - **L1913 CN**: 打开一个新的词法作用域或复合语句块。
- **L1914 EN**: Executes a call or declaration centered on `*=`.
  - **L1914 CN**: 执行以 `*=` 为核心的调用或声明。
- **L1915 EN**: Closes the current lexical scope or compound statement.
  - **L1915 CN**: 结束当前词法作用域或复合语句块。
- **L1916 EN**: Returns from the current function with `ratio`.
  - **L1916 CN**: 以 `ratio` 从当前函数返回。
- **L1917 EN**: Closes the current lexical scope or compound statement.
  - **L1917 CN**: 结束当前词法作用域或复合语句块。
- **L1918 EN**: Blank line separating nearby declarations or logic.
  - **L1918 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1919 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1919 CN**: 结束当前预处理条件块或头文件保护。
- **L1920 EN**: Blank line separating nearby declarations or logic.
  - **L1920 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1921 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1921 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1922 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1922 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1923 EN**: Opens a new lexical scope or compound statement.
  - **L1923 CN**: 打开一个新的词法作用域或复合语句块。
- **L1924 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1924 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1925 EN**: Blank line separating nearby declarations or logic.
  - **L1925 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1926 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1926 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1927 EN**: Opens a new lexical scope or compound statement.
  - **L1927 CN**: 打开一个新的词法作用域或复合语句块。
- **L1928 EN**: Comment documents nearby intent or usage notes: `This isn't very sophisticated, or accurate, but it does work:`.
  - **L1928 CN**: 注释说明附近代码的意图或使用说明：`This isn't very sophisticated, or accurate, but it does work:`。
- **L1929 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L1929 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L1930 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1930 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1931 EN**: Opens a new lexical scope or compound statement.
  - **L1931 CN**: 打开一个新的词法作用域或复合语句块。
- **L1932 EN**: Returns from the current function with `::tgammaf(z) / ::tgammaf(z + delta)`.
  - **L1932 CN**: 以 `::tgammaf(z) / ::tgammaf(z + delta)` 从当前函数返回。

### Lines 1933-1960 / 第 1933-1960 行

````cpp
1933:       }
1934:       else
1935:       {
1936:          return ::tgamma(z) / ::tgamma(z + delta);
1937:       }
1938:       #else
1939:       return boost::math::tgamma(z, pol) / boost::math::tgamma(z + delta, pol);
1940:       #endif
1941:    }
1942: 
1943:    if(floor(delta) == delta)
1944:    {
1945:       if(floor(z) == z)
1946:       {
1947:          //
1948:          // Both z and delta are integers, see if we can just use table lookup
1949:          // of the factorials to get the result:
1950:          //
1951:          if((z <= max_factorial<T>::value) && (z + delta <= max_factorial<T>::value))
1952:          {
1953:             return unchecked_factorial<T>((unsigned)itrunc(z, pol) - 1) / unchecked_factorial<T>((unsigned)itrunc(T(z + delta), pol) - 1);
1954:          }
1955:       }
1956:       if(fabs(delta) < 20)
1957:       {
1958:          //
1959:          // delta is a small integer, we can use a finite product:
1960:          //
````
- **L1933 EN**: Closes the current lexical scope or compound statement.
  - **L1933 CN**: 结束当前词法作用域或复合语句块。
- **L1934 EN**: Starts the alternative branch of the preceding conditional.
  - **L1934 CN**: 开始前一个条件语句的备选分支。
- **L1935 EN**: Opens a new lexical scope or compound statement.
  - **L1935 CN**: 打开一个新的词法作用域或复合语句块。
- **L1936 EN**: Returns from the current function with `::tgamma(z) / ::tgamma(z + delta)`.
  - **L1936 CN**: 以 `::tgamma(z) / ::tgamma(z + delta)` 从当前函数返回。
- **L1937 EN**: Closes the current lexical scope or compound statement.
  - **L1937 CN**: 结束当前词法作用域或复合语句块。
- **L1938 EN**: Continues the current preprocessor branch selection.
  - **L1938 CN**: 继续当前的预处理分支选择。
- **L1939 EN**: Returns from the current function with `boost::math::tgamma(z, pol) / boost::math::tgamma(z + delta, pol)`.
  - **L1939 CN**: 以 `boost::math::tgamma(z, pol) / boost::math::tgamma(z + delta, pol)` 从当前函数返回。
- **L1940 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1940 CN**: 结束当前预处理条件块或头文件保护。
- **L1941 EN**: Closes the current lexical scope or compound statement.
  - **L1941 CN**: 结束当前词法作用域或复合语句块。
- **L1942 EN**: Blank line separating nearby declarations or logic.
  - **L1942 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1943 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1943 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1944 EN**: Opens a new lexical scope or compound statement.
  - **L1944 CN**: 打开一个新的词法作用域或复合语句块。
- **L1945 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1945 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1946 EN**: Opens a new lexical scope or compound statement.
  - **L1946 CN**: 打开一个新的词法作用域或复合语句块。
- **L1947 EN**: Separator comment used for visual grouping.
  - **L1947 CN**: 分隔注释，用于视觉分组。
- **L1948 EN**: Comment documents nearby intent or usage notes: `Both z and delta are integers, see if we can just use table lookup`.
  - **L1948 CN**: 注释说明附近代码的意图或使用说明：`Both z and delta are integers, see if we can just use table lookup`。
- **L1949 EN**: Comment documents nearby intent or usage notes: `of the factorials to get the result:`.
  - **L1949 CN**: 注释说明附近代码的意图或使用说明：`of the factorials to get the result:`。
- **L1950 EN**: Separator comment used for visual grouping.
  - **L1950 CN**: 分隔注释，用于视觉分组。
- **L1951 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1951 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1952 EN**: Opens a new lexical scope or compound statement.
  - **L1952 CN**: 打开一个新的词法作用域或复合语句块。
- **L1953 EN**: Returns from the current function with `unchecked_factorial<T>((unsigned)itrunc(z, pol) - 1) / unchecked_factorial<T>((unsigned)itrunc(T(z + delta), pol) - 1)`.
  - **L1953 CN**: 以 `unchecked_factorial<T>((unsigned)itrunc(z, pol) - 1) / unchecked_factorial<T>((unsigned)itrunc(T(z + delta), pol) - 1)` 从当前函数返回。
- **L1954 EN**: Closes the current lexical scope or compound statement.
  - **L1954 CN**: 结束当前词法作用域或复合语句块。
- **L1955 EN**: Closes the current lexical scope or compound statement.
  - **L1955 CN**: 结束当前词法作用域或复合语句块。
- **L1956 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1956 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1957 EN**: Opens a new lexical scope or compound statement.
  - **L1957 CN**: 打开一个新的词法作用域或复合语句块。
- **L1958 EN**: Separator comment used for visual grouping.
  - **L1958 CN**: 分隔注释，用于视觉分组。
- **L1959 EN**: Comment documents nearby intent or usage notes: `delta is a small integer, we can use a finite product:`.
  - **L1959 CN**: 注释说明附近代码的意图或使用说明：`delta is a small integer, we can use a finite product:`。
- **L1960 EN**: Separator comment used for visual grouping.
  - **L1960 CN**: 分隔注释，用于视觉分组。

### Lines 1961-1988 / 第 1961-1988 行

````cpp
1961:          if(delta == 0)
1962:             return 1;
1963:          if(delta < 0)
1964:          {
1965:             z -= 1;
1966:             T result = z;
1967:             while(0 != (delta += 1))
1968:             {
1969:                z -= 1;
1970:                result *= z;
1971:             }
1972:             return result;
1973:          }
1974:          else
1975:          {
1976:             T result = 1 / z;
1977:             while(0 != (delta -= 1))
1978:             {
1979:                z += 1;
1980:                result /= z;
1981:             }
1982:             return result;
1983:          }
1984:       }
1985:    }
1986:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
1987:    return tgamma_delta_ratio_imp_lanczos(z, delta, pol, lanczos_type());
1988: }
````
- **L1961 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1961 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1962 EN**: Returns from the current function with `1`.
  - **L1962 CN**: 以 `1` 从当前函数返回。
- **L1963 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1963 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1964 EN**: Opens a new lexical scope or compound statement.
  - **L1964 CN**: 打开一个新的词法作用域或复合语句块。
- **L1965 EN**: Executes a standalone statement or declaration: `z -= 1;`.
  - **L1965 CN**: 执行一条独立语句或声明：`z -= 1;`。
- **L1966 EN**: Executes a standalone statement or declaration: `T result = z;`.
  - **L1966 CN**: 执行一条独立语句或声明：`T result = z;`。
- **L1967 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L1967 CN**: 开始 `while` 控制流语句并计算其条件。
- **L1968 EN**: Opens a new lexical scope or compound statement.
  - **L1968 CN**: 打开一个新的词法作用域或复合语句块。
- **L1969 EN**: Executes a standalone statement or declaration: `z -= 1;`.
  - **L1969 CN**: 执行一条独立语句或声明：`z -= 1;`。
- **L1970 EN**: Executes a standalone statement or declaration: `result *= z;`.
  - **L1970 CN**: 执行一条独立语句或声明：`result *= z;`。
- **L1971 EN**: Closes the current lexical scope or compound statement.
  - **L1971 CN**: 结束当前词法作用域或复合语句块。
- **L1972 EN**: Returns from the current function with `result`.
  - **L1972 CN**: 以 `result` 从当前函数返回。
- **L1973 EN**: Closes the current lexical scope or compound statement.
  - **L1973 CN**: 结束当前词法作用域或复合语句块。
- **L1974 EN**: Starts the alternative branch of the preceding conditional.
  - **L1974 CN**: 开始前一个条件语句的备选分支。
- **L1975 EN**: Opens a new lexical scope or compound statement.
  - **L1975 CN**: 打开一个新的词法作用域或复合语句块。
- **L1976 EN**: Executes a standalone statement or declaration: `T result = 1 / z;`.
  - **L1976 CN**: 执行一条独立语句或声明：`T result = 1 / z;`。
- **L1977 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L1977 CN**: 开始 `while` 控制流语句并计算其条件。
- **L1978 EN**: Opens a new lexical scope or compound statement.
  - **L1978 CN**: 打开一个新的词法作用域或复合语句块。
- **L1979 EN**: Executes a standalone statement or declaration: `z += 1;`.
  - **L1979 CN**: 执行一条独立语句或声明：`z += 1;`。
- **L1980 EN**: Executes a standalone statement or declaration: `result /= z;`.
  - **L1980 CN**: 执行一条独立语句或声明：`result /= z;`。
- **L1981 EN**: Closes the current lexical scope or compound statement.
  - **L1981 CN**: 结束当前词法作用域或复合语句块。
- **L1982 EN**: Returns from the current function with `result`.
  - **L1982 CN**: 以 `result` 从当前函数返回。
- **L1983 EN**: Closes the current lexical scope or compound statement.
  - **L1983 CN**: 结束当前词法作用域或复合语句块。
- **L1984 EN**: Closes the current lexical scope or compound statement.
  - **L1984 CN**: 结束当前词法作用域或复合语句块。
- **L1985 EN**: Closes the current lexical scope or compound statement.
  - **L1985 CN**: 结束当前词法作用域或复合语句块。
- **L1986 EN**: Introduces a legacy type alias or function typedef: `typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;`.
  - **L1986 CN**: 引入传统类型别名或函数 typedef：`typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;`。
- **L1987 EN**: Returns from the current function with `tgamma_delta_ratio_imp_lanczos(z, delta, pol, lanczos_type())`.
  - **L1987 CN**: 以 `tgamma_delta_ratio_imp_lanczos(z, delta, pol, lanczos_type())` 从当前函数返回。
- **L1988 EN**: Closes the current lexical scope or compound statement.
  - **L1988 CN**: 结束当前词法作用域或复合语句块。

### Lines 1989-2016 / 第 1989-2016 行

````cpp
1989: 
1990: template <class T, class Policy>
1991: BOOST_MATH_GPU_ENABLED T tgamma_ratio_imp(T x, T y, const Policy& pol)
1992: {
1993:    BOOST_MATH_STD_USING
1994: 
1995:    if((x <= 0) || (boost::math::isinf)(x))
1996:       return policies::raise_domain_error<T>("boost::math::tgamma_ratio<%1%>(%1%, %1%)", "Gamma function ratios only implemented for positive arguments (got a=%1%).", x, pol);
1997:    if((y <= 0) || (boost::math::isinf)(y))
1998:       return policies::raise_domain_error<T>("boost::math::tgamma_ratio<%1%>(%1%, %1%)", "Gamma function ratios only implemented for positive arguments (got b=%1%).", y, pol);
1999: 
2000:    // We don't need to worry about the denorm case on device
2001:    // And this has the added bonus of removing recursion
2002:    #ifndef BOOST_MATH_HAS_GPU_SUPPORT
2003:    if(x <= tools::min_value<T>())
2004:    {
2005:       // Special case for denorms...Ugh.
2006:       T shift = ldexp(T(1), tools::digits<T>());
2007:       return shift * tgamma_ratio_imp(T(x * shift), y, pol);
2008:    }
2009:    #endif
2010: 
2011:    if((x < max_factorial<T>::value) && (y < max_factorial<T>::value))
2012:    {
2013:       // Rather than subtracting values, lets just call the gamma functions directly:
2014:       #ifdef BOOST_MATH_HAS_NVRTC
2015:       if (boost::math::is_same_v<T, float>)
2016:       {
````
- **L1989 EN**: Blank line separating nearby declarations or logic.
  - **L1989 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1990 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1990 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1991 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1991 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1992 EN**: Opens a new lexical scope or compound statement.
  - **L1992 CN**: 打开一个新的词法作用域或复合语句块。
- **L1993 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1993 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1994 EN**: Blank line separating nearby declarations or logic.
  - **L1994 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1995 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1995 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1996 EN**: Returns from the current function with `policies::raise_domain_error<T>("boost::math::tgamma_ratio<%1%>(%1%, %1%)", "Gamma function ratios only implemented for positive arguments (got a=%1%).", x, pol)`.
  - **L1996 CN**: 以 `policies::raise_domain_error<T>("boost::math::tgamma_ratio<%1%>(%1%, %1%)", "Gamma function ratios only implemented for positive arguments (got a=%1%).", x, pol)` 从当前函数返回。
- **L1997 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1997 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1998 EN**: Returns from the current function with `policies::raise_domain_error<T>("boost::math::tgamma_ratio<%1%>(%1%, %1%)", "Gamma function ratios only implemented for positive arguments (got b=%1%).", y, pol)`.
  - **L1998 CN**: 以 `policies::raise_domain_error<T>("boost::math::tgamma_ratio<%1%>(%1%, %1%)", "Gamma function ratios only implemented for positive arguments (got b=%1%).", y, pol)` 从当前函数返回。
- **L1999 EN**: Blank line separating nearby declarations or logic.
  - **L1999 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2000 EN**: Comment documents nearby intent or usage notes: `We don't need to worry about the denorm case on device`.
  - **L2000 CN**: 注释说明附近代码的意图或使用说明：`We don't need to worry about the denorm case on device`。
- **L2001 EN**: Comment documents nearby intent or usage notes: `And this has the added bonus of removing recursion`.
  - **L2001 CN**: 注释说明附近代码的意图或使用说明：`And this has the added bonus of removing recursion`。
- **L2002 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_HAS_GPU_SUPPORT`.
  - **L2002 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_HAS_GPU_SUPPORT`。
- **L2003 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2003 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2004 EN**: Opens a new lexical scope or compound statement.
  - **L2004 CN**: 打开一个新的词法作用域或复合语句块。
- **L2005 EN**: Comment documents nearby intent or usage notes: `Special case for denorms...Ugh.`.
  - **L2005 CN**: 注释说明附近代码的意图或使用说明：`Special case for denorms...Ugh.`。
- **L2006 EN**: Executes a call or declaration centered on `ldexp`.
  - **L2006 CN**: 执行以 `ldexp` 为核心的调用或声明。
- **L2007 EN**: Returns from the current function with `shift * tgamma_ratio_imp(T(x * shift), y, pol)`.
  - **L2007 CN**: 以 `shift * tgamma_ratio_imp(T(x * shift), y, pol)` 从当前函数返回。
- **L2008 EN**: Closes the current lexical scope or compound statement.
  - **L2008 CN**: 结束当前词法作用域或复合语句块。
- **L2009 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2009 CN**: 结束当前预处理条件块或头文件保护。
- **L2010 EN**: Blank line separating nearby declarations or logic.
  - **L2010 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2011 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2011 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2012 EN**: Opens a new lexical scope or compound statement.
  - **L2012 CN**: 打开一个新的词法作用域或复合语句块。
- **L2013 EN**: Comment documents nearby intent or usage notes: `Rather than subtracting values, lets just call the gamma functions directly:`.
  - **L2013 CN**: 注释说明附近代码的意图或使用说明：`Rather than subtracting values, lets just call the gamma functions directly:`。
- **L2014 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L2014 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L2015 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2015 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2016 EN**: Opens a new lexical scope or compound statement.
  - **L2016 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 2017-2044 / 第 2017-2044 行

````cpp
2017:          return ::tgammaf(x) / ::tgammaf(y);
2018:       }
2019:       else
2020:       {
2021:          return ::tgamma(x) / ::tgamma(y);
2022:       }
2023:       #else
2024:       return boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol);
2025:       #endif
2026:    }
2027:    T prefix = 1;
2028:    if(x < 1)
2029:    {
2030:       if(y < 2 * max_factorial<T>::value)
2031:       {
2032:          // We need to sidestep on x as well, otherwise we'll underflow
2033:          // before we get to factor in the prefix term:
2034:          prefix /= x;
2035:          x += 1;
2036:          while(y >=  max_factorial<T>::value)
2037:          {
2038:             y -= 1;
2039:             prefix /= y;
2040:          }
2041: 
2042:          #ifdef BOOST_MATH_HAS_NVRTC
2043:          if (boost::math::is_same_v<T, float>)
2044:          {
````
- **L2017 EN**: Returns from the current function with `::tgammaf(x) / ::tgammaf(y)`.
  - **L2017 CN**: 以 `::tgammaf(x) / ::tgammaf(y)` 从当前函数返回。
- **L2018 EN**: Closes the current lexical scope or compound statement.
  - **L2018 CN**: 结束当前词法作用域或复合语句块。
- **L2019 EN**: Starts the alternative branch of the preceding conditional.
  - **L2019 CN**: 开始前一个条件语句的备选分支。
- **L2020 EN**: Opens a new lexical scope or compound statement.
  - **L2020 CN**: 打开一个新的词法作用域或复合语句块。
- **L2021 EN**: Returns from the current function with `::tgamma(x) / ::tgamma(y)`.
  - **L2021 CN**: 以 `::tgamma(x) / ::tgamma(y)` 从当前函数返回。
- **L2022 EN**: Closes the current lexical scope or compound statement.
  - **L2022 CN**: 结束当前词法作用域或复合语句块。
- **L2023 EN**: Continues the current preprocessor branch selection.
  - **L2023 CN**: 继续当前的预处理分支选择。
- **L2024 EN**: Returns from the current function with `boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol)`.
  - **L2024 CN**: 以 `boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol)` 从当前函数返回。
- **L2025 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2025 CN**: 结束当前预处理条件块或头文件保护。
- **L2026 EN**: Closes the current lexical scope or compound statement.
  - **L2026 CN**: 结束当前词法作用域或复合语句块。
- **L2027 EN**: Executes a standalone statement or declaration: `T prefix = 1;`.
  - **L2027 CN**: 执行一条独立语句或声明：`T prefix = 1;`。
- **L2028 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2028 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2029 EN**: Opens a new lexical scope or compound statement.
  - **L2029 CN**: 打开一个新的词法作用域或复合语句块。
- **L2030 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2030 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2031 EN**: Opens a new lexical scope or compound statement.
  - **L2031 CN**: 打开一个新的词法作用域或复合语句块。
- **L2032 EN**: Comment documents nearby intent or usage notes: `We need to sidestep on x as well, otherwise we'll underflow`.
  - **L2032 CN**: 注释说明附近代码的意图或使用说明：`We need to sidestep on x as well, otherwise we'll underflow`。
- **L2033 EN**: Comment documents nearby intent or usage notes: `before we get to factor in the prefix term:`.
  - **L2033 CN**: 注释说明附近代码的意图或使用说明：`before we get to factor in the prefix term:`。
- **L2034 EN**: Executes a standalone statement or declaration: `prefix /= x;`.
  - **L2034 CN**: 执行一条独立语句或声明：`prefix /= x;`。
- **L2035 EN**: Executes a standalone statement or declaration: `x += 1;`.
  - **L2035 CN**: 执行一条独立语句或声明：`x += 1;`。
- **L2036 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L2036 CN**: 开始 `while` 控制流语句并计算其条件。
- **L2037 EN**: Opens a new lexical scope or compound statement.
  - **L2037 CN**: 打开一个新的词法作用域或复合语句块。
- **L2038 EN**: Executes a standalone statement or declaration: `y -= 1;`.
  - **L2038 CN**: 执行一条独立语句或声明：`y -= 1;`。
- **L2039 EN**: Executes a standalone statement or declaration: `prefix /= y;`.
  - **L2039 CN**: 执行一条独立语句或声明：`prefix /= y;`。
- **L2040 EN**: Closes the current lexical scope or compound statement.
  - **L2040 CN**: 结束当前词法作用域或复合语句块。
- **L2041 EN**: Blank line separating nearby declarations or logic.
  - **L2041 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2042 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L2042 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L2043 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2043 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2044 EN**: Opens a new lexical scope or compound statement.
  - **L2044 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 2045-2072 / 第 2045-2072 行

````cpp
2045:             return prefix * ::tgammaf(x) / ::tgammaf(y);
2046:          }
2047:          else
2048:          {
2049:             return prefix * ::tgamma(x) / ::tgamma(y);
2050:          }
2051:          #else
2052:          return prefix * boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol);
2053:          #endif
2054:       }
2055:       //
2056:       // result is almost certainly going to underflow to zero, try logs just in case:
2057:       //
2058:       #ifdef BOOST_MATH_HAS_NVRTC
2059:       if (boost::math::is_same_v<T, float>)
2060:       {
2061:          return ::expf(::lgammaf(x) - ::lgammaf(y));
2062:       }
2063:       else
2064:       {
2065:          return ::exp(::lgamma(x) - ::lgamma(y));
2066:       }
2067:       #else
2068:       return exp(boost::math::lgamma(x, pol) - boost::math::lgamma(y, pol));
2069:       #endif
2070:    }
2071:    if(y < 1)
2072:    {
````
- **L2045 EN**: Returns from the current function with `prefix * ::tgammaf(x) / ::tgammaf(y)`.
  - **L2045 CN**: 以 `prefix * ::tgammaf(x) / ::tgammaf(y)` 从当前函数返回。
- **L2046 EN**: Closes the current lexical scope or compound statement.
  - **L2046 CN**: 结束当前词法作用域或复合语句块。
- **L2047 EN**: Starts the alternative branch of the preceding conditional.
  - **L2047 CN**: 开始前一个条件语句的备选分支。
- **L2048 EN**: Opens a new lexical scope or compound statement.
  - **L2048 CN**: 打开一个新的词法作用域或复合语句块。
- **L2049 EN**: Returns from the current function with `prefix * ::tgamma(x) / ::tgamma(y)`.
  - **L2049 CN**: 以 `prefix * ::tgamma(x) / ::tgamma(y)` 从当前函数返回。
- **L2050 EN**: Closes the current lexical scope or compound statement.
  - **L2050 CN**: 结束当前词法作用域或复合语句块。
- **L2051 EN**: Continues the current preprocessor branch selection.
  - **L2051 CN**: 继续当前的预处理分支选择。
- **L2052 EN**: Returns from the current function with `prefix * boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol)`.
  - **L2052 CN**: 以 `prefix * boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol)` 从当前函数返回。
- **L2053 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2053 CN**: 结束当前预处理条件块或头文件保护。
- **L2054 EN**: Closes the current lexical scope or compound statement.
  - **L2054 CN**: 结束当前词法作用域或复合语句块。
- **L2055 EN**: Separator comment used for visual grouping.
  - **L2055 CN**: 分隔注释，用于视觉分组。
- **L2056 EN**: Comment documents nearby intent or usage notes: `result is almost certainly going to underflow to zero, try logs just in case:`.
  - **L2056 CN**: 注释说明附近代码的意图或使用说明：`result is almost certainly going to underflow to zero, try logs just in case:`。
- **L2057 EN**: Separator comment used for visual grouping.
  - **L2057 CN**: 分隔注释，用于视觉分组。
- **L2058 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L2058 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L2059 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2059 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2060 EN**: Opens a new lexical scope or compound statement.
  - **L2060 CN**: 打开一个新的词法作用域或复合语句块。
- **L2061 EN**: Returns from the current function with `::expf(::lgammaf(x) - ::lgammaf(y))`.
  - **L2061 CN**: 以 `::expf(::lgammaf(x) - ::lgammaf(y))` 从当前函数返回。
- **L2062 EN**: Closes the current lexical scope or compound statement.
  - **L2062 CN**: 结束当前词法作用域或复合语句块。
- **L2063 EN**: Starts the alternative branch of the preceding conditional.
  - **L2063 CN**: 开始前一个条件语句的备选分支。
- **L2064 EN**: Opens a new lexical scope or compound statement.
  - **L2064 CN**: 打开一个新的词法作用域或复合语句块。
- **L2065 EN**: Returns from the current function with `::exp(::lgamma(x) - ::lgamma(y))`.
  - **L2065 CN**: 以 `::exp(::lgamma(x) - ::lgamma(y))` 从当前函数返回。
- **L2066 EN**: Closes the current lexical scope or compound statement.
  - **L2066 CN**: 结束当前词法作用域或复合语句块。
- **L2067 EN**: Continues the current preprocessor branch selection.
  - **L2067 CN**: 继续当前的预处理分支选择。
- **L2068 EN**: Returns from the current function with `exp(boost::math::lgamma(x, pol) - boost::math::lgamma(y, pol))`.
  - **L2068 CN**: 以 `exp(boost::math::lgamma(x, pol) - boost::math::lgamma(y, pol))` 从当前函数返回。
- **L2069 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2069 CN**: 结束当前预处理条件块或头文件保护。
- **L2070 EN**: Closes the current lexical scope or compound statement.
  - **L2070 CN**: 结束当前词法作用域或复合语句块。
- **L2071 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2071 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2072 EN**: Opens a new lexical scope or compound statement.
  - **L2072 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 2073-2100 / 第 2073-2100 行

````cpp
2073:       if(x < 2 * max_factorial<T>::value)
2074:       {
2075:          // We need to sidestep on y as well, otherwise we'll overflow
2076:          // before we get to factor in the prefix term:
2077:          prefix *= y;
2078:          y += 1;
2079:          while(x >= max_factorial<T>::value)
2080:          {
2081:             x -= 1;
2082:             prefix *= x;
2083:          }
2084: 
2085:          #ifdef BOOST_MATH_HAS_NVRTC
2086:          if (boost::math::is_same_v<T, float>)
2087:          {
2088:             return prefix * ::tgammaf(x) / ::tgammaf(y);
2089:          }
2090:          else
2091:          {
2092:             return prefix * ::tgamma(x) / ::tgamma(y);
2093:          }
2094:          #else
2095:          return prefix * boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol);
2096:          #endif
2097:       }
2098:       //
2099:       // Result will almost certainly overflow, try logs just in case:
2100:       //
````
- **L2073 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2073 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2074 EN**: Opens a new lexical scope or compound statement.
  - **L2074 CN**: 打开一个新的词法作用域或复合语句块。
- **L2075 EN**: Comment documents nearby intent or usage notes: `We need to sidestep on y as well, otherwise we'll overflow`.
  - **L2075 CN**: 注释说明附近代码的意图或使用说明：`We need to sidestep on y as well, otherwise we'll overflow`。
- **L2076 EN**: Comment documents nearby intent or usage notes: `before we get to factor in the prefix term:`.
  - **L2076 CN**: 注释说明附近代码的意图或使用说明：`before we get to factor in the prefix term:`。
- **L2077 EN**: Executes a standalone statement or declaration: `prefix *= y;`.
  - **L2077 CN**: 执行一条独立语句或声明：`prefix *= y;`。
- **L2078 EN**: Executes a standalone statement or declaration: `y += 1;`.
  - **L2078 CN**: 执行一条独立语句或声明：`y += 1;`。
- **L2079 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L2079 CN**: 开始 `while` 控制流语句并计算其条件。
- **L2080 EN**: Opens a new lexical scope or compound statement.
  - **L2080 CN**: 打开一个新的词法作用域或复合语句块。
- **L2081 EN**: Executes a standalone statement or declaration: `x -= 1;`.
  - **L2081 CN**: 执行一条独立语句或声明：`x -= 1;`。
- **L2082 EN**: Executes a standalone statement or declaration: `prefix *= x;`.
  - **L2082 CN**: 执行一条独立语句或声明：`prefix *= x;`。
- **L2083 EN**: Closes the current lexical scope or compound statement.
  - **L2083 CN**: 结束当前词法作用域或复合语句块。
- **L2084 EN**: Blank line separating nearby declarations or logic.
  - **L2084 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2085 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L2085 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L2086 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2086 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2087 EN**: Opens a new lexical scope or compound statement.
  - **L2087 CN**: 打开一个新的词法作用域或复合语句块。
- **L2088 EN**: Returns from the current function with `prefix * ::tgammaf(x) / ::tgammaf(y)`.
  - **L2088 CN**: 以 `prefix * ::tgammaf(x) / ::tgammaf(y)` 从当前函数返回。
- **L2089 EN**: Closes the current lexical scope or compound statement.
  - **L2089 CN**: 结束当前词法作用域或复合语句块。
- **L2090 EN**: Starts the alternative branch of the preceding conditional.
  - **L2090 CN**: 开始前一个条件语句的备选分支。
- **L2091 EN**: Opens a new lexical scope or compound statement.
  - **L2091 CN**: 打开一个新的词法作用域或复合语句块。
- **L2092 EN**: Returns from the current function with `prefix * ::tgamma(x) / ::tgamma(y)`.
  - **L2092 CN**: 以 `prefix * ::tgamma(x) / ::tgamma(y)` 从当前函数返回。
- **L2093 EN**: Closes the current lexical scope or compound statement.
  - **L2093 CN**: 结束当前词法作用域或复合语句块。
- **L2094 EN**: Continues the current preprocessor branch selection.
  - **L2094 CN**: 继续当前的预处理分支选择。
- **L2095 EN**: Returns from the current function with `prefix * boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol)`.
  - **L2095 CN**: 以 `prefix * boost::math::tgamma(x, pol) / boost::math::tgamma(y, pol)` 从当前函数返回。
- **L2096 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2096 CN**: 结束当前预处理条件块或头文件保护。
- **L2097 EN**: Closes the current lexical scope or compound statement.
  - **L2097 CN**: 结束当前词法作用域或复合语句块。
- **L2098 EN**: Separator comment used for visual grouping.
  - **L2098 CN**: 分隔注释，用于视觉分组。
- **L2099 EN**: Comment documents nearby intent or usage notes: `Result will almost certainly overflow, try logs just in case:`.
  - **L2099 CN**: 注释说明附近代码的意图或使用说明：`Result will almost certainly overflow, try logs just in case:`。
- **L2100 EN**: Separator comment used for visual grouping.
  - **L2100 CN**: 分隔注释，用于视觉分组。

### Lines 2101-2128 / 第 2101-2128 行

````cpp
2101:       BOOST_MATH_IF_CONSTEXPR(boost::math::is_same<T, float>::value || boost::math::is_same<T, double>::value)
2102:       {
2103:          // straight to the scene of the accident, since the result is larger than max_factorial:
2104:          return policies::raise_overflow_error<T>("tgamma_ratio", nullptr, pol);
2105:       }
2106:       else
2107:       {
2108:          #ifdef BOOST_MATH_HAS_NVRTC
2109:          if (boost::math::is_same_v<T, float>)
2110:          {
2111:             prefix = ::lgammaf(x) - ::lgammaf(y);
2112:          }
2113:          else
2114:          {
2115:             prefix = ::lgamma(x) - ::lgamma(y);
2116:          }
2117:          #else
2118:          prefix = boost::math::lgamma(x, pol) - boost::math::lgamma(y, pol);
2119:          #endif
2120:          if (prefix > boost::math::tools::log_max_value<T>())
2121:             return policies::raise_overflow_error<T>("tgamma_ratio", nullptr, pol);
2122:          //
2123:          // This is unreachable, unless max_factorial is small compared to the exponent
2124:          // range of the type, ie multiprecision types only here...
2125:          //
2126:          return exp(prefix);  // LCOV_EXCL_LINE
2127:       }
2128:    }
````
- **L2101 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2101 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2102 EN**: Opens a new lexical scope or compound statement.
  - **L2102 CN**: 打开一个新的词法作用域或复合语句块。
- **L2103 EN**: Comment documents nearby intent or usage notes: `straight to the scene of the accident, since the result is larger than max_factorial:`.
  - **L2103 CN**: 注释说明附近代码的意图或使用说明：`straight to the scene of the accident, since the result is larger than max_factorial:`。
- **L2104 EN**: Returns from the current function with `policies::raise_overflow_error<T>("tgamma_ratio", nullptr, pol)`.
  - **L2104 CN**: 以 `policies::raise_overflow_error<T>("tgamma_ratio", nullptr, pol)` 从当前函数返回。
- **L2105 EN**: Closes the current lexical scope or compound statement.
  - **L2105 CN**: 结束当前词法作用域或复合语句块。
- **L2106 EN**: Starts the alternative branch of the preceding conditional.
  - **L2106 CN**: 开始前一个条件语句的备选分支。
- **L2107 EN**: Opens a new lexical scope or compound statement.
  - **L2107 CN**: 打开一个新的词法作用域或复合语句块。
- **L2108 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L2108 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L2109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2110 EN**: Opens a new lexical scope or compound statement.
  - **L2110 CN**: 打开一个新的词法作用域或复合语句块。
- **L2111 EN**: Executes a call or declaration centered on `::lgammaf`.
  - **L2111 CN**: 执行以 `::lgammaf` 为核心的调用或声明。
- **L2112 EN**: Closes the current lexical scope or compound statement.
  - **L2112 CN**: 结束当前词法作用域或复合语句块。
- **L2113 EN**: Starts the alternative branch of the preceding conditional.
  - **L2113 CN**: 开始前一个条件语句的备选分支。
- **L2114 EN**: Opens a new lexical scope or compound statement.
  - **L2114 CN**: 打开一个新的词法作用域或复合语句块。
- **L2115 EN**: Executes a call or declaration centered on `::lgamma`.
  - **L2115 CN**: 执行以 `::lgamma` 为核心的调用或声明。
- **L2116 EN**: Closes the current lexical scope or compound statement.
  - **L2116 CN**: 结束当前词法作用域或复合语句块。
- **L2117 EN**: Continues the current preprocessor branch selection.
  - **L2117 CN**: 继续当前的预处理分支选择。
- **L2118 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L2118 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L2119 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2119 CN**: 结束当前预处理条件块或头文件保护。
- **L2120 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2120 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2121 EN**: Returns from the current function with `policies::raise_overflow_error<T>("tgamma_ratio", nullptr, pol)`.
  - **L2121 CN**: 以 `policies::raise_overflow_error<T>("tgamma_ratio", nullptr, pol)` 从当前函数返回。
- **L2122 EN**: Separator comment used for visual grouping.
  - **L2122 CN**: 分隔注释，用于视觉分组。
- **L2123 EN**: Comment documents nearby intent or usage notes: `This is unreachable, unless max_factorial is small compared to the exponent`.
  - **L2123 CN**: 注释说明附近代码的意图或使用说明：`This is unreachable, unless max_factorial is small compared to the exponent`。
- **L2124 EN**: Comment documents nearby intent or usage notes: `range of the type, ie multiprecision types only here...`.
  - **L2124 CN**: 注释说明附近代码的意图或使用说明：`range of the type, ie multiprecision types only here...`。
- **L2125 EN**: Separator comment used for visual grouping.
  - **L2125 CN**: 分隔注释，用于视觉分组。
- **L2126 EN**: Returns from the current function with `exp(prefix);  // LCOV_EXCL_LINE`.
  - **L2126 CN**: 以 `exp(prefix);  // LCOV_EXCL_LINE` 从当前函数返回。
- **L2127 EN**: Closes the current lexical scope or compound statement.
  - **L2127 CN**: 结束当前词法作用域或复合语句块。
- **L2128 EN**: Closes the current lexical scope or compound statement.
  - **L2128 CN**: 结束当前词法作用域或复合语句块。

### Lines 2129-2156 / 第 2129-2156 行

````cpp
2129:    //
2130:    // Regular case, x and y both large and similar in magnitude:
2131:    //
2132:    #ifdef BOOST_MATH_HAS_NVRTC
2133:    return detail::tgamma_delta_ratio_imp(x, y - x, pol);
2134:    #else
2135:    return boost::math::tgamma_delta_ratio(x, y - x, pol);
2136:    #endif
2137: }
2138: 
2139: template <class T, class Policy>
2140: BOOST_MATH_GPU_ENABLED T gamma_p_derivative_imp(T a, T x, const Policy& pol)
2141: {
2142:    BOOST_MATH_STD_USING
2143:    //
2144:    // Usual error checks first:
2145:    //
2146:    if(a <= 0)
2147:       return policies::raise_domain_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", "Argument a to the incomplete gamma function must be greater than zero (got a=%1%).", a, pol);
2148:    if(x < 0)
2149:       return policies::raise_domain_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", "Argument x to the incomplete gamma function must be >= 0 (got x=%1%).", x, pol);
2150:    //
2151:    // Now special cases:
2152:    //
2153:    if(x == 0)
2154:    {
2155:       return (a > 1) ? T(0) :
2156:          (a == 1) ? T(1) : policies::raise_overflow_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", nullptr, pol);
````
- **L2129 EN**: Separator comment used for visual grouping.
  - **L2129 CN**: 分隔注释，用于视觉分组。
- **L2130 EN**: Comment documents nearby intent or usage notes: `Regular case, x and y both large and similar in magnitude:`.
  - **L2130 CN**: 注释说明附近代码的意图或使用说明：`Regular case, x and y both large and similar in magnitude:`。
- **L2131 EN**: Separator comment used for visual grouping.
  - **L2131 CN**: 分隔注释，用于视觉分组。
- **L2132 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L2132 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L2133 EN**: Returns from the current function with `detail::tgamma_delta_ratio_imp(x, y - x, pol)`.
  - **L2133 CN**: 以 `detail::tgamma_delta_ratio_imp(x, y - x, pol)` 从当前函数返回。
- **L2134 EN**: Continues the current preprocessor branch selection.
  - **L2134 CN**: 继续当前的预处理分支选择。
- **L2135 EN**: Returns from the current function with `boost::math::tgamma_delta_ratio(x, y - x, pol)`.
  - **L2135 CN**: 以 `boost::math::tgamma_delta_ratio(x, y - x, pol)` 从当前函数返回。
- **L2136 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2136 CN**: 结束当前预处理条件块或头文件保护。
- **L2137 EN**: Closes the current lexical scope or compound statement.
  - **L2137 CN**: 结束当前词法作用域或复合语句块。
- **L2138 EN**: Blank line separating nearby declarations or logic.
  - **L2138 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2139 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L2139 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L2140 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2140 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2141 EN**: Opens a new lexical scope or compound statement.
  - **L2141 CN**: 打开一个新的词法作用域或复合语句块。
- **L2142 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2142 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2143 EN**: Separator comment used for visual grouping.
  - **L2143 CN**: 分隔注释，用于视觉分组。
- **L2144 EN**: Comment documents nearby intent or usage notes: `Usual error checks first:`.
  - **L2144 CN**: 注释说明附近代码的意图或使用说明：`Usual error checks first:`。
- **L2145 EN**: Separator comment used for visual grouping.
  - **L2145 CN**: 分隔注释，用于视觉分组。
- **L2146 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2146 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2147 EN**: Returns from the current function with `policies::raise_domain_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", "Argument a to the incomplete gamma function must be greater than zero (got a=%1%).", a, pol)`.
  - **L2147 CN**: 以 `policies::raise_domain_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", "Argument a to the incomplete gamma function must be greater than zero (got a=%1%).", a, pol)` 从当前函数返回。
- **L2148 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2148 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2149 EN**: Returns from the current function with `policies::raise_domain_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", "Argument x to the incomplete gamma function must be >= 0 (got x=%1%).", x, pol)`.
  - **L2149 CN**: 以 `policies::raise_domain_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", "Argument x to the incomplete gamma function must be >= 0 (got x=%1%).", x, pol)` 从当前函数返回。
- **L2150 EN**: Separator comment used for visual grouping.
  - **L2150 CN**: 分隔注释，用于视觉分组。
- **L2151 EN**: Comment documents nearby intent or usage notes: `Now special cases:`.
  - **L2151 CN**: 注释说明附近代码的意图或使用说明：`Now special cases:`。
- **L2152 EN**: Separator comment used for visual grouping.
  - **L2152 CN**: 分隔注释，用于视觉分组。
- **L2153 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2153 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2154 EN**: Opens a new lexical scope or compound statement.
  - **L2154 CN**: 打开一个新的词法作用域或复合语句块。
- **L2155 EN**: Returns from the current function with `(a > 1) ? T(0) :`.
  - **L2155 CN**: 以 `(a > 1) ? T(0) :` 从当前函数返回。
- **L2156 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L2156 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 2157-2184 / 第 2157-2184 行

````cpp
2157:    }
2158:    //
2159:    // Normal case:
2160:    //
2161:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
2162:    T f1 = detail::regularised_gamma_prefix(a, x, pol, lanczos_type());
2163:    /*
2164:    * Derivative goes to zero as x -> 0, this should be unreachable:
2165:    * 
2166:    if((x < 1) && (tools::max_value<T>() * x < f1))
2167:    {
2168:       // overflow:
2169:       return policies::raise_overflow_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", nullptr, pol);
2170:    }
2171:    */
2172:    if(f1 == 0)
2173:    {
2174:       // Underflow in calculation, use logs instead:
2175:       #ifdef BOOST_MATH_HAS_NVRTC
2176:       if (boost::math::is_same_v<T, float>)
2177:       {
2178:          f1 = a * ::logf(x) - x - ::lgammaf(a) - ::logf(x);
2179:       }
2180:       else
2181:       {
2182:          f1 = a * ::log(x) - x - ::lgamma(a) - ::log(x);
2183:       }
2184:       #else
````
- **L2157 EN**: Closes the current lexical scope or compound statement.
  - **L2157 CN**: 结束当前词法作用域或复合语句块。
- **L2158 EN**: Separator comment used for visual grouping.
  - **L2158 CN**: 分隔注释，用于视觉分组。
- **L2159 EN**: Comment documents nearby intent or usage notes: `Normal case:`.
  - **L2159 CN**: 注释说明附近代码的意图或使用说明：`Normal case:`。
- **L2160 EN**: Separator comment used for visual grouping.
  - **L2160 CN**: 分隔注释，用于视觉分组。
- **L2161 EN**: Introduces a legacy type alias or function typedef: `typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;`.
  - **L2161 CN**: 引入传统类型别名或函数 typedef：`typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;`。
- **L2162 EN**: Executes a call or declaration centered on `detail::regularised_gamma_prefix`.
  - **L2162 CN**: 执行以 `detail::regularised_gamma_prefix` 为核心的调用或声明。
- **L2163 EN**: Separator comment used for visual grouping.
  - **L2163 CN**: 分隔注释，用于视觉分组。
- **L2164 EN**: Comment documents nearby intent or usage notes: `Derivative goes to zero as x -> 0, this should be unreachable:`.
  - **L2164 CN**: 注释说明附近代码的意图或使用说明：`Derivative goes to zero as x -> 0, this should be unreachable:`。
- **L2165 EN**: Separator comment used for visual grouping.
  - **L2165 CN**: 分隔注释，用于视觉分组。
- **L2166 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2166 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2167 EN**: Opens a new lexical scope or compound statement.
  - **L2167 CN**: 打开一个新的词法作用域或复合语句块。
- **L2168 EN**: Comment documents nearby intent or usage notes: `overflow:`.
  - **L2168 CN**: 注释说明附近代码的意图或使用说明：`overflow:`。
- **L2169 EN**: Returns from the current function with `policies::raise_overflow_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", nullptr, pol)`.
  - **L2169 CN**: 以 `policies::raise_overflow_error<T>("boost::math::gamma_p_derivative<%1%>(%1%, %1%)", nullptr, pol)` 从当前函数返回。
- **L2170 EN**: Closes the current lexical scope or compound statement.
  - **L2170 CN**: 结束当前词法作用域或复合语句块。
- **L2171 EN**: Comment documents nearby intent or usage notes: `/`.
  - **L2171 CN**: 注释说明附近代码的意图或使用说明：`/`。
- **L2172 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2172 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2173 EN**: Opens a new lexical scope or compound statement.
  - **L2173 CN**: 打开一个新的词法作用域或复合语句块。
- **L2174 EN**: Comment documents nearby intent or usage notes: `Underflow in calculation, use logs instead:`.
  - **L2174 CN**: 注释说明附近代码的意图或使用说明：`Underflow in calculation, use logs instead:`。
- **L2175 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L2175 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L2176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L2176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L2177 EN**: Opens a new lexical scope or compound statement.
  - **L2177 CN**: 打开一个新的词法作用域或复合语句块。
- **L2178 EN**: Executes a call or declaration centered on `::logf`.
  - **L2178 CN**: 执行以 `::logf` 为核心的调用或声明。
- **L2179 EN**: Closes the current lexical scope or compound statement.
  - **L2179 CN**: 结束当前词法作用域或复合语句块。
- **L2180 EN**: Starts the alternative branch of the preceding conditional.
  - **L2180 CN**: 开始前一个条件语句的备选分支。
- **L2181 EN**: Opens a new lexical scope or compound statement.
  - **L2181 CN**: 打开一个新的词法作用域或复合语句块。
- **L2182 EN**: Executes a call or declaration centered on `::log`.
  - **L2182 CN**: 执行以 `::log` 为核心的调用或声明。
- **L2183 EN**: Closes the current lexical scope or compound statement.
  - **L2183 CN**: 结束当前词法作用域或复合语句块。
- **L2184 EN**: Continues the current preprocessor branch selection.
  - **L2184 CN**: 继续当前的预处理分支选择。

### Lines 2185-2212 / 第 2185-2212 行

````cpp
2185:       f1 = a * log(x) - x - lgamma(a, pol) - log(x);
2186:       #endif
2187:       f1 = exp(f1);
2188:    }
2189:    else
2190:       f1 /= x;
2191: 
2192:    return f1;
2193: }
2194: 
2195: template <class T, class Policy>
2196: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2197:    tgamma(T z, const Policy& /* pol */, const boost::math::true_type)
2198: {
2199:    BOOST_FPU_EXCEPTION_GUARD
2200:    typedef typename tools::promote_args<T>::type result_type;
2201:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2202:    typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
2203:    typedef typename policies::normalise<
2204:       Policy,
2205:       policies::promote_float<false>,
2206:       policies::promote_double<false>,
2207:       policies::discrete_quantile<>,
2208:       policies::assert_undefined<> >::type forwarding_policy;
2209:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::gamma_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type()), "boost::math::tgamma<%1%>(%1%)");
2210: }
2211: 
2212: template <class T1, class T2, class Policy>
````
- **L2185 EN**: Executes a call or declaration centered on `log`.
  - **L2185 CN**: 执行以 `log` 为核心的调用或声明。
- **L2186 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2186 CN**: 结束当前预处理条件块或头文件保护。
- **L2187 EN**: Executes a call or declaration centered on `exp`.
  - **L2187 CN**: 执行以 `exp` 为核心的调用或声明。
- **L2188 EN**: Closes the current lexical scope or compound statement.
  - **L2188 CN**: 结束当前词法作用域或复合语句块。
- **L2189 EN**: Starts the alternative branch of the preceding conditional.
  - **L2189 CN**: 开始前一个条件语句的备选分支。
- **L2190 EN**: Executes a standalone statement or declaration: `f1 /= x;`.
  - **L2190 CN**: 执行一条独立语句或声明：`f1 /= x;`。
- **L2191 EN**: Blank line separating nearby declarations or logic.
  - **L2191 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2192 EN**: Returns from the current function with `f1`.
  - **L2192 CN**: 以 `f1` 从当前函数返回。
- **L2193 EN**: Closes the current lexical scope or compound statement.
  - **L2193 CN**: 结束当前词法作用域或复合语句块。
- **L2194 EN**: Blank line separating nearby declarations or logic.
  - **L2194 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2195 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L2195 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L2196 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2196 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2197 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L2197 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L2198 EN**: Opens a new lexical scope or compound statement.
  - **L2198 CN**: 打开一个新的词法作用域或复合语句块。
- **L2199 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2199 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2200 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T>::type result_type;`.
  - **L2200 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T>::type result_type;`。
- **L2201 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2201 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2202 EN**: Introduces a legacy type alias or function typedef: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L2202 CN**: 引入传统类型别名或函数 typedef：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L2203 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2203 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2204 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2205 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2206 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2207 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2208 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2208 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2209 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::gamma_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type()), "boost::math::tgamma<%1%>(%1%)")`.
  - **L2209 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::gamma_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type()), "boost::math::tgamma<%1%>(%1%)")` 从当前函数返回。
- **L2210 EN**: Closes the current lexical scope or compound statement.
  - **L2210 CN**: 结束当前词法作用域或复合语句块。
- **L2211 EN**: Blank line separating nearby declarations or logic.
  - **L2211 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2212 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2212 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。

### Lines 2213-2240 / 第 2213-2240 行

````cpp
2213: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2214:    tgamma(T1 a, T2 z, const Policy&, const boost::math::false_type)
2215: {
2216:    BOOST_FPU_EXCEPTION_GUARD
2217:    typedef tools::promote_args_t<T1, T2> result_type;
2218:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2219:    // typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
2220:    typedef typename policies::normalise<
2221:       Policy,
2222:       policies::promote_float<false>,
2223:       policies::promote_double<false>,
2224:       policies::discrete_quantile<>,
2225:       policies::assert_undefined<> >::type forwarding_policy;
2226: 
2227:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(
2228:       detail::gamma_incomplete_imp(static_cast<value_type>(a),
2229:       static_cast<value_type>(z), false, true,
2230:       forwarding_policy(), static_cast<value_type*>(nullptr)), "boost::math::tgamma<%1%>(%1%, %1%)");
2231: }
2232: 
2233: template <class T1, class T2>
2234: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2235:    tgamma(T1 a, T2 z, const boost::math::false_type& tag)
2236: {
2237:    return tgamma(a, z, policies::policy<>(), tag);
2238: }
2239: 
2240: 
````
- **L2213 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2213 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2214 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L2214 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L2215 EN**: Opens a new lexical scope or compound statement.
  - **L2215 CN**: 打开一个新的词法作用域或复合语句块。
- **L2216 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2216 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2217 EN**: Introduces a legacy type alias or function typedef: `typedef tools::promote_args_t<T1, T2> result_type;`.
  - **L2217 CN**: 引入传统类型别名或函数 typedef：`typedef tools::promote_args_t<T1, T2> result_type;`。
- **L2218 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2218 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2219 EN**: Comment documents nearby intent or usage notes: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L2219 CN**: 注释说明附近代码的意图或使用说明：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L2220 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2220 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2221 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2222 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2223 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2224 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2225 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2225 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2226 EN**: Blank line separating nearby declarations or logic.
  - **L2226 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2227 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(`.
  - **L2227 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(` 从当前函数返回。
- **L2228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `detail::gamma_incomplete_imp(static_cast<value_type>(a),`.
  - **L2228 CN**: 继续一个多行参数列表、初始化器或聚合项：`detail::gamma_incomplete_imp(static_cast<value_type>(a),`。
- **L2229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(z), false, true,`.
  - **L2229 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(z), false, true,`。
- **L2230 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L2230 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L2231 EN**: Closes the current lexical scope or compound statement.
  - **L2231 CN**: 结束当前词法作用域或复合语句块。
- **L2232 EN**: Blank line separating nearby declarations or logic.
  - **L2232 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2233 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2233 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2234 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2234 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2235 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L2235 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L2236 EN**: Opens a new lexical scope or compound statement.
  - **L2236 CN**: 打开一个新的词法作用域或复合语句块。
- **L2237 EN**: Returns from the current function with `tgamma(a, z, policies::policy<>(), tag)`.
  - **L2237 CN**: 以 `tgamma(a, z, policies::policy<>(), tag)` 从当前函数返回。
- **L2238 EN**: Closes the current lexical scope or compound statement.
  - **L2238 CN**: 结束当前词法作用域或复合语句块。
- **L2239 EN**: Blank line separating nearby declarations or logic.
  - **L2239 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2240 EN**: Blank line separating nearby declarations or logic.
  - **L2240 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 2241-2268 / 第 2241-2268 行

````cpp
2241: } // namespace detail
2242: 
2243: template <class T, class Policy>
2244: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2245:    lgamma(T z, int* sign, const Policy&)
2246: {
2247:    BOOST_FPU_EXCEPTION_GUARD
2248:    typedef typename tools::promote_args<T>::type result_type;
2249:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2250:    typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
2251:    typedef typename policies::normalise<
2252:       Policy,
2253:       policies::promote_float<false>,
2254:       policies::promote_double<false>,
2255:       policies::discrete_quantile<>,
2256:       policies::assert_undefined<> >::type forwarding_policy;
2257: 
2258:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::lgamma_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type(), sign), "boost::math::lgamma<%1%>(%1%)");
2259: }
2260: 
2261: template <class T>
2262: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2263:    lgamma(T z, int* sign)
2264: {
2265:    return lgamma(z, sign, policies::policy<>());
2266: }
2267: 
2268: template <class T, class Policy>
````
- **L2241 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L2241 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。
- **L2242 EN**: Blank line separating nearby declarations or logic.
  - **L2242 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2243 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L2243 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L2244 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2244 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2245 EN**: Continues logic associated with callable symbol `lgamma`.
  - **L2245 CN**: 继续与可调用符号 `lgamma` 相关的逻辑。
- **L2246 EN**: Opens a new lexical scope or compound statement.
  - **L2246 CN**: 打开一个新的词法作用域或复合语句块。
- **L2247 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2247 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2248 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T>::type result_type;`.
  - **L2248 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T>::type result_type;`。
- **L2249 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2249 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2250 EN**: Introduces a legacy type alias or function typedef: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L2250 CN**: 引入传统类型别名或函数 typedef：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L2251 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2251 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2252 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2253 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2254 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2255 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2256 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2256 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2257 EN**: Blank line separating nearby declarations or logic.
  - **L2257 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2258 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::lgamma_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type(), sign), "boost::math::lgamma<%1%>(%1%)")`.
  - **L2258 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::lgamma_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type(), sign), "boost::math::lgamma<%1%>(%1%)")` 从当前函数返回。
- **L2259 EN**: Closes the current lexical scope or compound statement.
  - **L2259 CN**: 结束当前词法作用域或复合语句块。
- **L2260 EN**: Blank line separating nearby declarations or logic.
  - **L2260 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2261 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L2261 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L2262 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2262 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2263 EN**: Continues logic associated with callable symbol `lgamma`.
  - **L2263 CN**: 继续与可调用符号 `lgamma` 相关的逻辑。
- **L2264 EN**: Opens a new lexical scope or compound statement.
  - **L2264 CN**: 打开一个新的词法作用域或复合语句块。
- **L2265 EN**: Returns from the current function with `lgamma(z, sign, policies::policy<>())`.
  - **L2265 CN**: 以 `lgamma(z, sign, policies::policy<>())` 从当前函数返回。
- **L2266 EN**: Closes the current lexical scope or compound statement.
  - **L2266 CN**: 结束当前词法作用域或复合语句块。
- **L2267 EN**: Blank line separating nearby declarations or logic.
  - **L2267 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2268 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L2268 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。

### Lines 2269-2296 / 第 2269-2296 行

````cpp
2269: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2270:    lgamma(T x, const Policy& pol)
2271: {
2272:    return ::boost::math::lgamma(x, nullptr, pol);
2273: }
2274: 
2275: template <class T>
2276: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2277:    lgamma(T x)
2278: {
2279:    return ::boost::math::lgamma(x, nullptr, policies::policy<>());
2280: }
2281: 
2282: template <class T, class Policy>
2283: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2284:    tgamma1pm1(T z, const Policy& /* pol */)
2285: {
2286:    BOOST_FPU_EXCEPTION_GUARD
2287:    typedef typename tools::promote_args<T>::type result_type;
2288:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2289:    typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
2290:    typedef typename policies::normalise<
2291:       Policy,
2292:       policies::promote_float<false>,
2293:       policies::promote_double<false>,
2294:       policies::discrete_quantile<>,
2295:       policies::assert_undefined<> >::type forwarding_policy;
2296: 
````
- **L2269 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2269 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2270 EN**: Continues logic associated with callable symbol `lgamma`.
  - **L2270 CN**: 继续与可调用符号 `lgamma` 相关的逻辑。
- **L2271 EN**: Opens a new lexical scope or compound statement.
  - **L2271 CN**: 打开一个新的词法作用域或复合语句块。
- **L2272 EN**: Returns from the current function with `::boost::math::lgamma(x, nullptr, pol)`.
  - **L2272 CN**: 以 `::boost::math::lgamma(x, nullptr, pol)` 从当前函数返回。
- **L2273 EN**: Closes the current lexical scope or compound statement.
  - **L2273 CN**: 结束当前词法作用域或复合语句块。
- **L2274 EN**: Blank line separating nearby declarations or logic.
  - **L2274 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2275 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L2275 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L2276 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2276 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2277 EN**: Continues logic associated with callable symbol `lgamma`.
  - **L2277 CN**: 继续与可调用符号 `lgamma` 相关的逻辑。
- **L2278 EN**: Opens a new lexical scope or compound statement.
  - **L2278 CN**: 打开一个新的词法作用域或复合语句块。
- **L2279 EN**: Returns from the current function with `::boost::math::lgamma(x, nullptr, policies::policy<>())`.
  - **L2279 CN**: 以 `::boost::math::lgamma(x, nullptr, policies::policy<>())` 从当前函数返回。
- **L2280 EN**: Closes the current lexical scope or compound statement.
  - **L2280 CN**: 结束当前词法作用域或复合语句块。
- **L2281 EN**: Blank line separating nearby declarations or logic.
  - **L2281 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2282 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L2282 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L2283 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2283 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2284 EN**: Continues logic associated with callable symbol `tgamma1pm1`.
  - **L2284 CN**: 继续与可调用符号 `tgamma1pm1` 相关的逻辑。
- **L2285 EN**: Opens a new lexical scope or compound statement.
  - **L2285 CN**: 打开一个新的词法作用域或复合语句块。
- **L2286 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2286 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2287 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T>::type result_type;`.
  - **L2287 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T>::type result_type;`。
- **L2288 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2288 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2289 EN**: Introduces a legacy type alias or function typedef: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L2289 CN**: 引入传统类型别名或函数 typedef：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L2290 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2290 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2291 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2292 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2293 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2294 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2295 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2295 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2296 EN**: Blank line separating nearby declarations or logic.
  - **L2296 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 2297-2324 / 第 2297-2324 行

````cpp
2297:    return policies::checked_narrowing_cast<typename boost::math::remove_cv<result_type>::type, forwarding_policy>(detail::tgammap1m1_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type()), "boost::math::tgamma1pm1<%!%>(%1%)");
2298: }
2299: 
2300: template <class T>
2301: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2302:    tgamma1pm1(T z)
2303: {
2304:    return tgamma1pm1(z, policies::policy<>());
2305: }
2306: 
2307: //
2308: // Full upper incomplete gamma:
2309: //
2310: template <class T1, class T2>
2311: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2312:    tgamma(T1 a, T2 z)
2313: {
2314:    //
2315:    // Type T2 could be a policy object, or a value, select the
2316:    // right overload based on T2:
2317:    //
2318:    using maybe_policy = typename policies::is_policy<T2>::type;
2319:    using result_type = tools::promote_args_t<T1, T2>;
2320:    return static_cast<result_type>(detail::tgamma(a, z, maybe_policy()));
2321: }
2322: template <class T1, class T2, class Policy>
2323: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2324:    tgamma(T1 a, T2 z, const Policy& pol)
````
- **L2297 EN**: Returns from the current function with `policies::checked_narrowing_cast<typename boost::math::remove_cv<result_type>::type, forwarding_policy>(detail::tgammap1m1_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type()), "boost::math::tgamma1pm1<%!%>(%1%)")`.
  - **L2297 CN**: 以 `policies::checked_narrowing_cast<typename boost::math::remove_cv<result_type>::type, forwarding_policy>(detail::tgammap1m1_imp(static_cast<value_type>(z), forwarding_policy(), evaluation_type()), "boost::math::tgamma1pm1<%!%>(%1%)")` 从当前函数返回。
- **L2298 EN**: Closes the current lexical scope or compound statement.
  - **L2298 CN**: 结束当前词法作用域或复合语句块。
- **L2299 EN**: Blank line separating nearby declarations or logic.
  - **L2299 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2300 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L2300 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L2301 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2301 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2302 EN**: Continues logic associated with callable symbol `tgamma1pm1`.
  - **L2302 CN**: 继续与可调用符号 `tgamma1pm1` 相关的逻辑。
- **L2303 EN**: Opens a new lexical scope or compound statement.
  - **L2303 CN**: 打开一个新的词法作用域或复合语句块。
- **L2304 EN**: Returns from the current function with `tgamma1pm1(z, policies::policy<>())`.
  - **L2304 CN**: 以 `tgamma1pm1(z, policies::policy<>())` 从当前函数返回。
- **L2305 EN**: Closes the current lexical scope or compound statement.
  - **L2305 CN**: 结束当前词法作用域或复合语句块。
- **L2306 EN**: Blank line separating nearby declarations or logic.
  - **L2306 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2307 EN**: Separator comment used for visual grouping.
  - **L2307 CN**: 分隔注释，用于视觉分组。
- **L2308 EN**: Comment documents nearby intent or usage notes: `Full upper incomplete gamma:`.
  - **L2308 CN**: 注释说明附近代码的意图或使用说明：`Full upper incomplete gamma:`。
- **L2309 EN**: Separator comment used for visual grouping.
  - **L2309 CN**: 分隔注释，用于视觉分组。
- **L2310 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2310 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2311 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2311 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2312 EN**: Continues logic associated with callable symbol `tgamma`.
  - **L2312 CN**: 继续与可调用符号 `tgamma` 相关的逻辑。
- **L2313 EN**: Opens a new lexical scope or compound statement.
  - **L2313 CN**: 打开一个新的词法作用域或复合语句块。
- **L2314 EN**: Separator comment used for visual grouping.
  - **L2314 CN**: 分隔注释，用于视觉分组。
- **L2315 EN**: Comment documents nearby intent or usage notes: `Type T2 could be a policy object, or a value, select the`.
  - **L2315 CN**: 注释说明附近代码的意图或使用说明：`Type T2 could be a policy object, or a value, select the`。
- **L2316 EN**: Comment documents nearby intent or usage notes: `right overload based on T2:`.
  - **L2316 CN**: 注释说明附近代码的意图或使用说明：`right overload based on T2:`。
- **L2317 EN**: Separator comment used for visual grouping.
  - **L2317 CN**: 分隔注释，用于视觉分组。
- **L2318 EN**: Defines alias `maybe_policy` to simplify later code.
  - **L2318 CN**: 定义别名 `maybe_policy` 以简化后续代码。
- **L2319 EN**: Defines alias `result_type` to simplify later code.
  - **L2319 CN**: 定义别名 `result_type` 以简化后续代码。
- **L2320 EN**: Returns from the current function with `static_cast<result_type>(detail::tgamma(a, z, maybe_policy()))`.
  - **L2320 CN**: 以 `static_cast<result_type>(detail::tgamma(a, z, maybe_policy()))` 从当前函数返回。
- **L2321 EN**: Closes the current lexical scope or compound statement.
  - **L2321 CN**: 结束当前词法作用域或复合语句块。
- **L2322 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2322 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L2323 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2323 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2324 EN**: Continues logic associated with callable symbol `tgamma`.
  - **L2324 CN**: 继续与可调用符号 `tgamma` 相关的逻辑。

### Lines 2325-2352 / 第 2325-2352 行

````cpp
2325: {
2326:    using result_type = tools::promote_args_t<T1, T2>;
2327:    return static_cast<result_type>(detail::tgamma(a, z, pol, boost::math::false_type()));
2328: }
2329: template <class T>
2330: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
2331:    tgamma(T z)
2332: {
2333:    return tgamma(z, policies::policy<>());
2334: }
2335: //
2336: // Full lower incomplete gamma:
2337: //
2338: template <class T1, class T2, class Policy>
2339: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2340:    tgamma_lower(T1 a, T2 z, const Policy&)
2341: {
2342:    BOOST_FPU_EXCEPTION_GUARD
2343:    typedef tools::promote_args_t<T1, T2> result_type;
2344:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2345:    // typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
2346:    typedef typename policies::normalise<
2347:       Policy,
2348:       policies::promote_float<false>,
2349:       policies::promote_double<false>,
2350:       policies::discrete_quantile<>,
2351:       policies::assert_undefined<> >::type forwarding_policy;
2352: 
````
- **L2325 EN**: Opens a new lexical scope or compound statement.
  - **L2325 CN**: 打开一个新的词法作用域或复合语句块。
- **L2326 EN**: Defines alias `result_type` to simplify later code.
  - **L2326 CN**: 定义别名 `result_type` 以简化后续代码。
- **L2327 EN**: Returns from the current function with `static_cast<result_type>(detail::tgamma(a, z, pol, boost::math::false_type()))`.
  - **L2327 CN**: 以 `static_cast<result_type>(detail::tgamma(a, z, pol, boost::math::false_type()))` 从当前函数返回。
- **L2328 EN**: Closes the current lexical scope or compound statement.
  - **L2328 CN**: 结束当前词法作用域或复合语句块。
- **L2329 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L2329 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L2330 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2330 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2331 EN**: Continues logic associated with callable symbol `tgamma`.
  - **L2331 CN**: 继续与可调用符号 `tgamma` 相关的逻辑。
- **L2332 EN**: Opens a new lexical scope or compound statement.
  - **L2332 CN**: 打开一个新的词法作用域或复合语句块。
- **L2333 EN**: Returns from the current function with `tgamma(z, policies::policy<>())`.
  - **L2333 CN**: 以 `tgamma(z, policies::policy<>())` 从当前函数返回。
- **L2334 EN**: Closes the current lexical scope or compound statement.
  - **L2334 CN**: 结束当前词法作用域或复合语句块。
- **L2335 EN**: Separator comment used for visual grouping.
  - **L2335 CN**: 分隔注释，用于视觉分组。
- **L2336 EN**: Comment documents nearby intent or usage notes: `Full lower incomplete gamma:`.
  - **L2336 CN**: 注释说明附近代码的意图或使用说明：`Full lower incomplete gamma:`。
- **L2337 EN**: Separator comment used for visual grouping.
  - **L2337 CN**: 分隔注释，用于视觉分组。
- **L2338 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2338 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L2339 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2339 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2340 EN**: Continues logic associated with callable symbol `tgamma_lower`.
  - **L2340 CN**: 继续与可调用符号 `tgamma_lower` 相关的逻辑。
- **L2341 EN**: Opens a new lexical scope or compound statement.
  - **L2341 CN**: 打开一个新的词法作用域或复合语句块。
- **L2342 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2342 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2343 EN**: Introduces a legacy type alias or function typedef: `typedef tools::promote_args_t<T1, T2> result_type;`.
  - **L2343 CN**: 引入传统类型别名或函数 typedef：`typedef tools::promote_args_t<T1, T2> result_type;`。
- **L2344 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2344 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2345 EN**: Comment documents nearby intent or usage notes: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L2345 CN**: 注释说明附近代码的意图或使用说明：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L2346 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2346 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2347 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2348 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2349 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2350 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2351 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2351 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2352 EN**: Blank line separating nearby declarations or logic.
  - **L2352 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 2353-2380 / 第 2353-2380 行

````cpp
2353:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(
2354:       detail::gamma_incomplete_imp(static_cast<value_type>(a),
2355:       static_cast<value_type>(z), false, false,
2356:       forwarding_policy(), static_cast<value_type*>(nullptr)), "tgamma_lower<%1%>(%1%, %1%)");
2357: }
2358: template <class T1, class T2>
2359: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2360:    tgamma_lower(T1 a, T2 z)
2361: {
2362:    return tgamma_lower(a, z, policies::policy<>());
2363: }
2364: //
2365: // Regularised upper incomplete gamma:
2366: //
2367: template <class T1, class T2, class Policy>
2368: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2369:    gamma_q(T1 a, T2 z, const Policy& /* pol */)
2370: {
2371:    BOOST_FPU_EXCEPTION_GUARD
2372:    typedef tools::promote_args_t<T1, T2> result_type;
2373:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2374:    // typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
2375:    typedef typename policies::normalise<
2376:       Policy,
2377:       policies::promote_float<false>,
2378:       policies::promote_double<false>,
2379:       policies::discrete_quantile<>,
2380:       policies::assert_undefined<> >::type forwarding_policy;
````
- **L2353 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(`.
  - **L2353 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(` 从当前函数返回。
- **L2354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `detail::gamma_incomplete_imp(static_cast<value_type>(a),`.
  - **L2354 CN**: 继续一个多行参数列表、初始化器或聚合项：`detail::gamma_incomplete_imp(static_cast<value_type>(a),`。
- **L2355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(z), false, false,`.
  - **L2355 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(z), false, false,`。
- **L2356 EN**: Executes a call or declaration centered on `forwarding_policy`.
  - **L2356 CN**: 执行以 `forwarding_policy` 为核心的调用或声明。
- **L2357 EN**: Closes the current lexical scope or compound statement.
  - **L2357 CN**: 结束当前词法作用域或复合语句块。
- **L2358 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2358 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2359 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2359 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2360 EN**: Continues logic associated with callable symbol `tgamma_lower`.
  - **L2360 CN**: 继续与可调用符号 `tgamma_lower` 相关的逻辑。
- **L2361 EN**: Opens a new lexical scope or compound statement.
  - **L2361 CN**: 打开一个新的词法作用域或复合语句块。
- **L2362 EN**: Returns from the current function with `tgamma_lower(a, z, policies::policy<>())`.
  - **L2362 CN**: 以 `tgamma_lower(a, z, policies::policy<>())` 从当前函数返回。
- **L2363 EN**: Closes the current lexical scope or compound statement.
  - **L2363 CN**: 结束当前词法作用域或复合语句块。
- **L2364 EN**: Separator comment used for visual grouping.
  - **L2364 CN**: 分隔注释，用于视觉分组。
- **L2365 EN**: Comment documents nearby intent or usage notes: `Regularised upper incomplete gamma:`.
  - **L2365 CN**: 注释说明附近代码的意图或使用说明：`Regularised upper incomplete gamma:`。
- **L2366 EN**: Separator comment used for visual grouping.
  - **L2366 CN**: 分隔注释，用于视觉分组。
- **L2367 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2367 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L2368 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2368 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2369 EN**: Continues logic associated with callable symbol `gamma_q`.
  - **L2369 CN**: 继续与可调用符号 `gamma_q` 相关的逻辑。
- **L2370 EN**: Opens a new lexical scope or compound statement.
  - **L2370 CN**: 打开一个新的词法作用域或复合语句块。
- **L2371 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2371 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2372 EN**: Introduces a legacy type alias or function typedef: `typedef tools::promote_args_t<T1, T2> result_type;`.
  - **L2372 CN**: 引入传统类型别名或函数 typedef：`typedef tools::promote_args_t<T1, T2> result_type;`。
- **L2373 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2373 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2374 EN**: Comment documents nearby intent or usage notes: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L2374 CN**: 注释说明附近代码的意图或使用说明：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L2375 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2375 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2376 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2377 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2378 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2379 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2380 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2380 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。

### Lines 2381-2408 / 第 2381-2408 行

````cpp
2381: 
2382:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(
2383:       detail::gamma_incomplete_imp(static_cast<value_type>(a),
2384:       static_cast<value_type>(z), true, true,
2385:       forwarding_policy(), static_cast<value_type*>(nullptr)), "gamma_q<%1%>(%1%, %1%)");
2386: }
2387: template <class T1, class T2>
2388: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2389:    gamma_q(T1 a, T2 z)
2390: {
2391:    return gamma_q(a, z, policies::policy<>());
2392: }
2393: //
2394: // Regularised lower incomplete gamma:
2395: //
2396: template <class T1, class T2, class Policy>
2397: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2398:    gamma_p(T1 a, T2 z, const Policy&)
2399: {
2400:    BOOST_FPU_EXCEPTION_GUARD
2401:    typedef tools::promote_args_t<T1, T2> result_type;
2402:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2403:    // typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
2404:    typedef typename policies::normalise<
2405:       Policy,
2406:       policies::promote_float<false>,
2407:       policies::promote_double<false>,
2408:       policies::discrete_quantile<>,
````
- **L2381 EN**: Blank line separating nearby declarations or logic.
  - **L2381 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2382 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(`.
  - **L2382 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(` 从当前函数返回。
- **L2383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `detail::gamma_incomplete_imp(static_cast<value_type>(a),`.
  - **L2383 CN**: 继续一个多行参数列表、初始化器或聚合项：`detail::gamma_incomplete_imp(static_cast<value_type>(a),`。
- **L2384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(z), true, true,`.
  - **L2384 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(z), true, true,`。
- **L2385 EN**: Executes a call or declaration centered on `forwarding_policy`.
  - **L2385 CN**: 执行以 `forwarding_policy` 为核心的调用或声明。
- **L2386 EN**: Closes the current lexical scope or compound statement.
  - **L2386 CN**: 结束当前词法作用域或复合语句块。
- **L2387 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2387 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2388 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2388 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2389 EN**: Continues logic associated with callable symbol `gamma_q`.
  - **L2389 CN**: 继续与可调用符号 `gamma_q` 相关的逻辑。
- **L2390 EN**: Opens a new lexical scope or compound statement.
  - **L2390 CN**: 打开一个新的词法作用域或复合语句块。
- **L2391 EN**: Returns from the current function with `gamma_q(a, z, policies::policy<>())`.
  - **L2391 CN**: 以 `gamma_q(a, z, policies::policy<>())` 从当前函数返回。
- **L2392 EN**: Closes the current lexical scope or compound statement.
  - **L2392 CN**: 结束当前词法作用域或复合语句块。
- **L2393 EN**: Separator comment used for visual grouping.
  - **L2393 CN**: 分隔注释，用于视觉分组。
- **L2394 EN**: Comment documents nearby intent or usage notes: `Regularised lower incomplete gamma:`.
  - **L2394 CN**: 注释说明附近代码的意图或使用说明：`Regularised lower incomplete gamma:`。
- **L2395 EN**: Separator comment used for visual grouping.
  - **L2395 CN**: 分隔注释，用于视觉分组。
- **L2396 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2396 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L2397 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2397 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2398 EN**: Continues logic associated with callable symbol `gamma_p`.
  - **L2398 CN**: 继续与可调用符号 `gamma_p` 相关的逻辑。
- **L2399 EN**: Opens a new lexical scope or compound statement.
  - **L2399 CN**: 打开一个新的词法作用域或复合语句块。
- **L2400 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2400 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2401 EN**: Introduces a legacy type alias or function typedef: `typedef tools::promote_args_t<T1, T2> result_type;`.
  - **L2401 CN**: 引入传统类型别名或函数 typedef：`typedef tools::promote_args_t<T1, T2> result_type;`。
- **L2402 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2402 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2403 EN**: Comment documents nearby intent or usage notes: `typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`.
  - **L2403 CN**: 注释说明附近代码的意图或使用说明：`typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;`。
- **L2404 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2404 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2405 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2406 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2407 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2408 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。

### Lines 2409-2436 / 第 2409-2436 行

````cpp
2409:       policies::assert_undefined<> >::type forwarding_policy;
2410: 
2411:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(
2412:       detail::gamma_incomplete_imp(static_cast<value_type>(a),
2413:       static_cast<value_type>(z), true, false,
2414:       forwarding_policy(), static_cast<value_type*>(nullptr)), "gamma_p<%1%>(%1%, %1%)");
2415: }
2416: template <class T1, class T2>
2417: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2418:    gamma_p(T1 a, T2 z)
2419: {
2420:    return gamma_p(a, z, policies::policy<>());
2421: }
2422: 
2423: // ratios of gamma functions:
2424: template <class T1, class T2, class Policy>
2425: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2426:    tgamma_delta_ratio(T1 z, T2 delta, const Policy& /* pol */)
2427: {
2428:    BOOST_FPU_EXCEPTION_GUARD
2429:    typedef tools::promote_args_t<T1, T2> result_type;
2430:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2431:    typedef typename policies::normalise<
2432:       Policy,
2433:       policies::promote_float<false>,
2434:       policies::promote_double<false>,
2435:       policies::discrete_quantile<>,
2436:       policies::assert_undefined<> >::type forwarding_policy;
````
- **L2409 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2409 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2410 EN**: Blank line separating nearby declarations or logic.
  - **L2410 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2411 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(`.
  - **L2411 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(` 从当前函数返回。
- **L2412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `detail::gamma_incomplete_imp(static_cast<value_type>(a),`.
  - **L2412 CN**: 继续一个多行参数列表、初始化器或聚合项：`detail::gamma_incomplete_imp(static_cast<value_type>(a),`。
- **L2413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(z), true, false,`.
  - **L2413 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(z), true, false,`。
- **L2414 EN**: Executes a call or declaration centered on `forwarding_policy`.
  - **L2414 CN**: 执行以 `forwarding_policy` 为核心的调用或声明。
- **L2415 EN**: Closes the current lexical scope or compound statement.
  - **L2415 CN**: 结束当前词法作用域或复合语句块。
- **L2416 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2416 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2417 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2417 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2418 EN**: Continues logic associated with callable symbol `gamma_p`.
  - **L2418 CN**: 继续与可调用符号 `gamma_p` 相关的逻辑。
- **L2419 EN**: Opens a new lexical scope or compound statement.
  - **L2419 CN**: 打开一个新的词法作用域或复合语句块。
- **L2420 EN**: Returns from the current function with `gamma_p(a, z, policies::policy<>())`.
  - **L2420 CN**: 以 `gamma_p(a, z, policies::policy<>())` 从当前函数返回。
- **L2421 EN**: Closes the current lexical scope or compound statement.
  - **L2421 CN**: 结束当前词法作用域或复合语句块。
- **L2422 EN**: Blank line separating nearby declarations or logic.
  - **L2422 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2423 EN**: Comment documents nearby intent or usage notes: `ratios of gamma functions:`.
  - **L2423 CN**: 注释说明附近代码的意图或使用说明：`ratios of gamma functions:`。
- **L2424 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2424 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L2425 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2425 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2426 EN**: Continues logic associated with callable symbol `tgamma_delta_ratio`.
  - **L2426 CN**: 继续与可调用符号 `tgamma_delta_ratio` 相关的逻辑。
- **L2427 EN**: Opens a new lexical scope or compound statement.
  - **L2427 CN**: 打开一个新的词法作用域或复合语句块。
- **L2428 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2428 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2429 EN**: Introduces a legacy type alias or function typedef: `typedef tools::promote_args_t<T1, T2> result_type;`.
  - **L2429 CN**: 引入传统类型别名或函数 typedef：`typedef tools::promote_args_t<T1, T2> result_type;`。
- **L2430 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2430 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2431 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2431 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2432 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2433 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2434 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2435 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2436 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2436 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。

### Lines 2437-2464 / 第 2437-2464 行

````cpp
2437: 
2438:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::tgamma_delta_ratio_imp(static_cast<value_type>(z), static_cast<value_type>(delta), forwarding_policy()), "boost::math::tgamma_delta_ratio<%1%>(%1%, %1%)");
2439: }
2440: template <class T1, class T2>
2441: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2442:    tgamma_delta_ratio(T1 z, T2 delta)
2443: {
2444:    return tgamma_delta_ratio(z, delta, policies::policy<>());
2445: }
2446: template <class T1, class T2, class Policy>
2447: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2448:    tgamma_ratio(T1 a, T2 b, const Policy&)
2449: {
2450:    typedef tools::promote_args_t<T1, T2> result_type;
2451:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2452:    typedef typename policies::normalise<
2453:       Policy,
2454:       policies::promote_float<false>,
2455:       policies::promote_double<false>,
2456:       policies::discrete_quantile<>,
2457:       policies::assert_undefined<> >::type forwarding_policy;
2458: 
2459:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::tgamma_ratio_imp(static_cast<value_type>(a), static_cast<value_type>(b), forwarding_policy()), "boost::math::tgamma_delta_ratio<%1%>(%1%, %1%)");
2460: }
2461: template <class T1, class T2>
2462: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2463:    tgamma_ratio(T1 a, T2 b)
2464: {
````
- **L2437 EN**: Blank line separating nearby declarations or logic.
  - **L2437 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2438 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::tgamma_delta_ratio_imp(static_cast<value_type>(z), static_cast<value_type>(delta), forwarding_policy()), "boost::math::tgamma_delta_ratio<%1%>(%1%, %1%)")`.
  - **L2438 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::tgamma_delta_ratio_imp(static_cast<value_type>(z), static_cast<value_type>(delta), forwarding_policy()), "boost::math::tgamma_delta_ratio<%1%>(%1%, %1%)")` 从当前函数返回。
- **L2439 EN**: Closes the current lexical scope or compound statement.
  - **L2439 CN**: 结束当前词法作用域或复合语句块。
- **L2440 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2440 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2441 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2441 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2442 EN**: Continues logic associated with callable symbol `tgamma_delta_ratio`.
  - **L2442 CN**: 继续与可调用符号 `tgamma_delta_ratio` 相关的逻辑。
- **L2443 EN**: Opens a new lexical scope or compound statement.
  - **L2443 CN**: 打开一个新的词法作用域或复合语句块。
- **L2444 EN**: Returns from the current function with `tgamma_delta_ratio(z, delta, policies::policy<>())`.
  - **L2444 CN**: 以 `tgamma_delta_ratio(z, delta, policies::policy<>())` 从当前函数返回。
- **L2445 EN**: Closes the current lexical scope or compound statement.
  - **L2445 CN**: 结束当前词法作用域或复合语句块。
- **L2446 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2446 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L2447 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2447 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2448 EN**: Continues logic associated with callable symbol `tgamma_ratio`.
  - **L2448 CN**: 继续与可调用符号 `tgamma_ratio` 相关的逻辑。
- **L2449 EN**: Opens a new lexical scope or compound statement.
  - **L2449 CN**: 打开一个新的词法作用域或复合语句块。
- **L2450 EN**: Introduces a legacy type alias or function typedef: `typedef tools::promote_args_t<T1, T2> result_type;`.
  - **L2450 CN**: 引入传统类型别名或函数 typedef：`typedef tools::promote_args_t<T1, T2> result_type;`。
- **L2451 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2451 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2452 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2452 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2453 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2454 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2455 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2456 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2457 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2457 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2458 EN**: Blank line separating nearby declarations or logic.
  - **L2458 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2459 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::tgamma_ratio_imp(static_cast<value_type>(a), static_cast<value_type>(b), forwarding_policy()), "boost::math::tgamma_delta_ratio<%1%>(%1%, %1%)")`.
  - **L2459 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::tgamma_ratio_imp(static_cast<value_type>(a), static_cast<value_type>(b), forwarding_policy()), "boost::math::tgamma_delta_ratio<%1%>(%1%, %1%)")` 从当前函数返回。
- **L2460 EN**: Closes the current lexical scope or compound statement.
  - **L2460 CN**: 结束当前词法作用域或复合语句块。
- **L2461 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2461 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2462 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2462 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2463 EN**: Continues logic associated with callable symbol `tgamma_ratio`.
  - **L2463 CN**: 继续与可调用符号 `tgamma_ratio` 相关的逻辑。
- **L2464 EN**: Opens a new lexical scope or compound statement.
  - **L2464 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 2465-2492 / 第 2465-2492 行

````cpp
2465:    return tgamma_ratio(a, b, policies::policy<>());
2466: }
2467: 
2468: template <class T1, class T2, class Policy>
2469: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2470:    gamma_p_derivative(T1 a, T2 x, const Policy&)
2471: {
2472:    BOOST_FPU_EXCEPTION_GUARD
2473:    typedef tools::promote_args_t<T1, T2> result_type;
2474:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
2475:    typedef typename policies::normalise<
2476:       Policy,
2477:       policies::promote_float<false>,
2478:       policies::promote_double<false>,
2479:       policies::discrete_quantile<>,
2480:       policies::assert_undefined<> >::type forwarding_policy;
2481: 
2482:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::gamma_p_derivative_imp(static_cast<value_type>(a), static_cast<value_type>(x), forwarding_policy()), "boost::math::gamma_p_derivative<%1%>(%1%, %1%)");
2483: }
2484: template <class T1, class T2>
2485: BOOST_MATH_GPU_ENABLED inline tools::promote_args_t<T1, T2>
2486:    gamma_p_derivative(T1 a, T2 x)
2487: {
2488:    return gamma_p_derivative(a, x, policies::policy<>());
2489: }
2490: 
2491: } // namespace math
2492: } // namespace boost
````
- **L2465 EN**: Returns from the current function with `tgamma_ratio(a, b, policies::policy<>())`.
  - **L2465 CN**: 以 `tgamma_ratio(a, b, policies::policy<>())` 从当前函数返回。
- **L2466 EN**: Closes the current lexical scope or compound statement.
  - **L2466 CN**: 结束当前词法作用域或复合语句块。
- **L2467 EN**: Blank line separating nearby declarations or logic.
  - **L2467 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2468 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L2468 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L2469 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2469 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2470 EN**: Continues logic associated with callable symbol `gamma_p_derivative`.
  - **L2470 CN**: 继续与可调用符号 `gamma_p_derivative` 相关的逻辑。
- **L2471 EN**: Opens a new lexical scope or compound statement.
  - **L2471 CN**: 打开一个新的词法作用域或复合语句块。
- **L2472 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L2472 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L2473 EN**: Introduces a legacy type alias or function typedef: `typedef tools::promote_args_t<T1, T2> result_type;`.
  - **L2473 CN**: 引入传统类型别名或函数 typedef：`typedef tools::promote_args_t<T1, T2> result_type;`。
- **L2474 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L2474 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L2475 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L2475 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L2476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L2476 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L2477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L2477 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L2478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L2478 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L2479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L2479 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L2480 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L2480 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L2481 EN**: Blank line separating nearby declarations or logic.
  - **L2481 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2482 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::gamma_p_derivative_imp(static_cast<value_type>(a), static_cast<value_type>(x), forwarding_policy()), "boost::math::gamma_p_derivative<%1%>(%1%, %1%)")`.
  - **L2482 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::gamma_p_derivative_imp(static_cast<value_type>(a), static_cast<value_type>(x), forwarding_policy()), "boost::math::gamma_p_derivative<%1%>(%1%, %1%)")` 从当前函数返回。
- **L2483 EN**: Closes the current lexical scope or compound statement.
  - **L2483 CN**: 结束当前词法作用域或复合语句块。
- **L2484 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L2484 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L2485 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L2485 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L2486 EN**: Continues logic associated with callable symbol `gamma_p_derivative`.
  - **L2486 CN**: 继续与可调用符号 `gamma_p_derivative` 相关的逻辑。
- **L2487 EN**: Opens a new lexical scope or compound statement.
  - **L2487 CN**: 打开一个新的词法作用域或复合语句块。
- **L2488 EN**: Returns from the current function with `gamma_p_derivative(a, x, policies::policy<>())`.
  - **L2488 CN**: 以 `gamma_p_derivative(a, x, policies::policy<>())` 从当前函数返回。
- **L2489 EN**: Closes the current lexical scope or compound statement.
  - **L2489 CN**: 结束当前词法作用域或复合语句块。
- **L2490 EN**: Blank line separating nearby declarations or logic.
  - **L2490 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2491 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace math`.
  - **L2491 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace math`。
- **L2492 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace boost`.
  - **L2492 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace boost`。

### Lines 2493-2502 / 第 2493-2502 行

````cpp
2493: 
2494: #ifdef _MSC_VER
2495: # pragma warning(pop)
2496: #endif
2497: 
2498: #include <boost/math/special_functions/detail/igamma_inverse.hpp>
2499: #include <boost/math/special_functions/detail/gamma_inva.hpp>
2500: #include <boost/math/special_functions/erf.hpp>
2501: 
2502: #endif // BOOST_MATH_SF_GAMMA_HPP
````
- **L2493 EN**: Blank line separating nearby declarations or logic.
  - **L2493 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2494 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L2494 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L2495 EN**: Continues logic associated with callable symbol `warning`.
  - **L2495 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L2496 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2496 CN**: 结束当前预处理条件块或头文件保护。
- **L2497 EN**: Blank line separating nearby declarations or logic.
  - **L2497 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2498 EN**: Includes <boost/math/special_functions/detail/igamma_inverse.hpp> to access Boost.Math special-function declarations.
  - **L2498 CN**: 引入 <boost/math/special_functions/detail/igamma_inverse.hpp> 以使用Boost.Math 特殊函数声明。
- **L2499 EN**: Includes <boost/math/special_functions/detail/gamma_inva.hpp> to access Boost.Math special-function declarations.
  - **L2499 CN**: 引入 <boost/math/special_functions/detail/gamma_inva.hpp> 以使用Boost.Math 特殊函数声明。
- **L2500 EN**: Includes <boost/math/special_functions/erf.hpp> to access Boost.Math special-function declarations.
  - **L2500 CN**: 引入 <boost/math/special_functions/erf.hpp> 以使用Boost.Math 特殊函数声明。
- **L2501 EN**: Blank line separating nearby declarations or logic.
  - **L2501 CN**: 空行，用于分隔相邻声明或逻辑。
- **L2502 EN**: Closes the current preprocessor conditional block or header guard.
  - **L2502 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Special-function numerics / 特殊函数数值计算**:
  - **EN**: Implements or exposes numerically stable special functions such as gamma, elliptic, or transcendental routines.
  - **CN**: 实现或暴露数值稳定的特殊函数，例如 gamma、椭圆积分或超越函数例程。
- **Template genericity / 模板泛型**:
  - **EN**: Uses C++ templates to adapt behavior across types, policies, or compile-time parameters.
  - **CN**: 使用 C++ 模板使行为适配不同类型、策略或编译期参数。
- **Policy customization / 策略定制**:
  - **EN**: Routes behavior through policy objects or compile-time policy choices.
  - **CN**: 通过策略对象或编译期策略选择来路由行为。
- **Type traits and metaprogramming / 类型 traits 与元编程**:
  - **EN**: Uses compile-time traits to select implementations and validate types.
  - **CN**: 使用编译期 traits 来选择实现并校验类型。
- **Assertions and contracts / 断言与契约**:
  - **EN**: Checks preconditions, postconditions, or test outcomes to keep behavior well defined.
  - **CN**: 检查前置条件、后置条件或测试结果，以保持行为有明确定义。
- **Advanced numeric formulas / 高级数值公式**:
  - **EN**: Implements carefully conditioned mathematical formulas, recurrences, or approximations.
  - **CN**: 实现经过精心条件化的数学公式、递推关系或近似算法。
- **Compile-time evaluation / 编译期求值**:
  - **EN**: Marks values or functions so some work can be performed during compilation.
  - **CN**: 标记值或函数，使部分工作可在编译期间完成。
- **Boost integration / Boost 集成**:
  - **EN**: Fits the implementation into Boost naming, configuration, and portability conventions.
  - **CN**: 让实现融入 Boost 的命名、配置与可移植性约定。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `boost/math/tools/config.hpp`, `boost/math/tools/series.hpp`, `boost/math/tools/fraction.hpp`, `boost/math/tools/precision.hpp`, `boost/math/tools/promotion.hpp`, `boost/math/tools/type_traits.hpp`, `boost/math/tools/numeric_limits.hpp`, `boost/math/tools/cstdint.hpp`, `boost/math/tools/assert.hpp`, `boost/math/policies/error_handling.hpp`, `boost/math/constants/constants.hpp`, `boost/math/special_functions/math_fwd.hpp` ... (+14 more)
- **Dependency categories / 依赖类别**: Boost.Math special-function declarations / Boost.Math 特殊函数声明 (15), Boost.Math numeric tool helpers / Boost.Math 数值工具辅助逻辑 (9), Boost.Math policy configuration / Boost.Math 策略配置 (1), Boost.Math numeric constants / Boost.Math 数值常量 (1)

- **EN**: `boost/math/tools/config.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/config.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/series.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/series.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/fraction.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/fraction.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/precision.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/precision.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/promotion.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/promotion.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/type_traits.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/type_traits.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/numeric_limits.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/numeric_limits.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/cstdint.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/cstdint.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/assert.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/assert.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/policies/error_handling.hpp` provides Boost.Math policy configuration.
  - **CN**: `boost/math/policies/error_handling.hpp` 提供Boost.Math 策略配置。
- **EN**: `boost/math/constants/constants.hpp` provides Boost.Math numeric constants.
  - **CN**: `boost/math/constants/constants.hpp` 提供Boost.Math 数值常量。
- **EN**: `boost/math/special_functions/math_fwd.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/math_fwd.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/log1p.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/log1p.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/trunc.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/trunc.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/powm1.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/powm1.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/sqrt1pm1.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/sqrt1pm1.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/lanczos.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/lanczos.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/fpclassify.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/fpclassify.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/detail/igamma_large.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/detail/igamma_large.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/detail/unchecked_factorial.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/detail/unchecked_factorial.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/detail/lgamma_small.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/detail/lgamma_small.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/bernoulli.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/bernoulli.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/polygamma.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/polygamma.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/detail/igamma_inverse.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/detail/igamma_inverse.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/detail/gamma_inva.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/detail/gamma_inva.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/erf.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/erf.hpp` 提供Boost.Math 特殊函数声明。
