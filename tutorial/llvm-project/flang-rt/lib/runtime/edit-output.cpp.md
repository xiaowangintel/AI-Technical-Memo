# edit-output.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `flang-rt/lib/runtime/edit-output.cpp` | `flang-rt/lib/runtime/edit-output.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements the Flang runtime library, including descriptors, I/O, memory management, numerics, and execution support. This file centers on `edit output`. | 实现 Flang 运行时库，包括描述符、I/O、内存管理、数值计算与执行支持。 本文件聚焦于 `edit output`。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/runtime/edit-output.cpp -----------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "edit-output.h"
#include "flang-rt/runtime/emit-encoded.h"
#include "flang-rt/runtime/utf.h"
#include "flang/Common/real.h"
#include "flang/Common/uint128.h"
#include <algorithm>

namespace Fortran::runtime::io {
RT_OFFLOAD_API_GROUP_BEGIN

// In output statement, add a space between numbers and characters.
static RT_API_ATTRS void AddSpaceBeforeCharacter(IoStatementState &io) {
  if (auto *list{io.get_if<ListDirectedStatementState<Direction::Output>>()}) {
    list->set_lastWasUndelimitedCharacter(false);
  }
}
````

- **L1 EN**: Comment documents intent or context: `lib/runtime/edit-output.cpp -----------------------------*- C++ -*-===//`.
  **L1 CN**: 注释记录了意图或上下文：`lib/runtime/edit-output.cpp -----------------------------*- C++ -*-===//`。
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
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes `edit-output.h` to access project-local declarations and helper interfaces.
  **L9 CN**: 引入 `edit-output.h` 以使用 项目内声明与辅助接口。
- **L10 EN**: Includes `flang-rt/runtime/emit-encoded.h` to access Flang runtime public headers.
  **L10 CN**: 引入 `flang-rt/runtime/emit-encoded.h` 以使用 Flang 运行时公共头文件。
- **L11 EN**: Includes `flang-rt/runtime/utf.h` to access Flang runtime public headers.
  **L11 CN**: 引入 `flang-rt/runtime/utf.h` 以使用 Flang 运行时公共头文件。
- **L12 EN**: Includes `flang/Common/real.h` to access Flang common data structures and compiler-wide helpers.
  **L12 CN**: 引入 `flang/Common/real.h` 以使用 Flang 通用数据结构与编译器级辅助工具。
- **L13 EN**: Includes `flang/Common/uint128.h` to access Flang common data structures and compiler-wide helpers.
  **L13 CN**: 引入 `flang/Common/uint128.h` 以使用 Flang 通用数据结构与编译器级辅助工具。
- **L14 EN**: Includes `algorithm` to access standard algorithms and helpers.
  **L14 CN**: 引入 `algorithm` 以使用 标准算法与辅助工具。
- **L15 EN**: Blank line separates nearby declarations or logic blocks.
  **L15 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L16 EN**: Enters namespace `Fortran` to scope related declarations.
  **L16 CN**: 进入命名空间 `Fortran` 以组织相关声明。
- **L17 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L17 CN**: 延续周围的声明、表达式或控制流结构。
- **L18 EN**: Blank line separates nearby declarations or logic blocks.
  **L18 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L19 EN**: Comment documents intent or context: `In output statement, add a space between numbers and characters.`.
  **L19 CN**: 注释记录了意图或上下文：`In output statement, add a space between numbers and characters.`。
- **L20 EN**: Declares or defines callable `AddSpaceBeforeCharacter`.
  **L20 CN**: 声明或定义可调用实体 `AddSpaceBeforeCharacter`。
- **L21 EN**: Introduces conditional control flow with an `if` statement.
  **L21 CN**: 通过 `if` 语句引入条件控制流。
- **L22 EN**: Executes statement involving `set_lastWasUndelimitedCharacter`.
  **L22 CN**: 执行涉及 `set_lastWasUndelimitedCharacter` 的语句。
- **L23 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L23 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L24 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L24 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 25-48

````cpp

// B/O/Z output of arbitrarily sized data emits a binary/octal/hexadecimal
// representation of what is interpreted to be a single unsigned integer value.
// When used with character data, endianness is exposed.
template <int LOG2_BASE>
static RT_API_ATTRS bool EditBOZOutput(IoStatementState &io,
    const DataEdit &edit, const unsigned char *data0, std::size_t bytes) {
  AddSpaceBeforeCharacter(io);
  int digits{static_cast<int>((bytes * 8) / LOG2_BASE)};
  int get{static_cast<int>(bytes * 8) - digits * LOG2_BASE};
  if (get > 0) {
    ++digits;
  } else {
    get = LOG2_BASE;
  }
  int shift{7};
  int increment{isHostLittleEndian ? -1 : 1};
  const unsigned char *data{data0 + (isHostLittleEndian ? bytes - 1 : 0)};
  int skippedZeroes{0};
  int digit{0};
  // The same algorithm is used to generate digits for real (below)
  // as well as for generating them only to skip leading zeroes (here).
  // Bits are copied one at a time from the source data.
  // TODO: Multiple bit copies for hexadecimal, where misalignment
````

- **L25 EN**: Blank line separates nearby declarations or logic blocks.
  **L25 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L26 EN**: Comment documents intent or context: `B/O/Z output of arbitrarily sized data emits a binary/octal/hexadecimal`.
  **L26 CN**: 注释记录了意图或上下文：`B/O/Z output of arbitrarily sized data emits a binary/octal/hexadecimal`。
- **L27 EN**: Comment documents intent or context: `representation of what is interpreted to be a single unsigned integer value.`.
  **L27 CN**: 注释记录了意图或上下文：`representation of what is interpreted to be a single unsigned integer value.`。
- **L28 EN**: Comment documents intent or context: `When used with character data, endianness is exposed.`.
  **L28 CN**: 注释记录了意图或上下文：`When used with character data, endianness is exposed.`。
- **L29 EN**: Begins a template declaration parameterizing subsequent code.
  **L29 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L30 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L30 CN**: 延续周围的声明、表达式或控制流结构。
- **L31 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L31 CN**: 延续周围的声明、表达式或控制流结构。
- **L32 EN**: Executes statement involving `AddSpaceBeforeCharacter`.
  **L32 CN**: 执行涉及 `AddSpaceBeforeCharacter` 的语句。
- **L33 EN**: Executes statement `int digits{static_cast<int>((bytes * 8) / LOG2_BASE)};`.
  **L33 CN**: 执行语句 `int digits{static_cast<int>((bytes * 8) / LOG2_BASE)};`。
- **L34 EN**: Executes statement `int get{static_cast<int>(bytes * 8) - digits * LOG2_BASE};`.
  **L34 CN**: 执行语句 `int get{static_cast<int>(bytes * 8) - digits * LOG2_BASE};`。
- **L35 EN**: Introduces conditional control flow with an `if` statement.
  **L35 CN**: 通过 `if` 语句引入条件控制流。
- **L36 EN**: Executes statement `++digits;`.
  **L36 CN**: 执行语句 `++digits;`。
- **L37 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L37 CN**: 延续周围的声明、表达式或控制流结构。
- **L38 EN**: Initializes or updates `get`.
  **L38 CN**: 初始化或更新 `get`。
- **L39 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L39 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L40 EN**: Executes statement `int shift{7};`.
  **L40 CN**: 执行语句 `int shift{7};`。
- **L41 EN**: Executes statement `int increment{isHostLittleEndian ? -1 : 1};`.
  **L41 CN**: 执行语句 `int increment{isHostLittleEndian ? -1 : 1};`。
- **L42 EN**: Executes statement `const unsigned char *data{data0 + (isHostLittleEndian ? bytes - 1 : 0)};`.
  **L42 CN**: 执行语句 `const unsigned char *data{data0 + (isHostLittleEndian ? bytes - 1 : 0)};`。
- **L43 EN**: Executes statement `int skippedZeroes{0};`.
  **L43 CN**: 执行语句 `int skippedZeroes{0};`。
- **L44 EN**: Executes statement `int digit{0};`.
  **L44 CN**: 执行语句 `int digit{0};`。
- **L45 EN**: Comment documents intent or context: `The same algorithm is used to generate digits for real (below)`.
  **L45 CN**: 注释记录了意图或上下文：`The same algorithm is used to generate digits for real (below)`。
- **L46 EN**: Comment documents intent or context: `as well as for generating them only to skip leading zeroes (here).`.
  **L46 CN**: 注释记录了意图或上下文：`as well as for generating them only to skip leading zeroes (here).`。
- **L47 EN**: Comment documents intent or context: `Bits are copied one at a time from the source data.`.
  **L47 CN**: 注释记录了意图或上下文：`Bits are copied one at a time from the source data.`。
- **L48 EN**: Comment documents intent or context: `TODO: Multiple bit copies for hexadecimal, where misalignment`.
  **L48 CN**: 注释记录了意图或上下文：`TODO: Multiple bit copies for hexadecimal, where misalignment`。

### Lines 49-72

````cpp
  // is not possible; or for octal when all 3 bits come from the
  // same byte.
  while (bytes > 0) {
    if (get == 0) {
      if (digit != 0) {
        break; // first nonzero leading digit
      }
      ++skippedZeroes;
      get = LOG2_BASE;
    } else if (shift < 0) {
      data += increment;
      --bytes;
      shift = 7;
    } else {
      digit = 2 * digit + ((*data >> shift--) & 1);
      --get;
    }
  }
  // Emit leading spaces and zeroes; detect field overflow
  int leadingZeroes{0};
  int editWidth{edit.width.value_or(0)};
  int significant{digits - skippedZeroes};
  if (edit.digits && significant <= *edit.digits) { // Bw.m, Ow.m, Zw.m
    if (*edit.digits == 0 && bytes == 0) {
````

- **L49 EN**: Comment documents intent or context: `is not possible; or for octal when all 3 bits come from the`.
  **L49 CN**: 注释记录了意图或上下文：`is not possible; or for octal when all 3 bits come from the`。
- **L50 EN**: Comment documents intent or context: `same byte.`.
  **L50 CN**: 注释记录了意图或上下文：`same byte.`。
- **L51 EN**: Starts a `while` loop controlled by a runtime condition.
  **L51 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L52 EN**: Introduces conditional control flow with an `if` statement.
  **L52 CN**: 通过 `if` 语句引入条件控制流。
- **L53 EN**: Introduces conditional control flow with an `if` statement.
  **L53 CN**: 通过 `if` 语句引入条件控制流。
- **L54 EN**: Breaks out of the current loop or switch.
  **L54 CN**: 跳出当前循环或 switch。
- **L55 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L55 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L56 EN**: Executes statement `++skippedZeroes;`.
  **L56 CN**: 执行语句 `++skippedZeroes;`。
- **L57 EN**: Initializes or updates `get`.
  **L57 CN**: 初始化或更新 `get`。
- **L58 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L58 CN**: 延续周围的声明、表达式或控制流结构。
- **L59 EN**: Initializes or updates `+`.
  **L59 CN**: 初始化或更新 `+`。
- **L60 EN**: Executes statement `--bytes;`.
  **L60 CN**: 执行语句 `--bytes;`。
- **L61 EN**: Initializes or updates `shift`.
  **L61 CN**: 初始化或更新 `shift`。
- **L62 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L62 CN**: 延续周围的声明、表达式或控制流结构。
- **L63 EN**: Initializes or updates `digit`.
  **L63 CN**: 初始化或更新 `digit`。
- **L64 EN**: Executes statement `--get;`.
  **L64 CN**: 执行语句 `--get;`。
- **L65 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L65 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L66 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L66 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L67 EN**: Comment documents intent or context: `Emit leading spaces and zeroes; detect field overflow`.
  **L67 CN**: 注释记录了意图或上下文：`Emit leading spaces and zeroes; detect field overflow`。
- **L68 EN**: Executes statement `int leadingZeroes{0};`.
  **L68 CN**: 执行语句 `int leadingZeroes{0};`。
- **L69 EN**: Executes statement involving `value_or`.
  **L69 CN**: 执行涉及 `value_or` 的语句。
- **L70 EN**: Executes statement `int significant{digits - skippedZeroes};`.
  **L70 CN**: 执行语句 `int significant{digits - skippedZeroes};`。
- **L71 EN**: Introduces conditional control flow with an `if` statement.
  **L71 CN**: 通过 `if` 语句引入条件控制流。
- **L72 EN**: Introduces conditional control flow with an `if` statement.
  **L72 CN**: 通过 `if` 语句引入条件控制流。

### Lines 73-96

````cpp
      editWidth = std::max(1, editWidth);
    } else {
      leadingZeroes = *edit.digits - significant;
    }
  } else if (bytes == 0) {
    leadingZeroes = 1;
  }
  int subTotal{leadingZeroes + significant};
  int leadingSpaces{std::max(0, editWidth - subTotal)};
  if (editWidth > 0 && leadingSpaces + subTotal > editWidth) {
    return EmitRepeated(io, '*', editWidth);
  }
  if (!(EmitRepeated(io, ' ', leadingSpaces) &&
          EmitRepeated(io, '0', leadingZeroes))) {
    return false;
  }
  // Emit remaining digits
  while (bytes > 0) {
    if (get == 0) {
      char ch{static_cast<char>(digit >= 10 ? 'A' + digit - 10 : '0' + digit)};
      if (!EmitAscii(io, &ch, 1)) {
        return false;
      }
      get = LOG2_BASE;
````

- **L73 EN**: Initializes or updates `editWidth`.
  **L73 CN**: 初始化或更新 `editWidth`。
- **L74 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L74 CN**: 延续周围的声明、表达式或控制流结构。
- **L75 EN**: Initializes or updates `leadingZeroes`.
  **L75 CN**: 初始化或更新 `leadingZeroes`。
- **L76 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L76 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L77 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L77 CN**: 延续周围的声明、表达式或控制流结构。
- **L78 EN**: Initializes or updates `leadingZeroes`.
  **L78 CN**: 初始化或更新 `leadingZeroes`。
- **L79 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L79 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L80 EN**: Executes statement `int subTotal{leadingZeroes + significant};`.
  **L80 CN**: 执行语句 `int subTotal{leadingZeroes + significant};`。
- **L81 EN**: Executes statement involving `max`.
  **L81 CN**: 执行涉及 `max` 的语句。
- **L82 EN**: Introduces conditional control flow with an `if` statement.
  **L82 CN**: 通过 `if` 语句引入条件控制流。
- **L83 EN**: Returns from the current function, often propagating a computed result.
  **L83 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L84 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L84 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L85 EN**: Introduces conditional control flow with an `if` statement.
  **L85 CN**: 通过 `if` 语句引入条件控制流。
- **L86 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L86 CN**: 延续周围的声明、表达式或控制流结构。
- **L87 EN**: Returns from the current function, often propagating a computed result.
  **L87 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L88 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L88 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L89 EN**: Comment documents intent or context: `Emit remaining digits`.
  **L89 CN**: 注释记录了意图或上下文：`Emit remaining digits`。
- **L90 EN**: Starts a `while` loop controlled by a runtime condition.
  **L90 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L91 EN**: Introduces conditional control flow with an `if` statement.
  **L91 CN**: 通过 `if` 语句引入条件控制流。
- **L92 EN**: Executes statement `char ch{static_cast<char>(digit >= 10 ? 'A' + digit - 10 : '0' + digit)};`.
  **L92 CN**: 执行语句 `char ch{static_cast<char>(digit >= 10 ? 'A' + digit - 10 : '0' + digit)};`。
- **L93 EN**: Introduces conditional control flow with an `if` statement.
  **L93 CN**: 通过 `if` 语句引入条件控制流。
- **L94 EN**: Returns from the current function, often propagating a computed result.
  **L94 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L95 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L95 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L96 EN**: Initializes or updates `get`.
  **L96 CN**: 初始化或更新 `get`。

### Lines 97-120

````cpp
      digit = 0;
    } else if (shift < 0) {
      data += increment;
      --bytes;
      shift = 7;
    } else {
      digit = 2 * digit + ((*data >> shift--) & 1);
      --get;
    }
  }
  return true;
}

template <int KIND>
bool RT_API_ATTRS EditIntegerOutput(IoStatementState &io, const DataEdit &edit,
    common::HostSignedIntType<8 * KIND> n, bool isSigned) {
  AddSpaceBeforeCharacter(io);
  switch (edit.descriptor) {
  case DataEdit::ListDirected:
  case 'G':
  case 'I':
    break;
  case 'B':
    return EditBOZOutput<1>(
````

- **L97 EN**: Initializes or updates `digit`.
  **L97 CN**: 初始化或更新 `digit`。
- **L98 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L98 CN**: 延续周围的声明、表达式或控制流结构。
- **L99 EN**: Initializes or updates `+`.
  **L99 CN**: 初始化或更新 `+`。
- **L100 EN**: Executes statement `--bytes;`.
  **L100 CN**: 执行语句 `--bytes;`。
- **L101 EN**: Initializes or updates `shift`.
  **L101 CN**: 初始化或更新 `shift`。
- **L102 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L102 CN**: 延续周围的声明、表达式或控制流结构。
- **L103 EN**: Initializes or updates `digit`.
  **L103 CN**: 初始化或更新 `digit`。
- **L104 EN**: Executes statement `--get;`.
  **L104 CN**: 执行语句 `--get;`。
- **L105 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L105 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L106 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L106 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L107 EN**: Returns from the current function, often propagating a computed result.
  **L107 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L108 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L108 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L109 EN**: Blank line separates nearby declarations or logic blocks.
  **L109 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L110 EN**: Begins a template declaration parameterizing subsequent code.
  **L110 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L111 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L111 CN**: 延续周围的声明、表达式或控制流结构。
- **L112 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L112 CN**: 延续周围的声明、表达式或控制流结构。
- **L113 EN**: Executes statement involving `AddSpaceBeforeCharacter`.
  **L113 CN**: 执行涉及 `AddSpaceBeforeCharacter` 的语句。
- **L114 EN**: Begins a `switch` dispatch over discrete cases.
  **L114 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L115 EN**: Marks one `switch` case label.
  **L115 CN**: 标记一个 `switch` 的 case 标签。
- **L116 EN**: Marks one `switch` case label.
  **L116 CN**: 标记一个 `switch` 的 case 标签。
- **L117 EN**: Marks one `switch` case label.
  **L117 CN**: 标记一个 `switch` 的 case 标签。
- **L118 EN**: Breaks out of the current loop or switch.
  **L118 CN**: 跳出当前循环或 switch。
- **L119 EN**: Marks one `switch` case label.
  **L119 CN**: 标记一个 `switch` 的 case 标签。
- **L120 EN**: Returns from the current function, often propagating a computed result.
  **L120 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 121-144

````cpp
        io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);
  case 'O':
    return EditBOZOutput<3>(
        io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);
  case 'Z':
    return EditBOZOutput<4>(
        io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);
  case 'L':
    return EditLogicalOutput(io, edit, n != 0 ? true : false);
  case 'A': // legacy extension
    return EditCharacterOutput(
        io, edit, reinterpret_cast<char *>(&n), sizeof n);
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInFormat,
        "Data edit descriptor '%c' may not be used with an INTEGER data item",
        edit.descriptor);
    return false;
  }
  char buffer[130], *end{&buffer[sizeof buffer]}, *p{end};
  bool isNegative{isSigned && n < 0};
  using Unsigned = common::HostUnsignedIntType<8 * KIND>;
  Unsigned un{static_cast<Unsigned>(n)};
  int signChars{0};
  if (isNegative) {
````

- **L121 EN**: Executes statement `io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);`.
  **L121 CN**: 执行语句 `io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);`。
- **L122 EN**: Marks one `switch` case label.
  **L122 CN**: 标记一个 `switch` 的 case 标签。
- **L123 EN**: Returns from the current function, often propagating a computed result.
  **L123 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L124 EN**: Executes statement `io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);`.
  **L124 CN**: 执行语句 `io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);`。
- **L125 EN**: Marks one `switch` case label.
  **L125 CN**: 标记一个 `switch` 的 case 标签。
- **L126 EN**: Returns from the current function, often propagating a computed result.
  **L126 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L127 EN**: Executes statement `io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);`.
  **L127 CN**: 执行语句 `io, edit, reinterpret_cast<const unsigned char *>(&n), KIND);`。
- **L128 EN**: Marks one `switch` case label.
  **L128 CN**: 标记一个 `switch` 的 case 标签。
- **L129 EN**: Returns from the current function, often propagating a computed result.
  **L129 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L130 EN**: Marks one `switch` case label.
  **L130 CN**: 标记一个 `switch` 的 case 标签。
- **L131 EN**: Returns from the current function, often propagating a computed result.
  **L131 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L132 EN**: Executes statement `io, edit, reinterpret_cast<char *>(&n), sizeof n);`.
  **L132 CN**: 执行语句 `io, edit, reinterpret_cast<char *>(&n), sizeof n);`。
- **L133 EN**: Provides the default branch for a `switch` statement.
  **L133 CN**: 为 `switch` 语句提供默认分支。
- **L134 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L134 CN**: 延续周围的声明、表达式或控制流结构。
- **L135 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L135 CN**: 延续周围的声明、表达式或控制流结构。
- **L136 EN**: Executes statement `edit.descriptor);`.
  **L136 CN**: 执行语句 `edit.descriptor);`。
- **L137 EN**: Returns from the current function, often propagating a computed result.
  **L137 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L138 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L138 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L139 EN**: Executes statement `char buffer[130], *end{&buffer[sizeof buffer]}, *p{end};`.
  **L139 CN**: 执行语句 `char buffer[130], *end{&buffer[sizeof buffer]}, *p{end};`。
- **L140 EN**: Executes statement `bool isNegative{isSigned && n < 0};`.
  **L140 CN**: 执行语句 `bool isNegative{isSigned && n < 0};`。
- **L141 EN**: Defines type alias `Unsigned` for readability or ABI convenience.
  **L141 CN**: 定义类型别名 `Unsigned`，以提升可读性或满足 ABI 便利性。
- **L142 EN**: Executes statement `Unsigned un{static_cast<Unsigned>(n)};`.
  **L142 CN**: 执行语句 `Unsigned un{static_cast<Unsigned>(n)};`。
- **L143 EN**: Executes statement `int signChars{0};`.
  **L143 CN**: 执行语句 `int signChars{0};`。
- **L144 EN**: Introduces conditional control flow with an `if` statement.
  **L144 CN**: 通过 `if` 语句引入条件控制流。

### Lines 145-168

````cpp
    un = -un;
  }
  if (isNegative || (edit.modes.editingFlags & signPlus)) {
    signChars = 1; // '-' or '+'
  }
  while (un > 0) {
    auto quotient{un / 10u};
    *--p = '0' + static_cast<int>(un - Unsigned{10} * quotient);
    un = quotient;
  }
  int digits = end - p;
  int leadingZeroes{0};
  int editWidth{edit.width.value_or(0)};
  if (edit.descriptor == 'I' && edit.digits && digits <= *edit.digits) {
    // Only Iw.m can produce leading zeroes, not Gw.d (F'202X 13.7.5.2.2)
    if (*edit.digits == 0 && n == 0) {
      // Iw.0 with zero value: output field must be blank.  For I0.0
      // and a zero value, emit one blank character.
      signChars = 0; // in case of SP
      editWidth = std::max(1, editWidth);
    } else {
      leadingZeroes = *edit.digits - digits;
    }
  } else if (n == 0) {
````

- **L145 EN**: Initializes or updates `un`.
  **L145 CN**: 初始化或更新 `un`。
- **L146 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L146 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L147 EN**: Introduces conditional control flow with an `if` statement.
  **L147 CN**: 通过 `if` 语句引入条件控制流。
- **L148 EN**: Initializes or updates `signChars`.
  **L148 CN**: 初始化或更新 `signChars`。
- **L149 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L149 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L150 EN**: Starts a `while` loop controlled by a runtime condition.
  **L150 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L151 EN**: Executes statement `auto quotient{un / 10u};`.
  **L151 CN**: 执行语句 `auto quotient{un / 10u};`。
- **L152 EN**: Comment documents intent or context: `--p = '0' + static_cast<int>(un - Unsigned{10} * quotient);`.
  **L152 CN**: 注释记录了意图或上下文：`--p = '0' + static_cast<int>(un - Unsigned{10} * quotient);`。
- **L153 EN**: Initializes or updates `un`.
  **L153 CN**: 初始化或更新 `un`。
- **L154 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L154 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L155 EN**: Initializes or updates `digits`.
  **L155 CN**: 初始化或更新 `digits`。
- **L156 EN**: Executes statement `int leadingZeroes{0};`.
  **L156 CN**: 执行语句 `int leadingZeroes{0};`。
- **L157 EN**: Executes statement involving `value_or`.
  **L157 CN**: 执行涉及 `value_or` 的语句。
- **L158 EN**: Introduces conditional control flow with an `if` statement.
  **L158 CN**: 通过 `if` 语句引入条件控制流。
- **L159 EN**: Comment documents intent or context: `Only Iw.m can produce leading zeroes, not Gw.d (F'202X 13.7.5.2.2)`.
  **L159 CN**: 注释记录了意图或上下文：`Only Iw.m can produce leading zeroes, not Gw.d (F'202X 13.7.5.2.2)`。
- **L160 EN**: Introduces conditional control flow with an `if` statement.
  **L160 CN**: 通过 `if` 语句引入条件控制流。
- **L161 EN**: Comment documents intent or context: `Iw.0 with zero value: output field must be blank. For I0.0`.
  **L161 CN**: 注释记录了意图或上下文：`Iw.0 with zero value: output field must be blank. For I0.0`。
- **L162 EN**: Comment documents intent or context: `and a zero value, emit one blank character.`.
  **L162 CN**: 注释记录了意图或上下文：`and a zero value, emit one blank character.`。
- **L163 EN**: Initializes or updates `signChars`.
  **L163 CN**: 初始化或更新 `signChars`。
- **L164 EN**: Initializes or updates `editWidth`.
  **L164 CN**: 初始化或更新 `editWidth`。
- **L165 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L165 CN**: 延续周围的声明、表达式或控制流结构。
- **L166 EN**: Initializes or updates `leadingZeroes`.
  **L166 CN**: 初始化或更新 `leadingZeroes`。
- **L167 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L167 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L168 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L168 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 169-192

````cpp
    leadingZeroes = 1;
  }
  int subTotal{signChars + leadingZeroes + digits};
  int leadingSpaces{std::max(0, editWidth - subTotal)};
  if (editWidth > 0 && leadingSpaces + subTotal > editWidth) {
    return EmitRepeated(io, '*', editWidth);
  }
  if (edit.IsListDirected()) {
    int total{std::max(leadingSpaces, 1) + subTotal};
    if (io.GetConnectionState().NeedAdvance(static_cast<std::size_t>(total))) {
      if (!io.AdvanceRecord()) {
        return false;
      }
    }
    leadingSpaces = 1;
  } else if (!edit.width) {
    // Bare 'I' and 'G' are interpreted with various default widths in the
    // compilers that support them, so there's always some leading space
    // after column 1.
    if (io.GetConnectionState().positionInRecord > 0) {
      leadingSpaces = 1;
    }
  }
  return EmitRepeated(io, ' ', leadingSpaces) &&
````

- **L169 EN**: Initializes or updates `leadingZeroes`.
  **L169 CN**: 初始化或更新 `leadingZeroes`。
- **L170 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L170 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L171 EN**: Executes statement `int subTotal{signChars + leadingZeroes + digits};`.
  **L171 CN**: 执行语句 `int subTotal{signChars + leadingZeroes + digits};`。
- **L172 EN**: Executes statement involving `max`.
  **L172 CN**: 执行涉及 `max` 的语句。
- **L173 EN**: Introduces conditional control flow with an `if` statement.
  **L173 CN**: 通过 `if` 语句引入条件控制流。
- **L174 EN**: Returns from the current function, often propagating a computed result.
  **L174 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L175 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L175 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L176 EN**: Introduces conditional control flow with an `if` statement.
  **L176 CN**: 通过 `if` 语句引入条件控制流。
- **L177 EN**: Executes statement involving `max`.
  **L177 CN**: 执行涉及 `max` 的语句。
- **L178 EN**: Introduces conditional control flow with an `if` statement.
  **L178 CN**: 通过 `if` 语句引入条件控制流。
- **L179 EN**: Introduces conditional control flow with an `if` statement.
  **L179 CN**: 通过 `if` 语句引入条件控制流。
- **L180 EN**: Returns from the current function, often propagating a computed result.
  **L180 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L181 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L181 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L182 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L182 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L183 EN**: Initializes or updates `leadingSpaces`.
  **L183 CN**: 初始化或更新 `leadingSpaces`。
- **L184 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L184 CN**: 延续周围的声明、表达式或控制流结构。
- **L185 EN**: Comment documents intent or context: `Bare 'I' and 'G' are interpreted with various default widths in the`.
  **L185 CN**: 注释记录了意图或上下文：`Bare 'I' and 'G' are interpreted with various default widths in the`。
- **L186 EN**: Comment documents intent or context: `compilers that support them, so there's always some leading space`.
  **L186 CN**: 注释记录了意图或上下文：`compilers that support them, so there's always some leading space`。
- **L187 EN**: Comment documents intent or context: `after column 1.`.
  **L187 CN**: 注释记录了意图或上下文：`after column 1.`。
- **L188 EN**: Introduces conditional control flow with an `if` statement.
  **L188 CN**: 通过 `if` 语句引入条件控制流。
- **L189 EN**: Initializes or updates `leadingSpaces`.
  **L189 CN**: 初始化或更新 `leadingSpaces`。
- **L190 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L190 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L191 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L191 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L192 EN**: Returns from the current function, often propagating a computed result.
  **L192 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 193-216

````cpp
      EmitAscii(io, n < 0 ? "-" : "+", signChars) &&
      EmitRepeated(io, '0', leadingZeroes) && EmitAscii(io, p, digits);
}

// Formats the exponent (see table 13.1 for all the cases)
RT_API_ATTRS const char *RealOutputEditingBase::FormatExponent(
    int expo, const DataEdit &edit, int &length) {
  char *eEnd{&exponent_[sizeof exponent_]};
  char *exponent{eEnd};
  for (unsigned e{static_cast<unsigned>(std::abs(expo))}; e > 0;) {
    unsigned quotient{e / 10u};
    *--exponent = '0' + e - 10 * quotient;
    e = quotient;
  }
  bool overflow{false};
  if (edit.expoDigits) {
    if (int ed{*edit.expoDigits}) { // Ew.dEe with e > 0
      overflow = exponent + ed < eEnd;
      while (exponent > exponent_ + 2 /*E+*/ && exponent + ed > eEnd) {
        *--exponent = '0';
      }
    } else if (exponent == eEnd) {
      *--exponent = '0'; // Ew.dE0 with zero-valued exponent
    }
````

- **L193 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L193 CN**: 延续周围的声明、表达式或控制流结构。
- **L194 EN**: Executes statement involving `EmitRepeated`.
  **L194 CN**: 执行涉及 `EmitRepeated` 的语句。
- **L195 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L195 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L196 EN**: Blank line separates nearby declarations or logic blocks.
  **L196 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L197 EN**: Comment documents intent or context: `Formats the exponent (see table 13.1 for all the cases)`.
  **L197 CN**: 注释记录了意图或上下文：`Formats the exponent (see table 13.1 for all the cases)`。
- **L198 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L198 CN**: 延续周围的声明、表达式或控制流结构。
- **L199 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L199 CN**: 延续周围的声明、表达式或控制流结构。
- **L200 EN**: Executes statement `char *eEnd{&exponent_[sizeof exponent_]};`.
  **L200 CN**: 执行语句 `char *eEnd{&exponent_[sizeof exponent_]};`。
- **L201 EN**: Executes statement `char *exponent{eEnd};`.
  **L201 CN**: 执行语句 `char *exponent{eEnd};`。
- **L202 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L202 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L203 EN**: Executes statement `unsigned quotient{e / 10u};`.
  **L203 CN**: 执行语句 `unsigned quotient{e / 10u};`。
- **L204 EN**: Comment documents intent or context: `--exponent = '0' + e - 10 * quotient;`.
  **L204 CN**: 注释记录了意图或上下文：`--exponent = '0' + e - 10 * quotient;`。
- **L205 EN**: Initializes or updates `e`.
  **L205 CN**: 初始化或更新 `e`。
- **L206 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L206 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L207 EN**: Executes statement `bool overflow{false};`.
  **L207 CN**: 执行语句 `bool overflow{false};`。
- **L208 EN**: Introduces conditional control flow with an `if` statement.
  **L208 CN**: 通过 `if` 语句引入条件控制流。
- **L209 EN**: Introduces conditional control flow with an `if` statement.
  **L209 CN**: 通过 `if` 语句引入条件控制流。
- **L210 EN**: Initializes or updates `overflow`.
  **L210 CN**: 初始化或更新 `overflow`。
- **L211 EN**: Starts a `while` loop controlled by a runtime condition.
  **L211 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L212 EN**: Comment documents intent or context: `--exponent = '0';`.
  **L212 CN**: 注释记录了意图或上下文：`--exponent = '0';`。
- **L213 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L213 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L214 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L214 CN**: 延续周围的声明、表达式或控制流结构。
- **L215 EN**: Comment documents intent or context: `--exponent = '0'; // Ew.dE0 with zero-valued exponent`.
  **L215 CN**: 注释记录了意图或上下文：`--exponent = '0'; // Ew.dE0 with zero-valued exponent`。
- **L216 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L216 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 217-240

````cpp
  } else if (edit.variation == 'X') {
    if (expo == 0) {
      *--exponent = '0'; // EX without Ee and zero-valued exponent
    }
  } else {
    // Ensure at least two exponent digits unless EX
    while (exponent + 2 > eEnd) {
      *--exponent = '0';
    }
  }
  *--exponent = expo < 0 ? '-' : '+';
  if (edit.variation == 'X') {
    *--exponent = 'P';
  } else if (edit.expoDigits || edit.IsListDirected() || exponent + 3 == eEnd) {
    *--exponent = edit.descriptor == 'D' ? 'D' : 'E'; // not 'G' or 'Q'
  }
  length = eEnd - exponent;
  return overflow ? nullptr : exponent;
}

RT_API_ATTRS bool RealOutputEditingBase::EmitPrefix(
    const DataEdit &edit, std::size_t length, std::size_t width) {
  if (edit.IsListDirected()) {
    int prefixLength{edit.descriptor == DataEdit::ListDirectedRealPart ? 2
````

- **L217 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L217 CN**: 延续周围的声明、表达式或控制流结构。
- **L218 EN**: Introduces conditional control flow with an `if` statement.
  **L218 CN**: 通过 `if` 语句引入条件控制流。
- **L219 EN**: Comment documents intent or context: `--exponent = '0'; // EX without Ee and zero-valued exponent`.
  **L219 CN**: 注释记录了意图或上下文：`--exponent = '0'; // EX without Ee and zero-valued exponent`。
- **L220 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L220 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L221 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L221 CN**: 延续周围的声明、表达式或控制流结构。
- **L222 EN**: Comment documents intent or context: `Ensure at least two exponent digits unless EX`.
  **L222 CN**: 注释记录了意图或上下文：`Ensure at least two exponent digits unless EX`。
- **L223 EN**: Starts a `while` loop controlled by a runtime condition.
  **L223 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L224 EN**: Comment documents intent or context: `--exponent = '0';`.
  **L224 CN**: 注释记录了意图或上下文：`--exponent = '0';`。
- **L225 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L225 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L226 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L226 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L227 EN**: Comment documents intent or context: `--exponent = expo < 0 ? '-' : '+';`.
  **L227 CN**: 注释记录了意图或上下文：`--exponent = expo < 0 ? '-' : '+';`。
- **L228 EN**: Introduces conditional control flow with an `if` statement.
  **L228 CN**: 通过 `if` 语句引入条件控制流。
- **L229 EN**: Comment documents intent or context: `--exponent = 'P';`.
  **L229 CN**: 注释记录了意图或上下文：`--exponent = 'P';`。
- **L230 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L230 CN**: 延续周围的声明、表达式或控制流结构。
- **L231 EN**: Comment documents intent or context: `--exponent = edit.descriptor == 'D' ? 'D' : 'E'; // not 'G' or 'Q'`.
  **L231 CN**: 注释记录了意图或上下文：`--exponent = edit.descriptor == 'D' ? 'D' : 'E'; // not 'G' or 'Q'`。
- **L232 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L232 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L233 EN**: Initializes or updates `length`.
  **L233 CN**: 初始化或更新 `length`。
- **L234 EN**: Returns from the current function, often propagating a computed result.
  **L234 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L235 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L235 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L236 EN**: Blank line separates nearby declarations or logic blocks.
  **L236 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L237 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L237 CN**: 延续周围的声明、表达式或控制流结构。
- **L238 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L238 CN**: 延续周围的声明、表达式或控制流结构。
- **L239 EN**: Introduces conditional control flow with an `if` statement.
  **L239 CN**: 通过 `if` 语句引入条件控制流。
- **L240 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L240 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 241-264

````cpp
            : edit.descriptor == DataEdit::ListDirectedImaginaryPart   ? 0
                                                                       : 1};
    int suffixLength{edit.descriptor == DataEdit::ListDirectedRealPart ||
                edit.descriptor == DataEdit::ListDirectedImaginaryPart
            ? 1
            : 0};
    length += prefixLength + suffixLength;
    ConnectionState &connection{io_.GetConnectionState()};
    return (!connection.NeedAdvance(length) || io_.AdvanceRecord()) &&
        EmitAscii(io_, " (", prefixLength);
  } else if (width > length) {
    return EmitRepeated(io_, ' ', width - length);
  } else {
    return true;
  }
}

RT_API_ATTRS bool RealOutputEditingBase::EmitSuffix(const DataEdit &edit) {
  if (edit.descriptor == DataEdit::ListDirectedRealPart) {
    return EmitAscii(
        io_, edit.modes.editingFlags & decimalComma ? ";" : ",", 1);
  } else if (edit.descriptor == DataEdit::ListDirectedImaginaryPart) {
    return EmitAscii(io_, ")", 1);
  } else {
````

- **L241 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L241 CN**: 延续周围的声明、表达式或控制流结构。
- **L242 EN**: Executes statement `: 1};`.
  **L242 CN**: 执行语句 `: 1};`。
- **L243 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L243 CN**: 延续周围的声明、表达式或控制流结构。
- **L244 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L244 CN**: 延续周围的声明、表达式或控制流结构。
- **L245 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L245 CN**: 延续周围的声明、表达式或控制流结构。
- **L246 EN**: Executes statement `: 0};`.
  **L246 CN**: 执行语句 `: 0};`。
- **L247 EN**: Initializes or updates `+`.
  **L247 CN**: 初始化或更新 `+`。
- **L248 EN**: Executes statement involving `GetConnectionState`.
  **L248 CN**: 执行涉及 `GetConnectionState` 的语句。
- **L249 EN**: Returns from the current function, often propagating a computed result.
  **L249 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L250 EN**: Executes statement involving `EmitAscii`.
  **L250 CN**: 执行涉及 `EmitAscii` 的语句。
- **L251 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L251 CN**: 延续周围的声明、表达式或控制流结构。
- **L252 EN**: Returns from the current function, often propagating a computed result.
  **L252 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L253 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L253 CN**: 延续周围的声明、表达式或控制流结构。
- **L254 EN**: Returns from the current function, often propagating a computed result.
  **L254 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L255 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L255 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L256 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L256 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L257 EN**: Blank line separates nearby declarations or logic blocks.
  **L257 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L258 EN**: Declares or defines callable `EmitSuffix`.
  **L258 CN**: 声明或定义可调用实体 `EmitSuffix`。
- **L259 EN**: Introduces conditional control flow with an `if` statement.
  **L259 CN**: 通过 `if` 语句引入条件控制流。
- **L260 EN**: Returns from the current function, often propagating a computed result.
  **L260 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L261 EN**: Executes statement `io_, edit.modes.editingFlags & decimalComma ? ";" : ",", 1);`.
  **L261 CN**: 执行语句 `io_, edit.modes.editingFlags & decimalComma ? ";" : ",", 1);`。
- **L262 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L262 CN**: 延续周围的声明、表达式或控制流结构。
- **L263 EN**: Returns from the current function, often propagating a computed result.
  **L263 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L264 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L264 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 265-288

````cpp
    return true;
  }
}

static RT_API_ATTRS char IsInfOrNaN(const char *p, std::size_t length) {
  if (!p || length < 1) {
    return '\0';
  }
  if (*p == '-' || *p == '+') {
    if (length == 1) {
      return '\0';
    }
    ++p;
  }
  return *p == 'I' || *p == 'N' ? *p : '\0';
}

template <int KIND>
RT_API_ATTRS decimal::ConversionToDecimalResult
RealOutputEditing<KIND>::ConvertToDecimal(int significantDigits,
    enum decimal::FortranRounding rounding, int width, int flags) {
  auto converted{decimal::ConvertToDecimal<binaryPrecision>(buffer_,
      sizeof buffer_, static_cast<enum decimal::DecimalConversionFlags>(flags),
      significantDigits, rounding, x_)};
````

- **L265 EN**: Returns from the current function, often propagating a computed result.
  **L265 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L266 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L266 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L267 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L267 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L268 EN**: Blank line separates nearby declarations or logic blocks.
  **L268 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L269 EN**: Declares or defines callable `IsInfOrNaN`.
  **L269 CN**: 声明或定义可调用实体 `IsInfOrNaN`。
- **L270 EN**: Introduces conditional control flow with an `if` statement.
  **L270 CN**: 通过 `if` 语句引入条件控制流。
- **L271 EN**: Returns from the current function, often propagating a computed result.
  **L271 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L272 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L272 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L273 EN**: Introduces conditional control flow with an `if` statement.
  **L273 CN**: 通过 `if` 语句引入条件控制流。
- **L274 EN**: Introduces conditional control flow with an `if` statement.
  **L274 CN**: 通过 `if` 语句引入条件控制流。
- **L275 EN**: Returns from the current function, often propagating a computed result.
  **L275 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L276 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L276 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L277 EN**: Executes statement `++p;`.
  **L277 CN**: 执行语句 `++p;`。
- **L278 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L278 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L279 EN**: Returns from the current function, often propagating a computed result.
  **L279 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L280 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L280 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L281 EN**: Blank line separates nearby declarations or logic blocks.
  **L281 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L282 EN**: Begins a template declaration parameterizing subsequent code.
  **L282 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L283 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L283 CN**: 延续周围的声明、表达式或控制流结构。
- **L284 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L284 CN**: 延续周围的声明、表达式或控制流结构。
- **L285 EN**: Declares or defines enum `decimal`.
  **L285 CN**: 声明或定义 enum `decimal`。
- **L286 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L286 CN**: 延续周围的声明、表达式或控制流结构。
- **L287 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L287 CN**: 延续周围的声明、表达式或控制流结构。
- **L288 EN**: Executes statement `significantDigits, rounding, x_)};`.
  **L288 CN**: 执行语句 `significantDigits, rounding, x_)};`。

### Lines 289-312

````cpp
  if (!converted.str) { // overflow
    io_.GetIoErrorHandler().Crash(
        "RealOutputEditing::ConvertToDecimal: buffer size %zd was insufficient",
        sizeof buffer_);
  } else if (IsInfOrNaN(converted.str, converted.length) == 'I' &&
      converted.length <= 4 &&
      static_cast<int>(converted.length + 5) <= width) {
    // Emit "Infinity" rather than "Inf" (F'2023 13.7.2.3.2 p9), possibly signed
    std::memcpy(buffer_, converted.str, converted.length);
    std::memcpy(buffer_ + converted.length, "inity", 5);
    converted.str = buffer_;
    converted.length += 5;
  }
  return converted;
}

// 13.7.2.3.3 in F'2018
template <int KIND>
RT_API_ATTRS bool RealOutputEditing<KIND>::EditEorDOutput(
    const DataEdit &edit) {
  AddSpaceBeforeCharacter(io_);
  int editDigits{edit.digits.value_or(0)}; // 'd' field
  int editWidth{edit.width.value_or(0)}; // 'w' field
  int significantDigits{editDigits};
````

- **L289 EN**: Introduces conditional control flow with an `if` statement.
  **L289 CN**: 通过 `if` 语句引入条件控制流。
- **L290 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L290 CN**: 延续周围的声明、表达式或控制流结构。
- **L291 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L291 CN**: 延续周围的声明、表达式或控制流结构。
- **L292 EN**: Executes statement `sizeof buffer_);`.
  **L292 CN**: 执行语句 `sizeof buffer_);`。
- **L293 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L293 CN**: 延续周围的声明、表达式或控制流结构。
- **L294 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L294 CN**: 延续周围的声明、表达式或控制流结构。
- **L295 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L295 CN**: 延续周围的声明、表达式或控制流结构。
- **L296 EN**: Comment documents intent or context: `Emit "Infinity" rather than "Inf" (F'2023 13.7.2.3.2 p9), possibly signed`.
  **L296 CN**: 注释记录了意图或上下文：`Emit "Infinity" rather than "Inf" (F'2023 13.7.2.3.2 p9), possibly signed`。
- **L297 EN**: Executes statement involving `memcpy`.
  **L297 CN**: 执行涉及 `memcpy` 的语句。
- **L298 EN**: Executes statement involving `memcpy`.
  **L298 CN**: 执行涉及 `memcpy` 的语句。
- **L299 EN**: Initializes or updates `converted.str`.
  **L299 CN**: 初始化或更新 `converted.str`。
- **L300 EN**: Initializes or updates `+`.
  **L300 CN**: 初始化或更新 `+`。
- **L301 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L301 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L302 EN**: Returns from the current function, often propagating a computed result.
  **L302 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L303 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L303 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L304 EN**: Blank line separates nearby declarations or logic blocks.
  **L304 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L305 EN**: Comment documents intent or context: `13.7.2.3.3 in F'2018`.
  **L305 CN**: 注释记录了意图或上下文：`13.7.2.3.3 in F'2018`。
- **L306 EN**: Begins a template declaration parameterizing subsequent code.
  **L306 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L307 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L307 CN**: 延续周围的声明、表达式或控制流结构。
- **L308 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L308 CN**: 延续周围的声明、表达式或控制流结构。
- **L309 EN**: Executes statement involving `AddSpaceBeforeCharacter`.
  **L309 CN**: 执行涉及 `AddSpaceBeforeCharacter` 的语句。
- **L310 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L310 CN**: 延续周围的声明、表达式或控制流结构。
- **L311 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L311 CN**: 延续周围的声明、表达式或控制流结构。
- **L312 EN**: Executes statement `int significantDigits{editDigits};`.
  **L312 CN**: 执行语句 `int significantDigits{editDigits};`。

### Lines 313-336

````cpp
  int flags{0};
  if (edit.modes.editingFlags & signPlus) {
    flags |= decimal::AlwaysSign;
  }
  int scale{edit.modes.scale}; // 'kP' value
  bool isEN{edit.variation == 'N'};
  bool isES{edit.variation == 'S'};
  if (editWidth == 0) { // "the processor selects the field width"
    if (edit.digits.has_value()) { // E0.d
      if (editDigits == 0 && scale <= 0) { // E0.0
        significantDigits = isEN || isES ? 0 : 1;
      }
    } else { // E0
      flags |= decimal::Minimize;
      significantDigits =
          sizeof buffer_ - 5; // sign, NUL, + 3 extra for EN scaling
    }
  }
  int zeroesAfterPoint{0};
  if (isEN) {
    scale = IsZero() ? 1 : 3;
    significantDigits += scale;
  } else if (isES) {
    scale = 1;
````

- **L313 EN**: Executes statement `int flags{0};`.
  **L313 CN**: 执行语句 `int flags{0};`。
- **L314 EN**: Introduces conditional control flow with an `if` statement.
  **L314 CN**: 通过 `if` 语句引入条件控制流。
- **L315 EN**: Initializes or updates `|`.
  **L315 CN**: 初始化或更新 `|`。
- **L316 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L316 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L317 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L317 CN**: 延续周围的声明、表达式或控制流结构。
- **L318 EN**: Executes statement `bool isEN{edit.variation == 'N'};`.
  **L318 CN**: 执行语句 `bool isEN{edit.variation == 'N'};`。
- **L319 EN**: Executes statement `bool isES{edit.variation == 'S'};`.
  **L319 CN**: 执行语句 `bool isES{edit.variation == 'S'};`。
- **L320 EN**: Introduces conditional control flow with an `if` statement.
  **L320 CN**: 通过 `if` 语句引入条件控制流。
- **L321 EN**: Introduces conditional control flow with an `if` statement.
  **L321 CN**: 通过 `if` 语句引入条件控制流。
- **L322 EN**: Introduces conditional control flow with an `if` statement.
  **L322 CN**: 通过 `if` 语句引入条件控制流。
- **L323 EN**: Initializes or updates `significantDigits`.
  **L323 CN**: 初始化或更新 `significantDigits`。
- **L324 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L324 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L325 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L325 CN**: 延续周围的声明、表达式或控制流结构。
- **L326 EN**: Initializes or updates `|`.
  **L326 CN**: 初始化或更新 `|`。
- **L327 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L327 CN**: 延续周围的声明、表达式或控制流结构。
- **L328 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L328 CN**: 延续周围的声明、表达式或控制流结构。
- **L329 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L329 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L330 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L330 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L331 EN**: Executes statement `int zeroesAfterPoint{0};`.
  **L331 CN**: 执行语句 `int zeroesAfterPoint{0};`。
- **L332 EN**: Introduces conditional control flow with an `if` statement.
  **L332 CN**: 通过 `if` 语句引入条件控制流。
- **L333 EN**: Initializes or updates `scale`.
  **L333 CN**: 初始化或更新 `scale`。
- **L334 EN**: Initializes or updates `+`.
  **L334 CN**: 初始化或更新 `+`。
- **L335 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L335 CN**: 延续周围的声明、表达式或控制流结构。
- **L336 EN**: Initializes or updates `scale`.
  **L336 CN**: 初始化或更新 `scale`。

### Lines 337-360

````cpp
    ++significantDigits;
  } else if (scale < 0) {
    if (scale <= -editDigits) {
      io_.GetIoErrorHandler().SignalError(IostatBadScaleFactor,
          "Scale factor (kP) %d cannot be less than -d (%d)", scale,
          -editDigits);
      return false;
    }
    zeroesAfterPoint = -scale;
    significantDigits = std::max(0, significantDigits - zeroesAfterPoint);
  } else if (scale > 0) {
    if (scale >= editDigits + 2) {
      io_.GetIoErrorHandler().SignalError(IostatBadScaleFactor,
          "Scale factor (kP) %d cannot be greater than d+2 (%d)", scale,
          editDigits + 2);
      return false;
    }
    ++significantDigits;
    scale = std::min(scale, significantDigits + 1);
  } else if (edit.digits.value_or(1) == 0 && !edit.variation) {
    // F'2023 13.7.2.3.3 p5; does not apply to Gw.0(Ee) or E0(no d)
    io_.GetIoErrorHandler().SignalError(IostatErrorInFormat,
        "Output edit descriptor %cw.d must have d>0", edit.descriptor);
    return false;
````

- **L337 EN**: Executes statement `++significantDigits;`.
  **L337 CN**: 执行语句 `++significantDigits;`。
- **L338 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L338 CN**: 延续周围的声明、表达式或控制流结构。
- **L339 EN**: Introduces conditional control flow with an `if` statement.
  **L339 CN**: 通过 `if` 语句引入条件控制流。
- **L340 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L340 CN**: 延续周围的声明、表达式或控制流结构。
- **L341 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L341 CN**: 延续周围的声明、表达式或控制流结构。
- **L342 EN**: Executes statement `-editDigits);`.
  **L342 CN**: 执行语句 `-editDigits);`。
- **L343 EN**: Returns from the current function, often propagating a computed result.
  **L343 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L344 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L344 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L345 EN**: Initializes or updates `zeroesAfterPoint`.
  **L345 CN**: 初始化或更新 `zeroesAfterPoint`。
- **L346 EN**: Initializes or updates `significantDigits`.
  **L346 CN**: 初始化或更新 `significantDigits`。
- **L347 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L347 CN**: 延续周围的声明、表达式或控制流结构。
- **L348 EN**: Introduces conditional control flow with an `if` statement.
  **L348 CN**: 通过 `if` 语句引入条件控制流。
- **L349 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L349 CN**: 延续周围的声明、表达式或控制流结构。
- **L350 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L350 CN**: 延续周围的声明、表达式或控制流结构。
- **L351 EN**: Executes statement `editDigits + 2);`.
  **L351 CN**: 执行语句 `editDigits + 2);`。
- **L352 EN**: Returns from the current function, often propagating a computed result.
  **L352 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L353 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L353 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L354 EN**: Executes statement `++significantDigits;`.
  **L354 CN**: 执行语句 `++significantDigits;`。
- **L355 EN**: Initializes or updates `scale`.
  **L355 CN**: 初始化或更新 `scale`。
- **L356 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L356 CN**: 延续周围的声明、表达式或控制流结构。
- **L357 EN**: Comment documents intent or context: `F'2023 13.7.2.3.3 p5; does not apply to Gw.0(Ee) or E0(no d)`.
  **L357 CN**: 注释记录了意图或上下文：`F'2023 13.7.2.3.3 p5; does not apply to Gw.0(Ee) or E0(no d)`。
- **L358 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L358 CN**: 延续周围的声明、表达式或控制流结构。
- **L359 EN**: Executes statement `"Output edit descriptor %cw.d must have d>0", edit.descriptor);`.
  **L359 CN**: 执行语句 `"Output edit descriptor %cw.d must have d>0", edit.descriptor);`。
- **L360 EN**: Returns from the current function, often propagating a computed result.
  **L360 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 361-384

````cpp
  }
  // In EN editing, multiple attempts may be necessary, so this is a loop.
  while (true) {
    decimal::ConversionToDecimalResult converted{ConvertToDecimal(
        significantDigits, edit.modes.round, editWidth, flags)};
    if (IsInfOrNaN(converted.str, converted.length)) {
      return editWidth > 0 &&
              converted.length + trailingBlanks_ >
                  static_cast<std::size_t>(editWidth)
          ? EmitRepeated(io_, '*', editWidth)
          : EmitPrefix(edit, converted.length, editWidth) &&
              EmitAscii(io_, converted.str, converted.length) &&
              EmitRepeated(io_, ' ', trailingBlanks_) && EmitSuffix(edit);
    }
    if (!IsZero()) {
      converted.decimalExponent -= scale;
    }
    if (isEN) {
      // EN mode: we need an effective exponent field that is
      // a multiple of three.
      if (int modulus{converted.decimalExponent % 3}; modulus != 0) {
        if (significantDigits > 1) {
          --significantDigits;
          --scale;
````

- **L361 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L361 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L362 EN**: Comment documents intent or context: `In EN editing, multiple attempts may be necessary, so this is a loop.`.
  **L362 CN**: 注释记录了意图或上下文：`In EN editing, multiple attempts may be necessary, so this is a loop.`。
- **L363 EN**: Starts a `while` loop controlled by a runtime condition.
  **L363 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L364 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L364 CN**: 延续周围的声明、表达式或控制流结构。
- **L365 EN**: Executes statement `significantDigits, edit.modes.round, editWidth, flags)};`.
  **L365 CN**: 执行语句 `significantDigits, edit.modes.round, editWidth, flags)};`。
- **L366 EN**: Introduces conditional control flow with an `if` statement.
  **L366 CN**: 通过 `if` 语句引入条件控制流。
- **L367 EN**: Returns from the current function, often propagating a computed result.
  **L367 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L368 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L368 CN**: 延续周围的声明、表达式或控制流结构。
- **L369 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L369 CN**: 延续周围的声明、表达式或控制流结构。
- **L370 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L370 CN**: 延续周围的声明、表达式或控制流结构。
- **L371 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L371 CN**: 延续周围的声明、表达式或控制流结构。
- **L372 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L372 CN**: 延续周围的声明、表达式或控制流结构。
- **L373 EN**: Executes statement involving `EmitRepeated`.
  **L373 CN**: 执行涉及 `EmitRepeated` 的语句。
- **L374 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L374 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L375 EN**: Introduces conditional control flow with an `if` statement.
  **L375 CN**: 通过 `if` 语句引入条件控制流。
- **L376 EN**: Initializes or updates `-`.
  **L376 CN**: 初始化或更新 `-`。
- **L377 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L377 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L378 EN**: Introduces conditional control flow with an `if` statement.
  **L378 CN**: 通过 `if` 语句引入条件控制流。
- **L379 EN**: Comment documents intent or context: `EN mode: we need an effective exponent field that is`.
  **L379 CN**: 注释记录了意图或上下文：`EN mode: we need an effective exponent field that is`。
- **L380 EN**: Comment documents intent or context: `a multiple of three.`.
  **L380 CN**: 注释记录了意图或上下文：`a multiple of three.`。
- **L381 EN**: Introduces conditional control flow with an `if` statement.
  **L381 CN**: 通过 `if` 语句引入条件控制流。
- **L382 EN**: Introduces conditional control flow with an `if` statement.
  **L382 CN**: 通过 `if` 语句引入条件控制流。
- **L383 EN**: Executes statement `--significantDigits;`.
  **L383 CN**: 执行语句 `--significantDigits;`。
- **L384 EN**: Executes statement `--scale;`.
  **L384 CN**: 执行语句 `--scale;`。

### Lines 385-408

````cpp
          continue;
        }
        // Rounded nines up to a 1.
        scale += modulus;
        converted.decimalExponent -= modulus;
      }
      if (scale > 3) {
        int adjust{3 * (scale / 3)};
        scale -= adjust;
        converted.decimalExponent += adjust;
      } else if (scale < 1) {
        int adjust{3 - 3 * (scale / 3)};
        scale += adjust;
        converted.decimalExponent -= adjust;
      }
      significantDigits = editDigits + scale;
    }
    // Format the exponent (see table 13.1 for all the cases)
    int expoLength{0};
    const char *exponent{
        FormatExponent(converted.decimalExponent, edit, expoLength)};
    int signLength{*converted.str == '-' || *converted.str == '+' ? 1 : 0};
    int convertedDigits{static_cast<int>(converted.length) - signLength};
    int zeroesBeforePoint{std::max(0, scale - convertedDigits)};
````

- **L385 EN**: Skips to the next loop iteration.
  **L385 CN**: 跳到下一次循环迭代。
- **L386 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L386 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L387 EN**: Comment documents intent or context: `Rounded nines up to a 1.`.
  **L387 CN**: 注释记录了意图或上下文：`Rounded nines up to a 1.`。
- **L388 EN**: Initializes or updates `+`.
  **L388 CN**: 初始化或更新 `+`。
- **L389 EN**: Initializes or updates `-`.
  **L389 CN**: 初始化或更新 `-`。
- **L390 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L390 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L391 EN**: Introduces conditional control flow with an `if` statement.
  **L391 CN**: 通过 `if` 语句引入条件控制流。
- **L392 EN**: Executes statement `int adjust{3 * (scale / 3)};`.
  **L392 CN**: 执行语句 `int adjust{3 * (scale / 3)};`。
- **L393 EN**: Initializes or updates `-`.
  **L393 CN**: 初始化或更新 `-`。
- **L394 EN**: Initializes or updates `+`.
  **L394 CN**: 初始化或更新 `+`。
- **L395 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L395 CN**: 延续周围的声明、表达式或控制流结构。
- **L396 EN**: Executes statement `int adjust{3 - 3 * (scale / 3)};`.
  **L396 CN**: 执行语句 `int adjust{3 - 3 * (scale / 3)};`。
- **L397 EN**: Initializes or updates `+`.
  **L397 CN**: 初始化或更新 `+`。
- **L398 EN**: Initializes or updates `-`.
  **L398 CN**: 初始化或更新 `-`。
- **L399 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L399 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L400 EN**: Initializes or updates `significantDigits`.
  **L400 CN**: 初始化或更新 `significantDigits`。
- **L401 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L401 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L402 EN**: Comment documents intent or context: `Format the exponent (see table 13.1 for all the cases)`.
  **L402 CN**: 注释记录了意图或上下文：`Format the exponent (see table 13.1 for all the cases)`。
- **L403 EN**: Executes statement `int expoLength{0};`.
  **L403 CN**: 执行语句 `int expoLength{0};`。
- **L404 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L404 CN**: 延续周围的声明、表达式或控制流结构。
- **L405 EN**: Executes statement involving `FormatExponent`.
  **L405 CN**: 执行涉及 `FormatExponent` 的语句。
- **L406 EN**: Executes statement `int signLength{*converted.str == '-' || *converted.str == '+' ? 1 : 0};`.
  **L406 CN**: 执行语句 `int signLength{*converted.str == '-' || *converted.str == '+' ? 1 : 0};`。
- **L407 EN**: Executes statement `int convertedDigits{static_cast<int>(converted.length) - signLength};`.
  **L407 CN**: 执行语句 `int convertedDigits{static_cast<int>(converted.length) - signLength};`。
- **L408 EN**: Executes statement involving `max`.
  **L408 CN**: 执行涉及 `max` 的语句。

### Lines 409-432

````cpp
    int digitsBeforePoint{std::max(0, scale - zeroesBeforePoint)};
    int digitsAfterPoint{convertedDigits - digitsBeforePoint};
    int trailingZeroes{flags & decimal::Minimize
            ? 0
            : std::max(0,
                  significantDigits - (convertedDigits + zeroesBeforePoint))};
    int totalLength{signLength + digitsBeforePoint + zeroesBeforePoint +
        1 /*'.'*/ + zeroesAfterPoint + digitsAfterPoint + trailingZeroes +
        expoLength};
    int width{editWidth > 0 ? editWidth : totalLength};
    if (totalLength > width || !exponent) {
      return EmitRepeated(io_, '*', width);
    }
    if (totalLength < width && digitsBeforePoint == 0 &&
        zeroesBeforePoint == 0 &&
        !(edit.modes.editingFlags & leadingZeroSuppress)) {
      zeroesBeforePoint = 1;
      ++totalLength;
    }
    if (totalLength < width && editWidth == 0) {
      width = totalLength;
    }
    return EmitPrefix(edit, totalLength, width) &&
        EmitAscii(io_, converted.str, signLength + digitsBeforePoint) &&
````

- **L409 EN**: Executes statement involving `max`.
  **L409 CN**: 执行涉及 `max` 的语句。
- **L410 EN**: Executes statement `int digitsAfterPoint{convertedDigits - digitsBeforePoint};`.
  **L410 CN**: 执行语句 `int digitsAfterPoint{convertedDigits - digitsBeforePoint};`。
- **L411 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L411 CN**: 延续周围的声明、表达式或控制流结构。
- **L412 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L412 CN**: 延续周围的声明、表达式或控制流结构。
- **L413 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L413 CN**: 延续周围的声明、表达式或控制流结构。
- **L414 EN**: Executes statement `significantDigits - (convertedDigits + zeroesBeforePoint))};`.
  **L414 CN**: 执行语句 `significantDigits - (convertedDigits + zeroesBeforePoint))};`。
- **L415 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L415 CN**: 延续周围的声明、表达式或控制流结构。
- **L416 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L416 CN**: 延续周围的声明、表达式或控制流结构。
- **L417 EN**: Executes statement `expoLength};`.
  **L417 CN**: 执行语句 `expoLength};`。
- **L418 EN**: Executes statement `int width{editWidth > 0 ? editWidth : totalLength};`.
  **L418 CN**: 执行语句 `int width{editWidth > 0 ? editWidth : totalLength};`。
- **L419 EN**: Introduces conditional control flow with an `if` statement.
  **L419 CN**: 通过 `if` 语句引入条件控制流。
- **L420 EN**: Returns from the current function, often propagating a computed result.
  **L420 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L421 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L421 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L422 EN**: Introduces conditional control flow with an `if` statement.
  **L422 CN**: 通过 `if` 语句引入条件控制流。
- **L423 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L423 CN**: 延续周围的声明、表达式或控制流结构。
- **L424 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L424 CN**: 延续周围的声明、表达式或控制流结构。
- **L425 EN**: Initializes or updates `zeroesBeforePoint`.
  **L425 CN**: 初始化或更新 `zeroesBeforePoint`。
- **L426 EN**: Executes statement `++totalLength;`.
  **L426 CN**: 执行语句 `++totalLength;`。
- **L427 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L427 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L428 EN**: Introduces conditional control flow with an `if` statement.
  **L428 CN**: 通过 `if` 语句引入条件控制流。
- **L429 EN**: Initializes or updates `width`.
  **L429 CN**: 初始化或更新 `width`。
- **L430 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L430 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L431 EN**: Returns from the current function, often propagating a computed result.
  **L431 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L432 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L432 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 433-456

````cpp
        EmitRepeated(io_, '0', zeroesBeforePoint) &&
        EmitAscii(io_, edit.modes.editingFlags & decimalComma ? "," : ".", 1) &&
        EmitRepeated(io_, '0', zeroesAfterPoint) &&
        EmitAscii(io_, converted.str + signLength + digitsBeforePoint,
            digitsAfterPoint) &&
        EmitRepeated(io_, '0', trailingZeroes) &&
        EmitAscii(io_, exponent, expoLength) && EmitSuffix(edit);
  }
}

// 13.7.2.3.2 in F'2018
template <int KIND>
RT_API_ATTRS bool RealOutputEditing<KIND>::EditFOutput(const DataEdit &edit) {
  AddSpaceBeforeCharacter(io_);
  int fracDigits{edit.digits.value_or(0)}; // 'd' field
  const int editWidth{edit.width.value_or(0)}; // 'w' field
  enum decimal::FortranRounding rounding{edit.modes.round};
  int flags{0};
  if (edit.modes.editingFlags & signPlus) {
    flags |= decimal::AlwaysSign;
  }
  if (editWidth == 0) { // "the processor selects the field width"
    if (!edit.digits.has_value()) { // F0
      flags |= decimal::Minimize;
````

- **L433 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L433 CN**: 延续周围的声明、表达式或控制流结构。
- **L434 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L434 CN**: 延续周围的声明、表达式或控制流结构。
- **L435 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L435 CN**: 延续周围的声明、表达式或控制流结构。
- **L436 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L436 CN**: 延续周围的声明、表达式或控制流结构。
- **L437 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L437 CN**: 延续周围的声明、表达式或控制流结构。
- **L438 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L438 CN**: 延续周围的声明、表达式或控制流结构。
- **L439 EN**: Executes statement involving `EmitAscii`.
  **L439 CN**: 执行涉及 `EmitAscii` 的语句。
- **L440 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L440 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L441 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L441 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L442 EN**: Blank line separates nearby declarations or logic blocks.
  **L442 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L443 EN**: Comment documents intent or context: `13.7.2.3.2 in F'2018`.
  **L443 CN**: 注释记录了意图或上下文：`13.7.2.3.2 in F'2018`。
- **L444 EN**: Begins a template declaration parameterizing subsequent code.
  **L444 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L445 EN**: Declares or defines callable `EditFOutput`.
  **L445 CN**: 声明或定义可调用实体 `EditFOutput`。
- **L446 EN**: Executes statement involving `AddSpaceBeforeCharacter`.
  **L446 CN**: 执行涉及 `AddSpaceBeforeCharacter` 的语句。
- **L447 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L447 CN**: 延续周围的声明、表达式或控制流结构。
- **L448 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L448 CN**: 延续周围的声明、表达式或控制流结构。
- **L449 EN**: Declares or defines enum `decimal`.
  **L449 CN**: 声明或定义 enum `decimal`。
- **L450 EN**: Executes statement `int flags{0};`.
  **L450 CN**: 执行语句 `int flags{0};`。
- **L451 EN**: Introduces conditional control flow with an `if` statement.
  **L451 CN**: 通过 `if` 语句引入条件控制流。
- **L452 EN**: Initializes or updates `|`.
  **L452 CN**: 初始化或更新 `|`。
- **L453 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L453 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L454 EN**: Introduces conditional control flow with an `if` statement.
  **L454 CN**: 通过 `if` 语句引入条件控制流。
- **L455 EN**: Introduces conditional control flow with an `if` statement.
  **L455 CN**: 通过 `if` 语句引入条件控制流。
- **L456 EN**: Initializes or updates `|`.
  **L456 CN**: 初始化或更新 `|`。

### Lines 457-480

````cpp
      fracDigits = sizeof buffer_ - 2; // sign & NUL
    }
  }
  bool emitTrailingZeroes{!(flags & decimal::Minimize)};
  // Multiple conversions may be needed to get the right number of
  // effective rounded fractional digits.
  bool canIncrease{true};
  for (int extraDigits{fracDigits == 0 ? 1 : 0};;) {
    decimal::ConversionToDecimalResult converted{
        ConvertToDecimal(extraDigits + fracDigits, rounding, editWidth, flags)};
    const char *convertedStr{converted.str};
    if (IsInfOrNaN(converted.str, converted.length)) {
      return editWidth > 0 &&
              converted.length > static_cast<std::size_t>(editWidth)
          ? EmitRepeated(io_, '*', editWidth)
          : EmitPrefix(edit, converted.length, editWidth) &&
              EmitAscii(io_, convertedStr, converted.length) &&
              EmitSuffix(edit);
    }
    int expo{converted.decimalExponent + edit.modes.scale /*kP*/};
    int signLength{*convertedStr == '-' || *convertedStr == '+' ? 1 : 0};
    int convertedDigits{static_cast<int>(converted.length) - signLength};
    if (IsZero()) { // don't treat converted "0" as significant digit
      expo = 0;
````

- **L457 EN**: Initializes or updates `fracDigits`.
  **L457 CN**: 初始化或更新 `fracDigits`。
- **L458 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L458 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L459 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L459 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L460 EN**: Executes statement `bool emitTrailingZeroes{!(flags & decimal::Minimize)};`.
  **L460 CN**: 执行语句 `bool emitTrailingZeroes{!(flags & decimal::Minimize)};`。
- **L461 EN**: Comment documents intent or context: `Multiple conversions may be needed to get the right number of`.
  **L461 CN**: 注释记录了意图或上下文：`Multiple conversions may be needed to get the right number of`。
- **L462 EN**: Comment documents intent or context: `effective rounded fractional digits.`.
  **L462 CN**: 注释记录了意图或上下文：`effective rounded fractional digits.`。
- **L463 EN**: Executes statement `bool canIncrease{true};`.
  **L463 CN**: 执行语句 `bool canIncrease{true};`。
- **L464 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L464 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L465 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L465 CN**: 延续周围的声明、表达式或控制流结构。
- **L466 EN**: Executes statement involving `ConvertToDecimal`.
  **L466 CN**: 执行涉及 `ConvertToDecimal` 的语句。
- **L467 EN**: Executes statement `const char *convertedStr{converted.str};`.
  **L467 CN**: 执行语句 `const char *convertedStr{converted.str};`。
- **L468 EN**: Introduces conditional control flow with an `if` statement.
  **L468 CN**: 通过 `if` 语句引入条件控制流。
- **L469 EN**: Returns from the current function, often propagating a computed result.
  **L469 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L470 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L470 CN**: 延续周围的声明、表达式或控制流结构。
- **L471 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L471 CN**: 延续周围的声明、表达式或控制流结构。
- **L472 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L472 CN**: 延续周围的声明、表达式或控制流结构。
- **L473 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L473 CN**: 延续周围的声明、表达式或控制流结构。
- **L474 EN**: Executes statement involving `EmitSuffix`.
  **L474 CN**: 执行涉及 `EmitSuffix` 的语句。
- **L475 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L475 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L476 EN**: Executes statement `int expo{converted.decimalExponent + edit.modes.scale /*kP*/};`.
  **L476 CN**: 执行语句 `int expo{converted.decimalExponent + edit.modes.scale /*kP*/};`。
- **L477 EN**: Executes statement `int signLength{*convertedStr == '-' || *convertedStr == '+' ? 1 : 0};`.
  **L477 CN**: 执行语句 `int signLength{*convertedStr == '-' || *convertedStr == '+' ? 1 : 0};`。
- **L478 EN**: Executes statement `int convertedDigits{static_cast<int>(converted.length) - signLength};`.
  **L478 CN**: 执行语句 `int convertedDigits{static_cast<int>(converted.length) - signLength};`。
- **L479 EN**: Introduces conditional control flow with an `if` statement.
  **L479 CN**: 通过 `if` 语句引入条件控制流。
- **L480 EN**: Initializes or updates `expo`.
  **L480 CN**: 初始化或更新 `expo`。

### Lines 481-504

````cpp
      convertedDigits = 0;
    }
    bool isNegative{*convertedStr == '-'};
    char one[2];
    if (expo > extraDigits && extraDigits >= 0 && canIncrease) {
      extraDigits = expo;
      if (!edit.digits.has_value()) { // F0
        fracDigits = sizeof buffer_ - extraDigits - 2; // sign & NUL
      }
      canIncrease = false; // only once
      continue;
    } else if (expo == -fracDigits && convertedDigits > 0) {
      // Result will be either a signed zero or power of ten, depending
      // on rounding.
      char leading{convertedStr[signLength]};
      bool roundToPowerOfTen{false};
      switch (edit.modes.round) {
      case decimal::FortranRounding::RoundUp:
        roundToPowerOfTen = !isNegative;
        break;
      case decimal::FortranRounding::RoundDown:
        roundToPowerOfTen = isNegative;
        break;
      case decimal::FortranRounding::RoundToZero:
````

- **L481 EN**: Initializes or updates `convertedDigits`.
  **L481 CN**: 初始化或更新 `convertedDigits`。
- **L482 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L482 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L483 EN**: Executes statement `bool isNegative{*convertedStr == '-'};`.
  **L483 CN**: 执行语句 `bool isNegative{*convertedStr == '-'};`。
- **L484 EN**: Executes statement `char one[2];`.
  **L484 CN**: 执行语句 `char one[2];`。
- **L485 EN**: Introduces conditional control flow with an `if` statement.
  **L485 CN**: 通过 `if` 语句引入条件控制流。
- **L486 EN**: Initializes or updates `extraDigits`.
  **L486 CN**: 初始化或更新 `extraDigits`。
- **L487 EN**: Introduces conditional control flow with an `if` statement.
  **L487 CN**: 通过 `if` 语句引入条件控制流。
- **L488 EN**: Initializes or updates `fracDigits`.
  **L488 CN**: 初始化或更新 `fracDigits`。
- **L489 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L489 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L490 EN**: Initializes or updates `canIncrease`.
  **L490 CN**: 初始化或更新 `canIncrease`。
- **L491 EN**: Skips to the next loop iteration.
  **L491 CN**: 跳到下一次循环迭代。
- **L492 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L492 CN**: 延续周围的声明、表达式或控制流结构。
- **L493 EN**: Comment documents intent or context: `Result will be either a signed zero or power of ten, depending`.
  **L493 CN**: 注释记录了意图或上下文：`Result will be either a signed zero or power of ten, depending`。
- **L494 EN**: Comment documents intent or context: `on rounding.`.
  **L494 CN**: 注释记录了意图或上下文：`on rounding.`。
- **L495 EN**: Executes statement `char leading{convertedStr[signLength]};`.
  **L495 CN**: 执行语句 `char leading{convertedStr[signLength]};`。
- **L496 EN**: Executes statement `bool roundToPowerOfTen{false};`.
  **L496 CN**: 执行语句 `bool roundToPowerOfTen{false};`。
- **L497 EN**: Begins a `switch` dispatch over discrete cases.
  **L497 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L498 EN**: Marks one `switch` case label.
  **L498 CN**: 标记一个 `switch` 的 case 标签。
- **L499 EN**: Initializes or updates `roundToPowerOfTen`.
  **L499 CN**: 初始化或更新 `roundToPowerOfTen`。
- **L500 EN**: Breaks out of the current loop or switch.
  **L500 CN**: 跳出当前循环或 switch。
- **L501 EN**: Marks one `switch` case label.
  **L501 CN**: 标记一个 `switch` 的 case 标签。
- **L502 EN**: Initializes or updates `roundToPowerOfTen`.
  **L502 CN**: 初始化或更新 `roundToPowerOfTen`。
- **L503 EN**: Breaks out of the current loop or switch.
  **L503 CN**: 跳出当前循环或 switch。
- **L504 EN**: Marks one `switch` case label.
  **L504 CN**: 标记一个 `switch` 的 case 标签。

### Lines 505-528

````cpp
        break;
      case decimal::FortranRounding::RoundNearest:
        if (leading == '5' &&
            rounding == decimal::FortranRounding::RoundNearest) {
          // Try again, rounding away from zero.
          rounding = isNegative ? decimal::FortranRounding::RoundDown
                                : decimal::FortranRounding::RoundUp;
          extraDigits = 1 - fracDigits; // just one digit needed
          continue;
        }
        roundToPowerOfTen = leading > '5';
        break;
      case decimal::FortranRounding::RoundCompatible:
        roundToPowerOfTen = leading >= '5';
        break;
      }
      if (roundToPowerOfTen) {
        ++expo;
        convertedDigits = 1;
        if (signLength > 0) {
          one[0] = *convertedStr;
          one[1] = '1';
        } else {
          one[0] = '1';
````

- **L505 EN**: Breaks out of the current loop or switch.
  **L505 CN**: 跳出当前循环或 switch。
- **L506 EN**: Marks one `switch` case label.
  **L506 CN**: 标记一个 `switch` 的 case 标签。
- **L507 EN**: Introduces conditional control flow with an `if` statement.
  **L507 CN**: 通过 `if` 语句引入条件控制流。
- **L508 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L508 CN**: 延续周围的声明、表达式或控制流结构。
- **L509 EN**: Comment documents intent or context: `Try again, rounding away from zero.`.
  **L509 CN**: 注释记录了意图或上下文：`Try again, rounding away from zero.`。
- **L510 EN**: Initializes or updates `rounding`.
  **L510 CN**: 初始化或更新 `rounding`。
- **L511 EN**: Executes statement `: decimal::FortranRounding::RoundUp;`.
  **L511 CN**: 执行语句 `: decimal::FortranRounding::RoundUp;`。
- **L512 EN**: Initializes or updates `extraDigits`.
  **L512 CN**: 初始化或更新 `extraDigits`。
- **L513 EN**: Skips to the next loop iteration.
  **L513 CN**: 跳到下一次循环迭代。
- **L514 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L514 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L515 EN**: Initializes or updates `roundToPowerOfTen`.
  **L515 CN**: 初始化或更新 `roundToPowerOfTen`。
- **L516 EN**: Breaks out of the current loop or switch.
  **L516 CN**: 跳出当前循环或 switch。
- **L517 EN**: Marks one `switch` case label.
  **L517 CN**: 标记一个 `switch` 的 case 标签。
- **L518 EN**: Initializes or updates `roundToPowerOfTen`.
  **L518 CN**: 初始化或更新 `roundToPowerOfTen`。
- **L519 EN**: Breaks out of the current loop or switch.
  **L519 CN**: 跳出当前循环或 switch。
- **L520 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L520 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L521 EN**: Introduces conditional control flow with an `if` statement.
  **L521 CN**: 通过 `if` 语句引入条件控制流。
- **L522 EN**: Executes statement `++expo;`.
  **L522 CN**: 执行语句 `++expo;`。
- **L523 EN**: Initializes or updates `convertedDigits`.
  **L523 CN**: 初始化或更新 `convertedDigits`。
- **L524 EN**: Introduces conditional control flow with an `if` statement.
  **L524 CN**: 通过 `if` 语句引入条件控制流。
- **L525 EN**: Initializes or updates `one[0]`.
  **L525 CN**: 初始化或更新 `one[0]`。
- **L526 EN**: Initializes or updates `one[1]`.
  **L526 CN**: 初始化或更新 `one[1]`。
- **L527 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L527 CN**: 延续周围的声明、表达式或控制流结构。
- **L528 EN**: Initializes or updates `one[0]`.
  **L528 CN**: 初始化或更新 `one[0]`。

### Lines 529-552

````cpp
        }
        convertedStr = one;
      } else {
        expo = 0;
        convertedDigits = 0;
      }
    } else if (expo < extraDigits && extraDigits > -fracDigits) {
      extraDigits = std::max(expo, -fracDigits);
      continue;
    }
    int digitsBeforePoint{std::max(0, std::min(expo, convertedDigits))};
    int zeroesBeforePoint{std::max(0, expo - digitsBeforePoint)};
    if (zeroesBeforePoint > 0 && (flags & decimal::Minimize)) {
      // If a minimized result looks like an integer, emit all of
      // its digits rather than clipping some to zeroes.
      // This can happen with HUGE(0._2) == 65504._2.
      flags &= ~decimal::Minimize;
      continue;
    }
    int zeroesAfterPoint{std::min(fracDigits, std::max(0, -expo))};
    int digitsAfterPoint{convertedDigits - digitsBeforePoint};
    int trailingZeroes{emitTrailingZeroes
            ? std::max(0, fracDigits - (zeroesAfterPoint + digitsAfterPoint))
            : 0};
````

- **L529 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L529 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L530 EN**: Initializes or updates `convertedStr`.
  **L530 CN**: 初始化或更新 `convertedStr`。
- **L531 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L531 CN**: 延续周围的声明、表达式或控制流结构。
- **L532 EN**: Initializes or updates `expo`.
  **L532 CN**: 初始化或更新 `expo`。
- **L533 EN**: Initializes or updates `convertedDigits`.
  **L533 CN**: 初始化或更新 `convertedDigits`。
- **L534 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L534 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L535 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L535 CN**: 延续周围的声明、表达式或控制流结构。
- **L536 EN**: Initializes or updates `extraDigits`.
  **L536 CN**: 初始化或更新 `extraDigits`。
- **L537 EN**: Skips to the next loop iteration.
  **L537 CN**: 跳到下一次循环迭代。
- **L538 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L538 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L539 EN**: Executes statement involving `max`.
  **L539 CN**: 执行涉及 `max` 的语句。
- **L540 EN**: Executes statement involving `max`.
  **L540 CN**: 执行涉及 `max` 的语句。
- **L541 EN**: Introduces conditional control flow with an `if` statement.
  **L541 CN**: 通过 `if` 语句引入条件控制流。
- **L542 EN**: Comment documents intent or context: `If a minimized result looks like an integer, emit all of`.
  **L542 CN**: 注释记录了意图或上下文：`If a minimized result looks like an integer, emit all of`。
- **L543 EN**: Comment documents intent or context: `its digits rather than clipping some to zeroes.`.
  **L543 CN**: 注释记录了意图或上下文：`its digits rather than clipping some to zeroes.`。
- **L544 EN**: Comment documents intent or context: `This can happen with HUGE(0._2) == 65504._2.`.
  **L544 CN**: 注释记录了意图或上下文：`This can happen with HUGE(0._2) == 65504._2.`。
- **L545 EN**: Initializes or updates `&`.
  **L545 CN**: 初始化或更新 `&`。
- **L546 EN**: Skips to the next loop iteration.
  **L546 CN**: 跳到下一次循环迭代。
- **L547 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L547 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L548 EN**: Executes statement involving `min`.
  **L548 CN**: 执行涉及 `min` 的语句。
- **L549 EN**: Executes statement `int digitsAfterPoint{convertedDigits - digitsBeforePoint};`.
  **L549 CN**: 执行语句 `int digitsAfterPoint{convertedDigits - digitsBeforePoint};`。
- **L550 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L550 CN**: 延续周围的声明、表达式或控制流结构。
- **L551 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L551 CN**: 延续周围的声明、表达式或控制流结构。
- **L552 EN**: Executes statement `: 0};`.
  **L552 CN**: 执行语句 `: 0};`。

### Lines 553-576

````cpp
    if (digitsBeforePoint + zeroesBeforePoint + zeroesAfterPoint +
            digitsAfterPoint + trailingZeroes ==
        0) {
      zeroesBeforePoint = 1; // "." -> "0." (avoid bare decimal point)
    }
    int totalLength{signLength + digitsBeforePoint + zeroesBeforePoint +
        1 /*'.'*/ + zeroesAfterPoint + digitsAfterPoint + trailingZeroes +
        trailingBlanks_ /* G editing converted to F */};
    int width{editWidth > 0 || trailingBlanks_ ? editWidth : totalLength};
    if (totalLength > width) {
      return EmitRepeated(io_, '*', width);
    }
    if (totalLength < width && digitsBeforePoint + zeroesBeforePoint == 0 &&
        !(edit.modes.editingFlags & leadingZeroSuppress)) {
      zeroesBeforePoint = 1;
      ++totalLength;
    }
    return EmitPrefix(edit, totalLength, width) &&
        EmitAscii(io_, convertedStr, signLength + digitsBeforePoint) &&
        EmitRepeated(io_, '0', zeroesBeforePoint) &&
        EmitAscii(io_, edit.modes.editingFlags & decimalComma ? "," : ".", 1) &&
        EmitRepeated(io_, '0', zeroesAfterPoint) &&
        EmitAscii(io_, convertedStr + signLength + digitsBeforePoint,
            digitsAfterPoint) &&
````

- **L553 EN**: Introduces conditional control flow with an `if` statement.
  **L553 CN**: 通过 `if` 语句引入条件控制流。
- **L554 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L554 CN**: 延续周围的声明、表达式或控制流结构。
- **L555 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L555 CN**: 延续周围的声明、表达式或控制流结构。
- **L556 EN**: Initializes or updates `zeroesBeforePoint`.
  **L556 CN**: 初始化或更新 `zeroesBeforePoint`。
- **L557 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L557 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L558 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L558 CN**: 延续周围的声明、表达式或控制流结构。
- **L559 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L559 CN**: 延续周围的声明、表达式或控制流结构。
- **L560 EN**: Executes statement `trailingBlanks_ /* G editing converted to F */};`.
  **L560 CN**: 执行语句 `trailingBlanks_ /* G editing converted to F */};`。
- **L561 EN**: Executes statement `int width{editWidth > 0 || trailingBlanks_ ? editWidth : totalLength};`.
  **L561 CN**: 执行语句 `int width{editWidth > 0 || trailingBlanks_ ? editWidth : totalLength};`。
- **L562 EN**: Introduces conditional control flow with an `if` statement.
  **L562 CN**: 通过 `if` 语句引入条件控制流。
- **L563 EN**: Returns from the current function, often propagating a computed result.
  **L563 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L564 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L564 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L565 EN**: Introduces conditional control flow with an `if` statement.
  **L565 CN**: 通过 `if` 语句引入条件控制流。
- **L566 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L566 CN**: 延续周围的声明、表达式或控制流结构。
- **L567 EN**: Initializes or updates `zeroesBeforePoint`.
  **L567 CN**: 初始化或更新 `zeroesBeforePoint`。
- **L568 EN**: Executes statement `++totalLength;`.
  **L568 CN**: 执行语句 `++totalLength;`。
- **L569 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L569 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L570 EN**: Returns from the current function, often propagating a computed result.
  **L570 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L571 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L571 CN**: 延续周围的声明、表达式或控制流结构。
- **L572 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L572 CN**: 延续周围的声明、表达式或控制流结构。
- **L573 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L573 CN**: 延续周围的声明、表达式或控制流结构。
- **L574 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L574 CN**: 延续周围的声明、表达式或控制流结构。
- **L575 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L575 CN**: 延续周围的声明、表达式或控制流结构。
- **L576 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L576 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 577-600

````cpp
        EmitRepeated(io_, '0', trailingZeroes) &&
        EmitRepeated(io_, ' ', trailingBlanks_) && EmitSuffix(edit);
  }
}

// 13.7.5.2.3 in F'2018
template <int KIND>
RT_API_ATTRS DataEdit RealOutputEditing<KIND>::EditForGOutput(DataEdit edit) {
  edit.descriptor = 'E';
  edit.variation = 'G'; // to suppress error for Ew.0
  int editWidth{edit.width.value_or(0)};
  int significantDigits{edit.digits.value_or(
      static_cast<int>(BinaryFloatingPoint::decimalPrecision))}; // 'd'
  if (editWidth > 0 && significantDigits == 0) {
    return edit; // Gw.0Ee -> Ew.0Ee for w > 0
  }
  int flags{0};
  if (edit.modes.editingFlags & signPlus) {
    flags |= decimal::AlwaysSign;
  }
  decimal::ConversionToDecimalResult converted{
      ConvertToDecimal(significantDigits, edit.modes.round, editWidth, flags)};
  if (IsInfOrNaN(converted.str, converted.length)) {
    return edit; // Inf/Nan -> Ew.d (same as Fw.d)
````

- **L577 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L577 CN**: 延续周围的声明、表达式或控制流结构。
- **L578 EN**: Executes statement involving `EmitRepeated`.
  **L578 CN**: 执行涉及 `EmitRepeated` 的语句。
- **L579 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L579 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L580 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L580 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L581 EN**: Blank line separates nearby declarations or logic blocks.
  **L581 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L582 EN**: Comment documents intent or context: `13.7.5.2.3 in F'2018`.
  **L582 CN**: 注释记录了意图或上下文：`13.7.5.2.3 in F'2018`。
- **L583 EN**: Begins a template declaration parameterizing subsequent code.
  **L583 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L584 EN**: Declares or defines callable `EditForGOutput`.
  **L584 CN**: 声明或定义可调用实体 `EditForGOutput`。
- **L585 EN**: Initializes or updates `edit.descriptor`.
  **L585 CN**: 初始化或更新 `edit.descriptor`。
- **L586 EN**: Initializes or updates `edit.variation`.
  **L586 CN**: 初始化或更新 `edit.variation`。
- **L587 EN**: Executes statement involving `value_or`.
  **L587 CN**: 执行涉及 `value_or` 的语句。
- **L588 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L588 CN**: 延续周围的声明、表达式或控制流结构。
- **L589 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L589 CN**: 延续周围的声明、表达式或控制流结构。
- **L590 EN**: Introduces conditional control flow with an `if` statement.
  **L590 CN**: 通过 `if` 语句引入条件控制流。
- **L591 EN**: Returns from the current function, often propagating a computed result.
  **L591 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L592 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L592 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L593 EN**: Executes statement `int flags{0};`.
  **L593 CN**: 执行语句 `int flags{0};`。
- **L594 EN**: Introduces conditional control flow with an `if` statement.
  **L594 CN**: 通过 `if` 语句引入条件控制流。
- **L595 EN**: Initializes or updates `|`.
  **L595 CN**: 初始化或更新 `|`。
- **L596 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L596 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L597 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L597 CN**: 延续周围的声明、表达式或控制流结构。
- **L598 EN**: Executes statement involving `ConvertToDecimal`.
  **L598 CN**: 执行涉及 `ConvertToDecimal` 的语句。
- **L599 EN**: Introduces conditional control flow with an `if` statement.
  **L599 CN**: 通过 `if` 语句引入条件控制流。
- **L600 EN**: Returns from the current function, often propagating a computed result.
  **L600 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 601-624

````cpp
  }
  int expo{IsZero() ? 1 : converted.decimalExponent}; // 's'
  if (expo < 0 || expo > significantDigits) {
    if (editWidth == 0 && !edit.expoDigits) { // G0.d -> G0.dE0
      edit.expoDigits = 0;
    }
    return edit; // Ew.dEe
  }
  edit.descriptor = 'F';
  edit.modes.scale = 0; // kP is ignored for G when no exponent field
  trailingBlanks_ = 0;
  if (editWidth > 0) {
    int expoDigits{edit.expoDigits.value_or(0)};
    // F'2023 13.7.5.2.3 p5: "If 0 <= s <= d, the scale factor has no effect
    // and F(w − n).(d − s),n(’b’) editing is used where b is a blank and
    // n is 4 for Gw.d editing, e + 2 for Gw.dEe editing if e > 0, and
    // 4 for Gw.dE0 editing."
    trailingBlanks_ = expoDigits > 0 ? expoDigits + 2 : 4; // 'n'
  }
  if (edit.digits.has_value()) {
    *edit.digits = std::max(0, *edit.digits - expo);
  }
  return edit;
}
````

- **L601 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L601 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L602 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L602 CN**: 延续周围的声明、表达式或控制流结构。
- **L603 EN**: Introduces conditional control flow with an `if` statement.
  **L603 CN**: 通过 `if` 语句引入条件控制流。
- **L604 EN**: Introduces conditional control flow with an `if` statement.
  **L604 CN**: 通过 `if` 语句引入条件控制流。
- **L605 EN**: Initializes or updates `edit.expoDigits`.
  **L605 CN**: 初始化或更新 `edit.expoDigits`。
- **L606 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L606 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L607 EN**: Returns from the current function, often propagating a computed result.
  **L607 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L608 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L608 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L609 EN**: Initializes or updates `edit.descriptor`.
  **L609 CN**: 初始化或更新 `edit.descriptor`。
- **L610 EN**: Initializes or updates `edit.modes.scale`.
  **L610 CN**: 初始化或更新 `edit.modes.scale`。
- **L611 EN**: Initializes or updates `trailingBlanks_`.
  **L611 CN**: 初始化或更新 `trailingBlanks_`。
- **L612 EN**: Introduces conditional control flow with an `if` statement.
  **L612 CN**: 通过 `if` 语句引入条件控制流。
- **L613 EN**: Executes statement involving `value_or`.
  **L613 CN**: 执行涉及 `value_or` 的语句。
- **L614 EN**: Comment documents intent or context: `F'2023 13.7.5.2.3 p5: "If 0 <= s <= d, the scale factor has no effect`.
  **L614 CN**: 注释记录了意图或上下文：`F'2023 13.7.5.2.3 p5: "If 0 <= s <= d, the scale factor has no effect`。
- **L615 EN**: Comment documents intent or context: `and F(w − n).(d − s),n(’b’) editing is used where b is a blank and`.
  **L615 CN**: 注释记录了意图或上下文：`and F(w − n).(d − s),n(’b’) editing is used where b is a blank and`。
- **L616 EN**: Comment documents intent or context: `n is 4 for Gw.d editing, e + 2 for Gw.dEe editing if e > 0, and`.
  **L616 CN**: 注释记录了意图或上下文：`n is 4 for Gw.d editing, e + 2 for Gw.dEe editing if e > 0, and`。
- **L617 EN**: Comment documents intent or context: `4 for Gw.dE0 editing."`.
  **L617 CN**: 注释记录了意图或上下文：`4 for Gw.dE0 editing."`。
- **L618 EN**: Initializes or updates `trailingBlanks_`.
  **L618 CN**: 初始化或更新 `trailingBlanks_`。
- **L619 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L619 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L620 EN**: Introduces conditional control flow with an `if` statement.
  **L620 CN**: 通过 `if` 语句引入条件控制流。
- **L621 EN**: Comment documents intent or context: `edit.digits = std::max(0, *edit.digits - expo);`.
  **L621 CN**: 注释记录了意图或上下文：`edit.digits = std::max(0, *edit.digits - expo);`。
- **L622 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L622 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L623 EN**: Returns from the current function, often propagating a computed result.
  **L623 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L624 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L624 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 625-648

````cpp

// 13.10.4 in F'2018
template <int KIND>
RT_API_ATTRS bool RealOutputEditing<KIND>::EditListDirectedOutput(
    const DataEdit &edit) {
  decimal::ConversionToDecimalResult converted{
      ConvertToDecimal(1, edit.modes.round)};
  if (IsInfOrNaN(converted.str, converted.length)) {
    DataEdit copy{edit};
    copy.variation = DataEdit::ListDirected;
    return EditEorDOutput(copy);
  }
  int expo{converted.decimalExponent};
  // The decimal precision of 16-bit floating-point types is very low,
  // so use a reasonable cap of 6 to allow more values to be emitted
  // with Fw.d editing.
  static constexpr int maxExpo{
      std::max(6, BinaryFloatingPoint::decimalPrecision)};
  if (expo < 0 || expo > maxExpo) {
    DataEdit copy{edit};
    copy.variation = DataEdit::ListDirected;
    copy.modes.scale = 1; // 1P
    return EditEorDOutput(copy);
  } else {
````

- **L625 EN**: Blank line separates nearby declarations or logic blocks.
  **L625 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L626 EN**: Comment documents intent or context: `13.10.4 in F'2018`.
  **L626 CN**: 注释记录了意图或上下文：`13.10.4 in F'2018`。
- **L627 EN**: Begins a template declaration parameterizing subsequent code.
  **L627 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L628 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L628 CN**: 延续周围的声明、表达式或控制流结构。
- **L629 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L629 CN**: 延续周围的声明、表达式或控制流结构。
- **L630 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L630 CN**: 延续周围的声明、表达式或控制流结构。
- **L631 EN**: Executes statement involving `ConvertToDecimal`.
  **L631 CN**: 执行涉及 `ConvertToDecimal` 的语句。
- **L632 EN**: Introduces conditional control flow with an `if` statement.
  **L632 CN**: 通过 `if` 语句引入条件控制流。
- **L633 EN**: Executes statement `DataEdit copy{edit};`.
  **L633 CN**: 执行语句 `DataEdit copy{edit};`。
- **L634 EN**: Initializes or updates `copy.variation`.
  **L634 CN**: 初始化或更新 `copy.variation`。
- **L635 EN**: Returns from the current function, often propagating a computed result.
  **L635 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L636 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L636 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L637 EN**: Executes statement `int expo{converted.decimalExponent};`.
  **L637 CN**: 执行语句 `int expo{converted.decimalExponent};`。
- **L638 EN**: Comment documents intent or context: `The decimal precision of 16-bit floating-point types is very low,`.
  **L638 CN**: 注释记录了意图或上下文：`The decimal precision of 16-bit floating-point types is very low,`。
- **L639 EN**: Comment documents intent or context: `so use a reasonable cap of 6 to allow more values to be emitted`.
  **L639 CN**: 注释记录了意图或上下文：`so use a reasonable cap of 6 to allow more values to be emitted`。
- **L640 EN**: Comment documents intent or context: `with Fw.d editing.`.
  **L640 CN**: 注释记录了意图或上下文：`with Fw.d editing.`。
- **L641 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L641 CN**: 延续周围的声明、表达式或控制流结构。
- **L642 EN**: Executes statement involving `max`.
  **L642 CN**: 执行涉及 `max` 的语句。
- **L643 EN**: Introduces conditional control flow with an `if` statement.
  **L643 CN**: 通过 `if` 语句引入条件控制流。
- **L644 EN**: Executes statement `DataEdit copy{edit};`.
  **L644 CN**: 执行语句 `DataEdit copy{edit};`。
- **L645 EN**: Initializes or updates `copy.variation`.
  **L645 CN**: 初始化或更新 `copy.variation`。
- **L646 EN**: Initializes or updates `copy.modes.scale`.
  **L646 CN**: 初始化或更新 `copy.modes.scale`。
- **L647 EN**: Returns from the current function, often propagating a computed result.
  **L647 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L648 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L648 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 649-672

````cpp
    return EditFOutput(edit);
  }
}

// 13.7.2.3.6 in F'2023
// The specification for hexadecimal output, unfortunately for implementors,
// leaves as "implementation dependent" the choice of how to emit values
// with multiple hexadecimal output possibilities that are numerically
// equivalent.  The one working implementation of EX output that I can find
// apparently chooses to frame the nybbles from most to least significant,
// rather than trying to minimize the magnitude of the binary exponent.
// E.g., 2. is edited into 0X8.0P-2 rather than 0X2.0P0.  This implementation
// follows that precedent so as to avoid a gratuitous incompatibility.
template <int KIND>
RT_API_ATTRS auto RealOutputEditing<KIND>::ConvertToHexadecimal(
    int significantDigits, enum decimal::FortranRounding rounding,
    int editWidth, int flags) -> ConvertToHexadecimalResult {
  if (x_.IsNaN() || x_.IsInfinite()) {
    auto converted{
        ConvertToDecimal(significantDigits, rounding, editWidth, flags)};
    return {converted.str, static_cast<int>(converted.length), /*exponent=*/0};
  }
  x_.RoundToBits(4 * significantDigits, rounding);
  if (x_.IsInfinite()) { // rounded away to +/-Inf
````

- **L649 EN**: Returns from the current function, often propagating a computed result.
  **L649 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L650 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L650 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L651 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L651 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L652 EN**: Blank line separates nearby declarations or logic blocks.
  **L652 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L653 EN**: Comment documents intent or context: `13.7.2.3.6 in F'2023`.
  **L653 CN**: 注释记录了意图或上下文：`13.7.2.3.6 in F'2023`。
- **L654 EN**: Comment documents intent or context: `The specification for hexadecimal output, unfortunately for implementors,`.
  **L654 CN**: 注释记录了意图或上下文：`The specification for hexadecimal output, unfortunately for implementors,`。
- **L655 EN**: Comment documents intent or context: `leaves as "implementation dependent" the choice of how to emit values`.
  **L655 CN**: 注释记录了意图或上下文：`leaves as "implementation dependent" the choice of how to emit values`。
- **L656 EN**: Comment documents intent or context: `with multiple hexadecimal output possibilities that are numerically`.
  **L656 CN**: 注释记录了意图或上下文：`with multiple hexadecimal output possibilities that are numerically`。
- **L657 EN**: Comment documents intent or context: `equivalent. The one working implementation of EX output that I can find`.
  **L657 CN**: 注释记录了意图或上下文：`equivalent. The one working implementation of EX output that I can find`。
- **L658 EN**: Comment documents intent or context: `apparently chooses to frame the nybbles from most to least significant,`.
  **L658 CN**: 注释记录了意图或上下文：`apparently chooses to frame the nybbles from most to least significant,`。
- **L659 EN**: Comment documents intent or context: `rather than trying to minimize the magnitude of the binary exponent.`.
  **L659 CN**: 注释记录了意图或上下文：`rather than trying to minimize the magnitude of the binary exponent.`。
- **L660 EN**: Comment documents intent or context: `E.g., 2. is edited into 0X8.0P-2 rather than 0X2.0P0. This implementation`.
  **L660 CN**: 注释记录了意图或上下文：`E.g., 2. is edited into 0X8.0P-2 rather than 0X2.0P0. This implementation`。
- **L661 EN**: Comment documents intent or context: `follows that precedent so as to avoid a gratuitous incompatibility.`.
  **L661 CN**: 注释记录了意图或上下文：`follows that precedent so as to avoid a gratuitous incompatibility.`。
- **L662 EN**: Begins a template declaration parameterizing subsequent code.
  **L662 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L663 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L663 CN**: 延续周围的声明、表达式或控制流结构。
- **L664 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L664 CN**: 延续周围的声明、表达式或控制流结构。
- **L665 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L665 CN**: 延续周围的声明、表达式或控制流结构。
- **L666 EN**: Introduces conditional control flow with an `if` statement.
  **L666 CN**: 通过 `if` 语句引入条件控制流。
- **L667 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L667 CN**: 延续周围的声明、表达式或控制流结构。
- **L668 EN**: Executes statement involving `ConvertToDecimal`.
  **L668 CN**: 执行涉及 `ConvertToDecimal` 的语句。
- **L669 EN**: Returns from the current function, often propagating a computed result.
  **L669 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L670 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L670 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L671 EN**: Executes statement involving `RoundToBits`.
  **L671 CN**: 执行涉及 `RoundToBits` 的语句。
- **L672 EN**: Introduces conditional control flow with an `if` statement.
  **L672 CN**: 通过 `if` 语句引入条件控制流。

### Lines 673-696

````cpp
    auto converted{
        ConvertToDecimal(significantDigits, rounding, editWidth, flags)};
    return {converted.str, static_cast<int>(converted.length), /*exponent=*/0};
  }
  int len{0};
  if (x_.IsNegative()) {
    buffer_[len++] = '-';
  } else if (flags & decimal::AlwaysSign) {
    buffer_[len++] = '+';
  }
  auto fraction{x_.Fraction()};
  if (fraction == 0) {
    buffer_[len++] = '0';
    return {buffer_, len, 0};
  } else {
    // Ensure that the MSB is set.
    int expo{x_.UnbiasedExponent() - 3};
    while (!(fraction >> (x_.binaryPrecision - 1))) {
      fraction <<= 1;
      --expo;
    }
    // This is initially the right shift count needed to bring the
    // most-significant hexadecimal digit's bits into the LSBs.
    // x_.binaryPrecision is constant, so / can be used for readability.
````

- **L673 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L673 CN**: 延续周围的声明、表达式或控制流结构。
- **L674 EN**: Executes statement involving `ConvertToDecimal`.
  **L674 CN**: 执行涉及 `ConvertToDecimal` 的语句。
- **L675 EN**: Returns from the current function, often propagating a computed result.
  **L675 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L676 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L676 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L677 EN**: Executes statement `int len{0};`.
  **L677 CN**: 执行语句 `int len{0};`。
- **L678 EN**: Introduces conditional control flow with an `if` statement.
  **L678 CN**: 通过 `if` 语句引入条件控制流。
- **L679 EN**: Initializes or updates `buffer_[len++]`.
  **L679 CN**: 初始化或更新 `buffer_[len++]`。
- **L680 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L680 CN**: 延续周围的声明、表达式或控制流结构。
- **L681 EN**: Initializes or updates `buffer_[len++]`.
  **L681 CN**: 初始化或更新 `buffer_[len++]`。
- **L682 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L682 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L683 EN**: Executes statement involving `Fraction`.
  **L683 CN**: 执行涉及 `Fraction` 的语句。
- **L684 EN**: Introduces conditional control flow with an `if` statement.
  **L684 CN**: 通过 `if` 语句引入条件控制流。
- **L685 EN**: Initializes or updates `buffer_[len++]`.
  **L685 CN**: 初始化或更新 `buffer_[len++]`。
- **L686 EN**: Returns from the current function, often propagating a computed result.
  **L686 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L687 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L687 CN**: 延续周围的声明、表达式或控制流结构。
- **L688 EN**: Comment documents intent or context: `Ensure that the MSB is set.`.
  **L688 CN**: 注释记录了意图或上下文：`Ensure that the MSB is set.`。
- **L689 EN**: Executes statement involving `UnbiasedExponent`.
  **L689 CN**: 执行涉及 `UnbiasedExponent` 的语句。
- **L690 EN**: Starts a `while` loop controlled by a runtime condition.
  **L690 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L691 EN**: Executes statement `fraction <<= 1;`.
  **L691 CN**: 执行语句 `fraction <<= 1;`。
- **L692 EN**: Executes statement `--expo;`.
  **L692 CN**: 执行语句 `--expo;`。
- **L693 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L693 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L694 EN**: Comment documents intent or context: `This is initially the right shift count needed to bring the`.
  **L694 CN**: 注释记录了意图或上下文：`This is initially the right shift count needed to bring the`。
- **L695 EN**: Comment documents intent or context: `most-significant hexadecimal digit's bits into the LSBs.`.
  **L695 CN**: 注释记录了意图或上下文：`most-significant hexadecimal digit's bits into the LSBs.`。
- **L696 EN**: Comment documents intent or context: `x_.binaryPrecision is constant, so / can be used for readability.`.
  **L696 CN**: 注释记录了意图或上下文：`x_.binaryPrecision is constant, so / can be used for readability.`。

### Lines 697-720

````cpp
    int shift{x_.binaryPrecision - 4};
    typename BinaryFloatingPoint::RawType one{1};
    auto remaining{(one << x_.binaryPrecision) - one};
    for (int digits{0}; digits < significantDigits; ++digits) {
      if ((flags & decimal::Minimize) && !(fraction & remaining)) {
        break;
      }
      int hexDigit{0};
      if (shift >= 0) {
        hexDigit = int(fraction >> shift) & 0xf;
      } else if (shift >= -3) {
        hexDigit = int(fraction << -shift) & 0xf;
      }
      if (hexDigit >= 10) {
        buffer_[len++] = 'A' + hexDigit - 10;
      } else {
        buffer_[len++] = '0' + hexDigit;
      }
      shift -= 4;
      remaining >>= 4;
    }
    return {buffer_, len, expo};
  }
}
````

- **L697 EN**: Executes statement `int shift{x_.binaryPrecision - 4};`.
  **L697 CN**: 执行语句 `int shift{x_.binaryPrecision - 4};`。
- **L698 EN**: Executes statement `typename BinaryFloatingPoint::RawType one{1};`.
  **L698 CN**: 执行语句 `typename BinaryFloatingPoint::RawType one{1};`。
- **L699 EN**: Executes statement `auto remaining{(one << x_.binaryPrecision) - one};`.
  **L699 CN**: 执行语句 `auto remaining{(one << x_.binaryPrecision) - one};`。
- **L700 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L700 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L701 EN**: Introduces conditional control flow with an `if` statement.
  **L701 CN**: 通过 `if` 语句引入条件控制流。
- **L702 EN**: Breaks out of the current loop or switch.
  **L702 CN**: 跳出当前循环或 switch。
- **L703 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L703 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L704 EN**: Executes statement `int hexDigit{0};`.
  **L704 CN**: 执行语句 `int hexDigit{0};`。
- **L705 EN**: Introduces conditional control flow with an `if` statement.
  **L705 CN**: 通过 `if` 语句引入条件控制流。
- **L706 EN**: Initializes or updates `hexDigit`.
  **L706 CN**: 初始化或更新 `hexDigit`。
- **L707 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L707 CN**: 延续周围的声明、表达式或控制流结构。
- **L708 EN**: Initializes or updates `hexDigit`.
  **L708 CN**: 初始化或更新 `hexDigit`。
- **L709 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L709 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L710 EN**: Introduces conditional control flow with an `if` statement.
  **L710 CN**: 通过 `if` 语句引入条件控制流。
- **L711 EN**: Initializes or updates `buffer_[len++]`.
  **L711 CN**: 初始化或更新 `buffer_[len++]`。
- **L712 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L712 CN**: 延续周围的声明、表达式或控制流结构。
- **L713 EN**: Initializes or updates `buffer_[len++]`.
  **L713 CN**: 初始化或更新 `buffer_[len++]`。
- **L714 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L714 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L715 EN**: Initializes or updates `-`.
  **L715 CN**: 初始化或更新 `-`。
- **L716 EN**: Executes statement `remaining >>= 4;`.
  **L716 CN**: 执行语句 `remaining >>= 4;`。
- **L717 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L717 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L718 EN**: Returns from the current function, often propagating a computed result.
  **L718 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L719 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L719 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L720 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L720 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 721-744

````cpp

template <int KIND>
RT_API_ATTRS bool RealOutputEditing<KIND>::EditEXOutput(const DataEdit &edit) {
  AddSpaceBeforeCharacter(io_);
  int editDigits{edit.digits.value_or(0)}; // 'd' field
  int significantDigits{editDigits + 1};
  int flags{0};
  if (edit.modes.editingFlags & signPlus) {
    flags |= decimal::AlwaysSign;
  }
  int editWidth{edit.width.value_or(0)}; // 'w' field
  if ((editWidth == 0 && !edit.digits) || editDigits == 0) {
    // EX0 or EXw.0
    flags |= decimal::Minimize;
    static constexpr int maxSigHexDigits{
        (common::PrecisionOfRealKind(16) + 3) / 4};
    significantDigits = maxSigHexDigits;
  }
  auto converted{ConvertToHexadecimal(
      significantDigits, edit.modes.round, editWidth, flags)};
  if (IsInfOrNaN(converted.str, converted.length)) {
    return editWidth > 0 && converted.length > editWidth
        ? EmitRepeated(io_, '*', editWidth)
        : (editWidth <= converted.length ||
````

- **L721 EN**: Blank line separates nearby declarations or logic blocks.
  **L721 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L722 EN**: Begins a template declaration parameterizing subsequent code.
  **L722 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L723 EN**: Declares or defines callable `EditEXOutput`.
  **L723 CN**: 声明或定义可调用实体 `EditEXOutput`。
- **L724 EN**: Executes statement involving `AddSpaceBeforeCharacter`.
  **L724 CN**: 执行涉及 `AddSpaceBeforeCharacter` 的语句。
- **L725 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L725 CN**: 延续周围的声明、表达式或控制流结构。
- **L726 EN**: Executes statement `int significantDigits{editDigits + 1};`.
  **L726 CN**: 执行语句 `int significantDigits{editDigits + 1};`。
- **L727 EN**: Executes statement `int flags{0};`.
  **L727 CN**: 执行语句 `int flags{0};`。
- **L728 EN**: Introduces conditional control flow with an `if` statement.
  **L728 CN**: 通过 `if` 语句引入条件控制流。
- **L729 EN**: Initializes or updates `|`.
  **L729 CN**: 初始化或更新 `|`。
- **L730 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L730 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L731 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L731 CN**: 延续周围的声明、表达式或控制流结构。
- **L732 EN**: Introduces conditional control flow with an `if` statement.
  **L732 CN**: 通过 `if` 语句引入条件控制流。
- **L733 EN**: Comment documents intent or context: `EX0 or EXw.0`.
  **L733 CN**: 注释记录了意图或上下文：`EX0 or EXw.0`。
- **L734 EN**: Initializes or updates `|`.
  **L734 CN**: 初始化或更新 `|`。
- **L735 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L735 CN**: 延续周围的声明、表达式或控制流结构。
- **L736 EN**: Executes statement involving `PrecisionOfRealKind`.
  **L736 CN**: 执行涉及 `PrecisionOfRealKind` 的语句。
- **L737 EN**: Initializes or updates `significantDigits`.
  **L737 CN**: 初始化或更新 `significantDigits`。
- **L738 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L738 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L739 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L739 CN**: 延续周围的声明、表达式或控制流结构。
- **L740 EN**: Executes statement `significantDigits, edit.modes.round, editWidth, flags)};`.
  **L740 CN**: 执行语句 `significantDigits, edit.modes.round, editWidth, flags)};`。
- **L741 EN**: Introduces conditional control flow with an `if` statement.
  **L741 CN**: 通过 `if` 语句引入条件控制流。
- **L742 EN**: Returns from the current function, often propagating a computed result.
  **L742 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L743 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L743 CN**: 延续周围的声明、表达式或控制流结构。
- **L744 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L744 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 745-768

````cpp
              EmitRepeated(io_, ' ', editWidth - converted.length)) &&
            EmitAscii(io_, converted.str, converted.length);
  }
  int signLength{converted.length > 0 &&
              (converted.str[0] == '-' || converted.str[0] == '+')
          ? 1
          : 0};
  int convertedDigits{converted.length - signLength};
  int expoLength{0};
  const char *exponent{FormatExponent(converted.exponent, edit, expoLength)};
  int trailingZeroes{flags & decimal::Minimize
          ? 0
          : std::max(0, significantDigits - convertedDigits)};
  int totalLength{converted.length + trailingZeroes + expoLength + 3 /*0X.*/};
  int width{editWidth > 0 ? editWidth : totalLength};
  return totalLength > width || !exponent
      ? EmitRepeated(io_, '*', width)
      : EmitRepeated(io_, ' ', width - totalLength) &&
          EmitAscii(io_, converted.str, signLength) &&
          EmitAscii(io_, "0X", 2) &&
          EmitAscii(io_, converted.str + signLength, 1) &&
          EmitAscii(
              io_, edit.modes.editingFlags & decimalComma ? "," : ".", 1) &&
          EmitAscii(io_, converted.str + signLength + 1,
````

- **L745 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L745 CN**: 延续周围的声明、表达式或控制流结构。
- **L746 EN**: Executes statement involving `EmitAscii`.
  **L746 CN**: 执行涉及 `EmitAscii` 的语句。
- **L747 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L747 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L748 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L748 CN**: 延续周围的声明、表达式或控制流结构。
- **L749 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L749 CN**: 延续周围的声明、表达式或控制流结构。
- **L750 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L750 CN**: 延续周围的声明、表达式或控制流结构。
- **L751 EN**: Executes statement `: 0};`.
  **L751 CN**: 执行语句 `: 0};`。
- **L752 EN**: Executes statement `int convertedDigits{converted.length - signLength};`.
  **L752 CN**: 执行语句 `int convertedDigits{converted.length - signLength};`。
- **L753 EN**: Executes statement `int expoLength{0};`.
  **L753 CN**: 执行语句 `int expoLength{0};`。
- **L754 EN**: Executes statement involving `FormatExponent`.
  **L754 CN**: 执行涉及 `FormatExponent` 的语句。
- **L755 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L755 CN**: 延续周围的声明、表达式或控制流结构。
- **L756 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L756 CN**: 延续周围的声明、表达式或控制流结构。
- **L757 EN**: Executes statement involving `max`.
  **L757 CN**: 执行涉及 `max` 的语句。
- **L758 EN**: Executes statement `int totalLength{converted.length + trailingZeroes + expoLength + 3 /*0X.*/};`.
  **L758 CN**: 执行语句 `int totalLength{converted.length + trailingZeroes + expoLength + 3 /*0X.*/};`。
- **L759 EN**: Executes statement `int width{editWidth > 0 ? editWidth : totalLength};`.
  **L759 CN**: 执行语句 `int width{editWidth > 0 ? editWidth : totalLength};`。
- **L760 EN**: Returns from the current function, often propagating a computed result.
  **L760 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L761 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L761 CN**: 延续周围的声明、表达式或控制流结构。
- **L762 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L762 CN**: 延续周围的声明、表达式或控制流结构。
- **L763 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L763 CN**: 延续周围的声明、表达式或控制流结构。
- **L764 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L764 CN**: 延续周围的声明、表达式或控制流结构。
- **L765 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L765 CN**: 延续周围的声明、表达式或控制流结构。
- **L766 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L766 CN**: 延续周围的声明、表达式或控制流结构。
- **L767 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L767 CN**: 延续周围的声明、表达式或控制流结构。
- **L768 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L768 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 769-792

````cpp
              converted.length - (signLength + 1)) &&
          EmitRepeated(io_, '0', trailingZeroes) &&
          EmitAscii(io_, exponent, expoLength);
}

template <int KIND>
RT_API_ATTRS bool RealOutputEditing<KIND>::Edit(const DataEdit &edit) {
  const DataEdit *editPtr{&edit};
  DataEdit newEdit;
  if (editPtr->descriptor == 'G') {
    // Avoid recursive call as in Edit(EditForGOutput(edit)).
    newEdit = EditForGOutput(*editPtr);
    editPtr = &newEdit;
    RUNTIME_CHECK(io_.GetIoErrorHandler(), editPtr->descriptor != 'G');
  }
  switch (editPtr->descriptor) {
  case 'D':
    return EditEorDOutput(*editPtr);
  case 'E':
    if (editPtr->variation == 'X') {
      return EditEXOutput(*editPtr);
    } else {
      return EditEorDOutput(*editPtr);
    }
````

- **L769 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L769 CN**: 延续周围的声明、表达式或控制流结构。
- **L770 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L770 CN**: 延续周围的声明、表达式或控制流结构。
- **L771 EN**: Executes statement involving `EmitAscii`.
  **L771 CN**: 执行涉及 `EmitAscii` 的语句。
- **L772 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L772 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L773 EN**: Blank line separates nearby declarations or logic blocks.
  **L773 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L774 EN**: Begins a template declaration parameterizing subsequent code.
  **L774 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L775 EN**: Declares or defines callable `Edit`.
  **L775 CN**: 声明或定义可调用实体 `Edit`。
- **L776 EN**: Executes statement `const DataEdit *editPtr{&edit};`.
  **L776 CN**: 执行语句 `const DataEdit *editPtr{&edit};`。
- **L777 EN**: Executes statement `DataEdit newEdit;`.
  **L777 CN**: 执行语句 `DataEdit newEdit;`。
- **L778 EN**: Introduces conditional control flow with an `if` statement.
  **L778 CN**: 通过 `if` 语句引入条件控制流。
- **L779 EN**: Comment documents intent or context: `Avoid recursive call as in Edit(EditForGOutput(edit)).`.
  **L779 CN**: 注释记录了意图或上下文：`Avoid recursive call as in Edit(EditForGOutput(edit)).`。
- **L780 EN**: Initializes or updates `newEdit`.
  **L780 CN**: 初始化或更新 `newEdit`。
- **L781 EN**: Initializes or updates `editPtr`.
  **L781 CN**: 初始化或更新 `editPtr`。
- **L782 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L782 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L783 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L783 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L784 EN**: Begins a `switch` dispatch over discrete cases.
  **L784 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L785 EN**: Marks one `switch` case label.
  **L785 CN**: 标记一个 `switch` 的 case 标签。
- **L786 EN**: Returns from the current function, often propagating a computed result.
  **L786 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L787 EN**: Marks one `switch` case label.
  **L787 CN**: 标记一个 `switch` 的 case 标签。
- **L788 EN**: Introduces conditional control flow with an `if` statement.
  **L788 CN**: 通过 `if` 语句引入条件控制流。
- **L789 EN**: Returns from the current function, often propagating a computed result.
  **L789 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L790 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L790 CN**: 延续周围的声明、表达式或控制流结构。
- **L791 EN**: Returns from the current function, often propagating a computed result.
  **L791 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L792 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L792 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 793-816

````cpp
  case 'F':
    return EditFOutput(*editPtr);
  case 'B':
    return EditBOZOutput<1>(io_, *editPtr,
        reinterpret_cast<const unsigned char *>(&x_),
        common::BitsForBinaryPrecision(common::PrecisionOfRealKind(KIND)) >> 3);
  case 'O':
    return EditBOZOutput<3>(io_, *editPtr,
        reinterpret_cast<const unsigned char *>(&x_),
        common::BitsForBinaryPrecision(common::PrecisionOfRealKind(KIND)) >> 3);
  case 'Z':
    return EditBOZOutput<4>(io_, *editPtr,
        reinterpret_cast<const unsigned char *>(&x_),
        common::BitsForBinaryPrecision(common::PrecisionOfRealKind(KIND)) >> 3);
  case 'L':
    return EditLogicalOutput(
        io_, *editPtr, *reinterpret_cast<const char *>(&x_));
  case 'A': // legacy extension
    return EditCharacterOutput(
        io_, *editPtr, reinterpret_cast<char *>(&x_), sizeof x_);
  default:
    if (editPtr->IsListDirected()) {
      return EditListDirectedOutput(*editPtr);
    }
````

- **L793 EN**: Marks one `switch` case label.
  **L793 CN**: 标记一个 `switch` 的 case 标签。
- **L794 EN**: Returns from the current function, often propagating a computed result.
  **L794 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L795 EN**: Marks one `switch` case label.
  **L795 CN**: 标记一个 `switch` 的 case 标签。
- **L796 EN**: Returns from the current function, often propagating a computed result.
  **L796 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L797 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L797 CN**: 延续周围的声明、表达式或控制流结构。
- **L798 EN**: Executes statement involving `BitsForBinaryPrecision`.
  **L798 CN**: 执行涉及 `BitsForBinaryPrecision` 的语句。
- **L799 EN**: Marks one `switch` case label.
  **L799 CN**: 标记一个 `switch` 的 case 标签。
- **L800 EN**: Returns from the current function, often propagating a computed result.
  **L800 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L801 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L801 CN**: 延续周围的声明、表达式或控制流结构。
- **L802 EN**: Executes statement involving `BitsForBinaryPrecision`.
  **L802 CN**: 执行涉及 `BitsForBinaryPrecision` 的语句。
- **L803 EN**: Marks one `switch` case label.
  **L803 CN**: 标记一个 `switch` 的 case 标签。
- **L804 EN**: Returns from the current function, often propagating a computed result.
  **L804 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L805 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L805 CN**: 延续周围的声明、表达式或控制流结构。
- **L806 EN**: Executes statement involving `BitsForBinaryPrecision`.
  **L806 CN**: 执行涉及 `BitsForBinaryPrecision` 的语句。
- **L807 EN**: Marks one `switch` case label.
  **L807 CN**: 标记一个 `switch` 的 case 标签。
- **L808 EN**: Returns from the current function, often propagating a computed result.
  **L808 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L809 EN**: Executes statement `io_, *editPtr, *reinterpret_cast<const char *>(&x_));`.
  **L809 CN**: 执行语句 `io_, *editPtr, *reinterpret_cast<const char *>(&x_));`。
- **L810 EN**: Marks one `switch` case label.
  **L810 CN**: 标记一个 `switch` 的 case 标签。
- **L811 EN**: Returns from the current function, often propagating a computed result.
  **L811 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L812 EN**: Executes statement `io_, *editPtr, reinterpret_cast<char *>(&x_), sizeof x_);`.
  **L812 CN**: 执行语句 `io_, *editPtr, reinterpret_cast<char *>(&x_), sizeof x_);`。
- **L813 EN**: Provides the default branch for a `switch` statement.
  **L813 CN**: 为 `switch` 语句提供默认分支。
- **L814 EN**: Introduces conditional control flow with an `if` statement.
  **L814 CN**: 通过 `if` 语句引入条件控制流。
- **L815 EN**: Returns from the current function, often propagating a computed result.
  **L815 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L816 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L816 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 817-840

````cpp
    io_.GetIoErrorHandler().SignalError(IostatErrorInFormat,
        "Data edit descriptor '%c' may not be used with a REAL data item",
        editPtr->descriptor);
    return false;
  }
  return false;
}

RT_API_ATTRS bool ListDirectedLogicalOutput(IoStatementState &io,
    ListDirectedStatementState<Direction::Output> &list, bool truth) {
  return list.EmitLeadingSpaceOrAdvance(io) &&
      EmitAscii(io, truth ? "T" : "F", 1);
}

RT_API_ATTRS bool EditLogicalOutput(
    IoStatementState &io, const DataEdit &edit, bool truth) {
  switch (edit.descriptor) {
  case 'L':
  case 'G':
    return EmitRepeated(io, ' ', std::max(0, edit.width.value_or(1) - 1)) &&
        EmitAscii(io, truth ? "T" : "F", 1);
  case 'B':
    return EditBOZOutput<1>(io, edit,
        reinterpret_cast<const unsigned char *>(&truth), sizeof truth);
````

- **L817 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L817 CN**: 延续周围的声明、表达式或控制流结构。
- **L818 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L818 CN**: 延续周围的声明、表达式或控制流结构。
- **L819 EN**: Executes statement `editPtr->descriptor);`.
  **L819 CN**: 执行语句 `editPtr->descriptor);`。
- **L820 EN**: Returns from the current function, often propagating a computed result.
  **L820 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L821 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L821 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L822 EN**: Returns from the current function, often propagating a computed result.
  **L822 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L823 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L823 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L824 EN**: Blank line separates nearby declarations or logic blocks.
  **L824 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L825 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L825 CN**: 延续周围的声明、表达式或控制流结构。
- **L826 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L826 CN**: 延续周围的声明、表达式或控制流结构。
- **L827 EN**: Returns from the current function, often propagating a computed result.
  **L827 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L828 EN**: Executes statement involving `EmitAscii`.
  **L828 CN**: 执行涉及 `EmitAscii` 的语句。
- **L829 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L829 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L830 EN**: Blank line separates nearby declarations or logic blocks.
  **L830 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L831 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L831 CN**: 延续周围的声明、表达式或控制流结构。
- **L832 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L832 CN**: 延续周围的声明、表达式或控制流结构。
- **L833 EN**: Begins a `switch` dispatch over discrete cases.
  **L833 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L834 EN**: Marks one `switch` case label.
  **L834 CN**: 标记一个 `switch` 的 case 标签。
- **L835 EN**: Marks one `switch` case label.
  **L835 CN**: 标记一个 `switch` 的 case 标签。
- **L836 EN**: Returns from the current function, often propagating a computed result.
  **L836 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L837 EN**: Executes statement involving `EmitAscii`.
  **L837 CN**: 执行涉及 `EmitAscii` 的语句。
- **L838 EN**: Marks one `switch` case label.
  **L838 CN**: 标记一个 `switch` 的 case 标签。
- **L839 EN**: Returns from the current function, often propagating a computed result.
  **L839 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L840 EN**: Executes statement `reinterpret_cast<const unsigned char *>(&truth), sizeof truth);`.
  **L840 CN**: 执行语句 `reinterpret_cast<const unsigned char *>(&truth), sizeof truth);`。

### Lines 841-864

````cpp
  case 'O':
    return EditBOZOutput<3>(io, edit,
        reinterpret_cast<const unsigned char *>(&truth), sizeof truth);
  case 'Z':
    return EditBOZOutput<4>(io, edit,
        reinterpret_cast<const unsigned char *>(&truth), sizeof truth);
  case 'A': { // legacy extension
    int truthBits{truth};
    int len{sizeof truthBits};
    int width{edit.width.value_or(len)};
    return EmitRepeated(io, ' ', std::max(0, width - len)) &&
        EmitEncoded(
            io, reinterpret_cast<char *>(&truthBits), std::min(width, len));
  }
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInFormat,
        "Data edit descriptor '%c' may not be used with a LOGICAL data item",
        edit.descriptor);
    return false;
  }
}

template <typename CHAR>
RT_API_ATTRS bool ListDirectedCharacterOutput(IoStatementState &io,
````

- **L841 EN**: Marks one `switch` case label.
  **L841 CN**: 标记一个 `switch` 的 case 标签。
- **L842 EN**: Returns from the current function, often propagating a computed result.
  **L842 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L843 EN**: Executes statement `reinterpret_cast<const unsigned char *>(&truth), sizeof truth);`.
  **L843 CN**: 执行语句 `reinterpret_cast<const unsigned char *>(&truth), sizeof truth);`。
- **L844 EN**: Marks one `switch` case label.
  **L844 CN**: 标记一个 `switch` 的 case 标签。
- **L845 EN**: Returns from the current function, often propagating a computed result.
  **L845 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L846 EN**: Executes statement `reinterpret_cast<const unsigned char *>(&truth), sizeof truth);`.
  **L846 CN**: 执行语句 `reinterpret_cast<const unsigned char *>(&truth), sizeof truth);`。
- **L847 EN**: Marks one `switch` case label.
  **L847 CN**: 标记一个 `switch` 的 case 标签。
- **L848 EN**: Executes statement `int truthBits{truth};`.
  **L848 CN**: 执行语句 `int truthBits{truth};`。
- **L849 EN**: Executes statement `int len{sizeof truthBits};`.
  **L849 CN**: 执行语句 `int len{sizeof truthBits};`。
- **L850 EN**: Executes statement involving `value_or`.
  **L850 CN**: 执行涉及 `value_or` 的语句。
- **L851 EN**: Returns from the current function, often propagating a computed result.
  **L851 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L852 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L852 CN**: 延续周围的声明、表达式或控制流结构。
- **L853 EN**: Executes statement involving `min`.
  **L853 CN**: 执行涉及 `min` 的语句。
- **L854 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L854 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L855 EN**: Provides the default branch for a `switch` statement.
  **L855 CN**: 为 `switch` 语句提供默认分支。
- **L856 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L856 CN**: 延续周围的声明、表达式或控制流结构。
- **L857 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L857 CN**: 延续周围的声明、表达式或控制流结构。
- **L858 EN**: Executes statement `edit.descriptor);`.
  **L858 CN**: 执行语句 `edit.descriptor);`。
- **L859 EN**: Returns from the current function, often propagating a computed result.
  **L859 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L860 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L860 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L861 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L861 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L862 EN**: Blank line separates nearby declarations or logic blocks.
  **L862 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L863 EN**: Begins a template declaration parameterizing subsequent code.
  **L863 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L864 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L864 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 865-888

````cpp
    ListDirectedStatementState<Direction::Output> &list, const CHAR *x,
    std::size_t length) {
  bool ok{true};
  MutableModes &modes{io.mutableModes()};
  ConnectionState &connection{io.GetConnectionState()};
  if (modes.delim) {
    ok = ok && list.EmitLeadingSpaceOrAdvance(io);
    // Value is delimited with ' or " marks, and interior
    // instances of that character are doubled.
    auto EmitOne{[&](CHAR ch) {
      if (connection.NeedAdvance(1)) {
        ok = ok && io.AdvanceRecord();
      }
      ok = ok && EmitEncoded(io, &ch, 1);
    }};
    EmitOne(modes.delim);
    for (std::size_t j{0}; j < length; ++j) {
      // Doubled delimiters must be put on the same record
      // in order to be acceptable as list-directed or NAMELIST
      // input; however, this requirement is not always possible
      // when the records have a fixed length, as is the case with
      // internal output.  The standard is silent on what should
      // happen, and no two extant Fortran implementations do
      // the same thing when tested with this case.
````

- **L865 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L865 CN**: 延续周围的声明、表达式或控制流结构。
- **L866 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L866 CN**: 延续周围的声明、表达式或控制流结构。
- **L867 EN**: Executes statement `bool ok{true};`.
  **L867 CN**: 执行语句 `bool ok{true};`。
- **L868 EN**: Executes statement involving `mutableModes`.
  **L868 CN**: 执行涉及 `mutableModes` 的语句。
- **L869 EN**: Executes statement involving `GetConnectionState`.
  **L869 CN**: 执行涉及 `GetConnectionState` 的语句。
- **L870 EN**: Introduces conditional control flow with an `if` statement.
  **L870 CN**: 通过 `if` 语句引入条件控制流。
- **L871 EN**: Initializes or updates `ok`.
  **L871 CN**: 初始化或更新 `ok`。
- **L872 EN**: Comment documents intent or context: `Value is delimited with ' or " marks, and interior`.
  **L872 CN**: 注释记录了意图或上下文：`Value is delimited with ' or " marks, and interior`。
- **L873 EN**: Comment documents intent or context: `instances of that character are doubled.`.
  **L873 CN**: 注释记录了意图或上下文：`instances of that character are doubled.`。
- **L874 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L874 CN**: 延续周围的声明、表达式或控制流结构。
- **L875 EN**: Introduces conditional control flow with an `if` statement.
  **L875 CN**: 通过 `if` 语句引入条件控制流。
- **L876 EN**: Initializes or updates `ok`.
  **L876 CN**: 初始化或更新 `ok`。
- **L877 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L877 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L878 EN**: Initializes or updates `ok`.
  **L878 CN**: 初始化或更新 `ok`。
- **L879 EN**: Executes statement `}};`.
  **L879 CN**: 执行语句 `}};`。
- **L880 EN**: Executes statement involving `EmitOne`.
  **L880 CN**: 执行涉及 `EmitOne` 的语句。
- **L881 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L881 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L882 EN**: Comment documents intent or context: `Doubled delimiters must be put on the same record`.
  **L882 CN**: 注释记录了意图或上下文：`Doubled delimiters must be put on the same record`。
- **L883 EN**: Comment documents intent or context: `in order to be acceptable as list-directed or NAMELIST`.
  **L883 CN**: 注释记录了意图或上下文：`in order to be acceptable as list-directed or NAMELIST`。
- **L884 EN**: Comment documents intent or context: `input; however, this requirement is not always possible`.
  **L884 CN**: 注释记录了意图或上下文：`input; however, this requirement is not always possible`。
- **L885 EN**: Comment documents intent or context: `when the records have a fixed length, as is the case with`.
  **L885 CN**: 注释记录了意图或上下文：`when the records have a fixed length, as is the case with`。
- **L886 EN**: Comment documents intent or context: `internal output. The standard is silent on what should`.
  **L886 CN**: 注释记录了意图或上下文：`internal output. The standard is silent on what should`。
- **L887 EN**: Comment documents intent or context: `happen, and no two extant Fortran implementations do`.
  **L887 CN**: 注释记录了意图或上下文：`happen, and no two extant Fortran implementations do`。
- **L888 EN**: Comment documents intent or context: `the same thing when tested with this case.`.
  **L888 CN**: 注释记录了意图或上下文：`the same thing when tested with this case.`。

### Lines 889-912

````cpp
      // This runtime splits the doubled delimiters across
      // two records for lack of a better alternative.
      if (x[j] == static_cast<CHAR>(modes.delim)) {
        EmitOne(x[j]);
      }
      EmitOne(x[j]);
    }
    EmitOne(modes.delim);
  } else {
    // Undelimited list-directed output
    ok = ok && list.EmitLeadingSpaceOrAdvance(io, length > 0 ? 1 : 0, true);
    std::size_t put{0};
    std::size_t oneAtATime{
        connection.useUTF8<CHAR>() || connection.internalIoCharKind > 1
            ? 1
            : length};
    while (ok && put < length) {
      if (std::size_t chunk{std::min<std::size_t>(
              std::min<std::size_t>(length - put, oneAtATime),
              connection.RemainingSpaceInRecord())}) {
        ok = EmitEncoded(io, x + put, chunk);
        put += chunk;
      } else {
        ok = io.AdvanceRecord() && EmitAscii(io, " ", 1);
````

- **L889 EN**: Comment documents intent or context: `This runtime splits the doubled delimiters across`.
  **L889 CN**: 注释记录了意图或上下文：`This runtime splits the doubled delimiters across`。
- **L890 EN**: Comment documents intent or context: `two records for lack of a better alternative.`.
  **L890 CN**: 注释记录了意图或上下文：`two records for lack of a better alternative.`。
- **L891 EN**: Introduces conditional control flow with an `if` statement.
  **L891 CN**: 通过 `if` 语句引入条件控制流。
- **L892 EN**: Executes statement involving `EmitOne`.
  **L892 CN**: 执行涉及 `EmitOne` 的语句。
- **L893 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L893 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L894 EN**: Executes statement involving `EmitOne`.
  **L894 CN**: 执行涉及 `EmitOne` 的语句。
- **L895 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L895 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L896 EN**: Executes statement involving `EmitOne`.
  **L896 CN**: 执行涉及 `EmitOne` 的语句。
- **L897 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L897 CN**: 延续周围的声明、表达式或控制流结构。
- **L898 EN**: Comment documents intent or context: `Undelimited list-directed output`.
  **L898 CN**: 注释记录了意图或上下文：`Undelimited list-directed output`。
- **L899 EN**: Initializes or updates `ok`.
  **L899 CN**: 初始化或更新 `ok`。
- **L900 EN**: Executes statement `std::size_t put{0};`.
  **L900 CN**: 执行语句 `std::size_t put{0};`。
- **L901 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L901 CN**: 延续周围的声明、表达式或控制流结构。
- **L902 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L902 CN**: 延续周围的声明、表达式或控制流结构。
- **L903 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L903 CN**: 延续周围的声明、表达式或控制流结构。
- **L904 EN**: Executes statement `: length};`.
  **L904 CN**: 执行语句 `: length};`。
- **L905 EN**: Starts a `while` loop controlled by a runtime condition.
  **L905 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L906 EN**: Introduces conditional control flow with an `if` statement.
  **L906 CN**: 通过 `if` 语句引入条件控制流。
- **L907 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L907 CN**: 延续周围的声明、表达式或控制流结构。
- **L908 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L908 CN**: 延续周围的声明、表达式或控制流结构。
- **L909 EN**: Initializes or updates `ok`.
  **L909 CN**: 初始化或更新 `ok`。
- **L910 EN**: Initializes or updates `+`.
  **L910 CN**: 初始化或更新 `+`。
- **L911 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L911 CN**: 延续周围的声明、表达式或控制流结构。
- **L912 EN**: Initializes or updates `ok`.
  **L912 CN**: 初始化或更新 `ok`。

### Lines 913-936

````cpp
      }
    }
    list.set_lastWasUndelimitedCharacter(true);
  }
  return ok;
}

template <typename CHAR>
RT_API_ATTRS bool EditCharacterOutput(IoStatementState &io,
    const DataEdit &edit, const CHAR *x, std::size_t length) {
  int len{static_cast<int>(length)};
  if (edit.descriptor == 'A' && edit.variation == 'T') {
    // AT edit descriptor: output character value with trailing blanks
    // removed (F2023 13.7.5.3.1).
    while (len > 0 && x[len - 1] == static_cast<CHAR>(' ')) {
      --len;
    }
    return EmitEncoded(io, x, len);
  }
  int width{edit.width.value_or(len)};
  switch (edit.descriptor) {
  case 'A':
    break;
  case 'G':
````

- **L913 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L913 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L914 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L914 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L915 EN**: Executes statement involving `set_lastWasUndelimitedCharacter`.
  **L915 CN**: 执行涉及 `set_lastWasUndelimitedCharacter` 的语句。
- **L916 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L916 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L917 EN**: Returns from the current function, often propagating a computed result.
  **L917 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L918 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L918 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L919 EN**: Blank line separates nearby declarations or logic blocks.
  **L919 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L920 EN**: Begins a template declaration parameterizing subsequent code.
  **L920 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L921 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L921 CN**: 延续周围的声明、表达式或控制流结构。
- **L922 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L922 CN**: 延续周围的声明、表达式或控制流结构。
- **L923 EN**: Executes statement `int len{static_cast<int>(length)};`.
  **L923 CN**: 执行语句 `int len{static_cast<int>(length)};`。
- **L924 EN**: Introduces conditional control flow with an `if` statement.
  **L924 CN**: 通过 `if` 语句引入条件控制流。
- **L925 EN**: Comment documents intent or context: `AT edit descriptor: output character value with trailing blanks`.
  **L925 CN**: 注释记录了意图或上下文：`AT edit descriptor: output character value with trailing blanks`。
- **L926 EN**: Comment documents intent or context: `removed (F2023 13.7.5.3.1).`.
  **L926 CN**: 注释记录了意图或上下文：`removed (F2023 13.7.5.3.1).`。
- **L927 EN**: Starts a `while` loop controlled by a runtime condition.
  **L927 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L928 EN**: Executes statement `--len;`.
  **L928 CN**: 执行语句 `--len;`。
- **L929 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L929 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L930 EN**: Returns from the current function, often propagating a computed result.
  **L930 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L931 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L931 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L932 EN**: Executes statement involving `value_or`.
  **L932 CN**: 执行涉及 `value_or` 的语句。
- **L933 EN**: Begins a `switch` dispatch over discrete cases.
  **L933 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L934 EN**: Marks one `switch` case label.
  **L934 CN**: 标记一个 `switch` 的 case 标签。
- **L935 EN**: Breaks out of the current loop or switch.
  **L935 CN**: 跳出当前循环或 switch。
- **L936 EN**: Marks one `switch` case label.
  **L936 CN**: 标记一个 `switch` 的 case 标签。

### Lines 937-960

````cpp
    if (width == 0) {
      width = len;
    }
    break;
  case 'B':
    return EditBOZOutput<1>(io, edit,
        reinterpret_cast<const unsigned char *>(x), sizeof(CHAR) * length);
  case 'O':
    return EditBOZOutput<3>(io, edit,
        reinterpret_cast<const unsigned char *>(x), sizeof(CHAR) * length);
  case 'Z':
    return EditBOZOutput<4>(io, edit,
        reinterpret_cast<const unsigned char *>(x), sizeof(CHAR) * length);
  case 'L':
    return EditLogicalOutput(io, edit, *reinterpret_cast<const char *>(x));
  default:
    io.GetIoErrorHandler().SignalError(IostatErrorInFormat,
        "Data edit descriptor '%c' may not be used with a CHARACTER data item",
        edit.descriptor);
    return false;
  }
  return EmitRepeated(io, ' ', std::max(0, width - len)) &&
      EmitEncoded(io, x, std::min(width, len));
}
````

- **L937 EN**: Introduces conditional control flow with an `if` statement.
  **L937 CN**: 通过 `if` 语句引入条件控制流。
- **L938 EN**: Initializes or updates `width`.
  **L938 CN**: 初始化或更新 `width`。
- **L939 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L939 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L940 EN**: Breaks out of the current loop or switch.
  **L940 CN**: 跳出当前循环或 switch。
- **L941 EN**: Marks one `switch` case label.
  **L941 CN**: 标记一个 `switch` 的 case 标签。
- **L942 EN**: Returns from the current function, often propagating a computed result.
  **L942 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L943 EN**: Executes statement involving `sizeof`.
  **L943 CN**: 执行涉及 `sizeof` 的语句。
- **L944 EN**: Marks one `switch` case label.
  **L944 CN**: 标记一个 `switch` 的 case 标签。
- **L945 EN**: Returns from the current function, often propagating a computed result.
  **L945 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L946 EN**: Executes statement involving `sizeof`.
  **L946 CN**: 执行涉及 `sizeof` 的语句。
- **L947 EN**: Marks one `switch` case label.
  **L947 CN**: 标记一个 `switch` 的 case 标签。
- **L948 EN**: Returns from the current function, often propagating a computed result.
  **L948 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L949 EN**: Executes statement involving `sizeof`.
  **L949 CN**: 执行涉及 `sizeof` 的语句。
- **L950 EN**: Marks one `switch` case label.
  **L950 CN**: 标记一个 `switch` 的 case 标签。
- **L951 EN**: Returns from the current function, often propagating a computed result.
  **L951 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L952 EN**: Provides the default branch for a `switch` statement.
  **L952 CN**: 为 `switch` 语句提供默认分支。
- **L953 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L953 CN**: 延续周围的声明、表达式或控制流结构。
- **L954 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L954 CN**: 延续周围的声明、表达式或控制流结构。
- **L955 EN**: Executes statement `edit.descriptor);`.
  **L955 CN**: 执行语句 `edit.descriptor);`。
- **L956 EN**: Returns from the current function, often propagating a computed result.
  **L956 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L957 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L957 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L958 EN**: Returns from the current function, often propagating a computed result.
  **L958 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L959 EN**: Executes statement involving `EmitEncoded`.
  **L959 CN**: 执行涉及 `EmitEncoded` 的语句。
- **L960 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L960 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 961-984

````cpp

template RT_API_ATTRS bool EditIntegerOutput<1>(
    IoStatementState &, const DataEdit &, std::int8_t, bool);
template RT_API_ATTRS bool EditIntegerOutput<2>(
    IoStatementState &, const DataEdit &, std::int16_t, bool);
template RT_API_ATTRS bool EditIntegerOutput<4>(
    IoStatementState &, const DataEdit &, std::int32_t, bool);
template RT_API_ATTRS bool EditIntegerOutput<8>(
    IoStatementState &, const DataEdit &, std::int64_t, bool);
template RT_API_ATTRS bool EditIntegerOutput<16>(
    IoStatementState &, const DataEdit &, common::int128_t, bool);

template class RealOutputEditing<2>;
template class RealOutputEditing<3>;
template class RealOutputEditing<4>;
template class RealOutputEditing<8>;
template class RealOutputEditing<10>;
// TODO: double/double
template class RealOutputEditing<16>;

template RT_API_ATTRS bool ListDirectedCharacterOutput(IoStatementState &,
    ListDirectedStatementState<Direction::Output> &, const char *,
    std::size_t chars);
template RT_API_ATTRS bool ListDirectedCharacterOutput(IoStatementState &,
````

- **L961 EN**: Blank line separates nearby declarations or logic blocks.
  **L961 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L962 EN**: Begins a template declaration parameterizing subsequent code.
  **L962 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L963 EN**: Executes statement `IoStatementState &, const DataEdit &, std::int8_t, bool);`.
  **L963 CN**: 执行语句 `IoStatementState &, const DataEdit &, std::int8_t, bool);`。
- **L964 EN**: Begins a template declaration parameterizing subsequent code.
  **L964 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L965 EN**: Executes statement `IoStatementState &, const DataEdit &, std::int16_t, bool);`.
  **L965 CN**: 执行语句 `IoStatementState &, const DataEdit &, std::int16_t, bool);`。
- **L966 EN**: Begins a template declaration parameterizing subsequent code.
  **L966 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L967 EN**: Executes statement `IoStatementState &, const DataEdit &, std::int32_t, bool);`.
  **L967 CN**: 执行语句 `IoStatementState &, const DataEdit &, std::int32_t, bool);`。
- **L968 EN**: Begins a template declaration parameterizing subsequent code.
  **L968 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L969 EN**: Executes statement `IoStatementState &, const DataEdit &, std::int64_t, bool);`.
  **L969 CN**: 执行语句 `IoStatementState &, const DataEdit &, std::int64_t, bool);`。
- **L970 EN**: Begins a template declaration parameterizing subsequent code.
  **L970 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L971 EN**: Executes statement `IoStatementState &, const DataEdit &, common::int128_t, bool);`.
  **L971 CN**: 执行语句 `IoStatementState &, const DataEdit &, common::int128_t, bool);`。
- **L972 EN**: Blank line separates nearby declarations or logic blocks.
  **L972 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L973 EN**: Begins a template declaration parameterizing subsequent code.
  **L973 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L974 EN**: Begins a template declaration parameterizing subsequent code.
  **L974 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L975 EN**: Begins a template declaration parameterizing subsequent code.
  **L975 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L976 EN**: Begins a template declaration parameterizing subsequent code.
  **L976 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L977 EN**: Begins a template declaration parameterizing subsequent code.
  **L977 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L978 EN**: Comment documents intent or context: `TODO: double/double`.
  **L978 CN**: 注释记录了意图或上下文：`TODO: double/double`。
- **L979 EN**: Begins a template declaration parameterizing subsequent code.
  **L979 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L980 EN**: Blank line separates nearby declarations or logic blocks.
  **L980 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L981 EN**: Begins a template declaration parameterizing subsequent code.
  **L981 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L982 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L982 CN**: 延续周围的声明、表达式或控制流结构。
- **L983 EN**: Executes statement `std::size_t chars);`.
  **L983 CN**: 执行语句 `std::size_t chars);`。
- **L984 EN**: Begins a template declaration parameterizing subsequent code.
  **L984 CN**: 开始一个模板声明，为后续代码提供参数化能力。

### Lines 985-999

````cpp
    ListDirectedStatementState<Direction::Output> &, const char16_t *,
    std::size_t chars);
template RT_API_ATTRS bool ListDirectedCharacterOutput(IoStatementState &,
    ListDirectedStatementState<Direction::Output> &, const char32_t *,
    std::size_t chars);

template RT_API_ATTRS bool EditCharacterOutput(
    IoStatementState &, const DataEdit &, const char *, std::size_t chars);
template RT_API_ATTRS bool EditCharacterOutput(
    IoStatementState &, const DataEdit &, const char16_t *, std::size_t chars);
template RT_API_ATTRS bool EditCharacterOutput(
    IoStatementState &, const DataEdit &, const char32_t *, std::size_t chars);

RT_OFFLOAD_API_GROUP_END
} // namespace Fortran::runtime::io
````

- **L985 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L985 CN**: 延续周围的声明、表达式或控制流结构。
- **L986 EN**: Executes statement `std::size_t chars);`.
  **L986 CN**: 执行语句 `std::size_t chars);`。
- **L987 EN**: Begins a template declaration parameterizing subsequent code.
  **L987 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L988 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L988 CN**: 延续周围的声明、表达式或控制流结构。
- **L989 EN**: Executes statement `std::size_t chars);`.
  **L989 CN**: 执行语句 `std::size_t chars);`。
- **L990 EN**: Blank line separates nearby declarations or logic blocks.
  **L990 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L991 EN**: Begins a template declaration parameterizing subsequent code.
  **L991 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L992 EN**: Executes statement `IoStatementState &, const DataEdit &, const char *, std::size_t chars);`.
  **L992 CN**: 执行语句 `IoStatementState &, const DataEdit &, const char *, std::size_t chars);`。
- **L993 EN**: Begins a template declaration parameterizing subsequent code.
  **L993 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L994 EN**: Executes statement `IoStatementState &, const DataEdit &, const char16_t *, std::size_t chars);`.
  **L994 CN**: 执行语句 `IoStatementState &, const DataEdit &, const char16_t *, std::size_t chars);`。
- **L995 EN**: Begins a template declaration parameterizing subsequent code.
  **L995 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L996 EN**: Executes statement `IoStatementState &, const DataEdit &, const char32_t *, std::size_t chars);`.
  **L996 CN**: 执行语句 `IoStatementState &, const DataEdit &, const char32_t *, std::size_t chars);`。
- **L997 EN**: Blank line separates nearby declarations or logic blocks.
  **L997 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L998 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L998 CN**: 延续周围的声明、表达式或控制流结构。
- **L999 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L999 CN**: 延续周围的声明、表达式或控制流结构。

## Key Concepts / 关键概念

- **Scale / 规模**: The file contains approximately 999 source lines, which suggests a substantial implementation unit. / 该文件约有 999 行源码，说明它是一个较大的实现单元。
- **Fortran execution semantics / Fortran 执行语义**: The implementation materializes language-level behavior such as descriptors, I/O state, allocation, reductions, and error handling. / 实现把语言层面的行为具体化，例如描述符、I/O 状态、分配、归约与错误处理。
- **Runtime layering / 运行时分层**: Source files in this area usually bridge generated code, public runtime headers, and lower-level system facilities. / 此处源码通常桥接生成代码、公共运行时头文件以及更底层的系统设施。
- **Interface surface / 接口表面**: Direct includes such as `edit-output.h`, `flang-rt/runtime/emit-encoded.h`, `flang-rt/runtime/utf.h`, `flang/Common/real.h` show which nearby abstractions this file relies on first. / 直接包含的头文件（如 `edit-output.h`, `flang-rt/runtime/emit-encoded.h`, `flang-rt/runtime/utf.h`, `flang/Common/real.h`）展示了此文件首先依赖的周边抽象。
- **Primary entry points / 主要入口点**: Notable callables include `AddSpaceBeforeCharacter`, `EmitSuffix`, `IsInfOrNaN`, `EditFOutput`, `EditForGOutput`, `EditEXOutput`. / 值得关注的可调用实体包括 `AddSpaceBeforeCharacter`, `EmitSuffix`, `IsInfOrNaN`, `EditFOutput`, `EditForGOutput`, `EditEXOutput`。
- **Core types / 核心类型**: Important declared or referenced types include `Unsigned`, `decimal`. / 重要的已声明或被引用类型包括 `Unsigned`, `decimal`。
- **Namespaces / 命名空间**: The code uses namespaces such as `Fortran` to organize symbols. / 代码使用 `Fortran` 等命名空间来组织符号。

## Dependencies / 依赖关系

- **Project headers / 项目头文件**: `edit-output.h`, `flang-rt/runtime/emit-encoded.h`, `flang-rt/runtime/utf.h`, `flang/Common/real.h`, `flang/Common/uint128.h`. These provide subsystem-specific declarations. / 这些头文件提供子系统专用声明。
- **Standard or platform headers / 标准库或平台头文件**: `algorithm`. They connect the file to language-runtime or OS services. / 它们将该文件连接到语言运行时或操作系统服务。
- **Callable surface / 可调用表面**: Functions or methods defined here include `AddSpaceBeforeCharacter`, `EmitSuffix`, `IsInfOrNaN`, `EditFOutput`, `EditForGOutput`, `EditEXOutput`, `Edit`. These are likely the main entry points exported to nearby code. / 此处定义的函数或方法包括 `AddSpaceBeforeCharacter`, `EmitSuffix`, `IsInfOrNaN`, `EditFOutput`, `EditForGOutput`, `EditEXOutput`, `Edit`，它们通常是对周边代码暴露的主要入口。
- **Type coupling / 类型耦合**: Declared types such as `Unsigned`, `decimal` capture the data model shared with dependent code. / `Unsigned`, `decimal` 等声明类型体现了与依赖方共享的数据模型。
