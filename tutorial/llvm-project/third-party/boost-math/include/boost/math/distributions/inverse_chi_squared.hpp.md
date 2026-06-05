# inverse_chi_squared.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/inverse_chi_squared.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the inverse chi squared distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 inverse chi squared 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // Copyright John Maddock 2010.
   2: // Copyright Paul A. Bristow 2010.
   3: // Copyright Matt Borland 2024.
   4: // Use, modification and distribution are subject to the
   5: // Boost Software License, Version 1.0.
   6: // (See accompanying file LICENSE_1_0.txt
   7: // or copy at http://www.boost.org/LICENSE_1_0.txt)
   8: 
   9: #ifndef BOOST_MATH_DISTRIBUTIONS_INVERSE_CHI_SQUARED_HPP
  10: #define BOOST_MATH_DISTRIBUTIONS_INVERSE_CHI_SQUARED_HPP
  11: 
  12: #include <boost/math/tools/config.hpp>
  13: #include <boost/math/tools/tuple.hpp>
  14: #include <boost/math/distributions/fwd.hpp>
  15: #include <boost/math/special_functions/gamma.hpp> // for incomplete beta.
  16: #include <boost/math/distributions/complement.hpp> // for complements.
  17: #include <boost/math/distributions/detail/common_error_handling.hpp> // for error checks.
  18: #include <boost/math/special_functions/fpclassify.hpp> // for isfinite
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/distributions/fwd.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/distributions/fwd.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: 
  20: // See http://en.wikipedia.org/wiki/Scaled-inverse-chi-square_distribution
  21: // for definitions of this scaled version.
  22: // See http://en.wikipedia.org/wiki/Inverse-chi-square_distribution
  23: // for unscaled version.
  24: 
  25: // http://reference.wolfram.com/mathematica/ref/InverseChiSquareDistribution.html
  26: // Weisstein, Eric W. "Inverse Chi-Squared Distribution." From MathWorld--A Wolfram Web Resource.
  27: // http://mathworld.wolfram.com/InverseChi-SquaredDistribution.html
  28: 
  29: namespace boost{ namespace math{
  30: 
  31: namespace detail
  32: {
  33:   template <class RealType, class Policy>
  34:   BOOST_MATH_GPU_ENABLED inline bool check_inverse_chi_squared( // Check both distribution parameters.
  35:         const char* function,
  36:         RealType degrees_of_freedom, // degrees_of_freedom (aka nu).
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:         RealType scale,  // scale (aka sigma^2)
  38:         RealType* result,
  39:         const Policy& pol)
  40:   {
  41:      return check_scale(function, scale, result, pol)
  42:        && check_df(function, degrees_of_freedom,
  43:        result, pol);
  44:   } // bool check_inverse_chi_squared
  45: } // namespace detail
  46: 
  47: template <class RealType = double, class Policy = policies::policy<> >
  48: class inverse_chi_squared_distribution
  49: {
  50: public:
  51:    typedef RealType value_type;
  52:    typedef Policy policy_type;
  53: 
  54:    BOOST_MATH_GPU_ENABLED inverse_chi_squared_distribution(RealType df, RealType l_scale) : m_df(df), m_scale (l_scale)
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:    {
  56:       RealType result;
  57:       detail::check_df(
  58:          "boost::math::inverse_chi_squared_distribution<%1%>::inverse_chi_squared_distribution",
  59:          m_df, &result, Policy())
  60:          && detail::check_scale(
  61: "boost::math::inverse_chi_squared_distribution<%1%>::inverse_chi_squared_distribution",
  62:          m_scale, &result,  Policy());
  63:    } // inverse_chi_squared_distribution constructor
  64: 
  65:    BOOST_MATH_GPU_ENABLED inverse_chi_squared_distribution(RealType df = 1) : m_df(df)
  66:    {
  67:       RealType result;
  68:       m_scale = 1 / m_df ; // Default scale = 1 / degrees of freedom (Wikipedia definition 1).
  69:       detail::check_df(
  70:          "boost::math::inverse_chi_squared_distribution<%1%>::inverse_chi_squared_distribution",
  71:          m_df, &result, Policy());
  72:    } // inverse_chi_squared_distribution
~~~
- **EN:** This range declares or defines callable logic such as Policy. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73: 
  74:    BOOST_MATH_GPU_ENABLED RealType degrees_of_freedom()const
  75:    {
  76:       return m_df; // aka nu
  77:    }
  78:    BOOST_MATH_GPU_ENABLED RealType scale()const
  79:    {
  80:       return m_scale;  // aka xi
  81:    }
  82: 
  83:    // Parameter estimation:  NOT implemented yet.
  84:    //static RealType find_degrees_of_freedom(
  85:    //   RealType difference_from_variance,
  86:    //   RealType alpha,
  87:    //   RealType beta,
  88:    //   RealType variance,
  89:    //   RealType hint = 100);
  90: 
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91: private:
  92:    // Data members:
  93:    RealType m_df;  // degrees of freedom are treated as a real number.
  94:    RealType m_scale;  // distribution scale.
  95: 
  96: }; // class chi_squared_distribution
  97: 
  98: typedef inverse_chi_squared_distribution<double> inverse_chi_squared;
  99: 
 100: #ifdef __cpp_deduction_guides
 101: template <class RealType>
 102: inverse_chi_squared_distribution(RealType)->inverse_chi_squared_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 103: template <class RealType>
 104: inverse_chi_squared_distribution(RealType,RealType)->inverse_chi_squared_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 105: #endif
 106: 
 107: template <class RealType, class Policy>
 108: BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const inverse_chi_squared_distribution<RealType, Policy>& /*dist*/)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `chi_squared_distribution` as part of the file's main abstraction. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `chi_squared_distribution`，作为该文件核心抽象的一部分。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109: {  // Range of permissible values for random variable x.
 110:    using boost::math::tools::max_value;
 111:    return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>()); // 0 to + infinity.
 112: }
 113: 
 114: template <class RealType, class Policy>
 115: BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const inverse_chi_squared_distribution<RealType, Policy>& /*dist*/)
 116: {  // Range of supported values for random variable x.
 117:    // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 118:    return boost::math::pair<RealType, RealType>(static_cast<RealType>(0), tools::max_value<RealType>()); // 0 to + infinity.
 119: }
 120: 
 121: template <class RealType, class Policy>
 122: BOOST_MATH_GPU_ENABLED RealType pdf(const inverse_chi_squared_distribution<RealType, Policy>& dist, const RealType& x)
 123: {
 124:    BOOST_MATH_STD_USING  // for ADL of std functions.
 125:    RealType df = dist.degrees_of_freedom();
 126:    RealType scale = dist.scale();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, scale。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:    RealType error_result;
 128: 
 129:    constexpr auto function = "boost::math::pdf(const inverse_chi_squared_distribution<%1%>&, %1%)";
 130: 
 131:    if(false == detail::check_inverse_chi_squared
 132:      (function, df, scale, &error_result, Policy())
 133:      )
 134:    { // Bad distribution.
 135:       return error_result;
 136:    }
 137:    if((x < 0) || !(boost::math::isfinite)(x))
 138:    { // Bad x.
 139:       return policies::raise_domain_error<RealType>(
 140:          function, "inverse Chi Square parameter was %1%, but must be >= 0 !", x, Policy());
 141:    }
 142: 
 143:    if(x == 0)
 144:    { // Treat as special case.
~~~
- **EN:** This range declares or defines callable logic such as Policy. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:      return 0;
 146:    }
 147:    // Wikipedia scaled inverse chi sq (df, scale) related to inv gamma (df/2, df * scale /2)
 148:    // so use inverse gamma pdf with shape = df/2, scale df * scale /2
 149:    // RealType shape = df /2; // inv_gamma shape
 150:    // RealType scale = df * scale/2; // inv_gamma scale
 151:    // RealType result = gamma_p_derivative(shape, scale / x, Policy()) * scale / (x * x);
 152:    RealType result = df * scale/2 / x;
 153:    if(result < tools::min_value<RealType>())
 154:       return 0; // Random variable is near enough infinite.
 155:    result = gamma_p_derivative(df/2, result, Policy()) * df * scale/2;
 156:    if(result != 0) // prevent 0 / 0,  gamma_p_derivative -> 0 faster than x^2
 157:       result /= (x * x);
 158:    return result;
 159: } // pdf
 160: 
 161: template <class RealType, class Policy>
 162: BOOST_MATH_GPU_ENABLED inline RealType cdf(const inverse_chi_squared_distribution<RealType, Policy>& dist, const RealType& x)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as gamma_p_derivative.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 gamma_p_derivative。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163: {
 164:    constexpr auto function = "boost::math::cdf(const inverse_chi_squared_distribution<%1%>&, %1%)";
 165:    RealType df = dist.degrees_of_freedom();
 166:    RealType scale = dist.scale();
 167:    RealType error_result;
 168: 
 169:    if(false ==
 170:        detail::check_inverse_chi_squared(function, df, scale, &error_result, Policy())
 171:      )
 172:    { // Bad distribution.
 173:       return error_result;
 174:    }
 175:    if((x < 0) || !(boost::math::isfinite)(x))
 176:    { // Bad x.
 177:       return policies::raise_domain_error<RealType>(
 178:          function, "inverse Chi Square parameter was %1%, but must be >= 0 !", x, Policy());
 179:    }
 180:    if (x == 0)
~~~
- **EN:** This range declares or defines callable logic such as degrees_of_freedom, scale, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, scale, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:    { // Treat zero as a special case.
 182:      return 0;
 183:    }
 184:    // RealType shape = df /2; // inv_gamma shape,
 185:    // RealType scale = df * scale/2; // inv_gamma scale,
 186:    // result = boost::math::gamma_q(shape, scale / x, Policy()); // inverse_gamma code.
 187:    return boost::math::gamma_q(df / 2, (df * (scale / 2)) / x, Policy());
 188: } // cdf
 189: 
 190: template <class RealType, class Policy>
 191: BOOST_MATH_GPU_ENABLED inline RealType quantile(const inverse_chi_squared_distribution<RealType, Policy>& dist, const RealType& p)
 192: {
 193:    using boost::math::gamma_q_inv;
 194:    RealType df = dist.degrees_of_freedom();
 195:    RealType scale = dist.scale();
 196: 
 197:    constexpr auto function = "boost::math::quantile(const inverse_chi_squared_distribution<%1%>&, %1%)";
 198:    // Error check:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::gamma_q, degrees_of_freedom, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::gamma_q, degrees_of_freedom, ...。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:    RealType error_result;
 200:    if(false == detail::check_df(
 201:          function, df, &error_result, Policy())
 202:          && detail::check_probability(
 203:             function, p, &error_result, Policy()))
 204:    {
 205:       return error_result;
 206:    }
 207:    if(false == detail::check_probability(
 208:             function, p, &error_result, Policy()))
 209:    {
 210:       return error_result;
 211:    }
 212:    // RealType shape = df /2; // inv_gamma shape,
 213:    // RealType scale = df * scale/2; // inv_gamma scale,
 214:    // result = scale / gamma_q_inv(shape, p, Policy());
 215:       RealType result = gamma_q_inv(df /2, p, Policy());
 216:       if(result == 0)
~~~
- **EN:** This range declares or defines callable logic such as gamma_q_inv. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 gamma_q_inv。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:          return policies::raise_overflow_error<RealType, Policy>(function, "Random variable is infinite.", Policy());
 218:       result = df * (scale / 2) / result;
 219:       return result;
 220: } // quantile
 221: 
 222: template <class RealType, class Policy>
 223: BOOST_MATH_GPU_ENABLED inline RealType cdf(const complemented2_type<inverse_chi_squared_distribution<RealType, Policy>, RealType>& c)
 224: {
 225:    using boost::math::gamma_q_inv;
 226:    RealType const& df = c.dist.degrees_of_freedom();
 227:    RealType const& scale = c.dist.scale();
 228:    RealType const& x = c.param;
 229:    constexpr auto function = "boost::math::cdf(const inverse_chi_squared_distribution<%1%>&, %1%)";
 230:    // Error check:
 231:    RealType error_result;
 232:    if(false == detail::check_df(
 233:          function, df, &error_result, Policy()))
 234:    {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, scale。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:       return error_result;
 236:    }
 237:    if (x == 0)
 238:    { // Treat zero as a special case.
 239:      return 1;
 240:    }
 241:    if((x < 0) || !(boost::math::isfinite)(x))
 242:    {
 243:       return policies::raise_domain_error<RealType>(
 244:          function, "inverse Chi Square parameter was %1%, but must be > 0 !", x, Policy());
 245:    }
 246:    // RealType shape = df /2; // inv_gamma shape,
 247:    // RealType scale = df * scale/2; // inv_gamma scale,
 248:    // result = gamma_p(shape, scale/c.param, Policy()); use inv_gamma.
 249: 
 250:    return gamma_p(df / 2, (df * scale/2) / x, Policy()); // OK
 251: } // cdf(complemented
 252: 
~~~
- **EN:** This range declares or defines callable logic such as Policy, gamma_p. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy, gamma_p。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253: template <class RealType, class Policy>
 254: BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<inverse_chi_squared_distribution<RealType, Policy>, RealType>& c)
 255: {
 256:    using boost::math::gamma_q_inv;
 257: 
 258:    RealType const& df = c.dist.degrees_of_freedom();
 259:    RealType const& scale = c.dist.scale();
 260:    RealType const& q = c.param;
 261:    constexpr auto function = "boost::math::quantile(const inverse_chi_squared_distribution<%1%>&, %1%)";
 262:    // Error check:
 263:    RealType error_result;
 264:    if(false == detail::check_df(function, df, &error_result, Policy()))
 265:    {
 266:       return error_result;
 267:    }
 268:    if(false == detail::check_probability(function, q, &error_result, Policy()))
 269:    {
 270:       return error_result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, scale。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:    }
 272:    // RealType shape = df /2; // inv_gamma shape,
 273:    // RealType scale = df * scale/2; // inv_gamma scale,
 274:    // result = scale / gamma_p_inv(shape, q, Policy());  // using inv_gamma.
 275:    RealType result = gamma_p_inv(df/2, q, Policy());
 276:    if(result == 0)
 277:       return policies::raise_overflow_error<RealType, Policy>(function, "Random variable is infinite.", Policy());
 278:    result = (df * scale / 2) / result;
 279:    return result;
 280: } // quantile(const complement
 281: 
 282: template <class RealType, class Policy>
 283: BOOST_MATH_GPU_ENABLED inline RealType mean(const inverse_chi_squared_distribution<RealType, Policy>& dist)
 284: { // Mean of inverse Chi-Squared distribution.
 285:    RealType df = dist.degrees_of_freedom();
 286:    RealType scale = dist.scale();
 287: 
 288:    constexpr auto function = "boost::math::mean(const inverse_chi_squared_distribution<%1%>&)";
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as gamma_p_inv, degrees_of_freedom, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 gamma_p_inv, degrees_of_freedom, ...。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:    if(df <= 2)
 290:       return policies::raise_domain_error<RealType>(
 291:          function,
 292:          "inverse Chi-Squared distribution only has a mode for degrees of freedom > 2, but got degrees of freedom = %1%.",
 293:          df, Policy());
 294:   return (df * scale) / (df - 2);
 295: } // mean
 296: 
 297: template <class RealType, class Policy>
 298: BOOST_MATH_GPU_ENABLED inline RealType variance(const inverse_chi_squared_distribution<RealType, Policy>& dist)
 299: { // Variance of inverse Chi-Squared distribution.
 300:    RealType df = dist.degrees_of_freedom();
 301:    RealType scale = dist.scale();
 302:    constexpr auto function = "boost::math::variance(const inverse_chi_squared_distribution<%1%>&)";
 303:    if(df <= 4)
 304:    {
 305:       return policies::raise_domain_error<RealType>(
 306:          function,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, degrees_of_freedom, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, degrees_of_freedom, ...。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:          "inverse Chi-Squared distribution only has a variance for degrees of freedom > 4, but got degrees of freedom = %1%.",
 308:          df, Policy());
 309:    }
 310:    return 2 * df * df * scale * scale / ((df - 2)*(df - 2) * (df - 4));
 311: } // variance
 312: 
 313: template <class RealType, class Policy>
 314: BOOST_MATH_GPU_ENABLED inline RealType mode(const inverse_chi_squared_distribution<RealType, Policy>& dist)
 315: { // mode is not defined in Mathematica.
 316:   // See Discussion section http://en.wikipedia.org/wiki/Talk:Scaled-inverse-chi-square_distribution
 317:   // for origin of the formula used below.
 318: 
 319:    RealType df = dist.degrees_of_freedom();
 320:    RealType scale = dist.scale();
 321:    constexpr auto function = "boost::math::mode(const inverse_chi_squared_distribution<%1%>&)";
 322:    if(df < 0)
 323:       return policies::raise_domain_error<RealType>(
 324:          function,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, degrees_of_freedom, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, degrees_of_freedom, ...。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:          "inverse Chi-Squared distribution only has a mode for degrees of freedom >= 0, but got degrees of freedom = %1%.",
 326:          df, Policy());
 327:    return (df * scale) / (df + 2);
 328: }
 329: 
 330: //template <class RealType, class Policy>
 331: //inline RealType median(const inverse_chi_squared_distribution<RealType, Policy>& dist)
 332: //{ // Median is given by Quantile[dist, 1/2]
 333: //   RealType df = dist.degrees_of_freedom();
 334: //   if(df <= 1)
 335: //      return tools::domain_error<RealType>(
 336: //         BOOST_CURRENT_FUNCTION,
 337: //         "The inverse_Chi-Squared distribution only has a median for degrees of freedom >= 0, but got degrees of freedom = %1%.",
 338: //         df);
 339: //   return df;
 340: //}
 341: // Now implemented via quantile(half) in derived accessors.
 342: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, degrees_of_freedom.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, degrees_of_freedom。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343: template <class RealType, class Policy>
 344: BOOST_MATH_GPU_ENABLED inline RealType skewness(const inverse_chi_squared_distribution<RealType, Policy>& dist)
 345: {
 346:    BOOST_MATH_STD_USING // For ADL
 347:    RealType df = dist.degrees_of_freedom();
 348:    constexpr auto function = "boost::math::skewness(const inverse_chi_squared_distribution<%1%>&)";
 349:    if(df <= 6)
 350:       return policies::raise_domain_error<RealType>(
 351:          function,
 352:          "inverse Chi-Squared distribution only has a skewness for degrees of freedom > 6, but got degrees of freedom = %1%.",
 353:          df, Policy());
 354: 
 355:    return 4 * sqrt (2 * (df - 4)) / (df - 6);  // Not a function of scale.
 356: }
 357: 
 358: template <class RealType, class Policy>
 359: BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const inverse_chi_squared_distribution<RealType, Policy>& dist)
 360: {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, Policy。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:    RealType df = dist.degrees_of_freedom();
 362:    constexpr auto function = "boost::math::kurtosis(const inverse_chi_squared_distribution<%1%>&)";
 363:    if(df <= 8)
 364:       return policies::raise_domain_error<RealType>(
 365:          function,
 366:          "inverse Chi-Squared distribution only has a kurtosis for degrees of freedom > 8, but got degrees of freedom = %1%.",
 367:          df, Policy());
 368: 
 369:    return kurtosis_excess(dist) + 3;
 370: }
 371: 
 372: template <class RealType, class Policy>
 373: BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const inverse_chi_squared_distribution<RealType, Policy>& dist)
 374: {
 375:    RealType df = dist.degrees_of_freedom();
 376:    constexpr auto function = "boost::math::kurtosis(const inverse_chi_squared_distribution<%1%>&)";
 377:    if(df <= 8)
 378:       return policies::raise_domain_error<RealType>(
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as degrees_of_freedom, Policy.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 degrees_of_freedom, Policy。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:          function,
 380:          "inverse Chi-Squared distribution only has a kurtosis excess for degrees of freedom > 8, but got degrees of freedom = %1%.",
 381:          df, Policy());
 382: 
 383:    return 12 * (5 * df - 22) / ((df - 6 )*(df - 8));  // Not a function of scale.
 384: }
 385: 
 386: //
 387: // Parameter estimation comes last:
 388: //
 389: 
 390: } // namespace math
 391: } // namespace boost
 392: 
 393: // This include must be at the end, *after* the accessors
 394: // for this distribution have been defined, in order to
 395: // keep compilers that support two-phase lookup happy.
 396: #include <boost/math/distributions/detail/derived_accessors.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp so the surrounding code can use external declarations. The code enters namespace scope (math::boost) to keep symbols organized. This range declares or defines callable logic such as Policy.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围声明或定义了可调用逻辑，例如 Policy。

### Lines 397-398 / 第 397-398 行
~~~cpp
 397: 
 398: #endif // BOOST_MATH_DISTRIBUTIONS_INVERSE_CHI_SQUARED_HPP
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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/gamma.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/special_functions/fpclassify.hpp, boost/math/distributions/detail/derived_accessors.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `Policy, degrees_of_freedom, scale, gamma_p_derivative, boost::math::gamma_q, gamma_q_inv, gamma_p, gamma_p_inv`
