# bessel_i1.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/bessel_i1.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the bessel i1 special-function path.
- **作用（中文）**: 此头文件为 bessel i1 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  Copyright (c) 2017 John Maddock
   2: //  Copyright (c) 2024 Matt Borland
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: // Modified Bessel function of the first kind of order zero
   8: // we use the approximating forms derived in:
   9: // "Rational Approximations for the Modified Bessel Function of the First Kind - I1(x) for Computations with Double Precision"
  10: // by Pavel Holoborodko,
  11: // see http://www.advanpix.com/2015/11/12/rational-approximations-for-the-modified-bessel-function-of-the-first-kind-i1-for-computations-with-double-precision/
  12: // The actual coefficients used are our own, and extend Pavel's work to precision's other than double.
  13: 
  14: #ifndef BOOST_MATH_BESSEL_I1_HPP
  15: #define BOOST_MATH_BESSEL_I1_HPP
  16: 
  17: #ifdef _MSC_VER
  18: #pragma once
~~~
- **EN:** It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #endif
  20: 
  21: #include <boost/math/tools/config.hpp>
  22: #include <boost/math/tools/rational.hpp>
  23: #include <boost/math/tools/big_constant.hpp>
  24: #include <boost/math/tools/assert.hpp>
  25: #include <boost/math/tools/type_traits.hpp>
  26: #include <boost/math/tools/numeric_limits.hpp>
  27: #include <boost/math/tools/precision.hpp>
  28: 
  29: #if defined(__GNUC__) && defined(BOOST_MATH_USE_FLOAT128)
  30: //
  31: // This is the only way we can avoid
  32: // warning: non-standard suffix on floating constant [-Wpedantic]
  33: // when building with -Wall -pedantic.  Neither __extension__
  34: // nor #pragma diagnostic ignored work :(
  35: //
  36: #pragma GCC system_header
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/rational.hpp, boost/math/tools/big_constant.hpp, ... so the surrounding code can use external declarations. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/rational.hpp, boost/math/tools/big_constant.hpp, ... 等依赖，使周围代码可以使用外部声明。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: #endif
  38: 
  39: // Modified Bessel function of the first kind of order one
  40: // minimax rational approximations on intervals, see
  41: // Blair and Edwards, Chalk River Report AECL-4928, 1974
  42: 
  43: namespace boost { namespace math { namespace detail{
  44: 
  45: template <typename T>
  46: BOOST_MATH_GPU_ENABLED T bessel_i1(const T& x);
  47: 
  48: template <typename T, int N>
  49: BOOST_MATH_GPU_ENABLED T bessel_i1_imp(const T&, const boost::math::integral_constant<int, N>&)
  50: {
  51:    BOOST_MATH_ASSERT(0);
  52:    return 0;
  53: }
  54: 
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as bessel_i1, BOOST_MATH_ASSERT.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 bessel_i1, BOOST_MATH_ASSERT。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: template <typename T>
  56: BOOST_MATH_GPU_ENABLED T bessel_i1_imp(const T& x, const boost::math::integral_constant<int, 24>&)
  57: {
  58:    BOOST_MATH_STD_USING
  59:       if(x < 7.75)
  60:       {
  61:          //Max error in interpolated form : 1.348e-08
  62:          // Max Error found at float precision = Poly : 1.469121e-07
  63:          BOOST_MATH_STATIC const float P[] = {
  64:             8.333333221e-02f,
  65:             6.944453712e-03f,
  66:             3.472097211e-04f,
  67:             1.158047174e-05f,
  68:             2.739745142e-07f,
  69:             5.135884609e-09f,
  70:             5.262251502e-11f,
  71:             1.331933703e-12f
  72:          };
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:          T a = x * x / 4;
  74:          T Q[3] = { 1, 0.5f, boost::math::tools::evaluate_polynomial(P, a) };
  75:          return x * boost::math::tools::evaluate_polynomial(Q, a) / 2;
  76:       }
  77:       else
  78:       {
  79:          // Max error in interpolated form: 9.000e-08
  80:          // Max Error found at float precision = Poly: 1.044345e-07
  81: 
  82:          BOOST_MATH_STATIC const float P[] = {
  83:             3.98942115977513013e-01f,
  84:             -1.49581264836620262e-01f,
  85:             -4.76475741878486795e-02f,
  86:             -2.65157315524784407e-02f,
  87:             -1.47148600683672014e-01f
  88:          };
  89:          T ex = exp(x / 2);
  90:          T result = ex * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
~~~
- **EN:** This range declares or defines callable logic such as exp, boost::math::tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp, boost::math::tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:          result *= ex;
  92:          return result;
  93:       }
  94: }
  95: 
  96: template <typename T>
  97: BOOST_MATH_GPU_ENABLED T bessel_i1_imp(const T& x, const boost::math::integral_constant<int, 53>&)
  98: {
  99:    BOOST_MATH_STD_USING
 100:    if(x < 7.75)
 101:    {
 102:       // Bessel I0 over[10 ^ -16, 7.75]
 103:       // Max error in interpolated form: 5.639e-17
 104:       // Max Error found at double precision = Poly: 1.795559e-16
 105: 
 106:       BOOST_MATH_STATIC const double P[] = {
 107:          8.333333333333333803e-02,
 108:          6.944444444444341983e-03,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:          3.472222222225921045e-04,
 110:          1.157407407354987232e-05,
 111:          2.755731926254790268e-07,
 112:          4.920949692800671435e-09,
 113:          6.834657311305621830e-11,
 114:          7.593969849687574339e-13,
 115:          6.904822652741917551e-15,
 116:          5.220157095351373194e-17,
 117:          3.410720494727771276e-19,
 118:          1.625212890947171108e-21,
 119:          1.332898928162290861e-23
 120:       };
 121:       T a = x * x / 4;
 122:       T Q[3] = { 1, 0.5f, boost::math::tools::evaluate_polynomial(P, a) };
 123:       return x * boost::math::tools::evaluate_polynomial(Q, a) / 2;
 124:    }
 125:    else if(x < 500)
 126:    {
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:       // Max error in interpolated form: 1.796e-16
 128:       // Max Error found at double precision = Poly: 2.898731e-16
 129: 
 130:       BOOST_MATH_STATIC const double P[] = {
 131:          3.989422804014406054e-01,
 132:          -1.496033551613111533e-01,
 133:          -4.675104253598537322e-02,
 134:          -4.090895951581637791e-02,
 135:          -5.719036414430205390e-02,
 136:          -1.528189554374492735e-01,
 137:          3.458284470977172076e+00,
 138:          -2.426181371595021021e+02,
 139:          1.178785865993440669e+04,
 140:          -4.404655582443487334e+05,
 141:          1.277677779341446497e+07,
 142:          -2.903390398236656519e+08,
 143:          5.192386898222206474e+09,
 144:          -7.313784438967834057e+10,
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:          8.087824484994859552e+11,
 146:          -6.967602516005787001e+12,
 147:          4.614040809616582764e+13,
 148:          -2.298849639457172489e+14,
 149:          8.325554073334618015e+14,
 150:          -2.067285045778906105e+15,
 151:          3.146401654361325073e+15,
 152:          -2.213318202179221945e+15
 153:       };
 154:       return exp(x) * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 155:    }
 156:    else
 157:    {
 158:       // Max error in interpolated form: 1.320e-19
 159:       // Max Error found at double precision = Poly: 7.065357e-17
 160:       BOOST_MATH_STATIC const double P[] = {
 161:          3.989422804014314820e-01,
 162:          -1.496033551467584157e-01,
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:          -4.675105322571775911e-02,
 164:          -4.090421597376992892e-02,
 165:          -5.843630344778927582e-02
 166:       };
 167:       T ex = exp(x / 2);
 168:       T result = ex * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 169:       result *= ex;
 170:       return result;
 171:    }
 172: }
 173: 
 174: template <typename T>
 175: BOOST_MATH_GPU_ENABLED T bessel_i1_imp(const T& x, const boost::math::integral_constant<int, 64>&)
 176: {
 177:    BOOST_MATH_STD_USING
 178:       if(x < 7.75)
 179:       {
 180:          // Bessel I0 over[10 ^ -16, 7.75]
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as exp, boost::math::tools::evaluate_polynomial. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 exp, boost::math::tools::evaluate_polynomial。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:          // Max error in interpolated form: 8.086e-21
 182:          // Max Error found at float80 precision = Poly: 7.225090e-20
 183:          BOOST_MATH_STATIC const T P[] = {
 184:             BOOST_MATH_BIG_CONSTANT(T, 64, 8.33333333333333333340071817e-02),
 185:             BOOST_MATH_BIG_CONSTANT(T, 64, 6.94444444444444442462728070e-03),
 186:             BOOST_MATH_BIG_CONSTANT(T, 64, 3.47222222222222318886683883e-04),
 187:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.15740740740738880709555060e-05),
 188:             BOOST_MATH_BIG_CONSTANT(T, 64, 2.75573192240046222242685145e-07),
 189:             BOOST_MATH_BIG_CONSTANT(T, 64, 4.92094986131253986838697503e-09),
 190:             BOOST_MATH_BIG_CONSTANT(T, 64, 6.83465258979924922633502182e-11),
 191:             BOOST_MATH_BIG_CONSTANT(T, 64, 7.59405830675154933645967137e-13),
 192:             BOOST_MATH_BIG_CONSTANT(T, 64, 6.90369179710633344508897178e-15),
 193:             BOOST_MATH_BIG_CONSTANT(T, 64, 5.23003610041709452814262671e-17),
 194:             BOOST_MATH_BIG_CONSTANT(T, 64, 3.35291901027762552549170038e-19),
 195:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.83991379419781823063672109e-21),
 196:             BOOST_MATH_BIG_CONSTANT(T, 64, 8.87732714140192556332037815e-24),
 197:             BOOST_MATH_BIG_CONSTANT(T, 64, 3.32120654663773147206454247e-26),
 198:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.95294659305369207813486871e-28)
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:          };
 200:          T a = x * x / 4;
 201:          T Q[3] = { 1, 0.5f, boost::math::tools::evaluate_polynomial(P, a) };
 202:          return x * boost::math::tools::evaluate_polynomial(Q, a) / 2;
 203:       }
 204:       else if(x < 20)
 205:       {
 206:          // Max error in interpolated form: 4.258e-20
 207:          // Max Error found at float80 precision = Poly: 2.851105e-19
 208:          // Maximum Deviation Found : 3.887e-20
 209:          // Expected Error Term : 3.887e-20
 210:          // Maximum Relative Change in Control Points : 1.681e-04
 211:          BOOST_MATH_STATIC const T P[] = {
 212:             BOOST_MATH_BIG_CONSTANT(T, 64, 3.98942260530218897338680e-01),
 213:             BOOST_MATH_BIG_CONSTANT(T, 64, -1.49599542849073670179540e-01),
 214:             BOOST_MATH_BIG_CONSTANT(T, 64, -4.70492865454119188276875e-02),
 215:             BOOST_MATH_BIG_CONSTANT(T, 64, -3.12389893307392002405869e-02),
 216:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.49696126385202602071197e-01),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:             BOOST_MATH_BIG_CONSTANT(T, 64, -3.84206507612717711565967e+01),
 218:             BOOST_MATH_BIG_CONSTANT(T, 64, 2.14748094784412558689584e+03),
 219:             BOOST_MATH_BIG_CONSTANT(T, 64, -7.70652726663596993005669e+04),
 220:             BOOST_MATH_BIG_CONSTANT(T, 64, 2.01659736164815617174439e+06),
 221:             BOOST_MATH_BIG_CONSTANT(T, 64, -4.04740659606466305607544e+07),
 222:             BOOST_MATH_BIG_CONSTANT(T, 64, 6.38383394696382837263656e+08),
 223:             BOOST_MATH_BIG_CONSTANT(T, 64, -8.00779638649147623107378e+09),
 224:             BOOST_MATH_BIG_CONSTANT(T, 64, 8.02338237858684714480491e+10),
 225:             BOOST_MATH_BIG_CONSTANT(T, 64, -6.41198553664947312995879e+11),
 226:             BOOST_MATH_BIG_CONSTANT(T, 64, 4.05915186909564986897554e+12),
 227:             BOOST_MATH_BIG_CONSTANT(T, 64, -2.00907636964168581116181e+13),
 228:             BOOST_MATH_BIG_CONSTANT(T, 64, 7.60855263982359981275199e+13),
 229:             BOOST_MATH_BIG_CONSTANT(T, 64, -2.12901817219239205393806e+14),
 230:             BOOST_MATH_BIG_CONSTANT(T, 64, 4.14861794397709807823575e+14),
 231:             BOOST_MATH_BIG_CONSTANT(T, 64, -5.02808138522587680348583e+14),
 232:             BOOST_MATH_BIG_CONSTANT(T, 64, 2.85505477056514919387171e+14)
 233:          };
 234:          return exp(x) * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:       }
 236:       else if(x < 100)
 237:       {
 238:          // Bessel I0 over [15, 50]
 239:          // Maximum Deviation Found:                     2.444e-20
 240:          // Expected Error Term : 2.438e-20
 241:          // Maximum Relative Change in Control Points : 2.101e-03
 242:          // Max Error found at float80 precision = Poly : 6.029974e-20
 243: 
 244:          BOOST_MATH_STATIC const T P[] = {
 245:             BOOST_MATH_BIG_CONSTANT(T, 64, 3.98942280401431675205845e-01),
 246:             BOOST_MATH_BIG_CONSTANT(T, 64, -1.49603355149968887210170e-01),
 247:             BOOST_MATH_BIG_CONSTANT(T, 64, -4.67510486284376330257260e-02),
 248:             BOOST_MATH_BIG_CONSTANT(T, 64, -4.09071458907089270559464e-02),
 249:             BOOST_MATH_BIG_CONSTANT(T, 64, -5.75278280327696940044714e-02),
 250:             BOOST_MATH_BIG_CONSTANT(T, 64, -1.10591299500956620739254e-01),
 251:             BOOST_MATH_BIG_CONSTANT(T, 64, -2.77061766699949309115618e-01),
 252:             BOOST_MATH_BIG_CONSTANT(T, 64, -5.42683771801837596371638e-01),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:             BOOST_MATH_BIG_CONSTANT(T, 64, -9.17021412070404158464316e+00),
 254:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.04154379346763380543310e+02),
 255:             BOOST_MATH_BIG_CONSTANT(T, 64, -1.43462345357478348323006e+03),
 256:             BOOST_MATH_BIG_CONSTANT(T, 64, 9.98109660274422449523837e+03),
 257:             BOOST_MATH_BIG_CONSTANT(T, 64, -3.74438822767781410362757e+04)
 258:          };
 259:          return exp(x) * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 260:       }
 261:       else
 262:       {
 263:          // Bessel I0 over[100, INF]
 264:          // Max error in interpolated form: 2.456e-20
 265:          // Max Error found at float80 precision = Poly: 5.446356e-20
 266:          BOOST_MATH_STATIC const T P[] = {
 267:             BOOST_MATH_BIG_CONSTANT(T, 64, 3.98942280401432677958445e-01),
 268:             BOOST_MATH_BIG_CONSTANT(T, 64, -1.49603355150537411254359e-01),
 269:             BOOST_MATH_BIG_CONSTANT(T, 64, -4.67510484842456251368526e-02),
 270:             BOOST_MATH_BIG_CONSTANT(T, 64, -4.09071676503922479645155e-02),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:             BOOST_MATH_BIG_CONSTANT(T, 64, -5.75256179814881566010606e-02),
 272:             BOOST_MATH_BIG_CONSTANT(T, 64, -1.10754910257965227825040e-01),
 273:             BOOST_MATH_BIG_CONSTANT(T, 64, -2.67858639515616079840294e-01),
 274:             BOOST_MATH_BIG_CONSTANT(T, 64, -9.17266479586791298924367e-01)
 275:          };
 276:          T ex = exp(x / 2);
 277:          T result = ex * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 278:          result *= ex;
 279:          return result;
 280:       }
 281: }
 282: 
 283: template <typename T>
 284: BOOST_MATH_GPU_ENABLED T bessel_i1_imp(const T& x, const boost::math::integral_constant<int, 113>&)
 285: {
 286:    BOOST_MATH_STD_USING
 287:    if(x < 7.75)
 288:    {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as exp, boost::math::tools::evaluate_polynomial. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 exp, boost::math::tools::evaluate_polynomial。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:       // Bessel I0 over[10 ^ -34, 7.75]
 290:       // Max error in interpolated form: 1.835e-35
 291:       // Max Error found at float128 precision = Poly: 1.645036e-34
 292: 
 293:       BOOST_MATH_STATIC const T P[] = {
 294:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.3333333333333333333333333333333331804098e-02),
 295:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.9444444444444444444444444444445418303082e-03),
 296:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.4722222222222222222222222222119082346591e-04),
 297:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.1574074074074074074074074078415867655987e-05),
 298:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.7557319223985890652557318255143448192453e-07),
 299:          BOOST_MATH_BIG_CONSTANT(T, 113, 4.9209498614260519022423916850415000626427e-09),
 300:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.8346525853139609753354247043900442393686e-11),
 301:          BOOST_MATH_BIG_CONSTANT(T, 113, 7.5940584281266233060080535940234144302217e-13),
 302:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.9036894801151120925605467963949641957095e-15),
 303:          BOOST_MATH_BIG_CONSTANT(T, 113, 5.2300677879659941472662086395055636394839e-17),
 304:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.3526075563884539394691458717439115962233e-19),
 305:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.8420920639497841692288943167036233338434e-21),
 306:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.7718669711748690065381181691546032291365e-24),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.6549445715236427401845636880769861424730e-26),
 308:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.3437296196812697924703896979250126739676e-28),
 309:          BOOST_MATH_BIG_CONSTANT(T, 113, 4.3912734588619073883015937023564978854893e-31),
 310:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.2839967682792395867255384448052781306897e-33),
 311:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.3790094235693528861015312806394354114982e-36),
 312:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.0423861671932104308662362292359563970482e-39),
 313:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.7493858979396446292135661268130281652945e-41),
 314:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.2786079392547776769387921361408303035537e-44),
 315:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.2335693685833531118863552173880047183822e-47)
 316:       };
 317:       T a = x * x / 4;
 318:       T Q[3] = { 1, 0.5f, boost::math::tools::evaluate_polynomial(P, a) };
 319:       return x * boost::math::tools::evaluate_polynomial(Q, a) / 2;
 320:    }
 321:    else if(x < 11)
 322:    {
 323:       // Max error in interpolated form: 8.574e-36
 324:       // Maximum Deviation Found : 4.689e-36
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:       // Expected Error Term : 3.760e-36
 326:       // Maximum Relative Change in Control Points : 5.204e-03
 327:       // Max Error found at float128 precision = Poly : 2.882561e-34
 328: 
 329:       BOOST_MATH_STATIC const T P[] = {
 330:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.333333333333333326889717360850080939e-02),
 331:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.944444444444444511272790848815114507e-03),
 332:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.472222222222221892451965054394153443e-04),
 333:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.157407407407408437378868534321538798e-05),
 334:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.755731922398566216824909767320161880e-07),
 335:          BOOST_MATH_BIG_CONSTANT(T, 113, 4.920949861426434829568192525456800388e-09),
 336:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.834652585308926245465686943255486934e-11),
 337:          BOOST_MATH_BIG_CONSTANT(T, 113, 7.594058428179852047689599244015979196e-13),
 338:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.903689479655006062822949671528763738e-15),
 339:          BOOST_MATH_BIG_CONSTANT(T, 113, 5.230067791254403974475987777406992984e-17),
 340:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.352607536815161679702105115200693346e-19),
 341:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.842092161364672561828681848278567885e-21),
 342:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.771862912600611801856514076709932773e-24),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.654958704184380914803366733193713605e-26),
 344:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.343688672071130980471207297730607625e-28),
 345:          BOOST_MATH_BIG_CONSTANT(T, 113, 4.392252844664709532905868749753463950e-31),
 346:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.282086786672692641959912811902298600e-33),
 347:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.408812012322547015191398229942864809e-36),
 348:          BOOST_MATH_BIG_CONSTANT(T, 113, 7.681220437734066258673404589233009892e-39),
 349:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.072417451640733785626701738789290055e-41),
 350:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.352218520142636864158849446833681038e-44),
 351:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.407918492276267527897751358794783640e-46)
 352:       };
 353:       T a = x * x / 4;
 354:       T Q[3] = { 1, 0.5f, boost::math::tools::evaluate_polynomial(P, a) };
 355:       return x * boost::math::tools::evaluate_polynomial(Q, a) / 2;
 356:    }
 357:    else if(x < 15)
 358:    {
 359:       //Max error in interpolated form: 7.599e-36
 360:       // Maximum Deviation Found : 1.766e-35
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:       // Expected Error Term : 1.021e-35
 362:       // Maximum Relative Change in Control Points : 6.228e-03
 363:       BOOST_MATH_STATIC const T P[] = {
 364:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.333333333333255774414858563409941233e-02),
 365:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.944444444444897867884955912228700291e-03),
 366:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.472222222220954970397343617150959467e-04),
 367:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.157407407409660682751155024932538578e-05),
 368:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.755731922369973706427272809014190998e-07),
 369:          BOOST_MATH_BIG_CONSTANT(T, 113, 4.920949861702265600960449699129258153e-09),
 370:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.834652583208361401197752793379677147e-11),
 371:          BOOST_MATH_BIG_CONSTANT(T, 113, 7.594058441128280500819776168239988143e-13),
 372:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.903689413939268702265479276217647209e-15),
 373:          BOOST_MATH_BIG_CONSTANT(T, 113, 5.230068069012898202890718644753625569e-17),
 374:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.352606552027491657204243201021677257e-19),
 375:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.842095100698532984651921750204843362e-21),
 376:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.771789051329870174925649852681844169e-24),
 377:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.655114381199979536997025497438385062e-26),
 378:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.343415732516712339472538688374589373e-28),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:          BOOST_MATH_BIG_CONSTANT(T, 113, 4.396177019032432392793591204647901390e-31),
 380:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.277563309255167951005939802771456315e-33),
 381:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.449201419305514579791370198046544736e-36),
 382:          BOOST_MATH_BIG_CONSTANT(T, 113, 7.415430703400740634202379012388035255e-39),
 383:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.195458831864936225409005027914934499e-41),
 384:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.829726762743879793396637797534668039e-45),
 385:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.698302711685624490806751012380215488e-46),
 386:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.062520475425422618494185821587228317e-49),
 387:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.732372906742845717148185173723304360e-52)
 388:       };
 389:       T a = x * x / 4;
 390:       T Q[3] = { 1, 0.5f, boost::math::tools::evaluate_polynomial(P, a) };
 391:       return x * boost::math::tools::evaluate_polynomial(Q, a) / 2;
 392:    }
 393:    else if(x < 20)
 394:    {
 395:       // Max error in interpolated form: 8.864e-36
 396:       // Max Error found at float128 precision = Poly: 8.522841e-35
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:       BOOST_MATH_STATIC const T P[] = {
 398:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.989422793693152031514179994954750043e-01),
 399:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.496029423752889591425633234009799670e-01),
 400:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.682975926820553021482820043377990241e-02),
 401:          BOOST_MATH_BIG_CONSTANT(T, 113, -3.138871171577224532369979905856458929e-02),
 402:          BOOST_MATH_BIG_CONSTANT(T, 113, -8.765350219426341341990447005798111212e-01),
 403:          BOOST_MATH_BIG_CONSTANT(T, 113, 5.321389275507714530941178258122955540e+01),
 404:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.727748393898888756515271847678850411e+03),
 405:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.123040820686242586086564998713862335e+05),
 406:          BOOST_MATH_BIG_CONSTANT(T, 113, -3.784112378374753535335272752884808068e+06),
 407:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.054920416060932189433079126269416563e+08),
 408:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.450129415468060676827180524327749553e+09),
 409:          BOOST_MATH_BIG_CONSTANT(T, 113, 4.758831882046487398739784498047935515e+10),
 410:          BOOST_MATH_BIG_CONSTANT(T, 113, -7.736936520262204842199620784338052937e+11),
 411:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.051128683324042629513978256179115439e+13),
 412:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.188008285959794869092624343537262342e+14),
 413:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.108530004906954627420484180793165669e+15),
 414:          BOOST_MATH_BIG_CONSTANT(T, 113, -8.441516828490144766650287123765318484e+15),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:          BOOST_MATH_BIG_CONSTANT(T, 113, 5.158251664797753450664499268756393535e+16),
 416:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.467314522709016832128790443932896401e+17),
 417:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.896222045367960462945885220710294075e+17),
 418:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.273382139594876997203657902425653079e+18),
 419:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.669871448568623680543943144842394531e+18),
 420:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.813923031370708069940575240509912588e+18)
 421:       };
 422:       return exp(x) * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 423:    }
 424:    else if(x < 35)
 425:    {
 426:       // Max error in interpolated form: 6.028e-35
 427:       // Max Error found at float128 precision = Poly: 1.368313e-34
 428: 
 429:       BOOST_MATH_STATIC const T P[] = {
 430:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.989422804012941975429616956496046931e-01),
 431:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.496033550576049830976679315420681402e-01),
 432:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.675107835141866009896710750800622147e-02),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.090104965125365961928716504473692957e-02),
 434:          BOOST_MATH_BIG_CONSTANT(T, 113, -5.842241652296980863361375208605487570e-02),
 435:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.063604828033747303936724279018650633e-02),
 436:          BOOST_MATH_BIG_CONSTANT(T, 113, -9.113375972811586130949401996332817152e+00),
 437:          BOOST_MATH_BIG_CONSTANT(T, 113, 6.334748570425075872639817839399823709e+02),
 438:          BOOST_MATH_BIG_CONSTANT(T, 113, -3.759150758768733692594821032784124765e+04),
 439:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.863672813448915255286274382558526321e+06),
 440:          BOOST_MATH_BIG_CONSTANT(T, 113, -7.798248643371718775489178767529282534e+07),
 441:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.769963173932801026451013022000669267e+09),
 442:          BOOST_MATH_BIG_CONSTANT(T, 113, -8.381780137198278741566746511015220011e+10),
 443:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.163891337116820832871382141011952931e+12),
 444:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.764325864671438675151635117936912390e+13),
 445:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.925668307403332887856809510525154955e+14),
 446:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.416692606589060039334938090985713641e+16),
 447:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.892398600219306424294729851605944429e+17),
 448:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.107232903741874160308537145391245060e+18),
 449:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.930223393531877588898224144054112045e+19),
 450:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.427759576167665663373350433236061007e+20),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:          BOOST_MATH_BIG_CONSTANT(T, 113, 8.306019279465532835530812122374386654e+20),
 452:          BOOST_MATH_BIG_CONSTANT(T, 113, -3.653753000392125229440044977239174472e+21),
 453:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.140760686989511568435076842569804906e+22),
 454:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.249149337812510200795436107962504749e+22),
 455:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.101619088427348382058085685849420866e+22)
 456:       };
 457:       return exp(x) * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 458:    }
 459:    else if(x < 100)
 460:    {
 461:       // Max error in interpolated form: 5.494e-35
 462:       // Max Error found at float128 precision = Poly: 1.214651e-34
 463: 
 464:       BOOST_MATH_STATIC const T P[] = {
 465:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.989422804014326779399307367861631577e-01),
 466:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.496033551505372542086590873271571919e-01),
 467:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.675104848454290286276466276677172664e-02),
 468:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.090716742397105403027549796269213215e-02),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:          BOOST_MATH_BIG_CONSTANT(T, 113, -5.752570419098513588311026680089351230e-02),
 470:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.107369803696534592906420980901195808e-01),
 471:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.699214194000085622941721628134575121e-01),
 472:          BOOST_MATH_BIG_CONSTANT(T, 113, -7.953006169077813678478720427604462133e-01),
 473:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.746618809476524091493444128605380593e+00),
 474:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.084446249943196826652788161656973391e+01),
 475:          BOOST_MATH_BIG_CONSTANT(T, 113, -5.020325182518980633783194648285500554e+01),
 476:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.510195971266257573425196228564489134e+02),
 477:          BOOST_MATH_BIG_CONSTANT(T, 113, -5.241661863814900938075696173192225056e+03),
 478:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.323374362891993686413568398575539777e+05),
 479:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.112838452096066633754042734723911040e+06),
 480:          BOOST_MATH_BIG_CONSTANT(T, 113, 9.369270194978310081563767560113534023e+07),
 481:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.704295412488936504389347368131134993e+09),
 482:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.320829576277038198439987439508754886e+10),
 483:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.258818139077875493434420764260185306e+11),
 484:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.396791306321498426110315039064592443e+12),
 485:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.217617301585849875301440316301068439e+12)
 486:       };
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:       return exp(x) * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 488:    }
 489:    else
 490:    {
 491:       // Bessel I0 over[100, INF]
 492:       // Max error in interpolated form: 6.081e-35
 493:       // Max Error found at float128 precision = Poly: 1.407151e-34
 494:       BOOST_MATH_STATIC const T P[] = {
 495:          BOOST_MATH_BIG_CONSTANT(T, 113, 3.9894228040143267793994605993438200208417e-01),
 496:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.4960335515053725422747977247811372936584e-01),
 497:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.6751048484542891946087411826356811991039e-02),
 498:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.0907167423975030452875828826630006305665e-02),
 499:          BOOST_MATH_BIG_CONSTANT(T, 113, -5.7525704189964886494791082898669060345483e-02),
 500:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.1073698056568248642163476807108190176386e-01),
 501:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.6992139012879749064623499618582631684228e-01),
 502:          BOOST_MATH_BIG_CONSTANT(T, 113, -7.9530409594026597988098934027440110587905e-01),
 503:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.7462844478733532517044536719240098183686e+00),
 504:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.0870711340681926669381449306654104739256e+01),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:          BOOST_MATH_BIG_CONSTANT(T, 113, -4.8510175413216969245241059608553222505228e+01),
 506:          BOOST_MATH_BIG_CONSTANT(T, 113, -2.4094682286011573747064907919522894740063e+02),
 507:          BOOST_MATH_BIG_CONSTANT(T, 113, -1.3128845936764406865199641778959502795443e+03),
 508:          BOOST_MATH_BIG_CONSTANT(T, 113, -8.1655901321962541203257516341266838487359e+03),
 509:          BOOST_MATH_BIG_CONSTANT(T, 113, -3.8019591025686295090160445920753823994556e+04),
 510:          BOOST_MATH_BIG_CONSTANT(T, 113, -6.7008089049178178697338128837158732831105e+05)
 511:       };
 512:       T ex = exp(x / 2);
 513:       T result = ex * boost::math::tools::evaluate_polynomial(P, T(1 / x)) / sqrt(x);
 514:       result *= ex;
 515:       return result;
 516:    }
 517: }
 518: 
 519: template <typename T>
 520: BOOST_MATH_GPU_ENABLED T bessel_i1_imp(const T& x, const boost::math::integral_constant<int, 0>&)
 521: {
 522:    if(boost::math::tools::digits<T>() <= 24)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as exp, boost::math::tools::evaluate_polynomial. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 exp, boost::math::tools::evaluate_polynomial。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:       return bessel_i1_imp(x, boost::math::integral_constant<int, 24>());
 524:    else if(boost::math::tools::digits<T>() <= 53)
 525:       return bessel_i1_imp(x, boost::math::integral_constant<int, 53>());
 526:    else if(boost::math::tools::digits<T>() <= 64)
 527:       return bessel_i1_imp(x, boost::math::integral_constant<int, 64>());
 528:    else if(boost::math::tools::digits<T>() <= 113)
 529:       return bessel_i1_imp(x, boost::math::integral_constant<int, 113>());
 530:    BOOST_MATH_ASSERT(0);
 531:    return 0;
 532: }
 533: 
 534: template <typename T>
 535: BOOST_MATH_GPU_ENABLED inline T bessel_i1(const T& x)
 536: {
 537:    typedef boost::math::integral_constant<int,
 538:       ((boost::math::numeric_limits<T>::digits == 0) || (boost::math::numeric_limits<T>::radix != 2)) ?
 539:       0 :
 540:       boost::math::numeric_limits<T>::digits <= 24 ?
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as BOOST_MATH_ASSERT. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 541-556 / 第 541-556 行
~~~cpp
 541:       24 :
 542:       boost::math::numeric_limits<T>::digits <= 53 ?
 543:       53 :
 544:       boost::math::numeric_limits<T>::digits <= 64 ?
 545:       64 :
 546:       boost::math::numeric_limits<T>::digits <= 113 ?
 547:       113 : -1
 548:    > tag_type;
 549: 
 550:    return bessel_i1_imp(x, tag_type());
 551: }
 552: 
 553: }}} // namespaces
 554: 
 555: #endif // BOOST_MATH_BESSEL_I1_HPP
 556: 
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

## Key Concepts / 关键概念
- **Header guard / 头文件保护**: Prevents repeated inclusion during compilation. / 防止编译期间被重复包含。
- **Header dependencies / 头文件依赖**: Pulls in other declarations, traits, and standard facilities. / 引入其他声明、类型萃取和标准库能力。
- **Namespaces / 命名空间**: Keeps third-party symbols scoped and avoids collisions. / 通过命名空间约束第三方符号，避免冲突。
- **Templates / 模板**: Generalizes the implementation across numeric types or policies. / 让实现可适配不同数值类型或策略。
- **Functions / 函数**: Provides callable entry points or helpers. / 提供可调用的入口或辅助函数。
- **Macros / 宏**: Handles portability, visibility, or compile-time switches. / 处理可移植性、可见性或编译期开关。
- **Precomputed coefficients / 预计算系数**: Stores constants or tables used by numerical approximations. / 存储数值逼近使用的常量或查找表。
- **Control flow / 控制流**: Selects behavior through branches or loops. / 通过分支或循环选择行为。

## Dependencies / 依赖关系
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/rational.hpp, boost/math/tools/big_constant.hpp, boost/math/tools/assert.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/precision.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `bessel_i1, BOOST_MATH_ASSERT, exp, boost::math::tools::evaluate_polynomial`
