# lgamma_small.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/lgamma_small.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the lgamma small special-function path.
- **作用（中文）**: 此头文件为 lgamma small 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  (C) Copyright John Maddock 2006.
   2: //  (C) Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_SPECIAL_FUNCTIONS_DETAIL_LGAMMA_SMALL
   8: #define BOOST_MATH_SPECIAL_FUNCTIONS_DETAIL_LGAMMA_SMALL
   9: 
  10: #ifdef _MSC_VER
  11: #pragma once
  12: #endif
  13: 
  14: #include <boost/math/tools/config.hpp>
  15: #include <boost/math/tools/big_constant.hpp>
  16: #include <boost/math/tools/type_traits.hpp>
  17: #include <boost/math/tools/precision.hpp>
  18: #include <boost/math/special_functions/lanczos.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/big_constant.hpp, boost/math/tools/type_traits.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/big_constant.hpp, boost/math/tools/type_traits.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: 
  20: #if defined(__GNUC__) && defined(BOOST_MATH_USE_FLOAT128)
  21: //
  22: // This is the only way we can avoid
  23: // warning: non-standard suffix on floating constant [-Wpedantic]
  24: // when building with -Wall -pedantic.  Neither __extension__
  25: // nor #pragma diagnostic ignored work :(
  26: //
  27: #pragma GCC system_header
  28: #endif
  29: 
  30: namespace boost{ namespace math{ namespace detail{
  31: 
  32: //
  33: // These need forward declaring to keep GCC happy:
  34: //
  35: template <class T, class Policy, class Lanczos>
  36: BOOST_MATH_GPU_ENABLED T gamma_imp(T z, const Policy& pol, const Lanczos& l);
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: template <class T, class Policy>
  38: BOOST_MATH_GPU_ENABLED T gamma_imp(T z, const Policy& pol, const lanczos::undefined_lanczos& l);
  39: 
  40: //
  41: // lgamma for small arguments:
  42: //
  43: template <class T, class Policy, class Lanczos>
  44: BOOST_MATH_GPU_ENABLED T lgamma_small_imp(T z, T zm1, T zm2, const boost::math::integral_constant<int, 64>&, const Policy& /* l */, const Lanczos&)
  45: {
  46:    // This version uses rational approximations for small
  47:    // values of z accurate enough for 64-bit mantissas
  48:    // (80-bit long doubles), works well for 53-bit doubles as well.
  49:    // Lanczos is only used to select the Lanczos function.
  50: 
  51:    BOOST_MATH_STD_USING  // for ADL of std names
  52:    T result = 0;
  53: 
  54:    BOOST_MATH_ASSERT(z >= tools::root_epsilon<T>());
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as gamma_imp, BOOST_MATH_ASSERT.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 gamma_imp, BOOST_MATH_ASSERT。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:    /*
  56:    * Can not be reached:
  57:    *
  58:    if(z < tools::epsilon<T>())
  59:    {
  60:       result = -log(z);
  61:    }
  62:    */
  63:    if((zm1 == 0) || (zm2 == 0))
  64:    {
  65:       // nothing to do, result is zero....
  66:    }
  67:    else if(z > 2)
  68:    {
  69:       //
  70:       // Begin by performing argument reduction until
  71:       // z is in [2,3):
  72:       //
~~~
- **EN:** This range declares or defines callable logic such as log. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       if(z >= 3)
  74:       {
  75:          do
  76:          {
  77:             z -= 1;
  78:             zm2 -= 1;
  79:             result += log(z);
  80:          }while(z >= 3);
  81:          // Update zm2, we need it below:
  82:          zm2 = z - 2;
  83:       }
  84: 
  85:       //
  86:       // Use the following form:
  87:       //
  88:       // lgamma(z) = (z-2)(z+1)(Y + R(z-2))
  89:       //
  90:       // where R(z-2) is a rational approximation optimised for
~~~
- **EN:** This range declares or defines callable logic such as log, while. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, while。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:       // low absolute error - as long as it's absolute error
  92:       // is small compared to the constant Y - then any rounding
  93:       // error in it's computation will get wiped out.
  94:       //
  95:       // R(z-2) has the following properties:
  96:       //
  97:       // At double: Max error found:                    4.231e-18
  98:       // At long double: Max error found:               1.987e-21
  99:       // Maximum Deviation Found (approximation error): 5.900e-24
 100:       //
 101:       // LCOV_EXCL_START
 102:       BOOST_MATH_STATIC const T P[] = {
 103:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.180355685678449379109e-1)),
 104:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.25126649619989678683e-1)),
 105:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.494103151567532234274e-1)),
 106:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.172491608709613993966e-1)),
 107:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.259453563205438108893e-3)),
 108:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.541009869215204396339e-3)),
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.324588649825948492091e-4))
 110:       };
 111:       BOOST_MATH_STATIC const T Q[] = {
 112:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.1e1)),
 113:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.196202987197795200688e1)),
 114:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.148019669424231326694e1)),
 115:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.541391432071720958364e0)),
 116:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.988504251128010129477e-1)),
 117:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.82130967464889339326e-2)),
 118:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.224936291922115757597e-3)),
 119:          static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.223352763208617092964e-6))
 120:       };
 121: 
 122:       // LCOV_EXCL_STOP
 123:       constexpr float Y = 0.158963680267333984375e0f;
 124: 
 125:       T r = zm2 * (z + 1);
 126:       T R = tools::evaluate_polynomial(P, zm2);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:       R /= tools::evaluate_polynomial(Q, zm2);
 128: 
 129:       result +=  r * Y + r * R;
 130:    }
 131:    else
 132:    {
 133:       //
 134:       // If z is less than 1 use recurrence to shift to
 135:       // z in the interval [1,2]:
 136:       //
 137:       if(z < 1)
 138:       {
 139:          result += -log(z);
 140:          zm2 = zm1;
 141:          zm1 = z;
 142:          z += 1;
 143:       }
 144:       //
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, log. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, log。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:       // Two approximations, on for z in [1,1.5] and
 146:       // one for z in [1.5,2]:
 147:       //
 148:       if(z <= T(1.5))
 149:       {
 150:          //
 151:          // Use the following form:
 152:          //
 153:          // lgamma(z) = (z-1)(z-2)(Y + R(z-1))
 154:          //
 155:          // where R(z-1) is a rational approximation optimised for
 156:          // low absolute error - as long as it's absolute error
 157:          // is small compared to the constant Y - then any rounding
 158:          // error in it's computation will get wiped out.
 159:          //
 160:          // R(z-1) has the following properties:
 161:          //
 162:          // At double precision: Max error found:                1.230011e-17
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:          // At 80-bit long double precision:   Max error found:  5.631355e-21
 164:          // Maximum Deviation Found:                             3.139e-021
 165:          // Expected Error Term:                                 3.139e-021
 166: 
 167:          // LCOV_EXCL_START
 168:          constexpr float Y = 0.52815341949462890625f;
 169: 
 170:          BOOST_MATH_STATIC const T P[] = {
 171:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.490622454069039543534e-1)),
 172:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.969117530159521214579e-1)),
 173:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.414983358359495381969e0)),
 174:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.406567124211938417342e0)),
 175:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.158413586390692192217e0)),
 176:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.240149820648571559892e-1)),
 177:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.100346687696279557415e-2))
 178:          };
 179:          BOOST_MATH_STATIC const T Q[] = {
 180:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.1e1)),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.302349829846463038743e1)),
 182:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.348739585360723852576e1)),
 183:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.191415588274426679201e1)),
 184:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.507137738614363510846e0)),
 185:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.577039722690451849648e-1)),
 186:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.195768102601107189171e-2))
 187:          };
 188:          // LCOV_EXCL_STOP
 189: 
 190:          T r = tools::evaluate_polynomial(P, zm1) / tools::evaluate_polynomial(Q, zm1);
 191:          T prefix = zm1 * zm2;
 192: 
 193:          result += prefix * Y + prefix * r;
 194:       }
 195:       else
 196:       {
 197:          //
 198:          // Use the following form:
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:          //
 200:          // lgamma(z) = (2-z)(1-z)(Y + R(2-z))
 201:          //
 202:          // where R(2-z) is a rational approximation optimised for
 203:          // low absolute error - as long as it's absolute error
 204:          // is small compared to the constant Y - then any rounding
 205:          // error in it's computation will get wiped out.
 206:          //
 207:          // R(2-z) has the following properties:
 208:          //
 209:          // At double precision, max error found:              1.797565e-17
 210:          // At 80-bit long double precision, max error found:  9.306419e-21
 211:          // Maximum Deviation Found:                           2.151e-021
 212:          // Expected Error Term:                               2.150e-021
 213:          //
 214:          // LCOV_EXCL_START
 215:          constexpr float Y = 0.452017307281494140625f;
 216: 
~~~
- **EN:** This range continues the current implementation with a mix of declarations, structure, and low-level logic.
- **CN:** 此范围继续当前实现，混合了声明、结构与底层逻辑。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:          BOOST_MATH_STATIC const T P[] = {
 218:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.292329721830270012337e-1)),
 219:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.144216267757192309184e0)),
 220:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.142440390738631274135e0)),
 221:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.542809694055053558157e-1)),
 222:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.850535976868336437746e-2)),
 223:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.431171342679297331241e-3))
 224:          };
 225:          BOOST_MATH_STATIC const T Q[] = {
 226:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.1e1)),
 227:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.150169356054485044494e1)),
 228:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.846973248876495016101e0)),
 229:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.220095151814995745555e0)),
 230:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, 0.25582797155975869989e-1)),
 231:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.100666795539143372762e-2)),
 232:             static_cast<T>(BOOST_MATH_BIG_CONSTANT(T, 64, -0.827193521891290553639e-6))
 233:          };
 234:          // LCOV_EXCL_STOP
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235: 
 236:          T r = zm2 * zm1;
 237:          T R = tools::evaluate_polynomial(P, T(-zm2)) / tools::evaluate_polynomial(Q, T(-zm2));
 238: 
 239:          result += r * Y + r * R;
 240:       }
 241:    }
 242:    return result;
 243: }
 244: 
 245: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 246: 
 247: //
 248: // 128-bit floats aren't directly tested in our coverage tests (takes too long)
 249: // LCOV_EXCL_START
 250: //
 251: template <class T, class Policy, class Lanczos>
 252: T lgamma_small_imp(T z, T zm1, T zm2, const boost::math::integral_constant<int, 113>&, const Policy& /* l */, const Lanczos&)
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253: {
 254:    //
 255:    // This version uses rational approximations for small
 256:    // values of z accurate enough for 113-bit mantissas
 257:    // (128-bit long doubles).
 258:    //
 259:    BOOST_MATH_STD_USING  // for ADL of std names
 260:    T result = 0;
 261:    BOOST_MATH_ASSERT(z >= tools::root_epsilon<T>());
 262:    /*
 263:    *  Can not be reached:
 264:    if(z < tools::epsilon<T>())
 265:    {
 266:       result = -log(z);
 267:       BOOST_MATH_INSTRUMENT_CODE(result);
 268:    }
 269:    */
 270:    if((zm1 == 0) || (zm2 == 0))
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, log, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, log, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:    {
 272:       // nothing to do, result is zero....
 273:    }
 274:    else if(z > 2)
 275:    {
 276:       //
 277:       // Begin by performing argument reduction until
 278:       // z is in [2,3):
 279:       //
 280:       if(z >= 3)
 281:       {
 282:          do
 283:          {
 284:             z -= 1;
 285:             result += log(z);
 286:          }while(z >= 3);
 287:          zm2 = z - 2;
 288:       }
~~~
- **EN:** This range declares or defines callable logic such as log, while. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, while。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:       BOOST_MATH_INSTRUMENT_CODE(zm2);
 290:       BOOST_MATH_INSTRUMENT_CODE(z);
 291:       BOOST_MATH_INSTRUMENT_CODE(result);
 292: 
 293:       //
 294:       // Use the following form:
 295:       //
 296:       // lgamma(z) = (z-2)(z+1)(Y + R(z-2))
 297:       //
 298:       // where R(z-2) is a rational approximation optimised for
 299:       // low absolute error - as long as it's absolute error
 300:       // is small compared to the constant Y - then any rounding
 301:       // error in it's computation will get wiped out.
 302:       //
 303:       // Maximum Deviation Found (approximation error)      3.73e-37
 304: 
 305:       static const T P[] = {
 306:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.018035568567844937910504030027467476655),
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_CODE. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_CODE。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.013841458273109517271750705401202404195),
 308:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.062031842739486600078866923383017722399),
 309:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.052518418329052161202007865149435256093),
 310:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.01881718142472784129191838493267755758),
 311:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.0025104830367021839316463675028524702846),
 312:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.00021043176101831873281848891452678568311),
 313:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.00010249622350908722793327719494037981166),
 314:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.11381479670982006841716879074288176994e-4),
 315:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.49999811718089980992888533630523892389e-6),
 316:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.70529798686542184668416911331718963364e-8)
 317:       };
 318:       static const T Q[] = {
 319:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 320:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.5877485070422317542808137697939233685),
 321:          BOOST_MATH_BIG_CONSTANT(T, 113, 2.8797959228352591788629602533153837126),
 322:          BOOST_MATH_BIG_CONSTANT(T, 113, 1.8030885955284082026405495275461180977),
 323:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.69774331297747390169238306148355428436),
 324:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.17261566063277623942044077039756583802),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.02729301254544230229429621192443000121),
 326:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.0026776425891195270663133581960016620433),
 327:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.00015244249160486584591370355730402168106),
 328:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.43997034032479866020546814475414346627e-5),
 329:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.46295080708455613044541885534408170934e-7),
 330:          BOOST_MATH_BIG_CONSTANT(T, 113, -0.93326638207459533682980757982834180952e-11),
 331:          BOOST_MATH_BIG_CONSTANT(T, 113, 0.42316456553164995177177407325292867513e-13)
 332:       };
 333: 
 334:       T R = tools::evaluate_polynomial(P, zm2);
 335:       R /= tools::evaluate_polynomial(Q, zm2);
 336: 
 337:       static const float Y = 0.158963680267333984375F;
 338: 
 339:       T r = zm2 * (z + 1);
 340: 
 341:       result +=  r * Y + r * R;
 342:       BOOST_MATH_INSTRUMENT_CODE(result);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, BOOST_MATH_INSTRUMENT_CODE. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, BOOST_MATH_INSTRUMENT_CODE。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:    }
 344:    else
 345:    {
 346:       //
 347:       // If z is less than 1 use recurrence to shift to
 348:       // z in the interval [1,2]:
 349:       //
 350:       if(z < 1)
 351:       {
 352:          result += -log(z);
 353:          zm2 = zm1;
 354:          zm1 = z;
 355:          z += 1;
 356:       }
 357:       BOOST_MATH_INSTRUMENT_CODE(result);
 358:       BOOST_MATH_INSTRUMENT_CODE(z);
 359:       BOOST_MATH_INSTRUMENT_CODE(zm2);
 360:       //
~~~
- **EN:** This range declares or defines callable logic such as log, BOOST_MATH_INSTRUMENT_CODE. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, BOOST_MATH_INSTRUMENT_CODE。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:       // Three approximations, on for z in [1,1.35], [1.35,1.625] and [1.625,1]
 362:       //
 363:       if(z <= 1.35)
 364:       {
 365:          //
 366:          // Use the following form:
 367:          //
 368:          // lgamma(z) = (z-1)(z-2)(Y + R(z-1))
 369:          //
 370:          // where R(z-1) is a rational approximation optimised for
 371:          // low absolute error - as long as it's absolute error
 372:          // is small compared to the constant Y - then any rounding
 373:          // error in it's computation will get wiped out.
 374:          //
 375:          // R(z-1) has the following properties:
 376:          //
 377:          // Maximum Deviation Found (approximation error)            1.659e-36
 378:          // Expected Error Term (theoretical error)                  1.343e-36
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:          // Max error found at 128-bit long double precision         1.007e-35
 380:          //
 381:          static const float Y = 0.54076099395751953125f;
 382: 
 383:          static const T P[] = {
 384:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.036454670944013329356512090082402429697),
 385:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.066235835556476033710068679907798799959),
 386:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.67492399795577182387312206593595565371),
 387:             BOOST_MATH_BIG_CONSTANT(T, 113, -1.4345555263962411429855341651960000166),
 388:             BOOST_MATH_BIG_CONSTANT(T, 113, -1.4894319559821365820516771951249649563),
 389:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.87210277668067964629483299712322411566),
 390:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.29602090537771744401524080430529369136),
 391:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.0561832587517836908929331992218879676),
 392:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.0053236785487328044334381502530383140443),
 393:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.00018629360291358130461736386077971890789),
 394:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.10164985672213178500790406939467614498e-6),
 395:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.13680157145361387405588201461036338274e-8)
 396:          };
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:          static const T Q[] = {
 398:             BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 399:             BOOST_MATH_BIG_CONSTANT(T, 113, 4.9106336261005990534095838574132225599),
 400:             BOOST_MATH_BIG_CONSTANT(T, 113, 10.258804800866438510889341082793078432),
 401:             BOOST_MATH_BIG_CONSTANT(T, 113, 11.88588976846826108836629960537466889),
 402:             BOOST_MATH_BIG_CONSTANT(T, 113, 8.3455000546999704314454891036700998428),
 403:             BOOST_MATH_BIG_CONSTANT(T, 113, 3.6428823682421746343233362007194282703),
 404:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.97465989807254572142266753052776132252),
 405:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.15121052897097822172763084966793352524),
 406:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.012017363555383555123769849654484594893),
 407:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.0003583032812720649835431669893011257277)
 408:          };
 409: 
 410:          T r = tools::evaluate_polynomial(P, zm1) / tools::evaluate_polynomial(Q, zm1);
 411:          T prefix = zm1 * zm2;
 412: 
 413:          result += prefix * Y + prefix * r;
 414:          BOOST_MATH_INSTRUMENT_CODE(result);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, BOOST_MATH_INSTRUMENT_CODE. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, BOOST_MATH_INSTRUMENT_CODE。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:       }
 416:       else if(z <= 1.625)
 417:       {
 418:          //
 419:          // Use the following form:
 420:          //
 421:          // lgamma(z) = (2-z)(1-z)(Y + R(2-z))
 422:          //
 423:          // where R(2-z) is a rational approximation optimised for
 424:          // low absolute error - as long as it's absolute error
 425:          // is small compared to the constant Y - then any rounding
 426:          // error in it's computation will get wiped out.
 427:          //
 428:          // R(2-z) has the following properties:
 429:          //
 430:          // Max error found at 128-bit long double precision  9.634e-36
 431:          // Maximum Deviation Found (approximation error)     1.538e-37
 432:          // Expected Error Term (theoretical error)           2.350e-38
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:          //
 434:          static const float Y = 0.483787059783935546875f;
 435: 
 436:          static const T P[] = {
 437:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.017977422421608624353488126610933005432),
 438:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.18484528905298309555089509029244135703),
 439:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.40401251514859546989565001431430884082),
 440:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.40277179799147356461954182877921388182),
 441:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.21993421441282936476709677700477598816),
 442:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.069595742223850248095697771331107571011),
 443:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.012681481427699686635516772923547347328),
 444:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.0012489322866834830413292771335113136034),
 445:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.57058739515423112045108068834668269608e-4),
 446:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.8207548771933585614380644961342925976e-6)
 447:          };
 448:          static const T Q[] = {
 449:             BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 450:             BOOST_MATH_BIG_CONSTANT(T, 113, -2.9629552288944259229543137757200262073),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:             BOOST_MATH_BIG_CONSTANT(T, 113, 3.7118380799042118987185957298964772755),
 452:             BOOST_MATH_BIG_CONSTANT(T, 113, -2.5569815272165399297600586376727357187),
 453:             BOOST_MATH_BIG_CONSTANT(T, 113, 1.0546764918220835097855665680632153367),
 454:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.26574021300894401276478730940980810831),
 455:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.03996289731752081380552901986471233462),
 456:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.0033398680924544836817826046380586480873),
 457:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.00013288854760548251757651556792598235735),
 458:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.17194794958274081373243161848194745111e-5)
 459:          };
 460:          T r = zm2 * zm1;
 461:          T R = tools::evaluate_polynomial(P, T(0.625 - zm1)) / tools::evaluate_polynomial(Q, T(0.625 - zm1));
 462: 
 463:          result += r * Y + r * R;
 464:          BOOST_MATH_INSTRUMENT_CODE(result);
 465:       }
 466:       else
 467:       {
 468:          //
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, BOOST_MATH_INSTRUMENT_CODE. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, BOOST_MATH_INSTRUMENT_CODE。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:          // Same form as above.
 470:          //
 471:          // Max error found (at 128-bit long double precision) 1.831e-35
 472:          // Maximum Deviation Found (approximation error)      8.588e-36
 473:          // Expected Error Term (theoretical error)            1.458e-36
 474:          //
 475:          static const float Y = 0.443811893463134765625f;
 476: 
 477:          static const T P[] = {
 478:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.021027558364667626231512090082402429494),
 479:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.15128811104498736604523586803722368377),
 480:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.26249631480066246699388544451126410278),
 481:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.21148748610533489823742352180628489742),
 482:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.093964130697489071999873506148104370633),
 483:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.024292059227009051652542804957550866827),
 484:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.0036284453226534839926304745756906117066),
 485:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.0002939230129315195346843036254392485984),
 486:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.11088589183158123733132268042570710338e-4),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.13240510580220763969511741896361984162e-6)
 488:          };
 489:          static const T Q[] = {
 490:             BOOST_MATH_BIG_CONSTANT(T, 113, 1.0),
 491:             BOOST_MATH_BIG_CONSTANT(T, 113, -2.4240003754444040525462170802796471996),
 492:             BOOST_MATH_BIG_CONSTANT(T, 113, 2.4868383476933178722203278602342786002),
 493:             BOOST_MATH_BIG_CONSTANT(T, 113, -1.4047068395206343375520721509193698547),
 494:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.47583809087867443858344765659065773369),
 495:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.09865724264554556400463655444270700132),
 496:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.012238223514176587501074150988445109735),
 497:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.00084625068418239194670614419707491797097),
 498:             BOOST_MATH_BIG_CONSTANT(T, 113, 0.2796574430456237061420839429225710602e-4),
 499:             BOOST_MATH_BIG_CONSTANT(T, 113, -0.30202973883316730694433702165188835331e-6)
 500:          };
 501:          // (2 - x) * (1 - x) * (c + R(2 - x))
 502:          T r = zm2 * zm1;
 503:          T R = tools::evaluate_polynomial(P, T(-zm2)) / tools::evaluate_polynomial(Q, T(-zm2));
 504: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:          result += r * Y + r * R;
 506:          BOOST_MATH_INSTRUMENT_CODE(result);
 507:       }
 508:    }
 509:    BOOST_MATH_INSTRUMENT_CODE(result);
 510:    return result;
 511: }
 512: // LCOV_EXCL_STOP
 513: 
 514: template <class T, class Policy, class Lanczos>
 515: BOOST_MATH_GPU_ENABLED T lgamma_small_imp(T z, T zm1, T zm2, const boost::math::integral_constant<int, 0>&, const Policy& pol, const Lanczos& l)
 516: {
 517:    //
 518:    // No rational approximations are available because either
 519:    // T has no numeric_limits support (so we can't tell how
 520:    // many digits it has), or T has more digits than we know
 521:    // what to do with.... we do have a Lanczos approximation
 522:    // though, and that can be used to keep errors under control.
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_CODE.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_CODE。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:    //
 524:    BOOST_MATH_STD_USING  // for ADL of std names
 525:    T result = 0;
 526: 
 527:    BOOST_MATH_ASSERT(z >= tools::root_epsilon<T>());
 528:    /*
 529:    * Not reachable:
 530:    if(z < tools::epsilon<T>())
 531:    {
 532:       result = -log(z);
 533:    }
 534:    */
 535:    if(z < 0.5)
 536:    {
 537:       // taking the log of tgamma reduces the error, no danger of overflow here:
 538:       result = log(gamma_imp(z, pol, Lanczos()));
 539:    }
 540:    else if(z >= 3)
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, log. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, log。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:    {
 542:       // taking the log of tgamma reduces the error, no danger of overflow here:
 543:       result = log(gamma_imp(z, pol, Lanczos()));
 544:    }
 545:    else if(z >= 1.5)
 546:    {
 547:       // special case near 2:
 548:       T dz = zm2;
 549:       result = dz * log((z + lanczos_g_near_1_and_2(l) - T(0.5)) / boost::math::constants::e<T>());
 550:       result += boost::math::log1p(dz / (lanczos_g_near_1_and_2(l) + T(1.5)), pol) * T(1.5);
 551:       result += boost::math::log1p(Lanczos::lanczos_sum_near_2(dz), pol);
 552:    }
 553:    else
 554:    {
 555:       // special case near 1:
 556:       T dz = zm1;
 557:       result = dz * log((z + lanczos_g_near_1_and_2(l) - T(0.5)) / boost::math::constants::e<T>());
 558:       result += boost::math::log1p(dz / (lanczos_g_near_1_and_2(l) + T(0.5)), pol) / 2;
~~~
- **EN:** This range declares or defines callable logic such as log, boost::math::log1p. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log, boost::math::log1p。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 559-569 / 第 559-569 行
~~~cpp
 559:       result += boost::math::log1p(Lanczos::lanczos_sum_near_1(dz), pol);
 560:    }
 561:    return result;
 562: }
 563: 
 564: #endif // BOOST_MATH_HAS_GPU_SUPPORT
 565: 
 566: }}} // namespaces
 567: 
 568: #endif // BOOST_MATH_SPECIAL_FUNCTIONS_DETAIL_LGAMMA_SMALL
 569: 
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. This range declares or defines callable logic such as boost::math::log1p. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 此范围声明或定义了可调用逻辑，例如 boost::math::log1p。 return 语句会把计算结果或状态返回给调用方。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/big_constant.hpp, boost/math/tools/type_traits.hpp, boost/math/tools/precision.hpp, boost/math/special_functions/lanczos.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `gamma_imp, BOOST_MATH_ASSERT, log, while, tools::evaluate_polynomial, BOOST_MATH_INSTRUMENT_CODE, boost::math::log1p`
