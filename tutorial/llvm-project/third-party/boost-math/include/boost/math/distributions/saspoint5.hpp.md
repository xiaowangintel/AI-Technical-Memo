# saspoint5.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/saspoint5.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the saspoint5 distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 saspoint5 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40 / 第 1-40 行
~~~cpp
   1: //  Copyright Takuma Yoshimura 2024.
   2: //  Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_STATS_SASPOINT5_HPP
   8: #define BOOST_STATS_SASPOINT5_HPP
   9: 
  10: #ifdef _MSC_VER
  11: #pragma warning(push)
  12: #pragma warning(disable : 4127) // conditional expression is constant
  13: #endif
  14: 
  15: #include <boost/math/tools/config.hpp>
  16: #include <boost/math/tools/numeric_limits.hpp>
  17: #include <boost/math/tools/tuple.hpp>
  18: #include <boost/math/constants/constants.hpp>
  19: #include <boost/math/tools/big_constant.hpp>
  20: #include <boost/math/distributions/complement.hpp>
  21: #include <boost/math/distributions/detail/common_error_handling.hpp>
  22: #include <boost/math/distributions/detail/derived_accessors.hpp>
  23: #include <boost/math/tools/rational.hpp>
  24: #include <boost/math/tools/promotion.hpp>
  25: #include <boost/math/tools/type_traits.hpp>
  26: #include <boost/math/policies/policy.hpp>
  27: #include <boost/math/policies/error_handling.hpp>
  28: 
  29: #ifndef BOOST_MATH_HAS_NVRTC
  30: #include <boost/math/distributions/fwd.hpp>
  31: #include <cmath>
  32: #endif
  33: 
  34: namespace boost { namespace math {
  35: template <class RealType, class Policy>
  36: class saspoint5_distribution;
  37: 
  38: namespace detail {
  39: 
  40: template <class RealType>
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/tuple.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/tuple.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 41-80 / 第 41-80 行
~~~cpp
  41: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_pdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53>&)
  42: {
  43:     BOOST_MATH_STD_USING
  44:     RealType result;
  45: 
  46:     if (x < 0.125) {
  47:         // Rational Approximation
  48:         // Maximum Relative Error: 7.8747e-17
  49:         BOOST_MATH_STATIC const RealType P[13] = {
  50:             static_cast<RealType>(6.36619772367581343076e-1),
  51:             static_cast<RealType>(2.17275699713513462507e2),
  52:             static_cast<RealType>(3.49063163361344578910e4),
  53:             static_cast<RealType>(3.40332906932698464252e6),
  54:             static_cast<RealType>(2.19485577044357440949e8),
  55:             static_cast<RealType>(9.66086435948730562464e9),
  56:             static_cast<RealType>(2.90571833690383003932e11),
  57:             static_cast<RealType>(5.83089315593106044683e12),
  58:             static_cast<RealType>(7.37911022713775715766e13),
  59:             static_cast<RealType>(5.26757196603002476852e14),
  60:             static_cast<RealType>(1.75780353683063527570e15),
  61:             static_cast<RealType>(1.85883041942144306222e15),
  62:             static_cast<RealType>(4.19828222275972713819e14),
  63:         };
  64:         BOOST_MATH_STATIC const RealType Q[15] = {
  65:             static_cast<RealType>(1.),
  66:             static_cast<RealType>(3.41295871011779138155e2),
  67:             static_cast<RealType>(5.48907134827349102297e4),
  68:             static_cast<RealType>(5.36641455324410261980e6),
  69:             static_cast<RealType>(3.48045461004960397915e8),
  70:             static_cast<RealType>(1.54920747349701741537e10),
  71:             static_cast<RealType>(4.76490595358644532404e11),
  72:             static_cast<RealType>(1.00104823128402735005e13),
  73:             static_cast<RealType>(1.39703522470411802507e14),
  74:             static_cast<RealType>(1.23724881334160220266e15),
  75:             static_cast<RealType>(6.47437580921138359461e15),
  76:             static_cast<RealType>(1.77627318260037604066e16),
  77:             static_cast<RealType>(2.04792815832538146160e16),
  78:             static_cast<RealType>(7.45102534638640681964e15),
  79:             static_cast<RealType>(3.68496090049571174527e14),
  80:         };
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 81-120 / 第 81-120 行
~~~cpp
  81: 
  82:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
  83:     }
  84:     else if (x < 0.25) {
  85:         RealType t = x - static_cast <RealType>(0.125);
  86: 
  87:         // Rational Approximation
  88:         // Maximum Relative Error: 2.1471e-17
  89:         BOOST_MATH_STATIC const RealType P[7] = {
  90:             static_cast<RealType>(4.35668401768623200524e-1),
  91:             static_cast<RealType>(7.12477357389655327116e0),
  92:             static_cast<RealType>(4.02466317948738993787e1),
  93:             static_cast<RealType>(9.04888497628205955839e1),
  94:             static_cast<RealType>(7.56175387288619211460e1),
  95:             static_cast<RealType>(1.26950253999694502457e1),
  96:             static_cast<RealType>(-6.59304802132933325219e-1),
  97:         };
  98:         BOOST_MATH_STATIC const RealType Q[7] = {
  99:             static_cast<RealType>(1.),
 100:             static_cast<RealType>(1.98623818041545101115e1),
 101:             static_cast<RealType>(1.52856383017632616759e2),
 102:             static_cast<RealType>(5.70706902111659740041e2),
 103:             static_cast<RealType>(1.06454927680197927878e3),
 104:             static_cast<RealType>(9.13160352749764887791e2),
 105:             static_cast<RealType>(2.58872466837209126618e2),
 106:         };
 107: 
 108:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 109:     }
 110:     else if (x < 0.5) {
 111:         RealType t = x - static_cast <RealType>(0.25);
 112: 
 113:         // Rational Approximation
 114:         // Maximum Relative Error: 5.3265e-17
 115:         BOOST_MATH_STATIC const RealType P[8] = {
 116:             static_cast<RealType>(2.95645445681747568732e-1),
 117:             static_cast<RealType>(2.23779537590791610124e0),
 118:             static_cast<RealType>(5.01302198171248036052e0),
 119:             static_cast<RealType>(2.76363131116340641935e0),
 120:             static_cast<RealType>(1.18134858311074670327e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 121-160 / 第 121-160 行
~~~cpp
 121:             static_cast<RealType>(2.00287083462139382715e-2),
 122:             static_cast<RealType>(-7.53979800555375661516e-3),
 123:             static_cast<RealType>(1.37294648777729527395e-3),
 124:         };
 125:         BOOST_MATH_STATIC const RealType Q[6] = {
 126:             static_cast<RealType>(1.),
 127:             static_cast<RealType>(1.02879626214781666701e1),
 128:             static_cast<RealType>(3.85125274509784615691e1),
 129:             static_cast<RealType>(6.18474367367800231625e1),
 130:             static_cast<RealType>(3.77100050087302476029e1),
 131:             static_cast<RealType>(5.41866360740066443656e0),
 132:         };
 133: 
 134:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 135:     }
 136:     else if (x < 1) {
 137:         RealType t = x - static_cast <RealType>(0.5);
 138: 
 139:         // Rational Approximation
 140:         // Maximum Relative Error: 2.7947e-17
 141:         BOOST_MATH_STATIC const RealType P[7] = {
 142:             static_cast<RealType>(1.70762401725206223811e-1),
 143:             static_cast<RealType>(8.43343631021918972436e-1),
 144:             static_cast<RealType>(1.39703819152564365627e0),
 145:             static_cast<RealType>(8.75843324574692085009e-1),
 146:             static_cast<RealType>(1.86199552443747562584e-1),
 147:             static_cast<RealType>(7.35858280181579907616e-3),
 148:             static_cast<RealType>(-1.03693607694266081126e-4),
 149:         };
 150:         BOOST_MATH_STATIC const RealType Q[7] = {
 151:             static_cast<RealType>(1.),
 152:             static_cast<RealType>(6.73363440952557318819e0),
 153:             static_cast<RealType>(1.74288966619209299976e1),
 154:             static_cast<RealType>(2.15943268035083671893e1),
 155:             static_cast<RealType>(1.29818726981381859879e1),
 156:             static_cast<RealType>(3.40707211426946022041e0),
 157:             static_cast<RealType>(2.80229012541729457678e-1),
 158:         };
 159: 
 160:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 161-200 / 第 161-200 行
~~~cpp
 161:     }
 162:     else if (x < 2) {
 163:         RealType t = x - 1;
 164: 
 165:         // Rational Approximation
 166:         // Maximum Relative Error: 1.7051e-18
 167:         BOOST_MATH_STATIC const RealType P[7] = {
 168:             static_cast<RealType>(8.61071469126041183247e-2),
 169:             static_cast<RealType>(1.69689585946245345838e-1),
 170:             static_cast<RealType>(1.09494833291892212033e-1),
 171:             static_cast<RealType>(2.76619622453130604637e-2),
 172:             static_cast<RealType>(2.44972748006913061509e-3),
 173:             static_cast<RealType>(4.09853605772288438003e-5),
 174:             static_cast<RealType>(-2.63561415158954865283e-7),
 175:         };
 176:         BOOST_MATH_STATIC const RealType Q[7] = {
 177:             static_cast<RealType>(1.),
 178:             static_cast<RealType>(3.04082856018856244947e0),
 179:             static_cast<RealType>(3.52558663323956252986e0),
 180:             static_cast<RealType>(1.94795523079701426332e0),
 181:             static_cast<RealType>(5.23956733400745421623e-1),
 182:             static_cast<RealType>(6.19453597593998871667e-2),
 183:             static_cast<RealType>(2.31061984192347753499e-3),
 184:         };
 185: 
 186:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 187:     }
 188:     else if (x < 4) {
 189:         RealType t = x - 2;
 190: 
 191:         // Rational Approximation
 192:         // Maximum Relative Error: 2.9247e-17
 193:         BOOST_MATH_STATIC const RealType P[7] = {
 194:             static_cast<RealType>(3.91428580496513429479e-2),
 195:             static_cast<RealType>(4.07162484034780126757e-2),
 196:             static_cast<RealType>(1.43342733342753081931e-2),
 197:             static_cast<RealType>(2.01622178115394696215e-3),
 198:             static_cast<RealType>(1.00648013467757737201e-4),
 199:             static_cast<RealType>(9.51545046750892356441e-7),
 200:             static_cast<RealType>(-3.56598940936439037087e-9),
~~~
- **EN:** This range declares or defines callable logic such as if, tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 if, tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 201-240 / 第 201-240 行
~~~cpp
 201:         };
 202:         BOOST_MATH_STATIC const RealType Q[7] = {
 203:             static_cast<RealType>(1.),
 204:             static_cast<RealType>(1.63904431617187026619e0),
 205:             static_cast<RealType>(1.03812003196677309121e0),
 206:             static_cast<RealType>(3.18144310790210668797e-1),
 207:             static_cast<RealType>(4.81930155615666517263e-2),
 208:             static_cast<RealType>(3.25435391589941361778e-3),
 209:             static_cast<RealType>(7.01626957128181647457e-5),
 210:         };
 211: 
 212:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 213:     }
 214:     else if (x < 8) {
 215:         RealType t = x - 4;
 216: 
 217:         // Rational Approximation
 218:         // Maximum Relative Error: 2.6547e-17
 219:         BOOST_MATH_STATIC const RealType P[7] = {
 220:             static_cast<RealType>(1.65057384221262866484e-2),
 221:             static_cast<RealType>(8.05429762031495873704e-3),
 222:             static_cast<RealType>(1.35249234647852784985e-3),
 223:             static_cast<RealType>(9.18685252682786794440e-5),
 224:             static_cast<RealType>(2.23447790937806602674e-6),
 225:             static_cast<RealType>(1.03176916111395079569e-8),
 226:             static_cast<RealType>(-1.94913182592441292094e-11),
 227:         };
 228:         BOOST_MATH_STATIC const RealType Q[7] = {
 229:             static_cast<RealType>(1.),
 230:             static_cast<RealType>(8.10113554189626079232e-1),
 231:             static_cast<RealType>(2.54175325409968367580e-1),
 232:             static_cast<RealType>(3.87119072807894983910e-2),
 233:             static_cast<RealType>(2.92520770162792443587e-3),
 234:             static_cast<RealType>(9.89094130526684467420e-5),
 235:             static_cast<RealType>(1.07148513311070719488e-6),
 236:         };
 237: 
 238:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 239:     }
 240:     else if (x < 16) {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 241-280 / 第 241-280 行
~~~cpp
 241:         RealType t = x - 8;
 242: 
 243:         // Rational Approximation
 244:         // Maximum Relative Error: 2.5484e-17
 245:         BOOST_MATH_STATIC const RealType P[7] = {
 246:             static_cast<RealType>(6.60044810497290557553e-3),
 247:             static_cast<RealType>(1.59342644994950292031e-3),
 248:             static_cast<RealType>(1.32429706922966110874e-4),
 249:             static_cast<RealType>(4.45378136978435909660e-6),
 250:             static_cast<RealType>(5.36409958111394628239e-8),
 251:             static_cast<RealType>(1.22293787679910067873e-10),
 252:             static_cast<RealType>(-1.16300443044165216564e-13),
 253:         };
 254:         BOOST_MATH_STATIC const RealType Q[7] = {
 255:             static_cast<RealType>(1.),
 256:             static_cast<RealType>(4.10446485803039594111e-1),
 257:             static_cast<RealType>(6.51887342399859289520e-2),
 258:             static_cast<RealType>(5.02151225308643905366e-3),
 259:             static_cast<RealType>(1.91741179639551137839e-4),
 260:             static_cast<RealType>(3.27316600311598190022e-6),
 261:             static_cast<RealType>(1.78840301213102212857e-8),
 262:         };
 263: 
 264:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 265:     }
 266:     else if (x < 32) {
 267:         RealType t = x - 16;
 268: 
 269:         // Rational Approximation
 270:         // Maximum Relative Error: 2.9866e-17
 271:         BOOST_MATH_STATIC const RealType P[7] = {
 272:             static_cast<RealType>(2.54339461777955741686e-3),
 273:             static_cast<RealType>(3.10069525357852579756e-4),
 274:             static_cast<RealType>(1.30082682796085732756e-5),
 275:             static_cast<RealType>(2.20715868479255585050e-7),
 276:             static_cast<RealType>(1.33996659756026452288e-9),
 277:             static_cast<RealType>(1.53505360463827994365e-12),
 278:             static_cast<RealType>(-7.42649416356965421308e-16),
 279:         };
 280:         BOOST_MATH_STATIC const RealType Q[7] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 281-320 / 第 281-320 行
~~~cpp
 281:             static_cast<RealType>(1.),
 282:             static_cast<RealType>(2.09203384450859785642e-1),
 283:             static_cast<RealType>(1.69422626897631306130e-2),
 284:             static_cast<RealType>(6.65649059670689720386e-4),
 285:             static_cast<RealType>(1.29654785666009849481e-5),
 286:             static_cast<RealType>(1.12886139474560969619e-7),
 287:             static_cast<RealType>(3.14420104899170413840e-10),
 288:         };
 289: 
 290:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 291:     }
 292:     else if (x < 64) {
 293:         RealType t = x - 32;
 294: 
 295:         // Rational Approximation
 296:         // Maximum Relative Error: 3.3581e-17
 297:         BOOST_MATH_STATIC const RealType P[7] = {
 298:             static_cast<RealType>(9.55085695067883584460e-4),
 299:             static_cast<RealType>(5.86125496733202756668e-5),
 300:             static_cast<RealType>(1.23753971325810931282e-6),
 301:             static_cast<RealType>(1.05643819745933041408e-8),
 302:             static_cast<RealType>(3.22502949410095015524e-11),
 303:             static_cast<RealType>(1.85366144680157942079e-14),
 304:             static_cast<RealType>(-4.53975807317403152058e-18),
 305:         };
 306:         BOOST_MATH_STATIC const RealType Q[7] = {
 307:             static_cast<RealType>(1.),
 308:             static_cast<RealType>(1.05980850386474826374e-1),
 309:             static_cast<RealType>(4.34966042652000070674e-3),
 310:             static_cast<RealType>(8.66341538387446465700e-5),
 311:             static_cast<RealType>(8.55608082202236124363e-7),
 312:             static_cast<RealType>(3.77719968378509293354e-9),
 313:             static_cast<RealType>(5.33287361559571716670e-12),
 314:         };
 315: 
 316:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 317:     }
 318:     else {
 319:         RealType t = 1 / sqrt(x);
 320: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 321-360 / 第 321-360 行
~~~cpp
 321:         // Rational Approximation
 322:         // Maximum Relative Error: 4.7450e-19
 323:         BOOST_MATH_STATIC const RealType P[5] = {
 324:             static_cast<RealType>(1.99471140200716338970e-1),
 325:             static_cast<RealType>(-1.93310094131437487158e-2),
 326:             static_cast<RealType>(-8.44282614309073196195e-3),
 327:             static_cast<RealType>(3.47296024282356038069e-3),
 328:             static_cast<RealType>(-4.05398011689821941383e-4),
 329:         };
 330:         BOOST_MATH_STATIC const RealType Q[5] = {
 331:             static_cast<RealType>(1.),
 332:             static_cast<RealType>(7.00973251258577238892e-1),
 333:             static_cast<RealType>(2.66969681258835723157e-1),
 334:             static_cast<RealType>(5.51785147503612200456e-2),
 335:             static_cast<RealType>(6.50130030979966274341e-3),
 336:         };
 337: 
 338:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t / x;
 339:     }
 340: 
 341:     return result;
 342: }
 343: 
 344: 
 345: template <class RealType>
 346: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_pdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>&)
 347: {
 348:     BOOST_MATH_STD_USING
 349:     RealType result;
 350: 
 351:     if (x < 0.0625) {
 352:         // Rational Approximation
 353:         // Maximum Relative Error: 8.8841e-35
 354:         // LCOV_EXCL_START
 355:         BOOST_MATH_STATIC const RealType P[27] = {
 356:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.36619772367581343075535053490057448138e-1),
 357:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.57459506929453385798277946154823008327e2),
 358:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.46717322844023441698710451505816706570e5),
 359:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71501459971530549476153273173061194095e8),
 360:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.76700973495278431084530045707075552432e10),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 361-400 / 第 361-400 行
~~~cpp
 361:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.01328150775099946510145440412520620021e13),
 362:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70028222513668830210058353057559790101e15),
 363:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.29641781943744384078006991488193839955e17),
 364:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.52611994112742436432957758588495082163e19),
 365:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.27833177267552931459542318826727288124e21),
 366:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.68946162731840551853993619351896931533e23),
 367:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02965010233956763504899745874128908220e25),
 368:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.14128569264874914146628076133997950655e26),
 369:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.09103580386900060922163883603492216942e28),
 370:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.86778299087452621293332172137014749128e29),
 371:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.80029712249744334924217328667885673985e31),
 372:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.70890080432228368476255091774238573277e32),
 373:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.88600513999992354909078399482884993261e33),
 374:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.01189178534848836605739139176681647755e34),
 375:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.06531475170803043941021113424602440078e35),
 376:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.64956999370443524098457423629252855270e36),
 377:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44276098283517934229787916584447559248e37),
 378:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.45856704224433991524661028965741649584e37),
 379:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.47263237190968408624388275549716907309e37),
 380:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.66186300951901408251743228798832386260e37),
 381:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.48064966533519934186356663849904556319e36),
 382:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.64877082086372991309408001661535573441e35),
 383:         };
 384:         BOOST_MATH_STATIC const RealType Q[28] = {
 385:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 386:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18981461118065892086304195732751798634e3),
 387:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.01761929839041982958990681130944341399e5),
 388:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.69465252239913021973760046507387620537e8),
 389:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.49221044103838155300076098325950584061e10),
 390:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59327386289821190042576978177896481082e13),
 391:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.67528179803224728786405503232064643870e15),
 392:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.61672367849271591791062829736720884633e17),
 393:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.98395893917909208201801908435620016552e19),
 394:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.60025693881358827551113845076726845495e21),
 395:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.67730578745705562356709169493821118109e23),
 396:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.63843883526710042156562706339553092312e25),
 397:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.23075214698024188140971761421762265880e26),
 398:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.37775321923937393366376907114580842429e28),
 399:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12444724625354796650300159037364355605e30),
 400:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.00860835602766063447009568106012449767e31),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 401-440 / 第 401-440 行
~~~cpp
 401:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.39614080159468893509273006948526469708e32),
 402:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06547095715472468415058181351212520255e34),
 403:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.36755997709303811764051969789337337957e35),
 404:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.32519530489892818585066019217287415587e36),
 405:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.45230390606834183602522256278256501404e36),
 406:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.80344475131699029428900627020022801971e37),
 407:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.66469314795307459840482483320814279444e38),
 408:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.70209065673736156218117594311801487932e38),
 409:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.84490531246108754748100009460860427732e38),
 410:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.30215083398643966091721732133851539475e38),
 411:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.58032845332990262754766784625271262271e37),
 412:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.50461648438613634025964361513066059697e36),
 413:         };
 414:         // LCOV_EXCL_STOP
 415:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
 416:     }
 417:     else if (x < 0.125) {
 418:         RealType t = x - static_cast <RealType>(0.0625);
 419: 
 420:         // Rational Approximation
 421:         // Maximum Relative Error: 3.4585e-35
 422:         // LCOV_EXCL_START
 423:         BOOST_MATH_STATIC const RealType P[14] = {
 424:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.46416716200748206779925127900698754119e-1),
 425:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.41771273526123373239570033672829787791e1),
 426:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.56142610225585235535211648703534340871e3),
 427:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.15655694129872563686497490176725921724e4),
 428:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.00791883661952751945853742455643714995e5),
 429:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.30252591667828615354689186280704562254e6),
 430:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.76168115448224677276551213052798322583e7),
 431:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.88534624532179841393387625270218172719e7),
 432:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14740447137831585842166880265350244623e8),
 433:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14904082614021239315925958812100948136e8),
 434:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.76866867279164114004579652405104553404e7),
 435:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.53475339598769347326916978463911377965e6),
 436:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.88896160275915786487519266368539625326e5),
 437:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.05543800791717482823610940401201712196e4),
 438:         };
 439:         BOOST_MATH_STATIC const RealType Q[13] = {
 440:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 441-480 / 第 441-480 行
~~~cpp
 441:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.44407579416524903840331499438398472639e1),
 442:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.15911780811299460009161345260146251462e3),
 443:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.88457596285725454686358792906273558406e4),
 444:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.66501639812506059997744549411633476528e5),
 445:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.12674134216028769532305433586266118000e6),
 446:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.87676063477990584593444083577765264392e7),
 447:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.56084282739608760299329382263598821653e8),
 448:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.34250986378665047914811630036201995871e8),
 449:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31288233106689286803200674021353188597e9),
 450:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33621494302241474082474689597125896975e9),
 451:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.63379428046258653791600947328520263412e8),
 452:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14558538557562267533922961110917101850e8),
 453:         };
 454:         // LCOV_EXCL_STOP
 455:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 456:     }
 457:     else if (x < 0.25) {
 458:         RealType t = x - static_cast <RealType>(0.125);
 459: 
 460:         // Rational Approximation
 461:         // Maximum Relative Error: 6.9278e-35
 462:         // LCOV_EXCL_START
 463:         BOOST_MATH_STATIC const RealType P[14] = {
 464:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.35668401768623200524372663239480799018e-1),
 465:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30066509937988171489091367354416214000e1),
 466:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.05924026744937322690717755156090122074e2),
 467:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.12998524955326375684693500551926325112e3),
 468:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.52237930808361186011042950178715609183e4),
 469:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.10734809597587633852077152938985998879e5),
 470:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.20796157836149826988172603622242119074e5),
 471:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12398478061053302537736799402801934778e6),
 472:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17841330491647012385157454335820786724e6),
 473:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.46281413765362795389526259057436151953e5),
 474:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52220357379402116641048490644093497829e5),
 475:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.51130316105543847380510577656570543736e3),
 476:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.32201781975497810173532067354797097401e2),
 477:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.96874547436310030183519174847668703774e0),
 478:         };
 479:         BOOST_MATH_STATIC const RealType Q[13] = {
 480:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 481-520 / 第 481-520 行
~~~cpp
 481:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.63164311578114868477819520857286165076e1),
 482:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34964379844144961683927306966955217328e3),
 483:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.82966031793809959278519002412667883288e4),
 484:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.56215285850856046267451500310816276675e5),
 485:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.81046679663412610005501878092824281161e5),
 486:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.33868038251479411246071640628518434659e6),
 487:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.46262495881941625571640264458627940579e6),
 488:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40052628730443097561652737049917920495e7),
 489:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44394803828297754346261138417756941544e7),
 490:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.56647617803506258343236509255155360957e6),
 491:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.53513095899009948733175317927025056561e6),
 492:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.69130675750530663088963759279778748696e5),
 493:         };
 494:         // LCOV_EXCL_STOP
 495:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 496:     }
 497:     else if (x < 0.5) {
 498:         RealType t = x - static_cast <RealType>(0.25);
 499: 
 500:         // Rational Approximation
 501:         // Maximum Relative Error: 6.9378e-36
 502:         // LCOV_EXCL_START
 503:         BOOST_MATH_STATIC const RealType P[14] = {
 504:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.95645445681747568731488283573032414811e-1),
 505:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.83246437763964151893665752064650172391e0),
 506:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.85333417559435252576820440080930004674e1),
 507:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.90974714199542064991001365628659054084e2),
 508:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.39707205668285805800884524044738261436e3),
 509:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.24814598419826565698241508792385416075e3),
 510:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.95012897118808793886195172068123345314e3),
 511:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.87265743900139300849404272909665705025e3),
 512:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.98795164648056126707212245325405968413e3),
 513:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07012128790318535418330629467906917213e3),
 514:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.99797198893523173981812955075412130913e2),
 515:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.55029227544167913873724286459253168886e1),
 516:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.54064889901609722583601330171719819660e-1),
 517:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.72254289950537680833853394958874977464e-3),
 518:         };
 519:         BOOST_MATH_STATIC const RealType Q[14] = {
 520:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 521-560 / 第 521-560 行
~~~cpp
 521:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.58291085070053442257438623486099473087e1),
 522:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.95618461039379226195473938654286975682e2),
 523:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.97427161745150579714266897556974326502e3),
 524:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.52730436681412535198281529590508861106e3),
 525:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.49521185356761585062135933350225236726e4),
 526:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.03881178612341724262911142022761966061e4),
 527:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.02360046338629039644581819847209730553e4),
 528:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.65580339066083507998465454599272345735e4),
 529:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.15462499626138125314518636645472893045e4),
 530:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.61951767959774678843021179589300545717e4),
 531:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.60745557054877240279811529503888551492e3),
 532:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.91061555870569579915258835459255406575e2),
 533:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43045229010040855016672246098687100063e1),
 534:         };
 535:         // LCOV_EXCL_STOP
 536:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 537:     }
 538:     else if (x < 1) {
 539:         RealType t = x - static_cast <RealType>(0.5);
 540: 
 541:         // Rational Approximation
 542:         // Maximum Relative Error: 6.4363e-35
 543:         // LCOV_EXCL_START
 544:         BOOST_MATH_STATIC const RealType P[14] = {
 545:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70762401725206223811383500786268939645e-1),
 546:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19353011456197635663058525904929358535e0),
 547:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.22974648900600015961253465796487372402e1),
 548:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.91951696059042324975935209295355569292e1),
 549:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.79039444119906169910281912009369164227e1),
 550:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00089963120992100860902142265631127046e2),
 551:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.37108883429306700857182028809960789020e1),
 552:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.49586566873564432788366931251358248417e1),
 553:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49790521605774884174840168128255220471e1),
 554:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.90660338063979435668763608259382712726e0),
 555:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.93409982383888149064797608605579930804e-1),
 556:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.22802459215932860445033185874876812040e-2),
 557:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07739227340181463034286653569468171767e-4),
 558:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.02669738424010290973023004028523684766e-7),
 559:         };
 560:         BOOST_MATH_STATIC const RealType Q[14] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 561-600 / 第 561-600 行
~~~cpp
 561:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 562:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.46404480283267324138113869370306506431e1),
 563:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.54550184643308468933661600211579108422e1),
 564:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.63602410602063476726031476852965502123e2),
 565:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.94463479638213888403144706176973026333e2),
 566:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.48607087483870766806529883069123352339e3),
 567:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.69715692924508994524755312953665710218e3),
 568:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33237849965272853370191827043868842100e3),
 569:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.08460086451666825383009487734769646087e2),
 570:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.47365552394788536087148438788608689300e2),
 571:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.38010282703940184371247559455167674975e1),
 572:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.67219842525655806370702248122668214685e0),
 573:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.01852843874982199859775136086676841910e-1),
 574:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.14767043526088185802569803397824432028e-3),
 575:         };
 576:         // LCOV_EXCL_STOP
 577:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 578:     }
 579:     else if (x < 2) {
 580:         RealType t = x - 1;
 581: 
 582:         // Rational Approximation
 583:         // Maximum Relative Error: 9.1244e-35
 584:         // LCOV_EXCL_START
 585:         BOOST_MATH_STATIC const RealType P[13] = {
 586:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.61071469126041183247373313827161939454e-2),
 587:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.35837460186564880289965856498718321896e-1),
 588:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.47783071967681246738651796742079530382e-1),
 589:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16019502727107539284403003943433359877e0),
 590:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.80510046274709592896987229782879937271e-1),
 591:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.30456542768955299533391113704078540955e-1),
 592:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.36539167913428133313942008990965988621e-1),
 593:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76450743657913389896743235938695682829e-2),
 594:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.42847090205575096649865021874905747106e-3),
 595:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.41380341540026027117735179862124402398e-4),
 596:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.40549721587212773424211923602910622515e-6),
 597:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09089653391032945883918434200567278139e-7),
 598:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.21403900721572475664926557233205232491e-10),
 599:         };
 600:         BOOST_MATH_STATIC const RealType Q[14] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 601-640 / 第 601-640 行
~~~cpp
 601:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 602:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.13172035933794917563324458011617112124e0),
 603:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.65687100738157412154132860910003018338e1),
 604:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.59672433683883998168388916533196510994e1),
 605:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.61469557815097583209668778301921207455e1),
 606:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.77070955301136405523492329700943077340e1),
 607:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.20825570431301943907348077675777546304e0),
 608:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.60136197167727810483751794121979805142e0),
 609:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.53723076053642006159503073104152703814e-1),
 610:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.63397465217490984394478518334313362490e-2),
 611:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.40577918603319523990542237990107206371e-3),
 612:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.94376458316662573143947719026985667328e-4),
 613:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.09333568224541559157192543410988474886e-6),
 614:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.59947287428695057506683902409023760438e-8),
 615:         };
 616:         // LCOV_EXCL_STOP
 617:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 618:     }
 619:     else if (x < 4) {
 620:         RealType t = x - 2;
 621: 
 622:         // Rational Approximation
 623:         // Maximum Relative Error: 8.1110e-35
 624:         // LCOV_EXCL_START
 625:         BOOST_MATH_STATIC const RealType P[13] = {
 626:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.91428580496513429479068747515164587814e-2),
 627:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.69015019070193436467106672180804948494e-2),
 628:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.03147451266231819912643754579290008651e-1),
 629:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.18825170881552297150779588545792258740e-2),
 630:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30548850262278582401286533053286406505e-2),
 631:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.54315108501815531776138839512564427279e-3),
 632:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.66434584176931077662201101557716482514e-4),
 633:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.66158632576958238392567355014249971287e-5),
 634:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.31365802206301246598393821671437863818e-6),
 635:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.85378389166807263837732376845556856416e-7),
 636:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.20375363151456683883984823721339648679e-9),
 637:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06637401794693307359898089790558771957e-11),
 638:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.08663671047376684678494625068451888284e-14),
 639:         };
 640:         BOOST_MATH_STATIC const RealType Q[14] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 641-680 / 第 641-680 行
~~~cpp
 641:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 642:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.07443397096591141329212291707948432414e0),
 643:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.16665031056584124503224711639009530348e0),
 644:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.27666060511630720485121299731204403783e0),
 645:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.65646979169107732387032821262953301311e0),
 646:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.63594064986880863092994744424349361396e-1),
 647:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31360114173642293100378020953197965181e-1),
 648:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.09489929949457075237756409511944811481e-2),
 649:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.24574519309785870806550506199124944514e-3),
 650:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.56048486483867679310086683710523566607e-4),
 651:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.60417286783794818094722636906776809193e-6),
 652:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.53154117367296710469692755461431646999e-7),
 653:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.60041713691072903334637560080298818163e-9),
 654:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.77381528950794767694352468734042252745e-12),
 655:         };
 656:         // LCOV_EXCL_STOP
 657:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 658:     }
 659:     else if (x < 8) {
 660:         RealType t = x - 4;
 661: 
 662:         // Rational Approximation
 663:         // Maximum Relative Error: 2.5228e-35
 664:         // LCOV_EXCL_START
 665:         BOOST_MATH_STATIC const RealType P[14] = {
 666:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.65057384221262866484014802392420311075e-2),
 667:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.92801687242885330588201777283015178448e-2),
 668:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.65508815862861196424333614846876229064e-3),
 669:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.71545573465295958468808641544341412235e-3),
 670:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.72077718130407940498710469661947719216e-4),
 671:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.26299620525538984108147098966692839348e-5),
 672:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.77971404992990847565880351976461271350e-6),
 673:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71176235845517643695464740679643640241e-7),
 674:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.64603919225244695533557520384631958897e-9),
 675:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.85274347406803894317891882905083368489e-11),
 676:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.48564096627181435612831469651920186491e-13),
 677:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.90886715044580341917806394089282500340e-16),
 678:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.39396206221935864416563232680283312796e-19),
 679:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.37760675743046300528308203869876086823e-22),
 680:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 681-720 / 第 681-720 行
~~~cpp
 681:         BOOST_MATH_STATIC const RealType Q[13] = {
 682:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 683:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49023284463742780238035958819642738891e0),
 684:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.76284367953836866133894756472541395734e-1),
 685:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.69932155343422362573146811195224195135e-1),
 686:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.97593541520549770519034085640975455763e-2),
 687:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.45862809001322359249894968573830094537e-2),
 688:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.61348135835522976885804369721316193713e-3),
 689:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21069949470458047530981551232427019037e-4),
 690:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.03132437580490629136144285669590192597e-6),
 691:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.91030348024641585284338958059030520141e-7),
 692:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.56320479309161046934628280237629402373e-9),
 693:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.39524198476052364627683067034422502163e-11),
 694:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18666081063885228839052386515073873844e-13),
 695:         };
 696:         // LCOV_EXCL_STOP
 697:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 698:     }
 699:     else if (x < 16) {
 700:         RealType t = x - 8;
 701: 
 702:         // Rational Approximation
 703:         // Maximum Relative Error: 9.6732e-36
 704:         // LCOV_EXCL_START
 705:         BOOST_MATH_STATIC const RealType P[14] = {
 706:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.60044810497290557552736366450372523266e-3),
 707:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.27034183360438185616541260923634443241e-3),
 708:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19813403884333707962156711479716066536e-3),
 709:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.91346554854771687970018076643044998737e-4),
 710:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.91975837766081548424458764226669789039e-5),
 711:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.26031304514411902758114277797443618334e-6),
 712:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.47127194811140370123712253347211626753e-8),
 713:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.55248861254135821097921903190564312000e-9),
 714:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.78340847719683652633864722047250151066e-11),
 715:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.95888612422041337572422846394029849086e-13),
 716:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.66363005792960308636467394552324255493e-15),
 717:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.93244800648299424751906591077496534948e-18),
 718:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.95046217952146113063614290717113024410e-21),
 719:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.46784746963816915795587433372284530785e-25),
 720:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 721-760 / 第 721-760 行
~~~cpp
 721:         BOOST_MATH_STATIC const RealType Q[14] = {
 722:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 723:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.16012189991825507132967712656930682478e-1),
 724:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.95202772611563835130347051925062280272e-1),
 725:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.23801477561401113332870463345197159418e-2),
 726:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.54022665579711946784722766000062263305e-3),
 727:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.94266182294627770206082679848878391116e-4),
 728:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.11782839184878848480753630961211685630e-5),
 729:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.28827067686094594197542725283923947812e-6),
 730:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.00220719177374237332018587370837457299e-8),
 731:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.42250513143925626748132661121749401409e-9),
 732:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.82007216963767723991309138907689681422e-11),
 733:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34214834652884406013489167210936679359e-13),
 734:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.85519293212465087373898447546710143008e-16),
 735:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.96728437809303144188312623363453475831e-19),
 736:         };
 737:         // LCOV_EXCL_STOP
 738:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 739:     }
 740:     else if (x < 32) {
 741:         RealType t = x - 16;
 742: 
 743:         // Rational Approximation
 744:         // Maximum Relative Error: 1.0113e-35
 745:         // LCOV_EXCL_START
 746:         BOOST_MATH_STATIC const RealType P[14] = {
 747:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.54339461777955741686401041938275102207e-3),
 748:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.17747085249877439037826121862689145081e-4),
 749:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14104576580586095462211756659036062930e-4),
 750:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.06903778663262313120049231822412184382e-6),
 751:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.53115958954246158081703822428768781010e-7),
 752:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.48225007017630665357941682179157662142e-8),
 753:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.20810829523286181556951002345409843125e-10),
 754:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.54070972719909957155251432996372246019e-12),
 755:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.06258623970363729581390609798632080752e-14),
 756:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.15603641527498625694677136504611545743e-16),
 757:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.05376970060354261667000502105893106009e-19),
 758:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.14727542705613448694396750352455931731e-22),
 759:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.76883960167449461476228984331517762578e-25),
 760:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.03558202009465610972808653993060437679e-29),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 761-800 / 第 761-800 行
~~~cpp
 761:         };
 762:         BOOST_MATH_STATIC const RealType Q[14] = {
 763:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 764:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.08809672969012756295937194823378109391e-1),
 765:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.41148083436617376855422685448827300528e-2),
 766:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.85101541143091590863368934606849033688e-3),
 767:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.38984899982960112626157576750593711628e-4),
 768:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.51437845497783812562009857096371643785e-5),
 769:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.12891276596072815764119699444334380521e-7),
 770:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.82412500887161687329929693518498698716e-8),
 771:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.80215715026891688444965605768621763721e-10),
 772:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.85838684678780184082810752634454259831e-12),
 773:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83675729736846176693608812315852523556e-14),
 774:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.80347165008408134158968403924819637224e-17),
 775:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23639219622240634094606955067799349447e-19),
 776:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.63446235885036169537726818244420509024e-23),
 777:         };
 778:         // LCOV_EXCL_STOP
 779:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 780:     }
 781:     else if (x < 64) {
 782:         RealType t = x - 32;
 783: 
 784:         // Rational Approximation
 785:         // Maximum Relative Error: 9.7056e-36
 786:         // LCOV_EXCL_START
 787:         BOOST_MATH_STATIC const RealType P[14] = {
 788:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.55085695067883584460317653567009454037e-4),
 789:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52919532248638251721278667010429548877e-4),
 790:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06266842295477991789450356745903177571e-5),
 791:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.20671609948319334255323512011575892813e-7),
 792:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04692714549374449244320605137676408001e-8),
 793:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70605481454469287545965803970738264158e-10),
 794:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83960996572005209177458712170004097587e-12),
 795:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.29732261733491885750067029092181853751e-14),
 796:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.78385693918239619309147428897790440735e-17),
 797:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52969197316398995616879018998891661712e-19),
 798:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.14063120299947677255281707434419044806e-22),
 799:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.25957675329657493245893497219459256248e-25),
 800:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.55238112862817593053765898004447484717e-29),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 801-840 / 第 801-840 行
~~~cpp
 801:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.93970406521541790658675747195982964585e-34),
 802:         };
 803:         BOOST_MATH_STATIC const RealType Q[14] = {
 804:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 805:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.04722757068068234153968603374387493579e-1),
 806:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.85854131835804458353300285777969427206e-2),
 807:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.85809281481040288085436275150792074968e-4),
 808:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.38860750164285700051427698379841626305e-5),
 809:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.91463283601681120487987016215594255423e-7),
 810:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.28104952818420195583669572450494959042e-8),
 811:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43912720109615655035554724090181888734e-10),
 812:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.10668954229813492117417896681856998595e-12),
 813:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.65093571330749369067212003571435698558e-15),
 814:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.81758227619561958470583781325371429458e-17),
 815:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.36970757752002915423191164330598255294e-20),
 816:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.06487673393164724939989217811068656932e-23),
 817:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.47121057452822097779067717258050172115e-27),
 818:         };
 819:         // LCOV_EXCL_STOP
 820:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 821:     }
 822:     else {
 823:         RealType t = 1 / sqrt(x);
 824: 
 825:         // Rational Approximation
 826:         // Maximum Relative Error: 7.1032e-36
 827:         // LCOV_EXCL_START
 828:         BOOST_MATH_STATIC const RealType P[8] = {
 829:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.99471140200716338969973029967190934238e-1),
 830:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.82846732476244747063962056024672844211e-2),
 831:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.69724475658159099827638225237895868258e-3),
 832:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.21259630917863228526439367416146293173e-3),
 833:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.13469812721679130825429547254346177005e-5),
 834:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.73237434182338329541631611908947123606e-5),
 835:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.72986150007117100707304201395140411630e-6),
 836:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.53567129749337040254350979652515879881e-7),
 837:         };
 838:         BOOST_MATH_STATIC const RealType Q[9] = {
 839:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 840:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.89815449697874475254942178935516387239e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 841-880 / 第 841-880 行
~~~cpp
 841:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.21223228867921988134838870379132038419e-1),
 842:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.79514417558927397512722128659468888701e-1),
 843:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.43331254539687594239741585764730095049e-2),
 844:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.99078779616201786316256750758748178864e-3),
 845:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04590833634768023225748107112347131311e-3),
 846:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.17497990182339853998751740288392648984e-5),
 847:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.53420609011698705803549938558385779137e-6),
 848:         };
 849:         // LCOV_EXCL_STOP
 850:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t / x;
 851:     }
 852: 
 853:     return result;
 854: }
 855: 
 856: 
 857: template <class RealType>
 858: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_pdf_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53> &tag) {
 859:     BOOST_MATH_STD_USING // for ADL of std functions
 860: 
 861:     return saspoint5_pdf_plus_imp_prec<RealType>(abs(x), tag);
 862: }
 863: 
 864: template <class RealType>
 865: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_pdf_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>& tag) {
 866:     BOOST_MATH_STD_USING // for ADL of std functions
 867: 
 868:     return saspoint5_pdf_plus_imp_prec<RealType>(abs(x), tag);
 869: }
 870: 
 871: template <class RealType, class Policy>
 872: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_pdf_imp(const saspoint5_distribution<RealType, Policy>& dist, const RealType& x) {
 873:     //
 874:     // This calculates the pdf of the Saspoint5 distribution and/or its complement.
 875:     //
 876: 
 877:     BOOST_MATH_STD_USING // for ADL of std functions
 878:     constexpr auto function = "boost::math::pdf(saspoint5<%1%>&, %1%)";
 879:     RealType result = 0;
 880:     RealType location = dist.location();
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as saspoint5_pdf_imp_prec, saspoint5_pdf_imp, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 saspoint5_pdf_imp_prec, saspoint5_pdf_imp, ...。

### Lines 881-920 / 第 881-920 行
~~~cpp
 881:     RealType scale = dist.scale();
 882: 
 883:     if (false == detail::check_location(function, location, &result, Policy()))
 884:     {
 885:         return result;
 886:     }
 887:     if (false == detail::check_scale(function, scale, &result, Policy()))
 888:     {
 889:         return result;
 890:     }
 891:     if (false == detail::check_x(function, x, &result, Policy()))
 892:     {
 893:         return result;
 894:     }
 895: 
 896:     typedef typename tools::promote_args<RealType>::type result_type;
 897:     typedef typename policies::precision<result_type, Policy>::type precision_type;
 898:     typedef boost::math::integral_constant<int,
 899:         precision_type::value <= 0 ? 0 :
 900:         precision_type::value <= 53 ? 53 :
 901:         precision_type::value <= 113 ? 113 : 0
 902:     > tag_type;
 903: 
 904:     static_assert(tag_type::value, "The SaS point5 distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
 905: 
 906:     RealType u = (x - location) / scale;
 907: 
 908:     result = saspoint5_pdf_imp_prec(u, tag_type()) / scale;
 909: 
 910:     return result;
 911: }
 912: 
 913: template <class RealType>
 914: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_cdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53>&)
 915: {
 916:     BOOST_MATH_STD_USING
 917:     RealType result;
 918: 
 919:     if (x < 0.5) {
 920:         // Rational Approximation
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, static_assert.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, static_assert。

### Lines 921-960 / 第 921-960 行
~~~cpp
 921:         // Maximum Relative Error: 2.6225e-17
 922:         BOOST_MATH_STATIC const RealType P[16] = {
 923:             static_cast<RealType>(5.0e-1),
 924:             static_cast<RealType>(1.11530082549581486148e2),
 925:             static_cast<RealType>(1.18564167533523512811e4),
 926:             static_cast<RealType>(7.51503793077701705413e5),
 927:             static_cast<RealType>(3.05648233678438482191e7),
 928:             static_cast<RealType>(8.12176734530090957088e8),
 929:             static_cast<RealType>(1.39533182836234507573e10),
 930:             static_cast<RealType>(1.50394359286077974212e11),
 931:             static_cast<RealType>(9.79057903542935575811e11),
 932:             static_cast<RealType>(3.73800992855150140014e12),
 933:             static_cast<RealType>(8.12697090329432868343e12),
 934:             static_cast<RealType>(9.63154058643818290870e12),
 935:             static_cast<RealType>(5.77714904017642642181e12),
 936:             static_cast<RealType>(1.53321958252091815685e12),
 937:             static_cast<RealType>(1.36220966258718212359e11),
 938:             static_cast<RealType>(1.70766655065405022702e9),
 939:         };
 940:         BOOST_MATH_STATIC const RealType Q[16] = {
 941:             static_cast<RealType>(1.),
 942:             static_cast<RealType>(2.24333404643898143947e2),
 943:             static_cast<RealType>(2.39984636687021023600e4),
 944:             static_cast<RealType>(1.53353791432086858132e6),
 945:             static_cast<RealType>(6.30764952479861776476e7),
 946:             static_cast<RealType>(1.70405769169309597488e9),
 947:             static_cast<RealType>(3.00381227010195289341e10),
 948:             static_cast<RealType>(3.37519046677507392667e11),
 949:             static_cast<RealType>(2.35001610518109063314e12),
 950:             static_cast<RealType>(9.90961948200767679416e12),
 951:             static_cast<RealType>(2.47066673978544828258e13),
 952:             static_cast<RealType>(3.51442593932882610556e13),
 953:             static_cast<RealType>(2.68891431106117733130e13),
 954:             static_cast<RealType>(9.99723484253582494535e12),
 955:             static_cast<RealType>(1.49190229409236772612e12),
 956:             static_cast<RealType>(5.68752980146893975323e10),
 957:         };
 958: 
 959:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
 960:     }
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 961-1000 / 第 961-1000 行
~~~cpp
 961:     else if (x < 1) {
 962:         RealType t = x - static_cast <RealType>(0.5);
 963: 
 964:         // Rational Approximation
 965:         // Maximum Relative Error: 9.2135e-19
 966:         BOOST_MATH_STATIC const RealType P[8] = {
 967:             static_cast<RealType>(3.31309550000758082456e-1),
 968:             static_cast<RealType>(1.63012162307622129396e0),
 969:             static_cast<RealType>(2.97763161467248770571e0),
 970:             static_cast<RealType>(2.49277948739575294031e0),
 971:             static_cast<RealType>(9.49619262302649586821e-1),
 972:             static_cast<RealType>(1.38360148984087584165e-1),
 973:             static_cast<RealType>(4.00812864075652334798e-3),
 974:             static_cast<RealType>(-4.82051978765960490940e-5),
 975:         };
 976:         BOOST_MATH_STATIC const RealType Q[7] = {
 977:             static_cast<RealType>(1.),
 978:             static_cast<RealType>(5.43565383128046471592e0),
 979:             static_cast<RealType>(1.13265160672130133152e1),
 980:             static_cast<RealType>(1.13352316246726435292e1),
 981:             static_cast<RealType>(5.56671465170409694873e0),
 982:             static_cast<RealType>(1.21011708389501479550e0),
 983:             static_cast<RealType>(8.34618282872428849500e-2),
 984:         };
 985: 
 986:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 987:     }
 988:     else if (x < 2) {
 989:         RealType t = x - 1;
 990: 
 991:         // Rational Approximation
 992:         // Maximum Relative Error: 6.4688e-17
 993:         BOOST_MATH_STATIC const RealType P[7] = {
 994:             static_cast<RealType>(2.71280312689343248819e-1),
 995:             static_cast<RealType>(7.44610837974139249205e-1),
 996:             static_cast<RealType>(7.17844128359406982825e-1),
 997:             static_cast<RealType>(2.98789060945288850507e-1),
 998:             static_cast<RealType>(5.22747411439102272576e-2),
 999:             static_cast<RealType>(3.06447984437786430265e-3),
1000:             static_cast<RealType>(2.60407071021044908690e-5),
~~~
- **EN:** This range declares or defines callable logic such as if, tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 if, tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1001-1040 / 第 1001-1040 行
~~~cpp
1001:         };
1002:         BOOST_MATH_STATIC const RealType Q[7] = {
1003:             static_cast<RealType>(1.),
1004:             static_cast<RealType>(3.06221257507188300824e0),
1005:             static_cast<RealType>(3.44827372231472308047e0),
1006:             static_cast<RealType>(1.78166113338930668519e0),
1007:             static_cast<RealType>(4.25580478492907232687e-1),
1008:             static_cast<RealType>(4.09983847731128510426e-2),
1009:             static_cast<RealType>(1.04343172183467651240e-3),
1010:         };
1011: 
1012:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1013:     }
1014:     else if (x < 4) {
1015:         RealType t = x - 2;
1016: 
1017:         // Rational Approximation
1018:         // Maximum Relative Error: 8.2289e-18
1019:         BOOST_MATH_STATIC const RealType P[7] = {
1020:             static_cast<RealType>(2.13928162275383716645e-1),
1021:             static_cast<RealType>(2.35139109235828185307e-1),
1022:             static_cast<RealType>(9.35967515134932733243e-2),
1023:             static_cast<RealType>(1.64310489592753858417e-2),
1024:             static_cast<RealType>(1.23186728989215889119e-3),
1025:             static_cast<RealType>(3.13500969261032539402e-5),
1026:             static_cast<RealType>(1.17021346758965979212e-7),
1027:         };
1028:         BOOST_MATH_STATIC const RealType Q[7] = {
1029:             static_cast<RealType>(1.),
1030:             static_cast<RealType>(1.28212183177829510267e0),
1031:             static_cast<RealType>(6.17321009406850420793e-1),
1032:             static_cast<RealType>(1.38400318019319970893e-1),
1033:             static_cast<RealType>(1.44994794535896837497e-2),
1034:             static_cast<RealType>(6.17774446282546623636e-4),
1035:             static_cast<RealType>(7.00521050169239269819e-6),
1036:         };
1037: 
1038:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1039:     }
1040:     else if (x < 8) {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1041-1080 / 第 1041-1080 行
~~~cpp
1041:         RealType t = x - 4;
1042: 
1043:         // Rational Approximation
1044:         // Maximum Relative Error: 3.7284e-17
1045:         BOOST_MATH_STATIC const RealType P[7] = {
1046:             static_cast<RealType>(1.63772802979087193656e-1),
1047:             static_cast<RealType>(9.69009603942214234119e-2),
1048:             static_cast<RealType>(2.08261725719828138744e-2),
1049:             static_cast<RealType>(1.97965182693146960970e-3),
1050:             static_cast<RealType>(8.05499273532204276894e-5),
1051:             static_cast<RealType>(1.11401971145777879684e-6),
1052:             static_cast<RealType>(2.25932082770588727842e-9),
1053:         };
1054:         BOOST_MATH_STATIC const RealType Q[7] = {
1055:             static_cast<RealType>(1.),
1056:             static_cast<RealType>(6.92463563872865541733e-1),
1057:             static_cast<RealType>(1.80720987166755982366e-1),
1058:             static_cast<RealType>(2.20416647324531054557e-2),
1059:             static_cast<RealType>(1.26052070140663063778e-3),
1060:             static_cast<RealType>(2.93967534265875431639e-5),
1061:             static_cast<RealType>(1.82706995042259549615e-7),
1062:         };
1063: 
1064:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1065:     }
1066:     else if (x < 16) {
1067:         RealType t = x - 8;
1068: 
1069:         // Rational Approximation
1070:         // Maximum Relative Error: 4.9609e-17
1071:         BOOST_MATH_STATIC const RealType P[7] = {
1072:             static_cast<RealType>(1.22610122564874280532e-1),
1073:             static_cast<RealType>(3.70273222121572231593e-2),
1074:             static_cast<RealType>(4.06083618461789591121e-3),
1075:             static_cast<RealType>(1.96898134215932126299e-4),
1076:             static_cast<RealType>(4.08421066512186972853e-6),
1077:             static_cast<RealType>(2.87707419853226244584e-8),
1078:             static_cast<RealType>(2.96850126180387702894e-11),
1079:         };
1080:         BOOST_MATH_STATIC const RealType Q[7] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1081-1120 / 第 1081-1120 行
~~~cpp
1081:             static_cast<RealType>(1.),
1082:             static_cast<RealType>(3.55825191301363023576e-1),
1083:             static_cast<RealType>(4.77251766176046719729e-2),
1084:             static_cast<RealType>(2.99136605131226103925e-3),
1085:             static_cast<RealType>(8.78895785432321899939e-5),
1086:             static_cast<RealType>(1.05235770624006494709e-6),
1087:             static_cast<RealType>(3.35423877769913468556e-9),
1088:         };
1089: 
1090:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1091:     }
1092:     else if (x < 32) {
1093:         RealType t = x - 16;
1094: 
1095:         // Rational Approximation
1096:         // Maximum Relative Error: 5.6559e-17
1097:         BOOST_MATH_STATIC const RealType P[7] = {
1098:             static_cast<RealType>(9.03056141356415077080e-2),
1099:             static_cast<RealType>(1.37568904417652631821e-2),
1100:             static_cast<RealType>(7.60947271383247418831e-4),
1101:             static_cast<RealType>(1.86048302967560067128e-5),
1102:             static_cast<RealType>(1.94537860496575427218e-7),
1103:             static_cast<RealType>(6.90524093915996283104e-10),
1104:             static_cast<RealType>(3.58808434477817122371e-13),
1105:         };
1106:         BOOST_MATH_STATIC const RealType Q[7] = {
1107:             static_cast<RealType>(1.),
1108:             static_cast<RealType>(1.80501347735272292079e-1),
1109:             static_cast<RealType>(1.22807958286146936376e-2),
1110:             static_cast<RealType>(3.90421541115275676253e-4),
1111:             static_cast<RealType>(5.81669449234915057779e-6),
1112:             static_cast<RealType>(3.53005415676201803667e-8),
1113:             static_cast<RealType>(5.69883025435873921433e-11),
1114:         };
1115: 
1116:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1117:     }
1118:     else if (x < 64) {
1119:         RealType t = x - 32;
1120: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1121-1160 / 第 1121-1160 行
~~~cpp
1121:         // Rational Approximation
1122:         // Maximum Relative Error: 6.0653e-17
1123:         BOOST_MATH_STATIC const RealType P[7] = {
1124:             static_cast<RealType>(6.57333571766941474226e-2),
1125:             static_cast<RealType>(5.02795551798163084224e-3),
1126:             static_cast<RealType>(1.39633616037997111325e-4),
1127:             static_cast<RealType>(1.71386564634533872559e-6),
1128:             static_cast<RealType>(8.99508156357247137439e-9),
1129:             static_cast<RealType>(1.60229460572297160486e-11),
1130:             static_cast<RealType>(4.17711709622960498456e-15),
1131:         };
1132:         BOOST_MATH_STATIC const RealType Q[7] = {
1133:             static_cast<RealType>(1.),
1134:             static_cast<RealType>(9.10198637347368265508e-2),
1135:             static_cast<RealType>(3.12263472357578263712e-3),
1136:             static_cast<RealType>(5.00524795130325614005e-5),
1137:             static_cast<RealType>(3.75913188747149725195e-7),
1138:             static_cast<RealType>(1.14970132098893394023e-9),
1139:             static_cast<RealType>(9.34957119271300093120e-13),
1140:         };
1141: 
1142:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1143:     }
1144:     else {
1145:         RealType t = 1 / sqrt(x);
1146: 
1147:         // Rational Approximation
1148:         // Maximum Relative Error: 2.0104e-20
1149:         BOOST_MATH_STATIC const RealType P[5] = {
1150:             static_cast<RealType>(3.98942280401432677940e-1),
1151:             static_cast<RealType>(8.12222388783621449146e-2),
1152:             static_cast<RealType>(1.68515703707271703934e-2),
1153:             static_cast<RealType>(2.19801627205374824460e-3),
1154:             static_cast<RealType>(-5.63321705854968264807e-5),
1155:         };
1156:         BOOST_MATH_STATIC const RealType Q[5] = {
1157:             static_cast<RealType>(1.),
1158:             static_cast<RealType>(6.02536240902768558315e-1),
1159:             static_cast<RealType>(1.99284471400121092380e-1),
1160:             static_cast<RealType>(3.48012577961755452113e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1161-1200 / 第 1161-1200 行
~~~cpp
1161:             static_cast<RealType>(3.38545004473058881799e-3),
1162:         };
1163: 
1164:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t;
1165:     }
1166: 
1167:     return result;
1168: }
1169: 
1170: 
1171: template <class RealType>
1172: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_cdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>&)
1173: {
1174:     BOOST_MATH_STD_USING
1175:     RealType result;
1176: 
1177:     if (x < 0.125) {
1178:         // Rational Approximation
1179:         // Maximum Relative Error: 6.9340e-35
1180:         // LCOV_EXCL_START
1181:         BOOST_MATH_STATIC const RealType P[30] = {
1182:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.0e-1),
1183:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.25520067710293108163697513129883130648e2),
1184:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.70866020657515874782126804139443323023e5),
1185:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.00865235319309486225795793030882782077e7),
1186:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.15226363537737769449645357346965170790e10),
1187:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.90371247243851280277289046301838071764e12),
1188:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.55124590509169425751300134399513503679e14),
1189:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.31282020412787511681760982839078664474e16),
1190:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.81134278666896523873256421982740565131e18),
1191:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.36154530125229747305141034242362609073e20),
1192:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.67793867640429875837167908549938345465e22),
1193:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34584264816825205490037614178084070903e24),
1194:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.52622279567059369718208827282730379468e25),
1195:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.84678324511679577282571711018484545185e27),
1196:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.99412564257799793932936828924325638617e28),
1197:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.08467105431111959283045453636520222779e30),
1198:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.87466808926544728702827204697734995611e31),
1199:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.55020252231174414164534905191762212055e32),
1200:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.69582736077420504345389671165954321163e33),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 1201-1240 / 第 1201-1240 行
~~~cpp
1201:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.18203860972249826626461130638196586188e34),
1202:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.32955733788770318392204091471121129386e35),
1203:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.97972270315674052071792562126668438695e35),
1204:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.93941537398987201071027348577636994465e36),
1205:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.40818708062034138095495206258366082481e36),
1206:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.76833406751769751643745383413977973530e36),
1207:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.67873467711368838525239991688791162617e36),
1208:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.94179310584115437584091984619858795365e36),
1209:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24348215908456320362232906012152922949e36),
1210:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71625432346533320597285660433110657670e35),
1211:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.54662474187354179772157464533408058525e33),
1212:         };
1213:         BOOST_MATH_STATIC const RealType Q[31] = {
1214:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1215:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.05231337496532137901354609636674085703e3),
1216:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.43071888317491317900094470796567113997e5),
1217:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.80864482202910830302921131771345102044e8),
1218:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.32755297215862998181755216820621285536e10),
1219:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.86251123527611073428156549377791985741e12),
1220:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12025543961949466786297141758805461421e15),
1221:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.27681657695574252637426145112570596483e17),
1222:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17850553865715973904162289375819555884e19),
1223:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.87285897504702686250962844939736867339e20),
1224:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.46852796231948446334549476317560711795e22),
1225:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76101689878844725930808096548998198853e24),
1226:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14017776727845251567032313915953239178e26),
1227:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.83738954971390158348334918235614003163e27),
1228:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04701345216121451992682705965658316871e29),
1229:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.29994190638467725374533751141434904865e30),
1230:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.03334024242845994501493644478442360593e31),
1231:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.59094378123268840693978620156028975277e32),
1232:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.05630254163426327113368743426054256780e33),
1233:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.06008195534030444387061989883493342898e34),
1234:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.21262490304347036689874956206774563906e35),
1235:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52353024633841796119920505314785365242e36),
1236:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.28839143293381125956284415313626962263e36),
1237:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31074057704096457802547386358094338369e37),
1238:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.24762412200364040971704861346921094354e37),
1239:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.55663903116458425420509083471048286114e37),
1240:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.81839283802391753865642022579846918253e37),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 1241-1280 / 第 1241-1280 行
~~~cpp
1241:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.40026559327708207943879092058654410696e36),
1242:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.48767474646810049293505781106444169229e36),
1243:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.10591353097667671736865938428051885499e35),
1244:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.26980708896893794012677171239610721832e33),
1245:         };
1246:         // LCOV_EXCL_STOP
1247:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
1248:     }
1249:     else if (x < 0.25) {
1250:         RealType t = x - static_cast <RealType>(0.125);
1251: 
1252:         // Rational Approximation
1253:         // Maximum Relative Error: 9.6106e-35
1254:         // LCOV_EXCL_START
1255:         BOOST_MATH_STATIC const RealType P[14] = {
1256:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.31887921568009055676985827521151969069e-1),
1257:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.62791448964529380666250180886203090183e1),
1258:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.18238045199893937316918299064825702894e2),
1259:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.77274519306540522227493503092956314136e3),
1260:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.89424638466340765479970877448972418958e4),
1261:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.84027004420207996285174223581748706097e5),
1262:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.84633134142285937075423713704784530853e5),
1263:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.76780579189423063605715733542379494552e6),
1264:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19812409802969581112716039533798357401e6),
1265:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.60039008588877024309600768114757310858e6),
1266:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.10268260529501421009222937882726290612e5),
1267:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.72169594688819848498039471657587836720e4),
1268:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.27181379647139697258984772894869505788e3),
1269:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.73617450590346508706222885401965820190e1),
1270:         };
1271:         BOOST_MATH_STATIC const RealType Q[13] = {
1272:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1273:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.18558935411552146390814444666395959919e1),
1274:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49210559503096368944407109881023223654e3),
1275:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.93959323596111340518285858313038058302e4),
1276:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.53590607436758691037825792660167970938e5),
1277:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.82700985983018132572589829602100319330e5),
1278:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.62137033935442506086127262036686905276e6),
1279:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.76014299715348555304267927238963139228e6),
1280:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.12336796972134088340556958396544477713e6),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1281-1320 / 第 1281-1320 行
~~~cpp
1281:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.01952132024838508233050167059872220508e6),
1282:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.41846547214877387780832317250797043384e6),
1283:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.02083431572388097955901208994308271581e5),
1284:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.30401057171447074343957754855656724141e4),
1285:         };
1286:         // LCOV_EXCL_STOP
1287:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1288:     }
1289:     else if (x < 0.5) {
1290:         RealType t = x - static_cast <RealType>(0.25);
1291: 
1292:         // Rational Approximation
1293:         // Maximum Relative Error: 3.1519e-35
1294:         // LCOV_EXCL_START
1295:         BOOST_MATH_STATIC const RealType P[14] = {
1296:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.87119665000174806422420129219814467874e-1),
1297:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.60769554551148293079169764245570645155e0),
1298:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.49181979810834706538329284478129952168e1),
1299:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.15722765491675871778645250624425739489e2),
1300:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.65973147084701923411221710174830072860e2),
1301:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.93709338011482232037110656459951914303e3),
1302:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.57393131299425403017769538642434714791e3),
1303:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.24110491141294379107651487490031694257e3),
1304:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23670394514211681515965192338544032862e3),
1305:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.06141024932329394052395469123628405389e2),
1306:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.08599362073145455095790192415468286304e1),
1307:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.22746783794652085925801188098270888502e0),
1308:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.19652234873414609727168969049557770989e-2),
1309:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.73529976407853894192156335785920329181e-4),
1310:         };
1311:         BOOST_MATH_STATIC const RealType Q[13] = {
1312:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1313:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.04157569499592889296640733909653747983e1),
1314:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.82248883130787159161541119440215325308e2),
1315:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.35191216924911901198168794737654512677e2),
1316:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.05099314677808235578577204150229855903e3),
1317:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.61081069236463123032873733048661305746e3),
1318:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.65340645555368229718826047069323437201e3),
1319:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.44526681322128674428653420882660351679e3),
1320:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.03963804195353853550682049993122898950e3),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1321-1360 / 第 1321-1360 行
~~~cpp
1321:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.40399236835577953127465726826981753422e3),
1322:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.46764755170079991793106428011388637748e2),
1323:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.06384106042490712972156545051459068443e1),
1324:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.27614406724572981099586665536543423891e0),
1325:         };
1326:         // LCOV_EXCL_STOP
1327:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1328:     }
1329:     else if (x < 1) {
1330:         RealType t = x - static_cast <RealType>(0.5);
1331: 
1332:         // Rational Approximation
1333:         // Maximum Relative Error: 7.1196e-37
1334:         // LCOV_EXCL_START
1335:         BOOST_MATH_STATIC const RealType P[15] = {
1336:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.31309550000758082761278726632760756847e-1),
1337:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.07242222531117199094690544171275415854e0),
1338:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.18286763141875580859241637334381199648e1),
1339:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.72102024869298528501604761974348686708e1),
1340:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31748399999514540052066169132819656757e2),
1341:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.72168003284748405703923567644025252608e2),
1342:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52648991506052496046447777354251378257e2),
1343:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.16777263528764704804758173026143295383e1),
1344:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.66044453196259367950849328889468385159e1),
1345:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.35095952392355288307377427145581700484e0),
1346:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43011308494452327007589069222668324337e0),
1347:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16582092138863383294685790744721021189e-1),
1348:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.02261914949200575965813000131964695720e-3),
1349:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.93943913630044161720796150617166047233e-5),
1350:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.76395009419307902351328300308365369814e-8),
1351:         };
1352:         BOOST_MATH_STATIC const RealType Q[14] = {
1353:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1354:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.28073115520716780203055949058270715651e1),
1355:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.20245752585870752942356137496087189194e1),
1356:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.34548337034735803039553186623067144497e2),
1357:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.90925817267776213429724248532378895039e2),
1358:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.92883822651628140083115301005227577059e2),
1359:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.72868136219107985834601503784789993218e2),
1360:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.50498744791568911029110559017896701095e2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1361-1400 / 第 1361-1400 行
~~~cpp
1361:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.05178276667813671578581259848923964311e2),
1362:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.16880263792490095344135867620645018480e1),
1363:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16199396397514668672304602774610890666e1),
1364:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.25543193822942088303609988399416145281e0),
1365:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.62522294286034117189844614005500278984e-2),
1366:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18342889744790118595835138444372660676e-3),
1367:         };
1368:         // LCOV_EXCL_STOP
1369:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1370:     }
1371:     else if (x < 2) {
1372:         RealType t = x - 1;
1373: 
1374:         // Rational Approximation
1375:         // Maximum Relative Error: 9.5605e-36
1376:         // LCOV_EXCL_START
1377:         BOOST_MATH_STATIC const RealType P[14] = {
1378:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.71280312689343266367958859259591541365e-1),
1379:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49628636612698702680819948707479820292e0),
1380:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.61090930375686902075245639803646265081e0),
1381:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.01191924051756106307211298794294657688e0),
1382:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.42496510376427957390465373165464672088e0),
1383:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.59577769624139820954046058289100998534e0),
1384:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02664809521258420718170586857797408674e0),
1385:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.72258711278476951299824066502536249701e-1),
1386:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.72578941800687566921553416498339481887e-2),
1387:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.12553368488232553360765667155702324159e-3),
1388:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.20749770911901442251726681861858323649e-4),
1389:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00621121212654384864006297569770703900e-5),
1390:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18976033102817074104109472578202752346e-7),
1391:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.22093548539863254922531707899658394458e-10),
1392:         };
1393:         BOOST_MATH_STATIC const RealType Q[14] = {
1394:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1395:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.83305694892673455436552817409325835774e0),
1396:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49922543669955056754932640312490112609e1),
1397:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.23488972536322019584648241457582608908e1),
1398:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.14051527527038669918848981363974859889e1),
1399:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.37891280136777182304388426277537358346e1),
1400:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.08058775103864815769223385606687612117e0),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1401-1440 / 第 1401-1440 行
~~~cpp
1401:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83305488980337433132332401784292281716e0),
1402:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.71072208215804671719811563659227630554e-1),
1403:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.86332040813989094594982937011005305263e-2),
1404:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.87698178237970337664105782546771501188e-3),
1405:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.69113555019737313680732855691540088318e-4),
1406:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.31888539972217875242352157306613891243e-6),
1407:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.85633766164682554126992822326956560433e-8),
1408:         };
1409:         // LCOV_EXCL_STOP
1410:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1411:     }
1412:     else if (x < 4) {
1413:         RealType t = x - 2;
1414: 
1415:         // Rational Approximation
1416:         // Maximum Relative Error: 1.1494e-35
1417:         // LCOV_EXCL_START
1418:         BOOST_MATH_STATIC const RealType P[14] = {
1419:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.13928162275383718405630406427822960090e-1),
1420:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.90742307267701162395764574873947997211e-1),
1421:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.13821826367941514387521090205756466068e-1),
1422:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.96186879146063565484800486550739025293e-1),
1423:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19438785955706463753454881511977831603e-1),
1424:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.44969124820016994689518539612465708536e-2),
1425:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.27841835070651018079759230944461773079e-2),
1426:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.69952429132675045239242077293594666305e-3),
1427:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47919853099168659881487026035933933068e-4),
1428:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.04644774117864306055402364094681541437e-6),
1429:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.52604718870921084048756263996119841957e-7),
1430:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.97610950633031564892821158058978809537e-9),
1431:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.35934159016861180185992558083703785765e-11),
1432:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.21044098237798939057079316997065892072e-14),
1433:         };
1434:         BOOST_MATH_STATIC const RealType Q[14] = {
1435:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1436:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.94437702178976797218081686254875998984e0),
1437:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.82068837586514484653828718675654460991e0),
1438:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.87606058269189306593797764456467061128e0),
1439:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.39130528408903116343256483948950693356e0),
1440:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.52792074489091396425713962375223436022e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1441-1480 / 第 1441-1480 行
~~~cpp
1441:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00892233011840867583848470677898363716e-1),
1442:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.53717105060592851173320646706141911461e-2),
1443:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.57293857675930200001382624769341451561e-3),
1444:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04219251796696135508847408131139677925e-4),
1445:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.20053006131133304932740325113068767057e-6),
1446:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.26384707028090985155079342718673255493e-8),
1447:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.19146442700994823924806249608315505708e-10),
1448:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.59879298772002950043508762057850408213e-12),
1449:         };
1450:         // LCOV_EXCL_STOP
1451:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1452:     }
1453:     else if (x < 8) {
1454:         RealType t = x - 4;
1455: 
1456:         // Rational Approximation
1457:         // Maximum Relative Error: 1.9710e-35
1458:         // LCOV_EXCL_START
1459:         BOOST_MATH_STATIC const RealType P[14] = {
1460:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.63772802979087199762340235165979751298e-1),
1461:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.31941534705372320785274994658709390116e-1),
1462:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43643960022585762678456016437621064500e-1),
1463:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.11415302325466272779041471612529728187e-2),
1464:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15767742459744253874067896740220951622e-2),
1465:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.74049309186016489825053763513176160256e-3),
1466:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.76349898574685150849080543168157785281e-4),
1467:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19757202370729036627932327405149840205e-5),
1468:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.32067727965321839898287320520750897894e-7),
1469:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47636340015260789807543414080472136575e-8),
1470:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.36236727340568181129875213546468908164e-10),
1471:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.89379573960280486883733996547662506245e-12),
1472:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.71832232038263988173042637335112603365e-15),
1473:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.72380245500539326441037770757072641975e-18),
1474:         };
1475:         BOOST_MATH_STATIC const RealType Q[14] = {
1476:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1477:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.51702400281458104713682413542736419584e0),
1478:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.01375164846907815766683647295932603968e0),
1479:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.92796007869834847612192314006582598557e-1),
1480:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.77580441164023725582659445614058463183e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1481-1520 / 第 1481-1520 行
~~~cpp
1481:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.63592331843149724480258804892989851727e-2),
1482:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.87334158717610115008450674967492650941e-3),
1483:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.46596056941432875244263245821845070102e-4),
1484:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.69980051560936361597177347949112822752e-6),
1485:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.61690034211585843423761830218320365457e-7),
1486:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.40619773800285766355596852314940341504e-9),
1487:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.10624533319804091814643828283820958419e-11),
1488:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.10009654621246392691126133176423833259e-13),
1489:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.48070591106986983088640496621926852293e-16),
1490:         };
1491:         // LCOV_EXCL_STOP
1492:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1493:     }
1494:     else if (x < 16) {
1495:         RealType t = x - 8;
1496: 
1497:         // Rational Approximation
1498:         // Maximum Relative Error: 5.2049e-36
1499:         // LCOV_EXCL_START
1500:         BOOST_MATH_STATIC const RealType P[14] = {
1501:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.22610122564874286614786819620499101143e-1),
1502:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.30352481858382230273216195795534959290e-2),
1503:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.45933050053542949214164590814846222512e-2),
1504:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.18776888646200567321599584635465632591e-3),
1505:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.53421996228923143480455729204878676265e-4),
1506:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.26075686557831306993734433164305349875e-5),
1507:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.58045762501721375879877727645933749122e-6),
1508:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.13469629033419341069106781092024950086e-8),
1509:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09157226556088521407323375433512662525e-9),
1510:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44961350323527660188267669752380722085e-11),
1511:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11052101325523147964890915835024505324e-13),
1512:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.26421404354976214191891992583151033361e-16),
1513:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.17133505681224996657291059553060754343e-19),
1514:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41010917905686427164414364663355769988e-22),
1515:         };
1516:         BOOST_MATH_STATIC const RealType Q[14] = {
1517:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1518:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.31062773739451672808456319166347015167e-1),
1519:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.35386721434011881226168110614121649232e-1),
1520:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.39357338312443465616015226804775178232e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1521-1560 / 第 1521-1560 行
~~~cpp
1521:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.26630144036271792027494677957363535353e-3),
1522:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.24340476859846183414651435036807677467e-4),
1523:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.33917136421389571662908749253850939876e-5),
1524:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.80972141456523767244381195690041498939e-7),
1525:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.22653625120465488656616983786525028119e-8),
1526:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.64072452032620505897896978124863889812e-10),
1527:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.61842001579321492488462230987972104386e-12),
1528:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.96631619425501661980194304605724632777e-14),
1529:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.80392324086028812772385536034034039168e-17),
1530:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.34254502871215949266781048808984963366e-20),
1531:         };
1532:         // LCOV_EXCL_STOP
1533:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1534:     }
1535:     else if (x < 32) {
1536:         RealType t = x - 16;
1537: 
1538:         // Rational Approximation
1539:         // Maximum Relative Error: 1.7434e-35
1540:         // LCOV_EXCL_START
1541:         BOOST_MATH_STATIC const RealType P[14] = {
1542:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.03056141356415128156562790092782153630e-2),
1543:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.15331583242023443256381237551843296356e-2),
1544:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.81847913073640285776566199343276995613e-3),
1545:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.23578443960486030170636772457627141406e-4),
1546:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.36906354316016270165240908809929957836e-5),
1547:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.80584421020238085239890207672296651219e-7),
1548:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.20726437845755296397071540583729544203e-8),
1549:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.71042563703818585243207722641746283288e-10),
1550:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.08307373360265947158569900625482137206e-12),
1551:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.80776566500233755365518221977875432763e-14),
1552:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11405351639704510305055492207286172753e-16),
1553:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.21575609293568296049921888011966327905e-19),
1554:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.66081982641748223969990279975752576675e-22),
1555:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.96483118060215455299182487430511998831e-26),
1556:         };
1557:         BOOST_MATH_STATIC const RealType Q[14] = {
1558:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1559:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.77347004038951368607085827825968614455e-1),
1560:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.27559305780716801070924630708599448466e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1561-1600 / 第 1561-1600 行
~~~cpp
1561:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.05452382624230160738008550961679711827e-3),
1562:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.75369667590360521677018734348769796476e-4),
1563:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.56551290985905942229892419848093494661e-5),
1564:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.46972102361871185271727958608184616388e-7),
1565:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.72423917010499649257775199140781647069e-9),
1566:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14337306905269302583746182007852069459e-10),
1567:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.69949885309711859563395555285232232606e-13),
1568:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.00318719634300754237920041312234711548e-15),
1569:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41139664927184402637020651515172315287e-17),
1570:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.79013190225240505774959477465594797961e-20),
1571:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.46536966503325413797061462062918707370e-24),
1572:         };
1573:         // LCOV_EXCL_STOP
1574:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1575:     }
1576:     else if (x < 64) {
1577:         RealType t = x - 32;
1578: 
1579:         // Rational Approximation
1580:         // Maximum Relative Error: 2.0402e-35
1581:         // LCOV_EXCL_START
1582:         BOOST_MATH_STATIC const RealType P[14] = {
1583:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.57333571766941514095434647381791040479e-2),
1584:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15416685251021339933358981066948923001e-2),
1585:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.86806753164417557035166075399588122481e-4),
1586:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.91972616817770660098405128729991574724e-5),
1587:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.10225768760715861978198010761036882002e-6),
1588:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.05986998674039047865566990469266534338e-8),
1589:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.59572646670205456333051888086612875871e-10),
1590:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19347294198055585461131949159508730257e-12),
1591:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21328285448498841418774425071549974153e-14),
1592:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.19331596847283822557042655221763459728e-17),
1593:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.36128017817576942059191451016251062072e-20),
1594:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.35600223942735523925477855247725326228e-23),
1595:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.14658948592500290756690769268766876322e-26),
1596:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.86984591055448991335081550609451649866e-30),
1597:         };
1598:         BOOST_MATH_STATIC const RealType Q[14] = {
1599:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1600:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.90112824856612652807095815199496602262e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1601-1640 / 第 1601-1640 行
~~~cpp
1601:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59291524937386142936420775839969648652e-2),
1602:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.74245361925275011235694006013677228467e-4),
1603:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.41828589449615478387532599798645159282e-5),
1604:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.08088176420557205743676774127863572768e-7),
1605:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.30760429417424419297000535744450830697e-9),
1606:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.18464910867914234357511605329900284981e-11),
1607:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.74255540513281299503596269087176674333e-13),
1608:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.02616028440371294233330747672966435921e-15),
1609:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.26276597941744408946918920573146445795e-18),
1610:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.47463109867603732992337779860914933775e-21),
1611:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.77217411888267832243050973915295217582e-24),
1612:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.67397425207383164084527830512920206074e-28),
1613:         };
1614:         // LCOV_EXCL_STOP
1615:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1616:     }
1617:     else {
1618:         RealType t = 1 / sqrt(x);
1619: 
1620:         // Rational Approximation
1621:         // Maximum Relative Error: 9.2612e-35
1622:         // LCOV_EXCL_START
1623:         BOOST_MATH_STATIC const RealType P[9] = {
1624:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.98942280401432677939946059934381868476e-1),
1625:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33908701314796522684603310107061150444e-1),
1626:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.92120397142832495974006972404741124398e-2),
1627:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.15463147603421962834297353867930971657e-3),
1628:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.44488751006069172847577645328482300099e-4),
1629:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44057582804743599116332797864164802887e-5),
1630:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.02968018188491417839349438941039867033e-6),
1631:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.75092244933846337077999183310087492887e-7),
1632:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.35099582728548602389917143511323566818e-8),
1633:         };
1634:         BOOST_MATH_STATIC const RealType Q[7] = {
1635:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1636:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.34601617336219074065534356705298927390e-1),
1637:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.82954035780824611941899463895040327299e-1),
1638:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.70929001162671283123255408612494541378e-2),
1639:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.05508596604210030533747793197422815105e-2),
1640:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02913299057943756875992272236063124608e-3),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1641-1680 / 第 1641-1680 行
~~~cpp
1641:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.37824426836648736125759177846682556245e-5),
1642:         };
1643:         // LCOV_EXCL_STOP
1644:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t;
1645:     }
1646: 
1647:     return result;
1648: }
1649: 
1650: template <class RealType>
1651: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_cdf_imp_prec(const RealType& x, bool complement, const boost::math::integral_constant<int, 53>& tag) {
1652:     if (x >= 0) {
1653:         return complement ? saspoint5_cdf_plus_imp_prec(x, tag) : 1 - saspoint5_cdf_plus_imp_prec(x, tag);
1654:     }
1655:     else if (x <= 0) {
1656:         return complement ? 1 - saspoint5_cdf_plus_imp_prec(-x, tag) : saspoint5_cdf_plus_imp_prec(-x, tag);
1657:     }
1658:     else {
1659:         return boost::math::numeric_limits<RealType>::quiet_NaN();
1660:     }
1661: }
1662: 
1663: template <class RealType>
1664: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_cdf_imp_prec(const RealType& x, bool complement, const boost::math::integral_constant<int, 113>& tag) {
1665:     if (x >= 0) {
1666:         return complement ? saspoint5_cdf_plus_imp_prec(x, tag) : 1 - saspoint5_cdf_plus_imp_prec(x, tag);
1667:     }
1668:     else if (x <= 0) {
1669:         return complement ? 1 - saspoint5_cdf_plus_imp_prec(-x, tag) : saspoint5_cdf_plus_imp_prec(-x, tag);
1670:     }
1671:     else {
1672:         return boost::math::numeric_limits<RealType>::quiet_NaN();
1673:     }
1674: }
1675: 
1676: template <class RealType, class Policy>
1677: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_cdf_imp(const saspoint5_distribution<RealType, Policy>& dist, const RealType& x, bool complement) {
1678:     //
1679:     // This calculates the cdf of the Saspoint5 distribution and/or its complement.
1680:     //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as saspoint5_cdf_imp_prec, if, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 saspoint5_cdf_imp_prec, if, ...。

### Lines 1681-1720 / 第 1681-1720 行
~~~cpp
1681: 
1682:     BOOST_MATH_STD_USING // for ADL of std functions
1683:     constexpr auto function = "boost::math::cdf(saspoint5<%1%>&, %1%)";
1684:     RealType result = 0;
1685:     RealType location = dist.location();
1686:     RealType scale = dist.scale();
1687: 
1688:     if (false == detail::check_location(function, location, &result, Policy()))
1689:     {
1690:         return result;
1691:     }
1692:     if (false == detail::check_scale(function, scale, &result, Policy()))
1693:     {
1694:         return result;
1695:     }
1696:     if (false == detail::check_x(function, x, &result, Policy()))
1697:     {
1698:         return result;
1699:     }
1700: 
1701:     typedef typename tools::promote_args<RealType>::type result_type;
1702:     typedef typename policies::precision<result_type, Policy>::type precision_type;
1703:     typedef boost::math::integral_constant<int,
1704:         precision_type::value <= 0 ? 0 :
1705:         precision_type::value <= 53 ? 53 :
1706:         precision_type::value <= 113 ? 113 : 0
1707:     > tag_type;
1708: 
1709:     static_assert(tag_type::value, "The SaS point5 distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
1710: 
1711:     RealType u = (x - location) / scale;
1712: 
1713:     result = saspoint5_cdf_imp_prec(u, complement, tag_type());
1714: 
1715:     return result;
1716: }
1717: 
1718: template <class RealType>
1719: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_quantile_upper_imp_prec(const RealType& p, const boost::math::integral_constant<int, 53>&)
1720: {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as location, scale, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 location, scale, ...。

### Lines 1721-1760 / 第 1721-1760 行
~~~cpp
1721:     BOOST_MATH_STD_USING
1722:     RealType result;
1723: 
1724:     if (ilogb(p) >= -2) {
1725:         RealType u = -log2(ldexp(p, 1));
1726: 
1727:         if (u < 0.125) {
1728:             // Rational Approximation
1729:             // Maximum Relative Error: 4.2616e-17
1730:             BOOST_MATH_STATIC const RealType P[13] = {
1731:                 static_cast<RealType>(1.36099130643975127045e-1),
1732:                 static_cast<RealType>(2.19634434498311523885e1),
1733:                 static_cast<RealType>(1.70276954848343179287e3),
1734:                 static_cast<RealType>(8.02187341786354339306e4),
1735:                 static_cast<RealType>(2.48750112198456813443e6),
1736:                 static_cast<RealType>(5.20617858300443231437e7),
1737:                 static_cast<RealType>(7.31202030685167303439e8),
1738:                 static_cast<RealType>(6.66061403138355591915e9),
1739:                 static_cast<RealType>(3.65687892725590813998e10),
1740:                 static_cast<RealType>(1.06061776220305595494e11),
1741:                 static_cast<RealType>(1.23930642673461465346e11),
1742:                 static_cast<RealType>(1.49986408149520127078e10),
1743:                 static_cast<RealType>(-6.17325587219357123900e8),
1744:             };
1745:             BOOST_MATH_STATIC const RealType Q[13] = {
1746:                 static_cast<RealType>(1.),
1747:                 static_cast<RealType>(1.63111146753825227716e2),
1748:                 static_cast<RealType>(1.27864461509685444043e4),
1749:                 static_cast<RealType>(6.10371533241799228037e5),
1750:                 static_cast<RealType>(1.92422115963507708309e7),
1751:                 static_cast<RealType>(4.11544185502250709497e8),
1752:                 static_cast<RealType>(5.95343302992055062258e9),
1753:                 static_cast<RealType>(5.65615858889758369947e10),
1754:                 static_cast<RealType>(3.30833154992293143503e11),
1755:                 static_cast<RealType>(1.06032392136054207216e12),
1756:                 static_cast<RealType>(1.50071282012095447931e12),
1757:                 static_cast<RealType>(5.43552396263989180433e11),
1758:                 static_cast<RealType>(9.57434915768660935004e10),
1759:             };
1760: 
~~~
- **EN:** This range declares or defines callable logic such as log2. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 log2。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1761-1800 / 第 1761-1800 行
~~~cpp
1761:             result = u * tools::evaluate_polynomial(P, u) / (tools::evaluate_polynomial(Q, u) * (p * p));
1762:         }
1763:         else if (u < 0.25) {
1764:             RealType t = u - static_cast <RealType>(0.125);
1765: 
1766:             // Rational Approximation
1767:             // Maximum Relative Error: 2.3770e-19
1768:             BOOST_MATH_STATIC const RealType P[8] = {
1769:                 static_cast<RealType>(1.46698650748920243698e-2),
1770:                 static_cast<RealType>(3.58380131788385557227e-1),
1771:                 static_cast<RealType>(3.39153750029553194566e0),
1772:                 static_cast<RealType>(1.55457424873957272207e1),
1773:                 static_cast<RealType>(3.44403897039657057261e1),
1774:                 static_cast<RealType>(3.01881531964962975320e1),
1775:                 static_cast<RealType>(2.77679052294606319767e0),
1776:                 static_cast<RealType>(-7.76665288232972435969e-2),
1777:             };
1778:             BOOST_MATH_STATIC const RealType Q[7] = {
1779:                 static_cast<RealType>(1.),
1780:                 static_cast<RealType>(1.72584280323876188464e1),
1781:                 static_cast<RealType>(1.11983518800147654866e2),
1782:                 static_cast<RealType>(3.25969893054048132145e2),
1783:                 static_cast<RealType>(3.91978809680672051666e2),
1784:                 static_cast<RealType>(1.29874252720714897530e2),
1785:                 static_cast<RealType>(2.08740114519610102248e1),
1786:             };
1787: 
1788:             result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1789:         }
1790:         else if (u < 0.5) {
1791:             RealType t = u - static_cast <RealType>(0.25);
1792: 
1793:             // Rational Approximation
1794:             // Maximum Relative Error: 9.2445e-18
1795:             BOOST_MATH_STATIC const RealType P[8] = {
1796:                 static_cast<RealType>(2.69627866689346445458e-2),
1797:                 static_cast<RealType>(3.23091180507445216811e-1),
1798:                 static_cast<RealType>(1.42164019533549860681e0),
1799:                 static_cast<RealType>(2.74613170828120023406e0),
1800:                 static_cast<RealType>(2.07865023346180997996e0),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1801-1840 / 第 1801-1840 行
~~~cpp
1801:                 static_cast<RealType>(2.53267176863740856907e-1),
1802:                 static_cast<RealType>(-2.55816250186301841152e-2),
1803:                 static_cast<RealType>(3.02683750470398342224e-3),
1804:             };
1805:             BOOST_MATH_STATIC const RealType Q[6] = {
1806:                 static_cast<RealType>(1.),
1807:                 static_cast<RealType>(8.55049920135376003042e0),
1808:                 static_cast<RealType>(2.48726119139047911316e1),
1809:                 static_cast<RealType>(2.79519589592198994574e1),
1810:                 static_cast<RealType>(9.88212916161823866098e0),
1811:                 static_cast<RealType>(1.39749417956251951564e0),
1812:             };
1813: 
1814:             result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1815:         }
1816:         else {
1817:             RealType t = u - static_cast <RealType>(0.5);
1818: 
1819:             // Rational Approximation
1820:             // Maximum Relative Error: 2.2918e-20
1821:             BOOST_MATH_STATIC const RealType P[9] = {
1822:                 static_cast<RealType>(4.79518653373241051274e-2),
1823:                 static_cast<RealType>(3.81837125793765918564e-1),
1824:                 static_cast<RealType>(1.13370353708146321188e0),
1825:                 static_cast<RealType>(1.55218145762186846509e0),
1826:                 static_cast<RealType>(9.60938271141036509605e-1),
1827:                 static_cast<RealType>(2.11811755464425606950e-1),
1828:                 static_cast<RealType>(8.84533960603915742831e-3),
1829:                 static_cast<RealType>(1.73314614571009160225e-3),
1830:                 static_cast<RealType>(-3.63491208733876986098e-5),
1831:             };
1832:             BOOST_MATH_STATIC const RealType Q[8] = {
1833:                 static_cast<RealType>(1.),
1834:                 static_cast<RealType>(6.36954463000253710936e0),
1835:                 static_cast<RealType>(1.40601897306833147611e1),
1836:                 static_cast<RealType>(1.33838075106916667084e1),
1837:                 static_cast<RealType>(5.60958095533108032859e0),
1838:                 static_cast<RealType>(1.11796035623375210182e0),
1839:                 static_cast<RealType>(1.12508482637488861060e-1),
1840:                 static_cast<RealType>(5.18503975949799718538e-3),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 1841-1880 / 第 1841-1880 行
~~~cpp
1841:             };
1842: 
1843:             result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1844:         }
1845:     }
1846:     else if (ilogb(p) >= -4) {
1847:         RealType t = -log2(ldexp(p, 2));
1848: 
1849:         // Rational Approximation
1850:         // Maximum Relative Error: 4.2057e-18
1851:         BOOST_MATH_STATIC const RealType P[10] = {
1852:             static_cast<RealType>(8.02395484493329835881e-2),
1853:             static_cast<RealType>(2.46132933068351274622e-1),
1854:             static_cast<RealType>(2.81820176867119231101e-1),
1855:             static_cast<RealType>(1.47754061028371025893e-1),
1856:             static_cast<RealType>(3.54638964490281023406e-2),
1857:             static_cast<RealType>(3.99998730093393774294e-3),
1858:             static_cast<RealType>(3.81581928434827040262e-4),
1859:             static_cast<RealType>(1.82520920154354221101e-5),
1860:             static_cast<RealType>(-2.06151396745690348445e-7),
1861:             static_cast<RealType>(6.77986548138011345849e-9),
1862:         };
1863:         BOOST_MATH_STATIC const RealType Q[8] = {
1864:             static_cast<RealType>(1.),
1865:             static_cast<RealType>(2.39244329037830026691e0),
1866:             static_cast<RealType>(2.12683465416376620896e0),
1867:             static_cast<RealType>(9.02612272334554457823e-1),
1868:             static_cast<RealType>(2.06667959191488815314e-1),
1869:             static_cast<RealType>(2.79328968525257867541e-2),
1870:             static_cast<RealType>(2.28216286216537879937e-3),
1871:             static_cast<RealType>(1.04195690531437767679e-4),
1872:         };
1873: 
1874:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1875:     }
1876:     else if (ilogb(p) >= -8) {
1877:         RealType t = -log2(ldexp(p, 4));
1878: 
1879:         // Rational Approximation
1880:         // Maximum Relative Error: 3.3944e-17
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1881-1920 / 第 1881-1920 行
~~~cpp
1881:         BOOST_MATH_STATIC const RealType P[9] = {
1882:             static_cast<RealType>(1.39293493266195561875e-1),
1883:             static_cast<RealType>(1.26741380938661691592e-1),
1884:             static_cast<RealType>(4.31117040307200265931e-2),
1885:             static_cast<RealType>(7.50528269269498076949e-3),
1886:             static_cast<RealType>(8.63100497178570310436e-4),
1887:             static_cast<RealType>(6.75686286034521991703e-5),
1888:             static_cast<RealType>(3.11102625473120771882e-6),
1889:             static_cast<RealType>(9.63513655399980075083e-8),
1890:             static_cast<RealType>(-6.40223609013005302318e-11),
1891:         };
1892:         BOOST_MATH_STATIC const RealType Q[8] = {
1893:             static_cast<RealType>(1.),
1894:             static_cast<RealType>(8.11234548272888947555e-1),
1895:             static_cast<RealType>(2.63525516991753831892e-1),
1896:             static_cast<RealType>(4.77118226533147280522e-2),
1897:             static_cast<RealType>(5.46090741266888954909e-3),
1898:             static_cast<RealType>(4.15325425646862026425e-4),
1899:             static_cast<RealType>(2.02377681998442384863e-5),
1900:             static_cast<RealType>(5.79823311154876056655e-7),
1901:         };
1902: 
1903:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1904:     }
1905:     else if (ilogb(p) >= -16) {
1906:         RealType t = -log2(ldexp(p, 8));
1907: 
1908:         // Rational Approximation
1909:         // Maximum Relative Error: 4.1544e-17
1910:         BOOST_MATH_STATIC const RealType P[9] = {
1911:             static_cast<RealType>(1.57911660613037760235e-1),
1912:             static_cast<RealType>(5.59740955695099219682e-2),
1913:             static_cast<RealType>(8.92895854008560399142e-3),
1914:             static_cast<RealType>(8.88795299273855801726e-4),
1915:             static_cast<RealType>(5.66358335596607738071e-5),
1916:             static_cast<RealType>(2.46733195253941569922e-6),
1917:             static_cast<RealType>(6.44829870181825872501e-8),
1918:             static_cast<RealType>(7.62193242864380357931e-10),
1919:             static_cast<RealType>(-7.82035413331699873450e-14),
1920:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1921-1960 / 第 1921-1960 行
~~~cpp
1921:         BOOST_MATH_STATIC const RealType Q[8] = {
1922:             static_cast<RealType>(1.),
1923:             static_cast<RealType>(3.49007782566002620811e-1),
1924:             static_cast<RealType>(5.65303702876260444572e-2),
1925:             static_cast<RealType>(5.54316442661801299351e-3),
1926:             static_cast<RealType>(3.58498995501703237922e-4),
1927:             static_cast<RealType>(1.53872913968336341278e-5),
1928:             static_cast<RealType>(4.08512152326482573624e-7),
1929:             static_cast<RealType>(4.72959615756470826429e-9),
1930:         };
1931: 
1932:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1933:     }
1934:     else if (ilogb(p) >= -32) {
1935:         RealType t = -log2(ldexp(p, 16));
1936: 
1937:         // Rational Approximation
1938:         // Maximum Relative Error: 8.5877e-18
1939:         BOOST_MATH_STATIC const RealType P[10] = {
1940:             static_cast<RealType>(1.59150086070234563099e-1),
1941:             static_cast<RealType>(6.07144002506911115092e-2),
1942:             static_cast<RealType>(1.10026443723891740392e-2),
1943:             static_cast<RealType>(1.24892739209332398698e-3),
1944:             static_cast<RealType>(9.82922518655171276487e-5),
1945:             static_cast<RealType>(5.58366837526347222893e-6),
1946:             static_cast<RealType>(2.29005408647580194007e-7),
1947:             static_cast<RealType>(6.44325718317518336404e-9),
1948:             static_cast<RealType>(1.05110361316230054467e-10),
1949:             static_cast<RealType>(1.48083450629432857655e-18),
1950:         };
1951:         BOOST_MATH_STATIC const RealType Q[9] = {
1952:             static_cast<RealType>(1.),
1953:             static_cast<RealType>(3.81470315977341203351e-1),
1954:             static_cast<RealType>(6.91330250512167919573e-2),
1955:             static_cast<RealType>(7.84712209182587717077e-3),
1956:             static_cast<RealType>(6.17595479676821181012e-4),
1957:             static_cast<RealType>(3.50829361179041199953e-5),
1958:             static_cast<RealType>(1.43889153071571504712e-6),
1959:             static_cast<RealType>(4.04840254888235877998e-8),
1960:             static_cast<RealType>(6.60429636407045050112e-10),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1961-2000 / 第 1961-2000 行
~~~cpp
1961:         };
1962: 
1963:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1964:     }
1965:     else if (ilogb(p) >= -64) {
1966:         RealType t = -log2(ldexp(p, 32));
1967: 
1968:         // Rational Approximation
1969:         // Maximum Relative Error: 8.7254e-17
1970:         BOOST_MATH_STATIC const RealType P[9] = {
1971:             static_cast<RealType>(1.59154943017783026201e-1),
1972:             static_cast<RealType>(6.91506515614472069475e-2),
1973:             static_cast<RealType>(1.44590186111155933843e-2),
1974:             static_cast<RealType>(1.92616138327724025421e-3),
1975:             static_cast<RealType>(1.79640147906775699469e-4),
1976:             static_cast<RealType>(1.30852535070639833809e-5),
1977:             static_cast<RealType>(5.55259657884038297268e-7),
1978:             static_cast<RealType>(3.50107118687544980820e-8),
1979:             static_cast<RealType>(-1.47102592933729597720e-22),
1980:         };
1981:         BOOST_MATH_STATIC const RealType Q[8] = {
1982:             static_cast<RealType>(1.),
1983:             static_cast<RealType>(4.34486357752330500669e-1),
1984:             static_cast<RealType>(9.08486933075320995164e-2),
1985:             static_cast<RealType>(1.21024289017243304241e-2),
1986:             static_cast<RealType>(1.12871233794777525784e-3),
1987:             static_cast<RealType>(8.22170725751776749123e-5),
1988:             static_cast<RealType>(3.48879932410650101194e-6),
1989:             static_cast<RealType>(2.19978790407451988423e-7),
1990:         };
1991: 
1992:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
1993:     }
1994:     else {
1995:         result = 1 / (p * p * constants::two_pi<RealType>());
1996:     }
1997: 
1998:     return result;
1999: }
2000: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2001-2040 / 第 2001-2040 行
~~~cpp
2001: 
2002: template <class RealType>
2003: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_quantile_upper_imp_prec(const RealType& p, const boost::math::integral_constant<int, 113>&)
2004: {
2005:     BOOST_MATH_STD_USING
2006:     RealType result;
2007: 
2008:     if (ilogb(p) >= -2) {
2009:         RealType u = -log2(ldexp(p, 1));
2010: 
2011:         if (u < 0.125) {
2012:             // Rational Approximation
2013:             // Maximum Relative Error: 2.5675e-36
2014:             // LCOV_EXCL_START
2015:             BOOST_MATH_STATIC const RealType P[31] = {
2016:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.36099130643975133156293056139850872219e-1),
2017:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.03940482189350763127508703926866548690e1),
2018:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.00518276893354880480781640750482315271e4),
2019:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.55844903094077096941027360107304259099e6),
2020:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04507684135310729583474324660276395831e9),
2021:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.28519957085041757616278379578781441623e11),
2022:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.26054173986187219679917530171252145632e13),
2023:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00693075272502479915569708465960917906e15),
2024:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.64153695410984136395853200311209462775e16),
2025:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.64993034609287363745840801813540992383e18),
2026:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.68080300629977787949474098413155901197e20),
2027:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.50632142671665246974634799849090331338e21),
2028:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.11943753054362349397013211631038480307e23),
2029:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.80601829873419334580289886671478701625e24),
2030:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33441650581633426542372642262736818512e26),
2031:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.56279427934163518272441555879970370340e27),
2032:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.08899113985387092689705022477814364717e28),
2033:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.37750989391907347952902900750138805007e29),
2034:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.76961267256299304213687639380275530721e30),
2035:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.98417586455955659885944915688130612888e31),
2036:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.40932923796679251232655132670811114351e32),
2037:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.80810239916688876216017180714744912573e33),
2038:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.23907429566810200929293428832485038147e33),
2039:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.11441754640405256305951569489818422227e34),
2040:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.30534222360394829628175800718529342304e34),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as log2.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log2。

### Lines 2041-2080 / 第 2041-2080 行
~~~cpp
2041:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.73301799323855143458670230536670073483e34),
2042:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.53142592196246595846485130434777396548e34),
2043:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.81719621726393542967303806360105998384e34),
2044:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.00188544550531824809437206713326495544e33),
2045:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.62706943144847786115732327787879709587e32),
2046:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.32129438774563059735783287456769609571e31),
2047:             };
2048:             BOOST_MATH_STATIC const RealType Q[32] = {
2049:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2050:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.65910866673514847742559406762379054364e2),
2051:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.21954860438789969160116317316418373146e5),
2052:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.85684385746348850219351196129081986508e7),
2053:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.76131116920014625994371306210585646224e9),
2054:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.57402411617965582839975369786525269977e11),
2055:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.42213951996062253608905591667405322835e13),
2056:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.55477693883842522631954327528060778834e15),
2057:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.00397907346473927493255003955380711046e17),
2058:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76305959503723486331556274939198109922e19),
2059:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.27926540483498824808520492399128682366e21),
2060:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.98253913105291675445666919447864520248e22),
2061:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.63457445658532249936389003141915626894e24),
2062:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.51446616633910582673057455450707805902e25),
2063:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04744823698010333311911891992022528040e27),
2064:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.03400927415310540137351756981742318263e28),
2065:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.28761940359662123632247441327784689568e29),
2066:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.39016138777648624292953560568071708327e30),
2067:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.79639567867465767764785448609833337532e31),
2068:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.23406781975678544311073661662680006588e32),
2069:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.97261483656310352862554580475760827374e33),
2070:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62715040832592600542933595577003951697e34),
2071:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.77359945057399130202830211722221279906e34),
2072:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.07842295432910751940058270741081867701e35),
2073:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.51739306780247334064265249344359460675e35),
2074:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.60574331076505049588401700048488577194e35),
2075:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.08286808700840316336961663635580879141e35),
2076:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.27661033115008662284071342245200272702e35),
2077:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00465576791024249023365007797010262700e35),
2078:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83311248273885136105510175099322638440e34),
2079:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.96635220211386288597285960837372073054e33),
2080:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23849744128418288892902205619933047730e32),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 2081-2120 / 第 2081-2120 行
~~~cpp
2081:             };
2082:             // LCOV_EXCL_STOP
2083:             result = u * tools::evaluate_polynomial(P, u) / (tools::evaluate_polynomial(Q, u) * (p * p));
2084:         }
2085:         else if (u < 0.25) {
2086:             RealType t = u - static_cast <RealType>(0.125);
2087: 
2088:             // Rational Approximation
2089:             // Maximum Relative Error: 9.0663e-36
2090:             // LCOV_EXCL_START
2091:             BOOST_MATH_STATIC const RealType P[13] = {
2092:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.46698650748920243663487731226111319705e-2),
2093:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.39021286045890143123252180276484388346e-1),
2094:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21933242816562043224009451007344301143e1),
2095:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33741547463966207206741888477702151242e2),
2096:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.29556944160837955334643715180923663741e2),
2097:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.25261081330476435844217173674285740857e3),
2098:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.28690563577245995896389783271544510833e4),
2099:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.51764495004238264050843085122188741180e4),
2100:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.00501773552098137637598813101153206656e4),
2101:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.93991776883375928647775429233323885440e4),
2102:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.13059418708769178567954713937745050279e3),
2103:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.41565791250614170744069436181282300453e2),
2104:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.35838723365672196069179944509778281549e1),
2105:             };
2106:             BOOST_MATH_STATIC const RealType Q[13] = {
2107:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2108:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.63888803456697300467924455320638435538e1),
2109:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.74785179836182339383932806919167693991e2),
2110:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.15133301804008879476562749311747788645e3),
2111:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.87361675398393057971764841741518474061e4),
2112:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02992617475892211368309739891693879676e5),
2113:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.36356854400440662641546588001882412251e5),
2114:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.35807552915245783626759227539698719908e5),
2115:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.75959389290929178190646034566377062463e5),
2116:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18514088996371641206828142820042918681e5),
2117:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.54881978220293930450469794941944831047e4),
2118:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.83958740186543542804045767758191509433e3),
2119:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.26637084978098507405883170227585648985e2),
2120:             };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2121-2160 / 第 2121-2160 行
~~~cpp
2121:             // LCOV_EXCL_STOP
2122:             result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2123:         }
2124:         else if (u < 0.5) {
2125:             RealType t = u - static_cast <RealType>(0.25);
2126: 
2127:             // Rational Approximation
2128:             // Maximum Relative Error: 7.1265e-35
2129:             // LCOV_EXCL_START
2130:             BOOST_MATH_STATIC const RealType P[14] = {
2131:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.69627866689346442965083437425920959525e-2),
2132:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.28948812330446670380449765578224539665e-1),
2133:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.38832694133021352110245148952631526683e0),
2134:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.70206624753427831733487031852769976576e1),
2135:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34677850226082773550206949299306677736e2),
2136:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.18657422004942861459539366963056149110e2),
2137:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.90933843076824719761937043667767333536e2),
2138:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.78597771586582252472927601403235921029e2),
2139:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76489020985978559079198751910122765603e2),
2140:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.37662018494780327201390375334403954354e1),
2141:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11303058491765900888068268844399186476e1),
2142:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.38147649159947518976483710606042789880e-1),
2143:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.81260575060831053615857196033574207714e-2),
2144:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.26783311530618626413866321968979725353e-3),
2145:             };
2146:             BOOST_MATH_STATIC const RealType Q[13] = {
2147:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2148:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.98941943311823528497840052715295329781e1),
2149:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70142252619301982454969690308614487433e2),
2150:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.17472255695869018956165466705137979540e2),
2151:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.42016169942136311355803413981032780219e3),
2152:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.55874385736597452997483327962434131932e3),
2153:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.45782883079400958761816030672202996788e3),
2154:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.05272877129840019671123017296056938361e3),
2155:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.79833037593794381103412381177370862105e3),
2156:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.67388248713896792948592889733513376054e2),
2157:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.19952164110429183557842014635391021832e1),
2158:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.43813483967503071358907030110791934870e0),
2159:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.21327682641358836049127780506729428797e-1),
2160:             };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2161-2200 / 第 2161-2200 行
~~~cpp
2161:             // LCOV_EXCL_STOP
2162:             result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2163:         }
2164:         else {
2165:             RealType t = u - static_cast <RealType>(0.5);
2166: 
2167:             // Rational Approximation
2168:             // Maximum Relative Error: 2.7048e-37
2169:             // LCOV_EXCL_START
2170:             BOOST_MATH_STATIC const RealType P[15] = {
2171:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.79518653373241051262822702930040975338e-2),
2172:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.62230291299220868262265687829866364204e-1),
2173:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.87315544620612697712513318458226575394e0),
2174:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.38993950875334507399211313740958438201e0),
2175:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.54257654902026056547861805085572437922e1),
2176:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.85673656862223617197701693270067722169e1),
2177:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47842193222521213922734312546590337064e1),
2178:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.76640627287007744941009407221495229316e0),
2179:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.71884893887802925773271837595143776207e0),
2180:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.87432154629995817972739015224205530101e-1),
2181:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44664933176248007092868241686074743562e-1),
2182:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.32094739938150047092982705610586287965e-2),
2183:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.18537678581395571564129512698022192316e-3),
2184:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.99365265557355974918712592061740510276e-4),
2185:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.66467016868206844419002547523627548705e-6),
2186:             };
2187:             BOOST_MATH_STATIC const RealType Q[15] = {
2188:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2189:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.01315080955831561204744043759079263546e1),
2190:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.43409077070585581955481063438385546913e1),
2191:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09863540097812452102765922256432103612e2),
2192:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.69971336507400724019217277303598318934e2),
2193:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71444880426858110981683485927452024652e2),
2194:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15252748520663939799185721687082682973e2),
2195:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.28399989835264172624148638350889215004e1),
2196:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.73464700365199500083227290575797895127e1),
2197:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.40421770918884020099427978511354197438e0),
2198:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.97023025282119988988976542004620759235e-1),
2199:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.38774609088015115009880504176630591783e-1),
2200:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52748138528630655371589047000668876440e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 2201-2240 / 第 2201-2240 行
~~~cpp
2201:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.13088455793478303045390386135591069087e-3),
2202:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.18220605549460262119565543089703387122e-5),
2203:             };
2204:             // LCOV_EXCL_STOP
2205:             result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2206:         }
2207:     }
2208:     else if (ilogb(p) >= -4) {
2209:         RealType t = -log2(ldexp(p, 2));
2210: 
2211:         // Rational Approximation
2212:         // Maximum Relative Error: 3.8969e-35
2213:         // LCOV_EXCL_START
2214:         BOOST_MATH_STATIC const RealType P[18] = {
2215:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.02395484493329839255216366819344305871e-2),
2216:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.02703992140456336967688958960484716694e-1),
2217:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.38779662796374026809611637926067177436e0),
2218:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.22326903547451397450399124548020897393e0),
2219:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.29321119874906326000117036864856138032e0),
2220:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.60045794013093831332658415095234082115e0),
2221:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.75863216252160126657107771372004587438e-1),
2222:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.65658718311497180532644775193008407069e-1),
2223:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.53225259384404343896446164609240157391e-2),
2224:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17876243295156782920260122855798305258e-2),
2225:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59647007234516896762020830535717539733e-3),
2226:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.64519789656979327339865975091579252352e-4),
2227:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.29566724776730544346201080459027524931e-5),
2228:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.40979492647851412567441418477263395917e-7),
2229:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.78894057948338305679452471174923939381e-8),
2230:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.97064244496171921075006182915678263370e-10),
2231:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.13496588267213644899739513941375650458e-13),
2232:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.75224691413667093006312591320754720811e-14),
2233:         };
2234:         BOOST_MATH_STATIC const RealType Q[17] = {
2235:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2236:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.59000688626663121310675150262772434285e0),
2237:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.37518060227321498297232252379976917550e1),
2238:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.96559443266702775026538144474892076437e1),
2239:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.81865210018244220041408788510705356696e1),
2240:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15188291931842064325756652570456168425e1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2241-2280 / 第 2241-2280 行
~~~cpp
2241:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.16909307081950035111952362482113369939e0),
2242:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.68454509269150307761046136063890222011e0),
2243:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.06391236761753712424925832120306727169e-1),
2244:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.35744804731044427608283991933125506859e-2),
2245:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00655928177646208520006978937806043639e-2),
2246:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04093230988242553633939757013466501271e-3),
2247:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.07921031269974885975846184199640060403e-5),
2248:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.61356596082773699708092475561216104426e-6),
2249:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83968520269928804453766899533464507543e-7),
2250:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.44620973323561344735660659502096499899e-9),
2251:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.84398925760354259350870730551452956164e-11),
2252:         };
2253:         // LCOV_EXCL_STOP
2254:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2255:     }
2256:     else if (ilogb(p) >= -8) {
2257:         RealType t = -log2(ldexp(p, 4));
2258: 
2259:         // Rational Approximation
2260:         // Maximum Relative Error: 4.0176e-35
2261:         // LCOV_EXCL_START
2262:         BOOST_MATH_STATIC const RealType P[17] = {
2263:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.39293493266195566603513288406748830312e-1),
2264:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.75724665658983779947977436518056682748e-1),
2265:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.42437549740894393207094008058345312893e-1),
2266:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.26189619865771499663660627120168211026e-1),
2267:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.38952430871711360962228087792821341859e-2),
2268:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09604487371653920602809626594722822237e-2),
2269:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06215021409396534038209460967790566899e-3),
2270:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.02245531075243838209245241246011523536e-4),
2271:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.52822482024384335373072062232322682354e-5),
2272:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.32527687997718638700761890588399465467e-6),
2273:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.54799997015944073019842889902521208940e-7),
2274:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59368565314052950335981455903474908073e-8),
2275:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.09459594346367728583560281313278117879e-10),
2276:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.30296867679720593932307487485758431355e-11),
2277:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04594079707862644415224596859620253913e-12),
2278:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.40274507498190913768918372242285652373e-14),
2279:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.48644117815971872777609922455371868747e-16),
2280:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2281-2320 / 第 2281-2320 行
~~~cpp
2281:         BOOST_MATH_STATIC const RealType Q[17] = {
2282:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2283:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.88079839671202113888025645668230104601e0),
2284:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.58898753182105924446845274197682915131e0),
2285:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.05418719178760837974322764299800701708e-1),
2286:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76439568495464423890950166804368135632e-1),
2287:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.87661284201828717694596419805804620767e-2),
2288:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.29462021166220769918154388930589492957e-2),
2289:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.89960014717788045459266868996575581278e-3),
2290:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.21759236630028632465777310665839652757e-4),
2291:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.08884467282860764261728614542418632608e-5),
2292:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.60098870889198704716300891829788260654e-6),
2293:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00120123451682223443624210304146589040e-7),
2294:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.08868117923724451329261971335574401646e-9),
2295:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.07346130275947166224129347124306950150e-10),
2296:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.57848230665832873347797099944091265220e-12),
2297:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.50841502849442327828534131901583916707e-13),
2298:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19165038770000448560339443014882434202e-15),
2299:         };
2300:         // LCOV_EXCL_STOP
2301:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2302:     }
2303:     else if (ilogb(p) >= -16) {
2304:         RealType t = -log2(ldexp(p, 8));
2305: 
2306:         // Rational Approximation
2307:         // Maximum Relative Error: 4.1682e-36
2308:         // LCOV_EXCL_START
2309:         BOOST_MATH_STATIC const RealType P[17] = {
2310:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.57911660613037766795694241662819364797e-1),
2311:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.28799302413396670477035614399187456630e-1),
2312:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.87488304496324715063356722168914018093e-2),
2313:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15106082041721012436439208357739139578e-2),
2314:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.91744691940169259573871742836817806248e-3),
2315:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.40707390548486625606656777332664791183e-4),
2316:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.37047148097688601398129659532643297674e-5),
2317:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.88039545021930711122085375901243257574e-6),
2318:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.22254460725736448552173288004145978774e-7),
2319:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.58462349007293730244197837509157696852e-9),
2320:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.95242372547984999431208546685672497090e-10),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2321-2360 / 第 2321-2360 行
~~~cpp
2321:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.10113734998651793201123616276573169622e-11),
2322:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.38963677413425618019569452771868834246e-13),
2323:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.45242599273032563942546507899265865936e-15),
2324:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.64855118157117311049698715635863670233e-16),
2325:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.31679318790012894619592273346600264199e-18),
2326:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.97289727214495789126072009268721022605e-20),
2327:         };
2328:         BOOST_MATH_STATIC const RealType Q[17] = {
2329:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2330:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.10184661848812835285809771940181522329e-1),
2331:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.06179300560230499194426573196970342618e-1),
2332:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.23171547302923911058112454487643162794e-2),
2333:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.20486436116678834807354529081908850425e-2),
2334:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.51239574861351183874145649960640500707e-3),
2335:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.48939385253081273966380467344920741615e-4),
2336:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18148716720470800170115047757600735127e-5),
2337:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.68156131480770927662478944117713742978e-7),
2338:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.13720275846166334505537351224097058812e-8),
2339:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.85505701632948614345319635028225905820e-9),
2340:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.91876669388212587242659571229471930880e-11),
2341:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.12971661051277278610784329698988278013e-12),
2342:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.31096179726750865531615367639563072055e-14),
2343:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.03579138802970748888093188937926461893e-15),
2344:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.45570688568663643410924100311054014175e-17),
2345:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23959804461200982866930072222355142173e-19),
2346:         };
2347:         // LCOV_EXCL_STOP
2348:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2349:     }
2350:     else if (ilogb(p) >= -32) {
2351:         RealType t = -log2(ldexp(p, 16));
2352: 
2353:         // Rational Approximation
2354:         // Maximum Relative Error: 6.2158e-37
2355:         // LCOV_EXCL_START
2356:         BOOST_MATH_STATIC const RealType P[17] = {
2357:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59150086070234561732507586188017224084e-1),
2358:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.80849532387385837583114307010320459997e-2),
2359:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41158479406270598752210344238285334672e-2),
2360:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.88824037165656723581890282427897772492e-3),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2361-2400 / 第 2361-2400 行
~~~cpp
2361:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.82912940787568736176030025420621547622e-4),
2362:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.36469458704261637785603215754389736108e-5),
2363:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.13801486421774537025334682673091205328e-7),
2364:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.97058432407176502984043208925327069250e-8),
2365:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.60823277541385163663463406307766614689e-9),
2366:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.45016369260792040947272022706860047646e-11),
2367:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.54846457278644736871929319230398689553e-12),
2368:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.67291749890916930953794688556299297735e-14),
2369:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.18742803398417392282841454979723852423e-16),
2370:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.13337431668170547244474715030235433597e-17),
2371:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.37782648734897338547414800391203459036e-19),
2372:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17863064141234633971470839644872485483e-21),
2373:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.58768205048500915346781559321978174829e-24),
2374:         };
2375:         BOOST_MATH_STATIC const RealType Q[17] = {
2376:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2377:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.27782279546086824129750042200649907991e-1),
2378:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.86934772625607907724733810228981095894e-2),
2379:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18640923531164938140838239032346416143e-2),
2380:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14927915778694317602192656254187608215e-3),
2381:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.57462121236985574785071163024761935943e-5),
2382:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.11326475640883361512750692176665937785e-6),
2383:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.49479333407835032831192117009600622344e-7),
2384:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.01048164044583907219965175201239136609e-8),
2385:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.42444147056333448589159611785359705792e-10),
2386:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.72928365136507710372724683325279209464e-12),
2387:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30776323450676114149657959931149982200e-13),
2388:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.51599272091669693373558919762006698549e-15),
2389:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.12120236145539526122748260176385899774e-17),
2390:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.65713807525694136400636427188839379484e-19),
2391:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.40555520515542383952495965093730818381e-21),
2392:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.51084407433793180162386990118245623958e-23),
2393:         };
2394:         // LCOV_EXCL_STOP
2395:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2396:     }
2397:     else if (ilogb(p) >= -64) {
2398:         RealType t = -log2(ldexp(p, 32));
2399: 
2400:         // Rational Approximation
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2401-2440 / 第 2401-2440 行
~~~cpp
2401:         // Maximum Relative Error: 9.8515e-35
2402:         // LCOV_EXCL_START
2403:         BOOST_MATH_STATIC const RealType P[18] = {
2404:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59154943017783040087729009335921759322e-1),
2405:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.35955784629344586058432079844665517425e-2),
2406:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24333525582177610783141409282489279582e-2),
2407:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.58257137499954581519132407255793210808e-3),
2408:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47191495695958634792434622715063010854e-4),
2409:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06408464185207904662485396901099847317e-5),
2410:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.20796977470988464880970001894205834196e-7),
2411:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.99451680244976178843047944033382023574e-8),
2412:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21331607817814211329055723244764031561e-9),
2413:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.16997758215752306644496702331954449485e-11),
2414:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.22151810180865778439184946086488092970e-12),
2415:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.05017329554372903197056366604190738772e-14),
2416:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.45919279055502465343977575104142733356e-16),
2417:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14611865933281087898817644094411667861e-17),
2418:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.66574579315129285098834562564888533591e-19),
2419:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.90098275536617376789480602467351545227e-21),
2420:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.56200324658873566425094389271790730206e-23),
2421:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.35526648761411463124801128103381691418e-26),
2422:         };
2423:         BOOST_MATH_STATIC const RealType Q[18] = {
2424:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2425:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.99582804063194774835771688139366152937e-1),
2426:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.81210581320456331960539046132284190053e-2),
2427:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.94358920922810097599951120081974275145e-3),
2428:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.24831443209858422294319043037419780210e-4),
2429:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.68584098673893150929178892200446909375e-5),
2430:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.90058244779420124535106788512940199547e-6),
2431:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.88151039746201934320258158884886191886e-7),
2432:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.62348975553160355852344937226490493460e-9),
2433:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.62007418751593938350474825754731470453e-10),
2434:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.67502458979132962529935588245058477825e-12),
2435:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.91648040348401277706598232576212305626e-13),
2436:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.05843052379331618504561151714467880641e-15),
2437:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.20127592059771206959014911028588129920e-17),
2438:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04661894930286305556240859086772458465e-18),
2439:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19442269177165740287568170417649762849e-20),
2440:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.81435584873372180820418114652670864136e-23),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 2441-2480 / 第 2441-2480 行
~~~cpp
2441:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.62145023253666168339801687459484937001e-25),
2442:         };
2443:         // LCOV_EXCL_STOP
2444:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2445:     }
2446:     else if (ilogb(p) >= -128) {
2447:         RealType t = -log2(ldexp(p, 64));
2448: 
2449:         // Rational Approximation
2450:         // Maximum Relative Error: 2.2157e-35
2451:         // LCOV_EXCL_START
2452:         BOOST_MATH_STATIC const RealType P[18] = {
2453:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59154943091895335751628149866310390641e-1),
2454:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.91164927854420277537616294413463565970e-2),
2455:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47557801928232619499125670863084398577e-2),
2456:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06172621625221091203249391660455847328e-3),
2457:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.11720157411653968975956625234656001375e-4),
2458:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70086412127379161257840749700428137407e-5),
2459:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11069186177775505692019195793079552937e-6),
2460:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.04581765901792649215653828121992908775e-8),
2461:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.78996797234624395264657873201296117159e-9),
2462:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.10365978021268853654282661591051834468e-10),
2463:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.76744621013787434243259445839624450867e-12),
2464:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11110170303355425599446515240949433934e-13),
2465:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.83669090335022069229153919882930282425e-15),
2466:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.09633460833089193733622172621696983652e-17),
2467:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16200852052266861122422190933586966917e-18),
2468:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47795810090424252745150042033544310609e-20),
2469:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.35722092370326505616747155207965300634e-22),
2470:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.98381676423023212724768510437325359364e-51),
2471:         };
2472:         BOOST_MATH_STATIC const RealType Q[17] = {
2473:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2474:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.34271731953273239691423485699928257808e-1),
2475:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.27133013035186849140772481980360839559e-2),
2476:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.29542078693828543560388747333519393752e-2),
2477:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33027698228265344561650492600955601983e-3),
2478:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06868444562964057795387778972002636261e-4),
2479:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.97868278672593071151212650783507879919e-6),
2480:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.79869926850283188785885503049178903204e-7),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2481-2520 / 第 2481-2520 行
~~~cpp
2481:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.75298857713475428388051708713106549590e-8),
2482:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.93449891515741631942400181171061740767e-10),
2483:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.36715626731277089044713008494971829270e-11),
2484:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.98125789528264426960496891930942311971e-13),
2485:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.78234546049400950544724588355539821858e-14),
2486:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.83044000387150792693434128054414524740e-16),
2487:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.30111486296552039483196431122555524886e-18),
2488:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.28628462422858135083238952377446358986e-20),
2489:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.48108558735886480298604270981393793162e-21),
2490:         };
2491:         // LCOV_EXCL_STOP
2492:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * (p * p));
2493:     }
2494:     else {
2495:         result = 1 / (p * p * constants::two_pi<RealType>());
2496:     }
2497: 
2498:     return result;
2499: }
2500: 
2501: template <class RealType>
2502: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_quantile_imp_prec(const RealType& p, bool complement, const boost::math::integral_constant<int, 53>& tag)
2503: {
2504:     if (p > 0.5) {
2505:         return !complement ? saspoint5_quantile_upper_imp_prec(1 - p, tag) : -saspoint5_quantile_upper_imp_prec(1 - p, tag);
2506:     }
2507: 
2508:     return complement ? saspoint5_quantile_upper_imp_prec(p, tag) : -saspoint5_quantile_upper_imp_prec(p, tag);
2509: }
2510: 
2511: template <class RealType>
2512: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_quantile_imp_prec(const RealType& p, bool complement, const boost::math::integral_constant<int, 113>& tag)
2513: {
2514:     if (p > 0.5) {
2515:         return !complement ? saspoint5_quantile_upper_imp_prec(1 - p, tag) : -saspoint5_quantile_upper_imp_prec(1 - p, tag);
2516:     }
2517: 
2518:     return complement ? saspoint5_quantile_upper_imp_prec(p, tag) : -saspoint5_quantile_upper_imp_prec(p, tag);
2519: }
2520: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。

### Lines 2521-2560 / 第 2521-2560 行
~~~cpp
2521: template <class RealType, class Policy>
2522: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_quantile_imp(const saspoint5_distribution<RealType, Policy>& dist, const RealType& p, bool complement)
2523: {
2524:     // This routine implements the quantile for the Saspoint5 distribution,
2525:     // the value p may be the probability, or its complement if complement=true.
2526: 
2527:     constexpr auto function = "boost::math::quantile(saspoint5<%1%>&, %1%)";
2528:     BOOST_MATH_STD_USING // for ADL of std functions
2529: 
2530:     RealType result = 0;
2531:     RealType scale = dist.scale();
2532:     RealType location = dist.location();
2533: 
2534:     if (false == detail::check_location(function, location, &result, Policy()))
2535:     {
2536:         return result;
2537:     }
2538:     if (false == detail::check_scale(function, scale, &result, Policy()))
2539:     {
2540:         return result;
2541:     }
2542:     if (false == detail::check_probability(function, p, &result, Policy()))
2543:     {
2544:         return result;
2545:     }
2546: 
2547:     typedef typename tools::promote_args<RealType>::type result_type;
2548:     typedef typename policies::precision<result_type, Policy>::type precision_type;
2549:     typedef boost::math::integral_constant<int,
2550:         precision_type::value <= 0 ? 0 :
2551:         precision_type::value <= 53 ? 53 :
2552:         precision_type::value <= 113 ? 113 : 0
2553:     > tag_type;
2554: 
2555:     static_assert(tag_type::value, "The SaS point5 distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
2556: 
2557:     result = location + scale * saspoint5_quantile_imp_prec(p, complement, tag_type());
2558: 
2559:     return result;
2560: }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, location, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, location, ...。

### Lines 2561-2600 / 第 2561-2600 行
~~~cpp
2561: 
2562: template <class RealType>
2563: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_entropy_imp_prec(const boost::math::integral_constant<int, 53>&)
2564: {
2565:     return static_cast<RealType>(3.63992444568030649573);
2566: }
2567: 
2568: template <class RealType>
2569: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_entropy_imp_prec(const boost::math::integral_constant<int, 113>&)
2570: {
2571:     return BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.6399244456803064957308496039071853510);
2572: }
2573: 
2574: template <class RealType, class Policy>
2575: BOOST_MATH_GPU_ENABLED inline RealType saspoint5_entropy_imp(const saspoint5_distribution<RealType, Policy>& dist)
2576: {
2577:     // This implements the entropy for the Saspoint5 distribution,
2578: 
2579:     constexpr auto function = "boost::math::entropy(saspoint5<%1%>&, %1%)";
2580:     BOOST_MATH_STD_USING // for ADL of std functions
2581: 
2582:     RealType result = 0;
2583:     RealType scale = dist.scale();
2584: 
2585:     if (false == detail::check_scale(function, scale, &result, Policy()))
2586:     {
2587:         return result;
2588:     }
2589: 
2590:     typedef typename tools::promote_args<RealType>::type result_type;
2591:     typedef typename policies::precision<result_type, Policy>::type precision_type;
2592:     typedef boost::math::integral_constant<int,
2593:         precision_type::value <= 0 ? 0 :
2594:         precision_type::value <= 53 ? 53 :
2595:         precision_type::value <= 113 ? 113 : 0
2596:     > tag_type;
2597: 
2598:     static_assert(tag_type::value, "The SaS point5 distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
2599: 
2600:     result = saspoint5_entropy_imp_prec<RealType>(tag_type()) + log(scale);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, static_assert, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, static_assert, ...。

### Lines 2601-2640 / 第 2601-2640 行
~~~cpp
2601: 
2602:     return result;
2603: }
2604: 
2605: } // detail
2606: 
2607: template <class RealType = double, class Policy = policies::policy<> >
2608: class saspoint5_distribution
2609: {
2610:     public:
2611:     typedef RealType value_type;
2612:     typedef Policy policy_type;
2613: 
2614:     BOOST_MATH_GPU_ENABLED saspoint5_distribution(RealType l_location = 0, RealType l_scale = 1)
2615:         : mu(l_location), c(l_scale)
2616:     {
2617:         constexpr auto function = "boost::math::saspoint5_distribution<%1%>::saspoint5_distribution";
2618:         RealType result = 0;
2619:         detail::check_location(function, l_location, &result, Policy());
2620:         detail::check_scale(function, l_scale, &result, Policy());
2621:     } // saspoint5_distribution
2622: 
2623:     BOOST_MATH_GPU_ENABLED RealType location()const
2624:     {
2625:         return mu;
2626:     }
2627:     BOOST_MATH_GPU_ENABLED RealType scale()const
2628:     {
2629:         return c;
2630:     }
2631: 
2632:     private:
2633:     RealType mu;    // The location parameter.
2634:     RealType c;     // The scale parameter.
2635: };
2636: 
2637: typedef saspoint5_distribution<double> saspoint5;
2638: 
2639: #ifdef __cpp_deduction_guides
2640: template <class RealType>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as detail::check_location, detail::check_scale.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 detail::check_location, detail::check_scale。

### Lines 2641-2680 / 第 2641-2680 行
~~~cpp
2641: saspoint5_distribution(RealType) -> saspoint5_distribution<typename boost::math::tools::promote_args<RealType>::type>;
2642: template <class RealType>
2643: saspoint5_distribution(RealType, RealType) -> saspoint5_distribution<typename boost::math::tools::promote_args<RealType>::type>;
2644: #endif
2645: 
2646: template <class RealType, class Policy>
2647: BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const saspoint5_distribution<RealType, Policy>&)
2648: { // Range of permissible values for random variable x.
2649:     BOOST_MATH_IF_CONSTEXPR (boost::math::numeric_limits<RealType>::has_infinity)
2650:     {
2651:         return boost::math::pair<RealType, RealType>(-boost::math::numeric_limits<RealType>::infinity(), boost::math::numeric_limits<RealType>::infinity()); // - to + infinity.
2652:     }
2653:     else
2654:     { // Can only use max_value.
2655:         using boost::math::tools::max_value;
2656:         return boost::math::pair<RealType, RealType>(-max_value<RealType>(), max_value<RealType>()); // - to + max.
2657:     }
2658: }
2659: 
2660: template <class RealType, class Policy>
2661: BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const saspoint5_distribution<RealType, Policy>&)
2662: { // Range of supported values for random variable x.
2663:    // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
2664:     BOOST_MATH_IF_CONSTEXPR (boost::math::numeric_limits<RealType>::has_infinity)
2665:     {
2666:         return boost::math::pair<RealType, RealType>(-boost::math::numeric_limits<RealType>::infinity(), boost::math::numeric_limits<RealType>::infinity()); // - to + infinity.
2667:     }
2668:     else
2669:     { // Can only use max_value.
2670:         using boost::math::tools::max_value;
2671:         return boost::math::pair<RealType, RealType>(-tools::max_value<RealType>(), max_value<RealType>()); // - to + max.
2672:     }
2673: }
2674: 
2675: template <class RealType, class Policy>
2676: BOOST_MATH_GPU_ENABLED inline RealType pdf(const saspoint5_distribution<RealType, Policy>& dist, const RealType& x)
2677: {
2678:     return detail::saspoint5_pdf_imp(dist, x);
2679: } // pdf
2680: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 2681-2720 / 第 2681-2720 行
~~~cpp
2681: template <class RealType, class Policy>
2682: BOOST_MATH_GPU_ENABLED inline RealType cdf(const saspoint5_distribution<RealType, Policy>& dist, const RealType& x)
2683: {
2684:     return detail::saspoint5_cdf_imp(dist, x, false);
2685: } // cdf
2686: 
2687: template <class RealType, class Policy>
2688: BOOST_MATH_GPU_ENABLED inline RealType quantile(const saspoint5_distribution<RealType, Policy>& dist, const RealType& p)
2689: {
2690:     return detail::saspoint5_quantile_imp(dist, p, false);
2691: } // quantile
2692: 
2693: template <class RealType, class Policy>
2694: BOOST_MATH_GPU_ENABLED inline RealType cdf(const complemented2_type<saspoint5_distribution<RealType, Policy>, RealType>& c)
2695: {
2696:     return detail::saspoint5_cdf_imp(c.dist, c.param, true);
2697: } //  cdf complement
2698: 
2699: template <class RealType, class Policy>
2700: BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<saspoint5_distribution<RealType, Policy>, RealType>& c)
2701: {
2702:     return detail::saspoint5_quantile_imp(c.dist, c.param, true);
2703: } // quantile complement
2704: 
2705: template <class RealType, class Policy>
2706: BOOST_MATH_GPU_ENABLED inline RealType mean(const saspoint5_distribution<RealType, Policy> &dist)
2707: {
2708:     // There is no mean:
2709:     typedef typename Policy::assert_undefined_type assert_type;
2710:     static_assert(assert_type::value == 0, "The SaS point5 Distribution has no mean");
2711: 
2712:     return policies::raise_domain_error<RealType>(
2713:         "boost::math::mean(saspoint5<%1%>&)",
2714:         "The SaS point5 distribution does not have a mean: "
2715:         "the only possible return value is %1%.",
2716:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy());
2717: }
2718: 
2719: template <class RealType, class Policy>
2720: BOOST_MATH_GPU_ENABLED inline RealType variance(const saspoint5_distribution<RealType, Policy>& /*dist*/)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, quiet_NaN.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, quiet_NaN。

### Lines 2721-2760 / 第 2721-2760 行
~~~cpp
2721: {
2722:     // There is no variance:
2723:     typedef typename Policy::assert_undefined_type assert_type;
2724:     static_assert(assert_type::value == 0, "The SaS point5 Distribution has no variance");
2725: 
2726:     return policies::raise_domain_error<RealType>(
2727:         "boost::math::variance(saspoint5<%1%>&)",
2728:         "The SaS point5 distribution does not have a variance: "
2729:         "the only possible return value is %1%.",
2730:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy());
2731: }
2732: 
2733: template <class RealType, class Policy>
2734: BOOST_MATH_GPU_ENABLED inline RealType mode(const saspoint5_distribution<RealType, Policy>& dist)
2735: {
2736:     return dist.location();
2737: }
2738: 
2739: template <class RealType, class Policy>
2740: BOOST_MATH_GPU_ENABLED inline RealType median(const saspoint5_distribution<RealType, Policy>& dist)
2741: {
2742:     return dist.location();
2743: }
2744: 
2745: template <class RealType, class Policy>
2746: BOOST_MATH_GPU_ENABLED inline RealType skewness(const saspoint5_distribution<RealType, Policy>& /*dist*/)
2747: {
2748:     // There is no skewness:
2749:     typedef typename Policy::assert_undefined_type assert_type;
2750:     static_assert(assert_type::value == 0, "The SaS point5 Distribution has no skewness");
2751: 
2752:     return policies::raise_domain_error<RealType>(
2753:         "boost::math::skewness(saspoint5<%1%>&)",
2754:         "The SaS point5 distribution does not have a skewness: "
2755:         "the only possible return value is %1%.",
2756:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy()); // infinity?
2757: }
2758: 
2759: template <class RealType, class Policy>
2760: BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const saspoint5_distribution<RealType, Policy>& /*dist*/)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, quiet_NaN.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, quiet_NaN。

### Lines 2761-2796 / 第 2761-2796 行
~~~cpp
2761: {
2762:     // There is no kurtosis:
2763:     typedef typename Policy::assert_undefined_type assert_type;
2764:     static_assert(assert_type::value == 0, "The SaS point5 Distribution has no kurtosis");
2765: 
2766:     return policies::raise_domain_error<RealType>(
2767:         "boost::math::kurtosis(saspoint5<%1%>&)",
2768:         "The SaS point5 distribution does not have a kurtosis: "
2769:         "the only possible return value is %1%.",
2770:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy());
2771: }
2772: 
2773: template <class RealType, class Policy>
2774: BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const saspoint5_distribution<RealType, Policy>& /*dist*/)
2775: {
2776:     // There is no kurtosis excess:
2777:     typedef typename Policy::assert_undefined_type assert_type;
2778:     static_assert(assert_type::value == 0, "The SaS point5 Distribution has no kurtosis excess");
2779: 
2780:     return policies::raise_domain_error<RealType>(
2781:         "boost::math::kurtosis_excess(saspoint5<%1%>&)",
2782:         "The SaS point5 distribution does not have a kurtosis: "
2783:         "the only possible return value is %1%.",
2784:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy());
2785: }
2786: 
2787: template <class RealType, class Policy>
2788: BOOST_MATH_GPU_ENABLED inline RealType entropy(const saspoint5_distribution<RealType, Policy>& dist)
2789: {
2790:     return detail::saspoint5_entropy_imp(dist);
2791: }
2792: 
2793: }} // namespaces
2794: 
2795: 
2796: #endif // BOOST_STATS_SASPOINT5_HPP
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。

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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/tuple.hpp, boost/math/constants/constants.hpp, boost/math/tools/big_constant.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/detail/derived_accessors.hpp, boost/math/tools/rational.hpp, boost/math/tools/promotion.hpp, boost/math/tools/type_traits.hpp, boost/math/policies/policy.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `tools::evaluate_polynomial, if, sqrt, saspoint5_pdf_imp_prec, saspoint5_pdf_imp, location, scale, static_assert, ...`
