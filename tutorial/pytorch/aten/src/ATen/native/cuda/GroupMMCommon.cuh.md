# GroupMMCommon.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/GroupMMCommon.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `prepare_grouped_gemm_data`.
- 用途（中文）: 声明或定义与 `prepare_grouped_gemm_data` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #pragma once
   2: #include <cutlass/util/packed_stride.hpp>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cutlass/util/packed_stride.hpp>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cutlass/util/packed_stride.hpp>`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 4-25
```cpp
   4: namespace at::cuda::detail {
   5: 
   6: using Strides = std::array<int64_t, 3>;
   7: 
   8: template <
   9:     typename DtypeA,
  10:     typename DtypeB,
  11:     typename DtypeOutput,
  12:     typename DtypeScale,
  13:     typename ProblemShape,
  14:     typename StrideA,
  15:     typename StrideB,
  16:     typename StrideOutput>
  17: __global__ void prepare_grouped_gemm_data(
  18:     DtypeA* A,
  19:     DtypeB* B,
  20:     DtypeOutput* output,
  21:     DtypeScale* scale_A,
  22:     DtypeScale* scale_B,
  23:     DtypeA** A_ptrs,
  24:     DtypeB** B_ptrs,
  25:     DtypeOutput** output_ptrs,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `prepare_grouped_gemm_data`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `prepare_grouped_gemm_data`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 26-39
```cpp
  26:     DtypeScale** inputA_scale_ptrs,
  27:     DtypeScale** inputB_scale_ptrs,
  28:     ProblemShape* problem_sizes,
  29:     // Strides for cutlass, cute::Stride
  30:     StrideA* stride_A,
  31:     StrideB* stride_B,
  32:     StrideOutput* stride_output,
  33:     const int32_t* offs,
  34:     int32_t M,
  35:     int32_t N,
  36:     int32_t K,
  37:     // Original strides of the input tensors
  38:     Strides tensor_StrideA,
  39:     Strides tensor_StrideB,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 40-61
```cpp
  40:     Strides tensor_StrideOutput,
  41:     Strides tensor_ShapeA,
  42:     Strides tensor_ShapeB,
  43:     int64_t a_scale_stride,
  44:     int64_t b_scale_stride,
  45:     bool a_row_major = true,
  46:     bool b_row_major = false) {
  47:   int32_t tid = threadIdx.x;
  48:   int32_t delta = 0;
  49:   int32_t offset = 0;
  50:   if (offs != nullptr) {
  51:     int32_t start = tid == 0 ? 0 : offs[tid - 1];
  52:     offset = offs[tid];
  53:     delta = offset - start;
  54:     CUDA_KERNEL_ASSERT(delta >=0 && "expected gemm dimension to be greater or equal 0\n");
  55: 
  56:     // TMA transfers require global memory tensor addresses to be
  57:     // aligned to 16 bytes.
  58:     if (tid < blockDim.x - 1) {
  59:       // Check this requirement for input tensors, in case group
  60:       // addresses are increased along the dynamic dimension.
  61:       if ((K < 0 && a_row_major) ||       // 2D/2D: check along K dimension
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 62-74
```cpp
  62:           (M < 0 && !a_row_major)) {      // 3D/2D: check along N dimension
  63:         int align = 128 / cutlass::sizeof_bits<DtypeA>::value;
  64:         CUDA_KERNEL_ASSERT(
  65:                            delta % align == 0 &&
  66:                            "expected input tensor dynamic dimension byte size to be non-negative multiple of 16\n");
  67:       }
  68:       if ((K < 0 && !b_row_major) ||      // 2D/2D: check along K dimension
  69:           (N < 0 && b_row_major)) {       // 3D/2D: check along N dimension
  70:         int align = 128 / cutlass::sizeof_bits<DtypeB>::value;
  71:         CUDA_KERNEL_ASSERT(
  72:                            delta % align == 0 &&
  73:                            "expected input tensor dynamic dimension byte size to be non-negative multiple of 16\n");
  74:       }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 76-77
```cpp
  76:       // Check the same requirement for output tensor (that is always
  77:       // contiguous, and in row-major layout).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 78-91
```cpp
  78:       if (N < 0) {
  79:         int align = 128 / cutlass::sizeof_bits<DtypeOutput>::value;
  80:         CUDA_KERNEL_ASSERT(
  81:                            delta % align == 0 &&
  82:                            "expected output tensor dynamic dimension byte size to be non-negative multiple of 16\n");
  83:       }
  84:     }
  85:   }
  86:   int64_t lda, ldb, ldoutput;
  87:   if (M < 0) {
  88:     // A and output is 2d
  89:     CUDA_KERNEL_ASSERT(offset <= tensor_ShapeA[0] && "expected offset to be less than tensor size\n");
  90:     M = delta;
  91:     lda = a_row_major ? tensor_StrideA[0] : tensor_StrideA[1];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 92-105
```cpp
  92:     ldb = b_row_major ? tensor_StrideB[1] : tensor_StrideB[2];
  93:     ldoutput = tensor_StrideOutput[0];
  94:     A_ptrs[tid] = tid == 0 ? A : A + offs[tid - 1] * tensor_StrideA[0];
  95:     if (scale_A != nullptr) {
  96:       inputA_scale_ptrs[tid] = tid == 0 ? scale_A : scale_A + offs[tid - 1];
  97:       inputB_scale_ptrs[tid] = scale_B + tid * b_scale_stride;
  98:     }
  99:     output_ptrs[tid] = tid == 0 ? output : output + offs[tid - 1] * ldoutput;
 100:     B_ptrs[tid] = B + tid * tensor_StrideB[0];
 101:   } else if (N < 0) {
 102:     CUDA_KERNEL_ASSERT(offset <= tensor_ShapeB[1] && "expected offset to be less than tensor size\n");
 103:     N = delta;
 104:     lda = a_row_major ? tensor_StrideA[1] : tensor_StrideA[2];
 105:     ldb = b_row_major ? tensor_StrideB[0] : tensor_StrideB[1]; // B is transposed
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 106-119
```cpp
 106:     ldoutput = tensor_StrideOutput[0];
 107:     A_ptrs[tid] = A + tid * tensor_StrideA[0];
 108:     output_ptrs[tid] = tid == 0 ? output : output + offs[tid - 1];
 109:     B_ptrs[tid] = tid == 0 ? B : B + offs[tid - 1] * tensor_StrideB[1];
 110:     if (scale_A != nullptr) {
 111:       inputA_scale_ptrs[tid] = scale_A + tid * a_scale_stride;
 112:       inputB_scale_ptrs[tid] = tid == 0 ? scale_B : scale_B + offs[tid - 1];
 113:     }
 114:   } else if (K < 0) {
 115:     CUDA_KERNEL_ASSERT(offset <= tensor_ShapeA[1] && offset <= tensor_ShapeB[0] && "expected offset to be less than tensor size\n");
 116:     // A, B is 2d, output is 3d
 117:     K = delta;
 118:     lda = a_row_major ? tensor_StrideA[0] : tensor_StrideA[1];
 119:     ldb = b_row_major ? tensor_StrideB[0] : tensor_StrideB[1];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 120-133
```cpp
 120:     ldoutput = tensor_StrideOutput[1];
 121:     A_ptrs[tid] = tid == 0 ? A : A + offs[tid - 1] * tensor_StrideA[1];
 122:     B_ptrs[tid] = tid == 0 ? B : B + offs[tid - 1] * tensor_StrideB[0];
 123:     output_ptrs[tid] = output + tid * tensor_StrideOutput[0];
 124:     if (scale_A != nullptr) {
 125:       inputA_scale_ptrs[tid] = scale_A + tid * M;
 126:       inputB_scale_ptrs[tid] = scale_B + tid * N;
 127:     }
 128:   } else {
 129:     // A, B, output are 3D
 130:     lda = a_row_major ? tensor_StrideA[1] : tensor_StrideA[2];
 131:     ldb = b_row_major ? tensor_StrideB[1] : tensor_StrideB[2];
 132:     ldoutput = tensor_StrideOutput[1];
 133:     A_ptrs[tid] = A + tid * tensor_StrideA[0];
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 134-141
```cpp
 134:     B_ptrs[tid] = B + tid * tensor_StrideB[0];
 135:     output_ptrs[tid] = output + tid * tensor_StrideOutput[0];
 136:     if (scale_A != nullptr) {
 137:       inputA_scale_ptrs[tid] = scale_A + tid * a_scale_stride;
 138:       inputB_scale_ptrs[tid] = scale_B + tid * b_scale_stride;
 139:     }
 140:   }
 141:   problem_sizes[tid] = ProblemShape(M, N, K);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 143-150
```cpp
 143:   // make_cute_packed_stride only replaces one of the stride elements with
 144:   // one the provided values in the shape arguments
 145:   // the indices of the src/dst depend on whether A/B are row-major
 146:   // so constructing shape argument with two similar lda values
 147:   // while it looks non-sensical (and it is a nonsensical shape)
 148:   // is fine for these stride construction purposes - the one that will be used
 149:   // for replacement is correct, the other one is ignored, and we don't have to
 150:   // branch on whether A/B are row-major
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 151-156
```cpp
 151:   stride_A[tid] = cutlass::make_cute_packed_stride(StrideA{}, {lda, lda, 1});
 152:   stride_B[tid] = cutlass::make_cute_packed_stride(StrideB{}, {ldb, ldb, 1});
 153:   stride_output[tid] =
 154:       cutlass::make_cute_packed_stride(StrideOutput{}, {M, ldoutput, 1});
 155: }
 156: } // namespace at::cuda::detail
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

## Key Concepts / 关键概念

- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<cutlass/util/packed_stride.hpp>`
- Runtime symbols / 运行时符号:
  - `at::cuda::detail`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
