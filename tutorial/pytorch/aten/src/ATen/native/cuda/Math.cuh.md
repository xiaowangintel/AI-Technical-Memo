# Math.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/Math.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `ndtri`, `lcm`, `exp2_impl`, `exp2_kernel`.
- 用途（中文）: 声明或定义与 `ndtri`, `lcm`, `exp2_impl`, `exp2_kernel` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```cpp
   1: #pragma once
   2: 
   3: #include <ATen/AccumulateType.h>
   4: #include <ATen/jit_macros.h>
   5: #include <c10/macros/Macros.h>
   6: #include <ATen/native/cuda/jit_utils.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/AccumulateType.h>`, `<ATen/jit_macros.h>`, `<c10/macros/Macros.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/AccumulateType.h>`, `<ATen/jit_macros.h>`, `<c10/macros/Macros.h>`。

### Lines 8-29
```cpp
   8: namespace at::native {
   9: // See note [Jiterator]
  10: // TODO: elaborate in this comment on the structure of math.cuh
  11: #if AT_USE_JITERATOR()
  12: 
  13: const auto ndtri_string = jiterator_stringify(
  14:   /*
  15:   * This function is derived from the implementation of the digamma function in the Cephes Math Library.
  16:   * See note [3-Clause BSD License for the Cephes Math Library].
  17:   *
  18:   * Evaluates polynomial of degree N:
  19:   *
  20:   *                     2          N
  21:   * y  =  C  + C x + C x  +...+ C x
  22:   *        0    1     2          N
  23:   *
  24:   * Coefficients are stored in reverse order:
  25:   *
  26:   * coef[0] = C  , ..., coef[N] = C  .
  27:   *            N                   0
  28:   */
  29:   template <typename T>
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 30-39
```cpp
  30:   T polevl(const T x, const T A[], const int len) {
  31:     // NOTE: This `polevl` is different from other `polevl`
  32:     // implementation (in PyTorch) which expect the `len` to be
  33:     // `len(A) - 1` instead of `len(A)`.
  34:     T result = 0;
  35:     for (int i = 0; i < len; ++i) {
  36:       result = result * x + A[i];
  37:     }
  38:     return result;
  39:   }
```
- EN: This block defines or continues the implementation of `polevl`.
- CN: 该代码块定义或继续实现 `polevl`。

### Lines 41-47
```cpp
  41:   /*
  42:   * This function is derived from the implementation of the i1e function in the Cephes Math Library.
  43:   * See note [3-Clause BSD License for the Cephes Math Library].
  44:   *
  45:   * Computes the argument, x, for which the area under the Gaussian probability density function
  46:   * (integrated from minus infinity to x) is equal to y.
  47:   */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 48-69
```cpp
  48:   template <typename T>
  49:   T ndtri(T y0) {
  50: 
  51:     constexpr T zero = 0;
  52:     constexpr T one = 1;
  53: 
  54:     // Handles special cases
  55:     if (y0 == zero) {
  56:       return NEG_INFINITY;
  57:     }
  58:     if (y0 == one) {
  59:       return POS_INFINITY;
  60:     }
  61:     if (y0 < zero || y0 > one) {
  62:       return NAN;
  63:     }
  64: 
  65:     bool code = true;
  66:     T y = y0;
  67:     // Note: the constant 0.135... is equal to exp(-2)
  68:     if (y > one - T{0.13533528323661269189}) {
  69:       y = one - y;
```
- EN: This block defines or continues the implementation of `ndtri`.
- CN: 该代码块定义或继续实现 `ndtri`。

### Lines 70-71
```cpp
  70:       code = false;
  71:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 73-93
```cpp
  73:     if (y > T{0.13533528323661269189}) {
  74:       /* approximation for 0 <= |y - 0.5| <= 3/8 */
  75:       static const T P0[5] = {
  76:           -5.99633501014107895267E1,
  77:           9.80010754185999661536E1,
  78:           -5.66762857469070293439E1,
  79:           1.39312609387279679503E1,
  80:           -1.23916583867381258016E0,
  81:       };
  82: 
  83:       static const T Q0[9] = {
  84:         1.00000000000000000000E0,
  85:         1.95448858338141759834E0,
  86:         4.67627912898881538453E0,
  87:         8.63602421390890590575E1,
  88:         -2.25462687854119370527E2,
  89:         2.00260212380060660359E2,
  90:         -8.20372256168333339912E1,
  91:         1.59056225126211695515E1,
  92:         -1.18331621121330003142E0,
  93:       };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 95-95
```cpp
  95:       /* sqrt(2pi) */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 96-96
```cpp
  96:       constexpr T s2pi = 2.50662827463100050242E0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 98-102
```cpp
  98:       y = y - T{0.5};
  99:       const T y2 = y * y;
 100:       T x = y + y * (y2 * polevl(y2, P0, int{5}) / polevl(y2, Q0, int{9}));
 101:       return x * s2pi;
 102:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 104-105
```cpp
 104:     T x = sqrt(T{-2.} * log(y));
 105:     const T x0 = x - (log(x) / x);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 107-108
```cpp
 107:     const T z = one / x;
 108:     T x1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 110-110
```cpp
 110:     /* y > exp(-32) = 1.2664165549e-14 */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 111-132
```cpp
 111:     if (x < T{8.0}) {
 112:       /* Approximation for interval z = sqrt(-2 log y ) between 2 and 8
 113:       * i.e., y between exp(-2) = .135 and exp(-32) = 1.27e-14.
 114:       */
 115:       static const T P1[9] = {
 116:         4.05544892305962419923E0,
 117:         3.15251094599893866154E1,
 118:         5.71628192246421288162E1,
 119:         4.40805073893200834700E1,
 120:         1.46849561928858024014E1,
 121:         2.18663306850790267539E0,
 122:         -1.40256079171354495875E-1,
 123:         -3.50424626827848203418E-2,
 124:         -8.57456785154685413611E-4,
 125:       };
 126: 
 127:       static const T Q1[9] = {
 128:         1.00000000000000000000E0,
 129:         1.57799883256466749731E1,
 130:         4.53907635128879210584E1,
 131:         4.13172038254672030440E1,
 132:         1.50425385692907503408E1,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 133-137
```cpp
 133:         2.50464946208309415979E0,
 134:         -1.42182922854787788574E-1,
 135:         -3.80806407691578277194E-2,
 136:         -9.33259480895457427372E-4,
 137:       };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 139-154
```cpp
 139:       x1 = z * polevl(z, P1, int{9}) / polevl(z, Q1, int{9});
 140:     } else {
 141:       /* Approximation for interval z = sqrt(-2 log y ) between 8 and 64
 142:       * i.e., y between exp(-32) = 1.27e-14 and exp(-2048) = 3.67e-890.
 143:       */
 144:       static const T P2[9] = {
 145:         3.23774891776946035970E0,
 146:         6.91522889068984211695E0,
 147:         3.93881025292474443415E0,
 148:         1.33303460815807542389E0,
 149:         2.01485389549179081538E-1,
 150:         1.23716634817820021358E-2,
 151:         3.01581553508235416007E-4,
 152:         2.65806974686737550832E-6,
 153:         6.23974539184983293730E-9,
 154:       };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 156-166
```cpp
 156:       static const T Q2[9] = {
 157:         1.00000000000000000000E0,
 158:         6.02427039364742014255E0,
 159:         3.67983563856160859403E0,
 160:         1.37702099489081330271E0,
 161:         2.16236993594496635890E-1,
 162:         1.34204006088543189037E-2,
 163:         3.28014464682127739104E-4,
 164:         2.89247864745380683936E-6,
 165:         6.79019408009981274425E-9,
 166:       };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 168-169
```cpp
 168:       x1 = z * polevl(z, P2, int{9}) / polevl(z, Q2, int{9});
 169:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 171-174
```cpp
 171:     x = x0 - x1;
 172:     return (!code) ? x : -x;
 173:   }
 174: ); // ndtri_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 176-187
```cpp
 176: const auto log_ndtr_string = jiterator_stringify(
 177:   template <typename T>
 178:   T log_ndtr(T x) {
 179:     constexpr T SQRT1_2{0.707106781186547524400844362104849039};   // 1/sqrt(2)
 180:     T t = x * SQRT1_2;
 181:     if (x < T{-1.0}) {
 182:       return log(erfcx(-t) / 2) - t * t;
 183:     } else {
 184:       return log1p(-erfc(t) / 2);
 185:     }
 186:   }
 187: ); // log_ndtr_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 189-202
```cpp
 189: const auto gcd_string = jiterator_stringify(
 190:   template <typename T>
 191:   T gcd(const T a_in, const T b_in) {
 192:     T a = abs(a_in);
 193:     T b = abs(b_in);
 194: 
 195:     while (a != T{0}) {
 196:       T c = a;
 197:       a = b % a;
 198:       b = c;
 199:     }
 200: 
 201:     return b;
 202:   }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 203-203
```cpp
 203: ); // gcd_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 205-218
```cpp
 205: const auto lcm_string = jiterator_stringify(
 206:   template <typename T>
 207:   T gcd(const T a_in, const T b_in) {
 208:     T a = abs(a_in);
 209:     T b = abs(b_in);
 210: 
 211:     while (a != T{0}) {
 212:       T c = a;
 213:       a = b % a;
 214:       b = c;
 215:     }
 216: 
 217:     return b;
 218:   }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 220-225
```cpp
 220:   template <typename T>
 221:   T lcm(const T a, const T b) {
 222:     T g = gcd(a, b);
 223:     return (g == T{0}) ? T{0} : abs(a / g * b);
 224:   }
 225: ); // lcm_string
```
- EN: This block defines or continues the implementation of `lcm`.
- CN: 该代码块定义或继续实现 `lcm`。

### Lines 227-230
```cpp
 227: /*
 228:  * For licensing information, please refer to the cpu implementation located in "ATen/native/Math.h".
 229:  */
 230: // [C++ Standard Reference: Gamma Function] https://en.cppreference.com/w/cpp/numeric/math/tgamma
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 231-252
```cpp
 231: const auto digamma_string = jiterator_stringify(
 232:   template <typename T>
 233:   T digamma(T x) {
 234:     static constexpr double PI_f64 = 3.14159265358979323846;
 235: 
 236:     // Short-circuits if x is +/- 0 and returns -/+ ∞ per the C++ standard
 237:     if (x == 0) {
 238:       return copysign(POS_INFINITY, -x);
 239:     }
 240: 
 241:     T result = 0;
 242:     if (x < 0) {
 243:       // Short-circuits if x is a negative integer and returns NaN
 244:       //   per the C++ standard
 245:       const bool x_is_integer = (x == trunc(x));
 246:       if (x_is_integer) {
 247:         return NAN;
 248:       }
 249: 
 250:       // Extracts the fractional part of x as r, since tan(pi * r) is more numerically
 251:       // accurate than tan(pi * x). While these operations are mathematically equivalent
 252:       // since both x and r are in radians and tan() has a periodicity of pi, in practice
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 253-253
```cpp
 253:       // the computation of pi * x is a source of error (when |x| > 1).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 254-258
```cpp
 254:       double q, r;
 255:       r = modf(static_cast<double>(x), &q);
 256:       result = - PI_f64 / tan(PI_f64 * r);
 257:       x = 1 - x;
 258:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 260-263
```cpp
 260:     while (x < T{10}) {
 261:       result -= T{1} / x;
 262:       x += T{1};
 263:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 265-267
```cpp
 265:     if (x == T{10}) {
 266:       return result + T{2.25175258906672110764};
 267:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 269-289
```cpp
 269:     T y = 0;
 270:     if (x < T{1.0e17}) {
 271:       const T A[] = {
 272:         8.33333333333333333333E-2,
 273:         -2.10927960927960927961E-2,
 274:         7.57575757575757575758E-3,
 275:         -4.16666666666666666667E-3,
 276:         3.96825396825396825397E-3,
 277:         -8.33333333333333333333E-3,
 278:         8.33333333333333333333E-2,
 279:       };
 280: 
 281: 
 282:       T z = T{1} / (x * x);
 283: 
 284:       T polevl_result = 0;
 285:       for (int i = 0; i <= 6; i++) {
 286:         polevl_result = polevl_result * z + A[i];
 287:       }
 288:       y = z * polevl_result;
 289:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 291-293
```cpp
 291:     return log(x) - (T{0.5} / x) - y + result;
 292:   }
 293: ); // digamma_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 295-298
```cpp
 295: /*
 296:  * This function is derived from the implementation of the zeta function in the Cephes Math Library.
 297:  * See note [3-Clause BSD License for the Cephes Math Library].
 298:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 299-319
```cpp
 299: const auto zeta_string = jiterator_stringify(
 300:   template <typename T>
 301:   T zeta(T x, T q) {
 302:     const T MACHEP{1.11022302462515654042E-16};
 303:     constexpr T zero{0};
 304:     constexpr T half{0.5};
 305:     constexpr T one{1};
 306:     static const T A[] = {
 307:         12.0,
 308:         -720.0,
 309:         30240.0,
 310:         -1209600.0,
 311:         47900160.0,
 312:         -1.8924375803183791606e9, /*1.307674368e12/691*/
 313:         7.47242496e10,
 314:         -2.950130727918164224e12, /*1.067062284288e16/3617*/
 315:         1.1646782814350067249e14, /*5.109094217170944e18/43867*/
 316:         -4.5979787224074726105e15, /*8.028576626982912e20/174611*/
 317:         1.8152105401943546773e17, /*1.5511210043330985984e23/854513*/
 318:         -7.1661652561756670113e18 /*1.6938241367317436694528e27/236364091*/
 319:     };
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 321-322
```cpp
 321:     int i = 0;
 322:     T a, b, k, s, t, w;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 324-324
```cpp
 324:     // Short-circuits x -> +infty
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 325-327
```cpp
 325:     if (x == one) {
 326:       return POS_INFINITY;
 327:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 329-329
```cpp
 329:     // Short-circuits x < 1 -> NaN
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 330-332
```cpp
 330:     if (x < one) {
 331:       return NAN;
 332:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 334-335
```cpp
 334:     // Short-circuits negative q integers map to +infty,
 335:     //   negative q non-integers map to NaN
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 336-343
```cpp
 336:     if (q <= zero) {
 337:       if (q == floor(q)) {
 338:         return POS_INFINITY;
 339:       }
 340:       if (x != floor(x)) {
 341:         return NAN;
 342:       }
 343:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 345-357
```cpp
 345:     s = pow(q, -x);
 346:     a = q;
 347:     i = 0;
 348:     b = zero;
 349:     while ((i < 9) || (a <= T{9.0})) {
 350:       i += 1;
 351:       a += one;
 352:       b = pow(a, -x);
 353:       s += b;
 354:       if ((-MACHEP * s < b) && (b < MACHEP * s)) {
 355:         return s;
 356:       }
 357:     };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 359-379
```cpp
 359:     w = a;
 360:     s += b * w / (x - one);
 361:     s -= half * b;
 362:     a = one;
 363:     k = zero;
 364:     for (int i = 0; i < 12; i++) {
 365:       a *= x + k;
 366:       b /= w;
 367:       t = a * b / A[i];
 368:       s = s + t;
 369:       t = fabs(t / s);
 370: 
 371:       if (t < MACHEP) {
 372:         return s;
 373:       }
 374: 
 375:       k += one;
 376:       a *= x + k;
 377:       b /= w;
 378:       k += one;
 379:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 381-383
```cpp
 381:     return s;
 382:   }
 383: ); // zeta_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 385-406
```cpp
 385: const auto trigamma_string = jiterator_stringify(
 386:   template <typename T>
 387:   T trigamma(T x) {
 388:     const T PI{3.14159265358979323846};
 389:     T sign = 1;
 390:     T result = 0;
 391: 
 392:     if (x < T{0.5}) {
 393:       sign = -1;
 394:       T sin_pi_x = sin(PI * x);
 395:       result -= (PI * PI) / (sin_pi_x * sin_pi_x);
 396:       x = 1 - x;
 397:     }
 398: 
 399:     for (int i = 0; i < 6; ++i) {
 400:       result += T{1} / (x * x);
 401:       x += 1;
 402:     }
 403: 
 404:     const T one{1};
 405:     const T ixx = one / (x*x);
 406:     result += (one + one / (T{2}*x) + ixx * (one/T{6} - ixx * (one/T{30} - ixx * (one/T{42})))) / x;
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 407-409
```cpp
 407:     return sign * result;
 408: }
 409: ); // trigamma_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 411-416
```cpp
 411: const auto lgamma_string = jiterator_stringify(
 412:   template <typename T>
 413:   T lgamma_kernel(T a) {
 414:     return lgamma(a);
 415:   }
 416: ); // lgamma_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 418-426
```cpp
 418: const auto polygamma_string = zeta_string + jiterator_stringify(
 419:   template <typename T>
 420:   T polygamma(T x, int n) {
 421:     // already blocked if n <= 1
 422:     const auto one = T{1};
 423:     return ((n % 2) ? one : -one) * exp(lgamma(static_cast<T>(n) + one)) *
 424:         zeta<T>(static_cast<T>(n + 1), x);
 425:   }
 426: ); // polygamma_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 428-432
```cpp
 428: const auto exp2_string = jiterator_stringify(
 429:   template <typename T>
 430:   T exp2_impl(T a) {
 431:     return exp2(a);
 432:   }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 434-441
```cpp
 434:   namespace std { template <typename _Ty> class complex; }
 435:   template <typename T>
 436:   std::complex<T> exp2_impl(std::complex<T> x) {
 437:     // There is no std::exp2 overload for complex, so instead
 438:     // use the identity 2^x = e^(ln(2) * x)
 439:     const auto ln_2 = static_cast<T>(0.693147180559945309417232121458176);
 440:     return exp(ln_2 * x);
 441:   }
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `exp2_impl`.
- CN: 该代码块定义或继续实现 `exp2_impl`。

### Lines 443-447
```cpp
 443:   template <typename T>
 444:   T exp2_kernel(T a) {
 445:     return exp2_impl(a);
 446:   }
 447: ); // exp2_string
```
- EN: This block defines or continues the implementation of `exp2_kernel`.
- CN: 该代码块定义或继续实现 `exp2_kernel`。

### Lines 449-454
```cpp
 449: const auto erfc_string = jiterator_stringify(
 450:   template <typename T>
 451:   T erfc_kernel(T a) {
 452:     return erfc(a);
 453:   }
 454: ); // erfc_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 456-461
```cpp
 456: const auto erfinv_string = jiterator_stringify(
 457:   template <typename T>
 458:   T erfinv_kernel(T a) {
 459:     return erfinv(a);
 460:   }
 461: ); // erfinv_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 463-479
```cpp
 463: const auto entr_string = jiterator_stringify(
 464:   template <typename T>
 465:   T entr(T a) {
 466:     if (a != a) {
 467:       return a;
 468:     }
 469: 
 470:     if (a > 0) {
 471:       return -a * log(a);
 472:     }
 473: 
 474:     if (a == 0) {
 475:       return 0;
 476:     }
 477: 
 478:     return NEG_INFINITY;
 479:   }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 480-480
```cpp
 480: ); // entr_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 482-483
```cpp
 482: // NOTE: `kaiser_window_string` depends on `i0_string`
 483: //       for its implementation.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 484-500
```cpp
 484: const auto i0_string = jiterator_stringify(
 485:   template<typename T>
 486:   T chbevl(T x, const T array[], const int len) {
 487: 
 488:       T b0, b1, b2;
 489: 
 490:       b0 = array[0];
 491:       b1 = 0;
 492: 
 493:       for (int i = 1; i < len; ++i)  {
 494:           b2 = b1;
 495:           b1 = b0;
 496:           b0 = x * b1 - b2 + array[i];
 497:       }
 498: 
 499:       return T{0.5} * (b0 - b2);
 500:   }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 502-523
```cpp
 502:   template<typename T>
 503:   T i0(T _x) {
 504:       T x = fabs(_x);
 505: 
 506:       if (x <= T{8.0}) {
 507:           /* Chebyshev coefficients for exp(-x) I0(x)
 508:           *   in the interval [0,8].
 509:           *
 510:           * lim(x->0){ exp(-x) I0(x) } = 1.
 511:           */
 512:           static const T A[] = {
 513:               -4.41534164647933937950E-18, 3.33079451882223809783E-17,
 514:               -2.43127984654795469359E-16, 1.71539128555513303061E-15,
 515:               -1.16853328779934516808E-14, 7.67618549860493561688E-14,
 516:               -4.85644678311192946090E-13, 2.95505266312963983461E-12,
 517:               -1.72682629144155570723E-11, 9.67580903537323691224E-11,
 518:               -5.18979560163526290666E-10, 2.65982372468238665035E-9,
 519:               -1.30002500998624804212E-8,  6.04699502254191894932E-8,
 520:               -2.67079385394061173391E-7,  1.11738753912010371815E-6,
 521:               -4.41673835845875056359E-6,  1.64484480707288970893E-5,
 522:               -5.75419501008210370398E-5,  1.88502885095841655729E-4,
 523:               -5.76375574538582365885E-4,  1.63947561694133579842E-3,
```
- EN: This block defines or continues the implementation of `i0`, `exp`.
- CN: 该代码块定义或继续实现 `i0`, `exp`。

### Lines 524-527
```cpp
 524:               -4.32430999505057594430E-3,  1.05464603945949983183E-2,
 525:               -2.37374148058994688156E-2,  4.93052842396707084878E-2,
 526:               -9.49010970480476444210E-2,  1.71620901522208775349E-1,
 527:               -3.04682672343198398683E-1,  6.76795274409476084995E-1};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 529-531
```cpp
 529:           T y = (x / T{2.0}) - T{2.0};
 530:           return exp(x) * chbevl(y, A, int{30});
 531:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 533-538
```cpp
 533:       // Handles x > 8 case
 534:       /* Chebyshev coefficients for exp(-x) sqrt(x) I0(x)
 535:       * in the inverted interval [8,infinity].
 536:       *
 537:       * lim(x->inf){ exp(-x) sqrt(x) I0(x) } = 1/sqrt(2pi).
 538:       */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This block defines or continues the implementation of `exp`.
- CN: 该代码块定义或继续实现 `exp`。

### Lines 539-552
```cpp
 539:       const T B[] = {
 540:           -7.23318048787475395456E-18, -4.83050448594418207126E-18,
 541:           4.46562142029675999901E-17,  3.46122286769746109310E-17,
 542:           -2.82762398051658348494E-16, -3.42548561967721913462E-16,
 543:           1.77256013305652638360E-15,  3.81168066935262242075E-15,
 544:           -9.55484669882830764870E-15, -4.15056934728722208663E-14,
 545:           1.54008621752140982691E-14,  3.85277838274214270114E-13,
 546:           7.18012445138366623367E-13,  -1.79417853150680611778E-12,
 547:           -1.32158118404477131188E-11, -3.14991652796324136454E-11,
 548:           1.18891471078464383424E-11,  4.94060238822496958910E-10,
 549:           3.39623202570838634515E-9,   2.26666899049817806459E-8,
 550:           2.04891858946906374183E-7,   2.89137052083475648297E-6,
 551:           6.88975834691682398426E-5,   3.36911647825569408990E-3,
 552:           8.04490411014108831608E-1};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 554-556
```cpp
 554:       return (exp(x) * chbevl(T{32.0} / x - T{2.0}, B, int{25})) / sqrt(x);
 555:   }
 556: ); // i0_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 558-573
```cpp
 558: const auto i1_string = jiterator_stringify(
 559:   template<typename T>
 560:   T chbevl(const T x, const T array[], const int len) {
 561:       T b0, b1, b2;
 562: 
 563:       b0 = array[0];
 564:       b1 = 0;
 565: 
 566:       for (int i = 1; i < len; ++i)  {
 567:           b2 = b1;
 568:           b1 = b0;
 569:           b0 = x * b1 - b2 + array[i];
 570:       }
 571: 
 572:       return T{0.5} * (b0 - b2);
 573:   }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 575-596
```cpp
 575:   template <typename T>
 576:   T i1(T _x) {
 577:     const T x = fabs(_x);
 578: 
 579:     if (x <= T{8.0}) {
 580:       // Chebyshev coefficients for exp(-x) i1(x) in the internal [0, 8]
 581:       //   lim(x->0){ exp(-x) i1(x) / x } = 1/2
 582:       static const T coefficients[] = {
 583:           2.77791411276104639959E-18, -2.11142121435816608115E-17,
 584:           1.55363195773620046921E-16, -1.10559694773538630805E-15,
 585:           7.60068429473540693410E-15, -5.04218550472791168711E-14,
 586:           3.22379336594557470981E-13, -1.98397439776494371520E-12,
 587:           1.17361862988909016308E-11, -6.66348972350202774223E-11,
 588:           3.62559028155211703701E-10, -1.88724975172282928790E-9,
 589:           9.38153738649577178388E-9,  -4.44505912879632808065E-8,
 590:           2.00329475355213526229E-7,  -8.56872026469545474066E-7,
 591:           3.47025130813767847674E-6,  -1.32731636560394358279E-5,
 592:           4.78156510755005422638E-5,  -1.61760815825896745588E-4,
 593:           5.12285956168575772895E-4,  -1.51357245063125314899E-3,
 594:           4.15642294431288815669E-3,  -1.05640848946261981558E-2,
 595:           2.47264490306265168283E-2,  -5.29459812080949914269E-2,
 596:           1.02643658689847095384E-1,  -1.76416518357834055153E-1,
```
- EN: This block defines or continues the implementation of `i1`, `exp`.
- CN: 该代码块定义或继续实现 `i1`, `exp`。

### Lines 597-601
```cpp
 597:           2.52587186443633654823E-1};
 598:       const T y = x / T{2.0} - T{2.0};
 599:       const T out = exp(x) * x * chbevl(y, coefficients, int{29});
 600:       return (_x < T{0.0}) ? -out : out;
 601:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 603-605
```cpp
 603:     // Chebyshev coefficients for exp(-x) sqrt(x) i1(x)
 604:     //   in the inverted interval [8, infinity]
 605:     //   lim(x->inf){ exp(-x) sqrt(x) i1(x) } = 1/sqrt(2pi)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This block defines or continues the implementation of `exp`.
- CN: 该代码块定义或继续实现 `exp`。

### Lines 606-619
```cpp
 606:     static const T coefficients[] = {
 607:       7.51729631084210481353E-18,  4.41434832307170791151E-18,
 608:       -4.65030536848935832153E-17, -3.20952592199342395980E-17,
 609:       2.96262899764595013876E-16,  3.30820231092092828324E-16,
 610:       -1.88035477551078244854E-15, -3.81440307243700780478E-15,
 611:       1.04202769841288027642E-14,  4.27244001671195135429E-14,
 612:       -2.10154184277266431302E-14, -4.08355111109219731823E-13,
 613:       -7.19855177624590851209E-13, 2.03562854414708950722E-12,
 614:       1.41258074366137813316E-11,  3.25260358301548823856E-11,
 615:       -1.89749581235054123450E-11, -5.58974346219658380687E-10,
 616:       -3.83538038596423702205E-9,  -2.63146884688951950684E-8,
 617:       -2.51223623787020892529E-7,  -3.88256480887769039346E-6,
 618:       -1.10588938762623716291E-4,  -9.76109749136146840777E-3,
 619:       7.78576235018280120474E-1};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 620-623
```cpp
 620:     const T out = (exp(x) * chbevl(T{32.} / x - T{2.}, coefficients, int{25})) / sqrt(x);
 621:     return (_x < T{0.}) ? -out : out;
 622:   }
 623: ); // i1_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 625-640
```cpp
 625: const auto i1e_string = jiterator_stringify(
 626:   template<typename T>
 627:   T chbevl(const T x, const T array[], const int len) {
 628:       T b0, b1, b2;
 629: 
 630:       b0 = array[0];
 631:       b1 = 0;
 632: 
 633:       for (int i = 1; i < len; ++i)  {
 634:           b2 = b1;
 635:           b1 = b0;
 636:           b0 = x * b1 - b2 + array[i];
 637:       }
 638: 
 639:       return T{0.5} * (b0 - b2);
 640:   }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 642-642
```cpp
 642:   // See double and float instantiations below
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 643-644
```cpp
 643:   template <typename T>
 644:   T i1e(T _x) { }
```
- EN: This block defines or continues the implementation of `i1e`.
- CN: 该代码块定义或继续实现 `i1e`。

### Lines 646-646
```cpp
 646:   // Double specialization (uses different coefficients than the float version)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 647-668
```cpp
 647:   template<>
 648:   double i1e(double _x) {
 649:     const double x = fabs(_x);
 650:     if (x <= double{8.}) {
 651:       // Chebyshev double coefficients for exp(-x) i1(x) in the interval [0,8].
 652:       // Note: lim(x->0){ exp(-x) i1(x) / x } = 1/2.
 653:       static const double coefficients[] = {
 654:         2.77791411276104639959E-18, -2.11142121435816608115E-17,
 655:         1.55363195773620046921E-16, -1.10559694773538630805E-15,
 656:         7.60068429473540693410E-15, -5.04218550472791168711E-14,
 657:         3.22379336594557470981E-13, -1.98397439776494371520E-12,
 658:         1.17361862988909016308E-11, -6.66348972350202774223E-11,
 659:         3.62559028155211703701E-10, -1.88724975172282928790E-9,
 660:         9.38153738649577178388E-9,  -4.44505912879632808065E-8,
 661:         2.00329475355213526229E-7,  -8.56872026469545474066E-7,
 662:         3.47025130813767847674E-6,  -1.32731636560394358279E-5,
 663:         4.78156510755005422638E-5,  -1.61760815825896745588E-4,
 664:         5.12285956168575772895E-4,  -1.51357245063125314899E-3,
 665:         4.15642294431288815669E-3,  -1.05640848946261981558E-2,
 666:         2.47264490306265168283E-2,  -5.29459812080949914269E-2,
 667:         1.02643658689847095384E-1,  -1.76416518357834055153E-1,
 668:         2.52587186443633654823E-1};
```
- EN: This block defines or continues the implementation of `i1e`, `exp`.
- CN: 该代码块定义或继续实现 `i1e`, `exp`。

### Lines 669-672
```cpp
 669:       const double y = x / double{2.} - double{2.};
 670:       const double out = chbevl(y, coefficients, int{29}) * x;
 671:       return (_x < 0.) ? -out : out;
 672:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 674-678
```cpp
 674:     // Chebyshev coefficients for exp(-x) sqrt(x) i1(x)
 675:     //   in the inverted interval (8, infinity].
 676:     // Note: lim(x->inf){ exp(-x) sqrt(x) i1(x) } = 1/sqrt(2pi).
 677:     // TODO: what's an "inverted interval"? Open on the left
 678:     //   and closed on the right?
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This block defines or continues the implementation of `exp`.
- CN: 该代码块定义或继续实现 `exp`。

### Lines 679-692
```cpp
 679:   static const double coefficients[] = {
 680:       7.51729631084210481353E-18,  4.41434832307170791151E-18,
 681:       -4.65030536848935832153E-17, -3.20952592199342395980E-17,
 682:       2.96262899764595013876E-16,  3.30820231092092828324E-16,
 683:       -1.88035477551078244854E-15, -3.81440307243700780478E-15,
 684:       1.04202769841288027642E-14,  4.27244001671195135429E-14,
 685:       -2.10154184277266431302E-14, -4.08355111109219731823E-13,
 686:       -7.19855177624590851209E-13, 2.03562854414708950722E-12,
 687:       1.41258074366137813316E-11,  3.25260358301548823856E-11,
 688:       -1.89749581235054123450E-11, -5.58974346219658380687E-10,
 689:       -3.83538038596423702205E-9,  -2.63146884688951950684E-8,
 690:       -2.51223623787020892529E-7,  -3.88256480887769039346E-6,
 691:       -1.10588938762623716291E-4,  -9.76109749136146840777E-3,
 692:       7.78576235018280120474E-1};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 694-696
```cpp
 694:     const double out = chbevl(double{32.} / x - double{2.}, coefficients, int{25}) / sqrt(x);
 695:     return (_x < double{0.}) ? -out : out;
 696:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 698-698
```cpp
 698:   // Float specialization (uses different coefficients than the double version)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 699-720
```cpp
 699:   template<>
 700:   float i1e(float _x) {
 701:     const float x = fabsf(_x);
 702:     if (x <= float{8.}) {
 703:       // Chebyshev double coefficients for exp(-x) i1(x) in the interval [0,8].
 704:       // Note: lim(x->0){ exp(-x) i1(x) / x } = 1/2.
 705:       static const float coefficients[] = {
 706:         9.38153738649577178388E-9f,
 707:         -4.44505912879632808065E-8f,
 708:         2.00329475355213526229E-7f,
 709:         -8.56872026469545474066E-7f,
 710:         3.47025130813767847674E-6f,
 711:         -1.32731636560394358279E-5f,
 712:         4.78156510755005422638E-5f,
 713:         -1.61760815825896745588E-4f,
 714:         5.12285956168575772895E-4f,
 715:         -1.51357245063125314899E-3f,
 716:         4.15642294431288815669E-3f,
 717:         -1.05640848946261981558E-2f,
 718:         2.47264490306265168283E-2f,
 719:         -5.29459812080949914269E-2f,
 720:         1.02643658689847095384E-1f,
```
- EN: This block defines or continues the implementation of `i1e`, `exp`.
- CN: 该代码块定义或继续实现 `i1e`, `exp`。

### Lines 721-726
```cpp
 721:         -1.76416518357834055153E-1f,
 722:         2.52587186443633654823E-1f};
 723:       const float y = x / float{2.} - float{2.};
 724:       const float out = chbevl(y, coefficients, int{17}) * x;
 725:       return (_x < 0.) ? -out : out;
 726:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 728-732
```cpp
 728:     // Chebyshev coefficients for exp(-x) sqrt(x) i1(x)
 729:     //   in the inverted interval (8, infinity].
 730:     // Note: lim(x->inf){ exp(-x) sqrt(x) i1(x) } = 1/sqrt(2pi).
 731:     // TODO: what's an "inverted interval"? Open on the left
 732:     //   and closed on the right?
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This block defines or continues the implementation of `exp`.
- CN: 该代码块定义或继续实现 `exp`。

### Lines 733-740
```cpp
 733:   static const float coefficients[] = {
 734:       -3.83538038596423702205E-9f,
 735:       -2.63146884688951950684E-8f,
 736:       -2.51223623787020892529E-7f,
 737:       -3.88256480887769039346E-6f,
 738:       -1.10588938762623716291E-4f,
 739:       -9.76109749136146840777E-3f,
 740:       7.78576235018280120474E-1f};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 742-745
```cpp
 742:     const float out = chbevl(float{32.} / x - float{2.}, coefficients, int{7}) / sqrt(x);
 743:     return (_x < float{0.}) ? -out : out;
 744:   }
 745: ); // i1e_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 747-754
```cpp
 747: const auto kaiser_window_string = i0_string + jiterator_stringify(
 748:   template <typename T>
 749:   T kaiser_window(T a, T inv_alpha, T beta, T inv_i0_beta) {
 750:     T x = a * inv_alpha - T{1};
 751:     T y = max(T{0}, T{1} - x * x);
 752:     return i0(beta * sqrt(y)) * inv_i0_beta;
 753:   }
 754: ); // kaiser_window_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 756-766
```cpp
 756: const auto sinc_string = jiterator_stringify(
 757:   template <typename T>
 758:   T sinc(T a) {
 759:     if (a == T(0)) {
 760:       return T(1);
 761:     }
 762:     constexpr T pi = T(3.14159265358979323846L);
 763:     T product = pi * a;
 764:     return std::sin(product) / product;
 765:   }
 766: ); // sinc_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 768-769
```cpp
 768: const auto erfcx_string = jiterator_stringify(
 769:   /* The next function is taken from http://ab-initio.mit.edu/faddeeva */
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 771-784
```cpp
 771:   /* Copyright (c) 2012 Massachusetts Institute of Technology
 772:   *
 773:   * Permission is hereby granted, free of charge, to any person obtaining
 774:   * a copy of this software and associated documentation files (the
 775:   * "Software"), to deal in the Software without restriction, including
 776:   * without limitation the rights to use, copy, modify, merge, publish,
 777:   * distribute, sublicense, and/or sell copies of the Software, and to
 778:   * permit persons to whom the Software is furnished to do so, subject to
 779:   * the following conditions:
 780:   *
 781:   * The above copyright notice and this permission notice shall be
 782:   * included in all copies or substantial portions of the Software.
 783:   *
 784:   * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 785-793
```cpp
 785:   * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
 786:   * MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
 787:   * NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
 788:   * LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
 789:   * OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
 790:   * WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 791:   */
 792: 
 793:   /* erfcx(x) = exp(x^2) erfc(x) function, for real x, written by
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 794-794
```cpp
 794:     Steven G. Johnson, October 2012.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 796-796
```cpp
 796:     This function combines a few different ideas.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 798-799
```cpp
 798:     First, for x > 50, it uses a continued-fraction expansion (same as
 799:     for the Faddeeva function, but with algebraic simplifications for z=i*x).
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 801-802
```cpp
 801:     Second, for 0 <= x <= 50, it uses Chebyshev polynomial approximations,
 802:     but with two twists:
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 804-807
```cpp
 804:         a) It maps x to y = 4 / (4+x) in [0,1].  This simple transformation,
 805:           inspired by a similar transformation in the octave-forge/specfun
 806:           erfcx by Soren Hauberg, results in much faster Chebyshev convergence
 807:           than other simple transformations I have examined.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 809-813
```cpp
 809:         b) Instead of using a single Chebyshev polynomial for the entire
 810:           [0,1] y interval, we break the interval up into 100 equal
 811:           subintervals, with a switch/lookup table, and use much lower
 812:           degree Chebyshev polynomials in each subinterval. This greatly
 813:           improves performance in my tests.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 815-816
```cpp
 815:     For x < 0, we use the relationship erfcx(-x) = 2 exp(x^2) - erfc(x),
 816:     with the usual checks for overflow etcetera.
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 818-822
```cpp
 818:     Performance-wise, it seems to be substantially faster than either
 819:     the SLATEC DERFC function [or an erfcx function derived there from]
 820:     or Cody's CALERF function (from netlib.org/specfun), while
 821:     retaining near machine precision in accuracy.
 822:   */
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 824-824
```cpp
 824:   /* Given y100 = 100 * y, where y = 4 / (4 + x) for x >= 0, compute erfc(x).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 826-831
```cpp
 826:     Uses a look-up table of 100 different Chebyshev polynomials
 827:     for y intervals [0,0.01], [0.01,0.02], ...., [0.99,1], generated
 828:     with the help of Maple and a little shell script.   This allows
 829:     the Chebyshev polynomials to be of significantly lower degree (about 1/4)
 830:     compared to fitting the whole [0,1] interval with a single polynomial.
 831:   */
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 833-833
```cpp
 833:   // TODO: review if this is computing in double when given a float input
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 834-855
```cpp
 834:   template <typename T>
 835:   T erfcx_y100(T y100) {
 836:     switch (static_cast<int>(y100)) {
 837:       case 0: {
 838:       T t = 2*y100 - 1;
 839:       return 0.70878032454106438663e-3 + (0.71234091047026302958e-3 + (0.35779077297597742384e-5 + (0.17403143962587937815e-7 + (0.81710660047307788845e-10 + (0.36885022360434957634e-12 + 0.15917038551111111111e-14 * t) * t) * t) * t) * t) * t;
 840:       }
 841:       case 1: {
 842:       T t = 2*y100 - 3;
 843:       return 0.21479143208285144230e-2 + (0.72686402367379996033e-3 + (0.36843175430938995552e-5 + (0.18071841272149201685e-7 + (0.85496449296040325555e-10 + (0.38852037518534291510e-12 + 0.16868473576888888889e-14 * t) * t) * t) * t) * t) * t;
 844:       }
 845:       case 2: {
 846:       T t = 2*y100 - 5;
 847:       return 0.36165255935630175090e-2 + (0.74182092323555510862e-3 + (0.37948319957528242260e-5 + (0.18771627021793087350e-7 + (0.89484715122415089123e-10 + (0.40935858517772440862e-12 + 0.17872061464888888889e-14 * t) * t) * t) * t) * t) * t;
 848:       }
 849:       case 3: {
 850:       T t = 2*y100 - 7;
 851:       return 0.51154983860031979264e-2 + (0.75722840734791660540e-3 + (0.39096425726735703941e-5 + (0.19504168704300468210e-7 + (0.93687503063178993915e-10 + (0.43143925959079664747e-12 + 0.18939926435555555556e-14 * t) * t) * t) * t) * t) * t;
 852:       }
 853:       case 4: {
 854:       T t = 2*y100 - 9;
 855:       return 0.66457513172673049824e-2 + (0.77310406054447454920e-3 + (0.40289510589399439385e-5 + (0.20271233238288381092e-7 + (0.98117631321709100264e-10 + (0.45484207406017752971e-12 + 0.20076352213333333333e-14 * t) * t) * t) * t) * t) * t;
```
- EN: This block defines or continues the implementation of `erfcx_y100`.
- CN: 该代码块定义或继续实现 `erfcx_y100`。

### Lines 856-869
```cpp
 856:       }
 857:       case 5: {
 858:       T t = 2*y100 - 11;
 859:       return 0.82082389970241207883e-2 + (0.78946629611881710721e-3 + (0.41529701552622656574e-5 + (0.21074693344544655714e-7 + (0.10278874108587317989e-9 + (0.47965201390613339638e-12 + 0.21285907413333333333e-14 * t) * t) * t) * t) * t) * t;
 860:       }
 861:       case 6: {
 862:       T t = 2*y100 - 13;
 863:       return 0.98039537275352193165e-2 + (0.80633440108342840956e-3 + (0.42819241329736982942e-5 + (0.21916534346907168612e-7 + (0.10771535136565470914e-9 + (0.50595972623692822410e-12 + 0.22573462684444444444e-14 * t) * t) * t) * t) * t) * t;
 864:       }
 865:       case 7: {
 866:       T t = 2*y100 - 15;
 867:       return 0.11433927298290302370e-1 + (0.82372858383196561209e-3 + (0.44160495311765438816e-5 + (0.22798861426211986056e-7 + (0.11291291745879239736e-9 + (0.53386189365816880454e-12 + 0.23944209546666666667e-14 * t) * t) * t) * t) * t) * t;
 868:       }
 869:       case 8: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 870-883
```cpp
 870:       T t = 2*y100 - 17;
 871:       return 0.13099232878814653979e-1 + (0.84167002467906968214e-3 + (0.45555958988457506002e-5 + (0.23723907357214175198e-7 + (0.11839789326602695603e-9 + (0.56346163067550237877e-12 + 0.25403679644444444444e-14 * t) * t) * t) * t) * t) * t;
 872:       }
 873:       case 9: {
 874:       T t = 2*y100 - 19;
 875:       return 0.14800987015587535621e-1 + (0.86018092946345943214e-3 + (0.47008265848816866105e-5 + (0.24694040760197315333e-7 + (0.12418779768752299093e-9 + (0.59486890370320261949e-12 + 0.26957764568888888889e-14 * t) * t) * t) * t) * t) * t;
 876:       }
 877:       case 10: {
 878:       T t = 2*y100 - 21;
 879:       return 0.16540351739394069380e-1 + (0.87928458641241463952e-3 + (0.48520195793001753903e-5 + (0.25711774900881709176e-7 + (0.13030128534230822419e-9 + (0.62820097586874779402e-12 + 0.28612737351111111111e-14 * t) * t) * t) * t) * t) * t;
 880:       }
 881:       case 11: {
 882:       T t = 2*y100 - 23;
 883:       return 0.18318536789842392647e-1 + (0.89900542647891721692e-3 + (0.50094684089553365810e-5 + (0.26779777074218070482e-7 + (0.13675822186304615566e-9 + (0.66358287745352705725e-12 + 0.30375273884444444444e-14 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 884-897
```cpp
 884:       }
 885:       case 12: {
 886:       T t = 2*y100 - 25;
 887:       return 0.20136801964214276775e-1 + (0.91936908737673676012e-3 + (0.51734830914104276820e-5 + (0.27900878609710432673e-7 + (0.14357976402809042257e-9 + (0.70114790311043728387e-12 + 0.32252476000000000000e-14 * t) * t) * t) * t) * t) * t;
 888:       }
 889:       case 13: {
 890:       T t = 2*y100 - 27;
 891:       return 0.21996459598282740954e-1 + (0.94040248155366777784e-3 + (0.53443911508041164739e-5 + (0.29078085538049374673e-7 + (0.15078844500329731137e-9 + (0.74103813647499204269e-12 + 0.34251892320000000000e-14 * t) * t) * t) * t) * t) * t;
 892:       }
 893:       case 14: {
 894:       T t = 2*y100 - 29;
 895:       return 0.23898877187226319502e-1 + (0.96213386835900177540e-3 + (0.55225386998049012752e-5 + (0.30314589961047687059e-7 + (0.15840826497296335264e-9 + (0.78340500472414454395e-12 + 0.36381553564444444445e-14 * t) * t) * t) * t) * t) * t;
 896:       }
 897:       case 15: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 898-911
```cpp
 898:       T t = 2*y100 - 31;
 899:       return 0.25845480155298518485e-1 + (0.98459293067820123389e-3 + (0.57082915920051843672e-5 + (0.31613782169164830118e-7 + (0.16646478745529630813e-9 + (0.82840985928785407942e-12 + 0.38649975768888888890e-14 * t) * t) * t) * t) * t) * t;
 900:       }
 901:       case 16: {
 902:       T t = 2*y100 - 33;
 903:       return 0.27837754783474696598e-1 + (0.10078108563256892757e-2 + (0.59020366493792212221e-5 + (0.32979263553246520417e-7 + (0.17498524159268458073e-9 + (0.87622459124842525110e-12 + 0.41066206488888888890e-14 * t) * t) * t) * t) * t) * t;
 904:       }
 905:       case 17: {
 906:       T t = 2*y100 - 35;
 907:       return 0.29877251304899307550e-1 + (0.10318204245057349310e-2 + (0.61041829697162055093e-5 + (0.34414860359542720579e-7 + (0.18399863072934089607e-9 + (0.92703227366365046533e-12 + 0.43639844053333333334e-14 * t) * t) * t) * t) * t) * t;
 908:       }
 909:       case 18: {
 910:       T t = 2*y100 - 37;
 911:       return 0.31965587178596443475e-1 + (0.10566560976716574401e-2 + (0.63151633192414586770e-5 + (0.35924638339521924242e-7 + (0.19353584758781174038e-9 + (0.98102783859889264382e-12 + 0.46381060817777777779e-14 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 912-925
```cpp
 912:       }
 913:       case 19: {
 914:       T t = 2*y100 - 39;
 915:       return 0.34104450552588334840e-1 + (0.10823541191350532574e-2 + (0.65354356159553934436e-5 + (0.37512918348533521149e-7 + (0.20362979635817883229e-9 + (0.10384187833037282363e-11 + 0.49300625262222222221e-14 * t) * t) * t) * t) * t) * t;
 916:       }
 917:       case 20: {
 918:       T t = 2*y100 - 41;
 919:       return 0.36295603928292425716e-1 + (0.11089526167995268200e-2 + (0.67654845095518363577e-5 + (0.39184292949913591646e-7 + (0.21431552202133775150e-9 + (0.10994259106646731797e-11 + 0.52409949102222222221e-14 * t) * t) * t) * t) * t) * t;
 920:       }
 921:       case 21: {
 922:       T t = 2*y100 - 43;
 923:       return 0.38540888038840509795e-1 + (0.11364917134175420009e-2 + (0.70058230641246312003e-5 + (0.40943644083718586939e-7 + (0.22563034723692881631e-9 + (0.11642841011361992885e-11 + 0.55721092871111111110e-14 * t) * t) * t) * t) * t) * t;
 924:       }
 925:       case 22: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 926-939
```cpp
 926:       T t = 2*y100 - 45;
 927:       return 0.40842225954785960651e-1 + (0.11650136437945673891e-2 + (0.72569945502343006619e-5 + (0.42796161861855042273e-7 + (0.23761401711005024162e-9 + (0.12332431172381557035e-11 + 0.59246802364444444445e-14 * t) * t) * t) * t) * t) * t;
 928:       }
 929:       case 23: {
 930:       T t = 2*y100 - 47;
 931:       return 0.43201627431540222422e-1 + (0.11945628793917272199e-2 + (0.75195743532849206263e-5 + (0.44747364553960993492e-7 + (0.25030885216472953674e-9 + (0.13065684400300476484e-11 + 0.63000532853333333334e-14 * t) * t) * t) * t) * t) * t;
 932:       }
 933:       case 24: {
 934:       T t = 2*y100 - 49;
 935:       return 0.45621193513810471438e-1 + (0.12251862608067529503e-2 + (0.77941720055551920319e-5 + (0.46803119830954460212e-7 + (0.26375990983978426273e-9 + (0.13845421370977119765e-11 + 0.66996477404444444445e-14 * t) * t) * t) * t) * t) * t;
 936:       }
 937:       case 25: {
 938:       T t = 2*y100 - 51;
 939:       return 0.48103121413299865517e-1 + (0.12569331386432195113e-2 + (0.80814333496367673980e-5 + (0.48969667335682018324e-7 + (0.27801515481905748484e-9 + (0.14674637611609884208e-11 + 0.71249589351111111110e-14 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 940-953
```cpp
 940:       }
 941:       case 26: {
 942:       T t = 2*y100 - 53;
 943:       return 0.50649709676983338501e-1 + (0.12898555233099055810e-2 + (0.83820428414568799654e-5 + (0.51253642652551838659e-7 + (0.29312563849675507232e-9 + (0.15556512782814827846e-11 + 0.75775607822222222221e-14 * t) * t) * t) * t) * t) * t;
 944:       }
 945:       case 27: {
 946:       T t = 2*y100 - 55;
 947:       return 0.53263363664388864181e-1 + (0.13240082443256975769e-2 + (0.86967260015007658418e-5 + (0.53662102750396795566e-7 + (0.30914568786634796807e-9 + (0.16494420240828493176e-11 + 0.80591079644444444445e-14 * t) * t) * t) * t) * t) * t;
 948:       }
 949:       case 28: {
 950:       T t = 2*y100 - 57;
 951:       return 0.55946601353500013794e-1 + (0.13594491197408190706e-2 + (0.90262520233016380987e-5 + (0.56202552975056695376e-7 + (0.32613310410503135996e-9 + (0.17491936862246367398e-11 + 0.85713381688888888890e-14 * t) * t) * t) * t) * t) * t;
 952:       }
 953:       case 29: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 954-967
```cpp
 954:       T t = 2*y100 - 59;
 955:       return 0.58702059496154081813e-1 + (0.13962391363223647892e-2 + (0.93714365487312784270e-5 + (0.58882975670265286526e-7 + (0.34414937110591753387e-9 + (0.18552853109751857859e-11 + 0.91160736711111111110e-14 * t) * t) * t) * t) * t) * t;
 956:       }
 957:       case 30: {
 958:       T t = 2*y100 - 61;
 959:       return 0.61532500145144778048e-1 + (0.14344426411912015247e-2 + (0.97331446201016809696e-5 + (0.61711860507347175097e-7 + (0.36325987418295300221e-9 + (0.19681183310134518232e-11 + 0.96952238400000000000e-14 * t) * t) * t) * t) * t) * t;
 960:       }
 961:       case 31: {
 962:       T t = 2*y100 - 63;
 963:       return 0.64440817576653297993e-1 + (0.14741275456383131151e-2 + (0.10112293819576437838e-4 + (0.64698236605933246196e-7 + (0.38353412915303665586e-9 + (0.20881176114385120186e-11 + 0.10310784480000000000e-13 * t) * t) * t) * t) * t) * t;
 964:       }
 965:       case 32: {
 966:       T t = 2*y100 - 65;
 967:       return 0.67430045633130393282e-1 + (0.15153655418916540370e-2 + (0.10509857606888328667e-4 + (0.67851706529363332855e-7 + (0.40504602194811140006e-9 + (0.22157325110542534469e-11 + 0.10964842115555555556e-13 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 968-981
```cpp
 968:       }
 969:       case 33: {
 970:       T t = 2*y100 - 67;
 971:       return 0.70503365513338850709e-1 + (0.15582323336495709827e-2 + (0.10926868866865231089e-4 + (0.71182482239613507542e-7 + (0.42787405890153386710e-9 + (0.23514379522274416437e-11 + 0.11659571751111111111e-13 * t) * t) * t) * t) * t) * t;
 972:       }
 973:       case 34: {
 974:       T t = 2*y100 - 69;
 975:       return 0.73664114037944596353e-1 + (0.16028078812438820413e-2 + (0.11364423678778207991e-4 + (0.74701423097423182009e-7 + (0.45210162777476488324e-9 + (0.24957355004088569134e-11 + 0.12397238257777777778e-13 * t) * t) * t) * t) * t) * t;
 976:       }
 977:       case 35: {
 978:       T t = 2*y100 - 71;
 979:       return 0.76915792420819562379e-1 + (0.16491766623447889354e-2 + (0.11823685320041302169e-4 + (0.78420075993781544386e-7 + (0.47781726956916478925e-9 + (0.26491544403815724749e-11 + 0.13180196462222222222e-13 * t) * t) * t) * t) * t) * t;
 980:       }
 981:       case 36: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 982-995
```cpp
 982:       T t = 2*y100 - 73;
 983:       return 0.80262075578094612819e-1 + (0.16974279491709504117e-2 + (0.12305888517309891674e-4 + (0.82350717698979042290e-7 + (0.50511496109857113929e-9 + (0.28122528497626897696e-11 + 0.14010889635555555556e-13 * t) * t) * t) * t) * t) * t;
 984:       }
 985:       case 37: {
 986:       T t = 2*y100 - 75;
 987:       return 0.83706822008980357446e-1 + (0.17476561032212656962e-2 + (0.12812343958540763368e-4 + (0.86506399515036435592e-7 + (0.53409440823869467453e-9 + (0.29856186620887555043e-11 + 0.14891851591111111111e-13 * t) * t) * t) * t) * t) * t;
 988:       }
 989:       case 38: {
 990:       T t = 2*y100 - 77;
 991:       return 0.87254084284461718231e-1 + (0.17999608886001962327e-2 + (0.13344443080089492218e-4 + (0.90900994316429008631e-7 + (0.56486134972616465316e-9 + (0.31698707080033956934e-11 + 0.15825697795555555556e-13 * t) * t) * t) * t) * t) * t;
 992:       }
 993:       case 39: {
 994:       T t = 2*y100 - 79;
 995:       return 0.90908120182172748487e-1 + (0.18544478050657699758e-2 + (0.13903663143426120077e-4 + (0.95549246062549906177e-7 + (0.59752787125242054315e-9 + (0.33656597366099099413e-11 + 0.16815130613333333333e-13 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 996-1009
```cpp
 996:       }
 997:       case 40: {
 998:       T t = 2*y100 - 81;
 999:       return 0.94673404508075481121e-1 + (0.19112284419887303347e-2 + (0.14491572616545004930e-4 + (0.10046682186333613697e-6 + (0.63221272959791000515e-9 + (0.35736693975589130818e-11 + 0.17862931591111111111e-13 * t) * t) * t) * t) * t) * t;
1000:       }
1001:       case 41: {
1002:       T t = 2*y100 - 83;
1003:       return 0.98554641648004456555e-1 + (0.19704208544725622126e-2 + (0.15109836875625443935e-4 + (0.10567036667675984067e-6 + (0.66904168640019354565e-9 + (0.37946171850824333014e-11 + 0.18971959040000000000e-13 * t) * t) * t) * t) * t) * t;
1004:       }
1005:       case 42: {
1006:       T t = 2*y100 - 85;
1007:       return 0.10255677889470089531e0 + (0.20321499629472857418e-2 + (0.15760224242962179564e-4 + (0.11117756071353507391e-6 + (0.70814785110097658502e-9 + (0.40292553276632563925e-11 + 0.20145143075555555556e-13 * t) * t) * t) * t) * t) * t;
1008:       }
1009:       case 43: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1010-1023
```cpp
1010:       T t = 2*y100 - 87;
1011:       return 0.10668502059865093318e0 + (0.20965479776148731610e-2 + (0.16444612377624983565e-4 + (0.11700717962026152749e-6 + (0.74967203250938418991e-9 + (0.42783716186085922176e-11 + 0.21385479360000000000e-13 * t) * t) * t) * t) * t) * t;
1012:       }
1013:       case 44: {
1014:       T t = 2*y100 - 89;
1015:       return 0.11094484319386444474e0 + (0.21637548491908170841e-2 + (0.17164995035719657111e-4 + (0.12317915750735938089e-6 + (0.79376309831499633734e-9 + (0.45427901763106353914e-11 + 0.22696025653333333333e-13 * t) * t) * t) * t) * t) * t;
1016:       }
1017:       case 45: {
1018:       T t = 2*y100 - 91;
1019:       return 0.11534201115268804714e0 + (0.22339187474546420375e-2 + (0.17923489217504226813e-4 + (0.12971465288245997681e-6 + (0.84057834180389073587e-9 + (0.48233721206418027227e-11 + 0.24079890062222222222e-13 * t) * t) * t) * t) * t) * t;
1020:       }
1021:       case 46: {
1022:       T t = 2*y100 - 93;
1023:       return 0.11988259392684094740e0 + (0.23071965691918689601e-2 + (0.18722342718958935446e-4 + (0.13663611754337957520e-6 + (0.89028385488493287005e-9 + (0.51210161569225846701e-11 + 0.25540227111111111111e-13 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1024-1037
```cpp
1024:       }
1025:       case 47: {
1026:       T t = 2*y100 - 95;
1027:       return 0.12457298393509812907e0 + (0.23837544771809575380e-2 + (0.19563942105711612475e-4 + (0.14396736847739470782e-6 + (0.94305490646459247016e-9 + (0.54366590583134218096e-11 + 0.27080225920000000000e-13 * t) * t) * t) * t) * t) * t;
1028:       }
1029:       case 48: {
1030:       T t = 2*y100 - 97;
1031:       return 0.12941991566142438816e0 + (0.24637684719508859484e-2 + (0.20450821127475879816e-4 + (0.15173366280523906622e-6 + (0.99907632506389027739e-9 + (0.57712760311351625221e-11 + 0.28703099555555555556e-13 * t) * t) * t) * t) * t) * t;
1032:       }
1033:       case 49: {
1034:       T t = 2*y100 - 99;
1035:       return 0.13443048593088696613e0 + (0.25474249981080823877e-2 + (0.21385669591362915223e-4 + (0.15996177579900443030e-6 + (0.10585428844575134013e-8 + (0.61258809536787882989e-11 + 0.30412080142222222222e-13 * t) * t) * t) * t) * t) * t;
1036:       }
1037:       case 50: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1038-1051
```cpp
1038:       T t = 2*y100 - 101;
1039:       return 0.13961217543434561353e0 + (0.26349215871051761416e-2 + (0.22371342712572567744e-4 + (0.16868008199296822247e-6 + (0.11216596910444996246e-8 + (0.65015264753090890662e-11 + 0.32210394506666666666e-13 * t) * t) * t) * t) * t) * t;
1040:       }
1041:       case 51: {
1042:       T t = 2*y100 - 103;
1043:       return 0.14497287157673800690e0 + (0.27264675383982439814e-2 + (0.23410870961050950197e-4 + (0.17791863939526376477e-6 + (0.11886425714330958106e-8 + (0.68993039665054288034e-11 + 0.34101266222222222221e-13 * t) * t) * t) * t) * t) * t;
1044:       }
1045:       case 52: {
1046:       T t = 2*y100 - 105;
1047:       return 0.15052089272774618151e0 + (0.28222846410136238008e-2 + (0.24507470422713397006e-4 + (0.18770927679626136909e-6 + (0.12597184587583370712e-8 + (0.73203433049229821618e-11 + 0.36087889048888888890e-13 * t) * t) * t) * t) * t) * t;
1048:       }
1049:       case 53: {
1050:       T t = 2*y100 - 107;
1051:       return 0.15626501395774612325e0 + (0.29226079376196624949e-2 + (0.25664553693768450545e-4 + (0.19808568415654461964e-6 + (0.13351257759815557897e-8 + (0.77658124891046760667e-11 + 0.38173420035555555555e-13 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1052-1065
```cpp
1052:       }
1053:       case 54: {
1054:       T t = 2*y100 - 109;
1055:       return 0.16221449434620737567e0 + (0.30276865332726475672e-2 + (0.26885741326534564336e-4 + (0.20908350604346384143e-6 + (0.14151148144240728728e-8 + (0.82369170665974313027e-11 + 0.40360957457777777779e-13 * t) * t) * t) * t) * t) * t;
1056:       }
1057:       case 55: {
1058:       T t = 2*y100 - 111;
1059:       return 0.16837910595412130659e0 + (0.31377844510793082301e-2 + (0.28174873844911175026e-4 + (0.22074043807045782387e-6 + (0.14999481055996090039e-8 + (0.87348993661930809254e-11 + 0.42653528977777777779e-13 * t) * t) * t) * t) * t) * t;
1060:       }
1061:       case 56: {
1062:       T t = 2*y100 - 113;
1063:       return 0.17476916455659369953e0 + (0.32531815370903068316e-2 + (0.29536024347344364074e-4 + (0.23309632627767074202e-6 + (0.15899007843582444846e-8 + (0.92610375235427359475e-11 + 0.45054073102222222221e-13 * t) * t) * t) * t) * t) * t;
1064:       }
1065:       case 57: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1066-1079
```cpp
1066:       T t = 2*y100 - 115;
1067:       return 0.18139556223643701364e0 + (0.33741744168096996041e-2 + (0.30973511714709500836e-4 + (0.24619326937592290996e-6 + (0.16852609412267750744e-8 + (0.98166442942854895573e-11 + 0.47565418097777777779e-13 * t) * t) * t) * t) * t) * t;
1068:       }
1069:       case 58: {
1070:       T t = 2*y100 - 117;
1071:       return 0.18826980194443664549e0 + (0.35010775057740317997e-2 + (0.32491914440014267480e-4 + (0.26007572375886319028e-6 + (0.17863299617388376116e-8 + (0.10403065638343878679e-10 + 0.50190265831111111110e-13 * t) * t) * t) * t) * t) * t;
1072:       }
1073:       case 59: {
1074:       T t = 2*y100 - 119;
1075:       return 0.19540403413693967350e0 + (0.36342240767211326315e-2 + (0.34096085096200907289e-4 + (0.27479061117017637474e-6 + (0.18934228504790032826e-8 + (0.11021679075323598664e-10 + 0.52931171733333333334e-13 * t) * t) * t) * t) * t) * t;
1076:       }
1077:       case 60: {
1078:       T t = 2*y100 - 121;
1079:       return 0.20281109560651886959e0 + (0.37739673859323597060e-2 + (0.35791165457592409054e-4 + (0.29038742889416172404e-6 + (0.20068685374849001770e-8 + (0.11673891799578381999e-10 + 0.55790523093333333334e-13 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1080-1093
```cpp
1080:       }
1081:       case 61: {
1082:       T t = 2*y100 - 123;
1083:       return 0.21050455062669334978e0 + (0.39206818613925652425e-2 + (0.37582602289680101704e-4 + (0.30691836231886877385e-6 + (0.21270101645763677824e-8 + (0.12361138551062899455e-10 + 0.58770520160000000000e-13 * t) * t) * t) * t) * t) * t;
1084:       }
1085:       case 62: {
1086:       T t = 2*y100 - 125;
1087:       return 0.21849873453703332479e0 + (0.40747643554689586041e-2 + (0.39476163820986711501e-4 + (0.32443839970139918836e-6 + (0.22542053491518680200e-8 + (0.13084879235290858490e-10 + 0.61873153262222222221e-13 * t) * t) * t) * t) * t) * t;
1088:       }
1089:       case 63: {
1090:       T t = 2*y100 - 127;
1091:       return 0.22680879990043229327e0 + (0.42366354648628516935e-2 + (0.41477956909656896779e-4 + (0.34300544894502810002e-6 + (0.23888264229264067658e-8 + (0.13846596292818514601e-10 + 0.65100183751111111110e-13 * t) * t) * t) * t) * t) * t;
1092:       }
1093:       case 64: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1094-1107
```cpp
1094:       T t = 2*y100 - 129;
1095:       return 0.23545076536988703937e0 + (0.44067409206365170888e-2 + (0.43594444916224700881e-4 + (0.36268045617760415178e-6 + (0.25312606430853202748e-8 + (0.14647791812837903061e-10 + 0.68453122631111111110e-13 * t) * t) * t) * t) * t) * t;
1096:       }
1097:       case 65: {
1098:       T t = 2*y100 - 131;
1099:       return 0.24444156740777432838e0 + (0.45855530511605787178e-2 + (0.45832466292683085475e-4 + (0.38352752590033030472e-6 + (0.26819103733055603460e-8 + (0.15489984390884756993e-10 + 0.71933206364444444445e-13 * t) * t) * t) * t) * t) * t;
1100:       }
1101:       case 66: {
1102:       T t = 2*y100 - 133;
1103:       return 0.25379911500634264643e0 + (0.47735723208650032167e-2 + (0.48199253896534185372e-4 + (0.40561404245564732314e-6 + (0.28411932320871165585e-8 + (0.16374705736458320149e-10 + 0.75541379822222222221e-13 * t) * t) * t) * t) * t) * t;
1104:       }
1105:       case 67: {
1106:       T t = 2*y100 - 135;
1107:       return 0.26354234756393613032e0 + (0.49713289477083781266e-2 + (0.50702455036930367504e-4 + (0.42901079254268185722e-6 + (0.30095422058900481753e-8 + (0.17303497025347342498e-10 + 0.79278273368888888890e-13 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1108-1121
```cpp
1108:       }
1109:       case 68: {
1110:       T t = 2*y100 - 137;
1111:       return 0.27369129607732343398e0 + (0.51793846023052643767e-2 + (0.53350152258326602629e-4 + (0.45379208848865015485e-6 + (0.31874057245814381257e-8 + (0.18277905010245111046e-10 + 0.83144182364444444445e-13 * t) * t) * t) * t) * t) * t;
1112:       }
1113:       case 69: {
1114:       T t = 2*y100 - 139;
1115:       return 0.28426714781640316172e0 + (0.53983341916695141966e-2 + (0.56150884865255810638e-4 + (0.48003589196494734238e-6 + (0.33752476967570796349e-8 + (0.19299477888083469086e-10 + 0.87139049137777777779e-13 * t) * t) * t) * t) * t) * t;
1116:       }
1117:       case 70: {
1118:       T t = 2*y100 - 141;
1119:       return 0.29529231465348519920e0 + (0.56288077305420795663e-2 + (0.59113671189913307427e-4 + (0.50782393781744840482e-6 + (0.35735475025851713168e-8 + (0.20369760937017070382e-10 + 0.91262442613333333334e-13 * t) * t) * t) * t) * t) * t;
1120:       }
1121:       case 71: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1122-1135
```cpp
1122:       T t = 2*y100 - 143;
1123:       return 0.30679050522528838613e0 + (0.58714723032745403331e-2 + (0.62248031602197686791e-4 + (0.53724185766200945789e-6 + (0.37827999418960232678e-8 + (0.21490291930444538307e-10 + 0.95513539182222222221e-13 * t) * t) * t) * t) * t) * t;
1124:       }
1125:       case 72: {
1126:       T t = 2*y100 - 145;
1127:       return 0.31878680111173319425e0 + (0.61270341192339103514e-2 + (0.65564012259707640976e-4 + (0.56837930287837738996e-6 + (0.40035151353392378882e-8 + (0.22662596341239294792e-10 + 0.99891109760000000000e-13 * t) * t) * t) * t) * t) * t;
1128:       }
1129:       case 73: {
1130:       T t = 2*y100 - 147;
1131:       return 0.33130773722152622027e0 + (0.63962406646798080903e-2 + (0.69072209592942396666e-4 + (0.60133006661885941812e-6 + (0.42362183765883466691e-8 + (0.23888182347073698382e-10 + 0.10439349811555555556e-12 * t) * t) * t) * t) * t) * t;
1132:       }
1133:       case 74: {
1134:       T t = 2*y100 - 149;
1135:       return 0.34438138658041336523e0 + (0.66798829540414007258e-2 + (0.72783795518603561144e-4 + (0.63619220443228800680e-6 + (0.44814499336514453364e-8 + (0.25168535651285475274e-10 + 0.10901861383111111111e-12 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1136-1149
```cpp
1136:       }
1137:       case 75: {
1138:       T t = 2*y100 - 151;
1139:       return 0.35803744972380175583e0 + (0.69787978834882685031e-2 + (0.76710543371454822497e-4 + (0.67306815308917386747e-6 + (0.47397647975845228205e-8 + (0.26505114141143050509e-10 + 0.11376390933333333333e-12 * t) * t) * t) * t) * t) * t;
1140:       }
1141:       case 76: {
1142:       T t = 2*y100 - 153;
1143:       return 0.37230734890119724188e0 + (0.72938706896461381003e-2 + (0.80864854542670714092e-4 + (0.71206484718062688779e-6 + (0.50117323769745883805e-8 + (0.27899342394100074165e-10 + 0.11862637614222222222e-12 * t) * t) * t) * t) * t) * t;
1144:       }
1145:       case 77: {
1146:       T t = 2*y100 - 155;
1147:       return 0.38722432730555448223e0 + (0.76260375162549802745e-2 + (0.85259785810004603848e-4 + (0.75329383305171327677e-6 + (0.52979361368388119355e-8 + (0.29352606054164086709e-10 + 0.12360253370666666667e-12 * t) * t) * t) * t) * t) * t;
1148:       }
1149:       case 78: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1150-1163
```cpp
1150:       T t = 2*y100 - 157;
1151:       return 0.40282355354616940667e0 + (0.79762880915029728079e-2 + (0.89909077342438246452e-4 + (0.79687137961956194579e-6 + (0.55989731807360403195e-8 + (0.30866246101464869050e-10 + 0.12868841946666666667e-12 * t) * t) * t) * t) * t) * t;
1152:       }
1153:       case 79: {
1154:       T t = 2*y100 - 159;
1155:       return 0.41914223158913787649e0 + (0.83456685186950463538e-2 + (0.94827181359250161335e-4 + (0.84291858561783141014e-6 + (0.59154537751083485684e-8 + (0.32441553034347469291e-10 + 0.13387957943111111111e-12 * t) * t) * t) * t) * t) * t;
1156:       }
1157:       case 80: {
1158:       T t = 2*y100 - 161;
1159:       return 0.43621971639463786896e0 + (0.87352841828289495773e-2 + (0.10002929142066799966e-3 + (0.89156148280219880024e-6 + (0.62480008150788597147e-8 + (0.34079760983458878910e-10 + 0.13917107176888888889e-12 * t) * t) * t) * t) * t) * t;
1160:       }
1161:       case 81: {
1162:       T t = 2*y100 - 163;
1163:       return 0.45409763548534330981e0 + (0.91463027755548240654e-2 + (0.10553137232446167258e-3 + (0.94293113464638623798e-6 + (0.65972492312219959885e-8 + (0.35782041795476563662e-10 + 0.14455745872000000000e-12 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1164-1177
```cpp
1164:       }
1165:       case 82: {
1166:       T t = 2*y100 - 165;
1167:       return 0.47282001668512331468e0 + (0.95799574408860463394e-2 + (0.11135019058000067469e-3 + (0.99716373005509038080e-6 + (0.69638453369956970347e-8 + (0.37549499088161345850e-10 + 0.15003280712888888889e-12 * t) * t) * t) * t) * t) * t;
1168:       }
1169:       case 83: {
1170:       T t = 2*y100 - 167;
1171:       return 0.49243342227179841649e0 + (0.10037550043909497071e-1 + (0.11750334542845234952e-3 + (0.10544006716188967172e-5 + (0.73484461168242224872e-8 + (0.39383162326435752965e-10 + 0.15559069118222222222e-12 * t) * t) * t) * t) * t) * t;
1172:       }
1173:       case 84: {
1174:       T t = 2*y100 - 169;
1175:       return 0.51298708979209258326e0 + (0.10520454564612427224e-1 + (0.12400930037494996655e-3 + (0.11147886579371265246e-5 + (0.77517184550568711454e-8 + (0.41283980931872622611e-10 + 0.16122419680000000000e-12 * t) * t) * t) * t) * t) * t;
1176:       }
1177:       case 85: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1178-1191
```cpp
1178:       T t = 2*y100 - 171;
1179:       return 0.53453307979101369843e0 + (0.11030120618800726938e-1 + (0.13088741519572269581e-3 + (0.11784797595374515432e-5 + (0.81743383063044825400e-8 + (0.43252818449517081051e-10 + 0.16692592640000000000e-12 * t) * t) * t) * t) * t) * t;
1180:       }
1181:       case 86: {
1182:       T t = 2*y100 - 173;
1183:       return 0.55712643071169299478e0 + (0.11568077107929735233e-1 + (0.13815797838036651289e-3 + (0.12456314879260904558e-5 + (0.86169898078969313597e-8 + (0.45290446811539652525e-10 + 0.17268801084444444444e-12 * t) * t) * t) * t) * t) * t;
1184:       }
1185:       case 87: {
1186:       T t = 2*y100 - 175;
1187:       return 0.58082532122519320968e0 + (0.12135935999503877077e-1 + (0.14584223996665838559e-3 + (0.13164068573095710742e-5 + (0.90803643355106020163e-8 + (0.47397540713124619155e-10 + 0.17850211608888888889e-12 * t) * t) * t) * t) * t) * t;
1188:       }
1189:       case 88: {
1190:       T t = 2*y100 - 177;
1191:       return 0.60569124025293375554e0 + (0.12735396239525550361e-1 + (0.15396244472258863344e-3 + (0.13909744385382818253e-5 + (0.95651595032306228245e-8 + (0.49574672127669041550e-10 + 0.18435945564444444444e-12 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1192-1205
```cpp
1192:       }
1193:       case 89: {
1194:       T t = 2*y100 - 179;
1195:       return 0.63178916494715716894e0 + (0.13368247798287030927e-1 + (0.16254186562762076141e-3 + (0.14695084048334056083e-5 + (0.10072078109604152350e-7 + (0.51822304995680707483e-10 + 0.19025081422222222222e-12 * t) * t) * t) * t) * t) * t;
1196:       }
1197:       case 90: {
1198:       T t = 2*y100 - 181;
1199:       return 0.65918774689725319200e0 + (0.14036375850601992063e-1 + (0.17160483760259706354e-3 + (0.15521885688723188371e-5 + (0.10601827031535280590e-7 + (0.54140790105837520499e-10 + 0.19616655146666666667e-12 * t) * t) * t) * t) * t) * t;
1200:       }
1201:       case 91: {
1202:       T t = 2*y100 - 183;
1203:       return 0.68795950683174433822e0 + (0.14741765091365869084e-1 + (0.18117679143520433835e-3 + (0.16392004108230585213e-5 + (0.11155116068018043001e-7 + (0.56530360194925690374e-10 + 0.20209663662222222222e-12 * t) * t) * t) * t) * t) * t;
1204:       }
1205:       case 92: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1206-1219
```cpp
1206:       T t = 2*y100 - 185;
1207:       return 0.71818103808729967036e0 + (0.15486504187117112279e-1 + (0.19128428784550923217e-3 + (0.17307350969359975848e-5 + (0.11732656736113607751e-7 + (0.58991125287563833603e-10 + 0.20803065333333333333e-12 * t) * t) * t) * t) * t) * t;
1208:       }
1209:       case 93: {
1210:       T t = 2*y100 - 187;
1211:       return 0.74993321911726254661e0 + (0.16272790364044783382e-1 + (0.20195505163377912645e-3 + (0.18269894883203346953e-5 + (0.12335161021630225535e-7 + (0.61523068312169087227e-10 + 0.21395783431111111111e-12 * t) * t) * t) * t) * t) * t;
1212:       }
1213:       case 94: {
1214:       T t = 2*y100 - 189;
1215:       return 0.78330143531283492729e0 + (0.17102934132652429240e-1 + (0.21321800585063327041e-3 + (0.19281661395543913713e-5 + (0.12963340087354341574e-7 + (0.64126040998066348872e-10 + 0.21986708942222222222e-12 * t) * t) * t) * t) * t) * t;
1216:       }
1217:       case 95: {
1218:       T t = 2*y100 - 191;
1219:       return 0.81837581041023811832e0 + (0.17979364149044223802e-1 + (0.22510330592753129006e-3 + (0.20344732868018175389e-5 + (0.13617902941839949718e-7 + (0.66799760083972474642e-10 + 0.22574701262222222222e-12 * t) * t) * t) * t) * t) * t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1220-1233
```cpp
1220:       }
1221:       case 96: {
1222:       T t = 2*y100 - 193;
1223:       return 0.85525144775685126237e0 + (0.18904632212547561026e-1 + (0.23764237370371255638e-3 + (0.21461248251306387979e-5 + (0.14299555071870523786e-7 + (0.69543803864694171934e-10 + 0.23158593688888888889e-12 * t) * t) * t) * t) * t) * t;
1224:       }
1225:       case 97: {
1226:       T t = 2*y100 - 195;
1227:       return 0.89402868170849933734e0 + (0.19881418399127202569e-1 + (0.25086793128395995798e-3 + (0.22633402747585233180e-5 + (0.15008997042116532283e-7 + (0.72357609075043941261e-10 + 0.23737194737777777778e-12 * t) * t) * t) * t) * t) * t;
1228:       }
1229:       case 98: {
1230:       T t = 2*y100 - 197;
1231:       return 0.93481333942870796363e0 + (0.20912536329780368893e-1 + (0.26481403465998477969e-3 + (0.23863447359754921676e-5 + (0.15746923065472184451e-7 + (0.75240468141720143653e-10 + 0.24309291271111111111e-12 * t) * t) * t) * t) * t) * t;
1232:       }
1233:       case 99: {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1234-1237
```cpp
1234:       T t = 2*y100 - 199;
1235:       return 0.97771701335885035464e0 + (0.22000938572830479551e-1 + (0.27951610702682383001e-3 + (0.25153688325245314530e-5 + (0.16514019547822821453e-7 + (0.78191526829368231251e-10 + 0.24873652355555555556e-12 * t) * t) * t) * t) * t) * t;
1236:       }
1237:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1239-1240
```cpp
1239:     // we only get here if y = 1, i.e. |x| < 4*eps, in which case
1240:     // erfcx is within 1e-15 of 1..
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1241-1242
```cpp
1241:     return 1.;
1242:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1244-1265
```cpp
1244:   template <typename T>
1245:   T erfcx(T x) {
1246:     // Short-circuits on NaN (returning NaN)
1247:     if (x != x) {
1248:       return x;
1249:     }
1250: 
1251:     if (x >= 0) {
1252:       if (x > T{50}) { // continued-fraction expansion is faster
1253:         const T ispi = 0.56418958354775628694807945156; // 1 / sqrt(pi)
1254: 
1255:         if (x > T{5e7}) { // 1-term expansion, important to avoid overflow
1256:           return ispi / x;
1257:         }
1258: 
1259:         /* 5-term expansion (rely on compiler for CSE), simplified from:
1260:                   ispi / (x+0.5/(x+1/(x+1.5/(x+2/x))))  */
1261:         return ispi * ((x*x) * (x*x+T{4.5}) + T{2}) / (x * ((x*x) * (x*x+T{5}) + T{3.75}));
1262:       }
1263: 
1264:       // x >= 0 x <= 50
1265:       return erfcx_y100(T{400} / (T{4} + x));
```
- EN: This block defines or continues the implementation of `erfcx`, `NaN`.
- CN: 该代码块定义或继续实现 `erfcx`, `NaN`。

### Lines 1266-1266
```cpp
1266:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1268-1268
```cpp
1268:     // x < 0
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1269-1273
```cpp
1269:     if (x < T{-26.7}) {
1270:       return POS_INFINITY;
1271:     } else if (x < T{-6.1}) {
1272:       return T{2} * exp(x * x);
1273:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1275-1275
```cpp
1275:     // x < 0 and x >= -6.1
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1276-1278
```cpp
1276:     return T{2} * exp(x * x) - erfcx_y100(T{400} / (T{4} - x));
1277:   }
1278: ); // erfcx_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1280-1301
```cpp
1280: const auto airy_ai_string = jiterator_stringify(
1281:     template<typename T>
1282:     T airy_ai_forward(T x) {
1283:         static const T AN[] = {
1284:                 +3.46538101525629032477e-01,
1285:                 +1.20075952739645805542e+01,
1286:                 +7.62796053615234516538e+01,
1287:                 +1.68089224934630576269e+02,
1288:                 +1.59756391350164413639e+02,
1289:                 +7.05360906840444183113e+01,
1290:                 +1.40264691163389668864e+01,
1291:                 +9.99999999999999995305e-01,
1292:         };
1293: 
1294:         static const T AD[] = {
1295:                 +5.67594532638770212846e-01,
1296:                 +1.47562562584847203173e+01,
1297:                 +8.45138970141474626562e+01,
1298:                 +1.77318088145400459522e+02,
1299:                 +1.64234692871529701831e+02,
1300:                 +7.14778400825575695274e+01,
1301:                 +1.40959135607834029598e+01,
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 1302-1303
```cpp
1302:                 +1.00000000000000000470e+00,
1303:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1305-1315
```cpp
1305:         static const T AFN[] = {
1306:                 -1.31696323418331795333e-01,
1307:                 -6.26456544431912369773e-01,
1308:                 -6.93158036036933542233e-01,
1309:                 -2.79779981545119124951e-01,
1310:                 -4.91900132609500318020e-02,
1311:                 -4.06265923594885404393e-03,
1312:                 -1.59276496239262096340e-04,
1313:                 -2.77649108155232920844e-06,
1314:                 -1.67787698489114633780e-08,
1315:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1317-1327
```cpp
1317:         static const T AFD[] = {
1318:                 +1.33560420706553243746e+01,
1319:                 +3.26825032795224613948e+01,
1320:                 +2.67367040941499554804e+01,
1321:                 +9.18707402907259625840e+00,
1322:                 +1.47529146771666414581e+00,
1323:                 +1.15687173795188044134e-01,
1324:                 +4.40291641615211203805e-03,
1325:                 +7.54720348287414296618e-05,
1326:                 +4.51850092970580378464e-07,
1327:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1329-1341
```cpp
1329:         static const T AGN[] = {
1330:                 +1.97339932091685679179e-02,
1331:                 +3.91103029615688277255e-01,
1332:                 +1.06579897599595591108e+00,
1333:                 +9.39169229816650230044e-01,
1334:                 +3.51465656105547619242e-01,
1335:                 +6.33888919628925490927e-02,
1336:                 +5.85804113048388458567e-03,
1337:                 +2.82851600836737019778e-04,
1338:                 +6.98793669997260967291e-06,
1339:                 +8.11789239554389293311e-08,
1340:                 +3.41551784765923618484e-10,
1341:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1343-1354
```cpp
1343:         static const T AGD[] = {
1344:                 +9.30892908077441974853e+00,
1345:                 +1.98352928718312140417e+01,
1346:                 +1.55646628932864612953e+01,
1347:                 +5.47686069422975497931e+00,
1348:                 +9.54293611618961883998e-01,
1349:                 +8.64580826352392193095e-02,
1350:                 +4.12656523824222607191e-03,
1351:                 +1.01259085116509135510e-04,
1352:                 +1.17166733214413521882e-06,
1353:                 +4.91834570062930015649e-09,
1354:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1356-1356
```cpp
1356:         int domain_flag = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1358-1358
```cpp
1358:         T ai;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1360-1362
```cpp
1360:         if (isinf(x)) {
1361:             return NAN;
1362:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1364-1366
```cpp
1364:         if (x > T(103.892)) {
1365:             return T(0.0);
1366:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1368-1370
```cpp
1368:         T f;
1369:         T g;
1370:         T k;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1372-1393
```cpp
1372:         if (x < T(-2.09)) {
1373:             T z = T(1.0) / (T(-2.0) * x * sqrt(-x) / T(3.0));
1374: 
1375:             T afn = 0.0;
1376: 
1377:             for (uint8_t index = 0; index <= 8; index++) {
1378:                 afn = afn * (z * z) + AFN[index];
1379:             }
1380: 
1381:             T afd = 0.0;
1382: 
1383:             for (uint8_t index = 0; index <= 8; index++) {
1384:                 afd = afd * (z * z) + AFD[index];
1385:             }
1386: 
1387:             T agn = 0.0;
1388: 
1389:             for (uint8_t index = 0; index <= 10 + 0; index++) {
1390:                 agn = agn * (z * z) + AGN[index];
1391:             }
1392: 
1393:             T agd = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1395-1397
```cpp
1395:             for (uint8_t index = 0; index <= 10 - 1; index++) {
1396:                 agd = agd * (z * z) + AGD[index];
1397:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1399-1399
```cpp
1399:             T t = T(-2.0) * x * sqrt(-x) / T(3.0) + T(0.25) * T(3.14159265358979323846);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1401-1402
```cpp
1401:             return T(5.64189583547756286948e-01) / sqrt(sqrt(-x)) * (sin(t) * (T(1.0) + z * z * afn / afd) - cos(t) * (z * agn / agd));
1402:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1404-1425
```cpp
1404:         if (x >= T(2.09)) {
1405:             domain_flag = 5;
1406: 
1407:             T zeta = T(2.0) * x * sqrt(x) / T(3.0);
1408: 
1409:             T an = 0.0;
1410: 
1411:             for (uint8_t index = 0; index <= 7; index++) {
1412:                 an = an * (T(1.0) / zeta) + AN[index];
1413:             }
1414: 
1415:             T ad = 0.0;
1416: 
1417:             for (uint8_t index = 0; index <= 7; index++) {
1418:                 ad = ad * (T(1.0) / zeta) + AD[index];
1419:             }
1420: 
1421:             ai = T(5.64189583547756286948e-01) * (an / ad) / (T(2.0) * sqrt(sqrt(x)) * exp(zeta));
1422: 
1423:             if (x > T(8.3203353)) {
1424:                 return ai;
1425:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1426-1426
```cpp
1426:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1428-1430
```cpp
1428:         f = 1.0;
1429:         g = x;
1430:         k = 1.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1432-1435
```cpp
1432:         T m = 1.0;
1433:         T n = x;
1434:         T t = 1.0;
1435:         T z = x * x * x;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1437-1451
```cpp
1437:         while (t > T(1.11022302462515654042e-16)) {
1438:             m *= z;
1439:             k += T(1.0);
1440:             m /= k;
1441:             n *= z;
1442:             k += T(1.0);
1443:             n /= k;
1444:             m /= k;
1445:             f += m;
1446:             k += T(1.0);
1447:             n /= k;
1448:             g += n;
1449: 
1450:             t = abs(m / f);
1451:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1453-1455
```cpp
1453:         if ((domain_flag & 1) == 0) {
1454:             return T(0.355028053887817239260) * f - T(0.258819403792806798405) * g;
1455:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1457-1459
```cpp
1457:         return ai;
1458:     } // T airy_ai(T x)
1459: ); // airy_ai_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1461-1482
```cpp
1461: const auto bessel_j0_string = jiterator_stringify(
1462:     template<typename T>
1463:     T bessel_j0_forward(T x) {
1464:         static const T PP[] = {
1465:                 +7.96936729297347051624e-04,
1466:                 +8.28352392107440799803e-02,
1467:                 +1.23953371646414299388e+00,
1468:                 +5.44725003058768775090e+00,
1469:                 +8.74716500199817011941e+00,
1470:                 +5.30324038235394892183e+00,
1471:                 +9.99999999999999997821e-01,
1472:         };
1473: 
1474:         static const T PQ[] = {
1475:                 +9.24408810558863637013e-04,
1476:                 +8.56288474354474431428e-02,
1477:                 +1.25352743901058953537e+00,
1478:                 +5.47097740330417105182e+00,
1479:                 +8.76190883237069594232e+00,
1480:                 +5.30605288235394617618e+00,
1481:                 +1.00000000000000000218e+00,
1482:         };
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 1484-1493
```cpp
1484:         static const T QP[] = {
1485:                 -1.13663838898469149931e-02,
1486:                 -1.28252718670509318512e+00,
1487:                 -1.95539544257735972385e+01,
1488:                 -9.32060152123768231369e+01,
1489:                 -1.77681167980488050595e+02,
1490:                 -1.47077505154951170175e+02,
1491:                 -5.14105326766599330220e+01,
1492:                 -6.05014350600728481186e+00,
1493:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1495-1503
```cpp
1495:         static const T QQ[] = {
1496:                 +6.43178256118178023184e+01,
1497:                 +8.56430025976980587198e+02,
1498:                 +3.88240183605401609683e+03,
1499:                 +7.24046774195652478189e+03,
1500:                 +5.93072701187316984827e+03,
1501:                 +2.06209331660327847417e+03,
1502:                 +2.42005740240291393179e+02,
1503:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1505-1510
```cpp
1505:         static const T RP[] = {
1506:                 -4.79443220978201773821e+09,
1507:                 +1.95617491946556577543e+12,
1508:                 -2.49248344360967716204e+14,
1509:                 +9.70862251047306323952e+15,
1510:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1512-1521
```cpp
1512:         static const T RQ[] = {
1513:                 +4.99563147152651017219e+02,
1514:                 +1.73785401676374683123e+05,
1515:                 +4.84409658339962045305e+07,
1516:                 +1.11855537045356834862e+10,
1517:                 +2.11277520115489217587e+12,
1518:                 +3.10518229857422583814e+14,
1519:                 +3.18121955943204943306e+16,
1520:                 +1.71086294081043136091e+18,
1521:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1523-1525
```cpp
1523:         if (x < T(0)) {
1524:             x = -x;
1525:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1527-1545
```cpp
1527:         if (x <= T(5.0)) {
1528:             if (x < T(0.00001)) {
1529:                 return T(1.0) - x * x / T(4.0);
1530:             }
1531: 
1532:             T rp = 0.0;
1533: 
1534:             for (uint8_t index = 0; index <= 3; index++) {
1535:                 rp = rp * (x * x) + RP[index];
1536:             }
1537: 
1538:             T rq = 0.0;
1539: 
1540:             for (uint8_t index = 0; index <= 7; index++) {
1541:                 rq = rq * (x * x) + RQ[index];
1542:             }
1543: 
1544:             return (x * x - T(5.78318596294678452118e+00)) * (x * x - T(3.04712623436620863991e+01)) * rp / rq;
1545:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1547-1547
```cpp
1547:         T pp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1549-1551
```cpp
1549:         for (uint8_t index = 0; index <= 6; index++) {
1550:             pp = pp * (T(25.0) / (x * x)) + PP[index];
1551:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1553-1553
```cpp
1553:         T pq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1555-1557
```cpp
1555:         for (uint8_t index = 0; index <= 6; index++) {
1556:             pq = pq * (T(25.0) / (x * x)) + PQ[index];
1557:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1559-1559
```cpp
1559:         T qp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1561-1563
```cpp
1561:         for (uint8_t index = 0; index <= 7; index++) {
1562:             qp = qp * (T(25.0) / (x * x)) + QP[index];
1563:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1565-1565
```cpp
1565:         T qq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1567-1569
```cpp
1567:         for (uint8_t index = 0; index <= 6; index++) {
1568:             qq = qq * (T(25.0) / (x * x)) + QQ[index];
1569:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1571-1573
```cpp
1571:         return (pp / pq * cos(x - T(0.785398163397448309615660845819875721)) - T(5.0) / x * (qp / qq) * sin(x - T(0.785398163397448309615660845819875721))) * T(0.797884560802865355879892119868763737) / sqrt(x);
1572:     } // bessel_j0_forward(T x)
1573: ); // bessel_j0_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1575-1596
```cpp
1575: const auto bessel_y0_string = bessel_j0_string + jiterator_stringify(
1576:     template<typename T>
1577:     T bessel_y0_forward(T x) {
1578:         static const T PP[] = {
1579:                 +7.96936729297347051624e-04,
1580:                 +8.28352392107440799803e-02,
1581:                 +1.23953371646414299388e+00,
1582:                 +5.44725003058768775090e+00,
1583:                 +8.74716500199817011941e+00,
1584:                 +5.30324038235394892183e+00,
1585:                 +9.99999999999999997821e-01,
1586:         };
1587: 
1588:         static const T PQ[] = {
1589:                 +9.24408810558863637013e-04,
1590:                 +8.56288474354474431428e-02,
1591:                 +1.25352743901058953537e+00,
1592:                 +5.47097740330417105182e+00,
1593:                 +8.76190883237069594232e+00,
1594:                 +5.30605288235394617618e+00,
1595:                 +1.00000000000000000218e+00,
1596:         };
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 1598-1607
```cpp
1598:         static const T QP[] = {
1599:                 -1.13663838898469149931e-02,
1600:                 -1.28252718670509318512e+00,
1601:                 -1.95539544257735972385e+01,
1602:                 -9.32060152123768231369e+01,
1603:                 -1.77681167980488050595e+02,
1604:                 -1.47077505154951170175e+02,
1605:                 -5.14105326766599330220e+01,
1606:                 -6.05014350600728481186e+00,
1607:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1609-1617
```cpp
1609:         static const T QQ[] = {
1610:                 +6.43178256118178023184e+01,
1611:                 +8.56430025976980587198e+02,
1612:                 +3.88240183605401609683e+03,
1613:                 +7.24046774195652478189e+03,
1614:                 +5.93072701187316984827e+03,
1615:                 +2.06209331660327847417e+03,
1616:                 +2.42005740240291393179e+02,
1617:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1619-1628
```cpp
1619:         static const T YP[] = {
1620:                 +1.55924367855235737965e+04,
1621:                 -1.46639295903971606143e+07,
1622:                 +5.43526477051876500413e+09,
1623:                 -9.82136065717911466409e+11,
1624:                 +8.75906394395366999549e+13,
1625:                 -3.46628303384729719441e+15,
1626:                 +4.42733268572569800351e+16,
1627:                 -1.84950800436986690637e+16,
1628:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1630-1638
```cpp
1630:         static const T YQ[] = {
1631:                 +1.04128353664259848412e+03,
1632:                 +6.26107330137134956842e+05,
1633:                 +2.68919633393814121987e+08,
1634:                 +8.64002487103935000337e+10,
1635:                 +2.02979612750105546709e+13,
1636:                 +3.17157752842975028269e+15,
1637:                 +2.50596256172653059228e+17,
1638:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1640-1661
```cpp
1640:         if (x <= T(5.0)) {
1641:             if (x == T(0.0)) {
1642:                 return NEG_INFINITY;
1643:             }
1644: 
1645:             if (x < T(0.0)) {
1646:                 NAN;
1647:             }
1648: 
1649:             T yp = 0.0;
1650: 
1651:             for (uint8_t index = 0; index <= 7; index++) {
1652:                 yp = yp * (x * x) + YP[index];
1653:             }
1654: 
1655:             T yq = 0.0;
1656: 
1657:             for (uint8_t index = 0; index <= 6; index++) {
1658:                 yq = yq * (x * x) + YQ[index];
1659:             }
1660: 
1661:             return yp / yq + (T(0.636619772367581343075535053490057448) * log(x) * bessel_j0_forward(x));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1662-1662
```cpp
1662:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1664-1664
```cpp
1664:         T pp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1666-1668
```cpp
1666:         for (uint8_t index = 0; index <= 6; index++) {
1667:             pp = pp * (T(25.0) / (x * x)) + PP[index];
1668:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1670-1670
```cpp
1670:         T pq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1672-1674
```cpp
1672:         for (uint8_t index = 0; index <= 6; index++) {
1673:             pq = pq * (T(25.0) / (x * x)) + PQ[index];
1674:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1676-1676
```cpp
1676:         T qp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1678-1680
```cpp
1678:         for (uint8_t index = 0; index <= 7; index++) {
1679:             qp = qp * (T(25.0) / (x * x)) + QP[index];
1680:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1682-1682
```cpp
1682:         T qq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1684-1686
```cpp
1684:         for (uint8_t index = 0; index <= 6; index++) {
1685:             qq = qq * (T(25.0) / (x * x)) + QQ[index];
1686:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1688-1690
```cpp
1688:         return (pp / pq * sin(x - T(0.785398163397448309615660845819875721)) + T(5.0) / x * (qp / qq) * cos(x - T(0.785398163397448309615660845819875721))) * T(0.797884560802865355879892119868763737) / sqrt(x);
1689:     } // bessel_y0_forward(T x)
1690: ); // bessel_y0_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1692-1713
```cpp
1692: const auto bessel_j1_string = jiterator_stringify(
1693:     template<typename T>
1694:     T bessel_j1_forward(T x) {
1695:         static const T PP[] = {
1696:                 +7.62125616208173112003e-04,
1697:                 +7.31397056940917570436e-02,
1698:                 +1.12719608129684925192e+00,
1699:                 +5.11207951146807644818e+00,
1700:                 +8.42404590141772420927e+00,
1701:                 +5.21451598682361504063e+00,
1702:                 +1.00000000000000000254e+00,
1703:         };
1704: 
1705:         static const T PQ[] = {
1706:                 +5.71323128072548699714e-04,
1707:                 +6.88455908754495404082e-02,
1708:                 +1.10514232634061696926e+00,
1709:                 +5.07386386128601488557e+00,
1710:                 +8.39985554327604159757e+00,
1711:                 +5.20982848682361821619e+00,
1712:                 +9.99999999999999997461e-01,
1713:         };
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 1715-1724
```cpp
1715:         static const T QP[] = {
1716:                 +5.10862594750176621635e-02,
1717:                 +4.98213872951233449420e+00,
1718:                 +7.58238284132545283818e+01,
1719:                 +3.66779609360150777800e+02,
1720:                 +7.10856304998926107277e+02,
1721:                 +5.97489612400613639965e+02,
1722:                 +2.11688757100572135698e+02,
1723:                 +2.52070205858023719784e+01,
1724:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1726-1734
```cpp
1726:         static const T QQ[] = {
1727:                 +7.42373277035675149943e+01,
1728:                 +1.05644886038262816351e+03,
1729:                 +4.98641058337653607651e+03,
1730:                 +9.56231892404756170795e+03,
1731:                 +7.99704160447350683650e+03,
1732:                 +2.82619278517639096600e+03,
1733:                 +3.36093607810698293419e+02,
1734:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1736-1741
```cpp
1736:         static const T RP[] = {
1737:                 -8.99971225705559398224e+08,
1738:                 +4.52228297998194034323e+11,
1739:                 -7.27494245221818276015e+13,
1740:                 +3.68295732863852883286e+15,
1741:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1743-1752
```cpp
1743:         static const T RQ[] = {
1744:                 +6.20836478118054335476e+02,
1745:                 +2.56987256757748830383e+05,
1746:                 +8.35146791431949253037e+07,
1747:                 +2.21511595479792499675e+10,
1748:                 +4.74914122079991414898e+12,
1749:                 +7.84369607876235854894e+14,
1750:                 +8.95222336184627338078e+16,
1751:                 +5.32278620332680085395e+18,
1752:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1754-1756
```cpp
1754:         if (x < T(0.0)) {
1755:             return -bessel_j1_forward(-x);
1756:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1758-1772
```cpp
1758:         if (x <= T(5.0)) {
1759:             T rp = 0.0;
1760: 
1761:             for (uint8_t index = 0; index <= 3; index++) {
1762:                 rp = rp * (x * x) + RP[index];
1763:             }
1764: 
1765:             T rq = 0.0;
1766: 
1767:             for (uint8_t index = 0; index <= 7; index++) {
1768:                 rq = rq * (x * x) + RQ[index];
1769:             }
1770: 
1771:             return rp / rq * x * (x * x - T(1.46819706421238932572e+01)) * (x * x - T(4.92184563216946036703e+01));
1772:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1774-1774
```cpp
1774:         T pp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1776-1778
```cpp
1776:         for (uint8_t index = 0; index <= 6; index++) {
1777:             pp = pp * (T(5.0) / x * (T(5.0) / x)) + PP[index];
1778:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1780-1780
```cpp
1780:         T pq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1782-1784
```cpp
1782:         for (uint8_t index = 0; index <= 6; index++) {
1783:             pq = pq * (T(5.0) / x * (T(5.0) / x)) + PQ[index];
1784:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1786-1786
```cpp
1786:         T qp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1788-1790
```cpp
1788:         for (uint8_t index = 0; index <= 7; index++) {
1789:             qp = qp * (T(5.0) / x * (T(5.0) / x)) + QP[index];
1790:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1792-1792
```cpp
1792:         T qq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1794-1796
```cpp
1794:         for (uint8_t index = 0; index <= 6; index++) {
1795:             qq = qq * (T(5.0) / x * (T(5.0) / x)) + QQ[index];
1796:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1798-1800
```cpp
1798:         return (pp / pq * cos(x - T(2.356194490192344928846982537459627163)) - T(5.0) / x * (qp / qq) * sin(x - T(2.356194490192344928846982537459627163))) * T(0.797884560802865355879892119868763737) / sqrt(x);
1799:     } // bessel_j1_forward(T x)
1800: ); // bessel_j1_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1802-1823
```cpp
1802: const auto bessel_y1_string = bessel_j1_string + jiterator_stringify(
1803:     template<typename T>
1804:     T bessel_y1_forward(T x) {
1805:         static const T PP[] = {
1806:                 +7.62125616208173112003e-04,
1807:                 +7.31397056940917570436e-02,
1808:                 +1.12719608129684925192e+00,
1809:                 +5.11207951146807644818e+00,
1810:                 +8.42404590141772420927e+00,
1811:                 +5.21451598682361504063e+00,
1812:                 +1.00000000000000000254e+00,
1813:         };
1814: 
1815:         static const T PQ[] = {
1816:                 +5.71323128072548699714e-04,
1817:                 +6.88455908754495404082e-02,
1818:                 +1.10514232634061696926e+00,
1819:                 +5.07386386128601488557e+00,
1820:                 +8.39985554327604159757e+00,
1821:                 +5.20982848682361821619e+00,
1822:                 +9.99999999999999997461e-01,
1823:         };
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 1825-1834
```cpp
1825:         static const T QP[] = {
1826:                 +5.10862594750176621635e-02,
1827:                 +4.98213872951233449420e+00,
1828:                 +7.58238284132545283818e+01,
1829:                 +3.66779609360150777800e+02,
1830:                 +7.10856304998926107277e+02,
1831:                 +5.97489612400613639965e+02,
1832:                 +2.11688757100572135698e+02,
1833:                 +2.52070205858023719784e+01,
1834:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1836-1844
```cpp
1836:         static const T QQ[] = {
1837:                 +7.42373277035675149943e+01,
1838:                 +1.05644886038262816351e+03,
1839:                 +4.98641058337653607651e+03,
1840:                 +9.56231892404756170795e+03,
1841:                 +7.99704160447350683650e+03,
1842:                 +2.82619278517639096600e+03,
1843:                 +3.36093607810698293419e+02,
1844:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1846-1853
```cpp
1846:         static const T YP[] = {
1847:                 +1.26320474790178026440e+09,
1848:                 -6.47355876379160291031e+11,
1849:                 +1.14509511541823727583e+14,
1850:                 -8.12770255501325109621e+15,
1851:                 +2.02439475713594898196e+17,
1852:                 -7.78877196265950026825e+17,
1853:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1855-1864
```cpp
1855:         static const T YQ[] = {
1856:                 +5.94301592346128195359e+02,
1857:                 +2.35564092943068577943e+05,
1858:                 +7.34811944459721705660e+07,
1859:                 +1.87601316108706159478e+10,
1860:                 +3.88231277496238566008e+12,
1861:                 +6.20557727146953693363e+14,
1862:                 +6.87141087355300489866e+16,
1863:                 +3.97270608116560655612e+18,
1864:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1866-1887
```cpp
1866:         if (x <= T(5.0)) {
1867:             if (x == T(0.0)) {
1868:                 return NEG_INFINITY;
1869:             }
1870: 
1871:             if (x <= T(0.0)) {
1872:                 return NAN;
1873:             }
1874: 
1875:             T yp = 0.0;
1876: 
1877:             for (uint8_t index = 0; index <= 5; index++) {
1878:                 yp = yp * (x * x) + YP[index];
1879:             }
1880: 
1881:             T yq = 0.0;
1882: 
1883:             for (uint8_t index = 0; index <= 7; index++) {
1884:                 yq = yq * (x * x) + YQ[index];
1885:             }
1886: 
1887:             return x * (yp / yq) + (T(0.636619772367581343075535053490057448) * (bessel_j1_forward(x) * log(x) - T(1.0) / x));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1888-1888
```cpp
1888:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1890-1890
```cpp
1890:         T pp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1892-1894
```cpp
1892:         for (uint8_t index = 0; index <= 6; index++) {
1893:             pp = pp * (T(5.0) / x * (T(5.0) / x)) + PP[index];
1894:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1896-1896
```cpp
1896:         T pq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1898-1900
```cpp
1898:         for (uint8_t index = 0; index <= 6; index++) {
1899:             pq = pq * (T(5.0) / x * (T(5.0) / x)) + PQ[index];
1900:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1902-1902
```cpp
1902:         T qp = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1904-1906
```cpp
1904:         for (uint8_t index = 0; index <= 7; index++) {
1905:             qp = qp * (T(5.0) / x * (T(5.0) / x)) + QP[index];
1906:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1908-1908
```cpp
1908:         T qq = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1910-1912
```cpp
1910:         for (uint8_t index = 0; index <= 6; index++) {
1911:             qq = qq * (T(5.0) / x * (T(5.0) / x)) + QQ[index];
1912:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1914-1916
```cpp
1914:         return (pp / pq * sin(x - T(2.356194490192344928846982537459627163)) + T(5.0) / x * (qp / qq) * cos(x - T(2.356194490192344928846982537459627163))) * T(0.797884560802865355879892119868763737) / sqrt(x);
1915:     } // bessel_y1_forward(T x)
1916: ); // bessel_y1_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1918-1939
```cpp
1918: const auto chebyshev_polynomial_t_string = jiterator_stringify(
1919:     template<typename T>
1920:     T chebyshev_polynomial_t_forward(T x, int64_t n) {
1921:         if (n < 0) {
1922:             return T(0.0);
1923:         }
1924: 
1925:         if (abs(x) == T(1.0)) {
1926:             if (x > T(0.0) || n % 2 == 0) {
1927:                 return T(1.0);
1928:             }
1929: 
1930:             return T(-1.0);
1931:         }
1932: 
1933:         if ((n > 6) && (abs(x) < T(1.0))) {
1934:             return cos(n * acos(x));
1935:         }
1936: 
1937:         if (n == 0) {
1938:             return T(1.0);
1939:         }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 1941-1943
```cpp
1941:         if (n == 1) {
1942:             return x;
1943:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1945-1947
```cpp
1945:         T p = T(1.0);
1946:         T q = x;
1947:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1949-1953
```cpp
1949:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
1950:             r = (x + x) * q - p;
1951:             p = q;
1952:             q = r;
1953:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1955-1956
```cpp
1955:         return r;
1956:     } // chebyshev_polynomial_t_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1958-1962
```cpp
1958:     template<typename T>
1959:     T chebyshev_polynomial_t_forward(T x, T n) {
1960:         return chebyshev_polynomial_t_forward(x, static_cast<int64_t>(n));
1961:     } // chebyshev_polynomial_t_forward(T x, T n)
1962: ); // chebyshev_polynomial_t_string
```
- EN: This block defines or continues the implementation of `chebyshev_polynomial_t_forward`.
- CN: 该代码块定义或继续实现 `chebyshev_polynomial_t_forward`。

### Lines 1964-1985
```cpp
1964: const auto chebyshev_polynomial_u_string = jiterator_stringify(
1965:     template<typename T>
1966:     T chebyshev_polynomial_u_forward(T x, int64_t n) {
1967:         if (n < 0) {
1968:             return T(0.0);
1969:         }
1970: 
1971:         if (abs(x) == T(1.0)) {
1972:             if (x > T(0.0) || n % 2 == 0) {
1973:                 return n + 1;
1974:             }
1975: 
1976:             return -(n + 1);
1977:         }
1978: 
1979:         if ((n > 8) && (abs(x) < T(1.0))) {
1980:             if (sin(acos(x)) != T(0.0)) {
1981:                 return sin((n + 1) * acos(x)) / sin(acos(x));
1982:             }
1983: 
1984:             return (n + 1) * cos((n + 1) * acos(x)) / x;
1985:         }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 1987-1989
```cpp
1987:         if (n == 0) {
1988:             return T(1.0);
1989:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1991-1993
```cpp
1991:         if (n == 1) {
1992:             return x + x;
1993:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1995-1997
```cpp
1995:         T p = T(1.0);
1996:         T q = x + x;
1997:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1999-2003
```cpp
1999:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
2000:             r = (x + x) * q - p;
2001:             p = q;
2002:             q = r;
2003:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2005-2006
```cpp
2005:         return r;
2006:     } // chebyshev_polynomial_u_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2008-2012
```cpp
2008:     template<typename T>
2009:     T chebyshev_polynomial_u_forward(T x, T n) {
2010:         return chebyshev_polynomial_u_forward(x, static_cast<int64_t>(n));
2011:     } // chebyshev_polynomial_u_forward(T x, T n)
2012: ); // chebyshev_polynomial_u_string
```
- EN: This block defines or continues the implementation of `chebyshev_polynomial_u_forward`.
- CN: 该代码块定义或继续实现 `chebyshev_polynomial_u_forward`。

### Lines 2014-2035
```cpp
2014: const auto chebyshev_polynomial_v_string = jiterator_stringify(
2015:     template<typename T>
2016:     T chebyshev_polynomial_v_forward(T x, int64_t n) {
2017:         if (n < 0) {
2018:             return T(0.0);
2019:         }
2020: 
2021:         if (abs(x) == T(1.0)) {
2022:             if (x > T(0.0)) {
2023:                 return T(1.0);
2024:             }
2025: 
2026:             if (n % 2 == 0) {
2027:                 return n + n + 1;
2028:             }
2029: 
2030:             return -(n + n + 1);
2031:         }
2032: 
2033:         if ((n > 8) && (abs(x) < T(1.0))) {
2034:             if (sin(acos(x) / T(2.0)) != T(1.0)) {
2035:                 return cos((n + T(0.5)) * acos(x)) / cos(acos(x) / T(2.0));
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2036-2036
```cpp
2036:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2038-2040
```cpp
2038:             if (n % 2 == 0) {
2039:                 return n + n + 1;
2040:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2042-2043
```cpp
2042:             return -(n + n + 1);
2043:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2045-2047
```cpp
2045:         if (n == 0) {
2046:             return T(1.0);
2047:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2049-2051
```cpp
2049:         if (n == 1) {
2050:             return x + x - T(1.0);
2051:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2053-2055
```cpp
2053:         T p = T(1.0);
2054:         T q = x + x - T(1.0);
2055:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2057-2061
```cpp
2057:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
2058:             r = (x + x) * q - p;
2059:             p = q;
2060:             q = r;
2061:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2063-2064
```cpp
2063:         return r;
2064:     } // chebyshev_polynomial_v_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2066-2070
```cpp
2066:     template<typename T>
2067:     T chebyshev_polynomial_v_forward(T x, T n) {
2068:         return chebyshev_polynomial_v_forward(x, static_cast<int64_t>(n));
2069:     } // chebyshev_polynomial_v_forward(T x, T n)
2070: ); // chebyshev_polynomial_v_string
```
- EN: This block defines or continues the implementation of `chebyshev_polynomial_v_forward`.
- CN: 该代码块定义或继续实现 `chebyshev_polynomial_v_forward`。

### Lines 2072-2093
```cpp
2072: const auto chebyshev_polynomial_w_string = jiterator_stringify(
2073:     template<typename T>
2074:     T chebyshev_polynomial_w_forward(T x, int64_t n) {
2075:         if (n < 0) {
2076:             return T(0.0);
2077:         }
2078: 
2079:         if (abs(x) == T(1.0)) {
2080:             if (x > T(0.0)) {
2081:                 return n + n + 1;
2082:             }
2083: 
2084:             if (n % 2 == 0) {
2085:                 return T(1.0);
2086:             }
2087: 
2088:             return T(-1.0);
2089:         }
2090: 
2091:         if ((n > 8) && (abs(x) < T(1.0))) {
2092:             if (cos(acos(x) / T(2.0)) != T(1.0)) {
2093:                 return sin((n + T(0.5)) * acos(x)) / sin(acos(x) / T(2.0));
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2094-2094
```cpp
2094:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2096-2098
```cpp
2096:             if (x > T(0.0)) {
2097:                 return n + n + 1;
2098:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2100-2102
```cpp
2100:             if (n % 2 == 0) {
2101:                 return T(1.0);
2102:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2104-2105
```cpp
2104:             return T(-1.0);
2105:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2107-2109
```cpp
2107:         if (n == 0) {
2108:             return T(1.0);
2109:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2111-2113
```cpp
2111:         if (n == 1) {
2112:             return x + x + T(1.0);
2113:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2115-2117
```cpp
2115:         T p = T(1.0);
2116:         T q = x + x + T(1.0);
2117:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2119-2123
```cpp
2119:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
2120:             r = (x + x) * q - p;
2121:             p = q;
2122:             q = r;
2123:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2125-2126
```cpp
2125:         return r;
2126:     } // chebyshev_polynomial_w_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2128-2132
```cpp
2128:     template<typename T>
2129:     T chebyshev_polynomial_w_forward(T x, T n) {
2130:         return chebyshev_polynomial_w_forward(x, static_cast<int64_t>(n));
2131:     } // chebyshev_polynomial_w_forward(T x, T n)
2132: ); // chebyshev_polynomial_w_string
```
- EN: This block defines or continues the implementation of `chebyshev_polynomial_w_forward`.
- CN: 该代码块定义或继续实现 `chebyshev_polynomial_w_forward`。

### Lines 2134-2144
```cpp
2134: const auto hermite_polynomial_h_string = jiterator_stringify(
2135:     template<typename T>
2136:     unsigned short getHermitianLimit() {
2137:         if (sizeof(T) <= sizeof(float)) {
2138:             return 128;
2139:         } else if (sizeof(T) <= sizeof(double)) {
2140:             return 512;
2141:         } else {
2142:             return 1024;
2143:         }
2144:     }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2146-2166
```cpp
2146:     template<typename T>
2147:     T hermite_polynomial_h_forward(T x, int64_t n) {
2148:         if (n < 0) {
2149:             return T(0.0);
2150:         }
2151: 
2152:         if (n == 0) {
2153:             return T(1.0);
2154:         }
2155: 
2156:         if (n == 1) {
2157:             return x + x;
2158:         }
2159: 
2160:         if (n > getHermitianLimit<T>()) {
2161:             return NAN;
2162:         }
2163: 
2164:         T p = T(1.0);
2165:         T q = x + x;
2166:         T r = T(0.0);
```
- EN: This block defines or continues the implementation of `hermite_polynomial_h_forward`.
- CN: 该代码块定义或继续实现 `hermite_polynomial_h_forward`。

### Lines 2168-2172
```cpp
2168:         for (int64_t k = 2; k < n + n; k += 2) {
2169:             r = (x + x) * q - k * p;
2170:             p = q;
2171:             q = r;
2172:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2174-2175
```cpp
2174:         return r;
2175:     } // hermite_polynomial_h_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2177-2181
```cpp
2177:     template<typename T>
2178:     T hermite_polynomial_h_forward(T x, T n) {
2179:         return hermite_polynomial_h_forward(x, static_cast<int64_t>(n));
2180:     } // hermite_polynomial_h_forward(T x, T n)
2181: ); // hermite_polynomial_h_string
```
- EN: This block defines or continues the implementation of `hermite_polynomial_h_forward`.
- CN: 该代码块定义或继续实现 `hermite_polynomial_h_forward`。

### Lines 2183-2193
```cpp
2183: const auto hermite_polynomial_he_string = jiterator_stringify(
2184:     template<typename T>
2185:     unsigned short getHermitianLimit() {
2186:         if (sizeof(T) <= sizeof(float)) {
2187:             return 128;
2188:         } else if (sizeof(T) <= sizeof(double)) {
2189:             return 512;
2190:         } else {
2191:             return 1024;
2192:         }
2193:     }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2195-2215
```cpp
2195:     template<typename T>
2196:     T hermite_polynomial_he_forward(T x, int64_t n) {
2197:         if (n < 0) {
2198:             return T(0.0);
2199:         }
2200: 
2201:         if (n == 0) {
2202:             return T(1.0);
2203:         }
2204: 
2205:         if (n == 1) {
2206:             return x;
2207:         }
2208: 
2209:         if (n > getHermitianLimit<T>()) {
2210:             return NAN;
2211:         }
2212: 
2213:         T p = T(1.0);
2214:         T q = x;
2215:         T r;
```
- EN: This block defines or continues the implementation of `hermite_polynomial_he_forward`.
- CN: 该代码块定义或继续实现 `hermite_polynomial_he_forward`。

### Lines 2217-2221
```cpp
2217:         for (int64_t k = 1; k < n; k++) {
2218:             r = x * q - k * p;
2219:             p = q;
2220:             q = r;
2221:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2223-2224
```cpp
2223:         return r;
2224:     } // hermite_polynomial_he_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2226-2230
```cpp
2226:     template<typename T>
2227:     T hermite_polynomial_he_forward(T x, T n) {
2228:         return hermite_polynomial_he_forward(x, static_cast<int64_t>(n));
2229:     } // hermite_polynomial_he_forward(T x, T n)
2230: ); // hermite_polynomial_he_string
```
- EN: This block defines or continues the implementation of `hermite_polynomial_he_forward`.
- CN: 该代码块定义或继续实现 `hermite_polynomial_he_forward`。

### Lines 2232-2253
```cpp
2232: const auto laguerre_polynomial_l_string = jiterator_stringify(
2233:     template<typename T>
2234:     T laguerre_polynomial_l_forward(T x, int64_t n) {
2235:         if (n < 0) {
2236:             return T(0.0);
2237:         }
2238: 
2239:         if (abs(x) == T(0.0)) {
2240:             return T(1.0);
2241:         }
2242: 
2243:         if (n == 0) {
2244:             return T(1.0);
2245:         }
2246: 
2247:         if (n == 1) {
2248:             return T(1.0) - x;
2249:         }
2250: 
2251:         T p = T(1.0);
2252:         T q = T(1.0) - x;
2253:         T r;
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2255-2259
```cpp
2255:         for (int64_t k = 1; (k < n) && !isnan(q); k++) {
2256:             r = (((k + k) + (T(1.0) - x)) * q - k * p) / (k + 1);
2257:             p = q;
2258:             q = r;
2259:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2261-2262
```cpp
2261:         return r;
2262:     } // laguerre_polynomial_l_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2264-2268
```cpp
2264:     template<typename T>
2265:     T laguerre_polynomial_l_forward(T x, T n) {
2266:         return laguerre_polynomial_l_forward(x, static_cast<int64_t>(n));
2267:     } // laguerre_polynomial_l_forward(T x, T n)
2268: ); // laguerre_polynomial_l_string
```
- EN: This block defines or continues the implementation of `laguerre_polynomial_l_forward`.
- CN: 该代码块定义或继续实现 `laguerre_polynomial_l_forward`。

### Lines 2270-2291
```cpp
2270: const auto legendre_polynomial_p_string = jiterator_stringify(
2271:     template<typename T>
2272:     T legendre_polynomial_p_forward(T x, int64_t n) {
2273:         if (n < 0) {
2274:             return T(0.0);
2275:         }
2276: 
2277:         if (abs(x) == T(1.0)) {
2278:             if (x > T(0.0) || n % 2 == 0) {
2279:                 return T(1.0);
2280:             }
2281: 
2282:             return T(-1.0);
2283:         }
2284: 
2285:         if (n == 0) {
2286:             return T(1.0);
2287:         }
2288: 
2289:         if (n == 1) {
2290:             return x;
2291:         }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2293-2295
```cpp
2293:         T p = T(1.0);
2294:         T q = x;
2295:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2297-2301
```cpp
2297:         for (int64_t k = 1; (k < n) && !isnan(q); k++) {
2298:             r = ((k + k + 1) * x * q - k * p) / (k + 1);
2299:             p = q;
2300:             q = r;
2301:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2303-2304
```cpp
2303:         return r;
2304:     } // legendre_polynomial_p_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2306-2310
```cpp
2306:     template<typename T>
2307:     T legendre_polynomial_p_forward(T x, T n) {
2308:         return legendre_polynomial_p_forward(x, static_cast<int64_t>(n));
2309:     } // legendre_polynomial_p_forward(T x, T n)
2310: ); // legendre_polynomial_p_string
```
- EN: This block defines or continues the implementation of `legendre_polynomial_p_forward`.
- CN: 该代码块定义或继续实现 `legendre_polynomial_p_forward`。

### Lines 2312-2333
```cpp
2312: const auto modified_bessel_i0_string = jiterator_stringify(
2313:     template<typename T>
2314:     T modified_bessel_i0_forward(T x) {
2315:         static const T A[] = {
2316:                 -4.41534164647933937950e-18,
2317:                 +3.33079451882223809783e-17,
2318:                 -2.43127984654795469359e-16,
2319:                 +1.71539128555513303061e-15,
2320:                 -1.16853328779934516808e-14,
2321:                 +7.67618549860493561688e-14,
2322:                 -4.85644678311192946090e-13,
2323:                 +2.95505266312963983461e-12,
2324:                 -1.72682629144155570723e-11,
2325:                 +9.67580903537323691224e-11,
2326:                 -5.18979560163526290666e-10,
2327:                 +2.65982372468238665035e-09,
2328:                 -1.30002500998624804212e-08,
2329:                 +6.04699502254191894932e-08,
2330:                 -2.67079385394061173391e-07,
2331:                 +1.11738753912010371815e-06,
2332:                 -4.41673835845875056359e-06,
2333:                 +1.64484480707288970893e-05,
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2334-2346
```cpp
2334:                 -5.75419501008210370398e-05,
2335:                 +1.88502885095841655729e-04,
2336:                 -5.76375574538582365885e-04,
2337:                 +1.63947561694133579842e-03,
2338:                 -4.32430999505057594430e-03,
2339:                 +1.05464603945949983183e-02,
2340:                 -2.37374148058994688156e-02,
2341:                 +4.93052842396707084878e-02,
2342:                 -9.49010970480476444210e-02,
2343:                 +1.71620901522208775349e-01,
2344:                 -3.04682672343198398683e-01,
2345:                 +6.76795274409476084995e-01,
2346:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2348-2369
```cpp
2348:         static const T B[] = {
2349:                 -7.23318048787475395456e-18,
2350:                 -4.83050448594418207126e-18,
2351:                 +4.46562142029675999901e-17,
2352:                 +3.46122286769746109310e-17,
2353:                 -2.82762398051658348494e-16,
2354:                 -3.42548561967721913462e-16,
2355:                 +1.77256013305652638360e-15,
2356:                 +3.81168066935262242075e-15,
2357:                 -9.55484669882830764870e-15,
2358:                 -4.15056934728722208663e-14,
2359:                 +1.54008621752140982691e-14,
2360:                 +3.85277838274214270114e-13,
2361:                 +7.18012445138366623367e-13,
2362:                 -1.79417853150680611778e-12,
2363:                 -1.32158118404477131188e-11,
2364:                 -3.14991652796324136454e-11,
2365:                 +1.18891471078464383424e-11,
2366:                 +4.94060238822496958910e-10,
2367:                 +3.39623202570838634515e-09,
2368:                 +2.26666899049817806459e-08,
2369:                 +2.04891858946906374183e-07,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2370-2374
```cpp
2370:                 +2.89137052083475648297e-06,
2371:                 +6.88975834691682398426e-05,
2372:                 +3.36911647825569408990e-03,
2373:                 +8.04490411014108831608e-01,
2374:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2376-2377
```cpp
2376:         T p;
2377:         T q = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2379-2389
```cpp
2379:         if (abs(x) <= T(8.0)) {
2380:             T a = A[0];
2381: 
2382:             for (uint8_t index = 1; index < 30; index++) {
2383:                 p = q;
2384:                 q = a;
2385:                 a = ((abs(x) / T(2.0)) - T(2.0)) * q - p + A[index];
2386:             }
2387: 
2388:             return exp(abs(x)) * (T(0.5) * (a - p));
2389:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2391-2391
```cpp
2391:         T b = B[0];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2393-2397
```cpp
2393:         for (uint8_t index = 1; index < 25; index++) {
2394:             p = q;
2395:             q = b;
2396:             b = (T(32.0) / abs(x) - T(2.0)) * q - p + B[index];
2397:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2399-2401
```cpp
2399:         return exp(abs(x)) * (T(0.5) * (b - p)) / sqrt(abs(x));
2400:     } // modified_bessel_i0_forward(T x)
2401: ); // modified_bessel_i0_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2403-2424
```cpp
2403: const auto modified_bessel_i1_string = jiterator_stringify(
2404:     template<typename T>
2405:     T modified_bessel_i1_forward(T x) {
2406:         static const T A[] = {
2407:                 +2.77791411276104639959e-18,
2408:                 -2.11142121435816608115e-17,
2409:                 +1.55363195773620046921e-16,
2410:                 -1.10559694773538630805e-15,
2411:                 +7.60068429473540693410e-15,
2412:                 -5.04218550472791168711e-14,
2413:                 +3.22379336594557470981e-13,
2414:                 -1.98397439776494371520e-12,
2415:                 +1.17361862988909016308e-11,
2416:                 -6.66348972350202774223e-11,
2417:                 +3.62559028155211703701e-10,
2418:                 -1.88724975172282928790e-09,
2419:                 +9.38153738649577178388e-09,
2420:                 -4.44505912879632808065e-08,
2421:                 +2.00329475355213526229e-07,
2422:                 -8.56872026469545474066e-07,
2423:                 +3.47025130813767847674e-06,
2424:                 -1.32731636560394358279e-05,
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2425-2436
```cpp
2425:                 +4.78156510755005422638e-05,
2426:                 -1.61760815825896745588e-04,
2427:                 +5.12285956168575772895e-04,
2428:                 -1.51357245063125314899e-03,
2429:                 +4.15642294431288815669e-03,
2430:                 -1.05640848946261981558e-02,
2431:                 +2.47264490306265168283e-02,
2432:                 -5.29459812080949914269e-02,
2433:                 +1.02643658689847095384e-01,
2434:                 -1.76416518357834055153e-01,
2435:                 +2.52587186443633654823e-01,
2436:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2438-2459
```cpp
2438:         static const T B[] = {
2439:                 +7.51729631084210481353e-18,
2440:                 +4.41434832307170791151e-18,
2441:                 -4.65030536848935832153e-17,
2442:                 -3.20952592199342395980e-17,
2443:                 +2.96262899764595013876e-16,
2444:                 +3.30820231092092828324e-16,
2445:                 -1.88035477551078244854e-15,
2446:                 -3.81440307243700780478e-15,
2447:                 +1.04202769841288027642e-14,
2448:                 +4.27244001671195135429e-14,
2449:                 -2.10154184277266431302e-14,
2450:                 -4.08355111109219731823e-13,
2451:                 -7.19855177624590851209e-13,
2452:                 +2.03562854414708950722e-12,
2453:                 +1.41258074366137813316e-11,
2454:                 +3.25260358301548823856e-11,
2455:                 -1.89749581235054123450e-11,
2456:                 -5.58974346219658380687e-10,
2457:                 -3.83538038596423702205e-09,
2458:                 -2.63146884688951950684e-08,
2459:                 -2.51223623787020892529e-07,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2460-2464
```cpp
2460:                 -3.88256480887769039346e-06,
2461:                 -1.10588938762623716291e-04,
2462:                 -9.76109749136146840777e-03,
2463:                 +7.78576235018280120474e-01,
2464:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2466-2467
```cpp
2466:         T p;
2467:         T q = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2469-2483
```cpp
2469:         if (abs(x) <= T(8.0)) {
2470:             T a = A[0];
2471: 
2472:             for (uint8_t index = 1; index < 29; index++) {
2473:                 p = q;
2474:                 q = a;
2475:                 a = ((abs(x) / T(2.0)) - T(2.0)) * q - p + A[index];
2476:             }
2477: 
2478:             if (x < T(0.0)) {
2479:                 return -(T(0.5) * (a - p) * abs(x) * exp(abs(x)));
2480:             }
2481: 
2482:             return T(0.5) * (a - p) * abs(x) * exp(abs(x));
2483:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2485-2485
```cpp
2485:         T b = B[0];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2487-2491
```cpp
2487:         for (uint8_t index = 1; index < 25; index++) {
2488:             p = q;
2489:             q = b;
2490:             b = (T(32.0) / abs(x) - T(2.0)) * q - p + B[index];
2491:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2493-2495
```cpp
2493:         if (x < T(0.0)) {
2494:             return -(exp(abs(x)) * (T(0.5) * (b - p)) / sqrt(abs(x)));
2495:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2497-2499
```cpp
2497:         return exp(abs(x)) * (T(0.5) * (b - p)) / sqrt(abs(x));
2498:     } // modified_bessel_i1_forward(T x)
2499: ); // modified_bessel_i1_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2501-2522
```cpp
2501: const auto modified_bessel_k0_string = modified_bessel_i0_string + jiterator_stringify(
2502:     template<typename T>
2503:     T modified_bessel_k0_forward(T x) {
2504:         static const T A[] = {
2505:                 +1.37446543561352307156e-16,
2506:                 +4.25981614279661018399e-14,
2507:                 +1.03496952576338420167e-11,
2508:                 +1.90451637722020886025e-09,
2509:                 +2.53479107902614945675e-07,
2510:                 +2.28621210311945178607e-05,
2511:                 +1.26461541144692592338e-03,
2512:                 +3.59799365153615016266e-02,
2513:                 +3.44289899924628486886e-01,
2514:                 -5.35327393233902768720e-01,
2515:         };
2516: 
2517:         static const T B[] = {
2518:                 +5.30043377268626276149e-18,
2519:                 -1.64758043015242134646e-17,
2520:                 +5.21039150503902756861e-17,
2521:                 -1.67823109680541210385e-16,
2522:                 +5.51205597852431940784e-16,
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2523-2536
```cpp
2523:                 -1.84859337734377901440e-15,
2524:                 +6.34007647740507060557e-15,
2525:                 -2.22751332699166985548e-14,
2526:                 +8.03289077536357521100e-14,
2527:                 -2.98009692317273043925e-13,
2528:                 +1.14034058820847496303e-12,
2529:                 -4.51459788337394416547e-12,
2530:                 +1.85594911495471785253e-11,
2531:                 -7.95748924447710747776e-11,
2532:                 +3.57739728140030116597e-10,
2533:                 -1.69753450938905987466e-09,
2534:                 +8.57403401741422608519e-09,
2535:                 -4.66048989768794782956e-08,
2536:                 +2.76681363944501510342e-07,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2537-2543
```cpp
2537:                 -1.83175552271911948767e-06,
2538:                 +1.39498137188764993662e-05,
2539:                 -1.28495495816278026384e-04,
2540:                 +1.56988388573005337491e-03,
2541:                 -3.14481013119645005427e-02,
2542:                 +2.44030308206595545468e+00,
2543:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2545-2547
```cpp
2545:         if (x == T(0.0)) {
2546:             return INFINITY;
2547:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2549-2551
```cpp
2549:         if (x < T(0.0)) {
2550:             return NAN;
2551:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2553-2554
```cpp
2553:         T p;
2554:         T q = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2556-2566
```cpp
2556:         if (x <= T(2.0)) {
2557:             T a = A[0];
2558: 
2559:             for (uint8_t index = 1; index < 10; index++) {
2560:                 p = q;
2561:                 q = a;
2562:                 a = (x * x - T(2.0)) * q - p + A[index];
2563:             }
2564: 
2565:             return T(0.5) * (a - p) - log(0.5 * x) * modified_bessel_i0_forward(x);
2566:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2568-2568
```cpp
2568:         T b = B[0];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2570-2574
```cpp
2570:         for (uint8_t index = 1; index < 25; index++) {
2571:             p = q;
2572:             q = b;
2573:             b = (T(8.0) / x - T(2.0)) * q - p + B[index];
2574:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2576-2578
```cpp
2576:         return exp(-x) * (T(0.5) * (b - p)) / sqrt(x);
2577:     } // modified_bessel_k0_forward(T x)
2578: ); // modified_bessel_k0_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2580-2601
```cpp
2580: const auto scaled_modified_bessel_k0_string = modified_bessel_i0_string + jiterator_stringify(
2581:     template<typename T>
2582:     T scaled_modified_bessel_k0_forward(T x) {
2583:         static const T A[] = {
2584:                 +1.37446543561352307156e-16,
2585:                 +4.25981614279661018399e-14,
2586:                 +1.03496952576338420167e-11,
2587:                 +1.90451637722020886025e-09,
2588:                 +2.53479107902614945675e-07,
2589:                 +2.28621210311945178607e-05,
2590:                 +1.26461541144692592338e-03,
2591:                 +3.59799365153615016266e-02,
2592:                 +3.44289899924628486886e-01,
2593:                 -5.35327393233902768720e-01,
2594:         };
2595: 
2596:         static const T B[] = {
2597:                 +5.30043377268626276149e-18,
2598:                 -1.64758043015242134646e-17,
2599:                 +5.21039150503902756861e-17,
2600:                 -1.67823109680541210385e-16,
2601:                 +5.51205597852431940784e-16,
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2602-2615
```cpp
2602:                 -1.84859337734377901440e-15,
2603:                 +6.34007647740507060557e-15,
2604:                 -2.22751332699166985548e-14,
2605:                 +8.03289077536357521100e-14,
2606:                 -2.98009692317273043925e-13,
2607:                 +1.14034058820847496303e-12,
2608:                 -4.51459788337394416547e-12,
2609:                 +1.85594911495471785253e-11,
2610:                 -7.95748924447710747776e-11,
2611:                 +3.57739728140030116597e-10,
2612:                 -1.69753450938905987466e-09,
2613:                 +8.57403401741422608519e-09,
2614:                 -4.66048989768794782956e-08,
2615:                 +2.76681363944501510342e-07,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2616-2622
```cpp
2616:                 -1.83175552271911948767e-06,
2617:                 +1.39498137188764993662e-05,
2618:                 -1.28495495816278026384e-04,
2619:                 +1.56988388573005337491e-03,
2620:                 -3.14481013119645005427e-02,
2621:                 +2.44030308206595545468e+00,
2622:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2624-2626
```cpp
2624:         if (x == T(0.0)) {
2625:             return INFINITY;
2626:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2628-2630
```cpp
2628:         if (x < T(0.0)) {
2629:             return NAN;
2630:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2632-2633
```cpp
2632:         T p;
2633:         T q = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2635-2645
```cpp
2635:         if (x <= T(2.0)) {
2636:             T a = A[0];
2637: 
2638:             for (uint8_t index = 1; index < 10; index++) {
2639:                 p = q;
2640:                 q = a;
2641:                 a = (x * x - T(2.0)) * q - p + A[index];
2642:             }
2643: 
2644:             return (T(0.5) * (a - p) - log(T(0.5) * x) * modified_bessel_i0_forward(x)) * exp(x);
2645:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2647-2647
```cpp
2647:         T b = B[0];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2649-2653
```cpp
2649:         for (uint8_t index = 1; index < 25; index++) {
2650:             p = q;
2651:             q = b;
2652:             b = (T(8.0) / x - T(2.0)) * q - p + B[index];
2653:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2655-2657
```cpp
2655:         return T(0.5) * (b - p) / sqrt(x);
2656:     } // T scaled_modified_bessel_k0_forward(T x)
2657: ); // scaled_modified_bessel_k0_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2659-2680
```cpp
2659: const auto modified_bessel_k1_string = modified_bessel_i1_string + jiterator_stringify(
2660:     template<typename T>
2661:     T modified_bessel_k1_forward(T x) {
2662:         static const T A[] = {
2663:                 -7.02386347938628759343e-18,
2664:                 -2.42744985051936593393e-15,
2665:                 -6.66690169419932900609e-13,
2666:                 -1.41148839263352776110e-10,
2667:                 -2.21338763073472585583e-08,
2668:                 -2.43340614156596823496e-06,
2669:                 -1.73028895751305206302e-04,
2670:                 -6.97572385963986435018e-03,
2671:                 -1.22611180822657148235e-01,
2672:                 -3.53155960776544875667e-01,
2673:                 +1.52530022733894777053e+00,
2674:         };
2675: 
2676:         static const T B[] = {
2677:                 -5.75674448366501715755e-18,
2678:                 +1.79405087314755922667e-17,
2679:                 -5.68946255844285935196e-17,
2680:                 +1.83809354436663880070e-16,
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2681-2694
```cpp
2681:                 -6.05704724837331885336e-16,
2682:                 +2.03870316562433424052e-15,
2683:                 -7.01983709041831346144e-15,
2684:                 +2.47715442448130437068e-14,
2685:                 -8.97670518232499435011e-14,
2686:                 +3.34841966607842919884e-13,
2687:                 -1.28917396095102890680e-12,
2688:                 +5.13963967348173025100e-12,
2689:                 -2.12996783842756842877e-11,
2690:                 +9.21831518760500529508e-11,
2691:                 -4.19035475934189648750e-10,
2692:                 +2.01504975519703286596e-09,
2693:                 -1.03457624656780970260e-08,
2694:                 +5.74108412545004946722e-08,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2695-2702
```cpp
2695:                 -3.50196060308781257119e-07,
2696:                 +2.40648494783721712015e-06,
2697:                 -1.93619797416608296024e-05,
2698:                 +1.95215518471351631108e-04,
2699:                 -2.85781685962277938680e-03,
2700:                 +1.03923736576817238437e-01,
2701:                 +2.72062619048444266945e+00,
2702:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2704-2706
```cpp
2704:         if (x == T(0.0)) {
2705:             return INFINITY;
2706:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2708-2710
```cpp
2708:         if (x < T(0.0)) {
2709:             return NAN;
2710:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2712-2713
```cpp
2712:         T p;
2713:         T q = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2715-2725
```cpp
2715:         if (x <= T(2.0)) {
2716:             T a = A[0];
2717: 
2718:             for (uint8_t index = 1; index < 11; index++) {
2719:                 p = q;
2720:                 q = a;
2721:                 a = (x * x - T(2.0)) * q - p + A[index];
2722:             }
2723: 
2724:             return log(T(0.5) * x) * modified_bessel_i1_forward(x) + T(0.5) * (a - p) / x;
2725:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2727-2727
```cpp
2727:         T b = B[0];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2729-2733
```cpp
2729:         for (uint8_t index = 1; index < 25; index++) {
2730:             p = q;
2731:             q = b;
2732:             b = (T(8.0) / x - T(2.0)) * q - p + B[index];
2733:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2735-2737
```cpp
2735:         return exp(-x) * (T(0.5) * (b - p)) / sqrt(x);
2736:     } // modified_bessel_k1_forward(T x)
2737: ); // modified_bessel_k1_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2739-2760
```cpp
2739: const auto scaled_modified_bessel_k1_string = modified_bessel_i1_string + jiterator_stringify(
2740:     template<typename T>
2741:     T scaled_modified_bessel_k1_forward(T x) {
2742:         static const T A[] = {
2743:                 -7.02386347938628759343e-18,
2744:                 -2.42744985051936593393e-15,
2745:                 -6.66690169419932900609e-13,
2746:                 -1.41148839263352776110e-10,
2747:                 -2.21338763073472585583e-08,
2748:                 -2.43340614156596823496e-06,
2749:                 -1.73028895751305206302e-04,
2750:                 -6.97572385963986435018e-03,
2751:                 -1.22611180822657148235e-01,
2752:                 -3.53155960776544875667e-01,
2753:                 +1.52530022733894777053e+00,
2754:         };
2755: 
2756:         static const T B[] = {
2757:                 -5.75674448366501715755e-18,
2758:                 +1.79405087314755922667e-17,
2759:                 -5.68946255844285935196e-17,
2760:                 +1.83809354436663880070e-16,
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2761-2774
```cpp
2761:                 -6.05704724837331885336e-16,
2762:                 +2.03870316562433424052e-15,
2763:                 -7.01983709041831346144e-15,
2764:                 +2.47715442448130437068e-14,
2765:                 -8.97670518232499435011e-14,
2766:                 +3.34841966607842919884e-13,
2767:                 -1.28917396095102890680e-12,
2768:                 +5.13963967348173025100e-12,
2769:                 -2.12996783842756842877e-11,
2770:                 +9.21831518760500529508e-11,
2771:                 -4.19035475934189648750e-10,
2772:                 +2.01504975519703286596e-09,
2773:                 -1.03457624656780970260e-08,
2774:                 +5.74108412545004946722e-08,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2775-2782
```cpp
2775:                 -3.50196060308781257119e-07,
2776:                 +2.40648494783721712015e-06,
2777:                 -1.93619797416608296024e-05,
2778:                 +1.95215518471351631108e-04,
2779:                 -2.85781685962277938680e-03,
2780:                 +1.03923736576817238437e-01,
2781:                 +2.72062619048444266945e+00,
2782:         };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2784-2786
```cpp
2784:         if (x == T(0.0)) {
2785:             return INFINITY;
2786:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2788-2790
```cpp
2788:         if (x < T(0.0)) {
2789:             return NAN;
2790:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2792-2793
```cpp
2792:         T p;
2793:         T q = 0.0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2795-2805
```cpp
2795:         if (x <= T(2.0)) {
2796:             T a = A[0];
2797: 
2798:             for (uint8_t index = 1; index < 11; index++) {
2799:                 p = q;
2800:                 q = a;
2801:                 a = (x * x - T(2.0)) * q - p + A[index];
2802:             }
2803: 
2804:             return (log(T(0.5) * x) * modified_bessel_i1_forward(x) + T(0.5) * (a - p) / x) * exp(x);
2805:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2807-2807
```cpp
2807:         T b = B[0];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2809-2813
```cpp
2809:         for (uint8_t index = 1; index < 25; index++) {
2810:             p = q;
2811:             q = b;
2812:             b = (T(8.0) / x - T(2.0)) * q - p + B[index];
2813:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2815-2817
```cpp
2815:         return (T(0.5) * (b - p) / sqrt(x));
2816:     } // T scaled_modified_bessel_k1_forward(T x)
2817: ); // scaled_modified_bessel_k1_string
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2819-2840
```cpp
2819: const auto shifted_chebyshev_polynomial_t_string = jiterator_stringify(
2820:     template<typename T>
2821:     T shifted_chebyshev_polynomial_t_forward(T x, int64_t n) {
2822:         if (n < 0) {
2823:             return T(0.0);
2824:         }
2825: 
2826:         if (x == T(1.0)) {
2827:             return T(1.0);
2828:         }
2829: 
2830:         if (x == T(0.0)) {
2831:             if (n % 2 == 0) {
2832:                 return T(1.0);
2833:             }
2834: 
2835:             return T(-1.0);
2836:         }
2837: 
2838:         if ((n > 6) && (abs(x + x - T(1.0)) < T(1.0))) {
2839:             return cos(n * acos(x + x - T(1.0)));
2840:         }
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2842-2844
```cpp
2842:         if (n == 0) {
2843:             return T(1.0);
2844:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2846-2848
```cpp
2846:         if (n == 1) {
2847:             return x + x - T(1.0);
2848:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2850-2852
```cpp
2850:         T p = T(1.0);
2851:         T q = x + x - T(1.0);
2852:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2854-2858
```cpp
2854:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
2855:             r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
2856:             p = q;
2857:             q = r;
2858:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2860-2861
```cpp
2860:         return r;
2861:     } // shifted_chebyshev_polynomial_t_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2863-2867
```cpp
2863:     template<typename T>
2864:     T shifted_chebyshev_polynomial_t_forward(T x, T n) {
2865:         return shifted_chebyshev_polynomial_t_forward(x, static_cast<int64_t>(n));
2866:     } // shifted_chebyshev_polynomial_t_forward(T x, T n)
2867: ); // shifted_chebyshev_polynomial_t_string
```
- EN: This block defines or continues the implementation of `shifted_chebyshev_polynomial_t_forward`.
- CN: 该代码块定义或继续实现 `shifted_chebyshev_polynomial_t_forward`。

### Lines 2869-2890
```cpp
2869: const auto shifted_chebyshev_polynomial_u_string = jiterator_stringify(
2870:     template<typename T>
2871:     T shifted_chebyshev_polynomial_u_forward(T x, int64_t n) {
2872:         if (n < 0) {
2873:             return T(0.0);
2874:         }
2875: 
2876:         if (x == T(1.0)) {
2877:             return n + 1;
2878:         }
2879: 
2880:         if (x == T(0.0)) {
2881:             if (n % 2 == 0) {
2882:                 return n + 1;
2883:             }
2884: 
2885:             return -(n + 1);
2886:         }
2887: 
2888:         if ((n > 6) && (abs(x + x - T(1.0)) < T(1.0))) {
2889:             if (sin(acos(x + x - T(1.0))) != T(0.0)) {
2890:                 return sin((n + 1) * acos(x + x - T(1.0))) / sin(acos(x + x - T(1.0)));
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2891-2891
```cpp
2891:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2893-2894
```cpp
2893:             return (n + 1) * cos((n + 1) * acos(x + x - T(1.0))) / (x + x - T(1.0));
2894:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2896-2898
```cpp
2896:         if (n == 0) {
2897:             return T(1.0);
2898:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2900-2902
```cpp
2900:         if (n == 1) {
2901:             return x + x - T(1.0) + (x + x - T(1.0));
2902:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2904-2906
```cpp
2904:         T p = T(1.0);
2905:         T q = x + x - T(1.0) + (x + x - T(1.0));
2906:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2908-2912
```cpp
2908:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
2909:             r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
2910:             p = q;
2911:             q = r;
2912:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2914-2915
```cpp
2914:         return r;
2915:     } // shifted_chebyshev_polynomial_u_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2917-2921
```cpp
2917:     template<typename T>
2918:     T shifted_chebyshev_polynomial_u_forward(T x, T n) {
2919:         return shifted_chebyshev_polynomial_u_forward(x, static_cast<int64_t>(n));
2920:     } // shifted_chebyshev_polynomial_u_forward(T x, T n)
2921: ); // shifted_chebyshev_polynomial_u_string
```
- EN: This block defines or continues the implementation of `shifted_chebyshev_polynomial_u_forward`.
- CN: 该代码块定义或继续实现 `shifted_chebyshev_polynomial_u_forward`。

### Lines 2923-2944
```cpp
2923: const auto shifted_chebyshev_polynomial_v_string = jiterator_stringify(
2924:     template<typename T>
2925:     T shifted_chebyshev_polynomial_v_forward(T x, int64_t n) {
2926:         if (n < 0) {
2927:             return T(0.0);
2928:         }
2929: 
2930:         if (x == T(1.0)) {
2931:             return T(1.0);
2932:         }
2933: 
2934:         if (x == T(0.0)) {
2935:             if (n % 2 == 0) {
2936:                 return (n + n + 1);
2937:             }
2938: 
2939:             return -(n + n + 1);
2940:         }
2941: 
2942:         if ((n > 6) && (abs(x + x - T(1.0)) < T(1.0))) {
2943:             if (sin(acos(x + x - T(1.0)) / T(2.0)) != T(1.0)) {
2944:                 return cos(((n) + T(0.5)) * acos(x + x - T(1.0))) / cos(acos(x + x - T(1.0)) / T(2.0));
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 2945-2945
```cpp
2945:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2947-2949
```cpp
2947:             if (n % 2 == 0) {
2948:                 return n + n + 1;
2949:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2951-2952
```cpp
2951:             return -(n + n + 1);
2952:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2954-2956
```cpp
2954:         if (n == 0) {
2955:             return T(1.0);
2956:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2958-2960
```cpp
2958:         if (n == 1) {
2959:             return x + x - T(1.0) + (x + x - T(1.0)) - T(1.0);
2960:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2962-2964
```cpp
2962:         T p = T(1.0);
2963:         T q = x + x - T(1.0) + (x + x - T(1.0)) - T(1.0);
2964:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2966-2970
```cpp
2966:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
2967:             r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
2968:             p = q;
2969:             q = r;
2970:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2972-2973
```cpp
2972:         return r;
2973:     } // shifted_chebyshev_polynomial_v_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 2975-2979
```cpp
2975:     template<typename T>
2976:     T shifted_chebyshev_polynomial_v_forward(T x, T n) {
2977:         return shifted_chebyshev_polynomial_v_forward(x, static_cast<int64_t>(n));
2978:     } // shifted_chebyshev_polynomial_v_forward(T x, T n)
2979: ); // shifted_chebyshev_polynomial_v_string
```
- EN: This block defines or continues the implementation of `shifted_chebyshev_polynomial_v_forward`.
- CN: 该代码块定义或继续实现 `shifted_chebyshev_polynomial_v_forward`。

### Lines 2981-3002
```cpp
2981: const auto shifted_chebyshev_polynomial_w_string = jiterator_stringify(
2982:     template<typename T>
2983:     T shifted_chebyshev_polynomial_w_forward(T x, int64_t n) {
2984:         if (n < 0) {
2985:             return T(0.0);
2986:         }
2987: 
2988:         if (x == T(1.0)) {
2989:             return n + n + 1;
2990:         }
2991: 
2992:         if (x == T(0.0)) {
2993:             if (n % 2 == 0) {
2994:                 return T(1.0);
2995:             }
2996: 
2997:             return T(-1.0);
2998:         }
2999: 
3000:         if ((n > 4) && (abs(x + x - T(1.0)) < T(1.0))) {
3001:             if (cos(acos(x + x - T(1.0)) / T(2.0)) != T(1.0)) {
3002:                 return sin((n + T(0.5)) * acos(x + x - T(1.0))) / sin(acos(x + x - T(1.0)) / T(2.0));
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 3003-3003
```cpp
3003:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3005-3007
```cpp
3005:             if (n % 2 == 0) {
3006:                 return T(1.0);
3007:             }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3009-3010
```cpp
3009:             return T(-1.0);
3010:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3012-3014
```cpp
3012:         if (n == 0) {
3013:             return T(1.0);
3014:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3016-3018
```cpp
3016:         if (n == 1) {
3017:             return x + x - T(1.0) + (x + x - T(1.0)) + T(1.0);
3018:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3020-3022
```cpp
3020:         T p = T(1.0);
3021:         T q = x + x - T(1.0) + (x + x - T(1.0)) + T(1.0);
3022:         T r;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3024-3028
```cpp
3024:         for (int64_t k = 2; (k <= n) && !isnan(q); k++) {
3025:             r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
3026:             p = q;
3027:             q = r;
3028:         }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3030-3031
```cpp
3030:         return r;
3031:     } // shifted_chebyshev_polynomial_w_forward(T x, int64_t n)
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3033-3037
```cpp
3033:     template<typename T>
3034:     T shifted_chebyshev_polynomial_w_forward(T x, T n) {
3035:         return shifted_chebyshev_polynomial_w_forward(x, static_cast<int64_t>(n));
3036:     } // shifted_chebyshev_polynomial_w_forward(T x, T n)
3037: ); // shifted_chebyshev_polynomial_w_string
```
- EN: This block defines or continues the implementation of `shifted_chebyshev_polynomial_w_forward`.
- CN: 该代码块定义或继续实现 `shifted_chebyshev_polynomial_w_forward`。

### Lines 3039-3052
```cpp
3039: const auto spherical_bessel_j0_string = jiterator_stringify(
3040:     template<typename T>
3041:     T spherical_bessel_j0_forward(T x) {
3042:         if (isinf(x)) {
3043:             return T(0.0);
3044:         }
3045: 
3046:         if (abs(x) < T(0.5)) {
3047:             return T(1.0) + x * x * (T(-1.0) / T(6.0) + x * x * (T(1.0) / T(120.0) + x * x * (T(-1.0) / T(5040.0) + x * x * (T(1.0) / T(362880.0) + x * x * (T(-1.0) / T(39916800.0) + x * x * (T(1.0) / T(6227020800.0)))))));
3048:         }
3049: 
3050:         return sin(x) / x;
3051:     } // T spherical_bessel_j0_forward(T x)
3052: ); // spherical_bessel_j0_string
```
- EN: This block defines or continues the implementation of `jiterator_stringify`.
- CN: 该代码块定义或继续实现 `jiterator_stringify`。

### Lines 3054-3054
```cpp
3054: #else // !AT_USE_JITERATOR() -- kernels must be precompiled
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 3056-3066
```cpp
3056: template <typename scalar_t>
3057: static inline C10_HOST_DEVICE scalar_t calc_gcd(scalar_t a_in, scalar_t b_in) {
3058:   scalar_t a = ::abs(a_in);
3059:   scalar_t b = ::abs(b_in);
3060:   while (a != 0) {
3061:     scalar_t c = a;
3062:     a = b % a;
3063:     b = c;
3064:   }
3065:   return b;
3066: }
```
- EN: This block defines or continues the implementation of `calc_gcd`.
- CN: 该代码块定义或继续实现 `calc_gcd`。

### Lines 3068-3070
```cpp
3068: /*
3069:  * For licensing information, please refer to the cpu implementation located in "ATen/native/Math.h".
3070:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 3071-3092
```cpp
3071: template <typename scalar_t>
3072: static inline C10_HOST_DEVICE scalar_t calc_digamma(scalar_t in) {
3073:   // [C++ Standard Reference: Gamma Function] https://en.cppreference.com/w/cpp/numeric/math/tgamma
3074:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
3075:   static constexpr double PI_f64 = 3.14159265358979323846;
3076:   constexpr accscalar_t PSI_10 = 2.25175258906672110764;
3077:   constexpr accscalar_t A[] = {
3078:       8.33333333333333333333E-2,
3079:       -2.10927960927960927961E-2,
3080:       7.57575757575757575758E-3,
3081:       -4.16666666666666666667E-3,
3082:       3.96825396825396825397E-3,
3083:       -8.33333333333333333333E-3,
3084:       8.33333333333333333333E-2,
3085:   };
3086: 
3087:   accscalar_t x = static_cast<accscalar_t>(in);
3088:   if (x == 0) {
3089:     // As per C++ standard for gamma related functions and SciPy,
3090:     // If the argument is ±0, ±∞ is returned
3091:     return std::copysign(static_cast<scalar_t>(INFINITY), -x);
3092:   }
```
- EN: This block defines or continues the implementation of `calc_digamma`.
- CN: 该代码块定义或继续实现 `calc_digamma`。

### Lines 3094-3110
```cpp
3094:   bool x_is_integer = x == ::trunc(x);
3095:   accscalar_t result = 0;
3096:   if (x < 0) {
3097:     if (x_is_integer) {
3098:       // As per C++ standard for gamma related functions and SciPy,
3099:       // If the argument is a negative integer, NaN is returned
3100:       return static_cast<scalar_t>(NAN);
3101:     }
3102:     // Extracts the fractional part of x as r, since tan(pi * r) is more numerically
3103:     // accurate than tan(pi * x). While these operations are mathematically equivalent
3104:     // since both x and r are in radians and tan() has a periodicity of pi, in practice
3105:     // the computation of pi * x is a source of error (when |x| > 1).
3106:     double q, r;
3107:     r = ::modf(static_cast<double>(x), &q);
3108:     result = static_cast<accscalar_t>(- PI_f64 / ::tan(PI_f64 * r));
3109:     x = 1 - x;
3110:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3112-3118
```cpp
3112:   while (x < 10) {
3113:     result -= 1 / x;
3114:     x += 1;
3115:   }
3116:   if (x == 10) {
3117:     return static_cast<scalar_t>(result + PSI_10);
3118:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3120-3129
```cpp
3120:   accscalar_t y = 0;
3121:   if (x < 1.0e17) {
3122:     accscalar_t z = 1 / (x * x);
3123: 
3124:     accscalar_t polevl_result = 0;
3125:     for (int i = 0; i <= 6; i++) {
3126:       polevl_result = polevl_result * z + A[i];
3127:     }
3128:     y = z * polevl_result;
3129:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3131-3132
```cpp
3131:   return static_cast<scalar_t>(::log(x) - (static_cast<accscalar_t>(0.5) / x) - y + result);
3132: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3134-3155
```cpp
3134: template <typename scalar_t>
3135: static inline C10_HOST_DEVICE scalar_t calc_trigamma(scalar_t in) {
3136:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
3137:   const accscalar_t PI = 3.14159265358979323846;
3138:   accscalar_t x = static_cast<accscalar_t>(in);
3139:   accscalar_t sign = +1;
3140:   accscalar_t result = 0;
3141:   if (x < 0.5f) {
3142:     sign = -1;
3143:     accscalar_t sin_pi_x = ::sin(PI * x);
3144:     result -= (PI * PI) / (sin_pi_x * sin_pi_x);
3145:     x = 1 - x;
3146:   }
3147:   for (int i = 0; i < 6; ++i) {
3148:     result += 1 / (x * x);
3149:     x += 1;
3150:   }
3151:   const accscalar_t one = static_cast<scalar_t>(1);
3152:   const accscalar_t ixx = 1 / (x*x);
3153:   result += (1 + 1 / (2*x) + ixx * (one/6 - ixx * (one/30 - ixx * (one/42)))) / x;
3154:   return static_cast<scalar_t>(sign * result);
3155: }
```
- EN: This block defines or continues the implementation of `calc_trigamma`.
- CN: 该代码块定义或继续实现 `calc_trigamma`。

### Lines 3157-3159
```cpp
3157: /*
3158:  * For licensing information and documentation, please refer to the cpu implementation located in "ATen/native/Math.h".
3159:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 3160-3177
```cpp
3160: template <typename scalar_t>
3161: static inline C10_HOST_DEVICE scalar_t
3162: chbevl(scalar_t _x, const scalar_t array[], size_t len) {
3163:   static_assert(!std::is_same<scalar_t, Half>() && !std::is_same<scalar_t, BFloat16>(), "don't instantiate with low precision type");
3164: 
3165:   scalar_t b0, b1, b2;
3166: 
3167:   b0 = array[0];
3168:   b1 = 0;
3169: 
3170:   for (size_t i = 1; i < len; ++i)  {
3171:     b2 = b1;
3172:     b1 = b0;
3173:     b0 = _x * b1 - b2 + array[i];
3174:   }
3175: 
3176:   return (0.5 * (b0 - b2));
3177: }
```
- EN: This block defines or continues the implementation of `chbevl`.
- CN: 该代码块定义或继续实现 `chbevl`。

### Lines 3179-3181
```cpp
3179: /*
3180:  * For licensing information and documentation, please refer to the cpu implementation located in "ATen/native/Math.h".
3181:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 3182-3203
```cpp
3182: template <typename T>
3183: C10_HOST_DEVICE inline std::tuple<const T*, size_t> chebyshev_coefficients_i0e_A() {
3184:   /* Chebyshev coefficients for exp(-x) I0(x)
3185:    * in the interval [0,8].
3186:    *
3187:    * lim(x->0){ exp(-x) I0(x) } = 1.
3188:    */
3189:   static const T coefficients[] = {
3190:       -4.41534164647933937950E-18, 3.33079451882223809783E-17,
3191:       -2.43127984654795469359E-16, 1.71539128555513303061E-15,
3192:       -1.16853328779934516808E-14, 7.67618549860493561688E-14,
3193:       -4.85644678311192946090E-13, 2.95505266312963983461E-12,
3194:       -1.72682629144155570723E-11, 9.67580903537323691224E-11,
3195:       -5.18979560163526290666E-10, 2.65982372468238665035E-9,
3196:       -1.30002500998624804212E-8,  6.04699502254191894932E-8,
3197:       -2.67079385394061173391E-7,  1.11738753912010371815E-6,
3198:       -4.41673835845875056359E-6,  1.64484480707288970893E-5,
3199:       -5.75419501008210370398E-5,  1.88502885095841655729E-4,
3200:       -5.76375574538582365885E-4,  1.63947561694133579842E-3,
3201:       -4.32430999505057594430E-3,  1.05464603945949983183E-2,
3202:       -2.37374148058994688156E-2,  4.93052842396707084878E-2,
3203:       -9.49010970480476444210E-2,  1.71620901522208775349E-1,
```
- EN: This block defines or continues the implementation of `chebyshev_coefficients_i0e_A`, `exp`.
- CN: 该代码块定义或继续实现 `chebyshev_coefficients_i0e_A`, `exp`。

### Lines 3204-3204
```cpp
3204:       -3.04682672343198398683E-1,  6.76795274409476084995E-1};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3206-3207
```cpp
3206:   return std::make_tuple(coefficients, 30);
3207: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3209-3229
```cpp
3209: template <typename T>
3210: C10_HOST_DEVICE inline std::tuple<const T*, size_t> chebyshev_coefficients_i0e_B() {
3211:   /* Chebyshev coefficients for exp(-x) sqrt(x) I0(x)
3212:    * in the inverted interval [8,infinity].
3213:    *
3214:    * lim(x->inf){ exp(-x) sqrt(x) I0(x) } = 1/sqrt(2pi).
3215:    */
3216:   static const T coefficients[] = {
3217:       -7.23318048787475395456E-18, -4.83050448594418207126E-18,
3218:       4.46562142029675999901E-17,  3.46122286769746109310E-17,
3219:       -2.82762398051658348494E-16, -3.42548561967721913462E-16,
3220:       1.77256013305652638360E-15,  3.81168066935262242075E-15,
3221:       -9.55484669882830764870E-15, -4.15056934728722208663E-14,
3222:       1.54008621752140982691E-14,  3.85277838274214270114E-13,
3223:       7.18012445138366623367E-13,  -1.79417853150680611778E-12,
3224:       -1.32158118404477131188E-11, -3.14991652796324136454E-11,
3225:       1.18891471078464383424E-11,  4.94060238822496958910E-10,
3226:       3.39623202570838634515E-9,   2.26666899049817806459E-8,
3227:       2.04891858946906374183E-7,   2.89137052083475648297E-6,
3228:       6.88975834691682398426E-5,   3.36911647825569408990E-3,
3229:       8.04490411014108831608E-1};
```
- EN: This block defines or continues the implementation of `chebyshev_coefficients_i0e_B`, `exp`.
- CN: 该代码块定义或继续实现 `chebyshev_coefficients_i0e_B`, `exp`。

### Lines 3231-3232
```cpp
3231:   return std::make_tuple(coefficients, 25);
3232: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3234-3249
```cpp
3234: template <typename scalar_t>
3235: static inline C10_HOST_DEVICE scalar_t calc_i0(scalar_t _x) {
3236:   static_assert(!std::is_same<scalar_t, Half>() && !std::is_same<scalar_t, BFloat16>(), "don't instantiate with low precision type");
3237:   // Upcast input for numerical accuracy purposes
3238:   // Needed for accurate results if input is bfloat16 or float16
3239:   scalar_t x = ::abs(_x);
3240: 
3241:   if (x <= scalar_t{8.0}) {
3242:     auto [A, len] = chebyshev_coefficients_i0e_A<scalar_t>();
3243:     scalar_t y = (x / scalar_t{2.0}) - scalar_t{2.0};
3244:     return (::exp(x) * chbevl(y, A, len));
3245:   }
3246: 
3247:   auto [B, len] = chebyshev_coefficients_i0e_B<scalar_t>();
3248:   return (::exp(x) * chbevl(scalar_t{32.0} / x - scalar_t{2.0}, B, len) / ::sqrt(x));
3249: }
```
- EN: This block defines or continues the implementation of `calc_i0`.
- CN: 该代码块定义或继续实现 `calc_i0`。

### Lines 3251-3272
```cpp
3251: template <typename T>
3252: C10_HOST_DEVICE inline
3253:     typename std::enable_if_t<std::is_same_v<double, T>, std::tuple<const T*, size_t>>
3254:     chebyshev_coefficients_i1e_A() {
3255:   /* Chebyshev coefficients for exp(-x) I1(x)
3256:    * in the interval [0,8].
3257:    *
3258:    * lim(x->0){ exp(-x) I1(x) / x } = 1/2.
3259:    */
3260:   static const T coefficients[] = {
3261:       2.77791411276104639959E-18, -2.11142121435816608115E-17,
3262:       1.55363195773620046921E-16, -1.10559694773538630805E-15,
3263:       7.60068429473540693410E-15, -5.04218550472791168711E-14,
3264:       3.22379336594557470981E-13, -1.98397439776494371520E-12,
3265:       1.17361862988909016308E-11, -6.66348972350202774223E-11,
3266:       3.62559028155211703701E-10, -1.88724975172282928790E-9,
3267:       9.38153738649577178388E-9,  -4.44505912879632808065E-8,
3268:       2.00329475355213526229E-7,  -8.56872026469545474066E-7,
3269:       3.47025130813767847674E-6,  -1.32731636560394358279E-5,
3270:       4.78156510755005422638E-5,  -1.61760815825896745588E-4,
3271:       5.12285956168575772895E-4,  -1.51357245063125314899E-3,
3272:       4.15642294431288815669E-3,  -1.05640848946261981558E-2,
```
- EN: This block defines or continues the implementation of `chebyshev_coefficients_i1e_A`, `exp`.
- CN: 该代码块定义或继续实现 `chebyshev_coefficients_i1e_A`, `exp`。

### Lines 3273-3275
```cpp
3273:       2.47264490306265168283E-2,  -5.29459812080949914269E-2,
3274:       1.02643658689847095384E-1,  -1.76416518357834055153E-1,
3275:       2.52587186443633654823E-1};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3277-3278
```cpp
3277:   return std::make_tuple(coefficients, 29);
3278: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3280-3301
```cpp
3280: template <typename T>
3281: C10_HOST_DEVICE inline
3282:     typename std::enable_if_t<std::is_same_v<float, T>, std::tuple<const T*, size_t>>
3283:     chebyshev_coefficients_i1e_A() {
3284:   /* Chebyshev coefficients for exp(-x) I1(x)
3285:    * in the interval [0,8].
3286:    *
3287:    * lim(x->0){ exp(-x) I1(x) / x } = 1/2.
3288:    */
3289:   static const T coeff[] = {
3290:       9.38153738649577178388E-9f,
3291:       -4.44505912879632808065E-8f,
3292:       2.00329475355213526229E-7f,
3293:       -8.56872026469545474066E-7f,
3294:       3.47025130813767847674E-6f,
3295:       -1.32731636560394358279E-5f,
3296:       4.78156510755005422638E-5f,
3297:       -1.61760815825896745588E-4f,
3298:       5.12285956168575772895E-4f,
3299:       -1.51357245063125314899E-3f,
3300:       4.15642294431288815669E-3f,
3301:       -1.05640848946261981558E-2f,
```
- EN: This block defines or continues the implementation of `chebyshev_coefficients_i1e_A`, `exp`.
- CN: 该代码块定义或继续实现 `chebyshev_coefficients_i1e_A`, `exp`。

### Lines 3302-3308
```cpp
3302:       2.47264490306265168283E-2f,
3303:       -5.29459812080949914269E-2f,
3304:       1.02643658689847095384E-1f,
3305:       -1.76416518357834055153E-1f,
3306:       2.52587186443633654823E-1f};
3307:   return std::make_tuple(coeff, 17);
3308: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3310-3331
```cpp
3310: template <typename T>
3311: C10_HOST_DEVICE inline
3312:     typename std::enable_if_t<std::is_same_v<double, T>, std::tuple<const T*, size_t>>
3313:     chebyshev_coefficients_i1e_B() {
3314:   /* Chebyshev coefficients for exp(-x) sqrt(x) I1(x)
3315:    * in the inverted interval [8,infinity].
3316:    *
3317:    * lim(x->inf){ exp(-x) sqrt(x) I1(x) } = 1/sqrt(2pi).
3318:    */
3319:   static const T coefficients[] = {
3320:       7.51729631084210481353E-18,  4.41434832307170791151E-18,
3321:       -4.65030536848935832153E-17, -3.20952592199342395980E-17,
3322:       2.96262899764595013876E-16,  3.30820231092092828324E-16,
3323:       -1.88035477551078244854E-15, -3.81440307243700780478E-15,
3324:       1.04202769841288027642E-14,  4.27244001671195135429E-14,
3325:       -2.10154184277266431302E-14, -4.08355111109219731823E-13,
3326:       -7.19855177624590851209E-13, 2.03562854414708950722E-12,
3327:       1.41258074366137813316E-11,  3.25260358301548823856E-11,
3328:       -1.89749581235054123450E-11, -5.58974346219658380687E-10,
3329:       -3.83538038596423702205E-9,  -2.63146884688951950684E-8,
3330:       -2.51223623787020892529E-7,  -3.88256480887769039346E-6,
3331:       -1.10588938762623716291E-4,  -9.76109749136146840777E-3,
```
- EN: This block defines or continues the implementation of `chebyshev_coefficients_i1e_B`, `exp`.
- CN: 该代码块定义或继续实现 `chebyshev_coefficients_i1e_B`, `exp`。

### Lines 3332-3332
```cpp
3332:       7.78576235018280120474E-1};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3334-3335
```cpp
3334:   return std::make_tuple(coefficients, 25);
3335: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 3337-3356
```cpp
3337: template <typename T>
3338: C10_HOST_DEVICE inline
3339:     typename std::enable_if_t<std::is_same_v<float, T>, std::tuple<const T*, size_t>>
3340:     chebyshev_coefficients_i1e_B() {
3341:   /* Chebyshev coefficients for exp(-x) sqrt(x) I1(x)
3342:    * in the inverted interval [8,infinity].
3343:    *
3344:    * lim(x->inf){ exp(-x) sqrt(x) I1(x) } = 1/sqrt(2pi).
3345:    */
3346:   static const T coeff[] = {
3347:       -3.83538038596423702205E-9f,
3348:       -2.63146884688951950684E-8f,
3349:       -2.51223623787020892529E-7f,
3350:       -3.88256480887769039346E-6f,
3351:       -1.10588938762623716291E-4f,
3352:       -9.76109749136146840777E-3f,
3353:       7.78576235018280120474E-1f};
3354: 
3355:   return std::make_tuple(coeff, 7);
3356: };
```
- EN: This block defines or continues the implementation of `chebyshev_coefficients_i1e_B`, `exp`.
- CN: 该代码块定义或继续实现 `chebyshev_coefficients_i1e_B`, `exp`。

### Lines 3358-3371
```cpp
3358: template <typename scalar_t>
3359: static inline C10_HOST_DEVICE scalar_t calc_i1(scalar_t _x) {
3360:   const auto x = ::abs(_x);
3361:   if (x <= scalar_t{8.0}) {
3362:     auto [A, len] = chebyshev_coefficients_i1e_A<scalar_t>();
3363:     scalar_t y = x / scalar_t{2.0} - scalar_t{2.0};
3364:     const scalar_t out = ::exp(x) * x * chbevl(y, A, len);
3365:     return (_x < scalar_t{0.0}) ? -out : out;
3366:   }
3367: 
3368:   auto [B, len] = chebyshev_coefficients_i1e_B<scalar_t>();
3369:   const scalar_t out = (::exp(x) * chbevl(scalar_t{32.0} / x - scalar_t{2.0}, B, len)) / ::sqrt(x);
3370:   return (_x < scalar_t{0.0}) ? -out : out;
3371: }
```
- EN: This block defines or continues the implementation of `calc_i1`.
- CN: 该代码块定义或继续实现 `calc_i1`。

### Lines 3373-3386
```cpp
3373: template <typename scalar_t>
3374: static inline C10_HOST_DEVICE scalar_t calc_i1e(scalar_t _x) {
3375:   const auto x = ::abs(_x);
3376:   if (x <= scalar_t{8.0}) {
3377:     auto [A, len] = chebyshev_coefficients_i1e_A<scalar_t>();
3378:     const scalar_t y = x / scalar_t{2.0} - scalar_t{2.0};
3379:     const scalar_t out = chbevl(y, A, len) * x;
3380:     return (_x < scalar_t{0.0}) ? -out : out;
3381:   }
3382: 
3383:   auto [B, len] = chebyshev_coefficients_i1e_B<scalar_t>();
3384:   const scalar_t out = chbevl(scalar_t{32.0} / x - scalar_t{2.0}, B, len) / ::sqrt(x);
3385:   return (_x < scalar_t{0.0}) ? -out : out;
3386: }
```
- EN: This block defines or continues the implementation of `calc_i1e`.
- CN: 该代码块定义或继续实现 `calc_i1e`。

### Lines 3388-3388
```cpp
3388: #endif // AT_USE_JITERATOR() (this closes the "else" branch of a if/else preprocessor directive)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 3390-3390
```cpp
3390: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- CUDA-native implementation details are concentrated here, combining PyTorch tensor abstractions with GPU execution. / 这里集中体现了 CUDA 原生实现细节，把 PyTorch 张量抽象与 GPU 执行连接起来。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/AccumulateType.h>`
  - `<ATen/jit_macros.h>`
  - `<c10/macros/Macros.h>`
  - `<ATen/native/cuda/jit_utils.h>`
- Runtime symbols / 运行时符号: no obvious helper symbols were extracted; dependencies are mostly local or implicit / 未提取到明显辅助符号，依赖主要是局部实现或隐式机制。
