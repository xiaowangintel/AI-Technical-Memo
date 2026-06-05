# negative_binomial.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/negative_binomial.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the negative binomial distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 negative binomial 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // boost\math\special_functions\negative_binomial.hpp
   2: 
   3: // Copyright Paul A. Bristow 2007.
   4: // Copyright John Maddock 2007.
   5: 
   6: // Use, modification and distribution are subject to the
   7: // Boost Software License, Version 1.0.
   8: // (See accompanying file LICENSE_1_0.txt
   9: // or copy at http://www.boost.org/LICENSE_1_0.txt)
  10: 
  11: // http://en.wikipedia.org/wiki/negative_binomial_distribution
  12: // http://mathworld.wolfram.com/NegativeBinomialDistribution.html
  13: // http://documents.wolfram.com/teachersedition/Teacher/Statistics/DiscreteDistributions.html
  14: 
  15: // The negative binomial distribution NegativeBinomialDistribution[n, p]
  16: // is the distribution of the number (k) of failures that occur in a sequence of trials before
  17: // r successes have occurred, where the probability of success in each trial is p.
  18: 
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: // In a sequence of Bernoulli trials or events
  20: // (independent, yes or no, succeed or fail) with success_fraction probability p,
  21: // negative_binomial is the probability that k or fewer failures
  22: // precede the r th trial's success.
  23: // random variable k is the number of failures (NOT the probability).
  24: 
  25: // Negative_binomial distribution is a discrete probability distribution.
  26: // But note that the negative binomial distribution
  27: // (like others including the binomial, Poisson & Bernoulli)
  28: // is strictly defined as a discrete function: only integral values of k are envisaged.
  29: // However because of the method of calculation using a continuous gamma function,
  30: // it is convenient to treat it as if a continuous function,
  31: // and permit non-integral values of k.
  32: 
  33: // However, by default the policy is to use discrete_quantile_policy.
  34: 
  35: // To enforce the strict mathematical model, users should use conversion
  36: // on k outside this function to ensure that k is integral.
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: 
  38: // MATHCAD cumulative negative binomial pnbinom(k, n, p)
  39: 
  40: // Implementation note: much greater speed, and perhaps greater accuracy,
  41: // might be achieved for extreme values by using a normal approximation.
  42: // This is NOT been tested or implemented.
  43: 
  44: #ifndef BOOST_MATH_SPECIAL_NEGATIVE_BINOMIAL_HPP
  45: #define BOOST_MATH_SPECIAL_NEGATIVE_BINOMIAL_HPP
  46: 
  47: #include <boost/math/tools/config.hpp>
  48: #include <boost/math/tools/tuple.hpp>
  49: #include <boost/math/tools/numeric_limits.hpp>
  50: #include <boost/math/tools/cstdint.hpp>
  51: #include <boost/math/distributions/fwd.hpp>
  52: #include <boost/math/special_functions/beta.hpp> // for ibeta(a, b, x) == Ix(a, b).
  53: #include <boost/math/distributions/complement.hpp> // complement.
  54: #include <boost/math/distributions/detail/common_error_handling.hpp> // error checks domain_error & logic_error.
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: #include <boost/math/special_functions/fpclassify.hpp> // isnan.
  56: #include <boost/math/tools/roots.hpp> // for root finding.
  57: #include <boost/math/distributions/detail/inv_discrete_quantile.hpp>
  58: #include <boost/math/policies/error_handling.hpp>
  59: 
  60: #if defined (BOOST_MSVC)
  61: #  pragma warning(push)
  62: // This believed not now necessary, so commented out.
  63: //#  pragma warning(disable: 4702) // unreachable code.
  64: // in domain_error_imp in error_handling.
  65: #endif
  66: 
  67: namespace boost
  68: {
  69:   namespace math
  70:   {
  71:     namespace negative_binomial_detail
  72:     {
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/fpclassify.hpp, boost/math/tools/roots.hpp, boost/math/distributions/detail/inv_discrete_quantile.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math::negative_binomial_detail) to keep symbols organized. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/special_functions/fpclassify.hpp, boost/math/tools/roots.hpp, boost/math/distributions/detail/inv_discrete_quantile.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::negative_binomial_detail），以保持符号组织清晰。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       // Common error checking routines for negative binomial distribution functions:
  74:       template <class RealType, class Policy>
  75:       BOOST_MATH_GPU_ENABLED inline bool check_successes(const char* function, const RealType& r, RealType* result, const Policy& pol)
  76:       {
  77:         if( !(boost::math::isfinite)(r) || (r <= 0) )
  78:         {
  79:           *result = policies::raise_domain_error<RealType>(
  80:             function,
  81:             "Number of successes argument is %1%, but must be > 0 !", r, pol);
  82:           return false;
  83:         }
  84:         return true;
  85:       }
  86:       template <class RealType, class Policy>
  87:       BOOST_MATH_GPU_ENABLED inline bool check_success_fraction(const char* function, const RealType& p, RealType* result, const Policy& pol)
  88:       {
  89:         if( !(boost::math::isfinite)(p) || (p < 0) || (p > 1) )
  90:         {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:           *result = policies::raise_domain_error<RealType>(
  92:             function,
  93:             "Success fraction argument is %1%, but must be >= 0 and <= 1 !", p, pol);
  94:           return false;
  95:         }
  96:         return true;
  97:       }
  98:       template <class RealType, class Policy>
  99:       BOOST_MATH_GPU_ENABLED inline bool check_dist(const char* function, const RealType& r, const RealType& p, RealType* result, const Policy& pol)
 100:       {
 101:         return check_success_fraction(function, p, result, pol)
 102:           && check_successes(function, r, result, pol);
 103:       }
 104:       template <class RealType, class Policy>
 105:       BOOST_MATH_GPU_ENABLED inline bool check_dist_and_k(const char* function, const RealType& r, const RealType& p, RealType k, RealType* result, const Policy& pol)
 106:       {
 107:         if(check_dist(function, r, p, result, pol) == false)
 108:         {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as check_successes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 check_successes。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:           return false;
 110:         }
 111:         if( !(boost::math::isfinite)(k) || (k < 0) )
 112:         { // Check k failures.
 113:           *result = policies::raise_domain_error<RealType>(
 114:             function,
 115:             "Number of failures argument is %1%, but must be >= 0 !", k, pol);
 116:           return false;
 117:         }
 118:         return true;
 119:       } // Check_dist_and_k
 120: 
 121:       template <class RealType, class Policy>
 122:       BOOST_MATH_GPU_ENABLED inline bool check_dist_and_prob(const char* function, const RealType& r, RealType p, RealType prob, RealType* result, const Policy& pol)
 123:       {
 124:         if((check_dist(function, r, p, result, pol) && detail::check_probability(function, prob, result, pol)) == false)
 125:         {
 126:           return false;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:         }
 128:         return true;
 129:       } // check_dist_and_prob
 130:     } //  namespace negative_binomial_detail
 131: 
 132:     template <class RealType = double, class Policy = policies::policy<> >
 133:     class negative_binomial_distribution
 134:     {
 135:     public:
 136:       typedef RealType value_type;
 137:       typedef Policy policy_type;
 138: 
 139:       BOOST_MATH_GPU_ENABLED negative_binomial_distribution(RealType r, RealType p) : m_r(r), m_p(p)
 140:       { // Constructor.
 141:         RealType result;
 142:         negative_binomial_detail::check_dist(
 143:           "negative_binomial_distribution<%1%>::negative_binomial_distribution",
 144:           m_r, // Check successes r > 0.
~~~
- **EN:** The code enters namespace scope (negative_binomial_detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（negative_binomial_detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:           m_p, // Check success_fraction 0 <= p <= 1.
 146:           &result, Policy());
 147:       } // negative_binomial_distribution constructor.
 148: 
 149:       // Private data getter class member functions.
 150:       BOOST_MATH_GPU_ENABLED RealType success_fraction() const
 151:       { // Probability of success as fraction in range 0 to 1.
 152:         return m_p;
 153:       }
 154:       BOOST_MATH_GPU_ENABLED RealType successes() const
 155:       { // Total number of successes r.
 156:         return m_r;
 157:       }
 158: 
 159:       BOOST_MATH_GPU_ENABLED static RealType find_lower_bound_on_p(
 160:         RealType trials,
 161:         RealType successes,
 162:         RealType alpha) // alpha 0.05 equivalent to 95% for one-sided test.
~~~
- **EN:** It introduces the class `member` as part of the file's main abstraction. This range declares or defines callable logic such as Policy. Return statements hand the computed result or status back to the caller.
- **CN:** 它引入了 class `member`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy。 return 语句会把计算结果或状态返回给调用方。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:       {
 164:         constexpr auto function = "boost::math::negative_binomial<%1%>::find_lower_bound_on_p";
 165:         RealType result = 0;  // of error checks.
 166:         RealType failures = trials - successes;
 167:         if(false == detail::check_probability(function, alpha, &result, Policy())
 168:           && negative_binomial_detail::check_dist_and_k(
 169:           function, successes, RealType(0), failures, &result, Policy()))
 170:         {
 171:           return result;
 172:         }
 173:         // Use complement ibeta_inv function for lower bound.
 174:         // This is adapted from the corresponding binomial formula
 175:         // here: http://www.itl.nist.gov/div898/handbook/prc/section2/prc241.htm
 176:         // This is a Clopper-Pearson interval, and may be overly conservative,
 177:         // see also "A Simple Improved Inferential Method for Some
 178:         // Discrete Distributions" Yong CAI and K. KRISHNAMOORTHY
 179:         // http://www.ucs.louisiana.edu/~kxk4695/Discrete_new.pdf
 180:         //
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:         return ibeta_inv(successes, failures + 1, alpha, static_cast<RealType*>(nullptr), Policy());
 182:       } // find_lower_bound_on_p
 183: 
 184:       BOOST_MATH_GPU_ENABLED static RealType find_upper_bound_on_p(
 185:         RealType trials,
 186:         RealType successes,
 187:         RealType alpha) // alpha 0.05 equivalent to 95% for one-sided test.
 188:       {
 189:         constexpr auto function = "boost::math::negative_binomial<%1%>::find_upper_bound_on_p";
 190:         RealType result = 0;  // of error checks.
 191:         RealType failures = trials - successes;
 192:         if(false == negative_binomial_detail::check_dist_and_k(
 193:           function, successes, RealType(0), failures, &result, Policy())
 194:           && detail::check_probability(function, alpha, &result, Policy()))
 195:         {
 196:           return result;
 197:         }
 198:         if(failures == 0)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:            return 1;
 200:         // Use complement ibetac_inv function for upper bound.
 201:         // Note adjusted failures value: *not* failures+1 as usual.
 202:         // This is adapted from the corresponding binomial formula
 203:         // here: http://www.itl.nist.gov/div898/handbook/prc/section2/prc241.htm
 204:         // This is a Clopper-Pearson interval, and may be overly conservative,
 205:         // see also "A Simple Improved Inferential Method for Some
 206:         // Discrete Distributions" Yong CAI and K. KRISHNAMOORTHY
 207:         // http://www.ucs.louisiana.edu/~kxk4695/Discrete_new.pdf
 208:         //
 209:         return ibetac_inv(successes, failures, alpha, static_cast<RealType*>(nullptr), Policy());
 210:       } // find_upper_bound_on_p
 211: 
 212:       // Estimate number of trials :
 213:       // "How many trials do I need to be P% sure of seeing k or fewer failures?"
 214: 
 215:       BOOST_MATH_GPU_ENABLED static RealType find_minimum_number_of_trials(
 216:         RealType k,     // number of failures (k >= 0).
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:         RealType p,     // success fraction 0 <= p <= 1.
 218:         RealType alpha) // risk level threshold 0 <= alpha <= 1.
 219:       {
 220:         constexpr auto function = "boost::math::negative_binomial<%1%>::find_minimum_number_of_trials";
 221:         // Error checks:
 222:         RealType result = 0;
 223:         if(false == negative_binomial_detail::check_dist_and_k(
 224:           function, RealType(1), p, k, &result, Policy())
 225:           && detail::check_probability(function, alpha, &result, Policy()))
 226:         { return result; }
 227: 
 228:         result = ibeta_inva(k + 1, p, alpha, Policy());  // returns n - k
 229:         return result + k;
 230:       } // RealType find_number_of_failures
 231: 
 232:       BOOST_MATH_GPU_ENABLED static RealType find_maximum_number_of_trials(
 233:         RealType k,     // number of failures (k >= 0).
 234:         RealType p,     // success fraction 0 <= p <= 1.
~~~
- **EN:** This range declares or defines callable logic such as ibeta_inva. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ibeta_inva。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:         RealType alpha) // risk level threshold 0 <= alpha <= 1.
 236:       {
 237:         constexpr auto function = "boost::math::negative_binomial<%1%>::find_maximum_number_of_trials";
 238:         // Error checks:
 239:         RealType result = 0;
 240:         if(false == negative_binomial_detail::check_dist_and_k(
 241:           function, RealType(1), p, k, &result, Policy())
 242:           &&  detail::check_probability(function, alpha, &result, Policy()))
 243:         { return result; }
 244: 
 245:         result = ibetac_inva(k + 1, p, alpha, Policy());  // returns n - k
 246:         return result + k;
 247:       } // RealType find_number_of_trials complemented
 248: 
 249:     private:
 250:       RealType m_r; // successes.
 251:       RealType m_p; // success_fraction
 252:     }; // template <class RealType, class Policy> class negative_binomial_distribution
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as ibetac_inva.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 ibetac_inva。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253: 
 254:     typedef negative_binomial_distribution<double> negative_binomial; // Reserved name of type double.
 255: 
 256:     #ifdef __cpp_deduction_guides
 257:     template <class RealType>
 258:     negative_binomial_distribution(RealType,RealType)->negative_binomial_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 259:     #endif
 260: 
 261:     template <class RealType, class Policy>
 262:     BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const negative_binomial_distribution<RealType, Policy>& /* dist */)
 263:     { // Range of permissible values for random variable k.
 264:        using boost::math::tools::max_value;
 265:        return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>()); // max_integer?
 266:     }
 267: 
 268:     template <class RealType, class Policy>
 269:     BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const negative_binomial_distribution<RealType, Policy>& /* dist */)
 270:     { // Range of supported values for random variable k.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:        // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 272:        using boost::math::tools::max_value;
 273:        return boost::math::pair<RealType, RealType>(static_cast<RealType>(0),  max_value<RealType>()); // max_integer?
 274:     }
 275: 
 276:     template <class RealType, class Policy>
 277:     BOOST_MATH_GPU_ENABLED inline RealType mean(const negative_binomial_distribution<RealType, Policy>& dist)
 278:     { // Mean of Negative Binomial distribution = r(1-p)/p.
 279:       return dist.successes() * (1 - dist.success_fraction() ) / dist.success_fraction();
 280:     } // mean
 281: 
 282:     //template <class RealType, class Policy>
 283:     //inline RealType median(const negative_binomial_distribution<RealType, Policy>& dist)
 284:     //{ // Median of negative_binomial_distribution is not defined.
 285:     //  return policies::raise_domain_error<RealType>(BOOST_CURRENT_FUNCTION, "Median is not implemented, result is %1%!", std::numeric_limits<RealType>::quiet_NaN());
 286:     //} // median
 287:     // Now implemented via quantile(half) in derived accessors.
 288: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as quiet_NaN.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 quiet_NaN。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:     template <class RealType, class Policy>
 290:     BOOST_MATH_GPU_ENABLED inline RealType mode(const negative_binomial_distribution<RealType, Policy>& dist)
 291:     { // Mode of Negative Binomial distribution = floor[(r-1) * (1 - p)/p]
 292:       BOOST_MATH_STD_USING // ADL of std functions.
 293:       return floor((dist.successes() -1) * (1 - dist.success_fraction()) / dist.success_fraction());
 294:     } // mode
 295: 
 296:     template <class RealType, class Policy>
 297:     BOOST_MATH_GPU_ENABLED inline RealType skewness(const negative_binomial_distribution<RealType, Policy>& dist)
 298:     { // skewness of Negative Binomial distribution = 2-p / (sqrt(r(1-p))
 299:       BOOST_MATH_STD_USING // ADL of std functions.
 300:       RealType p = dist.success_fraction();
 301:       RealType r = dist.successes();
 302: 
 303:       return (2 - p) /
 304:         sqrt(r * (1 - p));
 305:     } // skewness
 306: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction, successes, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction, successes, ...。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:     template <class RealType, class Policy>
 308:     BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const negative_binomial_distribution<RealType, Policy>& dist)
 309:     { // kurtosis of Negative Binomial distribution
 310:       // http://en.wikipedia.org/wiki/Negative_binomial is kurtosis_excess so add 3
 311:       RealType p = dist.success_fraction();
 312:       RealType r = dist.successes();
 313:       return 3 + (6 / r) + ((p * p) / (r * (1 - p)));
 314:     } // kurtosis
 315: 
 316:      template <class RealType, class Policy>
 317:     BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const negative_binomial_distribution<RealType, Policy>& dist)
 318:     { // kurtosis excess of Negative Binomial distribution
 319:       // http://mathworld.wolfram.com/Kurtosis.html table of kurtosis_excess
 320:       RealType p = dist.success_fraction();
 321:       RealType r = dist.successes();
 322:       return (6 - p * (6-p)) / (r * (1-p));
 323:     } // kurtosis_excess
 324: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction, successes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction, successes。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:     template <class RealType, class Policy>
 326:     BOOST_MATH_GPU_ENABLED inline RealType variance(const negative_binomial_distribution<RealType, Policy>& dist)
 327:     { // Variance of Binomial distribution = r (1-p) / p^2.
 328:       return  dist.successes() * (1 - dist.success_fraction())
 329:         / (dist.success_fraction() * dist.success_fraction());
 330:     } // variance
 331: 
 332:     // RealType standard_deviation(const negative_binomial_distribution<RealType, Policy>& dist)
 333:     // standard_deviation provided by derived accessors.
 334:     // RealType hazard(const negative_binomial_distribution<RealType, Policy>& dist)
 335:     // hazard of Negative Binomial distribution provided by derived accessors.
 336:     // RealType chf(const negative_binomial_distribution<RealType, Policy>& dist)
 337:     // chf of Negative Binomial distribution provided by derived accessors.
 338: 
 339:     template <class RealType, class Policy>
 340:     BOOST_MATH_GPU_ENABLED inline RealType pdf(const negative_binomial_distribution<RealType, Policy>& dist, const RealType& k)
 341:     { // Probability Density/Mass Function.
 342:       BOOST_FPU_EXCEPTION_GUARD
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343: 
 344:       constexpr auto function = "boost::math::pdf(const negative_binomial_distribution<%1%>&, %1%)";
 345: 
 346:       RealType r = dist.successes();
 347:       RealType p = dist.success_fraction();
 348:       RealType result = 0;
 349:       if(false == negative_binomial_detail::check_dist_and_k(
 350:         function,
 351:         r,
 352:         dist.success_fraction(),
 353:         k,
 354:         &result, Policy()))
 355:       {
 356:         return result;
 357:       }
 358: 
 359:       result = (p/(r + k)) * ibeta_derivative(r, static_cast<RealType>(k+1), p, Policy());
 360:       // Equivalent to:
~~~
- **EN:** This range declares or defines callable logic such as successes, success_fraction, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 successes, success_fraction, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:       // return exp(lgamma(r + k) - lgamma(r) - lgamma(k+1)) * pow(p, r) * pow((1-p), k);
 362:       return result;
 363:     } // negative_binomial_pdf
 364: 
 365:     template <class RealType, class Policy>
 366:     BOOST_MATH_GPU_ENABLED inline RealType cdf(const negative_binomial_distribution<RealType, Policy>& dist, const RealType& k)
 367:     { // Cumulative Distribution Function of Negative Binomial.
 368:       constexpr auto function = "boost::math::cdf(const negative_binomial_distribution<%1%>&, %1%)";
 369:       using boost::math::ibeta; // Regularized incomplete beta function.
 370:       // k argument may be integral, signed, or unsigned, or floating point.
 371:       // If necessary, it has already been promoted from an integral type.
 372:       RealType p = dist.success_fraction();
 373:       RealType r = dist.successes();
 374:       // Error check:
 375:       RealType result = 0;
 376:       if(false == negative_binomial_detail::check_dist_and_k(
 377:         function,
 378:         r,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as exp, success_fraction, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 exp, success_fraction, ...。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:         dist.success_fraction(),
 380:         k,
 381:         &result, Policy()))
 382:       {
 383:         return result;
 384:       }
 385: 
 386:       RealType probability = ibeta(r, static_cast<RealType>(k+1), p, Policy());
 387:       // Ip(r, k+1) = ibeta(r, k+1, p)
 388:       return probability;
 389:     } // cdf Cumulative Distribution Function Negative Binomial.
 390: 
 391:       template <class RealType, class Policy>
 392:       BOOST_MATH_GPU_ENABLED inline RealType cdf(const complemented2_type<negative_binomial_distribution<RealType, Policy>, RealType>& c)
 393:       { // Complemented Cumulative Distribution Function Negative Binomial.
 394: 
 395:       constexpr auto function = "boost::math::cdf(const negative_binomial_distribution<%1%>&, %1%)";
 396:       using boost::math::ibetac; // Regularized incomplete beta function complement.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as ibeta.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 ibeta。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:       // k argument may be integral, signed, or unsigned, or floating point.
 398:       // If necessary, it has already been promoted from an integral type.
 399:       RealType const& k = c.param;
 400:       negative_binomial_distribution<RealType, Policy> const& dist = c.dist;
 401:       RealType p = dist.success_fraction();
 402:       RealType r = dist.successes();
 403:       // Error check:
 404:       RealType result = 0;
 405:       if(false == negative_binomial_detail::check_dist_and_k(
 406:         function,
 407:         r,
 408:         p,
 409:         k,
 410:         &result, Policy()))
 411:       {
 412:         return result;
 413:       }
 414:       // Calculate cdf negative binomial using the incomplete beta function.
~~~
- **EN:** This range declares or defines callable logic such as success_fraction, successes. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 success_fraction, successes。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:       // Use of ibeta here prevents cancellation errors in calculating
 416:       // 1-p if p is very small, perhaps smaller than machine epsilon.
 417:       // Ip(k+1, r) = ibetac(r, k+1, p)
 418:       // constrain_probability here?
 419:      RealType probability = ibetac(r, static_cast<RealType>(k+1), p, Policy());
 420:       // Numerical errors might cause probability to be slightly outside the range < 0 or > 1.
 421:       // This might cause trouble downstream, so warn, possibly throw exception, but constrain to the limits.
 422:       return probability;
 423:     } // cdf Cumulative Distribution Function Negative Binomial.
 424: 
 425:     template <class RealType, class Policy>
 426:     BOOST_MATH_GPU_ENABLED inline RealType quantile(const negative_binomial_distribution<RealType, Policy>& dist, const RealType& P)
 427:     { // Quantile, percentile/100 or Percent Point Negative Binomial function.
 428:       // Return the number of expected failures k for a given probability p.
 429: 
 430:       // Inverse cumulative Distribution Function or Quantile (percentile / 100) of negative_binomial Probability.
 431:       // MAthCAD pnbinom return smallest k such that negative_binomial(k, n, p) >= probability.
 432:       // k argument may be integral, signed, or unsigned, or floating point.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as ibetac.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 ibetac。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:       // BUT Cephes/CodeCogs says: finds argument p (0 to 1) such that cdf(k, n, p) = y
 434:       constexpr auto function = "boost::math::quantile(const negative_binomial_distribution<%1%>&, %1%)";
 435:       BOOST_MATH_STD_USING // ADL of std functions.
 436: 
 437:       RealType p = dist.success_fraction();
 438:       RealType r = dist.successes();
 439:       // Check dist and P.
 440:       RealType result = 0;
 441:       if(false == negative_binomial_detail::check_dist_and_prob
 442:         (function, r, p, P, &result, Policy()))
 443:       {
 444:         return result;
 445:       }
 446: 
 447:       // Special cases.
 448:       if (P == 1)
 449:       {  // Would need +infinity failures for total confidence.
 450:         result = policies::raise_overflow_error<RealType>(
~~~
- **EN:** This range declares or defines callable logic such as success_fraction, successes. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 success_fraction, successes。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:             function,
 452:             "Probability argument is 1, which implies infinite failures !", Policy());
 453:         return result;
 454:        // usually means return +std::numeric_limits<RealType>::infinity();
 455:        // unless #define BOOST_MATH_THROW_ON_OVERFLOW_ERROR
 456:       }
 457:       if (P == 0)
 458:       { // No failures are expected if P = 0.
 459:         return 0; // Total trials will be just dist.successes.
 460:       }
 461:       if (P <= pow(dist.success_fraction(), dist.successes()))
 462:       { // p <= pdf(dist, 0) == cdf(dist, 0)
 463:         return 0;
 464:       }
 465:       if(p == 0)
 466:       {  // Would need +infinity failures for total confidence.
 467:          result = policies::raise_overflow_error<RealType>(
 468:             function,
~~~
- **EN:** This range declares or defines callable logic such as Policy, infinity. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy, infinity。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:             "Success fraction is 0, which implies infinite failures !", Policy());
 470:          return result;
 471:          // usually means return +std::numeric_limits<RealType>::infinity();
 472:          // unless #define BOOST_MATH_THROW_ON_OVERFLOW_ERROR
 473:       }
 474:       /*
 475:       // Calculate quantile of negative_binomial using the inverse incomplete beta function.
 476:       using boost::math::ibeta_invb;
 477:       return ibeta_invb(r, p, P, Policy()) - 1; //
 478:       */
 479:       RealType guess = 0;
 480:       RealType factor = 5;
 481:       if(r * r * r * P * p > 0.005)
 482:          guess = detail::inverse_negative_binomial_cornish_fisher(r, p, RealType(1-p), P, RealType(1-P), Policy());
 483: 
 484:       if(guess < 10)
 485:       {
 486:          //
~~~
- **EN:** This range declares or defines callable logic such as Policy, infinity, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy, infinity, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:          // Cornish-Fisher Negative binomial approximation not accurate in this area:
 488:          //
 489:          guess = BOOST_MATH_GPU_SAFE_MIN(RealType(r * 2), RealType(10));
 490:       }
 491:       else
 492:          factor = (1-P < sqrt(tools::epsilon<RealType>())) ? 2 : (guess < 20 ? 1.2f : 1.1f);
 493:       BOOST_MATH_INSTRUMENT_CODE("guess = " << guess);
 494:       //
 495:       // Max iterations permitted:
 496:       //
 497:       boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 498:       typedef typename Policy::discrete_quantile_type discrete_type;
 499:       return detail::inverse_discrete_quantile(
 500:          dist,
 501:          P,
 502:          false,
 503:          guess,
 504:          factor,
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_GPU_SAFE_MIN, sqrt, .... Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_GPU_SAFE_MIN, sqrt, ...。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:          RealType(1),
 506:          discrete_type(),
 507:          max_iter);
 508:     } // RealType quantile(const negative_binomial_distribution dist, p)
 509: 
 510:     template <class RealType, class Policy>
 511:     BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<negative_binomial_distribution<RealType, Policy>, RealType>& c)
 512:     {  // Quantile or Percent Point Binomial function.
 513:        // Return the number of expected failures k for a given
 514:        // complement of the probability Q = 1 - P.
 515:        constexpr auto function = "boost::math::quantile(const negative_binomial_distribution<%1%>&, %1%)";
 516:        BOOST_MATH_STD_USING
 517: 
 518:        // Error checks:
 519:        RealType Q = c.param;
 520:        const negative_binomial_distribution<RealType, Policy>& dist = c.dist;
 521:        RealType p = dist.success_fraction();
 522:        RealType r = dist.successes();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction, successes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction, successes。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:        RealType result = 0;
 524:        if(false == negative_binomial_detail::check_dist_and_prob(
 525:           function,
 526:           r,
 527:           p,
 528:           Q,
 529:           &result, Policy()))
 530:        {
 531:           return result;
 532:        }
 533: 
 534:        // Special cases:
 535:        //
 536:        if(Q == 1)
 537:        {  // There may actually be no answer to this question,
 538:           // since the probability of zero failures may be non-zero,
 539:           return 0; // but zero is the best we can do:
 540:        }
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:        if(Q == 0)
 542:        {  // Probability 1 - Q  == 1 so infinite failures to achieve certainty.
 543:           // Would need +infinity failures for total confidence.
 544:           result = policies::raise_overflow_error<RealType>(
 545:              function,
 546:              "Probability argument complement is 0, which implies infinite failures !", Policy());
 547:           return result;
 548:           // usually means return +std::numeric_limits<RealType>::infinity();
 549:           // unless #define BOOST_MATH_THROW_ON_OVERFLOW_ERROR
 550:        }
 551:        if (-Q <= boost::math::powm1(dist.success_fraction(), dist.successes(), Policy()))
 552:        {  // q <= cdf(complement(dist, 0)) == pdf(dist, 0)
 553:           return 0; //
 554:        }
 555:        if(p == 0)
 556:        {  // Success fraction is 0 so infinite failures to achieve certainty.
 557:           // Would need +infinity failures for total confidence.
 558:           result = policies::raise_overflow_error<RealType>(
~~~
- **EN:** This range declares or defines callable logic such as Policy, infinity. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy, infinity。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 559-576 / 第 559-576 行
~~~cpp
 559:              function,
 560:              "Success fraction is 0, which implies infinite failures !", Policy());
 561:           return result;
 562:           // usually means return +std::numeric_limits<RealType>::infinity();
 563:           // unless #define BOOST_MATH_THROW_ON_OVERFLOW_ERROR
 564:        }
 565:        //return ibetac_invb(r, p, Q, Policy()) -1;
 566:        RealType guess = 0;
 567:        RealType factor = 5;
 568:        if(r * r * r * (1-Q) * p > 0.005)
 569:           guess = detail::inverse_negative_binomial_cornish_fisher(r, p, RealType(1-p), RealType(1-Q), Q, Policy());
 570: 
 571:        if(guess < 10)
 572:        {
 573:           //
 574:           // Cornish-Fisher Negative binomial approximation not accurate in this area:
 575:           //
 576:           guess = BOOST_MATH_GPU_SAFE_MIN(RealType(r * 2), RealType(10));
~~~
- **EN:** This range declares or defines callable logic such as Policy, infinity, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy, infinity, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 577-594 / 第 577-594 行
~~~cpp
 577:        }
 578:        else
 579:           factor = (Q < sqrt(tools::epsilon<RealType>())) ? 2 : (guess < 20 ? 1.2f : 1.1f);
 580:        BOOST_MATH_INSTRUMENT_CODE("guess = " << guess);
 581:        //
 582:        // Max iterations permitted:
 583:        //
 584:        boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 585:        typedef typename Policy::discrete_quantile_type discrete_type;
 586:        return detail::inverse_discrete_quantile(
 587:           dist,
 588:           Q,
 589:           true,
 590:           guess,
 591:           factor,
 592:           RealType(1),
 593:           discrete_type(),
 594:           max_iter);
~~~
- **EN:** This range declares or defines callable logic such as sqrt, BOOST_MATH_INSTRUMENT_CODE. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, BOOST_MATH_INSTRUMENT_CODE。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 595-609 / 第 595-609 行
~~~cpp
 595:     } // quantile complement
 596: 
 597:  } // namespace math
 598: } // namespace boost
 599: 
 600: // This include must be at the end, *after* the accessors
 601: // for this distribution have been defined, in order to
 602: // keep compilers that support two-phase lookup happy.
 603: #include <boost/math/distributions/detail/derived_accessors.hpp>
 604: 
 605: #if defined (BOOST_MSVC)
 606: # pragma warning(pop)
 607: #endif
 608: 
 609: #endif // BOOST_MATH_SPECIAL_NEGATIVE_BINOMIAL_HPP
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp so the surrounding code can use external declarations. The code enters namespace scope (math::boost) to keep symbols organized. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/cstdint.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/beta.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/special_functions/fpclassify.hpp, boost/math/tools/roots.hpp, boost/math/distributions/detail/inv_discrete_quantile.hpp, boost/math/policies/error_handling.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, negative_binomial_detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `check_successes, Policy, ibeta_inva, ibetac_inva, quiet_NaN, success_fraction, successes, sqrt, ...`
