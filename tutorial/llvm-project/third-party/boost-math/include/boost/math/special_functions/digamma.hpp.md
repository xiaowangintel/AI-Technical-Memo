# digamma.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/digamma.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the digamma special function and related helpers.
- **作用（中文）**: 此头文件实现 digamma 特殊函数及相关辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  (C) Copyright John Maddock 2006.
   2: //  (C) Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_SF_DIGAMMA_HPP
   8: #define BOOST_MATH_SF_DIGAMMA_HPP
   9: 
  10: #ifdef _MSC_VER
  11: #pragma once
  12: #pragma warning(push)
  13: #pragma warning(disable:4702) // Unreachable code (release mode only warning)
  14: #endif
  15: 
  16: #include <boost/math/tools/config.hpp>
  17: #include <boost/math/tools/type_traits.hpp>
  18: #include <boost/math/tools/rational.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/rational.hpp so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/rational.hpp 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/tools/promotion.hpp>
  20: #include <boost/math/policies/policy.hpp>
  21: #include <boost/math/policies/error_handling.hpp>
  22: #include <boost/math/constants/constants.hpp>
  23: 
  24: #ifndef BOOST_MATH_HAS_NVRTC
  25: #include <boost/math/special_functions/math_fwd.hpp>
  26: #include <boost/math/tools/series.hpp>
  27: #include <boost/math/policies/error_handling.hpp>
  28: #include <boost/math/constants/constants.hpp>
  29: #include <boost/math/tools/big_constant.hpp>
  30: #endif
  31: 
  32: #if defined(__GNUC__) && defined(BOOST_MATH_USE_FLOAT128)
  33: //
  34: // This is the only way we can avoid
  35: // warning: non-standard suffix on floating constant [-Wpedantic]
  36: // when building with -Wall -pedantic.  Neither __extension__
~~~
- **EN:** This block imports dependencies such as boost/math/tools/promotion.hpp, boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, ... so the surrounding code can use external declarations. It begins the file-level inclusion guard or portability wrapper. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/promotion.hpp, boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, ... 等依赖，使周围代码可以使用外部声明。 它开始设置文件级防重包含或可移植性包装层。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: // nor #pragma diagnostic ignored work :(
  38: //
  39: #pragma GCC system_header
  40: #endif
  41: 
  42: namespace boost{
  43: namespace math{
  44: namespace detail{
  45: //
  46: // Begin by defining the smallest value for which it is safe to
  47: // use the asymptotic expansion for digamma:
  48: //
  49: BOOST_MATH_GPU_ENABLED inline unsigned digamma_large_lim(const boost::math::integral_constant<int, 0>*)
  50: {  return 20;  }
  51: BOOST_MATH_GPU_ENABLED inline unsigned digamma_large_lim(const boost::math::integral_constant<int, 113>*)
  52: {  return 20;  }
  53: BOOST_MATH_GPU_ENABLED inline unsigned digamma_large_lim(const void*)
  54: {  return 10;  }
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: //
  56: // Implementations of the asymptotic expansion come next,
  57: // the coefficients of the series have been evaluated
  58: // in advance at high precision, and the series truncated
  59: // at the first term that's too small to effect the result.
  60: // Note that the series becomes divergent after a while
  61: // so truncation is very important.
  62: //
  63: // This first one gives 34-digit precision for x >= 20:
  64: //
  65: 
  66: #ifndef BOOST_MATH_HAS_NVRTC
  67: template <class T>
  68: inline T digamma_imp_large(T x, const boost::math::integral_constant<int, 113>*)
  69: {
  70:    BOOST_MATH_STD_USING // ADL of std functions.
  71:    static const T P[] = {
  72:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.083333333333333333333333333333333333333333333333333),
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0083333333333333333333333333333333333333333333333333),
  74:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.003968253968253968253968253968253968253968253968254),
  75:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0041666666666666666666666666666666666666666666666667),
  76:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0075757575757575757575757575757575757575757575757576),
  77:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.021092796092796092796092796092796092796092796092796),
  78:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.083333333333333333333333333333333333333333333333333),
  79:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.44325980392156862745098039215686274509803921568627),
  80:       BOOST_MATH_BIG_CONSTANT(T, 113, 3.0539543302701197438039543302701197438039543302701),
  81:       BOOST_MATH_BIG_CONSTANT(T, 113, -26.456212121212121212121212121212121212121212121212),
  82:       BOOST_MATH_BIG_CONSTANT(T, 113, 281.4601449275362318840579710144927536231884057971),
  83:       BOOST_MATH_BIG_CONSTANT(T, 113, -3607.510546398046398046398046398046398046398046398),
  84:       BOOST_MATH_BIG_CONSTANT(T, 113, 54827.583333333333333333333333333333333333333333333),
  85:       BOOST_MATH_BIG_CONSTANT(T, 113, -974936.82385057471264367816091954022988505747126437),
  86:       BOOST_MATH_BIG_CONSTANT(T, 113, 20052695.796688078946143462272494530559046688078946),
  87:       BOOST_MATH_BIG_CONSTANT(T, 113, -472384867.72162990196078431372549019607843137254902),
  88:       BOOST_MATH_BIG_CONSTANT(T, 113, 12635724795.916666666666666666666666666666666666667)
  89:    };
  90:    x -= 1;
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:    T result = log(x);
  92:    result += 1 / (2 * x);
  93:    T z = 1 / (x*x);
  94:    result -= z * tools::evaluate_polynomial(P, z);
  95:    return result;
  96: }
  97: //
  98: // 19-digit precision for x >= 10:
  99: //
 100: template <class T>
 101: inline T digamma_imp_large(T x, const boost::math::integral_constant<int, 64>*)
 102: {
 103:    BOOST_MATH_STD_USING // ADL of std functions.
 104:    static const T P[] = {
 105:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.083333333333333333333333333333333333333333333333333),
 106:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.0083333333333333333333333333333333333333333333333333),
 107:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.003968253968253968253968253968253968253968253968254),
 108:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.0041666666666666666666666666666666666666666666666667),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as log, tools::evaluate_polynomial.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log, tools::evaluate_polynomial。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.0075757575757575757575757575757575757575757575757576),
 110:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.021092796092796092796092796092796092796092796092796),
 111:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.083333333333333333333333333333333333333333333333333),
 112:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.44325980392156862745098039215686274509803921568627),
 113:       BOOST_MATH_BIG_CONSTANT(T, 64, 3.0539543302701197438039543302701197438039543302701),
 114:       BOOST_MATH_BIG_CONSTANT(T, 64, -26.456212121212121212121212121212121212121212121212),
 115:       BOOST_MATH_BIG_CONSTANT(T, 64, 281.4601449275362318840579710144927536231884057971),
 116:    };
 117:    x -= 1;
 118:    T result = log(x);
 119:    result += 1 / (2 * x);
 120:    T z = 1 / (x*x);
 121:    result -= z * tools::evaluate_polynomial(P, z);
 122:    return result;
 123: }
 124: #endif
 125: //
 126: // 17-digit precision for x >= 10:
~~~
- **EN:** This range declares or defines callable logic such as log, tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127: //
 128: template <class T>
 129: BOOST_MATH_GPU_ENABLED inline T digamma_imp_large(T x, const boost::math::integral_constant<int, 53>*)
 130: {
 131:    BOOST_MATH_STD_USING // ADL of std functions.
 132:    BOOST_MATH_STATIC const T P[] = {
 133:       0.083333333333333333333333333333333333333333333333333,
 134:       -0.0083333333333333333333333333333333333333333333333333,
 135:       0.003968253968253968253968253968253968253968253968254,
 136:       -0.0041666666666666666666666666666666666666666666666667,
 137:       0.0075757575757575757575757575757575757575757575757576,
 138:       -0.021092796092796092796092796092796092796092796092796,
 139:       0.083333333333333333333333333333333333333333333333333,
 140:       -0.44325980392156862745098039215686274509803921568627
 141:    };
 142:    x -= 1;
 143:    T result = log(x);
 144:    result += 1 / (2 * x);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as log.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:    T z = 1 / (x*x);
 146:    result -= z * tools::evaluate_polynomial(P, z);
 147:    return result;
 148: }
 149: //
 150: // 9-digit precision for x >= 10:
 151: //
 152: template <class T>
 153: BOOST_MATH_GPU_ENABLED inline T digamma_imp_large(T x, const boost::math::integral_constant<int, 24>*)
 154: {
 155:    BOOST_MATH_STD_USING // ADL of std functions.
 156:    BOOST_MATH_STATIC const T P[] = {
 157:       0.083333333333333333333333333333333333333333333333333f,
 158:       -0.0083333333333333333333333333333333333333333333333333f,
 159:       0.003968253968253968253968253968253968253968253968254f
 160:    };
 161:    x -= 1;
 162:    T result = log(x);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, log.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, log。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:    result += 1 / (2 * x);
 164:    T z = 1 / (x*x);
 165:    result -= z * tools::evaluate_polynomial(P, z);
 166:    return result;
 167: }
 168: 
 169: #ifndef BOOST_MATH_HAS_NVRTC
 170: //
 171: // Fully generic asymptotic expansion in terms of Bernoulli numbers, see:
 172: // http://functions.wolfram.com/06.14.06.0012.01
 173: //
 174: // LCOV_EXCL_START muliprecision only.
 175: template <class T>
 176: struct digamma_series_func
 177: {
 178: private:
 179:    int k;
 180:    T xx;
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:    T term;
 182: public:
 183:    digamma_series_func(T x) : k(1), xx(x * x), term(1 / (x * x)) {}
 184:    T operator()()
 185:    {
 186:       T result = term * boost::math::bernoulli_b2n<T>(k) / (2 * k);
 187:       term /= xx;
 188:       ++k;
 189:       return result;
 190:    }
 191:    typedef T result_type;
 192: };
 193: 
 194: template <class T, class Policy>
 195: inline T digamma_imp_large(T x, const Policy& pol, const boost::math::integral_constant<int, 0>*)
 196: {
 197:    BOOST_MATH_STD_USING
 198:    digamma_series_func<T> s(x);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as digamma_series_func, s.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 digamma_series_func, s。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:    T result = log(x) - 1 / (2 * x);
 200:    std::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 201:    result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter, -result);
 202:    result = -result;
 203:    policies::check_series_iterations<T>("boost::math::digamma<%1%>(%1%)", max_iter, pol);
 204:    return result;
 205: }
 206: // LCOV_EXCL_STOP
 207: //
 208: // Now follow rational approximations over the range [1,2].
 209: //
 210: // 35-digit precision:
 211: //
 212: template <class T>
 213: T digamma_imp_1_2(T x, const boost::math::integral_constant<int, 113>*)
 214: {
 215:    //
 216:    // Now the approximation, we use the form:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as log, boost::math::tools::sum_series.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log, boost::math::tools::sum_series。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:    //
 218:    // digamma(x) = (x - root) * (Y + R(x-1))
 219:    //
 220:    // Where root is the location of the positive root of digamma,
 221:    // Y is a constant, and R is optimised for low absolute error
 222:    // compared to Y.
 223:    //
 224:    // Max error found at 128-bit long double precision:  5.541e-35
 225:    // Maximum Deviation Found (approximation error):     1.965e-35
 226:    //
 227:    // LCOV_EXCL_START
 228:    static const float Y = 0.99558162689208984375F;
 229: 
 230:    static const T root1 = T(1569415565) / 1073741824uL;
 231:    static const T root2 = (T(381566830) / 1073741824uL) / 1073741824uL;
 232:    static const T root3 = ((T(111616537) / 1073741824uL) / 1073741824uL) / 1073741824uL;
 233:    static const T root4 = (((T(503992070) / 1073741824uL) / 1073741824uL) / 1073741824uL) / 1073741824uL;
 234:    static const T root5 = BOOST_MATH_BIG_CONSTANT(T, 113, 0.52112228569249997894452490385577338504019838794544e-36);
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_BIG_CONSTANT. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_BIG_CONSTANT。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235: 
 236:    static const T P[] = {
 237:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.25479851061131551526977464225335883769),
 238:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.18684290534374944114622235683619897417),
 239:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.80360876047931768958995775910991929922),
 240:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.67227342794829064330498117008564270136),
 241:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.26569010991230617151285010695543858005),
 242:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.05775672694575986971640757748003553385),
 243:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0071432147823164975485922555833274240665),
 244:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00048740753910766168912364555706064993274),
 245:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.16454996865214115723416538844975174761e-4),
 246:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.20327832297631728077731148515093164955e-6)
 247:    };
 248:    static const T Q[] = {
 249:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 250:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.6210924610812025425088411043163287646),
 251:       BOOST_MATH_BIG_CONSTANT(T, 113, 2.6850757078559596612621337395886392594),
 252:       BOOST_MATH_BIG_CONSTANT(T, 113, 1.4320913706209965531250495490639289418),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.4410872083455009362557012239501953402),
 254:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.081385727399251729505165509278152487225),
 255:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0089478633066857163432104815183858149496),
 256:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00055861622855066424871506755481997374154),
 257:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.1760168552357342401304462967950178554e-4),
 258:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.20585454493572473724556649516040874384e-6),
 259:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.90745971844439990284514121823069162795e-11),
 260:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.48857673606545846774761343500033283272e-13),
 261:    };
 262:    // LCOV_EXCL_STOP
 263:    T g = x - root1;
 264:    g -= root2;
 265:    g -= root3;
 266:    g -= root4;
 267:    g -= root5;
 268:    T r = tools::evaluate_polynomial(P, T(x-1)) / tools::evaluate_polynomial(Q, T(x-1));
 269:    T result = g * Y + g * r;
 270: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:    return result;
 272: }
 273: //
 274: // 19-digit precision:
 275: //
 276: template <class T>
 277: T digamma_imp_1_2(T x, const boost::math::integral_constant<int, 64>*)
 278: {
 279:    //
 280:    // Now the approximation, we use the form:
 281:    //
 282:    // digamma(x) = (x - root) * (Y + R(x-1))
 283:    //
 284:    // Where root is the location of the positive root of digamma,
 285:    // Y is a constant, and R is optimised for low absolute error
 286:    // compared to Y.
 287:    //
 288:    // Max error found at 80-bit long double precision:   5.016e-20
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:    // Maximum Deviation Found (approximation error):     3.575e-20
 290:    //
 291:    // LCOV_EXCL_START
 292:    static const float Y = 0.99558162689208984375F;
 293: 
 294:    static const T root1 = T(1569415565) / 1073741824uL;
 295:    static const T root2 = (T(381566830) / 1073741824uL) / 1073741824uL;
 296:    static const T root3 = BOOST_MATH_BIG_CONSTANT(T, 64, 0.9016312093258695918615325266959189453125e-19);
 297: 
 298:    static const T P[] = {
 299:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.254798510611315515235),
 300:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.314628554532916496608),
 301:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.665836341559876230295),
 302:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.314767657147375752913),
 303:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.0541156266153505273939),
 304:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00289268368333918761452)
 305:    };
 306:    static const T Q[] = {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_BIG_CONSTANT. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_BIG_CONSTANT。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:       BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 308:       BOOST_MATH_BIG_CONSTANT(T, 64, 2.1195759927055347547),
 309:       BOOST_MATH_BIG_CONSTANT(T, 64, 1.54350554664961128724),
 310:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.486986018231042975162),
 311:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.0660481487173569812846),
 312:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00298999662592323990972),
 313:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.165079794012604905639e-5),
 314:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.317940243105952177571e-7)
 315:    };
 316:    // LCOV_EXCL_STOP
 317:    T g = x - root1;
 318:    g -= root2;
 319:    g -= root3;
 320:    T r = tools::evaluate_polynomial(P, T(x-1)) / tools::evaluate_polynomial(Q, T(x-1));
 321:    T result = g * Y + g * r;
 322: 
 323:    return result;
 324: }
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325: 
 326: #endif
 327: //
 328: // 18-digit precision:
 329: //
 330: template <class T>
 331: BOOST_MATH_GPU_ENABLED T digamma_imp_1_2(T x, const boost::math::integral_constant<int, 53>*)
 332: {
 333:    //
 334:    // Now the approximation, we use the form:
 335:    //
 336:    // digamma(x) = (x - root) * (Y + R(x-1))
 337:    //
 338:    // Where root is the location of the positive root of digamma,
 339:    // Y is a constant, and R is optimised for low absolute error
 340:    // compared to Y.
 341:    //
 342:    // Maximum Deviation Found:               1.466e-18
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:    // At double precision, max error found:  2.452e-17
 344:    //
 345:    // LCOV_EXCL_START
 346:    BOOST_MATH_STATIC const float Y = 0.99558162689208984F;
 347: 
 348:    BOOST_MATH_STATIC const T root1 = T(1569415565) / 1073741824uL;
 349:    BOOST_MATH_STATIC const T root2 = (T(381566830) / 1073741824uL) / 1073741824uL;
 350:    BOOST_MATH_STATIC const T root3 = BOOST_MATH_BIG_CONSTANT(T, 53, 0.9016312093258695918615325266959189453125e-19);
 351: 
 352:    BOOST_MATH_STATIC const T P[] = {
 353:       BOOST_MATH_BIG_CONSTANT(T, 53, 0.25479851061131551),
 354:       BOOST_MATH_BIG_CONSTANT(T, 53, -0.32555031186804491),
 355:       BOOST_MATH_BIG_CONSTANT(T, 53, -0.65031853770896507),
 356:       BOOST_MATH_BIG_CONSTANT(T, 53, -0.28919126444774784),
 357:       BOOST_MATH_BIG_CONSTANT(T, 53, -0.045251321448739056),
 358:       BOOST_MATH_BIG_CONSTANT(T, 53, -0.0020713321167745952)
 359:    };
 360:    BOOST_MATH_STATIC const T Q[] = {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_BIG_CONSTANT. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_BIG_CONSTANT。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:       BOOST_MATH_BIG_CONSTANT(T, 53, 1.0),
 362:       BOOST_MATH_BIG_CONSTANT(T, 53, 2.0767117023730469),
 363:       BOOST_MATH_BIG_CONSTANT(T, 53, 1.4606242909763515),
 364:       BOOST_MATH_BIG_CONSTANT(T, 53, 0.43593529692665969),
 365:       BOOST_MATH_BIG_CONSTANT(T, 53, 0.054151797245674225),
 366:       BOOST_MATH_BIG_CONSTANT(T, 53, 0.0021284987017821144),
 367:       BOOST_MATH_BIG_CONSTANT(T, 53, -0.55789841321675513e-6)
 368:    };
 369:    // LCOV_EXCL_STOP
 370:    T g = x - root1;
 371:    g -= root2;
 372:    g -= root3;
 373:    T r = tools::evaluate_polynomial(P, T(x-1)) / tools::evaluate_polynomial(Q, T(x-1));
 374:    T result = g * Y + g * r;
 375: 
 376:    return result;
 377: }
 378: //
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379: // 9-digit precision:
 380: //
 381: template <class T>
 382: BOOST_MATH_GPU_ENABLED inline T digamma_imp_1_2(T x, const boost::math::integral_constant<int, 24>*)
 383: {
 384:    //
 385:    // Now the approximation, we use the form:
 386:    //
 387:    // digamma(x) = (x - root) * (Y + R(x-1))
 388:    //
 389:    // Where root is the location of the positive root of digamma,
 390:    // Y is a constant, and R is optimised for low absolute error
 391:    // compared to Y.
 392:    //
 393:    // Maximum Deviation Found:              3.388e-010
 394:    // At float precision, max error found:  2.008725e-008
 395:    //
 396:    // LCOV_EXCL_START
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:    BOOST_MATH_STATIC const float Y = 0.99558162689208984f;
 398:    BOOST_MATH_STATIC const T root = 1532632.0f / 1048576;
 399:    BOOST_MATH_STATIC const T root_minor = static_cast<T>(0.3700660185912626595423257213284682051735604e-6L);
 400:    BOOST_MATH_STATIC const T P[] = {
 401:       0.25479851023250261e0f,
 402:       -0.44981331915268368e0f,
 403:       -0.43916936919946835e0f,
 404:       -0.61041765350579073e-1f
 405:    };
 406:    BOOST_MATH_STATIC const T Q[] = {
 407:       0.1e1f,
 408:       0.15890202430554952e1f,
 409:       0.65341249856146947e0f,
 410:       0.63851690523355715e-1f
 411:    };
 412:    // LCOV_EXCL_STOP
 413:    T g = x - root;
 414:    g -= root_minor;
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:    T r = tools::evaluate_polynomial(P, T(x-1)) / tools::evaluate_polynomial(Q, T(x-1));
 416:    T result = g * Y + g * r;
 417: 
 418:    return result;
 419: }
 420: 
 421: template <class T, class Tag, class Policy>
 422: BOOST_MATH_GPU_ENABLED T digamma_imp(T x, const Tag* t, const Policy& pol)
 423: {
 424:    //
 425:    // This handles reflection of negative arguments, and all our
 426:    // error handling, then forwards to the T-specific approximation.
 427:    //
 428:    BOOST_MATH_STD_USING // ADL of std functions.
 429: 
 430:    T result = 0;
 431:    //
 432:    // Check for negative arguments and use reflection:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:    //
 434:    if(x <= -1)
 435:    {
 436:       // Reflect:
 437:       x = 1 - x;
 438:       // Argument reduction for tan:
 439:       T remainder = x - floor(x);
 440:       // Shift to negative if > 0.5:
 441:       if(remainder > T(0.5))
 442:       {
 443:          remainder -= 1;
 444:       }
 445:       //
 446:       // check for evaluation at a negative pole:
 447:       //
 448:       if(remainder == 0)
 449:       {
 450:          return policies::raise_pole_error<T>("boost::math::digamma<%1%>(%1%)", nullptr, (1-x), pol);
~~~
- **EN:** This range declares or defines callable logic such as floor. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 floor。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:       }
 452:       result = constants::pi<T>() / tan(constants::pi<T>() * remainder);
 453:    }
 454:    if(x == 0)
 455:       return policies::raise_pole_error<T>("boost::math::digamma<%1%>(%1%)", nullptr, x, pol);
 456:    //
 457:    // If we're above the lower-limit for the
 458:    // asymptotic expansion then use it:
 459:    //
 460:    #ifndef BOOST_MATH_HAS_NVRTC
 461:    if(x >= digamma_large_lim(t))
 462:    {
 463:       result += digamma_imp_large(x, t);
 464:    }
 465:    else
 466:    #endif
 467:    {
 468:       //
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. This range declares or defines callable logic such as tan, digamma_imp_large. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围声明或定义了可调用逻辑，例如 tan, digamma_imp_large。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:       // If x > 2 reduce to the interval [1,2]:
 470:       //
 471:       while(x > 2)
 472:       {
 473:          x -= 1;
 474:          result += 1/x;
 475:       }
 476:       //
 477:       // If x < 1 use recurrence to shift to > 1:
 478:       //
 479:       while(x < 1)
 480:       {
 481:          result -= 1/x;
 482:          x += 1;
 483:       }
 484:       result += digamma_imp_1_2(x, t);
 485:    }
 486:    return result;
~~~
- **EN:** This range declares or defines callable logic such as digamma_imp_1_2. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 digamma_imp_1_2。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 return 语句会把计算结果或状态返回给调用方。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487: }
 488: 
 489: #ifndef BOOST_MATH_HAS_NVRTC
 490: 
 491: // LCOV_EXCL_START
 492: template <class T, class Policy>
 493: T digamma_imp(T x, const boost::math::integral_constant<int, 0>* t, const Policy& pol)
 494: {
 495:    //
 496:    // This handles reflection of negative arguments, and all our
 497:    // error handling, then forwards to the T-specific approximation.
 498:    //
 499:    // This is covered by our real_concept tests, but these are disabled for
 500:    // code coverage runs for performance reasons.
 501:    //
 502:    BOOST_MATH_STD_USING // ADL of std functions.
 503: 
 504:    T result = 0;
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:    //
 506:    // Check for negative arguments and use reflection:
 507:    //
 508:    if(x <= -1)
 509:    {
 510:       // Reflect:
 511:       x = 1 - x;
 512:       // Argument reduction for tan:
 513:       T remainder = x - floor(x);
 514:       // Shift to negative if > 0.5:
 515:       if(remainder > T(0.5))
 516:       {
 517:          remainder -= 1;
 518:       }
 519:       //
 520:       // check for evaluation at a negative pole:
 521:       //
 522:       if(remainder == 0)
~~~
- **EN:** This range declares or defines callable logic such as floor. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 floor。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:       {
 524:          return policies::raise_pole_error<T>("boost::math::digamma<%1%>(%1%)", nullptr, (1 - x), pol);
 525:       }
 526:       result = constants::pi<T>() / tan(constants::pi<T>() * remainder);
 527:    }
 528:    if(x == 0)
 529:       return policies::raise_pole_error<T>("boost::math::digamma<%1%>(%1%)", nullptr, x, pol);
 530:    //
 531:    // If we're above the lower-limit for the
 532:    // asymptotic expansion then use it, the
 533:    // limit is a linear interpolation with
 534:    // limit = 10 at 50 bit precision and
 535:    // limit = 250 at 1000 bit precision.
 536:    //
 537:    int lim = 10 + ((tools::digits<T>() - 50) * 240L) / 950;
 538:    T two_x = ldexp(x, 1);
 539:    if(x >= lim)
 540:    {
~~~
- **EN:** This range declares or defines callable logic such as tan, ldexp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tan, ldexp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:       result += digamma_imp_large(x, pol, t);
 542:    }
 543:    else if(floor(x) == x)
 544:    {
 545:       //
 546:       // Special case for integer arguments, see
 547:       // http://functions.wolfram.com/06.14.03.0001.01
 548:       //
 549:       result = -constants::euler<T, Policy>();
 550:       T val = 1;
 551:       while(val < x)
 552:       {
 553:          result += 1 / val;
 554:          val += 1;
 555:       }
 556:    }
 557:    else if(floor(two_x) == two_x)
 558:    {
~~~
- **EN:** This range declares or defines callable logic such as digamma_imp_large. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 digamma_imp_large。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 559-576 / 第 559-576 行
~~~cpp
 559:       //
 560:       // Special case for half integer arguments, see:
 561:       // http://functions.wolfram.com/06.14.03.0007.01
 562:       //
 563:       result = -2 * constants::ln_two<T, Policy>() - constants::euler<T, Policy>();
 564:       int n = itrunc(x);
 565:       if(n)
 566:       {
 567:          for(int k = 1; k < n; ++k)
 568:             result += 1 / T(k);
 569:          for(int k = n; k <= 2 * n - 1; ++k)
 570:             result += 2 / T(k);
 571:       }
 572:    }
 573:    else
 574:    {
 575:       //
 576:       // Rescale so we can use the asymptotic expansion:
~~~
- **EN:** This range declares or defines callable logic such as itrunc, T. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc, T。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 577-594 / 第 577-594 行
~~~cpp
 577:       //
 578:       while(x < lim)
 579:       {
 580:          result -= 1 / x;
 581:          x += 1;
 582:       }
 583:       result += digamma_imp_large(x, pol, t);
 584:    }
 585:    return result;
 586: }
 587: // LCOV_EXCL_STOP
 588: 
 589: #endif
 590: 
 591: } // namespace detail
 592: 
 593: template <class T, class Policy>
 594: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 595-612 / 第 595-612 行
~~~cpp
 595:    digamma(T x, const Policy&)
 596: {
 597:    typedef typename tools::promote_args<T>::type result_type;
 598:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
 599:    typedef typename policies::precision<T, Policy>::type precision_type;
 600:    typedef boost::math::integral_constant<int,
 601:       (precision_type::value <= 0) || (precision_type::value > 113) ? 0 :
 602:       precision_type::value <= 24 ? 24 :
 603:       precision_type::value <= 53 ? 53 :
 604:       precision_type::value <= 64 ? 64 :
 605:       precision_type::value <= 113 ? 113 : 0 > tag_type;
 606:    typedef typename policies::normalise<
 607:       Policy,
 608:       policies::promote_float<false>,
 609:       policies::promote_double<false>,
 610:       policies::discrete_quantile<>,
 611:       policies::assert_undefined<> >::type forwarding_policy;
 612: 
~~~
- **EN:** Type aliases simplify verbose template names and make later declarations easier to read. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 类型别名可以简化冗长的模板名称，使后续声明更易阅读。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 613-630 / 第 613-630 行
~~~cpp
 613:    return policies::checked_narrowing_cast<result_type, Policy>(detail::digamma_imp(static_cast<value_type>(x), static_cast<const tag_type*>(nullptr), forwarding_policy()), "boost::math::digamma<%1%>(%1%)");
 614: }
 615: 
 616: template <class T>
 617: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type
 618:    digamma(T x)
 619: {
 620:    return digamma(x, policies::policy<>());
 621: }
 622: 
 623: } // namespace math
 624: } // namespace boost
 625: 
 626: #ifdef _MSC_VER
 627: #pragma warning(pop)
 628: #endif
 629: 
 630: #endif
~~~
- **EN:** The code enters namespace scope (math::boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 631-631 / 第 631-631 行
~~~cpp
 631: 
~~~
- **EN:** This range is intentionally blank and separates neighboring declarations or algorithmic steps.
- **CN:** 此范围是有意保留的空行，用于分隔相邻声明或算法步骤。

## Key Concepts / 关键概念
- **Header guard / 头文件保护**: Prevents repeated inclusion during compilation. / 防止编译期间被重复包含。
- **Header dependencies / 头文件依赖**: Pulls in other declarations, traits, and standard facilities. / 引入其他声明、类型萃取和标准库能力。
- **Namespaces / 命名空间**: Keeps third-party symbols scoped and avoids collisions. / 通过命名空间约束第三方符号，避免冲突。
- **Templates / 模板**: Generalizes the implementation across numeric types or policies. / 让实现可适配不同数值类型或策略。
- **Types / 类型定义**: Introduces classes or structs that model the library abstraction. / 引入类或结构体来表达库中的抽象。
- **Functions / 函数**: Provides callable entry points or helpers. / 提供可调用的入口或辅助函数。
- **Macros / 宏**: Handles portability, visibility, or compile-time switches. / 处理可移植性、可见性或编译期开关。
- **Precomputed coefficients / 预计算系数**: Stores constants or tables used by numerical approximations. / 存储数值逼近使用的常量或查找表。

## Dependencies / 依赖关系
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/rational.hpp, boost/math/tools/promotion.hpp, boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, boost/math/constants/constants.hpp, boost/math/special_functions/math_fwd.hpp, boost/math/tools/series.hpp, boost/math/policies/error_handling.hpp, boost/math/constants/constants.hpp, boost/math/tools/big_constant.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `log, tools::evaluate_polynomial, digamma_series_func, s, boost::math::tools::sum_series, BOOST_MATH_BIG_CONSTANT, floor, tan, ...`
