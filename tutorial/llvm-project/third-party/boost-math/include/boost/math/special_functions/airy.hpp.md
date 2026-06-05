# airy.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/airy.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the airy special function and related helpers.
- **作用（中文）**: 此头文件实现 airy 特殊函数及相关辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // Copyright John Maddock 2012.
   2: // Copyright Matt Borland 2024.
   3: // Use, modification and distribution are subject to the
   4: // Boost Software License, Version 1.0.
   5: // (See accompanying file LICENSE_1_0.txt
   6: // or copy at http://www.boost.org/LICENSE_1_0.txt)
   7: 
   8: #ifndef BOOST_MATH_AIRY_HPP
   9: #define BOOST_MATH_AIRY_HPP
  10: 
  11: #include <boost/math/tools/config.hpp>
  12: #include <boost/math/tools/numeric_limits.hpp>
  13: #include <boost/math/tools/precision.hpp>
  14: #include <boost/math/tools/cstdint.hpp>
  15: #include <boost/math/special_functions/math_fwd.hpp>
  16: #include <boost/math/special_functions/bessel.hpp>
  17: #include <boost/math/special_functions/cbrt.hpp>
  18: #include <boost/math/special_functions/detail/airy_ai_bi_zero.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/precision.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/precision.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/tools/roots.hpp>
  20: #include <boost/math/policies/error_handling.hpp>
  21: #include <boost/math/constants/constants.hpp>
  22: 
  23: namespace boost{ namespace math{
  24: 
  25: namespace detail{
  26: 
  27: template <class T, class Policy>
  28: BOOST_MATH_GPU_ENABLED T airy_ai_imp(T x, const Policy& pol)
  29: {
  30:    BOOST_MATH_STD_USING
  31: 
  32:    if(x < 0)
  33:    {
  34:       T p = (-x * sqrt(-x) * 2) / 3;
  35:       T v = T(1) / 3;
  36:       T j1 = boost::math::cyl_bessel_j(v, p, pol);
~~~
- **EN:** This block imports dependencies such as boost/math/tools/roots.hpp, boost/math/policies/error_handling.hpp, boost/math/constants/constants.hpp so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/tools/roots.hpp, boost/math/policies/error_handling.hpp, boost/math/constants/constants.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:       T j2 = boost::math::cyl_bessel_j(-v, p, pol);
  38:       T ai = sqrt(-x) * (j1 + j2) / 3;
  39:       //T bi = sqrt(-x / 3) * (j2 - j1);
  40:       return ai;
  41:    }
  42:    else if(fabs(x * x * x) / 6 < tools::epsilon<T>())
  43:    {
  44:       T tg = boost::math::tgamma(constants::twothirds<T>(), pol);
  45:       T ai = 1 / (pow(T(3), constants::twothirds<T>()) * tg);
  46:       //T bi = 1 / (sqrt(boost::math::cbrt(T(3))) * tg);
  47:       return ai;
  48:    }
  49:    else
  50:    {
  51:       T p = 2 * x * sqrt(x) / 3;
  52:       T v = T(1) / 3;
  53:       //T j1 = boost::math::cyl_bessel_i(-v, p, pol);
  54:       //T j2 = boost::math::cyl_bessel_i(v, p, pol);
~~~
- **EN:** This range declares or defines callable logic such as boost::math::cyl_bessel_j, sqrt, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::cyl_bessel_j, sqrt, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:       //
  56:       // Note that although we can calculate ai from j1 and j2, the accuracy is horrible
  57:       // as we're subtracting two very large values, so use the Bessel K relation instead:
  58:       //
  59:       T ai = cyl_bessel_k(v, p, pol) * sqrt(x / 3) / boost::math::constants::pi<T>();  //sqrt(x) * (j1 - j2) / 3;
  60:       //T bi = sqrt(x / 3) * (j1 + j2);
  61:       return ai;
  62:    }
  63: }
  64: 
  65: template <class T, class Policy>
  66: BOOST_MATH_GPU_ENABLED T airy_bi_imp(T x, const Policy& pol)
  67: {
  68:    BOOST_MATH_STD_USING
  69: 
  70:    if(x < 0)
  71:    {
  72:       T p = (-x * sqrt(-x) * 2) / 3;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as cyl_bessel_k, sqrt.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 cyl_bessel_k, sqrt。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       T v = T(1) / 3;
  74:       T j1 = boost::math::cyl_bessel_j(v, p, pol);
  75:       T j2 = boost::math::cyl_bessel_j(-v, p, pol);
  76:       //T ai = sqrt(-x) * (j1 + j2) / 3;
  77:       T bi = sqrt(-x / 3) * (j2 - j1);
  78:       return bi;
  79:    }
  80:    else if(fabs(x * x * x) / 6 < tools::epsilon<T>())
  81:    {
  82:       T tg = boost::math::tgamma(constants::twothirds<T>(), pol);
  83:       //T ai = 1 / (pow(T(3), constants::twothirds<T>()) * tg);
  84:       T bi = 1 / (sqrt(boost::math::cbrt(T(3), pol)) * tg);
  85:       return bi;
  86:    }
  87:    else
  88:    {
  89:       T p = 2 * x * sqrt(x) / 3;
  90:       T v = T(1) / 3;
~~~
- **EN:** This range declares or defines callable logic such as boost::math::cyl_bessel_j, sqrt, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::cyl_bessel_j, sqrt, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:       T j1 = boost::math::cyl_bessel_i(-v, p, pol);
  92:       T j2 = boost::math::cyl_bessel_i(v, p, pol);
  93:       T bi = sqrt(x / 3) * (j1 + j2);
  94:       return bi;
  95:    }
  96: }
  97: 
  98: template <class T, class Policy>
  99: BOOST_MATH_GPU_ENABLED T airy_ai_prime_imp(T x, const Policy& pol)
 100: {
 101:    BOOST_MATH_STD_USING
 102: 
 103:    if(x < 0)
 104:    {
 105:       T p = (-x * sqrt(-x) * 2) / 3;
 106:       T v = T(2) / 3;
 107:       T j1 = boost::math::cyl_bessel_j(v, p, pol);
 108:       T j2 = boost::math::cyl_bessel_j(-v, p, pol);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::cyl_bessel_i, sqrt, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::cyl_bessel_i, sqrt, ...。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:       T aip = -x * (j1 - j2) / 3;
 110:       return aip;
 111:    }
 112:    else if(fabs(x * x) / 2 < tools::epsilon<T>())
 113:    {
 114:       T tg = boost::math::tgamma(constants::third<T>(), pol);
 115:       T aip = 1 / (boost::math::cbrt(T(3), pol) * tg);
 116:       return -aip;
 117:    }
 118:    else
 119:    {
 120:       T p = 2 * x * sqrt(x) / 3;
 121:       T v = T(2) / 3;
 122:       //T j1 = boost::math::cyl_bessel_i(-v, p, pol);
 123:       //T j2 = boost::math::cyl_bessel_i(v, p, pol);
 124:       //
 125:       // Note that although we can calculate ai from j1 and j2, the accuracy is horrible
 126:       // as we're subtracting two very large values, so use the Bessel K relation instead:
~~~
- **EN:** This range declares or defines callable logic such as boost::math::tgamma, boost::math::cbrt, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::tgamma, boost::math::cbrt, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:       //
 128:       T aip = -cyl_bessel_k(v, p, pol) * x / (boost::math::constants::root_three<T>() * boost::math::constants::pi<T>());
 129:       return aip;
 130:    }
 131: }
 132: 
 133: template <class T, class Policy>
 134: BOOST_MATH_GPU_ENABLED T airy_bi_prime_imp(T x, const Policy& pol)
 135: {
 136:    BOOST_MATH_STD_USING
 137: 
 138:    if(x < 0)
 139:    {
 140:       T p = (-x * sqrt(-x) * 2) / 3;
 141:       T v = T(2) / 3;
 142:       T j1 = boost::math::cyl_bessel_j(v, p, pol);
 143:       T j2 = boost::math::cyl_bessel_j(-v, p, pol);
 144:       T aip = -x * (j1 + j2) / constants::root_three<T>();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as cyl_bessel_k, boost::math::cyl_bessel_j.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 cyl_bessel_k, boost::math::cyl_bessel_j。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:       return aip;
 146:    }
 147:    else if(fabs(x * x) / 2 < tools::epsilon<T>())
 148:    {
 149:       T tg = boost::math::tgamma(constants::third<T>(), pol);
 150:       T bip = sqrt(boost::math::cbrt(T(3), pol)) / tg;
 151:       return bip;
 152:    }
 153:    else
 154:    {
 155:       T p = 2 * x * sqrt(x) / 3;
 156:       T v = T(2) / 3;
 157:       T j1 = boost::math::cyl_bessel_i(-v, p, pol);
 158:       T j2 = boost::math::cyl_bessel_i(v, p, pol);
 159:       T aip = x * (j1 + j2) / boost::math::constants::root_three<T>();
 160:       return aip;
 161:    }
 162: }
~~~
- **EN:** This range declares or defines callable logic such as boost::math::tgamma, boost::math::cyl_bessel_i. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::tgamma, boost::math::cyl_bessel_i。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163: 
 164: template <class T, class Policy>
 165: BOOST_MATH_GPU_ENABLED T airy_ai_zero_imp(int m, const Policy& pol)
 166: {
 167:    BOOST_MATH_STD_USING // ADL of std names, needed for log, sqrt.
 168: 
 169:    // Handle cases when a negative zero (negative rank) is requested.
 170:    if(m < 0)
 171:    {
 172:       return policies::raise_domain_error<T>("boost::math::airy_ai_zero<%1%>(%1%, int)",
 173:          "Requested the %1%'th zero, but the rank must be 1 or more !", static_cast<T>(m), pol);
 174:    }
 175: 
 176:    // Handle case when the zero'th zero is requested.
 177:    if(m == 0U)
 178:    {
 179:       return policies::raise_domain_error<T>("boost::math::airy_ai_zero<%1%>(%1%,%1%)",
 180:         "The requested rank of the zero is %1%, but must be 1 or more !", static_cast<T>(m), pol);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:    }
 182: 
 183:    // Set up the initial guess for the upcoming root-finding.
 184:    const T guess_root = boost::math::detail::airy_zero::airy_ai_zero_detail::initial_guess<T>(m, pol);
 185: 
 186:    // Select the maximum allowed iterations based on the number
 187:    // of decimal digits in the numeric type T, being at least 12.
 188:    const int my_digits10 = static_cast<int>(static_cast<float>(policies::digits<T, Policy>() * 0.301F));
 189: 
 190:    const std::uintmax_t iterations_allowed = static_cast<std::uintmax_t>(BOOST_MATH_GPU_SAFE_MAX(12, my_digits10 * 2));
 191: 
 192:    std::uintmax_t iterations_used = iterations_allowed;
 193: 
 194:    // Use a dynamic tolerance because the roots get closer the higher m gets.
 195:    T tolerance;  // LCOV_EXCL_LINE
 196: 
 197:    if     (m <=   10) { tolerance = T(0.3F); }
 198:    else if(m <=  100) { tolerance = T(0.1F); }
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_MAX, if. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_MAX, if。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:    else if(m <= 1000) { tolerance = T(0.05F); }
 200:    else               { tolerance = T(1) / sqrt(T(m)); }
 201: 
 202:    // Perform the root-finding using Newton-Raphson iteration from Boost.Math.
 203:    const T am =
 204:       boost::math::tools::newton_raphson_iterate(
 205:          boost::math::detail::airy_zero::airy_ai_zero_detail::function_object_ai_and_ai_prime<T, Policy>(pol),
 206:          guess_root,
 207:          T(guess_root - tolerance),
 208:          T(guess_root + tolerance),
 209:          policies::digits<T, Policy>(),
 210:          iterations_used);
 211: 
 212:    static_cast<void>(iterations_used);
 213: 
 214:    return am;
 215: }
 216: 
~~~
- **EN:** This range declares or defines callable logic such as if, T. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 if, T。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217: template <class T, class Policy>
 218: BOOST_MATH_GPU_ENABLED T airy_bi_zero_imp(int m, const Policy& pol)
 219: {
 220:    BOOST_MATH_STD_USING // ADL of std names, needed for log, sqrt.
 221: 
 222:    // Handle cases when a negative zero (negative rank) is requested.
 223:    if(m < 0)
 224:    {
 225:       return policies::raise_domain_error<T>("boost::math::airy_bi_zero<%1%>(%1%, int)",
 226:          "Requested the %1%'th zero, but the rank must 1 or more !", static_cast<T>(m), pol);
 227:    }
 228: 
 229:    // Handle case when the zero'th zero is requested.
 230:    if(m == 0U)
 231:    {
 232:       return policies::raise_domain_error<T>("boost::math::airy_bi_zero<%1%>(%1%,%1%)",
 233:         "The requested rank of the zero is %1%, but must be 1 or more !", static_cast<T>(m), pol);
 234:    }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:    // Set up the initial guess for the upcoming root-finding.
 236:    const T guess_root = boost::math::detail::airy_zero::airy_bi_zero_detail::initial_guess<T>(m, pol);
 237: 
 238:    // Select the maximum allowed iterations based on the number
 239:    // of decimal digits in the numeric type T, being at least 12.
 240:    const int my_digits10 = static_cast<int>(static_cast<float>(policies::digits<T, Policy>() * 0.301F));
 241: 
 242:    const std::uintmax_t iterations_allowed = static_cast<std::uintmax_t>(BOOST_MATH_GPU_SAFE_MAX(12, my_digits10 * 2));
 243: 
 244:    std::uintmax_t iterations_used = iterations_allowed;
 245: 
 246:    // Use a dynamic tolerance because the roots get closer the higher m gets.
 247:    T tolerance; // LCOV_EXCL_LINE
 248: 
 249:    if     (m <=   10) { tolerance = T(0.3F); }
 250:    else if(m <=  100) { tolerance = T(0.1F); }
 251:    else if(m <= 1000) { tolerance = T(0.05F); }
 252:    else               { tolerance = T(1) / sqrt(T(m)); }
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_MAX, if, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_MAX, if, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253: 
 254:    // Perform the root-finding using Newton-Raphson iteration from Boost.Math.
 255:    const T bm =
 256:       boost::math::tools::newton_raphson_iterate(
 257:          boost::math::detail::airy_zero::airy_bi_zero_detail::function_object_bi_and_bi_prime<T, Policy>(pol),
 258:          guess_root,
 259:          T(guess_root - tolerance),
 260:          T(guess_root + tolerance),
 261:          policies::digits<T, Policy>(),
 262:          iterations_used);
 263: 
 264:    static_cast<void>(iterations_used);
 265: 
 266:    return bm;
 267: }
 268: 
 269: } // namespace detail
 270: 
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Return statements hand the computed result or status back to the caller.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271: template <class T, class Policy>
 272: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_ai(T x, const Policy&)
 273: {
 274:    BOOST_FPU_EXCEPTION_GUARD
 275:    typedef typename tools::promote_args<T>::type result_type;
 276:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
 277:    typedef typename policies::normalise<
 278:       Policy,
 279:       policies::promote_float<false>,
 280:       policies::promote_double<false>,
 281:       policies::discrete_quantile<>,
 282:       policies::assert_undefined<> >::type forwarding_policy;
 283: 
 284:    return policies::checked_narrowing_cast<result_type, Policy>(detail::airy_ai_imp<value_type>(static_cast<value_type>(x), forwarding_policy()), "boost::math::airy<%1%>(%1%)");
 285: }
 286: 
 287: template <class T>
 288: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_ai(T x)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289: {
 290:    return airy_ai(x, policies::policy<>());
 291: }
 292: 
 293: template <class T, class Policy>
 294: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_bi(T x, const Policy&)
 295: {
 296:    BOOST_FPU_EXCEPTION_GUARD
 297:    typedef typename tools::promote_args<T>::type result_type;
 298:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
 299:    typedef typename policies::normalise<
 300:       Policy,
 301:       policies::promote_float<false>,
 302:       policies::promote_double<false>,
 303:       policies::discrete_quantile<>,
 304:       policies::assert_undefined<> >::type forwarding_policy;
 305: 
 306:    return policies::checked_narrowing_cast<result_type, Policy>(detail::airy_bi_imp<value_type>(static_cast<value_type>(x), forwarding_policy()), "boost::math::airy<%1%>(%1%)");
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307: }
 308: 
 309: template <class T>
 310: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_bi(T x)
 311: {
 312:    return airy_bi(x, policies::policy<>());
 313: }
 314: 
 315: template <class T, class Policy>
 316: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_ai_prime(T x, const Policy&)
 317: {
 318:    BOOST_FPU_EXCEPTION_GUARD
 319:    typedef typename tools::promote_args<T>::type result_type;
 320:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
 321:    typedef typename policies::normalise<
 322:       Policy,
 323:       policies::promote_float<false>,
 324:       policies::promote_double<false>,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:       policies::discrete_quantile<>,
 326:       policies::assert_undefined<> >::type forwarding_policy;
 327: 
 328:    return policies::checked_narrowing_cast<result_type, Policy>(detail::airy_ai_prime_imp<value_type>(static_cast<value_type>(x), forwarding_policy()), "boost::math::airy<%1%>(%1%)");
 329: }
 330: 
 331: template <class T>
 332: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_ai_prime(T x)
 333: {
 334:    return airy_ai_prime(x, policies::policy<>());
 335: }
 336: 
 337: template <class T, class Policy>
 338: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_bi_prime(T x, const Policy&)
 339: {
 340:    BOOST_FPU_EXCEPTION_GUARD
 341:    typedef typename tools::promote_args<T>::type result_type;
 342:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:    typedef typename policies::normalise<
 344:       Policy,
 345:       policies::promote_float<false>,
 346:       policies::promote_double<false>,
 347:       policies::discrete_quantile<>,
 348:       policies::assert_undefined<> >::type forwarding_policy;
 349: 
 350:    return policies::checked_narrowing_cast<result_type, Policy>(detail::airy_bi_prime_imp<value_type>(static_cast<value_type>(x), forwarding_policy()), "boost::math::airy<%1%>(%1%)");
 351: }
 352: 
 353: template <class T>
 354: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type airy_bi_prime(T x)
 355: {
 356:    return airy_bi_prime(x, policies::policy<>());
 357: }
 358: 
 359: template <class T, class Policy>
 360: BOOST_MATH_GPU_ENABLED inline T airy_ai_zero(int m, const Policy& /*pol*/)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361: {
 362:    BOOST_FPU_EXCEPTION_GUARD
 363:    typedef typename policies::evaluation<T, Policy>::type value_type;
 364:    typedef typename policies::normalise<
 365:       Policy,
 366:       policies::promote_float<false>,
 367:       policies::promote_double<false>,
 368:       policies::discrete_quantile<>,
 369:       policies::assert_undefined<> >::type forwarding_policy;
 370: 
 371:    static_assert(    false == std::numeric_limits<T>::is_specialized
 372:                            || (   true  == std::numeric_limits<T>::is_specialized
 373:                                && false == std::numeric_limits<T>::is_integer),
 374:                            "Airy value type must be a floating-point type.");
 375: 
 376:    return policies::checked_narrowing_cast<T, Policy>(detail::airy_ai_zero_imp<value_type>(m, forwarding_policy()), "boost::math::airy_ai_zero<%1%>(unsigned)");
 377: }
 378: 
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379: template <class T>
 380: BOOST_MATH_GPU_ENABLED inline T airy_ai_zero(int m)
 381: {
 382:    return airy_ai_zero<T>(m, policies::policy<>());
 383: }
 384: 
 385: template <class T, class OutputIterator, class Policy>
 386: BOOST_MATH_GPU_ENABLED inline OutputIterator airy_ai_zero(
 387:                          int start_index,
 388:                          unsigned number_of_zeros,
 389:                          OutputIterator out_it,
 390:                          const Policy& pol)
 391: {
 392:    typedef T result_type;
 393: 
 394:    static_assert(    false == std::numeric_limits<T>::is_specialized
 395:                            || (   true  == std::numeric_limits<T>::is_specialized
 396:                                && false == std::numeric_limits<T>::is_integer),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:                            "Airy value type must be a floating-point type.");
 398: 
 399:    for(unsigned i = 0; i < number_of_zeros; ++i)
 400:    {
 401:       *out_it = boost::math::airy_ai_zero<result_type>(start_index + i, pol);
 402:       ++out_it;
 403:    }
 404:    return out_it;
 405: }
 406: 
 407: template <class T, class OutputIterator>
 408: BOOST_MATH_GPU_ENABLED inline OutputIterator airy_ai_zero(
 409:                          int start_index,
 410:                          unsigned number_of_zeros,
 411:                          OutputIterator out_it)
 412: {
 413:    return airy_ai_zero<T>(start_index, number_of_zeros, out_it, policies::policy<>());
 414: }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415: 
 416: template <class T, class Policy>
 417: BOOST_MATH_GPU_ENABLED inline T airy_bi_zero(int m, const Policy& /*pol*/)
 418: {
 419:    BOOST_FPU_EXCEPTION_GUARD
 420:    typedef typename policies::evaluation<T, Policy>::type value_type;
 421:    typedef typename policies::normalise<
 422:       Policy,
 423:       policies::promote_float<false>,
 424:       policies::promote_double<false>,
 425:       policies::discrete_quantile<>,
 426:       policies::assert_undefined<> >::type forwarding_policy;
 427: 
 428:    static_assert(    false == std::numeric_limits<T>::is_specialized
 429:                            || (   true  == std::numeric_limits<T>::is_specialized
 430:                                && false == std::numeric_limits<T>::is_integer),
 431:                            "Airy value type must be a floating-point type.");
 432: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:    return policies::checked_narrowing_cast<T, Policy>(detail::airy_bi_zero_imp<value_type>(m, forwarding_policy()), "boost::math::airy_bi_zero<%1%>(unsigned)");
 434: }
 435: 
 436: template <typename T>
 437: BOOST_MATH_GPU_ENABLED inline T airy_bi_zero(int m)
 438: {
 439:    return airy_bi_zero<T>(m, policies::policy<>());
 440: }
 441: 
 442: template <class T, class OutputIterator, class Policy>
 443: BOOST_MATH_GPU_ENABLED inline OutputIterator airy_bi_zero(
 444:                          int start_index,
 445:                          unsigned number_of_zeros,
 446:                          OutputIterator out_it,
 447:                          const Policy& pol)
 448: {
 449:    typedef T result_type;
 450: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:    static_assert(    false == std::numeric_limits<T>::is_specialized
 452:                            || (   true  == std::numeric_limits<T>::is_specialized
 453:                                && false == std::numeric_limits<T>::is_integer),
 454:                            "Airy value type must be a floating-point type.");
 455: 
 456:    for(unsigned i = 0; i < number_of_zeros; ++i)
 457:    {
 458:       *out_it = boost::math::airy_bi_zero<result_type>(start_index + i, pol);
 459:       ++out_it;
 460:    }
 461:    return out_it;
 462: }
 463: 
 464: template <class T, class OutputIterator>
 465: BOOST_MATH_GPU_ENABLED inline OutputIterator airy_bi_zero(
 466:                          int start_index,
 467:                          unsigned number_of_zeros,
 468:                          OutputIterator out_it)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 469-475 / 第 469-475 行
~~~cpp
 469: {
 470:    return airy_bi_zero<T>(start_index, number_of_zeros, out_it, policies::policy<>());
 471: }
 472: 
 473: }} // namespaces
 474: 
 475: #endif // BOOST_MATH_AIRY_HPP
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/precision.hpp, boost/math/tools/cstdint.hpp, boost/math/special_functions/math_fwd.hpp, boost/math/special_functions/bessel.hpp, boost/math/special_functions/cbrt.hpp, boost/math/special_functions/detail/airy_ai_bi_zero.hpp, boost/math/tools/roots.hpp, boost/math/policies/error_handling.hpp, boost/math/constants/constants.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `boost::math::cyl_bessel_j, sqrt, boost::math::tgamma, pow, boost::math::cyl_bessel_i, cyl_bessel_k, boost::math::cbrt, BOOST_MATH_GPU_SAFE_MAX, ...`
