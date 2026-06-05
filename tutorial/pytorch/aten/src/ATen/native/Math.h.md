# Math.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Math.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Math. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 数学 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <ATen/AccumulateType.h>
0004: #include <ATen/NumericUtils.h>
0005: #include <ATen/jiterator_macros.h>
0006: #include <c10/macros/Macros.h>
0007: #include <c10/util/BFloat16.h>
0008: #include <c10/util/Half.h>
0009: #include <c10/util/MathConstants.h>
0010: #include <cfloat>
0011: #include <cmath>
0012: #include <cstdint>
0013: #include <cstdlib>
0014: #include <limits>
0015: #include <type_traits>
0016: 
0017: C10_CLANG_DIAGNOSTIC_PUSH()
0018: #if C10_CLANG_HAS_WARNING("-Wimplicit-float-conversion")
0019: C10_CLANG_DIAGNOSTIC_IGNORE("-Wimplicit-float-conversion")
0020: #endif
0021: 
0022: /* The next function is taken from  https://github.com/antelopeusersgroup/antelope_contrib/blob/master/lib/location/libgenloc/erfinv.c.
0023: Below is the copyright.
0024: Output was modified to be inf or -inf when input is 1 or -1. */
0025: 
0026: 
0027: /*
0028:     Copyright (c) 2014 Indiana University
0029:     All rights reserved.
0030: 
```
- **EN**: Lines 1-30 mainly cover header inclusion, macro-based glue, comments/documentation. Notable symbols: C10_CLANG_DIAGNOSTIC_PUSH, C10_CLANG_HAS_WARNING, C10_CLANG_DIAGNOSTIC_IGNORE, Copyright.
- **CN**: 第 1-30 行主要涉及头文件包含、宏定义或宏调用、注释或说明。 值得关注的符号包括：C10_CLANG_DIAGNOSTIC_PUSH, C10_CLANG_HAS_WARNING, C10_CLANG_DIAGNOSTIC_IGNORE, Copyright。

### Lines 31-60 / 第 31-60 行
```cpp
0031:     Written by Prof. Gary L. Pavlis, Dept. of Geol. Sci.,
0032:             Indiana University, Bloomington, IN
0033: 
0034:     This software is licensed under the New BSD license:
0035: 
0036:     Redistribution and use in source and binary forms,
0037:     with or without modification, are permitted provided
0038:     that the following conditions are met:
0039: 
0040:     Redistributions of source code must retain the above
0041:     copyright notice, this list of conditions and the
0042:     following disclaimer.
0043: 
0044:     Redistributions in binary form must reproduce the
0045:     above copyright notice, this list of conditions and
0046:     the following disclaimer in the documentation and/or
0047:     other materials provided with the distribution.
0048: 
0049:     Neither the name of Indiana University nor
0050:     the names of its contributors may be used to endorse
0051:     or promote products derived from this software without
0052:     specific prior written permission.
0053: 
0054:     THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND
0055:     CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED
0056:     WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
0057:     WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
0058:     PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL
0059:     THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY
0060:     DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
```
- **EN**: Lines 31-60 mainly cover expressions/calls.
- **CN**: 第 31-60 行主要涉及表达式或调用。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
0062:     PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF
0063:     USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
0064:     HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER
0065:     IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
0066:     NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE
0067:     USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
0068:     POSSIBILITY OF SUCH DAMAGE.
0069: */
0070: 
0071: namespace {
0072: /*
0073:  * This function is derived from the implementation of the i0e function in the
0074:  * Cephes Math Library. See note [3-Clause BSD License for the Cephes Math
0075:  * Library].
0076:  *
0077:  * Computes an approximation of the exponentially scaled zeroth order modified
0078:  * Bessel function of the first kind. The approximation is actually two
0079:  * (sub)approximations, both using a Chebyshev polynomial expansion. One
0080:  * approximates the function over [0, 8], and the other over (8, infinity). This
0081:  * function takes the absolute value of all inputs to convert them into the
0082:  * domain of the approximation.
0083:  */
0084: jiterator_also_stringify_as(jiterator_code(
0085:   template <typename T>
0086:   JITERATOR_HOST_DEVICE T chbevl(T x, const T array[], const int len) {
0087:     T b0, b1, b2 = 0;
0088: 
0089:     b0 = array[0];
0090:     b1 = 0;
```
- **EN**: Lines 61-90 mainly cover comments/documentation, expressions/calls, state/variable declarations. Notable symbols: DAMAGES, TORT, over, jiterator_also_stringify_as.
- **CN**: 第 61-90 行主要涉及注释或说明、表达式或调用、变量/别名声明。 值得关注的符号包括：DAMAGES, TORT, over, jiterator_also_stringify_as。

### Lines 91-120 / 第 91-120 行
```cpp
0091: 
0092:     for (int i = 1; i < len; ++i) {
0093:       b2 = b1;
0094:       b1 = b0;
0095:       b0 = x * b1 - b2 + array[i];
0096:     }
0097: 
0098:     return T{0.5} * (b0 - b2);
0099:   }
0100: 
0101:   template <typename T>
0102:   JITERATOR_HOST_DEVICE T calc_i0e(T _x) {
0103:     T x = std::fabs(_x);
0104: 
0105:     if (x <= T{8.0}) {
0106:       static const T coefficients[] = {
0107:           -4.41534164647933937950E-18, 3.33079451882223809783E-17,
0108:           -2.43127984654795469359E-16, 1.71539128555513303061E-15,
0109:           -1.16853328779934516808E-14, 7.67618549860493561688E-14,
0110:           -4.85644678311192946090E-13, 2.95505266312963983461E-12,
0111:           -1.72682629144155570723E-11, 9.67580903537323691224E-11,
0112:           -5.18979560163526290666E-10, 2.65982372468238665035E-9,
0113:           -1.30002500998624804212E-8,  6.04699502254191894932E-8,
0114:           -2.67079385394061173391E-7,  1.11738753912010371815E-6,
0115:           -4.41673835845875056359E-6,  1.64484480707288970893E-5,
0116:           -5.75419501008210370398E-5,  1.88502885095841655729E-4,
0117:           -5.76375574538582365885E-4,  1.63947561694133579842E-3,
0118:           -4.32430999505057594430E-3,  1.05464603945949983183E-2,
0119:           -2.37374148058994688156E-2,  4.93052842396707084878E-2,
0120:           -9.49010970480476444210E-2,  1.71620901522208775349E-1,
```
- **EN**: Lines 91-120 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: calc_i0e, fabs.
- **CN**: 第 91-120 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：calc_i0e, fabs。

### Lines 121-150 / 第 121-150 行
```cpp
0121:           -3.04682672343198398683E-1,  6.76795274409476084995E-1};
0122: 
0123:       T y = (x / T{2.0}) - T{2.0};
0124:       return chbevl(y, coefficients, int{30});
0125:     }
0126: 
0127:     // x > 8
0128:     static const T coefficients[] = {
0129:         -7.23318048787475395456E-18, -4.83050448594418207126E-18,
0130:         4.46562142029675999901E-17,  3.46122286769746109310E-17,
0131:         -2.82762398051658348494E-16, -3.42548561967721913462E-16,
0132:         1.77256013305652638360E-15,  3.81168066935262242075E-15,
0133:         -9.55484669882830764870E-15, -4.15056934728722208663E-14,
0134:         1.54008621752140982691E-14,  3.85277838274214270114E-13,
0135:         7.18012445138366623367E-13,  -1.79417853150680611778E-12,
0136:         -1.32158118404477131188E-11, -3.14991652796324136454E-11,
0137:         1.18891471078464383424E-11,  4.94060238822496958910E-10,
0138:         3.39623202570838634515E-9,   2.26666899049817806459E-8,
0139:         2.04891858946906374183E-7,   2.89137052083475648297E-6,
0140:         6.88975834691682398426E-5,   3.36911647825569408990E-3,
0141:         8.04490411014108831608E-1};
0142: 
0143:     return chbevl(T{32.0} / x - T{2.0}, coefficients, int{25}) / std::sqrt(x);
0144:   }),
0145:   i0e_string) // i0e_string
0146: }
0147: 
0148: #define CENTRAL_RANGE 0.7
0149: 
0150: template <typename T>
```
- **EN**: Lines 121-150 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: chbevl, sqrt.
- **CN**: 第 121-150 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：chbevl, sqrt。

### Lines 151-180 / 第 151-180 行
```cpp
0151: inline typename std::enable_if_t<std::is_floating_point_v<T>, T>
0152: calc_erfinv(T y) {
0153: /* Function to calculate inverse error function.  Rational approximation
0154: is used to generate an initial approximation, which is then improved to
0155: full accuracy by two steps of Newton's method.  Code is a direct
0156: translation of the erfinv m file in matlab version 2.0.
0157: Author:  Gary L. Pavlis, Indiana University
0158: Date:  February 1996
0159: */
0160:   T x, z, num, dem; /*working variables */
0161:   /* coefficients in rational expansion */
0162:   T a[4] = {  T(0.886226899), T(-1.645349621),  T(0.914624893), T(-0.140543331) };
0163:   T b[4] = { T(-2.118377725),  T(1.442710462), T(-0.329097515),  T(0.012229801) };
0164:   T c[4] = { T(-1.970840454), T(-1.624906493),  T(3.429567803),  T(1.641345311) };
0165:   T d[2] = {  T(3.543889200),  T(1.637067800) };
0166:   T y_abs = std::abs(y);
0167:   if(y_abs > 1.0) return std::numeric_limits<T>::quiet_NaN();
0168: #ifdef _WIN32
0169:   // error C2039: '_copysign': is not a member of 'std'
0170:   if(y_abs == 1.0) return copysign(std::numeric_limits<T>::infinity(), y);
0171: #else
0172:   if(y_abs == 1.0) return std::copysign(std::numeric_limits<T>::infinity(), y);
0173: #endif
0174:   if(y_abs <= static_cast<T>(CENTRAL_RANGE)) {
0175:     z = y * y;
0176:     num = (((a[3]*z + a[2])*z + a[1])*z + a[0]);
0177:     dem = ((((b[3]*z + b[2])*z + b[1])*z +b[0]) * z + static_cast<T>(1.0));
0178:     x = y * num / dem;
0179:   }
0180:   else{
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: calc_erfinv, T, abs, quiet_NaN.
- **CN**: 第 151-180 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：calc_erfinv, T, abs, quiet_NaN。

### Lines 181-210 / 第 181-210 行
```cpp
0181:     z = std::sqrt(-std::log((static_cast<T>(1.0)-y_abs)/static_cast<T>(2.0)));
0182:     num = ((c[3]*z + c[2])*z + c[1]) * z + c[0];
0183:     dem = (d[1]*z + d[0])*z + static_cast<T>(1.0);
0184: #ifdef _WIN32
0185:     // error C2039: '_copysign': is not a member of 'std'
0186:     x = copysign(num, y) / dem;
0187: #else
0188:     x = std::copysign(num, y) / dem;
0189: #endif
0190:   }
0191:   /* Two steps of Newton-Raphson correction */
0192:   x = x - (std::erf(x) - y) / ((static_cast<T>(2.0)*c10::frac_1_sqrt_pi<T>)*std::exp(-x*x));
0193:   x = x - (std::erf(x) - y) / ((static_cast<T>(2.0)*c10::frac_1_sqrt_pi<T>)*std::exp(-x*x));
0194: 
0195:   return x;
0196: }
0197: 
0198: #undef CENTRAL_RANGE
0199: 
0200: /*
0201:  * Note [3-Clause BSD License for the Cephes Math Library]
0202:  * Code derived from implementations in the Cephes Math Library should mention its derivation and reference
0203:  * this note (ex. 'This function is derived from the implementation of X in the Cephes Math Library. See note
0204:  * [3-Clause BSD License for the Cephes Math Library]. The license is:
0205:  * Copyright (c) 2018, Steven Moshier
0206:  * All rights reserved.
0207:  *
0208:  * Redistribution and use in source and binary forms, with or without
0209:  * modification, are permitted provided that the following conditions are met:
0210:  * * Redistributions of source code must retain the above copyright
```
- **EN**: Lines 181-210 mainly cover comments/documentation, state/variable declarations, conditional compilation. Notable symbols: sqrt, log, copysign, erf.
- **CN**: 第 181-210 行主要涉及注释或说明、变量/别名声明、预处理条件。 值得关注的符号包括：sqrt, log, copysign, erf。

### Lines 211-240 / 第 211-240 行
```cpp
0211:  * notice, this list of conditions and the following disclaimer.
0212:  * * Redistributions in binary form must reproduce the above copyright
0213:  * notice, this list of conditions and the following disclaimer in the
0214:  * documentation and/or other materials provided with the distribution.
0215:  * * Neither the name of the nor the
0216:  * names of its contributors may be used to endorse or promote products
0217:  * derived from this software without specific prior written permission.
0218:  *
0219:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
0220:  * ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
0221:  * WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
0222:  * DISCLAIMED. IN NO EVENT SHALL Steven Moshier BE LIABLE FOR ANY
0223:  * DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
0224:  * (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
0225:  * LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
0226:  * ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
0227:  * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
0228:  * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
0229:  */
0230: 
0231: /*
0232:  * This function is derived from the implementation of the zeta function in the Cephes Math Library.
0233:  * See note [3-Clause BSD License for the Cephes Math Library].
0234:  */
0235: template <typename scalar_t, bool is_cuda=false>
0236: C10_HOST_DEVICE inline scalar_t zeta(scalar_t x, scalar_t q) __ubsan_ignore_float_divide_by_zero__ {
0237:   using acc_t = at::acc_type<scalar_t, is_cuda>;
0238:   const acc_t MACHEP = acc_t{1.11022302462515654042E-16};
0239:   constexpr acc_t zero = acc_t{0.0};
0240:   constexpr acc_t half = acc_t{0.5};
```
- **EN**: Lines 211-240 mainly cover comments/documentation, state/variable declarations, template setup. Notable symbols: zeta.
- **CN**: 第 211-240 行主要涉及注释或说明、变量/别名声明、模板声明。 值得关注的符号包括：zeta。

### Lines 241-270 / 第 241-270 行
```cpp
0241:   constexpr acc_t one = acc_t{1.0};
0242:   static const acc_t A[] = {
0243:       12.0,
0244:       -720.0,
0245:       30240.0,
0246:       -1209600.0,
0247:       47900160.0,
0248:       -1.8924375803183791606e9, /*1.307674368e12/691*/
0249:       7.47242496e10,
0250:       -2.950130727918164224e12, /*1.067062284288e16/3617*/
0251:       1.1646782814350067249e14, /*5.109094217170944e18/43867*/
0252:       -4.5979787224074726105e15, /*8.028576626982912e20/174611*/
0253:       1.8152105401943546773e17, /*1.5511210043330985984e23/854513*/
0254:       -7.1661652561756670113e18 /*1.6938241367317436694528e27/236364091*/
0255:   };
0256: 
0257:   acc_t a, b, k, s, t, w;
0258:   if (x == one) {
0259:     return std::numeric_limits<scalar_t>::infinity();
0260:   }
0261: 
0262:   if (x < one) {
0263:     return std::numeric_limits<scalar_t>::quiet_NaN();
0264:   }
0265: 
0266:   if (q <= zero) {
0267:     if (q == std::floor(q)) {
0268:       return std::numeric_limits<scalar_t>::infinity();
0269:     }
0270:     if (x != std::floor(x)) {
```
- **EN**: Lines 241-270 mainly cover expressions/calls, comments/documentation, control-flow checks. Notable symbols: infinity, quiet_NaN, floor.
- **CN**: 第 241-270 行主要涉及表达式或调用、注释或说明、控制流逻辑。 值得关注的符号包括：infinity, quiet_NaN, floor。

### Lines 271-300 / 第 271-300 行
```cpp
0271:       return std::numeric_limits<scalar_t>::quiet_NaN();
0272:     }
0273:   }
0274: 
0275:   s = std::pow(q, -x);
0276:   a = q;
0277:   int i = 0;
0278:   b = zero;
0279:   while ((i < 9) || (a <= acc_t{9.0})) {
0280:     i += 1;
0281:     a += one;
0282:     b = ::pow(a, -x);
0283:     s += b;
0284:     if ((-MACHEP * s < b) && (b < MACHEP * s)) {
0285:       return static_cast<scalar_t>(s);
0286:     }
0287:   };
0288: 
0289:   w = a;
0290:   s += b * w / (x - one);
0291:   s -= half * b;
0292:   a = one;
0293:   k = zero;
0294:   for (i = 0; i < 12; i++) {
0295:     a *= x + k;
0296:     b /= w;
0297:     t = a * b / A[i];
0298:     s = s + t;
0299:     t = ::fabs(t / s);
0300:     if (t < MACHEP) {
```
- **EN**: Lines 271-300 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: quiet_NaN, pow, fabs.
- **CN**: 第 271-300 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：quiet_NaN, pow, fabs。

### Lines 301-330 / 第 301-330 行
```cpp
0301:       return static_cast<scalar_t>(s);
0302:     }
0303:     k += one;
0304:     a *= x + k;
0305:     b /= w;
0306:     k += one;
0307:   }
0308:   return static_cast<scalar_t>(s);
0309: }
0310: 
0311: /*
0312:  * This function is derived from the implementation of the digamma function in the Cephes Math Library.
0313:  * See note [3-Clause BSD License for the Cephes Math Library].
0314:  *
0315:  * Evaluates polynomial of degree N:
0316:  *
0317:  *                     2          N
0318:  * y  =  C  + C x + C x  +...+ C x
0319:  *        0    1     2          N
0320:  *
0321:  * Coefficients are stored in reverse order:
0322:  *
0323:  * coef[0] = C  , ..., coef[N] = C  .
0324:  *            N                   0
0325:  */
0326: template <typename T>
0327: C10_HOST_DEVICE inline T polevl(const T x, const T A[], size_t len) {
0328:   T result = 0;
0329:   for (size_t i = 0; i <= len; i++) {
0330:     result = result * x + A[i];
```
- **EN**: Lines 301-330 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: polevl.
- **CN**: 第 301-330 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：polevl。

### Lines 331-360 / 第 331-360 行
```cpp
0331:   }
0332:   return result;
0333: }
0334: 
0335: inline double trigamma(double x) __ubsan_ignore_float_divide_by_zero__ {
0336:   double sign = +1;
0337:   double result = 0;
0338:   if (x < 0.5) {
0339:     sign = -1;
0340:     const double sin_pi_x = sin(c10::pi<double> * x);
0341:     result -= (c10::pi<double> * c10::pi<double>) / (sin_pi_x * sin_pi_x);
0342:     x = 1 - x;
0343:   }
0344:   for (int i = 0; i < 6; ++i) {
0345:     result += 1 / (x * x);
0346:     x += 1;
0347:   }
0348:   const double ixx = 1 / (x*x);
0349:   result += (1 + 1 / (2*x) + ixx * (1./6 - ixx * (1./30 - ixx * (1./42)))) / x;
0350:   return sign * result;
0351: }
0352: 
0353: inline float trigamma(float x) __ubsan_ignore_float_divide_by_zero__ {
0354:   float sign = +1;
0355:   float result = 0;
0356:   if (x < 0.5f) {
0357:     sign = -1;
0358:     const float sin_pi_x = sinf(c10::pi<float> * x);
0359:     result -= (c10::pi<float> * c10::pi<float>) / (sin_pi_x * sin_pi_x);
0360:     x = 1 - x;
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: trigamma, sin, sinf.
- **CN**: 第 331-360 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：trigamma, sin, sinf。

### Lines 361-390 / 第 361-390 行
```cpp
0361:   }
0362:   for (int i = 0; i < 6; ++i) {
0363:     result += 1 / (x * x);
0364:     x += 1;
0365:   }
0366:   const float ixx = 1 / (x*x);
0367:   result += (1 + 1 / (2*x) + ixx * (1.f/6 - ixx * (1.f/30 - ixx * (1.f/42)))) / x;
0368:   return sign * result;
0369: }
0370: 
0371: /*
0372:  * This function is derived from the implementation of the digamma function in the Cephes Math Library.
0373:  * See note [3-Clause BSD License for the Cephes Math Library].
0374:  */
0375: inline double calc_digamma(double x) {
0376:   // [C++ Standard Reference: Gamma Function] https://en.cppreference.com/w/cpp/numeric/math/tgamma
0377:   static double PSI_10 = 2.25175258906672110764;
0378:   if (x == 0) {
0379:     // As per C++ standard for gamma related functions and SciPy,
0380:     // If the argument is ±0, ±∞ is returned
0381:     return std::copysign(INFINITY, -x);
0382:   }
0383: 
0384:   bool x_is_integer = x == trunc(x);
0385:   if (x < 0) {
0386:     if (x_is_integer) {
0387:       // As per C++ standard for gamma related functions and SciPy,
0388:       // If the argument is a negative integer, NaN is returned
0389:       return std::numeric_limits<double>::quiet_NaN();
0390:     }
```
- **EN**: Lines 361-390 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: calc_digamma, copysign, trunc, quiet_NaN.
- **CN**: 第 361-390 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：calc_digamma, copysign, trunc, quiet_NaN。

### Lines 391-420 / 第 391-420 行
```cpp
0391:     // Extracts the fractional part of x as r, since tan(pi * r) is more numerically
0392:     // accurate than tan(pi * x). While these operations are mathematically equivalent
0393:     // since both x and r are in radians and tan() has a periodicity of pi, in practice
0394:     // the computation of pi * x is a source of error (when |x| > 1).
0395:     double q, r;
0396:     r = std::modf(x, &q);
0397:     return calc_digamma(1 - x) - c10::pi<double> / tan(c10::pi<double> * r);
0398:   }
0399: 
0400:   // Push x to be >= 10
0401:   double result = 0;
0402:   while (x < 10) {
0403:     result -= 1 / x;
0404:     x += 1;
0405:   }
0406:   if (x == 10) {
0407:     return result + PSI_10;
0408:   }
0409: 
0410:   // Compute asymptotic digamma
0411:   static const double A[] = {
0412:       8.33333333333333333333E-2,
0413:       -2.10927960927960927961E-2,
0414:       7.57575757575757575758E-3,
0415:       -4.16666666666666666667E-3,
0416:       3.96825396825396825397E-3,
0417:       -8.33333333333333333333E-3,
0418:       8.33333333333333333333E-2,
0419:   };
0420: 
```
- **EN**: Lines 391-420 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: tan, error, modf, calc_digamma.
- **CN**: 第 391-420 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：tan, error, modf, calc_digamma。

### Lines 421-450 / 第 421-450 行
```cpp
0421:   double y = 0;
0422:   if (x < 1.0e17) {
0423:     double z = 1.0 / (x * x);
0424:     y = z * polevl(z, A, 6);
0425:   }
0426:   return result + log(x) - (0.5 / x) - y;
0427: }
0428: 
0429: /*
0430:  * This function is derived from the implementation of the digamma function in the Cephes Math Library.
0431:  * See note [3-Clause BSD License for the Cephes Math Library].
0432:  */
0433: inline float calc_digamma(float x) {
0434:   // See [C++ Standard Reference: Gamma Function]
0435:   static float PSI_10 = 2.25175258906672110764f;
0436:   if (x == 0) {
0437:     // As per C++ standard for gamma related functions and SciPy,
0438:     // If the argument is ±0, ±∞ is returned
0439:     return std::copysign(INFINITY, -x);
0440:   }
0441: 
0442:   bool x_is_integer = x == truncf(x);
0443:   if (x < 0) {
0444:     if (x_is_integer) {
0445:     // As per C++ standard for gamma related functions and SciPy,
0446:     // If the argument is a negative integer, NaN is returned
0447:       return std::numeric_limits<float>::quiet_NaN();
0448:     }
0449:     // Extracts the fractional part of x as r, since tan(pi * r) is more numerically
0450:     // accurate than tan(pi * x). While these operations are mathematically equivalent
```
- **EN**: Lines 421-450 mainly cover comments/documentation, state/variable declarations, control-flow checks. Notable symbols: polevl, log, calc_digamma, copysign.
- **CN**: 第 421-450 行主要涉及注释或说明、变量/别名声明、控制流逻辑。 值得关注的符号包括：polevl, log, calc_digamma, copysign。

### Lines 451-480 / 第 451-480 行
```cpp
0451:     // since both x and r are in radians and tan() has a periodicity of pi, in practice
0452:     // the computation of pi * x is a source of error (when |x| > 1).
0453:     double q, r;
0454:     r = std::modf(x, &q);
0455:     float pi_over_tan_pi_x = (float)(c10::pi<double> / tan(c10::pi<double> * r));
0456:     return calc_digamma(1 - x) - pi_over_tan_pi_x;
0457:   }
0458: 
0459:   // Push x to be >= 10
0460:   float result = 0;
0461:   while (x < 10) {
0462:     result -= 1 / x;
0463:     x += 1;
0464:   }
0465:   if (x == 10) {
0466:     return result + PSI_10;
0467:   }
0468: 
0469:   // Compute asymptotic digamma
0470:   static const float A[] = {
0471:       8.33333333333333333333E-2f,
0472:       -2.10927960927960927961E-2f,
0473:       7.57575757575757575758E-3f,
0474:       -4.16666666666666666667E-3f,
0475:       3.96825396825396825397E-3f,
0476:       -8.33333333333333333333E-3f,
0477:       8.33333333333333333333E-2f,
0478:   };
0479: 
0480:   float y = 0;
```
- **EN**: Lines 451-480 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: tan, error, modf, calc_digamma.
- **CN**: 第 451-480 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：tan, error, modf, calc_digamma。

### Lines 481-510 / 第 481-510 行
```cpp
0481:   if (x < 1.0e17f) {
0482:     float z = 1 / (x * x);
0483:     y = z * polevl(z, A, 6);
0484:   }
0485:   return result + logf(x) - (0.5f / x) - y;
0486: }
0487: 
0488: inline c10::BFloat16 calc_digamma(c10::BFloat16 a) {
0489:   return calc_digamma(static_cast<float>(a));
0490: }
0491: 
0492: inline c10::Half calc_digamma(c10::Half a) {
0493:   return calc_digamma(static_cast<float>(a));
0494: }
0495: 
0496: template <typename scalar_t, bool is_cuda=false>
0497: inline C10_HOST_DEVICE scalar_t calc_polygamma(scalar_t x, int n) {
0498:   // already blocked if n <= 1
0499:   const auto one = scalar_t{1};
0500:   return ((n % 2) ? one : -one) *
0501:       std::exp(std::lgamma(static_cast<scalar_t>(n) + one)) *
0502:       zeta<scalar_t, is_cuda>(static_cast<scalar_t>(n + 1), x);
0503: }
0504: 
0505: // regularized lower incomplete gamma
0506: // the regularized lower, upper incomplete gamma, as well as their
0507: // helper functions follow SciPy's implementation
0508: 
0509: /* References
0510:  * [igam1] "The Digital Library of Mathematical Functions", dlmf.nist.gov
```
- **EN**: Lines 481-510 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: polevl, logf, calc_digamma, calc_polygamma.
- **CN**: 第 481-510 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：polevl, logf, calc_digamma, calc_polygamma。

### Lines 511-540 / 第 511-540 行
```cpp
0511:  * [igam2] Maddock et al., "Incomplete Gamma Functions",
0512:  *     https://www.boost.org/doc/libs/1_61_0/libs/math/doc/html/math_toolkit/sf_gamma/igamma.html
0513:  */
0514: 
0515: /*
0516:  * This implementation of the regularized incomplete gamma functions and
0517:  * their helper functions are derived from the implementation of SciPy's
0518:  * gammainc, Cephes's igam and igamc, and Boost's Lanczos approximations.
0519:  * See NOTICE for the licenses.
0520:  */
0521: template <typename scalar_t>
0522: scalar_t ratevl(scalar_t x, const scalar_t num[], int64_t M,
0523:     const scalar_t denom[], int64_t N) {
0524:   // evaluating rational function, i.e., the ratio of two polynomials
0525:   // the coefficients for numerator are given by `num` while coeffs for
0526:   // denumerator are given by `denom`
0527: 
0528:   int64_t i, dir;
0529:   scalar_t y, num_ans, denom_ans;
0530:   scalar_t absx = std::fabs(x);
0531:   const scalar_t *p;
0532: 
0533:   if (absx > 1) {
0534:     /* Evaluate as a polynomial in 1/x. */
0535:     dir = -1;
0536:     p = num + M;
0537:     y = 1 / x;
0538:   }
0539:   else {
0540:     dir = 1;
```
- **EN**: Lines 511-540 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: ratevl, fabs.
- **CN**: 第 511-540 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：ratevl, fabs。

### Lines 541-570 / 第 541-570 行
```cpp
0541:     p = num;
0542:     y = x;
0543:   }
0544: 
0545:   /* Evaluate the numerator */
0546:   num_ans = *p;
0547:   p += dir;
0548:   for (i = 1; i <= M; i++) {
0549:     num_ans = num_ans * y + *p;
0550:     p += dir;
0551:   }
0552:   /* Evaluate the denominator */
0553:   if (absx > 1) {
0554:     p = denom + N;
0555:   }
0556:   else {
0557:     p = denom;
0558:   }
0559: 
0560:   denom_ans = *p;
0561:   p += dir;
0562:   for (i = 1; i <= N; i++) {
0563:     denom_ans = denom_ans * y + *p;
0564:     p += dir;
0565:   }
0566:   if (absx > 1) {
0567:     i = N - M;
0568:     return std::pow(x, i) * num_ans / denom_ans;
0569:   }
0570:   else {
```
- **EN**: Lines 541-570 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: pow.
- **CN**: 第 541-570 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：pow。

### Lines 571-600 / 第 571-600 行
```cpp
0571:     return num_ans / denom_ans;
0572:   }
0573: }
0574: 
0575: // SciPy's lanczos implementation is taken from Boost
0576: /* (C) Copyright John Maddock 2006.
0577:  * Use, modification and distribution are subject to the
0578:  * Boost Software License, Version 1.0. See
0579:  * https://www.boost.org/LICENSE_1_0.txt or see NOTICE.
0580:  */
0581: template <typename scalar_t>
0582: static scalar_t lanczos_sum_expg_scaled(scalar_t x) {
0583:   // lanczos approximation
0584:   static constexpr scalar_t lanczos_sum_expg_scaled_num[13] = {
0585:     0.006061842346248906525783753964555936883222,
0586:     0.5098416655656676188125178644804694509993,
0587:     19.51992788247617482847860966235652136208,
0588:     449.9445569063168119446858607650988409623,
0589:     6955.999602515376140356310115515198987526,
0590:     75999.29304014542649875303443598909137092,
0591:     601859.6171681098786670226533699352302507,
0592:     3481712.15498064590882071018964774556468,
0593:     14605578.08768506808414169982791359218571,
0594:     43338889.32467613834773723740590533316085,
0595:     86363131.28813859145546927288977868422342,
0596:     103794043.1163445451906271053616070238554,
0597:     56906521.91347156388090791033559122686859
0598:   };
0599:   static constexpr scalar_t lanczos_sum_expg_scaled_denom[13] = {
0600:     1.,
```
- **EN**: Lines 571-600 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: lanczos_sum_expg_scaled.
- **CN**: 第 571-600 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：lanczos_sum_expg_scaled。

### Lines 601-630 / 第 601-630 行
```cpp
0601:     66.,
0602:     1925.,
0603:     32670.,
0604:     357423.,
0605:     2637558.,
0606:     13339535.,
0607:     45995730.,
0608:     105258076.,
0609:     150917976.,
0610:     120543840.,
0611:     39916800.,
0612:     0.
0613:   };
0614:   return ratevl(x, lanczos_sum_expg_scaled_num,
0615:       sizeof(lanczos_sum_expg_scaled_num) / sizeof(lanczos_sum_expg_scaled_num[0]) - 1,
0616:       lanczos_sum_expg_scaled_denom,
0617:       sizeof(lanczos_sum_expg_scaled_denom) / sizeof(lanczos_sum_expg_scaled_denom[0]) - 1);
0618: }
0619: 
0620: template <typename scalar_t>
0621: static scalar_t _igam_helper_fac(scalar_t a, scalar_t x) {
0622:   // compute x^a * exp(-a) / gamma(a)
0623:   // corrected from (15) and (16) in [igam2] by replacing exp(x - a) with
0624:   // exp(a - x).
0625: 
0626:   scalar_t ax, fac, res, num, numfac;
0627:   static scalar_t MAXLOG = std::is_same_v<scalar_t,double> ?
0628:     7.09782712893383996843E2 : 88.72283905206835;
0629:   static scalar_t EXP1 = 2.718281828459045;
0630:   static scalar_t lanczos_g = 6.024680040776729583740234375;
```
- **EN**: Lines 601-630 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: ratevl, _igam_helper_fac, exp, gamma.
- **CN**: 第 601-630 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：ratevl, _igam_helper_fac, exp, gamma。

### Lines 631-660 / 第 631-660 行
```cpp
0631: 
0632:   if (std::fabs(a - x) > 0.4 * std::fabs(a)) {
0633:     ax = a * std::log(x) - x - std::lgamma(a);
0634:     if (ax < -MAXLOG) {
0635:       return 0.0;
0636:     }
0637:     return std::exp(ax);
0638:   }
0639: 
0640:   fac = a + lanczos_g - 0.5;
0641:   res = std::sqrt(fac / EXP1) / lanczos_sum_expg_scaled(a);
0642: 
0643:   if ((a < 200) && (x < 200)) {
0644:     res *= std::exp(a - x) * std::pow(x / fac, a);
0645:   }
0646:   else {
0647:     num = x - a - lanczos_g + 0.5;
0648:     numfac = num / fac;
0649:     res *= std::exp(a * (std::log1p(numfac) - numfac) + x * (0.5 - lanczos_g) / fac);
0650:   }
0651:   return res;
0652: }
0653: 
0654: template <typename scalar_t>
0655: static scalar_t _igam_helper_series(scalar_t a, scalar_t x) {
0656:   // Compute igam using DLMF 8.11.4. [igam1]
0657:   static scalar_t MACHEP = std::is_same_v<scalar_t, double> ?
0658:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
0659:   static int MAXITER = 2000;
0660: 
```
- **EN**: Lines 631-660 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: fabs, log, lgamma, exp.
- **CN**: 第 631-660 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：fabs, log, lgamma, exp。

### Lines 661-690 / 第 661-690 行
```cpp
0661:   int i;
0662:   scalar_t ans, ax, c, r;
0663: 
0664:   ax = _igam_helper_fac(a, x);
0665:   if (ax == 0.0) {
0666:     return 0.0;
0667:   }
0668: 
0669:   /* power series */
0670:   r = a;
0671:   c = 1.0;
0672:   ans = 1.0;
0673: 
0674:   for (i = 0; i < MAXITER; i++) {
0675:     r += 1.0;
0676:     c *= x / r;
0677:     ans += c;
0678:     if (c <= MACHEP * ans) {
0679:       break;
0680:     }
0681:   }
0682:   return (ans * ax / a);
0683: }
0684: 
0685: template <typename scalar_t>
0686: static scalar_t _igamc_helper_series(scalar_t a, scalar_t x) {
0687:   // Compute igamc using DLMF 8.7.3 [igam1]. This is related to the series in
0688:   // _igam_helper_series but extra care is taken to avoid cancellation.
0689: 
0690:   int n;
```
- **EN**: Lines 661-690 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: _igam_helper_fac, _igamc_helper_series.
- **CN**: 第 661-690 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：_igam_helper_fac, _igamc_helper_series。

### Lines 691-720 / 第 691-720 行
```cpp
0691:   scalar_t fac = 1;
0692:   scalar_t sum = 0;
0693:   scalar_t term, logx;
0694:   static scalar_t MAXITER = 2000;
0695:   static scalar_t MACHEP = std::is_same_v<scalar_t, double> ?
0696:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
0697: 
0698:   for (n = 1; n < MAXITER; n++) {
0699:     fac *= -x / n;
0700:     term = fac / (a + n);
0701:     sum += term;
0702:     if (std::fabs(term) <= MACHEP * std::fabs(sum)) {
0703:         break;
0704:     }
0705:   }
0706: 
0707:   logx = std::log(x);
0708:   term = -std::expm1(a * logx - std::lgamma(1+a));
0709:   return term - std::exp(a * logx - std::lgamma(a)) * sum;
0710: }
0711: 
0712: template <typename scalar_t>
0713: static scalar_t _igam_helper_asymptotic_series(scalar_t a, scalar_t x, bool igam) {
0714:   // Compute igam/igamc using DLMF 8.12.3/8.12.4 [igam1]
0715:   static constexpr scalar_t d[25][25] =
0716:     {{-3.3333333333333333e-1, 8.3333333333333333e-2, -1.4814814814814815e-2,
0717:       1.1574074074074074e-3, 3.527336860670194e-4, -1.7875514403292181e-4,
0718:       3.9192631785224378e-5, -2.1854485106799922e-6, -1.85406221071516e-6,
0719:       8.296711340953086e-7, -1.7665952736826079e-7, 6.7078535434014986e-9,
0720:       1.0261809784240308e-8, -4.3820360184533532e-9, 9.1476995822367902e-10,
```
- **EN**: Lines 691-720 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: fabs, log, expm1, lgamma.
- **CN**: 第 691-720 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：fabs, log, expm1, lgamma。

### Lines 721-750 / 第 721-750 行
```cpp
0721:       -2.551419399494625e-11, -5.8307721325504251e-11, 2.4361948020667416e-11,
0722:       -5.0276692801141756e-12, 1.1004392031956135e-13, 3.3717632624009854e-13,
0723:       -1.3923887224181621e-13, 2.8534893807047443e-14, -5.1391118342425726e-16,
0724:       -1.9752288294349443e-15},
0725:     {-1.8518518518518519e-3, -3.4722222222222222e-3, 2.6455026455026455e-3,
0726:       -9.9022633744855967e-4, 2.0576131687242798e-4, -4.0187757201646091e-7,
0727:       -1.8098550334489978e-5, 7.6491609160811101e-6, -1.6120900894563446e-6,
0728:       4.6471278028074343e-9, 1.378633446915721e-7, -5.752545603517705e-8,
0729:       1.1951628599778147e-8, -1.7543241719747648e-11, -1.0091543710600413e-9,
0730:       4.1627929918425826e-10, -8.5639070264929806e-11, 6.0672151016047586e-14,
0731:       7.1624989648114854e-12, -2.9331866437714371e-12, 5.9966963656836887e-13,
0732:       -2.1671786527323314e-16, -4.9783399723692616e-14, 2.0291628823713425e-14,
0733:       -4.13125571381061e-15},
0734:     {4.1335978835978836e-3, -2.6813271604938272e-3, 7.7160493827160494e-4,
0735:       2.0093878600823045e-6, -1.0736653226365161e-4, 5.2923448829120125e-5,
0736:       -1.2760635188618728e-5, 3.4235787340961381e-8, 1.3721957309062933e-6,
0737:       -6.298992138380055e-7, 1.4280614206064242e-7, -2.0477098421990866e-10,
0738:       -1.4092529910867521e-8, 6.228974084922022e-9, -1.3670488396617113e-9,
0739:       9.4283561590146782e-13, 1.2872252400089318e-10, -5.5645956134363321e-11,
0740:       1.1975935546366981e-11, -4.1689782251838635e-15, -1.0940640427884594e-12,
0741:       4.6622399463901357e-13, -9.905105763906906e-14, 1.8931876768373515e-17,
0742:       8.8592218725911273e-15},
0743:     {6.4943415637860082e-4, 2.2947209362139918e-4, -4.6918949439525571e-4,
0744:       2.6772063206283885e-4, -7.5618016718839764e-5, -2.3965051138672967e-7,
0745:       1.1082654115347302e-5, -5.6749528269915966e-6, 1.4230900732435884e-6,
0746:       -2.7861080291528142e-11, -1.6958404091930277e-7, 8.0994649053880824e-8,
0747:       -1.9111168485973654e-8, 2.3928620439808118e-12, 2.0620131815488798e-9,
0748:       -9.4604966618551322e-10, 2.1541049775774908e-10, -1.388823336813903e-14,
0749:       -2.1894761681963939e-11, 9.7909989511716851e-12, -2.1782191880180962e-12,
0750:       6.2088195734079014e-17, 2.126978363279737e-13, -9.3446887915174333e-14,
```
- **EN**: Lines 721-750 mainly cover expressions/calls.
- **CN**: 第 721-750 行主要涉及表达式或调用。

### Lines 751-780 / 第 751-780 行
```cpp
0751:       2.0453671226782849e-14},
0752:     {-8.618882909167117e-4, 7.8403922172006663e-4, -2.9907248030319018e-4,
0753:       -1.4638452578843418e-6, 6.6414982154651222e-5, -3.9683650471794347e-5,
0754:       1.1375726970678419e-5, 2.5074972262375328e-10, -1.6954149536558306e-6,
0755:       8.9075075322053097e-7, -2.2929348340008049e-7, 2.956794137544049e-11,
0756:       2.8865829742708784e-8, -1.4189739437803219e-8, 3.4463580499464897e-9,
0757:       -2.3024517174528067e-13, -3.9409233028046405e-10, 1.8602338968504502e-10,
0758:       -4.356323005056618e-11, 1.2786001016296231e-15, 4.6792750266579195e-12,
0759:       -2.1492464706134829e-12, 4.9088156148096522e-13, -6.3385914848915603e-18,
0760:       -5.0453320690800944e-14},
0761:     {-3.3679855336635815e-4, -6.9728137583658578e-5, 2.7727532449593921e-4,
0762:       -1.9932570516188848e-4, 6.7977804779372078e-5, 1.419062920643967e-7,
0763:       -1.3594048189768693e-5, 8.0184702563342015e-6, -2.2914811765080952e-6,
0764:       -3.252473551298454e-10, 3.4652846491085265e-7, -1.8447187191171343e-7,
0765:       4.8240967037894181e-8, -1.7989466721743515e-14, -6.3061945000135234e-9,
0766:       3.1624176287745679e-9, -7.8409242536974293e-10, 5.1926791652540407e-15,
0767:       9.3589442423067836e-11, -4.5134262161632782e-11, 1.0799129993116827e-11,
0768:       -3.661886712685252e-17, -1.210902069055155e-12, 5.6807435849905643e-13,
0769:       -1.3249659916340829e-13},
0770:     {5.3130793646399222e-4, -5.9216643735369388e-4, 2.7087820967180448e-4,
0771:       7.9023532326603279e-7, -8.1539693675619688e-5, 5.6116827531062497e-5,
0772:       -1.8329116582843376e-5, -3.0796134506033048e-9, 3.4651553688036091e-6,
0773:       -2.0291327396058604e-6, 5.7887928631490037e-7, 2.338630673826657e-13,
0774:       -8.8286007463304835e-8, 4.7435958880408128e-8, -1.2545415020710382e-8,
0775:       8.6496488580102925e-14, 1.6846058979264063e-9, -8.5754928235775947e-10,
0776:       2.1598224929232125e-10, -7.6132305204761539e-16, -2.6639822008536144e-11,
0777:       1.3065700536611057e-11, -3.1799163902367977e-12, 4.7109761213674315e-18,
0778:       3.6902800842763467e-13},
0779:     {3.4436760689237767e-4, 5.1717909082605922e-5, -3.3493161081142236e-4,
0780:       2.812695154763237e-4, -1.0976582244684731e-4, -1.2741009095484485e-7,
```
- **EN**: Lines 751-780 mainly cover expressions/calls.
- **CN**: 第 751-780 行主要涉及表达式或调用。

### Lines 781-810 / 第 781-810 行
```cpp
0781:       2.7744451511563644e-5, -1.8263488805711333e-5, 5.7876949497350524e-6,
0782:       4.9387589339362704e-10, -1.0595367014026043e-6, 6.1667143761104075e-7,
0783:       -1.7562973359060462e-7, -1.2974473287015439e-12, 2.695423606288966e-8,
0784:       -1.4578352908731271e-8, 3.887645959386175e-9, -3.8810022510194121e-17,
0785:       -5.3279941738772867e-10, 2.7437977643314845e-10, -6.9957960920705679e-11,
0786:       2.5899863874868481e-17, 8.8566890996696381e-12, -4.403168815871311e-12,
0787:       1.0865561947091654e-12},
0788:     {-6.5262391859530942e-4, 8.3949872067208728e-4, -4.3829709854172101e-4,
0789:       -6.969091458420552e-7, 1.6644846642067548e-4, -1.2783517679769219e-4,
0790:       4.6299532636913043e-5, 4.5579098679227077e-9, -1.0595271125805195e-5,
0791:       6.7833429048651666e-6, -2.1075476666258804e-6, -1.7213731432817145e-11,
0792:       3.7735877416110979e-7, -2.1867506700122867e-7, 6.2202288040189269e-8,
0793:       6.5977038267330006e-16, -9.5903864974256858e-9, 5.2132144922808078e-9,
0794:       -1.3991589583935709e-9, 5.382058999060575e-16, 1.9484714275467745e-10,
0795:       -1.0127287556389682e-10, 2.6077347197254926e-11, -5.0904186999932993e-18,
0796:       -3.3721464474854592e-12},
0797:     {-5.9676129019274625e-4, -7.2048954160200106e-5, 6.7823088376673284e-4,
0798:       -6.4014752602627585e-4, 2.7750107634328704e-4, 1.8197008380465151e-7,
0799:       -8.4795071170685032e-5, 6.105192082501531e-5, -2.1073920183404862e-5,
0800:       -8.8585890141255994e-10, 4.5284535953805377e-6, -2.8427815022504408e-6,
0801:       8.7082341778646412e-7, 3.6886101871706965e-12, -1.5344695190702061e-7,
0802:       8.862466778790695e-8, -2.5184812301826817e-8, -1.0225912098215092e-14,
0803:       3.8969470758154777e-9, -2.1267304792235635e-9, 5.7370135528051385e-10,
0804:       -1.887749850169741e-19, -8.0931538694657866e-11, 4.2382723283449199e-11,
0805:       -1.1002224534207726e-11},
0806:     {1.3324454494800656e-3, -1.9144384985654775e-3, 1.1089369134596637e-3,
0807:       9.932404122642299e-7, -5.0874501293093199e-4, 4.2735056665392884e-4,
0808:       -1.6858853767910799e-4, -8.1301893922784998e-9, 4.5284402370562147e-5,
0809:       -3.127053674781734e-5, 1.044986828530338e-5, 4.8435226265680926e-11,
0810:       -2.1482565873456258e-6, 1.329369701097492e-6, -4.0295693092101029e-7,
```
- **EN**: Lines 781-810 mainly cover expressions/calls.
- **CN**: 第 781-810 行主要涉及表达式或调用。

### Lines 811-840 / 第 811-840 行
```cpp
0811:       -1.7567877666323291e-13, 7.0145043163668257e-8, -4.040787734999483e-8,
0812:       1.1474026743371963e-8, 3.9642746853563325e-18, -1.7804938269892714e-9,
0813:       9.7480262548731646e-10, -2.6405338676507616e-10, 5.794875163403742e-18,
0814:       3.7647749553543836e-11},
0815:     {1.579727660730835e-3, 1.6251626278391582e-4, -2.0633421035543276e-3,
0816:       2.1389686185689098e-3, -1.0108559391263003e-3, -3.9912705529919201e-7,
0817:       3.6235025084764691e-4, -2.8143901463712154e-4, 1.0449513336495887e-4,
0818:       2.1211418491830297e-9, -2.5779417251947842e-5, 1.7281818956040463e-5,
0819:       -5.6413773872904282e-6, -1.1024320105776174e-11, 1.1223224418895175e-6,
0820:       -6.8693396379526735e-7, 2.0653236975414887e-7, 4.6714772409838506e-14,
0821:       -3.5609886164949055e-8, 2.0470855345905963e-8, -5.8091738633283358e-9,
0822:       -1.332821287582869e-16, 9.0354604391335133e-10, -4.9598782517330834e-10,
0823:       1.3481607129399749e-10},
0824:     {-4.0725121195140166e-3, 6.4033628338080698e-3, -4.0410161081676618e-3,
0825:       -2.183732802866233e-6, 2.1740441801254639e-3, -1.9700440518418892e-3,
0826:       8.3595469747962458e-4, 1.9445447567109655e-8, -2.5779387120421696e-4,
0827:       1.9009987368139304e-4, -6.7696499937438965e-5, -1.4440629666426572e-10,
0828:       1.5712512518742269e-5, -1.0304008744776893e-5, 3.304517767401387e-6,
0829:       7.9829760242325709e-13, -6.4097794149313004e-7, 3.8894624761300056e-7,
0830:       -1.1618347644948869e-7, -2.816808630596451e-15, 1.9878012911297093e-8,
0831:       -1.1407719956357511e-8, 3.2355857064185555e-9, 4.1759468293455945e-20,
0832:       -5.0423112718105824e-10},
0833:     {-5.9475779383993003e-3, -5.4016476789260452e-4, 8.7910413550767898e-3,
0834:       -9.8576315587856125e-3, 5.0134695031021538e-3, 1.2807521786221875e-6,
0835:       -2.0626019342754683e-3, 1.7109128573523058e-3, -6.7695312714133799e-4,
0836:       -6.9011545676562133e-9, 1.8855128143995902e-4, -1.3395215663491969e-4,
0837:       4.6263183033528039e-5, 4.0034230613321351e-11, -1.0255652921494033e-5,
0838:       6.612086372797651e-6, -2.0913022027253008e-6, -2.0951775649603837e-13,
0839:       3.9756029041993247e-7, -2.3956211978815887e-7, 7.1182883382145864e-8,
0840:       8.925574873053455e-16, -1.2101547235064676e-8, 6.9350618248334386e-9,
```
- **EN**: Lines 811-840 mainly cover expressions/calls.
- **CN**: 第 811-840 行主要涉及表达式或调用。

### Lines 841-870 / 第 841-870 行
```cpp
0841:       -1.9661464453856102e-9},
0842:     {1.7402027787522711e-2, -2.9527880945699121e-2, 2.0045875571402799e-2,
0843:       7.0289515966903407e-6, -1.2375421071343148e-2, 1.1976293444235254e-2,
0844:       -5.4156038466518525e-3, -6.3290893396418616e-8, 1.8855118129005065e-3,
0845:       -1.473473274825001e-3, 5.5515810097708387e-4, 5.2406834412550662e-10,
0846:       -1.4357913535784836e-4, 9.9181293224943297e-5, -3.3460834749478311e-5,
0847:       -3.5755837291098993e-12, 7.1560851960630076e-6, -4.5516802628155526e-6,
0848:       1.4236576649271475e-6, 1.8803149082089664e-14, -2.6623403898929211e-7,
0849:       1.5950642189595716e-7, -4.7187514673841102e-8, -6.5107872958755177e-17,
0850:       7.9795091026746235e-9},
0851:     {3.0249124160905891e-2, 2.4817436002649977e-3, -4.9939134373457022e-2,
0852:       5.9915643009307869e-2, -3.2483207601623391e-2, -5.7212968652103441e-6,
0853:       1.5085251778569354e-2, -1.3261324005088445e-2, 5.5515262632426148e-3,
0854:       3.0263182257030016e-8, -1.7229548406756723e-3, 1.2893570099929637e-3,
0855:       -4.6845138348319876e-4, -1.830259937893045e-10, 1.1449739014822654e-4,
0856:       -7.7378565221244477e-5, 2.5625836246985201e-5, 1.0766165333192814e-12,
0857:       -5.3246809282422621e-6, 3.349634863064464e-6, -1.0381253128684018e-6,
0858:       -5.608909920621128e-15, 1.9150821930676591e-7, -1.1418365800203486e-7,
0859:       3.3654425209171788e-8},
0860:     {-9.9051020880159045e-2, 1.7954011706123486e-1, -1.2989606383463778e-1,
0861:       -3.1478872752284357e-5, 9.0510635276848131e-2, -9.2828824411184397e-2,
0862:       4.4412112839877808e-2, 2.7779236316835888e-7, -1.7229543805449697e-2,
0863:       1.4182925050891573e-2, -5.6214161633747336e-3, -2.39598509186381e-9,
0864:       1.6029634366079908e-3, -1.1606784674435773e-3, 4.1001337768153873e-4,
0865:       1.8365800754090661e-11, -9.5844256563655903e-5, 6.3643062337764708e-5,
0866:       -2.076250624489065e-5, -1.1806020912804483e-13, 4.2131808239120649e-6,
0867:       -2.6262241337012467e-6, 8.0770620494930662e-7, 6.0125912123632725e-16,
0868:       -1.4729737374018841e-7},
0869:     {-1.9994542198219728e-1, -1.5056113040026424e-2, 3.6470239469348489e-1,
0870:       -4.6435192311733545e-1, 2.6640934719197893e-1, 3.4038266027147191e-5,
```
- **EN**: Lines 841-870 mainly cover expressions/calls.
- **CN**: 第 841-870 行主要涉及表达式或调用。

### Lines 871-900 / 第 871-900 行
```cpp
0871:       -1.3784338709329624e-1, 1.276467178337056e-1, -5.6213828755200985e-2,
0872:       -1.753150885483011e-7, 1.9235592956768113e-2, -1.5088821281095315e-2,
0873:       5.7401854451350123e-3, 1.0622382710310225e-9, -1.5335082692563998e-3,
0874:       1.0819320643228214e-3, -3.7372510193945659e-4, -6.6170909729031985e-12,
0875:       8.4263617380909628e-5, -5.5150706827483479e-5, 1.7769536448348069e-5,
0876:       3.8827923210205533e-14, -3.53513697488768e-6, 2.1865832130045269e-6,
0877:       -6.6812849447625594e-7},
0878:     {7.2438608504029431e-1, -1.3918010932653375, 1.0654143352413968,
0879:       1.876173868950258e-4, -8.2705501176152696e-1, 8.9352433347828414e-1,
0880:       -4.4971003995291339e-1, -1.6107401567546652e-6, 1.9235590165271091e-1,
0881:       -1.6597702160042609e-1, 6.8882222681814333e-2, 1.3910091724608687e-8,
0882:       -2.146911561508663e-2, 1.6228980898865892e-2, -5.9796016172584256e-3,
0883:       -1.1287469112826745e-10, 1.5167451119784857e-3, -1.0478634293553899e-3,
0884:       3.5539072889126421e-4, 8.1704322111801517e-13, -7.7773013442452395e-5,
0885:       5.0291413897007722e-5, -1.6035083867000518e-5, 1.2469354315487605e-14,
0886:       3.1369106244517615e-6},
0887:     {1.6668949727276811, 1.165462765994632e-1, -3.3288393225018906,
0888:       4.4692325482864037, -2.6977693045875807, -2.600667859891061e-4,
0889:       1.5389017615694539, -1.4937962361134612, 6.8881964633233148e-1,
0890:       1.3077482004552385e-6, -2.5762963325596288e-1, 2.1097676102125449e-1,
0891:       -8.3714408359219882e-2, -7.7920428881354753e-9, 2.4267923064833599e-2,
0892:       -1.7813678334552311e-2, 6.3970330388900056e-3, 4.9430807090480523e-11,
0893:       -1.5554602758465635e-3, 1.0561196919903214e-3, -3.5277184460472902e-4,
0894:       9.3002334645022459e-14, 7.5285855026557172e-5, -4.8186515569156351e-5,
0895:       1.5227271505597605e-5},
0896:     {-6.6188298861372935, 1.3397985455142589e+1, -1.0789350606845146e+1,
0897:       -1.4352254537875018e-3, 9.2333694596189809, -1.0456552819547769e+1,
0898:       5.5105526029033471, 1.2024439690716742e-5, -2.5762961164755816,
0899:       2.3207442745387179, -1.0045728797216284, -1.0207833290021914e-7,
0900:       3.3975092171169466e-1, -2.6720517450757468e-1, 1.0235252851562706e-1,
```
- **EN**: Lines 871-900 mainly cover expressions/calls.
- **CN**: 第 871-900 行主要涉及表达式或调用。

### Lines 901-930 / 第 901-930 行
```cpp
0901:       8.4329730484871625e-10, -2.7998284958442595e-2, 2.0066274144976813e-2,
0902:       -7.0554368915086242e-3, 1.9402238183698188e-12, 1.6562888105449611e-3,
0903:       -1.1082898580743683e-3, 3.654545161310169e-4, -5.1290032026971794e-11,
0904:       -7.6340103696869031e-5},
0905:     {-1.7112706061976095e+1, -1.1208044642899116, 3.7131966511885444e+1,
0906:       -5.2298271025348962e+1, 3.3058589696624618e+1, 2.4791298976200222e-3,
0907:       -2.061089403411526e+1, 2.088672775145582e+1, -1.0045703956517752e+1,
0908:       -1.2238783449063012e-5, 4.0770134274221141, -3.473667358470195,
0909:       1.4329352617312006, 7.1359914411879712e-8, -4.4797257159115612e-1,
0910:       3.4112666080644461e-1, -1.2699786326594923e-1, -2.8953677269081528e-10,
0911:       3.3125776278259863e-2, -2.3274087021036101e-2, 8.0399993503648882e-3,
0912:       -1.177805216235265e-9, -1.8321624891071668e-3, 1.2108282933588665e-3,
0913:       -3.9479941246822517e-4},
0914:     {7.389033153567425e+1, -1.5680141270402273e+2, 1.322177542759164e+2,
0915:       1.3692876877324546e-2, -1.2366496885920151e+2, 1.4620689391062729e+2,
0916:       -8.0365587724865346e+1, -1.1259851148881298e-4, 4.0770132196179938e+1,
0917:       -3.8210340013273034e+1, 1.719522294277362e+1, 9.3519707955168356e-7,
0918:       -6.2716159907747034, 5.1168999071852637, -2.0319658112299095,
0919:       -4.9507215582761543e-9, 5.9626397294332597e-1, -4.4220765337238094e-1,
0920:       1.6079998700166273e-1, -2.4733786203223402e-8, -4.0307574759979762e-2,
0921:       2.7849050747097869e-2, -9.4751858992054221e-3, 6.419922235909132e-6,
0922:       2.1250180774699461e-3},
0923:     {2.1216837098382522e+2, 1.3107863022633868e+1, -4.9698285932871748e+2,
0924:       7.3121595266969204e+2, -4.8213821720890847e+2, -2.8817248692894889e-2,
0925:       3.2616720302947102e+2, -3.4389340280087117e+2, 1.7195193870816232e+2,
0926:       1.4038077378096158e-4, -7.52594195897599e+1, 6.651969984520934e+1,
0927:       -2.8447519748152462e+1, -7.613702615875391e-7, 9.5402237105304373,
0928:       -7.5175301113311376, 2.8943997568871961, -4.6612194999538201e-7,
0929:       -8.0615149598794088e-1, 5.8483006570631029e-1, -2.0845408972964956e-1,
0930:       1.4765818959305817e-4, 5.1000433863753019e-2, -3.3066252141883665e-2,
```
- **EN**: Lines 901-930 mainly cover expressions/calls.
- **CN**: 第 901-930 行主要涉及表达式或调用。

### Lines 931-960 / 第 931-960 行
```cpp
0931:       1.5109265210467774e-2},
0932:     {-9.8959643098322368e+2, 2.1925555360905233e+3, -1.9283586782723356e+3,
0933:       -1.5925738122215253e-1, 1.9569985945919857e+3, -2.4072514765081556e+3,
0934:       1.3756149959336496e+3, 1.2920735237496668e-3, -7.525941715948055e+2,
0935:       7.3171668742208716e+2, -3.4137023466220065e+2, -9.9857390260608043e-6,
0936:       1.3356313181291573e+2, -1.1276295161252794e+2, 4.6310396098204458e+1,
0937:       -7.9237387133614756e-6, -1.4510726927018646e+1, 1.1111771248100563e+1,
0938:       -4.1690817945270892, 3.1008219800117808e-3, 1.1220095449981468,
0939:       -7.6052379926149916e-1, 3.6262236505085254e-1, 2.216867741940747e-1,
0940:       4.8683443692930507e-1}};
0941: 
0942:   int k, n, sgn;
0943:   int maxpow = 0;
0944:   static scalar_t MACHEP = std::is_same_v<scalar_t, double> ?
0945:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
0946:   scalar_t lambda = x / a;
0947:   scalar_t sigma = (x - a) / a;
0948:   scalar_t eta, res, ck, ckterm, term, absterm;
0949:   scalar_t absoldterm = INFINITY;
0950:   scalar_t etapow[25] = {1};
0951:   scalar_t sum = 0;
0952:   scalar_t afac = 1;
0953: 
0954:   if (igam) {
0955:     sgn = -1;
0956:   }
0957:   else {
0958:     sgn = 1;
0959:   }
0960: 
```
- **EN**: Lines 931-960 mainly cover state/variable declarations, expressions/calls, control-flow checks.
- **CN**: 第 931-960 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。

### Lines 961-990 / 第 961-990 行
```cpp
0961:   if (lambda > 1) {
0962:     eta = std::sqrt(-2 * (std::log1p(sigma) - sigma));
0963:   }
0964:   else if (lambda < 1) {
0965:     eta = -std::sqrt(-2 * (std::log1p(sigma) - sigma));
0966:   }
0967:   else {
0968:     eta = 0;
0969:   }
0970:   res = 0.5 * std::erfc(sgn * eta * std::sqrt(a / 2));
0971: 
0972:   for (k = 0; k < 25; k++) {
0973:     ck = d[k][0];
0974:     for (n = 1; n < 25; n++) {
0975:       if (n > maxpow) {
0976:         etapow[n] = eta * etapow[n-1];
0977:         maxpow += 1;
0978:       }
0979:       ckterm = d[k][n]*etapow[n];
0980:       ck += ckterm;
0981:       if (std::fabs(ckterm) < MACHEP * std::fabs(ck)) {
0982:         break;
0983:       }
0984:     }
0985:     term = ck * afac;
0986:     absterm = std::fabs(term);
0987:     if (absterm > absoldterm) {
0988:       break;
0989:     }
0990:     sum += term;
```
- **EN**: Lines 961-990 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: sqrt, log1p, erfc, fabs.
- **CN**: 第 961-990 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：sqrt, log1p, erfc, fabs。

### Lines 991-1020 / 第 991-1020 行
```cpp
0991:     if (absterm < MACHEP * std::fabs(sum)) {
0992:       break;
0993:     }
0994:     absoldterm = absterm;
0995:     afac /= a;
0996:   }
0997:   res += sgn * std::exp(-0.5 * a * eta * eta) * sum / std::sqrt(2 * c10::pi<float> * a);
0998: 
0999:   return res;
1000: }
1001: 
1002: template <typename scalar_t>
1003: static scalar_t _igamc_helper_continued_fraction(scalar_t a, scalar_t x) {
1004:   // Compute igamc using DLMF 8.9.2. [igam1]
1005:   int i;
1006:   scalar_t ans, ax, c, yc, r, t, y, z;
1007:   scalar_t pk, pkm1, pkm2, qk, qkm1, qkm2;
1008:   int MAXITER = 2000;
1009:   static scalar_t MACHEP = std::is_same_v<scalar_t, double> ?
1010:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
1011:   static scalar_t BIG = std::is_same_v<scalar_t,double> ?
1012:     4.503599627370496e15 : 16777216.;
1013:   static scalar_t BIGINV = std::is_same_v<scalar_t,double> ?
1014:     2.22044604925031308085e-16 : 5.9604644775390625E-8;
1015: 
1016:   ax = _igam_helper_fac(a, x);
1017:   if (ax == 0.0) {
1018:     return 0.0;
1019:   }
1020: 
```
- **EN**: Lines 991-1020 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: fabs, exp, sqrt, _igamc_helper_continued_fraction.
- **CN**: 第 991-1020 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：fabs, exp, sqrt, _igamc_helper_continued_fraction。

### Lines 1021-1050 / 第 1021-1050 行
```cpp
1021:   /* continued fraction */
1022:   y = 1.0 - a;
1023:   z = x + y + 1.0;
1024:   c = 0.0;
1025:   pkm2 = 1.0;
1026:   qkm2 = x;
1027:   pkm1 = x + 1.0;
1028:   qkm1 = z * x;
1029:   ans = pkm1 / qkm1;
1030: 
1031:   for (i = 0; i < MAXITER; i++) {
1032:     c += 1.0;
1033:     y += 1.0;
1034:     z += 2.0;
1035:     yc = y * c;
1036:     pk = pkm1 * z - pkm2 * yc;
1037:     qk = qkm1 * z - qkm2 * yc;
1038:     if (qk != 0) {
1039:       r = pk / qk;
1040:       t = std::fabs((ans - r) / r);
1041:       ans = r;
1042:     }
1043:     else {
1044:       t = 1.0;
1045:     }
1046:     pkm2 = pkm1;
1047:     pkm1 = pk;
1048:     qkm2 = qkm1;
1049:     qkm1 = qk;
1050:     if (std::fabs(pk) > BIG) {
```
- **EN**: Lines 1021-1050 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: fabs.
- **CN**: 第 1021-1050 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：fabs。

### Lines 1051-1080 / 第 1051-1080 行
```cpp
1051:       pkm2 *= BIGINV;
1052:       pkm1 *= BIGINV;
1053:       qkm2 *= BIGINV;
1054:       qkm1 *= BIGINV;
1055:     }
1056:     if (t <= MACHEP) {
1057:       break;
1058:     }
1059:   }
1060:   return ans * ax;
1061: }
1062: 
1063: template <typename scalar_t>
1064: inline scalar_t calc_igammac(scalar_t a, scalar_t x) {
1065:   /* the calculation of the regularized upper incomplete gamma function
1066:    * is done differently based on the values of a and x:
1067:    * - if x and/or a is at the boundary of defined region, then assign the
1068:    *   result at the boundary
1069:    * - if a is large and a ~ x, then using Uniform Asymptotic Expansions for
1070:    *   Large Parameter (see DLMF 8.12.4 [igam1])
1071:    * - if x > 1.1 and x < a, using the subtraction from the regularized lower
1072:    *   incomplete gamma
1073:    * - otherwise, calculate the series from [igam2] eq (5)
1074:    */
1075:   scalar_t absxma_a;
1076: 
1077:   static scalar_t SMALL = 20.0;
1078:   static scalar_t LARGE = 200.0;
1079:   static scalar_t SMALLRATIO = 0.3;
1080:   static scalar_t LARGERATIO = 4.5;
```
- **EN**: Lines 1051-1080 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: calc_igammac, Parameter, eq.
- **CN**: 第 1051-1080 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：calc_igammac, Parameter, eq。

### Lines 1081-1110 / 第 1081-1110 行
```cpp
1081: 
1082:   // note that in SciPy, a and x are non-negative, with exclusive 0s (i.e.,
1083:   // at most 1 of them can be 0), where igammac(0, x) = 0.0 iff x > 0.
1084:   if ((x < 0) || (a < 0)) {
1085:     // out of defined-region of the function
1086:     return std::numeric_limits<scalar_t>::quiet_NaN();
1087:   }
1088:   else if (a == 0) {
1089:     if (x > 0) {
1090:       return 0.0;
1091:     }
1092:     else {
1093:       return std::numeric_limits<scalar_t>::quiet_NaN();
1094:     }
1095:   }
1096:   else if (x == 0) {
1097:     return 1.0;
1098:   }
1099:   else if (std::isinf(a)) {
1100:     if (std::isinf(x)) {
1101:       return std::numeric_limits<scalar_t>::quiet_NaN();
1102:     }
1103:     return 1.0;
1104:   }
1105:   else if (std::isinf(x)) {
1106:     return 0.0;
1107:   }
1108: 
1109:   absxma_a = std::fabs(x - a) / a;
1110:   if ((a > SMALL) && (a < LARGE) && (absxma_a < SMALLRATIO)) {
```
- **EN**: Lines 1081-1110 mainly cover control-flow checks, expressions/calls, return paths. Notable symbols: s, igammac, quiet_NaN, isinf.
- **CN**: 第 1081-1110 行主要涉及控制流逻辑、表达式或调用、返回路径。 值得关注的符号包括：s, igammac, quiet_NaN, isinf。

### Lines 1111-1140 / 第 1111-1140 行
```cpp
1111:      return _igam_helper_asymptotic_series(a, x, 0);
1112:   }
1113:   else if ((a > LARGE) && (absxma_a < LARGERATIO / std::sqrt(a))) {
1114:      return _igam_helper_asymptotic_series(a, x, 0);
1115:   }
1116: 
1117:   if (x > 1.1) {
1118:     if (x < a) {
1119:       return 1.0 - _igam_helper_series(a, x);
1120:     }
1121:     else {
1122:       return _igamc_helper_continued_fraction(a, x);
1123:     }
1124:   }
1125:   else if (x <= 0.5) {
1126:     if (-0.4 / std::log(x) < a) {
1127:       return 1.0 - _igam_helper_series(a, x);
1128:     }
1129:     else {
1130:       return _igamc_helper_series(a, x);
1131:     }
1132:   }
1133:   else {
1134:     if (x * 1.1 < a) {
1135:       return 1.0 - _igam_helper_series(a, x);
1136:     }
1137:     else {
1138:       return _igamc_helper_series(a, x);
1139:     }
1140:   }
```
- **EN**: Lines 1111-1140 mainly cover expressions/calls, control-flow checks, return paths. Notable symbols: _igam_helper_asymptotic_series, sqrt, _igam_helper_series, _igamc_helper_continued_fraction.
- **CN**: 第 1111-1140 行主要涉及表达式或调用、控制流逻辑、返回路径。 值得关注的符号包括：_igam_helper_asymptotic_series, sqrt, _igam_helper_series, _igamc_helper_continued_fraction。

### Lines 1141-1170 / 第 1141-1170 行
```cpp
1141: }
1142: 
1143: template <typename scalar_t>
1144: scalar_t calc_igamma(scalar_t a, scalar_t x) {
1145:   /* the calculation of the regularized lower incomplete gamma function
1146:    * is done differently based on the values of a and x:
1147:    * - if x and/or a is at the boundary of defined region, then assign the
1148:    *   result at the boundary
1149:    * - if a is large and a ~ x, then using Uniform Asymptotic Expansions for
1150:    *   Large Parameter (see DLMF 8.12.3 [igam1])
1151:    * - if x > 1 and x > a, using the subtraction from the regularized upper
1152:    *   incomplete gamma
1153:    * - otherwise, calculate the series from [igam2] eq (4)
1154:    */
1155:   scalar_t absxma_a;
1156:   static scalar_t SMALL = 20.0;
1157:   static scalar_t LARGE = 200.0;
1158:   static scalar_t SMALLRATIO = 0.3;
1159:   static scalar_t LARGERATIO = 4.5;
1160: 
1161:   // boundary values following SciPy
1162:   // note that in SciPy, a and x are non-negative, with exclusive 0s (i.e.,
1163:   // at most 1 of them can be 0), where igamma(0, x) = 1.0 iff x > 0.
1164:   if ((x < 0) || (a < 0)) {
1165:     // out of defined-region of the function
1166:     return std::numeric_limits<scalar_t>::quiet_NaN();
1167:   }
1168:   else if (a == 0) {
1169:     if (x > 0) {
1170:       return 1.0;
```
- **EN**: Lines 1141-1170 mainly cover comments/documentation, state/variable declarations, control-flow checks. Notable symbols: calc_igamma, Parameter, eq, s.
- **CN**: 第 1141-1170 行主要涉及注释或说明、变量/别名声明、控制流逻辑。 值得关注的符号包括：calc_igamma, Parameter, eq, s。

### Lines 1171-1200 / 第 1171-1200 行
```cpp
1171:     }
1172:     else {
1173:       return std::numeric_limits<scalar_t>::quiet_NaN();
1174:     }
1175:   }
1176:   else if (x == 0) {
1177:     return 0.0; // zero integration limit
1178:   }
1179:   else if (std::isinf(a)) {
1180:     if (std::isinf(x)) {
1181:       return std::numeric_limits<scalar_t>::quiet_NaN();
1182:     }
1183:     return 0.0;
1184:   }
1185:   else if (std::isinf(x)) {
1186:     return 1.0;
1187:   }
1188: 
1189:   /* Asymptotic regime where a ~ x. See [igam2] */
1190:   absxma_a = std::fabs(x - a) / a;
1191:   if ((a > SMALL) && (a < LARGE) && (absxma_a < SMALLRATIO)) {
1192:     return _igam_helper_asymptotic_series(a, x, 1);
1193:   }
1194:   else if ((a > LARGE) && (absxma_a < LARGERATIO / std::sqrt(a))) {
1195:     return _igam_helper_asymptotic_series(a, x, 1);
1196:   }
1197: 
1198:   if ((x > 1.0) && (x > a)) {
1199:     return 1.0 - calc_igammac(a, x);
1200:   }
```
- **EN**: Lines 1171-1200 mainly cover expressions/calls, control-flow checks, return paths. Notable symbols: quiet_NaN, isinf, fabs, _igam_helper_asymptotic_series.
- **CN**: 第 1171-1200 行主要涉及表达式或调用、控制流逻辑、返回路径。 值得关注的符号包括：quiet_NaN, isinf, fabs, _igam_helper_asymptotic_series。

### Lines 1201-1230 / 第 1201-1230 行
```cpp
1201: 
1202:   return _igam_helper_series(a, x);
1203: }
1204: 
1205: template <>
1206: [[maybe_unused]] inline c10::BFloat16 calc_igamma<c10::BFloat16>(
1207:     c10::BFloat16 a,
1208:     c10::BFloat16 x) {
1209:   return calc_igamma<float>(float(a), float(x));
1210: }
1211: 
1212: template <>
1213: [[maybe_unused]] inline c10::Half calc_igamma<c10::Half>(
1214:     c10::Half a,
1215:     c10::Half x) {
1216:   return calc_igamma<float>(float(a), float(x));
1217: }
1218: 
1219: template <>
1220: [[maybe_unused]] inline c10::BFloat16 calc_igammac<c10::BFloat16>(
1221:     c10::BFloat16 a,
1222:     c10::BFloat16 x) {
1223:   return calc_igammac<float>(float(a), float(x));
1224: }
1225: 
1226: template <>
1227: [[maybe_unused]] inline c10::Half calc_igammac<c10::Half>(
1228:     c10::Half a,
1229:     c10::Half x) {
1230:   return calc_igammac<float>(float(a), float(x));
```
- **EN**: Lines 1201-1230 mainly cover expressions/calls, return paths, template setup. Notable symbols: _igam_helper_series, float.
- **CN**: 第 1201-1230 行主要涉及表达式或调用、返回路径、模板声明。 值得关注的符号包括：_igam_helper_series, float。

### Lines 1231-1260 / 第 1231-1260 行
```cpp
1231: }
1232: 
1233: inline c10::BFloat16 calc_erfinv(c10::BFloat16 a) { return calc_erfinv(float(a)); }
1234: 
1235: template <typename T>
1236: inline T abs_impl(T v) {
1237:   return std::abs(v);
1238: }
1239: 
1240: template <>
1241: [[maybe_unused]] inline uint8_t abs_impl(uint8_t v) {
1242:   return v;
1243: }
1244: 
1245: template <typename T>
1246: inline typename std::enable_if_t<std::is_integral_v<T>, T>
1247: calc_gcd(T a, T b) {
1248:   a = abs_impl(a);
1249:   b = abs_impl(b);
1250:   while (a != 0) {
1251:     T c = a;
1252:     a = b % a;
1253:     b = c;
1254:   }
1255:   return b;
1256: }
1257: 
1258: template <typename T>
1259: C10_HOST_DEVICE T exp2_impl(T x) {
1260:   return std::exp2(x);
```
- **EN**: Lines 1231-1260 mainly cover state/variable declarations, expressions/calls, template setup. Notable symbols: calc_erfinv, float, abs_impl, abs.
- **CN**: 第 1231-1260 行主要涉及变量/别名声明、表达式或调用、模板声明。 值得关注的符号包括：calc_erfinv, float, abs_impl, abs。

### Lines 1261-1290 / 第 1261-1290 行
```cpp
1261: }
1262: 
1263: template <typename T>
1264: C10_HOST_DEVICE c10::complex<T> exp2_impl(c10::complex<T> x) {
1265:   // There is no std::exp2 overload for complex, so instead
1266:   // use the identity 2^x = e^(ln(2) * x)
1267:   constexpr auto ln2 = c10::ln_2<T>;
1268:   return std::exp(ln2 * x);
1269: }
1270: 
1271: /*
1272:  * This function is derived from the implementation of the chbevl function in the Cephes Math Library.
1273:  * See note [3-Clause BSD License for the Cephes Math Library].
1274:  *
1275:  * Evaluates the series
1276:  *
1277:  *       len-1
1278:  *         - '
1279:  *  y  =   >   array[i] T (x/2)
1280:  *         -             i
1281:  *        i=0
1282:  *
1283:  * of Chebyshev polynomials Ti at argument x/2.
1284:  *
1285:  * Coefficients are stored in reverse order, i.e. the zero order term is last in the array.  Note len is the number of
1286:  * coefficients, not the order.
1287:  *
1288:  * If coefficients are for the interval a to b, x must have been transformed to x -> 2(2x - b - a)/(b-a) before
1289:  * entering the routine.  This maps x from (a, b) to (-1, 1), over which the Chebyshev polynomials are defined.
1290:  *
```
- **EN**: Lines 1261-1290 mainly cover comments/documentation, expressions/calls, template setup. Notable symbols: exp2_impl, ln, exp, T.
- **CN**: 第 1261-1290 行主要涉及注释或说明、表达式或调用、模板声明。 值得关注的符号包括：exp2_impl, ln, exp, T。

### Lines 1291-1320 / 第 1291-1320 行
```cpp
1291:  * If the coefficients are for the inverted interval, in which (a, b) is mapped to (1/b, 1/a), the transformation
1292:  * required is x -> 2(2ab/x - b - a)/(b-a).  If b is infinity, this becomes x -> 4a/x - 1.
1293:  */
1294: template <typename T>
1295: inline typename std::enable_if_t<std::is_floating_point_v<T>, T>
1296: chbevl(const T x, const T array[], size_t len) {
1297:   T b0, b1, b2 = static_cast<T>(0.0);
1298: 
1299:   b0 = array[0];
1300:   b1 = static_cast<T>(0.0);
1301: 
1302:   for (size_t i = 1; i < len; ++i) {
1303:     b2 = b1;
1304:     b1 = b0;
1305:     b0 = x * b1 - b2 + array[i];
1306:   }
1307: 
1308:   return (static_cast<T>(0.5) * (b0 - b2));
1309: }
1310: 
1311: /*
1312:  * This function is derived from the implementation of the i0 function in the Cephes Math Library.
1313:  * See note [3-Clause BSD License for the Cephes Math Library].
1314:  *
1315:  * Computes an approximation of the zeroth order modified Bessel function of the first kind.
1316:  * The approximation is actually two (sub)approximations, both using a Chebyshev polynomial expansion.
1317:  * One approximates the function over [0, 8], and the other over (8, infinity). This function takes the absolute value
1318:  * of all inputs to convert them into the domain of the approximation.
1319:  */
1320: template <typename T>
```
- **EN**: Lines 1291-1320 mainly cover comments/documentation, state/variable declarations, template setup. Notable symbols: which, to, chbevl, two.
- **CN**: 第 1291-1320 行主要涉及注释或说明、变量/别名声明、模板声明。 值得关注的符号包括：which, to, chbevl, two。

### Lines 1321-1350 / 第 1321-1350 行
```cpp
1321: inline std::tuple<const T*, size_t> chebyshev_coefficients_i0e_A() {
1322:   /* Chebyshev coefficients for exp(-x) I0(x)
1323:    * in the interval [0,8].
1324:    *
1325:    * lim(x->0){ exp(-x) I0(x) } = 1.
1326:    */
1327:   static const T coeff[] = {
1328:       -4.41534164647933937950E-18, 3.33079451882223809783E-17,
1329:       -2.43127984654795469359E-16, 1.71539128555513303061E-15,
1330:       -1.16853328779934516808E-14, 7.67618549860493561688E-14,
1331:       -4.85644678311192946090E-13, 2.95505266312963983461E-12,
1332:       -1.72682629144155570723E-11, 9.67580903537323691224E-11,
1333:       -5.18979560163526290666E-10, 2.65982372468238665035E-9,
1334:       -1.30002500998624804212E-8,  6.04699502254191894932E-8,
1335:       -2.67079385394061173391E-7,  1.11738753912010371815E-6,
1336:       -4.41673835845875056359E-6,  1.64484480707288970893E-5,
1337:       -5.75419501008210370398E-5,  1.88502885095841655729E-4,
1338:       -5.76375574538582365885E-4,  1.63947561694133579842E-3,
1339:       -4.32430999505057594430E-3,  1.05464603945949983183E-2,
1340:       -2.37374148058994688156E-2,  4.93052842396707084878E-2,
1341:       -9.49010970480476444210E-2,  1.71620901522208775349E-1,
1342:       -3.04682672343198398683E-1,  6.76795274409476084995E-1};
1343:   return std::make_tuple(coeff, 30);
1344: }
1345: 
1346: template <typename T>
1347: inline std::tuple<const T*, size_t> chebyshev_coefficients_i0e_B() {
1348:   /* Chebyshev coefficients for exp(-x) sqrt(x) I0(x)
1349:    * in the inverted interval [8,infinity].
1350:    *
```
- **EN**: Lines 1321-1350 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: chebyshev_coefficients_i0e_A, exp, I0, lim.
- **CN**: 第 1321-1350 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：chebyshev_coefficients_i0e_A, exp, I0, lim。

### Lines 1351-1380 / 第 1351-1380 行
```cpp
1351:    * lim(x->inf){ exp(-x) sqrt(x) I0(x) } = 1/sqrt(2pi).
1352:    */
1353:   static const T coeff[] = {
1354:       -7.23318048787475395456E-18, -4.83050448594418207126E-18,
1355:       4.46562142029675999901E-17,  3.46122286769746109310E-17,
1356:       -2.82762398051658348494E-16, -3.42548561967721913462E-16,
1357:       1.77256013305652638360E-15,  3.81168066935262242075E-15,
1358:       -9.55484669882830764870E-15, -4.15056934728722208663E-14,
1359:       1.54008621752140982691E-14,  3.85277838274214270114E-13,
1360:       7.18012445138366623367E-13,  -1.79417853150680611778E-12,
1361:       -1.32158118404477131188E-11, -3.14991652796324136454E-11,
1362:       1.18891471078464383424E-11,  4.94060238822496958910E-10,
1363:       3.39623202570838634515E-9,   2.26666899049817806459E-8,
1364:       2.04891858946906374183E-7,   2.89137052083475648297E-6,
1365:       6.88975834691682398426E-5,   3.36911647825569408990E-3,
1366:       8.04490411014108831608E-1};
1367: 
1368:   return std::make_tuple(coeff, 25);
1369: }
1370: 
1371: template <typename T>
1372: inline typename std::enable_if_t<std::is_same_v<double, T>, std::tuple<const T*, size_t>>
1373: chebyshev_coefficients_i1e_A() {
1374:   /* Chebyshev coefficients for exp(-x) I1(x)
1375:    * in the interval [0,8].
1376:    *
1377:    * lim(x->0){ exp(-x) I1(x) / x } = 1/2.
1378:    */
1379:   static const T coeff[] = {
1380:       2.77791411276104639959E-18, -2.11142121435816608115E-17,
```
- **EN**: Lines 1351-1380 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: lim, exp, sqrt, I0.
- **CN**: 第 1351-1380 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：lim, exp, sqrt, I0。

### Lines 1381-1410 / 第 1381-1410 行
```cpp
1381:       1.55363195773620046921E-16, -1.10559694773538630805E-15,
1382:       7.60068429473540693410E-15, -5.04218550472791168711E-14,
1383:       3.22379336594557470981E-13, -1.98397439776494371520E-12,
1384:       1.17361862988909016308E-11, -6.66348972350202774223E-11,
1385:       3.62559028155211703701E-10, -1.88724975172282928790E-9,
1386:       9.38153738649577178388E-9,  -4.44505912879632808065E-8,
1387:       2.00329475355213526229E-7,  -8.56872026469545474066E-7,
1388:       3.47025130813767847674E-6,  -1.32731636560394358279E-5,
1389:       4.78156510755005422638E-5,  -1.61760815825896745588E-4,
1390:       5.12285956168575772895E-4,  -1.51357245063125314899E-3,
1391:       4.15642294431288815669E-3,  -1.05640848946261981558E-2,
1392:       2.47264490306265168283E-2,  -5.29459812080949914269E-2,
1393:       1.02643658689847095384E-1,  -1.76416518357834055153E-1,
1394:       2.52587186443633654823E-1};
1395:   return std::make_tuple(coeff, 29);
1396: }
1397: 
1398: template <typename T>
1399: inline typename std::enable_if_t<std::is_same_v<float, T>, std::tuple<const T*, size_t>>
1400: chebyshev_coefficients_i1e_A() {
1401:   /* Chebyshev coefficients for exp(-x) I1(x)
1402:    * in the interval [0,8].
1403:    *
1404:    * lim(x->0){ exp(-x) I1(x) / x } = 1/2.
1405:    */
1406:   static const T coeff[] = {
1407:       9.38153738649577178388E-9f,
1408:       -4.44505912879632808065E-8f,
1409:       2.00329475355213526229E-7f,
1410:       -8.56872026469545474066E-7f,
```
- **EN**: Lines 1381-1410 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: make_tuple, chebyshev_coefficients_i1e_A, exp, I1.
- **CN**: 第 1381-1410 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：make_tuple, chebyshev_coefficients_i1e_A, exp, I1。

### Lines 1411-1440 / 第 1411-1440 行
```cpp
1411:       3.47025130813767847674E-6f,
1412:       -1.32731636560394358279E-5f,
1413:       4.78156510755005422638E-5f,
1414:       -1.61760815825896745588E-4f,
1415:       5.12285956168575772895E-4f,
1416:       -1.51357245063125314899E-3f,
1417:       4.15642294431288815669E-3f,
1418:       -1.05640848946261981558E-2f,
1419:       2.47264490306265168283E-2f,
1420:       -5.29459812080949914269E-2f,
1421:       1.02643658689847095384E-1f,
1422:       -1.76416518357834055153E-1f,
1423:       2.52587186443633654823E-1f};
1424:   return std::make_tuple(coeff, 17);
1425: }
1426: 
1427: template <typename T>
1428: inline typename std::enable_if_t<std::is_same_v<double, T>, std::tuple<const T*, size_t>>
1429: chebyshev_coefficients_i1e_B() {
1430:   /* Chebyshev coefficients for exp(-x) sqrt(x) I1(x)
1431:    * in the inverted interval [8,infinity].
1432:    *
1433:    * lim(x->inf){ exp(-x) sqrt(x) I1(x) } = 1/sqrt(2pi).
1434:    */
1435:   static const T coeff[] = {
1436:       7.51729631084210481353E-18,  4.41434832307170791151E-18,
1437:       -4.65030536848935832153E-17, -3.20952592199342395980E-17,
1438:       2.96262899764595013876E-16,  3.30820231092092828324E-16,
1439:       -1.88035477551078244854E-15, -3.81440307243700780478E-15,
1440:       1.04202769841288027642E-14,  4.27244001671195135429E-14,
```
- **EN**: Lines 1411-1440 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: make_tuple, chebyshev_coefficients_i1e_B, exp, sqrt.
- **CN**: 第 1411-1440 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：make_tuple, chebyshev_coefficients_i1e_B, exp, sqrt。

### Lines 1441-1470 / 第 1441-1470 行
```cpp
1441:       -2.10154184277266431302E-14, -4.08355111109219731823E-13,
1442:       -7.19855177624590851209E-13, 2.03562854414708950722E-12,
1443:       1.41258074366137813316E-11,  3.25260358301548823856E-11,
1444:       -1.89749581235054123450E-11, -5.58974346219658380687E-10,
1445:       -3.83538038596423702205E-9,  -2.63146884688951950684E-8,
1446:       -2.51223623787020892529E-7,  -3.88256480887769039346E-6,
1447:       -1.10588938762623716291E-4,  -9.76109749136146840777E-3,
1448:       7.78576235018280120474E-1};
1449: 
1450:   return std::make_tuple(coeff, 25);
1451: }
1452: 
1453: template <typename T>
1454: inline typename std::enable_if_t<std::is_same_v<float, T>, std::tuple<const T*, size_t>>
1455: chebyshev_coefficients_i1e_B() {
1456:   /* Chebyshev coefficients for exp(-x) sqrt(x) I1(x)
1457:    * in the inverted interval [8,infinity].
1458:    *
1459:    * lim(x->inf){ exp(-x) sqrt(x) I1(x) } = 1/sqrt(2pi).
1460:    */
1461:   static const T coeff[] = {
1462:       -3.83538038596423702205E-9f,
1463:       -2.63146884688951950684E-8f,
1464:       -2.51223623787020892529E-7f,
1465:       -3.88256480887769039346E-6f,
1466:       -1.10588938762623716291E-4f,
1467:       -9.76109749136146840777E-3f,
1468:       7.78576235018280120474E-1f};
1469: 
1470:   return std::make_tuple(coeff, 7);
```
- **EN**: Lines 1441-1470 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: make_tuple, chebyshev_coefficients_i1e_B, exp, sqrt.
- **CN**: 第 1441-1470 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：make_tuple, chebyshev_coefficients_i1e_B, exp, sqrt。

### Lines 1471-1500 / 第 1471-1500 行
```cpp
1471: }
1472: 
1473: template <typename T>
1474: inline typename std::enable_if_t<std::is_floating_point_v<T>, T>
1475: calc_i0(T _x) {
1476:   T x = std::abs(_x);
1477: 
1478:   if (x <= T{8.0}) {
1479:     auto [A, len] = chebyshev_coefficients_i0e_A<T>();
1480:     T y = (x / T{2.0}) - T{2.0};
1481:     return static_cast<T>(std::exp(x) * chbevl(y, A, len));
1482:   }
1483:   auto [B, len] = chebyshev_coefficients_i0e_B<T>();
1484:   return std::exp(x) * chbevl(T{32.0} / x - T{2.0}, B, len) / std::sqrt(x);
1485: }
1486: 
1487: // Upcast bfloat16/half input to float for numerical accuracy purposes
1488: inline c10::BFloat16 calc_i0(c10::BFloat16 a) { return calc_i0(static_cast<float>(a)); }
1489: inline c10::Half calc_i0(c10::Half a) { return calc_i0(static_cast<float>(a)); }
1490: 
1491: /*
1492:  * This function is derived from the implementation of the i1 function in the Cephes Math Library.
1493:  * See note [3-Clause BSD License for the Cephes Math Library].
1494:  *
1495:  * Computes an approximation of the first order modified Bessel function of the first kind.
1496:  * The approximation is actually two (sub)approximations, both using a Chebyshev polynomial expansion.
1497:  * One approximates the function over [0, 8], and the other over (8, infinity). This function takes the absolute value
1498:  * of all inputs to convert them into the domain of the approximation.
1499:  */
1500: template <typename T>
```
- **EN**: Lines 1471-1500 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: calc_i0, abs, exp, chbevl.
- **CN**: 第 1471-1500 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：calc_i0, abs, exp, chbevl。

### Lines 1501-1530 / 第 1501-1530 行
```cpp
1501: inline typename std::enable_if_t<std::is_floating_point_v<T>, T>
1502: calc_i1(T _x) {
1503:   T x = std::abs(_x);
1504: 
1505:   if (x <= T{8.0}) {
1506:     auto [A, len] = chebyshev_coefficients_i1e_A<T>();
1507:     T y = (x / T{2.0}) - T{2.0};
1508:     const T out = std::exp(x) * x * chbevl(y, A, len);
1509:     return (_x < T{0.0}) ? -out : out;
1510:   }
1511:   auto [B, len] = chebyshev_coefficients_i1e_B<T>();
1512:   const T out = (std::exp(x) * chbevl(T{32.0} / x - T{2.0}, B, len)) / std::sqrt(x);
1513:   return (_x < T{0.0}) ? -out : out;
1514: }
1515: 
1516: // Upcast bfloat16/half input to float for numerical accuracy purposes
1517: inline c10::BFloat16 calc_i1(c10::BFloat16 a) { return calc_i1(static_cast<float>(a)); }
1518: inline c10::Half calc_i1(c10::Half a) { return calc_i1(static_cast<float>(a)); }
1519: 
1520: 
1521: /*
1522:  * This function is derived from the implementation of the i1e function in the Cephes Math Library.
1523:  * See note [3-Clause BSD License for the Cephes Math Library].
1524:  *
1525:  * Computes an approximation of the exponentially scaled first order modified Bessel function of the first kind.
1526:  * The approximation is actually two (sub)approximations, both using a Chebyshev polynomial expansion.
1527:  * One approximates the function over [0, 8], and the other over (8, infinity). This function takes the absolute value
1528:  * of all inputs to convert them into the domain of the approximation.
1529:  */
1530: template <typename T>
```
- **EN**: Lines 1501-1530 mainly cover comments/documentation, state/variable declarations, return paths. Notable symbols: calc_i1, abs, exp, chbevl.
- **CN**: 第 1501-1530 行主要涉及注释或说明、变量/别名声明、返回路径。 值得关注的符号包括：calc_i1, abs, exp, chbevl。

### Lines 1531-1560 / 第 1531-1560 行
```cpp
1531: inline typename std::enable_if_t<std::is_floating_point_v<T>, T>
1532: calc_i1e(T _x) {
1533:   T x = std::abs(_x);
1534: 
1535:   if (x <= T{8.0}) {
1536:     auto [A, len] = chebyshev_coefficients_i1e_A<T>();
1537:     T y = (x / T{2.0}) - T{2.0};
1538:     const T out = chbevl(y, A, len) * x;
1539:     return (_x < T{0.0}) ? -out : out;
1540:   }
1541:   auto [B, len] = chebyshev_coefficients_i1e_B<T>();
1542:   const auto out = chbevl(T{32.0} / x - T{2.0}, B, len) / std::sqrt(x);
1543:   return (_x < T{0.0}) ? -out : out;
1544: }
1545: 
1546: // Upcast bfloat16/half input to float for numerical accuracy purposes
1547: inline c10::BFloat16 calc_i1e(c10::BFloat16 a) { return calc_i1e(static_cast<float>(a)); }
1548: inline c10::Half calc_i1e(c10::Half a) { return calc_i1e(static_cast<float>(a)); }
1549: 
1550: 
1551: /*
1552:  * This function is derived from the implementation of the i1e function in the Cephes Math Library.
1553:  * See note [3-Clause BSD License for the Cephes Math Library].
1554:  *
1555:  * Computes the argument, x, for which the area under the Gaussian probability density function
1556:  * (integrated from minus infinity to x) is equal to y.
1557:  */
1558: template <typename T>
1559: inline C10_HOST_DEVICE T calc_ndtri(T y0) {
1560: 
```
- **EN**: Lines 1531-1560 mainly cover state/variable declarations, comments/documentation, return paths. Notable symbols: calc_i1e, abs, chbevl, sqrt.
- **CN**: 第 1531-1560 行主要涉及变量/别名声明、注释或说明、返回路径。 值得关注的符号包括：calc_i1e, abs, chbevl, sqrt。

### Lines 1561-1590 / 第 1561-1590 行
```cpp
1561:   /* sqrt(2pi) */
1562:   constexpr T s2pi = 2.50662827463100050242E0;
1563:   constexpr T one = 1;
1564:   constexpr T zero = 0;
1565: 
1566:   /* approximation for 0 <= |y - 0.5| <= 3/8 */
1567:   static const T P0[5] = {
1568:       -5.99633501014107895267E1,
1569:       9.80010754185999661536E1,
1570:       -5.66762857469070293439E1,
1571:       1.39312609387279679503E1,
1572:       -1.23916583867381258016E0,
1573:   };
1574: 
1575:   static const T Q0[9] = {
1576:       1.00000000000000000000E0,
1577:       1.95448858338141759834E0,
1578:       4.67627912898881538453E0,
1579:       8.63602421390890590575E1,
1580:       -2.25462687854119370527E2,
1581:       2.00260212380060660359E2,
1582:       -8.20372256168333339912E1,
1583:       1.59056225126211695515E1,
1584:       -1.18331621121330003142E0,
1585:   };
1586: 
1587:   /* Approximation for interval z = sqrt(-2 log y ) between 2 and 8
1588:   * i.e., y between exp(-2) = .135 and exp(-32) = 1.27e-14.
1589:   */
1590:   static const T P1[9] = {
```
- **EN**: Lines 1561-1590 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: sqrt, exp.
- **CN**: 第 1561-1590 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：sqrt, exp。

### Lines 1591-1620 / 第 1591-1620 行
```cpp
1591:       4.05544892305962419923E0,
1592:       3.15251094599893866154E1,
1593:       5.71628192246421288162E1,
1594:       4.40805073893200834700E1,
1595:       1.46849561928858024014E1,
1596:       2.18663306850790267539E0,
1597:       -1.40256079171354495875E-1,
1598:       -3.50424626827848203418E-2,
1599:       -8.57456785154685413611E-4,
1600:   };
1601: 
1602:   static const T Q1[9] = {
1603:       1.00000000000000000000E0,
1604:       1.57799883256466749731E1,
1605:       4.53907635128879210584E1,
1606:       4.13172038254672030440E1,
1607:       1.50425385692907503408E1,
1608:       2.50464946208309415979E0,
1609:       -1.42182922854787788574E-1,
1610:       -3.80806407691578277194E-2,
1611:       -9.33259480895457427372E-4,
1612:   };
1613: 
1614:   /* Approximation for interval z = sqrt(-2 log y ) between 8 and 64
1615:   * i.e., y between exp(-32) = 1.27e-14 and exp(-2048) = 3.67e-890.
1616:   */
1617: 
1618:   static const T P2[9] = {
1619:       3.23774891776946035970E0,
1620:       6.91522889068984211695E0,
```
- **EN**: Lines 1591-1620 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: sqrt, exp.
- **CN**: 第 1591-1620 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：sqrt, exp。

### Lines 1621-1650 / 第 1621-1650 行
```cpp
1621:       3.93881025292474443415E0,
1622:       1.33303460815807542389E0,
1623:       2.01485389549179081538E-1,
1624:       1.23716634817820021358E-2,
1625:       3.01581553508235416007E-4,
1626:       2.65806974686737550832E-6,
1627:       6.23974539184983293730E-9,
1628:   };
1629: 
1630:   static const T Q2[9] = {
1631:       1.00000000000000000000E0,
1632:       6.02427039364742014255E0,
1633:       3.67983563856160859403E0,
1634:       1.37702099489081330271E0,
1635:       2.16236993594496635890E-1,
1636:       1.34204006088543189037E-2,
1637:       3.28014464682127739104E-4,
1638:       2.89247864745380683936E-6,
1639:       6.79019408009981274425E-9,
1640:   };
1641: 
1642:   if (y0 == zero) {
1643:     return -std::numeric_limits<T>::infinity();
1644:   }
1645:   if (y0 == one) {
1646:     return std::numeric_limits<T>::infinity();
1647:   }
1648:   if (y0 < zero || y0 > one) {
1649:     return std::numeric_limits<T>::quiet_NaN();
1650:   }
```
- **EN**: Lines 1621-1650 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: infinity, quiet_NaN.
- **CN**: 第 1621-1650 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：infinity, quiet_NaN。

### Lines 1651-1680 / 第 1651-1680 行
```cpp
1651:   bool code = true;
1652:   T y = y0;
1653:   if (y > one - T{0.13533528323661269189}) { /* 0.135... = exp(-2) */
1654:     y = one - y;
1655:     code = false;
1656:   }
1657: 
1658:   if (y > T{0.13533528323661269189}) {
1659:     y = y - T{0.5};
1660:     const T y2 = y * y;
1661:     T x = y + y * (y2 * polevl(y2, P0, 4) / polevl(y2, Q0, 8));
1662:     return (x * s2pi);
1663:   }
1664: 
1665:   T x = ::sqrt(T{-2.0} * ::log(y));
1666:   const T x0 = x - ::log(x) / x;
1667: 
1668:   const T z = one / x;
1669:   T x1;
1670:   if (x < T{8.0}) /* y > exp(-32) = 1.2664165549e-14 */
1671:   {
1672:     x1 = z * polevl(z, P1, 8) / polevl(z, Q1, 8);
1673:   } else {
1674:     x1 = z * polevl(z, P2, 8) / polevl(z, Q2, 8);
1675:   }
1676:   x = x0 - x1;
1677:   if (code) {
1678:     x = -x;
1679:   }
1680:   return x;
```
- **EN**: Lines 1651-1680 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: exp, polevl, sqrt, log.
- **CN**: 第 1651-1680 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：exp, polevl, sqrt, log。

### Lines 1681-1710 / 第 1681-1710 行
```cpp
1681: }
1682: 
1683: /* The next function is taken from http://ab-initio.mit.edu/faddeeva */
1684: 
1685: /* Copyright (c) 2012 Massachusetts Institute of Technology
1686:  *
1687:  * Permission is hereby granted, free of charge, to any person obtaining
1688:  * a copy of this software and associated documentation files (the
1689:  * "Software"), to deal in the Software without restriction, including
1690:  * without limitation the rights to use, copy, modify, merge, publish,
1691:  * distribute, sublicense, and/or sell copies of the Software, and to
1692:  * permit persons to whom the Software is furnished to do so, subject to
1693:  * the following conditions:
1694:  *
1695:  * The above copyright notice and this permission notice shall be
1696:  * included in all copies or substantial portions of the Software.
1697:  *
1698:  * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
1699:  * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
1700:  * MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
1701:  * NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
1702:  * LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
1703:  * OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
1704:  * WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
1705:  */
1706: 
1707: /* erfcx(x) = exp(x^2) erfc(x) function, for real x, written by
1708:    Steven G. Johnson, October 2012.
1709: 
1710:    This function combines a few different ideas.
```
- **EN**: Lines 1681-1710 mainly cover comments/documentation, expressions/calls. Notable symbols: Copyright, files, erfcx, exp.
- **CN**: 第 1681-1710 行主要涉及注释或说明、表达式或调用。 值得关注的符号包括：Copyright, files, erfcx, exp。

### Lines 1711-1740 / 第 1711-1740 行
```cpp
1711: 
1712:    First, for x > 50, it uses a continued-fraction expansion (same as
1713:    for the Faddeeva function, but with algebraic simplifications for z=i*x).
1714: 
1715:    Second, for 0 <= x <= 50, it uses Chebyshev polynomial approximations,
1716:    but with two twists:
1717: 
1718:       a) It maps x to y = 4 / (4+x) in [0,1].  This simple transformation,
1719:          inspired by a similar transformation in the octave-forge/specfun
1720:          erfcx by Soren Hauberg, results in much faster Chebyshev convergence
1721:          than other simple transformations I have examined.
1722: 
1723:       b) Instead of using a single Chebyshev polynomial for the entire
1724:          [0,1] y interval, we break the interval up into 100 equal
1725:          subintervals, with a switch/lookup table, and use much lower
1726:          degree Chebyshev polynomials in each subinterval. This greatly
1727:          improves performance in my tests.
1728: 
1729:    For x < 0, we use the relationship erfcx(-x) = 2 exp(x^2) - erfc(x),
1730:    with the usual checks for overflow etcetera.
1731: 
1732:    Performance-wise, it seems to be substantially faster than either
1733:    the SLATEC DERFC function [or an erfcx function derived there from]
1734:    or Cody's CALERF function (from netlib.org/specfun), while
1735:    retaining near machine precision in accuracy.  */
1736: 
1737: /* Given y100=100*y, where y = 4/(4+x) for x >= 0, compute erfc(x).
1738: 
1739:    Uses a look-up table of 100 different Chebyshev polynomials
1740:    for y intervals [0,0.01], [0.01,0.02], ...., [0.99,1], generated
```
- **EN**: Lines 1711-1740 mainly cover expressions/calls, function signatures/definitions, control-flow checks. Notable symbols: expansion, erfcx, exp, erfc.
- **CN**: 第 1711-1740 行主要涉及表达式或调用、函数签名或实现、控制流逻辑。 值得关注的符号包括：expansion, erfcx, exp, erfc。

### Lines 1741-1770 / 第 1741-1770 行
```cpp
1741:    with the help of Maple and a little shell script.   This allows
1742:    the Chebyshev polynomials to be of significantly lower degree (about 1/4)
1743:    compared to fitting the whole [0,1] interval with a single polynomial. */
1744: 
1745: 
1746: template <typename T>
1747: C10_HOST_DEVICE  inline typename std::enable_if_t<std::is_floating_point_v<T>, T>
1748: erfcx_y100(T y100)
1749: {
1750:   switch (static_cast<int>(y100)) {
1751: case 0: {
1752: T t = 2*y100 - 1;
1753: return 0.70878032454106438663e-3 + (0.71234091047026302958e-3 + (0.35779077297597742384e-5 + (0.17403143962587937815e-7 + (0.81710660047307788845e-10 + (0.36885022360434957634e-12 + 0.15917038551111111111e-14 * t) * t) * t) * t) * t) * t;
1754: }
1755: case 1: {
1756: T t = 2*y100 - 3;
1757: return 0.21479143208285144230e-2 + (0.72686402367379996033e-3 + (0.36843175430938995552e-5 + (0.18071841272149201685e-7 + (0.85496449296040325555e-10 + (0.38852037518534291510e-12 + 0.16868473576888888889e-14 * t) * t) * t) * t) * t) * t;
1758: }
1759: case 2: {
1760: T t = 2*y100 - 5;
1761: return 0.36165255935630175090e-2 + (0.74182092323555510862e-3 + (0.37948319957528242260e-5 + (0.18771627021793087350e-7 + (0.89484715122415089123e-10 + (0.40935858517772440862e-12 + 0.17872061464888888889e-14 * t) * t) * t) * t) * t) * t;
1762: }
1763: case 3: {
1764: T t = 2*y100 - 7;
1765: return 0.51154983860031979264e-2 + (0.75722840734791660540e-3 + (0.39096425726735703941e-5 + (0.19504168704300468210e-7 + (0.93687503063178993915e-10 + (0.43143925959079664747e-12 + 0.18939926435555555556e-14 * t) * t) * t) * t) * t) * t;
1766: }
1767: case 4: {
1768: T t = 2*y100 - 9;
1769: return 0.66457513172673049824e-2 + (0.77310406054447454920e-3 + (0.40289510589399439385e-5 + (0.20271233238288381092e-7 + (0.98117631321709100264e-10 + (0.45484207406017752971e-12 + 0.20076352213333333333e-14 * t) * t) * t) * t) * t) * t;
1770: }
```
- **EN**: Lines 1741-1770 mainly cover expressions/calls, control-flow checks, state/variable declarations. Notable symbols: degree, erfcx_y100.
- **CN**: 第 1741-1770 行主要涉及表达式或调用、控制流逻辑、变量/别名声明。 值得关注的符号包括：degree, erfcx_y100。

### Lines 1771-1800 / 第 1771-1800 行
```cpp
1771: case 5: {
1772: T t = 2*y100 - 11;
1773: return 0.82082389970241207883e-2 + (0.78946629611881710721e-3 + (0.41529701552622656574e-5 + (0.21074693344544655714e-7 + (0.10278874108587317989e-9 + (0.47965201390613339638e-12 + 0.21285907413333333333e-14 * t) * t) * t) * t) * t) * t;
1774: }
1775: case 6: {
1776: T t = 2*y100 - 13;
1777: return 0.98039537275352193165e-2 + (0.80633440108342840956e-3 + (0.42819241329736982942e-5 + (0.21916534346907168612e-7 + (0.10771535136565470914e-9 + (0.50595972623692822410e-12 + 0.22573462684444444444e-14 * t) * t) * t) * t) * t) * t;
1778: }
1779: case 7: {
1780: T t = 2*y100 - 15;
1781: return 0.11433927298290302370e-1 + (0.82372858383196561209e-3 + (0.44160495311765438816e-5 + (0.22798861426211986056e-7 + (0.11291291745879239736e-9 + (0.53386189365816880454e-12 + 0.23944209546666666667e-14 * t) * t) * t) * t) * t) * t;
1782: }
1783: case 8: {
1784: T t = 2*y100 - 17;
1785: return 0.13099232878814653979e-1 + (0.84167002467906968214e-3 + (0.45555958988457506002e-5 + (0.23723907357214175198e-7 + (0.11839789326602695603e-9 + (0.56346163067550237877e-12 + 0.25403679644444444444e-14 * t) * t) * t) * t) * t) * t;
1786: }
1787: case 9: {
1788: T t = 2*y100 - 19;
1789: return 0.14800987015587535621e-1 + (0.86018092946345943214e-3 + (0.47008265848816866105e-5 + (0.24694040760197315333e-7 + (0.12418779768752299093e-9 + (0.59486890370320261949e-12 + 0.26957764568888888889e-14 * t) * t) * t) * t) * t) * t;
1790: }
1791: case 10: {
1792: T t = 2*y100 - 21;
1793: return 0.16540351739394069380e-1 + (0.87928458641241463952e-3 + (0.48520195793001753903e-5 + (0.25711774900881709176e-7 + (0.13030128534230822419e-9 + (0.62820097586874779402e-12 + 0.28612737351111111111e-14 * t) * t) * t) * t) * t) * t;
1794: }
1795: case 11: {
1796: T t = 2*y100 - 23;
1797: return 0.18318536789842392647e-1 + (0.89900542647891721692e-3 + (0.50094684089553365810e-5 + (0.26779777074218070482e-7 + (0.13675822186304615566e-9 + (0.66358287745352705725e-12 + 0.30375273884444444444e-14 * t) * t) * t) * t) * t) * t;
1798: }
1799: case 12: {
1800: T t = 2*y100 - 25;
```
- **EN**: Lines 1771-1800 mainly cover control-flow checks, state/variable declarations, return paths.
- **CN**: 第 1771-1800 行主要涉及控制流逻辑、变量/别名声明、返回路径。

### Lines 1801-1830 / 第 1801-1830 行
```cpp
1801: return 0.20136801964214276775e-1 + (0.91936908737673676012e-3 + (0.51734830914104276820e-5 + (0.27900878609710432673e-7 + (0.14357976402809042257e-9 + (0.70114790311043728387e-12 + 0.32252476000000000000e-14 * t) * t) * t) * t) * t) * t;
1802: }
1803: case 13: {
1804: T t = 2*y100 - 27;
1805: return 0.21996459598282740954e-1 + (0.94040248155366777784e-3 + (0.53443911508041164739e-5 + (0.29078085538049374673e-7 + (0.15078844500329731137e-9 + (0.74103813647499204269e-12 + 0.34251892320000000000e-14 * t) * t) * t) * t) * t) * t;
1806: }
1807: case 14: {
1808: T t = 2*y100 - 29;
1809: return 0.23898877187226319502e-1 + (0.96213386835900177540e-3 + (0.55225386998049012752e-5 + (0.30314589961047687059e-7 + (0.15840826497296335264e-9 + (0.78340500472414454395e-12 + 0.36381553564444444445e-14 * t) * t) * t) * t) * t) * t;
1810: }
1811: case 15: {
1812: T t = 2*y100 - 31;
1813: return 0.25845480155298518485e-1 + (0.98459293067820123389e-3 + (0.57082915920051843672e-5 + (0.31613782169164830118e-7 + (0.16646478745529630813e-9 + (0.82840985928785407942e-12 + 0.38649975768888888890e-14 * t) * t) * t) * t) * t) * t;
1814: }
1815: case 16: {
1816: T t = 2*y100 - 33;
1817: return 0.27837754783474696598e-1 + (0.10078108563256892757e-2 + (0.59020366493792212221e-5 + (0.32979263553246520417e-7 + (0.17498524159268458073e-9 + (0.87622459124842525110e-12 + 0.41066206488888888890e-14 * t) * t) * t) * t) * t) * t;
1818: }
1819: case 17: {
1820: T t = 2*y100 - 35;
1821: return 0.29877251304899307550e-1 + (0.10318204245057349310e-2 + (0.61041829697162055093e-5 + (0.34414860359542720579e-7 + (0.18399863072934089607e-9 + (0.92703227366365046533e-12 + 0.43639844053333333334e-14 * t) * t) * t) * t) * t) * t;
1822: }
1823: case 18: {
1824: T t = 2*y100 - 37;
1825: return 0.31965587178596443475e-1 + (0.10566560976716574401e-2 + (0.63151633192414586770e-5 + (0.35924638339521924242e-7 + (0.19353584758781174038e-9 + (0.98102783859889264382e-12 + 0.46381060817777777779e-14 * t) * t) * t) * t) * t) * t;
1826: }
1827: case 19: {
1828: T t = 2*y100 - 39;
1829: return 0.34104450552588334840e-1 + (0.10823541191350532574e-2 + (0.65354356159553934436e-5 + (0.37512918348533521149e-7 + (0.20362979635817883229e-9 + (0.10384187833037282363e-11 + 0.49300625262222222221e-14 * t) * t) * t) * t) * t) * t;
1830: }
```
- **EN**: Lines 1801-1830 mainly cover return paths, expressions/calls, control-flow checks.
- **CN**: 第 1801-1830 行主要涉及返回路径、表达式或调用、控制流逻辑。

### Lines 1831-1860 / 第 1831-1860 行
```cpp
1831: case 20: {
1832: T t = 2*y100 - 41;
1833: return 0.36295603928292425716e-1 + (0.11089526167995268200e-2 + (0.67654845095518363577e-5 + (0.39184292949913591646e-7 + (0.21431552202133775150e-9 + (0.10994259106646731797e-11 + 0.52409949102222222221e-14 * t) * t) * t) * t) * t) * t;
1834: }
1835: case 21: {
1836: T t = 2*y100 - 43;
1837: return 0.38540888038840509795e-1 + (0.11364917134175420009e-2 + (0.70058230641246312003e-5 + (0.40943644083718586939e-7 + (0.22563034723692881631e-9 + (0.11642841011361992885e-11 + 0.55721092871111111110e-14 * t) * t) * t) * t) * t) * t;
1838: }
1839: case 22: {
1840: T t = 2*y100 - 45;
1841: return 0.40842225954785960651e-1 + (0.11650136437945673891e-2 + (0.72569945502343006619e-5 + (0.42796161861855042273e-7 + (0.23761401711005024162e-9 + (0.12332431172381557035e-11 + 0.59246802364444444445e-14 * t) * t) * t) * t) * t) * t;
1842: }
1843: case 23: {
1844: T t = 2*y100 - 47;
1845: return 0.43201627431540222422e-1 + (0.11945628793917272199e-2 + (0.75195743532849206263e-5 + (0.44747364553960993492e-7 + (0.25030885216472953674e-9 + (0.13065684400300476484e-11 + 0.63000532853333333334e-14 * t) * t) * t) * t) * t) * t;
1846: }
1847: case 24: {
1848: T t = 2*y100 - 49;
1849: return 0.45621193513810471438e-1 + (0.12251862608067529503e-2 + (0.77941720055551920319e-5 + (0.46803119830954460212e-7 + (0.26375990983978426273e-9 + (0.13845421370977119765e-11 + 0.66996477404444444445e-14 * t) * t) * t) * t) * t) * t;
1850: }
1851: case 25: {
1852: T t = 2*y100 - 51;
1853: return 0.48103121413299865517e-1 + (0.12569331386432195113e-2 + (0.80814333496367673980e-5 + (0.48969667335682018324e-7 + (0.27801515481905748484e-9 + (0.14674637611609884208e-11 + 0.71249589351111111110e-14 * t) * t) * t) * t) * t) * t;
1854: }
1855: case 26: {
1856: T t = 2*y100 - 53;
1857: return 0.50649709676983338501e-1 + (0.12898555233099055810e-2 + (0.83820428414568799654e-5 + (0.51253642652551838659e-7 + (0.29312563849675507232e-9 + (0.15556512782814827846e-11 + 0.75775607822222222221e-14 * t) * t) * t) * t) * t) * t;
1858: }
1859: case 27: {
1860: T t = 2*y100 - 55;
```
- **EN**: Lines 1831-1860 mainly cover control-flow checks, state/variable declarations, return paths.
- **CN**: 第 1831-1860 行主要涉及控制流逻辑、变量/别名声明、返回路径。

### Lines 1861-1890 / 第 1861-1890 行
```cpp
1861: return 0.53263363664388864181e-1 + (0.13240082443256975769e-2 + (0.86967260015007658418e-5 + (0.53662102750396795566e-7 + (0.30914568786634796807e-9 + (0.16494420240828493176e-11 + 0.80591079644444444445e-14 * t) * t) * t) * t) * t) * t;
1862: }
1863: case 28: {
1864: T t = 2*y100 - 57;
1865: return 0.55946601353500013794e-1 + (0.13594491197408190706e-2 + (0.90262520233016380987e-5 + (0.56202552975056695376e-7 + (0.32613310410503135996e-9 + (0.17491936862246367398e-11 + 0.85713381688888888890e-14 * t) * t) * t) * t) * t) * t;
1866: }
1867: case 29: {
1868: T t = 2*y100 - 59;
1869: return 0.58702059496154081813e-1 + (0.13962391363223647892e-2 + (0.93714365487312784270e-5 + (0.58882975670265286526e-7 + (0.34414937110591753387e-9 + (0.18552853109751857859e-11 + 0.91160736711111111110e-14 * t) * t) * t) * t) * t) * t;
1870: }
1871: case 30: {
1872: T t = 2*y100 - 61;
1873: return 0.61532500145144778048e-1 + (0.14344426411912015247e-2 + (0.97331446201016809696e-5 + (0.61711860507347175097e-7 + (0.36325987418295300221e-9 + (0.19681183310134518232e-11 + 0.96952238400000000000e-14 * t) * t) * t) * t) * t) * t;
1874: }
1875: case 31: {
1876: T t = 2*y100 - 63;
1877: return 0.64440817576653297993e-1 + (0.14741275456383131151e-2 + (0.10112293819576437838e-4 + (0.64698236605933246196e-7 + (0.38353412915303665586e-9 + (0.20881176114385120186e-11 + 0.10310784480000000000e-13 * t) * t) * t) * t) * t) * t;
1878: }
1879: case 32: {
1880: T t = 2*y100 - 65;
1881: return 0.67430045633130393282e-1 + (0.15153655418916540370e-2 + (0.10509857606888328667e-4 + (0.67851706529363332855e-7 + (0.40504602194811140006e-9 + (0.22157325110542534469e-11 + 0.10964842115555555556e-13 * t) * t) * t) * t) * t) * t;
1882: }
1883: case 33: {
1884: T t = 2*y100 - 67;
1885: return 0.70503365513338850709e-1 + (0.15582323336495709827e-2 + (0.10926868866865231089e-4 + (0.71182482239613507542e-7 + (0.42787405890153386710e-9 + (0.23514379522274416437e-11 + 0.11659571751111111111e-13 * t) * t) * t) * t) * t) * t;
1886: }
1887: case 34: {
1888: T t = 2*y100 - 69;
1889: return 0.73664114037944596353e-1 + (0.16028078812438820413e-2 + (0.11364423678778207991e-4 + (0.74701423097423182009e-7 + (0.45210162777476488324e-9 + (0.24957355004088569134e-11 + 0.12397238257777777778e-13 * t) * t) * t) * t) * t) * t;
1890: }
```
- **EN**: Lines 1861-1890 mainly cover return paths, expressions/calls, control-flow checks.
- **CN**: 第 1861-1890 行主要涉及返回路径、表达式或调用、控制流逻辑。

### Lines 1891-1920 / 第 1891-1920 行
```cpp
1891: case 35: {
1892: T t = 2*y100 - 71;
1893: return 0.76915792420819562379e-1 + (0.16491766623447889354e-2 + (0.11823685320041302169e-4 + (0.78420075993781544386e-7 + (0.47781726956916478925e-9 + (0.26491544403815724749e-11 + 0.13180196462222222222e-13 * t) * t) * t) * t) * t) * t;
1894: }
1895: case 36: {
1896: T t = 2*y100 - 73;
1897: return 0.80262075578094612819e-1 + (0.16974279491709504117e-2 + (0.12305888517309891674e-4 + (0.82350717698979042290e-7 + (0.50511496109857113929e-9 + (0.28122528497626897696e-11 + 0.14010889635555555556e-13 * t) * t) * t) * t) * t) * t;
1898: }
1899: case 37: {
1900: T t = 2*y100 - 75;
1901: return 0.83706822008980357446e-1 + (0.17476561032212656962e-2 + (0.12812343958540763368e-4 + (0.86506399515036435592e-7 + (0.53409440823869467453e-9 + (0.29856186620887555043e-11 + 0.14891851591111111111e-13 * t) * t) * t) * t) * t) * t;
1902: }
1903: case 38: {
1904: T t = 2*y100 - 77;
1905: return 0.87254084284461718231e-1 + (0.17999608886001962327e-2 + (0.13344443080089492218e-4 + (0.90900994316429008631e-7 + (0.56486134972616465316e-9 + (0.31698707080033956934e-11 + 0.15825697795555555556e-13 * t) * t) * t) * t) * t) * t;
1906: }
1907: case 39: {
1908: T t = 2*y100 - 79;
1909: return 0.90908120182172748487e-1 + (0.18544478050657699758e-2 + (0.13903663143426120077e-4 + (0.95549246062549906177e-7 + (0.59752787125242054315e-9 + (0.33656597366099099413e-11 + 0.16815130613333333333e-13 * t) * t) * t) * t) * t) * t;
1910: }
1911: case 40: {
1912: T t = 2*y100 - 81;
1913: return 0.94673404508075481121e-1 + (0.19112284419887303347e-2 + (0.14491572616545004930e-4 + (0.10046682186333613697e-6 + (0.63221272959791000515e-9 + (0.35736693975589130818e-11 + 0.17862931591111111111e-13 * t) * t) * t) * t) * t) * t;
1914: }
1915: case 41: {
1916: T t = 2*y100 - 83;
1917: return 0.98554641648004456555e-1 + (0.19704208544725622126e-2 + (0.15109836875625443935e-4 + (0.10567036667675984067e-6 + (0.66904168640019354565e-9 + (0.37946171850824333014e-11 + 0.18971959040000000000e-13 * t) * t) * t) * t) * t) * t;
1918: }
1919: case 42: {
1920: T t = 2*y100 - 85;
```
- **EN**: Lines 1891-1920 mainly cover control-flow checks, state/variable declarations, return paths.
- **CN**: 第 1891-1920 行主要涉及控制流逻辑、变量/别名声明、返回路径。

### Lines 1921-1950 / 第 1921-1950 行
```cpp
1921: return 0.10255677889470089531e0 + (0.20321499629472857418e-2 + (0.15760224242962179564e-4 + (0.11117756071353507391e-6 + (0.70814785110097658502e-9 + (0.40292553276632563925e-11 + 0.20145143075555555556e-13 * t) * t) * t) * t) * t) * t;
1922: }
1923: case 43: {
1924: T t = 2*y100 - 87;
1925: return 0.10668502059865093318e0 + (0.20965479776148731610e-2 + (0.16444612377624983565e-4 + (0.11700717962026152749e-6 + (0.74967203250938418991e-9 + (0.42783716186085922176e-11 + 0.21385479360000000000e-13 * t) * t) * t) * t) * t) * t;
1926: }
1927: case 44: {
1928: T t = 2*y100 - 89;
1929: return 0.11094484319386444474e0 + (0.21637548491908170841e-2 + (0.17164995035719657111e-4 + (0.12317915750735938089e-6 + (0.79376309831499633734e-9 + (0.45427901763106353914e-11 + 0.22696025653333333333e-13 * t) * t) * t) * t) * t) * t;
1930: }
1931: case 45: {
1932: T t = 2*y100 - 91;
1933: return 0.11534201115268804714e0 + (0.22339187474546420375e-2 + (0.17923489217504226813e-4 + (0.12971465288245997681e-6 + (0.84057834180389073587e-9 + (0.48233721206418027227e-11 + 0.24079890062222222222e-13 * t) * t) * t) * t) * t) * t;
1934: }
1935: case 46: {
1936: T t = 2*y100 - 93;
1937: return 0.11988259392684094740e0 + (0.23071965691918689601e-2 + (0.18722342718958935446e-4 + (0.13663611754337957520e-6 + (0.89028385488493287005e-9 + (0.51210161569225846701e-11 + 0.25540227111111111111e-13 * t) * t) * t) * t) * t) * t;
1938: }
1939: case 47: {
1940: T t = 2*y100 - 95;
1941: return 0.12457298393509812907e0 + (0.23837544771809575380e-2 + (0.19563942105711612475e-4 + (0.14396736847739470782e-6 + (0.94305490646459247016e-9 + (0.54366590583134218096e-11 + 0.27080225920000000000e-13 * t) * t) * t) * t) * t) * t;
1942: }
1943: case 48: {
1944: T t = 2*y100 - 97;
1945: return 0.12941991566142438816e0 + (0.24637684719508859484e-2 + (0.20450821127475879816e-4 + (0.15173366280523906622e-6 + (0.99907632506389027739e-9 + (0.57712760311351625221e-11 + 0.28703099555555555556e-13 * t) * t) * t) * t) * t) * t;
1946: }
1947: case 49: {
1948: T t = 2*y100 - 99;
1949: return 0.13443048593088696613e0 + (0.25474249981080823877e-2 + (0.21385669591362915223e-4 + (0.15996177579900443030e-6 + (0.10585428844575134013e-8 + (0.61258809536787882989e-11 + 0.30412080142222222222e-13 * t) * t) * t) * t) * t) * t;
1950: }
```
- **EN**: Lines 1921-1950 mainly cover return paths, expressions/calls, control-flow checks.
- **CN**: 第 1921-1950 行主要涉及返回路径、表达式或调用、控制流逻辑。

### Lines 1951-1980 / 第 1951-1980 行
```cpp
1951: case 50: {
1952: T t = 2*y100 - 101;
1953: return 0.13961217543434561353e0 + (0.26349215871051761416e-2 + (0.22371342712572567744e-4 + (0.16868008199296822247e-6 + (0.11216596910444996246e-8 + (0.65015264753090890662e-11 + 0.32210394506666666666e-13 * t) * t) * t) * t) * t) * t;
1954: }
1955: case 51: {
1956: T t = 2*y100 - 103;
1957: return 0.14497287157673800690e0 + (0.27264675383982439814e-2 + (0.23410870961050950197e-4 + (0.17791863939526376477e-6 + (0.11886425714330958106e-8 + (0.68993039665054288034e-11 + 0.34101266222222222221e-13 * t) * t) * t) * t) * t) * t;
1958: }
1959: case 52: {
1960: T t = 2*y100 - 105;
1961: return 0.15052089272774618151e0 + (0.28222846410136238008e-2 + (0.24507470422713397006e-4 + (0.18770927679626136909e-6 + (0.12597184587583370712e-8 + (0.73203433049229821618e-11 + 0.36087889048888888890e-13 * t) * t) * t) * t) * t) * t;
1962: }
1963: case 53: {
1964: T t = 2*y100 - 107;
1965: return 0.15626501395774612325e0 + (0.29226079376196624949e-2 + (0.25664553693768450545e-4 + (0.19808568415654461964e-6 + (0.13351257759815557897e-8 + (0.77658124891046760667e-11 + 0.38173420035555555555e-13 * t) * t) * t) * t) * t) * t;
1966: }
1967: case 54: {
1968: T t = 2*y100 - 109;
1969: return 0.16221449434620737567e0 + (0.30276865332726475672e-2 + (0.26885741326534564336e-4 + (0.20908350604346384143e-6 + (0.14151148144240728728e-8 + (0.82369170665974313027e-11 + 0.40360957457777777779e-13 * t) * t) * t) * t) * t) * t;
1970: }
1971: case 55: {
1972: T t = 2*y100 - 111;
1973: return 0.16837910595412130659e0 + (0.31377844510793082301e-2 + (0.28174873844911175026e-4 + (0.22074043807045782387e-6 + (0.14999481055996090039e-8 + (0.87348993661930809254e-11 + 0.42653528977777777779e-13 * t) * t) * t) * t) * t) * t;
1974: }
1975: case 56: {
1976: T t = 2*y100 - 113;
1977: return 0.17476916455659369953e0 + (0.32531815370903068316e-2 + (0.29536024347344364074e-4 + (0.23309632627767074202e-6 + (0.15899007843582444846e-8 + (0.92610375235427359475e-11 + 0.45054073102222222221e-13 * t) * t) * t) * t) * t) * t;
1978: }
1979: case 57: {
1980: T t = 2*y100 - 115;
```
- **EN**: Lines 1951-1980 mainly cover control-flow checks, state/variable declarations, return paths.
- **CN**: 第 1951-1980 行主要涉及控制流逻辑、变量/别名声明、返回路径。

### Lines 1981-2010 / 第 1981-2010 行
```cpp
1981: return 0.18139556223643701364e0 + (0.33741744168096996041e-2 + (0.30973511714709500836e-4 + (0.24619326937592290996e-6 + (0.16852609412267750744e-8 + (0.98166442942854895573e-11 + 0.47565418097777777779e-13 * t) * t) * t) * t) * t) * t;
1982: }
1983: case 58: {
1984: T t = 2*y100 - 117;
1985: return 0.18826980194443664549e0 + (0.35010775057740317997e-2 + (0.32491914440014267480e-4 + (0.26007572375886319028e-6 + (0.17863299617388376116e-8 + (0.10403065638343878679e-10 + 0.50190265831111111110e-13 * t) * t) * t) * t) * t) * t;
1986: }
1987: case 59: {
1988: T t = 2*y100 - 119;
1989: return 0.19540403413693967350e0 + (0.36342240767211326315e-2 + (0.34096085096200907289e-4 + (0.27479061117017637474e-6 + (0.18934228504790032826e-8 + (0.11021679075323598664e-10 + 0.52931171733333333334e-13 * t) * t) * t) * t) * t) * t;
1990: }
1991: case 60: {
1992: T t = 2*y100 - 121;
1993: return 0.20281109560651886959e0 + (0.37739673859323597060e-2 + (0.35791165457592409054e-4 + (0.29038742889416172404e-6 + (0.20068685374849001770e-8 + (0.11673891799578381999e-10 + 0.55790523093333333334e-13 * t) * t) * t) * t) * t) * t;
1994: }
1995: case 61: {
1996: T t = 2*y100 - 123;
1997: return 0.21050455062669334978e0 + (0.39206818613925652425e-2 + (0.37582602289680101704e-4 + (0.30691836231886877385e-6 + (0.21270101645763677824e-8 + (0.12361138551062899455e-10 + 0.58770520160000000000e-13 * t) * t) * t) * t) * t) * t;
1998: }
1999: case 62: {
2000: T t = 2*y100 - 125;
2001: return 0.21849873453703332479e0 + (0.40747643554689586041e-2 + (0.39476163820986711501e-4 + (0.32443839970139918836e-6 + (0.22542053491518680200e-8 + (0.13084879235290858490e-10 + 0.61873153262222222221e-13 * t) * t) * t) * t) * t) * t;
2002: }
2003: case 63: {
2004: T t = 2*y100 - 127;
2005: return 0.22680879990043229327e0 + (0.42366354648628516935e-2 + (0.41477956909656896779e-4 + (0.34300544894502810002e-6 + (0.23888264229264067658e-8 + (0.13846596292818514601e-10 + 0.65100183751111111110e-13 * t) * t) * t) * t) * t) * t;
2006: }
2007: case 64: {
2008: T t = 2*y100 - 129;
2009: return 0.23545076536988703937e0 + (0.44067409206365170888e-2 + (0.43594444916224700881e-4 + (0.36268045617760415178e-6 + (0.25312606430853202748e-8 + (0.14647791812837903061e-10 + 0.68453122631111111110e-13 * t) * t) * t) * t) * t) * t;
2010: }
```
- **EN**: Lines 1981-2010 mainly cover return paths, expressions/calls, control-flow checks.
- **CN**: 第 1981-2010 行主要涉及返回路径、表达式或调用、控制流逻辑。

### Lines 2011-2040 / 第 2011-2040 行
```cpp
2011: case 65: {
2012: T t = 2*y100 - 131;
2013: return 0.24444156740777432838e0 + (0.45855530511605787178e-2 + (0.45832466292683085475e-4 + (0.38352752590033030472e-6 + (0.26819103733055603460e-8 + (0.15489984390884756993e-10 + 0.71933206364444444445e-13 * t) * t) * t) * t) * t) * t;
2014: }
2015: case 66: {
2016: T t = 2*y100 - 133;
2017: return 0.25379911500634264643e0 + (0.47735723208650032167e-2 + (0.48199253896534185372e-4 + (0.40561404245564732314e-6 + (0.28411932320871165585e-8 + (0.16374705736458320149e-10 + 0.75541379822222222221e-13 * t) * t) * t) * t) * t) * t;
2018: }
2019: case 67: {
2020: T t = 2*y100 - 135;
2021: return 0.26354234756393613032e0 + (0.49713289477083781266e-2 + (0.50702455036930367504e-4 + (0.42901079254268185722e-6 + (0.30095422058900481753e-8 + (0.17303497025347342498e-10 + 0.79278273368888888890e-13 * t) * t) * t) * t) * t) * t;
2022: }
2023: case 68: {
2024: T t = 2*y100 - 137;
2025: return 0.27369129607732343398e0 + (0.51793846023052643767e-2 + (0.53350152258326602629e-4 + (0.45379208848865015485e-6 + (0.31874057245814381257e-8 + (0.18277905010245111046e-10 + 0.83144182364444444445e-13 * t) * t) * t) * t) * t) * t;
2026: }
2027: case 69: {
2028: T t = 2*y100 - 139;
2029: return 0.28426714781640316172e0 + (0.53983341916695141966e-2 + (0.56150884865255810638e-4 + (0.48003589196494734238e-6 + (0.33752476967570796349e-8 + (0.19299477888083469086e-10 + 0.87139049137777777779e-13 * t) * t) * t) * t) * t) * t;
2030: }
2031: case 70: {
2032: T t = 2*y100 - 141;
2033: return 0.29529231465348519920e0 + (0.56288077305420795663e-2 + (0.59113671189913307427e-4 + (0.50782393781744840482e-6 + (0.35735475025851713168e-8 + (0.20369760937017070382e-10 + 0.91262442613333333334e-13 * t) * t) * t) * t) * t) * t;
2034: }
2035: case 71: {
2036: T t = 2*y100 - 143;
2037: return 0.30679050522528838613e0 + (0.58714723032745403331e-2 + (0.62248031602197686791e-4 + (0.53724185766200945789e-6 + (0.37827999418960232678e-8 + (0.21490291930444538307e-10 + 0.95513539182222222221e-13 * t) * t) * t) * t) * t) * t;
2038: }
2039: case 72: {
2040: T t = 2*y100 - 145;
```
- **EN**: Lines 2011-2040 mainly cover control-flow checks, state/variable declarations, return paths.
- **CN**: 第 2011-2040 行主要涉及控制流逻辑、变量/别名声明、返回路径。

### Lines 2041-2070 / 第 2041-2070 行
```cpp
2041: return 0.31878680111173319425e0 + (0.61270341192339103514e-2 + (0.65564012259707640976e-4 + (0.56837930287837738996e-6 + (0.40035151353392378882e-8 + (0.22662596341239294792e-10 + 0.99891109760000000000e-13 * t) * t) * t) * t) * t) * t;
2042: }
2043: case 73: {
2044: T t = 2*y100 - 147;
2045: return 0.33130773722152622027e0 + (0.63962406646798080903e-2 + (0.69072209592942396666e-4 + (0.60133006661885941812e-6 + (0.42362183765883466691e-8 + (0.23888182347073698382e-10 + 0.10439349811555555556e-12 * t) * t) * t) * t) * t) * t;
2046: }
2047: case 74: {
2048: T t = 2*y100 - 149;
2049: return 0.34438138658041336523e0 + (0.66798829540414007258e-2 + (0.72783795518603561144e-4 + (0.63619220443228800680e-6 + (0.44814499336514453364e-8 + (0.25168535651285475274e-10 + 0.10901861383111111111e-12 * t) * t) * t) * t) * t) * t;
2050: }
2051: case 75: {
2052: T t = 2*y100 - 151;
2053: return 0.35803744972380175583e0 + (0.69787978834882685031e-2 + (0.76710543371454822497e-4 + (0.67306815308917386747e-6 + (0.47397647975845228205e-8 + (0.26505114141143050509e-10 + 0.11376390933333333333e-12 * t) * t) * t) * t) * t) * t;
2054: }
2055: case 76: {
2056: T t = 2*y100 - 153;
2057: return 0.37230734890119724188e0 + (0.72938706896461381003e-2 + (0.80864854542670714092e-4 + (0.71206484718062688779e-6 + (0.50117323769745883805e-8 + (0.27899342394100074165e-10 + 0.11862637614222222222e-12 * t) * t) * t) * t) * t) * t;
2058: }
2059: case 77: {
2060: T t = 2*y100 - 155;
2061: return 0.38722432730555448223e0 + (0.76260375162549802745e-2 + (0.85259785810004603848e-4 + (0.75329383305171327677e-6 + (0.52979361368388119355e-8 + (0.29352606054164086709e-10 + 0.12360253370666666667e-12 * t) * t) * t) * t) * t) * t;
2062: }
2063: case 78: {
2064: T t = 2*y100 - 157;
2065: return 0.40282355354616940667e0 + (0.79762880915029728079e-2 + (0.89909077342438246452e-4 + (0.79687137961956194579e-6 + (0.55989731807360403195e-8 + (0.30866246101464869050e-10 + 0.12868841946666666667e-12 * t) * t) * t) * t) * t) * t;
2066: }
2067: case 79: {
2068: T t = 2*y100 - 159;
2069: return 0.41914223158913787649e0 + (0.83456685186950463538e-2 + (0.94827181359250161335e-4 + (0.84291858561783141014e-6 + (0.59154537751083485684e-8 + (0.32441553034347469291e-10 + 0.13387957943111111111e-12 * t) * t) * t) * t) * t) * t;
2070: }
```
- **EN**: Lines 2041-2070 mainly cover return paths, expressions/calls, control-flow checks.
- **CN**: 第 2041-2070 行主要涉及返回路径、表达式或调用、控制流逻辑。

### Lines 2071-2100 / 第 2071-2100 行
```cpp
2071: case 80: {
2072: T t = 2*y100 - 161;
2073: return 0.43621971639463786896e0 + (0.87352841828289495773e-2 + (0.10002929142066799966e-3 + (0.89156148280219880024e-6 + (0.62480008150788597147e-8 + (0.34079760983458878910e-10 + 0.13917107176888888889e-12 * t) * t) * t) * t) * t) * t;
2074: }
2075: case 81: {
2076: T t = 2*y100 - 163;
2077: return 0.45409763548534330981e0 + (0.91463027755548240654e-2 + (0.10553137232446167258e-3 + (0.94293113464638623798e-6 + (0.65972492312219959885e-8 + (0.35782041795476563662e-10 + 0.14455745872000000000e-12 * t) * t) * t) * t) * t) * t;
2078: }
2079: case 82: {
2080: T t = 2*y100 - 165;
2081: return 0.47282001668512331468e0 + (0.95799574408860463394e-2 + (0.11135019058000067469e-3 + (0.99716373005509038080e-6 + (0.69638453369956970347e-8 + (0.37549499088161345850e-10 + 0.15003280712888888889e-12 * t) * t) * t) * t) * t) * t;
2082: }
2083: case 83: {
2084: T t = 2*y100 - 167;
2085: return 0.49243342227179841649e0 + (0.10037550043909497071e-1 + (0.11750334542845234952e-3 + (0.10544006716188967172e-5 + (0.73484461168242224872e-8 + (0.39383162326435752965e-10 + 0.15559069118222222222e-12 * t) * t) * t) * t) * t) * t;
2086: }
2087: case 84: {
2088: T t = 2*y100 - 169;
2089: return 0.51298708979209258326e0 + (0.10520454564612427224e-1 + (0.12400930037494996655e-3 + (0.11147886579371265246e-5 + (0.77517184550568711454e-8 + (0.41283980931872622611e-10 + 0.16122419680000000000e-12 * t) * t) * t) * t) * t) * t;
2090: }
2091: case 85: {
2092: T t = 2*y100 - 171;
2093: return 0.53453307979101369843e0 + (0.11030120618800726938e-1 + (0.13088741519572269581e-3 + (0.11784797595374515432e-5 + (0.81743383063044825400e-8 + (0.43252818449517081051e-10 + 0.16692592640000000000e-12 * t) * t) * t) * t) * t) * t;
2094: }
2095: case 86: {
2096: T t = 2*y100 - 173;
2097: return 0.55712643071169299478e0 + (0.11568077107929735233e-1 + (0.13815797838036651289e-3 + (0.12456314879260904558e-5 + (0.86169898078969313597e-8 + (0.45290446811539652525e-10 + 0.17268801084444444444e-12 * t) * t) * t) * t) * t) * t;
2098: }
2099: case 87: {
2100: T t = 2*y100 - 175;
```
- **EN**: Lines 2071-2100 mainly cover control-flow checks, state/variable declarations, return paths.
- **CN**: 第 2071-2100 行主要涉及控制流逻辑、变量/别名声明、返回路径。

### Lines 2101-2130 / 第 2101-2130 行
```cpp
2101: return 0.58082532122519320968e0 + (0.12135935999503877077e-1 + (0.14584223996665838559e-3 + (0.13164068573095710742e-5 + (0.90803643355106020163e-8 + (0.47397540713124619155e-10 + 0.17850211608888888889e-12 * t) * t) * t) * t) * t) * t;
2102: }
2103: case 88: {
2104: T t = 2*y100 - 177;
2105: return 0.60569124025293375554e0 + (0.12735396239525550361e-1 + (0.15396244472258863344e-3 + (0.13909744385382818253e-5 + (0.95651595032306228245e-8 + (0.49574672127669041550e-10 + 0.18435945564444444444e-12 * t) * t) * t) * t) * t) * t;
2106: }
2107: case 89: {
2108: T t = 2*y100 - 179;
2109: return 0.63178916494715716894e0 + (0.13368247798287030927e-1 + (0.16254186562762076141e-3 + (0.14695084048334056083e-5 + (0.10072078109604152350e-7 + (0.51822304995680707483e-10 + 0.19025081422222222222e-12 * t) * t) * t) * t) * t) * t;
2110: }
2111: case 90: {
2112: T t = 2*y100 - 181;
2113: return 0.65918774689725319200e0 + (0.14036375850601992063e-1 + (0.17160483760259706354e-3 + (0.15521885688723188371e-5 + (0.10601827031535280590e-7 + (0.54140790105837520499e-10 + 0.19616655146666666667e-12 * t) * t) * t) * t) * t) * t;
2114: }
2115: case 91: {
2116: T t = 2*y100 - 183;
2117: return 0.68795950683174433822e0 + (0.14741765091365869084e-1 + (0.18117679143520433835e-3 + (0.16392004108230585213e-5 + (0.11155116068018043001e-7 + (0.56530360194925690374e-10 + 0.20209663662222222222e-12 * t) * t) * t) * t) * t) * t;
2118: }
2119: case 92: {
2120: T t = 2*y100 - 185;
2121: return 0.71818103808729967036e0 + (0.15486504187117112279e-1 + (0.19128428784550923217e-3 + (0.17307350969359975848e-5 + (0.11732656736113607751e-7 + (0.58991125287563833603e-10 + 0.20803065333333333333e-12 * t) * t) * t) * t) * t) * t;
2122: }
2123: case 93: {
2124: T t = 2*y100 - 187;
2125: return 0.74993321911726254661e0 + (0.16272790364044783382e-1 + (0.20195505163377912645e-3 + (0.18269894883203346953e-5 + (0.12335161021630225535e-7 + (0.61523068312169087227e-10 + 0.21395783431111111111e-12 * t) * t) * t) * t) * t) * t;
2126: }
2127: case 94: {
2128: T t = 2*y100 - 189;
2129: return 0.78330143531283492729e0 + (0.17102934132652429240e-1 + (0.21321800585063327041e-3 + (0.19281661395543913713e-5 + (0.12963340087354341574e-7 + (0.64126040998066348872e-10 + 0.21986708942222222222e-12 * t) * t) * t) * t) * t) * t;
2130: }
```
- **EN**: Lines 2101-2130 mainly cover return paths, expressions/calls, control-flow checks.
- **CN**: 第 2101-2130 行主要涉及返回路径、表达式或调用、控制流逻辑。

### Lines 2131-2160 / 第 2131-2160 行
```cpp
2131: case 95: {
2132: T t = 2*y100 - 191;
2133: return 0.81837581041023811832e0 + (0.17979364149044223802e-1 + (0.22510330592753129006e-3 + (0.20344732868018175389e-5 + (0.13617902941839949718e-7 + (0.66799760083972474642e-10 + 0.22574701262222222222e-12 * t) * t) * t) * t) * t) * t;
2134: }
2135: case 96: {
2136: T t = 2*y100 - 193;
2137: return 0.85525144775685126237e0 + (0.18904632212547561026e-1 + (0.23764237370371255638e-3 + (0.21461248251306387979e-5 + (0.14299555071870523786e-7 + (0.69543803864694171934e-10 + 0.23158593688888888889e-12 * t) * t) * t) * t) * t) * t;
2138: }
2139: case 97: {
2140: T t = 2*y100 - 195;
2141: return 0.89402868170849933734e0 + (0.19881418399127202569e-1 + (0.25086793128395995798e-3 + (0.22633402747585233180e-5 + (0.15008997042116532283e-7 + (0.72357609075043941261e-10 + 0.23737194737777777778e-12 * t) * t) * t) * t) * t) * t;
2142: }
2143: case 98: {
2144: T t = 2*y100 - 197;
2145: return 0.93481333942870796363e0 + (0.20912536329780368893e-1 + (0.26481403465998477969e-3 + (0.23863447359754921676e-5 + (0.15746923065472184451e-7 + (0.75240468141720143653e-10 + 0.24309291271111111111e-12 * t) * t) * t) * t) * t) * t;
2146: }
2147: case 99: {
2148: T t = 2*y100 - 199;
2149: return 0.97771701335885035464e0 + (0.22000938572830479551e-1 + (0.27951610702682383001e-3 + (0.25153688325245314530e-5 + (0.16514019547822821453e-7 + (0.78191526829368231251e-10 + 0.24873652355555555556e-12 * t) * t) * t) * t) * t) * t;
2150: }
2151:   }
2152:   // we only get here if y = 1, i.e. |x| < 4*eps, in which case
2153:   // erfcx is within 1e-15 of 1..
2154:   return 1.0;
2155: }
2156: 
2157: template <typename T>
2158: C10_HOST_DEVICE inline typename std::enable_if_t<std::is_floating_point_v<T>, T>
2159: calc_erfcx(T x)
2160: {
```
- **EN**: Lines 2131-2160 mainly cover expressions/calls, return paths, control-flow checks. Notable symbols: calc_erfcx.
- **CN**: 第 2131-2160 行主要涉及表达式或调用、返回路径、控制流逻辑。 值得关注的符号包括：calc_erfcx。

### Lines 2161-2190 / 第 2161-2190 行
```cpp
2161:   if (at::_isnan(x)) {
2162:     return x;
2163:   }
2164: 
2165:   if (x >= 0) {
2166:     if (x > 50) { // continued-fraction expansion is faster
2167:       const T ispi = 0.56418958354775628694807945156; // 1 / sqrt(pi)
2168:       if (x > 5e7) { // 1-term expansion, important to avoid overflow
2169:         return ispi / x;
2170:       }
2171:       /* 5-term expansion (rely on compiler for CSE), simplified from:
2172:                 ispi / (x+0.5/(x+1/(x+1.5/(x+2/x))))  */
2173:       return ispi*((x*x) * (x*x+4.5) + 2) / (x * ((x*x) * (x*x+5) + 3.75));
2174:     }
2175:     return erfcx_y100(400/(4+x));
2176:   }
2177:   else {
2178:     if (x < -26.7) {
2179:       return std::numeric_limits<T>::infinity();
2180:     }
2181:     else if (x < -6.1) {
2182:       return 2*exp(x*x);
2183:     }
2184:     else {
2185:       return 2*exp(x*x) - erfcx_y100(400/(4-x));
2186:     }
2187:   }
2188: }
2189: 
2190: /*
```
- **EN**: Lines 2161-2190 mainly cover expressions/calls, control-flow checks, return paths. Notable symbols: _isnan, sqrt, expansion, erfcx_y100.
- **CN**: 第 2161-2190 行主要涉及表达式或调用、控制流逻辑、返回路径。 值得关注的符号包括：_isnan, sqrt, expansion, erfcx_y100。

### Lines 2191-2220 / 第 2191-2220 行
```cpp
2191:  * Logarithm of Gaussian cumulative distribution function.
2192: 
2193:  * This implementation of log_ndtr and its helper functions
2194:  * follow SciPy's implementation
2195:  * See NOTICE for the licenses.
2196:  */
2197: template <typename T>
2198: inline C10_HOST_DEVICE T calc_log_ndtr(T x) {
2199:   T t = x * c10::frac_sqrt_2<T>;
2200:   if (x < T{-1.0}) {
2201:     return std::log(calc_erfcx(-t) / 2) - t * t;
2202:   } else {
2203:     return std::log1p(-std::erfc(t) / 2);
2204:   }
2205: }
2206: 
2207: template<typename T>
2208: inline C10_HOST_DEVICE T airy_ai_forward(T x) {
2209:     static const T AN[] = {
2210:             +3.46538101525629032477e-01,
2211:             +1.20075952739645805542e+01,
2212:             +7.62796053615234516538e+01,
2213:             +1.68089224934630576269e+02,
2214:             +1.59756391350164413639e+02,
2215:             +7.05360906840444183113e+01,
2216:             +1.40264691163389668864e+01,
2217:             +9.99999999999999995305e-01,
2218:     };
2219: 
2220:     static const T AD[] = {
```
- **EN**: Lines 2191-2220 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: calc_log_ndtr, log, calc_erfcx, log1p.
- **CN**: 第 2191-2220 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：calc_log_ndtr, log, calc_erfcx, log1p。

### Lines 2221-2250 / 第 2221-2250 行
```cpp
2221:             +5.67594532638770212846e-01,
2222:             +1.47562562584847203173e+01,
2223:             +8.45138970141474626562e+01,
2224:             +1.77318088145400459522e+02,
2225:             +1.64234692871529701831e+02,
2226:             +7.14778400825575695274e+01,
2227:             +1.40959135607834029598e+01,
2228:             +1.00000000000000000470e+00,
2229:     };
2230: 
2231:     static const T AFN[] = {
2232:             -1.31696323418331795333e-01,
2233:             -6.26456544431912369773e-01,
2234:             -6.93158036036933542233e-01,
2235:             -2.79779981545119124951e-01,
2236:             -4.91900132609500318020e-02,
2237:             -4.06265923594885404393e-03,
2238:             -1.59276496239262096340e-04,
2239:             -2.77649108155232920844e-06,
2240:             -1.67787698489114633780e-08,
2241:     };
2242: 
2243:     static const T AFD[] = {
2244:             +1.33560420706553243746e+01,
2245:             +3.26825032795224613948e+01,
2246:             +2.67367040941499554804e+01,
2247:             +9.18707402907259625840e+00,
2248:             +1.47529146771666414581e+00,
2249:             +1.15687173795188044134e-01,
2250:             +4.40291641615211203805e-03,
```
- **EN**: Lines 2221-2250 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 2221-2250 行主要涉及表达式或调用、变量/别名声明。

### Lines 2251-2280 / 第 2251-2280 行
```cpp
2251:             +7.54720348287414296618e-05,
2252:             +4.51850092970580378464e-07,
2253:     };
2254: 
2255:     static const T AGN[] = {
2256:             +1.97339932091685679179e-02,
2257:             +3.91103029615688277255e-01,
2258:             +1.06579897599595591108e+00,
2259:             +9.39169229816650230044e-01,
2260:             +3.51465656105547619242e-01,
2261:             +6.33888919628925490927e-02,
2262:             +5.85804113048388458567e-03,
2263:             +2.82851600836737019778e-04,
2264:             +6.98793669997260967291e-06,
2265:             +8.11789239554389293311e-08,
2266:             +3.41551784765923618484e-10,
2267:     };
2268: 
2269:     static const T AGD[] = {
2270:             +9.30892908077441974853e+00,
2271:             +1.98352928718312140417e+01,
2272:             +1.55646628932864612953e+01,
2273:             +5.47686069422975497931e+00,
2274:             +9.54293611618961883998e-01,
2275:             +8.64580826352392193095e-02,
2276:             +4.12656523824222607191e-03,
2277:             +1.01259085116509135510e-04,
2278:             +1.17166733214413521882e-06,
2279:             +4.91834570062930015649e-09,
2280:     };
```
- **EN**: Lines 2251-2280 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 2251-2280 行主要涉及表达式或调用、变量/别名声明。

### Lines 2281-2310 / 第 2281-2310 行
```cpp
2281: 
2282:     int domain_flag = 0;
2283: 
2284:     T ai = T(0.0);
2285: 
2286:     if (std::isinf(x)) {
2287:         return std::numeric_limits<T>::quiet_NaN();
2288:     }
2289: 
2290:     if (x > T(103.892)) {
2291:         return T(0.0);
2292:     }
2293: 
2294:     T f;
2295:     T g;
2296:     T k;
2297: 
2298:     if (x < T(-2.09)) {
2299:         T z = T(1.0) / (T(-2.0) * x * std::sqrt(-x) / T(3.0));
2300: 
2301:         T afn = 0.0;
2302: 
2303:         for (uint8_t index = 0; index <= 8; index++) {
2304:             afn = afn * (z * z) + AFN[index];
2305:         }
2306: 
2307:         T afd = 0.0;
2308: 
2309:         for (uint8_t index = 0; index <= 8; index++) {
2310:             afd = afd * (z * z) + AFD[index];
```
- **EN**: Lines 2281-2310 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: T, isinf, quiet_NaN, sqrt.
- **CN**: 第 2281-2310 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：T, isinf, quiet_NaN, sqrt。

### Lines 2311-2340 / 第 2311-2340 行
```cpp
2311:         }
2312: 
2313:         T agn = 0.0;
2314: 
2315:         for (uint8_t index = 0; index <= 10 + 0; index++) {
2316:             agn = agn * (z * z) + AGN[index];
2317:         }
2318: 
2319:         T agd = 0.0;
2320: 
2321:         for (uint8_t index = 0; index <= 10 - 1; index++) {
2322:             agd = agd * (z * z) + AGD[index];
2323:         }
2324: 
2325:         T t = T(-2.0) * x * std::sqrt(-x) / T(3.0) + T(0.25) * c10::pi<T>;
2326: 
2327:         return T(5.64189583547756286948e-01) / std::sqrt(std::sqrt(-x)) * (std::sin(t) * (T(1.0) + z * z * afn / afd) - std::cos(t) * (z * agn / agd));
2328:     }
2329: 
2330:     if (x >= T(2.09)) {
2331:         domain_flag = 5;
2332: 
2333:         T zeta = T(2.0) * x * std::sqrt(x) / T(3.0);
2334: 
2335:         T an = 0.0;
2336: 
2337:         for (uint8_t index = 0; index <= 7; index++) {
2338:             an = an * (T(1.0) / zeta) + AN[index];
2339:         }
2340: 
```
- **EN**: Lines 2311-2340 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: T, sqrt, sin, cos.
- **CN**: 第 2311-2340 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：T, sqrt, sin, cos。

### Lines 2341-2370 / 第 2341-2370 行
```cpp
2341:         T ad = 0.0;
2342: 
2343:         for (uint8_t index = 0; index <= 7; index++) {
2344:             ad = ad * (T(1.0) / zeta) + AD[index];
2345:         }
2346: 
2347:         ai = T(5.64189583547756286948e-01) * (an / ad) / (T(2.0) * std::sqrt(std::sqrt(x)) * std::exp(zeta));
2348: 
2349:         if (x > T(8.3203353)) {
2350:             return ai;
2351:         }
2352:     }
2353: 
2354:     f = 1.0;
2355:     g = x;
2356:     k = 1.0;
2357: 
2358:     T m = 1.0;
2359:     T n = x;
2360:     T t = 1.0;
2361:     T z = x * x * x;
2362: 
2363:     while (t > std::numeric_limits<T>::epsilon()) {
2364:         m *= z;
2365:         k += T(1.0);
2366:         m /= k;
2367:         n *= z;
2368:         k += T(1.0);
2369:         n /= k;
2370:         m /= k;
```
- **EN**: Lines 2341-2370 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: T, sqrt, exp, epsilon.
- **CN**: 第 2341-2370 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：T, sqrt, exp, epsilon。

### Lines 2371-2400 / 第 2371-2400 行
```cpp
2371:         f += m;
2372:         k += T(1.0);
2373:         n /= k;
2374:         g += n;
2375: 
2376:         t = std::abs(m / f);
2377:     }
2378: 
2379:     if ((domain_flag & 1) == 0) {
2380:         return T(0.355028053887817239260) * f - T(0.258819403792806798405) * g;
2381:     }
2382: 
2383:     return ai;
2384: } // T airy_ai(T x)
2385: 
2386: template<typename T>
2387: inline C10_HOST_DEVICE T bessel_j0_forward(T x) {
2388:     static const T PP[] = {
2389:             +7.96936729297347051624e-04,
2390:             +8.28352392107440799803e-02,
2391:             +1.23953371646414299388e+00,
2392:             +5.44725003058768775090e+00,
2393:             +8.74716500199817011941e+00,
2394:             +5.30324038235394892183e+00,
2395:             +9.99999999999999997821e-01,
2396:     };
2397: 
2398:     static const T PQ[] = {
2399:             +9.24408810558863637013e-04,
2400:             +8.56288474354474431428e-02,
```
- **EN**: Lines 2371-2400 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: T, abs, airy_ai, bessel_j0_forward.
- **CN**: 第 2371-2400 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：T, abs, airy_ai, bessel_j0_forward。

### Lines 2401-2430 / 第 2401-2430 行
```cpp
2401:             +1.25352743901058953537e+00,
2402:             +5.47097740330417105182e+00,
2403:             +8.76190883237069594232e+00,
2404:             +5.30605288235394617618e+00,
2405:             +1.00000000000000000218e+00,
2406:     };
2407: 
2408:     static const T QP[] = {
2409:             -1.13663838898469149931e-02,
2410:             -1.28252718670509318512e+00,
2411:             -1.95539544257735972385e+01,
2412:             -9.32060152123768231369e+01,
2413:             -1.77681167980488050595e+02,
2414:             -1.47077505154951170175e+02,
2415:             -5.14105326766599330220e+01,
2416:             -6.05014350600728481186e+00,
2417:     };
2418: 
2419:     static const T QQ[] = {
2420:             +6.43178256118178023184e+01,
2421:             +8.56430025976980587198e+02,
2422:             +3.88240183605401609683e+03,
2423:             +7.24046774195652478189e+03,
2424:             +5.93072701187316984827e+03,
2425:             +2.06209331660327847417e+03,
2426:             +2.42005740240291393179e+02,
2427:     };
2428: 
2429:     static const T RP[] = {
2430:             -4.79443220978201773821e+09,
```
- **EN**: Lines 2401-2430 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 2401-2430 行主要涉及表达式或调用、变量/别名声明。

### Lines 2431-2460 / 第 2431-2460 行
```cpp
2431:             +1.95617491946556577543e+12,
2432:             -2.49248344360967716204e+14,
2433:             +9.70862251047306323952e+15,
2434:     };
2435: 
2436:     static const T RQ[] = {
2437:             +4.99563147152651017219e+02,
2438:             +1.73785401676374683123e+05,
2439:             +4.84409658339962045305e+07,
2440:             +1.11855537045356834862e+10,
2441:             +2.11277520115489217587e+12,
2442:             +3.10518229857422583814e+14,
2443:             +3.18121955943204943306e+16,
2444:             +1.71086294081043136091e+18,
2445:     };
2446: 
2447:     if (x < T(0)) {
2448:         x = -x;
2449:     }
2450: 
2451:     if (x <= T(5.0)) {
2452:         if (x < T(0.00001)) {
2453:             return T(1.0) - x * x / T(4.0);
2454:         }
2455: 
2456:         T rp = 0.0;
2457: 
2458:         for (uint8_t index = 0; index <= 3; index++) {
2459:             rp = rp * (x * x) + RP[index];
2460:         }
```
- **EN**: Lines 2431-2460 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T.
- **CN**: 第 2431-2460 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T。

### Lines 2461-2490 / 第 2461-2490 行
```cpp
2461: 
2462:         T rq = 0.0;
2463: 
2464:         for (uint8_t index = 0; index <= 7; index++) {
2465:             rq = rq * (x * x) + RQ[index];
2466:         }
2467: 
2468:         return (x * x - T(5.78318596294678452118e+00)) * (x * x - T(3.04712623436620863991e+01)) * rp / rq;
2469:     }
2470: 
2471:     T pp = 0.0;
2472: 
2473:     for (uint8_t index = 0; index <= 6; index++) {
2474:         pp = pp * (T(25.0) / (x * x)) + PP[index];
2475:     }
2476: 
2477:     T pq = 0.0;
2478: 
2479:     for (uint8_t index = 0; index <= 6; index++) {
2480:         pq = pq * (T(25.0) / (x * x)) + PQ[index];
2481:     }
2482: 
2483:     T qp = 0.0;
2484: 
2485:     for (uint8_t index = 0; index <= 7; index++) {
2486:         qp = qp * (T(25.0) / (x * x)) + QP[index];
2487:     }
2488: 
2489:     T qq = 0.0;
2490: 
```
- **EN**: Lines 2461-2490 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: T.
- **CN**: 第 2461-2490 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：T。

### Lines 2491-2520 / 第 2491-2520 行
```cpp
2491:     for (uint8_t index = 0; index <= 6; index++) {
2492:         qq = qq * (T(25.0) / (x * x)) + QQ[index];
2493:     }
2494: 
2495:     return (pp / pq * std::cos(x - T(0.785398163397448309615660845819875721)) - T(5.0) / x * (qp / qq) * std::sin(x - T(0.785398163397448309615660845819875721))) * T(0.797884560802865355879892119868763737) / std::sqrt(x);
2496: } // bessel_j0_forward(T x)
2497: 
2498: template<typename T>
2499: inline C10_HOST_DEVICE T bessel_j1_forward(T x) {
2500:     static const T PP[] = {
2501:             +7.62125616208173112003e-04,
2502:             +7.31397056940917570436e-02,
2503:             +1.12719608129684925192e+00,
2504:             +5.11207951146807644818e+00,
2505:             +8.42404590141772420927e+00,
2506:             +5.21451598682361504063e+00,
2507:             +1.00000000000000000254e+00,
2508:     };
2509: 
2510:     static const T PQ[] = {
2511:             +5.71323128072548699714e-04,
2512:             +6.88455908754495404082e-02,
2513:             +1.10514232634061696926e+00,
2514:             +5.07386386128601488557e+00,
2515:             +8.39985554327604159757e+00,
2516:             +5.20982848682361821619e+00,
2517:             +9.99999999999999997461e-01,
2518:     };
2519: 
2520:     static const T QP[] = {
```
- **EN**: Lines 2491-2520 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T, cos, sin, sqrt.
- **CN**: 第 2491-2520 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T, cos, sin, sqrt。

### Lines 2521-2550 / 第 2521-2550 行
```cpp
2521:             +5.10862594750176621635e-02,
2522:             +4.98213872951233449420e+00,
2523:             +7.58238284132545283818e+01,
2524:             +3.66779609360150777800e+02,
2525:             +7.10856304998926107277e+02,
2526:             +5.97489612400613639965e+02,
2527:             +2.11688757100572135698e+02,
2528:             +2.52070205858023719784e+01,
2529:     };
2530: 
2531:     static const T QQ[] = {
2532:             +7.42373277035675149943e+01,
2533:             +1.05644886038262816351e+03,
2534:             +4.98641058337653607651e+03,
2535:             +9.56231892404756170795e+03,
2536:             +7.99704160447350683650e+03,
2537:             +2.82619278517639096600e+03,
2538:             +3.36093607810698293419e+02,
2539:     };
2540: 
2541:     static const T RP[] = {
2542:             -8.99971225705559398224e+08,
2543:             +4.52228297998194034323e+11,
2544:             -7.27494245221818276015e+13,
2545:             +3.68295732863852883286e+15,
2546:     };
2547: 
2548:     static const T RQ[] = {
2549:             +6.20836478118054335476e+02,
2550:             +2.56987256757748830383e+05,
```
- **EN**: Lines 2521-2550 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 2521-2550 行主要涉及表达式或调用、变量/别名声明。

### Lines 2551-2580 / 第 2551-2580 行
```cpp
2551:             +8.35146791431949253037e+07,
2552:             +2.21511595479792499675e+10,
2553:             +4.74914122079991414898e+12,
2554:             +7.84369607876235854894e+14,
2555:             +8.95222336184627338078e+16,
2556:             +5.32278620332680085395e+18,
2557:     };
2558: 
2559:     if (x < T(0.0)) {
2560:         return -bessel_j1_forward(-x);
2561:     }
2562: 
2563:     if (x <= T(5.0)) {
2564:         T rp = 0.0;
2565: 
2566:         for (uint8_t index = 0; index <= 3; index++) {
2567:             rp = rp * (x * x) + RP[index];
2568:         }
2569: 
2570:         T rq = 0.0;
2571: 
2572:         for (uint8_t index = 0; index <= 7; index++) {
2573:             rq = rq * (x * x) + RQ[index];
2574:         }
2575: 
2576:         return rp / rq * x * (x * x - T(1.46819706421238932572e+01)) * (x * x - T(4.92184563216946036703e+01));
2577:     }
2578: 
2579:     T pp = 0.0;
2580: 
```
- **EN**: Lines 2551-2580 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T, bessel_j1_forward.
- **CN**: 第 2551-2580 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T, bessel_j1_forward。

### Lines 2581-2610 / 第 2581-2610 行
```cpp
2581:     for (uint8_t index = 0; index <= 6; index++) {
2582:         pp = pp * (T(5.0) / x * (T(5.0) / x)) + PP[index];
2583:     }
2584: 
2585:     T pq = 0.0;
2586: 
2587:     for (uint8_t index = 0; index <= 6; index++) {
2588:         pq = pq * (T(5.0) / x * (T(5.0) / x)) + PQ[index];
2589:     }
2590: 
2591:     T qp = 0.0;
2592: 
2593:     for (uint8_t index = 0; index <= 7; index++) {
2594:         qp = qp * (T(5.0) / x * (T(5.0) / x)) + QP[index];
2595:     }
2596: 
2597:     T qq = 0.0;
2598: 
2599:     for (uint8_t index = 0; index <= 6; index++) {
2600:         qq = qq * (T(5.0) / x * (T(5.0) / x)) + QQ[index];
2601:     }
2602: 
2603:     return (pp / pq * std::cos(x - T(2.356194490192344928846982537459627163)) - T(5.0) / x * (qp / qq) * std::sin(x - T(2.356194490192344928846982537459627163))) * T(0.797884560802865355879892119868763737) / std::sqrt(x);
2604: } // bessel_j1_forward(T x)
2605: 
2606: template<typename T>
2607: inline C10_HOST_DEVICE T bessel_y0_forward(T x) {
2608:     static const T PP[] = {
2609:             +7.96936729297347051624e-04,
2610:             +8.28352392107440799803e-02,
```
- **EN**: Lines 2581-2610 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: T, cos, sin, sqrt.
- **CN**: 第 2581-2610 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：T, cos, sin, sqrt。

### Lines 2611-2640 / 第 2611-2640 行
```cpp
2611:             +1.23953371646414299388e+00,
2612:             +5.44725003058768775090e+00,
2613:             +8.74716500199817011941e+00,
2614:             +5.30324038235394892183e+00,
2615:             +9.99999999999999997821e-01,
2616:     };
2617: 
2618:     static const T PQ[] = {
2619:             +9.24408810558863637013e-04,
2620:             +8.56288474354474431428e-02,
2621:             +1.25352743901058953537e+00,
2622:             +5.47097740330417105182e+00,
2623:             +8.76190883237069594232e+00,
2624:             +5.30605288235394617618e+00,
2625:             +1.00000000000000000218e+00,
2626:     };
2627: 
2628:     static const T QP[] = {
2629:             -1.13663838898469149931e-02,
2630:             -1.28252718670509318512e+00,
2631:             -1.95539544257735972385e+01,
2632:             -9.32060152123768231369e+01,
2633:             -1.77681167980488050595e+02,
2634:             -1.47077505154951170175e+02,
2635:             -5.14105326766599330220e+01,
2636:             -6.05014350600728481186e+00,
2637:     };
2638: 
2639:     static const T QQ[] = {
2640:             +6.43178256118178023184e+01,
```
- **EN**: Lines 2611-2640 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 2611-2640 行主要涉及表达式或调用、变量/别名声明。

### Lines 2641-2670 / 第 2641-2670 行
```cpp
2641:             +8.56430025976980587198e+02,
2642:             +3.88240183605401609683e+03,
2643:             +7.24046774195652478189e+03,
2644:             +5.93072701187316984827e+03,
2645:             +2.06209331660327847417e+03,
2646:             +2.42005740240291393179e+02,
2647:     };
2648: 
2649:     static const T YP[] = {
2650:             +1.55924367855235737965e+04,
2651:             -1.46639295903971606143e+07,
2652:             +5.43526477051876500413e+09,
2653:             -9.82136065717911466409e+11,
2654:             +8.75906394395366999549e+13,
2655:             -3.46628303384729719441e+15,
2656:             +4.42733268572569800351e+16,
2657:             -1.84950800436986690637e+16,
2658:     };
2659: 
2660:     static const T YQ[] = {
2661:             +1.04128353664259848412e+03,
2662:             +6.26107330137134956842e+05,
2663:             +2.68919633393814121987e+08,
2664:             +8.64002487103935000337e+10,
2665:             +2.02979612750105546709e+13,
2666:             +3.17157752842975028269e+15,
2667:             +2.50596256172653059228e+17,
2668:     };
2669: 
2670:     if (x <= T(5.0)) {
```
- **EN**: Lines 2641-2670 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T.
- **CN**: 第 2641-2670 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T。

### Lines 2671-2700 / 第 2671-2700 行
```cpp
2671:         if (x == T(0.0)) {
2672:             return -std::numeric_limits<T>::infinity();
2673:         }
2674: 
2675:         if (x < T(0.0)) {
2676:             return std::numeric_limits<T>::quiet_NaN();
2677:         }
2678: 
2679:         T yp = 0.0;
2680: 
2681:         for (uint8_t index = 0; index <= 7; index++) {
2682:             yp = yp * (x * x) + YP[index];
2683:         }
2684: 
2685:         T yq = 0.0;
2686: 
2687:         for (uint8_t index = 0; index <= 6; index++) {
2688:             yq = yq * (x * x) + YQ[index];
2689:         }
2690: 
2691:         return yp / yq + (T(0.636619772367581343075535053490057448) * std::log(x) * bessel_j0_forward(x));
2692:     }
2693: 
2694:     T pp = 0.0;
2695: 
2696:     for (uint8_t index = 0; index <= 6; index++) {
2697:         pp = pp * (T(25.0) / (x * x)) + PP[index];
2698:     }
2699: 
2700:     T pq = 0.0;
```
- **EN**: Lines 2671-2700 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: T, infinity, quiet_NaN, log.
- **CN**: 第 2671-2700 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：T, infinity, quiet_NaN, log。

### Lines 2701-2730 / 第 2701-2730 行
```cpp
2701: 
2702:     for (uint8_t index = 0; index <= 6; index++) {
2703:         pq = pq * (T(25.0) / (x * x)) + PQ[index];
2704:     }
2705: 
2706:     T qp = 0.0;
2707: 
2708:     for (uint8_t index = 0; index <= 7; index++) {
2709:         qp = qp * (T(25.0) / (x * x)) + QP[index];
2710:     }
2711: 
2712:     T qq = 0.0;
2713: 
2714:     for (uint8_t index = 0; index <= 6; index++) {
2715:         qq = qq * (T(25.0) / (x * x)) + QQ[index];
2716:     }
2717: 
2718:     return (pp / pq * std::sin(x - T(0.785398163397448309615660845819875721)) + T(5.0) / x * (qp / qq) * std::cos(x - T(0.785398163397448309615660845819875721))) * T(0.797884560802865355879892119868763737) / std::sqrt(x);
2719: } // bessel_y0_forward(T x)
2720: 
2721: template<typename T>
2722: inline C10_HOST_DEVICE T bessel_y1_forward(T x) {
2723:     static const T PP[] = {
2724:             +7.62125616208173112003e-04,
2725:             +7.31397056940917570436e-02,
2726:             +1.12719608129684925192e+00,
2727:             +5.11207951146807644818e+00,
2728:             +8.42404590141772420927e+00,
2729:             +5.21451598682361504063e+00,
2730:             +1.00000000000000000254e+00,
```
- **EN**: Lines 2701-2730 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T, sin, cos, sqrt.
- **CN**: 第 2701-2730 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T, sin, cos, sqrt。

### Lines 2731-2760 / 第 2731-2760 行
```cpp
2731:     };
2732: 
2733:     static const T PQ[] = {
2734:             +5.71323128072548699714e-04,
2735:             +6.88455908754495404082e-02,
2736:             +1.10514232634061696926e+00,
2737:             +5.07386386128601488557e+00,
2738:             +8.39985554327604159757e+00,
2739:             +5.20982848682361821619e+00,
2740:             +9.99999999999999997461e-01,
2741:     };
2742: 
2743:     static const T QP[] = {
2744:             +5.10862594750176621635e-02,
2745:             +4.98213872951233449420e+00,
2746:             +7.58238284132545283818e+01,
2747:             +3.66779609360150777800e+02,
2748:             +7.10856304998926107277e+02,
2749:             +5.97489612400613639965e+02,
2750:             +2.11688757100572135698e+02,
2751:             +2.52070205858023719784e+01,
2752:     };
2753: 
2754:     static const T QQ[] = {
2755:             +7.42373277035675149943e+01,
2756:             +1.05644886038262816351e+03,
2757:             +4.98641058337653607651e+03,
2758:             +9.56231892404756170795e+03,
2759:             +7.99704160447350683650e+03,
2760:             +2.82619278517639096600e+03,
```
- **EN**: Lines 2731-2760 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 2731-2760 行主要涉及表达式或调用、变量/别名声明。

### Lines 2761-2790 / 第 2761-2790 行
```cpp
2761:             +3.36093607810698293419e+02,
2762:     };
2763: 
2764:     static const T YP[] = {
2765:             +1.26320474790178026440e+09,
2766:             -6.47355876379160291031e+11,
2767:             +1.14509511541823727583e+14,
2768:             -8.12770255501325109621e+15,
2769:             +2.02439475713594898196e+17,
2770:             -7.78877196265950026825e+17,
2771:     };
2772: 
2773:     static const T YQ[] = {
2774:             +5.94301592346128195359e+02,
2775:             +2.35564092943068577943e+05,
2776:             +7.34811944459721705660e+07,
2777:             +1.87601316108706159478e+10,
2778:             +3.88231277496238566008e+12,
2779:             +6.20557727146953693363e+14,
2780:             +6.87141087355300489866e+16,
2781:             +3.97270608116560655612e+18,
2782:     };
2783: 
2784:     if (x <= T(5.0)) {
2785:         if (x == T(0.0)) {
2786:             return -std::numeric_limits<T>::infinity();
2787:         }
2788: 
2789:         if (x <= T(0.0)) {
2790:             return std::numeric_limits<T>::quiet_NaN();
```
- **EN**: Lines 2761-2790 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T, infinity, quiet_NaN.
- **CN**: 第 2761-2790 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T, infinity, quiet_NaN。

### Lines 2791-2820 / 第 2791-2820 行
```cpp
2791:         }
2792: 
2793:         T yp = 0.0;
2794: 
2795:         for (uint8_t index = 0; index <= 5; index++) {
2796:             yp = yp * (x * x) + YP[index];
2797:         }
2798: 
2799:         T yq = 0.0;
2800: 
2801:         for (uint8_t index = 0; index <= 7; index++) {
2802:             yq = yq * (x * x) + YQ[index];
2803:         }
2804: 
2805:         return x * (yp / yq) + (T(0.636619772367581343075535053490057448) * (bessel_j1_forward(x) * std::log(x) - T(1.0) / x));
2806:     }
2807: 
2808:     T pp = 0.0;
2809: 
2810:     for (uint8_t index = 0; index <= 6; index++) {
2811:         pp = pp * (T(5.0) / x * (T(5.0) / x)) + PP[index];
2812:     }
2813: 
2814:     T pq = 0.0;
2815: 
2816:     for (uint8_t index = 0; index <= 6; index++) {
2817:         pq = pq * (T(5.0) / x * (T(5.0) / x)) + PQ[index];
2818:     }
2819: 
2820:     T qp = 0.0;
```
- **EN**: Lines 2791-2820 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: T, bessel_j1_forward, log.
- **CN**: 第 2791-2820 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：T, bessel_j1_forward, log。

### Lines 2821-2850 / 第 2821-2850 行
```cpp
2821: 
2822:     for (uint8_t index = 0; index <= 7; index++) {
2823:         qp = qp * (T(5.0) / x * (T(5.0) / x)) + QP[index];
2824:     }
2825: 
2826:     T qq = 0.0;
2827: 
2828:     for (uint8_t index = 0; index <= 6; index++) {
2829:         qq = qq * (T(5.0) / x * (T(5.0) / x)) + QQ[index];
2830:     }
2831: 
2832:     return (pp / pq * std::sin(x - T(2.356194490192344928846982537459627163)) + T(5.0) / x * (qp / qq) * std::cos(x - T(2.356194490192344928846982537459627163))) * T(0.797884560802865355879892119868763737) / std::sqrt(x);
2833: } // bessel_y1_forward(T x)
2834: 
2835: template<typename T>
2836: inline C10_HOST_DEVICE T chebyshev_polynomial_t_forward(T x, int64_t n) {
2837:     if (n < 0) {
2838:         return T(0.0);
2839:     }
2840: 
2841:     if (std::abs(x) == T(1.0)) {
2842:         if (x > T(0.0) || n % 2 == 0) {
2843:             return T(1.0);
2844:         }
2845: 
2846:         return T(-1.0);
2847:     }
2848: 
2849:     if ((n > 6) && (std::abs(x) < T(1.0))) {
2850:         return std::cos(n * std::acos(x));
```
- **EN**: Lines 2821-2850 mainly cover control-flow checks, expressions/calls, return paths. Notable symbols: T, sin, cos, sqrt.
- **CN**: 第 2821-2850 行主要涉及控制流逻辑、表达式或调用、返回路径。 值得关注的符号包括：T, sin, cos, sqrt。

### Lines 2851-2880 / 第 2851-2880 行
```cpp
2851:     }
2852: 
2853:     if (n == 0) {
2854:         return T(1.0);
2855:     }
2856: 
2857:     if (n == 1) {
2858:         return x;
2859:     }
2860: 
2861:     T p = T(1.0);
2862:     T q = x;
2863:     T r;
2864: 
2865:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
2866:         r = (x + x) * q - p;
2867:         p = q;
2868:         q = r;
2869:     }
2870: 
2871:     return r;
2872: } // chebyshev_polynomial_t_forward(T x, int64_t n)
2873: 
2874: template<typename T, bool is_cuda=false>
2875: inline C10_HOST_DEVICE T chebyshev_polynomial_t_forward(T x, T n) {
2876:     return chebyshev_polynomial_t_forward(x, static_cast<int64_t>(n));
2877: } // chebyshev_polynomial_t_forward(T x, T n)
2878: 
2879: template<typename T>
2880: inline C10_HOST_DEVICE T chebyshev_polynomial_u_forward(T x, int64_t n) {
```
- **EN**: Lines 2851-2880 mainly cover state/variable declarations, expressions/calls, return paths. Notable symbols: T, isnan, chebyshev_polynomial_t_forward, chebyshev_polynomial_u_forward.
- **CN**: 第 2851-2880 行主要涉及变量/别名声明、表达式或调用、返回路径。 值得关注的符号包括：T, isnan, chebyshev_polynomial_t_forward, chebyshev_polynomial_u_forward。

### Lines 2881-2910 / 第 2881-2910 行
```cpp
2881:     if (n < 0) {
2882:         return T(0.0);
2883:     }
2884: 
2885:     if (std::abs(x) == T(1.0)) {
2886:         if (x > T(0.0) || n % 2 == 0) {
2887:             return n + 1;
2888:         }
2889: 
2890:         return -(n + 1);
2891:     }
2892: 
2893:     if ((n > 8) && (std::abs(x) < T(1.0))) {
2894:         if (std::sin(std::acos(x)) != T(0.0)) {
2895:             return std::sin((n + 1) * std::acos(x)) / std::sin(std::acos(x));
2896:         }
2897: 
2898:         return (n + 1) * std::cos((n + 1) * std::acos(x)) / x;
2899:     }
2900: 
2901:     if (n == 0) {
2902:         return T(1.0);
2903:     }
2904: 
2905:     if (n == 1) {
2906:         return x + x;
2907:     }
2908: 
2909:     T p = T(1.0);
2910:     T q = x + x;
```
- **EN**: Lines 2881-2910 mainly cover control-flow checks, return paths, expressions/calls. Notable symbols: T, abs, sin, acos.
- **CN**: 第 2881-2910 行主要涉及控制流逻辑、返回路径、表达式或调用。 值得关注的符号包括：T, abs, sin, acos。

### Lines 2911-2940 / 第 2911-2940 行
```cpp
2911:     T r;
2912: 
2913:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
2914:         r = (x + x) * q - p;
2915:         p = q;
2916:         q = r;
2917:     }
2918: 
2919:     return r;
2920: } // chebyshev_polynomial_u_forward(T x, int64_t n)
2921: 
2922: template<typename T, bool is_cuda=false>
2923: inline C10_HOST_DEVICE T chebyshev_polynomial_u_forward(T x, T n) {
2924:     return chebyshev_polynomial_u_forward(x, static_cast<int64_t>(n));
2925: } // chebyshev_polynomial_u_forward(T x, T n)
2926: 
2927: template<typename T>
2928: inline C10_HOST_DEVICE T chebyshev_polynomial_v_forward(T x, int64_t n) {
2929:     if (n < 0) {
2930:         return T(0.0);
2931:     }
2932: 
2933:     if (std::abs(x) == T(1.0)) {
2934:         if (x > T(0.0)) {
2935:             return T(1.0);
2936:         }
2937: 
2938:         if (n % 2 == 0) {
2939:             return n + n + 1;
2940:         }
```
- **EN**: Lines 2911-2940 mainly cover state/variable declarations, control-flow checks, return paths. Notable symbols: isnan, chebyshev_polynomial_u_forward, chebyshev_polynomial_v_forward, T.
- **CN**: 第 2911-2940 行主要涉及变量/别名声明、控制流逻辑、返回路径。 值得关注的符号包括：isnan, chebyshev_polynomial_u_forward, chebyshev_polynomial_v_forward, T。

### Lines 2941-2970 / 第 2941-2970 行
```cpp
2941: 
2942:         return -(n + n + 1);
2943:     }
2944: 
2945:     if ((n > 8) && (std::abs(x) < T(1.0))) {
2946:         if (std::sin(std::acos(x) / T(2.0)) != T(1.0)) {
2947:             return std::cos((n + T(0.5)) * std::acos(x)) / std::cos(std::acos(x) / T(2.0));
2948:         }
2949: 
2950:         if (n % 2 == 0) {
2951:             return n + n + 1;
2952:         }
2953: 
2954:         return -(n + n + 1);
2955:     }
2956: 
2957:     if (n == 0) {
2958:         return T(1.0);
2959:     }
2960: 
2961:     if (n == 1) {
2962:         return x + x - T(1.0);
2963:     }
2964: 
2965:     T p = T(1.0);
2966:     T q = x + x - T(1.0);
2967:     T r;
2968: 
2969:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
2970:         r = (x + x) * q - p;
```
- **EN**: Lines 2941-2970 mainly cover return paths, expressions/calls, control-flow checks. Notable symbols: abs, T, sin, acos.
- **CN**: 第 2941-2970 行主要涉及返回路径、表达式或调用、控制流逻辑。 值得关注的符号包括：abs, T, sin, acos。

### Lines 2971-3000 / 第 2971-3000 行
```cpp
2971:         p = q;
2972:         q = r;
2973:     }
2974: 
2975:     return r;
2976: } // chebyshev_polynomial_v_forward(T x, int64_t n)
2977: 
2978: template<typename T, bool is_cuda=false>
2979: inline C10_HOST_DEVICE T chebyshev_polynomial_v_forward(T x, T n) {
2980:     return chebyshev_polynomial_v_forward(x, static_cast<int64_t>(n));
2981: } // chebyshev_polynomial_v_forward(T x, T n)
2982: 
2983: template<typename T>
2984: inline C10_HOST_DEVICE T chebyshev_polynomial_w_forward(T x, int64_t n) {
2985:     if (n < 0) {
2986:         return T(0.0);
2987:     }
2988: 
2989:     if (std::abs(x) == T(1.0)) {
2990:         if (x > T(0.0)) {
2991:             return n + n + 1;
2992:         }
2993: 
2994:         if (n % 2 == 0) {
2995:             return T(1.0);
2996:         }
2997: 
2998:         return T(-1.0);
2999:     }
3000: 
```
- **EN**: Lines 2971-3000 mainly cover return paths, expressions/calls, state/variable declarations. Notable symbols: chebyshev_polynomial_v_forward, chebyshev_polynomial_w_forward, T, abs.
- **CN**: 第 2971-3000 行主要涉及返回路径、表达式或调用、变量/别名声明。 值得关注的符号包括：chebyshev_polynomial_v_forward, chebyshev_polynomial_w_forward, T, abs。

### Lines 3001-3030 / 第 3001-3030 行
```cpp
3001:     if ((n > 8) && (std::abs(x) < T(1.0))) {
3002:         if (std::cos(std::acos(x) / T(2.0)) != T(1.0)) {
3003:             return std::sin((n + T(0.5)) * std::acos(x)) / std::sin(std::acos(x) / T(2.0));
3004:         }
3005: 
3006:         if (x > T(0.0)) {
3007:             return n + n + 1;
3008:         }
3009: 
3010:         if (n % 2 == 0) {
3011:             return T(1.0);
3012:         }
3013: 
3014:         return T(-1.0);
3015:     }
3016: 
3017:     if (n == 0) {
3018:         return T(1.0);
3019:     }
3020: 
3021:     if (n == 1) {
3022:         return x + x + T(1.0);
3023:     }
3024: 
3025:     T p = T(1.0);
3026:     T q = x + x + T(1.0);
3027:     T r;
3028: 
3029:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
3030:         r = (x + x) * q - p;
```
- **EN**: Lines 3001-3030 mainly cover control-flow checks, return paths, expressions/calls. Notable symbols: abs, T, cos, acos.
- **CN**: 第 3001-3030 行主要涉及控制流逻辑、返回路径、表达式或调用。 值得关注的符号包括：abs, T, cos, acos。

### Lines 3031-3060 / 第 3031-3060 行
```cpp
3031:         p = q;
3032:         q = r;
3033:     }
3034: 
3035:     return r;
3036: } // chebyshev_polynomial_w_forward(T x, int64_t n)
3037: 
3038: template<typename T, bool is_cuda=false>
3039: inline C10_HOST_DEVICE T chebyshev_polynomial_w_forward(T x, T n) {
3040:     return chebyshev_polynomial_w_forward(x, static_cast<int64_t>(n));
3041: } // chebyshev_polynomial_w_forward(T x, T n)
3042: 
3043: template<typename T>
3044: constexpr auto getHermitianLimit() {
3045:     if constexpr (std::is_same_v<T, float>) {
3046:         return 128;
3047:     } else if constexpr (std::is_same_v<T, double>) {
3048:         return 512;
3049:     } else {
3050:         return 1024;
3051:     }
3052: }
3053: 
3054: template<typename T>
3055: inline C10_HOST_DEVICE T hermite_polynomial_h_forward(T x, int64_t n) {
3056:     if (n < 0) {
3057:         return T(0.0);
3058:     }
3059: 
3060:     if (n == 0) {
```
- **EN**: Lines 3031-3060 mainly cover return paths, state/variable declarations, expressions/calls. Notable symbols: chebyshev_polynomial_w_forward, getHermitianLimit, constexpr, hermite_polynomial_h_forward.
- **CN**: 第 3031-3060 行主要涉及返回路径、变量/别名声明、表达式或调用。 值得关注的符号包括：chebyshev_polynomial_w_forward, getHermitianLimit, constexpr, hermite_polynomial_h_forward。

### Lines 3061-3090 / 第 3061-3090 行
```cpp
3061:         return T(1.0);
3062:     }
3063: 
3064:     if (n == 1) {
3065:         return x + x;
3066:     }
3067: 
3068:     if (n > getHermitianLimit<T>()) {
3069:         return std::numeric_limits<T>::quiet_NaN();
3070:     }
3071: 
3072:     T p = T(1.0);
3073:     T q = x + x;
3074:     T r = T(0.0);
3075: 
3076:     for (int64_t k = 2; k < n + n; k += 2) {
3077:         r = (x + x) * q - k * p;
3078:         p = q;
3079:         q = r;
3080:     }
3081: 
3082:     return r;
3083: } // hermite_polynomial_h_forward(T x, int64_t n)
3084: 
3085: template<typename T, bool is_cuda=false, std::enable_if_t<!std::is_floating_point_v<T>, int> = 0>
3086: inline C10_HOST_DEVICE T hermite_polynomial_h_forward(T x, T n) {
3087:     return hermite_polynomial_h_forward(x, static_cast<int64_t>(n));
3088: } // hermite_polynomial_h_forward(T x, T n)
3089: 
3090: template<typename T, bool is_cuda=false, std::enable_if_t<std::is_floating_point_v<T>, int> = 0>
```
- **EN**: Lines 3061-3090 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: T, quiet_NaN, hermite_polynomial_h_forward.
- **CN**: 第 3061-3090 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：T, quiet_NaN, hermite_polynomial_h_forward。

### Lines 3091-3120 / 第 3091-3120 行
```cpp
3091: __ubsan_ignore_float_cast_overflow__ inline C10_HOST_DEVICE T hermite_polynomial_h_forward(T x, T n) {
3092:     return hermite_polynomial_h_forward(x, (!std::isinf(n) && !std::isnan(n)) ? static_cast<int64_t>(n) : static_cast<int64_t>(-1));
3093: } // hermite_polynomial_h_forward(T x, T n)
3094: 
3095: template<typename T>
3096: inline C10_HOST_DEVICE T hermite_polynomial_he_forward(T x, int64_t n) {
3097:     if (n < 0) {
3098:         return T(0.0);
3099:     }
3100: 
3101:     if (n == 0) {
3102:         return T(1.0);
3103:     }
3104: 
3105:     if (n == 1) {
3106:         return x;
3107:     }
3108: 
3109:     if (n > getHermitianLimit<T>()) {
3110:         return std::numeric_limits<T>::quiet_NaN();
3111:     }
3112: 
3113:     T p = T(1.0);
3114:     T q = x;
3115:     T r;
3116: 
3117:     for (int64_t k = 1; k < n; k++) {
3118:         r = x * q - k * p;
3119:         p = q;
3120:         q = r;
```
- **EN**: Lines 3091-3120 mainly cover state/variable declarations, return paths, control-flow checks. Notable symbols: hermite_polynomial_h_forward, isinf, isnan, hermite_polynomial_he_forward.
- **CN**: 第 3091-3120 行主要涉及变量/别名声明、返回路径、控制流逻辑。 值得关注的符号包括：hermite_polynomial_h_forward, isinf, isnan, hermite_polynomial_he_forward。

### Lines 3121-3150 / 第 3121-3150 行
```cpp
3121:     }
3122: 
3123:     return r;
3124: } // hermite_polynomial_he_forward(T x, int64_t n)
3125: 
3126: template<typename T, bool is_cuda=false>
3127: inline C10_HOST_DEVICE T hermite_polynomial_he_forward(T x, T n) {
3128:     return hermite_polynomial_he_forward(x, static_cast<int64_t>(n));
3129: } // hermite_polynomial_he_forward(T x, T n)
3130: 
3131: template<typename T>
3132: inline C10_HOST_DEVICE T laguerre_polynomial_l_forward(T x, int64_t n) {
3133:     if (n < 0) {
3134:         return T(0.0);
3135:     }
3136: 
3137:     if (std::abs(x) == T(0.0)) {
3138:         return T(1.0);
3139:     }
3140: 
3141:     if (n == 0) {
3142:         return T(1.0);
3143:     }
3144: 
3145:     if (n == 1) {
3146:         return T(1.0) - x;
3147:     }
3148: 
3149:     T p = T(1.0);
3150:     T q = T(1.0) - x;
```
- **EN**: Lines 3121-3150 mainly cover return paths, expressions/calls, state/variable declarations. Notable symbols: hermite_polynomial_he_forward, laguerre_polynomial_l_forward, T, abs.
- **CN**: 第 3121-3150 行主要涉及返回路径、表达式或调用、变量/别名声明。 值得关注的符号包括：hermite_polynomial_he_forward, laguerre_polynomial_l_forward, T, abs。

### Lines 3151-3180 / 第 3151-3180 行
```cpp
3151:     T r;
3152: 
3153:     for (int64_t k = 1; (k < n) && !std::isnan(q); k++) {
3154:         r = (((k + k) + (T(1.0) - x)) * q - k * p) / (k + 1);
3155:         p = q;
3156:         q = r;
3157:     }
3158: 
3159:     return r;
3160: } // laguerre_polynomial_l_forward(T x, int64_t n)
3161: 
3162: template<typename T, bool is_cuda=false>
3163: inline C10_HOST_DEVICE T laguerre_polynomial_l_forward(T x, T n) {
3164:     return laguerre_polynomial_l_forward(x, static_cast<int64_t>(n));
3165: } // laguerre_polynomial_l_forward(T x, T n)
3166: 
3167: template<typename T>
3168: inline C10_HOST_DEVICE T legendre_polynomial_p_forward(T x, int64_t n) {
3169:     if (n < 0) {
3170:         return T(0.0);
3171:     }
3172: 
3173:     if (std::abs(x) == T(1.0)) {
3174:         if (x > T(0.0) || n % 2 == 0) {
3175:             return T(1.0);
3176:         }
3177: 
3178:         return T(-1.0);
3179:     }
3180: 
```
- **EN**: Lines 3151-3180 mainly cover state/variable declarations, return paths, control-flow checks. Notable symbols: isnan, T, laguerre_polynomial_l_forward, legendre_polynomial_p_forward.
- **CN**: 第 3151-3180 行主要涉及变量/别名声明、返回路径、控制流逻辑。 值得关注的符号包括：isnan, T, laguerre_polynomial_l_forward, legendre_polynomial_p_forward。

### Lines 3181-3210 / 第 3181-3210 行
```cpp
3181:     if (n == 0) {
3182:         return T(1.0);
3183:     }
3184: 
3185:     if (n == 1) {
3186:         return x;
3187:     }
3188: 
3189:     T p = T(1.0);
3190:     T q = x;
3191:     T r;
3192: 
3193:     for (int64_t k = 1; (k < n) && !std::isnan(q); k++) {
3194:         r = ((k + k + 1) * x * q - k * p) / (k + 1);
3195:         p = q;
3196:         q = r;
3197:     }
3198: 
3199:     return r;
3200: } // legendre_polynomial_p_forward(T x, int64_t n)
3201: 
3202: template<typename T, bool is_cuda=false>
3203: inline C10_HOST_DEVICE T legendre_polynomial_p_forward(T x, T n) {
3204:     return legendre_polynomial_p_forward(x, static_cast<int64_t>(n));
3205: } // legendre_polynomial_p_forward(T x, T n)
3206: 
3207: template<typename T>
3208: inline C10_HOST_DEVICE T modified_bessel_i0_forward(T x) {
3209:     static const T A[] = {
3210:             -4.41534164647933937950e-18,
```
- **EN**: Lines 3181-3210 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: T, isnan, legendre_polynomial_p_forward, modified_bessel_i0_forward.
- **CN**: 第 3181-3210 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：T, isnan, legendre_polynomial_p_forward, modified_bessel_i0_forward。

### Lines 3211-3240 / 第 3211-3240 行
```cpp
3211:             +3.33079451882223809783e-17,
3212:             -2.43127984654795469359e-16,
3213:             +1.71539128555513303061e-15,
3214:             -1.16853328779934516808e-14,
3215:             +7.67618549860493561688e-14,
3216:             -4.85644678311192946090e-13,
3217:             +2.95505266312963983461e-12,
3218:             -1.72682629144155570723e-11,
3219:             +9.67580903537323691224e-11,
3220:             -5.18979560163526290666e-10,
3221:             +2.65982372468238665035e-09,
3222:             -1.30002500998624804212e-08,
3223:             +6.04699502254191894932e-08,
3224:             -2.67079385394061173391e-07,
3225:             +1.11738753912010371815e-06,
3226:             -4.41673835845875056359e-06,
3227:             +1.64484480707288970893e-05,
3228:             -5.75419501008210370398e-05,
3229:             +1.88502885095841655729e-04,
3230:             -5.76375574538582365885e-04,
3231:             +1.63947561694133579842e-03,
3232:             -4.32430999505057594430e-03,
3233:             +1.05464603945949983183e-02,
3234:             -2.37374148058994688156e-02,
3235:             +4.93052842396707084878e-02,
3236:             -9.49010970480476444210e-02,
3237:             +1.71620901522208775349e-01,
3238:             -3.04682672343198398683e-01,
3239:             +6.76795274409476084995e-01,
3240:     };
```
- **EN**: Lines 3211-3240 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 3211-3240 行主要涉及表达式或调用、变量/别名声明。

### Lines 3241-3270 / 第 3241-3270 行
```cpp
3241: 
3242:     static const T B[] = {
3243:             -7.23318048787475395456e-18,
3244:             -4.83050448594418207126e-18,
3245:             +4.46562142029675999901e-17,
3246:             +3.46122286769746109310e-17,
3247:             -2.82762398051658348494e-16,
3248:             -3.42548561967721913462e-16,
3249:             +1.77256013305652638360e-15,
3250:             +3.81168066935262242075e-15,
3251:             -9.55484669882830764870e-15,
3252:             -4.15056934728722208663e-14,
3253:             +1.54008621752140982691e-14,
3254:             +3.85277838274214270114e-13,
3255:             +7.18012445138366623367e-13,
3256:             -1.79417853150680611778e-12,
3257:             -1.32158118404477131188e-11,
3258:             -3.14991652796324136454e-11,
3259:             +1.18891471078464383424e-11,
3260:             +4.94060238822496958910e-10,
3261:             +3.39623202570838634515e-09,
3262:             +2.26666899049817806459e-08,
3263:             +2.04891858946906374183e-07,
3264:             +2.89137052083475648297e-06,
3265:             +6.88975834691682398426e-05,
3266:             +3.36911647825569408990e-03,
3267:             +8.04490411014108831608e-01,
3268:     };
3269: 
3270:     T p = T{0};
```
- **EN**: Lines 3241-3270 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 3241-3270 行主要涉及表达式或调用、变量/别名声明。

### Lines 3271-3300 / 第 3271-3300 行
```cpp
3271:     T q = 0.0;
3272: 
3273:     if (std::abs(x) <= T(8.0)) {
3274:         T a = A[0];
3275: 
3276:         for (uint8_t index = 1; index < 30; index++) {
3277:             p = q;
3278:             q = a;
3279:             a = ((std::abs(x) / T(2.0)) - T(2.0)) * q - p + A[index];
3280:         }
3281: 
3282:         return std::exp(std::abs(x)) * (T(0.5) * (a - p));
3283:     }
3284: 
3285:     T b = B[0];
3286: 
3287:     for (uint8_t index = 1; index < 25; index++) {
3288:         p = q;
3289:         q = b;
3290:         b = (T(32.0) / std::abs(x) - T(2.0)) * q - p + B[index];
3291:     }
3292: 
3293:     return std::exp(std::abs(x)) * (T(0.5) * (b - p)) / std::sqrt(std::abs(x));
3294: } // modified_bessel_i0_forward(T x)
3295: 
3296: template<typename T>
3297: inline C10_HOST_DEVICE T modified_bessel_i1_forward(T x) {
3298:     static const T A[] = {
3299:             +2.77791411276104639959e-18,
3300:             -2.11142121435816608115e-17,
```
- **EN**: Lines 3271-3300 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: abs, T, exp, sqrt.
- **CN**: 第 3271-3300 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：abs, T, exp, sqrt。

### Lines 3301-3330 / 第 3301-3330 行
```cpp
3301:             +1.55363195773620046921e-16,
3302:             -1.10559694773538630805e-15,
3303:             +7.60068429473540693410e-15,
3304:             -5.04218550472791168711e-14,
3305:             +3.22379336594557470981e-13,
3306:             -1.98397439776494371520e-12,
3307:             +1.17361862988909016308e-11,
3308:             -6.66348972350202774223e-11,
3309:             +3.62559028155211703701e-10,
3310:             -1.88724975172282928790e-09,
3311:             +9.38153738649577178388e-09,
3312:             -4.44505912879632808065e-08,
3313:             +2.00329475355213526229e-07,
3314:             -8.56872026469545474066e-07,
3315:             +3.47025130813767847674e-06,
3316:             -1.32731636560394358279e-05,
3317:             +4.78156510755005422638e-05,
3318:             -1.61760815825896745588e-04,
3319:             +5.12285956168575772895e-04,
3320:             -1.51357245063125314899e-03,
3321:             +4.15642294431288815669e-03,
3322:             -1.05640848946261981558e-02,
3323:             +2.47264490306265168283e-02,
3324:             -5.29459812080949914269e-02,
3325:             +1.02643658689847095384e-01,
3326:             -1.76416518357834055153e-01,
3327:             +2.52587186443633654823e-01,
3328:     };
3329: 
3330:     static const T B[] = {
```
- **EN**: Lines 3301-3330 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 3301-3330 行主要涉及表达式或调用、变量/别名声明。

### Lines 3331-3360 / 第 3331-3360 行
```cpp
3331:             +7.51729631084210481353e-18,
3332:             +4.41434832307170791151e-18,
3333:             -4.65030536848935832153e-17,
3334:             -3.20952592199342395980e-17,
3335:             +2.96262899764595013876e-16,
3336:             +3.30820231092092828324e-16,
3337:             -1.88035477551078244854e-15,
3338:             -3.81440307243700780478e-15,
3339:             +1.04202769841288027642e-14,
3340:             +4.27244001671195135429e-14,
3341:             -2.10154184277266431302e-14,
3342:             -4.08355111109219731823e-13,
3343:             -7.19855177624590851209e-13,
3344:             +2.03562854414708950722e-12,
3345:             +1.41258074366137813316e-11,
3346:             +3.25260358301548823856e-11,
3347:             -1.89749581235054123450e-11,
3348:             -5.58974346219658380687e-10,
3349:             -3.83538038596423702205e-09,
3350:             -2.63146884688951950684e-08,
3351:             -2.51223623787020892529e-07,
3352:             -3.88256480887769039346e-06,
3353:             -1.10588938762623716291e-04,
3354:             -9.76109749136146840777e-03,
3355:             +7.78576235018280120474e-01,
3356:     };
3357: 
3358:     T p = T{0};
3359:     T q = 0.0;
3360: 
```
- **EN**: Lines 3331-3360 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 3331-3360 行主要涉及表达式或调用、变量/别名声明。

### Lines 3361-3390 / 第 3361-3390 行
```cpp
3361:     if (std::abs(x) <= T(8.0)) {
3362:         T a = A[0];
3363: 
3364:         for (uint8_t index = 1; index < 29; index++) {
3365:             p = q;
3366:             q = a;
3367:             a = ((std::abs(x) / T(2.0)) - T(2.0)) * q - p + A[index];
3368:         }
3369: 
3370:         if (x < T(0.0)) {
3371:             return -(T(0.5) * (a - p) * std::abs(x) * std::exp(std::abs(x)));
3372:         }
3373: 
3374:         return T(0.5) * (a - p) * std::abs(x) * std::exp(std::abs(x));
3375:     }
3376: 
3377:     T b = B[0];
3378: 
3379:     for (uint8_t index = 1; index < 25; index++) {
3380:         p = q;
3381:         q = b;
3382:         b = (T(32.0) / std::abs(x) - T(2.0)) * q - p + B[index];
3383:     }
3384: 
3385:     if (x < T(0.0)) {
3386:         return -(std::exp(std::abs(x)) * (T(0.5) * (b - p)) / std::sqrt(std::abs(x)));
3387:     }
3388: 
3389:     return std::exp(std::abs(x)) * (T(0.5) * (b - p)) / std::sqrt(std::abs(x));
3390: } // modified_bessel_i1_forward(T x)
```
- **EN**: Lines 3361-3390 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: abs, T, exp, sqrt.
- **CN**: 第 3361-3390 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：abs, T, exp, sqrt。

### Lines 3391-3420 / 第 3391-3420 行
```cpp
3391: 
3392: template<typename T>
3393: inline C10_HOST_DEVICE T modified_bessel_k0_forward(T x) {
3394:     static const T A[] = {
3395:             +1.37446543561352307156e-16,
3396:             +4.25981614279661018399e-14,
3397:             +1.03496952576338420167e-11,
3398:             +1.90451637722020886025e-09,
3399:             +2.53479107902614945675e-07,
3400:             +2.28621210311945178607e-05,
3401:             +1.26461541144692592338e-03,
3402:             +3.59799365153615016266e-02,
3403:             +3.44289899924628486886e-01,
3404:             -5.35327393233902768720e-01,
3405:     };
3406: 
3407:     static const T B[] = {
3408:             +5.30043377268626276149e-18,
3409:             -1.64758043015242134646e-17,
3410:             +5.21039150503902756861e-17,
3411:             -1.67823109680541210385e-16,
3412:             +5.51205597852431940784e-16,
3413:             -1.84859337734377901440e-15,
3414:             +6.34007647740507060557e-15,
3415:             -2.22751332699166985548e-14,
3416:             +8.03289077536357521100e-14,
3417:             -2.98009692317273043925e-13,
3418:             +1.14034058820847496303e-12,
3419:             -4.51459788337394416547e-12,
3420:             +1.85594911495471785253e-11,
```
- **EN**: Lines 3391-3420 mainly cover expressions/calls, state/variable declarations, template setup. Notable symbols: modified_bessel_k0_forward.
- **CN**: 第 3391-3420 行主要涉及表达式或调用、变量/别名声明、模板声明。 值得关注的符号包括：modified_bessel_k0_forward。

### Lines 3421-3450 / 第 3421-3450 行
```cpp
3421:             -7.95748924447710747776e-11,
3422:             +3.57739728140030116597e-10,
3423:             -1.69753450938905987466e-09,
3424:             +8.57403401741422608519e-09,
3425:             -4.66048989768794782956e-08,
3426:             +2.76681363944501510342e-07,
3427:             -1.83175552271911948767e-06,
3428:             +1.39498137188764993662e-05,
3429:             -1.28495495816278026384e-04,
3430:             +1.56988388573005337491e-03,
3431:             -3.14481013119645005427e-02,
3432:             +2.44030308206595545468e+00,
3433:     };
3434: 
3435:     if (x == T(0.0)) {
3436:         return std::numeric_limits<T>::infinity();
3437:     }
3438: 
3439:     if (x < T(0.0)) {
3440:         return std::numeric_limits<T>::quiet_NaN();
3441:     }
3442: 
3443:     T p = T{0};
3444:     T q = 0.0;
3445: 
3446:     if (x <= T(2.0)) {
3447:         T a = A[0];
3448: 
3449:         for (uint8_t index = 1; index < 10; index++) {
3450:             p = q;
```
- **EN**: Lines 3421-3450 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T, infinity, quiet_NaN.
- **CN**: 第 3421-3450 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T, infinity, quiet_NaN。

### Lines 3451-3480 / 第 3451-3480 行
```cpp
3451:             q = a;
3452:             a = (x * x - T(2.0)) * q - p + A[index];
3453:         }
3454: 
3455:         return T(0.5) * (a - p) - std::log(0.5 * x) * modified_bessel_i0_forward(x);
3456:     }
3457: 
3458:     T b = B[0];
3459: 
3460:     for (uint8_t index = 1; index < 25; index++) {
3461:         p = q;
3462:         q = b;
3463:         b = (T(8.0) / x - T(2.0)) * q - p + B[index];
3464:     }
3465: 
3466:     return std::exp(-x) * (T(0.5) * (b - p)) / std::sqrt(x);
3467: } // modified_bessel_k0_forward(T x)
3468: 
3469: template<typename T>
3470: inline C10_HOST_DEVICE T modified_bessel_k1_forward(T x) {
3471:     static const T A[] = {
3472:             -7.02386347938628759343e-18,
3473:             -2.42744985051936593393e-15,
3474:             -6.66690169419932900609e-13,
3475:             -1.41148839263352776110e-10,
3476:             -2.21338763073472585583e-08,
3477:             -2.43340614156596823496e-06,
3478:             -1.73028895751305206302e-04,
3479:             -6.97572385963986435018e-03,
3480:             -1.22611180822657148235e-01,
```
- **EN**: Lines 3451-3480 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: T, log, modified_bessel_i0_forward, exp.
- **CN**: 第 3451-3480 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：T, log, modified_bessel_i0_forward, exp。

### Lines 3481-3510 / 第 3481-3510 行
```cpp
3481:             -3.53155960776544875667e-01,
3482:             +1.52530022733894777053e+00,
3483:     };
3484: 
3485:     static const T B[] = {
3486:             -5.75674448366501715755e-18,
3487:             +1.79405087314755922667e-17,
3488:             -5.68946255844285935196e-17,
3489:             +1.83809354436663880070e-16,
3490:             -6.05704724837331885336e-16,
3491:             +2.03870316562433424052e-15,
3492:             -7.01983709041831346144e-15,
3493:             +2.47715442448130437068e-14,
3494:             -8.97670518232499435011e-14,
3495:             +3.34841966607842919884e-13,
3496:             -1.28917396095102890680e-12,
3497:             +5.13963967348173025100e-12,
3498:             -2.12996783842756842877e-11,
3499:             +9.21831518760500529508e-11,
3500:             -4.19035475934189648750e-10,
3501:             +2.01504975519703286596e-09,
3502:             -1.03457624656780970260e-08,
3503:             +5.74108412545004946722e-08,
3504:             -3.50196060308781257119e-07,
3505:             +2.40648494783721712015e-06,
3506:             -1.93619797416608296024e-05,
3507:             +1.95215518471351631108e-04,
3508:             -2.85781685962277938680e-03,
3509:             +1.03923736576817238437e-01,
3510:             +2.72062619048444266945e+00,
```
- **EN**: Lines 3481-3510 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 3481-3510 行主要涉及表达式或调用、变量/别名声明。

### Lines 3511-3540 / 第 3511-3540 行
```cpp
3511:     };
3512: 
3513:     if (x == T(0.0)) {
3514:         return std::numeric_limits<T>::infinity();
3515:     }
3516: 
3517:     if (x < T(0.0)) {
3518:         return std::numeric_limits<T>::quiet_NaN();
3519:     }
3520: 
3521:     T p = T{0};
3522:     T q = 0.0;
3523: 
3524:     if (x <= T(2.0)) {
3525:         T a = A[0];
3526: 
3527:         for (uint8_t index = 1; index < 11; index++) {
3528:             p = q;
3529:             q = a;
3530:             a = (x * x - T(2.0)) * q - p + A[index];
3531:         }
3532: 
3533:         return std::log(T(0.5) * x) * modified_bessel_i1_forward(x) + T(0.5) * (a - p) / x;
3534:     }
3535: 
3536:     T b = B[0];
3537: 
3538:     for (uint8_t index = 1; index < 25; index++) {
3539:         p = q;
3540:         q = b;
```
- **EN**: Lines 3511-3540 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: T, infinity, quiet_NaN, log.
- **CN**: 第 3511-3540 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：T, infinity, quiet_NaN, log。

### Lines 3541-3570 / 第 3541-3570 行
```cpp
3541:         b = (T(8.0) / x - T(2.0)) * q - p + B[index];
3542:     }
3543: 
3544:     return std::exp(-x) * (T(0.5) * (b - p)) / std::sqrt(x);
3545: } // modified_bessel_k1_forward(T x)
3546: 
3547: template<typename T>
3548: inline C10_HOST_DEVICE T scaled_modified_bessel_k0_forward(T x) {
3549:     static const T A[] = {
3550:             +1.37446543561352307156e-16,
3551:             +4.25981614279661018399e-14,
3552:             +1.03496952576338420167e-11,
3553:             +1.90451637722020886025e-09,
3554:             +2.53479107902614945675e-07,
3555:             +2.28621210311945178607e-05,
3556:             +1.26461541144692592338e-03,
3557:             +3.59799365153615016266e-02,
3558:             +3.44289899924628486886e-01,
3559:             -5.35327393233902768720e-01,
3560:     };
3561: 
3562:     static const T B[] = {
3563:             +5.30043377268626276149e-18,
3564:             -1.64758043015242134646e-17,
3565:             +5.21039150503902756861e-17,
3566:             -1.67823109680541210385e-16,
3567:             +5.51205597852431940784e-16,
3568:             -1.84859337734377901440e-15,
3569:             +6.34007647740507060557e-15,
3570:             -2.22751332699166985548e-14,
```
- **EN**: Lines 3541-3570 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: T, exp, sqrt, modified_bessel_k1_forward.
- **CN**: 第 3541-3570 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：T, exp, sqrt, modified_bessel_k1_forward。

### Lines 3571-3600 / 第 3571-3600 行
```cpp
3571:             +8.03289077536357521100e-14,
3572:             -2.98009692317273043925e-13,
3573:             +1.14034058820847496303e-12,
3574:             -4.51459788337394416547e-12,
3575:             +1.85594911495471785253e-11,
3576:             -7.95748924447710747776e-11,
3577:             +3.57739728140030116597e-10,
3578:             -1.69753450938905987466e-09,
3579:             +8.57403401741422608519e-09,
3580:             -4.66048989768794782956e-08,
3581:             +2.76681363944501510342e-07,
3582:             -1.83175552271911948767e-06,
3583:             +1.39498137188764993662e-05,
3584:             -1.28495495816278026384e-04,
3585:             +1.56988388573005337491e-03,
3586:             -3.14481013119645005427e-02,
3587:             +2.44030308206595545468e+00,
3588:     };
3589: 
3590:     if (x == T(0.0)) {
3591:         return std::numeric_limits<T>::infinity();
3592:     }
3593: 
3594:     if (x < T(0.0)) {
3595:         return std::numeric_limits<T>::quiet_NaN();
3596:     }
3597: 
3598:     T p = T{0};
3599:     T q = 0.0;
3600: 
```
- **EN**: Lines 3571-3600 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T, infinity, quiet_NaN.
- **CN**: 第 3571-3600 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T, infinity, quiet_NaN。

### Lines 3601-3630 / 第 3601-3630 行
```cpp
3601:     if (x <= T(2.0)) {
3602:         T a = A[0];
3603: 
3604:         for (uint64_t index = 1; index < 10; index++) {
3605:             p = q;
3606:             q = a;
3607:             a = (x * x - T(2.0)) * q - p + A[index];
3608:         }
3609: 
3610:         return (T(0.5) * (a - p) - std::log(T(0.5) * x) * modified_bessel_i0_forward(x)) * std::exp(x);
3611:     }
3612: 
3613:     T b = B[0];
3614: 
3615:     for (uint64_t index = 1; index < 25; index++) {
3616:         p = q;
3617:         q = b;
3618:         b = (T(8.0) / x - T(2.0)) * q - p + B[index];
3619:     }
3620: 
3621:     return T(0.5) * (b - p) / std::sqrt(x);
3622: } // T scaled_modified_bessel_k0_forward(T x)
3623: 
3624: template<typename T>
3625: inline C10_HOST_DEVICE T scaled_modified_bessel_k1_forward(T x) {
3626:     static const T A[] = {
3627:             -7.02386347938628759343e-18,
3628:             -2.42744985051936593393e-15,
3629:             -6.66690169419932900609e-13,
3630:             -1.41148839263352776110e-10,
```
- **EN**: Lines 3601-3630 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: T, log, modified_bessel_i0_forward, exp.
- **CN**: 第 3601-3630 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：T, log, modified_bessel_i0_forward, exp。

### Lines 3631-3660 / 第 3631-3660 行
```cpp
3631:             -2.21338763073472585583e-08,
3632:             -2.43340614156596823496e-06,
3633:             -1.73028895751305206302e-04,
3634:             -6.97572385963986435018e-03,
3635:             -1.22611180822657148235e-01,
3636:             -3.53155960776544875667e-01,
3637:             +1.52530022733894777053e+00,
3638:     };
3639: 
3640:     static const T B[] = {
3641:             -5.75674448366501715755e-18,
3642:             +1.79405087314755922667e-17,
3643:             -5.68946255844285935196e-17,
3644:             +1.83809354436663880070e-16,
3645:             -6.05704724837331885336e-16,
3646:             +2.03870316562433424052e-15,
3647:             -7.01983709041831346144e-15,
3648:             +2.47715442448130437068e-14,
3649:             -8.97670518232499435011e-14,
3650:             +3.34841966607842919884e-13,
3651:             -1.28917396095102890680e-12,
3652:             +5.13963967348173025100e-12,
3653:             -2.12996783842756842877e-11,
3654:             +9.21831518760500529508e-11,
3655:             -4.19035475934189648750e-10,
3656:             +2.01504975519703286596e-09,
3657:             -1.03457624656780970260e-08,
3658:             +5.74108412545004946722e-08,
3659:             -3.50196060308781257119e-07,
3660:             +2.40648494783721712015e-06,
```
- **EN**: Lines 3631-3660 mainly cover expressions/calls, state/variable declarations.
- **CN**: 第 3631-3660 行主要涉及表达式或调用、变量/别名声明。

### Lines 3661-3690 / 第 3661-3690 行
```cpp
3661:             -1.93619797416608296024e-05,
3662:             +1.95215518471351631108e-04,
3663:             -2.85781685962277938680e-03,
3664:             +1.03923736576817238437e-01,
3665:             +2.72062619048444266945e+00,
3666:     };
3667: 
3668:     if (x == T(0.0)) {
3669:         return std::numeric_limits<T>::infinity();
3670:     }
3671: 
3672:     if (x < T(0.0)) {
3673:         return std::numeric_limits<T>::quiet_NaN();
3674:     }
3675: 
3676:     T p = T{0};
3677:     T q = 0.0;
3678: 
3679:     if (x <= T(2.0)) {
3680:         T a = A[0];
3681: 
3682:         for (uint64_t index = 1; index < 11; index++) {
3683:             p = q;
3684:             q = a;
3685:             a = (x * x - T(2.0)) * q - p + A[index];
3686:         }
3687: 
3688:         return (std::log(T(0.5) * x) * modified_bessel_i1_forward(x) + T(0.5) * (a - p) / x) * std::exp(x);
3689:     }
3690: 
```
- **EN**: Lines 3661-3690 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: T, infinity, quiet_NaN, log.
- **CN**: 第 3661-3690 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：T, infinity, quiet_NaN, log。

### Lines 3691-3720 / 第 3691-3720 行
```cpp
3691:     T b = B[0];
3692: 
3693:     for (uint64_t index = 1; index < 25; index++) {
3694:         p = q;
3695:         q = b;
3696:         b = (T(8.0) / x - T(2.0)) * q - p + B[index];
3697:     }
3698: 
3699:     return (T(0.5) * (b - p) / std::sqrt(x));
3700: } // T scaled_modified_bessel_k1_forward(T x)
3701: 
3702: template<typename T>
3703: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_t_forward(T x, int64_t n) {
3704:     if (n < 0) {
3705:         return T(0.0);
3706:     }
3707: 
3708:     if (x == T(1.0)) {
3709:         return T(1.0);
3710:     }
3711: 
3712:     if (x == T(0.0)) {
3713:         if (n % 2 == 0) {
3714:             return T(1.0);
3715:         }
3716: 
3717:         return T(-1.0);
3718:     }
3719: 
3720:     if ((n > 6) && (std::abs(x + x - T(1.0)) < T(1.0))) {
```
- **EN**: Lines 3691-3720 mainly cover control-flow checks, state/variable declarations, expressions/calls. Notable symbols: T, sqrt, scaled_modified_bessel_k1_forward, shifted_chebyshev_polynomial_t_forward.
- **CN**: 第 3691-3720 行主要涉及控制流逻辑、变量/别名声明、表达式或调用。 值得关注的符号包括：T, sqrt, scaled_modified_bessel_k1_forward, shifted_chebyshev_polynomial_t_forward。

### Lines 3721-3750 / 第 3721-3750 行
```cpp
3721:         return std::cos(n * std::acos(x + x - T(1.0)));
3722:     }
3723: 
3724:     if (n == 0) {
3725:         return T(1.0);
3726:     }
3727: 
3728:     if (n == 1) {
3729:         return x + x - T(1.0);
3730:     }
3731: 
3732:     T p = T(1.0);
3733:     T q = x + x - T(1.0);
3734:     T r;
3735: 
3736:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
3737:         r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
3738:         p = q;
3739:         q = r;
3740:     }
3741: 
3742:     return r;
3743: } // shifted_chebyshev_polynomial_t_forward(T x, int64_t n)
3744: 
3745: template<typename T, bool is_cuda=false>
3746: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_t_forward(T x, T n) {
3747:     return shifted_chebyshev_polynomial_t_forward(x, static_cast<int64_t>(n));
3748: } // shifted_chebyshev_polynomial_t_forward(T x, T n)
3749: 
3750: template<typename T>
```
- **EN**: Lines 3721-3750 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: cos, acos, T, isnan.
- **CN**: 第 3721-3750 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：cos, acos, T, isnan。

### Lines 3751-3780 / 第 3751-3780 行
```cpp
3751: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_u_forward(T x, int64_t n) {
3752:     if (n < 0) {
3753:         return T(0.0);
3754:     }
3755: 
3756:     if (x == T(1.0)) {
3757:         return n + 1;
3758:     }
3759: 
3760:     if (x == T(0.0)) {
3761:         if (n % 2 == 0) {
3762:             return n + 1;
3763:         }
3764: 
3765:         return -(n + 1);
3766:     }
3767: 
3768:     if ((n > 6) && (std::abs(x + x - T(1.0)) < T(1.0))) {
3769:         if (std::sin(std::acos(x + x - T(1.0))) != T(0.0)) {
3770:             return std::sin((n + 1) * std::acos(x + x - T(1.0))) / std::sin(std::acos(x + x - T(1.0)));
3771:         }
3772: 
3773:         return (n + 1) * std::cos((n + 1) * std::acos(x + x - T(1.0))) / (x + x - T(1.0));
3774:     }
3775: 
3776:     if (n == 0) {
3777:         return T(1.0);
3778:     }
3779: 
3780:     if (n == 1) {
```
- **EN**: Lines 3751-3780 mainly cover control-flow checks, return paths, expressions/calls. Notable symbols: shifted_chebyshev_polynomial_u_forward, T, abs, sin.
- **CN**: 第 3751-3780 行主要涉及控制流逻辑、返回路径、表达式或调用。 值得关注的符号包括：shifted_chebyshev_polynomial_u_forward, T, abs, sin。

### Lines 3781-3810 / 第 3781-3810 行
```cpp
3781:         return x + x - T(1.0) + (x + x - T(1.0));
3782:     }
3783: 
3784:     T p = T(1.0);
3785:     T q = x + x - T(1.0) + (x + x - T(1.0));
3786:     T r;
3787: 
3788:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
3789:         r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
3790:         p = q;
3791:         q = r;
3792:     }
3793: 
3794:     return r;
3795: } // shifted_chebyshev_polynomial_u_forward(T x, int64_t n)
3796: 
3797: template<typename T, bool is_cuda=false>
3798: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_u_forward(T x, T n) {
3799:     return shifted_chebyshev_polynomial_u_forward(x, static_cast<int64_t>(n));
3800: } // shifted_chebyshev_polynomial_u_forward(T x, T n)
3801: 
3802: template<typename T>
3803: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_v_forward(T x, int64_t n) {
3804:     if (n < 0) {
3805:         return T(0.0);
3806:     }
3807: 
3808:     if (x == T(1.0)) {
3809:         return T(1.0);
3810:     }
```
- **EN**: Lines 3781-3810 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: T, isnan, shifted_chebyshev_polynomial_u_forward, shifted_chebyshev_polynomial_v_forward.
- **CN**: 第 3781-3810 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：T, isnan, shifted_chebyshev_polynomial_u_forward, shifted_chebyshev_polynomial_v_forward。

### Lines 3811-3840 / 第 3811-3840 行
```cpp
3811: 
3812:     if (x == T(0.0)) {
3813:         if (n % 2 == 0) {
3814:             return (n + n + 1);
3815:         }
3816: 
3817:         return -(n + n + 1);
3818:     }
3819: 
3820:     if ((n > 6) && (std::abs(x + x - T(1.0)) < T(1.0))) {
3821:         if (std::sin(std::acos(x + x - T(1.0)) / T(2.0)) != T(1.0)) {
3822:             return std::cos((n + T(0.5)) * std::acos(x + x - T(1.0))) / std::cos(std::acos(x + x - T(1.0)) / T(2.0));
3823:         }
3824: 
3825:         if (n % 2 == 0) {
3826:             return n + n + 1;
3827:         }
3828: 
3829:         return -(n + n + 1);
3830:     }
3831: 
3832:     if (n == 0) {
3833:         return T(1.0);
3834:     }
3835: 
3836:     if (n == 1) {
3837:         return x + x - T(1.0) + (x + x - T(1.0)) - T(1.0);
3838:     }
3839: 
3840:     T p = T(1.0);
```
- **EN**: Lines 3811-3840 mainly cover control-flow checks, return paths, expressions/calls. Notable symbols: T, abs, sin, acos.
- **CN**: 第 3811-3840 行主要涉及控制流逻辑、返回路径、表达式或调用。 值得关注的符号包括：T, abs, sin, acos。

### Lines 3841-3870 / 第 3841-3870 行
```cpp
3841:     T q = x + x - T(1.0) + (x + x - T(1.0)) - T(1.0);
3842:     T r;
3843: 
3844:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
3845:         r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
3846:         p = q;
3847:         q = r;
3848:     }
3849: 
3850:     return r;
3851: } // shifted_chebyshev_polynomial_v_forward(T x, int64_t n)
3852: 
3853: template<typename T, bool is_cuda=false>
3854: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_v_forward(T x, T n) {
3855:     return shifted_chebyshev_polynomial_v_forward(x, static_cast<int64_t>(n));
3856: } // shifted_chebyshev_polynomial_v_forward(T x, T n)
3857: 
3858: template<typename T>
3859: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_w_forward(T x, int64_t n) {
3860:     if (n < 0) {
3861:         return T(0.0);
3862:     }
3863: 
3864:     if (x == T(1.0)) {
3865:         return n + n + 1;
3866:     }
3867: 
3868:     if (x == T(0.0)) {
3869:         if (n % 2 == 0) {
3870:             return T(1.0);
```
- **EN**: Lines 3841-3870 mainly cover state/variable declarations, control-flow checks, return paths. Notable symbols: T, isnan, shifted_chebyshev_polynomial_v_forward, shifted_chebyshev_polynomial_w_forward.
- **CN**: 第 3841-3870 行主要涉及变量/别名声明、控制流逻辑、返回路径。 值得关注的符号包括：T, isnan, shifted_chebyshev_polynomial_v_forward, shifted_chebyshev_polynomial_w_forward。

### Lines 3871-3900 / 第 3871-3900 行
```cpp
3871:         }
3872: 
3873:         return T(-1.0);
3874:     }
3875: 
3876:     if ((n > 4) && (std::abs(x + x - T(1.0)) < T(1.0))) {
3877:         if (std::cos(std::acos(x + x - T(1.0)) / T(2.0)) != T(1.0)) {
3878:             return std::sin((n + T(0.5)) * std::acos(x + x - T(1.0))) / std::sin(std::acos(x + x - T(1.0)) / T(2.0));
3879:         }
3880: 
3881:         if (n % 2 == 0) {
3882:             return T(1.0);
3883:         }
3884: 
3885:         return T(-1.0);
3886:     }
3887: 
3888:     if (n == 0) {
3889:         return T(1.0);
3890:     }
3891: 
3892:     if (n == 1) {
3893:         return x + x - T(1.0) + (x + x - T(1.0)) + T(1.0);
3894:     }
3895: 
3896:     T p = T(1.0);
3897:     T q = x + x - T(1.0) + (x + x - T(1.0)) + T(1.0);
3898:     T r;
3899: 
3900:     for (int64_t k = 2; (k <= n) && !std::isnan(q); k++) {
```
- **EN**: Lines 3871-3900 mainly cover expressions/calls, return paths, control-flow checks. Notable symbols: T, abs, cos, acos.
- **CN**: 第 3871-3900 行主要涉及表达式或调用、返回路径、控制流逻辑。 值得关注的符号包括：T, abs, cos, acos。

### Lines 3901-3927 / 第 3901-3927 行
```cpp
3901:         r = (x + x - T(1.0) + (x + x - T(1.0))) * q - p;
3902:         p = q;
3903:         q = r;
3904:     }
3905: 
3906:     return r;
3907: } // shifted_chebyshev_polynomial_w_forward(T x, int64_t n)
3908: 
3909: template<typename T, bool is_cuda=false>
3910: inline C10_HOST_DEVICE T shifted_chebyshev_polynomial_w_forward(T x, T n) {
3911:     return shifted_chebyshev_polynomial_w_forward(x, static_cast<int64_t>(n));
3912: } // shifted_chebyshev_polynomial_w_forward(T x, T n)
3913: 
3914: template<typename T>
3915: inline C10_HOST_DEVICE T spherical_bessel_j0_forward(T x) {
3916:     if (std::isinf(x)) {
3917:         return T(0.0);
3918:     }
3919: 
3920:     if (std::abs(x) < T(0.5)) {
3921:         return T(1.0) + x * x * (T(-1.0) / T(6.0) + x * x * (T(1.0) / T(120.0) + x * x * (T(-1.0) / T(5040.0) + x * x * (T(1.0) / T(362880.0) + x * x * (T(-1.0) / T(39916800.0) + x * x * (T(1.0) / T(6227020800.0)))))));
3922:     }
3923: 
3924:     return std::sin(x) / x;
3925: } // T spherical_bessel_j0_forward(T x)
3926: 
3927: C10_CLANG_DIAGNOSTIC_POP()
```
- **EN**: Lines 3901-3927 mainly cover state/variable declarations, return paths, expressions/calls. Notable symbols: T, shifted_chebyshev_polynomial_w_forward, spherical_bessel_j0_forward, isinf.
- **CN**: 第 3901-3927 行主要涉及变量/别名声明、返回路径、表达式或调用。 值得关注的符号包括：T, shifted_chebyshev_polynomial_w_forward, spherical_bessel_j0_forward, isinf。

## Key Concepts / 关键概念
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/AccumulateType.h>`, `<ATen/NumericUtils.h>`, `<ATen/jiterator_macros.h>`, `<c10/macros/Macros.h>`, `<c10/util/BFloat16.h>`, `<c10/util/Half.h>`, `<c10/util/MathConstants.h>`, `<cfloat>`, `<cmath>`, `<cstdint>` ...
- **Macros / 宏**: None highlighted / 无特别标注
- **Namespaces / 命名空间**: `at::`, `c10::`, `std::`
