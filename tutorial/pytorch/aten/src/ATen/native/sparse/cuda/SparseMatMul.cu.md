# SparseMatMul.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/sparse/cuda/SparseMatMul.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Implements executable backend logic for CUDA sparse tensor kernels, centered on Sparse Mat Mul with emphasis on sparse tensor processing.
- 用途（中文）: 实现可执行的后端逻辑，属于CUDA 稀疏张量内核，核心主题是Sparse Mat Mul，重点关注稀疏张量处理。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-30

```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/Config.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/NamedTensorUtils.h>
   6: #include <ATen/Parallel.h>
   7: #include <ATen/SparseTensorImpl.h>
   8: #include <ATen/native/Resize.h>
   9: #include <ATen/native/SparseTensorUtils.h>
  10: #include <cuda_runtime.h>
  11: #include <type_traits>
  12: 
  13: #ifndef AT_PER_OPERATOR_HEADERS
  14: #include <ATen/Functions.h>
  15: #include <ATen/NativeFunctions.h>
  16: #else
  17: #include <ATen/ops/_sparse_sparse_matmul_native.h>
  18: #include <ATen/ops/empty.h>
  19: #include <ATen/ops/empty_like_native.h>
  20: #endif
  21: 
  22: #include <thrust/device_ptr.h>
  23: #include <thrust/for_each.h>
  24: #include <thrust/sequence.h>
  25: 
  26: #include <ATen/cuda/CUDAContext.h>
  27: #include <ATen/cuda/CUDADataType.h>
  28: #include <ATen/cuda/CUDAUtils.h>
  29: #include <ATen/cuda/ThrustAllocator.h>
  30: #include <cusparse.h>
```
- L1: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L2: Includes `ATen/core/Tensor.h` for ATen tensor/operator infrastructure. / 引入 `ATen/core/Tensor.h`，为 ATen 的张量/算子基础设施提供支持。
- L3: Includes `ATen/Config.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Config.h`，为 ATen 的张量/算子基础设施提供支持。
- L4: Includes `ATen/Dispatch.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Dispatch.h`，为 ATen 的张量/算子基础设施提供支持。
- L5: Includes `ATen/NamedTensorUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/NamedTensorUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L6: Includes `ATen/Parallel.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Parallel.h`，为 ATen 的张量/算子基础设施提供支持。
- L7: Includes `ATen/SparseTensorImpl.h` for ATen tensor/operator infrastructure. / 引入 `ATen/SparseTensorImpl.h`，为 ATen 的张量/算子基础设施提供支持。
- L8: Includes `ATen/native/Resize.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/Resize.h`，为 ATen 的张量/算子基础设施提供支持。
- L9: Includes `ATen/native/SparseTensorUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/SparseTensorUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L10: Includes `cuda_runtime.h` for standard-library or external support. / 引入 `cuda_runtime.h`，用于标准库或外部支持。
- L11: Includes `type_traits` for standard-library or external support. / 引入 `type_traits`，用于标准库或外部支持。
- L13: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L14: Includes `ATen/Functions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Functions.h`，为 ATen 的张量/算子基础设施提供支持。
- L15: Includes `ATen/NativeFunctions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/NativeFunctions.h`，为 ATen 的张量/算子基础设施提供支持。
- L16: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L17: Includes `ATen/ops/_sparse_sparse_matmul_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_sparse_matmul_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L18: Includes `ATen/ops/empty.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty.h`，为 ATen 的张量/算子基础设施提供支持。
- L19: Includes `ATen/ops/empty_like_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty_like_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L20: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L22: Includes `thrust/device_ptr.h` for standard-library or external support. / 引入 `thrust/device_ptr.h`，用于标准库或外部支持。
- L23: Includes `thrust/for_each.h` for standard-library or external support. / 引入 `thrust/for_each.h`，用于标准库或外部支持。
- L24: Includes `thrust/sequence.h` for standard-library or external support. / 引入 `thrust/sequence.h`，用于标准库或外部支持。
- L26: Includes `ATen/cuda/CUDAContext.h` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/CUDAContext.h`，为 ATen 的张量/算子基础设施提供支持。
- L27: Includes `ATen/cuda/CUDADataType.h` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/CUDADataType.h`，为 ATen 的张量/算子基础设施提供支持。
- L28: Includes `ATen/cuda/CUDAUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/CUDAUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L29: Includes `ATen/cuda/ThrustAllocator.h` for ATen tensor/operator infrastructure. / 引入 `ATen/cuda/ThrustAllocator.h`，为 ATen 的张量/算子基础设施提供支持。
- L30: Includes `cusparse.h` for standard-library or external support. / 引入 `cusparse.h`，用于标准库或外部支持。

### Lines 31-60

```cpp
  31: #include <ATen/native/sparse/cuda/SparseCUDABlas.h>
  32: #include <c10/cuda/CUDACachingAllocator.h>
  33: 
  34: #include <thrust/device_vector.h>
  35: #include <thrust/host_vector.h>
  36: #include <thrust/iterator/counting_iterator.h>
  37: #include <thrust/functional.h>
  38: #include <thrust/execution_policy.h>
  39: #include <thrust/iterator/discard_iterator.h>
  40: 
  41: 
  42: #include <library_types.h>
  43: 
  44: namespace at::native {
  45: 
  46: namespace {
  47: 
  48: using namespace at::sparse;
  49: 
  50: Tensor _to_csr_int(const Tensor& rowIndices, int64_t dim, int64_t nnz) {
  51:   Tensor csr = at::empty({dim + 1}, CUDA(kInt));
  52:   Tensor rowIndicesInt = at::empty({rowIndices.size(0)}, CUDA(kInt));
  53:   rowIndicesInt.copy_(rowIndices);
  54:   sparse::cuda::Xcoo2csr(
  55:       rowIndicesInt.data_ptr<int32_t>(), nnz, dim, csr.data_ptr<int32_t>());
  56:   return csr;
  57: }
  58: 
  59: 
  60: #pragma push
```
- L31: Includes `ATen/native/sparse/cuda/SparseCUDABlas.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/sparse/cuda/SparseCUDABlas.h`，为 ATen 的张量/算子基础设施提供支持。
- L32: Includes `c10/cuda/CUDACachingAllocator.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/cuda/CUDACachingAllocator.h`，用于 c10 核心运行时、工具或分发元数据。
- L34: Includes `thrust/device_vector.h` for standard-library or external support. / 引入 `thrust/device_vector.h`，用于标准库或外部支持。
- L35: Includes `thrust/host_vector.h` for standard-library or external support. / 引入 `thrust/host_vector.h`，用于标准库或外部支持。
- L36: Includes `thrust/iterator/counting_iterator.h` for standard-library or external support. / 引入 `thrust/iterator/counting_iterator.h`，用于标准库或外部支持。
- L37: Includes `thrust/functional.h` for standard-library or external support. / 引入 `thrust/functional.h`，用于标准库或外部支持。
- L38: Includes `thrust/execution_policy.h` for standard-library or external support. / 引入 `thrust/execution_policy.h`，用于标准库或外部支持。
- L39: Includes `thrust/iterator/discard_iterator.h` for standard-library or external support. / 引入 `thrust/iterator/discard_iterator.h`，用于标准库或外部支持。
- L42: Includes `library_types.h` for standard-library or external support. / 引入 `library_types.h`，用于标准库或外部支持。
- L44: Opens namespace `at::native` to scope the following declarations. / 打开命名空间 `at::native`，为后续声明限定作用域。
- L46: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L48: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L50: Defines function `_to_csr_int` and begins its implementation body. / 定义函数 `_to_csr_int`，并开始其实现体。
- L51: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L52: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L53: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L54: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L55: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L56: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L57: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L60: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 61-90

```cpp
  61: // NVCC complains that confirm_mult_size is not used,
  62: // but it is used in specializations of CusparseMatrixMultiplyOp below
  63: #pragma nv_diag_suppress 177   // Function was declared but never referenced
  64: int confirm_mult_size(const std::vector<int>& mat1_size, const std::vector<int>& mat2_size) {
  65:   TORCH_CHECK(
  66:       mat1_size[1] == mat2_size[0],
  67:       "mat1 and mat2 shapes cannot be multiplied (",
  68:       mat1_size[0],
  69:       "x",
  70:       mat1_size[1],
  71:       " and ",
  72:       mat2_size[0],
  73:       "x",
  74:       mat2_size[1],
  75:       ")");
  76:   return mat1_size[1];
  77: }
  78: #pragma pop
  79: 
  80: void create_general_description_(cusparseMatDescr_t& description_) {
  81:   TORCH_CUDASPARSE_CHECK(cusparseCreateMatDescr(&description_));
  82:   TORCH_CUDASPARSE_CHECK(cusparseSetMatType(description_, CUSPARSE_MATRIX_TYPE_GENERAL));
  83:   TORCH_CUDASPARSE_CHECK(cusparseSetMatIndexBase(description_, CUSPARSE_INDEX_BASE_ZERO));
  84: }
  85: 
  86: // csrMatrixRef is used to have a representation of a raw CSR matrix representation
  87: // coming from `sparse_sparse_matmul_cuda_kernel` function.
  88: // Moreover this implements a RAII guard for a cusparse descriptor
  89: template<class scalar_t>
  90: struct csrMatrixRef {
```
- L61: Documents the nearby logic: NVCC complains that confirm_mult_size is not used, / 说明附近逻辑的作用：NVCC complains that confirm_mult_size is not used,
- L62: Documents the nearby logic: but it is used in specializations of CusparseMatrixMultiplyOp below / 说明附近逻辑的作用：but it is used in specializations of CusparseMatrixMultiplyOp below
- L63: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L64: Defines function `confirm_mult_size` and begins its implementation body. / 定义函数 `confirm_mult_size`，并开始其实现体。
- L65: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L66: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L67: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L68: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L69: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L70: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L71: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L72: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L73: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L74: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L75: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L76: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L77: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L78: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L80: Defines function `create_general_description_` and begins its implementation body. / 定义函数 `create_general_description_`，并开始其实现体。
- L81: Declares function `TORCH_CUDASPARSE_CHECK` as part of this file's callable surface. / 声明函数 `TORCH_CUDASPARSE_CHECK`，作为本文件可调用接口的一部分。
- L82: Declares function `TORCH_CUDASPARSE_CHECK` as part of this file's callable surface. / 声明函数 `TORCH_CUDASPARSE_CHECK`，作为本文件可调用接口的一部分。
- L83: Declares function `TORCH_CUDASPARSE_CHECK` as part of this file's callable surface. / 声明函数 `TORCH_CUDASPARSE_CHECK`，作为本文件可调用接口的一部分。
- L84: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L86: Documents the nearby logic: csrMatrixRef is used to have a representation of a raw CSR matrix representation / 说明附近逻辑的作用：csrMatrixRef is used to have a representation of a raw CSR matrix representation
- L87: Documents the nearby logic: coming from `sparse_sparse_matmul_cuda_kernel` function. / 说明附近逻辑的作用：coming from `sparse_sparse_matmul_cuda_kernel` function.
- L88: Documents the nearby logic: Moreover this implements a RAII guard for a cusparse descriptor / 说明附近逻辑的作用：Moreover this implements a RAII guard for a cusparse descriptor
- L89: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L90: Declares struct `csrMatrixRef` as a reusable type in this module. / 声明struct `csrMatrixRef`，作为本模块中的可复用类型。

### Lines 91-120

```cpp
  91:   int* csr_indices_{nullptr};
  92:   int* csr_pointers_{nullptr};
  93:   scalar_t* csr_values_{nullptr};
  94:   int nnz_{0};
  95:   std::vector<int> size_{};
  96: 
  97:   cusparseSpMatDescr_t description_{0};
  98: 
  99:   csrMatrixRef() = default;
 100: 
 101:   csrMatrixRef(
 102:       int* csr_indices,
 103:       int* csr_pointers,
 104:       scalar_t* csr_values,
 105:       int nnz,
 106:       const std::vector<int>& size)
 107:       : csr_indices_{csr_indices},
 108:         csr_pointers_{csr_pointers},
 109:         csr_values_{csr_values},
 110:         nnz_{nnz},
 111:         size_{size} {
 112:       cudaDataType cuda_data_type = at::cuda::getCudaDataType<scalar_t>();
 113:       TORCH_CUDASPARSE_CHECK(cusparseCreateCsr(
 114:         &description_,
 115:         this->size(0),
 116:         this->size(1),
 117:         this->nnz_,
 118:         this->csr_pointers_,
 119:         this->csr_indices_,
 120:         this->csr_values_,
```
- L91: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L92: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L93: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L94: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L95: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L97: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L99: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L101: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L102: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L103: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L104: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L105: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L106: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L107: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L108: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L109: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L110: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L111: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L112: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L113: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L114: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L115: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L116: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L117: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L118: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L119: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L120: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 121-150

```cpp
 121:         CUSPARSE_INDEX_32I,
 122:         CUSPARSE_INDEX_32I,
 123:         CUSPARSE_INDEX_BASE_ZERO,
 124:         cuda_data_type));
 125:   }
 126: 
 127:   ~csrMatrixRef() {
 128:     cusparseDestroySpMat(description_);
 129:   }
 130: 
 131:   int size(int index) const {
 132:     return size_.at(index);
 133:   }
 134: };
 135: 
 136: // csrOutput is used to represent the output for `CusparseMatrixMultiplyOp`
 137: // Note that `csrOutput` is different from `csrMatrixRef` and the purpose
 138: // of this was to have a materialized  version of a CSR matrix.
 139: // Moreover this implements a RAII guard for a cusparse descriptor
 140: struct csrOutput {
 141:   Tensor csr_indices_{};
 142:   Tensor csr_pointers_{};
 143:   at::Tensor csr_values_{};
 144:   int nnz_{0};
 145:   std::vector<int> size_;
 146: 
 147:   cusparseMatDescr_t description_{0};
 148: 
 149:   csrOutput(const std::vector<int> &size) : size_{size} {
 150:     create_general_description_(description_);
```
- L121: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L122: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L123: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L124: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L125: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L127: Defines function `~csrMatrixRef` and begins its implementation body. / 定义函数 `~csrMatrixRef`，并开始其实现体。
- L128: Declares function `cusparseDestroySpMat` as part of this file's callable surface. / 声明函数 `cusparseDestroySpMat`，作为本文件可调用接口的一部分。
- L129: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L131: Defines function `size` and begins its implementation body. / 定义函数 `size`，并开始其实现体。
- L132: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L133: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L134: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L136: Documents the nearby logic: csrOutput is used to represent the output for `CusparseMatrixMultiplyOp` / 说明附近逻辑的作用：csrOutput is used to represent the output for `CusparseMatrixMultiplyOp`
- L137: Documents the nearby logic: Note that `csrOutput` is different from `csrMatrixRef` and the purpose / 说明附近逻辑的作用：Note that `csrOutput` is different from `csrMatrixRef` and the purpose
- L138: Documents the nearby logic: of this was to have a materialized  version of a CSR matrix. / 说明附近逻辑的作用：of this was to have a materialized  version of a CSR matrix.
- L139: Documents the nearby logic: Moreover this implements a RAII guard for a cusparse descriptor / 说明附近逻辑的作用：Moreover this implements a RAII guard for a cusparse descriptor
- L140: Declares struct `csrOutput` as a reusable type in this module. / 声明struct `csrOutput`，作为本模块中的可复用类型。
- L141: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L142: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L143: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L144: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L145: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L147: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L149: Defines function `csrOutput` and begins its implementation body. / 定义函数 `csrOutput`，并开始其实现体。
- L150: Declares function `create_general_description_` as part of this file's callable surface. / 声明函数 `create_general_description_`，作为本文件可调用接口的一部分。

### Lines 151-180

```cpp
 151:   }
 152: 
 153:   ~csrOutput() {
 154:     cusparseDestroyMatDescr(description_);
 155:   }
 156: 
 157:   csrOutput(const csrOutput&) = delete;
 158:   csrOutput& operator=(const csrOutput&) = delete;
 159:   csrOutput(csrOutput&& rhs) {
 160:     csr_indices_ = std::move(rhs.csr_indices_);
 161:     csr_pointers_ = std::move(rhs.csr_pointers_);
 162:     csr_values_ = std::move(rhs.csr_values_);
 163:     nnz_ = rhs.nnz_;
 164:     size_ = std::move(rhs.size_);
 165:     description_ = rhs.description_;
 166:     rhs.description_ = 0;
 167:   }
 168:   csrOutput& operator=(csrOutput&&) = delete;
 169:   int size(int index) const {
 170:     return size_.at(index);
 171:   }
 172: };
 173: 
 174: // RAII guard helps to support cuSparse 11 API for `A @ B` operation
 175: // This generic template exists because with cuSparse the `scalar_t` type could be a double or float
 176: template <class scalar_t>
 177: struct CusparseMatrixMultiplyOp {
 178: 
 179:   cusparseSpGEMMDescr_t spgemmDesc;
 180: 
```
- L151: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L153: Defines function `~csrOutput` and begins its implementation body. / 定义函数 `~csrOutput`，并开始其实现体。
- L154: Declares function `cusparseDestroyMatDescr` as part of this file's callable surface. / 声明函数 `cusparseDestroyMatDescr`，作为本文件可调用接口的一部分。
- L155: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L157: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L158: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L159: Defines function `csrOutput` and begins its implementation body. / 定义函数 `csrOutput`，并开始其实现体。
- L160: Uses move semantics to transfer ownership or contents efficiently. / 使用移动语义高效地转移所有权或内容。
- L161: Uses move semantics to transfer ownership or contents efficiently. / 使用移动语义高效地转移所有权或内容。
- L162: Uses move semantics to transfer ownership or contents efficiently. / 使用移动语义高效地转移所有权或内容。
- L163: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L164: Uses move semantics to transfer ownership or contents efficiently. / 使用移动语义高效地转移所有权或内容。
- L165: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L166: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L167: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L168: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L169: Defines function `size` and begins its implementation body. / 定义函数 `size`，并开始其实现体。
- L170: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L171: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L172: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L174: Documents the nearby logic: RAII guard helps to support cuSparse 11 API for `A @ B` operation / 说明附近逻辑的作用：RAII guard helps to support cuSparse 11 API for `A @ B` operation
- L175: Documents the nearby logic: This generic template exists because with cuSparse the `scalar_t` type could be a double or float / 说明附近逻辑的作用：This generic template exists because with cuSparse the `scalar_t` type could be a double or float
- L176: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L177: Declares struct `CusparseMatrixMultiplyOp` as a reusable type in this module. / 声明struct `CusparseMatrixMultiplyOp`，作为本模块中的可复用类型。
- L179: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 181-210

```cpp
 181:   CusparseMatrixMultiplyOp() {
 182:     static_assert(
 183:       #if !defined(USE_ROCM)
 184:           std::is_same_v<c10::Half, scalar_t> ||
 185:           std::is_same_v<c10::BFloat16, scalar_t> ||
 186:       #endif
 187:           std::is_same_v<float, scalar_t> ||
 188:           std::is_same_v<double, scalar_t> ||
 189:           std::is_same_v<c10::complex<float>, scalar_t> ||
 190:           std::is_same_v<c10::complex<double>, scalar_t>,
 191:       #if !defined(USE_ROCM)
 192:           "cusparseSpGEMM only supports data type of half, bfloat16, float, double and complex float, double."
 193:       #else
 194:           "cusparseSpGEMM only supports data type of float, double and complex float, double."
 195:       #endif
 196:       );
 197:     // SpGEMM Computation
 198:     TORCH_CUDASPARSE_CHECK(cusparseSpGEMM_createDescr(&spgemmDesc));
 199:   }
 200: 
 201:   ~CusparseMatrixMultiplyOp() {
 202:     // destroy matrix/vector descriptors
 203:     cusparseSpGEMM_destroyDescr(spgemmDesc);
 204:   }
 205: 
 206:   csrOutput operator ()(
 207:       const csrMatrixRef<scalar_t>& A,
 208:       const csrMatrixRef<scalar_t>& B,
 209:       Tensor& output_values,
 210:       Tensor& output_indices) {
```
- L181: Defines function `CusparseMatrixMultiplyOp` and begins its implementation body. / 定义函数 `CusparseMatrixMultiplyOp`，并开始其实现体。
- L182: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L183: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L184: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L185: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L186: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L187: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L188: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L189: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L190: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L191: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L192: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L193: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L194: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L195: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L196: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L197: Documents the nearby logic: SpGEMM Computation / 说明附近逻辑的作用：SpGEMM Computation
- L198: Declares function `TORCH_CUDASPARSE_CHECK` as part of this file's callable surface. / 声明函数 `TORCH_CUDASPARSE_CHECK`，作为本文件可调用接口的一部分。
- L199: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L201: Defines function `~CusparseMatrixMultiplyOp` and begins its implementation body. / 定义函数 `~CusparseMatrixMultiplyOp`，并开始其实现体。
- L202: Documents the nearby logic: destroy matrix/vector descriptors / 说明附近逻辑的作用：destroy matrix/vector descriptors
- L203: Declares function `cusparseSpGEMM_destroyDescr` as part of this file's callable surface. / 声明函数 `cusparseSpGEMM_destroyDescr`，作为本文件可调用接口的一部分。
- L204: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L206: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L207: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L208: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L209: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L210: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。

### Lines 211-240

```cpp
 211:     const int A_num_rows = A.size(0);
 212: 
 213:     const int B_num_cols = B.size(1);
 214: 
 215:     csrOutput out({A.size(0), B.size(1)});
 216: 
 217:     out.csr_pointers_ = at::empty({out.size(0) + 1}, output_indices.options().dtype(kInt));
 218: 
 219:     int* dC_csrOffsets = out.csr_pointers_.data_ptr<int>();
 220:     int* dC_columns = nullptr;
 221:     scalar_t* dC_values = nullptr;
 222: 
 223:     scalar_t alpha = 1.0f;
 224:     scalar_t beta = 0.0f;
 225:     cusparseOperation_t opA = CUSPARSE_OPERATION_NON_TRANSPOSE;
 226:     cusparseOperation_t opB = CUSPARSE_OPERATION_NON_TRANSPOSE;
 227: 
 228:     csrMatrixRef<scalar_t> C(
 229:       dC_columns,
 230:       dC_csrOffsets,
 231:       dC_values,
 232:       /*nnz*/0,
 233:       {A_num_rows, B_num_cols}
 234:     );
 235: 
 236:     //--------------------------------------------------------------------------
 237:     // CUSPARSE APIs
 238:     cusparseHandle_t handle = at::cuda::getCurrentCUDASparseHandle();
 239:     void *dBuffer1 = NULL, *dBuffer2 = NULL;
 240:     size_t bufferSize1 = 0, bufferSize2 = 0;
```
- L211: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L213: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L215: Declares function `out` as part of this file's callable surface. / 声明函数 `out`，作为本文件可调用接口的一部分。
- L217: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L219: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L220: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L221: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L223: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L224: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L225: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L226: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L228: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L229: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L230: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L231: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L232: Documents the nearby logic: nnz*/0, / 说明附近逻辑的作用：nnz*/0,
- L233: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L234: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L236: Documents the nearby logic: -------------------------------------------------------------------------- / 说明附近逻辑的作用：--------------------------------------------------------------------------
- L237: Documents the nearby logic: CUSPARSE APIs / 说明附近逻辑的作用：CUSPARSE APIs
- L238: Declares function `getCurrentCUDASparseHandle` as part of this file's callable surface. / 声明函数 `getCurrentCUDASparseHandle`，作为本文件可调用接口的一部分。
- L239: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L240: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 241-270

```cpp
 241: 
 242:     cusparseSpMatDescr_t matA = A.description_;
 243:     cusparseSpMatDescr_t matB = B.description_;
 244:     cusparseSpMatDescr_t matC = C.description_;
 245:     //--------------------------------------------------------------------------
 246: 
 247:     cudaDataType computeType = at::cuda::getCudaDataType<scalar_t>();
 248: 
 249:     // If a specific GPU model does not provide native support for a given data type,
 250:     // the routine returns CUSPARSE_STATUS_ARCH_MISMATCH error
 251:     #if defined(USE_ROCM)
 252:     TORCH_CHECK(!(computeType == CUDA_R_16F || computeType == CUDA_R_16BF),
 253:         "sparse_mm: Float16 and BFloat16 are not supported on ROCm");
 254:     #else // defined(USE_ROCM)
 255:     cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
 256:     TORCH_CHECK(prop->major >= 5 && !((10*prop->major + prop->minor) < 53 && computeType == CUDA_R_16F),
 257:         "sparse_mm: CUDA Float16 requires compute capability >= 53 (current: ", prop->major, prop->minor, ")");
 258:     TORCH_CHECK(!(prop->major < 8 && computeType == CUDA_R_16BF),
 259:         "sparse_mm: CUDA BFloat16 requires compute capability >= 80 (current: ", prop->major, prop->minor, ")");
 260:     #endif // defined(USE_ROCM)
 261: 
 262:     // ask bufferSize1 bytes for external memory
 263:     TORCH_CUDASPARSE_CHECK(cusparseSpGEMM_workEstimation(
 264:         handle,
 265:         opA,
 266:         opB,
 267:         &alpha,
 268:         matA,
 269:         matB,
 270:         &beta,
```
- L242: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L243: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L244: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L245: Documents the nearby logic: -------------------------------------------------------------------------- / 说明附近逻辑的作用：--------------------------------------------------------------------------
- L247: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L249: Documents the nearby logic: If a specific GPU model does not provide native support for a given data type, / 说明附近逻辑的作用：If a specific GPU model does not provide native support for a given data type,
- L250: Documents the nearby logic: the routine returns CUSPARSE_STATUS_ARCH_MISMATCH error / 说明附近逻辑的作用：the routine returns CUSPARSE_STATUS_ARCH_MISMATCH error
- L251: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L252: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L253: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L254: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L255: Declares function `getCurrentDeviceProperties` as part of this file's callable surface. / 声明函数 `getCurrentDeviceProperties`，作为本文件可调用接口的一部分。
- L256: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L257: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L258: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L259: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L260: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L262: Documents the nearby logic: ask bufferSize1 bytes for external memory / 说明附近逻辑的作用：ask bufferSize1 bytes for external memory
- L263: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L264: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L265: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L266: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L267: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L268: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L269: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L270: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 271-300

```cpp
 271:         matC,
 272:         computeType,
 273:         CUSPARSE_SPGEMM_DEFAULT,
 274:         spgemmDesc,
 275:         &bufferSize1,
 276:         NULL));
 277: 
 278:     auto& allocator = *::c10::cuda::CUDACachingAllocator::get();
 279: 
 280:     at::DataPtr dataPtr1 = allocator.allocate(bufferSize1);
 281:     dBuffer1 = dataPtr1.get();
 282:     // inspect the matrices A and B to understand the memory requirement for
 283:     // the next step
 284:     TORCH_CUDASPARSE_CHECK(cusparseSpGEMM_workEstimation(
 285:         handle,
 286:         opA,
 287:         opB,
 288:         &alpha,
 289:         matA,
 290:         matB,
 291:         &beta,
 292:         matC,
 293:         computeType,
 294:         CUSPARSE_SPGEMM_DEFAULT,
 295:         spgemmDesc,
 296:         &bufferSize1,
 297:         dBuffer1));
 298: 
 299:     // ask bufferSize2 bytes for external memory
 300:     TORCH_CUDASPARSE_CHECK(cusparseSpGEMM_compute(
```
- L271: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L272: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L273: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L274: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L275: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L276: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L278: Declares function `get` as part of this file's callable surface. / 声明函数 `get`，作为本文件可调用接口的一部分。
- L280: Declares function `allocate` as part of this file's callable surface. / 声明函数 `allocate`，作为本文件可调用接口的一部分。
- L281: Declares function `get` as part of this file's callable surface. / 声明函数 `get`，作为本文件可调用接口的一部分。
- L282: Documents the nearby logic: inspect the matrices A and B to understand the memory requirement for / 说明附近逻辑的作用：inspect the matrices A and B to understand the memory requirement for
- L283: Documents the nearby logic: the next step / 说明附近逻辑的作用：the next step
- L284: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L285: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L286: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L287: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L288: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L289: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L290: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L291: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L292: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L293: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L294: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L295: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L296: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L297: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L299: Documents the nearby logic: ask bufferSize2 bytes for external memory / 说明附近逻辑的作用：ask bufferSize2 bytes for external memory
- L300: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 301-330

```cpp
 301:         handle,
 302:         opA,
 303:         opB,
 304:         &alpha,
 305:         matA,
 306:         matB,
 307:         &beta,
 308:         matC,
 309:         computeType,
 310:         CUSPARSE_SPGEMM_DEFAULT,
 311:         spgemmDesc,
 312:         &bufferSize2,
 313:         NULL));
 314: 
 315:     at::DataPtr dataPtr2 = allocator.allocate(bufferSize2);
 316:     dBuffer2 = dataPtr2.get();
 317: 
 318:     // compute the intermediate product of A * B
 319:     TORCH_CUDASPARSE_CHECK(cusparseSpGEMM_compute(
 320:         handle,
 321:         opA,
 322:         opB,
 323:         &alpha,
 324:         matA,
 325:         matB,
 326:         &beta,
 327:         matC,
 328:         computeType,
 329:         CUSPARSE_SPGEMM_DEFAULT,
 330:         spgemmDesc,
```
- L301: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L302: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L303: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L304: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L305: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L306: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L307: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L308: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L309: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L310: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L311: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L312: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L313: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L315: Declares function `allocate` as part of this file's callable surface. / 声明函数 `allocate`，作为本文件可调用接口的一部分。
- L316: Declares function `get` as part of this file's callable surface. / 声明函数 `get`，作为本文件可调用接口的一部分。
- L318: Documents the nearby logic: compute the intermediate product of A * B / 说明附近逻辑的作用：compute the intermediate product of A * B
- L319: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L320: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L321: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L322: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L323: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L324: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L325: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L326: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L327: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L328: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L329: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L330: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 331-360

```cpp
 331:         &bufferSize2,
 332:         dBuffer2));
 333:     // get matrix C non-zero entries C_num_nnz1
 334:     int64_t C_num_rows1, C_num_cols1, C_num_nnz1;
 335:     TORCH_CUDASPARSE_CHECK(
 336:         cusparseSpMatGetSize(matC, &C_num_rows1, &C_num_cols1, &C_num_nnz1));
 337:     // allocate matrix C
 338:     // allocate C offsets
 339:     out.nnz_ = C_num_nnz1;
 340: 
 341:     out.csr_indices_ = at::empty({out.nnz_}, output_indices.options().dtype(kInt));
 342:     out.csr_values_ = at::empty({out.nnz_}, output_values.options());
 343:     dC_columns = out.csr_indices_.data_ptr<int>();
 344:     dC_values = out.csr_values_.data_ptr<scalar_t>();
 345: 
 346:     // update matC with the new pointers
 347:     TORCH_CUDASPARSE_CHECK(
 348:         cusparseCsrSetPointers(matC, dC_csrOffsets, dC_columns, dC_values));
 349: 
 350:     // copy the final products to the matrix C
 351:     TORCH_CUDASPARSE_CHECK(cusparseSpGEMM_copy(
 352:         handle,
 353:         opA,
 354:         opB,
 355:         &alpha,
 356:         matA,
 357:         matB,
 358:         &beta,
 359:         matC,
 360:         computeType,
```
- L331: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L332: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L333: Documents the nearby logic: get matrix C non-zero entries C_num_nnz1 / 说明附近逻辑的作用：get matrix C non-zero entries C_num_nnz1
- L334: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L335: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L336: Declares function `cusparseSpMatGetSize` as part of this file's callable surface. / 声明函数 `cusparseSpMatGetSize`，作为本文件可调用接口的一部分。
- L337: Documents the nearby logic: allocate matrix C / 说明附近逻辑的作用：allocate matrix C
- L338: Documents the nearby logic: allocate C offsets / 说明附近逻辑的作用：allocate C offsets
- L339: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L341: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L342: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L343: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L344: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L346: Documents the nearby logic: update matC with the new pointers / 说明附近逻辑的作用：update matC with the new pointers
- L347: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L348: Declares function `cusparseCsrSetPointers` as part of this file's callable surface. / 声明函数 `cusparseCsrSetPointers`，作为本文件可调用接口的一部分。
- L350: Documents the nearby logic: copy the final products to the matrix C / 说明附近逻辑的作用：copy the final products to the matrix C
- L351: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L352: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L353: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L354: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L355: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L356: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L357: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L358: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L359: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L360: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 361-390

```cpp
 361:         CUSPARSE_SPGEMM_DEFAULT,
 362:         spgemmDesc));
 363:     return out;
 364:   }
 365: };
 366: 
 367: 
 368: template struct CusparseMatrixMultiplyOp<float>;
 369: 
 370: template struct CusparseMatrixMultiplyOp<double>;
 371: 
 372: template <typename scalar_t>
 373: void sparse_sparse_matmul_cuda_kernel(
 374:     Tensor& result,
 375:     const Tensor& mat1,
 376:     const Tensor& mat2) {
 377: 
 378:   static_assert(
 379:     std::is_same_v<c10::Half, scalar_t> ||
 380:         std::is_same_v<c10::BFloat16, scalar_t> ||
 381:         std::is_same_v<float, scalar_t> ||
 382:         std::is_same_v<double, scalar_t> ||
 383:         std::is_same_v<c10::complex<float>, scalar_t> ||
 384:         std::is_same_v<c10::complex<double>, scalar_t>,
 385:     "sparse_sparse_matmul_cuda_kernel only supports data type of half, bfloat16, float, double and complex float, double.");
 386: 
 387:   Tensor mat1_indices_ = mat1._indices().contiguous();
 388:   Tensor mat1_values = mat1._values().contiguous();
 389: 
 390:   Tensor mat1_row_indices = mat1_indices_.select(0, 0);
```
- L361: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L362: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L363: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L364: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L365: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L368: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L370: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L372: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L373: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L374: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L375: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L376: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L378: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L379: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L380: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L381: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L382: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L383: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L384: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L385: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L387: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L388: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L390: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。

### Lines 391-420

```cpp
 391:   Tensor mat1_col_indices = mat1_indices_.select(0, 1);
 392: 
 393:   Tensor mat1_indptr = _to_csr_int(mat1_row_indices, mat1.size(0), mat1._nnz());
 394: 
 395:   Tensor mat1_indices = at::empty(
 396:       {mat1_col_indices.size(0)}, mat1_col_indices.options().dtype(kInt));
 397: 
 398:   mat1_indices.copy_(mat1_col_indices);
 399: 
 400:   Tensor mat2_indices_ = mat2._indices().contiguous();
 401:   Tensor mat2_values = mat2._values().contiguous();
 402:   Tensor mat2_row_indices = mat2_indices_.select(0, 0);
 403:   Tensor mat2_col_indices = mat2_indices_.select(0, 1);
 404: 
 405:   Tensor mat2_indptr = _to_csr_int(mat2_row_indices, mat2.size(0), mat2._nnz());
 406:   Tensor mat2_indices = at::empty({mat2_col_indices.size(0)}, mat2_col_indices.options().dtype(kInt));
 407:   mat2_indices.copy_(mat2_col_indices);
 408: 
 409:   auto m = mat1.size(0);
 410:   auto k1 = mat1.size(1);
 411: 
 412:   auto k2 = mat2.size(0);
 413:   auto n = mat2.size(1);
 414:   TORCH_CHECK((m <= INT_MAX) && (n <= INT_MAX) && (k1 <= INT_MAX),
 415:     "At the moment, cusparseDcsrgemm2 only supports m, n, k, nnz with the bound [val] <= ", INT_MAX, ".",
 416:     "If you need this, please file an issue on GitHub."
 417:   );
 418:   auto output_indices = result._indices();
 419:   auto output_values = result._values();
 420: 
```
- L391: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L393: Declares function `_to_csr_int` as part of this file's callable surface. / 声明函数 `_to_csr_int`，作为本文件可调用接口的一部分。
- L395: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L396: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L398: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L400: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L401: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L402: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L403: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L405: Declares function `_to_csr_int` as part of this file's callable surface. / 声明函数 `_to_csr_int`，作为本文件可调用接口的一部分。
- L406: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L407: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L409: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L410: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L412: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L413: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L414: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L415: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L416: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L417: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L418: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L419: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。

### Lines 421-450

```cpp
 421:   if ((k1 == 0 && k2 == 0) || (n == 0 && m == 0)) {
 422:     output_indices.zero_();
 423:     output_values.zero_();
 424:     return;
 425:   }
 426: 
 427:   csrMatrixRef<scalar_t> csr_mat1(
 428:       mat1_indices.data_ptr<int>(),
 429:       mat1_indptr.data_ptr<int>(),
 430:       mat1_values.data_ptr<scalar_t>(),
 431:       (int)mat1._nnz(),
 432:       {(int)mat1.size(0), (int)mat1.size(1)});
 433: 
 434:   csrMatrixRef<scalar_t> csr_mat2(
 435:       mat2_indices.data_ptr<int>(),
 436:       mat2_indptr.data_ptr<int>(),
 437:       mat2_values.data_ptr<scalar_t>(),
 438:       (int)mat2._nnz(),
 439:       {(int)mat2.size(0), (int)mat2.size(1)});
 440: 
 441:   // Sparse matrix multiplication
 442:   CusparseMatrixMultiplyOp<scalar_t> op;
 443:   csrOutput csr_output = op(csr_mat1, csr_mat2, output_values, output_indices);
 444:   auto nnz = csr_output.nnz_;
 445: 
 446:   output_values.set_(csr_output.csr_values_);
 447:   output_indices.resize_({2, nnz});
 448:   auto output_indices_accessor = output_indices.packed_accessor64<int64_t, 2>();
 449: 
 450:   auto csr_output_pointers_accessor =
```
- L421: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L422: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L423: Declares function `zero_` as part of this file's callable surface. / 声明函数 `zero_`，作为本文件可调用接口的一部分。
- L424: Returns from the current function without producing a value. / 从当前函数直接返回，不产生返回值。
- L425: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L427: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L428: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L429: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L430: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L431: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L432: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L434: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L435: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L436: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L437: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L438: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L439: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L441: Documents the nearby logic: Sparse matrix multiplication / 说明附近逻辑的作用：Sparse matrix multiplication
- L442: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L443: Declares function `op` as part of this file's callable surface. / 声明函数 `op`，作为本文件可调用接口的一部分。
- L444: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L446: Declares function `set_` as part of this file's callable surface. / 声明函数 `set_`，作为本文件可调用接口的一部分。
- L447: Declares function `resize_` as part of this file's callable surface. / 声明函数 `resize_`，作为本文件可调用接口的一部分。
- L448: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L450: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。

### Lines 451-480

```cpp
 451:       csr_output.csr_pointers_.packed_accessor64<int, 1>();
 452: 
 453:   auto csr_output_ind_accessor =
 454:       csr_output.csr_indices_.packed_accessor64<int, 1>();
 455: 
 456:   auto major_dim = result.size(0);
 457:   cudaStream_t stream = at::cuda::getCurrentCUDAStream();
 458:   at::cuda::ThrustAllocator allocator;
 459:   auto policy = thrust::cuda::par(allocator).on(stream);
 460: 
 461:   // Filling the COO row indices
 462:   thrust::for_each(
 463:       policy,
 464:       thrust::make_counting_iterator(int64_t(0)),
 465:       thrust::make_counting_iterator(int64_t(major_dim)),
 466:       [output_indices_accessor,
 467:        csr_output_pointers_accessor,
 468:        major_dim,
 469:        nnz] __device__(int64_t i) {
 470:         auto Ap = csr_output_pointers_accessor.data();
 471:         int64_t* indices_row = output_indices_accessor[0].data();
 472: 
 473:         for (int jj = Ap[i];  jj < Ap[i + 1]; jj++) {
 474:           indices_row[jj] = i;
 475:         }
 476:       });
 477: 
 478:   // Filling the COO column indices
 479:   thrust::for_each(
 480:     policy,
```
- L451: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L453: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L454: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L456: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L457: Declares function `getCurrentCUDAStream` as part of this file's callable surface. / 声明函数 `getCurrentCUDAStream`，作为本文件可调用接口的一部分。
- L458: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L459: Declares function `par` as part of this file's callable surface. / 声明函数 `par`，作为本文件可调用接口的一部分。
- L461: Documents the nearby logic: Filling the COO row indices / 说明附近逻辑的作用：Filling the COO row indices
- L462: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L463: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L464: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L465: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L466: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L467: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L468: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L469: Defines function `__device__` and begins its implementation body. / 定义函数 `__device__`，并开始其实现体。
- L470: Declares function `data` as part of this file's callable surface. / 声明函数 `data`，作为本文件可调用接口的一部分。
- L471: Declares function `data` as part of this file's callable surface. / 声明函数 `data`，作为本文件可调用接口的一部分。
- L473: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L474: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L475: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L476: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L478: Documents the nearby logic: Filling the COO column indices / 说明附近逻辑的作用：Filling the COO column indices
- L479: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L480: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 481-510

```cpp
 481:     thrust::make_counting_iterator(int64_t(0)),
 482:     thrust::make_counting_iterator(int64_t(csr_output.nnz_)),
 483:     [output_indices_accessor,
 484:       csr_output_pointers_accessor,
 485:       csr_output_ind_accessor,
 486:       major_dim,
 487:       nnz] __device__(int64_t i) {
 488:       int64_t* indices_col = output_indices_accessor[1].data();
 489:       indices_col[i] = csr_output_ind_accessor[i];
 490:     });
 491: }
 492: 
 493: } // end anonymous namespace
 494: 
 495: Tensor sparse_sparse_matmul_cuda(const Tensor& mat1_, const Tensor& mat2_) {
 496:   TORCH_INTERNAL_ASSERT(mat1_.is_sparse());
 497:   TORCH_INTERNAL_ASSERT(mat2_.is_sparse());
 498:   TORCH_CHECK(mat1_.dim() == 2);
 499:   TORCH_CHECK(mat2_.dim() == 2);
 500:   TORCH_CHECK(mat1_.dense_dim() == 0, "sparse_mm: scalar values expected, mat1 got ", mat1_.dense_dim(), "D values");
 501:   TORCH_CHECK(mat2_.dense_dim() == 0, "sparse_mm: scalar values expected, mat2 got ", mat2_.dense_dim(), "D values");
 502: 
 503:   TORCH_CHECK(
 504:       mat1_.size(1) == mat2_.size(0), "mat1 and mat2 shapes cannot be multiplied (",
 505:       mat1_.size(0), "x", mat1_.size(1), " and ", mat2_.size(0), "x", mat2_.size(1), ")");
 506: 
 507:   TORCH_CHECK(mat1_.scalar_type() == mat2_.scalar_type(),
 508:            "mat1 dtype ", mat1_.scalar_type(), " does not match mat2 dtype ", mat2_.scalar_type());
 509: 
 510:   auto output = at::native::empty_like(mat1_);
```
- L481: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L482: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L483: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L484: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L485: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L486: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L487: Defines function `__device__` and begins its implementation body. / 定义函数 `__device__`，并开始其实现体。
- L488: Declares function `data` as part of this file's callable surface. / 声明函数 `data`，作为本文件可调用接口的一部分。
- L489: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L490: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L491: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L493: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L495: Defines function `sparse_sparse_matmul_cuda` and begins its implementation body. / 定义函数 `sparse_sparse_matmul_cuda`，并开始其实现体。
- L496: Checks an internal invariant that should always hold inside PyTorch. / 检查 PyTorch 内部应始终成立的不变量。
- L497: Checks an internal invariant that should always hold inside PyTorch. / 检查 PyTorch 内部应始终成立的不变量。
- L498: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L499: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L500: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L501: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L503: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L504: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L505: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L507: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L508: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L510: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。

### Lines 511-526

```cpp
 511:   output.sparse_resize_and_clear_({mat1_.size(0), mat2_.size(1)}, mat1_.sparse_dim(), 0);
 512: 
 513: #if !defined(USE_ROCM)
 514:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2(kHalf, kBFloat16, mat1_.scalar_type(), "sparse_matmul", [&] {
 515:       sparse_sparse_matmul_cuda_kernel<scalar_t>(output, mat1_.coalesce(), mat2_.coalesce());
 516:   });
 517: #else
 518:   // ROCm does not support half and bfloat16 types for sparse_matmul
 519:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(mat1_.scalar_type(), "sparse_matmul", [&] {
 520:       sparse_sparse_matmul_cuda_kernel<scalar_t>(output, mat1_.coalesce(), mat2_.coalesce());
 521:   });
 522: #endif
 523:   return output;
 524: }
 525: 
 526: } // namespace at::native
```
- L511: Declares function `sparse_resize_and_clear_` as part of this file's callable surface. / 声明函数 `sparse_resize_and_clear_`，作为本文件可调用接口的一部分。
- L513: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L514: Defines function `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2` and begins its implementation body. / 定义函数 `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES_AND2`，并开始其实现体。
- L515: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L516: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L517: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L518: Documents the nearby logic: ROCm does not support half and bfloat16 types for sparse_matmul / 说明附近逻辑的作用：ROCm does not support half and bfloat16 types for sparse_matmul
- L519: Defines function `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES` and begins its implementation body. / 定义函数 `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES`，并开始其实现体。
- L520: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L521: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L522: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L523: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L524: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L526: Closes namespace `at::native` and returns to the outer scope. / 关闭命名空间 `at::native`，返回外层作用域。

## Key Concepts / 关键概念

- CUDA sparse tensor kernels / CUDA 稀疏张量内核
- Sparse tensor math and layout utilities / 稀疏张量数学与布局工具
- Sparse layout semantics and NNZ traversal / 稀疏布局语义与 NNZ 遍历
- CSR compressed sparse representation / CSR 压缩稀疏表示
- COO index/value representation / COO 索引/数值表示
- Tensor shape, stride, and dtype flow / Tensor 形状、步长与数据类型流转
- Dispatch stubs and backend selection / 分发桩与后端选择
- CUDA execution and specialization / CUDA 执行与特化

## Dependencies / 依赖关系

- `ATen/core/Tensor.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Config.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Dispatch.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/NamedTensorUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Parallel.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/SparseTensorImpl.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/Resize.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/SparseTensorUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `cuda_runtime.h` — standard or external dependency / 标准库或外部依赖
- `type_traits` — standard or external dependency / 标准库或外部依赖
- `ATen/Functions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/NativeFunctions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_sparse_matmul_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty_like_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `thrust/device_ptr.h` — standard or external dependency / 标准库或外部依赖
- `thrust/for_each.h` — standard or external dependency / 标准库或外部依赖
- `thrust/sequence.h` — standard or external dependency / 标准库或外部依赖
- `ATen/cuda/CUDAContext.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/cuda/CUDADataType.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/cuda/CUDAUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/cuda/ThrustAllocator.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `cusparse.h` — standard or external dependency / 标准库或外部依赖
- `ATen/native/sparse/cuda/SparseCUDABlas.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `c10/cuda/CUDACachingAllocator.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `thrust/device_vector.h` — standard or external dependency / 标准库或外部依赖
- `thrust/host_vector.h` — standard or external dependency / 标准库或外部依赖
- `thrust/iterator/counting_iterator.h` — standard or external dependency / 标准库或外部依赖
- `thrust/functional.h` — standard or external dependency / 标准库或外部依赖
- `thrust/execution_policy.h` — standard or external dependency / 标准库或外部依赖
- Subsystem tie-in: sparse layouts (COO/CSR/CSC/BSR), index transforms, and NNZ-oriented computation. / 子系统关联：稀疏布局（COO/CSR/CSC/BSR）、索引变换以及面向 NNZ 的计算。
