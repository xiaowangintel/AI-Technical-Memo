# DiagnosedSilenceableFailure.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/Utils/DiagnosedSilenceableFailure.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR DiagnosedSilenceableFailure component. The leading comments describe it as: This file declares the DiagnosedSilenceableFailure class allowing to store.
- **用途（CN）**: 声明 MLIR DiagnosedSilenceableFailure 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````cpp
//===- DiagnosedSilenceableFailure.h - Tri-state result ----------- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the DiagnosedSilenceableFailure class allowing to store
// a tri-state result (definite failure, recoverable failure, success) with an
// optional associated list of diagnostics.
//
//===----------------------------------------------------------------------===//

#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/Operation.h"
#include <optional>
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 19-20
````cpp
#ifndef MLIR_DIALECT_TRANSFORM_UTILS_DIAGNOSEDSILENCEABLEFAILURE_H
#define MLIR_DIALECT_TRANSFORM_UTILS_DIAGNOSEDSILENCEABLEFAILURE_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 22-68
````cpp
namespace mlir {
/// The result of a transform IR operation application. This can have one of the
/// three states:
///   - success;
///   - silenceable (recoverable) failure with yet-unreported diagnostic;
///   - definite failure.
/// Silenceable failure is intended to communicate information about
/// transformations that did not apply but in a way that supports recovery,
/// for example, they did not modify the payload IR or modified it in some
/// predictable way. They are associated with a Diagnostic that provides more
/// details on the failure. Silenceable failure can be discarded, turning the
/// result into success, or "reported", emitting the diagnostic and turning the
/// result into definite failure.
/// Transform IR operations containing other operations are allowed to do either
/// with the results of the nested transformations, but must propagate definite
/// failures as their diagnostics have been already reported to the user.
class [[nodiscard]] DiagnosedSilenceableFailure {
public:
  DiagnosedSilenceableFailure(const DiagnosedSilenceableFailure &) = delete;
  DiagnosedSilenceableFailure &
  operator=(const DiagnosedSilenceableFailure &) = delete;
  DiagnosedSilenceableFailure(DiagnosedSilenceableFailure &&) = default;
  DiagnosedSilenceableFailure &
  operator=(DiagnosedSilenceableFailure &&) = default;

  /// Constructs a DiagnosedSilenceableFailure in the success state.
  static DiagnosedSilenceableFailure success() {
    return DiagnosedSilenceableFailure(::mlir::success());
  }

  /// Constructs a DiagnosedSilenceableFailure in the failure state. Typically,
  /// a diagnostic has been emitted before this.
  static DiagnosedSilenceableFailure definiteFailure() {
    return DiagnosedSilenceableFailure(::mlir::failure());
  }

  /// Constructs a DiagnosedSilenceableFailure in the silenceable failure state,
  /// ready to emit the given diagnostic. This is considered a failure
  /// regardless of the diagnostic severity.
  static DiagnosedSilenceableFailure silenceableFailure(Diagnostic &&diag) {
    return DiagnosedSilenceableFailure(std::forward<Diagnostic>(diag));
  }
  static DiagnosedSilenceableFailure
  silenceableFailure(SmallVector<Diagnostic> &&diag) {
    return DiagnosedSilenceableFailure(
        std::forward<SmallVector<Diagnostic>>(diag));
  }
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `DiagnosedSilenceableFailure`. Representative entry points here include `DiagnosedSilenceableFailure`, `success`, `definiteFailure`, `failure`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `DiagnosedSilenceableFailure` API 表面的一部分。 这一段可见的代表性接口包括 `DiagnosedSilenceableFailure`, `success`, `definiteFailure`, `failure`。

### Lines 69-114
````cpp
  /// Converts all kinds of failure into a LogicalResult failure, emitting the
  /// diagnostic if necessary. Must not be called more than once.
  LogicalResult checkAndReport();

  /// Returns `true` if this is a success.
  bool succeeded() const {
    return ::mlir::succeeded(result) && diagnostics.empty();
  }

  /// Returns `true` if this is a definite failure.
  bool isDefiniteFailure() const {
    return ::mlir::failed(result) && diagnostics.empty();
  }

  /// Returns `true` if this is a silenceable failure.
  bool isSilenceableFailure() const { return !diagnostics.empty(); }

  /// Returns the diagnostic message without emitting it. Expects this object
  /// to be a silenceable failure.
  std::string getMessage() const {
    std::string res;
    for (auto &diagnostic : diagnostics) {
      res.append(diagnostic.str());
      res.append("\n");
    }
    return res;
  }

  /// Returns a string representation of the failure mode (for error reporting).
  std::string getStatusString() const {
    if (succeeded())
      return "success";
    if (isSilenceableFailure())
      return "silenceable failure";
    return "definite failure";
  }

  /// Converts silenceable failure into LogicalResult success without reporting
  /// the diagnostic, preserves the other states.
  LogicalResult silence() {
    if (!diagnostics.empty()) {
      diagnostics.clear();
      result = ::mlir::success();
    }
    return result;
  }
````
- **EN**: This block groups callable interfaces such as `checkAndReport`, `succeeded`, `empty`, `isDefiniteFailure`, indicating how `DiagnosedSilenceableFailure` is queried or updated.
- **CN**: 该代码块聚合了 `checkAndReport`, `succeeded`, `empty`, `isDefiniteFailure` 等可调用接口，展示了如何查询或更新 `DiagnosedSilenceableFailure`。

### Lines 116-161
````cpp
  /// Take the diagnostics and silence.
  void takeDiagnostics(SmallVectorImpl<Diagnostic> &diags) {
    assert(!diagnostics.empty() && "expected a diagnostic to be present");
    diags.append(std::make_move_iterator(diagnostics.begin()),
                 std::make_move_iterator(diagnostics.end()));
  }

  /// Streams the given values into the last diagnostic.
  /// Expects this object to be a silenceable failure.
  template <typename T>
  DiagnosedSilenceableFailure &operator<<(T &&value) & {
    assert(isSilenceableFailure() &&
           "can only append output in silenceable failure state");
    diagnostics.back() << std::forward<T>(value);
    return *this;
  }
  template <typename T>
  DiagnosedSilenceableFailure &&operator<<(T &&value) && {
    return std::move(this->operator<<(std::forward<T>(value)));
  }

  /// Attaches a note to the last diagnostic.
  /// Expects this object to be a silenceable failure.
  Diagnostic &attachNote(std::optional<Location> loc = std::nullopt) {
    assert(isSilenceableFailure() &&
           "can only attach notes to silenceable failures");
    return diagnostics.back().attachNote(loc);
  }

private:
  explicit DiagnosedSilenceableFailure(LogicalResult result) : result(result) {}
  explicit DiagnosedSilenceableFailure(Diagnostic &&diagnostic)
      : result(failure()) {
    diagnostics.emplace_back(std::move(diagnostic));
  }
  explicit DiagnosedSilenceableFailure(SmallVector<Diagnostic> &&diagnostics)
      : diagnostics(std::move(diagnostics)), result(failure()) {}

  /// The diagnostics associated with this object. If non-empty, the object is
  /// considered to be in the silenceable failure state regardless of the
  /// `result` field.
  SmallVector<Diagnostic, 1> diagnostics;

  /// The "definite" logical state, either success or failure.
  /// Ignored if the diagnostics message is present.
  LogicalResult result;
````
- **EN**: This block groups callable interfaces such as `takeDiagnostics`, `empty`, `append`, `make_move_iterator`, indicating how `DiagnosedSilenceableFailure` is queried or updated.
- **CN**: 该代码块聚合了 `takeDiagnostics`, `empty`, `append`, `make_move_iterator` 等可调用接口，展示了如何查询或更新 `DiagnosedSilenceableFailure`。

### Lines 163-212
````cpp
#if LLVM_ENABLE_ABI_BREAKING_CHECKS
  /// Whether the associated diagnostics have been reported.
  /// Diagnostics reporting consumes the diagnostics, so we need a mechanism to
  /// differentiate reported diagnostics from a state where it was never
  /// created.
  bool reported = false;
#endif // LLVM_ENABLE_ABI_BREAKING_CHECKS
};

class DiagnosedDefiniteFailure;

DiagnosedDefiniteFailure emitDefiniteFailure(Location loc,
                                             const Twine &message = {});

/// A compatibility class connecting `InFlightDiagnostic` to
/// `DiagnosedSilenceableFailure` while providing an interface similar to the
/// former. Implicitly convertible to `DiagnosticSilenceableFailure` in definite
/// failure state and to `LogicalResult` failure. Reports the error on
/// conversion or on destruction. Instances of this class can be created by
/// `emitDefiniteFailure()`.
class DiagnosedDefiniteFailure {
  friend DiagnosedDefiniteFailure emitDefiniteFailure(Location loc,
                                                      const Twine &message);

public:
  /// Only move-constructible because it carries an in-flight diagnostic.
  DiagnosedDefiniteFailure(DiagnosedDefiniteFailure &&) = default;

  /// Forward the message to the diagnostic.
  template <typename T>
  DiagnosedDefiniteFailure &operator<<(T &&value) & {
    diag << std::forward<T>(value);
    return *this;
  }
  template <typename T>
  DiagnosedDefiniteFailure &&operator<<(T &&value) && {
    return std::move(this->operator<<(std::forward<T>(value)));
  }

  /// Attaches a note to the error.
  Diagnostic &attachNote(std::optional<Location> loc = std::nullopt) {
    return diag.attachNote(loc);
  }

  /// Implicit conversion to DiagnosedSilenceableFailure in the definite failure
  /// state. Reports the error.
  operator DiagnosedSilenceableFailure() {
    diag.report();
    return DiagnosedSilenceableFailure::definiteFailure();
  }
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 214-264
````cpp
  /// Implicit conversion to LogicalResult in the failure state. Reports the
  /// error.
  operator LogicalResult() {
    diag.report();
    return failure();
  }

private:
  /// Constructs a definite failure at the given location with the given
  /// message.
  explicit DiagnosedDefiniteFailure(Location loc, const Twine &message)
      : diag(emitError(loc, message)) {}

  /// Copy-construction and any assignment is disallowed to prevent repeated
  /// error reporting.
  DiagnosedDefiniteFailure(const DiagnosedDefiniteFailure &) = delete;
  DiagnosedDefiniteFailure &
  operator=(const DiagnosedDefiniteFailure &) = delete;
  DiagnosedDefiniteFailure &operator=(DiagnosedDefiniteFailure &&) = delete;

  /// The error message.
  InFlightDiagnostic diag;
};

/// Emits a definite failure with the given message. The returned object allows
/// for last-minute modification to the error message, such as attaching notes
/// and completing the message. It will be reported when the object is
/// destructed or converted.
inline DiagnosedDefiniteFailure emitDefiniteFailure(Location loc,
                                                    const Twine &message) {
  return DiagnosedDefiniteFailure(loc, message);
}
inline DiagnosedDefiniteFailure emitDefiniteFailure(Operation *op,
                                                    const Twine &message = {}) {
  return emitDefiniteFailure(op->getLoc(), message);
}

/// Emits a silenceable failure with the given message. A silenceable failure
/// must be either suppressed or converted into a definite failure and reported
/// to the user.
inline DiagnosedSilenceableFailure
emitSilenceableFailure(Location loc, const Twine &message = {}) {
  Diagnostic diag(loc, DiagnosticSeverity::Error);
  diag << message;
  return DiagnosedSilenceableFailure::silenceableFailure(std::move(diag));
}
inline DiagnosedSilenceableFailure
emitSilenceableFailure(Operation *op, const Twine &message = {}) {
  return emitSilenceableFailure(op->getLoc(), message);
}
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `LogicalResult`, `report`, `failure`, `DiagnosedDefiniteFailure`, indicating how `DiagnosedSilenceableFailure` is queried or updated.
- **CN**: 该代码块聚合了 `LogicalResult`, `report`, `failure`, `DiagnosedDefiniteFailure` 等可调用接口，展示了如何查询或更新 `DiagnosedSilenceableFailure`。

### Lines 266-266
````cpp
#endif // MLIR_DIALECT_TRANSFORM_UTILS_DIAGNOSEDSILENCEABLEFAILURE_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Iterator-based traversal APIs
  **CN**: 基于迭代器的遍历接口
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口

## Dependencies / 依赖关系

- mlir/IR/Diagnostics.h
- mlir/IR/Operation.h
