# SparseCUDATensor.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/sparse/cuda/SparseCUDATensor.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Implements executable backend logic for CUDA sparse tensor kernels, centered on Sparse CUDATensor with emphasis on sparse tensor processing.
- 用途（中文）: 实现可执行的后端逻辑，属于CUDA 稀疏张量内核，核心主题是Sparse CUDATensor，重点关注稀疏张量处理。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-16

```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/AccumulateType.h>
   3: #include <ATen/core/Tensor.h>
   4: #include <ATen/ceil_div.h>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/cuda/CUDAContext.h>
   7: #include <ATen/cuda/ThrustAllocator.h>
   8: #include <ATen/native/sparse/cuda/SparseCUDAApplyUtils.cuh>
   9: #include <ATen/native/cuda/SortingCommon.cuh>
  10: #include <ATen/native/NonSymbolicBC.h>
  11: #include <ATen/native/SparseTensorUtils.h>
  12: #include <c10/macros/Macros.h>
  13: #include <c10/util/accumulate.h>
  14: 
  15: #ifndef AT_PER_OPERATOR_HEADERS
  16: #include <ATen/Functions.h>
```
- L1: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L2: Includes `ATen/AccumulateType.h` for ATen tensor/operator infrastructure. / 引入 `ATen/AccumulateType.h`，为 ATen 的张量/算子基础设施提供支持。
- L3: Includes `ATen/core/Tensor.h` for ATen tensor/operator infrastructure. / 引入 `ATen/core/Tensor.h`，为 ATen 的张量/算子基础设施提供支持。
- L4: Includes `ATen/ceil_div.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ceil_div.h`，为 ATen 的张量/算子基础设施提供支持。
- L5: Includes `ATen/Dispatch.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Dispatch.h`，为 ATen 的张量/算子基础设施提供支持。
- L6: Includes `ATen/cuda/CUDAContext.h` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/CUDAContext.h`，为 ATen 的张量/算子基础设施提供支持。
- L7: Includes `ATen/cuda/ThrustAllocator.h` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/ThrustAllocator.h`，为 ATen 的张量/算子基础设施提供支持。
- L8: Includes `ATen/native/sparse/cuda/SparseCUDAApplyUtils.cuh` for ATen tensor/operator infrastructure. / 引入 `ATen/native/sparse/cuda/SparseCUDAApplyUtils.cuh`，为 ATen 的张量/算子基础设施提供支持。
- L9: Includes `ATen/native/cuda/SortingCommon.cuh` for ATen tensor/operator infrastructure. / 引入 `ATen/native/cuda/SortingCommon.cuh`，为 ATen 的张量/算子基础设施提供支持。
- L10: Includes `ATen/native/NonSymbolicBC.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/NonSymbolicBC.h`，为 ATen 的张量/算子基础设施提供支持。
- L11: Includes `ATen/native/SparseTensorUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/SparseTensorUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L12: Includes `c10/macros/Macros.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/macros/Macros.h`，用于 c10 核心运行时、工具或分发元数据。
- L13: Includes `c10/util/accumulate.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/util/accumulate.h`，用于 c10 核心运行时、工具或分发元数据。
- L15: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L16: Includes `ATen/Functions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Functions.h`，为 ATen 的张量/算子基础设施提供支持。

### Lines 17-32

```cpp
  17: #include <ATen/NativeFunctions.h>
  18: #else
  19: #include <ATen/ops/_coalesce_native.h>
  20: #include <ATen/ops/_sparse_coo_tensor_unsafe_native.h>
  21: #include <ATen/ops/empty.h>
  22: #include <ATen/ops/zeros.h>
  23: #endif
  24: 
  25: #include <thrust/device_ptr.h>
  26: #include <thrust/device_vector.h>
  27: #include <thrust/gather.h>
  28: #include <thrust/generate.h>
  29: #include <thrust/scan.h>
  30: #include <thrust/sequence.h>
  31: #include <thrust/sort.h>
  32: #include <thrust/system/cuda/execution_policy.h>
```
- L17: Includes `ATen/NativeFunctions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/NativeFunctions.h`，为 ATen 的张量/算子基础设施提供支持。
- L18: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L19: Includes `ATen/ops/_coalesce_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_coalesce_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L20: Includes `ATen/ops/_sparse_coo_tensor_unsafe_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_coo_tensor_unsafe_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L21: Includes `ATen/ops/empty.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty.h`，为 ATen 的张量/算子基础设施提供支持。
- L22: Includes `ATen/ops/zeros.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/zeros.h`，为 ATen 的张量/算子基础设施提供支持。
- L23: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L25: Includes `thrust/device_ptr.h` for standard-library or external support. / 引入 `thrust/device_ptr.h`，用于标准库或外部支持。
- L26: Includes `thrust/device_vector.h` for standard-library or external support. / 引入 `thrust/device_vector.h`，用于标准库或外部支持。
- L27: Includes `thrust/gather.h` for standard-library or external support. / 引入 `thrust/gather.h`，用于标准库或外部支持。
- L28: Includes `thrust/generate.h` for standard-library or external support. / 引入 `thrust/generate.h`，用于标准库或外部支持。
- L29: Includes `thrust/scan.h` for standard-library or external support. / 引入 `thrust/scan.h`，用于标准库或外部支持。
- L30: Includes `thrust/sequence.h` for standard-library or external support. / 引入 `thrust/sequence.h`，用于标准库或外部支持。
- L31: Includes `thrust/sort.h` for standard-library or external support. / 引入 `thrust/sort.h`，用于标准库或外部支持。
- L32: Includes `thrust/system/cuda/execution_policy.h` for standard-library or external support. / 引入 `thrust/system/cuda/execution_policy.h`，用于标准库或外部支持。

### Lines 33-48

```cpp
  33: #include <thrust/transform.h>
  34: #include <thrust/unique.h>
  35: #include <thrust/system/cuda/execution_policy.h>
  36: #include <c10/macros/Macros.h>
  37: 
  38: namespace at::native {
  39: 
  40: using namespace at::sparse;
  41: 
  42: SparseTensor _coalesce_sparse_cuda(const SparseTensor& self) {
  43:   int64_t nnz = self._nnz();
  44:   TORCH_INTERNAL_ASSERT(!self.is_coalesced());
  45:   // NOTE: Since `coalesce` is not an in-place operation when `is_coalesced` is false,
  46:   // we should keep the original tensor intact and do coalesce on a copy of the tensor
  47:   if (nnz < 2) {
  48:     SparseTensor dst = self.clone();
```
- L33: Includes `thrust/transform.h` for standard-library or external support. / 引入 `thrust/transform.h`，用于标准库或外部支持。
- L34: Includes `thrust/unique.h` for standard-library or external support. / 引入 `thrust/unique.h`，用于标准库或外部支持。
- L35: Includes `thrust/system/cuda/execution_policy.h` for standard-library or external support. / 引入 `thrust/system/cuda/execution_policy.h`，用于标准库或外部支持。
- L36: Includes `c10/macros/Macros.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/macros/Macros.h`，用于 c10 核心运行时、工具或分发元数据。
- L38: Opens namespace `at::native` to scope the following declarations. / 打开命名空间 `at::native`，为后续声明限定作用域。
- L40: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L42: Defines function `_coalesce_sparse_cuda` and begins its implementation body. / 定义函数 `_coalesce_sparse_cuda`，并开始其实现体。
- L43: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L44: Checks an internal invariant that should always hold inside PyTorch. / 检查 PyTorch 内部应始终成立的不变量。
- L45: Documents the nearby logic: NOTE: Since `coalesce` is not an in-place operation when `is_coalesced` is false, / 说明附近逻辑的作用：NOTE: Since `coalesce` is not an in-place operation when `is_coalesced` is false,
- L46: Documents the nearby logic: we should keep the original tensor intact and do coalesce on a copy of the tensor / 说明附近逻辑的作用：we should keep the original tensor intact and do coalesce on a copy of the tensor
- L47: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L48: Declares function `clone` as part of this file's callable surface. / 声明函数 `clone`，作为本文件可调用接口的一部分。

### Lines 49-64

```cpp
  49:     dst._coalesced_(true);
  50:     return dst;
  51:   }
  52: 
  53:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  54:   at::cuda::ThrustAllocator allocator;
  55:   auto policy = thrust::cuda::par(allocator).on(stream);
  56:   // Replace instances with
  57: 
  58:   // For indices, a simple sort + unique suffices
  59:   // For values, we use a custom kernel for segmented reduction (can't use Thrust due to indirection).
  60: 
  61:   Tensor values = self._values();
  62: 
  63:   int64_t sparse_dim = self.sparse_dim();
  64: 
```
- L49: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L50: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L51: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L53: Declares function `getCurrentCUDAStream` as part of this file's callable surface. / 声明函数 `getCurrentCUDAStream`，作为本文件可调用接口的一部分。
- L54: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L55: Declares function `par` as part of this file's callable surface. / 声明函数 `par`，作为本文件可调用接口的一部分。
- L56: Documents the nearby logic: Replace instances with / 说明附近逻辑的作用：Replace instances with
- L58: Documents the nearby logic: For indices, a simple sort + unique suffices / 说明附近逻辑的作用：For indices, a simple sort + unique suffices
- L59: Documents the nearby logic: For values, we use a custom kernel for segmented reduction (can't use Thrust due to indirection). / 说明附近逻辑的作用：For values, we use a custom kernel for segmented reduction (can't use Thrust due to indirection).
- L61: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L63: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。

### Lines 65-80

```cpp
  65:   // indices will be modified by Thrust, so we have to clone or use new storage
  66:   // here.
  67:   Tensor indices1D = flatten_indices(self._indices(), self.sizes(), true);
  68: 
  69:   Tensor origIndices = at::empty({nnz}, self._indices().options());
  70:   Tensor uniqueOffsets = at::empty({nnz}, self._indices().options());
  71: 
  72:   typedef thrust::device_ptr<int64_t> thrust_ptr;
  73:   thrust_ptr indicesIter(indices1D.data_ptr<int64_t>());
  74:   thrust_ptr origIndicesIter(origIndices.data_ptr<int64_t>());
  75:   thrust_ptr uniqueOffsetsIter(uniqueOffsets.data_ptr<int64_t>());
  76: 
  77: 
  78:   // Fill sortedOrigIndices with sequential indices
  79:   thrust::counting_iterator<int64_t> countIterI(0);
  80:   thrust::counting_iterator<int64_t> countIterO(0);
```
- L65: Documents the nearby logic: indices will be modified by Thrust, so we have to clone or use new storage / 说明附近逻辑的作用：indices will be modified by Thrust, so we have to clone or use new storage
- L66: Documents the nearby logic: here. / 说明附近逻辑的作用：here.
- L67: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L69: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L70: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L72: Creates a typedef alias for repeated use in the surrounding implementation. / 创建 typedef 别名，供周围实现重复使用。
- L73: Declares function `indicesIter` as part of this file's callable surface. / 声明函数 `indicesIter`，作为本文件可调用接口的一部分。
- L74: Declares function `origIndicesIter` as part of this file's callable surface. / 声明函数 `origIndicesIter`，作为本文件可调用接口的一部分。
- L75: Declares function `uniqueOffsetsIter` as part of this file's callable surface. / 声明函数 `uniqueOffsetsIter`，作为本文件可调用接口的一部分。
- L78: Documents the nearby logic: Fill sortedOrigIndices with sequential indices / 说明附近逻辑的作用：Fill sortedOrigIndices with sequential indices
- L79: Declares function `countIterI` as part of this file's callable surface. / 声明函数 `countIterI`，作为本文件可调用接口的一部分。
- L80: Declares function `countIterO` as part of this file's callable surface. / 声明函数 `countIterO`，作为本文件可调用接口的一部分。

### Lines 81-96

```cpp
  81: 
  82:   thrust::copy(policy, countIterI, countIterI + nnz, origIndicesIter);
  83:   thrust::copy(policy, countIterO, countIterO + nnz, uniqueOffsetsIter);
  84: 
  85:   thrust::sort_by_key(policy,
  86:     indicesIter, indicesIter + nnz,
  87:     origIndicesIter, LTOp<int64_t>()
  88:   );
  89: 
  90:   // this forces device-host synchronization!
  91:   auto newEnd = thrust::unique_by_key(policy,
  92:     indicesIter, indicesIter + nnz,
  93:     uniqueOffsetsIter
  94:   );
  95:   int64_t newNnz = newEnd.first - indicesIter;
  96: 
```
- L82: Declares function `copy` as part of this file's callable surface. / 声明函数 `copy`，作为本文件可调用接口的一部分。
- L83: Declares function `copy` as part of this file's callable surface. / 声明函数 `copy`，作为本文件可调用接口的一部分。
- L85: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L86: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L87: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L88: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L90: Documents the nearby logic: this forces device-host synchronization! / 说明附近逻辑的作用：this forces device-host synchronization!
- L91: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L92: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L93: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L94: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L95: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 97-112

```cpp
  97:   indices1D.resize_({1, newNnz});
  98:   auto newValues_size = values.sizes().vec();
  99:   newValues_size[0] = newNnz;
 100:   Tensor newValues = at::empty(newValues_size, values.options());
 101: 
 102:   // If there is no values to copy, save running the kernel.
 103:   if (newValues.numel() > 0) {
 104:     const int SZ = 4;
 105:     values = values.contiguous();
 106:     int64_t stride = c10::multiply_integers(values.sizes().slice(1));
 107:     int warp_size = at::cuda::warp_size();
 108: #ifdef USE_ROCM
 109:     const int64_t BATCHING_SEGMENT = 4096;
 110:     int64_t nsegments = ceil_div(newNnz, (int64_t) SZ);
 111:     int64_t s_batch = ceil_div(nsegments, BATCHING_SEGMENT);
 112:     dim3 grid(s_batch, (s_batch == 1) ? nsegments : BATCHING_SEGMENT, ceil_div(stride, (int64_t) warp_size*SZ));
```
- L97: Declares function `resize_` as part of this file's callable surface. / 声明函数 `resize_`，作为本文件可调用接口的一部分。
- L98: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L99: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L100: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L102: Documents the nearby logic: If there is no values to copy, save running the kernel. / 说明附近逻辑的作用：If there is no values to copy, save running the kernel.
- L103: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L104: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L105: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L106: Declares function `multiply_integers` as part of this file's callable surface. / 声明函数 `multiply_integers`，作为本文件可调用接口的一部分。
- L107: Declares function `warp_size` as part of this file's callable surface. / 声明函数 `warp_size`，作为本文件可调用接口的一部分。
- L108: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L109: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L110: Declares function `ceil_div` as part of this file's callable surface. / 声明函数 `ceil_div`，作为本文件可调用接口的一部分。
- L111: Declares function `ceil_div` as part of this file's callable surface. / 声明函数 `ceil_div`，作为本文件可调用接口的一部分。
- L112: Declares function `grid` as part of this file's callable surface. / 声明函数 `grid`，作为本文件可调用接口的一部分。

### Lines 113-128

```cpp
 113: #else
 114:     dim3 grid(ceil_div(newNnz, (int64_t) SZ), ceil_div(stride, (int64_t) warp_size*SZ));
 115: #endif
 116:     dim3 block(warp_size, SZ);
 117: #ifdef USE_ROCM
 118:     // Must duplicate the whole section otherwise does not compile on Windows
 119:     AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
 120:       at::ScalarType::ComplexHalf, at::ScalarType::Half, at::ScalarType::BFloat16, at::ScalarType::Bool,
 121:       values.scalar_type(), "coalesce_sparse_cuda", [&] {
 122:         using cuda_accscalar_t = acc_type<scalar_t, /* is_cuda */ true>;
 123:         apply::coalesceValuesKernel<scalar_t, cuda_accscalar_t><<<grid, block, 0, stream>>>(
 124:           uniqueOffsets.data_ptr<int64_t>(),
 125:           origIndices.data_ptr<int64_t>(),
 126:           values.data_ptr<scalar_t>(),
 127:           newValues.data_ptr<scalar_t>(),
 128:           nnz,
```
- L113: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L114: Declares function `grid` as part of this file's callable surface. / 声明函数 `grid`，作为本文件可调用接口的一部分。
- L115: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L116: Declares function `block` as part of this file's callable surface. / 声明函数 `block`，作为本文件可调用接口的一部分。
- L117: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L118: Documents the nearby logic: Must duplicate the whole section otherwise does not compile on Windows / 说明附近逻辑的作用：Must duplicate the whole section otherwise does not compile on Windows
- L119: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L120: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L121: Defines function `scalar_type` and begins its implementation body. / 定义函数 `scalar_type`，并开始其实现体。
- L122: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L123: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L124: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L125: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L126: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L127: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L128: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 129-144

```cpp
 129:           newNnz,
 130:           nsegments,
 131:           stride
 132:         );
 133:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 134:       });
 135: #else
 136:     AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
 137:       at::ScalarType::ComplexHalf, at::ScalarType::Half, at::ScalarType::BFloat16, at::ScalarType::Bool,
 138:       values.scalar_type(), "coalesce_sparse_cuda", [&] {
 139:         using cuda_accscalar_t = acc_type<scalar_t, /* is_cuda */ true>;
 140:         apply::coalesceValuesKernel<scalar_t, cuda_accscalar_t><<<grid, block, 0, stream>>>(
 141:           uniqueOffsets.data_ptr<int64_t>(),
 142:           origIndices.data_ptr<int64_t>(),
 143:           values.data_ptr<scalar_t>(),
 144:           newValues.data_ptr<scalar_t>(),
```
- L129: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L130: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L131: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L132: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L133: Declares function `C10_CUDA_KERNEL_LAUNCH_CHECK` as part of this file's callable surface. / 声明函数 `C10_CUDA_KERNEL_LAUNCH_CHECK`，作为本文件可调用接口的一部分。
- L134: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L135: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L136: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L137: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L138: Defines function `scalar_type` and begins its implementation body. / 定义函数 `scalar_type`，并开始其实现体。
- L139: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L140: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L141: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L142: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L143: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L144: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 145-160

```cpp
 145:           nnz,
 146:           newNnz,
 147:           stride
 148:         );
 149:         C10_CUDA_KERNEL_LAUNCH_CHECK();
 150:       });
 151: #endif
 152:   }
 153: 
 154: // this grid-strided version is slower but probably more flexible
 155:   // to different sizes
 156:   // int64_t blockX = min(stride, (int64_t) 512);
 157:   // dim3 block(blockX, 512 / blockX);
 158:   // int64_t grid = min((int64_t) 1024, ceil_div((int64_t) newNnz * stride, (int64_t) block.x * block.y));
 159:   // THCSTensor_coalesceValuesKernel_gridStrided<real, accreal><<<grid, block, 0, stream> >>(
 160:   //   THCIndexTensor_(data)(state, uniqueOffsets),
```
- L145: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L146: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L147: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L148: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L149: Declares function `C10_CUDA_KERNEL_LAUNCH_CHECK` as part of this file's callable surface. / 声明函数 `C10_CUDA_KERNEL_LAUNCH_CHECK`，作为本文件可调用接口的一部分。
- L150: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L151: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L152: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L154: Documents the nearby logic: this grid-strided version is slower but probably more flexible / 说明附近逻辑的作用：this grid-strided version is slower but probably more flexible
- L155: Documents the nearby logic: to different sizes / 说明附近逻辑的作用：to different sizes
- L156: Documents the nearby logic: int64_t blockX = min(stride, (int64_t) 512); / 说明附近逻辑的作用：int64_t blockX = min(stride, (int64_t) 512);
- L157: Documents the nearby logic: dim3 block(blockX, 512 / blockX); / 说明附近逻辑的作用：dim3 block(blockX, 512 / blockX);
- L158: Documents the nearby logic: int64_t grid = min((int64_t) 1024, ceil_div((int64_t) newNnz * stride, (int64_t) block.x * block.y)); / 说明附近逻辑的作用：int64_t grid = min((int64_t) 1024, ceil_div((int64_t) newNnz * stride, (int64_t) block.x * block.y));
- L159: Documents the nearby logic: THCSTensor_coalesceValuesKernel_gridStrided<real, accreal><<<grid, block, 0, stream> >>( / 说明附近逻辑的作用：THCSTensor_coalesceValuesKernel_gridStrided<real, accreal><<<grid, block, 0, stream> >>(
- L160: Documents the nearby logic: THCIndexTensor_(data)(state, uniqueOffsets), / 说明附近逻辑的作用：THCIndexTensor_(data)(state, uniqueOffsets),

### Lines 161-176

```cpp
 161:   //   THCIndexTensor_(data)(state, origIndices),
 162:   //   THCTensor_(data)(state, values),
 163:   //   THCTensor_(data)(state, newValues),
 164:   //   nnz,
 165:   //   newNnz,
 166:   //   stride
 167:   // );
 168:   // C10_CUDA_KERNEL_LAUNCH_CHECK();
 169: 
 170:   ////////////////////////////////////////////////////////////
 171:   // unflatten indices if necessary
 172:   Tensor newIndices;
 173:   if (sparse_dim == 1) {
 174:     newIndices = indices1D;
 175:   } else {
 176:     newIndices = at::empty({sparse_dim, newNnz}, origIndices.options());
```
- L161: Documents the nearby logic: THCIndexTensor_(data)(state, origIndices), / 说明附近逻辑的作用：THCIndexTensor_(data)(state, origIndices),
- L162: Documents the nearby logic: THCTensor_(data)(state, values), / 说明附近逻辑的作用：THCTensor_(data)(state, values),
- L163: Documents the nearby logic: THCTensor_(data)(state, newValues), / 说明附近逻辑的作用：THCTensor_(data)(state, newValues),
- L164: Documents the nearby logic: nnz, / 说明附近逻辑的作用：nnz,
- L165: Documents the nearby logic: newNnz, / 说明附近逻辑的作用：newNnz,
- L166: Documents the nearby logic: stride / 说明附近逻辑的作用：stride
- L167: Documents the nearby logic: ); / 说明附近逻辑的作用：);
- L168: Documents the nearby logic: C10_CUDA_KERNEL_LAUNCH_CHECK(); / 说明附近逻辑的作用：C10_CUDA_KERNEL_LAUNCH_CHECK();
- L170: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L171: Documents the nearby logic: unflatten indices if necessary / 说明附近逻辑的作用：unflatten indices if necessary
- L172: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L173: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L174: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L175: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L176: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。

### Lines 177-192

```cpp
 177:     for (int64_t d = sparse_dim - 1; d >= 0; d--) {
 178:       // NB: Not a select, so I can preserve the outer dimension
 179:       Tensor indicesSlice = newIndices.narrow(0, d, 1);
 180:       indicesSlice.copy_(indices1D);
 181:       indices1D.divide_(self.size(d), "trunc");
 182:       indicesSlice.add_(indices1D, -self.size(d));
 183:     }
 184:   }
 185:   ////////////////////////////////////////////////////////////
 186:   // We can use unsafe sparse tensor constructor because the indices do not
 187:   // need to be revalidated as we do not add or change indices, just remove
 188:   // duplicates.
 189:   SparseTensor dst = ::at::native::_sparse_coo_tensor_unsafe(newIndices, newValues, self.sizes())._coalesced_(true);
 190: 
 191:   AT_CUDA_CHECK(cudaGetLastError());
 192:   return dst;
```
- L177: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L178: Documents the nearby logic: NB: Not a select, so I can preserve the outer dimension / 说明附近逻辑的作用：NB: Not a select, so I can preserve the outer dimension
- L179: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L180: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L181: Declares function `divide_` as part of this file's callable surface. / 声明函数 `divide_`，作为本文件可调用接口的一部分。
- L182: Declares function `add_` as part of this file's callable surface. / 声明函数 `add_`，作为本文件可调用接口的一部分。
- L183: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L184: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L185: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L186: Documents the nearby logic: We can use unsafe sparse tensor constructor because the indices do not / 说明附近逻辑的作用：We can use unsafe sparse tensor constructor because the indices do not
- L187: Documents the nearby logic: need to be revalidated as we do not add or change indices, just remove / 说明附近逻辑的作用：need to be revalidated as we do not add or change indices, just remove
- L188: Documents the nearby logic: duplicates. / 说明附近逻辑的作用：duplicates.
- L189: Declares function `_sparse_coo_tensor_unsafe` as part of this file's callable surface. / 声明函数 `_sparse_coo_tensor_unsafe`，作为本文件可调用接口的一部分。
- L191: Declares function `AT_CUDA_CHECK` as part of this file's callable surface. / 声明函数 `AT_CUDA_CHECK`，作为本文件可调用接口的一部分。
- L192: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。

### Lines 193-195

```cpp
 193: }
 194: 
 195: } // namespace at::native
```
- L193: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L195: Closes namespace `at::native` and returns to the outer scope. / 关闭命名空间 `at::native`，返回外层作用域。

## Key Concepts / 关键概念

- CUDA sparse tensor kernels / CUDA 稀疏张量内核
- Sparse tensor math and layout utilities / 稀疏张量数学与布局工具
- Sparse layout semantics and NNZ traversal / 稀疏布局语义与 NNZ 遍历
- COO index/value representation / COO 索引/数值表示
- Tensor shape, stride, and dtype flow / Tensor 形状、步长与数据类型流转
- Dispatch stubs and backend selection / 分发桩与后端选择
- CUDA execution and specialization / CUDA 执行与特化

## Dependencies / 依赖关系

- `ATen/AccumulateType.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/core/Tensor.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ceil_div.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Dispatch.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/cuda/CUDAContext.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/cuda/ThrustAllocator.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/sparse/cuda/SparseCUDAApplyUtils.cuh` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/cuda/SortingCommon.cuh` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/NonSymbolicBC.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/SparseTensorUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `c10/macros/Macros.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `c10/util/accumulate.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `ATen/Functions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/NativeFunctions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_coalesce_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_coo_tensor_unsafe_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/zeros.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `thrust/device_ptr.h` — standard or external dependency / 标准库或外部依赖
- `thrust/device_vector.h` — standard or external dependency / 标准库或外部依赖
- `thrust/gather.h` — standard or external dependency / 标准库或外部依赖
- `thrust/generate.h` — standard or external dependency / 标准库或外部依赖
- `thrust/scan.h` — standard or external dependency / 标准库或外部依赖
- `thrust/sequence.h` — standard or external dependency / 标准库或外部依赖
- `thrust/sort.h` — standard or external dependency / 标准库或外部依赖
- `thrust/system/cuda/execution_policy.h` — standard or external dependency / 标准库或外部依赖
- `thrust/transform.h` — standard or external dependency / 标准库或外部依赖
- `thrust/unique.h` — standard or external dependency / 标准库或外部依赖
- `thrust/system/cuda/execution_policy.h` — standard or external dependency / 标准库或外部依赖
- `c10/macros/Macros.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- Subsystem tie-in: sparse layouts (COO/CSR/CSC/BSR), index transforms, and NNZ-oriented computation. / 子系统关联：稀疏布局（COO/CSR/CSC/BSR）、索引变换以及面向 NNZ 的计算。
