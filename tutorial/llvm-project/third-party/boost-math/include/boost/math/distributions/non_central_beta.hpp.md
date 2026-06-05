# non_central_beta.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/non_central_beta.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the non central beta distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 non central beta 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行
~~~cpp
   1: // boost\math\distributions\non_central_beta.hpp
   2: 
   3: // Copyright John Maddock 2008.
   4: // Copyright Matt Borland 2024.
   5: // Use, modification and distribution are subject to the
   6: // Boost Software License, Version 1.0.
   7: // (See accompanying file LICENSE_1_0.txt
   8: // or copy at http://www.boost.org/LICENSE_1_0.txt)
   9: 
  10: #ifndef BOOST_MATH_SPECIAL_NON_CENTRAL_BETA_HPP
  11: #define BOOST_MATH_SPECIAL_NON_CENTRAL_BETA_HPP
  12: 
  13: #include <boost/math/tools/config.hpp>
  14: #include <boost/math/tools/tuple.hpp>
  15: #include <boost/math/tools/cstdint.hpp>
  16: #include <boost/math/tools/numeric_limits.hpp>
  17: #include <boost/math/distributions/fwd.hpp>
  18: #include <boost/math/special_functions/beta.hpp> // for incomplete gamma. gamma_q
  19: #include <boost/math/distributions/complement.hpp> // complements
  20: #include <boost/math/distributions/beta.hpp> // central distribution
  21: #include <boost/math/distributions/detail/generic_mode.hpp>
  22: #include <boost/math/distributions/detail/common_error_handling.hpp> // error checks
  23: #include <boost/math/special_functions/fpclassify.hpp> // isnan.
  24: #include <boost/math/special_functions/trunc.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 25-48 / 第 25-48 行
~~~cpp
  25: #include <boost/math/tools/roots.hpp> // for root finding.
  26: #include <boost/math/tools/series.hpp>
  27: #include <boost/math/policies/error_handling.hpp>
  28: 
  29: namespace boost
  30: {
  31:    namespace math
  32:    {
  33: 
  34:       template <class RealType, class Policy>
  35:       class non_central_beta_distribution;
  36: 
  37:       namespace detail{
  38: 
  39:          template <class T, class Policy>
  40:          BOOST_MATH_GPU_ENABLED T non_central_beta_p(T a, T b, T lam, T x, T y, const Policy& pol, T init_val = 0)
  41:          {
  42:             BOOST_MATH_STD_USING
  43:                using namespace boost::math;
  44:             //
  45:             // Variables come first:
  46:             //
  47:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
  48:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
~~~
- **EN:** This block imports dependencies such as boost/math/tools/roots.hpp, boost/math/tools/series.hpp, boost/math/policies/error_handling.hpp so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/tools/roots.hpp, boost/math/tools/series.hpp, boost/math/policies/error_handling.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 49-72 / 第 49-72 行
~~~cpp
  49:             T l2 = lam / 2;
  50:             //
  51:             // k is the starting point for iteration, and is the
  52:             // maximum of the poisson weighting term,
  53:             // note that unlike other similar code, we do not set
  54:             // k to zero, when l2 is small, as forward iteration
  55:             // is unstable:
  56:             //
  57:             long long k = lltrunc(l2);
  58:             if(k == 0)
  59:                k = 1;
  60:                // Starting Poisson weight:
  61:             T pois = gamma_p_derivative(T(k+1), l2, pol);
  62:             if(pois == 0)
  63:                return init_val;
  64:             // recurance term:
  65:             T xterm;
  66:             // Starting beta term:
  67:             T beta = x < y
  68:                ? detail::ibeta_imp(T(a + k), b, x, pol, false, true, &xterm)
  69:                : detail::ibeta_imp(b, T(a + k), y, pol, true, true, &xterm);
  70: 
  71:             while (fabs(beta * pois) < tools::min_value<T>())
  72:             {
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, gamma_p_derivative, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, gamma_p_derivative, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 73-96 / 第 73-96 行
~~~cpp
  73:                if ((k == 0) || (pois == 0))
  74:                   return init_val;
  75:                k /= 2;
  76:                pois = gamma_p_derivative(T(k + 1), l2, pol);
  77:                beta = x < y
  78:                   ? detail::ibeta_imp(T(a + k), b, x, pol, false, true, &xterm)
  79:                   : detail::ibeta_imp(b, T(a + k), y, pol, true, true, &xterm);
  80:             }
  81: 
  82:             xterm *= y / (a + b + k - 1);
  83:             T poisf(pois), betaf(beta), xtermf(xterm);
  84:             T sum = init_val;
  85: 
  86:             if((beta == 0) && (xterm == 0))
  87:                return init_val;
  88: 
  89:             //
  90:             // Backwards recursion first, this is the stable
  91:             // direction for recursion:
  92:             //
  93:             T last_term = 0;
  94:             boost::math::uintmax_t count = k;
  95:             for(auto i = k; i >= 0; --i)
  96:             {
~~~
- **EN:** This range declares or defines callable logic such as gamma_p_derivative, detail::ibeta_imp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 gamma_p_derivative, detail::ibeta_imp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 97-120 / 第 97-120 行
~~~cpp
  97:                T term = beta * pois;
  98:                sum += term;
  99:                if(((fabs(term/sum) < errtol) && (fabs(last_term) >= fabs(term))) || (term == 0))
 100:                {
 101:                   count = k - i;
 102:                   break;
 103:                }
 104:                pois *= i / l2;
 105:                beta += xterm;
 106: 
 107:                if (a + b + i != 2)
 108:                {
 109:                   xterm *= (a + i - 1) / (x * (a + b + i - 2));
 110:                }
 111: 
 112:                last_term = term;
 113:             }
 114:             last_term = 0;
 115:             for(auto i = k + 1; ; ++i)
 116:             {
 117:                poisf *= l2 / i;
 118:                xtermf *= (x * (a + b + i - 2)) / (a + i - 1);
 119:                betaf -= xtermf;
 120: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 121-144 / 第 121-144 行
~~~cpp
 121:                T term = poisf * betaf;
 122:                sum += term;
 123:                if(((fabs(term/sum) < errtol) && (fabs(last_term) >= fabs(term))) || (term == 0))
 124:                {
 125:                   break;
 126:                }
 127:                last_term = term;
 128:                if(static_cast<boost::math::uintmax_t>(count + i - k) > max_iter)
 129:                {
 130:                   return policies::raise_evaluation_error("cdf(non_central_beta_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 131:                }
 132:             }
 133:             return sum;
 134:          }
 135: 
 136:          template <class T, class Policy>
 137:          BOOST_MATH_GPU_ENABLED T non_central_beta_q(T a, T b, T lam, T x, T y, const Policy& pol, T init_val = 0)
 138:          {
 139:             BOOST_MATH_STD_USING
 140:                using namespace boost::math;
 141:             //
 142:             // Variables come first:
 143:             //
 144:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 145-168 / 第 145-168 行
~~~cpp
 145:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
 146:             T l2 = lam / 2;
 147:             //
 148:             // k is the starting point for iteration, and is the
 149:             // maximum of the poisson weighting term:
 150:             //
 151:             long long k = lltrunc(l2);
 152:             T pois;
 153:             if(k <= 30)
 154:             {
 155:                //
 156:                // Might as well start at 0 since we'll likely have this number of terms anyway:
 157:                //
 158:                if(a + b > 1)
 159:                   k = 0;
 160:                else if(k == 0)
 161:                   k = 1;
 162:             }
 163:             if(k == 0)
 164:             {
 165:                // Starting Poisson weight:
 166:                pois = exp(-l2);
 167:             }
 168:             else
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 169-192 / 第 169-192 行
~~~cpp
 169:             {
 170:                // Starting Poisson weight:
 171:                pois = gamma_p_derivative(T(k+1), l2, pol);
 172:             }
 173:             if(pois == 0)
 174:                return init_val;
 175:             // recurance term:
 176:             T xterm;
 177:             // Starting beta term:
 178:             T beta = x < y
 179:                ? detail::ibeta_imp(T(a + k), b, x, pol, true, true, &xterm)
 180:                : detail::ibeta_imp(b, T(a + k), y, pol, false, true, &xterm);
 181: 
 182:             xterm *= y / (a + b + k - 1);
 183:             T poisf(pois), betaf(beta), xtermf(xterm);
 184:             T sum = init_val;
 185:             if((beta == 0) && (xterm == 0))
 186:                return init_val;
 187:             //
 188:             // Forwards recursion first, this is the stable
 189:             // direction for recursion, and the location
 190:             // of the bulk of the sum:
 191:             //
 192:             T last_term = 0;
~~~
- **EN:** This range declares or defines callable logic such as gamma_p_derivative, detail::ibeta_imp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 gamma_p_derivative, detail::ibeta_imp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 193-216 / 第 193-216 行
~~~cpp
 193:             boost::math::uintmax_t count = 0;
 194:             for(long long i = k + 1; ; ++i)
 195:             {
 196:                poisf *= l2 / i;
 197:                xtermf *= (x * (a + b + (i - 2))) / (a + (i - 1));
 198:                betaf += xtermf;
 199: 
 200:                T term = poisf * betaf;
 201:                sum += term;
 202:                if((fabs(term/sum) < errtol) && (last_term >= term))
 203:                {
 204:                   count = i - k;
 205:                   break;
 206:                }
 207:                if(static_cast<boost::math::uintmax_t>(i - k) > max_iter)
 208:                {
 209:                   return policies::raise_evaluation_error("cdf(non_central_beta_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 210:                }
 211:                last_term = term;
 212:             }
 213:             for(auto i = k; i >= 0; --i)
 214:             {
 215:                T term = beta * pois;
 216:                sum += term;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-240 / 第 217-240 行
~~~cpp
 217:                if(fabs(term/sum) < errtol)
 218:                {
 219:                   break;
 220:                }
 221:                if(static_cast<boost::math::uintmax_t>(count + k - i) > max_iter)
 222:                {
 223:                   return policies::raise_evaluation_error("cdf(non_central_beta_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 224:                }
 225:                pois *= i / l2;
 226:                beta -= xterm;
 227:                if (a + b + i - 2 != 0)
 228:                {
 229:                    xterm *= (a + i - 1) / (x * (a + b + i - 2));
 230:                }
 231:             }
 232:             return sum;
 233:          }
 234: 
 235:          template <class RealType, class Policy>
 236:          BOOST_MATH_GPU_ENABLED inline RealType non_central_beta_cdf(RealType x, RealType y, RealType a, RealType b, RealType l, bool invert, const Policy&)
 237:          {
 238:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 239:             typedef typename policies::normalise<
 240:                Policy,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 241-264 / 第 241-264 行
~~~cpp
 241:                policies::promote_float<false>,
 242:                policies::promote_double<false>,
 243:                policies::discrete_quantile<>,
 244:                policies::assert_undefined<> >::type forwarding_policy;
 245: 
 246:             BOOST_MATH_STD_USING
 247: 
 248:             if(x == 0)
 249:                return invert ? 1.0f : 0.0f;
 250:             if(y == 0)
 251:                return invert ? 0.0f : 1.0f;
 252:             value_type result;
 253:             value_type c = a + b + l / 2;
 254:             value_type cross = 1 - (b / c) * (1 + l / (2 * c * c));
 255:             if(l == 0)
 256:                result = cdf(boost::math::beta_distribution<RealType, Policy>(a, b), x);
 257:             else if(x > cross)
 258:             {
 259:                // Complement is the smaller of the two:
 260:                result = detail::non_central_beta_q(
 261:                   static_cast<value_type>(a),
 262:                   static_cast<value_type>(b),
 263:                   static_cast<value_type>(l),
 264:                   static_cast<value_type>(x),
~~~
- **EN:** This range declares or defines callable logic such as cdf. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 cdf。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 265-288 / 第 265-288 行
~~~cpp
 265:                   static_cast<value_type>(y),
 266:                   forwarding_policy(),
 267:                   static_cast<value_type>(invert ? 0 : -1));
 268:                invert = !invert;
 269:             }
 270:             else
 271:             {
 272:                result = detail::non_central_beta_p(
 273:                   static_cast<value_type>(a),
 274:                   static_cast<value_type>(b),
 275:                   static_cast<value_type>(l),
 276:                   static_cast<value_type>(x),
 277:                   static_cast<value_type>(y),
 278:                   forwarding_policy(),
 279:                   static_cast<value_type>(invert ? -1 : 0));
 280:             }
 281:             if(invert)
 282:                result = -result;
 283:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 284:                result,
 285:                "boost::math::non_central_beta_cdf<%1%>(%1%, %1%, %1%)");
 286:          }
 287: 
 288:          template <class T, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 289-312 / 第 289-312 行
~~~cpp
 289:          struct nc_beta_quantile_functor
 290:          {
 291:             BOOST_MATH_GPU_ENABLED nc_beta_quantile_functor(const non_central_beta_distribution<T,Policy>& d, T t, bool c)
 292:                : dist(d), target(t), comp(c) {}
 293: 
 294:             BOOST_MATH_GPU_ENABLED T operator()(const T& x)
 295:             {
 296:                return comp ?
 297:                   T(target - cdf(complement(dist, x)))
 298:                   : T(cdf(dist, x) - target);
 299:             }
 300: 
 301:          private:
 302:             non_central_beta_distribution<T,Policy> dist;
 303:             T target;
 304:             bool comp;
 305:          };
 306: 
 307:          //
 308:          // This is more or less a copy of bracket_and_solve_root, but
 309:          // modified to search only the interval [0,1] using similar
 310:          // heuristics.
 311:          //
 312:          template <class F, class T, class Tol, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `nc_beta_quantile_functor` as part of the file's main abstraction. This range declares or defines callable logic such as dist, T.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `nc_beta_quantile_functor`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 dist, T。

### Lines 313-336 / 第 313-336 行
~~~cpp
 313:          BOOST_MATH_GPU_ENABLED boost::math::pair<T, T> bracket_and_solve_root_01(F f, const T& guess, T factor, bool rising, Tol tol, boost::math::uintmax_t& max_iter, const Policy& pol)
 314:          {
 315:             BOOST_MATH_STD_USING
 316:                constexpr auto function = "boost::math::tools::bracket_and_solve_root_01<%1%>";
 317:             //
 318:             // Set up initial brackets:
 319:             //
 320:             T a = guess;
 321:             T b = a;
 322:             T fa = f(a);
 323:             T fb = fa;
 324:             //
 325:             // Set up invocation count:
 326:             //
 327:             boost::math::uintmax_t count = max_iter - 1;
 328: 
 329:             if((fa < 0) == (guess < 0 ? !rising : rising))
 330:             {
 331:                //
 332:                // Zero is to the right of b, so walk upwards
 333:                // until we find it:
 334:                //
 335:                while((boost::math::sign)(fb) == (boost::math::sign)(fa))
 336:                {
~~~
- **EN:** This range declares or defines callable logic such as f. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 f。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 337-360 / 第 337-360 行
~~~cpp
 337:                   if(count == 0)
 338:                   {
 339:                      b = policies::raise_evaluation_error(function, "Unable to bracket root, last nearest value was %1%", b, pol); // LCOV_EXCL_LINE
 340:                      return boost::math::make_pair(a, b);
 341:                   }
 342:                   //
 343:                   // Heuristic: every 20 iterations we double the growth factor in case the
 344:                   // initial guess was *really* bad !
 345:                   //
 346:                   if((max_iter - count) % 20 == 0)
 347:                      factor *= 2;
 348:                   //
 349:                   // Now go ahead and move are guess by "factor",
 350:                   // we do this by reducing 1-guess by factor:
 351:                   //
 352:                   a = b;
 353:                   fa = fb;
 354:                   b = 1 - ((1 - b) / factor);
 355:                   fb = f(b);
 356:                   --count;
 357:                   BOOST_MATH_INSTRUMENT_CODE("a = " << a << " b = " << b << " fa = " << fa << " fb = " << fb << " count = " << count);
 358:                }
 359:             }
 360:             else
~~~
- **EN:** This range declares or defines callable logic such as policies::raise_evaluation_error, f, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 policies::raise_evaluation_error, f, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-384 / 第 361-384 行
~~~cpp
 361:             {
 362:                //
 363:                // Zero is to the left of a, so walk downwards
 364:                // until we find it:
 365:                //
 366:                while((boost::math::sign)(fb) == (boost::math::sign)(fa))
 367:                {
 368:                   if(fabs(a) < tools::min_value<T>())
 369:                   {
 370:                      // Escape route just in case the answer is zero!
 371:                      max_iter -= count;
 372:                      max_iter += 1;
 373:                      return a > 0 ? boost::math::make_pair(T(0), T(a)) : boost::math::make_pair(T(a), T(0));
 374:                   }
 375:                   if(count == 0)
 376:                   {
 377:                      a = policies::raise_evaluation_error(function, "Unable to bracket root, last nearest value was %1%", a, pol); // LCOV_EXCL_LINE
 378:                      return boost::math::make_pair(a, b);
 379:                   }
 380:                   //
 381:                   // Heuristic: every 20 iterations we double the growth factor in case the
 382:                   // initial guess was *really* bad !
 383:                   //
 384:                   if((max_iter - count) % 20 == 0)
~~~
- **EN:** This range declares or defines callable logic such as policies::raise_evaluation_error. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 policies::raise_evaluation_error。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 385-408 / 第 385-408 行
~~~cpp
 385:                      factor *= 2;
 386:                   //
 387:                   // Now go ahead and move are guess by "factor":
 388:                   //
 389:                   b = a;
 390:                   fb = fa;
 391:                   a /= factor;
 392:                   fa = f(a);
 393:                   --count;
 394:                   BOOST_MATH_INSTRUMENT_CODE("a = " << a << " b = " << b << " fa = " << fa << " fb = " << fb << " count = " << count);
 395:                }
 396:             }
 397:             max_iter -= count;
 398:             max_iter += 1;
 399:             boost::math::pair<T, T> r = toms748_solve(
 400:                f,
 401:                (a < 0 ? b : a),
 402:                (a < 0 ? a : b),
 403:                (a < 0 ? fb : fa),
 404:                (a < 0 ? fa : fb),
 405:                tol,
 406:                count,
 407:                pol);
 408:             max_iter += count;
~~~
- **EN:** This range declares or defines callable logic such as f, BOOST_MATH_INSTRUMENT_CODE. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 f, BOOST_MATH_INSTRUMENT_CODE。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 409-432 / 第 409-432 行
~~~cpp
 409:             BOOST_MATH_INSTRUMENT_CODE("max_iter = " << max_iter << " count = " << count);
 410:             return r;
 411:          }
 412: 
 413:          template <class RealType, class Policy>
 414:          BOOST_MATH_GPU_ENABLED RealType nc_beta_quantile(const non_central_beta_distribution<RealType, Policy>& dist, const RealType& p, bool comp)
 415:          {
 416:             constexpr auto function = "quantile(non_central_beta_distribution<%1%>, %1%)";
 417:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 418:             typedef typename policies::normalise<
 419:                Policy,
 420:                policies::promote_float<false>,
 421:                policies::promote_double<false>,
 422:                policies::discrete_quantile<>,
 423:                policies::assert_undefined<> >::type forwarding_policy;
 424: 
 425:             value_type a = dist.alpha();
 426:             value_type b = dist.beta();
 427:             value_type l = dist.non_centrality();
 428:             value_type r;
 429:             if(!beta_detail::check_alpha(
 430:                function,
 431:                a, &r, Policy())
 432:                ||
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_CODE, alpha, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_CODE, alpha, ...。

### Lines 433-456 / 第 433-456 行
~~~cpp
 433:             !beta_detail::check_beta(
 434:                function,
 435:                b, &r, Policy())
 436:                ||
 437:             !detail::check_non_centrality(
 438:                function,
 439:                l,
 440:                &r,
 441:                Policy())
 442:                ||
 443:             !detail::check_probability(
 444:                function,
 445:                static_cast<value_type>(p),
 446:                &r,
 447:                Policy()))
 448:                   return static_cast<RealType>(r);
 449:             //
 450:             // Special cases first:
 451:             //
 452:             if(p == 0)
 453:                return comp
 454:                ? 1.0f
 455:                : 0.0f;
 456:             if(p == 1)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 457-480 / 第 457-480 行
~~~cpp
 457:                return !comp
 458:                ? 1.0f
 459:                : 0.0f;
 460: 
 461:             value_type c = a + b + l / 2;
 462:             value_type mean = 1 - (b / c) * (1 + l / (2 * c * c));
 463:             /*
 464:             //
 465:             // Calculate a normal approximation to the quantile,
 466:             // uses mean and variance approximations from:
 467:             // Algorithm AS 310:
 468:             // Computing the Non-Central Beta Distribution Function
 469:             // R. Chattamvelli; R. Shanmugam
 470:             // Applied Statistics, Vol. 46, No. 1. (1997), pp. 146-156.
 471:             //
 472:             // Unfortunately, when this is wrong it tends to be *very*
 473:             // wrong, so it's disabled for now, even though it often
 474:             // gets the initial guess quite close.  Probably we could
 475:             // do much better by factoring in the skewness if only
 476:             // we could calculate it....
 477:             //
 478:             value_type delta = l / 2;
 479:             value_type delta2 = delta * delta;
 480:             value_type delta3 = delta * delta2;
~~~
- **EN:** Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 481-504 / 第 481-504 行
~~~cpp
 481:             value_type delta4 = delta2 * delta2;
 482:             value_type G = c * (c + 1) + delta;
 483:             value_type alpha = a + b;
 484:             value_type alpha2 = alpha * alpha;
 485:             value_type eta = (2 * alpha + 1) * (2 * alpha + 1) + 1;
 486:             value_type H = 3 * alpha2 + 5 * alpha + 2;
 487:             value_type F = alpha2 * (alpha + 1) + H * delta
 488:                + (2 * alpha + 4) * delta2 + delta3;
 489:             value_type P = (3 * alpha + 1) * (9 * alpha + 17)
 490:                + 2 * alpha * (3 * alpha + 2) * (3 * alpha + 4) + 15;
 491:             value_type Q = 54 * alpha2 + 162 * alpha + 130;
 492:             value_type R = 6 * (6 * alpha + 11);
 493:             value_type D = delta
 494:                * (H * H + 2 * P * delta + Q * delta2 + R * delta3 + 9 * delta4);
 495:             value_type variance = (b / G)
 496:                * (1 + delta * (l * l + 3 * l + eta) / (G * G))
 497:                - (b * b / F) * (1 + D / (F * F));
 498:             value_type sd = sqrt(variance);
 499: 
 500:             value_type guess = comp
 501:                ? quantile(complement(normal_distribution<RealType, Policy>(static_cast<RealType>(mean), static_cast<RealType>(sd)), p))
 502:                : quantile(normal_distribution<RealType, Policy>(static_cast<RealType>(mean), static_cast<RealType>(sd)), p);
 503: 
 504:             if(guess >= 1)
~~~
- **EN:** This range declares or defines callable logic such as sqrt, quantile. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, quantile。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 505-528 / 第 505-528 行
~~~cpp
 505:                guess = mean;
 506:             if(guess <= tools::min_value<value_type>())
 507:                guess = mean;
 508:             */
 509:             value_type guess = mean;
 510:             detail::nc_beta_quantile_functor<value_type, Policy>
 511:                f(non_central_beta_distribution<value_type, Policy>(a, b, l), p, comp);
 512:             tools::eps_tolerance<value_type> tol(policies::digits<RealType, Policy>());
 513:             boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 514: 
 515:             boost::math::pair<value_type, value_type> ir
 516:                = bracket_and_solve_root_01(
 517:                   f, guess, value_type(2.5), true, tol,
 518:                   max_iter, Policy());
 519:             value_type result = ir.first + (ir.second - ir.first) / 2;
 520: 
 521:             if(max_iter >= policies::get_max_root_iterations<Policy>())
 522:             {
 523:                // LCOV_EXCL_START
 524:                return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:"
 525:                   " either there is no answer to quantile of the non central beta distribution"
 526:                   " or the answer is infinite.  Current best guess is %1%",
 527:                   policies::checked_narrowing_cast<RealType, forwarding_policy>(
 528:                      result,
~~~
- **EN:** This range declares or defines callable logic such as f, tol, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 f, tol, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 529-552 / 第 529-552 行
~~~cpp
 529:                      function), Policy());
 530:                // LCOV_EXCL_STOP
 531:             }
 532:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 533:                result,
 534:                function);
 535:          }
 536: 
 537:          template <class T, class Policy>
 538:          BOOST_MATH_GPU_ENABLED T non_central_beta_pdf(T a, T b, T lam, T x, T y, const Policy& pol)
 539:          {
 540:             BOOST_MATH_STD_USING
 541:             //
 542:             // Special cases:
 543:             //
 544:             if((x == 0) || (y == 0))
 545:                return 0;
 546:             //
 547:             // Variables come first:
 548:             //
 549:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 550:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
 551:             T l2 = lam / 2;
 552:             //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy。

### Lines 553-576 / 第 553-576 行
~~~cpp
 553:             // k is the starting point for iteration, and is the
 554:             // maximum of the poisson weighting term:
 555:             //
 556:             long long k = lltrunc(l2);
 557:             // Starting Poisson weight:
 558:             T pois = gamma_p_derivative(T(k+1), l2, pol);
 559:             // Starting beta term:
 560:             T beta = x < y ?
 561:                ibeta_derivative(a + k, b, x, pol)
 562:                : ibeta_derivative(b, a + k, y, pol);
 563: 
 564:             while (fabs(beta * pois) < tools::min_value<T>())
 565:             {
 566:                if ((k == 0) || (pois == 0))
 567:                   return 0;  // Nothing else we can do!
 568:                //
 569:                // We only get here when a+k and b are large and x is small,
 570:                // in that case reduce k (bisect) until both terms are finite:
 571:                //
 572:                k /= 2;
 573:                pois = gamma_p_derivative(T(k + 1), l2, pol);
 574:                // Starting beta term:
 575:                beta = x < y ?
 576:                   ibeta_derivative(a + k, b, x, pol)
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, gamma_p_derivative, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, gamma_p_derivative, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 577-600 / 第 577-600 行
~~~cpp
 577:                   : ibeta_derivative(b, a + k, y, pol);
 578:             }
 579: 
 580: 
 581:             T sum = 0;
 582:             T poisf(pois);
 583:             T betaf(beta);
 584: 
 585:             //
 586:             // Stable backwards recursion first:
 587:             //
 588:             boost::math::uintmax_t count = k;
 589:             T ratio = 0;
 590:             T old_ratio = 0;
 591:             for(auto i = k; i >= 0; --i)
 592:             {
 593:                T term = beta * pois;
 594:                sum += term;
 595:                ratio = fabs(term / sum);
 596:                if(((ratio < errtol) && (ratio < old_ratio)) || (term == 0))
 597:                {
 598:                   count = k - i;
 599:                   break;
 600:                }
~~~
- **EN:** This range declares or defines callable logic such as ibeta_derivative, poisf, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ibeta_derivative, poisf, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 601-624 / 第 601-624 行
~~~cpp
 601:                ratio = old_ratio;
 602:                pois *= i / l2;
 603: 
 604:                if (a + b + i != 1)
 605:                {
 606:                   beta *= (a + i - 1) / (x * (a + i + b - 1));
 607:                }
 608:             }
 609:             old_ratio = 0;
 610:             for(auto i = k + 1; ; ++i)
 611:             {
 612:                poisf *= l2 / i;
 613:                betaf *= x * (a + b + i - 1) / (a + i - 1);
 614: 
 615:                T term = poisf * betaf;
 616:                sum += term;
 617:                ratio = fabs(term / sum);
 618:                if(((ratio < errtol) && (ratio < old_ratio)) || (term == 0))
 619:                {
 620:                   break;
 621:                }
 622:                old_ratio = ratio;
 623:                if(static_cast<boost::math::uintmax_t>(count + i - k) > max_iter)
 624:                {
~~~
- **EN:** This range declares or defines callable logic such as fabs. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 625-648 / 第 625-648 行
~~~cpp
 625:                   return policies::raise_evaluation_error("pdf(non_central_beta_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 626:                }
 627:             }
 628:             return sum;
 629:          }
 630: 
 631:          template <class RealType, class Policy>
 632:          BOOST_MATH_GPU_ENABLED RealType nc_beta_pdf(const non_central_beta_distribution<RealType, Policy>& dist, const RealType& x)
 633:          {
 634:             BOOST_MATH_STD_USING
 635:             constexpr auto function = "pdf(non_central_beta_distribution<%1%>, %1%)";
 636:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 637:             typedef typename policies::normalise<
 638:                Policy,
 639:                policies::promote_float<false>,
 640:                policies::promote_double<false>,
 641:                policies::discrete_quantile<>,
 642:                policies::assert_undefined<> >::type forwarding_policy;
 643: 
 644:             value_type a = dist.alpha();
 645:             value_type b = dist.beta();
 646:             value_type l = dist.non_centrality();
 647:             value_type r;
 648:             if(!beta_detail::check_alpha(
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as alpha, beta, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 alpha, beta, ...。

### Lines 649-672 / 第 649-672 行
~~~cpp
 649:                function,
 650:                a, &r, Policy())
 651:                ||
 652:             !beta_detail::check_beta(
 653:                function,
 654:                b, &r, Policy())
 655:                ||
 656:             !detail::check_non_centrality(
 657:                function,
 658:                l,
 659:                &r,
 660:                Policy())
 661:                ||
 662:             !beta_detail::check_x(
 663:                function,
 664:                static_cast<value_type>(x),
 665:                &r,
 666:                Policy()))
 667:                   return static_cast<RealType>(r);
 668: 
 669:             if(l == 0)
 670:                return pdf(boost::math::beta_distribution<RealType, Policy>(dist.alpha(), dist.beta()), x);
 671:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 672:                non_central_beta_pdf(a, b, l, static_cast<value_type>(x), value_type(1 - static_cast<value_type>(x)), forwarding_policy()),
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 673-696 / 第 673-696 行
~~~cpp
 673:                "function");
 674:          }
 675: 
 676:          template <class T>
 677:          struct hypergeometric_2F2_sum
 678:          {
 679:             typedef T result_type;
 680:             BOOST_MATH_GPU_ENABLED hypergeometric_2F2_sum(T a1_, T a2_, T b1_, T b2_, T z_) : a1(a1_), a2(a2_), b1(b1_), b2(b2_), z(z_), term(1), k(0) {}
 681:             BOOST_MATH_GPU_ENABLED T operator()()
 682:             {
 683:                T result = term;
 684:                term *= a1 * a2 / (b1 * b2);
 685:                a1 += 1;
 686:                a2 += 1;
 687:                b1 += 1;
 688:                b2 += 1;
 689:                k += 1;
 690:                term /= k;
 691:                term *= z;
 692:                return result;
 693:             }
 694:             T a1, a2, b1, b2, z, term, k;
 695:          };
 696: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as hypergeometric_2F2_sum.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 hypergeometric_2F2_sum。

### Lines 697-720 / 第 697-720 行
~~~cpp
 697:          template <class T, class Policy>
 698:          BOOST_MATH_GPU_ENABLED T hypergeometric_2F2(T a1, T a2, T b1, T b2, T z, const Policy& pol)
 699:          {
 700:             typedef typename policies::evaluation<T, Policy>::type value_type;
 701: 
 702:             const char* function = "boost::math::detail::hypergeometric_2F2<%1%>(%1%,%1%,%1%,%1%,%1%)";
 703: 
 704:             hypergeometric_2F2_sum<value_type> s(a1, a2, b1, b2, z);
 705:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 706: 
 707:             value_type result = boost::math::tools::sum_series(s, boost::math::policies::get_epsilon<value_type, Policy>(), max_iter);
 708: 
 709:             policies::check_series_iterations<T>(function, max_iter, pol);
 710:             return policies::checked_narrowing_cast<T, Policy>(result, function);
 711:          }
 712: 
 713:       } // namespace detail
 714: 
 715:       template <class RealType = double, class Policy = policies::policy<> >
 716:       class non_central_beta_distribution
 717:       {
 718:       public:
 719:          typedef RealType value_type;
 720:          typedef Policy policy_type;
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 721-744 / 第 721-744 行
~~~cpp
 721: 
 722:          BOOST_MATH_GPU_ENABLED non_central_beta_distribution(RealType a_, RealType b_, RealType lambda) : a(a_), b(b_), ncp(lambda)
 723:          {
 724:             const char* function = "boost::math::non_central_beta_distribution<%1%>::non_central_beta_distribution(%1%,%1%)";
 725:             RealType r;
 726:             beta_detail::check_alpha(
 727:                function,
 728:                a, &r, Policy());
 729:             beta_detail::check_beta(
 730:                function,
 731:                b, &r, Policy());
 732:             detail::check_non_centrality(
 733:                function,
 734:                lambda,
 735:                &r,
 736:                Policy());
 737:          } // non_central_beta_distribution constructor.
 738: 
 739:          BOOST_MATH_GPU_ENABLED RealType alpha() const
 740:          { // Private data getter function.
 741:             return a;
 742:          }
 743:          BOOST_MATH_GPU_ENABLED RealType beta() const
 744:          { // Private data getter function.
~~~
- **EN:** This range declares or defines callable logic such as Policy. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 745-768 / 第 745-768 行
~~~cpp
 745:             return b;
 746:          }
 747:          BOOST_MATH_GPU_ENABLED RealType non_centrality() const
 748:          { // Private data getter function.
 749:             return ncp;
 750:          }
 751:       private:
 752:          // Data member, initialized by constructor.
 753:          RealType a;   // alpha.
 754:          RealType b;   // beta.
 755:          RealType ncp; // non-centrality parameter
 756:       }; // template <class RealType, class Policy> class non_central_beta_distribution
 757: 
 758:       typedef non_central_beta_distribution<double> non_central_beta; // Reserved name of type double.
 759: 
 760:       #ifdef __cpp_deduction_guides
 761:       template <class RealType>
 762:       non_central_beta_distribution(RealType,RealType,RealType)->non_central_beta_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 763:       #endif
 764: 
 765:       // Non-member functions to give properties of the distribution.
 766: 
 767:       template <class RealType, class Policy>
 768:       BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const non_central_beta_distribution<RealType, Policy>& /* dist */)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 769-792 / 第 769-792 行
~~~cpp
 769:       { // Range of permissible values for random variable k.
 770:          using boost::math::tools::max_value;
 771:          return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), static_cast<RealType>(1));
 772:       }
 773: 
 774:       template <class RealType, class Policy>
 775:       BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const non_central_beta_distribution<RealType, Policy>& /* dist */)
 776:       { // Range of supported values for random variable k.
 777:          // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 778:          using boost::math::tools::max_value;
 779:          return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), static_cast<RealType>(1));
 780:       }
 781: 
 782:       template <class RealType, class Policy>
 783:       BOOST_MATH_GPU_ENABLED inline RealType mode(const non_central_beta_distribution<RealType, Policy>& dist)
 784:       { // mode.
 785:          constexpr auto function = "mode(non_central_beta_distribution<%1%> const&)";
 786: 
 787:          RealType a = dist.alpha();
 788:          RealType b = dist.beta();
 789:          RealType l = dist.non_centrality();
 790:          RealType r;
 791:          if(!beta_detail::check_alpha(
 792:                function,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as alpha, beta, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 alpha, beta, ...。

### Lines 793-816 / 第 793-816 行
~~~cpp
 793:                a, &r, Policy())
 794:                ||
 795:             !beta_detail::check_beta(
 796:                function,
 797:                b, &r, Policy())
 798:                ||
 799:             !detail::check_non_centrality(
 800:                function,
 801:                l,
 802:                &r,
 803:                Policy()))
 804:                   return static_cast<RealType>(r);
 805:          RealType c = a + b + l / 2;
 806:          RealType mean = 1 - (b / c) * (1 + l / (2 * c * c));
 807:          return detail::generic_find_mode_01(
 808:             dist,
 809:             mean,
 810:             function);
 811:       }
 812: 
 813:       //
 814:       // We don't have the necessary information to implement
 815:       // these at present.  These are just disabled for now,
 816:       // prototypes retained so we can fill in the blanks
~~~
- **EN:** Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 817-840 / 第 817-840 行
~~~cpp
 817:       // later:
 818:       //
 819:       template <class RealType, class Policy>
 820:       BOOST_MATH_GPU_ENABLED inline RealType mean(const non_central_beta_distribution<RealType, Policy>& dist)
 821:       {
 822:          BOOST_MATH_STD_USING
 823:          RealType a = dist.alpha();
 824:          RealType b = dist.beta();
 825:          RealType d = dist.non_centrality();
 826:          RealType apb = a + b;
 827:          return exp(-d / 2) * a * detail::hypergeometric_2F2<RealType, Policy>(1 + a, apb, a, 1 + apb, d / 2, Policy()) / apb;
 828:       } // mean
 829: 
 830:       template <class RealType, class Policy>
 831:       BOOST_MATH_GPU_ENABLED inline RealType variance(const non_central_beta_distribution<RealType, Policy>& dist)
 832:       {
 833:          //
 834:          // Relative error of this function may be arbitrarily large... absolute
 835:          // error will be small however... that's the best we can do for now.
 836:          //
 837:          BOOST_MATH_STD_USING
 838:          RealType a = dist.alpha();
 839:          RealType b = dist.beta();
 840:          RealType d = dist.non_centrality();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as alpha, beta, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 alpha, beta, ...。

### Lines 841-864 / 第 841-864 行
~~~cpp
 841:          RealType apb = a + b;
 842:          RealType result = detail::hypergeometric_2F2(RealType(1 + a), apb, a, RealType(1 + apb), RealType(d / 2), Policy());
 843:          result *= result * -exp(-d) * a * a / (apb * apb);
 844:          result += exp(-d / 2) * a * (1 + a) * detail::hypergeometric_2F2(RealType(2 + a), apb, a, RealType(2 + apb), RealType(d / 2), Policy()) / (apb * (1 + apb));
 845:          return result;
 846:       }
 847: 
 848:       // RealType standard_deviation(const non_central_beta_distribution<RealType, Policy>& dist)
 849:       // standard_deviation provided by derived accessors.
 850:       template <class RealType, class Policy>
 851:       BOOST_MATH_GPU_ENABLED inline RealType skewness(const non_central_beta_distribution<RealType, Policy>& /*dist*/)
 852:       { // skewness = sqrt(l).
 853:          const char* function = "boost::math::non_central_beta_distribution<%1%>::skewness()";
 854:          typedef typename Policy::assert_undefined_type assert_type;
 855:          static_assert(assert_type::value == 0, "The Non Central Beta Distribution has no skewness.");
 856: 
 857:          return policies::raise_evaluation_error<RealType>(function, "This function is not yet implemented, the only sensible result is %1%.", // LCOV_EXCL_LINE
 858:             boost::math::numeric_limits<RealType>::quiet_NaN(), Policy()); // infinity?  LCOV_EXCL_LINE
 859:       }
 860: 
 861:       template <class RealType, class Policy>
 862:       BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const non_central_beta_distribution<RealType, Policy>& /*dist*/)
 863:       {
 864:          const char* function = "boost::math::non_central_beta_distribution<%1%>::kurtosis_excess()";
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as detail::hypergeometric_2F2, exp, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 detail::hypergeometric_2F2, exp, ...。

### Lines 865-888 / 第 865-888 行
~~~cpp
 865:          typedef typename Policy::assert_undefined_type assert_type;
 866:          static_assert(assert_type::value == 0, "The Non Central Beta Distribution has no kurtosis excess.");
 867: 
 868:          return policies::raise_evaluation_error<RealType>(function, "This function is not yet implemented, the only sensible result is %1%.", // LCOV_EXCL_LINE
 869:             boost::math::numeric_limits<RealType>::quiet_NaN(), Policy()); // infinity?  LCOV_EXCL_LINE
 870:       } // kurtosis_excess
 871: 
 872:       template <class RealType, class Policy>
 873:       BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const non_central_beta_distribution<RealType, Policy>& dist)
 874:       {
 875:          return kurtosis_excess(dist) + 3;
 876:       }
 877: 
 878:       template <class RealType, class Policy>
 879:       BOOST_MATH_GPU_ENABLED inline RealType pdf(const non_central_beta_distribution<RealType, Policy>& dist, const RealType& x)
 880:       { // Probability Density/Mass Function.
 881:          return detail::nc_beta_pdf(dist, x);
 882:       } // pdf
 883: 
 884:       template <class RealType, class Policy>
 885:       BOOST_MATH_GPU_ENABLED RealType cdf(const non_central_beta_distribution<RealType, Policy>& dist, const RealType& x)
 886:       {
 887:          const char* function = "boost::math::non_central_beta_distribution<%1%>::cdf(%1%)";
 888:             RealType a = dist.alpha();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, quiet_NaN, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, quiet_NaN, ...。

### Lines 889-912 / 第 889-912 行
~~~cpp
 889:             RealType b = dist.beta();
 890:             RealType l = dist.non_centrality();
 891:             RealType r;
 892:             if(!beta_detail::check_alpha(
 893:                function,
 894:                a, &r, Policy())
 895:                ||
 896:             !beta_detail::check_beta(
 897:                function,
 898:                b, &r, Policy())
 899:                ||
 900:             !detail::check_non_centrality(
 901:                function,
 902:                l,
 903:                &r,
 904:                Policy())
 905:                ||
 906:             !beta_detail::check_x(
 907:                function,
 908:                x,
 909:                &r,
 910:                Policy()))
 911:                   return static_cast<RealType>(r);
 912: 
~~~
- **EN:** This range declares or defines callable logic such as beta, non_centrality. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 beta, non_centrality。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 913-936 / 第 913-936 行
~~~cpp
 913:          if(l == 0)
 914:             return cdf(beta_distribution<RealType, Policy>(a, b), x);
 915: 
 916:          return detail::non_central_beta_cdf(x, RealType(1 - x), a, b, l, false, Policy());
 917:       } // cdf
 918: 
 919:       template <class RealType, class Policy>
 920:       BOOST_MATH_GPU_ENABLED RealType cdf(const complemented2_type<non_central_beta_distribution<RealType, Policy>, RealType>& c)
 921:       { // Complemented Cumulative Distribution Function
 922:          const char* function = "boost::math::non_central_beta_distribution<%1%>::cdf(%1%)";
 923:          non_central_beta_distribution<RealType, Policy> const& dist = c.dist;
 924:             RealType a = dist.alpha();
 925:             RealType b = dist.beta();
 926:             RealType l = dist.non_centrality();
 927:             RealType x = c.param;
 928:             RealType r;
 929:             if(!beta_detail::check_alpha(
 930:                function,
 931:                a, &r, Policy())
 932:                ||
 933:             !beta_detail::check_beta(
 934:                function,
 935:                b, &r, Policy())
 936:                ||
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as alpha, beta, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 alpha, beta, ...。

### Lines 937-960 / 第 937-960 行
~~~cpp
 937:             !detail::check_non_centrality(
 938:                function,
 939:                l,
 940:                &r,
 941:                Policy())
 942:                ||
 943:             !beta_detail::check_x(
 944:                function,
 945:                x,
 946:                &r,
 947:                Policy()))
 948:                   return static_cast<RealType>(r);
 949: 
 950:          if(l == 0)
 951:             return cdf(complement(beta_distribution<RealType, Policy>(a, b), x));
 952: 
 953:          return detail::non_central_beta_cdf(x, RealType(1 - x), a, b, l, true, Policy());
 954:       } // ccdf
 955: 
 956:       template <class RealType, class Policy>
 957:       BOOST_MATH_GPU_ENABLED inline RealType quantile(const non_central_beta_distribution<RealType, Policy>& dist, const RealType& p)
 958:       { // Quantile (or Percent Point) function.
 959:          return detail::nc_beta_quantile(dist, p, false);
 960:       } // quantile
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 961-977 / 第 961-977 行
~~~cpp
 961: 
 962:       template <class RealType, class Policy>
 963:       BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<non_central_beta_distribution<RealType, Policy>, RealType>& c)
 964:       { // Quantile (or Percent Point) function.
 965:          return detail::nc_beta_quantile(c.dist, c.param, true);
 966:       } // quantile complement.
 967: 
 968:    } // namespace math
 969: } // namespace boost
 970: 
 971: // This include must be at the end, *after* the accessors
 972: // for this distribution have been defined, in order to
 973: // keep compilers that support two-phase lookup happy.
 974: #include <boost/math/distributions/detail/derived_accessors.hpp>
 975: 
 976: #endif // BOOST_MATH_SPECIAL_NON_CENTRAL_BETA_HPP
 977: 
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp so the surrounding code can use external declarations. The code enters namespace scope (math::boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/numeric_limits.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/beta.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/beta.hpp, boost/math/distributions/detail/generic_mode.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/special_functions/fpclassify.hpp, boost/math/special_functions/trunc.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `lltrunc, gamma_p_derivative, detail::ibeta_imp, poisf, exp, cdf, dist, T, ...`
