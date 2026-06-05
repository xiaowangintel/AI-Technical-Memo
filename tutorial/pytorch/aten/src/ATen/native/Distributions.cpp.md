# Distributions.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Distributions.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Distributions. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 distributions 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Dispatch.h>
0004: #include <ATen/TensorIterator.h>
0005: #include <ATen/TensorOperators.h>
0006: #include <c10/util/Exception.h>
0007: #include <optional>
0008: 
0009: #include <ATen/CPUGeneratorImpl.h>
0010: #include <ATen/core/DistributionsHelper.h>
0011: #include <ATen/native/Distributions.h>
0012: #include <ATen/native/DispatchStub.h>
0013: #include <ATen/native/UnaryOps.h>
0014: #include <ATen/native/DistributionTemplates.h>
0015: #include <ATen/NamedTensorUtils.h>
0016: #include <ATen/native/cpu/Loops.h>
0017: 
0018: #ifndef AT_PER_OPERATOR_HEADERS
0019: #include <ATen/Functions.h>
0020: #include <ATen/NativeFunctions.h>
0021: #else
0022: #include <ATen/ops/_dirichlet_grad_native.h>
0023: #include <ATen/ops/_sample_dirichlet_native.h>
0024: #include <ATen/ops/_standard_gamma_grad_native.h>
0025: #include <ATen/ops/_standard_gamma_native.h>
0026: #include <ATen/ops/_assert_async.h>
0027: #include <ATen/ops/argmax.h>
0028: #include <ATen/ops/bernoulli_native.h>
0029: #include <ATen/ops/binomial_native.h>
0030: #include <ATen/ops/cauchy_native.h>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/ops/div.h>
0032: #include <ATen/ops/empty.h>
0033: #include <ATen/ops/empty_like.h>
0034: #include <ATen/ops/exponential_native.h>
0035: #include <ATen/ops/geometric_native.h>
0036: #include <ATen/ops/log_normal_native.h>
0037: #include <ATen/ops/multinomial_native.h>
0038: #include <ATen/ops/normal_native.h>
0039: #include <ATen/ops/poisson_native.h>
0040: #include <ATen/ops/random_native.h>
0041: #include <ATen/ops/topk.h>
0042: #include <ATen/ops/uniform_native.h>
0043: #include <ATen/ops/zeros.h>
0044: #endif
0045: 
0046: #include <utility>
0047: 
0048: namespace {
0049: /*
0050:  * This section is a counterpart to Distributions.cu
0051:  *
0052:  */
0053: 
0054: // The function `sample_poisson`
0055: // is adapted from Numpy's distributions.c implementation.
0056: // It is MIT licensed, so here is the copyright:
0057: 
0058: /* Copyright 2005 Robert Kern (robert.kern@gmail.com)
0059:  *
0060:  * Permission is hereby granted, free of charge, to any person obtaining a
```
- **EN**: Lines 31-60 mainly cover header inclusion, comments/documentation, conditional compilation. Notable symbols: Kern.
- **CN**: 第 31-60 行主要涉及头文件包含、注释或说明、预处理条件。 值得关注的符号包括：Kern。

### Lines 61-90 / 第 61-90 行
```cpp
0061:  * copy of this software and associated documentation files (the
0062:  * "Software"), to deal in the Software without restriction, including
0063:  * without limitation the rights to use, copy, modify, merge, publish,
0064:  * distribute, sublicense, and/or sell copies of the Software, and to
0065:  * permit persons to whom the Software is furnished to do so, subject to
0066:  * the following conditions:
0067:  *
0068:  * The above copyright notice and this permission notice shall be included
0069:  * in all copies or substantial portions of the Software.
0070:  *
0071:  * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
0072:  * OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
0073:  * MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
0074:  * IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
0075:  * CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
0076:  * TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
0077:  * SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
0078:  */
0079: 
0080: 
0081: int64_t sample_poisson(double lambda, at::CPUGeneratorImpl* generator) {
0082:   TORCH_CHECK(lambda >= 0, "invalid Poisson rate, expected rate to be non-negative");
0083:   at::uniform_real_distribution<double> standard_uniform(0.0, 1.0);
0084:   if (lambda >= 10) {
0085:     // transformed rejection method, (Hoermann, 1993)
0086: 
0087:     double slam = std::sqrt(lambda);
0088:     double loglam = std::log(lambda);
0089:     double b = 0.931 + 2.53 * slam;
0090:     double a = -0.059 + 0.02483 * b;
```
- **EN**: Lines 61-90 mainly cover comments/documentation, state/variable declarations, function signatures/definitions. Notable symbols: files, sample_poisson, TORCH_CHECK, standard_uniform.
- **CN**: 第 61-90 行主要涉及注释或说明、变量/别名声明、函数签名或实现。 值得关注的符号包括：files, sample_poisson, TORCH_CHECK, standard_uniform。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     double invalpha = 1.1239 + 1.1328 / (b - 3.4);
0092:     double vr = 0.9277 - 3.6224 / (b - 2);
0093: 
0094:     while (true) {
0095:       double U = standard_uniform(generator) - 0.5;
0096:       double V = standard_uniform(generator);
0097:       double us = 0.5 - std::fabs(U);
0098:       auto k = std::floor((2 * a / us + b) * U + lambda + 0.43);
0099:       if ((us >= 0.07) && (V <= vr)) {
0100:         return static_cast<int64_t>(k);
0101:       }
0102:       if ((k < 0) || ((us < 0.013) && (V > us))) {
0103:         continue;
0104:       }
0105:       if ((std::log(V) + std::log(invalpha) - std::log(a / (us * us) + b)) <=
0106:           (-lambda + k * loglam - std::lgamma(k + 1))) {
0107:         return static_cast<int64_t>(k);
0108:       }
0109:     }
0110:   } else if (lambda == 0) {
0111:     return 0;
0112:   } else {
0113:     auto enlam = std::exp(-lambda);
0114:     int64_t X = 0;
0115:     auto prod = 1.0;
0116:     while (true) {
0117:       auto U = standard_uniform(generator);
0118:       prod *= U;
0119:       if (prod > enlam) {
0120:         X += 1;
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: standard_uniform, fabs, floor, log.
- **CN**: 第 91-120 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：standard_uniform, fabs, floor, log。

### Lines 121-150 / 第 121-150 行
```cpp
0121:       } else {
0122:         return X;
0123:       }
0124:     }
0125:   }
0126: }
0127: 
0128: } // namespace
0129: 
0130: namespace at::native {
0131: 
0132: DEFINE_DISPATCH(bernoulli_tensor_stub);
0133: DEFINE_DISPATCH(bernoulli_scalar_stub);
0134: DEFINE_DISPATCH(cauchy_stub);
0135: DEFINE_DISPATCH(exponential_stub);
0136: DEFINE_DISPATCH(multinomial_with_replacement_stub);
0137: DEFINE_DISPATCH(geometric_stub);
0138: DEFINE_DISPATCH(log_normal_stub);
0139: DEFINE_DISPATCH(uniform_stub);
0140: DEFINE_DISPATCH(normal_stub);
0141: DEFINE_DISPATCH(random_stub);
0142: DEFINE_DISPATCH(random_from_to_stub);
0143: DEFINE_DISPATCH(random_full_64_bits_range_stub);
0144: 
0145: // ==================================================== Bernoulli =====================================================
0146: 
0147: template<typename RNG>
0148: struct BernoulliStub {
0149:   void operator()(Tensor& self, const Tensor& p_, std::optional<Generator> gen) {
0150:     bernoulli_tensor_stub(self.device().type(), self, p_, gen);
```
- **EN**: Lines 121-150 mainly cover macro-based glue, expressions/calls, function signatures/definitions. Notable symbols: DEFINE_DISPATCH, bernoulli_tensor_stub, device, type.
- **CN**: 第 121-150 行主要涉及宏定义或宏调用、表达式或调用、函数签名或实现。 值得关注的符号包括：DEFINE_DISPATCH, bernoulli_tensor_stub, device, type。

### Lines 151-180 / 第 151-180 行
```cpp
0151:   }
0152: 
0153:   void operator()(Tensor& self, double p, std::optional<Generator> gen) {
0154:     bernoulli_scalar_stub(self.device().type(), self, p, gen);
0155:   }
0156: };
0157: 
0158: Tensor bernoulli(const Tensor& self, std::optional<Generator> gen) {
0159:   Tensor result = at::empty_like(self, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0160:   result.bernoulli_(self, std::move(gen));
0161:   return result;
0162: }
0163: 
0164: Tensor bernoulli(const Tensor& self, double p, std::optional<Generator> gen) {
0165:   Tensor result = at::empty_like(self, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0166:   result.bernoulli_(p, std::move(gen));
0167:   return result;
0168: }
0169: 
0170: Tensor& bernoulli_out(const Tensor& self, std::optional<Generator> gen, Tensor& result) {
0171:   return at::native::templates::bernoulli_out_impl<BernoulliStub, Generator>(result, self, std::move(gen));
0172: }
0173: 
0174: Tensor& bernoulli_(Tensor& self, const Tensor& p_, std::optional<Generator> gen) {
0175:   return at::native::templates::bernoulli_impl_<BernoulliStub, Generator>(self, p_, std::move(gen));
0176: }
0177: 
0178: Tensor& bernoulli_(Tensor& self, double p, std::optional<Generator> gen) {
0179:   return at::native::templates::bernoulli_impl_<BernoulliStub, Generator>(self, p, std::move(gen));
0180: }
```
- **EN**: Lines 151-180 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: bernoulli_scalar_stub, device, type, bernoulli.
- **CN**: 第 151-180 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：bernoulli_scalar_stub, device, type, bernoulli。

### Lines 181-210 / 第 181-210 行
```cpp
0181: 
0182: // ================================================== LogNormal =======================================================
0183: 
0184: template<typename RNG>
0185: struct LogNormalStub {
0186:   void operator()(TensorIteratorBase& iter, double mean, double std, std::optional<Generator> gen) {
0187:     log_normal_stub(iter.device_type(), iter, mean, std, gen);
0188:   }
0189: };
0190: 
0191: Tensor& log_normal_(Tensor& self, double mean, double std, std::optional<Generator> gen) {
0192:   return at::native::templates::log_normal_impl_<LogNormalStub, Generator>(self, mean, std, std::move(gen));
0193: }
0194: 
0195: // ==================================================== Cauchy ========================================================
0196: 
0197: template<typename RNG>
0198: struct CauchyStub {
0199:   void operator()(TensorIteratorBase& iter, double median, double sigma, std::optional<Generator> gen) {
0200:     cauchy_stub(iter.device_type(), iter, median, sigma, gen);
0201:   }
0202: };
0203: 
0204: Tensor& cauchy_(Tensor& self, double median, double sigma, std::optional<Generator> gen) {
0205:   return at::native::templates::cauchy_impl_<CauchyStub, Generator>(self, median, sigma, std::move(gen));
0206: }
0207: 
0208: // ================================================== Exponential =====================================================
0209: 
0210: template<typename RNG>
```
- **EN**: Lines 181-210 mainly cover function signatures/definitions, state/variable declarations, expressions/calls. Notable symbols: log_normal_stub, device_type, log_normal_, move.
- **CN**: 第 181-210 行主要涉及函数签名或实现、变量/别名声明、表达式或调用。 值得关注的符号包括：log_normal_stub, device_type, log_normal_, move。

### Lines 211-240 / 第 211-240 行
```cpp
0211: struct ExponentialStub {
0212:   void operator()(TensorIteratorBase& iter, double lambda, std::optional<Generator> gen) {
0213:     exponential_stub(iter.device_type(), iter, lambda, gen);
0214:   }
0215: };
0216: 
0217: Tensor& exponential_(Tensor& self, double lambda, std::optional<Generator> gen) {
0218:   return at::native::templates::exponential_impl_<ExponentialStub, Generator>(self, lambda, std::move(gen));
0219: }
0220: 
0221: // =================================================== Geometric ======================================================
0222: 
0223: template<typename RNG>
0224: struct GeometricStub {
0225:   void operator()(TensorIteratorBase& iter, double p, std::optional<Generator> gen) {
0226:     geometric_stub(iter.device_type(), iter, p, gen);
0227:   }
0228: };
0229: 
0230: Tensor& geometric_(Tensor& self, double p, std::optional<Generator> gen) {
0231:   return at::native::templates::geometric_impl_<GeometricStub, Generator>(self, p, std::move(gen));
0232: }
0233: 
0234: // ==================================================== Uniform =======================================================
0235: 
0236: template<typename RNG>
0237: struct UniformStub {
0238:   void operator()(TensorIteratorBase& iter, double from, double to, std::optional<Generator> gen) {
0239:     uniform_stub(iter.device_type(), iter, from, to, gen);
0240:   }
```
- **EN**: Lines 211-240 mainly cover function signatures/definitions, state/variable declarations, expressions/calls. Notable symbols: exponential_stub, device_type, exponential_, move.
- **CN**: 第 211-240 行主要涉及函数签名或实现、变量/别名声明、表达式或调用。 值得关注的符号包括：exponential_stub, device_type, exponential_, move。

### Lines 241-270 / 第 241-270 行
```cpp
0241: };
0242: 
0243: template<typename RNG>
0244: struct UniformMeta {
0245:   // No-op!
0246:   void operator()(TensorIteratorBase& iter, double from, double to, std::optional<Generator> gen) {
0247:   }
0248: };
0249: 
0250: Tensor& uniform_(Tensor& self, double from, double to, std::optional<Generator> gen) {
0251:   return at::native::templates::uniform_impl_<UniformStub, Generator>(self, from, to, std::move(gen));
0252: }
0253: 
0254: Tensor& uniform_meta_(Tensor& self, double from, double to, std::optional<Generator> gen) {
0255:   return at::native::templates::uniform_impl_<UniformMeta, Generator>(self, from, to, std::move(gen));
0256: }
0257: 
0258: // ==================================================== Normal ========================================================
0259: 
0260: template<typename RNG>
0261: struct NormalStub {
0262:   void operator()(Tensor& self, double mean, double std, std::optional<Generator> gen) {
0263:     normal_stub(self.device().type(), self, mean, std, gen);
0264:   }
0265: };
0266: 
0267: template<typename RNG>
0268: struct NormalMeta {
0269:   // No-op!
0270:   void operator()(Tensor& self, double mean, double std, std::optional<Generator> gen) {
```
- **EN**: Lines 241-270 mainly cover function signatures/definitions, state/variable declarations, expressions/calls. Notable symbols: uniform_, move, uniform_meta_, normal_stub.
- **CN**: 第 241-270 行主要涉及函数签名或实现、变量/别名声明、表达式或调用。 值得关注的符号包括：uniform_, move, uniform_meta_, normal_stub。

### Lines 271-300 / 第 271-300 行
```cpp
0271:   }
0272: };
0273: 
0274: // inplace
0275: Tensor& normal_(Tensor& self, double mean, double std, std::optional<Generator> gen) {
0276:   return at::native::templates::normal_impl_<NormalStub, Generator>(self, mean, std, std::move(gen));
0277: }
0278: 
0279: Tensor& normal_meta_(Tensor& self, double mean, double std, std::optional<Generator> gen) {
0280:   return at::native::templates::normal_impl_<NormalMeta, Generator>(self, mean, std, std::move(gen));
0281: }
0282: 
0283: // out tensor float
0284: Tensor& normal_out(const Tensor& mean, double std, std::optional<Generator> gen, Tensor& output) {
0285:   return at::native::templates::normal_out_impl<NormalStub, Generator>(output, mean, std, std::move(gen));
0286: }
0287: 
0288: Tensor& normal_out_meta(const Tensor& mean, double std, std::optional<Generator> gen, Tensor& output) {
0289:   return at::native::templates::normal_out_impl<NormalMeta, Generator>(output, mean, std, std::move(gen));
0290: }
0291: 
0292: // out float tensor
0293: Tensor& normal_out(double mean, const Tensor& std, std::optional<Generator> gen, Tensor& output) {
0294:   return at::native::templates::normal_out_impl<NormalStub, Generator>(output, mean, std, std::move(gen));
0295: }
0296: 
0297: Tensor& normal_out_meta(double mean, const Tensor& std, std::optional<Generator> gen, Tensor& output) {
0298:   return at::native::templates::normal_out_impl<NormalMeta, Generator>(output, mean, std, std::move(gen));
0299: 
0300: }
```
- **EN**: Lines 271-300 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: normal_, move, normal_meta_, normal_out.
- **CN**: 第 271-300 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：normal_, move, normal_meta_, normal_out。

### Lines 301-330 / 第 301-330 行
```cpp
0301: 
0302: // out tensor tensor
0303: Tensor& normal_out(const Tensor& mean, const Tensor& std, std::optional<Generator> gen, Tensor& output) {
0304:   return at::native::templates::normal_out_impl<NormalStub, Generator>(output, mean, std, std::move(gen));
0305: }
0306: 
0307: Tensor& normal_out_meta(const Tensor& mean, const Tensor& std, std::optional<Generator> gen, Tensor& output) {
0308:   return at::native::templates::normal_out_impl<NormalMeta, Generator>(output, mean, std, std::move(gen));
0309: }
0310: 
0311: // functional tensor float
0312: Tensor normal(const Tensor& mean, double std, std::optional<Generator> gen) {
0313:   return at::native::templates::normal_impl<NormalStub, Generator>(mean, std, std::move(gen));
0314: }
0315: 
0316: Tensor normal_meta(const Tensor& mean, double std, std::optional<Generator> gen) {
0317:   return at::native::templates::normal_impl<NormalMeta, Generator>(mean, std, std::move(gen));
0318: }
0319: 
0320: // functional float tensor
0321: Tensor normal(double mean, const Tensor& std, std::optional<Generator> gen) {
0322:   return at::native::templates::normal_impl<NormalStub, Generator>(mean, std, std::move(gen));
0323: }
0324: 
0325: Tensor normal_meta(double mean, const Tensor& std, std::optional<Generator> gen) {
0326:   return at::native::templates::normal_impl<NormalMeta, Generator>(mean, std, std::move(gen));
0327: }
0328: 
0329: // functional tensor tensor
0330: Tensor normal(const Tensor& mean, const Tensor& std, std::optional<Generator> gen) {
```
- **EN**: Lines 301-330 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: normal_out, move, normal_out_meta, normal.
- **CN**: 第 301-330 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：normal_out, move, normal_out_meta, normal。

### Lines 331-360 / 第 331-360 行
```cpp
0331:   return at::native::templates::normal_impl<NormalStub, Generator>(mean, std, std::move(gen));
0332: }
0333: 
0334: Tensor normal_meta(const Tensor& mean, const Tensor& std, std::optional<Generator> gen) {
0335:   return at::native::templates::normal_impl<NormalMeta, Generator>(mean, std, std::move(gen));
0336: }
0337: 
0338: // functional variant, only used by the functionalization pass.
0339: Tensor normal_functional(const Tensor& self, double mean, double std, std::optional<at::Generator> generator) {
0340:   return self.clone().normal_(mean, std, std::move(generator));
0341: }
0342: 
0343: // ==================================================== Random ========================================================
0344: 
0345: template<typename RNG>
0346: struct RandomStub {
0347:   void operator()(TensorIteratorBase& iter, std::optional<Generator> gen) {
0348:     random_stub(iter.device_type(), iter, gen);
0349:   }
0350: };
0351: 
0352: Tensor& random_(Tensor& self, std::optional<Generator> gen) {
0353:   return at::native::templates::random_impl<RandomStub, Generator>(self, std::move(gen));
0354: }
0355: 
0356: template<typename RNG>
0357: struct RandomFromToStub {
0358:   void operator()(TensorIteratorBase& iter, uint64_t range, int64_t from, std::optional<Generator> gen) {
0359:     random_from_to_stub(iter.device_type(), iter, range, from, gen);
0360:   }
```
- **EN**: Lines 331-360 mainly cover expressions/calls, function signatures/definitions, return paths. Notable symbols: move, normal_meta, normal_functional, clone.
- **CN**: 第 331-360 行主要涉及表达式或调用、函数签名或实现、返回路径。 值得关注的符号包括：move, normal_meta, normal_functional, clone。

### Lines 361-390 / 第 361-390 行
```cpp
0361:   void operator()(TensorIteratorBase& iter, std::optional<Generator> gen) {
0362:     random_full_64_bits_range_stub(iter.device_type(), iter, gen);
0363:   }
0364: };
0365: 
0366: Tensor& random_(Tensor& self, int64_t from, std::optional<int64_t> to, std::optional<Generator> gen) {
0367:   return at::native::templates::random_from_to_impl<RandomFromToStub, Generator>(self, from, to, std::move(gen));
0368: }
0369: 
0370: Tensor& random_(Tensor& self, int64_t to, std::optional<Generator> gen) {
0371:   return random_(self, 0, to, std::move(gen));
0372: }
0373: 
0374: Tensor& random_meta_(Tensor& self, std::optional<Generator> gen) {
0375:   // No error checking yay
0376:   return self;
0377: }
0378: 
0379: Tensor& random_meta_(Tensor& self, int64_t from, std::optional<int64_t> to, std::optional<Generator> gen) {
0380:   // No error checking yay
0381:   return self;
0382: }
0383: 
0384: Tensor& random_meta_(Tensor& self, int64_t to, std::optional<Generator> gen) {
0385:   // No error checking yay
0386:   return self;
0387: }
0388: 
0389: // ====================================================================================================================
0390: 
```
- **EN**: Lines 361-390 mainly cover function signatures/definitions, expressions/calls, return paths. Notable symbols: random_full_64_bits_range_stub, device_type, random_, move.
- **CN**: 第 361-390 行主要涉及函数签名或实现、表达式或调用、返回路径。 值得关注的符号包括：random_full_64_bits_range_stub, device_type, random_, move。

### Lines 391-420 / 第 391-420 行
```cpp
0391: Tensor _standard_gamma_grad_cpu(const Tensor& self, const Tensor& output) {
0392:   Tensor ret = at::empty(self.sizes(), self.options());
0393:   auto iter = TensorIteratorConfig()
0394:     .add_output(ret)
0395:     .add_input(self)
0396:     .add_input(output)
0397:     .build();
0398:   AT_DISPATCH_FLOATING_TYPES(self.scalar_type(), "_standard_gamma_grad_cpu", [&] {
0399:     cpu_serial_kernel(iter, [](scalar_t self_val, scalar_t output_val) -> scalar_t{
0400:       return standard_gamma_grad_one<scalar_t, double>(self_val, output_val);
0401:     });
0402:   });
0403:   return ret;
0404: }
0405: 
0406: Tensor _dirichlet_grad_cpu(const Tensor& x, const Tensor& alpha, const Tensor& total) {
0407:   Tensor ret = at::empty(x.sizes(), x.options());
0408:   auto iter = TensorIteratorConfig()
0409:     .add_output(ret)
0410:     .add_input(x)
0411:     .add_input(alpha)
0412:     .add_input(total)
0413:     .build();
0414:   AT_DISPATCH_FLOATING_TYPES(x.scalar_type(), "_dirichlet_grad_cpu", [&] {
0415:     cpu_serial_kernel(iter, [](scalar_t x_val, scalar_t alpha_val, scalar_t total_val) -> scalar_t{
0416:       return dirichlet_grad_one<scalar_t, double>(x_val, alpha_val, total_val);
0417:     });
0418:   });
0419:   return ret;
0420: }
```
- **EN**: Lines 391-420 mainly cover function signatures/definitions, state/variable declarations, return paths. Notable symbols: _standard_gamma_grad_cpu, empty, sizes, options.
- **CN**: 第 391-420 行主要涉及函数签名或实现、变量/别名声明、返回路径。 值得关注的符号包括：_standard_gamma_grad_cpu, empty, sizes, options。

### Lines 421-450 / 第 421-450 行
```cpp
0421: 
0422: /*
0423:  * This section is a counterpart to Distributions.cu
0424:  */
0425: 
0426: Tensor _s_binomial_cpu(const Tensor& count, const Tensor& prob, std::optional<Generator> gen) {
0427:   TORCH_CHECK_VALUE(
0428:       at::isFloatingType(count.scalar_type()),
0429:       "binomial only supports floating-point dtypes for count, got: ",
0430:       count.scalar_type());
0431:   TORCH_CHECK_VALUE(
0432:       at::isFloatingType(prob.scalar_type()),
0433:       "binomial only supports floating-point dtypes for prob, got: ",
0434:       prob.scalar_type());
0435:   Tensor ret = at::zeros(count.sizes(), count.options());
0436:   auto iter = TensorIteratorConfig()
0437:     .add_output(ret)
0438:     .add_input(count)
0439:     .add_input(prob)
0440:     .build();
0441:   AT_DISPATCH_FLOATING_TYPES(ret.scalar_type(), "binomial_cpu", [&] {
0442:     CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
0443:     // See Note [Acquire lock when using random generators]
0444:     std::lock_guard<std::mutex> lock(generator->mutex_);
0445:     cpu_serial_kernel(iter, [generator](scalar_t count_val, scalar_t prob_val) -> scalar_t{
0446:       auto uniform_lambda = [generator] () {
0447:         at::uniform_real_distribution<double> standard_uniform(0.0, 1.0);
0448:         return standard_uniform(generator);
0449:       };
0450:       BaseSampler<double, decltype(uniform_lambda)> standard_uniform(uniform_lambda);
```
- **EN**: Lines 421-450 mainly cover function signatures/definitions, state/variable declarations, comments/documentation. Notable symbols: _s_binomial_cpu, TORCH_CHECK_VALUE, isFloatingType, scalar_type.
- **CN**: 第 421-450 行主要涉及函数签名或实现、变量/别名声明、注释或说明。 值得关注的符号包括：_s_binomial_cpu, TORCH_CHECK_VALUE, isFloatingType, scalar_type。

### Lines 451-480 / 第 451-480 行
```cpp
0451: 
0452:       auto sample = sample_binomial<scalar_t, double, decltype(uniform_lambda)>(count_val, prob_val, standard_uniform);
0453:       return static_cast<scalar_t>(sample);
0454:     });
0455:   });
0456:   return ret;
0457: }
0458: 
0459: Tensor _s_poisson_cpu(const Tensor& lambda, std::optional<Generator> gen) {
0460:   Tensor ret = at::zeros(lambda.sizes(), lambda.options());
0461:   auto iter = TensorIteratorConfig()
0462:     .add_output(ret)
0463:     .add_input(lambda)
0464:     .build();
0465:   AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::BFloat16, at::ScalarType::Half, ret.scalar_type(), "poisson_cpu", [&] {
0466:     CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
0467:     // See Note [Acquire lock when using random generators]
0468:     std::lock_guard<std::mutex> lock(generator->mutex_);
0469:     cpu_serial_kernel(iter, [generator](scalar_t lambda_val) -> scalar_t{
0470:       return static_cast<scalar_t>(sample_poisson(static_cast<double>(lambda_val), generator));
0471:     });
0472:   });
0473:   return ret;
0474: }
0475: 
0476: Tensor _s_gamma_cpu(const Tensor& alpha, std::optional<Generator> gen) {
0477:   Tensor ret = at::zeros(alpha.sizes(), alpha.options());
0478:   auto iter = TensorIteratorConfig()
0479:     .add_output(ret)
0480:     .add_input(alpha)
```
- **EN**: Lines 451-480 mainly cover state/variable declarations, function signatures/definitions, return paths. Notable symbols: _s_poisson_cpu, zeros, sizes, options.
- **CN**: 第 451-480 行主要涉及变量/别名声明、函数签名或实现、返回路径。 值得关注的符号包括：_s_poisson_cpu, zeros, sizes, options。

### Lines 481-510 / 第 481-510 行
```cpp
0481:     .build();
0482:   AT_DISPATCH_FLOATING_TYPES(ret.scalar_type(), "gamma_cpu", [&] {
0483:     CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
0484:     // See Note [Acquire lock when using random generators]
0485:     std::lock_guard<std::mutex> lock(generator->mutex_);
0486:     cpu_serial_kernel(iter, [generator](scalar_t alpha_val) -> scalar_t{
0487:       auto uniform_lambda = [generator] () {
0488:         at::uniform_real_distribution<double> standard_uniform(0.0, 1.0);
0489:         return standard_uniform(generator);
0490:       };
0491:       BaseSampler<double, decltype(uniform_lambda)> standard_uniform(uniform_lambda);
0492: 
0493:       auto normal_lambda = [generator] () {
0494:         at::normal_distribution<double> normal(0.0, 1.0);
0495:         return normal(generator);
0496:       };
0497:       BaseSampler<double, decltype(normal_lambda)> standard_normal(normal_lambda);
0498:       auto sample = sample_gamma<scalar_t, double, decltype(uniform_lambda), decltype(normal_lambda)>(alpha_val, standard_uniform, standard_normal);
0499:       return std::max(std::numeric_limits<scalar_t>::min(), (scalar_t) sample);
0500:     });
0501:   });
0502: 
0503:   return ret;
0504: }
0505: 
0506: Tensor _s_dirichlet_cpu(const Tensor& alpha, std::optional<Generator> gen) {
0507:   Tensor ret = at::zeros(alpha.sizes(), alpha.options());
0508:   AT_DISPATCH_FLOATING_TYPES(ret.scalar_type(), "dirichlet", [&] {
0509:     Tensor gamma = at::zeros(alpha.sizes(), alpha.options().dtype(ScalarType::Double));
0510:     CPUGeneratorImpl* generator = get_generator_or_default<CPUGeneratorImpl>(gen, detail::getDefaultCPUGenerator());
```
- **EN**: Lines 481-510 mainly cover state/variable declarations, function signatures/definitions, return paths. Notable symbols: build, AT_DISPATCH_FLOATING_TYPES, scalar_type, getDefaultCPUGenerator.
- **CN**: 第 481-510 行主要涉及变量/别名声明、函数签名或实现、返回路径。 值得关注的符号包括：build, AT_DISPATCH_FLOATING_TYPES, scalar_type, getDefaultCPUGenerator。

### Lines 511-540 / 第 511-540 行
```cpp
0511:     // See Note [Acquire lock when using random generators]
0512:     std::lock_guard<std::mutex> lock(generator->mutex_);
0513:     /* Generate gamma sample by casting alpha to double to prevent underflow. */
0514:     auto iter1 = TensorIteratorConfig()
0515:       .add_output(gamma)
0516:       .add_input(alpha)
0517:       .check_all_same_dtype(false)
0518:       .build();
0519:     cpu_serial_kernel(iter1, [generator](scalar_t alpha_val) -> double{
0520:       auto uniform_lambda = [generator] () {
0521:         at::uniform_real_distribution<double> standard_uniform(0.0, 1.0);
0522:         return standard_uniform(generator);
0523:       };
0524:       BaseSampler<double, decltype(uniform_lambda)> standard_uniform(uniform_lambda);
0525: 
0526:       auto normal_lambda = [generator] () {
0527:         at::normal_distribution<double> normal(0.0, 1.0);
0528:         return normal(generator);
0529:       };
0530:       BaseSampler<double, decltype(normal_lambda)> standard_normal(normal_lambda);
0531:       auto sample = sample_gamma<double, double, decltype(uniform_lambda), decltype(normal_lambda)>
0532:         (alpha_val, standard_uniform, standard_normal);
0533:       return std::max(std::numeric_limits<double>::min(), sample);
0534:     });
0535:     /* Normalize and cast back to scalar_t. */
0536:     Tensor gamma_sum = gamma.sum(-1, true).expand(alpha.sizes());
0537:     auto iter2 = TensorIteratorConfig()
0538:       .add_output(ret)
0539:       .add_input(gamma)
0540:       .add_input(gamma_sum)
```
- **EN**: Lines 511-540 mainly cover function signatures/definitions, state/variable declarations, comments/documentation. Notable symbols: lock, TensorIteratorConfig, add_output, add_input.
- **CN**: 第 511-540 行主要涉及函数签名或实现、变量/别名声明、注释或说明。 值得关注的符号包括：lock, TensorIteratorConfig, add_output, add_input。

### Lines 541-570 / 第 541-570 行
```cpp
0541:       .check_all_same_dtype(false)
0542:       .build();
0543:     cpu_serial_kernel(iter2, [](double gamma_val, double gamma_sum_val) -> scalar_t{
0544:       auto ret_val = gamma_val / gamma_sum_val;
0545:       auto min_val = std::numeric_limits<scalar_t>::min();
0546:       auto max_val = std::nexttoward(static_cast<scalar_t>(1.0f), 0.0f);
0547:       return std::min(max_val, std::max(min_val, static_cast<scalar_t>(ret_val)));
0548:     });
0549:   });
0550:   return ret;
0551: }
0552: 
0553: /* The largest consecutive integer representable in float32 (2^24) */
0554: constexpr int64_t FLOAT32_MAX_CONSECUTIVE_INT = 1 << (FLT_MANT_DIG);
0555: 
0556: Tensor& multinomial_out(const Tensor& self,
0557:     int64_t n_sample,
0558:     bool with_replacement,
0559:     std::optional<Generator> gen,
0560:     Tensor& result) {
0561:   TORCH_CHECK(
0562:       result.device() == self.device(),
0563:       "multinomial arguments must have the same device");
0564:   TORCH_CHECK(
0565:       self.dim() > 0 && self.dim() <= 2, "prob_dist must be 1 or 2 dim");
0566:   TORCH_CHECK(
0567:       at::isFloatingType(self.scalar_type()),
0568:       "multinomial only supports floating-point dtypes for input, got: ",
0569:       self.scalar_type());
0570:   TORCH_CHECK(result.scalar_type() == ScalarType::Long,
```
- **EN**: Lines 541-570 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: check_all_same_dtype, build, cpu_serial_kernel, min.
- **CN**: 第 541-570 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：check_all_same_dtype, build, cpu_serial_kernel, min。

### Lines 571-600 / 第 571-600 行
```cpp
0571:       "multinomial expects Long tensor out, got: ", result.scalar_type());
0572:   TORCH_CHECK(n_sample > 0, "cannot sample n_sample <= 0 samples");
0573:   int64_t n_categories = self.size(-1);
0574:   TORCH_CHECK(with_replacement || (n_sample <= n_categories),
0575:       "cannot sample n_sample > prob_dist.size(-1) samples without replacement");
0576:   // Since the index tensor is float, numCategories cannot exceed max
0577:   // float integer precision
0578:   TORCH_CHECK(
0579:       n_categories <= FLOAT32_MAX_CONSECUTIVE_INT,
0580:       "number of categories cannot exceed 2^24");
0581: 
0582:   if (self.dim() == 1) {
0583:     result.resize_({n_sample});
0584:   } else {
0585:     const int64_t n_dist = self.size(0);
0586:     result.resize_({n_dist, n_sample});
0587:   }
0588:   if (result.numel() == 0) {
0589:     return result;
0590:   }
0591: 
0592:   // Fast-path for no replacement or if only one sample is drawn.
0593:   // Reference:
0594:   // https://github.com/pytorch/pytorch/issues/11931#issuecomment-625882503
0595:   if (!with_replacement || n_sample == 1) {
0596:     // Sanity checks on `self`.
0597:     auto [self_min, self_max] = self.aminmax();
0598:     auto is_valid = ((self_max < INFINITY) & (self_min >= 0));
0599:     at::_assert_async(is_valid, "probability tensor contains either `inf`, `nan` or element < 0");
0600:     at::Tensor zero_prob_condition;
```
- **EN**: Lines 571-600 mainly cover state/variable declarations, comments/documentation, macro-based glue. Notable symbols: scalar_type, TORCH_CHECK, size, dim.
- **CN**: 第 571-600 行主要涉及变量/别名声明、注释或说明、宏定义或宏调用。 值得关注的符号包括：scalar_type, TORCH_CHECK, size, dim。

### Lines 601-630 / 第 601-630 行
```cpp
0601:     if (self.dim() == 1){
0602:       zero_prob_condition = (self.sum() == 0);
0603:     } else {
0604:       zero_prob_condition = (self.sum(1) == 0).any();
0605:     }
0606:     at::_assert_async(~zero_prob_condition, "invalid multinomial distribution (sum of probabilities <= 0)");
0607: 
0608:     // The algorithm is from gumbel softmax.
0609:     // s = argmax( logp - log(-log(eps)) ) where eps ~ U(0, 1)
0610:     // Here we can apply exp to the formula which will not affect result of
0611:     // argmax or topk. Then we have
0612:     // s = argmax( p / (-log(eps)) ) where eps ~ U(0, 1).
0613:     // We can also simplify the formula above by
0614:     // s = argmax( p / q ) where q ~ Exp(1)
0615:     Tensor q = at::empty_like(self).exponential_(1, std::move(gen));
0616:     // In theory the probability to generate 0 from exponential distribution is
0617:     // 0. However, on CUDA side there is a protection to avoid 0s, but on CPU
0618:     // side, there is a very low probability to generate 0 from
0619:     // exponential<double>. The probability is about 2^(-DBL_MANT_DIG). We just
0620:     // ignore it here, but there may be some risk to get invalid output on CPU.
0621:     at::div_out(q, self, q);
0622:     if (n_sample == 1) {
0623:       at::argmax_out(result, q, /*dim=*/-1, /*keepdim=*/true);
0624:     } else {
0625:       Tensor vals = at::empty(result.sizes(), self.options());
0626:       at::topk_out(vals, result, q, n_sample);
0627:     }
0628:     return result;
0629:   }
0630: 
```
- **EN**: Lines 601-630 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: dim, sum, any, _assert_async.
- **CN**: 第 601-630 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：dim, sum, any, _assert_async。

### Lines 631-646 / 第 631-646 行
```cpp
0631:   multinomial_with_replacement_stub(
0632:       result.device().type(), result, self, n_sample, gen);
0633:   return result;
0634: }
0635: 
0636: Tensor multinomial(
0637:     const Tensor& self,
0638:     int64_t n_sample,
0639:     bool with_replacement,
0640:     std::optional<Generator> gen) {
0641:   Tensor result = at::empty({0}, self.options().dtype(kLong));
0642:   native::multinomial_out(self, n_sample, with_replacement, std::move(gen), result);
0643:   return result;
0644: }
0645: 
0646: } // namespace at::native
```
- **EN**: Lines 631-646 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: multinomial_with_replacement_stub, device, type, multinomial.
- **CN**: 第 631-646 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：multinomial_with_replacement_stub, device, type, multinomial。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/TensorIterator.h>`, `<ATen/TensorOperators.h>`, `<c10/util/Exception.h>`, `<optional>`, `<ATen/CPUGeneratorImpl.h>`, `<ATen/core/DistributionsHelper.h>`, `<ATen/native/Distributions.h>`, `<ATen/native/DispatchStub.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `std::`
