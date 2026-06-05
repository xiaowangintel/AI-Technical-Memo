# LinearAlgebraUtils.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/LinearAlgebraUtils.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Linear Algebra Utils. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 线性、代数、utils 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <c10/core/ScalarType.h>
0004: #include <c10/util/irange.h>
0005: #include <c10/util/Exception.h>
0006: #include <c10/util/strides.h>
0007: #include <ATen/core/Tensor.h>
0008: #include <ATen/ExpandUtils.h>
0009: #include <ATen/TensorUtils.h>
0010: #include <ATen/native/TensorIterator.h>
0011: #include <ATen/native/TransposeType.h>
0012: #include <limits>
0013: #include <type_traits>
0014: #include <sstream>
0015: #include <cstring>
0016: #include <cctype>
0017: 
0018: #ifndef AT_PER_OPERATOR_HEADERS
0019: #include <ATen/Functions.h>
0020: #else
0021: #include <ATen/ops/arange.h>
0022: #include <ATen/ops/empty.h>
0023: #include <ATen/ops/empty_like.h>
0024: #include <ATen/ops/empty_strided.h>
0025: #include <ATen/ops/zeros.h>
0026: #endif
0027: 
0028: namespace at::native {
0029: 
0030: inline c10::MaybeOwned<Tensor> expect_resolved_conj(const Tensor& tensor) {
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue. Notable symbols: expect_resolved_conj.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。 值得关注的符号包括：expect_resolved_conj。

### Lines 31-60 / 第 31-60 行
```cpp
0031:   if (tensor.is_conj()) {
0032:     return c10::MaybeOwned<Tensor>::owned(tensor.resolve_conj());
0033:   } else {
0034:     return c10::MaybeOwned<Tensor>::borrowed(tensor);
0035:   }
0036: }
0037: 
0038: inline DimVector batched_matrix_contiguous_strides(
0039:     const IntArrayRef sizes,
0040:     const bool f_contig = false) {
0041:   // f_contig chooses between the strides of a batch of Fortran (F-contiguous)
0042:   // and C-contiguous matrices
0043:   auto strides = c10::contiguous_strides(sizes);
0044:   auto dim = strides.size();
0045: 
0046:   if (f_contig && dim >= 2) {
0047:     // Fix the strides of the last two dimensions, so that we return
0048:     // C-contiguous batches of F-contiguous matrices.
0049:     strides[dim - 1] = std::max(sizes[dim - 2], static_cast<int64_t>(1));
0050:     strides[dim - 2] = 1;
0051:   }
0052:   return strides;
0053: }
0054: 
0055: /*
0056:  * Clones a Tensor so that the following conditions hold:
0057:  * If we think of a Tensor of having size (B, M, N), where B is any number
0058:  * of batch dimensions, then:
0059:  * - Each (M, N) matrix is in column major form
0060:  * - Let Tensor P have size (B, M, N) and Q have size (B, M', N').
```
- **EN**: Lines 31-60 mainly cover comments/documentation, expressions/calls, state/variable declarations. Notable symbols: is_conj, owned, resolve_conj, borrowed.
- **CN**: 第 31-60 行主要涉及注释或说明、表达式或调用、变量/别名声明。 值得关注的符号包括：is_conj, owned, resolve_conj, borrowed。

### Lines 61-90 / 第 61-90 行
```cpp
0061:  *   Then when laid out in memory, the M by N matrix starting at
0062:  *   P.data_ptr()[B * M * N] is of the same corresponding batch as the M' by N'
0063:  *   matrix starting at Q.data_ptr()[B * M' * N'].
0064:  */
0065: inline Tensor cloneBatchedColumnMajor(const Tensor& src) {
0066:   // If src is already in batched column major format, then
0067:   // this will be efficient (no reordering of the data will occur)
0068:   // because the first transpose will make the tensor contiguous,
0069:   // and cloning a contiguous tensor is fast.
0070:   auto result = src.mT().clone(at::MemoryFormat::Contiguous);
0071:   result.transpose_(-2, -1);
0072:   return result;
0073: }
0074: 
0075: /*
0076:  * contig chooses between C-contig (true) and F-contig (false)
0077:  */
0078: inline c10::MaybeOwned<Tensor> borrow_else_clone(const bool cond, const Tensor& borrow, const Tensor& clone, const bool contig) {
0079:   return cond ? c10::MaybeOwned<Tensor>::borrowed(borrow)
0080:               : c10::MaybeOwned<Tensor>::owned(contig ? clone.clone(MemoryFormat::Contiguous)
0081:                                                       : cloneBatchedColumnMajor(clone));
0082: }
0083: 
0084: /*
0085:  * This method is designed to be a faster alternative to
0086:  * `cloneBatchedColumnMajor` with some additional features,
0087:  * namely:
0088:  * 1. It uses `copy` instead of `clone` which could be much faster.
0089:  * 2. `nrows` parameter used to create inputs with the number of rows larger
0090:  *  than the original input, which is required for some LAPACK/MAGMA methods.
```
- **EN**: Lines 61-90 mainly cover comments/documentation, state/variable declarations, return paths. Notable symbols: data_ptr, cloneBatchedColumnMajor, efficient, mT.
- **CN**: 第 61-90 行主要涉及注释或说明、变量/别名声明、返回路径。 值得关注的符号包括：data_ptr, cloneBatchedColumnMajor, efficient, mT。

### Lines 91-120 / 第 91-120 行
```cpp
0091:  * 3. `desired_batch_size` is used to create copies with the batch size
0092:  *  which is either the original batch size of the input, or its larger
0093:  *  broadcasted shape.
0094:  */
0095: inline Tensor copyBatchedColumnMajor(const Tensor& src, int64_t nrows = -1,
0096:     at::OptionalIntArrayRef desired_batch_sizes = std::nullopt) {
0097:   nrows = (nrows == -1) ? src.size(-2) : nrows;
0098:   auto copy_sizes = desired_batch_sizes.has_value()
0099:     ? desired_batch_sizes.value().vec()
0100:     : IntArrayRef(src.sizes().data(), src.dim() - 2).vec();
0101:   copy_sizes.insert(copy_sizes.end(), {nrows, src.size(-1)});
0102:   const auto copy_strides = batched_matrix_contiguous_strides(copy_sizes, /*f-contig*/true);
0103:   auto copy = at::empty_strided(copy_sizes, copy_strides, src.options());
0104:   copy.narrow(-2, 0, src.size(-2)).copy_(src);
0105:   return copy;
0106: }
0107: 
0108: /*
0109:  * Given batches of matrices with arbitrary batch dim,
0110:  * computes the number of batches.
0111:  */
0112: inline int64_t batchCount(const Tensor& batched_matrices) {
0113:   int64_t result = 1;
0114:   for (int64_t i = 0; i < batched_matrices.ndimension() - 2; i++) {
0115:     result *= batched_matrices.size(i);
0116:   }
0117:   return result;
0118: }
0119: 
0120: // Computes the number of elements of a matrix in a batched matrix tensor
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: copyBatchedColumnMajor, size, has_value, value.
- **CN**: 第 91-120 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：copyBatchedColumnMajor, size, has_value, value。

### Lines 121-150 / 第 121-150 行
```cpp
0121: inline int64_t matrixStride(const Tensor& batched_matrices) {
0122:   return batched_matrices.size(-1) * batched_matrices.size(-2);
0123: }
0124: 
0125: // Validates input shapes for operations on batches of square matrices (inverse, cholesky, symeig, eig)
0126: inline void checkIsMatrix(const Tensor& A, const char* const f_name, const char* const arg_name = "A") {
0127:   TORCH_CHECK(A.dim() >= 2, f_name, ": The input tensor ", arg_name, " must have at least 2 dimensions.");
0128: }
0129: inline void squareCheckInputs(const Tensor& self, const char* const f_name, const char* const arg_name = "A") {
0130:   checkIsMatrix(self, f_name, arg_name);
0131:   TORCH_CHECK(self.sym_size(-1) == self.sym_size(-2),
0132:               f_name,
0133:               ": ", arg_name, " must be batches of square matrices, "
0134:               "but they are ", self.sym_size(-2), " by ", self.sym_size(-1), " matrices");
0135: }
0136: 
0137: inline void checkInputsSolver(const Tensor& A,
0138:                                      const Tensor& B,
0139:                                      const bool left,
0140:                                      const char* const f_name) {
0141:   squareCheckInputs(A, f_name, "A");
0142:   checkIsMatrix(B, f_name, "B");
0143:   TORCH_CHECK(left ? A.size(-2) == B.size(-2) : A.size(-1) == B.size(-1),
0144:               f_name, ": Incompatible shapes of A and B for the equation ",
0145:               left ? "AX = B" : "XA = B",
0146:               " (", A.size(-2), "x", A.size(-1), " and ", B.size(-2), "x", B.size(-1), ")");
0147: }
0148: 
0149: inline bool is_row_or_column_contiguous(const Tensor& t) {
0150:   // This could be made more general, similar to how it's checked in matmul, which would allow to
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: matrixStride, size, matrices, checkIsMatrix.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：matrixStride, size, matrices, checkIsMatrix。

### Lines 151-180 / 第 151-180 行
```cpp
0151:   // elide the copy with strides such as (6, 12, 1, 3) or (3, 1, 9), but this is quite tricky.
0152:   // We choose to be conservative for simplicity
0153:   return t.is_contiguous() || t.transpose(-2, -1).is_contiguous();
0154: }
0155: 
0156: inline TransposeType to_transpose_type(const bool contig, const bool conj) {
0157:   if (conj) {
0158:     if (contig) { TORCH_INTERNAL_ASSERT(false, "Invalid transpose type"); }
0159:     else {        return TransposeType::ConjTranspose; }
0160:   } else {
0161:     if (contig) { return TransposeType::NoTranspose; }
0162:     else {        return TransposeType::Transpose; }
0163:   }
0164: }
0165: 
0166: 
0167: // This function is designed to be used with linear algebra methods that minimize
0168: // L(ax - b) = 0, where L is generally the identity map (`solve`, for example)
0169: // or the L2 norm (`lstsq`).
0170: // It is expected that `a` and `b` are contiguous tensors of column-major matrices
0171: // (so that a.view({-1, a.size(-2), a.size(-1)}) succeeds, same for `b`),
0172: // with the following additional properties:
0173: //
0174: // 1. a.dim() == b.dim()
0175: // 2. a.shape[:-2] broadcasts over b.shape[:-2]
0176: // 3. a.size(i) <= b.size(i) for i=0,..., a.dim() - 3 (only for batch dimensions)
0177: //
0178: // MAGMA/LAPACK modify tensor `a` in-place, and the main goal of this method
0179: // is to be memory efficient, which means that if there exists an index i such that
0180: // a.shape[i] < b.shape[i], 0 <= i <= a.dim() - 3,
```
- **EN**: Lines 151-180 mainly cover comments/documentation, control-flow checks, expressions/calls. Notable symbols: as, or, is_contiguous, transpose.
- **CN**: 第 151-180 行主要涉及注释或说明、控制流逻辑、表达式或调用。 值得关注的符号包括：as, or, is_contiguous, transpose。

### Lines 181-210 / 第 181-210 行
```cpp
0181: // then instead of materializing copies of `a` in the broadcasted shape, we keep
0182: // a buffer copy of `a` along with flags that check whether specific batch dimension
0183: // indices for `a` were already accessed. If they were, we copy the data from the buffer
0184: // into `a`. The number of copies does not exceed
0185: // prod(max(a.shape[:-2], b.shape[:-2]) - a.shape[:-2] + 1)
0186: // and this value is attained by tensors with non-empty batch dimensions.
0187: //
0188: // func_t `f` is a callable that is being supplied with
0189: // scalar_t* a_working_ptr, scalar_t* b_working_ptr, int64_t a_linear_batch_idx.
0190: // a_working_ptr and b_working_ptr can directly be passed to LAPACK/MAGMA routines,
0191: // and a_linear_batch_idx is an index in the 3d representation which corresponds to
0192: // the memory a_working_ptr points to, in other words:
0193: // a_working_ptr == a.view({-1, a.size(-2), a.size(-1)}.select(0, a_linear_batch_idx).data_ptr<scalar_t>();
0194: // a_linear_batch_idx is useful to store metadata related to `a`, such as, for example,
0195: // its rank or singular values (see linalg_lstsq).
0196: template<typename scalar_t, typename func_t>
0197: void batch_iterator_with_broadcasting(const Tensor& a, const Tensor& b, const func_t& f) {
0198:   IntArrayRef a_batch_sizes(a.sizes().data(), a.dim() - 2);
0199:   IntArrayRef b_batch_sizes(b.sizes().data(), b.dim() - 2);
0200: 
0201:   auto a_linear_batch_idx = at::arange(batchCount(a)).view(a_batch_sizes);
0202:   auto b_linear_batch_idx = at::arange(batchCount(b)).view(b_batch_sizes);
0203: 
0204:   TensorIterator iter = TensorIteratorConfig()
0205:     .set_check_mem_overlap(false)
0206:     .check_all_same_dtype(false)
0207:     .resize_outputs(false)
0208:     .add_output(b_linear_batch_idx)
0209:     .add_input(a_linear_batch_idx)
0210:     .build();
```
- **EN**: Lines 181-210 mainly cover comments/documentation, function signatures/definitions, state/variable declarations. Notable symbols: prod, max, view, size.
- **CN**: 第 181-210 行主要涉及注释或说明、函数签名或实现、变量/别名声明。 值得关注的符号包括：prod, max, view, size。

### Lines 211-240 / 第 211-240 行
```cpp
0211: 
0212:   auto m = a.size(-2);
0213:   auto n = a.size(-1);
0214:   auto a_3d = a.view({batchCount(a), m, n});
0215:   auto b_3d = b.view({batchCount(b), b.size(-2), b.size(-1)});
0216: 
0217:   auto a_broadcasts_over_b = (a_batch_sizes != b_batch_sizes);
0218:   Tensor a_buffer, a_was_accessed, a_buffer_3d;
0219:   std::function<void(int64_t)> check_if_copy_needed_for_a
0220:     = [](int64_t /*a_curr_linear_batch_idx*/){};
0221:   if (a_broadcasts_over_b) {
0222:     a_buffer = at::empty_strided(a.sizes(), a.strides(), a.options())
0223:       .copy_(a);
0224:     a_was_accessed = at::zeros(batchCount(a), at::kBool);
0225:     a_buffer_3d = a_buffer.view({batchCount(a), m, n});
0226:     check_if_copy_needed_for_a = [&](int64_t a_curr_linear_batch_idx) {
0227:       auto* a_was_accessed_flag = a_was_accessed
0228:         .select(0, a_curr_linear_batch_idx)
0229:         .data_ptr<bool>();
0230:       if (!(*a_was_accessed_flag)) {
0231:         *a_was_accessed_flag = true;
0232:       }
0233:       else {
0234:         a_3d.select(0, a_curr_linear_batch_idx)
0235:           .copy_(a_buffer_3d.select(0, a_curr_linear_batch_idx));
0236:       }
0237:     };
0238:   }
0239: 
0240:   auto loop = [&](char** data, const int64_t* strides, int64_t nelems) {
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: size, view, batchCount, void.
- **CN**: 第 211-240 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：size, view, batchCount, void。

### Lines 241-270 / 第 241-270 行
```cpp
0241:     auto* b_batch_idx_ptr = data[0];
0242:     auto* a_batch_idx_ptr = data[1];
0243: 
0244:     for ([[maybe_unused]] const auto elem : c10::irange(nelems)) {
0245:       auto b_curr_linear_batch_idx =
0246:           *reinterpret_cast<int64_t*>(b_batch_idx_ptr);
0247:       auto a_curr_linear_batch_idx = *reinterpret_cast<int64_t*>(a_batch_idx_ptr);
0248: 
0249:       check_if_copy_needed_for_a(a_curr_linear_batch_idx);
0250: 
0251:       auto* a_working_ptr = a_3d.select(0, a_curr_linear_batch_idx)
0252:         .data_ptr<scalar_t>();
0253:       auto* b_working_ptr = b_3d.select(0, b_curr_linear_batch_idx)
0254:         .data_ptr<scalar_t>();
0255:       f(a_working_ptr, b_working_ptr, a_curr_linear_batch_idx);
0256: 
0257:       b_batch_idx_ptr += strides[0];
0258:       a_batch_idx_ptr += strides[1];
0259:     }
0260:   };
0261:   iter.serial_for_each(loop, {0, batchCount(b)});
0262: }
0263: 
0264: // Returns the epsilon value for floating types except half
0265: inline double _get_epsilon(const ScalarType& sc_type) {
0266:   switch (sc_type) {
0267:     case at::ScalarType::Float:
0268:       return static_cast<double>(std::numeric_limits<float>::epsilon());
0269:     case at::ScalarType::Double:
0270:       return std::numeric_limits<double>::epsilon();
```
- **EN**: Lines 241-270 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: irange, check_if_copy_needed_for_a, select, f.
- **CN**: 第 241-270 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：irange, check_if_copy_needed_for_a, select, f。

### Lines 271-300 / 第 271-300 行
```cpp
0271:     default:
0272:       TORCH_CHECK(false, "This function doesn't handle types other than float and double");
0273:   }
0274: }
0275: 
0276: // Validates input shapes and devices
0277: // for linear solve methods (solve, cholesky_solve, lu_solve, triangular_solve)
0278: inline void linearSolveCheckInputs(const Tensor& self, const Tensor& A, const char* name) {
0279:   TORCH_CHECK(self.device() == A.device(),
0280:               "Expected b and A to be on the same device, but found b on ",
0281:               self.device(), " and A on ", A.device(), " instead.");
0282: 
0283:   TORCH_CHECK(self.scalar_type() == A.scalar_type(),
0284:               "Expected b and A to have the same dtype, but found b of type ",
0285:               self.scalar_type(), " and A of type ", A.scalar_type(), " instead.");
0286: 
0287:   TORCH_CHECK(A.size(-1) == A.size(-2),
0288:               "A must be batches of square matrices, "
0289:               "but they are ", A.size(-2), " by ", A.size(-1), " matrices");
0290: 
0291:   TORCH_CHECK(A.size(-1) == self.size(-2),
0292:               "Incompatible matrix sizes for ", name, ": each A "
0293:               "matrix is ", A.size(-1), " by ", A.size(-1),
0294:               " but each b matrix is ", self.size(-2), " by ", self.size(-1));
0295: }
0296: 
0297: inline void checkFloatingOrComplex(const Tensor& t, const char* const f_name, const bool allow_low_precision_dtypes=true) {
0298:   auto dtype = t.scalar_type();
0299:   TORCH_CHECK((at::isFloatingType(dtype) || at::isComplexType(dtype)),
0300:               f_name, ": Expected a floating point or complex tensor as input. Got ", dtype);
```
- **EN**: Lines 271-300 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TORCH_CHECK, methods, linearSolveCheckInputs, device.
- **CN**: 第 271-300 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, methods, linearSolveCheckInputs, device。

### Lines 301-330 / 第 301-330 行
```cpp
0301:   if (!allow_low_precision_dtypes) {
0302:     TORCH_CHECK(dtype == kFloat || dtype == kDouble || dtype == kComplexFloat || dtype == kComplexDouble,
0303:                 f_name, ": Low precision dtypes not supported. Got ", dtype);
0304:   }
0305: }
0306: 
0307: 
0308: // Checks if all the Tensors in a TensorList are of the same dimensions
0309: inline void checkAllSameDim(TensorList tensors, int64_t dim) {
0310:   for (auto &t : tensors) {
0311:     TORCH_CHECK(t.dim() == dim, "Tensor dimension is ", t.dim(), ", expected ", dim, " instead.");
0312:   }
0313: }
0314: 
0315: inline std::tuple<std::vector<int64_t>, std::vector<int64_t>> _linalg_broadcast_batch_dims(const Tensor& arg1, const Tensor& arg2) {
0316:   // broadcast the batch dimensions of arg1 and arg2.
0317:   IntArrayRef arg1_batch_sizes(arg1.sizes().data(), arg1.ndimension() - 2);
0318:   IntArrayRef arg2_batch_sizes(arg2.sizes().data(), arg2.ndimension() - 2);
0319:   std::vector<int64_t> expand_batch_portion = infer_size(arg1_batch_sizes, arg2_batch_sizes);
0320: 
0321:   std::vector<int64_t> arg1_expand_size({expand_batch_portion});
0322:   arg1_expand_size.insert(arg1_expand_size.end(), { arg1.size(-2), arg1.size(-1) });
0323: 
0324:   std::vector<int64_t> arg2_expand_size({expand_batch_portion});
0325:   arg2_expand_size.insert(arg2_expand_size.end(), { arg2.size(-2), arg2.size(-1) });
0326:   return std::make_tuple(std::move(arg1_expand_size), std::move(arg2_expand_size));
0327: }
0328: 
0329: inline std::tuple<Tensor,Tensor> _linalg_broadcast_batch_dims(const Tensor& arg1, const Tensor& arg2, const char* name) {
0330:   // If there's no name we assume we don't want to check the errors
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: TORCH_CHECK, checkAllSameDim, dim, _linalg_broadcast_batch_dims.
- **CN**: 第 301-330 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：TORCH_CHECK, checkAllSameDim, dim, _linalg_broadcast_batch_dims。

### Lines 331-360 / 第 331-360 行
```cpp
0331:   if (name != nullptr) {
0332:     linearSolveCheckInputs(arg1, arg2, name);
0333:   }
0334: 
0335:   auto [arg1_expand_size, arg2_expand_size] = at::native::_linalg_broadcast_batch_dims(arg1, arg2);
0336: 
0337:   auto arg1_broadcasted  = arg1_expand_size == arg1.sizes() ? arg1 : arg1.expand(arg1_expand_size);
0338:   auto arg2_broadcasted  = arg2_expand_size == arg2.sizes() ? arg2 : arg2.expand(arg2_expand_size);
0339:   return std::make_tuple(std::move(arg1_broadcasted), std::move(arg2_broadcasted));
0340: }
0341: 
0342: inline std::vector<int64_t> broadcast_batch_size(const Tensor& t1, const Tensor& t2, int64_t n_batch_dims) {
0343:   IntArrayRef t1_batch_sizes(t1.sizes().data(), n_batch_dims);
0344:   IntArrayRef t2_batch_sizes(t2.sizes().data(), n_batch_dims);
0345:   auto broadcasted_batch_sizes = infer_size(t1_batch_sizes, t2_batch_sizes);
0346:   return broadcasted_batch_sizes;
0347: }
0348: 
0349: // Return a permutation with the given axes moved to the end.
0350: inline Tensor _move_to_end(const Tensor& self, IntArrayRef axes) {
0351:   const std::vector<int64_t> a = axes.vec();
0352:   const int64_t ndim = self.ndimension();
0353:   std::vector<int64_t> perm;
0354:   perm.reserve(static_cast<size_t>(std::max<int64_t>(0, ndim)));
0355: 
0356:   for (const auto i : c10::irange(ndim)) {
0357:     auto it = std::find(a.begin(), a.end(), i);
0358:     if (it == a.end()) {
0359:        perm.push_back(i);
0360:     }
```
- **EN**: Lines 331-360 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: linearSolveCheckInputs, _linalg_broadcast_batch_dims, sizes, expand.
- **CN**: 第 331-360 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：linearSolveCheckInputs, _linalg_broadcast_batch_dims, sizes, expand。

### Lines 361-390 / 第 361-390 行
```cpp
0361:   }
0362:   for (auto i : a) {
0363:     perm.push_back(i);
0364:   }
0365: 
0366:   TORCH_CHECK((int64_t)perm.size() == ndim,
0367:     "duplicate or invalid axis in 'dim' argument for tensor with ndim==", ndim);
0368: 
0369:   return self.permute(perm);
0370: }
0371: 
0372: // parse the "mode" param in linalg_qr: return a tuple of bools (compute_q, reduced)
0373: inline std::tuple<bool, bool> _parse_qr_mode(std::string_view mode) {
0374:   bool compute_q;
0375:   bool reduced;
0376:   if (mode == "reduced") {
0377:     compute_q = true;
0378:     reduced = true;
0379:   } else if (mode == "complete") {
0380:     compute_q = true;
0381:     reduced = false;
0382:   } else if (mode == "r") {
0383:     compute_q = false;
0384:     reduced = true; // this is actually irrelevant in this mode
0385:   } else {
0386:       TORCH_CHECK(false, "qr received unrecognized mode '", mode,
0387:                   "' but expected one of 'reduced' (default), 'r', or 'complete'");
0388:   }
0389:   return std::make_tuple(compute_q, reduced);
0390: }
```
- **EN**: Lines 361-390 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: push_back, TORCH_CHECK, size, permute.
- **CN**: 第 361-390 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：push_back, TORCH_CHECK, size, permute。

### Lines 391-420 / 第 391-420 行
```cpp
0391: 
0392: // Function to compute sizes, strides and the extra columns for the Q matrix in the QR Decomposition
0393: inline std::tuple<DimVector, DimVector, int64_t> _compute_geometry_for_Q(
0394:     const Tensor& input,
0395:     bool reduced) {
0396:   int64_t m = input.size(-2), n = input.size(-1);
0397:   int64_t n_columns_q;
0398: 
0399:   // We need to compute the required size of Q based on the `reduced` option
0400:   DimVector q_sizes(input.sizes());
0401:   if (!reduced && m > n) {
0402:     q_sizes[input.dim() - 1] = m;
0403:     n_columns_q = m;
0404:   } else {
0405:     q_sizes[input.dim() - 1] = n;
0406:     n_columns_q = std::min(m, n);
0407:   }
0408:   auto q_strides = batched_matrix_contiguous_strides(q_sizes, /*f-contig*/true);
0409:   return std::make_tuple(std::move(q_sizes), std::move(q_strides), n_columns_q);
0410: }
0411: 
0412: inline bool svd_uses_cusolver(const Tensor& A) {
0413:   // if cusolver is available, it is used unconditionally
0414:   return A.is_cuda()
0415:          && at::globalContext().hasCuSOLVER()
0416:          && at::globalContext().linalgPreferredBackend() != at::LinalgBackend::Magma;
0417: }
0418: 
0419: 
0420: // Function used instead of .to so that the original strides are retained
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: _compute_geometry_for_Q, size, q_sizes, sizes.
- **CN**: 第 391-420 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：_compute_geometry_for_Q, size, q_sizes, sizes。

### Lines 421-450 / 第 421-450 行
```cpp
0421: // .to doesn't retain strides and make the output tensor contiguous
0422: inline Tensor same_stride_to(const Tensor& original_tensor, const at::TensorOptions& options) {
0423:   auto strided_to = at::empty_strided(original_tensor.sizes(),
0424:                                       original_tensor.strides(),
0425:                                       options);
0426:   strided_to.copy_(original_tensor);
0427:   return strided_to;
0428: }
0429: 
0430: // Creates a dimension permutation array that can be given to `at::permute()`, which will shift
0431: // the two specified dimensions to the end of a tensor, without changing the order of
0432: // the other dimensions. `dim1` will be placed at the very end, and `dim0` will be
0433: // placed just to the left of it.
0434: //
0435: // For instance, given a 4-D tensor, dimensions 1 and 3 can be shifted to the end by
0436: // calling `create_dim_backshift_permutation(1, 3, 4)`. The resulting vector will
0437: // be `vec(0, 2, 1, 3)`.
0438: inline std::vector<int64_t> create_dim_backshift_permutation(int64_t dim0, int64_t dim1, int64_t ndim) {
0439:   TORCH_CHECK(
0440:     (dim0 != dim1) && (dim0 < ndim) && (dim0 >= 0) && (dim1 < ndim) && (dim1 >= 0),
0441:     "duplicate or invalid dimensions");
0442:   std::vector<int64_t> permutation(ndim);
0443:   int64_t cur_permuted_dim = 0;
0444:   for (const auto dim_ind : c10::irange(ndim)) {
0445:     if ((dim_ind != dim0) && (dim_ind != dim1)) {
0446:       permutation[cur_permuted_dim++] = dim_ind;
0447:     }
0448:   }
0449:   permutation[cur_permuted_dim++] = dim0;
0450:   permutation[cur_permuted_dim] = dim1;
```
- **EN**: Lines 421-450 mainly cover state/variable declarations, comments/documentation, function signatures/definitions. Notable symbols: same_stride_to, empty_strided, sizes, strides.
- **CN**: 第 421-450 行主要涉及变量/别名声明、注释或说明、函数签名或实现。 值得关注的符号包括：same_stride_to, empty_strided, sizes, strides。

### Lines 451-480 / 第 451-480 行
```cpp
0451:   return permutation;
0452: }
0453: 
0454: // Creates a dimension permutation array that can be given to `at::permute()`, which
0455: // will reverse a given permutation.
0456: // The reverse permutation array is created by swapping the indices and their
0457: // associated values from the given permutation array.
0458: inline std::vector<int64_t> create_reverse_permutation(std::vector<int64_t> permutation) {
0459:   int64_t ndim = permutation.size();
0460:   std::vector<int64_t> reverse_permutation(ndim);
0461:   for (const auto dim_ind : c10::irange(ndim)) {
0462:     reverse_permutation[permutation[dim_ind]] = dim_ind;
0463:   }
0464:   return reverse_permutation;
0465: }
0466: 
0467: // Compute R-work array size for MAGMA/LAPACK cgesdd/zgesdd
0468: // See https://github.com/Reference-LAPACK/lapack/blob/122506cd8b6ce050a200920c3d4c0b153b150fd8/SRC/cgesdd.f#L186
0469: inline int64_t computeLRWorkDim(const char jobz, int64_t m, int64_t n) {
0470:   auto mn = std::min(m, n);
0471:   auto mx = std::max(m, n);
0472:   if (jobz == 'N') {
0473: #ifdef __APPLE__
0474:     // According to `vecLib.framework/Headers/clapack.h` Accelerate.framework is based on LAPACK 3.2.1
0475:     return 7 * mn;
0476: #else
0477:     // These setting is valid for on LAPACK 3.6+
0478:     return 5 * mn;
0479: #endif
0480:   }
```
- **EN**: Lines 451-480 mainly cover comments/documentation, state/variable declarations, return paths. Notable symbols: permute, create_reverse_permutation, size, reverse_permutation.
- **CN**: 第 451-480 行主要涉及注释或说明、变量/别名声明、返回路径。 值得关注的符号包括：permute, create_reverse_permutation, size, reverse_permutation。

### Lines 481-510 / 第 481-510 行
```cpp
0481:   if (mx > 10 * mn) {
0482:     return 5 * mn * mn + 5 * mn;
0483:   }
0484:   return std::max(5 * mn * mn + 5 * mn, 2 * mx * mn + 2 * mn * mn + mn);
0485: }
0486: 
0487: // This function checks whether the uplo argument input is valid
0488: // Allowed strings are "u", "U", "l", "L"
0489: inline void checkUplo(const std::string_view uplo) {
0490:   // To use std::toupper safely with plain chars (or signed chars), the argument should first be converted to unsigned char
0491:   char uplo_uppercase = static_cast<char>(std::toupper(static_cast<unsigned char>(uplo[0])));
0492:   TORCH_CHECK(uplo.size() == 1 && (uplo_uppercase == 'U' || uplo_uppercase == 'L'),
0493:     "Expected UPLO argument to be 'L' or 'U', but got ", uplo);
0494: }
0495: 
0496: inline void checkSameDevice(const std::string& fn_name, Tensor result, Tensor input, const std::string& result_name = "result") {
0497:   TORCH_CHECK(
0498:       result.device() == input.device(),
0499:       fn_name,
0500:       ": Expected ", result_name, " and input tensors to be on the same device, but got ",
0501:       result_name, " on ", result.device(), " and input on ", input.device());
0502: }
0503: 
0504: // Check the dtype of result and input tensors (for _out variants).
0505: // Most linear algebra functions have the same dtype for input and output
0506: // (either floating or complex type input), so we can check whether input's dtype can be casted to result's dtype.
0507: // According to https://github.com/pytorch/pytorch/wiki/Developer-FAQ#how-does-out-work-in-pytorch
0508: // c10::canCast is used for checking the "safe copy" dtype requirements.
0509: inline void checkLinalgCompatibleDtype(const std::string& fn_name, Tensor result, Tensor input, const std::string& result_name = "result") {
0510:   bool can_cast = c10::canCast(input.scalar_type(), result.scalar_type());
```
- **EN**: Lines 481-510 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: max, checkUplo, chars, toupper.
- **CN**: 第 481-510 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：max, checkUplo, chars, toupper。

### Lines 511-540 / 第 511-540 行
```cpp
0511:   TORCH_CHECK(
0512:       can_cast,
0513:       fn_name,
0514:       ": Expected ", result_name, " to be safely castable from ", input.scalar_type(), " dtype, but got ",
0515:       result_name, " with dtype ", result.scalar_type());
0516: }
0517: 
0518: // Alternatively, we can check whether the specific expected output type (result_type) can be safely casted to out tensor dtype (out_type)
0519: inline void checkLinalgCompatibleDtype(const std::string& fn_name, ScalarType out_type, ScalarType result_type, const std::string& out_name = "result") {
0520:   bool can_cast = c10::canCast(result_type, out_type);
0521:   TORCH_CHECK(
0522:       can_cast,
0523:       fn_name,
0524:       ": Expected ", out_name, " to be safely castable from ", result_type, " dtype, but got ",
0525:       out_name, " with dtype ", out_type);
0526: }
0527: 
0528: inline void checkNotComplexTolerance(const Tensor& tol, const std::string_view f_name, const std::string_view tol_name) {
0529:   TORCH_CHECK(!at::isComplexType(tol.scalar_type()),
0530:               f_name, ": ", tol_name, " tensor of complex type is not supported. Got ", tol.scalar_type());
0531: }
0532: 
0533: /*
0534:   Two types of 'other' tensors are supported when solving
0535:   a system of linear equations matmul(input, x) = other:
0536:   * 1-dimensional (1D) tensor or batch of 1D tensors (vector case)
0537:   * 2-dimensional (2D) tensor or batch of 2D tensors (matrix case).
0538:   The original torch.solve supported only the matrix case, while NumPy works for both cases.
0539:   For the batched input we need to be able to distinguish them.
0540:   Let input.shape = (batch_dimensions, m, n), then 'other' is of vector type if other.shape == (batch_dimensions, m).
```
- **EN**: Lines 511-540 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: TORCH_CHECK, scalar_type, type, dtype.
- **CN**: 第 511-540 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：TORCH_CHECK, scalar_type, type, dtype。

### Lines 541-570 / 第 541-570 行
```cpp
0541:   This rule is compatible with NumPy, see https://github.com/numpy/numpy/blob/v1.20.0/numpy/linalg/linalg.py#L384-L389
0542: */
0543: inline bool linalg_solve_is_vector_rhs(const Tensor& input, const Tensor& other) {
0544:   auto expected_batched_rhs_shape = SymIntArrayRef(input.sym_sizes().data(), input.dim() - 1); // input.shape[:-1]
0545:   bool vector_case = other.dim() == 1 || (input.dim() - 1 == other.dim() && other.sym_sizes().equals(expected_batched_rhs_shape));
0546:   return vector_case;
0547: }
0548: 
0549: /*
0550:   Computes linear indices for a tensor with original_shape to access its elements like it was a materialized broadcast tensor.
0551: */
0552: inline Tensor get_linear_indices(int64_t numel, IntArrayRef original_shape, IntArrayRef broadcast_shape) {
0553:   TensorOptions options = at::TensorOptions().dtype(at::kLong).device(at::kCPU);
0554:   return at::arange(numel, options).view(original_shape).broadcast_to(broadcast_shape).contiguous();
0555: }
0556: 
0557: class BroadcastLinearIndices {
0558:  private:
0559:   Tensor linear_indices_;
0560:   bool is_broadcasting_;
0561: 
0562:  public:
0563:   BroadcastLinearIndices(
0564:       int64_t numel,
0565:       IntArrayRef original_shape,
0566:       IntArrayRef broadcast_shape) : is_broadcasting_(!original_shape.equals(broadcast_shape)) {
0567:     // The assumption is that the broadcast_shape is a materialized broadcast
0568:     // shape of the original_shape. We need to compute the linear indices
0569:     // compatible with the original_shape to access the elements in the original
0570:     // tensor corresponding to the broadcast tensor.
```
- **EN**: Lines 541-570 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: linalg_solve_is_vector_rhs, SymIntArrayRef, sym_sizes, data.
- **CN**: 第 541-570 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：linalg_solve_is_vector_rhs, SymIntArrayRef, sym_sizes, data。

### Lines 571-600 / 第 571-600 行
```cpp
0571:     if (is_broadcasting_) {
0572:       linear_indices_ =
0573:           get_linear_indices(numel, original_shape, broadcast_shape);
0574:     }
0575:   }
0576:   int64_t operator()(int64_t broadcast_linear_index) {
0577:     return is_broadcasting_
0578:         ? linear_indices_.data_ptr<int64_t>()[broadcast_linear_index]
0579:         : broadcast_linear_index;
0580:   }
0581: };
0582: 
0583: inline bool is_blas_compatible_column_major_order(const Tensor& input) {
0584:   IntArrayRef input_strides = input.strides();
0585:   IntArrayRef input_sizes = input.sizes();
0586:   auto ndim = input.dim();
0587:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(ndim >= 2);
0588:   if (ndim > 3) {
0589:     return input.transpose(-2, -1).is_contiguous();
0590:   }
0591:   auto leading_dimension = input_strides[ndim - 1];
0592:   auto rows = input_sizes[ndim - 2];
0593:   bool batch_stride_compatible = true;
0594:   if (ndim == 3) {
0595:     auto cols = input_sizes[ndim - 1];
0596:     batch_stride_compatible =
0597:         input_strides[ndim - 3] >= leading_dimension * cols;
0598:   }
0599:   return (input_strides[ndim - 2] == 1) &&
0600:       (leading_dimension >= std::max<int64_t>(1, rows)) &&
```
- **EN**: Lines 571-600 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: get_linear_indices, is_blas_compatible_column_major_order, strides, sizes.
- **CN**: 第 571-600 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：get_linear_indices, is_blas_compatible_column_major_order, strides, sizes。

### Lines 601-625 / 第 601-625 行
```cpp
0601:       batch_stride_compatible;
0602: }
0603: 
0604: inline bool is_blas_compatible_row_major_order(const Tensor& input) {
0605:   IntArrayRef input_strides = input.strides();
0606:   IntArrayRef input_sizes = input.sizes();
0607:   auto ndim = input.dim();
0608:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(ndim >= 2);
0609:   if (ndim > 3) {
0610:     return input.is_contiguous();
0611:   }
0612:   auto leading_dimension = input_strides[ndim - 2];
0613:   auto cols = input_sizes[ndim - 1];
0614:   bool batch_stride_compatible = true;
0615:   if (ndim == 3) {
0616:     auto rows = input_sizes[ndim - 2];
0617:     batch_stride_compatible =
0618:         input_strides[ndim - 3] >= leading_dimension * rows;
0619:   }
0620:   return (input_strides[ndim - 1] == 1) &&
0621:       (leading_dimension >= std::max<int64_t>(1, cols)) &&
0622:       batch_stride_compatible;
0623: }
0624: 
0625: }  // namespace at::native
```
- **EN**: Lines 601-625 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: is_blas_compatible_row_major_order, strides, sizes, dim.
- **CN**: 第 601-625 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：is_blas_compatible_row_major_order, strides, sizes, dim。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现

## Dependencies / 依赖关系
- **Headers / 头文件**: `<c10/core/ScalarType.h>`, `<c10/util/irange.h>`, `<c10/util/Exception.h>`, `<c10/util/strides.h>`, `<ATen/core/Tensor.h>`, `<ATen/ExpandUtils.h>`, `<ATen/TensorUtils.h>`, `<ATen/native/TensorIterator.h>`, `<ATen/native/TransposeType.h>`, `<limits>` ...
- **Macros / 宏**: `TORCH_CHECK`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
