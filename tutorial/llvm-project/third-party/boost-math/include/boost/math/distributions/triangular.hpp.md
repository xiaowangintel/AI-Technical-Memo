# triangular.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/triangular.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the triangular distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 triangular 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  Copyright John Maddock 2006, 2007.
   2: //  Copyright Paul A. Bristow 2006, 2007.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_STATS_TRIANGULAR_HPP
   8: #define BOOST_STATS_TRIANGULAR_HPP
   9: 
  10: // http://mathworld.wolfram.com/TriangularDistribution.html
  11: // Note that the 'constructors' defined by Wolfram are difference from those here,
  12: // for example
  13: // N[variance[triangulardistribution{1, +2}, 1.5], 50] computes
  14: // 0.041666666666666666666666666666666666666666666666667
  15: // TriangularDistribution{1, +2}, 1.5 is the analog of triangular_distribution(1, 1.5, 2)
  16: 
  17: // http://en.wikipedia.org/wiki/Triangular_distribution
  18: 
~~~
- **EN:** It establishes an inclusion guard so the header is processed only once per translation unit. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/tools/config.hpp>
  20: #include <boost/math/tools/tuple.hpp>
  21: #include <boost/math/tools/promotion.hpp>
  22: #include <boost/math/distributions/fwd.hpp>
  23: #include <boost/math/special_functions/expm1.hpp>
  24: #include <boost/math/distributions/detail/common_error_handling.hpp>
  25: #include <boost/math/distributions/complement.hpp>
  26: #include <boost/math/constants/constants.hpp>
  27: 
  28: namespace boost{ namespace math
  29: {
  30:   namespace detail
  31:   {
  32:     template <class RealType, class Policy>
  33:     BOOST_MATH_GPU_ENABLED inline bool check_triangular_lower(
  34:       const char* function,
  35:       RealType lower,
  36:       RealType* result, const Policy& pol)
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/promotion.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/promotion.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:     {
  38:       if((boost::math::isfinite)(lower))
  39:       { // Any finite value is OK.
  40:         return true;
  41:       }
  42:       else
  43:       { // Not finite: infinity or NaN.
  44:         *result = policies::raise_domain_error<RealType>(
  45:           function,
  46:           "Lower parameter is %1%, but must be finite!", lower, pol);
  47:         return false;
  48:       }
  49:     } // bool check_triangular_lower(
  50: 
  51:     template <class RealType, class Policy>
  52:     BOOST_MATH_GPU_ENABLED inline bool check_triangular_mode(
  53:       const char* function,
  54:       RealType mode,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:       RealType* result, const Policy& pol)
  56:     {
  57:       if((boost::math::isfinite)(mode))
  58:       { // any finite value is OK.
  59:         return true;
  60:       }
  61:       else
  62:       { // Not finite: infinity or NaN.
  63:         *result = policies::raise_domain_error<RealType>(
  64:           function,
  65:           "Mode parameter is %1%, but must be finite!", mode, pol);
  66:         return false;
  67:       }
  68:     } // bool check_triangular_mode(
  69: 
  70:     template <class RealType, class Policy>
  71:     BOOST_MATH_GPU_ENABLED inline bool check_triangular_upper(
  72:       const char* function,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       RealType upper,
  74:       RealType* result, const Policy& pol)
  75:     {
  76:       if((boost::math::isfinite)(upper))
  77:       { // any finite value is OK.
  78:         return true;
  79:       }
  80:       else
  81:       { // Not finite: infinity or NaN.
  82:         *result = policies::raise_domain_error<RealType>(
  83:           function,
  84:           "Upper parameter is %1%, but must be finite!", upper, pol);
  85:         return false;
  86:       }
  87:     } // bool check_triangular_upper(
  88: 
  89:     template <class RealType, class Policy>
  90:     BOOST_MATH_GPU_ENABLED inline bool check_triangular_x(
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:       const char* function,
  92:       RealType const& x,
  93:       RealType* result, const Policy& pol)
  94:     {
  95:       if((boost::math::isfinite)(x))
  96:       { // Any finite value is OK
  97:         return true;
  98:       }
  99:       else
 100:       { // Not finite: infinity or NaN.
 101:         *result = policies::raise_domain_error<RealType>(
 102:           function,
 103:           "x parameter is %1%, but must be finite!", x, pol);
 104:         return false;
 105:       }
 106:     } // bool check_triangular_x
 107: 
 108:     template <class RealType, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:     BOOST_MATH_GPU_ENABLED inline bool check_triangular(
 110:       const char* function,
 111:       RealType lower,
 112:       RealType mode,
 113:       RealType upper,
 114:       RealType* result, const Policy& pol)
 115:     {
 116:       if ((check_triangular_lower(function, lower, result, pol) == false)
 117:         || (check_triangular_mode(function, mode, result, pol) == false)
 118:         || (check_triangular_upper(function, upper, result, pol) == false))
 119:       { // Some parameter not finite.
 120:         return false;
 121:       }
 122:       else if (lower >= upper) // lower == upper NOT useful.
 123:       { // lower >= upper.
 124:         *result = policies::raise_domain_error<RealType>(
 125:           function,
 126:           "lower parameter is %1%, but must be less than upper!", lower, pol);
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:         return false;
 128:       }
 129:       else
 130:       { // Check lower <= mode <= upper.
 131:         if (mode < lower)
 132:         {
 133:           *result = policies::raise_domain_error<RealType>(
 134:             function,
 135:             "mode parameter is %1%, but must be >= than lower!", lower, pol);
 136:           return false;
 137:         }
 138:         if (mode > upper)
 139:         {
 140:           *result = policies::raise_domain_error<RealType>(
 141:             function,
 142:             "mode parameter is %1%, but must be <= than upper!", upper, pol);
 143:           return false;
 144:         }
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:         return true; // All OK.
 146:       }
 147:     } // bool check_triangular
 148:   } // namespace detail
 149: 
 150:   template <class RealType = double, class Policy = policies::policy<> >
 151:   class triangular_distribution
 152:   {
 153:   public:
 154:     typedef RealType value_type;
 155:     typedef Policy policy_type;
 156: 
 157:     BOOST_MATH_GPU_ENABLED triangular_distribution(RealType l_lower = -1, RealType l_mode = 0, RealType l_upper = 1)
 158:       : m_lower(l_lower), m_mode(l_mode), m_upper(l_upper) // Constructor.
 159:     { // Evans says 'standard triangular' is lower 0, mode 1/2, upper 1,
 160:       // has median sqrt(c/2) for c <=1/2 and 1 - sqrt(1-c)/2 for c >= 1/2
 161:       // But this -1, 0, 1 is more useful in most applications to approximate normal distribution,
 162:       // where the central value is the most likely and deviations either side equally likely.
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:       RealType result;
 164:       detail::check_triangular("boost::math::triangular_distribution<%1%>::triangular_distribution",l_lower, l_mode, l_upper, &result, Policy());
 165:     }
 166:     // Accessor functions.
 167:     BOOST_MATH_GPU_ENABLED RealType lower()const
 168:     {
 169:       return m_lower;
 170:     }
 171:     BOOST_MATH_GPU_ENABLED RealType mode()const
 172:     {
 173:       return m_mode;
 174:     }
 175:     BOOST_MATH_GPU_ENABLED RealType upper()const
 176:     {
 177:       return m_upper;
 178:     }
 179:   private:
 180:     // Data members:
~~~
- **EN:** This range declares or defines callable logic such as detail::check_triangular. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::check_triangular。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:     RealType m_lower;  // distribution lower aka a
 182:     RealType m_mode;  // distribution mode aka c
 183:     RealType m_upper;  // distribution upper aka b
 184:   }; // class triangular_distribution
 185: 
 186:   typedef triangular_distribution<double> triangular;
 187: 
 188:   #ifdef __cpp_deduction_guides
 189:   template <class RealType>
 190:   triangular_distribution(RealType)->triangular_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 191:   template <class RealType>
 192:   triangular_distribution(RealType,RealType)->triangular_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 193:   template <class RealType>
 194:   triangular_distribution(RealType,RealType,RealType)->triangular_distribution<typename boost::math::tools::promote_args<RealType>::type>;
 195:   #endif
 196: 
 197:   template <class RealType, class Policy>
 198:   BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const triangular_distribution<RealType, Policy>& /* dist */)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `triangular_distribution` as part of the file's main abstraction. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `triangular_distribution`，作为该文件核心抽象的一部分。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:   { // Range of permissible values for random variable x.
 200:     using boost::math::tools::max_value;
 201:     return boost::math::pair<RealType, RealType>(-max_value<RealType>(), max_value<RealType>());
 202:   }
 203: 
 204:   template <class RealType, class Policy>
 205:   BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const triangular_distribution<RealType, Policy>& dist)
 206:   { // Range of supported values for random variable x.
 207:     // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
 208:     return boost::math::pair<RealType, RealType>(dist.lower(), dist.upper());
 209:   }
 210: 
 211:   template <class RealType, class Policy>
 212:   BOOST_MATH_GPU_ENABLED RealType pdf(const triangular_distribution<RealType, Policy>& dist, const RealType& x)
 213:   {
 214:     constexpr auto function = "boost::math::pdf(const triangular_distribution<%1%>&, %1%)";
 215:     RealType lower = dist.lower();
 216:     RealType mode = dist.mode();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as lower, mode.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 lower, mode。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:     RealType upper = dist.upper();
 218:     RealType result = 0; // of checks.
 219:     if(false == detail::check_triangular(function, lower, mode, upper, &result, Policy()))
 220:     {
 221:       return result;
 222:     }
 223:     if(false == detail::check_triangular_x(function, x, &result, Policy()))
 224:     {
 225:       return result;
 226:     }
 227:     if((x < lower) || (x > upper))
 228:     {
 229:       return 0;
 230:     }
 231:     if (x == lower)
 232:     { // (mode - lower) == 0 which would lead to divide by zero!
 233:       return (mode == lower) ? 2 / (upper - lower) : RealType(0);
 234:     }
~~~
- **EN:** This range declares or defines callable logic such as upper. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 upper。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:     else if (x == upper)
 236:     {
 237:       return (mode == upper) ? 2 / (upper - lower) : RealType(0);
 238:     }
 239:     else if (x <= mode)
 240:     {
 241:       return 2 * (x - lower) / ((upper - lower) * (mode - lower));
 242:     }
 243:     else
 244:     {  // (x > mode)
 245:       return 2 * (upper - x) / ((upper - lower) * (upper - mode));
 246:     }
 247:   } // RealType pdf(const triangular_distribution<RealType, Policy>& dist, const RealType& x)
 248: 
 249:   template <class RealType, class Policy>
 250:   BOOST_MATH_GPU_ENABLED inline RealType cdf(const triangular_distribution<RealType, Policy>& dist, const RealType& x)
 251:   {
 252:     constexpr auto function = "boost::math::cdf(const triangular_distribution<%1%>&, %1%)";
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:     RealType lower = dist.lower();
 254:     RealType mode = dist.mode();
 255:     RealType upper = dist.upper();
 256:     RealType result = 0; // of checks.
 257:     if(false == detail::check_triangular(function, lower, mode, upper, &result, Policy()))
 258:     {
 259:       return result;
 260:     }
 261:     if(false == detail::check_triangular_x(function, x, &result, Policy()))
 262:     {
 263:       return result;
 264:     }
 265:     if((x <= lower))
 266:     {
 267:       return 0;
 268:     }
 269:     if (x >= upper)
 270:     {
~~~
- **EN:** This range declares or defines callable logic such as lower, mode, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lower, mode, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:       return 1;
 272:     }
 273:     // else lower < x < upper
 274:     if (x <= mode)
 275:     {
 276:       return ((x - lower) * (x - lower)) / ((upper - lower) * (mode - lower));
 277:     }
 278:     else
 279:     {
 280:       return 1 - (upper - x) *  (upper - x) / ((upper - lower) * (upper - mode));
 281:     }
 282:   } // RealType cdf(const triangular_distribution<RealType, Policy>& dist, const RealType& x)
 283: 
 284:   template <class RealType, class Policy>
 285:   BOOST_MATH_GPU_ENABLED RealType quantile(const triangular_distribution<RealType, Policy>& dist, const RealType& p)
 286:   {
 287:     BOOST_MATH_STD_USING  // for ADL of std functions (sqrt).
 288:     constexpr auto function = "boost::math::quantile(const triangular_distribution<%1%>&, %1%)";
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:     RealType lower = dist.lower();
 290:     RealType mode = dist.mode();
 291:     RealType upper = dist.upper();
 292:     RealType result = 0; // of checks
 293:     if(false == detail::check_triangular(function,lower, mode, upper, &result, Policy()))
 294:     {
 295:       return result;
 296:     }
 297:     if(false == detail::check_probability(function, p, &result, Policy()))
 298:     {
 299:       return result;
 300:     }
 301:     if(p == 0)
 302:     {
 303:       return lower;
 304:     }
 305:     if(p == 1)
 306:     {
~~~
- **EN:** This range declares or defines callable logic such as lower, mode, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lower, mode, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:       return upper;
 308:     }
 309:     RealType p0 = (mode - lower) / (upper - lower);
 310:     RealType q = 1 - p;
 311:     if (p < p0)
 312:     {
 313:       result = sqrt((upper - lower) * (mode - lower) * p) + lower;
 314:     }
 315:     else if (p == p0)
 316:     {
 317:       result = mode;
 318:     }
 319:     else // p > p0
 320:     {
 321:       result = upper - sqrt((upper - lower) * (upper - mode) * q);
 322:     }
 323:     return result;
 324: 
~~~
- **EN:** This range declares or defines callable logic such as sqrt. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:   } // RealType quantile(const triangular_distribution<RealType, Policy>& dist, const RealType& q)
 326: 
 327:   template <class RealType, class Policy>
 328:   BOOST_MATH_GPU_ENABLED RealType cdf(const complemented2_type<triangular_distribution<RealType, Policy>, RealType>& c)
 329:   {
 330:     constexpr auto function = "boost::math::cdf(const triangular_distribution<%1%>&, %1%)";
 331:     RealType lower = c.dist.lower();
 332:     RealType mode = c.dist.mode();
 333:     RealType upper = c.dist.upper();
 334:     RealType x = c.param;
 335:     RealType result = 0; // of checks.
 336:     if(false == detail::check_triangular(function, lower, mode, upper, &result, Policy()))
 337:     {
 338:       return result;
 339:     }
 340:     if(false == detail::check_triangular_x(function, x, &result, Policy()))
 341:     {
 342:       return result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as lower, mode, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 lower, mode, ...。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:     }
 344:     if (x <= lower)
 345:     {
 346:       return 1;
 347:     }
 348:     if (x >= upper)
 349:     {
 350:       return 0;
 351:     }
 352:     if (x <= mode)
 353:     {
 354:       return 1 - ((x - lower) * (x - lower)) / ((upper - lower) * (mode - lower));
 355:     }
 356:     else
 357:     {
 358:       return (upper - x) *  (upper - x) / ((upper - lower) * (upper - mode));
 359:     }
 360:   } // RealType cdf(const complemented2_type<triangular_distribution<RealType, Policy>, RealType>& c)
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361: 
 362:   template <class RealType, class Policy>
 363:   BOOST_MATH_GPU_ENABLED RealType quantile(const complemented2_type<triangular_distribution<RealType, Policy>, RealType>& c)
 364:   {
 365:     BOOST_MATH_STD_USING  // Aid ADL for sqrt.
 366:     constexpr auto function = "boost::math::quantile(const triangular_distribution<%1%>&, %1%)";
 367:     RealType l = c.dist.lower();
 368:     RealType m = c.dist.mode();
 369:     RealType u = c.dist.upper();
 370:     RealType q = c.param; // probability 0 to 1.
 371:     RealType result = 0; // of checks.
 372:     if(false == detail::check_triangular(function, l, m, u, &result, Policy()))
 373:     {
 374:       return result;
 375:     }
 376:     if(false == detail::check_probability(function, q, &result, Policy()))
 377:     {
 378:       return result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as lower, mode, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 lower, mode, ...。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:     }
 380:     if(q == 0)
 381:     {
 382:       return u;
 383:     }
 384:     if(q == 1)
 385:     {
 386:       return l;
 387:     }
 388:     RealType lower = c.dist.lower();
 389:     RealType mode = c.dist.mode();
 390:     RealType upper = c.dist.upper();
 391: 
 392:     RealType p = 1 - q;
 393:     RealType p0 = (mode - lower) / (upper - lower);
 394:     if(p < p0)
 395:     {
 396:       RealType s = (upper - lower) * (mode - lower);
~~~
- **EN:** This range declares or defines callable logic such as lower, mode, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lower, mode, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:       s *= p;
 398:       result = sqrt((upper - lower) * (mode - lower) * p) + lower;
 399:     }
 400:     else if (p == p0)
 401:     {
 402:       result = mode;
 403:     }
 404:     else // p > p0
 405:     {
 406:       result = upper - sqrt((upper - lower) * (upper - mode) * q);
 407:     }
 408:     return result;
 409:   } // RealType quantile(const complemented2_type<triangular_distribution<RealType, Policy>, RealType>& c)
 410: 
 411:   template <class RealType, class Policy>
 412:   BOOST_MATH_GPU_ENABLED inline RealType mean(const triangular_distribution<RealType, Policy>& dist)
 413:   {
 414:     constexpr auto function = "boost::math::mean(const triangular_distribution<%1%>&)";
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as sqrt.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 sqrt。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:     RealType lower = dist.lower();
 416:     RealType mode = dist.mode();
 417:     RealType upper = dist.upper();
 418:     RealType result = 0;  // of checks.
 419:     if(false == detail::check_triangular(function, lower, mode, upper, &result, Policy()))
 420:     {
 421:       return result;
 422:     }
 423:     return (lower + upper + mode) / 3;
 424:   } // RealType mean(const triangular_distribution<RealType, Policy>& dist)
 425: 
 426: 
 427:   template <class RealType, class Policy>
 428:   BOOST_MATH_GPU_ENABLED inline RealType variance(const triangular_distribution<RealType, Policy>& dist)
 429:   {
 430:     constexpr auto function = "boost::math::mean(const triangular_distribution<%1%>&)";
 431:     RealType lower = dist.lower();
 432:     RealType mode = dist.mode();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as lower, mode, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 lower, mode, ...。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:     RealType upper = dist.upper();
 434:     RealType result = 0; // of checks.
 435:     if(false == detail::check_triangular(function, lower, mode, upper, &result, Policy()))
 436:     {
 437:       return result;
 438:     }
 439:     return (lower * lower + upper * upper + mode * mode - lower * upper - lower * mode - upper * mode) / 18;
 440:   } // RealType variance(const triangular_distribution<RealType, Policy>& dist)
 441: 
 442:   template <class RealType, class Policy>
 443:   BOOST_MATH_GPU_ENABLED inline RealType mode(const triangular_distribution<RealType, Policy>& dist)
 444:   {
 445:     constexpr auto function = "boost::math::mode(const triangular_distribution<%1%>&)";
 446:     RealType mode = dist.mode();
 447:     RealType result = 0; // of checks.
 448:     if(false == detail::check_triangular_mode(function, mode, &result, Policy()))
 449:     { // This should never happen!
 450:       return result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as upper, mode.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 upper, mode。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:     }
 452:     return mode;
 453:   } // RealType mode
 454: 
 455:   template <class RealType, class Policy>
 456:   BOOST_MATH_GPU_ENABLED inline RealType median(const triangular_distribution<RealType, Policy>& dist)
 457:   {
 458:     BOOST_MATH_STD_USING // ADL of std functions.
 459:     constexpr auto function = "boost::math::median(const triangular_distribution<%1%>&)";
 460:     RealType mode = dist.mode();
 461:     RealType result = 0; // of checks.
 462:     if(false == detail::check_triangular_mode(function, mode, &result, Policy()))
 463:     { // This should never happen!
 464:       return result;
 465:     }
 466:     RealType lower = dist.lower();
 467:     RealType upper = dist.upper();
 468:     if (mode >= (upper + lower) / 2)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as mode, lower, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 mode, lower, ...。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:     {
 470:       return lower + sqrt((upper - lower) * (mode - lower)) / constants::root_two<RealType>();
 471:     }
 472:     else
 473:     {
 474:       return upper - sqrt((upper - lower) * (upper - mode)) / constants::root_two<RealType>();
 475:     }
 476:   } // RealType mode
 477: 
 478:   template <class RealType, class Policy>
 479:   BOOST_MATH_GPU_ENABLED inline RealType skewness(const triangular_distribution<RealType, Policy>& dist)
 480:   {
 481:     BOOST_MATH_STD_USING  // for ADL of std functions
 482:     using namespace boost::math::constants; // for root_two
 483:     constexpr auto function = "boost::math::skewness(const triangular_distribution<%1%>&)";
 484: 
 485:     RealType lower = dist.lower();
 486:     RealType mode = dist.mode();
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:     RealType upper = dist.upper();
 488:     RealType result = 0; // of checks.
 489:     if(false == boost::math::detail::check_triangular(function,lower, mode, upper, &result, Policy()))
 490:     {
 491:       return result;
 492:     }
 493:     return root_two<RealType>() * (lower + upper - 2 * mode) * (2 * lower - upper - mode) * (lower - 2 * upper + mode) /
 494:       (5 * pow((lower * lower + upper * upper + mode * mode
 495:         - lower * upper - lower * mode - upper * mode), RealType(3)/RealType(2)));
 496:     // #11768: Skewness formula for triangular distribution is incorrect -  corrected 29 Oct 2015 for release 1.61.
 497:   } // RealType skewness(const triangular_distribution<RealType, Policy>& dist)
 498: 
 499:   template <class RealType, class Policy>
 500:   BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const triangular_distribution<RealType, Policy>& dist)
 501:   { // These checks may be belt and braces as should have been checked on construction?
 502:     constexpr auto function = "boost::math::kurtosis(const triangular_distribution<%1%>&)";
 503:     RealType lower = dist.lower();
 504:     RealType upper = dist.upper();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as upper, RealType, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 upper, RealType, ...。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:     RealType mode = dist.mode();
 506:     RealType result = 0;  // of checks.
 507:     if(false == detail::check_triangular(function,lower, mode, upper, &result, Policy()))
 508:     {
 509:       return result;
 510:     }
 511:     return static_cast<RealType>(12)/5; //  12/5 = 2.4;
 512:   } // RealType kurtosis_excess(const triangular_distribution<RealType, Policy>& dist)
 513: 
 514:   template <class RealType, class Policy>
 515:   BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const triangular_distribution<RealType, Policy>& dist)
 516:   { // These checks may be belt and braces as should have been checked on construction?
 517:     constexpr auto function = "boost::math::kurtosis_excess(const triangular_distribution<%1%>&)";
 518:     RealType lower = dist.lower();
 519:     RealType upper = dist.upper();
 520:     RealType mode = dist.mode();
 521:     RealType result = 0;  // of checks.
 522:     if(false == detail::check_triangular(function,lower, mode, upper, &result, Policy()))
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as mode, lower, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 mode, lower, ...。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:     {
 524:       return result;
 525:     }
 526:     return static_cast<RealType>(-3)/5; // - 3/5 = -0.6
 527:     // Assuming mathworld really means kurtosis excess?  Wikipedia now corrected to match this.
 528:   }
 529: 
 530:   template <class RealType, class Policy>
 531:   BOOST_MATH_GPU_ENABLED inline RealType entropy(const triangular_distribution<RealType, Policy>& dist)
 532:   {
 533:     BOOST_MATH_STD_USING
 534:     return constants::half<RealType>() + log((dist.upper() - dist.lower())/2);
 535:   }
 536: 
 537: } // namespace math
 538: } // namespace boost
 539: 
 540: // This include must be at the end, *after* the accessors
~~~
- **EN:** The code enters namespace scope (math::boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

### Lines 541-548 / 第 541-548 行
~~~cpp
 541: // for this distribution have been defined, in order to
 542: // keep compilers that support two-phase lookup happy.
 543: #include <boost/math/distributions/detail/derived_accessors.hpp>
 544: 
 545: #endif // BOOST_STATS_TRIANGULAR_HPP
 546: 
 547: 
 548: 
~~~
- **EN:** This block imports dependencies such as boost/math/distributions/detail/derived_accessors.hpp so the surrounding code can use external declarations. This short range closes a conditional-compilation or header-guard region. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/distributions/detail/derived_accessors.hpp 等依赖，使周围代码可以使用外部声明。 这一小段用于结束条件编译区域或头文件保护区域。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/tuple.hpp, boost/math/tools/promotion.hpp, boost/math/distributions/fwd.hpp, boost/math/special_functions/expm1.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/complement.hpp, boost/math/constants/constants.hpp, boost/math/distributions/detail/derived_accessors.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `detail::check_triangular, lower, mode, upper, sqrt, RealType`
