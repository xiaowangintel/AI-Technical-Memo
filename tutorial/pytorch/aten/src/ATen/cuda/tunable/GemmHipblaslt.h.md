# GemmHipblaslt.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/cuda/tunable/GemmHipblaslt.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-side ATen helpers, kernels, runtime wrappers, or performance utilities. This specific file centers on `GemmHipblaslt.h`. Autotuning, benchmark selection, or runtime choice of fast kernels is part of the responsibility. Matrix multiplication and GEMM-style kernels are a central concern. Descriptor/handle lifecycle management is important here.
- **Purpose (CN)**: 实现 CUDA 侧的 ATen 辅助逻辑、内核、运行时包装器或性能工具。 该文件具体围绕 `GemmHipblaslt.h` 展开。 该文件还负责自动调优、基准测试后的最优实现选择，或运行时快速内核的挑选。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。 描述符/句柄的生命周期管理是这里的重要内容。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

```cpp
0001: // Copyright (c) Microsoft Corporation. All rights reserved.
0002: // Licensed under the MIT License.
0003: 
0004: #pragma once
0005: 
0006: #include <ATen/cuda/CUDAContext.h>
0007: #include <ATen/cuda/CUDADataType.h>
0008: #include <ATen/cuda/tunable/TunableOp.h>
0009: #include <ATen/cuda/tunable/GemmCommon.h>
0010: #include <c10/cuda/CUDACachingAllocator.h>
0011: #include <c10/util/StringUtil.h>
0012: #include <fmt/printf.h>
0013: 
0014: #include <hipblaslt/hipblaslt.h>
0015: #include <hipblaslt/hipblaslt-ext.hpp>
0016: 
0017: #define TORCH_HIPBLASLT_CHECK(EXPR)               \
0018:   do {                                            \
0019:     hipblasStatus_t __err = EXPR;                 \
0020:     TORCH_CHECK(__err == HIPBLAS_STATUS_SUCCESS,  \
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量。关键符号：无明显局部符号。

### Lines 21-40 / 第 21-40 行

```cpp
0021:                 "hipblaslt error: ",              \
0022:                 hipblasStatusToString(__err),     \
0023:                 " when calling `" #EXPR "`");     \
0024:   } while (0)
0025: 
0026: namespace at::cuda::tunable {
0027: 
0028: template <typename T>
0029: constexpr hipDataType HipDataTypeFor();
0030: 
0031: template <>
0032: constexpr hipDataType HipDataTypeFor<float>() {
0033:   return HIP_R_32F;
0034: }
0035: 
0036: template <>
0037: constexpr hipDataType HipDataTypeFor<Half>() {
0038:   return HIP_R_16F;
0039: }
0040: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `hipblasStatusToString`, `HipDataTypeFor`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`hipblasStatusToString`, `HipDataTypeFor`。

### Lines 41-60 / 第 41-60 行

```cpp
0041: template <>
0042: constexpr hipDataType HipDataTypeFor<BFloat16>() {
0043:   return HIP_R_16BF;
0044: }
0045: 
0046: template <>
0047: constexpr hipDataType HipDataTypeFor<double>() {
0048:   return HIP_R_64F;
0049: }
0050: 
0051: template <>
0052: constexpr hipDataType HipDataTypeFor<c10::Float8_e4m3fnuz>() {
0053:   return HIP_R_8F_E4M3_FNUZ;
0054: }
0055: 
0056: template <>
0057: constexpr hipDataType HipDataTypeFor<c10::Float8_e5m2fnuz>() {
0058:   return HIP_R_8F_E5M2_FNUZ;
0059: }
0060: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: no prominent local symbols.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：无明显局部符号。

### Lines 61-80 / 第 61-80 行

```cpp
0061: // This code is instantiated regardless of ROCm version.
0062: // Prior to ROCm 6.3, we hard-code the known enum values.
0063: template <>
0064: constexpr hipDataType HipDataTypeFor<c10::Float8_e4m3fn>() {
0065: #if ROCM_VERSION >= 60300
0066:   return HIP_R_8F_E4M3;
0067: #else
0068:   return static_cast<hipDataType>(28);
0069: #endif
0070: }
0071: 
0072: template <>
0073: constexpr hipDataType HipDataTypeFor<c10::Float8_e5m2>() {
0074: #if ROCM_VERSION >= 60300
0075:   return HIP_R_8F_E5M2;
0076: #else
0077:   return static_cast<hipDataType>(29);
0078: #endif
0079: }
0080: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: no prominent local symbols.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：无明显局部符号。

### Lines 81-101 / 第 81-101 行

```cpp
0081: // This type is not intended for matrix types but rather a scale factor.
0082: // Return a dummy value to satisfy linker.
0083: template <>
0084: constexpr hipDataType HipDataTypeFor<c10::Float8_e8m0fnu>() {
0085:   return static_cast<hipDataType>(500);
0086: }
0087: 
0088: template <>
0089: constexpr hipDataType HipDataTypeFor<c10::Float4_e2m1fn_x2>() {
0090: #if ROCM_VERSION >= 70000
0091:   return HIP_R_4F_E2M1;
0092: #else
0093:   return static_cast<hipDataType>(33);
0094: #endif
0095: }
0096: 
0097: template <typename T>
0098: constexpr hipblasComputeType_t HipBlasComputeTypeFor() {
0099:   return HIPBLAS_COMPUTE_32F;
0100: }
0101: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `HipBlasComputeTypeFor`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`HipBlasComputeTypeFor`。

### Lines 102-121 / 第 102-121 行

```cpp
0102: template <>
0103: constexpr hipblasComputeType_t HipBlasComputeTypeFor<double>() {
0104:   return HIPBLAS_COMPUTE_64F;
0105: }
0106: 
0107: template <typename T>
0108: int GetBatchFromParams(const GemmParams<T>* params) {
0109:   return 1;
0110: }
0111: 
0112: template <typename T>
0113: int GetBatchFromParams(const GemmAndBiasParams<T>* params) {
0114:   return 1;
0115: }
0116: 
0117: template <typename T>
0118: int GetBatchFromParams(const GemmStridedBatchedParams<T>* params) {
0119:   return params->batch;
0120: }
0121: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBatchFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBatchFromParams`。

### Lines 122-141 / 第 122-141 行

```cpp
0122: template <typename T>
0123: int GetBatchFromParams(const ScaledGemmParams<T>* params) {
0124:   return 1;
0125: }
0126: 
0127: template <typename T>
0128: int GetStrideAFromParams(const GemmParams<T>* params) {
0129:   return 1;
0130: }
0131: 
0132: template <typename T>
0133: int GetStrideAFromParams(const GemmAndBiasParams<T>* params) {
0134:   return 1;
0135: }
0136: 
0137: template <typename T>
0138: int GetStrideAFromParams(const GemmStridedBatchedParams<T>* params) {
0139:   return params->stride_a;
0140: }
0141: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBatchFromParams`, `GetStrideAFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBatchFromParams`, `GetStrideAFromParams`。

### Lines 142-161 / 第 142-161 行

```cpp
0142: template <typename T>
0143: int GetStrideAFromParams(const ScaledGemmParams<T>* params) {
0144:   return 1;
0145: }
0146: 
0147: template <typename T>
0148: int GetStrideBFromParams(const GemmParams<T>* params) {
0149:   return 1;
0150: }
0151: 
0152: template <typename T>
0153: int GetStrideBFromParams(const GemmAndBiasParams<T>* params) {
0154:   return 1;
0155: }
0156: 
0157: template <typename T>
0158: int GetStrideBFromParams(const GemmStridedBatchedParams<T>* params) {
0159:   return params->stride_b;
0160: }
0161: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetStrideAFromParams`, `GetStrideBFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetStrideAFromParams`, `GetStrideBFromParams`。

### Lines 162-181 / 第 162-181 行

```cpp
0162: template <typename T>
0163: int GetStrideBFromParams(const ScaledGemmParams<T>* params) {
0164:   return 1;
0165: }
0166: 
0167: template <typename T>
0168: int GetStrideCFromParams(const GemmParams<T>* params) {
0169:   return 1;
0170: }
0171: 
0172: template <typename T>
0173: int GetStrideCFromParams(const GemmAndBiasParams<T>* params) {
0174:   return 1;
0175: }
0176: 
0177: template <typename T>
0178: int GetStrideCFromParams(const GemmStridedBatchedParams<T>* params) {
0179:   return params->stride_c;
0180: }
0181: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetStrideBFromParams`, `GetStrideCFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetStrideBFromParams`, `GetStrideCFromParams`。

### Lines 182-201 / 第 182-201 行

```cpp
0182: template <typename T>
0183: int GetStrideCFromParams(const ScaledGemmParams<T>* params) {
0184:   return 1;
0185: }
0186: 
0187: template <typename T>
0188: at::opmath_type<T> GetAlphaFromParams(const GemmParams<T>* params) {
0189:   return params->alpha;
0190: }
0191: 
0192: template <typename T>
0193: at::opmath_type<T> GetAlphaFromParams(const GemmAndBiasParams<T>* params) {
0194:   return params->alpha;
0195: }
0196: 
0197: template <typename T>
0198: at::opmath_type<T> GetAlphaFromParams(const GemmStridedBatchedParams<T>* params) {
0199:   return params->alpha;
0200: }
0201: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetStrideCFromParams`, `GetAlphaFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetStrideCFromParams`, `GetAlphaFromParams`。

### Lines 202-221 / 第 202-221 行

```cpp
0202: template <typename T>
0203: at::opmath_type<T> GetAlphaFromParams(const ScaledGemmParams<T>* params) {
0204:   return at::opmath_type<T>{1.0};
0205: }
0206: 
0207: template <typename T>
0208: at::opmath_type<T> GetBetaFromParams(const GemmParams<T>* params) {
0209:   return params->beta;
0210: }
0211: 
0212: template <typename T>
0213: at::opmath_type<T> GetBetaFromParams(const GemmAndBiasParams<T>* params) {
0214:   return at::opmath_type<T>{0.0};
0215: }
0216: 
0217: template <typename T>
0218: at::opmath_type<T> GetBetaFromParams(const GemmStridedBatchedParams<T>* params) {
0219:   return params->beta;
0220: }
0221: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetAlphaFromParams`, `GetBetaFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetAlphaFromParams`, `GetBetaFromParams`。

### Lines 222-241 / 第 222-241 行

```cpp
0222: template <typename T>
0223: at::opmath_type<T> GetBetaFromParams(const ScaledGemmParams<T>* params) {
0224:   return at::opmath_type<T>{0.0};
0225: }
0226: 
0227: template <typename T>
0228: ScalingType GetAScalingTypeFromParams(const GemmParams<T>* params) {
0229:   return ScalingType::TensorWise;
0230: }
0231: 
0232: template <typename T>
0233: ScalingType GetBScalingTypeFromParams(const GemmParams<T>* params) {
0234:   return ScalingType::TensorWise;
0235: }
0236: 
0237: template <typename T>
0238: ScalingType GetAScalingTypeFromParams(const GemmAndBiasParams<T>* params) {
0239:   return ScalingType::TensorWise;
0240: }
0241: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBetaFromParams`, `GetAScalingTypeFromParams`, `GetBScalingTypeFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBetaFromParams`, `GetAScalingTypeFromParams`, `GetBScalingTypeFromParams`。

### Lines 242-261 / 第 242-261 行

```cpp
0242: template <typename T>
0243: ScalingType GetBScalingTypeFromParams(const GemmAndBiasParams<T>* params) {
0244:   return ScalingType::TensorWise;
0245: }
0246: 
0247: template <typename T>
0248: ScalingType GetAScalingTypeFromParams(const GemmStridedBatchedParams<T>* params) {
0249:   return ScalingType::TensorWise;
0250: }
0251: 
0252: template <typename T>
0253: ScalingType GetBScalingTypeFromParams(const GemmStridedBatchedParams<T>* params) {
0254:   return ScalingType::TensorWise;
0255: }
0256: 
0257: template <typename T>
0258: ScalingType GetAScalingTypeFromParams(const ScaledGemmParams<T>* params) {
0259:   return params->a_scaling_type;
0260: }
0261: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBScalingTypeFromParams`, `GetAScalingTypeFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBScalingTypeFromParams`, `GetAScalingTypeFromParams`。

### Lines 262-281 / 第 262-281 行

```cpp
0262: template <typename T>
0263: ScalingType GetBScalingTypeFromParams(const ScaledGemmParams<T>* params) {
0264:   return params->b_scaling_type;
0265: }
0266: 
0267: template <typename T>
0268: const void* GetAScalePointerFromParams(const GemmParams<T>* params) {
0269:   return nullptr;
0270: }
0271: 
0272: template <typename T>
0273: const void* GetAScalePointerFromParams(const GemmAndBiasParams<T>* params) {
0274:   return nullptr;
0275: }
0276: 
0277: template <typename T>
0278: const void* GetAScalePointerFromParams(const GemmStridedBatchedParams<T>* params) {
0279:   return nullptr;
0280: }
0281: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBScalingTypeFromParams`, `GetAScalePointerFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBScalingTypeFromParams`, `GetAScalePointerFromParams`。

### Lines 282-301 / 第 282-301 行

```cpp
0282: template <typename T>
0283: const void* GetAScalePointerFromParams(const ScaledGemmParams<T>* params) {
0284:   return params->a_scale_ptr;
0285: }
0286: 
0287: template <typename T>
0288: const void* GetBScalePointerFromParams(const GemmParams<T>* params) {
0289:   return nullptr;
0290: }
0291: 
0292: template <typename T>
0293: const void* GetBScalePointerFromParams(const GemmAndBiasParams<T>* params) {
0294:   return nullptr;
0295: }
0296: 
0297: template <typename T>
0298: const void* GetBScalePointerFromParams(const GemmStridedBatchedParams<T>* params) {
0299:   return nullptr;
0300: }
0301: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetAScalePointerFromParams`, `GetBScalePointerFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetAScalePointerFromParams`, `GetBScalePointerFromParams`。

### Lines 302-321 / 第 302-321 行

```cpp
0302: template <typename T>
0303: const void* GetBScalePointerFromParams(const ScaledGemmParams<T>* params) {
0304:   return params->b_scale_ptr;
0305: }
0306: 
0307: template <typename T>
0308: const void* GetDScalePointerFromParams(const GemmParams<T>* params) {
0309:   return nullptr;
0310: }
0311: 
0312: template <typename T>
0313: const void* GetDScalePointerFromParams(const GemmAndBiasParams<T>* params) {
0314:   return nullptr;
0315: }
0316: 
0317: template <typename T>
0318: const void* GetDScalePointerFromParams(const GemmStridedBatchedParams<T>* params) {
0319:   return nullptr;
0320: }
0321: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBScalePointerFromParams`, `GetDScalePointerFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBScalePointerFromParams`, `GetDScalePointerFromParams`。

### Lines 322-341 / 第 322-341 行

```cpp
0322: template <typename T>
0323: const void* GetDScalePointerFromParams(const ScaledGemmParams<T>* params) {
0324:   return params->c_scale_ptr;
0325: }
0326: 
0327: template <typename T>
0328: const void* GetBiasPointerFromParams(const GemmParams<T>* params) {
0329:   return nullptr;
0330: }
0331: 
0332: template <typename T>
0333: const void* GetBiasPointerFromParams(const GemmAndBiasParams<T>* params) {
0334:   return params->bias;
0335: }
0336: 
0337: template <typename T>
0338: const void* GetBiasPointerFromParams(const GemmStridedBatchedParams<T>* params) {
0339:   return nullptr;
0340: }
0341: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetDScalePointerFromParams`, `GetBiasPointerFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetDScalePointerFromParams`, `GetBiasPointerFromParams`。

### Lines 342-361 / 第 342-361 行

```cpp
0342: template <typename T>
0343: const void* GetBiasPointerFromParams(const ScaledGemmParams<T>* params) {
0344:   return params->bias_ptr;
0345: }
0346: 
0347: template <typename T>
0348: hipDataType GetBiasTypeFromParams(const GemmParams<T>* params) {
0349:   return HIP_R_32F;
0350: }
0351: 
0352: template <typename T>
0353: hipDataType GetBiasTypeFromParams(const GemmAndBiasParams<T>* params) {
0354:   return HipDataTypeFor<T>();
0355: }
0356: 
0357: template <typename T>
0358: hipDataType GetBiasTypeFromParams(const GemmStridedBatchedParams<T>* params) {
0359:   return HIP_R_32F;
0360: }
0361: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBiasPointerFromParams`, `GetBiasTypeFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBiasPointerFromParams`, `GetBiasTypeFromParams`。

### Lines 362-381 / 第 362-381 行

```cpp
0362: template <typename T>
0363: hipDataType GetBiasTypeFromParams(const ScaledGemmParams<T>* params) {
0364:   return at::cuda::ScalarTypeToCudaDataType(params->bias_dtype);
0365: }
0366: 
0367: template <typename T>
0368: at::cuda::blas::GEMMAndBiasActivationEpilogue GetActivationFromParams(const GemmParams<T>* params) {
0369:   return at::cuda::blas::GEMMAndBiasActivationEpilogue::None;
0370: }
0371: 
0372: template <typename T>
0373: at::cuda::blas::GEMMAndBiasActivationEpilogue GetActivationFromParams(const GemmAndBiasParams<T>* params) {
0374:   return params->activation;
0375: }
0376: 
0377: template <typename T>
0378: at::cuda::blas::GEMMAndBiasActivationEpilogue GetActivationFromParams(const GemmStridedBatchedParams<T>* params) {
0379:   return at::cuda::blas::GEMMAndBiasActivationEpilogue::None;
0380: }
0381: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetBiasTypeFromParams`, `ScalarTypeToCudaDataType`, `GetActivationFromParams`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetBiasTypeFromParams`, `ScalarTypeToCudaDataType`, `GetActivationFromParams`。

### Lines 382-402 / 第 382-402 行

```cpp
0382: template <typename T>
0383: at::cuda::blas::GEMMAndBiasActivationEpilogue GetActivationFromParams(const ScaledGemmParams<T>* params) {
0384:   return at::cuda::blas::GEMMAndBiasActivationEpilogue::None;
0385: }
0386: 
0387: static hipblasOperation_t _hipblasOpFromChar(char op) {
0388:   switch (op) {
0389:     case 'n':
0390:     case 'N':
0391:       return HIPBLAS_OP_N;
0392:     case 't':
0393:     case 'T':
0394:       return HIPBLAS_OP_T;
0395:     case 'c':
0396:     case 'C':
0397:       return HIPBLAS_OP_C;
0398:   }
0399:   TORCH_CHECK(false,
0400:       "_hipblasOpFromChar input should be 't', 'n' or 'c' but got `", op, "`");
0401: }
0402: 
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetActivationFromParams`, `_hipblasOpFromChar`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetActivationFromParams`, `_hipblasOpFromChar`。

### Lines 403-422 / 第 403-422 行

```cpp
0403: static char _charFromhipblasOp(hipblasOperation_t op) {
0404:   switch (op) {
0405:     case HIPBLAS_OP_N:
0406:       return 'N';
0407:     case HIPBLAS_OP_T:
0408:       return 'T';
0409:     case HIPBLAS_OP_C:
0410:       return 'C';
0411:   }
0412:   TORCH_CHECK(false,
0413:       "_charFromhipblasOp input should be HIPBLAS_OP_N/T/C but got `", op, "`");
0414: }
0415: 
0416: static hipblasOperation_t MapLayoutToHipBlasLt(BlasOp layout) {
0417:   if (layout == BlasOp::N) {
0418:     return HIPBLAS_OP_N;
0419:   }
0420:   return HIPBLAS_OP_T;
0421: }
0422: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; validates runtime invariants before continuing. Key symbols: `_charFromhipblasOp`, `MapLayoutToHipBlasLt`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；在继续执行前校验运行时不变量。关键符号：`_charFromhipblasOp`, `MapLayoutToHipBlasLt`。

### Lines 423-444 / 第 423-444 行

```cpp
0423: template <typename T, cublasStatus_t (*destructor)(T*)>
0424: struct HipBlasLtDeleter {
0425:   void operator()(T* x) {
0426:     if (x != nullptr) {
0427:       TORCH_CUDABLAS_CHECK(destructor(x));
0428:     }
0429:   }
0430: };
0431: 
0432: template <typename T, hipblasStatus_t (*destructor)(T*)>
0433: class HipBlasLtDescriptor {
0434:  public:
0435:   T* descriptor() const {
0436:     return descriptor_.get();
0437:   }
0438:   T* descriptor() {
0439:     return descriptor_.get();
0440:   }
0441: 
0442:  protected:
0443:   std::unique_ptr<T, HipBlasLtDeleter<T, destructor>> descriptor_;
0444: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `HipBlasLtDeleter`, `HipBlasLtDescriptor`, `descriptor`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`HipBlasLtDeleter`, `HipBlasLtDescriptor`, `descriptor`。

### Lines 445-464 / 第 445-464 行

```cpp
0445: 
0446: class HipBlasLtMatmulDescriptor : public HipBlasLtDescriptor<
0447:                                      hipblasLtMatmulDescOpaque_t,
0448:                                      &hipblasLtMatmulDescDestroy> {
0449:  public:
0450:   HipBlasLtMatmulDescriptor(
0451:       hipblasComputeType_t compute_type,
0452:       hipDataType scale_type) {
0453:     hipblasLtMatmulDesc_t raw_descriptor = nullptr;
0454:     TORCH_HIPBLASLT_CHECK(
0455:         hipblasLtMatmulDescCreate(&raw_descriptor, compute_type, scale_type));
0456:     descriptor_.reset(raw_descriptor);
0457:   }
0458:   template <typename T>
0459:   inline void setAttribute(hipblasLtMatmulDescAttributes_t attr, const T value) {
0460:     TORCH_HIPBLASLT_CHECK(::hipblasLtMatmulDescSetAttribute(descriptor(), attr, &value, sizeof(T)));
0461:   }
0462: };
0463: 
0464: template <typename AT, typename BT, typename CT, BlasOp ALayout, BlasOp BLayout, typename ParamsT>
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `HipBlasLtMatmulDescriptor`, `setAttribute`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`HipBlasLtMatmulDescriptor`, `setAttribute`。

### Lines 465-485 / 第 465-485 行

```cpp
0465: class HipblasltGemmOp : public Callable<ParamsT> {
0466:   public:
0467:     HipblasltGemmOp(hipblasLtMatmulAlgo_t algo) : algo_{algo} {}
0468: 
0469:     TuningStatus Call(const ParamsT* params) override {
0470:       hipblasOperation_t transa_outer = MapLayoutToHipBlasLt(ALayout);
0471:       hipblasOperation_t transb_outer = MapLayoutToHipBlasLt(BLayout);
0472:       auto a_datatype = HipDataTypeFor<AT>();
0473:       auto b_datatype = HipDataTypeFor<BT>();
0474:       auto in_out_datatype = HipDataTypeFor<CT>();
0475:       auto opa = _hipblasOpFromChar(params->transa);
0476:       auto opb = _hipblasOpFromChar(params->transb);
0477: 
0478:       TORCH_CHECK(transa_outer == opa && transb_outer == opb, "trans mismatch, shouldn't happen");
0479: 
0480:       using opmath_t = at::opmath_type<CT>;
0481:       opmath_t alpha = GetAlphaFromParams<CT>(params);
0482:       opmath_t beta = GetBetaFromParams<CT>(params);
0483: 
0484:       hipblasLtMatrixLayout_t mat_a, mat_b, mat_c;
0485:       if (opa == HIPBLAS_OP_N) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `HipblasltGemmOp`, `opmath_t`, `Call`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`HipblasltGemmOp`, `opmath_t`, `Call`。

### Lines 486-505 / 第 486-505 行

```cpp
0486:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutCreate(&mat_a, a_datatype, params->m, params->k, params->lda));
0487:       }
0488:       else {
0489:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutCreate(&mat_a, a_datatype, params->k, params->m, params->lda));
0490:       }
0491:       if (opb == HIPBLAS_OP_N) {
0492:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutCreate(&mat_b, b_datatype, params->k, params->n, params->ldb));
0493:       }
0494:       else {
0495:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutCreate(&mat_b, b_datatype, params->n, params->k, params->ldb));
0496:       }
0497:       TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutCreate(&mat_c, in_out_datatype, params->m, params->n, params->ldc));
0498: 
0499:       // specific to batched gemmm
0500:       int batch = GetBatchFromParams<CT>(params);
0501:       if (batch > 1) {
0502:         int64_t stride_a = GetStrideAFromParams<CT>(params);
0503:         int64_t stride_b = GetStrideBFromParams<CT>(params);
0504:         int64_t stride_c = GetStrideCFromParams<CT>(params);
0505:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutSetAttribute(
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。

### Lines 506-529 / 第 506-529 行

```cpp
0506:             mat_a, HIPBLASLT_MATRIX_LAYOUT_BATCH_COUNT, &batch, sizeof(batch)));
0507:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutSetAttribute(
0508:             mat_a, HIPBLASLT_MATRIX_LAYOUT_STRIDED_BATCH_OFFSET, &stride_a, sizeof(stride_a)));
0509:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutSetAttribute(
0510:             mat_b, HIPBLASLT_MATRIX_LAYOUT_BATCH_COUNT, &batch, sizeof(batch)));
0511:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutSetAttribute(
0512:             mat_b, HIPBLASLT_MATRIX_LAYOUT_STRIDED_BATCH_OFFSET, &stride_b, sizeof(stride_b)));
0513:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutSetAttribute(
0514:             mat_c, HIPBLASLT_MATRIX_LAYOUT_BATCH_COUNT, &batch, sizeof(batch)));
0515:         TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutSetAttribute(
0516:             mat_c, HIPBLASLT_MATRIX_LAYOUT_STRIDED_BATCH_OFFSET, &stride_c, sizeof(stride_c)));
0517:       }
0518: 
0519:       hipblasComputeType_t computeType = HipBlasComputeTypeFor<CT>();
0520:       if constexpr (std::is_same_v<CT, float>) {
0521:         if (at::globalContext().float32Precision(at::Float32Backend::CUDA, at::Float32Op::MATMUL) == at::Float32Precision::TF32) {
0522:           computeType = HIPBLAS_COMPUTE_32F_FAST_TF32;
0523:         }
0524:       }
0525:       auto scale_type = HipDataTypeFor<opmath_t>();
0526:       HipBlasLtMatmulDescriptor matmul(computeType, scale_type);
0527:       matmul.setAttribute(HIPBLASLT_MATMUL_DESC_TRANSA, opa);
0528:       matmul.setAttribute(HIPBLASLT_MATMUL_DESC_TRANSB, opb);
0529: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: `constexpr`, `matmul`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`constexpr`, `matmul`。

### Lines 530-549 / 第 530-549 行

```cpp
0530:       // specific to scaled gemm
0531:       const void* mat1_scale_ptr = GetAScalePointerFromParams<CT>(params);
0532:       const void* mat2_scale_ptr = GetBScalePointerFromParams<CT>(params);
0533:       const void* result_scale_ptr = GetDScalePointerFromParams<CT>(params);
0534:       if (mat1_scale_ptr && mat2_scale_ptr) {
0535:         hipblasLtMatmulDescAttributes_t a_scale_ptr_desc = HIPBLASLT_MATMUL_DESC_A_SCALE_POINTER;
0536:         hipblasLtMatmulDescAttributes_t b_scale_ptr_desc = HIPBLASLT_MATMUL_DESC_B_SCALE_POINTER;
0537:         if (GetAScalingTypeFromParams<CT>(params) == ScalingType::RowWise) {
0538: #if defined(HIPBLASLT_OUTER_VEC)
0539:           matmul.setAttribute(HIPBLASLT_MATMUL_DESC_A_SCALE_MODE, HIPBLASLT_MATMUL_MATRIX_SCALE_OUTER_VEC_32F);
0540: #elif defined(HIPBLASLT_VEC_EXT)
0541:           a_scale_ptr_desc = HIPBLASLT_MATMUL_DESC_A_SCALE_POINTER_VEC_EXT;
0542: #endif
0543:         }
0544:         if (GetBScalingTypeFromParams<CT>(params) == ScalingType::RowWise) {
0545: #if defined(HIPBLASLT_OUTER_VEC)
0546:           matmul.setAttribute(HIPBLASLT_MATMUL_DESC_B_SCALE_MODE, HIPBLASLT_MATMUL_MATRIX_SCALE_OUTER_VEC_32F);
0547: #elif defined(HIPBLASLT_VEC_EXT)
0548:           b_scale_ptr_desc = HIPBLASLT_MATMUL_DESC_B_SCALE_POINTER_VEC_EXT;
0549: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。

### Lines 550-574 / 第 550-574 行

```cpp
0550:         }
0551:         matmul.setAttribute(a_scale_ptr_desc, mat1_scale_ptr);
0552:         matmul.setAttribute(b_scale_ptr_desc, mat2_scale_ptr);
0553:       }
0554:       if (result_scale_ptr) {
0555:         matmul.setAttribute(HIPBLASLT_MATMUL_DESC_D_SCALE_POINTER, result_scale_ptr);
0556:       }
0557: 
0558:       const void* bias_ptr = GetBiasPointerFromParams<CT>(params);
0559:       auto bias_datatype = GetBiasTypeFromParams<CT>(params);
0560:       if (bias_ptr) {
0561:         matmul.setAttribute(HIPBLASLT_MATMUL_DESC_BIAS_POINTER, bias_ptr);
0562:         matmul.setAttribute(HIPBLASLT_MATMUL_DESC_BIAS_DATA_TYPE, bias_datatype);
0563:         auto activation = GetActivationFromParams<CT>(params);
0564:         if (activation == at::cuda::blas::GEMMAndBiasActivationEpilogue::RELU) {
0565:           matmul.setAttribute(HIPBLASLT_MATMUL_DESC_EPILOGUE, HIPBLASLT_EPILOGUE_RELU_BIAS);
0566:         }
0567:         else if (activation == at::cuda::blas::GEMMAndBiasActivationEpilogue::GELU) {
0568:           matmul.setAttribute(HIPBLASLT_MATMUL_DESC_EPILOGUE, HIPBLASLT_EPILOGUE_GELU_BIAS);
0569:         }
0570:         else {
0571:           matmul.setAttribute(HIPBLASLT_MATMUL_DESC_EPILOGUE, HIPBLASLT_EPILOGUE_BIAS);
0572:         }
0573:       }
0574: 
```

- **EN:** This block handles conditional branches and special cases; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。

### Lines 575-597 / 第 575-597 行

```cpp
0575:       size_t workspace_size = at::cuda::getCUDABlasLtWorkspaceSize();
0576: 
0577:       auto op_handle = at::cuda::getCurrentCUDABlasLtHandle();
0578: 
0579:       size_t ret_workspace_size = 0;
0580:       auto status = hipblaslt_ext::matmulIsAlgoSupported(op_handle,
0581:           matmul.descriptor(),
0582:           &alpha,
0583:           mat_a,
0584:           mat_b,
0585:           &beta,
0586:           mat_c,
0587:           mat_c,
0588:           algo_,
0589:           ret_workspace_size);
0590: 
0591:       if (status == HIPBLAS_STATUS_SUCCESS) {
0592:         if (ret_workspace_size >= workspace_size) {
0593:           return FAIL;
0594:         }
0595:       }
0596:       else {
0597:         return FAIL;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：无明显局部符号。

### Lines 598-618 / 第 598-618 行

```cpp
0598:       }
0599: 
0600:       void* workspace_buffer = at::cuda::getCUDABlasLtWorkspace();
0601: 
0602:       TORCH_HIPBLASLT_CHECK(hipblasLtMatmul(op_handle,
0603:             matmul.descriptor(),
0604:             &alpha,
0605:             params->a,
0606:             mat_a,
0607:             params->b,
0608:             mat_b,
0609:             &beta,
0610:             params->c,
0611:             mat_c,
0612:             params->c,
0613:             mat_c,
0614:             &algo_,
0615:             workspace_buffer,
0616:             workspace_size,
0617:             at::cuda::getCurrentCUDAStream()));
0618: 
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries. Key symbols: no prominent local symbols.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界。关键符号：无明显局部符号。

### Lines 619-638 / 第 619-638 行

```cpp
0619:       //TORCH_HIPBLASLT_CHECK(hipblasLtMatmulDescDestroy(matmul));
0620:       TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutDestroy(mat_a));
0621:       TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutDestroy(mat_b));
0622:       TORCH_HIPBLASLT_CHECK(hipblasLtMatrixLayoutDestroy(mat_c));
0623:       return OK;
0624:     }
0625: 
0626:   private:
0627:     hipblasLtMatmulAlgo_t algo_;
0628: };
0629: 
0630: template <typename AT, typename BT, typename CT, BlasOp ALayout, BlasOp BLayout, typename ParamsT>
0631: auto GetHipBlasLtTypeStringAndOps() {
0632:   hipblasOperation_t transa_outer = MapLayoutToHipBlasLt(ALayout);
0633:   hipblasOperation_t transb_outer = MapLayoutToHipBlasLt(BLayout);
0634:   auto a_datatype = HipDataTypeFor<AT>();
0635:   auto b_datatype = HipDataTypeFor<BT>();
0636:   auto in_out_datatype = HipDataTypeFor<CT>();
0637:   std::vector<hipblasLtMatmulHeuristicResult_t> heuristic_result;
0638: #if ROCM_VERSION == 60400
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetHipBlasLtTypeStringAndOps`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetHipBlasLtTypeStringAndOps`。

### Lines 639-666 / 第 639-666 行

```cpp
0639:   // hipblaslt TT fp32 regression on ROCm 6.4, cannot use
0640:   if ((a_datatype == HIP_R_32F || b_datatype == HIP_R_32F || in_out_datatype == HIP_R_32F)
0641:           && (transa_outer == HIPBLAS_OP_T && transb_outer == HIPBLAS_OP_T)) {
0642:     std::vector<std::pair<std::string, std::unique_ptr<Callable<ParamsT>>>> ignore;
0643:     return ignore;
0644:   }
0645: #endif
0646: 
0647:   hipblasComputeType_t computeType = HipBlasComputeTypeFor<CT>();
0648:   if constexpr (std::is_same_v<CT, float>) {
0649:     if (at::globalContext().float32Precision(at::Float32Backend::CUDA, at::Float32Op::MATMUL) == at::Float32Precision::TF32) {
0650:       computeType = HIPBLAS_COMPUTE_32F_FAST_TF32;
0651:     }
0652:   }
0653: 
0654:   hipblasLtHandle_t handle;
0655:   TORCH_HIPBLASLT_CHECK(hipblasLtCreate(&handle));
0656:   TORCH_HIPBLASLT_CHECK(hipblaslt_ext::getAllAlgos(handle,
0657:         hipblaslt_ext::GemmType::HIPBLASLT_GEMM,
0658:         transa_outer,
0659:         transb_outer,
0660:         a_datatype,
0661:         b_datatype,
0662:         in_out_datatype,
0663:         in_out_datatype,
0664:         computeType,
0665:         heuristic_result));
0666:   TORCH_HIPBLASLT_CHECK(hipblasLtDestroy(handle));
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: `constexpr`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`constexpr`。

### Lines 667-686 / 第 667-686 行

```cpp
0667: 
0668:   int returned_algo_count = heuristic_result.size();
0669:   std::vector<std::pair<std::string, std::unique_ptr<Callable<ParamsT>>>> ret;
0670:   for (int i = 0; i < returned_algo_count; i++) {
0671:     auto algo = heuristic_result[i].algo;
0672:     int algo_index = hipblaslt_ext::getIndexFromAlgo(algo);
0673:     auto callable = std::make_unique<HipblasltGemmOp<AT, BT, CT, ALayout, BLayout, ParamsT>>(algo);
0674:     std::string type_string = fmt::sprintf("Gemm_Hipblaslt_%d", algo_index);
0675:     ret.emplace_back(type_string, std::move(callable));
0676:   }
0677: 
0678:   return ret;
0679: }
0680: 
0681: template <typename T, BlasOp ALayout, BlasOp BLayout>
0682: auto GetHipBlasLtGemmTypeStringAndOps() {
0683:   return GetHipBlasLtTypeStringAndOps<T, T, T, ALayout, BLayout, GemmParams<T>>();
0684: }
0685: 
0686: template <typename T, BlasOp ALayout, BlasOp BLayout>
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetHipBlasLtGemmTypeStringAndOps`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetHipBlasLtGemmTypeStringAndOps`。

### Lines 687-703 / 第 687-703 行

```cpp
0687: auto GetHipBlasLtGemmAndBiasTypeStringAndOps() {
0688:   return GetHipBlasLtTypeStringAndOps<T, T, T, ALayout, BLayout, GemmAndBiasParams<T>>();
0689: }
0690: 
0691: template <typename T, BlasOp ALayout, BlasOp BLayout>
0692: auto GetHipBlasLtGemmStridedBatchedTypeStringAndOps() {
0693:   return GetHipBlasLtTypeStringAndOps<T, T, T, ALayout, BLayout, GemmStridedBatchedParams<T>>();
0694: }
0695: 
0696: template <typename AT, typename BT, typename CT, BlasOp ALayout, BlasOp BLayout>
0697: auto GetHipBlasLtScaledGemmTypeStringAndOps() {
0698:   return GetHipBlasLtTypeStringAndOps<AT, BT, CT, ALayout, BLayout, ScaledGemmParams<CT>>();
0699: }
0700: 
0701: #undef TORCH_HIPBLASLT_CHECK
0702: 
0703: }  // namespace at::cuda::tunable
```

- **EN:** Builds a reusable template/helper layer around `GemmHipblaslt`. Key symbols: `GetHipBlasLtGemmAndBiasTypeStringAndOps`, `GetHipBlasLtGemmStridedBatchedTypeStringAndOps`, `GetHipBlasLtScaledGemmTypeStringAndOps`.
- **CN:** 围绕 `GemmHipblaslt` 构建可复用的模板或辅助层。关键符号：`GetHipBlasLtGemmAndBiasTypeStringAndOps`, `GetHipBlasLtGemmStridedBatchedTypeStringAndOps`, `GetHipBlasLtScaledGemmTypeStringAndOps`。


## Key Concepts / 关键概念
- **CUDA backend plumbing** — CUDA 后端基础设施
- **Tensor metadata and value flow** — 张量元数据与数值流
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: HipBlasLtDeleter, HipBlasLtDescriptor, HipBlasLtMatmulDescriptor, HipblasltGemmOp, opmath_t, HipDataTypeFor, HipBlasComputeTypeFor, GetBatchFromParams** — 核心符号：HipBlasLtDeleter、HipBlasLtDescriptor、HipBlasLtMatmulDescriptor、HipblasltGemmOp、opmath_t、HipDataTypeFor、HipBlasComputeTypeFor、GetBatchFromParams

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/cuda/CUDAContext.h`, `ATen/cuda/CUDADataType.h`, `ATen/cuda/tunable/TunableOp.h`, `ATen/cuda/tunable/GemmCommon.h`, `c10/cuda/CUDACachingAllocator.h`, `c10/util/StringUtil.h`
- **External includes / 外部头文件**: `fmt/printf.h`, `hipblaslt/hipblaslt.h`, `hipblaslt/hipblaslt-ext.hpp`
- **Namespaces / 命名空间**: `at::cuda::tunable`
- **Representative symbols / 代表性符号**: `HipBlasLtDeleter`, `HipBlasLtDescriptor`, `HipBlasLtMatmulDescriptor`, `HipblasltGemmOp`, `opmath_t`, `HipDataTypeFor`, `HipBlasComputeTypeFor`, `GetBatchFromParams`, `GetStrideAFromParams`, `GetStrideBFromParams`, `GetStrideCFromParams`, `GetAlphaFromParams`, `...`
