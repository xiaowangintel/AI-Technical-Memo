# igamma_large.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/igamma_large.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the igamma large special-function path.
- **作用（中文）**: 此头文件为 igamma large 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行
~~~cpp
   1: //  Copyright John Maddock 2006.
   2: //  Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: //
   7: // This file implements the asymptotic expansions of the incomplete
   8: // gamma functions P(a, x) and Q(a, x), used when a is large and
   9: // x ~ a.
  10: //
  11: // The primary reference is:
  12: //
  13: // "The Asymptotic Expansion of the Incomplete Gamma Functions"
  14: // N. M. Temme.
  15: // Siam J. Math Anal. Vol 10 No 4, July 1979, p757.
  16: //
  17: // A different way of evaluating these expansions,
  18: // plus a lot of very useful background information is in:
  19: //
  20: // "A Set of Algorithms For the Incomplete Gamma Functions."
  21: // N. M. Temme.
  22: // Probability in the Engineering and Informational Sciences,
  23: // 8, 1994, 291.
  24: //
~~~
- **EN:** These lines are comments or license text that document provenance, usage, or structure rather than runtime behavior.
- **CN:** 这些行是注释或许可文本，用于说明来源、用法或结构，而不是运行时行为。

### Lines 25-48 / 第 25-48 行
~~~cpp
  25: // An alternative implementation is in:
  26: //
  27: // "Computation of the Incomplete Gamma Function Ratios and their Inverse."
  28: // A. R. Didonato and A. H. Morris.
  29: // ACM TOMS, Vol 12, No 4, Dec 1986, p377.
  30: //
  31: // There are various versions of the same code below, each accurate
  32: // to a different precision.  To understand the code, refer to Didonato
  33: // and Morris, from Eq 17 and 18 onwards.
  34: //
  35: // The coefficients used here are not taken from Didonato and Morris:
  36: // the domain over which these expansions are used is slightly different
  37: // to theirs, and their constants are not quite accurate enough for
  38: // 128-bit long double's.  Instead the coefficients were calculated
  39: // using the methods described by Temme p762 from Eq 3.8 onwards.
  40: // The values obtained agree with those obtained by Didonato and Morris
  41: // (at least to the first 30 digits that they provide).
  42: // At double precision the degrees of polynomial required for full
  43: // machine precision are close to those recommended to Didonato and Morris,
  44: // but of course many more terms are needed for larger types.
  45: //
  46: #ifndef BOOST_MATH_DETAIL_IGAMMA_LARGE
  47: #define BOOST_MATH_DETAIL_IGAMMA_LARGE
  48: 
~~~
- **EN:** It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 49-72 / 第 49-72 行
~~~cpp
  49: #ifdef _MSC_VER
  50: #pragma once
  51: #endif
  52: 
  53: #if defined(__GNUC__) && defined(BOOST_MATH_USE_FLOAT128)
  54: //
  55: // This is the only way we can avoid
  56: // warning: non-standard suffix on floating constant [-Wpedantic]
  57: // when building with -Wall -pedantic.  Neither __extension__
  58: // nor #pragma diagnostic ignored work :(
  59: //
  60: #pragma GCC system_header
  61: #endif
  62: 
  63: #include <boost/math/tools/config.hpp>
  64: #include <boost/math/tools/type_traits.hpp>
  65: 
  66: namespace boost{ namespace math{ namespace detail{
  67: 
  68: // This version will never be called (at runtime), it's a stub used
  69: // when T is unsuitable to be passed to these routines:
  70: //
  71: template <class T, class Policy>
  72: BOOST_MATH_GPU_ENABLED inline T igamma_temme_large(T, T, const Policy& /* pol */, const boost::math::integral_constant<int, 0>&)
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp so the surrounding code can use external declarations. It begins the file-level inclusion guard or portability wrapper. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp 等依赖，使周围代码可以使用外部声明。 它开始设置文件级防重包含或可移植性包装层。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 73-96 / 第 73-96 行
~~~cpp
  73: {
  74:    // stub function, should never actually be called
  75:    BOOST_MATH_ASSERT(0);
  76:    return 0;
  77: }
  78: //
  79: // This version is accurate for up to 64-bit mantissa's,
  80: // (80-bit long double, or 10^-20).
  81: //
  82: 
  83: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
  84: 
  85: template <class T, class Policy>
  86: BOOST_MATH_GPU_ENABLED T igamma_temme_large(T a, T x, const Policy& pol, const boost::math::integral_constant<int, 64>&)
  87: {
  88:    BOOST_MATH_STD_USING // ADL of std functions
  89:    T sigma = (x - a) / a;
  90:    T phi = -boost::math::log1pmx(sigma, pol);
  91:    T y = a * phi;
  92:    T z = sqrt(2 * phi);
  93:    if(x < a)
  94:       z = -z;
  95: 
  96:    T workspace[13];
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 97-120 / 第 97-120 行
~~~cpp
  97: 
  98:    // LCOV_EXCL_START
  99:    BOOST_MATH_STATIC const T C0[] = {
 100:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.333333333333333333333),
 101:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.0833333333333333333333),
 102:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.0148148148148148148148),
 103:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00115740740740740740741),
 104:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000352733686067019400353),
 105:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.0001787551440329218107),
 106:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.39192631785224377817e-4),
 107:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.218544851067999216147e-5),
 108:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.18540622107151599607e-5),
 109:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.829671134095308600502e-6),
 110:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.176659527368260793044e-6),
 111:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.670785354340149858037e-8),
 112:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.102618097842403080426e-7),
 113:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.438203601845335318655e-8),
 114:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.914769958223679023418e-9),
 115:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.255141939949462497669e-10),
 116:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.583077213255042506746e-10),
 117:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.243619480206674162437e-10),
 118:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.502766928011417558909e-11),
 119:    };
 120:    workspace[0] = tools::evaluate_polynomial(C0, z);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 121-144 / 第 121-144 行
~~~cpp
 121: 
 122:    BOOST_MATH_STATIC const T C1[] = {
 123:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00185185185185185185185),
 124:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00347222222222222222222),
 125:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00264550264550264550265),
 126:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000990226337448559670782),
 127:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000205761316872427983539),
 128:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.40187757201646090535e-6),
 129:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.18098550334489977837e-4),
 130:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.764916091608111008464e-5),
 131:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.161209008945634460038e-5),
 132:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.464712780280743434226e-8),
 133:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.137863344691572095931e-6),
 134:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.575254560351770496402e-7),
 135:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.119516285997781473243e-7),
 136:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.175432417197476476238e-10),
 137:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.100915437106004126275e-8),
 138:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.416279299184258263623e-9),
 139:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.856390702649298063807e-10),
 140:    };
 141:    workspace[1] = tools::evaluate_polynomial(C1, z);
 142: 
 143:    BOOST_MATH_STATIC const T C2[] = {
 144:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00413359788359788359788),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-168 / 第 145-168 行
~~~cpp
 145:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00268132716049382716049),
 146:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000771604938271604938272),
 147:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.200938786008230452675e-5),
 148:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000107366532263651605215),
 149:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.529234488291201254164e-4),
 150:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.127606351886187277134e-4),
 151:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.342357873409613807419e-7),
 152:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.137219573090629332056e-5),
 153:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.629899213838005502291e-6),
 154:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.142806142060642417916e-6),
 155:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.204770984219908660149e-9),
 156:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.140925299108675210533e-7),
 157:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.622897408492202203356e-8),
 158:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.136704883966171134993e-8),
 159:    };
 160:    workspace[2] = tools::evaluate_polynomial(C2, z);
 161: 
 162:    BOOST_MATH_STATIC const T C3[] = {
 163:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000649434156378600823045),
 164:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000229472093621399176955),
 165:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000469189494395255712128),
 166:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000267720632062838852962),
 167:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.756180167188397641073e-4),
 168:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.239650511386729665193e-6),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 169-192 / 第 169-192 行
~~~cpp
 169:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.110826541153473023615e-4),
 170:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.56749528269915965675e-5),
 171:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.142309007324358839146e-5),
 172:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.278610802915281422406e-10),
 173:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.169584040919302772899e-6),
 174:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.809946490538808236335e-7),
 175:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.191111684859736540607e-7),
 176:    };
 177:    workspace[3] = tools::evaluate_polynomial(C3, z);
 178: 
 179:    BOOST_MATH_STATIC const T C4[] = {
 180:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000861888290916711698605),
 181:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000784039221720066627474),
 182:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000299072480303190179733),
 183:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.146384525788434181781e-5),
 184:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.664149821546512218666e-4),
 185:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.396836504717943466443e-4),
 186:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.113757269706784190981e-4),
 187:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.250749722623753280165e-9),
 188:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.169541495365583060147e-5),
 189:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.890750753220530968883e-6),
 190:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.229293483400080487057e-6),
 191:    };
 192:    workspace[4] = tools::evaluate_polynomial(C4, z);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 193-216 / 第 193-216 行
~~~cpp
 193: 
 194:    BOOST_MATH_STATIC const T C5[] = {
 195:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000336798553366358150309),
 196:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.697281375836585777429e-4),
 197:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000277275324495939207873),
 198:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000199325705161888477003),
 199:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.679778047793720783882e-4),
 200:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.141906292064396701483e-6),
 201:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.135940481897686932785e-4),
 202:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.801847025633420153972e-5),
 203:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.229148117650809517038e-5),
 204:    };
 205:    workspace[5] = tools::evaluate_polynomial(C5, z);
 206: 
 207:    BOOST_MATH_STATIC const T C6[] = {
 208:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000531307936463992223166),
 209:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000592166437353693882865),
 210:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000270878209671804482771),
 211:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.790235323266032787212e-6),
 212:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.815396936756196875093e-4),
 213:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.561168275310624965004e-4),
 214:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.183291165828433755673e-4),
 215:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.307961345060330478256e-8),
 216:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.346515536880360908674e-5),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 217-240 / 第 217-240 行
~~~cpp
 217:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.20291327396058603727e-5),
 218:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.57887928631490037089e-6),
 219:    };
 220:    workspace[6] = tools::evaluate_polynomial(C6, z);
 221: 
 222:    BOOST_MATH_STATIC const T C7[] = {
 223:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000344367606892377671254),
 224:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.517179090826059219337e-4),
 225:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000334931610811422363117),
 226:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000281269515476323702274),
 227:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000109765822446847310235),
 228:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.127410090954844853795e-6),
 229:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.277444515115636441571e-4),
 230:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.182634888057113326614e-4),
 231:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.578769494973505239894e-5),
 232:    };
 233:    workspace[7] = tools::evaluate_polynomial(C7, z);
 234: 
 235:    BOOST_MATH_STATIC const T C8[] = {
 236:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000652623918595309418922),
 237:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000839498720672087279993),
 238:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000438297098541721005061),
 239:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.696909145842055197137e-6),
 240:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000166448466420675478374),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 241-264 / 第 241-264 行
~~~cpp
 241:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000127835176797692185853),
 242:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.462995326369130429061e-4),
 243:    };
 244:    workspace[8] = tools::evaluate_polynomial(C8, z);
 245: 
 246:    BOOST_MATH_STATIC const T C9[] = {
 247:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.000596761290192746250124),
 248:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.720489541602001055909e-4),
 249:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000678230883766732836162),
 250:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.0006401475260262758451),
 251:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000277501076343287044992),
 252:    };
 253:    workspace[9] = tools::evaluate_polynomial(C9, z);
 254: 
 255:    BOOST_MATH_STATIC const T C10[] = {
 256:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00133244544948006563713),
 257:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.0019144384985654775265),
 258:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00110893691345966373396),
 259:    };
 260:    workspace[10] = tools::evaluate_polynomial(C10, z);
 261: 
 262:    BOOST_MATH_STATIC const T C11[] = {
 263:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00157972766073083495909),
 264:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.000162516262783915816899),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 265-288 / 第 265-288 行
~~~cpp
 265:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00206334210355432762645),
 266:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00213896861856890981541),
 267:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00101085593912630031708),
 268:    };
 269:    workspace[11] = tools::evaluate_polynomial(C11, z);
 270: 
 271:    BOOST_MATH_STATIC const T C12[] = {
 272:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00407251211951401664727),
 273:       BOOST_MATH_BIG_CONSTANT(T, 64, 0.00640336283380806979482),
 274:       BOOST_MATH_BIG_CONSTANT(T, 64, -0.00404101610816766177474),
 275:    };
 276:    // LCOV_EXCL_STOP
 277: 
 278:    workspace[12] = tools::evaluate_polynomial(C12, z);
 279: 
 280:    T result = tools::evaluate_polynomial<13, T, T>(workspace, 1/a);
 281:    result *= exp(-y) / sqrt(2 * constants::pi<T>() * a);
 282:    if(x < a)
 283:       result = -result;
 284: 
 285:    result += boost::math::erfc(sqrt(y), pol) / 2;
 286: 
 287:    return result;
 288: }
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, exp. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, exp。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 289-312 / 第 289-312 行
~~~cpp
 289: 
 290: #endif
 291: 
 292: //
 293: // This one is accurate for 53-bit mantissa's
 294: // (IEEE double precision or 10^-17).
 295: //
 296: template <class T, class Policy>
 297: BOOST_MATH_GPU_ENABLED T igamma_temme_large(T a, T x, const Policy& pol, const boost::math::integral_constant<int, 53>&)
 298: {
 299:    BOOST_MATH_STD_USING // ADL of std functions
 300:    T sigma = (x - a) / a;
 301:    T phi = -boost::math::log1pmx(sigma, pol);
 302:    T y = a * phi;
 303:    T z = sqrt(2 * phi);
 304:    if(x < a)
 305:       z = -z;
 306: 
 307:    T workspace[10];
 308: 
 309:    // LCOV_EXCL_START
 310:    BOOST_MATH_STATIC const T C0[] = {
 311:       static_cast<T>(-0.33333333333333333L),
 312:       static_cast<T>(0.083333333333333333L),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::log1pmx, sqrt.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::log1pmx, sqrt。

### Lines 313-336 / 第 313-336 行
~~~cpp
 313:       static_cast<T>(-0.014814814814814815L),
 314:       static_cast<T>(0.0011574074074074074L),
 315:       static_cast<T>(0.0003527336860670194L),
 316:       static_cast<T>(-0.00017875514403292181L),
 317:       static_cast<T>(0.39192631785224378e-4L),
 318:       static_cast<T>(-0.21854485106799922e-5L),
 319:       static_cast<T>(-0.185406221071516e-5L),
 320:       static_cast<T>(0.8296711340953086e-6L),
 321:       static_cast<T>(-0.17665952736826079e-6L),
 322:       static_cast<T>(0.67078535434014986e-8L),
 323:       static_cast<T>(0.10261809784240308e-7L),
 324:       static_cast<T>(-0.43820360184533532e-8L),
 325:       static_cast<T>(0.91476995822367902e-9L),
 326:    };
 327:    workspace[0] = tools::evaluate_polynomial(C0, z);
 328: 
 329:    BOOST_MATH_STATIC const T C1[] = {
 330:       static_cast<T>(-0.0018518518518518519L),
 331:       static_cast<T>(-0.0034722222222222222L),
 332:       static_cast<T>(0.0026455026455026455L),
 333:       static_cast<T>(-0.00099022633744855967L),
 334:       static_cast<T>(0.00020576131687242798L),
 335:       static_cast<T>(-0.40187757201646091e-6L),
 336:       static_cast<T>(-0.18098550334489978e-4L),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 337-360 / 第 337-360 行
~~~cpp
 337:       static_cast<T>(0.76491609160811101e-5L),
 338:       static_cast<T>(-0.16120900894563446e-5L),
 339:       static_cast<T>(0.46471278028074343e-8L),
 340:       static_cast<T>(0.1378633446915721e-6L),
 341:       static_cast<T>(-0.5752545603517705e-7L),
 342:       static_cast<T>(0.11951628599778147e-7L),
 343:    };
 344:    workspace[1] = tools::evaluate_polynomial(C1, z);
 345: 
 346:    BOOST_MATH_STATIC const T C2[] = {
 347:       static_cast<T>(0.0041335978835978836L),
 348:       static_cast<T>(-0.0026813271604938272L),
 349:       static_cast<T>(0.00077160493827160494L),
 350:       static_cast<T>(0.20093878600823045e-5L),
 351:       static_cast<T>(-0.00010736653226365161L),
 352:       static_cast<T>(0.52923448829120125e-4L),
 353:       static_cast<T>(-0.12760635188618728e-4L),
 354:       static_cast<T>(0.34235787340961381e-7L),
 355:       static_cast<T>(0.13721957309062933e-5L),
 356:       static_cast<T>(-0.6298992138380055e-6L),
 357:       static_cast<T>(0.14280614206064242e-6L),
 358:    };
 359:    workspace[2] = tools::evaluate_polynomial(C2, z);
 360: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 361-384 / 第 361-384 行
~~~cpp
 361:    BOOST_MATH_STATIC const T C3[] = {
 362:       static_cast<T>(0.00064943415637860082L),
 363:       static_cast<T>(0.00022947209362139918L),
 364:       static_cast<T>(-0.00046918949439525571L),
 365:       static_cast<T>(0.00026772063206283885L),
 366:       static_cast<T>(-0.75618016718839764e-4L),
 367:       static_cast<T>(-0.23965051138672967e-6L),
 368:       static_cast<T>(0.11082654115347302e-4L),
 369:       static_cast<T>(-0.56749528269915966e-5L),
 370:       static_cast<T>(0.14230900732435884e-5L),
 371:    };
 372:    workspace[3] = tools::evaluate_polynomial(C3, z);
 373: 
 374:    BOOST_MATH_STATIC const T C4[] = {
 375:       static_cast<T>(-0.0008618882909167117L),
 376:       static_cast<T>(0.00078403922172006663L),
 377:       static_cast<T>(-0.00029907248030319018L),
 378:       static_cast<T>(-0.14638452578843418e-5L),
 379:       static_cast<T>(0.66414982154651222e-4L),
 380:       static_cast<T>(-0.39683650471794347e-4L),
 381:       static_cast<T>(0.11375726970678419e-4L),
 382:    };
 383:    workspace[4] = tools::evaluate_polynomial(C4, z);
 384: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 385-408 / 第 385-408 行
~~~cpp
 385:    BOOST_MATH_STATIC const T C5[] = {
 386:       static_cast<T>(-0.00033679855336635815L),
 387:       static_cast<T>(-0.69728137583658578e-4L),
 388:       static_cast<T>(0.00027727532449593921L),
 389:       static_cast<T>(-0.00019932570516188848L),
 390:       static_cast<T>(0.67977804779372078e-4L),
 391:       static_cast<T>(0.1419062920643967e-6L),
 392:       static_cast<T>(-0.13594048189768693e-4L),
 393:       static_cast<T>(0.80184702563342015e-5L),
 394:       static_cast<T>(-0.22914811765080952e-5L),
 395:    };
 396:    workspace[5] = tools::evaluate_polynomial(C5, z);
 397: 
 398:    BOOST_MATH_STATIC const T C6[] = {
 399:       static_cast<T>(0.00053130793646399222L),
 400:       static_cast<T>(-0.00059216643735369388L),
 401:       static_cast<T>(0.00027087820967180448L),
 402:       static_cast<T>(0.79023532326603279e-6L),
 403:       static_cast<T>(-0.81539693675619688e-4L),
 404:       static_cast<T>(0.56116827531062497e-4L),
 405:       static_cast<T>(-0.18329116582843376e-4L),
 406:    };
 407:    workspace[6] = tools::evaluate_polynomial(C6, z);
 408: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 409-432 / 第 409-432 行
~~~cpp
 409:    BOOST_MATH_STATIC const T C7[] = {
 410:       static_cast<T>(0.00034436760689237767L),
 411:       static_cast<T>(0.51717909082605922e-4L),
 412:       static_cast<T>(-0.00033493161081142236L),
 413:       static_cast<T>(0.0002812695154763237L),
 414:       static_cast<T>(-0.00010976582244684731L),
 415:    };
 416:    workspace[7] = tools::evaluate_polynomial(C7, z);
 417: 
 418:    BOOST_MATH_STATIC const T C8[] = {
 419:       static_cast<T>(-0.00065262391859530942L),
 420:       static_cast<T>(0.00083949872067208728L),
 421:       static_cast<T>(-0.00043829709854172101L),
 422:    };
 423:    // LCOV_EXCL_STOP
 424:    workspace[8] = tools::evaluate_polynomial(C8, z);
 425:    workspace[9] = static_cast<T>(-0.00059676129019274625L);
 426: 
 427:    T result = tools::evaluate_polynomial<10, T, T>(workspace, 1/a);
 428:    result *= exp(-y) / sqrt(2 * constants::pi<T>() * a);
 429:    if(x < a)
 430:       result = -result;
 431: 
 432:    #ifdef BOOST_MATH_HAS_NVRTC
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, exp. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, exp。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 433-456 / 第 433-456 行
~~~cpp
 433:    if (boost::math::is_same_v<T, float>)
 434:    {
 435:       result += ::erfcf(::sqrtf(y)) / 2;
 436:    }
 437:    else
 438:    {
 439:       result += ::erfc(::sqrt(y)) / 2;
 440:    }
 441:    #else
 442:    result += boost::math::erfc(sqrt(y), pol) / 2;
 443:    #endif
 444: 
 445:    return result;
 446: }
 447: //
 448: // This one is accurate for 24-bit mantissa's
 449: // (IEEE float precision, or 10^-8)
 450: //
 451: template <class T, class Policy>
 452: BOOST_MATH_GPU_ENABLED T igamma_temme_large(T a, T x, const Policy& pol, const boost::math::integral_constant<int, 24>&)
 453: {
 454:    BOOST_MATH_STD_USING // ADL of std functions
 455:    T sigma = (x - a) / a;
 456:    T phi = -boost::math::log1pmx(sigma, pol);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::log1pmx.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::log1pmx。

### Lines 457-480 / 第 457-480 行
~~~cpp
 457:    T y = a * phi;
 458:    T z = sqrt(2 * phi);
 459:    if(x < a)
 460:       z = -z;
 461: 
 462:    T workspace[3];
 463: 
 464:    // LCOV_EXCL_START
 465:    BOOST_MATH_STATIC const T C0[] = {
 466:       static_cast<T>(-0.333333333L),
 467:       static_cast<T>(0.0833333333L),
 468:       static_cast<T>(-0.0148148148L),
 469:       static_cast<T>(0.00115740741L),
 470:       static_cast<T>(0.000352733686L),
 471:       static_cast<T>(-0.000178755144L),
 472:       static_cast<T>(0.391926318e-4L),
 473:    };
 474:    workspace[0] = tools::evaluate_polynomial(C0, z);
 475: 
 476:    BOOST_MATH_STATIC const T C1[] = {
 477:       static_cast<T>(-0.00185185185L),
 478:       static_cast<T>(-0.00347222222L),
 479:       static_cast<T>(0.00264550265L),
 480:       static_cast<T>(-0.000990226337L),
~~~
- **EN:** This range declares or defines callable logic such as sqrt, tools::evaluate_polynomial. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, tools::evaluate_polynomial。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 481-504 / 第 481-504 行
~~~cpp
 481:       static_cast<T>(0.000205761317L),
 482:    };
 483:    workspace[1] = tools::evaluate_polynomial(C1, z);
 484: 
 485:    BOOST_MATH_STATIC const T C2[] = {
 486:       static_cast<T>(0.00413359788L),
 487:       static_cast<T>(-0.00268132716L),
 488:       static_cast<T>(0.000771604938L),
 489:    };
 490:    workspace[2] = tools::evaluate_polynomial(C2, z);
 491:    // LCOV_EXCL_STOP
 492: 
 493:    T result = tools::evaluate_polynomial(workspace, 1/a);
 494:    result *= exp(-y) / sqrt(2 * constants::pi<T>() * a);
 495:    if(x < a)
 496:       result = -result;
 497: 
 498:    #ifdef BOOST_MATH_HAS_NVRTC
 499:    if (boost::math::is_same_v<T, float>)
 500:    {
 501:       result += ::erfcf(::sqrtf(y)) / 2;
 502:    }
 503:    else
 504:    {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 505-528 / 第 505-528 行
~~~cpp
 505:       result += ::erfc(::sqrt(y)) / 2;
 506:    }
 507:    #else
 508:    result += boost::math::erfc(sqrt(y), pol) / 2;
 509:    #endif
 510: 
 511:    return result;
 512: }
 513: //
 514: // And finally, a version for 113-bit mantissa's
 515: // (128-bit long doubles, or 10^-34).
 516: // Note this one has been optimised for a > 200
 517: // It's use for a < 200 is not recommended, that would
 518: // require many more terms in the polynomials.
 519: //
 520: // LCOV_EXCL_START: 128-bit floats not deliberately tested in our coverage tests (takes too long)
 521: #ifndef BOOST_MATH_HAS_GPU_SUPPORT
 522: 
 523: template <class T, class Policy>
 524: BOOST_MATH_GPU_ENABLED T igamma_temme_large(T a, T x, const Policy& pol, const boost::math::integral_constant<int, 113>&)
 525: {
 526:    BOOST_MATH_STD_USING // ADL of std functions
 527:    T sigma = (x - a) / a;
 528:    T phi = -boost::math::log1pmx(sigma, pol);
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 529-552 / 第 529-552 行
~~~cpp
 529:    T y = a * phi;
 530:    T z = sqrt(2 * phi);
 531:    if(x < a)
 532:       z = -z;
 533: 
 534:    T workspace[14];
 535: 
 536:    BOOST_MATH_STATIC const T C0[] = {
 537:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.333333333333333333333333333333333333),
 538:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0833333333333333333333333333333333333),
 539:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.0148148148148148148148148148148148148),
 540:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00115740740740740740740740740740740741),
 541:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0003527336860670194003527336860670194),
 542:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000178755144032921810699588477366255144),
 543:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.391926317852243778169704095630021556e-4),
 544:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.218544851067999216147364295512443661e-5),
 545:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.185406221071515996070179883622956325e-5),
 546:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.829671134095308600501624213166443227e-6),
 547:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.17665952736826079304360054245742403e-6),
 548:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.670785354340149858036939710029613572e-8),
 549:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.102618097842403080425739573227252951e-7),
 550:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.438203601845335318655297462244719123e-8),
 551:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.914769958223679023418248817633113681e-9),
 552:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.255141939949462497668779537993887013e-10),
~~~
- **EN:** This range declares or defines callable logic such as sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 553-576 / 第 553-576 行
~~~cpp
 553:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.583077213255042506746408945040035798e-10),
 554:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.243619480206674162436940696707789943e-10),
 555:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.502766928011417558909054985925744366e-11),
 556:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.110043920319561347708374174497293411e-12),
 557:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.337176326240098537882769884169200185e-12),
 558:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.13923887224181620659193661848957998e-12),
 559:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.285348938070474432039669099052828299e-13),
 560:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.513911183424257261899064580300494205e-15),
 561:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.197522882943494428353962401580710912e-14),
 562:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.809952115670456133407115668702575255e-15),
 563:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.165225312163981618191514820265351162e-15),
 564:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.253054300974788842327061090060267385e-17),
 565:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.116869397385595765888230876507793475e-16),
 566:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.477003704982048475822167804084816597e-17),
 567:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.969912605905623712420709685898585354e-18),
 568:    };
 569:    workspace[0] = tools::evaluate_polynomial(C0, z);
 570: 
 571:    BOOST_MATH_STATIC const T C1[] = {
 572:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00185185185185185185185185185185185185),
 573:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00347222222222222222222222222222222222),
 574:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0026455026455026455026455026455026455),
 575:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000990226337448559670781893004115226337),
 576:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000205761316872427983539094650205761317),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 577-600 / 第 577-600 行
~~~cpp
 577:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.401877572016460905349794238683127572e-6),
 578:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.180985503344899778370285914867533523e-4),
 579:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.76491609160811100846374214980916921e-5),
 580:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.16120900894563446003775221882217767e-5),
 581:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.464712780280743434226135033938722401e-8),
 582:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.137863344691572095931187533077488877e-6),
 583:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.575254560351770496402194531835048307e-7),
 584:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.119516285997781473243076536699698169e-7),
 585:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.175432417197476476237547551202312502e-10),
 586:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.100915437106004126274577504686681675e-8),
 587:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.416279299184258263623372347219858628e-9),
 588:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.856390702649298063807431562579670208e-10),
 589:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.606721510160475861512701762169919581e-13),
 590:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.716249896481148539007961017165545733e-11),
 591:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.293318664377143711740636683615595403e-11),
 592:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.599669636568368872330374527568788909e-12),
 593:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.216717865273233141017100472779701734e-15),
 594:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.497833997236926164052815522048108548e-13),
 595:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.202916288237134247736694804325894226e-13),
 596:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.413125571381061004935108332558187111e-14),
 597:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.828651623988309644380188591057589316e-18),
 598:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.341003088693333279336339355910600992e-15),
 599:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.138541953028939715357034547426313703e-15),
 600:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.281234665322887466568860332727259483e-16),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 601-624 / 第 601-624 行
~~~cpp
 601:    };
 602:    workspace[1] = tools::evaluate_polynomial(C1, z);
 603: 
 604:    BOOST_MATH_STATIC const T C2[] = {
 605:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0041335978835978835978835978835978836),
 606:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00268132716049382716049382716049382716),
 607:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000771604938271604938271604938271604938),
 608:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.200938786008230452674897119341563786e-5),
 609:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000107366532263651605215391223621676297),
 610:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.529234488291201254164217127180090143e-4),
 611:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.127606351886187277133779191392360117e-4),
 612:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.34235787340961380741902003904747389e-7),
 613:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.137219573090629332055943852926020279e-5),
 614:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.629899213838005502290672234278391876e-6),
 615:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.142806142060642417915846008822771748e-6),
 616:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.204770984219908660149195854409200226e-9),
 617:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.140925299108675210532930244154315272e-7),
 618:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.622897408492202203356394293530327112e-8),
 619:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.136704883966171134992724380284402402e-8),
 620:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.942835615901467819547711211663208075e-12),
 621:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.128722524000893180595479368872770442e-9),
 622:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.556459561343633211465414765894951439e-10),
 623:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.119759355463669810035898150310311343e-10),
 624:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.416897822518386350403836626692480096e-14),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 625-648 / 第 625-648 行
~~~cpp
 625:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.109406404278845944099299008640802908e-11),
 626:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.4662239946390135746326204922464679e-12),
 627:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.990510576390690597844122258212382301e-13),
 628:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.189318767683735145056885183170630169e-16),
 629:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.885922187259112726176031067028740667e-14),
 630:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.373782039804640545306560251777191937e-14),
 631:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.786883363903515525774088394065960751e-15),
 632:    };
 633:    workspace[2] = tools::evaluate_polynomial(C2, z);
 634: 
 635:    BOOST_MATH_STATIC const T C3[] = {
 636:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000649434156378600823045267489711934156),
 637:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000229472093621399176954732510288065844),
 638:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000469189494395255712128140111679206329),
 639:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000267720632062838852962309752433209223),
 640:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.756180167188397641072538191879755666e-4),
 641:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.239650511386729665193314027333231723e-6),
 642:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.110826541153473023614770299726861227e-4),
 643:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.567495282699159656749963105701560205e-5),
 644:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.14230900732435883914551894470580433e-5),
 645:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.278610802915281422405802158211174452e-10),
 646:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.16958404091930277289864168795820267e-6),
 647:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.809946490538808236335278504852724081e-7),
 648:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.191111684859736540606728140872727635e-7),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 649-672 / 第 649-672 行
~~~cpp
 649:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.239286204398081179686413514022282056e-11),
 650:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.206201318154887984369925818486654549e-8),
 651:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.946049666185513217375417988510192814e-9),
 652:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.215410497757749078380130268468744512e-9),
 653:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.138882333681390304603424682490735291e-13),
 654:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.218947616819639394064123400466489455e-10),
 655:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.979099895117168512568262802255883368e-11),
 656:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.217821918801809621153859472011393244e-11),
 657:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.62088195734079014258166361684972205e-16),
 658:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.212697836327973697696702537114614471e-12),
 659:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.934468879151743333127396765626749473e-13),
 660:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.204536712267828493249215913063207436e-13),
 661:    };
 662:    workspace[3] = tools::evaluate_polynomial(C3, z);
 663: 
 664:    BOOST_MATH_STATIC const T C4[] = {
 665:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000861888290916711698604702719929057378),
 666:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00078403922172006662747403488144228885),
 667:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000299072480303190179733389609932819809),
 668:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.146384525788434181781232535690697556e-5),
 669:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.664149821546512218665853782451862013e-4),
 670:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.396836504717943466443123507595386882e-4),
 671:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.113757269706784190980552042885831759e-4),
 672:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.250749722623753280165221942390057007e-9),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 673-696 / 第 673-696 行
~~~cpp
 673:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.169541495365583060147164356781525752e-5),
 674:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.890750753220530968882898422505515924e-6),
 675:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.229293483400080487057216364891158518e-6),
 676:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.295679413754404904696572852500004588e-10),
 677:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.288658297427087836297341274604184504e-7),
 678:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.141897394378032193894774303903982717e-7),
 679:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.344635804994648970659527720474194356e-8),
 680:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.230245171745280671320192735850147087e-12),
 681:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.394092330280464052750697640085291799e-9),
 682:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.186023389685045019134258533045185639e-9),
 683:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.435632300505661804380678327446262424e-10),
 684:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.127860010162962312660550463349930726e-14),
 685:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.467927502665791946200382739991760062e-11),
 686:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.214924647061348285410535341910721086e-11),
 687:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.490881561480965216323649688463984082e-12),
 688:    };
 689:    workspace[4] = tools::evaluate_polynomial(C4, z);
 690: 
 691:    BOOST_MATH_STATIC const T C5[] = {
 692:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000336798553366358150308767592718210002),
 693:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.697281375836585777429398828575783308e-4),
 694:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00027727532449593920787336425196507501),
 695:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000199325705161888477003360405280844238),
 696:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.679778047793720783881640176604435742e-4),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 697-720 / 第 697-720 行
~~~cpp
 697:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.141906292064396701483392727105575757e-6),
 698:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.135940481897686932784583938837504469e-4),
 699:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.80184702563342015397192571980419684e-5),
 700:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.229148117650809517038048790128781806e-5),
 701:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.325247355129845395166230137750005047e-9),
 702:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.346528464910852649559195496827579815e-6),
 703:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.184471871911713432765322367374920978e-6),
 704:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.482409670378941807563762631738989002e-7),
 705:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.179894667217435153025754291716644314e-13),
 706:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.630619450001352343517516981425944698e-8),
 707:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.316241762877456793773762181540969623e-8),
 708:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.784092425369742929000839303523267545e-9),
 709:    };
 710:    workspace[5] = tools::evaluate_polynomial(C5, z);
 711: 
 712:    BOOST_MATH_STATIC const T C6[] = {
 713:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00053130793646399222316574854297762391),
 714:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000592166437353693882864836225604401187),
 715:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000270878209671804482771279183488328692),
 716:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.790235323266032787212032944390816666e-6),
 717:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.815396936756196875092890088464682624e-4),
 718:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.561168275310624965003775619041471695e-4),
 719:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.183291165828433755673259749374098313e-4),
 720:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.307961345060330478256414192546677006e-8),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 721-744 / 第 721-744 行
~~~cpp
 721:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.346515536880360908673728529745376913e-5),
 722:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.202913273960586037269527254582695285e-5),
 723:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.578879286314900370889997586203187687e-6),
 724:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.233863067382665698933480579231637609e-12),
 725:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.88286007463304835250508524317926246e-7),
 726:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.474359588804081278032150770595852426e-7),
 727:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.125454150207103824457130611214783073e-7),
 728:    };
 729:    workspace[6] = tools::evaluate_polynomial(C6, z);
 730: 
 731:    BOOST_MATH_STATIC const T C7[] = {
 732:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000344367606892377671254279625108523655),
 733:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.517179090826059219337057843002058823e-4),
 734:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000334931610811422363116635090580012327),
 735:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000281269515476323702273722110707777978),
 736:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000109765822446847310235396824500789005),
 737:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.127410090954844853794579954588107623e-6),
 738:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.277444515115636441570715073933712622e-4),
 739:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.182634888057113326614324442681892723e-4),
 740:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.578769494973505239894178121070843383e-5),
 741:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.493875893393627039981813418398565502e-9),
 742:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.105953670140260427338098566209633945e-5),
 743:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.616671437611040747858836254004890765e-6),
 744:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.175629733590604619378669693914265388e-6),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 745-768 / 第 745-768 行
~~~cpp
 745:    };
 746:    workspace[7] = tools::evaluate_polynomial(C7, z);
 747: 
 748:    BOOST_MATH_STATIC const T C8[] = {
 749:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000652623918595309418922034919726622692),
 750:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000839498720672087279993357516764983445),
 751:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000438297098541721005061087953050560377),
 752:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.696909145842055197136911097362072702e-6),
 753:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00016644846642067547837384572662326101),
 754:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000127835176797692185853344001461664247),
 755:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.462995326369130429061361032704489636e-4),
 756:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.455790986792270771162749294232219616e-8),
 757:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.105952711258051954718238500312872328e-4),
 758:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.678334290486516662273073740749269432e-5),
 759:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.210754766662588042469972680229376445e-5),
 760:    };
 761:    workspace[8] = tools::evaluate_polynomial(C8, z);
 762: 
 763:    BOOST_MATH_STATIC const T C9[] = {
 764:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000596761290192746250124390067179459605),
 765:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.720489541602001055908571930225015052e-4),
 766:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000678230883766732836161951166000673426),
 767:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000640147526026275845100045652582354779),
 768:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000277501076343287044992374518205845463),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 769-792 / 第 769-792 行
~~~cpp
 769:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.181970083804651510461686554030325202e-6),
 770:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.847950711706850318239732559632810086e-4),
 771:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.610519208250153101764709122740859458e-4),
 772:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.210739201834048624082975255893773306e-4),
 773:    };
 774:    workspace[9] = tools::evaluate_polynomial(C9, z);
 775: 
 776:    BOOST_MATH_STATIC const T C10[] = {
 777:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00133244544948006563712694993432717968),
 778:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00191443849856547752650089885832852254),
 779:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.0011089369134596637339607446329267522),
 780:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.993240412264229896742295262075817566e-6),
 781:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000508745012930931989848393025305956774),
 782:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00042735056665392884328432271160040444),
 783:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.000168588537679107988033552814662382059),
 784:    };
 785:    workspace[10] = tools::evaluate_polynomial(C10, z);
 786: 
 787:    BOOST_MATH_STATIC const T C11[] = {
 788:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00157972766073083495908785631307733022),
 789:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.000162516262783915816898635123980270998),
 790:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00206334210355432762645284467690276817),
 791:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00213896861856890981541061922797693947),
 792:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00101085593912630031708085801712479376),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 793-816 / 第 793-816 行
~~~cpp
 793:    };
 794:    workspace[11] = tools::evaluate_polynomial(C11, z);
 795: 
 796:    BOOST_MATH_STATIC const T C12[] = {
 797:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00407251211951401664727281097914544601),
 798:       BOOST_MATH_BIG_CONSTANT(T, 113, 0.00640336283380806979482363809026579583),
 799:       BOOST_MATH_BIG_CONSTANT(T, 113, -0.00404101610816766177473974858518094879),
 800:    };
 801:    workspace[12] = tools::evaluate_polynomial(C12, z);
 802:    workspace[13] = -0.0059475779383993002845382844736066323L;
 803: 
 804:    T result = tools::evaluate_polynomial(workspace, T(1/a));
 805:    result *= exp(-y) / sqrt(2 * constants::pi<T>() * a);
 806:    if(x < a)
 807:       result = -result;
 808: 
 809:    result += boost::math::erfc(sqrt(y), pol) / 2;
 810: 
 811:    return result;
 812: }
 813: // LCOV_EXCL_STOP
 814: 
 815: #endif
 816: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 817-823 / 第 817-823 行
~~~cpp
 817: }  // namespace detail
 818: }  // namespace math
 819: }  // namespace math
 820: 
 821: 
 822: #endif // BOOST_MATH_DETAIL_IGAMMA_LARGE
 823: 
~~~
- **EN:** This short range closes a conditional-compilation or header-guard region. The code enters namespace scope (detail::math) to keep symbols organized. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这一小段用于结束条件编译区域或头文件保护区域。 代码进入命名空间作用域（detail::math），以保持符号组织清晰。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/type_traits.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `BOOST_MATH_ASSERT, boost::math::log1pmx, sqrt, tools::evaluate_polynomial, exp`
