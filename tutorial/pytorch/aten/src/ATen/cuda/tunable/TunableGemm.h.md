# TunableGemm.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/cuda/tunable/TunableGemm.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-side ATen helpers, kernels, runtime wrappers, or performance utilities. This specific file centers on `TunableGemm.h`. The file header highlights: "Original TunableOp is from onnxruntime. Adapting TunableOp into PyTorch." Autotuning, benchmark selection, or runtime choice of fast kernels is part of the responsibility. Matrix multiplication and GEMM-style kernels are a central concern.
- **Purpose (CN)**: 实现 CUDA 侧的 ATen 辅助逻辑、内核、运行时包装器或性能工具。 该文件具体围绕 `TunableGemm.h` 展开。 文件头部注释也概括了其核心职责。 该文件还负责自动调优、基准测试后的最优实现选择，或运行时快速内核的挑选。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16 / 第 1-16 行

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
0012: #include <ATen/cuda/tunable/GemmCommon.h>
0013: #ifdef USE_ROCM
0014: #include <ATen/cuda/tunable/GemmHipblaslt.h>
0015: #include <ATen/cuda/tunable/GemmRocblas.h>
0016: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。

### Lines 17-30 / 第 17-30 行

```cpp
0017: #include <ATen/cuda/tunable/TunableOp.h>
0018: #include <c10/cuda/CUDACachingAllocator.h>
0019: #include <c10/util/Float8_e4m3fn.h>
0020: #include <c10/util/Float8_e4m3fnuz.h>
0021: #include <c10/util/Float8_e5m2.h>
0022: #include <c10/util/Float8_e5m2fnuz.h>
0023: #include <c10/util/Float8_e8m0fnu.h>
0024: #include <c10/util/StringUtil.h>
0025: #include <fmt/printf.h>
0026: 
0027: namespace at::cuda::tunable {
0028: 
0029: template <typename T>
0030: class DefaultGemmOp : public Callable<GemmParams<T>> {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `DefaultGemmOp`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`DefaultGemmOp`。

### Lines 31-44 / 第 31-44 行

```cpp
0031:   public:
0032:     TuningStatus Call(const GemmParams<T>* params) override {
0033:       at::cuda::blas::gemm_internal<T>(
0034:           params->transa, params->transb,
0035:           params->m, params->n, params->k,
0036:           params->alpha,
0037:           params->a, params->lda,
0038:           params->b, params->ldb,
0039:           params->beta,
0040:           params->c, params->ldc);
0041:       return OK;
0042:     }
0043: };
0044: 
```

- **EN:** This block produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `Call`.
- **CN:** 该代码块返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`Call`。

### Lines 45-63 / 第 45-63 行

```cpp
0045: static bool _transposeBoolFromChar(char op) {
0046:   return op == 't' || op == 'T';
0047: }
0048: 
0049: template <typename T>
0050: class DefaultGemmAndBiasOp : public Callable<GemmAndBiasParams<T>> {
0051:   public:
0052:     TuningStatus Call(const GemmAndBiasParams<T>* params) override {
0053:       at::cuda::blas::gemm_and_bias<T>(
0054:           _transposeBoolFromChar(params->transa),
0055:           _transposeBoolFromChar(params->transb),
0056:           params->m, params->n, params->k,
0057:           params->alpha,
0058:           params->a, params->lda,
0059:           params->b, params->ldb,
0060:           params->bias,
0061:           params->c, params->ldc,
0062:           params->activation);
0063:       return OK;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `DefaultGemmAndBiasOp`, `_transposeBoolFromChar`, `Call`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`DefaultGemmAndBiasOp`, `_transposeBoolFromChar`, `Call`。

### Lines 64-80 / 第 64-80 行

```cpp
0064:     }
0065: };
0066: 
0067: template <typename T>
0068: class DefaultGemmStridedBatchedOp : public Callable<GemmStridedBatchedParams<T>> {
0069:   public:
0070:     TuningStatus Call(const GemmStridedBatchedParams<T>* params) override {
0071:       at::cuda::blas::bgemm_internal<T>(
0072:           params->transa, params->transb,
0073:           params->m, params->n, params->k,
0074:           params->alpha,
0075:           params->a, params->lda, params->stride_a,
0076:           params->b, params->ldb, params->stride_b,
0077:           params->beta,
0078:           params->c, params->ldc, params->stride_c,
0079:           params->batch);
0080:       return OK;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `DefaultGemmStridedBatchedOp`, `Call`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`DefaultGemmStridedBatchedOp`, `Call`。

### Lines 81-108 / 第 81-108 行

```cpp
0081:     }
0082: };
0083: 
0084: template <typename T>
0085: class DefaultScaledGemmOp : public Callable<ScaledGemmParams<T>> {
0086:   public:
0087:     TuningStatus Call(const ScaledGemmParams<T>* params) override {
0088:       at::cuda::blas::scaled_gemm(
0089:           params->transa,
0090:           params->transb,
0091:           params->m,
0092:           params->n,
0093:           params->k,
0094:           params->a,
0095:           params->a_scale_ptr,
0096:           params->lda,
0097:           params->a_dtype,
0098:           params->a_scale_dtype,
0099:           params->a_scaling_type,
0100:           params->b,
0101:           params->b_scale_ptr,
0102:           params->ldb,
0103:           params->b_dtype,
0104:           params->b_scale_dtype,
0105:           params->b_scaling_type,
0106:           params->bias_ptr,
0107:           params->bias_dtype,
0108:           params->c,
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `DefaultScaledGemmOp`, `Call`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`DefaultScaledGemmOp`, `Call`。

### Lines 109-122 / 第 109-122 行

```cpp
0109:           params->c_scale_ptr,
0110:           params->ldc,
0111:           params->c_dtype,
0112:           params->use_fast_accum,
0113:           std::nullopt /* alpha */);
0114:       return OK;
0115:     }
0116: };
0117: 
0118: template <typename T>
0119: inline bool IsZero(T v) {
0120:   return v == 0.0f;
0121: }
0122: 
```

- **EN:** Builds a reusable template/helper layer around `TunableGemm`. Key symbols: `IsZero`.
- **CN:** 围绕 `TunableGemm` 构建可复用的模板或辅助层。关键符号：`IsZero`。

### Lines 123-137 / 第 123-137 行

```cpp
0123: template <>
0124: inline bool IsZero(BFloat16 v) {
0125:   return v.x == 0;
0126: }
0127: 
0128: template <>
0129: inline bool IsZero(Half v) {
0130:   return float(v) == 0.0f;
0131: }
0132: 
0133: template <>
0134: inline bool IsZero(c10::complex<double> v) {
0135:   return v == 0.0;
0136: }
0137: 
```

- **EN:** Builds a reusable template/helper layer around `TunableGemm`. Key symbols: `IsZero`.
- **CN:** 围绕 `TunableGemm` 构建可复用的模板或辅助层。关键符号：`IsZero`。

### Lines 138-151 / 第 138-151 行

```cpp
0138: template <>
0139: inline bool IsZero(c10::complex<float> v) {
0140:   return v == 0.0f;
0141: }
0142: 
0143: template <typename T>
0144: inline const char* TypeName(T v) {
0145:   return "unknown";
0146: }
0147: 
0148: template <>
0149: inline const char* TypeName(float v) {
0150:   if (at::globalContext().allowTF32CuBLAS()) {
0151:     return "tf32";
```

- **EN:** Builds a reusable template/helper layer around `TunableGemm`. Key symbols: `IsZero`, `TypeName`.
- **CN:** 围绕 `TunableGemm` 构建可复用的模板或辅助层。关键符号：`IsZero`, `TypeName`。

### Lines 152-166 / 第 152-166 行

```cpp
0152:   } else {
0153:     return "float";
0154:   }
0155: }
0156: 
0157: template <>
0158: inline const char* TypeName(double v) {
0159:   return "double";
0160: }
0161: 
0162: template <>
0163: inline const char* TypeName(BFloat16 v) {
0164:   return "BFloat16";
0165: }
0166: 
```

- **EN:** Builds a reusable template/helper layer around `TunableGemm`. Key symbols: `TypeName`.
- **CN:** 围绕 `TunableGemm` 构建可复用的模板或辅助层。关键符号：`TypeName`。

### Lines 167-181 / 第 167-181 行

```cpp
0167: template <>
0168: inline const char* TypeName(Half v) {
0169:   return "Half";
0170: }
0171: 
0172: template <>
0173: inline const char* TypeName(Float8_e4m3fn v) {
0174:   return "Float8_e4m3fn";
0175: }
0176: 
0177: template <>
0178: inline const char* TypeName(Float8_e5m2 v) {
0179:   return "Float8_e5m2";
0180: }
0181: 
```

- **EN:** Builds a reusable template/helper layer around `TunableGemm`. Key symbols: `TypeName`.
- **CN:** 围绕 `TunableGemm` 构建可复用的模板或辅助层。关键符号：`TypeName`。

### Lines 182-196 / 第 182-196 行

```cpp
0182: template <>
0183: inline const char* TypeName(Float8_e4m3fnuz v) {
0184:   return "Float8_e4m3fnuz";
0185: }
0186: 
0187: template <>
0188: inline const char* TypeName(Float8_e5m2fnuz v) {
0189:   return "Float8_e5m2fnuz";
0190: }
0191: 
0192: template <>
0193: inline const char* TypeName(Float8_e8m0fnu v) {
0194:   return "Float8_e8m0fnu";
0195: }
0196: 
```

- **EN:** Builds a reusable template/helper layer around `TunableGemm`. Key symbols: `TypeName`.
- **CN:** 围绕 `TunableGemm` 构建可复用的模板或辅助层。关键符号：`TypeName`。

### Lines 197-212 / 第 197-212 行

```cpp
0197: template <>
0198: inline const char* TypeName(c10::complex<double> v) {
0199:   return "c10::complex<double>";
0200: }
0201: 
0202: template <>
0203: inline const char* TypeName(c10::complex<float> v) {
0204:   return "c10::complex<float>";
0205: }
0206: 
0207: template <typename T, BlasOp ALayout, BlasOp BLayout>
0208: class GemmTunableOp : public TunableOp<GemmParams<T>> {
0209:  public:
0210:   GemmTunableOp() {
0211:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultGemmOp<T>>());
0212: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `GemmTunableOp`, `TypeName`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`GemmTunableOp`, `TypeName`。

### Lines 213-227 / 第 213-227 行

```cpp
0213: #ifdef USE_ROCM
0214:     static const auto env_rocblas = c10::utils::check_env("PYTORCH_TUNABLEOP_ROCBLAS_ENABLED");
0215:     if (!env_rocblas.has_value() || env_rocblas.value()) {
0216:       for (auto&& [name, op] : GetRocBlasGemmTypeStringAndOps<T>()) {
0217:         this->RegisterOp(std::move(name), std::move(op));
0218:       }
0219:     }
0220: 
0221:     static const auto env_hipblaslt = c10::utils::check_env("PYTORCH_TUNABLEOP_HIPBLASLT_ENABLED");
0222:     if (!env_hipblaslt.has_value() || env_hipblaslt.value()) {
0223:       // disallow tuning of hipblaslt with c10::complex
0224:       if constexpr (
0225:           !std::is_same_v<T, c10::complex<float>> &&
0226:           !std::is_same_v<T, c10::complex<double>>) {
0227:         for (auto&& [name, op] : GetHipBlasLtGemmTypeStringAndOps<T, ALayout, BLayout>()) {
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details. Key symbols: `constexpr`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`constexpr`。

### Lines 228-241 / 第 228-241 行

```cpp
0228:           this->RegisterOp(std::move(name), std::move(op));
0229:         }
0230:       }
0231:     }
0232: #endif
0233: 
0234:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultGemmOp<T>>());
0235:   }
0236: 
0237:   std::string Signature() override {
0238:     return fmt::sprintf("GemmTunableOp_%s_%c%c", TypeName<T>(T{}), BlasOpToString(ALayout), BlasOpToString(BLayout));
0239:   }
0240: };
0241: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `Signature`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`Signature`。

### Lines 242-255 / 第 242-255 行

```cpp
0242: template <typename T, BlasOp ALayout, BlasOp BLayout>
0243: class GemmAndBiasTunableOp : public TunableOp<GemmAndBiasParams<T>> {
0244:  public:
0245:   GemmAndBiasTunableOp() {
0246:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultGemmAndBiasOp<T>>());
0247: 
0248: #ifdef USE_ROCM
0249:     static const auto env_hipblaslt = c10::utils::check_env("PYTORCH_TUNABLEOP_HIPBLASLT_ENABLED");
0250:     if (!env_hipblaslt.has_value() || env_hipblaslt.value()) {
0251:       // disallow tuning of hipblaslt with c10::complex
0252:       if constexpr (
0253:           !std::is_same_v<T, c10::complex<float>> &&
0254:           !std::is_same_v<T, c10::complex<double>>) {
0255:         for (auto&& [name, op] : GetHipBlasLtGemmAndBiasTypeStringAndOps<T, ALayout, BLayout>()) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `GemmAndBiasTunableOp`, `constexpr`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`GemmAndBiasTunableOp`, `constexpr`。

### Lines 256-269 / 第 256-269 行

```cpp
0256:           this->RegisterOp(std::move(name), std::move(op));
0257:         }
0258:       }
0259:     }
0260: #endif
0261: 
0262:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultGemmAndBiasOp<T>>());
0263:   }
0264: 
0265:   std::string Signature() override {
0266:     return fmt::sprintf("GemmAndBiasTunableOp_%s_%c%c", TypeName<T>(T{}), BlasOpToString(ALayout), BlasOpToString(BLayout));
0267:   }
0268: };
0269: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `Signature`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`Signature`。

### Lines 270-283 / 第 270-283 行

```cpp
0270: template <typename T, BlasOp ALayout, BlasOp BLayout>
0271: class GemmStridedBatchedTunableOp : public TunableOp<GemmStridedBatchedParams<T>> {
0272:  public:
0273:   GemmStridedBatchedTunableOp() {
0274:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultGemmStridedBatchedOp<T>>());
0275: 
0276: #ifdef USE_ROCM
0277:     static const auto env_rocblas = c10::utils::check_env("PYTORCH_TUNABLEOP_ROCBLAS_ENABLED");
0278:     if (!env_rocblas.has_value() || env_rocblas.value()) {
0279:       for (auto&& [name, op] : GetRocBlasGemmStridedBatchedTypeStringAndOps<T>()) {
0280:         this->RegisterOp(std::move(name), std::move(op));
0281:       }
0282:     }
0283: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `GemmStridedBatchedTunableOp`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`GemmStridedBatchedTunableOp`。

### Lines 284-299 / 第 284-299 行

```cpp
0284:     static const auto env_hipblaslt = c10::utils::check_env("PYTORCH_TUNABLEOP_HIPBLASLT_ENABLED");
0285:     if (!env_hipblaslt.has_value() || env_hipblaslt.value()) {
0286:       // disallow tuning of hipblaslt with c10::complex
0287:       if constexpr (
0288:           !std::is_same_v<T, c10::complex<float>> &&
0289:           !std::is_same_v<T, c10::complex<double>>) {
0290:         for (auto&& [name, op] : GetHipBlasLtGemmStridedBatchedTypeStringAndOps<T, ALayout, BLayout>()) {
0291:           this->RegisterOp(std::move(name), std::move(op));
0292:         }
0293:       }
0294:     }
0295: #endif
0296: 
0297:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultGemmStridedBatchedOp<T>>());
0298:   }
0299: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details. Key symbols: `constexpr`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`constexpr`。

### Lines 300-315 / 第 300-315 行

```cpp
0300:   std::string Signature() override {
0301:     return fmt::sprintf("GemmStridedBatchedTunableOp_%s_%c%c", TypeName<T>(T{}), BlasOpToString(ALayout), BlasOpToString(BLayout));
0302:   }
0303: };
0304: 
0305: template <typename AT, typename BT, typename CT, BlasOp ALayout, BlasOp BLayout>
0306: class ScaledGemmTunableOp : public TunableOp<ScaledGemmParams<CT>> {
0307:  public:
0308:   ScaledGemmTunableOp() {
0309:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultScaledGemmOp<CT>>());
0310: 
0311: #ifdef USE_ROCM
0312:     for (auto&& [name, op] : GetHipBlasLtScaledGemmTypeStringAndOps<AT, BT, CT, ALayout, BLayout>()) {
0313:       this->RegisterOp(std::move(name), std::move(op));
0314:     }
0315: #endif
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ScaledGemmTunableOp`, `Signature`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ScaledGemmTunableOp`, `Signature`。

### Lines 316-329 / 第 316-329 行

```cpp
0316: 
0317:     this->RegisterOp(std::string("Default"), std::make_unique<DefaultScaledGemmOp<CT>>());
0318:   }
0319: 
0320:   std::string Signature() override {
0321:     return fmt::sprintf("ScaledGemmTunableOp_%s_%s_%s_%c%c",
0322:       TypeName<AT>(AT{}),
0323:       TypeName<BT>(BT{}),
0324:       TypeName<CT>(CT{}),
0325:       BlasOpToString(ALayout), BlasOpToString(BLayout));
0326:   }
0327: };
0328: 
0329: } // namespace at::cuda::tunable
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details. Key symbols: `Signature`, `BlasOpToString`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`Signature`, `BlasOpToString`。


## Key Concepts / 关键概念
- **CUDA backend plumbing** — CUDA 后端基础设施
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: DefaultGemmOp, DefaultGemmAndBiasOp, DefaultGemmStridedBatchedOp, DefaultScaledGemmOp, GemmTunableOp, GemmAndBiasTunableOp, GemmStridedBatchedTunableOp, ScaledGemmTunableOp** — 核心符号：DefaultGemmOp、DefaultGemmAndBiasOp、DefaultGemmStridedBatchedOp、DefaultScaledGemmOp、GemmTunableOp、GemmAndBiasTunableOp、GemmStridedBatchedTunableOp、ScaledGemmTunableOp

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/cuda/tunable/GemmCommon.h`, `ATen/cuda/tunable/GemmHipblaslt.h`, `ATen/cuda/tunable/GemmRocblas.h`, `ATen/cuda/tunable/TunableOp.h`, `c10/cuda/CUDACachingAllocator.h`, `c10/util/Float8_e4m3fn.h`, `c10/util/Float8_e4m3fnuz.h`, `c10/util/Float8_e5m2.h`, `c10/util/Float8_e5m2fnuz.h`, `c10/util/Float8_e8m0fnu.h`, `c10/util/StringUtil.h`
- **External includes / 外部头文件**: `fmt/printf.h`
- **Namespaces / 命名空间**: `at::cuda::tunable`
- **Representative symbols / 代表性符号**: `DefaultGemmOp`, `DefaultGemmAndBiasOp`, `DefaultGemmStridedBatchedOp`, `DefaultScaledGemmOp`, `GemmTunableOp`, `GemmAndBiasTunableOp`, `GemmStridedBatchedTunableOp`, `ScaledGemmTunableOp`, `Call`, `_transposeBoolFromChar`, `scaled_gemm`, `IsZero`, `...`
