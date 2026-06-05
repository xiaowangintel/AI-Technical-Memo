# RunnerUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/ExecutionEngine/RunnerUtils.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR RunnerUtils component. The leading comments describe it as: This file declares basic classes and functions to debug structured MLIR.
- **用途（CN）**: 声明 MLIR RunnerUtils 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````cpp
//===- RunnerUtils.h - Utils for debugging MLIR execution -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares basic classes and functions to debug structured MLIR
// types at runtime. Entities in this file may not be compatible with targets
// without a C++ runtime. These may be progressively migrated to CRunnerUtils.h
// over time.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_EXECUTIONENGINE_RUNNERUTILS_H
#define MLIR_EXECUTIONENGINE_RUNNERUTILS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 19-19
````cpp
#ifdef _WIN32
````
- **EN**: This section focuses on #ifdef _win32, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“#ifdef _WIN32”这一主题，把相关声明与辅助接口组织在一起。

### Lines 20-28
````cpp
#ifndef MLIR_RUNNERUTILS_EXPORT
#ifdef mlir_runner_utils_EXPORTS
// We are building this library
#define MLIR_RUNNERUTILS_EXPORT __declspec(dllexport)
#else
// We are using this library
#define MLIR_RUNNERUTILS_EXPORT __declspec(dllimport)
#endif // mlir_runner_utils_EXPORTS
#endif // MLIR_RUNNERUTILS_EXPORT
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 29-41
````cpp
#else
// Non-windows: use visibility attributes.
#define MLIR_RUNNERUTILS_EXPORT __attribute__((visibility("default")))
#endif // _WIN32

#include <assert.h>
#include <cmath>
#include <complex>
#include <iomanip>
#include <iostream>

#include "mlir/ExecutionEngine/CRunnerUtils.h"
#include "mlir/ExecutionEngine/Float16bits.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 43-62
````cpp
template <typename T, typename StreamType>
void printMemRefMetaData(StreamType &os, const DynamicMemRefType<T> &v) {
  // Make the printed pointer format platform independent by casting it to an
  // integer and manually formatting it to a hex with prefix as tests expect.
  os << "base@ = " << std::hex << std::showbase
     << reinterpret_cast<std::intptr_t>(v.data) << std::dec << std::noshowbase
     << " rank = " << v.rank << " offset = " << v.offset;
  auto print = [&](const int64_t *ptr) {
    if (v.rank == 0)
      return;
    os << ptr[0];
    for (int64_t i = 1; i < v.rank; ++i)
      os << ", " << ptr[i];
  };
  os << " sizes = [";
  print(v.sizes);
  os << "] strides = [";
  print(v.strides);
  os << "]";
}
````
- **EN**: This block groups callable interfaces such as `printMemRefMetaData`, `print`, indicating how `RunnerUtils` is queried or updated.
- **CN**: 该代码块聚合了 `printMemRefMetaData`, `print` 等可调用接口，展示了如何查询或更新 `RunnerUtils`。

### Lines 64-69
````cpp
template <typename StreamType, typename T, int N>
void printMemRefMetaData(StreamType &os, StridedMemRefType<T, N> &v) {
  static_assert(N >= 0, "Expected N > 0");
  os << "MemRef ";
  printMemRefMetaData(os, DynamicMemRefType<T>(v));
}
````
- **EN**: This block groups callable interfaces such as `printMemRefMetaData`, `static_assert`, indicating how `RunnerUtils` is queried or updated.
- **CN**: 该代码块聚合了 `printMemRefMetaData`, `static_assert` 等可调用接口，展示了如何查询或更新 `RunnerUtils`。

### Lines 71-134
````cpp
template <typename StreamType, typename T>
void printUnrankedMemRefMetaData(StreamType &os, UnrankedMemRefType<T> &v) {
  os << "Unranked MemRef ";
  printMemRefMetaData(os, DynamicMemRefType<T>(v));
}

////////////////////////////////////////////////////////////////////////////////
// Templated instantiation follows.
////////////////////////////////////////////////////////////////////////////////
namespace impl {
using index_type = uint64_t;
using complex64 = std::complex<double>;
using complex32 = std::complex<float>;

template <typename T, int M, int... Dims>
std::ostream &operator<<(std::ostream &os, const Vector<T, M, Dims...> &v);

template <int... Dims>
struct StaticSizeMult {
  static constexpr int value = 1;
};

template <int N, int... Dims>
struct StaticSizeMult<N, Dims...> {
  static constexpr int value = N * StaticSizeMult<Dims...>::value;
};

static inline void printSpace(std::ostream &os, int count) {
  for (int i = 0; i < count; ++i) {
    os << ' ';
  }
}

template <typename T, int M, int... Dims>
struct VectorDataPrinter {
  static void print(std::ostream &os, const Vector<T, M, Dims...> &val);
};

template <typename T, int M, int... Dims>
void VectorDataPrinter<T, M, Dims...>::print(std::ostream &os,
                                             const Vector<T, M, Dims...> &val) {
  static_assert(M > 0, "0 dimensioned tensor");
  static_assert(sizeof(val) == M * StaticSizeMult<Dims...>::value * sizeof(T),
                "Incorrect vector size!");
  // First
  os << "(" << val[0];
  if (M > 1)
    os << ", ";
  if (sizeof...(Dims) > 1)
    os << "\n";
  // Kernel
  for (unsigned i = 1; i + 1 < M; ++i) {
    printSpace(os, 2 * sizeof...(Dims));
    os << val[i] << ", ";
    if (sizeof...(Dims) > 1)
      os << "\n";
  }
  // Last
  if (M > 1) {
    printSpace(os, sizeof...(Dims));
    os << val[M - 1];
  }
  os << ")";
}
````
- **EN**: This C++ declaration introduces `StaticSizeMult` and establishes part of the API surface for `RunnerUtils`. Representative entry points here include `printUnrankedMemRefMetaData`, `printMemRefMetaData`, `printSpace`, `print`.
- **CN**: 该 C++ 声明引入了 `StaticSizeMult`，并构成 `RunnerUtils` API 表面的一部分。 这一段可见的代表性接口包括 `printUnrankedMemRefMetaData`, `printMemRefMetaData`, `printSpace`, `print`。

### Lines 135-191
````cpp
template <typename T, int M, int... Dims>
std::ostream &operator<<(std::ostream &os, const Vector<T, M, Dims...> &v) {
  VectorDataPrinter<T, M, Dims...>::print(os, v);
  return os;
}

template <typename T>
struct MemRefDataPrinter {
  static void print(std::ostream &os, T *base, int64_t dim, int64_t rank,
                    int64_t offset, const int64_t *sizes,
                    const int64_t *strides);
  static void printFirst(std::ostream &os, T *base, int64_t dim, int64_t rank,
                         int64_t offset, const int64_t *sizes,
                         const int64_t *strides);
  static void printLast(std::ostream &os, T *base, int64_t dim, int64_t rank,
                        int64_t offset, const int64_t *sizes,
                        const int64_t *strides);
};

template <typename T>
void MemRefDataPrinter<T>::printFirst(std::ostream &os, T *base, int64_t dim,
                                      int64_t rank, int64_t offset,
                                      const int64_t *sizes,
                                      const int64_t *strides) {
  os << "[";
  print(os, base, dim - 1, rank, offset, sizes + 1, strides + 1);
  // If single element, close square bracket and return early.
  if (sizes[0] <= 1) {
    os << "]";
    return;
  }
  os << ", ";
  if (dim > 1)
    os << "\n";
}

template <typename T>
void MemRefDataPrinter<T>::print(std::ostream &os, T *base, int64_t dim,
                                 int64_t rank, int64_t offset,
                                 const int64_t *sizes, const int64_t *strides) {
  if (dim == 0) {
    os << base[offset];
    return;
  }
  printFirst(os, base, dim, rank, offset, sizes, strides);
  for (unsigned i = 1; i + 1 < sizes[0]; ++i) {
    printSpace(os, rank - dim + 1);
    print(os, base, dim - 1, rank, offset + i * strides[0], sizes + 1,
          strides + 1);
    os << ", ";
    if (dim > 1)
      os << "\n";
  }
  if (sizes[0] <= 1)
    return;
  printLast(os, base, dim, rank, offset, sizes, strides);
}
````
- **EN**: This C++ declaration introduces `MemRefDataPrinter` and establishes part of the API surface for `RunnerUtils`. Representative entry points here include `print`, `printFirst`, `printLast`, `printSpace`.
- **CN**: 该 C++ 声明引入了 `MemRefDataPrinter`，并构成 `RunnerUtils` API 表面的一部分。 这一段可见的代表性接口包括 `print`, `printFirst`, `printLast`, `printSpace`。

### Lines 193-239
````cpp
template <typename T>
void MemRefDataPrinter<T>::printLast(std::ostream &os, T *base, int64_t dim,
                                     int64_t rank, int64_t offset,
                                     const int64_t *sizes,
                                     const int64_t *strides) {
  printSpace(os, rank - dim + 1);
  print(os, base, dim - 1, rank, offset + (sizes[0] - 1) * (*strides),
        sizes + 1, strides + 1);
  os << "]";
}

template <typename T, int N>
void printMemRefShape(StridedMemRefType<T, N> &m) {
  std::cout << "Memref ";
  printMemRefMetaData(std::cout, DynamicMemRefType<T>(m));
}

template <typename T>
void printMemRefShape(UnrankedMemRefType<T> &m) {
  std::cout << "Unranked Memref ";
  printMemRefMetaData(std::cout, DynamicMemRefType<T>(m));
}

template <typename T>
void printMemRef(const DynamicMemRefType<T> &m) {
  printMemRefMetaData(std::cout, m);
  std::cout << " data = \n";
  if (m.rank == 0)
    std::cout << "[";
  MemRefDataPrinter<T>::print(std::cout, m.data, m.rank, m.rank, m.offset,
                              m.sizes, m.strides);
  if (m.rank == 0)
    std::cout << "]";
  std::cout << '\n' << std::flush;
}

template <typename T, int N>
void printMemRef(StridedMemRefType<T, N> &m) {
  std::cout << "Memref ";
  printMemRef(DynamicMemRefType<T>(m));
}

template <typename T>
void printMemRef(UnrankedMemRefType<T> &m) {
  std::cout << "Unranked Memref ";
  printMemRef(DynamicMemRefType<T>(m));
}
````
- **EN**: This block groups callable interfaces such as `printLast`, `printSpace`, `print`, `printMemRefShape`, indicating how `RunnerUtils` is queried or updated.
- **CN**: 该代码块聚合了 `printLast`, `printSpace`, `print`, `printMemRefShape` 等可调用接口，展示了如何查询或更新 `RunnerUtils`。

### Lines 241-286
````cpp
/// Verify the result of two computations are equivalent up to a small
/// numerical error and return the number of errors.
template <typename T>
struct MemRefDataVerifier {
  /// Maximum number of errors printed by the verifier.
  static constexpr int printLimit = 10;

  /// Verify the relative difference of the values is smaller than epsilon.
  static bool verifyRelErrorSmallerThan(T actual, T expected, T epsilon);

  /// Verify the values are equivalent (integers) or are close (floating-point).
  static bool verifyElem(T actual, T expected);

  /// Verify the data element-by-element and return the number of errors.
  static int64_t verify(std::ostream &os, T *actualBasePtr, T *expectedBasePtr,
                        int64_t dim, int64_t offset, const int64_t *sizes,
                        const int64_t *strides, int64_t &printCounter);
};

template <typename T>
bool MemRefDataVerifier<T>::verifyRelErrorSmallerThan(T actual, T expected,
                                                      T epsilon) {
  // Return an error if one of the values is infinite or NaN.
  if (!std::isfinite(actual) || !std::isfinite(expected))
    return false;
  // Return true if the relative error is smaller than epsilon.
  T delta = std::abs(actual - expected);
  return (delta <= epsilon * std::abs(expected));
}

template <typename T>
bool MemRefDataVerifier<T>::verifyElem(T actual, T expected) {
  return actual == expected;
}

template <>
inline bool MemRefDataVerifier<double>::verifyElem(double actual,
                                                   double expected) {
  return verifyRelErrorSmallerThan(actual, expected, 1e-12);
}

template <>
inline bool MemRefDataVerifier<float>::verifyElem(float actual,
                                                  float expected) {
  return verifyRelErrorSmallerThan(actual, expected, 1e-6f);
}
````
- **EN**: This C++ declaration introduces `MemRefDataVerifier` and establishes part of the API surface for `RunnerUtils`. Representative entry points here include `verifyRelErrorSmallerThan`, `verifyElem`, `verify`, `isfinite`.
- **CN**: 该 C++ 声明引入了 `MemRefDataVerifier`，并构成 `RunnerUtils` API 表面的一部分。 这一段可见的代表性接口包括 `verifyRelErrorSmallerThan`, `verifyElem`, `verify`, `isfinite`。

### Lines 288-336
````cpp
template <typename T>
int64_t MemRefDataVerifier<T>::verify(std::ostream &os, T *actualBasePtr,
                                      T *expectedBasePtr, int64_t dim,
                                      int64_t offset, const int64_t *sizes,
                                      const int64_t *strides,
                                      int64_t &printCounter) {
  int64_t errors = 0;
  // Verify the elements at the current offset.
  if (dim == 0) {
    if (!verifyElem(actualBasePtr[offset], expectedBasePtr[offset])) {
      if (printCounter < printLimit) {
        os << actualBasePtr[offset] << " != " << expectedBasePtr[offset]
           << " offset = " << offset << "\n";
        printCounter++;
      }
      errors++;
    }
  } else {
    // Iterate the current dimension and verify recursively.
    for (int64_t i = 0; i < sizes[0]; ++i) {
      errors +=
          verify(os, actualBasePtr, expectedBasePtr, dim - 1,
                 offset + i * strides[0], sizes + 1, strides + 1, printCounter);
    }
  }
  return errors;
}

/// Verify the equivalence of two dynamic memrefs and return the number of
/// errors or -1 if the shape of the memrefs do not match.
template <typename T>
int64_t verifyMemRef(const DynamicMemRefType<T> &actual,
                     const DynamicMemRefType<T> &expected) {
  // Check if the memref shapes match.
  for (int64_t i = 0; i < actual.rank; ++i) {
    if (expected.rank != actual.rank || actual.offset != expected.offset ||
        actual.sizes[i] != expected.sizes[i] ||
        actual.strides[i] != expected.strides[i]) {
      printMemRefMetaData(std::cerr, actual);
      printMemRefMetaData(std::cerr, expected);
      return -1;
    }
  }
  // Return the number of errors.
  int64_t printCounter = 0;
  return MemRefDataVerifier<T>::verify(std::cerr, actual.data, expected.data,
                                       actual.rank, actual.offset, actual.sizes,
                                       actual.strides, printCounter);
}
````
- **EN**: This block groups callable interfaces such as `verify`, `verifyElem`, `verifyMemRef`, `printMemRefMetaData`, indicating how `RunnerUtils` is queried or updated.
- **CN**: 该代码块聚合了 `verify`, `verifyElem`, `verifyMemRef`, `printMemRefMetaData` 等可调用接口，展示了如何查询或更新 `RunnerUtils`。

### Lines 338-390
````cpp
/// Verify the equivalence of two unranked memrefs and return the number of
/// errors or -1 if the shape of the memrefs do not match.
template <typename T>
int64_t verifyMemRef(UnrankedMemRefType<T> &actual,
                     UnrankedMemRefType<T> &expected) {
  return verifyMemRef(DynamicMemRefType<T>(actual),
                      DynamicMemRefType<T>(expected));
}

} // namespace impl

////////////////////////////////////////////////////////////////////////////////
// Currently exposed C API.
////////////////////////////////////////////////////////////////////////////////
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeI8(UnrankedMemRefType<int8_t> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeI32(UnrankedMemRefType<int32_t> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeI64(UnrankedMemRefType<int64_t> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeF32(UnrankedMemRefType<float> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeF64(UnrankedMemRefType<double> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeInd(UnrankedMemRefType<impl::index_type> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeC32(UnrankedMemRefType<impl::complex32> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefShapeC64(UnrankedMemRefType<impl::complex64> *m);

extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefI8(UnrankedMemRefType<int8_t> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefI16(UnrankedMemRefType<int16_t> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefI32(UnrankedMemRefType<int32_t> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefI64(UnrankedMemRefType<int64_t> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefF16(UnrankedMemRefType<f16> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefBF16(UnrankedMemRefType<bf16> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefF32(UnrankedMemRefType<float> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefF64(UnrankedMemRefType<double> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefInd(UnrankedMemRefType<impl::index_type> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefC32(UnrankedMemRefType<impl::complex32> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemrefC64(UnrankedMemRefType<impl::complex64> *m);
````
- **EN**: This block groups callable interfaces such as `verifyMemRef`, `_mlir_ciface_printMemrefShapeI8`, `_mlir_ciface_printMemrefShapeI32`, `_mlir_ciface_printMemrefShapeI64`, indicating how `RunnerUtils` is queried or updated.
- **CN**: 该代码块聚合了 `verifyMemRef`, `_mlir_ciface_printMemrefShapeI8`, `_mlir_ciface_printMemrefShapeI32`, `_mlir_ciface_printMemrefShapeI64` 等可调用接口，展示了如何查询或更新 `RunnerUtils`。

### Lines 392-455
````cpp
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_nanoTime();

extern "C" MLIR_RUNNERUTILS_EXPORT void printMemrefI32(int64_t rank, void *ptr);
extern "C" MLIR_RUNNERUTILS_EXPORT void printMemrefI64(int64_t rank, void *ptr);
extern "C" MLIR_RUNNERUTILS_EXPORT void printMemrefF32(int64_t rank, void *ptr);
extern "C" MLIR_RUNNERUTILS_EXPORT void printMemrefF64(int64_t rank, void *ptr);
extern "C" MLIR_RUNNERUTILS_EXPORT void printMemrefInd(int64_t rank, void *ptr);
extern "C" MLIR_RUNNERUTILS_EXPORT void printMemrefC32(int64_t rank, void *ptr);
extern "C" MLIR_RUNNERUTILS_EXPORT void printMemrefC64(int64_t rank, void *ptr);

extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref0dF32(StridedMemRefType<float, 0> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dF32(StridedMemRefType<float, 1> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref2dF32(StridedMemRefType<float, 2> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref3dF32(StridedMemRefType<float, 3> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref4dF32(StridedMemRefType<float, 4> *m);

extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dI8(StridedMemRefType<int8_t, 1> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dI32(StridedMemRefType<int32_t, 1> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dI64(StridedMemRefType<int64_t, 1> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dF64(StridedMemRefType<double, 1> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dInd(StridedMemRefType<impl::index_type, 1> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dC32(StridedMemRefType<impl::complex32, 1> *m);
extern "C" MLIR_RUNNERUTILS_EXPORT void
_mlir_ciface_printMemref1dC64(StridedMemRefType<impl::complex64, 1> *m);

extern "C" MLIR_RUNNERUTILS_EXPORT void _mlir_ciface_printMemrefVector4x4xf32(
    StridedMemRefType<Vector2D<4, 4, float>, 2> *m);

extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefI8(
    UnrankedMemRefType<int8_t> *actual, UnrankedMemRefType<int8_t> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefI16(
    UnrankedMemRefType<int16_t> *actual, UnrankedMemRefType<int16_t> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefI32(
    UnrankedMemRefType<int32_t> *actual, UnrankedMemRefType<int32_t> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefI64(
    UnrankedMemRefType<int64_t> *actual, UnrankedMemRefType<int64_t> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefBF16(
    UnrankedMemRefType<bf16> *actual, UnrankedMemRefType<bf16> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefF16(
    UnrankedMemRefType<f16> *actual, UnrankedMemRefType<f16> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefF32(
    UnrankedMemRefType<float> *actual, UnrankedMemRefType<float> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t _mlir_ciface_verifyMemRefF64(
    UnrankedMemRefType<double> *actual, UnrankedMemRefType<double> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t
_mlir_ciface_verifyMemRefInd(UnrankedMemRefType<impl::index_type> *actual,
                             UnrankedMemRefType<impl::index_type> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t
_mlir_ciface_verifyMemRefC32(UnrankedMemRefType<impl::complex32> *actual,
                             UnrankedMemRefType<impl::complex32> *expected);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t
_mlir_ciface_verifyMemRefC64(UnrankedMemRefType<impl::complex64> *actual,
                             UnrankedMemRefType<impl::complex64> *expected);
````
- **EN**: This block groups callable interfaces such as `_mlir_ciface_nanoTime`, `printMemrefI32`, `printMemrefI64`, `printMemrefF32`, indicating how `RunnerUtils` is queried or updated.
- **CN**: 该代码块聚合了 `_mlir_ciface_nanoTime`, `printMemrefI32`, `printMemrefI64`, `printMemrefF32` 等可调用接口，展示了如何查询或更新 `RunnerUtils`。

### Lines 457-474
````cpp
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t verifyMemRefI32(int64_t rank,
                                                           void *actualPtr,
                                                           void *expectedPtr);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t verifyMemRefF32(int64_t rank,
                                                           void *actualPtr,
                                                           void *expectedPtr);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t verifyMemRefF64(int64_t rank,
                                                           void *actualPtr,
                                                           void *expectedPtr);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t verifyMemRefInd(int64_t rank,
                                                           void *actualPtr,
                                                           void *expectedPtr);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t verifyMemRefC32(int64_t rank,
                                                           void *actualPtr,
                                                           void *expectedPtr);
extern "C" MLIR_RUNNERUTILS_EXPORT int64_t verifyMemRefC64(int64_t rank,
                                                           void *actualPtr,
                                                           void *expectedPtr);
````
- **EN**: This block groups callable interfaces such as `verifyMemRefI32`, `verifyMemRefF32`, `verifyMemRefF64`, `verifyMemRefInd`, indicating how `RunnerUtils` is queried or updated.
- **CN**: 该代码块聚合了 `verifyMemRefI32`, `verifyMemRefF32`, `verifyMemRefF64`, `verifyMemRefInd` 等可调用接口，展示了如何查询或更新 `RunnerUtils`。

### Lines 477-477
````cpp
#endif // MLIR_EXECUTIONENGINE_RUNNERUTILS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/ExecutionEngine/CRunnerUtils.h
- mlir/ExecutionEngine/Float16bits.h
