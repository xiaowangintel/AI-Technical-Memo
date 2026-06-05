# inverse_gamma.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/inverse_gamma.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the inverse gamma distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 inverse gamma 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // inverse_gamma.hpp
   2: 
   3: //  Copyright Paul A. Bristow 2010.
   4: //  Copyright John Maddock 2010.
   5: //  Copyright Matt Borland 2024.
   6: //  Use, modification and distribution are subject to the
   7: //  Boost Software License, Version 1.0. (See accompanying file
   8: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   9: 
  10: #ifndef BOOST_STATS_INVERSE_GAMMA_HPP
  11: #define BOOST_STATS_INVERSE_GAMMA_HPP
  12: 
  13: // Inverse Gamma Distribution is a two-parameter family
  14: // of continuous probability distributions
  15: // on the positive real line, which is the distribution of
  16: // the reciprocal of a variable distributed according to the gamma distribution.
  17: 
  18: // http://en.wikipedia.org/wiki/Inverse-gamma_distribution
~~~
- **EN:** It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: // http://rss.acs.unt.edu/Rdoc/library/pscl/html/igamma.html
  20: 
  21: // See also gamma distribution at gamma.hpp:
  22: // http://www.itl.nist.gov/div898/handbook/eda/section3/eda366b.htm
  23: // http://mathworld.wolfram.com/GammaDistribution.html
  24: // http://en.wikipedia.org/wiki/Gamma_distribution
  25: 
  26: #include <boost/math/tools/config.hpp>
  27: #include <boost/math/tools/tuple.hpp>
  28: #include <boost/math/tools/numeric_limits.hpp>
  29: #include <boost/math/distributions/fwd.hpp>
  30: #include <boost/math/special_functions/gamma.hpp>
  31: #include <boost/math/distributions/detail/common_error_handling.hpp>
  32: #include <boost/math/distributions/complement.hpp>
  33: 
  34: namespace boost{ namespace math
  35: {
  36: namespace detail
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: {
  38: 
  39: template <class RealType, class Policy>
  40: BOOST_MATH_GPU_ENABLED inline bool check_inverse_gamma_shape(
  41:       const char* function, // inverse_gamma
  42:       RealType shape, // shape aka alpha
  43:       RealType* result, // to update, perhaps with NaN
  44:       const Policy& pol)
  45: {  // Sources say shape argument must be > 0
  46:    // but seems logical to allow shape zero as special case,
  47:    // returning pdf and cdf zero (but not < 0).
  48:    // (Functions like mean, variance with other limits on shape are checked
  49:    // in version including an operator & limit below).
  50:    if((shape < 0) || !(boost::math::isfinite)(shape))
  51:    {
  52:       *result = policies::raise_domain_error<RealType>(
  53:          function,
  54:          "Shape parameter is %1%, but must be >= 0 !", shape, pol);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:       return false;
  56:    }
  57:    return true;
  58: } //bool check_inverse_gamma_shape
  59: 
  60: template <class RealType, class Policy>
  61: BOOST_MATH_GPU_ENABLED inline bool check_inverse_gamma_x(
  62:       const char* function,
  63:       RealType const& x,
  64:       RealType* result, const Policy& pol)
  65: {
  66:    if((x < 0) || !(boost::math::isfinite)(x))
  67:    {
  68:       *result = policies::raise_domain_error<RealType>(
  69:          function,
  70:          "Random variate is %1% but must be >= 0 !", x, pol);
  71:       return false;
  72:    }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:    return true;
  74: }
  75: 
  76: template <class RealType, class Policy>
  77: BOOST_MATH_GPU_ENABLED inline bool check_inverse_gamma(
  78:       const char* function, // TODO swap these over, so shape is first.
  79:       RealType scale,  // scale aka beta
  80:       RealType shape, // shape aka alpha
  81:       RealType* result, const Policy& pol)
  82: {
  83:    return check_scale(function, scale, result, pol)
  84:      && check_inverse_gamma_shape(function, shape, result, pol);
  85: } // bool check_inverse_gamma
  86: 
  87: } // namespace detail
  88: 
  89: template <class RealType = double, class Policy = policies::policy<> >
  90: class inverse_gamma_distribution
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91: {
  92: public:
  93:    using value_type = RealType;
  94:    using policy_type = Policy;
  95: 
  96:    BOOST_MATH_GPU_ENABLED explicit inverse_gamma_distribution(RealType l_shape = 1, RealType l_scale = 1)
  97:       : m_shape(l_shape), m_scale(l_scale)
  98:    {
  99:       RealType result;
 100:       detail::check_inverse_gamma(
 101:         "boost::math::inverse_gamma_distribution<%1%>::inverse_gamma_distribution",
 102:         l_scale, l_shape, &result, Policy());
 103:    }
 104: 
 105:    BOOST_MATH_GPU_ENABLED RealType shape()const
 106:    {
 107:       return m_shape;
 108:    }
~~~
- **EN:** This range declares or defines callable logic such as Policy. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109: 
 110:    BOOST_MATH_GPU_ENABLED RealType scale()const
 111:    {
 112:       return m_scale;
 113:    }
 114: private:
 115:    //
 116:    // Data members:
 117:    //
 118:    RealType m_shape;     // distribution shape
 119:    RealType m_scale;     // distribution scale
 120: };
 121: 
 122: using inverse_gamma = inverse_gamma_distribution<double>;
 123: // typedef - but potential clash with name of inverse gamma *function*.
 124: // but there is a typedef for the gamma distribution (gamma)
 125: 
 126: #ifdef __cpp_deduction_guides
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127: template <class RealType>
 128: inverse_gamma_distribution(RealType)->inverse_gamma_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 129: template <class RealType>
 130: inverse_gamma_distribution(RealType,RealType)->inverse_gamma_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 131: #endif
 132: 
 133: // Allow random variable x to be zero, treated as a special case (unlike some definitions).
 134: 
 135: template <class RealType, class Policy>
 136: BOOST_MATH_GPU_ENABLED inline boost::math::pair<RealType, RealType> range(const inverse_gamma_distribution<RealType, Policy>& /* dist */)
 137: {  // Range of permissible values for random variable x.
 138:    using boost::math::tools::max_value;
 139:    return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>());
 140: }
 141: 
 142: template <class RealType, class Policy>
 143: BOOST_MATH_GPU_ENABLED inline boost::math::pair<RealType, RealType> support(const inverse_gamma_distribution<RealType, Policy>& /* dist */)
 144: {  // Range of supported values for random variable x.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:    // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 146:    using boost::math::tools::max_value;
 147:    using boost::math::tools::min_value;
 148:    return boost::math::pair<RealType, RealType>(static_cast<RealType>(0),  max_value<RealType>());
 149: }
 150: 
 151: template <class RealType, class Policy>
 152: BOOST_MATH_GPU_ENABLED inline RealType pdf(const inverse_gamma_distribution<RealType, Policy>& dist, const RealType& x)
 153: {
 154:    BOOST_MATH_STD_USING  // for ADL of std functions
 155: 
 156:    constexpr auto function = "boost::math::pdf(const inverse_gamma_distribution<%1%>&, %1%)";
 157: 
 158:    RealType shape = dist.shape();
 159:    RealType scale = dist.scale();
 160: 
 161:    RealType result = 0;
 162:    if(false == detail::check_inverse_gamma(function, scale, shape, &result, Policy()))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as shape, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 shape, scale。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:    { // distribution parameters bad.
 164:       return result;
 165:    }
 166:    if(x == 0)
 167:    { // Treat random variate zero as a special case.
 168:       return 0;
 169:    }
 170:    else if(false == detail::check_inverse_gamma_x(function, x, &result, Policy()))
 171:    { // x bad.
 172:       return result;
 173:    }
 174:    result = scale / x;
 175:    if(result < tools::min_value<RealType>())
 176:       return 0;  // random variable is infinite or so close as to make no difference.
 177:    result = gamma_p_derivative(shape, result, Policy()) * scale;
 178:    if(0 != result)
 179:    {
 180:       if(x < 0)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:       {
 182:          // x * x may under or overflow, likewise our result,
 183:          // so be extra careful about the arithmetic:
 184:          RealType lim = tools::max_value<RealType>() * x;
 185:          if(lim < result)
 186:             return policies::raise_overflow_error<RealType, Policy>(function, "PDF is infinite.", Policy());
 187:          result /= x;
 188:          if(lim < result)
 189:             return policies::raise_overflow_error<RealType, Policy>(function, "PDF is infinite.", Policy());
 190:          result /= x;
 191:       }
 192:       result /= (x * x);
 193:    }
 194: 
 195:    return result;
 196: } // pdf
 197: 
 198: template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199: BOOST_MATH_GPU_ENABLED inline RealType logpdf(const inverse_gamma_distribution<RealType, Policy>& dist, const RealType& x)
 200: {
 201:    BOOST_MATH_STD_USING  // for ADL of std functions
 202:    using boost::math::lgamma;
 203: 
 204:    constexpr auto function = "boost::math::logpdf(const inverse_gamma_distribution<%1%>&, %1%)";
 205: 
 206:    RealType shape = dist.shape();
 207:    RealType scale = dist.scale();
 208: 
 209:    RealType result = -boost::math::numeric_limits<RealType>::infinity();
 210:    if(false == detail::check_inverse_gamma(function, scale, shape, &result, Policy()))
 211:    { // distribution parameters bad.
 212:       return result;
 213:    }
 214:    if(x == 0)
 215:    { // Treat random variate zero as a special case.
 216:       return result;
~~~
- **EN:** This range declares or defines callable logic such as shape, scale, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 shape, scale, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:    }
 218:    else if(false == detail::check_inverse_gamma_x(function, x, &result, Policy()))
 219:    { // x bad.
 220:       return result;
 221:    }
 222:    result = scale / x;
 223:    if(result < tools::min_value<RealType>())
 224:       return result;  // random variable is infinite or so close as to make no difference.
 225: 
 226:    // x * x may under or overflow, likewise our result
 227:    if (!(boost::math::isfinite)(x*x))
 228:    {
 229:       return policies::raise_overflow_error<RealType, Policy>(function, "PDF is infinite.", Policy());
 230:    }
 231: 
 232:    return shape * log(scale) + (-shape-1)*log(x) - lgamma(shape) - (scale/x);
 233: } // pdf
 234: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235: template <class RealType, class Policy>
 236: BOOST_MATH_GPU_ENABLED inline RealType cdf(const inverse_gamma_distribution<RealType, Policy>& dist, const RealType& x)
 237: {
 238:    BOOST_MATH_STD_USING  // for ADL of std functions
 239: 
 240:    constexpr auto function = "boost::math::cdf(const inverse_gamma_distribution<%1%>&, %1%)";
 241: 
 242:    RealType shape = dist.shape();
 243:    RealType scale = dist.scale();
 244: 
 245:    RealType result = 0;
 246:    if(false == detail::check_inverse_gamma(function, scale, shape, &result, Policy()))
 247:    { // distribution parameters bad.
 248:       return result;
 249:    }
 250:    if (x == 0)
 251:    { // Treat zero as a special case.
 252:      return 0;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as shape, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 shape, scale。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:    }
 254:    else if(false == detail::check_inverse_gamma_x(function, x, &result, Policy()))
 255:    { // x bad
 256:       return result;
 257:    }
 258:    result = boost::math::gamma_q(shape, scale / x, Policy());
 259:    // result = tgamma(shape, scale / x) / tgamma(shape); // naive using tgamma
 260:    return result;
 261: } // cdf
 262: 
 263: template <class RealType, class Policy>
 264: BOOST_MATH_GPU_ENABLED inline RealType quantile(const inverse_gamma_distribution<RealType, Policy>& dist, const RealType& p)
 265: {
 266:    BOOST_MATH_STD_USING  // for ADL of std functions
 267:    using boost::math::gamma_q_inv;
 268: 
 269:    constexpr auto function = "boost::math::quantile(const inverse_gamma_distribution<%1%>&, %1%)";
 270: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::gamma_q, tgamma.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::gamma_q, tgamma。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:    RealType shape = dist.shape();
 272:    RealType scale = dist.scale();
 273: 
 274:    RealType result = 0;
 275:    if(false == detail::check_inverse_gamma(function, scale, shape, &result, Policy()))
 276:       return result;
 277:    if(false == detail::check_probability(function, p, &result, Policy()))
 278:       return result;
 279:    if(p == 1)
 280:    {
 281:       return policies::raise_overflow_error<RealType>(function, 0, Policy());
 282:    }
 283:    result = gamma_q_inv(shape, p, Policy());
 284:    if((result < 1) && (result * tools::max_value<RealType>() < scale))
 285:       return policies::raise_overflow_error<RealType, Policy>(function, "Value of random variable in inverse gamma distribution quantile is infinite.", Policy());
 286:    result = scale / result;
 287:    return result;
 288: }
~~~
- **EN:** This range declares or defines callable logic such as shape, scale, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 shape, scale, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289: 
 290: template <class RealType, class Policy>
 291: BOOST_MATH_GPU_ENABLED inline RealType cdf(const complemented2_type<inverse_gamma_distribution<RealType, Policy>, RealType>& c)
 292: {
 293:    BOOST_MATH_STD_USING  // for ADL of std functions
 294: 
 295:    constexpr auto function = "boost::math::quantile(const gamma_distribution<%1%>&, %1%)";
 296: 
 297:    RealType shape = c.dist.shape();
 298:    RealType scale = c.dist.scale();
 299: 
 300:    RealType result = 0;
 301:    if(false == detail::check_inverse_gamma(function, scale, shape, &result, Policy()))
 302:       return result;
 303:    if(false == detail::check_inverse_gamma_x(function, c.param, &result, Policy()))
 304:       return result;
 305: 
 306:    if(c.param == 0)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as shape, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 shape, scale。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:       return 1; // Avoid division by zero
 308: 
 309:    result = gamma_p(shape, scale/c.param, Policy());
 310:    return result;
 311: }
 312: 
 313: template <class RealType, class Policy>
 314: BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<inverse_gamma_distribution<RealType, Policy>, RealType>& c)
 315: {
 316:    BOOST_MATH_STD_USING  // for ADL of std functions
 317: 
 318:    constexpr auto function = "boost::math::quantile(const inverse_gamma_distribution<%1%>&, %1%)";
 319: 
 320:    RealType shape = c.dist.shape();
 321:    RealType scale = c.dist.scale();
 322:    RealType q = c.param;
 323: 
 324:    RealType result = 0;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as gamma_p, shape, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 gamma_p, shape, ...。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:    if(false == detail::check_inverse_gamma(function, scale, shape, &result, Policy()))
 326:       return result;
 327:    if(false == detail::check_probability(function, q, &result, Policy()))
 328:       return result;
 329: 
 330:    if(q == 0)
 331:    {
 332:       return policies::raise_overflow_error<RealType>(function, 0, Policy());
 333:    }
 334:    result = gamma_p_inv(shape, q, Policy());
 335:    if((result < 1) && (result * tools::max_value<RealType>() < scale))
 336:       return policies::raise_overflow_error<RealType, Policy>(function, "Value of random variable in inverse gamma distribution quantile is infinite.", Policy());
 337:    result = scale / result;
 338:    return result;
 339: }
 340: 
 341: template <class RealType, class Policy>
 342: BOOST_MATH_GPU_ENABLED inline RealType mean(const inverse_gamma_distribution<RealType, Policy>& dist)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as gamma_p_inv.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 gamma_p_inv。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343: {
 344:    BOOST_MATH_STD_USING  // for ADL of std functions
 345: 
 346:    constexpr auto function = "boost::math::mean(const inverse_gamma_distribution<%1%>&)";
 347: 
 348:    RealType shape = dist.shape();
 349:    RealType scale = dist.scale();
 350: 
 351:    RealType result = 0;
 352: 
 353:    if(false == detail::check_scale(function, scale, &result, Policy()))
 354:    {
 355:      return result;
 356:    }
 357:    if((shape <= 1) || !(boost::math::isfinite)(shape))
 358:    {
 359:      result = policies::raise_domain_error<RealType>(
 360:        function,
~~~
- **EN:** This range declares or defines callable logic such as shape, scale. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 shape, scale。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:        "Shape parameter is %1%, but for a defined mean it must be > 1", shape, Policy());
 362:      return result;
 363:    }
 364:   result = scale / (shape - 1);
 365:   return result;
 366: } // mean
 367: 
 368: template <class RealType, class Policy>
 369: BOOST_MATH_GPU_ENABLED inline RealType variance(const inverse_gamma_distribution<RealType, Policy>& dist)
 370: {
 371:    BOOST_MATH_STD_USING  // for ADL of std functions
 372: 
 373:    constexpr auto function = "boost::math::variance(const inverse_gamma_distribution<%1%>&)";
 374: 
 375:    RealType shape = dist.shape();
 376:    RealType scale = dist.scale();
 377: 
 378:    RealType result = 0;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, shape, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, shape, ...。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:       if(false == detail::check_scale(function, scale, &result, Policy()))
 380:    {
 381:      return result;
 382:    }
 383:    if((shape <= 2) || !(boost::math::isfinite)(shape))
 384:    {
 385:      result = policies::raise_domain_error<RealType>(
 386:        function,
 387:        "Shape parameter is %1%, but for a defined variance it must be > 2", shape, Policy());
 388:      return result;
 389:    }
 390:    result = (scale * scale) / ((shape - 1) * (shape -1) * (shape -2));
 391:    return result;
 392: }
 393: 
 394: template <class RealType, class Policy>
 395: BOOST_MATH_GPU_ENABLED inline RealType mode(const inverse_gamma_distribution<RealType, Policy>& dist)
 396: {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:    BOOST_MATH_STD_USING  // for ADL of std functions
 398: 
 399:    constexpr auto function = "boost::math::mode(const inverse_gamma_distribution<%1%>&)";
 400: 
 401:    RealType shape = dist.shape();
 402:    RealType scale = dist.scale();
 403: 
 404:    RealType result = 0;
 405:    if(false == detail::check_inverse_gamma(function, scale, shape, &result, Policy()))
 406:    {
 407:       return result;
 408:    }
 409:    // Only defined for shape >= 0, but is checked by check_inverse_gamma.
 410:    result = scale / (shape + 1);
 411:    return result;
 412: }
 413: 
 414: //template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as shape, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 shape, scale。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415: //inline RealType median(const gamma_distribution<RealType, Policy>& dist)
 416: //{  // Wikipedia does not define median,
 417:      // so rely on default definition quantile(0.5) in derived accessors.
 418: //  return result.
 419: //}
 420: 
 421: template <class RealType, class Policy>
 422: BOOST_MATH_GPU_ENABLED inline RealType skewness(const inverse_gamma_distribution<RealType, Policy>& dist)
 423: {
 424:    BOOST_MATH_STD_USING  // for ADL of std functions
 425: 
 426:    constexpr auto function = "boost::math::skewness(const inverse_gamma_distribution<%1%>&)";
 427: 
 428:    RealType shape = dist.shape();
 429:    RealType scale = dist.scale();
 430:    RealType result = 0;
 431: 
 432:    if(false == detail::check_scale(function, scale, &result, Policy()))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as shape, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 shape, scale。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:    {
 434:      return result;
 435:    }
 436:    if((shape <= 3) || !(boost::math::isfinite)(shape))
 437:    {
 438:      result = policies::raise_domain_error<RealType>(
 439:        function,
 440:        "Shape parameter is %1%, but for a defined skewness it must be > 3", shape, Policy());
 441:      return result;
 442:    }
 443:    result = (4 * sqrt(shape - 2) ) / (shape - 3);
 444:    return result;
 445: }
 446: 
 447: template <class RealType, class Policy>
 448: BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const inverse_gamma_distribution<RealType, Policy>& dist)
 449: {
 450:    BOOST_MATH_STD_USING  // for ADL of std functions
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, sqrt.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, sqrt。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451: 
 452:    constexpr auto function = "boost::math::kurtosis_excess(const inverse_gamma_distribution<%1%>&)";
 453: 
 454:    RealType shape = dist.shape();
 455:    RealType scale = dist.scale();
 456: 
 457:    RealType result = 0;
 458:    if(false == detail::check_scale(function, scale, &result, Policy()))
 459:    {
 460:      return result;
 461:    }
 462:    if((shape <= 4) || !(boost::math::isfinite)(shape))
 463:    {
 464:      result = policies::raise_domain_error<RealType>(
 465:        function,
 466:        "Shape parameter is %1%, but for a defined kurtosis excess it must be > 4", shape, Policy());
 467:      return result;
 468:    }
~~~
- **EN:** This range declares or defines callable logic such as shape, scale, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 shape, scale, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:    result = (30 * shape - 66) / ((shape - 3) * (shape - 4));
 470:    return result;
 471: }
 472: 
 473: template <class RealType, class Policy>
 474: BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const inverse_gamma_distribution<RealType, Policy>& dist)
 475: {
 476:   constexpr auto function = "boost::math::kurtosis(const inverse_gamma_distribution<%1%>&)";
 477:    RealType shape = dist.shape();
 478:    RealType scale = dist.scale();
 479: 
 480:    RealType result = 0;
 481: 
 482:   if(false == detail::check_scale(function, scale, &result, Policy()))
 483:    {
 484:      return result;
 485:    }
 486:    if((shape <= 4) || !(boost::math::isfinite)(shape))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as shape, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 shape, scale。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:    {
 488:      result = policies::raise_domain_error<RealType>(
 489:        function,
 490:        "Shape parameter is %1%, but for a defined kurtosis it must be > 4", shape, Policy());
 491:      return result;
 492:    }
 493:   return kurtosis_excess(dist) + 3;
 494: }
 495: 
 496: } // namespace math
 497: } // namespace boost
 498: 
 499: // This include must be at the end, *after* the accessors
 500: // for this distribution have been defined, in order to
 501: // keep compilers that support two-phase lookup happy.
 502: #include <boost/math/distributions/detail/derived_accessors.hpp>
 503: 
 504: #endif // BOOST_STATS_INVERSE_GAMMA_HPP
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp so the surrounding code can use external declarations. The code enters namespace scope (math::boost) to keep symbols organized. This range declares or defines callable logic such as Policy.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围声明或定义了可调用逻辑，例如 Policy。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/numeric_limits.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/gamma.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/derived_accessors.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `check_inverse_gamma_shape, Policy, shape, scale, infinity, boost::math::gamma_q, tgamma, gamma_q_inv, ...`
