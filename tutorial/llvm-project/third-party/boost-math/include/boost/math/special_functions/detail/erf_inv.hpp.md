# erf_inv.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/erf_inv.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the erf inv special-function path.
- **作用（中文）**: 此头文件为 erf inv 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  (C) Copyright John Maddock 2006.
   2: //  (C) Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_SF_ERF_INV_HPP
   8: #define BOOST_MATH_SF_ERF_INV_HPP
   9: 
  10: #ifdef _MSC_VER
  11: #pragma once
  12: #pragma warning(push)
  13: #pragma warning(disable:4127) // Conditional expression is constant
  14: #pragma warning(disable:4702) // Unreachable code: optimization warning
  15: #endif
  16: 
  17: #include <boost/math/tools/config.hpp>
  18: 
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #ifndef BOOST_MATH_HAS_NVRTC
  20: 
  21: #include <type_traits>
  22: 
  23: namespace boost{ namespace math{
  24: 
  25: namespace detail{
  26: //
  27: // The inverse erf and erfc functions share a common implementation,
  28: // this version is for 80-bit long double's and smaller:
  29: //
  30: template <class T, class Policy>
  31: BOOST_MATH_GPU_ENABLED T erf_inv_imp(const T& p, const T& q, const Policy&, const std::integral_constant<int, 64>&)
  32: {
  33:    BOOST_MATH_STD_USING // for ADL of std names.
  34: 
  35:    T result = 0;
  36: 
~~~
- **EN:** This block imports dependencies such as type_traits so the surrounding code can use external declarations. It begins the file-level inclusion guard or portability wrapper. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 type_traits 等依赖，使周围代码可以使用外部声明。 它开始设置文件级防重包含或可移植性包装层。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:    if(p <= 0.5)
  38:    {
  39:       //
  40:       // Evaluate inverse erf using the rational approximation:
  41:       //
  42:       // x = p(p+10)(Y+R(p))
  43:       //
  44:       // Where Y is a constant, and R(p) is optimised for a low
  45:       // absolute error compared to |Y|.
  46:       //
  47:       // double: Max error found: 2.001849e-18
  48:       // long double: Max error found: 1.017064e-20
  49:       // Maximum Deviation Found (actual error term at infinite precision) 8.030e-21
  50:       //
  51:       // LCOV_EXCL_START
  52:       BOOST_MATH_STATIC_LOCAL_VARIABLE const float Y = 0.0891314744949340820313f;
  53:       BOOST_MATH_STATIC const T P[] = {
  54:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.000508781949658280665617),
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00836874819741736770379),
  56:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0334806625409744615033),
  57:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0126926147662974029034),
  58:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0365637971411762664006),
  59:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0219878681111168899165),
  60:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.00822687874676915743155),
  61:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00538772965071242932965)
  62:       };
  63:       BOOST_MATH_STATIC const T Q[] = {
  64:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
  65:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.970005043303290640362),
  66:          BOOST_MATH_BIG_CONSTANT(T, 64, -1.56574558234175846809),
  67:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.56221558398423026363),
  68:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.662328840472002992063),
  69:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.71228902341542847553),
  70:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.0527396382340099713954),
  71:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.0795283687341571680018),
  72:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.00233393759374190016776),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.000886216390456424707504)
  74:       };
  75:       // LCOV_EXCL_STOP
  76:       T g = p * (p + 10);
  77:       T r = tools::evaluate_polynomial(P, p) / tools::evaluate_polynomial(Q, p);
  78:       result = g * Y + g * r;
  79:    }
  80:    else if(q >= 0.25)
  81:    {
  82:       //
  83:       // Rational approximation for 0.5 > q >= 0.25
  84:       //
  85:       // x = sqrt(-2*log(q)) / (Y + R(q))
  86:       //
  87:       // Where Y is a constant, and R(q) is optimised for a low
  88:       // absolute error compared to Y.
  89:       //
  90:       // double : Max error found: 7.403372e-17
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:       // long double : Max error found: 6.084616e-20
  92:       // Maximum Deviation Found (error term) 4.811e-20
  93:       //
  94:       // LCOV_EXCL_START
  95:       BOOST_MATH_STATIC_LOCAL_VARIABLE const float Y = 2.249481201171875f;
  96:       BOOST_MATH_STATIC const T P[] = {
  97:          BOOST_MATH_BIG_CONSTANT(T, 64, -0.202433508355938759655),
  98:          BOOST_MATH_BIG_CONSTANT(T, 64, 0.105264680699391713268),
  99:          BOOST_MATH_BIG_CONSTANT(T, 64, 8.37050328343119927838),
 100:          BOOST_MATH_BIG_CONSTANT(T, 64, 17.6447298408374015486),
 101:          BOOST_MATH_BIG_CONSTANT(T, 64, -18.8510648058714251895),
 102:          BOOST_MATH_BIG_CONSTANT(T, 64, -44.6382324441786960818),
 103:          BOOST_MATH_BIG_CONSTANT(T, 64, 17.445385985570866523),
 104:          BOOST_MATH_BIG_CONSTANT(T, 64, 21.1294655448340526258),
 105:          BOOST_MATH_BIG_CONSTANT(T, 64, -3.67192254707729348546)
 106:       };
 107:       BOOST_MATH_STATIC const T Q[] = {
 108:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:          BOOST_MATH_BIG_CONSTANT(T, 64, 6.24264124854247537712),
 110:          BOOST_MATH_BIG_CONSTANT(T, 64, 3.9713437953343869095),
 111:          BOOST_MATH_BIG_CONSTANT(T, 64, -28.6608180499800029974),
 112:          BOOST_MATH_BIG_CONSTANT(T, 64, -20.1432634680485188801),
 113:          BOOST_MATH_BIG_CONSTANT(T, 64, 48.5609213108739935468),
 114:          BOOST_MATH_BIG_CONSTANT(T, 64, 10.8268667355460159008),
 115:          BOOST_MATH_BIG_CONSTANT(T, 64, -22.6436933413139721736),
 116:          BOOST_MATH_BIG_CONSTANT(T, 64, 1.72114765761200282724)
 117:       };
 118:       // LCOV_EXCL_STOP
 119:       T g = sqrt(-2 * log(q));
 120:       T xs = q - 0.25f;
 121:       T r = tools::evaluate_polynomial(P, xs) / tools::evaluate_polynomial(Q, xs);
 122:       result = g / (Y + r);
 123:    }
 124:    else
 125:    {
 126:       //
~~~
- **EN:** This range declares or defines callable logic such as sqrt, tools::evaluate_polynomial. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, tools::evaluate_polynomial。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:       // For q < 0.25 we have a series of rational approximations all
 128:       // of the general form:
 129:       //
 130:       // let: x = sqrt(-log(q))
 131:       //
 132:       // Then the result is given by:
 133:       //
 134:       // x(Y+R(x-B))
 135:       //
 136:       // where Y is a constant, B is the lowest value of x for which
 137:       // the approximation is valid, and R(x-B) is optimised for a low
 138:       // absolute error compared to Y.
 139:       //
 140:       // Note that almost all code will really go through the first
 141:       // or maybe second approximation.  After than we're dealing with very
 142:       // small input values indeed: 80 and 128 bit long double's go all the
 143:       // way down to ~ 1e-5000 so the "tail" is rather long...
 144:       //
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:       T x = sqrt(-log(q));
 146:       if(x < 3)
 147:       {
 148:          // LCOV_EXCL_START
 149:          // Max error found: 1.089051e-20
 150:          BOOST_MATH_STATIC_LOCAL_VARIABLE const float Y = 0.807220458984375f;
 151:          BOOST_MATH_STATIC const T P[] = {
 152:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.131102781679951906451),
 153:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.163794047193317060787),
 154:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.117030156341995252019),
 155:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.387079738972604337464),
 156:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.337785538912035898924),
 157:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.142869534408157156766),
 158:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.0290157910005329060432),
 159:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.00214558995388805277169),
 160:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.679465575181126350155e-6),
 161:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.285225331782217055858e-7),
 162:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.681149956853776992068e-9)
~~~
- **EN:** This range declares or defines callable logic such as sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:          };
 164:          BOOST_MATH_STATIC const T Q[] = {
 165:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 166:             BOOST_MATH_BIG_CONSTANT(T, 64, 3.46625407242567245975),
 167:             BOOST_MATH_BIG_CONSTANT(T, 64, 5.38168345707006855425),
 168:             BOOST_MATH_BIG_CONSTANT(T, 64, 4.77846592945843778382),
 169:             BOOST_MATH_BIG_CONSTANT(T, 64, 2.59301921623620271374),
 170:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.848854343457902036425),
 171:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.152264338295331783612),
 172:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.01105924229346489121)
 173:          };
 174:          // LCOV_EXCL_STOP
 175:          T xs = x - 1.125f;
 176:          T R = tools::evaluate_polynomial(P, xs) / tools::evaluate_polynomial(Q, xs);
 177:          result = Y * x + R * x;
 178:       }
 179:       else if(x < 6)
 180:       {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:          // LCOV_EXCL_START
 182:          // Max error found: 8.389174e-21
 183:          BOOST_MATH_STATIC_LOCAL_VARIABLE const float Y = 0.93995571136474609375f;
 184:          BOOST_MATH_STATIC const T P[] = {
 185:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.0350353787183177984712),
 186:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.00222426529213447927281),
 187:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.0185573306514231072324),
 188:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.00950804701325919603619),
 189:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.00187123492819559223345),
 190:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.000157544617424960554631),
 191:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.460469890584317994083e-5),
 192:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.230404776911882601748e-9),
 193:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.266339227425782031962e-11)
 194:          };
 195:          BOOST_MATH_STATIC const T Q[] = {
 196:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 197:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.3653349817554063097),
 198:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.762059164553623404043),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.220091105764131249824),
 200:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.0341589143670947727934),
 201:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.00263861676657015992959),
 202:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.764675292302794483503e-4)
 203:          };
 204:          // LCOV_EXCL_STOP
 205:          T xs = x - 3;
 206:          T R = tools::evaluate_polynomial(P, xs) / tools::evaluate_polynomial(Q, xs);
 207:          result = Y * x + R * x;
 208:       }
 209:       else if(x < 18)
 210:       {
 211:          // LCOV_EXCL_START
 212:          // Max error found: 1.481312e-19
 213:          BOOST_MATH_STATIC_LOCAL_VARIABLE const float Y = 0.98362827301025390625f;
 214:          BOOST_MATH_STATIC const T P[] = {
 215:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.0167431005076633737133),
 216:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.00112951438745580278863),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.00105628862152492910091),
 218:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.000209386317487588078668),
 219:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.149624783758342370182e-4),
 220:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.449696789927706453732e-6),
 221:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.462596163522878599135e-8),
 222:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.281128735628831791805e-13),
 223:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.99055709973310326855e-16)
 224:          };
 225:          BOOST_MATH_STATIC const T Q[] = {
 226:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 227:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.591429344886417493481),
 228:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.138151865749083321638),
 229:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.0160746087093676504695),
 230:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.000964011807005165528527),
 231:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.275335474764726041141e-4),
 232:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.282243172016108031869e-6)
 233:          };
 234:          // LCOV_EXCL_STOP
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:          T xs = x - 6;
 236:          T R = tools::evaluate_polynomial(P, xs) / tools::evaluate_polynomial(Q, xs);
 237:          result = Y * x + R * x;
 238:       }
 239:       else if(x < 44)
 240:       {
 241:          // LCOV_EXCL_START
 242:          // Max error found: 5.697761e-20
 243:          BOOST_MATH_STATIC_LOCAL_VARIABLE const float Y = 0.99714565277099609375f;
 244:          BOOST_MATH_STATIC const T P[] = {
 245:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.0024978212791898131227),
 246:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.779190719229053954292e-5),
 247:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.254723037413027451751e-4),
 248:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.162397777342510920873e-5),
 249:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.396341011304801168516e-7),
 250:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.411632831190944208473e-9),
 251:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.145596286718675035587e-11),
 252:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.116765012397184275695e-17)
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:          };
 254:          BOOST_MATH_STATIC const T Q[] = {
 255:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 256:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.207123112214422517181),
 257:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.0169410838120975906478),
 258:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.000690538265622684595676),
 259:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.145007359818232637924e-4),
 260:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.144437756628144157666e-6),
 261:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.509761276599778486139e-9)
 262:          };
 263:          // LCOV_EXCL_STOP
 264:          T xs = x - 18;
 265:          T R = tools::evaluate_polynomial(P, xs) / tools::evaluate_polynomial(Q, xs);
 266:          result = Y * x + R * x;
 267:       }
 268:       else
 269:       {
 270:          // LCOV_EXCL_START
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:          // Max error found: 1.279746e-20
 272:          BOOST_MATH_STATIC_LOCAL_VARIABLE const float Y = 0.99941349029541015625f;
 273:          BOOST_MATH_STATIC const T P[] = {
 274:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.000539042911019078575891),
 275:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.28398759004727721098e-6),
 276:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.899465114892291446442e-6),
 277:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.229345859265920864296e-7),
 278:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.225561444863500149219e-9),
 279:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.947846627503022684216e-12),
 280:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.135880130108924861008e-14),
 281:             BOOST_MATH_BIG_CONSTANT(T, 64, -0.348890393399948882918e-21)
 282:          };
 283:          BOOST_MATH_STATIC const T Q[] = {
 284:             BOOST_MATH_BIG_CONSTANT(T, 64, 1.0),
 285:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.0845746234001899436914),
 286:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.00282092984726264681981),
 287:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.468292921940894236786e-4),
 288:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.399968812193862100054e-6),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.161809290887904476097e-8),
 290:             BOOST_MATH_BIG_CONSTANT(T, 64, 0.231558608310259605225e-11)
 291:          };
 292:          // LCOV_EXCL_STOP
 293:          T xs = x - 44;
 294:          T R = tools::evaluate_polynomial(P, xs) / tools::evaluate_polynomial(Q, xs);
 295:          result = Y * x + R * x;
 296:       }
 297:    }
 298:    return result;
 299: }
 300: 
 301: template <class T, class Policy>
 302: struct erf_roots
 303: {
 304:    boost::math::tuple<T,T,T> operator()(const T& guess)
 305:    {
 306:       BOOST_MATH_STD_USING
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:       T derivative = sign * (2 / sqrt(constants::pi<T>())) * exp(-(guess * guess));
 308:       T derivative2 = -2 * guess * derivative;
 309:       return boost::math::make_tuple(((sign > 0) ? static_cast<T>(boost::math::erf(guess, Policy()) - target) : static_cast<T>(boost::math::erfc(guess, Policy())) - target), derivative, derivative2);
 310:    }
 311:    erf_roots(T z, int s) : target(z), sign(s) {}
 312: private:
 313:    T target;
 314:    int sign;
 315: };
 316: 
 317: template <class T, class Policy>
 318: T erf_inv_imp(const T& p, const T& q, const Policy& pol, const std::integral_constant<int, 0>&)
 319: {
 320:    //
 321:    // Generic version, get a guess that's accurate to 64-bits (10^-19)
 322:    //
 323:    using tag_type = std::integral_constant<int, 64>;
 324:    T guess = erf_inv_imp(p, q, pol, tag_type());
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as sqrt, erf_roots, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 sqrt, erf_roots, ...。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:    T result;
 326:    //
 327:    // If T has more bit's than 64 in it's mantissa then we need to iterate,
 328:    // otherwise we can just return the result:
 329:    //
 330:    if(policies::digits<T, Policy>() > 64)
 331:    {
 332:       std::uintmax_t max_iter = policies::get_max_root_iterations<Policy>();
 333:       if(p <= 0.5)
 334:       {
 335:          result = tools::halley_iterate(detail::erf_roots<typename std::remove_cv<T>::type, Policy>(p, 1), guess, static_cast<T>(0), tools::max_value<T>(), (policies::digits<T, Policy>() * 2) / 3, max_iter);
 336:       }
 337:       else
 338:       {
 339:          result = tools::halley_iterate(detail::erf_roots<typename std::remove_cv<T>::type, Policy>(q, -1), guess, static_cast<T>(0), tools::max_value<T>(), (policies::digits<T, Policy>() * 2) / 3, max_iter);
 340:       }
 341:       policies::check_root_iterations<T>("boost::math::erf_inv<%1%>", max_iter, pol);
 342:    }
~~~
- **EN:** This range declares or defines callable logic such as tools::halley_iterate. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::halley_iterate。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:    else
 344:    {
 345:       result = guess;
 346:    }
 347:    return result;
 348: }
 349: 
 350: } // namespace detail
 351: 
 352: template <class T, class Policy>
 353: BOOST_MATH_GPU_ENABLED typename tools::promote_args<T>::type erfc_inv(T z, const Policy& pol)
 354: {
 355:    typedef typename tools::promote_args<T>::type result_type;
 356: 
 357:    //
 358:    // Begin by testing for domain errors, and other special cases:
 359:    //
 360:    constexpr auto function = "boost::math::erfc_inv<%1%>(%1%, %1%)";
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:    if((z < 0) || (z > 2))
 362:       return policies::raise_domain_error<result_type>(function, "Argument outside range [0,2] in inverse erfc function (got p=%1%).", z, pol);
 363:    if(z == 0)
 364:       return policies::raise_overflow_error<result_type>(function, nullptr, pol);
 365:    if(z == 2)
 366:       return -policies::raise_overflow_error<result_type>(function, nullptr, pol);
 367:    //
 368:    // Normalise the input, so it's in the range [0,1], we will
 369:    // negate the result if z is outside that range.  This is a simple
 370:    // application of the erfc reflection formula: erfc(-z) = 2 - erfc(z)
 371:    //
 372:    result_type p, q, s;
 373:    if(z > 1)
 374:    {
 375:       q = 2 - z;
 376:       p = 1 - q;
 377:       s = -1;
 378:    }
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:    else
 380:    {
 381:       p = 1 - z;
 382:       q = z;
 383:       s = 1;
 384:    }
 385:    //
 386:    // A bit of meta-programming to figure out which implementation
 387:    // to use, based on the number of bits in the mantissa of T:
 388:    //
 389:    typedef typename policies::precision<result_type, Policy>::type precision_type;
 390:    typedef std::integral_constant<int,
 391:       precision_type::value <= 0 ? 0 :
 392:       precision_type::value <= 64 ? 64 : 0
 393:    > tag_type;
 394:    //
 395:    // Likewise use internal promotion, so we evaluate at a higher
 396:    // precision internally if it's appropriate:
~~~
- **EN:** Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:    //
 398:    typedef typename policies::evaluation<result_type, Policy>::type eval_type;
 399:    typedef typename policies::normalise<
 400:       Policy,
 401:       policies::promote_float<false>,
 402:       policies::promote_double<false>,
 403:       policies::discrete_quantile<>,
 404:       policies::assert_undefined<> >::type forwarding_policy;
 405: 
 406:    //
 407:    // And get the result, negating where required:
 408:    //
 409:    return s * policies::checked_narrowing_cast<result_type, forwarding_policy>(
 410:       detail::erf_inv_imp(static_cast<eval_type>(p), static_cast<eval_type>(q), forwarding_policy(), tag_type()), function);
 411: }
 412: 
 413: template <class T, class Policy>
 414: BOOST_MATH_GPU_ENABLED typename tools::promote_args<T>::type erf_inv(T z, const Policy& pol)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as detail::erf_inv_imp.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 detail::erf_inv_imp。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415: {
 416:    typedef typename tools::promote_args<T>::type result_type;
 417: 
 418:    //
 419:    // Begin by testing for domain errors, and other special cases:
 420:    //
 421:    constexpr auto function = "boost::math::erf_inv<%1%>(%1%, %1%)";
 422:    if((z < -1) || (z > 1))
 423:       return policies::raise_domain_error<result_type>(function, "Argument outside range [-1, 1] in inverse erf function (got p=%1%).", z, pol);
 424:    if(z == 1)
 425:       return policies::raise_overflow_error<result_type>(function, nullptr, pol);
 426:    if(z == -1)
 427:       return -policies::raise_overflow_error<result_type>(function, nullptr, pol);
 428:    if(z == 0)
 429:       return 0;
 430:    //
 431:    // Normalise the input, so it's in the range [0,1], we will
 432:    // negate the result if z is outside that range.  This is a simple
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:    // application of the erf reflection formula: erf(-z) = -erf(z)
 434:    //
 435:    result_type p, q, s;
 436:    if(z < 0)
 437:    {
 438:       p = -z;
 439:       q = 1 - p;
 440:       s = -1;
 441:    }
 442:    else
 443:    {
 444:       p = z;
 445:       q = 1 - z;
 446:       s = 1;
 447:    }
 448:    //
 449:    // A bit of meta-programming to figure out which implementation
 450:    // to use, based on the number of bits in the mantissa of T:
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:    //
 452:    typedef typename policies::precision<result_type, Policy>::type precision_type;
 453:    typedef std::integral_constant<int,
 454:       precision_type::value <= 0 ? 0 :
 455:       precision_type::value <= 64 ? 64 : 0
 456:    > tag_type;
 457:    //
 458:    // Likewise use internal promotion, so we evaluate at a higher
 459:    // precision internally if it's appropriate:
 460:    //
 461:    typedef typename policies::evaluation<result_type, Policy>::type eval_type;
 462:    typedef typename policies::normalise<
 463:       Policy,
 464:       policies::promote_float<false>,
 465:       policies::promote_double<false>,
 466:       policies::discrete_quantile<>,
 467:       policies::assert_undefined<> >::type forwarding_policy;
 468:    //
~~~
- **EN:** Type aliases simplify verbose template names and make later declarations easier to read. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 类型别名可以简化冗长的模板名称，使后续声明更易阅读。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:    // Likewise use internal promotion, so we evaluate at a higher
 470:    // precision internally if it's appropriate:
 471:    //
 472:    typedef typename policies::evaluation<result_type, Policy>::type eval_type;
 473: 
 474:    //
 475:    // And get the result, negating where required:
 476:    //
 477:    return s * policies::checked_narrowing_cast<result_type, forwarding_policy>(
 478:       detail::erf_inv_imp(static_cast<eval_type>(p), static_cast<eval_type>(q), forwarding_policy(), tag_type()), function);
 479: }
 480: 
 481: template <class T>
 482: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type erfc_inv(T z)
 483: {
 484:    return erfc_inv(z, policies::policy<>());
 485: }
 486: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as detail::erf_inv_imp.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 detail::erf_inv_imp。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487: template <class T>
 488: BOOST_MATH_GPU_ENABLED inline typename tools::promote_args<T>::type erf_inv(T z)
 489: {
 490:    return erf_inv(z, policies::policy<>());
 491: }
 492: 
 493: } // namespace math
 494: } // namespace boost
 495: 
 496: #else // Special handling for NVRTC
 497: 
 498: namespace boost {
 499: namespace math {
 500: 
 501: template <typename T>
 502: BOOST_MATH_GPU_ENABLED auto erf_inv(T x)
 503: {
 504:    return ::erfinv(x);
~~~
- **EN:** The code enters namespace scope (math::boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505: }
 506: 
 507: template <>
 508: BOOST_MATH_GPU_ENABLED auto erf_inv(float x)
 509: {
 510:    return ::erfinvf(x);
 511: }
 512: 
 513: template <typename T, typename Policy>
 514: BOOST_MATH_GPU_ENABLED auto erf_inv(T x, const Policy&)
 515: {
 516:    return ::erfinv(x);
 517: }
 518: 
 519: template <typename Policy>
 520: BOOST_MATH_GPU_ENABLED auto erf_inv(float x, const Policy&)
 521: {
 522:    return ::erfinvf(x);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523: }
 524: 
 525: template <typename T>
 526: BOOST_MATH_GPU_ENABLED auto erfc_inv(T x)
 527: {
 528:    return ::erfcinv(x);
 529: }
 530: 
 531: template <>
 532: BOOST_MATH_GPU_ENABLED auto erfc_inv(float x)
 533: {
 534:    return ::erfcinvf(x);
 535: }
 536: 
 537: template <typename T, typename Policy>
 538: BOOST_MATH_GPU_ENABLED auto erfc_inv(T x, const Policy&)
 539: {
 540:    return ::erfcinv(x);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541: }
 542: 
 543: template <typename Policy>
 544: BOOST_MATH_GPU_ENABLED auto erfc_inv(float x, const Policy&)
 545: {
 546:    return ::erfcinvf(x);
 547: }
 548: 
 549: } // namespace math
 550: } // namespace boost
 551: 
 552: #endif // BOOST_MATH_HAS_NVRTV
 553: 
 554: #ifdef _MSC_VER
 555: #pragma warning(pop)
 556: #endif
 557: 
 558: #endif // BOOST_MATH_SF_ERF_INV_HPP
~~~
- **EN:** The code enters namespace scope (math::boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. Return statements hand the computed result or status back to the caller.
- **CN:** 代码进入命名空间作用域（math::boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 return 语句会把计算结果或状态返回给调用方。

### Lines 559-559 / 第 559-559 行
~~~cpp
 559: 
~~~
- **EN:** This range is intentionally blank and separates neighboring declarations or algorithmic steps.
- **CN:** 此范围是有意保留的空行，用于分隔相邻声明或算法步骤。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, type_traits`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `tools::evaluate_polynomial, sqrt, erf_roots, erf_inv_imp, tools::halley_iterate, detail::erf_inv_imp`
