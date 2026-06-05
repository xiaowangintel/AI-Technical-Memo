# hypergeometric_1F1_bessel.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/hypergeometric_1F1_bessel.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the hypergeometric 1F1 bessel special-function path.
- **作用（中文）**: 此头文件为 hypergeometric 1F1 bessel 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行
~~~cpp
   1: ///////////////////////////////////////////////////////////////////////////////
   2: //  Copyright 2014 Anton Bikineev
   3: //  Copyright 2014 Christopher Kormanyos
   4: //  Copyright 2014 John Maddock
   5: //  Copyright 2014 Paul Bristow
   6: //  Distributed under the Boost
   7: //  Software License, Version 1.0. (See accompanying file
   8: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   9: //
  10: #ifndef BOOST_MATH_HYPERGEOMETRIC_1F1_BESSEL_HPP
  11: #define BOOST_MATH_HYPERGEOMETRIC_1F1_BESSEL_HPP
  12: 
  13: #include <boost/math/tools/series.hpp>
  14: #include <boost/math/special_functions/bessel.hpp>
  15: #include <boost/math/special_functions/laguerre.hpp>
  16: #include <boost/math/special_functions/detail/hypergeometric_pFq_checked_series.hpp>
  17: #include <boost/math/special_functions/bessel_iterators.hpp>
  18: 
  19: 
  20:   namespace boost { namespace math { namespace detail {
  21: 
  22:      template <class T, class Policy>
  23:      T hypergeometric_1F1_divergent_fallback(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling);
  24: 
~~~
- **EN:** This block imports dependencies such as boost/math/tools/series.hpp, boost/math/special_functions/bessel.hpp, boost/math/special_functions/laguerre.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/tools/series.hpp, boost/math/special_functions/bessel.hpp, boost/math/special_functions/laguerre.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 25-48 / 第 25-48 行
~~~cpp
  25:      template <class T>
  26:      bool hypergeometric_1F1_is_tricomi_viable_positive_b(const T& a, const T& b, const T& z)
  27:      {
  28:         BOOST_MATH_STD_USING
  29:            if ((z < b) && (a > -50))
  30:               return false;  // might as well fall through to recursion
  31:         if (b <= 100)
  32:            return true;
  33:         // Even though we're in a reasonable domain for Tricomi's approximation,
  34:         // the arguments to the Bessel functions may be so large that we can't
  35:         // actually evaluate them:
  36:         T x = sqrt(fabs(2 * z * b - 4 * a * z));
  37:         T v = b - 1;
  38:         return log(boost::math::constants::e<T>() * x / (2 * v)) * v > tools::log_min_value<T>();
  39:      }
  40: 
  41:      //
  42:      // Returns an arbitrarily small value compared to "target" for use as a seed
  43:      // value for Bessel recurrences.  Note that we'd better not make it too small
  44:      // or underflow may occur resulting in either one of the terms in the
  45:      // recurrence being zero, or else the result being zero.  Using 1/epsilon
  46:      // as a safety factor ensures that if we do underflow to zero, all of the digits
  47:      // will have been cancelled out anyway:
  48:      //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as sqrt.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 sqrt。

### Lines 49-72 / 第 49-72 行
~~~cpp
  49:      template <class T>
  50:      T arbitrary_small_value(const T& target)
  51:      {
  52:         using std::fabs;
  53:         return (tools::min_value<T>() / tools::epsilon<T>()) * (fabs(target) > 1 ? target : 1);
  54:      }
  55: 
  56: 
  57:      template <class T, class Policy>
  58:      struct hypergeometric_1F1_AS_13_3_7_tricomi_series
  59:      {
  60:         typedef T result_type;
  61: 
  62:         enum { cache_size = 64 };
  63: 
  64:         hypergeometric_1F1_AS_13_3_7_tricomi_series(const T& a, const T& b, const T& z, const Policy& pol_)
  65:            : A_minus_2(1), A_minus_1(0), A(b / 2), mult(z / 2), term(1), b_minus_1_plus_n(b - 1),
  66:             bessel_arg((b / 2 - a) * z),
  67:            two_a_minus_b(2 * a - b), pol(pol_), n(2)
  68:         {
  69:            BOOST_MATH_STD_USING
  70:            term /= pow(fabs(bessel_arg), b_minus_1_plus_n / 2);
  71:            mult /= sqrt(fabs(bessel_arg));
  72:            bessel_cache[cache_size - 1] = bessel_arg > 0 ? boost::math::cyl_bessel_j(b_minus_1_plus_n - 1, 2 * sqrt(bessel_arg), pol) : boost::math::cyl_bessel_i(b_minus_1_plus_n - 1, 2 * sqrt(-bessel_arg), pol);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as pow, sqrt, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 pow, sqrt, ...。

### Lines 73-96 / 第 73-96 行
~~~cpp
  73:            if (fabs(bessel_cache[cache_size - 1]) < tools::min_value<T>() / tools::epsilon<T>())
  74:            {
  75:               // We get very limited precision due to rapid denormalisation/underflow of the Bessel values, raise an exception and try something else:
  76:               policies::raise_evaluation_error("hypergeometric_1F1_AS_13_3_7_tricomi_series<%1%>", "Underflow in Bessel functions", bessel_cache[cache_size - 1], pol);
  77:               // Exceptions are off if we get here, just fill the cache with NaN's and we'll let this method fail and fallback later:
  78:               std::fill(bessel_cache.begin(), bessel_cache.end(), std::numeric_limits<T>::quiet_NaN());
  79:               cache_offset = -cache_size;
  80:               return;
  81:            }
  82:            if ((fabs(term * bessel_cache[cache_size - 1]) < tools::min_value<T>() / (tools::epsilon<T>() * tools::epsilon<T>())) || !(boost::math::isfinite)(term) || (!std::numeric_limits<T>::has_infinity && (fabs(term) > tools::max_value<T>())))
  83:            {
  84:               term = -log(fabs(bessel_arg)) * b_minus_1_plus_n / 2;
  85:               log_scale = lltrunc(term);
  86:               term -= log_scale;
  87:               term = exp(term);
  88:            }
  89:            else
  90:               log_scale = 0;
  91: #ifndef BOOST_MATH_NO_CXX17_IF_CONSTEXPR
  92:            if constexpr (std::numeric_limits<T>::has_infinity)
  93:            {
  94:               if (!(boost::math::isfinite)(bessel_cache[cache_size - 1]))
  95:               {
  96:                  policies::raise_evaluation_error("hypergeometric_1F1_AS_13_3_7_tricomi_series<%1%>", "Expected finite Bessel function result but got %1%", bessel_cache[cache_size - 1], pol);
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. This range declares or defines callable logic such as policies::raise_evaluation_error, std::fill, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围声明或定义了可调用逻辑，例如 policies::raise_evaluation_error, std::fill, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 97-120 / 第 97-120 行
~~~cpp
  97:                  // Exceptions are off if we get here, just fill the cache with NaN's and we'll let this method fail and fallback later:
  98:                  std::fill(bessel_cache.begin(), bessel_cache.end(), std::numeric_limits<T>::quiet_NaN());
  99:               }
 100:            }
 101:            else
 102:               if ((boost::math::isnan)(bessel_cache[cache_size - 1]) || (fabs(bessel_cache[cache_size - 1]) >= tools::max_value<T>()))
 103:               {
 104:                  policies::raise_evaluation_error("hypergeometric_1F1_AS_13_3_7_tricomi_series<%1%>", "Expected finite Bessel function result but got %1%", bessel_cache[cache_size - 1], pol);
 105:                  // Exceptions are off if we get here, just fill the cache with NaN's and we'll let this method fail and fallback later:
 106:                  std::fill(bessel_cache.begin(), bessel_cache.end(), std::numeric_limits<T>::quiet_NaN());
 107:               }
 108: #else
 109:            if ((std::numeric_limits<T>::has_infinity && !(boost::math::isfinite)(bessel_cache[cache_size - 1]))
 110:               || (!std::numeric_limits<T>::has_infinity && ((boost::math::isnan)(bessel_cache[cache_size - 1]) || (fabs(bessel_cache[cache_size - 1]) >= tools::max_value<T>()))))
 111:            {
 112:               policies::raise_evaluation_error("hypergeometric_1F1_AS_13_3_7_tricomi_series<%1%>", "Expected finite Bessel function result but got %1%", bessel_cache[cache_size - 1], pol);
 113:               // Exceptions are off if we get here, just fill the cache with NaN's and we'll let this method fail and fallback later:
 114:               std::fill(bessel_cache.begin(), bessel_cache.end(), std::numeric_limits<T>::quiet_NaN());
 115:            }
 116: #endif
 117:            cache_offset = -cache_size;
 118:            refill_cache();
 119:         }
 120:         T operator()()
~~~
- **EN:** This range declares or defines callable logic such as std::fill, policies::raise_evaluation_error, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::fill, policies::raise_evaluation_error, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 121-144 / 第 121-144 行
~~~cpp
 121:         {
 122:            //
 123:            // We return the n-2 term, and do 2 terms at once as every other term can be
 124:            // very small (or zero) when b == 2a:
 125:            //
 126:            BOOST_MATH_STD_USING
 127:            //
 128:            // Except in the multiprecision case, we have probably illiminated anything
 129:            // would need more than the default 64 Bessel Functions.  Anything more
 130:            // than that risks becoming a divergent series anyway...
 131:            //
 132:            if(n - 2 - cache_offset >= cache_size)
 133:               refill_cache();  // LCOV_EXCL_LINE
 134:            T result = A_minus_2 * term * bessel_cache[n - 2 - cache_offset];
 135:            term *= mult;
 136:            ++n;
 137:            T A_next = ((b_minus_1_plus_n + 2) * A_minus_1 + two_a_minus_b * A_minus_2) / n;
 138:            b_minus_1_plus_n += 1;
 139:            A_minus_2 = A_minus_1;
 140:            A_minus_1 = A;
 141:            A = A_next;
 142: 
 143:            if (A_minus_2 != 0)
 144:            {
~~~
- **EN:** This range declares or defines callable logic such as refill_cache. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 refill_cache。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 145-168 / 第 145-168 行
~~~cpp
 145:               if (n - 2 - cache_offset >= cache_size)
 146:                  refill_cache(); // LCOV_EXCL_LINE
 147:               result += A_minus_2 * term * bessel_cache[n - 2 - cache_offset];
 148:            }
 149:            term *= mult;
 150:            ++n;
 151:            A_next = ((b_minus_1_plus_n + 2) * A_minus_1 + two_a_minus_b * A_minus_2) / n;
 152:            b_minus_1_plus_n += 1;
 153:            A_minus_2 = A_minus_1;
 154:            A_minus_1 = A;
 155:            A = A_next;
 156: 
 157:            return result;
 158:         }
 159: 
 160:         long long scale()const
 161:         {
 162:            return log_scale;
 163:         }
 164: 
 165:      private:
 166:         T A_minus_2, A_minus_1, A, mult, term, b_minus_1_plus_n, bessel_arg, two_a_minus_b;
 167:         std::array<T, cache_size> bessel_cache;
 168:         const Policy& pol;
~~~
- **EN:** This range declares or defines callable logic such as refill_cache. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 refill_cache。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 169-192 / 第 169-192 行
~~~cpp
 169:         int n, cache_offset;
 170:         long long log_scale;
 171: 
 172:         hypergeometric_1F1_AS_13_3_7_tricomi_series operator=(const hypergeometric_1F1_AS_13_3_7_tricomi_series&) = delete;
 173: 
 174:         void refill_cache()
 175:         {
 176:            BOOST_MATH_STD_USING
 177:            //
 178:            // We don't calculate a new bessel I/J value: instead start our iterator off
 179:            // with an arbitrary small value, then when we get back to the last value in the previous cache
 180:            // calculate the ratio and use it to renormalise all the new values.  This is more efficient, but
 181:            // also avoids problems with J_v(x) or I_v(x) underflowing to zero.
 182:            //
 183:            cache_offset += cache_size;
 184:            T last_value = bessel_cache.back();
 185:            T ratio;
 186:            if (bessel_arg > 0)
 187:            {
 188:               //
 189:               // We will be calculating Bessel J.
 190:               // We need a different recurrence strategy for positive and negative orders:
 191:               //
 192:               if (b_minus_1_plus_n > 0)
~~~
- **EN:** This range declares or defines callable logic such as back. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 back。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 193-216 / 第 193-216 行
~~~cpp
 193:               {
 194:                  bessel_j_backwards_iterator<T, Policy> i(b_minus_1_plus_n + (int)cache_size - 1, 2 * sqrt(bessel_arg), arbitrary_small_value(last_value));
 195: 
 196:                  for (int j = cache_size - 1; j >= 0; --j, ++i)
 197:                  {
 198:                     bessel_cache[j] = *i;
 199:                     //
 200:                     // Depending on the value of bessel_arg, the values stored in the cache can grow so
 201:                     // large as to overflow, if that looks likely then we need to rescale all the
 202:                     // existing terms (most of which will then underflow to zero).  In this situation
 203:                     // it's likely that our series will only need 1 or 2 terms of the series but we
 204:                     // can't be sure of that:
 205:                     //
 206:                     if ((j < cache_size - 2) && (tools::max_value<T>() / fabs(64 * bessel_cache[j] / bessel_cache[j + 1]) < fabs(bessel_cache[j])))
 207:                     {
 208:                        T rescale = static_cast<T>(pow(fabs(bessel_cache[j] / bessel_cache[j + 1]), T(j + 1)) * 2);
 209:                        if (!((boost::math::isfinite)(rescale)))
 210:                           rescale = tools::max_value<T>();
 211:                        for (int k = j; k < cache_size; ++k)
 212:                           bessel_cache[k] /= rescale;
 213:                        bessel_j_backwards_iterator<T, Policy> ti(b_minus_1_plus_n + j, 2 * sqrt(bessel_arg), bessel_cache[j + 1], bessel_cache[j]);
 214:                        i = ti;
 215:                     }
 216:                  }
~~~
- **EN:** This range declares or defines callable logic such as i, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 i, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 217-240 / 第 217-240 行
~~~cpp
 217:                  ratio = last_value / *i;
 218:               }
 219:               else
 220:               {
 221:                  //
 222:                  // Negative order is difficult: the J_v(x) recurrence relations are unstable
 223:                  // *in both directions* for v < 0, except as v -> -INF when we have
 224:                  // J_-v(x)  ~= -sin(pi.v)Y_v(x).
 225:                  // For small v what we can do is compute every other Bessel function and
 226:                  // then fill in the gaps using the recurrence relation.  This *is* stable
 227:                  // provided that v is not so negative that the above approximation holds.
 228:                  //
 229:                  bessel_cache[1] = cyl_bessel_j(b_minus_1_plus_n + 1, 2 * sqrt(bessel_arg), pol);
 230:                  bessel_cache[0] = (last_value + bessel_cache[1]) / (b_minus_1_plus_n / sqrt(bessel_arg));
 231:                  int pos = 2;
 232:                  while ((pos < cache_size - 1) && (b_minus_1_plus_n + pos < 0))
 233:                  {
 234:                     bessel_cache[pos + 1] = cyl_bessel_j(b_minus_1_plus_n + pos + 1, 2 * sqrt(bessel_arg), pol);
 235:                     bessel_cache[pos] = (bessel_cache[pos-1] + bessel_cache[pos+1]) / ((b_minus_1_plus_n + pos) / sqrt(bessel_arg));
 236:                     pos += 2;
 237:                  }
 238:                  if (pos < cache_size)
 239:                  {
 240:                     //
~~~
- **EN:** This range declares or defines callable logic such as cyl_bessel_j, sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 cyl_bessel_j, sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 241-264 / 第 241-264 行
~~~cpp
 241:                     // We have crossed over into the region where backward recursion is the stable direction
 242:                     // start from arbitrary value and recurse down to "pos" and normalise:
 243:                     //
 244:                     bessel_j_backwards_iterator<T, Policy> i2(b_minus_1_plus_n + (int)cache_size - 1, 2 * sqrt(bessel_arg), arbitrary_small_value(bessel_cache[pos-1]));
 245:                     for (int loc = cache_size - 1; loc >= pos; --loc)
 246:                        bessel_cache[loc] = *i2++;
 247:                     ratio = bessel_cache[pos - 1] / *i2;
 248:                     //
 249:                     // Sanity check, if we normalised to an unusually small value then it was likely
 250:                     // to be near a root and the calculated ratio is garbage, if so perform one
 251:                     // more J_v(x) evaluation at position and normalise again:
 252:                     //
 253:                     if (fabs(bessel_cache[pos] * ratio / bessel_cache[pos - 1]) > 5)
 254:                        ratio = cyl_bessel_j(b_minus_1_plus_n + pos, 2 * sqrt(bessel_arg), pol) / bessel_cache[pos];
 255:                     while (pos < cache_size)
 256:                        bessel_cache[pos++] *= ratio;
 257:                  }
 258:                  ratio = 1;
 259:               }
 260:            }
 261:            else
 262:            {
 263:               //
 264:               // Bessel I.
~~~
- **EN:** This range declares or defines callable logic such as i2. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 i2。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 265-288 / 第 265-288 行
~~~cpp
 265:               // We need a different recurrence strategy for positive and negative orders:
 266:               //
 267:               if (b_minus_1_plus_n > 0)
 268:               {
 269:                  bessel_i_backwards_iterator<T, Policy> i(b_minus_1_plus_n + (int)cache_size - 1, 2 * sqrt(-bessel_arg), arbitrary_small_value(last_value));
 270: 
 271:                  for (int j = cache_size - 1; j >= 0; --j, ++i)
 272:                  {
 273:                     bessel_cache[j] = *i;
 274:                     //
 275:                     // Depending on the value of bessel_arg, the values stored in the cache can grow so
 276:                     // large as to overflow, if that looks likely then we need to rescale all the
 277:                     // existing terms (most of which will then underflow to zero).  In this situation
 278:                     // it's likely that our series will only need 1 or 2 terms of the series but we
 279:                     // can't be sure of that:
 280:                     //
 281:                     if ((j < cache_size - 2) && (tools::max_value<T>() / fabs(64 * bessel_cache[j] / bessel_cache[j + 1]) < fabs(bessel_cache[j])))
 282:                     {
 283:                        T rescale = static_cast<T>(pow(fabs(bessel_cache[j] / bessel_cache[j + 1]), T(j + 1)) * 2);
 284:                        if (!((boost::math::isfinite)(rescale)))
 285:                           rescale = tools::max_value<T>();
 286:                        for (int k = j; k < cache_size; ++k)
 287:                           bessel_cache[k] /= rescale;
 288:                        i = bessel_i_backwards_iterator<T, Policy>(b_minus_1_plus_n + j, 2 * sqrt(-bessel_arg), bessel_cache[j + 1], bessel_cache[j]);
~~~
- **EN:** This range declares or defines callable logic such as i, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 i, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 289-312 / 第 289-312 行
~~~cpp
 289:                     }
 290:                  }
 291:                  ratio = last_value / *i;
 292:               }
 293:               else
 294:               {
 295:                  //
 296:                  // Forwards iteration is stable:
 297:                  //
 298:                  bessel_i_forwards_iterator<T, Policy> i(b_minus_1_plus_n, 2 * sqrt(-bessel_arg));
 299:                  int pos = 0;
 300:                  while ((pos < cache_size) && (b_minus_1_plus_n + pos < 0.5))
 301:                  {
 302:                     bessel_cache[pos++] = *i++;
 303:                  }
 304:                  if (pos < cache_size)
 305:                  {
 306:                     //
 307:                     // We have crossed over into the region where backward recursion is the stable direction
 308:                     // start from arbitrary value and recurse down to "pos" and normalise:
 309:                     //
 310:                     bessel_i_backwards_iterator<T, Policy> i2(b_minus_1_plus_n + (int)cache_size - 1, 2 * sqrt(-bessel_arg), arbitrary_small_value(last_value));
 311:                     for (int loc = cache_size - 1; loc >= pos; --loc)
 312:                        bessel_cache[loc] = *i2++;
~~~
- **EN:** This range declares or defines callable logic such as i, i2. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 i, i2。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 313-336 / 第 313-336 行
~~~cpp
 313:                     ratio = bessel_cache[pos - 1] / *i2;
 314:                     while (pos < cache_size)
 315:                        bessel_cache[pos++] *= ratio;
 316:                  }
 317:                  ratio = 1;
 318:               }
 319:            }
 320:            if(ratio != 1)
 321:               for (auto j = bessel_cache.begin(); j != bessel_cache.end(); ++j)
 322:                  *j *= ratio;
 323:            //
 324:            // Very occasionally our normalisation fails because the normalisztion value
 325:            // is sitting right on top of a root (or very close to it).  When that happens
 326:            // best to calculate a fresh Bessel evaluation and normalise again.
 327:            //
 328:            if (fabs(bessel_cache[0] / last_value) > 5)
 329:            {
 330:               ratio = (bessel_arg < 0 ? cyl_bessel_i(b_minus_1_plus_n, 2 * sqrt(-bessel_arg), pol) : cyl_bessel_j(b_minus_1_plus_n, 2 * sqrt(bessel_arg), pol)) / bessel_cache[0];
 331:               if (ratio != 1)
 332:                  for (auto j = bessel_cache.begin(); j != bessel_cache.end(); ++j)
 333:                     *j *= ratio;
 334:            }
 335:         }
 336:      };
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 337-360 / 第 337-360 行
~~~cpp
 337: 
 338:      template <class T, class Policy>
 339:      T hypergeometric_1F1_AS_13_3_7_tricomi(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scale)
 340:      {
 341:         BOOST_MATH_STD_USING
 342:         //
 343:         // Works for a < 0, b < 0, z > 0.
 344:         //
 345:         // For convergence we require A * term to be converging otherwise we get
 346:         // a divergent alternating series.  It's actually really hard to analyse this
 347:         // and the best purely heuristic policy we've found is
 348:         // z < fabs((2 * a - b) / (sqrt(fabs(a)))) ; b > 0  or:
 349:         // z < fabs((2 * a - b) / (sqrt(fabs(ab)))) ; b < 0
 350:         //
 351:         T prefix(0);
 352:         int prefix_sgn(0);
 353:         bool use_logs = false;
 354:         long long scale = 0;
 355:         //
 356:         // We can actually support the b == 2a case within here, but we haven't
 357:         // as we appear never to get here in practice.  Which means this get out
 358:         // clause is a bit of defensive programming....
 359:         //
 360:         if(b == 2 * a)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as fabs, prefix, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 fabs, prefix, ...。

### Lines 361-384 / 第 361-384 行
~~~cpp
 361:            return hypergeometric_1F1_divergent_fallback(a, b, z, pol, log_scale);
 362: 
 363: #ifndef BOOST_MATH_NO_EXCEPTIONS
 364:         try
 365: #endif
 366:         {
 367:            prefix = boost::math::tgamma(b, pol);
 368:            prefix *= exp(z / 2);
 369:         }
 370: #ifndef BOOST_MATH_NO_EXCEPTIONS
 371:         catch (const std::runtime_error&)
 372:         {
 373:            use_logs = true;
 374:         }
 375: #endif
 376:         if (use_logs || (prefix == 0) || !(boost::math::isfinite)(prefix) || (!std::numeric_limits<T>::has_infinity && (fabs(prefix) >= tools::max_value<T>())))
 377:         {
 378:            use_logs = true;
 379:            prefix = boost::math::lgamma(b, &prefix_sgn, pol) + z / 2;
 380:            scale = lltrunc(prefix);
 381:            log_scale += scale;
 382:            prefix -= scale;
 383:         }
 384:         T result(0);
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. This range declares or defines callable logic such as boost::math::tgamma, exp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围声明或定义了可调用逻辑，例如 boost::math::tgamma, exp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 385-408 / 第 385-408 行
~~~cpp
 385:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 386:         bool retry = false;
 387:         long long series_scale = 0;
 388: #ifndef BOOST_MATH_NO_EXCEPTIONS
 389:         try
 390: #endif
 391:         {
 392:            hypergeometric_1F1_AS_13_3_7_tricomi_series<T, Policy> s(a, b, z, pol);
 393:            series_scale = s.scale();
 394:            log_scale += s.scale();
 395: #ifndef BOOST_MATH_NO_EXCEPTIONS
 396:            try
 397: #endif
 398:            {
 399:               T norm = 0;
 400:               result = 0;
 401:               if((a < 0) && (b < 0))
 402:                  result = boost::math::tools::checked_sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter, result, norm);
 403:               else
 404:                  result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter, result);
 405:               if (!(boost::math::isfinite)(result) || (result == 0) || (!std::numeric_limits<T>::has_infinity && (fabs(result) >= tools::max_value<T>())))
 406:                  retry = true;
 407:               if (norm / fabs(result) > 1 / boost::math::tools::root_epsilon<T>())
 408:                  retry = true;  // fatal cancellation
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. This range declares or defines callable logic such as s, scale, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围声明或定义了可调用逻辑，例如 s, scale, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 409-432 / 第 409-432 行
~~~cpp
 409:            }
 410: #ifndef BOOST_MATH_NO_EXCEPTIONS
 411:            catch (const std::overflow_error&)
 412:            {
 413:               retry = true;
 414:            }
 415:            catch (const boost::math::evaluation_error&)
 416:            {
 417:               retry = true;
 418:            }
 419: #endif
 420:         }
 421: #ifndef BOOST_MATH_NO_EXCEPTIONS
 422:         catch (const std::overflow_error&)
 423:         {
 424:            log_scale -= scale;
 425:            return hypergeometric_1F1_divergent_fallback(a, b, z, pol, log_scale);
 426:         }
 427:         catch (const boost::math::evaluation_error&)
 428:         {
 429:            log_scale -= scale;
 430:            return hypergeometric_1F1_divergent_fallback(a, b, z, pol, log_scale);
 431:         }
 432: #endif
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 433-456 / 第 433-456 行
~~~cpp
 433:         if (retry)
 434:         {
 435:            log_scale -= scale;
 436:            log_scale -= series_scale;
 437:            return hypergeometric_1F1_divergent_fallback(a, b, z, pol, log_scale);
 438:         }
 439:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_AS_13_3_7<%1%>(%1%,%1%,%1%)", max_iter, pol);
 440:         if (use_logs)
 441:         {
 442:            int sgn = boost::math::sign(result);
 443:            prefix += log(fabs(result));
 444:            result = sgn * prefix_sgn * exp(prefix);
 445:         }
 446:         else
 447:         {
 448:            if ((fabs(result) > 1) && (fabs(prefix) > 1) && (tools::max_value<T>() / fabs(result) < fabs(prefix)))
 449:            {
 450:               // Overflow:
 451:               scale = lltrunc(tools::log_max_value<T>()) - 10;
 452:               log_scale += scale;
 453:               result /= exp(T(scale));
 454:            }
 455:            result *= prefix;
 456:         }
~~~
- **EN:** This range declares or defines callable logic such as boost::math::sign, log, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::sign, log, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 457-480 / 第 457-480 行
~~~cpp
 457:         return result;
 458:      }
 459: 
 460: 
 461:      template <class T>
 462:      struct cyl_bessel_i_large_x_sum
 463:      {
 464:         typedef T result_type;
 465: 
 466:         cyl_bessel_i_large_x_sum(const T& v, const T& x) : v(v), z(x), term(1), k(0) {}
 467: 
 468:         T operator()()
 469:         {
 470:            T result = term;
 471:            ++k;
 472:            term *= -(4 * v * v - (2 * k - 1) * (2 * k - 1)) / (8 * k * z);
 473:            return result;
 474:         }
 475:         T v, z, term;
 476:         int k;
 477:      };
 478: 
 479:      template <class T, class Policy>
 480:      T cyl_bessel_i_large_x_scaled(const T& v, const T& x, long long& log_scaling, const Policy& pol)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as cyl_bessel_i_large_x_sum.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 cyl_bessel_i_large_x_sum。

### Lines 481-504 / 第 481-504 行
~~~cpp
 481:      {
 482:         BOOST_MATH_STD_USING
 483:            cyl_bessel_i_large_x_sum<T> s(v, x);
 484:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 485:         T result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 486:         boost::math::policies::check_series_iterations<T>("boost::math::cyl_bessel_i_large_x<%1%>(%1%,%1%)", max_iter, pol);
 487:         long long scale = boost::math::lltrunc(x);
 488:         log_scaling += scale;
 489:         return result * exp(x - scale) / sqrt(boost::math::constants::two_pi<T>() * x);
 490:      }
 491: 
 492: 
 493: 
 494:      template <class T, class Policy>
 495:      struct hypergeometric_1F1_AS_13_3_6_series
 496:      {
 497:         typedef T result_type;
 498: 
 499:         enum { cache_size = 64 };
 500:         //
 501:         // This series is only convergent/useful for a and b approximately equal
 502:         // (ideally |a-b| < 1).  The series can also go divergent after a while
 503:         // when b < 0, which limits precision to around that of double.  In that
 504:         // situation we return 0 to terminate the series as otherwise the divergent
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as s, boost::math::tools::sum_series, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 s, boost::math::tools::sum_series, ...。

### Lines 505-528 / 第 505-528 行
~~~cpp
 505:         // terms will destroy all the bits in our result before they do eventually
 506:         // converge again.  One important use case for this series is for z < 0
 507:         // and |a| << |b| so that either b-a == b or at least most of the digits in a
 508:         // are lost in the subtraction.  Note that while you can easily convince yourself
 509:         // that the result should be unity when b-a == b, in fact this is not (quite)
 510:         // the case for large z.
 511:         //
 512:         hypergeometric_1F1_AS_13_3_6_series(const T& a, const T& b, const T& z, const T& b_minus_a, const Policy& pol_)
 513:            : b_minus_a(b_minus_a), half_z(z / 2), poch_1(2 * b_minus_a - 1), poch_2(b_minus_a - a), b_poch(b), term(1), last_result(1), sign(1), n(0), cache_offset(-cache_size), scale(0), pol(pol_)
 514:         {
 515:            bessel_i_cache[cache_size - 1] = half_z > tools::log_max_value<T>() ?
 516:               cyl_bessel_i_large_x_scaled(T(b_minus_a - 1.5f), half_z, scale, pol) : boost::math::cyl_bessel_i(b_minus_a - 1.5f, half_z, pol);
 517:            refill_cache();
 518:         }
 519:         T operator()()
 520:         {
 521:            BOOST_MATH_STD_USING
 522:            if(n - cache_offset >= cache_size)
 523:               refill_cache();
 524: 
 525:            T result = term * (b_minus_a - 0.5f + n) * sign * bessel_i_cache[n - cache_offset];
 526:            ++n;
 527:            term *= poch_1;
 528:            poch_1 = (n == 1) ? T(2 * b_minus_a) : T(poch_1 + 1);
~~~
- **EN:** This range declares or defines callable logic such as cyl_bessel_i_large_x_scaled, refill_cache, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 cyl_bessel_i_large_x_scaled, refill_cache, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 529-552 / 第 529-552 行
~~~cpp
 529:            term *= poch_2;
 530:            poch_2 += 1;
 531:            term /= n;
 532:            term /= b_poch;
 533:            b_poch += 1;
 534:            sign = -sign;
 535: 
 536:            if ((fabs(result) > fabs(last_result)) && (n > 100))
 537:               return 0;  // series has gone divergent!
 538: 
 539:            last_result = result;
 540: 
 541:            return result;
 542:         }
 543: 
 544:         long long scaling()const
 545:         {
 546:            return scale;
 547:         }
 548: 
 549:      private:
 550:         T b_minus_a, half_z, poch_1, poch_2, b_poch, term, last_result;
 551:         int sign;
 552:         int n, cache_offset;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 553-576 / 第 553-576 行
~~~cpp
 553:         long long scale;
 554:         const Policy& pol;
 555:         std::array<T, cache_size> bessel_i_cache;
 556: 
 557:         void refill_cache()
 558:         {
 559:            BOOST_MATH_STD_USING
 560:            //
 561:            // We don't calculate a new bessel I value: instead start our iterator off
 562:            // with an arbitrary small value, then when we get back to the last value in the previous cache
 563:            // calculate the ratio and use it to renormalise all the values.  This is more efficient, but
 564:            // also avoids problems with I_v(x) underflowing to zero.
 565:            //
 566:            cache_offset += cache_size;
 567:            T last_value = bessel_i_cache.back();
 568:            bessel_i_backwards_iterator<T, Policy> i(b_minus_a + cache_offset + (int)cache_size - 1.5f, half_z, tools::min_value<T>() * (fabs(last_value) > 1 ? last_value : 1) / tools::epsilon<T>());
 569: 
 570:            for (int j = cache_size - 1; j >= 0; --j, ++i)
 571:            {
 572:               bessel_i_cache[j] = *i;
 573:               //
 574:               // Depending on the value of half_z, the values stored in the cache can grow so
 575:               // large as to overflow, if that looks likely then we need to rescale all the
 576:               // existing terms (most of which will then underflow to zero).  In this situation
~~~
- **EN:** This range declares or defines callable logic such as back, i. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 back, i。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 577-600 / 第 577-600 行
~~~cpp
 577:               // it's likely that our series will only need 1 or 2 terms of the series but we
 578:               // can't be sure of that:
 579:               //
 580:               if((j < cache_size - 2) && (bessel_i_cache[j + 1] != 0) && (tools::max_value<T>() / fabs(64 * bessel_i_cache[j] / bessel_i_cache[j + 1]) < fabs(bessel_i_cache[j])))
 581:               {
 582:                  T rescale = static_cast<T>(pow(fabs(bessel_i_cache[j] / bessel_i_cache[j + 1]), T(j + 1)) * 2);
 583:                  if (rescale > tools::max_value<T>())
 584:                     rescale = tools::max_value<T>();
 585:                  for (int k = j; k < cache_size; ++k)
 586:                     bessel_i_cache[k] /= rescale;
 587:                  i = bessel_i_backwards_iterator<T, Policy>(b_minus_a -0.5f + cache_offset + j, half_z, bessel_i_cache[j + 1], bessel_i_cache[j]);
 588:               }
 589:            }
 590:            T ratio = last_value / *i;
 591:            for (auto j = bessel_i_cache.begin(); j != bessel_i_cache.end(); ++j)
 592:               *j *= ratio;
 593:         }
 594: 
 595:         hypergeometric_1F1_AS_13_3_6_series() = delete;
 596:         hypergeometric_1F1_AS_13_3_6_series(const hypergeometric_1F1_AS_13_3_6_series&) = delete;
 597:         hypergeometric_1F1_AS_13_3_6_series& operator=(const hypergeometric_1F1_AS_13_3_6_series&) = delete;
 598:      };
 599: 
 600:      template <class T, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as pow.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 pow。

### Lines 601-624 / 第 601-624 行
~~~cpp
 601:      T hypergeometric_1F1_AS_13_3_6(const T& a, const T& b, const T& z, const T& b_minus_a, const Policy& pol, long long& log_scaling)
 602:      {
 603:         BOOST_MATH_STD_USING
 604:         if(b_minus_a == 0)
 605:         {
 606:            // special case: M(a,a,z) == exp(z)
 607:            long long scale = lltrunc(z, pol);
 608:            log_scaling += scale;
 609:            return exp(z - scale);
 610:         }
 611:         hypergeometric_1F1_AS_13_3_6_series<T, Policy> s(a, b, z, b_minus_a, pol);
 612:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 613:         T result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 614:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_AS_13_3_6<%1%>(%1%,%1%,%1%)", max_iter, pol);
 615:         result *= boost::math::tgamma(b_minus_a - 0.5f, pol) * pow(z / 4, -b_minus_a + T(0.5f));
 616:         long long scale = lltrunc(z / 2);
 617:         log_scaling += scale;
 618:         log_scaling += s.scaling();
 619:         result *= exp(z / 2 - scale);
 620:         return result;
 621:      }
 622: 
 623:      /****************************************************************************************************************/
 624:      //
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, s, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, s, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 625-648 / 第 625-648 行
~~~cpp
 625:      // The following are not used at present and are commented out for that reason:
 626:      //
 627:      /****************************************************************************************************************/
 628: 
 629: #if 0
 630: 
 631:      template <class T, class Policy>
 632:      struct hypergeometric_1F1_AS_13_3_8_series
 633:      {
 634:         //
 635:         // TODO: store and cache Bessel function evaluations via backwards recurrence.
 636:         //
 637:         // The C term grows by at least an order of magnitude with each iteration, and
 638:         // rate of growth is largely independent of the arguments.  Free parameter h
 639:         // seems to give accurate results for small values (almost zero) or h=1.
 640:         // Convergence and accuracy, only when -a/z > 100, this appears to have no
 641:         // or little benefit over 13.3.7 as it generally requires more iterations?
 642:         //
 643:         hypergeometric_1F1_AS_13_3_8_series(const T& a, const T& b, const T& z, const T& h, const Policy& pol_)
 644:            : C_minus_2(1), C_minus_1(-b * h), C(b * (b + 1) * h * h / 2 - (2 * h - 1) * a / 2),
 645:            bessel_arg(2 * sqrt(-a * z)), bessel_order(b - 1), power_term(std::pow(-a * z, (1 - b) / 2)), mult(z / std::sqrt(-a * z)),
 646:            a_(a), b_(b), z_(z), h_(h), n(2), pol(pol_)
 647:         {
 648:         }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 649-672 / 第 649-672 行
~~~cpp
 649:         T operator()()
 650:         {
 651:            // we actually return the n-2 term:
 652:            T result = C_minus_2 * power_term * boost::math::cyl_bessel_j(bessel_order, bessel_arg, pol);
 653:            bessel_order += 1;
 654:            power_term *= mult;
 655:            ++n;
 656:            T C_next = ((1 - 2 * h_) * (n - 1) - b_ * h_) * C
 657:               + ((1 - 2 * h_) * a_ - h_ * (h_ - 1) *(b_ + n - 2)) * C_minus_1
 658:               - h_ * (h_ - 1) * a_ * C_minus_2;
 659:            C_next /= n;
 660:            C_minus_2 = C_minus_1;
 661:            C_minus_1 = C;
 662:            C = C_next;
 663:            return result;
 664:         }
 665:         T C, C_minus_1, C_minus_2, bessel_arg, bessel_order, power_term, mult, a_, b_, z_, h_;
 666:         const Policy& pol;
 667:         int n;
 668: 
 669:         typedef T result_type;
 670:      };
 671: 
 672:      template <class T, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::cyl_bessel_j.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::cyl_bessel_j。

### Lines 673-696 / 第 673-696 行
~~~cpp
 673:      T hypergeometric_1F1_AS_13_3_8(const T& a, const T& b, const T& z, const T& h, const Policy& pol)
 674:      {
 675:         BOOST_MATH_STD_USING
 676:         T prefix = exp(h * z) * boost::math::tgamma(b);
 677:         hypergeometric_1F1_AS_13_3_8_series<T, Policy> s(a, b, z, h, pol);
 678:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 679:         T result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 680:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_AS_13_3_8<%1%>(%1%,%1%,%1%)", max_iter, pol);
 681:         result *= prefix;
 682:         return result;
 683:      }
 684: 
 685:      //
 686:      // This is the series from https://dlmf.nist.gov/13.11
 687:      // It appears to be unusable for a,z < 0, and for
 688:      // b < 0 appears to never be better than the defining series
 689:      // for 1F1.
 690:      //
 691:      template <class T, class Policy>
 692:      struct hypergeometric_1f1_13_11_1_series
 693:      {
 694:         typedef T result_type;
 695: 
 696:         hypergeometric_1f1_13_11_1_series(const T& a, const T& b, const T& z, const Policy& pol_)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as exp, s, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 exp, s, ...。

### Lines 697-720 / 第 697-720 行
~~~cpp
 697:            : term(1), two_a_minus_1_plus_s(2 * a - 1), two_a_minus_b_plus_s(2 * a - b), b_plus_s(b), a_minus_half_plus_s(a - 0.5f), half_z(z / 2), s(0), pol(pol_)
 698:         {
 699:         }
 700:         T operator()()
 701:         {
 702:            T result = term * a_minus_half_plus_s * boost::math::cyl_bessel_i(a_minus_half_plus_s, half_z, pol);
 703: 
 704:            term *= two_a_minus_1_plus_s * two_a_minus_b_plus_s / (b_plus_s * ++s);
 705:            two_a_minus_1_plus_s += 1;
 706:            a_minus_half_plus_s += 1;
 707:            two_a_minus_b_plus_s += 1;
 708:            b_plus_s += 1;
 709: 
 710:            return result;
 711:         }
 712:         T term, two_a_minus_1_plus_s, two_a_minus_b_plus_s, b_plus_s, a_minus_half_plus_s, half_z;
 713:         long long s;
 714:         const Policy& pol;
 715:      };
 716: 
 717:      template <class T, class Policy>
 718:      T hypergeometric_1f1_13_11_1(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling)
 719:      {
 720:         BOOST_MATH_STD_USING
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::cyl_bessel_i.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::cyl_bessel_i。

### Lines 721-744 / 第 721-744 行
~~~cpp
 721:            bool use_logs = false;
 722:         T prefix;
 723:         int prefix_sgn = 1;
 724:         if (true/*(a < boost::math::max_factorial<T>::value) && (a > 0)*/)
 725:            prefix = boost::math::tgamma(a - 0.5f, pol);
 726:         else
 727:         {
 728:            prefix = boost::math::lgamma(a - 0.5f, &prefix_sgn, pol);
 729:            use_logs = true;
 730:         }
 731:         if (use_logs)
 732:         {
 733:            prefix += z / 2;
 734:            prefix += log(z / 4) * (0.5f - a);
 735:         }
 736:         else if (z > 0)
 737:         {
 738:            prefix *= pow(z / 4, 0.5f - a);
 739:            prefix *= exp(z / 2);
 740:         }
 741:         else
 742:         {
 743:            prefix *= exp(z / 2);
 744:            prefix *= pow(z / 4, 0.5f - a);
~~~
- **EN:** This range declares or defines callable logic such as boost::math::tgamma, boost::math::lgamma, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::tgamma, boost::math::lgamma, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 745-768 / 第 745-768 行
~~~cpp
 745:         }
 746: 
 747:         hypergeometric_1f1_13_11_1_series<T, Policy> s(a, b, z, pol);
 748:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 749:         T result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 750:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1f1_13_11_1<%1%>(%1%,%1%,%1%)", max_iter, pol);
 751:         if (use_logs)
 752:         {
 753:            long long scaling = lltrunc(prefix);
 754:            log_scaling += scaling;
 755:            prefix -= scaling;
 756:            result *= exp(prefix) * prefix_sgn;
 757:         }
 758:         else
 759:            result *= prefix;
 760: 
 761:         return result;
 762:      }
 763: 
 764: #endif
 765: 
 766:   } } } // namespaces
 767: 
 768: #endif // BOOST_MATH_HYPERGEOMETRIC_1F1_BESSEL_HPP
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. This range declares or defines callable logic such as s, boost::math::tools::sum_series, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 此范围声明或定义了可调用逻辑，例如 s, boost::math::tools::sum_series, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

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
- **Included headers / 包含头文件**: `boost/math/tools/series.hpp, boost/math/special_functions/bessel.hpp, boost/math/special_functions/laguerre.hpp, boost/math/special_functions/detail/hypergeometric_pFq_checked_series.hpp, boost/math/special_functions/bessel_iterators.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `hypergeometric_1F1_divergent_fallback, sqrt, pow, boost::math::cyl_bessel_j, policies::raise_evaluation_error, std::fill, lltrunc, exp, ...`
