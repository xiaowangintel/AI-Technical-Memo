# SparseBlasImpl.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkl/SparseBlasImpl.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the MKL integration in PyTorch ATen native code and focuses on sparse blas impl; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的MKL 集成，主题聚焦于 sparse blas impl；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/Dispatch.h>
#include <ATen/SparseCsrTensorImpl.h>
#include <ATen/Tensor.h>
#include <ATen/mkl/Sparse.h>
#include <ATen/native/LinearAlgebraUtils.h>
#include <ATen/SparseCsrTensorUtils.h>
#include <ATen/native/mkl/SparseBlasImpl.h>

#include <c10/core/ScalarType.h>
#include <c10/util/MaybeOwned.h>

#if AT_USE_MKL_SPARSE()
#include <ATen/mkl/SparseBlas.h>
#include <ATen/mkl/SparseDescriptors.h>
#include <ATen/mkl/Utils.h>
#endif

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/cat.h>
#include <ATen/ops/sparse_coo_tensor.h>
#endif

namespace at::native::sparse::impl::mkl {

namespace {

#if AT_USE_MKL_SPARSE()
c10::MaybeOwned<Tensor> prepare_dense_matrix_for_mkl(
    const Tensor& tensor) {
  if (tensor.is_non_overlapping_and_dense() ||
      is_blas_compatible_row_major_order(tensor) ||
      is_blas_compatible_column_major_order(tensor)) {
    return at::native::expect_resolved_conj(tensor);
  } else {
    return c10::MaybeOwned<Tensor>::owned(
        tensor.clone(at::MemoryFormat::Contiguous));
  }
}

/*
  Get row-major or column-major matrix.

  Args:
  * `tensor` - 2D strided Tensor.
  * `row_major` - controls the memory layout.
*/
c10::MaybeOwned<Tensor> prepare_dense_matrix_for_mkl(
    const Tensor& tensor,
    bool row_major) {
  if (is_blas_compatible_row_major_order(tensor) && row_major) {
    return at::native::expect_resolved_conj(tensor);
  } else {
    if (row_major) {
      return c10::MaybeOwned<Tensor>::owned(
          tensor.clone(at::MemoryFormat::Contiguous));
    } else {
```
- EN: Lines 1-60 pull in 16 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_USE_MKL_SPARSE, is_blas_compatible_column_major_order, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 16 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_USE_MKL_SPARSE, is_blas_compatible_column_major_order，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
      return c10::MaybeOwned<Tensor>::owned(cloneBatchedColumnMajor(tensor));
    }
  }
}

c10::MaybeOwned<Tensor> inline prepare_dense_vector_for_mkl(
    const Tensor& tensor) {
  if (tensor.is_non_overlapping_and_dense()) {
    return c10::MaybeOwned<Tensor>::borrowed(tensor);
  } else {
    return c10::MaybeOwned<Tensor>::owned(
        tensor.clone(at::MemoryFormat::Contiguous));
  }
}

void inline indices_to_mkl_compatible_inplace(const Tensor& input) {
#ifdef MKL_ILP64
  // ILP64 is a 64-bit API version of MKL
  // Indices tensor must have ScalarType::Long type
  static_cast<SparseCsrTensorImpl*>(input.unsafeGetTensorImpl())
      ->set_member_tensors(
          input.crow_indices().to(kLong),
          input.col_indices().to(kLong),
          input.values(),
          input.sizes());
#else
  // LP64 is a 32-bit API version of MKL
  // Indices tensor must have ScalarType::Int type
  static_cast<SparseCsrTensorImpl*>(input.unsafeGetTensorImpl())
      ->set_member_tensors(
          input.crow_indices().to(kInt),
          input.col_indices().to(kInt),
          input.values(),
          input.sizes());
#endif
}

void inline col_indices_and_values_resize_(const Tensor& input, int64_t nnz) {
  static_cast<SparseCsrTensorImpl*>(input.unsafeGetTensorImpl())
      ->set_member_tensors(
          input.crow_indices(),
          input.col_indices().resize_({nnz}),
          input.values().resize_({nnz}),
          input.sizes());
}

/*
  Resizes `input` tensor and fills it with the data from MKL.
*/
template <typename scalar_t>
void mkl_result_copy_(const Tensor& input, sparse_matrix_t mkl_desc) {
  sparse_index_base_t indexing = SPARSE_INDEX_BASE_ZERO;
  MKL_INT rows = 0, cols = 0;
  MKL_INT *rows_start = nullptr, *rows_end = nullptr, *columns = nullptr;
  scalar_t* values = nullptr;
  at::mkl::sparse::export_csr(
      mkl_desc,
      &indexing,
      &rows,
      &cols,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are indices_to_mkl_compatible_inplace, unsafeGetTensorImpl, col_indices_and_values_resize_, mkl_result_copy_, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 indices_to_mkl_compatible_inplace, unsafeGetTensorImpl, col_indices_and_values_resize_, mkl_result_copy_，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-180
```cpp
      &rows_start,
      &rows_end,
      &columns,
      &values);

  // Resize input using nnz information from MKL
  MKL_INT nnz = rows_end[rows - 1];
  col_indices_and_values_resize_(input, nnz);

  auto crow_indices = input.crow_indices();
  auto col_indices = input.col_indices();
  auto input_values = input.values();

  // NB: When nnz is zero it is possible that input_values.data_ptr<scalar_t> is
  // a nullptr, if input was created via empty. As such we need to check that
  // nnz is not zero to avoid passing nullptr to std::memcpy. We will apply
  // the same precautions to crow_indices.data_ptr<MKL_INT>.
  //
  // Otherwise ASAN will complain.

  if (nnz > 0) {
    // MKL Sparse Inspector-Executor doesn't have a way to provide external
    // buffers So we have to copy the memory allocated by MKL
    std::memcpy(
        input_values.mutable_data_ptr<scalar_t>(), values, nnz * sizeof(scalar_t));
    std::memcpy(
        col_indices.mutable_data_ptr<MKL_INT>(), columns, nnz * sizeof(MKL_INT));
  }
  if (rows > 0) {
    std::memcpy(
        crow_indices.mutable_data_ptr<MKL_INT>(), rows_start, rows * sizeof(MKL_INT));
  }
  crow_indices.mutable_data_ptr<MKL_INT>()[rows] = nnz;
}
#endif

/*
  Computes a sparse matrix-dense matrix product defined as
  C <- alpha*(A*B) + beta*C

  Args:
  * `A` - Sparse Tensor storing m x k matrix.
  * `B` - Dense Tensor storing k x n matrix.
  * `C` - [in] Dense Tensor storing matrix of size m x n.
          [out] result of the operation.
*/
void addmm_dense_result(
    const Tensor& A,
    const Tensor& B,
    const Scalar& beta,
    const Scalar& alpha,
    const Tensor& C) {
#if !AT_USE_MKL_SPARSE()
  TORCH_CHECK(
      false,
      "Calling addmm on a sparse CPU tensor requires Linux platform. ",
      "Please use PyTorch built with MKL on Linux.");
#else
  c10::MaybeOwned<Tensor> C_ = prepare_dense_matrix_for_mkl(C);
  IntArrayRef C_strides = C_->strides();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_USE_MKL_SPARSE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_USE_MKL_SPARSE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-240
```cpp
  auto ndim = C_->dim();
  bool is_C_row_major = (C_strides[ndim - 1] == 1);

  // MKL requires same storage layout of matrices
  c10::MaybeOwned<Tensor> B_ = prepare_dense_matrix_for_mkl(B, is_C_row_major);
  IntArrayRef B_strides = B_->strides();
  bool is_B_row_major = (B_strides[ndim - 1] == 1);

  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(!(is_C_row_major ^ is_B_row_major));

  auto order =
      is_C_row_major ? SPARSE_LAYOUT_ROW_MAJOR : SPARSE_LAYOUT_COLUMN_MAJOR;
  auto ldc = is_C_row_major ? C_strides[ndim - 2] : C_strides[ndim - 1];
  auto ldb = is_B_row_major ? B_strides[ndim - 2] : B_strides[ndim - 1];
  auto columns_C = mkl_int_cast(C.size(-1), "columns_C");

  matrix_descr descrA{};
  descrA.type = SPARSE_MATRIX_TYPE_GENERAL;

  AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
      C.scalar_type(), "addmm_out_sparse_csr_impl_mkl", [&] {
        auto beta_ = beta.to<scalar_t>();
        auto alpha_ = alpha.to<scalar_t>();

        auto mkl_sparse_mat =
            at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(A);
        at::mkl::sparse::mm<scalar_t>(
            SPARSE_OPERATION_NON_TRANSPOSE,
            alpha_,
            mkl_sparse_mat.descriptor(),
            descrA,
            order,
            B_->data_ptr<scalar_t>(),
            columns_C,
            ldb,
            beta_,
            C_->data_ptr<scalar_t>(),
            ldc);
      });

  if (!C.is_same(*C_)) {
    C.copy_(*C_);
  }
#endif
}

/*
  Computes a sparse matrix-sparse matrix product with dense result defined as
  C <- alpha*(A*B) + beta*C

  Args:
  * `A` - Sparse Tensor storing m x k matrix.
  * `B` - Sparse Tensor storing k x n matrix.
  * `C` - [in] Dense Tensor storing matrix of size m x n.
          [out] result of the operation.
*/
void addmm_sparse_input_dense_result(
    const Tensor& A,
    const Tensor& B,
    const Scalar& beta,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-300
```cpp
    const Scalar& alpha,
    const Tensor& C) {
#if !AT_USE_MKL_SPARSE()
  TORCH_CHECK(
      false,
      "Calling addmm on a sparse CPU tensor requires Linux platform. ",
      "Please use PyTorch built with MKL on Linux.");
#else
  // MKL function computes C <- A*B
  // So we need a temporary matrix to store the result
  // and then add it to C
  auto C_ = at::empty(C.sizes(), C.options());
  auto order = SPARSE_LAYOUT_ROW_MAJOR;
  auto ldc = C_.stride(-2);

  AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
      C.scalar_type(), "addmm_sparse_input_dense_result", [&] {
        auto mkl_A = at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(A);
        auto mkl_B = at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(B);
        at::mkl::sparse::spmmd<scalar_t>(
            SPARSE_OPERATION_NON_TRANSPOSE,
            mkl_A.descriptor(),
            mkl_B.descriptor(),
            order,
            C_.data_ptr<scalar_t>(),
            ldc);
      });

  // If beta is zero NaN and Inf should not be propagated to the result
  if (beta.toComplexDouble() == 0.) {
    C.zero_();
  } else {
    C.mul_(beta);
  }
  C.add_(C_, alpha);
#endif
}

/*
  Computes a sparse matrix-sparse matrix product defined as
  C <- alpha*(A*B) + beta*C

  Args:
  * `mat1` - Sparse CSR Tensor storing m x k matrix A.
  * `mat2` - Sparse CSR Tensor storing k x n matrix B.
  * `result` - [in] Sparse CSR Tensor storing matrix C of size m x n.
               [out] result of the operation.
*/
void addmm_sparse_result(
    const Tensor& mat1,
    const Tensor& mat2,
    const Scalar& beta,
    const Scalar& alpha,
    const Tensor& result) {
#if !AT_USE_MKL_SPARSE()
  TORCH_CHECK(
      false,
      "Calling add on a sparse CPU tensor requires Linux platform. ",
      "Please use PyTorch built with MKL on Linux.");
#else
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_USE_MKL_SPARSE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_USE_MKL_SPARSE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-360
```cpp
  // Compute beta*result because MKL doesn't do it
  // If beta is zero NaN and Inf should not be propagated to the result
  if (beta.toComplexDouble() == 0.) {
    result.values().zero_();
  } else {
    result.values().mul_(beta);
  }

  // MKL doesn't work with empty matrices
  if (mat1._nnz() == 0 || mat2._nnz() == 0) {
    return;
  }

  // MKL doesn't have an interface to compute alpha*(A*B) + beta*C at once
  Tensor mat1_mat2 = at::zeros(result.sizes(), result.options());
  indices_to_mkl_compatible_inplace(mat1_mat2);

  AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
      result.scalar_type(), "addmm_out_sparse_csr_impl_mkl_sparse", [&] {
        auto mkl_sparse_mat1 =
            at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(mat1);
        auto mkl_sparse_mat2 =
            at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(mat2);
        auto mkl_result = at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>();
        auto result_desc = mkl_result.descriptor();

        TORCH_MKLSPARSE_CHECK(mkl_sparse_spmm(
            SPARSE_OPERATION_NON_TRANSPOSE,
            mkl_sparse_mat1.descriptor(),
            mkl_sparse_mat2.descriptor(),
            &result_desc));

        // copy the data from MKL, otherwise computed result will be destroyed
        // together with `mkl_result`
        mkl_result_copy_<scalar_t>(mat1_mat2, result_desc);
      });

  result.add_(mat1_mat2, alpha);
#endif
}

} // anonymous namespace

/*
  Computes a matrix-matrix product defined as
  C <- alpha*(A*B) + beta*C

  Args:
  * `mat1` - Tensor storing m x k matrix A.
  * `mat2` - Tensor storing k x n matrix B.
  * `result` - [in] Tensor storing matrix C of size m x n.
               [out] result of the operation.
*/
void addmm_out_sparse_csr(
    const Tensor& mat1,
    const Tensor& mat2,
    const Scalar& beta,
    const Scalar& alpha,
    const Tensor& result) {
  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 361-420
```cpp
      mat1.dim() == 2 && mat2.dim() == 2 && result.dim() == 2);
  TORCH_INTERNAL_ASSERT(
      !((mat1.layout() == kStrided) && (mat2.layout() == kStrided) &&
        (result.layout() == kStrided)),
      "Expected at least one sparse input");

  // Layout checks are nested mat1, mat2, result
  // Conditions are ordered strided, csr, csc, bsr, bsc.
  // Valid combinations terminate in a return
  // Invalid combinations are omitted and will fall though to the TORCH check
  // generating an informative error message
  if (mat1.layout() == kStrided) {
    if (mat2.layout() == kSparseCsr) {
      if (result.layout() == kStrided) {
        // TODO: Add native CSC support via cuSPARSE if supported.
        addmm_dense_result(
            mat2.transpose(0, 1).to_sparse_csr(),
            mat1.transpose(0, 1),
            beta,
            alpha,
            result.transpose(0, 1));
            return;
      }
    }
    if (mat2.layout() == kSparseCsc) {
      if (result.layout() == kStrided) {
        addmm_dense_result(
            mat2.transpose(-2, -1),
            mat1.transpose(-2, -1),
            beta,
            alpha,
            result.transpose(-2, -1));
            return;
      }
    }
    if (mat2.layout() == kSparseBsc) {
      if (result.layout() == kStrided) {
        addmm_dense_result(
            mat2.transpose(-2, -1),
            mat1.transpose(-2, -1),
            beta,
            alpha,
            result.transpose(-2, -1));
            return;
      }
    }
  }
  if (mat1.layout() == kSparseCsr) {
    if (mat2.layout() == kStrided) {
      if (result.layout() == kStrided) {
        addmm_dense_result(mat1, mat2, beta, alpha, result);
        return;
      }
    }
    if (mat2.layout() == kSparseCsr) {
      if (result.layout() == kStrided) {
        addmm_sparse_input_dense_result(mat1, mat2, beta, alpha, result);
        return;
      }
      if (result.layout() == kSparseCsr) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 421-480
```cpp
        addmm_sparse_result(mat1, mat2, beta, alpha, result);
        return;
      }
    }
    if (mat2.layout() == kSparseCsc) {
      if (result.layout() == kStrided) {
        // TODO: CSR @ CSC kernel would be very fast due to format alignment
        addmm_sparse_input_dense_result(
          mat1, mat2.to_sparse_csr(), beta, alpha, result);
        return;
      }
      if (result.layout() == kSparseCsr) {
        // TODO: CSR @ CSC kernel would be very fast due to format alignment
        addmm_sparse_result(
          mat1, mat2.to_sparse_csr(), beta, alpha, result);
        return;
      }
    }
  }
  if (mat1.layout() == kSparseCsc) {
    if (mat2.layout() == kStrided) {
      if (result.layout() == kStrided) {
        // TODO: avoid csc->csr conversion with native csc support
        addmm_dense_result(
          mat1.to_sparse_csr(), mat2, beta, alpha, result);
        return;
      }
    }
    if (mat2.layout() == kSparseCsr) {
      if (result.layout() == kSparseCsr) {
        // TODO: avoid csc->csr conversion with native csc support
        addmm_sparse_result(
          mat1.to_sparse_csr(), mat2, beta, alpha, result);
        return;
      }
    }
    if (mat2.layout() == kSparseCsc) {
      if (result.layout() == kStrided) {
        addmm_sparse_input_dense_result(
          mat2.transpose(-2, -1),
          mat1.transpose(-2, -1),
          beta,
          alpha,
          result.transpose(-2, -1));
        return;
      }
      if (result.layout() == kSparseCsr) {
        // TODO avoid csc->csr
        addmm_sparse_result(
          mat1.to_sparse_csr(), mat2.to_sparse_csr(), beta, alpha, result);
        return;
      }
      if (result.layout() == kSparseCsc) {
        addmm_sparse_result(
          mat2.transpose(-2, -1),
          mat1.transpose(-2, -1),
          beta,
          alpha,
          result.transpose(-2, -1));
        return;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 481-540
```cpp
      }
    }
  }
  if (mat1.layout() == kSparseBsr) {
    if (mat2.layout() == kStrided) {
      if (result.layout() == kStrided) {
        addmm_dense_result(mat1, mat2, beta, alpha, result);
        return;
      }
    }
  }
  TORCH_CHECK(
    false,
    "addmm: computation on CPU is not implemented for ",
    result.layout(),
    " + ",
    mat1.layout(),
    " @ ",
    mat2.layout());
}

/*
  Computes a sparse matrix-dense vector product defined as
  y <- alpha*op(A)*x + beta*y

  Args:
  * `mat` - Tensor storing sparse m x n matrix A.
  * `vec` - Tensor storing dense vector x of size n.
  * `result` - [in] Tensor storing dense vector y of size m.
               [out] result of the operation.
*/
void addmv_out_sparse_csr(
  const Tensor& mat,
  const Tensor& vec,
  const Scalar& beta,
  const Scalar& alpha,
  const Tensor& result) {
#if !AT_USE_MKL_SPARSE()
  TORCH_CHECK(
    false,
    "Calling addmv on a sparse CPU tensor requires Linux platform. ",
    "Please use PyTorch built with MKL on Linux.");
#else
  c10::MaybeOwned<Tensor> result_ = prepare_dense_vector_for_mkl(result);
  c10::MaybeOwned<Tensor> vec_ = prepare_dense_vector_for_mkl(vec);

  sparse_operation_t opA = SPARSE_OPERATION_NON_TRANSPOSE;
  matrix_descr descrA{};
  descrA.type = SPARSE_MATRIX_TYPE_GENERAL;

  AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
      result.scalar_type(), "addmv_out_sparse_csr_impl_mkl", [&] {
        auto beta_ = beta.to<scalar_t>();
        auto alpha_ = alpha.to<scalar_t>();

        auto mkl_sparse_mat =
            at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(mat);

        at::mkl::sparse::mv<scalar_t>(
            opA,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_USE_MKL_SPARSE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_USE_MKL_SPARSE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 541-600
```cpp
            alpha_,
            mkl_sparse_mat.descriptor(),
            descrA,
            vec_->data_ptr<scalar_t>(),
            beta_,
            result_->data_ptr<scalar_t>());
      });

  if (!result.is_same(*result_)) {
    result.copy_(*result_);
  }
#endif
}

void add_out_sparse_csr(
    const Tensor& mat1,
    const Tensor& mat2,
    const Scalar& alpha,
    const Tensor& result) {
#if !AT_USE_MKL_SPARSE()
  TORCH_CHECK(
      false,
      "Calling add on a sparse CPU tensor requires Linux platform. ",
      "Please use PyTorch built with MKL on Linux.");
#else

  // MKL doesn't work with empty matrices
  if (mat2._nnz() == 0) {
    col_indices_and_values_resize_(result, mat1._nnz());
    result.copy_(mat1);
    return;
  } else if (mat1._nnz() == 0) {
    col_indices_and_values_resize_(result, mat2._nnz());
    result.copy_(mat2);
    result.values().mul_(alpha);
    return;
  }

  // Modify `result` tensor in-place to swap indices tensors with 32-bit (or
  // 64-bit) variants
  const auto output_indices_dtype = promoteTypes(mat1.crow_indices().scalar_type(), mat2.crow_indices().scalar_type());
  auto result_crow_indices_backup = result.crow_indices();
  auto result_col_indices_backup = result.col_indices();
  indices_to_mkl_compatible_inplace(result);
  sparse_operation_t opA = SPARSE_OPERATION_NON_TRANSPOSE;

  AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
      result.scalar_type(), "add_out_sparse_csr_impl_mkl", [&] {
        auto alpha_ = alpha.to<scalar_t>();

        auto mkl_mat1 = at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(mat1);
        auto mkl_mat2 = at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(mat2);
        auto mkl_result = at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>();

        // Note that the order the order of mat1 and mat2 arguments is swapped
        // because MKL computes alpha*mat1 + mat2 while PyTorch needs mat1 +
        // alpha*mat2
        auto result_desc = mkl_result.descriptor();
        at::mkl::sparse::add<scalar_t>(
            opA,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_USE_MKL_SPARSE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_USE_MKL_SPARSE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 601-660
```cpp
            mkl_mat2.descriptor(),
            alpha_,
            mkl_mat1.descriptor(),
            &result_desc);

        // now copy data from `result_desc` to `result`
        mkl_result_copy_<scalar_t>(result, result_desc);
      });

  if (output_indices_dtype == at::kLong) {
    const auto res_nnz = result._nnz();
    static_cast<SparseCsrTensorImpl*>(result.unsafeGetTensorImpl())->set_member_tensors(
        result_crow_indices_backup.copy_(result.crow_indices()),
        result_col_indices_backup.resize_({res_nnz}).copy_(result.col_indices()),
        result.values(),
        result.sizes());
  }
#endif
}

void triangular_solve_out_sparse_csr(
    const Tensor& A_,
    const Tensor& B,
    const Tensor& X,
    bool upper,
    bool transpose,
    bool unitriangular) {
#if !AT_USE_MKL_SPARSE()
  TORCH_CHECK(
      false,
      "Calling triangular_solve on a sparse CPU tensor requires Linux platform. ",
      "Please use PyTorch built with MKL on Linux.");
#else
  if (B.numel() == 0 || X.numel() == 0 || A_._nnz() == 0) {
    // If A has no nnz, then A is singular and we can't solve.
    X.fill_(NAN);
    return;
  }

  const auto materialize_diagonal_indices = [](const Tensor& t) -> Tensor {
    const auto n = t.size(-1);
    const auto compressed_indices = std::get<0>(at::sparse_csr::getCompressedPlainIndices(t));
    const auto diag_indices = at::arange(n, compressed_indices.options()).unsqueeze(0).expand({2, n});
    const auto diag_values = at::zeros({1}, t.values().options()).expand({n});

    const auto t_coo = t.to_sparse();
    const auto expanded_indices = at::cat({t_coo._indices(), diag_indices}, /*dim=*/-1);
    const auto expanded_values = at::cat({t_coo._values(), diag_values}, /*dim=*/0);

    const auto t_expanded_coo = at::sparse_coo_tensor(expanded_indices, expanded_values, t_coo.sizes(), t_coo.options());
    return t_expanded_coo.to_sparse(t.layout());
  };

  // MKL has a bug for inputs with unmaterialized diagonal indices.
  // See https://github.com/pytorch/pytorch/issues/88890 and
  // the comments within.
  const auto A = unitriangular ? materialize_diagonal_indices(A_) : A_;

  c10::MaybeOwned<Tensor> X_ = prepare_dense_matrix_for_mkl(X);
  IntArrayRef X_strides = X_->strides();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_USE_MKL_SPARSE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_USE_MKL_SPARSE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 661-720
```cpp
  auto ndim = X_->dim();
  bool is_X_row_major = (ndim > 1) ? (X_strides[ndim - 1] == 1) : true;

  // MKL requires same storage layout of matrices
  c10::MaybeOwned<Tensor> B_ = prepare_dense_matrix_for_mkl(B, is_X_row_major);

  sparse_operation_t opA = transpose ? SPARSE_OPERATION_TRANSPOSE : SPARSE_OPERATION_NON_TRANSPOSE;
  matrix_descr descrA{};
  descrA.type = SPARSE_MATRIX_TYPE_TRIANGULAR;
  descrA.mode = upper ? SPARSE_FILL_MODE_UPPER : SPARSE_FILL_MODE_LOWER;
  descrA.diag = unitriangular ? SPARSE_DIAG_UNIT : SPARSE_DIAG_NON_UNIT;

  AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(
      X.scalar_type(), "triangular_solve_out_sparse_csr_impl_mkl", [&] {
        auto mkl_sparse_mat =
            at::mkl::sparse::MklSparseCsrDescriptor<scalar_t>(A);
        scalar_t alpha = 1;

        if (B.size(-1) == 1) {
          sparse_status_t status = at::mkl::sparse::trsv<scalar_t>(
              opA,
              alpha,
              mkl_sparse_mat.descriptor(),
              descrA,
              B_->data_ptr<scalar_t>(),
              X_->data_ptr<scalar_t>());
          // Emulate behavior of old MKL version that would set all elements of output array to -NaN
          // in case of invalid input matrices.
          if (status == SPARSE_STATUS_INVALID_VALUE) {
            X_->fill_(-std::numeric_limits<scalar_t>::quiet_NaN());
          }
        } else {
          IntArrayRef B_strides = B_->strides();
          bool is_B_row_major = (B_strides[ndim - 1] == 1);
          TORCH_INTERNAL_ASSERT_DEBUG_ONLY(!(is_X_row_major ^ is_B_row_major));

          auto order = is_X_row_major ? SPARSE_LAYOUT_ROW_MAJOR : SPARSE_LAYOUT_COLUMN_MAJOR;
          auto nrhs = mkl_int_cast(B.size(-1), "nrhs");
          auto ldx = is_X_row_major ? X_strides[ndim - 2] : X_strides[ndim - 1];
          auto ldb = is_B_row_major ? B_strides[ndim - 2] : B_strides[ndim - 1];
          sparse_status_t status = at::mkl::sparse::trsm<scalar_t>(
              opA,
              alpha,
              mkl_sparse_mat.descriptor(),
              descrA,
              order,
              B_->data_ptr<scalar_t>(),
              nrhs,
              ldb,
              X_->data_ptr<scalar_t>(),
              ldx);
          // Emulate behavior of old MKL version that would set all elements of output array to -NaN
          // in case of invalid input matrices.
          if (status == SPARSE_STATUS_INVALID_VALUE) {
            X_->fill_(-std::numeric_limits<scalar_t>::quiet_NaN());
          }
        }
      });

  if (!X.is_same(*X_)) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 721-726
```cpp
    X.copy_(*X_);
  }
#endif
}

} // namespace at
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。

## Key Concepts / 关键概念

- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Backend focus: sparse tensor support.
- CN: 后端重点：稀疏张量支持。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: prepare_dense_matrix_for_mkl, prepare_dense_vector_for_mkl, indices_to_mkl_compatible_inplace, col_indices_and_values_resize_, mkl_result_copy_, addmm_dense_result, addmm_sparse_input_dense_result, addmm_sparse_result.
- CN: 重要符号：prepare_dense_matrix_for_mkl, prepare_dense_vector_for_mkl, indices_to_mkl_compatible_inplace, col_indices_and_values_resize_, mkl_result_copy_, addmm_dense_result, addmm_sparse_input_dense_result, addmm_sparse_result。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/Dispatch.h, ATen/SparseCsrTensorImpl.h, ATen/Tensor.h, ATen/mkl/Sparse.h, ATen/native/LinearAlgebraUtils.h, ATen/SparseCsrTensorUtils.h, ATen/native/mkl/SparseBlasImpl.h, c10/core/ScalarType.h, c10/util/MaybeOwned.h, ATen/mkl/SparseBlas.h, ATen/mkl/SparseDescriptors.h, ATen/mkl/Utils.h`.
- CN: 主要内部头文件：`ATen/Dispatch.h, ATen/SparseCsrTensorImpl.h, ATen/Tensor.h, ATen/mkl/Sparse.h, ATen/native/LinearAlgebraUtils.h, ATen/SparseCsrTensorUtils.h, ATen/native/mkl/SparseBlasImpl.h, c10/core/ScalarType.h, c10/util/MaybeOwned.h, ATen/mkl/SparseBlas.h, ATen/mkl/SparseDescriptors.h, ATen/mkl/Utils.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `prepare_dense_matrix_for_mkl, prepare_dense_vector_for_mkl, indices_to_mkl_compatible_inplace, col_indices_and_values_resize_, mkl_result_copy_, addmm_dense_result, addmm_sparse_input_dense_result, addmm_sparse_result, addmm_out_sparse_csr, addmv_out_sparse_csr`.
- CN: 实现围绕 `prepare_dense_matrix_for_mkl, prepare_dense_vector_for_mkl, indices_to_mkl_compatible_inplace, col_indices_and_values_resize_, mkl_result_copy_, addmm_dense_result, addmm_sparse_input_dense_result, addmm_sparse_result, addmm_out_sparse_csr, addmv_out_sparse_csr` 等符号展开。
