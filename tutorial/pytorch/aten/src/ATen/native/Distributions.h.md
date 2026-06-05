# Distributions.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Distributions.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Distributions. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 distributions 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <array>
0004: #include <ATen/native/Math.h>
0005: #include <c10/macros/Macros.h>
0006: #include <c10/util/MathConstants.h>
0007: 
0008: // ROCm hip compiler doesn't work well with using std:: in kernel functions
0009: #if defined(__CUDA_ARCH__) || defined(__HIPCC__)
0010: #if defined(__CUDA_ARCH__)
0011: #include <c10/cuda/CUDAMathCompat.h>
0012: #elif defined(__HIPCC__)
0013: #include <c10/hip/HIPMathCompat.h>
0014: #endif
0015: #define compat_exp c10::cuda::compat::exp
0016: #define compat_ceil c10::cuda::compat::ceil
0017: #define compat_floor c10::cuda::compat::floor
0018: #define compat_log c10::cuda::compat::log
0019: #define compat_pow c10::cuda::compat::pow
0020: #define compat_sqrt c10::cuda::compat::sqrt
0021: #define compat_tan c10::cuda::compat::tan
0022: #define compat_abs c10::cuda::compat::abs
0023: #define compat_log1p c10::cuda::compat::log1p
0024: #else
0025: #define compat_exp std::exp
0026: #define compat_ceil std::ceil
0027: #define compat_floor std::floor
0028: #define compat_log std::log
0029: #define compat_pow std::pow
0030: #define compat_sqrt std::sqrt
```
- **EN**: Lines 1-30 mainly cover macro-based glue, header inclusion, conditional compilation. Notable symbols: defined.
- **CN**: 第 1-30 行主要涉及宏定义或宏调用、头文件包含、预处理条件。 值得关注的符号包括：defined。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #define compat_tan std::tan
0032: #define compat_abs std::abs
0033: #define compat_log1p std::log1p
0034: #endif
0035: 
0036: namespace {
0037: 
0038: #if !defined(__CUDA_ARCH__) && !defined(__HIPCC__)
0039: // we cannot use std::isnan directly due to some incompatibility of
0040: // gcc constexpr'ing and nvcc
0041: using std::isnan;
0042: #endif
0043: 
0044: // Here sampler_t should be function type scalar_t(void). For gpu
0045: // "sampler" is a device function, but since ROCM doesn't have
0046: // equivalent to nvstd::function, we use a template type parameter to
0047: // capture it.
0048: template<typename scalar_t, typename sampler_t>
0049: struct BaseSampler {
0050:   sampler_t sampler;
0051:   C10_DEVICE BaseSampler(const sampler_t& sampler): sampler(sampler) {}
0052:   C10_DEVICE scalar_t sample() {
0053:     return sampler();
0054:   }
0055: };
0056: 
0057: // The function `sample_gamma` is
0058: // is adapted from Numpy's distributions.c implementation.
0059: // It is MIT licensed, so here is the copyright:
0060: 
```
- **EN**: Lines 31-60 mainly cover comments/documentation, macro-based glue, conditional compilation. Notable symbols: defined, scalar_t, BaseSampler, sampler.
- **CN**: 第 31-60 行主要涉及注释或说明、宏定义或宏调用、预处理条件。 值得关注的符号包括：defined, scalar_t, BaseSampler, sampler。

### Lines 61-90 / 第 61-90 行
```cpp
0061: /* Copyright 2005 Robert Kern (robert.kern@gmail.com)
0062:  *
0063:  * Permission is hereby granted, free of charge, to any person obtaining a
0064:  * copy of this software and associated documentation files (the
0065:  * "Software"), to deal in the Software without restriction, including
0066:  * without limitation the rights to use, copy, modify, merge, publish,
0067:  * distribute, sublicense, and/or sell copies of the Software, and to
0068:  * permit persons to whom the Software is furnished to do so, subject to
0069:  * the following conditions:
0070:  *
0071:  * The above copyright notice and this permission notice shall be included
0072:  * in all copies or substantial portions of the Software.
0073:  *
0074:  * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
0075:  * OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
0076:  * MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
0077:  * IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
0078:  * CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
0079:  * TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
0080:  * SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
0081: */
0082: 
0083: template<typename scalar_t, typename accscalar_t, typename uniform_sampler_t, typename normal_sampler_t>
0084: C10_DEVICE scalar_t sample_gamma(scalar_t alpha, BaseSampler<accscalar_t, uniform_sampler_t>& standard_uniform, BaseSampler<accscalar_t, normal_sampler_t>& standard_normal) {
0085:   accscalar_t scale = 1.0f;
0086: 
0087:   // Boost alpha for higher acceptance probability.
0088:   if (alpha < 1.0f) {
0089:     if (alpha == 0.f) return 0.f;
0090:     scale *= compat_pow(1 - standard_uniform.sample(), 1.0f / alpha);
```
- **EN**: Lines 61-90 mainly cover comments/documentation, state/variable declarations, control-flow checks. Notable symbols: Kern, files, sample_gamma, compat_pow.
- **CN**: 第 61-90 行主要涉及注释或说明、变量/别名声明、控制流逻辑。 值得关注的符号包括：Kern, files, sample_gamma, compat_pow。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     alpha += 1.0f;
0092:   }
0093: 
0094:   // This implements the acceptance-rejection method of Marsaglia and Tsang (2000)
0095:   // doi:10.1145/358407.358414
0096:   const accscalar_t d = alpha - 1.0f / 3.0f;
0097:   const accscalar_t c = 1.0f / compat_sqrt(9.0f * d);
0098:   for (;;) {
0099:     accscalar_t x, y;
0100:     do {
0101:       x = standard_normal.sample();
0102:       y = 1.0f + c * x;
0103:     } while (y <= 0);
0104:     const accscalar_t v = y * y * y;
0105:     const accscalar_t u = 1 - standard_uniform.sample();
0106:     const accscalar_t xx = x * x;
0107:     if (u < 1.0f - 0.0331f * xx * xx)
0108:       return static_cast<scalar_t>(scale * d * v);
0109:     if (compat_log(u) < 0.5f * xx + d * (1.0f - v + compat_log(v)))
0110:       return static_cast<scalar_t>(scale * d * v);
0111:   }
0112: }
0113: 
0114: /* the functions stirling_approx_tail, binomial_inversion, and btrs are adapted
0115:  * from TensorFlow's random_binomial_op.cc implementation. That code is under
0116:  * copyright: 2019 The TensorFlow Authors.
0117:  *
0118:  * It was released under the Apache License, Version 2.0 (the "License"), available at:
0119:  *    http://www.apache.org/licenses/LICENSE-2.0
0120:  */
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, comments/documentation, control-flow checks. Notable symbols: Tsang, compat_sqrt, sample, compat_log.
- **CN**: 第 91-120 行主要涉及变量/别名声明、注释或说明、控制流逻辑。 值得关注的符号包括：Tsang, compat_sqrt, sample, compat_log。

### Lines 121-150 / 第 121-150 行
```cpp
0121: 
0122: template<typename scalar_t>
0123: C10_DEVICE scalar_t stirling_approx_tail(scalar_t k) {
0124:   constexpr static scalar_t kTailValues[] = {
0125:     0.0810614667953272,
0126:     0.0413406959554092,
0127:     0.0276779256849983,
0128:     0.02079067210376509,
0129:     0.0166446911898211,
0130:     0.0138761288230707,
0131:     0.0118967099458917,
0132:     0.0104112652619720,
0133:     0.00925546218271273,
0134:     0.00833056343336287
0135:   };
0136:   if (k < std::size(kTailValues)) {
0137:     return kTailValues[static_cast<size_t>(k)];
0138:   }
0139:   scalar_t kp1sq = (k + 1) * (k + 1);
0140:   return (1.0 / 12 - (1.0 / 360 - 1.0 / 1260 / kp1sq) / kp1sq) / (k + 1);
0141: }
0142: 
0143: 
0144: template<typename scalar_t, typename accscalar_t, typename uniform_sampler_t>
0145: C10_DEVICE scalar_t binomial_inversion(scalar_t count, scalar_t prob, BaseSampler<accscalar_t, uniform_sampler_t>& standard_uniform) {
0146:   accscalar_t U;
0147:   accscalar_t geom_sum = 0;
0148:   scalar_t num_geom = 0;
0149: 
0150:   accscalar_t logprob = compat_log1p(-prob);
```
- **EN**: Lines 121-150 mainly cover expressions/calls, state/variable declarations, template setup. Notable symbols: stirling_approx_tail, size, binomial_inversion, compat_log1p.
- **CN**: 第 121-150 行主要涉及表达式或调用、变量/别名声明、模板声明。 值得关注的符号包括：stirling_approx_tail, size, binomial_inversion, compat_log1p。

### Lines 151-180 / 第 151-180 行
```cpp
0151: 
0152:   while (true) {
0153:     U = standard_uniform.sample();
0154:     accscalar_t geom = compat_ceil(compat_log(U) / logprob);
0155:     geom_sum += geom;
0156:     if (geom_sum > count) {
0157:       break;
0158:     }
0159:     num_geom = num_geom + 1;
0160:   }
0161:   return num_geom;
0162: }
0163: 
0164: template<typename scalar_t, typename accscalar_t, typename uniform_sampler_t>
0165: C10_DEVICE scalar_t btrs(scalar_t count, scalar_t prob, BaseSampler<accscalar_t, uniform_sampler_t>& standard_uniform) {
0166:   scalar_t k;
0167:   accscalar_t U, V, us;
0168: 
0169:   // This is spq in the paper.
0170:   const accscalar_t stddev = compat_sqrt(count * prob * (1 - prob));
0171: 
0172:   // Other coefficients for Transformed Rejection sampling.
0173:   const accscalar_t b = 1.15 + 2.53 * stddev;
0174:   const accscalar_t a = -0.0873 + 0.0248 * b + 0.01 * prob;
0175:   const accscalar_t c = count * prob + 0.5;
0176:   const accscalar_t v_r = 0.92 - 4.2 / b;
0177:   const accscalar_t r = prob / (1 - prob);
0178: 
0179:   const accscalar_t alpha = (2.83 + 5.1 / b) * stddev;
0180:   const accscalar_t m = compat_floor((count + 1) * prob);
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: sample, compat_ceil, compat_log, btrs.
- **CN**: 第 151-180 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：sample, compat_ceil, compat_log, btrs。

### Lines 181-210 / 第 181-210 行
```cpp
0181: 
0182:   while (true) {
0183:     U = standard_uniform.sample() - 0.5;
0184:     V = standard_uniform.sample();
0185: 
0186:     us = 0.5 - compat_abs(U);
0187:     k = static_cast<scalar_t>(compat_floor((2 * a / us + b) * U + c));
0188: 
0189:     // Reject non-sensical answers.
0190:     if (k < 0 || k > count) {
0191:       continue;
0192:     }
0193:     // Region for which the box is tight, and we can return our calculated value.
0194:     // This should happen 0.86 * v_r times. In the limit as n * p is large,
0195:     // the acceptance rate converges to ~79% (and in the lower regime it is ~24%).
0196:     if (us >= 0.07 && V <= v_r) {
0197:       return k;
0198:     }
0199: 
0200:     // This deviates from Hormann's BTRS algorithm, as there is a log missing.
0201:     // For all (u, v) pairs outside of the bounding box, this calculates the
0202:     // transformed-reject ratio.
0203:     V = compat_log(V * alpha / (a / (us * us) + b));
0204:     accscalar_t upperbound =
0205:         ((m + 0.5) * compat_log((m + 1) / (r * (count - m + 1))) +
0206:          (count + 1) * compat_log((count - m + 1) / (count - k + 1)) +
0207:          (k + 0.5) * compat_log(r * (count - k + 1) / (k + 1)) +
0208:          stirling_approx_tail<accscalar_t>(m) + stirling_approx_tail<accscalar_t>(count - m) -
0209:          stirling_approx_tail<accscalar_t>(k) - stirling_approx_tail<accscalar_t>(count - k));
0210: 
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: sample, compat_abs, compat_floor, all.
- **CN**: 第 181-210 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：sample, compat_abs, compat_floor, all。

### Lines 211-240 / 第 211-240 行
```cpp
0211:     if (V <= upperbound) {
0212:       return k;
0213:     }
0214:   }
0215: }
0216: 
0217: template<typename scalar_t, typename accscalar_t, typename uniform_sampler_t>
0218: C10_DEVICE scalar_t sample_binomial(scalar_t count, scalar_t prob, BaseSampler<accscalar_t, uniform_sampler_t>& standard_uniform) {
0219:   if (count <= 0.0 || prob <= 0.0) {
0220:     return 0;
0221:   } else if (prob >= 1.0) {
0222:     return count;
0223:   } else if (prob <= 0.5) {
0224:     if (count * prob >= 10.0) {
0225:       // btrs
0226:       return btrs<scalar_t, accscalar_t, uniform_sampler_t>(count, prob, standard_uniform);
0227:     } else {
0228:       // binomial inversion
0229:       return binomial_inversion<scalar_t, accscalar_t, uniform_sampler_t>(count, prob, standard_uniform);
0230:     }
0231:   } else if (prob > 0.5) {
0232:     scalar_t qprob = 1.0 - prob;
0233:     if (count * qprob >= 10.0) {
0234:       // btrs
0235:       return count - btrs<scalar_t, accscalar_t, uniform_sampler_t>(count, qprob, standard_uniform);
0236:     } else {
0237:       // count - binomial inversion
0238:       return count - binomial_inversion<scalar_t, accscalar_t, uniform_sampler_t>(count, qprob, standard_uniform);
0239:     }
0240:   } else {
```
- **EN**: Lines 211-240 mainly cover return paths, function signatures/definitions, expressions/calls. Notable symbols: sample_binomial.
- **CN**: 第 211-240 行主要涉及返回路径、函数签名或实现、表达式或调用。 值得关注的符号包括：sample_binomial。

### Lines 241-270 / 第 241-270 行
```cpp
0241:     // prob is nan?
0242:     return static_cast<scalar_t>(NAN);
0243:   }
0244: }
0245: 
0246: /*
0247:  * This function is derived from the implementation of the digamma function in the Cephes Math Library.
0248:  * See note [3-Clause BSD License for the Cephes Math Library] in ATen/native/Math.h.
0249:  */
0250: template<typename scalar_t, typename accscalar_t>
0251: C10_DEVICE inline scalar_t digamma_one(scalar_t x) {
0252:   constexpr accscalar_t PSI_10 = 2.25175258906672110764;
0253:   if (x == 0) {
0254:     return INFINITY;
0255:   }
0256:   accscalar_t additional_summand = 0;
0257:   int x_is_integer = x == compat_floor(x);
0258:   if (x < 0) {
0259:     if (x_is_integer) {
0260:       return INFINITY;
0261:     }
0262:     // it is more standard to write this as recursion, but
0263:     // nvcc does not like that
0264:     additional_summand = -c10::pi<scalar_t> /
0265:         compat_tan(c10::pi<scalar_t> * x);
0266:     x = 1 - x;
0267:   }
0268: 
0269:   // Push x to be >= 10
0270:   accscalar_t result = 0;
```
- **EN**: Lines 241-270 mainly cover comments/documentation, expressions/calls, state/variable declarations. Notable symbols: digamma_one, compat_floor, compat_tan.
- **CN**: 第 241-270 行主要涉及注释或说明、表达式或调用、变量/别名声明。 值得关注的符号包括：digamma_one, compat_floor, compat_tan。

### Lines 271-300 / 第 271-300 行
```cpp
0271:   while (x < 10) {
0272:     result -= 1 / x;
0273:     x += 1;
0274:   }
0275:   if (x == 10) {
0276:     return result + PSI_10 + additional_summand;
0277:   }
0278: 
0279:   // Compute asymptotic digamma
0280:   static const accscalar_t A[] = {
0281:      8.33333333333333333333E-2,
0282:     -2.10927960927960927961E-2,
0283:      7.57575757575757575758E-3,
0284:     -4.16666666666666666667E-3,
0285:      3.96825396825396825397E-3,
0286:     -8.33333333333333333333E-3,
0287:      8.33333333333333333333E-2,
0288:   };
0289: 
0290:   accscalar_t y = 0;
0291:   if (x < 1.0e17f) {
0292:     accscalar_t z = 1.0 / (x * x);
0293:     y = z * polevl<accscalar_t>(z, A, 6);
0294:   }
0295:   return static_cast<scalar_t>(
0296:       result + compat_log(x) - (0.5f / x) - y + additional_summand);
0297: }
0298: 
0299: // Computes the reparameterized gradient -(d/dalpha cdf(x;alpha)) / pdf(x;alpha)
0300: // for random number x drawn from a standard Gamma distribution Gamma(alpha).
```
- **EN**: Lines 271-300 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: compat_log, cdf, pdf, Gamma.
- **CN**: 第 271-300 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：compat_log, cdf, pdf, Gamma。

### Lines 301-330 / 第 301-330 行
```cpp
0301: template <typename scalar_t, typename accscalar_t>
0302: C10_HOST_DEVICE scalar_t standard_gamma_grad_one(scalar_t alpha_, scalar_t x_) {
0303:   // Use a Taylor series expansion for small x.
0304:   accscalar_t x = static_cast<accscalar_t>(x_);
0305:   accscalar_t alpha = static_cast<accscalar_t>(alpha_);
0306:   if (x < 0.8f) {
0307:     accscalar_t numer = 1;
0308:     accscalar_t denom = alpha;
0309:     auto series1 = numer / denom;
0310:     auto series2 = numer / (denom * denom);
0311:     for (int i = 1; i <= 5; ++i) {
0312:       numer *= -x / static_cast<accscalar_t>(i);
0313:       denom += 1;
0314:       series1 += numer / denom;
0315:       series2 += numer / (denom * denom);
0316:     }
0317:     const auto pow_x_alpha = compat_pow(x, alpha);
0318:     const auto gamma_pdf = compat_pow(x, alpha - 1) * compat_exp(-x);
0319:     const auto gamma_cdf = pow_x_alpha * series1;
0320:     const auto gamma_cdf_alpha =
0321:         (compat_log(x) - digamma_one<accscalar_t, accscalar_t>(alpha)) *
0322:             gamma_cdf -
0323:         pow_x_alpha * series2;
0324:     const auto result = -gamma_cdf_alpha / gamma_pdf;
0325:     return isnan(result) ? static_cast<scalar_t>( 0.f ) : static_cast<scalar_t>(result);
0326:   }
0327: 
0328:   // Use a Rice saddle point expansion for large alpha.
0329:   if (alpha > 8.0f) {
0330:     if (0.9f * alpha <= x && x <= 1.1f * alpha) {
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: standard_gamma_grad_one, compat_pow, compat_exp, compat_log.
- **CN**: 第 301-330 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：standard_gamma_grad_one, compat_pow, compat_exp, compat_log。

### Lines 331-360 / 第 331-360 行
```cpp
0331:       const auto numer_1 = 1 + 24 * alpha * (1 + 12 * alpha);
0332:       const auto numer_2 = 1440 * (alpha * alpha) + 6 * x * (53 - 120 * x)
0333:           - 65 * x * x / alpha + alpha * (107 + 3600 * x);
0334:       const auto denom = 1244160 * (alpha * alpha) * (alpha * alpha);
0335:       return static_cast<scalar_t>(numer_1 * numer_2 / denom);
0336:     }
0337:     const auto denom = compat_sqrt(8 * alpha);
0338:     const auto term2 = denom / (alpha - x);
0339:     const auto term3 = compat_pow(
0340:         x - alpha - alpha * compat_log(x / alpha),
0341:         static_cast<accscalar_t>(-1.5));
0342:     const auto term23 = (x < alpha) ? term2 - term3 : term2 + term3;
0343:     const auto term1 = compat_log(x / alpha) * term23 -
0344:         compat_sqrt(2 / alpha) * (alpha + x) / ((alpha - x) * (alpha - x));
0345:     const auto stirling = 1 + 1 / (12 * alpha) * (1 + 1 / (24 * alpha));
0346:     const auto numer = x * term1;
0347:     return static_cast<scalar_t>(-stirling * numer / denom);
0348:   }
0349: 
0350:   // Use a bivariate rational approximation to the reparameterized gradient.
0351:   const auto u = compat_log(x / alpha);
0352:   const auto v = compat_log(alpha);
0353:   static const accscalar_t coef_uv[3][8] = {
0354:     {0.16009398, -0.094634809, 0.025146376, -0.0030648343,
0355:      1, 0.32668115, 0.10406089, 0.0014179084},
0356:     {0.53487893, 0.1298071, 0.065735949, -0.0015649758,
0357:      0.16639465, 0.020070113, -0.0035938915, -0.00058392623},
0358:     {0.040121004, -0.0065914022, -0.0026286047, -0.0013441777,
0359:      0.017050642, -0.0021309326, 0.00085092367, -1.5247877e-07},
0360:   };
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: compat_sqrt, compat_pow, compat_log.
- **CN**: 第 331-360 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：compat_sqrt, compat_pow, compat_log。

### Lines 361-390 / 第 361-390 行
```cpp
0361:   accscalar_t coef_v[8];
0362:   for (int i = 0; i < 8; ++ i) {
0363:     coef_v[i] = coef_uv[0][i] + u * (coef_uv[1][i] + u * coef_uv[2][i]);
0364:   }
0365:   const auto p = coef_v[0] + v * (coef_v[1] + v * (coef_v[2] + v * coef_v[3]));
0366:   const auto q = coef_v[4] + v * (coef_v[5] + v * (coef_v[6] + v * coef_v[7]));
0367:   return static_cast<scalar_t>(compat_exp(p / q));
0368: }
0369: 
0370: // Approximate reparameterized gradient of Beta(x,alpha,beta) wrt alpha.
0371: // Assumes x is close to zero and uses a Taylor expansion.
0372: template <typename scalar_t, typename accscalar_t>
0373: C10_DEVICE inline scalar_t _beta_grad_alpha_small(scalar_t x, scalar_t alpha, scalar_t beta) {
0374:   const scalar_t factor = digamma_one<scalar_t, accscalar_t>(alpha)
0375:                         - digamma_one<scalar_t, accscalar_t>(alpha + beta) - compat_log(x);
0376:   scalar_t numer = 1;
0377:   scalar_t series = numer / alpha * (factor + 1 / alpha);
0378:   for (int i = 1; i <= 10; ++i) {
0379:     scalar_t casted_i = static_cast<scalar_t>(i);
0380:     numer *= (casted_i - beta) * x / casted_i;
0381:     const scalar_t denom = alpha + casted_i;
0382:     series += numer / denom * (factor + 1 / denom);
0383:   }
0384:   const scalar_t result = x * compat_pow(1 - x, -beta) * series;
0385:   return isnan(result) ? static_cast<scalar_t>( 0.f ) : result;
0386: }
0387: 
0388: // Approximate reparameterized gradient of Beta(x,alpha,beta) wrt beta.
0389: // Assumes x is close to zero and uses a Taylor expansion.
0390: template <typename scalar_t, typename accscalar_t>
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: compat_exp, Beta, _beta_grad_alpha_small, compat_log.
- **CN**: 第 361-390 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：compat_exp, Beta, _beta_grad_alpha_small, compat_log。

### Lines 391-420 / 第 391-420 行
```cpp
0391: C10_DEVICE inline scalar_t _beta_grad_beta_small(scalar_t x, scalar_t alpha, scalar_t beta) {
0392:   const scalar_t factor = digamma_one<scalar_t, accscalar_t>(alpha + beta) - digamma_one<scalar_t, accscalar_t>(beta);
0393:   scalar_t numer = 1, betas = 1, dbetas = 0, series = factor / alpha;
0394:   for (int i = 1; i <= 8; ++i) {
0395:     scalar_t casted_i = static_cast<scalar_t>(i);
0396:     numer *= -x / casted_i;
0397:     dbetas = dbetas * (beta - casted_i) + betas;
0398:     betas = betas * (beta - casted_i);
0399:     series += numer / (alpha + casted_i) * (dbetas + factor * betas);
0400:   }
0401:   const scalar_t result = -compat_pow(1 - x, 1 - beta) * series;
0402:   return isnan(result) ? static_cast<scalar_t>( 0.f ) : result;
0403: }
0404: 
0405: // Approximate reparameterized gradient of Beta(x,alpha,beta) wrt alpha.
0406: // Assumes alpha and beta are both large and uses a Rice saddle point expansion.
0407: // To ensure numerical stability, this computation is performed at higher precision.
0408: template<typename scalar_t, typename accscalar_t>
0409: C10_DEVICE inline scalar_t _beta_grad_alpha_mid(accscalar_t x, accscalar_t alpha, accscalar_t beta) {
0410:   const accscalar_t total = alpha + beta;
0411:   const accscalar_t mean = alpha / total;
0412:   const accscalar_t std = compat_sqrt(alpha * beta / (total + 1)) / total;
0413:   if (mean - 0.1 * std <= x && x <= mean + 0.1 * std) {
0414:     // Avoid the singularity at x = mean.
0415:     const accscalar_t poly = 47 * x * (beta * beta) * (beta * beta) + alpha * (
0416:                            (43 + 20 * (16 + 27 * beta) * x) * (beta * beta) * beta + alpha * (
0417:                            3 * (59 + 180 * beta - 90 * x) * (beta * beta) + alpha * (
0418:                            (453 + 1620 * beta * (1 - x) - 455 * x) * beta + alpha * (
0419:                            8 * (1 - x) * (135 * beta - 11)))));
0420:     const accscalar_t prefactor_num = (1 + 12 * alpha) * (1 + 12 * beta) / (total * total);
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, function signatures/definitions, comments/documentation. Notable symbols: _beta_grad_beta_small, compat_pow, isnan, Beta.
- **CN**: 第 391-420 行主要涉及变量/别名声明、函数签名或实现、注释或说明。 值得关注的符号包括：_beta_grad_beta_small, compat_pow, isnan, Beta。

### Lines 421-450 / 第 421-450 行
```cpp
0421:     const accscalar_t prefactor_den = 12960 * alpha * alpha * alpha * beta * beta * (1 + 12 * total);
0422:     return prefactor_num / (1 - x) * poly / prefactor_den;
0423:   }
0424:   const accscalar_t prefactor = -x / compat_sqrt(2 * alpha * beta / total);
0425:   const accscalar_t stirling = (1 + 1 / (12 * alpha) + 1 / (288 * alpha * alpha))
0426:                              * (1 + 1 / (12 * beta) + 1 / (288 * beta * beta))
0427:                              / (1 + 1 / (12 * total) + 1 / (288 * total * total));
0428:   const accscalar_t term1_num = 2 * (alpha * alpha) * (x - 1) + alpha * beta * (x - 1) - x * (beta * beta);
0429:   const accscalar_t axbx = alpha * (x - 1) + beta * x;
0430:   const accscalar_t term1_den = compat_sqrt(2 * alpha / beta) * compat_pow(total, static_cast<accscalar_t>(1.5f)) * axbx * axbx;
0431:   const accscalar_t term1 = term1_num / term1_den;
0432:   const accscalar_t term2 = 0.5f * compat_log(alpha / (total * x));
0433:   const accscalar_t term3_num = compat_sqrt(8 * alpha * beta / total);
0434:   const accscalar_t term3_den = beta * x + alpha * (x - 1);
0435:   const accscalar_t term3 = term3_num / term3_den;
0436:   const accscalar_t term4_base = beta * compat_log(beta / (total * (1 - x))) +
0437:                                alpha * compat_log(alpha / (total * x));
0438:   const accscalar_t term4 = compat_pow(term4_base, static_cast<accscalar_t>(-1.5f));
0439:   const accscalar_t term1234 = term1 + term2 * (term3 + (x < mean ? term4 : -term4));
0440:   return static_cast<scalar_t>(stirling * prefactor * term1234);
0441: }
0442: 
0443: // Computes a scaled reparameterized gradient
0444: //   -(d/dalpha cdf(x;alpha,beta)) / pdf(x;alpha,beta) / (1-x)
0445: // for random number x drawn from a Beta distribution Beta(alpha,beta).
0446: // This function inputs total=alpha+beta to make it easy to implement
0447: // Dirichlet reparameterized gradients in terms of Betas.
0448: template<typename scalar_t, typename accscalar_t>
0449: C10_HOST_DEVICE inline scalar_t dirichlet_grad_one(scalar_t x, scalar_t alpha, scalar_t total) {
0450:   accscalar_t x_ = static_cast<accscalar_t>(x);
```
- **EN**: Lines 421-450 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: compat_sqrt, compat_pow, compat_log, cdf.
- **CN**: 第 421-450 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：compat_sqrt, compat_pow, compat_log, cdf。

### Lines 451-480 / 第 451-480 行
```cpp
0451:   accscalar_t alpha_ = static_cast<accscalar_t>(alpha);
0452:   accscalar_t total_ = static_cast<accscalar_t>(total);
0453: 
0454:   const scalar_t beta = total - alpha;
0455:   const accscalar_t beta_ = total_ - alpha_;
0456:   const scalar_t boundary = total * x * (1 - x);
0457: 
0458:   // Use an asymptotic approximation for x close to 0.
0459:   if (x <= 0.5f && boundary < 2.5f) {
0460:     return _beta_grad_alpha_small<scalar_t, accscalar_t>(x, alpha, beta);
0461:   }
0462: 
0463:   // Use an asymptotic approximation for x close to 1.
0464:   if (x >= 0.5f && boundary < 0.75f) {
0465:     return -_beta_grad_beta_small<scalar_t, accscalar_t>(1 - x, beta, alpha);
0466:   }
0467: 
0468:   // Use an asymptotic approximation when alpha and (total - alpha) are both large.
0469:   if (alpha > 6 && beta > 6) {
0470:     return _beta_grad_alpha_mid<scalar_t, accscalar_t>(x_, alpha_, beta_);
0471:   }
0472: 
0473:   // Use a rational correction to an analytic approximation.
0474:   static const accscalar_t c[2][3][3][4] = {
0475:     {{{1.003668233, -0.01061107488, -0.0657888334, 0.01201642863},
0476:       {0.6336835991, -0.3557432599, 0.05486251648, -0.001465281033},
0477:       {-0.03276231906, 0.004474107445, 0.002429354597, -0.0001557569013}},
0478:      {{0.221950385, -0.3187676331, 0.01799915743, 0.01074823814},
0479:       {-0.2951249643, 0.06219954479, 0.01535556598, 0.001550077057},
0480:       {0.02155310298, 0.004170831599, 0.001292462449, 6.976601077e-05}},
```
- **EN**: Lines 451-480 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: and.
- **CN**: 第 451-480 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：and。

### Lines 481-510 / 第 481-510 行
```cpp
0481:      {{-0.05980841433, 0.008441916499, 0.01085618172, 0.002319392565},
0482:       {0.02911413504, 0.01400243777, -0.002721828457, 0.000751041181},
0483:       {0.005900514878, -0.001936558688, -9.495446725e-06, 5.385558597e-05}}},
0484:     {{{1, -0.02924021934, -0.04438342661, 0.007285809825},
0485:       {0.6357567472, -0.3473456711, 0.05454656494, -0.002407477521},
0486:       {-0.03301322327, 0.004845219414, 0.00231480583, -0.0002307248149}},
0487:      {{0.5925320577, -0.1757678135, 0.01505928619, 0.000564515273},
0488:       {0.1014815858, -0.06589186703, 0.01272886114, -0.0007316646956},
0489:       {-0.007258481865, 0.001096195486, 0.0003934994223, -4.12701925e-05}},
0490:      {{0.06469649321, -0.0236701437, 0.002902096474, -5.896963079e-05},
0491:       {0.001925008108, -0.002869809258, 0.0008000589141, -6.063713228e-05},
0492:       {-0.0003477407336, 6.959756487e-05, 1.097287507e-05, -1.650964693e-06}}},
0493:   };
0494:   const accscalar_t u = compat_log(x_);
0495:   const accscalar_t a = compat_log(alpha_) - u;
0496:   const accscalar_t b = compat_log(total_) - a;
0497:   const accscalar_t pow_u[3] = {1, u, u * u};
0498:   const accscalar_t pow_a[3] = {1, a, a * a};
0499:   accscalar_t p = 0.0;
0500:   accscalar_t q = 0.0;
0501:   for (int i = 0; i < 3; ++i) {
0502:     for (int j = 0; j < 3; ++j) {
0503:       const accscalar_t ua = pow_u[i] * pow_a[j];
0504:       p += ua * (c[0][i][j][0] + b * (c[0][i][j][1] + b * (c[0][i][j][2] + b * c[0][i][j][3])));
0505:       q += ua * (c[1][i][j][0] + b * (c[1][i][j][1] + b * (c[1][i][j][2] + b * c[1][i][j][3])));
0506:     }
0507:   }
0508:   const accscalar_t approx = x_ * (digamma_one<scalar_t, accscalar_t>(total_) - digamma_one<scalar_t, accscalar_t>(alpha_)) / beta_;
0509:   return static_cast<scalar_t>(p / q * approx);
0510: }
```
- **EN**: Lines 481-510 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: compat_log.
- **CN**: 第 481-510 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：compat_log。

### Lines 511-512 / 第 511-512 行
```cpp
0511: 
0512: } // namespace
```
- **EN**: Lines 511-512 mainly cover namespace structuring.
- **CN**: 第 511-512 行主要涉及命名空间组织。

## Key Concepts / 关键概念
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化

## Dependencies / 依赖关系
- **Headers / 头文件**: `<array>`, `<ATen/native/Math.h>`, `<c10/macros/Macros.h>`, `<c10/util/MathConstants.h>`, `<c10/cuda/CUDAMathCompat.h>`, `<c10/hip/HIPMathCompat.h>`
- **Macros / 宏**: None highlighted / 无特别标注
- **Namespaces / 命名空间**: `at::`, `c10::`, `std::`
