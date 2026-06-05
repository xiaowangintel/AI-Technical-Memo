# AttrTypeSubElements.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/IR/AttrTypeSubElements.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLIR core IR objects, printing/parsing support, and builtin type or attribute behavior.
  - **CN**: 实现 MLIR 核心 IR 对象、打印/解析支持以及内建类型或属性行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- AttrTypeSubElements.cpp - Attr and Type SubElement Interfaces ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "mlir/IR/Operation.h"
#include <optional>

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/IR/Operation.h`, `optional`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/IR/Operation.h`, `optional`。

### Lines 12-24
```cpp
using namespace mlir;

//===----------------------------------------------------------------------===//
// AttrTypeWalker
//===----------------------------------------------------------------------===//

WalkResult AttrTypeWalker::walkImpl(Attribute attr, WalkOrder order) {
  return walkImpl(attr, attrWalkFns, order);
}
WalkResult AttrTypeWalker::walkImpl(Type type, WalkOrder order) {
  return walkImpl(type, typeWalkFns, order);
}

```
- **EN**: Implements logic around `walkImpl`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `walkImpl` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 25-34
```cpp
template <typename T, typename WalkFns>
WalkResult AttrTypeWalker::walkImpl(T element, WalkFns &walkFns,
                                    WalkOrder order) {
  // Check if we've already walk this element before.
  auto key = std::make_pair(element.getAsOpaquePointer(), (int)order);
  auto [it, inserted] =
      visitedAttrTypes.try_emplace(key, WalkResult::advance());
  if (!inserted)
    return it->second;

```
- **EN**: Implements logic around `walkImpl`, `make_pair`, `try_emplace`.
- **CN**: 围绕 `walkImpl`、`make_pair`、`try_emplace` 实现具体逻辑。

### Lines 35-49
```cpp
  // If we are walking in post order, walk the sub elements first.
  if (order == WalkOrder::PostOrder) {
    if (walkSubElements(element, order).wasInterrupted())
      return visitedAttrTypes[key] = WalkResult::interrupt();
  }

  // Walk this element, bailing if skipped or interrupted.
  for (auto &walkFn : llvm::reverse(walkFns)) {
    WalkResult walkResult = walkFn(element);
    if (walkResult.wasInterrupted())
      return visitedAttrTypes[key] = WalkResult::interrupt();
    if (walkResult.wasSkipped())
      return WalkResult::advance();
  }

```
- **EN**: Implements logic around `walkSubElements`, `interrupt`, `reverse`, `walkFn`, and 3 more symbols.
- **CN**: 围绕 `walkSubElements`、`interrupt`、`reverse`、`walkFn` 等另外 3 个符号 实现具体逻辑。

### Lines 50-68
```cpp
  // If we are walking in pre-order, walk the sub elements last.
  if (order == WalkOrder::PreOrder) {
    if (walkSubElements(element, order).wasInterrupted())
      return WalkResult::interrupt();
  }
  return WalkResult::advance();
}

template <typename T>
WalkResult AttrTypeWalker::walkSubElements(T interface, WalkOrder order) {
  WalkResult result = WalkResult::advance();
  auto walkFn = [&](auto element) {
    if (element && !result.wasInterrupted())
      result = walkImpl(element, order);
  };
  interface.walkImmediateSubElements(walkFn, walkFn);
  return result.wasInterrupted() ? result : WalkResult::advance();
}

```
- **EN**: Implements logic around `walkSubElements`, `interrupt`, `advance`, `wasInterrupted`, and 2 more symbols.
- **CN**: 围绕 `walkSubElements`、`interrupt`、`advance`、`wasInterrupted` 等另外 2 个符号 实现具体逻辑。

### Lines 69-78
```cpp
//===----------------------------------------------------------------------===//
/// AttrTypeReplacerBase
//===----------------------------------------------------------------------===//

template <typename Concrete>
void detail::AttrTypeReplacerBase<Concrete>::addReplacement(
    ReplaceFn<Attribute> fn) {
  attrReplacementFns.emplace_back(std::move(fn));
}

```
- **EN**: Implements logic around `addReplacement`, `emplace_back`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `addReplacement`、`emplace_back` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 79-94
```cpp
template <typename Concrete>
void detail::AttrTypeReplacerBase<Concrete>::addReplacement(
    ReplaceFn<Type> fn) {
  typeReplacementFns.push_back(std::move(fn));
}

template <typename Concrete>
void detail::AttrTypeReplacerBase<Concrete>::replaceElementsIn(
    Operation *op, bool replaceAttrs, bool replaceLocs, bool replaceTypes) {
  // Functor that replaces the given element if the new value is different,
  // otherwise returns nullptr.
  auto replaceIfDifferent = [&](auto element) {
    auto replacement = static_cast<Concrete *>(this)->replace(element);
    return (replacement && replacement != element) ? replacement : nullptr;
  };

```
- **EN**: Implements logic around `addReplacement`, `push_back`, `replaceElementsIn`, `replace`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `addReplacement`、`push_back`、`replaceElementsIn`、`replace` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 95-104
```cpp
  // Update the attribute dictionary.
  if (replaceAttrs) {
    if (auto newAttrs = replaceIfDifferent(op->getAttrDictionary()))
      op->setAttrs(cast<DictionaryAttr>(newAttrs));
  }

  // If we aren't updating locations or types, we're done.
  if (!replaceTypes && !replaceLocs)
    return;

```
- **EN**: Implements logic around `replaceIfDifferent`, `setAttrs`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replaceIfDifferent`、`setAttrs` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 105-117
```cpp
  // Update the location.
  if (replaceLocs) {
    if (Attribute newLoc = replaceIfDifferent(op->getLoc()))
      op->setLoc(cast<LocationAttr>(newLoc));
  }

  // Update the result types.
  if (replaceTypes) {
    for (OpResult result : op->getResults())
      if (Type newType = replaceIfDifferent(result.getType()))
        result.setType(newType);
  }

```
- **EN**: Implements logic around `replaceIfDifferent`, `setLoc`, `getResults`, `setType`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replaceIfDifferent`、`setLoc`、`getResults`、`setType` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 118-135
```cpp
  // Update any nested block arguments.
  for (Region &region : op->getRegions()) {
    for (Block &block : region) {
      for (BlockArgument &arg : block.getArguments()) {
        if (replaceLocs) {
          if (Attribute newLoc = replaceIfDifferent(arg.getLoc()))
            arg.setLoc(cast<LocationAttr>(newLoc));
        }

        if (replaceTypes) {
          if (Type newType = replaceIfDifferent(arg.getType()))
            arg.setType(newType);
        }
      }
    }
  }
}

```
- **EN**: Implements logic around `getRegions`, `getArguments`, `replaceIfDifferent`, `setLoc`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `getRegions`、`getArguments`、`replaceIfDifferent`、`setLoc` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 136-151
```cpp
template <typename Concrete>
void detail::AttrTypeReplacerBase<Concrete>::recursivelyReplaceElementsIn(
    Operation *op, bool replaceAttrs, bool replaceLocs, bool replaceTypes) {
  op->walk([&](Operation *nestedOp) {
    replaceElementsIn(nestedOp, replaceAttrs, replaceLocs, replaceTypes);
  });
}

template <typename T, typename Replacer>
static void updateSubElementImpl(T element, Replacer &replacer,
                                 SmallVectorImpl<T> &newElements,
                                 FailureOr<bool> &changed) {
  // Bail early if we failed at any point.
  if (failed(changed))
    return;

```
- **EN**: Implements logic around `recursivelyReplaceElementsIn`, `walk`, `replaceElementsIn`, `updateSubElementImpl`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `recursivelyReplaceElementsIn`、`walk`、`replaceElementsIn`、`updateSubElementImpl` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 152-167
```cpp
  // Guard against potentially null inputs. We always map null to null.
  if (!element) {
    newElements.push_back(nullptr);
    return;
  }

  // Replace the element.
  if (T result = replacer.replace(element)) {
    newElements.push_back(result);
    if (result != element)
      changed = true;
  } else {
    changed = failure();
  }
}

```
- **EN**: Implements logic around `push_back`, `replace`, `failure`.
- **CN**: 围绕 `push_back`、`replace`、`failure` 实现具体逻辑。

### Lines 168-183
```cpp
template <typename T, typename Replacer>
static T replaceSubElements(T interface, Replacer &replacer) {
  // Walk the current sub-elements, replacing them as necessary.
  SmallVector<Attribute, 16> newAttrs;
  SmallVector<Type, 16> newTypes;
  FailureOr<bool> changed = false;
  interface.walkImmediateSubElements(
      [&](Attribute element) {
        updateSubElementImpl(element, replacer, newAttrs, changed);
      },
      [&](Type element) {
        updateSubElementImpl(element, replacer, newTypes, changed);
      });
  if (failed(changed))
    return nullptr;

```
- **EN**: Implements logic around `replaceSubElements`, `walkImmediateSubElements`, `updateSubElementImpl`, `failed`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `replaceSubElements`、`walkImmediateSubElements`、`updateSubElementImpl`、`failed` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 184-203
```cpp
  // If any sub-elements changed, use the new elements during the replacement.
  T result = interface;
  if (*changed)
    result = interface.replaceImmediateSubElements(newAttrs, newTypes);
  return result;
}

/// Shared implementation of replacing a given attribute or type element.
template <typename T, typename ReplaceFns, typename Replacer>
static T replaceElementImpl(T element, ReplaceFns &replaceFns,
                            Replacer &replacer) {
  T result = element;
  WalkResult walkResult = WalkResult::advance();
  for (auto &replaceFn : llvm::reverse(replaceFns)) {
    if (std::optional<std::pair<T, WalkResult>> newRes = replaceFn(element)) {
      std::tie(result, walkResult) = *newRes;
      break;
    }
  }

```
- **EN**: Implements logic around `replaceImmediateSubElements`, `replaceElementImpl`, `advance`, `reverse`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replaceImmediateSubElements`、`replaceElementImpl`、`advance`、`reverse` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 204-216
```cpp
  // If an error occurred, return nullptr to indicate failure.
  if (walkResult.wasInterrupted() || !result) {
    return nullptr;
  }

  // Handle replacing sub-elements if this element is also a container.
  if (!walkResult.wasSkipped()) {
    // Replace the sub elements of this element, bailing if we fail.
    if (!(result = replaceSubElements(result, replacer))) {
      return nullptr;
    }
  }

```
- **EN**: Implements logic around `wasInterrupted`, `wasSkipped`, `replaceSubElements`.
- **CN**: 围绕 `wasInterrupted`、`wasSkipped`、`replaceSubElements` 实现具体逻辑。

### Lines 217-231
```cpp
  return result;
}

template <typename Concrete>
Attribute detail::AttrTypeReplacerBase<Concrete>::replaceBase(Attribute attr) {
  return replaceElementImpl(attr, attrReplacementFns,
                            *static_cast<Concrete *>(this));
}

template <typename Concrete>
Type detail::AttrTypeReplacerBase<Concrete>::replaceBase(Type type) {
  return replaceElementImpl(type, typeReplacementFns,
                            *static_cast<Concrete *>(this));
}

```
- **EN**: Implements logic around `replaceBase`, `replaceElementImpl`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replaceBase`、`replaceElementImpl` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 232-244
```cpp
//===----------------------------------------------------------------------===//
/// AttrTypeReplacer
//===----------------------------------------------------------------------===//

template class detail::AttrTypeReplacerBase<AttrTypeReplacer>;

template <typename T>
T AttrTypeReplacer::cachedReplaceImpl(T element) {
  const void *opaqueElement = element.getAsOpaquePointer();
  auto [it, inserted] = cache.try_emplace(opaqueElement, opaqueElement);
  if (!inserted)
    return T::getFromOpaquePointer(it->second);

```
- **EN**: Implements logic around `cachedReplaceImpl`, `getAsOpaquePointer`, `try_emplace`, `getFromOpaquePointer`.
- **CN**: 围绕 `cachedReplaceImpl`、`getAsOpaquePointer`、`try_emplace`、`getFromOpaquePointer` 实现具体逻辑。

### Lines 245-254
```cpp
  T result = replaceBase(element);

  cache[opaqueElement] = result.getAsOpaquePointer();
  return result;
}

Attribute AttrTypeReplacer::replace(Attribute attr) {
  return cachedReplaceImpl(attr);
}

```
- **EN**: Implements logic around `replaceBase`, `getAsOpaquePointer`, `replace`, `cachedReplaceImpl`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replaceBase`、`getAsOpaquePointer`、`replace`、`cachedReplaceImpl` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 255-265
```cpp
Type AttrTypeReplacer::replace(Type type) { return cachedReplaceImpl(type); }

//===----------------------------------------------------------------------===//
/// CyclicAttrTypeReplacer
//===----------------------------------------------------------------------===//

template class detail::AttrTypeReplacerBase<CyclicAttrTypeReplacer>;

CyclicAttrTypeReplacer::CyclicAttrTypeReplacer()
    : cache([&](void *attr) { return breakCycleImpl(attr); }) {}

```
- **EN**: Implements logic around `replace`, `CyclicAttrTypeReplacer`, `cache`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replace`、`CyclicAttrTypeReplacer`、`cache` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 266-281
```cpp
void CyclicAttrTypeReplacer::addCycleBreaker(CycleBreakerFn<Attribute> fn) {
  attrCycleBreakerFns.emplace_back(std::move(fn));
}

void CyclicAttrTypeReplacer::addCycleBreaker(CycleBreakerFn<Type> fn) {
  typeCycleBreakerFns.emplace_back(std::move(fn));
}

template <typename T>
T CyclicAttrTypeReplacer::cachedReplaceImpl(T element) {
  void *opaqueTaggedElement = AttrOrType(element).getOpaqueValue();
  CyclicReplacerCache<void *, const void *>::CacheEntry cacheEntry =
      cache.lookupOrInit(opaqueTaggedElement);
  if (auto resultOpt = cacheEntry.get())
    return T::getFromOpaquePointer(*resultOpt);

```
- **EN**: Implements logic around `addCycleBreaker`, `emplace_back`, `cachedReplaceImpl`, `AttrOrType`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `addCycleBreaker`、`emplace_back`、`cachedReplaceImpl`、`AttrOrType` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 282-291
```cpp
  T result = replaceBase(element);

  cacheEntry.resolve(result.getAsOpaquePointer());
  return result;
}

Attribute CyclicAttrTypeReplacer::replace(Attribute attr) {
  return cachedReplaceImpl(attr);
}

```
- **EN**: Implements logic around `replaceBase`, `resolve`, `replace`, `cachedReplaceImpl`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replaceBase`、`resolve`、`replace`、`cachedReplaceImpl` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 292-311
```cpp
Type CyclicAttrTypeReplacer::replace(Type type) {
  return cachedReplaceImpl(type);
}

std::optional<const void *>
CyclicAttrTypeReplacer::breakCycleImpl(void *element) {
  AttrOrType attrType = AttrOrType::getFromOpaqueValue(element);
  if (auto attr = dyn_cast<Attribute>(attrType)) {
    for (auto &cyclicReplaceFn : llvm::reverse(attrCycleBreakerFns)) {
      if (std::optional<Attribute> newRes = cyclicReplaceFn(attr)) {
        return newRes->getAsOpaquePointer();
      }
    }
  } else {
    auto type = dyn_cast<Type>(attrType);
    for (auto &cyclicReplaceFn : llvm::reverse(typeCycleBreakerFns)) {
      if (std::optional<Type> newRes = cyclicReplaceFn(type)) {
        return newRes->getAsOpaquePointer();
      }
    }
```
- **EN**: Implements logic around `replace`, `cachedReplaceImpl`, `breakCycleImpl`, `getFromOpaqueValue`, and 4 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `replace`、`cachedReplaceImpl`、`breakCycleImpl`、`getFromOpaqueValue` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 312-324
```cpp
  }
  return std::nullopt;
}

//===----------------------------------------------------------------------===//
// AttrTypeImmediateSubElementWalker
//===----------------------------------------------------------------------===//

void AttrTypeImmediateSubElementWalker::walk(Attribute element) {
  if (element)
    walkAttrsFn(element);
}

```
- **EN**: Implements logic around `walk`, `walkAttrsFn`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `walk`、`walkAttrsFn` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 325-328
```cpp
void AttrTypeImmediateSubElementWalker::walk(Type element) {
  if (element)
    walkTypesFn(element);
}
```
- **EN**: Implements logic around `walk`, `walkTypesFn`; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `walk`、`walkTypesFn` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

## Key Concepts / 关键概念

- **Core IR modeling / 核心 IR 建模**:
  - **EN**: Defines operations, regions, blocks, attributes, types, and the assembly form that ties them together.
  - **CN**: 定义操作、区域、块、属性、类型，以及将它们串联起来的汇编表示。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/IR/Operation.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: core MLIR IR abstractions / 核心 MLIR IR 抽象 (1)
