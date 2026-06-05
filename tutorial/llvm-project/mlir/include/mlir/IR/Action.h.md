# Action.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/Action.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Action component. The leading comments describe it as: This file contains definitions for the action framework. This framework.
- **用途（CN）**: 声明 MLIR Action 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
````cpp
//===- Action.h -  Action Support ---------------------*- C++ -*-=============//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains definitions for the action framework. This framework
// allows for external entities to control certain actions taken by the compiler
// by registering handler functions.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_ACTION_H
#define MLIR_IR_ACTION_H

#include "mlir/IR/Unit.h"
#include "mlir/Support/TypeID.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/Sequence.h"
#include "llvm/ADT/StringMap.h"
#include "llvm/Support/TypeName.h"
#include "llvm/Support/raw_ostream.h"
#include <functional>
#include <type_traits>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 28-76
````cpp
namespace mlir {
namespace tracing {

/// An action is a specific action that is to be taken by the compiler,
/// that can be toggled and controlled by an external user. There are no
/// constraints on the granularity of an action, it could be as simple as
/// "perform this fold" and as complex as "run this pass pipeline".
///
/// This class represents the base class of the ActionImpl class (see below).
/// This holds the template-invariant elements of the Action class.
class Action {
public:
  virtual ~Action() = default;

  /// Return the unique action id of this action, use for casting
  /// functionality.
  TypeID getActionID() const { return actionID; }

  /// Return a string "tag" which intends to uniquely identify this type of
  /// action. For example "pass-application" or "pattern-rewrite".
  virtual StringRef getTag() const = 0;

  virtual void print(raw_ostream &os) const {
    os << "Action \"" << getTag() << "\"";
  }

  /// Return the set of IR units that are associated with this action.
  virtual ArrayRef<IRUnit> getContextIRUnits() const { return irUnits; }

protected:
  Action(TypeID actionID, ArrayRef<IRUnit> irUnits)
      : actionID(actionID), irUnits(irUnits) {}

  /// The type of the derived action class, used for `isa`/`dyn_cast`.
  TypeID actionID;

  /// Set of IR units (operations, regions, blocks, values) that are associated
  /// with this action.
  ArrayRef<IRUnit> irUnits;
};

/// CRTP Implementation of an action. This class provides a base class for
/// implementing specific actions.
///  Derived classes are expected to provide the following:
///   * static constexpr StringLiteral tag = "...";
///     - This method returns a unique string identifier, similar to a command
///       line flag or DEBUG_TYPE.
template <typename Derived>
class ActionImpl : public Action {
````
- **EN**: This C++ declaration introduces `Action` and establishes part of the API surface for `Action`. Representative entry points here include `Action`, `getActionID`, `getTag`, `print`.
- **CN**: 该 C++ 声明引入了 `Action`，并构成 `Action` API 表面的一部分。 这一段可见的代表性接口包括 `Action`, `getActionID`, `getTag`, `print`。

### Lines 77-91
````cpp
public:
  ActionImpl(ArrayRef<IRUnit> irUnits = {})
      : Action(TypeID::get<Derived>(), irUnits) {}

  /// Provide classof to allow casting between action types.
  static bool classof(const Action *action) {
    return action->getActionID() == TypeID::get<Derived>();
  }

  /// Forward tag access to the derived class.
  StringRef getTag() const final { return Derived::tag; }
};

} // namespace tracing
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `ActionImpl`, `Action`, `classof`, `getActionID`, indicating how `Action` is queried or updated.
- **CN**: 该代码块聚合了 `ActionImpl`, `Action`, `classof`, `getActionID` 等可调用接口，展示了如何查询或更新 `Action`。

### Lines 93-93
````cpp
#endif // MLIR_IR_ACTION_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/Unit.h
- mlir/Support/TypeID.h
- llvm/ADT/ArrayRef.h
- llvm/ADT/Sequence.h
- llvm/ADT/StringMap.h
- llvm/Support/TypeName.h
- llvm/Support/raw_ostream.h
- ActionImpl inherits from public Action
- ActionImpl builds on public Action
