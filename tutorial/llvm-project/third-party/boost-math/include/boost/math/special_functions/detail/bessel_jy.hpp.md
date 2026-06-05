# bessel_jy.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/bessel_jy.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the bessel jy special-function path.
- **作用（中文）**: 此头文件为 bessel jy 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  Copyright (c) 2006 Xiaogang Zhang
   2: //  Use, modification and distribution are subject to the
   3: //  Boost Software License, Version 1.0. (See accompanying file
   4: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   5: 
   6: #ifndef BOOST_MATH_BESSEL_JY_HPP
   7: #define BOOST_MATH_BESSEL_JY_HPP
   8: 
   9: #ifdef _MSC_VER
  10: #pragma once
  11: #endif
  12: 
  13: #include <boost/math/tools/config.hpp>
  14: #include <boost/math/tools/numeric_limits.hpp>
  15: #include <boost/math/tools/type_traits.hpp>
  16: #include <boost/math/special_functions/gamma.hpp>
  17: #include <boost/math/special_functions/sign.hpp>
  18: #include <boost/math/special_functions/hypot.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/type_traits.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/type_traits.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/special_functions/sin_pi.hpp>
  20: #include <boost/math/special_functions/cos_pi.hpp>
  21: #include <boost/math/special_functions/round.hpp>
  22: #include <boost/math/special_functions/detail/bessel_jy_asym.hpp>
  23: #include <boost/math/special_functions/detail/bessel_jy_series.hpp>
  24: #include <boost/math/constants/constants.hpp>
  25: #include <boost/math/policies/error_handling.hpp>
  26: 
  27: // Bessel functions of the first and second kind of fractional order
  28: 
  29: namespace boost { namespace math {
  30: 
  31:    namespace detail {
  32: 
  33:       //
  34:       // Simultaneous calculation of A&S 9.2.9 and 9.2.10
  35:       // for use in A&S 9.2.5 and 9.2.6.
  36:       // This series is quick to evaluate, but divergent unless
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/sin_pi.hpp, boost/math/special_functions/cos_pi.hpp, boost/math/special_functions/round.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/special_functions/sin_pi.hpp, boost/math/special_functions/cos_pi.hpp, boost/math/special_functions/round.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:       // x is very large, in fact it's pretty hard to figure out
  38:       // with any degree of precision when this series actually
  39:       // *will* converge!!  Consequently, we may just have to
  40:       // try it and see...
  41:       //
  42:       template <class T, class Policy>
  43:       BOOST_MATH_GPU_ENABLED bool hankel_PQ(T v, T x, T* p, T* q, const Policy& )
  44:       {
  45:          BOOST_MATH_STD_USING
  46:             T tolerance = 2 * policies::get_epsilon<T, Policy>();
  47:          *p = 1;
  48:          *q = 0;
  49:          T k = 1;
  50:          T z8 = 8 * x;
  51:          T sq = 1;
  52:          T mu = 4 * v * v;
  53:          T term = 1;
  54:          bool ok = true;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:          do
  56:          {
  57:             term *= (mu - sq * sq) / (k * z8);
  58:             *q += term;
  59:             k += 1;
  60:             sq += 2;
  61:             T mult = (sq * sq - mu) / (k * z8);
  62:             ok = fabs(mult) < 0.5f;
  63:             term *= mult;
  64:             *p += term;
  65:             k += 1;
  66:             sq += 2;
  67:          }
  68:          while((fabs(term) > tolerance * *p) && ok);
  69:          return ok;
  70:       }
  71: 
  72:       // Calculate Y(v, x) and Y(v+1, x) by Temme's method, see
~~~
- **EN:** Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Return statements hand the computed result or status back to the caller.
- **CN:** 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 return 语句会把计算结果或状态返回给调用方。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       // Temme, Journal of Computational Physics, vol 21, 343 (1976)
  74:       template <typename T, typename Policy>
  75:       BOOST_MATH_GPU_ENABLED int temme_jy(T v, T x, T* Y, T* Y1, const Policy& pol)
  76:       {
  77:          T g, h, p, q, f, coef, sum, sum1, tolerance;
  78:          T a, d, e, sigma;
  79:          unsigned long k;
  80: 
  81:          BOOST_MATH_STD_USING
  82:             using namespace boost::math::tools;
  83:          using namespace boost::math::constants;
  84: 
  85:          BOOST_MATH_ASSERT(fabs(v) <= 0.5f);  // precondition for using this routine
  86: 
  87:          T gp = boost::math::tgamma1pm1(v, pol);
  88:          T gm = boost::math::tgamma1pm1(-v, pol);
  89:          T spv = boost::math::sin_pi(v, pol);
  90:          T spv2 = boost::math::sin_pi(v/2, pol);
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as BOOST_MATH_ASSERT, boost::math::tgamma1pm1, ....
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, boost::math::tgamma1pm1, ...。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:          T xp = pow(x/2, v);
  92: 
  93:          a = log(x / 2);
  94:          sigma = -a * v;
  95:          d = abs(sigma) < tools::epsilon<T>() ?
  96:             T(1) : sinh(sigma) / sigma;
  97:          e = abs(v) < tools::epsilon<T>() ? T(v*pi<T>()*pi<T>() / 2)
  98:             : T(2 * spv2 * spv2 / v);
  99: 
 100:          T g1 = (v == 0) ? T(-euler<T>()) : T((gp - gm) / ((1 + gp) * (1 + gm) * 2 * v));
 101:          T g2 = (2 + gp + gm) / ((1 + gp) * (1 + gm) * 2);
 102:          T vspv = (fabs(v) < tools::epsilon<T>()) ? T(1/constants::pi<T>()) : T(v / spv);
 103:          f = (g1 * cosh(sigma) - g2 * a * d) * 2 * vspv;
 104: 
 105:          p = vspv / (xp * (1 + gm));
 106:          q = vspv * xp / (1 + gp);
 107: 
 108:          g = f + e * q;
~~~
- **EN:** This range declares or defines callable logic such as pow, log, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. The logic belongs to a numerical-integration path, so weights, nodes, or convergence rules are central.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow, log, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 该逻辑属于数值积分路径，因此权重、节点或收敛规则是核心内容。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:          h = p;
 110:          coef = 1;
 111:          sum = coef * g;
 112:          sum1 = coef * h;
 113: 
 114:          T v2 = v * v;
 115:          T coef_mult = -x * x / 4;
 116: 
 117:          // series summation
 118:          tolerance = policies::get_epsilon<T, Policy>();
 119:          for (k = 1; k < policies::get_max_series_iterations<Policy>(); k++)
 120:          {
 121:             f = (k * f + p + q) / (k*k - v2);
 122:             p /= k - v;
 123:             q /= k + v;
 124:             g = f + e * q;
 125:             h = p - k * g;
 126:             coef *= coef_mult / k;
~~~
- **EN:** Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:             sum += coef * g;
 128:             sum1 += coef * h;
 129:             if (abs(coef * g) < abs(sum) * tolerance)
 130:             {
 131:                break;
 132:             }
 133:          }
 134:          policies::check_series_iterations<T>("boost::math::bessel_jy<%1%>(%1%,%1%) in temme_jy", k, pol);
 135:          *Y = -sum;
 136:          *Y1 = -2 * sum1 / x;
 137: 
 138:          return 0;
 139:       }
 140: 
 141:       // Evaluate continued fraction fv = J_(v+1) / J_v, see
 142:       // Abramowitz and Stegun, Handbook of Mathematical Functions, 1972, 9.1.73
 143:       template <typename T, typename Policy>
 144:       BOOST_MATH_GPU_ENABLED int CF1_jy(T v, T x, T* fv, int* sign, const Policy& pol)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:       {
 146:          T C, D, f, a, b, delta, tiny, tolerance;
 147:          unsigned long k;
 148:          int s = 1;
 149: 
 150:          BOOST_MATH_STD_USING
 151: 
 152:             // |x| <= |v|, CF1_jy converges rapidly
 153:             // |x| > |v|, CF1_jy needs O(|x|) iterations to converge
 154: 
 155:             // modified Lentz's method, see
 156:             // Lentz, Applied Optics, vol 15, 668 (1976)
 157:             tolerance = 2 * policies::get_epsilon<T, Policy>();
 158:          tiny = sqrt(tools::min_value<T>());
 159:          C = f = tiny;                           // b0 = 0, replace with tiny
 160:          D = 0;
 161:          for (k = 1; k < policies::get_max_series_iterations<Policy>() * 100; k++)
 162:          {
~~~
- **EN:** This range declares or defines callable logic such as sqrt. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:             a = -1;
 164:             b = 2 * (v + k) / x;
 165:             C = b + a / C;
 166:             D = b + a * D;
 167:             if (C == 0) { C = tiny; }
 168:             if (D == 0) { D = tiny; }
 169:             D = 1 / D;
 170:             delta = C * D;
 171:             f *= delta;
 172:             if (D < 0) { s = -s; }
 173:             if (abs(delta - 1) < tolerance)
 174:             { break; }
 175:          }
 176:          policies::check_series_iterations<T>("boost::math::bessel_jy<%1%>(%1%,%1%) in CF1_jy", k / 100, pol);
 177:          *fv = -f;
 178:          *sign = s;                              // sign of denominator
 179: 
 180:          return 0;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:       }
 182:       //
 183:       // This algorithm was originally written by Xiaogang Zhang
 184:       // using std::complex to perform the complex arithmetic.
 185:       // However, that turns out to 10x or more slower than using
 186:       // all real-valued arithmetic, so it's been rewritten using
 187:       // real values only.
 188:       //
 189:       template <typename T, typename Policy>
 190:       BOOST_MATH_GPU_ENABLED int CF2_jy(T v, T x, T* p, T* q, const Policy& pol)
 191:       {
 192:          BOOST_MATH_STD_USING
 193: 
 194:             T Cr, Ci, Dr, Di, fr, fi, a, br, bi, delta_r, delta_i, temp;
 195:          T tiny;
 196:          unsigned long k;
 197: 
 198:          // |x| >= |v|, CF2_jy converges rapidly
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:          // |x| -> 0, CF2_jy fails to converge
 200:          BOOST_MATH_ASSERT(fabs(x) > 1);
 201: 
 202:          // modified Lentz's method, complex numbers involved, see
 203:          // Lentz, Applied Optics, vol 15, 668 (1976)
 204:          T tolerance = 2 * policies::get_epsilon<T, Policy>();
 205:          tiny = sqrt(tools::min_value<T>());
 206:          Cr = fr = -0.5f / x;
 207:          Ci = fi = 1;
 208:          //Dr = Di = 0;
 209:          T v2 = v * v;
 210:          a = (0.25f - v2) / x; // Note complex this one time only!
 211:          br = 2 * x;
 212:          bi = 2;
 213:          temp = Cr * Cr + 1;
 214:          Ci = bi + a * Cr / temp;
 215:          Cr = br + a / temp;
 216:          Dr = br;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, sqrt. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, sqrt。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:          Di = bi;
 218:          if (fabs(Cr) + fabs(Ci) < tiny) { Cr = tiny; }
 219:          if (fabs(Dr) + fabs(Di) < tiny) { Dr = tiny; }
 220:          temp = Dr * Dr + Di * Di;
 221:          Dr = Dr / temp;
 222:          Di = -Di / temp;
 223:          delta_r = Cr * Dr - Ci * Di;
 224:          delta_i = Ci * Dr + Cr * Di;
 225:          temp = fr;
 226:          fr = temp * delta_r - fi * delta_i;
 227:          fi = temp * delta_i + fi * delta_r;
 228:          for (k = 2; k < policies::get_max_series_iterations<Policy>(); k++)
 229:          {
 230:             a = static_cast<T>(k) - 0.5f;
 231:             a *= a;
 232:             a -= v2;
 233:             bi += 2;
 234:             temp = Cr * Cr + Ci * Ci;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:             Cr = br + a * Cr / temp;
 236:             Ci = bi - a * Ci / temp;
 237:             Dr = br + a * Dr;
 238:             Di = bi + a * Di;
 239:             if (fabs(Cr) + fabs(Ci) < tiny) { Cr = tiny; }
 240:             if (fabs(Dr) + fabs(Di) < tiny) { Dr = tiny; }
 241:             temp = Dr * Dr + Di * Di;
 242:             Dr = Dr / temp;
 243:             Di = -Di / temp;
 244:             delta_r = Cr * Dr - Ci * Di;
 245:             delta_i = Ci * Dr + Cr * Di;
 246:             temp = fr;
 247:             fr = temp * delta_r - fi * delta_i;
 248:             fi = temp * delta_i + fi * delta_r;
 249:             if (fabs(delta_r - 1) + fabs(delta_i) < tolerance)
 250:                break;
 251:          }
 252:          policies::check_series_iterations<T>("boost::math::bessel_jy<%1%>(%1%,%1%) in CF2_jy", k, pol);
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:          *p = fr;
 254:          *q = fi;
 255: 
 256:          return 0;
 257:       }
 258: 
 259:       BOOST_MATH_STATIC const int need_j = 1;
 260:       BOOST_MATH_STATIC const int need_y = 2;
 261: 
 262:       // Compute J(v, x) and Y(v, x) simultaneously by Steed's method, see
 263:       // Barnett et al, Computer Physics Communications, vol 8, 377 (1974)
 264:       template <typename T, typename Policy>
 265:       BOOST_MATH_GPU_ENABLED int bessel_jy(T v, T x, T* J, T* Y, int kind, const Policy& pol)
 266:       {
 267:          BOOST_MATH_ASSERT(x >= 0);
 268: 
 269:          T u, Jv, Ju, Yv, Yv1, Yu, Yu1(0), fv, fu;
 270:          T W, p, q, gamma, current, prev, next;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as BOOST_MATH_ASSERT. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:          bool reflect = false;
 272:          unsigned n, k;
 273:          int s;
 274:          int org_kind = kind;
 275:          T cp = 0;
 276:          T sp = 0;
 277: 
 278:          constexpr auto function = "boost::math::bessel_jy<%1%>(%1%,%1%)";
 279: 
 280:          BOOST_MATH_STD_USING
 281:             using namespace boost::math::tools;
 282:          using namespace boost::math::constants;
 283: 
 284:          if (v < 0)
 285:          {
 286:             reflect = true;
 287:             v = -v;                             // v is non-negative from here
 288:          }
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:          if (v > static_cast<T>((boost::math::numeric_limits<int>::max)()))
 290:          {
 291:             *J = *Y = policies::raise_evaluation_error<T>(function, "Order of Bessel function is too large to evaluate: got %1%", v, pol);
 292:             return 1;  // LCOV_EXCL_LINE previous line will throw.
 293:          }
 294:          n = static_cast<unsigned>(iround(v, pol));
 295:          u = v - n;                              // -1/2 <= u < 1/2
 296: 
 297:          if(reflect)
 298:          {
 299:             T z = (u + n % 2);
 300:             cp = boost::math::cos_pi(z, pol);
 301:             sp = boost::math::sin_pi(z, pol);
 302:             if(u != 0)
 303:                kind = need_j|need_y;               // need both for reflection formula
 304:          }
 305: 
 306:          if(x == 0)
~~~
- **EN:** This range declares or defines callable logic such as iround, boost::math::cos_pi, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 iround, boost::math::cos_pi, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:          {
 308:             if (v == 0)
 309:                *J = 1; // LCOV_EXCL_LINE multiprecision case only
 310:             else if ((u == 0) || !reflect)
 311:                *J = 0;
 312:             else if(kind & need_j)
 313:                *J = policies::raise_domain_error<T>(function, "Value of Bessel J_v(x) is complex-infinity at %1%", x, pol); // complex infinity
 314:             else
 315:                *J = boost::math::numeric_limits<T>::quiet_NaN();  // LCOV_EXCL_LINE, we should never get here, any value will do, not using J.
 316: 
 317:             if((kind & need_y) == 0)
 318:                *Y = boost::math::numeric_limits<T>::quiet_NaN();  // any value will do, not using Y.
 319:             else
 320:             {
 321:                // We shoud never get here:
 322:                BOOST_MATH_ASSERT(x != 0); // LCOV_EXCL_LINE
 323:             }
 324:             return 1;
~~~
- **EN:** This range declares or defines callable logic such as J_v, quiet_NaN, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 J_v, quiet_NaN, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:          }
 326: 
 327:          // x is positive until reflection
 328:          W = T(2) / (x * pi<T>());               // Wronskian
 329:          T Yv_scale = 1;
 330:          if(((kind & need_y) == 0) && ((x < 1) || (v > x * x / 4) || (x < 5)))
 331:          {
 332:             //
 333:             // This series will actually converge rapidly for all small
 334:             // x - say up to x < 20 - but the first few terms are large
 335:             // and divergent which leads to large errors :-(
 336:             //
 337:             Jv = bessel_j_small_z_series(v, x, pol);
 338:             Yv = boost::math::numeric_limits<T>::quiet_NaN();
 339:          }
 340:          else if((x < 1) && (u != 0) && (log(policies::get_epsilon<T, Policy>() / 2) > v * log((x/2) * (x/2) / v)))
 341:          {
 342:             // Evaluate using series representations.
~~~
- **EN:** This range declares or defines callable logic such as T, bessel_j_small_z_series, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T, bessel_j_small_z_series, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:             // This is particularly important for x << v as in this
 344:             // area temme_jy may be slow to converge, if it converges at all.
 345:             // Requires x is not an integer.
 346:             if(kind&need_j)
 347:                Jv = bessel_j_small_z_series(v, x, pol);
 348:             else
 349:                Jv = boost::math::numeric_limits<T>::quiet_NaN();
 350:             if((org_kind&need_y && (!reflect || (cp != 0)))
 351:                || (org_kind & need_j && (reflect && (sp != 0))))
 352:             {
 353:                // Only calculate if we need it, and if the reflection formula will actually use it:
 354:                Yv = bessel_y_small_z_series(v, x, &Yv_scale, pol);
 355:             }
 356:             else
 357:                Yv = boost::math::numeric_limits<T>::quiet_NaN();
 358:          }
 359:          else if((u == 0) && (x < policies::get_epsilon<T, Policy>()))
 360:          {
~~~
- **EN:** This range declares or defines callable logic such as bessel_j_small_z_series, quiet_NaN, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 bessel_j_small_z_series, quiet_NaN, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:             // Truncated series evaluation for small x and v an integer,
 362:             // much quicker in this area than temme_jy below.
 363:             // This code is only used in the multiprecision case, otherwise
 364:             // we go via bessel_jn.
 365:             // LCOV_EXCL_START
 366:             if(kind&need_j)
 367:                Jv = bessel_j_small_z_series(v, x, pol);
 368:             else
 369:                Jv = boost::math::numeric_limits<T>::quiet_NaN();
 370:             if((org_kind&need_y && (!reflect || (cp != 0)))
 371:                || (org_kind & need_j && (reflect && (sp != 0))))
 372:             {
 373:                // Only calculate if we need it, and if the reflection formula will actually use it:
 374:                Yv = bessel_yn_small_z(static_cast<int>(n), x, &Yv_scale, pol);
 375:             }
 376:             else
 377:                Yv = boost::math::numeric_limits<T>::quiet_NaN();
 378:             // LCOV_EXCL_STOP
~~~
- **EN:** This range declares or defines callable logic such as bessel_j_small_z_series, quiet_NaN, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 bessel_j_small_z_series, quiet_NaN, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:          }
 380:          else if(asymptotic_bessel_large_x_limit(v, x))
 381:          {
 382:             if(kind&need_y)
 383:             {
 384:                Yv = asymptotic_bessel_y_large_x_2(v, x, pol);
 385:             }
 386:             else
 387:                Yv = boost::math::numeric_limits<T>::quiet_NaN(); // any value will do, we're not using it.
 388:             if(kind&need_j)
 389:             {
 390:                Jv = asymptotic_bessel_j_large_x_2(v, x, pol);
 391:             }
 392:             else
 393:                Jv = boost::math::numeric_limits<T>::quiet_NaN(); // any value will do, we're not using it.
 394:          }
 395:          else if((x > 8) && hankel_PQ(v, x, &p, &q, pol))
 396:          {
~~~
- **EN:** This range declares or defines callable logic such as asymptotic_bessel_y_large_x_2, quiet_NaN, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 asymptotic_bessel_y_large_x_2, quiet_NaN, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:             //
 398:             // Hankel approximation: note that this method works best when x
 399:             // is large, but in that case we end up calculating sines and cosines
 400:             // of large values, with horrendous resulting accuracy.  It is fast though
 401:             // when it works....
 402:             //
 403:             // Normally we calculate sin/cos(chi) where:
 404:             //
 405:             // chi = x - fmod(T(v / 2 + 0.25f), T(2)) * boost::math::constants::pi<T>();
 406:             //
 407:             // But this introduces large errors, so use sin/cos addition formulae to
 408:             // improve accuracy:
 409:             //
 410:             T mod_v = fmod(T(v / 2 + 0.25f), T(2));
 411:             T sx = sin(x);
 412:             T cx = cos(x);
 413:             T sv = boost::math::sin_pi(mod_v, pol);
 414:             T cv = boost::math::cos_pi(mod_v, pol);
~~~
- **EN:** This range declares or defines callable logic such as fmod, sin, ....
- **CN:** 此范围声明或定义了可调用逻辑，例如 fmod, sin, ...。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415: 
 416:             T sc = sx * cv - sv * cx; // == sin(chi);
 417:             T cc = cx * cv + sx * sv; // == cos(chi);
 418:             T chi = boost::math::constants::root_two<T>() / (boost::math::constants::root_pi<T>() * sqrt(x)); //sqrt(2 / (boost::math::constants::pi<T>() * x));
 419:             Yv = chi * (p * sc + q * cc);
 420:             Jv = chi * (p * cc - q * sc);
 421:          }
 422:          else if (x <= 2)                           // x in (0, 2]
 423:          {
 424:             if(temme_jy(u, x, &Yu, &Yu1, pol))             // Temme series
 425:             {
 426:                // domain error, this should really have already been handled.
 427:                *J = *Y = Yu; // LCOV_EXCL_LINE
 428:                return 1;     // LCOV_EXCL_LINE
 429:             }
 430:             prev = Yu;
 431:             current = Yu1;
 432:             T scale = 1;
~~~
- **EN:** This range declares or defines callable logic such as sin, cos, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sin, cos, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:             policies::check_series_iterations<T>(function, n, pol);
 434:             for (k = 1; k <= n; k++)            // forward recurrence for Y
 435:             {
 436:                T fact = 2 * (u + k) / x;
 437:                if((tools::max_value<T>() - fabs(prev)) / fact < fabs(current))
 438:                {
 439:                   scale /= current;
 440:                   prev /= current;
 441:                   current = 1;
 442:                }
 443:                next = fact * current - prev;
 444:                prev = current;
 445:                current = next;
 446:             }
 447:             Yv = prev;
 448:             Yv1 = current;
 449:             if(kind&need_j)
 450:             {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:                CF1_jy(v, x, &fv, &s, pol);                 // continued fraction CF1_jy
 452:                Jv = scale * W / (Yv * fv - Yv1);           // Wronskian relation
 453:             }
 454:             else
 455:                Jv = boost::math::numeric_limits<T>::quiet_NaN(); // any value will do, we're not using it.
 456:             Yv_scale = scale;
 457:          }
 458:          else                                    // x in (2, \infty)
 459:          {
 460:             // Get Y(u, x):
 461: 
 462:             T ratio;
 463:             CF1_jy(v, x, &fv, &s, pol);
 464:             // tiny initial value to prevent overflow
 465:             T init = sqrt(tools::min_value<T>());
 466:             BOOST_MATH_INSTRUMENT_VARIABLE(init);
 467:             prev = fv * s * init;
 468:             current = s * init;
~~~
- **EN:** This range declares or defines callable logic such as CF1_jy, quiet_NaN, .... Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 CF1_jy, quiet_NaN, ...。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:             if(v < max_factorial<T>::value)
 470:             {
 471:                policies::check_series_iterations<T>(function, n, pol);
 472:                for (k = n; k > 0; k--)             // backward recurrence for J
 473:                {
 474:                   next = 2 * (u + k) * current / x - prev;
 475:                   //
 476:                   // We can't allow next to completely cancel out or the subsequent logic breaks.
 477:                   // Pretend that one bit did not cancel:
 478:                   if (next == 0)
 479:                   {
 480:                      next = prev * tools::epsilon<T>() / 2;  // LCOV_EXCL_LINE requires specific hardware and rounding to trigger, does get tested on msvc
 481:                   }
 482:                   prev = current;
 483:                   current = next;
 484:                }
 485:                ratio = (s * init) / current;     // scaling ratio
 486:                // can also call CF1_jy() to get fu, not much difference in precision
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:                fu = prev / current;
 488:             }
 489:             else
 490:             {
 491:                //
 492:                // When v is large we may get overflow in this calculation
 493:                // leading to NaN's and other nasty surprises:
 494:                //
 495:                policies::check_series_iterations<T>(function, n, pol);
 496:                bool over = false;
 497:                for (k = n; k > 0; k--)             // backward recurrence for J
 498:                {
 499:                   T t = 2 * (u + k) / x;
 500:                   if((t > 1) && (tools::max_value<T>() / t < current))
 501:                   {
 502:                      over = true;
 503:                      break;
 504:                   }
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:                   next = t * current - prev;
 506:                   prev = current;
 507:                   current = next;
 508:                }
 509:                if(!over)
 510:                {
 511:                   ratio = (s * init) / current;     // scaling ratio
 512:                   // can also call CF1_jy() to get fu, not much difference in precision
 513:                   fu = prev / current;
 514:                }
 515:                else
 516:                {
 517:                   ratio = 0;
 518:                   fu = 1;
 519:                }
 520:             }
 521:             CF2_jy(u, x, &p, &q, pol);                  // continued fraction CF2_jy
 522:             T t = u / x - fu;                   // t = J'/J
~~~
- **EN:** This range declares or defines callable logic such as CF2_jy. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 CF2_jy。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:             gamma = (p - t) / q;
 524:             //
 525:             // We can't allow gamma to cancel out to zero completely as it messes up
 526:             // the subsequent logic.  So pretend that one bit didn't cancel out
 527:             // and set to a suitably small value.  The only test case we've been able to
 528:             // find for this, is when v = 8.5 and x = 4*PI.
 529:             //
 530:             if(gamma == 0)
 531:             {
 532:                gamma = u * tools::epsilon<T>() / x;  // LCOV_EXCL_LINE requires specific hardware and rounding to trigger, does get tested on msvc
 533:             }
 534:             BOOST_MATH_INSTRUMENT_VARIABLE(current);
 535:             BOOST_MATH_INSTRUMENT_VARIABLE(W);
 536:             BOOST_MATH_INSTRUMENT_VARIABLE(q);
 537:             BOOST_MATH_INSTRUMENT_VARIABLE(gamma);
 538:             BOOST_MATH_INSTRUMENT_VARIABLE(p);
 539:             BOOST_MATH_INSTRUMENT_VARIABLE(t);
 540:             Ju = sign(current) * sqrt(W / (q + gamma * (p - t)));
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, sign. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, sign。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:             BOOST_MATH_INSTRUMENT_VARIABLE(Ju);
 542: 
 543:             Jv = Ju * ratio;                    // normalization
 544: 
 545:             Yu = gamma * Ju;
 546:             Yu1 = Yu * (u/x - p - q/gamma);
 547: 
 548:             if(kind&need_y)
 549:             {
 550:                // compute Y:
 551:                prev = Yu;
 552:                current = Yu1;
 553:                policies::check_series_iterations<T>(function, n, pol);
 554:                for (k = 1; k <= n; k++)            // forward recurrence for Y
 555:                {
 556:                   T fact = 2 * (u + k) / x;
 557:                   if((tools::max_value<T>() - fabs(prev)) / fact < fabs(current))
 558:                   {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 559-576 / 第 559-576 行
~~~cpp
 559:                      prev /= current;
 560:                      Yv_scale /= current;
 561:                      current = 1;
 562:                   }
 563:                   next = fact * current - prev;
 564:                   prev = current;
 565:                   current = next;
 566:                }
 567:                Yv = prev;
 568:             }
 569:             else
 570:                Yv = boost::math::numeric_limits<T>::quiet_NaN(); // any value will do, we're not using it.
 571:          }
 572: 
 573:          if (reflect)
 574:          {
 575:             if((sp != 0) && (tools::max_value<T>() * fabs(Yv_scale) < fabs(sp * Yv)))
 576:                *J = org_kind & need_j ? T(-sign(sp) * sign(Yv) * (Yv_scale != 0 ? sign(Yv_scale) : 1) * policies::raise_overflow_error<T>(function, nullptr, pol)) : T(0);
~~~
- **EN:** This range declares or defines callable logic such as quiet_NaN, T. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 quiet_NaN, T。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 577-594 / 第 577-594 行
~~~cpp
 577:             else
 578:                *J = cp * Jv - (sp == 0 ? T(0) : T((sp * Yv) / Yv_scale));     // reflection formula
 579:             if((cp != 0) && (tools::max_value<T>() * fabs(Yv_scale) < fabs(cp * Yv)))
 580:                *Y = org_kind & need_y ? T(-sign(cp) * sign(Yv) * (Yv_scale != 0 ? sign(Yv_scale) : 1) * policies::raise_overflow_error<T>(function, nullptr, pol)) : T(0);
 581:             else
 582:                *Y = (sp != 0 ? sp * Jv : T(0)) + (cp == 0 ? T(0) : T((cp * Yv) / Yv_scale));
 583:          }
 584:          else
 585:          {
 586:             *J = Jv;
 587:             if(tools::max_value<T>() * fabs(Yv_scale) < fabs(Yv))
 588:                *Y = org_kind & need_y ? T(sign(Yv) * sign(Yv_scale) * policies::raise_overflow_error<T>(function, nullptr, pol)) : T(0);
 589:             else
 590:                *Y = Yv / Yv_scale;
 591:          }
 592: 
 593:          return 0;
 594:       }
~~~
- **EN:** This range declares or defines callable logic such as T. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 595-600 / 第 595-600 行
~~~cpp
 595: 
 596:    } // namespace detail
 597: 
 598: }} // namespaces
 599: 
 600: #endif // BOOST_MATH_BESSEL_JY_HPP
~~~
- **EN:** This short range closes a conditional-compilation or header-guard region. The code enters namespace scope (detail) to keep symbols organized. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这一小段用于结束条件编译区域或头文件保护区域。 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/type_traits.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/sign.hpp, boost/math/special_functions/hypot.hpp, boost/math/special_functions/sin_pi.hpp, boost/math/special_functions/cos_pi.hpp, boost/math/special_functions/round.hpp, boost/math/special_functions/detail/bessel_jy_asym.hpp, boost/math/special_functions/detail/bessel_jy_series.hpp, boost/math/constants/constants.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `BOOST_MATH_ASSERT, boost::math::tgamma1pm1, boost::math::sin_pi, pow, log, T, fabs, sqrt, ...`
