# poisson.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/poisson.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the poisson distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 poisson 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // boost\math\distributions\poisson.hpp
   2: 
   3: // Copyright John Maddock 2006.
   4: // Copyright Paul A. Bristow 2007.
   5: // Copyright Matt Borland 2024.
   6: 
   7: // Use, modification and distribution are subject to the
   8: // Boost Software License, Version 1.0.
   9: // (See accompanying file LICENSE_1_0.txt
  10: // or copy at http://www.boost.org/LICENSE_1_0.txt)
  11: 
  12: // Poisson distribution is a discrete probability distribution.
  13: // It expresses the probability of a number (k) of
  14: // events, occurrences, failures or arrivals occurring in a fixed time,
  15: // assuming these events occur with a known average or mean rate (lambda)
  16: // and are independent of the time since the last event.
  17: // The distribution was discovered by Simeon-Denis Poisson (1781-1840).
  18: 
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: // Parameter lambda is the mean number of events in the given time interval.
  20: // The random variate k is the number of events, occurrences or arrivals.
  21: // k argument may be integral, signed, or unsigned, or floating point.
  22: // If necessary, it has already been promoted from an integral type.
  23: 
  24: // Note that the Poisson distribution
  25: // (like others including the binomial, negative binomial & Bernoulli)
  26: // is strictly defined as a discrete function:
  27: // only integral values of k are envisaged.
  28: // However because the method of calculation uses a continuous gamma function,
  29: // it is convenient to treat it as if a continuous function,
  30: // and permit non-integral values of k.
  31: // To enforce the strict mathematical model, users should use floor or ceil functions
  32: // on k outside this function to ensure that k is integral.
  33: 
  34: // See http://en.wikipedia.org/wiki/Poisson_distribution
  35: // http://documents.wolfram.com/v5/Add-onsLinks/StandardPackages/Statistics/DiscreteDistributions.html
  36: 
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: #ifndef BOOST_MATH_SPECIAL_POISSON_HPP
  38: #define BOOST_MATH_SPECIAL_POISSON_HPP
  39: 
  40: #include <boost/math/tools/config.hpp>
  41: #include <boost/math/tools/tuple.hpp>
  42: #include <boost/math/tools/cstdint.hpp>
  43: #include <boost/math/tools/numeric_limits.hpp>
  44: #include <boost/math/distributions/fwd.hpp>
  45: #include <boost/math/special_functions/gamma.hpp> // for incomplete gamma. gamma_q
  46: #include <boost/math/special_functions/trunc.hpp> // for incomplete gamma. gamma_q
  47: #include <boost/math/distributions/complement.hpp> // complements
  48: #include <boost/math/distributions/detail/common_error_handling.hpp> // error checks
  49: #include <boost/math/special_functions/fpclassify.hpp> // isnan.
  50: #include <boost/math/special_functions/factorials.hpp> // factorials.
  51: #include <boost/math/tools/roots.hpp> // for root finding.
  52: #include <boost/math/distributions/detail/inv_discrete_quantile.hpp>
  53: 
  54: namespace boost
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost），以保持符号组织清晰。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: {
  56:   namespace math
  57:   {
  58:     namespace poisson_detail
  59:     {
  60:       // Common error checking routines for Poisson distribution functions.
  61:       // These are convoluted, & apparently redundant, to try to ensure that
  62:       // checks are always performed, even if exceptions are not enabled.
  63: 
  64:       template <class RealType, class Policy>
  65:       BOOST_MATH_GPU_ENABLED inline bool check_mean(const char* function, const RealType& mean, RealType* result, const Policy& pol)
  66:       {
  67:         if(!(boost::math::isfinite)(mean) || (mean < 0))
  68:         {
  69:           *result = policies::raise_domain_error<RealType>(
  70:             function,
  71:             "Mean argument is %1%, but must be >= 0 !", mean, pol);
  72:           return false;
~~~
- **EN:** The code enters namespace scope (math::poisson_detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（math::poisson_detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:         }
  74:         return true;
  75:       } // bool check_mean
  76: 
  77:       template <class RealType, class Policy>
  78:       BOOST_MATH_GPU_ENABLED inline bool check_mean_NZ(const char* function, const RealType& mean, RealType* result, const Policy& pol)
  79:       { // mean == 0 is considered an error.
  80:         if( !(boost::math::isfinite)(mean) || (mean <= 0))
  81:         {
  82:           *result = policies::raise_domain_error<RealType>(
  83:             function,
  84:             "Mean argument is %1%, but must be > 0 !", mean, pol);
  85:           return false;
  86:         }
  87:         return true;
  88:       } // bool check_mean_NZ
  89: 
  90:       template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:       BOOST_MATH_GPU_ENABLED inline bool check_dist(const char* function, const RealType& mean, RealType* result, const Policy& pol)
  92:       { // Only one check, so this is redundant really but should be optimized away.
  93:         return check_mean_NZ(function, mean, result, pol);
  94:       } // bool check_dist
  95: 
  96:       template <class RealType, class Policy>
  97:       BOOST_MATH_GPU_ENABLED inline bool check_k(const char* function, const RealType& k, RealType* result, const Policy& pol)
  98:       {
  99:         if((k < 0) || !(boost::math::isfinite)(k))
 100:         {
 101:           *result = policies::raise_domain_error<RealType>(
 102:             function,
 103:             "Number of events k argument is %1%, but must be >= 0 !", k, pol);
 104:           return false;
 105:         }
 106:         return true;
 107:       } // bool check_k
 108: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:       template <class RealType, class Policy>
 110:       BOOST_MATH_GPU_ENABLED inline bool check_dist_and_k(const char* function, RealType mean, RealType k, RealType* result, const Policy& pol)
 111:       {
 112:         if((check_dist(function, mean, result, pol) == false) ||
 113:           (check_k(function, k, result, pol) == false))
 114:         {
 115:           return false;
 116:         }
 117:         return true;
 118:       } // bool check_dist_and_k
 119: 
 120:       template <class RealType, class Policy>
 121:       BOOST_MATH_GPU_ENABLED inline bool check_prob(const char* function, const RealType& p, RealType* result, const Policy& pol)
 122:       { // Check 0 <= p <= 1
 123:         if(!(boost::math::isfinite)(p) || (p < 0) || (p > 1))
 124:         {
 125:           *result = policies::raise_domain_error<RealType>(
 126:             function,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:             "Probability argument is %1%, but must be >= 0 and <= 1 !", p, pol);
 128:           return false;
 129:         }
 130:         return true;
 131:       } // bool check_prob
 132: 
 133:       template <class RealType, class Policy>
 134:       BOOST_MATH_GPU_ENABLED inline bool check_dist_and_prob(const char* function, RealType mean,  RealType p, RealType* result, const Policy& pol)
 135:       {
 136:         if((check_dist(function, mean, result, pol) == false) ||
 137:           (check_prob(function, p, result, pol) == false))
 138:         {
 139:           return false;
 140:         }
 141:         return true;
 142:       } // bool check_dist_and_prob
 143: 
 144:     } // namespace poisson_detail
~~~
- **EN:** The code enters namespace scope (poisson_detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（poisson_detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145: 
 146:     template <class RealType = double, class Policy = policies::policy<> >
 147:     class poisson_distribution
 148:     {
 149:     public:
 150:       using value_type = RealType;
 151:       using policy_type = Policy;
 152: 
 153:       BOOST_MATH_GPU_ENABLED explicit poisson_distribution(RealType l_mean = 1) : m_l(l_mean) // mean (lambda).
 154:       { // Expected mean number of events that occur during the given interval.
 155:         RealType r;
 156:         poisson_detail::check_dist(
 157:            "boost::math::poisson_distribution<%1%>::poisson_distribution",
 158:           m_l,
 159:           &r, Policy());
 160:       } // poisson_distribution constructor.
 161: 
 162:       BOOST_MATH_GPU_ENABLED RealType mean() const
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:       { // Private data getter function.
 164:         return m_l;
 165:       }
 166:     private:
 167:       // Data member, initialized by constructor.
 168:       RealType m_l; // mean number of occurrences.
 169:     }; // template <class RealType, class Policy> class poisson_distribution
 170: 
 171:     using poisson = poisson_distribution<double>; // Reserved name of type double.
 172: 
 173:     #ifdef __cpp_deduction_guides
 174:     template <class RealType>
 175:     poisson_distribution(RealType)->poisson_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 176:     #endif
 177: 
 178:     // Non-member functions to give properties of the distribution.
 179: 
 180:     template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:     BOOST_MATH_GPU_ENABLED inline boost::math::pair<RealType, RealType> range(const poisson_distribution<RealType, Policy>& /* dist */)
 182:     { // Range of permissible values for random variable k.
 183:        using boost::math::tools::max_value;
 184:        return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>()); // Max integer?
 185:     }
 186: 
 187:     template <class RealType, class Policy>
 188:     BOOST_MATH_GPU_ENABLED inline boost::math::pair<RealType, RealType> support(const poisson_distribution<RealType, Policy>& /* dist */)
 189:     { // Range of supported values for random variable k.
 190:        // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 191:        using boost::math::tools::max_value;
 192:        return boost::math::pair<RealType, RealType>(static_cast<RealType>(0),  max_value<RealType>());
 193:     }
 194: 
 195:     template <class RealType, class Policy>
 196:     BOOST_MATH_GPU_ENABLED inline RealType mean(const poisson_distribution<RealType, Policy>& dist)
 197:     { // Mean of poisson distribution = lambda.
 198:       return dist.mean();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:     } // mean
 200: 
 201:     template <class RealType, class Policy>
 202:     BOOST_MATH_GPU_ENABLED inline RealType mode(const poisson_distribution<RealType, Policy>& dist)
 203:     { // mode.
 204:       BOOST_MATH_STD_USING // ADL of std functions.
 205:       return floor(dist.mean());
 206:     }
 207: 
 208:     // Median now implemented via quantile(half) in derived accessors.
 209: 
 210:     template <class RealType, class Policy>
 211:     BOOST_MATH_GPU_ENABLED inline RealType variance(const poisson_distribution<RealType, Policy>& dist)
 212:     { // variance.
 213:       return dist.mean();
 214:     }
 215: 
 216:     // standard_deviation provided by derived accessors.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217: 
 218:     template <class RealType, class Policy>
 219:     BOOST_MATH_GPU_ENABLED inline RealType skewness(const poisson_distribution<RealType, Policy>& dist)
 220:     { // skewness = sqrt(l).
 221:       BOOST_MATH_STD_USING // ADL of std functions.
 222:       return 1 / sqrt(dist.mean());
 223:     }
 224: 
 225:     template <class RealType, class Policy>
 226:     BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const poisson_distribution<RealType, Policy>& dist)
 227:     { // skewness = sqrt(l).
 228:       return 1 / dist.mean(); // kurtosis_excess 1/mean from Wiki & MathWorld eq 31.
 229:       // http://mathworld.wolfram.com/Kurtosis.html explains that the kurtosis excess
 230:       // is more convenient because the kurtosis excess of a normal distribution is zero
 231:       // whereas the true kurtosis is 3.
 232:     } // RealType kurtosis_excess
 233: 
 234:     template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:     BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const poisson_distribution<RealType, Policy>& dist)
 236:     { // kurtosis is 4th moment about the mean = u4 / sd ^ 4
 237:       // http://en.wikipedia.org/wiki/Kurtosis
 238:       // kurtosis can range from -2 (flat top) to +infinity (sharp peak & heavy tails).
 239:       // http://www.itl.nist.gov/div898/handbook/eda/section3/eda35b.htm
 240:       return 3 + 1 / dist.mean(); // NIST.
 241:       // http://mathworld.wolfram.com/Kurtosis.html explains that the kurtosis excess
 242:       // is more convenient because the kurtosis excess of a normal distribution is zero
 243:       // whereas the true kurtosis is 3.
 244:     } // RealType kurtosis
 245: 
 246:     template <class RealType, class Policy>
 247:     BOOST_MATH_GPU_ENABLED RealType pdf(const poisson_distribution<RealType, Policy>& dist, const RealType& k)
 248:     { // Probability Density/Mass Function.
 249:       // Probability that there are EXACTLY k occurrences (or arrivals).
 250:       BOOST_FPU_EXCEPTION_GUARD
 251: 
 252:       BOOST_MATH_STD_USING // for ADL of std functions.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253: 
 254:       RealType mean = dist.mean();
 255:       // Error check:
 256:       RealType result = 0;
 257:       if(false == poisson_detail::check_dist_and_k(
 258:         "boost::math::pdf(const poisson_distribution<%1%>&, %1%)",
 259:         mean,
 260:         k,
 261:         &result, Policy()))
 262:       {
 263:         return result;
 264:       }
 265: 
 266:       // Special case of mean zero, regardless of the number of events k.
 267:       if (mean == 0)
 268:       { // Probability for any k is zero.
 269:         return 0;
 270:       }
~~~
- **EN:** This range declares or defines callable logic such as mean. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 mean。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:       if (k == 0)
 272:       { // mean ^ k = 1, and k! = 1, so can simplify.
 273:         return exp(-mean);
 274:       }
 275:       return boost::math::gamma_p_derivative(k+1, mean, Policy());
 276:     } // pdf
 277: 
 278:     template <class RealType, class Policy>
 279:     BOOST_MATH_GPU_ENABLED RealType logpdf(const poisson_distribution<RealType, Policy>& dist, const RealType& k)
 280:     {
 281:       BOOST_FPU_EXCEPTION_GUARD
 282: 
 283:       BOOST_MATH_STD_USING // for ADL of std functions.
 284:       using boost::math::lgamma;
 285: 
 286:       RealType mean = dist.mean();
 287:       // Error check:
 288:       RealType result = -boost::math::numeric_limits<RealType>::infinity();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as mean, infinity.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 mean, infinity。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:       if(false == poisson_detail::check_dist_and_k(
 290:         "boost::math::pdf(const poisson_distribution<%1%>&, %1%)",
 291:         mean,
 292:         k,
 293:         &result, Policy()))
 294:       {
 295:         return result;
 296:       }
 297: 
 298:       // Special case of mean zero, regardless of the number of events k.
 299:       if (mean == 0)
 300:       { // Probability for any k is zero.
 301:         return boost::math::numeric_limits<RealType>::quiet_NaN();
 302:       }
 303: 
 304:       // Special case where k and lambda are both positive
 305:       if(k > 0 && mean > 0)
 306:       {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:         return -lgamma(k+1) + k*log(mean) - mean;
 308:       }
 309: 
 310:       result = log(pdf(dist, k));
 311:       return result;
 312:     }
 313: 
 314:     template <class RealType, class Policy>
 315:     BOOST_MATH_GPU_ENABLED RealType cdf(const poisson_distribution<RealType, Policy>& dist, const RealType& k)
 316:     { // Cumulative Distribution Function Poisson.
 317:       // The random variate k is the number of occurrences(or arrivals)
 318:       // k argument may be integral, signed, or unsigned, or floating point.
 319:       // If necessary, it has already been promoted from an integral type.
 320:       // Returns the sum of the terms 0 through k of the Poisson Probability Density or Mass (pdf).
 321: 
 322:       // But note that the Poisson distribution
 323:       // (like others including the binomial, negative binomial & Bernoulli)
 324:       // is strictly defined as a discrete function: only integral values of k are envisaged.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as log.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:       // However because of the method of calculation using a continuous gamma function,
 326:       // it is convenient to treat it as if it is a continuous function
 327:       // and permit non-integral values of k.
 328:       // To enforce the strict mathematical model, users should use floor or ceil functions
 329:       // outside this function to ensure that k is integral.
 330: 
 331:       // The terms are not summed directly (at least for larger k)
 332:       // instead the incomplete gamma integral is employed,
 333: 
 334:       BOOST_MATH_STD_USING // for ADL of std function exp.
 335: 
 336:       RealType mean = dist.mean();
 337:       // Error checks:
 338:       RealType result = 0;
 339:       if(false == poisson_detail::check_dist_and_k(
 340:         "boost::math::cdf(const poisson_distribution<%1%>&, %1%)",
 341:         mean,
 342:         k,
~~~
- **EN:** This range declares or defines callable logic such as mean. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 mean。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:         &result, Policy()))
 344:       {
 345:         return result;
 346:       }
 347:       // Special cases:
 348:       if (mean == 0)
 349:       { // Probability for any k is zero.
 350:         return 0;
 351:       }
 352:       if (k == 0)
 353:       {
 354:         // mean (and k) have already been checked,
 355:         // so this avoids unnecessary repeated checks.
 356:        return exp(-mean);
 357:       }
 358:       // For small integral k could use a finite sum -
 359:       // it's cheaper than the gamma function.
 360:       // BUT this is now done efficiently by gamma_q function.
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:       // Calculate poisson cdf using the gamma_q function.
 362:       return gamma_q(k+1, mean, Policy());
 363:     } // binomial cdf
 364: 
 365:     template <class RealType, class Policy>
 366:     BOOST_MATH_GPU_ENABLED RealType cdf(const complemented2_type<poisson_distribution<RealType, Policy>, RealType>& c)
 367:     { // Complemented Cumulative Distribution Function Poisson
 368:       // The random variate k is the number of events, occurrences or arrivals.
 369:       // k argument may be integral, signed, or unsigned, or floating point.
 370:       // If necessary, it has already been promoted from an integral type.
 371:       // But note that the Poisson distribution
 372:       // (like others including the binomial, negative binomial & Bernoulli)
 373:       // is strictly defined as a discrete function: only integral values of k are envisaged.
 374:       // However because of the method of calculation using a continuous gamma function,
 375:       // it is convenient to treat it as is it is a continuous function
 376:       // and permit non-integral values of k.
 377:       // To enforce the strict mathematical model, users should use floor or ceil functions
 378:       // outside this function to ensure that k is integral.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379: 
 380:       // Returns the sum of the terms k+1 through inf of the Poisson Probability Density/Mass (pdf).
 381:       // The terms are not summed directly (at least for larger k)
 382:       // instead the incomplete gamma integral is employed,
 383: 
 384:       RealType const& k = c.param;
 385:       poisson_distribution<RealType, Policy> const& dist = c.dist;
 386: 
 387:       RealType mean = dist.mean();
 388: 
 389:       // Error checks:
 390:       RealType result = 0;
 391:       if(false == poisson_detail::check_dist_and_k(
 392:         "boost::math::cdf(const poisson_distribution<%1%>&, %1%)",
 393:         mean,
 394:         k,
 395:         &result, Policy()))
 396:       {
~~~
- **EN:** This range declares or defines callable logic such as mean. Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 mean。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:         return result;
 398:       }
 399:       // Special case of mean, regardless of the number of events k.
 400:       if (mean == 0)
 401:       { // Probability for any k is unity, complement of zero.
 402:         return 1;
 403:       }
 404:       if (k == 0)
 405:       { // Avoid repeated checks on k and mean in gamma_p.
 406:          return -boost::math::expm1(-mean, Policy());
 407:       }
 408:       // Unlike un-complemented cdf (sum from 0 to k),
 409:       // can't use finite sum from k+1 to infinity for small integral k,
 410:       // anyway it is now done efficiently by gamma_p.
 411:       return gamma_p(k + 1, mean, Policy()); // Calculate Poisson cdf using the gamma_p function.
 412:       // CCDF = gamma_p(k+1, lambda)
 413:     } // poisson ccdf
 414: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:     template <class RealType, class Policy>
 416:     BOOST_MATH_GPU_ENABLED inline RealType quantile(const poisson_distribution<RealType, Policy>& dist, const RealType& p)
 417:     { // Quantile (or Percent Point) Poisson function.
 418:       // Return the number of expected events k for a given probability p.
 419:       constexpr auto function = "boost::math::quantile(const poisson_distribution<%1%>&, %1%)";
 420:       RealType result = 0; // of Argument checks:
 421:       if(false == poisson_detail::check_prob(
 422:         function,
 423:         p,
 424:         &result, Policy()))
 425:       {
 426:         return result;
 427:       }
 428:       // Special case:
 429:       if (dist.mean() == 0)
 430:       { // if mean = 0 then p = 0, so k can be anything?
 431:          if (false == poisson_detail::check_mean_NZ(
 432:          function,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:          dist.mean(),
 434:          &result, Policy()))
 435:         {
 436:           return result;
 437:         }
 438:       }
 439:       if(p == 0)
 440:       {
 441:          return 0; // Exact result regardless of discrete-quantile Policy
 442:       }
 443:       if(p == 1)
 444:       {
 445:          return policies::raise_overflow_error<RealType>(function, 0, Policy());
 446:       }
 447:       using discrete_type = typename Policy::discrete_quantile_type;
 448:       boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 449:       RealType guess;
 450:       RealType factor = 8;
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:       RealType z = dist.mean();
 452:       if(z < 1)
 453:          guess = z;
 454:       else
 455:          guess = boost::math::detail::inverse_poisson_cornish_fisher(z, p, RealType(1-p), Policy());
 456:       if(z > 5)
 457:       {
 458:          if(z > 1000)
 459:             factor = 1.01f;
 460:          else if(z > 50)
 461:             factor = 1.1f;
 462:          else if(guess > 10)
 463:             factor = 1.25f;
 464:          else
 465:             factor = 2;
 466:          if(guess < 1.1)
 467:             factor = 8;
 468:       }
~~~
- **EN:** This range declares or defines callable logic such as mean, boost::math::detail::inverse_poisson_cornish_fisher. Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 mean, boost::math::detail::inverse_poisson_cornish_fisher。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469: 
 470:       return detail::inverse_discrete_quantile(
 471:          dist,
 472:          p,
 473:          false,
 474:          guess,
 475:          factor,
 476:          RealType(1),
 477:          discrete_type(),
 478:          max_iter);
 479:    } // quantile
 480: 
 481:     template <class RealType, class Policy>
 482:     BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<poisson_distribution<RealType, Policy>, RealType>& c)
 483:     { // Quantile (or Percent Point) of Poisson function.
 484:       // Return the number of expected events k for a given
 485:       // complement of the probability q.
 486:       //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:       // Error checks:
 488:       constexpr auto function = "boost::math::quantile(complement(const poisson_distribution<%1%>&, %1%))";
 489:       RealType q = c.param;
 490:       const poisson_distribution<RealType, Policy>& dist = c.dist;
 491:       RealType result = 0;  // of argument checks.
 492:       if(false == poisson_detail::check_prob(
 493:         function,
 494:         q,
 495:         &result, Policy()))
 496:       {
 497:         return result;
 498:       }
 499:       // Special case:
 500:       if (dist.mean() == 0)
 501:       { // if mean = 0 then p = 0, so k can be anything?
 502:          if (false == poisson_detail::check_mean_NZ(
 503:          function,
 504:          dist.mean(),
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:          &result, Policy()))
 506:         {
 507:           return result;
 508:         }
 509:       }
 510:       if(q == 0)
 511:       {
 512:          return policies::raise_overflow_error<RealType>(function, 0, Policy());
 513:       }
 514:       if(q == 1)
 515:       {
 516:          return 0;  // Exact result regardless of discrete-quantile Policy
 517:       }
 518:       using discrete_type = typename Policy::discrete_quantile_type;
 519:       boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 520:       RealType guess;
 521:       RealType factor = 8;
 522:       RealType z = dist.mean();
~~~
- **EN:** This range declares or defines callable logic such as mean. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 mean。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:       if(z < 1)
 524:          guess = z;
 525:       else
 526:          guess = boost::math::detail::inverse_poisson_cornish_fisher(z, RealType(1-q), q, Policy());
 527:       if(z > 5)
 528:       {
 529:          if(z > 1000)
 530:             factor = 1.01f;
 531:          else if(z > 50)
 532:             factor = 1.1f;
 533:          else if(guess > 10)
 534:             factor = 1.25f;
 535:          else
 536:             factor = 2;
 537:          if(guess < 1.1)
 538:             factor = 8;
 539:       }
 540: 
~~~
- **EN:** This range declares or defines callable logic such as boost::math::detail::inverse_poisson_cornish_fisher. Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::detail::inverse_poisson_cornish_fisher。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:       return detail::inverse_discrete_quantile(
 542:          dist,
 543:          q,
 544:          true,
 545:          guess,
 546:          factor,
 547:          RealType(1),
 548:          discrete_type(),
 549:          max_iter);
 550:    } // quantile complement.
 551: 
 552:   } // namespace math
 553: } // namespace boost
 554: 
 555: // This include must be at the end, *after* the accessors
 556: // for this distribution have been defined, in order to
 557: // keep compilers that support two-phase lookup happy.
 558: #include <boost/math/distributions/detail/derived_accessors.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp so the surrounding code can use external declarations. The code enters namespace scope (math::boost) to keep symbols organized. Return statements hand the computed result or status back to the caller.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 return 语句会把计算结果或状态返回给调用方。

### Lines 559-563 / 第 559-563 行
~~~cpp
 559: 
 560: #endif // BOOST_MATH_SPECIAL_POISSON_HPP
 561: 
 562: 
 563: 
~~~
- **EN:** This short range closes a conditional-compilation or header-guard region. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这一小段用于结束条件编译区域或头文件保护区域。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/cstdint.hpp, boost/math/tools/numeric_limits.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/trunc.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/special_functions/fpclassify.hpp, boost/math/special_functions/factorials.hpp, boost/math/tools/roots.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, poisson_detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `Policy, mean, infinity, log, boost::math::detail::inverse_poisson_cornish_fisher`
