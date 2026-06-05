# GemmCommon.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/cuda/tunable/GemmCommon.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-side ATen helpers, kernels, runtime wrappers, or performance utilities. This specific file centers on `GemmCommon.h`. The file header highlights: "Original TunableOp is from onnxruntime. Adapting TunableOp into PyTorch." Autotuning, benchmark selection, or runtime choice of fast kernels is part of the responsibility. Matrix multiplication and GEMM-style kernels are a central concern.
- **Purpose (CN)**: 实现 CUDA 侧的 ATen 辅助逻辑、内核、运行时包装器或性能工具。 该文件具体围绕 `GemmCommon.h` 展开。 文件头部注释也概括了其核心职责。 该文件还负责自动调优、基准测试后的最优实现选择，或运行时快速内核的挑选。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

```cpp
0001: // Original TunableOp is from onnxruntime.
0002: // https://github.com/microsoft/onnxruntime/blob/main/onnxruntime/core/framework/tunable.h
0003: // https://github.com/microsoft/onnxruntime/tree/main/onnxruntime/core/providers/rocm/tunable
0004: // Copyright (c) Microsoft Corporation.
0005: // Licensed under the MIT license.
0006: //
0007: // Adapting TunableOp into PyTorch
0008: // Copyright (c) Advanced Micro Devices, Inc.
0009: //
0010: #pragma once
0011: 
0012: #include <string>
0013: #include <c10/core/ScalarType.h>
0014: 
0015: #include <ATen/cuda/tunable/TunableOp.h>
0016: #include <ATen/cuda/tunable/Tunable.h>
0017: #include <ATen/cuda/CUDABlas.h>
0018: #include <ATen/cuda/Exceptions.h>
0019: #include <c10/util/StringUtil.h>
0020: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。

### Lines 21-40 / 第 21-40 行

```cpp
0021: #ifndef AT_PER_OPERATOR_HEADERS
0022: #include <ATen/Functions.h>
0023: #include <ATen/NativeFunctions.h>
0024: #else
0025: #include <ATen/ops/allclose.h>
0026: #include <ATen/ops/from_blob.h>
0027: #endif
0028: #include <ATen/OpMathType.h>
0029: #include <fmt/printf.h>
0030: 
0031: namespace at::cuda::tunable {
0032: 
0033: using at::blas::ScalingType;
0034: 
0035: enum class BlasOp {
0036:   N = 0,
0037:   T = 1
0038: };
0039: 
0040: inline char BlasOpToString(BlasOp op) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `BlasOp`, `BlasOpToString`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`BlasOp`, `BlasOpToString`。

### Lines 41-60 / 第 41-60 行

```cpp
0041:   switch (op) {
0042:     case BlasOp::N:
0043:       return 'N';
0044:     case BlasOp::T:
0045:       return 'T';
0046:   }
0047:   TORCH_CHECK(false, "unrecognized BlasOp");
0048:   return 'N';
0049: }
0050: 
0051: template <typename T>
0052: inline const char* BLASTypeName(T v) {
0053:   return "unknown";
0054: }
0055: 
0056: template <>
0057: inline const char* BLASTypeName(float v) {
0058:   return "f32_r";
0059: }
0060: 
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: `BLASTypeName`.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：`BLASTypeName`。

### Lines 61-81 / 第 61-81 行

```cpp
0061: template <>
0062: inline const char* BLASTypeName(double v) {
0063:   return "f64_r";
0064: }
0065: 
0066: template <>
0067: inline const char* BLASTypeName(BFloat16 v) {
0068:   return "bf16_r";
0069: }
0070: 
0071: template <>
0072: inline const char* BLASTypeName(Half v) {
0073:   return "f16_r";
0074: }
0075: 
0076: //https://github.com/ROCm/hipBLASLt/blob/develop/library/src/include/auxiliary.hpp#L175
0077: template <>
0078: inline const char* BLASTypeName(Float8_e4m3fn v) {
0079:   return "f8_r";
0080: }
0081: 
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: `BLASTypeName`.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：`BLASTypeName`。

### Lines 82-101 / 第 82-101 行

```cpp
0082: template <>
0083: inline const char* BLASTypeName(Float8_e5m2 v) {
0084:   return "bf8_r";
0085: }
0086: 
0087: template <>
0088: inline const char* BLASTypeName(Float8_e4m3fnuz v) {
0089:   return "f8_fnuz_r";
0090: }
0091: 
0092: template <>
0093: inline const char* BLASTypeName(Float8_e5m2fnuz v) {
0094:   return "bf8_fnuz_r";
0095: }
0096: 
0097: template <>
0098: inline const char* BLASTypeName(c10::complex<double> v) {
0099:   return "f64_r";
0100: }
0101: 
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: `BLASTypeName`.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：`BLASTypeName`。

### Lines 102-141 / 第 102-141 行

```cpp
0102: template <>
0103: inline const char* BLASTypeName(c10::complex<float> v) {
0104:   return "f32_r";
0105: }
0106: 
0107: inline std::string ScalarTypeToBLASType(c10::ScalarType scalar_type) {
0108:   std::string BLASType;
0109:   switch (scalar_type) {
0110:     case c10::ScalarType::Float:{
0111:       BLASType = "f32_r";
0112:       break;
0113:     }
0114:     case c10::ScalarType::Double:{
0115:       BLASType = "f64_r";
0116:       break;
0117:     }
0118:     case c10::ScalarType::BFloat16:{
0119:       BLASType = "bf16_r";
0120:       break;
0121:     }
0122:     case c10::ScalarType::Half: {
0123:       BLASType = "f16_r";
0124:       break;
0125:     }
0126:     case c10::ScalarType::Float8_e4m3fn: {
0127:       BLASType = "f8_r";
0128:       break;
0129:     }
0130:     case c10::ScalarType::Float8_e5m2: {
0131:       BLASType = "bf8_r";
0132:       break;
0133:     }
0134:     case c10::ScalarType::Float8_e4m3fnuz: {
0135:       BLASType = "f8_fnuz_r";
0136:       break;
0137:     }
0138:     case c10::ScalarType::Float8_e5m2fnuz: {
0139:       BLASType = "bf8_fnuz_r";
0140:       break;
0141:     }
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: `BLASTypeName`, `ScalarTypeToBLASType`.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：`BLASTypeName`, `ScalarTypeToBLASType`。

### Lines 142-162 / 第 142-162 行

```cpp
0142:     case c10::ScalarType::ComplexFloat:{
0143:       BLASType = "f32_c";
0144:       break;
0145:     }
0146:     case c10::ScalarType::ComplexDouble:{
0147:       BLASType = "f64_c";
0148:       break;
0149:     }
0150:     default:
0151:       BLASType = "unknown";
0152:   }
0153:   return BLASType;
0154: 
0155: }
0156: 
0157: // Similar to Compute Type in GemmRocblas.h
0158: template <typename T>
0159: inline std::string ComputeTypeFor() {
0160:   return "Unknown ComputeType";
0161: }
0162: 
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: `ComputeTypeFor`.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：`ComputeTypeFor`。

### Lines 163-183 / 第 163-183 行

```cpp
0163: // This is a union of the compute types for
0164: // ROCBLAS and hipBLASLt.
0165: template <>
0166: inline std::string ComputeTypeFor<float>() {
0167:   if (at::globalContext().float32Precision(at::Float32Backend::CUDA, at::Float32Op::MATMUL) != at::Float32Precision::TF32) {
0168:     return "f32_r";
0169:   } else {
0170:     return "xf32_r";
0171:   }
0172: }
0173: 
0174: template <>
0175: inline std::string ComputeTypeFor<double>() {
0176:   return "f64_r";
0177: }
0178: 
0179: template <>
0180: inline std::string ComputeTypeFor<Half>() {
0181:   return "f32_r";
0182: }
0183: 
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: no prominent local symbols.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：无明显局部符号。

### Lines 184-203 / 第 184-203 行

```cpp
0184: template <>
0185: inline std::string ComputeTypeFor<BFloat16>() {
0186:   return "f32_r";
0187: }
0188: 
0189: template <>
0190: inline std::string ComputeTypeFor<c10::complex<float>>() {
0191:   return "f32_c";
0192: }
0193: 
0194: template <>
0195: inline std::string ComputeTypeFor<c10::complex<double>>() {
0196:   return "f64_c";
0197: }
0198: 
0199: template <>
0200: inline std::string ComputeTypeFor<Float8_e4m3fn>() {
0201:   return "f32_r";
0202: }
0203: 
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: no prominent local symbols.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：无明显局部符号。

### Lines 204-224 / 第 204-224 行

```cpp
0204: template <>
0205: inline std::string ComputeTypeFor<Float8_e5m2>() {
0206:   return "f32_r";
0207: }
0208: 
0209: template <>
0210: inline std::string ComputeTypeFor<Float8_e4m3fnuz>() {
0211:   return "f32_r";
0212: }
0213: 
0214: template <>
0215: inline std::string ComputeTypeFor<Float8_e5m2fnuz>() {
0216:   return "f32_r";
0217: }
0218: 
0219: // Convert opmath_type<T> to string
0220: template <typename T>
0221: inline std::string to_string_opmath(const at::opmath_type<T>& value) {
0222:     if constexpr (std::is_same_v<at::opmath_type<T>, c10::complex<float>> ||
0223:                   std::is_same_v<at::opmath_type<T>, c10::complex<double>>) {
0224:         return fmt::format("({:.4f}, {:.4f})", value.real(), value.imag());
```

- **EN:** Builds a reusable template/helper layer around `GemmCommon`. Key symbols: `to_string_opmath`, `constexpr`.
- **CN:** 围绕 `GemmCommon` 构建可复用的模板或辅助层。关键符号：`to_string_opmath`, `constexpr`。

### Lines 225-246 / 第 225-246 行

```cpp
0225:     } else {
0226:         return fmt::format("{:.4f}", value);
0227:     }
0228: }
0229: 
0230: // convert activation epilogue to string
0231: inline std::string to_string_epilogue(const at::cuda::blas::GEMMAndBiasActivationEpilogue& value) {
0232:   switch (value) {
0233:     case at::cuda::blas::GEMMAndBiasActivationEpilogue::None:
0234:       return std::string("None");
0235:       break;
0236:     case at::cuda::blas::GEMMAndBiasActivationEpilogue::RELU:
0237:       return std::string("RELU");
0238:       break;
0239:     case cuda::blas::GEMMAndBiasActivationEpilogue::GELU:
0240:       return std::string("GELU");
0241:       break;
0242:     default:
0243:       return std::string("unknown");
0244:   }
0245: }
0246: 
```

- **EN:** This block produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `to_string_epilogue`, `string`.
- **CN:** 该代码块返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`to_string_epilogue`, `string`。

### Lines 247-267 / 第 247-267 行

```cpp
0247: namespace detail {
0248: 
0249: static bool NumericalCheck(ScalarType dtype, void* c, void* other_c, int64_t size, const NumericalCheckConfig& config) {
0250: 
0251:   if (!config.enabled) {
0252:     return true; // skip when disabled
0253:   }
0254: 
0255:   auto options = at::TensorOptions().dtype(dtype).device(at::kCUDA);
0256:   at::Tensor ref = at::from_blob(c,       {size}, options);
0257:   at::Tensor oth = at::from_blob(other_c, {size}, options);
0258:   at::Tensor ref_float = ref.to(at::kFloat);
0259:   at::Tensor oth_float = oth.to(at::kFloat);
0260: 
0261:   const bool ok = at::allclose(ref_float, oth_float, config.rtol, config.atol);
0262:   if (ok) {
0263:     TUNABLE_LOG3("├──verify numerics: PASSED with atol=", config.atol, ", rtol=", config.rtol);
0264:   } else {
0265:     TUNABLE_LOG3("├──verify numerics: FAILED with atol=", config.atol, ", rtol=", config.rtol);
0266:   }
0267:   return ok;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; validates runtime invariants before continuing. Key symbols: `NumericalCheck`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；在继续执行前校验运行时不变量。关键符号：`NumericalCheck`。

### Lines 268-287 / 第 268-287 行

```cpp
0268: }
0269: 
0270: }
0271: 
0272: // Note on GetSizeA et al.
0273: // Tensors can be dense or arbitrarily strided. We only need our copies to be large enough.
0274: // Our copies must be at least as large as the m n k shapes dictate, but could be larger
0275: // depending on the lda ldb ldc values. Similarly for the batched case.
0276: 
0277: template <typename T>
0278: struct GemmParams : OpParams {
0279:   GemmParams() = default;
0280:   GemmParams(const GemmParams&) = default;
0281:   GemmParams& operator=(const GemmParams&) = default;
0282:   ~GemmParams() override = default;
0283: 
0284:   std::string BLASSignature() const override {
0285:     std::string alpha_str = to_string_opmath<T>(alpha);
0286:     std::string beta_str = to_string_opmath<T>(beta);
0287:     return fmt::sprintf("- { function: matmul, M: %ld, N: %ld, K: %ld, lda: %ld, ldb: %ld, ldc: %ld, ldd: %ld, stride_a: 0, stride_b: 0, stride_c: 0, stride_d: 0, "
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `GemmParams`, `BLASSignature`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`GemmParams`, `BLASSignature`。

### Lines 288-308 / 第 288-308 行

```cpp
0288:       "alpha: %s, beta: %s, transA: %c, transB: %c, batch_count: 1, a_type: %s, b_type: %s, c_type: %s, d_type: %s, scale_type: %s, bias_type: %s, compute_type: %s }",
0289:       m, n, k, lda, ldb, ldc, ldc, alpha_str, beta_str, transa, transb,
0290:       BLASTypeName<T>(T{}), BLASTypeName<T>(T{}), BLASTypeName<T>(T{}), BLASTypeName<T>(T{}), ComputeTypeFor<T>(), ComputeTypeFor<T>(), ComputeTypeFor<T>());
0291:   }
0292: 
0293:   std::string Signature() const override {
0294:     return fmt::sprintf("%c%c_%ld_%ld_%ld_ld_%ld_%ld_%ld", transa, transb, m, n, k, lda, ldb, ldc);
0295:   }
0296: 
0297:   size_t GetSizeA() const {
0298:     size_t size_stride = lda * ((transa == 'n' || transa == 'N') ? k : m);
0299:     size_t size_dense = m * k;
0300:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0301:   }
0302: 
0303:   size_t GetSizeB() const {
0304:     size_t size_stride = ldb * ((transb == 'n' || transb == 'N') ? n : k);
0305:     size_t size_dense = k * n;
0306:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0307:   }
0308: 
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: `Signature`, `sprintf`, `GetSizeA`, `GetSizeB`.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：`Signature`, `sprintf`, `GetSizeA`, `GetSizeB`。

### Lines 309-332 / 第 309-332 行

```cpp
0309:   size_t GetSizeC() const {
0310:     size_t size_stride = ldc * n;
0311:     size_t size_dense = m * n;
0312:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0313:   }
0314: 
0315:   size_t GetSize(bool duplicate_inputs) const {
0316:     size_t size = GetSizeC();
0317:     if (duplicate_inputs) {
0318:       size += GetSizeA();
0319:       size += GetSizeB();
0320:     }
0321:     return size;
0322:   }
0323: 
0324:   GemmParams* DeepCopy(bool duplicate_inputs) const {
0325:     GemmParams* copy = new GemmParams(*this);
0326:     c10::DeviceIndex device = 0;
0327:     AT_CUDA_CHECK(c10::cuda::GetDevice(&device));
0328:     size_t c_size = GetSizeC();
0329:     copy->c = static_cast<T*>(c10::cuda::CUDACachingAllocator::raw_alloc(c_size));
0330:     AT_CUDA_CHECK(c10::cuda::CUDACachingAllocator::memcpyAsync(
0331:         copy->c, device, c, device, c_size, getCurrentCUDAStream(device), true));
0332:     if (duplicate_inputs) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `GetSizeC`, `GetSize`, `DeepCopy`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`GetSizeC`, `GetSize`, `DeepCopy`。

### Lines 333-352 / 第 333-352 行

```cpp
0333:       size_t a_size = GetSizeA();
0334:       size_t b_size = GetSizeB();
0335:       copy->a = static_cast<const T*>(c10::cuda::CUDACachingAllocator::raw_alloc(a_size));
0336:       copy->b = static_cast<const T*>(c10::cuda::CUDACachingAllocator::raw_alloc(b_size));
0337:       copy->duplicate_inputs_ = true;
0338:     }
0339:     return copy;
0340:   }
0341: 
0342:   // only call on object returned by DeepCopy
0343:   void Delete() {
0344:     c10::cuda::CUDACachingAllocator::raw_delete(c);
0345:     if (duplicate_inputs_) {
0346:       // NOLINTNEXTLINE(*const-cast*)
0347:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<T*>(a));
0348:       // NOLINTNEXTLINE(*const-cast*)
0349:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<T*>(b));
0350:     }
0351:   }
0352: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `Delete`, `raw_delete`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`Delete`, `raw_delete`。

### Lines 353-375 / 第 353-375 行

```cpp
0353:   TuningStatus NumericalCheck(GemmParams<T> *other) {
0354:     auto* ctx = getTuningContext();
0355:     auto cfg = ctx->GetNumericalCheckConfig();
0356:     auto c_dtype = c10::CppTypeToScalarType<T>::value;
0357:     return detail::NumericalCheck(c_dtype, c, other->c, GetSizeC()/sizeof(T), cfg) ? OK : FAIL;
0358:   }
0359: 
0360:   char transa{};
0361:   char transb{};
0362:   int64_t m{};
0363:   int64_t n{};
0364:   int64_t k{};
0365:   at::opmath_type<T> alpha;
0366:   const T* a{};
0367:   int64_t lda{};
0368:   const T* b{};
0369:   int64_t ldb{};
0370:   at::opmath_type<T> beta;
0371:   T* c{};
0372:   int64_t ldc{};
0373: private:
0374:   bool duplicate_inputs_{false};
0375: };
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `NumericalCheck`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`NumericalCheck`。

### Lines 376-396 / 第 376-396 行

```cpp
0376: 
0377: template <typename T>
0378: struct GemmAndBiasParams : OpParams {
0379:   GemmAndBiasParams() = default;
0380:   GemmAndBiasParams(const GemmAndBiasParams&) = default;
0381:   GemmAndBiasParams(GemmAndBiasParams&&) noexcept = default;
0382:   GemmAndBiasParams& operator=(const GemmAndBiasParams&) = default;
0383:   GemmAndBiasParams& operator=(GemmAndBiasParams&&) noexcept = default;
0384:   ~GemmAndBiasParams() override = default;
0385: 
0386:   std::string BLASSignature() const override {
0387:     std::string alpha_str = to_string_opmath<T>(alpha);
0388:     std::string activation_str = to_string_epilogue(activation);
0389:     return fmt::sprintf("- { function: matmul, M: %ld, N: %ld, K: %ld, lda: %ld, ldb: %ld, ldc: %ld, ldd: %ld, stride_a: 0, stride_b: 0, stride_c: 0, stride_d: 0, "
0390:       "alpha: %s, transA: %c, transB: %c, batch_count: 1, a_type: %s, b_type: %s, c_type: %s, d_type: %s, activation: %s, bias_type: %s, scale_type: %s, compute_type: %s }",
0391:       m, n, k, lda, ldb, ldc, ldc, alpha_str, transa, transb,
0392:       BLASTypeName<T>(T{}), BLASTypeName<T>(T{}), BLASTypeName<T>(T{}), BLASTypeName<T>(T{}), activation_str, BLASTypeName<T>(T{}), ComputeTypeFor<T>(), ComputeTypeFor<T>(), ComputeTypeFor<T>());
0393:   }
0394: 
0395:   std::string Signature() const override {
0396:     return fmt::sprintf("%c%c_%ld_%ld_%ld_ld_%ld_%ld_%ld", transa, transb, m, n, k, lda, ldb, ldc);
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `GemmAndBiasParams`, `BLASSignature`, `Signature`, `sprintf`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`GemmAndBiasParams`, `BLASSignature`, `Signature`, `sprintf`。

### Lines 397-416 / 第 397-416 行

```cpp
0397:   }
0398: 
0399:   size_t GetSizeA() const {
0400:     size_t size_stride = lda * ((transa == 'n' || transa == 'N') ? k : m);
0401:     size_t size_dense = m * k;
0402:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0403:   }
0404: 
0405:   size_t GetSizeB() const {
0406:     size_t size_stride = ldb * ((transb == 'n' || transb == 'N') ? n : k);
0407:     size_t size_dense = k * n;
0408:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0409:   }
0410: 
0411:   size_t GetSizeC() const {
0412:     size_t size_stride = ldc * n;
0413:     size_t size_dense = m * n;
0414:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0415:   }
0416: 
```

- **EN:** This block produces a result or forwards a computed value. Key symbols: `GetSizeA`, `GetSizeB`, `GetSizeC`.
- **CN:** 该代码块返回结果或转发已计算的值。关键符号：`GetSizeA`, `GetSizeB`, `GetSizeC`。

### Lines 417-441 / 第 417-441 行

```cpp
0417:   size_t GetSize(bool duplicate_inputs) const {
0418:     size_t size = GetSizeC();
0419:     if (duplicate_inputs) {
0420:       size += GetSizeA();
0421:       size += GetSizeB();
0422:     }
0423:     return size;
0424:   }
0425: 
0426:   GemmAndBiasParams* DeepCopy(bool duplicate_inputs) const {
0427:     GemmAndBiasParams* copy = new GemmAndBiasParams(*this);
0428:     c10::DeviceIndex device = 0;
0429:     AT_CUDA_CHECK(c10::cuda::GetDevice(&device));
0430:     size_t c_size = GetSizeC();
0431:     copy->c = static_cast<T*>(c10::cuda::CUDACachingAllocator::raw_alloc(c_size));
0432:     AT_CUDA_CHECK(c10::cuda::CUDACachingAllocator::memcpyAsync(
0433:         copy->c, device, c, device, c_size, getCurrentCUDAStream(device), true));
0434:     if (duplicate_inputs) {
0435:       size_t a_size = GetSizeA();
0436:       size_t b_size = GetSizeB();
0437:       copy->a = static_cast<const T*>(c10::cuda::CUDACachingAllocator::raw_alloc(a_size));
0438:       copy->b = static_cast<const T*>(c10::cuda::CUDACachingAllocator::raw_alloc(b_size));
0439:       copy->duplicate_inputs_ = true;
0440:     }
0441:     return copy;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `GetSize`, `DeepCopy`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`GetSize`, `DeepCopy`。

### Lines 442-461 / 第 442-461 行

```cpp
0442:   }
0443: 
0444:   // only call on object returned by DeepCopy
0445:   void Delete() {
0446:     c10::cuda::CUDACachingAllocator::raw_delete(c);
0447:     if (duplicate_inputs_) {
0448:       // NOLINTNEXTLINE(*const-cast)
0449:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<T*>(a));
0450:       // NOLINTNEXTLINE(*const-cast)
0451:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<T*>(b));
0452:     }
0453:   }
0454: 
0455:   TuningStatus NumericalCheck(GemmAndBiasParams<T> *other) {
0456:     auto* ctx = getTuningContext();
0457:     auto cfg = ctx->GetNumericalCheckConfig();
0458:     auto c_dtype = c10::CppTypeToScalarType<T>::value;
0459:     return detail::NumericalCheck(c_dtype, c, other->c, GetSizeC()/sizeof(T), cfg) ? OK : FAIL;
0460:   }
0461: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `Delete`, `raw_delete`, `NumericalCheck`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`Delete`, `raw_delete`, `NumericalCheck`。

### Lines 462-481 / 第 462-481 行

```cpp
0462:   char transa{};
0463:   char transb{};
0464:   int64_t m{};
0465:   int64_t n{};
0466:   int64_t k{};
0467:   at::opmath_type<T> alpha{};
0468:   const T* a{};
0469:   int64_t lda{};
0470:   const T* b{};
0471:   int64_t ldb{};
0472:   T* c{};
0473:   int64_t ldc{};
0474:   const T* bias{};
0475:   at::cuda::blas::GEMMAndBiasActivationEpilogue activation{};
0476: private:
0477:   bool duplicate_inputs_{false};
0478: };
0479: 
0480: template <typename T, typename C_Dtype = T>
0481: struct GemmStridedBatchedParams : OpParams {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `GemmStridedBatchedParams`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`GemmStridedBatchedParams`。

### Lines 482-501 / 第 482-501 行

```cpp
0482:   GemmStridedBatchedParams() = default;
0483:   GemmStridedBatchedParams(const GemmStridedBatchedParams&) = default;
0484:   GemmStridedBatchedParams(GemmStridedBatchedParams&&) noexcept = default;
0485:   GemmStridedBatchedParams& operator=(const GemmStridedBatchedParams&) = default;
0486:   GemmStridedBatchedParams& operator=(GemmStridedBatchedParams&&) noexcept = default;
0487:   ~GemmStridedBatchedParams() override = default;
0488: 
0489:   std::string BLASSignature() const override {
0490:     std::string alpha_str = to_string_opmath<T>(alpha);
0491:     std::string beta_str = to_string_opmath<T>(beta);
0492:     return fmt::sprintf("- { function: matmul, M: %ld, N: %ld, K: %ld, lda: %ld, ldb: %ld, ldc: %ld, ldd: %ld, stride_a: %ld, stride_b: %ld, stride_c: %ld, stride_d: %ld, "
0493:       "alpha: %s, beta: %s, transA: %c, transB: %c, batch_count: %ld, a_type: %s, b_type: %s, c_type: %s, d_type: %s, scale_type: %s, compute_type: %s }",
0494:       m, n, k, lda, ldb, ldc, ldc, stride_a, stride_b, stride_c, stride_c, alpha_str, beta_str, transa, transb, batch,
0495:       BLASTypeName<T>(T{}), BLASTypeName<T>(T{}), BLASTypeName<C_Dtype>(C_Dtype{}), BLASTypeName<T>(T{}), ComputeTypeFor<T>(), ComputeTypeFor<T>());
0496:   }
0497: 
0498:   std::string Signature() const override {
0499:     return fmt::sprintf("%c%c_%ld_%ld_%ld_B_%ld_ld_%ld_%ld_%ld", transa, transb, m, n, k, batch, lda, ldb, ldc);
0500:   }
0501: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `BLASSignature`, `Signature`, `sprintf`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`BLASSignature`, `Signature`, `sprintf`。

### Lines 502-522 / 第 502-522 行

```cpp
0502:   size_t GetSizeA() const {
0503:     size_t size_stride = stride_a * batch;
0504:     size_t size_dense = m * k * batch;
0505:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0506:   }
0507: 
0508:   size_t GetSizeB() const {
0509:     size_t size_stride = stride_b * batch;
0510:     size_t size_dense = k * n * batch;
0511:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0512:   }
0513: 
0514:   size_t GetSizeC() const {
0515:     size_t size_stride = stride_c * batch;
0516:     size_t size_dense = m * n * batch;
0517:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0518:   }
0519: 
0520:   size_t GetSize(bool duplicate_inputs) const {
0521:     size_t size = GetSizeC();
0522:     if (duplicate_inputs) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `GetSizeA`, `GetSizeB`, `GetSizeC`, `GetSize`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`GetSizeA`, `GetSizeB`, `GetSizeC`, `GetSize`。

### Lines 523-546 / 第 523-546 行

```cpp
0523:       size += GetSizeA();
0524:       size += GetSizeB();
0525:     }
0526:     return size;
0527:   }
0528: 
0529:   GemmStridedBatchedParams* DeepCopy(bool duplicate_inputs) const {
0530:     GemmStridedBatchedParams* copy = new GemmStridedBatchedParams(*this);
0531:     c10::DeviceIndex device = 0;
0532:     AT_CUDA_CHECK(c10::cuda::GetDevice(&device));
0533:     size_t c_size = GetSizeC();
0534:     copy->c = static_cast<C_Dtype*>(c10::cuda::CUDACachingAllocator::raw_alloc(c_size));
0535:     AT_CUDA_CHECK(c10::cuda::CUDACachingAllocator::memcpyAsync(
0536:         copy->c, device, c, device, c_size, getCurrentCUDAStream(device), true));
0537:     if (duplicate_inputs) {
0538:       size_t a_size = GetSizeA();
0539:       size_t b_size = GetSizeB();
0540:       // NOLINTNEXTLINE(*const-cast*)
0541:       copy->a = static_cast<const T*>(c10::cuda::CUDACachingAllocator::raw_alloc(a_size));
0542:       // NOLINTNEXTLINE(*const-cast*)
0543:       copy->b = static_cast<const T*>(c10::cuda::CUDACachingAllocator::raw_alloc(b_size));
0544:       copy->duplicate_inputs_ = true;
0545:     }
0546:     return copy;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `DeepCopy`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`DeepCopy`。

### Lines 547-566 / 第 547-566 行

```cpp
0547:   }
0548: 
0549:   // only call on object returned by DeepCopy
0550:   void Delete() {
0551:     c10::cuda::CUDACachingAllocator::raw_delete(c);
0552:     if (duplicate_inputs_) {
0553:       // NOLINTNEXTLINE(*const-cast*)
0554:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<T*>(a));
0555:       // NOLINTNEXTLINE(*const-cast*)
0556:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<T*>(b));
0557:     }
0558:   }
0559: 
0560:   TuningStatus NumericalCheck(GemmStridedBatchedParams<T> *other) {
0561:     auto* ctx = getTuningContext();
0562:     auto cfg = ctx->GetNumericalCheckConfig();
0563:     auto c_dtype = c10::CppTypeToScalarType<C_Dtype>::value;
0564:     return detail::NumericalCheck(c_dtype, c, other->c, GetSizeC()/sizeof(T), cfg) ? OK : FAIL;
0565:   }
0566: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `Delete`, `raw_delete`, `NumericalCheck`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`Delete`, `raw_delete`, `NumericalCheck`。

### Lines 567-586 / 第 567-586 行

```cpp
0567:   char transa{};
0568:   char transb{};
0569:   int64_t m{};
0570:   int64_t n{};
0571:   int64_t k{};
0572:   at::opmath_type<T> alpha{};
0573:   const T* a{};
0574:   int64_t lda{};
0575:   int64_t stride_a{};
0576:   const T* b{};
0577:   int64_t ldb{};
0578:   int64_t stride_b{};
0579:   at::opmath_type<T> beta;
0580:   C_Dtype* c{};
0581:   int64_t ldc{};
0582:   int64_t stride_c{};
0583:   int64_t batch{};
0584: private:
0585:   bool duplicate_inputs_{false};
0586: };
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 587-607 / 第 587-607 行

```cpp
0587: 
0588: template <typename T>
0589: struct ScaledGemmParams : OpParams {
0590:   ScaledGemmParams() = default;
0591:   ScaledGemmParams(const ScaledGemmParams&) = default;
0592:   ScaledGemmParams(ScaledGemmParams&&) noexcept = default;
0593:   ScaledGemmParams& operator=(const ScaledGemmParams&) = default;
0594:   ScaledGemmParams& operator=(ScaledGemmParams&&) noexcept = default;
0595:   ~ScaledGemmParams() override = default;
0596: 
0597:   std::string BLASSignature() const override {
0598:     // Excluding use_fast_accum and use_rowise booleans for now
0599:     if (bias_ptr == nullptr) {
0600:       return fmt::sprintf("- { function: matmul, M: %ld, N: %ld, K: %ld, lda: %ld, ldb: %ld, ldc: %ld, ldd: %ld, stride_a: 0, stride_b: 0, stride_c: 0, stride_d: 0, "
0601:         "transA: %c, transB: %c, batch_count: 1, scaleA: f32_r, scaleB: f32_r, a_type: %s, b_type: %s, c_type: %s, d_type: %s, scale_type: %s, compute_type: %s }",
0602:         m, n, k, lda, ldb, ldc, ldc, transa, transb,
0603:         ScalarTypeToBLASType(a_dtype), ScalarTypeToBLASType(b_dtype), ScalarTypeToBLASType(c_dtype), ScalarTypeToBLASType(c_dtype),
0604:         ComputeTypeFor<T>(), ComputeTypeFor<T>());
0605:     }
0606:     else {
0607:       return fmt::sprintf("- { function: matmul, M: %ld, N: %ld, K: %ld, lda: %ld, ldb: %ld, ldc: %ld, ldd: %ld, stride_a: 0, stride_b: 0, stride_c: 0, stride_d: 0, "
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ScaledGemmParams`, `BLASSignature`, `ScalarTypeToBLASType`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ScaledGemmParams`, `BLASSignature`, `ScalarTypeToBLASType`。

### Lines 608-630 / 第 608-630 行

```cpp
0608:         "transA: %c, transB: %c, batch_count: 1, scaleA: f32_r, scaleB: f32_r, a_type: %s, b_type: %s, c_type: %s, d_type: %s, bias_type: %s, scale_type: %s, compute_type: %s }",
0609:         m, n, k, lda, ldb, ldc, ldc, transa, transb,
0610:         ScalarTypeToBLASType(a_dtype), ScalarTypeToBLASType(b_dtype), ScalarTypeToBLASType(c_dtype), ScalarTypeToBLASType(c_dtype), ScalarTypeToBLASType(bias_dtype),
0611:         ComputeTypeFor<T>(), ComputeTypeFor<T>());
0612:     }
0613:   }
0614: 
0615:   std::string Signature() const override {
0616:     // In Blas.cpp, code defaults to a bias_dtype of Half even when there is no bias vector.
0617:     // Search for this line::
0618:     // params.bias_dtype = bias ? bias->scalar_type() : isFloat8Type(out_dtype_) ? at::ScalarType::Half : out_dtype_;
0619:     //
0620:     // In TunableOp, we must distinguish in param signature these two cases: with and without a bias vector.
0621:     return fmt::sprintf("%c%c_%ld_%ld_%ld_ld_%ld_%ld_%ld_rw_%d_bias_%s",
0622:       transa, transb, m, n, k, lda, ldb, ldc,
0623:       a_scaling_type == ScalingType::RowWise && b_scaling_type == ScalingType::RowWise,
0624:       bias_ptr == nullptr ? "None" : at::toString(bias_dtype));
0625:   }
0626: 
0627:   size_t GetSizeA() const {
0628:     size_t size_stride = lda * ((transa == 'n' || transa == 'N') ? k : m);
0629:     size_t size_dense = m * k;
0630:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `ScalarTypeToBLASType`, `Signature`, `sprintf`, `GetSizeA`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`ScalarTypeToBLASType`, `Signature`, `sprintf`, `GetSizeA`。

### Lines 631-651 / 第 631-651 行

```cpp
0631:   }
0632: 
0633:   size_t GetSizeB() const {
0634:     size_t size_stride = ldb * ((transb == 'n' || transb == 'N') ? n : k);
0635:     size_t size_dense = k * n;
0636:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0637:   }
0638: 
0639:   size_t GetSizeC() const {
0640:     size_t size_stride = ldc * n;
0641:     size_t size_dense = m * n;
0642:     return sizeof(T) * (size_stride > size_dense ? size_stride : size_dense);
0643:   }
0644: 
0645:   size_t GetSize(bool duplicate_inputs) const {
0646:     size_t size = GetSizeC();
0647:     if (duplicate_inputs) {
0648:       size += GetSizeA();
0649:       size += GetSizeB();
0650:     }
0651:     return size;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `GetSizeB`, `GetSizeC`, `GetSize`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`GetSizeB`, `GetSizeC`, `GetSize`。

### Lines 652-671 / 第 652-671 行

```cpp
0652:   }
0653: 
0654:   ScaledGemmParams* DeepCopy(bool duplicate_inputs) const {
0655:     ScaledGemmParams* copy = new ScaledGemmParams(*this);
0656:     c10::DeviceIndex device = 0;
0657:     AT_CUDA_CHECK(c10::cuda::GetDevice(&device));
0658:     size_t c_size = GetSizeC();
0659:     copy->c = c10::cuda::CUDACachingAllocator::raw_alloc(c_size);
0660:     AT_CUDA_CHECK(c10::cuda::CUDACachingAllocator::memcpyAsync(
0661:         copy->c, device, c, device, c_size, getCurrentCUDAStream(device), true));
0662:     if (duplicate_inputs) {
0663:       size_t a_size = GetSizeA();
0664:       size_t b_size = GetSizeB();
0665:       copy->a = c10::cuda::CUDACachingAllocator::raw_alloc(a_size);
0666:       copy->b = c10::cuda::CUDACachingAllocator::raw_alloc(b_size);
0667:       copy->duplicate_inputs_ = true;
0668:     }
0669:     return copy;
0670:   }
0671: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `DeepCopy`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`DeepCopy`。

### Lines 672-711 / 第 672-711 行

```cpp
0672:   // only call on object returned by DeepCopy
0673:   void Delete() {
0674:     c10::cuda::CUDACachingAllocator::raw_delete(c);
0675:     if (duplicate_inputs_) {
0676:       // NOLINTNEXTLINE(*const-cast*)
0677:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<void*>(a));
0678:       // NOLINTNEXTLINE(*const-cast*)
0679:       c10::cuda::CUDACachingAllocator::raw_delete(const_cast<void*>(b));
0680:     }
0681:   }
0682: 
0683:   TuningStatus NumericalCheck(ScaledGemmParams<T> *other) {
0684:     auto* ctx = getTuningContext();
0685:     auto cfg = ctx->GetNumericalCheckConfig();
0686:     return detail::NumericalCheck(c_dtype, c, other->c, GetSizeC()/sizeof(T), cfg) ? OK : FAIL;
0687:   }
0688: 
0689:   char transa{};
0690:   char transb{};
0691:   int64_t m{};
0692:   int64_t n{};
0693:   int64_t k{};
0694:   const void* a{};
0695:   const void* a_scale_ptr{};
0696:   int64_t lda{};
0697:   ScalarType a_dtype{};
0698:   ScalarType a_scale_dtype{};
0699:   ScalingType a_scaling_type{};
0700:   const void* b{};
0701:   const void* b_scale_ptr{};
0702:   int64_t ldb{};
0703:   ScalarType b_dtype{};
0704:   ScalarType b_scale_dtype{};
0705:   ScalingType b_scaling_type{};
0706:   const void* bias_ptr{};
0707:   ScalarType bias_dtype{};
0708:   void* c{};
0709:   const void* c_scale_ptr{};
0710:   int64_t ldc{};
0711:   ScalarType c_dtype{};
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `Delete`, `raw_delete`, `NumericalCheck`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`Delete`, `raw_delete`, `NumericalCheck`。

### Lines 712-718 / 第 712-718 行

```cpp
0712:   void* amax_ptr{};
0713:   bool use_fast_accum{};
0714: private:
0715:   bool duplicate_inputs_{false};
0716: };
0717: 
0718: } // namespace at::cuda::tunable
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **CUDA backend plumbing** — CUDA 后端基础设施
- **Tensor metadata and value flow** — 张量元数据与数值流
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: BlasOp, GemmParams, GemmAndBiasParams, GemmStridedBatchedParams, ScaledGemmParams, BlasOpToString, BLASTypeName, ScalarTypeToBLASType** — 核心符号：BlasOp、GemmParams、GemmAndBiasParams、GemmStridedBatchedParams、ScaledGemmParams、BlasOpToString、BLASTypeName、ScalarTypeToBLASType

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `c10/core/ScalarType.h`, `ATen/cuda/tunable/TunableOp.h`, `ATen/cuda/tunable/Tunable.h`, `ATen/cuda/CUDABlas.h`, `ATen/cuda/Exceptions.h`, `c10/util/StringUtil.h`, `ATen/Functions.h`, `ATen/NativeFunctions.h`, `ATen/ops/allclose.h`, `ATen/ops/from_blob.h`, `ATen/OpMathType.h`
- **External includes / 外部头文件**: `string`, `fmt/printf.h`
- **Namespaces / 命名空间**: `at::cuda::tunable`, `detail`
- **Representative symbols / 代表性符号**: `BlasOp`, `GemmParams`, `GemmAndBiasParams`, `GemmStridedBatchedParams`, `ScaledGemmParams`, `BlasOpToString`, `BLASTypeName`, `ScalarTypeToBLASType`, `ComputeTypeFor`, `to_string_opmath`, `constexpr`, `to_string_epilogue`, `...`
