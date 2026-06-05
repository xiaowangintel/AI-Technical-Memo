# SparseMPSTensorMath.mm — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/sparse/mps/SparseMPSTensorMath.mm`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Implements executable backend logic for MPS sparse backend support, centered on Sparse MPSTensor Math with emphasis on sparse tensor processing.
- 用途（中文）: 实现可执行的后端逻辑，属于MPS 稀疏后端支持，核心主题是Sparse MPSTensor Math，重点关注稀疏张量处理。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-40

```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/native/SparseTensorUtils.h>
   3: #include <ATen/SparseCsrTensorUtils.h>
   4: #include <ATen/ExpandUtils.h>
   5: #include <ATen/WrapDimUtilsMulti.h>
   6: #include <ATen/native/mps/OperationUtils.h>
   7: #include <ATen/native/sparse/SparseStubs.h>
   8: #include <ATen/native/sparse/SparseBinaryOpIntersectionCommon.h>
   9: 
  10: #ifndef AT_PER_OPERATOR_HEADERS
  11: #include <ATen/Functions.h>
  12: #include <ATen/NativeFunctions.h>
  13: #else
  14: #include <ATen/ops/_coalesce_native.h>
  15: #include <ATen/ops/repeat_interleave_native.h>
  16: #include <ATen/ops/cumsum.h>
  17: #include <ATen/ops/_sparse_sparse_matmul_native.h>
  18: #include <ATen/ops/_sparse_coo_tensor_unsafe.h>
  19: #include <ATen/ops/_sparse_coo_tensor_unsafe_native.h>
  20: #include <ATen/ops/cat.h>
  21: #include <ATen/ops/softmax_native.h>
  22: #include <ATen/ops/log_softmax.h>
  23: #include <ATen/ops/_sparse_log_softmax_native.h>
  24: #include <ATen/ops/_sparse_softmax_native.h>
  25: #include <ATen/ops/_sparse_softmax_backward_data_native.h>
  26: #include <ATen/ops/_sparse_log_softmax_backward_data_native.h>
  27: #include <ATen/ops/add_native.h>
  28: #include <ATen/ops/mul_native.h>
  29: #include <ATen/ops/empty_native.h>
  30: #include <ATen/ops/zeros_native.h>
  31: #include <ATen/ops/index_select_native.h>
  32: #include <ATen/ops/remainder_native.h>
  33: #include <ATen/ops/ones_like.h>
  34: #include <ATen/ops/argsort.h>
  35: #include <ATen/ops/searchsorted_native.h>
  36: #include <ATen/ops/_sparse_sum_backward_native.h>
  37: #include <ATen/ops/result_type.h>
  38: #include <ATen/ops/bmm_native.h>
  39: #include <ATen/ops/addmm_native.h>
  40: #include <ATen/ops/copy_sparse_to_sparse.h>
```
- L1: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L2: Includes `ATen/native/SparseTensorUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/SparseTensorUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L3: Includes `ATen/SparseCsrTensorUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/SparseCsrTensorUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L4: Includes `ATen/ExpandUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ExpandUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L5: Includes `ATen/WrapDimUtilsMulti.h` for ATen tensor/operator infrastructure. / 引入 `ATen/WrapDimUtilsMulti.h`，为 ATen 的张量/算子基础设施提供支持。
- L6: Includes `ATen/native/mps/OperationUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/mps/OperationUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L7: Includes `ATen/native/sparse/SparseStubs.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/sparse/SparseStubs.h`，为 ATen 的张量/算子基础设施提供支持。
- L8: Includes `ATen/native/sparse/SparseBinaryOpIntersectionCommon.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/sparse/SparseBinaryOpIntersectionCommon.h`，为 ATen 的张量/算子基础设施提供支持。
- L10: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L11: Includes `ATen/Functions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Functions.h`，为 ATen 的张量/算子基础设施提供支持。
- L12: Includes `ATen/NativeFunctions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/NativeFunctions.h`，为 ATen 的张量/算子基础设施提供支持。
- L13: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L14: Includes `ATen/ops/_coalesce_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_coalesce_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L15: Includes `ATen/ops/repeat_interleave_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/repeat_interleave_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L16: Includes `ATen/ops/cumsum.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/cumsum.h`，为 ATen 的张量/算子基础设施提供支持。
- L17: Includes `ATen/ops/_sparse_sparse_matmul_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_sparse_matmul_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L18: Includes `ATen/ops/_sparse_coo_tensor_unsafe.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_coo_tensor_unsafe.h`，为 ATen 的张量/算子基础设施提供支持。
- L19: Includes `ATen/ops/_sparse_coo_tensor_unsafe_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_coo_tensor_unsafe_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L20: Includes `ATen/ops/cat.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/cat.h`，为 ATen 的张量/算子基础设施提供支持。
- L21: Includes `ATen/ops/softmax_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/softmax_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L22: Includes `ATen/ops/log_softmax.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/log_softmax.h`，为 ATen 的张量/算子基础设施提供支持。
- L23: Includes `ATen/ops/_sparse_log_softmax_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_log_softmax_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L24: Includes `ATen/ops/_sparse_softmax_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_softmax_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L25: Includes `ATen/ops/_sparse_softmax_backward_data_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_softmax_backward_data_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L26: Includes `ATen/ops/_sparse_log_softmax_backward_data_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_log_softmax_backward_data_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L27: Includes `ATen/ops/add_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/add_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L28: Includes `ATen/ops/mul_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/mul_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L29: Includes `ATen/ops/empty_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L30: Includes `ATen/ops/zeros_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/zeros_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L31: Includes `ATen/ops/index_select_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/index_select_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L32: Includes `ATen/ops/remainder_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/remainder_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L33: Includes `ATen/ops/ones_like.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/ones_like.h`，为 ATen 的张量/算子基础设施提供支持。
- L34: Includes `ATen/ops/argsort.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/argsort.h`，为 ATen 的张量/算子基础设施提供支持。
- L35: Includes `ATen/ops/searchsorted_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/searchsorted_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L36: Includes `ATen/ops/_sparse_sum_backward_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_sum_backward_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L37: Includes `ATen/ops/result_type.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/result_type.h`，为 ATen 的张量/算子基础设施提供支持。
- L38: Includes `ATen/ops/bmm_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/bmm_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L39: Includes `ATen/ops/addmm_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/addmm_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L40: Includes `ATen/ops/copy_sparse_to_sparse.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/copy_sparse_to_sparse.h`，为 ATen 的张量/算子基础设施提供支持。

### Lines 41-80

```cpp
  41: #include <ATen/ops/mul.h>
  42: #endif
  43: 
  44: namespace at::native {
  45: 
  46: using namespace at::sparse;
  47: using namespace mps;
  48: 
  49: #ifndef PYTORCH_JIT_COMPILE_SHADERS
  50: static auto& lib = MetalShaderLibrary::getBundledLibrary();
  51: #else
  52: #include <ATen/native/mps/SparseTensorMath_metallib.h>
  53: #endif
  54: 
  55: static Tensor& s_addmm_out_sparse_dense_mps(
  56:     Tensor& r,
  57:     const Tensor& t,
  58:     const SparseTensor& sparse_,
  59:     const Tensor& dense,
  60:     const Scalar& beta,
  61:     const Scalar& alpha) {
  62:   TORCH_CHECK(sparse_.sparse_dim() == 2, "addmm: sparse_dim must be 2, got ", sparse_.sparse_dim());
  63:   TORCH_CHECK(sparse_.dense_dim() == 0, "addmm: sparse values must be 0-dense-dim, got ", sparse_.dense_dim());
  64:   TORCH_CHECK(dense.dim() == 2, "addmm: 'dense' must be 2D, got ", dense.dim());
  65:   TORCH_CHECK(t.dim() == 2, "addmm: 't' must be 2D, got ", t.dim());
  66: 
  67:   const int64_t I = sparse_.size(0);
  68:   const int64_t J = sparse_.size(1);
  69:   const int64_t K = dense.size(1);
  70: 
  71:   TORCH_CHECK(dense.size(0) == J,
  72:       "addmm: dense (mat2) dim0 must be ", J, ", got ", dense.size(0));
  73:   TORCH_CHECK(t.size(0) == I && t.size(1) == K,
  74:       "addmm: 't' shape must be (", I, ", ", K, "), got (", t.size(0), ", ", t.size(1), ")");
  75: 
  76:   r.resize_({I, K});
  77: 
  78:   auto sparse = sparse_.coalesce();
  79:   const int64_t nnz = sparse._nnz();
  80: 
```
- L41: Includes `ATen/ops/mul.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/mul.h`，为 ATen 的张量/算子基础设施提供支持。
- L42: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L44: Opens namespace `at::native` to scope the following declarations. / 打开命名空间 `at::native`，为后续声明限定作用域。
- L46: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L47: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L49: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L50: Declares function `getBundledLibrary` as part of this file's callable surface. / 声明函数 `getBundledLibrary`，作为本文件可调用接口的一部分。
- L51: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L52: Includes `ATen/native/mps/SparseTensorMath_metallib.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/mps/SparseTensorMath_metallib.h`，为 ATen 的张量/算子基础设施提供支持。
- L53: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L55: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L56: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L57: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L58: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L59: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L60: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L61: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L62: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L63: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L64: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L65: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L67: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L68: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L69: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L71: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L72: Declares function `dense` as part of this file's callable surface. / 声明函数 `dense`，作为本文件可调用接口的一部分。
- L73: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L74: Declares function `be` as part of this file's callable surface. / 声明函数 `be`，作为本文件可调用接口的一部分。
- L76: Declares function `resize_` as part of this file's callable surface. / 声明函数 `resize_`，作为本文件可调用接口的一部分。
- L78: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L79: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。

### Lines 81-120

```cpp
  81:   if (nnz == 0 || I == 0 || K == 0) {
  82:     at::mul_out(r, t, beta);
  83:     return r;
  84:   }
  85: 
  86:   const auto v_dtype = sparse._values().scalar_type();
  87:   const auto d_dtype = dense.scalar_type();
  88:   const auto t_dtype = t.scalar_type();
  89:   auto compute_dtype = c10::promoteTypes(c10::promoteTypes(v_dtype, d_dtype), t_dtype);
  90: 
  91:   TORCH_CHECK(canCast(compute_dtype, r.scalar_type()),
  92:               "Can't convert computed type ", compute_dtype, " to output ", r.scalar_type());
  93: 
  94:   auto indices2d = sparse._indices().contiguous();
  95:   auto values = sparse._values().to(compute_dtype);
  96:   auto dense_c = dense.to(compute_dtype).contiguous();
  97:   auto t_c = t.to(compute_dtype).contiguous();
  98: 
  99:   const bool out_needs_cast = (r.scalar_type() != compute_dtype) || !r.is_contiguous();
 100:   Tensor out_buf = out_needs_cast
 101:       ? at::empty({I, K}, r.options().dtype(compute_dtype))
 102:       : r;
 103:   auto out_contig = out_buf.contiguous();
 104: 
 105:   auto device = r.device();
 106:   auto stream = getCurrentMPSStream();
 107: 
 108:   const float alpha_f = alpha.to<float>();
 109:   const float beta_f  = beta.to<float>();
 110: 
 111:   dispatch_sync_with_rethrow(stream->queue(), ^() {
 112:     @autoreleasepool {
 113:       const std::string func = "spmm_addmm_coo_" + mps::scalarToMetalTypeString(values);
 114:       auto pso = lib.getPipelineStateForFunc(func);
 115:       auto enc = stream->commandEncoder();
 116:       [enc setComputePipelineState:pso];
 117: 
 118:       const uint32_t tew = pso.threadExecutionWidth;
 119:       const uint32_t gridX = static_cast<uint32_t>(K);
 120:       const uint32_t gridZ = static_cast<uint32_t>(I);
```
- L81: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L82: Declares function `mul_out` as part of this file's callable surface. / 声明函数 `mul_out`，作为本文件可调用接口的一部分。
- L83: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L84: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L86: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L87: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L88: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L89: Declares function `promoteTypes` as part of this file's callable surface. / 声明函数 `promoteTypes`，作为本文件可调用接口的一部分。
- L91: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L92: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L94: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L95: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L96: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L97: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L99: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L100: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L101: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L102: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L103: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L105: Declares function `device` as part of this file's callable surface. / 声明函数 `device`，作为本文件可调用接口的一部分。
- L106: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L108: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L109: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L111: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L112: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L113: Declares function `scalarToMetalTypeString` as part of this file's callable surface. / 声明函数 `scalarToMetalTypeString`，作为本文件可调用接口的一部分。
- L114: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L115: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L116: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L118: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L119: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L120: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 121-160

```cpp
 121:       const uint32_t tgW = std::min<uint32_t>(gridX, tew);
 122: 
 123:       MTLSize grid = MTLSizeMake(gridX, 1, gridZ);
 124:       MTLSize tgs = MTLSizeMake(tgW, 1, 1);
 125: 
 126:       mtl_setArgs(enc,
 127:                   indices2d,
 128:                   values,
 129:                   dense_c,
 130:                   t_c,
 131:                   out_contig,
 132:                   std::array<uint32_t, 3>{static_cast<uint32_t>(I),
 133:                                            static_cast<uint32_t>(J),
 134:                                            static_cast<uint32_t>(K)},
 135:                   std::array<float, 2>{alpha_f, beta_f},
 136:                   static_cast<uint32_t>(nnz));
 137:       [enc dispatchThreads:grid threadsPerThreadgroup:tgs];
 138:     }
 139:   });
 140: 
 141:   if (out_needs_cast) {
 142:     r.copy_(out_contig.to(r.scalar_type()));
 143:   }
 144: 
 145:   return r;
 146: }
 147: 
 148: 
 149: static void build_batch_ptr_mps(
 150:     const Tensor& indices_dim0,
 151:     int64_t B,
 152:     Tensor& batch_ptr
 153: ) {
 154:   // Builds an array of pointers which point to each batches elements. Example:
 155:   // idx_b = [0, 0, 0, 1, 1, 2, 2, 2, 2]  // 9 non-zero elements
 156:   //          └─────┘  └──┘  └─────────┘
 157:   //          batch 0  batch 1  batch 2
 158:   // batch_ptr = [0, 3, 5, 9]
 159:   //              │  │  │  └─ end of batch 2 (total nnz)
 160:   //              │  │  └──── batch 2 starts at index 5
```
- L121: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L123: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L124: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L126: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L127: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L128: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L129: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L130: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L131: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L132: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L133: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L134: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L135: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L136: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L137: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L138: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L139: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L141: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L142: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L143: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L145: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L146: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L149: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L150: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L151: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L152: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L153: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L154: Documents the nearby logic: Builds an array of pointers which point to each batches elements. Example: / 说明附近逻辑的作用：Builds an array of pointers which point to each batches elements. Example:
- L155: Documents the nearby logic: idx_b = [0, 0, 0, 1, 1, 2, 2, 2, 2]  // 9 non-zero elements / 说明附近逻辑的作用：idx_b = [0, 0, 0, 1, 1, 2, 2, 2, 2]  // 9 non-zero elements
- L156: Documents the nearby logic: └─────┘  └──┘  └─────────┘ / 说明附近逻辑的作用：└─────┘  └──┘  └─────────┘
- L157: Documents the nearby logic: batch 0  batch 1  batch 2 / 说明附近逻辑的作用：batch 0  batch 1  batch 2
- L158: Documents the nearby logic: batch_ptr = [0, 3, 5, 9] / 说明附近逻辑的作用：batch_ptr = [0, 3, 5, 9]
- L159: Documents the nearby logic: │  │  │  └─ end of batch 2 (total nnz) / 说明附近逻辑的作用：│  │  │  └─ end of batch 2 (total nnz)
- L160: Documents the nearby logic: │  │  └──── batch 2 starts at index 5 / 说明附近逻辑的作用：│  │  └──── batch 2 starts at index 5

### Lines 161-200

```cpp
 161:   //              │  └─────── batch 1 starts at index 3
 162:   //              └────────── batch 0 starts at index 0
 163:   TORCH_CHECK(indices_dim0.is_mps() && batch_ptr.is_mps(), "MPS device expected");
 164:   auto device = indices_dim0.device();
 165:   auto stream = getCurrentMPSStream();
 166: 
 167:   const int64_t nnz = indices_dim0.numel();
 168: 
 169:   dispatch_sync_with_rethrow(stream->queue(), ^() {
 170:     @autoreleasepool {
 171:       auto pso = lib.getPipelineStateForFunc("build_batch_ptr_from_sorted_batches");
 172:       auto enc = stream->commandEncoder();
 173:       [enc setComputePipelineState:pso];
 174: 
 175:       const uint32_t tew = pso.threadExecutionWidth;
 176:       const uint32_t Q = static_cast<uint32_t>(B + 1);
 177:       const uint32_t tgW = std::min<uint32_t>(Q, tew);
 178:       MTLSize grid = MTLSizeMake(Q, 1, 1);
 179:       MTLSize tgs  = MTLSizeMake(tgW, 1, 1);
 180: 
 181:       mtl_setArgs(enc,
 182:                   indices_dim0,
 183:                   batch_ptr,
 184:                   std::array<uint32_t, 2>{static_cast<uint32_t>(nnz),
 185:                                           static_cast<uint32_t>(B)});
 186:       [enc dispatchThreads:grid threadsPerThreadgroup:tgs];
 187:     }
 188:   });
 189: }
 190: 
 191: static void build_row_ptr_per_batch_mps(
 192:     const Tensor& rows,
 193:     const Tensor& batch_ptr,
 194:     int64_t B,
 195:     int64_t I,
 196:     Tensor& row_ptr
 197: ) {
 198:   // Build per-batch CSR-style row pointer arrays from row indices sorted by batch
 199:   // Given:
 200:   //   rows: 1-D array of length nnz with row ids in [0, I), sorted within each batch
```
- L161: Documents the nearby logic: │  └─────── batch 1 starts at index 3 / 说明附近逻辑的作用：│  └─────── batch 1 starts at index 3
- L162: Documents the nearby logic: └────────── batch 0 starts at index 0 / 说明附近逻辑的作用：└────────── batch 0 starts at index 0
- L163: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L164: Declares function `device` as part of this file's callable surface. / 声明函数 `device`，作为本文件可调用接口的一部分。
- L165: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L167: Declares function `numel` as part of this file's callable surface. / 声明函数 `numel`，作为本文件可调用接口的一部分。
- L169: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L170: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L171: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L172: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L173: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L175: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L176: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L177: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L178: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L179: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L181: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L182: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L183: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L184: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L185: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L186: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L187: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L188: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L189: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L191: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L192: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L193: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L194: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L195: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L196: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L197: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L198: Documents the nearby logic: Build per-batch CSR-style row pointer arrays from row indices sorted by batch / 说明附近逻辑的作用：Build per-batch CSR-style row pointer arrays from row indices sorted by batch
- L199: Documents the nearby logic: Given: / 说明附近逻辑的作用：Given:
- L200: Documents the nearby logic: rows: 1-D array of length nnz with row ids in [0, I), sorted within each batch / 说明附近逻辑的作用：rows: 1-D array of length nnz with row ids in [0, I), sorted within each batch

### Lines 201-240

```cpp
 201:   //   batch_ptr: length B+1, where [batch_ptr[b], batch_ptr[b+1]) is the subrange for batch b
 202:   // Produces:
 203:   //   - row_ptr: shape [B, I+1]
 204:   //
 205:   // Example (B = 2, I = 4):
 206:   // rows       = [0,   0,   1,  3,  0,   2,    2]   // 7 non-zero elements
 207:   //               └─── batch 0 ──┘  └─ batch 1 ─┘
 208:   // batch_ptr  = [0, 4, 7]
 209:   //               │  │  └─ end of batch 1 (total nnz)
 210:   //               │  └──── end of batch 0/start of batch 1
 211:   //               └─────── start of batch 0
 212:   //
 213:   // per-batch row pointers (I+1 entries each):
 214:   //   row_ptr[0] = [0, 2, 3, 3, 4]
 215:   //   row_ptr[1] = [0, 1, 1, 3, 3]
 216:   // laid out in memory: [0, 2, 3, 3, 4,  0, 1, 1, 3, 3]
 217:   TORCH_CHECK(rows.is_mps() && batch_ptr.is_mps() && row_ptr.is_mps(), "MPS device expected");
 218:   auto stream = getCurrentMPSStream();
 219: 
 220:   dispatch_sync_with_rethrow(stream->queue(), ^() {
 221:     @autoreleasepool {
 222:       auto pso = lib.getPipelineStateForFunc("build_row_ptr_from_sorted_rows_by_batch");
 223:       auto enc = stream->commandEncoder();
 224:       [enc setComputePipelineState:pso];
 225: 
 226:       const uint32_t tew = pso.threadExecutionWidth;
 227:       const uint32_t Qx = static_cast<uint32_t>(I + 1);
 228:       const uint32_t Qy = static_cast<uint32_t>(B);
 229:       const uint32_t tgW = std::min<uint32_t>(Qx, tew);
 230: 
 231:       MTLSize grid = MTLSizeMake(Qx, Qy, 1);
 232:       MTLSize tgs = MTLSizeMake(tgW, 1, 1);
 233: 
 234:       mtl_setArgs(enc,
 235:                   rows,
 236:                   batch_ptr,
 237:                   row_ptr,
 238:                   std::array<uint32_t, 2>{static_cast<uint32_t>(I),
 239:                                            static_cast<uint32_t>(B)});
 240:       [enc dispatchThreads:grid threadsPerThreadgroup:tgs];
```
- L201: Documents the nearby logic: batch_ptr: length B+1, where [batch_ptr[b], batch_ptr[b+1]) is the subrange for batch b / 说明附近逻辑的作用：batch_ptr: length B+1, where [batch_ptr[b], batch_ptr[b+1]) is the subrange for batch b
- L202: Documents the nearby logic: Produces: / 说明附近逻辑的作用：Produces:
- L203: Documents the nearby logic: - row_ptr: shape [B, I+1] / 说明附近逻辑的作用：- row_ptr: shape [B, I+1]
- L204: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L205: Documents the nearby logic: Example (B = 2, I = 4): / 说明附近逻辑的作用：Example (B = 2, I = 4):
- L206: Documents the nearby logic: rows       = [0,   0,   1,  3,  0,   2,    2]   // 7 non-zero elements / 说明附近逻辑的作用：rows       = [0,   0,   1,  3,  0,   2,    2]   // 7 non-zero elements
- L207: Documents the nearby logic: └─── batch 0 ──┘  └─ batch 1 ─┘ / 说明附近逻辑的作用：└─── batch 0 ──┘  └─ batch 1 ─┘
- L208: Documents the nearby logic: batch_ptr  = [0, 4, 7] / 说明附近逻辑的作用：batch_ptr  = [0, 4, 7]
- L209: Documents the nearby logic: │  │  └─ end of batch 1 (total nnz) / 说明附近逻辑的作用：│  │  └─ end of batch 1 (total nnz)
- L210: Documents the nearby logic: │  └──── end of batch 0/start of batch 1 / 说明附近逻辑的作用：│  └──── end of batch 0/start of batch 1
- L211: Documents the nearby logic: └─────── start of batch 0 / 说明附近逻辑的作用：└─────── start of batch 0
- L212: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L213: Documents the nearby logic: per-batch row pointers (I+1 entries each): / 说明附近逻辑的作用：per-batch row pointers (I+1 entries each):
- L214: Documents the nearby logic: row_ptr[0] = [0, 2, 3, 3, 4] / 说明附近逻辑的作用：row_ptr[0] = [0, 2, 3, 3, 4]
- L215: Documents the nearby logic: row_ptr[1] = [0, 1, 1, 3, 3] / 说明附近逻辑的作用：row_ptr[1] = [0, 1, 1, 3, 3]
- L216: Documents the nearby logic: laid out in memory: [0, 2, 3, 3, 4,  0, 1, 1, 3, 3] / 说明附近逻辑的作用：laid out in memory: [0, 2, 3, 3, 4,  0, 1, 1, 3, 3]
- L217: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L218: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L220: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L221: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L222: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L223: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L224: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L226: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L227: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L228: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L229: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L231: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L232: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L234: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L235: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L236: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L237: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L238: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L239: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L240: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 241-280

```cpp
 241:     }
 242:   });
 243: }
 244: 
 245: Tensor& bmm_out_sparse_mps(const SparseTensor& self_, const Tensor& mat2_, Tensor& result_) {
 246:   TORCH_CHECK(result_.is_mps(), "bmm_sparse: expected 'out' to be MPS, got ", result_.device());
 247:   TORCH_CHECK(self_.is_mps(),  "bmm_sparse: expected 'self' to be MPS, got ", self_.device());
 248:   TORCH_CHECK(mat2_.is_mps(),  "bmm_sparse: expected 'mat2' to be MPS, got ", mat2_.device());
 249: 
 250:   TORCH_CHECK(self_.dense_dim() == 0, "bmm_sparse: Tensor 'self' must have 0 dense dims, but has ", self_.dense_dim());
 251:   TORCH_CHECK(self_.sparse_dim() == 3, "bmm_sparse: Tensor 'self' must have 3 sparse dims, but has ", self_.sparse_dim());
 252:   TORCH_CHECK(mat2_.dim() == 3, "bmm_sparse: Tensor 'mat2' must have 3 dims, but has ", mat2_.dim());
 253: 
 254:   TORCH_CHECK(self_.size(0) == mat2_.size(0), "bmm_sparse: 'self.size(0)' and 'mat2.size(0)' must match");
 255:   TORCH_CHECK(self_.size(2) == mat2_.size(1), "bmm_sparse: 'self.size(2)' and 'mat2.size(1)' must match");
 256: 
 257:   const int64_t B = self_.size(0);
 258:   const int64_t I = self_.size(1);
 259:   const int64_t J = self_.size(2);
 260:   const int64_t K = mat2_.size(2);
 261: 
 262:   auto self = self_.coalesce();
 263:   const int64_t nnz = self._nnz();
 264:   if (nnz == 0) {
 265:     return result_.zero_();
 266:   }
 267: 
 268:   const auto computeDtype = at::kFloat;
 269: 
 270:   auto indices = self._indices();
 271:   auto values  = self._values();
 272: 
 273:   auto values_c = values.scalar_type() == computeDtype ? values : values.to(computeDtype);
 274:   auto mat2_c = mat2_.scalar_type()   == computeDtype ? mat2_   : mat2_.to(computeDtype);
 275:   auto mat2_contig = mat2_c.contiguous();
 276: 
 277:   auto idx_b = indices.select(0, 0).contiguous();
 278:   auto idx_i = indices.select(0, 1).contiguous();
 279:   auto idx_j = indices.select(0, 2).contiguous();
 280: 
```
- L241: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L242: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L243: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L245: Defines function `bmm_out_sparse_mps` and begins its implementation body. / 定义函数 `bmm_out_sparse_mps`，并开始其实现体。
- L246: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L247: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L248: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L250: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L251: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L252: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L254: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L255: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L257: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L258: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L259: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L260: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L262: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L263: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L264: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L265: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L266: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L268: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L270: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L271: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L273: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L274: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L275: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L277: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L278: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L279: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。

### Lines 281-320

```cpp
 281:   // builds an array of pointers of where the batch_idx's pointer starts and ends
 282:   // look in function for better explanation
 283:   auto batch_ptr = at::empty({B + 1}, at::device(result_.device()).dtype(kLong));
 284:   build_batch_ptr_mps(idx_b, B, batch_ptr);
 285:   // build row_ptr per batch: for each (b, i) get [start, end) into rows/cols/vals
 286:   auto row_ptr = at::empty({B * (I + 1)}, at::device(result_.device()).dtype(kLong));
 287:   build_row_ptr_per_batch_mps(idx_i, batch_ptr, B, I, row_ptr);
 288: 
 289:   const bool out_needs_cast = (result_.scalar_type() != computeDtype) || !result_.is_contiguous();
 290:   Tensor out_buf = out_needs_cast
 291:       ? at::empty({B, I, K}, result_.options().dtype(computeDtype))
 292:       : result_;
 293:   auto out_contig = out_buf.contiguous();
 294: 
 295:   auto stream = getCurrentMPSStream();
 296:   dispatch_sync_with_rethrow(stream->queue(), ^() {
 297:     @autoreleasepool {
 298:       auto pso = lib.getPipelineStateForFunc("spmm_bmm_coo_rows_grouped_" + mps::scalarToMetalTypeString(values));
 299:       auto enc = stream->commandEncoder();
 300:       [enc setComputePipelineState:pso];
 301: 
 302:       const uint32_t tew = pso.threadExecutionWidth;
 303:       const uint32_t tgW = std::min<uint32_t>((uint32_t)K, tew);
 304: 
 305:       // One threadgroup per (row i, batch b), lanes cover K
 306:       MTLSize grid = MTLSizeMake(tgW, (uint32_t)I, (uint32_t)B);
 307:       MTLSize tgs  = MTLSizeMake(tgW, 1, 1);
 308: 
 309:       mtl_setArgs(enc,
 310:                   idx_i,
 311:                   idx_j,
 312:                   values_c,
 313:                   mat2_contig,
 314:                   out_contig,
 315:                   row_ptr,
 316:                   std::array<uint32_t, 4>{(uint32_t)B, (uint32_t)I, (uint32_t)J, (uint32_t)K});
 317:       [enc dispatchThreads:grid threadsPerThreadgroup:tgs];
 318:     }
 319:   });
 320:   if (out_needs_cast) {
```
- L281: Documents the nearby logic: builds an array of pointers of where the batch_idx's pointer starts and ends / 说明附近逻辑的作用：builds an array of pointers of where the batch_idx's pointer starts and ends
- L282: Documents the nearby logic: look in function for better explanation / 说明附近逻辑的作用：look in function for better explanation
- L283: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L284: Declares function `build_batch_ptr_mps` as part of this file's callable surface. / 声明函数 `build_batch_ptr_mps`，作为本文件可调用接口的一部分。
- L285: Documents the nearby logic: build row_ptr per batch: for each (b, i) get [start, end) into rows/cols/vals / 说明附近逻辑的作用：build row_ptr per batch: for each (b, i) get [start, end) into rows/cols/vals
- L286: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L287: Declares function `build_row_ptr_per_batch_mps` as part of this file's callable surface. / 声明函数 `build_row_ptr_per_batch_mps`，作为本文件可调用接口的一部分。
- L289: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L290: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L291: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L292: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L293: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L295: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L296: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L297: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L298: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L299: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L300: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L302: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L303: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L305: Documents the nearby logic: One threadgroup per (row i, batch b), lanes cover K / 说明附近逻辑的作用：One threadgroup per (row i, batch b), lanes cover K
- L306: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L307: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L309: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L310: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L311: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L312: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L313: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L314: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L315: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L316: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L317: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L318: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L319: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L320: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。

### Lines 321-360

```cpp
 321:     result_.copy_(out_contig.to(result_.scalar_type()));
 322:   }
 323:   return result_;
 324: }
 325: 
 326: Tensor bmm_sparse_mps(const Tensor& self, const Tensor& mat2) {
 327:   Tensor result = at::zeros({self.size(0), self.size(1), mat2.size(2)}, mat2.options());
 328:   return bmm_out_sparse_mps(self, mat2, result);
 329: }
 330: 
 331: Tensor& addmm_out_sparse_dense_mps(
 332:     const Tensor& self,
 333:     const SparseTensor& mat1,
 334:     const Tensor& mat2,
 335:     const Scalar& beta,
 336:     const Scalar& alpha,
 337:     Tensor& result) {
 338:   c10::MaybeOwned<Tensor> b_self = expand_size(self, {mat1.size(0), mat2.size(1)}, "addmm_out");
 339:   return s_addmm_out_sparse_dense_mps(result, *b_self, mat1, mat2, beta, alpha);
 340: }
 341: 
 342: Tensor addmm_sparse_dense_mps(
 343:     const Tensor& self,
 344:     const SparseTensor& mat1,
 345:     const Tensor& mat2,
 346:     const Scalar& beta,
 347:     const Scalar& alpha
 348: ) {
 349:   c10::MaybeOwned<Tensor> b_self = expand_size(self, {mat1.size(0), mat2.size(1)}, "addmm_out");
 350:   Tensor result = at::empty({0}, self.options());
 351:   return s_addmm_out_sparse_dense_mps(result, *b_self, mat1, mat2, beta, alpha);
 352: }
 353: 
 354: static SparseTensor& mul_out_dense_sparse_mps(
 355:     const Tensor& dense,
 356:     const Tensor& sparse,
 357:     SparseTensor& out) {
 358: 
 359:   TORCH_CHECK(sparse.is_sparse(), "mul: expected 'sparse' to be sparse COO");
 360:   TORCH_CHECK(sparse.is_mps(), "mul: expected 'sparse' to be MPS, got ", sparse.device());
```
- L321: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L322: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L323: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L324: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L326: Defines function `bmm_sparse_mps` and begins its implementation body. / 定义函数 `bmm_sparse_mps`，并开始其实现体。
- L327: Declares function `zeros` as part of this file's callable surface. / 声明函数 `zeros`，作为本文件可调用接口的一部分。
- L328: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L329: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L331: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L332: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L333: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L334: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L335: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L336: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L337: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L338: Declares function `expand_size` as part of this file's callable surface. / 声明函数 `expand_size`，作为本文件可调用接口的一部分。
- L339: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L340: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L342: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L343: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L344: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L345: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L346: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L347: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L348: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L349: Declares function `expand_size` as part of this file's callable surface. / 声明函数 `expand_size`，作为本文件可调用接口的一部分。
- L350: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L351: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L352: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L354: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L355: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L356: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L357: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L359: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L360: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。

### Lines 361-400

```cpp
 361:   TORCH_CHECK(out.is_mps(), "mul: expected 'out' to be MPS, got ", out.device());
 362: 
 363:   const bool scalar_like = (dense.dim() == 0) || (dense.numel() == 1);
 364:   TORCH_CHECK(dense.is_mps() || scalar_like,
 365:               "mul: expected 'dense' to be MPS or scalar-like, got ", dense.device());
 366: 
 367:   const int64_t nnz = sparse._nnz();
 368:   out.resize_as_(sparse);
 369: 
 370:   auto commonDtype = at::result_type(dense, sparse);
 371:   TORCH_CHECK(canCast(commonDtype, out.scalar_type()),
 372:               "Can't convert result type ", commonDtype, " to output ", out.scalar_type());
 373: 
 374:   auto indices = sparse._indices().contiguous();
 375:   auto values  = sparse._values().to(commonDtype).contiguous();
 376: 
 377:   if (nnz == 0) {
 378:     auto empty_vals = values.narrow(0, 0, 0);
 379:     alias_into_sparse(out,
 380:                       indices.narrow(1, 0, 0),
 381:                       (out.scalar_type() == commonDtype) ? empty_vals
 382:                                                           : empty_vals.to(out.scalar_type()));
 383:     out._coalesced_(sparse.is_coalesced());
 384:     return out;
 385:   }
 386: 
 387:   if (scalar_like) {
 388:     auto out_vals = values.mul(dense.to(values.options()));
 389:     if (out.scalar_type() != commonDtype) {
 390:       out_vals = out_vals.to(out.scalar_type());
 391:     }
 392: 
 393:     alias_into_sparse(out, indices, out_vals);
 394:     out._coalesced_(sparse.is_coalesced());
 395:     return out;
 396:   }
 397: 
 398:   TORCH_CHECK(dense.sizes().equals(sparse.sizes()),
 399:               "mul(dense, sparse): sizes must match exactly (no broadcasting): ",
 400:               dense.sizes(), " vs ", sparse.sizes());
```
- L361: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L363: Declares function `dim` as part of this file's callable surface. / 声明函数 `dim`，作为本文件可调用接口的一部分。
- L364: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L365: Declares function `device` as part of this file's callable surface. / 声明函数 `device`，作为本文件可调用接口的一部分。
- L367: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L368: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L370: Declares function `result_type` as part of this file's callable surface. / 声明函数 `result_type`，作为本文件可调用接口的一部分。
- L371: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L372: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L374: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L375: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L377: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L378: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L379: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L380: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L381: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L382: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L383: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L384: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L385: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L387: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L388: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L389: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L390: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L391: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L393: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L394: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L395: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L396: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L398: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L399: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L400: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。

### Lines 401-440

```cpp
 401: 
 402:   const int64_t ndim_i = sparse.sparse_dim();
 403:   const int64_t ndim = dense.dim();
 404:   TORCH_CHECK(
 405:     ndim_i <= ndim,
 406:     "mul(dense, sparse): sparse_dim=", ndim_i, " exceeds dense.dim()=", ndim);
 407: 
 408:   // Prepare shapes
 409:   int64_t view_rows = 1, view_cols = 1;
 410:   for (int64_t i = 0; i < ndim_i; ++i) view_rows *= sparse.size(i);
 411:   for (int64_t i = ndim_i; i < ndim; ++i) view_cols *= sparse.size(i);
 412: 
 413:   auto dense_mps = dense.to(commonDtype).contiguous().reshape({view_rows, view_cols});
 414:   auto out_vals = at::empty_like(values, values.options());
 415: 
 416:   const uint32_t u_view_cols = static_cast<uint32_t>(view_cols);
 417:   const uint32_t u_nnz = static_cast<uint32_t>(nnz);
 418:   const uint32_t u_ndim_i = static_cast<uint32_t>(ndim_i);
 419: 
 420:   auto stream = getCurrentMPSStream();
 421:   dispatch_sync_with_rethrow(stream->queue(), ^() {
 422:     @autoreleasepool {
 423:       auto pso = lib.getPipelineStateForFunc("dense_sparse_mul_kernel_" + mps::scalarToMetalTypeString(values));
 424:       auto computeEncoder = stream->commandEncoder();
 425:       [computeEncoder setComputePipelineState:pso];
 426: 
 427:       const uint32_t gridWidth = u_view_cols;
 428:       const uint32_t gridDepth = u_nnz;
 429:       MTLSize gridSize = MTLSizeMake(gridWidth, 1, gridDepth);
 430: 
 431:       const uint32_t maxThreadsPerGroup = pso.maxTotalThreadsPerThreadgroup;
 432:       const uint32_t tew = pso.threadExecutionWidth;
 433:       uint32_t tgWidth  = std::min(gridWidth, tew);
 434:       MTLSize threadgroupSize = MTLSizeMake(tgWidth, 1, 1);
 435: 
 436:       mtl_setArgs(
 437:         computeEncoder,
 438:         dense_mps,
 439:         values,
 440:         out_vals,
```
- L402: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L403: Declares function `dim` as part of this file's callable surface. / 声明函数 `dim`，作为本文件可调用接口的一部分。
- L404: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L405: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L406: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L408: Documents the nearby logic: Prepare shapes / 说明附近逻辑的作用：Prepare shapes
- L409: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L410: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L411: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L413: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L414: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L416: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L417: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L418: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L420: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L421: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L422: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L423: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L424: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L425: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L427: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L428: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L429: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L431: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L432: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L433: Declares function `min` as part of this file's callable surface. / 声明函数 `min`，作为本文件可调用接口的一部分。
- L434: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L436: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L437: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L438: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L439: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L440: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 441-480

```cpp
 441:         indices,
 442:         sparse.sizes(),
 443:         std::array<uint32_t, 3>{u_nnz, u_ndim_i, u_view_cols}
 444:       );
 445: 
 446:       [computeEncoder dispatchThreads:gridSize threadsPerThreadgroup:threadgroupSize];
 447:     }
 448:   });
 449: 
 450:   Tensor final_vals = out_vals;
 451:   if (out.scalar_type() != commonDtype) {
 452:     final_vals = final_vals.to(out.scalar_type());
 453:   }
 454: 
 455:   alias_into_sparse(out, indices, final_vals);
 456:   out._coalesced_(sparse.is_coalesced());
 457:   return out;
 458: }
 459: 
 460: static std::tuple<Tensor, Tensor, int64_t> mps_intersect_binary_search(
 461:     const Tensor& A_keys,
 462:     const Tensor& B_keys,
 463:     int64_t lenA,
 464:     int64_t lenB,
 465:     bool boolean_flag) {
 466: 
 467:   auto stream = getCurrentMPSStream();
 468:   auto outA_idx = at::empty({lenA}, A_keys.options().dtype(at::kLong));
 469:   auto outB_idx = at::empty({lenA}, A_keys.options().dtype(at::kLong));
 470:   auto counter = at::zeros({1}, A_keys.options().dtype(at::kInt));
 471: 
 472:   dispatch_sync_with_rethrow(stream->queue(), ^() {
 473:     @autoreleasepool {
 474:       auto pso = lib.getPipelineStateForFunc("intersect_binary_search");
 475:       auto enc = stream->commandEncoder();
 476:       [enc setComputePipelineState:pso];
 477:       mtl_setArgs(enc, A_keys, B_keys, outA_idx, outB_idx, counter,
 478:                   static_cast<uint32_t>(lenB), boolean_flag);
 479:       mtl_dispatch1DJob(enc, pso, static_cast<uint32_t>(lenA));
 480:     }
```
- L441: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L442: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L443: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L444: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L446: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L447: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L448: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L450: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L451: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L452: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L453: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L455: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L456: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L457: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L458: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L460: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L461: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L462: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L463: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L464: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L465: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L467: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L468: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L469: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L470: Declares function `zeros` as part of this file's callable surface. / 声明函数 `zeros`，作为本文件可调用接口的一部分。
- L472: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L473: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L474: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L475: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L476: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L477: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L478: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L479: Declares function `mtl_dispatch1DJob` as part of this file's callable surface. / 声明函数 `mtl_dispatch1DJob`，作为本文件可调用接口的一部分。
- L480: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 481-520

```cpp
 481:   });
 482: 
 483:   const auto match_count = static_cast<int64_t>(counter.item<int32_t>());
 484:   return std::make_tuple(std::move(outA_idx), std::move(outB_idx), match_count);
 485: }
 486: 
 487: 
 488: SparseTensor& mul_out_sparse_mps(const Tensor& t_, const Tensor& src_, SparseTensor& r_) {
 489:   TORCH_CHECK(r_.is_mps(), "mul: expected 'out' to be MPS, but got ", r_.device());
 490: 
 491:   // Dense x sparse fallback (keep dense first)
 492:   if (!t_.is_sparse() || !src_.is_sparse()) {
 493:     const Tensor& dense  = t_.is_sparse() ? src_ : t_;
 494:     const Tensor& sparse = t_.is_sparse() ? t_   : src_;
 495:     return mul_out_dense_sparse_mps(dense, sparse, r_);
 496:   }
 497: 
 498:   TORCH_CHECK(t_.is_mps(),   "mul: expected 'self' to be MPS, but got ", t_.device());
 499:   TORCH_CHECK(src_.is_mps(), "mul: expected 'other' to be MPS, but got ", src_.device());
 500:   TORCH_CHECK(t_.sparse_dim() == src_.sparse_dim(),
 501:               "mul(sparse, sparse): must have same sparse_dim, got ",
 502:               t_.sparse_dim(), " vs ", src_.sparse_dim());
 503: 
 504:   // Coalesce and structural info
 505:   auto lhs = t_.coalesce();
 506:   auto rhs = src_.coalesce();
 507:   const int64_t lhs_nnz = lhs._nnz();
 508:   const int64_t rhs_nnz = rhs._nnz();
 509:   const int64_t sd = lhs.sparse_dim();
 510: 
 511:   // dtype checks and promotion
 512:   auto commonDtype = at::result_type(lhs, rhs);
 513:   TORCH_CHECK(canCast(commonDtype, r_.scalar_type()),
 514:               "Can't convert result type ", commonDtype, " to output ", r_.scalar_type());
 515: 
 516:   // sparse sizes must match exactly, dense tails may broadcast
 517:   TORCH_CHECK(lhs.sizes().slice(0, sd).equals(rhs.sizes().slice(0, sd)),
 518:               "mul(sparse, sparse): sparse sizes must match exactly.");
 519: 
 520:   // dense tails and broadcasted dense tail
```
- L481: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L483: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L484: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L485: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L488: Defines function `mul_out_sparse_mps` and begins its implementation body. / 定义函数 `mul_out_sparse_mps`，并开始其实现体。
- L489: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L491: Documents the nearby logic: Dense x sparse fallback (keep dense first) / 说明附近逻辑的作用：Dense x sparse fallback (keep dense first)
- L492: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L493: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L494: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L495: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L496: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L498: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L499: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L500: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L501: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L502: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L504: Documents the nearby logic: Coalesce and structural info / 说明附近逻辑的作用：Coalesce and structural info
- L505: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L506: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L507: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L508: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L509: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L511: Documents the nearby logic: dtype checks and promotion / 说明附近逻辑的作用：dtype checks and promotion
- L512: Declares function `result_type` as part of this file's callable surface. / 声明函数 `result_type`，作为本文件可调用接口的一部分。
- L513: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L514: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L516: Documents the nearby logic: sparse sizes must match exactly, dense tails may broadcast / 说明附近逻辑的作用：sparse sizes must match exactly, dense tails may broadcast
- L517: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L518: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L520: Documents the nearby logic: dense tails and broadcasted dense tail / 说明附近逻辑的作用：dense tails and broadcasted dense tail

### Lines 521-560

```cpp
 521:   auto lhs_dense = lhs.sizes().slice(sd);
 522:   auto rhs_dense = rhs.sizes().slice(sd);
 523:   std::vector<int64_t> out_dense_vec = at::infer_size(lhs_dense, rhs_dense);
 524:   at::IntArrayRef out_dense(out_dense_vec);
 525: 
 526:   // full output sizes: [sparse_sizes] + [out_dense]
 527:   std::vector<int64_t> out_sizes;
 528:   out_sizes.reserve(sd + static_cast<int64_t>(out_dense.size()));
 529:   out_sizes.insert(out_sizes.end(), lhs.sizes().begin(), lhs.sizes().begin() + sd);
 530:   out_sizes.insert(out_sizes.end(), out_dense.begin(), out_dense.end());
 531:   r_.sparse_resize_(out_sizes, sd, static_cast<int64_t>(out_dense.size()));
 532: 
 533:   const auto device = r_.device();
 534: 
 535:   // if either is structurally empty, produce an empty sparse result with correct shape
 536:   if (!lhs_nnz || !rhs_nnz) {
 537:     Tensor out_indices = at::empty({sd, 0}, at::device(device).dtype(at::kLong));
 538: 
 539:     std::vector<int64_t> out_val_sizes;
 540:     out_val_sizes.reserve(1 + out_dense.size());
 541:     out_val_sizes.push_back(0);
 542:     out_val_sizes.insert(out_val_sizes.end(), out_dense.begin(), out_dense.end());
 543: 
 544:     Tensor out_values = at::empty(out_val_sizes, at::device(device).dtype(r_.scalar_type()));
 545: 
 546:     alias_into_sparse(r_, out_indices, out_values);
 547:     r_._coalesced_(true);
 548:     return r_;
 549:   }
 550: 
 551:   if (sd == 0) {
 552:     const bool has = (lhs_nnz && rhs_nnz);
 553: 
 554:     auto out_indices = at::empty({0, has ? 1 : 0}, lhs._indices().options());
 555: 
 556:     Tensor lhs_vals = lhs._values().to(commonDtype);
 557:     Tensor rhs_vals = rhs._values().to(commonDtype);
 558:     lhs_vals = lhs_vals.narrow(0, 0, has ? 1 : 0);
 559:     rhs_vals = rhs_vals.narrow(0, 0, has ? 1 : 0);
 560: 
```
- L521: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L522: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L523: Declares function `infer_size` as part of this file's callable surface. / 声明函数 `infer_size`，作为本文件可调用接口的一部分。
- L524: Declares function `out_dense` as part of this file's callable surface. / 声明函数 `out_dense`，作为本文件可调用接口的一部分。
- L526: Documents the nearby logic: full output sizes: [sparse_sizes] + [out_dense] / 说明附近逻辑的作用：full output sizes: [sparse_sizes] + [out_dense]
- L527: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L528: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L529: Inserts a new entry into a container or mapping structure. / 向容器或映射结构插入新条目。
- L530: Inserts a new entry into a container or mapping structure. / 向容器或映射结构插入新条目。
- L531: Declares function `sparse_resize_` as part of this file's callable surface. / 声明函数 `sparse_resize_`，作为本文件可调用接口的一部分。
- L533: Declares function `device` as part of this file's callable surface. / 声明函数 `device`，作为本文件可调用接口的一部分。
- L535: Documents the nearby logic: if either is structurally empty, produce an empty sparse result with correct shape / 说明附近逻辑的作用：if either is structurally empty, produce an empty sparse result with correct shape
- L536: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L537: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L539: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L540: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L541: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L542: Inserts a new entry into a container or mapping structure. / 向容器或映射结构插入新条目。
- L544: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L546: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L547: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L548: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L549: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L551: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L552: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L554: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L556: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L557: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L558: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L559: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。

### Lines 561-600

```cpp
 561:     Tensor out_values = lhs_vals.mul(rhs_vals);
 562:     if (r_.scalar_type() != commonDtype) {
 563:       out_values = out_values.to(r_.scalar_type());
 564:     }
 565: 
 566:     alias_into_sparse(r_, out_indices, out_values);
 567:     r_._coalesced_(true);
 568:     return r_;
 569:   }
 570: 
 571:   // General path, intersect keys, then gather + multiply on GPU
 572:   auto stream = getCurrentMPSStream();
 573: 
 574:   auto lhs_indices = lhs._indices().contiguous();
 575:   auto rhs_indices = rhs._indices().contiguous();
 576:   auto lhs_values  = lhs._values().to(commonDtype).contiguous();
 577:   auto rhs_values  = rhs._values().to(commonDtype).contiguous();
 578: 
 579:   // Flatten sparse indices to keys
 580:   auto lhs_keys = flatten_indices(lhs_indices, lhs.sizes().slice(0, sd));
 581:   auto rhs_keys = flatten_indices(rhs_indices, rhs.sizes().slice(0, sd));
 582: 
 583:   // Intersect sorted keys (search the shorter in the longer)
 584:   const bool A_is_lhs = (lhs_nnz <= rhs_nnz);
 585:   const int64_t lenA = A_is_lhs ? lhs_nnz : rhs_nnz;
 586:   const int64_t lenB = A_is_lhs ? rhs_nnz : lhs_nnz;
 587:   auto A_keys = A_is_lhs ? lhs_keys : rhs_keys;
 588:   auto B_keys = A_is_lhs ? rhs_keys : lhs_keys;
 589: 
 590:   auto [outA_idx, outB_idx, M_int64] = mps_intersect_binary_search(
 591:       A_keys, B_keys, lenA, lenB, A_is_lhs);
 592: 
 593:   const auto M = static_cast<uint32_t>(M_int64); // number of structural matches
 594: 
 595:   auto lhs_match = outA_idx.narrow(0, 0, M_int64);
 596:   auto rhs_match = outB_idx.narrow(0, 0, M_int64);
 597: 
 598:   int64_t cols64 = 1;
 599:   for (auto s : out_dense) cols64 *= s;
 600:   const uint32_t cols = static_cast<uint32_t>(std::max<int64_t>(cols64, 1));
```
- L561: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L562: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L563: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L564: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L566: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L567: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L568: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L569: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L571: Documents the nearby logic: General path, intersect keys, then gather + multiply on GPU / 说明附近逻辑的作用：General path, intersect keys, then gather + multiply on GPU
- L572: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L574: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L575: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L576: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L577: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L579: Documents the nearby logic: Flatten sparse indices to keys / 说明附近逻辑的作用：Flatten sparse indices to keys
- L580: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L581: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L583: Documents the nearby logic: Intersect sorted keys (search the shorter in the longer) / 说明附近逻辑的作用：Intersect sorted keys (search the shorter in the longer)
- L584: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L585: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L586: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L587: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L588: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L590: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L591: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L593: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L595: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L596: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L598: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L599: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L600: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 601-640

```cpp
 601: 
 602:   // to broadcast [nnz, *in_dense] -> [nnz, *out_dense] -> [nnz, cols]
 603:   auto broadcast_to_out2d = [&](const Tensor& vals, int64_t nnz, at::IntArrayRef in_dense) -> Tensor {
 604:     const int64_t d_in = in_dense.size();
 605:     const int64_t d_out = out_dense.size();
 606: 
 607:     std::vector<int64_t> view_shape;
 608:     view_shape.reserve(1 + d_out);
 609:     view_shape.push_back(nnz);
 610:     for (int64_t i = 0; i < d_out - d_in; ++i) {
 611:       view_shape.push_back(1);
 612:     }
 613:     view_shape.insert(view_shape.end(), in_dense.begin(), in_dense.end());
 614: 
 615:     std::vector<int64_t> expand_shape;
 616:     expand_shape.reserve(1 + d_out);
 617:     expand_shape.push_back(nnz);
 618:     expand_shape.insert(expand_shape.end(), out_dense.begin(), out_dense.end());
 619: 
 620:     Tensor v = vals.view(view_shape).expand(expand_shape);
 621:     return (cols64 > 0) ? v.contiguous().view({nnz, cols64})
 622:                         : v.contiguous().view({nnz, 0});
 623:   };
 624: 
 625:   // make both sides broadcasted 2d [nnz, cols] buffers so the kernel can index it
 626:   auto lhs_vals2d = broadcast_to_out2d(lhs_values, lhs_nnz, lhs_dense);
 627:   auto rhs_vals2d = broadcast_to_out2d(rhs_values, rhs_nnz, rhs_dense);
 628: 
 629:   std::vector<int64_t> out_val_sizes;
 630:   out_val_sizes.reserve(1 + out_dense.size());
 631:   out_val_sizes.push_back(static_cast<int64_t>(M));
 632:   out_val_sizes.insert(out_val_sizes.end(), out_dense.begin(), out_dense.end());
 633:   auto out_values = at::empty(out_val_sizes, lhs_values.options());
 634: 
 635:   Tensor out_indices;
 636:   if (M > 0 && cols64 > 0) {
 637:     out_indices = at::empty({sd, M}, at::device(device).dtype(at::kLong));
 638:     dispatch_sync_with_rethrow(stream->queue(), ^() {
 639:       @autoreleasepool {
 640:         auto pso = lib.getPipelineStateForFunc(
```
- L602: Documents the nearby logic: to broadcast [nnz, *in_dense] -> [nnz, *out_dense] -> [nnz, cols] / 说明附近逻辑的作用：to broadcast [nnz, *in_dense] -> [nnz, *out_dense] -> [nnz, cols]
- L603: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L604: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L605: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L607: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L608: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L609: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L610: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L611: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L612: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L613: Inserts a new entry into a container or mapping structure. / 向容器或映射结构插入新条目。
- L615: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L616: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L617: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L618: Inserts a new entry into a container or mapping structure. / 向容器或映射结构插入新条目。
- L620: Declares function `view` as part of this file's callable surface. / 声明函数 `view`，作为本文件可调用接口的一部分。
- L621: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L622: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L623: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L625: Documents the nearby logic: make both sides broadcasted 2d [nnz, cols] buffers so the kernel can index it / 说明附近逻辑的作用：make both sides broadcasted 2d [nnz, cols] buffers so the kernel can index it
- L626: Declares function `broadcast_to_out2d` as part of this file's callable surface. / 声明函数 `broadcast_to_out2d`，作为本文件可调用接口的一部分。
- L627: Declares function `broadcast_to_out2d` as part of this file's callable surface. / 声明函数 `broadcast_to_out2d`，作为本文件可调用接口的一部分。
- L629: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L630: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L631: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L632: Inserts a new entry into a container or mapping structure. / 向容器或映射结构插入新条目。
- L633: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L635: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L636: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L637: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L638: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L639: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L640: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。

### Lines 641-680

```cpp
 641:             "fused_gather_mul_kernel_" + mps::scalarToMetalTypeString(lhs_values));
 642:         auto enc = stream->commandEncoder();
 643:         [enc setComputePipelineState:pso];
 644: 
 645:         const uint32_t tew = pso.threadExecutionWidth;
 646:         const uint32_t gridW = std::max<uint32_t>(cols, 1u);
 647:         const uint32_t tgW = std::min(gridW, tew);
 648:         MTLSize grid = MTLSizeMake(gridW, 1, M);
 649:         MTLSize tgs  = MTLSizeMake(tgW, 1, 1);
 650: 
 651:         mtl_setArgs(enc,
 652:                     lhs_vals2d, rhs_vals2d,
 653:                     lhs_match, rhs_match,
 654:                     lhs_indices, out_indices,
 655:                     out_values,
 656:                     std::array<uint32_t, 2>{static_cast<uint32_t>(sd), static_cast<uint32_t>(lhs_nnz)},
 657:                     std::array<uint32_t, 2>{M, cols});
 658:         [enc dispatchThreads:grid threadsPerThreadgroup:tgs];
 659:       }
 660:     });
 661:   } else if (M > 0) {
 662:     // just select the matching coordinates
 663:     Tensor src_indices_for_out = A_is_lhs ? lhs_indices : rhs_indices;
 664:     Tensor src_match_for_out   = A_is_lhs ? lhs_match    : rhs_match;
 665:     out_indices = src_indices_for_out.index_select(1, src_match_for_out);
 666:   } else {
 667:     // M == 0
 668:     out_indices = at::empty({sd, 0}, at::device(device).dtype(at::kLong));
 669:   }
 670: 
 671:   if (r_.scalar_type() != commonDtype) {
 672:     out_values = out_values.to(r_.scalar_type());
 673:   }
 674: 
 675:   alias_into_sparse(r_, out_indices, out_values);
 676:   r_._coalesced_(true);
 677:   return r_;
 678: }
 679: 
 680: static Tensor& add_out_dense_sparse_mps(
```
- L641: Declares function `scalarToMetalTypeString` as part of this file's callable surface. / 声明函数 `scalarToMetalTypeString`，作为本文件可调用接口的一部分。
- L642: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L643: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L645: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L646: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L647: Declares function `min` as part of this file's callable surface. / 声明函数 `min`，作为本文件可调用接口的一部分。
- L648: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L649: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L651: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L652: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L653: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L654: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L655: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L656: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L657: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L658: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L659: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L660: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L661: Defines function `if` and begins its implementation body. / 定义函数 `if`，并开始其实现体。
- L662: Documents the nearby logic: just select the matching coordinates / 说明附近逻辑的作用：just select the matching coordinates
- L663: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L664: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L665: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L666: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L667: Documents the nearby logic: M == 0 / 说明附近逻辑的作用：M == 0
- L668: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L669: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L671: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L672: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L673: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L675: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L676: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L677: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L678: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L680: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 681-720

```cpp
 681:     Tensor& out,
 682:     const Tensor& dense,
 683:     const SparseTensor& sparse,
 684:     const Scalar& alpha) {
 685:   TORCH_CHECK(dense.is_mps(),  "add: expected 'self' to be an MPS tensor, got ", dense.device());
 686:   TORCH_CHECK(sparse.is_mps(), "add: expected 'other' to be an MPS tensor, got ", sparse.device());
 687:   TORCH_CHECK(out.is_mps(),    "add: expected 'out' to be an MPS tensor, got ", out.device());
 688:   TORCH_CHECK(dense.sizes().equals(sparse.sizes()),
 689:               "add: expected 'self' and 'other' to have same size, but self has size ",
 690:               dense.sizes(), " while other has size ", sparse.sizes(),
 691:               " (FYI: dense-sparse addition does not currently support broadcasting)");
 692: 
 693:   const int64_t nnz = sparse._nnz();
 694:   if (nnz == 0) {
 695:     out.resize_as_(dense);
 696:     out.copy_(dense);
 697:     return out;
 698:   }
 699: 
 700:   auto commonDtype = at::result_type(dense, sparse);
 701:   TORCH_CHECK(canCast(commonDtype, out.scalar_type()),
 702:               "Can't convert result type ", commonDtype, " to output ", out.scalar_type());
 703: 
 704:   Tensor r;
 705:   const bool need_separate_buffer = out.is_same(dense) || (out.scalar_type() != commonDtype);
 706:   if (need_separate_buffer) {
 707:     r = at::empty(dense.sizes(), out.options().dtype(commonDtype));
 708:   } else {
 709:     r = out;
 710:     r.resize_as_(dense);
 711:   }
 712: 
 713:   Tensor dense_buffer = dense.to(commonDtype);
 714:   if (!r.is_same(dense_buffer)) {
 715:     r.copy_(dense_buffer);
 716:   }
 717: 
 718:   Tensor indices = sparse._indices();
 719:   Tensor values  = sparse._values().to(commonDtype);
 720:   if (values.numel() == 0) {
```
- L681: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L682: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L683: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L684: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L685: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L686: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L687: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L688: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L689: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L690: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L691: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L693: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L694: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L695: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L696: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L697: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L698: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L700: Declares function `result_type` as part of this file's callable surface. / 声明函数 `result_type`，作为本文件可调用接口的一部分。
- L701: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L702: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L704: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L705: Declares function `is_same` as part of this file's callable surface. / 声明函数 `is_same`，作为本文件可调用接口的一部分。
- L706: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L707: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L708: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L709: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L710: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L711: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L713: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L714: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L715: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L716: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L718: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L719: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L720: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。

### Lines 721-760

```cpp
 721:     if (!out.is_same(r)) {
 722:       out.resize_as_(dense);
 723:       out.copy_(r);
 724:     }
 725:     return out;
 726:   }
 727: 
 728:   const int64_t nDim  = r.dim();
 729:   const int64_t nDimI = sparse.sparse_dim();
 730:   TORCH_CHECK(nDimI >= 0 && nDimI <= nDim,
 731:               "Invalid sparse_dim=", nDimI, " for dense tensor of dim ", nDim);
 732: 
 733:   Tensor indices1D = at::sparse::flatten_indices(indices, sparse.sizes()).contiguous();
 734: 
 735:   int64_t view_rows = 1;
 736:   int64_t view_cols = 1;
 737:   for (int64_t i = 0; i < nDimI; i++) {
 738:     view_rows *= r.size(i);
 739:   }
 740:   for (int64_t i = nDimI; i < nDim; i++) {
 741:     view_cols *= r.size(i);
 742:   }
 743: 
 744:   if (view_cols == 1) {
 745:     Tensor r_flat = r.reshape({view_rows});
 746:     Tensor values_1d  = values.reshape({nnz});
 747:     r_flat.index_add_(0, indices1D, values_1d, alpha);
 748:   } else {
 749:     Tensor r_view = r.view({view_rows, view_cols});
 750:     Tensor values_2d  = values.reshape({nnz, view_cols});
 751:     r_view.index_add_(0, indices1D, values_2d, alpha);
 752:   }
 753: 
 754:   if (!out.is_same(r)) {
 755:     out.resize_as_(dense);
 756:     out.copy_(r);
 757:   }
 758:   return out;
 759: }
 760: 
```
- L721: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L722: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L723: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L724: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L725: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L726: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L728: Declares function `dim` as part of this file's callable surface. / 声明函数 `dim`，作为本文件可调用接口的一部分。
- L729: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L730: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L731: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L733: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L735: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L736: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L737: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L738: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L739: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L740: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L741: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L742: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L744: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L745: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L746: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L747: Declares function `index_add_` as part of this file's callable surface. / 声明函数 `index_add_`，作为本文件可调用接口的一部分。
- L748: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L749: Declares function `view` as part of this file's callable surface. / 声明函数 `view`，作为本文件可调用接口的一部分。
- L750: Declares function `reshape` as part of this file's callable surface. / 声明函数 `reshape`，作为本文件可调用接口的一部分。
- L751: Declares function `index_add_` as part of this file's callable surface. / 声明函数 `index_add_`，作为本文件可调用接口的一部分。
- L752: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L754: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L755: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L756: Declares function `copy_` as part of this file's callable surface. / 声明函数 `copy_`，作为本文件可调用接口的一部分。
- L757: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L758: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L759: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 761-800

```cpp
 761: 
 762: SparseTensor& add_out_sparse_mps(const SparseTensor& self,
 763:                                  const SparseTensor& other,
 764:                                  const Scalar& alpha,
 765:                                  SparseTensor& out) {
 766:   TORCH_CHECK(other.is_sparse(), "add(sparse, dense) is not supported. Use add(dense, sparse) instead.");
 767:   TORCH_CHECK(self.is_mps(),  "add: expected 'self' to be MPS, but got ", self.device());
 768:   TORCH_CHECK(other.is_mps(), "add: expected 'other' to be MPS, but got ", other.device());
 769:   TORCH_CHECK(out.is_mps(),   "add: expected 'out' to be MPS, but got ", out.device());
 770:   if (!self.is_sparse()) {
 771:     return add_out_dense_sparse_mps(out, self, other, alpha);
 772:   }
 773:   auto commonDtype = at::result_type(self, other);
 774:   TORCH_CHECK(canCast(commonDtype, out.scalar_type()),
 775:               "Can't convert result type ", commonDtype, " to output ", out.scalar_type());
 776: 
 777:   TORCH_CHECK(self.sizes().equals(other.sizes()),
 778:               "add: expected 'self' and 'other' to have same size, but ", self.sizes(), " != ", other.sizes());
 779: 
 780:   if (other._nnz() == 0) {
 781:     out.resize_as_(self);
 782:     Tensor vals = self._values();
 783:     if (vals.scalar_type() != out.scalar_type()) {
 784:       vals = vals.to(out.scalar_type());
 785:     }
 786:     alias_into_sparse(out, self._indices(), vals);
 787:     out._coalesced_(self.is_coalesced());
 788:     return out;
 789:   }
 790: 
 791:   if (self._nnz() == 0) {
 792:     out.resize_as_(other);
 793:     Tensor vals = other._values();
 794:     if (!alpha.isIntegral(false) || alpha.to<double>() != 1.0) {
 795:       vals = at::mul(vals, alpha);
 796:     }
 797:     if (vals.scalar_type() != out.scalar_type()) {
 798:       vals = vals.to(out.scalar_type());
 799:     }
 800:     alias_into_sparse(out, other._indices(), vals);
```
- L762: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L763: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L764: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L765: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L766: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L767: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L768: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L769: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L770: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L771: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L772: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L773: Declares function `result_type` as part of this file's callable surface. / 声明函数 `result_type`，作为本文件可调用接口的一部分。
- L774: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L775: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L777: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L778: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L780: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L781: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L782: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L783: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L784: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L785: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L786: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L787: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L788: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L789: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L791: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L792: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L793: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L794: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L795: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L796: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L797: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L798: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L799: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L800: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。

### Lines 801-840

```cpp
 801:     out._coalesced_(other.is_coalesced());
 802:     return out;
 803:   }
 804: 
 805:   TORCH_CHECK(is_same_density(self, other),
 806:               "add: expected 'self' and 'other' to have same density, but 'self' has ",
 807:               self.sparse_dim(), " sparse dimensions while 'other' has ", other.sparse_dim(), " sparse dimensions");
 808: 
 809:   Tensor t_indices_ = self._indices();
 810:   Tensor s_indices_ = other._indices();
 811: 
 812:   Tensor t_values_ = self._values().to(commonDtype);
 813:   Tensor s_values_ = other._values().to(commonDtype);
 814:   if (!alpha.isIntegral(false) || alpha.to<double>() != 1.0) {
 815:     s_values_ = at::mul(s_values_, alpha);
 816:   }
 817: 
 818:   Tensor r_indices_ = at::cat({t_indices_, s_indices_}, 1);
 819:   Tensor r_values_  = at::cat({t_values_,  s_values_ }, 0);
 820: 
 821:   SparseTensor tmp = empty({0}, out.options().dtype(commonDtype));
 822:   tmp.resize_as_(other);
 823:   alias_into_sparse(tmp, r_indices_, r_values_);
 824:   tmp = _coalesce_sparse_mps(tmp);
 825: 
 826:   out.resize_as_(other);
 827:   Tensor out_vals = tmp._values();
 828:   if (out.scalar_type() != commonDtype) {
 829:     out_vals = out_vals.to(out.scalar_type());
 830:   }
 831:   alias_into_sparse(out, tmp._indices(), out_vals);
 832:   out._coalesced_(tmp.is_coalesced());
 833: 
 834:   return out;
 835: }
 836: 
 837: using OptTensor = std::optional<Tensor>;
 838: 
 839: static Tensor create_sparse_output_values(
 840:     const Tensor& template_values,
```
- L801: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L802: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L803: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L805: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L806: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L807: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L809: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L810: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L812: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L813: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L814: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L815: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L816: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L818: Declares function `cat` as part of this file's callable surface. / 声明函数 `cat`，作为本文件可调用接口的一部分。
- L819: Declares function `cat` as part of this file's callable surface. / 声明函数 `cat`，作为本文件可调用接口的一部分。
- L821: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L822: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L823: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L824: Declares function `_coalesce_sparse_mps` as part of this file's callable surface. / 声明函数 `_coalesce_sparse_mps`，作为本文件可调用接口的一部分。
- L826: Declares function `resize_as_` as part of this file's callable surface. / 声明函数 `resize_as_`，作为本文件可调用接口的一部分。
- L827: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L828: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L829: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L830: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L831: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L832: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L834: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L835: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L837: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L839: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L840: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 841-880

```cpp
 841:     int64_t output_nnz,
 842:     ScalarType dtype) {
 843:   auto out_val_sizes = template_values.sizes().vec();
 844:   out_val_sizes[0] = output_nnz;
 845:   return at::zeros(out_val_sizes, template_values.options().dtype(dtype));
 846: }
 847: 
 848: static void sparse_mask_apply_out_mps_kernel(
 849:     Tensor& result,
 850:     const Tensor& src_in,
 851:     const Tensor& mask_in,
 852:     bool accumulate_matches,
 853:     bool require_same_sizes,
 854:     bool coalesce_mask) {
 855:   TORCH_CHECK(src_in.is_sparse() && mask_in.is_sparse(),
 856:               "sparse_mask: expected both inputs to be sparse COO");
 857:   TORCH_CHECK(src_in.is_mps() && mask_in.is_mps(),
 858:               "sparse_mask: expected tensors to be on MPS device");
 859:   TORCH_CHECK(src_in.sparse_dim() == mask_in.sparse_dim(),
 860:               "sparse_mask: sparse_dim mismatch: ", src_in.sparse_dim(), " vs ", mask_in.sparse_dim());
 861:   if (require_same_sizes) {
 862:     TORCH_CHECK(src_in.sizes().equals(mask_in.sizes()),
 863:                 "sparse_mask: sizes must match exactly (no broadcasting)");
 864:   }
 865:   auto src  = src_in.coalesce();
 866:   auto mask = coalesce_mask ? mask_in.coalesce() : mask_in;
 867: 
 868:   const auto src_nnz = src._nnz();
 869:   const auto mask_nnz = mask._nnz();
 870:   const auto sd = src.sparse_dim();
 871:   result.sparse_resize_(mask.sizes(), mask.sparse_dim(), mask.dense_dim());
 872: 
 873:   auto commonDtype = at::result_type(src, mask);
 874:   TORCH_CHECK(canCast(commonDtype, result.scalar_type()),
 875:               "Can't convert result type ", commonDtype, " to output ", result.scalar_type());
 876: 
 877:   if (mask_nnz == 0) {
 878:     alias_into_sparse(
 879:         result,
 880:         mask._indices().narrow(1, 0, 0),
```
- L841: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L842: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L843: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L844: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L845: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L846: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L848: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L849: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L850: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L851: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L852: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L853: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L854: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L855: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L856: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L857: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L858: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L859: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L860: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L861: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L862: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L863: Declares function `exactly` as part of this file's callable surface. / 声明函数 `exactly`，作为本文件可调用接口的一部分。
- L864: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L865: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L866: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L868: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L869: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L870: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L871: Declares function `sparse_resize_` as part of this file's callable surface. / 声明函数 `sparse_resize_`，作为本文件可调用接口的一部分。
- L873: Declares function `result_type` as part of this file's callable surface. / 声明函数 `result_type`，作为本文件可调用接口的一部分。
- L874: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L875: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L877: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L878: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L879: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L880: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 881-920

```cpp
 881:         at::empty({0}, result.options().dtype(result.scalar_type())));
 882:     result._coalesced_(mask.is_coalesced());
 883:     return;
 884:   }
 885: 
 886:   TORCH_CHECK(sd > 0 || (src_nnz <= 1 && mask_nnz <= 1),
 887:               "sparse_mask: invalid sparse_dim or nnz");
 888: 
 889:   if (sd == 0) {
 890:     auto out_indices = mask._indices().narrow(1, 0, 1);
 891:     auto out_values = src_nnz
 892:       ? src._values().narrow(0, 0, 1).to(commonDtype)
 893:       : at::zeros({1}, at::device(result.device()).dtype(commonDtype));
 894:     alias_into_sparse(result, out_indices, out_values);
 895:     result._coalesced_(mask.is_coalesced());
 896:     return;
 897:   }
 898: 
 899:   auto mask_indices = mask._indices().contiguous();
 900:   auto src_values = src._values().to(commonDtype).contiguous();
 901:   auto out_values = create_sparse_output_values(src_values, mask_nnz, commonDtype);
 902: 
 903:   if (src_nnz == 0) {
 904:     alias_into_sparse(result, mask_indices, out_values);
 905:     result._coalesced_(mask.is_coalesced());
 906:     return;
 907:   }
 908: 
 909:   auto mask_keys = flatten_indices(mask._indices().contiguous(), mask.sizes().slice(0, sd)).contiguous();
 910:   auto src_keys  = flatten_indices(src._indices().contiguous(), src.sizes().slice(0, sd)).contiguous();
 911: 
 912:   const auto A_is_src = (src_nnz <= mask_nnz);
 913:   const auto lenA = A_is_src ? src_nnz  : mask_nnz;
 914:   const auto lenB = A_is_src ? mask_nnz : src_nnz;
 915:   auto A_keys = A_is_src ? src_keys  : mask_keys;
 916:   auto B_keys = A_is_src ? mask_keys : src_keys;
 917: 
 918:   auto [outA_idx, outB_idx, M] = mps_intersect_binary_search(
 919:       A_keys, B_keys, lenA, lenB, A_is_src);
 920: 
```
- L881: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L882: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L883: Returns from the current function without producing a value. / 从当前函数直接返回，不产生返回值。
- L884: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L886: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L887: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L889: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L890: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L891: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L892: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L893: Declares function `zeros` as part of this file's callable surface. / 声明函数 `zeros`，作为本文件可调用接口的一部分。
- L894: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L895: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L896: Returns from the current function without producing a value. / 从当前函数直接返回，不产生返回值。
- L897: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L899: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L900: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L901: Declares function `create_sparse_output_values` as part of this file's callable surface. / 声明函数 `create_sparse_output_values`，作为本文件可调用接口的一部分。
- L903: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L904: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L905: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L906: Returns from the current function without producing a value. / 从当前函数直接返回，不产生返回值。
- L907: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L909: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L910: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L912: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L913: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L914: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L915: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L916: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L918: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L919: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 921-960

```cpp
 921:   if (M > 0) {
 922:     auto src_match = outA_idx.narrow(0, 0, M);
 923:     auto mask_match = outB_idx.narrow(0, 0, M);
 924: 
 925:     auto src_rows = src_values.index_select(0, src_match);
 926:     if (accumulate_matches) {
 927:       out_values.index_add_(0, mask_match, src_rows);
 928:     } else {
 929:       out_values.index_copy_(0, mask_match, src_rows);
 930:     }
 931:   }
 932: 
 933:   alias_into_sparse(result, mask_indices, out_values);
 934:   result._coalesced_(mask.is_coalesced());
 935: }
 936: 
 937: static void sparse_mask_projection_out_mps_kernel(
 938:     Tensor& result,
 939:     const Tensor& lhs,
 940:     const Tensor& rhs,
 941:     const OptTensor& /*x_hash_opt*/,
 942:     bool accumulate_matches) {
 943: 
 944:   TORCH_CHECK(lhs.is_sparse() && rhs.is_sparse(), "sparse_mask_projection: expected sparse COO");
 945:   TORCH_CHECK(lhs.is_mps() && rhs.is_mps(), "sparse_mask_projection: expected MPS tensors");
 946:   TORCH_CHECK(lhs.sparse_dim() == rhs.sparse_dim(), "sparse_dim mismatch");
 947: 
 948:   auto lhs_c = lhs.coalesce();
 949:   auto rhs_c = rhs.coalesce();
 950: 
 951:   const auto sd = lhs_c.sparse_dim();
 952:   const auto lhs_nnz = lhs_c._nnz();
 953:   const auto rhs_nnz = rhs_c._nnz();
 954: 
 955:   auto commonDtype = at::result_type(lhs_c, rhs_c);
 956:   TORCH_CHECK(canCast(commonDtype, result.scalar_type()),
 957:               "Can't convert ", commonDtype, " to output ", result.scalar_type());
 958: 
 959:   result.sparse_resize_(lhs.sizes(), lhs.sparse_dim(), lhs.dense_dim());
 960: 
```
- L921: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L922: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L923: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L925: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L926: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L927: Declares function `index_add_` as part of this file's callable surface. / 声明函数 `index_add_`，作为本文件可调用接口的一部分。
- L928: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L929: Declares function `index_copy_` as part of this file's callable surface. / 声明函数 `index_copy_`，作为本文件可调用接口的一部分。
- L930: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L931: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L933: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L934: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L935: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L937: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L938: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L939: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L940: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L941: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L942: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L944: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L945: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L946: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L948: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L949: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L951: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L952: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L953: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L955: Declares function `result_type` as part of this file's callable surface. / 声明函数 `result_type`，作为本文件可调用接口的一部分。
- L956: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L957: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L959: Declares function `sparse_resize_` as part of this file's callable surface. / 声明函数 `sparse_resize_`，作为本文件可调用接口的一部分。

### Lines 961-1000

```cpp
 961:   auto lhs_indices = lhs_c._indices().contiguous();
 962:   auto rhs_values  = rhs_c._values().to(commonDtype).contiguous();
 963:   auto out_values = create_sparse_output_values(rhs_values, lhs_nnz, commonDtype);
 964: 
 965:   if (lhs_nnz > 0 && rhs_nnz > 0) {
 966:     auto lhs_keys = flatten_indices(lhs_indices, lhs_c.sizes().slice(0, sd)).contiguous();
 967:     auto rhs_keys = flatten_indices(rhs_c._indices().contiguous(), rhs_c.sizes().slice(0, sd)).contiguous();
 968: 
 969:     const auto A_is_lhs = (lhs_nnz <= rhs_nnz);
 970:     const auto lenA = A_is_lhs ? lhs_nnz : rhs_nnz;
 971:     const auto lenB = A_is_lhs ? rhs_nnz : lhs_nnz;
 972:     auto A_keys = A_is_lhs ? lhs_keys : rhs_keys;
 973:     auto B_keys = A_is_lhs ? rhs_keys : lhs_keys;
 974: 
 975:     auto [outA_idx, outB_idx, M] = mps_intersect_binary_search(
 976:         A_keys, B_keys, lenA, lenB, A_is_lhs);
 977: 
 978:     if (M > 0) {
 979:       auto idx_in_A = outA_idx.narrow(0, 0, M);
 980:       auto idx_in_B = outB_idx.narrow(0, 0, M);
 981:       auto idx_in_lhs = A_is_lhs ? idx_in_A : idx_in_B;
 982:       auto idx_in_rhs = A_is_lhs ? idx_in_B : idx_in_A;
 983: 
 984:       const auto view_cols = rhs_values.numel() / std::max<int64_t>(rhs_nnz, 1);
 985:       auto rhs_rows = rhs_values.index_select(0, idx_in_rhs).contiguous();
 986:       auto rhs_rows_2d = rhs_rows.view({M, view_cols});
 987:       auto out_2d = out_values.view({lhs_nnz, view_cols});
 988: 
 989:       if (accumulate_matches) {
 990:         out_2d.index_add_(0, idx_in_lhs, rhs_rows_2d);
 991:       } else {
 992:         out_2d.index_copy_(0, idx_in_lhs, rhs_rows_2d);
 993:       }
 994:     }
 995:   }
 996: 
 997:   alias_into_sparse(result, lhs._indices(), out_values);
 998:   result._coalesced_(lhs.is_coalesced());
 999: }
1000: 
```
- L961: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L962: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L963: Declares function `create_sparse_output_values` as part of this file's callable surface. / 声明函数 `create_sparse_output_values`，作为本文件可调用接口的一部分。
- L965: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L966: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L967: Declares function `flatten_indices` as part of this file's callable surface. / 声明函数 `flatten_indices`，作为本文件可调用接口的一部分。
- L969: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L970: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L971: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L972: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L973: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L975: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L976: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L978: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L979: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L980: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L981: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L982: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L984: Declares function `numel` as part of this file's callable surface. / 声明函数 `numel`，作为本文件可调用接口的一部分。
- L985: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L986: Declares function `view` as part of this file's callable surface. / 声明函数 `view`，作为本文件可调用接口的一部分。
- L987: Declares function `view` as part of this file's callable surface. / 声明函数 `view`，作为本文件可调用接口的一部分。
- L989: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L990: Declares function `index_add_` as part of this file's callable surface. / 声明函数 `index_add_`，作为本文件可调用接口的一部分。
- L991: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L992: Declares function `index_copy_` as part of this file's callable surface. / 声明函数 `index_copy_`，作为本文件可调用接口的一部分。
- L993: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L994: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L995: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L997: Declares function `alias_into_sparse` as part of this file's callable surface. / 声明函数 `alias_into_sparse`，作为本文件可调用接口的一部分。
- L998: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L999: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 1001-1040

```cpp
1001: static void sparse_mask_intersection_out_mps_kernel(
1002:     Tensor& result,
1003:     const Tensor& lhs,
1004:     const Tensor& rhs,
1005:     const OptTensor& = std::nullopt) {
1006:   sparse_mask_apply_out_mps_kernel(
1007:       result,
1008:       /*src_in=*/lhs,
1009:       /*mask_in=*/rhs,
1010:       /*accumulate_matches=*/false,
1011:       /*require_same_sizes=*/false,
1012:       /*coalesce_mask=*/false);
1013: }
1014: 
1015: Tensor sparse_sparse_matmul_mps(const Tensor& mat1_, const Tensor& mat2_) {
1016:   TORCH_CHECK(mat1_.is_sparse() && mat2_.is_sparse(),
1017:               "sparse_sparse_matmul_mps: both inputs must be sparse COO tensors");
1018:   TORCH_CHECK(mat1_.is_mps() && mat2_.is_mps(),
1019:               "sparse_sparse_matmul_mps: both inputs must be on MPS device");
1020:   TORCH_CHECK(mat1_.dim() == 2 && mat2_.dim() == 2,
1021:               "sparse_sparse_matmul_mps: both inputs must be 2D matrices");
1022:   TORCH_CHECK(mat1_.dense_dim() == 0 && mat2_.dense_dim() == 0,
1023:               "sparse_sparse_matmul_mps: only scalar values supported (dense_dim == 0)");
1024:   TORCH_CHECK(mat1_.size(1) == mat2_.size(0),
1025:               "mat1 and mat2 shapes cannot be multiplied (", mat1_.size(0), "x", mat1_.size(1), " and ", mat2_.size(0), "x", mat2_.size(1), ")");
1026:   TORCH_CHECK(mat1_.scalar_type() == mat2_.scalar_type(),
1027:               "sparse_sparse_matmul_mps: mat1 dtype ", mat1_.scalar_type(),
1028:               " does not match mat2 dtype ", mat2_.scalar_type());
1029: 
1030:   const auto device = mat1_.device();
1031: 
1032:   auto A = mat1_.coalesce();
1033:   auto B = mat2_.coalesce();
1034: 
1035:   const auto I = A.size(0);
1036:   const auto K = A.size(1);
1037:   const auto N = B.size(1);
1038: 
1039:   const auto nnzA = A._nnz();
1040:   const auto nnzB = B._nnz();
```
- L1001: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1002: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1003: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1004: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1005: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1006: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1007: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1008: Documents the nearby logic: src_in=*/lhs, / 说明附近逻辑的作用：src_in=*/lhs,
- L1009: Documents the nearby logic: mask_in=*/rhs, / 说明附近逻辑的作用：mask_in=*/rhs,
- L1010: Documents the nearby logic: accumulate_matches=*/false, / 说明附近逻辑的作用：accumulate_matches=*/false,
- L1011: Documents the nearby logic: require_same_sizes=*/false, / 说明附近逻辑的作用：require_same_sizes=*/false,
- L1012: Documents the nearby logic: coalesce_mask=*/false); / 说明附近逻辑的作用：coalesce_mask=*/false);
- L1013: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1015: Defines function `sparse_sparse_matmul_mps` and begins its implementation body. / 定义函数 `sparse_sparse_matmul_mps`，并开始其实现体。
- L1016: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1017: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1018: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1019: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1020: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1021: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1022: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1023: Declares function `supported` as part of this file's callable surface. / 声明函数 `supported`，作为本文件可调用接口的一部分。
- L1024: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1025: Declares function `multiplied` as part of this file's callable surface. / 声明函数 `multiplied`，作为本文件可调用接口的一部分。
- L1026: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1027: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1028: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L1030: Declares function `device` as part of this file's callable surface. / 声明函数 `device`，作为本文件可调用接口的一部分。
- L1032: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L1033: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L1035: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1036: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1037: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1039: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L1040: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。

### Lines 1041-1080

```cpp
1041: 
1042:   // Early empty result, return an empty, coalesced tensor
1043:   if (I == 0 || N == 0 || K == 0 || nnzA == 0 || nnzB == 0) {
1044:     auto empty_idx = at::empty({2, 0}, at::device(device).dtype(at::kLong));
1045:     auto empty_val = at::empty({0}, at::device(device).dtype(mat1_.scalar_type()));
1046:     auto out = _sparse_coo_tensor_unsafe(empty_idx, empty_val, {I, N}, mat1_.options());
1047:     out._coalesced_(true);
1048:     return out;
1049:   }
1050: 
1051:   const auto computeDtype = at::result_type(mat1_, mat2_);
1052: 
1053:   auto A_idx = A._indices().contiguous();
1054:   auto A_val = A._values().to(computeDtype).contiguous();
1055:   auto A_i = A_idx.select(0, 0).contiguous();
1056:   auto A_k = A_idx.select(0, 1).contiguous();
1057: 
1058:   auto B_idx = B._indices().contiguous();
1059:   auto B_val = B._values().to(computeDtype).contiguous();
1060:   auto B_k = B_idx.select(0, 0).contiguous();
1061:   auto B_j = B_idx.select(0, 1).contiguous();
1062: 
1063:   // csr-style row pointers for B by k (the shared dimension)
1064:   Tensor row_ptr_B;
1065:   {
1066:     auto batch_ptr = at::tensor({0LL, nnzB}, at::device(device).dtype(at::kLong));
1067:     row_ptr_B = at::empty({K + 1}, at::device(device).dtype(at::kLong));
1068:     build_row_ptr_per_batch_mps(B_k, batch_ptr, /*B=*/1, /*I=*/K, row_ptr_B);
1069:   }
1070: 
1071:   auto row_ptr_B_lo = row_ptr_B.narrow(0, 0, K);
1072:   auto row_ptr_B_hi = row_ptr_B.narrow(0, 1, K);
1073:   auto deg_B = row_ptr_B_hi.sub(row_ptr_B_lo);
1074: 
1075:   auto counts = deg_B.index_select(0, A_k);
1076: 
1077:   const int64_t P = counts.sum().item<int64_t>();
1078:   if (P == 0) {
1079:     auto empty_idx = at::empty({2, 0}, at::device(device).dtype(at::kLong));
1080:     auto empty_val = at::empty({0}, at::device(device).dtype(mat1_.scalar_type()));
```
- L1042: Documents the nearby logic: Early empty result, return an empty, coalesced tensor / 说明附近逻辑的作用：Early empty result, return an empty, coalesced tensor
- L1043: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1044: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1045: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1046: Declares function `_sparse_coo_tensor_unsafe` as part of this file's callable surface. / 声明函数 `_sparse_coo_tensor_unsafe`，作为本文件可调用接口的一部分。
- L1047: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L1048: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1049: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1051: Declares function `result_type` as part of this file's callable surface. / 声明函数 `result_type`，作为本文件可调用接口的一部分。
- L1053: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L1054: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1055: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L1056: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L1058: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L1059: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1060: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L1061: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L1063: Documents the nearby logic: csr-style row pointers for B by k (the shared dimension) / 说明附近逻辑的作用：csr-style row pointers for B by k (the shared dimension)
- L1064: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1065: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1066: Declares function `tensor` as part of this file's callable surface. / 声明函数 `tensor`，作为本文件可调用接口的一部分。
- L1067: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1068: Declares function `build_row_ptr_per_batch_mps` as part of this file's callable surface. / 声明函数 `build_row_ptr_per_batch_mps`，作为本文件可调用接口的一部分。
- L1069: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1071: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L1072: Declares function `narrow` as part of this file's callable surface. / 声明函数 `narrow`，作为本文件可调用接口的一部分。
- L1073: Declares function `sub` as part of this file's callable surface. / 声明函数 `sub`，作为本文件可调用接口的一部分。
- L1075: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1077: Declares function `sum` as part of this file's callable surface. / 声明函数 `sum`，作为本文件可调用接口的一部分。
- L1078: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1079: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1080: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。

### Lines 1081-1120

```cpp
1081:     auto out = _sparse_coo_tensor_unsafe(empty_idx, empty_val, {I, N}, mat1_.options());
1082:     out._coalesced_(true);
1083:     return out;
1084:   }
1085: 
1086:   auto group_ids = repeat_interleave_mps(counts);
1087: 
1088:   // exclusive cumsum of counts
1089:   auto offsets = cumsum(counts, /*dim=*/0).sub(counts);
1090:   auto offsets_gather = offsets.index_select(0, group_ids);
1091:   auto within = at::arange(P, at::device(device).dtype(at::kLong)).sub(offsets_gather);
1092: 
1093:   // Map each output element to its source B row and position
1094:   auto k_per_out = A_k.index_select(0, group_ids);
1095:   auto start_in_B = row_ptr_B.index_select(0, k_per_out);
1096:   auto seg_index = start_in_B.add(within);
1097: 
1098:   // Assemble candidate coo pairs and values
1099:   auto i_out = A_i.index_select(0, group_ids).contiguous();
1100:   auto j_out = B_j.index_select(0, seg_index).contiguous();
1101:   auto vA_out = A_val.index_select(0, group_ids).contiguous();
1102:   auto vB_out = B_val.index_select(0, seg_index).contiguous();
1103:   auto v_out = vA_out.mul(vB_out);
1104: 
1105:   // build (2, P) indices
1106:   auto out_indices = at::empty(
1107:       {2, P},
1108:       at::device(device).dtype(at::kLong).memory_format(c10::MemoryFormat::Contiguous));
1109:   out_indices.select(0, 0).copy_(i_out);
1110:   out_indices.select(0, 1).copy_(j_out);
1111: 
1112:   auto result = _sparse_coo_tensor_unsafe(
1113:       out_indices, v_out, {I, N}, mat1_.options().dtype(computeDtype));
1114: 
1115:   result = result.coalesce();
1116: 
1117:   if (result.scalar_type() != mat1_.scalar_type()) {
1118:     auto cast_vals = result._values().to(mat1_.scalar_type());
1119:     auto out = _sparse_coo_tensor_unsafe(result._indices(), cast_vals, {I, N}, mat1_.options());
1120:     out._coalesced_(true);
```
- L1081: Declares function `_sparse_coo_tensor_unsafe` as part of this file's callable surface. / 声明函数 `_sparse_coo_tensor_unsafe`，作为本文件可调用接口的一部分。
- L1082: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L1083: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1084: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1086: Declares function `repeat_interleave_mps` as part of this file's callable surface. / 声明函数 `repeat_interleave_mps`，作为本文件可调用接口的一部分。
- L1088: Documents the nearby logic: exclusive cumsum of counts / 说明附近逻辑的作用：exclusive cumsum of counts
- L1089: Declares function `cumsum` as part of this file's callable surface. / 声明函数 `cumsum`，作为本文件可调用接口的一部分。
- L1090: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1091: Declares function `arange` as part of this file's callable surface. / 声明函数 `arange`，作为本文件可调用接口的一部分。
- L1093: Documents the nearby logic: Map each output element to its source B row and position / 说明附近逻辑的作用：Map each output element to its source B row and position
- L1094: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1095: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1096: Declares function `add` as part of this file's callable surface. / 声明函数 `add`，作为本文件可调用接口的一部分。
- L1098: Documents the nearby logic: Assemble candidate coo pairs and values / 说明附近逻辑的作用：Assemble candidate coo pairs and values
- L1099: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1100: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1101: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1102: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1103: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L1105: Documents the nearby logic: build (2, P) indices / 说明附近逻辑的作用：build (2, P) indices
- L1106: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1107: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1108: Declares function `device` as part of this file's callable surface. / 声明函数 `device`，作为本文件可调用接口的一部分。
- L1109: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L1110: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L1112: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1113: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L1115: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L1117: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1118: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1119: Declares function `_sparse_coo_tensor_unsafe` as part of this file's callable surface. / 声明函数 `_sparse_coo_tensor_unsafe`，作为本文件可调用接口的一部分。
- L1120: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。

### Lines 1121-1160

```cpp
1121:     return out;
1122:   }
1123:   return result;
1124: }
1125: 
1126: 
1127: Tensor _sparse_sum_backward_mps(const Tensor& grad_, const SparseTensor& input_, IntArrayRef dims_to_sum) {
1128:   TORCH_CHECK(grad_.is_mps(), "_sparse_sum_backward_mps: expected 'grad_' to be MPS tensor, but got ", grad_.device());
1129:   TORCH_CHECK(input_.is_mps(), "_sparse_sum_backward_mps: expected 'input_' to be MPS tensor, but got ", input_.device());
1130: 
1131:   if (((grad_.is_sparse() || sparse_csr::is_sparse_compressed(grad_)) && !grad_._nnz()) || !grad_.numel()) {
1132:     return at::zeros_like(input_);
1133:   }
1134: 
1135:   auto input = input_.coalesce();
1136:   const auto input_dim = input.dim();
1137: 
1138:   auto dims_to_sum_v = dims_to_sum.vec();
1139:   maybe_wrap_dims(dims_to_sum_v, input_dim);
1140:   std::vector<bool> dims_to_sum_b(static_cast<size_t>(input_dim), false);
1141:   for (auto d : dims_to_sum_v) {
1142:     dims_to_sum_b[static_cast<size_t>(d)] = true;
1143:   }
1144: 
1145:   auto input_indices = input._indices();
1146:   auto input_values = input._values();
1147:   auto input_sizes = input.sizes();
1148:   const auto input_sparse_dim = input.sparse_dim();
1149:   const auto input_dense_dim = input.dense_dim();
1150:   const auto input_nnz = input._nnz();
1151: 
1152:   int64_t sparse_dims_to_sum_size = 0;
1153:   std::vector<int64_t> sparse_dims_to_keep_v;
1154:   std::vector<int64_t> dense_dims_to_sum_v;
1155:   sparse_dims_to_keep_v.reserve(input_sparse_dim);
1156:   dense_dims_to_sum_v.reserve(input_dense_dim);
1157: 
1158:   for (auto d = 0; d < input_dim; d++) {
1159:     if (dims_to_sum_b[static_cast<size_t>(d)]) {
1160:       if (d < input_sparse_dim) {
```
- L1121: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1122: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1123: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1124: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1127: Defines function `_sparse_sum_backward_mps` and begins its implementation body. / 定义函数 `_sparse_sum_backward_mps`，并开始其实现体。
- L1128: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1129: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1131: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1132: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1133: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1135: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L1136: Declares function `dim` as part of this file's callable surface. / 声明函数 `dim`，作为本文件可调用接口的一部分。
- L1138: Declares function `vec` as part of this file's callable surface. / 声明函数 `vec`，作为本文件可调用接口的一部分。
- L1139: Declares function `maybe_wrap_dims` as part of this file's callable surface. / 声明函数 `maybe_wrap_dims`，作为本文件可调用接口的一部分。
- L1140: Declares function `dims_to_sum_b` as part of this file's callable surface. / 声明函数 `dims_to_sum_b`，作为本文件可调用接口的一部分。
- L1141: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1142: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1143: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1145: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L1146: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1147: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1148: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L1149: Declares function `dense_dim` as part of this file's callable surface. / 声明函数 `dense_dim`，作为本文件可调用接口的一部分。
- L1150: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L1152: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1153: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1154: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1155: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L1156: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L1158: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1159: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1160: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。

### Lines 1161-1200

```cpp
1161:         sparse_dims_to_sum_size++;
1162:       } else {
1163:         dense_dims_to_sum_v.emplace_back(d + 1 - input_sparse_dim);
1164:       }
1165:     } else if (d < input_sparse_dim) {
1166:       sparse_dims_to_keep_v.emplace_back(d);
1167:     }
1168:   }
1169: 
1170:   const auto sum_all_sparse_dim = (input_sparse_dim == sparse_dims_to_sum_size);
1171:   const auto sum_dense_dim = !dense_dims_to_sum_v.empty();
1172:   const auto sum_sparse_dim = (sparse_dims_to_sum_size > 0);
1173: 
1174:   if (sum_all_sparse_dim) {
1175:     TORCH_CHECK(!grad_.is_sparse(), "_sparse_sum_backward_mps: expected grad Tensor to be dense since all sparse dims are summed");
1176: 
1177:     auto grad_input_values = grad_.contiguous();
1178:     auto expand_size = input_values.sizes().vec();
1179: 
1180:     if (sum_dense_dim) {
1181:       auto dense_expand_size = std::vector<int64_t>(expand_size.begin() + 1, expand_size.end());
1182:       for (auto d : dense_dims_to_sum_v) {
1183:         grad_input_values = grad_input_values.unsqueeze(d - 1);
1184:       }
1185:       grad_input_values = grad_input_values.expand(dense_expand_size);
1186:     }
1187:     grad_input_values = grad_input_values.expand(expand_size).clone(at::MemoryFormat::Contiguous);
1188: 
1189:     return at::_sparse_coo_tensor_with_dims_and_tensors(
1190:         input_sparse_dim, input_dense_dim, input_sizes,
1191:         input_indices.clone(at::MemoryFormat::Contiguous),
1192:         grad_input_values, input.options().dtype(grad_.dtype()));
1193:   }
1194: 
1195:   TORCH_CHECK(grad_.is_sparse(), "_sparse_sum_backward_mps: expected 'grad_' Tensor to be sparse when not all sparse dims are summed");
1196:   auto grad = grad_.coalesce();
1197:   auto grad_indices = grad._indices();
1198:   auto grad_values = grad._values();
1199:   const auto grad_sparse_dim = grad.sparse_dim();
1200:   const auto grad_nnz = grad._nnz();
```
- L1161: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1162: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1163: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L1164: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1165: Defines function `if` and begins its implementation body. / 定义函数 `if`，并开始其实现体。
- L1166: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L1167: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1168: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1170: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1171: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1172: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1174: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1175: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1177: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L1178: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1180: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1181: Declares function `begin` as part of this file's callable surface. / 声明函数 `begin`，作为本文件可调用接口的一部分。
- L1182: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1183: Declares function `unsqueeze` as part of this file's callable surface. / 声明函数 `unsqueeze`，作为本文件可调用接口的一部分。
- L1184: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1185: Declares function `expand` as part of this file's callable surface. / 声明函数 `expand`，作为本文件可调用接口的一部分。
- L1186: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1187: Declares function `expand` as part of this file's callable surface. / 声明函数 `expand`，作为本文件可调用接口的一部分。
- L1189: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1190: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1191: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1192: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L1193: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1195: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1196: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L1197: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L1198: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1199: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L1200: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。

### Lines 1201-1240

```cpp
1201: 
1202:   auto grad_values_expand = grad_values;
1203:   if (sum_dense_dim) {
1204:     auto expand_size = input_values.sizes().vec();
1205:     expand_size[0] = grad_values.size(0);
1206:     for (auto d : dense_dims_to_sum_v) {
1207:       grad_values_expand = grad_values_expand.unsqueeze(d);
1208:     }
1209:     grad_values_expand = grad_values_expand.expand(expand_size).clone(at::MemoryFormat::Contiguous);
1210:   }
1211: 
1212:   Tensor grad_input_values;
1213:   if (!sum_sparse_dim) {
1214:     grad_input_values = grad_values_expand;
1215:   } else {
1216:     auto grad_keys = at::sparse::flatten_indices(
1217:         grad_indices.contiguous(),
1218:         grad.sizes().slice(0, grad_sparse_dim)).to(at::TensorOptions().dtype(at::kInt).memory_format(at::MemoryFormat::Contiguous));
1219: 
1220:     std::vector<int64_t> sizes_keep;
1221:     sizes_keep.reserve(sparse_dims_to_keep_v.size());
1222:     for (auto d : sparse_dims_to_keep_v) {
1223:       sizes_keep.push_back(input_sizes[d]);
1224:     }
1225: 
1226:     auto keep_idx = at::tensor(sparse_dims_to_keep_v, at::device(input_indices.device()).dtype(at::kLong));
1227:     auto input_indices_keep = input_indices.index_select(0, keep_idx).contiguous();
1228:     auto input_keys = at::sparse::flatten_indices(
1229:         input_indices_keep, IntArrayRef(sizes_keep)).to(at::kInt).contiguous();
1230: 
1231:     auto idx_in_grad = searchsorted_mps(grad_keys, input_keys);
1232:     auto idx_in_grad_clamped = idx_in_grad.clamp_max(grad_nnz - 1);
1233:     auto matched_mask = grad_keys.index_select(0, idx_in_grad_clamped).eq(input_keys);
1234: 
1235:     grad_input_values = grad_values_expand.index_select(0, idx_in_grad_clamped);
1236: 
1237:     if (grad_input_values.dim() > 1) {
1238:       auto reshape_mask_size = std::vector<int64_t>(grad_input_values.dim(), 1);
1239:       reshape_mask_size[0] = input_nnz;
1240:       matched_mask = matched_mask.view(reshape_mask_size);
```
- L1202: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1203: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1204: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1205: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1206: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1207: Declares function `unsqueeze` as part of this file's callable surface. / 声明函数 `unsqueeze`，作为本文件可调用接口的一部分。
- L1208: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1209: Declares function `expand` as part of this file's callable surface. / 声明函数 `expand`，作为本文件可调用接口的一部分。
- L1210: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1212: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1213: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1214: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1215: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1216: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1217: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1218: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1220: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1221: Preallocates container capacity to reduce later reallocations. / 预先分配容器容量，以减少后续重新分配。
- L1222: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1223: Appends a new element to the end of a sequential container. / 向顺序容器尾部追加一个新元素。
- L1224: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1226: Declares function `tensor` as part of this file's callable surface. / 声明函数 `tensor`，作为本文件可调用接口的一部分。
- L1227: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1228: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1229: Declares function `IntArrayRef` as part of this file's callable surface. / 声明函数 `IntArrayRef`，作为本文件可调用接口的一部分。
- L1231: Declares function `searchsorted_mps` as part of this file's callable surface. / 声明函数 `searchsorted_mps`，作为本文件可调用接口的一部分。
- L1232: Declares function `clamp_max` as part of this file's callable surface. / 声明函数 `clamp_max`，作为本文件可调用接口的一部分。
- L1233: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1235: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1237: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1238: Declares function `dim` as part of this file's callable surface. / 声明函数 `dim`，作为本文件可调用接口的一部分。
- L1239: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1240: Declares function `view` as part of this file's callable surface. / 声明函数 `view`，作为本文件可调用接口的一部分。

### Lines 1241-1280

```cpp
1241:     }
1242:     grad_input_values.masked_fill_(matched_mask.logical_not(), 0);
1243:   }
1244: 
1245:   return at::_sparse_coo_tensor_with_dims_and_tensors(
1246:       input_sparse_dim, input_dense_dim, input_sizes,
1247:       input_indices,
1248:       grad_input_values, grad.options());
1249: }
1250: 
1251: Tensor index_select_sparse_mps(const Tensor& self_, int64_t dim, const Tensor& index) {
1252:   TORCH_CHECK(self_.is_sparse(), "index_select_sparse_mps: expected a sparse COO tensor");
1253:   TORCH_CHECK(self_.is_mps(), "index_select_sparse_mps: expected 'self' to be on MPS, got ", self_.device());
1254:   TORCH_CHECK(
1255:       index.dim() == 1 && index.dtype() == at::kLong && index.layout() == at::kStrided,
1256:       "index_select() argument index must be 1-D strided (non-sparse) long-tensor.");
1257: 
1258:   const auto ndim = self_.dim();
1259:   TORCH_CHECK_INDEX(ndim, "index_select() cannot be applied to a 0-dim tensor.");
1260:   dim = maybe_wrap_dim(dim, ndim);
1261: 
1262:   std::vector<int64_t> out_sizes = self_.sizes().vec();
1263:   const auto index_len = index.numel();
1264:   out_sizes[dim] = index_len;
1265: 
1266:   const auto sd = self_.sparse_dim();
1267:   const auto dd = self_.dense_dim();
1268:   const auto nnz = self_._nnz();
1269: 
1270:   // Indexing into dense dimensions only affects values
1271:   if (dim >= sd) {
1272:     const int64_t values_dim = dim - sd + 1;
1273:     TORCH_CHECK(values_dim >= 1 && values_dim < (1 + dd),
1274:                 "index_select_sparse_mps: invalid dense dimension to select: ", dim);
1275: 
1276:     Tensor out_values = self_._values().index_select(values_dim, index);
1277:     return _sparse_coo_tensor_unsafe(self_._indices(), out_values, out_sizes, self_.options());
1278:   }
1279: 
1280:   // Selecting along a sparse dimension
```
- L1241: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1242: Declares function `masked_fill_` as part of this file's callable surface. / 声明函数 `masked_fill_`，作为本文件可调用接口的一部分。
- L1243: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1245: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1246: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1247: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1248: Declares function `options` as part of this file's callable surface. / 声明函数 `options`，作为本文件可调用接口的一部分。
- L1249: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1251: Defines function `index_select_sparse_mps` and begins its implementation body. / 定义函数 `index_select_sparse_mps`，并开始其实现体。
- L1252: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1253: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1254: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1255: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1256: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1258: Declares function `dim` as part of this file's callable surface. / 声明函数 `dim`，作为本文件可调用接口的一部分。
- L1259: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1260: Declares function `maybe_wrap_dim` as part of this file's callable surface. / 声明函数 `maybe_wrap_dim`，作为本文件可调用接口的一部分。
- L1262: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1263: Declares function `numel` as part of this file's callable surface. / 声明函数 `numel`，作为本文件可调用接口的一部分。
- L1264: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1266: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L1267: Declares function `dense_dim` as part of this file's callable surface. / 声明函数 `dense_dim`，作为本文件可调用接口的一部分。
- L1268: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L1270: Documents the nearby logic: Indexing into dense dimensions only affects values / 说明附近逻辑的作用：Indexing into dense dimensions only affects values
- L1271: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1272: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1273: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1274: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1276: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1277: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1278: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1280: Documents the nearby logic: Selecting along a sparse dimension / 说明附近逻辑的作用：Selecting along a sparse dimension

### Lines 1281-1320

```cpp
1281:   const auto I = self_.size(dim);
1282:   TORCH_CHECK(I >= 0, "index_select_sparse_mps: invalid size for selected dim");
1283: 
1284:   if (I == 0) {
1285:     TORCH_CHECK(
1286:         index_len == 0,
1287:         "index_select(): index has to be empty when selecting from an empty dimension");
1288:     auto empty_idx = empty({sd, 0}, at::device(self_.device()).dtype(kLong));
1289:     auto tmpl_vals = self_._values();
1290:     std::vector<int64_t> val_sizes = tmpl_vals.sizes().vec();
1291:     val_sizes[0] = 0;
1292:     auto empty_vals = at::empty(val_sizes, tmpl_vals.options());
1293:     return _sparse_coo_tensor_unsafe(empty_idx, empty_vals, out_sizes, self_.options());
1294:   }
1295: 
1296:   if (index_len > 0) {
1297:     TORCH_CHECK(index.min().item<int64_t>() >= -I && index.max().item<int64_t>() < I,
1298:                 "index_select(): index out of bounds for dimension with size ", I);
1299:   }
1300:   // Normalize negative indices
1301:   auto nneg_index = remainder(index, I).contiguous();
1302: 
1303:   if (index_len == 0 || nnz == 0) {
1304:     auto empty_idx = empty({sd, 0}, at::device(self_.device()).dtype(kLong));
1305:     auto tmpl_vals = self_._values();
1306:     std::vector<int64_t> val_sizes = tmpl_vals.sizes().vec();
1307:     val_sizes[0] = 0;
1308:     auto empty_vals = empty(val_sizes, tmpl_vals.options());
1309:     return _sparse_coo_tensor_unsafe(empty_idx, empty_vals, out_sizes, self_.options());
1310:   }
1311: 
1312:   auto indices = self_._indices().contiguous();
1313:   auto values = self_._values().contiguous();
1314: 
1315:   auto dim_indices = indices.select(0, dim).contiguous();
1316: 
1317:   Tensor sorted_dim_indices, argsort_dim_indices;
1318:   if (dim == 0 && self_.is_coalesced()) {
1319:     sorted_dim_indices = dim_indices;
1320:     argsort_dim_indices = arange(nnz, at::device(self_.device()).dtype(kLong));
```
- L1281: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L1282: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1284: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1285: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1286: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1287: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1288: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1289: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1290: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1291: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1292: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1293: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1294: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1296: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1297: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L1298: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1299: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1300: Documents the nearby logic: Normalize negative indices / 说明附近逻辑的作用：Normalize negative indices
- L1301: Declares function `remainder` as part of this file's callable surface. / 声明函数 `remainder`，作为本文件可调用接口的一部分。
- L1303: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1304: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1305: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1306: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1307: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1308: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1309: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1310: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1312: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L1313: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1315: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L1317: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1318: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1319: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1320: Declares function `arange` as part of this file's callable surface. / 声明函数 `arange`，作为本文件可调用接口的一部分。

### Lines 1321-1360

```cpp
1321:   } else {
1322:     std::tie(sorted_dim_indices, argsort_dim_indices) = dim_indices.sort();
1323:   }
1324: 
1325:   // Build row_ptr for lower/upper bound lookups
1326:   auto batch_ptr = tensor({0LL, nnz}, at::device(self_.device()).dtype(kLong));
1327:   auto row_ptr = empty({I + 1}, at::device(self_.device()).dtype(kLong));
1328:   build_row_ptr_per_batch_mps(sorted_dim_indices, batch_ptr, /*B=*/1, /*I=*/I, row_ptr);
1329: 
1330:   auto lower = row_ptr.index_select(0, nneg_index);
1331:   auto nneg_index_plus1 = nneg_index.add(1);
1332:   auto upper = row_ptr.index_select(0, nneg_index_plus1);
1333:   auto counts = upper.sub(lower);
1334: 
1335:   const int64_t M = counts.sum().item<int64_t>();
1336:   if (M == 0) {
1337:     auto empty_idx = empty({sd, 0}, at::device(self_.device()).dtype(kLong));
1338:     auto tmpl_vals = values;
1339:     std::vector<int64_t> val_sizes = tmpl_vals.sizes().vec();
1340:     val_sizes[0] = 0;
1341:     auto empty_vals = empty(val_sizes, tmpl_vals.options());
1342:     return _sparse_coo_tensor_unsafe(empty_idx, empty_vals, out_sizes, self_.options());
1343:   }
1344: 
1345:   // Expand counts into group ids for each output element
1346:   auto group_ids = repeat_interleave_mps(counts);
1347: 
1348:   auto offsets = cumsum(counts, /*dim=*/0).sub(counts);
1349:   auto offsets_gather = offsets.index_select(0, group_ids);
1350:   auto within = arange(M, at::device(self_.device()).dtype(kLong)).sub(offsets_gather);
1351: 
1352:   auto start_pos = lower.index_select(0, group_ids);
1353:   auto pos_sorted = start_pos.add(within);
1354:   auto selected_dim_indices = argsort_dim_indices.index_select(0, pos_sorted);
1355: 
1356:   // group_ids become the new indices for the selected dimension
1357:   auto res_dim_indices = group_ids.contiguous();
1358: 
1359:   auto out_indices = indices.index_select(1, selected_dim_indices).contiguous();
1360:   out_indices.select(0, dim).copy_(res_dim_indices);
```
- L1321: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1322: Declares function `tie` as part of this file's callable surface. / 声明函数 `tie`，作为本文件可调用接口的一部分。
- L1323: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1325: Documents the nearby logic: Build row_ptr for lower/upper bound lookups / 说明附近逻辑的作用：Build row_ptr for lower/upper bound lookups
- L1326: Declares function `tensor` as part of this file's callable surface. / 声明函数 `tensor`，作为本文件可调用接口的一部分。
- L1327: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1328: Declares function `build_row_ptr_per_batch_mps` as part of this file's callable surface. / 声明函数 `build_row_ptr_per_batch_mps`，作为本文件可调用接口的一部分。
- L1330: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1331: Declares function `add` as part of this file's callable surface. / 声明函数 `add`，作为本文件可调用接口的一部分。
- L1332: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1333: Declares function `sub` as part of this file's callable surface. / 声明函数 `sub`，作为本文件可调用接口的一部分。
- L1335: Declares function `sum` as part of this file's callable surface. / 声明函数 `sum`，作为本文件可调用接口的一部分。
- L1336: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1337: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1338: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1339: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1340: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1341: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1342: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1343: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1345: Documents the nearby logic: Expand counts into group ids for each output element / 说明附近逻辑的作用：Expand counts into group ids for each output element
- L1346: Declares function `repeat_interleave_mps` as part of this file's callable surface. / 声明函数 `repeat_interleave_mps`，作为本文件可调用接口的一部分。
- L1348: Declares function `cumsum` as part of this file's callable surface. / 声明函数 `cumsum`，作为本文件可调用接口的一部分。
- L1349: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1350: Declares function `arange` as part of this file's callable surface. / 声明函数 `arange`，作为本文件可调用接口的一部分。
- L1352: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1353: Declares function `add` as part of this file's callable surface. / 声明函数 `add`，作为本文件可调用接口的一部分。
- L1354: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1356: Documents the nearby logic: group_ids become the new indices for the selected dimension / 说明附近逻辑的作用：group_ids become the new indices for the selected dimension
- L1357: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L1359: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1360: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。

### Lines 1361-1400

```cpp
1361: 
1362:   auto out_values = values.index_select(0, selected_dim_indices);
1363: 
1364:   return _sparse_coo_tensor_unsafe(out_indices, out_values, out_sizes, self_.options());
1365: }
1366: 
1367: static Tensor softmax_sparse_mps_impl(
1368:     const Tensor& input_,
1369:     const int64_t dim_,
1370:     const bool half_to_float,
1371:     const bool logsoftmax
1372: ) {
1373:     auto stream = getCurrentMPSStream();
1374: 
1375:     auto input = input_.coalesce();
1376:     auto values = input._values();
1377:     auto indices = input._indices();
1378: 
1379:     if (half_to_float && values.scalar_type() == kHalf) {
1380:         values = values.to(kFloat);
1381:     }
1382: 
1383:     auto sparse_dim = input.sparse_dim();
1384:     auto dim = at::maybe_wrap_dim(dim_, input.dim());
1385:     auto nnz = input._nnz();
1386:     if (nnz == 0) return input_.clone();
1387: 
1388:     if (dim >= sparse_dim) {
1389:         auto output_values = logsoftmax
1390:             ? log_softmax(values, dim - sparse_dim + 1, values.scalar_type())
1391:             : softmax(values, dim - sparse_dim + 1, values.scalar_type());
1392:         return at::_sparse_coo_tensor_unsafe(indices, output_values, input.sizes(), input.options().dtype(output_values.scalar_type()))._coalesced_(true);
1393:     }
1394: 
1395:     auto sizes = input.sizes();
1396:     std::vector<int64_t> strides(sparse_dim, 1);
1397:     for (int i = sparse_dim - 2; i >= 0; i--) strides[i] = strides[i+1] * sizes[i+1];
1398:     strides[dim] = 0;
1399: 
1400:     auto pool_indices = at::zeros({nnz}, indices.options().dtype(kLong));
```
- L1362: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1364: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1365: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1367: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1368: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1369: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1370: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1371: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1372: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1373: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L1375: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L1376: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1377: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L1379: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1380: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L1381: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1383: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L1384: Declares function `maybe_wrap_dim` as part of this file's callable surface. / 声明函数 `maybe_wrap_dim`，作为本文件可调用接口的一部分。
- L1385: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L1386: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1388: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1389: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1390: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1391: Declares function `softmax` as part of this file's callable surface. / 声明函数 `softmax`，作为本文件可调用接口的一部分。
- L1392: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1393: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1395: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1396: Declares function `strides` as part of this file's callable surface. / 声明函数 `strides`，作为本文件可调用接口的一部分。
- L1397: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1398: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1400: Declares function `zeros` as part of this file's callable surface. / 声明函数 `zeros`，作为本文件可调用接口的一部分。

### Lines 1401-1440

```cpp
1401:     for (auto i = 0; i < sparse_dim; i++) {
1402:         if (i != dim) pool_indices.add_(indices.select(0, i), strides[i]);
1403:     }
1404: 
1405:     auto sort_order = at::argsort(pool_indices);
1406:     auto sorted_pool_indices = pool_indices.index_select(0, sort_order);
1407:     auto sorted_values = values.index_select(0, sort_order);
1408: 
1409:     auto mask = at::empty({nnz}, sorted_pool_indices.options().dtype(kInt));
1410:     auto nnz_u = static_cast<uint32_t>(nnz);
1411: 
1412:     dispatch_sync_with_rethrow(stream->queue(), ^() {
1413:         auto pso = lib.getPipelineStateForFunc("mark_segments");
1414:         auto enc = stream->commandEncoder();
1415:         [enc setComputePipelineState:pso];
1416:         mtl_setArgs(enc, sorted_pool_indices, mask);
1417: 
1418:         auto gridSize = MTLSizeMake(nnz, 1, 1);
1419:         auto threadGroupSize = MTLSizeMake(std::min<uint64_t>(nnz, pso.maxTotalThreadsPerThreadgroup), 1, 1);
1420:         [enc dispatchThreads:gridSize threadsPerThreadgroup:threadGroupSize];
1421:     });
1422: 
1423:     auto scan = at::cumsum(mask, 0, kInt);
1424: 
1425:     auto offsets = at::empty({nnz}, mask.options());
1426:     auto counts = at::empty({nnz}, mask.options());
1427: 
1428:     dispatch_sync_with_rethrow(stream->queue(), ^() {
1429:         auto pso = lib.getPipelineStateForFunc("compute_offsets_and_counts");
1430:         auto enc = stream->commandEncoder();
1431:         [enc setComputePipelineState:pso];
1432:         mtl_setArgs(enc, scan, offsets, counts, nnz_u);
1433: 
1434:         auto gridSize = MTLSizeMake(nnz, 1, 1);
1435:         auto threadGroupSize = MTLSizeMake(std::min<uint64_t>(nnz, pso.maxTotalThreadsPerThreadgroup), 1, 1);
1436:         [enc dispatchThreads:gridSize threadsPerThreadgroup:threadGroupSize];
1437:     });
1438: 
1439:     auto output_sorted = at::empty_like(sorted_values);
1440:     auto nvalues = static_cast<uint32_t>(values.numel() / nnz);
```
- L1401: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1402: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1403: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1405: Declares function `argsort` as part of this file's callable surface. / 声明函数 `argsort`，作为本文件可调用接口的一部分。
- L1406: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1407: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1409: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1410: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1412: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L1413: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L1414: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L1415: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1416: Declares function `mtl_setArgs` as part of this file's callable surface. / 声明函数 `mtl_setArgs`，作为本文件可调用接口的一部分。
- L1418: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1419: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1420: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1421: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1423: Declares function `cumsum` as part of this file's callable surface. / 声明函数 `cumsum`，作为本文件可调用接口的一部分。
- L1425: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1426: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1428: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L1429: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L1430: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L1431: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1432: Declares function `mtl_setArgs` as part of this file's callable surface. / 声明函数 `mtl_setArgs`，作为本文件可调用接口的一部分。
- L1434: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1435: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1436: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1437: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1439: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L1440: Declares function `numel` as part of this file's callable surface. / 声明函数 `numel`，作为本文件可调用接口的一部分。

### Lines 1441-1480

```cpp
1441: 
1442:     dispatch_sync_with_rethrow(stream->queue(), ^() {
1443:         auto pso = lib.getPipelineStateForFunc("softmax_sparse_forward_" + mps::scalarToMetalTypeString(values));
1444:         auto enc = stream->commandEncoder();
1445:         [enc setComputePipelineState:pso];
1446:         mtl_setArgs(enc, sorted_values, output_sorted, offsets, counts, scan,
1447:                     std::array<uint32_t, 2>{nnz_u, nvalues}, logsoftmax);
1448:         auto gridSize = MTLSizeMake(nnz, 1, 1);
1449:         auto threadGroupSize = MTLSizeMake(std::min<uint64_t>(nnz, pso.maxTotalThreadsPerThreadgroup), 1, 1);
1450:         [enc dispatchThreads:gridSize threadsPerThreadgroup:threadGroupSize];
1451:     });
1452: 
1453:     auto inv_sort_order = at::empty_like(sort_order);
1454:     inv_sort_order.scatter_(0, sort_order, at::arange(sort_order.size(0), sort_order.options()));
1455:     auto final_values = output_sorted.index_select(0, inv_sort_order);
1456: 
1457:     auto result = at::_sparse_coo_tensor_unsafe(
1458:         indices, final_values, input.sizes(), input.options().dtype(final_values.scalar_type())
1459:     );
1460:     return result._coalesced_(true);
1461: }
1462: 
1463: static Tensor softmax_backward_sparse_mps_impl(
1464:     const Tensor& grad_,
1465:     const Tensor& output_,
1466:     int64_t dim_,
1467:     const Tensor& input_,
1468:     bool logsoftmax
1469: ) {
1470:     auto stream = getCurrentMPSStream();
1471: 
1472:     auto output = output_.coalesce();
1473:     auto grad = grad_.sparse_mask(output);
1474: 
1475:     auto indices = output._indices();
1476:     auto output_values = output._values();
1477:     auto grad_values = grad._values();
1478:     auto indices_safe = indices.contiguous().clone();
1479: 
1480:     bool is_half = (output_values.scalar_type() == kHalf);
```
- L1442: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L1443: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L1444: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L1445: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1446: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1447: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1448: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1449: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1450: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1451: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1453: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L1454: Declares function `scatter_` as part of this file's callable surface. / 声明函数 `scatter_`，作为本文件可调用接口的一部分。
- L1455: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1457: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1458: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1459: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1460: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1461: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1463: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1464: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1465: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1466: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1467: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1468: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1469: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1470: Declares function `getCurrentMPSStream` as part of this file's callable surface. / 声明函数 `getCurrentMPSStream`，作为本文件可调用接口的一部分。
- L1472: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L1473: Declares function `sparse_mask` as part of this file's callable surface. / 声明函数 `sparse_mask`，作为本文件可调用接口的一部分。
- L1475: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L1476: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1477: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L1478: Declares function `contiguous` as part of this file's callable surface. / 声明函数 `contiguous`，作为本文件可调用接口的一部分。
- L1480: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。

### Lines 1481-1520

```cpp
1481:     if (is_half) {
1482:         output_values = output_values.to(kFloat);
1483:         grad_values = grad_values.to(kFloat);
1484:     }
1485: 
1486:     const auto sparse_dim = output.sparse_dim();
1487:     const auto dim = at::maybe_wrap_dim(dim_, output.dim());
1488:     const auto nnz = output._nnz();
1489: 
1490:     if (dim >= sparse_dim) {
1491:         const auto dense_dim_idx = dim - sparse_dim + 1;
1492:         Tensor grad_input_values;
1493:         if (logsoftmax) {
1494:             auto sum_grad = grad_values.sum({dense_dim_idx}, true);
1495:             grad_input_values = grad_values.sub(output_values.exp().mul_(sum_grad));
1496:         } else {
1497:             auto term = output_values.mul(grad_values);
1498:             auto sum_term = term.sum({dense_dim_idx}, true);
1499:             grad_input_values = output_values.mul(grad_values.sub(sum_term));
1500:         }
1501:         if (is_half) grad_input_values = grad_input_values.to(kHalf);
1502:         return at::_sparse_coo_tensor_unsafe(indices_safe, grad_input_values, output.sizes(), output.options().dtype(grad_input_values.scalar_type()))._coalesced_(true);
1503:     }
1504: 
1505:     if (nnz == 0) return at::empty_like(output_);
1506: 
1507:     auto pool_indices = at::zeros({nnz}, indices.options().dtype(kLong));
1508:     auto sizes = output.sizes();
1509:     std::vector<int64_t> strides(sparse_dim, 1);
1510:     for (int i = static_cast<int>(sparse_dim) - 2; i >= 0; --i) strides[i] = strides[i + 1] * sizes[i + 1];
1511: 
1512:     for (int64_t i = 0; i < sparse_dim; ++i) {
1513:         if (i == dim) continue;
1514:         if (strides[i] > 0) pool_indices.add_(indices.select(0, i), strides[i]);
1515:     }
1516: 
1517:     auto sort_order = at::argsort(pool_indices);
1518:     auto sorted_pool_indices = pool_indices.index_select(0, sort_order);
1519:     auto sorted_output_values = output_values.index_select(0, sort_order);
1520:     auto sorted_grad_values = grad_values.index_select(0, sort_order);
```
- L1481: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1482: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L1483: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L1484: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1486: Declares function `sparse_dim` as part of this file's callable surface. / 声明函数 `sparse_dim`，作为本文件可调用接口的一部分。
- L1487: Declares function `maybe_wrap_dim` as part of this file's callable surface. / 声明函数 `maybe_wrap_dim`，作为本文件可调用接口的一部分。
- L1488: Declares function `_nnz` as part of this file's callable surface. / 声明函数 `_nnz`，作为本文件可调用接口的一部分。
- L1490: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1491: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1492: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1493: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1494: Declares function `sum` as part of this file's callable surface. / 声明函数 `sum`，作为本文件可调用接口的一部分。
- L1495: Declares function `sub` as part of this file's callable surface. / 声明函数 `sub`，作为本文件可调用接口的一部分。
- L1496: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L1497: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L1498: Declares function `sum` as part of this file's callable surface. / 声明函数 `sum`，作为本文件可调用接口的一部分。
- L1499: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L1500: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1501: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1502: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1503: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1505: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1507: Declares function `zeros` as part of this file's callable surface. / 声明函数 `zeros`，作为本文件可调用接口的一部分。
- L1508: Declares function `sizes` as part of this file's callable surface. / 声明函数 `sizes`，作为本文件可调用接口的一部分。
- L1509: Declares function `strides` as part of this file's callable surface. / 声明函数 `strides`，作为本文件可调用接口的一部分。
- L1510: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1512: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L1513: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1514: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1515: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1517: Declares function `argsort` as part of this file's callable surface. / 声明函数 `argsort`，作为本文件可调用接口的一部分。
- L1518: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1519: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1520: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。

### Lines 1521-1560

```cpp
1521: 
1522:     auto nnz_u = static_cast<uint32_t>(nnz);
1523:     auto mask = at::empty({nnz}, sorted_pool_indices.options().dtype(kInt));
1524: 
1525:     dispatch_sync_with_rethrow(stream->queue(), ^() {
1526:         auto pso = lib.getPipelineStateForFunc("mark_segments");
1527:         auto enc = stream->commandEncoder();
1528:         [enc setComputePipelineState:pso];
1529:         mtl_setArgs(enc, sorted_pool_indices, mask);
1530:         auto gridSize = MTLSizeMake(nnz, 1, 1);
1531:         auto threadGroupSize = MTLSizeMake(std::min<uint64_t>(nnz, pso.maxTotalThreadsPerThreadgroup), 1, 1);
1532:         [enc dispatchThreads:gridSize threadsPerThreadgroup:threadGroupSize];
1533:     });
1534: 
1535:     auto scan = at::cumsum(mask, 0, kInt);
1536: 
1537:     auto offsets = at::empty({nnz}, mask.options());
1538:     auto counts = at::empty({nnz}, mask.options());
1539: 
1540:     dispatch_sync_with_rethrow(stream->queue(), ^() {
1541:         auto pso = lib.getPipelineStateForFunc("compute_offsets_and_counts");
1542:         auto enc = stream->commandEncoder();
1543:         [enc setComputePipelineState:pso];
1544:         mtl_setArgs(enc, scan, offsets, counts, nnz_u);
1545: 
1546:         auto gridSize = MTLSizeMake(nnz, 1, 1);
1547:         auto threadGroupSize = MTLSizeMake(std::min<uint64_t>(nnz, pso.maxTotalThreadsPerThreadgroup), 1, 1);
1548:         [enc dispatchThreads:gridSize threadsPerThreadgroup:threadGroupSize];
1549:     });
1550: 
1551:     auto sorted_grad_input = at::empty_like(sorted_output_values);
1552: 
1553:     int64_t nvalues = 1;
1554:     if (output_values.dim() > 1) nvalues = output_values.numel() / output_values.size(0);
1555:     auto nval_u = static_cast<uint32_t>(nvalues);
1556: 
1557:     dispatch_sync_with_rethrow(stream->queue(), ^() {
1558:         auto pso = lib.getPipelineStateForFunc("softmax_sparse_backward_" + mps::scalarToMetalTypeString(sorted_grad_values));
1559:         auto enc = stream->commandEncoder();
1560:         [enc setComputePipelineState:pso];
```
- L1522: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1523: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1525: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L1526: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L1527: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L1528: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1529: Declares function `mtl_setArgs` as part of this file's callable surface. / 声明函数 `mtl_setArgs`，作为本文件可调用接口的一部分。
- L1530: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1531: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1532: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1533: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1535: Declares function `cumsum` as part of this file's callable surface. / 声明函数 `cumsum`，作为本文件可调用接口的一部分。
- L1537: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1538: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L1540: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L1541: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L1542: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L1543: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1544: Declares function `mtl_setArgs` as part of this file's callable surface. / 声明函数 `mtl_setArgs`，作为本文件可调用接口的一部分。
- L1546: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1547: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1548: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1549: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1551: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。
- L1553: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L1554: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1555: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L1557: Defines function `dispatch_sync_with_rethrow` and begins its implementation body. / 定义函数 `dispatch_sync_with_rethrow`，并开始其实现体。
- L1558: Declares function `getPipelineStateForFunc` as part of this file's callable surface. / 声明函数 `getPipelineStateForFunc`，作为本文件可调用接口的一部分。
- L1559: Declares function `commandEncoder` as part of this file's callable surface. / 声明函数 `commandEncoder`，作为本文件可调用接口的一部分。
- L1560: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 1561-1599

```cpp
1561:         mtl_setArgs(enc, sorted_grad_values, sorted_output_values, sorted_grad_input,
1562:                     offsets, counts, scan,
1563:                     std::array<uint32_t, 2>{nnz_u, nval_u}, logsoftmax);
1564: 
1565:         auto gridSize = MTLSizeMake(nnz, 1, 1);
1566:         auto threadGroupSize = MTLSizeMake(std::min<uint64_t>(nnz, pso.maxTotalThreadsPerThreadgroup), 1, 1);
1567:         [enc dispatchThreads:gridSize threadsPerThreadgroup:threadGroupSize];
1568:     });
1569: 
1570:     auto inv_sort_order = at::argsort(sort_order);
1571:     auto grad_input_values = sorted_grad_input.index_select(0, inv_sort_order);
1572:     if (is_half) {
1573:         grad_input_values = grad_input_values.to(kHalf);
1574:     }
1575:     return at::_sparse_coo_tensor_unsafe(
1576:         indices_safe, grad_input_values, output.sizes(),
1577:         output.options().dtype(grad_input_values.scalar_type())
1578:     )._coalesced_(true);
1579: }
1580: 
1581: Tensor softmax_sparse_mps(const Tensor& input, const int64_t dim, const bool half_to_float) {
1582:   return softmax_sparse_mps_impl(input, dim, half_to_float, false);
1583: }
1584: 
1585: Tensor log_softmax_sparse_mps(const Tensor& input, const int64_t dim, const bool half_to_float) {
1586:   return softmax_sparse_mps_impl(input, dim, half_to_float, true);
1587: }
1588: 
1589: Tensor softmax_backward_sparse_mps(const Tensor& grad, const Tensor& output, int64_t dim, const Tensor& input) {
1590:     return softmax_backward_sparse_mps_impl(grad, output, dim, input, false);
1591: }
1592: 
1593: Tensor log_softmax_backward_sparse_mps(const Tensor& grad, const Tensor& output, int64_t dim, const Tensor& input) {
1594:   return softmax_backward_sparse_mps_impl(grad, output, dim, input, true);
1595: }
1596: 
1597: REGISTER_MPS_DISPATCH(sparse_mask_intersection_out_stub, &sparse_mask_intersection_out_mps_kernel);
1598: REGISTER_MPS_DISPATCH(sparse_mask_projection_out_stub, &sparse_mask_projection_out_mps_kernel);
1599: } // namespace at::native
```
- L1561: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1562: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1563: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1565: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1566: Declares function `MTLSizeMake` as part of this file's callable surface. / 声明函数 `MTLSizeMake`，作为本文件可调用接口的一部分。
- L1567: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1568: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1570: Declares function `argsort` as part of this file's callable surface. / 声明函数 `argsort`，作为本文件可调用接口的一部分。
- L1571: Declares function `index_select` as part of this file's callable surface. / 声明函数 `index_select`，作为本文件可调用接口的一部分。
- L1572: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L1573: Declares function `to` as part of this file's callable surface. / 声明函数 `to`，作为本文件可调用接口的一部分。
- L1574: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1575: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1576: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1577: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L1578: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。
- L1579: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1581: Defines function `softmax_sparse_mps` and begins its implementation body. / 定义函数 `softmax_sparse_mps`，并开始其实现体。
- L1582: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1583: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1585: Defines function `log_softmax_sparse_mps` and begins its implementation body. / 定义函数 `log_softmax_sparse_mps`，并开始其实现体。
- L1586: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1587: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1589: Defines function `softmax_backward_sparse_mps` and begins its implementation body. / 定义函数 `softmax_backward_sparse_mps`，并开始其实现体。
- L1590: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1591: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1593: Defines function `log_softmax_backward_sparse_mps` and begins its implementation body. / 定义函数 `log_softmax_backward_sparse_mps`，并开始其实现体。
- L1594: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L1595: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L1597: Declares function `REGISTER_MPS_DISPATCH` as part of this file's callable surface. / 声明函数 `REGISTER_MPS_DISPATCH`，作为本文件可调用接口的一部分。
- L1598: Declares function `REGISTER_MPS_DISPATCH` as part of this file's callable surface. / 声明函数 `REGISTER_MPS_DISPATCH`，作为本文件可调用接口的一部分。
- L1599: Closes namespace `at::native` and returns to the outer scope. / 关闭命名空间 `at::native`，返回外层作用域。

## Key Concepts / 关键概念

- MPS sparse backend support / MPS 稀疏后端支持
- Sparse tensor math and layout utilities / 稀疏张量数学与布局工具
- Softmax normalization and numerical stability / Softmax 归一化与数值稳定性
- LogSoftmax accumulation strategy / LogSoftmax 累积策略
- Sparse layout semantics and NNZ traversal / 稀疏布局语义与 NNZ 遍历
- CSR compressed sparse representation / CSR 压缩稀疏表示
- COO index/value representation / COO 索引/数值表示
- Tensor shape, stride, and dtype flow / Tensor 形状、步长与数据类型流转

## Dependencies / 依赖关系

- `ATen/native/SparseTensorUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/SparseCsrTensorUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ExpandUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/WrapDimUtilsMulti.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/mps/OperationUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/sparse/SparseStubs.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/sparse/SparseBinaryOpIntersectionCommon.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Functions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/NativeFunctions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_coalesce_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/repeat_interleave_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/cumsum.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_sparse_matmul_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_coo_tensor_unsafe.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_coo_tensor_unsafe_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/cat.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/softmax_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/log_softmax.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_log_softmax_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_softmax_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_softmax_backward_data_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_log_softmax_backward_data_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/add_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/mul_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/zeros_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/index_select_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/remainder_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/ones_like.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/argsort.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- Subsystem tie-in: sparse layouts (COO/CSR/CSC/BSR), index transforms, and NNZ-oriented computation. / 子系统关联：稀疏布局（COO/CSR/CSC/BSR）、索引变换以及面向 NNZ 的计算。
