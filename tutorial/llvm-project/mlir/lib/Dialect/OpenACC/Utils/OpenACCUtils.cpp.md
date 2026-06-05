# OpenACCUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/OpenACC/Utils/OpenACCUtils.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements OpenACC dialect support for utility helpers shared by the dialect implementation, centered on `OpenACCUtils`.
  - **CN**: 实现 OpenACC 方言中围绕 `OpenACCUtils` 的方言实现共享的工具辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- OpenACCUtils.cpp ---------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/OpenACC/OpenACCUtils.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/OpenACCUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/OpenACCUtils.h`。

### Lines 11-22
```cpp
#include "mlir/Dialect/OpenACC/OpenACC.h"
#include "mlir/Dialect/Utils/StaticValueUtils.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Dominance.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include "mlir/Interfaces/ViewLikeInterface.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/Support/Casting.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Dominance.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Dominance.h`。

### Lines 23-32
```cpp
mlir::Operation *mlir::acc::getEnclosingComputeOp(mlir::Region &region) {
  return region
      .getParentOfType<ACC_COMPUTE_CONSTRUCT_OPS, mlir::acc::ComputeRegionOp>();
}

mlir::Operation *mlir::acc::getACCDataClauseOpForBlockArg(mlir::Value v) {
  auto barg = mlir::dyn_cast<mlir::BlockArgument>(v);
  if (!barg)
    return nullptr;

```
- **EN**: Implements logic around `getEnclosingComputeOp`, `ComputeRegionOp>`, `getACCDataClauseOpForBlockArg`, `BlockArgument>`.
- **CN**: 围绕 `getEnclosingComputeOp`, `ComputeRegionOp>`, `getACCDataClauseOpForBlockArg`, `BlockArgument>` 实现具体逻辑。

### Lines 33-45
```cpp
  mlir::Block *block = barg.getOwner();
  auto computeReg =
      mlir::dyn_cast<mlir::acc::ComputeRegionOp>(block->getParentOp());
  if (!computeReg || block != computeReg.getBody())
    return nullptr;

  mlir::Value orig = computeReg.getOperand(barg);
  if (!orig)
    return nullptr;
  mlir::Operation *def = orig.getDefiningOp();
  return mlir::isa_and_nonnull<ACC_DATA_ENTRY_OPS>(def) ? def : nullptr;
}

```
- **EN**: Implements logic around `getOwner`, `ComputeRegionOp>`, `getBody`, `getOperand`, and 2 more symbols.
- **CN**: 围绕 `getOwner`, `ComputeRegionOp>`, `getBody`, `getOperand`, and 2 more symbols 实现具体逻辑。

### Lines 46-55
```cpp
template <typename OpTy>
static bool isOnlyUsedByOpClauses(mlir::Value val, mlir::Region &region) {
  auto checkIfUsedOnlyByOpInside = [&](mlir::Operation *user) {
    // For any users which are not in the current acc region, we can ignore.
    // Return true so that it can be used in a `all_of` check.
    if (!region.isAncestor(user->getParentRegion()))
      return true;
    return mlir::isa<OpTy>(user);
  };

```
- **EN**: Implements logic around `isOnlyUsedByOpClauses`, `isAncestor`.
- **CN**: 围绕 `isOnlyUsedByOpClauses`, `isAncestor` 实现具体逻辑。

### Lines 56-68
```cpp
  return llvm::all_of(val.getUsers(), checkIfUsedOnlyByOpInside);
}

bool mlir::acc::isOnlyUsedByPrivateClauses(mlir::Value val,
                                           mlir::Region &region) {
  return isOnlyUsedByOpClauses<mlir::acc::PrivateOp>(val, region);
}

bool mlir::acc::isOnlyUsedByReductionClauses(mlir::Value val,
                                             mlir::Region &region) {
  return isOnlyUsedByOpClauses<mlir::acc::ReductionOp>(val, region);
}

```
- **EN**: Implements logic around `all_of`, `isOnlyUsedByPrivateClauses`, `PrivateOp>`, `isOnlyUsedByReductionClauses`, and 1 more symbols.
- **CN**: 围绕 `all_of`, `isOnlyUsedByPrivateClauses`, `PrivateOp>`, `isOnlyUsedByReductionClauses`, and 1 more symbols 实现具体逻辑。

### Lines 69-87
```cpp
std::optional<mlir::acc::ClauseDefaultValue>
mlir::acc::getDefaultAttr(Operation *op) {
  std::optional<mlir::acc::ClauseDefaultValue> defaultAttr;
  Operation *currOp = op;

  // Iterate outwards until a default clause is found (since OpenACC
  // specification notes that a visible default clause is the nearest default
  // clause appearing on the compute construct or a lexically containing data
  // construct.
  while (!defaultAttr.has_value() && currOp) {
    defaultAttr =
        llvm::TypeSwitch<mlir::Operation *,
                         std::optional<mlir::acc::ClauseDefaultValue>>(currOp)
            .Case<ACC_COMPUTE_CONSTRUCT_OPS, mlir::acc::DataOp>(
                [&](auto op) { return op.getDefaultAttr(); })
            .Default([&](Operation *) { return std::nullopt; });
    currOp = currOp->getParentOp();
  }

```
- **EN**: Implements logic around `getDefaultAttr`, `has_value`, `ClauseDefaultValue>>`, `DataOp>`, and 2 more symbols.
- **CN**: 围绕 `getDefaultAttr`, `has_value`, `ClauseDefaultValue>>`, `DataOp>`, and 2 more symbols 实现具体逻辑。

### Lines 88-103
```cpp
  return defaultAttr;
}

mlir::acc::VariableTypeCategory mlir::acc::getTypeCategory(mlir::Value var) {
  mlir::acc::VariableTypeCategory typeCategory =
      mlir::acc::VariableTypeCategory::uncategorized;
  if (auto mappableTy = dyn_cast<mlir::acc::MappableType>(var.getType()))
    typeCategory = mappableTy.getTypeCategory(var);
  else if (auto pointerLikeTy =
               dyn_cast<mlir::acc::PointerLikeType>(var.getType()))
    typeCategory = pointerLikeTy.getPointeeTypeCategory(
        cast<TypedValue<mlir::acc::PointerLikeType>>(var),
        pointerLikeTy.getElementType());
  return typeCategory;
}

```
- **EN**: Implements logic around `getTypeCategory`, `MappableType>`, `PointerLikeType>`, `getPointeeTypeCategory`, and 2 more symbols.
- **CN**: 围绕 `getTypeCategory`, `MappableType>`, `PointerLikeType>`, `getPointeeTypeCategory`, and 2 more symbols 实现具体逻辑。

### Lines 104-117
```cpp
std::string mlir::acc::getVariableName(mlir::Value v) {
  Value current = v;

  // Walk through view operations until a name is found or can't go further
  while (Operation *definingOp = current.getDefiningOp()) {
    // For integer constants, return their value as a string.
    if (std::optional<int64_t> constVal = getConstantIntValue(current))
      return std::to_string(*constVal);

    // Check for `acc.var_name` attribute
    if (auto varNameAttr =
            definingOp->getAttrOfType<VarNameAttr>(getVarNameAttrName()))
      return varNameAttr.getName().str();

```
- **EN**: Implements logic around `getVariableName`, `getDefiningOp`, `getConstantIntValue`, `to_string`, and 2 more symbols.
- **CN**: 围绕 `getVariableName`, `getDefiningOp`, `getConstantIntValue`, `to_string`, and 2 more symbols 实现具体逻辑。

### Lines 118-128
```cpp
    // If it is a data entry operation, get name via getVarName
    if (isa<ACC_DATA_ENTRY_OPS>(definingOp))
      if (auto name = acc::getVarName(definingOp))
        return name->str();

    // If it's a view operation, continue to the source
    if (auto viewOp = dyn_cast<ViewLikeOpInterface>(definingOp)) {
      current = viewOp.getViewSource();
      continue;
    }

```
- **EN**: Implements logic around `getVarName`, `str`, `getViewSource`.
- **CN**: 围绕 `getVarName`, `str`, `getViewSource` 实现具体逻辑。

### Lines 129-142
```cpp
    break;
  }

  return "";
}

std::string mlir::acc::getRecipeName(mlir::acc::RecipeKind kind,
                                     mlir::Type type) {
  assert(kind == mlir::acc::RecipeKind::private_recipe ||
         kind == mlir::acc::RecipeKind::firstprivate_recipe ||
         kind == mlir::acc::RecipeKind::reduction_recipe);
  if (!llvm::isa<mlir::acc::PointerLikeType, mlir::acc::MappableType>(type))
    return "";

```
- **EN**: Implements logic around `getRecipeName`, `assert`, `MappableType>`.
- **CN**: 围绕 `getRecipeName`, `assert`, `MappableType>` 实现具体逻辑。

### Lines 143-153
```cpp
  std::string recipeName;
  llvm::raw_string_ostream ss(recipeName);
  ss << (kind == mlir::acc::RecipeKind::private_recipe ? "privatization_"
         : kind == mlir::acc::RecipeKind::firstprivate_recipe
             ? "firstprivatization_"
             : "reduction_");

  // Print the type using its dialect-defined textual format.
  type.print(ss);
  ss.flush();

```
- **EN**: Implements logic around `ss`, `print`, `flush`; this block handles textual assembly parsing or printing concerns.
- **CN**: 围绕 `ss`, `print`, `flush` 实现具体逻辑；该代码块处理文本汇编解析或打印相关问题。

### Lines 154-169
```cpp
  // Replace invalid characters (anything that's not a letter, number, or
  // period) since this needs to be a valid MLIR identifier.
  for (char &c : recipeName) {
    if (!std::isalnum(static_cast<unsigned char>(c)) && c != '.' && c != '_') {
      if (c == '?')
        c = 'U';
      else if (c == '*')
        c = 'Z';
      else if (c == '(' || c == ')' || c == '[' || c == ']' || c == '{' ||
               c == '}' || c == '<' || c == '>')
        c = '_';
      else
        c = 'X';
    }
  }

```
- **EN**: Implements logic around `isalnum`.
- **CN**: 围绕 `isalnum` 实现具体逻辑。

### Lines 170-179
```cpp
  return recipeName;
}

mlir::Value mlir::acc::getBaseEntity(mlir::Value val) {
  if (auto partialEntityAccessOp =
          val.getDefiningOp<PartialEntityAccessOpInterface>()) {
    if (!partialEntityAccessOp.isCompleteView())
      return partialEntityAccessOp.getBaseEntity();
  }

```
- **EN**: Implements logic around `getBaseEntity`, `getDefiningOp`, `isCompleteView`.
- **CN**: 围绕 `getBaseEntity`, `getDefiningOp`, `isCompleteView` 实现具体逻辑。

### Lines 180-193
```cpp
  return val;
}

bool mlir::acc::isValidSymbolUse(mlir::Operation *user,
                                 mlir::SymbolRefAttr symbol,
                                 mlir::Operation **definingOpPtr) {
  mlir::Operation *definingOp =
      mlir::SymbolTable::lookupNearestSymbolFrom(user, symbol);

  // If there are no defining ops, we have no way to ensure validity because
  // we cannot check for any attributes.
  if (!definingOp)
    return false;

```
- **EN**: Implements logic around `isValidSymbolUse`, `lookupNearestSymbolFrom`.
- **CN**: 围绕 `isValidSymbolUse`, `lookupNearestSymbolFrom` 实现具体逻辑。

### Lines 194-203
```cpp
  if (definingOpPtr)
    *definingOpPtr = definingOp;

  // Check if the defining op is a recipe (private, reduction, firstprivate).
  // Recipes are valid as they get materialized before being offloaded to
  // device. They are only instructions for how to materialize.
  if (mlir::isa<mlir::acc::PrivateRecipeOp, mlir::acc::ReductionRecipeOp,
                mlir::acc::FirstprivateRecipeOp>(definingOp))
    return true;

```
- **EN**: Implements logic around `FirstprivateRecipeOp>`.
- **CN**: 围绕 `FirstprivateRecipeOp>` 实现具体逻辑。

### Lines 204-218
```cpp
  // Check if the defining op is a global variable that is device data.
  // Device data is already resident on the device and does not need mapping.
  if (auto globalVar =
          mlir::dyn_cast<mlir::acc::GlobalVariableOpInterface>(definingOp))
    if (globalVar.isDeviceData())
      return true;

  // Check if the defining op is a function
  if (auto func =
          mlir::dyn_cast_if_present<mlir::FunctionOpInterface>(definingOp)) {
    // If this symbol is actually an acc routine - then it is expected for it
    // to be offloaded - therefore it is valid.
    if (func->hasAttr(mlir::acc::getRoutineInfoAttrName()))
      return true;

```
- **EN**: Implements logic around `GlobalVariableOpInterface>`, `isDeviceData`, `FunctionOpInterface>`, `hasAttr`.
- **CN**: 围绕 `GlobalVariableOpInterface>`, `isDeviceData`, `FunctionOpInterface>`, `hasAttr` 实现具体逻辑。

### Lines 219-231
```cpp
    // If this symbol is a call to an LLVM intrinsic, then it is likely valid.
    // Check the following:
    // 1. The function is private
    // 2. The function has no body
    // 3. Name starts with "llvm."
    // 4. The function's name is a valid LLVM intrinsic name
    if (func.getVisibility() == mlir::SymbolTable::Visibility::Private &&
        func.getFunctionBody().empty() && func.getName().starts_with("llvm.") &&
        llvm::Intrinsic::lookupIntrinsicID(func.getName()) !=
            llvm::Intrinsic::not_intrinsic)
      return true;
  }

```
- **EN**: Implements logic around `getVisibility`, `getFunctionBody`, `lookupIntrinsicID`.
- **CN**: 围绕 `getVisibility`, `getFunctionBody`, `lookupIntrinsicID` 实现具体逻辑。

### Lines 232-243
```cpp
  // A declare attribute is needed for symbol references.
  bool hasDeclare = definingOp->hasAttr(mlir::acc::getDeclareAttrName());
  return hasDeclare;
}

bool mlir::acc::isDeviceValue(mlir::Value val) {
  // Check if the value is device data via type interfaces.
  // Device data is already resident on the device and does not need mapping.
  if (auto mappableTy = dyn_cast<mlir::acc::MappableType>(val.getType()))
    if (mappableTy.isDeviceData(val))
      return true;

```
- **EN**: Implements logic around `hasAttr`, `isDeviceValue`, `MappableType>`, `isDeviceData`.
- **CN**: 围绕 `hasAttr`, `isDeviceValue`, `MappableType>`, `isDeviceData` 实现具体逻辑。

### Lines 244-259
```cpp
  if (auto pointerLikeTy = dyn_cast<mlir::acc::PointerLikeType>(val.getType()))
    if (pointerLikeTy.isDeviceData(val))
      return true;

  mlir::Operation *defOp = val.getDefiningOp();
  if (!defOp)
    return false;

  // `acc.declare` with deviceptr marks data that is already associated with
  // the device.
  if (auto declareAttr = defOp->getAttrOfType<mlir::acc::DeclareAttr>(
          mlir::acc::getDeclareAttrName()))
    if (declareAttr.getDataClause().getValue() ==
        mlir::acc::DataClause::acc_deviceptr)
      return true;

```
- **EN**: Implements logic around `PointerLikeType>`, `isDeviceData`, `getDefiningOp`, `DeclareAttr>`, and 2 more symbols.
- **CN**: 围绕 `PointerLikeType>`, `isDeviceData`, `getDefiningOp`, `DeclareAttr>`, and 2 more symbols 实现具体逻辑。

### Lines 260-276
```cpp
  // Handle operations that access a partial entity - check if the base entity
  // is device data.
  if (auto partialAccess =
          dyn_cast<mlir::acc::PartialEntityAccessOpInterface>(defOp)) {
    if (mlir::Value base = partialAccess.getBaseEntity())
      return isDeviceValue(base);
  }

  // Handle address_of - check if the referenced global is device data.
  if (auto addrOfIface =
          dyn_cast<mlir::acc::AddressOfGlobalOpInterface>(defOp)) {
    auto symbol = addrOfIface.getSymbol();
    if (auto global = mlir::SymbolTable::lookupNearestSymbolFrom<
            mlir::acc::GlobalVariableOpInterface>(defOp, symbol))
      return global.isDeviceData();
  }

```
- **EN**: Implements logic around `PartialEntityAccessOpInterface>`, `getBaseEntity`, `isDeviceValue`, `AddressOfGlobalOpInterface>`, and 3 more symbols.
- **CN**: 围绕 `PartialEntityAccessOpInterface>`, `getBaseEntity`, `isDeviceValue`, `AddressOfGlobalOpInterface>`, and 3 more symbols 实现具体逻辑。

### Lines 277-286
```cpp
  return false;
}

bool mlir::acc::isValidValueUse(mlir::Value val, mlir::Region &region) {
  // Types that can be passed by value are legal.
  Type type = val.getType();
  if (type.isIntOrIndexOrFloat() || isa<mlir::ComplexType>(type) ||
      llvm::isa<mlir::VectorType>(type))
    return true;

```
- **EN**: Implements logic around `isValidValueUse`, `getType`, `isIntOrIndexOrFloat`, `VectorType>`.
- **CN**: 围绕 `isValidValueUse`, `getType`, `isIntOrIndexOrFloat`, `VectorType>` 实现具体逻辑。

### Lines 287-298
```cpp
  // If this is produced by an ACC data entry operation, it is valid.
  if (isa_and_nonnull<ACC_DATA_ENTRY_OPS>(val.getDefiningOp()))
    return true;

  // If the value is only used by private clauses, it is not a live-in.
  if (isOnlyUsedByPrivateClauses(val, region))
    return true;

  // If this is device data, it is valid.
  if (isDeviceValue(val))
    return true;

```
- **EN**: Implements logic around `isa_and_nonnull`, `isOnlyUsedByPrivateClauses`, `isDeviceValue`.
- **CN**: 围绕 `isa_and_nonnull`, `isOnlyUsedByPrivateClauses`, `isDeviceValue` 实现具体逻辑。

### Lines 299-316
```cpp
  return false;
}

llvm::SmallVector<mlir::Value>
mlir::acc::getDominatingDataClauses(mlir::Operation *computeConstructOp,
                                    mlir::DominanceInfo &domInfo,
                                    mlir::PostDominanceInfo &postDomInfo) {
  llvm::SmallSetVector<mlir::Value, 8> dominatingDataClauses;

  llvm::TypeSwitch<mlir::Operation *>(computeConstructOp)
      .Case<mlir::acc::ParallelOp, mlir::acc::KernelsOp, mlir::acc::SerialOp>(
          [&](auto op) {
            for (auto dataClause : op.getDataClauseOperands()) {
              dominatingDataClauses.insert(dataClause);
            }
          })
      .Default([](mlir::Operation *) {});

```
- **EN**: Implements logic around `getDominatingDataClauses`, `SerialOp>`, `getDataClauseOperands`, `insert`, and 1 more symbols.
- **CN**: 围绕 `getDominatingDataClauses`, `SerialOp>`, `getDataClauseOperands`, `insert`, and 1 more symbols 实现具体逻辑。

### Lines 317-328
```cpp
  // Collect the data clauses from enclosing data constructs.
  mlir::Operation *currParentOp = computeConstructOp->getParentOp();
  while (currParentOp) {
    if (mlir::isa<mlir::acc::DataOp>(currParentOp)) {
      for (auto dataClause : mlir::dyn_cast<mlir::acc::DataOp>(currParentOp)
                                 .getDataClauseOperands()) {
        dominatingDataClauses.insert(dataClause);
      }
    }
    currParentOp = currParentOp->getParentOp();
  }

```
- **EN**: Implements logic around `getParentOp`, `DataOp>`, `getDataClauseOperands`, `insert`.
- **CN**: 围绕 `getParentOp`, `DataOp>`, `getDataClauseOperands`, `insert` 实现具体逻辑。

### Lines 329-345
```cpp
  // Find the enclosing function/subroutine
  auto funcOp =
      computeConstructOp->getParentOfType<mlir::FunctionOpInterface>();
  if (!funcOp)
    return dominatingDataClauses.takeVector();

  // Walk the function to find `acc.declare_enter`/`acc.declare_exit` pairs that
  // dominate and post-dominate the compute construct and add their data
  // clauses to the list.
  funcOp->walk([&](mlir::acc::DeclareEnterOp declareEnterOp) {
    if (domInfo.dominates(declareEnterOp.getOperation(), computeConstructOp)) {
      // Collect all `acc.declare_exit` ops for this token.
      llvm::SmallVector<mlir::acc::DeclareExitOp> exits;
      for (auto *user : declareEnterOp.getToken().getUsers())
        if (auto declareExit = mlir::dyn_cast<mlir::acc::DeclareExitOp>(user))
          exits.push_back(declareExit);

```
- **EN**: Implements logic around `FunctionOpInterface>`, `takeVector`, `walk`, `dominates`, and 3 more symbols.
- **CN**: 围绕 `FunctionOpInterface>`, `takeVector`, `walk`, `dominates`, and 3 more symbols 实现具体逻辑。

### Lines 346-357
```cpp
      // Only add clauses if every `acc.declare_exit` op post-dominates the
      // compute construct.
      if (!exits.empty() &&
          llvm::all_of(exits, [&](mlir::acc::DeclareExitOp exitOp) {
            return postDomInfo.postDominates(exitOp, computeConstructOp);
          })) {
        for (auto dataClause : declareEnterOp.getDataClauseOperands())
          dominatingDataClauses.insert(dataClause);
      }
    }
  });

```
- **EN**: Implements logic around `empty`, `all_of`, `postDominates`, `getDataClauseOperands`, and 1 more symbols.
- **CN**: 围绕 `empty`, `all_of`, `postDominates`, `getDataClauseOperands`, and 1 more symbols 实现具体逻辑。

### Lines 358-370
```cpp
  return dominatingDataClauses.takeVector();
}

mlir::remark::detail::InFlightRemark
mlir::acc::emitRemark(mlir::Operation *op,
                      const std::function<std::string()> &messageFn,
                      llvm::StringRef category) {
  using namespace mlir::remark;
  mlir::Location loc = op->getLoc();
  auto *engine = loc->getContext()->getRemarkEngine();
  if (!engine)
    return remark::detail::InFlightRemark{};

```
- **EN**: Introduces declarations for `mlir::remark`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir::remark` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 371-380
```cpp
  llvm::StringRef funcName;
  if (auto func = dyn_cast<mlir::FunctionOpInterface>(op))
    funcName = func.getName();
  else if (auto funcOp = op->getParentOfType<mlir::FunctionOpInterface>())
    funcName = funcOp.getName();

  auto opts = RemarkOpts::name("openacc").category(category);
  if (!funcName.empty())
    opts = opts.function(funcName);

```
- **EN**: Implements logic around `FunctionOpInterface>`, `getName`, `name`, `empty`, and 1 more symbols.
- **CN**: 围绕 `FunctionOpInterface>`, `getName`, `name`, `empty`, and 1 more symbols 实现具体逻辑。

### Lines 381-385
```cpp
  auto remark = engine->emitOptimizationRemark(loc, opts);
  if (remark)
    remark << messageFn();
  return remark;
}
```
- **EN**: Implements logic around `emitOptimizationRemark`, `messageFn`.
- **CN**: 围绕 `emitOptimizationRemark`, `messageFn` 实现具体逻辑。

## Key Concepts / 关键概念

- **Dialect utilities / 方言工具**:
  - **EN**: Collects reusable helpers that keep core dialect logic factored and shareable.
  - **CN**: 汇集可复用辅助函数，使核心方言逻辑保持解耦并可共享。
- **Textual assembly syntax / 文本汇编语法**:
  - **EN**: Implements dialect-specific parsing and printing hooks for MLIR assembly.
  - **CN**: 实现 MLIR 汇编的方言专用解析与打印钩子。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/OpenACC/OpenACCUtils.h`, `mlir/Dialect/OpenACC/OpenACC.h`, `mlir/Dialect/Utils/StaticValueUtils.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Dominance.h`, `mlir/IR/SymbolTable.h`, `mlir/Interfaces/FunctionInterfaces.h`, `mlir/Interfaces/ViewLikeInterface.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/TypeSwitch.h` ... (+2 more)
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (2), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), LLVM subsystem declarations / LLVM 子系统声明 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
