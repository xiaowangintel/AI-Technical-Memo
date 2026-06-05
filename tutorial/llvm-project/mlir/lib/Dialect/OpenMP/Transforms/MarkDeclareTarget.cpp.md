# MarkDeclareTarget.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenMP/Transforms/MarkDeclareTarget.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Mark functions called from explicit target code as implicitly declare target.
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenMP/Transforms`，围绕 OpenMP 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MarkDeclareTarget.cpp ----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-21
```cpp
//
// Mark functions called from explicit target code as implicitly declare target.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/OpenMP/OpenMPDialect.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/TypeSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/IR/Operation.h`, `mlir/IR/SymbolTable.h`, `mlir/Interfaces/FunctionInterfaces.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/IR/Operation.h`, `mlir/IR/SymbolTable.h`, `mlir/Interfaces/FunctionInterfaces.h`。

### Lines 22-30
```cpp
namespace mlir {
namespace omp {

#define GEN_PASS_DEF_MARKDECLARETARGETPASS
#include "mlir/Dialect/OpenMP/Transforms/Passes.h.inc"

} // namespace omp
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`。

### Lines 31-37
```cpp
using namespace mlir;
namespace {

class MarkDeclareTargetPass
    : public omp::impl::MarkDeclareTargetPassBase<MarkDeclareTargetPass> {

  struct ParentInfo {
```
- **EN**: Introduces declarations for `mlir`, `MarkDeclareTargetPass`, `ParentInfo`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `MarkDeclareTargetPass`, `ParentInfo` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 38-51
```cpp
    omp::DeclareTargetDeviceType devTy;
    omp::DeclareTargetCaptureClause capClause;
    bool automap;
  };

  void processSymbolRef(SymbolRefAttr symRef, ParentInfo parentInfo,
                        llvm::SmallPtrSet<Operation *, 16> visited) {
    Operation *symOp = getOperation().lookupSymbol(symRef);
    if (!symOp)
      return;
    auto current = llvm::dyn_cast<omp::DeclareTargetInterface>(symOp);
    if (!current)
      return;

```
- **EN**: Implements logic around `processSymbolRef`, `getOperation`, `DeclareTargetInterface>`.
- **CN**: 围绕 `processSymbolRef`, `getOperation`, `DeclareTargetInterface>` 实现具体逻辑。

### Lines 52-65
```cpp
    if (current.isDeclareTarget()) {
      auto currentDt = current.getDeclareTargetDeviceType();

      // Found the same function twice, with different device_types,
      // mark as Any as it belongs to both
      if (currentDt != parentInfo.devTy &&
          currentDt != omp::DeclareTargetDeviceType::any) {
        current.setDeclareTarget(omp::DeclareTargetDeviceType::any,
                                 current.getDeclareTargetCaptureClause(),
                                 current.getDeclareTargetAutomap());
      }
    } else {
      current.setDeclareTarget(parentInfo.devTy, parentInfo.capClause,
                               parentInfo.automap);
```
- **EN**: Implements logic around `isDeclareTarget`, `getDeclareTargetDeviceType`, `setDeclareTarget`, `getDeclareTargetCaptureClause`, and 1 more symbols.
- **CN**: 围绕 `isDeclareTarget`, `getDeclareTargetDeviceType`, `setDeclareTarget`, `getDeclareTargetCaptureClause`, and 1 more symbols 实现具体逻辑。

### Lines 66-76
```cpp
    }

    markNestedFuncs(parentInfo, symOp, visited);
  }

  void processReductionRefs(std::optional<mlir::ArrayAttr> symRefs,
                            ParentInfo parentInfo,
                            llvm::SmallPtrSet<Operation *, 16> visited) {
    if (!symRefs)
      return;

```
- **EN**: Implements logic around `markNestedFuncs`, `processReductionRefs`.
- **CN**: 围绕 `markNestedFuncs`, `processReductionRefs` 实现具体逻辑。

### Lines 77-84
```cpp
    for (auto symRef : symRefs->getAsRange<mlir::SymbolRefAttr>()) {
      if (auto declareReductionOp =
              getOperation().lookupSymbol<omp::DeclareReductionOp>(symRef)) {
        markNestedFuncs(parentInfo, declareReductionOp, visited);
      }
    }
  }

```
- **EN**: Implements logic around `SymbolRefAttr>`, `getOperation`, `markNestedFuncs`.
- **CN**: 围绕 `SymbolRefAttr>`, `getOperation`, `markNestedFuncs` 实现具体逻辑。

### Lines 85-98
```cpp
  void processReductionClauses(Operation *op, ParentInfo parentInfo,
                               llvm::SmallPtrSet<Operation *, 16> visited) {
    llvm::TypeSwitch<Operation &>(*op)
        .Case([&](omp::LoopOp op) {
          processReductionRefs(op.getReductionSyms(), parentInfo, visited);
        })
        .Case([&](omp::ParallelOp op) {
          processReductionRefs(op.getReductionSyms(), parentInfo, visited);
        })
        .Case([&](omp::SectionsOp op) {
          processReductionRefs(op.getReductionSyms(), parentInfo, visited);
        })
        .Case([&](omp::SimdOp op) {
          processReductionRefs(op.getReductionSyms(), parentInfo, visited);
```
- **EN**: Implements logic around `processReductionClauses`, `Case`, `processReductionRefs`.
- **CN**: 围绕 `processReductionClauses`, `Case`, `processReductionRefs` 实现具体逻辑。

### Lines 99-112
```cpp
        })
        .Case([&](omp::TargetOp op) {
          processReductionRefs(op.getInReductionSyms(), parentInfo, visited);
        })
        .Case([&](omp::TaskgroupOp op) {
          processReductionRefs(op.getTaskReductionSyms(), parentInfo, visited);
        })
        .Case([&](omp::TaskloopContextOp op) {
          processReductionRefs(op.getReductionSyms(), parentInfo, visited);
          processReductionRefs(op.getInReductionSyms(), parentInfo, visited);
        })
        .Case([&](omp::TaskOp op) {
          processReductionRefs(op.getInReductionSyms(), parentInfo, visited);
        })
```
- **EN**: Implements logic around `Case`, `processReductionRefs`.
- **CN**: 围绕 `Case`, `processReductionRefs` 实现具体逻辑。

### Lines 113-121
```cpp
        .Case([&](omp::TeamsOp op) {
          processReductionRefs(op.getReductionSyms(), parentInfo, visited);
        })
        .Case([&](omp::WsloopOp op) {
          processReductionRefs(op.getReductionSyms(), parentInfo, visited);
        })
        .Default([](Operation &) {});
  }

```
- **EN**: Implements logic around `Case`, `processReductionRefs`, `Default`.
- **CN**: 围绕 `Case`, `processReductionRefs`, `Default` 实现具体逻辑。

### Lines 122-135
```cpp
  void markNestedFuncs(ParentInfo parentInfo, Operation *currOp,
                       llvm::SmallPtrSet<Operation *, 16> visited) {
    if (visited.contains(currOp))
      return;
    visited.insert(currOp);

    currOp->walk([&, this](Operation *op) {
      if (auto callOp = llvm::dyn_cast<CallOpInterface>(op)) {
        if (auto symRef = llvm::dyn_cast_if_present<mlir::SymbolRefAttr>(
                callOp.getCallableForCallee())) {
          processSymbolRef(symRef, parentInfo, visited);
        }
      }
      processReductionClauses(op, parentInfo, visited);
```
- **EN**: Implements logic around `markNestedFuncs`, `contains`, `insert`, `walk`, and 4 more symbols.
- **CN**: 围绕 `markNestedFuncs`, `contains`, `insert`, `walk`, and 4 more symbols 实现具体逻辑。

### Lines 136-149
```cpp
    });
  }

  // This pass executes on mlir::ModuleOp's marking functions contained within
  // as implicitly declare target if they are called from within an explicitly
  // marked declare target function or a target region (TargetOp)
  void runOnOperation() override {
    for (auto funcOp : getOperation().getOps<FunctionOpInterface>()) {
      auto declareTargetOp =
          llvm::dyn_cast<omp::DeclareTargetInterface>(funcOp.getOperation());
      if (!declareTargetOp || !declareTargetOp.isDeclareTarget())
        continue;
      llvm::SmallPtrSet<Operation *, 16> visited;
      ParentInfo parentInfo{declareTargetOp.getDeclareTargetDeviceType(),
```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `DeclareTargetInterface>`, `isDeclareTarget`, and 1 more symbols; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `DeclareTargetInterface>`, `isDeclareTarget`, and 1 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 150-163
```cpp
                            declareTargetOp.getDeclareTargetCaptureClause(),
                            declareTargetOp.getDeclareTargetAutomap()};
      markNestedFuncs(parentInfo, funcOp, visited);
    }

    // TODO: Extend to work with reverse-offloading, this shouldn't
    // require too much effort, just need to check the device clause
    // when it's lowering has been implemented and change the
    // DeclareTargetDeviceType argument from nohost to host depending on
    // the contents of the device clause
    getOperation()->walk([&](omp::TargetOp tarOp) {
      llvm::SmallPtrSet<Operation *, 16> visited;
      ParentInfo parentInfo = {
          /*devTy=*/omp::DeclareTargetDeviceType::nohost,
```
- **EN**: Implements logic around `getDeclareTargetCaptureClause`, `getDeclareTargetAutomap`, `markNestedFuncs`, `getOperation`; this block coordinates dialect conversion or lowering decisions.
- **CN**: 围绕 `getDeclareTargetCaptureClause`, `getDeclareTargetAutomap`, `markNestedFuncs`, `getOperation` 实现具体逻辑；该代码块协调方言转换或 lowering 决策。

### Lines 164-171
```cpp
          /*capClause=*/omp::DeclareTargetCaptureClause::to,
          /*automap=*/false,
      };
      markNestedFuncs(parentInfo, tarOp, visited);
    });
  }
};
} // namespace
```
- **EN**: Implements logic around `markNestedFuncs`.
- **CN**: 围绕 `markNestedFuncs` 实现具体逻辑。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenMP/OpenMPDialect.h`, `mlir/IR/Operation.h`, `mlir/IR/SymbolTable.h`, `mlir/Interfaces/FunctionInterfaces.h`, `mlir/Pass/Pass.h`, `mlir/Support/LLVM.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/TypeSwitch.h`, `mlir/Dialect/OpenMP/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (2), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1), pass infrastructure and registration support / Pass 基础设施与注册支持 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1)
