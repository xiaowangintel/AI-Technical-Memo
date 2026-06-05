# geometric.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/geometric.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the geometric distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 geometric 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // boost\math\distributions\geometric.hpp
   2: 
   3: // Copyright John Maddock 2010.
   4: // Copyright Paul A. Bristow 2010.
   5: 
   6: // Use, modification and distribution are subject to the
   7: // Boost Software License, Version 1.0.
   8: // (See accompanying file LICENSE_1_0.txt
   9: // or copy at http://www.boost.org/LICENSE_1_0.txt)
  10: 
  11: // geometric distribution is a discrete probability distribution.
  12: // It expresses the probability distribution of the number (k) of
  13: // events, occurrences, failures or arrivals before the first success.
  14: // supported on the set {0, 1, 2, 3...}
  15: 
  16: // Note that the set includes zero (unlike some definitions that start at one).
  17: 
  18: // The random variate k is the number of events, occurrences or arrivals.
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: // k argument may be integral, signed, or unsigned, or floating point.
  20: // If necessary, it has already been promoted from an integral type.
  21: 
  22: // Note that the geometric distribution
  23: // (like others including the binomial, geometric & Bernoulli)
  24: // is strictly defined as a discrete function:
  25: // only integral values of k are envisaged.
  26: // However because the method of calculation uses a continuous gamma function,
  27: // it is convenient to treat it as if a continuous function,
  28: // and permit non-integral values of k.
  29: // To enforce the strict mathematical model, users should use floor or ceil functions
  30: // on k outside this function to ensure that k is integral.
  31: 
  32: // See http://en.wikipedia.org/wiki/geometric_distribution
  33: // http://documents.wolfram.com/v5/Add-onsLinks/StandardPackages/Statistics/DiscreteDistributions.html
  34: // http://mathworld.wolfram.com/GeometricDistribution.html
  35: 
  36: #ifndef BOOST_MATH_SPECIAL_GEOMETRIC_HPP
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: #define BOOST_MATH_SPECIAL_GEOMETRIC_HPP
  38: 
  39: #include <boost/math/tools/config.hpp>
  40: #include <boost/math/tools/tuple.hpp>
  41: #include <boost/math/tools/numeric_limits.hpp>
  42: #include <boost/math/distributions/fwd.hpp>
  43: #include <boost/math/special_functions/beta.hpp> // for ibeta(a, b, x) == Ix(a, b).
  44: #include <boost/math/distributions/complement.hpp> // complement.
  45: #include <boost/math/distributions/detail/common_error_handling.hpp> // error checks domain_error & logic_error.
  46: #include <boost/math/special_functions/fpclassify.hpp> // isnan.
  47: #include <boost/math/tools/roots.hpp> // for root finding.
  48: #include <boost/math/distributions/detail/inv_discrete_quantile.hpp>
  49: #include <boost/math/special_functions/log1p.hpp>
  50: 
  51: #if defined (BOOST_MSVC)
  52: #  pragma warning(push)
  53: // This believed not now necessary, so commented out.
  54: //#  pragma warning(disable: 4702) // unreachable code.
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, ... so the surrounding code can use external declarations. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, ... 等依赖，使周围代码可以使用外部声明。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: // in domain_error_imp in error_handling.
  56: #endif
  57: 
  58: namespace boost
  59: {
  60:   namespace math
  61:   {
  62:     namespace geometric_detail
  63:     {
  64:       // Common error checking routines for geometric distribution function:
  65:       template <class RealType, class Policy>
  66:       BOOST_MATH_GPU_ENABLED inline bool check_success_fraction(const char* function, const RealType& p, RealType* result, const Policy& pol)
  67:       {
  68:         if( !(boost::math::isfinite)(p) || (p < 0) || (p > 1) )
  69:         {
  70:           *result = policies::raise_domain_error<RealType>(
  71:             function,
  72:             "Success fraction argument is %1%, but must be >= 0 and <= 1 !", p, pol);
~~~
- **EN:** The code enters namespace scope (boost::math::geometric_detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost::math::geometric_detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:           return false;
  74:         }
  75:         return true;
  76:       }
  77: 
  78:       template <class RealType, class Policy>
  79:       BOOST_MATH_GPU_ENABLED inline bool check_dist(const char* function, const RealType& p, RealType* result, const Policy& pol)
  80:       {
  81:         return check_success_fraction(function, p, result, pol);
  82:       }
  83: 
  84:       template <class RealType, class Policy>
  85:       BOOST_MATH_GPU_ENABLED inline bool check_dist_and_k(const char* function,  const RealType& p, RealType k, RealType* result, const Policy& pol)
  86:       {
  87:         if(check_dist(function, p, result, pol) == false)
  88:         {
  89:           return false;
  90:         }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:         if( !(boost::math::isfinite)(k) || (k < 0) )
  92:         { // Check k failures.
  93:           *result = policies::raise_domain_error<RealType>(
  94:             function,
  95:             "Number of failures argument is %1%, but must be >= 0 !", k, pol);
  96:           return false;
  97:         }
  98:         return true;
  99:       } // Check_dist_and_k
 100: 
 101:       template <class RealType, class Policy>
 102:       BOOST_MATH_GPU_ENABLED inline bool check_dist_and_prob(const char* function, RealType p, RealType prob, RealType* result, const Policy& pol)
 103:       {
 104:         if((check_dist(function, p, result, pol) && detail::check_probability(function, prob, result, pol)) == false)
 105:         {
 106:           return false;
 107:         }
 108:         return true;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:       } // check_dist_and_prob
 110:     } //  namespace geometric_detail
 111: 
 112:     template <class RealType = double, class Policy = policies::policy<> >
 113:     class geometric_distribution
 114:     {
 115:     public:
 116:       typedef RealType value_type;
 117:       typedef Policy policy_type;
 118: 
 119:       BOOST_MATH_GPU_ENABLED geometric_distribution(RealType p) : m_p(p)
 120:       { // Constructor stores success_fraction p.
 121:         RealType result;
 122:         geometric_detail::check_dist(
 123:           "geometric_distribution<%1%>::geometric_distribution",
 124:           m_p, // Check success_fraction 0 <= p <= 1.
 125:           &result, Policy());
 126:       } // geometric_distribution constructor.
~~~
- **EN:** The code enters namespace scope (geometric_detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（geometric_detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127: 
 128:       // Private data getter class member functions.
 129:       BOOST_MATH_GPU_ENABLED RealType success_fraction() const
 130:       { // Probability of success as fraction in range 0 to 1.
 131:         return m_p;
 132:       }
 133:       BOOST_MATH_GPU_ENABLED RealType successes() const
 134:       { // Total number of successes r = 1 (for compatibility with negative binomial?).
 135:         return 1;
 136:       }
 137: 
 138:       // Parameter estimation.
 139:       // (These are copies of negative_binomial distribution with successes = 1).
 140:       BOOST_MATH_GPU_ENABLED static RealType find_lower_bound_on_p(
 141:         RealType trials,
 142:         RealType alpha) // alpha 0.05 equivalent to 95% for one-sided test.
 143:       {
 144:         constexpr auto function = "boost::math::geometric<%1%>::find_lower_bound_on_p";
~~~
- **EN:** It introduces the class `member` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 它引入了 class `member`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:         RealType result = 0;  // of error checks.
 146:         RealType successes = 1;
 147:         RealType failures = trials - successes;
 148:         if(false == detail::check_probability(function, alpha, &result, Policy())
 149:           && geometric_detail::check_dist_and_k(
 150:           function, RealType(0), failures, &result, Policy()))
 151:         {
 152:           return result;
 153:         }
 154:         // Use complement ibeta_inv function for lower bound.
 155:         // This is adapted from the corresponding binomial formula
 156:         // here: http://www.itl.nist.gov/div898/handbook/prc/section2/prc241.htm
 157:         // This is a Clopper-Pearson interval, and may be overly conservative,
 158:         // see also "A Simple Improved Inferential Method for Some
 159:         // Discrete Distributions" Yong CAI and K. KRISHNAMOORTHY
 160:         // http://www.ucs.louisiana.edu/~kxk4695/Discrete_new.pdf
 161:         //
 162:         return ibeta_inv(successes, failures + 1, alpha, static_cast<RealType*>(nullptr), Policy());
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:       } // find_lower_bound_on_p
 164: 
 165:       BOOST_MATH_GPU_ENABLED static RealType find_upper_bound_on_p(
 166:         RealType trials,
 167:         RealType alpha) // alpha 0.05 equivalent to 95% for one-sided test.
 168:       {
 169:         constexpr auto function = "boost::math::geometric<%1%>::find_upper_bound_on_p";
 170:         RealType result = 0;  // of error checks.
 171:         RealType successes = 1;
 172:         RealType failures = trials - successes;
 173:         if(false == geometric_detail::check_dist_and_k(
 174:           function, RealType(0), failures, &result, Policy())
 175:           && detail::check_probability(function, alpha, &result, Policy()))
 176:         {
 177:           return result;
 178:         }
 179:         if(failures == 0)
 180:         {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:            return 1;
 182:         }// Use complement ibetac_inv function for upper bound.
 183:         // Note adjusted failures value: *not* failures+1 as usual.
 184:         // This is adapted from the corresponding binomial formula
 185:         // here: http://www.itl.nist.gov/div898/handbook/prc/section2/prc241.htm
 186:         // This is a Clopper-Pearson interval, and may be overly conservative,
 187:         // see also "A Simple Improved Inferential Method for Some
 188:         // Discrete Distributions" Yong CAI and K. Krishnamoorthy
 189:         // http://www.ucs.louisiana.edu/~kxk4695/Discrete_new.pdf
 190:         //
 191:         return ibetac_inv(successes, failures, alpha, static_cast<RealType*>(nullptr), Policy());
 192:       } // find_upper_bound_on_p
 193: 
 194:       // Estimate number of trials :
 195:       // "How many trials do I need to be P% sure of seeing k or fewer failures?"
 196: 
 197:       BOOST_MATH_GPU_ENABLED static RealType find_minimum_number_of_trials(
 198:         RealType k,     // number of failures (k >= 0).
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:         RealType p,     // success fraction 0 <= p <= 1.
 200:         RealType alpha) // risk level threshold 0 <= alpha <= 1.
 201:       {
 202:         constexpr auto function = "boost::math::geometric<%1%>::find_minimum_number_of_trials";
 203:         // Error checks:
 204:         RealType result = 0;
 205:         if(false == geometric_detail::check_dist_and_k(
 206:           function, p, k, &result, Policy())
 207:           && detail::check_probability(function, alpha, &result, Policy()))
 208:         {
 209:           return result;
 210:         }
 211:         result = ibeta_inva(k + 1, p, alpha, Policy());  // returns n - k
 212:         return result + k;
 213:       } // RealType find_number_of_failures
 214: 
 215:       BOOST_MATH_GPU_ENABLED static RealType find_maximum_number_of_trials(
 216:         RealType k,     // number of failures (k >= 0).
~~~
- **EN:** This range declares or defines callable logic such as ibeta_inva. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ibeta_inva。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:         RealType p,     // success fraction 0 <= p <= 1.
 218:         RealType alpha) // risk level threshold 0 <= alpha <= 1.
 219:       {
 220:         constexpr auto function = "boost::math::geometric<%1%>::find_maximum_number_of_trials";
 221:         // Error checks:
 222:         RealType result = 0;
 223:         if(false == geometric_detail::check_dist_and_k(
 224:           function, p, k, &result, Policy())
 225:           &&  detail::check_probability(function, alpha, &result, Policy()))
 226:         {
 227:           return result;
 228:         }
 229:         result = ibetac_inva(k + 1, p, alpha, Policy());  // returns n - k
 230:         return result + k;
 231:       } // RealType find_number_of_trials complemented
 232: 
 233:     private:
 234:       //RealType m_r; // successes fixed at unity.
~~~
- **EN:** This range declares or defines callable logic such as ibetac_inva. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ibetac_inva。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:       RealType m_p; // success_fraction
 236:     }; // template <class RealType, class Policy> class geometric_distribution
 237: 
 238:     typedef geometric_distribution<double> geometric; // Reserved name of type double.
 239: 
 240:     #ifdef __cpp_deduction_guides
 241:     template <class RealType>
 242:     geometric_distribution(RealType)->geometric_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 243:     #endif
 244: 
 245:     template <class RealType, class Policy>
 246:     BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const geometric_distribution<RealType, Policy>& /* dist */)
 247:     { // Range of permissible values for random variable k.
 248:        using boost::math::tools::max_value;
 249:        return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>()); // max_integer?
 250:     }
 251: 
 252:     template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:     BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const geometric_distribution<RealType, Policy>& /* dist */)
 254:     { // Range of supported values for random variable k.
 255:        // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 256:        using boost::math::tools::max_value;
 257:        return boost::math::pair<RealType, RealType>(static_cast<RealType>(0),  max_value<RealType>()); // max_integer?
 258:     }
 259: 
 260:     template <class RealType, class Policy>
 261:     BOOST_MATH_GPU_ENABLED inline RealType mean(const geometric_distribution<RealType, Policy>& dist)
 262:     { // Mean of geometric distribution = (1-p)/p.
 263:       return (1 - dist.success_fraction() ) / dist.success_fraction();
 264:     } // mean
 265: 
 266:     // median implemented via quantile(half) in derived accessors.
 267: 
 268:     template <class RealType, class Policy>
 269:     BOOST_MATH_GPU_ENABLED inline RealType mode(const geometric_distribution<RealType, Policy>&)
 270:     { // Mode of geometric distribution = zero.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:       BOOST_MATH_STD_USING // ADL of std functions.
 272:       return 0;
 273:     } // mode
 274: 
 275:     template <class RealType, class Policy>
 276:     BOOST_MATH_GPU_ENABLED inline RealType variance(const geometric_distribution<RealType, Policy>& dist)
 277:     { // Variance of Binomial distribution = (1-p) / p^2.
 278:       return  (1 - dist.success_fraction())
 279:         / (dist.success_fraction() * dist.success_fraction());
 280:     } // variance
 281: 
 282:     template <class RealType, class Policy>
 283:     BOOST_MATH_GPU_ENABLED inline RealType skewness(const geometric_distribution<RealType, Policy>& dist)
 284:     { // skewness of geometric distribution = 2-p / (sqrt(r(1-p))
 285:       BOOST_MATH_STD_USING // ADL of std functions.
 286:       RealType p = dist.success_fraction();
 287:       return (2 - p) / sqrt(1 - p);
 288:     } // skewness
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289: 
 290:     template <class RealType, class Policy>
 291:     BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const geometric_distribution<RealType, Policy>& dist)
 292:     { // kurtosis of geometric distribution
 293:       // http://en.wikipedia.org/wiki/geometric is kurtosis_excess so add 3
 294:       RealType p = dist.success_fraction();
 295:       return 3 + (p*p - 6*p + 6) / (1 - p);
 296:     } // kurtosis
 297: 
 298:      template <class RealType, class Policy>
 299:     BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const geometric_distribution<RealType, Policy>& dist)
 300:     { // kurtosis excess of geometric distribution
 301:       // http://mathworld.wolfram.com/Kurtosis.html table of kurtosis_excess
 302:       RealType p = dist.success_fraction();
 303:       return (p*p - 6*p + 6) / (1 - p);
 304:     } // kurtosis_excess
 305: 
 306:     // RealType standard_deviation(const geometric_distribution<RealType, Policy>& dist)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:     // standard_deviation provided by derived accessors.
 308:     // RealType hazard(const geometric_distribution<RealType, Policy>& dist)
 309:     // hazard of geometric distribution provided by derived accessors.
 310:     // RealType chf(const geometric_distribution<RealType, Policy>& dist)
 311:     // chf of geometric distribution provided by derived accessors.
 312: 
 313:     template <class RealType, class Policy>
 314:     BOOST_MATH_GPU_ENABLED inline RealType pdf(const geometric_distribution<RealType, Policy>& dist, const RealType& k)
 315:     { // Probability Density/Mass Function.
 316:       BOOST_FPU_EXCEPTION_GUARD
 317:       BOOST_MATH_STD_USING  // For ADL of math functions.
 318:       constexpr auto function = "boost::math::pdf(const geometric_distribution<%1%>&, %1%)";
 319: 
 320:       RealType p = dist.success_fraction();
 321:       RealType result = 0;
 322:       if(false == geometric_detail::check_dist_and_k(
 323:         function,
 324:         p,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:         k,
 326:         &result, Policy()))
 327:       {
 328:         return result;
 329:       }
 330:       if (k == 0)
 331:       {
 332:         return p; // success_fraction
 333:       }
 334:       RealType q = 1 - p;  // Inaccurate for small p?
 335:       // So try to avoid inaccuracy for large or small p.
 336:       // but has little effect > last significant bit.
 337:       //cout << "p *  pow(q, k) " << result << endl; // seems best whatever p
 338:       //cout << "exp(p * k * log1p(-p)) " << p * exp(k * log1p(-p)) << endl;
 339:       //if (p < 0.5)
 340:       //{
 341:       //  result = p *  pow(q, k);
 342:       //}
~~~
- **EN:** This range declares or defines callable logic such as pow. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:       //else
 344:       //{
 345:       //  result = p * exp(k * log1p(-p));
 346:       //}
 347:       result = p * pow(q, k);
 348:       return result;
 349:     } // geometric_pdf
 350: 
 351:     template <class RealType, class Policy>
 352:     BOOST_MATH_GPU_ENABLED inline RealType cdf(const geometric_distribution<RealType, Policy>& dist, const RealType& k)
 353:     { // Cumulative Distribution Function of geometric.
 354:       constexpr auto function = "boost::math::cdf(const geometric_distribution<%1%>&, %1%)";
 355: 
 356:       // k argument may be integral, signed, or unsigned, or floating point.
 357:       // If necessary, it has already been promoted from an integral type.
 358:       RealType p = dist.success_fraction();
 359:       // Error check:
 360:       RealType result = 0;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as exp, pow, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 exp, pow, ...。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:       if(false == geometric_detail::check_dist_and_k(
 362:         function,
 363:         p,
 364:         k,
 365:         &result, Policy()))
 366:       {
 367:         return result;
 368:       }
 369:       if(k == 0)
 370:       {
 371:         return p; // success_fraction
 372:       }
 373:       //RealType q = 1 - p;  // Bad for small p
 374:       //RealType probability = 1 - std::pow(q, k+1);
 375: 
 376:       RealType z = boost::math::log1p(-p, Policy()) * (k + 1);
 377:       RealType probability = -boost::math::expm1(z, Policy());
 378: 
~~~
- **EN:** This range declares or defines callable logic such as std::pow, boost::math::log1p, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::pow, boost::math::log1p, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:       return probability;
 380:     } // cdf Cumulative Distribution Function geometric.
 381: 
 382:     template <class RealType, class Policy>
 383:     BOOST_MATH_GPU_ENABLED inline RealType logcdf(const geometric_distribution<RealType, Policy>& dist, const RealType& k)
 384:     { // Cumulative Distribution Function of geometric.
 385:       BOOST_MATH_STD_USING
 386:       constexpr auto function = "boost::math::logcdf(const geometric_distribution<%1%>&, %1%)";
 387: 
 388:       // k argument may be integral, signed, or unsigned, or floating point.
 389:       // If necessary, it has already been promoted from an integral type.
 390:       RealType p = dist.success_fraction();
 391:       // Error check:
 392:       RealType result = 0;
 393:       if(false == geometric_detail::check_dist_and_k(
 394:         function,
 395:         p,
 396:         k,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:         &result, Policy()))
 398:       {
 399:         return -boost::math::numeric_limits<RealType>::infinity();
 400:       }
 401:       if(k == 0)
 402:       {
 403:         return log(p); // success_fraction
 404:       }
 405:       //RealType q = 1 - p;  // Bad for small p
 406:       //RealType probability = 1 - std::pow(q, k+1);
 407: 
 408:       RealType z = boost::math::log1p(-p, Policy()) * (k + 1);
 409:       return log1p(-exp(z), Policy());
 410:     } // logcdf Cumulative Distribution Function geometric.
 411: 
 412:     template <class RealType, class Policy>
 413:     BOOST_MATH_GPU_ENABLED inline RealType cdf(const complemented2_type<geometric_distribution<RealType, Policy>, RealType>& c)
 414:     { // Complemented Cumulative Distribution Function geometric.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as std::pow, boost::math::log1p.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 std::pow, boost::math::log1p。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:       BOOST_MATH_STD_USING
 416:       constexpr auto function = "boost::math::cdf(const geometric_distribution<%1%>&, %1%)";
 417:       // k argument may be integral, signed, or unsigned, or floating point.
 418:       // If necessary, it has already been promoted from an integral type.
 419:       RealType const& k = c.param;
 420:       geometric_distribution<RealType, Policy> const& dist = c.dist;
 421:       RealType p = dist.success_fraction();
 422:       // Error check:
 423:       RealType result = 0;
 424:       if(false == geometric_detail::check_dist_and_k(
 425:         function,
 426:         p,
 427:         k,
 428:         &result, Policy()))
 429:       {
 430:         return result;
 431:       }
 432:       RealType z = boost::math::log1p(-p, Policy()) * (k+1);
~~~
- **EN:** This range declares or defines callable logic such as success_fraction, boost::math::log1p. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 success_fraction, boost::math::log1p。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:       RealType probability = exp(z);
 434:       return probability;
 435:     } // cdf Complemented Cumulative Distribution Function geometric.
 436: 
 437:     template <class RealType, class Policy>
 438:     BOOST_MATH_GPU_ENABLED inline RealType logcdf(const complemented2_type<geometric_distribution<RealType, Policy>, RealType>& c)
 439:     { // Complemented Cumulative Distribution Function geometric.
 440:       BOOST_MATH_STD_USING
 441:       constexpr auto function = "boost::math::logcdf(const geometric_distribution<%1%>&, %1%)";
 442:       // k argument may be integral, signed, or unsigned, or floating point.
 443:       // If necessary, it has already been promoted from an integral type.
 444:       RealType const& k = c.param;
 445:       geometric_distribution<RealType, Policy> const& dist = c.dist;
 446:       RealType p = dist.success_fraction();
 447:       // Error check:
 448:       RealType result = 0;
 449:       if(false == geometric_detail::check_dist_and_k(
 450:         function,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as exp, success_fraction.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 exp, success_fraction。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:         p,
 452:         k,
 453:         &result, Policy()))
 454:       {
 455:         return -boost::math::numeric_limits<RealType>::infinity();
 456:       }
 457: 
 458:       return boost::math::log1p(-p, Policy()) * (k+1);
 459:     } // logcdf Complemented Cumulative Distribution Function geometric.
 460: 
 461:     template <class RealType, class Policy>
 462:     BOOST_MATH_GPU_ENABLED inline RealType quantile(const geometric_distribution<RealType, Policy>& dist, const RealType& x)
 463:     { // Quantile, percentile/100 or Percent Point geometric function.
 464:       // Return the number of expected failures k for a given probability p.
 465: 
 466:       // Inverse cumulative Distribution Function or Quantile (percentile / 100) of geometric Probability.
 467:       // k argument may be integral, signed, or unsigned, or floating point.
 468: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:       constexpr auto function = "boost::math::quantile(const geometric_distribution<%1%>&, %1%)";
 470:       BOOST_MATH_STD_USING // ADL of std functions.
 471: 
 472:       RealType success_fraction = dist.success_fraction();
 473:       // Check dist and x.
 474:       RealType result = 0;
 475:       if(false == geometric_detail::check_dist_and_prob
 476:         (function, success_fraction, x, &result, Policy()))
 477:       {
 478:         return result;
 479:       }
 480: 
 481:       // Special cases.
 482:       if (x == 1)
 483:       {  // Would need +infinity failures for total confidence.
 484:         result = policies::raise_overflow_error<RealType>(
 485:             function,
 486:             "Probability argument is 1, which implies infinite failures !", Policy());
~~~
- **EN:** This range declares or defines callable logic such as success_fraction, Policy. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 success_fraction, Policy。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:         return result;
 488:        // usually means return +std::numeric_limits<RealType>::infinity();
 489:        // unless #define BOOST_MATH_THROW_ON_OVERFLOW_ERROR
 490:       }
 491:       if (x == 0)
 492:       { // No failures are expected if P = 0.
 493:         return 0; // Total trials will be just dist.successes.
 494:       }
 495:       // if (P <= pow(dist.success_fraction(), 1))
 496:       if (x <= success_fraction)
 497:       { // p <= pdf(dist, 0) == cdf(dist, 0)
 498:         return 0;
 499:       }
 500:       if (x == 1)
 501:       {
 502:         return 0;
 503:       }
 504: 
~~~
- **EN:** This range declares or defines callable logic such as infinity. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 infinity。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:       // log(1-x) /log(1-success_fraction) -1; but use log1p in case success_fraction is small
 506:       result = boost::math::log1p(-x, Policy()) / boost::math::log1p(-success_fraction, Policy()) - 1;
 507:       // Subtract a few epsilons here too?
 508:       // to make sure it doesn't slip over, so ceil would be one too many.
 509:       return result;
 510:     } // RealType quantile(const geometric_distribution dist, p)
 511: 
 512:     template <class RealType, class Policy>
 513:     BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<geometric_distribution<RealType, Policy>, RealType>& c)
 514:     {  // Quantile or Percent Point Binomial function.
 515:        // Return the number of expected failures k for a given
 516:        // complement of the probability Q = 1 - P.
 517:        constexpr auto function = "boost::math::quantile(const geometric_distribution<%1%>&, %1%)";
 518:        BOOST_MATH_STD_USING
 519:        // Error checks:
 520:        RealType x = c.param;
 521:        const geometric_distribution<RealType, Policy>& dist = c.dist;
 522:        RealType success_fraction = dist.success_fraction();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as success_fraction.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 success_fraction。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:        RealType result = 0;
 524:        if(false == geometric_detail::check_dist_and_prob(
 525:           function,
 526:           success_fraction,
 527:           x,
 528:           &result, Policy()))
 529:        {
 530:           return result;
 531:        }
 532: 
 533:        // Special cases:
 534:        if(x == 1)
 535:        {  // There may actually be no answer to this question,
 536:           // since the probability of zero failures may be non-zero,
 537:           return 0; // but zero is the best we can do:
 538:        }
 539:        if (-x <= boost::math::powm1(dist.success_fraction(), dist.successes(), Policy()))
 540:        {  // q <= cdf(complement(dist, 0)) == pdf(dist, 0)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:           return 0; //
 542:        }
 543:        if(x == 0)
 544:        {  // Probability 1 - Q  == 1 so infinite failures to achieve certainty.
 545:           // Would need +infinity failures for total confidence.
 546:           result = policies::raise_overflow_error<RealType>(
 547:              function,
 548:              "Probability argument complement is 0, which implies infinite failures !", Policy());
 549:           return result;
 550:           // usually means return +std::numeric_limits<RealType>::infinity();
 551:           // unless #define BOOST_MATH_THROW_ON_OVERFLOW_ERROR
 552:        }
 553:        // log(x) /log(1-success_fraction) -1; but use log1p in case success_fraction is small
 554:        result = log(x) / boost::math::log1p(-success_fraction, Policy()) - 1;
 555:       return result;
 556: 
 557:     } // quantile complement
 558: 
~~~
- **EN:** This range declares or defines callable logic such as Policy, infinity. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy, infinity。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 559-571 / 第 559-571 行
~~~cpp
 559:  } // namespace math
 560: } // namespace boost
 561: 
 562: // This include must be at the end, *after* the accessors
 563: // for this distribution have been defined, in order to
 564: // keep compilers that support two-phase lookup happy.
 565: #include <boost/math/distributions/detail/derived_accessors.hpp>
 566: 
 567: #if defined (BOOST_MSVC)
 568: # pragma warning(pop)
 569: #endif
 570: 
 571: #endif // BOOST_MATH_SPECIAL_GEOMETRIC_HPP
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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/beta.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/special_functions/fpclassify.hpp, boost/math/tools/roots.hpp, boost/math/distributions/detail/inv_discrete_quantile.hpp, boost/math/special_functions/log1p.hpp, boost/math/distributions/detail/derived_accessors.hpp`
- **Namespaces / 命名空间**: `boost, math, geometric_detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `Policy, ibeta_inva, ibetac_inva, success_fraction, pow, exp, std::pow, boost::math::log1p, ...`
