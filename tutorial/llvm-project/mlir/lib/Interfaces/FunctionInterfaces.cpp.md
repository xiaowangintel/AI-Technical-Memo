# FunctionInterfaces.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Interfaces/FunctionInterfaces.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR interfaces or generated interface adaptation glue.
  - **CN**: 实现 MLIR 接口或生成的接口适配胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===- FunctionSupport.cpp - Utility types for function-like ops ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/Interfaces/FunctionInterfaces.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Interfaces/FunctionInterfaces.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Interfaces/FunctionInterfaces.h`。

### Lines 11-21
```cpp
using namespace mlir;

//===----------------------------------------------------------------------===//
// Tablegen Interface Definitions
//===----------------------------------------------------------------------===//

#include "mlir/Interfaces/FunctionInterfaces.cpp.inc"

//===----------------------------------------------------------------------===//
// Function Arguments and Results.
//===----------------------------------------------------------------------===//
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Interfaces/FunctionInterfaces.cpp.inc`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Interfaces/FunctionInterfaces.cpp.inc`。

### Lines 22-34
```cpp

static bool isEmptyAttrDict(Attribute attr) {
  return llvm::cast<DictionaryAttr>(attr).empty();
}

DictionaryAttr function_interface_impl::getArgAttrDict(FunctionOpInterface op,
                                                       unsigned index) {
  ArrayAttr attrs = op.getArgAttrsAttr();
  DictionaryAttr argAttrs =
      attrs ? llvm::cast<DictionaryAttr>(attrs[index]) : DictionaryAttr();
  return argAttrs;
}

```
- **EN**: Implements logic around `isEmptyAttrDict`, `cast`, `getArgAttrDict`, `getArgAttrsAttr`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `isEmptyAttrDict`、`cast`、`getArgAttrDict`、`getArgAttrsAttr` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 35-49
```cpp
DictionaryAttr
function_interface_impl::getResultAttrDict(FunctionOpInterface op,
                                           unsigned index) {
  ArrayAttr attrs = op.getResAttrsAttr();
  DictionaryAttr resAttrs =
      attrs ? llvm::cast<DictionaryAttr>(attrs[index]) : DictionaryAttr();
  return resAttrs;
}

ArrayRef<NamedAttribute>
function_interface_impl::getArgAttrs(FunctionOpInterface op, unsigned index) {
  auto argDict = getArgAttrDict(op, index);
  return argDict ? argDict.getValue() : ArrayRef<NamedAttribute>();
}

```
- **EN**: Implements logic around `getResultAttrDict`, `getResAttrsAttr`, `cast`, `getArgAttrs`, and 2 more symbols.
- **CN**: 围绕 `getResultAttrDict`、`getResAttrsAttr`、`cast`、`getArgAttrs` 等另外 2 个符号 实现具体逻辑。

### Lines 50-65
```cpp
ArrayRef<NamedAttribute>
function_interface_impl::getResultAttrs(FunctionOpInterface op,
                                        unsigned index) {
  auto resultDict = getResultAttrDict(op, index);
  return resultDict ? resultDict.getValue() : ArrayRef<NamedAttribute>();
}

/// Get either the argument or result attributes array.
template <bool isArg>
static ArrayAttr getArgResAttrs(FunctionOpInterface op) {
  if constexpr (isArg)
    return op.getArgAttrsAttr();
  else
    return op.getResAttrsAttr();
}

```
- **EN**: Implements logic around `getResultAttrs`, `getResultAttrDict`, `getValue`, `getArgResAttrs`, and 3 more symbols.
- **CN**: 围绕 `getResultAttrs`、`getResultAttrDict`、`getValue`、`getArgResAttrs` 等另外 3 个符号 实现具体逻辑。

### Lines 66-75
```cpp
/// Set either the argument or result attributes array.
template <bool isArg>
static void setArgResAttrs(FunctionOpInterface op, ArrayAttr attrs) {
  if constexpr (isArg)
    op.setArgAttrsAttr(attrs);
  else
    op.setResAttrsAttr(attrs);
}

/// Erase either the argument or result attributes array.
```
- **EN**: Implements logic around `setArgResAttrs`, `constexpr`, `setArgAttrsAttr`, `setResAttrsAttr`.
- **CN**: 围绕 `setArgResAttrs`、`constexpr`、`setArgAttrsAttr`、`setResAttrsAttr` 实现具体逻辑。

### Lines 76-93
```cpp
template <bool isArg>
static void removeArgResAttrs(FunctionOpInterface op) {
  if constexpr (isArg)
    op.removeArgAttrsAttr();
  else
    op.removeResAttrsAttr();
}

/// Set all of the argument or result attribute dictionaries for a function.
template <bool isArg>
static void setAllArgResAttrDicts(FunctionOpInterface op,
                                  ArrayRef<Attribute> attrs) {
  if (llvm::all_of(attrs, isEmptyAttrDict))
    removeArgResAttrs<isArg>(op);
  else
    setArgResAttrs<isArg>(op, ArrayAttr::get(op->getContext(), attrs));
}

```
- **EN**: Implements logic around `removeArgResAttrs`, `constexpr`, `removeArgAttrsAttr`, `removeResAttrsAttr`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `removeArgResAttrs`、`constexpr`、`removeArgAttrsAttr`、`removeResAttrsAttr` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 94-106
```cpp
void function_interface_impl::setAllArgAttrDicts(
    FunctionOpInterface op, ArrayRef<DictionaryAttr> attrs) {
  setAllArgAttrDicts(op, ArrayRef<Attribute>(attrs.data(), attrs.size()));
}

void function_interface_impl::setAllArgAttrDicts(FunctionOpInterface op,
                                                 ArrayRef<Attribute> attrs) {
  auto wrappedAttrs = llvm::map_range(attrs, [op](Attribute attr) -> Attribute {
    return !attr ? DictionaryAttr::get(op->getContext()) : attr;
  });
  setAllArgResAttrDicts</*isArg=*/true>(op, llvm::to_vector<8>(wrappedAttrs));
}

```
- **EN**: Implements logic around `setAllArgAttrDicts`, `map_range`, `get`, `true>`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `setAllArgAttrDicts`、`map_range`、`get`、`true>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 107-119
```cpp
void function_interface_impl::setAllResultAttrDicts(
    FunctionOpInterface op, ArrayRef<DictionaryAttr> attrs) {
  setAllResultAttrDicts(op, ArrayRef<Attribute>(attrs.data(), attrs.size()));
}

void function_interface_impl::setAllResultAttrDicts(FunctionOpInterface op,
                                                    ArrayRef<Attribute> attrs) {
  auto wrappedAttrs = llvm::map_range(attrs, [op](Attribute attr) -> Attribute {
    return !attr ? DictionaryAttr::get(op->getContext()) : attr;
  });
  setAllArgResAttrDicts</*isArg=*/false>(op, llvm::to_vector<8>(wrappedAttrs));
}

```
- **EN**: Implements logic around `setAllResultAttrDicts`, `map_range`, `get`, `false>`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `setAllResultAttrDicts`、`map_range`、`get`、`false>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 120-139
```cpp
/// Update the given index into an argument or result attribute dictionary.
template <bool isArg>
static void setArgResAttrDict(FunctionOpInterface op, unsigned numTotalIndices,
                              unsigned index, DictionaryAttr attrs) {
  ArrayAttr allAttrs = getArgResAttrs<isArg>(op);
  if (!allAttrs) {
    if (attrs.empty())
      return;

    // If this attribute is not empty, we need to create a new attribute array.
    SmallVector<Attribute, 8> newAttrs(numTotalIndices,
                                       DictionaryAttr::get(op->getContext()));
    newAttrs[index] = attrs;
    setArgResAttrs<isArg>(op, ArrayAttr::get(op->getContext(), newAttrs));
    return;
  }
  // Check to see if the attribute is different from what we already have.
  if (allAttrs[index] == attrs)
    return;

```
- **EN**: Implements logic around `setArgResAttrDict`, `getArgResAttrs`, `empty`, `newAttrs`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `setArgResAttrDict`、`getArgResAttrs`、`empty`、`newAttrs` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 140-153
```cpp
  // If it is, check to see if the attribute array would now contain only empty
  // dictionaries.
  ArrayRef<Attribute> rawAttrArray = allAttrs.getValue();
  if (attrs.empty() &&
      llvm::all_of(rawAttrArray.take_front(index), isEmptyAttrDict) &&
      llvm::all_of(rawAttrArray.drop_front(index + 1), isEmptyAttrDict))
    return removeArgResAttrs<isArg>(op);

  // Otherwise, create a new attribute array with the updated dictionary.
  SmallVector<Attribute, 8> newAttrs(rawAttrArray);
  newAttrs[index] = attrs;
  setArgResAttrs<isArg>(op, ArrayAttr::get(op->getContext(), newAttrs));
}

```
- **EN**: Implements logic around `getValue`, `empty`, `all_of`, `removeArgResAttrs`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getValue`、`empty`、`all_of`、`removeArgResAttrs` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 154-170
```cpp
void function_interface_impl::setArgAttrs(FunctionOpInterface op,
                                          unsigned index,
                                          ArrayRef<NamedAttribute> attributes) {
  assert(index < op.getNumArguments() && "invalid argument number");
  return setArgResAttrDict</*isArg=*/true>(
      op, op.getNumArguments(), index,
      DictionaryAttr::get(op->getContext(), attributes));
}

void function_interface_impl::setArgAttrs(FunctionOpInterface op,
                                          unsigned index,
                                          DictionaryAttr attributes) {
  return setArgResAttrDict</*isArg=*/true>(
      op, op.getNumArguments(), index,
      attributes ? attributes : DictionaryAttr::get(op->getContext()));
}

```
- **EN**: Implements logic around `setArgAttrs`, `assert`, `true>`, `getNumArguments`, and 1 more symbols.
- **CN**: 围绕 `setArgAttrs`、`assert`、`true>`、`getNumArguments` 等另外 1 个符号 实现具体逻辑。

### Lines 171-188
```cpp
void function_interface_impl::setResultAttrs(
    FunctionOpInterface op, unsigned index,
    ArrayRef<NamedAttribute> attributes) {
  assert(index < op.getNumResults() && "invalid result number");
  return setArgResAttrDict</*isArg=*/false>(
      op, op.getNumResults(), index,
      DictionaryAttr::get(op->getContext(), attributes));
}

void function_interface_impl::setResultAttrs(FunctionOpInterface op,
                                             unsigned index,
                                             DictionaryAttr attributes) {
  assert(index < op.getNumResults() && "invalid result number");
  return setArgResAttrDict</*isArg=*/false>(
      op, op.getNumResults(), index,
      attributes ? attributes : DictionaryAttr::get(op->getContext()));
}

```
- **EN**: Implements logic around `setResultAttrs`, `assert`, `false>`, `getNumResults`, and 1 more symbols.
- **CN**: 围绕 `setResultAttrs`、`assert`、`false>`、`getNumResults` 等另外 1 个符号 实现具体逻辑。

### Lines 189-198
```cpp
void function_interface_impl::insertFunctionArguments(
    FunctionOpInterface op, ArrayRef<unsigned> argIndices, TypeRange argTypes,
    ArrayRef<DictionaryAttr> argAttrs, ArrayRef<Location> argLocs,
    unsigned originalNumArgs, Type newType) {
  assert(argIndices.size() == argTypes.size());
  assert(argIndices.size() == argAttrs.size() || argAttrs.empty());
  assert(argIndices.size() == argLocs.size());
  if (argIndices.empty())
    return;

```
- **EN**: Implements logic around `insertFunctionArguments`, `assert`, `empty`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `insertFunctionArguments`、`assert`、`empty` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 199-218
```cpp
  // There are 3 things that need to be updated:
  // - Function type.
  // - Arg attrs.
  // - Block arguments of entry block, if not empty.

  // Update the argument attributes of the function.
  ArrayAttr oldArgAttrs = op.getArgAttrsAttr();
  if (oldArgAttrs || !argAttrs.empty()) {
    SmallVector<DictionaryAttr, 4> newArgAttrs;
    newArgAttrs.reserve(originalNumArgs + argIndices.size());
    unsigned oldIdx = 0;
    auto migrate = [&](unsigned untilIdx) {
      if (!oldArgAttrs) {
        newArgAttrs.resize(newArgAttrs.size() + untilIdx - oldIdx);
      } else {
        auto oldArgAttrRange = oldArgAttrs.getAsRange<DictionaryAttr>();
        newArgAttrs.append(oldArgAttrRange.begin() + oldIdx,
                           oldArgAttrRange.begin() + untilIdx);
      }
      oldIdx = untilIdx;
```
- **EN**: Implements logic around `getArgAttrsAttr`, `empty`, `reserve`, `resize`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getArgAttrsAttr`、`empty`、`reserve`、`resize` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 219-230
```cpp
    };
    for (unsigned i = 0, e = argIndices.size(); i < e; ++i) {
      migrate(argIndices[i]);
      newArgAttrs.push_back(argAttrs.empty() ? DictionaryAttr{} : argAttrs[i]);
    }
    migrate(originalNumArgs);
    setAllArgAttrDicts(op, newArgAttrs);
  }

  // Update the function type.
  op.setFunctionTypeAttr(TypeAttr::get(newType));

```
- **EN**: Implements logic around `size`, `migrate`, `push_back`, `setAllArgAttrDicts`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `size`、`migrate`、`push_back`、`setAllArgAttrDicts` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 231-247
```cpp
  // Update entry block arguments, if not empty.
  if (!op.isExternal()) {
    Block &entry = op->getRegion(0).front();
    for (unsigned i = 0, e = argIndices.size(); i < e; ++i)
      entry.insertArgument(argIndices[i] + i, argTypes[i], argLocs[i]);
  }
}

void function_interface_impl::insertFunctionResults(
    FunctionOpInterface op, ArrayRef<unsigned> resultIndices,
    TypeRange resultTypes, ArrayRef<DictionaryAttr> resultAttrs,
    unsigned originalNumResults, Type newType) {
  assert(resultIndices.size() == resultTypes.size());
  assert(resultIndices.size() == resultAttrs.size() || resultAttrs.empty());
  if (resultIndices.empty())
    return;

```
- **EN**: Implements logic around `isExternal`, `getRegion`, `size`, `insertArgument`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `isExternal`、`getRegion`、`size`、`insertArgument` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 248-267
```cpp
  // There are 2 things that need to be updated:
  // - Function type.
  // - Result attrs.

  // Update the result attributes of the function.
  ArrayAttr oldResultAttrs = op.getResAttrsAttr();
  if (oldResultAttrs || !resultAttrs.empty()) {
    SmallVector<DictionaryAttr, 4> newResultAttrs;
    newResultAttrs.reserve(originalNumResults + resultIndices.size());
    unsigned oldIdx = 0;
    auto migrate = [&](unsigned untilIdx) {
      if (!oldResultAttrs) {
        newResultAttrs.resize(newResultAttrs.size() + untilIdx - oldIdx);
      } else {
        auto oldResultAttrsRange = oldResultAttrs.getAsRange<DictionaryAttr>();
        newResultAttrs.append(oldResultAttrsRange.begin() + oldIdx,
                              oldResultAttrsRange.begin() + untilIdx);
      }
      oldIdx = untilIdx;
    };
```
- **EN**: Implements logic around `getResAttrsAttr`, `empty`, `reserve`, `resize`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getResAttrsAttr`、`empty`、`reserve`、`resize` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 268-280
```cpp
    for (unsigned i = 0, e = resultIndices.size(); i < e; ++i) {
      migrate(resultIndices[i]);
      newResultAttrs.push_back(resultAttrs.empty() ? DictionaryAttr{}
                                                   : resultAttrs[i]);
    }
    migrate(originalNumResults);
    setAllResultAttrDicts(op, newResultAttrs);
  }

  // Update the function type.
  op.setFunctionTypeAttr(TypeAttr::get(newType));
}

```
- **EN**: Implements logic around `size`, `migrate`, `push_back`, `setAllResultAttrDicts`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `size`、`migrate`、`push_back`、`setAllResultAttrDicts` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 281-297
```cpp
void function_interface_impl::eraseFunctionArguments(
    FunctionOpInterface op, const BitVector &argIndices, Type newType) {
  // There are 3 things that need to be updated:
  // - Function type.
  // - Arg attrs.
  // - Block arguments of entry block, if not empty.

  // Update the argument attributes of the function.
  if (ArrayAttr argAttrs = op.getArgAttrsAttr()) {
    SmallVector<DictionaryAttr, 4> newArgAttrs;
    newArgAttrs.reserve(argAttrs.size());
    for (unsigned i = 0, e = argIndices.size(); i < e; ++i)
      if (!argIndices[i])
        newArgAttrs.emplace_back(llvm::cast<DictionaryAttr>(argAttrs[i]));
    setAllArgAttrDicts(op, newArgAttrs);
  }

```
- **EN**: Implements logic around `eraseFunctionArguments`, `getArgAttrsAttr`, `reserve`, `size`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `eraseFunctionArguments`、`getArgAttrsAttr`、`reserve`、`size` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 298-307
```cpp
  // Update the function type.
  op.setFunctionTypeAttr(TypeAttr::get(newType));

  // Update entry block arguments, if not empty.
  if (!op.isExternal()) {
    Block &entry = op->getRegion(0).front();
    entry.eraseArguments(argIndices);
  }
}

```
- **EN**: Implements logic around `setFunctionTypeAttr`, `isExternal`, `getRegion`, `eraseArguments`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `setFunctionTypeAttr`、`isExternal`、`getRegion`、`eraseArguments` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 308-323
```cpp
void function_interface_impl::eraseFunctionResults(
    FunctionOpInterface op, const BitVector &resultIndices, Type newType) {
  // There are 2 things that need to be updated:
  // - Function type.
  // - Result attrs.

  // Update the result attributes of the function.
  if (ArrayAttr resAttrs = op.getResAttrsAttr()) {
    SmallVector<DictionaryAttr, 4> newResultAttrs;
    newResultAttrs.reserve(resAttrs.size());
    for (unsigned i = 0, e = resultIndices.size(); i < e; ++i)
      if (!resultIndices[i])
        newResultAttrs.emplace_back(llvm::cast<DictionaryAttr>(resAttrs[i]));
    setAllResultAttrDicts(op, newResultAttrs);
  }

```
- **EN**: Implements logic around `eraseFunctionResults`, `getResAttrsAttr`, `reserve`, `size`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `eraseFunctionResults`、`getResAttrsAttr`、`reserve`、`size` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 324-339
```cpp
  // Update the function type.
  op.setFunctionTypeAttr(TypeAttr::get(newType));
}

//===----------------------------------------------------------------------===//
// Function type signature.
//===----------------------------------------------------------------------===//

void function_interface_impl::setFunctionType(FunctionOpInterface op,
                                              Type newType) {
  unsigned oldNumArgs = op.getNumArguments();
  unsigned oldNumResults = op.getNumResults();
  op.setFunctionTypeAttr(TypeAttr::get(newType));
  unsigned newNumArgs = op.getNumArguments();
  unsigned newNumResults = op.getNumResults();

```
- **EN**: Implements logic around `setFunctionTypeAttr`, `setFunctionType`, `getNumArguments`, `getNumResults`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `setFunctionTypeAttr`、`setFunctionType`、`getNumArguments`、`getNumResults` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 340-353
```cpp
  // Functor used to update the argument and result attributes of the function.
  auto emptyDict = DictionaryAttr::get(op.getContext());
  auto updateAttrFn = [&](auto isArg, unsigned oldCount, unsigned newCount) {
    constexpr bool isArgVal = std::is_same_v<decltype(isArg), std::true_type>;

    if (oldCount == newCount)
      return;
    // The new type has no arguments/results, just drop the attribute.
    if (newCount == 0)
      return removeArgResAttrs<isArgVal>(op);
    ArrayAttr attrs = getArgResAttrs<isArgVal>(op);
    if (!attrs)
      return;

```
- **EN**: Implements logic around `get`, `is_same_v`, `removeArgResAttrs`, `getArgResAttrs`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `get`、`is_same_v`、`removeArgResAttrs`、`getArgResAttrs` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 354-365
```cpp
    // The new type has less arguments/results, take the first N attributes.
    if (newCount < oldCount)
      return setAllArgResAttrDicts<isArgVal>(
          op, attrs.getValue().take_front(newCount));

    // Otherwise, the new type has more arguments/results. Initialize the new
    // arguments/results with empty dictionary attributes.
    SmallVector<Attribute> newAttrs(attrs.begin(), attrs.end());
    newAttrs.resize(newCount, emptyDict);
    setAllArgResAttrDicts<isArgVal>(op, newAttrs);
  };

```
- **EN**: Implements logic around `setAllArgResAttrDicts`, `getValue`, `newAttrs`, `resize`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `setAllArgResAttrDicts`、`getValue`、`newAttrs`、`resize` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 366-369
```cpp
  // Update the argument and result attributes.
  updateAttrFn(std::true_type{}, oldNumArgs, newNumArgs);
  updateAttrFn(std::false_type{}, oldNumResults, newNumResults);
}
```
- **EN**: Implements logic around `updateAttrFn`.
- **CN**: 围绕 `updateAttrFn` 实现具体逻辑。

## Key Concepts / 关键概念

- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Uses MLIR interfaces to describe cross-cutting behavior independently from concrete operation classes.
  - **CN**: 使用 MLIR 接口在不依赖具体操作类的前提下描述横切行为。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **TableGen-driven generation / TableGen 驱动生成**:
  - **EN**: Declarative records are converted into generated MLIR declarations or implementation fragments.
  - **CN**: 把声明式记录转换为生成的 MLIR 声明或实现片段。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Interfaces/FunctionInterfaces.h`, `mlir/Interfaces/FunctionInterfaces.cpp.inc`
- **Subsystem categories / 子系统类别**: MLIR interface declarations / MLIR 接口声明 (2)
