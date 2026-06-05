# polygamma.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/polygamma.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the polygamma special-function path.
- **作用（中文）**: 此头文件为 polygamma 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: 
   2: ///////////////////////////////////////////////////////////////////////////////
   3: //  Copyright 2013 Nikhar Agrawal
   4: //  Copyright 2013 Christopher Kormanyos
   5: //  Copyright 2014 John Maddock
   6: //  Copyright 2013 Paul Bristow
   7: //  Distributed under the Boost
   8: //  Software License, Version 1.0. (See accompanying file
   9: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
  10: 
  11: #ifndef _BOOST_POLYGAMMA_DETAIL_2013_07_30_HPP_
  12:   #define _BOOST_POLYGAMMA_DETAIL_2013_07_30_HPP_
  13: 
  14: #include <cmath>
  15: #include <limits>
  16: #include <string>
  17: #include <boost/math/policies/policy.hpp>
  18: #include <boost/math/special_functions/bernoulli.hpp>
~~~
- **EN:** This block imports dependencies such as cmath, limits, string, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 cmath, limits, string, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/special_functions/trunc.hpp>
  20: #include <boost/math/special_functions/zeta.hpp>
  21: #include <boost/math/special_functions/digamma.hpp>
  22: #include <boost/math/special_functions/sin_pi.hpp>
  23: #include <boost/math/special_functions/cos_pi.hpp>
  24: #include <boost/math/special_functions/pow.hpp>
  25: #include <boost/math/tools/assert.hpp>
  26: #include <boost/math/tools/config.hpp>
  27: 
  28: #ifdef BOOST_MATH_HAS_THREADS
  29: #include <mutex>
  30: #endif
  31: 
  32: #ifdef _MSC_VER
  33: #pragma once
  34: #pragma warning(push)
  35: #pragma warning(disable:4702) // Unreachable code (release mode only warning)
  36: #endif
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/trunc.hpp, boost/math/special_functions/zeta.hpp, boost/math/special_functions/digamma.hpp, ... so the surrounding code can use external declarations. It begins the file-level inclusion guard or portability wrapper. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/special_functions/trunc.hpp, boost/math/special_functions/zeta.hpp, boost/math/special_functions/digamma.hpp, ... 等依赖，使周围代码可以使用外部声明。 它开始设置文件级防重包含或可移植性包装层。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: 
  38: namespace boost { namespace math { namespace detail{
  39: 
  40:   template<class T, class Policy>
  41:   T polygamma_atinfinityplus(const int n, const T& x, const Policy& pol, const char* function) // for large values of x such as for x> 400
  42:   {
  43:      // See http://functions.wolfram.com/GammaBetaErf/PolyGamma2/06/02/0001/
  44:      BOOST_MATH_STD_USING
  45:      //
  46:      // sum       == current value of accumulated sum.
  47:      // term      == value of current term to be added to sum.
  48:      // part_term == value of current term excluding the Bernoulli number part
  49:      //
  50:      if(n + x == x)
  51:      {
  52:         // x is crazy large, just concentrate on the first part of the expression and use logs:
  53:         if(n == 1) return 1 / x;
  54:         T nlx = n * log(x);
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:         if((nlx < tools::log_max_value<T>()) && (n < (int)max_factorial<T>::value))
  56:            return ((n & 1) ? 1 : -1) * boost::math::factorial<T>(n - 1, pol) * static_cast<T>(pow(x, T(-n)));
  57:         else
  58:          return ((n & 1) ? 1 : -1) * exp(boost::math::lgamma(T(n), pol) - n * log(x));
  59:      }
  60:      T term, sum, part_term;
  61:      T x_squared = x * x;
  62:      //
  63:      // Start by setting part_term to:
  64:      //
  65:      // (n-1)! / x^(n+1)
  66:      //
  67:      // which is common to both the first term of the series (with k = 1)
  68:      // and to the leading part.
  69:      // We can then get to the leading term by:
  70:      //
  71:      // part_term * (n + 2 * x) / 2
  72:      //
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:      // and to the first term in the series
  74:      // (excluding the Bernoulli number) by:
  75:      //
  76:      // part_term n * (n + 1) / (2x)
  77:      //
  78:      // If either the factorial would overflow,
  79:      // or the power term underflows, this just gets set to 0 and then we
  80:      // know that we have to use logs for the initial terms:
  81:      //
  82:      part_term = ((n > (int)boost::math::max_factorial<T>::value) && (T(n) * n > tools::log_max_value<T>()))
  83:         ? T(0) : static_cast<T>(boost::math::factorial<T>(n - 1, pol) * pow(x, T(-n - 1)));
  84:      if(part_term == 0)
  85:      {
  86:         // Either n is very large, or the power term underflows,
  87:         // set the initial values of part_term, term and sum via logs:
  88:         part_term = static_cast<T>(T(boost::math::lgamma(n, pol)) - (n + 1) * log(x));
  89:         sum = exp(part_term + log(n + 2 * x) - boost::math::constants::ln_two<T>());
  90:         part_term += log(T(n) * (n + 1)) - boost::math::constants::ln_two<T>() - log(x);
~~~
- **EN:** This range declares or defines callable logic such as T, exp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T, exp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:         part_term = exp(part_term);
  92:      }
  93:      else
  94:      {
  95:         sum = part_term * (n + 2 * x) / 2;
  96:         part_term *= (T(n) * (n + 1)) / 2;
  97:         part_term /= x;
  98:      }
  99:      //
 100:      // If the leading term is 0, so is the result:
 101:      //
 102:      if(sum == 0)
 103:         return sum;
 104: 
 105:      for(unsigned k = 1;;)
 106:      {
 107:         term = part_term * boost::math::bernoulli_b2n<T>(k, pol);
 108:         sum += term;
~~~
- **EN:** This range declares or defines callable logic such as exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:         //
 110:         // Normal termination condition:
 111:         //
 112:         if(fabs(term / sum) < tools::epsilon<T>())
 113:            break;
 114:         //
 115:         // Increment our counter, and move part_term on to the next value:
 116:         //
 117:         ++k;
 118:         part_term *= T(n + 2 * k - 2) * (n - 1 + 2 * k);
 119:         part_term /= (2 * k - 1) * 2 * k;
 120:         part_term /= x_squared;
 121:         //
 122:         // Emergency get out termination condition:
 123:         //
 124:         if(k > policies::get_max_series_iterations<Policy>())
 125:         {
 126:            return policies::raise_evaluation_error(function, "Series did not converge, closest value was %1%", sum, pol);
~~~
- **EN:** This range declares or defines callable logic such as T. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:         }
 128:      }
 129: 
 130:      if((n - 1) & 1)
 131:         sum = -sum;
 132: 
 133:      return sum;
 134:   }
 135: 
 136:   template<class T, class Policy>
 137:   T polygamma_attransitionplus(const int n, const T& x, const Policy& pol, const char* function)
 138:   {
 139:     // See: http://functions.wolfram.com/GammaBetaErf/PolyGamma2/16/01/01/0017/
 140: 
 141:     // Use N = (0.4 * digits) + (4 * n) for target value for x:
 142:     BOOST_MATH_STD_USING
 143:     const int d4d  = static_cast<int>(0.4F * policies::digits_base10<T, Policy>());
 144:     const int N = d4d + (4 * n);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:     const int m    = n;
 146:     const int iter = N - itrunc(x);
 147: 
 148:     if(iter > (int)policies::get_max_series_iterations<Policy>())
 149:        return policies::raise_evaluation_error<T>(function, ("Exceeded maximum series evaluations evaluating at n = " + std::to_string(n) + " and x = %1%").c_str(), x, pol);
 150: 
 151:     const int minus_m_minus_one = -m - 1;
 152: 
 153:     T z(x);
 154:     T sum0(0);
 155:     T z_plus_k_pow_minus_m_minus_one(0);
 156: 
 157:     // Forward recursion to larger x, need to check for overflow first though:
 158:     if(log(z + iter) * minus_m_minus_one > -tools::log_max_value<T>())
 159:     {
 160:        for(int k = 1; k <= iter; ++k)
 161:        {
 162:           z_plus_k_pow_minus_m_minus_one = static_cast<T>(pow(z, T(minus_m_minus_one)));
~~~
- **EN:** This range declares or defines callable logic such as itrunc, z, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc, z, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:           sum0 += z_plus_k_pow_minus_m_minus_one;
 164:           z += 1;
 165:        }
 166:        sum0 *= boost::math::factorial<T>(n, pol);
 167:     }
 168:     else
 169:     {
 170:        for(int k = 1; k <= iter; ++k)
 171:        {
 172:           T log_term = log(z) * minus_m_minus_one + boost::math::lgamma(T(n + 1), pol);
 173:           sum0 += exp(log_term);
 174:           z += 1;
 175:        }
 176:     }
 177:     if((n - 1) & 1)
 178:        sum0 = -sum0;
 179: 
 180:     return sum0 + polygamma_atinfinityplus(n, z, pol, function);
~~~
- **EN:** This range declares or defines callable logic such as log, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:   }
 182: 
 183:   template <class T, class Policy>
 184:   T polygamma_nearzero(int n, T x, const Policy& pol, const char* function)
 185:   {
 186:      BOOST_MATH_STD_USING
 187:      //
 188:      // If we take this expansion for polygamma: http://functions.wolfram.com/06.15.06.0003.02
 189:      // and substitute in this expression for polygamma(n, 1): http://functions.wolfram.com/06.15.03.0009.01
 190:      // we get an alternating series for polygamma when x is small in terms of zeta functions of
 191:      // integer arguments (which are easy to evaluate, at least when the integer is even).
 192:      //
 193:      // In order to avoid spurious overflow, save the n! term for later, and rescale at the end:
 194:      //
 195:      T scale = boost::math::factorial<T>(n, pol);
 196:      //
 197:      // "factorial_part" contains everything except the zeta function
 198:      // evaluations in each term:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:      //
 200:      T factorial_part = 1;
 201:      //
 202:      // "prefix" is what we'll be adding the accumulated sum to, it will
 203:      // be n! / z^(n+1), but since we're scaling by n! it's just
 204:      // 1 / z^(n+1) for now:
 205:      //
 206:      T prefix = static_cast<T>(pow(x, T(n + 1)));  // Warning supression: Integer power returns at least a double
 207:      if(prefix == 0)
 208:         return boost::math::policies::raise_overflow_error<T>(function, nullptr, pol);
 209:      prefix = 1 / prefix;
 210:      //
 211:      // First term in the series is necessarily < zeta(2) < 2, so
 212:      // ignore the sum if it will have no effect on the result anyway:
 213:      //
 214:      if(prefix > 2 / policies::get_epsilon<T, Policy>())
 215:         return ((n & 1) ? 1 : -1) *
 216:          (tools::max_value<T>() / prefix < scale ? policies::raise_overflow_error<T>(function, nullptr, pol) : prefix * scale);
~~~
- **EN:** This range declares or defines callable logic such as pow. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:      //
 218:      // As this is an alternating series we could accelerate it using
 219:      // "Convergence Acceleration of Alternating Series",
 220:      // Henri Cohen, Fernando Rodriguez Villegas, and Don Zagier, Experimental Mathematics, 1999.
 221:      // In practice however, it appears not to make any difference to the number of terms
 222:      // required except in some edge cases which are filtered out anyway before we get here.
 223:      //
 224:      T sum = prefix;
 225:      for(unsigned k = 0;;)
 226:      {
 227:         // Get the k'th term:
 228:         T term = factorial_part * boost::math::zeta(T(k + n + 1), pol);
 229:         sum += term;
 230:         // Termination condition:
 231:         if(fabs(term) < fabs(sum * boost::math::policies::get_epsilon<T, Policy>()))
 232:            break;
 233:         //
 234:         // Move on k and factorial_part:
~~~
- **EN:** This range declares or defines callable logic such as boost::math::zeta. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::zeta。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:         //
 236:         ++k;
 237:         factorial_part *= (-x * (n + k)) / k;
 238:         //
 239:         // Last chance exit:
 240:         //
 241:         if(k > policies::get_max_series_iterations<Policy>())
 242:            return policies::raise_evaluation_error<T>(function, "Series did not converge, best value is %1%", sum, pol);
 243:      }
 244:      //
 245:      // We need to multiply by the scale, at each stage checking for overflow:
 246:      //
 247:      if(boost::math::tools::max_value<T>() / scale < sum)
 248:         return boost::math::policies::raise_overflow_error<T>(function, nullptr, pol);
 249:      sum *= scale;
 250:      return n & 1 ? sum : T(-sum);
 251:   }
 252: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:   //
 254:   // Helper function which figures out which slot our coefficient is in
 255:   // given an angle multiplier for the cosine term of power:
 256:   //
 257:   template <class Table>
 258:   typename Table::value_type::reference dereference_table(Table& table, unsigned row, unsigned power)
 259:   {
 260:      return table[row][power / 2];
 261:   }
 262: 
 263: 
 264: 
 265:   template <class T, class Policy>
 266:   T poly_cot_pi(int n, T x, T xc, const Policy& pol, const char* function)
 267:   {
 268:      BOOST_MATH_STD_USING
 269:      // Return n'th derivative of cot(pi*x) at x, these are simply
 270:      // tabulated for up to n = 9, beyond that it is possible to
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `Table` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `Table`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:      // calculate coefficients as follows:
 272:      //
 273:      // The general form of each derivative is:
 274:      //
 275:      // pi^n * SUM{k=0, n} C[k,n] * cos^k(pi * x) * csc^(n+1)(pi * x)
 276:      //
 277:      // With constant C[0,1] = -1 and all other C[k,n] = 0;
 278:      // Then for each k < n+1:
 279:      // C[k-1, n+1]  -= k * C[k, n];
 280:      // C[k+1, n+1]  += (k-n-1) * C[k, n];
 281:      //
 282:      // Note that there are many different ways of representing this derivative thanks to
 283:      // the many trigonometric identies available.  In particular, the sum of powers of
 284:      // cosines could be replaced by a sum of cosine multiple angles, and indeed if you
 285:      // plug the derivative into Mathematica this is the form it will give.  The two
 286:      // forms are related via the Chebeshev polynomials of the first kind and
 287:      // T_n(cos(x)) = cos(n x).  The polynomial form has the great advantage that
 288:      // all the cosine terms are zero at half integer arguments - right where this
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:      // function has it's minimum - thus avoiding cancellation error in this region.
 290:      //
 291:      // And finally, since every other term in the polynomials is zero, we can save
 292:      // space by only storing the non-zero terms.  This greatly complexifies
 293:      // subscripting the tables in the calculation, but halves the storage space
 294:      // (and complexity for that matter).
 295:      //
 296:      T s = fabs(x) < fabs(xc) ? boost::math::sin_pi(x, pol) : boost::math::sin_pi(xc, pol);
 297:      T c = boost::math::cos_pi(x, pol);
 298:      switch(n)
 299:      {
 300:      case 1:
 301:         return -constants::pi<T, Policy>() / (s * s);
 302:      case 2:
 303:      {
 304:         return 2 * constants::pi<T, Policy>() * constants::pi<T, Policy>() * c / boost::math::pow<3>(s, pol);
 305:      }
 306:      case 3:
~~~
- **EN:** This range declares or defines callable logic such as fabs, boost::math::cos_pi. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs, boost::math::cos_pi。 return 语句会把计算结果或状态返回给调用方。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:      {
 308:         constexpr int P[] = { -2, -4 };
 309:         return boost::math::pow<3>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<4>(s, pol);
 310:      }
 311:      case 4:
 312:      {
 313:         constexpr int P[] = { 16, 8 };
 314:         return boost::math::pow<4>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<5>(s, pol);
 315:      }
 316:      case 5:
 317:      {
 318:         constexpr int P[] = { -16, -88, -16 };
 319:         return boost::math::pow<5>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<6>(s, pol);
 320:      }
 321:      case 6:
 322:      {
 323:         constexpr int P[] = { 272, 416, 32 };
 324:         return boost::math::pow<6>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<7>(s, pol);
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:      }
 326:      case 7:
 327:      {
 328:         constexpr int P[] = { -272, -2880, -1824, -64 };
 329:         return boost::math::pow<7>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<8>(s, pol);
 330:      }
 331:      case 8:
 332:      {
 333:         constexpr int P[] = { 7936, 24576, 7680, 128 };
 334:         return boost::math::pow<8>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<9>(s, pol);
 335:      }
 336:      case 9:
 337:      {
 338:         constexpr int P[] = { -7936, -137216, -185856, -31616, -256 };
 339:         return boost::math::pow<9>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<10>(s, pol);
 340:      }
 341:      case 10:
 342:      {
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:         constexpr int P[] = { 353792, 1841152, 1304832, 128512, 512 };
 344:         return boost::math::pow<10>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<11>(s, pol);
 345:      }
 346:      case 11:
 347:      {
 348:         constexpr int P[] = { -353792, -9061376, -21253376, -8728576, -518656, -1024};
 349:         return boost::math::pow<11>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<12>(s, pol);
 350:      }
 351:      case 12:
 352:      {
 353:         constexpr int P[] = { 22368256, 175627264, 222398464, 56520704, 2084864, 2048 };
 354:         return boost::math::pow<12>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<13>(s, pol);
 355:      }
 356: #ifndef BOOST_NO_LONG_LONG
 357:      case 13:
 358:      {
 359:         constexpr long long P[] = { -22368256LL, -795300864LL, -2868264960LL, -2174832640LL, -357888000LL, -8361984LL, -4096 };
 360:         return boost::math::pow<13>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<14>(s, pol);
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:      }
 362:      case 14:
 363:      {
 364:         constexpr long long P[] = { 1903757312LL, 21016670208LL, 41731645440LL, 20261765120LL, 2230947840LL, 33497088LL, 8192 };
 365:         return boost::math::pow<14>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<15>(s, pol);
 366:      }
 367:      case 15:
 368:      {
 369:         constexpr long long P[] = { -1903757312LL, -89702612992LL, -460858269696LL, -559148810240LL, -182172651520LL, -13754155008LL, -134094848LL, -16384 };
 370:         return boost::math::pow<15>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<16>(s, pol);
 371:      }
 372:      case 16:
 373:      {
 374:         constexpr long long P[] = { 209865342976LL, 3099269660672LL, 8885192097792LL, 7048869314560LL, 1594922762240LL, 84134068224LL, 536608768LL, 32768 };
 375:         return boost::math::pow<16>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<17>(s, pol);
 376:      }
 377:      case 17:
 378:      {
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:         constexpr long long P[] = { -209865342976LL, -12655654469632LL, -87815735738368LL, -155964390375424LL, -84842998005760LL, -13684856848384LL, -511780323328LL, -2146926592LL, -65536 };
 380:         return boost::math::pow<17>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<18>(s, pol);
 381:      }
 382:      case 18:
 383:      {
 384:         constexpr long long P[] = { 29088885112832LL, 553753414467584LL, 2165206642589696LL, 2550316668551168LL, 985278548541440LL, 115620218667008LL, 3100738912256LL, 8588754944LL, 131072 };
 385:         return boost::math::pow<18>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<19>(s, pol);
 386:      }
 387:      case 19:
 388:      {
 389:         constexpr long long P[] = { -29088885112832LL, -2184860175433728LL, -19686087844429824LL, -48165109676113920LL, -39471306959486976LL, -11124607890751488LL, -965271355195392LL, -18733264797696LL, -34357248000LL, -262144 };
 390:         return boost::math::pow<19>(constants::pi<T, Policy>(), pol) * tools::evaluate_even_polynomial(P, c) / boost::math::pow<20>(s, pol);
 391:      }
 392:      case 20:
 393:      {
 394:         constexpr long long P[] = { 4951498053124096LL, 118071834535526400LL, 603968063567560704LL, 990081991141490688LL, 584901762421358592LL, 122829335169859584LL, 7984436548730880LL, 112949304754176LL, 137433710592LL, 524288 };
 395:         return boost::math::pow<20>(constants::pi<T, Policy>(), pol) * c * tools::evaluate_even_polynomial(P, c) / boost::math::pow<21>(s, pol);
 396:      }
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397: #endif
 398:      }
 399: 
 400:      //
 401:      // We'll have to compute the coefficients up to n,
 402:      // complexity is O(n^2) which we don't worry about for now
 403:      // as the values are computed once and then cached.
 404:      // However, if the final evaluation would have too many
 405:      // terms just bail out right away:
 406:      //
 407:      if((unsigned)n / 2u > policies::get_max_series_iterations<Policy>())
 408:         return policies::raise_evaluation_error<T>(function, "The value of n is so large that we're unable to compute the result in reasonable time, best guess is %1%", 0, pol);
 409: #ifdef BOOST_MATH_HAS_THREADS
 410:      static std::mutex m;
 411:      std::lock_guard<std::mutex> l(m);
 412: #endif
 413:      static int digits = tools::digits<T>();
 414:      static std::vector<std::vector<T> > table(1, std::vector<T>(1, T(-1)));
~~~
- **EN:** This range declares or defines callable logic such as l, table. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 l, table。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415: 
 416:      int current_digits = tools::digits<T>();
 417: 
 418:      if(digits != current_digits)
 419:      {
 420:         // Oh my... our precision has changed!
 421:         table = std::vector<std::vector<T> >(1, std::vector<T>(1, T(-1)));
 422:         digits = current_digits;
 423:      }
 424: 
 425:      int index = n - 1;
 426: 
 427:      if(index >= (int)table.size())
 428:      {
 429:         for(int i = (int)table.size() - 1; i < index; ++i)
 430:         {
 431:            int offset = i & 1; // 1 if the first cos power is 0, otherwise 0.
 432:            int sin_order = i + 2;  // order of the sin term
~~~
- **EN:** This range declares or defines callable logic such as T. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:            int max_cos_order = sin_order - 1;  // largest order of the polynomial of cos terms
 434:            int max_columns = (max_cos_order - offset) / 2;  // How many entries there are in the current row.
 435:            int next_offset = offset ? 0 : 1;
 436:            int next_max_columns = (max_cos_order + 1 - next_offset) / 2;  // How many entries there will be in the next row
 437:            table.push_back(std::vector<T>(next_max_columns + 1, T(0)));
 438: 
 439:            for(int column = 0; column <= max_columns; ++column)
 440:            {
 441:               int cos_order = 2 * column + offset;  // order of the cosine term in entry "column"
 442:               BOOST_MATH_ASSERT(column < (int)table[i].size());
 443:               BOOST_MATH_ASSERT((cos_order + 1) / 2 < (int)table[i + 1].size());
 444:               table[i + 1][(cos_order + 1) / 2] += ((cos_order - sin_order) * table[i][column]) / (sin_order - 1);
 445:               if(cos_order)
 446:                 table[i + 1][(cos_order - 1) / 2] += (-cos_order * table[i][column]) / (sin_order - 1);
 447:            }
 448:         }
 449: 
 450:      }
~~~
- **EN:** This range declares or defines callable logic such as push_back, BOOST_MATH_ASSERT. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 push_back, BOOST_MATH_ASSERT。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:      T sum = boost::math::tools::evaluate_even_polynomial(&table[index][0], c, table[index].size());
 452:      if(index & 1)
 453:         sum *= c;  // First coefficient is order 1, and really an odd polynomial.
 454:      if(sum == 0)
 455:         return sum;
 456:      //
 457:      // The remaining terms are computed using logs since the powers and factorials
 458:      // get real large real quick:
 459:      //
 460:      T power_terms = n * log(boost::math::constants::pi<T>());
 461:      if(s == 0)
 462:         return sum * boost::math::policies::raise_overflow_error<T>(function, nullptr, pol);
 463:      power_terms -= log(fabs(s)) * (n + 1);
 464:      power_terms += boost::math::lgamma(T(n), pol);
 465:      power_terms += log(fabs(sum));
 466: 
 467:      if(power_terms > boost::math::tools::log_max_value<T>())
 468:         return sum * boost::math::policies::raise_overflow_error<T>(function, nullptr, pol);
~~~
- **EN:** This range declares or defines callable logic such as boost::math::tools::evaluate_even_polynomial, log, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::tools::evaluate_even_polynomial, log, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469: 
 470:      return exp(power_terms) * ((s < 0) && ((n + 1) & 1) ? -1 : 1) * boost::math::sign(sum);
 471:   }
 472: 
 473:   template<class T, class Policy>
 474:   inline T polygamma_imp(const int n, T x, const Policy &pol)
 475:   {
 476:     BOOST_MATH_STD_USING
 477:     static const char* function = "boost::math::polygamma<%1%>(int, %1%)";
 478: 
 479:     if(n < 0)
 480:        return policies::raise_domain_error<T>(function, "Order must be >= 0, but got %1%", static_cast<T>(n), pol);
 481:     if(x < 0)
 482:     {
 483:        if(floor(x) == x)
 484:        {
 485:           //
 486:           // Result is infinity if x is odd, and a pole error if x is even.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:           //
 488:           if(lltrunc(x) & 1)
 489:              return policies::raise_overflow_error<T>(function, nullptr, pol);
 490:           else
 491:              return policies::raise_pole_error<T>(function, "Evaluation at negative integer %1%", x, pol);
 492:        }
 493:        T z = 1 - x;
 494:        T result = polygamma_imp(n, z, pol) + constants::pi<T, Policy>() * poly_cot_pi(n, z, x, pol, function);
 495:        return n & 1 ? T(-result) : result;
 496:     }
 497:     //
 498:     // Limit for use of small-x-series is chosen
 499:     // so that the series doesn't go too divergent
 500:     // in the first few terms.  Ordinarily this
 501:     // would mean setting the limit to ~ 1 / n,
 502:     // but we can tolerate a small amount of divergence:
 503:     //
 504:     T small_x_limit = (std::min)(T(T(5) / n), T(0.25f));
~~~
- **EN:** This range declares or defines callable logic such as polygamma_imp, T. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 polygamma_imp, T。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:     if(x < small_x_limit)
 506:     {
 507:       return polygamma_nearzero(n, x, pol, function);
 508:     }
 509:     else if(x > 0.4F * policies::digits_base10<T, Policy>() + 4.0f * n)
 510:     {
 511:       return polygamma_atinfinityplus(n, x, pol, function);
 512:     }
 513:     else if(x == 1)
 514:     {
 515:        return (n & 1 ? 1 : -1) * boost::math::factorial<T>(n, pol) * boost::math::zeta(T(n + 1), pol);
 516:     }
 517:     else if(x == 0.5f)
 518:     {
 519:        T result = (n & 1 ? 1 : -1) * boost::math::factorial<T>(n, pol) * boost::math::zeta(T(n + 1), pol);
 520:        if(fabs(result) >= ldexp(tools::max_value<T>(), -n - 1))
 521:           return boost::math::sign(result) * policies::raise_overflow_error<T>(function, nullptr, pol);
 522:        result *= ldexp(T(1), n + 1) - 1;
~~~
- **EN:** This range declares or defines callable logic such as boost::math::zeta. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::zeta。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 523-538 / 第 523-538 行
~~~cpp
 523:        return result;
 524:     }
 525:     else
 526:     {
 527:       return polygamma_attransitionplus(n, x, pol, function);
 528:     }
 529:   }
 530: 
 531: } } } // namespace boost::math::detail
 532: 
 533: #ifdef _MSC_VER
 534: #pragma warning(pop)
 535: #endif
 536: 
 537: #endif // _BOOST_POLYGAMMA_DETAIL_2013_07_30_HPP_
 538: 
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `cmath, limits, string, boost/math/policies/policy.hpp, boost/math/special_functions/bernoulli.hpp, boost/math/special_functions/trunc.hpp, boost/math/special_functions/zeta.hpp, boost/math/special_functions/digamma.hpp, boost/math/special_functions/sin_pi.hpp, boost/math/special_functions/cos_pi.hpp, boost/math/special_functions/pow.hpp, boost/math/tools/assert.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `log, T, exp, itrunc, z, sum0, z_plus_k_pow_minus_m_minus_one, pow, ...`
