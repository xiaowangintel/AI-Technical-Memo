# mapairy.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/mapairy.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the mapairy distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 mapairy 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48 / 第 1-48 行
~~~cpp
   1: //  Copyright Takuma Yoshimura 2024.
   2: //  Copyright Matt Borland 2024.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_STATS_MAPAIRY_HPP
   8: #define BOOST_STATS_MAPAIRY_HPP
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
  24: #include <boost/math/special_functions/cbrt.hpp>
  25: #include <boost/math/policies/policy.hpp>
  26: #include <boost/math/policies/error_handling.hpp>
  27: #include <boost/math/tools/promotion.hpp>
  28: 
  29: #ifndef BOOST_MATH_HAS_NVRTC
  30: #include <boost/math/distributions/fwd.hpp>
  31: #include <cmath>
  32: #endif
  33: 
  34: namespace boost { namespace math {
  35: template <class RealType, class Policy>
  36: class mapairy_distribution;
  37: 
  38: namespace detail {
  39: 
  40: template <class RealType>
  41: BOOST_MATH_GPU_ENABLED inline RealType mapairy_pdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53>&)
  42: {
  43:     BOOST_MATH_STD_USING
  44:     RealType result;
  45: 
  46:     if (x < 1) {
  47:         // Rational Approximation
  48:         // Maximum Relative Error: 3.7591e-18
~~~
- **EN:** This block imports dependencies such as boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/tuple.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/tuple.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 49-96 / 第 49-96 行
~~~cpp
  49:         BOOST_MATH_STATIC const RealType P[8] = {
  50:             static_cast<RealType>(1.97516171847191855610e-1),
  51:             static_cast<RealType>(3.67488253628465083737e-2),
  52:             static_cast<RealType>(-9.73242224038828612673e-4),
  53:             static_cast<RealType>(2.32207514136635673061e-3),
  54:             static_cast<RealType>(5.69067907423210669037e-5),
  55:             static_cast<RealType>(-6.02637387141524535193e-5),
  56:             static_cast<RealType>(1.04960324426666933327e-5),
  57:             static_cast<RealType>(-6.58470237954242016920e-7),
  58:         };
  59:         BOOST_MATH_STATIC const RealType Q[7] = {
  60:             static_cast<RealType>(1.),
  61:             static_cast<RealType>(7.09464351647314165710e-1),
  62:             static_cast<RealType>(3.66413036246461392316e-1),
  63:             static_cast<RealType>(1.10947882302862241488e-1),
  64:             static_cast<RealType>(2.65928486676817177159e-2),
  65:             static_cast<RealType>(3.75507284977386290874e-3),
  66:             static_cast<RealType>(4.03789594641339005785e-4),
  67:         };
  68: 
  69:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
  70:     }
  71:     else if (x < 2) {
  72:         RealType t = x - 1;
  73: 
  74:         // Rational Approximation
  75:         // Maximum Relative Error: 1.5996e-20
  76:         BOOST_MATH_STATIC const RealType P[8] = {
  77:             static_cast<RealType>(1.06251243013238748252e-1),
  78:             static_cast<RealType>(1.38178831205785069108e-2),
  79:             static_cast<RealType>(4.19280374368049006206e-3),
  80:             static_cast<RealType>(8.54607219684690930289e-4),
  81:             static_cast<RealType>(-7.46881084120928210702e-5),
  82:             static_cast<RealType>(1.47110856483345063335e-5),
  83:             static_cast<RealType>(-1.30090180307471994500e-6),
  84:             static_cast<RealType>(5.24801123304330014713e-8),
  85:         };
  86:         BOOST_MATH_STATIC const RealType Q[8] = {
  87:             static_cast<RealType>(1.),
  88:             static_cast<RealType>(8.10853683888611687140e-1),
  89:             static_cast<RealType>(3.89361261627717143905e-1),
  90:             static_cast<RealType>(1.15124062681082170577e-1),
  91:             static_cast<RealType>(2.38803416611949902468e-2),
  92:             static_cast<RealType>(3.08616898814509065071e-3),
  93:             static_cast<RealType>(2.43760043942846261876e-4),
  94:             static_cast<RealType>(1.34538901435238836768e-6),
  95:         };
  96: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 97-144 / 第 97-144 行
~~~cpp
  97:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
  98:     }
  99:     else if (x < 4) {
 100:         RealType t = x - 2;
 101: 
 102:         // Rational Approximation
 103:         // Maximum Relative Error: 1.1592e-17
 104:         BOOST_MATH_STATIC const RealType P[9] = {
 105:             static_cast<RealType>(5.33842514891989443409e-2),
 106:             static_cast<RealType>(1.23301980674903270971e-2),
 107:             static_cast<RealType>(3.45717831433988631923e-3),
 108:             static_cast<RealType>(3.27034449923176875761e-4),
 109:             static_cast<RealType>(1.20406794831890291348e-5),
 110:             static_cast<RealType>(5.77489170397965604669e-7),
 111:             static_cast<RealType>(-1.15255267205685159063e-7),
 112:             static_cast<RealType>(9.15896323073109992939e-9),
 113:             static_cast<RealType>(-3.14068002815368247985e-10),
 114:         };
 115:         BOOST_MATH_STATIC const RealType Q[7] = {
 116:             static_cast<RealType>(1.),
 117:             static_cast<RealType>(9.08772985520393226044e-1),
 118:             static_cast<RealType>(4.26418573702560818267e-1),
 119:             static_cast<RealType>(1.22033746594868893316e-1),
 120:             static_cast<RealType>(2.27934009200310243172e-2),
 121:             static_cast<RealType>(2.60658999011198623962e-3),
 122:             static_cast<RealType>(1.54461660261435227768e-4),
 123:         };
 124: 
 125:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 126:     }
 127:     else if (x < 8) {
 128:         RealType t = x - 4;
 129: 
 130:         // Rational Approximation
 131:         // Maximum Relative Error: 9.2228e-17
 132:         BOOST_MATH_STATIC const RealType P[8] = {
 133:             static_cast<RealType>(1.58950538583133457384e-2),
 134:             static_cast<RealType>(7.47835440063141601948e-3),
 135:             static_cast<RealType>(1.81137244353261478410e-3),
 136:             static_cast<RealType>(2.26935565382135588558e-4),
 137:             static_cast<RealType>(1.43877113825683795505e-5),
 138:             static_cast<RealType>(2.08242747557417233626e-7),
 139:             static_cast<RealType>(-1.54976465724771282989e-9),
 140:             static_cast<RealType>(1.30762989300333026019e-11),
 141:         };
 142:         BOOST_MATH_STATIC const RealType Q[8] = {
 143:             static_cast<RealType>(1.),
 144:             static_cast<RealType>(9.95505437381674174441e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 145-192 / 第 145-192 行
~~~cpp
 145:             static_cast<RealType>(4.58882737262511297099e-1),
 146:             static_cast<RealType>(1.25031310192148865496e-1),
 147:             static_cast<RealType>(2.15727229249904102247e-2),
 148:             static_cast<RealType>(2.33597081566665672569e-3),
 149:             static_cast<RealType>(1.45198998318300328562e-4),
 150:             static_cast<RealType>(3.87962234445835345676e-6),
 151:         };
 152: 
 153:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 154:     }
 155:     else if (x < 16) {
 156:         RealType t = x - 8;
 157: 
 158:         // Rational Approximation
 159:         // Maximum Relative Error: 1.0257e-17
 160:         BOOST_MATH_STATIC const RealType P[8] = {
 161:             static_cast<RealType>(3.22517551525042172428e-3),
 162:             static_cast<RealType>(1.12822806030796339659e-3),
 163:             static_cast<RealType>(1.54489389961322571031e-4),
 164:             static_cast<RealType>(9.28479992527909796427e-6),
 165:             static_cast<RealType>(2.06168350199745832262e-7),
 166:             static_cast<RealType>(9.05110751997021418539e-10),
 167:             static_cast<RealType>(-2.15498112371756202097e-12),
 168:             static_cast<RealType>(6.41838355699777435924e-15),
 169:         };
 170:         BOOST_MATH_STATIC const RealType Q[8] = {
 171:             static_cast<RealType>(1.),
 172:             static_cast<RealType>(6.53390465399680164234e-1),
 173:             static_cast<RealType>(1.82759048270449018482e-1),
 174:             static_cast<RealType>(2.80407546367978533849e-2),
 175:             static_cast<RealType>(2.50853443923476718145e-3),
 176:             static_cast<RealType>(1.27671852825846245421e-4),
 177:             static_cast<RealType>(3.28380135691060279203e-6),
 178:             static_cast<RealType>(3.06545317089055335742e-8),
 179:         };
 180: 
 181:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 182:     }
 183:     else if (x < 32) {
 184:         RealType t = x - 16;
 185: 
 186:         // Rational Approximation
 187:         // Maximum Relative Error: 6.0510e-17
 188:         BOOST_MATH_STATIC const RealType P[8] = {
 189:             static_cast<RealType>(5.82527663232857270992e-4),
 190:             static_cast<RealType>(6.89502117025124630567e-5),
 191:             static_cast<RealType>(2.24909795087265741433e-6),
 192:             static_cast<RealType>(2.18576787334972903790e-8),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 193-240 / 第 193-240 行
~~~cpp
 193:             static_cast<RealType>(3.39014723444178274435e-11),
 194:             static_cast<RealType>(-9.74481309265612390297e-15),
 195:             static_cast<RealType>(-1.13308546492906818388e-16),
 196:             static_cast<RealType>(5.32472028720777735712e-19),
 197:         };
 198:         BOOST_MATH_STATIC const RealType Q[7] = {
 199:             static_cast<RealType>(1.),
 200:             static_cast<RealType>(2.74018883667663396766e-1),
 201:             static_cast<RealType>(2.95901195665990089660e-2),
 202:             static_cast<RealType>(1.57901733512147920251e-3),
 203:             static_cast<RealType>(4.24965124147621236633e-5),
 204:             static_cast<RealType>(5.17522027193205842016e-7),
 205:             static_cast<RealType>(2.00522219276570039934e-9),
 206:         };
 207: 
 208:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 209:     }
 210:     else if (x < 64) {
 211:         RealType t = x - 32;
 212: 
 213:         // Rational Approximation
 214:         // Maximum Relative Error: 7.3294e-18
 215:         BOOST_MATH_STATIC const RealType P[8] = {
 216:             static_cast<RealType>(1.03264853379349880039e-4),
 217:             static_cast<RealType>(5.35256306644392405447e-6),
 218:             static_cast<RealType>(9.00657716972118816692e-8),
 219:             static_cast<RealType>(5.34913574042209793720e-10),
 220:             static_cast<RealType>(6.70752605041678779380e-13),
 221:             static_cast<RealType>(-5.30089923101856817552e-16),
 222:             static_cast<RealType>(7.28133811621687143754e-19),
 223:             static_cast<RealType>(-7.38047553655951666420e-22),
 224:         };
 225:         BOOST_MATH_STATIC const RealType Q[7] = {
 226:             static_cast<RealType>(1.),
 227:             static_cast<RealType>(1.29920843258164337377e-1),
 228:             static_cast<RealType>(6.75018577147646502386e-3),
 229:             static_cast<RealType>(1.77694968039695671819e-4),
 230:             static_cast<RealType>(2.46428299911920942946e-6),
 231:             static_cast<RealType>(1.67165053157990942546e-8),
 232:             static_cast<RealType>(4.19496974141131087116e-11),
 233:         };
 234: 
 235:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 236:     }
 237:     else {
 238:         RealType t = 1 / sqrt(x * x * x);
 239: 
 240:         // Rational Approximation
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 241-288 / 第 241-288 行
~~~cpp
 241:         // Maximum Relative Error: 5.6693e-20
 242:         BOOST_MATH_STATIC const RealType P[5] = {
 243:             static_cast<RealType>(5.98413420602149016910e-1),
 244:             static_cast<RealType>(3.14584075817417883086e-5),
 245:             static_cast<RealType>(1.62977928311793051895e1),
 246:             static_cast<RealType>(-4.12903117172994371875e-4),
 247:             static_cast<RealType>(-1.06404478702135751872e2),
 248:         };
 249:         BOOST_MATH_STATIC const RealType Q[3] = {
 250:             static_cast<RealType>(1.),
 251:             static_cast<RealType>(5.25696892802060720079e-5),
 252:             static_cast<RealType>(4.03600055498020483920e1),
 253:         };
 254: 
 255:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t / x;
 256:     }
 257: 
 258:     return result;
 259: }
 260: 
 261: 
 262: template <class RealType>
 263: BOOST_MATH_GPU_ENABLED inline RealType mapairy_pdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>&)
 264: {
 265:     BOOST_MATH_STD_USING
 266:     RealType result;
 267: 
 268:     if (x < 1) {
 269:         // Rational Approximation
 270:         // Maximum Relative Error: 7.8308e-35
 271:         // LCOV_EXCL_START
 272:         BOOST_MATH_STATIC const RealType P[13] = {
 273:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.97516171847191855609649452292217911973e-1),
 274:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17531822787252717270400174744562144891e-1),
 275:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.85115358761409188259685286269086053296e-2),
 276:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18029395189535552537870932989876189597e-2),
 277:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.77412874842522285996566741532939343827e-3),
 278:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.77992070255086842672551073580133785334e-4),
 279:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.54573264286260796576738952968288691782e-5),
 280:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.94764012694602906119831079380500255557e-6),
 281:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.97596258932025712802674070104281981323e-7),
 282:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.45466169112247379589927514614067756956e-8),
 283:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.99760415118300349769641418430273526815e-10),
 284:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43150486566834492207695241913522311930e-13),
 285:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.46130347604880355784938321408765318948e-13),
 286:         };
 287:         BOOST_MATH_STATIC const RealType Q[13] = {
 288:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 289-336 / 第 289-336 行
~~~cpp
 289:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11845869711743584628289654085905424438e0),
 290:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.80391154854347711297249357734993136108e-1),
 291:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.75628443538173255184583966965162835227e-1),
 292:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41016303833742742212624596040074202424e-1),
 293:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.19142300833563644046500846364541891138e-2),
 294:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02421707708633106515934651956262614532e-2),
 295:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.03973732602338507411104824853671547615e-3),
 296:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.35206168908201402570766383018708660819e-4),
 297:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.38602606623008690327520130558254165564e-5),
 298:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.53740175911385378188372963739884519312e-6),
 299:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.27513004715414297729539702862351044344e-7),
 300:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.54510493017251997793679126704007098265e-8),
 301:         };
 302:         // LCOV_EXCL_STOP
 303:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
 304:     }
 305:     else if (x < 2) {
 306:         RealType t = x - 1;
 307: 
 308:         // Rational Approximation
 309:         // Maximum Relative Error: 3.0723e-35
 310:         // LCOV_EXCL_START
 311:         BOOST_MATH_STATIC const RealType P[12] = {
 312:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06251243013238748252181151646220197947e-1),
 313:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.92438638323563234519452281479338921158e-2),
 314:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.83335793178622701784730867677919844599e-2),
 315:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.84159075203218824591724451142550478306e-3),
 316:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.04213732090358859917896442076931334722e-3),
 317:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.72388220651785798237487005913708387756e-4),
 318:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.36099324022668533012286817710272936865e-5),
 319:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.74483270731217433628720245792741986795e-6),
 320:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.56461597064783966758904403291149549559e-7),
 321:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.28590608939674970691948223694855264817e-8),
 322:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.81756745849477762773082030302943341729e-10),
 323:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.65915115243311285178083515017249358853e-12),
 324:         };
 325:         BOOST_MATH_STATIC const RealType Q[13] = {
 326:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 327:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33250387018216706082200927591739589024e0),
 328:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.71707718560216685629188467984384070512e-1),
 329:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.81316277289673837399162302797006618384e-1),
 330:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.78475951599121894570443981591530879087e-1),
 331:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.16167801098514576400689883575304687623e-2),
 332:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19167794366424137722223009369062644830e-2),
 333:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.20831082064982892777497773490792080382e-3),
 334:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.27196399162146247210036306870401328410e-4),
 335:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.79335434374966775903734846875100087590e-5),
 336:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.30825409557870847168672662674521614782e-6),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 337-384 / 第 337-384 行
~~~cpp
 337:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.97296173230649275943984471731360073540e-7),
 338:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.48943057909563158917114503727080517958e-9),
 339:         };
 340:         // LCOV_EXCL_STOP
 341:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 342:     }
 343:     else if (x < 4) {
 344:         RealType t = x - 2;
 345: 
 346:         // Rational Approximation
 347:         // Maximum Relative Error: 4.0903e-35
 348:         // LCOV_EXCL_START
 349:         BOOST_MATH_STATIC const RealType P[13] = {
 350:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.33842514891989443409465171800884519331e-2),
 351:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.53264053296761245408991932692426094424e-2),
 352:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.23210520807186629205810670362048049836e-2),
 353:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.71104271443590027208545022968625306496e-3),
 354:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.98781446716778138729774954595209697813e-3),
 355:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.98895829308616657174932023565302947632e-4),
 356:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.25993639218721804661037829873135732687e-5),
 357:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.64669776700609853276056375742089715662e-6),
 358:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.11846243382610611156151291892877027869e-7),
 359:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.74830086064868141326053648144496072795e-8),
 360:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.07549997153431643849551871367000763445e-9),
 361:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.10030596535721362628619523622308581344e-11),
 362:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19376016170255697546854583591494809062e-13),
 363:         };
 364:         BOOST_MATH_STATIC const RealType Q[15] = {
 365:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 366:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52686177278870816414637961315363468426e0),
 367:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19872083945442288336636376283295310445e0),
 368:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.26633866969676511944680471882188527224e-1),
 369:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.41261867539396133951024374504099977090e-1),
 370:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.18852182132645783844766153200510014113e-2),
 371:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70152126044106007357033814742158353948e-2),
 372:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.23810508827493234517751339979902448944e-3),
 373:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.96161313274648769113605163816403306967e-4),
 374:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.06693316156193327359541953619174255726e-5),
 375:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.79366356086062616343285660797389238271e-6),
 376:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.14585835815353770175366834099001313472e-7),
 377:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.05314631662369743547568064896403143693e-8),
 378:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.90325380271096603676911761784650800378e-10),
 379:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.36933359079566550212098911224675011839e-12),
 380:         };
 381:         // LCOV_EXCL_STOP
 382:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 383:     }
 384:     else if (x < 8) {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 385-432 / 第 385-432 行
~~~cpp
 385:         RealType t = x - 4;
 386: 
 387:         // Rational Approximation
 388:         // Maximum Relative Error: 6.5015e-35
 389:         // LCOV_EXCL_START
 390:         BOOST_MATH_STATIC const RealType P[15] = {
 391:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.58950538583133457383574346194006716984e-2),
 392:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.25447644411503971725638816502617490834e-2),
 393:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47605882774114100209665040117276675598e-2),
 394:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.12224864838900383464124716266085521485e-3),
 395:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.79164249640537972514574059182421325541e-3),
 396:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.89668438166714230032406615413991628135e-4),
 397:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.44410389750700463263686630222653669837e-5),
 398:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.14788978994687095829140113472609739982e-6),
 399:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.79821680629333600844514042061772236495e-7),
 400:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.49636960435731257154960798035854124639e-8),
 401:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.70554745768928821263556963261516872171e-9),
 402:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.42293994855343109617040824208078534205e-11),
 403:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.37599287094703195312894833570340165019e-12),
 404:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.35248179978735448062307216459232932068e-15),
 405:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.53569375838863862590910010617140120876e-18),
 406:         };
 407:         BOOST_MATH_STATIC const RealType Q[17] = {
 408:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 409:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.94337325681904859647161946168957959628e0),
 410:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.77120402023938328899162557073347121463e0),
 411:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.01644685191130734907530007424741314392e0),
 412:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.12479655123720440909164080517207084404e-1),
 413:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.25556010526357752360439314019567992245e-1),
 414:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.96143273204038192262150849394970544022e-2),
 415:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.50612932318889495209230176354364299236e-3),
 416:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.12160918304376427109905628326638480473e-4),
 417:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.47696044292604039527013647985997661762e-5),
 418:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.64067652576843720823459199100800335854e-6),
 419:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.00745166063635113130434111509648306420e-7),
 420:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.05398901239421768403763864060147286105e-8),
 421:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.05698259572340563109985785513355912114e-9),
 422:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19362835269415404005406782719825077472e-11),
 423:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.15444386779802728200716489787161419304e-13),
 424:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.02452666470008756043350040893761339083e-16),
 425:         };
 426:         // LCOV_EXCL_STOP
 427:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 428:     }
 429:     else if (x < 16) {
 430:         RealType t = x - 8;
 431: 
 432:         // Rational Approximation
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 433-480 / 第 433-480 行
~~~cpp
 433:         // Maximum Relative Error: 2.0995e-35
 434:         // LCOV_EXCL_START
 435:         BOOST_MATH_STATIC const RealType P[16] = {
 436:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.22517551525042172427941302520759668293e-3),
 437:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.86576974828476461442549217748945498966e-3),
 438:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18419822818191546598384139622512477000e-3),
 439:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.98396184944524020019688823190946146641e-4),
 440:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.06686400532599396487775148973665625687e-5),
 441:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.05680178109228687159829475615095925679e-6),
 442:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.17554487015345146749705505971350254902e-7),
 443:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.14774751685364429557883242232797329274e-8),
 444:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33266124509168360207594600356349282805e-9),
 445:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.76332756800842989348756910429214676252e-11),
 446:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.60639771339252642992277508068105926919e-13),
 447:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.41859490403554144799385471141184829903e-15),
 448:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.77177795293424055655391515546880774987e-17),
 449:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76106923344461402353501262620681801053e-20),
 450:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.68829978902134103249656805130103045021e-23),
 451:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.42496376687241918803028631991083570963e-26),
 452:         };
 453:         BOOST_MATH_STATIC const RealType Q[16] = {
 454:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 455:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19213376162053391168605415200906099633e0),
 456:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.65578261958732385181558047087365997878e-1),
 457:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30046653564394292929001223763106276016e-1),
 458:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.48388301731958697028701215596777178117e-2),
 459:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.50873786049439122933188684993719288258e-3),
 460:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23255654647151798865208394342856435797e-3),
 461:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.20861791399969402003082323686080041040e-4),
 462:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.96882049090731653763684812243275884213e-6),
 463:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.98669985741073085290012296575736698103e-7),
 464:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.03383311816835346577432387682379226740e-8),
 465:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.87320682938150375144724980774245810905e-10),
 466:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.13573468677076838075146150847170057373e-11),
 467:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34526045003716422620879156626237175127e-13),
 468:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.35681579117696161282979297336282783473e-16),
 469:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.92944288060269290125987728528698476197e-18),
 470:         };
 471:         // LCOV_EXCL_STOP
 472:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 473:     }
 474:     else if (x < 32) {
 475:         RealType t = x - 16;
 476: 
 477:         // Rational Approximation
 478:         // Maximum Relative Error: 2.0937e-35
 479:         // LCOV_EXCL_START
 480:         BOOST_MATH_STATIC const RealType P[15] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 481-528 / 第 481-528 行
~~~cpp
 481:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.82527663232857270992129793621400616909e-4),
 482:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.41696401156754081476312871174198295322e-4),
 483:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.42036620449365724707919875710197564857e-5),
 484:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.67076745288708619632303078677641380627e-6),
 485:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.14278954094278648593125010577441869646e-7),
 486:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40092485054621853149602511539550254471e-8),
 487:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.17755660009065973828053533035808718033e-10),
 488:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.23871371557251644837598540542648782066e-12),
 489:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04069998646037977439620128812310273053e-13),
 490:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.94055978349016208777803296823455779097e-16),
 491:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.29866428982892883091537921429389750973e-18),
 492:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.06056281963023929277728535486590256573e-21),
 493:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.57963857545037466186123981516026589992e-24),
 494:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.81390322233700529779563477285232205886e-28),
 495:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.52190981930441828041102818178755246228e-31),
 496:         };
 497:         BOOST_MATH_STATIC const RealType Q[15] = {
 498:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 499:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.70564782441895707961338319466546005093e-1),
 500:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47770566490107388849474183308889339231e-1),
 501:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.29364672385303439788399215507370006639e-2),
 502:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.37279274083988250795581105436675097881e-3),
 503:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.72124151284421794872333348562536468054e-4),
 504:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.96970247774973902625712414297788402746e-6),
 505:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.38395055453444011915661055983937917120e-7),
 506:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.19605460410208704830882138883730331113e-9),
 507:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.76945301389475508747530234950023648137e-10),
 508:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.33624384932503964160642677987886086890e-12),
 509:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.01155130710615988897664213446593907596e-14),
 510:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.03959317021567084067518847978890548086e-16),
 511:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.78213669817351488671519066803835958715e-19),
 512:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.75492332026736176991870807903277324902e-22),
 513:         };
 514:         // LCOV_EXCL_STOP
 515:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 516:     }
 517:     else if (x < 64) {
 518:         RealType t = x - 32;
 519: 
 520:         // Rational Approximation
 521:         // Maximum Relative Error: 1.5856e-35
 522:         // LCOV_EXCL_START
 523:         BOOST_MATH_STATIC const RealType P[15] = {
 524:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.03264853379349880038687006045193401399e-4),
 525:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.79539964604630527636184900467871907171e-5),
 526:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34840369549460790638336121351837912308e-6),
 527:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.73087351972154879439617719914590729748e-8),
 528:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.51775493325347153520115736204545037264e-9),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 529-576 / 第 529-576 行
~~~cpp
 529:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.60104651860674451546102708885530128768e-11),
 530:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.90233449697112559539826150932808197444e-13),
 531:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06978852724410115655105118663137681992e-15),
 532:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.00399855296672416041126220131900937128e-18),
 533:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.18139748830278263202087699889457673035e-20),
 534:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.43070756487288399784700274808326343543e-23),
 535:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.70126687893706466023887757573369648552e-27),
 536:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.29405234560873665664952418690159194840e-30),
 537:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.69069082510020066864633718082941688708e-34),
 538:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.33468198065176301137949068264633336529e-37),
 539:         };
 540:         BOOST_MATH_STATIC const RealType Q[14] = {
 541:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 542:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.51951069241510130465691156908893803280e-1),
 543:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.84647597299970149588010858770320631739e-2),
 544:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.90239396588176334117512714878489376365e-3),
 545:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.35551585337774834346900776840459179841e-5),
 546:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.53375746264539501168763602838029023222e-6),
 547:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.42421935941736734247914078641324315900e-8),
 548:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.23835501607741697737129504173606231513e-10),
 549:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.79603272375172813955236187874231935324e-12),
 550:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.44624821303153251954931367754173356213e-14),
 551:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.10635081308984534416704147448323126303e-16),
 552:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.14627867347129520510628554651739571006e-19),
 553:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43792928765659831045040802615903432044e-21),
 554:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.79856365207259871336606847582889916798e-25),
 555:         };
 556:         // LCOV_EXCL_STOP
 557:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 558:     }
 559:     else {
 560:         RealType t = 1 / sqrt(x * x * x);
 561: 
 562:         // Rational Approximation
 563:         // Maximum Relative Error: 3.5081e-36
 564:         // LCOV_EXCL_START
 565:         BOOST_MATH_STATIC const RealType P[8] = {
 566:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.98413420602149016909919089901572802714e-1),
 567:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.30303835860684077803651094768293625633e-1),
 568:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.89097726237252419724261295392691855545e1),
 569:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.12696604472230480273239741428914666511e1),
 570:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.84517621403071494824886152940942995151e3),
 571:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.67577378292168927009421205756730205227e2),
 572:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.16343347002845084264982358165052437094e3),
 573:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.59558963351172885545760841064831356701e3),
 574:         };
 575:         BOOST_MATH_STATIC const RealType Q[7] = {
 576:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 577-624 / 第 577-624 行
~~~cpp
 577:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.51965956124978480521462518750569617550e-1),
 578:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.61700833299761977287211297600922591853e2),
 579:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.94988298508869748383898344668918510537e1),
 580:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.52494213749069142804725453333400335525e3),
 581:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.20093079283917759611690534481918040882e3),
 582:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.82564796242972192725215815897475246715e4),
 583:         };
 584:         // LCOV_EXCL_STOP
 585:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t / x;
 586:     }
 587: 
 588:     return result;
 589: }
 590: 
 591: template <class RealType>
 592: BOOST_MATH_GPU_ENABLED inline RealType mapairy_pdf_minus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53>&)
 593: {
 594:     BOOST_MATH_STD_USING
 595:     RealType result;
 596: 
 597:     if (x >= -1) {
 598:         RealType t = x + 1;
 599: 
 600:         // Rational Approximation
 601:         // Maximum Relative Error: 3.7525e-17
 602:         BOOST_MATH_STATIC const RealType P[8] = {
 603:             static_cast<RealType>(2.76859868856746781256e-1),
 604:             static_cast<RealType>(1.10489814676299003241e-1),
 605:             static_cast<RealType>(-6.25690643488236678667e-3),
 606:             static_cast<RealType>(-1.17905420222527577236e-3),
 607:             static_cast<RealType>(1.27188963720084274122e-3),
 608:             static_cast<RealType>(-7.20575105181207907889e-5),
 609:             static_cast<RealType>(-2.22575633858411851032e-5),
 610:             static_cast<RealType>(2.94270091008508492304e-6),
 611:         };
 612:         BOOST_MATH_STATIC const RealType Q[7] = {
 613:             static_cast<RealType>(1.),
 614:             static_cast<RealType>(4.98673671503410894284e-1),
 615:             static_cast<RealType>(3.15907666864554716291e-1),
 616:             static_cast<RealType>(8.34463558393629855977e-2),
 617:             static_cast<RealType>(2.71804643993972494173e-2),
 618:             static_cast<RealType>(3.52187050938036578406e-3),
 619:             static_cast<RealType>(7.03072974279509263844e-4),
 620:         };
 621: 
 622:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 623:     }
 624:     else if (x >= -2) {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, if.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。

### Lines 625-672 / 第 625-672 行
~~~cpp
 625:         RealType t = x + 2;
 626: 
 627:         // Rational Approximation
 628:         // Maximum Relative Error: 4.0995e-17
 629:         BOOST_MATH_STATIC const RealType P[8] = {
 630:             static_cast<RealType>(2.14483832832989822788e-1),
 631:             static_cast<RealType>(3.72789690317712876663e-1),
 632:             static_cast<RealType>(1.86473650057086284496e-1),
 633:             static_cast<RealType>(1.31182724166379598907e-2),
 634:             static_cast<RealType>(-9.00695064809774432392e-3),
 635:             static_cast<RealType>(3.46884420664996747052e-4),
 636:             static_cast<RealType>(4.88651392754189961173e-4),
 637:             static_cast<RealType>(-6.13516242712196835055e-5),
 638:         };
 639:         BOOST_MATH_STATIC const RealType Q[8] = {
 640:             static_cast<RealType>(1.),
 641:             static_cast<RealType>(1.06478618107122200489e0),
 642:             static_cast<RealType>(4.08809060854459518663e-1),
 643:             static_cast<RealType>(2.66617598099501800866e-1),
 644:             static_cast<RealType>(4.53526315786051807494e-2),
 645:             static_cast<RealType>(2.44078693689626940834e-2),
 646:             static_cast<RealType>(1.52822572478697831870e-3),
 647:             static_cast<RealType>(8.69480001029742502197e-4),
 648:         };
 649: 
 650:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 651:     }
 652:     else {
 653:         RealType s = exp(2 * x * x * x / 27) * sqrt(-x);
 654: 
 655:         if (x >= -4) {
 656:             RealType t = -x - 2;
 657: 
 658:             // Rational Approximation
 659:             // Maximum Relative Error: 2.4768e-18
 660:             BOOST_MATH_STATIC const RealType P[8] = {
 661:                 static_cast<RealType>(2.74308494787955998605e-1),
 662:                 static_cast<RealType>(4.87765991440983416392e-1),
 663:                 static_cast<RealType>(3.84524365110270427617e-1),
 664:                 static_cast<RealType>(1.77409497505926097339e-1),
 665:                 static_cast<RealType>(5.25612864287310961520e-2),
 666:                 static_cast<RealType>(1.01528615034079765421e-2),
 667:                 static_cast<RealType>(1.20417225696161842090e-3),
 668:                 static_cast<RealType>(6.97462693097107007719e-5),
 669:             };
 670:             BOOST_MATH_STATIC const RealType Q[8] = {
 671:                 static_cast<RealType>(1.),
 672:                 static_cast<RealType>(1.81256903248465876424e0),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, exp. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, exp。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 673-720 / 第 673-720 行
~~~cpp
 673:                 static_cast<RealType>(1.43959302060852067876e0),
 674:                 static_cast<RealType>(6.65882284117861804351e-1),
 675:                 static_cast<RealType>(1.97537712781845593211e-1),
 676:                 static_cast<RealType>(3.81732970028510912201e-2),
 677:                 static_cast<RealType>(4.52767489928026542226e-3),
 678:                 static_cast<RealType>(2.62240194911920120003e-4),
 679:             };
 680: 
 681:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 682:         }
 683:         else if (x >= -8) {
 684:             RealType t = -x - 4;
 685: 
 686:             // Rational Approximation
 687:             // Maximum Relative Error: 1.5741e-17
 688:             BOOST_MATH_STATIC const RealType P[8] = {
 689:                 static_cast<RealType>(2.67391547707456587286e-1),
 690:                 static_cast<RealType>(3.39319035621314371924e-1),
 691:                 static_cast<RealType>(1.85434799940724207230e-1),
 692:                 static_cast<RealType>(5.63667456320679857693e-2),
 693:                 static_cast<RealType>(1.01231164548944177474e-2),
 694:                 static_cast<RealType>(1.02501575174439362864e-3),
 695:                 static_cast<RealType>(4.60769537123286016400e-5),
 696:                 static_cast<RealType>(-4.92754650783224582641e-13),
 697:             };
 698:             BOOST_MATH_STATIC const RealType Q[7] = {
 699:                 static_cast<RealType>(1.),
 700:                 static_cast<RealType>(1.27271216837333318516e0),
 701:                 static_cast<RealType>(6.96551952883867277759e-1),
 702:                 static_cast<RealType>(2.11871363524516350422e-1),
 703:                 static_cast<RealType>(3.80622887806509632537e-2),
 704:                 static_cast<RealType>(3.85400280812991562328e-3),
 705:                 static_cast<RealType>(1.73246593953823694311e-4),
 706:             };
 707: 
 708:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 709:         }
 710:         else if (x >= -16) {
 711:             RealType t = -x - 8;
 712: 
 713:             // Rational Approximation
 714:             // Maximum Relative Error: 4.6579e-17
 715:             BOOST_MATH_STATIC const RealType P[6] = {
 716:                 static_cast<RealType>(2.66153901932100301337e-1),
 717:                 static_cast<RealType>(1.65767350677458230714e-1),
 718:                 static_cast<RealType>(4.19801402197670061146e-2),
 719:                 static_cast<RealType>(5.39337995172784579558e-3),
 720:                 static_cast<RealType>(3.50811247702301287586e-4),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 721-768 / 第 721-768 行
~~~cpp
 721:                 static_cast<RealType>(9.21758454778883157515e-6),
 722:             };
 723:             BOOST_MATH_STATIC const RealType Q[6] = {
 724:                 static_cast<RealType>(1.),
 725:                 static_cast<RealType>(6.23092941554668369107e-1),
 726:                 static_cast<RealType>(1.57829914506366827914e-1),
 727:                 static_cast<RealType>(2.02787979758160988615e-2),
 728:                 static_cast<RealType>(1.31903008994475216511e-3),
 729:                 static_cast<RealType>(3.46575870637847438219e-5),
 730:             };
 731: 
 732:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 733:         }
 734:         else if (x >= -32) {
 735:             RealType t = -x - 16;
 736: 
 737:             // Rational Approximation
 738:             // Maximum Relative Error: 5.2014e-17
 739:             BOOST_MATH_STATIC const RealType P[5] = {
 740:                 static_cast<RealType>(2.65985830928929730672e-1),
 741:                 static_cast<RealType>(7.19655029633308583205e-2),
 742:                 static_cast<RealType>(7.26293125679558421946e-3),
 743:                 static_cast<RealType>(3.24276402295343802262e-4),
 744:                 static_cast<RealType>(5.40508013573989841127e-6),
 745:             };
 746:             BOOST_MATH_STATIC const RealType Q[5] = {
 747:                 static_cast<RealType>(1.),
 748:                 static_cast<RealType>(2.70578525590448009961e-1),
 749:                 static_cast<RealType>(2.73082032706004833847e-2),
 750:                 static_cast<RealType>(1.21926059813954504560e-3),
 751:                 static_cast<RealType>(2.03227900426552177849e-5),
 752:             };
 753: 
 754:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 755:         }
 756:         else {
 757:             result = 0;
 758:         }
 759:     }
 760: 
 761:     return result;
 762: }
 763: 
 764: template <class RealType>
 765: BOOST_MATH_GPU_ENABLED inline RealType mapairy_pdf_minus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>&)
 766: {
 767:     BOOST_MATH_STD_USING
 768:     RealType result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, if.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。

### Lines 769-816 / 第 769-816 行
~~~cpp
 769: 
 770:     if (x >= -1) {
 771:         RealType t = x + 1;
 772: 
 773:         // Rational Approximation
 774:         // Maximum Relative Error: 5.2870e-35
 775:         // LCOV_EXCL_START
 776:         BOOST_MATH_STATIC const RealType P[14] = {
 777:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76859868856746781256050397658493368372e-1),
 778:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.13037642242224438972685982606987140111e-1),
 779:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.93206268361082760254653961897373271146e-2),
 780:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12844418906916902333116398594921450782e-2),
 781:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.36889326770180267250286619759335338794e-3),
 782:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.95272615884641416804001553871108995422e-4),
 783:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.53808638264746233799776679481568171506e-5),
 784:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.92177790427881393122479399837010657693e-6),
 785:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.93492737815019893169693306410980499366e-6),
 786:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.87510085148730083683110532987841223544e-8),
 787:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.28469424017979299382094276157986775969e-9),
 788:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.83693904015623816528442886551032709693e-9),
 789:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.77632857558257155545506847333166147492e-10),
 790:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00448215148716947837105979735199471601e-11),
 791:         };
 792:         BOOST_MATH_STATIC const RealType Q[13] = {
 793:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 794:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.69069814466926608209872727645156315374e-1),
 795:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.89657828158127300370734997707096744077e-1),
 796:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.62713433978940724622996782534485162816e-1),
 797:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.91600878366366974062522408704458777166e-2),
 798:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.89144035500328704769924414014440238441e-2),
 799:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.35263616916053275381069097012458200491e-3),
 800:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49136684724986851824746531490006769036e-3),
 801:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.65912003138912073317982729161392623277e-4),
 802:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.65931144405541620572732754508534372034e-5),
 803:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.40193555853535182510951061797573338442e-6),
 804:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.43625211359756249232841566256877823039e-7),
 805:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.33207781577559817130740123609636060998e-8),
 806:         };
 807:         // LCOV_EXCL_STOP
 808:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 809:     }
 810:     else if (x >= -2) {
 811:         RealType t = x + 2;
 812: 
 813:         // Rational Approximation
 814:         // Maximum Relative Error: 1.1977e-35
 815:         // LCOV_EXCL_START
 816:         BOOST_MATH_STATIC const RealType P[13] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 817-864 / 第 817-864 行
~~~cpp
 817:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.14483832832989822788477500521594411868e-1),
 818:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.75657192307644021285091474845448102656e-1),
 819:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40437358470633234235031852091608646844e-1),
 820:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.66609942512054705023295445270747546208e-2),
 821:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.54563774151184610728476161049657676321e-3),
 822:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.51479057544157089574005315379453615537e-3),
 823:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.59853789372610909788599341307719626846e-4),
 824:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.76919062715142378209907670793921883406e-5),
 825:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.58572738466179822770103948740437237476e-6),
 826:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.66618046393835590932491510543557226290e-7),
 827:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.26253044828460469263564567571249315188e-8),
 828:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11130363073235247786909976446790746902e-9),
 829:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.49023728251751416730708805268921994420e-10),
 830:         };
 831:         BOOST_MATH_STATIC const RealType Q[15] = {
 832:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 833:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.11919889346080886194925406930280687022e-1),
 834:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.99082771425048574611745923487528183522e-1),
 835:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.99525320878512488641033584061027063035e-1),
 836:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.20775109959302182467696345673111724657e-2),
 837:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.67505804311611026128557007926613964162e-2),
 838:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.77854913919309273628222660024596583623e-3),
 839:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.91661599559554233157812211199256222756e-3),
 840:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.83924945472605861063053622956144354568e-4),
 841:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.84286353909650034923710426843028632590e-5),
 842:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.57737060659799463556626420070111210218e-5),
 843:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.76047305116625604109657617040360402976e-6),
 844:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.86975509621224474718728318687795215895e-7),
 845:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71646204381423826495116781730719271111e-8),
 846:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30359141441663007574346497273327240071e-9),
 847:         };
 848:         // LCOV_EXCL_STOP
 849:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 850:     }
 851:     else {
 852:         RealType s = exp(2 * x * x * x / 27) * sqrt(-x);
 853: 
 854:         if (x >= -4) {
 855:             RealType t = -x - 2;
 856: 
 857:             // Rational Approximation
 858:             // Maximum Relative Error: 5.4547e-35
 859:             // LCOV_EXCL_START
 860:             BOOST_MATH_STATIC const RealType P[15] = {
 861:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.74308494787955998605105974174143750745e-1),
 862:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.56767876568276519015214709629156760546e-1),
 863:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23402577465454790961498400214198520261e0),
 864:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09577559351834952074671208183548972395e0),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, exp. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, exp。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 865-912 / 第 865-912 行
~~~cpp
 865:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.76209118910349927892265971592071407626e-1),
 866:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.09368637728788364895148841703533651597e-1),
 867:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09003822946777310058789032386408519829e-1),
 868:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.02362804210869367995322279203786166303e-2),
 869:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.67210045349462046966360849113168808620e-3),
 870:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17170437120510484976042000272825166724e-3),
 871:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62068279517157268391045945672600042900e-4),
 872:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.72238125522303876741011786930129571553e-5),
 873:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33906175951716762094473406744654874848e-6),
 874:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.88118741063309731598638313174835288433e-8),
 875:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.78908322579081615215057968216358892954e-9),
 876:             };
 877:             BOOST_MATH_STATIC const RealType Q[15] = {
 878:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 879:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.15777668058369565739250784347385217839e0),
 880:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.58275582332060589146223977924181161908e0),
 881:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.08890987062755381429904193744273374370e0),
 882:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.53062680969750921573862970262146744660e0),
 883:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15983695707064161504470525373678920004e0),
 884:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.09120624447001177857109399158887656977e-1),
 885:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.13566107440776375294261717406754395407e-1),
 886:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.50716565210262652091950832287627406780e-2),
 887:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.40417354541359829249609883808591989082e-3),
 888:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.09285589734746898623782466689035549135e-4),
 889:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.47580156629757526370271002425784456931e-5),
 890:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.03479533688660179064728081632921439825e-6),
 891:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.58728676819719406366664644282113323077e-7),
 892:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.72685000369623096389026353785111272994e-9),
 893:             };
 894:             // LCOV_EXCL_STOP
 895:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 896:         }
 897:         else if (x >= -8) {
 898:             RealType t = -x - 4;
 899: 
 900:             // Rational Approximation
 901:             // Maximum Relative Error: 1.8813e-35
 902:             // LCOV_EXCL_START
 903:             BOOST_MATH_STATIC const RealType P[15] = {
 904:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.67391547707456587286086623414017962238e-1),
 905:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.69944730920904699720804320295067934914e-1),
 906:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.80384452804523880914883464295008532437e-1),
 907:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.74832028145199140240423863864148009059e-1),
 908:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71728522451977382202061046054643165624e-1),
 909:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.91023495084678296967637417245526177858e-2),
 910:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.57730498044529764612538979048001166775e-2),
 911:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.31940820074475947691746555183209863058e-3),
 912:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.54175805821840981842505041345112198286e-4),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 913-960 / 第 913-960 行
~~~cpp
 913:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.31350452337838677820161124238784043790e-5),
 914:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.52175993144502511705213771924810467309e-6),
 915:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.85684239411667243910736588216628677445e-7),
 916:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.27124210379062272403030391492854565008e-8),
 917:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17645475312219452046348851569796494059e-9),
 918:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06306499345515479193219487228315566344e-11),
 919:             };
 920:             BOOST_MATH_STATIC const RealType Q[15] = {
 921:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 922:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.13521398369589479131299586715604029947e0),
 923:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.17680254721938920978999949995837883884e0),
 924:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40693619288419980101309080614788657638e0),
 925:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.44930162913500531579305526795523256972e-1),
 926:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.22044272115074113804712893993125987243e-1),
 927:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.92745159832354238503828226333417152767e-2),
 928:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24766164774700476810039401793119553409e-2),
 929:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.08325637569571782180723187639357833929e-3),
 930:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.74954547353553788519997212700557196088e-4),
 931:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.82800744682204649977844278025855329390e-5),
 932:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.20210992299988298543034791887173754015e-6),
 933:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.22996819257926038785424888617824130286e-7),
 934:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.42340212199922656577943251139931264313e-9),
 935:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.75700556749505163188370496864513941614e-11),
 936:             };
 937:             // LCOV_EXCL_STOP
 938:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 939:         }
 940:         else if (x >= -16) {
 941:             RealType t = -x - 8;
 942: 
 943:             // Rational Approximation
 944:             // Maximum Relative Error: 3.7501e-35
 945:             // LCOV_EXCL_START
 946:             BOOST_MATH_STATIC const RealType P[13] = {
 947:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.66153901932100301337118653561328446399e-1),
 948:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.52542079386371212946566450189144670788e-1),
 949:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.17560936304516198261138159102435548430e-1),
 950:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.26792904240601626330507068992045446428e-2),
 951:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.15418212265160879313643948347460896640e-2),
 952:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.05247220687656529725024232836519908641e-3),
 953:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.64228534097787946289779529645800775231e-4),
 954:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.85634097697132464418223150629017524118e-5),
 955:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.49585420710073223183176764488210189671e-6),
 956:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.48871040740917898530270248991342594461e-7),
 957:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.42577266655992039477272273926475476183e-9),
 958:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19214263302271253341410568192952269518e-10),
 959:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.36635313919771528255819112450043338510e-12),
 960:             };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 961-1008 / 第 961-1008 行
~~~cpp
 961:             BOOST_MATH_STATIC const RealType Q[13] = {
 962:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
 963:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.32484755553196872705775494679365596205e0),
 964:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.17714315014480774542066462899317631393e-1),
 965:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.10789882607024692577764888497624620277e-1),
 966:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.09821963157449764169644456445120769215e-2),
 967:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52354198870000121894280965999352991441e-2),
 968:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.12133327236256081067100384182795121111e-3),
 969:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.20187894923874357333806454001674518211e-4),
 970:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.69039238999927049119096278882765161803e-5),
 971:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.35737444680219098802811205475695127060e-7),
 972:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.54403624143647064402264521374546365073e-8),
 973:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.24233005893817070145949404296998119469e-10),
 974:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.89735152971223120087721392400123727326e-12),
 975:             };
 976:             // LCOV_EXCL_STOP
 977:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
 978:         }
 979:         else if (x >= -32) {
 980:             RealType t = -x - 16;
 981: 
 982:             // Rational Approximation
 983:             // Maximum Relative Error: 9.2696e-36
 984:             // LCOV_EXCL_START
 985:             BOOST_MATH_STATIC const RealType P[12] = {
 986:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.65985830928929730672052407058361701971e-1),
 987:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.80409998734303497641108024806388734755e-1),
 988:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.49286120625421787109350223436127409819e-2),
 989:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.89160491404149422833016337592047445082e-3),
 990:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16725811789351893632348469796802834008e-3),
 991:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.43438517439595919021069131504449842238e-5),
 992:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.29058184637190638359623120253986595623e-6),
 993:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.03288592271246432030980385908922413497e-7),
 994:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.12286831076824535034975676306286388291e-9),
 995:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.64563161552001551475186730009447111173e-11),
 996:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.13183856815615371136129883169639301710e-13),
 997:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.02405342795439598418033139109649640085e-35),
 998:             };
 999:             BOOST_MATH_STATIC const RealType Q[11] = {
1000:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1001:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.78286317363568496229516074305435186276e-1),
1002:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06519013547074134846431611115576250187e-1),
1003:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.71907733798006110542919988654989891098e-2),
1004:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.38874744033460851257697736304200953873e-3),
1005:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.54724289412996188575775800547576856966e-4),
1006:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.98922099980447626797646560786207812928e-5),
1007:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.64352676367403443733555974471752023206e-7),
1008:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.92616898324742524009679754162620171773e-8),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1009-1056 / 第 1009-1056 行
~~~cpp
1009:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.87471345773127482399498877510153906820e-10),
1010:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.92954174836731254818376396170511443820e-12),
1011:             };
1012:             // LCOV_EXCL_STOP
1013:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1014:         }
1015:         else if (x >= -64) {
1016:             RealType t = -x - 32;
1017: 
1018:             // Rational Approximation
1019:             // Maximum Relative Error: 2.3524e-35
1020:             // LCOV_EXCL_START
1021:             BOOST_MATH_STATIC const RealType P[10] = {
1022:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.65964563346442080104568381680822923977e-1),
1023:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.77958685324702990033291591478515962894e-2),
1024:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.56419338083136866686699803771820491401e-3),
1025:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.82465178504003399087279098324316458608e-4),
1026:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.92402911374159755476910533154145918079e-5),
1027:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.91224450962405933321548581824712789516e-7),
1028:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.84063939469145970625490205194192347630e-9),
1029:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.15300528698702940691774461674788639801e-11),
1030:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.85553643603397817535280932672322232325e-13),
1031:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.46207029637607033398822620480584537642e-38),
1032:             };
1033:             BOOST_MATH_STATIC const RealType Q[9] = {
1034:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1035:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.54906717312241693103173902792310528801e-1),
1036:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.84408124581401290943345932332007045483e-2),
1037:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.81403744024723164669745491417804917709e-3),
1038:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.23423244618880845765135047598258754409e-5),
1039:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.84697524433421334697753031272973192290e-6),
1040:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.94803525968789587050040294764458613062e-8),
1041:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.68948879514200831687856703804327184420e-10),
1042:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07366525547027105672618224029122809899e-12),
1043:             };
1044:             // LCOV_EXCL_STOP
1045:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1046:         }
1047:         else {
1048:             result = 0;
1049:         }
1050:     }
1051: 
1052:     return result;
1053: }
1054: 
1055: template <class RealType>
1056: BOOST_MATH_GPU_ENABLED inline RealType mapairy_pdf_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53> &tag) {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, if, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。

### Lines 1057-1104 / 第 1057-1104 行
~~~cpp
1057:     if (x >= 0) {
1058:         return mapairy_pdf_plus_imp_prec<RealType>(x, tag);
1059:     }
1060:     else if (x <= 0) {
1061:         return mapairy_pdf_minus_imp_prec<RealType>(x, tag);
1062:     }
1063:     else {
1064:         return boost::math::numeric_limits<RealType>::quiet_NaN();
1065:     }
1066: }
1067: 
1068: template <class RealType>
1069: BOOST_MATH_GPU_ENABLED inline RealType mapairy_pdf_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>& tag) {
1070:     if (x >= 0) {
1071:         return mapairy_pdf_plus_imp_prec<RealType>(x, tag);
1072:     }
1073:     else if (x <= 0) {
1074:         return mapairy_pdf_minus_imp_prec<RealType>(x, tag);
1075:     }
1076:     else {
1077:         return boost::math::numeric_limits<RealType>::quiet_NaN();
1078:     }
1079: }
1080: 
1081: template <class RealType, class Policy>
1082: BOOST_MATH_GPU_ENABLED inline RealType mapairy_pdf_imp(const mapairy_distribution<RealType, Policy>& dist, const RealType& x) {
1083:     //
1084:     // This calculates the pdf of the Map-Airy distribution and/or its complement.
1085:     //
1086: 
1087:     BOOST_MATH_STD_USING // for ADL of std functions
1088:     constexpr auto function = "boost::math::pdf(mapairy<%1%>&, %1%)";
1089:     RealType result = 0;
1090:     RealType location = dist.location();
1091:     RealType scale = dist.scale();
1092: 
1093:     if (false == detail::check_location(function, location, &result, Policy()))
1094:     {
1095:         return result;
1096:     }
1097:     if (false == detail::check_scale(function, scale, &result, Policy()))
1098:     {
1099:         return result;
1100:     }
1101:     if (false == detail::check_x(function, x, &result, Policy()))
1102:     {
1103:         return result;
1104:     }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as if, mapairy_pdf_imp_prec, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 if, mapairy_pdf_imp_prec, ...。

### Lines 1105-1152 / 第 1105-1152 行
~~~cpp
1105: 
1106:     typedef typename tools::promote_args<RealType>::type result_type;
1107:     typedef typename policies::precision<result_type, Policy>::type precision_type;
1108:     typedef boost::math::integral_constant<int,
1109:         precision_type::value <= 0 ? 0 :
1110:         precision_type::value <= 53 ? 53 :
1111:         precision_type::value <= 113 ? 113 : 0
1112:     > tag_type;
1113: 
1114:     static_assert(tag_type::value, "The Map-Airy distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
1115: 
1116:     RealType u = (x - location) / scale;
1117: 
1118:     result = mapairy_pdf_imp_prec(u, tag_type()) / scale;
1119: 
1120:     return result;
1121: }
1122: 
1123: template <class RealType>
1124: BOOST_MATH_GPU_ENABLED inline RealType mapairy_cdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53>&)
1125: {
1126:     BOOST_MATH_STD_USING
1127:     RealType result;
1128: 
1129:     if (x < 1) {
1130:         // Rational Approximation
1131:         // Maximum Relative Error: 2.9194e-17
1132:         BOOST_MATH_STATIC const RealType P[7] = {
1133:             static_cast<RealType>(3.33333333333333333333e-1),
1134:             static_cast<RealType>(7.49532137610545010591e-2),
1135:             static_cast<RealType>(9.25326921848155048716e-3),
1136:             static_cast<RealType>(6.59133092365796208900e-3),
1137:             static_cast<RealType>(-5.21942678326323374113e-4),
1138:             static_cast<RealType>(8.22766804917461941348e-5),
1139:             static_cast<RealType>(-3.97941251650023182117e-6),
1140:         };
1141:         BOOST_MATH_STATIC const RealType Q[7] = {
1142:             static_cast<RealType>(1.),
1143:             static_cast<RealType>(8.17408156824742736411e-1),
1144:             static_cast<RealType>(3.57041011418415988268e-1),
1145:             static_cast<RealType>(1.04580353775369716002e-1),
1146:             static_cast<RealType>(1.87521616934129432292e-2),
1147:             static_cast<RealType>(2.33232161135637085535e-3),
1148:             static_cast<RealType>(7.31285352607895467310e-5),
1149:         };
1150: 
1151:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
1152:     }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, tools::evaluate_polynomial.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, tools::evaluate_polynomial。

### Lines 1153-1200 / 第 1153-1200 行
~~~cpp
1153:     else if (x < 2) {
1154:         RealType t = x - 1;
1155: 
1156:         // Rational Approximation
1157:         // Maximum Relative Error: 3.1531e-17
1158:         BOOST_MATH_STATIC const RealType P[7] = {
1159:             static_cast<RealType>(1.84196970581015939888e-1),
1160:             static_cast<RealType>(-1.19398028299089933853e-3),
1161:             static_cast<RealType>(1.21954054797949597854e-2),
1162:             static_cast<RealType>(-9.37912675685073154845e-4),
1163:             static_cast<RealType>(1.66651954077980453212e-4),
1164:             static_cast<RealType>(-1.33271812303025233648e-5),
1165:             static_cast<RealType>(5.35982226125013888796e-7),
1166:         };
1167:         BOOST_MATH_STATIC const RealType Q[6] = {
1168:             static_cast<RealType>(1.),
1169:             static_cast<RealType>(5.70352826101668448273e-1),
1170:             static_cast<RealType>(1.98852010141232271304e-1),
1171:             static_cast<RealType>(3.64864882318453496161e-2),
1172:             static_cast<RealType>(4.22173125405065522298e-3),
1173:             static_cast<RealType>(1.20079284386796600356e-4),
1174:         };
1175: 
1176:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1177:     }
1178:     else if (x < 4) {
1179:         RealType t = x - 2;
1180: 
1181:         // Rational Approximation
1182:         // Maximum Relative Error: 1.8348e-17
1183:         BOOST_MATH_STATIC const RealType P[8] = {
1184:             static_cast<RealType>(1.07409273397524124098e-1),
1185:             static_cast<RealType>(3.83900318969331880402e-2),
1186:             static_cast<RealType>(1.17926652359826576790e-2),
1187:             static_cast<RealType>(1.52181625871479030046e-3),
1188:             static_cast<RealType>(1.50703424417132565662e-4),
1189:             static_cast<RealType>(2.10117959279448106308e-6),
1190:             static_cast<RealType>(1.97360985832285866640e-8),
1191:             static_cast<RealType>(-1.06076300080048408251e-9),
1192:         };
1193:         BOOST_MATH_STATIC const RealType Q[7] = {
1194:             static_cast<RealType>(1.),
1195:             static_cast<RealType>(8.54435380513870673497e-1),
1196:             static_cast<RealType>(3.66021233157880878411e-1),
1197:             static_cast<RealType>(9.42985570806905160687e-2),
1198:             static_cast<RealType>(1.54122343653998564507e-2),
1199:             static_cast<RealType>(1.49849056258932455548e-3),
1200:             static_cast<RealType>(6.94290406268856211707e-5),
~~~
- **EN:** This range declares or defines callable logic such as if, tools::evaluate_polynomial. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 if, tools::evaluate_polynomial。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1201-1248 / 第 1201-1248 行
~~~cpp
1201:         };
1202: 
1203:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1204:     }
1205:     else if (x < 8) {
1206:         RealType t = x - 4;
1207: 
1208:         // Rational Approximation
1209:         // Maximum Relative Error: 2.6624e-18
1210:         BOOST_MATH_STATIC const RealType P[8] = {
1211:             static_cast<RealType>(4.70720199535228802538e-2),
1212:             static_cast<RealType>(2.67200763833749070079e-2),
1213:             static_cast<RealType>(7.37400551855064729769e-3),
1214:             static_cast<RealType>(1.10592441765001623699e-3),
1215:             static_cast<RealType>(9.15846028547400212588e-5),
1216:             static_cast<RealType>(3.17801522553862136789e-6),
1217:             static_cast<RealType>(2.03102753319827713542e-8),
1218:             static_cast<RealType>(-5.16172854149066643529e-11),
1219:         };
1220:         BOOST_MATH_STATIC const RealType Q[8] = {
1221:             static_cast<RealType>(1.),
1222:             static_cast<RealType>(9.05317644829451086870e-1),
1223:             static_cast<RealType>(3.73713496637025562492e-1),
1224:             static_cast<RealType>(8.94434672792094976627e-2),
1225:             static_cast<RealType>(1.31846542255347106087e-2),
1226:             static_cast<RealType>(1.16680596342421447100e-3),
1227:             static_cast<RealType>(5.44719256441278863300e-5),
1228:             static_cast<RealType>(8.73131209154185067287e-7),
1229:         };
1230: 
1231:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1232:     }
1233:     else if (x < 16) {
1234:         RealType t = x - 8;
1235: 
1236:         // Rational Approximation
1237:         // Maximum Relative Error: 2.6243e-17
1238:         BOOST_MATH_STATIC const RealType P[8] = {
1239:             static_cast<RealType>(1.74847564444513000450e-2),
1240:             static_cast<RealType>(6.00209162595027323742e-3),
1241:             static_cast<RealType>(7.86550260761375576075e-4),
1242:             static_cast<RealType>(4.46682547335758521734e-5),
1243:             static_cast<RealType>(9.51329761417139273391e-7),
1244:             static_cast<RealType>(4.10313065114362712333e-9),
1245:             static_cast<RealType>(-9.81286503831545640189e-12),
1246:             static_cast<RealType>(2.98763969872672156104e-14),
1247:         };
1248:         BOOST_MATH_STATIC const RealType Q[7] = {
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1249-1296 / 第 1249-1296 行
~~~cpp
1249:             static_cast<RealType>(1.),
1250:             static_cast<RealType>(5.27732094554221674504e-1),
1251:             static_cast<RealType>(1.14330643482604301178e-1),
1252:             static_cast<RealType>(1.27722341942374066265e-2),
1253:             static_cast<RealType>(7.54563340152441778517e-4),
1254:             static_cast<RealType>(2.13377039814057925832e-5),
1255:             static_cast<RealType>(2.09670987094350618690e-7),
1256:         };
1257: 
1258:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1259:     }
1260:     else if (x < 32) {
1261:         RealType t = x - 16;
1262: 
1263:         // Rational Approximation
1264:         // Maximum Relative Error: 5.4684e-17
1265:         BOOST_MATH_STATIC const RealType P[8] = {
1266:             static_cast<RealType>(6.22684103170563193015e-3),
1267:             static_cast<RealType>(1.34714356588780958096e-3),
1268:             static_cast<RealType>(9.51289465377874891896e-5),
1269:             static_cast<RealType>(2.64918464474843134081e-6),
1270:             static_cast<RealType>(2.66703857491046795285e-8),
1271:             static_cast<RealType>(5.42037888457985833156e-11),
1272:             static_cast<RealType>(-6.18017115447736427379e-14),
1273:             static_cast<RealType>(9.11626234402148561268e-17),
1274:         };
1275:         BOOST_MATH_STATIC const RealType Q[7] = {
1276:             static_cast<RealType>(1.),
1277:             static_cast<RealType>(3.09895694991285975774e-1),
1278:             static_cast<RealType>(3.69874670435930773471e-2),
1279:             static_cast<RealType>(2.15708854325146400153e-3),
1280:             static_cast<RealType>(6.35345408451056881884e-5),
1281:             static_cast<RealType>(8.65722805575670770555e-7),
1282:             static_cast<RealType>(4.03153189557220023202e-9),
1283:         };
1284: 
1285:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1286:     }
1287:     else if (x < 64) {
1288:         RealType t = x - 32;
1289: 
1290:         // Rational Approximation
1291:         // Maximum Relative Error: 6.5947e-17
1292:         BOOST_MATH_STATIC const RealType P[7] = {
1293:             static_cast<RealType>(2.20357145727036120652e-3),
1294:             static_cast<RealType>(1.45412555771401325111e-4),
1295:             static_cast<RealType>(3.27819006009093198652e-6),
1296:             static_cast<RealType>(2.96786786716623870006e-8),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1297-1344 / 第 1297-1344 行
~~~cpp
1297:             static_cast<RealType>(9.54192199129339742308e-11),
1298:             static_cast<RealType>(5.71421706870777687254e-14),
1299:             static_cast<RealType>(-1.48321866072033823195e-17),
1300:         };
1301:         BOOST_MATH_STATIC const RealType Q[7] = {
1302:             static_cast<RealType>(1.),
1303:             static_cast<RealType>(1.12851983233980279746e-1),
1304:             static_cast<RealType>(4.94650928817638043712e-3),
1305:             static_cast<RealType>(1.05447405092956497114e-4),
1306:             static_cast<RealType>(1.11578464291338271178e-6),
1307:             static_cast<RealType>(5.27522295397347842625e-9),
1308:             static_cast<RealType>(7.95786524903707645399e-12),
1309:         };
1310: 
1311:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1312:     }
1313:     else {
1314:         RealType x_cube = x * x * x;
1315:         RealType t = static_cast<RealType>((boost::math::isnormal)(x_cube) ? 1 / sqrt(x_cube) : 1 / pow(sqrt(x), 3));
1316: 
1317:         // Rational Approximation
1318:         // Maximum Relative Error: 6.2709e-17
1319:         BOOST_MATH_STATIC const RealType P[4] = {
1320:             static_cast<RealType>(3.98942280401432677940e-1),
1321:             static_cast<RealType>(2.89752186412133782995e-2),
1322:             static_cast<RealType>(4.67360459917040710474e0),
1323:             static_cast<RealType>(-1.26770824563800250704e-1),
1324:         };
1325:         BOOST_MATH_STATIC const RealType Q[3] = {
1326:             static_cast<RealType>(1.),
1327:             static_cast<RealType>(7.26301023103568827709e-2),
1328:             static_cast<RealType>(1.60899894281099149848e1),
1329:         };
1330: 
1331:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t;
1332:     }
1333: 
1334:     return result;
1335: }
1336: 
1337: 
1338: template <class RealType>
1339: BOOST_MATH_GPU_ENABLED inline RealType mapairy_cdf_plus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>&)
1340: {
1341:     BOOST_MATH_STD_USING
1342:     RealType result;
1343: 
1344:     if (x < 1) {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, sqrt.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, sqrt。

### Lines 1345-1392 / 第 1345-1392 行
~~~cpp
1345:         // Rational Approximation
1346:         // Maximum Relative Error: 4.7720e-37
1347:         // LCOV_EXCL_START
1348:         BOOST_MATH_STATIC const RealType P[14] = {
1349:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.33333333333333333333333333333333333333e-1),
1350:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.38519736580901276671338330967060054188e-1),
1351:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.07012342772403725079487012557507575976e-2),
1352:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70163612228825567572185033570526547856e-2),
1353:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.16393313438726572630782132625753922397e-3),
1354:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.92141312947853945617138019222992750592e-4),
1355:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16513062047959961711747864068554379374e-4),
1356:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.08850391017085844154857927364247623649e-5),
1357:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07060872491334153829857156707699441084e-6),
1358:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.56961733740920438026573722084839596926e-8),
1359:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.93626747947476815631021107726714283086e-10),
1360:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.32967164823609209711923411113824666288e-10),
1361:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.23420723211833268177898025846064230665e-12),
1362:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.13807083548358335699029971528179486964e-13),
1363:         };
1364:         BOOST_MATH_STATIC const RealType Q[12] = {
1365:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1366:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00810772528427939684296334977783425582e0),
1367:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.24383652800043768524894854013745098654e-1),
1368:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.64696616559657052516796844068580626381e-1),
1369:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.62288747679271039067363492752820355369e-2),
1370:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19311779292286492714550084942827207241e-2),
1371:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.48436879303839576521077892946281025894e-3),
1372:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.28665316157256311138787387605249076674e-4),
1373:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.36350302380845433472593647100484547496e-5),
1374:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.05835458213330488018147374864403662878e-6),
1375:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.13919959493955187399856105325181806876e-7),
1376:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30960533107704070411766556906543316310e-8),
1377:         };
1378:         // LCOV_EXCL_STOP
1379:         result = tools::evaluate_polynomial(P, x) / tools::evaluate_polynomial(Q, x);
1380:     }
1381:     else if (x < 2) {
1382:         RealType t = x - 1;
1383: 
1384:         // Rational Approximation
1385:         // Maximum Relative Error: 1.6297e-35
1386:         // LCOV_EXCL_START
1387:         BOOST_MATH_STATIC const RealType P[13] = {
1388:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.84196970581015939887507434989936103587e-1),
1389:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23864910443500344832158256856064580005e-1),
1390:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.72066675347648126090497588433854314742e-2),
1391:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.81712740200456564860442639192891089515e-2),
1392:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.39091197181834765859741334477680768031e-3),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1393-1440 / 第 1393-1440 行
~~~cpp
1393:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.03759464781707198959689175957603165395e-4),
1394:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15298069568149410830642785868857309358e-4),
1395:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18910514301176322829267019223946392192e-5),
1396:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.16851691488007921400221017970691227149e-7),
1397:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.82031940093536875619655849638573432722e-8),
1398:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.30042143299959913519747484877532997335e-10),
1399:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.19848671456872291336347012756651759817e-11),
1400:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.00479393063394570750334218362674723065e-13),
1401:         };
1402:         BOOST_MATH_STATIC const RealType Q[12] = {
1403:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1404:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24929390929112144560152115661603117364e0),
1405:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.34853762543033883106055186520573363290e-1),
1406:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.73783624941936412984356492130276742707e-1),
1407:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23224734370942016023173307854505597524e-1),
1408:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.11116448823067697039703254343621931158e-2),
1409:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.12490054037308798338231679733816982120e-3),
1410:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.38701607014856856812627276285445001885e-4),
1411:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.10075199231657382435402462616587005087e-4),
1412:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.43662615015322880941108094510531477066e-6),
1413:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.37981396630189761210639158952200945512e-7),
1414:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.55820444854396304928946970937054949160e-8),
1415:         };
1416:         // LCOV_EXCL_STOP
1417:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1418:     }
1419:     else if (x < 4) {
1420:         RealType t = x - 2;
1421: 
1422:         // Rational Approximation
1423:         // Maximum Relative Error: 2.8103e-36
1424:         // LCOV_EXCL_START
1425:         BOOST_MATH_STATIC const RealType P[14] = {
1426:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07409273397524124098315500450332255837e-1),
1427:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.98373054365213259465522536994638631699e-2),
1428:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.30851284606709136235419547406278197945e-2),
1429:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.92686617543233900289721448026065555990e-2),
1430:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.18056394312484073294780140350522772329e-3),
1431:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07058343449035366484618967963264380933e-3),
1432:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71636108080692802684712497501670425230e-4),
1433:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.13155853034615230731719317488499751231e-5),
1434:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.00070273388376168880473457782396672044e-6),
1435:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.35528857373910910704625837069445190727e-7),
1436:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.99897218751541535347315078577172104436e-9),
1437:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.35092090729912631973050415647154137571e-10),
1438:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.72220647682193638971237255396233171508e-13),
1439:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.45008884108655511268690849420714428764e-15),
1440:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1441-1488 / 第 1441-1488 行
~~~cpp
1441:         BOOST_MATH_STATIC const RealType Q[14] = {
1442:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1443:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.42652074703683973183213296310906006173e0),
1444:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.03479786698331153607905223548719296572e0),
1445:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.95556520914240562719970700900964416000e-1),
1446:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.73127917601685318803655745157828471269e-1),
1447:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.63007065833918179119250623000791647836e-2),
1448:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.70652732923091039268400927316918354628e-3),
1449:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.60880782675229297981880241245777122866e-3),
1450:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.09979261868403910549978204036056659380e-4),
1451:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.12085610111710889118562321318284539217e-5),
1452:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59811533082647193392924345081953134304e-6),
1453:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.37211668706684650035086116219257276925e-8),
1454:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.62479830409039340826066305367893543134e-9),
1455:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.22039803134898937546371285610102850458e-11),
1456:         };
1457:         // LCOV_EXCL_STOP
1458:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1459:     }
1460:     else if (x < 8) {
1461:         RealType t = x - 4;
1462: 
1463:         // Rational Approximation
1464:         // Maximum Relative Error: 7.5930e-35
1465:         // LCOV_EXCL_START
1466:         BOOST_MATH_STATIC const RealType P[14] = {
1467:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.70720199535228802537946089633331273434e-2),
1468:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.85220706158476482443562698303252970927e-2),
1469:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.55090221054465759649629178911450010833e-2),
1470:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.70398047783095186291450019612979853708e-3),
1471:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.11846661331973171721224034349719801691e-3),
1472:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.83195024406409870789088752469490824640e-4),
1473:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.23908312140480103249294791529383548724e-5),
1474:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.40765128885655152415228193255890859830e-6),
1475:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.14294523267278070539100529759317560119e-7),
1476:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.26815059429007745850376987481747435820e-8),
1477:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.28142945635159623618312928455133399240e-10),
1478:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.77079683180868753715374495747422819326e-11),
1479:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.73710011278079325323578951018770847628e-13),
1480:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.70140037580287364298206334732060874507e-16),
1481:         };
1482:         BOOST_MATH_STATIC const RealType Q[16] = {
1483:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1484:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.36848014038411798213992770858203510748e0),
1485:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.15373052017549822413011375404872359177e-1),
1486:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.92705544967513282963463451395766172671e-1),
1487:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19899290805598090502434290420047460406e-1),
1488:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.74002906913724742582773116667380578990e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1489-1536 / 第 1489-1536 行
~~~cpp
1489:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.80632456977494447641985312297971970632e-3),
1490:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.53381530665983535467406445749348183915e-4),
1491:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.86606180756179817016240556949228031340e-5),
1492:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.49594666942152749850479792402078560469e-6),
1493:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.25231012522695972983928740617341887334e-7),
1494:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34987086926725472733984045599487947378e-8),
1495:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.58286136970918021841189712851698747417e-10),
1496:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.12238357666199366902936267515573231037e-12),
1497:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.82464168044335183356132979380360583444e-14),
1498:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.40073718480172265670072434562833527076e-17),
1499:         };
1500:         // LCOV_EXCL_STOP
1501:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1502:     }
1503:     else if (x < 16) {
1504:         RealType t = x - 8;
1505: 
1506:         // Rational Approximation
1507:         // Maximum Relative Error: 7.3609e-35
1508:         // LCOV_EXCL_START
1509:         BOOST_MATH_STATIC const RealType P[16] = {
1510:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.74847564444513000450056174922427854591e-2),
1511:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.56842503159303803254436983444304764079e-2),
1512:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.48504629497687889354406208309334148575e-3),
1513:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62327083507366120871877936416427790391e-3),
1514:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.72062210557023828776202679230979309963e-4),
1515:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.19153025667221102770398900522196418041e-5),
1516:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.66248482185063262034022017727375829162e-6),
1517:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.57390218395059632327421809878050974588e-7),
1518:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.45520328522839835737631604118833792570e-9),
1519:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.76327978880339919462910339138428389322e-10),
1520:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.99700625463451418394515481232159889297e-12),
1521:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.82943476668680389338853032002472541164e-14),
1522:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19415284760817575957617090798914089413e-16),
1523:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.17080879333540200065368097274334363537e-19),
1524:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.09912208206107606750610288716869139753e-23),
1525:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.98451733054622166748935243139556132704e-26),
1526:         };
1527:         BOOST_MATH_STATIC const RealType Q[15] = {
1528:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1529:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.08148065380582488495702136465010348576e0),
1530:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.42385352331252779422725444021027377277e-1),
1531:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.66510412535270623169792008730183916611e-1),
1532:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.47952712144801508762945315513819636452e-2),
1533:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.20703092334999244212988997416711617790e-3),
1534:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.71658889250345012472529115544710926154e-4),
1535:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.63905601023452497974798277091285373919e-5),
1536:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76730409484335386334980429532443217982e-6),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1537-1584 / 第 1537-1584 行
~~~cpp
1537:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19139408077753398896224794522985050607e-7),
1538:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.58025872548387600940275201648443410419e-9),
1539:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.11369336267349152895272975096509109414e-11),
1540:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.56182954522937999103610817174373785571e-13),
1541:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.35452907177197742692545044913125982311e-15),
1542:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.23587924912460218189929226092439805175e-17),
1543:         };
1544:         // LCOV_EXCL_STOP
1545:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1546:     }
1547:     else if (x < 32) {
1548:         RealType t = x - 16;
1549: 
1550:         // Rational Approximation
1551:         // Maximum Relative Error: 9.7192e-35
1552:         // LCOV_EXCL_START
1553:         BOOST_MATH_STATIC const RealType P[15] = {
1554:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.22684103170563193014558918295924551173e-3),
1555:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.55222688816852408105912768186300290291e-3),
1556:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.60747505331765587662432023547517953629e-4),
1557:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.80463770266821887100086895337451846880e-5),
1558:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.19190824169154471000496746227725070963e-6),
1559:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40646301571395681364881852739555404287e-7),
1560:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.15408836734496798091749932018121879724e-9),
1561:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.13676779930022341958128426888835497781e-11),
1562:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02435098103190516418351075792372986932e-12),
1563:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.82018920071479061978244972592746216377e-15),
1564:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.26435061215428679536159320644587957335e-17),
1565:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.05298407883178633891153989998582851270e-20),
1566:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.61156860101928352010449210760843428372e-23),
1567:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.02156808288545876198121127510075217184e-27),
1568:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.65549196385656698597261688277898043367e-30),
1569:         };
1570:         BOOST_MATH_STATIC const RealType Q[14] = {
1571:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1572:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.03426141030409708635168766288764563749e-1),
1573:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.13808987755928828118915442251025992769e-1),
1574:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52253239792170999949444502938290297674e-2),
1575:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33720936468171204432499390745432338841e-3),
1576:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.08713980159883984886576124842631646880e-5),
1577:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.43652846144339754840998823540656399165e-6),
1578:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02849693617024492825330133490278326951e-7),
1579:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.14110017452008167954262319462808192536e-9),
1580:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.01462578814695350559338360744897649915e-11),
1581:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.73495817568046489613308117490508832084e-13),
1582:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47445372925844096612021093857581987132e-15),
1583:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.08200002287534174751275097848899176785e-18),
1584:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.15305756373406702253187385797525419287e-21),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1585-1632 / 第 1585-1632 行
~~~cpp
1585:         };
1586:         // LCOV_EXCL_STOP
1587:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1588:     }
1589:     else if (x < 64) {
1590:         RealType t = x - 32;
1591: 
1592:         // Rational Approximation
1593:         // Maximum Relative Error: 9.7799e-35
1594:         // LCOV_EXCL_START
1595:         BOOST_MATH_STATIC const RealType P[13] = {
1596:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.20357145727036120652264700679701054983e-3),
1597:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.95712324967981162396595365933255312698e-4),
1598:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.08619492652809635942960438372427086939e-5),
1599:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.37140224583881547818087260161723208444e-6),
1600:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.83073777522092069988595553041062506001e-8),
1601:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.00473542739040742110568810201412321512e-10),
1602:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.47447289601822506789553624164171452120e-12),
1603:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.70913574957198131397471307249294758738e-14),
1604:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.36538119628489354953085829178695645929e-16),
1605:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00763343664814257170332492241110173166e-18),
1606:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62297585950798764290583627210836077239e-21),
1607:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15780217054514513493147192853488153246e-24),
1608:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.31961589164397397724611386366339562789e-28),
1609:         };
1610:         BOOST_MATH_STATIC const RealType Q[15] = {
1611:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1612:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.26440207646105117747875545474828367516e-1),
1613:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.27872879091838733280518786463281413334e-2),
1614:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.34256572873114675776148923422025029494e-3),
1615:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.13595637397535037957995766856628205747e-5),
1616:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33745879863685053883024090247009549434e-6),
1617:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.41792226523670940279016788831933559977e-8),
1618:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.03966147662273388060545199475024100492e-10),
1619:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.62177951640260313354050335795080248910e-12),
1620:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.50650165210517365082118441264513277196e-14),
1621:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.48413283257020741389298806290302772976e-17),
1622:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16439276222123152748426700489921412654e-19),
1623:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24969602890963356175782126478237865639e-22),
1624:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.08681155203261739689727004641345513984e-26),
1625:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.28282024196484688479115133027874255367e-30),
1626:         };
1627:         // LCOV_EXCL_STOP
1628:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1629:     }
1630:     else {
1631:         RealType x_cube = x * x * x;
1632:         RealType t = (boost::math::isnormal)(x_cube) ? 1 / sqrt(x_cube) : 1 / pow(sqrt(x), 3);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1633-1680 / 第 1633-1680 行
~~~cpp
1633: 
1634:         // Rational Approximation
1635:         // Maximum Relative Error: 3.5865e-37
1636:         // LCOV_EXCL_START
1637:         BOOST_MATH_STATIC const RealType P[8] = {
1638:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.98942280401432677939946059934381868476e-1),
1639:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.12426566605292130233061857505057433291e-1),
1640:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.91574528280329492283287073127040983832e1),
1641:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.69914217884224943794012165979483573091e1),
1642:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.30178902028403564086640591437738216288e3),
1643:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.96515490341559353794378324810127583810e2),
1644:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.44343825578434751356083230369361399507e3),
1645:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.07224810408790092272497403739984510394e2),
1646:         };
1647:         BOOST_MATH_STATIC const RealType Q[7] = {
1648:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1649:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.32474438135610721926278423612948794250e-1),
1650:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.27594461167587027771303292526448542806e2),
1651:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.49207539478843628626934249487055017677e1),
1652:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.75094412095634602055738687636893575929e3),
1653:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.51642534474780515366628648516673270623e2),
1654:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.05977615003758056284424652420774587813e4),
1655:         };
1656:         // LCOV_EXCL_STOP
1657:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t) * t;
1658:     }
1659: 
1660:     return result;
1661: }
1662: 
1663: template <class RealType>
1664: BOOST_MATH_GPU_ENABLED inline RealType mapairy_cdf_minus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 53>&)
1665: {
1666:     BOOST_MATH_STD_USING
1667:     RealType result;
1668: 
1669:     if (x >= -1) {
1670:         RealType t = x + 1;
1671: 
1672:         // Rational Approximation
1673:         // Maximum Relative Error: 1.6964e-18
1674:         BOOST_MATH_STATIC const RealType P[8] = {
1675:             static_cast<RealType>(4.23238998449671083670e-1),
1676:             static_cast<RealType>(4.95353582976475183891e-1),
1677:             static_cast<RealType>(2.45823281826037784270e-1),
1678:             static_cast<RealType>(7.29726507468813920788e-2),
1679:             static_cast<RealType>(1.63332856186819713346e-2),
1680:             static_cast<RealType>(2.82514634871307516142e-3),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 1681-1728 / 第 1681-1728 行
~~~cpp
1681:             static_cast<RealType>(2.66220579589280704089e-4),
1682:             static_cast<RealType>(3.09442180091323751049e-6),
1683:         };
1684:         BOOST_MATH_STATIC const RealType Q[7] = {
1685:             static_cast<RealType>(1.),
1686:             static_cast<RealType>(5.16241922223786900600e-1),
1687:             static_cast<RealType>(2.75690727171711638879e-1),
1688:             static_cast<RealType>(7.18707184893542884080e-2),
1689:             static_cast<RealType>(1.87136800286819336797e-2),
1690:             static_cast<RealType>(2.38383441176345054929e-3),
1691:             static_cast<RealType>(3.23509126477812051983e-4),
1692:         };
1693: 
1694:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1695:     }
1696:     else if (x >= -2) {
1697:         RealType t = x + 2;
1698: 
1699:         // Rational Approximation
1700:         // Maximum Relative Error: 5.8303e-17
1701:         BOOST_MATH_STATIC const RealType P[8] = {
1702:             static_cast<RealType>(1.62598955251978523175e-1),
1703:             static_cast<RealType>(2.30154661502402196205e-1),
1704:             static_cast<RealType>(1.29233975368291684522e-1),
1705:             static_cast<RealType>(3.80919553916980965587e-2),
1706:             static_cast<RealType>(8.17724414618808505948e-3),
1707:             static_cast<RealType>(1.95816800210481122544e-3),
1708:             static_cast<RealType>(3.35259917978421935141e-4),
1709:             static_cast<RealType>(1.22071311320012805777e-5),
1710:         };
1711:         BOOST_MATH_STATIC const RealType Q[7] = {
1712:             static_cast<RealType>(1.),
1713:             static_cast<RealType>(9.63771793313770952352e-2),
1714:             static_cast<RealType>(2.23602260938227310054e-1),
1715:             static_cast<RealType>(9.21944797677283179038e-3),
1716:             static_cast<RealType>(1.82181136341939651516e-2),
1717:             static_cast<RealType>(1.11216849284965970458e-4),
1718:             static_cast<RealType>(5.57446347676836375810e-4),
1719:         };
1720: 
1721:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1722:     }
1723:     else {
1724:         RealType s = exp(2 * x * x * x / 27) / sqrt(-x * x * x);
1725: 
1726:         if (x >= -4) {
1727:             RealType t = -x - 2;
1728: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1729-1776 / 第 1729-1776 行
~~~cpp
1729:             // Rational Approximation
1730:             // Maximum Relative Error: 3.6017e-18
1731:             BOOST_MATH_STATIC const RealType P[8] = {
1732:                 static_cast<RealType>(8.31806744221966404520e-1),
1733:                 static_cast<RealType>(1.34481067378012055850e0),
1734:                 static_cast<RealType>(9.12139427469494995264e-1),
1735:                 static_cast<RealType>(3.59706159222491124928e-1),
1736:                 static_cast<RealType>(9.48836332725688279299e-2),
1737:                 static_cast<RealType>(1.68259594978853951234e-2),
1738:                 static_cast<RealType>(1.89700733471520162946e-3),
1739:                 static_cast<RealType>(1.13854052826846329787e-4),
1740:             };
1741:             BOOST_MATH_STATIC const RealType Q[8] = {
1742:                 static_cast<RealType>(1.),
1743:                 static_cast<RealType>(1.29694286517571741097e0),
1744:                 static_cast<RealType>(7.99686735441213882518e-1),
1745:                 static_cast<RealType>(3.08198207583883597188e-1),
1746:                 static_cast<RealType>(7.97230139795658588972e-2),
1747:                 static_cast<RealType>(1.40742142048849462162e-2),
1748:                 static_cast<RealType>(1.58411440546277691506e-3),
1749:                 static_cast<RealType>(9.51560785730564046338e-5),
1750:             };
1751: 
1752:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1753:         }
1754:         else if (x >= -8) {
1755:             RealType t = -x - 4;
1756: 
1757:             // Rational Approximation
1758:             // Maximum Relative Error: 1.3504e-17
1759:             BOOST_MATH_STATIC const RealType P[8] = {
1760:                 static_cast<RealType>(1.10294551528734705946e0),
1761:                 static_cast<RealType>(1.26696377028973554615e0),
1762:                 static_cast<RealType>(6.63115985833429688941e-1),
1763:                 static_cast<RealType>(2.06289793717379095832e-1),
1764:                 static_cast<RealType>(4.11977615717846276227e-2),
1765:                 static_cast<RealType>(5.28620928618550859827e-3),
1766:                 static_cast<RealType>(4.04328442334023561279e-4),
1767:                 static_cast<RealType>(1.42364413902075896503e-5),
1768:             };
1769:             BOOST_MATH_STATIC const RealType Q[8] = {
1770:                 static_cast<RealType>(1.),
1771:                 static_cast<RealType>(1.09709853682665798542e0),
1772:                 static_cast<RealType>(5.63687797989627787500e-1),
1773:                 static_cast<RealType>(1.73604358560002859604e-1),
1774:                 static_cast<RealType>(3.44985744385890794044e-2),
1775:                 static_cast<RealType>(4.41683993064797272821e-3),
1776:                 static_cast<RealType>(3.37834206192286709492e-4),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1777-1824 / 第 1777-1824 行
~~~cpp
1777:                 static_cast<RealType>(1.18951465786445720729e-5),
1778:             };
1779: 
1780:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1781:         }
1782:         else if (x >= -16) {
1783:             RealType t = -x - 8;
1784: 
1785:             // Rational Approximation
1786:             // Maximum Relative Error: 8.8272e-18
1787:             BOOST_MATH_STATIC const RealType P[7] = {
1788:                 static_cast<RealType>(1.18246847255744057280e0),
1789:                 static_cast<RealType>(8.41320657699741240497e-1),
1790:                 static_cast<RealType>(2.55093097377551881478e-1),
1791:                 static_cast<RealType>(4.21261576802732715976e-2),
1792:                 static_cast<RealType>(3.98805044659990523312e-3),
1793:                 static_cast<RealType>(2.04688276265993954527e-4),
1794:                 static_cast<RealType>(4.43354791268634655473e-6),
1795:             };
1796:             BOOST_MATH_STATIC const RealType Q[7] = {
1797:                 static_cast<RealType>(1.),
1798:                 static_cast<RealType>(7.07103973315808077783e-1),
1799:                 static_cast<RealType>(2.13664682181055450396e-1),
1800:                 static_cast<RealType>(3.52218225168465984709e-2),
1801:                 static_cast<RealType>(3.33218664347896435919e-3),
1802:                 static_cast<RealType>(1.71025807471868853268e-4),
1803:                 static_cast<RealType>(3.70441884597642042665e-6),
1804:             };
1805: 
1806:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1807:         }
1808:         else if (x >= -32) {
1809:             RealType t = -x - 16;
1810: 
1811:             // Rational Approximation
1812:             // Maximum Relative Error: 2.6236e-18
1813:             BOOST_MATH_STATIC const RealType P[6] = {
1814:                 static_cast<RealType>(1.19497306481411168356e0),
1815:                 static_cast<RealType>(3.90497195765498241356e-1),
1816:                 static_cast<RealType>(5.13120330037626853257e-2),
1817:                 static_cast<RealType>(3.38574023921119491471e-3),
1818:                 static_cast<RealType>(1.12075935888344736993e-4),
1819:                 static_cast<RealType>(1.48743616420183584738e-6),
1820:             };
1821:             BOOST_MATH_STATIC const RealType Q[6] = {
1822:                 static_cast<RealType>(1.),
1823:                 static_cast<RealType>(3.26493785348088598123e-1),
1824:                 static_cast<RealType>(4.28813205161574223713e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1825-1872 / 第 1825-1872 行
~~~cpp
1825:                 static_cast<RealType>(2.82893073845390254969e-3),
1826:                 static_cast<RealType>(9.36442365966638579335e-5),
1827:                 static_cast<RealType>(1.24281651532469125315e-6),
1828:             };
1829: 
1830:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1831:         }
1832:         else {
1833:             result = 0;
1834:         }
1835:     }
1836: 
1837:     return result;
1838: }
1839: 
1840: template <class RealType>
1841: BOOST_MATH_GPU_ENABLED inline RealType mapairy_cdf_minus_imp_prec(const RealType& x, const boost::math::integral_constant<int, 113>&)
1842: {
1843:     BOOST_MATH_STD_USING
1844:     RealType result;
1845: 
1846:     if (x >= -1) {
1847:         RealType t = x + 1;
1848: 
1849:         // Rational Approximation
1850:         // Maximum Relative Error: 1.0688e-35
1851:         // LCOV_EXCL_START
1852:         BOOST_MATH_STATIC const RealType P[14] = {
1853:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.23238998449671083670041452413316011920e-1),
1854:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.14900991369455846775267187236501987891e-1),
1855:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.19132787054572299485638029221977944555e-1),
1856:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.87295743700300806662745209398368996653e-1),
1857:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.41994520703802035725356673887766112213e-2),
1858:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.78782099629586443747968633412271291734e-2),
1859:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.05200546520666366552864974572901349343e-3),
1860:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.51453477916196630939702866688348310208e-4),
1861:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15461354910584918402088506199099270742e-4),
1862:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43371674256124419899137414410592359185e-5),
1863:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.35849788347057186916350200082990102088e-6),
1864:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.50359296597872967493549820191745700442e-8),
1865:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.21838020977580479741299141050400953125e-9),
1866:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.46723648594704078875476888175530463986e-12),
1867:         };
1868:         BOOST_MATH_STATIC const RealType Q[13] = {
1869:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1870:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.98700317671474659677458220091101276158e-1),
1871:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.00405631175818416028878082789095587658e-1),
1872:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.04189939150805562128632256692765842568e-1),
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial。

### Lines 1873-1920 / 第 1873-1920 行
~~~cpp
1873:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.03621065280443734565418469521814125946e-2),
1874:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.85722257874304617269018116436650330070e-2),
1875:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.24191409213079401989695901900760076094e-3),
1876:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.64269032641964601932953114106294883156e-4),
1877:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19289631274036494326058240677240511431e-4),
1878:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41389309719775603006897751176159931569e-5),
1879:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.42000309062533491486426399210996541477e-6),
1880:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.02436961569668743353755318268149636644e-8),
1881:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.50130875023154569442119099173406269991e-9),
1882:         };
1883:         // LCOV_EXCL_STOP
1884:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1885:     }
1886:     else if (x >= -2) {
1887:         RealType t = x + 2;
1888: 
1889:         // Rational Approximation
1890:         // Maximum Relative Error: 5.1815e-35
1891:         // LCOV_EXCL_START
1892:         BOOST_MATH_STATIC const RealType P[14] = {
1893:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62598955251978523174755901843430986522e-1),
1894:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.08127698872954954678270473317137288772e-1),
1895:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.70144997468767751317246482211703706086e-1),
1896:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49486603823046766249106014234315835102e-1),
1897:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.07186495389828596786579668258622667573e-2),
1898:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.98334953533562948674335281457057445421e-2),
1899:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.44119017374895211020429143034854620303e-3),
1900:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.27080759819117162456137826659721634882e-3),
1901:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.53892796920597912362370019918933112349e-4),
1902:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.30530442651657077016130554430933607143e-5),
1903:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.04837779538527662990102489150650534390e-6),
1904:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.94354615171320374997141684442120888127e-7),
1905:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.30746545799073289786965697800049892311e-8),
1906:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.41870129065056783732691371215602982173e-9),
1907:         };
1908:         BOOST_MATH_STATIC const RealType Q[14] = {
1909:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1910:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.75919235734607601884356783586727272494e-1),
1911:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.57656678936617227532275100649989944452e-1),
1912:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.72617552401870454676736869003112018648e-1),
1913:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.59238104942208254162102314312757621047e-2),
1914:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06040513359343987972917295603514840777e-2),
1915:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.26922840063034349024167652148593396307e-3),
1916:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.25628506630180107357627955876231943531e-3),
1917:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.81600387497542714853225329159728694926e-4),
1918:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.08210973846891324886779444820838563800e-5),
1919:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.68632477858150229792523037059221563861e-6),
1920:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.43542789104866782087701759971538600076e-7),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1921-1968 / 第 1921-1968 行
~~~cpp
1921:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.70594730517167328271953424328890849790e-8),
1922:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.30162314557860623869079601905904538470e-9),
1923:         };
1924:         // LCOV_EXCL_STOP
1925:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1926:     }
1927:     else {
1928:         RealType s = exp(2 * x * x * x / 27) / sqrt(-x * x * x);
1929: 
1930:         if (x >= -4) {
1931:             RealType t = -x - 2;
1932: 
1933:             // Rational Approximation
1934:             // Maximum Relative Error: 6.4678e-36
1935:             // LCOV_EXCL_START
1936:             BOOST_MATH_STATIC const RealType P[16] = {
1937:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.31806744221966404520449104514474066823e-1),
1938:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.50292887071777664663197915067642779665e0),
1939:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.45140067157601150721516139901304901854e0),
1940:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.93227973605511286112712730820664209900e0),
1941:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.74259108933048973391560053531348126900e0),
1942:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.77677252890665602191818487592154553094e-1),
1943:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.71843197238558832510595724454548089268e-1),
1944:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.62811778285151415483649897138119310816e-2),
1945:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.74127763877120261698596916683136227034e-2),
1946:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.24832552591462216226478550702845438540e-3),
1947:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.93381036027487259940171548523889481080e-4),
1948:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.02261328789519398745019578211081412570e-5),
1949:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.75409238451885381267277435341417474231e-6),
1950:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.09526311389365895099871581844304449319e-7),
1951:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.96371756262605118060185816854433322493e-8),
1952:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.88472819535099746216179119978362211227e-10),
1953:             };
1954:             BOOST_MATH_STATIC const RealType Q[16] = {
1955:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
1956:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.68923525157720774962908922391133419863e0),
1957:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.40714902096062779527207435671907059131e0),
1958:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.73120596883364361220343183559076165363e0),
1959:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.56331267512666685349409906638266569733e0),
1960:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.80956276267438042306216894159447642323e-1),
1961:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.34213468750936211385520570062547991332e-1),
1962:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.50081600968590616549654807511166706919e-2),
1963:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47297208240850928379158677132220746750e-2),
1964:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.73392976579560287571141938466202325901e-3),
1965:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.13858821123741335782695407397784840241e-4),
1966:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.04103497389656828224053882850778186433e-5),
1967:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.81040189127998139689091455192659191796e-6),
1968:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.42176283104790992634826374270801565123e-7),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, exp. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, exp。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 1969-2016 / 第 1969-2016 行
~~~cpp
1969:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.64077137545614380065714904794220228239e-8),
1970:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.08139724991616322332901357866680220241e-10),
1971:             };
1972:             // LCOV_EXCL_STOP
1973:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
1974:         }
1975:         else if (x >= -8) {
1976:             RealType t = -x - 4;
1977: 
1978:             // Rational Approximation
1979:             // Maximum Relative Error: 3.5975e-36
1980:             // LCOV_EXCL_START
1981:             BOOST_MATH_STATIC const RealType P[16] = {
1982:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.10294551528734705945662709421382590676e0),
1983:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.26135857114883288617323671166863478751e0),
1984:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.23504465936865651893193560109437792738e0),
1985:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41598983788870071301270649341678962009e0),
1986:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.43871304031224174103636568402522086316e-1),
1987:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.22745720458050596514499383658714367529e-1),
1988:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.05976431838299244997805790000128175545e-2),
1989:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.32087500190238014890030606301748111874e-2),
1990:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.32754445514451500968404092049839985196e-3),
1991:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.31866016448921762610690552586049011375e-4),
1992:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.80197257671079297305525087998125408939e-5),
1993:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.44212088947602969374978384512149432847e-6),
1994:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.38857170416924025226203571589937286465e-7),
1995:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.20239999218390467567339789443070294182e-8),
1996:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.93965060142992479149039624149602039394e-10),
1997:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.36407983918582149239548869529460234702e-12),
1998:             };
1999:             BOOST_MATH_STATIC const RealType Q[16] = {
2000:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2001:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.99867960957804580209868321228347067213e0),
2002:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.94236623527818880544030470097296139679e0),
2003:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21644866845440678050425616384656052588e0),
2004:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.48653919287388803523090727546630060490e-1),
2005:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.88696531788490258477870877792341909659e-1),
2006:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.11157356307921406032115084386689196255e-2),
2007:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11071149696069503480091810333521267753e-2),
2008:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.95274844731679437274609760294652465905e-3),
2009:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.77971280158253322431071249000491659536e-4),
2010:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.18092258028773913076132483326275839950e-5),
2011:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.87773088535057996947643657676843842076e-6),
2012:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.99609277781492599950063871899582711550e-7),
2013:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00465660598924300723542908245498229301e-8),
2014:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.29174652982710100418474261697035968379e-10),
2015:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.31746082236506935340972706820707017875e-12),
2016:             };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2017-2064 / 第 2017-2064 行
~~~cpp
2017:             // LCOV_EXCL_STOP
2018:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2019:         }
2020:         else if (x >= -16) {
2021:             RealType t = -x - 8;
2022: 
2023:             // Rational Approximation
2024:             // Maximum Relative Error: 2.6792e-35
2025:             // LCOV_EXCL_START
2026:             BOOST_MATH_STATIC const RealType P[15] = {
2027:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18246847255744057280356900905660312795e0),
2028:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.77955886026107125189834586992142580148e0),
2029:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24948425302263641813107623611637262126e0),
2030:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.42593662659560333324287312162818766556e-1),
2031:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62714138002904073145045478360748042164e-1),
2032:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.56008984285541289474850396553042124777e-2),
2033:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.84858048549330525583286373950733005244e-3),
2034:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.30578460156038467943968005946143934751e-4),
2035:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.93974868941529258700281962314167648967e-5),
2036:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.95086664204515648622431580749060079100e-6),
2037:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.57811968176644056830002158465591081929e-7),
2038:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.27814751838906948007289825582251221538e-9),
2039:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06762893426725920159998333647896590440e-10),
2040:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.15388861641344998301210173677051088515e-12),
2041:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.83956842740198388242245209024484381888e-29),
2042:             };
2043:             BOOST_MATH_STATIC const RealType Q[14] = {
2044:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2045:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.50056124032615852703112365430040751173e0),
2046:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.05112559537845833793684655693572118348e0),
2047:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.55609497026127521043140534271852131858e-1),
2048:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.36430652394614121156238070755223942728e-1),
2049:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.98167215021940993097697777547641188697e-2),
2050:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.89418831310297071347013983522734394061e-3),
2051:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.10980717618462843498917526227524790487e-4),
2052:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.80119618735773019675212434416594954984e-5),
2053:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.13748676086657187580746476165248613583e-6),
2054:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.15424395860921826755718081823964568760e-7),
2055:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.75228896859720124469916340725146705309e-9),
2056:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.72759238269315282789451836388878919387e-10),
2057:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.79966603543593799412565926418879689461e-12),
2058:             };
2059:             // LCOV_EXCL_STOP
2060:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2061:         }
2062:         else if (x >= -32) {
2063:             RealType t = -x - 16;
2064: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2065-2112 / 第 2065-2112 行
~~~cpp
2065:             // Rational Approximation
2066:             // Maximum Relative Error: 2.1744e-36
2067:             // LCOV_EXCL_START
2068:             BOOST_MATH_STATIC const RealType P[12] = {
2069:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19497306481411168355692832231058399132e0),
2070:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.15593166681833539521403250736661720488e-1),
2071:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.54020260738207743315755235213180652303e-1),
2072:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.76467972857585566189917087631621063058e-2),
2073:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.97922795572348613358915532172847895070e-3),
2074:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.26998967192207380100354278434037095729e-4),
2075:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.32827180395699855050424881575240362199e-5),
2076:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.50587178182571637802022891868380669565e-6),
2077:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.78252548290929962236994183546354358888e-8),
2078:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.01519297007773622283120166415145520855e-9),
2079:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.29602226691665918537895803270497291716e-11),
2080:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.53666531487585211574942518181922132884e-14),
2081:             };
2082:             BOOST_MATH_STATIC const RealType Q[12] = {
2083:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2084:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.82230649578130958108098853863277631065e-1),
2085:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.12412482738973738235656376802445565005e-1),
2086:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.98320116955422615960870363549721494683e-2),
2087:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.99756654189000467678223166815845628725e-3),
2088:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.40414942475279981724792023159180203408e-4),
2089:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.78118445466942812088955228016254912391e-5),
2090:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.25827002637577602812624580692342616301e-6),
2091:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.99604467789028963216078448884632489822e-8),
2092:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.48237134334492289420105516726562561260e-10),
2093:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.08288201960155447241423587030002372229e-11),
2094:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.29720612489952110448407063201146274502e-14),
2095:             };
2096:             // LCOV_EXCL_STOP
2097:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2098:         }
2099:         else if (x >= -64) {
2100:             RealType t = -x - 32;
2101: 
2102:             // Rational Approximation
2103:             // Maximum Relative Error: 3.4699e-36
2104:             // LCOV_EXCL_START
2105:             BOOST_MATH_STATIC const RealType P[10] = {
2106:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.19659414007358083585943280640656311534e0),
2107:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.36969140730640253987817932335415532846e-1),
2108:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.21946928005759888612066397569236165853e-2),
2109:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.08341720579009422518863704766395201498e-3),
2110:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44908614491286780138818989614277172709e-4),
2111:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.54172482866925057749338312942859761961e-6),
2112:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.49281630950104861570255344237175124548e-8),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2113-2160 / 第 2113-2160 行
~~~cpp
2113:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.27586759709416364899010676712546639820e-9),
2114:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00054716479138657682306851175059678989e-11),
2115:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.48798342894235412426464893852098239746e-14),
2116:             };
2117:             BOOST_MATH_STATIC const RealType Q[10] = {
2118:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2119:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.81588658109851219975949691772676519853e-1),
2120:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.52583331848892383968186924120872369151e-2),
2121:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.57644670426430994363913234422346706991e-3),
2122:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21080164634428298820141591419770346977e-4),
2123:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.79484074949580980980061103238709314326e-6),
2124:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.93167250146504946763386377338487557826e-8),
2125:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06604193118724797924138056151582242604e-9),
2126:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.35999937789324222934257460080153249173e-12),
2127:                 BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.91435929481043135336094426837156247599e-14),
2128:             };
2129:             // LCOV_EXCL_STOP
2130:             result = s * tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2131:         }
2132:         else {
2133:             result = 0;
2134:         }
2135:     }
2136: 
2137:     return result;
2138: }
2139: 
2140: template <class RealType>
2141: BOOST_MATH_GPU_ENABLED inline RealType mapairy_cdf_imp_prec(const RealType& x, bool complement, const boost::math::integral_constant<int, 53>& tag) {
2142:     if (x >= 0) {
2143:         return complement ? mapairy_cdf_plus_imp_prec(x, tag) : 1 - mapairy_cdf_plus_imp_prec(x, tag);
2144:     }
2145:     else if (x <= 0) {
2146:         return complement ? 1 - mapairy_cdf_minus_imp_prec(x, tag) : mapairy_cdf_minus_imp_prec(x, tag);
2147:     }
2148:     else {
2149:         return boost::math::numeric_limits<RealType>::quiet_NaN();
2150:     }
2151: }
2152: 
2153: template <class RealType>
2154: BOOST_MATH_GPU_ENABLED inline RealType mapairy_cdf_imp_prec(const RealType& x, bool complement, const boost::math::integral_constant<int, 113>& tag) {
2155:     if (x >= 0) {
2156:         return complement ? mapairy_cdf_plus_imp_prec(x, tag) : 1 - mapairy_cdf_plus_imp_prec(x, tag);
2157:     }
2158:     else if (x <= 0) {
2159:         return complement ? 1 - mapairy_cdf_minus_imp_prec(x, tag) : mapairy_cdf_minus_imp_prec(x, tag);
2160:     }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, mapairy_cdf_imp_prec, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, mapairy_cdf_imp_prec, ...。

### Lines 2161-2208 / 第 2161-2208 行
~~~cpp
2161:     else {
2162:         return boost::math::numeric_limits<RealType>::quiet_NaN();
2163:     }
2164: }
2165: 
2166: template <class RealType, class Policy>
2167: BOOST_MATH_GPU_ENABLED inline RealType mapairy_cdf_imp(const mapairy_distribution<RealType, Policy>& dist, const RealType& x, bool complement) {
2168:     //
2169:     // This calculates the cdf of the Map-Airy distribution and/or its complement.
2170:     //
2171: 
2172:     BOOST_MATH_STD_USING // for ADL of std functions
2173:     constexpr auto function = "boost::math::cdf(mapairy<%1%>&, %1%)";
2174:     RealType result = 0;
2175:     RealType location = dist.location();
2176:     RealType scale = dist.scale();
2177: 
2178:     if (false == detail::check_location(function, location, &result, Policy()))
2179:     {
2180:         return result;
2181:     }
2182:     if (false == detail::check_scale(function, scale, &result, Policy()))
2183:     {
2184:         return result;
2185:     }
2186:     if (false == detail::check_x(function, x, &result, Policy()))
2187:     {
2188:         return result;
2189:     }
2190: 
2191:     typedef typename tools::promote_args<RealType>::type result_type;
2192:     typedef typename policies::precision<result_type, Policy>::type precision_type;
2193:     typedef boost::math::integral_constant<int,
2194:         precision_type::value <= 0 ? 0 :
2195:         precision_type::value <= 53 ? 53 :
2196:         precision_type::value <= 113 ? 113 : 0
2197:     > tag_type;
2198: 
2199:     static_assert(tag_type::value, "The Map-Airy distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
2200: 
2201:     RealType u = (x - location) / scale;
2202: 
2203:     result = mapairy_cdf_imp_prec(u, complement, tag_type());
2204: 
2205:     return result;
2206: }
2207: 
2208: template <class RealType>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as mapairy_cdf_imp, location, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 mapairy_cdf_imp, location, ...。

### Lines 2209-2256 / 第 2209-2256 行
~~~cpp
2209: BOOST_MATH_GPU_ENABLED inline RealType mapairy_quantile_lower_imp_prec(const RealType& p, const boost::math::integral_constant<int, 53>&)
2210: {
2211:     BOOST_MATH_STD_USING
2212:     RealType result;
2213: 
2214:     if (p >= 0.375) {
2215:         RealType t = p - static_cast <RealType>(0.375);
2216: 
2217:         // Rational Approximation
2218:         // Maximum Relative Error: 1.5488e-18
2219:         BOOST_MATH_STATIC const RealType P[7] = {
2220:             static_cast<RealType>(-1.17326074020471664075e0),
2221:             static_cast<RealType>(1.51461298154568349598e0),
2222:             static_cast<RealType>(1.19979368094343490487e1),
2223:             static_cast<RealType>(-5.94882121521324108164e0),
2224:             static_cast<RealType>(-2.20619749774447254528e1),
2225:             static_cast<RealType>(7.17766543775229176131e0),
2226:             static_cast<RealType>(4.79284243496552841508e0),
2227:         };
2228:         BOOST_MATH_STATIC const RealType Q[7] = {
2229:             static_cast<RealType>(1.),
2230:             static_cast<RealType>(1.76268072706610602584e0),
2231:             static_cast<RealType>(-4.88492535243404839734e0),
2232:             static_cast<RealType>(-5.67524172432687656881e0),
2233:             static_cast<RealType>(6.83327389947131710596e0),
2234:             static_cast<RealType>(2.91338085774159042709e0),
2235:             static_cast<RealType>(-1.41108918944159283950e0),
2236:         };
2237: 
2238:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2239:     }
2240:     else if (p >= 0.25) {
2241:         RealType t = p - static_cast <RealType>(0.25);
2242: 
2243:         // Rational Approximation
2244:         // Maximum Relative Error: 7.5181e-17
2245:         BOOST_MATH_STATIC const RealType P[7] = {
2246:             static_cast<RealType>(-1.63281240925531302762e0),
2247:             static_cast<RealType>(-4.92351310795930780147e0),
2248:             static_cast<RealType>(1.43448529253101759409e1),
2249:             static_cast<RealType>(3.33182629948094299473e1),
2250:             static_cast<RealType>(-3.06679026539368582747e1),
2251:             static_cast<RealType>(-2.87298447423841965301e1),
2252:             static_cast<RealType>(1.31575930750093554120e1),
2253:         };
2254:         BOOST_MATH_STATIC const RealType Q[6] = {
2255:             static_cast<RealType>(1.),
2256:             static_cast<RealType>(5.38761652244702318296e0),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2257-2304 / 第 2257-2304 行
~~~cpp
2257:             static_cast<RealType>(2.40932080746189543284e0),
2258:             static_cast<RealType>(-1.69465870062123632126e1),
2259:             static_cast<RealType>(-6.39998944283654848809e0),
2260:             static_cast<RealType>(1.27168434054332272391e1),
2261:         };
2262: 
2263:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2264:     }
2265:     else if (p >= 0.125) {
2266:         RealType t = p - static_cast <RealType>(0.125);
2267: 
2268:         // Rational Approximation
2269:         // Maximum Relative Error: 2.3028e-18
2270:         BOOST_MATH_STATIC const RealType P[8] = {
2271:             static_cast<RealType>(-2.18765177572396469657e0),
2272:             static_cast<RealType>(-3.65752788934974426531e1),
2273:             static_cast<RealType>(-1.81144810822028903904e2),
2274:             static_cast<RealType>(-1.22434531262312950288e2),
2275:             static_cast<RealType>(8.99451018491165823831e2),
2276:             static_cast<RealType>(9.11333307522308410858e2),
2277:             static_cast<RealType>(-8.76285742384616909177e2),
2278:             static_cast<RealType>(-2.33786726970025938837e2),
2279:         };
2280:         BOOST_MATH_STATIC const RealType Q[8] = {
2281:             static_cast<RealType>(1.),
2282:             static_cast<RealType>(1.91797638291395345792e1),
2283:             static_cast<RealType>(1.24293724082506952768e2),
2284:             static_cast<RealType>(2.82393116012902543276e2),
2285:             static_cast<RealType>(-1.80472369158936285558e1),
2286:             static_cast<RealType>(-5.31764390192922827093e2),
2287:             static_cast<RealType>(-5.60586018315854885788e1),
2288:             static_cast<RealType>(1.21284324755968033098e2),
2289:         };
2290: 
2291:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2292:     }
2293:     else if (ilogb(p) >= -4) {
2294:         RealType t = -log2(ldexp(p, 3));
2295: 
2296:         // Rational Approximation
2297:         // Maximum Relative Error: 6.1147e-18
2298:         BOOST_MATH_STATIC const RealType P[6] = {
2299:             static_cast<RealType>(-2.18765177572396470773e0),
2300:             static_cast<RealType>(-2.19887766409334094428e0),
2301:             static_cast<RealType>(-7.77080107207360785208e-1),
2302:             static_cast<RealType>(-1.15551765136654549650e-1),
2303:             static_cast<RealType>(-6.64711321022529990367e-3),
2304:             static_cast<RealType>(-9.74212491048543799073e-5),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2305-2352 / 第 2305-2352 行
~~~cpp
2305:         };
2306:         BOOST_MATH_STATIC const RealType Q[6] = {
2307:             static_cast<RealType>(1.),
2308:             static_cast<RealType>(7.91919722132624625590e-1),
2309:             static_cast<RealType>(2.17415447268626558639e-1),
2310:             static_cast<RealType>(2.41474762519410575392e-2),
2311:             static_cast<RealType>(9.41084107182696904714e-4),
2312:             static_cast<RealType>(6.65754108797614202364e-6),
2313:         };
2314: 
2315:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2316:     }
2317:     else if (ilogb(p) >= -8) {
2318:         RealType t = -log2(ldexp(p, 4));
2319: 
2320:         // Rational Approximation
2321:         // Maximum Relative Error: 2.0508e-17
2322:         BOOST_MATH_STATIC const RealType P[8] = {
2323:             static_cast<RealType>(-2.59822399410385085335e0),
2324:             static_cast<RealType>(-2.24306757759003016244e0),
2325:             static_cast<RealType>(-7.36208578161752060979e-1),
2326:             static_cast<RealType>(-1.15130762650287391576e-1),
2327:             static_cast<RealType>(-8.77652386123688618995e-3),
2328:             static_cast<RealType>(-2.96358888256575251437e-4),
2329:             static_cast<RealType>(-3.33661282483762192446e-6),
2330:             static_cast<RealType>(-4.19292241201527861927e-9),
2331:         };
2332:         BOOST_MATH_STATIC const RealType Q[7] = {
2333:             static_cast<RealType>(1.),
2334:             static_cast<RealType>(7.23065798041556418844e-1),
2335:             static_cast<RealType>(1.96731305131315877264e-1),
2336:             static_cast<RealType>(2.49952034298034383781e-2),
2337:             static_cast<RealType>(1.49149568322111062242e-3),
2338:             static_cast<RealType>(3.66010398525593921460e-5),
2339:             static_cast<RealType>(2.46857713549279930857e-7),
2340:         };
2341: 
2342:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2343:     }
2344:     else if (ilogb(p) >= -16) {
2345:         RealType t = -log2(ldexp(p, 8));
2346: 
2347:         // Rational Approximation
2348:         // Maximum Relative Error: 2.1997e-17
2349:         BOOST_MATH_STATIC const RealType P[8] = {
2350:             static_cast<RealType>(-3.67354365380697580447e0),
2351:             static_cast<RealType>(-1.52181685844845957618e0),
2352:             static_cast<RealType>(-2.40883948836320845233e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2353-2400 / 第 2353-2400 行
~~~cpp
2353:             static_cast<RealType>(-1.82424079258401987512e-2),
2354:             static_cast<RealType>(-6.75844978572417703979e-4),
2355:             static_cast<RealType>(-1.11273358356809152121e-5),
2356:             static_cast<RealType>(-6.12797605223700996671e-8),
2357:             static_cast<RealType>(-3.78061321691170114390e-11),
2358:         };
2359:         BOOST_MATH_STATIC const RealType Q[7] = {
2360:             static_cast<RealType>(1.),
2361:             static_cast<RealType>(3.57770840766081587688e-1),
2362:             static_cast<RealType>(4.81290550545412209056e-2),
2363:             static_cast<RealType>(3.02079969075162071807e-3),
2364:             static_cast<RealType>(8.89589626547135423615e-5),
2365:             static_cast<RealType>(1.07618717290978464257e-6),
2366:             static_cast<RealType>(3.57383804712249921193e-9),
2367:         };
2368: 
2369:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2370:     }
2371:     else if (ilogb(p) >= -32) {
2372:         RealType t = -log2(ldexp(p, 16));
2373: 
2374:         // Rational Approximation
2375:         // Maximum Relative Error: 2.4331e-17
2376:         BOOST_MATH_STATIC const RealType P[8] = {
2377:             static_cast<RealType>(-4.92187819510636697128e0),
2378:             static_cast<RealType>(-9.94924018698264727979e-1),
2379:             static_cast<RealType>(-7.69914962772717316098e-2),
2380:             static_cast<RealType>(-2.85558010159310978248e-3),
2381:             static_cast<RealType>(-5.19022578720207406789e-5),
2382:             static_cast<RealType>(-4.19975546950263453259e-7),
2383:             static_cast<RealType>(-1.13886013623971006760e-9),
2384:             static_cast<RealType>(-3.46758191090170732580e-13),
2385:         };
2386:         BOOST_MATH_STATIC const RealType Q[7] = {
2387:             static_cast<RealType>(1.),
2388:             static_cast<RealType>(1.77270673840643360017e-1),
2389:             static_cast<RealType>(1.18099604045834575786e-2),
2390:             static_cast<RealType>(3.66889581757166584963e-4),
2391:             static_cast<RealType>(5.34484782554469770841e-6),
2392:             static_cast<RealType>(3.19694601727035291809e-8),
2393:             static_cast<RealType>(5.24649233511937214948e-11),
2394:         };
2395: 
2396:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2397:     }
2398:     else if (ilogb(p) >= -64) {
2399:         RealType t = -log2(ldexp(p, 32));
2400: 
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2401-2448 / 第 2401-2448 行
~~~cpp
2401:         // Rational Approximation
2402:         // Maximum Relative Error: 2.7742e-17
2403:         BOOST_MATH_STATIC const RealType P[8] = {
2404:             static_cast<RealType>(-6.41443550638291133784e0),
2405:             static_cast<RealType>(-6.38369359780748328332e-1),
2406:             static_cast<RealType>(-2.43420704406734621618e-2),
2407:             static_cast<RealType>(-4.45274771094277987075e-4),
2408:             static_cast<RealType>(-3.99529078051262843241e-6),
2409:             static_cast<RealType>(-1.59758677464731620413e-8),
2410:             static_cast<RealType>(-2.14338367751477432622e-11),
2411:             static_cast<RealType>(-3.23343844538964435927e-15),
2412:         };
2413:         BOOST_MATH_STATIC const RealType Q[7] = {
2414:             static_cast<RealType>(1.),
2415:             static_cast<RealType>(8.79845511272943785289e-2),
2416:             static_cast<RealType>(2.90839059356197474893e-3),
2417:             static_cast<RealType>(4.48172838083912540123e-5),
2418:             static_cast<RealType>(3.23770691025690100895e-7),
2419:             static_cast<RealType>(9.60156044379859908674e-10),
2420:             static_cast<RealType>(7.81134095049301988435e-13),
2421:         };
2422: 
2423:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2424:     }
2425:     else if (ilogb(p) >= -128) {
2426:         RealType t = -log2(ldexp(p, 64));
2427: 
2428:         // Rational Approximation
2429:         // Maximum Relative Error: 3.2451e-17
2430:         BOOST_MATH_STATIC const RealType P[8] = {
2431:             static_cast<RealType>(-8.23500806363233610938e0),
2432:             static_cast<RealType>(-4.05652655284908839003e-1),
2433:             static_cast<RealType>(-7.65978833819859622912e-3),
2434:             static_cast<RealType>(-6.94194676058731901672e-5),
2435:             static_cast<RealType>(-3.08771646223818451436e-7),
2436:             static_cast<RealType>(-6.12443207313641110962e-10),
2437:             static_cast<RealType>(-4.07882839359528825925e-13),
2438:             static_cast<RealType>(-3.05720104049292610799e-17),
2439:         };
2440:         BOOST_MATH_STATIC const RealType Q[7] = {
2441:             static_cast<RealType>(1.),
2442:             static_cast<RealType>(4.37395212065018405474e-2),
2443:             static_cast<RealType>(7.18654254114820140590e-4),
2444:             static_cast<RealType>(5.50371158026951899491e-6),
2445:             static_cast<RealType>(1.97583864365011234715e-8),
2446:             static_cast<RealType>(2.91169706068202431036e-11),
2447:             static_cast<RealType>(1.17716830382540977039e-14),
2448:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2449-2496 / 第 2449-2496 行
~~~cpp
2449: 
2450:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2451:     }
2452:     else if (ilogb(p) >= -256) {
2453:         RealType t = -log2(ldexp(p, 128));
2454: 
2455:         // Rational Approximation
2456:         // Maximum Relative Error: 3.8732e-17
2457:         BOOST_MATH_STATIC const RealType P[8] = {
2458:             static_cast<RealType>(-1.04845570631944023913e1),
2459:             static_cast<RealType>(-2.56502856165700644836e-1),
2460:             static_cast<RealType>(-2.40615394566347412600e-3),
2461:             static_cast<RealType>(-1.08364601171893250764e-5),
2462:             static_cast<RealType>(-2.39603255140022514289e-8),
2463:             static_cast<RealType>(-2.36344017673944676435e-11),
2464:             static_cast<RealType>(-7.83146284114485675414e-15),
2465:             static_cast<RealType>(-2.92218240202835807955e-19),
2466:         };
2467:         BOOST_MATH_STATIC const RealType Q[7] = {
2468:             static_cast<RealType>(1.),
2469:             static_cast<RealType>(2.17740414929742679904e-2),
2470:             static_cast<RealType>(1.78084231709097280884e-4),
2471:             static_cast<RealType>(6.78870668961146609668e-7),
2472:             static_cast<RealType>(1.21313439060489363960e-9),
2473:             static_cast<RealType>(8.89917934953781122884e-13),
2474:             static_cast<RealType>(1.79115540847944524599e-16),
2475:         };
2476: 
2477:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2478:     }
2479:     else if (ilogb(p) >= -512) {
2480:         RealType t = -log2(ldexp(p, 256));
2481: 
2482:         // Rational Approximation
2483:         // Maximum Relative Error: 4.6946e-17
2484:         BOOST_MATH_STATIC const RealType P[8] = {
2485:             static_cast<RealType>(-1.32865827226175698181e1),
2486:             static_cast<RealType>(-1.61802434199627472010e-1),
2487:             static_cast<RealType>(-7.55642602577784211259e-4),
2488:             static_cast<RealType>(-1.69457608092375302291e-6),
2489:             static_cast<RealType>(-1.86612389867293722402e-9),
2490:             static_cast<RealType>(-9.17015770142364635163e-13),
2491:             static_cast<RealType>(-1.51422473889348610974e-16),
2492:             static_cast<RealType>(-2.81661279271583206526e-21),
2493:         };
2494:         BOOST_MATH_STATIC const RealType Q[7] = {
2495:             static_cast<RealType>(1.),
2496:             static_cast<RealType>(1.08518414679241420227e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2497-2544 / 第 2497-2544 行
~~~cpp
2497:             static_cast<RealType>(4.42335224797004486239e-5),
2498:             static_cast<RealType>(8.40387821972524402121e-8),
2499:             static_cast<RealType>(7.48486746424527560620e-11),
2500:             static_cast<RealType>(2.73676810622938942041e-14),
2501:             static_cast<RealType>(2.74588200481263214866e-18),
2502:         };
2503: 
2504:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2505:     }
2506:     else if (ilogb(p) >= -1024) {
2507:         RealType t = -log2(ldexp(p, 512));
2508: 
2509:         // Rational Approximation
2510:         // Maximum Relative Error: 5.7586e-17
2511:         BOOST_MATH_STATIC const RealType P[8] = {
2512:             static_cast<RealType>(-1.67937186583822375593e1),
2513:             static_cast<RealType>(-1.01958138247797604098e-1),
2514:             static_cast<RealType>(-2.37409774265951876695e-4),
2515:             static_cast<RealType>(-2.65483321307104128810e-7),
2516:             static_cast<RealType>(-1.45803536947907216594e-10),
2517:             static_cast<RealType>(-3.57375116523338994342e-14),
2518:             static_cast<RealType>(-2.94401318006358820268e-18),
2519:             static_cast<RealType>(-2.73260616170245224789e-23),
2520:         };
2521:         BOOST_MATH_STATIC const RealType Q[7] = {
2522:             static_cast<RealType>(1.),
2523:             static_cast<RealType>(5.41357843707822974161e-3),
2524:             static_cast<RealType>(1.10082540037527566536e-5),
2525:             static_cast<RealType>(1.04338126042963003178e-8),
2526:             static_cast<RealType>(4.63619608458569600346e-12),
2527:             static_cast<RealType>(8.45781310395535984099e-16),
2528:             static_cast<RealType>(4.23432554226506409568e-20),
2529:         };
2530: 
2531:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2532:     }
2533:     else {
2534:         result = -boost::math::numeric_limits<RealType>::infinity();
2535:     }
2536: 
2537:     return result;
2538: }
2539: 
2540: 
2541: template <class RealType>
2542: BOOST_MATH_GPU_ENABLED inline RealType mapairy_quantile_lower_imp_prec(const RealType& p, const boost::math::integral_constant<int, 113>&)
2543: {
2544:     BOOST_MATH_STD_USING
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, if, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。

### Lines 2545-2592 / 第 2545-2592 行
~~~cpp
2545:     RealType result;
2546: 
2547:     if (p >= 0.4375) {
2548:         RealType t = p - static_cast <RealType>(0.4375);
2549: 
2550:         // Rational Approximation
2551:         // Maximum Relative Error: 4.2901e-35
2552:         // LCOV_EXCL_START
2553:         BOOST_MATH_STATIC const RealType P[10] = {
2554:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.48344198262277235851026749871350753173e-1),
2555:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.18249834490570496537675012473572546187e0),
2556:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.20191368895639224466285643454767208281e1),
2557:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.88388953372157636908236843798588258539e1),
2558:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.59477796311326067051769635858472572709e1),
2559:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.88799146700484120781026039104654730797e1),
2560:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.15708831983930955608517858269193800412e1),
2561:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.01389336086567891484877690859385409842e2),
2562:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.16683694881010716925933071465043323946e1),
2563:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.04356966421177683585461937085598186805e1),
2564:         };
2565:         BOOST_MATH_STATIC const RealType Q[11] = {
2566:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2567:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.75444066345435020043849341970820565274e-1),
2568:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.95105673975812427406540024601734210826e0),
2569:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.20381124524894051002242766595737443257e0),
2570:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.48370658634610329590305283520183480026e1),
2571:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.52213602242009530270284305006282822794e-1),
2572:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.91028722773916006242187843372209197705e1),
2573:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.76130245344411748356977700519731978720e0),
2574:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30834721900169773543149860814908904224e1),
2575:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.37863084758381651884340710544840951679e0),
2576:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.46880981703613838666108664771931239970e0),
2577:         };
2578:         // LCOV_EXCL_STOP
2579:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2580:     }
2581:     else if (p >= 0.375) {
2582:         RealType t = p - static_cast <RealType>(0.375);
2583: 
2584:         // Rational Approximation
2585:         // Maximum Relative Error: 2.8433e-36
2586:         // LCOV_EXCL_START
2587:         BOOST_MATH_STATIC const RealType P[11] = {
2588:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.17326074020471664204142312429732771661e0),
2589:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.23412560010002723970559712941124583385e0),
2590:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83665111310407767293290698145068379130e1),
2591:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.38459476870110655357485107373883403534e1),
2592:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.28751995328228442619291346921055105808e1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2593-2640 / 第 2593-2640 行
~~~cpp
2593:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.31663592034507247231393516167247241037e1),
2594:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.13629333446941271397790762651183997586e2),
2595:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.80674058829101054663235662701823250421e1),
2596:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.53226182094253065852552393446365315319e1),
2597:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.14713948941614711932063053969010219677e1),
2598:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.62979741122708118776725634304028246971e0),
2599:         };
2600:         BOOST_MATH_STATIC const RealType Q[10] = {
2601:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2602:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.10550060286464202595779024353437346419e0),
2603:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.15893254630199957990897452211066782021e0),
2604:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.58964066823516762861256609311733069353e1),
2605:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.73352515261971291505497909338586980605e0),
2606:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.64737859211974163695241658186141083513e1),
2607:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.79137714768236053008878088337762178011e0),
2608:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.71851514659301019977259792564627124877e1),
2609:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.37210093190088984630526671624779422232e0),
2610:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06793750951779308425209267821815264457e1),
2611:         };
2612:         // LCOV_EXCL_STOP
2613:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2614:     }
2615:     else if (p >= 0.25) {
2616:         RealType t = p - static_cast <RealType>(0.25);
2617: 
2618:         // Rational Approximation
2619:         // Maximum Relative Error: 5.9072e-35
2620:         // LCOV_EXCL_START
2621:         BOOST_MATH_STATIC const RealType P[13] = {
2622:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.63281240925531315038207673147576291783e0),
2623:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.72733898245766165408685147762489513406e1),
2624:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.48666841594842113608962500631836790675e1),
2625:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.38711336213357101067420572773139678571e1),
2626:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.19536066931882831915715343914510496760e2),
2627:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70911330354860558400876197129777829223e2),
2628:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.46138758869321272507090399082047865434e3),
2629:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.42653825421465476333482312795245170700e2),
2630:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.68040069633027903153088221686431049116e3),
2631:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.63017854949929226947577854802720988740e2),
2632:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.57362168966659376351959631576588023516e2),
2633:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.48386631313725080746815524770260451090e2),
2634:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.03293129698111279047104766073456412318e1),
2635:         };
2636:         BOOST_MATH_STATIC const RealType Q[13] = {
2637:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2638:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.29511778027351594854005887702013466376e1),
2639:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.66155745848864270109281703659789474448e1),
2640:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.25628362783798417463294553777015370203e1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2641-2688 / 第 2641-2688 行
~~~cpp
2641:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.93162726153946899828589402901015679821e2),
2642:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.51582398149308841534372162372276623400e2),
2643:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.55512400116480727630652657714109740448e2),
2644:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.11949742749256615588470329024257669470e2),
2645:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.28090154738508864776480712360731968283e0),
2646:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.44307717248171941824014971579691790721e2),
2647:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.33595130666758203099507440236958725924e-1),
2648:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.76156378002782668186592725145930755636e1),
2649:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.70446647862725980215630194019740606935e0),
2650:         };
2651:         // LCOV_EXCL_STOP
2652:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2653:     }
2654:     else if (p >= 0.125) {
2655:         RealType t = p - static_cast <RealType>(0.125);
2656: 
2657:         // Rational Approximation
2658:         // Maximum Relative Error: 9.9092e-36
2659:         // LCOV_EXCL_START
2660:         BOOST_MATH_STATIC const RealType P[15] = {
2661:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.18765177572396470161180571018467019660e0),
2662:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.16991481696416567893311341049825218287e1),
2663:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.25497491118598918048058751362064598010e3),
2664:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.00259915194411316966036757165146681474e4),
2665:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.20350803730836873687692010728689867756e4),
2666:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.75441278117456011071671644613599089820e4),
2667:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.18844967505497310645091822621081741562e5),
2668:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.84771867850847121528386231811667556346e5),
2669:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.78112436422992766542256241612018834150e5),
2670:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.82617957794395420193751983521804760378e5),
2671:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.77260227244465268981198741998181334875e5),
2672:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.61918290776044518321561351472048170874e5),
2673:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.10309368217936941851272359946015001037e5),
2674:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.68917274690585744147547352309416731690e5),
2675:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.96914697182030973966321601422851730384e4),
2676:         };
2677:         BOOST_MATH_STATIC const RealType Q[15] = {
2678:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2679:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.98063872144867195074924232601423646991e1),
2680:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.65911346382127464683324945513128779971e2),
2681:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.02451223307009464199634546540152067898e3),
2682:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.12662676019712475980273334769644047369e4),
2683:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.77637305574675655673572303462430608857e4),
2684:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.65900204382557635710258095712789133767e4),
2685:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.61649498173261886264315880770449636676e5),
2686:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.29867325788870863753779283018061152414e5),
2687:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.31375646045453788071216808289409712455e4),
2688:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.91053361331987954531162452163243245571e5),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2689-2736 / 第 2689-2736 行
~~~cpp
2689:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.30917504462260061766689326034981496723e5),
2690:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.95779171217851232246427282884386844906e5),
2691:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.07234815204245866330282860014624832711e4),
2692:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21353269292094971546479026200435095695e4),
2693:         };
2694:         // LCOV_EXCL_STOP
2695:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2696:     }
2697:     else if (ilogb(p) >= -4) {
2698:         RealType t = -log2(ldexp(p, 3));
2699: 
2700:         // Rational Approximation
2701:         // Maximum Relative Error: 3.9653e-35
2702:         // LCOV_EXCL_START
2703:         BOOST_MATH_STATIC const RealType P[11] = {
2704:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.18765177572396470161180571018467025793e0),
2705:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.94718878144788678915739777385667044494e0),
2706:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.01760622104142726407095836139719210570e0),
2707:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.27585388152893587017559610649258643106e0),
2708:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.26494992809545184138230791849722703452e-1),
2709:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.19962820888633928632710264415572027960e-2),
2710:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.10249328404135065767844288193594496173e-3),
2711:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.94733898567966142295343935527193851633e-4),
2712:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.10350856810280579594619259121755788797e-6),
2713:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.23852908701349250480831167491889740823e-7),
2714:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.61008160195204632725691076288641221707e-10),
2715:         };
2716:         BOOST_MATH_STATIC const RealType Q[11] = {
2717:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2718:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59109221235949005113322202980300291082e0),
2719:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07119192591092503378838510797916225920e0),
2720:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.97313678065269932508447079892684333156e-1),
2721:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.85743007214453288049750256975889151838e-2),
2722:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21504290861269099964963866156493713716e-2),
2723:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00880286431998922077891394903879512720e-3),
2724:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.78806057460269900288838437267359072282e-5),
2725:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15390284861815831078443996558014864171e-6),
2726:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09877166004503937701692216421704042881e-8),
2727:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.01544823753120969225271131241177003165e-11),
2728:         };
2729:         // LCOV_EXCL_STOP
2730:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2731:     }
2732:     else if (ilogb(p) >= -8) {
2733:         RealType t = -log2(ldexp(p, 4));
2734: 
2735:         // Rational Approximation
2736:         // Maximum Relative Error: 6.7872e-36
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2737-2784 / 第 2737-2784 行
~~~cpp
2737:         // LCOV_EXCL_START
2738:         BOOST_MATH_STATIC const RealType P[15] = {
2739:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.59822399410385083283727681965013517187e0),
2740:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.30676107401101401386206170152508285083e0),
2741:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.14999782004905950712290914501961213222e0),
2742:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.33941786334132569296061539102765384372e0),
2743:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.67220197146826865151515598496049341734e-1),
2744:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.80553562310354708419148501358813792297e-2),
2745:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.69365728863553037992854715314245847166e-3),
2746:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.66571611322125393586164383361858996769e-4),
2747:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.60112572827002965346926427208336150737e-5),
2748:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.82368436189138780270310776927920829805e-6),
2749:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.53112953085778860983110669544602606343e-8),
2750:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.47363651755817041383574210879856850108e-10),
2751:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.56959356177318833325064543662295824581e-12),
2752:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.16259632533790175212174199386945953139e-14),
2753:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.51102540845397821195190063256442894688e-18),
2754:         };
2755:         BOOST_MATH_STATIC const RealType Q[14] = {
2756:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2757:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.51733661576324699382035973518172469602e0),
2758:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.01435607980568082538278883569729476204e0),
2759:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.93274478117803447229185270863587786287e-1),
2760:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.79695020433868416640781960667235896490e-2),
2761:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.64179255575983014759473815411232853821e-2),
2762:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.88411076775875459504324039642698874213e-3),
2763:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47924359965537384942568979646011627522e-4),
2764:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.80915944234873904741224397674033111178e-6),
2765:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.67530059257263142305079790717032648103e-7),
2766:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.59778634436027309520742387952911132163e-9),
2767:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.45772841855129835242992919296397034883e-11),
2768:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.40356170422999016176996652783329671363e-13),
2769:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.31127037096552892520323998665757802862e-16),
2770:         };
2771:         // LCOV_EXCL_STOP
2772:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2773:     }
2774:     else if (ilogb(p) >= -16) {
2775:         RealType t = -log2(ldexp(p, 8));
2776: 
2777:         // Rational Approximation
2778:         // Maximum Relative Error: 7.6679e-36
2779:         // LCOV_EXCL_START
2780:         BOOST_MATH_STATIC const RealType P[15] = {
2781:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.67354365380697578246790709817724831418e0),
2782:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.98681625368564496421038758088088788795e0),
2783:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.07310677200866436332040539417232353673e0),
2784:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.24429836496456823308103613923194387860e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2785-2832 / 第 2785-2832 行
~~~cpp
2785:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.02985399190977938381625172095575017346e-2),
2786:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.76782196972948240235456454778537838123e-3),
2787:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.74449002785643398012450514191731166637e-4),
2788:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.59599894044461264694825875303563328822e-6),
2789:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.25326249969126313897827328136779597159e-7),
2790:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.41714701672521323699602179629851858792e-9),
2791:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.43197925999694667433180053594831915164e-11),
2792:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.84385683045691486021670951412023644809e-13),
2793:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.34484740544060627138216383389282372695e-15),
2794:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.69717198111130468014375331439613690658e-18),
2795:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.54464319402577486444841981479085908190e-22),
2796:         };
2797:         BOOST_MATH_STATIC const RealType Q[14] = {
2798:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2799:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.56568169258142086426383908572004868200e-1),
2800:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.52144703581828715720555168887427064424e-1),
2801:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.87268567039210776554113754014224005739e-2),
2802:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.04907480436107533324385788289629535047e-3),
2803:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.05063218951887755000006493061952858632e-4),
2804:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.88707869862323507236241669797692957827e-5),
2805:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12876335369047582931728838551780783006e-6),
2806:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.96657547014655679104867167083078285517e-8),
2807:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.05799908632250375607393338998205481867e-10),
2808:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.26543514080500125624753383852230043206e-12),
2809:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.02102501711063497529014782040893679505e-14),
2810:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.91577723105757841509716090936343311518e-17),
2811:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.13824283239122976911704652025193325941e-20),
2812:         };
2813:         // LCOV_EXCL_STOP
2814:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2815:     }
2816:     else if (ilogb(p) >= -32) {
2817:         RealType t = -log2(ldexp(p, 16));
2818: 
2819:         // Rational Approximation
2820:         // Maximum Relative Error: 8.6323e-36
2821:         // LCOV_EXCL_START
2822:         BOOST_MATH_STATIC const RealType P[15] = {
2823:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.92187819510636694694450607724165689649e0),
2824:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.97763858433958798529675258052376253402e0),
2825:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.51315096979196319830162238831654165509e-1),
2826:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.63462872759639470195664268077372442947e-2),
2827:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.42843455093529447002457295994721102683e-3),
2828:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.09839699044798405685726233251577671229e-4),
2829:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.42918585783783247934440868680748693033e-6),
2830:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.39927851612709063686969934343256912856e-8),
2831:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.08811978806833318962489621493456773153e-9),
2832:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.05790315329766847040100971840989677130e-11),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2833-2880 / 第 2833-2880 行
~~~cpp
2833:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.45479905512618918078640786598987515012e-14),
2834:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.26725031195260767541308143946590024995e-16),
2835:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.93921283405116349017396651678347306610e-19),
2836:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.46989755992471397407520449698676945629e-22),
2837:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.84098833615882764168840211033822541979e-26),
2838:         };
2839:         BOOST_MATH_STATIC const RealType Q[14] = {
2840:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2841:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.76933186134913044021577076301874622292e-1),
2842:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.25794095712489484166470336696962749356e-2),
2843:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.02367257449622569623375613790000874499e-3),
2844:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.72422948147678152291655497515112236849e-4),
2845:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.54841786738844966378222670550160421679e-5),
2846:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.40690766057741905625149753730480294357e-7),
2847:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.57722740290698689456097239435447030950e-9),
2848:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12202417878861628530322715231540006386e-10),
2849:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.52053248659561670052645118655279630611e-13),
2850:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.93156242508301535729374373870786335203e-15),
2851:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40767922796889118151219837068812449420e-17),
2852:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83472702205100162081157644960354192597e-20),
2853:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.07575352729625387198150555665307193572e-24),
2854:         };
2855:         // LCOV_EXCL_STOP
2856:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2857:     }
2858:     else if (ilogb(p) >= -64) {
2859:         RealType t = -log2(ldexp(p, 32));
2860: 
2861:         // Rational Approximation
2862:         // Maximum Relative Error: 9.8799e-36
2863:         // LCOV_EXCL_START
2864:         BOOST_MATH_STATIC const RealType P[15] = {
2865:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.41443550638291131009585191506467028820e0),
2866:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.27887396494095561461365753577189254063e0),
2867:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.12752633002089885479040650194288302309e-1),
2868:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.79073355729340068320968150408320521772e-3),
2869:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.92104558368762745368896313096467732214e-4),
2870:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.31521786147036353766882145733055166296e-6),
2871:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.69209485282228501578601478546441260206e-8),
2872:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.17440286764020598209076590905417295956e-10),
2873:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.24323354132100896221825450145208350291e-12),
2874:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.53395893728239001663242998169841168859e-14),
2875:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.68737623513761169307963299679882178852e-17),
2876:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.34293243065056704017609034428511365032e-19),
2877:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.16077944943653158589001897848048630079e-22),
2878:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.62195354664281744711336666974567406606e-26),
2879:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.34400562353945663460416286570988365992e-30),
2880:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2881-2928 / 第 2881-2928 行
~~~cpp
2881:         BOOST_MATH_STATIC const RealType Q[14] = {
2882:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2883:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.87838171063584994806998206766890809367e-1),
2884:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.55395225505766120991458653457272783334e-2),
2885:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.45282436421694718640472363162421055686e-4),
2886:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.29570159526914149727970023744213510609e-5),
2887:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.75502365627517415214497786524319814617e-7),
2888:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.74140267916464056408693097635546173776e-9),
2889:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.53554952415602030474322188152855226456e-11),
2890:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.25818606585833092910042975933757268581e-13),
2891:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.79945579370700383986587672831350689541e-15),
2892:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.64188969985517050219881299229805701044e-18),
2893:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.59811800080967790439078895802795103852e-21),
2894:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.28225402720788909349967839966304553864e-24),
2895:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.22313199865592821923485268860178384308e-28),
2896:         };
2897:         // LCOV_EXCL_STOP
2898:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2899:     }
2900:     else if (ilogb(p) >= -128) {
2901:         RealType t = -log2(ldexp(p, 64));
2902: 
2903:         // Rational Approximation
2904:         // Maximum Relative Error: 1.1548e-35
2905:         // LCOV_EXCL_START
2906:         BOOST_MATH_STATIC const RealType P[15] = {
2907:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.23500806363233607692361021471929016922e0),
2908:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.16747938711332885952564885548768072606e-1),
2909:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.58252672740225210061031286151136185818e-2),
2910:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.15477223656783400505701577048140375949e-4),
2911:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.51130817987857130928725357067032472382e-5),
2912:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.68955185187558206711296837951129048907e-7),
2913:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.30418343536824247801373327173028702308e-9),
2914:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.96029044368524575193330776540736319950e-12),
2915:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.53259926294848786440686413056632823519e-14),
2916:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.09476918795964320022985872737468492126e-17),
2917:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.20864569882477440934325776445799604204e-20),
2918:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.01321468510611172635388570487951907905e-23),
2919:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.45079221107904118975166347269173516170e-26),
2920:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.36559452694774774399884349254686988041e-30),
2921:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.92396730553947142987611521115040472261e-35),
2922:         };
2923:         BOOST_MATH_STATIC const RealType Q[14] = {
2924:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2925:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.36599681872296382199486815169747516110e-2),
2926:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.86330544720458446620644055149504593514e-3),
2927:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.23793435123936109978741252388806998743e-5),
2928:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41867848381419339587560909646887411175e-6),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2929-2976 / 第 2929-2976 行
~~~cpp
2929:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.46493580168298395584601073832432583371e-8),
2930:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.03537470107933172406224278121518518287e-10),
2931:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.00375884189083338846655326801486182158e-13),
2932:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62515041281615938158455493618149216047e-15),
2933:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.42337949780187570810208014464208536484e-18),
2934:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.40196890950995648233637422892711654146e-21),
2935:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.11894256193449973803773962108906527772e-24),
2936:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.00900961462911160554915139090711911885e-27),
2937:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.65825716532665817972751320034032284421e-32),
2938:         };
2939:         // LCOV_EXCL_STOP
2940:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2941:     }
2942:     else if (ilogb(p) >= -256) {
2943:         RealType t = -log2(ldexp(p, 128));
2944: 
2945:         // Rational Approximation
2946:         // Maximum Relative Error: 1.3756e-35
2947:         // LCOV_EXCL_START
2948:         BOOST_MATH_STATIC const RealType P[15] = {
2949:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.04845570631944023525776899386112795330e1),
2950:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.18146685146173151383718092529868406030e-1),
2951:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.13255476532916847606354932879190731233e-2),
2952:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.44228173780288838603949849889291143631e-4),
2953:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.18663599891768480607165516401619315227e-6),
2954:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.61193813386433438633008774630150180359e-9),
2955:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.54402603714659392010463991032389692959e-11),
2956:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.76816632514967325885563032378775486543e-14),
2957:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.22774672528068516513970610441705738842e-16),
2958:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.47312348271366325243169398780745416279e-19),
2959:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.10984325972747808970318612951079014854e-22),
2960:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.81620524028936785168005732104270722618e-26),
2961:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.03443227423068771484783389914203726108e-29),
2962:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.02313300749670214384591200940841254958e-34),
2963:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.40321396496046206171642334628524367374e-39),
2964:         };
2965:         BOOST_MATH_STATIC const RealType Q[14] = {
2966:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
2967:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.67292043485384876322219919215413286868e-2),
2968:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.61652550158809553935603664087740554258e-4),
2969:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14722810796821047167211543031044501921e-5),
2970:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.78954660078305461714050086730116257387e-8),
2971:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.52794892087101750452585357544956835504e-10),
2972:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59652255206657812422503741672829368618e-12),
2973:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.84914442937017449248597857220675602148e-15),
2974:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.23662183613814475007146598734598810102e-18),
2975:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.55388618781592901470236982277678753407e-21),
2976:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.20418178834057564300014964843066904024e-24),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 2977-3024 / 第 2977-3024 行
~~~cpp
2977:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.48606639104883413456676877330419513129e-27),
2978:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.39845313960416564778273486179935754019e-31),
2979:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14538443937605324316706211070799970095e-35),
2980:         };
2981:         // LCOV_EXCL_STOP
2982:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
2983:     }
2984:     else if (ilogb(p) >= -512) {
2985:         RealType t = -log2(ldexp(p, 256));
2986: 
2987:         // Rational Approximation
2988:         // Maximum Relative Error: 1.6639e-35
2989:         // LCOV_EXCL_START
2990:         BOOST_MATH_STATIC const RealType P[15] = {
2991:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.32865827226175697711590794217590458484e1),
2992:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.27551166309670994513910580518431041518e-1),
2993:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.57161482253140058637495100797888501265e-3),
2994:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.26908727392152312216118985392395130974e-5),
2995:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.31391169101865809627389212651592902649e-8),
2996:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.58926174475498352884244229017384309804e-10),
2997:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.97074898765380614681225071978849430802e-13),
2998:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.59852103341093122669197704225736036199e-16),
2999:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.97287954178083606552531325613580819555e-19),
3000:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.57634773176875526612407357244997035312e-22),
3001:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.34467233210713881817055138794482883359e-25),
3002:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.91236170873875898506577053309622472122e-29),
3003:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.12220990075567880730037575497818287435e-33),
3004:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.20881889651487527801970182542596258873e-37),
3005:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.56848486878078288956741060120464349537e-43),
3006:         };
3007:         BOOST_MATH_STATIC const RealType Q[14] = {
3008:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3009:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.33267370502089423930888060969568705647e-2),
3010:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.39632352029900752622967578086289898150e-4),
3011:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.42703739831525305516280300008439396218e-6),
3012:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.45767617531685380458878368024246654652e-9),
3013:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40344962756110545138002101382437142038e-11),
3014:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.47015258371290492450093115369080460499e-14),
3015:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.97280918936580227687603348219414768787e-17),
3016:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.40441024286579579491205384492088325576e-20),
3017:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.26130000914236204012152918399995098882e-23),
3018:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.03893990417552709151955156348527062863e-27),
3019:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.12687002255767114781771099969545907763e-31),
3020:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.74218645918961186861014420578277888513e-35),
3021:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.36897549168687040570349702061165281706e-39),
3022:         };
3023:         // LCOV_EXCL_STOP
3024:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3025-3072 / 第 3025-3072 行
~~~cpp
3025:     }
3026:     else if (ilogb(p) >= -1024) {
3027:         RealType t = -log2(ldexp(p, 512));
3028: 
3029:         // Rational Approximation
3030:         // Maximum Relative Error: 2.0360e-35
3031:         // LCOV_EXCL_START
3032:         BOOST_MATH_STATIC const RealType P[15] = {
3033:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.67937186583822375017526293948703697225e1),
3034:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.06681764321003187068904973985967908140e-1),
3035:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.12508887240195683379004033347903251977e-3),
3036:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.56847823157999127998977939588643284176e-6),
3037:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.31281869105767454049413701029676766275e-9),
3038:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.01498951390503036399081209706853095793e-11),
3039:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.72866867304007090391517734634589972858e-15),
3040:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.44819806993104486828983054294866921869e-18),
3041:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.91441365806306460165885645136864045231e-21),
3042:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.71364347097027340365042558634044496149e-25),
3043:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.63601648491144929836375956218857970640e-28),
3044:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.76948956673676441236280803645598939735e-32),
3045:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.47367651137203634311843318915161504046e-37),
3046:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.83187517541957887917067558455828915184e-41),
3047:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.21480186561579326423946788448005430367e-47),
3048:         };
3049:         BOOST_MATH_STATIC const RealType Q[14] = {
3050:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3051:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16494591376838053609854716130343599036e-2),
3052:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.97651667629616309497454026431358820357e-5),
3053:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.77741398674456235952879526959641925087e-7),
3054:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.39478109667902532743651043316724748827e-10),
3055:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.35965905056304225411108295866332882930e-13),
3056:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.83206524996481183422082802793852630990e-16),
3057:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30320324476590103123012385840054658401e-19),
3058:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.30322948189714718819437477682869360798e-23),
3059:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.43729790298535728717477691270336818161e-26),
3060:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.89788697114251966298674871919685298106e-30),
3061:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.43510901856942238937717065880365530871e-34),
3062:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.38232043389918216652459244727737381677e-38),
3063:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.64599465785019268214108345671361994702e-43),
3064:         };
3065:         // LCOV_EXCL_STOP
3066:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3067:     }
3068:     else if (ilogb(p) >= -2048) {
3069:         RealType t = -log2(ldexp(p, 1024));
3070: 
3071:         // Rational Approximation
3072:         // Maximum Relative Error: 2.5130e-35
~~~
- **EN:** This range declares or defines callable logic such as if, log2, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 if, log2, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3073-3120 / 第 3073-3120 行
~~~cpp
3073:         // LCOV_EXCL_START
3074:         BOOST_MATH_STATIC const RealType P[15] = {
3075:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.11959316095291435774375635827672517008e1),
3076:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.30292575371366023255165927527306483022e-1),
3077:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.54260157439166096303943109715675142318e-4),
3078:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.61232819199170639867079290977704351939e-7),
3079:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.74481972848503486840161528924694379655e-10),
3080:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.98283577906243441829434029827766571263e-13),
3081:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.90691715740583828769850056130458574520e-16),
3082:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.31354728925505346732804698899977180508e-20),
3083:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.42545288372836698650371589645832759416e-23),
3084:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.12901629676328680681102537492164204387e-27),
3085:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.99690040253176100731314099573187027309e-31),
3086:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.07899506956133955785140496937520311210e-35),
3087:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.88607240095256436460507438213387199067e-40),
3088:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.78815984551154095621830792130401294111e-45),
3089:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.20516030880916148179297554212609531432e-51),
3090:         };
3091:         BOOST_MATH_STATIC const RealType Q[14] = {
3092:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3093:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.81973541224235020744673910266545976833e-3),
3094:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49156990107280109344880219729275939242e-5),
3095:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.21606730563542176411852745162267260946e-8),
3096:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.11449329529433741944366607648360521674e-11),
3097:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.35659138507940819452801802756409587220e-14),
3098:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.95708516718550485872934856595725983907e-18),
3099:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.78871290972777009292563576533612002908e-21),
3100:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.60955318960542258732596447917271198307e-25),
3101:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.72437581728117963125690670426402194936e-29),
3102:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.77473938855958487119889840032590783232e-33),
3103:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.66198494713809467076392278745811981500e-37),
3104:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.34142897042941614778352280692901008538e-42),
3105:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.98791658647635156162347063765388728959e-47),
3106:         };
3107:         // LCOV_EXCL_STOP
3108:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3109:     }
3110:     else if (ilogb(p) >= -4096) {
3111:         RealType t = -log2(ldexp(p, 2048));
3112: 
3113:         // Rational Approximation
3114:         // Maximum Relative Error: 3.1220e-35
3115:         // LCOV_EXCL_START
3116:         BOOST_MATH_STATIC const RealType P[15] = {
3117:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.67307564006689676593687414536012112755e1),
3118:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.21005690250741024744367516466433711478e-2),
3119:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.11537345869365655126739041291119096558e-4),
3120:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.82910297156061391001507891422501792453e-8),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3121-3168 / 第 3121-3168 行
~~~cpp
3121:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.51574933516708249049824513935386420692e-11),
3122:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.56433757960363802088718489136097249753e-14),
3123:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.74248235865301086829849817739500215149e-18),
3124:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.19151506367084295119369434315371762091e-22),
3125:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.98518959000360170320183116510466814569e-26),
3126:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.21330422702314763225472001861559380186e-30),
3127:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.44346922987964428874014866468161821471e-34),
3128:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.59773305152191273570416120169527607421e-39),
3129:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.81894412321532723356954669501665983316e-44),
3130:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.25766346856943928908756472385992861288e-49),
3131:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.44601448994095786447982489957909713982e-55),
3132:         };
3133:         BOOST_MATH_STATIC const RealType Q[14] = {
3134:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3135:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.90814053669730896497462224007523900520e-3),
3136:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.72450689508305973756255440356759005330e-6),
3137:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.76517273086984384225845151573287252506e-9),
3138:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31844251885317815627707511621078762352e-12),
3139:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.22689324865113257769413663010725436393e-16),
3140:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.27524826460469866934006001123700331335e-20),
3141:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.39172850948322201614266822896191911031e-23),
3142:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.40343613937272428197414545004329993769e-27),
3143:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.17936773028976355507339458927541970545e-32),
3144:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.66512598049860260933817550698863263184e-36),
3145:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.06432209670481882442649684139775366719e-41),
3146:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.10247886333916820534393624270217678968e-46),
3147:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.40899321122058714028548211810431871877e-51),
3148:         };
3149:         // LCOV_EXCL_STOP
3150:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3151:     }
3152:     else if (ilogb(p) >= -8192) {
3153:         RealType t = -log2(ldexp(p, 4096));
3154: 
3155:         // Rational Approximation
3156:         // Maximum Relative Error: 3.8974e-35
3157:         // LCOV_EXCL_START
3158:         BOOST_MATH_STATIC const RealType P[15] = {
3159:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.36960987939726803544369406181770745475e1),
3160:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.17239311695985070524235502979761682692e-2),
3161:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.51192208811996535244435318068035492922e-5),
3162:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.38938553034896173195617671475670860841e-8),
3163:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.55156910900732478717648524688116855303e-12),
3164:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.14905040433940475292279950923000597280e-16),
3165:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -7.35237464492052939771487320880614968639e-20),
3166:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.07937000518607459141766382199896010484e-24),
3167:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.42797358100745086706362563988598447929e-28),
3168:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.27871637324128856529004325499921407260e-32),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3169-3216 / 第 3169-3216 行
~~~cpp
3169:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.99553669724530906250814559570819049401e-37),
3170:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.04272644552406682186928100080598582627e-42),
3171:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.70093315570856725077212325128817808000e-47),
3172:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.51767319872105260145583037426067406953e-53),
3173:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.87159268409640967747617639113346310759e-59),
3174:         };
3175:         BOOST_MATH_STATIC const RealType Q[14] = {
3176:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3177:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.45350026842128595165328480395513258721e-3),
3178:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.30398532102631290226106936127181928207e-7),
3179:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.45242264812189519858570105609209495630e-10),
3180:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.22745462510042159972414219082495434039e-14),
3181:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31834645038348794443252730265421155969e-17),
3182:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44590392528760847619123404904356730177e-21),
3183:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.08436623062305193311891193246627599030e-25),
3184:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.46542554766048266351202730449796918707e-30),
3185:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.78672067064478133389628198943161640913e-34),
3186:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.56573050761582685018467077197376031818e-39),
3187:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.92145137276530136848088270840255715047e-44),
3188:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.96933863894082533505471662180541379922e-49),
3189:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.92661972206232945959915223259585457082e-55),
3190:         };
3191:         // LCOV_EXCL_STOP
3192:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3193:     }
3194:     else if (ilogb(p) >= -16384) {
3195:         RealType t = -log2(ldexp(p, 8192));
3196: 
3197:         // Rational Approximation
3198:         // Maximum Relative Error: 4.8819e-35
3199:         // LCOV_EXCL_START
3200:         BOOST_MATH_STATIC const RealType P[15] = {
3201:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.24662793339079714510108682543625432532e1),
3202:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.25841960642102016210295419419373971750e-2),
3203:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.10589156998251704634852108689102850747e-5),
3204:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.18697614924486382142056819421294206504e-9),
3205:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.79445222262445726654186491785652765635e-13),
3206:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.41847338407338901513049755299049551186e-17),
3207:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.44550500540299259432401029904726959214e-21),
3208:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.97463434518480676079167684683604645092e-26),
3209:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.68404349202062958045327516688040625516e-30),
3210:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.14018837476359778654965300153810397742e-35),
3211:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.67726222606571327724434861967972555751e-40),
3212:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.48082398191886705229604237754446294033e-45),
3213:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.28534456209055262678153908192583037946e-51),
3214:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.99466700145428173772768099494881455874e-57),
3215:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.43473066278196981345209422626769148425e-63),
3216:         };
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3217-3264 / 第 3217-3264 行
~~~cpp
3217:         BOOST_MATH_STATIC const RealType Q[14] = {
3218:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3219:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.26570199429958856038191879713341034013e-4),
3220:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.32484776300757286079244074394356908390e-7),
3221:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.31234182027812869096733088981702059020e-11),
3222:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.13711443044675425837293030288097468867e-15),
3223:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.11480036828082409994688474687120865023e-19),
3224:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.25592803132287127389756949487347562847e-23),
3225:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.45726006695535760451195102271978072855e-28),
3226:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.13082170504731110487003517418453709982e-32),
3227:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.48217827031663836930337143509338210426e-37),
3228:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.47389053144555736191304002865419453269e-42),
3229:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.90980968229123572201281013063229644814e-47),
3230:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.79448544326289688123648457587797649323e-53),
3231:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.56179793347045575604935927245529360950e-59),
3232:         };
3233:         // LCOV_EXCL_STOP
3234:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3235:     }
3236:     else {
3237:         result = -boost::math::numeric_limits<RealType>::infinity();
3238:     }
3239: 
3240:     return result;
3241: }
3242: 
3243: template <class RealType>
3244: BOOST_MATH_GPU_ENABLED inline RealType mapairy_quantile_upper_imp_prec(const RealType& p, const boost::math::integral_constant<int, 53>&)
3245: {
3246:     BOOST_MATH_STD_USING
3247:     RealType result;
3248: 
3249:     if (p >= 0.25) {
3250:         RealType t = p - static_cast <RealType>(0.25);
3251: 
3252:         // Rational Approximation
3253:         // Maximum Absolute Error: 1.8559e-18
3254:         BOOST_MATH_STATIC const RealType P[8] = {
3255:             static_cast<RealType>(4.81512108276093785320e-1),
3256:             static_cast<RealType>(-2.74296316128959647914e0),
3257:             static_cast<RealType>(-3.29973875964825685757e1),
3258:             static_cast<RealType>(-4.87536980816224603581e1),
3259:             static_cast<RealType>(8.22233203036734027999e1),
3260:             static_cast<RealType>(1.21654607908452130093e2),
3261:             static_cast<RealType>(-6.66681853240657307279e1),
3262:             static_cast<RealType>(-4.28101952511581488588e1),
3263:         };
3264:         BOOST_MATH_STATIC const RealType Q[10] = {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, infinity.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, infinity。

### Lines 3265-3312 / 第 3265-3312 行
~~~cpp
3265:             static_cast<RealType>(1.),
3266:             static_cast<RealType>(8.20189490825315245036e0),
3267:             static_cast<RealType>(1.63469912146101848441e1),
3268:             static_cast<RealType>(-1.52740920318273920072e1),
3269:             static_cast<RealType>(-5.41684560257839409762e1),
3270:             static_cast<RealType>(6.51733677169299416471e0),
3271:             static_cast<RealType>(3.93092001388102589237e1),
3272:             static_cast<RealType>(-9.59983666140749481195e-1),
3273:             static_cast<RealType>(-9.95648827557655863699e-1),
3274:             static_cast<RealType>(-1.32007124426778083829e0),
3275:         };
3276: 
3277:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3278:     }
3279:     else if (p >= 0.125) {
3280:         RealType t = p - static_cast <RealType>(0.125);
3281: 
3282:         // Rational Approximation
3283:         // Maximum Absolute Error: 4.6019e-17
3284:         BOOST_MATH_STATIC const RealType P[7] = {
3285:             static_cast<RealType>(1.70276979914029733585e0),
3286:             static_cast<RealType>(2.09991992116646276165e1),
3287:             static_cast<RealType>(2.26775403775298867998e1),
3288:             static_cast<RealType>(-4.85384304722129472833e2),
3289:             static_cast<RealType>(-1.47107146466495573999e3),
3290:             static_cast<RealType>(-7.08748473959943943929e1),
3291:             static_cast<RealType>(1.54245210917147215257e3),
3292:         };
3293:         BOOST_MATH_STATIC const RealType Q[8] = {
3294:             static_cast<RealType>(1.),
3295:             static_cast<RealType>(2.13092357122115486375e1),
3296:             static_cast<RealType>(1.57318281834689144053e2),
3297:             static_cast<RealType>(4.42261730187813035957e2),
3298:             static_cast<RealType>(2.10814431586717588454e2),
3299:             static_cast<RealType>(-6.36700983439599552504e2),
3300:             static_cast<RealType>(-2.82923881266630617596e2),
3301:             static_cast<RealType>(1.36613971025062750340e2),
3302:         };
3303: 
3304:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3305:     }
3306:     else if (ilogb(p) >= -4) {
3307:         RealType t = -log2(ldexp(p, 3));
3308: 
3309:         // Rational Approximation
3310:         // Maximum Relative Error: 1.2193e-19
3311:         BOOST_MATH_STATIC const RealType P[8] = {
3312:             static_cast<RealType>(4.25692449785074345588e-1),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3313-3360 / 第 3313-3360 行
~~~cpp
3313:             static_cast<RealType>(3.10963501706596356267e-1),
3314:             static_cast<RealType>(2.91357806215297069863e-2),
3315:             static_cast<RealType>(2.34716342676849303244e-2),
3316:             static_cast<RealType>(5.83137296293361915583e-3),
3317:             static_cast<RealType>(3.71792415497884868748e-4),
3318:             static_cast<RealType>(1.59538372221030642757e-4),
3319:             static_cast<RealType>(4.74040834029330213692e-6),
3320:         };
3321:         BOOST_MATH_STATIC const RealType Q[8] = {
3322:             static_cast<RealType>(1.),
3323:             static_cast<RealType>(4.14801234415100707213e-1),
3324:             static_cast<RealType>(1.04693730144480856638e-1),
3325:             static_cast<RealType>(3.81581484862997435076e-2),
3326:             static_cast<RealType>(8.95334009127358617362e-3),
3327:             static_cast<RealType>(1.43316686981760147226e-3),
3328:             static_cast<RealType>(1.81367766024620080990e-4),
3329:             static_cast<RealType>(1.54779999748286671973e-5),
3330:         };
3331: 
3332:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3333:     }
3334:     else if (ilogb(p) >= -8) {
3335:         RealType t = -log2(ldexp(p, 4));
3336: 
3337:         // Rational Approximation
3338:         // Maximum Relative Error: 4.4418e-17
3339:         BOOST_MATH_STATIC const RealType P[11] = {
3340:             static_cast<RealType>(5.07341098045260541890e-1),
3341:             static_cast<RealType>(3.11771145411143166935e-1),
3342:             static_cast<RealType>(1.74515601081894060888e-1),
3343:             static_cast<RealType>(8.46576990174024231338e-2),
3344:             static_cast<RealType>(2.57510090204322149315e-2),
3345:             static_cast<RealType>(8.26605326867021684811e-3),
3346:             static_cast<RealType>(1.73081423934722046819e-3),
3347:             static_cast<RealType>(3.36314161099011673569e-4),
3348:             static_cast<RealType>(4.50990441180388912803e-5),
3349:             static_cast<RealType>(4.53513191985642134268e-6),
3350:             static_cast<RealType>(2.62304611053075404923e-7),
3351:         };
3352:         BOOST_MATH_STATIC const RealType Q[11] = {
3353:             static_cast<RealType>(1.),
3354:             static_cast<RealType>(5.28225379952156944029e-1),
3355:             static_cast<RealType>(3.49662079845715371907e-1),
3356:             static_cast<RealType>(1.45408903426879603625e-1),
3357:             static_cast<RealType>(5.06773501409016231879e-2),
3358:             static_cast<RealType>(1.45385556714043243731e-2),
3359:             static_cast<RealType>(3.31235831325018043744e-3),
3360:             static_cast<RealType>(6.06977554525543056050e-4),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3361-3408 / 第 3361-3408 行
~~~cpp
3361:             static_cast<RealType>(8.42406730405209749492e-5),
3362:             static_cast<RealType>(8.32337989541696717905e-6),
3363:             static_cast<RealType>(4.84923196546857128337e-7),
3364:         };
3365: 
3366:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3367:     }
3368:     else if (ilogb(p) >= -16) {
3369:         RealType t = -log2(ldexp(p, 8));
3370: 
3371:         // Rational Approximation
3372:         // Maximum Relative Error: 5.7932e-17
3373:         BOOST_MATH_STATIC const RealType P[10] = {
3374:             static_cast<RealType>(5.41774626094491510395e-1),
3375:             static_cast<RealType>(4.11060141334529017898e-1),
3376:             static_cast<RealType>(1.48195601801946264526e-1),
3377:             static_cast<RealType>(3.33881552814492855873e-2),
3378:             static_cast<RealType>(5.20893974732203890418e-3),
3379:             static_cast<RealType>(5.84734765774178832854e-4),
3380:             static_cast<RealType>(4.71028150898133935445e-5),
3381:             static_cast<RealType>(2.59185739450631464618e-6),
3382:             static_cast<RealType>(7.77428184258777394627e-8),
3383:             static_cast<RealType>(2.51255632629650930196e-14),
3384:         };
3385:         BOOST_MATH_STATIC const RealType Q[9] = {
3386:             static_cast<RealType>(1.),
3387:             static_cast<RealType>(7.58341767924960527280e-1),
3388:             static_cast<RealType>(2.73511775500642961539e-1),
3389:             static_cast<RealType>(6.16011987856129890130e-2),
3390:             static_cast<RealType>(9.61296002312356116021e-3),
3391:             static_cast<RealType>(1.07890675777726076554e-3),
3392:             static_cast<RealType>(8.69223632953458271977e-5),
3393:             static_cast<RealType>(4.78248875031756169279e-6),
3394:             static_cast<RealType>(1.43460852065144859304e-7),
3395:         };
3396: 
3397:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3398:     }
3399:     else if (ilogb(p) >= -32) {
3400:         RealType t = -log2(ldexp(p, 16));
3401: 
3402:         // Rational Approximation
3403:         // Maximum Relative Error: 9.0396e-17
3404:         BOOST_MATH_STATIC const RealType P[9] = {
3405:             static_cast<RealType>(5.41926067826974905066e-1),
3406:             static_cast<RealType>(4.86926556246548518715e-1),
3407:             static_cast<RealType>(2.11963908288176005856e-1),
3408:             static_cast<RealType>(5.92200639925655576883e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3409-3456 / 第 3409-3456 行
~~~cpp
3409:             static_cast<RealType>(1.18859816815542567438e-2),
3410:             static_cast<RealType>(1.76833662992855443754e-3),
3411:             static_cast<RealType>(2.21226152157950219596e-4),
3412:             static_cast<RealType>(1.50444847316426133872e-5),
3413:             static_cast<RealType>(1.87458213915373906356e-6),
3414:         };
3415:         BOOST_MATH_STATIC const RealType Q[9] = {
3416:             static_cast<RealType>(1.),
3417:             static_cast<RealType>(8.98511036742503939380e-1),
3418:             static_cast<RealType>(3.91130673008184655152e-1),
3419:             static_cast<RealType>(1.09277016228474605069e-1),
3420:             static_cast<RealType>(2.19328471889880028208e-2),
3421:             static_cast<RealType>(3.26305879571349016107e-3),
3422:             static_cast<RealType>(4.08222014684743492069e-4),
3423:             static_cast<RealType>(2.77611385768697969181e-5),
3424:             static_cast<RealType>(3.45911046256304795257e-6),
3425:         };
3426: 
3427:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3428:     }
3429:     else {
3430:         RealType p_square = p * p;
3431: 
3432:         if ((boost::math::isnormal)(p_square)) {
3433:             result = 1 / cbrt(p_square * constants::two_pi<RealType>());
3434:         }
3435:         else if (p > 0) {
3436:             result = 1 / (cbrt(p) * cbrt(p) * cbrt(constants::two_pi<RealType>()));
3437:         }
3438:         else {
3439:             result = boost::math::numeric_limits<RealType>::infinity();
3440:         }
3441:     }
3442: 
3443:     return result;
3444: }
3445: 
3446: 
3447: template <class RealType>
3448: BOOST_MATH_GPU_ENABLED inline RealType mapairy_quantile_upper_imp_prec(const RealType& p, const boost::math::integral_constant<int, 113>&)
3449: {
3450:     BOOST_MATH_STD_USING
3451:     RealType result;
3452: 
3453:     if (p >= 0.375) {
3454:         RealType t = p - static_cast <RealType>(0.375);
3455: 
3456:         // Rational Approximation
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as tools::evaluate_polynomial, cbrt, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, cbrt, ...。

### Lines 3457-3504 / 第 3457-3504 行
~~~cpp
3457:         // Maximum Absolute Error: 4.0835e-35
3458:         // LCOV_EXCL_START
3459:         BOOST_MATH_STATIC const RealType P[13] = {
3460:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.00474815142578902619056852805926666121e-1),
3461:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.56422290947427848191079775267512708223e0),
3462:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.70103710180837859003070678080056933649e1),
3463:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.08521918131449191445864593768320217287e1),
3464:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.29340655781369686013042530147130581054e2),
3465:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.24198237124638368989049118891909723118e1),
3466:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.43382878809828906953609389440800537385e2),
3467:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.45564809127564867825118566276365267035e1),
3468:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.75881247317499884393790698530115428373e2),
3469:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.55845932095942777602241134226597158364e1),
3470:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.41328261385867825781522154621962338450e2),
3471:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.06758225510372847658316203115073730186e1),
3472:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.10895417312529385966062255102265009972e0),
3473:         };
3474:         BOOST_MATH_STATIC const RealType Q[12] = {
3475:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3476:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.88252553879196710256650370298744093367e0),
3477:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.54875259600848880869571364891152935969e0),
3478:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.78589587338618424770295921221996471887e1),
3479:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.15356831947775532414727361010652423453e1),
3480:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12951532118504570745988981200579372124e2),
3481:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.48163841544376327168780999614703092433e1),
3482:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.56786609618056303930232548304847911521e2),
3483:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.25610739352108840474197350343978451729e1),
3484:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.27063786175330237448255839666252978603e1),
3485:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.11941093895004369510720986032269722254e0),
3486:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.51487618026728514833542002963603231101e1),
3487:         };
3488:         // LCOV_EXCL_STOP
3489:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3490:     }
3491:     else if (p >= 0.25) {
3492:         RealType t = p - static_cast <RealType>(0.25);
3493: 
3494:         // Rational Approximation
3495:         // Maximum Absolute Error: 5.7633e-36
3496:         // LCOV_EXCL_START
3497:         BOOST_MATH_STATIC const RealType P[14] = {
3498:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.81512108276093787175849069715334402323e-1),
3499:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24417080443497141096829831516758083481e0),
3500:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.67006165991083501886186268944009973084e1),
3501:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.74402382755828993223083868408545308340e2),
3502:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.49182541725192134610277727922493871787e2),
3503:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.67273564707254788337557775618297381267e1),
3504:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.73476432616329813096120568871900178919e3),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3505-3552 / 第 3505-3552 行
~~~cpp
3505:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.31235376166262024838125198332476698090e3),
3506:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.59379285677781413393733801325840617522e3),
3507:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.38151434050794836595564739176884302539e3),
3508:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.33534676810383673962443893459127818078e3),
3509:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.38110822236764293910895765875742805411e3),
3510:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.42750073722992463087082849671338957023e2),
3511:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.54255748148299874514839812717054396793e2),
3512:         };
3513:         BOOST_MATH_STATIC const RealType Q[13] = {
3514:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3515:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.64823387375875361292425741663822893626e1),
3516:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.02973633484731117050245517938177308809e2),
3517:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.71288209768693917630236009171518272534e2),
3518:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.23837527610546426062625864735895938014e1),
3519:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.05056816585729983223036277071927165555e3),
3520:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.48087477651935811184913947280572029967e3),
3521:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04631058325147527913398256133791276127e3),
3522:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.69813394441679590721342220435891453447e3),
3523:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.92323371456465893290687995174952942311e2),
3524:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.68542430563281320943284015587559056621e3),
3525:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -4.17969051793607842221356465819951568080e1),
3526:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.82773308760283383020168853159163391394e2),
3527:         };
3528:         // LCOV_EXCL_STOP
3529:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3530:     }
3531:     else if (p >= 0.125) {
3532:         RealType t = p - static_cast <RealType>(0.125);
3533: 
3534:         // Rational Approximation
3535:         // Maximum Absolute Error: 2.1140e-36
3536:         // LCOV_EXCL_START
3537:         BOOST_MATH_STATIC const RealType P[15] = {
3538:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70276979914029738186601698003670175907e0),
3539:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.63126762626382548478172664328434577553e1),
3540:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04190225271045202674546813475341133174e3),
3541:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.93523974140998850492859698545966806498e3),
3542:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.19814006186501010136822066747124777014e4),
3543:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.55931423620290859807616748030589502039e4),
3544:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.78874021192395317496507459296221703565e5),
3545:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.03860533237347587977439662522389465152e6),
3546:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -2.77882648875352690605815508748162607271e6),
3547:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.05498612167816258406694194925933958145e6),
3548:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.05326361485692298778330190198630232666e7),
3549:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.85827791876754731187453265804790139032e5),
3550:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -8.93719378006868242377955041137674308589e6),
3551:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -5.56839957539576784391036362196229047625e5),
3552:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.95604329277359828898502487252547842378e6),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3553-3600 / 第 3553-3600 行
~~~cpp
3553:         };
3554:         BOOST_MATH_STATIC const RealType Q[16] = {
3555:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3556:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.79208640567193066236912382037923299779e1),
3557:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.94775812217734059201656828286490832145e2),
3558:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16467934643564936346029555887148320030e4),
3559:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.35525720248600096849901920839060920346e4),
3560:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.69760913594243328874861534307039589127e5),
3561:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.32330501005950982838953061458838040612e5),
3562:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.79610639577090112327353399739315606205e5),
3563:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.43314292923292828425630915931385776182e6),
3564:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.97538885038058371436244702169375622661e6),
3565:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -9.48431896958634429210349441846613832681e5),
3566:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.93459449030820736960297236799012798749e6),
3567:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.67200014823529787381847745962773726408e6),
3568:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.37035571075060153491151970623824940994e6),
3569:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -3.22682822001329636071591164177026394518e5),
3570:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.09781406768816062486819491582960840983e4),
3571:         };
3572:         // LCOV_EXCL_STOP
3573:         result = tools::evaluate_polynomial(P, t) / tools::evaluate_polynomial(Q, t);
3574:     }
3575:     else if (ilogb(p) >= -4) {
3576:         RealType t = -log2(ldexp(p, 3));
3577: 
3578:         // Rational Approximation
3579:         // Maximum Relative Error: 1.1409e-35
3580:         // LCOV_EXCL_START
3581:         BOOST_MATH_STATIC const RealType P[14] = {
3582:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.25692449785074345466504245009175450649e-1),
3583:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.75679137667345136118441108839649360362e-1),
3584:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06171803174020856964914440692439080669e-2),
3585:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.87798066278592051163038122952593080648e-2),
3586:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.20070543183347459409303407166630392077e-2),
3587:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.13457391270614708627745403376469848816e-4),
3588:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06743974464224003715510181633693539914e-3),
3589:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16870984737226212814217822779976770316e-4),
3590:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.21845093091651861426944931268861694026e-5),
3591:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.85357146081877929591916782097540632519e-6),
3592:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -6.19085800299127898508052519062782284785e-8),
3593:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.41985644250494046067095909812634573318e-7),
3594:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.30857042700765443668305406695750760693e-9),
3595:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.10466412567107519640190849286913680449e-10),
3596:         };
3597:         BOOST_MATH_STATIC const RealType Q[14] = {
3598:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3599:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.31914248618040435028023418981527961171e-1),
3600:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.73578090645412656850163531828709850171e-2),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3601-3648 / 第 3601-3648 行
~~~cpp
3601:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.57329813782272411333511950903192234311e-2),
3602:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.62736127875896578315177123764520823372e-2),
3603:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.76809643836078823237530990091078867553e-3),
3604:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.32026948719622983920194944841520771986e-3),
3605:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.45051018027743807545734050620973716634e-4),
3606:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.58281707210621813556068724127478674938e-5),
3607:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.63884527227517358294732620995363921547e-6),
3608:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15973602356223075515067915930205826229e-6),
3609:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.35069439950884795002182517078104942615e-7),
3610:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.15454119109586223908613596754794988609e-8),
3611:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.55273685376557721039847456564342945576e-10),
3612:         };
3613:         // LCOV_EXCL_STOP
3614:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3615:     }
3616:     else if (ilogb(p) >= -8) {
3617:         RealType t = -log2(ldexp(p, 4));
3618: 
3619:         // Rational Approximation
3620:         // Maximum Relative Error: 1.2521e-35
3621:         // LCOV_EXCL_START
3622:         BOOST_MATH_STATIC const RealType P[23] = {
3623:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.07341098045260497471001948654506267614e-1),
3624:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.16518383383878659278973043343250842753e-1),
3625:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.10029094208424121908983949243560936013e-1),
3626:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.04771840726172284780129819470963100749e-2),
3627:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.34173170868011689830672637082451998700e-2),
3628:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.41990262178664512140746911398264330173e-2),
3629:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.06779488545758366708787010705581103705e-3),
3630:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.41892665233583725631482443019441608726e-3),
3631:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.20692306716979208762785454648538891867e-4),
3632:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.11097906809673639231336894729060830995e-4),
3633:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.37476591232600886363441107536706973169e-5),
3634:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.02659053066396720145189153810309784416e-5),
3635:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.02209877191642023279303996697953314344e-6),
3636:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.56663781532392665205516573323950583901e-7),
3637:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.95655734237060800145227277584749429063e-7),
3638:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.06357695252098035545383649954315685077e-8),
3639:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.78759045059235560356343893064681290047e-9),
3640:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.95881339136963512103591745337914059651e-10),
3641:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.70156441275519927563064848389865812060e-11),
3642:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.99745225746277063516394774908346367811e-12),
3643:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.45718440382347867317547921045052714102e-13),
3644:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.39027665085346558512961348663034579801e-14),
3645:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.05739751797738770096482688062542436470e-15),
3646:         };
3647:         BOOST_MATH_STATIC const RealType Q[23] = {
3648:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3649-3696 / 第 3649-3696 行
~~~cpp
3649:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.43370372582239919321785765900615222895e-1),
3650:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.52872159582703775260145036441128318159e-1),
3651:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.28243735290178057451806192890274584778e-1),
3652:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.93375398009812888642212045868197435998e-2),
3653:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.73364866677217419593129631900708646445e-2),
3654:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.53645928499107852437053167521160449434e-2),
3655:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.74280939589407863107682593092148442428e-3),
3656:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.80095449855178765594835180574448729793e-3),
3657:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.65924845456946706158946250220103271334e-4),
3658:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.52170861715436344002253767944763106994e-4),
3659:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.87246437551620484806338690322735878649e-5),
3660:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.88631873230311653853089809596759382095e-6),
3661:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.84478812152918182782333415475103623486e-6),
3662:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.47998768403859674841488325856607782853e-7),
3663:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.82364683269852480160620586102339743788e-8),
3664:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.65854316058742127585142691993199177898e-9),
3665:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11358340340462071552670838135645042498e-9),
3666:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.22818744671190957896035448856159685984e-10),
3667:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.11038729491846772238262374112315536796e-11),
3668:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.21355801166652957655438257794658921155e-13),
3669:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.41278271853370874105923461404291742454e-14),
3670:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.95100373579692323015092323646110838623e-15),
3671:         };
3672:         // LCOV_EXCL_STOP
3673:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3674:     }
3675:     else if (ilogb(p) >= -16) {
3676:         RealType t = -log2(ldexp(p, 8));
3677: 
3678:         // Rational Approximation
3679:         // Maximum Relative Error: 2.0703e-35
3680:         // LCOV_EXCL_START
3681:         BOOST_MATH_STATIC const RealType P[21] = {
3682:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.41774626094491452462664949805613444094e-1),
3683:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.96383089261273022706449773421031102175e-1),
3684:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.16315295073029174376617863024082371446e-1),
3685:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.65377894193914426949840018839915119410e-1),
3686:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.33210993830236821503160637845009556016e-1),
3687:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.69315463529653886947182738378630780083e-2),
3688:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.09869947341518160436616160018702590834e-3),
3689:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44331691052908906654005398143769791881e-3),
3690:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.13131413925652085071882765653750661678e-4),
3691:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.64441840437413591336927030249538399459e-5),
3692:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.78393581596372725434038621824715039765e-6),
3693:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.50239319821178575427758224587858938204e-7),
3694:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.92619647697287767235953207451871137149e-8),
3695:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.26901081456833267780600560830367533351e-9),
3696:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.12151768312254597726918329997945574766e-11),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3697-3744 / 第 3697-3744 行
~~~cpp
3697:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.36907507996686107513673694597817437197e-12),
3698:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31699373909892506279113260845246144240e-13),
3699:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.11230682511893290562864133995544214588e-15),
3700:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.44627067257461788044784631155226503036e-17),
3701:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.39869585157420474301450400944478312794e-18),
3702:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.82128612844034824876694595066123093042e-27),
3703:         };
3704:         BOOST_MATH_STATIC const RealType Q[20] = {
3705:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3706:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.65414405277042133067228113526697909557e0),
3707:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.32179221250476209346757936207079534440e0),
3708:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.74217392682100275524983756207618144313e-1),
3709:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.45810448055940046896534973720645113799e-1),
3710:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.81487408603233765436807980794697048675e-2),
3711:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.49442843848941402948883852684502731460e-2),
3712:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.66330842256792791665907478718489013963e-3),
3713:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.93285292223845804061941359223505045576e-4),
3714:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.87966347754794288681626114849829710697e-5),
3715:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.13711644429711675111080150193733607164e-6),
3716:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.61758862007482013187806625777101452737e-7),
3717:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.55435556106272558989915248980090731639e-8),
3718:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.34166571320580242213843747025082914011e-9),
3719:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31411360099525131959755145015018410429e-10),
3720:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 6.21684839228785650625270026640716752452e-12),
3721:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.43021096301255274530428188746599779008e-13),
3722:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.58831290247776456235908211620983180005e-15),
3723:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.74309280855806399632683315923592902203e-16),
3724:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.58097100528573186098159133443927182780e-18),
3725:         };
3726:         // LCOV_EXCL_STOP
3727:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3728:     }
3729:     else if (ilogb(p) >= -32) {
3730:         RealType t = -log2(ldexp(p, 16));
3731: 
3732:         // Rational Approximation
3733:         // Maximum Relative Error: 3.4124e-35
3734:         // LCOV_EXCL_START
3735:         BOOST_MATH_STATIC const RealType P[19] = {
3736:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.41926067826974814669251179264786585885e-1),
3737:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.21141529920003643675474888047093566280e-1),
3738:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.59964592861304582755436075901659426485e-1),
3739:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.95135112971576806260593571877646426022e-2),
3740:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.12322024725362032809787183337883163254e-3),
3741:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.96758465518847580191799508363466893068e-4),
3742:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.12389553946694902774213055563291192175e-4),
3743:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.04599236076217479033545023949602272721e-5),
3744:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.16143771174487665823565565218797804931e-7),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3745-3792 / 第 3745-3792 行
~~~cpp
3745:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.38966874413947625866830582082846088427e-8),
3746:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.02590325514935982607907975481732376204e-9),
3747:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.44376747400143802055827426602151525955e-10),
3748:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.82088624006657184426589019067893704020e-12),
3749:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.95757210706845964048697237729100056232e-13),
3750:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.36096213291559182424937062842308387702e-15),
3751:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.14362780521873256616533770657488533993e-16),
3752:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.73571098395815275003552523759665474105e-18),
3753:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.47286214854389274681661944885238913581e-20),
3754:             BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.73701196181204039400706651811524874455e-34),
3755:         };
3756:         BOOST_MATH_STATIC const RealType Q[18] = {
3757:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3758:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.77119890916406072259446489508263892540e-1),
3759:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.95177888809731859578167185583119074026e-1),
3760:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.29131027214559081111011582466619105016e-2),
3761:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.31442657037887347262737789825299661237e-2),
3762:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.83928863984637222329515960387531101267e-3),
3763:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.07389089078167127136964851949662391744e-4),
3764:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.93013847797006474150589676891548600820e-5),
3765:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.50600573851533884594030683413819219915e-6),
3766:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.94539484213971921794449107859541806317e-8),
3767:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.58360895032645281635534287874266252341e-9),
3768:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.66414102108217999886628042310332365446e-10),
3769:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.07411076181287950822375436854492998754e-11),
3770:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.61224937285582228022463072515935601355e-13),
3771:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.89242339209389981530783624934733098598e-15),
3772:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.11030225010379194015550512905872992373e-16),
3773:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.20285566539355859922818448335043495666e-18),
3774:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.71782855576364068752705740544460766362e-20),
3775:         };
3776:         // LCOV_EXCL_STOP
3777:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3778:     }
3779:     else if (ilogb(p) >= -64) {
3780:         RealType t = -log2(ldexp(p, 32));
3781: 
3782:         // Rational Approximation
3783:         // Maximum Relative Error: 2.1680e-35
3784:         // LCOV_EXCL_START
3785:         BOOST_MATH_STATIC const RealType P[18] = {
3786:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.41926070139289008206183757488364846894e-1),
3787:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.78434820569480998586988738136492447574e-1),
3788:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.07939171933509333571821660328723436210e-1),
3789:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.92438439347811482522082798370060349739e-2),
3790:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.24288918322433485413615362874371441367e-2),
3791:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.04437759300344740815274986587186340509e-3),
3792:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.74063952231188399929705762263485071234e-4),
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, if, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, if, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3793-3840 / 第 3793-3840 行
~~~cpp
3793:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.07228849610363181194047955109059900544e-5),
3794:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.93120850707001212714821992328252707694e-6),
3795:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.40911049607879914351205073608184243057e-7),
3796:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.71898232013947717725198847649536278438e-8),
3797:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06963706982203753050300400912657068823e-9),
3798:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.79849166632277658631839126599110199710e-11),
3799:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.74682085785152276503345630444792840850e-12),
3800:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09650236336641219916377836114077389212e-13),
3801:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.97326394822836529817663710792553753811e-15),
3802:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 9.26635728806398747570910072594323836441e-17),
3803:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.96470010392255781222480229189380065951e-18),
3804:         };
3805:         BOOST_MATH_STATIC const RealType Q[18] = {
3806:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.),
3807:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 8.82841492468725267177870050157374330523e-1),
3808:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.83703946702662950408034486958999188355e-1),
3809:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.09320896703777230915306208582393356690e-1),
3810:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.29346630787642344947323515884281464979e-2),
3811:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.77242894492599243245354774839232776944e-3),
3812:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.05722029871614922850936250945431594997e-4),
3813:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.66920224988248720006255827987385374411e-5),
3814:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.40887155754772190509572243444386095560e-6),
3815:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 4.44545968319921473942351968892623238920e-7),
3816:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 3.17198676140022989760684932594389017027e-8),
3817:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.97376935482567419865730773801543995320e-9),
3818:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.06997835790265899882151030367297786861e-10),
3819:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.06862653266619706928282319356971834957e-12),
3820:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.02334307903766790059473763725329176667e-13),
3821:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 7.33174535634931487079630169746402085699e-15),
3822:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 1.70989324903345102377898775620363767855e-16),
3823:             BOOST_MATH_BIG_CONSTANT(RealType, 113, 5.47067260145014475572799216996976703615e-18),
3824:         };
3825:         // LCOV_EXCL_STOP
3826:         result = tools::evaluate_polynomial(P, t) / (tools::evaluate_polynomial(Q, t) * cbrt(p * p));
3827:     }
3828:     else {
3829:         RealType p_square = p * p;
3830: 
3831:         if ((boost::math::isnormal)(p_square)) {
3832:             result = 1 / cbrt(p_square * constants::two_pi<RealType>());
3833:         }
3834:         else if (p > 0) {
3835:             result = 1 / (cbrt(p) * cbrt(p) * cbrt(constants::two_pi<RealType>()));
3836:         }
3837:         else {
3838:             result = boost::math::numeric_limits<RealType>::infinity();
3839:         }
3840:     }
~~~
- **EN:** This range declares or defines callable logic such as tools::evaluate_polynomial, cbrt, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 tools::evaluate_polynomial, cbrt, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 3841-3888 / 第 3841-3888 行
~~~cpp
3841: 
3842:     return result;
3843: }
3844: 
3845: template <class RealType>
3846: BOOST_MATH_GPU_ENABLED inline RealType mapairy_quantile_imp_prec(const RealType& p, bool complement, const boost::math::integral_constant<int, 53>& tag)
3847: {
3848:     if (p > 0.5) {
3849:         return !complement ? mapairy_quantile_upper_imp_prec(1 - p, tag) : mapairy_quantile_lower_imp_prec(1 - p, tag);
3850:     }
3851: 
3852:     return complement ? mapairy_quantile_upper_imp_prec(p, tag) : mapairy_quantile_lower_imp_prec(p, tag);
3853: }
3854: 
3855: template <class RealType>
3856: BOOST_MATH_GPU_ENABLED inline RealType mapairy_quantile_imp_prec(const RealType& p, bool complement, const boost::math::integral_constant<int, 113>& tag)
3857: {
3858:     if (p > 0.5) {
3859:         return !complement ? mapairy_quantile_upper_imp_prec(1 - p, tag) : mapairy_quantile_lower_imp_prec(1 - p, tag);
3860:     }
3861: 
3862:     return complement ? mapairy_quantile_upper_imp_prec(p, tag) : mapairy_quantile_lower_imp_prec(p, tag);
3863: }
3864: 
3865: template <class RealType, class Policy>
3866: BOOST_MATH_GPU_ENABLED inline RealType mapairy_quantile_imp(const mapairy_distribution<RealType, Policy>& dist, const RealType& p, bool complement)
3867: {
3868:     // This routine implements the quantile for the Map-Airy distribution,
3869:     // the value p may be the probability, or its complement if complement=true.
3870: 
3871:     constexpr auto function = "boost::math::quantile(mapairy<%1%>&, %1%)";
3872:     BOOST_MATH_STD_USING // for ADL of std functions
3873: 
3874:     RealType result = 0;
3875:     RealType scale = dist.scale();
3876:     RealType location = dist.location();
3877: 
3878:     if (false == detail::check_location(function, location, &result, Policy()))
3879:     {
3880:         return result;
3881:     }
3882:     if (false == detail::check_scale(function, scale, &result, Policy()))
3883:     {
3884:         return result;
3885:     }
3886:     if (false == detail::check_probability(function, p, &result, Policy()))
3887:     {
3888:         return result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as scale, location.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 scale, location。

### Lines 3889-3936 / 第 3889-3936 行
~~~cpp
3889:     }
3890: 
3891:     typedef typename tools::promote_args<RealType>::type result_type;
3892:     typedef typename policies::precision<result_type, Policy>::type precision_type;
3893:     typedef boost::math::integral_constant<int,
3894:         precision_type::value <= 0 ? 0 :
3895:         precision_type::value <= 53 ? 53 :
3896:         precision_type::value <= 113 ? 113 : 0
3897:     > tag_type;
3898: 
3899:     static_assert(tag_type::value, "The Map-Airy distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
3900: 
3901:     result = location + scale * mapairy_quantile_imp_prec(p, complement, tag_type());
3902: 
3903:     return result;
3904: }
3905: 
3906: template <class RealType>
3907: BOOST_MATH_GPU_ENABLED inline RealType mapairy_mode_imp_prec(const boost::math::integral_constant<int, 53>&)
3908: {
3909:     return static_cast<RealType>(-1.16158727113597068525);
3910: }
3911: 
3912: template <class RealType>
3913: BOOST_MATH_GPU_ENABLED inline RealType mapairy_mode_imp_prec(const boost::math::integral_constant<int, 113>&)
3914: {
3915:     return BOOST_MATH_BIG_CONSTANT(RealType, 113, -1.1615872711359706852500000803029112987);
3916: }
3917: 
3918: template <class RealType, class Policy>
3919: BOOST_MATH_GPU_ENABLED inline RealType mapairy_mode_imp(const mapairy_distribution<RealType, Policy>& dist)
3920: {
3921:     // This implements the mode for the Map-Airy distribution,
3922: 
3923:     constexpr auto function = "boost::math::mode(mapairy<%1%>&, %1%)";
3924:     BOOST_MATH_STD_USING // for ADL of std functions
3925: 
3926:     RealType result = 0;
3927:     RealType scale = dist.scale();
3928:     RealType location = dist.location();
3929: 
3930:     if (false == detail::check_location(function, location, &result, Policy()))
3931:     {
3932:         return result;
3933:     }
3934:     if (false == detail::check_scale(function, scale, &result, Policy()))
3935:     {
3936:         return result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, mapairy_quantile_imp_prec, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, mapairy_quantile_imp_prec, ...。

### Lines 3937-3984 / 第 3937-3984 行
~~~cpp
3937:     }
3938: 
3939:     typedef typename tools::promote_args<RealType>::type result_type;
3940:     typedef typename policies::precision<result_type, Policy>::type precision_type;
3941:     typedef boost::math::integral_constant<int,
3942:         precision_type::value <= 0 ? 0 :
3943:         precision_type::value <= 53 ? 53 :
3944:         precision_type::value <= 113 ? 113 : 0
3945:     > tag_type;
3946: 
3947:     static_assert(tag_type::value, "The Map-Airy distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
3948: 
3949:     result = location + scale * mapairy_mode_imp_prec<RealType>(tag_type());
3950: 
3951:     return result;
3952: }
3953: 
3954: template <class RealType>
3955: BOOST_MATH_GPU_ENABLED inline RealType mapairy_median_imp_prec(const boost::math::integral_constant<int, 53>&)
3956: {
3957:     return static_cast<RealType>(-0.71671068545502205332);
3958: }
3959: 
3960: template <class RealType>
3961: BOOST_MATH_GPU_ENABLED inline RealType mapairy_median_imp_prec(const boost::math::integral_constant<int, 113>&)
3962: {
3963:     return BOOST_MATH_BIG_CONSTANT(RealType, 113, -0.71671068545502205331700196278067230944440);
3964: }
3965: 
3966: template <class RealType, class Policy>
3967: BOOST_MATH_GPU_ENABLED inline RealType mapairy_median_imp(const mapairy_distribution<RealType, Policy>& dist)
3968: {
3969:     // This implements the median for the Map-Airy distribution,
3970: 
3971:     constexpr auto function = "boost::math::median(mapairy<%1%>&, %1%)";
3972:     BOOST_MATH_STD_USING // for ADL of std functions
3973: 
3974:     RealType result = 0;
3975:     RealType scale = dist.scale();
3976:     RealType location = dist.location();
3977: 
3978:     if (false == detail::check_location(function, location, &result, Policy()))
3979:     {
3980:         return result;
3981:     }
3982:     if (false == detail::check_scale(function, scale, &result, Policy()))
3983:     {
3984:         return result;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, tag_type, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, tag_type, ...。

### Lines 3985-4032 / 第 3985-4032 行
~~~cpp
3985:     }
3986: 
3987:     typedef typename tools::promote_args<RealType>::type result_type;
3988:     typedef typename policies::precision<result_type, Policy>::type precision_type;
3989:     typedef boost::math::integral_constant<int,
3990:         precision_type::value <= 0 ? 0 :
3991:         precision_type::value <= 53 ? 53 :
3992:         precision_type::value <= 113 ? 113 : 0
3993:     > tag_type;
3994: 
3995:     static_assert(tag_type::value, "The Map-Airy distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
3996: 
3997:     result = location + scale * mapairy_median_imp_prec<RealType>(tag_type());
3998: 
3999:     return result;
4000: }
4001: 
4002: template <class RealType>
4003: BOOST_MATH_GPU_ENABLED inline RealType mapairy_entropy_imp_prec(const boost::math::integral_constant<int, 53>&)
4004: {
4005:     return static_cast<RealType>(2.00727681841065634600);
4006: }
4007: 
4008: template <class RealType>
4009: BOOST_MATH_GPU_ENABLED inline RealType mapairy_entropy_imp_prec(const boost::math::integral_constant<int, 113>&)
4010: {
4011:     return BOOST_MATH_BIG_CONSTANT(RealType, 113, 2.0072768184106563460003025875575283708);
4012: }
4013: 
4014: template <class RealType, class Policy>
4015: BOOST_MATH_GPU_ENABLED inline RealType mapairy_entropy_imp(const mapairy_distribution<RealType, Policy>& dist)
4016: {
4017:     // This implements the entropy for the Map-Airy distribution,
4018: 
4019:     constexpr auto function = "boost::math::entropy(mapairy<%1%>&, %1%)";
4020:     BOOST_MATH_STD_USING // for ADL of std functions
4021: 
4022:     RealType result = 0;
4023:     RealType scale = dist.scale();
4024: 
4025:     if (false == detail::check_scale(function, scale, &result, Policy()))
4026:     {
4027:         return result;
4028:     }
4029: 
4030:     typedef typename tools::promote_args<RealType>::type result_type;
4031:     typedef typename policies::precision<result_type, Policy>::type precision_type;
4032:     typedef boost::math::integral_constant<int,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, tag_type, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, tag_type, ...。

### Lines 4033-4080 / 第 4033-4080 行
~~~cpp
4033:         precision_type::value <= 0 ? 0 :
4034:         precision_type::value <= 53 ? 53 :
4035:         precision_type::value <= 113 ? 113 : 0
4036:     > tag_type;
4037: 
4038:     static_assert(tag_type::value, "The Map-Airy distribution is only implemented for types with known precision, and 113 bits or fewer in the mantissa (ie 128 bit quad-floats");
4039: 
4040:     result = mapairy_entropy_imp_prec<RealType>(tag_type()) + log(scale);
4041: 
4042:     return result;
4043: }
4044: 
4045: } // detail
4046: 
4047: template <class RealType = double, class Policy = policies::policy<> >
4048: class mapairy_distribution
4049: {
4050:     public:
4051:     typedef RealType value_type;
4052:     typedef Policy policy_type;
4053: 
4054:     BOOST_MATH_GPU_ENABLED mapairy_distribution(RealType l_location = 0, RealType l_scale = 1)
4055:         : mu(l_location), c(l_scale)
4056:     {
4057:         constexpr auto function = "boost::math::mapairy_distribution<%1%>::mapairy_distribution";
4058:         RealType result = 0;
4059:         detail::check_location(function, l_location, &result, Policy());
4060:         detail::check_scale(function, l_scale, &result, Policy());
4061:     } // mapairy_distribution
4062: 
4063:     BOOST_MATH_GPU_ENABLED RealType location()const
4064:     {
4065:         return mu;
4066:     }
4067:     BOOST_MATH_GPU_ENABLED RealType scale()const
4068:     {
4069:         return c;
4070:     }
4071: 
4072:     private:
4073:     RealType mu;    // The location parameter.
4074:     RealType c;     // The scale parameter.
4075: };
4076: 
4077: typedef mapairy_distribution<double> mapairy;
4078: 
4079: #ifdef __cpp_deduction_guides
4080: template <class RealType>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert, tag_type, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert, tag_type, ...。

### Lines 4081-4128 / 第 4081-4128 行
~~~cpp
4081: mapairy_distribution(RealType) -> mapairy_distribution<typename boost::math::tools::promote_args<RealType>::type>;
4082: template <class RealType>
4083: mapairy_distribution(RealType, RealType) -> mapairy_distribution<typename boost::math::tools::promote_args<RealType>::type>;
4084: #endif
4085: 
4086: template <class RealType, class Policy>
4087: BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> range(const mapairy_distribution<RealType, Policy>&)
4088: { // Range of permissible values for random variable x.
4089:     BOOST_MATH_IF_CONSTEXPR (boost::math::numeric_limits<RealType>::has_infinity)
4090:     {
4091:         return boost::math::pair<RealType, RealType>(-boost::math::numeric_limits<RealType>::infinity(), boost::math::numeric_limits<RealType>::infinity()); // - to + infinity.
4092:     }
4093:     else
4094:     { // Can only use max_value.
4095:         using boost::math::tools::max_value;
4096:         return boost::math::pair<RealType, RealType>(-max_value<RealType>(), max_value<RealType>()); // - to + max.
4097:     }
4098: }
4099: 
4100: template <class RealType, class Policy>
4101: BOOST_MATH_GPU_ENABLED inline const boost::math::pair<RealType, RealType> support(const mapairy_distribution<RealType, Policy>&)
4102: { // Range of supported values for random variable x.
4103:    // This is range where cdf rises from 0 to 1, and outside it, the pdf is zero.
4104:     BOOST_MATH_IF_CONSTEXPR (boost::math::numeric_limits<RealType>::has_infinity)
4105:     {
4106:         return boost::math::pair<RealType, RealType>(-boost::math::numeric_limits<RealType>::infinity(), boost::math::numeric_limits<RealType>::infinity()); // - to + infinity.
4107:     }
4108:     else
4109:     { // Can only use max_value.
4110:         using boost::math::tools::max_value;
4111:         return boost::math::pair<RealType, RealType>(-tools::max_value<RealType>(), max_value<RealType>()); // - to + max.
4112:     }
4113: }
4114: 
4115: template <class RealType, class Policy>
4116: BOOST_MATH_GPU_ENABLED inline RealType pdf(const mapairy_distribution<RealType, Policy>& dist, const RealType& x)
4117: {
4118:     return detail::mapairy_pdf_imp(dist, x);
4119: } // pdf
4120: 
4121: template <class RealType, class Policy>
4122: BOOST_MATH_GPU_ENABLED inline RealType cdf(const mapairy_distribution<RealType, Policy>& dist, const RealType& x)
4123: {
4124:     return detail::mapairy_cdf_imp(dist, x, false);
4125: } // cdf
4126: 
4127: template <class RealType, class Policy>
4128: BOOST_MATH_GPU_ENABLED inline RealType quantile(const mapairy_distribution<RealType, Policy>& dist, const RealType& p)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 4129-4176 / 第 4129-4176 行
~~~cpp
4129: {
4130:     return detail::mapairy_quantile_imp(dist, p, false);
4131: } // quantile
4132: 
4133: template <class RealType, class Policy>
4134: BOOST_MATH_GPU_ENABLED inline RealType cdf(const complemented2_type<mapairy_distribution<RealType, Policy>, RealType>& c)
4135: {
4136:     return detail::mapairy_cdf_imp(c.dist, c.param, true);
4137: } //  cdf complement
4138: 
4139: template <class RealType, class Policy>
4140: BOOST_MATH_GPU_ENABLED inline RealType quantile(const complemented2_type<mapairy_distribution<RealType, Policy>, RealType>& c)
4141: {
4142:     return detail::mapairy_quantile_imp(c.dist, c.param, true);
4143: } // quantile complement
4144: 
4145: template <class RealType, class Policy>
4146: BOOST_MATH_GPU_ENABLED inline RealType mean(const mapairy_distribution<RealType, Policy> &dist)
4147: {
4148:     return dist.location();
4149: }
4150: 
4151: template <class RealType, class Policy>
4152: BOOST_MATH_GPU_ENABLED inline RealType variance(const mapairy_distribution<RealType, Policy>& /*dist*/)
4153: {
4154:     return boost::math::numeric_limits<RealType>::infinity();
4155: }
4156: 
4157: template <class RealType, class Policy>
4158: BOOST_MATH_GPU_ENABLED inline RealType mode(const mapairy_distribution<RealType, Policy>& dist)
4159: {
4160:     return detail::mapairy_mode_imp(dist);
4161: }
4162: 
4163: template <class RealType, class Policy>
4164: BOOST_MATH_GPU_ENABLED inline RealType median(const mapairy_distribution<RealType, Policy>& dist)
4165: {
4166:     return detail::mapairy_median_imp(dist);
4167: }
4168: 
4169: template <class RealType, class Policy>
4170: BOOST_MATH_GPU_ENABLED inline RealType skewness(const mapairy_distribution<RealType, Policy>& /*dist*/)
4171: {
4172:     // There is no skewness:
4173:     typedef typename Policy::assert_undefined_type assert_type;
4174:     static_assert(assert_type::value == 0, "The Map-Airy Distribution has no skewness");
4175: 
4176:     return policies::raise_domain_error<RealType>(
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `RealType` as part of the file's main abstraction. This range declares or defines callable logic such as static_assert.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `RealType`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 static_assert。

### Lines 4177-4220 / 第 4177-4220 行
~~~cpp
4177:         "boost::math::skewness(mapairy<%1%>&)",
4178:         "The Map-Airy distribution does not have a skewness: "
4179:         "the only possible return value is %1%.",
4180:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy()); // infinity?
4181: }
4182: 
4183: template <class RealType, class Policy>
4184: BOOST_MATH_GPU_ENABLED inline RealType kurtosis(const mapairy_distribution<RealType, Policy>& /*dist*/)
4185: {
4186:     // There is no kurtosis:
4187:     typedef typename Policy::assert_undefined_type assert_type;
4188:     static_assert(assert_type::value == 0, "The Map-Airy Distribution has no kurtosis");
4189: 
4190:     return policies::raise_domain_error<RealType>(
4191:         "boost::math::kurtosis(mapairy<%1%>&)",
4192:         "The Map-Airy distribution does not have a kurtosis: "
4193:         "the only possible return value is %1%.",
4194:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy());
4195: }
4196: 
4197: template <class RealType, class Policy>
4198: BOOST_MATH_GPU_ENABLED inline RealType kurtosis_excess(const mapairy_distribution<RealType, Policy>& /*dist*/)
4199: {
4200:     // There is no kurtosis excess:
4201:     typedef typename Policy::assert_undefined_type assert_type;
4202:     static_assert(assert_type::value == 0, "The Map-Airy Distribution has no kurtosis excess");
4203: 
4204:     return policies::raise_domain_error<RealType>(
4205:         "boost::math::kurtosis_excess(mapairy<%1%>&)",
4206:         "The Map-Airy distribution does not have a kurtosis: "
4207:         "the only possible return value is %1%.",
4208:         boost::math::numeric_limits<RealType>::quiet_NaN(), Policy());
4209: }
4210: 
4211: template <class RealType, class Policy>
4212: BOOST_MATH_GPU_ENABLED inline RealType entropy(const mapairy_distribution<RealType, Policy>& dist)
4213: {
4214:     return detail::mapairy_entropy_imp(dist);
4215: }
4216: 
4217: }} // namespaces
4218: 
4219: 
4220: #endif // BOOST_STATS_MAPAIRY_HPP
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
- **Included headers / 包含头文件**: `boost/math/tools/config.hpp, boost/math/tools/numeric_limits.hpp, boost/math/tools/tuple.hpp, boost/math/constants/constants.hpp, boost/math/tools/big_constant.hpp, boost/math/distributions/complement.hpp, boost/math/distributions/detail/common_error_handling.hpp, boost/math/distributions/detail/derived_accessors.hpp, boost/math/tools/rational.hpp, boost/math/special_functions/cbrt.hpp, boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, ...`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `tools::evaluate_polynomial, if, sqrt, exp, mapairy_pdf_imp_prec, mapairy_pdf_imp, location, scale, ...`
