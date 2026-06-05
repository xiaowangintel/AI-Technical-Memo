# inverse_gaussian.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/inverse_gaussian.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the inverse gaussian distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 inverse gaussian 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  Copyright John Maddock 2010.
   2: //  Copyright Paul A. Bristow 2010.
   3: //  Copyright Matt Borland 2024.
   4: //  Use, modification and distribution are subject to the
   5: //  Boost Software License, Version 1.0. (See accompanying file
   6: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   7: 
   8: #ifndef BOOST_STATS_INVERSE_GAUSSIAN_HPP
   9: #define BOOST_STATS_INVERSE_GAUSSIAN_HPP
  10: 
  11: #ifdef _MSC_VER
  12: #pragma warning(disable: 4512) // assignment operator could not be generated
  13: #endif
  14: 
  15: // http://en.wikipedia.org/wiki/Normal-inverse_Gaussian_distribution
  16: // http://mathworld.wolfram.com/InverseGaussianDistribution.html
  17: 
  18: // The normal-inverse Gaussian distribution
~~~
- **EN:** It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: // also called the Wald distribution (some sources limit this to when mean = 1).
  20: 
  21: // It is the continuous probability distribution
  22: // that is defined as the normal variance-mean mixture where the mixing density is the
  23: // inverse Gaussian distribution. The tails of the distribution decrease more slowly
  24: // than the normal distribution. It is therefore suitable to model phenomena
  25: // where numerically large values are more probable than is the case for the normal distribution.
  26: 
  27: // The Inverse Gaussian distribution was first studied in relationship to Brownian motion.
  28: // In 1956 M.C.K. Tweedie used the name 'Inverse Gaussian' because there is an inverse
  29: // relationship between the time to cover a unit distance and distance covered in unit time.
  30: 
  31: // Examples are returns from financial assets and turbulent wind speeds.
  32: // The normal-inverse Gaussian distributions form
  33: // a subclass of the generalised hyperbolic distributions.
  34: 
  35: // See also
  36: 
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms. The logic belongs to a numerical-integration path, so weights, nodes, or convergence rules are central.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。 该逻辑属于数值积分路径，因此权重、节点或收敛规则是核心内容。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: // http://en.wikipedia.org/wiki/Normal_distribution
  38: // http://www.itl.nist.gov/div898/handbook/eda/section3/eda3661.htm
  39: // Also:
  40: // Weisstein, Eric W. "Normal Distribution."
  41: // From MathWorld--A Wolfram Web Resource.
  42: // http://mathworld.wolfram.com/NormalDistribution.html
  43: 
  44: // http://www.jstatsoft.org/v26/i04/paper General class of inverse Gaussian distributions.
  45: // ig package - withdrawn but at http://cran.r-project.org/src/contrib/Archive/ig/
  46: 
  47: // http://www.stat.ucl.ac.be/ISdidactique/Rhelp/library/SuppDists/html/inverse_gaussian.html
  48: // R package for dinverse_gaussian, ...
  49: 
  50: // http://www.statsci.org/s/inverse_gaussian.s  and http://www.statsci.org/s/inverse_gaussian.html
  51: 
  52: #include <boost/math/tools/config.hpp>
  53: #include <boost/math/tools/cstdint.hpp>
  54: #include <boost/math/special_functions/erf.hpp> // for erf/erfc.
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/special_functions/erf.hpp so the surrounding code can use external declarations. It introduces the class `of` as part of the file's main abstraction. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/special_functions/erf.hpp 等依赖，使周围代码可以使用外部声明。 它引入了 class `of`，作为该文件核心抽象的一部分。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: #include <boost/math/distributions/complement.hpp>
  56: #include <boost/math/distributions/detail/common_error_handling.hpp>
  57: #include <boost/math/distributions/normal.hpp>
  58: #include <boost/math/distributions/gamma.hpp> // for gamma function
  59: #include <boost/math/tools/tuple.hpp>
  60: #include <boost/math/tools/roots.hpp>
  61: #include <boost/math/policies/policy.hpp>
  62: #include <boost/math/policies/error_handling.hpp>
  63: 
  64: namespace boost{ namespace math{
  65: 
  66: template <class RealType = double, class Policy = policies::policy<> >
  67: class inverse_gaussian_distribution
  68: {
  69: public:
  70:    using value_type = RealType;
  71:    using policy_type = Policy;
  72: 
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/normal.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/normal.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:    BOOST_MATH_GPU_ENABLED explicit inverse_gaussian_distribution(RealType l_mean = 1, RealType l_scale = 1)
  74:       : m_mean(l_mean), m_scale(l_scale)
  75:    { // Default is a 1,1 inverse_gaussian distribution.
  76:      constexpr auto function = "boost::math::inverse_gaussian_distribution<%1%>::inverse_gaussian_distribution";
  77: 
  78:      RealType result;
  79:      detail::check_scale(function, l_scale, &result, Policy());
  80:      detail::check_location(function, l_mean, &result, Policy());
  81:      detail::check_x_gt0(function, l_mean, &result, Policy());
  82:    }
  83: 
  84:    BOOST_MATH_GPU_ENABLED RealType mean()const
  85:    { // alias for location.
  86:       return m_mean; // aka mu
  87:    }
  88: 
  89:    // Synonyms, provided to allow generic use of find_location and find_scale.
  90:    BOOST_MATH_GPU_ENABLED RealType location()const
~~~
- **EN:** This range declares or defines callable logic such as detail::check_scale, detail::check_location, .... Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::check_scale, detail::check_location, ...。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:    { // location, aka mu.
  92:       return m_mean;
  93:    }
  94:    BOOST_MATH_GPU_ENABLED RealType scale()const
  95:    { // scale, aka lambda.
  96:       return m_scale;
  97:    }
  98: 
  99:    BOOST_MATH_GPU_ENABLED RealType shape()const
 100:    { // shape, aka phi = lambda/mu.
 101:       return m_scale / m_mean;
 102:    }
 103: 
 104: private:
 105:    //
 106:    // Data members:
 107:    //
 108:    RealType m_mean;  // distribution mean or location, aka mu.
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:    RealType m_scale;    // distribution standard deviation or scale, aka lambda.
 110: }; // class normal_distribution
 111: 
 112: using inverse_gaussian = inverse_gaussian_distribution<double>;
 113: 
 114: #ifdef __cpp_deduction_guides
 115: template <class RealType>
 116: inverse_gaussian_distribution(RealType)->inverse_gaussian_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 117: template <class RealType>
 118: inverse_gaussian_distribution(RealType,RealType)->inverse_gaussian_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 119: #endif
 120: 
 121: template <class RealType, class Policy>
 122: BOOST_MATH_GPU_ENABLED inline boost::math::pair<RealType, RealType> range(const inverse_gaussian_distribution<RealType, Policy>& /*dist*/)
 123: { // Range of permissible values for random variable x, zero to max.
 124:    using boost::math::tools::max_value;
 125:    return boost::math::pair<RealType, RealType>(static_cast<RealType>(0.), max_value<RealType>()); // - to + max value.
 126: }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `normal_distribution` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `normal_distribution`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127: 
 128: template <class RealType, class Policy>
 129: BOOST_MATH_GPU_ENABLED inline boost::math::pair<RealType, RealType> support(const inverse_gaussian_distribution<RealType, Policy>& /*dist*/)
 130: { // Range of supported values for random variable x, zero to max.
 131:   // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 132:    using boost::math::tools::max_value;
 133:    return boost::math::pair<RealType, RealType>(static_cast<RealType>(0.),  max_value<RealType>()); // - to + max value.
 134: }
 135: 
 136: template <class RealType, class Policy>
 137: BOOST_MATH_GPU_ENABLED inline RealType pdf(const inverse_gaussian_distribution<RealType, Policy>& dist, const RealType& x)
 138: { // Probability Density Function
 139:    BOOST_MATH_STD_USING  // for ADL of std functions
 140: 
 141:    RealType scale = dist.scale();
 142:    RealType mean = dist.mean();
 143:    RealType result = 0;
 144:    constexpr auto function = "boost::math::pdf(const inverse_gaussian_distribution<%1%>&, %1%)";
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, mean.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, mean。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:    if(false == detail::check_scale(function, scale, &result, Policy()))
 146:    {
 147:       return result;
 148:    }
 149:    if(false == detail::check_location(function, mean, &result, Policy()))
 150:    {
 151:       return result;
 152:    }
 153:    if(false == detail::check_x_gt0(function, mean, &result, Policy()))
 154:    {
 155:       return result;
 156:    }
 157:    if(false == detail::check_positive_x(function, x, &result, Policy()))
 158:    {
 159:       return result;
 160:    }
 161: 
 162:    if (x == 0)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:    {
 164:      return 0; // Convenient, even if not defined mathematically.
 165:    }
 166: 
 167:    result =
 168:      sqrt(scale / (constants::two_pi<RealType>() * x * x * x))
 169:     * exp(-scale * (x - mean) * (x - mean) / (2 * x * mean * mean));
 170:    return result;
 171: } // pdf
 172: 
 173: template <class RealType, class Policy>
 174: BOOST_MATH_GPU_ENABLED inline RealType logpdf(const inverse_gaussian_distribution<RealType, Policy>& dist, const RealType& x)
 175: { // Probability Density Function
 176:    BOOST_MATH_STD_USING  // for ADL of std functions
 177: 
 178:    RealType scale = dist.scale();
 179:    RealType mean = dist.mean();
 180:    RealType result = -boost::math::numeric_limits<RealType>::infinity();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as exp, scale, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 exp, scale, ...。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:    constexpr auto function = "boost::math::logpdf(const inverse_gaussian_distribution<%1%>&, %1%)";
 182:    if(false == detail::check_scale(function, scale, &result, Policy()))
 183:    {
 184:       return result;
 185:    }
 186:    if(false == detail::check_location(function, mean, &result, Policy()))
 187:    {
 188:       return result;
 189:    }
 190:    if(false == detail::check_x_gt0(function, mean, &result, Policy()))
 191:    {
 192:       return result;
 193:    }
 194:    if(false == detail::check_positive_x(function, x, &result, Policy()))
 195:    {
 196:       return result;
 197:    }
 198: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:    if (x == 0)
 200:    {
 201:      return boost::math::numeric_limits<RealType>::quiet_NaN(); // Convenient, even if not defined mathematically. log(0)
 202:    }
 203: 
 204:    const RealType two_pi = boost::math::constants::two_pi<RealType>();
 205: 
 206:    result = (-scale*pow(mean - x, RealType(2))/(mean*mean*x) + log(scale) - 3*log(x) - log(two_pi)) / 2;
 207:    return result;
 208: } // pdf
 209: 
 210: template <class RealType, class Policy>
 211: BOOST_MATH_GPU_ENABLED inline RealType cdf(const inverse_gaussian_distribution<RealType, Policy>& dist, const RealType& x)
 212: { // Cumulative Density Function.
 213:    BOOST_MATH_STD_USING  // for ADL of std functions.
 214: 
 215:    RealType scale = dist.scale();
 216:    RealType mean = dist.mean();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, mean.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, mean。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:    constexpr auto function = "boost::math::cdf(const inverse_gaussian_distribution<%1%>&, %1%)";
 218:    RealType result = 0;
 219:    if(false == detail::check_scale(function, scale, &result, Policy()))
 220:    {
 221:       return result;
 222:    }
 223:    if(false == detail::check_location(function, mean, &result, Policy()))
 224:    {
 225:       return result;
 226:    }
 227:    if (false == detail::check_x_gt0(function, mean, &result, Policy()))
 228:    {
 229:       return result;
 230:    }
 231:    if(false == detail::check_positive_x(function, x, &result, Policy()))
 232:    {
 233:      return result;
 234:    }
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:    if (x == 0)
 236:    {
 237:      return 0; // Convenient, even if not defined mathematically.
 238:    }
 239:    // Problem with this formula for large scale > 1000 or small x
 240:    // so use normal distribution version:
 241:    // Wikipedia CDF equation http://en.wikipedia.org/wiki/Inverse_Gaussian_distribution.
 242: 
 243:    normal_distribution<RealType> n01;
 244: 
 245:    RealType n0 = sqrt(scale / x);
 246:    n0 *= ((x / mean) -1);
 247:    RealType n1 = cdf(n01, n0);
 248:    RealType expfactor = exp(2 * scale / mean);
 249:    RealType n3 = - sqrt(scale / x);
 250:    n3 *= (x / mean) + 1;
 251:    RealType n4 = cdf(n01, n3);
 252:    result = n1 + expfactor * n4;
~~~
- **EN:** This range declares or defines callable logic such as sqrt, cdf, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, cdf, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:    return result;
 254: } // cdf
 255: 
 256: template <class RealType, class Policy>
 257: struct inverse_gaussian_quantile_functor
 258: {
 259: 
 260:   BOOST_MATH_GPU_ENABLED inverse_gaussian_quantile_functor(const boost::math::inverse_gaussian_distribution<RealType, Policy> dist, RealType const& p)
 261:     : distribution(dist), prob(p)
 262:   {
 263:   }
 264:   BOOST_MATH_GPU_ENABLED boost::math::tuple<RealType, RealType> operator()(RealType const& x)
 265:   {
 266:     RealType c = cdf(distribution, x);
 267:     RealType fx = c - prob;  // Difference cdf - value - to minimize.
 268:     RealType dx = pdf(distribution, x); // pdf is 1st derivative.
 269:     // return both function evaluation difference f(x) and 1st derivative f'(x).
 270:     return boost::math::make_tuple(fx, dx);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as cdf, pdf.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 cdf, pdf。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:   }
 272:   private:
 273:   const boost::math::inverse_gaussian_distribution<RealType, Policy> distribution;
 274:   RealType prob;
 275: };
 276: 
 277: template <class RealType, class Policy>
 278: struct inverse_gaussian_quantile_complement_functor
 279: {
 280:   BOOST_MATH_GPU_ENABLED inverse_gaussian_quantile_complement_functor(const boost::math::inverse_gaussian_distribution<RealType, Policy> dist, RealType const& p)
 281:     : distribution(dist), prob(p)
 282:   {
 283:   }
 284:   BOOST_MATH_GPU_ENABLED boost::math::tuple<RealType, RealType> operator()(RealType const& x)
 285:   {
 286:     RealType c = cdf(complement(distribution, x));
 287:     RealType fx = c - prob;  // Difference cdf - value - to minimize.
 288:     RealType dx = -pdf(distribution, x); // pdf is 1st derivative.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as cdf, pdf.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 cdf, pdf。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:     // return both function evaluation difference f(x) and 1st derivative f'(x).
 290:     //return std::tr1::make_tuple(fx, dx); if available.
 291:     return boost::math::make_tuple(fx, dx);
 292:   }
 293:   private:
 294:   const boost::math::inverse_gaussian_distribution<RealType, Policy> distribution;
 295:   RealType prob;
 296: };
 297: 
 298: namespace detail
 299: {
 300:   template <class RealType>
 301:   BOOST_MATH_GPU_ENABLED inline RealType guess_ig(RealType p, RealType mu = 1, RealType lambda = 1)
 302:   { // guess at random variate value x for inverse gaussian quantile.
 303:     BOOST_MATH_STD_USING
 304:     using boost::math::policies::policy;
 305:     // Error type.
 306:     using boost::math::policies::overflow_error;
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:     // Action.
 308:     using boost::math::policies::ignore_error;
 309: 
 310:     using no_overthrow_policy = policy<overflow_error<ignore_error>>;
 311: 
 312:     RealType x; // result is guess at random variate value x.
 313:     RealType phi = lambda / mu;
 314:     if (phi > 2.)
 315:     { // Big phi, so starting to look like normal Gaussian distribution.
 316:       //
 317:       // Whitmore, G.A. and Yalovsky, M.
 318:       // A normalising logarithmic transformation for inverse Gaussian random variables,
 319:       // Technometrics 20-2, 207-208 (1978), but using expression from
 320:       // V Seshadri, Inverse Gaussian distribution (1998) ISBN 0387 98618 9, page 6.
 321: 
 322:       normal_distribution<RealType, no_overthrow_policy> n01;
 323:       x = mu * exp(quantile(n01, p) / sqrt(phi) - 1/(2 * phi));
 324:      }
~~~
- **EN:** This range declares or defines callable logic such as exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:     else
 326:     { // phi < 2 so much less symmetrical with long tail,
 327:       // so use gamma distribution as an approximation.
 328:       using boost::math::gamma_distribution;
 329: 
 330:       // Define the distribution, using gamma_nooverflow:
 331:       using gamma_nooverflow = gamma_distribution<RealType, no_overthrow_policy>;
 332: 
 333:       gamma_nooverflow g(static_cast<RealType>(0.5), static_cast<RealType>(1.));
 334: 
 335:       // R qgamma(0.2, 0.5, 1) = 0.0320923
 336:       RealType qg = quantile(complement(g, p));
 337:       x = lambda / (qg * 2);
 338:       //
 339:       if (x > mu/2) // x > mu /2?
 340:       { // x too large for the gamma approximation to work well.
 341:         //x = qgamma(p, 0.5, 1.0); // qgamma(0.270614, 0.5, 1) = 0.05983807
 342:         RealType q = quantile(g, p);
~~~
- **EN:** This range declares or defines callable logic such as g, quantile, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围声明或定义了可调用逻辑，例如 g, quantile, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:        // x = mu * exp(q * static_cast<RealType>(0.1));  // Said to improve at high p
 344:        // x = mu * x;  // Improves at high p?
 345:         x = mu * exp(q / sqrt(phi) - 1/(2 * phi));
 346:       }
 347:     }
 348:     return x;
 349:   }  // guess_ig
 350: } // namespace detail
 351: 
 352: template <class RealType, class Policy>
 353: BOOST_MATH_GPU_ENABLED inline RealType quantile(const inverse_gaussian_distribution<RealType, Policy>& dist, const RealType& p)
 354: {
 355:    BOOST_MATH_STD_USING  // for ADL of std functions.
 356:    // No closed form exists so guess and use Newton Raphson iteration.
 357: 
 358:    RealType mean = dist.mean();
 359:    RealType scale = dist.scale();
 360:    constexpr auto function = "boost::math::quantile(const inverse_gaussian_distribution<%1%>&, %1%)";
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361: 
 362:    RealType result = 0;
 363:    if(false == detail::check_scale(function, scale, &result, Policy()))
 364:       return result;
 365:    if(false == detail::check_location(function, mean, &result, Policy()))
 366:       return result;
 367:    if (false == detail::check_x_gt0(function, mean, &result, Policy()))
 368:       return result;
 369:    if(false == detail::check_probability(function, p, &result, Policy()))
 370:       return result;
 371:    if (p == 0)
 372:    {
 373:      return 0; // Convenient, even if not defined mathematically?
 374:    }
 375:    if (p == 1)
 376:    { // overflow
 377:       result = policies::raise_overflow_error<RealType>(function,
 378:         "probability parameter is 1, but must be < 1!", Policy());
~~~
- **EN:** This range declares or defines callable logic such as Policy. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:       return result; // infinity;
 380:    }
 381: 
 382:   RealType guess = detail::guess_ig(p, dist.mean(), dist.scale());
 383:   using boost::math::tools::max_value;
 384: 
 385:   RealType min = static_cast<RealType>(0); // Minimum possible value is bottom of range of distribution.
 386:   RealType max = max_value<RealType>();// Maximum possible value is top of range.
 387:   // int digits = std::numeric_limits<RealType>::digits; // Maximum possible binary digits accuracy for type T.
 388:   // digits used to control how accurate to try to make the result.
 389:   // To allow user to control accuracy versus speed,
 390:   int get_digits = policies::digits<RealType, Policy>();// get digits from policy,
 391:   boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>(); // and max iterations.
 392:   using boost::math::tools::newton_raphson_iterate;
 393:   result =
 394:     newton_raphson_iterate(inverse_gaussian_quantile_functor<RealType, Policy>(dist, p), guess, min, max, get_digits, max_iter);
 395:   if (max_iter >= policies::get_max_root_iterations<Policy>())
 396:   {
~~~
- **EN:** This range declares or defines callable logic such as detail::guess_ig, newton_raphson_iterate. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::guess_ig, newton_raphson_iterate。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:      return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 398:         " either there is no answer to quantile or the answer is infinite.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
 399:   }
 400:   return result;
 401: } // quantile
 402: 
 403: template <class RealType, class Policy>
 404: BOOST_MATH_GPU_ENABLED inline RealType cdf(const complemented2_type<inverse_gaussian_distribution<RealType, Policy>, RealType>& c)
 405: {
 406:    BOOST_MATH_STD_USING  // for ADL of std functions.
 407: 
 408:    RealType scale = c.dist.scale();
 409:    RealType mean = c.dist.mean();
 410:    RealType x = c.param;
 411:    constexpr auto function = "boost::math::cdf(const complement(inverse_gaussian_distribution<%1%>&), %1%)";
 412: 
 413:    RealType result = 0;
 414:    if(false == detail::check_scale(function, scale, &result, Policy()))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, scale, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, scale, ...。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:       return result;
 416:    if(false == detail::check_location(function, mean, &result, Policy()))
 417:       return result;
 418:    if (false == detail::check_x_gt0(function, mean, &result, Policy()))
 419:       return result;
 420:    if(false == detail::check_positive_x(function, x, &result, Policy()))
 421:       return result;
 422: 
 423:    normal_distribution<RealType> n01;
 424:    RealType n0 = sqrt(scale / x);
 425:    n0 *= ((x / mean) -1);
 426:    RealType cdf_1 = cdf(complement(n01, n0));
 427: 
 428:    RealType expfactor = exp(2 * scale / mean);
 429:    RealType n3 = - sqrt(scale / x);
 430:    n3 *= (x / mean) + 1;
 431: 
 432:    //RealType n5 = +sqrt(scale/x) * ((x /mean) + 1); // note now positive sign.
~~~
- **EN:** This range declares or defines callable logic such as sqrt, cdf, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, cdf, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:    RealType n6 = cdf(complement(n01, +sqrt(scale/x) * ((x /mean) + 1)));
 434:    // RealType n4 = cdf(n01, n3); // =
 435:    result = cdf_1 - expfactor * n6;
 436:    return result;
 437: } // cdf complement
 438: 
 439: template <class RealType, class Policy>
 440: BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<inverse_gaussian_distribution<RealType, Policy>, RealType>& c)
 441: {
 442:    BOOST_MATH_STD_USING  // for ADL of std functions
 443: 
 444:    RealType scale = c.dist.scale();
 445:    RealType mean = c.dist.mean();
 446:    constexpr auto function = "boost::math::quantile(const complement(inverse_gaussian_distribution<%1%>&), %1%)";
 447:    RealType result = 0;
 448:    if(false == detail::check_scale(function, scale, &result, Policy()))
 449:       return result;
 450:    if(false == detail::check_location(function, mean, &result, Policy()))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as cdf, scale, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 cdf, scale, ...。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:       return result;
 452:    if (false == detail::check_x_gt0(function, mean, &result, Policy()))
 453:       return result;
 454:    RealType q = c.param;
 455:    if(false == detail::check_probability(function, q, &result, Policy()))
 456:       return result;
 457: 
 458:    RealType guess = detail::guess_ig(q, mean, scale);
 459:    // Complement.
 460:    using boost::math::tools::max_value;
 461: 
 462:   RealType min = static_cast<RealType>(0); // Minimum possible value is bottom of range of distribution.
 463:   RealType max = max_value<RealType>();// Maximum possible value is top of range.
 464:   // int digits = std::numeric_limits<RealType>::digits; // Maximum possible binary digits accuracy for type T.
 465:   // digits used to control how accurate to try to make the result.
 466:   int get_digits = policies::digits<RealType, Policy>();
 467:   boost::math::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 468:   using boost::math::tools::newton_raphson_iterate;
~~~
- **EN:** This range declares or defines callable logic such as detail::guess_ig. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::guess_ig。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:   result = newton_raphson_iterate(inverse_gaussian_quantile_complement_functor<RealType, Policy>(c.dist, q), guess, min, max, get_digits, max_iter);
 470:   if (max_iter >= policies::get_max_root_iterations<Policy>())
 471:   {
 472:      return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 473:         " either there is no answer to quantile or the answer is infinite.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
 474:   }
 475:   return result;
 476: } // quantile
 477: 
 478: template <class RealType, class Policy>
 479: BOOST_MATH_GPU_ENABLED inline RealType mean(const inverse_gaussian_distribution<RealType, Policy>& dist)
 480: { // aka mu
 481:    return dist.mean();
 482: }
 483: 
 484: template <class RealType, class Policy>
 485: BOOST_MATH_GPU_ENABLED inline RealType scale(const inverse_gaussian_distribution<RealType, Policy>& dist)
 486: { // aka lambda
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as newton_raphson_iterate, Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 newton_raphson_iterate, Policy。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:    return dist.scale();
 488: }
 489: 
 490: template <class RealType, class Policy>
 491: BOOST_MATH_GPU_ENABLED inline RealType shape(const inverse_gaussian_distribution<RealType, Policy>& dist)
 492: { // aka phi
 493:    return dist.shape();
 494: }
 495: 
 496: template <class RealType, class Policy>
 497: BOOST_MATH_GPU_ENABLED inline RealType standard_deviation(const inverse_gaussian_distribution<RealType, Policy>& dist)
 498: {
 499:   BOOST_MATH_STD_USING
 500:   RealType scale = dist.scale();
 501:   RealType mean = dist.mean();
 502:   RealType result = sqrt(mean * mean * mean / scale);
 503:   return result;
 504: }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, mean, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, mean, ...。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505: 
 506: template <class RealType, class Policy>
 507: BOOST_MATH_GPU_ENABLED inline RealType mode(const inverse_gaussian_distribution<RealType, Policy>& dist)
 508: {
 509:   BOOST_MATH_STD_USING
 510:   RealType scale = dist.scale();
 511:   RealType  mean = dist.mean();
 512:   RealType result = mean * (sqrt(1 + (9 * mean * mean)/(4 * scale * scale))
 513:       - 3 * mean / (2 * scale));
 514:   return result;
 515: }
 516: 
 517: template <class RealType, class Policy>
 518: BOOST_MATH_GPU_ENABLED inline RealType skewness(const inverse_gaussian_distribution<RealType, Policy>& dist)
 519: {
 520:   BOOST_MATH_STD_USING
 521:   RealType scale = dist.scale();
 522:   RealType  mean = dist.mean();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, mean.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, mean。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:   RealType result = 3 * sqrt(mean/scale);
 524:   return result;
 525: }
 526: 
 527: template <class RealType, class Policy>
 528: BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const inverse_gaussian_distribution<RealType, Policy>& dist)
 529: {
 530:   RealType scale = dist.scale();
 531:   RealType  mean = dist.mean();
 532:   RealType result = 15 * mean / scale -3;
 533:   return result;
 534: }
 535: 
 536: template <class RealType, class Policy>
 537: BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const inverse_gaussian_distribution<RealType, Policy>& dist)
 538: {
 539:   RealType scale = dist.scale();
 540:   RealType  mean = dist.mean();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as sqrt, scale, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 sqrt, scale, ...。

### Lines 541-555 / 第 541-555 行
~~~cpp
 541:   RealType result = 15 * mean / scale;
 542:   return result;
 543: }
 544: 
 545: } // namespace math
 546: } // namespace boost
 547: 
 548: // This include must be at the end, *after* the accessors
 549: // for this distribution have been defined, in order to
 550: // keep compilers that support two-phase lookup happy.
 551: #include <boost/math/distributions/detail/derived_accessors.hpp>
 552: 
 553: #endif // BOOST_STATS_INVERSE_GAUSSIAN_HPP
 554: 
 555: 
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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/cstdint.hpp, boost/math/special_functions/erf.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/normal.hpp, boost/math/distributions/gamma.hpp, boost/math/tools/tuple.hpp, boost/math/tools/roots.hpp, boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, boost/math/distributions/detail/derived_accessors.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `detail::check_scale, detail::check_location, detail::check_x_gt0, scale, mean, exp, infinity, sqrt, ...`
