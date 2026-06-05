# non_central_t.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/non_central_t.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the non central t distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 non central t 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32 / 第 1-32 行
~~~cpp
   1: // boost\math\distributions\non_central_t.hpp
   2: 
   3: // Copyright John Maddock 2008.
   4: 
   5: // Use, modification and distribution are subject to the
   6: // Boost Software License, Version 1.0.
   7: // (See accompanying file LICENSE_1_0.txt
   8: // or copy at http://www.boost.org/LICENSE_1_0.txt)
   9: 
  10: #ifndef BOOST_MATH_SPECIAL_NON_CENTRAL_T_HPP
  11: #define BOOST_MATH_SPECIAL_NON_CENTRAL_T_HPP
  12: 
  13: #include <boost/math/distributions/fwd.hpp>
  14: #include <boost/math/distributions/non_central_beta.hpp> // for nc beta
  15: #include <boost/math/distributions/normal.hpp> // for normal CDF and quantile
  16: #include <boost/math/distributions/students_t.hpp>
  17: #include <boost/math/distributions/detail/generic_quantile.hpp> // quantile
  18: #include <boost/math/special_functions/trunc.hpp>
  19: #include <boost/math/special_functions/detail/hypergeometric_series.hpp>
  20: #include <boost/math/quadrature/exp_sinh.hpp>
  21: 
  22: namespace boost
  23: {
  24:    namespace math
  25:    {
  26: 
  27:       template <class RealType, class Policy>
  28:       class non_central_t_distribution;
  29: 
  30:       namespace detail{
  31: 
  32:          template <class T, class Policy>
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/fwd.hpp, boost/math/distributions/non_central_beta.hpp, boost/math/distributions/normal.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/distributions/fwd.hpp, boost/math/distributions/non_central_beta.hpp, boost/math/distributions/normal.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 33-64 / 第 33-64 行
~~~cpp
  33:          T non_central_t2_p(T v, T delta, T x, T y, const Policy& pol, T init_val)
  34:          {
  35:             BOOST_MATH_STD_USING
  36:             //
  37:             // Variables come first:
  38:             //
  39:             std::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
  40:             T errtol = policies::get_epsilon<T, Policy>();
  41:             T d2 = delta * delta / 2;
  42:             //
  43:             // k is the starting point for iteration, and is the
  44:             // maximum of the poisson weighting term, we don't
  45:             // ever allow k == 0 as this can lead to catastrophic
  46:             // cancellation errors later (test case is v = 1621286869049072.3
  47:             // delta = 0.16212868690490723, x = 0.86987415482475994).
  48:             //
  49:             long long k = lltrunc(d2);
  50:             T pois;
  51:             if(k == 0) k = 1;
  52:             // Starting Poisson weight:
  53:             pois = gamma_p_derivative(T(k+1), d2, pol)
  54:                * tgamma_delta_ratio(T(k + 1), T(0.5f))
  55:                * delta / constants::root_two<T>();
  56:             if(pois == 0)
  57:                return init_val;
  58:             T xterm, beta;
  59:             // Recurrence & starting beta terms:
  60:             beta = x < y
  61:                ? detail::ibeta_imp(T(k + 1), T(v / 2), x, pol, false, true, &xterm)
  62:                : detail::ibeta_imp(T(v / 2), T(k + 1), y, pol, true, true, &xterm);
  63: 
  64:             while (fabs(beta * pois) < tools::min_value<T>())
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, detail::ibeta_imp. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, detail::ibeta_imp。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 65-96 / 第 65-96 行
~~~cpp
  65:             {
  66:                if ((k == 0) || (pois == 0))
  67:                   return init_val;
  68:                k /= 2;
  69:                pois = gamma_p_derivative(T(k + 1), d2, pol)
  70:                   * tgamma_delta_ratio(T(k + 1), T(0.5f))
  71:                   * delta / constants::root_two<T>();
  72:                beta = x < y
  73:                   ? detail::ibeta_imp(T(k + 1), T(v / 2), x, pol, false, true, &xterm)
  74:                   : detail::ibeta_imp(T(v / 2), T(k + 1), y, pol, true, true, &xterm);
  75:             }
  76: 
  77:             xterm *= y / (v / 2 + k);
  78:             T poisf(pois), betaf(beta), xtermf(xterm);
  79:             T sum = init_val;
  80:             if((xterm == 0) && (beta == 0))
  81:                return init_val;
  82: 
  83:             //
  84:             // Backwards recursion first, this is the stable
  85:             // direction for recursion:
  86:             //
  87:             std::uintmax_t count = 0;
  88:             T last_term = 0;
  89:             for(auto i = k; i >= 0; --i)
  90:             {
  91:                T term = beta * pois;
  92:                sum += term;
  93:                // Don't terminate on first term in case we "fixed" k above:
  94:                if(((fabs(last_term) > fabs(term)) && fabs(term/sum) < errtol) || (v == 2 && i == 0))
  95:                   break;
  96:                last_term = term;
~~~
- **EN:** This range declares or defines callable logic such as detail::ibeta_imp, poisf. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::ibeta_imp, poisf。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 97-128 / 第 97-128 行
~~~cpp
  97:                pois *= (i + 0.5f) / d2;
  98:                beta += xterm;
  99:                xterm *= (i) / (x * (v / 2 + i - 1));
 100:                ++count;
 101:             }
 102:             last_term = 0;
 103:             for(auto i = k + 1; ; ++i)
 104:             {
 105:                poisf *= d2 / (i + 0.5f);
 106:                xtermf *= (x * (v / 2 + i - 1)) / (i);
 107:                betaf -= xtermf;
 108:                T term = poisf * betaf;
 109:                sum += term;
 110:                if((fabs(last_term) >= fabs(term)) && (fabs(term/sum) < errtol))
 111:                   break;
 112:                last_term = term;
 113:                ++count;
 114:                if(count > max_iter)
 115:                {
 116:                   return policies::raise_evaluation_error("cdf(non_central_t_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 117:                }
 118:             }
 119:             return sum;
 120:          }
 121: 
 122:          template <class T, class Policy>
 123:          T non_central_t2_q(T v, T delta, T x, T y, const Policy& pol, T init_val)
 124:          {
 125:             BOOST_MATH_STD_USING
 126:             //
 127:             // Variables come first:
 128:             //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 129-160 / 第 129-160 行
~~~cpp
 129:             std::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 130:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
 131:             T d2 = delta * delta / 2;
 132:             //
 133:             // k is the starting point for iteration, and is the
 134:             // maximum of the poisson weighting term, we don't allow
 135:             // k == 0 as this can cause catastrophic cancellation errors
 136:             // (test case is v = 561908036470413.25, delta = 0.056190803647041321,
 137:             // x = 1.6155232703966216):
 138:             //
 139:             long long k = lltrunc(d2);
 140:             if(k == 0) k = 1;
 141:             // Starting Poisson weight:
 142:             T pois;
 143:             if((k < static_cast<long long>(max_factorial<T>::value)) && (d2 < tools::log_max_value<T>()) && (log(d2) * k < tools::log_max_value<T>()))
 144:             {
 145:                //
 146:                // For small k we can optimise this calculation by using
 147:                // a simpler reduced formula:
 148:                //
 149:                pois = exp(-d2);
 150:                pois *= pow(d2, static_cast<T>(k));
 151:                pois /= boost::math::tgamma(T(k + 1 + 0.5), pol);
 152:                pois *= delta / constants::root_two<T>();
 153:             }
 154:             else
 155:             {
 156:                pois = gamma_p_derivative(T(k+1), d2, pol)
 157:                   * tgamma_delta_ratio(T(k + 1), T(0.5f))
 158:                   * delta / constants::root_two<T>();
 159:             }
 160:             if(pois == 0)
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, exp, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, exp, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 161-192 / 第 161-192 行
~~~cpp
 161:                return init_val;
 162:             // Recurance term:
 163:             T xterm;
 164:             T beta;
 165:             // Starting beta term:
 166:             if(k != 0)
 167:             {
 168:                beta = x < y
 169:                   ? detail::ibeta_imp(T(k + 1), T(v / 2), x, pol, true, true, &xterm)
 170:                   : detail::ibeta_imp(T(v / 2), T(k + 1), y, pol, false, true, &xterm);
 171: 
 172:                xterm *= y / (v / 2 + k);
 173:             }
 174:             else
 175:             {
 176:                beta = pow(y, v / 2);
 177:                xterm = beta;
 178:             }
 179:             T poisf(pois), betaf(beta), xtermf(xterm);
 180:             T sum = init_val;
 181:             if((xterm == 0) && (beta == 0))
 182:                return init_val;
 183: 
 184:             //
 185:             // Fused forward and backwards recursion:
 186:             //
 187:             std::uintmax_t count = 0;
 188:             T last_term = 0;
 189:             for(auto i = k + 1, j = k; ; ++i, --j)
 190:             {
 191:                poisf *= d2 / (i + 0.5f);
 192:                xtermf *= (x * (v / 2 + i - 1)) / (i);
~~~
- **EN:** This range declares or defines callable logic such as detail::ibeta_imp, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::ibeta_imp, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 193-224 / 第 193-224 行
~~~cpp
 193:                betaf += xtermf;
 194:                T term = poisf * betaf;
 195: 
 196:                if(j >= 0)
 197:                {
 198:                   term += beta * pois;
 199:                   pois *= (j + 0.5f) / d2;
 200:                   beta -= xterm;
 201:                   if(!(v == 2 && j == 0))
 202:                      xterm *= (j) / (x * (v / 2 + j - 1));
 203:                }
 204: 
 205:                sum += term;
 206:                // Don't terminate on first term in case we "fixed" the value of k above:
 207:                if((fabs(last_term) > fabs(term)) && fabs(term/sum) < errtol)
 208:                   break;
 209:                last_term = term;
 210:                if(count > max_iter)
 211:                {
 212:                   return policies::raise_evaluation_error("cdf(non_central_t_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 213:                }
 214:                ++count;
 215:             }
 216:             return sum;
 217:          }
 218: 
 219:          template <class T, class Policy>
 220:          T non_central_t_cdf(T v, T delta, T t, bool invert, const Policy& pol)
 221:          {
 222:             BOOST_MATH_STD_USING
 223:             if ((boost::math::isinf)(v))
 224:             { // Infinite degrees of freedom, so use normal distribution located at delta.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 225-256 / 第 225-256 行
~~~cpp
 225:                normal_distribution<T, Policy> n(delta, 1);
 226:                return cdf(n, t);
 227:             }
 228:             //
 229:             // Otherwise, for t < 0 we have to use the reflection formula:
 230:             if(t < 0)
 231:             {
 232:                t = -t;
 233:                delta = -delta;
 234:                invert = !invert;
 235:             }
 236:             if(fabs(delta / (4 * v)) < policies::get_epsilon<T, Policy>())
 237:             {
 238:                // Approximate with a Student's T centred on delta,
 239:                // the crossover point is based on eq 2.6 from
 240:                // "A Comparison of Approximations To Percentiles of the
 241:                // Noncentral t-Distribution".  H. Sahai and M. M. Ojeda,
 242:                // Revista Investigacion Operacional Vol 21, No 2, 2000.
 243:                // Original sources referenced in the above are:
 244:                // "Some Approximations to the Percentage Points of the Noncentral
 245:                // t-Distribution". C. van Eeden. International Statistical Review, 29, 4-31.
 246:                // "Continuous Univariate Distributions".  N.L. Johnson, S. Kotz and
 247:                // N. Balkrishnan. 1995. John Wiley and Sons New York.
 248:                T result = cdf(students_t_distribution<T, Policy>(v), t - delta);
 249:                return invert ? 1 - result : result;
 250:             }
 251:             //
 252:             // x and y are the corresponding random
 253:             // variables for the noncentral beta distribution,
 254:             // with y = 1 - x:
 255:             //
 256:             T x = t * t / (v + t * t);
~~~
- **EN:** This range declares or defines callable logic such as n, cdf. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 n, cdf。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 257-288 / 第 257-288 行
~~~cpp
 257:             T y = v / (v + t * t);
 258:             T d2 = delta * delta;
 259:             T a = 0.5f;
 260:             T b = v / 2;
 261:             T c = a + b + d2 / 2;
 262:             //
 263:             // Crossover point for calculating p or q is the same
 264:             // as for the noncentral beta:
 265:             //
 266:             T cross = 1 - (b / c) * (1 + d2 / (2 * c * c));
 267:             T result;
 268:             if(x < cross)
 269:             {
 270:                //
 271:                // Calculate p:
 272:                //
 273:                if(x != 0)
 274:                {
 275:                   result = non_central_beta_p(a, b, d2, x, y, pol);
 276:                   result = non_central_t2_p(v, delta, x, y, pol, result);
 277:                   result /= 2;
 278:                }
 279:                else
 280:                   result = 0;
 281:                if (invert)
 282:                {
 283:                   result = cdf(complement(boost::math::normal_distribution<T, Policy>(), -delta)) - result;
 284:                   invert = false;
 285:                }
 286:                else
 287:                   result += cdf(boost::math::normal_distribution<T, Policy>(), -delta);
 288:             }
~~~
- **EN:** This range declares or defines callable logic such as non_central_beta_p, non_central_t2_p, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 non_central_beta_p, non_central_t2_p, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 289-320 / 第 289-320 行
~~~cpp
 289:             else
 290:             {
 291:                //
 292:                // Calculate q:
 293:                //
 294:                invert = !invert;
 295:                if(x != 0)
 296:                {
 297:                   result = non_central_beta_q(a, b, d2, x, y, pol);
 298:                   result = non_central_t2_q(v, delta, x, y, pol, result);
 299:                   result /= 2;
 300:                }
 301:                else // x == 0
 302:                   result = cdf(complement(boost::math::normal_distribution<T, Policy>(), -delta));
 303:             }
 304:             if(invert)
 305:                result = 1 - result;
 306:             return result;
 307:          }
 308: 
 309:          template <class T, class Policy>
 310:          T non_central_t_quantile(const char* function, T v, T delta, T p, T q, const Policy&)
 311:          {
 312:             BOOST_MATH_STD_USING
 313:      //       static const char* function = "quantile(non_central_t_distribution<%1%>, %1%)";
 314:      // now passed as function
 315:             typedef typename policies::evaluation<T, Policy>::type value_type;
 316:             typedef typename policies::normalise<
 317:                Policy,
 318:                policies::promote_float<false>,
 319:                policies::promote_double<false>,
 320:                policies::discrete_quantile<>,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as non_central_beta_q, non_central_t2_q, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 non_central_beta_q, non_central_t2_q, ...。

### Lines 321-352 / 第 321-352 行
~~~cpp
 321:                policies::assert_undefined<> >::type forwarding_policy;
 322: 
 323:                T r;
 324:                if(!detail::check_df_gt0_to_inf(
 325:                   function,
 326:                   v, &r, Policy())
 327:                   ||
 328:                !detail::check_non_centrality(
 329:                   function,
 330:                   static_cast<T>(delta * delta),
 331:                   &r,
 332:                   Policy())
 333:                   ||
 334:                !detail::check_probability(
 335:                   function,
 336:                   p,
 337:                   &r,
 338:                   Policy()))
 339:                      return r;
 340: 
 341: 
 342:             value_type guess = 0;
 343:             if ( ((boost::math::isinf)(v)) || (v > 1 / boost::math::tools::epsilon<T>()) )
 344:             { // Infinite or very large degrees of freedom, so use normal distribution located at delta.
 345:                normal_distribution<T, Policy> n(delta, 1);
 346:                if (p < q)
 347:                {
 348:                  return quantile(n, p);
 349:                }
 350:                else
 351:                {
 352:                  return quantile(complement(n, q));
~~~
- **EN:** This range declares or defines callable logic such as n. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 n。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 353-384 / 第 353-384 行
~~~cpp
 353:                }
 354:             }
 355:             else if(v > 3)
 356:             { // Use normal distribution to calculate guess.
 357:                value_type mean = (v > 1 / policies::get_epsilon<T, Policy>()) ? delta : delta * sqrt(v / 2) * tgamma_delta_ratio((v - 1) * 0.5f, T(0.5f));
 358:                value_type var = (v > 1 / policies::get_epsilon<T, Policy>()) ? value_type(1) : (((delta * delta + 1) * v) / (v - 2) - mean * mean);
 359:                if(p < q)
 360:                   guess = quantile(normal_distribution<value_type, forwarding_policy>(mean, var), p);
 361:                else
 362:                   guess = quantile(complement(normal_distribution<value_type, forwarding_policy>(mean, var), q));
 363:             }
 364:             //
 365:             // We *must* get the sign of the initial guess correct,
 366:             // or our root-finder will fail, so double check it now:
 367:             //
 368:             value_type pzero = non_central_t_cdf(
 369:                static_cast<value_type>(v),
 370:                static_cast<value_type>(delta),
 371:                static_cast<value_type>(0),
 372:                !(p < q),
 373:                forwarding_policy());
 374:             int s;
 375:             if(p < q)
 376:                s = boost::math::sign(p - pzero);
 377:             else
 378:                s = boost::math::sign(pzero - q);
 379:             if(s != boost::math::sign(guess))
 380:             {
 381:                guess = static_cast<T>(s);
 382:             }
 383: 
 384:             value_type result = detail::generic_quantile(
~~~
- **EN:** This range declares or defines callable logic such as sqrt, value_type, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, value_type, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 385-416 / 第 385-416 行
~~~cpp
 385:                non_central_t_distribution<value_type, forwarding_policy>(v, delta),
 386:                (p < q ? p : q),
 387:                guess,
 388:                (p >= q),
 389:                function);
 390:             return policies::checked_narrowing_cast<T, forwarding_policy>(
 391:                result,
 392:                function);
 393:          }
 394: 
 395:          template <class T, class Policy>
 396:          T non_central_t_pdf_integral(T x, T v, T mu, const Policy& pol)
 397:          {
 398:             BOOST_MATH_STD_USING
 399:             boost::math::quadrature::exp_sinh<T, Policy> integrator;
 400:             T integral = pow(v, v / 2) * exp(-v * mu * mu / (2 * (x * x + v)));
 401:             if (integral != 0)
 402:             {
 403:                integral *= integrator.integrate([&x, v, mu](T y)
 404:                   {
 405:                      T p;
 406:                      if (v * log(y) < tools::log_max_value<T>())
 407:                         p = pow(y, v) * exp(boost::math::pow<2>((y - mu * x / sqrt(x * x + v))) / -2);
 408:                      else
 409:                         p = exp(log(y) * v + boost::math::pow<2>((y - mu * x / sqrt(x * x + v))) / -2);
 410:                      return p;
 411:                   });
 412:             }
 413:             integral /= boost::math::constants::root_pi<T>() * boost::math::tgamma(v / 2, pol) * pow(T(2), (v - 1) / 2) * pow(x * x + v, (v + 1) / 2);
 414:             return integral;
 415:          }
 416: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as pow, exp, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 pow, exp, ...。

### Lines 417-448 / 第 417-448 行
~~~cpp
 417:          template <class T, class Policy>
 418:          T non_central_t_pdf_hypergeometric(T x, T v, T mu, const Policy& pol)
 419:          {
 420:             BOOST_MATH_STD_USING
 421:             long long scale = 0;
 422:             const char* function = "non central T PDF";
 423:             //
 424:             // We only call this routine when we know that the series form of 1F1 is cheap to evaluate,
 425:             // so no need to call the whole 1F1 function, just the series will do:
 426:             //
 427:             T Av = hypergeometric_1F1_generic_series(static_cast<T>((v + 1) / 2), boost::math::constants::half<T>(), static_cast<T>(mu * mu * x * x / (2 * (x * x + v))), pol, scale, function);
 428:             Av = ldexp(Av, static_cast<int>(scale));
 429:             scale = 0;
 430:             T Bv = hypergeometric_1F1_generic_series(static_cast<T>(v / 2 + T(1)), static_cast<T>(T(3) / 2), static_cast<T>(mu * mu * x * x / (2 * (x * x + v))), pol, scale, function);
 431:             Bv = ldexp(Bv, static_cast<int>(scale));
 432:             Bv *= boost::math::tgamma_delta_ratio(v / 2 + T(1), -constants::half<T>(), pol);
 433:             Bv *= boost::math::constants::root_two<T>() * mu * x / sqrt(x * x + v);
 434: 
 435:             T tolerance = tools::root_epsilon<T>() * Av * 4;
 436:             Av += Bv;
 437: 
 438:             if (Av < tolerance)
 439:             {
 440:                // More than half the digits have cancelled, fall back to the integral method:
 441:                return non_central_t_pdf_integral(x, v, mu, pol);
 442:             }
 443: 
 444:             Av *= exp(-mu * mu / 2) * pow(1 + x * x / v, -(v + 1) / 2) * boost::math::tgamma_delta_ratio(v / 2 + constants::half<T>(), -constants::half<T>(), pol);
 445:             Av /= sqrt(v) * boost::math::constants::root_pi<T>();
 446:             return Av;
 447:          }
 448: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as hypergeometric_1F1_generic_series, ldexp, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 hypergeometric_1F1_generic_series, ldexp, ...。

### Lines 449-480 / 第 449-480 行
~~~cpp
 449:          template <class T, class Policy>
 450:          T non_central_t2_pdf(T n, T delta, T x, T y, const Policy& pol, T init_val)
 451:          {
 452:             BOOST_MATH_STD_USING
 453:             //
 454:             // Variables come first:
 455:             //
 456:             std::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 457:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
 458:             T d2 = delta * delta / 2;
 459:             //
 460:             // k is the starting point for iteration, and is the
 461:             // maximum of the poisson weighting term:
 462:             //
 463:             long long k = lltrunc(d2);
 464:             T pois, xterm;
 465:             if(k == 0)
 466:                k = 1;
 467:             // Starting Poisson weight:
 468:             pois = gamma_p_derivative(T(k+1), d2, pol)
 469:                * tgamma_delta_ratio(T(k + 1), T(0.5f))
 470:                * delta / constants::root_two<T>();
 471:             BOOST_MATH_INSTRUMENT_VARIABLE(pois);
 472:             // Starting beta term:
 473:             xterm = x < y
 474:                ? ibeta_derivative(T(k + 1), n / 2, x, pol)
 475:                : ibeta_derivative(n / 2, T(k + 1), y, pol);
 476:             BOOST_MATH_INSTRUMENT_VARIABLE(xterm);
 477: 
 478:             while (fabs(xterm * pois) < tools::min_value<T>())
 479:             {
 480:                if (k == 0)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as lltrunc, BOOST_MATH_INSTRUMENT_VARIABLE, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 lltrunc, BOOST_MATH_INSTRUMENT_VARIABLE, ...。

### Lines 481-512 / 第 481-512 行
~~~cpp
 481:                   return init_val;
 482:                k /= 2;
 483:                pois = gamma_p_derivative(T(k + 1), d2, pol)
 484:                   * tgamma_delta_ratio(T(k + 1), T(0.5f))
 485:                   * delta / constants::root_two<T>();
 486:                BOOST_MATH_INSTRUMENT_VARIABLE(pois);
 487:                // Starting beta term:
 488:                xterm = x < y
 489:                   ? ibeta_derivative(T(k + 1), n / 2, x, pol)
 490:                   : ibeta_derivative(n / 2, T(k + 1), y, pol);
 491:                BOOST_MATH_INSTRUMENT_VARIABLE(xterm);
 492:             }
 493: 
 494:             T poisf(pois), xtermf(xterm);
 495:             T sum = init_val;
 496: 
 497:             //
 498:             // Backwards recursion first, this is the stable
 499:             // direction for recursion:
 500:             //
 501:             std::uintmax_t count = 0;
 502:             T old_ratio = 1;  // arbitrary "large" value
 503:             for(auto i = k; i >= 0; --i)
 504:             {
 505:                T term = xterm * pois;
 506:                sum += term;
 507:                BOOST_MATH_INSTRUMENT_VARIABLE(sum);
 508:                T ratio = fabs(term / sum);
 509:                if(((ratio < errtol) && (i != k) && (ratio < old_ratio)) || (term == 0))
 510:                   break;
 511:                old_ratio = ratio;
 512:                pois *= (i + 0.5f) / d2;
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, ibeta_derivative, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, ibeta_derivative, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 513-544 / 第 513-544 行
~~~cpp
 513:                xterm *= (i) / (x * (n / 2 + i));
 514:                ++count;
 515:                if(count > max_iter)
 516:                {
 517:                   return policies::raise_evaluation_error("pdf(non_central_t_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 518:                }
 519:             }
 520:             BOOST_MATH_INSTRUMENT_VARIABLE(sum);
 521:             old_ratio = 0;
 522:             for(auto i = k + 1; ; ++i)
 523:             {
 524:                poisf *= d2 / (i + 0.5f);
 525:                xtermf *= (x * (n / 2 + i)) / (i);
 526:                T term = poisf * xtermf;
 527:                sum += term;
 528:                T ratio = fabs(term / sum);
 529:                if(((ratio < errtol) && (ratio < old_ratio)) || (term == 0))
 530:                   break;
 531:                ++count;
 532:                old_ratio = ratio;
 533:                if(count > max_iter)
 534:                {
 535:                   return policies::raise_evaluation_error("pdf(non_central_t_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 536:                }
 537:             }
 538:             BOOST_MATH_INSTRUMENT_VARIABLE(sum);
 539:             return sum;
 540:          }
 541: 
 542:          template <class T, class Policy>
 543:          T non_central_t_pdf(T n, T delta, T t, const Policy& pol)
 544:          {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, fabs.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, fabs。

### Lines 545-576 / 第 545-576 行
~~~cpp
 545:             BOOST_MATH_STD_USING
 546:             if ((boost::math::isinf)(n))
 547:             { // Infinite degrees of freedom, so use normal distribution located at delta.
 548:                normal_distribution<T, Policy> norm(delta, 1);
 549:                return pdf(norm, t);
 550:             }
 551:             if(t * t < tools::epsilon<T>())
 552:             {
 553:                //
 554:                // Handle this as a special case, using the formula
 555:                // from Weisstein, Eric W.
 556:                // "Noncentral Student's t-Distribution."
 557:                // From MathWorld--A Wolfram Web Resource.
 558:                // http://mathworld.wolfram.com/NoncentralStudentst-Distribution.html
 559:                //
 560:                // The formula is simplified thanks to the relation
 561:                // 1F1(a,b,0) = 1.
 562:                //
 563:                return tgamma_delta_ratio(n / 2 + 0.5f, T(0.5f))
 564:                   * sqrt(n / constants::pi<T>())
 565:                   * exp(-delta * delta / 2) / 2;
 566:             }
 567:             if(fabs(delta / (4 * n)) < policies::get_epsilon<T, Policy>())
 568:             {
 569:                // Approximate with a Student's T centred on delta,
 570:                // the crossover point is based on eq 2.6 from
 571:                // "A Comparison of Approximations To Percentiles of the
 572:                // Noncentral t-Distribution".  H. Sahai and M. M. Ojeda,
 573:                // Revista Investigacion Operacional Vol 21, No 2, 2000.
 574:                // Original sources referenced in the above are:
 575:                // "Some Approximations to the Percentage Points of the Noncentral
 576:                // t-Distribution". C. van Eeden. International Statistical Review, 29, 4-31.
~~~
- **EN:** This range declares or defines callable logic such as norm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 norm。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 577-608 / 第 577-608 行
~~~cpp
 577:                // "Continuous Univariate Distributions".  N.L. Johnson, S. Kotz and
 578:                // N. Balkrishnan. 1995. John Wiley and Sons New York.
 579:                return pdf(students_t_distribution<T, Policy>(n), t - delta);
 580:             }
 581:             //
 582:             // Figure out if the hypergeometric formula will be efficient or not,
 583:             // based on where the summit of the series.
 584:             //
 585:             T a = (n + 1) / 2;
 586:             T x = delta * delta * t * t / (2 * (t * t + n));
 587:             T summit = (sqrt(x * (4 * a + x)) + x) / 2;
 588:             if (summit < 40)
 589:                return non_central_t_pdf_hypergeometric(t, n, delta, pol);
 590:             //
 591:             // Otherwise, for t < 0 we have to use the reflection formula:
 592:             //
 593:             if (t < 0)
 594:             {
 595:                t = -t;
 596:                delta = -delta;
 597:             }
 598:             //
 599:             // x and y are the corresponding random
 600:             // variables for the noncentral beta distribution,
 601:             // with y = 1 - x:
 602:             //
 603:             x = t * t / (n + t * t);
 604:             T y = n / (n + t * t);
 605:             a = 0.5f;
 606:             T b = n / 2;
 607:             T d2 = delta * delta;
 608:             //
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 609-640 / 第 609-640 行
~~~cpp
 609:             // Calculate pdf:
 610:             //
 611:             T dt = n * t / (n * n + 2 * n * t * t + t * t * t * t);
 612:             BOOST_MATH_INSTRUMENT_VARIABLE(dt);
 613:             T result = non_central_beta_pdf(a, b, d2, x, y, pol);
 614:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 615:             T tol = tools::root_epsilon<T>() * result;
 616:             result = non_central_t2_pdf(n, delta, x, y, pol, result);
 617:             BOOST_MATH_INSTRUMENT_VARIABLE(result);
 618:             result *= dt;
 619:             if (result <= tol)
 620:             {
 621:                // More than half the digits in the result have cancelled,
 622:                // Try direct integration... super slow but reliable as far as we can tell...
 623:                if (delta < 0)
 624:                {
 625:                   // reflect back:
 626:                   delta = -delta;
 627:                   t = -t;
 628:                }
 629:                result = non_central_t_pdf_integral(t, n, delta, pol);
 630:             }
 631:             return result;
 632:          }
 633: 
 634:          template <class T, class Policy>
 635:          T mean(T v, T delta, const Policy& pol)
 636:          {
 637:             if ((boost::math::isinf)(v))
 638:             {
 639:                return delta;
 640:             }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, non_central_beta_pdf, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, non_central_beta_pdf, ...。

### Lines 641-672 / 第 641-672 行
~~~cpp
 641:             BOOST_MATH_STD_USING
 642:             if (v > 1 / boost::math::tools::epsilon<T>() )
 643:             {
 644:               //normal_distribution<T, Policy> n(delta, 1);
 645:               //return boost::math::mean(n);
 646:               return delta;
 647:             }
 648:             else
 649:             {
 650:              return delta * sqrt(v / 2) * tgamma_delta_ratio((v - 1) * 0.5f, T(0.5f), pol);
 651:             }
 652:             // Other moments use mean so using normal distribution is propagated.
 653:          }
 654: 
 655:          template <class T, class Policy>
 656:          T variance(T v, T delta, const Policy& pol)
 657:          {
 658:             if ((boost::math::isinf)(v))
 659:             {
 660:                return 1;
 661:             }
 662:             if (delta == 0)
 663:             {  // == Student's t
 664:               return v / (v - 2);
 665:             }
 666:             T result = ((delta * delta + 1) * v) / (v - 2);
 667:             T m = mean(v, delta, pol);
 668:             result -= m * m;
 669:             return result;
 670:          }
 671: 
 672:          template <class T, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as n, boost::math::mean, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 n, boost::math::mean, ...。

### Lines 673-704 / 第 673-704 行
~~~cpp
 673:          T skewness(T v, T delta, const Policy& pol)
 674:          {
 675:             BOOST_MATH_STD_USING
 676:             if ((boost::math::isinf)(v))
 677:             {
 678:                return 0;
 679:             }
 680:             if(delta == 0)
 681:             { // == Student's t
 682:               return 0;
 683:             }
 684:             T mean = boost::math::detail::mean(v, delta, pol);
 685:             T l2 = delta * delta;
 686:             T var = ((l2 + 1) * v) / (v - 2) - mean * mean;
 687:             T result = -2 * var;
 688:             result += v * (l2 + 2 * v - 3) / ((v - 3) * (v - 2));
 689:             result *= mean;
 690:             result /= pow(var, T(1.5f));
 691:             return result;
 692:          }
 693: 
 694:          template <class T, class Policy>
 695:          T kurtosis_excess(T v, T delta, const Policy& pol)
 696:          {
 697:             BOOST_MATH_STD_USING
 698:             if ((boost::math::isinf)(v))
 699:             {
 700:                return 1;
 701:             }
 702:             if (delta == 0)
 703:             { // == Student's t
 704:               return 1;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::detail::mean, pow.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::detail::mean, pow。

### Lines 705-736 / 第 705-736 行
~~~cpp
 705:             }
 706:             T mean = boost::math::detail::mean(v, delta, pol);
 707:             T l2 = delta * delta;
 708:             T var = ((l2 + 1) * v) / (v - 2) - mean * mean;
 709:             T result = -3 * var;
 710:             result += v * (l2 * (v + 1) + 3 * (3 * v - 5)) / ((v - 3) * (v - 2));
 711:             result *= -mean * mean;
 712:             result += v * v * (l2 * l2 + 6 * l2 + 3) / ((v - 4) * (v - 2));
 713:             result /= var * var;
 714:             result -= static_cast<T>(3);
 715:             return result;
 716:          }
 717: 
 718: #if 0
 719:          //
 720:          // This code is disabled, since there can be multiple answers to the
 721:          // question, and it's not clear how to find the "right" one.
 722:          //
 723:          template <class RealType, class Policy>
 724:          struct t_degrees_of_freedom_finder
 725:          {
 726:             t_degrees_of_freedom_finder(
 727:                RealType delta_, RealType x_, RealType p_, bool c)
 728:                : delta(delta_), x(x_), p(p_), comp(c) {}
 729: 
 730:             RealType operator()(const RealType& v)
 731:             {
 732:                non_central_t_distribution<RealType, Policy> d(v, delta);
 733:                return comp ?
 734:                   p - cdf(complement(d, x))
 735:                   : cdf(d, x) - p;
 736:             }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::detail::mean, delta, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::detail::mean, delta, ...。

### Lines 737-768 / 第 737-768 行
~~~cpp
 737:          private:
 738:             RealType delta;
 739:             RealType x;
 740:             RealType p;
 741:             bool comp;
 742:          };
 743: 
 744:          template <class RealType, class Policy>
 745:          inline RealType find_t_degrees_of_freedom(
 746:             RealType delta, RealType x, RealType p, RealType q, const Policy& pol)
 747:          {
 748:             const char* function = "non_central_t<%1%>::find_degrees_of_freedom";
 749:             if((p == 0) || (q == 0))
 750:             {
 751:                //
 752:                // Can't a thing if one of p and q is zero:
 753:                //
 754:                return policies::raise_evaluation_error<RealType>(function, "Can't find degrees of freedom when the probability is 0 or 1, only possible answer is %1%", // LCOV_EXCL_LINE
 755:                   RealType(std::numeric_limits<RealType>::quiet_NaN()), Policy()); // LCOV_EXCL_LINE
 756:             }
 757:             t_degrees_of_freedom_finder<RealType, Policy> f(delta, x, p < q ? p : q, p < q ? false : true);
 758:             tools::eps_tolerance<RealType> tol(policies::digits<RealType, Policy>());
 759:             std::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 760:             //
 761:             // Pick an initial guess:
 762:             //
 763:             RealType guess = 200;
 764:             std::pair<RealType, RealType> ir = tools::bracket_and_solve_root(
 765:                f, guess, RealType(2), false, tol, max_iter, pol);
 766:             RealType result = ir.first + (ir.second - ir.first) / 2;
 767:             if(max_iter >= policies::get_max_root_iterations<Policy>())
 768:             {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as RealType, f, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 RealType, f, ...。

### Lines 769-800 / 第 769-800 行
~~~cpp
 769:                return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 770:                   " or there is no answer to problem.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
 771:             }
 772:             return result;
 773:          }
 774: 
 775:          template <class RealType, class Policy>
 776:          struct t_non_centrality_finder
 777:          {
 778:             t_non_centrality_finder(
 779:                RealType v_, RealType x_, RealType p_, bool c)
 780:                : v(v_), x(x_), p(p_), comp(c) {}
 781: 
 782:             RealType operator()(const RealType& delta)
 783:             {
 784:                non_central_t_distribution<RealType, Policy> d(v, delta);
 785:                return comp ?
 786:                   p - cdf(complement(d, x))
 787:                   : cdf(d, x) - p;
 788:             }
 789:          private:
 790:             RealType v;
 791:             RealType x;
 792:             RealType p;
 793:             bool comp;
 794:          };
 795: 
 796:          template <class RealType, class Policy>
 797:          inline RealType find_t_non_centrality(
 798:             RealType v, RealType x, RealType p, RealType q, const Policy& pol)
 799:          {
 800:             const char* function = "non_central_t<%1%>::find_t_non_centrality";
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, v, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, v, ...。

### Lines 801-832 / 第 801-832 行
~~~cpp
 801:             if((p == 0) || (q == 0))
 802:             {
 803:                //
 804:                // Can't do a thing if one of p and q is zero:
 805:                //
 806:                return policies::raise_evaluation_error<RealType>(function, "Can't find non-centrality parameter when the probability is 0 or 1, only possible answer is %1%", // LCOV_EXCL_LINE
 807:                   RealType(std::numeric_limits<RealType>::quiet_NaN()), Policy()); // LCOV_EXCL_LINE
 808:             }
 809:             t_non_centrality_finder<RealType, Policy> f(v, x, p < q ? p : q, p < q ? false : true);
 810:             tools::eps_tolerance<RealType> tol(policies::digits<RealType, Policy>());
 811:             std::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 812:             //
 813:             // Pick an initial guess that we know is the right side of
 814:             // zero:
 815:             //
 816:             RealType guess;
 817:             if(f(0) < 0)
 818:                guess = 1;
 819:             else
 820:                guess = -1;
 821:             std::pair<RealType, RealType> ir = tools::bracket_and_solve_root(
 822:                f, guess, RealType(2), false, tol, max_iter, pol);
 823:             RealType result = ir.first + (ir.second - ir.first) / 2;
 824:             if(max_iter >= policies::get_max_root_iterations<Policy>())
 825:             {
 826:                return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 827:                   " or there is no answer to problem.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
 828:             }
 829:             return result;
 830:          }
 831: #endif
 832:       } // namespace detail ======================================================================
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. This range declares or defines callable logic such as RealType, f, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围声明或定义了可调用逻辑，例如 RealType, f, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 833-864 / 第 833-864 行
~~~cpp
 833: 
 834:       template <class RealType = double, class Policy = policies::policy<> >
 835:       class non_central_t_distribution
 836:       {
 837:       public:
 838:          typedef RealType value_type;
 839:          typedef Policy policy_type;
 840: 
 841:          non_central_t_distribution(RealType v_, RealType lambda) : v(v_), ncp(lambda)
 842:          {
 843:             const char* function = "boost::math::non_central_t_distribution<%1%>::non_central_t_distribution(%1%,%1%)";
 844:             RealType r;
 845:             detail::check_df_gt0_to_inf(
 846:                function,
 847:                v, &r, Policy());
 848:             detail::check_non_centrality(
 849:                function,
 850:                static_cast<value_type>(lambda * lambda),
 851:                &r,
 852:                Policy());
 853:          } // non_central_t_distribution constructor.
 854: 
 855:          RealType degrees_of_freedom() const
 856:          { // Private data getter function.
 857:             return v;
 858:          }
 859:          RealType non_centrality() const
 860:          { // Private data getter function.
 861:             return ncp;
 862:          }
 863: #if 0
 864:          //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy。

### Lines 865-896 / 第 865-896 行
~~~cpp
 865:          // This code is disabled, since there can be multiple answers to the
 866:          // question, and it's not clear how to find the "right" one.
 867:          //
 868:          static RealType find_degrees_of_freedom(RealType delta, RealType x, RealType p)
 869:          {
 870:             const char* function = "non_central_t<%1%>::find_degrees_of_freedom";
 871:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 872:             typedef typename policies::normalise<
 873:                Policy,
 874:                policies::promote_float<false>,
 875:                policies::promote_double<false>,
 876:                policies::discrete_quantile<>,
 877:                policies::assert_undefined<> >::type forwarding_policy;
 878:             value_type result = detail::find_t_degrees_of_freedom(
 879:                static_cast<value_type>(delta),
 880:                static_cast<value_type>(x),
 881:                static_cast<value_type>(p),
 882:                static_cast<value_type>(1-p),
 883:                forwarding_policy());
 884:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 885:                result,
 886:                function);
 887:          }
 888:          template <class A, class B, class C>
 889:          static RealType find_degrees_of_freedom(const complemented3_type<A,B,C>& c)
 890:          {
 891:             const char* function = "non_central_t<%1%>::find_degrees_of_freedom";
 892:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 893:             typedef typename policies::normalise<
 894:                Policy,
 895:                policies::promote_float<false>,
 896:                policies::promote_double<false>,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `A` as part of the file's main abstraction. This range declares or defines callable logic such as forwarding_policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `A`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 forwarding_policy。

### Lines 897-928 / 第 897-928 行
~~~cpp
 897:                policies::discrete_quantile<>,
 898:                policies::assert_undefined<> >::type forwarding_policy;
 899:             value_type result = detail::find_t_degrees_of_freedom(
 900:                static_cast<value_type>(c.dist),
 901:                static_cast<value_type>(c.param1),
 902:                static_cast<value_type>(1-c.param2),
 903:                static_cast<value_type>(c.param2),
 904:                forwarding_policy());
 905:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 906:                result,
 907:                function);
 908:          }
 909:          static RealType find_non_centrality(RealType v, RealType x, RealType p)
 910:          {
 911:             const char* function = "non_central_t<%1%>::find_t_non_centrality";
 912:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 913:             typedef typename policies::normalise<
 914:                Policy,
 915:                policies::promote_float<false>,
 916:                policies::promote_double<false>,
 917:                policies::discrete_quantile<>,
 918:                policies::assert_undefined<> >::type forwarding_policy;
 919:             value_type result = detail::find_t_non_centrality(
 920:                static_cast<value_type>(v),
 921:                static_cast<value_type>(x),
 922:                static_cast<value_type>(p),
 923:                static_cast<value_type>(1-p),
 924:                forwarding_policy());
 925:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 926:                result,
 927:                function);
 928:          }
~~~
- **EN:** This range declares or defines callable logic such as forwarding_policy. Return statements hand the computed result or status back to the caller. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围声明或定义了可调用逻辑，例如 forwarding_policy。 return 语句会把计算结果或状态返回给调用方。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 929-960 / 第 929-960 行
~~~cpp
 929:          template <class A, class B, class C>
 930:          static RealType find_non_centrality(const complemented3_type<A,B,C>& c)
 931:          {
 932:             const char* function = "non_central_t<%1%>::find_t_non_centrality";
 933:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 934:             typedef typename policies::normalise<
 935:                Policy,
 936:                policies::promote_float<false>,
 937:                policies::promote_double<false>,
 938:                policies::discrete_quantile<>,
 939:                policies::assert_undefined<> >::type forwarding_policy;
 940:             value_type result = detail::find_t_non_centrality(
 941:                static_cast<value_type>(c.dist),
 942:                static_cast<value_type>(c.param1),
 943:                static_cast<value_type>(1-c.param2),
 944:                static_cast<value_type>(c.param2),
 945:                forwarding_policy());
 946:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 947:                result,
 948:                function);
 949:          }
 950: #endif
 951:       private:
 952:          // Data member, initialized by constructor.
 953:          RealType v;   // degrees of freedom
 954:          RealType ncp; // non-centrality parameter
 955:       }; // template <class RealType, class Policy> class non_central_t_distribution
 956: 
 957:       typedef non_central_t_distribution<double> non_central_t; // Reserved name of type double.
 958: 
 959:       #ifdef __cpp_deduction_guides
 960:       template <class RealType>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `A` as part of the file's main abstraction. This range declares or defines callable logic such as forwarding_policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `A`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 forwarding_policy。

### Lines 961-992 / 第 961-992 行
~~~cpp
 961:       non_central_t_distribution(RealType,RealType)->non_central_t_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 962:       #endif
 963: 
 964:       // Non-member functions to give properties of the distribution.
 965: 
 966:       template <class RealType, class Policy>
 967:       inline const std::pair<RealType, RealType> range(const non_central_t_distribution<RealType, Policy>& /* dist */)
 968:       { // Range of permissible values for random variable k.
 969:          using boost::math::tools::max_value;
 970:          return std::pair<RealType, RealType>(-max_value<RealType>(), max_value<RealType>());
 971:       }
 972: 
 973:       template <class RealType, class Policy>
 974:       inline const std::pair<RealType, RealType> support(const non_central_t_distribution<RealType, Policy>& /* dist */)
 975:       { // Range of supported values for random variable k.
 976:          // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 977:          using boost::math::tools::max_value;
 978:          return std::pair<RealType, RealType>(-max_value<RealType>(), max_value<RealType>());
 979:       }
 980: 
 981:       template <class RealType, class Policy>
 982:       inline RealType mode(const non_central_t_distribution<RealType, Policy>& dist)
 983:       { // mode.
 984:          static const char* function = "mode(non_central_t_distribution<%1%> const&)";
 985:          RealType v = dist.degrees_of_freedom();
 986:          RealType l = dist.non_centrality();
 987:          RealType r;
 988:          if(!detail::check_df_gt0_to_inf(
 989:             function,
 990:             v, &r, Policy())
 991:             ||
 992:          !detail::check_non_centrality(
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 993-1024 / 第 993-1024 行
~~~cpp
 993:             function,
 994:             static_cast<RealType>(l * l),
 995:             &r,
 996:             Policy()))
 997:                return static_cast<RealType>(r);
 998: 
 999:          BOOST_MATH_STD_USING
1000: 
1001:          RealType m = v < 3 ? 0 : detail::mean(v, l, Policy());
1002:          RealType var = v < 4 ? 1 : detail::variance(v, l, Policy());
1003: 
1004:          return detail::generic_find_mode(
1005:             dist,
1006:             m,
1007:             function,
1008:             sqrt(var));
1009:       }
1010: 
1011:       template <class RealType, class Policy>
1012:       inline RealType mean(const non_central_t_distribution<RealType, Policy>& dist)
1013:       {
1014:          BOOST_MATH_STD_USING
1015:          const char* function = "mean(const non_central_t_distribution<%1%>&)";
1016:          typedef typename policies::evaluation<RealType, Policy>::type value_type;
1017:          typedef typename policies::normalise<
1018:             Policy,
1019:             policies::promote_float<false>,
1020:             policies::promote_double<false>,
1021:             policies::discrete_quantile<>,
1022:             policies::assert_undefined<> >::type forwarding_policy;
1023:          RealType v = dist.degrees_of_freedom();
1024:          RealType l = dist.non_centrality();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as detail::mean, detail::variance, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 detail::mean, detail::variance, ...。

### Lines 1025-1056 / 第 1025-1056 行
~~~cpp
1025:          RealType r;
1026:          if(!detail::check_df_gt0_to_inf(
1027:             function,
1028:             v, &r, Policy())
1029:             ||
1030:          !detail::check_non_centrality(
1031:             function,
1032:             static_cast<RealType>(l * l),
1033:             &r,
1034:             Policy()))
1035:                return static_cast<RealType>(r);
1036:          if(v <= 1)
1037:             return policies::raise_domain_error<RealType>(
1038:                function,
1039:                "The non-central t distribution has no defined mean for degrees of freedom <= 1: got v=%1%.", v, Policy());
1040:          // return l * sqrt(v / 2) * tgamma_delta_ratio((v - 1) * 0.5f, RealType(0.5f));
1041:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
1042:             detail::mean(static_cast<value_type>(v), static_cast<value_type>(l), forwarding_policy()), function);
1043: 
1044:       } // mean
1045: 
1046:       template <class RealType, class Policy>
1047:       inline RealType variance(const non_central_t_distribution<RealType, Policy>& dist)
1048:       { // variance.
1049:          const char* function = "variance(const non_central_t_distribution<%1%>&)";
1050:          typedef typename policies::evaluation<RealType, Policy>::type value_type;
1051:          typedef typename policies::normalise<
1052:             Policy,
1053:             policies::promote_float<false>,
1054:             policies::promote_double<false>,
1055:             policies::discrete_quantile<>,
1056:             policies::assert_undefined<> >::type forwarding_policy;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, sqrt, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, sqrt, ...。

### Lines 1057-1088 / 第 1057-1088 行
~~~cpp
1057:          BOOST_MATH_STD_USING
1058:          RealType v = dist.degrees_of_freedom();
1059:          RealType l = dist.non_centrality();
1060:          RealType r;
1061:          if(!detail::check_df_gt0_to_inf(
1062:             function,
1063:             v, &r, Policy())
1064:             ||
1065:          !detail::check_non_centrality(
1066:             function,
1067:             static_cast<RealType>(l * l),
1068:             &r,
1069:             Policy()))
1070:                return static_cast<RealType>(r);
1071:          if(v <= 2)
1072:             return policies::raise_domain_error<RealType>(
1073:                function,
1074:                "The non-central t distribution has no defined variance for degrees of freedom <= 2: got v=%1%.", v, Policy());
1075:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
1076:             detail::variance(static_cast<value_type>(v), static_cast<value_type>(l), forwarding_policy()), function);
1077:       }
1078: 
1079:       // RealType standard_deviation(const non_central_t_distribution<RealType, Policy>& dist)
1080:       // standard_deviation provided by derived accessors.
1081: 
1082:       template <class RealType, class Policy>
1083:       inline RealType skewness(const non_central_t_distribution<RealType, Policy>& dist)
1084:       { // skewness = sqrt(l).
1085:          const char* function = "skewness(const non_central_t_distribution<%1%>&)";
1086:          typedef typename policies::evaluation<RealType, Policy>::type value_type;
1087:          typedef typename policies::normalise<
1088:             Policy,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality, ...。

### Lines 1089-1120 / 第 1089-1120 行
~~~cpp
1089:             policies::promote_float<false>,
1090:             policies::promote_double<false>,
1091:             policies::discrete_quantile<>,
1092:             policies::assert_undefined<> >::type forwarding_policy;
1093:          RealType v = dist.degrees_of_freedom();
1094:          RealType l = dist.non_centrality();
1095:          RealType r;
1096:          if(!detail::check_df_gt0_to_inf(
1097:             function,
1098:             v, &r, Policy())
1099:             ||
1100:          !detail::check_non_centrality(
1101:             function,
1102:             static_cast<RealType>(l * l),
1103:             &r,
1104:             Policy()))
1105:                return static_cast<RealType>(r);
1106:          if(v <= 3)
1107:             return policies::raise_domain_error<RealType>(
1108:                function,
1109:                "The non-central t distribution has no defined skewness for degrees of freedom <= 3: got v=%1%.", v, Policy());;
1110:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
1111:             detail::skewness(static_cast<value_type>(v), static_cast<value_type>(l), forwarding_policy()), function);
1112:       }
1113: 
1114:       template <class RealType, class Policy>
1115:       inline RealType kurtosis_excess(const non_central_t_distribution<RealType, Policy>& dist)
1116:       {
1117:          const char* function = "kurtosis_excess(const non_central_t_distribution<%1%>&)";
1118:          typedef typename policies::evaluation<RealType, Policy>::type value_type;
1119:          typedef typename policies::normalise<
1120:             Policy,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality, ...。

### Lines 1121-1152 / 第 1121-1152 行
~~~cpp
1121:             policies::promote_float<false>,
1122:             policies::promote_double<false>,
1123:             policies::discrete_quantile<>,
1124:             policies::assert_undefined<> >::type forwarding_policy;
1125:          RealType v = dist.degrees_of_freedom();
1126:          RealType l = dist.non_centrality();
1127:          RealType r;
1128:          if(!detail::check_df_gt0_to_inf(
1129:             function,
1130:             v, &r, Policy())
1131:             ||
1132:          !detail::check_non_centrality(
1133:             function,
1134:             static_cast<RealType>(l * l),
1135:             &r,
1136:             Policy()))
1137:                return static_cast<RealType>(r);
1138:          if(v <= 4)
1139:             return policies::raise_domain_error<RealType>(
1140:                function,
1141:                "The non-central t distribution has no defined kurtosis for degrees of freedom <= 4: got v=%1%.", v, Policy());;
1142:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
1143:             detail::kurtosis_excess(static_cast<value_type>(v), static_cast<value_type>(l), forwarding_policy()), function);
1144:       } // kurtosis_excess
1145: 
1146:       template <class RealType, class Policy>
1147:       inline RealType kurtosis(const non_central_t_distribution<RealType, Policy>& dist)
1148:       {
1149:          return kurtosis_excess(dist) + 3;
1150:       }
1151: 
1152:       template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality, ...。

### Lines 1153-1184 / 第 1153-1184 行
~~~cpp
1153:       inline RealType pdf(const non_central_t_distribution<RealType, Policy>& dist, const RealType& t)
1154:       { // Probability Density/Mass Function.
1155:          const char* function = "pdf(non_central_t_distribution<%1%>, %1%)";
1156:          typedef typename policies::evaluation<RealType, Policy>::type value_type;
1157:          typedef typename policies::normalise<
1158:             Policy,
1159:             policies::promote_float<false>,
1160:             policies::promote_double<false>,
1161:             policies::discrete_quantile<>,
1162:             policies::assert_undefined<> >::type forwarding_policy;
1163: 
1164:          RealType v = dist.degrees_of_freedom();
1165:          RealType l = dist.non_centrality();
1166:          RealType r;
1167:          if(!detail::check_df_gt0_to_inf(
1168:             function,
1169:             v, &r, Policy())
1170:             ||
1171:          !detail::check_non_centrality(
1172:             function,
1173:             static_cast<RealType>(l * l), // we need l^2 to be countable.
1174:             &r,
1175:             Policy())
1176:             ||
1177:          !detail::check_x(
1178:             function,
1179:             t,
1180:             &r,
1181:             Policy()))
1182:                return static_cast<RealType>(r);
1183:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
1184:             detail::non_central_t_pdf(static_cast<value_type>(v),
~~~
- **EN:** This range declares or defines callable logic such as degrees_of_freedom, non_centrality. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 1185-1216 / 第 1185-1216 行
~~~cpp
1185:                static_cast<value_type>(l),
1186:                static_cast<value_type>(t),
1187:                Policy()),
1188:             function);
1189:       } // pdf
1190: 
1191:       template <class RealType, class Policy>
1192:       RealType cdf(const non_central_t_distribution<RealType, Policy>& dist, const RealType& x)
1193:       {
1194:          const char* function = "boost::math::cdf(non_central_t_distribution<%1%>&, %1%)";
1195: //   was const char* function = "boost::math::non_central_t_distribution<%1%>::cdf(%1%)";
1196:          typedef typename policies::evaluation<RealType, Policy>::type value_type;
1197:          typedef typename policies::normalise<
1198:             Policy,
1199:             policies::promote_float<false>,
1200:             policies::promote_double<false>,
1201:             policies::discrete_quantile<>,
1202:             policies::assert_undefined<> >::type forwarding_policy;
1203: 
1204:          RealType v = dist.degrees_of_freedom();
1205:          RealType l = dist.non_centrality();
1206:          RealType r;
1207:          if(!detail::check_df_gt0_to_inf(
1208:             function,
1209:             v, &r, Policy())
1210:             ||
1211:          !detail::check_non_centrality(
1212:             function,
1213:             static_cast<RealType>(l * l),
1214:             &r,
1215:             Policy())
1216:             ||
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 1217-1248 / 第 1217-1248 行
~~~cpp
1217:          !detail::check_x(
1218:             function,
1219:             x,
1220:             &r,
1221:             Policy()))
1222:                return static_cast<RealType>(r);
1223:          if ((boost::math::isinf)(v))
1224:           { // Infinite degrees of freedom, so use normal distribution located at delta.
1225:              normal_distribution<RealType, Policy> n(l, 1);
1226:              cdf(n, x);
1227:               //return cdf(normal_distribution<RealType, Policy>(l, 1), x);
1228:           }
1229: 
1230:          if(l == 0)
1231:          { // NO non-centrality, so use Student's t instead.
1232:             return cdf(students_t_distribution<RealType, Policy>(v), x);
1233:          }
1234:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
1235:             detail::non_central_t_cdf(
1236:                static_cast<value_type>(v),
1237:                static_cast<value_type>(l),
1238:                static_cast<value_type>(x),
1239:                false, Policy()),
1240:             function);
1241:       } // cdf
1242: 
1243:       template <class RealType, class Policy>
1244:       RealType cdf(const complemented2_type<non_central_t_distribution<RealType, Policy>, RealType>& c)
1245:       { // Complemented Cumulative Distribution Function
1246:   // was       const char* function = "boost::math::non_central_t_distribution<%1%>::cdf(%1%)";
1247:          const char* function = "boost::math::cdf(const complement(non_central_t_distribution<%1%>&), %1%)";
1248:          typedef typename policies::evaluation<RealType, Policy>::type value_type;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as n, cdf.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 n, cdf。

### Lines 1249-1280 / 第 1249-1280 行
~~~cpp
1249:          typedef typename policies::normalise<
1250:             Policy,
1251:             policies::promote_float<false>,
1252:             policies::promote_double<false>,
1253:             policies::discrete_quantile<>,
1254:             policies::assert_undefined<> >::type forwarding_policy;
1255: 
1256:          non_central_t_distribution<RealType, Policy> const& dist = c.dist;
1257:          RealType x = c.param;
1258:          RealType v = dist.degrees_of_freedom();
1259:          RealType l = dist.non_centrality(); // aka delta
1260:          RealType r;
1261:          if(!detail::check_df_gt0_to_inf(
1262:             function,
1263:             v, &r, Policy())
1264:             ||
1265:          !detail::check_non_centrality(
1266:             function,
1267:             static_cast<RealType>(l * l),
1268:             &r,
1269:             Policy())
1270:             ||
1271:          !detail::check_x(
1272:             function,
1273:             x,
1274:             &r,
1275:             Policy()))
1276:                return static_cast<RealType>(r);
1277: 
1278:          if ((boost::math::isinf)(v))
1279:          { // Infinite degrees of freedom, so use normal distribution located at delta.
1280:              normal_distribution<RealType, Policy> n(l, 1);
~~~
- **EN:** This range declares or defines callable logic such as degrees_of_freedom, non_centrality, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 1281-1312 / 第 1281-1312 行
~~~cpp
1281:              return cdf(complement(n, x));
1282:          }
1283:          if(l == 0)
1284:          { // zero non-centrality so use Student's t distribution.
1285:             return cdf(complement(students_t_distribution<RealType, Policy>(v), x));
1286:          }
1287:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
1288:             detail::non_central_t_cdf(
1289:                static_cast<value_type>(v),
1290:                static_cast<value_type>(l),
1291:                static_cast<value_type>(x),
1292:                true, Policy()),
1293:             function);
1294:       } // ccdf
1295: 
1296:       template <class RealType, class Policy>
1297:       inline RealType quantile(const non_central_t_distribution<RealType, Policy>& dist, const RealType& p)
1298:       { // Quantile (or Percent Point) function.
1299:          static const char* function = "quantile(const non_central_t_distribution<%1%>, %1%)";
1300:          RealType v = dist.degrees_of_freedom();
1301:          RealType l = dist.non_centrality();
1302:          return detail::non_central_t_quantile(function, v, l, p, RealType(1-p), Policy());
1303:       } // quantile
1304: 
1305:       template <class RealType, class Policy>
1306:       inline RealType quantile(const complemented2_type<non_central_t_distribution<RealType, Policy>, RealType>& c)
1307:       { // Quantile (or Percent Point) function.
1308:          static const char* function = "quantile(const complement(non_central_t_distribution<%1%>, %1%))";
1309:          non_central_t_distribution<RealType, Policy> const& dist = c.dist;
1310:          RealType q = c.param;
1311:          RealType v = dist.degrees_of_freedom();
1312:          RealType l = dist.non_centrality();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 1313-1325 / 第 1313-1325 行
~~~cpp
1313:          return detail::non_central_t_quantile(function, v, l, RealType(1-q), q, Policy());
1314:       } // quantile complement.
1315: 
1316:    } // namespace math
1317: } // namespace boost
1318: 
1319: // This include must be at the end, *after* the accessors
1320: // for this distribution have been defined, in order to
1321: // keep compilers that support two-phase lookup happy.
1322: #include <boost/math/distributions/detail/derived_accessors.hpp>
1323: 
1324: #endif // BOOST_MATH_SPECIAL_NON_CENTRAL_T_HPP
1325: 
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp so the surrounding code can use external declarations. The code enters namespace scope (math::boost) to keep symbols organized. Return statements hand the computed result or status back to the caller.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 return 语句会把计算结果或状态返回给调用方。

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
- **Included headers / 包含头文件**: `boost/math/distributions/fwd.hpp, boost/math/distributions/non_central_beta.hpp, boost/math/distributions/normal.hpp, boost/math/distributions/students_t.hpp, boost/math/distributions/detail/generic_quantile.hpp, boost/math/special_functions/trunc.hpp, boost/math/special_functions/detail/hypergeometric_series.hpp, boost/math/quadrature/exp_sinh.hpp, boost/math/distributions/detail/derived_accessors.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `lltrunc, detail::ibeta_imp, poisf, exp, pow, boost::math::tgamma, n, cdf, ...`
