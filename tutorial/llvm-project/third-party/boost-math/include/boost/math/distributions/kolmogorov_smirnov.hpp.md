# kolmogorov_smirnov.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/kolmogorov_smirnov.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the kolmogorov smirnov distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 kolmogorov smirnov 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // Kolmogorov-Smirnov 1st order asymptotic distribution
   2: // Copyright Evan Miller 2020
   3: //
   4: // Use, modification and distribution are subject to the
   5: // Boost Software License, Version 1.0. (See accompanying file
   6: // LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   7: //
   8: // The Kolmogorov-Smirnov test in statistics compares two empirical distributions,
   9: // or an empirical distribution against any theoretical distribution. It makes
  10: // use of a specific distribution which doesn't have a formal name, but which
  11: // is often called the Kolmogorv-Smirnov distribution for lack of anything
  12: // better. This file implements the limiting form of this distribution, first
  13: // identified by Andrey Kolmogorov in
  14: //
  15: // Kolmogorov, A. (1933) "Sulla Determinazione Empirica di una Legge di
  16: // Distribuzione." Giornale dell' Istituto Italiano degli Attuari
  17: //
  18: // This limiting form of the CDF is a first-order Taylor expansion that is
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: // easily implemented by the fourth Jacobi Theta function (setting z=0). The
  20: // PDF is then implemented here as a derivative of the Theta function. Note
  21: // that this derivative is with respect to x, which enters into \tau, and not
  22: // with respect to the z argument, which is always zero, and so the derivative
  23: // identities in DLMF 20.4 do not apply here.
  24: //
  25: // A higher order order expansion is possible, and was first outlined by
  26: //
  27: // Pelz W, Good IJ (1976). "Approximating the Lower Tail-Areas of the
  28: // Kolmogorov-Smirnov One-sample Statistic." Journal of the Royal Statistical
  29: // Society B.
  30: //
  31: // The terms in this expansion get fairly complicated, and as far as I know the
  32: // Pelz-Good expansion is not used in any statistics software. Someone could
  33: // consider updating this implementation to use the Pelz-Good expansion in the
  34: // future, but the math gets considerably hairier with each additional term.
  35: //
  36: // A formula for an exact version of the Kolmogorov-Smirnov test is laid out in
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: // Equation 2.4.4 of
  38: //
  39: // Durbin J (1973). "Distribution Theory for Tests Based on the Sample
  40: // Distribution Func- tion." In SIAM CBMS-NSF Regional Conference Series in
  41: // Applied Mathematics. SIAM, Philadelphia, PA.
  42: //
  43: // which is available in book form from Amazon and others. This exact version
  44: // involves taking powers of large matrices. To do that right you need to
  45: // compute eigenvalues and eigenvectors, which are beyond the scope of Boost.
  46: // (Some recent work indicates the exact form can also be computed via FFT, see
  47: // https://cran.r-project.org/web/packages/KSgeneral/KSgeneral.pdf).
  48: //
  49: // Even if the CDF of the exact distribution could be computed using Boost
  50: // libraries (which would be cumbersome), the PDF would present another
  51: // difficulty. Therefore I am limiting this implementation to the asymptotic
  52: // form, even though the exact form has trivial values for certain specific
  53: // values of x and n. For more on trivial values see
  54: //
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: // Ruben H, Gambino J (1982). "The Exact Distribution of Kolmogorov's Statistic
  56: // Dn for n <= 10." Annals of the Institute of Statistical Mathematics.
  57: //
  58: // For a good bibliography and overview of the various algorithms, including
  59: // both exact and asymptotic forms, see
  60: // https://www.jstatsoft.org/article/view/v039i11
  61: //
  62: // As for this implementation: the distribution is parameterized by n (number
  63: // of observations) in the spirit of chi-squared's degrees of freedom. It then
  64: // takes a single argument x. In terms of the Kolmogorov-Smirnov statistical
  65: // test, x represents the distribution of D_n, where D_n is the maximum
  66: // difference between the CDFs being compared, that is,
  67: //
  68: //   D_n = sup|F_n(x) - G(x)|
  69: //
  70: // In the exact distribution, x is confined to the support [0, 1], but in this
  71: // limiting approximation, we allow x to exceed unity (similar to how a normal
  72: // approximation always spills over any boundaries).
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73: //
  74: // As mentioned previously, the CDF is implemented using the \tau
  75: // parameterization of the fourth Jacobi Theta function as
  76: //
  77: // CDF=theta_4(0|2*x*x*n/pi)
  78: //
  79: // The PDF is a hand-coded derivative of that function. Actually, there are two
  80: // (independent) derivatives, as separate code paths are used for "small x"
  81: // (2*x*x*n < pi) and "large x", mirroring the separate code paths in the
  82: // Jacobi Theta implementation to achieve fast convergence. Quantiles are
  83: // computed using a Newton-Raphson iteration from an initial guess that I
  84: // arrived at by trial and error.
  85: //
  86: // The mean and variance are implemented using simple closed-form expressions.
  87: // Skewness and kurtosis use slightly more complicated closed-form expressions
  88: // that involve the zeta function. The mode is calculated at run-time by
  89: // maximizing the PDF. If you have an analytical solution for the mode, feel
  90: // free to plop it in.
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91: //
  92: // The CDF and PDF could almost certainly be re-implemented and sped up using a
  93: // polynomial or rational approximation, since the only meaningful argument is
  94: // x * sqrt(n). But that is left as an exercise for the next maintainer.
  95: //
  96: // In the future, the Pelz-Good approximation could be added. I suggest adding
  97: // a second parameter representing the order, e.g.
  98: //
  99: // kolmogorov_smirnov_dist<>(100) // N=100, order=1
 100: // kolmogorov_smirnov_dist<>(100, 1) // N=100, order=1, i.e. Kolmogorov's formula
 101: // kolmogorov_smirnov_dist<>(100, 4) // N=100, order=4, i.e. Pelz-Good formula
 102: //
 103: // The exact distribution could be added to the API with a special order
 104: // parameter (e.g. 0 or infinity), or a separate distribution type altogether
 105: // (e.g. kolmogorov_smirnov_exact_distribution).
 106: //
 107: #ifndef BOOST_MATH_DISTRIBUTIONS_KOLMOGOROV_SMIRNOV_HPP
 108: #define BOOST_MATH_DISTRIBUTIONS_KOLMOGOROV_SMIRNOV_HPP
~~~
- **EN:** It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109: 
 110: #include <boost/math/distributions/fwd.hpp>
 111: #include <boost/math/distributions/complement.hpp>
 112: #include <boost/math/distributions/detail/common_error_handling.hpp>
 113: #include <boost/math/special_functions/jacobi_theta.hpp>
 114: #include <boost/math/tools/tuple.hpp>
 115: #include <boost/math/tools/roots.hpp> // Newton-Raphson
 116: #include <boost/math/tools/minima.hpp> // For the mode
 117: 
 118: namespace boost { namespace math {
 119: 
 120: namespace detail {
 121: template <class RealType>
 122: inline RealType kolmogorov_smirnov_quantile_guess(RealType p) {
 123:     // Choose a starting point for the Newton-Raphson iteration
 124:     if (p > 0.9)
 125:         return RealType(1.8) - 5 * (1 - p);
 126:     if (p < 0.3)
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/fwd.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/distributions/fwd.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:         return p + RealType(0.45);
 128:     return p + RealType(0.3);
 129: }
 130: 
 131: // d/dk (theta2(0, 1/(2*k*k/M_PI))/sqrt(2*k*k*M_PI))
 132: template <class RealType, class Policy>
 133: RealType kolmogorov_smirnov_pdf_small_x(RealType x, RealType n, const Policy&) {
 134:     BOOST_MATH_STD_USING
 135:     RealType value = RealType(0), delta = RealType(0), last_delta = RealType(0);
 136:     RealType eps = policies::get_epsilon<RealType, Policy>();
 137:     int i = 0;
 138:     RealType pi2 = constants::pi_sqr<RealType>();
 139:     RealType x2n = x*x*n;
 140:     if (x2n*x2n == 0.0) {
 141:         return static_cast<RealType>(0);
 142:     }
 143:     while (true) {
 144:         delta = exp(-RealType(i+0.5)*RealType(i+0.5)*pi2/(2*x2n)) * (RealType(i+0.5)*RealType(i+0.5)*pi2 - x2n);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as kolmogorov_smirnov_pdf_small_x, RealType, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 kolmogorov_smirnov_pdf_small_x, RealType, ...。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145: 
 146:         if (delta == 0.0)
 147:             break;
 148: 
 149:         if (last_delta != 0.0 && fabs(delta/last_delta) < eps)
 150:             break;
 151: 
 152:         value += delta + delta;
 153:         last_delta = delta;
 154:         i++;
 155:     }
 156: 
 157:     return value * sqrt(n) * constants::root_half_pi<RealType>() / (x2n*x2n);
 158: }
 159: 
 160: // d/dx (theta4(0, 2*x*x*n/M_PI))
 161: template <class RealType, class Policy>
 162: inline RealType kolmogorov_smirnov_pdf_large_x(RealType x, RealType n, const Policy&) {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as kolmogorov_smirnov_pdf_large_x.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 kolmogorov_smirnov_pdf_large_x。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:     BOOST_MATH_STD_USING
 164:     RealType value = RealType(0), delta = RealType(0), last_delta = RealType(0);
 165:     RealType eps = policies::get_epsilon<RealType, Policy>();
 166:     int i = 1;
 167:     while (true) {
 168:         delta = 8*x*i*i*exp(-2*i*i*x*x*n);
 169: 
 170:         if (delta == 0.0)
 171:             break;
 172: 
 173:         if (last_delta != 0.0 && fabs(delta / last_delta) < eps)
 174:             break;
 175: 
 176:         if (i%2 == 0)
 177:             delta = -delta;
 178: 
 179:         value += delta;
 180:         last_delta = delta;
~~~
- **EN:** This range declares or defines callable logic such as RealType, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 RealType, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:         i++;
 182:     }
 183: 
 184:     return value * n;
 185: }
 186: 
 187: } // detail
 188: 
 189: template <class RealType = double, class Policy = policies::policy<> >
 190:     class kolmogorov_smirnov_distribution
 191: {
 192:     public:
 193:         typedef RealType value_type;
 194:         typedef Policy policy_type;
 195: 
 196:         // Constructor
 197:     kolmogorov_smirnov_distribution( RealType n ) : n_obs_(n)
 198:     {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:         RealType result;
 200:         detail::check_df(
 201:                 "boost::math::kolmogorov_smirnov_distribution<%1%>::kolmogorov_smirnov_distribution", n_obs_, &result, Policy());
 202:     }
 203: 
 204:     RealType number_of_observations()const
 205:     {
 206:         return n_obs_;
 207:     }
 208: 
 209:     private:
 210: 
 211:     RealType n_obs_; // positive integer
 212: };
 213: 
 214: typedef kolmogorov_smirnov_distribution<double> kolmogorov_k; // Convenience typedef for double version.
 215: 
 216: #ifdef __cpp_deduction_guides
~~~
- **EN:** This range declares or defines callable logic such as Policy. Return statements hand the computed result or status back to the caller. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Policy。 return 语句会把计算结果或状态返回给调用方。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217: template <class RealType>
 218: kolmogorov_smirnov_distribution(RealType)->kolmogorov_smirnov_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 219: #endif
 220: 
 221: namespace detail {
 222: template <class RealType, class Policy>
 223: struct kolmogorov_smirnov_quantile_functor
 224: {
 225:   kolmogorov_smirnov_quantile_functor(const boost::math::kolmogorov_smirnov_distribution<RealType, Policy> dist, RealType const& p)
 226:     : distribution(dist), prob(p)
 227:   {
 228:   }
 229: 
 230:   boost::math::tuple<RealType, RealType> operator()(RealType const& x)
 231:   {
 232:     RealType fx = cdf(distribution, x) - prob;  // Difference cdf - value - to minimize.
 233:     RealType dx = pdf(distribution, x); // pdf is 1st derivative.
 234:     // return both function evaluation difference f(x) and 1st derivative f'(x).
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:     return boost::math::make_tuple(fx, dx);
 236:   }
 237: private:
 238:   const boost::math::kolmogorov_smirnov_distribution<RealType, Policy> distribution;
 239:   RealType prob;
 240: };
 241: 
 242: template <class RealType, class Policy>
 243: struct kolmogorov_smirnov_complementary_quantile_functor
 244: {
 245:   kolmogorov_smirnov_complementary_quantile_functor(const boost::math::kolmogorov_smirnov_distribution<RealType, Policy> dist, RealType const& p)
 246:     : distribution(dist), prob(p)
 247:   {
 248:   }
 249: 
 250:   boost::math::tuple<RealType, RealType> operator()(RealType const& x)
 251:   {
 252:     RealType fx = cdf(complement(distribution, x)) - prob;  // Difference cdf - value - to minimize.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:     RealType dx = -pdf(distribution, x); // pdf is the negative of the derivative of (1-CDF)
 254:     // return both function evaluation difference f(x) and 1st derivative f'(x).
 255:     return boost::math::make_tuple(fx, dx);
 256:   }
 257: private:
 258:   const boost::math::kolmogorov_smirnov_distribution<RealType, Policy> distribution;
 259:   RealType prob;
 260: };
 261: 
 262: template <class RealType, class Policy>
 263: struct kolmogorov_smirnov_negative_pdf_functor
 264: {
 265:     RealType operator()(RealType const& x) {
 266:         if (2*x*x < constants::pi<RealType>()) {
 267:             return -kolmogorov_smirnov_pdf_small_x(x, static_cast<RealType>(1), Policy());
 268:         }
 269:         return -kolmogorov_smirnov_pdf_large_x(x, static_cast<RealType>(1), Policy());
 270:     }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as pdf, operator.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 pdf, operator。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271: };
 272: } // namespace detail
 273: 
 274: template <class RealType, class Policy>
 275: inline const std::pair<RealType, RealType> range(const kolmogorov_smirnov_distribution<RealType, Policy>& /*dist*/)
 276: { // Range of permissible values for random variable x.
 277:    using boost::math::tools::max_value;
 278:    return std::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>());
 279: }
 280: 
 281: template <class RealType, class Policy>
 282: inline const std::pair<RealType, RealType> support(const kolmogorov_smirnov_distribution<RealType, Policy>& /*dist*/)
 283: { // Range of supported values for random variable x.
 284:    // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 285:    // In the exact distribution, the upper limit would be 1.
 286:    using boost::math::tools::max_value;
 287:    return std::pair<RealType, RealType>(static_cast<RealType>(0), max_value<RealType>());
 288: }
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289: 
 290: template <class RealType, class Policy>
 291: inline RealType pdf(const kolmogorov_smirnov_distribution<RealType, Policy>& dist, const RealType& x)
 292: {
 293:    BOOST_FPU_EXCEPTION_GUARD
 294:    BOOST_MATH_STD_USING  // for ADL of std functions.
 295: 
 296:    RealType n = dist.number_of_observations();
 297:    RealType error_result;
 298:    static const char* function = "boost::math::pdf(const kolmogorov_smirnov_distribution<%1%>&, %1%)";
 299:    if(false == detail::check_x_not_NaN(function, x, &error_result, Policy()))
 300:       return error_result;
 301: 
 302:    if(false == detail::check_df(function, n, &error_result, Policy()))
 303:       return error_result;
 304: 
 305:    if (x < 0 || !(boost::math::isfinite)(x))
 306:    {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as number_of_observations.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 number_of_observations。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:       return policies::raise_domain_error<RealType>(
 308:          function, "Kolmogorov-Smirnov parameter was %1%, but must be > 0 !", x, Policy());
 309:    }
 310: 
 311:    if (2*x*x*n < constants::pi<RealType>()) {
 312:        return detail::kolmogorov_smirnov_pdf_small_x(x, n, Policy());
 313:    }
 314: 
 315:    return detail::kolmogorov_smirnov_pdf_large_x(x, n, Policy());
 316: } // pdf
 317: 
 318: template <class RealType, class Policy>
 319: inline RealType cdf(const kolmogorov_smirnov_distribution<RealType, Policy>& dist, const RealType& x)
 320: {
 321:     BOOST_MATH_STD_USING // for ADL of std function exp.
 322:    static const char* function = "boost::math::cdf(const kolmogorov_smirnov_distribution<%1%>&, %1%)";
 323:    RealType error_result;
 324:    RealType n = dist.number_of_observations();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, number_of_observations.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, number_of_observations。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:    if(false == detail::check_x_not_NaN(function, x, &error_result, Policy()))
 326:       return error_result;
 327:    if(false == detail::check_df(function, n, &error_result, Policy()))
 328:       return error_result;
 329:    if((x < 0) || !(boost::math::isfinite)(x)) {
 330:       return policies::raise_domain_error<RealType>(
 331:          function, "Random variable parameter was %1%, but must be between > 0 !", x, Policy());
 332:    }
 333: 
 334:    if (x*x*n == 0)
 335:        return 0;
 336: 
 337:    return jacobi_theta4tau(RealType(0), 2*x*x*n/constants::pi<RealType>(), Policy());
 338: } // cdf
 339: 
 340: template <class RealType, class Policy>
 341: inline RealType cdf(const complemented2_type<kolmogorov_smirnov_distribution<RealType, Policy>, RealType>& c) {
 342:     BOOST_MATH_STD_USING // for ADL of std function exp.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as Policy, cdf.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 Policy, cdf。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:     RealType x = c.param;
 344:    static const char* function = "boost::math::cdf(const complemented2_type<const kolmogorov_smirnov_distribution<%1%>&, %1%>)";
 345:    RealType error_result;
 346:    kolmogorov_smirnov_distribution<RealType, Policy> const& dist = c.dist;
 347:    RealType n = dist.number_of_observations();
 348: 
 349:    if(false == detail::check_x_not_NaN(function, x, &error_result, Policy()))
 350:       return error_result;
 351:    if(false == detail::check_df(function, n, &error_result, Policy()))
 352:       return error_result;
 353: 
 354:    if((x < 0) || !(boost::math::isfinite)(x))
 355:       return policies::raise_domain_error<RealType>(
 356:          function, "Random variable parameter was %1%, but must be between > 0 !", x, Policy());
 357: 
 358:    if (x*x*n == 0)
 359:        return 1;
 360: 
~~~
- **EN:** This range declares or defines callable logic such as number_of_observations, Policy. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 number_of_observations, Policy。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:    if (2*x*x*n > constants::pi<RealType>())
 362:        return -jacobi_theta4m1tau(RealType(0), 2*x*x*n/constants::pi<RealType>(), Policy());
 363: 
 364:    return RealType(1) - jacobi_theta4tau(RealType(0), 2*x*x*n/constants::pi<RealType>(), Policy());
 365: } // cdf (complemented)
 366: 
 367: template <class RealType, class Policy>
 368: inline RealType quantile(const kolmogorov_smirnov_distribution<RealType, Policy>& dist, const RealType& p)
 369: {
 370:     BOOST_MATH_STD_USING
 371:    static const char* function = "boost::math::quantile(const kolmogorov_smirnov_distribution<%1%>&, %1%)";
 372:    // Error check:
 373:    RealType error_result;
 374:    RealType n = dist.number_of_observations();
 375:    if(false == detail::check_probability(function, p, &error_result, Policy()))
 376:       return error_result;
 377:    if(false == detail::check_df(function, n, &error_result, Policy()))
 378:       return error_result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as number_of_observations.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 number_of_observations。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379: 
 380:    RealType k = detail::kolmogorov_smirnov_quantile_guess(p) / sqrt(n);
 381:    const int get_digits = policies::digits<RealType, Policy>();// get digits from policy,
 382:    std::uintmax_t max_iter = policies::get_max_root_iterations<Policy>(); // and max iterations.
 383: 
 384:    RealType result = tools::newton_raphson_iterate(detail::kolmogorov_smirnov_quantile_functor<RealType, Policy>(dist, p),
 385:            k, RealType(0), RealType(1), get_digits, max_iter);
 386:    if (max_iter >= policies::get_max_root_iterations<Policy>())
 387:    {
 388:       return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 389:          " either there is no answer to quantile or the answer is infinite.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
 390:    }
 391:    return result;
 392: } // quantile
 393: 
 394: template <class RealType, class Policy>
 395: inline RealType quantile(const complemented2_type<kolmogorov_smirnov_distribution<RealType, Policy>, RealType>& c) {
 396:     BOOST_MATH_STD_USING
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as detail::kolmogorov_smirnov_quantile_guess, RealType, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 detail::kolmogorov_smirnov_quantile_guess, RealType, ...。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:    static const char* function = "boost::math::quantile(const kolmogorov_smirnov_distribution<%1%>&, %1%)";
 398:    kolmogorov_smirnov_distribution<RealType, Policy> const& dist = c.dist;
 399:    RealType n = dist.number_of_observations();
 400:    // Error check:
 401:    RealType error_result;
 402:    RealType p = c.param;
 403: 
 404:    if(false == detail::check_probability(function, p, &error_result, Policy()))
 405:       return error_result;
 406:    if(false == detail::check_df(function, n, &error_result, Policy()))
 407:       return error_result;
 408: 
 409:    RealType k = detail::kolmogorov_smirnov_quantile_guess(RealType(1-p)) / sqrt(n);
 410: 
 411:    const int get_digits = policies::digits<RealType, Policy>();// get digits from policy,
 412:    std::uintmax_t max_iter = policies::get_max_root_iterations<Policy>(); // and max iterations.
 413: 
 414:    RealType result = tools::newton_raphson_iterate(
~~~
- **EN:** This range declares or defines callable logic such as number_of_observations, detail::kolmogorov_smirnov_quantile_guess. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 number_of_observations, detail::kolmogorov_smirnov_quantile_guess。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:            detail::kolmogorov_smirnov_complementary_quantile_functor<RealType, Policy>(dist, p),
 416:            k, RealType(0), RealType(1), get_digits, max_iter);
 417:    if (max_iter >= policies::get_max_root_iterations<Policy>())
 418:    {
 419:       return policies::raise_evaluation_error<RealType>(function, "Unable to locate solution in a reasonable time:" // LCOV_EXCL_LINE
 420:          " either there is no answer to quantile or the answer is infinite.  Current best guess is %1%", result, Policy()); // LCOV_EXCL_LINE
 421:    }
 422:    return result;
 423: } // quantile (complemented)
 424: 
 425: template <class RealType, class Policy>
 426: inline RealType mode(const kolmogorov_smirnov_distribution<RealType, Policy>& dist)
 427: {
 428:     BOOST_MATH_STD_USING
 429:    static const char* function = "boost::math::mode(const kolmogorov_smirnov_distribution<%1%>&)";
 430:    RealType n = dist.number_of_observations();
 431:    RealType error_result;
 432:    if(false == detail::check_df(function, n, &error_result, Policy()))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as RealType, Policy, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 RealType, Policy, ...。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:       return error_result;
 434: 
 435:     std::pair<RealType, RealType> r = boost::math::tools::brent_find_minima(
 436:             detail::kolmogorov_smirnov_negative_pdf_functor<RealType, Policy>(),
 437:             static_cast<RealType>(0), static_cast<RealType>(1), policies::digits<RealType, Policy>());
 438:     return r.first / sqrt(n);
 439: }
 440: 
 441: // Mean and variance come directly from
 442: // https://www.jstatsoft.org/article/view/v008i18 Section 3
 443: template <class RealType, class Policy>
 444: inline RealType mean(const kolmogorov_smirnov_distribution<RealType, Policy>& dist)
 445: {
 446:     BOOST_MATH_STD_USING
 447:    static const char* function = "boost::math::mean(const kolmogorov_smirnov_distribution<%1%>&)";
 448:     RealType n = dist.number_of_observations();
 449:     RealType error_result;
 450:     if(false == detail::check_df(function, n, &error_result, Policy()))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as number_of_observations.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 number_of_observations。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:         return error_result;
 452:     return constants::root_half_pi<RealType>() * constants::ln_two<RealType>() / sqrt(n);
 453: }
 454: 
 455: template <class RealType, class Policy>
 456: inline RealType variance(const kolmogorov_smirnov_distribution<RealType, Policy>& dist)
 457: {
 458:    static const char* function = "boost::math::variance(const kolmogorov_smirnov_distribution<%1%>&)";
 459:     RealType n = dist.number_of_observations();
 460:     RealType error_result;
 461:     if(false == detail::check_df(function, n, &error_result, Policy()))
 462:         return error_result;
 463:     return (constants::pi_sqr_div_six<RealType>()
 464:             - constants::pi<RealType>() * constants::ln_two<RealType>() * constants::ln_two<RealType>()) / (2*n);
 465: }
 466: 
 467: // Skewness and kurtosis come from integrating the PDF
 468: // The alternating series pops out a Dirichlet eta function which is related to the zeta function
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as number_of_observations.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 number_of_observations。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469: template <class RealType, class Policy>
 470: inline RealType skewness(const kolmogorov_smirnov_distribution<RealType, Policy>& dist)
 471: {
 472:     BOOST_MATH_STD_USING
 473:    static const char* function = "boost::math::skewness(const kolmogorov_smirnov_distribution<%1%>&)";
 474:     RealType n = dist.number_of_observations();
 475:     RealType error_result;
 476:     if(false == detail::check_df(function, n, &error_result, Policy()))
 477:         return error_result;
 478:     RealType ex3 = RealType(0.5625) * constants::root_half_pi<RealType>() * constants::zeta_three<RealType>() / n / sqrt(n);
 479:     RealType mean = boost::math::mean(dist);
 480:     RealType var = boost::math::variance(dist);
 481:     return (ex3 - 3 * mean * var - mean * mean * mean) / var / sqrt(var);
 482: }
 483: 
 484: template <class RealType, class Policy>
 485: inline RealType kurtosis(const kolmogorov_smirnov_distribution<RealType, Policy>& dist)
 486: {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as number_of_observations, RealType, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 number_of_observations, RealType, ...。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:     BOOST_MATH_STD_USING
 488:    static const char* function = "boost::math::kurtosis(const kolmogorov_smirnov_distribution<%1%>&)";
 489:     RealType n = dist.number_of_observations();
 490:     RealType error_result;
 491:     if(false == detail::check_df(function, n, &error_result, Policy()))
 492:         return error_result;
 493:     RealType ex4 = 7 * constants::pi_sqr_div_six<RealType>() * constants::pi_sqr_div_six<RealType>() / 20 / n / n;
 494:     RealType mean = boost::math::mean(dist);
 495:     RealType var = boost::math::variance(dist);
 496:     RealType skew = boost::math::skewness(dist);
 497:     return (ex4 - 4 * mean * skew * var * sqrt(var) - 6 * mean * mean * var - mean * mean * mean * mean) / var / var;
 498: }
 499: 
 500: template <class RealType, class Policy>
 501: inline RealType kurtosis_excess(const kolmogorov_smirnov_distribution<RealType, Policy>& dist)
 502: {
 503:    static const char* function = "boost::math::kurtosis_excess(const kolmogorov_smirnov_distribution<%1%>&)";
 504:     RealType n = dist.number_of_observations();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as number_of_observations, boost::math::mean, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 number_of_observations, boost::math::mean, ...。

### Lines 505-511 / 第 505-511 行
~~~cpp
 505:     RealType error_result;
 506:     if(false == detail::check_df(function, n, &error_result, Policy()))
 507:         return error_result;
 508:     return kurtosis(dist) - 3;
 509: }
 510: }}
 511: #endif
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

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
- **Included headers / 包含头文件**: `boost/math/distributions/fwd.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/special_functions/jacobi_theta.hpp, boost/math/tools/tuple.hpp, boost/math/tools/roots.hpp, boost/math/tools/minima.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `kolmogorov_smirnov_quantile_guess, kolmogorov_smirnov_pdf_small_x, RealType, exp, kolmogorov_smirnov_pdf_large_x, Policy, pdf, operator, ...`
