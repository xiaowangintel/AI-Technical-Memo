# beta.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/beta.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the beta special function and related helpers.
- **作用（中文）**: 此头文件实现 beta 特殊函数及相关辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32 / 第 1-32 行
~~~cpp
   1: //  (C) Copyright John Maddock 2006.
   2: //  (C) Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_SPECIAL_BETA_HPP
   8: #define BOOST_MATH_SPECIAL_BETA_HPP
   9: 
  10: #ifdef _MSC_VER
  11: #pragma once
  12: #endif
  13: 
  14: #include <boost/math/tools/config.hpp>
  15: #include <boost/math/tools/type_traits.hpp>
  16: #include <boost/math/tools/assert.hpp>
  17: #include <boost/math/tools/precision.hpp>
  18: #include <boost/math/tools/numeric_limits.hpp>
  19: #include <boost/math/tools/cstdint.hpp>
  20: #include <boost/math/tools/tuple.hpp>
  21: #include <boost/math/tools/promotion.hpp>
  22: #include <boost/math/tools/cstdint.hpp>
  23: #include <boost/math/special_functions/gamma.hpp>
  24: #include <boost/math/special_functions/erf.hpp>
  25: #include <boost/math/special_functions/log1p.hpp>
  26: #include <boost/math/special_functions/expm1.hpp>
  27: #include <boost/math/special_functions/trunc.hpp>
  28: #include <boost/math/special_functions/lanczos.hpp>
  29: #include <boost/math/policies/policy.hpp>
  30: #include <boost/math/policies/error_handling.hpp>
  31: #include <boost/math/constants/constants.hpp>
  32: #include <boost/math/special_functions/math_fwd.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/assert.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/assert.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 33-64 / 第 33-64 行
~~~cpp
  33: #include <boost/math/special_functions/binomial.hpp>
  34: #include <boost/math/special_functions/factorials.hpp>
  35: #include <boost/math/tools/roots.hpp>
  36: 
  37: namespace boost{ namespace math{
  38: 
  39: namespace detail{
  40: 
  41: //
  42: // Implementation of Beta(a,b) using the Lanczos approximation:
  43: //
  44: template <class T, class Lanczos, class Policy>
  45: BOOST_MATH_GPU_ENABLED T beta_imp(T a, T b, const Lanczos&, const Policy& pol)
  46: {
  47:    BOOST_MATH_STD_USING  // for ADL of std names
  48: 
  49:    if(a <= 0)
  50:       return policies::raise_domain_error<T>("boost::math::beta<%1%>(%1%,%1%)", "The arguments to the beta function must be greater than zero (got a=%1%).", a, pol);
  51:    if(b <= 0)
  52:       return policies::raise_domain_error<T>("boost::math::beta<%1%>(%1%,%1%)", "The arguments to the beta function must be greater than zero (got b=%1%).", b, pol);
  53: 
  54:    T result;  // LCOV_EXCL_LINE
  55: 
  56:    T prefix = 1;
  57:    T c = a + b;
  58: 
  59:    // Special cases:
  60:    if((c == a) && (b < tools::epsilon<T>()))
  61:       return 1 / b;
  62:    else if((c == b) && (a < tools::epsilon<T>()))
  63:       return 1 / a;
  64:    if(b == 1)
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/binomial.hpp, boost/math/special_functions/factorials.hpp, boost/math/tools/roots.hpp so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/special_functions/binomial.hpp, boost/math/special_functions/factorials.hpp, boost/math/tools/roots.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 65-96 / 第 65-96 行
~~~cpp
  65:       return 1/a;
  66:    else if(a == 1)
  67:       return 1/b;
  68:    else if(c < tools::epsilon<T>())
  69:    {
  70:       result = c / a;
  71:       result /= b;
  72:       return result;
  73:    }
  74: 
  75:    /*
  76:    //
  77:    // This code appears to be no longer necessary: it was
  78:    // used to offset errors introduced from the Lanczos
  79:    // approximation, but the current Lanczos approximations
  80:    // are sufficiently accurate for all z that we can ditch
  81:    // this.  It remains in the file for future reference...
  82:    //
  83:    // If a or b are less than 1, shift to greater than 1:
  84:    if(a < 1)
  85:    {
  86:       prefix *= c / a;
  87:       c += 1;
  88:       a += 1;
  89:    }
  90:    if(b < 1)
  91:    {
  92:       prefix *= c / b;
  93:       c += 1;
  94:       b += 1;
  95:    }
  96:    */
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 97-128 / 第 97-128 行
~~~cpp
  97: 
  98:    if(a < b)
  99:    {
 100:       BOOST_MATH_GPU_SAFE_SWAP(a, b);
 101:    }
 102: 
 103:    // Lanczos calculation:
 104:    T agh = static_cast<T>(a + Lanczos::g() - 0.5f);
 105:    T bgh = static_cast<T>(b + Lanczos::g() - 0.5f);
 106:    T cgh = static_cast<T>(c + Lanczos::g() - 0.5f);
 107:    result = Lanczos::lanczos_sum_expG_scaled(a) * (Lanczos::lanczos_sum_expG_scaled(b) / Lanczos::lanczos_sum_expG_scaled(c));
 108:    T ambh = a - 0.5f - b;
 109:    if((fabs(b * ambh) < (cgh * 100)) && (a > 100))
 110:    {
 111:       // Special case where the base of the power term is close to 1
 112:       // compute (1+x)^y instead:
 113:       result *= exp(ambh * boost::math::log1p(-b / cgh, pol));
 114:    }
 115:    else
 116:    {
 117:       result *= pow(agh / cgh, a - T(0.5) - b);
 118:    }
 119:    if(cgh > 1e10f)
 120:       // this avoids possible overflow, but appears to be marginally less accurate:
 121:       result *= pow((agh / cgh) * (bgh / cgh), b);
 122:    else
 123:       result *= pow((agh * bgh) / (cgh * cgh), b);
 124:    result *= sqrt(boost::math::constants::e<T>() / bgh);
 125: 
 126:    // If a and b were originally less than 1 we need to scale the result:
 127:    result *= prefix;
 128: 
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_SWAP, Lanczos::g, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_SWAP, Lanczos::g, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 129-160 / 第 129-160 行
~~~cpp
 129:    return result;
 130: } // template <class T, class Lanczos> beta_imp(T a, T b, const Lanczos&)
 131: 
 132: //
 133: // Generic implementation of Beta(a,b) without Lanczos approximation support
 134: // (Caution this is slow!!!):
 135: //
 136: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 137: template <class T, class Policy>
 138: BOOST_MATH_GPU_ENABLED T beta_imp(T a, T b, const lanczos::undefined_lanczos& l, const Policy& pol)
 139: {
 140:    BOOST_MATH_STD_USING
 141: 
 142:    if(a <= 0)
 143:       return policies::raise_domain_error<T>("boost::math::beta<%1%>(%1%,%1%)", "The arguments to the beta function must be greater than zero (got a=%1%).", a, pol);
 144:    if(b <= 0)
 145:       return policies::raise_domain_error<T>("boost::math::beta<%1%>(%1%,%1%)", "The arguments to the beta function must be greater than zero (got b=%1%).", b, pol);
 146: 
 147:    const T c = a + b;
 148: 
 149:    // Special cases:
 150:    if ((c == a) && (b < tools::epsilon<T>()))
 151:       return 1 / b;
 152:    else if ((c == b) && (a < tools::epsilon<T>()))
 153:       return 1 / a;
 154:    if (b == 1)
 155:       return 1 / a;
 156:    else if (a == 1)
 157:       return 1 / b;
 158:    else if (c < tools::epsilon<T>())
 159:    {
 160:       T result = c / a;
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 161-192 / 第 161-192 行
~~~cpp
 161:       result /= b;
 162:       return result;
 163:    }
 164: 
 165:    // Regular cases start here:
 166:    const T min_sterling = minimum_argument_for_bernoulli_recursion<T>();
 167: 
 168:    long shift_a = 0;
 169:    long shift_b = 0;
 170: 
 171:    if(a < min_sterling)
 172:       shift_a = 1 + ltrunc(min_sterling - a);
 173:    if(b < min_sterling)
 174:       shift_b = 1 + ltrunc(min_sterling - b);
 175:    long shift_c = shift_a + shift_b;
 176: 
 177:    if ((shift_a == 0) && (shift_b == 0))
 178:    {
 179:       return pow(a / c, a) * pow(b / c, b) * scaled_tgamma_no_lanczos(a, pol) * scaled_tgamma_no_lanczos(b, pol) / scaled_tgamma_no_lanczos(c, pol);
 180:    }
 181:    else if ((a < 1) && (b < 1))
 182:    {
 183:       return boost::math::tgamma(a, pol) * (boost::math::tgamma(b, pol) / boost::math::tgamma(c));
 184:    }
 185:    else if(a < 1)
 186:       return boost::math::tgamma(a, pol) * boost::math::tgamma_delta_ratio(b, a, pol);
 187:    else if(b < 1)
 188:       return boost::math::tgamma(b, pol) * boost::math::tgamma_delta_ratio(a, b, pol);
 189:    else
 190:    {
 191:       T result = beta_imp(T(a + shift_a), T(b + shift_b), l, pol);
 192:       //
~~~
- **EN:** This range declares or defines callable logic such as ltrunc, beta_imp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ltrunc, beta_imp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 193-224 / 第 193-224 行
~~~cpp
 193:       // Recursion:
 194:       //
 195:       for (long i = 0; i < shift_c; ++i)
 196:       {
 197:          result *= c + i;
 198:          if (i < shift_a)
 199:             result /= a + i;
 200:          if (i < shift_b)
 201:             result /= b + i;
 202:       }
 203:       return result;
 204:    }
 205: 
 206: } // template <class T>T beta_imp(T a, T b, const lanczos::undefined_lanczos& l)
 207: #endif
 208: 
 209: //
 210: // Compute the leading power terms in the incomplete Beta:
 211: //
 212: // (x^a)(y^b)/Beta(a,b) when normalised, and
 213: // (x^a)(y^b) otherwise.
 214: //
 215: // Almost all of the error in the incomplete beta comes from this
 216: // function: particularly when a and b are large. Computing large
 217: // powers are *hard* though, and using logarithms just leads to
 218: // horrendous cancellation errors.
 219: //
 220: template <class T, class Lanczos, class Policy>
 221: BOOST_MATH_GPU_ENABLED T ibeta_power_terms(T a,
 222:                         T b,
 223:                         T x,
 224:                         T y,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 225-256 / 第 225-256 行
~~~cpp
 225:                         const Lanczos&,
 226:                         bool normalised,
 227:                         const Policy& pol,
 228:                         T prefix = 1,
 229:                         const char* function = "boost::math::ibeta<%1%>(%1%, %1%, %1%)")
 230: {
 231:    BOOST_MATH_STD_USING
 232: 
 233:    if(!normalised)
 234:    {
 235:       // can we do better here?
 236:       return pow(x, a) * pow(y, b);
 237:    }
 238: 
 239:    T result;  // LCOV_EXCL_LINE
 240: 
 241:    T c = a + b;
 242: 
 243:    // combine power terms with Lanczos approximation:
 244:    T gh = Lanczos::g() - 0.5f;
 245:    T agh = static_cast<T>(a + gh);
 246:    T bgh = static_cast<T>(b + gh);
 247:    T cgh = static_cast<T>(c + gh);
 248:    if ((a < tools::min_value<T>()) || (b < tools::min_value<T>()))
 249:       result = 0;  // denominator overflows in this case
 250:    else
 251:       result = Lanczos::lanczos_sum_expG_scaled(c) / (Lanczos::lanczos_sum_expG_scaled(a) * Lanczos::lanczos_sum_expG_scaled(b));
 252:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 253:    result *= prefix;
 254:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 255:    // combine with the leftover terms from the Lanczos approximation:
 256:    result *= sqrt(bgh / boost::math::constants::e<T>());
~~~
- **EN:** This range declares or defines callable logic such as Lanczos::lanczos_sum_expG_scaled, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Lanczos::lanczos_sum_expG_scaled, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 257-288 / 第 257-288 行
~~~cpp
 257:    result *= sqrt(agh / cgh);
 258:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 259: 
 260:    // l1 and l2 are the base of the exponents minus one:
 261:    T l1 = ((x * b - y * a) - y * gh) / agh;
 262:    T l2 = ((y * a - x * b) - x * gh) / bgh;
 263:    if((BOOST_MATH_GPU_SAFE_MIN(fabs(l1), fabs(l2)) < 0.2))
 264:    {
 265:       // when the base of the exponent is very near 1 we get really
 266:       // gross errors unless extra care is taken:
 267:       if((l1 * l2 > 0) || (BOOST_MATH_GPU_SAFE_MIN(a, b) < 1))
 268:       {
 269:          //
 270:          // This first branch handles the simple cases where either:
 271:          //
 272:          // * The two power terms both go in the same direction
 273:          // (towards zero or towards infinity).  In this case if either
 274:          // term overflows or underflows, then the product of the two must
 275:          // do so also.
 276:          // *Alternatively if one exponent is less than one, then we
 277:          // can't productively use it to eliminate overflow or underflow
 278:          // from the other term.  Problems with spurious overflow/underflow
 279:          // can't be ruled out in this case, but it is *very* unlikely
 280:          // since one of the power terms will evaluate to a number close to 1.
 281:          //
 282:          if(fabs(l1) < 0.1)
 283:          {
 284:             result *= exp(a * boost::math::log1p(l1, pol));
 285:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 286:          }
 287:          else
 288:          {
~~~
- **EN:** This range declares or defines callable logic such as sqrt, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 289-320 / 第 289-320 行
~~~cpp
 289:             result *= pow((x * cgh) / agh, a);
 290:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 291:          }
 292:          if(fabs(l2) < 0.1)
 293:          {
 294:             result *= exp(b * boost::math::log1p(l2, pol));
 295:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 296:          }
 297:          else
 298:          {
 299:             result *= pow((y * cgh) / bgh, b);
 300:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 301:          }
 302:       }
 303:       else if(BOOST_MATH_GPU_SAFE_MAX(fabs(l1), fabs(l2)) < 0.5)
 304:       {
 305:          //
 306:          // Both exponents are near one and both the exponents are
 307:          // greater than one and further these two
 308:          // power terms tend in opposite directions (one towards zero,
 309:          // the other towards infinity), so we have to combine the terms
 310:          // to avoid any risk of overflow or underflow.
 311:          //
 312:          // We do this by moving one power term inside the other, we have:
 313:          //
 314:          //    (1 + l1)^a * (1 + l2)^b
 315:          //  = ((1 + l1)*(1 + l2)^(b/a))^a
 316:          //  = (1 + l1 + l3 + l1*l3)^a   ;  l3 = (1 + l2)^(b/a) - 1
 317:          //                                    = exp((b/a) * log(1 + l2)) - 1
 318:          //
 319:          // The tricky bit is deciding which term to move inside :-)
 320:          // By preference we move the larger term inside, so that the
~~~
- **EN:** This range declares or defines callable logic such as pow, BOOST_MATH_INSTRUMENT_VARIABLE, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 321-352 / 第 321-352 行
~~~cpp
 321:          // size of the largest exponent is reduced.  However, that can
 322:          // only be done as long as l3 (see above) is also small.
 323:          //
 324:          bool small_a = a < b;
 325:          T ratio = b / a;
 326:          if((small_a && (ratio * l2 < 0.1)) || (!small_a && (l1 / ratio > 0.1)))
 327:          {
 328:             T l3 = boost::math::expm1(ratio * boost::math::log1p(l2, pol), pol);
 329:             l3 = l1 + l3 + l3 * l1;
 330:             l3 = a * boost::math::log1p(l3, pol);
 331:             result *= exp(l3);
 332:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 333:          }
 334:          else
 335:          {
 336:             T l3 = boost::math::expm1(boost::math::log1p(l1, pol) / ratio, pol);
 337:             l3 = l2 + l3 + l3 * l2;
 338:             l3 = b * boost::math::log1p(l3, pol);
 339:             result *= exp(l3);
 340:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 341:          }
 342:       }
 343:       else if(fabs(l1) < fabs(l2))
 344:       {
 345:          // First base near 1 only:
 346:          T l = a * boost::math::log1p(l1, pol)
 347:             + b * log((y * cgh) / bgh);
 348:          if((l <= tools::log_min_value<T>()) || (l >= tools::log_max_value<T>()))
 349:          {
 350:             l += log(result);
 351:             if(l >= tools::log_max_value<T>())
 352:                return policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE we can probably never get here, probably!
~~~
- **EN:** This range declares or defines callable logic such as boost::math::expm1, boost::math::log1p, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::expm1, boost::math::log1p, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 353-384 / 第 353-384 行
~~~cpp
 353:             result = exp(l);
 354:          }
 355:          else
 356:             result *= exp(l);
 357:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 358:       }
 359:       else
 360:       {
 361:          // Second base near 1 only:
 362:          T l = b * boost::math::log1p(l2, pol)
 363:             + a * log((x * cgh) / agh);
 364:          if((l <= tools::log_min_value<T>()) || (l >= tools::log_max_value<T>()))
 365:          {
 366:             l += log(result);
 367:             if(l >= tools::log_max_value<T>())
 368:                return policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE we can probably never get here, probably!
 369:             result = exp(l);
 370:          }
 371:          else
 372:             result *= exp(l);
 373:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 374:       }
 375:    }
 376:    else
 377:    {
 378:       // general case:
 379:       T b1 = (x * cgh) / agh;
 380:       T b2 = (y * cgh) / bgh;
 381:       l1 = a * log(b1);
 382:       l2 = b * log(b2);
 383:       BOOST_MATH_INSTRUMENT_VARIABLE(b1);
 384:       BOOST_MATH_INSTRUMENT_VARIABLE(b2);
~~~
- **EN:** This range declares or defines callable logic such as exp, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 385-416 / 第 385-416 行
~~~cpp
 385:       BOOST_MATH_INSTRUMENT_VARIABLE(l1);
 386:       BOOST_MATH_INSTRUMENT_VARIABLE(l2);
 387:       if((l1 >= tools::log_max_value<T>())
 388:          || (l1 <= tools::log_min_value<T>())
 389:          || (l2 >= tools::log_max_value<T>())
 390:          || (l2 <= tools::log_min_value<T>())
 391:          )
 392:       {
 393:          // Oops, under/overflow, sidestep if we can:
 394:          if(a < b)
 395:          {
 396:             T p1 = pow(b2, b / a);
 397:             T l3 = (b1 != 0) && (p1 != 0) ? (a * (log(b1) + log(p1))) : tools::max_value<T>(); // arbitrary large value if the logs would fail!
 398:             if((l3 < tools::log_max_value<T>())
 399:                && (l3 > tools::log_min_value<T>()))
 400:             {
 401:                result *= pow(p1 * b1, a);
 402:             }
 403:             else
 404:             {
 405:                l2 += l1 + log(result);
 406:                if(l2 >= tools::log_max_value<T>())
 407:                   return policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE we can probably never get here, probably!
 408:                result = exp(l2);
 409:             }
 410:          }
 411:          else
 412:          {
 413:             // This protects against spurious overflow in a/b:
 414:             T p1 = (b1 < 1) && (b < 1) && (tools::max_value<T>() * b < a) ? static_cast<T>(0) : static_cast<T>(pow(b1, a / b));
 415:             T l3 = (p1 != 0) && (b2 != 0) ? (log(p1) + log(b2)) * b : tools::max_value<T>();  // arbitrary large value if the logs would fail!
 416:             if((l3 < tools::log_max_value<T>())
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, pow, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, pow, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 417-448 / 第 417-448 行
~~~cpp
 417:                && (l3 > tools::log_min_value<T>()))
 418:             {
 419:                result *= pow(p1 * b2, b);
 420:             }
 421:             else if(result != 0)  // we can elude the calculation below if we're already going to be zero
 422:             {
 423:                l2 += l1 + log(result);
 424:                if(l2 >= tools::log_max_value<T>())
 425:                   return policies::raise_overflow_error<T>(function, nullptr, pol);  // LCOV_EXCL_LINE we can probably never get here, probably!
 426:                result = exp(l2);
 427:             }
 428:          }
 429:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 430:       }
 431:       else
 432:       {
 433:          // finally the normal case:
 434:          result *= pow(b1, a) * pow(b2, b);
 435:          BOOST_MATH_INSTRUMENT_VARIABLE(result);
 436:       }
 437:    }
 438: 
 439:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 440: 
 441:    if (0 == result)
 442:    {
 443:       if ((a > 1) && (x == 0))
 444:          return result;  // true zero LCOV_EXCL_LINE we can probably never get here
 445:       if ((b > 1) && (y == 0))
 446:          return result; // true zero LCOV_EXCL_LINE we can probably never get here
 447:       return boost::math::policies::raise_underflow_error<T>(function, nullptr, pol);
 448:    }
~~~
- **EN:** This range declares or defines callable logic such as pow, log, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow, log, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 449-480 / 第 449-480 行
~~~cpp
 449: 
 450:    return result;
 451: }
 452: //
 453: // Compute the leading power terms in the incomplete Beta:
 454: //
 455: // (x^a)(y^b)/Beta(a,b) when normalised, and
 456: // (x^a)(y^b) otherwise.
 457: //
 458: // Almost all of the error in the incomplete beta comes from this
 459: // function: particularly when a and b are large. Computing large
 460: // powers are *hard* though, and using logarithms just leads to
 461: // horrendous cancellation errors.
 462: //
 463: // This version is generic, slow, and does not use the Lanczos approximation.
 464: //
 465: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 466: template <class T, class Policy>
 467: BOOST_MATH_GPU_ENABLED T ibeta_power_terms(T a,
 468:                         T b,
 469:                         T x,
 470:                         T y,
 471:                         const boost::math::lanczos::undefined_lanczos& l,
 472:                         bool normalised,
 473:                         const Policy& pol,
 474:                         T prefix = 1,
 475:                         const char* = "boost::math::ibeta<%1%>(%1%, %1%, %1%)")
 476: {
 477:    BOOST_MATH_STD_USING
 478: 
 479:    if(!normalised)
 480:    {
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 481-512 / 第 481-512 行
~~~cpp
 481:       return prefix * pow(x, a) * pow(y, b);
 482:    }
 483: 
 484:    T c = a + b;
 485: 
 486:    const T min_sterling = minimum_argument_for_bernoulli_recursion<T>();
 487: 
 488:    long shift_a = 0;
 489:    long shift_b = 0;
 490: 
 491:    if (a < min_sterling)
 492:       shift_a = 1 + ltrunc(min_sterling - a);
 493:    if (b < min_sterling)
 494:       shift_b = 1 + ltrunc(min_sterling - b);
 495: 
 496:    if ((shift_a == 0) && (shift_b == 0))
 497:    {
 498:       T power1, power2;
 499:       bool need_logs = false;
 500:       if (a < b)
 501:       {
 502:          BOOST_MATH_IF_CONSTEXPR(boost::math::numeric_limits<T>::has_infinity)
 503:          {
 504:             power1 = pow((x * y * c * c) / (a * b), a);
 505:             power2 = pow((y * c) / b, b - a);
 506:          }
 507:          else
 508:          {
 509:             // We calculate these logs purely so we can check for overflow in the power functions
 510:             T l1 = log((x * y * c * c) / (a * b));
 511:             T l2 = log((y * c) / b);
 512:             if ((l1 * a > tools::log_min_value<T>()) && (l1 * a < tools::log_max_value<T>()) && (l2 * (b - a) < tools::log_max_value<T>()) && (l2 * (b - a) > tools::log_min_value<T>()))
~~~
- **EN:** This range declares or defines callable logic such as ltrunc, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ltrunc, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 513-544 / 第 513-544 行
~~~cpp
 513:             {
 514:                power1 = pow((x * y * c * c) / (a * b), a);
 515:                power2 = pow((y * c) / b, b - a);
 516:             }
 517:             else
 518:             {
 519:                need_logs = true;
 520:             }
 521:          }
 522:       }
 523:       else
 524:       {
 525:          BOOST_MATH_IF_CONSTEXPR(boost::math::numeric_limits<T>::has_infinity)
 526:          {
 527:             power1 = pow((x * y * c * c) / (a * b), b);
 528:             power2 = pow((x * c) / a, a - b);
 529:          }
 530:          else
 531:          {
 532:             // We calculate these logs purely so we can check for overflow in the power functions
 533:             T l1 = log((x * y * c * c) / (a * b)) * b;
 534:             T l2 = log((x * c) / a) * (a - b);
 535:             if ((l1 * a > tools::log_min_value<T>()) && (l1 * a < tools::log_max_value<T>()) && (l2 * (b - a) < tools::log_max_value<T>()) && (l2 * (b - a) > tools::log_min_value<T>()))
 536:             {
 537:                power1 = pow((x * y * c * c) / (a * b), b);
 538:                power2 = pow((x * c) / a, a - b);
 539:             }
 540:             else
 541:                need_logs = true;
 542:          }
 543:       }
 544:       BOOST_MATH_IF_CONSTEXPR(boost::math::numeric_limits<T>::has_infinity)
~~~
- **EN:** This range declares or defines callable logic such as pow, log. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow, log。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 545-576 / 第 545-576 行
~~~cpp
 545:       {
 546:          if (!(boost::math::isnormal)(power1) || !(boost::math::isnormal)(power2))
 547:          {
 548:             need_logs = true;
 549:          }
 550:       }
 551:       if (need_logs)
 552:       {
 553:          //
 554:          // We want:
 555:          //
 556:          // (xc / a)^a (yc / b)^b
 557:          //
 558:          // But we know that one or other term will over / underflow and combining the logs will be next to useless as that will cause significant cancellation.
 559:          // If we assume b > a and express z ^ b as(z ^ b / a) ^ a with z = (yc / b) then we can move one power term inside the other :
 560:          //
 561:          // ((xc / a) * (yc / b)^(b / a))^a
 562:          //
 563:          // However, we're not quite there yet, as the term being exponentiated is quite likely to be close to unity, so let:
 564:          //
 565:          // xc / a = 1 + (xb - ya) / a
 566:          //
 567:          // analogously let :
 568:          //
 569:          // 1 + p = (yc / b) ^ (b / a) = 1 + expm1((b / a) * log1p((ya - xb) / b))
 570:          //
 571:          // so putting the two together we have :
 572:          //
 573:          // exp(a * log1p((xb - ya) / a + p + p(xb - ya) / a))
 574:          //
 575:          // Analogously, when a > b we can just swap all the terms around.
 576:          //
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 577-608 / 第 577-608 行
~~~cpp
 577:          // Finally, there are a few cases (x or y is unity) when the above logic can't be used
 578:          // or where there is no logarithmic cancellation and accuracy is better just using
 579:          // the regular formula:
 580:          //
 581:          T xc_a = x * c / a;
 582:          T yc_b = y * c / b;
 583:          if ((x == 1) || (y == 1) || (fabs(xc_a - 1) > 0.25) || (fabs(yc_b - 1) > 0.25))
 584:          {
 585:             // The above logic fails, the result is almost certainly zero:
 586:             power1 = exp(log(xc_a) * a + log(yc_b) * b);
 587:             power2 = 1;
 588:          }
 589:          else if (b > a)
 590:          {
 591:             T p = boost::math::expm1((b / a) * boost::math::log1p((y * a - x * b) / b));
 592:             power1 = exp(a * boost::math::log1p((x * b - y * a) / a + p * (x * c / a)));
 593:             power2 = 1;
 594:          }
 595:          else
 596:          {
 597:             T p = boost::math::expm1((a / b) * boost::math::log1p((x * b - y * a) / a));
 598:             power1 = exp(b * boost::math::log1p((y * a - x * b) / b + p * (y * c / b)));
 599:             power2 = 1;
 600:          }
 601:       }
 602:       return prefix * power1 * power2 * scaled_tgamma_no_lanczos(c, pol) / (scaled_tgamma_no_lanczos(a, pol) * scaled_tgamma_no_lanczos(b, pol));
 603:    }
 604: 
 605:    T power1 = pow(x, a);
 606:    T power2 = pow(y, b);
 607:    T bet = beta_imp(a, b, l, pol);
 608: 
~~~
- **EN:** This range declares or defines callable logic such as exp, boost::math::expm1, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp, boost::math::expm1, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 609-640 / 第 609-640 行
~~~cpp
 609:    if(!(boost::math::isnormal)(power1) || !(boost::math::isnormal)(power2) || !(boost::math::isnormal)(bet))
 610:    {
 611:       int shift_c = shift_a + shift_b;
 612:       T result = ibeta_power_terms(T(a + shift_a), T(b + shift_b), x, y, l, normalised, pol, prefix);
 613:       if ((boost::math::isnormal)(result))
 614:       {
 615:          for (int i = 0; i < shift_c; ++i)
 616:          {
 617:             result /= c + i;
 618:                if (i < shift_a)
 619:                {
 620:                   result *= a + i;
 621:                      result /= x;
 622:                }
 623:             if (i < shift_b)
 624:             {
 625:                result *= b + i;
 626:                result /= y;
 627:             }
 628:          }
 629:          return prefix * result;
 630:       }
 631:       else
 632:       {
 633:          T log_result = log(x) * a + log(y) * b + log(prefix);
 634:          if ((boost::math::isnormal)(bet))
 635:             log_result -= log(bet);
 636:          else
 637:             log_result += boost::math::lgamma(c, pol) - boost::math::lgamma(a, pol) - boost::math::lgamma(b, pol);
 638:          return exp(log_result);
 639:       }
 640:    }
~~~
- **EN:** This range declares or defines callable logic such as ibeta_power_terms, log, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ibeta_power_terms, log, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 641-672 / 第 641-672 行
~~~cpp
 641:    return prefix * power1 * (power2 / bet);
 642: }
 643: 
 644: #endif
 645: //
 646: // Series approximation to the incomplete beta:
 647: //
 648: template <class T>
 649: struct ibeta_series_t
 650: {
 651:    typedef T result_type;
 652:    BOOST_MATH_GPU_ENABLED ibeta_series_t(T a_, T b_, T x_, T mult) : result(mult), x(x_), apn(a_), poch(1-b_), n(1) {}
 653:    BOOST_MATH_GPU_ENABLED T operator()()
 654:    {
 655:       T r = result / apn;
 656:       apn += 1;
 657:       result *= poch * x / n;
 658:       ++n;
 659:       poch += 1;
 660:       return r;
 661:    }
 662: private:
 663:    T result, x, apn, poch;
 664:    int n;
 665: };
 666: 
 667: template <class T, class Lanczos, class Policy>
 668: BOOST_MATH_GPU_ENABLED T ibeta_series(T a, T b, T x, T s0, const Lanczos&, bool normalised, T* p_derivative, T y, const Policy& pol)
 669: {
 670:    BOOST_MATH_STD_USING
 671: 
 672:    T result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as ibeta_series_t.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 ibeta_series_t。

### Lines 673-704 / 第 673-704 行
~~~cpp
 673: 
 674:    BOOST_MATH_ASSERT((p_derivative == 0) || normalised);
 675: 
 676:    if(normalised)
 677:    {
 678:       T c = a + b;
 679: 
 680:       // incomplete beta power term, combined with the Lanczos approximation:
 681:       T agh = static_cast<T>(a + Lanczos::g() - 0.5f);
 682:       T bgh = static_cast<T>(b + Lanczos::g() - 0.5f);
 683:       T cgh = static_cast<T>(c + Lanczos::g() - 0.5f);
 684:       if ((a < tools::min_value<T>()) || (b < tools::min_value<T>()))
 685:          result = 0;  // denorms cause overflow in the Lanzos series, result will be zero anyway
 686:       else
 687:       {
 688:          T l1 = Lanczos::lanczos_sum_expG_scaled(c);
 689:          T l2 = Lanczos::lanczos_sum_expG_scaled(a);
 690:          T l3 = Lanczos::lanczos_sum_expG_scaled(b);
 691:          if ((l2 > 1) && (l3 > 1) && (tools::max_value<T>() / l2 < l3))
 692:             result = (l1 / l2) / l3;
 693:          else
 694:          result = l1 / (l2 * l3);
 695:       }
 696: 
 697:       if (!(boost::math::isfinite)(result))
 698:          result = 0;  // LCOV_EXCL_LINE we can probably never get here, covered already above?
 699: 
 700:       T l1 = log(cgh / bgh) * (b - 0.5f);
 701:       T l2 = log(x * cgh / agh) * a;
 702:       //
 703:       // Check for over/underflow in the power terms:
 704:       //
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, Lanczos::g, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, Lanczos::g, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 705-736 / 第 705-736 行
~~~cpp
 705:       if((l1 > tools::log_min_value<T>())
 706:          && (l1 < tools::log_max_value<T>())
 707:          && (l2 > tools::log_min_value<T>())
 708:          && (l2 < tools::log_max_value<T>()))
 709:       {
 710:          if(a * b < bgh * 10)
 711:             result *= exp((b - 0.5f) * boost::math::log1p(a / bgh, pol));
 712:          else
 713:             result *= pow(cgh / bgh, T(b - T(0.5)));
 714:          result *= pow(x * cgh / agh, a);
 715:          result *= sqrt(agh / boost::math::constants::e<T>());
 716: 
 717:          if(p_derivative)
 718:          {
 719:             *p_derivative = result * pow(y, b);
 720:             BOOST_MATH_ASSERT(*p_derivative >= 0);
 721:          }
 722:       }
 723:       else
 724:       {
 725:          //
 726:          // Oh dear, we need logs, and this *will* cancel:
 727:          //
 728:          if (result != 0)  // elude calculation when result will be zero.
 729:          {
 730:             result = log(result) + l1 + l2 + (log(agh) - 1) / 2;
 731:             if (p_derivative)
 732:                *p_derivative = exp(result + b * log(y));
 733:             result = exp(result);
 734:          }
 735:       }
 736:    }
~~~
- **EN:** This range declares or defines callable logic such as exp, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 737-768 / 第 737-768 行
~~~cpp
 737:    else
 738:    {
 739:       // Non-normalised, just compute the power:
 740:       result = pow(x, a);
 741:    }
 742:    if(result < tools::min_value<T>())
 743:       return s0; // Safeguard: series can't cope with denorms.
 744:    ibeta_series_t<T> s(a, b, x, result);
 745:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 746:    result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter, s0);
 747:    policies::check_series_iterations<T>("boost::math::ibeta<%1%>(%1%, %1%, %1%) in ibeta_series (with lanczos)", max_iter, pol);
 748:    return result;
 749: }
 750: //
 751: // Incomplete Beta series again, this time without Lanczos support:
 752: //
 753: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 754: template <class T, class Policy>
 755: BOOST_MATH_GPU_ENABLED T ibeta_series(T a, T b, T x, T s0, const boost::math::lanczos::undefined_lanczos& l, bool normalised, T* p_derivative, T y, const Policy& pol)
 756: {
 757:    BOOST_MATH_STD_USING
 758: 
 759:    T result;
 760:    BOOST_MATH_ASSERT((p_derivative == 0) || normalised);
 761: 
 762:    if(normalised)
 763:    {
 764:       const T min_sterling = minimum_argument_for_bernoulli_recursion<T>();
 765: 
 766:       long shift_a = 0;
 767:       long shift_b = 0;
 768: 
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 769-800 / 第 769-800 行
~~~cpp
 769:       if (a < min_sterling)
 770:          shift_a = 1 + ltrunc(min_sterling - a);
 771:       if (b < min_sterling)
 772:          shift_b = 1 + ltrunc(min_sterling - b);
 773: 
 774:       T c = a + b;
 775: 
 776:       if ((shift_a == 0) && (shift_b == 0))
 777:       {
 778:          result = pow(x * c / a, a) * pow(c / b, b) * scaled_tgamma_no_lanczos(c, pol) / (scaled_tgamma_no_lanczos(a, pol) * scaled_tgamma_no_lanczos(b, pol));
 779:       }
 780:       else if ((a < 1) && (b > 1))
 781:          result = pow(x, a) / (boost::math::tgamma(a, pol) * boost::math::tgamma_delta_ratio(b, a, pol));
 782:       else
 783:       {
 784:          T power = pow(x, a);
 785:          T bet = beta_imp(a, b, l, pol);
 786:          if (!(boost::math::isnormal)(power) || !(boost::math::isnormal)(bet))
 787:          {
 788:             result = exp(a * log(x) + boost::math::lgamma(c, pol) - boost::math::lgamma(a, pol) - boost::math::lgamma(b, pol));
 789:          }
 790:          else
 791:             result = power / bet;
 792:       }
 793:       if(p_derivative)
 794:       {
 795:          *p_derivative = result * pow(y, b);
 796:          BOOST_MATH_ASSERT(*p_derivative >= 0);
 797:       }
 798:    }
 799:    else
 800:    {
~~~
- **EN:** This range declares or defines callable logic such as ltrunc, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ltrunc, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 801-832 / 第 801-832 行
~~~cpp
 801:       // Non-normalised, just compute the power:
 802:       result = pow(x, a);
 803:    }
 804:    if(result < tools::min_value<T>())
 805:       return s0; // Safeguard: series can't cope with denorms.
 806:    ibeta_series_t<T> s(a, b, x, result);
 807:    boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 808:    result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter, s0);
 809:    policies::check_series_iterations<T>("boost::math::ibeta<%1%>(%1%, %1%, %1%) in ibeta_series (without lanczos)", max_iter, pol);
 810:    return result;
 811: }
 812: #endif
 813: //
 814: // Continued fraction for the incomplete beta:
 815: //
 816: template <class T>
 817: struct ibeta_fraction2_t
 818: {
 819:    typedef boost::math::pair<T, T> result_type;
 820: 
 821:    BOOST_MATH_GPU_ENABLED ibeta_fraction2_t(T a_, T b_, T x_, T y_) : a(a_), b(b_), x(x_), y(y_), m(0) {}
 822: 
 823:    BOOST_MATH_GPU_ENABLED result_type operator()()
 824:    {
 825:       T denom = (a + 2 * m - 1);
 826:       T aN = (m * (a + m - 1) / denom) * ((a + b + m - 1) / denom) * (b - m) * x * x;
 827: 
 828:       T bN = static_cast<T>(m);
 829:       bN += (m * (b - m) * x) / (a + 2*m - 1);
 830:       bN += ((a + m) * (a * y - b * x + 1 + m *(2 - x))) / (a + 2*m + 1);
 831: 
 832:       ++m;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as pow, s, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 pow, s, ...。

### Lines 833-864 / 第 833-864 行
~~~cpp
 833: 
 834:       return boost::math::make_pair(aN, bN);
 835:    }
 836: 
 837: private:
 838:    T a, b, x, y;
 839:    int m;
 840: };
 841: //
 842: // Evaluate the incomplete beta via the continued fraction representation:
 843: //
 844: template <class T, class Policy>
 845: BOOST_MATH_GPU_ENABLED inline T ibeta_fraction2(T a, T b, T x, T y, const Policy& pol, bool normalised, T* p_derivative)
 846: {
 847:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
 848:    BOOST_MATH_STD_USING
 849:    T result = ibeta_power_terms(a, b, x, y, lanczos_type(), normalised, pol);
 850:    if(p_derivative)
 851:    {
 852:       *p_derivative = result;
 853:       BOOST_MATH_ASSERT(*p_derivative >= 0);
 854:    }
 855:    if(result == 0)
 856:       return result;
 857: 
 858:    ibeta_fraction2_t<T> f(a, b, x, y);
 859:    boost::math::uintmax_t max_terms = boost::math::policies::get_max_series_iterations<Policy>();
 860:    T fract = boost::math::tools::continued_fraction_b(f, boost::math::policies::get_epsilon<T, Policy>(), max_terms);
 861:    boost::math::policies::check_series_iterations<T>("boost::math::ibeta", max_terms, pol);
 862:    BOOST_MATH_INSTRUMENT_VARIABLE(fract);
 863:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 864:    return result / fract;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as ibeta_power_terms, BOOST_MATH_ASSERT, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 ibeta_power_terms, BOOST_MATH_ASSERT, ...。

### Lines 865-896 / 第 865-896 行
~~~cpp
 865: }
 866: //
 867: // Computes the difference between ibeta(a,b,x) and ibeta(a+k,b,x):
 868: //
 869: template <class T, class Policy>
 870: BOOST_MATH_GPU_ENABLED T ibeta_a_step(T a, T b, T x, T y, int k, const Policy& pol, bool normalised, T* p_derivative)
 871: {
 872:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
 873: 
 874:    BOOST_MATH_INSTRUMENT_VARIABLE(k);
 875: 
 876:    T prefix = ibeta_power_terms(a, b, x, y, lanczos_type(), normalised, pol);
 877:    if(p_derivative)
 878:    {
 879:       *p_derivative = prefix;
 880:       BOOST_MATH_ASSERT(*p_derivative >= 0);
 881:    }
 882:    prefix /= a;
 883:    if(prefix == 0)
 884:       return prefix;
 885:    T sum = 1;
 886:    T term = 1;
 887:    // series summation from 0 to k-1:
 888:    for(int i = 0; i < k-1; ++i)
 889:    {
 890:       term *= (a+b+i) * x / (a+i+1);
 891:       sum += term;
 892:    }
 893:    prefix *= sum;
 894: 
 895:    return prefix;
 896: }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, ibeta_power_terms, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, ibeta_power_terms, ...。

### Lines 897-928 / 第 897-928 行
~~~cpp
 897: 
 898: //
 899: // This function is only needed for the non-regular incomplete beta,
 900: // it computes the delta in:
 901: // beta(a,b,x) = prefix + delta * beta(a+k,b,x)
 902: // it is currently only called for small k.
 903: //
 904: template <class T>
 905: BOOST_MATH_GPU_ENABLED inline T rising_factorial_ratio(T a, T b, int k)
 906: {
 907:    // calculate:
 908:    // (a)(a+1)(a+2)...(a+k-1)
 909:    // _______________________
 910:    // (b)(b+1)(b+2)...(b+k-1)
 911: 
 912:    // This is only called with small k, for large k
 913:    // it is grossly inefficient, do not use outside it's
 914:    // intended purpose!!!
 915:    BOOST_MATH_INSTRUMENT_VARIABLE(k);
 916:    BOOST_MATH_ASSERT(k > 0);
 917: 
 918:    T result = 1;
 919:    for(int i = 0; i < k; ++i)
 920:       result *= (a+i) / (b+i);
 921:    return result;
 922: }
 923: //
 924: // Routine for a > 15, b < 1
 925: //
 926: // Begin by figuring out how large our table of Pn's should be,
 927: // quoted accuracies are "guesstimates" based on empirical observation.
 928: // Note that the table size should never exceed the size of our
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_ASSERT.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_ASSERT。

### Lines 929-960 / 第 929-960 行
~~~cpp
 929: // tables of factorials.
 930: //
 931: template <class T>
 932: struct Pn_size
 933: {
 934:    // This is likely to be enough for ~35-50 digit accuracy
 935:    // but it's hard to quantify exactly:
 936:    #ifndef BOOST_MATH_HAS_NVRTC
 937:    static constexpr unsigned value =
 938:       ::boost::math::max_factorial<T>::value >= 100 ? 50
 939:    : ::boost::math::max_factorial<T>::value >= ::boost::math::max_factorial<double>::value ? 30
 940:    : ::boost::math::max_factorial<T>::value >= ::boost::math::max_factorial<float>::value ? 15 : 1;
 941:    static_assert(::boost::math::max_factorial<T>::value >= ::boost::math::max_factorial<float>::value, "Type does not provide for 35-50 digits of accuracy.");
 942:    #else
 943:    static constexpr unsigned value = 0; // Will never be called
 944:    #endif
 945: };
 946: template <>
 947: struct Pn_size<float>
 948: {
 949:    static constexpr unsigned value = 15; // ~8-15 digit accuracy
 950: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 951:    static_assert(::boost::math::max_factorial<float>::value >= 30, "Type does not provide for 8-15 digits of accuracy.");
 952: #endif
 953: };
 954: template <>
 955: struct Pn_size<double>
 956: {
 957:    static constexpr unsigned value = 30; // 16-20 digit accuracy
 958: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 959:    static_assert(::boost::math::max_factorial<double>::value >= 60, "Type does not provide for 16-20 digits of accuracy.");
 960: #endif
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 961-992 / 第 961-992 行
~~~cpp
 961: };
 962: template <>
 963: struct Pn_size<long double>
 964: {
 965:    static constexpr unsigned value = 50; // ~35-50 digit accuracy
 966: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 967:    static_assert(::boost::math::max_factorial<long double>::value >= 100, "Type does not provide for ~35-50 digits of accuracy");
 968: #endif
 969: };
 970: 
 971: template <class T, class Policy>
 972: BOOST_MATH_GPU_ENABLED T beta_small_b_large_a_series(T a, T b, T x, T y, T s0, T mult, const Policy& pol, bool normalised)
 973: {
 974:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
 975:    BOOST_MATH_STD_USING
 976:    //
 977:    // This is DiDonato and Morris's BGRAT routine, see Eq's 9 through 9.6.
 978:    //
 979:    // Some values we'll need later, these are Eq 9.1:
 980:    //
 981:    T bm1 = b - 1;
 982:    T t = a + bm1 / 2;
 983:    T lx, u;  // LCOV_EXCL_LINE
 984:    if(y < 0.35)
 985:       lx = boost::math::log1p(-y, pol);
 986:    else
 987:       lx = log(x);
 988:    u = -t * lx;
 989:    // and from from 9.2:
 990:    T prefix;  // LCOV_EXCL_LINE
 991:    T h = regularised_gamma_prefix(b, u, pol, lanczos_type());
 992:    if(h <= tools::min_value<T>())
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `Pn_size` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `Pn_size`，作为该文件核心抽象的一部分。

### Lines 993-1024 / 第 993-1024 行
~~~cpp
 993:       return s0;
 994:    if(normalised)
 995:    {
 996:       prefix = h / boost::math::tgamma_delta_ratio(a, b, pol);
 997:       prefix /= pow(t, b);
 998:    }
 999:    else
1000:    {
1001:       prefix = full_igamma_prefix(b, u, pol) / pow(t, b);
1002:    }
1003:    prefix *= mult;
1004:    //
1005:    // now we need the quantity Pn, unfortunately this is computed
1006:    // recursively, and requires a full history of all the previous values
1007:    // so no choice but to declare a big table and hope it's big enough...
1008:    //
1009:    T p[ ::boost::math::detail::Pn_size<T>::value ] = { 1 };  // see 9.3.
1010:    //
1011:    // Now an initial value for J, see 9.6:
1012:    //
1013:    T j = boost::math::gamma_q(b, u, pol) / h;
1014:    //
1015:    // Now we can start to pull things together and evaluate the sum in Eq 9:
1016:    //
1017:    T sum = s0 + prefix * j;  // Value at N = 0
1018:    // some variables we'll need:
1019:    unsigned tnp1 = 1; // 2*N+1
1020:    T lx2 = lx / 2;
1021:    lx2 *= lx2;
1022:    T lxp = 1;
1023:    T t4 = 4 * t * t;
1024:    T b2n = b;
~~~
- **EN:** This range declares or defines callable logic such as boost::math::tgamma_delta_ratio, pow, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::tgamma_delta_ratio, pow, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1025-1056 / 第 1025-1056 行
~~~cpp
1025: 
1026:    for(unsigned n = 1; n < sizeof(p)/sizeof(p[0]); ++n)
1027:    {
1028:       /*
1029:       // debugging code, enable this if you want to determine whether
1030:       // the table of Pn's is large enough...
1031:       //
1032:       static int max_count = 2;
1033:       if(n > max_count)
1034:       {
1035:          max_count = n;
1036:          std::cerr << "Max iterations in BGRAT was " << n << std::endl;
1037:       }
1038:       */
1039:       //
1040:       // begin by evaluating the next Pn from Eq 9.4:
1041:       //
1042:       tnp1 += 2;
1043:       p[n] = 0;
1044:       T mbn = b - n;
1045:       unsigned tmp1 = 3;
1046:       for(unsigned m = 1; m < n; ++m)
1047:       {
1048:          mbn = m * b - n;
1049:          p[n] += mbn * p[n-m] / boost::math::unchecked_factorial<T>(tmp1);
1050:          tmp1 += 2;
1051:       }
1052:       p[n] /= n;
1053:       p[n] += bm1 / boost::math::unchecked_factorial<T>(tnp1);
1054:       //
1055:       // Now we want Jn from Jn-1 using Eq 9.6:
1056:       //
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 1057-1088 / 第 1057-1088 行
~~~cpp
1057:       j = (b2n * (b2n + 1) * j + (u + b2n + 1) * lxp) / t4;
1058:       lxp *= lx2;
1059:       b2n += 2;
1060:       //
1061:       // pull it together with Eq 9:
1062:       //
1063:       T r = prefix * p[n] * j;
1064:       sum += r;
1065:       // r is always small:
1066:       BOOST_MATH_ASSERT(tools::max_value<T>() * tools::epsilon<T>() > fabs(r));
1067:       if(fabs(r / tools::epsilon<T>()) < fabs(sum))
1068:          break;
1069:    }
1070:    return sum;
1071: } // template <class T, class Lanczos>T beta_small_b_large_a_series(T a, T b, T x, T y, T s0, T mult, const Lanczos& l, bool normalised)
1072: 
1073: //
1074: // For integer arguments we can relate the incomplete beta to the
1075: // complement of the binomial distribution cdf and use this finite sum.
1076: //
1077: template <class T, class Policy>
1078: BOOST_MATH_GPU_ENABLED T binomial_ccdf(T n, T k, T x, T y, const Policy& pol)
1079: {
1080:    BOOST_MATH_STD_USING // ADL of std names
1081: 
1082:    T result = pow(x, n);
1083: 
1084:    if(result > tools::min_value<T>())
1085:    {
1086:       T term = result;
1087:       for(unsigned i = itrunc(T(n - 1)); i > k; --i)
1088:       {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_ASSERT, pow.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, pow。

### Lines 1089-1120 / 第 1089-1120 行
~~~cpp
1089:          term *= ((i + 1) * y) / ((n - i) * x);
1090:          result += term;
1091:       }
1092:    }
1093:    else
1094:    {
1095:       // First term underflows so we need to start at the mode of the
1096:       // distribution and work outwards:
1097:       int start = itrunc(n * x);
1098:       if(start <= k + 1)
1099:          start = itrunc(k + 2);
1100:       result = static_cast<T>(pow(x, T(start)) * pow(y, n - T(start)) * boost::math::binomial_coefficient<T>(itrunc(n), itrunc(start), pol));
1101:       if(result == 0)
1102:       {
1103:          // OK, starting slightly above the mode didn't work,
1104:          // we'll have to sum the terms the old fashioned way.
1105:          // Very hard to get here, possibly only when exponent
1106:          // range is very limited (as with type float):
1107:          // LCOV_EXCL_START
1108:          for(unsigned i = start - 1; i > k; --i)
1109:          {
1110:             result += static_cast<T>(pow(x, static_cast<T>(i)) * pow(y, n - i) * boost::math::binomial_coefficient<T>(itrunc(n), itrunc(i), pol));
1111:          }
1112:          // LCOV_EXCL_STOP
1113:       }
1114:       else
1115:       {
1116:          T term = result;
1117:          T start_term = result;
1118:          for(unsigned i = start - 1; i > k; --i)
1119:          {
1120:             term *= ((i + 1) * y) / ((n - i) * x);
~~~
- **EN:** This range declares or defines callable logic such as itrunc, pow. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc, pow。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 1121-1152 / 第 1121-1152 行
~~~cpp
1121:             result += term;
1122:          }
1123:          term = start_term;
1124:          for(unsigned i = start + 1; i <= n; ++i)
1125:          {
1126:             term *= (n - i + 1) * x / (i * y);
1127:             result += term;
1128:          }
1129:       }
1130:    }
1131: 
1132:    return result;
1133: }
1134: 
1135: template <class T, class Policy>
1136: BOOST_MATH_GPU_ENABLED T ibeta_large_ab(T a, T b, T x, T y, bool invert, bool normalised, const Policy& pol)
1137: {
1138:    //
1139:    // Large arguments, symetric case, see https://dlmf.nist.gov/8.18
1140:    //
1141:    BOOST_MATH_STD_USING
1142: 
1143:    T x0 = a / (a + b);
1144:    T y0 = b / (a + b);
1145:    T nu = x0 * log(x / x0) + y0 * log(y / y0);
1146:    //
1147:    // Above compution is unstable, force nu to zero if
1148:    // something went wrong:
1149:    //
1150:    if ((nu > 0) || (x == x0) || (y == y0))
1151:       nu = 0;
1152:    nu = sqrt(-2 * nu);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as log, sqrt.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log, sqrt。

### Lines 1153-1184 / 第 1153-1184 行
~~~cpp
1153:    //
1154:    // As per https://dlmf.nist.gov/8.18#E10 we need to make sure we have the correct root:
1155:    //
1156:    if ((nu != 0) && (nu / (x - x0) < 0))
1157:       nu = -nu;
1158:    //
1159:    // The correction term in https://dlmf.nist.gov/8.18#E9 is badly unstable, and often
1160:    // makes the compution worse not better, we exclude it for now:
1161:    /*
1162:    T c0 = 0;
1163: 
1164:    if (nu != 0)
1165:    {
1166:       c0 = 1 / nu;
1167:       T lim = fabs(10 * tools::epsilon<T>() * c0);
1168:       c0 -= sqrt(x0 * y0) / (x - x0);
1169:       if(fabs(c0) < lim)
1170:          c0 = (1 - 2 * x0) / (3 * sqrt(x0 * y0));
1171:       else
1172:          c0 *= exp(a * log(x / x0) + b * log(y / y0));
1173:       c0 /= sqrt(constants::two_pi<T>() * (a + b));
1174:    }
1175:    else
1176:    {
1177:       c0 = (1 - 2 * x0) / (3 * sqrt(x0 * y0));
1178:       c0 /= sqrt(constants::two_pi<T>() * (a + b));
1179:    }
1180:    */
1181:    T mul = 1;
1182:    if (!normalised)
1183:       mul = boost::math::beta(a, b, pol);
1184: 
~~~
- **EN:** This range declares or defines callable logic such as fabs, sqrt, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs, sqrt, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1185-1216 / 第 1185-1216 行
~~~cpp
1185:    return mul * ((invert ? (1 + boost::math::erf(-nu * sqrt((a + b) / 2), pol)) / 2 : boost::math::erfc(-nu * sqrt((a + b) / 2), pol) / 2));
1186: }
1187: 
1188: 
1189: 
1190: //
1191: // The incomplete beta function implementation:
1192: // This is just a big bunch of spaghetti code to divide up the
1193: // input range and select the right implementation method for
1194: // each domain:
1195: //
1196: 
1197: template <class T, class Policy>
1198: BOOST_MATH_GPU_ENABLED T ibeta_imp(T a, T b, T x, const Policy& pol, bool inv, bool normalised, T* p_derivative)
1199: {
1200:    constexpr auto function = "boost::math::ibeta<%1%>(%1%, %1%, %1%)";
1201:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
1202:    BOOST_MATH_STD_USING // for ADL of std math functions.
1203: 
1204:    BOOST_MATH_INSTRUMENT_VARIABLE(a);
1205:    BOOST_MATH_INSTRUMENT_VARIABLE(b);
1206:    BOOST_MATH_INSTRUMENT_VARIABLE(x);
1207:    BOOST_MATH_INSTRUMENT_VARIABLE(inv);
1208:    BOOST_MATH_INSTRUMENT_VARIABLE(normalised);
1209: 
1210:    bool invert = inv;
1211:    T fract;
1212:    T y = 1 - x;
1213: 
1214:    BOOST_MATH_ASSERT((p_derivative == 0) || normalised);
1215: 
1216:    if(!(boost::math::isfinite)(a))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_ASSERT.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, BOOST_MATH_ASSERT。

### Lines 1217-1248 / 第 1217-1248 行
~~~cpp
1217:       return policies::raise_domain_error<T>(function, "The argument a to the incomplete beta function must be finite (got a=%1%).", a, pol);
1218:    if(!(boost::math::isfinite)(b))
1219:       return policies::raise_domain_error<T>(function, "The argument b to the incomplete beta function must be finite (got b=%1%).", b, pol);
1220:    if (!(0 <= x && x <= 1))
1221:       return policies::raise_domain_error<T>(function, "The argument x to the incomplete beta function must be in [0,1] (got x=%1%).", x, pol);
1222: 
1223:    if(p_derivative)
1224:       *p_derivative = -1; // value not set.
1225: 
1226:    if(normalised)
1227:    {
1228:       if(a < 0)
1229:          return policies::raise_domain_error<T>(function, "The argument a to the incomplete beta function must be >= zero (got a=%1%).", a, pol);
1230:       if(b < 0)
1231:          return policies::raise_domain_error<T>(function, "The argument b to the incomplete beta function must be >= zero (got b=%1%).", b, pol);
1232:       // extend to a few very special cases:
1233:       if(a == 0)
1234:       {
1235:          if(b == 0)
1236:             return policies::raise_domain_error<T>(function, "The arguments a and b to the incomplete beta function cannot both be zero, with x=%1%.", x, pol);
1237:          if(b > 0)
1238:             return static_cast<T>(inv ? 0 : 1);
1239:       }
1240:       else if(b == 0)
1241:       {
1242:          if(a > 0)
1243:             return static_cast<T>(inv ? 1 : 0);
1244:       }
1245:    }
1246:    else
1247:    {
1248:       if(a <= 0)
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 1249-1280 / 第 1249-1280 行
~~~cpp
1249:          return policies::raise_domain_error<T>(function, "The argument a to the incomplete beta function must be greater than zero (got a=%1%).", a, pol);
1250:       if(b <= 0)
1251:          return policies::raise_domain_error<T>(function, "The argument b to the incomplete beta function must be greater than zero (got b=%1%).", b, pol);
1252:    }
1253: 
1254:    if(x == 0)
1255:    {
1256:       if(p_derivative)
1257:       {
1258:          *p_derivative = (a == 1) ? (T)1 : (a < 1) ? T(tools::max_value<T>() / 2) : T(tools::min_value<T>() * 2);
1259:       }
1260:       return (invert ? (normalised ? T(1) : boost::math::beta(a, b, pol)) : T(0));
1261:    }
1262:    if(x == 1)
1263:    {
1264:       if(p_derivative)
1265:       {
1266:          *p_derivative = (b == 1) ? T(1) : (b < 1) ? T(tools::max_value<T>() / 2) : T(tools::min_value<T>() * 2);
1267:       }
1268:       return (invert == 0 ? (normalised ? 1 : boost::math::beta(a, b, pol)) : 0);
1269:    }
1270:    if((a == 0.5f) && (b == 0.5f))
1271:    {
1272:       // We have an arcsine distribution:
1273:       if(p_derivative)
1274:       {
1275:          *p_derivative = 1 / (constants::pi<T>() * sqrt(y * x));
1276:       }
1277:       T p = invert ? asin(sqrt(y)) / constants::half_pi<T>() : asin(sqrt(x)) / constants::half_pi<T>();
1278:       if(!normalised)
1279:          p *= constants::pi<T>();
1280:       return p;
~~~
- **EN:** This range declares or defines callable logic such as T, sqrt, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T, sqrt, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1281-1312 / 第 1281-1312 行
~~~cpp
1281:    }
1282:    if(a == 1)
1283:    {
1284:       BOOST_MATH_GPU_SAFE_SWAP(a, b);
1285:       BOOST_MATH_GPU_SAFE_SWAP(x, y);
1286:       invert = !invert;
1287:    }
1288:    if(b == 1)
1289:    {
1290:       //
1291:       // Special case see: http://functions.wolfram.com/GammaBetaErf/BetaRegularized/03/01/01/
1292:       //
1293:       if(a == 1)
1294:       {
1295:          if(p_derivative)
1296:             *p_derivative = 1;
1297:          return invert ? y : x;
1298:       }
1299: 
1300:       if(p_derivative)
1301:       {
1302:          *p_derivative = a * pow(x, a - 1);
1303:       }
1304:       T p;  // LCOV_EXCL_LINE
1305:       if(y < 0.5)
1306:          p = invert ? T(-boost::math::expm1(a * boost::math::log1p(-y, pol), pol)) : T(exp(a * boost::math::log1p(-y, pol)));
1307:       else
1308:          p = invert ? T(-boost::math::powm1(x, a, pol)) : T(pow(x, a));
1309:       if(!normalised)
1310:          p /= a;
1311:       return p;
1312:    }
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_SWAP, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_SWAP, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 1313-1344 / 第 1313-1344 行
~~~cpp
1313: 
1314:    if(BOOST_MATH_GPU_SAFE_MIN(a, b) <= 1)
1315:    {
1316:       if(x > 0.5)
1317:       {
1318:          BOOST_MATH_GPU_SAFE_SWAP(a, b);
1319:          BOOST_MATH_GPU_SAFE_SWAP(x, y);
1320:          invert = !invert;
1321:          BOOST_MATH_INSTRUMENT_VARIABLE(invert);
1322:       }
1323:       if(BOOST_MATH_GPU_SAFE_MAX(a, b) <= 1)
1324:       {
1325:          // Both a,b < 1:
1326:          if((a >= BOOST_MATH_GPU_SAFE_MIN(T(0.2), b)) || (pow(x, a) <= 0.9))
1327:          {
1328:             if(!invert)
1329:             {
1330:                fract = ibeta_series(a, b, x, T(0), lanczos_type(), normalised, p_derivative, y, pol);
1331:                BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1332:             }
1333:             else
1334:             {
1335:                fract = -(normalised ? 1 : boost::math::beta(a, b, pol));
1336:                invert = false;
1337:                fract = -ibeta_series(a, b, x, fract, lanczos_type(), normalised, p_derivative, y, pol);
1338:                BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1339:             }
1340:          }
1341:          else
1342:          {
1343:             BOOST_MATH_GPU_SAFE_SWAP(a, b);
1344:             BOOST_MATH_GPU_SAFE_SWAP(x, y);
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_SWAP, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_SWAP, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1345-1376 / 第 1345-1376 行
~~~cpp
1345:             invert = !invert;
1346:             if(y >= 0.3)
1347:             {
1348:                if(!invert)
1349:                {
1350:                   fract = ibeta_series(a, b, x, T(0), lanczos_type(), normalised, p_derivative, y, pol);
1351:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1352:                }
1353:                else
1354:                {
1355:                   fract = -(normalised ? 1 : boost::math::beta(a, b, pol));
1356:                   invert = false;
1357:                   fract = -ibeta_series(a, b, x, fract, lanczos_type(), normalised, p_derivative, y, pol);
1358:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1359:                }
1360:             }
1361:             else
1362:             {
1363:                // Sidestep on a, and then use the series representation:
1364:                T prefix;  // LCOV_EXCL_LINE
1365:                if(!normalised)
1366:                {
1367:                   prefix = rising_factorial_ratio(T(a+b), a, 20);
1368:                }
1369:                else
1370:                {
1371:                   prefix = 1;
1372:                }
1373:                fract = ibeta_a_step(a, b, x, y, 20, pol, normalised, p_derivative);
1374:                if(!invert)
1375:                {
1376:                   fract = beta_small_b_large_a_series(T(a + 20), b, x, y, fract, prefix, pol, normalised);
~~~
- **EN:** This range declares or defines callable logic such as ibeta_series, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ibeta_series, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1377-1408 / 第 1377-1408 行
~~~cpp
1377:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1378:                }
1379:                else
1380:                {
1381:                   fract -= (normalised ? 1 : boost::math::beta(a, b, pol));
1382:                   invert = false;
1383:                   fract = -beta_small_b_large_a_series(T(a + 20), b, x, y, fract, prefix, pol, normalised);
1384:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1385:                }
1386:             }
1387:          }
1388:       }
1389:       else
1390:       {
1391:          // One of a, b < 1 only:
1392:          if((b <= 1) || ((x < 0.1) && (pow(b * x, a) <= 0.7)))
1393:          {
1394:             if(!invert)
1395:             {
1396:                fract = ibeta_series(a, b, x, T(0), lanczos_type(), normalised, p_derivative, y, pol);
1397:                BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1398:             }
1399:             else
1400:             {
1401:                fract = -(normalised ? 1 : boost::math::beta(a, b, pol));
1402:                invert = false;
1403:                fract = -ibeta_series(a, b, x, fract, lanczos_type(), normalised, p_derivative, y, pol);
1404:                BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1405:             }
1406:          }
1407:          else
1408:          {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, boost::math::beta, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, boost::math::beta, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1409-1440 / 第 1409-1440 行
~~~cpp
1409:             BOOST_MATH_GPU_SAFE_SWAP(a, b);
1410:             BOOST_MATH_GPU_SAFE_SWAP(x, y);
1411:             invert = !invert;
1412: 
1413:             if(y >= 0.3)
1414:             {
1415:                if(!invert)
1416:                {
1417:                   fract = ibeta_series(a, b, x, T(0), lanczos_type(), normalised, p_derivative, y, pol);
1418:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1419:                }
1420:                else
1421:                {
1422:                   fract = -(normalised ? 1 : boost::math::beta(a, b, pol));
1423:                   invert = false;
1424:                   fract = -ibeta_series(a, b, x, fract, lanczos_type(), normalised, p_derivative, y, pol);
1425:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1426:                }
1427:             }
1428:             else if(a >= 15)
1429:             {
1430:                if(!invert)
1431:                {
1432:                   fract = beta_small_b_large_a_series(a, b, x, y, T(0), T(1), pol, normalised);
1433:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1434:                }
1435:                else
1436:                {
1437:                   fract = -(normalised ? 1 : boost::math::beta(a, b, pol));
1438:                   invert = false;
1439:                   fract = -beta_small_b_large_a_series(a, b, x, y, fract, T(1), pol, normalised);
1440:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_SWAP, ibeta_series, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_SWAP, ibeta_series, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1441-1472 / 第 1441-1472 行
~~~cpp
1441:                }
1442:             }
1443:             else
1444:             {
1445:                // Sidestep to improve errors:
1446:                T prefix;  // LCOV_EXCL_LINE
1447:                if(!normalised)
1448:                {
1449:                   prefix = rising_factorial_ratio(T(a+b), a, 20);
1450:                }
1451:                else
1452:                {
1453:                   prefix = 1;
1454:                }
1455:                fract = ibeta_a_step(a, b, x, y, 20, pol, normalised, p_derivative);
1456:                BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1457:                if(!invert)
1458:                {
1459:                   fract = beta_small_b_large_a_series(T(a + 20), b, x, y, fract, prefix, pol, normalised);
1460:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1461:                }
1462:                else
1463:                {
1464:                   fract -= (normalised ? 1 : boost::math::beta(a, b, pol));
1465:                   invert = false;
1466:                   fract = -beta_small_b_large_a_series(T(a + 20), b, x, y, fract, prefix, pol, normalised);
1467:                   BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1468:                }
1469:             }
1470:          }
1471:       }
1472:    }
~~~
- **EN:** This range declares or defines callable logic such as rising_factorial_ratio, ibeta_a_step, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 rising_factorial_ratio, ibeta_a_step, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1473-1504 / 第 1473-1504 行
~~~cpp
1473:    else
1474:    {
1475:       // Both a,b >= 1:
1476:       T lambda;  // LCOV_EXCL_LINE
1477:       if(a < b)
1478:       {
1479:          lambda = a - (a + b) * x;
1480:       }
1481:       else
1482:       {
1483:          lambda = (a + b) * y - b;
1484:       }
1485:       if(lambda < 0)
1486:       {
1487:          BOOST_MATH_GPU_SAFE_SWAP(a, b);
1488:          BOOST_MATH_GPU_SAFE_SWAP(x, y);
1489:          invert = !invert;
1490:          BOOST_MATH_INSTRUMENT_VARIABLE(invert);
1491:       }
1492: 
1493:       if(b < 40)
1494:       {
1495:          if((floor(a) == a) && (floor(b) == b) && (a < static_cast<T>((boost::math::numeric_limits<int>::max)() - 100)) && (y != 1))
1496:          {
1497:             // relate to the binomial distribution and use a finite sum:
1498:             T k = a - 1;
1499:             T n = b + k;
1500:             fract = binomial_ccdf(n, k, x, y, pol);
1501:             if(!normalised)
1502:                fract *= boost::math::beta(a, b, pol);
1503:             BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1504:          }
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_SWAP, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_SWAP, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1505-1536 / 第 1505-1536 行
~~~cpp
1505:          else if(b * x <= 0.7)
1506:          {
1507:             if(!invert)
1508:             {
1509:                fract = ibeta_series(a, b, x, T(0), lanczos_type(), normalised, p_derivative, y, pol);
1510:                BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1511:             }
1512:             else
1513:             {
1514:                fract = -(normalised ? 1 : boost::math::beta(a, b, pol));
1515:                invert = false;
1516:                fract = -ibeta_series(a, b, x, fract, lanczos_type(), normalised, p_derivative, y, pol);
1517:                BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1518:             }
1519:          }
1520:          else if(a > 15)
1521:          {
1522:             // sidestep so we can use the series representation:
1523:             int n = itrunc(T(floor(b)), pol);
1524:             if(n == b)
1525:                --n;
1526:             T bbar = b - n;
1527:             T prefix; // LCOV_EXCL_LINE
1528:             if(!normalised)
1529:             {
1530:                prefix = rising_factorial_ratio(T(a+bbar), bbar, n);
1531:             }
1532:             else
1533:             {
1534:                prefix = 1;
1535:             }
1536:             fract = ibeta_a_step(bbar, a, y, x, n, pol, normalised, static_cast<T*>(nullptr));
~~~
- **EN:** This range declares or defines callable logic such as ibeta_series, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ibeta_series, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1537-1568 / 第 1537-1568 行
~~~cpp
1537:             fract = beta_small_b_large_a_series(a,  bbar, x, y, fract, T(1), pol, normalised);
1538:             fract /= prefix;
1539:             BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1540:          }
1541:          else if(normalised)
1542:          {
1543:             // The formula here for the non-normalised case is tricky to figure
1544:             // out (for me!!), and requires two pochhammer calculations rather
1545:             // than one, so leave it for now and only use this in the normalized case....
1546:             int n = itrunc(T(floor(b)), pol);
1547:             T bbar = b - n;
1548:             if(bbar <= 0)
1549:             {
1550:                --n;
1551:                bbar += 1;
1552:             }
1553:             fract = ibeta_a_step(bbar, a, y, x, n, pol, normalised, static_cast<T*>(nullptr));
1554:             fract += ibeta_a_step(a, bbar, x, y, 20, pol, normalised, static_cast<T*>(nullptr));
1555:             if(invert)
1556:                fract -= 1;  // Note this line would need changing if we ever enable this branch in non-normalized case
1557:             fract = beta_small_b_large_a_series(T(a+20),  bbar, x, y, fract, T(1), pol, normalised);
1558:             if(invert)
1559:             {
1560:                fract = -fract;
1561:                invert = false;
1562:             }
1563:             BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1564:          }
1565:          else
1566:          {
1567:             fract = ibeta_fraction2(a, b, x, y, pol, normalised, p_derivative);
1568:             BOOST_MATH_INSTRUMENT_VARIABLE(fract);
~~~
- **EN:** This range declares or defines callable logic such as beta_small_b_large_a_series, BOOST_MATH_INSTRUMENT_VARIABLE, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 beta_small_b_large_a_series, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1569-1600 / 第 1569-1600 行
~~~cpp
1569:          }
1570:       }
1571:       else
1572:       {
1573:          // a and b both large:
1574:          bool use_asym = false;
1575:          T ma = BOOST_MATH_GPU_SAFE_MAX(a, b);
1576:          T xa = ma == a ? x : y;
1577:          T saddle = ma / (a + b);
1578:          T powers = 0;
1579:          if ((ma > 1e-5f / tools::epsilon<T>()) && (ma / BOOST_MATH_GPU_SAFE_MIN(a, b) < (xa < saddle ? 2 : 15)))
1580:          {
1581:             if (a == b)
1582:                use_asym = true;
1583:             else
1584:             {
1585:                powers = exp(log(x / (a / (a + b))) * a + log(y / (b / (a + b))) * b);
1586:                if (powers < tools::epsilon<T>())
1587:                   use_asym = true;
1588:             }
1589:          }
1590:          if(use_asym)
1591:          {
1592:             fract = ibeta_large_ab(a, b, x, y, invert, normalised, pol);
1593:             if (fract * tools::epsilon<T>() < powers)
1594:             {
1595:                // Erf approximation failed, correction term is too large, fall back:
1596:                fract = ibeta_fraction2(a, b, x, y, pol, normalised, p_derivative);
1597:             }
1598:             else
1599:                invert = false;
1600:          }
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_MAX, exp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_MAX, exp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1601-1632 / 第 1601-1632 行
~~~cpp
1601:          else
1602:             fract = ibeta_fraction2(a, b, x, y, pol, normalised, p_derivative);
1603: 
1604:          BOOST_MATH_INSTRUMENT_VARIABLE(fract);
1605:       }
1606:    }
1607:    if(p_derivative)
1608:    {
1609:       if(*p_derivative < 0)
1610:       {
1611:          *p_derivative = ibeta_power_terms(a, b, x, y, lanczos_type(), true, pol);
1612:       }
1613:       T div = y * x;
1614: 
1615:       if(*p_derivative != 0)
1616:       {
1617:          if((tools::max_value<T>() * div < *p_derivative))
1618:          {
1619:             // overflow, return an arbitrarily large value:
1620:             *p_derivative = tools::max_value<T>() / 2;   // LCOV_EXCL_LINE  Probably can only get here with denormalized x.
1621:          }
1622:          else
1623:          {
1624:             *p_derivative /= div;
1625:          }
1626:       }
1627:    }
1628:    return invert ? (normalised ? 1 : boost::math::beta(a, b, pol)) - fract : fract;
1629: } // template <class T, class Lanczos>T ibeta_imp(T a, T b, T x, const Lanczos& l, bool inv, bool normalised)
1630: 
1631: template <class T, class Policy>
1632: BOOST_MATH_GPU_ENABLED inline T ibeta_imp(T a, T b, T x, const Policy& pol, bool inv, bool normalised)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as ibeta_fraction2, BOOST_MATH_INSTRUMENT_VARIABLE, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 ibeta_fraction2, BOOST_MATH_INSTRUMENT_VARIABLE, ...。

### Lines 1633-1664 / 第 1633-1664 行
~~~cpp
1633: {
1634:    return ibeta_imp(a, b, x, pol, inv, normalised, static_cast<T*>(nullptr));
1635: }
1636: 
1637: template <class T, class Policy>
1638: BOOST_MATH_GPU_ENABLED T ibeta_derivative_imp(T a, T b, T x, const Policy& pol)
1639: {
1640:    constexpr auto function = "ibeta_derivative<%1%>(%1%,%1%,%1%)";
1641:    //
1642:    // start with the usual error checks:
1643:    //
1644:    if (!(boost::math::isfinite)(a))
1645:       return policies::raise_domain_error<T>(function, "The argument a to the incomplete beta function must be finite (got a=%1%).", a, pol);
1646:    if (!(boost::math::isfinite)(b))
1647:       return policies::raise_domain_error<T>(function, "The argument b to the incomplete beta function must be finite (got b=%1%).", b, pol);
1648:    if (!(0 <= x && x <= 1))
1649:       return policies::raise_domain_error<T>(function, "The argument x to the incomplete beta function must be in [0,1] (got x=%1%).", x, pol);
1650: 
1651:    if(a <= 0)
1652:       return policies::raise_domain_error<T>(function, "The argument a to the incomplete beta function must be greater than zero (got a=%1%).", a, pol);
1653:    if(b <= 0)
1654:       return policies::raise_domain_error<T>(function, "The argument b to the incomplete beta function must be greater than zero (got b=%1%).", b, pol);
1655:    //
1656:    // Now the corner cases:
1657:    //
1658:    if(x == 0)
1659:    {
1660:       return (a > 1) ? 0 :
1661:          (a == 1) ? 1 / boost::math::beta(a, b, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);
1662:    }
1663:    else if(x == 1)
1664:    {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::beta.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::beta。

### Lines 1665-1696 / 第 1665-1696 行
~~~cpp
1665:       return (b > 1) ? 0 :
1666:          (b == 1) ? 1 / boost::math::beta(a, b, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);
1667:    }
1668:    //
1669:    // Now the regular cases:
1670:    //
1671:    typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
1672:    T y = (1 - x) * x;
1673:    T f1;
1674:    if (!(boost::math::isinf)(1 / y))
1675:    {
1676:       f1 = ibeta_power_terms<T>(a, b, x, 1 - x, lanczos_type(), true, pol, 1 / y, function);
1677:    }
1678:    else
1679:    {
1680:       return (a > 1) ? 0 : (a == 1) ? 1 / boost::math::beta(a, b, pol) : policies::raise_overflow_error<T>(function, nullptr, pol);
1681:    }
1682: 
1683:    return f1;
1684: }
1685: //
1686: // Some forwarding functions that disambiguate the third argument type:
1687: //
1688: template <class RT1, class RT2, class Policy>
1689: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2>::type
1690:    beta(RT1 a, RT2 b, const Policy&, const boost::math::true_type*)
1691: {
1692:    BOOST_FPU_EXCEPTION_GUARD
1693:    typedef typename tools::promote_args<RT1, RT2>::type result_type;
1694:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1695:    typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
1696:    typedef typename policies::normalise<
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RT1` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::beta, lanczos_type.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RT1`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::beta, lanczos_type。

### Lines 1697-1728 / 第 1697-1728 行
~~~cpp
1697:       Policy,
1698:       policies::promote_float<false>,
1699:       policies::promote_double<false>,
1700:       policies::discrete_quantile<>,
1701:       policies::assert_undefined<> >::type forwarding_policy;
1702: 
1703:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::beta_imp(static_cast<value_type>(a), static_cast<value_type>(b), evaluation_type(), forwarding_policy()), "boost::math::beta<%1%>(%1%,%1%)");
1704: }
1705: template <class RT1, class RT2, class RT3>
1706: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1707:    beta(RT1 a, RT2 b, RT3 x, const boost::math::false_type*)
1708: {
1709:    return boost::math::beta(a, b, x, policies::policy<>());
1710: }
1711: } // namespace detail
1712: 
1713: //
1714: // The actual function entry-points now follow, these just figure out
1715: // which Lanczos approximation to use
1716: // and forward to the implementation functions:
1717: //
1718: template <class RT1, class RT2, class A>
1719: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, A>::type
1720:    beta(RT1 a, RT2 b, A arg)
1721: {
1722:    using tag = typename policies::is_policy<A>::type;
1723:    using ReturnType = tools::promote_args_t<RT1, RT2, A>;
1724:    return static_cast<ReturnType>(boost::math::detail::beta(a, b, arg, static_cast<tag*>(nullptr)));
1725: }
1726: 
1727: template <class RT1, class RT2>
1728: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2>::type
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RT1` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RT1`，作为该文件核心抽象的一部分。

### Lines 1729-1760 / 第 1729-1760 行
~~~cpp
1729:    beta(RT1 a, RT2 b)
1730: {
1731:    return boost::math::beta(a, b, policies::policy<>());
1732: }
1733: 
1734: template <class RT1, class RT2, class RT3, class Policy>
1735: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1736:    beta(RT1 a, RT2 b, RT3 x, const Policy&)
1737: {
1738:    BOOST_FPU_EXCEPTION_GUARD
1739:    typedef typename tools::promote_args<RT1, RT2, RT3>::type result_type;
1740:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1741:    typedef typename policies::normalise<
1742:       Policy,
1743:       policies::promote_float<false>,
1744:       policies::promote_double<false>,
1745:       policies::discrete_quantile<>,
1746:       policies::assert_undefined<> >::type forwarding_policy;
1747: 
1748:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::ibeta_imp(static_cast<value_type>(a), static_cast<value_type>(b), static_cast<value_type>(x), forwarding_policy(), false, false), "boost::math::beta<%1%>(%1%,%1%,%1%)");
1749: }
1750: 
1751: template <class RT1, class RT2, class RT3, class Policy>
1752: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1753:    betac(RT1 a, RT2 b, RT3 x, const Policy&)
1754: {
1755:    BOOST_FPU_EXCEPTION_GUARD
1756:    typedef typename tools::promote_args<RT1, RT2, RT3>::type result_type;
1757:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1758:    typedef typename policies::normalise<
1759:       Policy,
1760:       policies::promote_float<false>,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RT1` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RT1`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 1761-1792 / 第 1761-1792 行
~~~cpp
1761:       policies::promote_double<false>,
1762:       policies::discrete_quantile<>,
1763:       policies::assert_undefined<> >::type forwarding_policy;
1764: 
1765:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::ibeta_imp(static_cast<value_type>(a), static_cast<value_type>(b), static_cast<value_type>(x), forwarding_policy(), true, false), "boost::math::betac<%1%>(%1%,%1%,%1%)");
1766: }
1767: template <class RT1, class RT2, class RT3>
1768: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1769:    betac(RT1 a, RT2 b, RT3 x)
1770: {
1771:    return boost::math::betac(a, b, x, policies::policy<>());
1772: }
1773: 
1774: template <class RT1, class RT2, class RT3, class Policy>
1775: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1776:    ibeta(RT1 a, RT2 b, RT3 x, const Policy&)
1777: {
1778:    BOOST_FPU_EXCEPTION_GUARD
1779:    typedef typename tools::promote_args<RT1, RT2, RT3>::type result_type;
1780:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1781:    typedef typename policies::normalise<
1782:       Policy,
1783:       policies::promote_float<false>,
1784:       policies::promote_double<false>,
1785:       policies::discrete_quantile<>,
1786:       policies::assert_undefined<> >::type forwarding_policy;
1787: 
1788:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::ibeta_imp(static_cast<value_type>(a), static_cast<value_type>(b), static_cast<value_type>(x), forwarding_policy(), false, true), "boost::math::ibeta<%1%>(%1%,%1%,%1%)");
1789: }
1790: template <class RT1, class RT2, class RT3>
1791: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1792:    ibeta(RT1 a, RT2 b, RT3 x)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RT1` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RT1`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 1793-1824 / 第 1793-1824 行
~~~cpp
1793: {
1794:    return boost::math::ibeta(a, b, x, policies::policy<>());
1795: }
1796: 
1797: template <class RT1, class RT2, class RT3, class Policy>
1798: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1799:    ibetac(RT1 a, RT2 b, RT3 x, const Policy&)
1800: {
1801:    BOOST_FPU_EXCEPTION_GUARD
1802:    typedef typename tools::promote_args<RT1, RT2, RT3>::type result_type;
1803:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1804:    typedef typename policies::normalise<
1805:       Policy,
1806:       policies::promote_float<false>,
1807:       policies::promote_double<false>,
1808:       policies::discrete_quantile<>,
1809:       policies::assert_undefined<> >::type forwarding_policy;
1810: 
1811:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::ibeta_imp(static_cast<value_type>(a), static_cast<value_type>(b), static_cast<value_type>(x), forwarding_policy(), true, true), "boost::math::ibetac<%1%>(%1%,%1%,%1%)");
1812: }
1813: template <class RT1, class RT2, class RT3>
1814: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1815:    ibetac(RT1 a, RT2 b, RT3 x)
1816: {
1817:    return boost::math::ibetac(a, b, x, policies::policy<>());
1818: }
1819: 
1820: template <class RT1, class RT2, class RT3, class Policy>
1821: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1822:    ibeta_derivative(RT1 a, RT2 b, RT3 x, const Policy&)
1823: {
1824:    BOOST_FPU_EXCEPTION_GUARD
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RT1` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RT1`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 1825-1849 / 第 1825-1849 行
~~~cpp
1825:    typedef typename tools::promote_args<RT1, RT2, RT3>::type result_type;
1826:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
1827:    typedef typename policies::normalise<
1828:       Policy,
1829:       policies::promote_float<false>,
1830:       policies::promote_double<false>,
1831:       policies::discrete_quantile<>,
1832:       policies::assert_undefined<> >::type forwarding_policy;
1833: 
1834:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(detail::ibeta_derivative_imp(static_cast<value_type>(a), static_cast<value_type>(b), static_cast<value_type>(x), forwarding_policy()), "boost::math::ibeta_derivative<%1%>(%1%,%1%,%1%)");
1835: }
1836: template <class RT1, class RT2, class RT3>
1837: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<RT1, RT2, RT3>::type
1838:    ibeta_derivative(RT1 a, RT2 b, RT3 x)
1839: {
1840:    return boost::math::ibeta_derivative(a, b, x, policies::policy<>());
1841: }
1842: 
1843: } // namespace math
1844: } // namespace boost
1845: 
1846: #include <boost/math/special_functions/detail/ibeta_inverse.hpp>
1847: #include <boost/math/special_functions/detail/ibeta_inv_ab.hpp>
1848: 
1849: #endif // BOOST_MATH_SPECIAL_BETA_HPP
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/detail/ibeta_inverse.hpp, boost/math/special_functions/detail/ibeta_inv_ab.hpp so the surrounding code can use external declarations. The code enters namespace scope (math::boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/special_functions/detail/ibeta_inverse.hpp, boost/math/special_functions/detail/ibeta_inv_ab.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/assert.hpp, boost/math/tools/precision.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/tuple.hpp, boost/math/tools/promotion.hpp, boost/math/tools/cstdint.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/erf.hpp, boost/math/special_functions/log1p.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `BOOST_MATH_GPU_SAFE_SWAP, Lanczos::g, Lanczos::lanczos_sum_expG_scaled, exp, pow, sqrt, ltrunc, beta_imp, ...`
