# math_fwd.hpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/math_fwd.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/` (`llvm-project`)
- **Purpose / 目的**:
  - **EN**: This header tODO revise completely for new distribution classes.
  - **CN**: 声明 Boost.Math 的特殊函数、近似计算内核以及按类型泛化的数值入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

````cpp
   1: // math_fwd.hpp
   2: 
   3: // TODO revise completely for new distribution classes.
   4: 
   5: // Copyright Paul A. Bristow 2006.
   6: // Copyright John Maddock 2006.
   7: // Copyright Matt Borland 2024
   8: 
   9: // Use, modification and distribution are subject to the
  10: // Boost Software License, Version 1.0.
  11: // (See accompanying file LICENSE_1_0.txt
  12: // or copy at http://www.boost.org/LICENSE_1_0.txt)
  13: 
  14: // Omnibus list of forward declarations of math special functions.
  15: 
  16: // IT = Integer type.
  17: // RT = Real type (built-in floating-point types, float, double, long double) & User Defined Types
  18: // AT = Integer or Real type
  19: 
  20: #ifndef BOOST_MATH_SPECIAL_MATH_FWD_HPP
  21: #define BOOST_MATH_SPECIAL_MATH_FWD_HPP
  22: 
  23: #ifdef _MSC_VER
  24: #pragma once
  25: #endif
  26: 
  27: #include <boost/math/tools/config.hpp>
  28: #include <boost/math/tools/promotion.hpp> // for argument promotion.
````
- **L1 EN**: Comment documents nearby intent or usage notes: `math_fwd.hpp`.
  - **L1 CN**: 注释说明附近代码的意图或使用说明：`math_fwd.hpp`。
- **L2 EN**: Blank line separating nearby declarations or logic.
  - **L2 CN**: 空行，用于分隔相邻声明或逻辑。
- **L3 EN**: Comment documents nearby intent or usage notes: `TODO revise completely for new distribution classes.`.
  - **L3 CN**: 注释说明附近代码的意图或使用说明：`TODO revise completely for new distribution classes.`。
- **L4 EN**: Blank line separating nearby declarations or logic.
  - **L4 CN**: 空行，用于分隔相邻声明或逻辑。
- **L5 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L5 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L6 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L6 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L7 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L7 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L8 EN**: Blank line separating nearby declarations or logic.
  - **L8 CN**: 空行，用于分隔相邻声明或逻辑。
- **L9 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L9 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L10 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L10 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L11 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L11 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L12 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L12 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L13 EN**: Blank line separating nearby declarations or logic.
  - **L13 CN**: 空行，用于分隔相邻声明或逻辑。
- **L14 EN**: Comment documents nearby intent or usage notes: `Omnibus list of forward declarations of math special functions.`.
  - **L14 CN**: 注释说明附近代码的意图或使用说明：`Omnibus list of forward declarations of math special functions.`。
- **L15 EN**: Blank line separating nearby declarations or logic.
  - **L15 CN**: 空行，用于分隔相邻声明或逻辑。
- **L16 EN**: Comment documents nearby intent or usage notes: `IT = Integer type.`.
  - **L16 CN**: 注释说明附近代码的意图或使用说明：`IT = Integer type.`。
- **L17 EN**: Comment documents nearby intent or usage notes: `RT = Real type (built-in floating-point types, float, double, long double) & User Defined Types`.
  - **L17 CN**: 注释说明附近代码的意图或使用说明：`RT = Real type (built-in floating-point types, float, double, long double) & User Defined Types`。
- **L18 EN**: Comment documents nearby intent or usage notes: `AT = Integer or Real type`.
  - **L18 CN**: 注释说明附近代码的意图或使用说明：`AT = Integer or Real type`。
- **L19 EN**: Blank line separating nearby declarations or logic.
  - **L19 CN**: 空行，用于分隔相邻声明或逻辑。
- **L20 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_SPECIAL_MATH_FWD_HPP`.
  - **L20 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_SPECIAL_MATH_FWD_HPP`。
- **L21 EN**: Defines macro `BOOST_MATH_SPECIAL_MATH_FWD_HPP` for compile-time control, shorthand, or generated boilerplate.
  - **L21 CN**: 定义宏 `BOOST_MATH_SPECIAL_MATH_FWD_HPP`，用于编译期控制、简写或生成样板代码。
- **L22 EN**: Blank line separating nearby declarations or logic.
  - **L22 CN**: 空行，用于分隔相邻声明或逻辑。
- **L23 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  - **L23 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L24 EN**: Uses `#pragma once` to prevent multiple inclusion of this header.
  - **L24 CN**: 使用 `#pragma once` 防止该头文件被重复包含。
- **L25 EN**: Closes the current preprocessor conditional block or header guard.
  - **L25 CN**: 结束当前预处理条件块或头文件保护。
- **L26 EN**: Blank line separating nearby declarations or logic.
  - **L26 CN**: 空行，用于分隔相邻声明或逻辑。
- **L27 EN**: Includes <boost/math/tools/config.hpp> to access Boost.Math numeric tool helpers.
  - **L27 CN**: 引入 <boost/math/tools/config.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L28 EN**: Includes <boost/math/tools/promotion.hpp> to access Boost.Math numeric tool helpers.
  - **L28 CN**: 引入 <boost/math/tools/promotion.hpp> 以使用Boost.Math 数值工具辅助逻辑。

### Lines 29-56 / 第 29-56 行

````cpp
  29: #include <boost/math/tools/type_traits.hpp>
  30: #include <boost/math/tools/complex.hpp>
  31: #include <boost/math/policies/policy.hpp>
  32: 
  33: #ifdef BOOST_MATH_HAS_NVRTC
  34: 
  35: namespace boost {
  36: namespace math {
  37: 
  38: template <class RT1, class RT2, class A>
  39: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, A>::type
  40: beta(RT1 a, RT2 b, A arg);
  41: 
  42: namespace detail{
  43: 
  44:    template <class T, class U, class V>
  45:    struct ellint_3_result
  46:    {
  47:       using type = typename boost::math::conditional<
  48:          policies::is_policy<V>::value,
  49:          tools::promote_args_t<T, U>,
  50:          tools::promote_args_t<T, U, V>
  51:       >::type;
  52:    };
  53: 
  54:    template <class T, class U>
  55:    struct expint_result
  56:    {
````
- **L29 EN**: Includes <boost/math/tools/type_traits.hpp> to access Boost.Math numeric tool helpers.
  - **L29 CN**: 引入 <boost/math/tools/type_traits.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L30 EN**: Includes <boost/math/tools/complex.hpp> to access Boost.Math numeric tool helpers.
  - **L30 CN**: 引入 <boost/math/tools/complex.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L31 EN**: Includes <boost/math/policies/policy.hpp> to access Boost.Math policy configuration.
  - **L31 CN**: 引入 <boost/math/policies/policy.hpp> 以使用Boost.Math 策略配置。
- **L32 EN**: Blank line separating nearby declarations or logic.
  - **L32 CN**: 空行，用于分隔相邻声明或逻辑。
- **L33 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_HAS_NVRTC`.
  - **L33 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_HAS_NVRTC`。
- **L34 EN**: Blank line separating nearby declarations or logic.
  - **L34 CN**: 空行，用于分隔相邻声明或逻辑。
- **L35 EN**: Opens namespace scope `boost`.
  - **L35 CN**: 打开命名空间作用域 `boost`。
- **L36 EN**: Opens namespace scope `math`.
  - **L36 CN**: 打开命名空间作用域 `math`。
- **L37 EN**: Blank line separating nearby declarations or logic.
  - **L37 CN**: 空行，用于分隔相邻声明或逻辑。
- **L38 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class A>`.
  - **L38 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class A>`。
- **L39 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L39 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L40 EN**: Executes a call or declaration centered on `beta`.
  - **L40 CN**: 执行以 `beta` 为核心的调用或声明。
- **L41 EN**: Blank line separating nearby declarations or logic.
  - **L41 CN**: 空行，用于分隔相邻声明或逻辑。
- **L42 EN**: Opens namespace scope `detail`.
  - **L42 CN**: 打开命名空间作用域 `detail`。
- **L43 EN**: Blank line separating nearby declarations or logic.
  - **L43 CN**: 空行，用于分隔相邻声明或逻辑。
- **L44 EN**: Introduces template parameters or specialization context: `template <class T, class U, class V>`.
  - **L44 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class V>`。
- **L45 EN**: Declares struct `ellint_3_result`.
  - **L45 CN**: 声明 struct `ellint_3_result`。
- **L46 EN**: Opens a new lexical scope or compound statement.
  - **L46 CN**: 打开一个新的词法作用域或复合语句块。
- **L47 EN**: Defines alias `type` to simplify later code.
  - **L47 CN**: 定义别名 `type` 以简化后续代码。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::is_policy<V>::value,`.
  - **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::is_policy<V>::value,`。
- **L49 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tools::promote_args_t<T, U>,`.
  - **L49 CN**: 继续一个多行参数列表、初始化器或聚合项：`tools::promote_args_t<T, U>,`。
- **L50 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T, U, V>`.
  - **L50 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T, U, V>`。
- **L51 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L51 CN**: 执行一条独立语句或声明：`>::type;`。
- **L52 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L52 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L53 EN**: Blank line separating nearby declarations or logic.
  - **L53 CN**: 空行，用于分隔相邻声明或逻辑。
- **L54 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L54 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L55 EN**: Declares struct `expint_result`.
  - **L55 CN**: 声明 struct `expint_result`。
- **L56 EN**: Opens a new lexical scope or compound statement.
  - **L56 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 57-84 / 第 57-84 行

````cpp
  57:       using type = typename boost::math::conditional<
  58:          policies::is_policy<U>::value,
  59:          tools::promote_args_t<T>,
  60:          typename tools::promote_args<U>::type
  61:       >::type;
  62:    };
  63: 
  64:    typedef boost::math::integral_constant<int, 0> bessel_no_int_tag;      // No integer optimisation possible.
  65:    typedef boost::math::integral_constant<int, 1> bessel_maybe_int_tag;   // Maybe integer optimisation.
  66:    typedef boost::math::integral_constant<int, 2> bessel_int_tag;         // Definite integer optimisation.
  67: 
  68:    template <class T1, class T2, class Policy>
  69:    struct bessel_traits
  70:    {
  71:       using result_type = typename boost::math::conditional<
  72:          boost::math::is_integral<T1>::value,
  73:          typename tools::promote_args<T2>::type,
  74:          tools::promote_args_t<T1, T2>
  75:       >::type;
  76: 
  77:       typedef typename policies::precision<result_type, Policy>::type precision_type;
  78: 
  79:       using optimisation_tag = typename boost::math::conditional<
  80:          (precision_type::value <= 0 || precision_type::value > 64),
  81:          bessel_no_int_tag,
  82:          typename boost::math::conditional<
  83:             boost::math::is_integral<T1>::value,
  84:             bessel_int_tag,
````
- **L57 EN**: Defines alias `type` to simplify later code.
  - **L57 CN**: 定义别名 `type` 以简化后续代码。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::is_policy<U>::value,`.
  - **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::is_policy<U>::value,`。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tools::promote_args_t<T>,`.
  - **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`tools::promote_args_t<T>,`。
- **L60 EN**: Continues the surrounding expression or declaration: `typename tools::promote_args<U>::type`.
  - **L60 CN**: 继续构造周围的表达式或声明：`typename tools::promote_args<U>::type`。
- **L61 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L61 CN**: 执行一条独立语句或声明：`>::type;`。
- **L62 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L62 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L63 EN**: Blank line separating nearby declarations or logic.
  - **L63 CN**: 空行，用于分隔相邻声明或逻辑。
- **L64 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int, 0> bessel_no_int_tag;      // No integer optimisation possible.`.
  - **L64 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int, 0> bessel_no_int_tag;      // No integer optimisation possible.`。
- **L65 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int, 1> bessel_maybe_int_tag;   // Maybe integer optimisation.`.
  - **L65 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int, 1> bessel_maybe_int_tag;   // Maybe integer optimisation.`。
- **L66 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int, 2> bessel_int_tag;         // Definite integer optimisation.`.
  - **L66 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int, 2> bessel_int_tag;         // Definite integer optimisation.`。
- **L67 EN**: Blank line separating nearby declarations or logic.
  - **L67 CN**: 空行，用于分隔相邻声明或逻辑。
- **L68 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L68 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L69 EN**: Declares struct `bessel_traits`.
  - **L69 CN**: 声明 struct `bessel_traits`。
- **L70 EN**: Opens a new lexical scope or compound statement.
  - **L70 CN**: 打开一个新的词法作用域或复合语句块。
- **L71 EN**: Defines alias `result_type` to simplify later code.
  - **L71 CN**: 定义别名 `result_type` 以简化后续代码。
- **L72 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L72 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `typename tools::promote_args<T2>::type,`.
  - **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`typename tools::promote_args<T2>::type,`。
- **L74 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2>`.
  - **L74 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2>`。
- **L75 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L75 CN**: 执行一条独立语句或声明：`>::type;`。
- **L76 EN**: Blank line separating nearby declarations or logic.
  - **L76 CN**: 空行，用于分隔相邻声明或逻辑。
- **L77 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::precision<result_type, Policy>::type precision_type;`.
  - **L77 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::precision<result_type, Policy>::type precision_type;`。
- **L78 EN**: Blank line separating nearby declarations or logic.
  - **L78 CN**: 空行，用于分隔相邻声明或逻辑。
- **L79 EN**: Defines alias `optimisation_tag` to simplify later code.
  - **L79 CN**: 定义别名 `optimisation_tag` 以简化后续代码。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(precision_type::value <= 0 || precision_type::value > 64),`.
  - **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`(precision_type::value <= 0 || precision_type::value > 64),`。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_no_int_tag,`.
  - **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_no_int_tag,`。
- **L82 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L82 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L83 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L83 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L84 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_int_tag,`.
  - **L84 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_int_tag,`。

### Lines 85-112 / 第 85-112 行

````cpp
  85:             bessel_maybe_int_tag
  86:          >::type
  87:       >::type;
  88: 
  89:       using optimisation_tag128 = typename boost::math::conditional<
  90:          (precision_type::value <= 0 || precision_type::value > 113),
  91:          bessel_no_int_tag,
  92:          typename boost::math::conditional<
  93:             boost::math::is_integral<T1>::value,
  94:             bessel_int_tag,
  95:             bessel_maybe_int_tag
  96:          >::type
  97:       >::type;
  98:    };
  99: 
 100: } // namespace detail
 101: 
 102: } // namespace math
 103: } // namespace boost
 104: 
 105: #else
 106: 
 107: #include <vector>
 108: #include <complex>
 109: #include <type_traits>
 110: #include <boost/math/special_functions/detail/round_fwd.hpp>
 111: #include <boost/math/tools/type_traits.hpp>
 112: #include <boost/math/policies/policy.hpp>
````
- **L85 EN**: Continues the surrounding expression or declaration: `bessel_maybe_int_tag`.
  - **L85 CN**: 继续构造周围的表达式或声明：`bessel_maybe_int_tag`。
- **L86 EN**: Continues the surrounding expression or declaration: `>::type`.
  - **L86 CN**: 继续构造周围的表达式或声明：`>::type`。
- **L87 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L87 CN**: 执行一条独立语句或声明：`>::type;`。
- **L88 EN**: Blank line separating nearby declarations or logic.
  - **L88 CN**: 空行，用于分隔相邻声明或逻辑。
- **L89 EN**: Defines alias `optimisation_tag128` to simplify later code.
  - **L89 CN**: 定义别名 `optimisation_tag128` 以简化后续代码。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(precision_type::value <= 0 || precision_type::value > 113),`.
  - **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`(precision_type::value <= 0 || precision_type::value > 113),`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_no_int_tag,`.
  - **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_no_int_tag,`。
- **L92 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L92 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L93 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L93 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_int_tag,`.
  - **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_int_tag,`。
- **L95 EN**: Continues the surrounding expression or declaration: `bessel_maybe_int_tag`.
  - **L95 CN**: 继续构造周围的表达式或声明：`bessel_maybe_int_tag`。
- **L96 EN**: Continues the surrounding expression or declaration: `>::type`.
  - **L96 CN**: 继续构造周围的表达式或声明：`>::type`。
- **L97 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L97 CN**: 执行一条独立语句或声明：`>::type;`。
- **L98 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L98 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L99 EN**: Blank line separating nearby declarations or logic.
  - **L99 CN**: 空行，用于分隔相邻声明或逻辑。
- **L100 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L100 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。
- **L101 EN**: Blank line separating nearby declarations or logic.
  - **L101 CN**: 空行，用于分隔相邻声明或逻辑。
- **L102 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace math`.
  - **L102 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace math`。
- **L103 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace boost`.
  - **L103 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace boost`。
- **L104 EN**: Blank line separating nearby declarations or logic.
  - **L104 CN**: 空行，用于分隔相邻声明或逻辑。
- **L105 EN**: Continues the current preprocessor branch selection.
  - **L105 CN**: 继续当前的预处理分支选择。
- **L106 EN**: Blank line separating nearby declarations or logic.
  - **L106 CN**: 空行，用于分隔相邻声明或逻辑。
- **L107 EN**: Includes <vector> to access C or C++ standard library facilities.
  - **L107 CN**: 引入 <vector> 以使用C 或 C++ 标准库设施。
- **L108 EN**: Includes <complex> to access C or C++ standard library facilities.
  - **L108 CN**: 引入 <complex> 以使用C 或 C++ 标准库设施。
- **L109 EN**: Includes <type_traits> to access C or C++ standard library facilities.
  - **L109 CN**: 引入 <type_traits> 以使用C 或 C++ 标准库设施。
- **L110 EN**: Includes <boost/math/special_functions/detail/round_fwd.hpp> to access Boost.Math special-function declarations.
  - **L110 CN**: 引入 <boost/math/special_functions/detail/round_fwd.hpp> 以使用Boost.Math 特殊函数声明。
- **L111 EN**: Includes <boost/math/tools/type_traits.hpp> to access Boost.Math numeric tool helpers.
  - **L111 CN**: 引入 <boost/math/tools/type_traits.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L112 EN**: Includes <boost/math/policies/policy.hpp> to access Boost.Math policy configuration.
  - **L112 CN**: 引入 <boost/math/policies/policy.hpp> 以使用Boost.Math 策略配置。

### Lines 113-140 / 第 113-140 行

````cpp
 113: 
 114: #define BOOST_NO_MACRO_EXPAND /**/
 115: 
 116: namespace boost
 117: {
 118:    namespace math
 119:    { // Math functions (in roughly alphabetic order).
 120: 
 121:    // Beta functions.
 122:    template <class RT1, class RT2>
 123:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2>
 124:          beta(RT1 a, RT2 b); // Beta function (2 arguments).
 125: 
 126:    template <class RT1, class RT2, class A>
 127:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, A>
 128:          beta(RT1 a, RT2 b, A x); // Beta function (3 arguments).
 129: 
 130:    template <class RT1, class RT2, class RT3, class Policy>
 131:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 132:          beta(RT1 a, RT2 b, RT3 x, const Policy& pol); // Beta function (3 arguments).
 133: 
 134:    template <class RT1, class RT2, class RT3>
 135:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 136:          betac(RT1 a, RT2 b, RT3 x);
 137: 
 138:    template <class RT1, class RT2, class RT3, class Policy>
 139:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 140:          betac(RT1 a, RT2 b, RT3 x, const Policy& pol);
````
- **L113 EN**: Blank line separating nearby declarations or logic.
  - **L113 CN**: 空行，用于分隔相邻声明或逻辑。
- **L114 EN**: Defines macro `BOOST_NO_MACRO_EXPAND` for compile-time control, shorthand, or generated boilerplate.
  - **L114 CN**: 定义宏 `BOOST_NO_MACRO_EXPAND`，用于编译期控制、简写或生成样板代码。
- **L115 EN**: Blank line separating nearby declarations or logic.
  - **L115 CN**: 空行，用于分隔相邻声明或逻辑。
- **L116 EN**: Continues the surrounding expression or declaration: `namespace boost`.
  - **L116 CN**: 继续构造周围的表达式或声明：`namespace boost`。
- **L117 EN**: Opens a new lexical scope or compound statement.
  - **L117 CN**: 打开一个新的词法作用域或复合语句块。
- **L118 EN**: Continues the surrounding expression or declaration: `namespace math`.
  - **L118 CN**: 继续构造周围的表达式或声明：`namespace math`。
- **L119 EN**: Continues logic associated with callable symbol `functions`.
  - **L119 CN**: 继续与可调用符号 `functions` 相关的逻辑。
- **L120 EN**: Blank line separating nearby declarations or logic.
  - **L120 CN**: 空行，用于分隔相邻声明或逻辑。
- **L121 EN**: Comment documents nearby intent or usage notes: `Beta functions.`.
  - **L121 CN**: 注释说明附近代码的意图或使用说明：`Beta functions.`。
- **L122 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>`.
  - **L122 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>`。
- **L123 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L123 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L124 EN**: Continues logic associated with callable symbol `beta`.
  - **L124 CN**: 继续与可调用符号 `beta` 相关的逻辑。
- **L125 EN**: Blank line separating nearby declarations or logic.
  - **L125 CN**: 空行，用于分隔相邻声明或逻辑。
- **L126 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class A>`.
  - **L126 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class A>`。
- **L127 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L127 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L128 EN**: Continues logic associated with callable symbol `beta`.
  - **L128 CN**: 继续与可调用符号 `beta` 相关的逻辑。
- **L129 EN**: Blank line separating nearby declarations or logic.
  - **L129 CN**: 空行，用于分隔相邻声明或逻辑。
- **L130 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L130 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L131 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L131 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L132 EN**: Continues logic associated with callable symbol `beta`.
  - **L132 CN**: 继续与可调用符号 `beta` 相关的逻辑。
- **L133 EN**: Blank line separating nearby declarations or logic.
  - **L133 CN**: 空行，用于分隔相邻声明或逻辑。
- **L134 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L134 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L135 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L135 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L136 EN**: Executes a call or declaration centered on `betac`.
  - **L136 CN**: 执行以 `betac` 为核心的调用或声明。
- **L137 EN**: Blank line separating nearby declarations or logic.
  - **L137 CN**: 空行，用于分隔相邻声明或逻辑。
- **L138 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L138 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L139 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L139 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L140 EN**: Executes a call or declaration centered on `betac`.
  - **L140 CN**: 执行以 `betac` 为核心的调用或声明。

### Lines 141-168 / 第 141-168 行

````cpp
 141: 
 142:    template <class RT1, class RT2, class RT3>
 143:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 144:          ibeta(RT1 a, RT2 b, RT3 x); // Incomplete beta function.
 145: 
 146:    template <class RT1, class RT2, class RT3, class Policy>
 147:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 148:          ibeta(RT1 a, RT2 b, RT3 x, const Policy& pol); // Incomplete beta function.
 149: 
 150:    template <class RT1, class RT2, class RT3>
 151:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 152:          ibetac(RT1 a, RT2 b, RT3 x); // Incomplete beta complement function.
 153: 
 154:    template <class RT1, class RT2, class RT3, class Policy>
 155:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 156:          ibetac(RT1 a, RT2 b, RT3 x, const Policy& pol); // Incomplete beta complement function.
 157: 
 158:    template <class T1, class T2, class T3, class T4>
 159:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3, T4>
 160:          ibeta_inv(T1 a, T2 b, T3 p, T4* py);
 161: 
 162:    template <class T1, class T2, class T3, class T4, class Policy>
 163:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3, T4>
 164:          ibeta_inv(T1 a, T2 b, T3 p, T4* py, const Policy& pol);
 165: 
 166:    template <class RT1, class RT2, class RT3>
 167:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 168:          ibeta_inv(RT1 a, RT2 b, RT3 p); // Incomplete beta inverse function.
````
- **L141 EN**: Blank line separating nearby declarations or logic.
  - **L141 CN**: 空行，用于分隔相邻声明或逻辑。
- **L142 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L142 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L143 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L143 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L144 EN**: Continues logic associated with callable symbol `ibeta`.
  - **L144 CN**: 继续与可调用符号 `ibeta` 相关的逻辑。
- **L145 EN**: Blank line separating nearby declarations or logic.
  - **L145 CN**: 空行，用于分隔相邻声明或逻辑。
- **L146 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L146 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L147 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L147 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L148 EN**: Continues logic associated with callable symbol `ibeta`.
  - **L148 CN**: 继续与可调用符号 `ibeta` 相关的逻辑。
- **L149 EN**: Blank line separating nearby declarations or logic.
  - **L149 CN**: 空行，用于分隔相邻声明或逻辑。
- **L150 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L150 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L151 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L151 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L152 EN**: Continues logic associated with callable symbol `ibetac`.
  - **L152 CN**: 继续与可调用符号 `ibetac` 相关的逻辑。
- **L153 EN**: Blank line separating nearby declarations or logic.
  - **L153 CN**: 空行，用于分隔相邻声明或逻辑。
- **L154 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L154 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L155 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L155 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L156 EN**: Continues logic associated with callable symbol `ibetac`.
  - **L156 CN**: 继续与可调用符号 `ibetac` 相关的逻辑。
- **L157 EN**: Blank line separating nearby declarations or logic.
  - **L157 CN**: 空行，用于分隔相邻声明或逻辑。
- **L158 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4>`.
  - **L158 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4>`。
- **L159 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L159 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L160 EN**: Executes a call or declaration centered on `ibeta_inv`.
  - **L160 CN**: 执行以 `ibeta_inv` 为核心的调用或声明。
- **L161 EN**: Blank line separating nearby declarations or logic.
  - **L161 CN**: 空行，用于分隔相邻声明或逻辑。
- **L162 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4, class Policy>`.
  - **L162 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4, class Policy>`。
- **L163 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L163 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L164 EN**: Executes a call or declaration centered on `ibeta_inv`.
  - **L164 CN**: 执行以 `ibeta_inv` 为核心的调用或声明。
- **L165 EN**: Blank line separating nearby declarations or logic.
  - **L165 CN**: 空行，用于分隔相邻声明或逻辑。
- **L166 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L166 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L167 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L167 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L168 EN**: Continues logic associated with callable symbol `ibeta_inv`.
  - **L168 CN**: 继续与可调用符号 `ibeta_inv` 相关的逻辑。

### Lines 169-196 / 第 169-196 行

````cpp
 169: 
 170:    template <class RT1, class RT2, class RT3, class Policy>
 171:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 172:          ibeta_inv(RT1 a, RT2 b, RT3 p, const Policy&); // Incomplete beta inverse function.
 173: 
 174:    template <class RT1, class RT2, class RT3>
 175:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 176:          ibeta_inva(RT1 a, RT2 b, RT3 p); // Incomplete beta inverse function.
 177: 
 178:    template <class RT1, class RT2, class RT3, class Policy>
 179:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 180:          ibeta_inva(RT1 a, RT2 b, RT3 p, const Policy&); // Incomplete beta inverse function.
 181: 
 182:    template <class RT1, class RT2, class RT3>
 183:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 184:          ibeta_invb(RT1 a, RT2 b, RT3 p); // Incomplete beta inverse function.
 185: 
 186:    template <class RT1, class RT2, class RT3, class Policy>
 187:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 188:          ibeta_invb(RT1 a, RT2 b, RT3 p, const Policy&); // Incomplete beta inverse function.
 189: 
 190:    template <class T1, class T2, class T3, class T4>
 191:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3, T4>
 192:          ibetac_inv(T1 a, T2 b, T3 q, T4* py);
 193: 
 194:    template <class T1, class T2, class T3, class T4, class Policy>
 195:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3, T4>
 196:          ibetac_inv(T1 a, T2 b, T3 q, T4* py, const Policy& pol);
````
- **L169 EN**: Blank line separating nearby declarations or logic.
  - **L169 CN**: 空行，用于分隔相邻声明或逻辑。
- **L170 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L170 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L171 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L171 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L172 EN**: Continues logic associated with callable symbol `ibeta_inv`.
  - **L172 CN**: 继续与可调用符号 `ibeta_inv` 相关的逻辑。
- **L173 EN**: Blank line separating nearby declarations or logic.
  - **L173 CN**: 空行，用于分隔相邻声明或逻辑。
- **L174 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L174 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L175 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L175 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L176 EN**: Continues logic associated with callable symbol `ibeta_inva`.
  - **L176 CN**: 继续与可调用符号 `ibeta_inva` 相关的逻辑。
- **L177 EN**: Blank line separating nearby declarations or logic.
  - **L177 CN**: 空行，用于分隔相邻声明或逻辑。
- **L178 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L178 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L179 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L179 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L180 EN**: Continues logic associated with callable symbol `ibeta_inva`.
  - **L180 CN**: 继续与可调用符号 `ibeta_inva` 相关的逻辑。
- **L181 EN**: Blank line separating nearby declarations or logic.
  - **L181 CN**: 空行，用于分隔相邻声明或逻辑。
- **L182 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L182 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L183 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L183 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L184 EN**: Continues logic associated with callable symbol `ibeta_invb`.
  - **L184 CN**: 继续与可调用符号 `ibeta_invb` 相关的逻辑。
- **L185 EN**: Blank line separating nearby declarations or logic.
  - **L185 CN**: 空行，用于分隔相邻声明或逻辑。
- **L186 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L186 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L187 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L187 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L188 EN**: Continues logic associated with callable symbol `ibeta_invb`.
  - **L188 CN**: 继续与可调用符号 `ibeta_invb` 相关的逻辑。
- **L189 EN**: Blank line separating nearby declarations or logic.
  - **L189 CN**: 空行，用于分隔相邻声明或逻辑。
- **L190 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4>`.
  - **L190 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4>`。
- **L191 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L191 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L192 EN**: Executes a call or declaration centered on `ibetac_inv`.
  - **L192 CN**: 执行以 `ibetac_inv` 为核心的调用或声明。
- **L193 EN**: Blank line separating nearby declarations or logic.
  - **L193 CN**: 空行，用于分隔相邻声明或逻辑。
- **L194 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4, class Policy>`.
  - **L194 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4, class Policy>`。
- **L195 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L195 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L196 EN**: Executes a call or declaration centered on `ibetac_inv`.
  - **L196 CN**: 执行以 `ibetac_inv` 为核心的调用或声明。

### Lines 197-224 / 第 197-224 行

````cpp
 197: 
 198:    template <class RT1, class RT2, class RT3>
 199:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 200:          ibetac_inv(RT1 a, RT2 b, RT3 q); // Incomplete beta complement inverse function.
 201: 
 202:    template <class RT1, class RT2, class RT3, class Policy>
 203:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 204:          ibetac_inv(RT1 a, RT2 b, RT3 q, const Policy&); // Incomplete beta complement inverse function.
 205: 
 206:    template <class RT1, class RT2, class RT3>
 207:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 208:          ibetac_inva(RT1 a, RT2 b, RT3 q); // Incomplete beta complement inverse function.
 209: 
 210:    template <class RT1, class RT2, class RT3, class Policy>
 211:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 212:          ibetac_inva(RT1 a, RT2 b, RT3 q, const Policy&); // Incomplete beta complement inverse function.
 213: 
 214:    template <class RT1, class RT2, class RT3>
 215:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 216:          ibetac_invb(RT1 a, RT2 b, RT3 q); // Incomplete beta complement inverse function.
 217: 
 218:    template <class RT1, class RT2, class RT3, class Policy>
 219:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 220:          ibetac_invb(RT1 a, RT2 b, RT3 q, const Policy&); // Incomplete beta complement inverse function.
 221: 
 222:    template <class RT1, class RT2, class RT3>
 223:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 224:          ibeta_derivative(RT1 a, RT2 b, RT3 x);  // derivative of incomplete beta
````
- **L197 EN**: Blank line separating nearby declarations or logic.
  - **L197 CN**: 空行，用于分隔相邻声明或逻辑。
- **L198 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L198 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L199 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L199 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L200 EN**: Continues logic associated with callable symbol `ibetac_inv`.
  - **L200 CN**: 继续与可调用符号 `ibetac_inv` 相关的逻辑。
- **L201 EN**: Blank line separating nearby declarations or logic.
  - **L201 CN**: 空行，用于分隔相邻声明或逻辑。
- **L202 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L202 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L203 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L203 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L204 EN**: Continues logic associated with callable symbol `ibetac_inv`.
  - **L204 CN**: 继续与可调用符号 `ibetac_inv` 相关的逻辑。
- **L205 EN**: Blank line separating nearby declarations or logic.
  - **L205 CN**: 空行，用于分隔相邻声明或逻辑。
- **L206 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L206 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L207 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L207 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L208 EN**: Continues logic associated with callable symbol `ibetac_inva`.
  - **L208 CN**: 继续与可调用符号 `ibetac_inva` 相关的逻辑。
- **L209 EN**: Blank line separating nearby declarations or logic.
  - **L209 CN**: 空行，用于分隔相邻声明或逻辑。
- **L210 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L210 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L211 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L211 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L212 EN**: Continues logic associated with callable symbol `ibetac_inva`.
  - **L212 CN**: 继续与可调用符号 `ibetac_inva` 相关的逻辑。
- **L213 EN**: Blank line separating nearby declarations or logic.
  - **L213 CN**: 空行，用于分隔相邻声明或逻辑。
- **L214 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L214 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L215 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L215 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L216 EN**: Continues logic associated with callable symbol `ibetac_invb`.
  - **L216 CN**: 继续与可调用符号 `ibetac_invb` 相关的逻辑。
- **L217 EN**: Blank line separating nearby declarations or logic.
  - **L217 CN**: 空行，用于分隔相邻声明或逻辑。
- **L218 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L218 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L219 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L219 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L220 EN**: Continues logic associated with callable symbol `ibetac_invb`.
  - **L220 CN**: 继续与可调用符号 `ibetac_invb` 相关的逻辑。
- **L221 EN**: Blank line separating nearby declarations or logic.
  - **L221 CN**: 空行，用于分隔相邻声明或逻辑。
- **L222 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>`.
  - **L222 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>`。
- **L223 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L223 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L224 EN**: Continues logic associated with callable symbol `ibeta_derivative`.
  - **L224 CN**: 继续与可调用符号 `ibeta_derivative` 相关的逻辑。

### Lines 225-252 / 第 225-252 行

````cpp
 225: 
 226:    template <class RT1, class RT2, class RT3, class Policy>
 227:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2, RT3>
 228:          ibeta_derivative(RT1 a, RT2 b, RT3 x, const Policy& pol);  // derivative of incomplete beta
 229: 
 230:    // Binomial:
 231:    template <class T, class Policy>
 232:    BOOST_MATH_GPU_ENABLED T binomial_coefficient(unsigned n, unsigned k, const Policy& pol);
 233:    template <class T>
 234:    BOOST_MATH_GPU_ENABLED T binomial_coefficient(unsigned n, unsigned k);
 235: 
 236:    // erf & erfc error functions.
 237:    template <class RT> // Error function.
 238:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erf(RT z);
 239:    template <class RT, class Policy> // Error function.
 240:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erf(RT z, const Policy&);
 241: 
 242:    template <class RT>// Error function complement.
 243:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erfc(RT z);
 244:    template <class RT, class Policy>// Error function complement.
 245:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erfc(RT z, const Policy&);
 246: 
 247:    template <class RT>// Error function inverse.
 248:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erf_inv(RT z);
 249:    template <class RT, class Policy>// Error function inverse.
 250:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erf_inv(RT z, const Policy& pol);
 251: 
 252:    template <class RT>// Error function complement inverse.
````
- **L225 EN**: Blank line separating nearby declarations or logic.
  - **L225 CN**: 空行，用于分隔相邻声明或逻辑。
- **L226 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3, class Policy>`.
  - **L226 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3, class Policy>`。
- **L227 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L227 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L228 EN**: Continues logic associated with callable symbol `ibeta_derivative`.
  - **L228 CN**: 继续与可调用符号 `ibeta_derivative` 相关的逻辑。
- **L229 EN**: Blank line separating nearby declarations or logic.
  - **L229 CN**: 空行，用于分隔相邻声明或逻辑。
- **L230 EN**: Comment documents nearby intent or usage notes: `Binomial:`.
  - **L230 CN**: 注释说明附近代码的意图或使用说明：`Binomial:`。
- **L231 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L231 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L232 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L232 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L233 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L233 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L234 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L234 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L235 EN**: Blank line separating nearby declarations or logic.
  - **L235 CN**: 空行，用于分隔相邻声明或逻辑。
- **L236 EN**: Comment documents nearby intent or usage notes: `erf & erfc error functions.`.
  - **L236 CN**: 注释说明附近代码的意图或使用说明：`erf & erfc error functions.`。
- **L237 EN**: Introduces template parameters or specialization context: `template <class RT> // Error function.`.
  - **L237 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT> // Error function.`。
- **L238 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L238 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L239 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy> // Error function.`.
  - **L239 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy> // Error function.`。
- **L240 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L240 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L241 EN**: Blank line separating nearby declarations or logic.
  - **L241 CN**: 空行，用于分隔相邻声明或逻辑。
- **L242 EN**: Introduces template parameters or specialization context: `template <class RT>// Error function complement.`.
  - **L242 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>// Error function complement.`。
- **L243 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L243 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L244 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>// Error function complement.`.
  - **L244 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>// Error function complement.`。
- **L245 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L245 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L246 EN**: Blank line separating nearby declarations or logic.
  - **L246 CN**: 空行，用于分隔相邻声明或逻辑。
- **L247 EN**: Introduces template parameters or specialization context: `template <class RT>// Error function inverse.`.
  - **L247 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>// Error function inverse.`。
- **L248 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L248 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L249 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>// Error function inverse.`.
  - **L249 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>// Error function inverse.`。
- **L250 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L250 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L251 EN**: Blank line separating nearby declarations or logic.
  - **L251 CN**: 空行，用于分隔相邻声明或逻辑。
- **L252 EN**: Introduces template parameters or specialization context: `template <class RT>// Error function complement inverse.`.
  - **L252 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>// Error function complement inverse.`。

### Lines 253-280 / 第 253-280 行

````cpp
 253:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erfc_inv(RT z);
 254:    template <class RT, class Policy>// Error function complement inverse.
 255:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> erfc_inv(RT z, const Policy& pol);
 256: 
 257:    // Polynomials:
 258:    template <class T1, class T2, class T3>
 259:    tools::promote_args_t<T1, T2, T3>
 260:          legendre_next(unsigned l, T1 x, T2 Pl, T3 Plm1);
 261: 
 262:    template <class T>
 263:    tools::promote_args_t<T>
 264:          legendre_p(int l, T x);
 265:    template <class T>
 266:    tools::promote_args_t<T>
 267:           legendre_p_prime(int l, T x);
 268: 
 269: 
 270:    template <class T, class Policy>
 271:    inline std::vector<T> legendre_p_zeros(int l, const Policy& pol);
 272: 
 273:    template <class T>
 274:    inline std::vector<T> legendre_p_zeros(int l);
 275: 
 276:    template <class T, class Policy>
 277:    typename std::enable_if<policies::is_policy<Policy>::value, tools::promote_args_t<T>>::type
 278:          legendre_p(int l, T x, const Policy& pol);
 279:    template <class T, class Policy>
 280:    inline typename std::enable_if<policies::is_policy<Policy>::value, tools::promote_args_t<T>>::type
````
- **L253 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L253 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L254 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>// Error function complement inverse.`.
  - **L254 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>// Error function complement inverse.`。
- **L255 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L255 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L256 EN**: Blank line separating nearby declarations or logic.
  - **L256 CN**: 空行，用于分隔相邻声明或逻辑。
- **L257 EN**: Comment documents nearby intent or usage notes: `Polynomials:`.
  - **L257 CN**: 注释说明附近代码的意图或使用说明：`Polynomials:`。
- **L258 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L258 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L259 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2, T3>`.
  - **L259 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2, T3>`。
- **L260 EN**: Executes a call or declaration centered on `legendre_next`.
  - **L260 CN**: 执行以 `legendre_next` 为核心的调用或声明。
- **L261 EN**: Blank line separating nearby declarations or logic.
  - **L261 CN**: 空行，用于分隔相邻声明或逻辑。
- **L262 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L262 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L263 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T>`.
  - **L263 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T>`。
- **L264 EN**: Executes a call or declaration centered on `legendre_p`.
  - **L264 CN**: 执行以 `legendre_p` 为核心的调用或声明。
- **L265 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L265 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L266 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T>`.
  - **L266 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T>`。
- **L267 EN**: Executes a call or declaration centered on `legendre_p_prime`.
  - **L267 CN**: 执行以 `legendre_p_prime` 为核心的调用或声明。
- **L268 EN**: Blank line separating nearby declarations or logic.
  - **L268 CN**: 空行，用于分隔相邻声明或逻辑。
- **L269 EN**: Blank line separating nearby declarations or logic.
  - **L269 CN**: 空行，用于分隔相邻声明或逻辑。
- **L270 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L270 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L271 EN**: Executes a call or declaration centered on `legendre_p_zeros`.
  - **L271 CN**: 执行以 `legendre_p_zeros` 为核心的调用或声明。
- **L272 EN**: Blank line separating nearby declarations or logic.
  - **L272 CN**: 空行，用于分隔相邻声明或逻辑。
- **L273 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L273 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L274 EN**: Executes a call or declaration centered on `legendre_p_zeros`.
  - **L274 CN**: 执行以 `legendre_p_zeros` 为核心的调用或声明。
- **L275 EN**: Blank line separating nearby declarations or logic.
  - **L275 CN**: 空行，用于分隔相邻声明或逻辑。
- **L276 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L276 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L277 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L277 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L278 EN**: Executes a call or declaration centered on `legendre_p`.
  - **L278 CN**: 执行以 `legendre_p` 为核心的调用或声明。
- **L279 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L279 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L280 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L280 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。

### Lines 281-308 / 第 281-308 行

````cpp
 281:       legendre_p_prime(int l, T x, const Policy& pol);
 282: 
 283:    template <class T>
 284:    tools::promote_args_t<T>
 285:          legendre_q(unsigned l, T x);
 286: 
 287:    template <class T, class Policy>
 288:    typename std::enable_if<policies::is_policy<Policy>::value, tools::promote_args_t<T>>::type
 289:          legendre_q(unsigned l, T x, const Policy& pol);
 290: 
 291:    template <class T1, class T2, class T3>
 292:    tools::promote_args_t<T1, T2, T3>
 293:          legendre_next(unsigned l, unsigned m, T1 x, T2 Pl, T3 Plm1);
 294: 
 295:    template <class T>
 296:    tools::promote_args_t<T>
 297:          legendre_p(int l, int m, T x);
 298: 
 299:    template <class T, class Policy>
 300:    tools::promote_args_t<T>
 301:          legendre_p(int l, int m, T x, const Policy& pol);
 302: 
 303:    template <class T1, class T2, class T3>
 304:    tools::promote_args_t<T1, T2, T3>
 305:          laguerre_next(unsigned n, T1 x, T2 Ln, T3 Lnm1);
 306: 
 307:    template <class T1, class T2, class T3>
 308:    tools::promote_args_t<T1, T2, T3>
````
- **L281 EN**: Executes a call or declaration centered on `legendre_p_prime`.
  - **L281 CN**: 执行以 `legendre_p_prime` 为核心的调用或声明。
- **L282 EN**: Blank line separating nearby declarations or logic.
  - **L282 CN**: 空行，用于分隔相邻声明或逻辑。
- **L283 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L283 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L284 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T>`.
  - **L284 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T>`。
- **L285 EN**: Executes a call or declaration centered on `legendre_q`.
  - **L285 CN**: 执行以 `legendre_q` 为核心的调用或声明。
- **L286 EN**: Blank line separating nearby declarations or logic.
  - **L286 CN**: 空行，用于分隔相邻声明或逻辑。
- **L287 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L287 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L288 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L288 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L289 EN**: Executes a call or declaration centered on `legendre_q`.
  - **L289 CN**: 执行以 `legendre_q` 为核心的调用或声明。
- **L290 EN**: Blank line separating nearby declarations or logic.
  - **L290 CN**: 空行，用于分隔相邻声明或逻辑。
- **L291 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L291 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L292 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2, T3>`.
  - **L292 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2, T3>`。
- **L293 EN**: Executes a call or declaration centered on `legendre_next`.
  - **L293 CN**: 执行以 `legendre_next` 为核心的调用或声明。
- **L294 EN**: Blank line separating nearby declarations or logic.
  - **L294 CN**: 空行，用于分隔相邻声明或逻辑。
- **L295 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L295 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L296 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T>`.
  - **L296 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T>`。
- **L297 EN**: Executes a call or declaration centered on `legendre_p`.
  - **L297 CN**: 执行以 `legendre_p` 为核心的调用或声明。
- **L298 EN**: Blank line separating nearby declarations or logic.
  - **L298 CN**: 空行，用于分隔相邻声明或逻辑。
- **L299 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L299 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L300 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T>`.
  - **L300 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T>`。
- **L301 EN**: Executes a call or declaration centered on `legendre_p`.
  - **L301 CN**: 执行以 `legendre_p` 为核心的调用或声明。
- **L302 EN**: Blank line separating nearby declarations or logic.
  - **L302 CN**: 空行，用于分隔相邻声明或逻辑。
- **L303 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L303 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L304 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2, T3>`.
  - **L304 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2, T3>`。
- **L305 EN**: Executes a call or declaration centered on `laguerre_next`.
  - **L305 CN**: 执行以 `laguerre_next` 为核心的调用或声明。
- **L306 EN**: Blank line separating nearby declarations or logic.
  - **L306 CN**: 空行，用于分隔相邻声明或逻辑。
- **L307 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L307 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L308 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2, T3>`.
  - **L308 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2, T3>`。

### Lines 309-336 / 第 309-336 行

````cpp
 309:       laguerre_next(unsigned n, unsigned l, T1 x, T2 Pl, T3 Plm1);
 310: 
 311:    template <class T>
 312:    tools::promote_args_t<T>
 313:       laguerre(unsigned n, T x);
 314: 
 315:    template <class T, class Policy>
 316:    tools::promote_args_t<T>
 317:       laguerre(unsigned n, unsigned m, T x, const Policy& pol);
 318: 
 319:    template <class T1, class T2>
 320:    struct laguerre_result
 321:    {
 322:       using type = typename std::conditional<
 323:          policies::is_policy<T2>::value,
 324:          typename tools::promote_args<T1>::type,
 325:          typename tools::promote_args<T2>::type
 326:       >::type;
 327:    };
 328: 
 329:    template <class T1, class T2>
 330:    typename laguerre_result<T1, T2>::type
 331:       laguerre(unsigned n, T1 m, T2 x);
 332: 
 333:    template <class T>
 334:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T>
 335:       hermite(unsigned n, T x);
 336: 
````
- **L309 EN**: Executes a call or declaration centered on `laguerre_next`.
  - **L309 CN**: 执行以 `laguerre_next` 为核心的调用或声明。
- **L310 EN**: Blank line separating nearby declarations or logic.
  - **L310 CN**: 空行，用于分隔相邻声明或逻辑。
- **L311 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L311 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L312 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T>`.
  - **L312 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T>`。
- **L313 EN**: Executes a call or declaration centered on `laguerre`.
  - **L313 CN**: 执行以 `laguerre` 为核心的调用或声明。
- **L314 EN**: Blank line separating nearby declarations or logic.
  - **L314 CN**: 空行，用于分隔相邻声明或逻辑。
- **L315 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L315 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L316 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T>`.
  - **L316 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T>`。
- **L317 EN**: Executes a call or declaration centered on `laguerre`.
  - **L317 CN**: 执行以 `laguerre` 为核心的调用或声明。
- **L318 EN**: Blank line separating nearby declarations or logic.
  - **L318 CN**: 空行，用于分隔相邻声明或逻辑。
- **L319 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L319 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L320 EN**: Declares struct `laguerre_result`.
  - **L320 CN**: 声明 struct `laguerre_result`。
- **L321 EN**: Opens a new lexical scope or compound statement.
  - **L321 CN**: 打开一个新的词法作用域或复合语句块。
- **L322 EN**: Defines alias `type` to simplify later code.
  - **L322 CN**: 定义别名 `type` 以简化后续代码。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::is_policy<T2>::value,`.
  - **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::is_policy<T2>::value,`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `typename tools::promote_args<T1>::type,`.
  - **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`typename tools::promote_args<T1>::type,`。
- **L325 EN**: Continues the surrounding expression or declaration: `typename tools::promote_args<T2>::type`.
  - **L325 CN**: 继续构造周围的表达式或声明：`typename tools::promote_args<T2>::type`。
- **L326 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L326 CN**: 执行一条独立语句或声明：`>::type;`。
- **L327 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L327 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L328 EN**: Blank line separating nearby declarations or logic.
  - **L328 CN**: 空行，用于分隔相邻声明或逻辑。
- **L329 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L329 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L330 EN**: Continues the surrounding expression or declaration: `typename laguerre_result<T1, T2>::type`.
  - **L330 CN**: 继续构造周围的表达式或声明：`typename laguerre_result<T1, T2>::type`。
- **L331 EN**: Executes a call or declaration centered on `laguerre`.
  - **L331 CN**: 执行以 `laguerre` 为核心的调用或声明。
- **L332 EN**: Blank line separating nearby declarations or logic.
  - **L332 CN**: 空行，用于分隔相邻声明或逻辑。
- **L333 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L333 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L334 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L334 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L335 EN**: Executes a call or declaration centered on `hermite`.
  - **L335 CN**: 执行以 `hermite` 为核心的调用或声明。
- **L336 EN**: Blank line separating nearby declarations or logic.
  - **L336 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 337-364 / 第 337-364 行

````cpp
 337:    template <class T, class Policy>
 338:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T>
 339:       hermite(unsigned n, T x, const Policy& pol);
 340: 
 341:    template <class T1, class T2, class T3>
 342:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3>
 343:       hermite_next(unsigned n, T1 x, T2 Hn, T3 Hnm1);
 344: 
 345:    template<class T1, class T2, class T3>
 346:    tools::promote_args_t<T1, T2, T3> chebyshev_next(T1 const & x, T2 const & Tn, T3 const & Tn_1);
 347: 
 348:    template <class Real, class Policy>
 349:    tools::promote_args_t<Real>
 350:       chebyshev_t(unsigned n, Real const & x, const Policy&);
 351:    template<class Real>
 352:    tools::promote_args_t<Real> chebyshev_t(unsigned n, Real const & x);
 353:    
 354:    template <class Real, class Policy>
 355:    tools::promote_args_t<Real>
 356:       chebyshev_u(unsigned n, Real const & x, const Policy&);
 357:    template<class Real>
 358:    tools::promote_args_t<Real> chebyshev_u(unsigned n, Real const & x);
 359: 
 360:    template <class Real, class Policy>
 361:    tools::promote_args_t<Real>
 362:       chebyshev_t_prime(unsigned n, Real const & x, const Policy&);
 363:    template<class Real>
 364:    tools::promote_args_t<Real> chebyshev_t_prime(unsigned n, Real const & x);
````
- **L337 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L337 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L338 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L338 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L339 EN**: Executes a call or declaration centered on `hermite`.
  - **L339 CN**: 执行以 `hermite` 为核心的调用或声明。
- **L340 EN**: Blank line separating nearby declarations or logic.
  - **L340 CN**: 空行，用于分隔相邻声明或逻辑。
- **L341 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L341 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L342 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L342 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L343 EN**: Executes a call or declaration centered on `hermite_next`.
  - **L343 CN**: 执行以 `hermite_next` 为核心的调用或声明。
- **L344 EN**: Blank line separating nearby declarations or logic.
  - **L344 CN**: 空行，用于分隔相邻声明或逻辑。
- **L345 EN**: Introduces template parameters or specialization context: `template<class T1, class T2, class T3>`.
  - **L345 CN**: 为后续声明引入模板参数或特化上下文：`template<class T1, class T2, class T3>`。
- **L346 EN**: Executes a call or declaration centered on `chebyshev_next`.
  - **L346 CN**: 执行以 `chebyshev_next` 为核心的调用或声明。
- **L347 EN**: Blank line separating nearby declarations or logic.
  - **L347 CN**: 空行，用于分隔相邻声明或逻辑。
- **L348 EN**: Introduces template parameters or specialization context: `template <class Real, class Policy>`.
  - **L348 CN**: 为后续声明引入模板参数或特化上下文：`template <class Real, class Policy>`。
- **L349 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<Real>`.
  - **L349 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<Real>`。
- **L350 EN**: Executes a call or declaration centered on `chebyshev_t`.
  - **L350 CN**: 执行以 `chebyshev_t` 为核心的调用或声明。
- **L351 EN**: Introduces template parameters or specialization context: `template<class Real>`.
  - **L351 CN**: 为后续声明引入模板参数或特化上下文：`template<class Real>`。
- **L352 EN**: Executes a call or declaration centered on `chebyshev_t`.
  - **L352 CN**: 执行以 `chebyshev_t` 为核心的调用或声明。
- **L353 EN**: Blank line separating nearby declarations or logic.
  - **L353 CN**: 空行，用于分隔相邻声明或逻辑。
- **L354 EN**: Introduces template parameters or specialization context: `template <class Real, class Policy>`.
  - **L354 CN**: 为后续声明引入模板参数或特化上下文：`template <class Real, class Policy>`。
- **L355 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<Real>`.
  - **L355 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<Real>`。
- **L356 EN**: Executes a call or declaration centered on `chebyshev_u`.
  - **L356 CN**: 执行以 `chebyshev_u` 为核心的调用或声明。
- **L357 EN**: Introduces template parameters or specialization context: `template<class Real>`.
  - **L357 CN**: 为后续声明引入模板参数或特化上下文：`template<class Real>`。
- **L358 EN**: Executes a call or declaration centered on `chebyshev_u`.
  - **L358 CN**: 执行以 `chebyshev_u` 为核心的调用或声明。
- **L359 EN**: Blank line separating nearby declarations or logic.
  - **L359 CN**: 空行，用于分隔相邻声明或逻辑。
- **L360 EN**: Introduces template parameters or specialization context: `template <class Real, class Policy>`.
  - **L360 CN**: 为后续声明引入模板参数或特化上下文：`template <class Real, class Policy>`。
- **L361 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<Real>`.
  - **L361 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<Real>`。
- **L362 EN**: Executes a call or declaration centered on `chebyshev_t_prime`.
  - **L362 CN**: 执行以 `chebyshev_t_prime` 为核心的调用或声明。
- **L363 EN**: Introduces template parameters or specialization context: `template<class Real>`.
  - **L363 CN**: 为后续声明引入模板参数或特化上下文：`template<class Real>`。
- **L364 EN**: Executes a call or declaration centered on `chebyshev_t_prime`.
  - **L364 CN**: 执行以 `chebyshev_t_prime` 为核心的调用或声明。

### Lines 365-392 / 第 365-392 行

````cpp
 365: 
 366:    template<class Real, class T2>
 367:    Real chebyshev_clenshaw_recurrence(const Real* const c, size_t length, const T2& x);
 368: 
 369:    template <class T1, class T2>
 370:    std::complex<tools::promote_args_t<T1, T2>>
 371:          spherical_harmonic(unsigned n, int m, T1 theta, T2 phi);
 372: 
 373:    template <class T1, class T2, class Policy>
 374:    std::complex<tools::promote_args_t<T1, T2>>
 375:       spherical_harmonic(unsigned n, int m, T1 theta, T2 phi, const Policy& pol);
 376: 
 377:    template <class T1, class T2>
 378:    tools::promote_args_t<T1, T2>
 379:          spherical_harmonic_r(unsigned n, int m, T1 theta, T2 phi);
 380: 
 381:    template <class T1, class T2, class Policy>
 382:    tools::promote_args_t<T1, T2>
 383:       spherical_harmonic_r(unsigned n, int m, T1 theta, T2 phi, const Policy& pol);
 384: 
 385:    template <class T1, class T2>
 386:    tools::promote_args_t<T1, T2>
 387:          spherical_harmonic_i(unsigned n, int m, T1 theta, T2 phi);
 388: 
 389:    template <class T1, class T2, class Policy>
 390:    tools::promote_args_t<T1, T2>
 391:       spherical_harmonic_i(unsigned n, int m, T1 theta, T2 phi, const Policy& pol);
 392: 
````
- **L365 EN**: Blank line separating nearby declarations or logic.
  - **L365 CN**: 空行，用于分隔相邻声明或逻辑。
- **L366 EN**: Introduces template parameters or specialization context: `template<class Real, class T2>`.
  - **L366 CN**: 为后续声明引入模板参数或特化上下文：`template<class Real, class T2>`。
- **L367 EN**: Executes a call or declaration centered on `chebyshev_clenshaw_recurrence`.
  - **L367 CN**: 执行以 `chebyshev_clenshaw_recurrence` 为核心的调用或声明。
- **L368 EN**: Blank line separating nearby declarations or logic.
  - **L368 CN**: 空行，用于分隔相邻声明或逻辑。
- **L369 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L369 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L370 EN**: Continues the surrounding expression or declaration: `std::complex<tools::promote_args_t<T1, T2>>`.
  - **L370 CN**: 继续构造周围的表达式或声明：`std::complex<tools::promote_args_t<T1, T2>>`。
- **L371 EN**: Executes a call or declaration centered on `spherical_harmonic`.
  - **L371 CN**: 执行以 `spherical_harmonic` 为核心的调用或声明。
- **L372 EN**: Blank line separating nearby declarations or logic.
  - **L372 CN**: 空行，用于分隔相邻声明或逻辑。
- **L373 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L373 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L374 EN**: Continues the surrounding expression or declaration: `std::complex<tools::promote_args_t<T1, T2>>`.
  - **L374 CN**: 继续构造周围的表达式或声明：`std::complex<tools::promote_args_t<T1, T2>>`。
- **L375 EN**: Executes a call or declaration centered on `spherical_harmonic`.
  - **L375 CN**: 执行以 `spherical_harmonic` 为核心的调用或声明。
- **L376 EN**: Blank line separating nearby declarations or logic.
  - **L376 CN**: 空行，用于分隔相邻声明或逻辑。
- **L377 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L377 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L378 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2>`.
  - **L378 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2>`。
- **L379 EN**: Executes a call or declaration centered on `spherical_harmonic_r`.
  - **L379 CN**: 执行以 `spherical_harmonic_r` 为核心的调用或声明。
- **L380 EN**: Blank line separating nearby declarations or logic.
  - **L380 CN**: 空行，用于分隔相邻声明或逻辑。
- **L381 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L381 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L382 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2>`.
  - **L382 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2>`。
- **L383 EN**: Executes a call or declaration centered on `spherical_harmonic_r`.
  - **L383 CN**: 执行以 `spherical_harmonic_r` 为核心的调用或声明。
- **L384 EN**: Blank line separating nearby declarations or logic.
  - **L384 CN**: 空行，用于分隔相邻声明或逻辑。
- **L385 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L385 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L386 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2>`.
  - **L386 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2>`。
- **L387 EN**: Executes a call or declaration centered on `spherical_harmonic_i`.
  - **L387 CN**: 执行以 `spherical_harmonic_i` 为核心的调用或声明。
- **L388 EN**: Blank line separating nearby declarations or logic.
  - **L388 CN**: 空行，用于分隔相邻声明或逻辑。
- **L389 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L389 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L390 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2>`.
  - **L390 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2>`。
- **L391 EN**: Executes a call or declaration centered on `spherical_harmonic_i`.
  - **L391 CN**: 执行以 `spherical_harmonic_i` 为核心的调用或声明。
- **L392 EN**: Blank line separating nearby declarations or logic.
  - **L392 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 393-420 / 第 393-420 行

````cpp
 393:    // Elliptic integrals:
 394:    template <class T1, class T2, class T3>
 395:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3>
 396:          ellint_rf(T1 x, T2 y, T3 z);
 397: 
 398:    template <class T1, class T2, class T3, class Policy>
 399:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3>
 400:          ellint_rf(T1 x, T2 y, T3 z, const Policy& pol);
 401: 
 402:    template <class T1, class T2, class T3>
 403:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3>
 404:          ellint_rd(T1 x, T2 y, T3 z);
 405: 
 406:    template <class T1, class T2, class T3, class Policy>
 407:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3>
 408:          ellint_rd(T1 x, T2 y, T3 z, const Policy& pol);
 409: 
 410:    template <class T1, class T2>
 411:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2>
 412:          ellint_rc(T1 x, T2 y);
 413: 
 414:    template <class T1, class T2, class Policy>
 415:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2>
 416:          ellint_rc(T1 x, T2 y, const Policy& pol);
 417: 
 418:    template <class T1, class T2, class T3, class T4>
 419:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3, T4>
 420:          ellint_rj(T1 x, T2 y, T3 z, T4 p);
````
- **L393 EN**: Comment documents nearby intent or usage notes: `Elliptic integrals:`.
  - **L393 CN**: 注释说明附近代码的意图或使用说明：`Elliptic integrals:`。
- **L394 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L394 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L395 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L395 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L396 EN**: Executes a call or declaration centered on `ellint_rf`.
  - **L396 CN**: 执行以 `ellint_rf` 为核心的调用或声明。
- **L397 EN**: Blank line separating nearby declarations or logic.
  - **L397 CN**: 空行，用于分隔相邻声明或逻辑。
- **L398 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class Policy>`.
  - **L398 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class Policy>`。
- **L399 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L399 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L400 EN**: Executes a call or declaration centered on `ellint_rf`.
  - **L400 CN**: 执行以 `ellint_rf` 为核心的调用或声明。
- **L401 EN**: Blank line separating nearby declarations or logic.
  - **L401 CN**: 空行，用于分隔相邻声明或逻辑。
- **L402 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L402 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L403 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L403 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L404 EN**: Executes a call or declaration centered on `ellint_rd`.
  - **L404 CN**: 执行以 `ellint_rd` 为核心的调用或声明。
- **L405 EN**: Blank line separating nearby declarations or logic.
  - **L405 CN**: 空行，用于分隔相邻声明或逻辑。
- **L406 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class Policy>`.
  - **L406 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class Policy>`。
- **L407 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L407 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L408 EN**: Executes a call or declaration centered on `ellint_rd`.
  - **L408 CN**: 执行以 `ellint_rd` 为核心的调用或声明。
- **L409 EN**: Blank line separating nearby declarations or logic.
  - **L409 CN**: 空行，用于分隔相邻声明或逻辑。
- **L410 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L410 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L411 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L411 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L412 EN**: Executes a call or declaration centered on `ellint_rc`.
  - **L412 CN**: 执行以 `ellint_rc` 为核心的调用或声明。
- **L413 EN**: Blank line separating nearby declarations or logic.
  - **L413 CN**: 空行，用于分隔相邻声明或逻辑。
- **L414 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L414 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L415 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L415 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L416 EN**: Executes a call or declaration centered on `ellint_rc`.
  - **L416 CN**: 执行以 `ellint_rc` 为核心的调用或声明。
- **L417 EN**: Blank line separating nearby declarations or logic.
  - **L417 CN**: 空行，用于分隔相邻声明或逻辑。
- **L418 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4>`.
  - **L418 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4>`。
- **L419 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L419 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L420 EN**: Executes a call or declaration centered on `ellint_rj`.
  - **L420 CN**: 执行以 `ellint_rj` 为核心的调用或声明。

### Lines 421-448 / 第 421-448 行

````cpp
 421: 
 422:    template <class T1, class T2, class T3, class T4, class Policy>
 423:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3, T4>
 424:          ellint_rj(T1 x, T2 y, T3 z, T4 p, const Policy& pol);
 425: 
 426:    template <class T1, class T2, class T3>
 427:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3>
 428:       ellint_rg(T1 x, T2 y, T3 z);
 429: 
 430:    template <class T1, class T2, class T3, class Policy>
 431:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3>
 432:       ellint_rg(T1 x, T2 y, T3 z, const Policy& pol);
 433: 
 434:    template <typename T>
 435:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> ellint_2(T k);
 436: 
 437:    template <class T1, class T2>
 438:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> ellint_2(T1 k, T2 phi);
 439: 
 440:    template <class T1, class T2, class Policy>
 441:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> ellint_2(T1 k, T2 phi, const Policy& pol);
 442: 
 443:    template <typename T>
 444:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> ellint_1(T k);
 445: 
 446:    template <class T1, class T2>
 447:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> ellint_1(T1 k, T2 phi);
 448: 
````
- **L421 EN**: Blank line separating nearby declarations or logic.
  - **L421 CN**: 空行，用于分隔相邻声明或逻辑。
- **L422 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4, class Policy>`.
  - **L422 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4, class Policy>`。
- **L423 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L423 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L424 EN**: Executes a call or declaration centered on `ellint_rj`.
  - **L424 CN**: 执行以 `ellint_rj` 为核心的调用或声明。
- **L425 EN**: Blank line separating nearby declarations or logic.
  - **L425 CN**: 空行，用于分隔相邻声明或逻辑。
- **L426 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L426 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L427 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L427 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L428 EN**: Executes a call or declaration centered on `ellint_rg`.
  - **L428 CN**: 执行以 `ellint_rg` 为核心的调用或声明。
- **L429 EN**: Blank line separating nearby declarations or logic.
  - **L429 CN**: 空行，用于分隔相邻声明或逻辑。
- **L430 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class Policy>`.
  - **L430 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class Policy>`。
- **L431 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L431 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L432 EN**: Executes a call or declaration centered on `ellint_rg`.
  - **L432 CN**: 执行以 `ellint_rg` 为核心的调用或声明。
- **L433 EN**: Blank line separating nearby declarations or logic.
  - **L433 CN**: 空行，用于分隔相邻声明或逻辑。
- **L434 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  - **L434 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L435 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L435 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L436 EN**: Blank line separating nearby declarations or logic.
  - **L436 CN**: 空行，用于分隔相邻声明或逻辑。
- **L437 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L437 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L438 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L438 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L439 EN**: Blank line separating nearby declarations or logic.
  - **L439 CN**: 空行，用于分隔相邻声明或逻辑。
- **L440 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L440 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L441 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L441 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L442 EN**: Blank line separating nearby declarations or logic.
  - **L442 CN**: 空行，用于分隔相邻声明或逻辑。
- **L443 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  - **L443 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L444 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L444 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L445 EN**: Blank line separating nearby declarations or logic.
  - **L445 CN**: 空行，用于分隔相邻声明或逻辑。
- **L446 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L446 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L447 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L447 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L448 EN**: Blank line separating nearby declarations or logic.
  - **L448 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 449-476 / 第 449-476 行

````cpp
 449:    template <class T1, class T2, class Policy>
 450:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> ellint_1(T1 k, T2 phi, const Policy& pol);
 451: 
 452:    template <typename T>
 453:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> ellint_d(T k);
 454: 
 455:    template <class T1, class T2>
 456:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> ellint_d(T1 k, T2 phi);
 457: 
 458:    template <class T1, class T2, class Policy>
 459:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> ellint_d(T1 k, T2 phi, const Policy& pol);
 460: 
 461:    template <class T1, class T2>
 462:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> jacobi_zeta(T1 k, T2 phi);
 463: 
 464:    template <class T1, class T2, class Policy>
 465:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> jacobi_zeta(T1 k, T2 phi, const Policy& pol);
 466: 
 467:    template <class T1, class T2>
 468:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> heuman_lambda(T1 k, T2 phi);
 469: 
 470:    template <class T1, class T2, class Policy>
 471:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> heuman_lambda(T1 k, T2 phi, const Policy& pol);
 472: 
 473:    namespace detail{
 474: 
 475:    template <class T, class U, class V>
 476:    struct ellint_3_result
````
- **L449 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L449 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L450 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L450 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L451 EN**: Blank line separating nearby declarations or logic.
  - **L451 CN**: 空行，用于分隔相邻声明或逻辑。
- **L452 EN**: Introduces template parameters or specialization context: `template <typename T>`.
  - **L452 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>`。
- **L453 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L453 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L454 EN**: Blank line separating nearby declarations or logic.
  - **L454 CN**: 空行，用于分隔相邻声明或逻辑。
- **L455 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L455 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L456 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L456 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L457 EN**: Blank line separating nearby declarations or logic.
  - **L457 CN**: 空行，用于分隔相邻声明或逻辑。
- **L458 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L458 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L459 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L459 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L460 EN**: Blank line separating nearby declarations or logic.
  - **L460 CN**: 空行，用于分隔相邻声明或逻辑。
- **L461 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L461 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L462 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L462 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L463 EN**: Blank line separating nearby declarations or logic.
  - **L463 CN**: 空行，用于分隔相邻声明或逻辑。
- **L464 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L464 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L465 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L465 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L466 EN**: Blank line separating nearby declarations or logic.
  - **L466 CN**: 空行，用于分隔相邻声明或逻辑。
- **L467 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L467 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L468 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L468 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L469 EN**: Blank line separating nearby declarations or logic.
  - **L469 CN**: 空行，用于分隔相邻声明或逻辑。
- **L470 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L470 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L471 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L471 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L472 EN**: Blank line separating nearby declarations or logic.
  - **L472 CN**: 空行，用于分隔相邻声明或逻辑。
- **L473 EN**: Opens namespace scope `detail`.
  - **L473 CN**: 打开命名空间作用域 `detail`。
- **L474 EN**: Blank line separating nearby declarations or logic.
  - **L474 CN**: 空行，用于分隔相邻声明或逻辑。
- **L475 EN**: Introduces template parameters or specialization context: `template <class T, class U, class V>`.
  - **L475 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class V>`。
- **L476 EN**: Declares struct `ellint_3_result`.
  - **L476 CN**: 声明 struct `ellint_3_result`。

### Lines 477-504 / 第 477-504 行

````cpp
 477:    {
 478:       using type = typename boost::math::conditional<
 479:          policies::is_policy<V>::value,
 480:          tools::promote_args_t<T, U>,
 481:          tools::promote_args_t<T, U, V>
 482:       >::type;
 483:    };
 484: 
 485:    } // namespace detail
 486: 
 487: 
 488:    template <class T1, class T2, class T3>
 489:    BOOST_MATH_GPU_ENABLED typename detail::ellint_3_result<T1, T2, T3>::type ellint_3(T1 k, T2 v, T3 phi);
 490: 
 491:    template <class T1, class T2, class T3, class Policy>
 492:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2, T3> ellint_3(T1 k, T2 v, T3 phi, const Policy& pol);
 493: 
 494:    template <class T1, class T2>
 495:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> ellint_3(T1 k, T2 v);
 496: 
 497:    // Factorial functions.
 498:    // Note: not for integral types, at present.
 499:    template <class RT>
 500:    struct max_factorial;
 501:    template <class RT>
 502:    BOOST_MATH_GPU_ENABLED RT factorial(unsigned int);
 503:    template <class RT, class Policy>
 504:    BOOST_MATH_GPU_ENABLED RT factorial(unsigned int, const Policy& pol);
````
- **L477 EN**: Opens a new lexical scope or compound statement.
  - **L477 CN**: 打开一个新的词法作用域或复合语句块。
- **L478 EN**: Defines alias `type` to simplify later code.
  - **L478 CN**: 定义别名 `type` 以简化后续代码。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::is_policy<V>::value,`.
  - **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::is_policy<V>::value,`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tools::promote_args_t<T, U>,`.
  - **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`tools::promote_args_t<T, U>,`。
- **L481 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T, U, V>`.
  - **L481 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T, U, V>`。
- **L482 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L482 CN**: 执行一条独立语句或声明：`>::type;`。
- **L483 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L483 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L484 EN**: Blank line separating nearby declarations or logic.
  - **L484 CN**: 空行，用于分隔相邻声明或逻辑。
- **L485 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L485 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。
- **L486 EN**: Blank line separating nearby declarations or logic.
  - **L486 CN**: 空行，用于分隔相邻声明或逻辑。
- **L487 EN**: Blank line separating nearby declarations or logic.
  - **L487 CN**: 空行，用于分隔相邻声明或逻辑。
- **L488 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>`.
  - **L488 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>`。
- **L489 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L489 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L490 EN**: Blank line separating nearby declarations or logic.
  - **L490 CN**: 空行，用于分隔相邻声明或逻辑。
- **L491 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class Policy>`.
  - **L491 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class Policy>`。
- **L492 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L492 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L493 EN**: Blank line separating nearby declarations or logic.
  - **L493 CN**: 空行，用于分隔相邻声明或逻辑。
- **L494 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L494 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L495 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L495 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L496 EN**: Blank line separating nearby declarations or logic.
  - **L496 CN**: 空行，用于分隔相邻声明或逻辑。
- **L497 EN**: Comment documents nearby intent or usage notes: `Factorial functions.`.
  - **L497 CN**: 注释说明附近代码的意图或使用说明：`Factorial functions.`。
- **L498 EN**: Comment documents nearby intent or usage notes: `Note: not for integral types, at present.`.
  - **L498 CN**: 注释说明附近代码的意图或使用说明：`Note: not for integral types, at present.`。
- **L499 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L499 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L500 EN**: Declares struct `max_factorial`.
  - **L500 CN**: 声明 struct `max_factorial`。
- **L501 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L501 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L502 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L502 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L503 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L503 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L504 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L504 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 505-532 / 第 505-532 行

````cpp
 505:    template <class RT>
 506:    BOOST_MATH_GPU_ENABLED RT unchecked_factorial(unsigned int BOOST_MATH_APPEND_EXPLICIT_TEMPLATE_TYPE(RT));
 507:    template <class RT>
 508:    BOOST_MATH_GPU_ENABLED RT double_factorial(unsigned i);
 509:    template <class RT, class Policy>
 510:    BOOST_MATH_GPU_ENABLED RT double_factorial(unsigned i, const Policy& pol);
 511: 
 512:    template <class RT>
 513:    tools::promote_args_t<RT> falling_factorial(RT x, unsigned n);
 514: 
 515:    template <class RT, class Policy>
 516:    tools::promote_args_t<RT> falling_factorial(RT x, unsigned n, const Policy& pol);
 517: 
 518:    template <class RT>
 519:    tools::promote_args_t<RT> rising_factorial(RT x, int n);
 520: 
 521:    template <class RT, class Policy>
 522:    tools::promote_args_t<RT> rising_factorial(RT x, int n, const Policy& pol);
 523: 
 524:    // Gamma functions.
 525:    template <class RT>
 526:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> tgamma(RT z);
 527: 
 528:    template <class RT>
 529:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> tgamma1pm1(RT z);
 530: 
 531:    template <class RT, class Policy>
 532:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> tgamma1pm1(RT z, const Policy& pol);
````
- **L505 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L505 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L506 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L506 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L507 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L507 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L508 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L508 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L509 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L509 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L510 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L510 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L511 EN**: Blank line separating nearby declarations or logic.
  - **L511 CN**: 空行，用于分隔相邻声明或逻辑。
- **L512 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L512 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L513 EN**: Executes a call or declaration centered on `falling_factorial`.
  - **L513 CN**: 执行以 `falling_factorial` 为核心的调用或声明。
- **L514 EN**: Blank line separating nearby declarations or logic.
  - **L514 CN**: 空行，用于分隔相邻声明或逻辑。
- **L515 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L515 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L516 EN**: Executes a call or declaration centered on `falling_factorial`.
  - **L516 CN**: 执行以 `falling_factorial` 为核心的调用或声明。
- **L517 EN**: Blank line separating nearby declarations or logic.
  - **L517 CN**: 空行，用于分隔相邻声明或逻辑。
- **L518 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L518 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L519 EN**: Executes a call or declaration centered on `rising_factorial`.
  - **L519 CN**: 执行以 `rising_factorial` 为核心的调用或声明。
- **L520 EN**: Blank line separating nearby declarations or logic.
  - **L520 CN**: 空行，用于分隔相邻声明或逻辑。
- **L521 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L521 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L522 EN**: Executes a call or declaration centered on `rising_factorial`.
  - **L522 CN**: 执行以 `rising_factorial` 为核心的调用或声明。
- **L523 EN**: Blank line separating nearby declarations or logic.
  - **L523 CN**: 空行，用于分隔相邻声明或逻辑。
- **L524 EN**: Comment documents nearby intent or usage notes: `Gamma functions.`.
  - **L524 CN**: 注释说明附近代码的意图或使用说明：`Gamma functions.`。
- **L525 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L525 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L526 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L526 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L527 EN**: Blank line separating nearby declarations or logic.
  - **L527 CN**: 空行，用于分隔相邻声明或逻辑。
- **L528 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L528 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L529 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L529 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L530 EN**: Blank line separating nearby declarations or logic.
  - **L530 CN**: 空行，用于分隔相邻声明或逻辑。
- **L531 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L531 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L532 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L532 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 533-560 / 第 533-560 行

````cpp
 533: 
 534:    template <class RT1, class RT2>
 535:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> tgamma(RT1 a, RT2 z);
 536: 
 537:    template <class RT1, class RT2, class Policy>
 538:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> tgamma(RT1 a, RT2 z, const Policy& pol);
 539: 
 540:    template <class RT>
 541:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> lgamma(RT z, int* sign);
 542: 
 543:    template <class RT, class Policy>
 544:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> lgamma(RT z, int* sign, const Policy& pol);
 545: 
 546:    template <class RT>
 547:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> lgamma(RT x);
 548: 
 549:    template <class RT, class Policy>
 550:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> lgamma(RT x, const Policy& pol);
 551: 
 552:    template <class RT1, class RT2>
 553:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> tgamma_lower(RT1 a, RT2 z);
 554: 
 555:    template <class RT1, class RT2, class Policy>
 556:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> tgamma_lower(RT1 a, RT2 z, const Policy&);
 557: 
 558:    template <class RT1, class RT2>
 559:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> gamma_q(RT1 a, RT2 z);
 560: 
````
- **L533 EN**: Blank line separating nearby declarations or logic.
  - **L533 CN**: 空行，用于分隔相邻声明或逻辑。
- **L534 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>`.
  - **L534 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>`。
- **L535 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L535 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L536 EN**: Blank line separating nearby declarations or logic.
  - **L536 CN**: 空行，用于分隔相邻声明或逻辑。
- **L537 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class Policy>`.
  - **L537 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class Policy>`。
- **L538 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L538 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L539 EN**: Blank line separating nearby declarations or logic.
  - **L539 CN**: 空行，用于分隔相邻声明或逻辑。
- **L540 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L540 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L541 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L541 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L542 EN**: Blank line separating nearby declarations or logic.
  - **L542 CN**: 空行，用于分隔相邻声明或逻辑。
- **L543 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L543 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L544 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L544 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L545 EN**: Blank line separating nearby declarations or logic.
  - **L545 CN**: 空行，用于分隔相邻声明或逻辑。
- **L546 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L546 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。
- **L547 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L547 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L548 EN**: Blank line separating nearby declarations or logic.
  - **L548 CN**: 空行，用于分隔相邻声明或逻辑。
- **L549 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L549 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L550 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L550 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L551 EN**: Blank line separating nearby declarations or logic.
  - **L551 CN**: 空行，用于分隔相邻声明或逻辑。
- **L552 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>`.
  - **L552 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>`。
- **L553 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L553 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L554 EN**: Blank line separating nearby declarations or logic.
  - **L554 CN**: 空行，用于分隔相邻声明或逻辑。
- **L555 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class Policy>`.
  - **L555 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class Policy>`。
- **L556 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L556 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L557 EN**: Blank line separating nearby declarations or logic.
  - **L557 CN**: 空行，用于分隔相邻声明或逻辑。
- **L558 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>`.
  - **L558 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>`。
- **L559 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L559 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L560 EN**: Blank line separating nearby declarations or logic.
  - **L560 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 561-588 / 第 561-588 行

````cpp
 561:    template <class RT1, class RT2, class Policy>
 562:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> gamma_q(RT1 a, RT2 z, const Policy&);
 563: 
 564:    template <class RT1, class RT2>
 565:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> gamma_p(RT1 a, RT2 z);
 566: 
 567:    template <class RT1, class RT2, class Policy>
 568:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT1, RT2> gamma_p(RT1 a, RT2 z, const Policy&);
 569: 
 570:    template <class T1, class T2>
 571:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> tgamma_delta_ratio(T1 z, T2 delta);
 572: 
 573:    template <class T1, class T2, class Policy>
 574:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> tgamma_delta_ratio(T1 z, T2 delta, const Policy&);
 575: 
 576:    template <class T1, class T2>
 577:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> tgamma_ratio(T1 a, T2 b);
 578: 
 579:    template <class T1, class T2, class Policy>
 580:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> tgamma_ratio(T1 a, T2 b, const Policy&);
 581: 
 582:    template <class T1, class T2>
 583:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_p_derivative(T1 a, T2 x);
 584: 
 585:    template <class T1, class T2, class Policy>
 586:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_p_derivative(T1 a, T2 x, const Policy&);
 587: 
 588:    // gamma inverse.
````
- **L561 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class Policy>`.
  - **L561 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class Policy>`。
- **L562 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L562 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L563 EN**: Blank line separating nearby declarations or logic.
  - **L563 CN**: 空行，用于分隔相邻声明或逻辑。
- **L564 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>`.
  - **L564 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>`。
- **L565 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L565 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L566 EN**: Blank line separating nearby declarations or logic.
  - **L566 CN**: 空行，用于分隔相邻声明或逻辑。
- **L567 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class Policy>`.
  - **L567 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class Policy>`。
- **L568 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L568 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L569 EN**: Blank line separating nearby declarations or logic.
  - **L569 CN**: 空行，用于分隔相邻声明或逻辑。
- **L570 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L570 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L571 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L571 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L572 EN**: Blank line separating nearby declarations or logic.
  - **L572 CN**: 空行，用于分隔相邻声明或逻辑。
- **L573 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L573 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L574 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L574 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L575 EN**: Blank line separating nearby declarations or logic.
  - **L575 CN**: 空行，用于分隔相邻声明或逻辑。
- **L576 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L576 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L577 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L577 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L578 EN**: Blank line separating nearby declarations or logic.
  - **L578 CN**: 空行，用于分隔相邻声明或逻辑。
- **L579 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L579 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L580 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L580 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L581 EN**: Blank line separating nearby declarations or logic.
  - **L581 CN**: 空行，用于分隔相邻声明或逻辑。
- **L582 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L582 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L583 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L583 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L584 EN**: Blank line separating nearby declarations or logic.
  - **L584 CN**: 空行，用于分隔相邻声明或逻辑。
- **L585 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L585 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L586 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L586 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L587 EN**: Blank line separating nearby declarations or logic.
  - **L587 CN**: 空行，用于分隔相邻声明或逻辑。
- **L588 EN**: Comment documents nearby intent or usage notes: `gamma inverse.`.
  - **L588 CN**: 注释说明附近代码的意图或使用说明：`gamma inverse.`。

### Lines 589-616 / 第 589-616 行

````cpp
 589:    template <class T1, class T2>
 590:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_p_inv(T1 a, T2 p);
 591: 
 592:    template <class T1, class T2, class Policy>
 593:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_p_inva(T1 a, T2 p, const Policy&);
 594: 
 595:    template <class T1, class T2>
 596:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_p_inva(T1 a, T2 p);
 597: 
 598:    template <class T1, class T2, class Policy>
 599:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_p_inv(T1 a, T2 p, const Policy&);
 600: 
 601:    template <class T1, class T2>
 602:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_q_inv(T1 a, T2 q);
 603: 
 604:    template <class T1, class T2, class Policy>
 605:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_q_inv(T1 a, T2 q, const Policy&);
 606: 
 607:    template <class T1, class T2>
 608:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_q_inva(T1 a, T2 q);
 609: 
 610:    template <class T1, class T2, class Policy>
 611:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2> gamma_q_inva(T1 a, T2 q, const Policy&);
 612: 
 613:    // digamma:
 614:    template <class T>
 615:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> digamma(T x);
 616: 
````
- **L589 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L589 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L590 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L590 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L591 EN**: Blank line separating nearby declarations or logic.
  - **L591 CN**: 空行，用于分隔相邻声明或逻辑。
- **L592 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L592 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L593 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L593 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L594 EN**: Blank line separating nearby declarations or logic.
  - **L594 CN**: 空行，用于分隔相邻声明或逻辑。
- **L595 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L595 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L596 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L596 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L597 EN**: Blank line separating nearby declarations or logic.
  - **L597 CN**: 空行，用于分隔相邻声明或逻辑。
- **L598 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L598 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L599 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L599 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L600 EN**: Blank line separating nearby declarations or logic.
  - **L600 CN**: 空行，用于分隔相邻声明或逻辑。
- **L601 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L601 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L602 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L602 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L603 EN**: Blank line separating nearby declarations or logic.
  - **L603 CN**: 空行，用于分隔相邻声明或逻辑。
- **L604 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L604 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L605 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L605 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L606 EN**: Blank line separating nearby declarations or logic.
  - **L606 CN**: 空行，用于分隔相邻声明或逻辑。
- **L607 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L607 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L608 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L608 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L609 EN**: Blank line separating nearby declarations or logic.
  - **L609 CN**: 空行，用于分隔相邻声明或逻辑。
- **L610 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L610 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L611 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L611 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L612 EN**: Blank line separating nearby declarations or logic.
  - **L612 CN**: 空行，用于分隔相邻声明或逻辑。
- **L613 EN**: Comment documents nearby intent or usage notes: `digamma:`.
  - **L613 CN**: 注释说明附近代码的意图或使用说明：`digamma:`。
- **L614 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L614 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L615 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L615 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L616 EN**: Blank line separating nearby declarations or logic.
  - **L616 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 617-644 / 第 617-644 行

````cpp
 617:    template <class T, class Policy>
 618:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> digamma(T x, const Policy&);
 619: 
 620:    // trigamma:
 621:    template <class T>
 622:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> trigamma(T x);
 623: 
 624:    template <class T, class Policy>
 625:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> trigamma(T x, const Policy&);
 626: 
 627:    // polygamma:
 628:    template <class T>
 629:    tools::promote_args_t<T> polygamma(int n, T x);
 630: 
 631:    template <class T, class Policy>
 632:    tools::promote_args_t<T> polygamma(int n, T x, const Policy&);
 633: 
 634:    // Hypotenuse function sqrt(x ^ 2 + y ^ 2).
 635:    template <class T1, class T2>
 636:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2>
 637:          hypot(T1 x, T2 y);
 638: 
 639:    template <class T1, class T2, class Policy>
 640:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2>
 641:          hypot(T1 x, T2 y, const Policy&);
 642: 
 643:    // cbrt - cube root.
 644:    template <class RT>
````
- **L617 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L617 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L618 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L618 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L619 EN**: Blank line separating nearby declarations or logic.
  - **L619 CN**: 空行，用于分隔相邻声明或逻辑。
- **L620 EN**: Comment documents nearby intent or usage notes: `trigamma:`.
  - **L620 CN**: 注释说明附近代码的意图或使用说明：`trigamma:`。
- **L621 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L621 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L622 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L622 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L623 EN**: Blank line separating nearby declarations or logic.
  - **L623 CN**: 空行，用于分隔相邻声明或逻辑。
- **L624 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L624 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L625 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L625 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L626 EN**: Blank line separating nearby declarations or logic.
  - **L626 CN**: 空行，用于分隔相邻声明或逻辑。
- **L627 EN**: Comment documents nearby intent or usage notes: `polygamma:`.
  - **L627 CN**: 注释说明附近代码的意图或使用说明：`polygamma:`。
- **L628 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L628 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L629 EN**: Executes a call or declaration centered on `polygamma`.
  - **L629 CN**: 执行以 `polygamma` 为核心的调用或声明。
- **L630 EN**: Blank line separating nearby declarations or logic.
  - **L630 CN**: 空行，用于分隔相邻声明或逻辑。
- **L631 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L631 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L632 EN**: Executes a call or declaration centered on `polygamma`.
  - **L632 CN**: 执行以 `polygamma` 为核心的调用或声明。
- **L633 EN**: Blank line separating nearby declarations or logic.
  - **L633 CN**: 空行，用于分隔相邻声明或逻辑。
- **L634 EN**: Comment documents nearby intent or usage notes: `Hypotenuse function sqrt(x ^ 2 + y ^ 2).`.
  - **L634 CN**: 注释说明附近代码的意图或使用说明：`Hypotenuse function sqrt(x ^ 2 + y ^ 2).`。
- **L635 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L635 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L636 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L636 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L637 EN**: Executes a call or declaration centered on `hypot`.
  - **L637 CN**: 执行以 `hypot` 为核心的调用或声明。
- **L638 EN**: Blank line separating nearby declarations or logic.
  - **L638 CN**: 空行，用于分隔相邻声明或逻辑。
- **L639 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L639 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L640 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L640 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L641 EN**: Executes a call or declaration centered on `hypot`.
  - **L641 CN**: 执行以 `hypot` 为核心的调用或声明。
- **L642 EN**: Blank line separating nearby declarations or logic.
  - **L642 CN**: 空行，用于分隔相邻声明或逻辑。
- **L643 EN**: Comment documents nearby intent or usage notes: `cbrt - cube root.`.
  - **L643 CN**: 注释说明附近代码的意图或使用说明：`cbrt - cube root.`。
- **L644 EN**: Introduces template parameters or specialization context: `template <class RT>`.
  - **L644 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>`。

### Lines 645-672 / 第 645-672 行

````cpp
 645:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> cbrt(RT z);
 646: 
 647:    template <class RT, class Policy>
 648:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<RT> cbrt(RT z, const Policy&);
 649: 
 650:    // log1p is log(x + 1)
 651:    template <class T>
 652:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> log1p(T);
 653: 
 654:    template <class T, class Policy>
 655:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> log1p(T, const Policy&);
 656: 
 657:    // log1pmx is log(x + 1) - x
 658:    template <class T>
 659:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> log1pmx(T);
 660: 
 661:    template <class T, class Policy>
 662:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> log1pmx(T, const Policy&);
 663: 
 664:    // Exp (x) minus 1 functions.
 665:    template <class T>
 666:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> expm1(T);
 667: 
 668:    template <class T, class Policy>
 669:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> expm1(T, const Policy&);
 670: 
 671:    // Power - 1
 672:    template <class T1, class T2>
````
- **L645 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L645 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L646 EN**: Blank line separating nearby declarations or logic.
  - **L646 CN**: 空行，用于分隔相邻声明或逻辑。
- **L647 EN**: Introduces template parameters or specialization context: `template <class RT, class Policy>`.
  - **L647 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT, class Policy>`。
- **L648 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L648 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L649 EN**: Blank line separating nearby declarations or logic.
  - **L649 CN**: 空行，用于分隔相邻声明或逻辑。
- **L650 EN**: Comment documents nearby intent or usage notes: `log1p is log(x + 1)`.
  - **L650 CN**: 注释说明附近代码的意图或使用说明：`log1p is log(x + 1)`。
- **L651 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L651 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L652 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L652 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L653 EN**: Blank line separating nearby declarations or logic.
  - **L653 CN**: 空行，用于分隔相邻声明或逻辑。
- **L654 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L654 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L655 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L655 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L656 EN**: Blank line separating nearby declarations or logic.
  - **L656 CN**: 空行，用于分隔相邻声明或逻辑。
- **L657 EN**: Comment documents nearby intent or usage notes: `log1pmx is log(x + 1) - x`.
  - **L657 CN**: 注释说明附近代码的意图或使用说明：`log1pmx is log(x + 1) - x`。
- **L658 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L658 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L659 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L659 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L660 EN**: Blank line separating nearby declarations or logic.
  - **L660 CN**: 空行，用于分隔相邻声明或逻辑。
- **L661 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L661 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L662 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L662 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L663 EN**: Blank line separating nearby declarations or logic.
  - **L663 CN**: 空行，用于分隔相邻声明或逻辑。
- **L664 EN**: Comment documents nearby intent or usage notes: `Exp (x) minus 1 functions.`.
  - **L664 CN**: 注释说明附近代码的意图或使用说明：`Exp (x) minus 1 functions.`。
- **L665 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L665 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L666 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L666 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L667 EN**: Blank line separating nearby declarations or logic.
  - **L667 CN**: 空行，用于分隔相邻声明或逻辑。
- **L668 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L668 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L669 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L669 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L670 EN**: Blank line separating nearby declarations or logic.
  - **L670 CN**: 空行，用于分隔相邻声明或逻辑。
- **L671 EN**: Comment documents nearby intent or usage notes: `Power - 1`.
  - **L671 CN**: 注释说明附近代码的意图或使用说明：`Power - 1`。
- **L672 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L672 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。

### Lines 673-700 / 第 673-700 行

````cpp
 673:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2>
 674:          powm1(const T1 a, const T2 z);
 675: 
 676:    template <class T1, class T2, class Policy>
 677:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T1, T2>
 678:          powm1(const T1 a, const T2 z, const Policy&);
 679: 
 680:    // sqrt(1+x) - 1
 681:    template <class T>
 682:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> sqrt1pm1(const T& val);
 683: 
 684:    template <class T, class Policy>
 685:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> sqrt1pm1(const T& val, const Policy&);
 686: 
 687:    // sinus cardinals:
 688:    template <class T>
 689:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> sinc_pi(T x);
 690: 
 691:    template <class T, class Policy>
 692:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> sinc_pi(T x, const Policy&);
 693: 
 694:    template <class T>
 695:    tools::promote_args_t<T> sinhc_pi(T x);
 696: 
 697:    template <class T, class Policy>
 698:    tools::promote_args_t<T> sinhc_pi(T x, const Policy&);
 699: 
 700:    // inverse hyperbolics:
````
- **L673 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L673 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L674 EN**: Executes a call or declaration centered on `powm1`.
  - **L674 CN**: 执行以 `powm1` 为核心的调用或声明。
- **L675 EN**: Blank line separating nearby declarations or logic.
  - **L675 CN**: 空行，用于分隔相邻声明或逻辑。
- **L676 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L676 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L677 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L677 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L678 EN**: Executes a call or declaration centered on `powm1`.
  - **L678 CN**: 执行以 `powm1` 为核心的调用或声明。
- **L679 EN**: Blank line separating nearby declarations or logic.
  - **L679 CN**: 空行，用于分隔相邻声明或逻辑。
- **L680 EN**: Comment documents nearby intent or usage notes: `sqrt(1+x) - 1`.
  - **L680 CN**: 注释说明附近代码的意图或使用说明：`sqrt(1+x) - 1`。
- **L681 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L681 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L682 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L682 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L683 EN**: Blank line separating nearby declarations or logic.
  - **L683 CN**: 空行，用于分隔相邻声明或逻辑。
- **L684 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L684 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L685 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L685 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L686 EN**: Blank line separating nearby declarations or logic.
  - **L686 CN**: 空行，用于分隔相邻声明或逻辑。
- **L687 EN**: Comment documents nearby intent or usage notes: `sinus cardinals:`.
  - **L687 CN**: 注释说明附近代码的意图或使用说明：`sinus cardinals:`。
- **L688 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L688 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L689 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L689 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L690 EN**: Blank line separating nearby declarations or logic.
  - **L690 CN**: 空行，用于分隔相邻声明或逻辑。
- **L691 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L691 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L692 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L692 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L693 EN**: Blank line separating nearby declarations or logic.
  - **L693 CN**: 空行，用于分隔相邻声明或逻辑。
- **L694 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L694 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L695 EN**: Executes a call or declaration centered on `sinhc_pi`.
  - **L695 CN**: 执行以 `sinhc_pi` 为核心的调用或声明。
- **L696 EN**: Blank line separating nearby declarations or logic.
  - **L696 CN**: 空行，用于分隔相邻声明或逻辑。
- **L697 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L697 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L698 EN**: Executes a call or declaration centered on `sinhc_pi`.
  - **L698 CN**: 执行以 `sinhc_pi` 为核心的调用或声明。
- **L699 EN**: Blank line separating nearby declarations or logic.
  - **L699 CN**: 空行，用于分隔相邻声明或逻辑。
- **L700 EN**: Comment documents nearby intent or usage notes: `inverse hyperbolics:`.
  - **L700 CN**: 注释说明附近代码的意图或使用说明：`inverse hyperbolics:`。

### Lines 701-728 / 第 701-728 行

````cpp
 701:    template<typename T>
 702:    tools::promote_args_t<T> asinh(T x);
 703: 
 704:    template<typename T, class Policy>
 705:    tools::promote_args_t<T> asinh(T x, const Policy&);
 706: 
 707:    template<typename T>
 708:    tools::promote_args_t<T> acosh(T x);
 709: 
 710:    template<typename T, class Policy>
 711:    tools::promote_args_t<T> acosh(T x, const Policy&);
 712: 
 713:    template<typename T>
 714:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> atanh(T x);
 715: 
 716:    template<typename T, class Policy>
 717:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> atanh(T x, const Policy&);
 718: 
 719:    namespace detail{
 720: 
 721:       typedef boost::math::integral_constant<int, 0> bessel_no_int_tag;      // No integer optimisation possible.
 722:       typedef boost::math::integral_constant<int, 1> bessel_maybe_int_tag;   // Maybe integer optimisation.
 723:       typedef boost::math::integral_constant<int, 2> bessel_int_tag;         // Definite integer optimisation.
 724: 
 725:       template <class T1, class T2, class Policy>
 726:       struct bessel_traits
 727:       {
 728:          using result_type = typename boost::math::conditional<
````
- **L701 EN**: Introduces template parameters or specialization context: `template<typename T>`.
  - **L701 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T>`。
- **L702 EN**: Executes a call or declaration centered on `asinh`.
  - **L702 CN**: 执行以 `asinh` 为核心的调用或声明。
- **L703 EN**: Blank line separating nearby declarations or logic.
  - **L703 CN**: 空行，用于分隔相邻声明或逻辑。
- **L704 EN**: Introduces template parameters or specialization context: `template<typename T, class Policy>`.
  - **L704 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T, class Policy>`。
- **L705 EN**: Executes a call or declaration centered on `asinh`.
  - **L705 CN**: 执行以 `asinh` 为核心的调用或声明。
- **L706 EN**: Blank line separating nearby declarations or logic.
  - **L706 CN**: 空行，用于分隔相邻声明或逻辑。
- **L707 EN**: Introduces template parameters or specialization context: `template<typename T>`.
  - **L707 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T>`。
- **L708 EN**: Executes a call or declaration centered on `acosh`.
  - **L708 CN**: 执行以 `acosh` 为核心的调用或声明。
- **L709 EN**: Blank line separating nearby declarations or logic.
  - **L709 CN**: 空行，用于分隔相邻声明或逻辑。
- **L710 EN**: Introduces template parameters or specialization context: `template<typename T, class Policy>`.
  - **L710 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T, class Policy>`。
- **L711 EN**: Executes a call or declaration centered on `acosh`.
  - **L711 CN**: 执行以 `acosh` 为核心的调用或声明。
- **L712 EN**: Blank line separating nearby declarations or logic.
  - **L712 CN**: 空行，用于分隔相邻声明或逻辑。
- **L713 EN**: Introduces template parameters or specialization context: `template<typename T>`.
  - **L713 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T>`。
- **L714 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L714 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L715 EN**: Blank line separating nearby declarations or logic.
  - **L715 CN**: 空行，用于分隔相邻声明或逻辑。
- **L716 EN**: Introduces template parameters or specialization context: `template<typename T, class Policy>`.
  - **L716 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T, class Policy>`。
- **L717 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L717 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L718 EN**: Blank line separating nearby declarations or logic.
  - **L718 CN**: 空行，用于分隔相邻声明或逻辑。
- **L719 EN**: Opens namespace scope `detail`.
  - **L719 CN**: 打开命名空间作用域 `detail`。
- **L720 EN**: Blank line separating nearby declarations or logic.
  - **L720 CN**: 空行，用于分隔相邻声明或逻辑。
- **L721 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int, 0> bessel_no_int_tag;      // No integer optimisation possible.`.
  - **L721 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int, 0> bessel_no_int_tag;      // No integer optimisation possible.`。
- **L722 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int, 1> bessel_maybe_int_tag;   // Maybe integer optimisation.`.
  - **L722 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int, 1> bessel_maybe_int_tag;   // Maybe integer optimisation.`。
- **L723 EN**: Introduces a legacy type alias or function typedef: `typedef boost::math::integral_constant<int, 2> bessel_int_tag;         // Definite integer optimisation.`.
  - **L723 CN**: 引入传统类型别名或函数 typedef：`typedef boost::math::integral_constant<int, 2> bessel_int_tag;         // Definite integer optimisation.`。
- **L724 EN**: Blank line separating nearby declarations or logic.
  - **L724 CN**: 空行，用于分隔相邻声明或逻辑。
- **L725 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L725 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L726 EN**: Declares struct `bessel_traits`.
  - **L726 CN**: 声明 struct `bessel_traits`。
- **L727 EN**: Opens a new lexical scope or compound statement.
  - **L727 CN**: 打开一个新的词法作用域或复合语句块。
- **L728 EN**: Defines alias `result_type` to simplify later code.
  - **L728 CN**: 定义别名 `result_type` 以简化后续代码。

### Lines 729-756 / 第 729-756 行

````cpp
 729:             boost::math::is_integral<T1>::value,
 730:             typename tools::promote_args<T2>::type,
 731:             tools::promote_args_t<T1, T2>
 732:          >::type;
 733: 
 734:          typedef typename policies::precision<result_type, Policy>::type precision_type;
 735: 
 736:          using optimisation_tag = typename boost::math::conditional<
 737:             (precision_type::value <= 0 || precision_type::value > 64),
 738:             bessel_no_int_tag,
 739:             typename boost::math::conditional<
 740:                boost::math::is_integral<T1>::value,
 741:                bessel_int_tag,
 742:                bessel_maybe_int_tag
 743:             >::type
 744:          >::type;
 745: 
 746:          using optimisation_tag128 = typename boost::math::conditional<
 747:             (precision_type::value <= 0 || precision_type::value > 113),
 748:             bessel_no_int_tag,
 749:             typename boost::math::conditional<
 750:                boost::math::is_integral<T1>::value,
 751:                bessel_int_tag,
 752:                bessel_maybe_int_tag
 753:             >::type
 754:          >::type;
 755:       };
 756:    } // detail
````
- **L729 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L729 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `typename tools::promote_args<T2>::type,`.
  - **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`typename tools::promote_args<T2>::type,`。
- **L731 EN**: Continues the surrounding expression or declaration: `tools::promote_args_t<T1, T2>`.
  - **L731 CN**: 继续构造周围的表达式或声明：`tools::promote_args_t<T1, T2>`。
- **L732 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L732 CN**: 执行一条独立语句或声明：`>::type;`。
- **L733 EN**: Blank line separating nearby declarations or logic.
  - **L733 CN**: 空行，用于分隔相邻声明或逻辑。
- **L734 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::precision<result_type, Policy>::type precision_type;`.
  - **L734 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::precision<result_type, Policy>::type precision_type;`。
- **L735 EN**: Blank line separating nearby declarations or logic.
  - **L735 CN**: 空行，用于分隔相邻声明或逻辑。
- **L736 EN**: Defines alias `optimisation_tag` to simplify later code.
  - **L736 CN**: 定义别名 `optimisation_tag` 以简化后续代码。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(precision_type::value <= 0 || precision_type::value > 64),`.
  - **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`(precision_type::value <= 0 || precision_type::value > 64),`。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_no_int_tag,`.
  - **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_no_int_tag,`。
- **L739 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L739 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L740 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L740 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_int_tag,`.
  - **L741 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_int_tag,`。
- **L742 EN**: Continues the surrounding expression or declaration: `bessel_maybe_int_tag`.
  - **L742 CN**: 继续构造周围的表达式或声明：`bessel_maybe_int_tag`。
- **L743 EN**: Continues the surrounding expression or declaration: `>::type`.
  - **L743 CN**: 继续构造周围的表达式或声明：`>::type`。
- **L744 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L744 CN**: 执行一条独立语句或声明：`>::type;`。
- **L745 EN**: Blank line separating nearby declarations or logic.
  - **L745 CN**: 空行，用于分隔相邻声明或逻辑。
- **L746 EN**: Defines alias `optimisation_tag128` to simplify later code.
  - **L746 CN**: 定义别名 `optimisation_tag128` 以简化后续代码。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(precision_type::value <= 0 || precision_type::value > 113),`.
  - **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`(precision_type::value <= 0 || precision_type::value > 113),`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_no_int_tag,`.
  - **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_no_int_tag,`。
- **L749 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L749 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L750 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L750 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bessel_int_tag,`.
  - **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`bessel_int_tag,`。
- **L752 EN**: Continues the surrounding expression or declaration: `bessel_maybe_int_tag`.
  - **L752 CN**: 继续构造周围的表达式或声明：`bessel_maybe_int_tag`。
- **L753 EN**: Continues the surrounding expression or declaration: `>::type`.
  - **L753 CN**: 继续构造周围的表达式或声明：`>::type`。
- **L754 EN**: Executes a standalone statement or declaration: `>::type;`.
  - **L754 CN**: 执行一条独立语句或声明：`>::type;`。
- **L755 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L755 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L756 EN**: Continues the surrounding expression or declaration: `} // detail`.
  - **L756 CN**: 继续构造周围的表达式或声明：`} // detail`。

### Lines 757-784 / 第 757-784 行

````cpp
 757: 
 758:    // Bessel functions:
 759:    template <class T1, class T2, class Policy>
 760:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_bessel_j(T1 v, T2 x, const Policy& pol);
 761:    template <class T1, class T2, class Policy>
 762:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_bessel_j_prime(T1 v, T2 x, const Policy& pol);
 763: 
 764:    template <class T1, class T2>
 765:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_bessel_j(T1 v, T2 x);
 766:    template <class T1, class T2>
 767:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_bessel_j_prime(T1 v, T2 x);
 768: 
 769:    template <class T, class Policy>
 770:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, Policy>::result_type sph_bessel(unsigned v, T x, const Policy& pol);
 771:    template <class T, class Policy>
 772:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, Policy>::result_type sph_bessel_prime(unsigned v, T x, const Policy& pol);
 773: 
 774:    template <class T>
 775:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, policies::policy<> >::result_type sph_bessel(unsigned v, T x);
 776:    template <class T>
 777:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, policies::policy<> >::result_type sph_bessel_prime(unsigned v, T x);
 778: 
 779:    template <class T1, class T2, class Policy>
 780:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_bessel_i(T1 v, T2 x, const Policy& pol);
 781:    template <class T1, class T2, class Policy>
 782:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_bessel_i_prime(T1 v, T2 x, const Policy& pol);
 783: 
 784:    template <class T1, class T2>
````
- **L757 EN**: Blank line separating nearby declarations or logic.
  - **L757 CN**: 空行，用于分隔相邻声明或逻辑。
- **L758 EN**: Comment documents nearby intent or usage notes: `Bessel functions:`.
  - **L758 CN**: 注释说明附近代码的意图或使用说明：`Bessel functions:`。
- **L759 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L759 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L760 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L760 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L761 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L761 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L762 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L762 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L763 EN**: Blank line separating nearby declarations or logic.
  - **L763 CN**: 空行，用于分隔相邻声明或逻辑。
- **L764 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L764 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L765 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L765 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L766 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L766 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L767 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L767 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L768 EN**: Blank line separating nearby declarations or logic.
  - **L768 CN**: 空行，用于分隔相邻声明或逻辑。
- **L769 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L769 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L770 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L770 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L771 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L771 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L772 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L772 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L773 EN**: Blank line separating nearby declarations or logic.
  - **L773 CN**: 空行，用于分隔相邻声明或逻辑。
- **L774 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L774 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L775 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L775 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L776 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L776 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L777 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L777 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L778 EN**: Blank line separating nearby declarations or logic.
  - **L778 CN**: 空行，用于分隔相邻声明或逻辑。
- **L779 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L779 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L780 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L780 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L781 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L781 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L782 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L782 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L783 EN**: Blank line separating nearby declarations or logic.
  - **L783 CN**: 空行，用于分隔相邻声明或逻辑。
- **L784 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L784 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。

### Lines 785-812 / 第 785-812 行

````cpp
 785:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_bessel_i(T1 v, T2 x);
 786:    template <class T1, class T2>
 787:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_bessel_i_prime(T1 v, T2 x);
 788: 
 789:    template <class T1, class T2, class Policy>
 790:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_bessel_k(T1 v, T2 x, const Policy& pol);
 791:    template <class T1, class T2, class Policy>
 792:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_bessel_k_prime(T1 v, T2 x, const Policy& pol);
 793: 
 794:    template <class T1, class T2>
 795:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_bessel_k(T1 v, T2 x);
 796:    template <class T1, class T2>
 797:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_bessel_k_prime(T1 v, T2 x);
 798: 
 799:    template <class T1, class T2, class Policy>
 800:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_neumann(T1 v, T2 x, const Policy& pol);
 801:    template <class T1, class T2, class Policy>
 802:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, Policy>::result_type cyl_neumann_prime(T1 v, T2 x, const Policy& pol);
 803: 
 804:    template <class T1, class T2>
 805:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_neumann(T1 v, T2 x);
 806:    template <class T1, class T2>
 807:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type cyl_neumann_prime(T1 v, T2 x);
 808: 
 809:    template <class T, class Policy>
 810:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, Policy>::result_type sph_neumann(unsigned v, T x, const Policy& pol);
 811:    template <class T, class Policy>
 812:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, Policy>::result_type sph_neumann_prime(unsigned v, T x, const Policy& pol);
````
- **L785 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L785 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L786 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L786 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L787 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L787 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L788 EN**: Blank line separating nearby declarations or logic.
  - **L788 CN**: 空行，用于分隔相邻声明或逻辑。
- **L789 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L789 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L790 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L790 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L791 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L791 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L792 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L792 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L793 EN**: Blank line separating nearby declarations or logic.
  - **L793 CN**: 空行，用于分隔相邻声明或逻辑。
- **L794 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L794 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L795 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L795 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L796 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L796 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L797 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L797 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L798 EN**: Blank line separating nearby declarations or logic.
  - **L798 CN**: 空行，用于分隔相邻声明或逻辑。
- **L799 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L799 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L800 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L800 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L801 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L801 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L802 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L802 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L803 EN**: Blank line separating nearby declarations or logic.
  - **L803 CN**: 空行，用于分隔相邻声明或逻辑。
- **L804 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L804 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L805 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L805 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L806 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L806 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L807 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L807 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L808 EN**: Blank line separating nearby declarations or logic.
  - **L808 CN**: 空行，用于分隔相邻声明或逻辑。
- **L809 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L809 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L810 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L810 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L811 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L811 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L812 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L812 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 813-840 / 第 813-840 行

````cpp
 813: 
 814:    template <class T>
 815:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, policies::policy<> >::result_type sph_neumann(unsigned v, T x);
 816:    template <class T>
 817:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, policies::policy<> >::result_type sph_neumann_prime(unsigned v, T x);
 818: 
 819:    template <class T, class Policy>
 820:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, Policy>::result_type cyl_bessel_j_zero(T v, int m, const Policy& pol);
 821: 
 822:    template <class T>
 823:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, policies::policy<> >::result_type cyl_bessel_j_zero(T v, int m);
 824: 
 825:    template <class T, class OutputIterator>
 826:    BOOST_MATH_GPU_ENABLED OutputIterator cyl_bessel_j_zero(T v,
 827:                           int start_index,
 828:                           unsigned number_of_zeros,
 829:                           OutputIterator out_it);
 830: 
 831:    template <class T, class OutputIterator, class Policy>
 832:    BOOST_MATH_GPU_ENABLED OutputIterator cyl_bessel_j_zero(T v,
 833:                           int start_index,
 834:                           unsigned number_of_zeros,
 835:                           OutputIterator out_it,
 836:                           const Policy&);
 837: 
 838:    template <class T, class Policy>
 839:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, Policy>::result_type cyl_neumann_zero(T v, int m, const Policy& pol);
 840: 
````
- **L813 EN**: Blank line separating nearby declarations or logic.
  - **L813 CN**: 空行，用于分隔相邻声明或逻辑。
- **L814 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L814 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L815 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L815 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L816 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L816 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L817 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L817 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L818 EN**: Blank line separating nearby declarations or logic.
  - **L818 CN**: 空行，用于分隔相邻声明或逻辑。
- **L819 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L819 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L820 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L820 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L821 EN**: Blank line separating nearby declarations or logic.
  - **L821 CN**: 空行，用于分隔相邻声明或逻辑。
- **L822 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L822 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L823 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L823 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L824 EN**: Blank line separating nearby declarations or logic.
  - **L824 CN**: 空行，用于分隔相邻声明或逻辑。
- **L825 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>`.
  - **L825 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>`。
- **L826 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L826 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L827 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L829 EN**: Executes a standalone statement or declaration: `OutputIterator out_it);`.
  - **L829 CN**: 执行一条独立语句或声明：`OutputIterator out_it);`。
- **L830 EN**: Blank line separating nearby declarations or logic.
  - **L830 CN**: 空行，用于分隔相邻声明或逻辑。
- **L831 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator, class Policy>`.
  - **L831 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator, class Policy>`。
- **L832 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L832 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator out_it,`.
  - **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator out_it,`。
- **L836 EN**: Executes a standalone statement or declaration: `const Policy&);`.
  - **L836 CN**: 执行一条独立语句或声明：`const Policy&);`。
- **L837 EN**: Blank line separating nearby declarations or logic.
  - **L837 CN**: 空行，用于分隔相邻声明或逻辑。
- **L838 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L838 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L839 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L839 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L840 EN**: Blank line separating nearby declarations or logic.
  - **L840 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 841-868 / 第 841-868 行

````cpp
 841:    template <class T>
 842:    BOOST_MATH_GPU_ENABLED typename detail::bessel_traits<T, T, policies::policy<> >::result_type cyl_neumann_zero(T v, int m);
 843: 
 844:    template <class T, class OutputIterator>
 845:    BOOST_MATH_GPU_ENABLED OutputIterator cyl_neumann_zero(T v,
 846:                          int start_index,
 847:                          unsigned number_of_zeros,
 848:                          OutputIterator out_it);
 849: 
 850:    template <class T, class OutputIterator, class Policy>
 851:    BOOST_MATH_GPU_ENABLED OutputIterator cyl_neumann_zero(T v,
 852:                          int start_index,
 853:                          unsigned number_of_zeros,
 854:                          OutputIterator out_it,
 855:                          const Policy&);
 856: 
 857:    template <class T1, class T2>
 858:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type> cyl_hankel_1(T1 v, T2 x);
 859: 
 860:    template <class T1, class T2, class Policy>
 861:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, Policy>::result_type> cyl_hankel_1(T1 v, T2 x, const Policy& pol);
 862: 
 863:    template <class T1, class T2, class Policy>
 864:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, Policy>::result_type> cyl_hankel_2(T1 v, T2 x, const Policy& pol);
 865: 
 866:    template <class T1, class T2>
 867:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type> cyl_hankel_2(T1 v, T2 x);
 868: 
````
- **L841 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L841 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L842 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L842 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L843 EN**: Blank line separating nearby declarations or logic.
  - **L843 CN**: 空行，用于分隔相邻声明或逻辑。
- **L844 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>`.
  - **L844 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>`。
- **L845 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L845 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L846 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L848 EN**: Executes a standalone statement or declaration: `OutputIterator out_it);`.
  - **L848 CN**: 执行一条独立语句或声明：`OutputIterator out_it);`。
- **L849 EN**: Blank line separating nearby declarations or logic.
  - **L849 CN**: 空行，用于分隔相邻声明或逻辑。
- **L850 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator, class Policy>`.
  - **L850 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator, class Policy>`。
- **L851 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L851 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator out_it,`.
  - **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator out_it,`。
- **L855 EN**: Executes a standalone statement or declaration: `const Policy&);`.
  - **L855 CN**: 执行一条独立语句或声明：`const Policy&);`。
- **L856 EN**: Blank line separating nearby declarations or logic.
  - **L856 CN**: 空行，用于分隔相邻声明或逻辑。
- **L857 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L857 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L858 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L858 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L859 EN**: Blank line separating nearby declarations or logic.
  - **L859 CN**: 空行，用于分隔相邻声明或逻辑。
- **L860 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L860 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L861 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L861 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L862 EN**: Blank line separating nearby declarations or logic.
  - **L862 CN**: 空行，用于分隔相邻声明或逻辑。
- **L863 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L863 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L864 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L864 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L865 EN**: Blank line separating nearby declarations or logic.
  - **L865 CN**: 空行，用于分隔相邻声明或逻辑。
- **L866 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L866 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L867 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L867 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L868 EN**: Blank line separating nearby declarations or logic.
  - **L868 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 869-896 / 第 869-896 行

````cpp
 869:    template <class T1, class T2, class Policy>
 870:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, Policy>::result_type> sph_hankel_1(T1 v, T2 x, const Policy& pol);
 871: 
 872:    template <class T1, class T2>
 873:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type> sph_hankel_1(T1 v, T2 x);
 874: 
 875:    template <class T1, class T2, class Policy>
 876:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, Policy>::result_type> sph_hankel_2(T1 v, T2 x, const Policy& pol);
 877: 
 878:    template <class T1, class T2>
 879:    BOOST_MATH_GPU_ENABLED boost::math::complex<typename detail::bessel_traits<T1, T2, policies::policy<> >::result_type> sph_hankel_2(T1 v, T2 x);
 880: 
 881:    template <class T, class Policy>
 882:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_ai(T x, const Policy&);
 883: 
 884:    template <class T>
 885:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_ai(T x);
 886: 
 887:    template <class T, class Policy>
 888:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_bi(T x, const Policy&);
 889: 
 890:    template <class T>
 891:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_bi(T x);
 892: 
 893:    template <class T, class Policy>
 894:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_ai_prime(T x, const Policy&);
 895: 
 896:    template <class T>
````
- **L869 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L869 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L870 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L870 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L871 EN**: Blank line separating nearby declarations or logic.
  - **L871 CN**: 空行，用于分隔相邻声明或逻辑。
- **L872 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L872 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L873 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L873 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L874 EN**: Blank line separating nearby declarations or logic.
  - **L874 CN**: 空行，用于分隔相邻声明或逻辑。
- **L875 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L875 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L876 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L876 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L877 EN**: Blank line separating nearby declarations or logic.
  - **L877 CN**: 空行，用于分隔相邻声明或逻辑。
- **L878 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L878 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L879 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L879 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L880 EN**: Blank line separating nearby declarations or logic.
  - **L880 CN**: 空行，用于分隔相邻声明或逻辑。
- **L881 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L881 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L882 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L882 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L883 EN**: Blank line separating nearby declarations or logic.
  - **L883 CN**: 空行，用于分隔相邻声明或逻辑。
- **L884 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L884 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L885 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L885 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L886 EN**: Blank line separating nearby declarations or logic.
  - **L886 CN**: 空行，用于分隔相邻声明或逻辑。
- **L887 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L887 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L888 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L888 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L889 EN**: Blank line separating nearby declarations or logic.
  - **L889 CN**: 空行，用于分隔相邻声明或逻辑。
- **L890 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L890 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L891 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L891 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L892 EN**: Blank line separating nearby declarations or logic.
  - **L892 CN**: 空行，用于分隔相邻声明或逻辑。
- **L893 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L893 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L894 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L894 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L895 EN**: Blank line separating nearby declarations or logic.
  - **L895 CN**: 空行，用于分隔相邻声明或逻辑。
- **L896 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L896 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。

### Lines 897-924 / 第 897-924 行

````cpp
 897:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_ai_prime(T x);
 898: 
 899:    template <class T, class Policy>
 900:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_bi_prime(T x, const Policy&);
 901: 
 902:    template <class T>
 903:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> airy_bi_prime(T x);
 904: 
 905:    template <class T>
 906:    BOOST_MATH_GPU_ENABLED T airy_ai_zero(int m);
 907:    template <class T, class Policy>
 908:    BOOST_MATH_GPU_ENABLED T airy_ai_zero(int m, const Policy&);
 909: 
 910:    template <class OutputIterator>
 911:    BOOST_MATH_GPU_ENABLED OutputIterator airy_ai_zero(
 912:                      int start_index,
 913:                      unsigned number_of_zeros,
 914:                      OutputIterator out_it);
 915:    template <class OutputIterator, class Policy>
 916:    BOOST_MATH_GPU_ENABLED OutputIterator airy_ai_zero(
 917:                      int start_index,
 918:                      unsigned number_of_zeros,
 919:                      OutputIterator out_it,
 920:                      const Policy&);
 921: 
 922:    template <class T>
 923:    BOOST_MATH_GPU_ENABLED T airy_bi_zero(int m);
 924:    template <class T, class Policy>
````
- **L897 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L897 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L898 EN**: Blank line separating nearby declarations or logic.
  - **L898 CN**: 空行，用于分隔相邻声明或逻辑。
- **L899 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L899 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L900 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L900 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L901 EN**: Blank line separating nearby declarations or logic.
  - **L901 CN**: 空行，用于分隔相邻声明或逻辑。
- **L902 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L902 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L903 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L903 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L904 EN**: Blank line separating nearby declarations or logic.
  - **L904 CN**: 空行，用于分隔相邻声明或逻辑。
- **L905 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L905 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L906 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L906 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L907 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L907 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L908 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L908 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L909 EN**: Blank line separating nearby declarations or logic.
  - **L909 CN**: 空行，用于分隔相邻声明或逻辑。
- **L910 EN**: Introduces template parameters or specialization context: `template <class OutputIterator>`.
  - **L910 CN**: 为后续声明引入模板参数或特化上下文：`template <class OutputIterator>`。
- **L911 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L911 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L914 EN**: Executes a standalone statement or declaration: `OutputIterator out_it);`.
  - **L914 CN**: 执行一条独立语句或声明：`OutputIterator out_it);`。
- **L915 EN**: Introduces template parameters or specialization context: `template <class OutputIterator, class Policy>`.
  - **L915 CN**: 为后续声明引入模板参数或特化上下文：`template <class OutputIterator, class Policy>`。
- **L916 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L916 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator out_it,`.
  - **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator out_it,`。
- **L920 EN**: Executes a standalone statement or declaration: `const Policy&);`.
  - **L920 CN**: 执行一条独立语句或声明：`const Policy&);`。
- **L921 EN**: Blank line separating nearby declarations or logic.
  - **L921 CN**: 空行，用于分隔相邻声明或逻辑。
- **L922 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L922 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L923 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L923 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L924 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L924 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。

### Lines 925-952 / 第 925-952 行

````cpp
 925:    BOOST_MATH_GPU_ENABLED T airy_bi_zero(int m, const Policy&);
 926: 
 927:    template <class OutputIterator>
 928:    BOOST_MATH_GPU_ENABLED OutputIterator airy_bi_zero(
 929:                      int start_index,
 930:                      unsigned number_of_zeros,
 931:                      OutputIterator out_it);
 932:    template <class OutputIterator, class Policy>
 933:    BOOST_MATH_GPU_ENABLED OutputIterator airy_bi_zero(
 934:                      int start_index,
 935:                      unsigned number_of_zeros,
 936:                      OutputIterator out_it,
 937:                      const Policy&);
 938: 
 939:    template <class T, class Policy>
 940:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> sin_pi(T x, const Policy&);
 941: 
 942:    template <class T>
 943:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> sin_pi(T x);
 944: 
 945:    template <class T, class Policy>
 946:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> cos_pi(T x, const Policy&);
 947: 
 948:    template <class T>
 949:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> cos_pi(T x);
 950: 
 951:    template <class T>
 952:    BOOST_MATH_GPU_ENABLED int fpclassify BOOST_NO_MACRO_EXPAND(T t);
````
- **L925 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L925 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L926 EN**: Blank line separating nearby declarations or logic.
  - **L926 CN**: 空行，用于分隔相邻声明或逻辑。
- **L927 EN**: Introduces template parameters or specialization context: `template <class OutputIterator>`.
  - **L927 CN**: 为后续声明引入模板参数或特化上下文：`template <class OutputIterator>`。
- **L928 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L928 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L931 EN**: Executes a standalone statement or declaration: `OutputIterator out_it);`.
  - **L931 CN**: 执行一条独立语句或声明：`OutputIterator out_it);`。
- **L932 EN**: Introduces template parameters or specialization context: `template <class OutputIterator, class Policy>`.
  - **L932 CN**: 为后续声明引入模板参数或特化上下文：`template <class OutputIterator, class Policy>`。
- **L933 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L933 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int start_index,`.
  - **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`int start_index,`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned number_of_zeros,`.
  - **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned number_of_zeros,`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator out_it,`.
  - **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator out_it,`。
- **L937 EN**: Executes a standalone statement or declaration: `const Policy&);`.
  - **L937 CN**: 执行一条独立语句或声明：`const Policy&);`。
- **L938 EN**: Blank line separating nearby declarations or logic.
  - **L938 CN**: 空行，用于分隔相邻声明或逻辑。
- **L939 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L939 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L940 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L940 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L941 EN**: Blank line separating nearby declarations or logic.
  - **L941 CN**: 空行，用于分隔相邻声明或逻辑。
- **L942 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L942 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L943 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L943 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L944 EN**: Blank line separating nearby declarations or logic.
  - **L944 CN**: 空行，用于分隔相邻声明或逻辑。
- **L945 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L945 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L946 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L946 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L947 EN**: Blank line separating nearby declarations or logic.
  - **L947 CN**: 空行，用于分隔相邻声明或逻辑。
- **L948 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L948 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L949 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L949 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L950 EN**: Blank line separating nearby declarations or logic.
  - **L950 CN**: 空行，用于分隔相邻声明或逻辑。
- **L951 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L951 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L952 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L952 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 953-980 / 第 953-980 行

````cpp
 953: 
 954:    template <class T>
 955:    BOOST_MATH_GPU_ENABLED bool isfinite BOOST_NO_MACRO_EXPAND(T z);
 956: 
 957:    template <class T>
 958:    BOOST_MATH_GPU_ENABLED bool isinf BOOST_NO_MACRO_EXPAND(T t);
 959: 
 960:    template <class T>
 961:    BOOST_MATH_GPU_ENABLED bool isnan BOOST_NO_MACRO_EXPAND(T t);
 962: 
 963:    template <class T>
 964:    BOOST_MATH_GPU_ENABLED bool isnormal BOOST_NO_MACRO_EXPAND(T t);
 965: 
 966:    template<class T>
 967:    BOOST_MATH_GPU_ENABLED int signbit BOOST_NO_MACRO_EXPAND(T x);
 968: 
 969:    template <class T>
 970:    BOOST_MATH_GPU_ENABLED int sign BOOST_NO_MACRO_EXPAND(const T& z);
 971: 
 972:    template <class T, class U>
 973:    BOOST_MATH_GPU_ENABLED typename tools::promote_args_permissive<T, U>::type 
 974:    copysign BOOST_NO_MACRO_EXPAND(const T& x, const U& y);
 975: 
 976:    template <class T>
 977:    BOOST_MATH_GPU_ENABLED typename tools::promote_args_permissive<T>::type 
 978:    changesign BOOST_NO_MACRO_EXPAND(const T& z);
 979: 
 980:    // Exponential integrals:
````
- **L953 EN**: Blank line separating nearby declarations or logic.
  - **L953 CN**: 空行，用于分隔相邻声明或逻辑。
- **L954 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L954 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L955 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L955 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L956 EN**: Blank line separating nearby declarations or logic.
  - **L956 CN**: 空行，用于分隔相邻声明或逻辑。
- **L957 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L957 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L958 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L958 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L959 EN**: Blank line separating nearby declarations or logic.
  - **L959 CN**: 空行，用于分隔相邻声明或逻辑。
- **L960 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L960 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L961 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L961 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L962 EN**: Blank line separating nearby declarations or logic.
  - **L962 CN**: 空行，用于分隔相邻声明或逻辑。
- **L963 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L963 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L964 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L964 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L965 EN**: Blank line separating nearby declarations or logic.
  - **L965 CN**: 空行，用于分隔相邻声明或逻辑。
- **L966 EN**: Introduces template parameters or specialization context: `template<class T>`.
  - **L966 CN**: 为后续声明引入模板参数或特化上下文：`template<class T>`。
- **L967 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L967 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L968 EN**: Blank line separating nearby declarations or logic.
  - **L968 CN**: 空行，用于分隔相邻声明或逻辑。
- **L969 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L969 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L970 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L970 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L971 EN**: Blank line separating nearby declarations or logic.
  - **L971 CN**: 空行，用于分隔相邻声明或逻辑。
- **L972 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L972 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L973 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L973 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L974 EN**: Executes a call or declaration centered on `BOOST_NO_MACRO_EXPAND`.
  - **L974 CN**: 执行以 `BOOST_NO_MACRO_EXPAND` 为核心的调用或声明。
- **L975 EN**: Blank line separating nearby declarations or logic.
  - **L975 CN**: 空行，用于分隔相邻声明或逻辑。
- **L976 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L976 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L977 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L977 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L978 EN**: Executes a call or declaration centered on `BOOST_NO_MACRO_EXPAND`.
  - **L978 CN**: 执行以 `BOOST_NO_MACRO_EXPAND` 为核心的调用或声明。
- **L979 EN**: Blank line separating nearby declarations or logic.
  - **L979 CN**: 空行，用于分隔相邻声明或逻辑。
- **L980 EN**: Comment documents nearby intent or usage notes: `Exponential integrals:`.
  - **L980 CN**: 注释说明附近代码的意图或使用说明：`Exponential integrals:`。

### Lines 981-1008 / 第 981-1008 行

````cpp
 981:    namespace detail{
 982: 
 983:    template <class T, class U>
 984:    struct expint_result
 985:    {
 986:       typedef typename boost::math::conditional<
 987:          policies::is_policy<U>::value,
 988:          tools::promote_args_t<T>,
 989:          typename tools::promote_args<U>::type
 990:       >::type type;
 991:    };
 992: 
 993:    } // namespace detail
 994: 
 995:    template <class T, class Policy>
 996:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> expint(unsigned n, T z, const Policy&);
 997: 
 998:    template <class T, class U>
 999:    BOOST_MATH_GPU_ENABLED typename detail::expint_result<T, U>::type expint(T const z, U const u);
1000: 
1001:    template <class T>
1002:    BOOST_MATH_GPU_ENABLED tools::promote_args_t<T> expint(T z);
1003: 
1004:    // Zeta:
1005:    template <class T, class Policy>
1006:    tools::promote_args_t<T> zeta(T s, const Policy&);
1007: 
1008:    // Owen's T function:
````
- **L981 EN**: Opens namespace scope `detail`.
  - **L981 CN**: 打开命名空间作用域 `detail`。
- **L982 EN**: Blank line separating nearby declarations or logic.
  - **L982 CN**: 空行，用于分隔相邻声明或逻辑。
- **L983 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L983 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L984 EN**: Declares struct `expint_result`.
  - **L984 CN**: 声明 struct `expint_result`。
- **L985 EN**: Opens a new lexical scope or compound statement.
  - **L985 CN**: 打开一个新的词法作用域或复合语句块。
- **L986 EN**: Introduces a legacy type alias or function typedef: `typedef typename boost::math::conditional<`.
  - **L986 CN**: 引入传统类型别名或函数 typedef：`typedef typename boost::math::conditional<`。
- **L987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::is_policy<U>::value,`.
  - **L987 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::is_policy<U>::value,`。
- **L988 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tools::promote_args_t<T>,`.
  - **L988 CN**: 继续一个多行参数列表、初始化器或聚合项：`tools::promote_args_t<T>,`。
- **L989 EN**: Continues the surrounding expression or declaration: `typename tools::promote_args<U>::type`.
  - **L989 CN**: 继续构造周围的表达式或声明：`typename tools::promote_args<U>::type`。
- **L990 EN**: Executes a standalone statement or declaration: `>::type type;`.
  - **L990 CN**: 执行一条独立语句或声明：`>::type type;`。
- **L991 EN**: Closes the current declaration scope such as a class, struct, or enum.
  - **L991 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L992 EN**: Blank line separating nearby declarations or logic.
  - **L992 CN**: 空行，用于分隔相邻声明或逻辑。
- **L993 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L993 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。
- **L994 EN**: Blank line separating nearby declarations or logic.
  - **L994 CN**: 空行，用于分隔相邻声明或逻辑。
- **L995 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L995 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L996 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L996 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L997 EN**: Blank line separating nearby declarations or logic.
  - **L997 CN**: 空行，用于分隔相邻声明或逻辑。
- **L998 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L998 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L999 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L999 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1000 EN**: Blank line separating nearby declarations or logic.
  - **L1000 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1001 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1001 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1002 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1002 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1003 EN**: Blank line separating nearby declarations or logic.
  - **L1003 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1004 EN**: Comment documents nearby intent or usage notes: `Zeta:`.
  - **L1004 CN**: 注释说明附近代码的意图或使用说明：`Zeta:`。
- **L1005 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1005 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1006 EN**: Executes a call or declaration centered on `zeta`.
  - **L1006 CN**: 执行以 `zeta` 为核心的调用或声明。
- **L1007 EN**: Blank line separating nearby declarations or logic.
  - **L1007 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1008 EN**: Comment documents nearby intent or usage notes: `Owen's T function:`.
  - **L1008 CN**: 注释说明附近代码的意图或使用说明：`Owen's T function:`。

### Lines 1009-1036 / 第 1009-1036 行

````cpp
1009:    template <class T1, class T2, class Policy>
1010:    tools::promote_args_t<T1, T2> owens_t(T1 h, T2 a, const Policy& pol);
1011: 
1012:    template <class T1, class T2>
1013:    tools::promote_args_t<T1, T2> owens_t(T1 h, T2 a);
1014: 
1015:    // Jacobi Functions:
1016:    template <class T, class U, class V, class Policy>
1017:    tools::promote_args_t<T, U, V> jacobi_elliptic(T k, U theta, V* pcn, V* pdn, const Policy&);
1018: 
1019:    template <class T, class U, class V>
1020:    tools::promote_args_t<T, U, V> jacobi_elliptic(T k, U theta, V* pcn = 0, V* pdn = 0);
1021: 
1022:    template <class U, class T, class Policy>
1023:    tools::promote_args_t<T, U> jacobi_sn(U k, T theta, const Policy& pol);
1024: 
1025:    template <class U, class T>
1026:    tools::promote_args_t<T, U> jacobi_sn(U k, T theta);
1027: 
1028:    template <class T, class U, class Policy>
1029:    tools::promote_args_t<T, U> jacobi_cn(T k, U theta, const Policy& pol);
1030: 
1031:    template <class T, class U>
1032:    tools::promote_args_t<T, U> jacobi_cn(T k, U theta);
1033: 
1034:    template <class T, class U, class Policy>
1035:    tools::promote_args_t<T, U> jacobi_dn(T k, U theta, const Policy& pol);
1036: 
````
- **L1009 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>`.
  - **L1009 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>`。
- **L1010 EN**: Executes a call or declaration centered on `owens_t`.
  - **L1010 CN**: 执行以 `owens_t` 为核心的调用或声明。
- **L1011 EN**: Blank line separating nearby declarations or logic.
  - **L1011 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1012 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>`.
  - **L1012 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>`。
- **L1013 EN**: Executes a call or declaration centered on `owens_t`.
  - **L1013 CN**: 执行以 `owens_t` 为核心的调用或声明。
- **L1014 EN**: Blank line separating nearby declarations or logic.
  - **L1014 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1015 EN**: Comment documents nearby intent or usage notes: `Jacobi Functions:`.
  - **L1015 CN**: 注释说明附近代码的意图或使用说明：`Jacobi Functions:`。
- **L1016 EN**: Introduces template parameters or specialization context: `template <class T, class U, class V, class Policy>`.
  - **L1016 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class V, class Policy>`。
- **L1017 EN**: Executes a call or declaration centered on `jacobi_elliptic`.
  - **L1017 CN**: 执行以 `jacobi_elliptic` 为核心的调用或声明。
- **L1018 EN**: Blank line separating nearby declarations or logic.
  - **L1018 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1019 EN**: Introduces template parameters or specialization context: `template <class T, class U, class V>`.
  - **L1019 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class V>`。
- **L1020 EN**: Executes a call or declaration centered on `jacobi_elliptic`.
  - **L1020 CN**: 执行以 `jacobi_elliptic` 为核心的调用或声明。
- **L1021 EN**: Blank line separating nearby declarations or logic.
  - **L1021 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1022 EN**: Introduces template parameters or specialization context: `template <class U, class T, class Policy>`.
  - **L1022 CN**: 为后续声明引入模板参数或特化上下文：`template <class U, class T, class Policy>`。
- **L1023 EN**: Executes a call or declaration centered on `jacobi_sn`.
  - **L1023 CN**: 执行以 `jacobi_sn` 为核心的调用或声明。
- **L1024 EN**: Blank line separating nearby declarations or logic.
  - **L1024 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1025 EN**: Introduces template parameters or specialization context: `template <class U, class T>`.
  - **L1025 CN**: 为后续声明引入模板参数或特化上下文：`template <class U, class T>`。
- **L1026 EN**: Executes a call or declaration centered on `jacobi_sn`.
  - **L1026 CN**: 执行以 `jacobi_sn` 为核心的调用或声明。
- **L1027 EN**: Blank line separating nearby declarations or logic.
  - **L1027 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1028 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1028 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1029 EN**: Executes a call or declaration centered on `jacobi_cn`.
  - **L1029 CN**: 执行以 `jacobi_cn` 为核心的调用或声明。
- **L1030 EN**: Blank line separating nearby declarations or logic.
  - **L1030 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1031 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1031 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1032 EN**: Executes a call or declaration centered on `jacobi_cn`.
  - **L1032 CN**: 执行以 `jacobi_cn` 为核心的调用或声明。
- **L1033 EN**: Blank line separating nearby declarations or logic.
  - **L1033 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1034 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1034 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1035 EN**: Executes a call or declaration centered on `jacobi_dn`.
  - **L1035 CN**: 执行以 `jacobi_dn` 为核心的调用或声明。
- **L1036 EN**: Blank line separating nearby declarations or logic.
  - **L1036 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 1037-1064 / 第 1037-1064 行

````cpp
1037:    template <class T, class U>
1038:    tools::promote_args_t<T, U> jacobi_dn(T k, U theta);
1039: 
1040:    template <class T, class U, class Policy>
1041:    tools::promote_args_t<T, U> jacobi_cd(T k, U theta, const Policy& pol);
1042: 
1043:    template <class T, class U>
1044:    tools::promote_args_t<T, U> jacobi_cd(T k, U theta);
1045: 
1046:    template <class T, class U, class Policy>
1047:    tools::promote_args_t<T, U> jacobi_dc(T k, U theta, const Policy& pol);
1048: 
1049:    template <class T, class U>
1050:    tools::promote_args_t<T, U> jacobi_dc(T k, U theta);
1051: 
1052:    template <class T, class U, class Policy>
1053:    tools::promote_args_t<T, U> jacobi_ns(T k, U theta, const Policy& pol);
1054: 
1055:    template <class T, class U>
1056:    tools::promote_args_t<T, U> jacobi_ns(T k, U theta);
1057: 
1058:    template <class T, class U, class Policy>
1059:    tools::promote_args_t<T, U> jacobi_sd(T k, U theta, const Policy& pol);
1060: 
1061:    template <class T, class U>
1062:    tools::promote_args_t<T, U> jacobi_sd(T k, U theta);
1063: 
1064:    template <class T, class U, class Policy>
````
- **L1037 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1037 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1038 EN**: Executes a call or declaration centered on `jacobi_dn`.
  - **L1038 CN**: 执行以 `jacobi_dn` 为核心的调用或声明。
- **L1039 EN**: Blank line separating nearby declarations or logic.
  - **L1039 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1040 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1040 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1041 EN**: Executes a call or declaration centered on `jacobi_cd`.
  - **L1041 CN**: 执行以 `jacobi_cd` 为核心的调用或声明。
- **L1042 EN**: Blank line separating nearby declarations or logic.
  - **L1042 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1043 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1043 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1044 EN**: Executes a call or declaration centered on `jacobi_cd`.
  - **L1044 CN**: 执行以 `jacobi_cd` 为核心的调用或声明。
- **L1045 EN**: Blank line separating nearby declarations or logic.
  - **L1045 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1046 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1046 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1047 EN**: Executes a call or declaration centered on `jacobi_dc`.
  - **L1047 CN**: 执行以 `jacobi_dc` 为核心的调用或声明。
- **L1048 EN**: Blank line separating nearby declarations or logic.
  - **L1048 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1049 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1049 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1050 EN**: Executes a call or declaration centered on `jacobi_dc`.
  - **L1050 CN**: 执行以 `jacobi_dc` 为核心的调用或声明。
- **L1051 EN**: Blank line separating nearby declarations or logic.
  - **L1051 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1052 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1052 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1053 EN**: Executes a call or declaration centered on `jacobi_ns`.
  - **L1053 CN**: 执行以 `jacobi_ns` 为核心的调用或声明。
- **L1054 EN**: Blank line separating nearby declarations or logic.
  - **L1054 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1055 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1055 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1056 EN**: Executes a call or declaration centered on `jacobi_ns`.
  - **L1056 CN**: 执行以 `jacobi_ns` 为核心的调用或声明。
- **L1057 EN**: Blank line separating nearby declarations or logic.
  - **L1057 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1058 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1058 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1059 EN**: Executes a call or declaration centered on `jacobi_sd`.
  - **L1059 CN**: 执行以 `jacobi_sd` 为核心的调用或声明。
- **L1060 EN**: Blank line separating nearby declarations or logic.
  - **L1060 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1061 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1061 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1062 EN**: Executes a call or declaration centered on `jacobi_sd`.
  - **L1062 CN**: 执行以 `jacobi_sd` 为核心的调用或声明。
- **L1063 EN**: Blank line separating nearby declarations or logic.
  - **L1063 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1064 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1064 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。

### Lines 1065-1092 / 第 1065-1092 行

````cpp
1065:    tools::promote_args_t<T, U> jacobi_ds(T k, U theta, const Policy& pol);
1066: 
1067:    template <class T, class U>
1068:    tools::promote_args_t<T, U> jacobi_ds(T k, U theta);
1069: 
1070:    template <class T, class U, class Policy>
1071:    tools::promote_args_t<T, U> jacobi_nc(T k, U theta, const Policy& pol);
1072: 
1073:    template <class T, class U>
1074:    tools::promote_args_t<T, U> jacobi_nc(T k, U theta);
1075: 
1076:    template <class T, class U, class Policy>
1077:    tools::promote_args_t<T, U> jacobi_nd(T k, U theta, const Policy& pol);
1078: 
1079:    template <class T, class U>
1080:    tools::promote_args_t<T, U> jacobi_nd(T k, U theta);
1081: 
1082:    template <class T, class U, class Policy>
1083:    tools::promote_args_t<T, U> jacobi_sc(T k, U theta, const Policy& pol);
1084: 
1085:    template <class T, class U>
1086:    tools::promote_args_t<T, U> jacobi_sc(T k, U theta);
1087: 
1088:    template <class T, class U, class Policy>
1089:    tools::promote_args_t<T, U> jacobi_cs(T k, U theta, const Policy& pol);
1090: 
1091:    template <class T, class U>
1092:    tools::promote_args_t<T, U> jacobi_cs(T k, U theta);
````
- **L1065 EN**: Executes a call or declaration centered on `jacobi_ds`.
  - **L1065 CN**: 执行以 `jacobi_ds` 为核心的调用或声明。
- **L1066 EN**: Blank line separating nearby declarations or logic.
  - **L1066 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1067 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1067 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1068 EN**: Executes a call or declaration centered on `jacobi_ds`.
  - **L1068 CN**: 执行以 `jacobi_ds` 为核心的调用或声明。
- **L1069 EN**: Blank line separating nearby declarations or logic.
  - **L1069 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1070 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1070 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1071 EN**: Executes a call or declaration centered on `jacobi_nc`.
  - **L1071 CN**: 执行以 `jacobi_nc` 为核心的调用或声明。
- **L1072 EN**: Blank line separating nearby declarations or logic.
  - **L1072 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1073 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1073 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1074 EN**: Executes a call or declaration centered on `jacobi_nc`.
  - **L1074 CN**: 执行以 `jacobi_nc` 为核心的调用或声明。
- **L1075 EN**: Blank line separating nearby declarations or logic.
  - **L1075 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1076 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1076 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1077 EN**: Executes a call or declaration centered on `jacobi_nd`.
  - **L1077 CN**: 执行以 `jacobi_nd` 为核心的调用或声明。
- **L1078 EN**: Blank line separating nearby declarations or logic.
  - **L1078 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1079 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1079 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1080 EN**: Executes a call or declaration centered on `jacobi_nd`.
  - **L1080 CN**: 执行以 `jacobi_nd` 为核心的调用或声明。
- **L1081 EN**: Blank line separating nearby declarations or logic.
  - **L1081 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1082 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1082 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1083 EN**: Executes a call or declaration centered on `jacobi_sc`.
  - **L1083 CN**: 执行以 `jacobi_sc` 为核心的调用或声明。
- **L1084 EN**: Blank line separating nearby declarations or logic.
  - **L1084 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1085 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1085 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1086 EN**: Executes a call or declaration centered on `jacobi_sc`.
  - **L1086 CN**: 执行以 `jacobi_sc` 为核心的调用或声明。
- **L1087 EN**: Blank line separating nearby declarations or logic.
  - **L1087 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1088 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1088 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1089 EN**: Executes a call or declaration centered on `jacobi_cs`.
  - **L1089 CN**: 执行以 `jacobi_cs` 为核心的调用或声明。
- **L1090 EN**: Blank line separating nearby declarations or logic.
  - **L1090 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1091 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1091 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1092 EN**: Executes a call or declaration centered on `jacobi_cs`.
  - **L1092 CN**: 执行以 `jacobi_cs` 为核心的调用或声明。

### Lines 1093-1120 / 第 1093-1120 行

````cpp
1093: 
1094:    // Jacobi Theta Functions:
1095:    template <class T, class U, class Policy>
1096:    tools::promote_args_t<T, U> jacobi_theta1(T z, U q, const Policy& pol);
1097: 
1098:    template <class T, class U>
1099:    tools::promote_args_t<T, U> jacobi_theta1(T z, U q);
1100: 
1101:    template <class T, class U, class Policy>
1102:    tools::promote_args_t<T, U> jacobi_theta2(T z, U q, const Policy& pol);
1103: 
1104:    template <class T, class U>
1105:    tools::promote_args_t<T, U> jacobi_theta2(T z, U q);
1106: 
1107:    template <class T, class U, class Policy>
1108:    tools::promote_args_t<T, U> jacobi_theta3(T z, U q, const Policy& pol);
1109: 
1110:    template <class T, class U>
1111:    tools::promote_args_t<T, U> jacobi_theta3(T z, U q);
1112: 
1113:    template <class T, class U, class Policy>
1114:    tools::promote_args_t<T, U> jacobi_theta4(T z, U q, const Policy& pol);
1115: 
1116:    template <class T, class U>
1117:    tools::promote_args_t<T, U> jacobi_theta4(T z, U q);
1118: 
1119:    template <class T, class U, class Policy>
1120:    tools::promote_args_t<T, U> jacobi_theta1tau(T z, U tau, const Policy& pol);
````
- **L1093 EN**: Blank line separating nearby declarations or logic.
  - **L1093 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1094 EN**: Comment documents nearby intent or usage notes: `Jacobi Theta Functions:`.
  - **L1094 CN**: 注释说明附近代码的意图或使用说明：`Jacobi Theta Functions:`。
- **L1095 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1095 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1096 EN**: Executes a call or declaration centered on `jacobi_theta1`.
  - **L1096 CN**: 执行以 `jacobi_theta1` 为核心的调用或声明。
- **L1097 EN**: Blank line separating nearby declarations or logic.
  - **L1097 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1098 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1098 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1099 EN**: Executes a call or declaration centered on `jacobi_theta1`.
  - **L1099 CN**: 执行以 `jacobi_theta1` 为核心的调用或声明。
- **L1100 EN**: Blank line separating nearby declarations or logic.
  - **L1100 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1101 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1101 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1102 EN**: Executes a call or declaration centered on `jacobi_theta2`.
  - **L1102 CN**: 执行以 `jacobi_theta2` 为核心的调用或声明。
- **L1103 EN**: Blank line separating nearby declarations or logic.
  - **L1103 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1104 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1104 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1105 EN**: Executes a call or declaration centered on `jacobi_theta2`.
  - **L1105 CN**: 执行以 `jacobi_theta2` 为核心的调用或声明。
- **L1106 EN**: Blank line separating nearby declarations or logic.
  - **L1106 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1107 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1107 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1108 EN**: Executes a call or declaration centered on `jacobi_theta3`.
  - **L1108 CN**: 执行以 `jacobi_theta3` 为核心的调用或声明。
- **L1109 EN**: Blank line separating nearby declarations or logic.
  - **L1109 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1110 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1110 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1111 EN**: Executes a call or declaration centered on `jacobi_theta3`.
  - **L1111 CN**: 执行以 `jacobi_theta3` 为核心的调用或声明。
- **L1112 EN**: Blank line separating nearby declarations or logic.
  - **L1112 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1113 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1113 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1114 EN**: Executes a call or declaration centered on `jacobi_theta4`.
  - **L1114 CN**: 执行以 `jacobi_theta4` 为核心的调用或声明。
- **L1115 EN**: Blank line separating nearby declarations or logic.
  - **L1115 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1116 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1116 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1117 EN**: Executes a call or declaration centered on `jacobi_theta4`.
  - **L1117 CN**: 执行以 `jacobi_theta4` 为核心的调用或声明。
- **L1118 EN**: Blank line separating nearby declarations or logic.
  - **L1118 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1119 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1119 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1120 EN**: Executes a call or declaration centered on `jacobi_theta1tau`.
  - **L1120 CN**: 执行以 `jacobi_theta1tau` 为核心的调用或声明。

### Lines 1121-1148 / 第 1121-1148 行

````cpp
1121: 
1122:    template <class T, class U>
1123:    tools::promote_args_t<T, U> jacobi_theta1tau(T z, U tau);
1124: 
1125:    template <class T, class U, class Policy>
1126:    tools::promote_args_t<T, U> jacobi_theta2tau(T z, U tau, const Policy& pol);
1127: 
1128:    template <class T, class U>
1129:    tools::promote_args_t<T, U> jacobi_theta2tau(T z, U tau);
1130: 
1131:    template <class T, class U, class Policy>
1132:    tools::promote_args_t<T, U> jacobi_theta3tau(T z, U tau, const Policy& pol);
1133: 
1134:    template <class T, class U>
1135:    tools::promote_args_t<T, U> jacobi_theta3tau(T z, U tau);
1136: 
1137:    template <class T, class U, class Policy>
1138:    tools::promote_args_t<T, U> jacobi_theta4tau(T z, U tau, const Policy& pol);
1139: 
1140:    template <class T, class U>
1141:    tools::promote_args_t<T, U> jacobi_theta4tau(T z, U tau);
1142: 
1143:    template <class T, class U, class Policy>
1144:    tools::promote_args_t<T, U> jacobi_theta3m1(T z, U q, const Policy& pol);
1145: 
1146:    template <class T, class U>
1147:    tools::promote_args_t<T, U> jacobi_theta3m1(T z, U q);
1148: 
````
- **L1121 EN**: Blank line separating nearby declarations or logic.
  - **L1121 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1122 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1122 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1123 EN**: Executes a call or declaration centered on `jacobi_theta1tau`.
  - **L1123 CN**: 执行以 `jacobi_theta1tau` 为核心的调用或声明。
- **L1124 EN**: Blank line separating nearby declarations or logic.
  - **L1124 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1125 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1125 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1126 EN**: Executes a call or declaration centered on `jacobi_theta2tau`.
  - **L1126 CN**: 执行以 `jacobi_theta2tau` 为核心的调用或声明。
- **L1127 EN**: Blank line separating nearby declarations or logic.
  - **L1127 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1128 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1128 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1129 EN**: Executes a call or declaration centered on `jacobi_theta2tau`.
  - **L1129 CN**: 执行以 `jacobi_theta2tau` 为核心的调用或声明。
- **L1130 EN**: Blank line separating nearby declarations or logic.
  - **L1130 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1131 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1131 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1132 EN**: Executes a call or declaration centered on `jacobi_theta3tau`.
  - **L1132 CN**: 执行以 `jacobi_theta3tau` 为核心的调用或声明。
- **L1133 EN**: Blank line separating nearby declarations or logic.
  - **L1133 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1134 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1134 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1135 EN**: Executes a call or declaration centered on `jacobi_theta3tau`.
  - **L1135 CN**: 执行以 `jacobi_theta3tau` 为核心的调用或声明。
- **L1136 EN**: Blank line separating nearby declarations or logic.
  - **L1136 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1137 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1137 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1138 EN**: Executes a call or declaration centered on `jacobi_theta4tau`.
  - **L1138 CN**: 执行以 `jacobi_theta4tau` 为核心的调用或声明。
- **L1139 EN**: Blank line separating nearby declarations or logic.
  - **L1139 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1140 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1140 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1141 EN**: Executes a call or declaration centered on `jacobi_theta4tau`.
  - **L1141 CN**: 执行以 `jacobi_theta4tau` 为核心的调用或声明。
- **L1142 EN**: Blank line separating nearby declarations or logic.
  - **L1142 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1143 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1143 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1144 EN**: Executes a call or declaration centered on `jacobi_theta3m1`.
  - **L1144 CN**: 执行以 `jacobi_theta3m1` 为核心的调用或声明。
- **L1145 EN**: Blank line separating nearby declarations or logic.
  - **L1145 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1146 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1146 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1147 EN**: Executes a call or declaration centered on `jacobi_theta3m1`.
  - **L1147 CN**: 执行以 `jacobi_theta3m1` 为核心的调用或声明。
- **L1148 EN**: Blank line separating nearby declarations or logic.
  - **L1148 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 1149-1176 / 第 1149-1176 行

````cpp
1149:    template <class T, class U, class Policy>
1150:    tools::promote_args_t<T, U> jacobi_theta4m1(T z, U q, const Policy& pol);
1151: 
1152:    template <class T, class U>
1153:    tools::promote_args_t<T, U> jacobi_theta4m1(T z, U q);
1154: 
1155:    template <class T, class U, class Policy>
1156:    tools::promote_args_t<T, U> jacobi_theta3m1tau(T z, U tau, const Policy& pol);
1157: 
1158:    template <class T, class U>
1159:    tools::promote_args_t<T, U> jacobi_theta3m1tau(T z, U tau);
1160: 
1161:    template <class T, class U, class Policy>
1162:    tools::promote_args_t<T, U> jacobi_theta4m1tau(T z, U tau, const Policy& pol);
1163: 
1164:    template <class T, class U>
1165:    tools::promote_args_t<T, U> jacobi_theta4m1tau(T z, U tau);
1166: 
1167: 
1168:    template <class T>
1169:    tools::promote_args_t<T> zeta(T s);
1170: 
1171:    // pow:
1172:    template <int N, typename T, class Policy>
1173:    BOOST_MATH_GPU_ENABLED BOOST_MATH_CXX14_CONSTEXPR tools::promote_args_t<T> pow(T base, const Policy& policy);
1174: 
1175:    template <int N, typename T>
1176:    BOOST_MATH_GPU_ENABLED BOOST_MATH_CXX14_CONSTEXPR tools::promote_args_t<T> pow(T base);
````
- **L1149 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1149 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1150 EN**: Executes a call or declaration centered on `jacobi_theta4m1`.
  - **L1150 CN**: 执行以 `jacobi_theta4m1` 为核心的调用或声明。
- **L1151 EN**: Blank line separating nearby declarations or logic.
  - **L1151 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1152 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1152 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1153 EN**: Executes a call or declaration centered on `jacobi_theta4m1`.
  - **L1153 CN**: 执行以 `jacobi_theta4m1` 为核心的调用或声明。
- **L1154 EN**: Blank line separating nearby declarations or logic.
  - **L1154 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1155 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1155 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1156 EN**: Executes a call or declaration centered on `jacobi_theta3m1tau`.
  - **L1156 CN**: 执行以 `jacobi_theta3m1tau` 为核心的调用或声明。
- **L1157 EN**: Blank line separating nearby declarations or logic.
  - **L1157 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1158 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1158 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1159 EN**: Executes a call or declaration centered on `jacobi_theta3m1tau`.
  - **L1159 CN**: 执行以 `jacobi_theta3m1tau` 为核心的调用或声明。
- **L1160 EN**: Blank line separating nearby declarations or logic.
  - **L1160 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1161 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1161 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1162 EN**: Executes a call or declaration centered on `jacobi_theta4m1tau`.
  - **L1162 CN**: 执行以 `jacobi_theta4m1tau` 为核心的调用或声明。
- **L1163 EN**: Blank line separating nearby declarations or logic.
  - **L1163 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1164 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1164 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1165 EN**: Executes a call or declaration centered on `jacobi_theta4m1tau`.
  - **L1165 CN**: 执行以 `jacobi_theta4m1tau` 为核心的调用或声明。
- **L1166 EN**: Blank line separating nearby declarations or logic.
  - **L1166 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1167 EN**: Blank line separating nearby declarations or logic.
  - **L1167 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1168 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1168 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1169 EN**: Executes a call or declaration centered on `zeta`.
  - **L1169 CN**: 执行以 `zeta` 为核心的调用或声明。
- **L1170 EN**: Blank line separating nearby declarations or logic.
  - **L1170 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1171 EN**: Comment documents nearby intent or usage notes: `pow:`.
  - **L1171 CN**: 注释说明附近代码的意图或使用说明：`pow:`。
- **L1172 EN**: Introduces template parameters or specialization context: `template <int N, typename T, class Policy>`.
  - **L1172 CN**: 为后续声明引入模板参数或特化上下文：`template <int N, typename T, class Policy>`。
- **L1173 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1173 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1174 EN**: Blank line separating nearby declarations or logic.
  - **L1174 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1175 EN**: Introduces template parameters or specialization context: `template <int N, typename T>`.
  - **L1175 CN**: 为后续声明引入模板参数或特化上下文：`template <int N, typename T>`。
- **L1176 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1176 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1177-1204 / 第 1177-1204 行

````cpp
1177: 
1178:    // next:
1179:    template <class T, class U, class Policy>
1180:    tools::promote_args_t<T, U> nextafter(const T&, const U&, const Policy&);
1181:    template <class T, class U>
1182:    tools::promote_args_t<T, U> nextafter(const T&, const U&);
1183:    template <class T, class Policy>
1184:    tools::promote_args_t<T> float_next(const T&, const Policy&);
1185:    template <class T>
1186:    tools::promote_args_t<T> float_next(const T&);
1187:    template <class T, class Policy>
1188:    tools::promote_args_t<T> float_prior(const T&, const Policy&);
1189:    template <class T>
1190:    tools::promote_args_t<T> float_prior(const T&);
1191:    template <class T, class U, class Policy>
1192:    tools::promote_args_t<T, U> float_distance(const T&, const U&, const Policy&);
1193:    template <class T, class U>
1194:    tools::promote_args_t<T, U> float_distance(const T&, const U&);
1195:    template <class T, class Policy>
1196:    tools::promote_args_t<T> float_advance(T val, int distance, const Policy& pol);
1197:    template <class T>
1198:    tools::promote_args_t<T> float_advance(const T& val, int distance);
1199: 
1200:    template <class T, class Policy>
1201:    tools::promote_args_t<T> ulp(const T& val, const Policy& pol);
1202:    template <class T>
1203:    tools::promote_args_t<T> ulp(const T& val);
1204: 
````
- **L1177 EN**: Blank line separating nearby declarations or logic.
  - **L1177 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1178 EN**: Comment documents nearby intent or usage notes: `next:`.
  - **L1178 CN**: 注释说明附近代码的意图或使用说明：`next:`。
- **L1179 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1179 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1180 EN**: Executes a call or declaration centered on `nextafter`.
  - **L1180 CN**: 执行以 `nextafter` 为核心的调用或声明。
- **L1181 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1181 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1182 EN**: Executes a call or declaration centered on `nextafter`.
  - **L1182 CN**: 执行以 `nextafter` 为核心的调用或声明。
- **L1183 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1183 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1184 EN**: Executes a call or declaration centered on `float_next`.
  - **L1184 CN**: 执行以 `float_next` 为核心的调用或声明。
- **L1185 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1185 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1186 EN**: Executes a call or declaration centered on `float_next`.
  - **L1186 CN**: 执行以 `float_next` 为核心的调用或声明。
- **L1187 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1187 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1188 EN**: Executes a call or declaration centered on `float_prior`.
  - **L1188 CN**: 执行以 `float_prior` 为核心的调用或声明。
- **L1189 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1189 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1190 EN**: Executes a call or declaration centered on `float_prior`.
  - **L1190 CN**: 执行以 `float_prior` 为核心的调用或声明。
- **L1191 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L1191 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L1192 EN**: Executes a call or declaration centered on `float_distance`.
  - **L1192 CN**: 执行以 `float_distance` 为核心的调用或声明。
- **L1193 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1193 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1194 EN**: Executes a call or declaration centered on `float_distance`.
  - **L1194 CN**: 执行以 `float_distance` 为核心的调用或声明。
- **L1195 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1195 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1196 EN**: Executes a call or declaration centered on `float_advance`.
  - **L1196 CN**: 执行以 `float_advance` 为核心的调用或声明。
- **L1197 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1197 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1198 EN**: Executes a call or declaration centered on `float_advance`.
  - **L1198 CN**: 执行以 `float_advance` 为核心的调用或声明。
- **L1199 EN**: Blank line separating nearby declarations or logic.
  - **L1199 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1200 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1200 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1201 EN**: Executes a call or declaration centered on `ulp`.
  - **L1201 CN**: 执行以 `ulp` 为核心的调用或声明。
- **L1202 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1202 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1203 EN**: Executes a call or declaration centered on `ulp`.
  - **L1203 CN**: 执行以 `ulp` 为核心的调用或声明。
- **L1204 EN**: Blank line separating nearby declarations or logic.
  - **L1204 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 1205-1232 / 第 1205-1232 行

````cpp
1205:    template <class T, class U>
1206:    tools::promote_args_t<T, U> relative_difference(const T&, const U&);
1207:    template <class T, class U>
1208:    tools::promote_args_t<T, U> epsilon_difference(const T&, const U&);
1209: 
1210:    template<class T>
1211:    BOOST_MATH_CONSTEXPR_TABLE_FUNCTION T unchecked_bernoulli_b2n(const std::size_t n);
1212:    template <class T, class Policy>
1213:    T bernoulli_b2n(const int i, const Policy &pol);
1214:    template <class T>
1215:    T bernoulli_b2n(const int i);
1216:    template <class T, class OutputIterator, class Policy>
1217:    OutputIterator bernoulli_b2n(const int start_index,
1218:                                        const unsigned number_of_bernoullis_b2n,
1219:                                        OutputIterator out_it,
1220:                                        const Policy& pol);
1221:    template <class T, class OutputIterator>
1222:    OutputIterator bernoulli_b2n(const int start_index,
1223:                                        const unsigned number_of_bernoullis_b2n,
1224:                                        OutputIterator out_it);
1225:    template <class T, class Policy>
1226:    T tangent_t2n(const int i, const Policy &pol);
1227:    template <class T>
1228:    T tangent_t2n(const int i);
1229:    template <class T, class OutputIterator, class Policy>
1230:    OutputIterator tangent_t2n(const int start_index,
1231:                                        const unsigned number_of_bernoullis_b2n,
1232:                                        OutputIterator out_it,
````
- **L1205 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1205 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1206 EN**: Executes a call or declaration centered on `relative_difference`.
  - **L1206 CN**: 执行以 `relative_difference` 为核心的调用或声明。
- **L1207 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L1207 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L1208 EN**: Executes a call or declaration centered on `epsilon_difference`.
  - **L1208 CN**: 执行以 `epsilon_difference` 为核心的调用或声明。
- **L1209 EN**: Blank line separating nearby declarations or logic.
  - **L1209 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1210 EN**: Introduces template parameters or specialization context: `template<class T>`.
  - **L1210 CN**: 为后续声明引入模板参数或特化上下文：`template<class T>`。
- **L1211 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1211 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1212 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1212 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1213 EN**: Executes a call or declaration centered on `bernoulli_b2n`.
  - **L1213 CN**: 执行以 `bernoulli_b2n` 为核心的调用或声明。
- **L1214 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1214 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1215 EN**: Executes a call or declaration centered on `bernoulli_b2n`.
  - **L1215 CN**: 执行以 `bernoulli_b2n` 为核心的调用或声明。
- **L1216 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator, class Policy>`.
  - **L1216 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator, class Policy>`。
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator bernoulli_b2n(const int start_index,`.
  - **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator bernoulli_b2n(const int start_index,`。
- **L1218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const unsigned number_of_bernoullis_b2n,`.
  - **L1218 CN**: 继续一个多行参数列表、初始化器或聚合项：`const unsigned number_of_bernoullis_b2n,`。
- **L1219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator out_it,`.
  - **L1219 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator out_it,`。
- **L1220 EN**: Executes a standalone statement or declaration: `const Policy& pol);`.
  - **L1220 CN**: 执行一条独立语句或声明：`const Policy& pol);`。
- **L1221 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>`.
  - **L1221 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>`。
- **L1222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator bernoulli_b2n(const int start_index,`.
  - **L1222 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator bernoulli_b2n(const int start_index,`。
- **L1223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const unsigned number_of_bernoullis_b2n,`.
  - **L1223 CN**: 继续一个多行参数列表、初始化器或聚合项：`const unsigned number_of_bernoullis_b2n,`。
- **L1224 EN**: Executes a standalone statement or declaration: `OutputIterator out_it);`.
  - **L1224 CN**: 执行一条独立语句或声明：`OutputIterator out_it);`。
- **L1225 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1225 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1226 EN**: Executes a call or declaration centered on `tangent_t2n`.
  - **L1226 CN**: 执行以 `tangent_t2n` 为核心的调用或声明。
- **L1227 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1227 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1228 EN**: Executes a call or declaration centered on `tangent_t2n`.
  - **L1228 CN**: 执行以 `tangent_t2n` 为核心的调用或声明。
- **L1229 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator, class Policy>`.
  - **L1229 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator, class Policy>`。
- **L1230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator tangent_t2n(const int start_index,`.
  - **L1230 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator tangent_t2n(const int start_index,`。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const unsigned number_of_bernoullis_b2n,`.
  - **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`const unsigned number_of_bernoullis_b2n,`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator out_it,`.
  - **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator out_it,`。

### Lines 1233-1260 / 第 1233-1260 行

````cpp
1233:                                        const Policy& pol);
1234:    template <class T, class OutputIterator>
1235:    OutputIterator tangent_t2n(const int start_index,
1236:                                        const unsigned number_of_bernoullis_b2n,
1237:                                        OutputIterator out_it);
1238: 
1239:    // Lambert W:
1240:    template <class T, class Policy>
1241:    boost::math::tools::promote_args_t<T> lambert_w0(T z, const Policy& pol);
1242:    template <class T>
1243:    boost::math::tools::promote_args_t<T> lambert_w0(T z);
1244:    template <class T, class Policy>
1245:    boost::math::tools::promote_args_t<T> lambert_wm1(T z, const Policy& pol);
1246:    template <class T>
1247:    boost::math::tools::promote_args_t<T> lambert_wm1(T z);
1248:    template <class T, class Policy>
1249:    boost::math::tools::promote_args_t<T> lambert_w0_prime(T z, const Policy& pol);
1250:    template <class T>
1251:    boost::math::tools::promote_args_t<T> lambert_w0_prime(T z);
1252:    template <class T, class Policy>
1253:    boost::math::tools::promote_args_t<T> lambert_wm1_prime(T z, const Policy& pol);
1254:    template <class T>
1255:    boost::math::tools::promote_args_t<T> lambert_wm1_prime(T z);
1256: 
1257:    // Hypergeometrics:
1258:    template <class T1, class T2> tools::promote_args_t<T1, T2> hypergeometric_1F0(T1 a, T2 z);
1259:    template <class T1, class T2, class Policy> tools::promote_args_t<T1, T2> hypergeometric_1F0(T1 a, T2 z, const Policy&);
1260: 
````
- **L1233 EN**: Executes a standalone statement or declaration: `const Policy& pol);`.
  - **L1233 CN**: 执行一条独立语句或声明：`const Policy& pol);`。
- **L1234 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>`.
  - **L1234 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>`。
- **L1235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OutputIterator tangent_t2n(const int start_index,`.
  - **L1235 CN**: 继续一个多行参数列表、初始化器或聚合项：`OutputIterator tangent_t2n(const int start_index,`。
- **L1236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const unsigned number_of_bernoullis_b2n,`.
  - **L1236 CN**: 继续一个多行参数列表、初始化器或聚合项：`const unsigned number_of_bernoullis_b2n,`。
- **L1237 EN**: Executes a standalone statement or declaration: `OutputIterator out_it);`.
  - **L1237 CN**: 执行一条独立语句或声明：`OutputIterator out_it);`。
- **L1238 EN**: Blank line separating nearby declarations or logic.
  - **L1238 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1239 EN**: Comment documents nearby intent or usage notes: `Lambert W:`.
  - **L1239 CN**: 注释说明附近代码的意图或使用说明：`Lambert W:`。
- **L1240 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1240 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1241 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1241 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1242 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1242 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1243 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1243 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1244 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1244 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1245 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1245 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1246 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1246 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1247 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1247 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1248 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1248 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1249 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1249 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1250 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1250 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1251 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1251 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1252 EN**: Introduces template parameters or specialization context: `template <class T, class Policy>`.
  - **L1252 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class Policy>`。
- **L1253 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1253 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1254 EN**: Introduces template parameters or specialization context: `template <class T>`.
  - **L1254 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>`。
- **L1255 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1255 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1256 EN**: Blank line separating nearby declarations or logic.
  - **L1256 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1257 EN**: Comment documents nearby intent or usage notes: `Hypergeometrics:`.
  - **L1257 CN**: 注释说明附近代码的意图或使用说明：`Hypergeometrics:`。
- **L1258 EN**: Introduces template parameters or specialization context: `template <class T1, class T2> tools::promote_args_t<T1, T2> hypergeometric_1F0(T1 a, T2 z);`.
  - **L1258 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2> tools::promote_args_t<T1, T2> hypergeometric_1F0(T1 a, T2 z);`。
- **L1259 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy> tools::promote_args_t<T1, T2> hypergeometric_1F0(T1 a, T2 z, const Policy&);`.
  - **L1259 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy> tools::promote_args_t<T1, T2> hypergeometric_1F0(T1 a, T2 z, const Policy&);`。
- **L1260 EN**: Blank line separating nearby declarations or logic.
  - **L1260 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 1261-1288 / 第 1261-1288 行

````cpp
1261:    template <class T1, class T2> tools::promote_args_t<T1, T2> hypergeometric_0F1(T1 b, T2 z);
1262:    template <class T1, class T2, class Policy> tools::promote_args_t<T1, T2> hypergeometric_0F1(T1 b, T2 z, const Policy&);
1263: 
1264:    template <class T1, class T2, class T3> tools::promote_args_t<T1, T2, T3> hypergeometric_2F0(T1 a1, T2 a2, T3 z);
1265:    template <class T1, class T2, class T3, class Policy> tools::promote_args_t<T1, T2, T3> hypergeometric_2F0(T1 a1, T2 a2, T3 z, const Policy&);
1266: 
1267:    template <class T1, class T2, class T3> tools::promote_args_t<T1, T2, T3> hypergeometric_1F1(T1 a, T2 b, T3 z);
1268:    template <class T1, class T2, class T3, class Policy> tools::promote_args_t<T1, T2, T3> hypergeometric_1F1(T1 a, T2 b, T3 z, const Policy&);
1269: 
1270: 
1271:     } // namespace math
1272: } // namespace boost
1273: 
1274: #define BOOST_MATH_DETAIL_LL_FUNC(Policy)\
1275:    \
1276:    template <class T>\
1277:    BOOST_MATH_GPU_ENABLED inline T modf(const T& v, long long* ipart){ using boost::math::modf; return modf(v, ipart, Policy()); }\
1278:    \
1279:    template <class T>\
1280:    BOOST_MATH_GPU_ENABLED inline long long lltrunc(const T& v){ using boost::math::lltrunc; return lltrunc(v, Policy()); }\
1281:    \
1282:    template <class T>\
1283:    BOOST_MATH_GPU_ENABLED inline long long llround(const T& v){ using boost::math::llround; return llround(v, Policy()); }\
1284: 
1285: #  define BOOST_MATH_DETAIL_11_FUNC(Policy)\
1286:    template <class T, class U, class V>\
1287:    inline boost::math::tools::promote_args_t<T, U> hypergeometric_1F1(const T& a, const U& b, const V& z)\
1288:    { return boost::math::hypergeometric_1F1(a, b, z, Policy()); }\
````
- **L1261 EN**: Introduces template parameters or specialization context: `template <class T1, class T2> tools::promote_args_t<T1, T2> hypergeometric_0F1(T1 b, T2 z);`.
  - **L1261 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2> tools::promote_args_t<T1, T2> hypergeometric_0F1(T1 b, T2 z);`。
- **L1262 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy> tools::promote_args_t<T1, T2> hypergeometric_0F1(T1 b, T2 z, const Policy&);`.
  - **L1262 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy> tools::promote_args_t<T1, T2> hypergeometric_0F1(T1 b, T2 z, const Policy&);`。
- **L1263 EN**: Blank line separating nearby declarations or logic.
  - **L1263 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1264 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3> tools::promote_args_t<T1, T2, T3> hypergeometric_2F0(T1 a1, T2 a2, T3 z);`.
  - **L1264 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3> tools::promote_args_t<T1, T2, T3> hypergeometric_2F0(T1 a1, T2 a2, T3 z);`。
- **L1265 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class Policy> tools::promote_args_t<T1, T2, T3> hypergeometric_2F0(T1 a1, T2 a2, T3 z, const Policy&);`.
  - **L1265 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class Policy> tools::promote_args_t<T1, T2, T3> hypergeometric_2F0(T1 a1, T2 a2, T3 z, const Policy&);`。
- **L1266 EN**: Blank line separating nearby declarations or logic.
  - **L1266 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1267 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3> tools::promote_args_t<T1, T2, T3> hypergeometric_1F1(T1 a, T2 b, T3 z);`.
  - **L1267 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3> tools::promote_args_t<T1, T2, T3> hypergeometric_1F1(T1 a, T2 b, T3 z);`。
- **L1268 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class Policy> tools::promote_args_t<T1, T2, T3> hypergeometric_1F1(T1 a, T2 b, T3 z, const Policy&);`.
  - **L1268 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class Policy> tools::promote_args_t<T1, T2, T3> hypergeometric_1F1(T1 a, T2 b, T3 z, const Policy&);`。
- **L1269 EN**: Blank line separating nearby declarations or logic.
  - **L1269 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1270 EN**: Blank line separating nearby declarations or logic.
  - **L1270 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1271 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace math`.
  - **L1271 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace math`。
- **L1272 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace boost`.
  - **L1272 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace boost`。
- **L1273 EN**: Blank line separating nearby declarations or logic.
  - **L1273 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1274 EN**: Defines macro `BOOST_MATH_DETAIL_LL_FUNC` for compile-time control, shorthand, or generated boilerplate.
  - **L1274 CN**: 定义宏 `BOOST_MATH_DETAIL_LL_FUNC`，用于编译期控制、简写或生成样板代码。
- **L1275 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1275 CN**: 继续构造周围的表达式或声明：`\`。
- **L1276 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1276 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1277 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1277 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1278 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1278 CN**: 继续构造周围的表达式或声明：`\`。
- **L1279 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1279 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1280 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1280 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1281 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1281 CN**: 继续构造周围的表达式或声明：`\`。
- **L1282 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1282 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1283 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1283 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1284 EN**: Blank line separating nearby declarations or logic.
  - **L1284 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1285 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1285 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1286 EN**: Introduces template parameters or specialization context: `template <class T, class U, class V>\`.
  - **L1286 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class V>\`。
- **L1287 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1287 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1288 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1288 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1289-1316 / 第 1289-1316 行

````cpp
1289: 
1290: #define BOOST_MATH_DECLARE_SPECIAL_FUNCTIONS(Policy)\
1291:    \
1292:    BOOST_MATH_DETAIL_LL_FUNC(Policy)\
1293:    BOOST_MATH_DETAIL_11_FUNC(Policy)\
1294:    \
1295:    template <class RT1, class RT2>\
1296:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2> \
1297:    beta(RT1 a, RT2 b) { return ::boost::math::beta(a, b, Policy()); }\
1298: \
1299:    template <class RT1, class RT2, class A>\
1300:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, A> \
1301:    beta(RT1 a, RT2 b, A x){ return ::boost::math::beta(a, b, x, Policy()); }\
1302: \
1303:    template <class RT1, class RT2, class RT3>\
1304:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
1305:    betac(RT1 a, RT2 b, RT3 x) { return ::boost::math::betac(a, b, x, Policy()); }\
1306: \
1307:    template <class RT1, class RT2, class RT3>\
1308:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
1309:    ibeta(RT1 a, RT2 b, RT3 x){ return ::boost::math::ibeta(a, b, x, Policy()); }\
1310: \
1311:    template <class RT1, class RT2, class RT3>\
1312:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
1313:    ibetac(RT1 a, RT2 b, RT3 x){ return ::boost::math::ibetac(a, b, x, Policy()); }\
1314: \
1315:    template <class T1, class T2, class T3, class T4>\
1316:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3, T4>  \
````
- **L1289 EN**: Blank line separating nearby declarations or logic.
  - **L1289 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1290 EN**: Defines macro `BOOST_MATH_DECLARE_SPECIAL_FUNCTIONS` for compile-time control, shorthand, or generated boilerplate.
  - **L1290 CN**: 定义宏 `BOOST_MATH_DECLARE_SPECIAL_FUNCTIONS`，用于编译期控制、简写或生成样板代码。
- **L1291 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1291 CN**: 继续构造周围的表达式或声明：`\`。
- **L1292 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1292 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1293 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1293 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1294 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1294 CN**: 继续构造周围的表达式或声明：`\`。
- **L1295 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>\`.
  - **L1295 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>\`。
- **L1296 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1296 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1297 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1297 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1298 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1298 CN**: 继续构造周围的表达式或声明：`\`。
- **L1299 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class A>\`.
  - **L1299 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class A>\`。
- **L1300 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1300 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1301 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1301 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1302 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1302 CN**: 继续构造周围的表达式或声明：`\`。
- **L1303 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1303 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1304 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1304 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1305 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1305 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1306 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1306 CN**: 继续构造周围的表达式或声明：`\`。
- **L1307 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1307 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1308 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1308 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1309 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1309 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1310 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1310 CN**: 继续构造周围的表达式或声明：`\`。
- **L1311 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1311 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1312 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1312 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1313 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1313 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1314 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1314 CN**: 继续构造周围的表达式或声明：`\`。
- **L1315 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4>\`.
  - **L1315 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4>\`。
- **L1316 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1316 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1317-1344 / 第 1317-1344 行

````cpp
1317:    ibeta_inv(T1 a, T2 b, T3 p, T4* py){ return ::boost::math::ibeta_inv(a, b, p, py, Policy()); }\
1318: \
1319:    template <class RT1, class RT2, class RT3>\
1320:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
1321:    ibeta_inv(RT1 a, RT2 b, RT3 p){ return ::boost::math::ibeta_inv(a, b, p, Policy()); }\
1322: \
1323:    template <class T1, class T2, class T3, class T4>\
1324:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3, T4> \
1325:    ibetac_inv(T1 a, T2 b, T3 q, T4* py){ return ::boost::math::ibetac_inv(a, b, q, py, Policy()); }\
1326: \
1327:    template <class RT1, class RT2, class RT3>\
1328:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
1329:    ibeta_inva(RT1 a, RT2 b, RT3 p){ return ::boost::math::ibeta_inva(a, b, p, Policy()); }\
1330: \
1331:    template <class T1, class T2, class T3>\
1332:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3> \
1333:    ibetac_inva(T1 a, T2 b, T3 q){ return ::boost::math::ibetac_inva(a, b, q, Policy()); }\
1334: \
1335:    template <class RT1, class RT2, class RT3>\
1336:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
1337:    ibeta_invb(RT1 a, RT2 b, RT3 p){ return ::boost::math::ibeta_invb(a, b, p, Policy()); }\
1338: \
1339:    template <class T1, class T2, class T3>\
1340:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3> \
1341:    ibetac_invb(T1 a, T2 b, T3 q){ return ::boost::math::ibetac_invb(a, b, q, Policy()); }\
1342: \
1343:    template <class RT1, class RT2, class RT3>\
1344:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
````
- **L1317 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1317 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1318 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1318 CN**: 继续构造周围的表达式或声明：`\`。
- **L1319 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1319 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1320 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1320 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1321 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1321 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1322 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1322 CN**: 继续构造周围的表达式或声明：`\`。
- **L1323 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4>\`.
  - **L1323 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4>\`。
- **L1324 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1324 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1325 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1325 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1326 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1326 CN**: 继续构造周围的表达式或声明：`\`。
- **L1327 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1327 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1328 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1328 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1329 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1329 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1330 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1330 CN**: 继续构造周围的表达式或声明：`\`。
- **L1331 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>\`.
  - **L1331 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>\`。
- **L1332 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1332 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1333 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1333 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1334 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1334 CN**: 继续构造周围的表达式或声明：`\`。
- **L1335 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1335 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1336 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1336 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1337 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1337 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1338 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1338 CN**: 继续构造周围的表达式或声明：`\`。
- **L1339 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>\`.
  - **L1339 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>\`。
- **L1340 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1340 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1341 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1341 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1342 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1342 CN**: 继续构造周围的表达式或声明：`\`。
- **L1343 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1343 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1344 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1344 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1345-1372 / 第 1345-1372 行

````cpp
1345:    ibetac_inv(RT1 a, RT2 b, RT3 q){ return ::boost::math::ibetac_inv(a, b, q, Policy()); }\
1346: \
1347:    template <class RT1, class RT2, class RT3>\
1348:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2, RT3> \
1349:    ibeta_derivative(RT1 a, RT2 b, RT3 x){ return ::boost::math::ibeta_derivative(a, b, x, Policy()); }\
1350: \
1351:    template <class T> BOOST_MATH_GPU_ENABLED T binomial_coefficient(unsigned n, unsigned k){ return ::boost::math::binomial_coefficient<T, Policy>(n, k, Policy()); }\
1352: \
1353:    template <class RT>\
1354:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> erf(RT z) { return ::boost::math::erf(z, Policy()); }\
1355: \
1356:    template <class RT>\
1357:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> erfc(RT z){ return ::boost::math::erfc(z, Policy()); }\
1358: \
1359:    template <class RT>\
1360:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> erf_inv(RT z) { return ::boost::math::erf_inv(z, Policy()); }\
1361: \
1362:    template <class RT>\
1363:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> erfc_inv(RT z){ return ::boost::math::erfc_inv(z, Policy()); }\
1364: \
1365:    using boost::math::legendre_next;\
1366: \
1367:    template <class T>\
1368:    inline boost::math::tools::promote_args_t<T> \
1369:    legendre_p(int l, T x){ return ::boost::math::legendre_p(l, x, Policy()); }\
1370: \
1371:    template <class T>\
1372:    inline boost::math::tools::promote_args_t<T> \
````
- **L1345 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1345 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1346 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1346 CN**: 继续构造周围的表达式或声明：`\`。
- **L1347 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2, class RT3>\`.
  - **L1347 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2, class RT3>\`。
- **L1348 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1348 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1349 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1349 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1350 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1350 CN**: 继续构造周围的表达式或声明：`\`。
- **L1351 EN**: Introduces template parameters or specialization context: `template <class T> BOOST_MATH_GPU_ENABLED T binomial_coefficient(unsigned n, unsigned k){ return ::boost::math::binomial_coefficient<T, Policy>(n, k, Policy()); }\`.
  - **L1351 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> BOOST_MATH_GPU_ENABLED T binomial_coefficient(unsigned n, unsigned k){ return ::boost::math::binomial_coefficient<T, Policy>(n, k, Policy()); }\`。
- **L1352 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1352 CN**: 继续构造周围的表达式或声明：`\`。
- **L1353 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1353 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1354 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1354 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1355 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1355 CN**: 继续构造周围的表达式或声明：`\`。
- **L1356 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1356 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1357 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1357 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1358 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1358 CN**: 继续构造周围的表达式或声明：`\`。
- **L1359 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1359 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1360 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1360 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1361 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1361 CN**: 继续构造周围的表达式或声明：`\`。
- **L1362 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1362 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1363 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1363 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1364 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1364 CN**: 继续构造周围的表达式或声明：`\`。
- **L1365 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1365 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1366 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1366 CN**: 继续构造周围的表达式或声明：`\`。
- **L1367 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1367 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1368 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1368 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1369 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1369 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1370 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1370 CN**: 继续构造周围的表达式或声明：`\`。
- **L1371 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1371 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1372 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1372 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1373-1400 / 第 1373-1400 行

````cpp
1373:    legendre_p_prime(int l, T x){ return ::boost::math::legendre_p(l, x, Policy()); }\
1374: \
1375:    template <class T>\
1376:    inline boost::math::tools::promote_args_t<T> \
1377:    legendre_q(unsigned l, T x){ return ::boost::math::legendre_q(l, x, Policy()); }\
1378: \
1379:    using ::boost::math::legendre_next;\
1380: \
1381:    template <class T>\
1382:    inline boost::math::tools::promote_args_t<T> \
1383:    legendre_p(int l, int m, T x){ return ::boost::math::legendre_p(l, m, x, Policy()); }\
1384: \
1385:    using ::boost::math::laguerre_next;\
1386: \
1387:    template <class T>\
1388:    inline boost::math::tools::promote_args_t<T> \
1389:    laguerre(unsigned n, T x){ return ::boost::math::laguerre(n, x, Policy()); }\
1390: \
1391:    template <class T1, class T2>\
1392:    inline typename boost::math::laguerre_result<T1, T2>::type \
1393:    laguerre(unsigned n, T1 m, T2 x) { return ::boost::math::laguerre(n, m, x, Policy()); }\
1394: \
1395:    template <class T>\
1396:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> \
1397:    hermite(unsigned n, T x){ return ::boost::math::hermite(n, x, Policy()); }\
1398: \
1399:    using boost::math::hermite_next;\
1400: \
````
- **L1373 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1373 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1374 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1374 CN**: 继续构造周围的表达式或声明：`\`。
- **L1375 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1375 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1376 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1376 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1377 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1377 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1378 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1378 CN**: 继续构造周围的表达式或声明：`\`。
- **L1379 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1379 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1380 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1380 CN**: 继续构造周围的表达式或声明：`\`。
- **L1381 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1381 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1382 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1382 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1383 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1383 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1384 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1384 CN**: 继续构造周围的表达式或声明：`\`。
- **L1385 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1385 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1386 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1386 CN**: 继续构造周围的表达式或声明：`\`。
- **L1387 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1387 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1388 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1388 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1389 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1389 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1390 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1390 CN**: 继续构造周围的表达式或声明：`\`。
- **L1391 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1391 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1392 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1392 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1393 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1393 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1394 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1394 CN**: 继续构造周围的表达式或声明：`\`。
- **L1395 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1395 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1396 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1396 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1397 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1397 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1398 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1398 CN**: 继续构造周围的表达式或声明：`\`。
- **L1399 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1399 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1400 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1400 CN**: 继续构造周围的表达式或声明：`\`。

### Lines 1401-1428 / 第 1401-1428 行

````cpp
1401:    using boost::math::chebyshev_next;\
1402: \
1403:   template<class Real>\
1404:   Real chebyshev_t(unsigned n, Real const & x){ return ::boost::math::chebyshev_t(n, x, Policy()); }\
1405: \
1406:   template<class Real>\
1407:   Real chebyshev_u(unsigned n, Real const & x){ return ::boost::math::chebyshev_u(n, x, Policy()); }\
1408: \
1409:   template<class Real>\
1410:   Real chebyshev_t_prime(unsigned n, Real const & x){ return ::boost::math::chebyshev_t_prime(n, x, Policy()); }\
1411: \
1412:   using ::boost::math::chebyshev_clenshaw_recurrence;\
1413: \
1414:    template <class T1, class T2>\
1415:    inline std::complex<boost::math::tools::promote_args_t<T1, T2>> \
1416:    spherical_harmonic(unsigned n, int m, T1 theta, T2 phi){ return boost::math::spherical_harmonic(n, m, theta, phi, Policy()); }\
1417: \
1418:    template <class T1, class T2>\
1419:    inline boost::math::tools::promote_args_t<T1, T2> \
1420:    spherical_harmonic_r(unsigned n, int m, T1 theta, T2 phi){ return ::boost::math::spherical_harmonic_r(n, m, theta, phi, Policy()); }\
1421: \
1422:    template <class T1, class T2>\
1423:    inline boost::math::tools::promote_args_t<T1, T2> \
1424:    spherical_harmonic_i(unsigned n, int m, T1 theta, T2 phi){ return boost::math::spherical_harmonic_i(n, m, theta, phi, Policy()); }\
1425: \
1426:    template <class T1, class T2, class Policy>\
1427:    inline boost::math::tools::promote_args_t<T1, T2> \
1428:       spherical_harmonic_i(unsigned n, int m, T1 theta, T2 phi, const Policy& pol);\
````
- **L1401 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1401 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1402 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1402 CN**: 继续构造周围的表达式或声明：`\`。
- **L1403 EN**: Introduces template parameters or specialization context: `template<class Real>\`.
  - **L1403 CN**: 为后续声明引入模板参数或特化上下文：`template<class Real>\`。
- **L1404 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1404 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1405 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1405 CN**: 继续构造周围的表达式或声明：`\`。
- **L1406 EN**: Introduces template parameters or specialization context: `template<class Real>\`.
  - **L1406 CN**: 为后续声明引入模板参数或特化上下文：`template<class Real>\`。
- **L1407 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1407 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1408 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1408 CN**: 继续构造周围的表达式或声明：`\`。
- **L1409 EN**: Introduces template parameters or specialization context: `template<class Real>\`.
  - **L1409 CN**: 为后续声明引入模板参数或特化上下文：`template<class Real>\`。
- **L1410 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1410 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1411 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1411 CN**: 继续构造周围的表达式或声明：`\`。
- **L1412 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1412 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1413 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1413 CN**: 继续构造周围的表达式或声明：`\`。
- **L1414 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1414 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1415 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1415 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1416 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1416 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1417 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1417 CN**: 继续构造周围的表达式或声明：`\`。
- **L1418 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1418 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1419 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1419 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1420 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1420 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1421 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1421 CN**: 继续构造周围的表达式或声明：`\`。
- **L1422 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1422 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1423 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1423 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1424 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1424 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1425 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1425 CN**: 继续构造周围的表达式或声明：`\`。
- **L1426 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class Policy>\`.
  - **L1426 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class Policy>\`。
- **L1427 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1427 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1428 EN**: Continues logic associated with callable symbol `spherical_harmonic_i`.
  - **L1428 CN**: 继续与可调用符号 `spherical_harmonic_i` 相关的逻辑。

### Lines 1429-1456 / 第 1429-1456 行

````cpp
1429: \
1430:    template <class T1, class T2, class T3>\
1431:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3> \
1432:    ellint_rf(T1 x, T2 y, T3 z){ return ::boost::math::ellint_rf(x, y, z, Policy()); }\
1433: \
1434:    template <class T1, class T2, class T3>\
1435:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3> \
1436:    ellint_rd(T1 x, T2 y, T3 z){ return ::boost::math::ellint_rd(x, y, z, Policy()); }\
1437: \
1438:    template <class T1, class T2>\
1439:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> \
1440:    ellint_rc(T1 x, T2 y){ return ::boost::math::ellint_rc(x, y, Policy()); }\
1441: \
1442:    template <class T1, class T2, class T3, class T4>\
1443:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3, T4> \
1444:    ellint_rj(T1 x, T2 y, T3 z, T4 p){ return boost::math::ellint_rj(x, y, z, p, Policy()); }\
1445: \
1446:    template <class T1, class T2, class T3>\
1447:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3> \
1448:    ellint_rg(T1 x, T2 y, T3 z){ return ::boost::math::ellint_rg(x, y, z, Policy()); }\
1449:    \
1450:    template <typename T>\
1451:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> ellint_2(T k){ return boost::math::ellint_2(k, Policy()); }\
1452: \
1453:    template <class T1, class T2>\
1454:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> ellint_2(T1 k, T2 phi){ return boost::math::ellint_2(k, phi, Policy()); }\
1455: \
1456:    template <typename T>\
````
- **L1429 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1429 CN**: 继续构造周围的表达式或声明：`\`。
- **L1430 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>\`.
  - **L1430 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>\`。
- **L1431 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1431 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1432 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1432 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1433 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1433 CN**: 继续构造周围的表达式或声明：`\`。
- **L1434 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>\`.
  - **L1434 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>\`。
- **L1435 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1435 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1436 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1436 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1437 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1437 CN**: 继续构造周围的表达式或声明：`\`。
- **L1438 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1438 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1439 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1439 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1440 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1440 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1441 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1441 CN**: 继续构造周围的表达式或声明：`\`。
- **L1442 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3, class T4>\`.
  - **L1442 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3, class T4>\`。
- **L1443 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1443 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1444 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1444 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1445 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1445 CN**: 继续构造周围的表达式或声明：`\`。
- **L1446 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>\`.
  - **L1446 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>\`。
- **L1447 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1447 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1448 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1448 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1449 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1449 CN**: 继续构造周围的表达式或声明：`\`。
- **L1450 EN**: Introduces template parameters or specialization context: `template <typename T>\`.
  - **L1450 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>\`。
- **L1451 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1451 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1452 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1452 CN**: 继续构造周围的表达式或声明：`\`。
- **L1453 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1453 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1454 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1454 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1455 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1455 CN**: 继续构造周围的表达式或声明：`\`。
- **L1456 EN**: Introduces template parameters or specialization context: `template <typename T>\`.
  - **L1456 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>\`。

### Lines 1457-1484 / 第 1457-1484 行

````cpp
1457:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> ellint_d(T k){ return boost::math::ellint_d(k, Policy()); }\
1458: \
1459:    template <class T1, class T2>\
1460:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> ellint_d(T1 k, T2 phi){ return boost::math::ellint_d(k, phi, Policy()); }\
1461: \
1462:    template <class T1, class T2>\
1463:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> jacobi_zeta(T1 k, T2 phi){ return boost::math::jacobi_zeta(k, phi, Policy()); }\
1464: \
1465:    template <class T1, class T2>\
1466:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> heuman_lambda(T1 k, T2 phi){ return boost::math::heuman_lambda(k, phi, Policy()); }\
1467: \
1468:    template <typename T>\
1469:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> ellint_1(T k){ return boost::math::ellint_1(k, Policy()); }\
1470: \
1471:    template <class T1, class T2>\
1472:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> ellint_1(T1 k, T2 phi){ return boost::math::ellint_1(k, phi, Policy()); }\
1473: \
1474:    template <class T1, class T2, class T3>\
1475:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2, T3> ellint_3(T1 k, T2 v, T3 phi){ return boost::math::ellint_3(k, v, phi, Policy()); }\
1476: \
1477:    template <class T1, class T2>\
1478:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> ellint_3(T1 k, T2 v){ return boost::math::ellint_3(k, v, Policy()); }\
1479: \
1480:    using boost::math::max_factorial;\
1481:    template <class RT>\
1482:    BOOST_MATH_GPU_ENABLED inline RT factorial(unsigned int i) { return boost::math::factorial<RT>(i, Policy()); }\
1483:    using boost::math::unchecked_factorial;\
1484:    template <class RT>\
````
- **L1457 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1457 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1458 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1458 CN**: 继续构造周围的表达式或声明：`\`。
- **L1459 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1459 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1460 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1460 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1461 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1461 CN**: 继续构造周围的表达式或声明：`\`。
- **L1462 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1462 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1463 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1463 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1464 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1464 CN**: 继续构造周围的表达式或声明：`\`。
- **L1465 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1465 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1466 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1466 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1467 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1467 CN**: 继续构造周围的表达式或声明：`\`。
- **L1468 EN**: Introduces template parameters or specialization context: `template <typename T>\`.
  - **L1468 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T>\`。
- **L1469 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1469 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1470 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1470 CN**: 继续构造周围的表达式或声明：`\`。
- **L1471 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1471 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1472 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1472 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1473 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1473 CN**: 继续构造周围的表达式或声明：`\`。
- **L1474 EN**: Introduces template parameters or specialization context: `template <class T1, class T2, class T3>\`.
  - **L1474 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2, class T3>\`。
- **L1475 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1475 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1476 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1476 CN**: 继续构造周围的表达式或声明：`\`。
- **L1477 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1477 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1478 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1478 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1479 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1479 CN**: 继续构造周围的表达式或声明：`\`。
- **L1480 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1480 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1481 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1481 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1482 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1482 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1483 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1483 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1484 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1484 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。

### Lines 1485-1512 / 第 1485-1512 行

````cpp
1485:    BOOST_MATH_GPU_ENABLED inline RT double_factorial(unsigned i){ return boost::math::double_factorial<RT>(i, Policy()); }\
1486:    template <class RT>\
1487:    inline boost::math::tools::promote_args_t<RT> falling_factorial(RT x, unsigned n){ return boost::math::falling_factorial(x, n, Policy()); }\
1488:    template <class RT>\
1489:    inline boost::math::tools::promote_args_t<RT> rising_factorial(RT x, unsigned n){ return boost::math::rising_factorial(x, n, Policy()); }\
1490: \
1491:    template <class RT>\
1492:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> tgamma(RT z){ return boost::math::tgamma(z, Policy()); }\
1493: \
1494:    template <class RT>\
1495:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> tgamma1pm1(RT z){ return boost::math::tgamma1pm1(z, Policy()); }\
1496: \
1497:    template <class RT1, class RT2>\
1498:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2> tgamma(RT1 a, RT2 z){ return boost::math::tgamma(a, z, Policy()); }\
1499: \
1500:    template <class RT>\
1501:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> lgamma(RT z, int* sign){ return boost::math::lgamma(z, sign, Policy()); }\
1502: \
1503:    template <class RT>\
1504:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT> lgamma(RT x){ return boost::math::lgamma(x, Policy()); }\
1505: \
1506:    template <class RT1, class RT2>\
1507:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2> tgamma_lower(RT1 a, RT2 z){ return boost::math::tgamma_lower(a, z, Policy()); }\
1508: \
1509:    template <class RT1, class RT2>\
1510:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2> gamma_q(RT1 a, RT2 z){ return boost::math::gamma_q(a, z, Policy()); }\
1511: \
1512:    template <class RT1, class RT2>\
````
- **L1485 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1485 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1486 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1486 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1487 EN**: Starts a function or method definition for `falling_factorial`.
  - **L1487 CN**: 开始定义函数或方法 `falling_factorial`。
- **L1488 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1488 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1489 EN**: Starts a function or method definition for `rising_factorial`.
  - **L1489 CN**: 开始定义函数或方法 `rising_factorial`。
- **L1490 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1490 CN**: 继续构造周围的表达式或声明：`\`。
- **L1491 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1491 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1492 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1492 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1493 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1493 CN**: 继续构造周围的表达式或声明：`\`。
- **L1494 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1494 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1495 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1495 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1496 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1496 CN**: 继续构造周围的表达式或声明：`\`。
- **L1497 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>\`.
  - **L1497 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>\`。
- **L1498 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1498 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1499 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1499 CN**: 继续构造周围的表达式或声明：`\`。
- **L1500 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1500 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1501 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1501 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1502 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1502 CN**: 继续构造周围的表达式或声明：`\`。
- **L1503 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1503 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1504 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1504 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1505 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1505 CN**: 继续构造周围的表达式或声明：`\`。
- **L1506 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>\`.
  - **L1506 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>\`。
- **L1507 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1507 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1508 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1508 CN**: 继续构造周围的表达式或声明：`\`。
- **L1509 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>\`.
  - **L1509 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>\`。
- **L1510 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1510 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1511 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1511 CN**: 继续构造周围的表达式或声明：`\`。
- **L1512 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>\`.
  - **L1512 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>\`。

### Lines 1513-1540 / 第 1513-1540 行

````cpp
1513:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<RT1, RT2> gamma_p(RT1 a, RT2 z){ return boost::math::gamma_p(a, z, Policy()); }\
1514: \
1515:    template <class T1, class T2>\
1516:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> tgamma_delta_ratio(T1 z, T2 delta){ return boost::math::tgamma_delta_ratio(z, delta, Policy()); }\
1517: \
1518:    template <class T1, class T2>\
1519:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> tgamma_ratio(T1 a, T2 b) { return boost::math::tgamma_ratio(a, b, Policy()); }\
1520: \
1521:    template <class T1, class T2>\
1522:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> gamma_p_derivative(T1 a, T2 x){ return boost::math::gamma_p_derivative(a, x, Policy()); }\
1523: \
1524:    template <class T1, class T2>\
1525:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> gamma_p_inv(T1 a, T2 p){ return boost::math::gamma_p_inv(a, p, Policy()); }\
1526: \
1527:    template <class T1, class T2>\
1528:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> gamma_p_inva(T1 a, T2 p){ return boost::math::gamma_p_inva(a, p, Policy()); }\
1529: \
1530:    template <class T1, class T2>\
1531:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> gamma_q_inv(T1 a, T2 q){ return boost::math::gamma_q_inv(a, q, Policy()); }\
1532: \
1533:    template <class T1, class T2>\
1534:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T1, T2> gamma_q_inva(T1 a, T2 q){ return boost::math::gamma_q_inva(a, q, Policy()); }\
1535: \
1536:    template <class T>\
1537:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> digamma(T x){ return boost::math::digamma(x, Policy()); }\
1538: \
1539:    template <class T>\
1540:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> trigamma(T x){ return boost::math::trigamma(x, Policy()); }\
````
- **L1513 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1513 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1514 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1514 CN**: 继续构造周围的表达式或声明：`\`。
- **L1515 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1515 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1516 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1516 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1517 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1517 CN**: 继续构造周围的表达式或声明：`\`。
- **L1518 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1518 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1519 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1519 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1520 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1520 CN**: 继续构造周围的表达式或声明：`\`。
- **L1521 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1521 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1522 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1522 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1523 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1523 CN**: 继续构造周围的表达式或声明：`\`。
- **L1524 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1524 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1525 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1525 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1526 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1526 CN**: 继续构造周围的表达式或声明：`\`。
- **L1527 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1527 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1528 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1528 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1529 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1529 CN**: 继续构造周围的表达式或声明：`\`。
- **L1530 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1530 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1531 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1531 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1532 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1532 CN**: 继续构造周围的表达式或声明：`\`。
- **L1533 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1533 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1534 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1534 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1535 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1535 CN**: 继续构造周围的表达式或声明：`\`。
- **L1536 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1536 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1537 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1537 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1538 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1538 CN**: 继续构造周围的表达式或声明：`\`。
- **L1539 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1539 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1540 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1540 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1541-1568 / 第 1541-1568 行

````cpp
1541: \
1542:    template <class T>\
1543:    inline boost::math::tools::promote_args_t<T> polygamma(int n, T x){ return boost::math::polygamma(n, x, Policy()); }\
1544:    \
1545:    template <class T1, class T2>\
1546:    inline boost::math::tools::promote_args_t<T1, T2> \
1547:    BOOST_MATH_GPU_ENABLED hypot(T1 x, T2 y){ return boost::math::hypot(x, y, Policy()); }\
1548: \
1549:    template <class RT>\
1550:    inline boost::math::tools::promote_args_t<RT> cbrt(RT z){ return boost::math::cbrt(z, Policy()); }\
1551: \
1552:    template <class T>\
1553:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> log1p(T x){ return boost::math::log1p(x, Policy()); }\
1554: \
1555:    template <class T>\
1556:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> log1pmx(T x){ return boost::math::log1pmx(x, Policy()); }\
1557: \
1558:    template <class T>\
1559:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> expm1(T x){ return boost::math::expm1(x, Policy()); }\
1560: \
1561:    template <class T1, class T2>\
1562:    inline boost::math::tools::promote_args_t<T1, T2> \
1563:    BOOST_MATH_GPU_ENABLED powm1(const T1 a, const T2 z){ return boost::math::powm1(a, z, Policy()); }\
1564: \
1565:    template <class T>\
1566:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> sqrt1pm1(const T& val){ return boost::math::sqrt1pm1(val, Policy()); }\
1567: \
1568:    template <class T>\
````
- **L1541 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1541 CN**: 继续构造周围的表达式或声明：`\`。
- **L1542 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1542 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1543 EN**: Starts a function or method definition for `polygamma`.
  - **L1543 CN**: 开始定义函数或方法 `polygamma`。
- **L1544 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1544 CN**: 继续构造周围的表达式或声明：`\`。
- **L1545 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1545 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1546 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1546 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1547 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1547 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1548 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1548 CN**: 继续构造周围的表达式或声明：`\`。
- **L1549 EN**: Introduces template parameters or specialization context: `template <class RT>\`.
  - **L1549 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT>\`。
- **L1550 EN**: Starts a function or method definition for `cbrt`.
  - **L1550 CN**: 开始定义函数或方法 `cbrt`。
- **L1551 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1551 CN**: 继续构造周围的表达式或声明：`\`。
- **L1552 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1552 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1553 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1553 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1554 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1554 CN**: 继续构造周围的表达式或声明：`\`。
- **L1555 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1555 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1556 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1556 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1557 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1557 CN**: 继续构造周围的表达式或声明：`\`。
- **L1558 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1558 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1559 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1559 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1560 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1560 CN**: 继续构造周围的表达式或声明：`\`。
- **L1561 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1561 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1562 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1562 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1563 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1563 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1564 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1564 CN**: 继续构造周围的表达式或声明：`\`。
- **L1565 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1565 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1566 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1566 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1567 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1567 CN**: 继续构造周围的表达式或声明：`\`。
- **L1568 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1568 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。

### Lines 1569-1596 / 第 1569-1596 行

````cpp
1569:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> sinc_pi(T x){ return boost::math::sinc_pi(x, Policy()); }\
1570: \
1571:    template <class T>\
1572:    inline boost::math::tools::promote_args_t<T> sinhc_pi(T x){ return boost::math::sinhc_pi(x, Policy()); }\
1573: \
1574:    template<typename T>\
1575:    inline boost::math::tools::promote_args_t<T> asinh(const T x){ return boost::math::asinh(x, Policy()); }\
1576: \
1577:    template<typename T>\
1578:    inline boost::math::tools::promote_args_t<T> acosh(const T x){ return boost::math::acosh(x, Policy()); }\
1579: \
1580:    template<typename T>\
1581:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> atanh(const T x){ return boost::math::atanh(x, Policy()); }\
1582: \
1583:    template <class T1, class T2>\
1584:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type cyl_bessel_j(T1 v, T2 x)\
1585:    { return boost::math::cyl_bessel_j(v, x, Policy()); }\
1586: \
1587:    template <class T1, class T2>\
1588:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type cyl_bessel_j_prime(T1 v, T2 x)\
1589:    { return boost::math::cyl_bessel_j_prime(v, x, Policy()); }\
1590: \
1591:    template <class T>\
1592:    inline typename boost::math::detail::bessel_traits<T, T, Policy >::result_type sph_bessel(unsigned v, T x)\
1593:    { return boost::math::sph_bessel(v, x, Policy()); }\
1594: \
1595:    template <class T>\
1596:    inline typename boost::math::detail::bessel_traits<T, T, Policy >::result_type sph_bessel_prime(unsigned v, T x)\
````
- **L1569 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1569 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1570 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1570 CN**: 继续构造周围的表达式或声明：`\`。
- **L1571 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1571 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1572 EN**: Starts a function or method definition for `sinhc_pi`.
  - **L1572 CN**: 开始定义函数或方法 `sinhc_pi`。
- **L1573 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1573 CN**: 继续构造周围的表达式或声明：`\`。
- **L1574 EN**: Introduces template parameters or specialization context: `template<typename T>\`.
  - **L1574 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T>\`。
- **L1575 EN**: Starts a function or method definition for `asinh`.
  - **L1575 CN**: 开始定义函数或方法 `asinh`。
- **L1576 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1576 CN**: 继续构造周围的表达式或声明：`\`。
- **L1577 EN**: Introduces template parameters or specialization context: `template<typename T>\`.
  - **L1577 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T>\`。
- **L1578 EN**: Starts a function or method definition for `acosh`.
  - **L1578 CN**: 开始定义函数或方法 `acosh`。
- **L1579 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1579 CN**: 继续构造周围的表达式或声明：`\`。
- **L1580 EN**: Introduces template parameters or specialization context: `template<typename T>\`.
  - **L1580 CN**: 为后续声明引入模板参数或特化上下文：`template<typename T>\`。
- **L1581 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1581 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1582 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1582 CN**: 继续构造周围的表达式或声明：`\`。
- **L1583 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1583 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1584 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1584 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1585 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1585 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1586 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1586 CN**: 继续构造周围的表达式或声明：`\`。
- **L1587 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1587 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1588 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1588 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1589 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1589 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1590 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1590 CN**: 继续构造周围的表达式或声明：`\`。
- **L1591 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1591 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1592 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1592 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1593 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1593 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1594 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1594 CN**: 继续构造周围的表达式或声明：`\`。
- **L1595 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1595 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1596 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1596 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1597-1624 / 第 1597-1624 行

````cpp
1597:    { return boost::math::sph_bessel_prime(v, x, Policy()); }\
1598: \
1599:    template <class T1, class T2>\
1600:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type \
1601:    cyl_bessel_i(T1 v, T2 x) { return boost::math::cyl_bessel_i(v, x, Policy()); }\
1602: \
1603:    template <class T1, class T2>\
1604:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type \
1605:    cyl_bessel_i_prime(T1 v, T2 x) { return boost::math::cyl_bessel_i_prime(v, x, Policy()); }\
1606: \
1607:    template <class T1, class T2>\
1608:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type \
1609:    cyl_bessel_k(T1 v, T2 x) { return boost::math::cyl_bessel_k(v, x, Policy()); }\
1610: \
1611:    template <class T1, class T2>\
1612:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type \
1613:    cyl_bessel_k_prime(T1 v, T2 x) { return boost::math::cyl_bessel_k_prime(v, x, Policy()); }\
1614: \
1615:    template <class T1, class T2>\
1616:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type \
1617:    cyl_neumann(T1 v, T2 x){ return boost::math::cyl_neumann(v, x, Policy()); }\
1618: \
1619:    template <class T1, class T2>\
1620:    inline typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type \
1621:    cyl_neumann_prime(T1 v, T2 x){ return boost::math::cyl_neumann_prime(v, x, Policy()); }\
1622: \
1623:    template <class T>\
1624:    inline typename boost::math::detail::bessel_traits<T, T, Policy >::result_type \
````
- **L1597 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1597 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1598 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1598 CN**: 继续构造周围的表达式或声明：`\`。
- **L1599 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1599 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1600 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1600 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1601 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1601 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1602 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1602 CN**: 继续构造周围的表达式或声明：`\`。
- **L1603 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1603 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1604 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1604 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1605 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1605 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1606 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1606 CN**: 继续构造周围的表达式或声明：`\`。
- **L1607 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1607 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1608 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1608 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1609 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1609 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1610 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1610 CN**: 继续构造周围的表达式或声明：`\`。
- **L1611 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1611 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1612 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1612 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1613 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1613 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1614 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1614 CN**: 继续构造周围的表达式或声明：`\`。
- **L1615 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1615 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1616 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1616 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1617 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1617 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1618 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1618 CN**: 继续构造周围的表达式或声明：`\`。
- **L1619 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1619 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1620 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1620 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1621 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1621 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1622 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1622 CN**: 继续构造周围的表达式或声明：`\`。
- **L1623 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1623 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1624 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1624 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1625-1652 / 第 1625-1652 行

````cpp
1625:    sph_neumann(unsigned v, T x){ return boost::math::sph_neumann(v, x, Policy()); }\
1626: \
1627:    template <class T>\
1628:    inline typename boost::math::detail::bessel_traits<T, T, Policy >::result_type \
1629:    sph_neumann_prime(unsigned v, T x){ return boost::math::sph_neumann_prime(v, x, Policy()); }\
1630: \
1631:    template <class T>\
1632:    inline typename boost::math::detail::bessel_traits<T, T, Policy >::result_type cyl_bessel_j_zero(T v, int m)\
1633:    { return boost::math::cyl_bessel_j_zero(v, m, Policy()); }\
1634: \
1635: template <class OutputIterator, class T>\
1636:    inline void cyl_bessel_j_zero(T v,\
1637:                                  int start_index,\
1638:                                  unsigned number_of_zeros,\
1639:                                  OutputIterator out_it)\
1640:    { boost::math::cyl_bessel_j_zero(v, start_index, number_of_zeros, out_it, Policy()); }\
1641: \
1642:    template <class T>\
1643:    inline typename boost::math::detail::bessel_traits<T, T, Policy >::result_type cyl_neumann_zero(T v, int m)\
1644:    { return boost::math::cyl_neumann_zero(v, m, Policy()); }\
1645: \
1646: template <class OutputIterator, class T>\
1647:    inline void cyl_neumann_zero(T v,\
1648:                                 int start_index,\
1649:                                 unsigned number_of_zeros,\
1650:                                 OutputIterator out_it)\
1651:    { boost::math::cyl_neumann_zero(v, start_index, number_of_zeros, out_it, Policy()); }\
1652: \
````
- **L1625 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1625 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1626 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1626 CN**: 继续构造周围的表达式或声明：`\`。
- **L1627 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1627 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1628 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1628 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1629 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1629 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1630 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1630 CN**: 继续构造周围的表达式或声明：`\`。
- **L1631 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1631 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1632 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1632 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1633 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1633 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1634 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1634 CN**: 继续构造周围的表达式或声明：`\`。
- **L1635 EN**: Introduces template parameters or specialization context: `template <class OutputIterator, class T>\`.
  - **L1635 CN**: 为后续声明引入模板参数或特化上下文：`template <class OutputIterator, class T>\`。
- **L1636 EN**: Continues logic associated with callable symbol `cyl_bessel_j_zero`.
  - **L1636 CN**: 继续与可调用符号 `cyl_bessel_j_zero` 相关的逻辑。
- **L1637 EN**: Continues the surrounding expression or declaration: `int start_index,\`.
  - **L1637 CN**: 继续构造周围的表达式或声明：`int start_index,\`。
- **L1638 EN**: Continues the surrounding expression or declaration: `unsigned number_of_zeros,\`.
  - **L1638 CN**: 继续构造周围的表达式或声明：`unsigned number_of_zeros,\`。
- **L1639 EN**: Continues the surrounding expression or declaration: `OutputIterator out_it)\`.
  - **L1639 CN**: 继续构造周围的表达式或声明：`OutputIterator out_it)\`。
- **L1640 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1640 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1641 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1641 CN**: 继续构造周围的表达式或声明：`\`。
- **L1642 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1642 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1643 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1643 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1644 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1644 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1645 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1645 CN**: 继续构造周围的表达式或声明：`\`。
- **L1646 EN**: Introduces template parameters or specialization context: `template <class OutputIterator, class T>\`.
  - **L1646 CN**: 为后续声明引入模板参数或特化上下文：`template <class OutputIterator, class T>\`。
- **L1647 EN**: Continues logic associated with callable symbol `cyl_neumann_zero`.
  - **L1647 CN**: 继续与可调用符号 `cyl_neumann_zero` 相关的逻辑。
- **L1648 EN**: Continues the surrounding expression or declaration: `int start_index,\`.
  - **L1648 CN**: 继续构造周围的表达式或声明：`int start_index,\`。
- **L1649 EN**: Continues the surrounding expression or declaration: `unsigned number_of_zeros,\`.
  - **L1649 CN**: 继续构造周围的表达式或声明：`unsigned number_of_zeros,\`。
- **L1650 EN**: Continues the surrounding expression or declaration: `OutputIterator out_it)\`.
  - **L1650 CN**: 继续构造周围的表达式或声明：`OutputIterator out_it)\`。
- **L1651 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1651 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1652 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1652 CN**: 继续构造周围的表达式或声明：`\`。

### Lines 1653-1680 / 第 1653-1680 行

````cpp
1653:    template <class T>\
1654:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> sin_pi(T x){ return boost::math::sin_pi(x, Policy()); }\
1655: \
1656:    template <class T>\
1657:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> cos_pi(T x){ return boost::math::cos_pi(x, Policy()); }\
1658: \
1659:    using boost::math::fpclassify;\
1660:    using boost::math::isfinite;\
1661:    using boost::math::isinf;\
1662:    using boost::math::isnan;\
1663:    using boost::math::isnormal;\
1664:    using boost::math::signbit;\
1665:    using boost::math::sign;\
1666:    using boost::math::copysign;\
1667:    using boost::math::changesign;\
1668:    \
1669:    template <class T, class U>\
1670:    BOOST_MATH_GPU_ENABLED inline typename boost::math::tools::promote_args_t<T,U> expint(T const& z, U const& u)\
1671:    { return boost::math::expint(z, u, Policy()); }\
1672:    \
1673:    template <class T>\
1674:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> expint(T z){ return boost::math::expint(z, Policy()); }\
1675:    \
1676:    template <class T>\
1677:    inline boost::math::tools::promote_args_t<T> zeta(T s){ return boost::math::zeta(s, Policy()); }\
1678:    \
1679:    template <class T>\
1680:    BOOST_MATH_GPU_ENABLED inline T round(const T& v){ using boost::math::round; return round(v, Policy()); }\
````
- **L1653 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1653 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1654 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1654 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1655 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1655 CN**: 继续构造周围的表达式或声明：`\`。
- **L1656 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1656 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1657 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1657 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1658 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1658 CN**: 继续构造周围的表达式或声明：`\`。
- **L1659 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1659 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1660 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1660 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1661 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1661 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1662 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1662 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1663 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1663 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1664 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1664 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1665 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1665 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1666 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1666 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1667 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1667 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1668 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1668 CN**: 继续构造周围的表达式或声明：`\`。
- **L1669 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1669 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1670 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1670 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1671 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1671 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1672 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1672 CN**: 继续构造周围的表达式或声明：`\`。
- **L1673 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1673 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1674 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1674 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1675 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1675 CN**: 继续构造周围的表达式或声明：`\`。
- **L1676 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1676 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1677 EN**: Starts a function or method definition for `zeta`.
  - **L1677 CN**: 开始定义函数或方法 `zeta`。
- **L1678 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1678 CN**: 继续构造周围的表达式或声明：`\`。
- **L1679 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1679 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1680 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1680 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 1681-1708 / 第 1681-1708 行

````cpp
1681:    \
1682:    template <class T>\
1683:    BOOST_MATH_GPU_ENABLED inline int iround(const T& v){ using boost::math::iround; return iround(v, Policy()); }\
1684:    \
1685:    template <class T>\
1686:    BOOST_MATH_GPU_ENABLED inline long lround(const T& v){ using boost::math::lround; return lround(v, Policy()); }\
1687:    \
1688:    template <class T>\
1689:    BOOST_MATH_GPU_ENABLED inline T trunc(const T& v){ using boost::math::trunc; return trunc(v, Policy()); }\
1690:    \
1691:    template <class T>\
1692:    BOOST_MATH_GPU_ENABLED inline int itrunc(const T& v){ using boost::math::itrunc; return itrunc(v, Policy()); }\
1693:    \
1694:    template <class T>\
1695:    BOOST_MATH_GPU_ENABLED inline long ltrunc(const T& v){ using boost::math::ltrunc; return ltrunc(v, Policy()); }\
1696:    \
1697:    template <class T>\
1698:    BOOST_MATH_GPU_ENABLED inline T modf(const T& v, T* ipart){ using boost::math::modf; return modf(v, ipart, Policy()); }\
1699:    \
1700:    template <class T>\
1701:    BOOST_MATH_GPU_ENABLED inline T modf(const T& v, int* ipart){ using boost::math::modf; return modf(v, ipart, Policy()); }\
1702:    \
1703:    template <class T>\
1704:    BOOST_MATH_GPU_ENABLED inline T modf(const T& v, long* ipart){ using boost::math::modf; return modf(v, ipart, Policy()); }\
1705:    \
1706:    template <int N, class T>\
1707:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> pow(T v){ return boost::math::pow<N>(v, Policy()); }\
1708:    \
````
- **L1681 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1681 CN**: 继续构造周围的表达式或声明：`\`。
- **L1682 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1682 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1683 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1683 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1684 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1684 CN**: 继续构造周围的表达式或声明：`\`。
- **L1685 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1685 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1686 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1686 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1687 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1687 CN**: 继续构造周围的表达式或声明：`\`。
- **L1688 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1688 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1689 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1689 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1690 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1690 CN**: 继续构造周围的表达式或声明：`\`。
- **L1691 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1691 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1692 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1692 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1693 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1693 CN**: 继续构造周围的表达式或声明：`\`。
- **L1694 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1694 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1695 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1695 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1696 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1696 CN**: 继续构造周围的表达式或声明：`\`。
- **L1697 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1697 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1698 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1698 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1699 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1699 CN**: 继续构造周围的表达式或声明：`\`。
- **L1700 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1700 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1701 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1701 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1702 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1702 CN**: 继续构造周围的表达式或声明：`\`。
- **L1703 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1703 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1704 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1704 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1705 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1705 CN**: 继续构造周围的表达式或声明：`\`。
- **L1706 EN**: Introduces template parameters or specialization context: `template <int N, class T>\`.
  - **L1706 CN**: 为后续声明引入模板参数或特化上下文：`template <int N, class T>\`。
- **L1707 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1707 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1708 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1708 CN**: 继续构造周围的表达式或声明：`\`。

### Lines 1709-1736 / 第 1709-1736 行

````cpp
1709:    template <class T> T nextafter(const T& a, const T& b){ return static_cast<T>(boost::math::nextafter(a, b, Policy())); }\
1710:    template <class T> T float_next(const T& a){ return static_cast<T>(boost::math::float_next(a, Policy())); }\
1711:    template <class T> T float_prior(const T& a){ return static_cast<T>(boost::math::float_prior(a, Policy())); }\
1712:    template <class T> T float_distance(const T& a, const T& b){ return static_cast<T>(boost::math::float_distance(a, b, Policy())); }\
1713:    template <class T> T ulp(const T& a){ return static_cast<T>(boost::math::ulp(a, Policy())); }\
1714:    \
1715:    template <class RT1, class RT2>\
1716:    inline boost::math::tools::promote_args_t<RT1, RT2> owens_t(RT1 a, RT2 z){ return boost::math::owens_t(a, z, Policy()); }\
1717:    \
1718:    template <class T1, class T2>\
1719:    BOOST_MATH_GPU_ENABLED inline boost::math::complex<typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type> cyl_hankel_1(T1 v, T2 x)\
1720:    {  return boost::math::cyl_hankel_1(v, x, Policy()); }\
1721:    \
1722:    template <class T1, class T2>\
1723:    BOOST_MATH_GPU_ENABLED inline boost::math::complex<typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type> cyl_hankel_2(T1 v, T2 x)\
1724:    { return boost::math::cyl_hankel_2(v, x, Policy()); }\
1725:    \
1726:    template <class T1, class T2>\
1727:    BOOST_MATH_GPU_ENABLED inline boost::math::complex<typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type> sph_hankel_1(T1 v, T2 x)\
1728:    { return boost::math::sph_hankel_1(v, x, Policy()); }\
1729:    \
1730:    template <class T1, class T2>\
1731:    BOOST_MATH_GPU_ENABLED inline boost::math::complex<typename boost::math::detail::bessel_traits<T1, T2, Policy >::result_type> sph_hankel_2(T1 v, T2 x)\
1732:    { return boost::math::sph_hankel_2(v, x, Policy()); }\
1733:    \
1734:    template <class T>\
1735:    inline boost::math::tools::promote_args_t<T> jacobi_elliptic(T k, T theta, T* pcn, T* pdn)\
1736:    { return static_cast<boost::math::tools::promote_args_t<T>>(boost::math::jacobi_elliptic(k, theta, pcn, pdn, Policy())); }\
````
- **L1709 EN**: Introduces template parameters or specialization context: `template <class T> T nextafter(const T& a, const T& b){ return static_cast<T>(boost::math::nextafter(a, b, Policy())); }\`.
  - **L1709 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> T nextafter(const T& a, const T& b){ return static_cast<T>(boost::math::nextafter(a, b, Policy())); }\`。
- **L1710 EN**: Introduces template parameters or specialization context: `template <class T> T float_next(const T& a){ return static_cast<T>(boost::math::float_next(a, Policy())); }\`.
  - **L1710 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> T float_next(const T& a){ return static_cast<T>(boost::math::float_next(a, Policy())); }\`。
- **L1711 EN**: Introduces template parameters or specialization context: `template <class T> T float_prior(const T& a){ return static_cast<T>(boost::math::float_prior(a, Policy())); }\`.
  - **L1711 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> T float_prior(const T& a){ return static_cast<T>(boost::math::float_prior(a, Policy())); }\`。
- **L1712 EN**: Introduces template parameters or specialization context: `template <class T> T float_distance(const T& a, const T& b){ return static_cast<T>(boost::math::float_distance(a, b, Policy())); }\`.
  - **L1712 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> T float_distance(const T& a, const T& b){ return static_cast<T>(boost::math::float_distance(a, b, Policy())); }\`。
- **L1713 EN**: Introduces template parameters or specialization context: `template <class T> T ulp(const T& a){ return static_cast<T>(boost::math::ulp(a, Policy())); }\`.
  - **L1713 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> T ulp(const T& a){ return static_cast<T>(boost::math::ulp(a, Policy())); }\`。
- **L1714 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1714 CN**: 继续构造周围的表达式或声明：`\`。
- **L1715 EN**: Introduces template parameters or specialization context: `template <class RT1, class RT2>\`.
  - **L1715 CN**: 为后续声明引入模板参数或特化上下文：`template <class RT1, class RT2>\`。
- **L1716 EN**: Starts a function or method definition for `owens_t`.
  - **L1716 CN**: 开始定义函数或方法 `owens_t`。
- **L1717 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1717 CN**: 继续构造周围的表达式或声明：`\`。
- **L1718 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1718 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1719 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1719 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1720 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1720 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1721 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1721 CN**: 继续构造周围的表达式或声明：`\`。
- **L1722 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1722 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1723 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1723 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1724 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1724 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1725 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1725 CN**: 继续构造周围的表达式或声明：`\`。
- **L1726 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1726 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1727 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1727 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1728 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1728 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1729 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1729 CN**: 继续构造周围的表达式或声明：`\`。
- **L1730 EN**: Introduces template parameters or specialization context: `template <class T1, class T2>\`.
  - **L1730 CN**: 为后续声明引入模板参数或特化上下文：`template <class T1, class T2>\`。
- **L1731 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1731 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1732 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1732 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1733 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1733 CN**: 继续构造周围的表达式或声明：`\`。
- **L1734 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1734 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1735 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1735 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1736 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1736 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1737-1764 / 第 1737-1764 行

````cpp
1737:    \
1738:    template <class U, class T>\
1739:    inline boost::math::tools::promote_args_t<T, U> jacobi_sn(U k, T theta)\
1740:    { return boost::math::jacobi_sn(k, theta, Policy()); }\
1741:    \
1742:    template <class T, class U>\
1743:    inline boost::math::tools::promote_args_t<T, U> jacobi_cn(T k, U theta)\
1744:    { return boost::math::jacobi_cn(k, theta, Policy()); }\
1745:    \
1746:    template <class T, class U>\
1747:    inline boost::math::tools::promote_args_t<T, U> jacobi_dn(T k, U theta)\
1748:    { return boost::math::jacobi_dn(k, theta, Policy()); }\
1749:    \
1750:    template <class T, class U>\
1751:    inline boost::math::tools::promote_args_t<T, U> jacobi_cd(T k, U theta)\
1752:    { return boost::math::jacobi_cd(k, theta, Policy()); }\
1753:    \
1754:    template <class T, class U>\
1755:    inline boost::math::tools::promote_args_t<T, U> jacobi_dc(T k, U theta)\
1756:    { return boost::math::jacobi_dc(k, theta, Policy()); }\
1757:    \
1758:    template <class T, class U>\
1759:    inline boost::math::tools::promote_args_t<T, U> jacobi_ns(T k, U theta)\
1760:    { return boost::math::jacobi_ns(k, theta, Policy()); }\
1761:    \
1762:    template <class T, class U>\
1763:    inline boost::math::tools::promote_args_t<T, U> jacobi_sd(T k, U theta)\
1764:    { return boost::math::jacobi_sd(k, theta, Policy()); }\
````
- **L1737 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1737 CN**: 继续构造周围的表达式或声明：`\`。
- **L1738 EN**: Introduces template parameters or specialization context: `template <class U, class T>\`.
  - **L1738 CN**: 为后续声明引入模板参数或特化上下文：`template <class U, class T>\`。
- **L1739 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1739 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1740 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1740 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1741 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1741 CN**: 继续构造周围的表达式或声明：`\`。
- **L1742 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1742 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1743 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1743 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1744 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1744 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1745 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1745 CN**: 继续构造周围的表达式或声明：`\`。
- **L1746 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1746 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1747 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1747 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1748 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1748 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1749 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1749 CN**: 继续构造周围的表达式或声明：`\`。
- **L1750 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1750 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1751 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1751 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1752 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1752 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1753 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1753 CN**: 继续构造周围的表达式或声明：`\`。
- **L1754 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1754 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1755 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1755 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1756 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1756 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1757 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1757 CN**: 继续构造周围的表达式或声明：`\`。
- **L1758 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1758 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1759 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1759 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1760 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1760 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1761 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1761 CN**: 继续构造周围的表达式或声明：`\`。
- **L1762 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1762 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1763 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1763 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1764 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1764 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1765-1792 / 第 1765-1792 行

````cpp
1765:    \
1766:    template <class T, class U>\
1767:    inline boost::math::tools::promote_args_t<T, U> jacobi_ds(T k, U theta)\
1768:    { return boost::math::jacobi_ds(k, theta, Policy()); }\
1769:    \
1770:    template <class T, class U>\
1771:    inline boost::math::tools::promote_args_t<T, U> jacobi_nc(T k, U theta)\
1772:    { return boost::math::jacobi_nc(k, theta, Policy()); }\
1773:    \
1774:    template <class T, class U>\
1775:    inline boost::math::tools::promote_args_t<T, U> jacobi_nd(T k, U theta)\
1776:    { return boost::math::jacobi_nd(k, theta, Policy()); }\
1777:    \
1778:    template <class T, class U>\
1779:    inline boost::math::tools::promote_args_t<T, U> jacobi_sc(T k, U theta)\
1780:    { return boost::math::jacobi_sc(k, theta, Policy()); }\
1781:    \
1782:    template <class T, class U>\
1783:    inline boost::math::tools::promote_args_t<T, U> jacobi_cs(T k, U theta)\
1784:    { return boost::math::jacobi_cs(k, theta, Policy()); }\
1785:    \
1786:    template <class T, class U>\
1787:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta1(T z, U q)\
1788:    { return boost::math::jacobi_theta1(z, q, Policy()); }\
1789:    \
1790:    template <class T, class U>\
1791:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta2(T z, U q)\
1792:    { return boost::math::jacobi_theta2(z, q, Policy()); }\
````
- **L1765 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1765 CN**: 继续构造周围的表达式或声明：`\`。
- **L1766 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1766 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1767 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1767 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1768 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1768 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1769 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1769 CN**: 继续构造周围的表达式或声明：`\`。
- **L1770 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1770 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1771 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1771 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1772 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1772 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1773 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1773 CN**: 继续构造周围的表达式或声明：`\`。
- **L1774 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1774 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1775 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1775 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1776 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1776 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1777 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1777 CN**: 继续构造周围的表达式或声明：`\`。
- **L1778 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1778 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1779 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1779 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1780 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1780 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1781 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1781 CN**: 继续构造周围的表达式或声明：`\`。
- **L1782 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1782 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1783 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1783 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1784 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1784 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1785 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1785 CN**: 继续构造周围的表达式或声明：`\`。
- **L1786 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1786 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1787 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1787 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1788 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1788 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1789 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1789 CN**: 继续构造周围的表达式或声明：`\`。
- **L1790 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1790 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1791 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1791 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1792 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1792 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1793-1820 / 第 1793-1820 行

````cpp
1793:    \
1794:    template <class T, class U>\
1795:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta3(T z, U q)\
1796:    { return boost::math::jacobi_theta3(z, q, Policy()); }\
1797:    \
1798:    template <class T, class U>\
1799:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta4(T z, U q)\
1800:    { return boost::math::jacobi_theta4(z, q, Policy()); }\
1801:    \
1802:    template <class T, class U>\
1803:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta1tau(T z, U q)\
1804:    { return boost::math::jacobi_theta1tau(z, q, Policy()); }\
1805:    \
1806:    template <class T, class U>\
1807:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta2tau(T z, U q)\
1808:    { return boost::math::jacobi_theta2tau(z, q, Policy()); }\
1809:    \
1810:    template <class T, class U>\
1811:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta3tau(T z, U q)\
1812:    { return boost::math::jacobi_theta3tau(z, q, Policy()); }\
1813:    \
1814:    template <class T, class U>\
1815:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta4tau(T z, U q)\
1816:    { return boost::math::jacobi_theta4tau(z, q, Policy()); }\
1817:    \
1818:    template <class T, class U>\
1819:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta3m1(T z, U q)\
1820:    { return boost::math::jacobi_theta3m1(z, q, Policy()); }\
````
- **L1793 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1793 CN**: 继续构造周围的表达式或声明：`\`。
- **L1794 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1794 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1795 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1795 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1796 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1796 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1797 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1797 CN**: 继续构造周围的表达式或声明：`\`。
- **L1798 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1798 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1799 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1799 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1800 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1800 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1801 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1801 CN**: 继续构造周围的表达式或声明：`\`。
- **L1802 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1802 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1803 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1803 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1804 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1804 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1805 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1805 CN**: 继续构造周围的表达式或声明：`\`。
- **L1806 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1806 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1807 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1807 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1808 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1808 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1809 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1809 CN**: 继续构造周围的表达式或声明：`\`。
- **L1810 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1810 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1811 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1811 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1812 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1812 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1813 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1813 CN**: 继续构造周围的表达式或声明：`\`。
- **L1814 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1814 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1815 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1815 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1816 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1816 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1817 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1817 CN**: 继续构造周围的表达式或声明：`\`。
- **L1818 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1818 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1819 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1819 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1820 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1820 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1821-1848 / 第 1821-1848 行

````cpp
1821:    \
1822:    template <class T, class U>\
1823:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta4m1(T z, U q)\
1824:    { return boost::math::jacobi_theta4m1(z, q, Policy()); }\
1825:    \
1826:    template <class T, class U>\
1827:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta3m1tau(T z, U q)\
1828:    { return boost::math::jacobi_theta3m1tau(z, q, Policy()); }\
1829:    \
1830:    template <class T, class U>\
1831:    inline boost::math::tools::promote_args_t<T, U> jacobi_theta4m1tau(T z, U q)\
1832:    { return boost::math::jacobi_theta4m1tau(z, q, Policy()); }\
1833:    \
1834:    template <class T>\
1835:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> airy_ai(T x)\
1836:    {  return boost::math::airy_ai(x, Policy());  }\
1837:    \
1838:    template <class T>\
1839:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> airy_bi(T x)\
1840:    {  return boost::math::airy_bi(x, Policy());  }\
1841:    \
1842:    template <class T>\
1843:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> airy_ai_prime(T x)\
1844:    {  return boost::math::airy_ai_prime(x, Policy());  }\
1845:    \
1846:    template <class T>\
1847:    BOOST_MATH_GPU_ENABLED inline boost::math::tools::promote_args_t<T> airy_bi_prime(T x)\
1848:    {  return boost::math::airy_bi_prime(x, Policy());  }\
````
- **L1821 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1821 CN**: 继续构造周围的表达式或声明：`\`。
- **L1822 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1822 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1823 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1823 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1824 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1824 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1825 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1825 CN**: 继续构造周围的表达式或声明：`\`。
- **L1826 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1826 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1827 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1827 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1828 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1828 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1829 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1829 CN**: 继续构造周围的表达式或声明：`\`。
- **L1830 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1830 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1831 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1831 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1832 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1832 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1833 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1833 CN**: 继续构造周围的表达式或声明：`\`。
- **L1834 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1834 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1835 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1835 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1836 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1836 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1837 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1837 CN**: 继续构造周围的表达式或声明：`\`。
- **L1838 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1838 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1839 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1839 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1840 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1840 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1841 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1841 CN**: 继续构造周围的表达式或声明：`\`。
- **L1842 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1842 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1843 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1843 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1844 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1844 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1845 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1845 CN**: 继续构造周围的表达式或声明：`\`。
- **L1846 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1846 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1847 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1847 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1848 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1848 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1849-1876 / 第 1849-1876 行

````cpp
1849:    \
1850:    template <class T>\
1851:    BOOST_MATH_GPU_ENABLED inline T airy_ai_zero(int m)\
1852:    { return boost::math::airy_ai_zero<T>(m, Policy()); }\
1853:    template <class T, class OutputIterator>\
1854:    BOOST_MATH_GPU_ENABLED OutputIterator airy_ai_zero(int start_index, unsigned number_of_zeros, OutputIterator out_it)\
1855:    { return boost::math::airy_ai_zero<T>(start_index, number_of_zeros, out_it, Policy()); }\
1856:    \
1857:    template <class T>\
1858:    BOOST_MATH_GPU_ENABLED inline T airy_bi_zero(int m)\
1859:    { return boost::math::airy_bi_zero<T>(m, Policy()); }\
1860:    template <class T, class OutputIterator>\
1861:    BOOST_MATH_GPU_ENABLED OutputIterator airy_bi_zero(int start_index, unsigned number_of_zeros, OutputIterator out_it)\
1862:    { return boost::math::airy_bi_zero<T>(start_index, number_of_zeros, out_it, Policy()); }\
1863:    \
1864:    template <class T>\
1865:    T bernoulli_b2n(const int i)\
1866:    { return boost::math::bernoulli_b2n<T>(i, Policy()); }\
1867:    template <class T, class OutputIterator>\
1868:    OutputIterator bernoulli_b2n(int start_index, unsigned number_of_bernoullis_b2n, OutputIterator out_it)\
1869:    { return boost::math::bernoulli_b2n<T>(start_index, number_of_bernoullis_b2n, out_it, Policy()); }\
1870:    \
1871:    template <class T>\
1872:    T tangent_t2n(const int i)\
1873:    { return boost::math::tangent_t2n<T>(i, Policy()); }\
1874:    template <class T, class OutputIterator>\
1875:    OutputIterator tangent_t2n(int start_index, unsigned number_of_bernoullis_b2n, OutputIterator out_it)\
1876:    { return boost::math::tangent_t2n<T>(start_index, number_of_bernoullis_b2n, out_it, Policy()); }\
````
- **L1849 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1849 CN**: 继续构造周围的表达式或声明：`\`。
- **L1850 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1850 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1851 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1851 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1852 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1852 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1853 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>\`.
  - **L1853 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>\`。
- **L1854 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1854 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1855 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1855 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1856 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1856 CN**: 继续构造周围的表达式或声明：`\`。
- **L1857 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1857 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1858 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1858 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1859 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1859 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1860 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>\`.
  - **L1860 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>\`。
- **L1861 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L1861 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L1862 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1862 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1863 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1863 CN**: 继续构造周围的表达式或声明：`\`。
- **L1864 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1864 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1865 EN**: Continues logic associated with callable symbol `bernoulli_b2n`.
  - **L1865 CN**: 继续与可调用符号 `bernoulli_b2n` 相关的逻辑。
- **L1866 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1866 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1867 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>\`.
  - **L1867 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>\`。
- **L1868 EN**: Continues logic associated with callable symbol `bernoulli_b2n`.
  - **L1868 CN**: 继续与可调用符号 `bernoulli_b2n` 相关的逻辑。
- **L1869 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1869 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1870 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1870 CN**: 继续构造周围的表达式或声明：`\`。
- **L1871 EN**: Introduces template parameters or specialization context: `template <class T>\`.
  - **L1871 CN**: 为后续声明引入模板参数或特化上下文：`template <class T>\`。
- **L1872 EN**: Continues logic associated with callable symbol `tangent_t2n`.
  - **L1872 CN**: 继续与可调用符号 `tangent_t2n` 相关的逻辑。
- **L1873 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1873 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1874 EN**: Introduces template parameters or specialization context: `template <class T, class OutputIterator>\`.
  - **L1874 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class OutputIterator>\`。
- **L1875 EN**: Continues logic associated with callable symbol `tangent_t2n`.
  - **L1875 CN**: 继续与可调用符号 `tangent_t2n` 相关的逻辑。
- **L1876 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1876 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 1877-1901 / 第 1877-1901 行

````cpp
1877:    \
1878:    template <class T> inline boost::math::tools::promote_args_t<T> lambert_w0(T z) { return boost::math::lambert_w0(z, Policy()); }\
1879:    template <class T> inline boost::math::tools::promote_args_t<T> lambert_wm1(T z) { return boost::math::lambert_w0(z, Policy()); }\
1880:    template <class T> inline boost::math::tools::promote_args_t<T> lambert_w0_prime(T z) { return boost::math::lambert_w0(z, Policy()); }\
1881:    template <class T> inline boost::math::tools::promote_args_t<T> lambert_wm1_prime(T z) { return boost::math::lambert_w0(z, Policy()); }\
1882:    \
1883:    template <class T, class U>\
1884:    inline boost::math::tools::promote_args_t<T, U> hypergeometric_1F0(const T& a, const U& z)\
1885:    { return boost::math::hypergeometric_1F0(a, z, Policy()); }\
1886:    \
1887:    template <class T, class U>\
1888:    inline boost::math::tools::promote_args_t<T, U> hypergeometric_0F1(const T& a, const U& z)\
1889:    { return boost::math::hypergeometric_0F1(a, z, Policy()); }\
1890:    \
1891:    template <class T, class U, class V>\
1892:    inline boost::math::tools::promote_args_t<T, U> hypergeometric_2F0(const T& a1, const U& a2, const V& z)\
1893:    { return boost::math::hypergeometric_2F0(a1, a2, z, Policy()); }\
1894:    \
1895: 
1896: 
1897: 
1898: 
1899: #endif // BOOST_MATH_HAS_NVRTC
1900: 
1901: #endif // BOOST_MATH_SPECIAL_MATH_FWD_HPP
````
- **L1877 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1877 CN**: 继续构造周围的表达式或声明：`\`。
- **L1878 EN**: Introduces template parameters or specialization context: `template <class T> inline boost::math::tools::promote_args_t<T> lambert_w0(T z) { return boost::math::lambert_w0(z, Policy()); }\`.
  - **L1878 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> inline boost::math::tools::promote_args_t<T> lambert_w0(T z) { return boost::math::lambert_w0(z, Policy()); }\`。
- **L1879 EN**: Introduces template parameters or specialization context: `template <class T> inline boost::math::tools::promote_args_t<T> lambert_wm1(T z) { return boost::math::lambert_w0(z, Policy()); }\`.
  - **L1879 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> inline boost::math::tools::promote_args_t<T> lambert_wm1(T z) { return boost::math::lambert_w0(z, Policy()); }\`。
- **L1880 EN**: Introduces template parameters or specialization context: `template <class T> inline boost::math::tools::promote_args_t<T> lambert_w0_prime(T z) { return boost::math::lambert_w0(z, Policy()); }\`.
  - **L1880 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> inline boost::math::tools::promote_args_t<T> lambert_w0_prime(T z) { return boost::math::lambert_w0(z, Policy()); }\`。
- **L1881 EN**: Introduces template parameters or specialization context: `template <class T> inline boost::math::tools::promote_args_t<T> lambert_wm1_prime(T z) { return boost::math::lambert_w0(z, Policy()); }\`.
  - **L1881 CN**: 为后续声明引入模板参数或特化上下文：`template <class T> inline boost::math::tools::promote_args_t<T> lambert_wm1_prime(T z) { return boost::math::lambert_w0(z, Policy()); }\`。
- **L1882 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1882 CN**: 继续构造周围的表达式或声明：`\`。
- **L1883 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1883 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1884 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1884 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1885 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1885 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1886 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1886 CN**: 继续构造周围的表达式或声明：`\`。
- **L1887 EN**: Introduces template parameters or specialization context: `template <class T, class U>\`.
  - **L1887 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>\`。
- **L1888 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1888 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1889 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1889 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1890 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1890 CN**: 继续构造周围的表达式或声明：`\`。
- **L1891 EN**: Introduces template parameters or specialization context: `template <class T, class U, class V>\`.
  - **L1891 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class V>\`。
- **L1892 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1892 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1893 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1893 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1894 EN**: Continues the surrounding expression or declaration: `\`.
  - **L1894 CN**: 继续构造周围的表达式或声明：`\`。
- **L1895 EN**: Blank line separating nearby declarations or logic.
  - **L1895 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1896 EN**: Blank line separating nearby declarations or logic.
  - **L1896 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1897 EN**: Blank line separating nearby declarations or logic.
  - **L1897 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1898 EN**: Blank line separating nearby declarations or logic.
  - **L1898 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1899 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1899 CN**: 结束当前预处理条件块或头文件保护。
- **L1900 EN**: Blank line separating nearby declarations or logic.
  - **L1900 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1901 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1901 CN**: 结束当前预处理条件块或头文件保护。

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
- **Iterator-based algorithms / 基于迭代器的算法**:
  - **EN**: Operates over iterator ranges and generic containers instead of concrete storage types.
  - **CN**: 面向迭代器区间和泛型容器工作，而不是绑定具体存储类型。
- **Compile-time evaluation / 编译期求值**:
  - **EN**: Marks values or functions so some work can be performed during compilation.
  - **CN**: 标记值或函数，使部分工作可在编译期间完成。
- **Boost integration / Boost 集成**:
  - **EN**: Fits the implementation into Boost naming, configuration, and portability conventions.
  - **CN**: 让实现融入 Boost 的命名、配置与可移植性约定。
- **Dependency layering / 依赖分层**:
  - **EN**: Builds on included headers instead of re-declaring reusable infrastructure.
  - **CN**: 构建在已包含头文件之上，而不是重复声明可复用基础设施。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `boost/math/tools/config.hpp`, `boost/math/tools/promotion.hpp`, `boost/math/tools/type_traits.hpp`, `boost/math/tools/complex.hpp`, `boost/math/policies/policy.hpp`, `vector`, `complex`, `type_traits`, `boost/math/special_functions/detail/round_fwd.hpp`
- **Dependency categories / 依赖类别**: Boost.Math numeric tool helpers / Boost.Math 数值工具辅助逻辑 (4), C or C++ standard library facilities / C 或 C++ 标准库设施 (3), Boost.Math policy configuration / Boost.Math 策略配置 (1), Boost.Math special-function declarations / Boost.Math 特殊函数声明 (1)

- **EN**: `boost/math/tools/config.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/config.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/promotion.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/promotion.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/type_traits.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/type_traits.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/complex.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/complex.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/policies/policy.hpp` provides Boost.Math policy configuration.
  - **CN**: `boost/math/policies/policy.hpp` 提供Boost.Math 策略配置。
- **EN**: `vector` provides C or C++ standard library facilities.
  - **CN**: `vector` 提供C 或 C++ 标准库设施。
- **EN**: `complex` provides C or C++ standard library facilities.
  - **CN**: `complex` 提供C 或 C++ 标准库设施。
- **EN**: `type_traits` provides C or C++ standard library facilities.
  - **CN**: `type_traits` 提供C 或 C++ 标准库设施。
- **EN**: `boost/math/special_functions/detail/round_fwd.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/detail/round_fwd.hpp` 提供Boost.Math 特殊函数声明。
