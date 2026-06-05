# hyperexponential.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/hyperexponential.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the hyperexponential distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 hyperexponential 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  Copyright 2014 Marco Guazzone (marco.guazzone@gmail.com)
   2: //
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: //
   7: // This module implements the Hyper-Exponential distribution.
   8: //
   9: // References:
  10: // - "Queueing Theory in Manufacturing Systems Analysis and Design" by H.T. Papadopolous, C. Heavey and J. Browne (Chapman & Hall/CRC, 1993)
  11: // - http://reference.wolfram.com/language/ref/HyperexponentialDistribution.html
  12: // - http://en.wikipedia.org/wiki/Hyperexponential_distribution
  13: //
  14: 
  15: #ifndef BOOST_MATH_DISTRIBUTIONS_HYPEREXPONENTIAL_HPP
  16: #define BOOST_MATH_DISTRIBUTIONS_HYPEREXPONENTIAL_HPP
  17: 
  18: #include <boost/math/tools/cxx03_warn.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/cxx03_warn.hpp so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/cxx03_warn.hpp 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/distributions/complement.hpp>
  20: #include <boost/math/distributions/detail/common_error_handling.hpp>
  21: #include <boost/math/distributions/exponential.hpp>
  22: #include <boost/math/policies/policy.hpp>
  23: #include <boost/math/special_functions/fpclassify.hpp>
  24: #include <boost/math/tools/precision.hpp>
  25: #include <boost/math/tools/roots.hpp>
  26: #include <boost/math/tools/is_detected.hpp>
  27: #include <cstddef>
  28: #include <iterator>
  29: #include <limits>
  30: #include <numeric>
  31: #include <utility>
  32: #include <vector>
  33: #include <type_traits>
  34: #include <initializer_list>
  35: 
  36: 
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/exponential.hpp, ... so the surrounding code can use external declarations. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此代码块引入了 boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/exponential.hpp, ... 等依赖，使周围代码可以使用外部声明。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: #ifdef _MSC_VER
  38: # pragma warning (push)
  39: # pragma warning(disable:4127) // conditional expression is constant
  40: # pragma warning(disable:4389) // '==' : signed/unsigned mismatch in test_tools
  41: #endif // _MSC_VER
  42: 
  43: namespace boost { namespace math {
  44: 
  45: namespace detail {
  46: 
  47: template <typename Dist>
  48: typename Dist::value_type generic_quantile(const Dist& dist, const typename Dist::value_type& p, const typename Dist::value_type& guess, bool comp, const char* function);
  49: 
  50: } // Namespace detail
  51: 
  52: 
  53: template <typename RealT, typename PolicyT>
  54: class hyperexponential_distribution;
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `hyperexponential_distribution` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `hyperexponential_distribution`，作为该文件核心抽象的一部分。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: 
  56: 
  57: namespace /*<unnamed>*/ { namespace hyperexp_detail {
  58: 
  59: template <typename T>
  60: void normalize(std::vector<T>& v)
  61: {
  62:    if(!v.size())
  63:       return;  // Our error handlers will get this later
  64:     const T sum = std::accumulate(v.begin(), v.end(), static_cast<T>(0));
  65:     T final_sum = 0;
  66:     const typename std::vector<T>::iterator end = --v.end();
  67:     for (typename std::vector<T>::iterator it = v.begin();
  68:          it != end;
  69:          ++it)
  70:     {
  71:         *it /= sum;
  72:         final_sum += *it;
~~~
- **EN:** The code enters namespace scope (hyperexp_detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as std::accumulate, end.
- **CN:** 代码进入命名空间作用域（hyperexp_detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 std::accumulate, end。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:     }
  74:     *end = 1 - final_sum;  // avoids round off errors, ensures the probs really do sum to 1.
  75: }
  76: 
  77: template <typename RealT, typename PolicyT>
  78: bool check_probabilities(char const* function, std::vector<RealT> const& probabilities, RealT* presult, PolicyT const& pol)
  79: {
  80:     BOOST_MATH_STD_USING
  81:     const std::size_t n = probabilities.size();
  82:     RealT sum = 0;
  83:     for (std::size_t i = 0; i < n; ++i)
  84:     {
  85:         if (probabilities[i] < 0
  86:             || probabilities[i] > 1
  87:             || !(boost::math::isfinite)(probabilities[i]))
  88:         {
  89:             *presult = policies::raise_domain_error<RealT>(function,
  90:                                                            "The elements of parameter \"probabilities\" must be >= 0 and <= 1, but at least one of them was: %1%.",
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as size. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 size。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:                                                            probabilities[i],
  92:                                                            pol);
  93:             return false;
  94:         }
  95:         sum += probabilities[i];
  96:     }
  97: 
  98:     //
  99:     // We try to keep phase probabilities correctly normalized in the distribution constructors,
 100:     // however in practice we have to allow for a very slight divergence from a sum of exactly 1:
 101:     //
 102:     if (fabs(sum - 1) > tools::epsilon<RealT>() * 2)
 103:     {
 104:         *presult = policies::raise_domain_error<RealT>(function,
 105:                                                        "The elements of parameter \"probabilities\" must sum to 1, but their sum is: %1%.",
 106:                                                        sum,
 107:                                                        pol);
 108:         return false;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:     }
 110: 
 111:     return true;
 112: }
 113: 
 114: template <typename RealT, typename PolicyT>
 115: bool check_rates(char const* function, std::vector<RealT> const& rates, RealT* presult, PolicyT const& pol)
 116: {
 117:     const std::size_t n = rates.size();
 118:     for (std::size_t i = 0; i < n; ++i)
 119:     {
 120:         if (rates[i] <= 0
 121:             || !(boost::math::isfinite)(rates[i]))
 122:         {
 123:             *presult = policies::raise_domain_error<RealT>(function,
 124:                                                            "The elements of parameter \"rates\" must be > 0, but at least one of them is: %1%.",
 125:                                                            rates[i],
 126:                                                            pol);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as size. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 size。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:             return false;
 128:         }
 129:     }
 130:     return true;
 131: }
 132: 
 133: template <typename RealT, typename PolicyT>
 134: bool check_dist(char const* function, std::vector<RealT> const& probabilities, std::vector<RealT> const& rates, RealT* presult, PolicyT const& pol)
 135: {
 136:     BOOST_MATH_STD_USING
 137:     if (probabilities.size() != rates.size())
 138:     {
 139:         *presult = policies::raise_domain_error<RealT>(function,
 140:                                                        R"(The parameters "probabilities" and "rates" must have the same length, but their size differ by: %1%.)",
 141:                                                        fabs(static_cast<RealT>(probabilities.size())-static_cast<RealT>(rates.size())),
 142:                                                        pol);
 143:         return false;
 144:     }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145: 
 146:     return check_probabilities(function, probabilities, presult, pol)
 147:            && check_rates(function, rates, presult, pol);
 148: }
 149: 
 150: template <typename RealT, typename PolicyT>
 151: bool check_x(char const* function, RealT x, RealT* presult, PolicyT const& pol)
 152: {
 153:     if (x < 0 || (boost::math::isnan)(x))
 154:     {
 155:         *presult = policies::raise_domain_error<RealT>(function, "The random variable must be >= 0, but is: %1%.", x, pol);
 156:         return false;
 157:     }
 158:     return true;
 159: }
 160: 
 161: template <typename RealT, typename PolicyT>
 162: bool check_probability(char const* function, RealT p, RealT* presult, PolicyT const& pol)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as check_rates. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 check_rates。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163: {
 164:     if (p < 0 || p > 1 || (boost::math::isnan)(p))
 165:     {
 166:         *presult = policies::raise_domain_error<RealT>(function, "The probability be >= 0 and <= 1, but is: %1%.", p, pol);
 167:         return false;
 168:     }
 169:     return true;
 170: }
 171: 
 172: template <typename RealT, typename PolicyT>
 173: RealT quantile_impl(hyperexponential_distribution<RealT, PolicyT> const& dist, RealT const& p, bool comp)
 174: {
 175:     // Don't have a closed form so try to numerically solve the inverse CDF...
 176: 
 177:     typedef typename policies::evaluation<RealT, PolicyT>::type value_type;
 178:     typedef typename policies::normalise<PolicyT,
 179:                                          policies::promote_float<false>,
 180:                                          policies::promote_double<false>,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:                                          policies::discrete_quantile<>,
 182:                                          policies::assert_undefined<> >::type forwarding_policy;
 183: 
 184:     static const char* function = comp ? "boost::math::quantile(const boost::math::complemented2_type<boost::math::hyperexponential_distribution<%1%>, %1%>&)"
 185:                                        : "boost::math::quantile(const boost::math::hyperexponential_distribution<%1%>&, %1%)";
 186: 
 187:     RealT result = 0;
 188: 
 189:     if (!check_probability(function, p, &result, PolicyT()))
 190:     {
 191:         return result;
 192:     }
 193: 
 194:     const std::size_t n = dist.num_phases();
 195:     const std::vector<RealT> probs = dist.probabilities();
 196:     const std::vector<RealT> rates = dist.rates();
 197: 
 198:     // A possible (but inaccurate) approximation is given below, where the
~~~
- **EN:** This range declares or defines callable logic such as num_phases, probabilities, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 num_phases, probabilities, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:     // quantile is given by the weighted sum of exponential quantiles:
 200:     RealT guess = 0;
 201:     if (comp)
 202:     {
 203:         for (std::size_t i = 0; i < n; ++i)
 204:         {
 205:             const exponential_distribution<RealT,PolicyT> exp(rates[i]);
 206: 
 207:             guess += probs[i]*quantile(complement(exp, p));
 208:         }
 209:     }
 210:     else
 211:     {
 212:         for (std::size_t i = 0; i < n; ++i)
 213:         {
 214:             const exponential_distribution<RealT,PolicyT> exp(rates[i]);
 215: 
 216:             guess += probs[i]*quantile(exp, p);
~~~
- **EN:** This range declares or defines callable logic such as exp, quantile. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp, quantile。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:         }
 218:     }
 219: 
 220:     // Fast return in case the Hyper-Exponential is essentially an Exponential
 221:     if (n == 1)
 222:     {
 223:         return guess;
 224:     }
 225: 
 226:     value_type q;
 227:     q = detail::generic_quantile(hyperexponential_distribution<RealT,forwarding_policy>(probs, rates),
 228:                                  p,
 229:                                  guess,
 230:                                  comp,
 231:                                  function);
 232: 
 233:     result = policies::checked_narrowing_cast<RealT,forwarding_policy>(q, function);
 234: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:     return result;
 236: }
 237: 
 238: }} // Namespace <unnamed>::hyperexp_detail
 239: 
 240: 
 241: template <typename RealT = double, typename PolicyT = policies::policy<> >
 242: class hyperexponential_distribution
 243: {
 244:     public: typedef RealT value_type;
 245:     public: typedef PolicyT policy_type;
 246: 
 247: 
 248:     public: hyperexponential_distribution()
 249:     : probs_(1, 1),
 250:       rates_(1, 1)
 251:     {
 252:         RealT err;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `hyperexponential_distribution` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `hyperexponential_distribution`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:         hyperexp_detail::check_dist("boost::math::hyperexponential_distribution<%1%>::hyperexponential_distribution",
 254:                                     probs_,
 255:                                     rates_,
 256:                                     &err,
 257:                                     PolicyT());
 258:     }
 259: 
 260:     // Four arg constructor: no ambiguity here, the arguments must be two pairs of iterators:
 261:     public: template <typename ProbIterT, typename RateIterT>
 262:             hyperexponential_distribution(ProbIterT prob_first, ProbIterT prob_last,
 263:                                           RateIterT rate_first, RateIterT rate_last)
 264:     : probs_(prob_first, prob_last),
 265:       rates_(rate_first, rate_last)
 266:     {
 267:         hyperexp_detail::normalize(probs_);
 268:         RealT err;
 269:         hyperexp_detail::check_dist("boost::math::hyperexponential_distribution<%1%>::hyperexponential_distribution",
 270:                                     probs_,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as PolicyT, hyperexp_detail::normalize. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 PolicyT, hyperexp_detail::normalize。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:                                     rates_,
 272:                                     &err,
 273:                                     PolicyT());
 274:     }
 275:     private: template <typename T, typename = void>
 276:              struct is_iterator
 277:              {
 278:                  static constexpr bool value = false;
 279:              };
 280: 
 281:              template <typename T>
 282:              struct is_iterator<T, boost::math::tools::void_t<typename std::iterator_traits<T>::difference_type>>
 283:              {
 284:                  // std::iterator_traits<T>::difference_type returns void for invalid types
 285:                  static constexpr bool value = !std::is_same<typename std::iterator_traits<T>::difference_type, void>::value;
 286:              };
 287: 
 288:     // Two arg constructor from 2 ranges, we SFINAE this out of existence if
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `is_iterator` as part of the file's main abstraction. This range declares or defines callable logic such as PolicyT.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `is_iterator`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 PolicyT。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:     // either argument type is incrementable as in that case the type is
 290:     // probably an iterator:
 291:     public: template <typename ProbRangeT, typename RateRangeT,
 292:                       typename std::enable_if<!is_iterator<ProbRangeT>::value &&
 293:                                               !is_iterator<RateRangeT>::value, bool>::type = true>
 294:             hyperexponential_distribution(ProbRangeT const& prob_range,
 295:                                           RateRangeT const& rate_range)
 296:     : probs_(std::begin(prob_range), std::end(prob_range)),
 297:       rates_(std::begin(rate_range), std::end(rate_range))
 298:     {
 299:         hyperexp_detail::normalize(probs_);
 300: 
 301:         RealT err;
 302:         hyperexp_detail::check_dist("boost::math::hyperexponential_distribution<%1%>::hyperexponential_distribution",
 303:                                     probs_,
 304:                                     rates_,
 305:                                     &err,
 306:                                     PolicyT());
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as hyperexp_detail::normalize, PolicyT. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 hyperexp_detail::normalize, PolicyT。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:     }
 308: 
 309:     // Two arg constructor for a pair of iterators: we SFINAE this out of
 310:     // existence if neither argument types are incrementable.
 311:     // Note that we allow different argument types here to allow for
 312:     // construction from an array plus a pointer into that array.
 313:     public: template <typename RateIterT, typename RateIterT2,
 314:                       typename std::enable_if<is_iterator<RateIterT>::value ||
 315:                                               is_iterator<RateIterT2>::value, bool>::type = true>
 316:             hyperexponential_distribution(RateIterT const& rate_first,
 317:                                           RateIterT2 const& rate_last)
 318:     : probs_(std::distance(rate_first, rate_last), 1), // will be normalized below
 319:       rates_(rate_first, rate_last)
 320:     {
 321:         hyperexp_detail::normalize(probs_);
 322: 
 323:         RealT err;
 324:         hyperexp_detail::check_dist("boost::math::hyperexponential_distribution<%1%>::hyperexponential_distribution",
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as hyperexp_detail::normalize. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 hyperexp_detail::normalize。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:                                     probs_,
 326:                                     rates_,
 327:                                     &err,
 328:                                     PolicyT());
 329:     }
 330: 
 331:       // Initializer list constructor: allows for construction from array literals:
 332: public: hyperexponential_distribution(std::initializer_list<RealT> l1, std::initializer_list<RealT> l2)
 333:       : probs_(l1.begin(), l1.end()),
 334:         rates_(l2.begin(), l2.end())
 335:       {
 336:          hyperexp_detail::normalize(probs_);
 337: 
 338:          RealT err;
 339:          hyperexp_detail::check_dist("boost::math::hyperexponential_distribution<%1%>::hyperexponential_distribution",
 340:             probs_,
 341:             rates_,
 342:             &err,
~~~
- **EN:** This range declares or defines callable logic such as PolicyT, hyperexp_detail::normalize. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 PolicyT, hyperexp_detail::normalize。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:             PolicyT());
 344:       }
 345: 
 346: public: hyperexponential_distribution(std::initializer_list<RealT> l1)
 347:       : probs_(l1.size(), 1),
 348:         rates_(l1.begin(), l1.end())
 349:       {
 350:          hyperexp_detail::normalize(probs_);
 351: 
 352:          RealT err;
 353:          hyperexp_detail::check_dist("boost::math::hyperexponential_distribution<%1%>::hyperexponential_distribution",
 354:             probs_,
 355:             rates_,
 356:             &err,
 357:             PolicyT());
 358:       }
 359: 
 360:     // Single argument constructor: argument must be a range.
~~~
- **EN:** This range declares or defines callable logic such as PolicyT, hyperexp_detail::normalize. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 PolicyT, hyperexp_detail::normalize。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:     public: template <typename RateRangeT>
 362:     hyperexponential_distribution(RateRangeT const& rate_range)
 363:     : probs_(std::distance(std::begin(rate_range), std::end(rate_range)), 1), // will be normalized below
 364:       rates_(std::begin(rate_range), std::end(rate_range))
 365:     {
 366:         hyperexp_detail::normalize(probs_);
 367: 
 368:         RealT err;
 369:         hyperexp_detail::check_dist("boost::math::hyperexponential_distribution<%1%>::hyperexponential_distribution",
 370:                                     probs_,
 371:                                     rates_,
 372:                                     &err,
 373:                                     PolicyT());
 374:     }
 375: 
 376:     public: std::vector<RealT> probabilities() const
 377:     {
 378:         return probs_;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as hyperexp_detail::normalize, PolicyT. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 hyperexp_detail::normalize, PolicyT。 return 语句会把计算结果或状态返回给调用方。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:     }
 380: 
 381:     public: std::vector<RealT> rates() const
 382:     {
 383:         return rates_;
 384:     }
 385: 
 386:     public: std::size_t num_phases() const
 387:     {
 388:         return rates_.size();
 389:     }
 390: 
 391: 
 392:     private: std::vector<RealT> probs_;
 393:     private: std::vector<RealT> rates_;
 394: }; // class hyperexponential_distribution
 395: 
 396: 
~~~
- **EN:** It introduces the class `hyperexponential_distribution` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 它引入了 class `hyperexponential_distribution`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397: // Convenient type synonym for double.
 398: typedef hyperexponential_distribution<double> hyperexponential;
 399: 
 400: 
 401: // Range of permissible values for random variable x
 402: template <typename RealT, typename PolicyT>
 403: std::pair<RealT,RealT> range(hyperexponential_distribution<RealT,PolicyT> const&)
 404: {
 405:     if (std::numeric_limits<RealT>::has_infinity)
 406:     {
 407:         return std::make_pair(static_cast<RealT>(0), std::numeric_limits<RealT>::infinity()); // 0 to +inf.
 408:     }
 409: 
 410:     return std::make_pair(static_cast<RealT>(0), tools::max_value<RealT>()); // 0 to +<max value>
 411: }
 412: 
 413: // Range of supported values for random variable x.
 414: // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415: template <typename RealT, typename PolicyT>
 416: std::pair<RealT,RealT> support(hyperexponential_distribution<RealT,PolicyT> const&)
 417: {
 418:     return std::make_pair(tools::min_value<RealT>(), tools::max_value<RealT>()); // <min value> to +<max value>.
 419: }
 420: 
 421: template <typename RealT, typename PolicyT>
 422: RealT pdf(hyperexponential_distribution<RealT, PolicyT> const& dist, RealT const& x)
 423: {
 424:     BOOST_MATH_STD_USING
 425:     RealT result = 0;
 426: 
 427:     if (!hyperexp_detail::check_x("boost::math::pdf(const boost::math::hyperexponential_distribution<%1%>&, %1%)", x, &result, PolicyT()))
 428:     {
 429:         return result;
 430:     }
 431: 
 432:     const std::size_t n = dist.num_phases();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as num_phases. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 num_phases。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:     const std::vector<RealT> probs = dist.probabilities();
 434:     const std::vector<RealT> rates = dist.rates();
 435: 
 436:     for (std::size_t i = 0; i < n; ++i)
 437:     {
 438:         const exponential_distribution<RealT,PolicyT> exp(rates[i]);
 439: 
 440:         result += probs[i]*pdf(exp, x);
 441:         //result += probs[i]*rates[i]*exp(-rates[i]*x);
 442:     }
 443: 
 444:     return result;
 445: }
 446: 
 447: template <typename RealT, typename PolicyT>
 448: RealT cdf(hyperexponential_distribution<RealT, PolicyT> const& dist, RealT const& x)
 449: {
 450:     RealT result = 0;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as probabilities, rates, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 probabilities, rates, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451: 
 452:     if (!hyperexp_detail::check_x("boost::math::cdf(const boost::math::hyperexponential_distribution<%1%>&, %1%)", x, &result, PolicyT()))
 453:     {
 454:         return result;
 455:     }
 456: 
 457:     const std::size_t n = dist.num_phases();
 458:     const std::vector<RealT> probs = dist.probabilities();
 459:     const std::vector<RealT> rates = dist.rates();
 460: 
 461:     for (std::size_t i = 0; i < n; ++i)
 462:     {
 463:         const exponential_distribution<RealT,PolicyT> exp(rates[i]);
 464: 
 465:         result += probs[i]*cdf(exp, x);
 466:     }
 467: 
 468:     return result;
~~~
- **EN:** This range declares or defines callable logic such as num_phases, probabilities, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 num_phases, probabilities, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469: }
 470: 
 471: template <typename RealT, typename PolicyT>
 472: RealT quantile(hyperexponential_distribution<RealT, PolicyT> const& dist, RealT const& p)
 473: {
 474:     return hyperexp_detail::quantile_impl(dist, p , false);
 475: }
 476: 
 477: template <typename RealT, typename PolicyT>
 478: RealT cdf(complemented2_type<hyperexponential_distribution<RealT,PolicyT>, RealT> const& c)
 479: {
 480:     RealT const& x = c.param;
 481:     hyperexponential_distribution<RealT,PolicyT> const& dist = c.dist;
 482: 
 483:     RealT result = 0;
 484: 
 485:     if (!hyperexp_detail::check_x("boost::math::cdf(boost::math::complemented2_type<const boost::math::hyperexponential_distribution<%1%>&, %1%>)", x, &result, PolicyT()))
 486:     {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:         return result;
 488:     }
 489: 
 490:     const std::size_t n = dist.num_phases();
 491:     const std::vector<RealT> probs = dist.probabilities();
 492:     const std::vector<RealT> rates = dist.rates();
 493: 
 494:     for (std::size_t i = 0; i < n; ++i)
 495:     {
 496:         const exponential_distribution<RealT,PolicyT> exp(rates[i]);
 497: 
 498:         result += probs[i]*cdf(complement(exp, x));
 499:     }
 500: 
 501:     return result;
 502: }
 503: 
 504: 
~~~
- **EN:** This range declares or defines callable logic such as num_phases, probabilities, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 num_phases, probabilities, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 return 语句会把计算结果或状态返回给调用方。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505: template <typename RealT, typename PolicyT>
 506: RealT quantile(complemented2_type<hyperexponential_distribution<RealT, PolicyT>, RealT> const& c)
 507: {
 508:     RealT const& p = c.param;
 509:     hyperexponential_distribution<RealT,PolicyT> const& dist = c.dist;
 510: 
 511:     return hyperexp_detail::quantile_impl(dist, p , true);
 512: }
 513: 
 514: template <typename RealT, typename PolicyT>
 515: RealT mean(hyperexponential_distribution<RealT, PolicyT> const& dist)
 516: {
 517:     RealT result = 0;
 518: 
 519:     const std::size_t n = dist.num_phases();
 520:     const std::vector<RealT> probs = dist.probabilities();
 521:     const std::vector<RealT> rates = dist.rates();
 522: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as num_phases, probabilities, .... Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 num_phases, probabilities, ...。 return 语句会把计算结果或状态返回给调用方。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:     for (std::size_t i = 0; i < n; ++i)
 524:     {
 525:         const exponential_distribution<RealT,PolicyT> exp(rates[i]);
 526: 
 527:         result += probs[i]*mean(exp);
 528:     }
 529: 
 530:     return result;
 531: }
 532: 
 533: template <typename RealT, typename PolicyT>
 534: RealT variance(hyperexponential_distribution<RealT, PolicyT> const& dist)
 535: {
 536:     RealT result = 0;
 537: 
 538:     const std::size_t n = dist.num_phases();
 539:     const std::vector<RealT> probs = dist.probabilities();
 540:     const std::vector<RealT> rates = dist.rates();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as exp, mean, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 exp, mean, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541: 
 542:     for (std::size_t i = 0; i < n; ++i)
 543:     {
 544:         result += probs[i]/(rates[i]*rates[i]);
 545:     }
 546: 
 547:     const RealT mean = boost::math::mean(dist);
 548: 
 549:     result = 2*result-mean*mean;
 550: 
 551:     return result;
 552: }
 553: 
 554: template <typename RealT, typename PolicyT>
 555: RealT skewness(hyperexponential_distribution<RealT,PolicyT> const& dist)
 556: {
 557:     BOOST_MATH_STD_USING
 558:     const std::size_t n = dist.num_phases();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as boost::math::mean, num_phases. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 boost::math::mean, num_phases。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 559-576 / 第 559-576 行
~~~cpp
 559:     const std::vector<RealT> probs = dist.probabilities();
 560:     const std::vector<RealT> rates = dist.rates();
 561: 
 562:     RealT s1 = 0; // \sum_{i=1}^n \frac{p_i}{\lambda_i}
 563:     RealT s2 = 0; // \sum_{i=1}^n \frac{p_i}{\lambda_i^2}
 564:     RealT s3 = 0; // \sum_{i=1}^n \frac{p_i}{\lambda_i^3}
 565:     for (std::size_t i = 0; i < n; ++i)
 566:     {
 567:         const RealT p = probs[i];
 568:         const RealT r = rates[i];
 569:         const RealT r2 = r*r;
 570:         const RealT r3 = r2*r;
 571: 
 572:         s1 += p/r;
 573:         s2 += p/r2;
 574:         s3 += p/r3;
 575:     }
 576: 
~~~
- **EN:** This range declares or defines callable logic such as probabilities, rates. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 probabilities, rates。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 577-594 / 第 577-594 行
~~~cpp
 577:     const RealT s1s1 = s1*s1;
 578: 
 579:     const RealT num = (6*s3 - (3*(2*s2 - s1s1) + s1s1)*s1);
 580:     const RealT den = (2*s2 - s1s1);
 581: 
 582:     return num / pow(den, static_cast<RealT>(1.5));
 583: }
 584: 
 585: template <typename RealT, typename PolicyT>
 586: RealT kurtosis(hyperexponential_distribution<RealT,PolicyT> const& dist)
 587: {
 588:     const std::size_t n = dist.num_phases();
 589:     const std::vector<RealT> probs = dist.probabilities();
 590:     const std::vector<RealT> rates = dist.rates();
 591: 
 592:     RealT s1 = 0; // \sum_{i=1}^n \frac{p_i}{\lambda_i}
 593:     RealT s2 = 0; // \sum_{i=1}^n \frac{p_i}{\lambda_i^2}
 594:     RealT s3 = 0; // \sum_{i=1}^n \frac{p_i}{\lambda_i^3}
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as num_phases, probabilities, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 num_phases, probabilities, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 595-612 / 第 595-612 行
~~~cpp
 595:     RealT s4 = 0; // \sum_{i=1}^n \frac{p_i}{\lambda_i^4}
 596:     for (std::size_t i = 0; i < n; ++i)
 597:     {
 598:         const RealT p = probs[i];
 599:         const RealT r = rates[i];
 600:         const RealT r2 = r*r;
 601:         const RealT r3 = r2*r;
 602:         const RealT r4 = r3*r;
 603: 
 604:         s1 += p/r;
 605:         s2 += p/r2;
 606:         s3 += p/r3;
 607:         s4 += p/r4;
 608:     }
 609: 
 610:     const RealT s1s1 = s1*s1;
 611: 
 612:     const RealT num = (24*s4 - 24*s3*s1 + 3*(2*(2*s2 - s1s1) + s1s1)*s1s1);
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 613-630 / 第 613-630 行
~~~cpp
 613:     const RealT den = (2*s2 - s1s1);
 614: 
 615:     return num/(den*den);
 616: }
 617: 
 618: template <typename RealT, typename PolicyT>
 619: RealT kurtosis_excess(hyperexponential_distribution<RealT,PolicyT> const& dist)
 620: {
 621:     return kurtosis(dist) - 3;
 622: }
 623: 
 624: template <typename RealT, typename PolicyT>
 625: RealT mode(hyperexponential_distribution<RealT,PolicyT> const& /*dist*/)
 626: {
 627:     return 0;
 628: }
 629: 
 630: }} // namespace boost::math
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. Return statements hand the computed result or status back to the caller.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 return 语句会把计算结果或状态返回给调用方。

### Lines 631-641 / 第 631-641 行
~~~cpp
 631: 
 632: #ifdef _MSC_VER
 633: #pragma warning (pop)
 634: #endif
 635: // This include must be at the end, *after* the accessors
 636: // for this distribution have been defined, in order to
 637: // keep compilers that support two-phase lookup happy.
 638: #include <boost/math/distributions/detail/derived_accessors.hpp>
 639: #include <boost/math/distributions/detail/generic_quantile.hpp>
 640: 
 641: #endif // BOOST_MATH_DISTRIBUTIONS_HYPEREXPONENTIAL
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp, boost/math/distributions/detail/generic_quantile.hpp so the surrounding code can use external declarations. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp, boost/math/distributions/detail/generic_quantile.hpp 等依赖，使周围代码可以使用外部声明。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

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
- **Included headers / 包含头文件**: `boost/math/tools/cxx03_warn.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/exponential.hpp, boost/math/policies/policy.hpp, boost/math/special_functions/fpclassify.hpp, boost/math/tools/precision.hpp, boost/math/tools/roots.hpp, boost/math/tools/is_detected.hpp, cstddef, iterator, limits, ...`
- **Namespaces / 命名空间**: `boost, math, detail, hyperexp_detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `generic_quantile, std::accumulate, end, size, check_rates, num_phases, probabilities, rates, ...`
