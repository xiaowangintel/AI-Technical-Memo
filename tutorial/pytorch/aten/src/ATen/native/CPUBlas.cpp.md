# CPUBlas.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/CPUBlas.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to CPUBlas. It also wires backend dispatch paths.
- **Purpose (CN)**: 实现或声明与 cpublas 相关的 ATen 原生逻辑。 它还负责连接不同后端的调度路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/native/CPUBlas.h>
0003: #include <ATen/native/mkl/LinearAlgebra.h>
0004: #include <ATen/native/mkldnn/Matmul.h>
0005: 
0006: #include <c10/util/SmallBuffer.h>
0007: #include <c10/util/irange.h>
0008: 
0009: #include <climits>
0010: #if !defined(__s390x__ ) && !defined(__powerpc__)
0011: #include <cpuinfo.h>
0012: #endif
0013: 
0014: #if AT_BUILD_WITH_BLAS()
0015: #if C10_IOS
0016: #include <Accelerate/Accelerate.h>
0017: #elif !defined(_ARMPL_H)
0018: extern "C" void dgemm_(char *transa, char *transb, int *m, int *n, int *k, double *alpha, const double *a, int *lda, const double *b, int *ldb, double *beta, double *c, int *ldc);
0019: extern "C" void sgemm_(char *transa, char *transb, int *m, int *n, int *k, float *alpha, const float *a, int *lda, const float *b, int *ldb, float *beta, float *c, int *ldc);
0020: extern "C" void cgemm_(char *transa, char *transb, int *m, int *n, int *k, void *alpha, const void *a, int *lda, const void *b, int *ldb, void *beta, void *c, int *ldc);
0021: extern "C" void zgemm_(char *transa, char *transb, int *m, int *n, int *k, void *alpha, const void *a, int *lda, const void *b, int *ldb, void *beta, void *c, int *ldc);
0022: #ifdef BLAS_HAS_SBGEMM
0023: extern "C" void sbgemm_(char *transa, char *transb, int *m, int *n, int *k,
0024:                 float *alpha,
0025:                 const at::BFloat16 *a, int *lda,
0026:                 const at::BFloat16 *b, int *ldb,
0027:                 float *beta,
0028:                 float *c, int *ldc);
0029: #endif  // BLAS_HAS_SBGEMM
0030: #ifdef BLAS_HAS_SHGEMM
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, state/variable declarations. Notable symbols: defined, AT_BUILD_WITH_BLAS, dgemm_, sgemm_.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、变量/别名声明。 值得关注的符号包括：defined, AT_BUILD_WITH_BLAS, dgemm_, sgemm_。

### Lines 31-60 / 第 31-60 行
```cpp
0031: extern "C" void shgemm_(char *transa, char *transb, int *m, int *n, int *k,
0032:                 float *alpha,
0033:                 const at::Half *a, int *lda,
0034:                 const at::Half *b, int *ldb,
0035:                 float *beta,
0036:                 float *c, int *ldc);
0037: #endif  // BLAS_HAS_SHGEMM
0038: extern "C" void cswap_(int *n, const void *x, int *incx, void *y, int *incy);
0039: extern "C" void dcopy_(int *n, const double *x, int *incx, double *y, int *incy);
0040: extern "C" void scopy_(int *n, const float *x, int *incx, float *y, int *incy);
0041: extern "C" void zcopy_(int *n, const void *x, int *incx, void *y, int *incy);
0042: extern "C" void ccopy_(int *n, const void *x, int *incx, void *y, int *incy);
0043: extern "C" void daxpy_(int *n, double *a, const double *x, int *incx, double *y, int *incy);
0044: extern "C" void saxpy_(int *n, float *a, const float *x, int *incx, float *y, int *incy);
0045: extern "C" void caxpy_(int *n, void *a, const void *x, int *incx, void *y, int *incy);
0046: extern "C" void zaxpy_(int *n, void *a, const void *x, int *incx, void *y, int *incy);
0047: #endif  // C10_IOS
0048: #endif  // AT_BUILD_WITH_BLAS
0049: 
0050: #ifdef USE_FBGEMM
0051: #include <fbgemm/FbgemmI64.h>
0052: #endif  // USE_FBGEMM
0053: 
0054: #if AT_MKLDNN_ENABLED()
0055: #include <ideep.hpp>
0056: // Add uKernel API versioning to be compatible with different oneDNN versions
0057: // oneDNN 3.6.x updates the ukernel APIs of brgemm and brgemm_pack_B
0058: // brgemm_pack_B is changed to transform and the setting of brgemm beta is changed to set_add_C
0059: #if (IDEEP_VERSION_MAJOR == 3 && IDEEP_VERSION_MINOR == 5)
0060: #define ONEDNN_UKERNEL_1
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, conditional compilation, expressions/calls. Notable symbols: shgemm_, cswap_, dcopy_, scopy_.
- **CN**: 第 31-60 行主要涉及变量/别名声明、预处理条件、表达式或调用。 值得关注的符号包括：shgemm_, cswap_, dcopy_, scopy_。

### Lines 61-90 / 第 61-90 行
```cpp
0061: #elif ((IDEEP_VERSION_MAJOR == 3 && IDEEP_VERSION_MINOR >= 6) || (IDEEP_VERSION_MAJOR > 3))
0062: #define ONEDNN_UKERNEL_2
0063: #endif
0064: #if ((defined(ONEDNN_UKERNEL_1) || defined(ONEDNN_UKERNEL_2)) && (defined(__x86_64__) || (defined(_M_X64) && !defined(_M_ARM64EC))))
0065: #define ONEDNN_UKERNEL_ENABLED
0066: #endif
0067: 
0068: #if IDEEP_PREREQ(3, 9, 0, 0)
0069: #define ONEDNN_FP8_UKERNEL_AVAILABLE
0070: #endif
0071: 
0072: #endif  // AT_MKLDNN_ENABLED()
0073: 
0074: #if defined(ONEDNN_UKERNEL_ENABLED)
0075: #include <oneapi/dnnl/dnnl_ukernel.hpp>
0076: #include <oneapi/dnnl/dnnl.hpp>
0077: #endif // oneDNN BRGEMM
0078: 
0079: namespace at::native::cpublas {
0080: namespace internal {
0081: 
0082: void normalize_last_dims(
0083:     TransposeType transa, TransposeType transb,
0084:     int64_t m, int64_t n, int64_t k,
0085:     int64_t *lda, int64_t *ldb, int64_t *ldc) {
0086:   if (n == 1) {
0087:     *ldc = m;
0088:   }
0089: 
0090:   if(transa != TransposeType::NoTranspose) {
```
- **EN**: Lines 61-90 mainly cover conditional compilation, expressions/calls, macro-based glue. Notable symbols: elif, defined, IDEEP_PREREQ, AT_MKLDNN_ENABLED.
- **CN**: 第 61-90 行主要涉及预处理条件、表达式或调用、宏定义或宏调用。 值得关注的符号包括：elif, defined, IDEEP_PREREQ, AT_MKLDNN_ENABLED。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     if (m == 1) {
0092:       *lda = k;
0093:     }
0094:   } else if(k == 1) {
0095:     *lda = m;
0096:   }
0097: 
0098:   if(transb != TransposeType::NoTranspose) {
0099:     if (k == 1) {
0100:       *ldb = n;
0101:     }
0102:   } else if (n == 1) {
0103:     *ldb = k;
0104:   }
0105: }
0106: }  // namespace internal
0107: 
0108: namespace {
0109: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunneeded-internal-declaration")
0110: bool use_blas_gemm(
0111:     TransposeType transa, TransposeType transb,
0112:     int64_t m, int64_t n, int64_t k,
0113:     int64_t lda, int64_t ldb, int64_t ldc) {
0114:   const bool transa_ = transa != TransposeType::NoTranspose;
0115:   const bool transb_ = transb != TransposeType::NoTranspose;
0116:   return (
0117:       (m <= INT_MAX) && (n <= INT_MAX) && (k <= INT_MAX) &&
0118:       (lda <= INT_MAX) && (ldb <= INT_MAX) && (ldc <= INT_MAX) &&
0119:       (lda >= std::max(int64_t{1}, (transa_ ? k : m))) &&
0120:       (ldb >= std::max(int64_t{1}, (transb_ ? n : k))) &&
```
- **EN**: Lines 91-120 mainly cover expressions/calls, function signatures/definitions, comments/documentation. Notable symbols: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED, use_blas_gemm, max.
- **CN**: 第 91-120 行主要涉及表达式或调用、函数签名或实现、注释或说明。 值得关注的符号包括：C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED, use_blas_gemm, max。

### Lines 121-150 / 第 121-150 行
```cpp
0121:       (ldc >= std::max(int64_t{1}, m)));
0122: }
0123: C10_DIAGNOSTIC_POP()
0124: 
0125: #ifdef USE_FBGEMM
0126: fbgemm::matrix_op_t to_fbgemm(TransposeType trans) {
0127:   switch (trans) {
0128:     case TransposeType::Transpose: return fbgemm::matrix_op_t::Transpose;
0129:     case TransposeType::NoTranspose: return fbgemm::matrix_op_t::NoTranspose;
0130:     case TransposeType::ConjTranspose: TORCH_INTERNAL_ASSERT(false, "ConjTranspose type is not supported in fbgemm");
0131:   }
0132:   TORCH_INTERNAL_ASSERT(false, "Invalid transpose type");
0133: }
0134: #endif  // USE_FBGEMM
0135: 
0136: #if (AT_BUILD_WITH_BLAS() && C10_IOS)
0137: CBLAS_TRANSPOSE to_apple_accelerate_transpose(TransposeType trans) {
0138:   switch (trans) {
0139:     case TransposeType::Transpose: return CblasTrans;
0140:     case TransposeType::NoTranspose: return CblasNoTrans;
0141:     case TransposeType::ConjTranspose: return CblasConjTrans;
0142:   }
0143:   TORCH_INTERNAL_ASSERT(false, "Invalid transpose type");
0144: }
0145: #endif
0146: 
0147: }  // namespace (anonymous)
0148: 
0149: DEFINE_DISPATCH(gemm_stub);
0150: DEFINE_DISPATCH(gemm_no_downcast_stub);
```
- **EN**: Lines 121-150 mainly cover control-flow checks, expressions/calls, macro-based glue. Notable symbols: max, C10_DIAGNOSTIC_POP, to_fbgemm, TORCH_INTERNAL_ASSERT.
- **CN**: 第 121-150 行主要涉及控制流逻辑、表达式或调用、宏定义或宏调用。 值得关注的符号包括：max, C10_DIAGNOSTIC_POP, to_fbgemm, TORCH_INTERNAL_ASSERT。

### Lines 151-180 / 第 151-180 行
```cpp
0151: 
0152: void gemm(
0153:     TransposeType transa, TransposeType transb,
0154:     int64_t m, int64_t n, int64_t k,
0155:     const double alpha,
0156:     const double *a, int64_t lda,
0157:     const double *b, int64_t ldb,
0158:     const double beta,
0159:     double *c, int64_t ldc) {
0160:   internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0161: #if AT_BUILD_WITH_BLAS()
0162:   if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0163:     int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0164:     double alpha_ = alpha, beta_ = beta;
0165:     #if C10_IOS
0166:     CBLAS_TRANSPOSE transa_ = to_apple_accelerate_transpose(transa);
0167:     CBLAS_TRANSPOSE transb_ = to_apple_accelerate_transpose(transb);
0168:     cblas_dgemm(CblasColMajor,
0169:       transa_, transb_,
0170:       m_, n_, k_,
0171:       alpha_,
0172:       a, lda_,
0173:       b, ldb_,
0174:       beta_,
0175:       c, ldc_);
0176:     #else
0177:     char transa_ = to_blas(transa), transb_ = to_blas(transb);
0178:     dgemm_(
0179:         &transa_, &transb_,
0180:         &m_, &n_, &k_,
```
- **EN**: Lines 151-180 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: gemm, normalize_last_dims, AT_BUILD_WITH_BLAS, use_blas_gemm.
- **CN**: 第 151-180 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：gemm, normalize_last_dims, AT_BUILD_WITH_BLAS, use_blas_gemm。

### Lines 181-210 / 第 181-210 行
```cpp
0181:         &alpha_,
0182:         a, &lda_,
0183:         b, &ldb_,
0184:         &beta_,
0185:         c, &ldc_);
0186:     #endif
0187:     return;
0188:   }
0189: #endif
0190:   gemm_stub(
0191:       at::kCPU, at::kDouble,
0192:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0193: }
0194: 
0195: #ifndef armpl_doublecomplex_t
0196: #define COMPLEX_DBL(a) a
0197: #define COMPLEX_DBL_CONST(a) a
0198: #define COMPLEX_FLOAT(a) a
0199: #define COMPLEX_FLOAT_CONST(a) a
0200: #else
0201: #define COMPLEX_DBL(a) ((armpl_doublecomplex_t*)a)
0202: #define COMPLEX_DBL_CONST(a) ((const armpl_doublecomplex_t*)a)
0203: #define COMPLEX_FLOAT(a) ((armpl_singlecomplex_t*)a)
0204: #define COMPLEX_FLOAT_CONST(a) ((const armpl_singlecomplex_t*)a)
0205: #endif
0206: 
0207: void gemm(
0208:     TransposeType transa, TransposeType transb,
0209:     int64_t m, int64_t n, int64_t k,
0210:     const float alpha,
```
- **EN**: Lines 181-210 mainly cover expressions/calls, macro-based glue, conditional compilation. Notable symbols: gemm_stub, COMPLEX_DBL, COMPLEX_DBL_CONST, COMPLEX_FLOAT.
- **CN**: 第 181-210 行主要涉及表达式或调用、宏定义或宏调用、预处理条件。 值得关注的符号包括：gemm_stub, COMPLEX_DBL, COMPLEX_DBL_CONST, COMPLEX_FLOAT。

### Lines 211-240 / 第 211-240 行
```cpp
0211:     const float *a, int64_t lda,
0212:     const float *b, int64_t ldb,
0213:     const float beta,
0214:     float *c, int64_t ldc) {
0215:   internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0216: #if AT_MKLDNN_ENABLED()
0217:    if (mkldnn_reduced_f32_gemm(transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc)) {
0218:      return;
0219:    }
0220: #endif
0221: #if AT_BUILD_WITH_BLAS()
0222:   if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0223:     int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0224:     float alpha_ = alpha, beta_ = beta;
0225:     #if C10_IOS
0226:     CBLAS_TRANSPOSE transa_ = to_apple_accelerate_transpose(transa);
0227:     CBLAS_TRANSPOSE transb_ = to_apple_accelerate_transpose(transb);
0228:     cblas_sgemm(CblasColMajor,
0229:       transa_, transb_,
0230:       m_, n_, k_,
0231:       alpha_,
0232:       a, lda_,
0233:       b, ldb_,
0234:       beta_,
0235:       c, ldc_);
0236:     #else
0237:     char transa_ = to_blas(transa), transb_ = to_blas(transb);
0238:     sgemm_(
0239:         &transa_, &transb_,
0240:         &m_, &n_, &k_,
```
- **EN**: Lines 211-240 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: normalize_last_dims, AT_MKLDNN_ENABLED, mkldnn_reduced_f32_gemm, AT_BUILD_WITH_BLAS.
- **CN**: 第 211-240 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：normalize_last_dims, AT_MKLDNN_ENABLED, mkldnn_reduced_f32_gemm, AT_BUILD_WITH_BLAS。

### Lines 241-270 / 第 241-270 行
```cpp
0241:         &alpha_,
0242:         a, &lda_,
0243:         b, &ldb_,
0244:         &beta_,
0245:         c, &ldc_);
0246:     #endif
0247:     return;
0248:   }
0249: #endif
0250:   gemm_stub(
0251:       at::kCPU, at::kFloat,
0252:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0253: }
0254: 
0255: void gemm(
0256:     TransposeType transa, TransposeType transb,
0257:     int64_t m, int64_t n, int64_t k,
0258:     const c10::complex<double> alpha,
0259:     const c10::complex<double> *a, int64_t lda,
0260:     const c10::complex<double> *b, int64_t ldb,
0261:     const c10::complex<double> beta,
0262:     c10::complex<double> *c, int64_t ldc) {
0263:   internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0264: #if AT_BUILD_WITH_BLAS()
0265:   if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0266:     int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0267:     c10::complex<double> alpha_ = alpha, beta_ = beta;
0268:     #if C10_IOS
0269:     CBLAS_TRANSPOSE transa_ = to_apple_accelerate_transpose(transa);
0270:     CBLAS_TRANSPOSE transb_ = to_apple_accelerate_transpose(transb);
```
- **EN**: Lines 241-270 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: gemm_stub, gemm, normalize_last_dims, AT_BUILD_WITH_BLAS.
- **CN**: 第 241-270 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：gemm_stub, gemm, normalize_last_dims, AT_BUILD_WITH_BLAS。

### Lines 271-300 / 第 271-300 行
```cpp
0271:     cblas_zgemm(CblasColMajor,
0272:       transa_, transb_,
0273:       m_, n_, k_,
0274:       &alpha_,
0275:       a, lda_,
0276:       b, ldb_,
0277:       &beta_,
0278:       c, ldc_);
0279:     #else
0280:     char transa_ = to_blas(transa), transb_ = to_blas(transb);
0281:     zgemm_(
0282:         &transa_, &transb_,
0283:         &m_, &n_, &k_,
0284:         COMPLEX_DBL_CONST(&alpha_),
0285:         COMPLEX_DBL_CONST(a), &lda_,
0286:         COMPLEX_DBL_CONST(b), &ldb_,
0287:         COMPLEX_DBL_CONST(&beta_),
0288:         COMPLEX_DBL(c), &ldc_);
0289:     #endif
0290:     return;
0291:   }
0292: #endif
0293:   gemm_stub(
0294:       at::kCPU, at::kComplexDouble,
0295:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0296: }
0297: 
0298: void gemm(
0299:     TransposeType transa, TransposeType transb,
0300:     int64_t m, int64_t n, int64_t k,
```
- **EN**: Lines 271-300 mainly cover expressions/calls, macro-based glue, state/variable declarations. Notable symbols: cblas_zgemm, to_blas, zgemm_, COMPLEX_DBL_CONST.
- **CN**: 第 271-300 行主要涉及表达式或调用、宏定义或宏调用、变量/别名声明。 值得关注的符号包括：cblas_zgemm, to_blas, zgemm_, COMPLEX_DBL_CONST。

### Lines 301-330 / 第 301-330 行
```cpp
0301:     const c10::complex<float> alpha,
0302:     const c10::complex<float> *a, int64_t lda,
0303:     const c10::complex<float> *b, int64_t ldb,
0304:     const c10::complex<float> beta,
0305:     c10::complex<float> *c, int64_t ldc) {
0306:   internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0307: #if AT_BUILD_WITH_BLAS()
0308:   if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0309:     int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0310:     c10::complex<float> alpha_ = alpha, beta_ = beta;
0311:     #if C10_IOS
0312:     CBLAS_TRANSPOSE transa_ = to_apple_accelerate_transpose(transa);
0313:     CBLAS_TRANSPOSE transb_ = to_apple_accelerate_transpose(transb);
0314:     cblas_cgemm(CblasColMajor,
0315:       transa_, transb_,
0316:       m_, n_, k_,
0317:       &alpha_,
0318:       a, lda_,
0319:       b, ldb_,
0320:       &beta_,
0321:       c, ldc_);
0322:     #else
0323:     char transa_ = to_blas(transa), transb_ = to_blas(transb);
0324:     cgemm_(
0325:         &transa_, &transb_,
0326:         &m_, &n_, &k_,
0327:         COMPLEX_FLOAT_CONST(&alpha_),
0328:         COMPLEX_FLOAT_CONST(a), &lda_,
0329:         COMPLEX_FLOAT_CONST(b), &ldb_,
0330:         COMPLEX_FLOAT_CONST(&beta_),
```
- **EN**: Lines 301-330 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: normalize_last_dims, AT_BUILD_WITH_BLAS, use_blas_gemm, to_apple_accelerate_transpose.
- **CN**: 第 301-330 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：normalize_last_dims, AT_BUILD_WITH_BLAS, use_blas_gemm, to_apple_accelerate_transpose。

### Lines 331-360 / 第 331-360 行
```cpp
0331:         COMPLEX_FLOAT(c), &ldc_);
0332:     #endif
0333:     return;
0334:   }
0335: #endif
0336:   gemm_stub(
0337:       at::kCPU, at::kComplexFloat,
0338:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0339: }
0340: 
0341: void gemm(
0342:    TransposeType transa, TransposeType transb,
0343:    int64_t m, int64_t n, int64_t k,
0344:    const float alpha,
0345:    const at::BFloat16 *a, int64_t lda,
0346:    const at::BFloat16 *b, int64_t ldb,
0347:    const float beta,
0348:    at::BFloat16 *c, int64_t ldc) {
0349:    internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0350: #if AT_MKLDNN_ENABLED()
0351: #ifdef __aarch64__
0352:    // MKLDNN also supports ARM for bf16, and the bypass is only
0353:    // currently intended for x86/x86_64.
0354:    const bool use_bf16_gemv_trans = false;
0355: #elif defined(__powerpc__)
0356:    const bool use_bf16_gemv_trans = false;
0357: #else
0358:    const bool bf16_gemv_trans_would_be_faster = cpuinfo_initialize() &&
0359:      !cpuinfo_has_x86_avx512bf16();
0360:    const bool use_bf16_gemv_trans = bf16_gemv_trans_would_be_faster &&
```
- **EN**: Lines 331-360 mainly cover expressions/calls, conditional compilation, state/variable declarations. Notable symbols: COMPLEX_FLOAT, gemm_stub, gemm, normalize_last_dims.
- **CN**: 第 331-360 行主要涉及表达式或调用、预处理条件、变量/别名声明。 值得关注的符号包括：COMPLEX_FLOAT, gemm_stub, gemm, normalize_last_dims。

### Lines 361-390 / 第 361-390 行
```cpp
0361:      transa == TransposeType::Transpose &&
0362:      transb == TransposeType::NoTranspose && n == 1 && alpha == 1.0;
0363: #endif
0364:    if (!use_bf16_gemv_trans && mkldnn_bf16_gemm(transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc)) {
0365:      return;
0366:    }
0367: #endif
0368: #if AT_BUILD_WITH_BLAS() && defined(BLAS_HAS_SBGEMM)
0369:    if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0370:       int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0371:       char transa_ = to_blas(transa), transb_ = to_blas(transb);
0372:       float alpha_ = alpha, beta_ = beta;
0373:       int c_size = n_ * m_;
0374:       // C matrix in OpenBLAS sbgemm are of type "float" so we have to convert, copy and copy back.
0375:       std::vector<float> float_v(c_size, 0.0f);
0376:       for (const auto j : c10::irange(n)) {
0377:         for (const auto i : c10::irange(m)) {
0378:           float_v[j * m_ + i] = c10::convert<float>(c[j * ldc_ + i]);
0379:         }
0380:       }
0381:       sbgemm_(&transa_, &transb_,
0382:               &m_, &n_, &k_,
0383:               &alpha_,
0384:               a, &lda_,
0385:               b, &ldb_,
0386:               &beta_,
0387:               float_v.data(), &m_);
0388:       for (const auto j : c10::irange(n)) {
0389:         for (const auto i : c10::irange(m)) {
0390:           c[j * ldc_ + i] = c10::convert<at::BFloat16>(float_v[j * m_ + i]);
```
- **EN**: Lines 361-390 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: mkldnn_bf16_gemm, AT_BUILD_WITH_BLAS, defined, use_blas_gemm.
- **CN**: 第 361-390 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：mkldnn_bf16_gemm, AT_BUILD_WITH_BLAS, defined, use_blas_gemm。

### Lines 391-420 / 第 391-420 行
```cpp
0391:         }
0392:       }
0393:       return;
0394:    }
0395: #endif
0396:    gemm_stub(
0397:       at::kCPU, at::kBFloat16,
0398:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0399: }
0400: 
0401: void gemm(
0402:    TransposeType transa, TransposeType transb,
0403:    int64_t m, int64_t n, int64_t k,
0404:    const float alpha,
0405:    const at::Half *a, int64_t lda,
0406:    const at::Half *b, int64_t ldb,
0407:    const float beta,
0408:    at::Half *c, int64_t ldc) {
0409:    internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0410: #if AT_MKLDNN_ENABLED()
0411:    // Per https://github.com/pytorch/pytorch/pull/137918#discussion_r1825460179 ,
0412:    // we should not bother checking for !cpuinfo_has_x86_avx512fp16() here,
0413:    // because "onednn (mkldnn) won't use avx512fp16 to compute gemms by default
0414:    // because the avx512fp16 fma would incur accuracy loss".
0415: #if defined(__powerpc__)
0416:    const bool fp16_gemv_trans_would_be_faster = false;
0417: #else
0418:    const bool fp16_gemv_trans_would_be_faster = cpuinfo_initialize() &&
0419:      cpuinfo_has_x86_f16c();
0420: #endif
```
- **EN**: Lines 391-420 mainly cover expressions/calls, conditional compilation, state/variable declarations. Notable symbols: gemm_stub, gemm, normalize_last_dims, AT_MKLDNN_ENABLED.
- **CN**: 第 391-420 行主要涉及表达式或调用、预处理条件、变量/别名声明。 值得关注的符号包括：gemm_stub, gemm, normalize_last_dims, AT_MKLDNN_ENABLED。

### Lines 421-450 / 第 421-450 行
```cpp
0421:    const bool use_fp16_gemv_trans = fp16_gemv_trans_would_be_faster &&
0422:      transa == TransposeType::Transpose &&
0423:      transb == TransposeType::NoTranspose && n == 1 && alpha == 1.0;
0424:    if (!use_fp16_gemv_trans &&
0425:        mkldnn_fp16_gemm(transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc)) {
0426:      return;
0427:    }
0428: #endif
0429: #if AT_BUILD_WITH_BLAS() && defined(BLAS_HAS_SHGEMM)
0430:    if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0431:       int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0432:       char transa_ = to_blas(transa), transb_ = to_blas(transb);
0433:       float alpha_ = alpha, beta_ = beta;
0434:       int c_size = n_ * m_;
0435:       // C matrix in OpenBLAS shgemm are of type "float" so we have to convert, copy and copy back.
0436:       std::vector<float> float_v(c_size, 0.0f);
0437:       for (const auto j : c10::irange(n)) {
0438:         for (const auto i : c10::irange(m)) {
0439:           float_v[j * m_ + i] = c10::convert<float>(c[j * ldc_ + i]);
0440:         }
0441:       }
0442:       shgemm_(&transa_, &transb_,
0443:               &m_, &n_, &k_,
0444:               &alpha_,
0445:               a, &lda_,
0446:               b, &ldb_,
0447:               &beta_,
0448:               float_v.data(), &m_);
0449:       for (const auto j : c10::irange(n)) {
0450:         for (const auto i : c10::irange(m)) {
```
- **EN**: Lines 421-450 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: mkldnn_fp16_gemm, AT_BUILD_WITH_BLAS, defined, use_blas_gemm.
- **CN**: 第 421-450 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：mkldnn_fp16_gemm, AT_BUILD_WITH_BLAS, defined, use_blas_gemm。

### Lines 451-480 / 第 451-480 行
```cpp
0451:           c[j * ldc_ + i] = c10::convert<at::Half>(float_v[j * m_ + i]);
0452:         }
0453:       }
0454:       return;
0455:    }
0456: #endif
0457:    gemm_stub(
0458:       at::kCPU, at::kHalf,
0459:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0460: }
0461: 
0462: void gemm(
0463:     TransposeType transa, TransposeType transb,
0464:     int64_t m, int64_t n, int64_t k,
0465:     const float alpha,
0466:     const at::BFloat16 *a, int64_t lda,
0467:     const at::BFloat16 *b, int64_t ldb,
0468:     const float beta,
0469:     float *c, int64_t ldc) {
0470:   internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0471: #if AT_BUILD_WITH_BLAS() && defined(BLAS_HAS_SBGEMM)
0472:    if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0473:       int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0474:       char transa_ = to_blas(transa), transb_ = to_blas(transb);
0475:       float alpha_ = alpha, beta_ = beta;
0476:       sbgemm_(&transa_, &transb_,
0477:               &m_, &n_, &k_,
0478:               &alpha_,
0479:               a, &lda_,
0480:               b, &ldb_,
```
- **EN**: Lines 451-480 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: gemm_stub, gemm, normalize_last_dims, AT_BUILD_WITH_BLAS.
- **CN**: 第 451-480 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：gemm_stub, gemm, normalize_last_dims, AT_BUILD_WITH_BLAS。

### Lines 481-510 / 第 481-510 行
```cpp
0481:               &beta_,
0482:               c, &ldc_);
0483:       return;
0484:    }
0485: #endif
0486: #if AT_MKLDNN_ACL_ENABLED()
0487: // add heuristic based on shape to dispatch to sbgemm_ vs MKLDNN
0488:    if (mkldnn_bf16f32_gemm(transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc)) {
0489:      return;
0490:    }
0491: #endif //AT_MKLDNN_ACL_ENABLED
0492: 
0493: #ifdef MKL_HAS_SBGEMM
0494:   if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0495:     int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0496:     mkl_gemm_bf16bf16f32(transa, transb, m_, n_, k_, alpha, a, lda_, b, ldb_, beta, c, ldc_);
0497:     return;
0498:   }
0499: #endif
0500:   gemm_no_downcast_stub(
0501:       at::kCPU, at::kBFloat16,
0502:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0503: }
0504: 
0505: void gemm(
0506:     TransposeType transa, TransposeType transb,
0507:     int64_t m, int64_t n, int64_t k,
0508:     const float alpha,
0509:     const at::Half *a, int64_t lda,
0510:     const at::Half *b, int64_t ldb,
```
- **EN**: Lines 481-510 mainly cover expressions/calls, conditional compilation, state/variable declarations. Notable symbols: AT_MKLDNN_ACL_ENABLED, mkldnn_bf16f32_gemm, use_blas_gemm, mkl_gemm_bf16bf16f32.
- **CN**: 第 481-510 行主要涉及表达式或调用、预处理条件、变量/别名声明。 值得关注的符号包括：AT_MKLDNN_ACL_ENABLED, mkldnn_bf16f32_gemm, use_blas_gemm, mkl_gemm_bf16bf16f32。

### Lines 511-540 / 第 511-540 行
```cpp
0511:     const float beta,
0512:     float *c, int64_t ldc) {
0513:   internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0514: #if AT_BUILD_WITH_BLAS() && defined(BLAS_HAS_SHGEMM)
0515:   if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0516:     int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0517:     char transa_ = to_blas(transa), transb_ = to_blas(transb);
0518:     float alpha_ = alpha, beta_ = beta;
0519:     shgemm_(&transa_, &transb_,
0520:             &m_, &n_, &k_,
0521:             &alpha_,
0522:             a, &lda_,
0523:             b, &ldb_,
0524:             &beta_,
0525:             c, &ldc_);
0526:     return;
0527:   }
0528: #endif
0529: #ifdef MKL_HAS_SHGEMM
0530:   if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0531:     int m_ = m, n_ = n, k_ = k, lda_ = lda, ldb_ = ldb, ldc_ = ldc;
0532:     mkl_gemm_f16f16f32(transa, transb, m_, n_, k_, alpha, a, lda_, b, ldb_, beta, c, ldc_);
0533:     return;
0534:   }
0535: #endif
0536:   gemm_no_downcast_stub(
0537:       at::kCPU, at::kHalf,
0538:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0539: }
0540: 
```
- **EN**: Lines 511-540 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: normalize_last_dims, AT_BUILD_WITH_BLAS, defined, use_blas_gemm.
- **CN**: 第 511-540 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：normalize_last_dims, AT_BUILD_WITH_BLAS, defined, use_blas_gemm。

### Lines 541-570 / 第 541-570 行
```cpp
0541: void gemm(
0542:     TransposeType transa, TransposeType transb,
0543:     int64_t m, int64_t n, int64_t k,
0544:     const int64_t alpha,
0545:     const int64_t *a, int64_t lda,
0546:     const int64_t *b, int64_t ldb,
0547:     const int64_t beta,
0548:     int64_t *c, int64_t ldc) {
0549:   internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0550: #ifdef USE_FBGEMM
0551:   if (alpha == 1 && (beta == 0 || beta == 1)) {
0552:     // In FBGEMM, we assume row-major ordering; However, here we assume the
0553:     // column-major ordering following the FORTRAN tradition in BLAS interface
0554:     // in this function: we can configure the layout (row/column-major ordering)
0555:     // of A and B by changing transa_ and transb_, but we cannot change the
0556:     // layout of C with this FORTRAN-style BLAS interface.
0557:     //
0558:     // The workaround is that we compute
0559:     // C^T (n x m) = B^T (n x k) * A^T (k x m) instead.
0560:     //
0561:     // In this way we view C^T as the row-major ordering when passing to FBGEMM.
0562:     fbgemm::cblas_gemm_i64_i64acc(
0563:         to_fbgemm(transb),
0564:         to_fbgemm(transa),
0565:         n,
0566:         m,
0567:         k,
0568:         b,
0569:         ldb,
0570:         a,
```
- **EN**: Lines 541-570 mainly cover expressions/calls, comments/documentation, function signatures/definitions. Notable symbols: gemm, normalize_last_dims, layout, T.
- **CN**: 第 541-570 行主要涉及表达式或调用、注释或说明、函数签名或实现。 值得关注的符号包括：gemm, normalize_last_dims, layout, T。

### Lines 571-600 / 第 571-600 行
```cpp
0571:         lda,
0572:         beta == 1,
0573:         c,
0574:         ldc);
0575:     return;
0576:   }
0577: #endif
0578: 
0579:   gemm_stub(
0580:       kCPU, kLong,
0581:       transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0582: }
0583: 
0584: template <typename scalar_t>
0585: static void gemm_batched_mkl_impl(
0586:       TransposeType transa, TransposeType transb,
0587:       int64_t batch_size, int64_t m, int64_t n, int64_t k,
0588:       scalar_t alpha,
0589:       const scalar_t **a, int64_t lda,
0590:       const scalar_t **b, int64_t ldb,
0591:       scalar_t beta,
0592:       scalar_t **c, int64_t ldc) {
0593:   for (int64_t i = 0; i < batch_size;) {
0594:     int sub_batch = std::min(batch_size - i, int64_t{INT_MAX});
0595:     mkl_gemm_batched(transa, transb, sub_batch, m, n, k, alpha,
0596:                      &a[i], lda, &b[i], ldb, beta, &c[i], ldc);
0597:     i += sub_batch;
0598:   }
0599: }
0600: 
```
- **EN**: Lines 571-600 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: gemm_stub, gemm_batched_mkl_impl, min, mkl_gemm_batched.
- **CN**: 第 571-600 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：gemm_stub, gemm_batched_mkl_impl, min, mkl_gemm_batched。

### Lines 601-630 / 第 601-630 行
```cpp
0601: template <typename scalar_t>
0602: using is_blas_library_type = std::integral_constant<bool,
0603:     std::is_same_v<scalar_t, double> ||
0604:     std::is_same_v<scalar_t, float> ||
0605:     std::is_same_v<scalar_t, c10::complex<double>> ||
0606:     std::is_same_v<scalar_t, c10::complex<float>>>;
0607: 
0608: template <typename scalar_t>
0609: static void gemm_batched_generic(
0610:     TransposeType transa, TransposeType transb,
0611:     int64_t batch_size, int64_t m, int64_t n, int64_t k,
0612:     scalar_t alpha,
0613:     const scalar_t **a, int64_t lda,
0614:     const scalar_t **b, int64_t ldb,
0615:     scalar_t beta,
0616:     scalar_t **c, int64_t ldc) {
0617:   for (const auto batch : c10::irange(batch_size)) {
0618:     gemm(transa, transb, m, n, k, alpha, a[batch], lda, b[batch], ldb, beta, c[batch], ldc);
0619:   }
0620: }
0621: 
0622: template <typename scalar_t>
0623: static void gemm_batched(
0624:     TransposeType transa, TransposeType transb,
0625:     int64_t batch_size, int64_t m, int64_t n, int64_t k,
0626:     scalar_t alpha,
0627:     const scalar_t **a, int64_t lda,
0628:     const scalar_t **b, int64_t ldb,
0629:     scalar_t beta,
0630:     scalar_t **c, int64_t ldc) {
```
- **EN**: Lines 601-630 mainly cover expressions/calls, state/variable declarations, template setup. Notable symbols: gemm_batched_generic, irange, gemm, gemm_batched.
- **CN**: 第 601-630 行主要涉及表达式或调用、变量/别名声明、模板声明。 值得关注的符号包括：gemm_batched_generic, irange, gemm, gemm_batched。

### Lines 631-660 / 第 631-660 行
```cpp
0631:   if (batch_size == 1) {
0632:     return gemm(transa, transb, m, n, k, alpha, a[0], lda, b[0], ldb, beta, c[0], ldc);
0633:   }
0634: 
0635:   if constexpr (AT_MKL_ENABLED() && is_blas_library_type<scalar_t>::value) {
0636:     internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0637:     if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0638:       gemm_batched_mkl_impl(
0639:           transa, transb, batch_size, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0640:     } else {
0641:       gemm_batched_generic(
0642:           transa, transb, batch_size, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0643:     }
0644:   } else {
0645:     gemm_batched_generic(
0646:         transa, transb, batch_size, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0647:   }
0648: }
0649: 
0650: template <typename scalar_t>
0651: static void gemm_batched_with_stride_generic(
0652:     TransposeType transa, TransposeType transb,
0653:     int64_t batch_size, int64_t m, int64_t n, int64_t k,
0654:     scalar_t alpha,
0655:     const scalar_t *a, int64_t lda, int64_t batch_stride_a,
0656:     const scalar_t *b, int64_t ldb, int64_t batch_stride_b,
0657:     scalar_t beta,
0658:     scalar_t *c, int64_t ldc, int64_t batch_stride_c) {
0659:   for (const auto batch : c10::irange(batch_size)) {
0660:     const auto a_batch = a + batch_stride_a * batch;
```
- **EN**: Lines 631-660 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: gemm, constexpr, AT_MKL_ENABLED, normalize_last_dims.
- **CN**: 第 631-660 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：gemm, constexpr, AT_MKL_ENABLED, normalize_last_dims。

### Lines 661-690 / 第 661-690 行
```cpp
0661:     const auto b_batch = b + batch_stride_b * batch;
0662:     const auto c_batch = c + batch_stride_c * batch;
0663:     gemm(transa, transb, m, n, k, alpha, a_batch, lda, b_batch, ldb, beta, c_batch, ldc);
0664:   }
0665: }
0666: 
0667: template <typename scalar_t>
0668: void gemm_batched_with_stride(
0669:     TransposeType transa, TransposeType transb,
0670:     int64_t batch_size, int64_t m, int64_t n, int64_t k,
0671:     scalar_t alpha,
0672:     const scalar_t *a, int64_t lda, int64_t batch_stride_a,
0673:     const scalar_t *b, int64_t ldb, int64_t batch_stride_b,
0674:     scalar_t beta,
0675:     scalar_t *c, int64_t ldc, int64_t batch_stride_c) {
0676:   if (batch_size == 1) {
0677:     return gemm(transa, transb, m, n, k, alpha, a, lda, b, ldb, beta, c, ldc);
0678:   }
0679: 
0680:   if constexpr (AT_MKL_ENABLED() && is_blas_library_type<scalar_t>::value) {
0681:     internal::normalize_last_dims(transa, transb, m, n, k, &lda, &ldb, &ldc);
0682:     if (use_blas_gemm(transa, transb, m, n, k, lda, ldb, ldc)) {
0683:       c10::SmallBuffer<const scalar_t*, 16> a_ptrs(batch_size);
0684:       c10::SmallBuffer<const scalar_t*, 16> b_ptrs(batch_size);
0685:       c10::SmallBuffer<scalar_t*, 16> c_ptrs(batch_size);
0686: 
0687:       for (const auto batch : c10::irange(batch_size)) {
0688:         a_ptrs[batch] = a + batch_stride_a * batch;
0689:         b_ptrs[batch] = b + batch_stride_b * batch;
0690:         c_ptrs[batch] = c + batch_stride_c * batch;
```
- **EN**: Lines 661-690 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: gemm, gemm_batched_with_stride, constexpr, AT_MKL_ENABLED.
- **CN**: 第 661-690 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：gemm, gemm_batched_with_stride, constexpr, AT_MKL_ENABLED。

### Lines 691-720 / 第 691-720 行
```cpp
0691:       }
0692:       gemm_batched_mkl_impl(
0693:           transa, transb, batch_size, m, n, k, alpha, a_ptrs.data(), lda,
0694:           b_ptrs.data(), ldb, beta, c_ptrs.data(), ldc);
0695:     } else {
0696:       gemm_batched_with_stride_generic(
0697:           transa, transb, batch_size, m, n, k, alpha, a, lda, batch_stride_a,
0698:           b, ldb, batch_stride_b, beta, c, ldc, batch_stride_c);
0699:     }
0700:   } else {
0701:     gemm_batched_with_stride_generic(transa, transb, batch_size, m, n, k, alpha,
0702:                                      a, lda, batch_stride_a, b, ldb, batch_stride_b,
0703:                                      beta, c, ldc, batch_stride_c);
0704:   }
0705: }
0706: 
0707: #define INSTANTIATE_BATCHED_GEMM(scalar_t, DType)               \
0708:   template void gemm_batched(                                   \
0709:       TransposeType transa, TransposeType transb,               \
0710:       int64_t batch_size, int64_t m, int64_t n, int64_t k,      \
0711:       scalar_t alpha,                                           \
0712:       const scalar_t **a, int64_t lda,                          \
0713:       const scalar_t **b, int64_t ldb,                          \
0714:       scalar_t beta,                                            \
0715:       scalar_t **c, int64_t ldc);                               \
0716:   template void gemm_batched_with_stride(                       \
0717:       TransposeType transa, TransposeType transb,               \
0718:       int64_t batch_size, int64_t m, int64_t n, int64_t k,      \
0719:       scalar_t alpha,                                           \
0720:       const scalar_t *a, int64_t lda, int64_t batch_stride_a,   \
```
- **EN**: Lines 691-720 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: gemm_batched_mkl_impl, data, gemm_batched_with_stride_generic, INSTANTIATE_BATCHED_GEMM.
- **CN**: 第 691-720 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：gemm_batched_mkl_impl, data, gemm_batched_with_stride_generic, INSTANTIATE_BATCHED_GEMM。

### Lines 721-750 / 第 721-750 行
```cpp
0721:       const scalar_t *b, int64_t ldb, int64_t batch_stride_b,   \
0722:       scalar_t beta,                                            \
0723:       scalar_t *c, int64_t ldc, int64_t batch_stride_c);
0724: 
0725: AT_FORALL_SCALAR_TYPES_WITH_COMPLEX_EXCEPT_COMPLEX_HALF_F8NZ(INSTANTIATE_BATCHED_GEMM)
0726: 
0727: DEFINE_DISPATCH(axpy_stub);
0728: 
0729: void axpy(int64_t n, double a, const double *x, int64_t incx, double *y, int64_t incy) {
0730:   if(n == 1)
0731:   {
0732:     incx = 1;
0733:     incy = 1;
0734:   }
0735:   #if AT_BUILD_WITH_BLAS()
0736:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) )
0737:   {
0738:     int i_n = static_cast<int>(n);
0739:     int i_incx = static_cast<int>(incx);
0740:     int i_incy = static_cast<int>(incy);
0741:     #if C10_IOS
0742:     cblas_daxpy(i_n, a, x, i_incx, y, i_incy);
0743:     #else
0744:     daxpy_(&i_n, &a, x, &i_incx, y, &i_incy);
0745:     #endif
0746:     return;
0747:   }
0748:   #endif
0749:   axpy_stub(
0750:       kCPU, at::kDouble,
```
- **EN**: Lines 721-750 mainly cover state/variable declarations, expressions/calls, conditional compilation. Notable symbols: AT_FORALL_SCALAR_TYPES_WITH_COMPLEX_EXCEPT_COMPLEX_HALF_F8NZ, DEFINE_DISPATCH, axpy, AT_BUILD_WITH_BLAS.
- **CN**: 第 721-750 行主要涉及变量/别名声明、表达式或调用、预处理条件。 值得关注的符号包括：AT_FORALL_SCALAR_TYPES_WITH_COMPLEX_EXCEPT_COMPLEX_HALF_F8NZ, DEFINE_DISPATCH, axpy, AT_BUILD_WITH_BLAS。

### Lines 751-780 / 第 751-780 行
```cpp
0751:       n, a, x, incx, y, incy);
0752: }
0753: 
0754: void axpy(int64_t n, float a, const float *x, int64_t incx, float *y, int64_t incy) {
0755:   if(n == 1)
0756:   {
0757:     incx = 1;
0758:     incy = 1;
0759:   }
0760:   #if AT_BUILD_WITH_BLAS()
0761:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) )
0762:   {
0763:     int i_n = static_cast<int>(n);
0764:     int i_incx = static_cast<int>(incx);
0765:     int i_incy = static_cast<int>(incy);
0766:     #if C10_IOS
0767:     cblas_saxpy(i_n, a, x, i_incx, y, i_incy);
0768:     #else
0769:     saxpy_(&i_n, &a, x, &i_incx, y, &i_incy);
0770:     #endif
0771:     return;
0772:   }
0773:   #endif
0774:   axpy_stub(
0775:       kCPU, at::kFloat,
0776:       n, a, x, incx, y, incy);
0777: }
0778: 
0779: void axpy(int64_t n, c10::complex<double> a, const c10::complex<double> *x, int64_t incx, c10::complex<double> *y, int64_t incy) {
0780:   if(n == 1)
```
- **EN**: Lines 751-780 mainly cover state/variable declarations, expressions/calls, conditional compilation. Notable symbols: axpy, AT_BUILD_WITH_BLAS, cblas_saxpy, saxpy_.
- **CN**: 第 751-780 行主要涉及变量/别名声明、表达式或调用、预处理条件。 值得关注的符号包括：axpy, AT_BUILD_WITH_BLAS, cblas_saxpy, saxpy_。

### Lines 781-810 / 第 781-810 行
```cpp
0781:   {
0782:     incx = 1;
0783:     incy = 1;
0784:   }
0785:   #if AT_BUILD_WITH_BLAS()
0786:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) )
0787:   {
0788:     int i_n = static_cast<int>(n);
0789:     int i_incx = static_cast<int>(incx);
0790:     int i_incy = static_cast<int>(incy);
0791:     #if C10_IOS
0792:     cblas_zaxpy(i_n, &a, x, i_incx, y, i_incy);
0793:     #else
0794:     zaxpy_(&i_n, COMPLEX_DBL(&a), COMPLEX_DBL_CONST(x), &i_incx, COMPLEX_DBL(y), &i_incy);
0795:     #endif
0796:     return;
0797:   }
0798:   #endif
0799:   axpy_stub(
0800:       kCPU, at::kComplexDouble,
0801:       n, a, x, incx, y, incy);
0802: }
0803: 
0804: void axpy(int64_t n, c10::complex<float> a, const c10::complex<float> *x, int64_t incx, c10::complex<float> *y, int64_t incy) {
0805:   if(n == 1)
0806:   {
0807:     incx = 1;
0808:     incy = 1;
0809:   }
0810:   #if AT_BUILD_WITH_BLAS()
```
- **EN**: Lines 781-810 mainly cover state/variable declarations, expressions/calls, conditional compilation. Notable symbols: AT_BUILD_WITH_BLAS, cblas_zaxpy, zaxpy_, COMPLEX_DBL.
- **CN**: 第 781-810 行主要涉及变量/别名声明、表达式或调用、预处理条件。 值得关注的符号包括：AT_BUILD_WITH_BLAS, cblas_zaxpy, zaxpy_, COMPLEX_DBL。

### Lines 811-840 / 第 811-840 行
```cpp
0811:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) )
0812:   {
0813:     int i_n = static_cast<int>(n);
0814:     int i_incx = static_cast<int>(incx);
0815:     int i_incy = static_cast<int>(incy);
0816:     #if C10_IOS
0817:     cblas_caxpy(i_n, &a, x, i_incx, y, i_incy);
0818:     #else
0819:     caxpy_(&i_n, COMPLEX_FLOAT(&a), COMPLEX_FLOAT_CONST(x), &i_incx, COMPLEX_FLOAT(y), &i_incy);
0820:     #endif
0821:     return;
0822:   }
0823:   #endif
0824:   axpy_stub(
0825:       kCPU, at::kComplexFloat,
0826:       n, a, x, incx, y, incy);
0827: }
0828: 
0829: DEFINE_DISPATCH(copy_stub);
0830: 
0831: void copy(int64_t n, const double *x, int64_t incx, double *y, int64_t incy) {
0832:   if(n == 1)
0833:   {
0834:     incx = 1;
0835:     incy = 1;
0836:   }
0837:   #if AT_BUILD_WITH_BLAS()
0838:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) ) {
0839:     int i_n = static_cast<int>(n);
0840:     int i_incx = static_cast<int>(incx);
```
- **EN**: Lines 811-840 mainly cover state/variable declarations, conditional compilation, expressions/calls. Notable symbols: cblas_caxpy, caxpy_, COMPLEX_FLOAT, COMPLEX_FLOAT_CONST.
- **CN**: 第 811-840 行主要涉及变量/别名声明、预处理条件、表达式或调用。 值得关注的符号包括：cblas_caxpy, caxpy_, COMPLEX_FLOAT, COMPLEX_FLOAT_CONST。

### Lines 841-870 / 第 841-870 行
```cpp
0841:     int i_incy = static_cast<int>(incy);
0842:     #if C10_IOS
0843:     cblas_dcopy(i_n, x, i_incx, y, i_incy);
0844:     #else
0845:     dcopy_(&i_n, x, &i_incx, y, &i_incy);
0846:     #endif
0847:     return;
0848:   }
0849:   #endif
0850:   copy_stub(
0851:       kCPU, at::kDouble,
0852:       n, x, incx, y, incy);
0853: }
0854: 
0855: void copy(int64_t n, const float *x, int64_t incx, float *y, int64_t incy) {
0856:   if(n == 1)
0857:   {
0858:     incx = 1;
0859:     incy = 1;
0860:   }
0861:   #if AT_BUILD_WITH_BLAS()
0862:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) ) {
0863:     int i_n = static_cast<int>(n);
0864:     int i_incx = static_cast<int>(incx);
0865:     int i_incy = static_cast<int>(incy);
0866:     #if C10_IOS
0867:     cblas_scopy(i_n, x, i_incx, y, i_incy);
0868:     #else
0869:     scopy_(&i_n, x, &i_incx, y, &i_incy);
0870:     #endif
```
- **EN**: Lines 841-870 mainly cover state/variable declarations, conditional compilation, expressions/calls. Notable symbols: cblas_dcopy, dcopy_, copy_stub, copy.
- **CN**: 第 841-870 行主要涉及变量/别名声明、预处理条件、表达式或调用。 值得关注的符号包括：cblas_dcopy, dcopy_, copy_stub, copy。

### Lines 871-900 / 第 871-900 行
```cpp
0871:     return;
0872:   }
0873:   #endif
0874:   copy_stub(
0875:       kCPU, at::kFloat,
0876:       n, x, incx, y, incy);
0877: }
0878: 
0879: void copy(int64_t n, const c10::complex<double> *x, int64_t incx, c10::complex<double> *y, int64_t incy) {
0880:   if(n == 1)
0881:   {
0882:     incx = 1;
0883:     incy = 1;
0884:   }
0885:   #if AT_BUILD_WITH_BLAS()
0886:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) ) {
0887:     int i_n = static_cast<int>(n);
0888:     int i_incx = static_cast<int>(incx);
0889:     int i_incy = static_cast<int>(incy);
0890:     #if C10_IOS
0891:     cblas_zcopy(i_n, x, i_incx, y, i_incy);
0892:     #else
0893:     zcopy_(&i_n, COMPLEX_DBL_CONST(x), &i_incx, COMPLEX_DBL(y), &i_incy);
0894:     #endif
0895:     return;
0896:   }
0897:   #endif
0898:   copy_stub(
0899:       kCPU, at::kComplexDouble,
0900:       n, x, incx, y, incy);
```
- **EN**: Lines 871-900 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: copy_stub, copy, AT_BUILD_WITH_BLAS, cblas_zcopy.
- **CN**: 第 871-900 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：copy_stub, copy, AT_BUILD_WITH_BLAS, cblas_zcopy。

### Lines 901-930 / 第 901-930 行
```cpp
0901: }
0902: 
0903: void copy(int64_t n, const c10::complex<float> *x, int64_t incx, c10::complex<float> *y, int64_t incy){
0904:   if(n == 1)
0905:   {
0906:     incx = 1;
0907:     incy = 1;
0908:   }
0909:   #if AT_BUILD_WITH_BLAS()
0910:   if( (n <= INT_MAX) && (incx <= INT_MAX) && (incy <= INT_MAX) ) {
0911:     int i_n = static_cast<int>(n);
0912:     int i_incx = static_cast<int>(incx);
0913:     int i_incy = static_cast<int>(incy);
0914:     #if C10_IOS
0915:     cblas_ccopy(i_n, &x, i_incx, y, i_incy);
0916:     #else
0917:     ccopy_(&i_n, COMPLEX_FLOAT(x), &i_incx, COMPLEX_FLOAT(y), &i_incy);
0918:     #endif
0919:     return;
0920:   }
0921:   #endif
0922:   copy_stub(
0923:       kCPU, at::kComplexFloat,
0924:       n, x, incx, y, incy);
0925: }
0926: 
0927: // oneDNN BRGEMM
0928: #if defined(ONEDNN_UKERNEL_ENABLED)
0929: struct BrgemmKey {
0930:   int64_t M;
```
- **EN**: Lines 901-930 mainly cover state/variable declarations, expressions/calls, conditional compilation. Notable symbols: copy, AT_BUILD_WITH_BLAS, cblas_ccopy, ccopy_.
- **CN**: 第 901-930 行主要涉及变量/别名声明、表达式或调用、预处理条件。 值得关注的符号包括：copy, AT_BUILD_WITH_BLAS, cblas_ccopy, ccopy_。

### Lines 931-960 / 第 931-960 行
```cpp
0931:   int64_t N;
0932:   int64_t K;
0933:   int64_t batch_size;
0934:   int64_t lda;
0935:   int64_t ldb;
0936:   int64_t ldc;
0937:   ScalarType dt_a;
0938:   ScalarType dt_b;
0939:   ScalarType dt_c;
0940:   bool add_C;
0941: 
0942:   BrgemmKey(
0943:       int64_t M,
0944:       int64_t N,
0945:       int64_t K,
0946:       int64_t batch_size,
0947:       int64_t lda,
0948:       int64_t ldb,
0949:       int64_t ldc,
0950:       ScalarType dt_a,
0951:       ScalarType dt_b,
0952:       ScalarType dt_c,
0953:       bool add_C)
0954:       : M(M),
0955:         N(N),
0956:         K(K),
0957:         batch_size(batch_size),
0958:         lda(lda),
0959:         ldb(ldb),
0960:         ldc(ldc),
```
- **EN**: Lines 931-960 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: BrgemmKey, M, N, K.
- **CN**: 第 931-960 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：BrgemmKey, M, N, K。

### Lines 961-990 / 第 961-990 行
```cpp
0961:         dt_a(dt_a),
0962:         dt_b(dt_b),
0963:         dt_c(dt_c),
0964:         add_C(add_C) {}
0965:   bool operator==(const BrgemmKey& other) const {
0966:     return M == other.M && N == other.N && K == other.K &&
0967:         batch_size == other.batch_size && lda == other.lda &&
0968:         ldb == other.ldb && ldc == other.ldc && dt_a == other.dt_a &&
0969:         dt_b == other.dt_b && dt_c == other.dt_c && add_C == other.add_C;
0970:   }
0971: };
0972: 
0973: struct PackKey {
0974:   int64_t K;
0975:   int64_t N;
0976:   int64_t ld_in;
0977:   int64_t ld_out;
0978:   ScalarType dt_in;
0979:   ScalarType dt_out;
0980:   PackKey(
0981:       int64_t K,
0982:       int64_t N,
0983:       int64_t ld_in,
0984:       int64_t ld_out,
0985:       ScalarType dt_in,
0986:       ScalarType dt_out)
0987:       : K(K),
0988:         N(N),
0989:         ld_in(ld_in),
0990:         ld_out(ld_out),
```
- **EN**: Lines 961-990 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: dt_a, dt_b, dt_c, add_C.
- **CN**: 第 961-990 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：dt_a, dt_b, dt_c, add_C。

### Lines 991-1020 / 第 991-1020 行
```cpp
0991:         dt_in(dt_in),
0992:         dt_out(dt_out) {}
0993:   bool operator==(const PackKey& other) const {
0994:     return N == other.N && K == other.K && ld_in == other.ld_in &&
0995:         ld_out == other.ld_out && dt_in == other.dt_in &&
0996:         dt_out == other.dt_out;
0997:   }
0998: };
0999: 
1000: static inline dnnl::memory::data_type get_dnnl_dtype(ScalarType dtype) {
1001:   using at::ScalarType;
1002:   using data_type = dnnl::memory::data_type;
1003:   switch (dtype) {
1004:     case ScalarType::Float:          return data_type::f32;
1005:     case ScalarType::BFloat16:       return data_type::bf16;
1006:     case ScalarType::Half:           return data_type::f16;
1007:     case ScalarType::Int:            return data_type::s32;
1008:     case ScalarType::Byte:           return data_type::u8;
1009:     case ScalarType::Char:           return data_type::s8;
1010:     case ScalarType::Float8_e4m3fn:  return data_type::f8_e4m3;
1011:     case ScalarType::Float8_e5m2:    return data_type::f8_e5m2;
1012:     default:
1013:       TORCH_CHECK(false, "Unsupported dtype for oneDNN");
1014:   }
1015: }
1016: 
1017: template<typename key_t>
1018: struct UnsafeUkernelKeyHasher {
1019:   std::size_t operator()(const key_t& key) const;
1020: };
```
- **EN**: Lines 991-1020 mainly cover control-flow checks, state/variable declarations, expressions/calls. Notable symbols: dt_in, dt_out, get_dnnl_dtype, TORCH_CHECK.
- **CN**: 第 991-1020 行主要涉及控制流逻辑、变量/别名声明、表达式或调用。 值得关注的符号包括：dt_in, dt_out, get_dnnl_dtype, TORCH_CHECK。

### Lines 1021-1050 / 第 1021-1050 行
```cpp
1021: 
1022: template<>
1023: std::size_t UnsafeUkernelKeyHasher<BrgemmKey>::operator()(const BrgemmKey& key) const {
1024:   // Use M, N, K add_C, and ldc to compute hash to reduce the overhead as
1025:   // batch size and data types are unlikely to change within the same kernel and
1026:   // lda/ldb are likely to be related to M, K, N or use fixed values.
1027:   std::size_t h = std::hash<int64_t>()(key.M);
1028:   h = std::hash<int64_t>()(key.N) ^ (h << 1);
1029:   h = std::hash<int64_t>()(key.K) ^ (h << 1);
1030:   h = std::hash<bool>()(key.add_C) ^ (h << 1);
1031:   h = std::hash<int64_t>()(key.ldc) ^ (h << 1);
1032:   return h;
1033: }
1034: 
1035: template<>
1036: std::size_t UnsafeUkernelKeyHasher<PackKey>::operator()(const PackKey& key) const {
1037:   // Use K and N to compute hash to reduce the overhead as
1038:   // data types are unlikely to change and
1039:   // ld_in/ld_out is likely to be related to K, N or use fixed values
1040:   std::size_t h = std::hash<int64_t>()(key.K);
1041:   h = std::hash<int64_t>()(key.N) ^ (h << 1);
1042:   return h;
1043: }
1044: 
1045: template <typename key_t, typename value_t>
1046: struct KernelCache  {
1047:   using kstore_t = std::unordered_map<key_t, std::shared_ptr<value_t>, UnsafeUkernelKeyHasher<key_t>>;
1048:   static std::shared_ptr<value_t>&& fetch_or_create(
1049:       const key_t& key,
1050:       const std::function<std::shared_ptr<value_t>()>& callback) {
```
- **EN**: Lines 1021-1050 mainly cover state/variable declarations, comments/documentation, template setup. Notable symbols: fetch_or_create.
- **CN**: 第 1021-1050 行主要涉及变量/别名声明、注释或说明、模板声明。 值得关注的符号包括：fetch_or_create。

### Lines 1051-1080 / 第 1051-1080 行
```cpp
1051:     auto&& search = get_store().find(key);
1052:     if (search != get_store().end()) {
1053:       return std::move(search->second);
1054:     } else {
1055:       get_store().insert({key, callback()});
1056:       return std::move(get_store()[key]);
1057:     }
1058:   }
1059: 
1060:   static kstore_t& get_store() {
1061:     static thread_local kstore_t cache_kernels;
1062:     return cache_kernels;
1063:   }
1064: };
1065: 
1066: // Helper struct for convenient brgemm configuration
1067: struct GemmHelper {
1068:   GemmHelper(
1069:       int64_t M,
1070:       int64_t N,
1071:       int64_t K,
1072:       int64_t bs,
1073:       int64_t ld_a,
1074:       int64_t ld_b,
1075:       int64_t ld_c,
1076:       ScalarType dt_a,
1077:       ScalarType dt_b,
1078:       ScalarType dt_c,
1079:       const bool add_C) {
1080:     // Create brgemm
```
- **EN**: Lines 1051-1080 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: get_store, find, end, move.
- **CN**: 第 1051-1080 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：get_store, find, end, move。

### Lines 1081-1110 / 第 1081-1110 行
```cpp
1081: #if defined(ONEDNN_UKERNEL_1)
1082:     brg = dnnl::ukernel::brgemm(
1083:         M,
1084:         N,
1085:         K,
1086:         bs,
1087:         ld_a,
1088:         ld_b,
1089:         ld_c,
1090:         get_dnnl_dtype(dt_a),
1091:         get_dnnl_dtype(dt_b),
1092:         get_dnnl_dtype(dt_c),
1093:         1,
1094:         add_C ? 1 : 0);
1095: #elif defined(ONEDNN_UKERNEL_2)
1096:     brg = dnnl::ukernel::brgemm(
1097:         M,
1098:         N,
1099:         K,
1100:         bs,
1101:         ld_a,
1102:         ld_b,
1103:         ld_c,
1104:         get_dnnl_dtype(dt_a),
1105:         get_dnnl_dtype(dt_b),
1106:         get_dnnl_dtype(dt_c));
1107:     brg.set_add_C(add_C);
1108:     brg.finalize();
1109: #endif
1110:     // Create a scratchpad buffer for the brgemm execution
```
- **EN**: Lines 1081-1110 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: defined, brgemm, get_dnnl_dtype, set_add_C.
- **CN**: 第 1081-1110 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：defined, brgemm, get_dnnl_dtype, set_add_C。

### Lines 1111-1140 / 第 1111-1140 行
```cpp
1111:     scratchpad = std::vector<uint8_t>(brg.get_scratchpad_size());
1112:     // Prepare default vector of pairs of tensors A and B offsets for each batch.
1113:     A_B_offsets.resize(1);
1114:     A_B_offsets[0] = std::make_pair(0, 0);
1115:   }
1116:   dnnl::ukernel::brgemm brg;
1117:   std::vector<uint8_t> scratchpad;
1118:   std::vector<std::pair<int64_t, int64_t>> A_B_offsets;
1119: };
1120: 
1121: struct Brgemm : public KernelCache <BrgemmKey, GemmHelper> {
1122:   // Fetch/create GemmHelper object and execute brgemm with batch size = 1
1123:   template <typename scalar_t_a, typename scalar_t_b, typename scalar_t_c>
1124:   static void call(
1125:       int64_t M,
1126:       int64_t N,
1127:       int64_t K,
1128:       int64_t ld_a,
1129:       int64_t ld_b,
1130:       int64_t ld_c,
1131:       const bool add_C,
1132:       const scalar_t_a* A,
1133:       const scalar_t_b* B,
1134:       scalar_t_c* C) {
1135:     auto&& key = BrgemmKey(
1136:         M,
1137:         N,
1138:         K,
1139:         1,
1140:         ld_a,
```
- **EN**: Lines 1111-1140 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: get_scratchpad_size, resize, make_pair, call.
- **CN**: 第 1111-1140 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：get_scratchpad_size, resize, make_pair, call。

### Lines 1141-1170 / 第 1141-1170 行
```cpp
1141:         ld_b,
1142:         ld_c,
1143:         c10::CppTypeToScalarType<scalar_t_a>::value,
1144:         c10::CppTypeToScalarType<scalar_t_b>::value,
1145:         c10::CppTypeToScalarType<scalar_t_c>::value,
1146:         add_C);
1147:     // Fetch/create GemmHelper object
1148:     auto&& value = fetch_or_create(key, [&]() {
1149:       auto&& v = std::make_shared<GemmHelper>(
1150:           M,
1151:           N,
1152:           K,
1153:           1,
1154:           ld_a,
1155:           ld_b,
1156:           ld_c,
1157:           c10::CppTypeToScalarType<scalar_t_a>::value,
1158:           c10::CppTypeToScalarType<scalar_t_b>::value,
1159:           c10::CppTypeToScalarType<scalar_t_c>::value,
1160:           add_C);
1161:       (*v).brg.generate();
1162:       return std::move(v);
1163:     });
1164:     if (get_current() != value) {
1165: #if defined(ONEDNN_UKERNEL_1)
1166:       dnnl::ukernel::brgemm::release_hw_context();
1167: #endif
1168:       ((*value).brg).set_hw_context();
1169:       get_current() = value;
1170:     }
```
- **EN**: Lines 1141-1170 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: fetch_or_create, generate, move, get_current.
- **CN**: 第 1141-1170 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：fetch_or_create, generate, move, get_current。

### Lines 1171-1200 / 第 1171-1200 行
```cpp
1171:     ((*value).brg)
1172:         .execute(A, B, (*value).A_B_offsets, C, (*value).scratchpad.data());
1173:   }
1174: 
1175:   static std::shared_ptr<GemmHelper>& get_current() {
1176:     static thread_local std::shared_ptr<GemmHelper> current;
1177:     return current;
1178:   }
1179: 
1180:   static bool device_check(ScalarType dtype) {
1181:     if (!at::globalContext().userEnabledMkldnn()) {
1182:       return false;
1183:     }
1184:     static bool fp16_support = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_fp16;
1185:     static bool fp32_support = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx2;
1186:     static bool bf16_support = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core;
1187:     static bool u8_support = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_amx;
1188:     static bool s8_support = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_vnni;
1189: #ifdef ONEDNN_FP8_UKERNEL_AVAILABLE
1190:     static bool f8_support = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_amx;
1191: #else
1192:     static bool f8_support = false;
1193: #endif
1194:     switch (dtype) {
1195:       case ScalarType::Half:          return fp16_support;
1196:       case ScalarType::Float:         return fp32_support;
1197:       case ScalarType::BFloat16:      return bf16_support;
1198:       case ScalarType::Byte:          return u8_support;
1199:       case ScalarType::Char:          return s8_support;
1200:       case ScalarType::Float8_e4m3fn: return f8_support;
```
- **EN**: Lines 1171-1200 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: execute, data, get_current, device_check.
- **CN**: 第 1171-1200 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：execute, data, get_current, device_check。

### Lines 1201-1230 / 第 1201-1230 行
```cpp
1201:       case ScalarType::Float8_e5m2:   return f8_support;
1202:       default:                        return false;
1203:     }
1204:   }
1205: };
1206: 
1207: #if defined(ONEDNN_UKERNEL_1)
1208: using pack_t = dnnl::ukernel::brgemm_pack_B;
1209: #elif defined(ONEDNN_UKERNEL_2)
1210: using pack_t = dnnl::ukernel::transform;
1211: #endif
1212: struct Pack : public KernelCache <PackKey, pack_t> {
1213:   static void call(
1214:       int64_t K,
1215:       int64_t N,
1216:       int64_t ld_in,
1217:       int64_t ld_out,
1218:       ScalarType dt_in,
1219:       ScalarType dt_out,
1220:       const void* in,
1221:       void* out) {
1222:     auto&& key = PackKey(K, N, ld_in, ld_out, dt_in, dt_out);
1223:     auto&& pack = fetch_or_create(key, [&]() {
1224:       auto&& p = std::make_shared<pack_t>(
1225: #if defined(ONEDNN_UKERNEL_1)
1226:           K, N, ld_in, ld_out, get_dnnl_dtype(dt_in), get_dnnl_dtype(dt_out));
1227: #elif defined(ONEDNN_UKERNEL_2)
1228:           K, N, dnnl::ukernel::pack_type::no_trans, ld_in, ld_out, get_dnnl_dtype(dt_in), get_dnnl_dtype(dt_out));
1229: #endif
1230:       if (could_pack(dt_in)) {
```
- **EN**: Lines 1201-1230 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: defined, call, PackKey, fetch_or_create.
- **CN**: 第 1201-1230 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：defined, call, PackKey, fetch_or_create。

### Lines 1231-1260 / 第 1231-1260 行
```cpp
1231:         (*p).generate();
1232:       }
1233:       return std::move(p);
1234:     });
1235:     if (could_pack(dt_in)) {
1236:       (*pack).execute(in, out);
1237:     } else {
1238:       TORCH_CHECK(false, "No need to pack");
1239:     }
1240:   }
1241: 
1242:   static bool could_pack(ScalarType dtype) {
1243:     if (!at::globalContext().userEnabledMkldnn()) {
1244:       return false;
1245:     }
1246:     static bool fp16_pack = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_amx_fp16;
1247:     static bool bf16_pack = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_amx;
1248:     static bool bit8_pack = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_amx;
1249: #ifdef ONEDNN_FP8_UKERNEL_AVAILABLE
1250:     static bool fp8_pack = dnnl::get_effective_cpu_isa() >= dnnl::cpu_isa::avx512_core_amx;
1251: #else
1252:     static bool fp8_pack = false;
1253: #endif
1254:     switch (dtype) {
1255:       case ScalarType::Half:          return fp16_pack;
1256:       case ScalarType::BFloat16:      return bf16_pack;
1257:       case ScalarType::Byte:          return bit8_pack;
1258:       case ScalarType::Char:          return bit8_pack;
1259:       case ScalarType::Float8_e4m3fn: return fp8_pack;
1260:       case ScalarType::Float8_e5m2:   return fp8_pack;
```
- **EN**: Lines 1231-1260 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: generate, move, could_pack, execute.
- **CN**: 第 1231-1260 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：generate, move, could_pack, execute。

### Lines 1261-1290 / 第 1261-1290 行
```cpp
1261:       default:                        return false;
1262:     }
1263:   }
1264: };
1265: #endif
1266: 
1267: void brgemm(
1268:     int64_t M,
1269:     int64_t N,
1270:     int64_t K,
1271:     int64_t ld_a,
1272:     int64_t ld_b,
1273:     int64_t ld_c,
1274:     const bool add_C,
1275:     const float* A,
1276:     const float* B,
1277:     float* C,
1278:     bool is_vnni) {
1279: 
1280:   TORCH_CHECK(!is_vnni,
1281:     "Float Brgemm does not support vnni layout.");
1282: 
1283: #if defined(ONEDNN_UKERNEL_ENABLED)
1284:   if (Brgemm::device_check(ScalarType::Float)) {
1285:     Brgemm::call<float, float, float>(
1286:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1287:     return;
1288:   }
1289: #endif
1290:   // fallback path
```
- **EN**: Lines 1261-1290 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: brgemm, TORCH_CHECK, defined, device_check.
- **CN**: 第 1261-1290 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：brgemm, TORCH_CHECK, defined, device_check。

### Lines 1291-1320 / 第 1291-1320 行
```cpp
1291:   auto beta = add_C ? 1 : 0;
1292:   gemm(
1293:     at::native::TransposeType::NoTranspose,
1294:     at::native::TransposeType::NoTranspose,
1295:     N, M, K, 1,
1296:     B, ld_b, A, ld_a,
1297:     beta, C, ld_c);
1298: }
1299: 
1300: void brgemm(
1301:     int64_t M,
1302:     int64_t N,
1303:     int64_t K,
1304:     int64_t ld_a,
1305:     int64_t ld_b,
1306:     int64_t ld_c,
1307:     const bool add_C,
1308:     const at::BFloat16* A,
1309:     const at::BFloat16* B,
1310:     float* C,
1311:     bool is_vnni) {
1312: #if defined(ONEDNN_UKERNEL_ENABLED)
1313:   if (is_vnni && Brgemm::device_check(ScalarType::BFloat16)) {
1314:     Brgemm::call<at::BFloat16, at::BFloat16, float>(
1315:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1316:     return;
1317:   }
1318: #endif
1319:   // fallback path
1320:   TORCH_CHECK(!is_vnni,
```
- **EN**: Lines 1291-1320 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: gemm, brgemm, defined, device_check.
- **CN**: 第 1291-1320 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：gemm, brgemm, defined, device_check。

### Lines 1321-1350 / 第 1321-1350 行
```cpp
1321:     "BFloat16 Brgemm VNNI format is only supported on X64 when oneDNN ukernel is enabled and `amx` is supported");
1322:   auto beta = add_C ? 1 : 0;
1323:   gemm(
1324:     at::native::TransposeType::NoTranspose,
1325:     at::native::TransposeType::NoTranspose,
1326:     N, M, K, 1,
1327:     B, ld_b, A, ld_a,
1328:     beta, C, ld_c);
1329: }
1330: 
1331: void brgemm(
1332:     int64_t M,
1333:     int64_t N,
1334:     int64_t K,
1335:     int64_t ld_a,
1336:     int64_t ld_b,
1337:     int64_t ld_c,
1338:     const bool add_C,
1339:     const at::Half* A,
1340:     const at::Half* B,
1341:     float* C,
1342:     bool is_vnni) {
1343: #if defined(ONEDNN_UKERNEL_ENABLED)
1344:   if (is_vnni && Brgemm::device_check(ScalarType::Half)) {
1345:     Brgemm::call<at::Half, at::Half, float>(
1346:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1347:     return;
1348:   }
1349: #endif
1350:   // fallback path
```
- **EN**: Lines 1321-1350 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: gemm, brgemm, defined, device_check.
- **CN**: 第 1321-1350 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：gemm, brgemm, defined, device_check。

### Lines 1351-1380 / 第 1351-1380 行
```cpp
1351:   TORCH_CHECK(!is_vnni,
1352:     "Half Brgemm VNNI format is only supported on X64 when oneDNN ukernel is enabled and `amx_fp16` is supported");
1353:   auto beta = add_C ? 1 : 0;
1354:   gemm(
1355:     at::native::TransposeType::NoTranspose,
1356:     at::native::TransposeType::NoTranspose,
1357:     N, M, K, 1,
1358:     B, ld_b, A, ld_a,
1359:     beta, C, ld_c);
1360: }
1361: 
1362: void brgemm(
1363:     int64_t M,
1364:     int64_t N,
1365:     int64_t K,
1366:     int64_t ld_a,
1367:     int64_t ld_b,
1368:     int64_t ld_c,
1369:     const bool add_C,
1370:     const unsigned char* A,
1371:     const unsigned char* B,
1372:     int32_t* C,
1373:     bool is_vnni) {
1374: #if defined(ONEDNN_UKERNEL_ENABLED)
1375:   if (is_vnni && Brgemm::device_check(ScalarType::Byte)) {
1376:     Brgemm::call<unsigned char, unsigned char, int32_t>(
1377:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1378:     return;
1379:   }
1380: #endif
```
- **EN**: Lines 1351-1380 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: TORCH_CHECK, gemm, brgemm, defined.
- **CN**: 第 1351-1380 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：TORCH_CHECK, gemm, brgemm, defined。

### Lines 1381-1410 / 第 1381-1410 行
```cpp
1381:   // raise an error if the path is not supported
1382:   TORCH_CHECK(false,
1383:     "U8 Brgemm is only supported on X64 when oneDNN ukernel is enabled and `amx` is supported");
1384: }
1385: 
1386: void brgemm(
1387:     int64_t M,
1388:     int64_t N,
1389:     int64_t K,
1390:     int64_t ld_a,
1391:     int64_t ld_b,
1392:     int64_t ld_c,
1393:     const bool add_C,
1394:     const unsigned char* A,
1395:     const signed char* B,
1396:     int32_t* C,
1397:     bool is_vnni) {
1398: #if defined(ONEDNN_UKERNEL_ENABLED)
1399:   if (is_vnni && Brgemm::device_check(ScalarType::Char)) {
1400:     Brgemm::call<unsigned char, signed char, int32_t>(
1401:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1402:     return;
1403:   }
1404: #endif
1405:   // raise an error if the path is not supported
1406:   TORCH_CHECK(false,
1407:     "I8 Brgemm is only supported on X64 when oneDNN ukernel is enabled and `amx` is supported");
1408: }
1409: 
1410: void brgemm(
```
- **EN**: Lines 1381-1410 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: TORCH_CHECK, brgemm, defined, device_check.
- **CN**: 第 1381-1410 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：TORCH_CHECK, brgemm, defined, device_check。

### Lines 1411-1440 / 第 1411-1440 行
```cpp
1411:     int64_t M,
1412:     int64_t N,
1413:     int64_t K,
1414:     int64_t ld_a,
1415:     int64_t ld_b,
1416:     int64_t ld_c,
1417:     const bool add_C,
1418:     const signed char* A,
1419:     const signed char* B,
1420:     int32_t* C,
1421:     bool is_vnni) {
1422: #if defined(ONEDNN_UKERNEL_ENABLED)
1423:   if (is_vnni && Brgemm::device_check(ScalarType::Char)) {
1424:     Brgemm::call<signed char, signed char, int32_t>(
1425:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1426:     return;
1427:   }
1428: #endif
1429:   // raise an error if the path is not supported
1430:   TORCH_CHECK(false,
1431:     "I8 Brgemm is only supported on X64 when oneDNN ukernel is enabled and `amx` is supported");
1432: }
1433: 
1434: void brgemm(
1435:     int64_t M,
1436:     int64_t N,
1437:     int64_t K,
1438:     int64_t ld_a,
1439:     int64_t ld_b,
1440:     int64_t ld_c,
```
- **EN**: Lines 1411-1440 mainly cover expressions/calls, conditional compilation, state/variable declarations. Notable symbols: defined, device_check, TORCH_CHECK, brgemm.
- **CN**: 第 1411-1440 行主要涉及表达式或调用、预处理条件、变量/别名声明。 值得关注的符号包括：defined, device_check, TORCH_CHECK, brgemm。

### Lines 1441-1470 / 第 1441-1470 行
```cpp
1441:     const bool add_C,
1442:     const at::Float8_e4m3fn* A,
1443:     const at::Float8_e4m3fn* B,
1444:     float* C,
1445:     bool is_vnni) {
1446: #if defined(ONEDNN_UKERNEL_ENABLED) && defined(ONEDNN_FP8_UKERNEL_AVAILABLE)
1447:   if (is_vnni && Brgemm::device_check(ScalarType::Float8_e4m3fn)) {
1448:     Brgemm::call<at::Float8_e4m3fn, at::Float8_e4m3fn, float>(
1449:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1450:     return;
1451:   }
1452: #endif
1453:   // raise an error if the path is not supported
1454:   TORCH_CHECK(false,
1455:     "F8 Brgemm is only supported on X64 when oneDNN ukernel is enabled and `amx` is supported");
1456: }
1457: 
1458: void brgemm(
1459:     int64_t M,
1460:     int64_t N,
1461:     int64_t K,
1462:     int64_t ld_a,
1463:     int64_t ld_b,
1464:     int64_t ld_c,
1465:     const bool add_C,
1466:     const at::Float8_e5m2* A,
1467:     const at::Float8_e5m2* B,
1468:     float* C,
1469:     bool is_vnni) {
1470: #if defined(ONEDNN_UKERNEL_ENABLED) && defined(ONEDNN_FP8_UKERNEL_AVAILABLE)
```
- **EN**: Lines 1441-1470 mainly cover expressions/calls, conditional compilation, function signatures/definitions. Notable symbols: defined, device_check, TORCH_CHECK, brgemm.
- **CN**: 第 1441-1470 行主要涉及表达式或调用、预处理条件、函数签名或实现。 值得关注的符号包括：defined, device_check, TORCH_CHECK, brgemm。

### Lines 1471-1500 / 第 1471-1500 行
```cpp
1471:   if (is_vnni && Brgemm::device_check(ScalarType::Float8_e5m2)) {
1472:     Brgemm::call<at::Float8_e5m2, at::Float8_e5m2, float>(
1473:       M, N, K, ld_a, ld_b, ld_c, add_C, A, B, C);
1474:     return;
1475:   }
1476: #endif
1477:   // raise an error if the path is not supported
1478:   TORCH_CHECK(false,
1479:     "F8 Brgemm is only supported on X64 when oneDNN ukernel is enabled and `amx` is supported");
1480: }
1481: 
1482: void brgemm_release(bool is_vnni) {
1483: #if defined(ONEDNN_UKERNEL_ENABLED)
1484:   if (is_vnni) {
1485:     dnnl::ukernel::brgemm::release_hw_context();
1486:     Brgemm::get_current() = nullptr;
1487:   }
1488: #endif
1489: }
1490: 
1491: void pack(
1492:     int64_t K,
1493:     int64_t N,
1494:     int64_t ld_in,
1495:     int64_t ld_out,
1496:     ScalarType dt_in,
1497:     ScalarType dt_out,
1498:     const void* in,
1499:     void* out) {
1500: #if defined(ONEDNN_UKERNEL_ENABLED)
```
- **EN**: Lines 1471-1500 mainly cover expressions/calls, state/variable declarations, conditional compilation. Notable symbols: device_check, TORCH_CHECK, brgemm_release, defined.
- **CN**: 第 1471-1500 行主要涉及表达式或调用、变量/别名声明、预处理条件。 值得关注的符号包括：device_check, TORCH_CHECK, brgemm_release, defined。

### Lines 1501-1515 / 第 1501-1515 行
```cpp
1501:   Pack::call(K, N, ld_in, ld_out, dt_in, dt_out, in, out);
1502: #else
1503:   TORCH_CHECK(false, "pack is only supported on X64 with oneDNN ukernel enabled");
1504: #endif
1505: }
1506: 
1507: bool could_pack(ScalarType dt_in) {
1508: #if defined(ONEDNN_UKERNEL_ENABLED)
1509:   return Pack::could_pack(dt_in);
1510: #else
1511:   return false;
1512: #endif
1513: }
1514: 
1515: } // namespace at::native::cpublas
```
- **EN**: Lines 1501-1515 mainly cover conditional compilation, expressions/calls, return paths. Notable symbols: call, TORCH_CHECK, could_pack, defined.
- **CN**: 第 1501-1515 行主要涉及预处理条件、表达式或调用、返回路径。 值得关注的符号包括：call, TORCH_CHECK, could_pack, defined。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Dispatch stub definition  
  **CN**: 调度桩定义
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Scalar/tensor mixed arithmetic  
  **CN**: 标量与张量混合运算

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/native/CPUBlas.h>`, `<ATen/native/mkl/LinearAlgebra.h>`, `<ATen/native/mkldnn/Matmul.h>`, `<c10/util/SmallBuffer.h>`, `<c10/util/irange.h>`, `<climits>`, `<cpuinfo.h>`, `<Accelerate/Accelerate.h>`, `<fbgemm/FbgemmI64.h>`, `<ideep.hpp>` ...
- **Macros / 宏**: `TORCH_CHECK`, `DEFINE_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
