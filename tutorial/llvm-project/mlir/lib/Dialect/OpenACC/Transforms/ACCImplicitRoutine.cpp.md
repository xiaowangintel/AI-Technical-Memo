# ACCImplicitRoutine.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Transforms/ACCImplicitRoutine.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This pass implements the implicit rules described in OpenACC specification for `Routine Directive` (OpenACC 3.4 spec, section 2.15.1).
  - **CN**: 该文件位于 `mlir/lib/Dialect/OpenACC/Transforms`，围绕 OpenACC 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ACCImplicitRoutine.cpp - OpenACC Implicit Routine Transform -------===//
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
// This pass implements the implicit rules described in OpenACC specification
// for `Routine Directive` (OpenACC 3.4 spec, section 2.15.1).
//
// "If no explicit routine directive applies to a procedure whose definition
// appears in the program unit being compiled, then the implementation applies
// an implicit routine directive to that procedure if any of the following
// conditions holds:
// - The procedure is called or its address is accessed in a compute region."
//
// The specification further states:
// "When the implementation applies an implicit routine directive to a
// procedure, it must recursively apply implicit routine directives to other
// procedures for which the above rules specify relevant dependencies. Such
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 22-35
```cpp
// dependencies can form a cycle, so the implementation must take care to avoid
// infinite recursion."
//
// This pass implements these requirements by:
// 1. Walking through all OpenACC compute constructs and functions already
//    marked with `acc routine` in the module and identifying function calls
//    within these regions.
// 2. Creating implicit `acc.routine` operations for functions that don't
//    already have routine declarations.
// 3. Recursively walking through all existing `acc routine` and creating
//    implicit routine operations for function calls within these routines,
//    while avoiding infinite recursion through proper tracking.
//
// Requirements:
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 36-46
```cpp
// -------------
// To use this pass in a pipeline, the following requirements must be met:
//
// 1. Operation Interface Implementation: Operations that define functions
//    or call functions should implement `mlir::FunctionOpInterface` and
//    `mlir::CallOpInterface` respectively.
//
// 2. Analysis Registration (Optional): If custom behavior is needed for
//    determining if a symbol use is valid within GPU regions, the dialect
//    should pre-register the `acc::OpenACCSupport` analysis.
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 47-60
```cpp

#include "mlir/Dialect/OpenACC/Transforms/Passes.h"

#include "mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h"
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinAttributes.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/Value.h"
#include "mlir/Interfaces/CallInterfaces.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include <queue>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/Builders.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/Builders.h`。

### Lines 61-69
```cpp
#define DEBUG_TYPE "acc-implicit-routine"

namespace mlir {
namespace acc {
#define GEN_PASS_DEF_ACCIMPLICITROUTINE
#include "mlir/Dialect/OpenACC/Transforms/Passes.h.inc"
} // namespace acc
} // namespace mlir

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/Transforms/Passes.h.inc`。

### Lines 70-76
```cpp
namespace {

using namespace mlir;

class ACCImplicitRoutine
    : public acc::impl::ACCImplicitRoutineBase<ACCImplicitRoutine> {
private:
```
- **EN**: Introduces declarations for `mlir`, `ACCImplicitRoutine`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir`, `ACCImplicitRoutine` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 77-84
```cpp
  unsigned routineCounter = 0;
  static constexpr llvm::StringRef accRoutinePrefix = "acc_routine_";

  // Count existing routine operations and update counter
  void initRoutineCounter(ModuleOp module) {
    module.walk([&](acc::RoutineOp routineOp) { routineCounter++; });
  }

```
- **EN**: Implements logic around `initRoutineCounter`, `walk`.
- **CN**: 围绕 `initRoutineCounter`, `walk` 实现具体逻辑。

### Lines 85-96
```cpp
  // Check if routine has a default bind clause or a device-type specific bind
  // clause. Returns true if `acc routine` has a default bind clause or
  // a device-type specific bind clause.
  bool isACCRoutineBindDefaultOrDeviceType(acc::RoutineOp op,
                                           acc::DeviceType deviceType) {
    // Fast check to avoid device-type specific lookups.
    if (!op.getBindIdName() && !op.getBindStrName())
      return false;
    return op.getBindNameValue().has_value() ||
           op.getBindNameValue(deviceType).has_value();
  }

```
- **EN**: Implements logic around `isACCRoutineBindDefaultOrDeviceType`, `getBindIdName`, `getBindNameValue`.
- **CN**: 围绕 `isACCRoutineBindDefaultOrDeviceType`, `getBindIdName`, `getBindNameValue` 实现具体逻辑。

### Lines 97-110
```cpp
  // Generate a unique name for the routine and create the routine operation
  acc::RoutineOp createRoutineOp(OpBuilder &builder, Location loc,
                                 FunctionOpInterface &callee) {
    std::string routineName =
        (accRoutinePrefix + std::to_string(routineCounter++)).str();
    auto routineOp = acc::RoutineOp::create(
        builder, loc,
        /* sym_name=*/builder.getStringAttr(routineName),
        /* func_name=*/
        mlir::SymbolRefAttr::get(builder.getContext(),
                                 builder.getStringAttr(callee.getName())),
        /* bindIdName=*/nullptr,
        /* bindStrName=*/nullptr,
        /* bindIdNameDeviceType=*/nullptr,
```
- **EN**: Implements logic around `createRoutineOp`, `to_string`, `create`, `getStringAttr`, and 1 more symbols.
- **CN**: 围绕 `createRoutineOp`, `to_string`, `create`, `getStringAttr`, and 1 more symbols 实现具体逻辑。

### Lines 111-120
```cpp
        /* bindStrNameDeviceType=*/nullptr,
        /* worker=*/nullptr,
        /* vector=*/nullptr,
        /* seq=*/nullptr,
        /* nohost=*/nullptr,
        /* implicit=*/builder.getUnitAttr(),
        /* gang=*/nullptr,
        /* gangDim=*/nullptr,
        /* gangDimDeviceType=*/nullptr);

```
- **EN**: Documents the next declarations, transformation intent, or design constraints for this file.
- **CN**: 为后续声明、变换意图或设计约束提供说明。

### Lines 121-133
```cpp
    // Assert that the callee does not already have routine info attribute
    assert(!callee->hasAttr(acc::getRoutineInfoAttrName()) &&
           "function is already associated with a routine");

    callee->setAttr(
        acc::getRoutineInfoAttrName(),
        mlir::acc::RoutineInfoAttr::get(
            builder.getContext(),
            {mlir::SymbolRefAttr::get(builder.getContext(),
                                      builder.getStringAttr(routineName))}));
    return routineOp;
  }

```
- **EN**: Implements logic around `assert`, `setAttr`, `getRoutineInfoAttrName`, `get`, and 2 more symbols.
- **CN**: 围绕 `assert`, `setAttr`, `getRoutineInfoAttrName`, `get`, and 2 more symbols 实现具体逻辑。

### Lines 134-142
```cpp
  // Used to walk through a compute region looking for function calls.
  void
  implicitRoutineForCallsInComputeRegions(Operation *op, SymbolTable &symTab,
                                          mlir::OpBuilder &builder,
                                          acc::OpenACCSupport &accSupport) {
    op->walk([&](CallOpInterface callOp) {
      if (!callOp.getCallableForCallee())
        return;

```
- **EN**: Implements logic around `implicitRoutineForCallsInComputeRegions`, `walk`, `getCallableForCallee`.
- **CN**: 围绕 `implicitRoutineForCallsInComputeRegions`, `walk`, `getCallableForCallee` 实现具体逻辑。

### Lines 143-149
```cpp
      auto calleeSymbolRef =
          dyn_cast<SymbolRefAttr>(callOp.getCallableForCallee());
      // When call is done through ssa value, the callee is not a symbol.
      // Skip it because we don't know the call target.
      if (!calleeSymbolRef)
        return;

```
- **EN**: Implements logic around `getCallableForCallee`.
- **CN**: 围绕 `getCallableForCallee` 实现具体逻辑。

### Lines 150-162
```cpp
      auto callee = symTab.lookup<FunctionOpInterface>(
          calleeSymbolRef.getLeafReference().str());
      // If the callee does not exist or is already a valid symbol for GPU
      // regions, skip it

      assert(callee && "callee function must be found in symbol table");
      if (accSupport.isValidSymbolUse(callOp.getOperation(), calleeSymbolRef))
        return;
      builder.setInsertionPoint(callee);
      createRoutineOp(builder, callee.getLoc(), callee);
    });
  }

```
- **EN**: Implements logic around `lookup`, `getLeafReference`, `assert`, `isValidSymbolUse`, and 2 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lookup`, `getLeafReference`, `assert`, `isValidSymbolUse`, and 2 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 163-174
```cpp
  // Recursively handle calls within a routine operation
  void implicitRoutineForCallsInRoutine(acc::RoutineOp routineOp,
                                        mlir::OpBuilder &builder,
                                        acc::OpenACCSupport &accSupport,
                                        acc::DeviceType targetDeviceType) {
    // When bind clause is used, it means that the target is different than the
    // function to which the `acc routine` is used with. Skip this case to
    // avoid implicitly recursively marking calls that would not end up on
    // device.
    if (isACCRoutineBindDefaultOrDeviceType(routineOp, targetDeviceType))
      return;

```
- **EN**: Implements logic around `implicitRoutineForCallsInRoutine`, `isACCRoutineBindDefaultOrDeviceType`.
- **CN**: 围绕 `implicitRoutineForCallsInRoutine`, `isACCRoutineBindDefaultOrDeviceType` 实现具体逻辑。

### Lines 175-186
```cpp
    SymbolTable symTab(routineOp->getParentOfType<ModuleOp>());
    std::queue<acc::RoutineOp> routineQueue;
    routineQueue.push(routineOp);
    while (!routineQueue.empty()) {
      auto currentRoutine = routineQueue.front();
      routineQueue.pop();
      auto func = symTab.lookup<FunctionOpInterface>(
          currentRoutine.getFuncName().getLeafReference());
      func.walk([&](CallOpInterface callOp) {
        if (!callOp.getCallableForCallee())
          return;

```
- **EN**: Implements logic around `symTab`, `push`, `empty`, `front`, and 5 more symbols.
- **CN**: 围绕 `symTab`, `push`, `empty`, `front`, and 5 more symbols 实现具体逻辑。

### Lines 187-193
```cpp
        auto calleeSymbolRef =
            dyn_cast<SymbolRefAttr>(callOp.getCallableForCallee());
        // When call is done through ssa value, the callee is not a symbol.
        // Skip it because we don't know the call target.
        if (!calleeSymbolRef)
          return;

```
- **EN**: Implements logic around `getCallableForCallee`.
- **CN**: 围绕 `getCallableForCallee` 实现具体逻辑。

### Lines 194-207
```cpp
        auto callee = symTab.lookup<FunctionOpInterface>(
            calleeSymbolRef.getLeafReference().str());
        // If the callee does not exist or is already a valid symbol for GPU
        // regions, skip it
        assert(callee && "callee function must be found in symbol table");
        if (accSupport.isValidSymbolUse(callOp.getOperation(), calleeSymbolRef))
          return;
        builder.setInsertionPoint(callee);
        auto newRoutineOp = createRoutineOp(builder, callee.getLoc(), callee);
        routineQueue.push(newRoutineOp);
      });
    }
  }

```
- **EN**: Implements logic around `lookup`, `getLeafReference`, `assert`, `isValidSymbolUse`, and 3 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lookup`, `getLeafReference`, `assert`, `isValidSymbolUse`, and 3 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 208-216
```cpp
public:
  using ACCImplicitRoutineBase<ACCImplicitRoutine>::ACCImplicitRoutineBase;

  void runOnOperation() override {
    auto module = getOperation();
    mlir::OpBuilder builder(module.getContext());
    SymbolTable symTab(module);
    initRoutineCounter(module);

```
- **EN**: Implements logic around `runOnOperation`, `getOperation`, `builder`, `symTab`, and 1 more symbols; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `getOperation`, `builder`, `symTab`, and 1 more symbols 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 217-225
```cpp
    acc::OpenACCSupport &accSupport = getAnalysis<acc::OpenACCSupport>();

    // Handle compute regions
    module.walk([&](Operation *op) {
      if (isa<ACC_COMPUTE_CONSTRUCT_OPS>(op))
        implicitRoutineForCallsInComputeRegions(op, symTab, builder,
                                                accSupport);
    });

```
- **EN**: Implements logic around `OpenACCSupport>`, `walk`, `implicitRoutineForCallsInComputeRegions`.
- **CN**: 围绕 `OpenACCSupport>`, `walk`, `implicitRoutineForCallsInComputeRegions` 实现具体逻辑。

### Lines 226-236
```cpp
    // Use the device type option from the pass options.
    acc::DeviceType targetDeviceType = deviceType;

    // Handle existing routines
    module.walk([&](acc::RoutineOp routineOp) {
      implicitRoutineForCallsInRoutine(routineOp, builder, accSupport,
                                       targetDeviceType);
    });
  }
};

```
- **EN**: Implements logic around `walk`, `implicitRoutineForCallsInRoutine`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `walk`, `implicitRoutineForCallsInRoutine` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 237-237
```cpp
} // namespace
```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

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
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/Transforms/Passes.h`, `mlir/Dialect/OpenACC/Analysis/OpenACCSupport.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinAttributes.h`, `mlir/IR/BuiltinOps.h`, `mlir/IR/Operation.h`, `mlir/IR/Value.h`, `mlir/Interfaces/CallInterfaces.h`, `mlir/Interfaces/FunctionInterfaces.h` ... (+1 more)
- **Standard-library headers / 标准库头文件**: `<queue>`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (5), dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (4), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2)
