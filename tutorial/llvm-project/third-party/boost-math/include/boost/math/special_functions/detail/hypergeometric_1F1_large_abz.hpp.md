# hypergeometric_1F1_large_abz.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/hypergeometric_1F1_large_abz.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the hypergeometric 1F1 large abz special-function path.
- **作用（中文）**: 此头文件为 hypergeometric 1F1 large abz 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: 
   2: ///////////////////////////////////////////////////////////////////////////////
   3: //  Copyright 2018 John Maddock
   4: //  Distributed under the Boost
   5: //  Software License, Version 1.0. (See accompanying file
   6: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   7: 
   8: #ifndef BOOST_HYPERGEOMETRIC_1F1_LARGE_ABZ_HPP_
   9: #define BOOST_HYPERGEOMETRIC_1F1_LARGE_ABZ_HPP_
  10: 
  11: #include <boost/math/special_functions/detail/hypergeometric_1F1_bessel.hpp>
  12: #include <boost/math/special_functions/detail/hypergeometric_series.hpp>
  13: #include <boost/math/special_functions/gamma.hpp>
  14: #include <boost/math/special_functions/trunc.hpp>
  15: 
  16:   namespace boost { namespace math { namespace detail {
  17: 
  18:      template <class T>
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/detail/hypergeometric_1F1_bessel.hpp, boost/math/special_functions/detail/hypergeometric_series.hpp, boost/math/special_functions/gamma.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/special_functions/detail/hypergeometric_1F1_bessel.hpp, boost/math/special_functions/detail/hypergeometric_series.hpp, boost/math/special_functions/gamma.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19:      inline bool is_negative_integer(const T& x)
  20:      {
  21:         using std::floor;
  22:         return (x <= 0) && (floor(x) == x);
  23:      }
  24: 
  25: 
  26:      template <class T, class Policy>
  27:      struct hypergeometric_1F1_igamma_series
  28:      {
  29:         enum{ cache_size = 64 };
  30: 
  31:         typedef T result_type;
  32:         hypergeometric_1F1_igamma_series(const T& alpha, const T& delta, const T& x, const Policy& pol)
  33:            : delta_poch(-delta), alpha_poch(alpha), x(x), k(0), cache_offset(0), pol(pol)
  34:         {
  35:            BOOST_MATH_STD_USING
  36:            T log_term = log(x) * -alpha;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:            log_scaling = lltrunc(log_term - 3 - boost::math::tools::log_min_value<T>() / 50);
  38:            term = exp(log_term - log_scaling);
  39:            refill_cache();
  40:         }
  41:         T operator()()
  42:         {
  43:            if (k - cache_offset >= cache_size)
  44:            {
  45:               cache_offset += cache_size;
  46:               refill_cache();
  47:            }
  48:            T result = term * gamma_cache[k - cache_offset];
  49:            term *= delta_poch * alpha_poch / (++k * x);
  50:            delta_poch += 1;
  51:            alpha_poch += 1;
  52:            return result;
  53:         }
  54:         void refill_cache()
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, exp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, exp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:         {
  56:            typedef typename lanczos::lanczos<T, Policy>::type lanczos_type;
  57: 
  58:            gamma_cache[cache_size - 1] = boost::math::gamma_p(alpha_poch + ((int)cache_size - 1), x, pol);
  59:            for (int i = cache_size - 1; i > 0; --i)
  60:            {
  61:               gamma_cache[i - 1] = gamma_cache[i] >= 1 ? T(1) : T(gamma_cache[i] + regularised_gamma_prefix(T(alpha_poch + (i - 1)), x, pol, lanczos_type()) / (alpha_poch + (i - 1)));
  62:            }
  63:         }
  64:         T delta_poch, alpha_poch, x, term;
  65:         T gamma_cache[cache_size];
  66:         int k;
  67:         long long log_scaling;
  68:         int cache_offset;
  69:         Policy pol;
  70:      };
  71: 
  72:      template <class T, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::gamma_p, T.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::gamma_p, T。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:      T hypergeometric_1F1_igamma(const T& a, const T& b, const T& x, const T& b_minus_a, const Policy& pol, long long& log_scaling)
  74:      {
  75:         BOOST_MATH_STD_USING
  76:         if (b_minus_a == 0)
  77:         {
  78:            // special case: M(a,a,z) == exp(z)
  79:            long long scale = lltrunc(x, pol);
  80:            log_scaling += scale;
  81:            return exp(x - scale);
  82:         }
  83:         hypergeometric_1F1_igamma_series<T, Policy> s(b_minus_a, a - 1, x, pol);
  84:         log_scaling += s.log_scaling;
  85:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
  86:         T result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
  87:         boost::math::policies::check_series_iterations<T>("boost::math::tgamma<%1%>(%1%,%1%)", max_iter, pol);
  88:         T log_prefix = x + boost::math::lgamma(b, pol) - boost::math::lgamma(a, pol);
  89:         long long scale = lltrunc(log_prefix);
  90:         log_scaling += scale;
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, s, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, s, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:         return result * exp(log_prefix - scale);
  92:      }
  93: 
  94:      template <class T, class Policy>
  95:      T hypergeometric_1F1_shift_on_a(T h, const T& a_local, const T& b_local, const T& x, int a_shift, const Policy& pol, long long& log_scaling)
  96:      {
  97:         BOOST_MATH_STD_USING
  98:         T a = a_local + a_shift;
  99:         if (a_shift == 0)
 100:            return h;
 101:         else if (a_shift > 0)
 102:         {
 103:            //
 104:            // Forward recursion on a is stable as long as 2a-b+z > 0.
 105:            // If 2a-b+z < 0 then backwards recursion is stable even though
 106:            // the function may be strictly increasing with a.  Potentially
 107:            // we may need to split the recurrence in 2 sections - one using
 108:            // forward recursion, and one backwards.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:            //
 110:            // We will get the next seed value from the ratio
 111:            // on b as that's stable and quick to compute.
 112:            //
 113: 
 114:            T crossover_a = (b_local - x) / 2;
 115:            int crossover_shift = itrunc(crossover_a - a_local);
 116: 
 117:            if (crossover_shift > 1)
 118:            {
 119:               //
 120:               // Forwards recursion will start off unstable, but may switch to the stable direction later.
 121:               // Start in the middle and go in both directions:
 122:               //
 123:               if (crossover_shift > a_shift)
 124:                  crossover_shift = a_shift;
 125:               crossover_a = a_local + crossover_shift;
 126:               boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> b_coef(crossover_a, b_local, x);
~~~
- **EN:** This range declares or defines callable logic such as itrunc, b_coef. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc, b_coef。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:               std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 128:               T b_ratio = boost::math::tools::function_ratio_from_backwards_recurrence(b_coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 129:               boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_large_abz<%1%>(%1%,%1%,%1%)", max_iter, pol);
 130:               //
 131:               // Convert to a ratio:
 132:               //         (1+a-b)M(a, b, z) - aM(a+1, b, z) + (b-1)M(a, b-1, z) = 0
 133:               //
 134:               //  hence: M(a+1,b,z) = ((1+a-b) / a) M(a,b,z) + ((b-1) / a) M(a,b,z)/b_ratio
 135:               //
 136:               T first = 1;
 137:               T second = ((1 + crossover_a - b_local) / crossover_a) + ((b_local - 1) / crossover_a) / b_ratio;
 138:               //
 139:               // Recurse down to a_local, compare values and re-normalise first and second:
 140:               //
 141:               boost::math::detail::hypergeometric_1F1_recurrence_a_coefficients<T> a_coef(crossover_a, b_local, x);
 142:               long long backwards_scale = 0;
 143:               T comparitor = boost::math::tools::apply_recurrence_relation_backward(a_coef, crossover_shift, second, first, &backwards_scale);
 144:               log_scaling -= backwards_scale;
~~~
- **EN:** This range declares or defines callable logic such as boost::math::tools::function_ratio_from_backwards_recurrence, a_coef, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::tools::function_ratio_from_backwards_recurrence, a_coef, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:               if ((h < 1) && (tools::max_value<T>() * h > comparitor))
 146:               {
 147:                  // Need to rescale!
 148:                  long long scale = lltrunc(log(h), pol) + 1;
 149:                  h *= exp(T(-scale));
 150:                  log_scaling += scale;
 151:               }
 152:               comparitor /= h;
 153:               first /= comparitor;
 154:               second /= comparitor;
 155:               //
 156:               // Now we can recurse forwards for the rest of the range:
 157:               //
 158:               if (crossover_shift < a_shift)
 159:               {
 160:                  boost::math::detail::hypergeometric_1F1_recurrence_a_coefficients<T> a_coef_2(crossover_a + 1, b_local, x);
 161:                  h = boost::math::tools::apply_recurrence_relation_forward(a_coef_2, a_shift - crossover_shift - 1, first, second, &log_scaling);
 162:               }
~~~
- **EN:** This range declares or defines callable logic such as exp, a_coef_2, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp, a_coef_2, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:               else
 164:                  h = first;
 165:            }
 166:            else
 167:            {
 168:               //
 169:               // Regular case where forwards iteration is stable right from the start:
 170:               //
 171:               boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> b_coef(a_local, b_local, x);
 172:               std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 173:               T b_ratio = boost::math::tools::function_ratio_from_backwards_recurrence(b_coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 174:               boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_large_abz<%1%>(%1%,%1%,%1%)", max_iter, pol);
 175:               //
 176:               // Convert to a ratio:
 177:               //         (1+a-b)M(a, b, z) - aM(a+1, b, z) + (b-1)M(a, b-1, z) = 0
 178:               //
 179:               //  hence: M(a+1,b,z) = ((1+a-b) / a) M(a,b,z) + ((b-1) / a) M(a,b,z)/b_ratio
 180:               //
~~~
- **EN:** This range declares or defines callable logic such as b_coef, boost::math::tools::function_ratio_from_backwards_recurrence.
- **CN:** 此范围声明或定义了可调用逻辑，例如 b_coef, boost::math::tools::function_ratio_from_backwards_recurrence。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:               T second = ((1 + a_local - b_local) / a_local) * h + ((b_local - 1) / a_local) * h / b_ratio;
 182:               boost::math::detail::hypergeometric_1F1_recurrence_a_coefficients<T> a_coef(a_local + 1, b_local, x);
 183:               h = boost::math::tools::apply_recurrence_relation_forward(a_coef, --a_shift, h, second, &log_scaling);
 184:            }
 185:         }
 186:         else
 187:         {
 188:            //
 189:            // We've calculated h for a larger value of a than we want, and need to recurse down.
 190:            // However, only forward iteration is stable, so calculate the ratio, compare values,
 191:            // and normalise.  Note that we calculate the ratio on b and convert to a since the
 192:            // direction is the minimal solution for N->+INF.
 193:            //
 194:            // IMPORTANT: this is only currently called for a > b and therefore forwards iteration
 195:            // is the only stable direction as we will only iterate down until a ~ b, but we
 196:            // will check this with an assert:
 197:            //
 198:            BOOST_MATH_ASSERT(2 * a - b_local + x > 0);
~~~
- **EN:** This range declares or defines callable logic such as a_coef, boost::math::tools::apply_recurrence_relation_forward, .... Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 a_coef, boost::math::tools::apply_recurrence_relation_forward, ...。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:            boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> b_coef(a, b_local, x);
 200:            std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 201:            T b_ratio = boost::math::tools::function_ratio_from_backwards_recurrence(b_coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 202:            boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_large_abz<%1%>(%1%,%1%,%1%)", max_iter, pol);
 203:            //
 204:            // Convert to a ratio:
 205:            //         (1+a-b)M(a, b, z) - aM(a+1, b, z) + (b-1)M(a, b-1, z) = 0
 206:            //
 207:            //  hence: M(a+1,b,z) = (1+a-b) / a M(a,b,z) + (b-1) / a M(a,b,z)/ (M(a,b,z)/M(a,b-1,z))
 208:            //
 209:            T first = 1;  // arbitrary value;
 210:            T second = ((1 + a - b_local) / a) + ((b_local - 1) / a) * (1 / b_ratio);
 211: 
 212:            if (a_shift == -1)
 213:               h = h / second;
 214:            else
 215:            {
 216:               boost::math::detail::hypergeometric_1F1_recurrence_a_coefficients<T> a_coef(a + 1, b_local, x);
~~~
- **EN:** This range declares or defines callable logic such as b_coef, boost::math::tools::function_ratio_from_backwards_recurrence, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 b_coef, boost::math::tools::function_ratio_from_backwards_recurrence, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:               T comparitor = boost::math::tools::apply_recurrence_relation_forward(a_coef, -(a_shift + 1), first, second);
 218:               if (boost::math::tools::min_value<T>() * comparitor > h)
 219:               {
 220:                  // Ooops, need to rescale h:
 221:                  long long rescale = lltrunc(log(fabs(h)));
 222:                  T scale = exp(T(-rescale));
 223:                  h *= scale;
 224:                  log_scaling += rescale;
 225:               }
 226:               h = h / comparitor;
 227:            }
 228:         }
 229:         return h;
 230:      }
 231: 
 232:      template <class T, class Policy>
 233:      T hypergeometric_1F1_shift_on_b(T h, const T& a, const T& b_local, const T& x, int b_shift, const Policy& pol, long long& log_scaling)
 234:      {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::tools::apply_recurrence_relation_forward, lltrunc, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::tools::apply_recurrence_relation_forward, lltrunc, ...。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:         BOOST_MATH_STD_USING
 236: 
 237:         T b = b_local + b_shift;
 238:         if (b_shift == 0)
 239:            return h;
 240:         else if (b_shift > 0)
 241:         {
 242:            //
 243:            // We get here for b_shift > 0 when b > z.  We can't use forward recursion on b - it's unstable,
 244:            // so grab the ratio and work backwards to b - b_shift and normalise.
 245:            //
 246:            boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> b_coef(a, b, x);
 247:            std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 248: 
 249:            T first = 1;  // arbitrary value;
 250:            T second = 1 / boost::math::tools::function_ratio_from_backwards_recurrence(b_coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 251:            boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_large_abz<%1%>(%1%,%1%,%1%)", max_iter, pol);
 252:            if (b_shift == 1)
~~~
- **EN:** This range declares or defines callable logic such as b_coef, boost::math::tools::function_ratio_from_backwards_recurrence. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 b_coef, boost::math::tools::function_ratio_from_backwards_recurrence。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:               h = h / second;
 254:            else
 255:            {
 256:               //
 257:               // Reset coefficients and recurse:
 258:               //
 259:               boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> b_coef_2(a, b - 1, x);
 260:               long long local_scale = 0;
 261:               T comparitor = boost::math::tools::apply_recurrence_relation_backward(b_coef_2, --b_shift, first, second, &local_scale);
 262:               log_scaling -= local_scale;
 263:               if (boost::math::tools::min_value<T>() * comparitor > h)
 264:               {
 265:                  // Ooops, need to rescale h:
 266:                  long long rescale = lltrunc(log(fabs(h)));
 267:                  T scale = exp(T(-rescale));
 268:                  h *= scale;
 269:                  log_scaling += rescale;
 270:               }
~~~
- **EN:** This range declares or defines callable logic such as b_coef_2, boost::math::tools::apply_recurrence_relation_backward, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 b_coef_2, boost::math::tools::apply_recurrence_relation_backward, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:               h = h / comparitor;
 272:            }
 273:         }
 274:         else
 275:         {
 276:            T second;
 277:            if (a == b_local)
 278:            {
 279:                // recurrence is trivial for a == b and method of ratios fails as the c-term goes to zero:
 280:               second = -b_local * (1 - b_local - x) * h / (b_local * (b_local - 1));
 281:            }
 282:            else
 283:            {
 284:               BOOST_MATH_ASSERT(!is_negative_integer(b - a));
 285:               boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> b_coef(a, b_local, x);
 286:               std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 287:               second = h / boost::math::tools::function_ratio_from_backwards_recurrence(b_coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 288:               boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_large_abz<%1%>(%1%,%1%,%1%)", max_iter, pol);
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, b_coef, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, b_coef, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:            }
 290:            if (b_shift == -1)
 291:               h = second;
 292:            else
 293:            {
 294:               boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> b_coef_2(a, b_local - 1, x);
 295:               h = boost::math::tools::apply_recurrence_relation_backward(b_coef_2, -(++b_shift), h, second, &log_scaling);
 296:            }
 297:         }
 298:         return h;
 299:      }
 300: 
 301: 
 302:      template <class T, class Policy>
 303:      T hypergeometric_1F1_large_igamma(const T& a, const T& b, const T& x, const T& b_minus_a, const Policy& pol, long long& log_scaling)
 304:      {
 305:         BOOST_MATH_STD_USING
 306:         //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as b_coef_2, boost::math::tools::apply_recurrence_relation_backward.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 b_coef_2, boost::math::tools::apply_recurrence_relation_backward。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:         // We need a < b < z in order to ensure there's at least a chance of convergence,
 308:         // we can use recurrence relations to shift forwards on a+b or just a to achieve this,
 309:         // for decent accuracy, try to keep 2b - 1 < a < 2b < z
 310:         //
 311:         int b_shift = b * 2 < x ? 0 : itrunc(b - x / 2);
 312:         int a_shift = a > b - b_shift ? -itrunc(b - b_shift - a - 1) : -itrunc(b - b_shift - a);
 313: 
 314:         if (a_shift < 0)
 315:         {
 316:            // Might as well do all the shifting on b as scale a downwards:
 317:            b_shift -= a_shift;
 318:            a_shift = 0;
 319:         }
 320: 
 321:         T a_local = a - a_shift;
 322:         T b_local = b - b_shift;
 323:         T b_minus_a_local = (a_shift == 0) && (b_shift == 0) ? b_minus_a : b_local - a_local;
 324:         long long local_scaling = 0;
~~~
- **EN:** This range declares or defines callable logic such as itrunc. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:         T h = hypergeometric_1F1_igamma(a_local, b_local, x, b_minus_a_local, pol, local_scaling);
 326:         log_scaling += local_scaling;
 327: 
 328:         //
 329:         // Apply shifts on a and b as required:
 330:         //
 331:         h = hypergeometric_1F1_shift_on_a(h, a_local, b_local, x, a_shift, pol, log_scaling);
 332:         h = hypergeometric_1F1_shift_on_b(h, a, b_local, x, b_shift, pol, log_scaling);
 333: 
 334:         return h;
 335:      }
 336: 
 337:      template <class T, class Policy>
 338:      T hypergeometric_1F1_large_series(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling)
 339:      {
 340:         BOOST_MATH_STD_USING
 341:         //
 342:         // We make a small, and b > z:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as hypergeometric_1F1_igamma, hypergeometric_1F1_shift_on_a, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 hypergeometric_1F1_igamma, hypergeometric_1F1_shift_on_a, ...。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:         //
 344:         int a_shift(0), b_shift(0);
 345:         if (a * z > b)
 346:         {
 347:            a_shift = itrunc(a) - 5;
 348:            b_shift = b < z ? itrunc(b - z - 1) : 0;
 349:         }
 350:         //
 351:         // If a_shift is trivially small, there's really not much point in losing
 352:         // accuracy to save a couple of iterations:
 353:         //
 354:         if (a_shift < 5)
 355:            a_shift = 0;
 356:         T a_local = a - a_shift;
 357:         T b_local = b - b_shift;
 358:         T h = boost::math::detail::hypergeometric_1F1_generic_series(a_local, b_local, z, pol, log_scaling, "hypergeometric_1F1_large_series<%1%>(a,b,z)");
 359:         //
 360:         // Apply shifts on a and b as required:
~~~
- **EN:** This range declares or defines callable logic such as a_shift, boost::math::detail::hypergeometric_1F1_generic_series. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 a_shift, boost::math::detail::hypergeometric_1F1_generic_series。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:         //
 362:         if (a_shift && (a_local == 0))
 363:         {
 364:            //
 365:            // Shifting on a via method of ratios in hypergeometric_1F1_shift_on_a fails when
 366:            // a_local == 0.  However, the value of h calculated was trivial (unity), so
 367:            // calculate a second 1F1 for a == 1 and recurse as normal:
 368:            //
 369:            long long scale = 0;
 370:            T h2 = boost::math::detail::hypergeometric_1F1_generic_series(T(a_local + 1), b_local, z, pol, scale, "hypergeometric_1F1_large_series<%1%>(a,b,z)");
 371:            if (scale != log_scaling)
 372:            {
 373:               h2 *= exp(T(scale - log_scaling));
 374:            }
 375:            boost::math::detail::hypergeometric_1F1_recurrence_a_coefficients<T> coef(a_local + 1, b_local, z);
 376:            h = boost::math::tools::apply_recurrence_relation_forward(coef, a_shift - 1, h, h2, &log_scaling);
 377:            h = hypergeometric_1F1_shift_on_b(h, a, b_local, z, b_shift, pol, log_scaling);
 378:         }
~~~
- **EN:** This range declares or defines callable logic such as boost::math::detail::hypergeometric_1F1_generic_series, exp, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::detail::hypergeometric_1F1_generic_series, exp, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:         else
 380:         {
 381:            h = hypergeometric_1F1_shift_on_a(h, a_local, b_local, z, a_shift, pol, log_scaling);
 382:            h = hypergeometric_1F1_shift_on_b(h, a, b_local, z, b_shift, pol, log_scaling);
 383:         }
 384:         return h;
 385:      }
 386: 
 387:      template <class T, class Policy>
 388:      T hypergeometric_1F1_large_13_3_6_series(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling)
 389:      {
 390:         BOOST_MATH_STD_USING
 391:         //
 392:         // A&S 13.3.6 is good only when a ~ b, but isn't too fussy on the size of z.
 393:         // So shift b to match a (b shifting seems to be more stable via method of ratios).
 394:         //
 395:         int b_shift = itrunc(b - a);
 396:         if ((b_shift < 0) && (b - b_shift != a))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as hypergeometric_1F1_shift_on_a, hypergeometric_1F1_shift_on_b, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 hypergeometric_1F1_shift_on_a, hypergeometric_1F1_shift_on_b, ...。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:            b_shift -= 1;
 398:         T b_local = b - b_shift;
 399:         if ((b_local - a - 0.5 <= 0) && (b_local != a))
 400:         {
 401:            // Make sure b_local - a - 0.5 > 0
 402:            b_shift -= 1;
 403:            b_local += 1;
 404:         }
 405:         T h = boost::math::detail::hypergeometric_1F1_AS_13_3_6(a, b_local, z, T(b_local - a), pol, log_scaling);
 406:         return hypergeometric_1F1_shift_on_b(h, a, b_local, z, b_shift, pol, log_scaling);
 407:      }
 408: 
 409:      template <class T, class Policy>
 410:      T hypergeometric_1F1_large_abz(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling)
 411:      {
 412:         BOOST_MATH_STD_USING
 413:         //
 414:         // This is the selection logic to pick the "best" method.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::detail::hypergeometric_1F1_AS_13_3_6.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::detail::hypergeometric_1F1_AS_13_3_6。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:         // We have a,b,z >> 0 and need to compute the approximate cost of each method
 416:         // and then select whichever wins out.
 417:         //
 418:         enum method
 419:         {
 420:            method_series = 0,
 421:            method_shifted_series,
 422:            method_gamma,
 423:            method_bessel
 424:         };
 425:         //
 426:         // Cost of direct series, is the approx number of terms required until we hit convergence:
 427:         //
 428:         T current_cost = (sqrt(16 * z * (3 * a + z) + 9 * b * b - 24 * b * z) - 3 * b + 4 * z) / 6;
 429:         method current_method = method_series;
 430:         //
 431:         // Cost of shifted series, is the number of recurrences required to move to a zone where
 432:         // the series is convergent right from the start.
~~~
- **EN:** The enum `method` names a constrained set of compile-time or runtime states.
- **CN:** 枚举 `method` 为一组受限的编译期或运行期状态命名。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:         // Note that recurrence relations fail for very small b, and too many recurrences on a
 434:         // will completely destroy all our digits.
 435:         // Also note that the method fails when b-a is a negative integer unless b is already
 436:         // larger than z and thus does not need shifting.
 437:         //
 438:         T cost = a + ((b < z) ? T(z - b) : T(0));
 439:         if((b > 1) && (cost < current_cost) && ((b > z) || !is_negative_integer(b-a)))
 440:         {
 441:            current_method = method_shifted_series;
 442:            current_cost = cost;
 443:         }
 444:         //
 445:         // Cost for gamma function method is the number of recurrences required to move it
 446:         // into a convergent zone, note that recurrence relations fail for very small b.
 447:         // Also add on a fudge factor to account for the fact that this method is both
 448:         // more expensive to compute (requires gamma functions), and less accurate than the
 449:         // methods above:
 450:         //
~~~
- **EN:** This range declares or defines callable logic such as T. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 T。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:         T b_shift = fabs(b * 2 < z ? T(0) : T(b - z / 2));
 452:         T a_shift = fabs(a > b - b_shift ? T(-(b - b_shift - a - 1)) : T(-(b - b_shift - a)));
 453:         cost = 1000 + b_shift + a_shift;
 454:         if((b > 1) && (cost <= current_cost))
 455:         {
 456:            current_method = method_gamma;
 457:            current_cost = cost;
 458:         }
 459:         //
 460:         // Cost for bessel approximation is the number of recurrences required to make a ~ b,
 461:         // Note that recurrence relations fail for very small b.  We also have issue with large
 462:         // z: either overflow/numeric instability or else the series goes divergent.  We seem to be
 463:         // OK for z smaller than log_max_value<Quad> though, maybe we can stretch a little further
 464:         // but that's not clear...
 465:         // Also need to add on a fudge factor to the cost to account for the fact that we need
 466:         // to calculate the Bessel functions, this is not quite as high as the gamma function
 467:         // method above as this is generally more accurate and so preferred if the methods are close:
 468:         //
~~~
- **EN:** This range declares or defines callable logic such as fabs. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:         cost = 50 + fabs(b - a);
 470:         if((b > 1) && (cost <= current_cost) && (z < tools::log_max_value<T>()) && (z < 11356) && (b - a != 0.5f))
 471:         {
 472:            current_method = method_bessel;
 473:            current_cost = cost;
 474:         }
 475: 
 476:         switch (current_method)
 477:         {
 478:         case method_series:
 479:            return detail::hypergeometric_1F1_generic_series(a, b, z, pol, log_scaling, "hypergeometric_1f1_large_abz<%1%>(a,b,z)");
 480:         case method_shifted_series:
 481:            return detail::hypergeometric_1F1_large_series(a, b, z, pol, log_scaling);
 482:         case method_gamma:
 483:            return detail::hypergeometric_1F1_large_igamma(a, b, z, T(b - a), pol, log_scaling);
 484:         case method_bessel:
 485:            return detail::hypergeometric_1F1_large_13_3_6_series(a, b, z, pol, log_scaling);
 486:         }
~~~
- **EN:** This range declares or defines callable logic such as fabs. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 487-492 / 第 487-492 行
~~~cpp
 487:         return 0; // We don't get here!
 488:      }
 489: 
 490:   } } } // namespaces
 491: 
 492: #endif // BOOST_HYPERGEOMETRIC_1F1_LARGE_ABZ_HPP_
~~~
- **EN:** This short range closes a conditional-compilation or header-guard region. This range mainly closes the preceding type, namespace, or conditional scope. Return statements hand the computed result or status back to the caller.
- **CN:** 这一小段用于结束条件编译区域或头文件保护区域。 此范围主要用于结束前面的类型、命名空间或条件作用域。 return 语句会把计算结果或状态返回给调用方。

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
- **Included headers / 包含头文件**: `boost/math/special_functions/detail/hypergeometric_1F1_bessel.hpp, boost/math/special_functions/detail/hypergeometric_series.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/trunc.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `lltrunc, exp, refill_cache, boost::math::gamma_p, T, s, boost::math::tools::sum_series, boost::math::lgamma, ...`
