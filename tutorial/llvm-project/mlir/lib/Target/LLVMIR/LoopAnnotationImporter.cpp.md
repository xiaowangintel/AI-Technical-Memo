# LoopAnnotationImporter.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Target/LLVMIR/LoopAnnotationImporter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR target import/export, translation, or serialization support.
  - **CN**: 实现 MLIR 目标导入/导出、翻译或序列化支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- LoopAnnotationImporter.cpp - Loop annotation import ----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "LoopAnnotationImporter.h"
#include "llvm/IR/Constants.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `LoopAnnotationImporter.h`, `llvm/IR/Constants.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `LoopAnnotationImporter.h`, `llvm/IR/Constants.h`。

### Lines 12-23
```cpp
using namespace mlir;
using namespace mlir::LLVM;
using namespace mlir::LLVM::detail;

namespace {
/// Helper class that keeps the state of one metadata to attribute conversion.
struct LoopMetadataConversion {
  LoopMetadataConversion(const llvm::MDNode *node, Location loc,
                         LoopAnnotationImporter &loopAnnotationImporter)
      : node(node), loc(loc), loopAnnotationImporter(loopAnnotationImporter),
        ctx(loc->getContext()){};
  /// Converts this structs loop metadata node into a LoopAnnotationAttr.
```
- **EN**: Introduces declarations for `LoopMetadataConversion`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `LoopMetadataConversion` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 24-33
```cpp
  LoopAnnotationAttr convert();

  /// Initializes the shared state for the conversion member functions.
  LogicalResult initConversionState();

  /// Helper function to get and erase a property.
  const llvm::MDNode *lookupAndEraseProperty(StringRef name);

  /// Helper functions to lookup and convert MDNodes into a specifc attribute
  /// kind. These functions return null-attributes if there is no node with the
```
- **EN**: Implements logic around `convert`, `initConversionState`, `lookupAndEraseProperty`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `convert`、`initConversionState`、`lookupAndEraseProperty` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 34-45
```cpp
  /// specified name, or failure, if the node is ill-formatted.
  FailureOr<BoolAttr> lookupUnitNode(StringRef name);
  FailureOr<BoolAttr> lookupBoolNode(StringRef name, bool negated = false);
  FailureOr<BoolAttr> lookupIntNodeAsBoolAttr(StringRef name);
  FailureOr<IntegerAttr> lookupIntNode(StringRef name);
  FailureOr<llvm::MDNode *> lookupMDNode(StringRef name);
  FailureOr<SmallVector<llvm::MDNode *>> lookupMDNodes(StringRef name);
  FailureOr<LoopAnnotationAttr> lookupFollowupNode(StringRef name);
  FailureOr<BoolAttr> lookupBooleanUnitNode(StringRef enableName,
                                            StringRef disableName,
                                            bool negated = false);

```
- **EN**: Implements logic around `lookupUnitNode`, `lookupBoolNode`, `lookupIntNodeAsBoolAttr`, `lookupIntNode`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupUnitNode`、`lookupBoolNode`、`lookupIntNodeAsBoolAttr`、`lookupIntNode` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 46-59
```cpp
  /// Conversion functions for sub-attributes.
  FailureOr<LoopVectorizeAttr> convertVectorizeAttr();
  FailureOr<LoopInterleaveAttr> convertInterleaveAttr();
  FailureOr<LoopUnrollAttr> convertUnrollAttr();
  FailureOr<LoopUnrollAndJamAttr> convertUnrollAndJamAttr();
  FailureOr<LoopLICMAttr> convertLICMAttr();
  FailureOr<LoopDistributeAttr> convertDistributeAttr();
  FailureOr<LoopPipelineAttr> convertPipelineAttr();
  FailureOr<LoopPeeledAttr> convertPeeledAttr();
  FailureOr<LoopUnswitchAttr> convertUnswitchAttr();
  FailureOr<SmallVector<AccessGroupAttr>> convertParallelAccesses();
  FusedLoc convertStartLoc();
  FailureOr<FusedLoc> convertEndLoc();

```
- **EN**: Implements logic around `convertVectorizeAttr`, `convertInterleaveAttr`, `convertUnrollAttr`, `convertUnrollAndJamAttr`, and 8 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `convertVectorizeAttr`、`convertInterleaveAttr`、`convertUnrollAttr`、`convertUnrollAndJamAttr` 等另外 8 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 60-74
```cpp
  llvm::SmallVector<llvm::DILocation *, 2> locations;
  llvm::StringMap<const llvm::MDNode *> propertyMap;
  const llvm::MDNode *node;
  Location loc;
  LoopAnnotationImporter &loopAnnotationImporter;
  MLIRContext *ctx;
};
} // namespace

LogicalResult LoopMetadataConversion::initConversionState() {
  // Check if it's a valid node.
  if (node->getNumOperands() == 0 ||
      dyn_cast<llvm::MDNode>(node->getOperand(0)) != node)
    return emitWarning(loc) << "invalid loop node";

```
- **EN**: Implements logic around `initConversionState`, `getNumOperands`, `MDNode>`, `emitWarning`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `initConversionState`、`getNumOperands`、`MDNode>`、`emitWarning` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 75-85
```cpp
  for (const llvm::MDOperand &operand : llvm::drop_begin(node->operands())) {
    if (auto *diLoc = dyn_cast<llvm::DILocation>(operand)) {
      locations.push_back(diLoc);
      continue;
    }

    auto *property = dyn_cast<llvm::MDNode>(operand);
    if (!property)
      return emitWarning(loc) << "expected all loop properties to be either "
                                 "debug locations or metadata nodes";

```
- **EN**: Implements logic around `drop_begin`, `DILocation>`, `push_back`, `MDNode>`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `drop_begin`、`DILocation>`、`push_back`、`MDNode>` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 86-99
```cpp
    if (property->getNumOperands() == 0)
      return emitWarning(loc) << "cannot import empty loop property";

    auto *nameNode = dyn_cast<llvm::MDString>(property->getOperand(0));
    if (!nameNode)
      return emitWarning(loc) << "cannot import loop property without a name";
    StringRef name = nameNode->getString();

    bool succ = propertyMap.try_emplace(name, property).second;
    if (!succ)
      return emitWarning(loc)
             << "cannot import loop properties with duplicated names " << name;
  }

```
- **EN**: Implements logic around `getNumOperands`, `emitWarning`, `MDString>`, `getString`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `getNumOperands`、`emitWarning`、`MDString>`、`getString` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 100-112
```cpp
  return success();
}

const llvm::MDNode *
LoopMetadataConversion::lookupAndEraseProperty(StringRef name) {
  auto it = propertyMap.find(name);
  if (it == propertyMap.end())
    return nullptr;
  const llvm::MDNode *property = it->getValue();
  propertyMap.erase(it);
  return property;
}

```
- **EN**: Implements logic around `success`, `lookupAndEraseProperty`, `find`, `end`, and 2 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `success`、`lookupAndEraseProperty`、`find`、`end` 等另外 2 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 113-124
```cpp
FailureOr<BoolAttr> LoopMetadataConversion::lookupUnitNode(StringRef name) {
  const llvm::MDNode *property = lookupAndEraseProperty(name);
  if (!property)
    return BoolAttr(nullptr);

  if (property->getNumOperands() != 1)
    return emitWarning(loc)
           << "expected metadata node " << name << " to hold no value";

  return BoolAttr::get(ctx, true);
}

```
- **EN**: Implements logic around `lookupUnitNode`, `lookupAndEraseProperty`, `BoolAttr`, `getNumOperands`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupUnitNode`、`lookupAndEraseProperty`、`BoolAttr`、`getNumOperands` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 125-136
```cpp
FailureOr<BoolAttr> LoopMetadataConversion::lookupBooleanUnitNode(
    StringRef enableName, StringRef disableName, bool negated) {
  auto enable = lookupUnitNode(enableName);
  auto disable = lookupUnitNode(disableName);
  if (failed(enable) || failed(disable))
    return failure();

  if (*enable && *disable)
    return emitWarning(loc)
           << "expected metadata nodes " << enableName << " and " << disableName
           << " to be mutually exclusive.";

```
- **EN**: Implements logic around `lookupBooleanUnitNode`, `lookupUnitNode`, `failed`, `failure`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupBooleanUnitNode`、`lookupUnitNode`、`failed`、`failure` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 137-150
```cpp
  if (*enable)
    return BoolAttr::get(ctx, !negated);

  if (*disable)
    return BoolAttr::get(ctx, negated);
  return BoolAttr(nullptr);
}

FailureOr<BoolAttr> LoopMetadataConversion::lookupBoolNode(StringRef name,
                                                           bool negated) {
  const llvm::MDNode *property = lookupAndEraseProperty(name);
  if (!property)
    return BoolAttr(nullptr);

```
- **EN**: Implements logic around `get`, `BoolAttr`, `lookupBoolNode`, `lookupAndEraseProperty`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `get`、`BoolAttr`、`lookupBoolNode`、`lookupAndEraseProperty` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 151-162
```cpp
  auto emitNodeWarning = [&]() {
    return emitWarning(loc)
           << "expected metadata node " << name << " to hold a boolean value";
  };

  if (property->getNumOperands() != 2)
    return emitNodeWarning();
  llvm::ConstantInt *val =
      llvm::mdconst::dyn_extract<llvm::ConstantInt>(property->getOperand(1));
  if (!val || val->getBitWidth() != 1)
    return emitNodeWarning();

```
- **EN**: Implements logic around `emitWarning`, `getNumOperands`, `emitNodeWarning`, `ConstantInt>`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `emitWarning`、`getNumOperands`、`emitNodeWarning`、`ConstantInt>` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 163-176
```cpp
  return BoolAttr::get(ctx, val->getValue().getLimitedValue(1) ^ negated);
}

FailureOr<BoolAttr>
LoopMetadataConversion::lookupIntNodeAsBoolAttr(StringRef name) {
  const llvm::MDNode *property = lookupAndEraseProperty(name);
  if (!property)
    return BoolAttr(nullptr);

  auto emitNodeWarning = [&]() {
    return emitWarning(loc)
           << "expected metadata node " << name << " to hold an integer value";
  };

```
- **EN**: Implements logic around `get`, `lookupIntNodeAsBoolAttr`, `lookupAndEraseProperty`, `BoolAttr`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `get`、`lookupIntNodeAsBoolAttr`、`lookupAndEraseProperty`、`BoolAttr` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 177-186
```cpp
  if (property->getNumOperands() != 2)
    return emitNodeWarning();
  llvm::ConstantInt *val =
      llvm::mdconst::dyn_extract<llvm::ConstantInt>(property->getOperand(1));
  if (!val || val->getBitWidth() != 32)
    return emitNodeWarning();

  return BoolAttr::get(ctx, val->getValue().getLimitedValue(1));
}

```
- **EN**: Implements logic around `getNumOperands`, `emitNodeWarning`, `ConstantInt>`, `getBitWidth`, and 1 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `getNumOperands`、`emitNodeWarning`、`ConstantInt>`、`getBitWidth` 等另外 1 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 187-196
```cpp
FailureOr<IntegerAttr> LoopMetadataConversion::lookupIntNode(StringRef name) {
  const llvm::MDNode *property = lookupAndEraseProperty(name);
  if (!property)
    return IntegerAttr(nullptr);

  auto emitNodeWarning = [&]() {
    return emitWarning(loc)
           << "expected metadata node " << name << " to hold an i32 value";
  };

```
- **EN**: Implements logic around `lookupIntNode`, `lookupAndEraseProperty`, `IntegerAttr`, `emitWarning`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupIntNode`、`lookupAndEraseProperty`、`IntegerAttr`、`emitWarning` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 197-208
```cpp
  if (property->getNumOperands() != 2)
    return emitNodeWarning();

  llvm::ConstantInt *val =
      llvm::mdconst::dyn_extract<llvm::ConstantInt>(property->getOperand(1));
  if (!val || val->getBitWidth() != 32)
    return emitNodeWarning();

  return IntegerAttr::get(IntegerType::get(ctx, 32),
                          val->getValue().getLimitedValue());
}

```
- **EN**: Implements logic around `getNumOperands`, `emitNodeWarning`, `ConstantInt>`, `getBitWidth`, and 2 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `getNumOperands`、`emitNodeWarning`、`ConstantInt>`、`getBitWidth` 等另外 2 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 209-218
```cpp
FailureOr<llvm::MDNode *> LoopMetadataConversion::lookupMDNode(StringRef name) {
  const llvm::MDNode *property = lookupAndEraseProperty(name);
  if (!property)
    return nullptr;

  auto emitNodeWarning = [&]() {
    return emitWarning(loc)
           << "expected metadata node " << name << " to hold an MDNode";
  };

```
- **EN**: Implements logic around `lookupMDNode`, `lookupAndEraseProperty`, `emitWarning`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupMDNode`、`lookupAndEraseProperty`、`emitWarning` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 219-228
```cpp
  if (property->getNumOperands() != 2)
    return emitNodeWarning();

  auto *node = dyn_cast<llvm::MDNode>(property->getOperand(1));
  if (!node)
    return emitNodeWarning();

  return node;
}

```
- **EN**: Implements logic around `getNumOperands`, `emitNodeWarning`, `MDNode>`; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `getNumOperands`、`emitNodeWarning`、`MDNode>` 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 229-240
```cpp
FailureOr<SmallVector<llvm::MDNode *>>
LoopMetadataConversion::lookupMDNodes(StringRef name) {
  const llvm::MDNode *property = lookupAndEraseProperty(name);
  SmallVector<llvm::MDNode *> res;
  if (!property)
    return res;

  auto emitNodeWarning = [&]() {
    return emitWarning(loc) << "expected metadata node " << name
                            << " to hold one or multiple MDNodes";
  };

```
- **EN**: Implements logic around `lookupMDNodes`, `lookupAndEraseProperty`, `emitWarning`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupMDNodes`、`lookupAndEraseProperty`、`emitWarning` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 241-250
```cpp
  if (property->getNumOperands() < 2)
    return emitNodeWarning();

  for (unsigned i = 1, e = property->getNumOperands(); i < e; ++i) {
    auto *node = dyn_cast<llvm::MDNode>(property->getOperand(i));
    if (!node)
      return emitNodeWarning();
    res.push_back(node);
  }

```
- **EN**: Implements logic around `getNumOperands`, `emitNodeWarning`, `MDNode>`, `push_back`; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `getNumOperands`、`emitNodeWarning`、`MDNode>`、`push_back` 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 251-261
```cpp
  return res;
}

FailureOr<LoopAnnotationAttr>
LoopMetadataConversion::lookupFollowupNode(StringRef name) {
  auto node = lookupMDNode(name);
  if (failed(node))
    return failure();
  if (*node == nullptr)
    return LoopAnnotationAttr(nullptr);

```
- **EN**: Implements logic around `lookupFollowupNode`, `lookupMDNode`, `failed`, `failure`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupFollowupNode`、`lookupMDNode`、`failed`、`failure` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 262-271
```cpp
  return loopAnnotationImporter.translateLoopAnnotation(*node, loc);
}

static bool isEmptyOrNull(const Attribute attr) { return !attr; }

template <typename T>
static bool isEmptyOrNull(const SmallVectorImpl<T> &vec) {
  return vec.empty();
}

```
- **EN**: Implements logic around `translateLoopAnnotation`, `isEmptyOrNull`, `empty`; this block manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `translateLoopAnnotation`、`isEmptyOrNull`、`empty` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 272-283
```cpp
/// Helper function that only creates and attribute of type T if all argument
/// conversion were successfull and at least one of them holds a non-null value.
template <typename T, typename... P>
static T createIfNonNull(MLIRContext *ctx, const P &...args) {
  bool anyFailed = (failed(args) || ...);
  if (anyFailed)
    return {};

  bool allEmpty = (isEmptyOrNull(*args) && ...);
  if (allEmpty)
    return {};

```
- **EN**: Implements logic around `createIfNonNull`, `failed`, `isEmptyOrNull`; this block manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `createIfNonNull`、`failed`、`isEmptyOrNull` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 284-301
```cpp
  return T::get(ctx, *args...);
}

FailureOr<LoopVectorizeAttr> LoopMetadataConversion::convertVectorizeAttr() {
  FailureOr<BoolAttr> enable =
      lookupBoolNode("llvm.loop.vectorize.enable", true);
  FailureOr<BoolAttr> predicateEnable =
      lookupBoolNode("llvm.loop.vectorize.predicate.enable");
  FailureOr<BoolAttr> scalableEnable =
      lookupBoolNode("llvm.loop.vectorize.scalable.enable");
  FailureOr<IntegerAttr> width = lookupIntNode("llvm.loop.vectorize.width");
  FailureOr<LoopAnnotationAttr> followupVec =
      lookupFollowupNode("llvm.loop.vectorize.followup_vectorized");
  FailureOr<LoopAnnotationAttr> followupEpi =
      lookupFollowupNode("llvm.loop.vectorize.followup_epilogue");
  FailureOr<LoopAnnotationAttr> followupAll =
      lookupFollowupNode("llvm.loop.vectorize.followup_all");

```
- **EN**: Implements logic around `get`, `convertVectorizeAttr`, `lookupBoolNode`, `lookupIntNode`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `get`、`convertVectorizeAttr`、`lookupBoolNode`、`lookupIntNode` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 302-311
```cpp
  return createIfNonNull<LoopVectorizeAttr>(ctx, enable, predicateEnable,
                                            scalableEnable, width, followupVec,
                                            followupEpi, followupAll);
}

FailureOr<LoopInterleaveAttr> LoopMetadataConversion::convertInterleaveAttr() {
  FailureOr<IntegerAttr> count = lookupIntNode("llvm.loop.interleave.count");
  return createIfNonNull<LoopInterleaveAttr>(ctx, count);
}

```
- **EN**: Implements logic around `createIfNonNull`, `convertInterleaveAttr`, `lookupIntNode`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `createIfNonNull`、`convertInterleaveAttr`、`lookupIntNode` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 312-325
```cpp
FailureOr<LoopUnrollAttr> LoopMetadataConversion::convertUnrollAttr() {
  FailureOr<BoolAttr> disable = lookupBooleanUnitNode(
      "llvm.loop.unroll.enable", "llvm.loop.unroll.disable", /*negated=*/true);
  FailureOr<IntegerAttr> count = lookupIntNode("llvm.loop.unroll.count");
  FailureOr<BoolAttr> runtimeDisable =
      lookupUnitNode("llvm.loop.unroll.runtime.disable");
  FailureOr<BoolAttr> full = lookupUnitNode("llvm.loop.unroll.full");
  FailureOr<LoopAnnotationAttr> followupUnrolled =
      lookupFollowupNode("llvm.loop.unroll.followup_unrolled");
  FailureOr<LoopAnnotationAttr> followupRemainder =
      lookupFollowupNode("llvm.loop.unroll.followup_remainder");
  FailureOr<LoopAnnotationAttr> followupAll =
      lookupFollowupNode("llvm.loop.unroll.followup_all");

```
- **EN**: Implements logic around `convertUnrollAttr`, `lookupBooleanUnitNode`, `lookupIntNode`, `lookupUnitNode`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; bridges MLIR-produced code with runtime entry points; connects IR to external target or serialization formats.
- **CN**: 围绕 `convertUnrollAttr`、`lookupBooleanUnitNode`、`lookupIntNode`、`lookupUnitNode` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并在 MLIR 生成代码与运行时入口点之间建立桥接，并把 IR 连接到外部目标或序列化格式。

### Lines 326-345
```cpp
  return createIfNonNull<LoopUnrollAttr>(ctx, disable, count, runtimeDisable,
                                         full, followupUnrolled,
                                         followupRemainder, followupAll);
}

FailureOr<LoopUnrollAndJamAttr>
LoopMetadataConversion::convertUnrollAndJamAttr() {
  FailureOr<BoolAttr> disable = lookupBooleanUnitNode(
      "llvm.loop.unroll_and_jam.enable", "llvm.loop.unroll_and_jam.disable",
      /*negated=*/true);
  FailureOr<IntegerAttr> count =
      lookupIntNode("llvm.loop.unroll_and_jam.count");
  FailureOr<LoopAnnotationAttr> followupOuter =
      lookupFollowupNode("llvm.loop.unroll_and_jam.followup_outer");
  FailureOr<LoopAnnotationAttr> followupInner =
      lookupFollowupNode("llvm.loop.unroll_and_jam.followup_inner");
  FailureOr<LoopAnnotationAttr> followupRemainderOuter =
      lookupFollowupNode("llvm.loop.unroll_and_jam.followup_remainder_outer");
  FailureOr<LoopAnnotationAttr> followupRemainderInner =
      lookupFollowupNode("llvm.loop.unroll_and_jam.followup_remainder_inner");
```
- **EN**: Implements logic around `createIfNonNull`, `convertUnrollAndJamAttr`, `lookupBooleanUnitNode`, `lookupIntNode`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `createIfNonNull`、`convertUnrollAndJamAttr`、`lookupBooleanUnitNode`、`lookupIntNode` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 346-359
```cpp
  FailureOr<LoopAnnotationAttr> followupAll =
      lookupFollowupNode("llvm.loop.unroll_and_jam.followup_all");
  return createIfNonNull<LoopUnrollAndJamAttr>(
      ctx, disable, count, followupOuter, followupInner, followupRemainderOuter,
      followupRemainderInner, followupAll);
}

FailureOr<LoopLICMAttr> LoopMetadataConversion::convertLICMAttr() {
  FailureOr<BoolAttr> disable = lookupUnitNode("llvm.licm.disable");
  FailureOr<BoolAttr> versioningDisable =
      lookupUnitNode("llvm.loop.licm_versioning.disable");
  return createIfNonNull<LoopLICMAttr>(ctx, disable, versioningDisable);
}

```
- **EN**: Implements logic around `lookupFollowupNode`, `createIfNonNull`, `convertLICMAttr`, `lookupUnitNode`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats; implements transformation or simplification logic.
- **CN**: 围绕 `lookupFollowupNode`、`createIfNonNull`、`convertLICMAttr`、`lookupUnitNode` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式，并实现变换或简化逻辑。

### Lines 360-375
```cpp
FailureOr<LoopDistributeAttr> LoopMetadataConversion::convertDistributeAttr() {
  FailureOr<BoolAttr> disable =
      lookupBoolNode("llvm.loop.distribute.enable", true);
  FailureOr<LoopAnnotationAttr> followupCoincident =
      lookupFollowupNode("llvm.loop.distribute.followup_coincident");
  FailureOr<LoopAnnotationAttr> followupSequential =
      lookupFollowupNode("llvm.loop.distribute.followup_sequential");
  FailureOr<LoopAnnotationAttr> followupFallback =
      lookupFollowupNode("llvm.loop.distribute.followup_fallback");
  FailureOr<LoopAnnotationAttr> followupAll =
      lookupFollowupNode("llvm.loop.distribute.followup_all");
  return createIfNonNull<LoopDistributeAttr>(ctx, disable, followupCoincident,
                                             followupSequential,
                                             followupFallback, followupAll);
}

```
- **EN**: Implements logic around `convertDistributeAttr`, `lookupBoolNode`, `lookupFollowupNode`, `createIfNonNull`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `convertDistributeAttr`、`lookupBoolNode`、`lookupFollowupNode`、`createIfNonNull` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 376-387
```cpp
FailureOr<LoopPipelineAttr> LoopMetadataConversion::convertPipelineAttr() {
  FailureOr<BoolAttr> disable = lookupBoolNode("llvm.loop.pipeline.disable");
  FailureOr<IntegerAttr> initiationinterval =
      lookupIntNode("llvm.loop.pipeline.initiationinterval");
  return createIfNonNull<LoopPipelineAttr>(ctx, disable, initiationinterval);
}

FailureOr<LoopPeeledAttr> LoopMetadataConversion::convertPeeledAttr() {
  FailureOr<IntegerAttr> count = lookupIntNode("llvm.loop.peeled.count");
  return createIfNonNull<LoopPeeledAttr>(ctx, count);
}

```
- **EN**: Implements logic around `convertPipelineAttr`, `lookupBoolNode`, `lookupIntNode`, `createIfNonNull`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; coordinates pass execution or analysis state; connects IR to external target or serialization formats.
- **CN**: 围绕 `convertPipelineAttr`、`lookupBoolNode`、`lookupIntNode`、`createIfNonNull` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并协调 Pass 执行或分析状态，并把 IR 连接到外部目标或序列化格式。

### Lines 388-407
```cpp
FailureOr<LoopUnswitchAttr> LoopMetadataConversion::convertUnswitchAttr() {
  FailureOr<BoolAttr> partialDisable =
      lookupUnitNode("llvm.loop.unswitch.partial.disable");
  return createIfNonNull<LoopUnswitchAttr>(ctx, partialDisable);
}

FailureOr<SmallVector<AccessGroupAttr>>
LoopMetadataConversion::convertParallelAccesses() {
  FailureOr<SmallVector<llvm::MDNode *>> nodes =
      lookupMDNodes("llvm.loop.parallel_accesses");
  if (failed(nodes))
    return failure();
  SmallVector<AccessGroupAttr> refs;
  for (llvm::MDNode *node : *nodes) {
    FailureOr<SmallVector<AccessGroupAttr>> accessGroups =
        loopAnnotationImporter.lookupAccessGroupAttrs(node);
    if (failed(accessGroups)) {
      emitWarning(loc) << "could not lookup access group";
      continue;
    }
```
- **EN**: Implements logic around `convertUnswitchAttr`, `lookupUnitNode`, `createIfNonNull`, `convertParallelAccesses`, and 5 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `convertUnswitchAttr`、`lookupUnitNode`、`createIfNonNull`、`convertParallelAccesses` 等另外 5 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 408-419
```cpp
    llvm::append_range(refs, *accessGroups);
  }
  return refs;
}

FusedLoc LoopMetadataConversion::convertStartLoc() {
  if (locations.empty())
    return {};
  return dyn_cast<FusedLoc>(
      loopAnnotationImporter.moduleImport.translateLoc(locations[0]));
}

```
- **EN**: Implements logic around `append_range`, `convertStartLoc`, `empty`, `dyn_cast`, and 1 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `append_range`、`convertStartLoc`、`empty`、`dyn_cast` 等另外 1 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 420-429
```cpp
FailureOr<FusedLoc> LoopMetadataConversion::convertEndLoc() {
  if (locations.size() < 2)
    return FusedLoc();
  if (locations.size() > 2)
    return emitError(loc)
           << "expected loop metadata to have at most two DILocations";
  return dyn_cast<FusedLoc>(
      loopAnnotationImporter.moduleImport.translateLoc(locations[1]));
}

```
- **EN**: Implements logic around `convertEndLoc`, `size`, `FusedLoc`, `emitError`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `convertEndLoc`、`size`、`FusedLoc`、`emitError` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 430-449
```cpp
LoopAnnotationAttr LoopMetadataConversion::convert() {
  if (failed(initConversionState()))
    return {};

  FailureOr<BoolAttr> disableNonForced =
      lookupUnitNode("llvm.loop.disable_nonforced");
  FailureOr<LoopVectorizeAttr> vecAttr = convertVectorizeAttr();
  FailureOr<LoopInterleaveAttr> interleaveAttr = convertInterleaveAttr();
  FailureOr<LoopUnrollAttr> unrollAttr = convertUnrollAttr();
  FailureOr<LoopUnrollAndJamAttr> unrollAndJamAttr = convertUnrollAndJamAttr();
  FailureOr<LoopLICMAttr> licmAttr = convertLICMAttr();
  FailureOr<LoopDistributeAttr> distributeAttr = convertDistributeAttr();
  FailureOr<LoopPipelineAttr> pipelineAttr = convertPipelineAttr();
  FailureOr<LoopPeeledAttr> peeledAttr = convertPeeledAttr();
  FailureOr<LoopUnswitchAttr> unswitchAttr = convertUnswitchAttr();
  FailureOr<BoolAttr> mustProgress = lookupUnitNode("llvm.loop.mustprogress");
  FailureOr<BoolAttr> isVectorized =
      lookupIntNodeAsBoolAttr("llvm.loop.isvectorized");
  FailureOr<SmallVector<AccessGroupAttr>> parallelAccesses =
      convertParallelAccesses();
```
- **EN**: Implements logic around `convert`, `failed`, `lookupUnitNode`, `convertVectorizeAttr`, and 10 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `convert`、`failed`、`lookupUnitNode`、`convertVectorizeAttr` 等另外 10 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 450-460
```cpp

  // Drop the metadata if there are parts that cannot be imported.
  if (!propertyMap.empty()) {
    for (auto name : propertyMap.keys())
      emitWarning(loc) << "unknown loop annotation " << name;
    return {};
  }

  FailureOr<FusedLoc> startLoc = convertStartLoc();
  FailureOr<FusedLoc> endLoc = convertEndLoc();

```
- **EN**: Implements logic around `empty`, `keys`, `emitWarning`, `convertStartLoc`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `empty`、`keys`、`emitWarning`、`convertStartLoc` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 461-473
```cpp
  return createIfNonNull<LoopAnnotationAttr>(
      ctx, disableNonForced, vecAttr, interleaveAttr, unrollAttr,
      unrollAndJamAttr, licmAttr, distributeAttr, pipelineAttr, peeledAttr,
      unswitchAttr, mustProgress, isVectorized, startLoc, endLoc,
      parallelAccesses);
}

LoopAnnotationAttr
LoopAnnotationImporter::translateLoopAnnotation(const llvm::MDNode *node,
                                                Location loc) {
  if (!node)
    return {};

```
- **EN**: Implements logic around `createIfNonNull`, `translateLoopAnnotation`; this block manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `createIfNonNull`、`translateLoopAnnotation` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 474-485
```cpp
  // Note: This check is necessary to distinguish between failed translations
  // and not yet attempted translations.
  auto it = loopMetadataMapping.find(node);
  if (it != loopMetadataMapping.end())
    return it->getSecond();

  LoopAnnotationAttr attr = LoopMetadataConversion(node, loc, *this).convert();

  mapLoopMetadata(node, attr);
  return attr;
}

```
- **EN**: Implements logic around `find`, `end`, `getSecond`, `LoopMetadataConversion`, and 1 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `find`、`end`、`getSecond`、`LoopMetadataConversion` 等另外 1 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 486-498
```cpp
LogicalResult
LoopAnnotationImporter::translateAccessGroup(const llvm::MDNode *node,
                                             Location loc) {
  SmallVector<const llvm::MDNode *> accessGroups;
  if (!node->getNumOperands())
    accessGroups.push_back(node);
  for (const llvm::MDOperand &operand : node->operands()) {
    auto *childNode = dyn_cast<llvm::MDNode>(operand);
    if (!childNode)
      return failure();
    accessGroups.push_back(cast<llvm::MDNode>(operand.get()));
  }

```
- **EN**: Implements logic around `translateAccessGroup`, `getNumOperands`, `push_back`, `operands`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `translateAccessGroup`、`getNumOperands`、`push_back`、`operands` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 499-513
```cpp
  // Convert all entries of the access group list to access group operations.
  for (const llvm::MDNode *accessGroup : accessGroups) {
    if (accessGroupMapping.count(accessGroup))
      continue;
    // Verify the access group node is distinct and empty.
    if (accessGroup->getNumOperands() != 0 || !accessGroup->isDistinct())
      return emitWarning(loc)
             << "expected an access group node to be empty and distinct";

    // Add a mapping from the access group node to the newly created attribute.
    accessGroupMapping[accessGroup] = builder.getAttr<AccessGroupAttr>();
  }
  return success();
}

```
- **EN**: Implements logic around `count`, `getNumOperands`, `emitWarning`, `getAttr`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `count`、`getNumOperands`、`emitWarning`、`getAttr` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 514-529
```cpp
FailureOr<SmallVector<AccessGroupAttr>>
LoopAnnotationImporter::lookupAccessGroupAttrs(const llvm::MDNode *node) const {
  // An access group node is either a single access group or an access group
  // list.
  SmallVector<AccessGroupAttr> accessGroups;
  if (!node->getNumOperands())
    accessGroups.push_back(accessGroupMapping.lookup(node));
  for (const llvm::MDOperand &operand : node->operands()) {
    auto *node = cast<llvm::MDNode>(operand.get());
    accessGroups.push_back(accessGroupMapping.lookup(node));
  }
  // Exit if one of the access group node lookups failed.
  if (llvm::is_contained(accessGroups, nullptr))
    return failure();
  return accessGroups;
}
```
- **EN**: Implements logic around `lookupAccessGroupAttrs`, `getNumOperands`, `push_back`, `operands`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `lookupAccessGroupAttrs`、`getNumOperands`、`push_back`、`operands` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

## Key Concepts / 关键概念

- **Target translation / 目标翻译**:
  - **EN**: Connects MLIR programs to external target formats such as LLVM IR or runtime ABIs.
  - **CN**: 将 MLIR 程序连接到 LLVM IR 或运行时 ABI 等外部目标格式。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Pass pipelines / Pass 流水线**:
  - **EN**: The file participates in pass registration, scheduling, or execution orchestration.
  - **CN**: 该文件参与 Pass 的注册、调度或执行编排。
- **Runtime interop / 运行时互操作**:
  - **EN**: The code exposes or consumes ABI-level hooks used by generated code at runtime.
  - **CN**: 代码暴露或消费运行时由生成代码使用的 ABI 级钩子。
- **Target or format bridging / 目标或格式桥接**:
  - **EN**: IR is translated to, from, or alongside external target representations.
  - **CN**: 该文件把 IR 翻译到外部目标表示、从外部目标表示恢复，或与之协同工作。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `LoopAnnotationImporter.h`, `llvm/IR/Constants.h`
- **Subsystem categories / 子系统类别**: LLVM IR support APIs / LLVM IR 支持 API (1)
