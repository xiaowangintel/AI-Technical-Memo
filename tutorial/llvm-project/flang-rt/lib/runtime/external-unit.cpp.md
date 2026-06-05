# external-unit.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `flang-rt/lib/runtime/external-unit.cpp` | `flang-rt/lib/runtime/external-unit.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements the Flang runtime library, including descriptors, I/O, memory management, numerics, and execution support. In this file, the main focus is `external unit`; the header comment highlights: Implemenation of ExternalFileUnit for RT_USE_PSEUDO_FILE_UNIT=0.. | 实现 Flang 运行时库，包括描述符、I/O、内存管理、数值计算与执行支持。 本文件的核心主题是 `external unit`；文件头注释强调：Implemenation of ExternalFileUnit for RT_USE_PSEUDO_FILE_UNIT=0.。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===-- lib/runtime/external-unit.cpp ---------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implemenation of ExternalFileUnit for RT_USE_PSEUDO_FILE_UNIT=0.
//
//===----------------------------------------------------------------------===//

#include "unit-map.h"
#include "unit.h"
#include "flang-rt/runtime/io-error.h"
#include "flang-rt/runtime/lock.h"
#include "flang-rt/runtime/tools.h"

````

- **L1 EN**: Comment documents intent or context: `lib/runtime/external-unit.cpp ---------------------------*- C++ -*-===//`.
  **L1 CN**: 注释记录了意图或上下文：`lib/runtime/external-unit.cpp ---------------------------*- C++ -*-===//`。
- **L2 EN**: Comment line provides narrative context.
  **L2 CN**: 注释行提供叙述性上下文。
- **L3 EN**: Comment documents intent or context: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释记录了意图或上下文：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment documents intent or context: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释记录了意图或上下文：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents intent or context: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释记录了意图或上下文：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Comment line provides narrative context.
  **L6 CN**: 注释行提供叙述性上下文。
- **L7 EN**: Comment documents intent or context: `//`.
  **L7 CN**: 注释记录了意图或上下文：`//`。
- **L8 EN**: Comment line provides narrative context.
  **L8 CN**: 注释行提供叙述性上下文。
- **L9 EN**: Comment documents intent or context: `Implemenation of ExternalFileUnit for RT_USE_PSEUDO_FILE_UNIT=0.`.
  **L9 CN**: 注释记录了意图或上下文：`Implemenation of ExternalFileUnit for RT_USE_PSEUDO_FILE_UNIT=0.`。
- **L10 EN**: Comment line provides narrative context.
  **L10 CN**: 注释行提供叙述性上下文。
- **L11 EN**: Comment documents intent or context: `//`.
  **L11 CN**: 注释记录了意图或上下文：`//`。
- **L12 EN**: Blank line separates nearby declarations or logic blocks.
  **L12 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes `unit-map.h` to access project-local declarations and helper interfaces.
  **L13 CN**: 引入 `unit-map.h` 以使用 项目内声明与辅助接口。
- **L14 EN**: Includes `unit.h` to access project-local declarations and helper interfaces.
  **L14 CN**: 引入 `unit.h` 以使用 项目内声明与辅助接口。
- **L15 EN**: Includes `flang-rt/runtime/io-error.h` to access Flang runtime public headers.
  **L15 CN**: 引入 `flang-rt/runtime/io-error.h` 以使用 Flang 运行时公共头文件。
- **L16 EN**: Includes `flang-rt/runtime/lock.h` to access Flang runtime public headers.
  **L16 CN**: 引入 `flang-rt/runtime/lock.h` 以使用 Flang 运行时公共头文件。
- **L17 EN**: Includes `flang-rt/runtime/tools.h` to access Flang runtime public headers.
  **L17 CN**: 引入 `flang-rt/runtime/tools.h` 以使用 Flang 运行时公共头文件。
- **L18 EN**: Blank line separates nearby declarations or logic blocks.
  **L18 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 19-36

````cpp
#if !defined(RT_USE_PSEUDO_FILE_UNIT)

#include <cstdio>
#include <limits>

namespace Fortran::runtime::io {

// The per-unit data structures are created on demand so that Fortran I/O
// should work without a Fortran main program.
static Lock unitMapLock;
static Lock createOpenLock;
static UnitMap *unitMap{nullptr};

void FlushOutputOnCrash(const Terminator &terminator) {
  if (!defaultOutput && !errorOutput) {
    return;
  }
  IoErrorHandler handler{terminator};
````

- **L19 EN**: Preprocessor directive manages conditional compilation or macros: `#if !defined(RT_USE_PSEUDO_FILE_UNIT)`.
  **L19 CN**: 预处理指令管理条件编译或宏：`#if !defined(RT_USE_PSEUDO_FILE_UNIT)`。
- **L20 EN**: Blank line separates nearby declarations or logic blocks.
  **L20 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L21 EN**: Includes `cstdio` to access C stdio facilities.
  **L21 CN**: 引入 `cstdio` 以使用 C 标准输入输出设施。
- **L22 EN**: Includes `limits` to access type limits.
  **L22 CN**: 引入 `limits` 以使用 类型范围。
- **L23 EN**: Blank line separates nearby declarations or logic blocks.
  **L23 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L24 EN**: Enters namespace `Fortran` to scope related declarations.
  **L24 CN**: 进入命名空间 `Fortran` 以组织相关声明。
- **L25 EN**: Blank line separates nearby declarations or logic blocks.
  **L25 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L26 EN**: Comment documents intent or context: `The per-unit data structures are created on demand so that Fortran I/O`.
  **L26 CN**: 注释记录了意图或上下文：`The per-unit data structures are created on demand so that Fortran I/O`。
- **L27 EN**: Comment documents intent or context: `should work without a Fortran main program.`.
  **L27 CN**: 注释记录了意图或上下文：`should work without a Fortran main program.`。
- **L28 EN**: Executes statement `static Lock unitMapLock;`.
  **L28 CN**: 执行语句 `static Lock unitMapLock;`。
- **L29 EN**: Executes statement `static Lock createOpenLock;`.
  **L29 CN**: 执行语句 `static Lock createOpenLock;`。
- **L30 EN**: Executes statement `static UnitMap *unitMap{nullptr};`.
  **L30 CN**: 执行语句 `static UnitMap *unitMap{nullptr};`。
- **L31 EN**: Blank line separates nearby declarations or logic blocks.
  **L31 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L32 EN**: Declares or defines callable `FlushOutputOnCrash`.
  **L32 CN**: 声明或定义可调用实体 `FlushOutputOnCrash`。
- **L33 EN**: Introduces conditional control flow with an `if` statement.
  **L33 CN**: 通过 `if` 语句引入条件控制流。
- **L34 EN**: Returns from the current function, often propagating a computed result.
  **L34 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L35 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L35 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L36 EN**: Executes statement `IoErrorHandler handler{terminator};`.
  **L36 CN**: 执行语句 `IoErrorHandler handler{terminator};`。

### Lines 37-54

````cpp
  handler.HasIoStat(); // prevent nested crash if flush has error
  CriticalSection critical{unitMapLock};
  if (defaultOutput) {
    defaultOutput->FlushOutput(handler);
  }
  if (errorOutput) {
    errorOutput->FlushOutput(handler);
  }
}

ExternalFileUnit *ExternalFileUnit::LookUp(int unit, Terminator &terminator) {
  return GetUnitMap(terminator).LookUp(unit);
}

ExternalFileUnit *ExternalFileUnit::LookUpOrCreate(
    int unit, const Terminator &terminator, bool &wasExtant) {
  return GetUnitMap(terminator).LookUpOrCreate(unit, terminator, wasExtant);
}
````

- **L37 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L37 CN**: 延续周围的声明、表达式或控制流结构。
- **L38 EN**: Executes statement `CriticalSection critical{unitMapLock};`.
  **L38 CN**: 执行语句 `CriticalSection critical{unitMapLock};`。
- **L39 EN**: Introduces conditional control flow with an `if` statement.
  **L39 CN**: 通过 `if` 语句引入条件控制流。
- **L40 EN**: Executes statement involving `FlushOutput`.
  **L40 CN**: 执行涉及 `FlushOutput` 的语句。
- **L41 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L41 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L42 EN**: Introduces conditional control flow with an `if` statement.
  **L42 CN**: 通过 `if` 语句引入条件控制流。
- **L43 EN**: Executes statement involving `FlushOutput`.
  **L43 CN**: 执行涉及 `FlushOutput` 的语句。
- **L44 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L44 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L45 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L45 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L46 EN**: Blank line separates nearby declarations or logic blocks.
  **L46 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L47 EN**: Declares or defines callable `LookUp`.
  **L47 CN**: 声明或定义可调用实体 `LookUp`。
- **L48 EN**: Returns from the current function, often propagating a computed result.
  **L48 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L49 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L49 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L50 EN**: Blank line separates nearby declarations or logic blocks.
  **L50 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L51 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L51 CN**: 延续周围的声明、表达式或控制流结构。
- **L52 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L52 CN**: 延续周围的声明、表达式或控制流结构。
- **L53 EN**: Returns from the current function, often propagating a computed result.
  **L53 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L54 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L54 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 55-72

````cpp

ExternalFileUnit *ExternalFileUnit::LookUpOrCreateAnonymous(int unit,
    Direction dir, common::optional<bool> isUnformatted,
    IoErrorHandler &handler) {
  // Make sure that the returned anonymous unit has been opened,
  // not just created in the unitMap.
  CriticalSection critical{createOpenLock};
  bool exists{false};
  ExternalFileUnit *result{
      GetUnitMap(handler).LookUpOrCreate(unit, handler, exists)};
  if (result && !exists) {
    common::optional<Action> action;
    if (dir == Direction::Output) {
      action = Action::ReadWrite;
    }
    if (!result->OpenAnonymousUnit(
            dir == Direction::Input ? OpenStatus::Unknown : OpenStatus::Replace,
            action, Position::Rewind, Convert::Unknown, handler)) {
````

- **L55 EN**: Blank line separates nearby declarations or logic blocks.
  **L55 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L56 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L56 CN**: 延续周围的声明、表达式或控制流结构。
- **L57 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L57 CN**: 延续周围的声明、表达式或控制流结构。
- **L58 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L58 CN**: 延续周围的声明、表达式或控制流结构。
- **L59 EN**: Comment documents intent or context: `Make sure that the returned anonymous unit has been opened,`.
  **L59 CN**: 注释记录了意图或上下文：`Make sure that the returned anonymous unit has been opened,`。
- **L60 EN**: Comment documents intent or context: `not just created in the unitMap.`.
  **L60 CN**: 注释记录了意图或上下文：`not just created in the unitMap.`。
- **L61 EN**: Executes statement `CriticalSection critical{createOpenLock};`.
  **L61 CN**: 执行语句 `CriticalSection critical{createOpenLock};`。
- **L62 EN**: Executes statement `bool exists{false};`.
  **L62 CN**: 执行语句 `bool exists{false};`。
- **L63 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L63 CN**: 延续周围的声明、表达式或控制流结构。
- **L64 EN**: Executes statement involving `GetUnitMap`.
  **L64 CN**: 执行涉及 `GetUnitMap` 的语句。
- **L65 EN**: Introduces conditional control flow with an `if` statement.
  **L65 CN**: 通过 `if` 语句引入条件控制流。
- **L66 EN**: Executes statement `common::optional<Action> action;`.
  **L66 CN**: 执行语句 `common::optional<Action> action;`。
- **L67 EN**: Introduces conditional control flow with an `if` statement.
  **L67 CN**: 通过 `if` 语句引入条件控制流。
- **L68 EN**: Initializes or updates `action`.
  **L68 CN**: 初始化或更新 `action`。
- **L69 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L69 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L70 EN**: Introduces conditional control flow with an `if` statement.
  **L70 CN**: 通过 `if` 语句引入条件控制流。
- **L71 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L71 CN**: 延续周围的声明、表达式或控制流结构。
- **L72 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L72 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 73-90

````cpp
      // fort.N isn't a writable file
      if (ExternalFileUnit *
          closed{LookUpForClose(result->unitNumber(), handler)}) {
        closed->DestroyClosed(handler);
      }
      result = nullptr;
    } else {
      result->isUnformatted = isUnformatted;
    }
  }
  return result;
}

ExternalFileUnit *ExternalFileUnit::LookUp(
    const char *path, std::size_t pathLen, Terminator &terminator) {
  return GetUnitMap(terminator).LookUp(path, pathLen);
}

````

- **L73 EN**: Comment documents intent or context: `fort.N isn't a writable file`.
  **L73 CN**: 注释记录了意图或上下文：`fort.N isn't a writable file`。
- **L74 EN**: Introduces conditional control flow with an `if` statement.
  **L74 CN**: 通过 `if` 语句引入条件控制流。
- **L75 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L75 CN**: 延续周围的声明、表达式或控制流结构。
- **L76 EN**: Executes statement involving `DestroyClosed`.
  **L76 CN**: 执行涉及 `DestroyClosed` 的语句。
- **L77 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L77 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L78 EN**: Initializes or updates `result`.
  **L78 CN**: 初始化或更新 `result`。
- **L79 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L79 CN**: 延续周围的声明、表达式或控制流结构。
- **L80 EN**: Initializes or updates `result->isUnformatted`.
  **L80 CN**: 初始化或更新 `result->isUnformatted`。
- **L81 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L81 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L82 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L82 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L83 EN**: Returns from the current function, often propagating a computed result.
  **L83 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L84 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L84 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L85 EN**: Blank line separates nearby declarations or logic blocks.
  **L85 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L86 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L86 CN**: 延续周围的声明、表达式或控制流结构。
- **L87 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L87 CN**: 延续周围的声明、表达式或控制流结构。
- **L88 EN**: Returns from the current function, often propagating a computed result.
  **L88 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L89 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L89 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L90 EN**: Blank line separates nearby declarations or logic blocks.
  **L90 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 91-108

````cpp
ExternalFileUnit &ExternalFileUnit::CreateNew(
    int unit, const Terminator &terminator) {
  bool wasExtant{false};
  ExternalFileUnit *result{
      GetUnitMap(terminator).LookUpOrCreate(unit, terminator, wasExtant)};
  RUNTIME_CHECK(terminator, result && !wasExtant);
  return *result;
}

ExternalFileUnit *ExternalFileUnit::LookUpForClose(
    int unit, Terminator &terminator) {
  return GetUnitMap(terminator).LookUpForClose(unit);
}

ExternalFileUnit &ExternalFileUnit::NewUnit(
    const Terminator &terminator, bool forChildIo) {
  ExternalFileUnit &unit{GetUnitMap(terminator).NewUnit(terminator)};
  unit.createdForInternalChildIo_ = forChildIo;
````

- **L91 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L91 CN**: 延续周围的声明、表达式或控制流结构。
- **L92 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L92 CN**: 延续周围的声明、表达式或控制流结构。
- **L93 EN**: Executes statement `bool wasExtant{false};`.
  **L93 CN**: 执行语句 `bool wasExtant{false};`。
- **L94 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L94 CN**: 延续周围的声明、表达式或控制流结构。
- **L95 EN**: Executes statement involving `GetUnitMap`.
  **L95 CN**: 执行涉及 `GetUnitMap` 的语句。
- **L96 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L96 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L97 EN**: Returns from the current function, often propagating a computed result.
  **L97 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L98 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L98 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L99 EN**: Blank line separates nearby declarations or logic blocks.
  **L99 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L100 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L100 CN**: 延续周围的声明、表达式或控制流结构。
- **L101 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L101 CN**: 延续周围的声明、表达式或控制流结构。
- **L102 EN**: Returns from the current function, often propagating a computed result.
  **L102 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L103 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L103 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L104 EN**: Blank line separates nearby declarations or logic blocks.
  **L104 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L105 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L105 CN**: 延续周围的声明、表达式或控制流结构。
- **L106 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L106 CN**: 延续周围的声明、表达式或控制流结构。
- **L107 EN**: Executes statement involving `GetUnitMap`.
  **L107 CN**: 执行涉及 `GetUnitMap` 的语句。
- **L108 EN**: Initializes or updates `unit.createdForInternalChildIo_`.
  **L108 CN**: 初始化或更新 `unit.createdForInternalChildIo_`。

### Lines 109-126

````cpp
  return unit;
}

bool ExternalFileUnit::OpenUnit(common::optional<OpenStatus> status,
    common::optional<Action> action, Position position,
    OwningPtr<char> &&newPath, std::size_t newPathLength, Convert convert,
    IoErrorHandler &handler) {
  if (convert == Convert::Unknown) {
    convert = executionEnvironment.conversion;
  }
  swapEndianness_ = convert == Convert::Swap ||
      (convert == Convert::LittleEndian && !isHostLittleEndian) ||
      (convert == Convert::BigEndian && isHostLittleEndian);
  bool impliedClose{false};
  if (IsConnected()) {
    bool isSamePath{newPath.get() && path() && pathLength() == newPathLength &&
        runtime::memcmp(path(), newPath.get(), newPathLength) == 0};
    if (status && *status != OpenStatus::Old && isSamePath) {
````

- **L109 EN**: Returns from the current function, often propagating a computed result.
  **L109 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L110 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L110 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L111 EN**: Blank line separates nearby declarations or logic blocks.
  **L111 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L112 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L112 CN**: 延续周围的声明、表达式或控制流结构。
- **L113 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L113 CN**: 延续周围的声明、表达式或控制流结构。
- **L114 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L114 CN**: 延续周围的声明、表达式或控制流结构。
- **L115 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L115 CN**: 延续周围的声明、表达式或控制流结构。
- **L116 EN**: Introduces conditional control flow with an `if` statement.
  **L116 CN**: 通过 `if` 语句引入条件控制流。
- **L117 EN**: Initializes or updates `convert`.
  **L117 CN**: 初始化或更新 `convert`。
- **L118 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L118 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L119 EN**: Initializes or updates `swapEndianness_`.
  **L119 CN**: 初始化或更新 `swapEndianness_`。
- **L120 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L120 CN**: 延续周围的声明、表达式或控制流结构。
- **L121 EN**: Executes statement `(convert == Convert::BigEndian && isHostLittleEndian);`.
  **L121 CN**: 执行语句 `(convert == Convert::BigEndian && isHostLittleEndian);`。
- **L122 EN**: Executes statement `bool impliedClose{false};`.
  **L122 CN**: 执行语句 `bool impliedClose{false};`。
- **L123 EN**: Introduces conditional control flow with an `if` statement.
  **L123 CN**: 通过 `if` 语句引入条件控制流。
- **L124 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L124 CN**: 延续周围的声明、表达式或控制流结构。
- **L125 EN**: Executes statement involving `memcmp`.
  **L125 CN**: 执行涉及 `memcmp` 的语句。
- **L126 EN**: Introduces conditional control flow with an `if` statement.
  **L126 CN**: 通过 `if` 语句引入条件控制流。

### Lines 127-144

````cpp
      handler.SignalError("OPEN statement for connected unit may not have "
                          "explicit STATUS= other than 'OLD'");
      return impliedClose;
    }
    if (!newPath.get() || isSamePath) {
      // OPEN of existing unit, STATUS='OLD' or unspecified, not new FILE=
      newPath.reset();
      Open(status.value_or(OpenStatus::Old), action, position, handler);
      return impliedClose;
    }
    // Otherwise, OPEN on open unit with new FILE= implies CLOSE
    DoImpliedEndfile(handler);
    FlushOutput(handler);
    TruncateFrame(0, handler);
    Close(CloseStatus::Keep, handler);
    impliedClose = true;
  }
  if (newPath.get() && newPathLength > 0) {
````

- **L127 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L127 CN**: 延续周围的声明、表达式或控制流结构。
- **L128 EN**: Executes statement `"explicit STATUS= other than 'OLD'");`.
  **L128 CN**: 执行语句 `"explicit STATUS= other than 'OLD'");`。
- **L129 EN**: Returns from the current function, often propagating a computed result.
  **L129 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L130 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L130 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L131 EN**: Introduces conditional control flow with an `if` statement.
  **L131 CN**: 通过 `if` 语句引入条件控制流。
- **L132 EN**: Comment documents intent or context: `OPEN of existing unit, STATUS='OLD' or unspecified, not new FILE=`.
  **L132 CN**: 注释记录了意图或上下文：`OPEN of existing unit, STATUS='OLD' or unspecified, not new FILE=`。
- **L133 EN**: Executes statement involving `reset`.
  **L133 CN**: 执行涉及 `reset` 的语句。
- **L134 EN**: Executes statement involving `Open`.
  **L134 CN**: 执行涉及 `Open` 的语句。
- **L135 EN**: Returns from the current function, often propagating a computed result.
  **L135 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L136 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L136 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L137 EN**: Comment documents intent or context: `Otherwise, OPEN on open unit with new FILE= implies CLOSE`.
  **L137 CN**: 注释记录了意图或上下文：`Otherwise, OPEN on open unit with new FILE= implies CLOSE`。
- **L138 EN**: Executes statement involving `DoImpliedEndfile`.
  **L138 CN**: 执行涉及 `DoImpliedEndfile` 的语句。
- **L139 EN**: Executes statement involving `FlushOutput`.
  **L139 CN**: 执行涉及 `FlushOutput` 的语句。
- **L140 EN**: Executes statement involving `TruncateFrame`.
  **L140 CN**: 执行涉及 `TruncateFrame` 的语句。
- **L141 EN**: Executes statement involving `Close`.
  **L141 CN**: 执行涉及 `Close` 的语句。
- **L142 EN**: Initializes or updates `impliedClose`.
  **L142 CN**: 初始化或更新 `impliedClose`。
- **L143 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L143 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L144 EN**: Introduces conditional control flow with an `if` statement.
  **L144 CN**: 通过 `if` 语句引入条件控制流。

### Lines 145-162

````cpp
    if (const auto *already{
            GetUnitMap(handler).LookUp(newPath.get(), newPathLength)}) {
      handler.SignalError(IostatOpenAlreadyConnected,
          "OPEN(UNIT=%d,FILE='%.*s'): file is already connected to unit %d",
          unitNumber_, static_cast<int>(newPathLength), newPath.get(),
          already->unitNumber_);
      return impliedClose;
    }
  }
  set_path(std::move(newPath), newPathLength);
  Open(status.value_or(OpenStatus::Unknown), action, position, handler);
  if (handler.InError()) {
    return impliedClose;
  }
  auto totalBytes{knownSize()};
  if (access == Access::Direct) {
    if (!openRecl) {
      handler.SignalError(IostatOpenBadRecl,
````

- **L145 EN**: Introduces conditional control flow with an `if` statement.
  **L145 CN**: 通过 `if` 语句引入条件控制流。
- **L146 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L146 CN**: 延续周围的声明、表达式或控制流结构。
- **L147 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L147 CN**: 延续周围的声明、表达式或控制流结构。
- **L148 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L148 CN**: 延续周围的声明、表达式或控制流结构。
- **L149 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L149 CN**: 延续周围的声明、表达式或控制流结构。
- **L150 EN**: Executes statement `already->unitNumber_);`.
  **L150 CN**: 执行语句 `already->unitNumber_);`。
- **L151 EN**: Returns from the current function, often propagating a computed result.
  **L151 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L152 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L152 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L153 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L153 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L154 EN**: Executes statement involving `set_path`.
  **L154 CN**: 执行涉及 `set_path` 的语句。
- **L155 EN**: Executes statement involving `Open`.
  **L155 CN**: 执行涉及 `Open` 的语句。
- **L156 EN**: Introduces conditional control flow with an `if` statement.
  **L156 CN**: 通过 `if` 语句引入条件控制流。
- **L157 EN**: Returns from the current function, often propagating a computed result.
  **L157 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L158 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L158 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L159 EN**: Executes statement involving `knownSize`.
  **L159 CN**: 执行涉及 `knownSize` 的语句。
- **L160 EN**: Introduces conditional control flow with an `if` statement.
  **L160 CN**: 通过 `if` 语句引入条件控制流。
- **L161 EN**: Introduces conditional control flow with an `if` statement.
  **L161 CN**: 通过 `if` 语句引入条件控制流。
- **L162 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L162 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 163-180

````cpp
          "OPEN(UNIT=%d,ACCESS='DIRECT'): record length is not known",
          unitNumber());
    } else if (*openRecl <= 0) {
      handler.SignalError(IostatOpenBadRecl,
          "OPEN(UNIT=%d,ACCESS='DIRECT',RECL=%jd): record length is invalid",
          unitNumber(), static_cast<std::intmax_t>(*openRecl));
    } else if (totalBytes && (*totalBytes % *openRecl != 0)) {
      handler.SignalError(IostatOpenBadRecl,
          "OPEN(UNIT=%d,ACCESS='DIRECT',RECL=%jd): record length is not an "
          "even divisor of the file size %jd",
          unitNumber(), static_cast<std::intmax_t>(*openRecl),
          static_cast<std::intmax_t>(*totalBytes));
    }
    recordLength = openRecl;
  }
  endfileRecordNumber.reset();
  currentRecordNumber = 1;
  if (totalBytes && access == Access::Direct && openRecl.value_or(0) > 0) {
````

- **L163 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L163 CN**: 延续周围的声明、表达式或控制流结构。
- **L164 EN**: Executes statement involving `unitNumber`.
  **L164 CN**: 执行涉及 `unitNumber` 的语句。
- **L165 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L165 CN**: 延续周围的声明、表达式或控制流结构。
- **L166 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L166 CN**: 延续周围的声明、表达式或控制流结构。
- **L167 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L167 CN**: 延续周围的声明、表达式或控制流结构。
- **L168 EN**: Executes statement involving `unitNumber`.
  **L168 CN**: 执行涉及 `unitNumber` 的语句。
- **L169 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L169 CN**: 延续周围的声明、表达式或控制流结构。
- **L170 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L170 CN**: 延续周围的声明、表达式或控制流结构。
- **L171 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L171 CN**: 延续周围的声明、表达式或控制流结构。
- **L172 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L172 CN**: 延续周围的声明、表达式或控制流结构。
- **L173 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L173 CN**: 延续周围的声明、表达式或控制流结构。
- **L174 EN**: Executes statement `static_cast<std::intmax_t>(*totalBytes));`.
  **L174 CN**: 执行语句 `static_cast<std::intmax_t>(*totalBytes));`。
- **L175 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L175 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L176 EN**: Initializes or updates `recordLength`.
  **L176 CN**: 初始化或更新 `recordLength`。
- **L177 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L177 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L178 EN**: Executes statement involving `reset`.
  **L178 CN**: 执行涉及 `reset` 的语句。
- **L179 EN**: Initializes or updates `currentRecordNumber`.
  **L179 CN**: 初始化或更新 `currentRecordNumber`。
- **L180 EN**: Introduces conditional control flow with an `if` statement.
  **L180 CN**: 通过 `if` 语句引入条件控制流。

### Lines 181-198

````cpp
    endfileRecordNumber = 1 + (*totalBytes / *openRecl);
  }
  if (position == Position::Append) {
    if (totalBytes) {
      frameOffsetInFile_ = *totalBytes;
    }
    if (access != Access::Stream) {
      if (!endfileRecordNumber) {
        // Fake it so that we can backspace relative from the end
        endfileRecordNumber = std::numeric_limits<std::int64_t>::max() - 2;
      }
      currentRecordNumber = *endfileRecordNumber;
    }
  }
  return impliedClose;
}

bool ExternalFileUnit::OpenAnonymousUnit(common::optional<OpenStatus> status,
````

- **L181 EN**: Initializes or updates `endfileRecordNumber`.
  **L181 CN**: 初始化或更新 `endfileRecordNumber`。
- **L182 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L182 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L183 EN**: Introduces conditional control flow with an `if` statement.
  **L183 CN**: 通过 `if` 语句引入条件控制流。
- **L184 EN**: Introduces conditional control flow with an `if` statement.
  **L184 CN**: 通过 `if` 语句引入条件控制流。
- **L185 EN**: Initializes or updates `frameOffsetInFile_`.
  **L185 CN**: 初始化或更新 `frameOffsetInFile_`。
- **L186 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L186 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L187 EN**: Introduces conditional control flow with an `if` statement.
  **L187 CN**: 通过 `if` 语句引入条件控制流。
- **L188 EN**: Introduces conditional control flow with an `if` statement.
  **L188 CN**: 通过 `if` 语句引入条件控制流。
- **L189 EN**: Comment documents intent or context: `Fake it so that we can backspace relative from the end`.
  **L189 CN**: 注释记录了意图或上下文：`Fake it so that we can backspace relative from the end`。
- **L190 EN**: Initializes or updates `endfileRecordNumber`.
  **L190 CN**: 初始化或更新 `endfileRecordNumber`。
- **L191 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L191 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L192 EN**: Initializes or updates `currentRecordNumber`.
  **L192 CN**: 初始化或更新 `currentRecordNumber`。
- **L193 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L193 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L194 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L194 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L195 EN**: Returns from the current function, often propagating a computed result.
  **L195 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L196 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L196 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L197 EN**: Blank line separates nearby declarations or logic blocks.
  **L197 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L198 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L198 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 199-216

````cpp
    common::optional<Action> action, Position position, Convert convert,
    IoErrorHandler &handler) {
  // I/O to an unconnected unit reads/creates a local file, e.g. fort.7
  std::size_t pathMaxLen{32};
  auto path{SizedNew<char>{handler}(pathMaxLen)};
  int len = std::snprintf(path.get(), pathMaxLen, "fort.%d", unitNumber_);
  OpenUnit(status, action, position, std::move(path),
      len >= 0 ? static_cast<std::size_t>(len) : 0, convert, handler);
  return IsConnected();
}

void ExternalFileUnit::CloseUnit(CloseStatus status, IoErrorHandler &handler) {
  DoImpliedEndfile(handler);
  FlushOutput(handler);
  Close(status, handler);
}

void ExternalFileUnit::DestroyClosed(Terminator &terminator) {
````

- **L199 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L199 CN**: 延续周围的声明、表达式或控制流结构。
- **L200 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L200 CN**: 延续周围的声明、表达式或控制流结构。
- **L201 EN**: Comment documents intent or context: `I/O to an unconnected unit reads/creates a local file, e.g. fort.7`.
  **L201 CN**: 注释记录了意图或上下文：`I/O to an unconnected unit reads/creates a local file, e.g. fort.7`。
- **L202 EN**: Executes statement `std::size_t pathMaxLen{32};`.
  **L202 CN**: 执行语句 `std::size_t pathMaxLen{32};`。
- **L203 EN**: Executes statement `auto path{SizedNew<char>{handler}(pathMaxLen)};`.
  **L203 CN**: 执行语句 `auto path{SizedNew<char>{handler}(pathMaxLen)};`。
- **L204 EN**: Initializes or updates `len`.
  **L204 CN**: 初始化或更新 `len`。
- **L205 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L205 CN**: 延续周围的声明、表达式或控制流结构。
- **L206 EN**: Executes statement `len >= 0 ? static_cast<std::size_t>(len) : 0, convert, handler);`.
  **L206 CN**: 执行语句 `len >= 0 ? static_cast<std::size_t>(len) : 0, convert, handler);`。
- **L207 EN**: Returns from the current function, often propagating a computed result.
  **L207 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L208 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L208 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L209 EN**: Blank line separates nearby declarations or logic blocks.
  **L209 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L210 EN**: Declares or defines callable `CloseUnit`.
  **L210 CN**: 声明或定义可调用实体 `CloseUnit`。
- **L211 EN**: Executes statement involving `DoImpliedEndfile`.
  **L211 CN**: 执行涉及 `DoImpliedEndfile` 的语句。
- **L212 EN**: Executes statement involving `FlushOutput`.
  **L212 CN**: 执行涉及 `FlushOutput` 的语句。
- **L213 EN**: Executes statement involving `Close`.
  **L213 CN**: 执行涉及 `Close` 的语句。
- **L214 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L214 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L215 EN**: Blank line separates nearby declarations or logic blocks.
  **L215 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L216 EN**: Declares or defines callable `DestroyClosed`.
  **L216 CN**: 声明或定义可调用实体 `DestroyClosed`。

### Lines 217-234

````cpp
  GetUnitMap(terminator).DestroyClosed(*this); // destroys *this
}

Iostat ExternalFileUnit::SetDirection(Direction direction) {
  if (direction == Direction::Input) {
    if (mayRead()) {
      direction_ = Direction::Input;
      return IostatOk;
    } else {
      return IostatReadFromWriteOnly;
    }
  } else {
    if (mayWrite()) {
      if (direction_ == Direction::Input) {
        // Don't retain any input data from previous record, like a
        // variable-length unformatted record footer, in the frame,
        // since we're going start writing frames.
        frameOffsetInFile_ += recordOffsetInFrame_;
````

- **L217 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L217 CN**: 延续周围的声明、表达式或控制流结构。
- **L218 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L218 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L219 EN**: Blank line separates nearby declarations or logic blocks.
  **L219 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L220 EN**: Declares or defines callable `SetDirection`.
  **L220 CN**: 声明或定义可调用实体 `SetDirection`。
- **L221 EN**: Introduces conditional control flow with an `if` statement.
  **L221 CN**: 通过 `if` 语句引入条件控制流。
- **L222 EN**: Introduces conditional control flow with an `if` statement.
  **L222 CN**: 通过 `if` 语句引入条件控制流。
- **L223 EN**: Initializes or updates `direction_`.
  **L223 CN**: 初始化或更新 `direction_`。
- **L224 EN**: Returns from the current function, often propagating a computed result.
  **L224 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L225 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L225 CN**: 延续周围的声明、表达式或控制流结构。
- **L226 EN**: Returns from the current function, often propagating a computed result.
  **L226 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L227 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L227 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L228 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L228 CN**: 延续周围的声明、表达式或控制流结构。
- **L229 EN**: Introduces conditional control flow with an `if` statement.
  **L229 CN**: 通过 `if` 语句引入条件控制流。
- **L230 EN**: Introduces conditional control flow with an `if` statement.
  **L230 CN**: 通过 `if` 语句引入条件控制流。
- **L231 EN**: Comment documents intent or context: `Don't retain any input data from previous record, like a`.
  **L231 CN**: 注释记录了意图或上下文：`Don't retain any input data from previous record, like a`。
- **L232 EN**: Comment documents intent or context: `variable-length unformatted record footer, in the frame,`.
  **L232 CN**: 注释记录了意图或上下文：`variable-length unformatted record footer, in the frame,`。
- **L233 EN**: Comment documents intent or context: `since we're going start writing frames.`.
  **L233 CN**: 注释记录了意图或上下文：`since we're going start writing frames.`。
- **L234 EN**: Initializes or updates `+`.
  **L234 CN**: 初始化或更新 `+`。

### Lines 235-252

````cpp
        recordOffsetInFrame_ = 0;
      }
      direction_ = Direction::Output;
      return IostatOk;
    } else {
      return IostatWriteToReadOnly;
    }
  }
}

UnitMap &ExternalFileUnit::CreateUnitMap(const Terminator &terminator) {
  IoErrorHandler handler{terminator};
  UnitMap &newUnitMap{*New<UnitMap>{terminator}().release()};

  bool wasExtant{false};
  ExternalFileUnit &out{*newUnitMap.LookUpOrCreate(
      FORTRAN_DEFAULT_OUTPUT_UNIT, terminator, wasExtant)};
  RUNTIME_CHECK(terminator, !wasExtant);
````

- **L235 EN**: Initializes or updates `recordOffsetInFrame_`.
  **L235 CN**: 初始化或更新 `recordOffsetInFrame_`。
- **L236 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L236 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L237 EN**: Initializes or updates `direction_`.
  **L237 CN**: 初始化或更新 `direction_`。
- **L238 EN**: Returns from the current function, often propagating a computed result.
  **L238 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L239 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L239 CN**: 延续周围的声明、表达式或控制流结构。
- **L240 EN**: Returns from the current function, often propagating a computed result.
  **L240 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L241 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L241 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L242 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L242 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L243 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L243 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L244 EN**: Blank line separates nearby declarations or logic blocks.
  **L244 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L245 EN**: Declares or defines callable `CreateUnitMap`.
  **L245 CN**: 声明或定义可调用实体 `CreateUnitMap`。
- **L246 EN**: Executes statement `IoErrorHandler handler{terminator};`.
  **L246 CN**: 执行语句 `IoErrorHandler handler{terminator};`。
- **L247 EN**: Executes statement involving `release`.
  **L247 CN**: 执行涉及 `release` 的语句。
- **L248 EN**: Blank line separates nearby declarations or logic blocks.
  **L248 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L249 EN**: Executes statement `bool wasExtant{false};`.
  **L249 CN**: 执行语句 `bool wasExtant{false};`。
- **L250 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L250 CN**: 延续周围的声明、表达式或控制流结构。
- **L251 EN**: Executes statement `FORTRAN_DEFAULT_OUTPUT_UNIT, terminator, wasExtant)};`.
  **L251 CN**: 执行语句 `FORTRAN_DEFAULT_OUTPUT_UNIT, terminator, wasExtant)};`。
- **L252 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L252 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。

### Lines 253-270

````cpp
  out.Predefine(1);
  handler.SignalError(out.SetDirection(Direction::Output));
  out.isUnformatted = false;
  defaultOutput = &out;

  ExternalFileUnit &in{*newUnitMap.LookUpOrCreate(
      FORTRAN_DEFAULT_INPUT_UNIT, terminator, wasExtant)};
  RUNTIME_CHECK(terminator, !wasExtant);
  in.Predefine(0);
  handler.SignalError(in.SetDirection(Direction::Input));
  in.isUnformatted = false;
  defaultInput = &in;

  ExternalFileUnit &error{
      *newUnitMap.LookUpOrCreate(FORTRAN_ERROR_UNIT, terminator, wasExtant)};
  RUNTIME_CHECK(terminator, !wasExtant);
  error.Predefine(2);
  handler.SignalError(error.SetDirection(Direction::Output));
````

- **L253 EN**: Executes statement involving `Predefine`.
  **L253 CN**: 执行涉及 `Predefine` 的语句。
- **L254 EN**: Executes statement involving `SignalError`.
  **L254 CN**: 执行涉及 `SignalError` 的语句。
- **L255 EN**: Initializes or updates `out.isUnformatted`.
  **L255 CN**: 初始化或更新 `out.isUnformatted`。
- **L256 EN**: Initializes or updates `defaultOutput`.
  **L256 CN**: 初始化或更新 `defaultOutput`。
- **L257 EN**: Blank line separates nearby declarations or logic blocks.
  **L257 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L258 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L258 CN**: 延续周围的声明、表达式或控制流结构。
- **L259 EN**: Executes statement `FORTRAN_DEFAULT_INPUT_UNIT, terminator, wasExtant)};`.
  **L259 CN**: 执行语句 `FORTRAN_DEFAULT_INPUT_UNIT, terminator, wasExtant)};`。
- **L260 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L260 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L261 EN**: Executes statement involving `Predefine`.
  **L261 CN**: 执行涉及 `Predefine` 的语句。
- **L262 EN**: Executes statement involving `SignalError`.
  **L262 CN**: 执行涉及 `SignalError` 的语句。
- **L263 EN**: Initializes or updates `in.isUnformatted`.
  **L263 CN**: 初始化或更新 `in.isUnformatted`。
- **L264 EN**: Initializes or updates `defaultInput`.
  **L264 CN**: 初始化或更新 `defaultInput`。
- **L265 EN**: Blank line separates nearby declarations or logic blocks.
  **L265 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L266 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L266 CN**: 延续周围的声明、表达式或控制流结构。
- **L267 EN**: Comment documents intent or context: `newUnitMap.LookUpOrCreate(FORTRAN_ERROR_UNIT, terminator, wasExtant)};`.
  **L267 CN**: 注释记录了意图或上下文：`newUnitMap.LookUpOrCreate(FORTRAN_ERROR_UNIT, terminator, wasExtant)};`。
- **L268 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L268 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L269 EN**: Executes statement involving `Predefine`.
  **L269 CN**: 执行涉及 `Predefine` 的语句。
- **L270 EN**: Executes statement involving `SignalError`.
  **L270 CN**: 执行涉及 `SignalError` 的语句。

### Lines 271-288

````cpp
  error.isUnformatted = false;
  errorOutput = &error;

  return newUnitMap;
}

// A back-up atexit() handler for programs that don't terminate with a main
// program END or a STOP statement or other Fortran-initiated program shutdown,
// such as programs with a C main() that terminate normally.  It flushes all
// external I/O units.  It is registered once the first time that any external
// I/O is attempted.
static void CloseAllExternalUnits() {
  IoErrorHandler handler{"Fortran program termination"};
  ExternalFileUnit::CloseAll(handler);
}

UnitMap &ExternalFileUnit::GetUnitMap(const Terminator &terminator) {
  if (unitMap) {
````

- **L271 EN**: Initializes or updates `error.isUnformatted`.
  **L271 CN**: 初始化或更新 `error.isUnformatted`。
- **L272 EN**: Initializes or updates `errorOutput`.
  **L272 CN**: 初始化或更新 `errorOutput`。
- **L273 EN**: Blank line separates nearby declarations or logic blocks.
  **L273 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L274 EN**: Returns from the current function, often propagating a computed result.
  **L274 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L275 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L275 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L276 EN**: Blank line separates nearby declarations or logic blocks.
  **L276 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L277 EN**: Comment documents intent or context: `A back-up atexit() handler for programs that don't terminate with a main`.
  **L277 CN**: 注释记录了意图或上下文：`A back-up atexit() handler for programs that don't terminate with a main`。
- **L278 EN**: Comment documents intent or context: `program END or a STOP statement or other Fortran-initiated program shutdown,`.
  **L278 CN**: 注释记录了意图或上下文：`program END or a STOP statement or other Fortran-initiated program shutdown,`。
- **L279 EN**: Comment documents intent or context: `such as programs with a C main() that terminate normally. It flushes all`.
  **L279 CN**: 注释记录了意图或上下文：`such as programs with a C main() that terminate normally. It flushes all`。
- **L280 EN**: Comment documents intent or context: `external I/O units. It is registered once the first time that any external`.
  **L280 CN**: 注释记录了意图或上下文：`external I/O units. It is registered once the first time that any external`。
- **L281 EN**: Comment documents intent or context: `I/O is attempted.`.
  **L281 CN**: 注释记录了意图或上下文：`I/O is attempted.`。
- **L282 EN**: Declares or defines callable `CloseAllExternalUnits`.
  **L282 CN**: 声明或定义可调用实体 `CloseAllExternalUnits`。
- **L283 EN**: Executes statement `IoErrorHandler handler{"Fortran program termination"};`.
  **L283 CN**: 执行语句 `IoErrorHandler handler{"Fortran program termination"};`。
- **L284 EN**: Executes statement involving `CloseAll`.
  **L284 CN**: 执行涉及 `CloseAll` 的语句。
- **L285 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L285 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L286 EN**: Blank line separates nearby declarations or logic blocks.
  **L286 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L287 EN**: Declares or defines callable `GetUnitMap`.
  **L287 CN**: 声明或定义可调用实体 `GetUnitMap`。
- **L288 EN**: Introduces conditional control flow with an `if` statement.
  **L288 CN**: 通过 `if` 语句引入条件控制流。

### Lines 289-306

````cpp
    return *unitMap;
  }
  {
    CriticalSection critical{unitMapLock};
    if (unitMap) {
      return *unitMap;
    }
    unitMap = &CreateUnitMap(terminator);
  }
  std::atexit(CloseAllExternalUnits);
  return *unitMap;
}

void ExternalFileUnit::CloseAll(IoErrorHandler &handler) {
  CriticalSection critical{unitMapLock};
  if (unitMap) {
    unitMap->CloseAll(handler);
    FreeMemoryAndNullify(unitMap);
````

- **L289 EN**: Returns from the current function, often propagating a computed result.
  **L289 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L290 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L290 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L291 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L291 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L292 EN**: Executes statement `CriticalSection critical{unitMapLock};`.
  **L292 CN**: 执行语句 `CriticalSection critical{unitMapLock};`。
- **L293 EN**: Introduces conditional control flow with an `if` statement.
  **L293 CN**: 通过 `if` 语句引入条件控制流。
- **L294 EN**: Returns from the current function, often propagating a computed result.
  **L294 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L295 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L295 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L296 EN**: Initializes or updates `unitMap`.
  **L296 CN**: 初始化或更新 `unitMap`。
- **L297 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L297 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L298 EN**: Executes statement involving `atexit`.
  **L298 CN**: 执行涉及 `atexit` 的语句。
- **L299 EN**: Returns from the current function, often propagating a computed result.
  **L299 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L300 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L300 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L301 EN**: Blank line separates nearby declarations or logic blocks.
  **L301 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L302 EN**: Declares or defines callable `CloseAll`.
  **L302 CN**: 声明或定义可调用实体 `CloseAll`。
- **L303 EN**: Executes statement `CriticalSection critical{unitMapLock};`.
  **L303 CN**: 执行语句 `CriticalSection critical{unitMapLock};`。
- **L304 EN**: Introduces conditional control flow with an `if` statement.
  **L304 CN**: 通过 `if` 语句引入条件控制流。
- **L305 EN**: Executes statement involving `CloseAll`.
  **L305 CN**: 执行涉及 `CloseAll` 的语句。
- **L306 EN**: Executes statement involving `FreeMemoryAndNullify`.
  **L306 CN**: 执行涉及 `FreeMemoryAndNullify` 的语句。

### Lines 307-324

````cpp
  }
  defaultOutput = nullptr;
  defaultInput = nullptr;
  errorOutput = nullptr;
}

void ExternalFileUnit::FlushAll(IoErrorHandler &handler) {
  CriticalSection critical{unitMapLock};
  if (unitMap) {
    unitMap->FlushAll(handler);
  }
}

int ExternalFileUnit::GetAsynchronousId(IoErrorHandler &handler) {
  if (!mayAsynchronous()) {
    handler.SignalError(IostatBadAsynchronous);
    return -1;
  } else {
````

- **L307 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L307 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L308 EN**: Initializes or updates `defaultOutput`.
  **L308 CN**: 初始化或更新 `defaultOutput`。
- **L309 EN**: Initializes or updates `defaultInput`.
  **L309 CN**: 初始化或更新 `defaultInput`。
- **L310 EN**: Initializes or updates `errorOutput`.
  **L310 CN**: 初始化或更新 `errorOutput`。
- **L311 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L311 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L312 EN**: Blank line separates nearby declarations or logic blocks.
  **L312 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L313 EN**: Declares or defines callable `FlushAll`.
  **L313 CN**: 声明或定义可调用实体 `FlushAll`。
- **L314 EN**: Executes statement `CriticalSection critical{unitMapLock};`.
  **L314 CN**: 执行语句 `CriticalSection critical{unitMapLock};`。
- **L315 EN**: Introduces conditional control flow with an `if` statement.
  **L315 CN**: 通过 `if` 语句引入条件控制流。
- **L316 EN**: Executes statement involving `FlushAll`.
  **L316 CN**: 执行涉及 `FlushAll` 的语句。
- **L317 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L317 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L318 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L318 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L319 EN**: Blank line separates nearby declarations or logic blocks.
  **L319 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L320 EN**: Declares or defines callable `GetAsynchronousId`.
  **L320 CN**: 声明或定义可调用实体 `GetAsynchronousId`。
- **L321 EN**: Introduces conditional control flow with an `if` statement.
  **L321 CN**: 通过 `if` 语句引入条件控制流。
- **L322 EN**: Executes statement involving `SignalError`.
  **L322 CN**: 执行涉及 `SignalError` 的语句。
- **L323 EN**: Returns from the current function, often propagating a computed result.
  **L323 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L324 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L324 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 325-342

````cpp
    for (int j{0}; 64 * j < maxAsyncIds; ++j) {
      if (auto least{asyncIdAvailable_[j].LeastElement()}) {
        asyncIdAvailable_[j].reset(*least);
        return 64 * j + static_cast<int>(*least);
      }
    }
    handler.SignalError(IostatTooManyAsyncOps);
    return -1;
  }
}

bool ExternalFileUnit::Wait(int id) {
  if (static_cast<std::size_t>(id) >= maxAsyncIds ||
      asyncIdAvailable_[id / 64].test(id % 64)) {
    return false;
  } else {
    if (id == 0) { // means "all IDs"
      for (int j{0}; 64 * j < maxAsyncIds; ++j) {
````

- **L325 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L325 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L326 EN**: Introduces conditional control flow with an `if` statement.
  **L326 CN**: 通过 `if` 语句引入条件控制流。
- **L327 EN**: Executes statement involving `reset`.
  **L327 CN**: 执行涉及 `reset` 的语句。
- **L328 EN**: Returns from the current function, often propagating a computed result.
  **L328 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L329 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L329 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L330 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L330 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L331 EN**: Executes statement involving `SignalError`.
  **L331 CN**: 执行涉及 `SignalError` 的语句。
- **L332 EN**: Returns from the current function, often propagating a computed result.
  **L332 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L333 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L333 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L334 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L334 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L335 EN**: Blank line separates nearby declarations or logic blocks.
  **L335 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L336 EN**: Declares or defines callable `Wait`.
  **L336 CN**: 声明或定义可调用实体 `Wait`。
- **L337 EN**: Introduces conditional control flow with an `if` statement.
  **L337 CN**: 通过 `if` 语句引入条件控制流。
- **L338 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L338 CN**: 延续周围的声明、表达式或控制流结构。
- **L339 EN**: Returns from the current function, often propagating a computed result.
  **L339 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L340 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L340 CN**: 延续周围的声明、表达式或控制流结构。
- **L341 EN**: Introduces conditional control flow with an `if` statement.
  **L341 CN**: 通过 `if` 语句引入条件控制流。
- **L342 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L342 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。

### Lines 343-354

````cpp
        asyncIdAvailable_[j].set();
      }
      asyncIdAvailable_[0].reset(0);
    } else {
      asyncIdAvailable_[id / 64].set(id % 64);
    }
    return true;
  }
}

} // namespace Fortran::runtime::io
#endif // !defined(RT_USE_PSEUDO_FILE_UNIT)
````

- **L343 EN**: Executes statement involving `set`.
  **L343 CN**: 执行涉及 `set` 的语句。
- **L344 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L344 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L345 EN**: Executes statement involving `reset`.
  **L345 CN**: 执行涉及 `reset` 的语句。
- **L346 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L346 CN**: 延续周围的声明、表达式或控制流结构。
- **L347 EN**: Executes statement involving `set`.
  **L347 CN**: 执行涉及 `set` 的语句。
- **L348 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L348 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L349 EN**: Returns from the current function, often propagating a computed result.
  **L349 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L350 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L350 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L351 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L351 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L352 EN**: Blank line separates nearby declarations or logic blocks.
  **L352 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L353 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L353 CN**: 延续周围的声明、表达式或控制流结构。
- **L354 EN**: Preprocessor directive manages conditional compilation or macros: `#endif // !defined(RT_USE_PSEUDO_FILE_UNIT)`.
  **L354 CN**: 预处理指令管理条件编译或宏：`#endif // !defined(RT_USE_PSEUDO_FILE_UNIT)`。

## Key Concepts / 关键概念

- **Scale / 规模**: The file contains approximately 354 source lines, which suggests a medium-sized implementation unit. / 该文件约有 354 行源码，说明它是一个中等规模的实现单元。
- **Fortran execution semantics / Fortran 执行语义**: The implementation materializes language-level behavior such as descriptors, I/O state, allocation, reductions, and error handling. / 实现把语言层面的行为具体化，例如描述符、I/O 状态、分配、归约与错误处理。
- **Runtime layering / 运行时分层**: Source files in this area usually bridge generated code, public runtime headers, and lower-level system facilities. / 此处源码通常桥接生成代码、公共运行时头文件以及更底层的系统设施。
- **Interface surface / 接口表面**: Direct includes such as `unit-map.h`, `unit.h`, `flang-rt/runtime/io-error.h`, `flang-rt/runtime/lock.h` show which nearby abstractions this file relies on first. / 直接包含的头文件（如 `unit-map.h`, `unit.h`, `flang-rt/runtime/io-error.h`, `flang-rt/runtime/lock.h`）展示了此文件首先依赖的周边抽象。
- **Primary entry points / 主要入口点**: Notable callables include `FlushOutputOnCrash`, `LookUp`, `CloseUnit`, `DestroyClosed`, `SetDirection`, `CreateUnitMap`. / 值得关注的可调用实体包括 `FlushOutputOnCrash`, `LookUp`, `CloseUnit`, `DestroyClosed`, `SetDirection`, `CreateUnitMap`。
- **Namespaces / 命名空间**: The code uses namespaces such as `Fortran` to organize symbols. / 代码使用 `Fortran` 等命名空间来组织符号。

## Dependencies / 依赖关系

- **Project headers / 项目头文件**: `unit-map.h`, `unit.h`, `flang-rt/runtime/io-error.h`, `flang-rt/runtime/lock.h`, `flang-rt/runtime/tools.h`. These provide subsystem-specific declarations. / 这些头文件提供子系统专用声明。
- **Standard or platform headers / 标准库或平台头文件**: `cstdio`, `limits`. They connect the file to language-runtime or OS services. / 它们将该文件连接到语言运行时或操作系统服务。
- **Callable surface / 可调用表面**: Functions or methods defined here include `FlushOutputOnCrash`, `LookUp`, `CloseUnit`, `DestroyClosed`, `SetDirection`, `CreateUnitMap`, `CloseAllExternalUnits`, `GetUnitMap`, `CloseAll`, `FlushAll`. These are likely the main entry points exported to nearby code. / 此处定义的函数或方法包括 `FlushOutputOnCrash`, `LookUp`, `CloseUnit`, `DestroyClosed`, `SetDirection`, `CreateUnitMap`, `CloseAllExternalUnits`, `GetUnitMap`, `CloseAll`, `FlushAll`，它们通常是对周边代码暴露的主要入口。
