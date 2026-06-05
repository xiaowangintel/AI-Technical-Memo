# expint.hpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/expint.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/` (`llvm-project`)
- **Purpose / 目的**:
  - **EN**: Declares Boost.Math special functions, approximation kernels, and type-generic numeric entry points.
  - **CN**: 声明 Boost.Math 的特殊函数、近似计算内核以及按类型泛化的数值入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行

````cpp
   1: //  Copyright John Maddock 2007.
   2: //  Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_EXPINT_HPP
   8: #define BOOST_MATH_EXPINT_HPP
   9: 
  10: #ifdef _MSC_VER
  11: #pragma once
  12: #pragma warning(push)
  13: #pragma warning(disable:4702) // Unreachable code (release mode only warning)
  14: #endif
  15: 
  16: #include <boost/math/tools/config.hpp>
  17: #include <boost/math/tools/cstdint.hpp>
  18: #include <boost/math/tools/type_traits.hpp>
  19: #include <boost/math/tools/tuple.hpp>
  20: #include <boost/math/tools/precision.hpp>
  21: #include <boost/math/tools/promotion.hpp>
  22: #include <boost/math/tools/fraction.hpp>
  23: #include <boost/math/tools/series.hpp>
  24: #include <boost/math/policies/error_handling.hpp>
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
- **L6 EN**: Blank line separating nearby declarations or logic.
  - **L6 CN**: 空行，用于分隔相邻声明或逻辑。
- **L7 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_EXPINT_HPP`.
  - **L7 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_EXPINT_HPP`。
- **L8 EN**: Defines macro `BOOST_MATH_EXPINT_HPP` for compile-time control, shorthand, or generated boilerplate.
  - **L8 CN**: 定义宏 `BOOST_MATH_EXPINT_HPP`，用于编译期控制、简写或生成样板代码。
- **L9 EN**: Blank line separating nearby declarations or logic.
  - **L9 CN**: 空行，用于分隔相邻声明或逻辑。
- **L10 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L10 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L11 EN**: Uses `#pragma once` to prevent multiple inclusion of this header.
  - **L11 CN**: 使用 `#pragma once` 防止该头文件被重复包含。
- **L12 EN**: Continues logic associated with callable symbol `warning`.
  - **L12 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L13 EN**: Continues logic associated with callable symbol `warning`.
  - **L13 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L14 EN**: Closes the current preprocessor conditional block or header guard.
  - **L14 CN**: 结束当前预处理条件块或头文件保护。
- **L15 EN**: Blank line separating nearby declarations or logic.
  - **L15 CN**: 空行，用于分隔相邻声明或逻辑。
- **L16 EN**: Includes <boost/math/tools/config.hpp> to access Boost.Math numeric tool helpers.
  - **L16 CN**: 引入 <boost/math/tools/config.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L17 EN**: Includes <boost/math/tools/cstdint.hpp> to access Boost.Math numeric tool helpers.
  - **L17 CN**: 引入 <boost/math/tools/cstdint.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L18 EN**: Includes <boost/math/tools/type_traits.hpp> to access Boost.Math numeric tool helpers.
  - **L18 CN**: 引入 <boost/math/tools/type_traits.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L19 EN**: Includes <boost/math/tools/tuple.hpp> to access Boost.Math numeric tool helpers.
  - **L19 CN**: 引入 <boost/math/tools/tuple.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L20 EN**: Includes <boost/math/tools/precision.hpp> to access Boost.Math numeric tool helpers.
  - **L20 CN**: 引入 <boost/math/tools/precision.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L21 EN**: Includes <boost/math/tools/promotion.hpp> to access Boost.Math numeric tool helpers.
  - **L21 CN**: 引入 <boost/math/tools/promotion.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L22 EN**: Includes <boost/math/tools/fraction.hpp> to access Boost.Math numeric tool helpers.
  - **L22 CN**: 引入 <boost/math/tools/fraction.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L23 EN**: Includes <boost/math/tools/series.hpp> to access Boost.Math numeric tool helpers.
  - **L23 CN**: 引入 <boost/math/tools/series.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L24 EN**: Includes <boost/math/policies/error_handling.hpp> to access Boost.Math policy configuration.
  - **L24 CN**: 引入 <boost/math/policies/error_handling.hpp> 以使用Boost.Math 策略配置。

### Lines 25-48 / 第 25-48 行

````cpp
  25: #include <boost/math/special_functions/math_fwd.hpp>
  26: #include <boost/math/special_functions/digamma.hpp>
  27: #include <boost/math/special_functions/log1p.hpp>
  28: 
  29: #if defined(__GNUC__) && defined(BOOST_MATH_USE_FLOAT128)
  30: //
  31: // This is the only way we can avoid
  32: // warning: non-standard suffix on floating constant [-Wpedantic]
  33: // when building with -Wall -pedantic.  Neither __extension__
  34: // nor #pragma diagnostic ignored work :(
  35: //
  36: #pragma GCC system_header
  37: #endif
  38: 
  39: namespace boost{ namespace math{
  40: 
  41: template <class T, class Policy>
  42: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
  43:    expint(unsigned n, T z, const Policy& /*pol*/);
  44: 
  45: namespace detail{
  46: 
  47: template <class T>
  48: BOOST_MATH_GPU_ENABLED inline T expint_1_rational(const T& z, const boost::math::integral_constant<int, 0>&)
````
- **L25 EN**: Includes <boost/math/special_functions/math_fwd.hpp> to access Boost.Math special-function declarations.
  - **L25 CN**: 引入 <boost/math/special_functions/math_fwd.hpp> 以使用Boost.Math 特殊函数声明。
- **L26 EN**: Includes <boost/math/special_functions/digamma.hpp> to access Boost.Math special-function declarations.
  - **L26 CN**: 引入 <boost/math/special_functions/digamma.hpp> 以使用Boost.Math 特殊函数声明。
- **L27 EN**: Includes <boost/math/special_functions/log1p.hpp> to access Boost.Math special-function declarations.
  - **L27 CN**: 引入 <boost/math/special_functions/log1p.hpp> 以使用Boost.Math 特殊函数声明。
- **L28 EN**: Blank line separating nearby declarations or logic.
  - **L28 CN**: 空行，用于分隔相邻声明或逻辑。
- **L29 EN**: Starts a preprocessor conditional block: `#if defined(__GNUC__) && defined(BOOST_MATH_USE_FLOAT128)`.
  - **L29 CN**: 开始一个预处理条件块：`#if defined(__GNUC__) && defined(BOOST_MATH_USE_FLOAT128)`。
- **L30 EN**: Separator comment used for visual grouping.
  - **L30 CN**: 分隔注释，用于视觉分组。
- **L31 EN**: Comment documents nearby intent or usage notes: `This is the only way we can avoid`.
  - **L31 CN**: 注释说明附近代码的意图或使用说明：`This is the only way we can avoid`。
- **L32 EN**: Comment documents nearby intent or usage notes: `warning: non-standard suffix on floating constant [-Wpedantic]`.
  - **L32 CN**: 注释说明附近代码的意图或使用说明：`warning: non-standard suffix on floating constant [-Wpedantic]`。
- **L33 EN**: Comment documents nearby intent or usage notes: `when building with -Wall -pedantic.  Neither __extension__`.
  - **L33 CN**: 注释说明附近代码的意图或使用说明：`when building with -Wall -pedantic.  Neither __extension__`。
- **L34 EN**: Comment documents nearby intent or usage notes: `nor #pragma diagnostic ignored work :(`.
  - **L34 CN**: 注释说明附近代码的意图或使用说明：`nor #pragma diagnostic ignored work :(`。
- **L35 EN**: Separator comment used for visual grouping.
  - **L35 CN**: 分隔注释，用于视觉分组。
- **L36 EN**: Continues the surrounding expression or declaration: `#pragma GCC system_header`.
  - **L36 CN**: 继续构造周围的表达式或声明：`#pragma GCC system_header`。
- **L37 EN**: Closes the current preprocessor conditional block or header guard.
  - **L37 CN**: 结束当前预处理条件块或头文件保护。
- **L38 EN**: Blank line separating nearby declarations or logic.
  - **L38 CN**: 空行，用于分隔相邻声明或逻辑。
- **L39 EN**: Opens namespace scope `boost{ namespace math`.
  - **L39 CN**: 打开命名空间作用域 `boost{ namespace math`。
- **L40 EN**: Blank line separating nearby declarations or logic.
  - **L40 CN**: 空行，用于分隔相邻声明或逻辑。
- **L41 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L41 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L42 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L42 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L43 EN**: Executes a call or declaration centered on `expint`.
  - **L43 CN**: 执行以 `expint` 为核心的调用或声明。
- **L44 EN**: Blank line separating nearby declarations or logic.
  - **L44 CN**: 空行，用于分隔相邻声明或逻辑。
- **L45 EN**: Opens namespace scope `detail`.
  - **L45 CN**: 打开命名空间作用域 `detail`。
- **L46 EN**: Blank line separating nearby declarations or logic.
  - **L46 CN**: 空行，用于分隔相邻声明或逻辑。
- **L47 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L47 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L48 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L48 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 49-72 / 第 49-72 行

````cpp
  49: {
  50:    // this function is never actually called
  51:    BOOST_MATH_ASSERT(0);
  52:    return z;
  53: }
  54: 
  55: template <class T>
  56: BOOST_MATH_GPU_ENABLED T expint_1_rational(const T& z, const boost::math::integral_constant<int, 53>&)
  57: {
  58:    BOOST_MATH_STD_USING
  59:    T result;
  60:    if(z <= 1)
  61:    {
  62:       // Maximum Deviation Found:                     2.006e-18
  63:       // Expected Error Term:                         2.006e-18
  64:       // Max error found at double precision:         2.760e-17
  65:       // LCOV_EXCL_START
  66:       static const T Y = 0.66373538970947265625F;
  67:       static const T P[6] = {
  68:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.0865197248079397976498),
  69:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.0320913665303559189999),
  70:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.245088216639761496153),
  71:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0368031736257943745142),
  72:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00399167106081113256961),
````
- **L49 EN**: Opens a new lexical scope or compound statement.
  - **L49 CN**: 打开一个新的词法作用域或复合语句块。
- **L50 EN**: Comment documents nearby intent or usage notes: `this function is never actually called`.
  - **L50 CN**: 注释说明附近代码的意图或使用说明：`this function is never actually called`。
- **L51 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L51 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L52 EN**: Returns from the current function with `z`.
  - **L52 CN**: 以 `z` 从当前函数返回。
- **L53 EN**: Closes the current lexical scope or compound statement.
  - **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Blank line separating nearby declarations or logic.
  - **L54 CN**: 空行，用于分隔相邻声明或逻辑。
- **L55 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L55 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L56 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L56 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L57 EN**: Opens a new lexical scope or compound statement.
  - **L57 CN**: 打开一个新的词法作用域或复合语句块。
- **L58 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L58 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L59 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L59 CN**: 执行一条独立语句或声明：`T result;`。
- **L60 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L60 CN**: 开始 `if` 控制流语句并计算其条件。
- **L61 EN**: Opens a new lexical scope or compound statement.
  - **L61 CN**: 打开一个新的词法作用域或复合语句块。
- **L62 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     2.006e-18`.
  - **L62 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     2.006e-18`。
- **L63 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         2.006e-18`.
  - **L63 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         2.006e-18`。
- **L64 EN**: Comment documents nearby intent or usage notes: `Max error found at double precision:         2.760e-17`.
  - **L64 CN**: 注释说明附近代码的意图或使用说明：`Max error found at double precision:         2.760e-17`。
- **L65 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L65 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L66 EN**: Initializes variable `Y` from the right-hand expression.
  - **L66 CN**: 使用右侧表达式初始化变量 `Y`。
- **L67 EN**: Continues the surrounding expression or declaration: `static const T P[6] = {`.
  - **L67 CN**: 继续构造周围的表达式或声明：`static const T P[6] = {`。
- **L68 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L68 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L69 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L69 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L70 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L70 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L71 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L71 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L72 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L72 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 73-96 / 第 73-96 行

````cpp
  73:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.000111507792921197858394)
  74:       };
  75:       static const T Q[6] = {
  76:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
  77:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.37091387659397013215),
  78:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.056770677104207528384),
  79:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.00427347600017103698101),
  80:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.000131049900798434683324),
  81:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.528611029520217142048e-6)
  82:       };
  83:       // LCOV_EXCL_STOP
  84:       result = tools::evaluate_polynomial(P, z)
  85:          / tools::evaluate_polynomial(Q, z);
  86:       result += z - log(z) - Y;
  87:    }
  88:    else if(z < -boost::math::tools::log_min_value<T>())
  89:    {
  90:       // Maximum Deviation Found (interpolated):      1.444e-17
  91:       // Max error found at double precision:         3.119e-17
  92:       // LCOV_EXCL_START
  93:       static const T P[11] = {
  94:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.121013190657725568138e-18),
  95:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.999999999999998811143),
  96:          BOOST_MATH_BIG_CONSTANT(T, 53, -43.3058660811817946037),
````
- **L73 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L73 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L74 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L74 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L75 EN**: Continues the surrounding expression or declaration: `static const T Q[6] = {`.
  - **L75 CN**: 继续构造周围的表达式或声明：`static const T Q[6] = {`。
- **L76 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L76 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L77 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L77 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L78 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L78 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L79 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L79 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L80 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L80 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L81 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L81 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L82 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L82 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L83 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L83 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L84 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L84 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L85 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L85 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L86 EN**: Executes a call or declaration centered on `log`.
  - **L86 CN**: 执行以 `log` 为核心的调用或声明。
- **L87 EN**: Closes the current lexical scope or compound statement.
  - **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Starts the alternative branch of the preceding conditional.
  - **L88 CN**: 开始前一个条件语句的备选分支。
- **L89 EN**: Opens a new lexical scope or compound statement.
  - **L89 CN**: 打开一个新的词法作用域或复合语句块。
- **L90 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found (interpolated):      1.444e-17`.
  - **L90 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found (interpolated):      1.444e-17`。
- **L91 EN**: Comment documents nearby intent or usage notes: `Max error found at double precision:         3.119e-17`.
  - **L91 CN**: 注释说明附近代码的意图或使用说明：`Max error found at double precision:         3.119e-17`。
- **L92 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L92 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L93 EN**: Continues the surrounding expression or declaration: `static const T P[11] = {`.
  - **L93 CN**: 继续构造周围的表达式或声明：`static const T P[11] = {`。
- **L94 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L94 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L95 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L95 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L96 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L96 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 97-120 / 第 97-120 行

````cpp
  97:          BOOST_MATH_BIG_CONSTANT(T, 53, -724.581482791462469795),
  98:          BOOST_MATH_BIG_CONSTANT(T, 53, -6046.8250112711035463),
  99:          BOOST_MATH_BIG_CONSTANT(T, 53, -27182.6254466733970467),
 100:          BOOST_MATH_BIG_CONSTANT(T, 53, -66598.2652345418633509),
 101:          BOOST_MATH_BIG_CONSTANT(T, 53, -86273.1567711649528784),
 102:          BOOST_MATH_BIG_CONSTANT(T, 53, -54844.4587226402067411),
 103:          BOOST_MATH_BIG_CONSTANT(T, 53, -14751.4895786128450662),
 104:          BOOST_MATH_BIG_CONSTANT(T, 53, -1185.45720315201027667)
 105:       };
 106:       static const T Q[12] = {
 107:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
 108:          BOOST_MATH_BIG_CONSTANT(T, 53, 45.3058660811801465927),
 109:          BOOST_MATH_BIG_CONSTANT(T, 53, 809.193214954550328455),
 110:          BOOST_MATH_BIG_CONSTANT(T, 53, 7417.37624454689546708),
 111:          BOOST_MATH_BIG_CONSTANT(T, 53, 38129.5594484818471461),
 112:          BOOST_MATH_BIG_CONSTANT(T, 53, 113057.05869159631492),
 113:          BOOST_MATH_BIG_CONSTANT(T, 53, 192104.047790227984431),
 114:          BOOST_MATH_BIG_CONSTANT(T, 53, 180329.498380501819718),
 115:          BOOST_MATH_BIG_CONSTANT(T, 53, 86722.3403467334749201),
 116:          BOOST_MATH_BIG_CONSTANT(T, 53, 18455.4124737722049515),
 117:          BOOST_MATH_BIG_CONSTANT(T, 53, 1229.20784182403048905),
 118:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.776491285282330997549)
 119:       };
 120:       // LCOV_EXCL_STOP
````
- **L97 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L97 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L98 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L98 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L99 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L99 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L100 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L100 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L101 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L101 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L102 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L102 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L103 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L103 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L104 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L104 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L105 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L105 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L106 EN**: Continues the surrounding expression or declaration: `static const T Q[12] = {`.
  - **L106 CN**: 继续构造周围的表达式或声明：`static const T Q[12] = {`。
- **L107 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L107 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L108 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L108 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L109 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L109 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L110 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L110 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L111 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L111 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L112 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L112 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L113 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L113 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L114 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L114 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L115 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L115 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L116 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L116 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L117 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L117 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L118 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L118 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L119 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L119 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L120 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L120 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。

### Lines 121-144 / 第 121-144 行

````cpp
 121:       T recip = 1 / z;
 122:       result = 1 + tools::evaluate_polynomial(P, recip)
 123:          / tools::evaluate_polynomial(Q, recip);
 124:       result *= exp(-z) * recip;
 125:    }
 126:    else
 127:    {
 128:       result = 0;
 129:    }
 130:    return result;
 131: }
 132: 
 133: template <class T>
 134: BOOST_MATH_GPU_ENABLED T expint_1_rational(const T& z, const boost::math::integral_constant<int, 64>&)
 135: {
 136:    BOOST_MATH_STD_USING
 137:    T result;
 138:    if(z <= 1)
 139:    {
 140:       // Maximum Deviation Found:                     3.807e-20
 141:       // Expected Error Term:                         3.807e-20
 142:       // Max error found at long double precision:    6.249e-20
 143:       // LCOV_EXCL_START
 144:       static const T Y = 0.66373538970947265625F;
````
- **L121 EN**: Executes a standalone statement or declaration: `T recip = 1 / z;`.
  - **L121 CN**: 执行一条独立语句或声明：`T recip = 1 / z;`。
- **L122 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L122 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L123 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L123 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L124 EN**: Executes a call or declaration centered on `exp`.
  - **L124 CN**: 执行以 `exp` 为核心的调用或声明。
- **L125 EN**: Closes the current lexical scope or compound statement.
  - **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Starts the alternative branch of the preceding conditional.
  - **L126 CN**: 开始前一个条件语句的备选分支。
- **L127 EN**: Opens a new lexical scope or compound statement.
  - **L127 CN**: 打开一个新的词法作用域或复合语句块。
- **L128 EN**: Executes a standalone statement or declaration: `result = 0;`.
  - **L128 CN**: 执行一条独立语句或声明：`result = 0;`。
- **L129 EN**: Closes the current lexical scope or compound statement.
  - **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Returns from the current function with `result`.
  - **L130 CN**: 以 `result` 从当前函数返回。
- **L131 EN**: Closes the current lexical scope or compound statement.
  - **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic.
  - **L132 CN**: 空行，用于分隔相邻声明或逻辑。
- **L133 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L133 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L134 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L134 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L135 EN**: Opens a new lexical scope or compound statement.
  - **L135 CN**: 打开一个新的词法作用域或复合语句块。
- **L136 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L136 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L137 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L137 CN**: 执行一条独立语句或声明：`T result;`。
- **L138 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L138 CN**: 开始 `if` 控制流语句并计算其条件。
- **L139 EN**: Opens a new lexical scope or compound statement.
  - **L139 CN**: 打开一个新的词法作用域或复合语句块。
- **L140 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     3.807e-20`.
  - **L140 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     3.807e-20`。
- **L141 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         3.807e-20`.
  - **L141 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         3.807e-20`。
- **L142 EN**: Comment documents nearby intent or usage notes: `Max error found at long double precision:    6.249e-20`.
  - **L142 CN**: 注释说明附近代码的意图或使用说明：`Max error found at long double precision:    6.249e-20`。
- **L143 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L143 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L144 EN**: Initializes variable `Y` from the right-hand expression.
  - **L144 CN**: 使用右侧表达式初始化变量 `Y`。

### Lines 145-168 / 第 145-168 行

````cpp
 145:       static const T P[6] = {
 146:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0865197248079397956816),
 147:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0275114007037026844633),
 148:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.246594388074877139824),
 149:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0237624819878732642231),
 150:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00259113319641673986276),
 151:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.30853660894346057053e-4)
 152:       };
 153:       static const T Q[7] = {
 154:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 155:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.317978365797784100273),
 156:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0393622602554758722511),
 157:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.00204062029115966323229),
 158:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.732512107100088047854e-5),
 159:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.202872781770207871975e-5),
 160:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.52779248094603709945e-7)
 161:       };
 162:       // LCOV_EXCL_STOP
 163:       result = tools::evaluate_polynomial(P, z)
 164:          / tools::evaluate_polynomial(Q, z);
 165:       result += z - log(z) - Y;
 166:    }
 167:    else if(z < -boost::math::tools::log_min_value<T>())
 168:    {
````
- **L145 EN**: Continues the surrounding expression or declaration: `static const T P[6] = {`.
  - **L145 CN**: 继续构造周围的表达式或声明：`static const T P[6] = {`。
- **L146 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L146 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L147 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L147 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L148 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L148 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L149 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L149 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L150 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L150 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L151 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L151 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L152 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L152 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L153 EN**: Continues the surrounding expression or declaration: `static const T Q[7] = {`.
  - **L153 CN**: 继续构造周围的表达式或声明：`static const T Q[7] = {`。
- **L154 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L154 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L155 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L155 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L156 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L156 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L157 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L157 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L158 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L158 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L159 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L159 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L160 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L160 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L161 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L161 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L162 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L162 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L163 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L163 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L164 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L164 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L165 EN**: Executes a call or declaration centered on `log`.
  - **L165 CN**: 执行以 `log` 为核心的调用或声明。
- **L166 EN**: Closes the current lexical scope or compound statement.
  - **L166 CN**: 结束当前词法作用域或复合语句块。
- **L167 EN**: Starts the alternative branch of the preceding conditional.
  - **L167 CN**: 开始前一个条件语句的备选分支。
- **L168 EN**: Opens a new lexical scope or compound statement.
  - **L168 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 169-192 / 第 169-192 行

````cpp
 169:       // Maximum Deviation Found (interpolated):     2.220e-20
 170:       // Max error found at long double precision:   1.346e-19
 171:       // LCOV_EXCL_START
 172:       static const T P[14] = {
 173:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.534401189080684443046e-23),
 174:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.999999999999999999905),
 175:          BOOST_MATH_BIG_CONSTANT(T, 64, -62.1517806091379402505),
 176:          BOOST_MATH_BIG_CONSTANT(T, 64, -1568.45688271895145277),
 177:          BOOST_MATH_BIG_CONSTANT(T, 64, -21015.3431990874009619),
 178:          BOOST_MATH_BIG_CONSTANT(T, 64, -164333.011755931661949),
 179:          BOOST_MATH_BIG_CONSTANT(T, 64, -777917.270775426696103),
 180:          BOOST_MATH_BIG_CONSTANT(T, 64, -2244188.56195255112937),
 181:          BOOST_MATH_BIG_CONSTANT(T, 64, -3888702.98145335643429),
 182:          BOOST_MATH_BIG_CONSTANT(T, 64, -3909822.65621952648353),
 183:          BOOST_MATH_BIG_CONSTANT(T, 64, -2149033.9538897398457),
 184:          BOOST_MATH_BIG_CONSTANT(T, 64, -584705.537139793925189),
 185:          BOOST_MATH_BIG_CONSTANT(T, 64, -65815.2605361889477244),
 186:          BOOST_MATH_BIG_CONSTANT(T, 64, -2038.82870680427258038)
 187:       };
 188:       static const T Q[14] = {
 189:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 190:          BOOST_MATH_BIG_CONSTANT(T, 64, 64.1517806091379399478),
 191:          BOOST_MATH_BIG_CONSTANT(T, 64, 1690.76044393722763785),
 192:          BOOST_MATH_BIG_CONSTANT(T, 64, 24035.9534033068949426),
````
- **L169 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found (interpolated):     2.220e-20`.
  - **L169 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found (interpolated):     2.220e-20`。
- **L170 EN**: Comment documents nearby intent or usage notes: `Max error found at long double precision:   1.346e-19`.
  - **L170 CN**: 注释说明附近代码的意图或使用说明：`Max error found at long double precision:   1.346e-19`。
- **L171 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L171 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L172 EN**: Continues the surrounding expression or declaration: `static const T P[14] = {`.
  - **L172 CN**: 继续构造周围的表达式或声明：`static const T P[14] = {`。
- **L173 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L173 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L174 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L174 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L175 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L175 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L176 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L176 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L177 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L177 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L178 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L178 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L179 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L179 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L180 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L180 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L181 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L181 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L182 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L182 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L183 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L183 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L184 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L184 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L185 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L185 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L186 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L186 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L187 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L187 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L188 EN**: Continues the surrounding expression or declaration: `static const T Q[14] = {`.
  - **L188 CN**: 继续构造周围的表达式或声明：`static const T Q[14] = {`。
- **L189 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L189 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L190 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L190 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L191 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L191 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L192 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L192 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 193-216 / 第 193-216 行

````cpp
 193:          BOOST_MATH_BIG_CONSTANT(T, 64, 203679.998633572361706),
 194:          BOOST_MATH_BIG_CONSTANT(T, 64, 1074661.58459976978285),
 195:          BOOST_MATH_BIG_CONSTANT(T, 64, 3586552.65020899358773),
 196:          BOOST_MATH_BIG_CONSTANT(T, 64, 7552186.84989547621411),
 197:          BOOST_MATH_BIG_CONSTANT(T, 64, 9853333.79353054111434),
 198:          BOOST_MATH_BIG_CONSTANT(T, 64, 7689642.74550683631258),
 199:          BOOST_MATH_BIG_CONSTANT(T, 64, 3385553.35146759180739),
 200:          BOOST_MATH_BIG_CONSTANT(T, 64, 763218.072732396428725),
 201:          BOOST_MATH_BIG_CONSTANT(T, 64, 73930.2995984054930821),
 202:          BOOST_MATH_BIG_CONSTANT(T, 64, 2063.86994219629165937)
 203:       };
 204:       // LCOV_EXCL_STOP
 205:       T recip = 1 / z;
 206:       result = 1 + tools::evaluate_polynomial(P, recip)
 207:          / tools::evaluate_polynomial(Q, recip);
 208:       result *= exp(-z) * recip;
 209:    }
 210:    else
 211:    {
 212:       result = 0;
 213:    }
 214:    return result;
 215: }
 216: 
````
- **L193 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L193 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L194 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L194 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L195 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L195 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L196 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L196 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L197 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L197 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L198 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L198 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L199 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L199 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L200 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L200 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L201 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L201 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L202 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L202 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L203 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L203 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L204 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L204 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L205 EN**: Executes a standalone statement or declaration: `T recip = 1 / z;`.
  - **L205 CN**: 执行一条独立语句或声明：`T recip = 1 / z;`。
- **L206 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L206 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L207 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L207 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L208 EN**: Executes a call or declaration centered on `exp`.
  - **L208 CN**: 执行以 `exp` 为核心的调用或声明。
- **L209 EN**: Closes the current lexical scope or compound statement.
  - **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Starts the alternative branch of the preceding conditional.
  - **L210 CN**: 开始前一个条件语句的备选分支。
- **L211 EN**: Opens a new lexical scope or compound statement.
  - **L211 CN**: 打开一个新的词法作用域或复合语句块。
- **L212 EN**: Executes a standalone statement or declaration: `result = 0;`.
  - **L212 CN**: 执行一条独立语句或声明：`result = 0;`。
- **L213 EN**: Closes the current lexical scope or compound statement.
  - **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Returns from the current function with `result`.
  - **L214 CN**: 以 `result` 从当前函数返回。
- **L215 EN**: Closes the current lexical scope or compound statement.
  - **L215 CN**: 结束当前词法作用域或复合语句块。
- **L216 EN**: Blank line separating nearby declarations or logic.
  - **L216 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 217-240 / 第 217-240 行

````cpp
 217: template <class T>
 218: BOOST_MATH_GPU_ENABLED T expint_1_rational(const T& z, const boost::math::integral_constant<int, 113>&)
 219: {
 220:    BOOST_MATH_STD_USING
 221:    T result;
 222:    if(z <= 1)
 223:    {
 224:       // Maximum Deviation Found:                     2.477e-35
 225:       // Expected Error Term:                         2.477e-35
 226:       // Max error found at long double precision:    6.810e-35
 227:       // LCOV_EXCL_START
 228:       static const T Y = 0.66373538970947265625F;
 229:       static const T P[10] = {
 230:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.0865197248079397956434879099175975937),
 231:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.0369066175910795772830865304506087759),
 232:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.24272036838415474665971599314725545),
 233:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.0502166331248948515282379137550178307),
 234:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.00768384138547489410285101483730424919),
 235:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.000612574337702109683505224915484717162),
 236:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.380207107950635046971492617061708534e-4),
 237:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.136528159460768830763009294683628406e-5),
 238:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.346839106212658259681029388908658618e-7),
 239:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.340500302777838063940402160594523429e-9)
 240:       };
````
- **L217 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L217 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L218 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L218 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L219 EN**: Opens a new lexical scope or compound statement.
  - **L219 CN**: 打开一个新的词法作用域或复合语句块。
- **L220 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L220 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L221 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L221 CN**: 执行一条独立语句或声明：`T result;`。
- **L222 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L222 CN**: 开始 `if` 控制流语句并计算其条件。
- **L223 EN**: Opens a new lexical scope or compound statement.
  - **L223 CN**: 打开一个新的词法作用域或复合语句块。
- **L224 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     2.477e-35`.
  - **L224 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     2.477e-35`。
- **L225 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         2.477e-35`.
  - **L225 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         2.477e-35`。
- **L226 EN**: Comment documents nearby intent or usage notes: `Max error found at long double precision:    6.810e-35`.
  - **L226 CN**: 注释说明附近代码的意图或使用说明：`Max error found at long double precision:    6.810e-35`。
- **L227 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L227 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L228 EN**: Initializes variable `Y` from the right-hand expression.
  - **L228 CN**: 使用右侧表达式初始化变量 `Y`。
- **L229 EN**: Continues the surrounding expression or declaration: `static const T P[10] = {`.
  - **L229 CN**: 继续构造周围的表达式或声明：`static const T P[10] = {`。
- **L230 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L230 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L231 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L231 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L232 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L232 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L233 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L233 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L234 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L234 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L235 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L235 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L236 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L236 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L237 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L237 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L238 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L238 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L239 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L239 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L240 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L240 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 241-264 / 第 241-264 行

````cpp
 241:       static const T Q[10] = {
 242:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 243:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.426568827778942588160423015589537302),
 244:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.0841384046470893490592450881447510148),
 245:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.0100557215850668029618957359471132995),
 246:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.000799334870474627021737357294799839363),
 247:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.434452090903862735242423068552687688e-4),
 248:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.15829674748799079874182885081231252e-5),
 249:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.354406206738023762100882270033082198e-7),
 250:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.369373328141051577845488477377890236e-9),
 251:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.274149801370933606409282434677600112e-12)
 252:       };
 253:       // LCOV_EXCL_STOP
 254:       result = tools::evaluate_polynomial(P, z)
 255:          / tools::evaluate_polynomial(Q, z);
 256:       result += z - log(z) - Y;
 257:    }
 258:    else if(z <= 4)
 259:    {
 260:       // Max error in interpolated form:             5.614e-35
 261:       // Max error found at long double precision:   7.979e-35
 262:       // LCOV_EXCL_START
 263:       static const T Y = 0.70190334320068359375F;
 264: 
````
- **L241 EN**: Continues the surrounding expression or declaration: `static const T Q[10] = {`.
  - **L241 CN**: 继续构造周围的表达式或声明：`static const T Q[10] = {`。
- **L242 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L242 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L243 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L243 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L244 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L244 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L245 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L245 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L246 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L246 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L247 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L247 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L248 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L248 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L249 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L249 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L250 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L250 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L251 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L251 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L252 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L252 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L253 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L253 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L254 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L254 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L255 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L255 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L256 EN**: Executes a call or declaration centered on `log`.
  - **L256 CN**: 执行以 `log` 为核心的调用或声明。
- **L257 EN**: Closes the current lexical scope or compound statement.
  - **L257 CN**: 结束当前词法作用域或复合语句块。
- **L258 EN**: Starts the alternative branch of the preceding conditional.
  - **L258 CN**: 开始前一个条件语句的备选分支。
- **L259 EN**: Opens a new lexical scope or compound statement.
  - **L259 CN**: 打开一个新的词法作用域或复合语句块。
- **L260 EN**: Comment documents nearby intent or usage notes: `Max error in interpolated form:             5.614e-35`.
  - **L260 CN**: 注释说明附近代码的意图或使用说明：`Max error in interpolated form:             5.614e-35`。
- **L261 EN**: Comment documents nearby intent or usage notes: `Max error found at long double precision:   7.979e-35`.
  - **L261 CN**: 注释说明附近代码的意图或使用说明：`Max error found at long double precision:   7.979e-35`。
- **L262 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L262 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L263 EN**: Initializes variable `Y` from the right-hand expression.
  - **L263 CN**: 使用右侧表达式初始化变量 `Y`。
- **L264 EN**: Blank line separating nearby declarations or logic.
  - **L264 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 265-288 / 第 265-288 行

````cpp
 265:       static const T P[16] = {
 266:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.298096656795020369955077350585959794),
 267:          BOOST_MATH_BIG_CONSTANT(T, 113, 12.9314045995266142913135497455971247),
 268:          BOOST_MATH_BIG_CONSTANT(T, 113, 226.144334921582637462526628217345501),
 269:          BOOST_MATH_BIG_CONSTANT(T, 113, 2070.83670924261732722117682067381405),
 270:          BOOST_MATH_BIG_CONSTANT(T, 113, 10715.1115684330959908244769731347186),
 271:          BOOST_MATH_BIG_CONSTANT(T, 113, 30728.7876355542048019664777316053311),
 272:          BOOST_MATH_BIG_CONSTANT(T, 113, 38520.6078609349855436936232610875297),
 273:          BOOST_MATH_BIG_CONSTANT(T, 113, -27606.0780981527583168728339620565165),
 274:          BOOST_MATH_BIG_CONSTANT(T, 113, -169026.485055785605958655247592604835),
 275:          BOOST_MATH_BIG_CONSTANT(T, 113, -254361.919204983608659069868035092282),
 276:          BOOST_MATH_BIG_CONSTANT(T, 113, -195765.706874132267953259272028679935),
 277:          BOOST_MATH_BIG_CONSTANT(T, 113, -83352.6826013533205474990119962408675),
 278:          BOOST_MATH_BIG_CONSTANT(T, 113, -19251.6828496869586415162597993050194),
 279:          BOOST_MATH_BIG_CONSTANT(T, 113, -2226.64251774578542836725386936102339),
 280:          BOOST_MATH_BIG_CONSTANT(T, 113, -109.009437301400845902228611986479816),
 281:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.51492042209561411434644938098833499)
 282:       };
 283:       static const T Q[16] = {
 284:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 285:          BOOST_MATH_BIG_CONSTANT(T, 113, 46.734521442032505570517810766704587),
 286:          BOOST_MATH_BIG_CONSTANT(T, 113, 908.694714348462269000247450058595655),
 287:          BOOST_MATH_BIG_CONSTANT(T, 113, 9701.76053033673927362784882748513195),
 288:          BOOST_MATH_BIG_CONSTANT(T, 113, 63254.2815292641314236625196594947774),
````
- **L265 EN**: Continues the surrounding expression or declaration: `static const T P[16] = {`.
  - **L265 CN**: 继续构造周围的表达式或声明：`static const T P[16] = {`。
- **L266 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L266 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L267 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L267 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L268 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L268 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L269 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L269 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L270 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L270 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L271 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L271 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L272 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L272 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L273 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L273 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L274 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L274 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L275 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L275 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L276 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L276 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L277 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L277 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L278 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L278 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L279 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L279 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L280 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L280 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L281 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L281 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L282 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L282 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L283 EN**: Continues the surrounding expression or declaration: `static const T Q[16] = {`.
  - **L283 CN**: 继续构造周围的表达式或声明：`static const T Q[16] = {`。
- **L284 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L284 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L285 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L285 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L286 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L286 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L287 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L287 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L288 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L288 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 289-312 / 第 289-312 行

````cpp
 289:          BOOST_MATH_BIG_CONSTANT(T, 113, 265115.641285880437335106541757711092),
 290:          BOOST_MATH_BIG_CONSTANT(T, 113, 732707.841188071900498536533086567735),
 291:          BOOST_MATH_BIG_CONSTANT(T, 113, 1348514.02492635723327306628712057794),
 292:          BOOST_MATH_BIG_CONSTANT(T, 113, 1649986.81455283047769673308781585991),
 293:          BOOST_MATH_BIG_CONSTANT(T, 113, 1326000.828522976970116271208812099),
 294:          BOOST_MATH_BIG_CONSTANT(T, 113, 683643.09490612171772350481773951341),
 295:          BOOST_MATH_BIG_CONSTANT(T, 113, 217640.505137263607952365685653352229),
 296:          BOOST_MATH_BIG_CONSTANT(T, 113, 40288.3467237411710881822569476155485),
 297:          BOOST_MATH_BIG_CONSTANT(T, 113, 3932.89353979531632559232883283175754),
 298:          BOOST_MATH_BIG_CONSTANT(T, 113, 169.845369689596739824177412096477219),
 299:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.17607292280092201170768401876895354)
 300:       };
 301:       // LCOV_EXCL_STOP
 302:       T recip = 1 / z;
 303:       result = Y + tools::evaluate_polynomial(P, recip)
 304:          / tools::evaluate_polynomial(Q, recip);
 305:       result *= exp(-z) * recip;
 306:    }
 307:    else if(z < -boost::math::tools::log_min_value<T>())
 308:    {
 309:       // Max error in interpolated form:             4.413e-35
 310:       // Max error found at long double precision:   8.928e-35
 311:       // LCOV_EXCL_START
 312:       static const T P[19] = {
````
- **L289 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L289 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L290 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L290 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L291 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L291 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L292 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L292 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L293 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L293 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L294 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L294 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L295 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L295 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L296 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L296 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L297 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L297 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L298 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L298 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L299 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L299 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L300 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L300 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L301 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L301 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L302 EN**: Executes a standalone statement or declaration: `T recip = 1 / z;`.
  - **L302 CN**: 执行一条独立语句或声明：`T recip = 1 / z;`。
- **L303 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L303 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L304 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L304 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L305 EN**: Executes a call or declaration centered on `exp`.
  - **L305 CN**: 执行以 `exp` 为核心的调用或声明。
- **L306 EN**: Closes the current lexical scope or compound statement.
  - **L306 CN**: 结束当前词法作用域或复合语句块。
- **L307 EN**: Starts the alternative branch of the preceding conditional.
  - **L307 CN**: 开始前一个条件语句的备选分支。
- **L308 EN**: Opens a new lexical scope or compound statement.
  - **L308 CN**: 打开一个新的词法作用域或复合语句块。
- **L309 EN**: Comment documents nearby intent or usage notes: `Max error in interpolated form:             4.413e-35`.
  - **L309 CN**: 注释说明附近代码的意图或使用说明：`Max error in interpolated form:             4.413e-35`。
- **L310 EN**: Comment documents nearby intent or usage notes: `Max error found at long double precision:   8.928e-35`.
  - **L310 CN**: 注释说明附近代码的意图或使用说明：`Max error found at long double precision:   8.928e-35`。
- **L311 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L311 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L312 EN**: Continues the surrounding expression or declaration: `static const T P[19] = {`.
  - **L312 CN**: 继续构造周围的表达式或声明：`static const T P[19] = {`。

### Lines 313-336 / 第 313-336 行

````cpp
 313:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.559148411832951463689610809550083986e-40),
 314:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.999999999999999999999999999999999997),
 315:          BOOST_MATH_BIG_CONSTANT(T, 113, -166.542326331163836642960118190147367),
 316:          BOOST_MATH_BIG_CONSTANT(T, 113, -12204.639128796330005065904675153652),
 317:          BOOST_MATH_BIG_CONSTANT(T, 113, -520807.069767086071806275022036146855),
 318:          BOOST_MATH_BIG_CONSTANT(T, 113, -14435981.5242137970691490903863125326),
 319:          BOOST_MATH_BIG_CONSTANT(T, 113, -274574945.737064301247496460758654196),
 320:          BOOST_MATH_BIG_CONSTANT(T, 113, -3691611582.99810039356254671781473079),
 321:          BOOST_MATH_BIG_CONSTANT(T, 113, -35622515944.8255047299363690814678763),
 322:          BOOST_MATH_BIG_CONSTANT(T, 113, -248040014774.502043161750715548451142),
 323:          BOOST_MATH_BIG_CONSTANT(T, 113, -1243190389769.53458416330946622607913),
 324:          BOOST_MATH_BIG_CONSTANT(T, 113, -4441730126135.54739052731990368425339),
 325:          BOOST_MATH_BIG_CONSTANT(T, 113, -11117043181899.7388524310281751971366),
 326:          BOOST_MATH_BIG_CONSTANT(T, 113, -18976497615396.9717776601813519498961),
 327:          BOOST_MATH_BIG_CONSTANT(T, 113, -21237496819711.1011661104761906067131),
 328:          BOOST_MATH_BIG_CONSTANT(T, 113, -14695899122092.5161620333466757812848),
 329:          BOOST_MATH_BIG_CONSTANT(T, 113, -5737221535080.30569711574295785864903),
 330:          BOOST_MATH_BIG_CONSTANT(T, 113, -1077042281708.42654526404581272546244),
 331:          BOOST_MATH_BIG_CONSTANT(T, 113, -68028222642.1941480871395695677675137)
 332:       };
 333:       static const T Q[20] = {
 334:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 335:          BOOST_MATH_BIG_CONSTANT(T, 113, 168.542326331163836642960118190147311),
 336:          BOOST_MATH_BIG_CONSTANT(T, 113, 12535.7237814586576783518249115343619),
````
- **L313 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L313 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L314 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L314 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L315 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L315 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L316 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L316 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L317 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L317 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L318 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L318 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L319 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L319 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L320 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L320 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L321 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L321 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L322 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L322 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L323 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L323 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L324 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L324 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L325 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L325 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L326 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L326 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L327 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L327 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L328 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L328 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L329 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L329 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L330 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L330 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L331 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L331 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L332 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L332 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L333 EN**: Continues the surrounding expression or declaration: `static const T Q[20] = {`.
  - **L333 CN**: 继续构造周围的表达式或声明：`static const T Q[20] = {`。
- **L334 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L334 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L335 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L335 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L336 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L336 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 337-360 / 第 337-360 行

````cpp
 337:          BOOST_MATH_BIG_CONSTANT(T, 113, 544891.263372016404143120911148640627),
 338:          BOOST_MATH_BIG_CONSTANT(T, 113, 15454474.7241010258634446523045237762),
 339:          BOOST_MATH_BIG_CONSTANT(T, 113, 302495899.896629522673410325891717381),
 340:          BOOST_MATH_BIG_CONSTANT(T, 113, 4215565948.38886507646911672693270307),
 341:          BOOST_MATH_BIG_CONSTANT(T, 113, 42552409471.7951815668506556705733344),
 342:          BOOST_MATH_BIG_CONSTANT(T, 113, 313592377066.753173979584098301610186),
 343:          BOOST_MATH_BIG_CONSTANT(T, 113, 1688763640223.4541980740597514904542),
 344:          BOOST_MATH_BIG_CONSTANT(T, 113, 6610992294901.59589748057620192145704),
 345:          BOOST_MATH_BIG_CONSTANT(T, 113, 18601637235659.6059890851321772682606),
 346:          BOOST_MATH_BIG_CONSTANT(T, 113, 36944278231087.2571020964163402941583),
 347:          BOOST_MATH_BIG_CONSTANT(T, 113, 50425858518481.7497071917028793820058),
 348:          BOOST_MATH_BIG_CONSTANT(T, 113, 45508060902865.0899967797848815980644),
 349:          BOOST_MATH_BIG_CONSTANT(T, 113, 25649955002765.3817331501988304758142),
 350:          BOOST_MATH_BIG_CONSTANT(T, 113, 8259575619094.6518520988612711292331),
 351:          BOOST_MATH_BIG_CONSTANT(T, 113, 1299981487496.12607474362723586264515),
 352:          BOOST_MATH_BIG_CONSTANT(T, 113, 70242279152.8241187845178443118302693),
 353:          BOOST_MATH_BIG_CONSTANT(T, 113, -37633302.9409263839042721539363416685)
 354:       };
 355:       // LCOV_EXCL_STOP
 356:       T recip = 1 / z;
 357:       result = 1 + tools::evaluate_polynomial(P, recip)
 358:          / tools::evaluate_polynomial(Q, recip);
 359:       result *= exp(-z) * recip;
 360:    }
````
- **L337 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L337 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L338 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L338 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L339 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L339 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L340 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L340 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L341 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L341 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L342 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L342 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L343 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L343 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L344 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L344 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L345 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L345 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L346 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L346 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L347 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L347 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L348 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L348 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L349 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L349 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L350 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L350 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L351 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L351 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L352 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L352 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L353 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L353 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L354 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L354 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L355 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L355 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L356 EN**: Executes a standalone statement or declaration: `T recip = 1 / z;`.
  - **L356 CN**: 执行一条独立语句或声明：`T recip = 1 / z;`。
- **L357 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L357 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L358 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L358 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L359 EN**: Executes a call or declaration centered on `exp`.
  - **L359 CN**: 执行以 `exp` 为核心的调用或声明。
- **L360 EN**: Closes the current lexical scope or compound statement.
  - **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384 / 第 361-384 行

````cpp
 361:    else
 362:    {
 363:       result = 0;
 364:    }
 365:    return result;
 366: }
 367: 
 368: 
 369: template <class T>
 370: struct expint_fraction
 371: {
 372:    typedef boost::math::pair<T,T> result_type;
 373:    BOOST_MATH_GPU_ENABLED expint_fraction(unsigned n_, T z_) : b(n_ + z_), i(-1), n(n_){}
 374:    BOOST_MATH_GPU_ENABLED boost::math::pair<T,T> operator()()
 375:    {
 376:       boost::math::pair<T,T> result = boost::math::make_pair(-static_cast<T>((i+1) * (n+i)), b);
 377:       b += 2;
 378:       ++i;
 379:       return result;
 380:    }
 381: private:
 382:    T b;
 383:    int i;
 384:    unsigned n;
````
- **L361 EN**: Starts the alternative branch of the preceding conditional.
  - **L361 CN**: 开始前一个条件语句的备选分支。
- **L362 EN**: Opens a new lexical scope or compound statement.
  - **L362 CN**: 打开一个新的词法作用域或复合语句块。
- **L363 EN**: Executes a standalone statement or declaration: `result = 0;`.
  - **L363 CN**: 执行一条独立语句或声明：`result = 0;`。
- **L364 EN**: Closes the current lexical scope or compound statement.
  - **L364 CN**: 结束当前词法作用域或复合语句块。
- **L365 EN**: Returns from the current function with `result`.
  - **L365 CN**: 以 `result` 从当前函数返回。
- **L366 EN**: Closes the current lexical scope or compound statement.
  - **L366 CN**: 结束当前词法作用域或复合语句块。
- **L367 EN**: Blank line separating nearby declarations or logic.
  - **L367 CN**: 空行，用于分隔相邻声明或逻辑。
- **L368 EN**: Blank line separating nearby declarations or logic.
  - **L368 CN**: 空行，用于分隔相邻声明或逻辑。
- **L369 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L369 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L370 EN**: Declares struct `expint_fraction`.
  - **L370 CN**: 声明 struct `expint_fraction`。
- **L371 EN**: Opens a new lexical scope or compound statement.
  - **L371 CN**: 打开一个新的词法作用域或复合语句块。
- **L372 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::pair<T,T> result_type;`.
  - **L372 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::pair<T,T> result_type;`。
- **L373 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L373 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L374 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L374 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L375 EN**: Opens a new lexical scope or compound statement.
  - **L375 CN**: 打开一个新的词法作用域或复合语句块。
- **L376 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L376 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L377 EN**: Executes a standalone statement or declaration: `b += 2;`.
  - **L377 CN**: 执行一条独立语句或声明：`b += 2;`。
- **L378 EN**: Executes a standalone statement or declaration: `++i;`.
  - **L378 CN**: 执行一条独立语句或声明：`++i;`。
- **L379 EN**: Returns from the current function with `result`.
  - **L379 CN**: 以 `result` 从当前函数返回。
- **L380 EN**: Closes the current lexical scope or compound statement.
  - **L380 CN**: 结束当前词法作用域或复合语句块。
- **L381 EN**: Sets the following members to `private` access.
  - **L381 CN**: 将后续成员的访问级别设为 `private`。
- **L382 EN**: Executes a standalone statement or declaration: `T b;`.
  - **L382 CN**: 执行一条独立语句或声明：`T b;`。
- **L383 EN**: Executes a standalone statement or declaration: `int i;`.
  - **L383 CN**: 执行一条独立语句或声明：`int i;`。
- **L384 EN**: Executes a standalone statement or declaration: `unsigned n;`.
  - **L384 CN**: 执行一条独立语句或声明：`unsigned n;`。

### Lines 385-408 / 第 385-408 行

````cpp
 385: };
 386: 
 387: template <class T, class Policy>
 388: BOOST_MATH_GPU_ENABLED inline T expint_as_fraction(unsigned n, T z, const Policy& pol)
 389: {
 390:    BOOST_MATH_STD_USING
 391:    BOOST_MATH_INSTRUMENT_VARIABLE(z)
 392:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 393:    expint_fraction<T> f(n, z);
 394:    T result = tools::continued_fraction_b(
 395:       f,
 396:       boost::math::policies::get_epsilon<T, Policy>(),
 397:       max_iter);
 398:    policies::check_series_iterations<T>("boost::math::expint_continued_fraction<%1%>(unsigned,%1%)", max_iter, pol);
 399:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
 400:    BOOST_MATH_INSTRUMENT_VARIABLE(max_iter)
 401:    result = exp(-z) / result;
 402:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
 403:    return result;
 404: }
 405: 
 406: template <class T>
 407: struct expint_series
 408: {
````
- **L385 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L385 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L386 EN**: Blank line separating nearby declarations or logic.
  - **L386 CN**: 空行，用于分隔相邻声明或逻辑。
- **L387 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L387 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L388 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L388 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L389 EN**: Opens a new lexical scope or compound statement.
  - **L389 CN**: 打开一个新的词法作用域或复合语句块。
- **L390 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L390 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L391 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L391 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L392 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L392 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L393 EN**: Executes a call or declaration centered on `f`.
  - **L393 CN**: 执行以 `f` 为核心的调用或声明。
- **L394 EN**: Continues logic associated with callable symbol `continued_fraction_b`.
  - **L394 CN**: 继续与可调用符号 `continued_fraction_b` 相关的逻辑。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `f,`.
  - **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`f,`。
- **L396 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L396 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L397 EN**: Executes a standalone statement or declaration: `max_iter);`.
  - **L397 CN**: 执行一条独立语句或声明：`max_iter);`。
- **L398 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L398 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L399 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L399 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L400 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L400 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L401 EN**: Executes a call or declaration centered on `exp`.
  - **L401 CN**: 执行以 `exp` 为核心的调用或声明。
- **L402 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L402 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L403 EN**: Returns from the current function with `result`.
  - **L403 CN**: 以 `result` 从当前函数返回。
- **L404 EN**: Closes the current lexical scope or compound statement.
  - **L404 CN**: 结束当前词法作用域或复合语句块。
- **L405 EN**: Blank line separating nearby declarations or logic.
  - **L405 CN**: 空行，用于分隔相邻声明或逻辑。
- **L406 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L406 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L407 EN**: Declares struct `expint_series`.
  - **L407 CN**: 声明 struct `expint_series`。
- **L408 EN**: Opens a new lexical scope or compound statement.
  - **L408 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 409-432 / 第 409-432 行

````cpp
 409:    typedef T result_type;
 410:    BOOST_MATH_GPU_ENABLED expint_series(unsigned k_, T z_, T x_k_, T denom_, T fact_)
 411:       : k(k_), z(z_), x_k(x_k_), denom(denom_), fact(fact_){}
 412:    BOOST_MATH_GPU_ENABLED T operator()()
 413:    {
 414:       x_k *= -z;
 415:       denom += 1;
 416:       fact *= ++k;
 417:       return x_k / (denom * fact);
 418:    }
 419: private:
 420:    unsigned k;
 421:    T z;
 422:    T x_k;
 423:    T denom;
 424:    T fact;
 425: };
 426: 
 427: template <class T, class Policy>
 428: BOOST_MATH_GPU_ENABLED inline T expint_as_series(unsigned n, T z, const Policy& pol)
 429: {
 430:    BOOST_MATH_STD_USING
 431:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 432: 
````
- **L409 EN**: Introduces a legacy type alias or function typedef: `typedef T result_type;`.
  - **L409 CN**: 引入传统类型别名或函数 typedef：`typedef T result_type;`。
- **L410 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L410 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L411 EN**: Continues logic associated with callable symbol `k`.
  - **L411 CN**: 继续与可调用符号 `k` 相关的逻辑。
- **L412 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L412 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L413 EN**: Opens a new lexical scope or compound statement.
  - **L413 CN**: 打开一个新的词法作用域或复合语句块。
- **L414 EN**: Executes a standalone statement or declaration: `x_k *= -z;`.
  - **L414 CN**: 执行一条独立语句或声明：`x_k *= -z;`。
- **L415 EN**: Executes a standalone statement or declaration: `denom += 1;`.
  - **L415 CN**: 执行一条独立语句或声明：`denom += 1;`。
- **L416 EN**: Executes a standalone statement or declaration: `fact *= ++k;`.
  - **L416 CN**: 执行一条独立语句或声明：`fact *= ++k;`。
- **L417 EN**: Returns from the current function with `x_k / (denom * fact)`.
  - **L417 CN**: 以 `x_k / (denom * fact)` 从当前函数返回。
- **L418 EN**: Closes the current lexical scope or compound statement.
  - **L418 CN**: 结束当前词法作用域或复合语句块。
- **L419 EN**: Sets the following members to `private` access.
  - **L419 CN**: 将后续成员的访问级别设为 `private`。
- **L420 EN**: Executes a standalone statement or declaration: `unsigned k;`.
  - **L420 CN**: 执行一条独立语句或声明：`unsigned k;`。
- **L421 EN**: Executes a standalone statement or declaration: `T z;`.
  - **L421 CN**: 执行一条独立语句或声明：`T z;`。
- **L422 EN**: Executes a standalone statement or declaration: `T x_k;`.
  - **L422 CN**: 执行一条独立语句或声明：`T x_k;`。
- **L423 EN**: Executes a standalone statement or declaration: `T denom;`.
  - **L423 CN**: 执行一条独立语句或声明：`T denom;`。
- **L424 EN**: Executes a standalone statement or declaration: `T fact;`.
  - **L424 CN**: 执行一条独立语句或声明：`T fact;`。
- **L425 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L425 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L426 EN**: Blank line separating nearby declarations or logic.
  - **L426 CN**: 空行，用于分隔相邻声明或逻辑。
- **L427 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L427 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L428 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L428 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L429 EN**: Opens a new lexical scope or compound statement.
  - **L429 CN**: 打开一个新的词法作用域或复合语句块。
- **L430 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L430 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L431 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L431 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L432 EN**: Blank line separating nearby declarations or logic.
  - **L432 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 433-456 / 第 433-456 行

````cpp
 433:    BOOST_MATH_INSTRUMENT_VARIABLE(z)
 434: 
 435:    T result = 0;
 436:    T x_k = -1;
 437:    T denom = T(1) - n;
 438:    T fact = 1;
 439:    unsigned k = 0;
 440:    for(; k < n - 1;)
 441:    {
 442:       result += x_k / (denom * fact);
 443:       denom += 1;
 444:       x_k *= -z;
 445:       fact *= ++k;
 446:    }
 447:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
 448:    result += pow(-z, static_cast<T>(n - 1))
 449:       * (boost::math::digamma(static_cast<T>(n), pol) - log(z)) / fact;
 450:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
 451: 
 452:    expint_series<T> s(k, z, x_k, denom, fact);
 453:    result = tools::sum_series(s, policies::get_epsilon<T, Policy>(), max_iter, result);
 454:    policies::check_series_iterations<T>("boost::math::expint_series<%1%>(unsigned,%1%)", max_iter, pol);
 455:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
 456:    BOOST_MATH_INSTRUMENT_VARIABLE(max_iter)
````
- **L433 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L433 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L434 EN**: Blank line separating nearby declarations or logic.
  - **L434 CN**: 空行，用于分隔相邻声明或逻辑。
- **L435 EN**: Executes a standalone statement or declaration: `T result = 0;`.
  - **L435 CN**: 执行一条独立语句或声明：`T result = 0;`。
- **L436 EN**: Executes a standalone statement or declaration: `T x_k = -1;`.
  - **L436 CN**: 执行一条独立语句或声明：`T x_k = -1;`。
- **L437 EN**: Executes a call or declaration centered on `T`.
  - **L437 CN**: 执行以 `T` 为核心的调用或声明。
- **L438 EN**: Executes a standalone statement or declaration: `T fact = 1;`.
  - **L438 CN**: 执行一条独立语句或声明：`T fact = 1;`。
- **L439 EN**: Initializes variable `k` from the right-hand expression.
  - **L439 CN**: 使用右侧表达式初始化变量 `k`。
- **L440 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L440 CN**: 开始 `for` 控制流语句并计算其条件。
- **L441 EN**: Opens a new lexical scope or compound statement.
  - **L441 CN**: 打开一个新的词法作用域或复合语句块。
- **L442 EN**: Executes a call or declaration centered on `/`.
  - **L442 CN**: 执行以 `/` 为核心的调用或声明。
- **L443 EN**: Executes a standalone statement or declaration: `denom += 1;`.
  - **L443 CN**: 执行一条独立语句或声明：`denom += 1;`。
- **L444 EN**: Executes a standalone statement or declaration: `x_k *= -z;`.
  - **L444 CN**: 执行一条独立语句或声明：`x_k *= -z;`。
- **L445 EN**: Executes a standalone statement or declaration: `fact *= ++k;`.
  - **L445 CN**: 执行一条独立语句或声明：`fact *= ++k;`。
- **L446 EN**: Closes the current lexical scope or compound statement.
  - **L446 CN**: 结束当前词法作用域或复合语句块。
- **L447 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L447 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L448 EN**: Continues logic associated with callable symbol `pow`.
  - **L448 CN**: 继续与可调用符号 `pow` 相关的逻辑。
- **L449 EN**: Comment documents nearby intent or usage notes: `(boost::math::digamma(static_cast<T>(n), pol) - log(z)) / fact;`.
  - **L449 CN**: 注释说明附近代码的意图或使用说明：`(boost::math::digamma(static_cast<T>(n), pol) - log(z)) / fact;`。
- **L450 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L450 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L451 EN**: Blank line separating nearby declarations or logic.
  - **L451 CN**: 空行，用于分隔相邻声明或逻辑。
- **L452 EN**: Executes a call or declaration centered on `s`.
  - **L452 CN**: 执行以 `s` 为核心的调用或声明。
- **L453 EN**: Executes a call or declaration centered on `tools::sum_series`.
  - **L453 CN**: 执行以 `tools::sum_series` 为核心的调用或声明。
- **L454 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L454 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L455 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L455 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L456 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L456 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 457-480 / 第 457-480 行

````cpp
 457:    return result;
 458: }
 459: 
 460: template <class T, class Policy, class Tag>
 461: BOOST_MATH_GPU_ENABLED T expint_imp(unsigned n, T z, const Policy& pol, const Tag& tag)
 462: {
 463:    BOOST_MATH_STD_USING
 464:    constexpr auto function = "boost::math::expint<%1%>(unsigned, %1%)";
 465:    if(z < 0)
 466:       return policies::raise_domain_error<T>(function, "Function requires z >= 0 but got %1%.", z, pol);
 467:    if(z == 0)
 468:       return n == 1 ? policies::raise_overflow_error<T>(function, nullptr, pol) : T(1 / (static_cast<T>(n - 1)));
 469: 
 470:    T result;
 471: 
 472:    bool f;
 473:    if(n < 3)
 474:    {
 475:       f = z < T(0.5);
 476:    }
 477:    else
 478:    {
 479:       f = z < (static_cast<T>(n - 2) / static_cast<T>(n - 1));
 480:    }
````
- **L457 EN**: Returns from the current function with `result`.
  - **L457 CN**: 以 `result` 从当前函数返回。
- **L458 EN**: Closes the current lexical scope or compound statement.
  - **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Blank line separating nearby declarations or logic.
  - **L459 CN**: 空行，用于分隔相邻声明或逻辑。
- **L460 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Tag>`.
  - **L460 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Tag>`。
- **L461 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L461 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L462 EN**: Opens a new lexical scope or compound statement.
  - **L462 CN**: 打开一个新的词法作用域或复合语句块。
- **L463 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L463 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L464 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L464 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L465 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L465 CN**: 开始 `if` 控制流语句并计算其条件。
- **L466 EN**: Returns from the current function with `policies::raise_domain_error<T>(function, "Function requires z >= 0 but got %1%.", z, pol)`.
  - **L466 CN**: 以 `policies::raise_domain_error<T>(function, "Function requires z >= 0 but got %1%.", z, pol)` 从当前函数返回。
- **L467 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L467 CN**: 开始 `if` 控制流语句并计算其条件。
- **L468 EN**: Returns from the current function with `n == 1 ? policies::raise_overflow_error<T>(function, nullptr, pol) : T(1 / (static_cast<T>(n - 1)))`.
  - **L468 CN**: 以 `n == 1 ? policies::raise_overflow_error<T>(function, nullptr, pol) : T(1 / (static_cast<T>(n - 1)))` 从当前函数返回。
- **L469 EN**: Blank line separating nearby declarations or logic.
  - **L469 CN**: 空行，用于分隔相邻声明或逻辑。
- **L470 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L470 CN**: 执行一条独立语句或声明：`T result;`。
- **L471 EN**: Blank line separating nearby declarations or logic.
  - **L471 CN**: 空行，用于分隔相邻声明或逻辑。
- **L472 EN**: Executes a standalone statement or declaration: `bool f;`.
  - **L472 CN**: 执行一条独立语句或声明：`bool f;`。
- **L473 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L473 CN**: 开始 `if` 控制流语句并计算其条件。
- **L474 EN**: Opens a new lexical scope or compound statement.
  - **L474 CN**: 打开一个新的词法作用域或复合语句块。
- **L475 EN**: Executes a call or declaration centered on `T`.
  - **L475 CN**: 执行以 `T` 为核心的调用或声明。
- **L476 EN**: Closes the current lexical scope or compound statement.
  - **L476 CN**: 结束当前词法作用域或复合语句块。
- **L477 EN**: Starts the alternative branch of the preceding conditional.
  - **L477 CN**: 开始前一个条件语句的备选分支。
- **L478 EN**: Opens a new lexical scope or compound statement.
  - **L478 CN**: 打开一个新的词法作用域或复合语句块。
- **L479 EN**: Executes a call or declaration centered on `<`.
  - **L479 CN**: 执行以 `<` 为核心的调用或声明。
- **L480 EN**: Closes the current lexical scope or compound statement.
  - **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-504 / 第 481-504 行

````cpp
 481: #ifdef _MSC_VER
 482: #  pragma warning(push)
 483: #  pragma warning(disable:4127) // conditional expression is constant
 484: #endif
 485:    if(n == 0)
 486:    {
 487:       result = exp(-z) / z;
 488:    }
 489:    else if((n == 1) && (Tag::value))
 490:    {
 491:       result = expint_1_rational(z, tag);
 492:    }
 493:    else if(f)
 494:    {
 495:       result = expint_as_series(n, z, pol);
 496:    }
 497:    else
 498:    {
 499:       result = expint_as_fraction(n, z, pol);
 500:    }
 501: #ifdef _MSC_VER
 502: #  pragma warning(pop)
 503: #endif
 504: 
````
- **L481 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L481 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L482 EN**: Continues logic associated with callable symbol `warning`.
  - **L482 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L483 EN**: Continues logic associated with callable symbol `warning`.
  - **L483 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L484 EN**: Closes the current preprocessor conditional block or header guard.
  - **L484 CN**: 结束当前预处理条件块或头文件保护。
- **L485 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L485 CN**: 开始 `if` 控制流语句并计算其条件。
- **L486 EN**: Opens a new lexical scope or compound statement.
  - **L486 CN**: 打开一个新的词法作用域或复合语句块。
- **L487 EN**: Executes a call or declaration centered on `exp`.
  - **L487 CN**: 执行以 `exp` 为核心的调用或声明。
- **L488 EN**: Closes the current lexical scope or compound statement.
  - **L488 CN**: 结束当前词法作用域或复合语句块。
- **L489 EN**: Starts the alternative branch of the preceding conditional.
  - **L489 CN**: 开始前一个条件语句的备选分支。
- **L490 EN**: Opens a new lexical scope or compound statement.
  - **L490 CN**: 打开一个新的词法作用域或复合语句块。
- **L491 EN**: Executes a call or declaration centered on `expint_1_rational`.
  - **L491 CN**: 执行以 `expint_1_rational` 为核心的调用或声明。
- **L492 EN**: Closes the current lexical scope or compound statement.
  - **L492 CN**: 结束当前词法作用域或复合语句块。
- **L493 EN**: Starts the alternative branch of the preceding conditional.
  - **L493 CN**: 开始前一个条件语句的备选分支。
- **L494 EN**: Opens a new lexical scope or compound statement.
  - **L494 CN**: 打开一个新的词法作用域或复合语句块。
- **L495 EN**: Executes a call or declaration centered on `expint_as_series`.
  - **L495 CN**: 执行以 `expint_as_series` 为核心的调用或声明。
- **L496 EN**: Closes the current lexical scope or compound statement.
  - **L496 CN**: 结束当前词法作用域或复合语句块。
- **L497 EN**: Starts the alternative branch of the preceding conditional.
  - **L497 CN**: 开始前一个条件语句的备选分支。
- **L498 EN**: Opens a new lexical scope or compound statement.
  - **L498 CN**: 打开一个新的词法作用域或复合语句块。
- **L499 EN**: Executes a call or declaration centered on `expint_as_fraction`.
  - **L499 CN**: 执行以 `expint_as_fraction` 为核心的调用或声明。
- **L500 EN**: Closes the current lexical scope or compound statement.
  - **L500 CN**: 结束当前词法作用域或复合语句块。
- **L501 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L501 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L502 EN**: Continues logic associated with callable symbol `warning`.
  - **L502 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L503 EN**: Closes the current preprocessor conditional block or header guard.
  - **L503 CN**: 结束当前预处理条件块或头文件保护。
- **L504 EN**: Blank line separating nearby declarations or logic.
  - **L504 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 505-528 / 第 505-528 行

````cpp
 505:    return result;
 506: }
 507: 
 508: template <class T>
 509: struct expint_i_series
 510: {
 511:    typedef T result_type;
 512:    BOOST_MATH_GPU_ENABLED expint_i_series(T z_) : k(0), z_k(1), z(z_){}
 513:    BOOST_MATH_GPU_ENABLED T operator()()
 514:    {
 515:       z_k *= z / ++k;
 516:       return z_k / k;
 517:    }
 518: private:
 519:    unsigned k;
 520:    T z_k;
 521:    T z;
 522: };
 523: 
 524: template <class T, class Policy>
 525: BOOST_MATH_GPU_ENABLED T expint_i_as_series(T z, const Policy& pol)
 526: {
 527:    BOOST_MATH_STD_USING
 528:    T result = log(z); // (log(z) - log(1 / z)) / 2;
````
- **L505 EN**: Returns from the current function with `result`.
  - **L505 CN**: 以 `result` 从当前函数返回。
- **L506 EN**: Closes the current lexical scope or compound statement.
  - **L506 CN**: 结束当前词法作用域或复合语句块。
- **L507 EN**: Blank line separating nearby declarations or logic.
  - **L507 CN**: 空行，用于分隔相邻声明或逻辑。
- **L508 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L508 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L509 EN**: Declares struct `expint_i_series`.
  - **L509 CN**: 声明 struct `expint_i_series`。
- **L510 EN**: Opens a new lexical scope or compound statement.
  - **L510 CN**: 打开一个新的词法作用域或复合语句块。
- **L511 EN**: Introduces a legacy type alias or function typedef: `typedef T result_type;`.
  - **L511 CN**: 引入传统类型别名或函数 typedef：`typedef T result_type;`。
- **L512 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L512 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L513 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L513 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L514 EN**: Opens a new lexical scope or compound statement.
  - **L514 CN**: 打开一个新的词法作用域或复合语句块。
- **L515 EN**: Executes a standalone statement or declaration: `z_k *= z / ++k;`.
  - **L515 CN**: 执行一条独立语句或声明：`z_k *= z / ++k;`。
- **L516 EN**: Returns from the current function with `z_k / k`.
  - **L516 CN**: 以 `z_k / k` 从当前函数返回。
- **L517 EN**: Closes the current lexical scope or compound statement.
  - **L517 CN**: 结束当前词法作用域或复合语句块。
- **L518 EN**: Sets the following members to `private` access.
  - **L518 CN**: 将后续成员的访问级别设为 `private`。
- **L519 EN**: Executes a standalone statement or declaration: `unsigned k;`.
  - **L519 CN**: 执行一条独立语句或声明：`unsigned k;`。
- **L520 EN**: Executes a standalone statement or declaration: `T z_k;`.
  - **L520 CN**: 执行一条独立语句或声明：`T z_k;`。
- **L521 EN**: Executes a standalone statement or declaration: `T z;`.
  - **L521 CN**: 执行一条独立语句或声明：`T z;`。
- **L522 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L522 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L523 EN**: Blank line separating nearby declarations or logic.
  - **L523 CN**: 空行，用于分隔相邻声明或逻辑。
- **L524 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L524 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L525 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L525 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L526 EN**: Opens a new lexical scope or compound statement.
  - **L526 CN**: 打开一个新的词法作用域或复合语句块。
- **L527 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L527 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L528 EN**: Executes a call or declaration centered on `log`.
  - **L528 CN**: 执行以 `log` 为核心的调用或声明。

### Lines 529-552 / 第 529-552 行

````cpp
 529:    result += constants::euler<T>();
 530:    expint_i_series<T> s(z);
 531:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 532:    result = tools::sum_series(s, policies::get_epsilon<T, Policy>(), max_iter, result);
 533:    policies::check_series_iterations<T>("boost::math::expint_i_series<%1%>(%1%)", max_iter, pol);
 534:    return result;
 535: }
 536: 
 537: template <class T, class Policy, class Tag>
 538: BOOST_MATH_GPU_ENABLED T expint_i_imp(T z, const Policy& pol, const Tag& tag)
 539: {
 540:    constexpr auto function = "boost::math::expint<%1%>(%1%)";
 541:    if(z < 0)
 542:       return -expint_imp(1, T(-z), pol, tag);
 543:    if(z == 0)
 544:       return -policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE confirmed covered by real_concept tests
 545:    return expint_i_as_series(z, pol);
 546: }
 547: 
 548: template <class T, class Policy>
 549: BOOST_MATH_GPU_ENABLED T expint_i_imp(T z, const Policy& pol, const boost::math::integral_constant<int, 53>& tag)
 550: {
 551:    BOOST_MATH_STD_USING
 552:    constexpr auto function = "boost::math::expint<%1%>(%1%)";
````
- **L529 EN**: Executes a call or declaration centered on `constants::euler<T>`.
  - **L529 CN**: 执行以 `constants::euler<T>` 为核心的调用或声明。
- **L530 EN**: Executes a call or declaration centered on `s`.
  - **L530 CN**: 执行以 `s` 为核心的调用或声明。
- **L531 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L531 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L532 EN**: Executes a call or declaration centered on `tools::sum_series`.
  - **L532 CN**: 执行以 `tools::sum_series` 为核心的调用或声明。
- **L533 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L533 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L534 EN**: Returns from the current function with `result`.
  - **L534 CN**: 以 `result` 从当前函数返回。
- **L535 EN**: Closes the current lexical scope or compound statement.
  - **L535 CN**: 结束当前词法作用域或复合语句块。
- **L536 EN**: Blank line separating nearby declarations or logic.
  - **L536 CN**: 空行，用于分隔相邻声明或逻辑。
- **L537 EN**: Introduces template parameters or specialization context: `template <class T, class Policy, class Tag>`.
  - **L537 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy, class Tag>`。
- **L538 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L538 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L539 EN**: Opens a new lexical scope or compound statement.
  - **L539 CN**: 打开一个新的词法作用域或复合语句块。
- **L540 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L540 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L541 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L541 CN**: 开始 `if` 控制流语句并计算其条件。
- **L542 EN**: Returns from the current function with `-expint_imp(1, T(-z), pol, tag)`.
  - **L542 CN**: 以 `-expint_imp(1, T(-z), pol, tag)` 从当前函数返回。
- **L543 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L543 CN**: 开始 `if` 控制流语句并计算其条件。
- **L544 EN**: Returns from the current function with `-policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE confirmed covered by real_concept tests`.
  - **L544 CN**: 以 `-policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE confirmed covered by real_concept tests` 从当前函数返回。
- **L545 EN**: Returns from the current function with `expint_i_as_series(z, pol)`.
  - **L545 CN**: 以 `expint_i_as_series(z, pol)` 从当前函数返回。
- **L546 EN**: Closes the current lexical scope or compound statement.
  - **L546 CN**: 结束当前词法作用域或复合语句块。
- **L547 EN**: Blank line separating nearby declarations or logic.
  - **L547 CN**: 空行，用于分隔相邻声明或逻辑。
- **L548 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L548 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L549 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L549 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L550 EN**: Opens a new lexical scope or compound statement.
  - **L550 CN**: 打开一个新的词法作用域或复合语句块。
- **L551 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L551 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L552 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L552 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 553-576 / 第 553-576 行

````cpp
 553:    if(z < 0)
 554:       return -expint_imp(1, T(-z), pol, tag);
 555:    if(z == 0)
 556:       return -policies::raise_overflow_error<T>(function, nullptr, pol);
 557: 
 558:    T result;
 559: 
 560:    if(z <= 6)
 561:    {
 562:       // Maximum Deviation Found:                     2.852e-18
 563:       // Expected Error Term:                         2.852e-18
 564:       // Max Error found at double precision =        Poly: 2.636335e-16   Cheb: 4.187027e-16
 565:       // LCOV_EXCL_START
 566:       BOOST_MATH_STATIC const T P[10] = {
 567:          BOOST_MATH_BIG_CONSTANT(T, 53, 2.98677224343598593013),
 568:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.356343618769377415068),
 569:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.780836076283730801839),
 570:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.114670926327032002811),
 571:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.0499434773576515260534),
 572:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.00726224593341228159561),
 573:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.00115478237227804306827),
 574:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.000116419523609765200999),
 575:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.798296365679269702435e-5),
 576:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.2777056254402008721e-6)
````
- **L553 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L553 CN**: 开始 `if` 控制流语句并计算其条件。
- **L554 EN**: Returns from the current function with `-expint_imp(1, T(-z), pol, tag)`.
  - **L554 CN**: 以 `-expint_imp(1, T(-z), pol, tag)` 从当前函数返回。
- **L555 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L555 CN**: 开始 `if` 控制流语句并计算其条件。
- **L556 EN**: Returns from the current function with `-policies::raise_overflow_error<T>(function, nullptr, pol)`.
  - **L556 CN**: 以 `-policies::raise_overflow_error<T>(function, nullptr, pol)` 从当前函数返回。
- **L557 EN**: Blank line separating nearby declarations or logic.
  - **L557 CN**: 空行，用于分隔相邻声明或逻辑。
- **L558 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L558 CN**: 执行一条独立语句或声明：`T result;`。
- **L559 EN**: Blank line separating nearby declarations or logic.
  - **L559 CN**: 空行，用于分隔相邻声明或逻辑。
- **L560 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L560 CN**: 开始 `if` 控制流语句并计算其条件。
- **L561 EN**: Opens a new lexical scope or compound statement.
  - **L561 CN**: 打开一个新的词法作用域或复合语句块。
- **L562 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     2.852e-18`.
  - **L562 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     2.852e-18`。
- **L563 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         2.852e-18`.
  - **L563 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         2.852e-18`。
- **L564 EN**: Comment documents nearby intent or usage notes: `Max Error found at double precision =        Poly: 2.636335e-16   Cheb: 4.187027e-16`.
  - **L564 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at double precision =        Poly: 2.636335e-16   Cheb: 4.187027e-16`。
- **L565 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L565 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L566 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L566 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L567 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L567 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L568 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L568 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L569 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L569 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L570 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L570 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L571 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L571 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L572 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L572 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L573 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L573 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L574 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L574 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L575 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L575 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L576 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L576 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 577-600 / 第 577-600 行

````cpp
 577:       };
 578:       BOOST_MATH_STATIC const T Q[8] = {
 579:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
 580:          BOOST_MATH_BIG_CONSTANT(T, 53, -1.17090412365413911947),
 581:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.62215109846016746276),
 582:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.195114782069495403315),
 583:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.0391523431392967238166),
 584:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00504800158663705747345),
 585:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.000389034007436065401822),
 586:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.138972589601781706598e-4)
 587:       };
 588: 
 589:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T c1 = BOOST_MATH_BIG_CONSTANT(T, 53, 1677624236387711.0);
 590:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T c2 = BOOST_MATH_BIG_CONSTANT(T, 53, 4503599627370496.0);
 591:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T r1 = static_cast<T>(c1 / c2);
 592:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T r2 = BOOST_MATH_BIG_CONSTANT(T, 53, 0.131401834143860282009280387409357165515556574352422001206362e-16);
 593:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T r = static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 53, 0.372507410781366634461991866580119133535689497771654051555657435242200120636201854384926049951548942392));
 594:       // LCOV_EXCL_STOP
 595:       T t = (z / 3) - 1;
 596:       result = tools::evaluate_polynomial(P, t)
 597:          / tools::evaluate_polynomial(Q, t);
 598:       t = (z - r1) - r2;
 599:       result *= t;
 600:       if(fabs(t) < T(0.1))
````
- **L577 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L577 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L578 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L578 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L579 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L579 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L580 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L580 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L581 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L581 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L582 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L582 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L583 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L583 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L584 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L584 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L585 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L585 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L586 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L586 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L587 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L587 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L588 EN**: Blank line separating nearby declarations or logic.
  - **L588 CN**: 空行，用于分隔相邻声明或逻辑。
- **L589 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L589 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L590 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L590 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L591 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L591 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L592 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L592 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L593 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L593 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L594 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L594 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L595 EN**: Executes a call or declaration centered on `=`.
  - **L595 CN**: 执行以 `=` 为核心的调用或声明。
- **L596 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L596 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L597 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L597 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L598 EN**: Executes a call or declaration centered on `=`.
  - **L598 CN**: 执行以 `=` 为核心的调用或声明。
- **L599 EN**: Executes a standalone statement or declaration: `result *= t;`.
  - **L599 CN**: 执行一条独立语句或声明：`result *= t;`。
- **L600 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L600 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 601-624 / 第 601-624 行

````cpp
 601:       {
 602:          result += boost::math::log1p(t / r, pol);
 603:       }
 604:       else
 605:       {
 606:          result += log(z / r);
 607:       }
 608:    }
 609:    else if (z <= 10)
 610:    {
 611:       // Maximum Deviation Found:                     6.546e-17
 612:       // Expected Error Term:                         6.546e-17
 613:       // Max Error found at double precision =        Poly: 6.890169e-17   Cheb: 6.772128e-17
 614:       // LCOV_EXCL_START
 615:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T Y = 1.158985137939453125F;
 616:       BOOST_MATH_STATIC const T P[8] = {
 617:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.00139324086199402804173),
 618:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0349921221823888744966),
 619:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0264095520754134848538),
 620:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00761224003005476438412),
 621:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00247496209592143627977),
 622:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.000374885917942100256775),
 623:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.554086272024881826253e-4),
 624:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.396487648924804510056e-5)
````
- **L601 EN**: Opens a new lexical scope or compound statement.
  - **L601 CN**: 打开一个新的词法作用域或复合语句块。
- **L602 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L602 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L603 EN**: Closes the current lexical scope or compound statement.
  - **L603 CN**: 结束当前词法作用域或复合语句块。
- **L604 EN**: Starts the alternative branch of the preceding conditional.
  - **L604 CN**: 开始前一个条件语句的备选分支。
- **L605 EN**: Opens a new lexical scope or compound statement.
  - **L605 CN**: 打开一个新的词法作用域或复合语句块。
- **L606 EN**: Executes a call or declaration centered on `log`.
  - **L606 CN**: 执行以 `log` 为核心的调用或声明。
- **L607 EN**: Closes the current lexical scope or compound statement.
  - **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Closes the current lexical scope or compound statement.
  - **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Starts the alternative branch of the preceding conditional.
  - **L609 CN**: 开始前一个条件语句的备选分支。
- **L610 EN**: Opens a new lexical scope or compound statement.
  - **L610 CN**: 打开一个新的词法作用域或复合语句块。
- **L611 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     6.546e-17`.
  - **L611 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     6.546e-17`。
- **L612 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         6.546e-17`.
  - **L612 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         6.546e-17`。
- **L613 EN**: Comment documents nearby intent or usage notes: `Max Error found at double precision =        Poly: 6.890169e-17   Cheb: 6.772128e-17`.
  - **L613 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at double precision =        Poly: 6.890169e-17   Cheb: 6.772128e-17`。
- **L614 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L614 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L615 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L615 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L616 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L616 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L617 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L617 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L618 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L618 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L619 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L619 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L620 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L620 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L621 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L621 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L622 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L622 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L623 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L623 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L624 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L624 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 625-648 / 第 625-648 行

````cpp
 625:       };
 626:       BOOST_MATH_STATIC const T Q[8] = {
 627:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
 628:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.744625566823272107711),
 629:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.329061095011767059236),
 630:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.100128624977313872323),
 631:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.0223851099128506347278),
 632:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.00365334190742316650106),
 633:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.000402453408512476836472),
 634:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.263649630720255691787e-4)
 635:       };
 636:       // LCOV_EXCL_STOP
 637:       T t = z / 2 - 4;
 638:       result = Y + tools::evaluate_polynomial(P, t)
 639:          / tools::evaluate_polynomial(Q, t);
 640:       result *= exp(z) / z;
 641:       result += z;
 642:    }
 643:    else if(z <= 20)
 644:    {
 645:       // Maximum Deviation Found:                     1.843e-17
 646:       // Expected Error Term:                         -1.842e-17
 647:       // Max Error found at double precision =        Poly: 4.375868e-17   Cheb: 5.860967e-17
 648:       // LCOV_EXCL_START
````
- **L625 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L625 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L626 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L626 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L627 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L627 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L628 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L628 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L629 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L629 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L630 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L630 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L631 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L631 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L632 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L632 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L633 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L633 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L634 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L634 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L635 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L635 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L636 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L636 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L637 EN**: Executes a standalone statement or declaration: `T t = z / 2 - 4;`.
  - **L637 CN**: 执行一条独立语句或声明：`T t = z / 2 - 4;`。
- **L638 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L638 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L639 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L639 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L640 EN**: Executes a call or declaration centered on `exp`.
  - **L640 CN**: 执行以 `exp` 为核心的调用或声明。
- **L641 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L641 CN**: 执行一条独立语句或声明：`result += z;`。
- **L642 EN**: Closes the current lexical scope or compound statement.
  - **L642 CN**: 结束当前词法作用域或复合语句块。
- **L643 EN**: Starts the alternative branch of the preceding conditional.
  - **L643 CN**: 开始前一个条件语句的备选分支。
- **L644 EN**: Opens a new lexical scope or compound statement.
  - **L644 CN**: 打开一个新的词法作用域或复合语句块。
- **L645 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     1.843e-17`.
  - **L645 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     1.843e-17`。
- **L646 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         -1.842e-17`.
  - **L646 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         -1.842e-17`。
- **L647 EN**: Comment documents nearby intent or usage notes: `Max Error found at double precision =        Poly: 4.375868e-17   Cheb: 5.860967e-17`.
  - **L647 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at double precision =        Poly: 4.375868e-17   Cheb: 5.860967e-17`。
- **L648 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L648 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。

### Lines 649-672 / 第 649-672 行

````cpp
 649:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T Y = 1.0869731903076171875F;
 650:       BOOST_MATH_STATIC const T P[9] = {
 651:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00893891094356945667451),
 652:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0484607730127134045806),
 653:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0652810444222236895772),
 654:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0478447572647309671455),
 655:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0226059218923777094596),
 656:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00720603636917482065907),
 657:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00155941947035972031334),
 658:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.000209750022660200888349),
 659:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.138652200349182596186e-4)
 660:       };
 661:       BOOST_MATH_STATIC const T Q[9] = {
 662:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
 663:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.97017214039061194971),
 664:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.86232465043073157508),
 665:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.09601437090337519977),
 666:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.438873285773088870812),
 667:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.122537731979686102756),
 668:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.0233458478275769288159),
 669:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.00278170769163303669021),
 670:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.000159150281166108755531)
 671:       };
 672:       // LCOV_EXCL_STOP
````
- **L649 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L649 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L650 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L650 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L651 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L651 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L652 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L652 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L653 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L653 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L654 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L654 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L655 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L655 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L656 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L656 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L657 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L657 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L658 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L658 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L659 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L659 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L660 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L660 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L661 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L661 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L662 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L662 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L663 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L663 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L664 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L664 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L665 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L665 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L666 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L666 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L667 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L667 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L668 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L668 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L669 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L669 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L670 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L670 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L671 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L671 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L672 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L672 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。

### Lines 673-696 / 第 673-696 行

````cpp
 673:       T t = z / 5 - 3;
 674:       result = Y + tools::evaluate_polynomial(P, t)
 675:          / tools::evaluate_polynomial(Q, t);
 676:       result *= exp(z) / z;
 677:       result += z;
 678:    }
 679:    else if(z <= 40)
 680:    {
 681:       // Maximum Deviation Found:                     5.102e-18
 682:       // Expected Error Term:                         5.101e-18
 683:       // Max Error found at double precision =        Poly: 1.441088e-16   Cheb: 1.864792e-16
 684:       // LCOV_EXCL_START
 685:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T Y = 1.03937530517578125F;
 686:       BOOST_MATH_STATIC const T P[9] = {
 687:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00356165148914447597995),
 688:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0229930320357982333406),
 689:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0449814350482277917716),
 690:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0453759383048193402336),
 691:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0272050837209380717069),
 692:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00994403059883350813295),
 693:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.00207592267812291726961),
 694:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.000192178045857733706044),
 695:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.113161784705911400295e-9)
 696:       };
````
- **L673 EN**: Executes a standalone statement or declaration: `T t = z / 5 - 3;`.
  - **L673 CN**: 执行一条独立语句或声明：`T t = z / 5 - 3;`。
- **L674 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L674 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L675 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L675 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L676 EN**: Executes a call or declaration centered on `exp`.
  - **L676 CN**: 执行以 `exp` 为核心的调用或声明。
- **L677 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L677 CN**: 执行一条独立语句或声明：`result += z;`。
- **L678 EN**: Closes the current lexical scope or compound statement.
  - **L678 CN**: 结束当前词法作用域或复合语句块。
- **L679 EN**: Starts the alternative branch of the preceding conditional.
  - **L679 CN**: 开始前一个条件语句的备选分支。
- **L680 EN**: Opens a new lexical scope or compound statement.
  - **L680 CN**: 打开一个新的词法作用域或复合语句块。
- **L681 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     5.102e-18`.
  - **L681 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     5.102e-18`。
- **L682 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         5.101e-18`.
  - **L682 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         5.101e-18`。
- **L683 EN**: Comment documents nearby intent or usage notes: `Max Error found at double precision =        Poly: 1.441088e-16   Cheb: 1.864792e-16`.
  - **L683 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at double precision =        Poly: 1.441088e-16   Cheb: 1.864792e-16`。
- **L684 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L684 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L685 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L685 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L686 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L686 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L687 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L687 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L688 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L688 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L689 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L689 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L690 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L690 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L691 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L691 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L692 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L692 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L693 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L693 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L694 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L694 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L695 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L695 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L696 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L696 CN**: 结束当前声明作用域，例如类、结构体或枚举。

### Lines 697-720 / 第 697-720 行

````cpp
 697:       BOOST_MATH_STATIC const T Q[9] = {
 698:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
 699:          BOOST_MATH_BIG_CONSTANT(T, 53, 2.84354408840148561131),
 700:          BOOST_MATH_BIG_CONSTANT(T, 53, 3.6599610090072393012),
 701:          BOOST_MATH_BIG_CONSTANT(T, 53, 2.75088464344293083595),
 702:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.2985244073998398643),
 703:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.383213198510794507409),
 704:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.0651165455496281337831),
 705:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.00488071077519227853585)
 706:       };
 707:       // LCOV_EXCL_STOP
 708:       T t = z / 10 - 3;
 709:       result = Y + tools::evaluate_polynomial(P, t)
 710:          / tools::evaluate_polynomial(Q, t);
 711:       result *= exp(z) / z;
 712:       result += z;
 713:    }
 714:    else
 715:    {
 716:       // Max Error found at double precision =        3.381886e-17
 717:       // LCOV_EXCL_START
 718:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T exp40 = static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 53, 2.35385266837019985407899910749034804508871617254555467236651e17));
 719:       BOOST_MATH_STATIC_LOCAL_VARIABLE const T Y= 1.013065338134765625F;
 720:       BOOST_MATH_STATIC const T P[6] = {
````
- **L697 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L697 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L698 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L698 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L699 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L699 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L700 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L700 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L701 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L701 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L702 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L702 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L703 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L703 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L704 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L704 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L705 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L705 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L706 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L706 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L707 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L707 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L708 EN**: Executes a standalone statement or declaration: `T t = z / 10 - 3;`.
  - **L708 CN**: 执行一条独立语句或声明：`T t = z / 10 - 3;`。
- **L709 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L709 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L710 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L710 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L711 EN**: Executes a call or declaration centered on `exp`.
  - **L711 CN**: 执行以 `exp` 为核心的调用或声明。
- **L712 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L712 CN**: 执行一条独立语句或声明：`result += z;`。
- **L713 EN**: Closes the current lexical scope or compound statement.
  - **L713 CN**: 结束当前词法作用域或复合语句块。
- **L714 EN**: Starts the alternative branch of the preceding conditional.
  - **L714 CN**: 开始前一个条件语句的备选分支。
- **L715 EN**: Opens a new lexical scope or compound statement.
  - **L715 CN**: 打开一个新的词法作用域或复合语句块。
- **L716 EN**: Comment documents nearby intent or usage notes: `Max Error found at double precision =        3.381886e-17`.
  - **L716 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at double precision =        3.381886e-17`。
- **L717 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L717 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L718 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L718 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L719 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L719 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L720 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L720 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 721-744 / 第 721-744 行

````cpp
 721:          BOOST_MATH_BIG_CONSTANT(T, 53, -0.0130653381347656243849),
 722:          BOOST_MATH_BIG_CONSTANT(T, 53, 0.19029710559486576682),
 723:          BOOST_MATH_BIG_CONSTANT(T, 53, 94.7365094537197236011),
 724:          BOOST_MATH_BIG_CONSTANT(T, 53, -2516.35323679844256203),
 725:          BOOST_MATH_BIG_CONSTANT(T, 53, 18932.0850014925993025),
 726:          BOOST_MATH_BIG_CONSTANT(T, 53, -38703.1431362056714134)
 727:       };
 728:       BOOST_MATH_STATIC const T Q[7] = {
 729:          BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
 730:          BOOST_MATH_BIG_CONSTANT(T, 53, 61.9733592849439884145),
 731:          BOOST_MATH_BIG_CONSTANT(T, 53, -2354.56211323420194283),
 732:          BOOST_MATH_BIG_CONSTANT(T, 53, 22329.1459489893079041),
 733:          BOOST_MATH_BIG_CONSTANT(T, 53, -70126.245140396567133),
 734:          BOOST_MATH_BIG_CONSTANT(T, 53, 54738.2833147775537106),
 735:          BOOST_MATH_BIG_CONSTANT(T, 53, 8297.16296356518409347)
 736:       };
 737:       // LCOV_EXCL_STOP
 738:       T t = 1 / z;
 739:       result = Y + tools::evaluate_polynomial(P, t)
 740:          / tools::evaluate_polynomial(Q, t);
 741:       if(z < 41)
 742:          result *= exp(z) / z;
 743:       else
 744:       {
````
- **L721 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L721 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L722 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L722 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L723 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L723 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L724 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L724 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L725 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L725 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L726 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L726 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L727 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L727 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L728 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L728 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L729 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L729 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L730 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L730 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L731 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L731 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L732 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L732 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L733 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L733 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L734 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L734 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L735 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L735 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L736 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L736 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L737 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L737 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L738 EN**: Executes a standalone statement or declaration: `T t = 1 / z;`.
  - **L738 CN**: 执行一条独立语句或声明：`T t = 1 / z;`。
- **L739 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L739 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L740 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L740 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L741 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L741 CN**: 开始 `if` 控制流语句并计算其条件。
- **L742 EN**: Executes a call or declaration centered on `exp`.
  - **L742 CN**: 执行以 `exp` 为核心的调用或声明。
- **L743 EN**: Starts the alternative branch of the preceding conditional.
  - **L743 CN**: 开始前一个条件语句的备选分支。
- **L744 EN**: Opens a new lexical scope or compound statement.
  - **L744 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 745-768 / 第 745-768 行

````cpp
 745:          // Avoid premature overflow if we can:
 746:          t = z - 40;
 747:          if(t > tools::log_max_value<T>())
 748:          {
 749:             result = policies::raise_overflow_error<T>(function, nullptr, pol);
 750:          }
 751:          else
 752:          {
 753:             result *= exp(z - 40) / z;
 754:             if(result > tools::max_value<T>() / exp40)
 755:             {
 756:                result = policies::raise_overflow_error<T>(function, nullptr, pol);
 757:             }
 758:             else
 759:             {
 760:                result *= exp40;
 761:             }
 762:          }
 763:       }
 764:       result += z;
 765:    }
 766:    return result;
 767: }
 768: 
````
- **L745 EN**: Comment documents nearby intent or usage notes: `Avoid premature overflow if we can:`.
  - **L745 CN**: 注释说明附近代码的意图或使用说明：`Avoid premature overflow if we can:`。
- **L746 EN**: Executes a standalone statement or declaration: `t = z - 40;`.
  - **L746 CN**: 执行一条独立语句或声明：`t = z - 40;`。
- **L747 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L747 CN**: 开始 `if` 控制流语句并计算其条件。
- **L748 EN**: Opens a new lexical scope or compound statement.
  - **L748 CN**: 打开一个新的词法作用域或复合语句块。
- **L749 EN**: Executes a call or declaration centered on `policies::raise_overflow_error<T>`.
  - **L749 CN**: 执行以 `policies::raise_overflow_error<T>` 为核心的调用或声明。
- **L750 EN**: Closes the current lexical scope or compound statement.
  - **L750 CN**: 结束当前词法作用域或复合语句块。
- **L751 EN**: Starts the alternative branch of the preceding conditional.
  - **L751 CN**: 开始前一个条件语句的备选分支。
- **L752 EN**: Opens a new lexical scope or compound statement.
  - **L752 CN**: 打开一个新的词法作用域或复合语句块。
- **L753 EN**: Executes a call or declaration centered on `exp`.
  - **L753 CN**: 执行以 `exp` 为核心的调用或声明。
- **L754 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L754 CN**: 开始 `if` 控制流语句并计算其条件。
- **L755 EN**: Opens a new lexical scope or compound statement.
  - **L755 CN**: 打开一个新的词法作用域或复合语句块。
- **L756 EN**: Executes a call or declaration centered on `policies::raise_overflow_error<T>`.
  - **L756 CN**: 执行以 `policies::raise_overflow_error<T>` 为核心的调用或声明。
- **L757 EN**: Closes the current lexical scope or compound statement.
  - **L757 CN**: 结束当前词法作用域或复合语句块。
- **L758 EN**: Starts the alternative branch of the preceding conditional.
  - **L758 CN**: 开始前一个条件语句的备选分支。
- **L759 EN**: Opens a new lexical scope or compound statement.
  - **L759 CN**: 打开一个新的词法作用域或复合语句块。
- **L760 EN**: Executes a standalone statement or declaration: `result *= exp40;`.
  - **L760 CN**: 执行一条独立语句或声明：`result *= exp40;`。
- **L761 EN**: Closes the current lexical scope or compound statement.
  - **L761 CN**: 结束当前词法作用域或复合语句块。
- **L762 EN**: Closes the current lexical scope or compound statement.
  - **L762 CN**: 结束当前词法作用域或复合语句块。
- **L763 EN**: Closes the current lexical scope or compound statement.
  - **L763 CN**: 结束当前词法作用域或复合语句块。
- **L764 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L764 CN**: 执行一条独立语句或声明：`result += z;`。
- **L765 EN**: Closes the current lexical scope or compound statement.
  - **L765 CN**: 结束当前词法作用域或复合语句块。
- **L766 EN**: Returns from the current function with `result`.
  - **L766 CN**: 以 `result` 从当前函数返回。
- **L767 EN**: Closes the current lexical scope or compound statement.
  - **L767 CN**: 结束当前词法作用域或复合语句块。
- **L768 EN**: Blank line separating nearby declarations or logic.
  - **L768 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 769-792 / 第 769-792 行

````cpp
 769: template <class T, class Policy>
 770: BOOST_MATH_GPU_ENABLED T expint_i_imp(T z, const Policy& pol, const boost::math::integral_constant<int, 64>& tag)
 771: {
 772:    BOOST_MATH_STD_USING
 773:    constexpr auto function = "boost::math::expint<%1%>(%1%)";
 774:    if(z < 0)
 775:       return -expint_imp(1, T(-z), pol, tag);
 776:    if(z == 0)
 777:       return -policies::raise_overflow_error<T>(function, nullptr, pol);
 778: 
 779:    T result;
 780: 
 781:    if(z <= 6)
 782:    {
 783:       // Maximum Deviation Found:                     3.883e-21
 784:       // Expected Error Term:                         3.883e-21
 785:       // Max Error found at long double precision =   Poly: 3.344801e-19   Cheb: 4.989937e-19
 786: 
 787:       // LCOV_EXCL_START
 788:       static const T P[11] = {
 789:          BOOST_MATH_BIG_CONSTANT(T, 64, 2.98677224343598593764),
 790:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.25891613550886736592),
 791:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.789323584998672832285),
 792:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.092432587824602399339),
````
- **L769 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L769 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L770 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L770 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L771 EN**: Opens a new lexical scope or compound statement.
  - **L771 CN**: 打开一个新的词法作用域或复合语句块。
- **L772 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L772 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L773 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L773 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L774 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L774 CN**: 开始 `if` 控制流语句并计算其条件。
- **L775 EN**: Returns from the current function with `-expint_imp(1, T(-z), pol, tag)`.
  - **L775 CN**: 以 `-expint_imp(1, T(-z), pol, tag)` 从当前函数返回。
- **L776 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L776 CN**: 开始 `if` 控制流语句并计算其条件。
- **L777 EN**: Returns from the current function with `-policies::raise_overflow_error<T>(function, nullptr, pol)`.
  - **L777 CN**: 以 `-policies::raise_overflow_error<T>(function, nullptr, pol)` 从当前函数返回。
- **L778 EN**: Blank line separating nearby declarations or logic.
  - **L778 CN**: 空行，用于分隔相邻声明或逻辑。
- **L779 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L779 CN**: 执行一条独立语句或声明：`T result;`。
- **L780 EN**: Blank line separating nearby declarations or logic.
  - **L780 CN**: 空行，用于分隔相邻声明或逻辑。
- **L781 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L781 CN**: 开始 `if` 控制流语句并计算其条件。
- **L782 EN**: Opens a new lexical scope or compound statement.
  - **L782 CN**: 打开一个新的词法作用域或复合语句块。
- **L783 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     3.883e-21`.
  - **L783 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     3.883e-21`。
- **L784 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         3.883e-21`.
  - **L784 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         3.883e-21`。
- **L785 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 3.344801e-19   Cheb: 4.989937e-19`.
  - **L785 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 3.344801e-19   Cheb: 4.989937e-19`。
- **L786 EN**: Blank line separating nearby declarations or logic.
  - **L786 CN**: 空行，用于分隔相邻声明或逻辑。
- **L787 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L787 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L788 EN**: Continues the surrounding expression or declaration: `static const T P[11] = {`.
  - **L788 CN**: 继续构造周围的表达式或声明：`static const T P[11] = {`。
- **L789 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L789 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L790 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L790 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L791 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L791 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L792 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L792 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 793-816 / 第 793-816 行

````cpp
 793:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0514236978728625906656),
 794:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.00658477469745132977921),
 795:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.00124914538197086254233),
 796:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.000131429679565472408551),
 797:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.11293331317982763165e-4),
 798:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.629499283139417444244e-6),
 799:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.177833045143692498221e-7)
 800:       };
 801:       static const T Q[9] = {
 802:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 803:          BOOST_MATH_BIG_CONSTANT(T, 64, -1.20352377969742325748),
 804:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.66707904942606479811),
 805:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.223014531629140771914),
 806:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0493340022262908008636),
 807:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00741934273050807310677),
 808:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.00074353567782087939294),
 809:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.455861727069603367656e-4),
 810:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.131515429329812837701e-5)
 811:       };
 812: 
 813:       static const T c1 = BOOST_MATH_BIG_CONSTANT(T, 64, 1677624236387711.0);
 814:       static const T c2 = BOOST_MATH_BIG_CONSTANT(T, 64, 4503599627370496.0);
 815:       static const T r1 = c1 / c2;
 816:       static const T r2 = BOOST_MATH_BIG_CONSTANT(T, 64, 0.131401834143860282009280387409357165515556574352422001206362e-16);
````
- **L793 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L793 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L794 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L794 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L795 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L795 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L796 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L796 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L797 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L797 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L798 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L798 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L799 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L799 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L800 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L800 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L801 EN**: Continues the surrounding expression or declaration: `static const T Q[9] = {`.
  - **L801 CN**: 继续构造周围的表达式或声明：`static const T Q[9] = {`。
- **L802 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L802 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L803 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L803 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L804 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L804 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L805 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L805 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L806 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L806 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L807 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L807 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L808 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L808 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L809 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L809 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L810 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L810 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L811 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L811 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L812 EN**: Blank line separating nearby declarations or logic.
  - **L812 CN**: 空行，用于分隔相邻声明或逻辑。
- **L813 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L813 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L814 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L814 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L815 EN**: Initializes variable `r1` from the right-hand expression.
  - **L815 CN**: 使用右侧表达式初始化变量 `r1`。
- **L816 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L816 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 817-840 / 第 817-840 行

````cpp
 817:       static const T r = static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.372507410781366634461991866580119133535689497771654051555657435242200120636201854384926049951548942392));
 818:       // LCOV_EXCL_STOP
 819: 
 820:       T t = (z / 3) - 1;
 821:       result = tools::evaluate_polynomial(P, t)
 822:          / tools::evaluate_polynomial(Q, t);
 823:       t = (z - r1) - r2;
 824:       result *= t;
 825:       if(fabs(t) < T(0.1))
 826:       {
 827:          result += boost::math::log1p(t / r, pol);
 828:       }
 829:       else
 830:       {
 831:          result += log(z / r);
 832:       }
 833:    }
 834:    else if (z <= 10)
 835:    {
 836:       // Maximum Deviation Found:                     2.622e-21
 837:       // Expected Error Term:                         -2.622e-21
 838:       // Max Error found at long double precision =   Poly: 1.208328e-20   Cheb: 1.073723e-20
 839:       // LCOV_EXCL_START
 840:       static const T Y = 1.158985137939453125F;
````
- **L817 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L817 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L818 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L818 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L819 EN**: Blank line separating nearby declarations or logic.
  - **L819 CN**: 空行，用于分隔相邻声明或逻辑。
- **L820 EN**: Executes a call or declaration centered on `=`.
  - **L820 CN**: 执行以 `=` 为核心的调用或声明。
- **L821 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L821 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L822 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L822 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L823 EN**: Executes a call or declaration centered on `=`.
  - **L823 CN**: 执行以 `=` 为核心的调用或声明。
- **L824 EN**: Executes a standalone statement or declaration: `result *= t;`.
  - **L824 CN**: 执行一条独立语句或声明：`result *= t;`。
- **L825 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L825 CN**: 开始 `if` 控制流语句并计算其条件。
- **L826 EN**: Opens a new lexical scope or compound statement.
  - **L826 CN**: 打开一个新的词法作用域或复合语句块。
- **L827 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L827 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L828 EN**: Closes the current lexical scope or compound statement.
  - **L828 CN**: 结束当前词法作用域或复合语句块。
- **L829 EN**: Starts the alternative branch of the preceding conditional.
  - **L829 CN**: 开始前一个条件语句的备选分支。
- **L830 EN**: Opens a new lexical scope or compound statement.
  - **L830 CN**: 打开一个新的词法作用域或复合语句块。
- **L831 EN**: Executes a call or declaration centered on `log`.
  - **L831 CN**: 执行以 `log` 为核心的调用或声明。
- **L832 EN**: Closes the current lexical scope or compound statement.
  - **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Closes the current lexical scope or compound statement.
  - **L833 CN**: 结束当前词法作用域或复合语句块。
- **L834 EN**: Starts the alternative branch of the preceding conditional.
  - **L834 CN**: 开始前一个条件语句的备选分支。
- **L835 EN**: Opens a new lexical scope or compound statement.
  - **L835 CN**: 打开一个新的词法作用域或复合语句块。
- **L836 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     2.622e-21`.
  - **L836 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     2.622e-21`。
- **L837 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         -2.622e-21`.
  - **L837 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         -2.622e-21`。
- **L838 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 1.208328e-20   Cheb: 1.073723e-20`.
  - **L838 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 1.208328e-20   Cheb: 1.073723e-20`。
- **L839 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L839 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L840 EN**: Initializes variable `Y` from the right-hand expression.
  - **L840 CN**: 使用右侧表达式初始化变量 `Y`。

### Lines 841-864 / 第 841-864 行

````cpp
 841:       static const T P[9] = {
 842:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.00139324086199409049399),
 843:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0345238388952337563247),
 844:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0382065278072592940767),
 845:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0156117003070560727392),
 846:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00383276012430495387102),
 847:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.000697070540945496497992),
 848:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.877310384591205930343e-4),
 849:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.623067256376494930067e-5),
 850:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.377246883283337141444e-6)
 851:       };
 852:       static const T Q[10] = {
 853:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 854:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.08073635708902053767),
 855:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.553681133533942532909),
 856:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.176763647137553797451),
 857:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0387891748253869928121),
 858:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0060603004848394727017),
 859:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.000670519492939992806051),
 860:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.4947357050100855646e-4),
 861:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.204339282037446434827e-5),
 862:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.146951181174930425744e-7)
 863:       };
 864:       // LCOV_EXCL_STOP
````
- **L841 EN**: Continues the surrounding expression or declaration: `static const T P[9] = {`.
  - **L841 CN**: 继续构造周围的表达式或声明：`static const T P[9] = {`。
- **L842 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L842 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L843 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L843 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L844 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L844 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L845 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L845 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L846 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L846 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L847 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L847 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L848 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L848 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L849 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L849 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L850 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L850 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L851 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L851 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L852 EN**: Continues the surrounding expression or declaration: `static const T Q[10] = {`.
  - **L852 CN**: 继续构造周围的表达式或声明：`static const T Q[10] = {`。
- **L853 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L853 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L854 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L854 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L855 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L855 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L856 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L856 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L857 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L857 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L858 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L858 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L859 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L859 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L860 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L860 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L861 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L861 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L862 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L862 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L863 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L863 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L864 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L864 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。

### Lines 865-888 / 第 865-888 行

````cpp
 865:       T t = z / 2 - 4;
 866:       result = Y + tools::evaluate_polynomial(P, t)
 867:          / tools::evaluate_polynomial(Q, t);
 868:       result *= exp(z) / z;
 869:       result += z;
 870:    }
 871:    else if(z <= 20)
 872:    {
 873:       // Maximum Deviation Found:                     3.220e-20
 874:       // Expected Error Term:                         3.220e-20
 875:       // Max Error found at long double precision =   Poly: 7.696841e-20   Cheb: 6.205163e-20
 876: 
 877:       // LCOV_EXCL_START
 878:       static const T Y = 1.0869731903076171875F;
 879:       static const T P[10] = {
 880:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00893891094356946995368),
 881:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0487562980088748775943),
 882:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0670568657950041926085),
 883:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0509577352851442932713),
 884:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.02551800927409034206),
 885:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00892913759760086687083),
 886:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00224469630207344379888),
 887:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.000392477245911296982776),
 888:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.44424044184395578775e-4),
````
- **L865 EN**: Executes a standalone statement or declaration: `T t = z / 2 - 4;`.
  - **L865 CN**: 执行一条独立语句或声明：`T t = z / 2 - 4;`。
- **L866 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L866 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L867 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L867 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L868 EN**: Executes a call or declaration centered on `exp`.
  - **L868 CN**: 执行以 `exp` 为核心的调用或声明。
- **L869 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L869 CN**: 执行一条独立语句或声明：`result += z;`。
- **L870 EN**: Closes the current lexical scope or compound statement.
  - **L870 CN**: 结束当前词法作用域或复合语句块。
- **L871 EN**: Starts the alternative branch of the preceding conditional.
  - **L871 CN**: 开始前一个条件语句的备选分支。
- **L872 EN**: Opens a new lexical scope or compound statement.
  - **L872 CN**: 打开一个新的词法作用域或复合语句块。
- **L873 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     3.220e-20`.
  - **L873 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     3.220e-20`。
- **L874 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         3.220e-20`.
  - **L874 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         3.220e-20`。
- **L875 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 7.696841e-20   Cheb: 6.205163e-20`.
  - **L875 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 7.696841e-20   Cheb: 6.205163e-20`。
- **L876 EN**: Blank line separating nearby declarations or logic.
  - **L876 CN**: 空行，用于分隔相邻声明或逻辑。
- **L877 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L877 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L878 EN**: Initializes variable `Y` from the right-hand expression.
  - **L878 CN**: 使用右侧表达式初始化变量 `Y`。
- **L879 EN**: Continues the surrounding expression or declaration: `static const T P[10] = {`.
  - **L879 CN**: 继续构造周围的表达式或声明：`static const T P[10] = {`。
- **L880 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L880 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L881 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L881 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L882 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L882 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L883 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L883 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L884 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L884 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L885 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L885 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L886 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L886 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L887 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L887 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L888 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L888 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 889-912 / 第 889-912 行

````cpp
 889:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.252788029251437017959e-5)
 890:       };
 891:       static const T Q[10] = {
 892:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 893:          BOOST_MATH_BIG_CONSTANT(T, 64, 2.00323265503572414261),
 894:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.94688958187256383178),
 895:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.19733638134417472296),
 896:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.513137726038353385661),
 897:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.159135395578007264547),
 898:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0358233587351620919881),
 899:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0056716655597009417875),
 900:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.000577048986213535829925),
 901:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.290976943033493216793e-4)
 902:       };
 903:       // LCOV_EXCL_STOP
 904:       T t = z / 5 - 3;
 905:       result = Y + tools::evaluate_polynomial(P, t)
 906:          / tools::evaluate_polynomial(Q, t);
 907:       result *= exp(z) / z;
 908:       result += z;
 909:    }
 910:    else if(z <= 40)
 911:    {
 912:       // Maximum Deviation Found:                     2.940e-21
````
- **L889 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L889 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L890 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L890 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L891 EN**: Continues the surrounding expression or declaration: `static const T Q[10] = {`.
  - **L891 CN**: 继续构造周围的表达式或声明：`static const T Q[10] = {`。
- **L892 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L892 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L893 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L893 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L894 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L894 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L895 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L895 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L896 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L896 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L897 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L897 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L898 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L898 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L899 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L899 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L900 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L900 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L901 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L901 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L902 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L902 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L903 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L903 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L904 EN**: Executes a standalone statement or declaration: `T t = z / 5 - 3;`.
  - **L904 CN**: 执行一条独立语句或声明：`T t = z / 5 - 3;`。
- **L905 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L905 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L906 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L906 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L907 EN**: Executes a call or declaration centered on `exp`.
  - **L907 CN**: 执行以 `exp` 为核心的调用或声明。
- **L908 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L908 CN**: 执行一条独立语句或声明：`result += z;`。
- **L909 EN**: Closes the current lexical scope or compound statement.
  - **L909 CN**: 结束当前词法作用域或复合语句块。
- **L910 EN**: Starts the alternative branch of the preceding conditional.
  - **L910 CN**: 开始前一个条件语句的备选分支。
- **L911 EN**: Opens a new lexical scope or compound statement.
  - **L911 CN**: 打开一个新的词法作用域或复合语句块。
- **L912 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     2.940e-21`.
  - **L912 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     2.940e-21`。

### Lines 913-936 / 第 913-936 行

````cpp
 913:       // Expected Error Term:                         -2.938e-21
 914:       // Max Error found at long double precision =   Poly: 3.419893e-19   Cheb: 3.359874e-19
 915:       // LCOV_EXCL_START
 916:       static const T Y = 1.03937530517578125F;
 917:       static const T P[12] = {
 918:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00356165148914447278177),
 919:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0240235006148610849678),
 920:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0516699967278057976119),
 921:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0586603078706856245674),
 922:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0409960120868776180825),
 923:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0185485073689590665153),
 924:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00537842101034123222417),
 925:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.000920988084778273760609),
 926:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.716742618812210980263e-4),
 927:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.504623302166487346677e-9),
 928:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.712662196671896837736e-10),
 929:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.533769629702262072175e-11)
 930:       };
 931:       static const T Q[9] = {
 932:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 933:          BOOST_MATH_BIG_CONSTANT(T, 64, 3.13286733695729715455),
 934:          BOOST_MATH_BIG_CONSTANT(T, 64, 4.49281223045653491929),
 935:          BOOST_MATH_BIG_CONSTANT(T, 64, 3.84900294427622911374),
 936:          BOOST_MATH_BIG_CONSTANT(T, 64, 2.15205199043580378211),
````
- **L913 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         -2.938e-21`.
  - **L913 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         -2.938e-21`。
- **L914 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 3.419893e-19   Cheb: 3.359874e-19`.
  - **L914 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 3.419893e-19   Cheb: 3.359874e-19`。
- **L915 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L915 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L916 EN**: Initializes variable `Y` from the right-hand expression.
  - **L916 CN**: 使用右侧表达式初始化变量 `Y`。
- **L917 EN**: Continues the surrounding expression or declaration: `static const T P[12] = {`.
  - **L917 CN**: 继续构造周围的表达式或声明：`static const T P[12] = {`。
- **L918 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L918 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L919 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L919 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L920 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L920 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L921 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L921 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L922 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L922 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L923 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L923 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L924 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L924 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L925 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L925 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L926 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L926 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L927 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L927 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L928 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L928 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L929 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L929 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L930 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L930 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L931 EN**: Continues the surrounding expression or declaration: `static const T Q[9] = {`.
  - **L931 CN**: 继续构造周围的表达式或声明：`static const T Q[9] = {`。
- **L932 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L932 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L933 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L933 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L934 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L934 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L935 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L935 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L936 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L936 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 937-960 / 第 937-960 行

````cpp
 937:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.802912186540269232424),
 938:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.194793170017818925388),
 939:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0280128013584653182994),
 940:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.00182034930799902922549)
 941:       };
 942:       // LCOV_EXCL_STOP
 943:       T t = z / 10 - 3;
 944:       result = Y + tools::evaluate_polynomial(P, t)
 945:          / tools::evaluate_polynomial(Q, t);
 946:       BOOST_MATH_INSTRUMENT_VARIABLE(result)
 947:       result *= exp(z) / z;
 948:       BOOST_MATH_INSTRUMENT_VARIABLE(result)
 949:       result += z;
 950:       BOOST_MATH_INSTRUMENT_VARIABLE(result)
 951:    }
 952:    else
 953:    {
 954:       // Maximum Deviation Found:                     3.536e-20
 955:       // Max Error found at long double precision =   Poly: 1.310671e-19   Cheb: 8.630943e-11
 956:       // LCOV_EXCL_START
 957:       static const T exp40 = static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 2.35385266837019985407899910749034804508871617254555467236651e17));
 958:       static const T Y= 1.013065338134765625F;
 959:       static const T P[9] = {
 960:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0130653381347656250004),
````
- **L937 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L937 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L938 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L938 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L939 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L939 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L940 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L940 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L941 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L941 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L942 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L942 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L943 EN**: Executes a standalone statement or declaration: `T t = z / 10 - 3;`.
  - **L943 CN**: 执行一条独立语句或声明：`T t = z / 10 - 3;`。
- **L944 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L944 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L945 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L945 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L946 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L946 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L947 EN**: Executes a call or declaration centered on `exp`.
  - **L947 CN**: 执行以 `exp` 为核心的调用或声明。
- **L948 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L948 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L949 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L949 CN**: 执行一条独立语句或声明：`result += z;`。
- **L950 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L950 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L951 EN**: Closes the current lexical scope or compound statement.
  - **L951 CN**: 结束当前词法作用域或复合语句块。
- **L952 EN**: Starts the alternative branch of the preceding conditional.
  - **L952 CN**: 开始前一个条件语句的备选分支。
- **L953 EN**: Opens a new lexical scope or compound statement.
  - **L953 CN**: 打开一个新的词法作用域或复合语句块。
- **L954 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     3.536e-20`.
  - **L954 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     3.536e-20`。
- **L955 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 1.310671e-19   Cheb: 8.630943e-11`.
  - **L955 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 1.310671e-19   Cheb: 8.630943e-11`。
- **L956 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L956 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L957 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L957 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L958 EN**: Initializes variable `Y` from the right-hand expression.
  - **L958 CN**: 使用右侧表达式初始化变量 `Y`。
- **L959 EN**: Continues the surrounding expression or declaration: `static const T P[9] = {`.
  - **L959 CN**: 继续构造周围的表达式或声明：`static const T P[9] = {`。
- **L960 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L960 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 961-984 / 第 961-984 行

````cpp
 961:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.644487780349757303739),
 962:          BOOST_MATH_BIG_CONSTANT(T, 64, 143.995670348227433964),
 963:          BOOST_MATH_BIG_CONSTANT(T, 64, -13918.9322758014173709),
 964:          BOOST_MATH_BIG_CONSTANT(T, 64, 476260.975133624194484),
 965:          BOOST_MATH_BIG_CONSTANT(T, 64, -7437102.15135982802122),
 966:          BOOST_MATH_BIG_CONSTANT(T, 64, 53732298.8764767916542),
 967:          BOOST_MATH_BIG_CONSTANT(T, 64, -160695051.957997452509),
 968:          BOOST_MATH_BIG_CONSTANT(T, 64, 137839271.592778020028)
 969:       };
 970:       static const T Q[9] = {
 971:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 972:          BOOST_MATH_BIG_CONSTANT(T, 64, 27.2103343964943718802),
 973:          BOOST_MATH_BIG_CONSTANT(T, 64, -8785.48528692879413676),
 974:          BOOST_MATH_BIG_CONSTANT(T, 64, 397530.290000322626766),
 975:          BOOST_MATH_BIG_CONSTANT(T, 64, -7356441.34957799368252),
 976:          BOOST_MATH_BIG_CONSTANT(T, 64, 63050914.5343400957524),
 977:          BOOST_MATH_BIG_CONSTANT(T, 64, -246143779.638307701369),
 978:          BOOST_MATH_BIG_CONSTANT(T, 64, 384647824.678554961174),
 979:          BOOST_MATH_BIG_CONSTANT(T, 64, -166288297.874583961493)
 980:       };
 981:       // LCOV_EXCL_STOP
 982:       T t = 1 / z;
 983:       result = Y + tools::evaluate_polynomial(P, t)
 984:          / tools::evaluate_polynomial(Q, t);
````
- **L961 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L961 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L962 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L962 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L963 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L963 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L964 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L964 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L965 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L965 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L966 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L966 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L967 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L967 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L968 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L968 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L969 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L969 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L970 EN**: Continues the surrounding expression or declaration: `static const T Q[9] = {`.
  - **L970 CN**: 继续构造周围的表达式或声明：`static const T Q[9] = {`。
- **L971 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L971 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L972 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L972 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L973 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L973 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L974 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L974 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L975 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L975 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L976 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L976 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L977 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L977 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L978 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L978 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L979 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L979 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L980 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L980 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L981 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L981 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L982 EN**: Executes a standalone statement or declaration: `T t = 1 / z;`.
  - **L982 CN**: 执行一条独立语句或声明：`T t = 1 / z;`。
- **L983 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L983 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L984 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L984 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。

### Lines 985-1008 / 第 985-1008 行

````cpp
 985:       if(z < 41)
 986:          result *= exp(z) / z;
 987:       else
 988:       {
 989:          // Avoid premature overflow if we can:
 990:          t = z - 40;
 991:          if(t > tools::log_max_value<T>())
 992:          {
 993:             result = policies::raise_overflow_error<T>(function, nullptr, pol);
 994:          }
 995:          else
 996:          {
 997:             result *= exp(z - 40) / z;
 998:             if(result > tools::max_value<T>() / exp40)
 999:             {
1000:                result = policies::raise_overflow_error<T>(function, nullptr, pol);
1001:             }
1002:             else
1003:             {
1004:                result *= exp40;
1005:             }
1006:          }
1007:       }
1008:       result += z;
````
- **L985 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L985 CN**: 开始 `if` 控制流语句并计算其条件。
- **L986 EN**: Executes a call or declaration centered on `exp`.
  - **L986 CN**: 执行以 `exp` 为核心的调用或声明。
- **L987 EN**: Starts the alternative branch of the preceding conditional.
  - **L987 CN**: 开始前一个条件语句的备选分支。
- **L988 EN**: Opens a new lexical scope or compound statement.
  - **L988 CN**: 打开一个新的词法作用域或复合语句块。
- **L989 EN**: Comment documents nearby intent or usage notes: `Avoid premature overflow if we can:`.
  - **L989 CN**: 注释说明附近代码的意图或使用说明：`Avoid premature overflow if we can:`。
- **L990 EN**: Executes a standalone statement or declaration: `t = z - 40;`.
  - **L990 CN**: 执行一条独立语句或声明：`t = z - 40;`。
- **L991 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L991 CN**: 开始 `if` 控制流语句并计算其条件。
- **L992 EN**: Opens a new lexical scope or compound statement.
  - **L992 CN**: 打开一个新的词法作用域或复合语句块。
- **L993 EN**: Executes a call or declaration centered on `policies::raise_overflow_error<T>`.
  - **L993 CN**: 执行以 `policies::raise_overflow_error<T>` 为核心的调用或声明。
- **L994 EN**: Closes the current lexical scope or compound statement.
  - **L994 CN**: 结束当前词法作用域或复合语句块。
- **L995 EN**: Starts the alternative branch of the preceding conditional.
  - **L995 CN**: 开始前一个条件语句的备选分支。
- **L996 EN**: Opens a new lexical scope or compound statement.
  - **L996 CN**: 打开一个新的词法作用域或复合语句块。
- **L997 EN**: Executes a call or declaration centered on `exp`.
  - **L997 CN**: 执行以 `exp` 为核心的调用或声明。
- **L998 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L998 CN**: 开始 `if` 控制流语句并计算其条件。
- **L999 EN**: Opens a new lexical scope or compound statement.
  - **L999 CN**: 打开一个新的词法作用域或复合语句块。
- **L1000 EN**: Executes a call or declaration centered on `policies::raise_overflow_error<T>`.
  - **L1000 CN**: 执行以 `policies::raise_overflow_error<T>` 为核心的调用或声明。
- **L1001 EN**: Closes the current lexical scope or compound statement.
  - **L1001 CN**: 结束当前词法作用域或复合语句块。
- **L1002 EN**: Starts the alternative branch of the preceding conditional.
  - **L1002 CN**: 开始前一个条件语句的备选分支。
- **L1003 EN**: Opens a new lexical scope or compound statement.
  - **L1003 CN**: 打开一个新的词法作用域或复合语句块。
- **L1004 EN**: Executes a standalone statement or declaration: `result *= exp40;`.
  - **L1004 CN**: 执行一条独立语句或声明：`result *= exp40;`。
- **L1005 EN**: Closes the current lexical scope or compound statement.
  - **L1005 CN**: 结束当前词法作用域或复合语句块。
- **L1006 EN**: Closes the current lexical scope or compound statement.
  - **L1006 CN**: 结束当前词法作用域或复合语句块。
- **L1007 EN**: Closes the current lexical scope or compound statement.
  - **L1007 CN**: 结束当前词法作用域或复合语句块。
- **L1008 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1008 CN**: 执行一条独立语句或声明：`result += z;`。

### Lines 1009-1032 / 第 1009-1032 行

````cpp
1009:    }
1010:    return result;
1011: }
1012: 
1013: template <class T, class Policy>
1014: BOOST_MATH_GPU_ENABLED void expint_i_imp_113a(T& result, const T& z, const Policy& pol)
1015: {
1016:    BOOST_MATH_STD_USING
1017:    // Maximum Deviation Found:                     1.230e-36
1018:    // Expected Error Term:                         -1.230e-36
1019:    // Max Error found at long double precision =   Poly: 4.355299e-34   Cheb: 7.512581e-34
1020: 
1021:    // LCOV_EXCL_START
1022:    static const T P[15] = {
1023:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.98677224343598593765287235997328555),
1024:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.333256034674702967028780537349334037),
1025:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.851831522798101228384971644036708463),
1026:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0657854833494646206186773614110374948),
1027:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0630065662557284456000060708977935073),
1028:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00311759191425309373327784154659649232),
1029:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00176213568201493949664478471656026771),
1030:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.491548660404172089488535218163952295e-4),
1031:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.207764227621061706075562107748176592e-4),
1032:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.225445398156913584846374273379402765e-6),
````
- **L1009 EN**: Closes the current lexical scope or compound statement.
  - **L1009 CN**: 结束当前词法作用域或复合语句块。
- **L1010 EN**: Returns from the current function with `result`.
  - **L1010 CN**: 以 `result` 从当前函数返回。
- **L1011 EN**: Closes the current lexical scope or compound statement.
  - **L1011 CN**: 结束当前词法作用域或复合语句块。
- **L1012 EN**: Blank line separating nearby declarations or logic.
  - **L1012 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1013 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1013 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1014 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1014 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1015 EN**: Opens a new lexical scope or compound statement.
  - **L1015 CN**: 打开一个新的词法作用域或复合语句块。
- **L1016 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1016 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1017 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     1.230e-36`.
  - **L1017 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     1.230e-36`。
- **L1018 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         -1.230e-36`.
  - **L1018 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         -1.230e-36`。
- **L1019 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 4.355299e-34   Cheb: 7.512581e-34`.
  - **L1019 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 4.355299e-34   Cheb: 7.512581e-34`。
- **L1020 EN**: Blank line separating nearby declarations or logic.
  - **L1020 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1021 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1021 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1022 EN**: Continues the surrounding expression or declaration: `static const T P[15] = {`.
  - **L1022 CN**: 继续构造周围的表达式或声明：`static const T P[15] = {`。
- **L1023 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1023 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1024 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1024 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1025 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1025 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1026 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1026 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1027 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1027 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1028 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1028 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1029 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1029 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1030 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1030 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1031 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1031 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1032 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1032 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1033-1056 / 第 1033-1056 行

````cpp
1033:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.996939977231410319761273881672601592e-7),
1034:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.212546902052178643330520878928100847e-9),
1035:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.154646053060262871360159325115980023e-9),
1036:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.143971277122049197323415503594302307e-11),
1037:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.306243138978114692252817805327426657e-13)
1038:    };
1039:    static const T Q[15] = {
1040:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1041:       BOOST_MATH_BIG_CONSTANT(T, 113, -1.40178870313943798705491944989231793),
1042:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.943810968269701047641218856758605284),
1043:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.405026631534345064600850391026113165),
1044:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.123924153524614086482627660399122762),
1045:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0286364505373369439591132549624317707),
1046:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00516148845910606985396596845494015963),
1047:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000738330799456364820380739850924783649),
1048:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.843737760991856114061953265870882637e-4),
1049:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.767957673431982543213661388914587589e-5),
1050:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.549136847313854595809952100614840031e-6),
1051:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.299801381513743676764008325949325404e-7),
1052:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.118419479055346106118129130945423483e-8),
1053:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.30372295663095470359211949045344607e-10),
1054:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.382742953753485333207877784720070523e-12)
1055:    };
1056: 
````
- **L1033 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1033 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1034 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1034 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1035 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1035 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1036 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1036 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1037 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1037 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1038 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1038 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1039 EN**: Continues the surrounding expression or declaration: `static const T Q[15] = {`.
  - **L1039 CN**: 继续构造周围的表达式或声明：`static const T Q[15] = {`。
- **L1040 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1040 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1041 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1041 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1042 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1042 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1043 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1043 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1044 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1044 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1045 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1045 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1046 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1046 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1047 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1047 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1048 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1048 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1049 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1049 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1050 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1050 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1051 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1051 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1052 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1052 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1053 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1053 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1054 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1054 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1055 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1055 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1056 EN**: Blank line separating nearby declarations or logic.
  - **L1056 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 1057-1080 / 第 1057-1080 行

````cpp
1057:    static const T c1 = BOOST_MATH_BIG_CONSTANT(T, 113, 1677624236387711.0);
1058:    static const T c2 = BOOST_MATH_BIG_CONSTANT(T, 113, 4503599627370496.0);
1059:    static const T c3 = BOOST_MATH_BIG_CONSTANT(T, 113, 266514582277687.0);
1060:    static const T c4 = BOOST_MATH_BIG_CONSTANT(T, 113, 4503599627370496.0);
1061:    static const T c5 = BOOST_MATH_BIG_CONSTANT(T, 113, 4503599627370496.0);
1062:    static const T r1 = c1 / c2;
1063:    static const T r2 = c3 / c4 / c5;
1064:    static const T r3 = static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 113, 0.283806480836357377069325311780969887585024578164571984232357e-31));
1065:    static const T r = static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 113, 0.372507410781366634461991866580119133535689497771654051555657435242200120636201854384926049951548942392));
1066:    // LCOV_EXCL_STOP
1067:    T t = (z / 3) - 1;
1068:    result = tools::evaluate_polynomial(P, t)
1069:       / tools::evaluate_polynomial(Q, t);
1070:    t = ((z - r1) - r2) - r3;
1071:    result *= t;
1072:    if(fabs(t) < 0.1)
1073:    {
1074:       result += boost::math::log1p(t / r, pol);
1075:    }
1076:    else
1077:    {
1078:       result += log(z / r);
1079:    }
1080: }
````
- **L1057 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1057 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1058 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1058 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1059 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1059 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1060 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1060 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1061 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1061 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1062 EN**: Initializes variable `r1` from the right-hand expression.
  - **L1062 CN**: 使用右侧表达式初始化变量 `r1`。
- **L1063 EN**: Initializes variable `r2` from the right-hand expression.
  - **L1063 CN**: 使用右侧表达式初始化变量 `r2`。
- **L1064 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1064 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1065 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1065 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1066 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1066 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1067 EN**: Executes a call or declaration centered on `=`.
  - **L1067 CN**: 执行以 `=` 为核心的调用或声明。
- **L1068 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1068 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1069 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1069 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1070 EN**: Executes a call or declaration centered on `=`.
  - **L1070 CN**: 执行以 `=` 为核心的调用或声明。
- **L1071 EN**: Executes a standalone statement or declaration: `result *= t;`.
  - **L1071 CN**: 执行一条独立语句或声明：`result *= t;`。
- **L1072 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1072 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1073 EN**: Opens a new lexical scope or compound statement.
  - **L1073 CN**: 打开一个新的词法作用域或复合语句块。
- **L1074 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1074 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  - **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Starts the alternative branch of the preceding conditional.
  - **L1076 CN**: 开始前一个条件语句的备选分支。
- **L1077 EN**: Opens a new lexical scope or compound statement.
  - **L1077 CN**: 打开一个新的词法作用域或复合语句块。
- **L1078 EN**: Executes a call or declaration centered on `log`.
  - **L1078 CN**: 执行以 `log` 为核心的调用或声明。
- **L1079 EN**: Closes the current lexical scope or compound statement.
  - **L1079 CN**: 结束当前词法作用域或复合语句块。
- **L1080 EN**: Closes the current lexical scope or compound statement.
  - **L1080 CN**: 结束当前词法作用域或复合语句块。

### Lines 1081-1104 / 第 1081-1104 行

````cpp
1081: 
1082: template <class T>
1083: BOOST_MATH_GPU_ENABLED void expint_i_113b(T& result, const T& z)
1084: {
1085:    BOOST_MATH_STD_USING
1086:    // Maximum Deviation Found:                     7.779e-36
1087:    // Expected Error Term:                         -7.779e-36
1088:    // Max Error found at long double precision =   Poly: 2.576723e-35   Cheb: 1.236001e-34
1089:    // LCOV_EXCL_START
1090:    static const T Y = 1.158985137939453125F;
1091:    static const T P[15] = {
1092:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00139324086199409049282472239613554817),
1093:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0338173111691991289178779840307998955),
1094:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0555972290794371306259684845277620556),
1095:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0378677976003456171563136909186202177),
1096:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0152221583517528358782902783914356667),
1097:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00428283334203873035104248217403126905),
1098:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000922782631491644846511553601323435286),
1099:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000155513428088853161562660696055496696),
1100:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.205756580255359882813545261519317096e-4),
1101:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.220327406578552089820753181821115181e-5),
1102:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.189483157545587592043421445645377439e-6),
1103:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.122426571518570587750898968123803867e-7),
1104:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.635187358949437991465353268374523944e-9),
````
- **L1081 EN**: Blank line separating nearby declarations or logic.
  - **L1081 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1082 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1082 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1083 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1083 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1084 EN**: Opens a new lexical scope or compound statement.
  - **L1084 CN**: 打开一个新的词法作用域或复合语句块。
- **L1085 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1085 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1086 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     7.779e-36`.
  - **L1086 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     7.779e-36`。
- **L1087 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         -7.779e-36`.
  - **L1087 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         -7.779e-36`。
- **L1088 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 2.576723e-35   Cheb: 1.236001e-34`.
  - **L1088 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 2.576723e-35   Cheb: 1.236001e-34`。
- **L1089 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1089 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1090 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1090 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1091 EN**: Continues the surrounding expression or declaration: `static const T P[15] = {`.
  - **L1091 CN**: 继续构造周围的表达式或声明：`static const T P[15] = {`。
- **L1092 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1092 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1093 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1093 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1094 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1094 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1095 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1095 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1096 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1096 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1097 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1097 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1098 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1098 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1099 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1099 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1100 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1100 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1101 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1101 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1102 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1102 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1103 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1103 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1104 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1104 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1105-1128 / 第 1105-1128 行

````cpp
1105:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.203015132965870311935118337194860863e-10),
1106:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.384276705503357655108096065452950822e-12)
1107:    };
1108:    static const T Q[15] = {
1109:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1110:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.58784732785354597996617046880946257),
1111:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.18550755302279446339364262338114098),
1112:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.55598993549661368604527040349702836),
1113:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.184290888380564236919107835030984453),
1114:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0459658051803613282360464632326866113),
1115:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0089505064268613225167835599456014705),
1116:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00139042673882987693424772855926289077),
1117:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000174210708041584097450805790176479012),
1118:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.176324034009707558089086875136647376e-4),
1119:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.142935845999505649273084545313710581e-5),
1120:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.907502324487057260675816233312747784e-7),
1121:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.431044337808893270797934621235918418e-8),
1122:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.139007266881450521776529705677086902e-9),
1123:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.234715286125516430792452741830364672e-11)
1124:    };
1125:    // LCOV_EXCL_STOP
1126:    T t = z / 2 - 4;
1127:    result = Y + tools::evaluate_polynomial(P, t)
1128:       / tools::evaluate_polynomial(Q, t);
````
- **L1105 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1105 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1106 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1106 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1107 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1107 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1108 EN**: Continues the surrounding expression or declaration: `static const T Q[15] = {`.
  - **L1108 CN**: 继续构造周围的表达式或声明：`static const T Q[15] = {`。
- **L1109 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1109 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1110 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1110 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1111 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1111 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1112 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1112 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1113 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1113 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1114 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1114 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1115 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1115 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1116 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1116 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1117 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1117 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1118 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1118 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1119 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1119 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1120 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1120 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1121 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1121 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1122 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1122 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1123 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1123 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1124 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1124 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1125 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1125 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1126 EN**: Executes a standalone statement or declaration: `T t = z / 2 - 4;`.
  - **L1126 CN**: 执行一条独立语句或声明：`T t = z / 2 - 4;`。
- **L1127 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1127 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1128 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1128 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。

### Lines 1129-1152 / 第 1129-1152 行

````cpp
1129:    result *= exp(z) / z;
1130:    result += z;
1131: }
1132: 
1133: template <class T>
1134: BOOST_MATH_GPU_ENABLED void expint_i_113c(T& result, const T& z)
1135: {
1136:    BOOST_MATH_STD_USING
1137:    // Maximum Deviation Found:                     1.082e-34
1138:    // Expected Error Term:                         1.080e-34
1139:    // Max Error found at long double precision =   Poly: 1.958294e-34   Cheb: 2.472261e-34
1140: 
1141:    // LCOV_EXCL_START
1142:    static const T Y = 1.091579437255859375F;
1143:    static const T P[17] = {
1144:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00685089599550151282724924894258520532),
1145:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0443313550253580053324487059748497467),
1146:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.071538561252424027443296958795814874),
1147:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0622923153354102682285444067843300583),
1148:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0361631270264607478205393775461208794),
1149:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0153192826839624850298106509601033261),
1150:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00496967904961260031539602977748408242),
1151:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00126989079663425780800919171538920589),
1152:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000258933143097125199914724875206326698),
````
- **L1129 EN**: Executes a call or declaration centered on `exp`.
  - **L1129 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1130 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1130 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1131 EN**: Closes the current lexical scope or compound statement.
  - **L1131 CN**: 结束当前词法作用域或复合语句块。
- **L1132 EN**: Blank line separating nearby declarations or logic.
  - **L1132 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1133 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1133 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1134 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1134 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1135 EN**: Opens a new lexical scope or compound statement.
  - **L1135 CN**: 打开一个新的词法作用域或复合语句块。
- **L1136 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1136 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1137 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     1.082e-34`.
  - **L1137 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     1.082e-34`。
- **L1138 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         1.080e-34`.
  - **L1138 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         1.080e-34`。
- **L1139 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 1.958294e-34   Cheb: 2.472261e-34`.
  - **L1139 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 1.958294e-34   Cheb: 2.472261e-34`。
- **L1140 EN**: Blank line separating nearby declarations or logic.
  - **L1140 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1141 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1141 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1142 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1142 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1143 EN**: Continues the surrounding expression or declaration: `static const T P[17] = {`.
  - **L1143 CN**: 继续构造周围的表达式或声明：`static const T P[17] = {`。
- **L1144 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1144 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1145 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1145 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1146 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1146 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1147 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1147 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1148 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1148 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1149 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1149 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1150 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1150 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1151 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1151 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1152 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1152 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1153-1176 / 第 1153-1176 行

````cpp
1153:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.422110326689204794443002330541441956e-4),
1154:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.546004547590412661451073996127115221e-5),
1155:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.546775260262202177131068692199272241e-6),
1156:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.404157632825805803833379568956559215e-7),
1157:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.200612596196561323832327013027419284e-8),
1158:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.502538501472133913417609379765434153e-10),
1159:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.326283053716799774936661568391296584e-13),
1160:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.869226483473172853557775877908693647e-15)
1161:    };
1162:    static const T Q[15] = {
1163:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1164:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.23227220874479061894038229141871087),
1165:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.40221000361027971895657505660959863),
1166:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.65476320985936174728238416007084214),
1167:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.816828602963895720369875535001248227),
1168:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.306337922909446903672123418670921066),
1169:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0902400121654409267774593230720600752),
1170:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0212708882169429206498765100993228086),
1171:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00404442626252467471957713495828165491),
1172:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0006195601618842253612635241404054589),
1173:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.755930932686543009521454653994321843e-4),
1174:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.716004532773778954193609582677482803e-5),
1175:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.500881663076471627699290821742924233e-6),
1176:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.233593219218823384508105943657387644e-7),
````
- **L1153 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1153 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1154 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1154 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1155 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1155 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1156 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1156 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1157 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1157 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1158 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1158 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1159 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1159 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1160 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1160 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1161 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1161 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1162 EN**: Continues the surrounding expression or declaration: `static const T Q[15] = {`.
  - **L1162 CN**: 继续构造周围的表达式或声明：`static const T Q[15] = {`。
- **L1163 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1163 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1164 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1164 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1165 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1165 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1166 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1166 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1167 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1167 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1168 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1168 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1169 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1169 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1170 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1170 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1171 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1171 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1172 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1172 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1173 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1173 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1174 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1174 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1175 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1175 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1176 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1176 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1177-1200 / 第 1177-1200 行

````cpp
1177:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.554900353169148897444104962034267682e-9)
1178:    };
1179:    // LCOV_EXCL_STOP
1180:    T t = z / 4 - 3.5;
1181:    result = Y + tools::evaluate_polynomial(P, t)
1182:       / tools::evaluate_polynomial(Q, t);
1183:    result *= exp(z) / z;
1184:    result += z;
1185: }
1186: 
1187: template <class T>
1188: BOOST_MATH_GPU_ENABLED void expint_i_113d(T& result, const T& z)
1189: {
1190:    BOOST_MATH_STD_USING
1191:    // Maximum Deviation Found:                     3.163e-35
1192:    // Expected Error Term:                         3.163e-35
1193:    // Max Error found at long double precision =   Poly: 4.158110e-35   Cheb: 5.385532e-35
1194:    // LCOV_EXCL_START
1195:    static const T Y = 1.051731109619140625F;
1196:    static const T P[14] = {
1197:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00144552494420652573815404828020593565),
1198:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0126747451594545338365684731262912741),
1199:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.01757394877502366717526779263438073),
1200:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0126838952395506921945756139424722588),
````
- **L1177 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1177 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1178 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1178 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1179 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1179 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1180 EN**: Executes a standalone statement or declaration: `T t = z / 4 - 3.5;`.
  - **L1180 CN**: 执行一条独立语句或声明：`T t = z / 4 - 3.5;`。
- **L1181 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1181 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1182 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1182 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1183 EN**: Executes a call or declaration centered on `exp`.
  - **L1183 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1184 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1184 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1185 EN**: Closes the current lexical scope or compound statement.
  - **L1185 CN**: 结束当前词法作用域或复合语句块。
- **L1186 EN**: Blank line separating nearby declarations or logic.
  - **L1186 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1187 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1187 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1188 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1188 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1189 EN**: Opens a new lexical scope or compound statement.
  - **L1189 CN**: 打开一个新的词法作用域或复合语句块。
- **L1190 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1190 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1191 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     3.163e-35`.
  - **L1191 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     3.163e-35`。
- **L1192 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         3.163e-35`.
  - **L1192 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         3.163e-35`。
- **L1193 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 4.158110e-35   Cheb: 5.385532e-35`.
  - **L1193 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 4.158110e-35   Cheb: 5.385532e-35`。
- **L1194 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1194 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1195 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1195 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1196 EN**: Continues the surrounding expression or declaration: `static const T P[14] = {`.
  - **L1196 CN**: 继续构造周围的表达式或声明：`static const T P[14] = {`。
- **L1197 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1197 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1198 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1198 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1199 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1199 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1200 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1200 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1201-1224 / 第 1201-1224 行

````cpp
1201:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0060045057928894974954756789352443522),
1202:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00205349237147226126653803455793107903),
1203:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000532606040579654887676082220195624207),
1204:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000107344687098019891474772069139014662),
1205:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.169536802705805811859089949943435152e-4),
1206:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.20863311729206543881826553010120078e-5),
1207:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.195670358542116256713560296776654385e-6),
1208:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.133291168587253145439184028259772437e-7),
1209:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.595500337089495614285777067722823397e-9),
1210:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.133141358866324100955927979606981328e-10)
1211:    };
1212:    static const T Q[14] = {
1213:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1214:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.72490783907582654629537013560044682),
1215:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.44524329516800613088375685659759765),
1216:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.778241785539308257585068744978050181),
1217:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.300520486589206605184097270225725584),
1218:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0879346899691339661394537806057953957),
1219:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0200802415843802892793583043470125006),
1220:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00362842049172586254520256100538273214),
1221:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000519731362862955132062751246769469957),
1222:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.584092147914050999895178697392282665e-4),
1223:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.501851497707855358002773398333542337e-5),
1224:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.313085677467921096644895738538865537e-6),
````
- **L1201 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1201 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1202 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1202 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1203 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1203 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1204 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1204 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1205 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1205 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1206 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1206 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1207 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1207 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1208 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1208 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1209 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1209 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1210 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1210 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1211 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1211 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1212 EN**: Continues the surrounding expression or declaration: `static const T Q[14] = {`.
  - **L1212 CN**: 继续构造周围的表达式或声明：`static const T Q[14] = {`。
- **L1213 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1213 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1214 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1214 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1215 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1215 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1216 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1216 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1217 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1217 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1218 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1218 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1219 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1219 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1220 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1220 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1221 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1221 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1222 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1222 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1223 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1223 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1224 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1224 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1225-1248 / 第 1225-1248 行

````cpp
1225:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.127552010539733113371132321521204458e-7),
1226:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.25737310826983451144405899970774587e-9)
1227:    };
1228:    // LCOV_EXCL_STOP
1229:    T t = z / 4 - 5.5;
1230:    result = Y + tools::evaluate_polynomial(P, t)
1231:       / tools::evaluate_polynomial(Q, t);
1232:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1233:    result *= exp(z) / z;
1234:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1235:    result += z;
1236:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1237: }
1238: 
1239: template <class T>
1240: BOOST_MATH_GPU_ENABLED void expint_i_113e(T& result, const T& z)
1241: {
1242:    BOOST_MATH_STD_USING
1243:    // Maximum Deviation Found:                     7.972e-36
1244:    // Expected Error Term:                         7.962e-36
1245:    // Max Error found at long double precision =   Poly: 1.711721e-34   Cheb: 3.100018e-34
1246:    // LCOV_EXCL_START
1247:    static const T Y = 1.032726287841796875F;
1248:    static const T P[15] = {
````
- **L1225 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1225 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1226 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1226 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1227 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1227 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1228 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1228 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1229 EN**: Executes a standalone statement or declaration: `T t = z / 4 - 5.5;`.
  - **L1229 CN**: 执行一条独立语句或声明：`T t = z / 4 - 5.5;`。
- **L1230 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1230 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1231 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1231 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1232 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1232 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1233 EN**: Executes a call or declaration centered on `exp`.
  - **L1233 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1234 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1234 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1235 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1235 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1236 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1236 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  - **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Blank line separating nearby declarations or logic.
  - **L1238 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1239 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1239 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1240 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1240 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1241 EN**: Opens a new lexical scope or compound statement.
  - **L1241 CN**: 打开一个新的词法作用域或复合语句块。
- **L1242 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1242 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1243 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     7.972e-36`.
  - **L1243 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     7.972e-36`。
- **L1244 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         7.962e-36`.
  - **L1244 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         7.962e-36`。
- **L1245 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 1.711721e-34   Cheb: 3.100018e-34`.
  - **L1245 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 1.711721e-34   Cheb: 3.100018e-34`。
- **L1246 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1246 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1247 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1247 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1248 EN**: Continues the surrounding expression or declaration: `static const T P[15] = {`.
  - **L1248 CN**: 继续构造周围的表达式或声明：`static const T P[15] = {`。

### Lines 1249-1272 / 第 1249-1272 行

````cpp
1249:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00141056919297307534690895009969373233),
1250:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0123384175302540291339020257071411437),
1251:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0298127270706864057791526083667396115),
1252:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0390686759471630584626293670260768098),
1253:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0338226792912607409822059922949035589),
1254:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0211659736179834946452561197559654582),
1255:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0100428887460879377373158821400070313),
1256:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00370717396015165148484022792801682932),
1257:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0010768667551001624764329000496561659),
1258:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000246127328761027039347584096573123531),
1259:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.437318110527818613580613051861991198e-4),
1260:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.587532682329299591501065482317771497e-5),
1261:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.565697065670893984610852937110819467e-6),
1262:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.350233957364028523971768887437839573e-7),
1263:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.105428907085424234504608142258423505e-8)
1264:    };
1265:    static const T Q[16] = {
1266:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1267:       BOOST_MATH_BIG_CONSTANT(T, 113, 3.17261315255467581204685605414005525),
1268:       BOOST_MATH_BIG_CONSTANT(T, 113, 4.85267952971640525245338392887217426),
1269:       BOOST_MATH_BIG_CONSTANT(T, 113, 4.74341914912439861451492872946725151),
1270:       BOOST_MATH_BIG_CONSTANT(T, 113, 3.31108463283559911602405970817931801),
1271:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.74657006336994649386607925179848899),
1272:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.718255607416072737965933040353653244),
````
- **L1249 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1249 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1250 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1250 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1251 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1251 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1252 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1252 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1253 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1253 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1254 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1254 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1255 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1255 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1256 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1256 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1257 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1257 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1258 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1258 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1259 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1259 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1260 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1260 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1261 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1261 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1262 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1262 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1263 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1263 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1264 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1264 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1265 EN**: Continues the surrounding expression or declaration: `static const T Q[16] = {`.
  - **L1265 CN**: 继续构造周围的表达式或声明：`static const T Q[16] = {`。
- **L1266 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1266 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1267 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1267 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1268 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1268 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1269 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1269 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1270 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1270 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1271 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1271 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1272 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1272 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1273-1296 / 第 1273-1296 行

````cpp
1273:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.234037553177354542791975767960643864),
1274:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0607470145906491602476833515412605389),
1275:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0125048143774226921434854172947548724),
1276:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00201034366420433762935768458656609163),
1277:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000244823338417452367656368849303165721),
1278:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.213511655166983177960471085462540807e-4),
1279:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.119323998465870686327170541547982932e-5),
1280:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.322153582559488797803027773591727565e-7),
1281:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.161635525318683508633792845159942312e-16)
1282:    };
1283:    // LCOV_EXCL_STOP
1284:    T t = z / 8 - 4.25;
1285:    result = Y + tools::evaluate_polynomial(P, t)
1286:       / tools::evaluate_polynomial(Q, t);
1287:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1288:    result *= exp(z) / z;
1289:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1290:    result += z;
1291:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1292: }
1293: 
1294: template <class T>
1295: BOOST_MATH_GPU_ENABLED void expint_i_113f(T& result, const T& z)
1296: {
````
- **L1273 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1273 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1274 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1274 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1275 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1275 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1276 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1276 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1277 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1277 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1278 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1278 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1279 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1279 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1280 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1280 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1281 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1281 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1282 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1282 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1283 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1283 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1284 EN**: Executes a standalone statement or declaration: `T t = z / 8 - 4.25;`.
  - **L1284 CN**: 执行一条独立语句或声明：`T t = z / 8 - 4.25;`。
- **L1285 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1285 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1286 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1286 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1287 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1287 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1288 EN**: Executes a call or declaration centered on `exp`.
  - **L1288 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1289 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1289 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1290 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1290 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1291 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1291 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1292 EN**: Closes the current lexical scope or compound statement.
  - **L1292 CN**: 结束当前词法作用域或复合语句块。
- **L1293 EN**: Blank line separating nearby declarations or logic.
  - **L1293 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1294 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1294 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1295 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1295 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1296 EN**: Opens a new lexical scope or compound statement.
  - **L1296 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 1297-1320 / 第 1297-1320 行

````cpp
1297:    BOOST_MATH_STD_USING
1298:    // Maximum Deviation Found:                     4.469e-36
1299:    // Expected Error Term:                         4.468e-36
1300:    // Max Error found at long double precision =   Poly: 1.288958e-35   Cheb: 2.304586e-35
1301:    // LCOV_EXCL_START
1302:    static const T Y = 1.0216197967529296875F;
1303:    static const T P[12] = {
1304:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000322999116096627043476023926572650045),
1305:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00385606067447365187909164609294113346),
1306:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00686514524727568176735949971985244415),
1307:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00606260649593050194602676772589601799),
1308:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00334382362017147544335054575436194357),
1309:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00126108534260253075708625583630318043),
1310:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000337881489347846058951220431209276776),
1311:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.648480902304640018785370650254018022e-4),
1312:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.87652644082970492211455290209092766e-5),
1313:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.794712243338068631557849449519994144e-6),
1314:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.434084023639508143975983454830954835e-7),
1315:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.107839681938752337160494412638656696e-8)
1316:    };
1317:    static const T Q[12] = {
1318:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1319:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.09913805456661084097134805151524958),
1320:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.07041755535439919593503171320431849),
````
- **L1297 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1297 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1298 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     4.469e-36`.
  - **L1298 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     4.469e-36`。
- **L1299 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         4.468e-36`.
  - **L1299 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         4.468e-36`。
- **L1300 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 1.288958e-35   Cheb: 2.304586e-35`.
  - **L1300 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 1.288958e-35   Cheb: 2.304586e-35`。
- **L1301 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1301 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1302 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1302 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1303 EN**: Continues the surrounding expression or declaration: `static const T P[12] = {`.
  - **L1303 CN**: 继续构造周围的表达式或声明：`static const T P[12] = {`。
- **L1304 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1304 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1305 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1305 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1306 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1306 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1307 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1307 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1308 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1308 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1309 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1309 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1310 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1310 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1311 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1311 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1312 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1312 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1313 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1313 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1314 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1314 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1315 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1315 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1316 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1316 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1317 EN**: Continues the surrounding expression or declaration: `static const T Q[12] = {`.
  - **L1317 CN**: 继续构造周围的表达式或声明：`static const T Q[12] = {`。
- **L1318 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1318 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1319 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1319 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1320 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1320 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1321-1344 / 第 1321-1344 行

````cpp
1321:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.26406517226052371320416108604874734),
1322:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.529689923703770353961553223973435569),
1323:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.159578150879536711042269658656115746),
1324:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0351720877642000691155202082629857131),
1325:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00565313621289648752407123620997063122),
1326:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000646920278540515480093843570291218295),
1327:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.499904084850091676776993523323213591e-4),
1328:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.233740058688179614344680531486267142e-5),
1329:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.498800627828842754845418576305379469e-7)
1330:    };
1331:    // LCOV_EXCL_STOP
1332:    T t = z / 7 - 7;
1333:    result = Y + tools::evaluate_polynomial(P, t)
1334:       / tools::evaluate_polynomial(Q, t);
1335:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1336:    result *= exp(z) / z;
1337:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1338:    result += z;
1339:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1340: }
1341: 
1342: template <class T>
1343: BOOST_MATH_GPU_ENABLED void expint_i_113g(T& result, const T& z)
1344: {
````
- **L1321 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1321 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1322 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1322 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1323 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1323 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1324 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1324 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1325 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1325 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1326 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1326 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1327 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1327 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1328 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1328 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1329 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1329 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1330 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1330 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1331 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1331 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1332 EN**: Executes a standalone statement or declaration: `T t = z / 7 - 7;`.
  - **L1332 CN**: 执行一条独立语句或声明：`T t = z / 7 - 7;`。
- **L1333 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1333 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1334 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1334 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1335 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1335 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1336 EN**: Executes a call or declaration centered on `exp`.
  - **L1336 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1337 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1337 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1338 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1338 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1339 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1339 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1340 EN**: Closes the current lexical scope or compound statement.
  - **L1340 CN**: 结束当前词法作用域或复合语句块。
- **L1341 EN**: Blank line separating nearby declarations or logic.
  - **L1341 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1342 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1342 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1343 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1343 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1344 EN**: Opens a new lexical scope or compound statement.
  - **L1344 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 1345-1368 / 第 1345-1368 行

````cpp
1345:    BOOST_MATH_STD_USING
1346:    // Maximum Deviation Found:                     5.588e-35
1347:    // Expected Error Term:                         -5.566e-35
1348:    // Max Error found at long double precision =   Poly: 9.976345e-35   Cheb: 8.358865e-35
1349:    // LCOV_EXCL_START
1350:    static const T Y = 1.015148162841796875F;
1351:    static const T P[11] = {
1352:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000435714784725086961464589957142615216),
1353:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00432114324353830636009453048419094314),
1354:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0100740363285526177522819204820582424),
1355:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0116744115827059174392383504427640362),
1356:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00816145387784261141360062395898644652),
1357:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00371380272673500791322744465394211508),
1358:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00112958263488611536502153195005736563),
1359:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000228316462389404645183269923754256664),
1360:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.29462181955852860250359064291292577e-4),
1361:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.21972450610957417963227028788460299e-5),
1362:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.720558173805289167524715527536874694e-7)
1363:    };
1364:    static const T Q[11] = {
1365:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1366:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.95918362458402597039366979529287095),
1367:       BOOST_MATH_BIG_CONSTANT(T, 113, 3.96472247520659077944638411856748924),
1368:       BOOST_MATH_BIG_CONSTANT(T, 113, 3.15563251550528513747923714884142131),
````
- **L1345 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1345 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1346 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     5.588e-35`.
  - **L1346 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     5.588e-35`。
- **L1347 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         -5.566e-35`.
  - **L1347 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         -5.566e-35`。
- **L1348 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 9.976345e-35   Cheb: 8.358865e-35`.
  - **L1348 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 9.976345e-35   Cheb: 8.358865e-35`。
- **L1349 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1349 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1350 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1350 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1351 EN**: Continues the surrounding expression or declaration: `static const T P[11] = {`.
  - **L1351 CN**: 继续构造周围的表达式或声明：`static const T P[11] = {`。
- **L1352 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1352 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1353 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1353 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1354 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1354 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1355 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1355 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1356 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1356 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1357 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1357 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1358 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1358 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1359 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1359 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1360 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1360 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1361 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1361 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1362 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1362 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1363 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1363 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1364 EN**: Continues the surrounding expression or declaration: `static const T Q[11] = {`.
  - **L1364 CN**: 继续构造周围的表达式或声明：`static const T Q[11] = {`。
- **L1365 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1365 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1366 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1366 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1367 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1367 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1368 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1368 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1369-1392 / 第 1369-1392 行

````cpp
1369:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.64674612007093983894215359287448334),
1370:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.58695020129846594405856226787156424),
1371:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.144358385319329396231755457772362793),
1372:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.024146911506411684815134916238348063),
1373:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0026257132337460784266874572001650153),
1374:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000167479843750859222348869769094711093),
1375:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.475673638665358075556452220192497036e-5)
1376:    };
1377:    // LCOV_EXCL_STOP
1378:    T t = z / 14 - 5;
1379:    result = Y + tools::evaluate_polynomial(P, t)
1380:       / tools::evaluate_polynomial(Q, t);
1381:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1382:    result *= exp(z) / z;
1383:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1384:    result += z;
1385:    BOOST_MATH_INSTRUMENT_VARIABLE(result)
1386: }
1387: 
1388: template <class T>
1389: BOOST_MATH_GPU_ENABLED void expint_i_113h(T& result, const T& z)
1390: {
1391:    BOOST_MATH_STD_USING
1392:    // Maximum Deviation Found:                     4.448e-36
````
- **L1369 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1369 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1370 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1370 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1371 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1371 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1372 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1372 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1373 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1373 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1374 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1374 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1375 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1375 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1376 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1376 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1377 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1377 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1378 EN**: Executes a standalone statement or declaration: `T t = z / 14 - 5;`.
  - **L1378 CN**: 执行一条独立语句或声明：`T t = z / 14 - 5;`。
- **L1379 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1379 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1380 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1380 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1381 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1381 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1382 EN**: Executes a call or declaration centered on `exp`.
  - **L1382 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1383 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1383 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1384 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1384 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1385 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1385 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1386 EN**: Closes the current lexical scope or compound statement.
  - **L1386 CN**: 结束当前词法作用域或复合语句块。
- **L1387 EN**: Blank line separating nearby declarations or logic.
  - **L1387 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1388 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1388 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1389 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1389 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1390 EN**: Opens a new lexical scope or compound statement.
  - **L1390 CN**: 打开一个新的词法作用域或复合语句块。
- **L1391 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1391 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1392 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     4.448e-36`.
  - **L1392 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     4.448e-36`。

### Lines 1393-1416 / 第 1393-1416 行

````cpp
1393:    // Expected Error Term:                         4.445e-36
1394:    // Max Error found at long double precision =   Poly: 2.058532e-35   Cheb: 2.165465e-27
1395:    // LCOV_EXCL_START
1396:    static const T Y= 1.00849151611328125F;
1397:    static const T P[9] = {
1398:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0084915161132812500000001440233607358),
1399:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.84479378737716028341394223076147872),
1400:       BOOST_MATH_BIG_CONSTANT(T, 113, -130.431146923726715674081563022115568),
1401:       BOOST_MATH_BIG_CONSTANT(T, 113, 4336.26945491571504885214176203512015),
1402:       BOOST_MATH_BIG_CONSTANT(T, 113, -76279.0031974974730095170437591004177),
1403:       BOOST_MATH_BIG_CONSTANT(T, 113, 729577.956271997673695191455111727774),
1404:       BOOST_MATH_BIG_CONSTANT(T, 113, -3661928.69330208734947103004900349266),
1405:       BOOST_MATH_BIG_CONSTANT(T, 113, 8570600.041606912735872059184527855),
1406:       BOOST_MATH_BIG_CONSTANT(T, 113, -6758379.93672362080947905580906028645)
1407:    };
1408:    static const T Q[10] = {
1409:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1410:       BOOST_MATH_BIG_CONSTANT(T, 113, -99.4868026047611434569541483506091713),
1411:       BOOST_MATH_BIG_CONSTANT(T, 113, 3879.67753690517114249705089803055473),
1412:       BOOST_MATH_BIG_CONSTANT(T, 113, -76495.82413252517165830203774900806),
1413:       BOOST_MATH_BIG_CONSTANT(T, 113, 820773.726408311894342553758526282667),
1414:       BOOST_MATH_BIG_CONSTANT(T, 113, -4803087.64956923577571031564909646579),
1415:       BOOST_MATH_BIG_CONSTANT(T, 113, 14521246.227703545012713173740895477),
1416:       BOOST_MATH_BIG_CONSTANT(T, 113, -19762752.0196769712258527849159393044),
````
- **L1393 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         4.445e-36`.
  - **L1393 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         4.445e-36`。
- **L1394 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 2.058532e-35   Cheb: 2.165465e-27`.
  - **L1394 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 2.058532e-35   Cheb: 2.165465e-27`。
- **L1395 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_START`.
  - **L1395 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_START`。
- **L1396 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1396 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1397 EN**: Continues the surrounding expression or declaration: `static const T P[9] = {`.
  - **L1397 CN**: 继续构造周围的表达式或声明：`static const T P[9] = {`。
- **L1398 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1398 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1399 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1399 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1400 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1400 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1401 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1401 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1402 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1402 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1403 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1403 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1404 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1404 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1405 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1405 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1406 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1406 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1407 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1407 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1408 EN**: Continues the surrounding expression or declaration: `static const T Q[10] = {`.
  - **L1408 CN**: 继续构造周围的表达式或声明：`static const T Q[10] = {`。
- **L1409 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1409 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1410 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1410 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1411 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1411 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1412 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1412 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1413 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1413 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1414 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1414 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1415 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1415 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1416 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1416 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1417-1440 / 第 1417-1440 行

````cpp
1417:       BOOST_MATH_BIG_CONSTANT(T, 113, 8354144.67882768405803322344185185517),
1418:       BOOST_MATH_BIG_CONSTANT(T, 113, 355076.853106511136734454134915432571)
1419:    };
1420:    // LCOV_EXCL_STOP
1421:    T t = 1 / z;
1422:    result = Y + tools::evaluate_polynomial(P, t)
1423:       / tools::evaluate_polynomial(Q, t);
1424:    result *= exp(z) / z;
1425:    result += z;
1426: }
1427: 
1428: template <class T, class Policy>
1429: BOOST_MATH_GPU_ENABLED T expint_i_imp(T z, const Policy& pol, const boost::math::integral_constant<int, 113>& tag)
1430: {
1431:    BOOST_MATH_STD_USING
1432:    constexpr auto function = "boost::math::expint<%1%>(%1%)";
1433:    if(z < 0)
1434:       return -expint_imp(1, T(-z), pol, tag);
1435:    if(z == 0)
1436:       return -policies::raise_overflow_error<T>(function, nullptr, pol);
1437: 
1438:    T result;
1439: 
1440:    if(z <= 6)
````
- **L1417 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1417 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1418 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1418 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1419 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1419 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1420 EN**: Comment documents nearby intent or usage notes: `LCOV_EXCL_STOP`.
  - **L1420 CN**: 注释说明附近代码的意图或使用说明：`LCOV_EXCL_STOP`。
- **L1421 EN**: Executes a standalone statement or declaration: `T t = 1 / z;`.
  - **L1421 CN**: 执行一条独立语句或声明：`T t = 1 / z;`。
- **L1422 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1422 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1423 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1423 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1424 EN**: Executes a call or declaration centered on `exp`.
  - **L1424 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1425 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1425 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1426 EN**: Closes the current lexical scope or compound statement.
  - **L1426 CN**: 结束当前词法作用域或复合语句块。
- **L1427 EN**: Blank line separating nearby declarations or logic.
  - **L1427 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1428 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1428 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1429 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1429 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1430 EN**: Opens a new lexical scope or compound statement.
  - **L1430 CN**: 打开一个新的词法作用域或复合语句块。
- **L1431 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1431 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1432 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1432 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1433 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1433 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1434 EN**: Returns from the current function with `-expint_imp(1, T(-z), pol, tag)`.
  - **L1434 CN**: 以 `-expint_imp(1, T(-z), pol, tag)` 从当前函数返回。
- **L1435 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1435 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1436 EN**: Returns from the current function with `-policies::raise_overflow_error<T>(function, nullptr, pol)`.
  - **L1436 CN**: 以 `-policies::raise_overflow_error<T>(function, nullptr, pol)` 从当前函数返回。
- **L1437 EN**: Blank line separating nearby declarations or logic.
  - **L1437 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1438 EN**: Executes a standalone statement or declaration: `T result;`.
  - **L1438 CN**: 执行一条独立语句或声明：`T result;`。
- **L1439 EN**: Blank line separating nearby declarations or logic.
  - **L1439 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1440 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1440 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1441-1464 / 第 1441-1464 行

````cpp
1441:    {
1442:       expint_i_imp_113a(result, z, pol);
1443:    }
1444:    else if (z <= 10)
1445:    {
1446:       expint_i_113b(result, z);
1447:    }
1448:    else if(z <= 18)
1449:    {
1450:       expint_i_113c(result, z);
1451:    }
1452:    else if(z <= 26)
1453:    {
1454:       expint_i_113d(result, z);
1455:    }
1456:    else if(z <= 42)
1457:    {
1458:       expint_i_113e(result, z);
1459:    }
1460:    else if(z <= 56)
1461:    {
1462:       expint_i_113f(result, z);
1463:    }
1464:    else if(z <= 84)
````
- **L1441 EN**: Opens a new lexical scope or compound statement.
  - **L1441 CN**: 打开一个新的词法作用域或复合语句块。
- **L1442 EN**: Executes a call or declaration centered on `expint_i_imp_113a`.
  - **L1442 CN**: 执行以 `expint_i_imp_113a` 为核心的调用或声明。
- **L1443 EN**: Closes the current lexical scope or compound statement.
  - **L1443 CN**: 结束当前词法作用域或复合语句块。
- **L1444 EN**: Starts the alternative branch of the preceding conditional.
  - **L1444 CN**: 开始前一个条件语句的备选分支。
- **L1445 EN**: Opens a new lexical scope or compound statement.
  - **L1445 CN**: 打开一个新的词法作用域或复合语句块。
- **L1446 EN**: Executes a call or declaration centered on `expint_i_113b`.
  - **L1446 CN**: 执行以 `expint_i_113b` 为核心的调用或声明。
- **L1447 EN**: Closes the current lexical scope or compound statement.
  - **L1447 CN**: 结束当前词法作用域或复合语句块。
- **L1448 EN**: Starts the alternative branch of the preceding conditional.
  - **L1448 CN**: 开始前一个条件语句的备选分支。
- **L1449 EN**: Opens a new lexical scope or compound statement.
  - **L1449 CN**: 打开一个新的词法作用域或复合语句块。
- **L1450 EN**: Executes a call or declaration centered on `expint_i_113c`.
  - **L1450 CN**: 执行以 `expint_i_113c` 为核心的调用或声明。
- **L1451 EN**: Closes the current lexical scope or compound statement.
  - **L1451 CN**: 结束当前词法作用域或复合语句块。
- **L1452 EN**: Starts the alternative branch of the preceding conditional.
  - **L1452 CN**: 开始前一个条件语句的备选分支。
- **L1453 EN**: Opens a new lexical scope or compound statement.
  - **L1453 CN**: 打开一个新的词法作用域或复合语句块。
- **L1454 EN**: Executes a call or declaration centered on `expint_i_113d`.
  - **L1454 CN**: 执行以 `expint_i_113d` 为核心的调用或声明。
- **L1455 EN**: Closes the current lexical scope or compound statement.
  - **L1455 CN**: 结束当前词法作用域或复合语句块。
- **L1456 EN**: Starts the alternative branch of the preceding conditional.
  - **L1456 CN**: 开始前一个条件语句的备选分支。
- **L1457 EN**: Opens a new lexical scope or compound statement.
  - **L1457 CN**: 打开一个新的词法作用域或复合语句块。
- **L1458 EN**: Executes a call or declaration centered on `expint_i_113e`.
  - **L1458 CN**: 执行以 `expint_i_113e` 为核心的调用或声明。
- **L1459 EN**: Closes the current lexical scope or compound statement.
  - **L1459 CN**: 结束当前词法作用域或复合语句块。
- **L1460 EN**: Starts the alternative branch of the preceding conditional.
  - **L1460 CN**: 开始前一个条件语句的备选分支。
- **L1461 EN**: Opens a new lexical scope or compound statement.
  - **L1461 CN**: 打开一个新的词法作用域或复合语句块。
- **L1462 EN**: Executes a call or declaration centered on `expint_i_113f`.
  - **L1462 CN**: 执行以 `expint_i_113f` 为核心的调用或声明。
- **L1463 EN**: Closes the current lexical scope or compound statement.
  - **L1463 CN**: 结束当前词法作用域或复合语句块。
- **L1464 EN**: Starts the alternative branch of the preceding conditional.
  - **L1464 CN**: 开始前一个条件语句的备选分支。

### Lines 1465-1488 / 第 1465-1488 行

````cpp
1465:    {
1466:       expint_i_113g(result, z);
1467:    }
1468:    else if(z <= 210)
1469:    {
1470:       expint_i_113h(result, z);
1471:    }
1472:    else // z > 210
1473:    {
1474:       // Maximum Deviation Found:                     3.963e-37
1475:       // Expected Error Term:                         3.963e-37
1476:       // Max Error found at long double precision =   Poly: 1.248049e-36   Cheb: 2.843486e-29
1477: 
1478:       static const T exp40 = static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 113, 2.35385266837019985407899910749034804508871617254555467236651e17));
1479:       static const T Y= 1.00252532958984375F;
1480:       static const T P[8] = {
1481:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.00252532958984375000000000000000000085),
1482:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.16591386866059087390621952073890359),
1483:          BOOST_MATH_BIG_CONSTANT(T, 113, -67.8483431314018462417456828499277579),
1484:          BOOST_MATH_BIG_CONSTANT(T, 113, 1567.68688154683822956359536287575892),
1485:          BOOST_MATH_BIG_CONSTANT(T, 113, -17335.4683325819116482498725687644986),
1486:          BOOST_MATH_BIG_CONSTANT(T, 113, 93632.6567462673524739954389166550069),
1487:          BOOST_MATH_BIG_CONSTANT(T, 113, -225025.189335919133214440347510936787),
1488:          BOOST_MATH_BIG_CONSTANT(T, 113, 175864.614717440010942804684741336853)
````
- **L1465 EN**: Opens a new lexical scope or compound statement.
  - **L1465 CN**: 打开一个新的词法作用域或复合语句块。
- **L1466 EN**: Executes a call or declaration centered on `expint_i_113g`.
  - **L1466 CN**: 执行以 `expint_i_113g` 为核心的调用或声明。
- **L1467 EN**: Closes the current lexical scope or compound statement.
  - **L1467 CN**: 结束当前词法作用域或复合语句块。
- **L1468 EN**: Starts the alternative branch of the preceding conditional.
  - **L1468 CN**: 开始前一个条件语句的备选分支。
- **L1469 EN**: Opens a new lexical scope or compound statement.
  - **L1469 CN**: 打开一个新的词法作用域或复合语句块。
- **L1470 EN**: Executes a call or declaration centered on `expint_i_113h`.
  - **L1470 CN**: 执行以 `expint_i_113h` 为核心的调用或声明。
- **L1471 EN**: Closes the current lexical scope or compound statement.
  - **L1471 CN**: 结束当前词法作用域或复合语句块。
- **L1472 EN**: Starts the alternative branch of the preceding conditional.
  - **L1472 CN**: 开始前一个条件语句的备选分支。
- **L1473 EN**: Opens a new lexical scope or compound statement.
  - **L1473 CN**: 打开一个新的词法作用域或复合语句块。
- **L1474 EN**: Comment documents nearby intent or usage notes: `Maximum Deviation Found:                     3.963e-37`.
  - **L1474 CN**: 注释说明附近代码的意图或使用说明：`Maximum Deviation Found:                     3.963e-37`。
- **L1475 EN**: Comment documents nearby intent or usage notes: `Expected Error Term:                         3.963e-37`.
  - **L1475 CN**: 注释说明附近代码的意图或使用说明：`Expected Error Term:                         3.963e-37`。
- **L1476 EN**: Comment documents nearby intent or usage notes: `Max Error found at long double precision =   Poly: 1.248049e-36   Cheb: 2.843486e-29`.
  - **L1476 CN**: 注释说明附近代码的意图或使用说明：`Max Error found at long double precision =   Poly: 1.248049e-36   Cheb: 2.843486e-29`。
- **L1477 EN**: Blank line separating nearby declarations or logic.
  - **L1477 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1478 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1478 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1479 EN**: Initializes variable `Y` from the right-hand expression.
  - **L1479 CN**: 使用右侧表达式初始化变量 `Y`。
- **L1480 EN**: Continues the surrounding expression or declaration: `static const T P[8] = {`.
  - **L1480 CN**: 继续构造周围的表达式或声明：`static const T P[8] = {`。
- **L1481 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1481 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1482 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1482 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1483 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1483 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1484 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1484 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1485 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1485 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1486 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1486 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1487 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1487 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1488 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1488 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1489-1512 / 第 1489-1512 行

````cpp
1489:       };
1490:       static const T Q[9] = {
1491:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
1492:          BOOST_MATH_BIG_CONSTANT(T, 113, -65.6998869881600212224652719706425129),
1493:          BOOST_MATH_BIG_CONSTANT(T, 113, 1642.73850032324014781607859416890077),
1494:          BOOST_MATH_BIG_CONSTANT(T, 113, -19937.2610222467322481947237312818575),
1495:          BOOST_MATH_BIG_CONSTANT(T, 113, 124136.267326632742667972126625064538),
1496:          BOOST_MATH_BIG_CONSTANT(T, 113, -384614.251466704550678760562965502293),
1497:          BOOST_MATH_BIG_CONSTANT(T, 113, 523355.035910385688578278384032026998),
1498:          BOOST_MATH_BIG_CONSTANT(T, 113, -217809.552260834025885677791936351294),
1499:          BOOST_MATH_BIG_CONSTANT(T, 113, -8555.81719551123640677261226549550872)
1500:       };
1501:       T t = 1 / z;
1502:       result = Y + tools::evaluate_polynomial(P, t)
1503:          / tools::evaluate_polynomial(Q, t);
1504:       if(z < 41)
1505:          result *= exp(z) / z;
1506:       else
1507:       {
1508:          // Avoid premature overflow if we can:
1509:          t = z - 40;
1510:          if(t > tools::log_max_value<T>())
1511:          {
1512:             result = policies::raise_overflow_error<T>(function, nullptr, pol);
````
- **L1489 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1489 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1490 EN**: Continues the surrounding expression or declaration: `static const T Q[9] = {`.
  - **L1490 CN**: 继续构造周围的表达式或声明：`static const T Q[9] = {`。
- **L1491 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1491 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1492 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1492 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1493 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1493 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1494 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1494 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1495 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1495 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1496 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1496 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1497 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1497 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1498 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1498 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1499 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1499 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1500 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L1500 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1501 EN**: Executes a standalone statement or declaration: `T t = 1 / z;`.
  - **L1501 CN**: 执行一条独立语句或声明：`T t = 1 / z;`。
- **L1502 EN**: Continues logic associated with callable symbol `evaluate_polynomial`.
  - **L1502 CN**: 继续与可调用符号 `evaluate_polynomial` 相关的逻辑。
- **L1503 EN**: Executes a call or declaration centered on `tools::evaluate_polynomial`.
  - **L1503 CN**: 执行以 `tools::evaluate_polynomial` 为核心的调用或声明。
- **L1504 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1504 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1505 EN**: Executes a call or declaration centered on `exp`.
  - **L1505 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1506 EN**: Starts the alternative branch of the preceding conditional.
  - **L1506 CN**: 开始前一个条件语句的备选分支。
- **L1507 EN**: Opens a new lexical scope or compound statement.
  - **L1507 CN**: 打开一个新的词法作用域或复合语句块。
- **L1508 EN**: Comment documents nearby intent or usage notes: `Avoid premature overflow if we can:`.
  - **L1508 CN**: 注释说明附近代码的意图或使用说明：`Avoid premature overflow if we can:`。
- **L1509 EN**: Executes a standalone statement or declaration: `t = z - 40;`.
  - **L1509 CN**: 执行一条独立语句或声明：`t = z - 40;`。
- **L1510 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1510 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1511 EN**: Opens a new lexical scope or compound statement.
  - **L1511 CN**: 打开一个新的词法作用域或复合语句块。
- **L1512 EN**: Executes a call or declaration centered on `policies::raise_overflow_error<T>`.
  - **L1512 CN**: 执行以 `policies::raise_overflow_error<T>` 为核心的调用或声明。

### Lines 1513-1536 / 第 1513-1536 行

````cpp
1513:          }
1514:          else
1515:          {
1516:             result *= exp(z - 40) / z;
1517:             if(result > tools::max_value<T>() / exp40)
1518:             {
1519:                result = policies::raise_overflow_error<T>(function, nullptr, pol);
1520:             }
1521:             else
1522:             {
1523:                result *= exp40;
1524:             }
1525:          }
1526:       }
1527:       result += z;
1528:    }
1529:    return result;
1530: }
1531: 
1532: template <class T, class Policy>
1533: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
1534:    expint_forwarder(T z, const Policy& /*pol*/, boost::math::true_type const&)
1535: {
1536:    typedef typename tools::promote_args<T>::type result_type;
````
- **L1513 EN**: Closes the current lexical scope or compound statement.
  - **L1513 CN**: 结束当前词法作用域或复合语句块。
- **L1514 EN**: Starts the alternative branch of the preceding conditional.
  - **L1514 CN**: 开始前一个条件语句的备选分支。
- **L1515 EN**: Opens a new lexical scope or compound statement.
  - **L1515 CN**: 打开一个新的词法作用域或复合语句块。
- **L1516 EN**: Executes a call or declaration centered on `exp`.
  - **L1516 CN**: 执行以 `exp` 为核心的调用或声明。
- **L1517 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1517 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1518 EN**: Opens a new lexical scope or compound statement.
  - **L1518 CN**: 打开一个新的词法作用域或复合语句块。
- **L1519 EN**: Executes a call or declaration centered on `policies::raise_overflow_error<T>`.
  - **L1519 CN**: 执行以 `policies::raise_overflow_error<T>` 为核心的调用或声明。
- **L1520 EN**: Closes the current lexical scope or compound statement.
  - **L1520 CN**: 结束当前词法作用域或复合语句块。
- **L1521 EN**: Starts the alternative branch of the preceding conditional.
  - **L1521 CN**: 开始前一个条件语句的备选分支。
- **L1522 EN**: Opens a new lexical scope or compound statement.
  - **L1522 CN**: 打开一个新的词法作用域或复合语句块。
- **L1523 EN**: Executes a standalone statement or declaration: `result *= exp40;`.
  - **L1523 CN**: 执行一条独立语句或声明：`result *= exp40;`。
- **L1524 EN**: Closes the current lexical scope or compound statement.
  - **L1524 CN**: 结束当前词法作用域或复合语句块。
- **L1525 EN**: Closes the current lexical scope or compound statement.
  - **L1525 CN**: 结束当前词法作用域或复合语句块。
- **L1526 EN**: Closes the current lexical scope or compound statement.
  - **L1526 CN**: 结束当前词法作用域或复合语句块。
- **L1527 EN**: Executes a standalone statement or declaration: `result += z;`.
  - **L1527 CN**: 执行一条独立语句或声明：`result += z;`。
- **L1528 EN**: Closes the current lexical scope or compound statement.
  - **L1528 CN**: 结束当前词法作用域或复合语句块。
- **L1529 EN**: Returns from the current function with `result`.
  - **L1529 CN**: 以 `result` 从当前函数返回。
- **L1530 EN**: Closes the current lexical scope or compound statement.
  - **L1530 CN**: 结束当前词法作用域或复合语句块。
- **L1531 EN**: Blank line separating nearby declarations or logic.
  - **L1531 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1532 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1532 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1533 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1533 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1534 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1534 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1535 EN**: Opens a new lexical scope or compound statement.
  - **L1535 CN**: 打开一个新的词法作用域或复合语句块。
- **L1536 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T>::type result_type;`.
  - **L1536 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T>::type result_type;`。

### Lines 1537-1560 / 第 1537-1560 行

````cpp
1537:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1538:    typedef typename policies::precision<result_type, Policy>::type precision_type;
1539:    typedef typename policies::normalise<
1540:       Policy,
1541:       policies::promote_float<false>,
1542:       policies::promote_double<false>,
1543:       policies::discrete_quantile<>,
1544:       policies::assert_undefined<> >::type forwarding_policy;
1545:    typedef boost::math::integral_constant<int,
1546:       precision_type::value <= 0 ? 0 :
1547:       precision_type::value <= 53 ? 53 :
1548:       precision_type::value <= 64 ? 64 :
1549:       precision_type::value <= 113 ? 113 : 0
1550:    > tag_type;
1551: 
1552:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::expint_i_imp(static_cast<value_type>(z), forwarding_policy(), tag_type()), "boost::math::expint<%1%>(%1%)");
1553: }
1554: 
1555: template <class T>
1556: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
1557: expint_forwarder(unsigned n, T z, const boost::math::false_type&)
1558: {
1559:    return boost::math::expint(n, z, policies::policy<>());
1560: }
````
- **L1537 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L1537 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L1538 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::precision<result_type, Policy>::type precision_type;`.
  - **L1538 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::precision<result_type, Policy>::type precision_type;`。
- **L1539 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L1539 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L1540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L1540 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L1541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L1541 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L1543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L1543 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L1544 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L1544 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L1545 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int,`.
  - **L1545 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int,`。
- **L1546 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 0 ? 0 :`.
  - **L1546 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 0 ? 0 :`。
- **L1547 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 53 ? 53 :`.
  - **L1547 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 53 ? 53 :`。
- **L1548 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 64 ? 64 :`.
  - **L1548 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 64 ? 64 :`。
- **L1549 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 113 ? 113 : 0`.
  - **L1549 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 113 ? 113 : 0`。
- **L1550 EN**: Executes a standalone statement or declaration: `> tag_type;`.
  - **L1550 CN**: 执行一条独立语句或声明：`> tag_type;`。
- **L1551 EN**: Blank line separating nearby declarations or logic.
  - **L1551 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1552 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::expint_i_imp(static_cast<value_type>(z), forwarding_policy(), tag_type()), "boost::math::expint<%1%>(%1%)")`.
  - **L1552 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::expint_i_imp(static_cast<value_type>(z), forwarding_policy(), tag_type()), "boost::math::expint<%1%>(%1%)")` 从当前函数返回。
- **L1553 EN**: Closes the current lexical scope or compound statement.
  - **L1553 CN**: 结束当前词法作用域或复合语句块。
- **L1554 EN**: Blank line separating nearby declarations or logic.
  - **L1554 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1555 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1555 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1556 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1556 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1557 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1557 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1558 EN**: Opens a new lexical scope or compound statement.
  - **L1558 CN**: 打开一个新的词法作用域或复合语句块。
- **L1559 EN**: Returns from the current function with `boost::math::expint(n, z, policies::policy<>())`.
  - **L1559 CN**: 以 `boost::math::expint(n, z, policies::policy<>())` 从当前函数返回。
- **L1560 EN**: Closes the current lexical scope or compound statement.
  - **L1560 CN**: 结束当前词法作用域或复合语句块。

### Lines 1561-1584 / 第 1561-1584 行

````cpp
1561: 
1562: } // namespace detail
1563: 
1564: template <class T, class Policy>
1565: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
1566:    expint(unsigned n, T z, const Policy& /*pol*/)
1567: {
1568:    typedef typename tools::promote_args<T>::type result_type;
1569:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1570:    typedef typename policies::precision<result_type, Policy>::type precision_type;
1571:    typedef typename policies::normalise<
1572:       Policy,
1573:       policies::promote_float<false>,
1574:       policies::promote_double<false>,
1575:       policies::discrete_quantile<>,
1576:       policies::assert_undefined<> >::type forwarding_policy;
1577:    typedef boost::math::integral_constant<int,
1578:       precision_type::value <= 0 ? 0 :
1579:       precision_type::value <= 53 ? 53 :
1580:       precision_type::value <= 64 ? 64 :
1581:       precision_type::value <= 113 ? 113 : 0
1582:    > tag_type;
1583: 
1584:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::expint_imp(
````
- **L1561 EN**: Blank line separating nearby declarations or logic.
  - **L1561 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1562 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L1562 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。
- **L1563 EN**: Blank line separating nearby declarations or logic.
  - **L1563 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1564 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1564 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1565 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1565 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1566 EN**: Continues logic associated with callable symbol `expint`.
  - **L1566 CN**: 继续与可调用符号 `expint` 相关的逻辑。
- **L1567 EN**: Opens a new lexical scope or compound statement.
  - **L1567 CN**: 打开一个新的词法作用域或复合语句块。
- **L1568 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T>::type result_type;`.
  - **L1568 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T>::type result_type;`。
- **L1569 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::evaluation<result_type, Policy>::type value_type;`.
  - **L1569 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::evaluation<result_type, Policy>::type value_type;`。
- **L1570 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::precision<result_type, Policy>::type precision_type;`.
  - **L1570 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::precision<result_type, Policy>::type precision_type;`。
- **L1571 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L1571 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L1572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L1572 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L1573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L1573 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L1574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L1574 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L1576 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L1576 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L1577 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int,`.
  - **L1577 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int,`。
- **L1578 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 0 ? 0 :`.
  - **L1578 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 0 ? 0 :`。
- **L1579 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 53 ? 53 :`.
  - **L1579 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 53 ? 53 :`。
- **L1580 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 64 ? 64 :`.
  - **L1580 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 64 ? 64 :`。
- **L1581 EN**: Continues the surrounding expression or declaration: `precision_type::value <= 113 ? 113 : 0`.
  - **L1581 CN**: 继续构造周围的表达式或声明：`precision_type::value <= 113 ? 113 : 0`。
- **L1582 EN**: Executes a standalone statement or declaration: `> tag_type;`.
  - **L1582 CN**: 执行一条独立语句或声明：`> tag_type;`。
- **L1583 EN**: Blank line separating nearby declarations or logic.
  - **L1583 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1584 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::expint_imp(`.
  - **L1584 CN**: 以 `policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::expint_imp(` 从当前函数返回。

### Lines 1585-1608 / 第 1585-1608 行

````cpp
1585:       n,
1586:       static_cast<value_type>(z),
1587:       forwarding_policy(),
1588:       tag_type()), "boost::math::expint<%1%>(unsigned, %1%)");
1589: }
1590: 
1591: template <class T, class U>
1592: BOOST_MATH_GPU_ENABLED inline typename detail::expint_result<T, U>::type
1593:    expint(T const z, U const u)
1594: {
1595:    typedef typename policies::is_policy<U>::type tag_type;
1596:    return detail::expint_forwarder(z, u, tag_type());
1597: }
1598: 
1599: template <class T>
1600: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
1601:    expint(T z)
1602: {
1603:    return expint(z, policies::policy<>());
1604: }
1605: 
1606: }} // namespaces
1607: 
1608: #ifdef _MSC_VER
````
- **L1585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `n,`.
  - **L1585 CN**: 继续一个多行参数列表、初始化器或聚合项：`n,`。
- **L1586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static_cast<value_type>(z),`.
  - **L1586 CN**: 继续一个多行参数列表、初始化器或聚合项：`static_cast<value_type>(z),`。
- **L1587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `forwarding_policy(),`.
  - **L1587 CN**: 继续一个多行参数列表、初始化器或聚合项：`forwarding_policy(),`。
- **L1588 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1588 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1589 EN**: Closes the current lexical scope or compound statement.
  - **L1589 CN**: 结束当前词法作用域或复合语句块。
- **L1590 EN**: Blank line separating nearby declarations or logic.
  - **L1590 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1591 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1591 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1592 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1592 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1593 EN**: Continues logic associated with callable symbol `expint`.
  - **L1593 CN**: 继续与可调用符号 `expint` 相关的逻辑。
- **L1594 EN**: Opens a new lexical scope or compound statement.
  - **L1594 CN**: 打开一个新的词法作用域或复合语句块。
- **L1595 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::is_policy<U>::type tag_type;`.
  - **L1595 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::is_policy<U>::type tag_type;`。
- **L1596 EN**: Returns from the current function with `detail::expint_forwarder(z, u, tag_type())`.
  - **L1596 CN**: 以 `detail::expint_forwarder(z, u, tag_type())` 从当前函数返回。
- **L1597 EN**: Closes the current lexical scope or compound statement.
  - **L1597 CN**: 结束当前词法作用域或复合语句块。
- **L1598 EN**: Blank line separating nearby declarations or logic.
  - **L1598 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1599 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1599 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1600 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1600 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1601 EN**: Continues logic associated with callable symbol `expint`.
  - **L1601 CN**: 继续与可调用符号 `expint` 相关的逻辑。
- **L1602 EN**: Opens a new lexical scope or compound statement.
  - **L1602 CN**: 打开一个新的词法作用域或复合语句块。
- **L1603 EN**: Returns from the current function with `expint(z, policies::policy<>())`.
  - **L1603 CN**: 以 `expint(z, policies::policy<>())` 从当前函数返回。
- **L1604 EN**: Closes the current lexical scope or compound statement.
  - **L1604 CN**: 结束当前词法作用域或复合语句块。
- **L1605 EN**: Blank line separating nearby declarations or logic.
  - **L1605 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1606 EN**: Continues the surrounding expression or declaration: `}} // namespaces`.
  - **L1606 CN**: 继续构造周围的表达式或声明：`}} // namespaces`。
- **L1607 EN**: Blank line separating nearby declarations or logic.
  - **L1607 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1608 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L1608 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。

### Lines 1609-1614 / 第 1609-1614 行

````cpp
1609: #pragma warning(pop)
1610: #endif
1611: 
1612: #endif // BOOST_MATH_EXPINT_HPP
1613: 
1614: 
````
- **L1609 EN**: Continues logic associated with callable symbol `warning`.
  - **L1609 CN**: 继续与可调用符号 `warning` 相关的逻辑。
- **L1610 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1610 CN**: 结束当前预处理条件块或头文件保护。
- **L1611 EN**: Blank line separating nearby declarations or logic.
  - **L1611 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1612 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1612 CN**: 结束当前预处理条件块或头文件保护。
- **L1613 EN**: Blank line separating nearby declarations or logic.
  - **L1613 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1614 EN**: Blank line separating nearby declarations or logic.
  - **L1614 CN**: 空行，用于分隔相邻声明或逻辑。

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

- **Direct local/internal includes / 直接本地或内部包含**: `boost/math/tools/config.hpp`, `boost/math/tools/cstdint.hpp`, `boost/math/tools/type_traits.hpp`, `boost/math/tools/tuple.hpp`, `boost/math/tools/precision.hpp`, `boost/math/tools/promotion.hpp`, `boost/math/tools/fraction.hpp`, `boost/math/tools/series.hpp`, `boost/math/policies/error_handling.hpp`, `boost/math/special_functions/math_fwd.hpp`, `boost/math/special_functions/digamma.hpp`, `boost/math/special_functions/log1p.hpp`
- **Dependency categories / 依赖类别**: Boost.Math numeric tool helpers / Boost.Math 数值工具辅助逻辑 (8), Boost.Math special-function declarations / Boost.Math 特殊函数声明 (3), Boost.Math policy configuration / Boost.Math 策略配置 (1)

- **EN**: `boost/math/tools/config.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/config.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/cstdint.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/cstdint.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/type_traits.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/type_traits.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/tuple.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/tuple.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/precision.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/precision.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/promotion.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/promotion.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/fraction.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/fraction.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/series.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/series.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/policies/error_handling.hpp` provides Boost.Math policy configuration.
  - **CN**: `boost/math/policies/error_handling.hpp` 提供Boost.Math 策略配置。
- **EN**: `boost/math/special_functions/math_fwd.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/math_fwd.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/digamma.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/digamma.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `boost/math/special_functions/log1p.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/log1p.hpp` 提供Boost.Math 特殊函数声明。
