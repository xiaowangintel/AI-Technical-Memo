# GemmRocblas.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/cuda/tunable/GemmRocblas.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-side ATen helpers, kernels, runtime wrappers, or performance utilities. This specific file centers on `GemmRocblas.h`. Autotuning, benchmark selection, or runtime choice of fast kernels is part of the responsibility. Matrix multiplication and GEMM-style kernels are a central concern. Descriptor/handle lifecycle management is important here.
- **Purpose (CN)**: 实现 CUDA 侧的 ATen 辅助逻辑、内核、运行时包装器或性能工具。 该文件具体围绕 `GemmRocblas.h` 展开。 该文件还负责自动调优、基准测试后的最优实现选择，或运行时快速内核的挑选。 矩阵乘法与 GEMM 风格内核是该文件的核心关注点。 描述符/句柄的生命周期管理是这里的重要内容。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

```cpp
0001: // Copyright (c) Microsoft Corporation. All rights reserved.
0002: // Licensed under the MIT License.
0003: 
0004: #pragma once
0005: 
0006: #include <ATen/cuda/CUDAContext.h>
0007: #include <ATen/cuda/tunable/TunableOp.h>
0008: #include <ATen/cuda/tunable/GemmCommon.h>
0009: #include <c10/util/StringUtil.h>
0010: #include <fmt/printf.h>
0011: 
0012: #define ROCBLAS_BETA_FEATURES_API
0013: #include <rocblas/rocblas.h>
0014: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。

### Lines 15-28 / 第 15-28 行

```cpp
0015: #define TORCH_ROCBLAS_CHECK(EXPR)                 \
0016:   do {                                            \
0017:     rocblas_status __err = EXPR;                  \
0018:     TORCH_CHECK(__err == rocblas_status_success,  \
0019:                 "rocblas error: ",                \
0020:                 rocblas_status_to_string(__err),  \
0021:                 " when calling `" #EXPR "`");     \
0022:   } while (0)
0023: 
0024: namespace at::cuda::tunable {
0025: 
0026: template <typename T>
0027: constexpr rocblas_datatype RocBlasDataTypeFor();
0028: 
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: `RocBlasDataTypeFor`.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：`RocBlasDataTypeFor`。

### Lines 29-43 / 第 29-43 行

```cpp
0029: template <>
0030: constexpr rocblas_datatype RocBlasDataTypeFor<float>() {
0031:   return rocblas_datatype_f32_r;
0032: }
0033: 
0034: template <>
0035: constexpr rocblas_datatype RocBlasDataTypeFor<double>() {
0036:   return rocblas_datatype_f64_r;
0037: }
0038: 
0039: template <>
0040: constexpr rocblas_datatype RocBlasDataTypeFor<Half>() {
0041:   return rocblas_datatype_f16_r;
0042: }
0043: 
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: no prominent local symbols.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：无明显局部符号。

### Lines 44-58 / 第 44-58 行

```cpp
0044: template <>
0045: constexpr rocblas_datatype RocBlasDataTypeFor<BFloat16>() {
0046:   return rocblas_datatype_bf16_r;
0047: }
0048: 
0049: template <>
0050: constexpr rocblas_datatype RocBlasDataTypeFor<c10::complex<float>>() {
0051:   return rocblas_datatype_f32_c;
0052: }
0053: 
0054: template <>
0055: constexpr rocblas_datatype RocBlasDataTypeFor<c10::complex<double>>() {
0056:   return rocblas_datatype_f64_c;
0057: }
0058: 
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: no prominent local symbols.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：无明显局部符号。

### Lines 59-72 / 第 59-72 行

```cpp
0059: template <typename T>
0060: constexpr rocblas_datatype RocBlasComputeTypeFor();
0061: 
0062: template <>
0063: constexpr rocblas_datatype RocBlasComputeTypeFor<float>() {
0064:   return rocblas_datatype_f32_r;
0065: }
0066: 
0067: template <>
0068: constexpr rocblas_datatype RocBlasComputeTypeFor<double>() {
0069:   return rocblas_datatype_f64_r;
0070: }
0071: 
0072: template <>
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: `RocBlasComputeTypeFor`.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：`RocBlasComputeTypeFor`。

### Lines 73-89 / 第 73-89 行

```cpp
0073: constexpr rocblas_datatype RocBlasComputeTypeFor<Half>() {
0074:   // Note that we're returning the _compute_ type for a given datatype.
0075:   // As of 12/2022, using compute type FP16 for 16-bit floats was much
0076:   // slower than using compute type FP32. So we use FP32 compute even for
0077:   // FP16 datatypes. This is how GEMM is implemented even in the function
0078:   // rocblasGemmHelper (see fpgeneric.h)
0079:   return rocblas_datatype_f32_r;
0080: }
0081: 
0082: template <>
0083: constexpr rocblas_datatype RocBlasComputeTypeFor<BFloat16>() {
0084:   // Note that we're returning the _compute_ type for a given datatype.
0085:   // As of 12/2022, using compute type FP16 for 16-bit floats was much
0086:   // slower than using compute type FP32. So we use FP32 compute even for
0087:   // BF16 datatypes. This is how GEMM is implemented even in the function
0088:   // rocblasGemmHelper (see fpgeneric.h)
0089:   return rocblas_datatype_f32_r;
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: no prominent local symbols.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：无明显局部符号。

### Lines 90-104 / 第 90-104 行

```cpp
0090: }
0091: 
0092: template <>
0093: constexpr rocblas_datatype RocBlasComputeTypeFor<c10::complex<float>>() {
0094:   return rocblas_datatype_f32_c;
0095: }
0096: 
0097: template <>
0098: constexpr rocblas_datatype RocBlasComputeTypeFor<c10::complex<double>>() {
0099:   return rocblas_datatype_f64_c;
0100: }
0101: 
0102: template <typename T>
0103: auto DoCastForHalfOrBfloat16(const T fp) {
0104:   return fp;
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: `DoCastForHalfOrBfloat16`.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：`DoCastForHalfOrBfloat16`。

### Lines 105-118 / 第 105-118 行

```cpp
0105: }
0106: 
0107: template <>
0108: inline auto DoCastForHalfOrBfloat16<Half>(const Half fp) {
0109:   // alpha and beta should be the same as compute_type, in Half case it is float.
0110:   float h = fp;
0111:   return h;
0112: }
0113: 
0114: template <>
0115: inline auto DoCastForHalfOrBfloat16<BFloat16>(const BFloat16 fp) {
0116:   // alpha and beta should be the same as compute_type, in bfloat16 case it is float.
0117:   float h = fp;
0118:   return h;
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: no prominent local symbols.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：无明显局部符号。

### Lines 119-133 / 第 119-133 行

```cpp
0119: }
0120: 
0121: static rocblas_operation _rocblasOpFromChar(char op) {
0122:   switch (op) {
0123:     case 'n':
0124:     case 'N':
0125:       return rocblas_operation_none;
0126:     case 't':
0127:     case 'T':
0128:       return rocblas_operation_transpose;
0129:     case 'c':
0130:     case 'C':
0131:       return rocblas_operation_conjugate_transpose;
0132:   }
0133:   TORCH_CHECK(false,
```

- **EN:** This block produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; validates runtime invariants before continuing. Key symbols: `_rocblasOpFromChar`.
- **CN:** 该代码块返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；在继续执行前校验运行时不变量。关键符号：`_rocblasOpFromChar`。

### Lines 134-161 / 第 134-161 行

```cpp
0134:       "_rocblasOpFromChar input should be 't', 'n' or 'c' but got `", op, "`");
0135: }
0136: 
0137: template <typename T>
0138: class RocblasGemmOp : public Callable<GemmParams<T>> {
0139:   public:
0140:     RocblasGemmOp(int solution) : solution_{solution} {}
0141: 
0142:     TuningStatus Call(const GemmParams<T>* params) override {
0143:       auto input_output_type = RocBlasDataTypeFor<T>();
0144:       if (at::globalContext().float32Precision(at::Float32Backend::CUDA, at::Float32Op::MATMUL) == at::Float32Precision::TF32 && input_output_type == rocblas_datatype_f32_r)
0145:         return FAIL;  // no support for TF32 in rocBLAS
0146:       auto compute_type = RocBlasComputeTypeFor<T>();
0147:       auto h_a = DoCastForHalfOrBfloat16(params->alpha);
0148:       auto h_b = DoCastForHalfOrBfloat16(params->beta);
0149:       auto status = rocblas_gemm_ex(
0150:           (rocblas_handle)at::cuda::getCurrentCUDABlasHandle(),
0151:           _rocblasOpFromChar(params->transa),
0152:           _rocblasOpFromChar(params->transb),
0153:           params->m, params->n, params->k,
0154:           &h_a,
0155:           params->a, input_output_type, params->lda,
0156:           params->b, input_output_type, params->ldb,
0157:           &h_b,
0158:           params->c, input_output_type, params->ldc,
0159:           params->c, input_output_type, params->ldc,
0160:           compute_type,
0161:           rocblas_gemm_algo_solution_index,
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RocblasGemmOp`, `Call`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RocblasGemmOp`, `Call`。

### Lines 162-181 / 第 162-181 行

```cpp
0162:           solution_,
0163:           rocblas_gemm_flags_none);
0164:       if (status != rocblas_status_success) {
0165:         return FAIL;
0166:       }
0167:       return OK;
0168:     }
0169: 
0170:   private:
0171:     int solution_;
0172: };
0173: 
0174: template <typename T>
0175: auto GetRocBlasGemmTypeStringAndOps() {
0176:   rocblas_handle handle = (rocblas_handle)at::cuda::getCurrentCUDABlasHandle();
0177:   int solution_size;
0178:   auto input_output_type = RocBlasDataTypeFor<T>();
0179:   auto compute_type = RocBlasComputeTypeFor<T>();
0180:   // Get the number of available solutions
0181:   TORCH_ROCBLAS_CHECK(rocblas_gemm_ex_get_solutions_by_type(handle,
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: `GetRocBlasGemmTypeStringAndOps`.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：`GetRocBlasGemmTypeStringAndOps`。

### Lines 182-198 / 第 182-198 行

```cpp
0182:                                                             input_output_type,
0183:                                                             input_output_type,
0184:                                                             compute_type,
0185:                                                             rocblas_gemm_flags_none,
0186:                                                             nullptr,
0187:                                                             &solution_size));
0188:   std::vector<int> solutions(solution_size);
0189:   // Get the list of available solutions
0190:   TORCH_ROCBLAS_CHECK(rocblas_gemm_ex_get_solutions_by_type(handle,
0191:                                                             input_output_type,
0192:                                                             input_output_type,
0193:                                                             compute_type,
0194:                                                             rocblas_gemm_flags_none,
0195:                                                             solutions.data(),
0196:                                                             &solution_size));
0197:   std::vector<std::pair<std::string, std::unique_ptr<Callable<GemmParams<T>>>>> ret;
0198:   for (size_t i = 0; i < solutions.size(); ++i) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: `solutions`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`solutions`。

### Lines 199-212 / 第 199-212 行

```cpp
0199:     auto callable = std::make_unique<RocblasGemmOp<T>>(solutions[i]);
0200:     ret.emplace_back(std::make_pair(fmt::sprintf("Gemm_Rocblas_%d", solutions[i]), std::move(callable)));
0201:   }
0202:   return ret;
0203: }
0204: 
0205: template <typename T>
0206: class RocblasGemmStridedBatchedOp : public Callable<GemmStridedBatchedParams<T>> {
0207:   public:
0208:     RocblasGemmStridedBatchedOp(int solution) : solution_{solution} {}
0209: 
0210:     TuningStatus Call(const GemmStridedBatchedParams<T>* params) override {
0211:       auto input_output_type = RocBlasDataTypeFor<T>();
0212:       if (at::globalContext().float32Precision(at::Float32Backend::CUDA, at::Float32Op::MATMUL) == at::Float32Precision::TF32 && input_output_type == rocblas_datatype_f32_r)
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RocblasGemmStridedBatchedOp`, `Call`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RocblasGemmStridedBatchedOp`, `Call`。

### Lines 213-233 / 第 213-233 行

```cpp
0213:         return FAIL;  // no support for TF32 in rocBLAS
0214:       auto compute_type = RocBlasComputeTypeFor<T>();
0215:       auto h_a = DoCastForHalfOrBfloat16(params->alpha);
0216:       auto h_b = DoCastForHalfOrBfloat16(params->beta);
0217:       auto status = rocblas_gemm_strided_batched_ex(
0218:           (rocblas_handle)at::cuda::getCurrentCUDABlasHandle(),
0219:           _rocblasOpFromChar(params->transa),
0220:           _rocblasOpFromChar(params->transb),
0221:           params->m, params->n, params->k,
0222:           &h_a,
0223:           params->a, input_output_type, params->lda, params->stride_a,
0224:           params->b, input_output_type, params->ldb, params->stride_b,
0225:           &h_b,
0226:           params->c, input_output_type, params->ldc, params->stride_c,
0227:           params->c, input_output_type, params->ldc, params->stride_c,
0228:           params->batch,
0229:           compute_type,
0230:           rocblas_gemm_algo_solution_index,
0231:           solution_,
0232:           rocblas_gemm_flags_none);
0233:       if (status != rocblas_status_success) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: `_rocblasOpFromChar`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`_rocblasOpFromChar`。

### Lines 234-250 / 第 234-250 行

```cpp
0234:         return FAIL;
0235:       }
0236:       return OK;
0237:     }
0238: 
0239:   private:
0240:     int solution_;
0241: };
0242: 
0243: template <typename T>
0244: auto GetRocBlasGemmStridedBatchedTypeStringAndOps() {
0245:   rocblas_handle handle = (rocblas_handle)at::cuda::getCurrentCUDABlasHandle();
0246:   int solution_size;
0247:   auto input_output_type = RocBlasDataTypeFor<T>();
0248:   auto compute_type = RocBlasComputeTypeFor<T>();
0249:   // Get the number of available solutions
0250:   TORCH_ROCBLAS_CHECK(rocblas_gemm_ex_get_solutions_by_type(handle,
```

- **EN:** Builds a reusable template/helper layer around `GemmRocblas`. Key symbols: `GetRocBlasGemmStridedBatchedTypeStringAndOps`.
- **CN:** 围绕 `GemmRocblas` 构建可复用的模板或辅助层。关键符号：`GetRocBlasGemmStridedBatchedTypeStringAndOps`。

### Lines 251-268 / 第 251-268 行

```cpp
0251:                                                             input_output_type,
0252:                                                             input_output_type,
0253:                                                             compute_type,
0254:                                                             rocblas_gemm_flags_none,
0255:                                                             nullptr,
0256:                                                             &solution_size));
0257:   std::vector<int> solutions(solution_size);
0258:   // Get the list of available solutions
0259:   TORCH_ROCBLAS_CHECK(rocblas_gemm_ex_get_solutions_by_type(handle,
0260:                                                             input_output_type,
0261:                                                             input_output_type,
0262:                                                             compute_type,
0263:                                                             rocblas_gemm_flags_none,
0264:                                                             solutions.data(),
0265:                                                             &solution_size));
0266:   // Sort the solutions in ascending order to make the solution vector deterministic across runs
0267:   std::sort(solutions.begin(), solutions.end());
0268: 
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: `solutions`, `sort`.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`solutions`, `sort`。

### Lines 269-277 / 第 269-277 行

```cpp
0269:   std::vector<std::pair<std::string, std::unique_ptr<Callable<GemmStridedBatchedParams<T>>>>> ret;
0270:   for (size_t i = 0; i < solutions.size(); ++i) {
0271:     auto callable = std::make_unique<RocblasGemmStridedBatchedOp<T>>(solutions[i]);
0272:     ret.emplace_back(std::make_pair(c10::str("Gemm_Rocblas_", solutions[i]), std::move(callable)));
0273:   }
0274:   return ret;
0275: }
0276: 
0277: }  // namespace at::cuda::tunable
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **CUDA backend plumbing** — CUDA 后端基础设施
- **Tensor metadata and value flow** — 张量元数据与数值流
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Linear algebra backend integration** — 线性代数后端集成
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: RocblasGemmOp, RocblasGemmStridedBatchedOp, RocBlasDataTypeFor, RocBlasComputeTypeFor, DoCastForHalfOrBfloat16, _rocblasOpFromChar, Call, GetRocBlasGemmTypeStringAndOps** — 核心符号：RocblasGemmOp、RocblasGemmStridedBatchedOp、RocBlasDataTypeFor、RocBlasComputeTypeFor、DoCastForHalfOrBfloat16、_rocblasOpFromChar、Call、GetRocBlasGemmTypeStringAndOps

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/cuda/CUDAContext.h`, `ATen/cuda/tunable/TunableOp.h`, `ATen/cuda/tunable/GemmCommon.h`, `c10/util/StringUtil.h`
- **External includes / 外部头文件**: `fmt/printf.h`, `rocblas/rocblas.h`
- **Namespaces / 命名空间**: `at::cuda::tunable`
- **Representative symbols / 代表性符号**: `RocblasGemmOp`, `RocblasGemmStridedBatchedOp`, `RocBlasDataTypeFor`, `RocBlasComputeTypeFor`, `DoCastForHalfOrBfloat16`, `_rocblasOpFromChar`, `Call`, `GetRocBlasGemmTypeStringAndOps`, `solutions`, `GetRocBlasGemmStridedBatchedTypeStringAndOps`, `sort`
