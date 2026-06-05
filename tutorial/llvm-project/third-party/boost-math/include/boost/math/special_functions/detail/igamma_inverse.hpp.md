# igamma_inverse.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/igamma_inverse.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the igamma inverse special-function path.
- **作用（中文）**: 此头文件为 igamma inverse 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  (C) Copyright John Maddock 2006.
   2: //  (C) Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_SPECIAL_FUNCTIONS_IGAMMA_INVERSE_HPP
   8: #define BOOST_MATH_SPECIAL_FUNCTIONS_IGAMMA_INVERSE_HPP
   9: 
  10: #ifdef _MSC_VER
  11: #pragma once
  12: #endif
  13: 
  14: #include <boost/math/tools/config.hpp>
  15: #include <boost/math/tools/cstdint.hpp>
  16: #include <boost/math/tools/tuple.hpp>
  17: #include <boost/math/special_functions/gamma.hpp>
  18: #include <boost/math/special_functions/sign.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/tuple.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/tuple.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/tools/roots.hpp>
  20: #include <boost/math/policies/error_handling.hpp>
  21: 
  22: namespace boost{ namespace math{
  23: 
  24: namespace detail{
  25: 
  26: template <class T>
  27: BOOST_MATH_GPU_ENABLED T find_inverse_s(T p, T q)
  28: {
  29:    //
  30:    // Computation of the Incomplete Gamma Function Ratios and their Inverse
  31:    // ARMIDO R. DIDONATO and ALFRED H. MORRIS, JR.
  32:    // ACM Transactions on Mathematical Software, Vol. 12, No. 4,
  33:    // December 1986, Pages 377-393.
  34:    //
  35:    // See equation 32.
  36:    //
~~~
- **EN:** This block imports dependencies such as boost/math/tools/roots.hpp, boost/math/policies/error_handling.hpp so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/tools/roots.hpp, boost/math/policies/error_handling.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:    BOOST_MATH_STD_USING
  38:    T t;
  39:    if(p < T(0.5))
  40:    {
  41:       t = sqrt(-2 * log(p));
  42:    }
  43:    else
  44:    {
  45:       t = sqrt(-2 * log(q));
  46:    }
  47:    BOOST_MATH_STATIC const double a[4] = { 3.31125922108741, 11.6616720288968, 4.28342155967104, 0.213623493715853 };
  48:    BOOST_MATH_STATIC const double b[5] = { 1, 6.61053765625462, 6.40691597760039, 1.27364489782223, 0.3611708101884203e-1 };
  49:    T s = t - tools::evaluate_polynomial(a, t) / tools::evaluate_polynomial(b, t);
  50:    if(p < T(0.5))
  51:       s = -s;
  52:    return s;
  53: }
  54: 
~~~
- **EN:** This range declares or defines callable logic such as sqrt, tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: template <class T>
  56: BOOST_MATH_GPU_ENABLED T didonato_SN(T a, T x, unsigned N, T tolerance = 0)
  57: {
  58:    //
  59:    // Computation of the Incomplete Gamma Function Ratios and their Inverse
  60:    // ARMIDO R. DIDONATO and ALFRED H. MORRIS, JR.
  61:    // ACM Transactions on Mathematical Software, Vol. 12, No. 4,
  62:    // December 1986, Pages 377-393.
  63:    //
  64:    // See equation 34.
  65:    //
  66:    T sum = 1;
  67:    if(N >= 1)
  68:    {
  69:       T partial = x / (a + 1);
  70:       sum += partial;
  71:       for(unsigned i = 2; i <= N; ++i)
  72:       {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:          partial *= x / (a + i);
  74:          sum += partial;
  75:          if(partial < tolerance)
  76:             break;
  77:       }
  78:    }
  79:    return sum;
  80: }
  81: 
  82: template <class T, class Policy>
  83: BOOST_MATH_GPU_ENABLED inline T didonato_FN(T p, T a, T x, unsigned N, T tolerance, const Policy& pol)
  84: {
  85:    //
  86:    // Computation of the Incomplete Gamma Function Ratios and their Inverse
  87:    // ARMIDO R. DIDONATO and ALFRED H. MORRIS, JR.
  88:    // ACM Transactions on Mathematical Software, Vol. 12, No. 4,
  89:    // December 1986, Pages 377-393.
  90:    //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:    // See equation 34.
  92:    //
  93:    BOOST_MATH_STD_USING
  94:    T u = log(p) + boost::math::lgamma(a + 1, pol);
  95:    return exp((u + x - log(didonato_SN(a, x, N, tolerance))) / a);
  96: }
  97: 
  98: template <class T, class Policy>
  99: BOOST_MATH_GPU_ENABLED T find_inverse_gamma(T a, T p, T q, const Policy& pol, bool* p_has_10_digits)
 100: {
 101:    //
 102:    // In order to understand what's going on here, you will
 103:    // need to refer to:
 104:    //
 105:    // Computation of the Incomplete Gamma Function Ratios and their Inverse
 106:    // ARMIDO R. DIDONATO and ALFRED H. MORRIS, JR.
 107:    // ACM Transactions on Mathematical Software, Vol. 12, No. 4,
 108:    // December 1986, Pages 377-393.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as log.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:    //
 110:    BOOST_MATH_STD_USING
 111: 
 112:    T result;
 113:    *p_has_10_digits = false;
 114: 
 115:    if(a == 1)
 116:    {
 117:       result = -log(q);
 118:       BOOST_MATH_INSTRUMENT_VARIABLE(result);
 119:    }
 120:    else if(a < 1)
 121:    {
 122:       T g = boost::math::tgamma(a, pol);
 123:       T b = q * g;
 124:       BOOST_MATH_INSTRUMENT_VARIABLE(g);
 125:       BOOST_MATH_INSTRUMENT_VARIABLE(b);
 126:       if((b >T(0.6)) || ((b >= T(0.45)) && (a >= T(0.3))))
~~~
- **EN:** This range declares or defines callable logic such as log, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:       {
 128:          // DiDonato & Morris Eq 21:
 129:          //
 130:          // There is a slight variation from DiDonato and Morris here:
 131:          // the first form given here is unstable when p is close to 1,
 132:          // making it impossible to compute the inverse of Q(a,x) for small
 133:          // q.  Fortunately the second form works perfectly well in this case.
 134:          //
 135:          T u;
 136:          if((b * q > T(1e-8)) && (q > T(1e-5)))
 137:          {
 138:             u = pow(p * g * a, 1 / a);
 139:             BOOST_MATH_INSTRUMENT_VARIABLE(u);
 140:          }
 141:          else
 142:          {
 143:             u = exp((-q / a) - constants::euler<T>());
 144:             BOOST_MATH_INSTRUMENT_VARIABLE(u);
~~~
- **EN:** This range declares or defines callable logic such as pow, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:          }
 146:          result = u / (1 - (u / (a + 1)));
 147:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 148:       }
 149:       else if((a < 0.3) && (b >= 0.35))
 150:       {
 151:          // DiDonato & Morris Eq 22:
 152:          T t = exp(-constants::euler<T>() - b);
 153:          T u = t * exp(t);
 154:          result = t * exp(u);
 155:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 156:       }
 157:       else if((b > 0.15) || (a >= 0.3))
 158:       {
 159:          // DiDonato & Morris Eq 23:
 160:          T y = -log(b);
 161:          T u = y - (1 - a) * log(y);
 162:          result = y - (1 - a) * log(u) - log(1 + (1 - a) / (1 + u));
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, exp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, exp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 164:       }
 165:       else if (b > 0.1)
 166:       {
 167:          // DiDonato & Morris Eq 24:
 168:          T y = -log(b);
 169:          T u = y - (1 - a) * log(y);
 170:          result = y - (1 - a) * log(u) - log((u * u + 2 * (3 - a) * u + (2 - a) * (3 - a)) / (u * u + (5 - a) * u + 2));
 171:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 172:       }
 173:       else
 174:       {
 175:          // DiDonato & Morris Eq 25:
 176:          T y = -log(b);
 177:          T c1 = (a - 1) * log(y);
 178:          T c1_2 = c1 * c1;
 179:          T c1_3 = c1_2 * c1;
 180:          T c1_4 = c1_2 * c1_2;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, log. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, log。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:          T a_2 = a * a;
 182:          T a_3 = a_2 * a;
 183: 
 184:          T c2 = (a - 1) * (1 + c1);
 185:          T c3 = (a - 1) * (-(c1_2 / 2) + (a - 2) * c1 + (3 * a - 5) / 2);
 186:          T c4 = (a - 1) * ((c1_3 / 3) - (3 * a - 5) * c1_2 / 2 + (a_2 - 6 * a + 7) * c1 + (11 * a_2 - 46 * a + 47) / 6);
 187:          T c5 = (a - 1) * (-(c1_4 / 4)
 188:                            + (11 * a - 17) * c1_3 / 6
 189:                            + (-3 * a_2 + 13 * a -13) * c1_2
 190:                            + (2 * a_3 - 25 * a_2 + 72 * a - 61) * c1 / 2
 191:                            + (25 * a_3 - 195 * a_2 + 477 * a - 379) / 12);
 192: 
 193:          T y_2 = y * y;
 194:          T y_3 = y_2 * y;
 195:          T y_4 = y_2 * y_2;
 196:          result = y + c1 + (c2 / y) + (c3 / y_2) + (c4 / y_3) + (c5 / y_4);
 197:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 198:          if(b < 1e-28f)
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:             *p_has_10_digits = true;
 200:       }
 201:    }
 202:    else
 203:    {
 204:       // DiDonato and Morris Eq 31:
 205:       T s = find_inverse_s(p, q);
 206: 
 207:       BOOST_MATH_INSTRUMENT_VARIABLE(s);
 208: 
 209:       T s_2 = s * s;
 210:       T s_3 = s_2 * s;
 211:       T s_4 = s_2 * s_2;
 212:       T s_5 = s_4 * s;
 213:       T ra = sqrt(a);
 214: 
 215:       BOOST_MATH_INSTRUMENT_VARIABLE(ra);
 216: 
~~~
- **EN:** This range declares or defines callable logic such as find_inverse_s, BOOST_MATH_INSTRUMENT_VARIABLE, .... Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 find_inverse_s, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:       T w = a + s * ra + (s * s -1) / 3;
 218:       w += (s_3 - 7 * s) / (36 * ra);
 219:       w -= (3 * s_4 + 7 * s_2 - 16) / (810 * a);
 220:       w += (9 * s_5 + 256 * s_3 - 433 * s) / (38880 * a * ra);
 221: 
 222:       BOOST_MATH_INSTRUMENT_VARIABLE(w);
 223: 
 224:       if((a >= 500) && (fabs(1 - w / a) < 1e-6))
 225:       {
 226:          result = w;
 227:          *p_has_10_digits = true;
 228:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 229:       }
 230:       else if (p > 0.5)
 231:       {
 232:          if(w < 3 * a)
 233:          {
 234:             result = w;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 236:          }
 237:          else
 238:          {
 239:             T D = BOOST_MATH_GPU_SAFE_MAX(T(2), T(a * (a - 1)));
 240:             T lg = boost::math::lgamma(a, pol);
 241:             T lb = log(q) + lg;
 242:             if(lb < -D * T(2.3))
 243:             {
 244:                // DiDonato and Morris Eq 25:
 245:                T y = -lb;
 246:                T c1 = (a - 1) * log(y);
 247:                T c1_2 = c1 * c1;
 248:                T c1_3 = c1_2 * c1;
 249:                T c1_4 = c1_2 * c1_2;
 250:                T a_2 = a * a;
 251:                T a_3 = a_2 * a;
 252: 
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_GPU_SAFE_MAX, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_GPU_SAFE_MAX, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:                T c2 = (a - 1) * (1 + c1);
 254:                T c3 = (a - 1) * (-(c1_2 / 2) + (a - 2) * c1 + (3 * a - 5) / 2);
 255:                T c4 = (a - 1) * ((c1_3 / 3) - (3 * a - 5) * c1_2 / 2 + (a_2 - 6 * a + 7) * c1 + (11 * a_2 - 46 * a + 47) / 6);
 256:                T c5 = (a - 1) * (-(c1_4 / 4)
 257:                                  + (11 * a - 17) * c1_3 / 6
 258:                                  + (-3 * a_2 + 13 * a -13) * c1_2
 259:                                  + (2 * a_3 - 25 * a_2 + 72 * a - 61) * c1 / 2
 260:                                  + (25 * a_3 - 195 * a_2 + 477 * a - 379) / 12);
 261: 
 262:                T y_2 = y * y;
 263:                T y_3 = y_2 * y;
 264:                T y_4 = y_2 * y_2;
 265:                result = y + c1 + (c2 / y) + (c3 / y_2) + (c4 / y_3) + (c5 / y_4);
 266:                BOOST_MATH_INSTRUMENT_VARIABLE(result);
 267:             }
 268:             else
 269:             {
 270:                // DiDonato and Morris Eq 33:
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:                T u = -lb + (a - 1) * log(w) - log(1 + (1 - a) / (1 + w));
 272:                result = -lb + (a - 1) * log(u) - log(1 + (1 - a) / (1 + u));
 273:                BOOST_MATH_INSTRUMENT_VARIABLE(result);
 274:             }
 275:          }
 276:       }
 277:       else
 278:       {
 279:          T z = w;
 280:          T ap1 = a + 1;
 281:          T ap2 = a + 2;
 282:          if(w < 0.15f * ap1)
 283:          {
 284:             // DiDonato and Morris Eq 35:
 285:             T v = log(p) + boost::math::lgamma(ap1, pol);
 286:             z = exp((v + w) / a);
 287:             s = boost::math::log1p(z / ap1 * (1 + z / ap2), pol);
 288:             z = exp((v + z - s) / a);
~~~
- **EN:** This range declares or defines callable logic such as log, BOOST_MATH_INSTRUMENT_VARIABLE, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:             s = boost::math::log1p(z / ap1 * (1 + z / ap2), pol);
 290:             z = exp((v + z - s) / a);
 291:             s = boost::math::log1p(z / ap1 * (1 + z / ap2 * (1 + z / (a + 3))), pol);
 292:             z = exp((v + z - s) / a);
 293:             BOOST_MATH_INSTRUMENT_VARIABLE(z);
 294:          }
 295: 
 296:          if((z <= 0.01 * ap1) || (z > 0.7 * ap1))
 297:          {
 298:             result = z;
 299:             if(z <= T(0.002) * ap1)
 300:                *p_has_10_digits = true;
 301:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 302:          }
 303:          else
 304:          {
 305:             // DiDonato and Morris Eq 36:
 306:             T ls = log(didonato_SN(a, z, 100, T(1e-4)));
~~~
- **EN:** This range declares or defines callable logic such as boost::math::log1p, exp, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::log1p, exp, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:             T v = log(p) + boost::math::lgamma(ap1, pol);
 308:             z = exp((v + z - ls) / a);
 309:             result = z * (1 - (a * log(z) - z - v + ls) / (a - z));
 310: 
 311:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 312:          }
 313:       }
 314:    }
 315:    return result;
 316: }
 317: 
 318: template <class T, class Policy>
 319: struct gamma_p_inverse_func
 320: {
 321:    BOOST_MATH_GPU_ENABLED gamma_p_inverse_func(T a_, T p_, bool inv) : a(a_), p(p_), invert(inv)
 322:    {
 323:       //
 324:       // If p is too near 1 then P(x) - p suffers from cancellation
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as log, exp, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log, exp, ...。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:       // errors causing our root-finding algorithms to "thrash", better
 326:       // to invert in this case and calculate Q(x) - (1-p) instead.
 327:       //
 328:       // Of course if p is *very* close to 1, then the answer we get will
 329:       // be inaccurate anyway (because there's not enough information in p)
 330:       // but at least we will converge on the (inaccurate) answer quickly.
 331:       //
 332:       if(p > T(0.9))
 333:       {
 334:          p = 1 - p;
 335:          invert = !invert;
 336:       }
 337:    }
 338: 
 339:    BOOST_MATH_GPU_ENABLED boost::math::tuple<T, T, T> operator()(const T& x)const
 340:    {
 341:       BOOST_FPU_EXCEPTION_GUARD
 342:       //
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:       // Calculate P(x) - p and the first two derivates, or if the invert
 344:       // flag is set, then Q(x) - q and it's derivatives.
 345:       //
 346:       typedef typename policies::evaluation<T, Policy>::type value_type;
 347:       // typedef typename lanczos::lanczos<T, Policy>::type evaluation_type;
 348:       typedef typename policies::normalise<
 349:          Policy,
 350:          policies::promote_float<false>,
 351:          policies::promote_double<false>,
 352:          policies::discrete_quantile<>,
 353:          policies::assert_undefined<> >::type forwarding_policy;
 354: 
 355:       BOOST_MATH_STD_USING  // For ADL of std functions.
 356: 
 357:       T f, f1;
 358:       value_type ft;
 359:       f = static_cast<T>(boost::math::detail::gamma_incomplete_imp(
 360:                static_cast<value_type>(a),
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate. Type aliases simplify verbose template names and make later declarations easier to read. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:                static_cast<value_type>(x),
 362:                true, invert,
 363:                forwarding_policy(), &ft));
 364:       f1 = static_cast<T>(ft);
 365:       T f2;
 366:       T div = (a - x - 1) / x;
 367:       f2 = f1;
 368:       if(fabs(div) > 1)
 369:       {
 370:          // split if statement to address M1 mac clang bug;
 371:          // see issue 826
 372:          if (tools::max_value<T>() / fabs(div) < f2)
 373:          {
 374:             // overflow:
 375:             f2 = -tools::max_value<T>() / 2;
 376:          }
 377:          else
 378:          {
~~~
- **EN:** This range declares or defines callable logic such as forwarding_policy. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 forwarding_policy。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:             f2 *= div;
 380:          }
 381:       }
 382:       else
 383:       {
 384:          f2 *= div;
 385:       }
 386: 
 387:       if(invert)
 388:       {
 389:          f1 = -f1;
 390:          f2 = -f2;
 391:       }
 392: 
 393:       return boost::math::make_tuple(static_cast<T>(f - p), f1, f2);
 394:    }
 395: private:
 396:    T a, p;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:    bool invert;
 398: };
 399: 
 400: template <class T, class Policy>
 401: BOOST_MATH_GPU_ENABLED T gamma_p_inv_imp(T a, T p, const Policy& pol)
 402: {
 403:    BOOST_MATH_STD_USING  // ADL of std functions.
 404: 
 405:    constexpr auto function = "boost::math::gamma_p_inv<%1%>(%1%, %1%)";
 406: 
 407:    BOOST_MATH_INSTRUMENT_VARIABLE(a);
 408:    BOOST_MATH_INSTRUMENT_VARIABLE(p);
 409: 
 410:    if(a <= 0)
 411:       return policies::raise_domain_error<T>(function, "Argument a in the incomplete gamma function inverse must be >= 0 (got a=%1%).", a, pol);
 412:    if((p < 0) || (p > 1))
 413:       return policies::raise_domain_error<T>(function, "Probability must be in the range [0,1] in the incomplete gamma function inverse (got p=%1%).", p, pol);
 414:    if(p == 1)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:       return policies::raise_overflow_error<T>(function, nullptr, Policy());
 416:    if(p == 0)
 417:       return 0;
 418:    bool has_10_digits;
 419:    T guess = detail::find_inverse_gamma<T>(a, p, 1 - p, pol, &has_10_digits);
 420:    if((policies::digits<T, Policy>() <= 36) && has_10_digits)
 421:       return guess;
 422:    T lower = tools::min_value<T>();
 423:    if(guess <= lower)
 424:       guess = tools::min_value<T>();
 425:    BOOST_MATH_INSTRUMENT_VARIABLE(guess);
 426:    //
 427:    // Work out how many digits to converge to, normally this is
 428:    // 2/3 of the digits in T, but if the first derivative is very
 429:    // large convergence is slow, so we'll bump it up to full
 430:    // precision to prevent premature termination of the root-finding routine.
 431:    //
 432:    unsigned digits = policies::digits<T, Policy>();
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:    if(digits < 30)
 434:    {
 435:       digits *= 2;
 436:       digits /= 3;
 437:    }
 438:    else
 439:    {
 440:       digits /= 2;
 441:       digits -= 1;
 442:    }
 443:    if((a < T(0.125)) && (fabs(gamma_p_derivative(a, guess, pol)) > 1 / sqrt(tools::epsilon<T>())))
 444:       digits = policies::digits<T, Policy>() - 2;
 445:    //
 446:    // Go ahead and iterate:
 447:    //
 448:    boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 449: 
 450:    #ifndef BOOST_MATH_HAS_GPU_SUPPORT
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:    guess = tools::halley_iterate(
 452:       detail::gamma_p_inverse_func<T, Policy>(a, p, false),
 453:       guess,
 454:       lower,
 455:       tools::max_value<T>(),
 456:       digits,
 457:       max_iter);
 458:    #else
 459:    guess = tools::newton_raphson_iterate(
 460:       detail::gamma_p_inverse_func<T, Policy>(a, p, false),
 461:       guess,
 462:       lower,
 463:       tools::max_value<T>(),
 464:       digits,
 465:       max_iter);
 466:    #endif
 467: 
 468:    policies::check_root_iterations<T>(function, max_iter, pol);
~~~
- **EN:** This range continues the current implementation with a mix of declarations, structure, and low-level logic.
- **CN:** 此范围继续当前实现，混合了声明、结构与底层逻辑。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:    BOOST_MATH_INSTRUMENT_VARIABLE(guess);
 470:    if(guess == lower)
 471:       guess = policies::raise_underflow_error<T>(function, "Expected result known to be non-zero, but is smaller than the smallest available number.", pol);
 472:    return guess;
 473: }
 474: 
 475: template <class T, class Policy>
 476: BOOST_MATH_GPU_ENABLED T gamma_q_inv_imp(T a, T q, const Policy& pol)
 477: {
 478:    BOOST_MATH_STD_USING  // ADL of std functions.
 479: 
 480:    constexpr auto function = "boost::math::gamma_q_inv<%1%>(%1%, %1%)";
 481: 
 482:    if(a <= 0)
 483:       return policies::raise_domain_error<T>(function, "Argument a in the incomplete gamma function inverse must be >= 0 (got a=%1%).", a, pol);
 484:    if((q < 0) || (q > 1))
 485:       return policies::raise_domain_error<T>(function, "Probability must be in the range [0,1] in the incomplete gamma function inverse (got q=%1%).", q, pol);
 486:    if(q == 0)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:       return policies::raise_overflow_error<T>(function, nullptr, Policy());
 488:    if(q == 1)
 489:       return 0;
 490:    bool has_10_digits;
 491:    T guess = detail::find_inverse_gamma<T>(a, 1 - q, q, pol, &has_10_digits);
 492:    if((policies::digits<T, Policy>() <= 36) && has_10_digits)
 493:       return guess;
 494:    T lower = tools::min_value<T>();
 495:    if(guess <= lower)
 496:       guess = tools::min_value<T>();
 497:    //
 498:    // Work out how many digits to converge to, normally this is
 499:    // 2/3 of the digits in T, but if the first derivative is very
 500:    // large convergence is slow, so we'll bump it up to full
 501:    // precision to prevent premature termination of the root-finding routine.
 502:    //
 503:    unsigned digits = policies::digits<T, Policy>();
 504:    if(digits < 30)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:    {
 506:       digits *= 2;
 507:       digits /= 3;
 508:    }
 509:    else
 510:    {
 511:       digits /= 2;
 512:       digits -= 1;
 513:    }
 514:    if((a < 0.125) && (fabs(gamma_p_derivative(a, guess, pol)) > 1 / sqrt(tools::epsilon<T>())))
 515:       digits = policies::digits<T, Policy>();
 516:    //
 517:    // Go ahead and iterate:
 518:    //
 519:    boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 520: 
 521:    #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 522:    guess = tools::halley_iterate(
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:       detail::gamma_p_inverse_func<T, Policy>(a, q, true),
 524:       guess,
 525:       lower,
 526:       tools::max_value<T>(),
 527:       digits,
 528:       max_iter);
 529:    #else
 530:    guess = tools::newton_raphson_iterate(
 531:       detail::gamma_p_inverse_func<T, Policy>(a, q, true),
 532:       guess,
 533:       lower,
 534:       tools::max_value<T>(),
 535:       digits,
 536:       max_iter);
 537:    #endif
 538: 
 539:    policies::check_root_iterations<T>(function, max_iter, pol);
 540:    if(guess == lower)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:       guess = policies::raise_underflow_error<T>(function, "Expected result known to be non-zero, but is smaller than the smallest available number.", pol);
 542:    return guess;
 543: }
 544: 
 545: } // namespace detail
 546: 
 547: template <class T1, class T2, class Policy>
 548: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T1, T2>::type
 549:    gamma_p_inv(T1 a, T2 p, const Policy& pol)
 550: {
 551:    typedef typename tools::promote_args<T1, T2>::type result_type;
 552:    return detail::gamma_p_inv_imp(
 553:       static_cast<result_type>(a),
 554:       static_cast<result_type>(p), pol);
 555: }
 556: 
 557: template <class T1, class T2, class Policy>
 558: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T1, T2>::type
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T1` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T1`，作为该文件核心抽象的一部分。

### Lines 559-576 / 第 559-576 行
~~~cpp
 559:    gamma_q_inv(T1 a, T2 p, const Policy& pol)
 560: {
 561:    typedef typename tools::promote_args<T1, T2>::type result_type;
 562:    return detail::gamma_q_inv_imp(
 563:       static_cast<result_type>(a),
 564:       static_cast<result_type>(p), pol);
 565: }
 566: 
 567: template <class T1, class T2>
 568: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T1, T2>::type
 569:    gamma_p_inv(T1 a, T2 p)
 570: {
 571:    return gamma_p_inv(a, p, policies::policy<>());
 572: }
 573: 
 574: template <class T1, class T2>
 575: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T1, T2>::type
 576:    gamma_q_inv(T1 a, T2 p)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T1` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T1`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 577-587 / 第 577-587 行
~~~cpp
 577: {
 578:    return gamma_q_inv(a, p, policies::policy<>());
 579: }
 580: 
 581: } // namespace math
 582: } // namespace boost
 583: 
 584: #endif // BOOST_MATH_SPECIAL_FUNCTIONS_IGAMMA_INVERSE_HPP
 585: 
 586: 
 587: 
~~~
- **EN:** The code enters namespace scope (math::boost) to keep symbols organized. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/tuple.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/sign.hpp, boost/math/tools/roots.hpp, boost/math/policies/error_handling.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `sqrt, tools::evaluate_polynomial, log, BOOST_MATH_INSTRUMENT_VARIABLE, boost::math::tgamma, pow, exp, find_inverse_s, ...`
