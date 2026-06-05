# StmtOpenMP.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/AST/StmtOpenMP.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements the subclasses of Stmt class declared in StmtOpenMP.h.
  - **CN**: 实现语句节点行为、profiling 或序列化辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```cpp
//===--- StmtOpenMP.cpp - Classes for OpenMP directives -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the subclasses of Stmt class declared in StmtOpenMP.h
//
//===----------------------------------------------------------------------===//

#include "clang/AST/ASTContext.h"
#include "clang/AST/StmtOpenMP.h"

using namespace clang;
using namespace llvm::omp;

```
- **EN**: Pulls in the headers needed by this translation unit, including `clang/AST/ASTContext.h`, `clang/AST/StmtOpenMP.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `clang/AST/ASTContext.h`, `clang/AST/StmtOpenMP.h`。

### Lines 19-36
```cpp
size_t OMPChildren::size(unsigned NumClauses, bool HasAssociatedStmt,
                         unsigned NumChildren) {
  return llvm::alignTo(
      totalSizeToAlloc<OMPClause *, Stmt *>(
          NumClauses, NumChildren + (HasAssociatedStmt ? 1 : 0)),
      alignof(OMPChildren));
}

void OMPChildren::setClauses(ArrayRef<OMPClause *> Clauses) {
  assert(Clauses.size() == NumClauses &&
         "Number of clauses is not the same as the preallocated buffer");
  llvm::copy(Clauses, getTrailingObjects<OMPClause *>());
}

MutableArrayRef<Stmt *> OMPChildren::getChildren() {
  return getTrailingObjects<Stmt *>(NumChildren);
}

```
- **EN**: Implements logic around `size`, `alignTo`, `setClauses`, `assert`, and 2 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `size`, `alignTo`, `setClauses`, `assert`, and 2 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 37-57
```cpp
OMPChildren *OMPChildren::Create(void *Mem, ArrayRef<OMPClause *> Clauses) {
  auto *Data = CreateEmpty(Mem, Clauses.size());
  Data->setClauses(Clauses);
  return Data;
}

OMPChildren *OMPChildren::Create(void *Mem, ArrayRef<OMPClause *> Clauses,
                                 Stmt *S, unsigned NumChildren) {
  auto *Data = CreateEmpty(Mem, Clauses.size(), S, NumChildren);
  Data->setClauses(Clauses);
  if (S)
    Data->setAssociatedStmt(S);
  return Data;
}

OMPChildren *OMPChildren::CreateEmpty(void *Mem, unsigned NumClauses,
                                      bool HasAssociatedStmt,
                                      unsigned NumChildren) {
  return new (Mem) OMPChildren(NumClauses, NumChildren, HasAssociatedStmt);
}

```
- **EN**: Implements logic around `Create`, `CreateEmpty`, `setClauses`, `setAssociatedStmt`, and 1 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `CreateEmpty`, `setClauses`, `setAssociatedStmt`, and 1 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 58-76
```cpp
bool OMPExecutableDirective::isStandaloneDirective() const {
  // Special case: 'omp target enter data', 'omp target exit data',
  // 'omp target update' are stand-alone directives, but for implementation
  // reasons they have empty synthetic structured block, to simplify codegen.
  if (isa<OMPTargetEnterDataDirective>(this) ||
      isa<OMPTargetExitDataDirective>(this) ||
      isa<OMPTargetUpdateDirective>(this))
    return true;
  return !hasAssociatedStmt();
}

Stmt *OMPExecutableDirective::getStructuredBlock() {
  assert(!isStandaloneDirective() &&
         "Standalone Executable Directives don't have Structured Blocks.");
  if (auto *LD = dyn_cast<OMPLoopDirective>(this))
    return LD->getBody();
  return getRawStmt();
}

```
- **EN**: Implements logic around `isStandaloneDirective`, `isa`, `hasAssociatedStmt`, `getStructuredBlock`, and 4 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `isStandaloneDirective`, `isa`, `hasAssociatedStmt`, `getStructuredBlock`, and 4 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 77-112
```cpp
Stmt *
OMPLoopBasedDirective::tryToFindNextInnerLoop(Stmt *CurStmt,
                                              bool TryImperfectlyNestedLoops) {
  Stmt *OrigStmt = CurStmt;
  CurStmt = CurStmt->IgnoreContainers();
  // Additional work for imperfectly nested loops, introduced in OpenMP 5.0.
  if (TryImperfectlyNestedLoops) {
    if (auto *CS = dyn_cast<CompoundStmt>(CurStmt)) {
      CurStmt = nullptr;
      SmallVector<CompoundStmt *, 4> Statements(1, CS);
      SmallVector<CompoundStmt *, 4> NextStatements;
      while (!Statements.empty()) {
        CS = Statements.pop_back_val();
        if (!CS)
          continue;
        for (Stmt *S : CS->body()) {
          if (!S)
            continue;
          if (auto *CanonLoop = dyn_cast<OMPCanonicalLoop>(S))
            S = CanonLoop->getLoopStmt();
          if (isa<ForStmt>(S) || isa<CXXForRangeStmt>(S) ||
              (isa<OMPLoopBasedDirective>(S) && !isa<OMPLoopDirective>(S))) {
            // Only single loop construct is allowed.
            if (CurStmt) {
              CurStmt = OrigStmt;
              break;
            }
            CurStmt = S;
            continue;
          }
          S = S->IgnoreContainers();
          if (auto *InnerCS = dyn_cast_or_null<CompoundStmt>(S))
            NextStatements.push_back(InnerCS);
        }
        if (Statements.empty()) {
          // Found single inner loop or multiple loops - exit.
```
- **EN**: Implements logic around `tryToFindNextInnerLoop`, `IgnoreContainers`, `dyn_cast`, `Statements`, and 7 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `tryToFindNextInnerLoop`, `IgnoreContainers`, `dyn_cast`, `Statements`, and 7 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 113-136
```cpp
          if (CurStmt)
            break;
          Statements.swap(NextStatements);
        }
      }
      if (!CurStmt)
        CurStmt = OrigStmt;
    }
  }
  return CurStmt;
}

bool OMPLoopBasedDirective::doForAllLoops(
    Stmt *CurStmt, bool TryImperfectlyNestedLoops, unsigned NumLoops,
    llvm::function_ref<bool(unsigned, Stmt *)> Callback,
    llvm::function_ref<void(OMPLoopTransformationDirective *)>
        OnTransformationCallback) {
  CurStmt = CurStmt->IgnoreContainers();
  for (unsigned Cnt = 0; Cnt < NumLoops; ++Cnt) {
    while (true) {
      auto *Dir = dyn_cast<OMPLoopTransformationDirective>(CurStmt);
      if (!Dir)
        break;

```
- **EN**: Implements logic around `swap`, `doForAllLoops`, `function_ref`, `IgnoreContainers`, and 1 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `swap`, `doForAllLoops`, `function_ref`, `IgnoreContainers`, and 1 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 137-155
```cpp
      OnTransformationCallback(Dir);

      Stmt *TransformedStmt = Dir->getTransformedStmt();
      if (!TransformedStmt) {
        unsigned NumGeneratedTopLevelLoops =
            Dir->getNumGeneratedTopLevelLoops();
        if (NumGeneratedTopLevelLoops == 0) {
          // May happen if the loop transformation does not result in a
          // generated loop (such as full unrolling).
          break;
        }
        if (NumGeneratedTopLevelLoops > 0) {
          // The loop transformation construct has generated loops, but these
          // may not have been generated yet due to being in a dependent
          // context.
          return true;
        }
      }

```
- **EN**: Implements logic around `OnTransformationCallback`, `getTransformedStmt`, `getNumGeneratedTopLevelLoops`; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `OnTransformationCallback`, `getTransformedStmt`, `getNumGeneratedTopLevelLoops` 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 156-179
```cpp
      CurStmt = TransformedStmt;
    }
    if (auto *CanonLoop = dyn_cast<OMPCanonicalLoop>(CurStmt))
      CurStmt = CanonLoop->getLoopStmt();
    if (Callback(Cnt, CurStmt))
      return false;
    // Move on to the next nested for loop, or to the loop body.
    // OpenMP [2.8.1, simd construct, Restrictions]
    // All loops associated with the construct must be perfectly nested; that
    // is, there must be no intervening code nor any OpenMP directive between
    // any two loops.
    if (auto *For = dyn_cast<ForStmt>(CurStmt)) {
      CurStmt = For->getBody();
    } else {
      assert(isa<CXXForRangeStmt>(CurStmt) &&
             "Expected canonical for or range-based for loops.");
      CurStmt = cast<CXXForRangeStmt>(CurStmt)->getBody();
    }
    CurStmt = OMPLoopBasedDirective::tryToFindNextInnerLoop(
        CurStmt, TryImperfectlyNestedLoops);
  }
  return true;
}

```
- **EN**: Implements logic around `dyn_cast`, `getLoopStmt`, `Callback`, `getBody`, and 3 more symbols; this block queries or canonicalizes Clang type-system state; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `dyn_cast`, `getLoopStmt`, `Callback`, `getBody`, and 3 more symbols 实现具体逻辑；该代码块查询或规范化 Clang 类型系统状态，并处理 OpenMP 专用 AST 构造。

### Lines 180-202
```cpp
void OMPLoopBasedDirective::doForAllLoopsBodies(
    Stmt *CurStmt, bool TryImperfectlyNestedLoops, unsigned NumLoops,
    llvm::function_ref<void(unsigned, Stmt *, Stmt *)> Callback) {
  bool Res = OMPLoopBasedDirective::doForAllLoops(
      CurStmt, TryImperfectlyNestedLoops, NumLoops,
      [Callback](unsigned Cnt, Stmt *Loop) {
        Stmt *Body = nullptr;
        if (auto *For = dyn_cast<ForStmt>(Loop)) {
          Body = For->getBody();
        } else {
          assert(isa<CXXForRangeStmt>(Loop) &&
                 "Expected canonical for or range-based for loops.");
          Body = cast<CXXForRangeStmt>(Loop)->getBody();
        }
        if (auto *CanonLoop = dyn_cast<OMPCanonicalLoop>(Body))
          Body = CanonLoop->getLoopStmt();
        Callback(Cnt, Loop, Body);
        return false;
      });
  assert(Res && "Expected only loops");
  (void)Res;
}

```
- **EN**: Implements logic around `doForAllLoopsBodies`, `function_ref`, `doForAllLoops`, `dyn_cast`, and 5 more symbols; this block traverses or manipulates statement/expression trees; queries or canonicalizes Clang type-system state; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `doForAllLoopsBodies`, `function_ref`, `doForAllLoops`, `dyn_cast`, and 5 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并查询或规范化 Clang 类型系统状态，并处理 OpenMP 专用 AST 构造。

### Lines 203-225
```cpp
Stmt *OMPLoopDirective::getBody() {
  // This relies on the loop form is already checked by Sema.
  Stmt *Body = nullptr;
  OMPLoopBasedDirective::doForAllLoopsBodies(
      Data->getRawStmt(), /*TryImperfectlyNestedLoops=*/true,
      NumAssociatedLoops,
      [&Body](unsigned, Stmt *, Stmt *BodyStmt) { Body = BodyStmt; });
  return Body;
}

void OMPLoopDirective::setCounters(ArrayRef<Expr *> A) {
  assert(A.size() == getLoopsNumber() &&
         "Number of loop counters is not the same as the collapsed number");
  llvm::copy(A, getCounters().begin());
}

void OMPLoopDirective::setPrivateCounters(ArrayRef<Expr *> A) {
  assert(A.size() == getLoopsNumber() && "Number of loop private counters "
                                         "is not the same as the collapsed "
                                         "number");
  llvm::copy(A, getPrivateCounters().begin());
}

```
- **EN**: Implements logic around `getBody`, `doForAllLoopsBodies`, `getRawStmt`, `setCounters`, and 3 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `getBody`, `doForAllLoopsBodies`, `getRawStmt`, `setCounters`, and 3 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 226-243
```cpp
void OMPLoopDirective::setInits(ArrayRef<Expr *> A) {
  assert(A.size() == getLoopsNumber() &&
         "Number of counter inits is not the same as the collapsed number");
  llvm::copy(A, getInits().begin());
}

void OMPLoopDirective::setUpdates(ArrayRef<Expr *> A) {
  assert(A.size() == getLoopsNumber() &&
         "Number of counter updates is not the same as the collapsed number");
  llvm::copy(A, getUpdates().begin());
}

void OMPLoopDirective::setFinals(ArrayRef<Expr *> A) {
  assert(A.size() == getLoopsNumber() &&
         "Number of counter finals is not the same as the collapsed number");
  llvm::copy(A, getFinals().begin());
}

```
- **EN**: Implements logic around `setInits`, `assert`, `copy`, `setUpdates`, and 1 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setInits`, `assert`, `copy`, `setUpdates`, and 1 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 244-262
```cpp
void OMPLoopDirective::setDependentCounters(ArrayRef<Expr *> A) {
  assert(
      A.size() == getLoopsNumber() &&
      "Number of dependent counters is not the same as the collapsed number");
  llvm::copy(A, getDependentCounters().begin());
}

void OMPLoopDirective::setDependentInits(ArrayRef<Expr *> A) {
  assert(A.size() == getLoopsNumber() &&
         "Number of dependent inits is not the same as the collapsed number");
  llvm::copy(A, getDependentInits().begin());
}

void OMPLoopDirective::setFinalsConditions(ArrayRef<Expr *> A) {
  assert(A.size() == getLoopsNumber() &&
         "Number of finals conditions is not the same as the collapsed number");
  llvm::copy(A, getFinalsConditions().begin());
}

```
- **EN**: Implements logic around `setDependentCounters`, `assert`, `size`, `copy`, and 2 more symbols; this block traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setDependentCounters`, `assert`, `size`, `copy`, and 2 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 263-281
```cpp
OMPMetaDirective *OMPMetaDirective::Create(const ASTContext &C,
                                           SourceLocation StartLoc,
                                           SourceLocation EndLoc,
                                           ArrayRef<OMPClause *> Clauses,
                                           Stmt *AssociatedStmt, Stmt *IfStmt) {
  auto *Dir = createDirective<OMPMetaDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/1, StartLoc, EndLoc);
  Dir->setIfStmt(IfStmt);
  return Dir;
}

OMPMetaDirective *OMPMetaDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                EmptyShell) {
  return createEmptyDirective<OMPMetaDirective>(C, NumClauses,
                                                /*HasAssociatedStmt=*/true,
                                                /*NumChildren=*/1);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `setIfStmt`, `CreateEmpty`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `setIfStmt`, `CreateEmpty`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 282-300
```cpp
OMPParallelDirective *OMPParallelDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt, Expr *TaskRedRef,
    bool HasCancel) {
  auto *Dir = createDirective<OMPParallelDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/1, StartLoc, EndLoc);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

OMPParallelDirective *OMPParallelDirective::CreateEmpty(const ASTContext &C,
                                                        unsigned NumClauses,
                                                        EmptyShell) {
  return createEmptyDirective<OMPParallelDirective>(C, NumClauses,
                                                    /*HasAssociatedStmt=*/true,
                                                    /*NumChildren=*/1);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `setTaskReductionRefExpr`, `setHasCancel`, and 2 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `setTaskReductionRefExpr`, `setHasCancel`, and 2 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 301-327
```cpp
OMPSimdDirective *
OMPSimdDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                         SourceLocation EndLoc, unsigned CollapsedNum,
                         ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
                         const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPSimdDirective>(
      C, Clauses, AssociatedStmt, numLoopChildren(CollapsedNum, OMPD_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 15 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 15 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 328-363
```cpp
OMPSimdDirective *OMPSimdDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                unsigned CollapsedNum,
                                                EmptyShell) {
  return createEmptyDirective<OMPSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_simd), CollapsedNum);
}

OMPForDirective *OMPForDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, Expr *TaskRedRef, bool HasCancel) {
  auto *Dir = createDirective<OMPForDirective>(
      C, Clauses, AssociatedStmt, numLoopChildren(CollapsedNum, OMPD_for) + 1,
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 21 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 21 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 364-388
```cpp
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

Stmt *OMPLoopTransformationDirective::getTransformedStmt() const {
  if (auto *D = dyn_cast<OMPCanonicalLoopNestTransformationDirective>(S))
    return D->getTransformedStmt();
  if (auto *D = dyn_cast<OMPCanonicalLoopSequenceTransformationDirective>(S))
    return D->getTransformedStmt();
  llvm_unreachable("unexpected object type");
}

Stmt *OMPLoopTransformationDirective::getPreInits() const {
  if (auto *D = dyn_cast<OMPCanonicalLoopNestTransformationDirective>(S))
    return D->getPreInits();
  if (auto *D = dyn_cast<OMPCanonicalLoopSequenceTransformationDirective>(S))
    return D->getPreInits();
  llvm_unreachable("unexpected object type");
}

```
- **EN**: Implements logic around `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, `setPreInits`, and 6 more symbols; this block traverses or manipulates statement/expression trees; queries or canonicalizes Clang type-system state; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, `setPreInits`, and 6 more symbols 实现具体逻辑；该代码块遍历或操作语句/表达式树，并查询或规范化 Clang 类型系统状态，并处理 OpenMP 专用 AST 构造。

### Lines 389-414
```cpp
Stmt *OMPCanonicalLoopNestTransformationDirective::getTransformedStmt() const {
  switch (getStmtClass()) {
#define STMT(CLASS, PARENT)
#define ABSTRACT_STMT(CLASS)
#define OMPCANONICALLOOPNESTTRANSFORMATIONDIRECTIVE(CLASS, PARENT)             \
  case Stmt::CLASS##Class:                                                     \
    return static_cast<const CLASS *>(this)->getTransformedStmt();
#include "clang/AST/StmtNodes.inc"
  default:
    llvm_unreachable("Not a loop transformation for canonical loop nests");
  }
}

Stmt *OMPCanonicalLoopNestTransformationDirective::getPreInits() const {
  switch (getStmtClass()) {
#define STMT(CLASS, PARENT)
#define ABSTRACT_STMT(CLASS)
#define OMPCANONICALLOOPNESTTRANSFORMATIONDIRECTIVE(CLASS, PARENT)             \
  case Stmt::CLASS##Class:                                                     \
    return static_cast<const CLASS *>(this)->getPreInits();
#include "clang/AST/StmtNodes.inc"
  default:
    llvm_unreachable("Not a loop transformation for canonical loop nests");
  }
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `clang/AST/StmtNodes.inc`, `clang/AST/StmtNodes.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `clang/AST/StmtNodes.inc`, `clang/AST/StmtNodes.inc`。

### Lines 415-441
```cpp
Stmt *
OMPCanonicalLoopSequenceTransformationDirective::getTransformedStmt() const {
  switch (getStmtClass()) {
#define STMT(CLASS, PARENT)
#define ABSTRACT_STMT(CLASS)
#define OMPCANONICALLOOPSEQUENCETRANSFORMATIONDIRECTIVE(CLASS, PARENT)         \
  case Stmt::CLASS##Class:                                                     \
    return static_cast<const CLASS *>(this)->getTransformedStmt();
#include "clang/AST/StmtNodes.inc"
  default:
    llvm_unreachable("Not a loop transformation for canonical loop sequences");
  }
}

Stmt *OMPCanonicalLoopSequenceTransformationDirective::getPreInits() const {
  switch (getStmtClass()) {
#define STMT(CLASS, PARENT)
#define ABSTRACT_STMT(CLASS)
#define OMPCANONICALLOOPSEQUENCETRANSFORMATIONDIRECTIVE(CLASS, PARENT)         \
  case Stmt::CLASS##Class:                                                     \
    return static_cast<const CLASS *>(this)->getPreInits();
#include "clang/AST/StmtNodes.inc"
  default:
    llvm_unreachable("Not a loop transformation for canonical loop sequences");
  }
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `clang/AST/StmtNodes.inc`, `clang/AST/StmtNodes.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `clang/AST/StmtNodes.inc`, `clang/AST/StmtNodes.inc`。

### Lines 442-463
```cpp
OMPForDirective *OMPForDirective::CreateEmpty(const ASTContext &C,
                                              unsigned NumClauses,
                                              unsigned CollapsedNum,
                                              EmptyShell) {
  return createEmptyDirective<OMPForDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_for) + 1, CollapsedNum);
}

OMPTileDirective *
OMPTileDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                         SourceLocation EndLoc, ArrayRef<OMPClause *> Clauses,
                         unsigned NumLoops, Stmt *AssociatedStmt,
                         Stmt *TransformedStmt, Stmt *PreInits) {
  OMPTileDirective *Dir = createDirective<OMPTileDirective>(
      C, Clauses, AssociatedStmt, TransformedStmtOffset + 1, StartLoc, EndLoc,
      NumLoops);
  Dir->setTransformedStmt(TransformedStmt);
  Dir->setPreInits(PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 3 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 3 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 464-484
```cpp
OMPTileDirective *OMPTileDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                unsigned NumLoops) {
  return createEmptyDirective<OMPTileDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, TransformedStmtOffset + 1,
      SourceLocation(), SourceLocation(), NumLoops);
}

OMPStripeDirective *
OMPStripeDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                           SourceLocation EndLoc, ArrayRef<OMPClause *> Clauses,
                           unsigned NumLoops, Stmt *AssociatedStmt,
                           Stmt *TransformedStmt, Stmt *PreInits) {
  OMPStripeDirective *Dir = createDirective<OMPStripeDirective>(
      C, Clauses, AssociatedStmt, TransformedStmtOffset + 1, StartLoc, EndLoc,
      NumLoops);
  Dir->setTransformedStmt(TransformedStmt);
  Dir->setPreInits(PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 485-507
```cpp
OMPStripeDirective *OMPStripeDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    unsigned NumLoops) {
  return createEmptyDirective<OMPStripeDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, TransformedStmtOffset + 1,
      SourceLocation(), SourceLocation(), NumLoops);
}

OMPUnrollDirective *OMPUnrollDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    unsigned NumGeneratedTopLevelLoops, Stmt *TransformedStmt, Stmt *PreInits) {
  assert(NumGeneratedTopLevelLoops <= 1 &&
         "Unrolling generates at most one loop");

  auto *Dir = createDirective<OMPUnrollDirective>(
      C, Clauses, AssociatedStmt, TransformedStmtOffset + 1, StartLoc, EndLoc);
  Dir->setNumGeneratedTopLevelLoops(NumGeneratedTopLevelLoops);
  Dir->setTransformedStmt(TransformedStmt);
  Dir->setPreInits(PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 5 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 5 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 508-527
```cpp
OMPUnrollDirective *OMPUnrollDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses) {
  return createEmptyDirective<OMPUnrollDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, TransformedStmtOffset + 1,
      SourceLocation(), SourceLocation());
}

OMPReverseDirective *
OMPReverseDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                            SourceLocation EndLoc, Stmt *AssociatedStmt,
                            unsigned NumLoops, Stmt *TransformedStmt,
                            Stmt *PreInits) {
  OMPReverseDirective *Dir = createDirective<OMPReverseDirective>(
      C, {}, AssociatedStmt, TransformedStmtOffset + 1, StartLoc, EndLoc,
      NumLoops);
  Dir->setTransformedStmt(TransformedStmt);
  Dir->setPreInits(PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 528-546
```cpp
OMPReverseDirective *OMPReverseDirective::CreateEmpty(const ASTContext &C,
                                                      unsigned NumLoops) {
  return createEmptyDirective<OMPReverseDirective>(
      C, /*NumClauses=*/0, /*HasAssociatedStmt=*/true,
      TransformedStmtOffset + 1, SourceLocation(), SourceLocation(), NumLoops);
}

OMPInterchangeDirective *OMPInterchangeDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, unsigned NumLoops, Stmt *AssociatedStmt,
    Stmt *TransformedStmt, Stmt *PreInits) {
  OMPInterchangeDirective *Dir = createDirective<OMPInterchangeDirective>(
      C, Clauses, AssociatedStmt, TransformedStmtOffset + 1, StartLoc, EndLoc,
      NumLoops);
  Dir->setTransformedStmt(TransformedStmt);
  Dir->setPreInits(PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 547-567
```cpp
OMPInterchangeDirective *
OMPInterchangeDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                     unsigned NumLoops) {
  return createEmptyDirective<OMPInterchangeDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, TransformedStmtOffset + 1,
      SourceLocation(), SourceLocation(), NumLoops);
}

OMPSplitDirective *
OMPSplitDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                          SourceLocation EndLoc, ArrayRef<OMPClause *> Clauses,
                          unsigned NumLoops, Stmt *AssociatedStmt,
                          Stmt *TransformedStmt, Stmt *PreInits) {
  OMPSplitDirective *Dir = createDirective<OMPSplitDirective>(
      C, Clauses, AssociatedStmt, TransformedStmtOffset + 1, StartLoc, EndLoc,
      NumLoops);
  Dir->setTransformedStmt(TransformedStmt);
  Dir->setPreInits(PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 3 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 568-588
```cpp
OMPSplitDirective *OMPSplitDirective::CreateEmpty(const ASTContext &C,
                                                  unsigned NumClauses,
                                                  unsigned NumLoops) {
  return createEmptyDirective<OMPSplitDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, TransformedStmtOffset + 1,
      SourceLocation(), SourceLocation(), NumLoops);
}

OMPFuseDirective *OMPFuseDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, unsigned NumGeneratedTopLevelLoops,
    Stmt *AssociatedStmt, Stmt *TransformedStmt, Stmt *PreInits) {

  OMPFuseDirective *Dir = createDirective<OMPFuseDirective>(
      C, Clauses, AssociatedStmt, TransformedStmtOffset + 1, StartLoc, EndLoc);
  Dir->setTransformedStmt(TransformedStmt);
  Dir->setPreInits(PreInits);
  Dir->setNumGeneratedTopLevelLoops(NumGeneratedTopLevelLoops);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 4 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 4 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 589-624
```cpp
OMPFuseDirective *OMPFuseDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses) {
  OMPFuseDirective *Dir = createEmptyDirective<OMPFuseDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, TransformedStmtOffset + 1,
      SourceLocation(), SourceLocation());
  return Dir;
}

OMPForSimdDirective *
OMPForSimdDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                            SourceLocation EndLoc, unsigned CollapsedNum,
                            ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
                            const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPForSimdDirective>(
      C, Clauses, AssociatedStmt, numLoopChildren(CollapsedNum, OMPD_for_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 22 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `SourceLocation`, `Create`, and 22 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 625-652
```cpp
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPForSimdDirective *OMPForSimdDirective::CreateEmpty(const ASTContext &C,
                                                      unsigned NumClauses,
                                                      unsigned CollapsedNum,
                                                      EmptyShell) {
  return createEmptyDirective<OMPForSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_for_simd), CollapsedNum);
}

OMPSectionsDirective *OMPSectionsDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt, Expr *TaskRedRef,
    bool HasCancel) {
  auto *Dir = createDirective<OMPSectionsDirective>(C, Clauses, AssociatedStmt,
                                                    /*NumChildren=*/1, StartLoc,
                                                    EndLoc);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

```
- **EN**: Implements logic around `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, `setPreInits`, and 7 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, `setPreInits`, and 7 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 653-672
```cpp
OMPSectionsDirective *OMPSectionsDirective::CreateEmpty(const ASTContext &C,
                                                        unsigned NumClauses,
                                                        EmptyShell) {
  return createEmptyDirective<OMPSectionsDirective>(C, NumClauses,
                                                    /*HasAssociatedStmt=*/true,
                                                    /*NumChildren=*/1);
}

OMPSectionDirective *OMPSectionDirective::Create(const ASTContext &C,
                                                 SourceLocation StartLoc,
                                                 SourceLocation EndLoc,
                                                 Stmt *AssociatedStmt,
                                                 bool HasCancel) {
  auto *Dir =
      createDirective<OMPSectionDirective>(C, {}, AssociatedStmt,
                                           /*NumChildren=*/0, StartLoc, EndLoc);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 673-695
```cpp
OMPSectionDirective *OMPSectionDirective::CreateEmpty(const ASTContext &C,
                                                      EmptyShell) {
  return createEmptyDirective<OMPSectionDirective>(C, /*NumClauses=*/0,
                                                   /*HasAssociatedStmt=*/true);
}

OMPScopeDirective *OMPScopeDirective::Create(const ASTContext &C,
                                             SourceLocation StartLoc,
                                             SourceLocation EndLoc,
                                             ArrayRef<OMPClause *> Clauses,
                                             Stmt *AssociatedStmt) {
  return createDirective<OMPScopeDirective>(C, Clauses, AssociatedStmt,
                                            /*NumChildren=*/0, StartLoc,
                                            EndLoc);
}

OMPScopeDirective *OMPScopeDirective::CreateEmpty(const ASTContext &C,
                                                  unsigned NumClauses,
                                                  EmptyShell) {
  return createEmptyDirective<OMPScopeDirective>(C, NumClauses,
                                                 /*HasAssociatedStmt=*/true);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 696-721
```cpp
OMPSingleDirective *OMPSingleDirective::Create(const ASTContext &C,
                                               SourceLocation StartLoc,
                                               SourceLocation EndLoc,
                                               ArrayRef<OMPClause *> Clauses,
                                               Stmt *AssociatedStmt) {
  return createDirective<OMPSingleDirective>(C, Clauses, AssociatedStmt,
                                             /*NumChildren=*/0, StartLoc,
                                             EndLoc);
}

OMPSingleDirective *OMPSingleDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    EmptyShell) {
  return createEmptyDirective<OMPSingleDirective>(C, NumClauses,
                                                  /*HasAssociatedStmt=*/true);
}

OMPMasterDirective *OMPMasterDirective::Create(const ASTContext &C,
                                               SourceLocation StartLoc,
                                               SourceLocation EndLoc,
                                               Stmt *AssociatedStmt) {
  return createDirective<OMPMasterDirective>(C, {}, AssociatedStmt,
                                             /*NumChildren=*/0, StartLoc,
                                             EndLoc);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 722-743
```cpp
OMPMasterDirective *OMPMasterDirective::CreateEmpty(const ASTContext &C,
                                                    EmptyShell) {
  return createEmptyDirective<OMPMasterDirective>(C, /*NumClauses=*/0,
                                                  /*HasAssociatedStmt=*/true);
}

OMPCriticalDirective *OMPCriticalDirective::Create(
    const ASTContext &C, const DeclarationNameInfo &Name,
    SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt) {
  return createDirective<OMPCriticalDirective>(C, Clauses, AssociatedStmt,
                                               /*NumChildren=*/0, Name,
                                               StartLoc, EndLoc);
}

OMPCriticalDirective *OMPCriticalDirective::CreateEmpty(const ASTContext &C,
                                                        unsigned NumClauses,
                                                        EmptyShell) {
  return createEmptyDirective<OMPCriticalDirective>(C, NumClauses,
                                                    /*HasAssociatedStmt=*/true);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 744-779
```cpp
OMPParallelForDirective *OMPParallelForDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, Expr *TaskRedRef, bool HasCancel) {
  auto *Dir = createDirective<OMPParallelForDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_parallel_for) + 1, StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->setHasCancel(HasCancel);
  return Dir;
}
```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 25 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 25 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 780-815
```cpp

OMPParallelForDirective *
OMPParallelForDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                     unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPParallelForDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_parallel_for) + 1, CollapsedNum);
}

OMPParallelForSimdDirective *OMPParallelForSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPParallelForSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_parallel_for_simd), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 816-841
```cpp
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPParallelForSimdDirective *
OMPParallelForSimdDirective::CreateEmpty(const ASTContext &C,
                                         unsigned NumClauses,
                                         unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPParallelForSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_parallel_for_simd), CollapsedNum);
}

OMPParallelMasterDirective *OMPParallelMasterDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt, Expr *TaskRedRef) {
  auto *Dir = createDirective<OMPParallelMasterDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/1, StartLoc, EndLoc);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  return Dir;
}

```
- **EN**: Implements logic around `setFinals`, `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, and 7 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setFinals`, `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, and 7 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 842-864
```cpp
OMPParallelMasterDirective *
OMPParallelMasterDirective::CreateEmpty(const ASTContext &C,
                                        unsigned NumClauses, EmptyShell) {
  return createEmptyDirective<OMPParallelMasterDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, /*NumChildren=*/1);
}

OMPParallelMaskedDirective *OMPParallelMaskedDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt, Expr *TaskRedRef) {
  auto *Dir = createDirective<OMPParallelMaskedDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/1, StartLoc, EndLoc);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  return Dir;
}

OMPParallelMaskedDirective *
OMPParallelMaskedDirective::CreateEmpty(const ASTContext &C,
                                        unsigned NumClauses, EmptyShell) {
  return createEmptyDirective<OMPParallelMaskedDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, /*NumChildren=*/1);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 865-882
```cpp
OMPParallelSectionsDirective *OMPParallelSectionsDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt, Expr *TaskRedRef,
    bool HasCancel) {
  auto *Dir = createDirective<OMPParallelSectionsDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/1, StartLoc, EndLoc);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

OMPParallelSectionsDirective *
OMPParallelSectionsDirective::CreateEmpty(const ASTContext &C,
                                          unsigned NumClauses, EmptyShell) {
  return createEmptyDirective<OMPParallelSectionsDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, /*NumChildren=*/1);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `setTaskReductionRefExpr`, `setHasCancel`, and 2 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `setTaskReductionRefExpr`, `setHasCancel`, and 2 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 883-905
```cpp
OMPTaskDirective *
OMPTaskDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                         SourceLocation EndLoc, ArrayRef<OMPClause *> Clauses,
                         Stmt *AssociatedStmt, bool HasCancel) {
  auto *Dir = createDirective<OMPTaskDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/0, StartLoc, EndLoc);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

OMPTaskDirective *OMPTaskDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                EmptyShell) {
  return createEmptyDirective<OMPTaskDirective>(C, NumClauses,
                                                /*HasAssociatedStmt=*/true);
}

OMPTaskyieldDirective *OMPTaskyieldDirective::Create(const ASTContext &C,
                                                     SourceLocation StartLoc,
                                                     SourceLocation EndLoc) {
  return new (C) OMPTaskyieldDirective(StartLoc, EndLoc);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `setHasCancel`, `CreateEmpty`, and 2 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `setHasCancel`, `CreateEmpty`, and 2 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 906-927
```cpp
OMPTaskyieldDirective *OMPTaskyieldDirective::CreateEmpty(const ASTContext &C,
                                                          EmptyShell) {
  return new (C) OMPTaskyieldDirective();
}

OMPAssumeDirective *OMPAssumeDirective::Create(const ASTContext &C,
                                               SourceLocation StartLoc,
                                               SourceLocation EndLoc,
                                               ArrayRef<OMPClause *> Clauses,
                                               Stmt *AStmt) {
  return createDirective<OMPAssumeDirective>(C, Clauses, AStmt,
                                             /*NumChildren=*/0, StartLoc,
                                             EndLoc);
}

OMPAssumeDirective *OMPAssumeDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    EmptyShell) {
  return createEmptyDirective<OMPAssumeDirective>(C, NumClauses,
                                                  /*HasAssociatedStmt=*/true);
}

```
- **EN**: Implements logic around `CreateEmpty`, `new`, `Create`, `createDirective`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `new`, `Create`, `createDirective`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 928-948
```cpp
OMPErrorDirective *OMPErrorDirective::Create(const ASTContext &C,
                                             SourceLocation StartLoc,
                                             SourceLocation EndLoc,
                                             ArrayRef<OMPClause *> Clauses) {
  return createDirective<OMPErrorDirective>(
      C, Clauses, /*AssociatedStmt=*/nullptr, /*NumChildren=*/0, StartLoc,
      EndLoc);
}

OMPErrorDirective *OMPErrorDirective::CreateEmpty(const ASTContext &C,
                                                  unsigned NumClauses,
                                                  EmptyShell) {
  return createEmptyDirective<OMPErrorDirective>(C, NumClauses);
}

OMPBarrierDirective *OMPBarrierDirective::Create(const ASTContext &C,
                                                 SourceLocation StartLoc,
                                                 SourceLocation EndLoc) {
  return new (C) OMPBarrierDirective(StartLoc, EndLoc);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并处理 OpenMP 专用 AST 构造。

### Lines 949-968
```cpp
OMPBarrierDirective *OMPBarrierDirective::CreateEmpty(const ASTContext &C,
                                                      EmptyShell) {
  return new (C) OMPBarrierDirective();
}

OMPTaskwaitDirective *
OMPTaskwaitDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                             SourceLocation EndLoc,
                             ArrayRef<OMPClause *> Clauses) {
  return createDirective<OMPTaskwaitDirective>(
      C, Clauses, /*AssociatedStmt=*/nullptr, /*NumChildren=*/0, StartLoc,
      EndLoc);
}

OMPTaskwaitDirective *OMPTaskwaitDirective::CreateEmpty(const ASTContext &C,
                                                        unsigned NumClauses,
                                                        EmptyShell) {
  return createEmptyDirective<OMPTaskwaitDirective>(C, NumClauses);
}

```
- **EN**: Implements logic around `CreateEmpty`, `new`, `Create`, `createDirective`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `new`, `Create`, `createDirective`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并处理 OpenMP 专用 AST 构造。

### Lines 969-992
```cpp
OMPTaskgroupDirective *OMPTaskgroupDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt, Expr *ReductionRef) {
  auto *Dir = createDirective<OMPTaskgroupDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/1, StartLoc, EndLoc);
  Dir->setReductionRef(ReductionRef);
  return Dir;
}

OMPTaskgroupDirective *OMPTaskgroupDirective::CreateEmpty(const ASTContext &C,
                                                          unsigned NumClauses,
                                                          EmptyShell) {
  return createEmptyDirective<OMPTaskgroupDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, /*NumChildren=*/1);
}

OMPCancellationPointDirective *OMPCancellationPointDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    OpenMPDirectiveKind CancelRegion) {
  auto *Dir = new (C) OMPCancellationPointDirective(StartLoc, EndLoc);
  Dir->setCancelRegion(CancelRegion);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `setReductionRef`, `CreateEmpty`, and 3 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `setReductionRef`, `CreateEmpty`, and 3 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 993-1014
```cpp
OMPCancellationPointDirective *
OMPCancellationPointDirective::CreateEmpty(const ASTContext &C, EmptyShell) {
  return new (C) OMPCancellationPointDirective();
}

OMPCancelDirective *
OMPCancelDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                           SourceLocation EndLoc, ArrayRef<OMPClause *> Clauses,
                           OpenMPDirectiveKind CancelRegion) {
  auto *Dir = createDirective<OMPCancelDirective>(
      C, Clauses, /*AssociatedStmt=*/nullptr, /*NumChildren=*/0, StartLoc,
      EndLoc);
  Dir->setCancelRegion(CancelRegion);
  return Dir;
}

OMPCancelDirective *OMPCancelDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    EmptyShell) {
  return createEmptyDirective<OMPCancelDirective>(C, NumClauses);
}

```
- **EN**: Implements logic around `CreateEmpty`, `new`, `Create`, `createDirective`, and 2 more symbols; this block tracks source-location information and source-to-AST mapping; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `new`, `Create`, `createDirective`, and 2 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并处理 OpenMP 专用 AST 构造。

### Lines 1015-1038
```cpp
OMPFlushDirective *OMPFlushDirective::Create(const ASTContext &C,
                                             SourceLocation StartLoc,
                                             SourceLocation EndLoc,
                                             ArrayRef<OMPClause *> Clauses) {
  return createDirective<OMPFlushDirective>(
      C, Clauses, /*AssociatedStmt=*/nullptr, /*NumChildren=*/0, StartLoc,
      EndLoc);
}

OMPFlushDirective *OMPFlushDirective::CreateEmpty(const ASTContext &C,
                                                  unsigned NumClauses,
                                                  EmptyShell) {
  return createEmptyDirective<OMPFlushDirective>(C, NumClauses);
}

OMPDepobjDirective *OMPDepobjDirective::Create(const ASTContext &C,
                                               SourceLocation StartLoc,
                                               SourceLocation EndLoc,
                                               ArrayRef<OMPClause *> Clauses) {
  return createDirective<OMPDepobjDirective>(
      C, Clauses, /*AssociatedStmt=*/nullptr,
      /*NumChildren=*/0, StartLoc, EndLoc);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`; this block tracks source-location information and source-to-AST mapping; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并处理 OpenMP 专用 AST 构造。

### Lines 1039-1059
```cpp
OMPDepobjDirective *OMPDepobjDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    EmptyShell) {
  return createEmptyDirective<OMPDepobjDirective>(C, NumClauses);
}

OMPScanDirective *OMPScanDirective::Create(const ASTContext &C,
                                           SourceLocation StartLoc,
                                           SourceLocation EndLoc,
                                           ArrayRef<OMPClause *> Clauses) {
  return createDirective<OMPScanDirective>(C, Clauses,
                                           /*AssociatedStmt=*/nullptr,
                                           /*NumChildren=*/0, StartLoc, EndLoc);
}

OMPScanDirective *OMPScanDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                EmptyShell) {
  return createEmptyDirective<OMPScanDirective>(C, NumClauses);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`; this block tracks source-location information and source-to-AST mapping; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并处理 OpenMP 专用 AST 构造。

### Lines 1060-1077
```cpp
OMPOrderedDirective *OMPOrderedDirective::Create(const ASTContext &C,
                                                 SourceLocation StartLoc,
                                                 SourceLocation EndLoc,
                                                 ArrayRef<OMPClause *> Clauses,
                                                 Stmt *AssociatedStmt) {
  return createDirective<OMPOrderedDirective>(
      C, Clauses, cast_or_null<CapturedStmt>(AssociatedStmt),
      /*NumChildren=*/0, StartLoc, EndLoc);
}

OMPOrderedDirective *OMPOrderedDirective::CreateEmpty(const ASTContext &C,
                                                      unsigned NumClauses,
                                                      bool IsStandalone,
                                                      EmptyShell) {
  return createEmptyDirective<OMPOrderedDirective>(C, NumClauses,
                                                   !IsStandalone);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `cast_or_null`, `CreateEmpty`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `cast_or_null`, `CreateEmpty`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1078-1096
```cpp
OMPAtomicDirective *
OMPAtomicDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                           SourceLocation EndLoc, ArrayRef<OMPClause *> Clauses,
                           Stmt *AssociatedStmt, Expressions Exprs) {
  auto *Dir = createDirective<OMPAtomicDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/7, StartLoc, EndLoc);
  Dir->setX(Exprs.X);
  Dir->setV(Exprs.V);
  Dir->setR(Exprs.R);
  Dir->setExpr(Exprs.E);
  Dir->setUpdateExpr(Exprs.UE);
  Dir->setD(Exprs.D);
  Dir->setCond(Exprs.Cond);
  Dir->Flags.IsXLHSInRHSPart = Exprs.IsXLHSInRHSPart ? 1 : 0;
  Dir->Flags.IsPostfixUpdate = Exprs.IsPostfixUpdate ? 1 : 0;
  Dir->Flags.IsFailOnly = Exprs.IsFailOnly ? 1 : 0;
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `setX`, `setV`, and 5 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `setX`, `setV`, and 5 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1097-1119
```cpp
OMPAtomicDirective *OMPAtomicDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    EmptyShell) {
  return createEmptyDirective<OMPAtomicDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, /*NumChildren=*/7);
}

OMPTargetDirective *OMPTargetDirective::Create(const ASTContext &C,
                                               SourceLocation StartLoc,
                                               SourceLocation EndLoc,
                                               ArrayRef<OMPClause *> Clauses,
                                               Stmt *AssociatedStmt) {
  return createDirective<OMPTargetDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/0, StartLoc, EndLoc);
}

OMPTargetDirective *OMPTargetDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    EmptyShell) {
  return createEmptyDirective<OMPTargetDirective>(C, NumClauses,
                                                  /*HasAssociatedStmt=*/true);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1120-1137
```cpp
OMPTargetParallelDirective *OMPTargetParallelDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt, Expr *TaskRedRef,
    bool HasCancel) {
  auto *Dir = createDirective<OMPTargetParallelDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/1, StartLoc, EndLoc);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

OMPTargetParallelDirective *
OMPTargetParallelDirective::CreateEmpty(const ASTContext &C,
                                        unsigned NumClauses, EmptyShell) {
  return createEmptyDirective<OMPTargetParallelDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true, /*NumChildren=*/1);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `setTaskReductionRefExpr`, `setHasCancel`, and 2 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `setTaskReductionRefExpr`, `setHasCancel`, and 2 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1138-1173
```cpp
OMPTargetParallelForDirective *OMPTargetParallelForDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, Expr *TaskRedRef, bool HasCancel) {
  auto *Dir = createDirective<OMPTargetParallelForDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_parallel_for) + 1, StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->setHasCancel(HasCancel);
  return Dir;
}
```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 25 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 25 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1174-1191
```cpp

OMPTargetParallelForDirective *
OMPTargetParallelForDirective::CreateEmpty(const ASTContext &C,
                                           unsigned NumClauses,
                                           unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPTargetParallelForDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_parallel_for) + 1,
      CollapsedNum);
}

OMPTargetDataDirective *OMPTargetDataDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt) {
  return createDirective<OMPTargetDataDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/0, StartLoc, EndLoc);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1192-1212
```cpp
OMPTargetDataDirective *OMPTargetDataDirective::CreateEmpty(const ASTContext &C,
                                                            unsigned N,
                                                            EmptyShell) {
  return createEmptyDirective<OMPTargetDataDirective>(
      C, N, /*HasAssociatedStmt=*/true);
}

OMPTargetEnterDataDirective *OMPTargetEnterDataDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt) {
  return createDirective<OMPTargetEnterDataDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/0, StartLoc, EndLoc);
}

OMPTargetEnterDataDirective *
OMPTargetEnterDataDirective::CreateEmpty(const ASTContext &C, unsigned N,
                                         EmptyShell) {
  return createEmptyDirective<OMPTargetEnterDataDirective>(
      C, N, /*HasAssociatedStmt=*/true);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1213-1235
```cpp
OMPTargetExitDataDirective *OMPTargetExitDataDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt) {
  return createDirective<OMPTargetExitDataDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/0, StartLoc, EndLoc);
}

OMPTargetExitDataDirective *
OMPTargetExitDataDirective::CreateEmpty(const ASTContext &C, unsigned N,
                                        EmptyShell) {
  return createEmptyDirective<OMPTargetExitDataDirective>(
      C, N, /*HasAssociatedStmt=*/true);
}

OMPTeamsDirective *OMPTeamsDirective::Create(const ASTContext &C,
                                             SourceLocation StartLoc,
                                             SourceLocation EndLoc,
                                             ArrayRef<OMPClause *> Clauses,
                                             Stmt *AssociatedStmt) {
  return createDirective<OMPTeamsDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/0, StartLoc, EndLoc);
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1236-1271
```cpp
OMPTeamsDirective *OMPTeamsDirective::CreateEmpty(const ASTContext &C,
                                                  unsigned NumClauses,
                                                  EmptyShell) {
  return createEmptyDirective<OMPTeamsDirective>(C, NumClauses,
                                                 /*HasAssociatedStmt=*/true);
}

OMPTaskLoopDirective *OMPTaskLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, bool HasCancel) {
  auto *Dir = createDirective<OMPTaskLoopDirective>(
      C, Clauses, AssociatedStmt, numLoopChildren(CollapsedNum, OMPD_taskloop),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 23 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 23 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1272-1307
```cpp
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

OMPTaskLoopDirective *OMPTaskLoopDirective::CreateEmpty(const ASTContext &C,
                                                        unsigned NumClauses,
                                                        unsigned CollapsedNum,
                                                        EmptyShell) {
  return createEmptyDirective<OMPTaskLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_taskloop), CollapsedNum);
}

OMPTaskLoopSimdDirective *OMPTaskLoopSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTaskLoopSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_taskloop_simd), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
```
- **EN**: Implements logic around `setFinalsConditions`, `setPreInits`, `setHasCancel`, `CreateEmpty`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setFinalsConditions`, `setPreInits`, `setHasCancel`, `CreateEmpty`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1308-1329
```cpp
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPTaskLoopSimdDirective *
OMPTaskLoopSimdDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                      unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPTaskLoopSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_taskloop_simd), CollapsedNum);
}

```
- **EN**: Implements logic around `setNextUpperBound`, `setNumIterations`, `setCounters`, `setPrivateCounters`, and 10 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setNextUpperBound`, `setNumIterations`, `setCounters`, `setPrivateCounters`, and 10 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1330-1365
```cpp
OMPMasterTaskLoopDirective *OMPMasterTaskLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, bool HasCancel) {
  auto *Dir = createDirective<OMPMasterTaskLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_master_taskloop), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 24 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 24 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1366-1401
```cpp
OMPMasterTaskLoopDirective *
OMPMasterTaskLoopDirective::CreateEmpty(const ASTContext &C,
                                        unsigned NumClauses,
                                        unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPMasterTaskLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_master_taskloop), CollapsedNum);
}

OMPMaskedTaskLoopDirective *OMPMaskedTaskLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, bool HasCancel) {
  auto *Dir = createDirective<OMPMaskedTaskLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_masked_taskloop), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1402-1419
```cpp
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

OMPMaskedTaskLoopDirective *
OMPMaskedTaskLoopDirective::CreateEmpty(const ASTContext &C,
                                        unsigned NumClauses,
                                        unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPMaskedTaskLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_masked_taskloop), CollapsedNum);
}

```
- **EN**: Implements logic around `setFinals`, `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, and 5 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setFinals`, `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, and 5 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1420-1454
```cpp
OMPMasterTaskLoopSimdDirective *OMPMasterTaskLoopSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPMasterTaskLoopSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_master_taskloop_simd), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1455-1490
```cpp
OMPMasterTaskLoopSimdDirective *
OMPMasterTaskLoopSimdDirective::CreateEmpty(const ASTContext &C,
                                            unsigned NumClauses,
                                            unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPMasterTaskLoopSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_master_taskloop_simd), CollapsedNum);
}

OMPMaskedTaskLoopSimdDirective *OMPMaskedTaskLoopSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPMaskedTaskLoopSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_masked_taskloop_simd), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1491-1526
```cpp
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPMaskedTaskLoopSimdDirective *
OMPMaskedTaskLoopSimdDirective::CreateEmpty(const ASTContext &C,
                                            unsigned NumClauses,
                                            unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPMaskedTaskLoopSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_masked_taskloop_simd), CollapsedNum);
}

OMPParallelMasterTaskLoopDirective *OMPParallelMasterTaskLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, bool HasCancel) {
  auto *Dir = createDirective<OMPParallelMasterTaskLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_parallel_master_taskloop), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
```
- **EN**: Implements logic around `setFinals`, `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setFinals`, `setDependentCounters`, `setDependentInits`, `setFinalsConditions`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1527-1554
```cpp
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

OMPParallelMasterTaskLoopDirective *
OMPParallelMasterTaskLoopDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                unsigned CollapsedNum,
                                                EmptyShell) {
  return createEmptyDirective<OMPParallelMasterTaskLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_parallel_master_taskloop),
      CollapsedNum);
}

```
- **EN**: Implements logic around `setEnsureUpperBound`, `setNextLowerBound`, `setNextUpperBound`, `setNumIterations`, and 13 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setEnsureUpperBound`, `setNextLowerBound`, `setNextUpperBound`, `setNumIterations`, and 13 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1555-1590
```cpp
OMPParallelMaskedTaskLoopDirective *OMPParallelMaskedTaskLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, bool HasCancel) {
  auto *Dir = createDirective<OMPParallelMaskedTaskLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_parallel_masked_taskloop), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setHasCancel(HasCancel);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 24 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 24 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1591-1626
```cpp
OMPParallelMaskedTaskLoopDirective *
OMPParallelMaskedTaskLoopDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                unsigned CollapsedNum,
                                                EmptyShell) {
  return createEmptyDirective<OMPParallelMaskedTaskLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_parallel_masked_taskloop),
      CollapsedNum);
}

OMPParallelMasterTaskLoopSimdDirective *
OMPParallelMasterTaskLoopSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPParallelMasterTaskLoopSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_parallel_master_taskloop_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1627-1648
```cpp
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPParallelMasterTaskLoopSimdDirective *
OMPParallelMasterTaskLoopSimdDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    unsigned CollapsedNum,
                                                    EmptyShell) {
  return createEmptyDirective<OMPParallelMasterTaskLoopSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_parallel_master_taskloop_simd),
      CollapsedNum);
}

```
- **EN**: Implements logic around `setPrivateCounters`, `setInits`, `setUpdates`, `setFinals`, and 7 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setPrivateCounters`, `setInits`, `setUpdates`, `setFinals`, and 7 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1649-1684
```cpp
OMPParallelMaskedTaskLoopSimdDirective *
OMPParallelMaskedTaskLoopSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPParallelMaskedTaskLoopSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_parallel_masked_taskloop_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1685-1720
```cpp
OMPParallelMaskedTaskLoopSimdDirective *
OMPParallelMaskedTaskLoopSimdDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    unsigned CollapsedNum,
                                                    EmptyShell) {
  return createEmptyDirective<OMPParallelMaskedTaskLoopSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_parallel_masked_taskloop_simd),
      CollapsedNum);
}

OMPDistributeDirective *
OMPDistributeDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                               SourceLocation EndLoc, unsigned CollapsedNum,
                               ArrayRef<OMPClause *> Clauses,
                               Stmt *AssociatedStmt, const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPDistributeDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_distribute), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1721-1739
```cpp
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPDistributeDirective *
OMPDistributeDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                    unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPDistributeDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_distribute), CollapsedNum);
}

```
- **EN**: Implements logic around `setPrivateCounters`, `setInits`, `setUpdates`, `setFinals`, and 7 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setPrivateCounters`, `setInits`, `setUpdates`, `setFinals`, and 7 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1740-1775
```cpp
OMPTargetUpdateDirective *OMPTargetUpdateDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt) {
  return createDirective<OMPTargetUpdateDirective>(C, Clauses, AssociatedStmt,
                                                   /*NumChildren=*/0, StartLoc,
                                                   EndLoc);
}

OMPTargetUpdateDirective *
OMPTargetUpdateDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                      EmptyShell) {
  return createEmptyDirective<OMPTargetUpdateDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true);
}

OMPDistributeParallelForDirective *OMPDistributeParallelForDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, Expr *TaskRedRef, bool HasCancel) {
  auto *Dir = createDirective<OMPDistributeParallelForDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_distribute_parallel_for) + 1, StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
```
- **EN**: Implements logic around `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`, and 14 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`, and 14 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1776-1804
```cpp
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->HasCancel = HasCancel;
  return Dir;
}

```
- **EN**: Implements logic around `setNextUpperBound`, `setNumIterations`, `setPrevLowerBoundVariable`, `setPrevUpperBoundVariable`, and 21 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setNextUpperBound`, `setNumIterations`, `setPrevLowerBoundVariable`, `setPrevUpperBoundVariable`, and 21 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1805-1840
```cpp
OMPDistributeParallelForDirective *
OMPDistributeParallelForDirective::CreateEmpty(const ASTContext &C,
                                               unsigned NumClauses,
                                               unsigned CollapsedNum,
                                               EmptyShell) {
  return createEmptyDirective<OMPDistributeParallelForDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_distribute_parallel_for) + 1,
      CollapsedNum);
}

OMPDistributeParallelForSimdDirective *
OMPDistributeParallelForSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPDistributeParallelForSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_distribute_parallel_for_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1841-1864
```cpp
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  return Dir;
}

```
- **EN**: Implements logic around `setPrevUpperBoundVariable`, `setDistInc`, `setPrevEnsureUpperBound`, `setCounters`, and 17 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setPrevUpperBoundVariable`, `setDistInc`, `setPrevEnsureUpperBound`, `setCounters`, and 17 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1865-1900
```cpp
OMPDistributeParallelForSimdDirective *
OMPDistributeParallelForSimdDirective::CreateEmpty(const ASTContext &C,
                                                   unsigned NumClauses,
                                                   unsigned CollapsedNum,
                                                   EmptyShell) {
  return createEmptyDirective<OMPDistributeParallelForSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_distribute_parallel_for_simd),
      CollapsedNum);
}

OMPDistributeSimdDirective *OMPDistributeSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPDistributeSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_distribute_simd), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 18 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 18 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1901-1919
```cpp
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPDistributeSimdDirective *
OMPDistributeSimdDirective::CreateEmpty(const ASTContext &C,
                                        unsigned NumClauses,
                                        unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPDistributeSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_distribute_simd), CollapsedNum);
}

```
- **EN**: Implements logic around `setInits`, `setUpdates`, `setFinals`, `setDependentCounters`, and 6 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setInits`, `setUpdates`, `setFinals`, `setDependentCounters`, and 6 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 1920-1954
```cpp
OMPTargetParallelForSimdDirective *OMPTargetParallelForSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTargetParallelForSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_parallel_for_simd), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1955-1990
```cpp
OMPTargetParallelForSimdDirective *
OMPTargetParallelForSimdDirective::CreateEmpty(const ASTContext &C,
                                               unsigned NumClauses,
                                               unsigned CollapsedNum,
                                               EmptyShell) {
  return createEmptyDirective<OMPTargetParallelForSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_parallel_for_simd),
      CollapsedNum);
}

OMPTargetSimdDirective *
OMPTargetSimdDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                               SourceLocation EndLoc, unsigned CollapsedNum,
                               ArrayRef<OMPClause *> Clauses,
                               Stmt *AssociatedStmt, const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTargetSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_simd), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 1991-2026
```cpp
  return Dir;
}

OMPTargetSimdDirective *
OMPTargetSimdDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                    unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPTargetSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_simd), CollapsedNum);
}

OMPTeamsDistributeDirective *OMPTeamsDistributeDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTeamsDistributeDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 18 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 18 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2027-2045
```cpp
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPTeamsDistributeDirective *
OMPTeamsDistributeDirective::CreateEmpty(const ASTContext &C,
                                         unsigned NumClauses,
                                         unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPTeamsDistributeDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute), CollapsedNum);
}

```
- **EN**: Implements logic around `setInits`, `setUpdates`, `setFinals`, `setDependentCounters`, and 6 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setInits`, `setUpdates`, `setFinals`, `setDependentCounters`, and 6 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2046-2080
```cpp
OMPTeamsDistributeSimdDirective *OMPTeamsDistributeSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTeamsDistributeSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute_simd), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `numLoopChildren`, `setIterationVariable`, and 23 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2081-2116
```cpp
OMPTeamsDistributeSimdDirective *OMPTeamsDistributeSimdDirective::CreateEmpty(
    const ASTContext &C, unsigned NumClauses, unsigned CollapsedNum,
    EmptyShell) {
  return createEmptyDirective<OMPTeamsDistributeSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute_simd), CollapsedNum);
}

OMPTeamsDistributeParallelForSimdDirective *
OMPTeamsDistributeParallelForSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTeamsDistributeParallelForSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute_parallel_for_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2117-2137
```cpp
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  return Dir;
}

```
- **EN**: Implements logic around `setCounters`, `setPrivateCounters`, `setInits`, `setUpdates`, and 14 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setCounters`, `setPrivateCounters`, `setInits`, `setUpdates`, and 14 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2138-2173
```cpp
OMPTeamsDistributeParallelForSimdDirective *
OMPTeamsDistributeParallelForSimdDirective::CreateEmpty(const ASTContext &C,
                                                        unsigned NumClauses,
                                                        unsigned CollapsedNum,
                                                        EmptyShell) {
  return createEmptyDirective<OMPTeamsDistributeParallelForSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute_parallel_for_simd),
      CollapsedNum);
}

OMPTeamsDistributeParallelForDirective *
OMPTeamsDistributeParallelForDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, Expr *TaskRedRef, bool HasCancel) {
  auto *Dir = createDirective<OMPTeamsDistributeParallelForDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute_parallel_for) + 1,
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2174-2199
```cpp
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->HasCancel = HasCancel;
  return Dir;
}

```
- **EN**: Implements logic around `setPrevUpperBoundVariable`, `setDistInc`, `setPrevEnsureUpperBound`, `setCounters`, and 18 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setPrevUpperBoundVariable`, `setDistInc`, `setPrevEnsureUpperBound`, `setCounters`, and 18 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2200-2218
```cpp
OMPTeamsDistributeParallelForDirective *
OMPTeamsDistributeParallelForDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    unsigned CollapsedNum,
                                                    EmptyShell) {
  return createEmptyDirective<OMPTeamsDistributeParallelForDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_teams_distribute_parallel_for) + 1,
      CollapsedNum);
}

OMPTargetTeamsDirective *OMPTargetTeamsDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt) {
  return createDirective<OMPTargetTeamsDirective>(C, Clauses, AssociatedStmt,
                                                  /*NumChildren=*/0, StartLoc,
                                                  EndLoc);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2219-2254
```cpp
OMPTargetTeamsDirective *
OMPTargetTeamsDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                     EmptyShell) {
  return createEmptyDirective<OMPTargetTeamsDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true);
}

OMPTargetTeamsDistributeDirective *OMPTargetTeamsDistributeDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTargetTeamsDistributeDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_teams_distribute), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 22 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 22 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2255-2290
```cpp
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPTargetTeamsDistributeDirective *
OMPTargetTeamsDistributeDirective::CreateEmpty(const ASTContext &C,
                                               unsigned NumClauses,
                                               unsigned CollapsedNum,
                                               EmptyShell) {
  return createEmptyDirective<OMPTargetTeamsDistributeDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_teams_distribute),
      CollapsedNum);
}

OMPTargetTeamsDistributeParallelForDirective *
OMPTargetTeamsDistributeParallelForDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, Expr *TaskRedRef, bool HasCancel) {
  auto *Dir = createDirective<OMPTargetTeamsDistributeParallelForDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_teams_distribute_parallel_for) +
          1,
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
```
- **EN**: Implements logic around `setDependentInits`, `setFinalsConditions`, `setPreInits`, `CreateEmpty`, and 13 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setDependentInits`, `setFinalsConditions`, `setPreInits`, `CreateEmpty`, and 13 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2291-2323
```cpp
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  Dir->setTaskReductionRefExpr(TaskRedRef);
  Dir->HasCancel = HasCancel;
  return Dir;
}

```
- **EN**: Implements logic around `setUpperBoundVariable`, `setStrideVariable`, `setEnsureUpperBound`, `setNextLowerBound`, and 25 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setUpperBoundVariable`, `setStrideVariable`, `setEnsureUpperBound`, `setNextLowerBound`, and 25 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2324-2359
```cpp
OMPTargetTeamsDistributeParallelForDirective *
OMPTargetTeamsDistributeParallelForDirective::CreateEmpty(const ASTContext &C,
                                                          unsigned NumClauses,
                                                          unsigned CollapsedNum,
                                                          EmptyShell) {
  return createEmptyDirective<OMPTargetTeamsDistributeParallelForDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_teams_distribute_parallel_for) +
          1,
      CollapsedNum);
}

OMPTargetTeamsDistributeParallelForSimdDirective *
OMPTargetTeamsDistributeParallelForSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTargetTeamsDistributeParallelForSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum,
                      OMPD_target_teams_distribute_parallel_for_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 15 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 15 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2360-2385
```cpp
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  return Dir;
}

```
- **EN**: Implements logic around `setNumIterations`, `setPrevLowerBoundVariable`, `setPrevUpperBoundVariable`, `setDistInc`, and 19 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setNumIterations`, `setPrevLowerBoundVariable`, `setPrevUpperBoundVariable`, `setDistInc`, and 19 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2386-2421
```cpp
OMPTargetTeamsDistributeParallelForSimdDirective *
OMPTargetTeamsDistributeParallelForSimdDirective::CreateEmpty(
    const ASTContext &C, unsigned NumClauses, unsigned CollapsedNum,
    EmptyShell) {
  return createEmptyDirective<OMPTargetTeamsDistributeParallelForSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum,
                      OMPD_target_teams_distribute_parallel_for_simd),
      CollapsedNum);
}

OMPTargetTeamsDistributeSimdDirective *
OMPTargetTeamsDistributeSimdDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTargetTeamsDistributeSimdDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_teams_distribute_simd),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2422-2443
```cpp
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPTargetTeamsDistributeSimdDirective *
OMPTargetTeamsDistributeSimdDirective::CreateEmpty(const ASTContext &C,
                                                   unsigned NumClauses,
                                                   unsigned CollapsedNum,
                                                   EmptyShell) {
  return createEmptyDirective<OMPTargetTeamsDistributeSimdDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_teams_distribute_simd),
      CollapsedNum);
}

```
- **EN**: Implements logic around `setPrivateCounters`, `setInits`, `setUpdates`, `setFinals`, and 7 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setPrivateCounters`, `setInits`, `setUpdates`, `setFinals`, and 7 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2444-2468
```cpp
OMPInteropDirective *
OMPInteropDirective::Create(const ASTContext &C, SourceLocation StartLoc,
                            SourceLocation EndLoc,
                            ArrayRef<OMPClause *> Clauses) {
  return createDirective<OMPInteropDirective>(
      C, Clauses, /*AssociatedStmt=*/nullptr, /*NumChildren=*/0, StartLoc,
      EndLoc);
}

OMPInteropDirective *OMPInteropDirective::CreateEmpty(const ASTContext &C,
                                                      unsigned NumClauses,
                                                      EmptyShell) {
  return createEmptyDirective<OMPInteropDirective>(C, NumClauses);
}

OMPDispatchDirective *OMPDispatchDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    SourceLocation TargetCallLoc) {
  auto *Dir = createDirective<OMPDispatchDirective>(
      C, Clauses, AssociatedStmt, /*NumChildren=*/0, StartLoc, EndLoc);
  Dir->setTargetCallLoc(TargetCallLoc);
  return Dir;
}

```
- **EN**: Implements logic around `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`, and 1 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `Create`, `createDirective`, `CreateEmpty`, `createEmptyDirective`, and 1 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2469-2486
```cpp
OMPDispatchDirective *OMPDispatchDirective::CreateEmpty(const ASTContext &C,
                                                        unsigned NumClauses,
                                                        EmptyShell) {
  return createEmptyDirective<OMPDispatchDirective>(C, NumClauses,
                                                    /*HasAssociatedStmt=*/true,
                                                    /*NumChildren=*/0);
}

OMPMaskedDirective *OMPMaskedDirective::Create(const ASTContext &C,
                                               SourceLocation StartLoc,
                                               SourceLocation EndLoc,
                                               ArrayRef<OMPClause *> Clauses,
                                               Stmt *AssociatedStmt) {
  return createDirective<OMPMaskedDirective>(C, Clauses, AssociatedStmt,
                                             /*NumChildren=*/0, StartLoc,
                                             EndLoc);
}

```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective` 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2487-2522
```cpp
OMPMaskedDirective *OMPMaskedDirective::CreateEmpty(const ASTContext &C,
                                                    unsigned NumClauses,
                                                    EmptyShell) {
  return createEmptyDirective<OMPMaskedDirective>(C, NumClauses,
                                                  /*HasAssociatedStmt=*/true);
}

OMPGenericLoopDirective *OMPGenericLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPGenericLoopDirective>(
      C, Clauses, AssociatedStmt, numLoopChildren(CollapsedNum, OMPD_loop),
      StartLoc, EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 23 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `Create`, `createDirective`, and 23 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2523-2558
```cpp
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPGenericLoopDirective *
OMPGenericLoopDirective::CreateEmpty(const ASTContext &C, unsigned NumClauses,
                                     unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPGenericLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_loop), CollapsedNum);
}

OMPTeamsGenericLoopDirective *OMPTeamsGenericLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTeamsGenericLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_teams_loop), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
```
- **EN**: Implements logic around `setFinalsConditions`, `setPreInits`, `CreateEmpty`, `createEmptyDirective`, and 18 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setFinalsConditions`, `setPreInits`, `CreateEmpty`, `createEmptyDirective`, and 18 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2559-2583
```cpp
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  return Dir;
}

```
- **EN**: Implements logic around `setPrevLowerBoundVariable`, `setPrevUpperBoundVariable`, `setDistInc`, `setPrevEnsureUpperBound`, and 18 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setPrevLowerBoundVariable`, `setPrevUpperBoundVariable`, `setDistInc`, `setPrevEnsureUpperBound`, and 18 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2584-2619
```cpp
OMPTeamsGenericLoopDirective *
OMPTeamsGenericLoopDirective::CreateEmpty(const ASTContext &C,
                                          unsigned NumClauses,
                                          unsigned CollapsedNum, EmptyShell) {
  return createEmptyDirective<OMPTeamsGenericLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_teams_loop), CollapsedNum);
}

OMPTargetTeamsGenericLoopDirective *OMPTargetTeamsGenericLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs, bool CanBeParallelFor) {
  auto *Dir = createDirective<OMPTargetTeamsGenericLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_teams_loop), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setPrevLowerBoundVariable(Exprs.PrevLB);
  Dir->setPrevUpperBoundVariable(Exprs.PrevUB);
  Dir->setDistInc(Exprs.DistInc);
  Dir->setPrevEnsureUpperBound(Exprs.PrevEUB);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 20 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2620-2641
```cpp
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  Dir->setCombinedLowerBoundVariable(Exprs.DistCombinedFields.LB);
  Dir->setCombinedUpperBoundVariable(Exprs.DistCombinedFields.UB);
  Dir->setCombinedEnsureUpperBound(Exprs.DistCombinedFields.EUB);
  Dir->setCombinedInit(Exprs.DistCombinedFields.Init);
  Dir->setCombinedCond(Exprs.DistCombinedFields.Cond);
  Dir->setCombinedNextLowerBound(Exprs.DistCombinedFields.NLB);
  Dir->setCombinedNextUpperBound(Exprs.DistCombinedFields.NUB);
  Dir->setCombinedDistCond(Exprs.DistCombinedFields.DistCond);
  Dir->setCombinedParForInDistCond(Exprs.DistCombinedFields.ParForInDistCond);
  Dir->setCanBeParallelFor(CanBeParallelFor);
  return Dir;
}

```
- **EN**: Implements logic around `setCounters`, `setPrivateCounters`, `setInits`, `setUpdates`, and 15 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setCounters`, `setPrivateCounters`, `setInits`, `setUpdates`, and 15 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

### Lines 2642-2677
```cpp
OMPTargetTeamsGenericLoopDirective *
OMPTargetTeamsGenericLoopDirective::CreateEmpty(const ASTContext &C,
                                                unsigned NumClauses,
                                                unsigned CollapsedNum,
                                                EmptyShell) {
  return createEmptyDirective<OMPTargetTeamsGenericLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_teams_loop), CollapsedNum);
}

OMPParallelGenericLoopDirective *OMPParallelGenericLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPParallelGenericLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_parallel_loop), StartLoc, EndLoc,
      CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
```
- **EN**: Implements logic around `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 19 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `CreateEmpty`, `createEmptyDirective`, `numLoopChildren`, `Create`, and 19 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2678-2713
```cpp
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPParallelGenericLoopDirective *OMPParallelGenericLoopDirective::CreateEmpty(
    const ASTContext &C, unsigned NumClauses, unsigned CollapsedNum,
    EmptyShell) {
  return createEmptyDirective<OMPParallelGenericLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_parallel_loop), CollapsedNum);
}

OMPTargetParallelGenericLoopDirective *
OMPTargetParallelGenericLoopDirective::Create(
    const ASTContext &C, SourceLocation StartLoc, SourceLocation EndLoc,
    unsigned CollapsedNum, ArrayRef<OMPClause *> Clauses, Stmt *AssociatedStmt,
    const HelperExprs &Exprs) {
  auto *Dir = createDirective<OMPTargetParallelGenericLoopDirective>(
      C, Clauses, AssociatedStmt,
      numLoopChildren(CollapsedNum, OMPD_target_parallel_loop), StartLoc,
      EndLoc, CollapsedNum);
  Dir->setIterationVariable(Exprs.IterationVarRef);
  Dir->setLastIteration(Exprs.LastIteration);
  Dir->setCalcLastIteration(Exprs.CalcLastIteration);
  Dir->setPreCond(Exprs.PreCond);
  Dir->setCond(Exprs.Cond);
  Dir->setInit(Exprs.Init);
  Dir->setInc(Exprs.Inc);
  Dir->setIsLastIterVariable(Exprs.IL);
  Dir->setLowerBoundVariable(Exprs.LB);
  Dir->setUpperBoundVariable(Exprs.UB);
```
- **EN**: Implements logic around `setUpdates`, `setFinals`, `setDependentCounters`, `setDependentInits`, and 17 more symbols; this block tracks source-location information and source-to-AST mapping; traverses or manipulates statement/expression trees; handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setUpdates`, `setFinals`, `setDependentCounters`, `setDependentInits`, and 17 more symbols 实现具体逻辑；该代码块跟踪源码位置信息以及源码到 AST 的映射，并遍历或操作语句/表达式树，并处理 OpenMP 专用 AST 构造。

### Lines 2714-2739
```cpp
  Dir->setStrideVariable(Exprs.ST);
  Dir->setEnsureUpperBound(Exprs.EUB);
  Dir->setNextLowerBound(Exprs.NLB);
  Dir->setNextUpperBound(Exprs.NUB);
  Dir->setNumIterations(Exprs.NumIterations);
  Dir->setCounters(Exprs.Counters);
  Dir->setPrivateCounters(Exprs.PrivateCounters);
  Dir->setInits(Exprs.Inits);
  Dir->setUpdates(Exprs.Updates);
  Dir->setFinals(Exprs.Finals);
  Dir->setDependentCounters(Exprs.DependentCounters);
  Dir->setDependentInits(Exprs.DependentInits);
  Dir->setFinalsConditions(Exprs.FinalsConditions);
  Dir->setPreInits(Exprs.PreInits);
  return Dir;
}

OMPTargetParallelGenericLoopDirective *
OMPTargetParallelGenericLoopDirective::CreateEmpty(const ASTContext &C,
                                                   unsigned NumClauses,
                                                   unsigned CollapsedNum,
                                                   EmptyShell) {
  return createEmptyDirective<OMPTargetParallelGenericLoopDirective>(
      C, NumClauses, /*HasAssociatedStmt=*/true,
      numLoopChildren(CollapsedNum, OMPD_target_parallel_loop), CollapsedNum);
}
```
- **EN**: Implements logic around `setStrideVariable`, `setEnsureUpperBound`, `setNextLowerBound`, `setNextUpperBound`, and 13 more symbols; this block handles OpenMP-specific AST constructs.
- **CN**: 围绕 `setStrideVariable`, `setEnsureUpperBound`, `setNextLowerBound`, `setNextUpperBound`, and 13 more symbols 实现具体逻辑；该代码块处理 OpenMP 专用 AST 构造。

## Key Concepts / 关键概念

- **Clang AST infrastructure / Clang AST 基础设施**:
  - **EN**: Explains how Clang stores and manipulates source-level syntax and semantic entities.
  - **CN**: 说明 Clang 如何存储并操作源码级语法与语义实体。
- **AST context ownership / AST 上下文所有权**:
  - **EN**: Centralizes allocation, uniquing, and lifetime management for AST objects.
  - **CN**: 集中管理 AST 对象的分配、唯一化与生命周期。
- **Declaration hierarchy / 声明层次结构**:
  - **EN**: Covers declaration nodes, redeclaration chains, and lookup-visible entities.
  - **CN**: 涵盖声明节点、重声明链以及参与查找的实体。
- **Statement hierarchy / 语句层次结构**:
  - **EN**: Models executable syntax nodes and traversal across statement trees.
  - **CN**: 建模可执行语法节点以及语句树遍历。
- **Expression semantics / 表达式语义**:
  - **EN**: Represents typed expression nodes, value categories, and semantic queries.
  - **CN**: 表示带类型的表达式节点、值类别以及语义查询。
- **Type-system modeling / 类型系统建模**:
  - **EN**: Captures canonical types, qualifiers, and source-facing type sugar.
  - **CN**: 刻画规范类型、限定符以及面向源码的类型语法糖。
- **Source locations / 源码位置**:
  - **EN**: Tracks source ranges, spelling locations, and mapping back to original files.
  - **CN**: 跟踪源码范围、拼写位置以及回溯到原始文件的映射。
- **OpenMP AST / OpenMP AST**:
  - **EN**: Models directives, clauses, and helper data for OpenMP constructs.
  - **CN**: 建模 OpenMP 构造的指令、子句与辅助数据。
- **Source mapping / 源码映射**:
  - **EN**: Relates AST nodes back to files, tokens, and original source ranges.
  - **CN**: 将 AST 节点关联回文件、Token 与原始源码范围。
- **Declaration names / 声明名称**:
  - **EN**: Stores lookup-oriented name forms used by declarations and templates.
  - **CN**: 存储供声明与模板查找使用的名称形式。

## Dependencies / 依赖关系

- **Direct LLVM/Clang includes / 直接的 LLVM/Clang 包含**: `clang/AST/ASTContext.h`, `clang/AST/StmtOpenMP.h`, `clang/AST/StmtNodes.inc`
- **Subsystem categories / 子系统类别**: Clang AST node definitions and semantic data structures / Clang AST 节点定义与语义数据结构 (3)
