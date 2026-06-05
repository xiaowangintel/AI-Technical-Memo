# NestedMatcher.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/Affine/Analysis/NestedMatcher.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares public interfaces for the Affine dialect, focused on analysis helpers over dialect operations and types and `NestedMatcher`.
  - **CN**: 声明 Affine 方言中聚焦 `NestedMatcher` 的公共接口，覆盖针对方言操作与类型的分析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- NestedMacher.h - Nested matcher for Function -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-15
```cpp

#ifndef MLIR_DIALECT_AFFINE_ANALYSIS_NESTEDMATCHER_H
#define MLIR_DIALECT_AFFINE_ANALYSIS_NESTEDMATCHER_H

#include "mlir/IR/BuiltinOps.h"
#include "mlir/IR/Operation.h"
#include "llvm/Support/Allocator.h"

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/IR/BuiltinOps.h`, `mlir/IR/Operation.h`, `llvm/Support/Allocator.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/IR/BuiltinOps.h`, `mlir/IR/Operation.h`, `llvm/Support/Allocator.h`。

### Lines 16-21
```cpp
namespace mlir {
class Operation;

namespace affine {
class NestedPattern;

```
- **EN**: Introduces declarations for `mlir`, `Operation`, `affine`, `NestedPattern`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `Operation`, `affine`, `NestedPattern` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 22-27
```cpp
/// An NestedPattern captures nested patterns in the IR.
/// It is used in conjunction with a scoped NestedPatternContext which is an
/// llvm::BumpPtrAllocator that handles memory allocations efficiently and
/// avoids ownership issues.
///
/// In order to use NestedPatterns, first create a scoped context.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 28-33
```cpp
/// When the context goes out of scope, everything is freed.
/// This design simplifies the API by avoiding references to the context and
/// makes it clear that references to matchers must not escape.
///
/// Example:
///   {
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 34-39
```cpp
///      NestedPatternContext context;
///      auto gemmLike = Doall(Doall(Red(LoadStores())));
///      auto matches = gemmLike.match(f);
///      // do work on matches
///   }  // everything is freed
///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 40-45
```cpp
///
/// Nested abstraction for matching results.
/// Provides access to the nested Operation* captured by a Matcher.
///
/// A NestedMatch contains an Operation* and the children NestedMatch and is
/// thus cheap to copy. NestedMatch is stored in a scoped bumper allocator whose
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 46-53
```cpp
/// lifetime is managed by an RAII NestedPatternContext.
class NestedMatch {
public:
  static NestedMatch build(Operation *operation,
                           ArrayRef<NestedMatch> nestedMatches);
  NestedMatch(const NestedMatch &) = default;
  NestedMatch &operator=(const NestedMatch &) = default;

```
- **EN**: Introduces declarations for `NestedMatch`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NestedMatch` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 54-59
```cpp
  explicit operator bool() { return matchedOperation != nullptr; }

  Operation *getMatchedOperation() const { return matchedOperation; }
  ArrayRef<NestedMatch> getMatchedChildren() { return matchedChildren; }

private:
```
- **EN**: Implements logic around `bool`, `getMatchedOperation`, `getMatchedChildren`.
- **CN**: 围绕 `bool`, `getMatchedOperation`, `getMatchedChildren` 实现具体逻辑。

### Lines 60-65
```cpp
  friend class NestedPattern;
  friend class NestedPatternContext;

  /// Underlying global bump allocator managed by a NestedPatternContext.
  static llvm::BumpPtrAllocator *&allocator();

```
- **EN**: Introduces declarations for `NestedPattern`, `NestedPatternContext`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NestedPattern`, `NestedPatternContext` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 66-72
```cpp
  NestedMatch() = default;

  /// Payload, holds a NestedMatch and all its children along this branch.
  Operation *matchedOperation = nullptr;
  ArrayRef<NestedMatch> matchedChildren;
};

```
- **EN**: Declares APIs or declarative rules around `NestedMatch`.
- **CN**: 声明与 `NestedMatch` 相关的 API 或声明式规则。

### Lines 73-78
```cpp
/// A NestedPattern is a nested operation walker that:
///   1. recursively matches a substructure in the tree;
///   2. uses a filter function to refine matches with extra semantic
///      constraints (passed via a lambda of type FilterFunctionType);
///   3. TODO: optionally applies actions (lambda).
///
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 79-84
```cpp
/// Nested patterns are meant to capture imperfectly nested loops while matching
/// properties over the whole loop nest. For instance, in vectorization we are
/// interested in capturing all the imperfectly nested loops of a certain type
/// and such that all the load and stores have certain access patterns along the
/// loops' induction variables). Such NestedMatches are first captured using the
/// `match` function and are later processed to analyze properties and apply
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 85-90
```cpp
/// transformations in a non-greedy way.
///
/// The NestedMatches captured in the IR can grow large, especially after
/// aggressive unrolling. As experience has shown, it is generally better to use
/// a plain walk over operations to match flat patterns but the current
/// implementation is competitive nonetheless.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 91-99
```cpp
using FilterFunctionType = std::function<bool(Operation &)>;
inline bool defaultFilterFunction(Operation &) { return true; }
class NestedPattern {
public:
  NestedPattern(ArrayRef<NestedPattern> nested,
                FilterFunctionType filter = defaultFilterFunction);
  NestedPattern(const NestedPattern &other);
  NestedPattern &operator=(const NestedPattern &other);

```
- **EN**: Introduces declarations for `NestedPattern`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NestedPattern` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 100-106
```cpp
  ~NestedPattern() {
    // Call destructors manually, ArrayRef is non-owning so it wouldn't call
    // them, but we should free the memory allocated by std::function outside of
    // the arena allocator.
    freeNested();
  }

```
- **EN**: Implements logic around `~NestedPattern`, `freeNested`.
- **CN**: 围绕 `~NestedPattern`, `freeNested` 实现具体逻辑。

### Lines 107-112
```cpp
  /// Returns all the top-level matches in `op`.
  void match(Operation *op, SmallVectorImpl<NestedMatch> *matches) {
    op->walk([&](Operation *child) { matchOne(child, matches); });
  }

  /// Returns the depth of the pattern.
```
- **EN**: Implements logic around `match`, `walk`.
- **CN**: 围绕 `match`, `walk` 实现具体逻辑。

### Lines 113-119
```cpp
  unsigned getDepth() const;

private:
  friend class NestedPatternContext;
  friend class NestedMatch;
  friend struct State;

```
- **EN**: Introduces declarations for `NestedPatternContext`, `NestedMatch`, `State`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NestedPatternContext`, `NestedMatch`, `State` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 120-126
```cpp
  /// Copies the list of nested patterns to the arena allocator associated with
  /// this pattern.
  void copyNestedToThis(ArrayRef<NestedPattern> nested);

  /// Calls destructors on nested patterns.
  void freeNested();

```
- **EN**: Declares APIs or declarative rules around `copyNestedToThis`, `freeNested`.
- **CN**: 声明与 `copyNestedToThis`, `freeNested` 相关的 API 或声明式规则。

### Lines 127-133
```cpp
  /// Underlying global bump allocator managed by a NestedPatternContext.
  static llvm::BumpPtrAllocator *&allocator();

  /// Matches this pattern against a single `op` and fills matches with the
  /// result.
  void matchOne(Operation *op, SmallVectorImpl<NestedMatch> *matches);

```
- **EN**: Declares APIs or declarative rules around `allocator`, `matchOne`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `allocator`, `matchOne` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 134-139
```cpp
  /// Nested patterns to be matched.
  ArrayRef<NestedPattern> nestedPatterns;

  /// Extra filter function to apply to prune patterns as the IR is walked.
  FilterFunctionType filter;

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 140-145
```cpp
  /// skip is an implementation detail needed so that we can implement match
  /// without switching on the type of the Operation. The idea is that a
  /// NestedPattern first checks if it matches locally and then recursively
  /// applies its nested matchers to its elem->nested. Since we want to rely on
  /// the existing operation walking functionality rather than duplicate
  /// it, we allow an off-by-one traversal to account for the fact that we
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 146-151
```cpp
  /// write:
  ///
  ///  void match(Operation *elem) {
  ///    for (auto &c : getNestedPatterns()) {
  ///      NestedPattern childPattern(...);
  ///                                  ^~~~ Needs off-by-one skip.
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 152-157
```cpp
  ///
  Operation *skip;
};

/// RAII structure to transparently manage the bump allocator for
/// NestedPattern and NestedMatch classes. This avoids passing a context to
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 158-169
```cpp
/// all the API functions.
class NestedPatternContext {
public:
  NestedPatternContext() {
    assert(NestedMatch::allocator() == nullptr &&
           "Only a single NestedPatternContext is supported");
    assert(NestedPattern::allocator() == nullptr &&
           "Only a single NestedPatternContext is supported");
    NestedMatch::allocator() = &allocator;
    NestedPattern::allocator() = &allocator;
  }
  ~NestedPatternContext() {
```
- **EN**: Introduces declarations for `NestedPatternContext`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `NestedPatternContext` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 170-175
```cpp
    NestedMatch::allocator() = nullptr;
    NestedPattern::allocator() = nullptr;
  }
  llvm::BumpPtrAllocator allocator;
};

```
- **EN**: Declares APIs or declarative rules around `allocator`.
- **CN**: 声明与 `allocator` 相关的 API 或声明式规则。

### Lines 176-187
```cpp
namespace matcher {
// Syntactic sugar NestedPattern builder functions.
NestedPattern Op(FilterFunctionType filter = defaultFilterFunction);
NestedPattern If(const NestedPattern &child);
NestedPattern If(const FilterFunctionType &filter, const NestedPattern &child);
NestedPattern If(ArrayRef<NestedPattern> nested = {});
NestedPattern If(const FilterFunctionType &filter,
                 ArrayRef<NestedPattern> nested = {});
NestedPattern For(const NestedPattern &child);
NestedPattern For(const FilterFunctionType &filter, const NestedPattern &child);
NestedPattern For(ArrayRef<NestedPattern> nested = {});
NestedPattern For(const FilterFunctionType &filter,
```
- **EN**: Introduces declarations for `matcher`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `matcher` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 188-193
```cpp
                  ArrayRef<NestedPattern> nested = {});

bool isParallelLoop(Operation &op);
bool isReductionLoop(Operation &op);
bool isLoadOrStore(Operation &op);

```
- **EN**: Implements logic around `isParallelLoop`, `isReductionLoop`, `isLoadOrStore`.
- **CN**: 围绕 `isParallelLoop`, `isReductionLoop`, `isLoadOrStore` 实现具体逻辑。

### Lines 194-198
```cpp
} // namespace matcher
} // namespace affine
} // namespace mlir

#endif // MLIR_DIALECT_AFFINE_ANALYSIS_NESTEDMATCHER_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/IR/BuiltinOps.h`, `mlir/IR/Operation.h`, `llvm/Support/Allocator.h`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), LLVM support-library facilities / LLVM Support 库设施 (1)
