# ellint_rj.hpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/ellint_rj.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/` (`llvm-project`)
- **Purpose / 目的**:
  - **EN**: History: XZ wrote the original of this file as part of the Google Summer of Code 2006. JM modified it to fit into the Boost.Math conceptual framework better, and to correctly handle the p < 0 case. Updated 2015 to.
  - **CN**: 声明 Boost.Math 的特殊函数、近似计算内核以及按类型泛化的数值入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

````cpp
   1: //  Copyright (c) 2006 Xiaogang Zhang, 2015 John Maddock
   2: //  Copyright (c) 2024 Matt Borland
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: //
   7: //  History:
   8: //  XZ wrote the original of this file as part of the Google
   9: //  Summer of Code 2006.  JM modified it to fit into the
  10: //  Boost.Math conceptual framework better, and to correctly
  11: //  handle the p < 0 case.
  12: //  Updated 2015 to use Carlson's latest methods.
  13: //
  14: 
  15: #ifndef BOOST_MATH_ELLINT_RJ_HPP
  16: #define BOOST_MATH_ELLINT_RJ_HPP
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
- **L6 EN**: Separator comment used for visual grouping.
  - **L6 CN**: 分隔注释，用于视觉分组。
- **L7 EN**: Comment documents nearby intent or usage notes: `History:`.
  - **L7 CN**: 注释说明附近代码的意图或使用说明：`History:`。
- **L8 EN**: Comment documents nearby intent or usage notes: `XZ wrote the original of this file as part of the Google`.
  - **L8 CN**: 注释说明附近代码的意图或使用说明：`XZ wrote the original of this file as part of the Google`。
- **L9 EN**: Comment documents nearby intent or usage notes: `Summer of Code 2006.  JM modified it to fit into the`.
  - **L9 CN**: 注释说明附近代码的意图或使用说明：`Summer of Code 2006.  JM modified it to fit into the`。
- **L10 EN**: Comment documents nearby intent or usage notes: `Boost.Math conceptual framework better, and to correctly`.
  - **L10 CN**: 注释说明附近代码的意图或使用说明：`Boost.Math conceptual framework better, and to correctly`。
- **L11 EN**: Comment documents nearby intent or usage notes: `handle the p < 0 case.`.
  - **L11 CN**: 注释说明附近代码的意图或使用说明：`handle the p < 0 case.`。
- **L12 EN**: Comment documents nearby intent or usage notes: `Updated 2015 to use Carlson's latest methods.`.
  - **L12 CN**: 注释说明附近代码的意图或使用说明：`Updated 2015 to use Carlson's latest methods.`。
- **L13 EN**: Separator comment used for visual grouping.
  - **L13 CN**: 分隔注释，用于视觉分组。
- **L14 EN**: Blank line separating nearby declarations or logic.
  - **L14 CN**: 空行，用于分隔相邻声明或逻辑。
- **L15 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_ELLINT_RJ_HPP`.
  - **L15 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_ELLINT_RJ_HPP`。
- **L16 EN**: Defines macro `BOOST_MATH_ELLINT_RJ_HPP` for compile-time control, shorthand, or generated boilerplate.
  - **L16 CN**: 定义宏 `BOOST_MATH_ELLINT_RJ_HPP`，用于编译期控制、简写或生成样板代码。

### Lines 17-32 / 第 17-32 行

````cpp
  17: 
  18: #ifdef _MSC_VER
  19: #pragma once
  20: #endif
  21: 
  22: #include <boost/math/tools/config.hpp>
  23: #include <boost/math/tools/numeric_limits.hpp>
  24: #include <boost/math/special_functions/math_fwd.hpp>
  25: #include <boost/math/policies/error_handling.hpp>
  26: #include <boost/math/special_functions/ellint_rc.hpp>
  27: #include <boost/math/special_functions/ellint_rf.hpp>
  28: #include <boost/math/special_functions/ellint_rd.hpp>
  29: 
  30: // Carlson's elliptic integral of the third kind
  31: // R_J(x, y, z, p) = 1.5 * \int_{0}^{\infty} (t+p)^{-1} [(t+x)(t+y)(t+z)]^{-1/2} dt
  32: // Carlson, Numerische Mathematik, vol 33, 1 (1979)
````
- **L17 EN**: Blank line separating nearby declarations or logic.
  - **L17 CN**: 空行，用于分隔相邻声明或逻辑。
- **L18 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L18 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L19 EN**: Uses `#pragma once` to prevent multiple inclusion of this header.
  - **L19 CN**: 使用 `#pragma once` 防止该头文件被重复包含。
- **L20 EN**: Closes the current preprocessor conditional block or header guard.
  - **L20 CN**: 结束当前预处理条件块或头文件保护。
- **L21 EN**: Blank line separating nearby declarations or logic.
  - **L21 CN**: 空行，用于分隔相邻声明或逻辑。
- **L22 EN**: Includes <boost/math/tools/config.hpp> to access Boost.Math numeric tool helpers.
  - **L22 CN**: 引入 <boost/math/tools/config.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L23 EN**: Includes <boost/math/tools/numeric_limits.hpp> to access Boost.Math numeric tool helpers.
  - **L23 CN**: 引入 <boost/math/tools/numeric_limits.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L24 EN**: Includes <boost/math/special_functions/math_fwd.hpp> to access Boost.Math special-function declarations.
  - **L24 CN**: 引入 <boost/math/special_functions/math_fwd.hpp> 以使用Boost.Math 特殊函数声明。
- **L25 EN**: Includes <boost/math/policies/error_handling.hpp> to access Boost.Math policy configuration.
  - **L25 CN**: 引入 <boost/math/policies/error_handling.hpp> 以使用Boost.Math 策略配置。
- **L26 EN**: Includes <boost/math/special_functions/ellint_rc.hpp> to access Boost.Math special-function declarations.
  - **L26 CN**: 引入 <boost/math/special_functions/ellint_rc.hpp> 以使用Boost.Math 特殊函数声明。
- **L27 EN**: Includes <boost/math/special_functions/ellint_rf.hpp> to access Boost.Math special-function declarations.
  - **L27 CN**: 引入 <boost/math/special_functions/ellint_rf.hpp> 以使用Boost.Math 特殊函数声明。
- **L28 EN**: Includes <boost/math/special_functions/ellint_rd.hpp> to access Boost.Math special-function declarations.
  - **L28 CN**: 引入 <boost/math/special_functions/ellint_rd.hpp> 以使用Boost.Math 特殊函数声明。
- **L29 EN**: Blank line separating nearby declarations or logic.
  - **L29 CN**: 空行，用于分隔相邻声明或逻辑。
- **L30 EN**: Comment documents nearby intent or usage notes: `Carlson's elliptic integral of the third kind`.
  - **L30 CN**: 注释说明附近代码的意图或使用说明：`Carlson's elliptic integral of the third kind`。
- **L31 EN**: Comment documents nearby intent or usage notes: `R_J(x, y, z, p) = 1.5 * \int_{0}^{\infty} (t+p)^{-1} [(t+x)(t+y)(t+z)]^{-1/2} dt`.
  - **L31 CN**: 注释说明附近代码的意图或使用说明：`R_J(x, y, z, p) = 1.5 * \int_{0}^{\infty} (t+p)^{-1} [(t+x)(t+y)(t+z)]^{-1/2} dt`。
- **L32 EN**: Comment documents nearby intent or usage notes: `Carlson, Numerische Mathematik, vol 33, 1 (1979)`.
  - **L32 CN**: 注释说明附近代码的意图或使用说明：`Carlson, Numerische Mathematik, vol 33, 1 (1979)`。

### Lines 33-48 / 第 33-48 行

````cpp
  33: 
  34: namespace boost { namespace math { namespace detail{
  35: 
  36: template <typename T, typename Policy>
  37: BOOST_MATH_GPU_ENABLED T ellint_rc1p_imp(T y, const Policy& pol)
  38: {
  39:    using namespace boost::math;
  40:    // Calculate RC(1, 1 + x)
  41:    BOOST_MATH_STD_USING
  42: 
  43:    BOOST_MATH_ASSERT(y != -1);
  44: 
  45:    // for 1 + y < 0, the integral is singular, return Cauchy principal value
  46:    T result;
  47:    if(y < -1)
  48:    {
````
- **L33 EN**: Blank line separating nearby declarations or logic.
  - **L33 CN**: 空行，用于分隔相邻声明或逻辑。
- **L34 EN**: Opens namespace scope `boost { namespace math { namespace detail`.
  - **L34 CN**: 打开命名空间作用域 `boost { namespace math { namespace detail`。
- **L35 EN**: Blank line separating nearby declarations or logic.
  - **L35 CN**: 空行，用于分隔相邻声明或逻辑。
- **L36 EN**: Introduces template parameters or specialization context: `template <typename T, typename Policy>`.
  - **L36 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T, typename Policy>`。
- **L37 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L37 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L38 EN**: Opens a new lexical scope or compound statement.
  - **L38 CN**: 打开一个新的词法作用域或复合语句块。
- **L39 EN**: Brings namespace `boost::math` into the local scope.
  - **L39 CN**: 将命名空间 `boost::math` 引入当前作用域。
- **L40 EN**: Comment documents nearby intent or usage notes: `Calculate RC(1, 1 + x)`.
  - **L40 CN**: 注释说明附近代码的意图或使用说明：`Calculate RC(1, 1 + x)`。
- **L41 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L41 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L42 EN**: Blank line separating nearby declarations or logic.
  - **L42 CN**: 空行，用于分隔相邻声明或逻辑。
- **L43 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L43 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L44 EN**: Blank line separating nearby declarations or logic.
  - **L44 CN**: 空行，用于分隔相邻声明或逻辑。
- **L45 EN**: Comment documents nearby intent or usage notes: `for 1 + y < 0, the integral is singular, return Cauchy principal value`.
  - **L45 CN**: 注释说明附近代码的意图或使用说明：`for 1 + y < 0, the integral is singular, return Cauchy principal value`。
- **L46 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L46 CN**: 执行一条独立语句或声明：`T result;`。
- **L47 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L47 CN**: 开始 `if` 控制流语句并计算其条件。
- **L48 EN**: Opens a new lexical scope or compound statement.
  - **L48 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 49-64 / 第 49-64 行

````cpp
  49:       result = sqrt(1 / -y) * detail::ellint_rc_imp(T(-y), T(-1 - y), pol);
  50:    }
  51:    else if(y == 0)
  52:    {
  53:       result = 1;
  54:    }
  55:    else if(y > 0)
  56:    {
  57:       result = atan(sqrt(y)) / sqrt(y);
  58:    }
  59:    else
  60:    {
  61:       if(y > T(-0.5))
  62:       {
  63:          T arg = sqrt(-y);
  64:          result = (boost::math::log1p(arg, pol) - boost::math::log1p(-arg, pol)) / (2 * sqrt(-y));
````
- **L49 EN**: Executes a call or declaration centered on `sqrt`.
  - **L49 CN**: 执行以 `sqrt` 为核心的调用或声明。
- **L50 EN**: Closes the current lexical scope or compound statement.
  - **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Starts the alternative branch of the preceding conditional.
  - **L51 CN**: 开始前一个条件语句的备选分支。
- **L52 EN**: Opens a new lexical scope or compound statement.
  - **L52 CN**: 打开一个新的词法作用域或复合语句块。
- **L53 EN**: Executes a standalone statement or declaration: `result = 1;`.
  - **L53 CN**: 执行一条独立语句或声明：`result = 1;`。
- **L54 EN**: Closes the current lexical scope or compound statement.
  - **L54 CN**: 结束当前词法作用域或复合语句块。
- **L55 EN**: Starts the alternative branch of the preceding conditional.
  - **L55 CN**: 开始前一个条件语句的备选分支。
- **L56 EN**: Opens a new lexical scope or compound statement.
  - **L56 CN**: 打开一个新的词法作用域或复合语句块。
- **L57 EN**: Executes a call or declaration centered on `atan`.
  - **L57 CN**: 执行以 `atan` 为核心的调用或声明。
- **L58 EN**: Closes the current lexical scope or compound statement.
  - **L58 CN**: 结束当前词法作用域或复合语句块。
- **L59 EN**: Starts the alternative branch of the preceding conditional.
  - **L59 CN**: 开始前一个条件语句的备选分支。
- **L60 EN**: Opens a new lexical scope or compound statement.
  - **L60 CN**: 打开一个新的词法作用域或复合语句块。
- **L61 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L61 CN**: 开始 `if` 控制流语句并计算其条件。
- **L62 EN**: Opens a new lexical scope or compound statement.
  - **L62 CN**: 打开一个新的词法作用域或复合语句块。
- **L63 EN**: Executes a call or declaration centered on `sqrt`.
  - **L63 CN**: 执行以 `sqrt` 为核心的调用或声明。
- **L64 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L64 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 65-80 / 第 65-80 行

````cpp
  65:       }
  66:       else
  67:       {
  68:          result = log((1 + sqrt(-y)) / sqrt(1 + y)) / sqrt(-y);
  69:       }
  70:    }
  71:    return result;
  72: }
  73: 
  74: template <typename T, typename Policy>
  75: BOOST_MATH_GPU_ENABLED T ellint_rj_imp_final(T x, T y, T z, T p, const Policy& pol)
  76: {
  77:    BOOST_MATH_STD_USING
  78: 
  79:    //
  80:    // Special cases from http://dlmf.nist.gov/19.20#iii
````
- **L65 EN**: Closes the current lexical scope or compound statement.
  - **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Starts the alternative branch of the preceding conditional.
  - **L66 CN**: 开始前一个条件语句的备选分支。
- **L67 EN**: Opens a new lexical scope or compound statement.
  - **L67 CN**: 打开一个新的词法作用域或复合语句块。
- **L68 EN**: Executes a call or declaration centered on `log`.
  - **L68 CN**: 执行以 `log` 为核心的调用或声明。
- **L69 EN**: Closes the current lexical scope or compound statement.
  - **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Closes the current lexical scope or compound statement.
  - **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Returns from the current function with `result`.
  - **L71 CN**: 以 `result` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  - **L72 CN**: 结束当前词法作用域或复合语句块。
- **L73 EN**: Blank line separating nearby declarations or logic.
  - **L73 CN**: 空行，用于分隔相邻声明或逻辑。
- **L74 EN**: Introduces template parameters or specialization context: `template <typename T, typename Policy>`.
  - **L74 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T, typename Policy>`。
- **L75 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L75 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L76 EN**: Opens a new lexical scope or compound statement.
  - **L76 CN**: 打开一个新的词法作用域或复合语句块。
- **L77 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L77 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L78 EN**: Blank line separating nearby declarations or logic.
  - **L78 CN**: 空行，用于分隔相邻声明或逻辑。
- **L79 EN**: Separator comment used for visual grouping.
  - **L79 CN**: 分隔注释，用于视觉分组。
- **L80 EN**: Comment documents nearby intent or usage notes: `Special cases from http://dlmf.nist.gov/19.20#iii`.
  - **L80 CN**: 注释说明附近代码的意图或使用说明：`Special cases from http://dlmf.nist.gov/19.20#iii`。

### Lines 81-96 / 第 81-96 行

````cpp
  81:    //
  82:    if(x == y)
  83:    {
  84:       if(x == z)
  85:       {
  86:          if(x == p)
  87:          {
  88:             // All values equal:
  89:             return 1 / (x * sqrt(x));
  90:          }
  91:          else
  92:          {
  93:             // x = y = z:
  94:             return 3 * (ellint_rc_imp(x, p, pol) - 1 / sqrt(x)) / (x - p);
  95:          }
  96:       }
````
- **L81 EN**: Separator comment used for visual grouping.
  - **L81 CN**: 分隔注释，用于视觉分组。
- **L82 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L82 CN**: 开始 `if` 控制流语句并计算其条件。
- **L83 EN**: Opens a new lexical scope or compound statement.
  - **L83 CN**: 打开一个新的词法作用域或复合语句块。
- **L84 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L84 CN**: 开始 `if` 控制流语句并计算其条件。
- **L85 EN**: Opens a new lexical scope or compound statement.
  - **L85 CN**: 打开一个新的词法作用域或复合语句块。
- **L86 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L86 CN**: 开始 `if` 控制流语句并计算其条件。
- **L87 EN**: Opens a new lexical scope or compound statement.
  - **L87 CN**: 打开一个新的词法作用域或复合语句块。
- **L88 EN**: Comment documents nearby intent or usage notes: `All values equal:`.
  - **L88 CN**: 注释说明附近代码的意图或使用说明：`All values equal:`。
- **L89 EN**: Returns from the current function with `1 / (x * sqrt(x))`.
  - **L89 CN**: 以 `1 / (x * sqrt(x))` 从当前函数返回。
- **L90 EN**: Closes the current lexical scope or compound statement.
  - **L90 CN**: 结束当前词法作用域或复合语句块。
- **L91 EN**: Starts the alternative branch of the preceding conditional.
  - **L91 CN**: 开始前一个条件语句的备选分支。
- **L92 EN**: Opens a new lexical scope or compound statement.
  - **L92 CN**: 打开一个新的词法作用域或复合语句块。
- **L93 EN**: Comment documents nearby intent or usage notes: `x = y = z:`.
  - **L93 CN**: 注释说明附近代码的意图或使用说明：`x = y = z:`。
- **L94 EN**: Returns from the current function with `3 * (ellint_rc_imp(x, p, pol) - 1 / sqrt(x)) / (x - p)`.
  - **L94 CN**: 以 `3 * (ellint_rc_imp(x, p, pol) - 1 / sqrt(x)) / (x - p)` 从当前函数返回。
- **L95 EN**: Closes the current lexical scope or compound statement.
  - **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Closes the current lexical scope or compound statement.
  - **L96 CN**: 结束当前词法作用域或复合语句块。

### Lines 97-112 / 第 97-112 行

````cpp
  97:       else
  98:       {
  99:          // x = y only, permute so y = z:
 100:          BOOST_MATH_GPU_SAFE_SWAP(x, z);
 101:          if(y == p)
 102:          {
 103:             return ellint_rd_imp(x, y, y, pol);
 104:          }
 105:          else if(BOOST_MATH_GPU_SAFE_MAX(y, p) / BOOST_MATH_GPU_SAFE_MIN(y, p) > T(1.2))
 106:          {
 107:             return 3 * (ellint_rc_imp(x, y, pol) - ellint_rc_imp(x, p, pol)) / (p - y);
 108:          }
 109:          // Otherwise fall through to normal method, special case above will suffer too much cancellation...
 110:       }
 111:    }
 112:    if(y == z)
````
- **L97 EN**: Starts the alternative branch of the preceding conditional.
  - **L97 CN**: 开始前一个条件语句的备选分支。
- **L98 EN**: Opens a new lexical scope or compound statement.
  - **L98 CN**: 打开一个新的词法作用域或复合语句块。
- **L99 EN**: Comment documents nearby intent or usage notes: `x = y only, permute so y = z:`.
  - **L99 CN**: 注释说明附近代码的意图或使用说明：`x = y only, permute so y = z:`。
- **L100 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L100 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L101 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L101 CN**: 开始 `if` 控制流语句并计算其条件。
- **L102 EN**: Opens a new lexical scope or compound statement.
  - **L102 CN**: 打开一个新的词法作用域或复合语句块。
- **L103 EN**: Returns from the current function with `ellint_rd_imp(x, y, y, pol)`.
  - **L103 CN**: 以 `ellint_rd_imp(x, y, y, pol)` 从当前函数返回。
- **L104 EN**: Closes the current lexical scope or compound statement.
  - **L104 CN**: 结束当前词法作用域或复合语句块。
- **L105 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L105 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L106 EN**: Opens a new lexical scope or compound statement.
  - **L106 CN**: 打开一个新的词法作用域或复合语句块。
- **L107 EN**: Returns from the current function with `3 * (ellint_rc_imp(x, y, pol) - ellint_rc_imp(x, p, pol)) / (p - y)`.
  - **L107 CN**: 以 `3 * (ellint_rc_imp(x, y, pol) - ellint_rc_imp(x, p, pol)) / (p - y)` 从当前函数返回。
- **L108 EN**: Closes the current lexical scope or compound statement.
  - **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Comment documents nearby intent or usage notes: `Otherwise fall through to normal method, special case above will suffer too much cancellation...`.
  - **L109 CN**: 注释说明附近代码的意图或使用说明：`Otherwise fall through to normal method, special case above will suffer too much cancellation...`。
- **L110 EN**: Closes the current lexical scope or compound statement.
  - **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Closes the current lexical scope or compound statement.
  - **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L112 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 113-128 / 第 113-128 行

````cpp
 113:    {
 114:       if(y == p)
 115:       {
 116:          // y = z = p:
 117:          return ellint_rd_imp(x, y, y, pol);
 118:       }
 119:       else if(BOOST_MATH_GPU_SAFE_MAX(y, p) / BOOST_MATH_GPU_SAFE_MIN(y, p) > T(1.2))
 120:       {
 121:          // y = z:
 122:          return 3 * (ellint_rc_imp(x, y, pol) - ellint_rc_imp(x, p, pol)) / (p - y);
 123:       }
 124:       // Otherwise fall through to normal method, special case above will suffer too much cancellation...
 125:    }
 126:    if(z == p)
 127:    {
 128:       return ellint_rd_imp(x, y, z, pol);
````
- **L113 EN**: Opens a new lexical scope or compound statement.
  - **L113 CN**: 打开一个新的词法作用域或复合语句块。
- **L114 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L114 CN**: 开始 `if` 控制流语句并计算其条件。
- **L115 EN**: Opens a new lexical scope or compound statement.
  - **L115 CN**: 打开一个新的词法作用域或复合语句块。
- **L116 EN**: Comment documents nearby intent or usage notes: `y = z = p:`.
  - **L116 CN**: 注释说明附近代码的意图或使用说明：`y = z = p:`。
- **L117 EN**: Returns from the current function with `ellint_rd_imp(x, y, y, pol)`.
  - **L117 CN**: 以 `ellint_rd_imp(x, y, y, pol)` 从当前函数返回。
- **L118 EN**: Closes the current lexical scope or compound statement.
  - **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L119 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L120 EN**: Opens a new lexical scope or compound statement.
  - **L120 CN**: 打开一个新的词法作用域或复合语句块。
- **L121 EN**: Comment documents nearby intent or usage notes: `y = z:`.
  - **L121 CN**: 注释说明附近代码的意图或使用说明：`y = z:`。
- **L122 EN**: Returns from the current function with `3 * (ellint_rc_imp(x, y, pol) - ellint_rc_imp(x, p, pol)) / (p - y)`.
  - **L122 CN**: 以 `3 * (ellint_rc_imp(x, y, pol) - ellint_rc_imp(x, p, pol)) / (p - y)` 从当前函数返回。
- **L123 EN**: Closes the current lexical scope or compound statement.
  - **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Comment documents nearby intent or usage notes: `Otherwise fall through to normal method, special case above will suffer too much cancellation...`.
  - **L124 CN**: 注释说明附近代码的意图或使用说明：`Otherwise fall through to normal method, special case above will suffer too much cancellation...`。
- **L125 EN**: Closes the current lexical scope or compound statement.
  - **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L126 CN**: 开始 `if` 控制流语句并计算其条件。
- **L127 EN**: Opens a new lexical scope or compound statement.
  - **L127 CN**: 打开一个新的词法作用域或复合语句块。
- **L128 EN**: Returns from the current function with `ellint_rd_imp(x, y, z, pol)`.
  - **L128 CN**: 以 `ellint_rd_imp(x, y, z, pol)` 从当前函数返回。

### Lines 129-144 / 第 129-144 行

````cpp
 129:    }
 130: 
 131:    T xn = x;
 132:    T yn = y;
 133:    T zn = z;
 134:    T pn = p;
 135:    T An = (x + y + z + 2 * p) / 5;
 136:    T A0 = An;
 137:    T delta = (p - x) * (p - y) * (p - z);
 138:    T Q = pow(tools::epsilon<T>() / 5, -T(1) / 8) * BOOST_MATH_GPU_SAFE_MAX(BOOST_MATH_GPU_SAFE_MAX(fabs(An - x), fabs(An - y)), BOOST_MATH_GPU_SAFE_MAX(fabs(An - z), fabs(An - p)));
 139: 
 140:    unsigned n;
 141:    T lambda;
 142:    T Dn;
 143:    T En;
 144:    T rx, ry, rz, rp;
````
- **L129 EN**: Closes the current lexical scope or compound statement.
  - **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Blank line separating nearby declarations or logic.
  - **L130 CN**: 空行，用于分隔相邻声明或逻辑。
- **L131 EN**: Executes a standalone statement or declaration: `T xn = x;`.
  - **L131 CN**: 执行一条独立语句或声明：`T xn = x;`。
- **L132 EN**: Executes a standalone statement or declaration: `T yn = y;`.
  - **L132 CN**: 执行一条独立语句或声明：`T yn = y;`。
- **L133 EN**: Executes a standalone statement or declaration: `T zn = z;`.
  - **L133 CN**: 执行一条独立语句或声明：`T zn = z;`。
- **L134 EN**: Executes a standalone statement or declaration: `T pn = p;`.
  - **L134 CN**: 执行一条独立语句或声明：`T pn = p;`。
- **L135 EN**: Executes a call or declaration centered on `=`.
  - **L135 CN**: 执行以 `=` 为核心的调用或声明。
- **L136 EN**: Executes a standalone statement or declaration: `T A0 = An;`.
  - **L136 CN**: 执行一条独立语句或声明：`T A0 = An;`。
- **L137 EN**: Executes a call or declaration centered on `=`.
  - **L137 CN**: 执行以 `=` 为核心的调用或声明。
- **L138 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L138 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L139 EN**: Blank line separating nearby declarations or logic.
  - **L139 CN**: 空行，用于分隔相邻声明或逻辑。
- **L140 EN**: Executes a standalone statement or declaration: `unsigned n;`.
  - **L140 CN**: 执行一条独立语句或声明：`unsigned n;`。
- **L141 EN**: Executes a standalone statement or declaration: `T lambda;`.
  - **L141 CN**: 执行一条独立语句或声明：`T lambda;`。
- **L142 EN**: Executes a standalone statement or declaration: `T Dn;`.
  - **L142 CN**: 执行一条独立语句或声明：`T Dn;`。
- **L143 EN**: Executes a standalone statement or declaration: `T En;`.
  - **L143 CN**: 执行一条独立语句或声明：`T En;`。
- **L144 EN**: Executes a standalone statement or declaration: `T rx, ry, rz, rp;`.
  - **L144 CN**: 执行一条独立语句或声明：`T rx, ry, rz, rp;`。

### Lines 145-160 / 第 145-160 行

````cpp
 145:    T fmn = 1; // 4^-n
 146:    T RC_sum = 0;
 147: 
 148:    for(n = 0; n < policies::get_max_series_iterations<Policy>(); ++n)
 149:    {
 150:       rx = sqrt(xn);
 151:       ry = sqrt(yn);
 152:       rz = sqrt(zn);
 153:       rp = sqrt(pn);
 154:       Dn = (rp + rx) * (rp + ry) * (rp + rz);
 155:       En = delta / Dn;
 156:       En /= Dn;
 157:       if((En < T(-0.5)) && (En > T(-1.5)))
 158:       {
 159:          //
 160:          // Occasionally En ~ -1, we then have no means of calculating
````
- **L145 EN**: Continues the surrounding expression or declaration: `T fmn = 1; // 4^-n`.
  - **L145 CN**: 继续构造周围的表达式或声明：`T fmn = 1; // 4^-n`。
- **L146 EN**: Executes a standalone statement or declaration: `T RC_sum = 0;`.
  - **L146 CN**: 执行一条独立语句或声明：`T RC_sum = 0;`。
- **L147 EN**: Blank line separating nearby declarations or logic.
  - **L147 CN**: 空行，用于分隔相邻声明或逻辑。
- **L148 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L148 CN**: 开始 `for` 控制流语句并计算其条件。
- **L149 EN**: Opens a new lexical scope or compound statement.
  - **L149 CN**: 打开一个新的词法作用域或复合语句块。
- **L150 EN**: Executes a call or declaration centered on `sqrt`.
  - **L150 CN**: 执行以 `sqrt` 为核心的调用或声明。
- **L151 EN**: Executes a call or declaration centered on `sqrt`.
  - **L151 CN**: 执行以 `sqrt` 为核心的调用或声明。
- **L152 EN**: Executes a call or declaration centered on `sqrt`.
  - **L152 CN**: 执行以 `sqrt` 为核心的调用或声明。
- **L153 EN**: Executes a call or declaration centered on `sqrt`.
  - **L153 CN**: 执行以 `sqrt` 为核心的调用或声明。
- **L154 EN**: Executes a call or declaration centered on `=`.
  - **L154 CN**: 执行以 `=` 为核心的调用或声明。
- **L155 EN**: Executes a standalone statement or declaration: `En = delta / Dn;`.
  - **L155 CN**: 执行一条独立语句或声明：`En = delta / Dn;`。
- **L156 EN**: Executes a standalone statement or declaration: `En /= Dn;`.
  - **L156 CN**: 执行一条独立语句或声明：`En /= Dn;`。
- **L157 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L157 CN**: 开始 `if` 控制流语句并计算其条件。
- **L158 EN**: Opens a new lexical scope or compound statement.
  - **L158 CN**: 打开一个新的词法作用域或复合语句块。
- **L159 EN**: Separator comment used for visual grouping.
  - **L159 CN**: 分隔注释，用于视觉分组。
- **L160 EN**: Comment documents nearby intent or usage notes: `Occasionally En ~ -1, we then have no means of calculating`.
  - **L160 CN**: 注释说明附近代码的意图或使用说明：`Occasionally En ~ -1, we then have no means of calculating`。

### Lines 161-176 / 第 161-176 行

````cpp
 161:          // RC(1, 1+En) without terrible cancellation error, so we
 162:          // need to get to 1+En directly.  By substitution we have
 163:          //
 164:          // 1+E_0 = 1 + (p-x)*(p-y)*(p-z)/((sqrt(p) + sqrt(x))*(sqrt(p)+sqrt(y))*(sqrt(p)+sqrt(z)))^2
 165:          //       = 2*sqrt(p)*(p+sqrt(x) * (sqrt(y)+sqrt(z)) + sqrt(y)*sqrt(z)) / ((sqrt(p) + sqrt(x))*(sqrt(p) + sqrt(y)*(sqrt(p)+sqrt(z))))
 166:          //
 167:          // And since this is just an application of the duplication formula for RJ, the same
 168:          // expression works for 1+En if we use x,y,z,p_n etc.
 169:          // This branch is taken only once or twice at the start of iteration,
 170:          // after than En reverts to it's usual very small values.
 171:          //
 172:          T b = 2 * rp * (pn + rx * (ry + rz) + ry * rz) / Dn;
 173:          RC_sum += fmn / Dn * detail::ellint_rc_imp(T(1), b, pol);
 174:       }
 175:       else
 176:       {
````
- **L161 EN**: Comment documents nearby intent or usage notes: `RC(1, 1+En) without terrible cancellation error, so we`.
  - **L161 CN**: 注释说明附近代码的意图或使用说明：`RC(1, 1+En) without terrible cancellation error, so we`。
- **L162 EN**: Comment documents nearby intent or usage notes: `need to get to 1+En directly.  By substitution we have`.
  - **L162 CN**: 注释说明附近代码的意图或使用说明：`need to get to 1+En directly.  By substitution we have`。
- **L163 EN**: Separator comment used for visual grouping.
  - **L163 CN**: 分隔注释，用于视觉分组。
- **L164 EN**: Comment documents nearby intent or usage notes: `1+E_0 = 1 + (p-x)*(p-y)*(p-z)/((sqrt(p) + sqrt(x))*(sqrt(p)+sqrt(y))*(sqrt(p)+sqrt(z)))^2`.
  - **L164 CN**: 注释说明附近代码的意图或使用说明：`1+E_0 = 1 + (p-x)*(p-y)*(p-z)/((sqrt(p) + sqrt(x))*(sqrt(p)+sqrt(y))*(sqrt(p)+sqrt(z)))^2`。
- **L165 EN**: Comment documents nearby intent or usage notes: `= 2*sqrt(p)*(p+sqrt(x) * (sqrt(y)+sqrt(z)) + sqrt(y)*sqrt(z)) / ((sqrt(p) + sqrt(x))*(sqrt(p) + sqrt(y)*(sqrt(p)+sqrt(z))))`.
  - **L165 CN**: 注释说明附近代码的意图或使用说明：`= 2*sqrt(p)*(p+sqrt(x) * (sqrt(y)+sqrt(z)) + sqrt(y)*sqrt(z)) / ((sqrt(p) + sqrt(x))*(sqrt(p) + sqrt(y)*(sqrt(p)+sqrt(z))))`。
- **L166 EN**: Separator comment used for visual grouping.
  - **L166 CN**: 分隔注释，用于视觉分组。
- **L167 EN**: Comment documents nearby intent or usage notes: `And since this is just an application of the duplication formula for RJ, the same`.
  - **L167 CN**: 注释说明附近代码的意图或使用说明：`And since this is just an application of the duplication formula for RJ, the same`。
- **L168 EN**: Comment documents nearby intent or usage notes: `expression works for 1+En if we use x,y,z,p_n etc.`.
  - **L168 CN**: 注释说明附近代码的意图或使用说明：`expression works for 1+En if we use x,y,z,p_n etc.`。
- **L169 EN**: Comment documents nearby intent or usage notes: `This branch is taken only once or twice at the start of iteration,`.
  - **L169 CN**: 注释说明附近代码的意图或使用说明：`This branch is taken only once or twice at the start of iteration,`。
- **L170 EN**: Comment documents nearby intent or usage notes: `after than En reverts to it's usual very small values.`.
  - **L170 CN**: 注释说明附近代码的意图或使用说明：`after than En reverts to it's usual very small values.`。
- **L171 EN**: Separator comment used for visual grouping.
  - **L171 CN**: 分隔注释，用于视觉分组。
- **L172 EN**: Executes a call or declaration centered on `*`.
  - **L172 CN**: 执行以 `*` 为核心的调用或声明。
- **L173 EN**: Executes a call or declaration centered on `detail::ellint_rc_imp`.
  - **L173 CN**: 执行以 `detail::ellint_rc_imp` 为核心的调用或声明。
- **L174 EN**: Closes the current lexical scope or compound statement.
  - **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Starts the alternative branch of the preceding conditional.
  - **L175 CN**: 开始前一个条件语句的备选分支。
- **L176 EN**: Opens a new lexical scope or compound statement.
  - **L176 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 177-192 / 第 177-192 行

````cpp
 177:          RC_sum += fmn / Dn * ellint_rc1p_imp(En, pol);
 178:       }
 179:       lambda = rx * ry + rx * rz + ry * rz;
 180: 
 181:       // From here on we move to n+1:
 182:       An = (An + lambda) / 4;
 183:       fmn /= 4;
 184: 
 185:       if(fmn * Q < An)
 186:          break;
 187: 
 188:       xn = (xn + lambda) / 4;
 189:       yn = (yn + lambda) / 4;
 190:       zn = (zn + lambda) / 4;
 191:       pn = (pn + lambda) / 4;
 192:       delta /= 64;
````
- **L177 EN**: Executes a call or declaration centered on `ellint_rc1p_imp`.
  - **L177 CN**: 执行以 `ellint_rc1p_imp` 为核心的调用或声明。
- **L178 EN**: Closes the current lexical scope or compound statement.
  - **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Executes a standalone statement or declaration: `lambda = rx * ry + rx * rz + ry * rz;`.
  - **L179 CN**: 执行一条独立语句或声明：`lambda = rx * ry + rx * rz + ry * rz;`。
- **L180 EN**: Blank line separating nearby declarations or logic.
  - **L180 CN**: 空行，用于分隔相邻声明或逻辑。
- **L181 EN**: Comment documents nearby intent or usage notes: `From here on we move to n+1:`.
  - **L181 CN**: 注释说明附近代码的意图或使用说明：`From here on we move to n+1:`。
- **L182 EN**: Executes a call or declaration centered on `=`.
  - **L182 CN**: 执行以 `=` 为核心的调用或声明。
- **L183 EN**: Executes a standalone statement or declaration: `fmn /= 4;`.
  - **L183 CN**: 执行一条独立语句或声明：`fmn /= 4;`。
- **L184 EN**: Blank line separating nearby declarations or logic.
  - **L184 CN**: 空行，用于分隔相邻声明或逻辑。
- **L185 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L185 CN**: 开始 `if` 控制流语句并计算其条件。
- **L186 EN**: Exits the nearest loop or switch statement.
  - **L186 CN**: 退出最近的循环或 switch 语句。
- **L187 EN**: Blank line separating nearby declarations or logic.
  - **L187 CN**: 空行，用于分隔相邻声明或逻辑。
- **L188 EN**: Executes a call or declaration centered on `=`.
  - **L188 CN**: 执行以 `=` 为核心的调用或声明。
- **L189 EN**: Executes a call or declaration centered on `=`.
  - **L189 CN**: 执行以 `=` 为核心的调用或声明。
- **L190 EN**: Executes a call or declaration centered on `=`.
  - **L190 CN**: 执行以 `=` 为核心的调用或声明。
- **L191 EN**: Executes a call or declaration centered on `=`.
  - **L191 CN**: 执行以 `=` 为核心的调用或声明。
- **L192 EN**: Executes a standalone statement or declaration: `delta /= 64;`.
  - **L192 CN**: 执行一条独立语句或声明：`delta /= 64;`。

### Lines 193-208 / 第 193-208 行

````cpp
 193:    }
 194: 
 195:    T X = fmn * (A0 - x) / An;
 196:    T Y = fmn * (A0 - y) / An;
 197:    T Z = fmn * (A0 - z) / An;
 198:    T P = (-X - Y - Z) / 2;
 199:    T E2 = X * Y + X * Z + Y * Z - 3 * P * P;
 200:    T E3 = X * Y * Z + 2 * E2 * P + 4 * P * P * P;
 201:    T E4 = (2 * X * Y * Z + E2 * P + 3 * P * P * P) * P;
 202:    T E5 = X * Y * Z * P * P;
 203:    T result = fmn * pow(An, T(-3) / 2) *
 204:       (1 - 3 * E2 / 14 + E3 / 6 + 9 * E2 * E2 / 88 - 3 * E4 / 22 - 9 * E2 * E3 / 52 + 3 * E5 / 26 - E2 * E2 * E2 / 16
 205:       + 3 * E3 * E3 / 40 + 3 * E2 * E4 / 20 + 45 * E2 * E2 * E3 / 272 - 9 * (E3 * E4 + E2 * E5) / 68);
 206: 
 207:    result += 6 * RC_sum;
 208:    return result;
````
- **L193 EN**: Closes the current lexical scope or compound statement.
  - **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic.
  - **L194 CN**: 空行，用于分隔相邻声明或逻辑。
- **L195 EN**: Executes a call or declaration centered on `*`.
  - **L195 CN**: 执行以 `*` 为核心的调用或声明。
- **L196 EN**: Executes a call or declaration centered on `*`.
  - **L196 CN**: 执行以 `*` 为核心的调用或声明。
- **L197 EN**: Executes a call or declaration centered on `*`.
  - **L197 CN**: 执行以 `*` 为核心的调用或声明。
- **L198 EN**: Executes a call or declaration centered on `=`.
  - **L198 CN**: 执行以 `=` 为核心的调用或声明。
- **L199 EN**: Executes a standalone statement or declaration: `T E2 = X * Y + X * Z + Y * Z - 3 * P * P;`.
  - **L199 CN**: 执行一条独立语句或声明：`T E2 = X * Y + X * Z + Y * Z - 3 * P * P;`。
- **L200 EN**: Executes a standalone statement or declaration: `T E3 = X * Y * Z + 2 * E2 * P + 4 * P * P * P;`.
  - **L200 CN**: 执行一条独立语句或声明：`T E3 = X * Y * Z + 2 * E2 * P + 4 * P * P * P;`。
- **L201 EN**: Executes a call or declaration centered on `=`.
  - **L201 CN**: 执行以 `=` 为核心的调用或声明。
- **L202 EN**: Executes a standalone statement or declaration: `T E5 = X * Y * Z * P * P;`.
  - **L202 CN**: 执行一条独立语句或声明：`T E5 = X * Y * Z * P * P;`。
- **L203 EN**: Continues logic associated with callable symbol `pow`.
  - **L203 CN**: 继续与可调用符号 `pow` 相关的逻辑。
- **L204 EN**: Continues the surrounding expression or declaration: `(1 - 3 * E2 / 14 + E3 / 6 + 9 * E2 * E2 / 88 - 3 * E4 / 22 - 9 * E2 * E3 / 52 + 3 * E5 / 26 - E2 * E2 * E2 / 16`.
  - **L204 CN**: 继续构造周围的表达式或声明：`(1 - 3 * E2 / 14 + E3 / 6 + 9 * E2 * E2 / 88 - 3 * E4 / 22 - 9 * E2 * E3 / 52 + 3 * E5 / 26 - E2 * E2 * E2 / 16`。
- **L205 EN**: Executes a call or declaration centered on `*`.
  - **L205 CN**: 执行以 `*` 为核心的调用或声明。
- **L206 EN**: Blank line separating nearby declarations or logic.
  - **L206 CN**: 空行，用于分隔相邻声明或逻辑。
- **L207 EN**: Executes a standalone statement or declaration: `result += 6 * RC_sum;`.
  - **L207 CN**: 执行一条独立语句或声明：`result += 6 * RC_sum;`。
- **L208 EN**: Returns from the current function with `result`.
  - **L208 CN**: 以 `result` 从当前函数返回。

### Lines 209-224 / 第 209-224 行

````cpp
 209: }
 210: 
 211: template <typename T, typename Policy>
 212: BOOST_MATH_GPU_ENABLED T ellint_rj_imp(T x, T y, T z, T p, const Policy& pol)
 213: {
 214:    BOOST_MATH_STD_USING
 215:    
 216:    constexpr auto function = "boost::math::ellint_rj<%1%>(%1%,%1%,%1%)";
 217: 
 218:    if(x < 0)
 219:    {
 220:       return policies::raise_domain_error<T>(function, "Argument x must be non-negative, but got x = %1%", x, pol);
 221:    }
 222:    if(y < 0)
 223:    {
 224:       return policies::raise_domain_error<T>(function, "Argument y must be non-negative, but got y = %1%", y, pol);
````
- **L209 EN**: Closes the current lexical scope or compound statement.
  - **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Blank line separating nearby declarations or logic.
  - **L210 CN**: 空行，用于分隔相邻声明或逻辑。
- **L211 EN**: Introduces template parameters or specialization context: `template <typename T, typename Policy>`.
  - **L211 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T, typename Policy>`。
- **L212 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L212 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L213 EN**: Opens a new lexical scope or compound statement.
  - **L213 CN**: 打开一个新的词法作用域或复合语句块。
- **L214 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L214 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L215 EN**: Blank line separating nearby declarations or logic.
  - **L215 CN**: 空行，用于分隔相邻声明或逻辑。
- **L216 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L216 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L217 EN**: Blank line separating nearby declarations or logic.
  - **L217 CN**: 空行，用于分隔相邻声明或逻辑。
- **L218 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L218 CN**: 开始 `if` 控制流语句并计算其条件。
- **L219 EN**: Opens a new lexical scope or compound statement.
  - **L219 CN**: 打开一个新的词法作用域或复合语句块。
- **L220 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Argument x must be non-negative, but got x = %1%", x, pol)`.
  - **L220 CN**: 以 `policies::raise_domain_error<T>(function, "Argument x must be non-negative, but got x = %1%", x, pol)` 从当前函数返回。
- **L221 EN**: Closes the current lexical scope or compound statement.
  - **L221 CN**: 结束当前词法作用域或复合语句块。
- **L222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L223 EN**: Opens a new lexical scope or compound statement.
  - **L223 CN**: 打开一个新的词法作用域或复合语句块。
- **L224 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Argument y must be non-negative, but got y = %1%", y, pol)`.
  - **L224 CN**: 以 `policies::raise_domain_error<T>(function, "Argument y must be non-negative, but got y = %1%", y, pol)` 从当前函数返回。

### Lines 225-240 / 第 225-240 行

````cpp
 225:    }
 226:    if(z < 0)
 227:    {
 228:       return policies::raise_domain_error<T>(function, "Argument z must be non-negative, but got z = %1%", z, pol);
 229:    }
 230:    if(p == 0)
 231:    {
 232:       return policies::raise_domain_error<T>(function, "Argument p must not be zero, but got p = %1%", p, pol);
 233:    }
 234:    if(x + y == 0 || y + z == 0 || z + x == 0)
 235:    {
 236:       return policies::raise_domain_error<T>(function, "At most one argument can be zero, only possible result is %1%.", boost::math::numeric_limits<T>::quiet_NaN(), pol);
 237:    }
 238: 
 239:    // for p < 0, the integral is singular, return Cauchy principal value
 240:    if(p < 0)
````
- **L225 EN**: Closes the current lexical scope or compound statement.
  - **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L226 CN**: 开始 `if` 控制流语句并计算其条件。
- **L227 EN**: Opens a new lexical scope or compound statement.
  - **L227 CN**: 打开一个新的词法作用域或复合语句块。
- **L228 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Argument z must be non-negative, but got z = %1%", z, pol)`.
  - **L228 CN**: 以 `policies::raise_domain_error<T>(function, "Argument z must be non-negative, but got z = %1%", z, pol)` 从当前函数返回。
- **L229 EN**: Closes the current lexical scope or compound statement.
  - **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L230 CN**: 开始 `if` 控制流语句并计算其条件。
- **L231 EN**: Opens a new lexical scope or compound statement.
  - **L231 CN**: 打开一个新的词法作用域或复合语句块。
- **L232 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Argument p must not be zero, but got p = %1%", p, pol)`.
  - **L232 CN**: 以 `policies::raise_domain_error<T>(function, "Argument p must not be zero, but got p = %1%", p, pol)` 从当前函数返回。
- **L233 EN**: Closes the current lexical scope or compound statement.
  - **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L234 CN**: 开始 `if` 控制流语句并计算其条件。
- **L235 EN**: Opens a new lexical scope or compound statement.
  - **L235 CN**: 打开一个新的词法作用域或复合语句块。
- **L236 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "At most one argument can be zero, only possible result is %1%.", boost::math::numeric_limits<T>::quiet_NaN(), pol)`.
  - **L236 CN**: 以 `policies::raise_domain_error<T>(function, "At most one argument can be zero, only possible result is %1%.", boost::math::numeric_limits<T>::quiet_NaN(), pol)` 从当前函数返回。
- **L237 EN**: Closes the current lexical scope or compound statement.
  - **L237 CN**: 结束当前词法作用域或复合语句块。
- **L238 EN**: Blank line separating nearby declarations or logic.
  - **L238 CN**: 空行，用于分隔相邻声明或逻辑。
- **L239 EN**: Comment documents nearby intent or usage notes: `for p < 0, the integral is singular, return Cauchy principal value`.
  - **L239 CN**: 注释说明附近代码的意图或使用说明：`for p < 0, the integral is singular, return Cauchy principal value`。
- **L240 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L240 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 241-256 / 第 241-256 行

````cpp
 241:    {
 242:       //
 243:       // We must ensure that x < y < z.
 244:       // Since the integral is symmetrical in x, y and z
 245:       // we can just permute the values:
 246:       //
 247:       if(x > y)
 248:          BOOST_MATH_GPU_SAFE_SWAP(x, y);
 249:       if(y > z)
 250:          BOOST_MATH_GPU_SAFE_SWAP(y, z);
 251:       if(x > y)
 252:          BOOST_MATH_GPU_SAFE_SWAP(x, y);
 253: 
 254:       BOOST_MATH_ASSERT(x <= y);
 255:       BOOST_MATH_ASSERT(y <= z);
 256: 
````
- **L241 EN**: Opens a new lexical scope or compound statement.
  - **L241 CN**: 打开一个新的词法作用域或复合语句块。
- **L242 EN**: Separator comment used for visual grouping.
  - **L242 CN**: 分隔注释，用于视觉分组。
- **L243 EN**: Comment documents nearby intent or usage notes: `We must ensure that x < y < z.`.
  - **L243 CN**: 注释说明附近代码的意图或使用说明：`We must ensure that x < y < z.`。
- **L244 EN**: Comment documents nearby intent or usage notes: `Since the integral is symmetrical in x, y and z`.
  - **L244 CN**: 注释说明附近代码的意图或使用说明：`Since the integral is symmetrical in x, y and z`。
- **L245 EN**: Comment documents nearby intent or usage notes: `we can just permute the values:`.
  - **L245 CN**: 注释说明附近代码的意图或使用说明：`we can just permute the values:`。
- **L246 EN**: Separator comment used for visual grouping.
  - **L246 CN**: 分隔注释，用于视觉分组。
- **L247 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L247 CN**: 开始 `if` 控制流语句并计算其条件。
- **L248 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L248 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L249 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L249 CN**: 开始 `if` 控制流语句并计算其条件。
- **L250 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L250 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L251 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L251 CN**: 开始 `if` 控制流语句并计算其条件。
- **L252 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L252 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L253 EN**: Blank line separating nearby declarations or logic.
  - **L253 CN**: 空行，用于分隔相邻声明或逻辑。
- **L254 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L254 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L255 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L255 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L256 EN**: Blank line separating nearby declarations or logic.
  - **L256 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 257-272 / 第 257-272 行

````cpp
 257:       T q = -p;
 258:       p = (z * (x + y + q) - x * y) / (z + q);
 259: 
 260:       BOOST_MATH_ASSERT(p >= 0);
 261: 
 262:       T value = (p - z) * ellint_rj_imp_final(x, y, z, p, pol);
 263:       value -= 3 * ellint_rf_imp(x, y, z, pol);
 264:       value += 3 * sqrt((x * y * z) / (x * y + p * q)) * ellint_rc_imp(T(x * y + p * q), T(p * q), pol);
 265:       value /= (z + q);
 266:       return value;
 267:    }
 268: 
 269:    return ellint_rj_imp_final(x, y, z, p, pol);
 270: }
 271: 
 272: } // namespace detail
````
- **L257 EN**: Executes a standalone statement or declaration: `T q = -p;`.
  - **L257 CN**: 执行一条独立语句或声明：`T q = -p;`。
- **L258 EN**: Executes a call or declaration centered on `=`.
  - **L258 CN**: 执行以 `=` 为核心的调用或声明。
- **L259 EN**: Blank line separating nearby declarations or logic.
  - **L259 CN**: 空行，用于分隔相邻声明或逻辑。
- **L260 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L260 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L261 EN**: Blank line separating nearby declarations or logic.
  - **L261 CN**: 空行，用于分隔相邻声明或逻辑。
- **L262 EN**: Executes a call or declaration centered on `=`.
  - **L262 CN**: 执行以 `=` 为核心的调用或声明。
- **L263 EN**: Executes a call or declaration centered on `ellint_rf_imp`.
  - **L263 CN**: 执行以 `ellint_rf_imp` 为核心的调用或声明。
- **L264 EN**: Executes a call or declaration centered on `sqrt`.
  - **L264 CN**: 执行以 `sqrt` 为核心的调用或声明。
- **L265 EN**: Executes a call or declaration centered on `/=`.
  - **L265 CN**: 执行以 `/=` 为核心的调用或声明。
- **L266 EN**: Returns from the current function with `value`.
  - **L266 CN**: 以 `value` 从当前函数返回。
- **L267 EN**: Closes the current lexical scope or compound statement.
  - **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Blank line separating nearby declarations or logic.
  - **L268 CN**: 空行，用于分隔相邻声明或逻辑。
- **L269 EN**: Returns from the current function with `ellint_rj_imp_final(x, y, z, p, pol)`.
  - **L269 CN**: 以 `ellint_rj_imp_final(x, y, z, p, pol)` 从当前函数返回。
- **L270 EN**: Closes the current lexical scope or compound statement.
  - **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Blank line separating nearby declarations or logic.
  - **L271 CN**: 空行，用于分隔相邻声明或逻辑。
- **L272 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L272 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。

### Lines 273-288 / 第 273-288 行

````cpp
 273: 
 274: template <class T1, class T2, class T3, class T4, class Policy>
 275: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T1, T2, T3, T4>::type 
 276:    ellint_rj(T1 x, T2 y, T3 z, T4 p, const Policy& pol)
 277: {
 278:    typedef typename tools::promote_args<T1, T2, T3, T4>::type result_type;
 279:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
 280:    return policies::checked_narrowing_cast<result_type, Policy>(
 281:       detail::ellint_rj_imp(
 282:          static_cast<value_type>(x),
 283:          static_cast<value_type>(y),
 284:          static_cast<value_type>(z),
 285:          static_cast<value_type>(p),
 286:          pol), "boost::math::ellint_rj<%1%>(%1%,%1%,%1%,%1%)");
 287: }
 288: 
````
- **L273 EN**: Blank line separating nearby declarations or logic.
  - **L273 CN**: 空行，用于分隔相邻声明或逻辑。
- **L274 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4, class Policy>`.
  - **L274 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4, class Policy>`。
- **L275 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L275 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L276 EN**: Continues logic associated with callable symbol `ellint_rj`.
  - **L276 CN**: 继续与可调用符号 `ellint_rj` 相关的逻辑。
- **L277 EN**: Opens a new lexical scope or compound statement.
  - **L277 CN**: 打开一个新的词法作用域或复合语句块。
- **L278 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T1, T2, T3, T4>::type result_type;`.
  - **L278 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T1, T2, T3, T4>::type result_type;`。
- **L279 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L279 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L280 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(`.
  - **L280 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(` 从当前函数返回。
- **L281 EN**: Continues logic associated with callable symbol `ellint_rj_imp`.
  - **L281 CN**: 继续与可调用符号 `ellint_rj_imp` 相关的逻辑。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(x),`.
  - **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(x),`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(y),`.
  - **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(y),`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(z),`.
  - **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(z),`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(p),`.
  - **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(p),`。
- **L286 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L286 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L287 EN**: Closes the current lexical scope or compound statement.
  - **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Blank line separating nearby declarations or logic.
  - **L288 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 289-299 / 第 289-299 行

````cpp
 289: template <class T1, class T2, class T3, class T4>
 290: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T1, T2, T3, T4>::type 
 291:    ellint_rj(T1 x, T2 y, T3 z, T4 p)
 292: {
 293:    return ellint_rj(x, y, z, p, policies::policy<>());
 294: }
 295: 
 296: }} // namespaces
 297: 
 298: #endif // BOOST_MATH_ELLINT_RJ_HPP
 299: 
````
- **L289 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4>`.
  - **L289 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4>`。
- **L290 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L290 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L291 EN**: Continues logic associated with callable symbol `ellint_rj`.
  - **L291 CN**: 继续与可调用符号 `ellint_rj` 相关的逻辑。
- **L292 EN**: Opens a new lexical scope or compound statement.
  - **L292 CN**: 打开一个新的词法作用域或复合语句块。
- **L293 EN**: Returns from the current function with `ellint_rj(x, y, z, p, policies::policy<>())`.
  - **L293 CN**: 以 `ellint_rj(x, y, z, p, policies::policy<>())` 从当前函数返回。
- **L294 EN**: Closes the current lexical scope or compound statement.
  - **L294 CN**: 结束当前词法作用域或复合语句块。
- **L295 EN**: Blank line separating nearby declarations or logic.
  - **L295 CN**: 空行，用于分隔相邻声明或逻辑。
- **L296 EN**: Continues the surrounding expression or declaration: `}} // namespaces`.
  - **L296 CN**: 继续构造周围的表达式或声明：`}} // namespaces`。
- **L297 EN**: Blank line separating nearby declarations or logic.
  - **L297 CN**: 空行，用于分隔相邻声明或逻辑。
- **L298 EN**: Closes the current preprocessor conditional block or header guard.
  - **L298 CN**: 结束当前预处理条件块或头文件保护。
- **L299 EN**: Blank line separating nearby declarations or logic.
  - **L299 CN**: 空行，用于分隔相邻声明或逻辑。

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

- **Direct local/internal includes / 直接本地或内部包含**: `boost/math/tools/config.hpp`, `boost/math/tools/numeric_limits.hpp`, `boost/math/special_functions/math_fwd.hpp`, `boost/math/policies/error_handling.hpp`, `boost/math/special_functions/ellint_rc.hpp`, `boost/math/special_functions/ellint_rf.hpp`, `boost/math/special_functions/ellint_rd.hpp`
- **Dependency categories / 依赖类别**: Boost.Math special-function declarations / Boost.Math 特殊函数声明 (4), Boost.Math numeric tool helpers / Boost.Math 数值工具辅助逻辑 (2), Boost.Math policy configuration / Boost.Math 策略配置 (1)

- **EN**: `boost/math/tools/config.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/config.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/numeric_limits.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/numeric_limits.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/special_functions/math_fwd.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/math_fwd.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/policies/error_handling.hpp` provides Boost.Math policy configuration.
  - **CN**: `boost/math/policies/error_handling.hpp` 提供Boost.Math 策略配置。
- **EN**: `boost/math/special_functions/ellint_rc.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/ellint_rc.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/ellint_rf.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/ellint_rf.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/ellint_rd.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/ellint_rd.hpp` 提供Boost.Math 特殊函数声明。
