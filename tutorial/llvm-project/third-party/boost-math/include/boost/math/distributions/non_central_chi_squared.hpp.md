# non_central_chi_squared.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/non_central_chi_squared.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the non central chi squared distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 non central chi squared 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行
~~~cpp
   1: // boost\math\distributions\non_central_chi_squared.hpp
   2: 
   3: // Copyright John Maddock 2008.
   4: // Copyright Matt Borland 2024.
   5: // Use, modification and distribution are subject to the
   6: // Boost Software License, Version 1.0.
   7: // (See accompanying file LICENSE_1_0.txt
   8: // or copy at http://www.boost.org/LICENSE_1_0.txt)
   9: 
  10: #ifndef BOOST_MATH_SPECIAL_NON_CENTRAL_CHI_SQUARE_HPP
  11: #define BOOST_MATH_SPECIAL_NON_CENTRAL_CHI_SQUARE_HPP
  12: 
  13: #include <boost/math/tools/config.hpp>
  14: #include <boost/math/tools/tuple.hpp>
  15: #include <boost/math/tools/cstdint.hpp>
  16: #include <boost/math/tools/numeric_limits.hpp>
  17: #include <boost/math/distributions/fwd.hpp>
  18: #include <boost/math/special_functions/gamma.hpp> // for incomplete gamma. gamma_q
  19: #include <boost/math/special_functions/bessel.hpp> // for cyl_bessel_i
  20: #include <boost/math/special_functions/round.hpp> // for llround
  21: #include <boost/math/distributions/complement.hpp> // complements
  22: #include <boost/math/distributions/chi_squared.hpp> // central distribution
  23: #include <boost/math/distributions/detail/common_error_handling.hpp> // error checks
  24: #include <boost/math/special_functions/fpclassify.hpp> // isnan.
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 25-48 / 第 25-48 行
~~~cpp
  25: #include <boost/math/tools/roots.hpp> // for root finding.
  26: #include <boost/math/distributions/detail/generic_mode.hpp>
  27: #include <boost/math/distributions/detail/generic_quantile.hpp>
  28: #include <boost/math/policies/policy.hpp>
  29: 
  30: namespace boost
  31: {
  32:    namespace math
  33:    {
  34: 
  35:       template <class RealType, class Policy>
  36:       class non_central_chi_squared_distribution;
  37: 
  38:       namespace detail{
  39: 
  40:          template <class T, class Policy>
  41:          BOOST_MATH_GPU_ENABLED T non_central_chi_square_q(T x, T f, T theta, const Policy& pol, T init_sum = 0)
  42:          {
  43:             //
  44:             // Computes the complement of the Non-Central Chi-Square
  45:             // Distribution CDF by summing a weighted sum of complements
  46:             // of the central-distributions.  The weighting factor is
  47:             // a Poisson Distribution.
  48:             //
~~~
- **EN:** This block imports dependencies such as boost/math/tools/roots.hpp, boost/math/distributions/detail/generic_mode.hpp, boost/math/distributions/detail/generic_quantile.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/tools/roots.hpp, boost/math/distributions/detail/generic_mode.hpp, boost/math/distributions/detail/generic_quantile.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 49-72 / 第 49-72 行
~~~cpp
  49:             // This is an application of the technique described in:
  50:             //
  51:             // Computing discrete mixtures of continuous
  52:             // distributions: noncentral chisquare, noncentral t
  53:             // and the distribution of the square of the sample
  54:             // multiple correlation coefficient.
  55:             // D. Benton, K. Krishnamoorthy.
  56:             // Computational Statistics & Data Analysis 43 (2003) 249 - 267
  57:             //
  58:             BOOST_MATH_STD_USING
  59: 
  60:             // Special case:
  61:             if(x == 0)
  62:                return 1;
  63: 
  64:             //
  65:             // Initialize the variables we'll be using:
  66:             //
  67:             T lambda = theta / 2;
  68:             T del = f / 2;
  69:             T y = x / 2;
  70:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
  71:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
  72:             T sum = init_sum;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 73-96 / 第 73-96 行
~~~cpp
  73:             //
  74:             // k is the starting location for iteration, we'll
  75:             // move both forwards and backwards from this point.
  76:             // k is chosen as the peek of the Poisson weights, which
  77:             // will occur *before* the largest term.
  78:             //
  79:             long long k = llround(lambda, pol);
  80:             // Forwards and backwards Poisson weights:
  81:             T poisf = boost::math::gamma_p_derivative(static_cast<T>(1 + k), lambda, pol);
  82:             T poisb = poisf * k / lambda;
  83:             // Initial forwards central chi squared term:
  84:             T gamf = boost::math::gamma_q(del + k, y, pol);
  85:             // Forwards and backwards recursion terms on the central chi squared:
  86:             T xtermf = boost::math::gamma_p_derivative(del + 1 + k, y, pol);
  87:             T xtermb = xtermf * (del + k) / y;
  88:             // Initial backwards central chi squared term:
  89:             T gamb = gamf - xtermb;
  90: 
  91:             //
  92:             // Forwards iteration first, this is the
  93:             // stable direction for the gamma function
  94:             // recurrences:
  95:             //
  96:             long long i;
~~~
- **EN:** This range declares or defines callable logic such as llround, boost::math::gamma_p_derivative, .... The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 llround, boost::math::gamma_p_derivative, ...。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 97-120 / 第 97-120 行
~~~cpp
  97:             for(i = k; static_cast<boost::math::uintmax_t>(i-k) < max_iter; ++i)
  98:             {
  99:                T term = poisf * gamf;
 100:                sum += term;
 101:                poisf *= lambda / (i + 1);
 102:                gamf += xtermf;
 103:                xtermf *= y / (del + i + 1);
 104:                if(((sum == 0) || (fabs(term / sum) < errtol)) && (term >= poisf * gamf))
 105:                   break;
 106:             }
 107:             //Error check:
 108:             if(static_cast<boost::math::uintmax_t>(i-k) >= max_iter)
 109:                return policies::raise_evaluation_error("cdf(non_central_chi_squared_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 110:             //
 111:             // Now backwards iteration: the gamma
 112:             // function recurrences are unstable in this
 113:             // direction, we rely on the terms diminishing in size
 114:             // faster than we introduce cancellation errors.
 115:             // For this reason it's very important that we start
 116:             // *before* the largest term so that backwards iteration
 117:             // is strictly converging.
 118:             //
 119:             for(i = k - 1; i >= 0; --i)
 120:             {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 return 语句会把计算结果或状态返回给调用方。

### Lines 121-144 / 第 121-144 行
~~~cpp
 121:                T term = poisb * gamb;
 122:                sum += term;
 123:                poisb *= i / lambda;
 124:                xtermb *= (del + i) / y;
 125:                gamb -= xtermb;
 126:                if((sum == 0) || (fabs(term / sum) < errtol))
 127:                   break;
 128:             }
 129: 
 130:             return sum;
 131:          }
 132: 
 133:          template <class T, class Policy>
 134:          BOOST_MATH_GPU_ENABLED T non_central_chi_square_p_ding(T x, T f, T theta, const Policy& pol, T init_sum = 0)
 135:          {
 136:             //
 137:             // This is an implementation of:
 138:             //
 139:             // Algorithm AS 275:
 140:             // Computing the Non-Central #2 Distribution Function
 141:             // Cherng G. Ding
 142:             // Applied Statistics, Vol. 41, No. 2. (1992), pp. 478-482.
 143:             //
 144:             // This uses a stable forward iteration to sum the
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 145-168 / 第 145-168 行
~~~cpp
 145:             // CDF, unfortunately this can not be used for large
 146:             // values of the non-centrality parameter because:
 147:             // * The first term may underflow to zero.
 148:             // * We may need an extra-ordinary number of terms
 149:             //   before we reach the first *significant* term.
 150:             //
 151:             BOOST_MATH_STD_USING
 152:             // Special case:
 153:             if(x == 0)
 154:                return 0;
 155:             T tk = boost::math::gamma_p_derivative(f/2 + 1, x/2, pol);
 156:             T lambda = theta / 2;
 157:             T vk = exp(-lambda);
 158:             T uk = vk;
 159:             T sum = init_sum + tk * vk;
 160:             if(sum == 0)
 161:                return sum;
 162: 
 163:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 164:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
 165: 
 166:             int i;
 167:             T lterm(0), term(0);
 168:             for(i = 1; static_cast<boost::math::uintmax_t>(i) < max_iter; ++i)
~~~
- **EN:** This range declares or defines callable logic such as boost::math::gamma_p_derivative, exp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::gamma_p_derivative, exp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 169-192 / 第 169-192 行
~~~cpp
 169:             {
 170:                tk = tk * x / (f + 2 * i);
 171:                uk = uk * lambda / i;
 172:                vk = vk + uk;
 173:                lterm = term;
 174:                term = vk * tk;
 175:                sum += term;
 176:                if((fabs(term / sum) < errtol) && (term <= lterm))
 177:                   break;
 178:             }
 179:             //Error check:
 180:             if(static_cast<boost::math::uintmax_t>(i) >= max_iter)
 181:                return policies::raise_evaluation_error("cdf(non_central_chi_squared_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 182:             return sum;
 183:          }
 184: 
 185: 
 186:          template <class T, class Policy>
 187:          BOOST_MATH_GPU_ENABLED T non_central_chi_square_p(T y, T n, T lambda, const Policy& pol, T init_sum)
 188:          {
 189:             //
 190:             // This is taken more or less directly from:
 191:             //
 192:             // Computing discrete mixtures of continuous
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 193-216 / 第 193-216 行
~~~cpp
 193:             // distributions: noncentral chisquare, noncentral t
 194:             // and the distribution of the square of the sample
 195:             // multiple correlation coefficient.
 196:             // D. Benton, K. Krishnamoorthy.
 197:             // Computational Statistics & Data Analysis 43 (2003) 249 - 267
 198:             //
 199:             // We're summing a Poisson weighting term multiplied by
 200:             // a central chi squared distribution.
 201:             //
 202:             BOOST_MATH_STD_USING
 203:             // Special case:
 204:             if(y == 0)
 205:                return 0;
 206:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 207:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
 208:             T errorf(0), errorb(0);
 209: 
 210:             T x = y / 2;
 211:             T del = lambda / 2;
 212:             //
 213:             // Starting location for the iteration, we'll iterate
 214:             // both forwards and backwards from this point.  The
 215:             // location chosen is the maximum of the Poisson weight
 216:             // function, which ocurrs *after* the largest term in the
~~~
- **EN:** This range declares or defines callable logic such as errorf. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 errorf。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-240 / 第 217-240 行
~~~cpp
 217:             // sum.
 218:             //
 219:             long long k = llround(del, pol);
 220:             T a = n / 2 + k;
 221:             // Central chi squared term for forward iteration:
 222:             T gamkf = boost::math::gamma_p(a, x, pol);
 223: 
 224:             if(lambda == 0)
 225:                return gamkf;
 226:             // Central chi squared term for backward iteration:
 227:             T gamkb = gamkf;
 228:             // Forwards Poisson weight:
 229:             T poiskf = gamma_p_derivative(static_cast<T>(k+1), del, pol);
 230:             // Backwards Poisson weight:
 231:             T poiskb = poiskf;
 232:             // Forwards gamma function recursion term:
 233:             T xtermf = boost::math::gamma_p_derivative(a, x, pol);
 234:             // Backwards gamma function recursion term:
 235:             T xtermb = xtermf * x / a;
 236:             T sum = init_sum + poiskf * gamkf;
 237:             if(sum == 0)
 238:                return sum;
 239:             int i = 1;
 240:             //
~~~
- **EN:** This range declares or defines callable logic such as llround, boost::math::gamma_p, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 llround, boost::math::gamma_p, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 241-264 / 第 241-264 行
~~~cpp
 241:             // Backwards recursion first, this is the stable
 242:             // direction for gamma function recurrences:
 243:             //
 244:             while(i <= k)
 245:             {
 246:                xtermb *= (a - i + 1) / x;
 247:                gamkb += xtermb;
 248:                poiskb = poiskb * (k - i + 1) / del;
 249:                errorf = errorb;
 250:                errorb = gamkb * poiskb;
 251:                sum += errorb;
 252:                if((fabs(errorb / sum) < errtol) && (errorb <= errorf))
 253:                   break;
 254:                ++i;
 255:             }
 256:             i = 1;
 257:             //
 258:             // Now forwards recursion, the gamma function
 259:             // recurrence relation is unstable in this direction,
 260:             // so we rely on the magnitude of successive terms
 261:             // decreasing faster than we introduce cancellation error.
 262:             // For this reason it's vital that k is chosen to be *after*
 263:             // the largest term, so that successive forward iterations
 264:             // are strictly (and rapidly) converging.
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 265-288 / 第 265-288 行
~~~cpp
 265:             //
 266:             do
 267:             {
 268:                xtermf = xtermf * x / (a + i - 1);
 269:                gamkf = gamkf - xtermf;
 270:                poiskf = poiskf * del / (k + i);
 271:                errorf = poiskf * gamkf;
 272:                sum += errorf;
 273:                ++i;
 274:             }while((fabs(errorf / sum) > errtol) && (static_cast<boost::math::uintmax_t>(i) < max_iter));
 275: 
 276:             //Error check:
 277:             if(static_cast<boost::math::uintmax_t>(i) >= max_iter)
 278:                return policies::raise_evaluation_error("cdf(non_central_chi_squared_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 279: 
 280:             return sum;
 281:          }
 282: 
 283:          template <class T, class Policy>
 284:          BOOST_MATH_GPU_ENABLED T non_central_chi_square_pdf(T x, T n, T lambda, const Policy& pol)
 285:          {
 286:             //
 287:             // As above but for the PDF:
 288:             //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as while.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 while。

### Lines 289-312 / 第 289-312 行
~~~cpp
 289:             BOOST_MATH_STD_USING
 290:             boost::math::uintmax_t max_iter = policies::get_max_series_iterations<Policy>();
 291:             T errtol = boost::math::policies::get_epsilon<T, Policy>();
 292:             T x2 = x / 2;
 293:             T n2 = n / 2;
 294:             T l2 = lambda / 2;
 295:             T sum = 0;
 296:             long long k = lltrunc(l2);
 297:             T pois = gamma_p_derivative(static_cast<T>(k + 1), l2, pol) * gamma_p_derivative(static_cast<T>(n2 + k), x2);
 298:             if(pois == 0)
 299:                return 0;
 300:             T poisb = pois;
 301:             for(long long i = k; ; ++i)
 302:             {
 303:                sum += pois;
 304:                if(pois / sum < errtol)
 305:                   break;
 306:                if(static_cast<boost::math::uintmax_t>(i - k) >= max_iter)
 307:                   return policies::raise_evaluation_error("pdf(non_central_chi_squared_distribution<%1%>, %1%)", "Series did not converge, closest value was %1%", sum, pol); // LCOV_EXCL_LINE
 308:                pois *= l2 * x2 / ((i + 1) * (n2 + i));
 309:             }
 310:             for(long long i = k - 1; i >= 0; --i)
 311:             {
 312:                poisb *= (i + 1) * (n2 + i) / (l2 * x2);
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, gamma_p_derivative. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, gamma_p_derivative。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 313-336 / 第 313-336 行
~~~cpp
 313:                sum += poisb;
 314:                if(poisb / sum < errtol)
 315:                   break;
 316:             }
 317:             return sum / 2;
 318:          }
 319: 
 320:          template <class RealType, class Policy>
 321:          BOOST_MATH_GPU_ENABLED inline RealType non_central_chi_squared_cdf(RealType x, RealType k, RealType l, bool invert, const Policy&)
 322:          {
 323:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 324:             typedef typename policies::normalise<
 325:                Policy,
 326:                policies::promote_float<false>,
 327:                policies::promote_double<false>,
 328:                policies::discrete_quantile<>,
 329:                policies::assert_undefined<> >::type forwarding_policy;
 330: 
 331:             BOOST_MATH_STD_USING
 332:             value_type result;
 333:             if(l == 0)
 334:               return invert == false ? cdf(boost::math::chi_squared_distribution<RealType, Policy>(k), x) : cdf(complement(boost::math::chi_squared_distribution<RealType, Policy>(k), x));
 335:             else if(x > k + l)
 336:             {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 337-360 / 第 337-360 行
~~~cpp
 337:                // Complement is the smaller of the two:
 338:                result = detail::non_central_chi_square_q(
 339:                   static_cast<value_type>(x),
 340:                   static_cast<value_type>(k),
 341:                   static_cast<value_type>(l),
 342:                   forwarding_policy(),
 343:                   static_cast<value_type>(invert ? 0 : -1));
 344:                invert = !invert;
 345:             }
 346:             else if(l < 200)
 347:             {
 348:                // For small values of the non-centrality parameter
 349:                // we can use Ding's method:
 350:                result = detail::non_central_chi_square_p_ding(
 351:                   static_cast<value_type>(x),
 352:                   static_cast<value_type>(k),
 353:                   static_cast<value_type>(l),
 354:                   forwarding_policy(),
 355:                   static_cast<value_type>(invert ? -1 : 0));
 356:             }
 357:             else
 358:             {
 359:                // For largers values of the non-centrality
 360:                // parameter Ding's method will consume an
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 361-384 / 第 361-384 行
~~~cpp
 361:                // extra-ordinary number of terms, and worse
 362:                // may return zero when the result is in fact
 363:                // finite, use Krishnamoorthy's method instead:
 364:                result = detail::non_central_chi_square_p(
 365:                   static_cast<value_type>(x),
 366:                   static_cast<value_type>(k),
 367:                   static_cast<value_type>(l),
 368:                   forwarding_policy(),
 369:                   static_cast<value_type>(invert ? -1 : 0));
 370:             }
 371:             if(invert)
 372:                result = -result;
 373:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 374:                result,
 375:                "boost::math::non_central_chi_squared_cdf<%1%>(%1%, %1%, %1%)");
 376:          }
 377: 
 378:          template <class T, class Policy>
 379:          struct nccs_quantile_functor
 380:          {
 381:             BOOST_MATH_GPU_ENABLED nccs_quantile_functor(const non_central_chi_squared_distribution<T,Policy>& d, T t, bool c)
 382:                : dist(d), target(t), comp(c) {}
 383: 
 384:             BOOST_MATH_GPU_ENABLED T operator()(const T& x)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as dist.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 dist。

### Lines 385-408 / 第 385-408 行
~~~cpp
 385:             {
 386:                return comp ?
 387:                   target - cdf(complement(dist, x))
 388:                   : cdf(dist, x) - target;
 389:             }
 390: 
 391:          private:
 392:             non_central_chi_squared_distribution<T,Policy> dist;
 393:             T target;
 394:             bool comp;
 395:          };
 396: 
 397:          template <class RealType, class Policy>
 398:          BOOST_MATH_GPU_ENABLED RealType nccs_quantile(const non_central_chi_squared_distribution<RealType, Policy>& dist, const RealType& p, bool comp)
 399:          {
 400:             BOOST_MATH_STD_USING
 401:             constexpr auto function = "quantile(non_central_chi_squared_distribution<%1%>, %1%)";
 402:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 403:             typedef typename policies::normalise<
 404:                Policy,
 405:                policies::promote_float<false>,
 406:                policies::promote_double<false>,
 407:                policies::discrete_quantile<>,
 408:                policies::assert_undefined<> >::type forwarding_policy;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 409-432 / 第 409-432 行
~~~cpp
 409: 
 410:             value_type k = dist.degrees_of_freedom();
 411:             value_type l = dist.non_centrality();
 412:             value_type r;
 413:             if(!detail::check_df(
 414:                function,
 415:                k, &r, Policy())
 416:                ||
 417:             !detail::check_non_centrality(
 418:                function,
 419:                l,
 420:                &r,
 421:                Policy())
 422:                ||
 423:             !detail::check_probability(
 424:                function,
 425:                static_cast<value_type>(p),
 426:                &r,
 427:                Policy()))
 428:                   return static_cast<RealType>(r);
 429:             //
 430:             // Special cases get short-circuited first:
 431:             //
 432:             if(p == 0)
~~~
- **EN:** This range declares or defines callable logic such as degrees_of_freedom, non_centrality. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 433-456 / 第 433-456 行
~~~cpp
 433:                return comp ? policies::raise_overflow_error<RealType>(function, 0, Policy()) : 0;
 434:             if(p == 1)
 435:                return comp ? 0 : policies::raise_overflow_error<RealType>(function, 0, Policy());
 436:             //
 437:             // This is Pearson's approximation to the quantile, see
 438:             // Pearson, E. S. (1959) "Note on an approximation to the distribution of
 439:             // noncentral chi squared", Biometrika 46: 364.
 440:             // See also:
 441:             // "A comparison of approximations to percentiles of the noncentral chi2-distribution",
 442:             // Hardeo Sahai and Mario Miguel Ojeda, Revista de Matematica: Teoria y Aplicaciones 2003 10(1-2) : 57-76.
 443:             // Note that the latter reference refers to an approximation of the CDF, when they really mean the quantile.
 444:             //
 445:             value_type b = -(l * l) / (k + 3 * l);
 446:             value_type c = (k + 3 * l) / (k + 2 * l);
 447:             value_type ff = (k + 2 * l) / (c * c);
 448:             value_type guess;
 449:             if(comp)
 450:             {
 451:                guess = b + c * quantile(complement(chi_squared_distribution<value_type, forwarding_policy>(ff), p));
 452:             }
 453:             else
 454:             {
 455:                guess = b + c * quantile(chi_squared_distribution<value_type, forwarding_policy>(ff), p);
 456:             }
~~~
- **EN:** This range declares or defines callable logic such as quantile. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 quantile。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 457-480 / 第 457-480 行
~~~cpp
 457:             //
 458:             // Sometimes guess goes very small or negative, in that case we have
 459:             // to do something else for the initial guess, this approximation
 460:             // was provided in a private communication from Thomas Luu, PhD candidate,
 461:             // University College London.  It's an asymptotic expansion for the
 462:             // quantile which usually gets us within an order of magnitude of the
 463:             // correct answer.
 464:             // Fast and accurate parallel computation of quantile functions for random number generation,
 465:             // Thomas LuuDoctorial Thesis 2016
 466:             // http://discovery.ucl.ac.uk/1482128/
 467:             //
 468:             if(guess < 0.005)
 469:             {
 470:                value_type pp = comp ? 1 - p : p;
 471:                //guess = pow(pow(value_type(2), (k / 2 - 1)) * exp(l / 2) * pp * k, 2 / k);
 472:                guess = pow(pow(value_type(2), (k / 2 - 1)) * exp(l / 2) * pp * k * boost::math::tgamma(k / 2, forwarding_policy()), (2 / k));
 473:                if(guess == 0)
 474:                   guess = tools::min_value<value_type>();
 475:             }
 476:             value_type result = detail::generic_quantile(
 477:                non_central_chi_squared_distribution<value_type, forwarding_policy>(k, l),
 478:                p,
 479:                guess,
 480:                comp,
~~~
- **EN:** This range declares or defines callable logic such as pow. Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 481-504 / 第 481-504 行
~~~cpp
 481:                function);
 482: 
 483:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 484:                result,
 485:                function);
 486:          }
 487: 
 488:          template <class RealType, class Policy>
 489:          BOOST_MATH_GPU_ENABLED RealType nccs_pdf(const non_central_chi_squared_distribution<RealType, Policy>& dist, const RealType& x)
 490:          {
 491:             BOOST_MATH_STD_USING
 492:             constexpr auto function = "pdf(non_central_chi_squared_distribution<%1%>, %1%)";
 493:             typedef typename policies::evaluation<RealType, Policy>::type value_type;
 494:             typedef typename policies::normalise<
 495:                Policy,
 496:                policies::promote_float<false>,
 497:                policies::promote_double<false>,
 498:                policies::discrete_quantile<>,
 499:                policies::assert_undefined<> >::type forwarding_policy;
 500: 
 501:             value_type k = dist.degrees_of_freedom();
 502:             value_type l = dist.non_centrality();
 503:             value_type r;
 504:             if(!detail::check_df(
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 505-528 / 第 505-528 行
~~~cpp
 505:                function,
 506:                k, &r, Policy())
 507:                ||
 508:             !detail::check_non_centrality(
 509:                function,
 510:                l,
 511:                &r,
 512:                Policy())
 513:                ||
 514:             !detail::check_positive_x(
 515:                function,
 516:                (value_type)x,
 517:                &r,
 518:                Policy()))
 519:                   return static_cast<RealType>(r);
 520: 
 521:          if(l == 0)
 522:             return pdf(boost::math::chi_squared_distribution<RealType, forwarding_policy>(dist.degrees_of_freedom()), x);
 523: 
 524:          // Special case:
 525:          if(x == 0)
 526:             return 0;
 527:          if(l > 50)
 528:          {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 529-552 / 第 529-552 行
~~~cpp
 529:             r = non_central_chi_square_pdf(static_cast<value_type>(x), k, l, forwarding_policy());
 530:          }
 531:          else
 532:          {
 533:             r = log(x / l) * (k / 4 - 0.5f) - (x + l) / 2;
 534:             if(fabs(r) >= tools::log_max_value<RealType>() / 4)
 535:             {
 536:                r = non_central_chi_square_pdf(static_cast<value_type>(x), k, l, forwarding_policy());
 537:             }
 538:             else
 539:             {
 540:                r = exp(r);
 541:                r = 0.5f * r
 542:                   * boost::math::cyl_bessel_i(k/2 - 1, sqrt(l * x), forwarding_policy());
 543:             }
 544:          }
 545:          return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 546:                r,
 547:                function);
 548:          }
 549: 
 550:          template <class RealType, class Policy>
 551:          struct degrees_of_freedom_finder
 552:          {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as non_central_chi_square_pdf, exp, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 non_central_chi_square_pdf, exp, ...。

### Lines 553-576 / 第 553-576 行
~~~cpp
 553:             BOOST_MATH_GPU_ENABLED degrees_of_freedom_finder(
 554:                RealType lam_, RealType x_, RealType p_, bool c)
 555:                : lam(lam_), x(x_), p(p_), comp(c) {}
 556: 
 557:             BOOST_MATH_GPU_ENABLED RealType operator()(const RealType& v)
 558:             {
 559:                non_central_chi_squared_distribution<RealType, Policy> d(v, lam);
 560:                return comp ?
 561:                   RealType(p - cdf(complement(d, x)))
 562:                   : RealType(cdf(d, x) - p);
 563:             }
 564:          private:
 565:             RealType lam;
 566:             RealType x;
 567:             RealType p;
 568:             bool comp;
 569:          };
 570: 
 571:          template <class RealType, class Policy>
 572:          BOOST_MATH_GPU_ENABLED inline RealType find_degrees_of_freedom(
 573:             RealType lam, RealType x, RealType p, RealType q, const Policy& pol)
 574:          {
 575:             constexpr auto function = "non_central_chi_squared<%1%>::find_degrees_of_freedom";
 576:             if((p == 0) || (q == 0))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as lam, d, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 lam, d, ...。

### Lines 577-600 / 第 577-600 行
~~~cpp
 577:             {
 578:                //
 579:                // Can't a thing if one of p and q is zero:
 580:                //
 581:                return policies::raise_evaluation_error<RealType>(function, "Can't find degrees of freedom when the probability is 0 or 1, only possible answer is %1%", // LCOV_EXCL_LINE
 582:                   RealType(boost::math::numeric_limits<RealType>::quiet_NaN()), Policy()); // LCOV_EXCL_LINE
 583:             }
 584:             degrees_of_freedom_finder<RealType, Policy> f(lam, x, p < q ? p : q, p < q ? false : true);
 585:             tools::eps_tolerance<RealType> tol(policies::digits<RealType, Policy>());
 586:             boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 587:             //
 588:             // Pick an initial guess that we know will give us a probability
 589:             // right around 0.5.
 590:             //
 591:             RealType guess = x - lam;
 592:             if(guess < 1)
 593:                guess = 1;
 594:             boost::math::pair<RealType, RealType> ir = tools::bracket_and_solve_root(
 595:                f, guess, RealType(2), false, tol, max_iter, pol);
 596:             RealType result = ir.first + (ir.second - ir.first) / 2;
 597:             if(max_iter >= policies::get_max_root_iterations<Policy>())
 598:             {
 599:                return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 600:                   " or there is no answer to problem.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
~~~
- **EN:** This range declares or defines callable logic such as RealType, f, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 RealType, f, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 601-624 / 第 601-624 行
~~~cpp
 601:             }
 602:             return result;
 603:          }
 604: 
 605:          template <class RealType, class Policy>
 606:          struct non_centrality_finder
 607:          {
 608:             BOOST_MATH_GPU_ENABLED non_centrality_finder(
 609:                RealType v_, RealType x_, RealType p_, bool c)
 610:                : v(v_), x(x_), p(p_), comp(c) {}
 611: 
 612:             BOOST_MATH_GPU_ENABLED RealType operator()(const RealType& lam)
 613:             {
 614:                non_central_chi_squared_distribution<RealType, Policy> d(v, lam);
 615:                return comp ?
 616:                   RealType(p - cdf(complement(d, x)))
 617:                   : RealType(cdf(d, x) - p);
 618:             }
 619:          private:
 620:             RealType v;
 621:             RealType x;
 622:             RealType p;
 623:             bool comp;
 624:          };
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as v, d, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 v, d, ...。

### Lines 625-648 / 第 625-648 行
~~~cpp
 625: 
 626:          template <class RealType, class Policy>
 627:          BOOST_MATH_GPU_ENABLED inline RealType find_non_centrality(
 628:             RealType v, RealType x, RealType p, RealType q, const Policy& pol)
 629:          {
 630:             constexpr auto function = "non_central_chi_squared<%1%>::find_non_centrality";
 631:             if((p == 0) || (q == 0))
 632:             {
 633:                //
 634:                // Can't do a thing if one of p and q is zero:
 635:                //
 636:                return policies::raise_evaluation_error<RealType>(function, "Can't find non centrality parameter when the probability is 0 or 1, only possible answer is %1%", // LCOV_EXCL_LINE
 637:                   RealType(boost::math::numeric_limits<RealType>::quiet_NaN()), Policy()); // LCOV_EXCL_LINE
 638:             }
 639:             non_centrality_finder<RealType, Policy> f(v, x, p < q ? p : q, p < q ? false : true);
 640:             tools::eps_tolerance<RealType> tol(policies::digits<RealType, Policy>());
 641:             boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 642:             //
 643:             // Pick an initial guess that we know will give us a probability
 644:             // right around 0.5.
 645:             //
 646:             RealType guess = x - v;
 647:             if(guess < 1)
 648:                guess = 1;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as RealType, f, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 RealType, f, ...。

### Lines 649-672 / 第 649-672 行
~~~cpp
 649:             boost::math::pair<RealType, RealType> ir = tools::bracket_and_solve_root(
 650:                f, guess, RealType(2), false, tol, max_iter, pol);
 651:             RealType result = ir.first + (ir.second - ir.first) / 2;
 652:             if(max_iter >= policies::get_max_root_iterations<Policy>())
 653:             {
 654:                return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 655:                   " or there is no answer to problem.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
 656:             }
 657:             return result;
 658:          }
 659: 
 660:       }
 661: 
 662:       template <class RealType = double, class Policy = policies::policy<> >
 663:       class non_central_chi_squared_distribution
 664:       {
 665:       public:
 666:          typedef RealType value_type;
 667:          typedef Policy policy_type;
 668: 
 669:          BOOST_MATH_GPU_ENABLED non_central_chi_squared_distribution(RealType df_, RealType lambda) : df(df_), ncp(lambda)
 670:          {
 671:             constexpr auto function = "boost::math::non_central_chi_squared_distribution<%1%>::non_central_chi_squared_distribution(%1%,%1%)";
 672:             RealType r;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as RealType, Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 RealType, Policy。

### Lines 673-696 / 第 673-696 行
~~~cpp
 673:             detail::check_df(
 674:                function,
 675:                df, &r, Policy());
 676:             detail::check_non_centrality(
 677:                function,
 678:                ncp,
 679:                &r,
 680:                Policy());
 681:          } // non_central_chi_squared_distribution constructor.
 682: 
 683:          BOOST_MATH_GPU_ENABLED RealType degrees_of_freedom() const
 684:          { // Private data getter function.
 685:             return df;
 686:          }
 687:          BOOST_MATH_GPU_ENABLED RealType non_centrality() const
 688:          { // Private data getter function.
 689:             return ncp;
 690:          }
 691:          BOOST_MATH_GPU_ENABLED static RealType find_degrees_of_freedom(RealType lam, RealType x, RealType p)
 692:          {
 693:             constexpr auto function = "non_central_chi_squared<%1%>::find_degrees_of_freedom";
 694:             typedef typename policies::evaluation<RealType, Policy>::type eval_type;
 695:             typedef typename policies::normalise<
 696:                Policy,
~~~
- **EN:** This range declares or defines callable logic such as Policy. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 697-720 / 第 697-720 行
~~~cpp
 697:                policies::promote_float<false>,
 698:                policies::promote_double<false>,
 699:                policies::discrete_quantile<>,
 700:                policies::assert_undefined<> >::type forwarding_policy;
 701:             eval_type result = detail::find_degrees_of_freedom(
 702:                static_cast<eval_type>(lam),
 703:                static_cast<eval_type>(x),
 704:                static_cast<eval_type>(p),
 705:                static_cast<eval_type>(1-p),
 706:                forwarding_policy());
 707:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 708:                result,
 709:                function);
 710:          }
 711:          template <class A, class B, class C>
 712:          BOOST_MATH_GPU_ENABLED static RealType find_degrees_of_freedom(const complemented3_type<A,B,C>& c)
 713:          {
 714:             constexpr auto function = "non_central_chi_squared<%1%>::find_degrees_of_freedom";
 715:             typedef typename policies::evaluation<RealType, Policy>::type eval_type;
 716:             typedef typename policies::normalise<
 717:                Policy,
 718:                policies::promote_float<false>,
 719:                policies::promote_double<false>,
 720:                policies::discrete_quantile<>,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `A` as part of the file's main abstraction. This range declares or defines callable logic such as forwarding_policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `A`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 forwarding_policy。

### Lines 721-744 / 第 721-744 行
~~~cpp
 721:                policies::assert_undefined<> >::type forwarding_policy;
 722:             eval_type result = detail::find_degrees_of_freedom(
 723:                static_cast<eval_type>(c.dist),
 724:                static_cast<eval_type>(c.param1),
 725:                static_cast<eval_type>(1-c.param2),
 726:                static_cast<eval_type>(c.param2),
 727:                forwarding_policy());
 728:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 729:                result,
 730:                function);
 731:          }
 732:          BOOST_MATH_GPU_ENABLED static RealType find_non_centrality(RealType v, RealType x, RealType p)
 733:          {
 734:             constexpr auto function = "non_central_chi_squared<%1%>::find_non_centrality";
 735:             typedef typename policies::evaluation<RealType, Policy>::type eval_type;
 736:             typedef typename policies::normalise<
 737:                Policy,
 738:                policies::promote_float<false>,
 739:                policies::promote_double<false>,
 740:                policies::discrete_quantile<>,
 741:                policies::assert_undefined<> >::type forwarding_policy;
 742:             eval_type result = detail::find_non_centrality(
 743:                static_cast<eval_type>(v),
 744:                static_cast<eval_type>(x),
~~~
- **EN:** This range declares or defines callable logic such as forwarding_policy. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 forwarding_policy。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 745-768 / 第 745-768 行
~~~cpp
 745:                static_cast<eval_type>(p),
 746:                static_cast<eval_type>(1-p),
 747:                forwarding_policy());
 748:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 749:                result,
 750:                function);
 751:          }
 752:          template <class A, class B, class C>
 753:          BOOST_MATH_GPU_ENABLED static RealType find_non_centrality(const complemented3_type<A,B,C>& c)
 754:          {
 755:             constexpr auto function = "non_central_chi_squared<%1%>::find_non_centrality";
 756:             typedef typename policies::evaluation<RealType, Policy>::type eval_type;
 757:             typedef typename policies::normalise<
 758:                Policy,
 759:                policies::promote_float<false>,
 760:                policies::promote_double<false>,
 761:                policies::discrete_quantile<>,
 762:                policies::assert_undefined<> >::type forwarding_policy;
 763:             eval_type result = detail::find_non_centrality(
 764:                static_cast<eval_type>(c.dist),
 765:                static_cast<eval_type>(c.param1),
 766:                static_cast<eval_type>(1-c.param2),
 767:                static_cast<eval_type>(c.param2),
 768:                forwarding_policy());
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `A` as part of the file's main abstraction. This range declares or defines callable logic such as forwarding_policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `A`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 forwarding_policy。

### Lines 769-792 / 第 769-792 行
~~~cpp
 769:             return policies::checked_narrowing_cast<RealType, forwarding_policy>(
 770:                result,
 771:                function);
 772:          }
 773:       private:
 774:          // Data member, initialized by constructor.
 775:          RealType df; // degrees of freedom.
 776:          RealType ncp; // non-centrality parameter
 777:       }; // template <class RealType, class Policy> class non_central_chi_squared_distribution
 778: 
 779:       typedef non_central_chi_squared_distribution<double> non_central_chi_squared; // Reserved name of type double.
 780: 
 781:       #ifdef __cpp_deduction_guides
 782:       template <class RealType>
 783:       non_central_chi_squared_distribution(RealType,RealType)->non_central_chi_squared_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 784:       #endif
 785: 
 786:       // Non-member functions to give properties of the distribution.
 787: 
 788:       template <class RealType, class Policy>
 789:       BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const non_central_chi_squared_distribution<RealType, Policy>& /* dist */)
 790:       { // Range of permissible values for random variable k.
 791:          using boost::math::tools::max_value;
 792:          return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>()); // Max integer?
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 793-816 / 第 793-816 行
~~~cpp
 793:       }
 794: 
 795:       template <class RealType, class Policy>
 796:       BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const non_central_chi_squared_distribution<RealType, Policy>& /* dist */)
 797:       { // Range of supported values for random variable k.
 798:          // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 799:          using boost::math::tools::max_value;
 800:          return boost::math::pair<RealType, RealType>(static_cast<RealType>(0),  max_value<RealType>());
 801:       }
 802: 
 803:       template <class RealType, class Policy>
 804:       BOOST_MATH_GPU_ENABLED inline RealType mean(const non_central_chi_squared_distribution<RealType, Policy>& dist)
 805:       { // Mean of poisson distribution = lambda.
 806:          constexpr auto function = "boost::math::non_central_chi_squared_distribution<%1%>::mean()";
 807:          RealType k = dist.degrees_of_freedom();
 808:          RealType l = dist.non_centrality();
 809:          RealType r;
 810:          if(!detail::check_df(
 811:             function,
 812:             k, &r, Policy())
 813:             ||
 814:          !detail::check_non_centrality(
 815:             function,
 816:             l,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 817-840 / 第 817-840 行
~~~cpp
 817:             &r,
 818:             Policy()))
 819:                return static_cast<RealType>(r);
 820:          return k + l;
 821:       } // mean
 822: 
 823:       template <class RealType, class Policy>
 824:       BOOST_MATH_GPU_ENABLED inline RealType mode(const non_central_chi_squared_distribution<RealType, Policy>& dist)
 825:       { // mode.
 826:          constexpr auto function = "mode(non_central_chi_squared_distribution<%1%> const&)";
 827: 
 828:          RealType k = dist.degrees_of_freedom();
 829:          RealType l = dist.non_centrality();
 830:          RealType r;
 831:          if(!detail::check_df(
 832:             function,
 833:             k, &r, Policy())
 834:             ||
 835:          !detail::check_non_centrality(
 836:             function,
 837:             l,
 838:             &r,
 839:             Policy()))
 840:                return static_cast<RealType>(r);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 841-864 / 第 841-864 行
~~~cpp
 841:          bool asymptotic_mode = k < l/4;
 842:          RealType starting_point = asymptotic_mode ? k + l - RealType(3) : RealType(1) + k;
 843:          return detail::generic_find_mode(dist, starting_point, function);
 844:       }
 845: 
 846:       template <class RealType, class Policy>
 847:       BOOST_MATH_GPU_ENABLED inline RealType variance(const non_central_chi_squared_distribution<RealType, Policy>& dist)
 848:       { // variance.
 849:          constexpr auto function = "boost::math::non_central_chi_squared_distribution<%1%>::variance()";
 850:          RealType k = dist.degrees_of_freedom();
 851:          RealType l = dist.non_centrality();
 852:          RealType r;
 853:          if(!detail::check_df(
 854:             function,
 855:             k, &r, Policy())
 856:             ||
 857:          !detail::check_non_centrality(
 858:             function,
 859:             l,
 860:             &r,
 861:             Policy()))
 862:                return static_cast<RealType>(r);
 863:          return 2 * (2 * l + k);
 864:       }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 865-888 / 第 865-888 行
~~~cpp
 865: 
 866:       // RealType standard_deviation(const non_central_chi_squared_distribution<RealType, Policy>& dist)
 867:       // standard_deviation provided by derived accessors.
 868: 
 869:       template <class RealType, class Policy>
 870:       BOOST_MATH_GPU_ENABLED inline RealType skewness(const non_central_chi_squared_distribution<RealType, Policy>& dist)
 871:       { // skewness = sqrt(l).
 872:          constexpr auto function = "boost::math::non_central_chi_squared_distribution<%1%>::skewness()";
 873:          RealType k = dist.degrees_of_freedom();
 874:          RealType l = dist.non_centrality();
 875:          RealType r;
 876:          if(!detail::check_df(
 877:             function,
 878:             k, &r, Policy())
 879:             ||
 880:          !detail::check_non_centrality(
 881:             function,
 882:             l,
 883:             &r,
 884:             Policy()))
 885:                return static_cast<RealType>(r);
 886:          BOOST_MATH_STD_USING
 887:             return pow(2 / (k + 2 * l), RealType(3)/2) * (k + 3 * l);
 888:       }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 889-912 / 第 889-912 行
~~~cpp
 889: 
 890:       template <class RealType, class Policy>
 891:       BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const non_central_chi_squared_distribution<RealType, Policy>& dist)
 892:       {
 893:          constexpr auto function = "boost::math::non_central_chi_squared_distribution<%1%>::kurtosis_excess()";
 894:          RealType k = dist.degrees_of_freedom();
 895:          RealType l = dist.non_centrality();
 896:          RealType r;
 897:          if(!detail::check_df(
 898:             function,
 899:             k, &r, Policy())
 900:             ||
 901:          !detail::check_non_centrality(
 902:             function,
 903:             l,
 904:             &r,
 905:             Policy()))
 906:                return static_cast<RealType>(r);
 907:          return 12 * (k + 4 * l) / ((k + 2 * l) * (k + 2 * l));
 908:       } // kurtosis_excess
 909: 
 910:       template <class RealType, class Policy>
 911:       BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const non_central_chi_squared_distribution<RealType, Policy>& dist)
 912:       {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 913-936 / 第 913-936 行
~~~cpp
 913:          return kurtosis_excess(dist) + 3;
 914:       }
 915: 
 916:       template <class RealType, class Policy>
 917:       BOOST_MATH_GPU_ENABLED inline RealType pdf(const non_central_chi_squared_distribution<RealType, Policy>& dist, const RealType& x)
 918:       { // Probability Density/Mass Function.
 919:          return detail::nccs_pdf(dist, x);
 920:       } // pdf
 921: 
 922:       template <class RealType, class Policy>
 923:       BOOST_MATH_GPU_ENABLED RealType cdf(const non_central_chi_squared_distribution<RealType, Policy>& dist, const RealType& x)
 924:       {
 925:          constexpr auto function = "boost::math::non_central_chi_squared_distribution<%1%>::cdf(%1%)";
 926:          RealType k = dist.degrees_of_freedom();
 927:          RealType l = dist.non_centrality();
 928:          RealType r;
 929:          if(!detail::check_df(
 930:             function,
 931:             k, &r, Policy())
 932:             ||
 933:          !detail::check_non_centrality(
 934:             function,
 935:             l,
 936:             &r,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 937-960 / 第 937-960 行
~~~cpp
 937:             Policy())
 938:             ||
 939:          !detail::check_positive_x(
 940:             function,
 941:             x,
 942:             &r,
 943:             Policy()))
 944:                return static_cast<RealType>(r);
 945: 
 946:          return detail::non_central_chi_squared_cdf(x, k, l, false, Policy());
 947:       } // cdf
 948: 
 949:       template <class RealType, class Policy>
 950:       BOOST_MATH_GPU_ENABLED RealType cdf(const complemented2_type<non_central_chi_squared_distribution<RealType, Policy>, RealType>& c)
 951:       { // Complemented Cumulative Distribution Function
 952:          constexpr auto function = "boost::math::non_central_chi_squared_distribution<%1%>::cdf(%1%)";
 953:          non_central_chi_squared_distribution<RealType, Policy> const& dist = c.dist;
 954:          RealType x = c.param;
 955:          RealType k = dist.degrees_of_freedom();
 956:          RealType l = dist.non_centrality();
 957:          RealType r;
 958:          if(!detail::check_df(
 959:             function,
 960:             k, &r, Policy())
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, non_centrality.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, non_centrality。

### Lines 961-984 / 第 961-984 行
~~~cpp
 961:             ||
 962:          !detail::check_non_centrality(
 963:             function,
 964:             l,
 965:             &r,
 966:             Policy())
 967:             ||
 968:          !detail::check_positive_x(
 969:             function,
 970:             x,
 971:             &r,
 972:             Policy()))
 973:                return static_cast<RealType>(r);
 974: 
 975:          return detail::non_central_chi_squared_cdf(x, k, l, true, Policy());
 976:       } // ccdf
 977: 
 978:       template <class RealType, class Policy>
 979:       BOOST_MATH_GPU_ENABLED inline RealType quantile(const non_central_chi_squared_distribution<RealType, Policy>& dist, const RealType& p)
 980:       { // Quantile (or Percent Point) function.
 981:          return detail::nccs_quantile(dist, p, false);
 982:       } // quantile
 983: 
 984:       template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 985-998 / 第 985-998 行
~~~cpp
 985:       BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<non_central_chi_squared_distribution<RealType, Policy>, RealType>& c)
 986:       { // Quantile (or Percent Point) function.
 987:          return detail::nccs_quantile(c.dist, c.param, true);
 988:       } // quantile complement.
 989: 
 990:    } // namespace math
 991: } // namespace boost
 992: 
 993: // This include must be at the end, *after* the accessors
 994: // for this distribution have been defined, in order to
 995: // keep compilers that support two-phase lookup happy.
 996: #include <boost/math/distributions/detail/derived_accessors.hpp>
 997: 
 998: #endif // BOOST_MATH_SPECIAL_NON_CENTRAL_CHI_SQUARE_HPP
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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/numeric_limits.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/bessel.hpp, boost/math/special_functions/round.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/chi_squared.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/special_functions/fpclassify.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `llround, boost::math::gamma_p_derivative, boost::math::gamma_q, exp, lterm, errorf, boost::math::gamma_p, gamma_p_derivative, ...`
