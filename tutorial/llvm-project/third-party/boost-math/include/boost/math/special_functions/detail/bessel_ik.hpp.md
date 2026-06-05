# bessel_ik.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/bessel_ik.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the bessel ik special-function path.
- **作用（中文）**: 此头文件为 bessel ik 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  Copyright (c) 2006 Xiaogang Zhang
   2: //  Copyright (c) 2024 Matt Borland
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_BESSEL_IK_HPP
   8: #define BOOST_MATH_BESSEL_IK_HPP
   9: 
  10: #ifdef _MSC_VER
  11: #pragma once
  12: #endif
  13: 
  14: #include <boost/math/tools/config.hpp>
  15: #include <boost/math/tools/cstdint.hpp>
  16: #include <boost/math/tools/numeric_limits.hpp>
  17: #include <boost/math/tools/type_traits.hpp>
  18: #include <boost/math/tools/series.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/numeric_limits.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/numeric_limits.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/special_functions/sign.hpp>
  20: #include <boost/math/special_functions/round.hpp>
  21: #include <boost/math/special_functions/gamma.hpp>
  22: #include <boost/math/special_functions/sin_pi.hpp>
  23: #include <boost/math/constants/constants.hpp>
  24: #include <boost/math/policies/error_handling.hpp>
  25: 
  26: // Modified Bessel functions of the first and second kind of fractional order
  27: 
  28: namespace boost { namespace math {
  29: 
  30: namespace detail {
  31: 
  32: template <class T, class Policy>
  33: struct cyl_bessel_i_small_z
  34: {
  35:    typedef T result_type;
  36: 
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/sign.hpp, boost/math/special_functions/round.hpp, boost/math/special_functions/gamma.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/special_functions/sign.hpp, boost/math/special_functions/round.hpp, boost/math/special_functions/gamma.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:    BOOST_MATH_GPU_ENABLED cyl_bessel_i_small_z(T v_, T z_) : k(0), v(v_), mult(z_*z_/4)
  38:    {
  39:       BOOST_MATH_STD_USING
  40:       term = 1;
  41:    }
  42: 
  43:    BOOST_MATH_GPU_ENABLED T operator()()
  44:    {
  45:       T result = term;
  46:       ++k;
  47:       term *= mult / k;
  48:       term /= k + v;
  49:       return result;
  50:    }
  51: private:
  52:    unsigned k;
  53:    T v;
  54:    T term;
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:    T mult;
  56: };
  57: 
  58: template <class T, class Policy>
  59: BOOST_MATH_GPU_ENABLED inline T bessel_i_small_z_series(T v, T x, const Policy& pol)
  60: {
  61:    BOOST_MATH_STD_USING
  62:    T prefix;
  63:    if(v < max_factorial<T>::value)
  64:    {
  65:       prefix = pow(x / 2, v) / boost::math::tgamma(v + 1, pol);
  66:    }
  67:    else
  68:    {
  69:       prefix = v * log(x / 2) - boost::math::lgamma(v + 1, pol);
  70:       prefix = exp(prefix);
  71:    }
  72:    if(prefix == 0)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as pow, log, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 pow, log, ...。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       return prefix;
  74: 
  75:    cyl_bessel_i_small_z<T, Policy> s(v, x);
  76:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
  77: 
  78:    T result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
  79: 
  80:    policies::check_series_iterations<T>("boost::math::bessel_j_small_z_series<%1%>(%1%,%1%)", max_iter, pol);
  81:    return prefix * result;
  82: }
  83: 
  84: // Calculate K(v, x) and K(v+1, x) by method analogous to
  85: // Temme, Journal of Computational Physics, vol 21, 343 (1976)
  86: template <typename T, typename Policy>
  87: BOOST_MATH_GPU_ENABLED int temme_ik(T v, T x, T* result_K, T* K1, const Policy& pol)
  88: {
  89:     T f, h, p, q, coef, sum, sum1, tolerance;
  90:     T a, b, c, d, sigma, gamma1, gamma2;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as s, boost::math::tools::sum_series. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 s, boost::math::tools::sum_series。 return 语句会把计算结果或状态返回给调用方。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:     unsigned long k;
  92: 
  93:     BOOST_MATH_STD_USING
  94:     using namespace boost::math::tools;
  95:     using namespace boost::math::constants;
  96: 
  97: 
  98:     // |x| <= 2, Temme series converge rapidly
  99:     // |x| > 2, the larger the |x|, the slower the convergence
 100:     BOOST_MATH_ASSERT(abs(x) <= 2);
 101:     BOOST_MATH_ASSERT(abs(v) <= 0.5f);
 102: 
 103:     T gp = boost::math::tgamma1pm1(v, pol);
 104:     T gm = boost::math::tgamma1pm1(-v, pol);
 105: 
 106:     a = log(x / 2);
 107:     b = exp(v * a);
 108:     sigma = -a * v;
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. This range declares or defines callable logic such as BOOST_MATH_ASSERT, boost::math::tgamma1pm1, .... Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, boost::math::tgamma1pm1, ...。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:     c = abs(v) < tools::epsilon<T>() ?
 110:        T(1) : T(boost::math::sin_pi(v, pol) / (v * pi<T>()));
 111:     d = abs(sigma) < tools::epsilon<T>() ?
 112:         T(1) : T(sinh(sigma) / sigma);
 113:     gamma1 = abs(v) < tools::epsilon<T>() ?
 114:         T(-euler<T>()) : T((0.5f / v) * (gp - gm) * c);
 115:     gamma2 = (2 + gp + gm) * c / 2;
 116: 
 117:     // initial values
 118:     p = (gp + 1) / (2 * b);
 119:     q = (1 + gm) * b / 2;
 120:     f = (cosh(sigma) * gamma1 + d * (-a) * gamma2) / c;
 121:     h = p;
 122:     coef = 1;
 123:     sum = coef * f;
 124:     sum1 = coef * h;
 125: 
 126:     BOOST_MATH_INSTRUMENT_VARIABLE(p);
~~~
- **EN:** This range declares or defines callable logic such as T, BOOST_MATH_INSTRUMENT_VARIABLE. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The logic belongs to a numerical-integration path, so weights, nodes, or convergence rules are central.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T, BOOST_MATH_INSTRUMENT_VARIABLE。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该逻辑属于数值积分路径，因此权重、节点或收敛规则是核心内容。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:     BOOST_MATH_INSTRUMENT_VARIABLE(q);
 128:     BOOST_MATH_INSTRUMENT_VARIABLE(f);
 129:     BOOST_MATH_INSTRUMENT_VARIABLE(sigma);
 130:     BOOST_MATH_INSTRUMENT_CODE(sinh(sigma));
 131:     BOOST_MATH_INSTRUMENT_VARIABLE(gamma1);
 132:     BOOST_MATH_INSTRUMENT_VARIABLE(gamma2);
 133:     BOOST_MATH_INSTRUMENT_VARIABLE(c);
 134:     BOOST_MATH_INSTRUMENT_VARIABLE(d);
 135:     BOOST_MATH_INSTRUMENT_VARIABLE(a);
 136: 
 137:     // series summation
 138:     tolerance = tools::epsilon<T>();
 139:     for (k = 1; k < policies::get_max_series_iterations<Policy>(); k++)
 140:     {
 141:         f = (k * f + p + q) / (k*k - v*v);
 142:         p /= k - v;
 143:         q /= k + v;
 144:         h = p - k * f;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_INSTRUMENT_CODE. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_INSTRUMENT_CODE。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:         coef *= x * x / (4 * k);
 146:         sum += coef * f;
 147:         sum1 += coef * h;
 148:         if (abs(coef * f) < abs(sum) * tolerance)
 149:         {
 150:            break;
 151:         }
 152:     }
 153:     policies::check_series_iterations<T>("boost::math::bessel_ik<%1%>(%1%,%1%) in temme_ik", k, pol);
 154: 
 155:     *result_K = sum;
 156:     *K1 = 2 * sum1 / x;
 157: 
 158:     return 0;
 159: }
 160: 
 161: // Evaluate continued fraction fv = I_(v+1) / I_v, derived from
 162: // Abramowitz and Stegun, Handbook of Mathematical Functions, 1972, 9.1.73
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163: template <typename T, typename Policy>
 164: BOOST_MATH_GPU_ENABLED int CF1_ik(T v, T x, T* fv, const Policy& pol)
 165: {
 166:     T C, D, f, a, b, delta, tiny, tolerance;
 167:     unsigned long k;
 168: 
 169:     BOOST_MATH_STD_USING
 170: 
 171:     // |x| <= |v|, CF1_ik converges rapidly
 172:     // |x| > |v|, CF1_ik needs O(|x|) iterations to converge
 173: 
 174:     // modified Lentz's method, see
 175:     // Lentz, Applied Optics, vol 15, 668 (1976)
 176:     tolerance = 2 * tools::epsilon<T>();
 177:     BOOST_MATH_INSTRUMENT_VARIABLE(tolerance);
 178:     tiny = sqrt(tools::min_value<T>());
 179:     BOOST_MATH_INSTRUMENT_VARIABLE(tiny);
 180:     C = f = tiny;                           // b0 = 0, replace with tiny
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, sqrt. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, sqrt。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:     D = 0;
 182:     for (k = 1; k < policies::get_max_series_iterations<Policy>(); k++)
 183:     {
 184:         a = 1;
 185:         b = 2 * (v + k) / x;
 186:         C = b + a / C;
 187:         D = b + a * D;
 188:         if (C == 0) { C = tiny; }
 189:         if (D == 0) { D = tiny; }
 190:         D = 1 / D;
 191:         delta = C * D;
 192:         f *= delta;
 193:         BOOST_MATH_INSTRUMENT_VARIABLE(delta-1);
 194:         if (abs(delta - 1) <= tolerance)
 195:         {
 196:            break;
 197:         }
 198:     }
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:     BOOST_MATH_INSTRUMENT_VARIABLE(k);
 200:     policies::check_series_iterations<T>("boost::math::bessel_ik<%1%>(%1%,%1%) in CF1_ik", k, pol);
 201: 
 202:     *fv = f;
 203: 
 204:     return 0;
 205: }
 206: 
 207: // Calculate K(v, x) and K(v+1, x) by evaluating continued fraction
 208: // z1 / z0 = U(v+1.5, 2v+1, 2x) / U(v+0.5, 2v+1, 2x), see
 209: // Thompson and Barnett, Computer Physics Communications, vol 47, 245 (1987)
 210: template <typename T, typename Policy>
 211: BOOST_MATH_GPU_ENABLED int CF2_ik(T v, T x, T* Kv, T* Kv1, const Policy& pol)
 212: {
 213:     BOOST_MATH_STD_USING
 214:     using namespace boost::math::constants;
 215: 
 216:     T S, C, Q, D, f, a, b, q, delta, tolerance, current, prev;
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:     unsigned long k;
 218: 
 219:     // |x| >= |v|, CF2_ik converges rapidly
 220:     // |x| -> 0, CF2_ik fails to converge
 221: 
 222:     BOOST_MATH_ASSERT(abs(x) > 1);
 223: 
 224:     // Steed's algorithm, see Thompson and Barnett,
 225:     // Journal of Computational Physics, vol 64, 490 (1986)
 226:     tolerance = tools::epsilon<T>();
 227:     a = v * v - 0.25f;
 228:     b = 2 * (x + 1);                              // b1
 229:     D = 1 / b;                                    // D1 = 1 / b1
 230:     f = delta = D;                                // f1 = delta1 = D1, coincidence
 231:     prev = 0;                                     // q0
 232:     current = 1;                                  // q1
 233:     Q = C = -a;                                   // Q1 = C1 because q1 = 1
 234:     S = 1 + Q * delta;                            // S1
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:     BOOST_MATH_INSTRUMENT_VARIABLE(tolerance);
 236:     BOOST_MATH_INSTRUMENT_VARIABLE(a);
 237:     BOOST_MATH_INSTRUMENT_VARIABLE(b);
 238:     BOOST_MATH_INSTRUMENT_VARIABLE(D);
 239:     BOOST_MATH_INSTRUMENT_VARIABLE(f);
 240: 
 241:     for (k = 2; k < policies::get_max_series_iterations<Policy>(); k++)     // starting from 2
 242:     {
 243:         // continued fraction f = z1 / z0
 244:         a -= 2 * (k - 1);
 245:         b += 2;
 246:         D = 1 / (b + a * D);
 247:         delta *= b * D - 1;
 248:         f += delta;
 249: 
 250:         // series summation S = 1 + \sum_{n=1}^{\infty} C_n * z_n / z_0
 251:         q = (prev - (b - 2) * current) / a;
 252:         prev = current;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:         current = q;                        // forward recurrence for q
 254:         C *= -a / k;
 255:         Q += C * q;
 256:         S += Q * delta;
 257:         //
 258:         // Under some circumstances q can grow very small and C very
 259:         // large, leading to under/overflow.  This is particularly an
 260:         // issue for types which have many digits precision but a narrow
 261:         // exponent range.  A typical example being a "double double" type.
 262:         // To avoid this situation we can normalise q (and related prev/current)
 263:         // and C.  All other variables remain unchanged in value.  A typical
 264:         // test case occurs when x is close to 2, for example cyl_bessel_k(9.125, 2.125).
 265:         //
 266:         if(q < tools::epsilon<T>())
 267:         {
 268:            C *= q;
 269:            prev /= q;
 270:            current /= q;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:            q = 1;
 272:         }
 273: 
 274:         // S converges slower than f
 275:         BOOST_MATH_INSTRUMENT_VARIABLE(Q * delta);
 276:         BOOST_MATH_INSTRUMENT_VARIABLE(abs(S) * tolerance);
 277:         BOOST_MATH_INSTRUMENT_VARIABLE(S);
 278:         if (abs(Q * delta) < abs(S) * tolerance)
 279:         {
 280:            break;
 281:         }
 282:     }
 283:     policies::check_series_iterations<T>("boost::math::bessel_ik<%1%>(%1%,%1%) in CF2_ik", k, pol);
 284: 
 285:     if(-x < tools::log_min_value<T>())
 286:        *Kv = exp(0.5f * log(pi<T>() / (2 * x)) - x - log(S));
 287:     else
 288:       *Kv = sqrt(pi<T>() / (2 * x)) * exp(-x) / S;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:     *Kv1 = *Kv * (0.5f + v + x + (v * v - 0.25f) * f) / x;
 290:     BOOST_MATH_INSTRUMENT_VARIABLE(*Kv);
 291:     BOOST_MATH_INSTRUMENT_VARIABLE(*Kv1);
 292: 
 293:     return 0;
 294: }
 295: 
 296: enum{
 297:    need_i = 1,
 298:    need_k = 2
 299: };
 300: 
 301: // Compute I(v, x) and K(v, x) simultaneously by Temme's method, see
 302: // Temme, Journal of Computational Physics, vol 19, 324 (1975)
 303: template <typename T, typename Policy>
 304: BOOST_MATH_GPU_ENABLED int bessel_ik(T v, T x, T* result_I, T* result_K, int kind, const Policy& pol)
 305: {
 306:     // Kv1 = K_(v+1), fv = I_(v+1) / I_v
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 return 语句会把计算结果或状态返回给调用方。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:     // Ku1 = K_(u+1), fu = I_(u+1) / I_u
 308:     T u, Iv, Kv, Kv1, Ku, Ku1, fv;
 309:     T W, current, prev, next;
 310:     bool reflect = false;
 311:     unsigned n, k;
 312:     int org_kind = kind;
 313:     BOOST_MATH_INSTRUMENT_VARIABLE(v);
 314:     BOOST_MATH_INSTRUMENT_VARIABLE(x);
 315:     BOOST_MATH_INSTRUMENT_VARIABLE(kind);
 316: 
 317:     BOOST_MATH_STD_USING
 318:     using namespace boost::math::tools;
 319:     using namespace boost::math::constants;
 320: 
 321:     constexpr auto function = "boost::math::bessel_ik<%1%>(%1%,%1%)";
 322: 
 323:     if (v < 0)
 324:     {
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:         reflect = true;
 326:         v = -v;                             // v is non-negative from here
 327:         kind |= need_k;
 328:     }
 329: 
 330:     T scale = 1;
 331:     T scale_sign = 1;
 332: 
 333:     n = iround(v, pol);
 334:     u = v - n;                              // -1/2 <= u < 1/2
 335:     BOOST_MATH_INSTRUMENT_VARIABLE(n);
 336:     BOOST_MATH_INSTRUMENT_VARIABLE(u);
 337: 
 338:     if (((kind & need_i) == 0) && (fabs(4 * v * v - 25) / (8 * x) < tools::forth_root_epsilon<T>()))
 339:     {
 340:        // A&S 9.7.2
 341:        Iv = boost::math::numeric_limits<T>::quiet_NaN(); // any value will do
 342:        T mu = 4 * v * v;
~~~
- **EN:** This range declares or defines callable logic such as iround, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 iround, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:        T eight_z = 8 * x;
 344:        Kv = 1 + (mu - 1) / eight_z + (mu - 1) * (mu - 9) / (2 * eight_z * eight_z) + (mu - 1) * (mu - 9) * (mu - 25) / (6 * eight_z * eight_z * eight_z);
 345:        Kv *= exp(-x) * constants::root_pi<T>() / sqrt(2 * x);
 346:     }
 347:     else
 348:     {
 349:        BOOST_MATH_ASSERT(x > 0); // Error handling for x <= 0 handled in cyl_bessel_i and cyl_bessel_k
 350: 
 351:        // x is positive until reflection
 352:        W = 1 / x;                                 // Wronskian
 353:        if (x <= 2)                                // x in (0, 2]
 354:        {
 355:           temme_ik(u, x, &Ku, &Ku1, pol);             // Temme series
 356:        }
 357:        else                                       // x in (2, \infty)
 358:        {
 359:           CF2_ik(u, x, &Ku, &Ku1, pol);               // continued fraction CF2_ik
 360:        }
~~~
- **EN:** This range declares or defines callable logic such as exp, BOOST_MATH_ASSERT, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp, BOOST_MATH_ASSERT, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:        BOOST_MATH_INSTRUMENT_VARIABLE(Ku);
 362:        BOOST_MATH_INSTRUMENT_VARIABLE(Ku1);
 363:        prev = Ku;
 364:        current = Ku1;
 365:        for (k = 1; k <= n; k++)                   // forward recurrence for K
 366:        {
 367:           T fact = 2 * (u + k) / x;
 368:           // Check for overflow: if (max - |prev|) / fact > max, then overflow
 369:           // (max - |prev|) / fact > max
 370:           // max * (1 - fact) > |prev|
 371:           // if fact < 1: safe to compute overflow check
 372:           // if fact >= 1:  won't overflow
 373:           const bool will_overflow = (fact < 1)
 374:              ? tools::max_value<T>() * (1 - fact) > fabs(prev)
 375:              : false;
 376:           if (!will_overflow && ((tools::max_value<T>() - fabs(prev)) / fact < fabs(current)))
 377:           {
 378:              prev /= current;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:              scale /= current;
 380:              scale_sign *= ((boost::math::signbit)(current) ? -1 : 1);
 381:              current = 1;
 382:           }
 383:           next = fact * current + prev;
 384:           prev = current;
 385:           current = next;
 386:        }
 387:        Kv = prev;
 388:        Kv1 = current;
 389:        BOOST_MATH_INSTRUMENT_VARIABLE(Kv);
 390:        BOOST_MATH_INSTRUMENT_VARIABLE(Kv1);
 391:        if (kind & need_i)
 392:        {
 393:           T lim = (4 * v * v + 10) / (8 * x);
 394:           lim *= lim;
 395:           lim *= lim;
 396:           lim /= 24;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:           if ((lim < tools::epsilon<T>() * 10) && (x > 100))
 398:           {
 399:              // x is huge compared to v, CF1 may be very slow
 400:              // to converge so use asymptotic expansion for large
 401:              // x case instead.  Note that the asymptotic expansion
 402:              // isn't very accurate - so it's deliberately very hard
 403:              // to get here - probably we're going to overflow:
 404:              Iv = asymptotic_bessel_i_large_x(v, x, pol);
 405:           }
 406:           else if ((v > 0) && (x / v < 0.25))
 407:           {
 408:              Iv = bessel_i_small_z_series(v, x, pol);
 409:           }
 410:           else
 411:           {
 412:              CF1_ik(v, x, &fv, pol);                         // continued fraction CF1_ik
 413:              Iv = scale * W / (Kv * fv + Kv1);                  // Wronskian relation
 414:           }
~~~
- **EN:** This range declares or defines callable logic such as asymptotic_bessel_i_large_x, bessel_i_small_z_series, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 asymptotic_bessel_i_large_x, bessel_i_small_z_series, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:        }
 416:        else
 417:           Iv = boost::math::numeric_limits<T>::quiet_NaN(); // any value will do
 418:     }
 419:     if (reflect && (kind & need_i))
 420:     {
 421:         BOOST_MATH_ASSERT(fabs(v - n - u) < tools::forth_root_epsilon<T>());
 422:         T z = (u + n % 2);
 423:         T fact = (2 / pi<T>()) * (boost::math::sin_pi(z, pol) * Kv);
 424:         if(fact == 0)
 425:            *result_I = Iv;
 426:         else if(tools::max_value<T>() * scale < fact)
 427:            *result_I = (org_kind & need_i) ? T(sign(fact) * scale_sign * policies::raise_overflow_error<T>(function, nullptr, pol)) : T(0);
 428:         else
 429:          *result_I = Iv + fact / scale;   // reflection formula
 430:     }
 431:     else
 432:     {
~~~
- **EN:** This range declares or defines callable logic such as quiet_NaN, BOOST_MATH_ASSERT, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 quiet_NaN, BOOST_MATH_ASSERT, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 433-447 / 第 433-447 行
~~~cpp
 433:         *result_I = Iv;
 434:     }
 435:     if(tools::max_value<T>() * scale < Kv)
 436:        *result_K = (org_kind & need_k) ? T(sign(Kv) * scale_sign * policies::raise_overflow_error<T>(function, nullptr, pol)) : T(0);
 437:     else
 438:       *result_K = Kv / scale;
 439:     BOOST_MATH_INSTRUMENT_VARIABLE(*result_I);
 440:     BOOST_MATH_INSTRUMENT_VARIABLE(*result_K);
 441:     return 0;
 442: }
 443: 
 444: }}} // namespaces
 445: 
 446: #endif // BOOST_MATH_BESSEL_IK_HPP
 447: 
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. This range declares or defines callable logic such as T, BOOST_MATH_INSTRUMENT_VARIABLE. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 此范围声明或定义了可调用逻辑，例如 T, BOOST_MATH_INSTRUMENT_VARIABLE。 条件分支会根据输入区间、错误情况或配置模式切换行为。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/series.hpp, boost/math/special_functions/sign.hpp, boost/math/special_functions/round.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/sin_pi.hpp, boost/math/constants/constants.hpp, boost/math/policies/error_handling.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `pow, log, exp, s, boost::math::tools::sum_series, BOOST_MATH_ASSERT, boost::math::tgamma1pm1, T, ...`
